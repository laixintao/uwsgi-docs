uWSGI信号框架
==========================

.. warning:: uwsgi信号的原始使用只提供给高级用户。对于一个更优雅的抽象，你应该看看 :doc:`PythonDecorators` 。

.. note:: uWSGI信号量与UNIX/Posix信号量_毫无共同之处_ (如果你找的是那些，那么 :doc:`Management` 才是你要看的)。

随着时间的推移，你的uWSGI堆栈越来越大，你添加spooler，更多的进程，更多的插件，等等。你添加的功能越多，你就越需要所有这些组件之间能彼此通信。

现今丰富/高级的web应用的另一个重要任务是响应不同事件。一个事件可能是一个文件修改，一个新的集群节点冒出来，另一个（黯然）死去，一个定时器时间已经到了……任何你能想象到的事件。

通信和事件管理都由相同的子系统 —— uWSGI信号框架， 管理。

uWSGI信号是由socket管理的，因此它们*够可靠*。当你发送一个uWSGI信号时，你可以保证它会被转发。

信号表
-----------------

信号是简单的*1字节*消息，它可以由master进程路由给worker和spooler.

当一个workder接收到一个信号，它会搜索信号表，查找对应的处理程序并执行。

信号表由所有worker共享 (并通过共享锁对抗竞争条件)。

每个uWSGI进程 (虽然主要是master)可以写信号表，以设置信号处理程序和接收进程。

.. warning::
  
  要经常注意谁会运行信号处理程序。它必须能够访问信号处理程序本身。这意味着，如果你在 ``worker1`` 中定义一个新的函数，并将其注册为信号处理程序，那么只有 ``worker1`` 可以运行它。注册信号最好的方法是在master中定义它们，这样(多亏了 ``fork()``)所有的worker都能看到它们。

定义信号处理程序
------------------------

要管理信号表，uWSGI API提供了一个简单的函数， :py:meth:`uwsgi.register_signal`.

下面是两个简单的定义信号表项的例子，分别用Python和Lua编写。

.. code-block:: py

    import uwsgi
    
    def hello_signal(num):
        print "i am the signal %d" % num
    
    def hello_signal2(num):
        print "Hi, i am the signal %d" % num
    
    # define 2 signal table items (30 and 22)
    uwsgi.register_signal(30, "worker", hello_signal)
    uwsgi.register_signal(22, "workers", hello_signal2)

.. code-block:: lua

    function hello_signal(sig)
        print("i am Lua, received signal " .. sig ..)
    end
    
    # define a single signal table item (signal 1)
    uwsgi.register_signal(1, "worker", hello_signal)
    

信号目标
---------------

uwsgi.register_signal的第三个参数是'signal targer'。

它指示系统“谁”必须运行处理程序。默认情况下，目标是targer是表示“第一个可用worker”的'worker'。下面是可用的目标：

- workerN (只在workder N上运行信号处理程序)
- worker/worker0 (默认，在第一个可用的worker上运行信号处理程序)
- workers (在所有worker上运行信号处理程序)
- active-workers (在所有活跃的 [non-cheaped] worker上运行信号处理程序)
- spooler (在第一个可用的spooler上运行信号处理程序)
- mules (在所有的mule上运行信号处理程序)
- muleN (在mule N上运行信号处理程序)
- mule/mule0 (在第一个可用的mule上运行信号处理程序)
- farmN/farm_XXX (在mule farm N或者指定的XXX上运行信号处理程序)

引发信号
---------------

可以使用 :py:meth:`uwsgi.signal` 引发信号。当你发送一个信号时，它会被拷贝到master的队列中。然后，master会检查信号表并调度消息。

外部事件
---------------

uWSGI信号最有用的特性是，它们可以用于宣布外部事件。

编写可用外部事件的时机是

* 文件系统修改
* timer/rb_timer
* cron

其他事件是通过插件暴露出来的，例如，每当一个postgres通知通道准备好时，https://github.com/unbit/uwsgi-pgnotifyj就会引发一个信号。

文件系统修改
^^^^^^^^^^^^^^^^^^^^^^^^

要将一个特定的文件/目录修改事件映射到一个信号上，你可以使用 :py:meth:`uwsgi.add_file_monitor` 。

一个例子：

.. code-block:: py

    import uwsgi
    
    def hello_file(num):
            print "/tmp has been modified !!!"
    
    uwsgi.register_signal(17, "worker", hello_file)
    uwsgi.add_file_monitor(17, "/tmp")

从现在开始，每次 ``/tmp`` 被修改时，将会引发信号17，然后第一个可用worker将会运行 ``hello_file`` 。

定时器
^^^^^^ 

定时器是web编程中另一个有用特性 —— 例如清理会话、购物车等诸如此类。

定时器是利用内核工具实现的（BSD系统上的kqueue，以及现代Linux内核上的timerfd()）。uWSGI还包含对rb_timer的支持，这是一个在用户空间中，使用红黑树实现的定时器。

要注册一个定时器，可以使用 :meth:`uwsgi.add_timer` 。要注册一个rb_timer，可以使用 :meth:`uwsgi.add_rb_timer` 。

.. code-block:: py

    import uwsgi
    
    def hello_timer(num):
            print "2 seconds elapsed, signal %d raised" % num
    
    def oneshot_timer(num):
            print "40 seconds elapsed, signal %d raised. You will never see me again." % num
    
    
    uwsgi.register_signal(26, "worker", hello_timer)
    uwsgi.register_signal(30, "", oneshot_timer)
    
    uwsgi.add_timer(26, 2) # never-ending timer every 2 seconds
    uwsgi.add_rb_timer(30, 40, 1) # one shot rb timer after 40 seconds
    
每2秒钟就会引发一次信号26，并且由第一个可用worker处理。40秒过后会引发一次信号30，然后只执行一次。

signal_wait和signal_received
-------------------------------

未注册信号（那些没有相关处理程序的）将会路由到第一个可用worker，以使用 :meth:`uwsgi.signal_wait` 函数。

.. code-block:: xxx

    uwsgi.signal_wait()
    signum = uwsgi.signal_received()

你可以将外部事件（文件监控、定时器……）和这项技术结合起来，以实现基于事件的应用。一个很好的例子是聊天服务器，其中，每个核等待用户发送的文本。

你也可以通过传递一个信号数字给 ``signal_wait`` 来等待一个特定的（甚至注册了的）信号。

待办/已知问题
-----------------

* 不能移除信号表项（这会尽快解决）
* 迭代只适用于rb_timer
* uwsgi.signal_wait()在异步模式下无效（将会解决）
* 添加迭代到文件监控（以允许定时器的一次事件）
