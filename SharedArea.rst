SharedArea ——  uWSGI组件间共享内存页
=========================================================

.. warning::

  SharedArea是一种非常底层的机制。想要一个更易于使用的替代品，可以看看 :doc:`Caching<Caching>` 和 :doc:`Queue<Queue>` 框架。
  
.. warning::

  本页面适用于uWSGI 1.9.21中引进的“新一代”sharedarea，不再支持较老的API。

sharedarea子系统允许你以一种非常快速（安全）的方式，在你的uWSGI组件（worker, spooler, mule等等）之间共享内存页。

与较高层次的 :doc:`caching framework<Caching>` 相比，sharedarea操作是一种更快的方式 (单一的拷贝，而不是如缓存所需的那样要进行双重拷贝)，并且它为特定的需求提供不同的优化。

每一个共享区域 (是哒，你可以拥有多个区域) 都有一个大小 (一般根据页面数指定)，因此，如果你在拥有4 KiB页面的系统上需要一个8 KiB的共享区域，那么你会使用 ``sharedarea=2`` 。

共享区域子系统是完全线程安全的。

简单的选项 VS 键值
***********************

sharedarea子系统公开了 (目前) 一个单一的选项： ``--sharedarea``.

它接收两种参数：页面数 (简单的方法) 或者一个键值参数 (用于高级调试)。

可以使用以下键值键：

* ``pages`` -- 设置页面数
* ``file`` -- 从一个将会被 ``mmap`` 的文件中创建共享区域
* ``fd`` -- 从一个将会被 ``mmap`` 的文件描述符中创建共享区域
* ``size`` -- 主要和 ``fd`` 和 ``ptr`` 键使用，用来指定映射的大小 (也可以作为一个快捷方式使用，以避免对 ``pages`` 值的计算)
* ``ptr`` -- 直接映射该区域到指定的内存指针。

API
*******

API是非常大的，共享区域将会是编写高度优化的web应用（特别是嵌入系统）实际上的玩物。

大多数已记录的使用在拥有缓慢的CPU或者非常少量的内存的系统上有意义。

``sharedarea_read(id, pos[, len])``
    从指定的共享区域的偏移 ``pos`` 起读取 ``len`` 字节。如果未指定 ``len`` ，那么将会读取到内存尾部 (从 ``pos`` 开始)。
``sharedarea_write(id, pos, string)``
    将指定的 ``string`` (显然，它是语言相关的) 写入到指定的共享区域偏移 ``pos`` 的位置上。
``sharedarea_read8|16|32|64(id, pos)``
    从指定的位置读取有符号整数 (8, 16, 32或者64位)。
``sharedarea_write8|16|32|64(id, pos)``
    将一个有符号整数 (8, 16, 32或者64位) 写入到指定的位置。
``sharedarea_inc8|16|32|64(id, pos)``
    递增指定位置的有符号整数 (8, 16, 32或者64位)。
``sharedarea_dec8|16|32|64(id, pos)``
    递减指定位置的有符号整数 (8, 16, 32或者64位)。
``sharedarea_wait(id[, freq, timeout])``
    等待指定共享区域的修改 (见下)。
``sharedarea_rlock(id)``
    为读取锁定一个共享区域 (只有在你知道你在做什么的时候使用它，一般来说，共享区域api函数自己实现锁)
``sharedarea_wlock(id)``
    为写入锁定一个共享区域 (只有在你知道你在做什么的时候使用它，一般来说，共享区域api函数自己实现锁)
``sharedarea_unlock(id)``
    解锁一个共享区域 (只有在你知道你在做什么的时候使用它，一般来说，共享区域api函数自己实现锁)

等待更新
*******************

共享区域的最有力特性之一 (与缓存相比) 是“等待更新”。你的worker/thread/async_core可以被挂起，直到修改了一个共享区域。

从技术上讲，一个分辨率为毫秒的计时器会被触发，不断地检测更新 (这个操作是非常快的，因为共享区域对象有一个更新计数器，英寸我们只需要检测那个值就知道有没有改动了)。

可选API
************

以下函数要求语言特定特性，所以并不是所有的语言插件都能够支持它们的。

``sharedarea_readfast(id, pos, object, [, len])``
    从指定的共享区域的偏移量 ``pos`` 开始，读取 ``len`` 字节到指定的对象中中。如果不指定 ``len`` ，那么将会读取到内存尾部 (从 ``pos`` 开始)。目前只对Perl实现。
    
``sharedarea_memoryview(id)``
    返回你可以直接操作的python memoryview对象 (只在CPython上有用)

``sharedarea_object(id)``
    一些插件公开了一个从内部对象创建共享区域的可选的方式。这个函数返回原始对象 (目前只在CPython中，基于字节数组实现，使用 ``--py-sharedarea <size>`` 选项)

Websockets集成API
**************************

这个目前只在psgi/perl插件中支持：

``websocket_send_from_sharedarea(id, pos)``
    直接从指定的共享区域发送一个websocket消息

``websocket_send_binary_from_sharedarea(id, pos)``
    直接从指定的共享区域发送一个websocket二进制消息

高级使用 (从C)
***********************


正在进行中。

看看https://github.com/unbit/uwsgi-capture中的一个从C管理共享区域的例子
