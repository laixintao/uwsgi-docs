管理uWSGI服务器
=========================

.. seealso::

  如果你正在管理多个应用，或者一个高容量站点，那么看一看

  * :doc:`Emperor`
  * :doc:`Zerg`
  * :doc:`SubscriptionServer`


启动服务器
-------------------

启动一个uWSGI服务器是系统管理员的角色，例如启动Web服务器。不应该由Web服务器来启动uWSGI服务器 —— 虽然如果这适合你的架构，你也能那样做。

如何在引导的时候最好的启动uWSGI服务取决于你使用的操作系统。

在现代系统中，下述应该行得通。在“传统的”操作系统上，你可以使用 ``init.d``/``rc.d`` 脚本，或者诸如Supervisor, Daemontools 或 :doc:`inetd/xinetd <Inetd>` 这样的工具。

============== =========
系统           方法
============== =========
Ubuntu         :doc:`Upstart` (the official ``uwsgi`` package, available since Ubuntu 12.04 provides an init.d based solution. Read the README.)
Debian         :doc:`Systemd`
Arch Linux     :doc:`Systemd`
Fedora         :doc:`Systemd`
OSX            launchd
Solaris        SMF
============== =========


用来控制uWSGI的信号哦
-----------------------------

你可以使用 ``safe-pidfile`` 选项，指示uWSGI将master进程的PID写入到一个文件中。

uWSGI对以下信号进行响应。

==========  ========================================================================  ===================
信号        描述                                                                      便捷命令
==========  ========================================================================  ===================
`SIGHUP`    优雅地重载所有的worker和master进程                                        ``--reload``
`SIGTERM`   粗鲁地重载所有的worker和master进程                                        (使用 ``--die-on-term`` 以尊重关闭实例的传统)
`SIGINT`    立即杀死整个uWSGI栈                                                       ``--stop``
`SIGQUIT`   立即杀死整个uWSGI栈 
`SIGUSR1`   打印统计数据
`SIGUSR2`   打印worker状态或唤醒spooler
`SIGURG`    恢复一个快照
`SIGTSTP`   暂停/挂起/恢复一个实例
`SIGWINCH`  唤醒一个在系统调用中阻塞的worker (内部使用)
`SIGFPE`    生成C回溯
`SIGSEGV`   生成C回溯
==========  ========================================================================  ===================

注意：比起使用信号，有更好的管理你的实例的方式，例如，master-fifo就是一个更健壮的方式。

重载服务器
--------------------

当运行在 ``master`` 进程模式下时，uWSGI服务器可以在无需关闭主socket的情况下优雅地重启。

这个功能允许你在无需关闭与web服务器的连接以及丢失请求的情况下修补/更新uWSGI服务器。

当你发送 `SIGHUP` 给主进程时，它会试着优雅地停止所有的worker，等待任何当前运行中的请求完成。

然后，它关闭所有与uWSGI无关的最终打开的文件描述符。

最后，它使用一个新的来二进制修补 (使用 ``execve()``) uWSGI进程镜像，继承所有之前的文件描述符。

服务器将会知道它是一个已重载的实例，并且会跳过所有的socket初始化，重用之前的。

.. note::

   发送 `SIGTERM` 信号将会获得与优雅地重载相同的结果，但将不会等待运行中的请求的完成。

有多种方式来让uWSGI优雅地重启。

.. code-block:: sh

    # using kill to send the signal
    kill -HUP `cat /tmp/project-master.pid`
    # or the convenience option --reload
    uwsgi --reload /tmp/project-master.pid
    # or if uwsgi was started with touch-reload=/tmp/somefile
    touch /tmp/somefile

或在你的应用中，使用Python:

.. code-block:: python

    uwsgi.reload()

或者使用Ruby,

.. code-block:: ruby

    UWSGI.reload

停止服务器
-------------------

如果出于某些原因，你让uWSGI进程在前台运行，那么使用CTRL+C就可以杀死它了。

在处理后台进程时，你将需要再次使用master pidfile。SIGINT信号将会杀死uWSGI.

.. code-block:: sh

    kill -INT `cat /tmp/project-master.pid`
    # or for convenience...
    uwsgi --stop /tmp/project-master.pid

Master FIFO
---------------

自uWSGI 1.9.17起，添加了一个新的管理系统，它使用unix命名管道 (fifo): :doc:`MasterFIFO`
