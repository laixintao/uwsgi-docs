将你的1.x uWSGI实例升级至2.0 (work in progress)
============================================================

下面的说明是提供给那些从1.0, 1.2和1.4更新至uWSGI 2.0的用户的。

使用1.9版本树的用户可以跳过此文档，因为2.0是一个“稳定的/冻结的”1.9。

新特性
----------

License更改
**************

uWSGI is now GPL2 + linking exception instead of plain GPL2.

This should address some legal issues with users compiling uWSGI as a library (libuwsgi.so) and loading non-GPL compatible plugins/libraries.

默认为非阻塞
***********************

All of the I/O of the uWSGI stack (from the core to the plugins) is now fully non-blocking.

No area in the whole stack is allowed to block (except your app obviously), and plugins must use uWSGI's I/O API.

When you load loop engines like gevent or Coro::AnyEvent, the uWSGI internals are patched to support their specific non-blocking hooks.

What does this mean for app developers?

Well, the most important aspect is that network congestions or kernel problems do not block your instances and badly behaving peers
are closed if they do not unblock in the socket-timeout interval (default 4 seconds).

更新，更快以及更好的解析器
********************************

uWSGI 2.0 has support for pluggable protocols. The following protocols are supported and all of them have been updated
for better performance:

* ``uwsgi`` -- the classic uwsgi parser, improved for reduced syscall usage
* ``http`` -- the classic HTTP parser, improved for reduced syscall usage (supports the PROXY1 protocol)
* ``https`` -- (new) support for native HTTPS
* ``fastcgi`` -- classic FastCGI parser, improved for reduced syscall usage
* ``scgi`` -- (new) support for SCGI
* ``suwsgi`` -- (new) secured uwsgi, uwsgi over SSL (supported by Nginx 1.5)
* ``puwsgi`` -- (new) persistent uwsgi, uwsgi with persistent connections, supported only internally
* ``mongrel2`` -- classic zeromq/mongrel2 support, now it is exposed as a plugin
* ``raw`` -- (new) fake parser, allows you to write applications directly working on file descriptors

新的重载方式
******************

uWSGI 2.0 introduces a blast of new ways for reloading instances.

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

It is pretty fun (and easy) to write uWSGI plugins, but (funnily enough) the worst aspect was building them, as dealing with build profiles, cflags, ldflags and friends tend to lead to all sorts of bugs and crashes.

A simplified (and saner) build system for external plugins has been added. Now you only need to call the uwsgi binary you want to build the plugin for:

.. code-block:: sh

   uwsgi --build-plugin <plugin>
   
where <plugin> is the directory where the plugin sources (and the uwsgiplugin.py file) are stored.

.. seealso::

   :doc:`ThirdPartyPlugins`

Strict mode
***********

while having the freedom of defining custom options in uWSGI config files is a handy features, sometimes typos will
bring you lot of headaches.

Adding --strict to your instance options will instruct uWSGI config parser to raise an error when not-available options have been specified.

If you are in trouble and want to be sure you did not have written wrong options, add --strict and retry

Cygwin支持
**************

Yes, you can now build and run uWSGI on Windows systems :(

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

* Snapshotting mode is no longer available. Check the new graceful reloading ways for better approaches.
* Mongrel2 support is no longer a built-in. you have to build the 'mongrel2' plugin to pair uWSGI with Mongrel2.
* LDAP and Sqlite support has been moved to two plugins, you need to load them for using their features.
* Dynamic options are no more.
* The 'admin' plugin is gone.
* Probes have been removed, the alarm framework presents better ways to monitor services.
* The shared area API changed dramatically, check the new sharedarea docs.
