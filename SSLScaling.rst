扩展SSL连接 (uWSGI 1.9)
===================================

在一个集群中分发SSL服务器是个难点。最大的问题是在不同节点之间共享SSL会话。

由于OpenSSL对会话管理方式的限制，在非阻塞服务器上，问题被放大了。

例如，你不能在Memcached服务器上共享会话，并且以非阻塞方式访问它们。

时至今日，一种常见的解决方法 (嗯，好吧，或许是一种妥协)是使用单个SSL终止器来平衡到多个非加密后端的请求。这个解决方法还挺有效的，但是显然，它不能够扩展。

从uWSGI 1.9-dev起，已添加了一个分布式缓存的实现（基于 *stud* 项目）。

设置1：使用uWSGI缓存来存储SSL会话
*******************************************************

你可以配置uWSGI的SSL子系统来使用共享缓存。根据缓存项的过期值，SSL会话会超时。这样，缓存清道夫线程（由master管理）将会销毁缓存中的会话。

.. important:: 操作的次序是重要的。 ``cache`` 选项必须在 ``ssl-sessions-use-cache`` 和 ``https`` 选项之前指定。

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache = 20000
   ; 4k per object is enough for SSL sessions
   cache-blocksize = 4096
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = true
   ; set SSL session timeout (in seconds)
   ssl-sessions-timeout = 300
   ; set the session context string (see later)
   https-session-context = foobar
   ; spawn an HTTPS router
   https = 192.168.173.1:8443,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same session cache)
   http-processes = 8
   ; add a bunch of uwsgi nodes to relay traffic to
   http-to = 192.168.173.10:3031
   http-to = 192.168.173.11:3031
   http-to = 192.168.173.12:3031
   ; add stats
   stats = 127.0.0.1:5001

现在，启动你的HTTPS路由器，然后telnet到端口5001。在JSON输出的"cache"对象下，你应该看到值"items"和"hits"正在增长。值"miss"在每次于缓存中找不到一个会话的时候就会增长。它是用户可以期望的SSL性能的一个好度量。

设置2：同步不同HTTPS路由器的缓存
******************************************************

目标是同步每个分布缓存中的每个新会话。要完成这个目标，你必须在每个实例中生成一个特殊的线程 (``cache-udp-server``)，然后列出所有应该被同步的远程服务器。

可以使用一个纯TCP负载均衡器 (例如HAProxy或者uWSGI的Rawrouter)来在各种HTTPS路由器之间均衡负载。

下面是一个可能的Rawrouter配置。

.. code-block:: ini

   [uwsgi]
   master = true
   rawrouter = 192.168.173.99:443
   rawrouter-to = 192.168.173.1:8443
   rawrouter-to = 192.168.173.2:8443
   rawrouter-to = 192.168.173.3:8443
   
现在，你可以配置第一个节点 (新的选项位于.ini配置的尾部)

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache = 20000
   ; 4k per object is enough for SSL sessions
   cache-blocksize = 4096
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = true
   ; set SSL session timeout (in seconds)
   ssl-sessions-timeout = 300
   ; set the session context string (see later)
   https-session-context = foobar
   ; spawn an HTTPS router
   https = 192.168.173.1:8443,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same session cache)
   http-processes = 8
   ; add a bunch of uwsgi nodes to relay traffic to
   http-to = 192.168.173.10:3031
   http-to = 192.168.173.11:3031
   http-to = 192.168.173.12:3031
   ; add stats
   stats = 127.0.0.1:5001
   
   ; spawn the cache-udp-server
   cache-udp-server = 192.168.173.1:7171
   ; propagate updates to the other nodes
   cache-udp-node = 192.168.173.2:7171
   cache-udp-node = 192.168.173.3:7171

