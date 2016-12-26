FreeBSD Jails
=============

uWSGI 1.9.16引入了原生的FreeBSD jails支持。

FreeBSD jails可以被看成新一代的chroot()，能够细粒度调整“jail”可以访问的东西。

即使在更高一点的级别上，它们都非常类似于Linux的名字空间 (从API的角度来看)。

自FreeBSD 4起，jails就可用了


为什么用uWSGI管理jails?
*******************************

一般来说，jails是用系统工具“jail”及其实用程序管理的。

时至今日，在FreeBSD jails中运行uWSGI是非常常见的，但是对于真正大规模的设置 (即，托管业务)，其中，Emperor (比方说) 管理数百个无关的uWSGI实例，设置真真会磨死人。

在uWSGI配置文件中直接管理jails极大降低了系统管理员成本，并且帮助更好的组织整个基础设施。

老式的jails (FreeBSD < 8)
*****************************

FreeBSD公开了2个主要的API，用于管理jails。老的（更早的）那个是基于jail()函数的。

它自FreeBSD 4起可用，并且允许你设置rootfs、主机名和一个或多个ipv4/ipv6地址。

在一个jail中运行一个uWSGI实例需要2个选项：--jail和--jail-ip4/--jail-ip6 (实际上是3个，如果你使用IPv6的话)

``--jail <rootfs> [hostname] [jailname]``

``--jail-ip4 <address>`` (can be specified multiple times)

``--jail-ip6 <address>`` (can be specified multiple times)

显示如何为你的jail创建rootfs并非本文档的目的，但是就我个人而言，我讨厌从源代码重新构建，因此一般而言，我只是从官方库搞到base.tgz文件，并且chroot()到它那里，以进行微调。

一个你必须记得的事情是，附加到jail的ip地址必须在系统中可用（作为别名）。与往常一样，我们往往滥用uWSGI设施。在我们的例子中，--exec-pre-jail hook将会获得成功


.. code-block:: ini

   [uwsgi]
   ; create the jail with /jails/001 as rootfs and 'foobar' as hostname
   jail = /jails/001 foobar
   ; create the alias on 'em0'
   exec-pre-jail = ifconfig em0 192.168.0.40 alias
   ; attach the alias to the jail
   jail-ip4 = 192.168.0.40
   
   ; bind the http-socket (we are now in the jail)
   http-socket = 192.168.0.40:8080
   
   ; load the application (remember we are in the jail)
   wsgi-file = myapp.wsgi
   
   ; drop privileges
   uid = kratos
   gid = kratos
   
   ; common options
   master = true
   processes = 2

新式的jails (FreeBSD >= 8)
******************************

FreeBSD 8引入了一个用于管理jails的新的高级的api。基于jail_set()系统调用，libjail公开了几十个功能，并且允许jails的微调。要使用这个新的api，你需要--jail2选项 (别名为--libjail)

``--jail2 <key>[=value]``

每个--jail2选项都和一个jail属性1：1映射，因此你基本可以调整一切！

.. code-block:: ini

   [uwsgi]
   ; create the jail with /jails/001 as rootfs
   jail2 = path=/jails/001
   ; set hostname to 'foobar'
   jail2 = host.hostname=foobar
   ; create the alias on 'em0'
   exec-pre-jail = ifconfig em0 192.168.0.40 alias
   ; attach the alias to the jail
   jail2 = ip4.addr=192.168.0.40
   
   ; bind the http-socket (we are now in the jail)
   http-socket = 192.168.0.40:8080
   
   ; load the application (remember we are in the jail)
   wsgi-file = myapp.wsgi
   
   ; drop privileges
   uid = kratos
   gid = kratos
   
   ; common options
   master = true
   processes = 2
   

关于FreeBSD >= 8.4 但是 < 9.0 的注意事项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

uWSGI在FreeBSD < 9上使用ipc信号量 (较新的FreeBSD发行版支持POSIX信号量)。

从FreeBSD 8.4起，你需要明确在jails中允许sysvipc。因此，确保这样

.. code-block:: ini

   [uwsgi]
   ...
   jail2 = allow.sysvipc=1
   ...
   
   
DevFS
*****

DevFS虚拟文件系统管理FreeBSD上的/dev目录。

/dev文件系统并没有挂载在jail中，但是出于数以百计的原因，你可能需要它。

