Things to know (best practices and "issues") 必读！！！
========================================================

*  ``http`` 和 ``http-socket`` 选项是完全不同的options are entirely different beasts.
  The first one spawns an additional process forwarding requests to a series of workers (think about it as a form of shield, at the same level of apache or nginx), while the second one sets workers to natively speak the http protocol.
  TL/DR: if you plan to expose uWSGI directly to the public, use ``--http``, if you want to proxy it behind a webserver speaking http with backends, use ``--http-socket``.
  .. seealso:: :doc:`HTTP`

* 直到uWSGI 2.1，默认情况下，发送 ``SIGTERM`` 信号给uWSGI意味着“粗鲁地重载栈”，然而习惯是用 ``SIGTERM`` 信号来关闭一个应用。要关闭uWSGI，请使用 ``SIGINT`` 或者 ``SIGQUIT`` 代替。
  如果你完全不能忍受uWSGI如此地不尊重 ``SIGTERM`` ，那么务必启用 ``die-on-term`` 选项。幸运的是，这个糟糕的选择已经在uWSGI 2.1修复了。

* 如果你计划托管多个应用，那么就当做是帮你自己一个忙，看看 :doc:`Emperor` 文档。

* 始终使用uwsgitop，通过 :doc:`StatsServer` 或一些类似的东东来监控你的应用健康度。

* uWSGI可以在核心部分包含特性，也可以将特性作为插件加载。由操作系统发行版提供提供的uWSGI包倾向于模块化。在这样的步骤中，确保使用 ``plugins`` 选项加载你所需的插件。识别一个未加载的插件的一个好症状是日志中像"Unavailable modifier requested"这样的消息。如果你正使用由发行版提供的包，那么仔细检查你是否已经安装了用于你所选的语言的插件。

* 配置文件支持一种有限形式的继承、变量、if构造和简单的循环。看看 :doc:`ConfigLogic` 和 :doc:`ParsingOrder` 页面。

* To route requests to a specific plugin, the webserver needs to pass a magic number known as a modifier to the uWSGI instances. By default this number is set to 0, which is mapped to Python. As an example, routing a request to a :doc:`PSGI` app requires you to set the modifier to ``5`` - or optionally to load the PSGI plugin as modifier ``0``. (This will mean that all modifierless requests will be considered Perl.)

* 没有设置要使用的进程数或线程数的魔法规则。它是灰常依赖于应用和系统的。简单的算术，例如 ``processes = 2 * cpucores`` ，并不够。你需要对多种步骤进行实验，并且准备好不断监控当你的应用。 ``uwsgitop`` 会是一个帮你找到最佳值的好工具。

* 如果一个HTTP请求有请求体 (例如由表单生成的POST请求)，那么你 *必须* 在你的应用中读取 (消费) 它。如果你不这么做，那么通过你的web服务器的通信套接字可能会被重写。如果你懒癌犯了，那么你可以使用 ``post-buffering`` 选项，它会自动地为你读取数据。对于 :doc:`Rack` 应用，这是自动启用的。

* 经常检查你的应用的内存使用。 ``memory-report`` 选项会是你最好的伙伴。

* 如果你计划使用UNIX socket (与TCP相对)，那么记住，它们是标准的文件系统对象。这意味着，它们有权限，并且同样地，你的web服务器必须有对它们的写权限。

* 常识：不要以root用户运行uWSGI实例。你可以作为root启动你的uWSGI们，但是确保使用 ``uid`` 和 ``gid`` 选项来移除权限。

* uWSGI tries to (ab)use the Copy On Write semantics of the `fork() <http://en.wikipedia.org/wiki/Fork_%28operating_system%29>`_ call whenever possible. By default it will fork after having loaded your applications to share as much of their memory as possible. If this behavior is undesirable for some reason, use the ``lazy-apps`` option. This will instruct uWSGI to load the applications after each worker's ``fork()``. Beware as there is an older options named ``lazy`` that is way more invasive and highly discouraged (it is still here only for backward compatibility)

* 默认情况下，Python插件并不初始化GIL。这意味着你由应用生成的线程并不会运行。如果你需要线程，记得通过 ``enable-threads`` 启用它们。在多线程模式（使用 ``threads`` 选项）下运行uWSGI将会自动启用线程支持。这种“奇怪的”默认行为是出于性能考量的，这并不可耻。

* 如果你在请求期间生成一个新的进程，那么它户继承生成它的worker的文件描述符 - 包括连接到web服务器/路由器的socket。如果你并不想要这种行为，那么设置 ``close-on-exec`` 选项。

* The Ruby garbage collector is configured by default to run after every request. This is an aggressive policy that may slow down your apps a bit -- but CPU resources are cheaper than memory, and especially cheaper than running out of memory. To tune this frequency use the ``ruby-gc <freq>`` option.

* On OpenBSD, NetBSD and FreeBSD < 9, SysV IPC semaphores are used as the locking subsystem. These operating systems tend to limit the number of allocable semaphores to fairly small values. You should raise the default limits if you plan to run more than one uWSGI instance. FreeBSD 9 has POSIX semaphores, so you do not need to bother with that.

* Do not build plugins using a different config file than used to build the uWSGI binary itself -- unless you like pain or know *exactly* what you are doing.

* 默认情况下，uWSGI为每个请求头分配一个非常小的缓存 (4096字节)。如果你的日志中开始收到"invalid request block size"，这可能意味着你需要更大的缓存了。通过``buffer-size`` 选项来增加它（最大值为65535）。

  .. note::

     If you receive '21573' as the request block size in your logs, it could mean you are using the HTTP protocol to speak with an instance speaking the uwsgi protocol. Don't do this.

* 如果你的 (Linux) 服务器似乎有大量的idle状态的worker，但性能仍然不佳，那么你或许想要看看 ``ip_conntrack_max`` 系统变量 (``/proc/sys/net/ipv4/ip_conntrack_max``) 的值，然后增加它以看看是否有帮助。

* Some Linux distributions (read: Debian 4 Etch, RHEL / CentOS 5) make a mix of newer kernels with very old userspace. This kind of combination can make the uWSGI build system spit out errors (most notably on ``unshare()``, pthread locking, ``inotify``...). You can force uWSGI to configure itself for an older system prefixing the 'make' (or whatever way you use to build it) with ``CFLAGS="-DOBSOLETE_LINUX_KERNEL"``

* 默认情况下，在uWSGI启动的适合，stdin被重新映射到 ``/dev/null`` 。如果你需要一个有效的stdin (用于调试、管道等等)，那么添加 ``--honour-stdin`` 。

* 你可以轻松地添加不存在的选项到你的配置文件中 (例如占位符、自定义选项或者应用相关的配置项)。这是一个非常方便的特性，但一旦有错别字，那就头疼了。strict模式 (``--strict``) 将禁用此特性，并且只允许有效的uWSGI选项。

* 一些插件 (最明显的是Python和Perl) 具有代码自动重载机制。虽然这也许听起来很诱人，但是你必须只在开发阶段使用它们，因为它们真的很重。例如，Python的--py-autoreload选项将会在每个检查周期中扫描你整个模块树。

* ``wsgi.file_wrapper`` 是WSGI标准的一个优化。在个别情况下，它会引发错误。例如，当在Python 3.5中返回一个内存中字节缓存 (`io.Bytesio <https://docs.python.org/3/library/io.html#io.BytesIO>`_) 的时候。看看这个 `issue <https://github.com/unbit/uwsgi/issues/1126>`_ 。你可以通过设置选项 ``wsgi-disable-file-wrapper`` 为 ``true`` 来禁用它。
