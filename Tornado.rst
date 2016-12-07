Tornado循环引擎
=======================

可用版本自 ```uWSGI 1.9.19-dev``` 起

支持挂起引擎： ```greenlet```

支持CPython版本： ```所有支持tornado的版本```


tornado循环引擎允许你将你的uWSGI栈与Tornado IOLoop类集成。

基本上，服务器的每一个I/O操作会被映射到一个tornado IOLoop回调。进行RPC、远程缓存或者简单的写入响应则由Tornado引擎管理。

由于uWSGI不是用一个基于回调的编程方法写的，因此与那种类型的库集成需要某些类型“挂起”引擎 (绿色线程/协程)

目前唯一支持的挂起引擎是"greenlet"。Stackless python也能用 (需要测试)。

当前并不支持PyPy (尽管因为continulet，技术上是可行的)。如果你感兴趣的话，给Unbit员工发邮件吧。

为什么？
*******
Tornado项目自己包含了一个简单的WSGI服务器。在于Gevent插件相同的思想下，Loop引擎的目的是允许外部项目使用（尽情使用）uWSGI api，从而获得更好的性能、多功能性和 (也许是最重要的) 资源使用。

在你的tornado应用中，可以使用所有的uWSGI子系统 (从缓存，到websockets，到度量)，而WSGI引擎则是其中一个久经考验的uWSGI。


安装
************

当前默认不内置tornado插件。要在单个二进制文件中同时拥有tornado和greenlet，你可以这样

.. code-block:: sh

   UWSGI_EMBED_PLUGINS=tornado,greenlet pip install tornado greenlet uwsgi
   
或者 (来自uWSGI源代码，如果你已经安装了tornado和greenlet的话)

.. code-block:: sh

   UWSGI_EMBED_PLUGINS=tornado,greenlet make

运行之
**********

``--tornado`` 选项是由tornado插件公开的，允许你设置最佳参数：

.. code-block:: sh

   uwsgi --http-socket :9090 --wsgi-file myapp.py --tornado 100 --greenlet
   
这将会在http端口9090上运行一个uWSGI实例，使用tornado作为I/O（和时间）管理，greenlet作为挂起引擎

会分配100个异步核心，允许你管理多达100个并发请求

集成WSGI和tornado api
*************************************

出于WSGI的工作方式，处理基于回调的编程是相当难的 (如果有可能的话)。

有了greenlet，我们可以挂起我们的WSGI可调用的执行，直到一个tornado IOLoop事件可用：

.. code-block:: py

   from tornado.httpclient import AsyncHTTPClient
   import greenlet
   import functools
   
   # this gives us access to the main IOLoop (the same used by uWSGI)
   from tornado.ioloop import IOLoop
   io_loop = IOLoop.instance()
   
   # this is called at the end of the external HTTP request
   def handle_request(me, response):
       if response.error:
           print("Error:", response.error)
       else:
           me.result = response.body
       # back to the WSGI callable
       me.switch()
           
    def application(e, sr):
        me = greenlet.getcurrent()
        http_client = AsyncHTTPClient()
        http_client.fetch("http://localhost:9191/services", functools.partial(handle_request, me))
        # suspend the execution until an IOLoop event is available
        me.parent.switch()
        sr('200 OK', [('Content-Type','text/plain')])
        return me.result

欢迎来到回调地狱
************************

一如既往，判断编程方法并非uWSGI的工作。它是为系统管理员提供的工具，而系统管理员应该宽容开发者的选择。

使用这个方法，你将很快体验到的事情之一是回调地狱。

让我们扩展前面的例子，在发送响应回客户端之前等待10秒

.. code-block:: py

   from tornado.httpclient import AsyncHTTPClient
   import greenlet
   import functools
   
   # this gives us access to the main IOLoop (the same used by uWSGI)
   from tornado.ioloop import IOLoop
   io_loop = IOLoop.instance()
   
   def sleeper(me):
       #TIMED OUT
       # finally come back to WSGI callable
       me.switch()
   
   # this is called at the end of the external HTTP request
   def handle_request(me, response):
       if response.error:
           print("Error:", response.error)
       else:
           me.result = response.body
       # add another callback in the chain
       me.timeout = io_loop.add_timeout(time.time() + 10, functools.partial(sleeper, me))
           
    def application(e, sr):
        me = greenlet.getcurrent()
        http_client = AsyncHTTPClient()
        http_client.fetch("http://localhost:9191/services", functools.partial(handle_request, me))
        # suspend the execution until an IOLoop event is available
        me.parent.switch()
        # unregister the timer
        io_loop.remove_timeout(me.timeout)
        sr('200 OK', [('Content-Type','text/plain')])
        return me.result


这里，我们链接了两个回调，最后一个负责将控制权交还WSGI可调用

代码可能看起来丑或者过于复杂 (与其他诸如gevent的方法相比)，但是，这基本上是提高并发性最有效的方法 (同时在内存使用和性能方面)。诸如node.js这样的技术由于它们允许完成的结果，它们正变得流行起来。


WSGI生成器 (aka yield all over the place)
**********************************************

以下面的WSGI应用为例：

.. code-block:: py

   def application(e, sr):
       sr('200 OK', [('Content-Type','text/html')])
       yield "one"
       yield "two"
       yield "three"

如果你已经使用uWSGI异步模式，那么你就会知道每次yield内部调用使用的挂起引擎 (在我们的例子中，是greenlet.switch())。

那意味着，我们在调用"application()"后会立即进入tornado IOLoop引擎。如果我们不在等待事件，那么能如何将控制权交还给我们的可回调对象？

已扩展uWSGI异步API来支持"schedule_fix"钩子。它允许你在调用挂起引擎后立即调用一个钩子。

在tornado这种情况下，这个钩子会被映射到某些像这样的东东：

.. code-block:: py

   io_loop.add_callback(me.switch)
   
通过这种方式，在每次yield之后，一个me.switch()函数就会被调用，从而让可回调对象恢复。

有了这个钩子，你可以透明地托管标准的WSGI应用，而无需更改它们。


绑定和监听Tornado
**********************************

在每一个worker中，在fork()之后会执行Tornado IOLoop。如果你想把Tornado绑定到网络地址上，那么记得为每个worker使用不同的端口：

.. code-block:: py

   from uwsgidecorators import *
   import tornado.web

   # this is our Tornado-managed app
   class MainHandler(tornado.web.RequestHandler):
       def get(self):
           self.write("Hello, world")

   t_application = tornado.web.Application([
       (r"/", MainHandler),
   ])
   
   # here happens the magic, we bind after every fork()
   @postfork
   def start_the_tornado_servers():
       application.listen(8000 + uwsgi.worker_id())
       
   # this is our WSGI callable managed by uWSGI
   def application(e, sr):
       ...
   
   
记住：不要启动IOLoop类。一旦安装完成，uWSGI将会自己启动它。
