uWSGI 1.9.4
===========

更新日志20130330

错误修复
********

修复统计信息子系统导出的缓存统计信息 (Łukasz Mierzwa)

修复after_request中的CoroEV错误 (Tom Molesworth和John Berthels)

更新从永久化存储中恢复之后的缓存项 (Łukasz Mierzwa)

修复非worker进程中的信号处理

修复多个Mule设置中的惊群效应(thundering herd)

移植C++骨架插件到新API

修复当uWSGI作为共享库构建时的重载

新特性
********

SmartOS官方支持
^^^^^^^^^^^^^^^^^^^^^^^^

从现在起，官方支持的操作系统包含了SmartOS

V8初始支持
^^^^^^^^^^^^^^^^^^

之前关于使用Lua编写uWSGI路由规则和配置的建议，提醒了大量的javascript用户，他们指出，javascript自身可能是一个有效的替代品。V8插件现已推出，支持RPC，信号处理器和配置。要构建它，你需要libv8头文件：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/v8

.. code-block:: js

   var config = {};
   config['socket'] = [':3031', ':3032', ':3033'];
   config['master'] = true;
   config['processes'] = 3+1;
   config['module'] = 'werkzeug.testapp:test_app';

   config;

.. code-block:: sh

   uwsgi --plugin v8 --config foo.js

前一个例子将让你用javascript编写动态配置，而你可以通过RPC子系统来导出javascript函数：

.. code-block:: js

   function part1(request_uri, remote_addr) {
        return '<h1>i am part1 for ' + request_uri + ' ' + remote_addr + "</h1>" ;
   }

   function part2(request_uri, remote_addr) {
        return '<h2>i am part2 for ' + request_uri + ' ' + remote_addr + "</h2>" ;
   }

   function part3(request_uri, remote_addr) {
        return '<h3>i am part3 for ' + request_uri + ' ' + remote_addr + "</h3>" ;
   }

   uwsgi_register_rpc('part1', part1);
   uwsgi_register_rpc('part2', part2);
   uwsgi_register_rpc('part3', part3);

.. code-block:: ini

   [uwsgi]
   plugin = v8
   v8-load = func.js
   cache2 = name=foobar,items=10

   http-socket = :9090

   route-run = addheader:Content-Type: text/html
   route-run = cache:key=pippo,name=foobar
   route-run = cachestore:key=pippo,name=foobar
   route-run = rpcnext:part1 ${REQUEST_URI} ${REMOTE_ADDR}
   route-run = rpcnext:part2 ${REQUEST_URI} ${REMOTE_ADDR}
   route-run = rpcnext:part3 ${REQUEST_URI} ${REMOTE_ADDR}
   route-run = break:

前一个例子从3个javascript函数中生成一个HTTP响应，让后将其存储在uWSGI缓存中。

对rpcnext感到好奇？

rpcnext路由动作
^^^^^^^^^^^^^^^^^^^^^^^^^^

我们已经可以调用路由子系统中的rpc函数来生成响应了。通过此rpcnext动作 (别名又为rpcblob)，你可以调用多个rpc函数，并且在一个单一的响应中组装返回值。

Legion改进
^^^^^^^^^^^^^^^^^^^

我们基本没对 :doc:`Legion` 进行加固。其目标是对于uWSGI 2.0，可以拥有一个坚如磐石的集群实现，让你甚至可以从你的应用使用它。

1.9.4中的代码已经被Łukasz Mierzwa做了一点重构，使得与外部插件的集成更容易些。

添加了一个新的"join"钩子，当一个节点变成legion的一个活跃的部分时会立即调用它 (也就是说，它是quorum的一部分)。

可用性
************

uWSGI 1.9.4已经在20130330发布，你可以从下面的地址中下载：

http://projects.unbit.it/downloads/uwsgi-1.9.4.tar.gz
