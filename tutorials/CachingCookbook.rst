uWSGI缓存烹饪指南
==========================

这是使用 :doc:`../InternalRouting`, :doc:`../Caching` 和 :doc:`../Transformations` 的各种缓存技术的烹饪指南。

例子假设使用模块化uWSGI构建。如果你正使用单片构建，那么你可以忽略'plugins'选项。

菜谱是在uWSGI 1.9.7之上测试的。较老的版本可能不能用。

我们开始吧
***********

这是一个简单的perl/PSGI Dancer应用，我们部署在一个http-socket上，使用4个进程。

.. code-block:: pl

   use Dancer;

   get '/' => sub {
           "Hello World!"
   };

   dance;

这是uWSGI配置。注意log-micros指令。uWSGI内存中缓存的目标是在不到1毫秒的时间内生成一个响应 (是哒，这是真的)，所以我们想让响应时间以微秒记录 (一毫秒的千分之一)。

.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true


在你的终端中运行uWSGI实例，然后对其进行一堆请求。

.. code-block:: sh

   curl -D /dev/stdout http://localhost:9090/

如果一切顺利，你应该会在你的uWSGI日志中看到类似的东东：

.. code-block:: sh

   [pid: 26586|app: 0|req: 1/1] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:06:58 2013] GET / => generated 12 bytes in 3497 micros (HTTP/1.1 200) 4 headers in 126 bytes (0 switches on core 0)
   [pid: 26586|app: 0|req: 2/2] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:07:14 2013] GET / => generated 12 bytes in 1134 micros (HTTP/1.1 200) 4 headers in 126 bytes (0 switches on core 0)
   [pid: 26586|app: 0|req: 3/3] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:07:16 2013] GET / => generated 12 bytes in 1249 micros (HTTP/1.1 200) 4 headers in 126 bytes (0 switches on core 0)
   [pid: 26586|app: 0|req: 4/4] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:07:17 2013] GET / => generated 12 bytes in 953 micros (HTTP/1.1 200) 4 headers in 126 bytes (0 switches on core 0)
   [pid: 26586|app: 0|req: 5/5] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:07:18 2013] GET / => generated 12 bytes in 1016 micros (HTTP/1.1 200) 4 headers in 126 bytes (0 switches on core 0)


而cURL将会返回：

.. code-block:: txt

   HTTP/1.1 200 OK
   Server: Perl Dancer 1.3112
   Content-Length: 12
   Content-Type: text/html
   X-Powered-By: Perl Dancer 1.3112

   Hello World!

一个进程上的第一个请求花费大概3毫秒 (这是正常的，因为对于第一个请求，会执行大量的代码)，但接下来的请求则花费大概1毫秒。

现在，我们想要将响应存储在uWSGI缓存中。

第一份菜单
****************

我们首先创建一个uWSGI缓存，命名为'mycache'，它带有100个槽，每个槽64 KiB (新选项位于配置的尾部)，而对于每个对'/'的请求，我们在其中搜索一个名为'myhome'的特定的项。

这次，我们也加载 ``router_cache`` 插件 (虽然在单片服务器上，是默认内建它的)。


.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; at each request for / check for a 'myhome' item in the 'mycache' cache
   ; 'route' apply a regexp to the PATH_INFO request var
   route = ^/$ cache:key=myhome,name=mycache

重启uWSGI，然后用cURL重新运行前面的恶测试。忧伤的是，没有任何改变。为什么？

因为你并没有指示uWSGI去将插件响应存储到缓存中。你需要使用 ``cachestore`` 路由动作……


.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; at each request for / check for a 'myhome' item in the 'mycache' cache
   ; 'route' apply a regexp to the PATH_INFO request var
   route = ^/$ cache:key=myhome,name=mycache
   ; store each successful request (200 http status code) for '/' in the 'myhome' item
   route = ^/$ cachestore:key=myhome,name=mycache

现在，重新运行测试，而你应该看到请求降到大概100-300微秒到范围内。增益取决于多种因素，但是你应该在响应时间上获得至少60%的增益。

日志行报告-1作为app id：

