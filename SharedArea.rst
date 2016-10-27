SharedArea ——  uWSGI组件间共享内存页
=========================================================

.. warning::

  SharedArea是一种非常底层的机制。想要一个更易于使用的替代品，可以看看 :doc:`Caching<Caching>` 和 :doc:`Queue<Queue>` 框架。
  
.. warning::

  本页面适用于uWSGI 1.9.21中引进的“新一代”sharedarea，不再支持较老的API。

sharedarea子系统允许你以一种非常快速（安全）的方式，在你的uWSGI组件（worker, spooler, mule等等）之间共享内存页。

与较高层次的 :doc:`caching framework<Caching>` 相比，sharedarea操作是一种更快的方式 (单一的拷贝，而不是如缓存所需的那样要进行双重拷贝)，并且它为特定的需求提供不同的优化。

每一个sharedarea (是哒，你可以拥有多个区域) 都有一个大小 (一般根据页面数指定)，因此，如果你在拥有4 KiB页面的系统上需要一个8 KiB的共享区域，那么你会使用 ``sharedarea=2`` 。

sharedarea子系统是完全线程安全的。

简单的选项 VS 键值
***********************

sharedarea子系统公开了 (目前) 一个单一的选项： ``--sharedarea``.

它接收两种参数：页面数 (简单的方法) 或者一个键值参数 (用于高级调试)。

The following keyval keys are available:

* ``pages`` -- 设置页面数
* ``file`` -- create the sharedarea from a file that will be ``mmap``\ ed
* ``fd`` -- create the sharedarea from a file descriptor that will be ``mmap``\ ed
* ``size`` -- mainly useful with the ``fd`` and ``ptr`` keys to specify the size of the map (can be used as a shortcut to avoid calculation of the ``pages`` value too)
* ``ptr`` -- 直接映射该区域到指定的内存指针。

API
*******

The API is pretty big, the sharedarea will be the de-facto toy for writing highly optimized web apps (especially for embedded systems).

Most of the documented uses make sense on systems with slow CPUs or very small amounts of memory.

``sharedarea_read(id, pos[, len])``
    Read ``len`` bytes from the specified sharedarea starting at offset ``pos``. 如果未指定 ``len`` ，那么将会the memory will be read til the end (从 ``pos`` 开始)。
``sharedarea_write(id, pos, string)``
    Write the specified ``string`` (it is language-dependent, obviously) to the specified sharedarea at offset ``pos``.
``sharedarea_read8|16|32|64(id, pos)``
    Read a signed integer (8, 16, 32 or 64 bit) from the specified position.
``sharedarea_write8|16|32|64(id, pos)``
    Write a signed integer (8, 16, 32 or 64 bit) to the specified position.
``sharedarea_inc8|16|32|64(id, pos)``
    Increment the signed integer (8, 16, 32 or 64 bit) at the specified position.
``sharedarea_dec8|16|32|64(id, pos)``
    Decrement the signed integer (8, 16, 32 or 64 bit) at the specified position.
``sharedarea_wait(id[, freq, timeout])``
    Wait for modifications of the specified sharedarea (see below).
``sharedarea_rlock(id)``
    lock a shared area for read (use only if you know what you are doing, generally the sharedarea api functions implement locking by themselves)
``sharedarea_wlock(id)``
    lock a shared area for write (use only if you know what you are doing, generally the sharedarea api functions implement locking by themselves)
``sharedarea_unlock(id)``
    unlock a shared area (use only if you know what you are doing, generally the sharedarea api functions implement locking by themselves)

等待更新
*******************

One of the most powerful features of sharedareas (compared to caching) is "waiting for updates". Your worker/thread/async_core can be suspended
until a sharedarea is modified.

Technically, a millisecond-resolution timer is triggered, constantly checking for updates (the operation is very fast, as the sharedarea object has an update counter, so we only need to check that value for changes).

可选API
************

The following functions require specific features from the language, so not all of the language plugins are able to support them.

``sharedarea_readfast(id, pos, object, [, len])``
    Read ``len`` bytes from the specified sharedarea starting at offset ``pos`` to the specified object. If ``len`` is not specified, the memory will be read til the end (starting from ``pos``).
    Currently is implemented only for Perl.
    
``sharedarea_memoryview(id)``
    returns python memoryview object you can directly manipulate (works only on CPython)

``sharedarea_object(id)``
    some plugin exposes an alternative way to create sharedareas from internal objects. This functions returns the original object (currently implemented only on CPython on top of bytearrays using ``--py-sharedarea <size>`` option)

Websockets integration API
**************************

This is currently supported only in the psgi/perl plugin:

``websocket_send_from_sharedarea(id, pos)``
    send a websocket message directly from the specified sharedarea

``websocket_send_binary_from_sharedarea(id, pos)``
    send a websocket binary message directly from the specified sharedarea

高级使用 (from C)
***********************


Work in progress.

Check https://github.com/unbit/uwsgi-capture for an example of sharedarea managed from C