有两个可用的主要方法：在创建jail之前，将其挂载在roots的/dev/目录中，或者允许jail挂载它


.. code-block:: ini

   [uwsgi]
   ; avoid re-mounting the file system every time
   if-not-exists = /jails/001/dev/zero
     exec-pre-jail = mount -t devfs devfs /jails/001/dev
   endif =
   ; create the jail with /jails/001 as rootfs
   jail2 = path=/jails/001
   ; set hostname to 'foobar'
   jail2 = host.hostname=foobar
   ; create the alias on 'em0'
   exec-pre-jail = ifconfig em0 192.168.0.40 alias
   ; attach the alias to the jail
   jail2 = ip4.addr=192.168.0.40
   
   ; bind the http-socket (we are now in the jail)
   http-socket = 192.168.0.40:8080
   
   ; load the application (remember we are in the jail)
   wsgi-file = myapp.wsgi
   
   ; drop privileges
   uid = kratos
   gid = kratos
   
   ; common options
   master = true
   processes = 2


或者 (允许jail自身挂载它)

.. code-block:: ini

   [uwsgi]
   ; create the jail with /jails/001 as rootfs
   jail2 = path=/jails/001
   ; set hostname to 'foobar'
   jail2 = host.hostname=foobar
   ; create the alias on 'em0'
   exec-pre-jail = ifconfig em0 192.168.0.40 alias
   ; attach the alias to the jail
   jail2 = ip4.addr=192.168.0.40
   
   ; allows mount of devfs in the jail
   jail2 = enforce_statfs=1
   jail2 = allow.mount
   jail2 = allow.mount.devfs
   ; ... and mount it
   if-not-exists = /dev/zero
     exec-post-jail = mount -t devfs devfs /dev
   endif =
   
   ; bind the http-socket (we are now in the jail)
   http-socket = 192.168.0.40:8080
   
   ; load the application (remember we are in the jail)
   wsgi-file = myapp.wsgi
   
   ; drop privileges
   uid = kratos
   gid = kratos
   
   ; common options
   master = true
   processes = 2


重载
*********

重载（或者二进制补丁）的管理有点烦人，因为uWSGI需要重新执行自身，所以在你的jail中，需要一个二进制文件、插件和配置文件的拷贝 (除非你可以牺牲掉优雅重载和简单指定Emperor来重新生成实例)

另一个方法是 (就和devfs一样) 在jail自身中，使用uwsgi二进制文件（和最终插件）挂载该目录，并且使用--binary-path来指示uWSGI使用这个新的路径


jid文件
***********

每个jail可以由一个唯一的名字 (可选的) 或者它的"jid"应用。这类似于一个"pid"，因为你可以用它来发送命令（和更新）到一个已经运行的jail。--jidfile <file>选项允许你存储jid到一个文件中，用于和外部应用使用。

附加到一个jail
*******************

你可以使用--jail-attach <id>将uWSGI实例附加到已经运行的jails（也可以是标准的持久化jail）上

id参数可以是jid或者jail的名字。

这个特性需要FreeBSD 8

Debian/kFreeBSD
***************

这是一个官方Debian项目，旨在构建一个带有FreeBSD内核和常见Debian用户空间的操作系统。

它工作良好，也支持jail。

让我们用debootstrap创建一个jail

.. code-block:: sh

   debootstrap wheezy /jails/wheezy
   
添加一个网络别名

.. code-block:: sh

   ifconfig em0 192.168.173.105 netmask 255.255.255.0 alias
   
(将em0改为你的网络接口名)

然后运行它

.. code-block:: sh

   uwsgi --http-socket 192.168.173.105:8080 --jail /jails/wheezy -jail-ip4 192.168.173.105
   

使用Forkpty Router的Jails
*************************

你可以使用 :doc:`ForkptyRouter` 轻松附加到FreeBSD jails

只是记得将/dev (嗯，/dev/ptmx) 挂载到你的jail，以允许forkpty()调用

学习如何处理devfs_ruleset以增加你的devfs的安全性


注意事项
**********

当jail中运行的最后一个进程死掉的时候，会销毁这个jail

默认情况下，所有挂载在rootfs（在进入jail之前）的东东将对jail自身可见 (前面在处理devfs的时候，我们已经看到了)
