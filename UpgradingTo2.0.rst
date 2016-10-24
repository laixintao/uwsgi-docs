将你的1.x uWSGI实例升级至2.0 (work in progress)
============================================================

下面的说明是提供给那些从1.0, 1.2和1.4更新至uWSGI 2.0的用户的。

使用1.9版本树的用户可以跳过此文档，因为2.0是一个“稳定的/冻结的”1.9。

新特性
----------

License更改
**************

uWSGI现在是GPL2 + linking exception，而不是纯GPL2了。

这应该会解决那些将uWSGI作为一个库(libuwsgi.so)进行编译，并加载GPL不兼容的插件/库的用户的一些合法性问题。

默认为非阻塞
***********************

uWSGI栈的所有I/O (从核心到插件) 现在完全是非阻塞的了。

整个栈没有一个地方允许阻塞（除非你对应用明显地进行了阻塞），而插件必须使用uWSGI的I/O API。

当你加载诸如gevent 或者 Coro::AnyEvent 这样的循环引擎时，应当修改uWSGI内部逻辑，以支持它们特定的非阻塞钩子。

那么，对于应用开发者而言，这又意味着什么呢？

嗯，最重要的方面是，网络堵塞，或者内核问题不会阻塞你的实例，并且如果表现糟糕的对端在socket超时间隔(默认是4秒)中阻塞了，会关闭它们.

更新，更快以及更好的解析器
********************************

uWSGI 2.0支持可插拔协议。支持以下协议，并对它们所有都进行了更新，以获取更好的性能：

* ``uwsgi`` -- 经典的uwsgi解析器，改善以减少系统调用的使用
* ``http`` -- 经典的HTTP解析器，改善以减少系统调用的使用 (支持PROXY1协议)
* ``https`` -- (新的)原生HTTPS支持
* ``fastcgi`` -- 经典的FastCGI解析器，改善以减少系统调用的使用
* ``scgi`` -- (新的) SCGI支持
* ``suwsgi`` -- (新的) 安全的uwsgi，SSL之上的uwsgi (由Nginx 1.5支持)
* ``puwsgi`` -- (新的) 持久化uwsgi，带有持久化连接的uwsgi，只有内部支持
* ``mongrel2`` -- 经典的zeromq/mongrel2支持，现在，它作为一个插件公开
* ``raw`` -- (新的) 假解析器，允许你编写直接运行在文件描述符之上的应用

新的重载方式
******************

uWSGI 2.0引入了一堆重载实例的新方法。

.. seealso::

   :doc:`TheArtOfGracefulReloading`

其他新特性
******************

* :doc:`MasterFIFO` -- A signal-free new approach for managing your instances.
* :doc:`Caching` & :doc:`tutorials/CachingCookbook` -- The new generation caching subsystem.
* :doc:`SharedArea` -- The new sharedarea.
* :doc:`SNI`
* :doc:`Legion`
* :doc:`WebSockets`
* :doc:`Hooks`
* :doc:`Transformations`
* :doc:`Namespaces`
* :doc:`FreeBSDJails`
* :doc:`Metrics`
* :doc:`tutorials/GraphiteAndMetrics`
* :doc:`RPC` -- now supports 64-bit length responses

新的插件构建系统
***********************

写uWSGI插件是灰常有趣（并且简单）的，但是（有趣的是），最糟糕的方面就是构建它们，因为处理构建配置文件， cflags, ldflags和friends易于导致所有类型的错误和崩溃。

已添加一个用于外部插件的简化（且健全）的构建系统。现在，你只需要调用你想要添加插件的uwsgi二进制文件：

.. code-block:: sh

   uwsgi --build-plugin <plugin>
   
其中，<plugin>是保存插件源代码（以及uwsgiplugin.py文件）的目录。

.. seealso::

   :doc:`ThirdPartyPlugins`

strict模式
***********

虽然可以自由地在uWSGI配置文件中定义自定义选项是一种便捷的特性，但有时，打字错误将让你很头疼。

添加--strict到你的实例选项中将会在指定了不可用的选项的时候，指示uWSGI配置解析器引发一个错误。

如果你遇到了麻烦，并且想要确保你并未写错选项，那么添加--strict之后重试

Cygwin支持
**************

是哒，现在，你可以在Windows系统上构建并运行uWSGI了 :(

kFreeBSD支持
****************

PyPy支持
************

JVM支持
***********

Mono支持
************

V8支持
**********

更新说明
---------------

* snapshotting模式不再可用。看看新的优雅地重载方式，以获取更好的方法。
* 不再内置Mongrel2支持。你必须构建'mongrel2'插件，从而让uWSGI与Mongrel2匹配。
* LDAP和Sqlite支持已经被移动到两个插件了，你需要加载它们来使用它们的功能。
* 不再有动态选项。
* 'admin'插件消失了
* 已经移除了Probes，告警框架是监控服务更好的方式。
* 共享区域API发生了很大的改变，查看新的sharedarea文档。
