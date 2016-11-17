uWSGI 1.9.17
============

更新日志[20130917]


错误修复
********

- 'pty'客户端现在是阻塞的 (更安全的方法)
- 移除strtok()使用 (由strtok_r()之上的一个新的uwsgi api函数取代)
- 修复--pty-exec (关于作者：C Anthony Risinger)
- listen_queue/somaxconn linux检查现已完成，甚至是对于UNIX socket也是可用的



新特性
********

Master FIFO
^^^^^^^^^^^^^^^

这是除了UNIX信号之外的一种新的管理方式。由于我们没有更多信号可以使用了 (并且一般处理信号和pid文件并不是非常有趣)，因此uWSGI的所有新的管理特性都将基于master fifo。

文档已经有了： :doc:`MasterFIFO`


asap钩子
^^^^^^^^^^^^^

关于作者：Matthijs Kooijman

添加了一个新的名为'asap'的钩子。在解析完选项后会立即运行它。

看看： :doc:`Hooks`

TCC (libtcc)插件
^^^^^^^^^^^^^^^^^^^^^^^

TCC是一个嵌入式C编译器。它包含了一个共享库 (libtcc)，你可以用来在运行时编译C代码字符串。

libtcc uWSGI插件允许编译c字符串来处理符号。目前，已实现了"tcc"钩子引擎：

.. code-block:: ini

   [uwsgi]
   hook-asap = tcc:mkdir("/var/run/sockets");printf("directory created\n");
   hook-as-user = tcc:printf("i am process with pid %d\n", getpid());
   hook-post-app = tcc:if (getenv("DESTROY_THE_WORLD")) exit(1);
   http-socket = /var/run/sockets/foobar.sock



forkptyrouter网关
^^^^^^^^^^^^^^^^^^^^^^^^^

由于关于Linux容器/名字空间的工作正持续进行中，因此我们添加了这个特殊的路由器/网关，允许在uWSGI实例中动态分配伪终端。要访问由forkptyrouter创建的socket，你可以使用由"pty"插件公开的--pty-connect选项。

相关文档正在编写中。

添加了一个新的魔术变量用于ANSI转义
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

已添加%[魔术变量，它允许你在日志中定义ANSI序列。

如果你喜欢彩色日志：

.. code-block:: ini

   log-encoder = format %[[33m${msgnl}%[[0m
   
可路由日志编码器
^^^^^^^^^^^^^^^^^^^^^

现在，你可以附加日志编码器到指定的日志路由上了：

.. code-block:: ini

   [uwsgi]
   logger = stderr file:/dev/tty
   log-route = stderr ubuntu
   log-route = stderr clock
   print = %[[34mHELLO%[[0m
   ; add an encoder to the 'stderr' logger
   log-encoder = format:stderr %[[33m${msgnl}%[[0m
   http-socket = :9090

--vassals-include
^^^^^^^^^^^^^^^^^

关于作者：Matthijs Kooijman

这就像--vassal-inherit，但是解析会“立即”进行 (因此，你可以使用占位符)

Emperor心跳系统现在是严苛的……
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Emperor子系统对于心跳的老方法是向坏掉的vassal请求“优雅”重载。

现在，没有发送心跳（在注册到心跳子系统后）的vassal会被kill -9

这个补丁的结果是对于坏掉的vassal的管理更加健壮

logpipe
^^^^^^^

作者：INADA Naoki

现在，你可以发送日志行到一个外部命令的标准输入：

.. code-block:: ini

   req-logger = pipe:/usr/local/bin/mylogger

添加"fd"记录器到"logfile"插件中
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你可以直接发送日志到一个文件描述符：

.. code-block:: ini

   req-logger = fd:17


可用性
************

uWSGI 1.9.17于2013年09月22日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.17.tar.gz
