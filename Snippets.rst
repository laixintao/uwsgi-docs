代码片段
========

这是uWSGI特性的一些最“有趣”的使用的集合。

X-Sendfile模拟
--------------------

即使你的前端代理/web服务器不支持X-Sendfile (或者不能访问你的静态资源)，但是你可以使用uWSGI的内部卸载（你的进程/线程将会委托实际的静态文件服务给卸载线程）来模拟它。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_static plugin (compiled in by default in monolithic profiles)
   plugins = router_static
   ; spawn 2 offload threads
   offload-threads = 2
   ; files under /private can be safely served
   static-safe = /private
   ; collect the X-Sendfile response header as X_SENDFILE var
   collect-header = X-Sendfile X_SENDFILE
   ; if X_SENDFILE is not empty, pass its value to the "static" routing action (it will automatically use offloading if available)
   response-route-if-not = empty:${X_SENDFILE} static:${X_SENDFILE}
   

强制使用HTTPS
-------------

这将会强制整个站点使用HTTPS。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   
而这个只会强制 ``/admin`` (使用HTTPS)

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route = ^/admin goto:https
   ; stop the chain
   route-run = last:
   
   route-label = https
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   
最终，你可能也想要发送HSTS (强制安全传输技术，HTTP Strict Transport Security)头。

.. code-block:: ini

   [uwsgi]
   ...
   ; load router_redirect plugin (compiled in by default in monolithic profiles)
   plugins = router_redirect
   route-if-not = equal:${HTTPS};on redirect-permanent:https://${HTTP_HOST}${REQUEST_URI}
   route-if = equal:${HTTPS};on addheader:Strict-Transport-Security: max-age=31536000
   
   
Python自动重载 (DEVELOPMENT ONLY!)
-----------------------------------------

在生产环境中，你可以监控文件/目录的改动来触发重载 (touch-reload, fs-reload...).

在开发期间，拥有一个用于所有的重载/已用python模块的监控器是很方便的。但是请只在开发期间才使用它。

检查是通过一个以指定频率扫描模块列表的线程来完成的：

.. code-block:: ini

   [uwsgi]
   ...
   py-autoreload = 2
   
将会每2秒检查一次python模块的改动，并最终重启实例。

再次说明：

.. warning:: 只在开发时使用它。


全栈CGI设置
--------------------

这个例子从一个uWSGI的邮件列表线程生成。

我们在/var/www中有静态文件，在/var/cgi中有cgi。将会使用/cgi-bin挂载点访问cgi。所以将会在到/cgi-bin/foo.lua的请求上运行/var/cgi/foo.lua

.. code-block:: ini

   [uwsgi]
   workdir = /var
   ipaddress = 0.0.0.0
 
   ; start an http router on port 8080
   http = %(ipaddress):8080
   ; enable the stats server on port 9191
   stats = 127.0.0.1:9191
   ; spawn 2 threads in 4 processes (concurrency level: 8)
   processes = 4
   threads = 2
   ; drop privileges
   uid = nobody
   gid = nogroup
   
   ; serve static files in /var/www
   static-index = index.html
   static-index = index.htm
   check-static = %(workdir)/www
   
   ; skip serving static files ending with .lua
   static-skip-ext = .lua

   ; route requests to the CGI plugin
   http-modifier1 = 9
   ; map /cgi-bin requests to /var/cgi
   cgi = /cgi-bin=%(workdir)/cgi
   ; only .lua script can be executed
   cgi-allowed-ext = .lua
   ; .lua files are executed with the 'lua' command (it avoids the need of giving execute permission to files)
   cgi-helper = .lua=lua
   ; search for index.lua if a directory is requested
   cgi-index = index.lua
   
   
不同挂载点中的多个flask应用
--------------------------------------------

写3个flask应用：

.. code-block:: py

   #app1.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app1"
       

.. code-block:: py

   #app2.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app2"
       
       
.. code-block:: py

   #app3.py
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def hello():
       return "Hello World! i am app3"

每个将会被分别挂载在/app1, /app2, /app3上

要在uWSGI中挂载一个使用指定“键”的应用，使用--mount选项：

```
--mount <mountpoint>=<app>
```

在我们的例子中，我们想要挂载3个python应用，每个使用WSGI SCRIPT_NAME变量名作为键：

