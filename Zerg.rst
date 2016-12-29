Zerg模式
=========

.. note::

  是哒，那就像“重质不重量”的星际争霸赛中的Zerg。如果你还没有玩过星际，那么做好准备，你会看到一些废话。

  .. note::

    请注意，这些废话大多仅限于命名。Zerg模式是一个严肃的事。

当你的站点负载时可变的时候，能够动态添加worker将会是不错的

显然，你可以编辑你的配置，增加 ``workers`` ，然后重载你的uWSGI实例，但是对于负载很多的应用而言，这是不可取的，并且老实说 —— 谁会想要做那样的手工活来扩展应用呢？

启用Zerg模式，你可以允许"uwsgi-zerg"实例附加到你已经运行的服务器上，然后在工作中帮助它。

Zerg模式显然只能用于本地。你不能用它来添加远程实例 —— 这个工作已经由 :doc:`Fastrouter` ， :doc:`HTTP plugin<HTTP>` 或者你的web服务器的负载均衡器更好地完成了。

启用zerg服务器
------------------------

如果你想要zerg冲进一个uWSGI实例，那么必须启用Zerg服务器。它将会绑定到一个UNIX socket上，并且会传递uwsgi socket文件描述符到连接到它上面的Zerg worker。

.. warning:: socket必须是一个UNIX socket，因为它必须能够通过文件描述符传递。一个TCP socket就是不能用。

出于安全，UNIX socket并不会继承 ``chmod-socket`` 选项，当将总会使用当前的umask。

如果你有文件系统权限问题，那么在Linux上，通过附加一个 ``@`` 到socket名前面，你可以使用抽象名字空间中的UNIX socket。

* 一个正常的UNIX socket：

  .. code-block:: sh

    ./uwsgi -M -p 8 --module welcome --zerg-server /var/run/mutalisk

* Linux抽象名字空间内的socket：

  .. code-block:: sh

    ./uwsgi -M -p 8 --module welcome --zerg-server @nydus


附加zerg到zerg服务器
----------------------------------

要添加一个新的实例到你的zerg池中，执行使用--zerg选项

.. code-block:: sh

  ./uwsgi --zerg /var/run/mutalisk --master --processes 4 --module welcome
  # (or --zerg @nydus, following the example above)

通过这种方式，4个新的worker将会开始为请求提供服务。

当你的负载回到正常值的时候，你可以简单关闭所有的uwsgi-zerg实例，不会有任何问题。

你可以附加无限数量的uwsgi-zerg实例。

在zerg服务器不可用的时候回退
------------------------------------------

默认情况下，如果Zerg服务器不能用的话，一个Zerg客户端将不会运行。因此，如果你的zerg服务器死掉了，并且重载zerg客户端，那么这个客户端将会简单关闭。

如果你想要避免这种行为，那么添加一个 ``--socket`` 指令，映射到所需的socket (那个应该由zerg服务器管理的)，并且添加 ``--zerg-fallback`` 选项。

通过这个设置，如果一个Zerg服务器不可用，那么Zerg客户端将会继续正常绑定到指定的socket。

.. TODO: This needs to be documented better. An example would rock.

将Zerg当成测试器使用
---------------------

你可以用的一个不错的技巧是，用 ``SIGTSTP`` 信号挂起主要的实例，然后在Zerg中加载你的应用的一个新的版本。如果代码不能用，那么你可以简单关闭掉这个Zerg，然后恢复主要的实例。

Zerg池
----------

Zerg池是特殊的Zerg服务器，只对Zerg客户端提供服务，仅此而已。

你可以用它们来构建高可用性的系统，减少测试/重载期间的挂机时间。

你可以运行无限数量的zerg池 (在几个UNIX socket上)，并且将无限数量的socket映射到它们上。

.. code-block:: ini

  [uwsgi]
  master = true
  zergpool = /tmp/zergpool_1:127.0.0.1:3031,127.0.0.1:3032
  zergpool = /tmp/zergpool_2:192.168.173.22:3031,192.168.173.22:3032

使用一个像这样的配置，你会拥有两个zerg池，每个为两个socket服务。

现在，你可以把实例附加到它们上了。

.. code-block:: sh

  uwsgi --zerg /tmp/zergpool_1 --wsgi-file myapp.wsgi --master --processes 8
  uwsgi --zerg /tmp/zergpool_2 --rails /var/www/myapp --master --processes 4

或者你可以附加一个单一的实例到多个Zerg服务器上。

.. code-block:: sh

  uwsgi --zerg /tmp/zergpool_1 --zerg /tmp/zergpool_2 --wsgi-file myapp.wsgi --master --processes 8