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

   :param oidnum: An integer containing the oid number target.
   :param value: An integer containing the new value of the counter or gauge.

   Sets the counter or gauge to a specific value.

.. function:: snmp_incr_counter32(oidnum, value)
.. function:: snmp_incr_counter64(oidnum, value)
.. function:: snmp_incr_gauge(oidnum, value)
.. function:: snmp_decr_counter32(oidnum, value)
.. function:: snmp_decr_counter64(oidnum, value)
.. function:: snmp_decr_gauge(oidnum, value)

   :param oidnum: An integer containing the oid number target.
   :param value: An integer containing the amount to increase or decrease the counter or gauge. If not specified the default is 1.

   Increases or decreases the counter or gauge by a specific amount.

   .. note:: uWSGI OID tree starts at 1.3.6.1.4.1.35156.17

spooler函数
-----------------

.. function:: send_to_spooler(message_dict=None, spooler=None, priority=None, at=None, body=None, **kwargs)

   :param message_dict: The message (string keys, string values) to spool. Either this, or **kwargs may be set.
   :param spooler: The spooler (id or directory) to use.
   :param priority: The priority of the message. Larger = less important.
   :param at: The minimum UNIX timestamp at which this message should be processed.
   :param body: A binary (bytestring) body to add to the message, in addition to the message dictionary itself. Its value will be available in the key ``body`` in the message.

   Send data to the :doc:`Spooler`. Also known as `spool()`.

   .. note:: Any of the keyword arguments may also be passed in the message dictionary. This means they're reserved words, in a way...

.. function:: set_spooler_frequency(seconds)

   Set how often the spooler runs.


.. function:: spooler_jobs()


.. function:: spooler_pid()

.. function:: spooler_get_task(path)

   :param path: The relative or absolute path to the task to read


高级方法
----------------

.. function:: send_message()

   Send a generic message using :doc:`Protocol`.

   .. note:: Until version `2f970ce58543278c851ff30e52758fd6d6e69fdc` this function was called ``send_uwsgi_message()``.


.. function:: route()


.. function:: send_multi_message()

   Send a generic message to multiple recipients using :doc:`Protocol`.

   .. note:: Until version `2f970ce58543278c851ff30e52758fd6d6e69fdc` this function was called ``send_multi_uwsgi_message()``.

   .. seealso:: :doc:`Clustering` for examples



.. function:: accepting(accepting=True)

   Set the accepting flag of the current worker to the value. This is
   required when using `Overriding Workers`_ and touch-chain-reload at
   the same time.

   .. seealso:: :doc:`WorkerOverride`


.. function:: reload()

   Gracefully reload the uWSGI server stack.

   .. seealso:: :doc:`Reload`


.. function:: stop()


.. function:: workers() -> dict

   Get a statistics dictionary of all the workers for the current server. A dictionary is returned.


.. function:: masterpid() -> int

   Return the process identifier (PID) of the uWSGI master process.


.. function:: total_requests() -> int

   Returns the total number of requests managed so far by the pool of uWSGI workers.

.. function:: get_option()

   Also available as `getoption()`.

.. function:: set_option()

   Also available as `setoption()`.


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

   :param locknum: The lock number to lock. Lock 0 is always available.


.. function:: is_locked()


.. function:: unlock(locknum=0)

   :param locknum: The lock number to unlock. Lock 0 is always available.


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

   :param num: the signal number to raise


.. function:: signal_wait([signum])

   Block the process/thread/async core until a signal is received. Use ``signal_received`` to get the number of the signal received.
   If a registered handler handles a signal, ``signal_wait`` will be interrupted and the actual handler will handle the signal.

   :param signum: Optional - the signal to wait for


.. function:: signal_registered()


.. function:: signal_received()

   Get the number of the last signal received. Used in conjunction with ``signal_wait``.


.. function:: add_file_monitor()


.. function:: add_timer(signum, seconds)

   :param signum: The signal number to raise.
   :param seconds: The interval at which to raise the signal.


.. function:: add_probe()


.. function:: add_rb_timer(signum, seconds[, iterations=0])

   Add an user-space (red-black tree backed) timer.

   :param signum: The signal number to raise.
   :param seconds: The interval at which to raise the signal.
   :param iterations: How many times to raise the signal. 0 (the default) means infinity.


