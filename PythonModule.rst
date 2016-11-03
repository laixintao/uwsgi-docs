uwsgi Python模块
=======================

uWSGI服务器自动添加一个 ``uwsgi`` 模块到你的Python应用中。

这对于配置uWSGI服务器是有用的，使用它的内部函数，并获取分析数据（以及检测你是否真的在uWSGI下运行）。

.. note:: 许多这些函数当前不幸未公开。

模块级别的全局
--------------------

.. default-domain:: py

.. module:: uwsgi

.. data:: numproc

   当前运行的进程/worker数。

.. data:: buffer_size

   当前配置的缓冲区大小，以字节为单位。

.. data:: started_on (int)

   uWSGI启动的Unix时间戳。

.. data:: fastfuncs

   这是一个用来定义 :doc:`FastFuncs` 的字典。

.. data:: applist

   这是当前配置的应用列表。

.. TODO: Practical use cases for applist?

.. data:: applications

   这是动态应用字典。

   .. seealso:: :ref:`PythonAppDict`

.. data:: message_manager_marshal

   当uWSGI服务器接收到一个已编组消息( marshalled message)时运行的回调。

.. TODO: What _is_ this?

.. data:: magic_table

   配置占位符的魔术表。

.. data:: opt

   当前配置选项，包括任何自定义占位符。

缓存函数
---------------


.. function:: cache_get(key[, cache_name])

   从缓存中获取一个值。

   :param key: 要读取的缓存键。
   :param cache_name: 多缓存模式中的缓存名字 (可以是name@address这样的格式)。可选。


.. function:: cache_set(key, value[, expire, cache_name])

   设置缓存中的一个值。

   :param key: 要写的缓存键。
   :param value: 要写的缓存值。
   :param expire: 值的失效时间，以秒为单位。
   :param cache_name: 多缓存模式中的缓存名字 (可以是name@address这样的格式)。可选。


.. function:: cache_update(key, value[, expire, cache_server])


.. function:: cache_del(key[, cache_name])

   从缓存中删除给定的缓存值。

   :param key: 要删除的缓存键
   :param cache_name: 多缓存模式中的缓存名字 (可以是name@address这样的格式)。可选。

.. function:: cache_exists(key[, cache_name])

   快速检查缓存中是否有与给定键关联的值。

   :param key: 要坚持的缓存键
   :param cache_name: 多缓存模式中的缓存名字 (可以是name@address这样的格式)。可选。

.. function:: cache_clear()

队列函数
---------------

.. function:: queue_get()


.. function:: queue_set()


.. function:: queue_last()


.. function:: queue_push()


.. function:: queue_pull()


.. function:: queue_pop()


.. function:: queue_slot()


.. function:: queue_pull_slot()


SNMP函数
--------------

.. function:: snmp_set_community(str)

   :param str: 包含新的community值的字符串。

   设置SNMP community字符串。

.. function:: snmp_set_counter32(oidnum, value)
.. function:: snmp_set_counter64(oidnum, value)
.. function:: snmp_set_gauge(oidnum, value)

   :param oidnum: 一个包含oid数字目标的整数
   :param value: 一个包含计数器或测量新值的整数。

   设置计数器或测量为一个指定的值。

.. function:: snmp_incr_counter32(oidnum, value)
.. function:: snmp_incr_counter64(oidnum, value)
.. function:: snmp_incr_gauge(oidnum, value)
.. function:: snmp_decr_counter32(oidnum, value)
.. function:: snmp_decr_counter64(oidnum, value)
.. function:: snmp_decr_gauge(oidnum, value)

   :param oidnum: 一个包含oid数字目标的整数。
   :param value: 一个包含增加/减少计数器或测量的量整数值。如果未指定，默认为1.

   增加或减少计数器或测量一个特定的量。

   .. note:: uWSGI OID树从1.3.6.1.4.1.35156.17开始

spooler函数
-----------------

.. function:: send_to_spooler(message_dict=None, spooler=None, priority=None, at=None, body=None, **kwargs)

   :param message_dict: 发送到spool的消息 (字符串键，字符串值)。要么这样，要么设置**kwargs。
   :param spooler: 要使用的spooler (id或者目录)
   :param priority: 消息的优先级。越大越不重要。
   :param at: 应该处理这条信息的最小的UNIX时间戳。
   :param body: 除了消息字典本身，添加到消息的一个二进制 (字节字符串)体。它的值可通过消息中的 ``body`` 键访问。

   发送数据到 :doc:`Spooler`. 又名 `spool()`.

   .. note:: 所有的关键字参数也可以在消息字典中传递。这意味着它们是保留字，嗯，某种程度上……

.. function:: set_spooler_frequency(seconds)

   设置spooler运行的频率。


.. function:: spooler_jobs()


.. function:: spooler_pid()

.. function:: spooler_get_task(path)

   :param path: 读取任务的相对/绝对路径


高级方法
----------------

