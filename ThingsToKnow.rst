需要知道的事情 (最佳实践和“问题”) 必读！！！
========================================================

*  ``http`` 和 ``http-socket`` 选项是完全不同的。第一个生成一个额外的进程，转发请求到一系列的worker (将它想象为一种形式的盾牌，与apache或者nginx同级)，而第二个设置worker为原生使用http协议。
  TL/DR: 如果你计划直接将uWSGI公开，那么使用 ``--http`` ，如果你想要在一个使用http与后端通信的web服务器后代理它，那么使用 ``--http-socket`` 。
  .. seealso:: :doc:`HTTP`

* 直到uWSGI 2.1，默认情况下，发送 ``SIGTERM`` 信号给uWSGI意味着“粗鲁地重载栈”，然而习惯是用 ``SIGTERM`` 信号来关闭一个应用。要关闭uWSGI，请使用 ``SIGINT`` 或者 ``SIGQUIT`` 代替。
  如果你完全不能忍受uWSGI如此地不尊重 ``SIGTERM`` ，那么务必启用 ``die-on-term`` 选项。幸运的是，这个糟糕的选择已经在uWSGI 2.1修复了。

* 如果你计划托管多个应用，那么就当做是帮你自己一个忙，看看 :doc:`Emperor` 文档。

* 始终使用uwsgitop，通过 :doc:`StatsServer` 或一些类似的东东来监控你的应用健康度。

* uWSGI可以在核心部分包含特性，也可以将特性作为插件加载。由操作系统发行版提供提供的uWSGI包倾向于模块化。在这样的步骤中，确保使用 ``plugins`` 选项加载你所需的插件。识别一个未加载的插件的一个好症状是日志中像"Unavailable modifier requested"这样的消息。如果你正使用由发行版提供的包，那么仔细检查你是否已经安装了用于你所选的语言的插件。

* 配置文件支持一种有限形式的继承、变量、if构造和简单的循环。看看 :doc:`ConfigLogic` 和 :doc:`ParsingOrder` 页面。

* 要路由请求到一个指定的插件，web服务器需要传递一个称为modifier的魔术数字到uWSGI实例。默认情况下，这个数字被设置为0，这会映射到Python。例如，路由一个请求到一个 :doc:`PSGI` 应用要求你设置modifier为 ``5`` - 或者可选地加载PSGI插件作为modifier ``0`` 。(这将意味着，所有的无modifier的请求将会被认为是Perl的。)

* 没有设置要使用的进程数或线程数的魔法规则。它是灰常依赖于应用和系统的。简单的算术，例如 ``processes = 2 * cpucores`` ，并不够。你需要对多种步骤进行实验，并且准备好不断监控当你的应用。 ``uwsgitop`` 会是一个帮你找到最佳值的好工具。

* 如果一个HTTP请求有请求体 (例如由表单生成的POST请求)，那么你 *必须* 在你的应用中读取 (消费) 它。如果你不这么做，那么通过你的web服务器的通信套接字可能会被重写。如果你懒癌犯了，那么你可以使用 ``post-buffering`` 选项，它会自动地为你读取数据。对于 :doc:`Rack` 应用，这是自动启用的。

* 经常检查你的应用的内存使用。 ``memory-report`` 选项会是你最好的伙伴。

* 如果你计划使用UNIX socket (与TCP相对)，那么记住，它们是标准的文件系统对象。这意味着，它们有权限，并且同样地，你的web服务器必须有对它们的写权限。

* 常识：不要以root用户运行uWSGI实例。你可以作为root启动你的uWSGI们，但是确保使用 ``uid`` 和 ``gid`` 选项来移除权限。

* uWSGI试图在一切可能的情况下(滥)用 `fork() <http://en.wikipedia.org/wiki/Fork_%28operating_system%29>`_ 调用的写时拷贝语义。默认情况下，它会在加载你的应用之后进行fork，以尽可能的共享它们的内存。如果出于某些原因，不期望这个行为，那么使用 ``lazy-apps`` 选项。这将会指示uWSGI在每个worker的 ``fork()`` 之后加载应用。注意，有一个更老的名为 ``lazy`` 的选项，它是一种更侵入以及极度不推荐的方式 (它仍然能用，只是为了向后兼容)

