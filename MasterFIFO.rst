Master FIFO
===============

自uWSGI 1.9.17起可用。

一般来说，使用UNIX信号来管理maser，但是我们正在用尽信号数，并且（更重要的是）不需要跟PID混在一起回极大地简化外部管理脚本的实现。

因此，取代信号，你可以告诉master创建一个UNIX命名管道 (FIFO)，这样，你刻意用它来发布命令给master。

要创建一个FIFO，仅需添加 ``--master-fifo <filename>`` ，然后开始发布命令给它。

.. code-block:: sh

   echo r > /tmp/yourfifo
   
你可以一次性发送多个命令。

.. code-block:: sh

   # add 3 workers and print stats
   echo +++s > /tmp/yourfifo

可用命令
******************

* '0' to '9' - 设置fifo slot (见下文)
* '+' - 在cheaper模式的时候增加worker数 (添加 ``--cheaper-algo manual`` 以获得完全控制)
* '-' - 在cheaper模式的时候减少worker数 (添加 ``--cheaper-algo manual`` 以获得完全控制)
* 'B' - 让Emperor进行加固 (broodlord模式，要求uWSGI >= 2.0.7)
* 'C' - 设置cheap模式
* 'c' - 出发链重载
* 'E' - 触发一次Emperor重新扫描
* 'f' - 重新fork master (危险，但很强大)
* 'l' - 重新打开日志文件 (需要 --log-master 和 --logto/--logto2)
* 'L' - 触发日志循环 (需要 --log-master 和 --logto/--logto2)
* 'p' - 暂定/恢复实例
* 'P' - 更新pidfile (在master重新fork之后会非常有用)
* 'Q' - 粗鲁地关闭实例
* 'q' - 优雅地关闭实例
* 'R' - 发送粗鲁重载
* 'r' - 发送优雅重载
* 'S' - 阻塞/消除阻塞订阅
* 's' - 在日志中打印统计信息
* 'W' - 粗鲁地重载worker
* 'w' - 优雅地重载worker

FIFO slot
**********

uWSGI支持多至10个不同的FIFO文件。默认情况下，绑定第一个指定的（映射为'0'）。

在实例的生命周期中，你可以通过简单发送要使用的FIFO slot的数字来从一个FIFO改成另一个。

.. code-block:: ini

   [uwsgi]
   master-fifo = /tmp/fifo0
   master-fifo = /tmp/fifo1
   master-fifo = /var/run/foofifo
   processes = 2
   ...

默认情况下，会分配 ``/tmp/fifo0`` ，但在发送以下命令之后：

.. code-block:: sh

   echo 1 > /tmp/fifo0
   
将会绑定 ``/tmp/fifo1`` 文件。

当你（滥）用“fork master”命令（'f'）时，映射FIFO文件到特定的实例是非常有用的。

.. code-block:: sh

   echo 1fp > /tmp/fifo0
   
在发送这个命令后，一个新的uWSGI实例(继承所有的绑定socket)将会生成，而老的将会被置为"暂停"模式('p'命令)。

如果我们在'f'和'p'之前已经发送了'1'命令，那么老的实例现在将会接收在/tmp/fifo1 (slot 1)的命令，而新的将会使用默认的('0')。

你可以做到很多技巧，也有很多方法滥用master的fork。

只需考虑到，极端情况可能会发生在任何地方，特别是如果你使用uWSGI最复杂的功能。

小抄
*****

* FIFO是创建于非阻塞模式，并且在每次客户端断连的时候由master重新创建。
* 你可以通过插件或者C钩子，使用全局数组 ``uwsgi_fifo_table`` 来覆盖（或添加）命令。
* 只有运行master的uid才有fifo的写权限。
