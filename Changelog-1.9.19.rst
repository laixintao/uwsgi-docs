uWSGI 1.9.19
============

更改日志 [20131109]

此版本为uWSGI 2.0 (定于2013年12月底)启动“固化”周期。

度量子系统是缺失的最后一块，而这个版本 (经过1年的分析之后) 最终包含了它。

在接下来的2个月内，我们将开始弃用那些不感冒的、已知不能用的或者就被更现代/高级的所取代的特性或插件。

目前，计划删除下列插件和功能：

- Go插件，由gccgo取代。 (如果fork()支持发生了某些改变，那么最终将会恢复这个Go插件)
- 自动快照，从未被公开过，它具有大量的极端情况错误，并且非常复杂。 :doc:`MasterFifo` 添加的特性允许对自动快照更好的实现。

待定：

- erlang插件超级老，设计糟糕，应该被完全重写。如果你是其用户之一，那么请求联系工作人员。非常有可能我们会在没有赞助的情况下不能对其进行维护。
- matheval支持很快将会被移除 (除非我们发现了某些需要它的特定使用)，用选项解析器中直接实现的某种形式的简单运算来取代它
- admin插件应被某些更高级的所取代。一个用于定义动态选项的API正在进行中

错误修复
********

- 在非root情况下，完全跳过cgroups初始化
- 由Riccardo Magliocchetti负责的大量post静态分析修复
- 修复greenlet插件引用计数
- 避免统计信息推送器线程的kevent风暴
- 修复rbtimers计算
- 'cache'和'file'路由器都有一个'no_content_length'关键选项来避免设置Content-Length头
- PyPy插件自动启用线程/GIL
- 管理HTTP解析器中的dot_segments
- 改进srand()使用

新特性
********

度量子系统
^^^^^^^^^^^^^^^^^^^^^

这是uWSGI 2.0之前缺失的最后一块。度量子系统允许你存储与监控、绘图和质量检测相关的“数字”，并以多种方式导出。

官方文档： :doc:`Metrics`

Tornado循环引擎
^^^^^^^^^^^^^^^^^^^^^^^

在进行nodejs集成的时候，我们意识到，与我们过去相信的相反，Tornado (Python中一个异步、基于回调的模块) 在uWSGI中是可用的。

注意：该插件默认不内置

官方文档： :doc:`Tornado`

'puwsgi'协议
^^^^^^^^^^^^^^^^^^^^^

添加了'uwsgi'解析器的一个"持久化" (keep-alive)版本，名字为'puwsgi' (持久化uwsgi)。

这个协议只对没有请求体的请求有用，并且需要来自前端的支持。它目前用于自定义客户端/应用上，没有web服务器处理程序支持它。

``--puwsgi-socket <addr>`` 将绑定一个puwsgi socket到指定的地址上

--vassal-set
^^^^^^^^^^^^

你可以使用--set，告诉Emperor传递指定选项给每个vassal：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   vassal-set = processes=8
   vassal-set = enable-metrics=1
   
这会添加 ``--set processes=8`` 和 ``--set enable-metrics=1`` 到每个vassal


'template'转换
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个允许你应用所有的内部路由模式到你的响应上的转换。

以下面的文件为例 (foo.html)

.. code-block:: html

   <html>
     <head>
       <title>Running on ${SERVER_NAME}</title>
     </head>
     <body>
       Your ip address is: ${REMOTE_ADDR}<br/>
       Served requests: ${metric[worker.0.requests]}<br/>
       Pid: ${uwsgi[pid]}<br/>
       A random UUID: ${uwsgi[uuid]}
     </body>
   </html>
   
我们将在它上面应用'template'转换：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; enable the metrics subsystem
   enable-metrics = true
   ; inject the route transformation
   route-run = template:
   ; return a file (transformation will be applied to it)
   route-run = file:filename=foo.html,no_content_length=1
   
内部路由子系统中任何可用的东西都能用在'template'转换上。

性能是不错的，因此，你或许想要试着使用它，来代替老式的服务器端包含（Server Side Includes）。

不够？将其与缓存结合在一起：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; enable the metrics subsystem
   enable-metrics = true
   ; load foo.html in the cache
   cache2 = name=mycache,items=10
   load-file-in-cache = foo.html
   ; inject the route transformation
   route-run = template:
   ; return the cache item (transformation will be applied to it)
   route-run = cache:key=foo.html,no_content_length=1
   
再来？

加上分块编码？

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; enable the metrics subsystem
   enable-metrics = true
   ; load foo.html in the cache
   cache2 = name=mycache,items=10
   load-file-in-cache = foo.html
   ; inject the route transformation
   route-run = template:
   ; inject chunked encoding
   route-run = chunked:
   ; return the cache item (transformation will be applied to it)
   route-run = cache:key=foo.html,no_content_length=1

或者gzip ?

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; enable the metrics subsystem
   enable-metrics = true
   ; load foo.html in the cache
   cache2 = name=mycache,items=10
   load-file-in-cache = foo.html
   ; inject the route transformation
   route-run = template:
   ; inject gzip
   route-run = gzip:
   ; return the cache item (transformation will be applied to it)
   route-run = cache:key=foo.html,no_content_length=1

可用性
************

uWSGI 1.9.19于2013年11月09日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.19.tar.gz