.. code-block:: sh

   [pid: 26703|app: -1|req: -1/2] 192.168.173.14 () {24 vars in 327 bytes} [Wed Apr 17 09:24:52 2013] GET / => generated 12 bytes in 122 micros (HTTP/1.1 200) 2 headers in 64 bytes (0 switches on core 0)

这是因为，当从缓存提供响应的时候，并未碰到你的应用／插件 (在这种情况下，不涉及任何perl调用)。

你也会注意到更少的响应头：

.. code-block:: txt

   HTTP/1.1 200 OK
   Content-Type: text/html
   Content-Length: 12

   Hello World!

这是因为只有响应体会被缓存。默认情况下，生成的响应会被设置为text/html，但是可以改动它，或者让MIME类型引擎为你做这个工作 (见下文)。

将它们统统都缓存起来！！！
***********************

我们想要缓存我们所有的请求。它们中的一些返回图像和css，而其他一些则总是返回text/html


.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; load the mime types engine
   mime-file = /etc/mime.types

   ; at each request starting with /img check it in the cache (use mime types engine for the content type)
   route = ^/img/(.+) cache:key=/img/$1,name=mycache,mime=1

   ; at each request ending with .css check it in the cache
   route = \.css$ cache:key=${REQUEST_URI},name=mycache,content_type=text/css

   ; fallback to text/html all of the others request
   route = .* cache:key=${REQUEST_URI},name=mycache
   ; store each successful request (200 http status code) in the 'mycache' cache using the REQUEST_URI as key
   route = .* cachestore:key=${REQUEST_URI},name=mycache


多缓存
***************

你或许／需要存储项到不同的缓存中。我们可以修改前面的菜谱来为图像、css和html响应使用3个不同的缓存。

.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100

   ; create a cache for images with dynamic size (images can be big, so do not waste memory)
   cache2 = name=images,items=20,bitmap=1,blocks=100

   ; a cache for css (20k per-item is more than enough)
   cache2 = name=stylesheets,items=30,blocksize=20000

   ; load the mime types engine
   mime-file = /etc/mime.types

   ; at each request starting with /img check it in the 'images' cache (use mime types engine for the content type)
   route = ^/img/(.+) cache:key=/img/$1,name=images,mime=1

   ; at each request ending with .css check it in the 'stylesheets' cache
   route = \.css$ cache:key=${REQUEST_URI},name=stylesheets,content_type=text/css

   ; fallback to text/html all of the others request
   route = .* cache:key=${REQUEST_URI},name=mycache

   ; store each successful request (200 http status code) in the 'mycache' cache using the REQUEST_URI as key
   route = .* cachestore:key=${REQUEST_URI},name=mycache
   ; store images and stylesheets in the corresponding caches
   route = ^/img/ cachestore:key=${REQUEST_URI},name=images
   route = ^/css/ cachestore:key=${REQUEST_URI},name=stylesheets


重要的是，每个匹配上的'cachestore'将会重写前一个。因此，我们添加.*作为第一条规则。

更激进点，Expires HTTP头部
**********************************************

你可以为每个缓存项设置过期时间。如果一个项拥有一个过期时间，那么它将会被转换成HTTP Expires头部。这意味着，一旦你发送了一个缓存项给浏览器，它将不会再请求这个项，直到过期！

我们使用前一个菜谱，简单添加不同的过期时间给项。


.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100

   ; create a cache for images with dynamic size (images can be big, so do not waste memory)
   cache2 = name=images,items=20,bitmap=1,blocks=100

   ; a cache for css (20k per-item is more than enough)
   cache2 = name=stylesheets,items=30,blocksize=20000

   ; load the mime types engine
   mime-file = /etc/mime.types

   ; at each request starting with /img check it in the 'images' cache (use mime types engine for the content type)
   route = ^/img/(.+) cache:key=/img/$1,name=images,mime=1

   ; at each request ending with .css check it in the 'stylesheets' cache
   route = \.css$ cache:key=${REQUEST_URI},name=stylesheets,content_type=text/css

   ; fallback to text/html all of the others request
   route = .* cache:key=${REQUEST_URI},name=mycache

   ; store each successful request (200 http status code) in the 'mycache' cache using the REQUEST_URI as key
   route = .* cachestore:key=${REQUEST_URI},name=mycache,expires=60
   ; store images and stylesheets in the corresponding caches
   route = ^/img/ cachestore:key=${REQUEST_URI},name=images,expires=3600
   route = ^/css/ cachestore:key=${REQUEST_URI},name=stylesheets,expires=3600

