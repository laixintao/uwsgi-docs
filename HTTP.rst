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
* You can make the HTTP server pass custom uwsgi variables to workers with the
  ``http-var KEY=VALUE`` option.
* You can use the ``http-modifier1`` option to pass a custom `modifier1` value
  to workers.

HTTPS支持
-------------

see :doc:`HTTPS`

HTTP Keep-Alive
---------------

If your backends set the correct HTTP headers, you can use the
``http-keepalive`` option.  Either your backends will need to set a valid
``Content-Length`` in each response, or you can use chunked encoding with
``http-auto-chunked``. Simply setting "Connection: close" is *not enough*.
Also remember to set "Connection: Keep-Alive" in your response. You can
automate that using the ``add-header = Connection: Keep-Alive`` option.

Since uWSGI 2.1 (master branch) you can use the ``http11-socket`` option.
``http11-socket`` may replace the ``add-header`` and ``http-keepalive`` options
(but it doesn't touch tcp stuff as so ``so-keepalive`` does).
Once set the server will try to maintain the connection opened if a bunch of
rules are respected. This is not a smart http 1.1 parser (to avoid parsing the
whole response) but assumes the developer is generating the right headers.
``http11-socket`` has been added to support RTSP protocol for video streaming.

HTTP自动gzip
-------------

With the ``http-auto-gzip`` option, uWSGI can automatically gzip content if the
uWSGI-encoding header is set to gzip while ``Content-Length`` and
``Content-Encoding`` are not set.

我可以在生产上使用uWSGI的HTTP能力吗？
--------------------------------------------------

If you need a load balancer/proxy it can be a very good idea. It will
automatically find new uWSGI instances and can load balance in various ways.
If you want to use it as a real webserver you should take into account that
serving static files in uWSGI instances is possible, but not as good as using a
dedicated full-featured web server.  If you host static assets in the cloud or
on a CDN, using uWSGI's HTTP capabilities you can definitely avoid configuring
a full webserver.

.. 注意:: If you use Amazon's ELB (Elastic Load Balancer) in HTTP mode in
   front of uWSGI in HTTP mode, either a valid ``Content-Length`` *must be set*
   by the backend, or chunked encoding must be used, e.g., with
   ``http-auto-chunked``. The ELB "health test" may still fail in HTTP mode
   regardless, in which case a TCP health test can be used instead.

.. 注意:: In particular, the Django backend does not set ``Content-Length`` by
   default, while most others do. If running behind ELB, either use chunked
   encoding as above, or force Django to specify ``Content-Length`` with the
   "Conditional GET" Django middleware.
