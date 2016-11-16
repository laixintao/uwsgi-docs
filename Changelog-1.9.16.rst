uWSGI 1.9.16
============

更新日志[20130914]


gevent插件的关闭／重载过程的重要改动！！！
*******************************************************************

gevent模式下的关闭／重载过程已经被修改以更好地与多线程（和multigreenlet）环境(最明显的是newrelic代理)进行集成。

与"加入"gevent hub相反，会生成和“加入”一个新的“虚拟”greenlet。

在关闭期间，只有由uWSGI生成的greenlet才会被考虑到，而在它们所有都被销毁之后，进程将会退出。这与老方法不同，后者会等待所有当前可用的greenlet（和猴子不定线程）。

如果你更喜欢旧的行为，那么仅需指定选项--gevent-wait-for-hub 


错误修复/改进
*********************

- 修复CPython在rpc和信号处理器中的引用计数错误
- 对QUERY_STRING,SCRIPT_NAME,SERVER_NAME和SERVER_PORT遵守Rack规格
- 报告缺少内部路由支持 (当libpcre缺失的时候，只是警告)
- 关闭和zerg模式期间更好的ipcsem支持 (添加--persistent-ipcsem作为特例)
- 修复apache mod_fastcgi公开的fastcgi错误
- 重载时不调用pre-jail钩子
- 强制在solaris上使用-lrt进行链接
- 报告thunder锁状态
- 允许在rsyslog插件中自定义优先级

新特性
********

FreeBSD jails原生支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

uWSGI有原生FreeBSD jails支持。官方文档在这里： :doc:`FreeBSDJails`

Rados插件
^^^^^^^^^^^^^^^^

作者：Javier Guerra

基于 :doc:`GlusterFS` 插件，推出了一个新的允许访问Rados对象存储的插件：

 :doc:`Rados`

TunTap路由器
^^^^^^^^^^^^^^^^^

这个新的网关时尝试使用Linux名字空间构建更好（也可以说是：稳定）的基础设施的过程中，大量让人头疼的问题的产物。

在处理uts, ipc, pid和文件系统的时候，名字空间是相当方便的，而管理网络真心痛苦。

我们在uWSGI 1.9.15引入了大量的措施 (特别是简化veth管理)，但是最后，我们意识到那些系统无法在管理方面扩展。

这个TunTap路由器试图解决在用户空间中移动jailed vassal的网络部分的问题。

基本上，每个vassal创建一个或多个tuntap设备。这些设备被连接（通过unix socket）到"tuntap路由器"，允许从vassal到外部网络的访问。

这意味着，在主名字空间中，有单个网络接口，而对于每个vassal，有一个网络接口。

性能已经是非常棒的了 (涉及到内核层次的路由，我们只损失了大约10%)，但还是可以优化。

除此之外，tuntap路由器有一个简单的用户空间防火墙，你可以用来管理复杂的路由规则。

文档还在编写中，但是你可以按照这个文件顶部的大大的注释来配置一个tuntap路由器：

https://github.com/unbit/uwsgi/blob/master/plugins/tuntap/tuntap.c

你可以用 ``--tuntap-device <dev> <socket>`` 来连接到它，其中，<dev>要在vassal/客户端中创建的tuntap设备，而<socket>是tuntap路由器的unix地址

一个样例Emperor

.. code-block:: ini

   [uwsgi]
   tuntap-router = emperor0 /tmp/tuntap.socket
   exec-as-root = ifconfig emperor0 192.168.0.1 netmask 255.255.255.0 up
   exec-as-root = iptables -t nat -F
   exec-as-root = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   exec-as-root = echo 1 >/proc/sys/net/ipv4/ip_forward
   emperor-use-clone = ipc,uts,fs,pid,net
   emperor = /etc/vassals

及其vassal之一：

.. code-block:: ini

   [uwsgi]
   tuntap-device = uwsgi0 /tmp/tuntap.socket
   exec-as-root = ifconfig lo up
   exec-as-root = ifconfig uwsgi0 192.168.0.2 netmask 255.255.255.0 up
   exec-as-root = route add default gw 192.168.0.1
   exec-as-root = hostname foobar
   socket = /var/www/foobar.socket
   psgi-file = foobar.pl

Linux O_TMPFILE
^^^^^^^^^^^^^^^

最新的Linux内核一种新的操作模式来打开文件：O_TMPFILE

这个标志在没有任何形式的竞争条件下打开一个临时文件 (也就是说：未链接)。

这个模式在可用的时候会自动使用 (无需任何选项)

Linux pivot-root
^^^^^^^^^^^^^^^^

在处理Linux名字空间时，修改根文件系统时其中一个主要任务。

chroot()一般来讲太简单了，而pivot-root允许你进行更多高级设置

语法是 ``--pivot-root <new_root> <old_root>``

Cheaper memlimit
^^^^^^^^^^^^^^^^

作者：Łukasz Mierzwa

这项新的检查允许控制基于RSS使用的动态进程生成：

https://uwsgi-docs.readthedocs.io/en/latest/Cheaper.html#setting-memory-limits

日志编码器
^^^^^^^^^^^^

时下，有几十个日志引擎和存储系统。原始的uWSGI方法是为每个引擎开发一个插件。

在使用logstash和fluentd的时候，我们意识到，大多数的日志插件是一次又一次地队相同概念的重新实现。

我们遵循一个甚至更加模块化的方法，引入日志编码器：

:doc:`LogEncoders`

它们基本上是你可以应用到每个日志行上到模式。

新的“高级”钩子
^^^^^^^^^^^^^^^^^^^^

有一系列开发者仅需对uWSGI核心进行少量修改的新的钩子可以用了。

关于整个钩子子系统的文档现已提供 (这是一项正在进行的工作):

:doc:`Hooks`

新的mount/umount钩子
^^^^^^^^^^^^^^^^^^^^^^

当处理名字空间和jails的时候，挂载和取消挂载文件系统是最常见的任务之一。

由于在设置阶段，mount和umount命令不可用，因此添加了这两个钩子，直接调用系统调用。

看看 :doc:`Hooks`


可用性
************

uWSGI 1.9.16于2013年09月14日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.16.tar.gz
