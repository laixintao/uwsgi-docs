Offloading Websockets and Server-Sent Events AKA "Combine them with Django safely"
==================================================================================

作者：Roberto De Ioris

日期：20140315

免责声明
----------

这篇文章显示了将websockets（或者sse）与Django以一种“安全的方式”结合起来的一种相当高级的方法。它不会向你展示websockets和sse有多酷，或者如何用它们写出更好的应用，而是试图让你避免使用它们的最差实践。

在我看来，Python面向web的世界正面临着一种通信/市场问题：大量的人在非阻塞技术（例如gevent）上运行大量阻塞应用（例如django）只是因为有人告诉他们这很酷并且会解决他们所有的扩展问题。

这是完全错误，危险和邪恶的，你不能将阻塞应用和非阻塞引擎混在一起，即使是一个单一、非常小的阻塞部分都能潜在摧毁你整个栈。正如我已经说了几十次一样，如果你的应用是99.9999999%非阻塞的，那么它仍然是阻塞的。

并且不是，你的Django应用上的猴子补丁并非魔法。除非你正使用高度自定义的数据库适配器，对工作在非阻塞模式进行了调整，否则你就是错的。

以看起来就是个超级大混蛋的代价，我强烈建议你完全忽略人们让你将你的Django应用移到gevent, eventlet, tornado或者其他什么的，而不警告你你会遇到数百个问题的建议。

话虽如此，我爱gevent，它可能是uWSGI项目中支持得最好的（带perl的Coro::AnyEvent）循环引擎了。因此，在这篇文章中，我将使用gevent来为Django部分管理websocket/sse流量和纯多进程。

如果这最后一句对你来说像是废话，那么你可能不知道uWSGI的卸载是什么……


uWSGI卸载
----------------

这个概念并非是个新东西，或者是uWSGI特有的。诸如nodejs或者twisted这样的项目已经用它多年了。

.. note:: 一个提供静态文件服务的web应用的例子并不非常有趣，也不是用来展示的最好的东西，但是在稍后展示一个使用X-Sendfile的真实世界的例子的时候，会游泳。

想象这个简单的WSGI应用：

.. code-block:: python

   def application(env, start_response):
       start_response('200 OK',[('Content-Type','text/plain')])
       f = open('/etc/services')
       # do not do it, if the file is 4GB it will allocate 4GB of memory !!!
       yield f.read()

这将会简单返回/etc/services的内容。它是一个相当小的文件，因此在几毫秒时间内，你的进程将会准备好处理另一个请求。

那要是/etc/services是4千兆字节呢？你的进程（或者线程）将会阻塞几秒（甚至几分钟），并且不能够管理其他请求，直到完全传输这个文件。

如果你可以告诉其他线程为你发送这个文件，这样你就能够管理其他请求，岂不是很酷？

卸载就是这样的：它会为你提供一个或多个线程来为你完成一些简单并且慢点任务。哪种任务呢？所有那些可以以一种非阻塞方式管理的任务，因此单个线程就可以为你管理上千个传输。

你可以将其视为你的电脑中的DMA引擎，你的CPU将会编程DMA来将内存从控制器传输到RAM，然后将会被释放，以完成其他任务，同时DMA在后台工作。

要在uWSGI中启用卸载，你只需要添加 ``--offload-threads <n>`` 选项，其中<n>是每个进程要生成的线程数。 (一般而言，单个线程就够了，但是如果你想要使用/滥用你的多CPU核，那么随意增加)

一旦启用了卸载，那么每当uWSGI检测到一个操作能够安全被卸载的时候，它将自动使用它。

在python/WSGI中，任何对wsgi.file_wrapper的使用将会被自动卸载，以及当你使用uWSGI代理特性来传递请求到其他使用uwsgi或者HTTP协议到服务器时。

一个很酷的例子 (甚至在uWSGI文档的Snippets页面也有展示) 是实现一个卸载助力的X-Sendfile特性：

.. code-block:: ini

   [uwsgi]
   ; load router_static plugin (compiled in by default in monolithic profiles)
   plugins = router_static
   
   ; spawn 2 offload threads
   offload-threads = 2
   
   ; files under /etc can be safely served (DANGEROUS !!!)
   static-safe = /etc
   
   ; collect the X-Sendfile response header as X_SENDFILE var
   collect-header = X-Sendfile X_SENDFILE
   
   ; if X_SENDFILE is not empty, pass its value to the "static" routing action (it will automatically use offloading if available)
   response-route-if-not = empty:${X_SENDFILE} static:${X_SENDFILE}

   ; now the classic options
   plugins = python
   ; bind to HTTP port 8080
   http-socket = :8080
   ; load a simple wsgi-app
   wsgi-file = myapp.py
  
  
现在，在我们的应用中，我们可以X-Sendfile来在无阻塞情况下发送静态文件：

.. code-block:: python

   def application(env, start_response):
       start_response('200 OK',[('X-Sendfile','/etc/services')])
       return []