.. code-block :: ini
   
   [uwsgi]
   plugin = python
   mount = /app1=app1.py
   mount = /app2=app2.py
   mount = /app3=app3.py
   ; generally flask apps expose the 'app' callable instead of 'application'
   callable = app

   ; tell uWSGI to rewrite PATH_INFO and SCRIPT_NAME according to mount-points
   manage-script-name = true

   ; bind to a socket
   socket = /var/run/uwsgi.sock



现在，直接将你的webserver.proxy指向实例socket (无需进行额外的配置)

注意事项：默认情况下，每个应用都会在一个新的python解释器中加载 (那意味着每个应用都有一个相当棒的隔离的名字空间)。如果你想要在相同的python vm中加载所有的应用，那么使用--single-interpreter选项。

另一个注意事项：你或许发现到一个不起眼的"modifier1 30"引用技巧。它已经被弃用了，并且非常丑陋。uWSGI能够以许多种高级方式重写请求变量

最后一个注意事项：默认情况下，第一个加载的应用作为"默认应用"挂载。当没有挂载点匹配上的时候，将会使用那个应用。


OSX上的rbenv (应该也能在其他平台上用)
-------------------------------------------------

安装rbenv

.. code-block:: sh

   brew update
   brew install rbenv ruby-build
   
(不要像经典的howto中描述的那样在.bash_profile中设置魔术行，因为我们希望不破坏环境，并且让uWSGI摆脱它)

获取一个uWSGI压缩包，并且构建'nolang'版本 (它是一个单片版本，其中并未编译任何语言插件)

.. code-block:: sh

   wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
   tar zxvf uwsgi-latest.tar.gz
   cd uwsgi-xxx
   make nolang
   
现在，开始安装你需要的ruby版本

.. code-block:: sh

   rbenv install 1.9.3-p551
   rbenv install 2.1.5
   
并且安装你需要的gem (这个例子中是sinatra)：

.. code-block:: sh

   # set the current ruby env
   rbenv local 1.9.3-p551
   # get the path of the gem binary
   rbenv which gem
   # /Users/roberta/.rbenv/versions/1.9.3-p551/bin/gem
   /Users/roberta/.rbenv/versions/1.9.3-p551/bin/gem install sinatra
   # from the uwsgi sources directory, build the rack plugin for 1.9.3-p551, naming it rack_193_plugin.so
   # the trick here is changing PATH to find the right ruby binary during the build procedure
   PATH=/Users/roberta/.rbenv/versions/1.9.3-p551/bin:$PATH ./uwsgi --build-plugin "plugins/rack rack_193"
   # set ruby 2.1.5
   rbenv local 2.1.5
   rbenv which gem
   # /Users/roberta/.rbenv/versions/2.1.5/bin/gem
   /Users/roberta/.rbenv/versions/2.1.5/bin/gem install sinatra
   PATH=/Users/roberta/.rbenv/versions/2.1.5/bin:$PATH ./uwsgi --build-plugin "plugins/rack rack_215"
   
现在，从一个ruby切换到另一个，只需修改插件：

.. code-block:: ini

   [uwsgi]
   plugin = rack_193
   rack = config.ru
   http-socket = :9090
   
或者

.. code-block:: ini

   [uwsgi]
   plugin = rack_215
   rack = config.ru
   http-socket = :9090

确保插件存储在当前的工作目录中，或者设置plugins-dir指令，又或者像这样涌绝对路径指定它们

.. code-block:: ini

   [uwsgi]
   plugin = /foobar/rack_215_plugin.so
   rack = config.ru
   http-socket = :9090


认证的WebSocket代理
-----------------------------

应用服务器识别websocket流量，相对于应用自身策略/基础架构，使用任何CGI变量来认证/鉴权用户，然后卸载/代理请求到一个简单的kafka-websocket后端。

首先，创建 ``auth_kafka.py``:

.. code-block:: python

   from pprint import pprint
   
   def application(environ, start_response):
       start_response('200 OK', [('Content-Type', 'text/plain')])
       return ['It Works!']
   
   def auth_kafka(request_uri, http_cookie, http_authorization):
       pprint(locals())
       return 'true'
   
   import uwsgi
   uwsgi.register_rpc('auth_kafka', auth_kafka)
   
