uGreen -- uWSGI Green线程
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

To ensure (relative) isolation of green threads, every stack area is protected by so called "guard pages".

An attempt to write out of the stack area of a green thread will result in a segmentation fault/bus error (and the process manager, if enabled, will respawn the worker without too much damage).

The context switch is very fast, we can see it as:

* On switch
  
  1. Save the Python Frame pointer
  2. Save the recursion depth of the Python environment (it is simply an int)
  3. Switch to the main stack

* On return

  1. Re-set the uGreen stack
  2. Re-set the recursion depth
  3. Re-set the frame pointer

The stack/registers switch is done by the POSIX ``swapcontext()`` call and we don't have to worry about it.


异步I/O
---------

For managing async I/O you can use the Async mode FD wait functions :py:func:`uwsgi.wait_fd_read` and :py:func:`uwsgi.wait_fd_write`.

栈大小
----------

You can choose the uGreen stack size using the ``ugreen-stacksize <pages>`` option. The argument is in pages, not bytes.

Is this better than Greenlet or Stackless Python?
-------------------------------------------------

Weeeeelll... it depends. uGreen is faster (the stack is preallocated) but requires more memory (to allocate a stack area for every core). Stackless and Greenlet probably require less memory... but Stackless requires a heavily patched version of Python. 

If you're heavily invested in making your app as async-snappy as possible, it's always best to do some tests to choose the best one for you. As far as uWSGI is concerned, you can move from async engine to another without changing your code.

What about ``python-coev``?
---------------------------

Lots of uGreen has been inspired by it. The author's way to map Python threads to their implementation allows ``python-coev`` to be a little more "trustworthy" than Stackless Python. However, like Stackless, it requires a patched version of Python... :(

我可以使用uGreen来写Comet应用吗？
-------------------------------------

Yeah! Sure! Go ahead. In the distribution you will find the ``ugreenchat.py`` script. It is a simple/dumb multiuser Comet chat. If you want to test it (for example 30 users) run it with

.. code-block:: sh

  ./uwsgi -s :3031 -w ugreenchat --async 30 --ugreen

The code has comments for every ugreen-related line. You'll need `Bottle`_, an amazing Python web micro framework to use it.

.. _Bottle: http://bottlepy.org/docs/dev/

Psycopg2改进
---------------------

uGreen can benefit from the new psycopg2 async extensions and the psycogreen project. 见 :file:`tests/psycopg2_green.py` 和 :file:`tests/psycogreen_green.py` 文件获得样例信息。