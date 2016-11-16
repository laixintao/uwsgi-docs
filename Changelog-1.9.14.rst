uWSGI 1.9.14
============

更新日志[20130721]


错误修复
********

- 修复python modifier1管理 (硬编码为0)
- 修复http和http-socket中的url解码 (它现在支持小写二进制，由Miles Shang发现)
- 为不可删除的unix socket添加更多用户友好型错误信息
- 修复http 1.1 keepalive模式中的--http-auto-chunked (André Cruz)
- 修复python wheel支持 (Fraser Nevett)
- 修复--safe-fd (没有被Emperor正确遵循)
- 修复ruby 2.x重载
- 改进对OSX Tiger的支持 (是的，OSX 10.4)
- 对监听队列负载的更好计算
- 修复OSX上的v8构建
- 修复pypy rpc
- 改进分块api性能
- 修复使用python3时的latin1编码
- 修复--spooler-ordered (Roberto Leandrini)
- 修复报告到请求日志中的状态行


新特性
********

Ruby 1.9.x/2.x原生线程支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ruby 1.9 (mri)引进了原生线程支持 (非常类似于CPython，由一个名为GVL的全局锁管理)。

由于各种原因 (看看在源插件上面的注释)，在uWSGI中的ruby线程支持被当成“循环引擎插件”实现。

你需要构建"rbthreads"插件 (当使用'ruby2'构建配置文件时会自动构建它)，并且用'--rbthreads'来启用它

已扩展了gem脚本，当检查到ruby >= 1.9的时候，会自动选择'ruby2'构建配置文件 (这对于Heroku用户而言，应该会更轻松)

Rails4是第一个支持和庇佑线程的Ruby on Rails版本 (在3.x中，你需要明确启用支持)。只有在“生产”模式下，你才能在Rails4中使用多线程，否则，在第一个请求之后，你的应用将会死锁。

一个样例配置：

.. code-block:: ini

   [uwsgi]
   plugins = rack,rbthreads
   master = true
   ; spawn 2 processes
   processes = 2
   ; spawn 8 threads
   threads = 8
   ; enable ruby threads
   rbthreads = true
   ; load the Rack app
   rack = config.ru
   ; bind to an http port
   http-socket = :9090
   http-socket-modifier1 = 7
   
它将会生成总共16个线程

文件系统监控接口 (fsmon)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

目前，uWSGI可以使用“简单的”--touch-*功能或者信号框架来监控文件系统修改 (使用各种操作系统API，例如inotify或者kqueue)。

已为插件编写者添加了一个名为"fsmon"的新接口，允许方便实现实时操作系统监控。

已添加三个新的选项：

   `--fs-reload <path>`
   
   
   `--fs-brutal-reload <path>`
   
   
   `--fs-signal <path> <signal>`
   
与--touch-*选项相反，它们是实时的 (一旦发生改变，就会立即唤醒master)，并且使用内核功能
(目前只支持inotify()和kqueue())。多亏了这个选择，现在，你可以监控一整个目录的改动了 (不需要像inotifywatch这样的外部进程/封装)

uClibc支持
^^^^^^^^^^^^^^

作者：Natanael Copa

现在，可以在基于uclibc的系统(通常是嵌入式系统)上构建uWSGI了

Alpine Linux是已测试支持的操作系统

Lua 5.2支持
^^^^^^^^^^^^^^^

作者：Natanael Copa

lua插件现在支持Lua 5.2

setscheme, setdocroot
^^^^^^^^^^^^^^^^^^^^^

这两个新的路由动作允许你动态覆盖DOCUMENT_ROOT和UWSGI_SCHEME

sendfile, fastfile
^^^^^^^^^^^^^^^^^^

这两个动作 (已添加到router_static插件中) 允许你绕过DOCUMENT_ROOT检查，返回静态文件给客户端。

第一个强制使用sendfile()系统调用 (如果有的话)，而第二个自动尝试选择最好的服务策略 (像卸载一样)

--reload-on-fd和--brutal-reload-on-fd
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

两个新的选项允许你在一个文件描述符准备好的时候重载一个实例。

目前，最好的使用场景是用于oom_control cgroup接口 (通过eventfd)。

假设你有一个进程封装器，分配一个eventfd()，报告OOM事件 (并且作为'OOM'环境变量公开)，那么你可以在内存不足的时候这样强制uWSGI重载：

.. code-block:: ini

   [uwsgi]
   ...
   reload-on-fd = $(OOM):8 OUT OF MEMORY !!!
   

它表示：

监控$(OOM)文件描述符，当准备好的时候，从中读取8个字节 (这是eventfd()要求的)，然后在日志中打印"OUT OF MEMORY !!!"，接着优雅重载该实例。

显然，这只是使用它的一种方式。UNIX世界是基于文件描述符的，因此，你有很多有趣的方式来使用它。


Spooler改进
^^^^^^^^^^^^^^^^^^^^

作者：Roberto Leandrini


所有的工作都已经在uwsgidecorators.py中有效地完成了。

现在，你可以传递给所有可用的spooler相关的装饰器"pass_arguments=True"选项了，来自动序列化spooler函数参数。这是一个抽象，避免你需要序列化/反序列化参数。

除此之外，已扩展了装饰器，实现了 __call__ ，这样，你就可以直接把由spoller装饰的函数当成正常函数调用了。

--emperor-nofollow
^^^^^^^^^^^^^^^^^^

启用这个选项将会让Emperor监控符号链接的mtime更新，而不是真正文件的mtime。

Alberto Scotto正在进行一个支持这两种的更新版本 (在下一个版本中应该就准备好了)

daemontools envdir支持
^^^^^^^^^^^^^^^^^^^^^^^^^^

尽管daemontools看起来过时了，但是像envdirs (http://cr.yp.to/daemontools/envdir.html)这样的东东被大量用在各种环境中。

uWSGI有两个新的选项 (--envdir <path>和--early-envdir <path>)，允许你支持这种特别的 (过时的?)配置方式。

xmldir改进
^^^^^^^^^^^^^^^^^^^

作者：Guido Berhoerster

改进了xmldir插件，支持基于iconv的utf8编码。各种小补丁已提交。

样例目录包含了两个新的文件，展示了xmldir+xslt使用


重磅消息 !!!
*****************

Servlet 2.5支持开发才刚刚起步。该插件存在版本树中，但它是不可用的 (它是一个硬编码jsp引擎)。夏天过后，我们预期会出一个测试版本。显然，对于非企业用户，我们无耻地认为 :doc:`JWSGI` 是一个比servlet更好的方法 ;)

可用性
************

从下面地址下载uWSGI 1.9.14

http://projects.unbit.it/downloads/uwsgi-1.9.14.tar.gz