然后创建 ``auth_kafka.ini``:

.. code-block:: ini

   [uwsgi]
   
   ; setup
   http-socket = 127.0.0.1:8000
   master = true
   module = auth_kafka
   
   ; critical! else worker timeouts apply to proxied websocket connections
   offload-threads = 2
   
   ; match websocket protocol
   kafka-ws-upgrade-regex = ^[Ww]eb[Ss]ocket$
   
   ; DRY place for websocket check
   is-kafka-ws-request =  regexp:${HTTP_UPGRADE};%(kafka-ws-upgrade-regex)
   
   ; location of the kafka-ws server
   kafka-ws-host = 127.0.0.1:7080
   
   ; base endpoint uri for websocket server
   kafka-ws-endpoint-uri = /v2/broker/
   
   ; call auth_kafka(...); if AUTH_KAFKA gets set, request is good!
   route-if = %(is-kafka-ws-request) rpcvar:AUTH_KAFKA auth_kafka ${REQUEST_URI} ${HTTP_COOKIE} ${HTTP_AUTHORIZATION}
   
   ; update request uri to websocket endpoint (rewrite only changes PATH_INFO?)
   route-if-not = empty:${AUTH_KAFKA} seturi:%(kafka-ws-endpoint-uri)?${QUERY_STRING}
   
   ; route the request to our websocket server
   route-if-not = empty:${AUTH_KAFKA} httpdumb:%(kafka-ws-host)
   
启动一个"kafka-websocket"服务器：

.. code-block:: bash

   nc -l -k -p 7080
   
现在，在一个web浏览器中访问 ``http://127.0.0.1:8000`` ！你应该看到 ``Hello!`` 。打开chrome的查看器或者firebug，然后输入：

.. code-block:: javascript

   ws = new WebSocket('ws://127.0.0.1:8000/?subscribe=true')
   
你应该看到这个请求代理到你的 ``nc`` 命令！这个模式允许内部网络托管一个或多或少全开/通用的kafka -> websocket网关，并且委托认证需求给应用服务器。使用 ``offload-threads`` 意味着代理请求 *不会* 阻塞worker；使用 ``httpdumb`` 避免了重整请求 (``http`` 动作强制使用 ``HTTP/1.0``)


SELinux和uWSGI
-----------------

SELinux允许你将web应用进程彼此隔离，并且限制每个程序只用于自身目的。应用可以被放置于高度隔离的独立沙箱中，将它们与其他应用以及底层操作系统分离开来。由于SELinux是在内核中实现的，因此不需要特殊编写或修改应用就能让其在SELinux之下使用。github上有一个 `SELinux security policy for web applications  <https://github.com/reinow/sepwebapp>`_ ，非常适于uWSGI。这个安全策略也支持运行在一个域中的uWSGI emperor进程，以及运行在一个分隔域中的每个web应用的worker进程，即使使用了Linux名字空间，worker进程也只要求最小的特权。当然，使用SELinux不要求emperor模式，或者Linux名字空间。

在Linux上，有可能使用文件系统、ipc、uts、网络、pid和uid的专有视图来运行每个vassal。然后，每个vassal可以，比方说，修改文件系统布局、网络和主机名，而无需损坏主系统。有了这个设置，特权任务，例如挂载文件系统、设置主机名、配置网络和设置worker进程的gid和uid就可以在修改vassal进程的SELinux安全上下文之前完成了，确保每个worker进程只需要最少的特权。

首先，配置、编译和加载SELinux web应用安全策略。然后，重新标记应用文件。关于如何配置web应用策略的进一步信息可以在 `SELinux security policy for web applications <https://github.com/reinow/sepwebapp>`_ 中包含的README.md上找到。最后，在每个vassal的配置文件中，调用libselinux的setcon函数来设置web应用的SELinux安全上下文：

.. code-block:: ini

	[uwsgi]
	...
	hook-as-user = callret:setcon system_u:system_r:webapp_id_t:s0

其中，id是域的标识。例如，foo是webapp_foo_t域的标识。

也许需要使用--dlopen选项在uWSGI地址空间内加载libselinux:

.. code-block:: ini

	/path/to/uwsgi --dlopen /path/to/libselinux.so