图像和样式表会被缓存1个小时，而html响应会被缓存1分钟

监控缓存
*****************

统计信息服务器公开了缓存信息。

有一个使用那个信息的基于ncurses的工具 (https://pypi.python.org/pypi/uwsgicachetop)。


存储一个对象的GZIP变体
*********************************

回到第一个菜谱。我们或许想用存储一个响应的两份拷贝。"干净的"一份，和gzip压缩过的一个，后者用于支持gzip编码的客户端。

要启用gzip拷贝，你只需要为这个项选择一个名字，然后将其当成cachestore动作的‘gzip’选项传递。

然后检查HTTP_ACCEPT_ENCODING请求头部。如果它包含'gzip'一词，那么你可以发送给它gzip变体。

.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; if the client support GZIP give it the gzip body
   route-if = contains:${HTTP_ACCEPT_ENCODING};gzip cache:key=gzipped_myhome,name=mycache,content_encoding=gzip
   ; else give it the clear version
   route = ^/$ cache:key=myhome,name=mycache

   ; store each successful request (200 http status code) for '/' in the 'myhome' item in gzip too
   route = ^/$ cachestore:key=myhome,gzip=gzipped_myhome,name=mycache


将静态文件存储到缓存中，以便提供快速服务
**************************************************

你可以在服务器启动的时候用静态文件填充uWSGI缓存，以便提供快速服务。选项--load-file-in-cache是对于这项工作的正确工具

.. code-block:: ini

   [uwsgi]
   plugins = 0:notfound,router_cache
   http-socket = :9090
   cache2 = name=files,bitmap=1,items=1000,blocksize=10000,blocks=2000
   load-file-in-cache = files /usr/share/doc/socat/index.html
   route-run = cache:key=${REQUEST_URI},name=files

你可以指定所有你需要的--load-file-in-cache指令，但是一个更好的方法可能是

.. code-block:: ini

   [uwsgi]
   plugins = router_cache
   http-socket = :9090
   cache2 = name=files,bitmap=1,items=1000,blocksize=10000,blocks=2000
   for-glob = /usr/share/doc/socat/*.html
      load-file-in-cache = files %(_)
   endfor =
   route-run = cache:key=${REQUEST_URI},name=files

这会存储所有的html文件到/usr/share/doc/socat中。

存储项，并把路径当成键。

当请求一个不存在的项时，会关闭连接，而你应该会得到一个丑陋的

.. code-block:: sh

   -- unavailable modifier requested: 0 --


这是因为内部路由系统无法管理请求，并且没有能够管理这个请求的请求插件。

你可以使用简单的“notfound”插件（它将总是返回404）来构建一个更好的基础设施

.. code-block:: ini

   [uwsgi]
   plugins = 0:notfound,router_cache
   http-socket = :9090
   cache2 = name=files,bitmap=1,items=1000,blocksize=10000,blocks=2000
   for-glob = /usr/share/doc/socat/*.html
      load-file-in-cache = files %(_)
   endfor =
   route-run = cache:key=${REQUEST_URI},name=files


你也可以使用--load-file-in-cache-gzip来将文件作为gzip存储到缓存中

这个选项不允许设置缓存项到名字，因此，要带以及不带gzip支持来支持客户端，我们可以使用2个不同的缓存

.. code-block:: ini

   [uwsgi]
   plugins = 0:notfound,router_cache
   http-socket = :9090
   cache2 = name=files,bitmap=1,items=1000,blocksize=10000,blocks=2000
   cache2 = name=compressedfiles,bitmap=1,items=1000,blocksize=10000,blocks=2000
   for-glob = /usr/share/doc/socat/*.html
      load-file-in-cache = files %(_)
      load-file-in-cache-gzip = compressedfiles %(_)
   endfor =
   ; take the item from the compressed cache
   route-if = contains:${HTTP_ACCEPT_ENCODING};gzip cache:key=${REQUEST_URI},name=compressedfiles,content_encoding=gzip
   ; fallback to the uncompressed one
   route-run = cache:key=${REQUEST_URI},name=files

为已鉴权用户缓存
*******************************

如果你通过http basic鉴权来鉴权用户，那么你可以使用${REMOTE_USER}请求变量来为每个区分缓存：

.. code-block:: ini

   [uwsgi]
   ; load the PSGI plugin as the default one
   plugins = 0:psgi,router_cache
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; check if the user is authenticated
   route-if-not = empty:${REMOTE_USER} goto:cacheme
   route-run = break:

   ; the following rules are executed only if REMOTE_USER is defined
   route-label = cacheme
   route = ^/$ cache:key=myhome_for_${REMOTE_USER},name=mycache
   ; store each successful request (200 http status code) for '/'
   route = ^/$ cachestore:key=myhome_for_${REMOTE_USER},name=mycache


基于Cookie的鉴权一般更复杂，但是绝大多数的时间，session id会作为cookie传递。

你或许想要将这个session_id作为键使用

.. code-block:: ini

   [uwsgi]
   ; load the PHP plugin as the default one
   plugins = 0:php,router_cache
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; check if the user is authenticated
   route-if-not = empty:${cookie[PHPSESSID]} goto:cacheme
   route-run = break:

   ; the following rules are executed only if the PHPSESSID cookie is defined
   route-label = cacheme
   route = ^/$ cache:key=myhome_for_${cookie[PHPSESSID]},name=mycache
   ; store each successful request (200 http status code) for '/'
   route = ^/$ cachestore:key=myhome_for_${cookie[PHPSESSID]},name=mycache


显然，恶意用户可以构建一个假的session id，然后也许会装满你的缓存。你应该总是检查这个session id。没有单个（好）方法，但是对于基于文件的php会话的一个好例子是下面这个：

.. code-block:: ini

   [uwsgi]
   ; load the PHP plugin as the default one
   plugins = 0:php,router_cache
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; create a cache with 100 items (default size per-item is 64k)
   cache2 = name=mycache,items=100
   ; check if the user is authenticated
   route-if-not = empty:${cookie[PHPSESSID]} goto:cacheme
   route-run = break:

   ; the following rules are executed only if the PHPSESSID cookie is defined
   route-label = cacheme
   ; stop if the session file does not exist
   route-if-not = isfile:/var/lib/php5/sessions/sess_${cookie[PHPSESSID]} break:
   route = ^/$ cache:key=myhome_for_${cookie[PHPSESSID]},name=mycache
   ; store each successful request (200 http status code) for '/'
   route = ^/$ cachestore:key=myhome_for_${cookie[PHPSESSID]},name=mycache

缓存到文件
****************

有时候，你想要存储静态文件，而不是缓存在内存中。

transformation_tofile插件让你在文件中存储响应：

.. code-block:: ini

   [uwsgi]
   ; load the PHP plugin as the default one
   plugins = 0:psgi,transformation_tofile,router_static
   ; load the Dancer app
   psgi = myapp.pl
   ; enable the master process
   master = true
   ; spawn 4 processes
   processes = 4
   ; bind an http socket to port 9090
   http-socket = :9090
   ; log response time with microseconds resolution
   log-micros = true

   ; check if a file exists
   route-if = isfile:/var/www/cache/${hex[PATH_INFO]}.html static:/var/www/cache/${hex[PATH_INFO]}.html
   ; otherwise store the response in it
   route-run = tofile:/var/www/cache/${hex[PATH_INFO]}.html

hex[]路由变量接收一个请求变量内容，然后以十六进制对其进行编码。因为PATH_INFO倾向于包含/，因此它是比存储完整的路径名（或者使用其他编码方案，例如也可以包含斜杠的base64）更好的方法。