.. function:: add_cron(signal, minute, hour, day, month, weekday)

   For the time parameters, you may use the syntax ``-n`` to denote "every n". For instance ``hour=-2`` would declare the signal to be sent every other hour.

   :param signal: The signal number to raise.
   :param minute: The minute on which to run this event.
   :param hour: The hour on which to run this event.
   :param day: The day on which to run this event. This is "OR"ed with ``weekday``.
   :param month: The month on which to run this event.
   :param weekday: The weekday on which to run this event. This is "OR"ed with ``day``. (In accordance with the POSIX standard, 0 is Sunday, 6 is Monday)

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

   :param string: The symbol name to extract.

   Extracts a symbol from the uWSGI binary image.

   .. seealso:: :doc:`Embed`


.. function:: extract()


.. function:: mule_msg(string[, id])

   :param string: The bytestring message to send.
   :param id: Optional - the mule ID to receive the message. If you do not specify an ID, the message will go to the first available programmed mule.

   Send a message to a mule.

.. function:: farm_msg()


.. function:: mule_get_msg()

   :return: A mule message, once one is received.

   Block until a mule message is received and return it. This can be called from multiple threads in the same programmed mule.


.. function:: farm_get_msg()


.. function:: in_farm()


.. function:: ready()


.. function:: set_user_harakiri()


异步函数
---------------


.. function:: async_sleep(seconds)

   Suspend handling the current request for ``seconds`` seconds and pass control to the next async core.

   :param seconds: Sleep time, in seconds.


.. function:: async_connect()


.. function:: async_send_message()


.. function:: green_schedule()


.. function:: suspend()

   Suspend handling the current request and pass control to the next async core clamoring for attention.

.. function:: wait_fd_read(fd[, timeout])

   Suspend handling the current request until there is something to be read on file descriptor ``fd``.
   May be called several times before yielding/suspending to add more file descriptors to the set to be watched.

   :param fd: File descriptor number.
   :param timeout: Optional timeout (infinite if omitted).


.. function:: wait_fd_write(fd[, timeout])

   Suspend handling the current request until there is nothing more to be written on file descriptor ``fd``.
   May be called several times to add more file descriptors to the set to be watched.

   :param fd: File descriptor number.
   :param timeout: Optional timeout (infinite if omitted).


.. _SharedAreaAPI:

SharedArea函数
--------------------

.. seealso:: :doc:`SharedArea`

.. function:: sharedarea_read(pos, len) -> bytes

   Read a byte string from the uWSGI :doc:`SharedArea`.

   :param pos: Starting position to read from.
   :param len: Number of bytes to read.
   :return: Bytes read, or ``None`` if the shared area is not enabled or the read request is invalid.

.. function:: sharedarea_write(pos, str) -> long

   Write a byte string into the uWSGI :doc:`SharedArea`.

   :param pos: Starting position to write to.
   :param str: Bytestring to write.
   :return: Number of bytes written, or ``None`` if the shared area is not enabled or the write could not be fully finished.

.. function:: sharedarea_readbyte(pos) -> int

   Read a single byte from the uWSGI :doc:`SharedArea`.

   :param pos: The position to read from.
   :return: Bytes read, or ``None`` if the shared area is not enabled or the read request is invalid.

.. function:: sharedarea_writebyte(pos, val) -> int

   Write a single byte into the uWSGI :doc:`SharedArea`.

   :param pos: The position to write the value to.
   :param val: The value to write.
   :type val: integer
   :return: The byte written, or ``None`` if the shared area is not enabled or the write request is invalid.

.. function:: sharedarea_readlong(pos) -> int

   Read a 64-bit (8-byte) long from the uWSGI :doc:`SharedArea`.

   :param pos: The position to read from.
   :return: The value read, or ``None`` if the shared area is not enabled or the read request is invalid.

.. function:: sharedarea_writelong(pos, val) -> int

   Write a 64-bit (8-byte) long into the uWSGI :doc:`SharedArea`.

   :param pos: The position to write the value to.
   :param val: The value to write.
   :type val: long
   :return: The value written, or ``None`` if the shared area is not enabled or the write request is invalid.

.. function:: sharedarea_inclong(pos) -> int

   Atomically increment a 64-bit long value in the uWSGI :doc:`SharedArea`.

   :param pos: The position of the value.
   :type val: long
   :return: The new value at the given position, or ``None`` if the shared area is not enabled or the read request is invalid.

Erlang函数
----------------

.. function:: erlang_send_message(node, process_name, message)

.. function:: erlang_register_process(process_name, callable)

.. function:: erlang_recv_message(node)

.. function:: erlang_connect(address)

   :return: File descriptor or -1 on error

.. function:: erlang_rpc(node, module, function, argument)

