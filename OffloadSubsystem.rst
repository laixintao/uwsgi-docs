uWSGI 卸载(offloading)子系统
==============================

卸载是一种优化小任务的方式，将它们委托给一个或多个线程。

这些线程在非阻塞/事件触发的方式中运行这样的任务，允许大量的并发。

uWSGI栈的各种组件是卸载友好型的，而长期目标是允许应用代码随意使用它们。


要启动卸载子系统，仅需添加--offload-threads <n>，其中<n>是要生成的线程数 (每个worker)。
它们是原生线程，无锁（无共享资源），无惊群效应（到该系统的请求会进行轮询），并且它们是任意使用你的CPU核心的最佳方式。

统计信息子系统的"offloaded_requests"度量中计算了卸载请求的数量。


卸载静态文件
***********************

第一个卸载感知的组件是静态文件服务系统。

当卸载线程可用的时候，文件的整个传输会被委托给其中一个线程，立即释放你的worker (这样它就准备好接收新的请求了)

例如：

.. code-block:: ini

   [uwsgi]
   socket = :3031
   check-static = /var/www
   offload-threads = 4

卸载内部路由
***************************

router_uwsgi和router_http插件是卸载友好型的。

你可以路由请求到外部uwsgi/HTTP服务器，而无需担忧在响应生成期间阻塞了worker。

例如：

.. code-block:: ini

   [uwsgi]
   socket = :3031
   offload-threads = 8
   route = ^/foo http:127.0.0.1:8080
   route = ^/bar http:127.0.0.1:8181
   route = ^/node http:127.0.0.1:9090

自1.9.11起， ``cache`` 路由器也是卸载友好型的了。

.. code-block:: ini

   [uwsgi]
   socket = :3031
   offload-threads = 8
   route-run = cache:key=${REQUEST_URI}

一旦从缓存检索到该对象，就会在其中一个卸载线程中对其进行传输。

未来
**********

卸载子系统具有很大的潜力，你可以把它想成一个软件DMA：编程它，然后让它单打独斗。

目前，它是相当整体的，但想法是允许更复杂的插件 (一个redis相关的正在进行中)。

下一步是运行用户通过uwsgi api来对它“进行编程”。