.. function:: send_message()

   使用 :doc:`Protocol` 发送一般消息。

   .. note:: 这个函数被称为 ``send_uwsgi_message()`` ，直到版本 `2f970ce58543278c851ff30e52758fd6d6e69fdc`.


.. function:: route()


.. function:: send_multi_message()

   使用 :doc:`Protocol` 发送一条一般消息到多个接收者

   .. note:: 这个函数被称为 ``send_multi_uwsgi_message()`` ，直到 `2f970ce58543278c851ff30e52758fd6d6e69fdc` 版本。

   .. seealso:: 查看 :doc:`Clustering` 获得更多例子



.. function:: accepting(accepting=True)

   设置当前worker的接受标志的值。在同时使用 `Overriding Workers`_ 和加载时建立链的时候必须。

   .. seealso:: :doc:`WorkerOverride`


.. function:: reload()

   优雅地重载uWSGI服务器栈。

   .. seealso:: :doc:`Reload`


.. function:: stop()


.. function:: workers() -> dict

   为当前服务器获取所有worker的统计数据字典。返回一个字典。


.. function:: masterpid() -> int

   返回uWSGI master进程的进程标识符 (PID)。


.. function:: total_requests() -> int

   返回的至今由uWSGI worker池管理的请求总数。

.. function:: get_option()

   也可作为 `getoption()` 使用。

.. function:: set_option()

   也可作为 `setoption()` 使用。


.. function:: sorry_i_need_to_block()


.. function:: request_id()


.. function:: worker_id()


.. function:: mule_id()


.. function:: log()


.. function:: log_this_request()


.. function:: set_logvar()


.. function:: get_logvar()


.. function:: disconnect()


.. function:: grunt()


.. function:: lock(locknum=0)

   :param locknum: 要锁的锁号。Lock 0总是可用的。


.. function:: is_locked()


.. function:: unlock(locknum=0)

   :param locknum: 要解锁的锁号。Lock 0总是可用的。


.. function:: cl()


.. function:: setprocname()


.. function:: listen_queue()


.. function:: register_signal(num, who, function)

   :param num:配置的信号数
   :param who: 一个魔法字符串，会设置哪个/些进程接收该信号。

      * ``worker``/``worker0`` 将发送信号给第一个可用worker。如果你指定过一个空字符串，那么这是默认值。
      * ``workers`` 会发送信号给每个worker。
      * ``workerN`` (N > 0) 会发送信号给worker N。
      * ``mule``/``mule0`` 会发送信号给第一个可用mule。 (见 :doc:`Mules`)
      * ``mules`` 会发送信号给所有mule。
      * ``muleN`` (N > 0) 会发送信号给mule N。
      * ``cluster`` 会发送信号给集群中的所有节点。警告：未实现。
      * ``subscribed`` 会发送信号给所有订阅节点。警告：未实现。
      * ``spooler`` 会发送信号给spooler。

      ``cluster`` 和 ``subscribed`` 特殊，因为它们会将信号发送给所有集群/订购节点的master。其他节点将不得不定义一个本地处理器，以避免可怕的信号风暴循环。

   :param function: 一个回调，接收一个数字参数。

.. function:: signal(num)

   :param num: 引发的信号


.. function:: signal_wait([signum])

   阻塞进程/线程/异步核心直到接收到了一个信号。使用 ``signal_received`` 来获取接收到的信号值。
   如果已经为该信号注册了一个处理函数，那么 ``signal_wait`` 将被终端，而实际的处理函数将处理该信号。

   :param signum: 可选 - 要等待的信号


.. function:: signal_registered()


.. function:: signal_received()

   获取最后收到的信号值。结合 ``signal_wait`` 使用。


.. function:: add_file_monitor()


.. function:: add_timer(signum, seconds)

   :param signum: 引发的信号值。
   :param seconds: 引发信号的时间间隔。


.. function:: add_probe()


.. function:: add_rb_timer(signum, seconds[, iterations=0])

   Add an user-space (red-black tree backed) timer.

   :param signum: 引发的信号值。
   :param seconds: 引发信号的时间间隔。
   :param iterations: 引发信号的次数。0 (默认) 表示无数次。


.. function:: add_cron(signal, minute, hour, day, month, weekday)

   对于时间参数，你可以使用语法 ``-n`` 来表示“每n”。例如， ``hour=-2`` 将声明会每隔一个小时发送一次信号。

   :param signal: 引发的信号值。
   :param minute: 运行该事件的分钟。
   :param hour: 运行该事件的小时。
   :param day: 运行该事件的天。这会和 ``weekday`` 进行或运算。
   :param month: 运行该事件的月。
   :param weekday: 运行该事件的工作日。这会与 ``day`` 进行或运算。 (根据POSIX标准，0是星期天，6是星期一)

.. function:: register_rpc()


.. function:: rpc()


.. function:: rpc_list()


.. function:: call()


.. function:: sendfile()


.. function:: set_warning_message()


.. function:: mem()


.. function:: has_hook()


.. function:: logsize()


