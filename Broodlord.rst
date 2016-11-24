利用Broodlord模式实现自动伸缩
================================

Broodlord (来自于Starcraft，就像 :doc:`Zerg` 模式) 是vassal向Emperor请求“加固”的方式。“加固”是按需生成新的vassal，一般会绑定到同一个socket上。单独的Broodlord模式并不是那么有用。但是，当与 :doc:`Zerg`, :doc:`Idle` 和 :doc:`Emperor` 组合在一起，它可以被用来实现你的应用的自动伸缩。

警告：如果你在寻找的是最红动态调整实例worker数的方式，那么看看 :doc:`Cheaper` 模式，Broodlord模式是用来生成完全新的实例。

一个“简单的”例子
------------------

我们将启动一个只有一个worker的应用，按需增加资源。Broodlord模式要求你的配置文件中要有一个额外的节，用于zerg。

.. code-block:: ini

  [uwsgi]
  socket = :3031
  master = true
  vassal-sos-backlog = 10
  module = werkzeug.testapp:test_app
  processes = 1
  zerg-server = /tmp/broodlord.sock
  disable-logging = true
  
  [zerg]
  zerg = /tmp/broodlord.sock
  master = true
  module = werkzeug.testapp:test_app
  processes = 1
  disable-logging = true
  idle = 30
  die-on-idle = true

``vassal-sos-backlog`` 选项 (仅适用于Linux和TCP socket) 将在监听队列高过给定的值的时候向Emperor请求zerg。默认情况，这个值是10.未来将会添加更多的"vassal-sos-"选项，从而允许更具体的过载检测系统。

``[zerg]`` 节是当一个vassal需要资源的时候，Emperor将运行的配置。 ``die-on-idle`` 选项将在zerg超过30秒不活跃的时候销毁它。这个配置显示如何结合不同的uWSGI特性来实现不同缩放手段。要运行这个
Emperor，我们需要指定可以运行多少个zerg实例：

.. code-block:: sh

  uwsgi --emperor /etc/vassals --emperor-broodlord 40

这将允许你为应用运行多达40个额外的zerg worker。

`--vassal-sos`
--------------

.. note::

   这个标识已于2.0.7添加。

`--vassal-sos` 允许vassal一旦其所有的worker都处于忙碌状态，立即请求加固。

这个选项接收一个整型值，请求一个新的加固之间等待的秒数。

手动请求加固
---------------------------------

你可以使用master FIFO的"B"命令来强制一个实例请求来自Emperor的加固。

.. code-block:: sh

   echo B > /var/run/master.fifo

钩子之下 (或：黑入broodlord模式)
--------------------------------------------

技术上，broodlord模式只是由vassal发送的一个简单的消息，用来“强制”Emperor生成另一个实例名中带':zerg'后缀的vassal。

即使后缀是':zerg'，这也不意味着你需要使用Zerg模式。一个'zerg'实例可以是一个简单订阅到一个路由器，或者绑定到一个SO_REUSEPORT socket的完全独立的实例。

这是一个使用订阅系统的例子。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:0
   subscribe2 = server=127.0.0.1:4040,key=foobar.it
   psgi = app.pl
   processes = 4
   vassal-sos = 3
   
   [zerg]
   socket = 127.0.0.1:0
   subscribe2 = server=127.0.0.1:4040,key=foobar.it
   psgi = app.pl
   idle = 60
   processes = 1
