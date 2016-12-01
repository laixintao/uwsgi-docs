uGreen -- uWSGI绿色线程（green thread）
=============================

uGreen是在 :doc:`uWSGI async platform<Async>` 之上的一个 `green threads`_ 实现。

它与Python的greenlet非常相似，但是是构建在POSIX的 ``swapcontext()`` 函数之上的。要利用uGreen，你必须设置将会映射到green线程的异步核心数。

例如，如果你想要生成30个green线程：

.. code-block:: sh

  ./uwsgi -w tests.cpubound_green -s :3031 --async 30 --ugreen

``ugreen`` 选项将会在async模式之上启用uGreen。

现在，当你在应用中调用 :py:func:`uwsgi.suspend` 时，你将会被切换到另一个green线程。

.. _green threads: http://en.wikipedia.org/wiki/Green_threads

安全性和性能
------------------------

为确保绿色线程的（相对）隔离，每个堆栈区都由所谓的“保护页”保护。

在绿色线程的堆栈区外尝试写入将会导致一个分段错误/总线错误 (而进程管理器，如果启用的话，将会在无太多损害的情况下重新生成worker)。

上下文切换非常快，我们可以将其视为：

* 转换时
  
  1. 保存Python帧指针
  2. 保存Python环境的递归深度 (它只是一个int)
  3. 切换到主堆栈

* 返回时

  1. 重置uGreen堆栈
  2. 重置递归深度
  3. 重置帧指针

堆栈/寄存器开关切换是由POSIX的 ``swapcontext()`` 调用完成的，我们无需担心它。


异步I/O
---------

要管理异步I/O，你可以使用异步模式FD等待函数 :py:func:`uwsgi.wait_fd_read` 和 :py:func:`uwsgi.wait_fd_write` 。

栈大小
----------

你可以使用 ``ugreen-stacksize <pages>`` 选项选择uGreen堆栈大小。参数是以页为单位的，不是字节。

它比Greenlet或者Stackless Python更好吗？
-------------------------------------------------

嗯……看情况。uGreen更快 (堆栈是预先分配的)，但需要更多内存 (为每个核分配一个堆栈区域)。而Stackless和Greenlet也许需要较少的内存……但是Stackless要求使用Python的一个重度打补丁版本。

如果你正投入巨大精力让你的应用约异步越好，那么做一些测试来选择最好的一个总是最好的。至于uWSGI，你可以在无需修改代码的情况下从异步引擎移到到另一个。

 ``python-coev`` 又如何？
---------------------------

uGreen的许多部分是受其启发的。作者将Python线程映射到它们的实现的方式允许 ``python-coev`` 比Stackless Python更“值得信赖”点。然而，正如Stackless，它要求Python的一个补丁版本…… :(

我可以使用uGreen来写Comet应用吗？
-------------------------------------

是哒！当然！前进吧。在发行版中，你将会找到 ``ugreenchat.py`` 脚本。它是一个简单的/基本的多用户Comet聊天应用。如果你想要测试它 (例如，30个用户)，那么这样运行它

.. code-block:: sh

  ./uwsgi -s :3031 -w ugreenchat --async 30 --ugreen

对于每一行ugreen相关的代码，都有注释。你会需要 `Bottle`_ ，一个惊人的Python web微框架来使用它。

.. _Bottle: http://bottlepy.org/docs/dev/

Psycopg2改进
---------------------

uGreen可以从新的psycopg2异步扩展和psycogreen项目受益。见 :file:`tests/psycopg2_green.py` 和 :file:`tests/psycogreen_green.py` 文件获得样例信息。