以及其他两个……

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache = 20000
   ; 4k per object is enough for SSL sessions
   cache-blocksize = 4096
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = true
   ; set SSL session timeout (in seconds)
   ssl-sessions-timeout = 300
   ; set the session context string (see later)
   https-session-context = foobar
   ; spawn an HTTPS router
   https = 192.168.173.1:8443,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same session cache)
   http-processes = 8
   ; add a bunch of uwsgi nodes to relay traffic to
   http-to = 192.168.173.10:3031
   http-to = 192.168.173.11:3031
   http-to = 192.168.173.12:3031
   ; add stats
   stats = 127.0.0.1:5001
   
   ; spawn the cache-udp-server
   cache-udp-server = 192.168.173.2:7171
   ; propagate updates to the other nodes
   cache-udp-node = 192.168.173.1:7171
   cache-udp-node = 192.168.173.3:7171

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache = 20000
   ; 4k per object is enough for SSL sessions
   cache-blocksize = 4096
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = true
   ; set SSL session timeout (in seconds)
   ssl-sessions-timeout = 300
   ; set the session context string (see later)
   https-session-context = foobar
   ; spawn an HTTPS router
   https = 192.168.173.1:8443,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same session cache)
   http-processes = 8
   ; add a bunch of uwsgi nodes to relay traffic to
   http-to = 192.168.173.10:3031
   http-to = 192.168.173.11:3031
   http-to = 192.168.173.12:3031
   ; add stats
   stats = 127.0.0.1:5001
   
   ; spawn the cache-udp-server
   cache-udp-server = 192.168.173.3:7171
   ; propagate updates to the other nodes
   cache-udp-node = 192.168.173.1:7171
   cache-udp-node = 192.168.173.2:7171


开始访问Rawrouter (记得使用一个支持持久化SSL会话的客户端，例如你的浏览器)，并从每个HTTPS终止器节点的统计数据服务器获得缓存统计信息。如果"hits"的次数比"miss"值高得多，那么系统工作良好，并且你的负载被分布，并处于超级高的性能模式。

所以，你问，什么是 ``https-session-context`` 呢？基本上，每个SSL会话在使用之前都会经过一个固定的字符串（会话上下文）检查。如果该会话不匹配那个字符串，那么它会被拒绝。默认情况下，会话上下文会被初始化为一个从HTTP服务器地址构建的值。将其强制为一个共享值将会避免一个节点中创建的一个会话被另一个节点拒绝。

使用命名缓存
******************

从uWSGI 1.9起，你可以有多个缓存。这是一个有2个节点，使用名为"ssl"的新一代缓存的设置。

``cache2`` 选项也允许设置一个自定义的键大小。由于SSL会话键并非非常长，因此我们可以用它来优化内存使用。在这个例子中，我们使用128字节键大小限制，这对于会话ID而言应该够了。

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache2 = name=ssl,items=20000,keysize=128,blocksize=4096,node=127.0.0.1:4242,udp=127.0.0.1:4141
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = ssl
   ; set sessions timeout (in seconds)
   ssl-sessions-timeout = 300
   ; set the session context string
   https-session-context = foobar
   ; spawn an HTTPS router
   https = :8443,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same session cache)
   http-processes = 8
   module = werkzeug.testapp:test_app
   ; add stats
   stats = :5001

以及第二个节点……

.. code-block:: ini

   [uwsgi]
   ; spawn the master process (it will run the cache sweeper thread)
   master = true
   ; store up to 20k sessions
   cache2 = name=ssl,items=20000,blocksize=4096,node=127.0.0.1:4141,udp=127.0.0.1:4242
   ; force the SSL subsystem to use the uWSGI cache as session storage
   ssl-sessions-use-cache = ssl
   ; set session timeout
   ssl-sessions-timeout = 300
   ; set the session context string
   https-session-context = foobar
   ; spawn an HTTPS router
   https = :8444,foobar.crt,foobar.key
   ; spawn 8 processes for the HTTPS router (all sharing the same sessions cache)
   http-processes = 8
   module = werkzeug.testapp:test_app
   ; add stats
   stats = :5002

注意事项
*********

如果你不想要手工配置缓存UDP节点，并且你的网络配置支持，那么你可以使用UDP多播。

.. code-block:: ini

   [uwsgi]
   ...
   cache-udp-server = 225.1.1.1:7171
   cache-udp-node = 225.1.1.1:7171

* 一个新的网关服务器正在开发中，它的名字是"udprepeater"。基本上，它将转发所有它接收的UDP包到订阅的后端节点。它将允许你维护订阅系统的零配置风格 (基本上，你只需要配置单个指向repeater的缓存UDP节点)。
* 目前，缓存节点之间并没有安全性。对于一些用户来说，这或许是一个巨大的问题，所以一个安全模式 (加密包) 正在开发中。