* 默认情况下，Python插件并不初始化GIL。这意味着你由应用生成的线程并不会运行。如果你需要线程，记得通过 ``enable-threads`` 启用它们。在多线程模式（使用 ``threads`` 选项）下运行uWSGI将会自动启用线程支持。这种“奇怪的”默认行为是出于性能考量的，这并不可耻。

* 如果你在请求期间生成一个新的进程，那么它户继承生成它的worker的文件描述符 - 包括连接到web服务器/路由器的socket。如果你并不想要这种行为，那么设置 ``close-on-exec`` 选项。

* Ruby垃圾回收器是默认配置的，它会在每个请求之后运行。这是一个激进的策略，可能会减缓一点你的应用 —— 但是CPU资源比内存便宜，特别是比耗尽内存便宜。要调整这个评论，则使用 ``ruby-gc <freq>`` 选项。

* 在OpenBSD, NetBSD和FreeBSD < 9上，SysV IPC信号量被用作锁子系统。这些操作系统倾向于限制可分配信号量的数目为一个非常小的值。如果你计划运行超过1个uWSGI实例，那么你应该提高默认限制。FreeBSD 9有POSIX信号量，所以你无需为它费心。

* 不要使用与构建uWSGI二进制文件本身的配置文件不同的配置文件来构建插件 —— 除非你喜欢遭罪，或者 *确切* 知道你在干什么。

* 默认情况下，uWSGI为每个请求头分配一个非常小的缓存 (4096字节)。如果你的日志中开始收到"invalid request block size"，这可能意味着你需要更大的缓存了。通过``buffer-size`` 选项来增加它（最大值为65535）。

  .. note::

     如果你的日志中接收到了请求块大小为'21573'，那这可能意味着你使用HTTP协议与一个使用uwsgi协议的实例进行通信。不要这样做。

* 如果你的 (Linux) 服务器似乎有大量的idle状态的worker，但性能仍然不佳，那么你或许想要看看 ``ip_conntrack_max`` 系统变量 (``/proc/sys/net/ipv4/ip_conntrack_max``) 的值，然后增加它以看看是否有帮助。

* 一些Linux发行版 (即，Debian 4 Etch, RHEL / CentOS 5)混合了较新的内核和非常老的用户空间。这种组合会让uWSGI构建系统显示错误 (在 ``unshare()`` , pthread locking, ``inotify``...之上最值得注意)。你可以给'make' (或者任何你用来构建它的方式) 加上前缀 ``CFLAGS="-DOBSOLETE_LINUX_KERNEL"`` ，强制uWSGI对较老的系统进行配置

* 默认情况下，在uWSGI启动的适合，stdin被重新映射到 ``/dev/null`` 。如果你需要一个有效的stdin (用于调试、管道等等)，那么添加 ``--honour-stdin`` 。

* 你可以轻松地添加不存在的选项到你的配置文件中 (例如占位符、自定义选项或者应用相关的配置项)。这是一个非常方便的特性，但一旦有错别字，那就头疼了。strict模式 (``--strict``) 将禁用此特性，并且只允许有效的uWSGI选项。

* 一些插件 (最明显的是Python和Perl) 具有代码自动重载机制。虽然这也许听起来很诱人，但是你必须只在开发阶段使用它们，因为它们真的很重。例如，Python的--py-autoreload选项将会在每个检查周期中扫描你整个模块树。

* ``wsgi.file_wrapper`` 是WSGI标准的一个优化。在个别情况下，它会引发错误。例如，当在Python 3.5中返回一个内存中字节缓存 (`io.Bytesio <https://docs.python.org/3/library/io.html#io.BytesIO>`_) 的时候。看看这个 `issue <https://github.com/unbit/uwsgi/issues/1126>`_ 。你可以通过设置选项 ``wsgi-disable-file-wrapper`` 为 ``true`` 来禁用它。
