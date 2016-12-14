为uWSGI的vassal可靠使用FUSE文件系统 (使用Linux)
============================================================

要求：uWSGI 1.9.18, 带FUSE和名字空间支持的Linux内核。

FUSE是一门允许在用户空间实现文件系统的技术 (因此，名字是：**F**\ ilesystem in **Use**\ rspace，即用户空间中的文件系统)。
有很多高质量的FUSE文件系统，因此，让你的应用依赖于它们是一种普遍形势。

FUSE文件系统是正常的系统进程，所以就像系统中的任何进程一样，它们会崩溃  (或许也许你不自觉地杀死了它们)。除此之外，如果你托管多个应用，每个都要求一个FUSE挂载点，那么你也许想要避免污染主挂载点名字空间，并且更重要的是，避免系统中存在未使用的挂载点 (例如，一个实例已经被完全移除了，而你不需要它的FUSE挂载点在系统中仍然可用)。

这个教程的目的是配置一个Emperor和一系列的vassal，每个挂载一个FUSE文件系统。

Zip文件系统
^^^^^^^^^^^^^^^^

`fuse-zip <https://code.google.com/p/fuse-zip/>`_ 是一个FUSE进程，将一个zip文件作为文件系统公开。

我们的目标是把整个应用存储在一个zip归档文件中，并且指示uWSGI将其作为一个文件系统（通过FUSE）挂载在 ``/app`` 之下。

Emperor 
***********

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = fs,pid
   
这里的技巧是使用Linux名字空间来在一个新的pid和文件名字空间中创建vassal。

第一个 (``fs``) 允许由vassal创建的挂载点只对这个vassal有用 (而不会混淆主要系统)，而 ``pid`` 允许uWSGI master成为这个vassal的“初始”进程 (pid 1)。成为"pid 1"意味着，当你死掉了，你所有的孩子也会死掉。在我们的场景之下 (其中，我们的vassal在启动的时候加载了一个FUSE进程)，它意味着当这个vassal被销毁的时候，FUSE进程也会被销毁，同时它的挂载点也会被销毁。

一个Vassal
********

.. code-block:: ini

   [uwsgi]
   uid = user001
   gid = user001
   
   ; mount FUSE filesystem under /app (but only if it is not a reload)
   if-not-reload =
     exec-as-user = fuse-zip -r /var/www/app001.zip /app
   endif =
   
   http-socket = :9090
   psgi = /app/myapp.pl
   
这里，我们使用 ``fuse-zip`` 命令的 ``-r`` 选项来获得一个只读挂载。

监控挂载点
***********************

当前设置的问题是，如果 ``fuse-zip`` 进程死掉了，那么实例将不再能够访问 ``/app`` ，知道重新生成实例。

uWSGI 1.9.18添加了 ``--mountpoint-check`` 选项。它强制master不断地验证指定文件系统。如果它失败了，那么整个实例将会被粗鲁地销毁。由于我们处于Emperor之下，因此vassal被销毁后会立即以一种干净的状态重启 (允许再次启动FUSE挂载点)。

.. code-block:: ini

   [uwsgi]
   uid = user001
   gid = user001
   
   ; mount FUSE filesystem under /app (but only if it is not a reload)
   if-not-reload =
     exec-as-user = fuse-zip -r /var/www/app001.zip /app
   endif =
   
   http-socket = :9090
   psgi = /app/myapp.pl
   
   mountpoint-check = /app
   
来点重金属：一个CoW rootfs (unionfs-fuse)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`unionfs-fuse <http://podgorny.cz/moin/UnionFsFuse>`_ 是一个联合文件系统（union filesystem）的用户空间实现。一个联合文件系统是多个文件系统的堆栈，因此，具有相同名字的目录被合并成单个视图。

联合文件系统不止这些，其中一个最有效的特性是写时拷贝 (COW或者CoW)。启用CoW意味着你将有一个不可变的/只读的挂载点基础，所有对其修改将会指向另一个挂载点。

我们的目标是拥有一个由我们所有客户共享的只读rootfs，以及对每个客户有一个可写挂载点 (配置为CoW)，其中，将会存储每个修改。

