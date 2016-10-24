原生HTTP支持
===================


.. toctree::
   :maxdepth: 1

   HTTPS

HTTP socket
------------

``http-socket <bind>`` 选项将会让uWSGI和原生HTTP通信。如果你的web服务器不支持 :doc:`uwsgi protocol <Protocol>` ，但是可以与上游HTTP代理通信，或者如果你正在用诸如Webfaction或者Heroku这样的服务来托管你的应用，那么你可以使用 ``http-socket`` 。如果你计划只通过uWSGI开放你的应用，那么用 ``http`` 选项来代替，因为路由器/代理/负载均衡器将会保护你。

uWSGI HTTP/HTTPS路由器
---------------------------

uWSGI包含一个HTTP/HTTPS路由器/代理/负载均衡器，它能转发请求到uWSGI worker。可以以两种方式使用服务器：嵌入或独立使用。在嵌入模式下，它会自动生成worker，并安装通信socket。在独立使用模式下，你必须指定要连接的uwsgi socket地址。

嵌入模式::

  ./uwsgi --http 127.0.0.1:8080 --master --module mywsgiapp --processes 4

这将会生成一个监听8080端口的HTTP服务器，它会转发请求到由master进程管理的4个uWSGI worker组成的池中。

独立使用模式::

  ./uwsgi --master --http 127.0.0.1:8080 --http-to /tmp/uwsgi.sock

这将会生成一个HTTP路由器（出于安全起见，由master管理），它将转发请求到uwsgi socket ``/tmp/uwsgi.sock`` 。你可以绑定到多个地址/端口。

.. code-block:: ini

  [uwsgi]
  
  http = 0.0.0.0:8080
  http = 192.168.173.17:8181
  http = 127.0.0.1:9090
  
  master = true
  
  http-to = /tmp/uwsgi.sock

以及，到多个节点的负载均衡：

.. code-block:: ini

  [uwsgi]
  
  http = 0.0.0.0:8080
  http = 192.168.173.17:8181
  http = 127.0.0.1:9090
  
  master = true
  
  http-to = /tmp/uwsgi.sock
  http-to = 192.168.173.1:3031
  http-to = 192.168.173.2:3031
  http-to = 192.168.173.3:3031


* If you want to go massive (virtualhosting and zero-conf scaling) combine the
  HTTP router with the :doc:`SubscriptionServer`.
* 通过  ``http-var KEY=VALUE`` 选项，你可以让HTTP服务器传递自定义的uwsgi变量给worker。
* 你可以用 ``http-modifier1`` 选项来传递一个自定义的 `modifier1` 值给worker。

HTTPS支持
-------------

see :doc:`HTTPS`

HTTP Keep-Alive
---------------

如果后端设置了正确的HTTP头，那么你可以使用 ``http-keepalive`` 选项。要么后端将需要在每个响应中设置有效的 ``Content-Length`` ，要么你可以通过 ``http-auto-chunked`` 使用块编码。简单设置"Connection: close"并 *不够* 。还要记住在响应中设置"Connection: Keep-Alive"。你可以通过使用 ``add-header = Connection: Keep-Alive`` 选项来自动设置。

自uWSGI 2.1 (master分支)起，你可以使用 ``http11-socket`` 选项。 ``http11-socket`` 可替换 ``add-header`` 和 ``http-keepalive`` 选项 (但它并不像 ``so-keepalive`` 那样会接触到tcp相关的东东)。一旦设置了，如果遵循一堆规则，那么服务器将会试着保持连接打开。这并不是一个智能http 1.1解析器 (避免解析整个响应)，但它假设开发者生成正确的头部。已添加 ``http11-socket`` 来支持用于视频流的RTSP协议。

HTTP自动gzip
-------------

使用 ``http-auto-gzip`` 选项，如果uWSGI-encoding头设置为gzip，并且未设置 ``Content-Length`` 和 ``Content-Encoding`` ，那么uWSGI可以自动的压缩(gzip)内容。

我可以在生产上使用uWSGI的HTTP功能吗？
--------------------------------------------------

如果你需要一个负载均衡器/代理，那么这会是一个非常棒的想法。它将会自动查找新的uWSGI实例，并且以多种方式进行负载均衡。如果你想将其当成一个真正的web服务器使用，那么你应该考虑到，在uWSGI实例中提供静态文件是可能的，但不如使用一个专用的全功能web服务器那么好用。如果你把静态资产托管到云或者CDN上，使用uWSGI的HTTP功能，你绝对可以避免配置一个完整的web服务器。

.. note:: If you use Amazon's ELB (Elastic Load Balancer) in HTTP mode in
   front of uWSGI in HTTP mode, either a valid ``Content-Length`` *must be set*
   by the backend, or chunked encoding must be used, e.g., with
   ``http-auto-chunked``. The ELB "health test" may still fail in HTTP mode
   regardless, in which case a TCP health test can be used instead.

.. note:: 特别是，默认情况下，Django后端并不设置 ``Content-Length`` ，而其他会。如果在ELB后运行，那么要么使用如上的块解码，要么通过"Conditional GET" Django中间件，强制Django指定 ``Content-Length`` 。