在这篇文章中将会使用一个非常类似的概念：我们将会使用一个正常的Django来设置我们的会话，来认证用户，以及任意（快的）东东，然后我们会返回一个特别的头，它会指示uWSGI卸载连接到另一个uWSGI实例 (监听一个私有socket)，这个实例将使用gevent，以一种非阻塞方式管理websocket/sse事务。

我们的SSE应用
-----------

SSE部分将非常简单，一个基于gevent的WSGI应用将会每秒发送当前时间：

.. code-block:: python

   from sse import Sse
   import time

   def application(e, start_response):
       print e
       # create the SSE session
       session = Sse()
       # prepare HTTP headers
       headers = []
       headers.append(('Content-Type','text/event-stream'))
       headers.append(('Cache-Control','no-cache'))
       start_response('200 OK', headers)
       # enter the loop
       while True:
           # monkey patching will prevent sleep() blocking
           time.sleep(1)
           # add the message
           session.add_message('message', str(time.time()))
           # send to the client
           yield str(session)
           
让我们在/tmp/foo UNIX socket上运行它 (将应用保存为sseapp.py)

.. code-block:: sh

   uwsgi --wsgi-file sseapp.py --socket /tmp/foo --gevent 1000 --gevent-monkey-patch
   
(time.sleep()需要猴子补丁，如果你想要/喜欢的话，随意使用gevent原语来休眠)

（无趣的）HTML/Javascript
----------------------------

.. code-block:: html

   <html>
       <head>
       </head>
       <body>
         <h1>Server sent events</h1>
         <div id="event"></div>
         <script type="text/javascript">

         var eventOutputContainer = document.getElementById("event");
         var evtSrc = new EventSource("/subscribe");

         evtSrc.onmessage = function(e) {
             console.log(e.data);
             eventOutputContainer.innerHTML = e.data;
         };

         </script>
       </body>
     </html>

它非常简单，它将连接到/subscribe，并且将开始等待事件。

Django视图
---------------

我们的django视图，将非常简单，它将简单生成一个特色的响应头 (我们讲称之为X-Offload-to-SSE)，并且把登录用户的用户名作为它的值：

.. code-block:: python

   def subscribe(request):
       response = HttpResponse()
       response['X-Offload-to-SSE'] = request.user
       return response
       
现在，我们已经为“高级”部分准备好了。

让我们卸载SSE事务
---------------------------------

配置看起来会有点复杂，但是它与之前看到的X-Sendfile概念相同：

.. code-block:: ini

   [uwsgi]
   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   ; collect X-Offload-to-SSE header and store in var X_OFFLOAD
   collect-header = X-Offload-to-SSE X_OFFLOAD
   ; if X_OFFLOAD is defined, do not send the headers generated by Django
   response-route-if-not = empty:${X_OFFLOAD} disableheaders:
   ; if X_OFFLOAD is defined, offload the request to the app running on /tmp/foo
   response-route-if-not = empty:${X_OFFLOAD} uwsgi:/tmp/foo,0,0
   
唯一“新的”部分是使用 ``disableheaders`` 路由动作。这是必须的，否则Django生成的头将会伴着由基于gevent的应用生成的头发送。

你可以避免它 (记住，只在2.0.3添加了 ``disableheaders`` )，在gevent应用中移除到start_response()到调用 (冒着被一些WSGI神诅咒的风险)，然后修改Django视图来设置正确的头部：

.. code-block:: python

   def subscribe(request):
       response = HttpResponse()
       response['Content-Type'] = 'text/event-stream'
       response['X-Offload-to-SSE'] = request.user
       return response
       
最终，你或许想要更加“精简”，并简单检测'text/event-stream' content_type存在：

.. code-block:: ini

   [uwsgi]
   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   ; collect Content-Type header and store in var CONTENT_TYPE
   collect-header = Content-Type CONTENT_TYPE
   ; if CONTENT_TYPE is 'text/event-stream', forward the request
   response-route-if = equal:${CONTENT_TYPE};text/event-stream uwsgi:/tmp/foo,0,0
   
   
现在，如何在gevent应用中访问Django登录用户的用户名呢？

你应该注意到，gevent应用在每个请求中打印了WSGI环境变量的内容。That environment is the same
of the Django app + the collected headers. So accessing environ['X_OFFLOAD'] will return the logged username. (obviously in the second example, where the content type is used, the variable with the username is no longer collected, so you should fix it)

You can pass all of the information you need using the same approach, you can collect all of the vars you need and so on.

You can even add variables at runtime:


.. code-block:: ini

   [uwsgi]
   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   ; collect Content-Type header and store in var CONTENT_TYPE
   collect-header = Content-Type CONTENT_TYPE
   
   response-route-if = equal:${CONTENT_TYPE};text/event-stream addvar:FOO=BAR
   response-route-if = equal:${CONTENT_TYPE};text/event-stream addvar:TEST1=TEST2
   
   ; if CONTENT_TYPE is 'text/event-stream', forward the request
   response-route-if = equal:${CONTENT_TYPE};text/event-stream uwsgi:/tmp/foo,0,0
   
