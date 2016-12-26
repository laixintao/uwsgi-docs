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

* ``setns-socket <addr>`` 公开了指定unix socket地址上的/proc/self/ns 
* ``setns <addr>`` 连接到指定的unix socket地址，获取文件描述符，并在它们上面使用setns()
* ``setns-preopen`` 如果启用了，那么会在启动的时候（移除特权之前）打开并缓存/proc/self/ns文件。这对于避免作为root运行主要实例是有用的。
* ``setns-socket-skip <name>`` /proc/self/ns中的一些文件会制造问题（大多数是'user'那个）。你可以指定名字来跳过它们. (你可以多次指定这个选项)

pivot_root
----------

这个选项允许你的修改你当前运行实例的rootfs。

它比chroot好，因为它允许你在（手工）取消挂载老的文件系统树之前访问它。

要正确掌控它会有点复杂，因为它需要几个假设：

``pivot_root <new> <old>``

<new>是要作为新的rootfs挂载的目录，而<old>是访问旧的文件系统树的路径。

<new>必须是一个已挂载的文件系统，而<old>必须在这个文件系统之下。

一个常见的模式是：

.. code-block:: ini

   [uwsgi]
   unshare = fs
   hook-post-jail = mount:none /distros/precise /ns bind
   pivot_root = /ns /ns/.old_root
   ...
   
(记得创建 ``/ns`` 和 ``/distro/precise/.old_root`` 。)

当你创建了新的文件系统布局，你就可以递归取消挂载/.old_root：

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
----------------

LXC (LinuX Containers)是一个允许你使用Linux名字空间构建完整子系统的项目。你可能会问，既然LXC实现了一个完整的“虚拟”系统，那么为什么还要“另起炉灶”。风马牛不相及……

LXC的目标是提供用户一个虚拟服务器视图。uWSGI的名字空间支持更低层次 —— 你可以用它来分离单个部件 (例如，你也许只想要取消共享IPC)，从而提高安全性和隔离性。

并非所有的场景都要求一个完整的类系统视图 (在大量的场景下，是次优的，而在其他场景下则是最佳方案)，试着把名字空间当成一种提高安全性和隔离性的方法，当你需要/可以分离一个组件时，使用克隆/取消共享来完成。当你想要让用户访问一个完整的类系统时，使用LXC。

老方法：--namespace选项
===================================

在1.9/2.0之前，添加了一个全功能的类系统名字空间支持。它的工作方式就像增强型chroot()。

很快，它就应该被移动成为一个外部的插件，但是将总会是主要发行的一部分，因为由于它的简单，很多人用它。

基本上，你需要设置一个根文件系统和一个主机名，在一个新的名字空间中启动你的实例：

让我们以为我们的jail创建一个新的根文件系统开始。你会需要 ``debootstrap`` (或者用于你的发行版的等价包)。我们将rootfs放在 ``/ns/001`` ，然后创建一个'uwsgi'用户，这个用户会运行uWSGI服务器。我们会使用chroot命令来在新的rootfs中'adduser'，并且我们会安装Flask包，这是uwsgicc要求的。

(所有这些都必须以root执行)

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
    
现在，在你真实系统上，运行
    
.. code-block:: sh

    uwsgi --socket 127.0.0.1:3031 --chdir /home/uwsgi/uwsgi --uid uwsgi --gid uwsgi --module uwsgicc --master --processes 4 --namespace /ns/001:mybeautifulhostname

如果一切顺利，uWSGI将会设置 ``/ns/001`` 作为新的root文件系统，分配 ``mybeautifulhostname`` 作为主机名，并且隐藏主机系统的PID和IPC。

你应该注意的第一件事是，在新的名字空间中，uWSGI master变成了PID 1 ("init"进程)。如果出现错误，那么所有由该uWSGI栈生成的进程将会重新更改父亲为这个master。如果master死掉了，那么所有被jail的进程都会死掉。

现在，把你的web浏览器指向你的web服务器，你应该会看到uWSGI Control Center界面。

注意下信息区域。节点名 (为集群子系统所用) 匹配真实的主机名，因为在同一个集群组里拥有多个jail并无意义。相反，在主机名字段，你会看到你设置的主机名。

另一个重点是，你可以看到来自你真实系统的所有jail进程（它们将会拥有一个不同的PID集合），英寸，如果你想要控制jail，那么可以容易做到。


.. note::

   限制jails的硬件使用的好方法是将它们与cgroups子系统结合在一起。

   .. seealso:: :doc:`Cgroups`

重载uWSGI
---------------

当运行在一个jail的时候，uWSGI使用另一个系统来重载：它会简单告诉worker开溜，然后退出。存活在名字空间外的父进程将会看到，并在一个新的jail中重新生成栈。

这种jail有多安全？
-----------------------------------

很难说！所有的软件在发现漏洞之前都倾向于安全的。

其他文件系统
----------------------

当把应用困在名字空间内时，它只能访问它的虚拟jail根文件系统。如果在jail目录中，有其他挂载着的文件系统，那么它们将不能被访问，除非你使用 ``namespace-keep-mount`` 。

.. code-block:: ini

    # app1 jail is located here
    namespace = /apps/app1
    
    # nfs share mounted on the host side
    namespace-keep-mount = /apps/app1/nfs

这将会绑定/apps/app1/nfs到jail，因此，被困住的应用可以在/nfs目录下访问它

.. code-block:: ini
    
    # app1 jail is located here
    namespace = /apps/app1
    
    # nfs share mounted on the host side
    namespace-keep-mount = /mnt/nfs1:/nfs

如果我们想要绑定的文件系统是被挂载在我们的jail里不包含的路径上，那么我们可以对--namespace-keep-mount使用"<source>:<dest>"语法。在这种情况下，/mnt/nfs1将会被绑定到jail内的/nfs目录上。
