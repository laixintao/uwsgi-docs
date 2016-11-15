uWSGI 1.9.11
============

更新日志[20130526]

错误修复
********

* 修复Python 3 stdout/stderr缓冲
* 修复mule消息 (``@mulefunc`` 现在是可靠的了)
* 修复在动态模式下的 ``SCRIPT_NAME`` 处理
* 修复gzip静态模式下的X-Sendfile
* 用自定义块大小修复缓存项最大大小
* 修复缓存路径处理

新特性
********

新的高性能PyPy插件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

关于作者：Maciej Fijalkowski

我们很高兴地宣布，新的PyPy插件可以用了。

PyPy团队给予了我们巨大的帮助。我们希望此uWSGI集成 (这给PyPy项目带来了新的挑战)将会帮助PyPy变得越来越好。

官方文档 :doc:`PyPy`

Cron改进
^^^^^^^^^^^^^^^^^

关于作者：Łukasz Mierzwa

唯一cron
------------

你现在可以避免重复cron了。uWSGI的master将会跟踪单个任务的结束，直到它结束了，才会触发相同的cron：

.. code-block:: ini

   [uwsgi]
   unique-cron = -1 -1 -1 -1 -1 my_script.sh

cron2语法
------------

--cron选项的一个键值对变体现已可用：

.. code-block:: ini

   [uwsgi]
   cron2 = minute=39,hour=23,month=-1,week=-1,day=-1,unique=1,legion=foobar,harakiri=30

harakiri cron
-------------

在使用 ``cron2`` 选项的时候，允许你为一个cron任务设置一个harakiri超时时间。仅需添加 ``harakiri=n`` 到选项中。

GNU Hurd支持
^^^^^^^^^^^^^^^^^^^^

Debian GNU/Hurd最近已经发布。可以在其上构建uWSGI 1.9.11，然而，已经挖槽的测试非常少。

内存卸载引擎
^^^^^^^^^^^^^^^^^^^^^^^^^

想法：Stefano Brentegani

当从缓存提供内容的时候，在从内存传输到socket期间可能会阻塞worker。

一个名为"memory"的新的卸载系统允许卸载内存传输。缓存路由器自动支持它。对更多领域的支持将会尽快添加。

要启用它，仅需添加 ``--offload-threads <n>``

新的Websockets聊天例子
^^^^^^^^^^^^^^^^^^^^^^^^^^^

已添加一个使用Redis的Websockets聊天例子到这个repo中：

https://github.com/unbit/uwsgi/blob/master/tests/websockets_chat.py

错误路由
^^^^^^^^^^^^

现在，你可以定义一个一旦在插件中设置HTTP状态码就会立即执行的路由表。

这让你可以完全修改响应。这对于自定义错误码很有用。

所有的路由标准选项都可用 (包括标签) ，加上一个优化的 ``error-route-status`` ，它匹配指定的HTTP状态码：

.. code-block:: ini

   [uwsgi]
   error-route-status = 502 redirect:http://unbit.it

支持wsgi.file_wrapper中的特殊情况使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一般情况下， ``wsgi.file_wrapper`` 可调用需要一个类文件对象。PEP 333/3333报告了一种当该对象不是一个文件时的特殊模式 (调用 ``read()`` ，直到消费完该对象)。uWSGI现在支持这种模式 (即使是以一种Hack方式)。

HTTP/HTTPS路由器keepalive改进
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

关于作者：André Cruz

在使用 ``--http-keepalive`` 的时候，你现在可以保持连接打开，甚至是在请求由请求体的情况下。


The harakiri路由动作
^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在，你可以使用内部路由，为每个请求设置一个harakiri计时器：

.. code-block:: ini

   [uwsgi]
   ; set harakiri to 30 seconds for request starting with /slow
   route = ^/slow harakiri:30

RPC封装
^^^^^^^^^^^^

已经扩展了RPC插件，以允许与其他标准进行相互操作。

目前，公开了一个简单的HTTP封装器和一个XML-RPC封装器。

该HTTP简单封装器通过解析 ``PATH_INFO`` 工作。

 ``/foo/bar/test`` 调用的结果将是

uwsgi.rpc('foo', 'bar', 'test')

要启用此HTTP模式，仅需设置 ``modifier2`` 为 '2':

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 173
   http-socket-modifier2 = 2
   ; load the rpc code
   import = myrpcfuncs.py
   
或者 (拥有更多的控制权)

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route-run = uwsgi:,173,2
   ; load the rpc code
   import = myrpcfuncs.py


XML-RPC封装器工作方式相同，但它使用的modifier2值为'3'。它要求uWSGI的构建启用了libxml2。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route-run = uwsgi:,173,3
   ; load the rpc code
   import = myrpcfuncs.py
   
然后只需调用它：

.. code-block:: python

   proxy = xmlrpclib.ServerProxy("http://localhost:9090')
   proxy.hello('foo','bar','test') 
   
你可以使用路由将多个封装器组合在一起。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; /xml force xmlrpc wrapper
   route = ^/xml uwsgi:,173,3
   ; fallback to HTTP simple
   route-if-not = startswith:${PATH_INFO};/xml uwsgi:,173,2
   ; load the rpc code
   import = myrpcfuncs.py


可用性
************
uWSGI 1.9.11于20130526发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.11.tar.gz