或者 (使用goto以获得更好的可读性):

.. code-block:: ini

   [uwsgi]
   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   ; collect Content-Type header and store in var CONTENT_TYPE
   collect-header = Content-Type CONTENT_TYPE
   
   response-route-if = equal:${CONTENT_TYPE};text/event-stream goto:offload
   response-route-run = last:
   
   response-route-label = offload
   response-route-run = addvar:FOO=BAR
   response-route-run = addvar:TEST1=TEST2
   response-route-run = uwsgi:/tmp/foo,0,0


使用uwsgi api (>= uWSGI 2.0.3) 进行简化
-------------------------------------------------------

While dealing with headers is pretty HTTP friendly, uWSGI 2.0.3 added the possibility to define per-request variables
directly in your code.

This allows a more "elegant" approach (even if highly non-portable):

.. code-block:: python

   import uwsgi
   
   def subscribe(request):
       uwsgi.add_var("LOGGED_IN_USER", request.user)
       uwsgi.add_var("USER_IS_UGLY", "probably")
       uwsgi.add_var("OFFLOAD_TO_SSE", "y")
       uwsgi.add_var("OFFLOAD_SERVER", "/tmp/foo")
       return HttpResponse()
       
现在，配置可以修改成更优雅：

.. code-block:: ini

   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   ; if OFFLOAD_TO_SSE is 'y', do not send the headers generated by Django
   response-route-if = equal:${OFFLOAD_TO_SSE};y disableheaders:
   ; if OFFLOAD_TO_SSE is 'y', offload the request to the app running on 'OFFLOAD_SERVER'
   response-route-if = equal:${OFFLOAD_TO_SSE};y uwsgi:${OFFLOAD_SERVER},0,0
   
Have you noted how we allowed the Django app to set the backend server to use using a request variable?

Now we can go even further. We will not use the routing framework (except for disabling headers generation):

.. code-block:: python

   import uwsgi
   
   def subscribe(request):
       uwsgi.add_var("LOGGED_IN_USER", request.user)
       uwsgi.add_var("USER_IS_UGLY", "probably")
       uwsgi.route("uwsgi", "/tmp/foo,0,0")
       return HttpResponse()
       
以及一个简单的：

.. code-block:: ini

   ; the boring part
   http-socket = :9090
   offload-threads = 2
   wsgi-file = sseproject/wsgi.py
   
   response-route = ^/subscribe disableheaders:


What about Websockets ?
-----------------------

We have seen how to offload SSE (that are mono-directional). We can offload websockets too (that are bidirectional).

The concept is the same, you only need to ensure (as before) that no headers are sent by django, (otherwise the websocket handshake will fail) and then you
can change your gevent app:

.. code-block:: python

   import time
   import uwsgi

   def application(e, start_response):
       print e
       uwsgi.websocket_handshake()
       # enter the loop
       while True:
           # monkey patching will prevent sleep() to block
           time.sleep(1)
           # send to the client
           uwsgi.websocket_send(str(time.time()))
           
使用redis或者uWSGI缓存框架
--------------------------------------

Request vars are handy (and funny), but they are limited (see below). If you need to pass a big amount of data between Django and the sse/websocket app, Redis
is a great way (and works perfectly with gevent). Basically you store infos from django to redis and than you pass only the hash key (via request vars) to the sse/websocket app.

The same can be accomplished with the uWSGI caching framework, but take into account redis has a lot of data primitives, while uWSGI only supports key->value items.

常见陷阱
---------------

* The amount of variables you can add per-request is limited by the uwsgi packet buffer (default 4k). You can increase it up to 64k with the --buffer-size option.

* This is the whole point of this article: do not use the Django ORM in your gevent apps unless you know what you are doing!!! (read: you have a django database adapter that supports gevent and does not suck compared to the standard ones...)

* Forget about finding a way to disable headers generation in django. This is a "limit/feature" of its WSGI adapter, use the uWSGI facilities (if available) or do not generate headers in your gevent app. Eventually you can modify wsgi.py in this way:

.. code-block:: python

   """
   WSGI config for sseproject project.

   It exposes the WSGI callable as a module-level variable named ``application``.

   For more information on this file, see
   https://docs.djangoproject.com/en/1.6/howto/deployment/wsgi/
   """

   import os
   os.environ.setdefault("DJANGO_SETTINGS_MODULE", "sseproject.settings")

   from django.core.wsgi import get_wsgi_application
   django_application = get_wsgi_application()
   
   def fake_start_response(status, headers, exc_info=None):
       pass
   
   def application(environ, start_response):
       if environ['PATH_INFO'] == '/subscribe':
           return django_application(environ, fake_start_response)
       return django_application(environ, start_response)
