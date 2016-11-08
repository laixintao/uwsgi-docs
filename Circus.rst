使用Circus运行uWSGI实例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Circus (https://circus.readthedocs.io/en/0.7/)是一个用Python写的进程管理器。它非常类似于像Supervisor这样的工程，但具有一些额外特性。虽然大部分，主要的功能在uWSGI中都能找到对应的，但是Circus可以被当成一个库使用，让你可以构建动态配置 (并且扩展uWSGI模式)。这个部分是非常重要的，并且可能就是Circus的真正卖点。


Socket激活
*****************

基于古老的inetd模式，Circu可以绑定到socket，并且将其传递给它的孩子。

从一个简单的Circus配置开始 (称之为circus.ini):

.. code-block:: ini

   [circus]
   endpoint = tcp://127.0.0.1:5555
   pubsub_endpoint = tcp://127.0.0.1:5556
   stats_endpoint = tcp://127.0.0.1:5557

   [watcher:dummy]
   cmd = uwsgi --http-socket fd://$(circus.sockets.foo) --wsgi-file yourapp.wsgi
   use_sockets = True
   send_hup = True
   stop_signal = QUIT

   [socket:foo]
   host = 0.0.0.0
   port = 8888

这样运行它

.. code-block:: sh

    circusd circus.ini

(更好的) Socket激活
**************************

如果你想按需生成实例，那么你可能会想要在不再使用它们的时候将其关闭。要实现这点，则使用--idle uWSGI选项。

.. code-block:: ini

   [circus]
   check_delay = 5
   endpoint = tcp://127.0.0.1:5555
   pubsub_endpoint = tcp://127.0.0.1:5556
   stats_endpoint = tcp://127.0.0.1:5557

   [watcher:dummy]
   cmd = uwsgi --master --idle 60 --http-socket fd://$(circus.sockets.foo) --wsgi-file yourapp.wsgi
   use_sockets = True
   warmup_delay = 0
   send_hup = True
   stop_signal = QUIT

   [socket:foo]
   host = 0.0.0.0
   port = 8888

这一次，我们启用了master进程。它将会管理--idle选项，在实例不活跃时间超过60秒的时候关闭它。