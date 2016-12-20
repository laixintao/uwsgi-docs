按需(On demand)vassal (socket激活)
=====================================

受古老的xinetd/inetd方法启发，你可以只在第一个到指定socket的连接之后才生成你的vassal。这个特性自1.9.1起可用。与–idle/–die-on-idle选项组合，你可以拥有真正按需应用。

当为特定的vassal激活按需特性的时候，emperor不会在启动（或者它的配置发生改变）之时生成它，但它将会为那个vassal创建socket，然后监控是否有任何东西连接到它上面。

在第一个连接时，会生成vassal，并将socket作为文件描述符0传递。文件描述符0总是由uWSGI检查，因此你无需在vassal文件中指定一个--socket选项。这会对uwsgi socket自动使用，如果你使用其他协议 (例如http或者fastcgi)，那么你必须涌--protocol选项来指定它

.. important::

  如果你将在你的vassal配置中定义与emperor用于按需动作相同的socket，那么vassal将会覆盖那个socket文件。那可能会导致非预期行为，例如，那个vassal的按需激活将会只能用一次。

带基于文件系统的帝国监控器的按需vassal
---------------------------------------------------------

对于基于文件系统的帝国监控器，例如 ``dir://`` 或者 ``glob://`` ，定义按需vassal会调用用于你的emperor的三个额外的设置之一的定义：

--emperor-on-demand-extension <ext>
***********************************

这将会指示Emperor检查一个名为<vassal>+<ext>的文件，如果这个文件可用，那么将会读取它，它的内容将会当成要等待的socket使用：

.. code-block:: sh

   uwsgi --emperor /etc/uwsgi/vassals --emperor-on-demand-extension .socket

假设/etc/uwsgi/vassals中有一个myapp.ini文件，将会搜索一个/etc/uwsgi/vassals/myapp.ini.socket（然后它的内容会被当成socket名）。注意，myapp.ini.socket并不是一个socket！这个文件只包含实际socket的路径 (tcp或者unix)。

--emperor-on-demand-directory <dir>
***********************************

这是一个不大通用的方法，只支持UNIX socket。基本上，vassal的名字（不带扩展名和路径）会被附加到指定目录之后，然后加上.socket扩展名，并被当成按需socket使用：

.. code-block:: sh

   uwsgi --emperor /etc/uwsgi/vassals --emperor-on-demand-directory /var/tmp

使用前一个例子，socket /var/tmp/myapp.socket将会被自动绑定。

--emperor-on-demand-exec <cmd>
******************************

这是为按需动作定义socket的最灵活的方式，并且（非常有可能）你将会把它用在非常大型的部署中。每当添加了一个新的vassal，就会运行提供的命令，将完整的路径作为第一个参数传递给vassal配置文件。这个命令的STDOUT会被当成socket名使用。

使用带其他帝国监控器的按需vassal
----------------------------------------------------

对于一些帝国监控器来说，例如 ``pg://``, ``mongodb://``, ``zmq://``，按需激活的socket是由帝国监控器自身返回的。例如，对于 ``pg://`` ，如果数据库上执行的查询返回了超过5个字段，那么第6个字段将会被当成按需激活的socket使用。见 :doc:`ImperialMonitors` 以获取更多信息。

对于一些帝国监控器来说，例如 ``amqp://`` ，尚不能使用socket激活。

将按需vassal与 ``--idle`` 和 ``--die-on-idle`` 组合在一起
-----------------------------------------------------------------

对于真正的按需应用，你可以添加 ``--idle`` 和 ``--die-on-idle`` 选项到每个vassal中。这将会允许挂起火灾完全关闭那些不再处理请求的应用。不带 ``--die-on-idle`` 使用 ``--idle`` 的工作方式将会非常像不用emperor，但是添加了 ``--die-on-idle`` 将会赐予你完全关闭应用并返回按需模式的超能力。

当vassal优雅地死掉时，Emperor会简单地将它放回按需模式，当有任何等待的请求或者socket的时候，会将其唤回。

.. important::

  正如前面提到的，你应该 **永远不** 把那个用于按需模式被传递给emperor的socket放到你的vassal配置文件中。对于unix socket，socket存活的文件路径将会被新的socket覆盖，但是老的socket将仍然连接到你的emperor。Emperor将会监听那个老的socket的连接，但所有的请求将会到新的socket上去。这意味着，如果你的vassal因为idle状态被关闭，那么它将 **永远不** 会被放回去 (emperor将不会为按需socket接收任何连接)。

  对于tcp socket，那将会使得每个请求被处理 **两次** 。
