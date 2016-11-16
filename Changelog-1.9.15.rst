uWSGI 1.9.15
============

更新日志[20130829]

错误修复
^^^^^^^^

* 修复jvm选项hashmap (#364)
* 修复python3 wsgi.file_wrapper
* 修复python3 --catch-exceptions
* 修复pypy wsgi.input.read中的类型
* 对pypy更好的符号检测
* 改进heroku上的ruby库管理
* 修复http-keepalive memleak
* 修复CPython下的spooler体管理
* 修复'fs'的unshare()使用
* 修复当使用--plugin构建插件时的UWSGI_PROFILE使用
* 改进SmartOS支持，以及添加OmniOS支持



新特性
^^^^^^^^^^^^

PTY插件
**************

这个新的插件允许你生成伪终端，并将其附加到你的worker上。

接着，通过网络 (UNIX或者TCP socket) 访问伪终端。

你可以将它们用于共享调试，或者让你的web应用拥有输入通道。

该插件出于开发初期 (特性非常少)，并且不是默认构建的，但是你可以做一些像这样有趣的事：

.. code-block:: ini

   [uwsgi]
   plugin = pty,rack
   ; generate a new pseudoterminal on port 5000 and map it to the first worker
   pty-socket = 127.0.0.1:5000
   
   ; classic options
   master = true
   processes = 2
   rack = myapp.ru
   socket = /tmp/uwsgi.socket
   
   ; run a ruby interactive console (will use the pseudoterminal)
   ; we use pry as it kick asses
   rbshell = require 'pry';binding.pry
   
现在，你可以这样访问伪终端了

.. code-block:: sh

   uwsgi --plugin pty --pty-connect 127.0.0.1:5000
   
你可以在多个窗口中运行该客户端，它将由所有客户端共享 (所有都将访问同一个伪终端)。

我们相信，关于它的新的有趣的使用很快就会出现

初步文档可以在 :doc:`Pty` 这里找到

strict模式
***********

编写uWSGI配置文件时最常见的错误之一是，选项名的拼写错误。

由于你可以添加任何选项到uWSGI配置文件中，因此系统将会接受任何你写的东东，即使它并不是一个真正的uWSGI选项。

虽然这个方法是非常强大的，允许大量有趣的hack，但是，它也有可能让你很头疼。

如果你想一步检查所有的选项，那么现在，你可以添加--strict选项了。未知选项将会触发一个致命错误。

回退配置
****************

uWSGI非常便宜 (在资源方面)， 并且支持大量的操作系统和架构，它被大量用于嵌入式系统中。

这种设备的常见特性之一就是"恢复到出厂设置"。

有了--fallback-config选项，uWSGI现在原生支持这类操作。

如果一个uWSGI实例随着exit(1)而亡，并且指定了回退配置，那么将会使用新的配置，将其作为唯一的参数，re-exec()该二进制文件。

让我们看看一个使用了不可绑定地址的配置 (非特权用户试着绑定到一个特权端口)

.. code-block:: ini

   [uwsgi]
   uid = 1000
   gid = 1000
   socket = :80
   
以及一个回退的配置 (绑定到非特权端口8080)

.. code-block:: ini

   [uwsgi]
   uid = 1000
   gid = 1000
   socket = :8080
   
(以root用户，因为我们想删除特权) 运行它：

.. code-block:: sh

   sudo uwsgi --ini wrong.ini --fallback-config right.ini
   
  
日志中将会是：

.. code-block:: sh

   ...
   bind(): Permission denied [core/socket.c line 755]
   Thu Aug 29 07:26:26 2013 - !!! /Users/roberta/uwsgi/uwsgi (pid: 12833) exited with status 1 !!!
   Thu Aug 29 07:26:26 2013 - !!! Fallback config to right.ini !!!
   [uWSGI] getting INI configuration from right.ini
   *** Starting uWSGI 1.9.15-dev-4046f76 (64bit) on [Thu Aug 29 07:26:26 2013] ***
   ...

--perl-exec和--perl-exec-post-fork
*************************************

现在，你可以在fork()调用之前和之后运行自定义perl代码了。

这两个选项都简单把perl脚本当成参数。

uwsgi.cache_keys([cache])
*************************

这个api函数已经别添加到python和pypy插件中。它允许你一个本地uWSGI缓存的键。

返回一个列表。

添加 `%(ftime)` 到日志格式
*****************************

这就像'ltime'，但它遵守--log-date格式

当另一个实例绑定到UNIX socket时，保护其不受破坏
********************************************************************

在启动时，uWSGI现在获得刚刚创建的unix socket的inode。

在vacuum下，如果inode发生了改变，那么会跳过socket的取消链接动作。

这应该有助于避免系统管理员破坏性竞争条件或错误配置

--worker-exec2
**************

这个就像--worker-exec，但在post_fork钩子之后运行

允许普通插件上的post_fork钩子
***************************************

普通插件 (没有.request钩子的那些) 现在可以公开.post_fork钩子了

--call钩子
************

与exec-*钩子有着相同的精神，call钩子工作方式相同，但是它会直接在当前进程的地址空间中调用函数 (必须将其作为有效符号公开)

以这个C源码为例 (称之为hello.c):

.. code-block:: c

   #include <stdio.h>
   
   void i_am_hello_world_for_uwsgi() {
       printf("Hello World!!!\n");
   }
   
并把它作为共享库编译：

.. code-block:: sh

    gcc -o libhello.so -shared -fPIC hello.c
    
现在，选择在uWSGI中何时 (以及何处) 调用它：

.. code-block:: sh

   ./uwsgi --help | grep -- --call-
    --call-pre-jail                        call the specified function before jailing
    --call-post-jail                       call the specified function after jailing
    --call-in-jail                         call the specified function in jail after initialization
    --call-as-root                         call the specified function before privileges drop
    --call-as-user                         call the specified function after privileges drop
    --call-as-user-atexit                  call the specified function before app exit and reload
    --call-pre-app                         call the specified function before app loading
    --call-post-app                        call the specified function after app loading
    --call-as-vassal                       call the specified function() before exec()ing the vassal
    --call-as-vassal1                      call the specified function(char *) before exec()ing the vassal
    --call-as-vassal3                      call the specified function(char *, uid_t, gid_t) before exec()ing the vassal
    --call-as-emperor                      call the specified function() in the emperor after the vassal has been started
    --call-as-emperor1                     call the specified function(char *) in the emperor after the vassal has been started
    --call-as-emperor2                     call the specified function(char *, pid_t) in the emperor after the vassal has been started
    --call-as-emperor4                     call the specified function(char *, pid_t, uid_t, gid_t) in the emperor after the vassal has been started
   
以数字结尾的选项是期望参数的变体 (其后缀是它们接收的参数的数目)

我们想要就在应用被加载之前调用函数：

.. code-block:: ini

   [uwsgi]
   ; load the shared library
   dlopen = ./libhello.so
   ; set the hook
   call-pre-app = i_am_hello_world_for_uwsgi
   ...


   
将会就在应用加载之前调用你的自定义函数。

考虑到那些函数是在进程地址空间中调用的，因此你可以对它们施加所有类型的（黑）魔法。

注：dlopen是对dlopen()函数的封装，因此所有相同的规则都适用 (必读：适用绝对路径！！！)
   
插件的init_func支持，以及--need-plugin变体
********************************************************

在加载插件的时候，你可以在dlopen()之后立即调用插件中定义的符号：

.. code-block:: sh

   uwsgi --plugin "foobar|myfunc" ...
   
uWSGI将会调用'foobar'插件公开的'myfunc'符号

--need-plugin就像--plugin，但是当插件加载失败的时候，会exit(1)进程

为pecan框架添加商品加载器(commodity loader)
**********************************************

作者：Ryan Petrello

为pecan WSGI框架添加了一个新的python加载器 (--pecan)

http://pecanpy.org/

https://uwsgi-docs.readthedocs.io/en/latest/Python.html#pecan-support

UWSGI_REMOVE_INCLUDES
*********************

在构建阶段，你可以用UWSGI_REMOVE_INCLUDES环境变量移除include头文件。

这对交叉编译有用，其中，一些自动检测的include文件可能是错误的。

router_expires
**************

我们在uWSGI核心中已经有多个设置Expires头的选项了。

添加这个路由器，以允许你对它们自定义：

.. code-block:: ini

   [uwsgi]
   route = /^foobar1(.*)/ expires:filename=foo$1poo,value=30
   route = /^foobar2(.*)/ expires:unix=${time[unix]},value=30

这个路由器接收一个filename的mtime或者一个unix time，添加'value'到其上，然后将它作为一个http日期返回。


在重载/关机时宣布Legion的死亡
******************************************

一旦触发了实例的重载（或者关闭），每一个legion成员现在都将会宣告它的死亡。

GlusterFS插件 (beta)
***************************

这个新的插件利用新的glusterfs c api，避免当提供存储在glusterfs服务器上的文件时的融合开销。

该插件支持多进程和多线程模式，而异步模式目前处于测试阶段。

文档在这里： :doc:`GlusterFS`

--force-gateway
***************

所有的网关 (fastrouter, httprouter, rawrouter, sslrouter ...)都必须运行在master进程之下。

通过指定--force-gateway，你可以绕过这个限制

初步的python3 profiler (测试版)
***********************************

--profiler pycall/pyline profiler已添加至python3中。它们处于测试阶段 (有内存泄漏问题)，但应该可用。

对OpenBSD,NetBSD,DragonFlyBSD的文件监控支持
****************************************************

在这些操作系统上，--fs-reload和@fmon装饰器现在都能使用。

--cwd
*****

你可以使用这个选项强制在“当前工作目录” (由--vacuum和重载子系统使用) 下启动。

在chroot设置中有用，其中，二进制可执行文件修改它的位置。

--add-gid
*********

这个选项允许你添加额外的组id到当前的进程中。你可以多次指定它。

Emperor和Linux名字空间改进
*****************************************

多亏了与pythonanywhere.com那些小伙伴的合作，已对Emperor进行了改进，使其能够与Linux名字空间更好的集成。

--emperor-use-clone选项允许你使用clone()而不是fork()来进行vassal生成。通过这种方式，你可以在一个新的名字空间中直接创建vassal。这个函数接收的参数与--unshare相同

.. code-block:: sh

   uwsgi --emperor /etc/vassals --emperor-use-clone pid,uts
   
将会在一个新的pid和uts名字空间中创建每个vassal

而

.. code-block:: sh

   uwsgi --emperor /etc/vassals --emperor-use-clone pid,uts,net,ipc,fs
   
将会基本上使用当前所有可用的名字空间。

两个新的exec (和call) 钩子现在可以用了：

--exec-as-emperor将会在生成一个vassal后立即在emperor中运行命令 (设置4个环境变量UWSGI_VASSAL_CONFIG, UWSGI_VASSAL_PID, UWSGI_VASSAL_UID和UWSGI_VASSAL_GID)

--exec-as-vassal将就在调用exec()之前，在vassal中运行命令 (所以直接在新的名字空间中)


--wait-for-interface
^^^^^^^^^^^^^^^^^^^^

由于处理Linux网络名字空间引入了大量的竞争条件 (特别是当使用虚拟以太网时)，这个新的选项让你暂停一个实例，直到有一个网络接口可用。

这在当等待emperor把veth移动到vassal名字空间的时候有用，避免vassal在接口可用之前在其之上运行命令


.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = pid,net,fs,ipc,uts
   ; each vassal should have its veth pair, so the following commands should be improved
   exec-as-emperor = ip link del veth0
   exec-as-emperor = ip link add veth0 type veth peer name veth1
   ; do not use the $(UWSGI_VASSAL_PID) form, otherwise the config parser will expand it on startup !!!
   exec-as-emperor = ip link set veth1 netns $UWSGI_VASSAL_PID




.. code-block:: ini

   [uwsgi]
   ; suspend until the emperor attach veth1
   wait-for-interface = veth1
   ; the following hook will be run only after veth1 is available
   exec-as-root = hostname vassal001
   exec-as-root = ifconfig lo up
   exec-as-root = ifconfig veth1 192.168.0.2
   uid = vassal001
   gid = vassal001
   socket = :3031
   ...


可用性
^^^^^^^^^^^^

uWSGI 1.9.15于2013年08月29日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.15.tar.gz