Emperor
***********

可以使用前面的Emperor配置，但是我们需要准备我们的文件系统。

层次将是：

.. code-block:: c

   /ufs (where we initially mount our unionfs for each vassal)
   /ns
     /ns/precise (the shared rootfs, based on Ubuntu Precise Pangolin)
     /ns/lucid (an alternative rootfs for old-fashioned customers, based on Ubuntu Lucid Lynx)
     /ns/saucy (another shared rootfs, based on Ubuntu Saucy Salamander)
     
     /ns/cow (the customers' writable areas)
       /ns/cow/user001
       /ns/cow/user002
       /ns/cow/userXXX
       ...
       
创建我们的rootfs:

.. code-block:: sh

   debootstrap precise /ns/precise
   debootstrap lucid /ns/lucid
   debootstrap saucy /ns/saucy
   
并且在每个中创建 ``.old_root`` 目录 (对 ``pivot_root`` 是必须的，见下)：

.. code-block:: sh

   mkdir /ns/precise/.old_root
   mkdir /ns/lucid/.old_root
   mkdir /ns/saucy/.old_root
   
确保安装所需的库到它们每一个中 (特别是你的语言所需的库)。

在这个rootfs中， ``uwsgi`` 二进制文件必须是可执行的，因此你必须花点时间在它上面 (一个好方法是对每个发行版编译一个语言插件，并且将其放公用目录中，例如，每个rootfs可以拥有一个 ``/opt/uwsgi/plugins/psgi_plugin.so`` 文件，以此类推)。

一个Vassal
***********

这里，事情变得有点复杂了。我们需要加载unionfs进程 (以root用户，因为它必须是我们新的rootfs)，然后调用 ``pivot_root`` (Linux上可以用的一个更高级的 ``chroot`` )。

:doc:`../Hooks` 是在各种uWSGI启动阶段运行自定义命令（或者函数）的最佳方式。

在我们的例子中，我们将在"pre-jail"阶段运行FUSE进程，然后在"as-root"阶段(在 ``pivot_root`` 之后)处理挂载点。

.. code-block:: ini

   [uwsgi]
   ; choose the approach that suits you best (plugins loading)
   ; this will be used for the first run ...
   plugins-dir = /ns/precise/opt/uwsgi/plugins
   ; and this after a reload (where our rootfs is already /ns/precise)
   plugins-dir = /opt/uwsgi/plugins
   plugin = psgi
   
   ; drop privileges
   uid = user001
   gid = user001
   
   ; chdir to / to avoid problems after pivot_root
   hook-pre-jail = callret:chdir /
   ; run unionfs-fuse using chroot (it is required to avoid deadlocks) and cow (we mount it under /ufs)
   hook-pre-jail = exec:unionfs-fuse -ocow,chroot=/ns,default_permissions,allow_other /precise=RO:/cow/%(uid)=RW /ufs

   ; change the rootfs to the unionfs one
   ; the .old_root directory is where the old rootfs is still available
   pivot_root = /ufs /ufs/.old_root
   
   ; now we are in the new rootfs and in 'as-root' phase
   ; remount the /proc filesystem
   hook-as-root = mount:proc none /proc
   ; bind mount the original /dev in the new rootfs (simplifies things a lot)
   hook-as-root = mount:none /.old_root/dev /dev bind
   ; recursively un-mount the old rootfs
   hook-as-root = umount:/.old_root rec,detach
   
   ; common bind
   http-socket = :9090
   
   ; load the app (fix it according to your requirements)
   psgi = /var/www/myapp.pl
   
   ; constantly check for the rootfs (seems odd but is is very useful)
   mountpoint-check = /
   
如果你的应用会试着写入它的文件系统，那么你会看到，在它的 ``/cow`` 目录中，所有的已创建/已更新文件都能用。

注释
^^^^^

一些FUSE文件系统不会提交写入，直到它们取消挂载。在这样的情况下，在vassal关闭的时候取消挂载是个不错的技巧：

.. code-block:: ini

   [uwsgi]
   ; vassal options ...
   ...
   ; umount on exit
   exec-as-user-atexit = fusermount -u /app
