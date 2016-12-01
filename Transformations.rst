uWSGI转换
=====================

从uWSGI 1.9.7起，一个“转换”API已被添加到 :doc:`InternalRouting` 。

一个转换就像是一个应用到由你的应用生成的响应上的过滤器。

转换可以是链式的 (一个转换的输出将会是接下来的转换的输入) ，并且可以完全覆盖响应头。

转换最常见的例子是gzip编码。你的应用的输出被传递到一个用gzip压缩它并且设置Content-Encoding头的函数中。这个特性依赖2个外部包：libpcre3-dev, Ubuntu上的libz-dev。

.. code-block:: ini

   [uwsgi]
   plugin = python,transformation_gzip
   http-socket = :9090
   ; load the werkzeug test app
   module = werkzeug.testapp:test_app
   ; if the client supports gzip encoding goto to the gzipper
   route-if = contains:${HTTP_ACCEPT_ENCODING};gzip goto:mygzipper
   route-run = last:

   route-label = mygzipper
   ; pass the response to the gzip transformation
   route = ^/$ gzip:

这个 ``cachestore`` 路由指令也是一个转换，因此，你可以缓存响应的各种状态。

.. code-block:: ini

   [uwsgi]
   plugin = python,transformation_gzip
   http-socket = :9090
   ; load the werkezeug test app
   module = werkzeug.testapp:test_app
   ; create a cache of 100 items
   cache = 100
   ; if the client support gzip encoding goto to the gzipper
   route-if = contains:${HTTP_ACCEPT_ENCODING};gzip goto:mygzipper
   route = ^/$ cache:key=werkzeug_homepage
   route = ^/$ cachestore:key=werkzeug_homepage
   route-run = last:

   route-label = mygzipper
   route = ^/$ cache:key=werkzeug_homepage.gz
   ; first cache the 'clean' response (for client not supporting gzip)
   route = ^/$ cachestore:key=werkzeug_homepage
   ; then pass the response to the gzip transformation
   route = ^/$ gzip:
   ; and cache it again in another item (gzipped)
   route = ^/$ cachestore:key=werkzeug_homepage.gz

另一个常见的转换是将样式表应用到XML文件上。 (见 :doc:`XSLT`)

 ``toxslt`` 转换由 ``xslt`` 插件公开：

.. code-block:: sh

   uwsgi --plugin xslt --http-socket :9090 -w mycd --route-run "toxslt:stylesheet=t/xslt/cd.xml.xslt,params=foobar=test&agent=\${HTTP_USER_AGENT}"

这里的 ``mycd`` 模块是一个简单的XML生成器。它的输出稍后被传递给XSLT转换。

流与缓冲
***********************

每个转换宣告自己是一个“流”，还是一个“缓冲”。

流是可以应用到响应块(部分)的转换。一个流转换的例子是gzip (要开始压缩它，你无需完整的body)。缓冲转换是那些在应用某些操作之前需要完整的body的转换。XSLT就是一个缓冲转换的例子。另一个缓冲转换的例子是那些用于将响应存储在某些缓存中的操作。

如果你整个管道是只由“流”转换组成的，那么你的客户端将会按块接收输出。另一方面，单个缓冲转换会让整个管道缓冲，因此，你的客户端只会在最后获得输出。

常用的流功能是gzip + chunked：

.. code-block:: ini

   [uwsgi]
   plugins = transformation_gzip,transformation_chunked
   route-run = gzip:
   route-run = chunked:
   ...

整个转换管道是由流插件组成的，英寸你将会实时获得每个HTTP块。

flush魔法
**************

“flush”转换是特别的。它让你发送当前转换缓冲内容到客户端 (无需清理缓冲)。

你可以用它在应用缓冲的时候实现流模式。常用的例子是流+缓存：

.. code-block:: ini

   [uwsgi]
   plugins = transformation_toupper,transform_tofile
   ; convert each char to uppercase
   route-run = toupper:
   ; after each chunk converted to upper case, flush to the client
   route-run = flush:
   ; buffer the whole response in memory for finally storing it in a file
   route-run = tofile:filename=/tmp/mycache
   ...

你可以多次以及在链的不同部分调用flush。试验一下吧...

可用转换 (最新更新 20130504)
************************************************

* ``gzip``, 由 ``transformation_gzip`` 插件公开 (编码响应缓冲到gzip)
* ``toupper``, 由 ``transformation_toupper`` 插件公开 (样例插件转换每个字符为大写)
* ``tofile``, 由 ``transformation_tofile`` 插件公开 (用来将响应缓冲缓存到一个静态文件中)
* ``toxslt``, 由 ``xslt`` 插件公开 (应用xslt样式表到一个XML响应缓冲)
* ``cachestore``, 由 ``router_cache`` 插件公开 (缓存响应缓冲到uWSGI缓存中)
* ``chunked``, 以HTTP chunked编码输出
* ``flush``, 刷新当前缓冲到客户端
* ``memcachedstore``, 将响应缓冲存储到一个memcached对象上
* ``redisstore``, 将响应缓冲存储到一个redis对象上
* ``template``, 应用路由转换到每个块上

正在进行中
**********

* ``rpc``, 允许应用rpc函数到一个响应缓冲 (限制64k大小)
* ``lua``, 应用一个lua函数到一个响应缓冲 (无大小限制)