.. function:: send_multicast_message()


.. function:: cluster_nodes()


.. function:: cluster_node_name()


.. function:: cluster()


.. function:: cluster_best_node()


.. function:: connect()


.. function:: connection_fd()


.. function:: is_connected()


.. function:: send()


.. function:: recv()


.. function:: recv_block()


.. function:: recv_frame()


.. function:: close()


.. function:: i_am_the_spooler()


.. function:: fcgi()


.. function:: parsefile()


.. function:: embedded_data(symbol_name)

   :param string: 要提取的符号名。

   从uWSGI二进制镜像中提取一个符号。

   .. seealso:: :doc:`Embed`


.. function:: extract()


.. function:: mule_msg(string[, id])

   :param string: 要发送的字节字符串消息。
   :param id: 可选 - 接收该消息的mule ID。如果你不指定一个ID，那么这条消息就会被发送到第一个可用的编程mule。

   发送一条消息给一个mule。

.. function:: farm_msg()


.. function:: mule_get_msg()

   :return: 一旦接收到一条mule消息，则返回。

   阻塞，直到接收到了一条Mule消息，并返回这条消息。可以在同一个编程mule中从多个线程中调用该函数。


.. function:: farm_get_msg()


.. function:: in_farm()


.. function:: ready()


.. function:: set_user_harakiri()


异步函数
---------------


.. function:: async_sleep(seconds)

   暂停处理当前请求 ``seconds`` 秒，并将控制权传给下一个异步核心。

   :param seconds: 休眠时间，以秒为单位。


.. function:: async_connect()


.. function:: async_send_message()


.. function:: green_schedule()


.. function:: suspend()

   挂起处理当前请求，并将控制权传给下一个要求控制权的异步核心。

.. function:: wait_fd_read(fd[, timeout])

   挂起处理当前请求，直到文件描述符 ``fd`` 上有可读数据。在挂起以添加更多文件描述符到被监控的集合中之前可能会被多次调用。

   :param fd: 文件描述符号。
   :param timeout: 可选的超时 (如果省略，则是无限).


.. function:: wait_fd_write(fd[, timeout])

   挂起处理当前请求，直到文件描述符 ``fd`` 上没有更多可写的内容。
   在添加更多文件描述符到被监控的集合中之前可能会被多次调用。

   :param fd: 文件描述符号。
   :param timeout: 可选的超时 (如果省略，则是无限).


.. _SharedAreaAPI:

SharedArea函数
--------------------

.. seealso:: :doc:`SharedArea`

.. function:: sharedarea_read(pos, len) -> bytes

   从uWSGI :doc:`SharedArea` 读取一个字节字符串。

   :param pos: 读取的起始位置。
   :param len: 要读取的字节数。
   :return: 读取的字节，或者当共享区域未启用/读取请求无效时，返回 ``None`` 。

.. function:: sharedarea_write(pos, str) -> long

   将一个字节字符串写入到uWSGI :doc:`SharedArea`.

   :param pos: 写入的起始位置。
   :param str: 要写入的字节字符串。
   :return: 已写入的字节数，或者当共享区域未启用/不能完全结束写入时，返回 ``None`` 。

.. function:: sharedarea_readbyte(pos) -> int

   从uWSGI :doc:`SharedArea` 读取单个字节。

   :param pos: 读取的起始位置。
   :return: 读取的字节，或者当共享区域未启用/读取请求无效时，返回 ``None`` 。

.. function:: sharedarea_writebyte(pos, val) -> int

   将单个字节写入到uWSGI :doc:`SharedArea`.

   :param pos: 写入该值的位置。
   :param val: 要写入的值。
   :type val: 整型
   :return: 已写入的字节，或者当共享区域未启用/写请求无效时，返回 ``None`` 。

.. function:: sharedarea_readlong(pos) -> int

   从uWSGI :doc:`SharedArea` 读取一个64位（8字节）长的值。

   :param pos: 读取的位置。
   :return: 读取的值，或者当共享区域未启用/读请求无效时，返回 ``None`` 。

.. function:: sharedarea_writelong(pos, val) -> int

   将一个64位（8字节）长的值写入到uWSGI :doc:`SharedArea`.

   :param pos: 要写入该值的位置。
   :param val: 要写的值。
   :type val: long
   :return: 已写的值，或者当共享区域未启用/写请求无效时，返回 ``None`` 。

.. function:: sharedarea_inclong(pos) -> int

   自动增加uWSGI :doc:`SharedArea` 中的一个64位long值。

   :param pos: 值的位置。
   :type val: long
   :return: 给定位置上的新的值，或者当共享区域未启用/读请求无效时，返回 ``None`` 。

Erlang函数
----------------

.. function:: erlang_send_message(node, process_name, message)

.. function:: erlang_register_process(process_name, callable)

.. function:: erlang_recv_message(node)

.. function:: erlang_connect(address)

   :return: 文件描述符，或者错误时返回-1

.. function:: erlang_rpc(node, module, function, argument)

