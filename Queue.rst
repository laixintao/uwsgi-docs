uWSGI队列框架
=========================

除了 :doc:`caching framework <Caching>` ，uWSGI还有一个共享队列。

在低层次，它是一个简单的基于块的共享数组，有两个可选的计数器，一个用于对于堆栈式，LIFO，另一个用于FIFO。

数组是环形的，因此，当两个指针的任意一个到达了尾部（或者首部），它会被重置。记住这点！

要启用队列，则使用 ``queue`` 选项。默认情况下，队列块是8KiB。使用 ``queue-blocksize`` 来修改其大小。

.. code-block:: sh

    # 100 slots, 8 KiB of data each
    uwsgi --socket :3031 --queue 100
    # 42 slots, 128 KiB of data each
    uwsgi --socket :3031 --queue 42 --queue-blocksize 131072

将队列当成共享数组使用
---------------------------------

.. code-block:: py

    # Put a binary string in slot 17.
    uwsgi.queue_set(17, "Hello, uWSGI queue!")
    
    # Get it back.
    print uwsgi.queue_get(17)


将队列当成共享堆栈使用
---------------------------------

.. warning::记住， :py:meth:`uwsgi.queue_pop` 和 :py:meth:`uwsgi.queue_last` 将会从队列中移除项。

.. code-block:: py

    # Push a value onto the end of the stack.
    uwsgi.queue_push("Hello, uWSGI stack!")
    
    # Pop it back
    print uwsgi.queue_pop()

    # Get the number of the next available slot in the stack
    print uwsgi.queue_slot()
    
    # Pop the last N items from the stack
    items = uwsgi.queue_last(3)

将队列当成一个FIFO队列使用
-------------------------------

.. note:: 当前，你只能pull，不能push。要入队一个元素，请使用 :py:meth:`uwsgi.queue_set` 。

.. code-block:: py

    # Grab an item from the queue
    uwsgi.queue_pull()
    # Get the current pull/slot position (this is independent from the stack-based one)
    print uwsgi.queue_pull_slot()

注释
-----

* 你可以通过使用 :py:data:`uwsgi.queue_size` 获取队列大小。
* 使用 ``queue-store`` 选项将队列在磁盘上持久化。使用 ``queue-store-sync`` (在master循环中 —— 通常是秒) 来强制磁盘同步队列。
* ``tests/queue.py`` 应用是一个完整的可用例子。