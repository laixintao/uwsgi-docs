使用Linux名字空间监禁(jailing)你的应用
========================================

如果你有最新的Linux内核 (>2.6.26)，并且不是使用Itanium架构，那么你可以使用名字空间支持。

名字空间是什么？
--------------------

它们是一种从内核特定层“分离”你的进程，并且将它们分配给新的内核层的优雅的（比大多数你可能会在其他操作系统发现的jail系统更优雅）方式。

Posix系统中可用的'chroot'系统是名字空间的原始形式：一个进程看到的是一个全新的文件系统root，并且无法访问原始的。

Linux将这个概念扩展到了其他OS层 (PID, user, IPC, 网络等等。)，因此，特定的进程可以存活在一个“虚拟操作系统”中，它均有一组新的pid，一组新的用户，一个完全不共享的IPC系统 (信号量，贡献内存等等。)，专用网络接口及其自身的主机名。

uWSGI在1.9/2.0开发周期中得到了完全的名字空间支持。

clone() vs unshare()
--------------------

要把当前进程放到一个新的名字空间中，你有两个系统调用：古老的 ``clone()`` ，它将在特定的名字空间中创建一个新的进程，以及块中的新方式， ``unshare()`` ，这为当前运行的进程改变名字空间。

``clone()`` 可以被Emperor用来在新的名字空间中生成附庸（vassal）：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = fs,net,ipc,uts,pid
   
运行的每个附庸中将会有自己专用的文件系统、网络、SysV IPC和UTS视图。

.. code-block:: ini

   [uwsgi]
   unshare = ipc,uts
   ...
   
会在特定的名字空间中运行当前的实例。

一些名字空间子系统出于理智的使用，需要额外的步骤（见下文）。

支持的名字空间
--------------------

* ``fs`` -> CLONE_NEWNS，文件系统
* ``ipc`` -> CLONE_NEWIPC, sysv ipc
* ``pid`` -> CLONE_NEWPID，当与unshare()一起使用的时候，需要额外的 ``fork()`` 。使用--refork-*中任意一个选项。
* ``uts`` -> CLONE_NEWUTS，主机名
* ``net`` -> CLONE_NEWNET，来自不同名字空间的新的网络，UNIX socket仍然可用，它们是名字空间之间通信的好方式
* ``user`` -> CLONE_NEWUSER，仍然管理复杂 (并在不同的内核版本之间表现不同)，慎用

setns()
-------

除了为新的进程创建新的名字空间，你还可以使用 ``setns()`` 调用附加到已经运行的（名字空间）。

每个进程通过 ``/proc/self/ns`` 目录公开它的名字空间。setns()系统调用使用从那个目录中的文件获得的文件描述符来附加到名字空间上。

正如我们已经看到的那样，UNIX socket是名字空间之间进行通信的一种好方式，uWSGI的 ``setns()`` 特性的工作方式是创建一个UNIX socket，用来接收那些想要加入它的名字空间的进程的请求。由于UNIX socket允许传递文件描述符，因此“客户端”只需要在它们之上调用setns()。

* ``setns-socket <addr>`` exposes /proc/self/ns on the specified unix socket address
* ``setns <addr>`` connect to the specified unix socket address, get the filedescriptors and use setns() on them
* ``setns-preopen`` if enabled the /proc/self/ns files are opened on startup (before privileges drop) and cached. This is useful for avoiding running the main instance as root.
* ``setns-socket-skip <name>`` some file in /proc/self/ns can create problems (mostly the 'user' one). You can skip them specifying the name. (you can specify this option multiple times)

pivot_root
----------

This option allows you to change the rootfs of your currently running instance.

It is better than chroot as it allows you to access the old file system tree before (manually) unmounting it.

It is a bit complex to master correctly as it requires a couple of assumptions:

``pivot_root <new> <old>``

<new> is the directory to mount as the new rootfs and <old> is where to access the old tree.

<new> must be a mounted file system, and <old> must be under this file system.

A common pattern is:

.. code-block:: ini

   [uwsgi]
   unshare = fs
   hook-post-jail = mount:none /distros/precise /ns bind
   pivot_root = /ns /ns/.old_root
   ...
   
(Remember to create ``/ns`` and ``/distro/precise/.old_root``.)

When you have created the new file system layout you can umount /.old_root recursively:

.. code-block:: ini

   [uwsgi]
   unshare = fs
   hook-post-jail = mount:none /distros/precise /ns bind
   pivot_root = /ns /ns/.old_root
   ; bind mount some useful fs like /dev and /proc
   hook-as-root = mount:proc none /proc nodev hidepid=2
   hook-as-root = mount:none /.old_root/dev /dev bind
   hook-as-root = mount:none /.old_root/dev/pts /dev/pts bind
   ; umount the old tree
   hook-as-root = umount:/.old_root rec,detach


为什么不使用lxc？
------------

LXC (LinuX Containers) is a project allowing you to build full subsystems using Linux namespaces. You may ask why "reinvent the wheel" while LXC implements
a fully "virtualized" system. Apples and oranges...

LXC's objective is giving users the view of a virtual server. uWSGI namespaces support is lower level -- you can use it to detach
single components (for example you may only want to unshare IPC) to increase security and isolation.

Not all the scenario requires a full system-like view (and in lot of case is suboptimal, while in other is the best approach), try to
see namespaces as a way to increase security and isolation, when you need/can isolate a component do it with clone/unshare. When you want
to give users a full system-like access go with LXC.

老方法：--namespace选项
===================================

Before 1.9/2.0 a full featured system-like namespace support was added. It works as a chroot() on steroids.

It should be moved as an external plugin pretty soon, but will be always part of the main distribution, as it is used by lot of people
for its simplicity.

You basically need to set a root filesystem and an hostname to start your instance in a new namespace:

Let's start by creating a new root filesystem for our jail. You'll need ``debootstrap`` (or an equivalent package for your distribution).
We're placing our rootfs in ``/ns/001``, and then create a 'uwsgi' user that will run the uWSGI server.
We will use the chroot command to 'adduser' in the new rootfs, and we will install the Flask package, required by uwsgicc.

(All this needs to be executed as root)

.. code-block:: sh

    mkdir -p /ns/001
    debootstrap maverick /ns/001
    chroot /ns/001
    # in the chroot jail now
    adduser uwsgi
    apt-get install mercurial python-flask
    su - uwsgi
    # as uwsgi now
    git clone https://github.com/unbit/uwsgicc.git .
    exit # out of su - uwsgi
    exit # out of the jail
    
Now on your real system run
    
.. code-block:: sh

    uwsgi --socket 127.0.0.1:3031 --chdir /home/uwsgi/uwsgi --uid uwsgi --gid uwsgi --module uwsgicc --master --processes 4 --namespace /ns/001:mybeautifulhostname

If all goes well, uWSGI will set ``/ns/001`` as the new root filesystem, assign ``mybeautifulhostname`` as the hostname and hide the PIDs and IPC of the host system.

The first thing you should note is the uWSGI master becoming PID 1 (the "init" process) in the new namespace.
All processes generated by the uWSGI stack will be reparented to it if something goes wrong. If the master dies, all jailed processes die.

Now point your web browser to your web server and you should see the uWSGI Control Center interface.

Pay attention to the information area. The node name (used by cluster subsystem) matches the real hostname as it does not make sense to have multiple jail in the same cluster group. In the hostname field instead you will see the hostname you have set.

Another important thing is that you can see all the jail processes from your real system (they will have a different set of PIDs), so if you want to take control of the jail
you can easily do it.


.. 注意::

   A good way to limit hardware usage of jails is to combine them with the cgroups subsystem.

   .. seealso:: :doc:`Cgroups`

重载uWSGI
---------------

When running in a jail, uWSGI uses another system for reloading: it'll simply tell workers to bugger off and then exit. The parent process living outside the namespace will see this and respawn the stack in a new jail.

How secure is this sort of jailing?
-----------------------------------

Hard to say! All software tends to be secure until a hole is found.

其他文件系统
----------------------

When app is jailed to namespace it only has access to its virtual jail root filesystem. If there is any other filesystem mounted inside the jail directory, it won't be accessible, unless you use ``namespace-keep-mount``.

.. code-block:: ini

    # app1 jail is located here
    namespace = /apps/app1
    
    # nfs share mounted on the host side
    namespace-keep-mount = /apps/app1/nfs

This will bind /apps/app1/nfs to jail, so that jailed app can access it under /nfs directory

.. code-block:: ini
    
    # app1 jail is located here
    namespace = /apps/app1
    
    # nfs share mounted on the host side
    namespace-keep-mount = /mnt/nfs1:/nfs

If the filesystem that we want to bind is mounted in path not contained inside our jail, than we can use "<source>:<dest>" syntax for --namespace-keep-mount. In this case the /mnt/nfs1 will be binded to /nfs directory inside the jail.
