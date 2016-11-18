uWSGI 1.9.18
============

更新日志 [20131011]

License变更
**************

uWSGI的这个版本是1.9版本树第一个使用GPL2 + linking exception而不是简单的GPL2的版本。

这个新的license应该会避免在把uWSGI作为共享库使用（或者使用非GPL2兼容的库链接它）的时候的任何问题

记住：如果你需要对uWSGI进行闭源修改，那么你可以购买一个商业license。

错误修复
********

- 修复大端机器上的uwsgi原生协议支持
- 为arm修复jvm构建系统 (Jorge Gallegos)
- 修复zlib管理中由cppcheck发现的内存泄漏
- 在每一个emperor glob迭代chdir()
- 正确遵循--force-cwd
- 修复ia64/Linux编译 (Jonas Smedegaard/Riccardo Magliocchetti)
- 修复ruby rvm路径解析顺序
- 在守护进程的SIGTERM后添加waitpid() (Łukasz Mierzwa)
- 修复--idle后的Pid编号 (Łukasz Mierzwa)
- 修复/改进cheaper内存限制 (Łukasz Mierzwa)
- 正确关闭网关中的继承socket
- 修复mmap()中对MAP_FAILED的检查 (而不是NULL)
- 修复FastCGI非阻塞请求体read() (由Arkaitz Jimenez提供补丁)
- 修复attach.py脚本
- 避免因不符合标准的PSGI响应头引发的崩溃
- 在非lazy时，（即使是在非apps模式下）运行python自动重载器

新特性
************

最小构建配置文件
^^^^^^^^^^^^^^^^^^^^^^

尽管uWSGI核心的内存使用量一般在1.8和2.5兆之间，但在一些用例下，你会想要一个最低限度的核心以及一组嵌入式插件。

例子诸如用户不利用uWSGI的指定特性，或者是uWSGI使用的库与其他库（例如openssl或者zeromq）的名字冲突这种情况。

添加了一堆“最小化”构建配置文件：

 * pyonly (构建一个最小化的CPython WSGI服务器)
 * pypyonly (构建一个最小化的PyPy WSGI服务器)
 * plonly (构建一个最小化的PSGI服务器)
 * rbonly (构建一个最小化的Rack服务器)
 
唯一支持的配置格式是.ini，而内部路由和legion子系统并未内置。

例如，如果你想要通过pip安装最小uWSGI二进制文件：

.. code-block:: sh

   UWSGI_PROFILE=pyonly pip install uwsgi
   
重要：最小构建配置文件并不会提高性能，由于设计uWSGI的方式，未使用的特性并不会消耗CPU。最小构建配置文件只影响最终的二进制文件大小
   
Auto-fix modifier1
^^^^^^^^^^^^^^^^^^

为非python插件设置modifier1是及其烦人的 (你往往会忘掉它)。

现在，如果请求的modifier1是0，但未加载python插件 (或者没有已加载的python应用)，那么第一个配置的应用将会被替代设置 (除非你使用--no-default-app来禁用这个特性)。

这意味着，现在，你可以运行：

.. code-block:: sh

   uwsgi --http-socket :9090 --psgi myapp.pl
   
而不是

.. code-block:: sh

   uwsgi --http-socket :9090 --http-socket-modifier1 5 --psgi myapp.pl

显然，试着总是设置modifier1，这只是一个便捷方式

Perl自动重载器
^^^^^^^^^^^^^^^^^^

--perl-auto-reload选项允许psgi插件在每次请求之后检查已修改的模块。它以扫描的频率（以秒为单位）作为参数。

该扫描在服务了一个请求之后进行。它是次优的，但是它也是最安全的选择。

"raw"模式 (预览技术，只用于CPython)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当在Unbit着手于一个新的服务器端项目时，我们有需要公开使用一个非常特别的协议的web应用 (uWSGI不支持它们中任何一个)。

我们的第一个方式是将新的协议作为插件添加，但很快我们就意识到这太具体了。因此，我们决定引入RAW模式。

Raw模式允许你直接在应用可调用对象中解析请求。与在可调用对象中获得CGI变量/头列表相反，accept()之后你只获得文件描述符。

然后你可以自由地对那个文件描述符进行read()/write()。

.. code-block:: python

   import os
   def application(fd):
      os.write(fd, "Hello World")
      
.. code-block:: sh

   uwsgi --raw-socket :7070 --python-raw yourapp.py

Raw mode禁用请求日志记录。目前，我们只对CPython支持，如果我们得知其他语言支持（或者感兴趣），那么我们肯定会添加支持。

重要：raw模式并非标准，因此别期待会应用任何中间件或者常见使用模式。将其当成一个低层次的socket封装器使用。



对于WSGI响应，CPython缓存协议的可选的非标准支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作者：yihuang，在INADA Naoki (methane)的帮助下

WSGI (PEP333/3333)对于响应的有效对象类型是非常清晰的：str用于python2，bytes用于python3

uWSGI (大量使用mod_wsgi作为参考) 总是强制这样的行为，因此，“怪异的”模式，例如返回不支持的字节数组。这样的使用在纯python应用服务器会有点无意识支持，只是因为它们在其之上简单调用write()，或者因为在返回之前将它们转换成字符串 (非常低效)

yihuang提出的补丁建议使用 CPython C api公开的低层次的缓存协议。字符串 (python2中) 和字节 (python3中) 支持该缓存协议，因此，它的使用是透明的，并且也保证向后兼容。 (给CPython C api专家：是哒，我们同时支持老的和新的缓存协议)

这是一个非标行为，你必须通过--wsgi-accept-buffer来自愿启用它。

小心使用，因为它可能会掩盖错误以及/或者错误行为。

注意：如果你尝试使用1.9.18-dev，那么你可能会注意到这个选项是默认被启用的。它是一个错误。多亏了Graham Dumpleton (mod_wsgi作者)指出。

Emperor和配置改进
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

关于作者：Matthijs Kooijman

已改进配置系统，相对于strict模式，它甚至更加一致 (提醒：使用--strict，你基本上为未知选项检查配置文件，避免了由于错别字引发的头疼问题)。

新增了魔术变量，公开原始配置文件的名字 (这在Emperor模式下简化了模板)，看一看：https://github.com/unbit/uwsgi-docs/blob/master/Configuration.rst#magic-variables

使用--emperor-cap选项，Emperor获得了Linux功能的支持。该选项以vassal作为root启动的时候，你想要保持的功能列表为参数：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-cap = setuid,net_bind_service
   
通过这个设置，你的vassal将只能够移除特权，并且绑定到<1024的端口

它的好基友是linux名字空间的CLONE_NEWUSER标志，现在uWSGI完全支持它：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = user
   emperor-cap = setuid,net_bind_service
   
这将会为vassal创建一个新的带有更少特权的root用户(CLONE_NEWUSER是非常难以理解的，但理解它的最好的方式是将其视为带有专有功能的新的root用户)

构建系统改进
^^^^^^^^^^^^^^^^^^^^^^^^^

已改进构建系统，它现在可以运行时链接自定义资源了。对于低层次的钩子，它运行良好：

.. code-block:: c

   // embed_me.c
   #include <stdio.h>
   
   void hello_i_am_foobar() {
           printf("I Am foobar");
   }

现在，我们可以一次性链接到这个文件到主uWSGI二进制文件上：


.. code-block:: sh

   UWSGI_ADDITIONAL_SOURCES=embed_me.c make

然后，你可以自动访问你的钩子：

.. code-block:: sh

   uwsgi --http-socket :9090 --call-asap hello_i_am_foobar
   
最后，Riccardo Magliocchetti重写了构建脚本，使用optparse，而不是原始/老式的sys.argv解析


插件化'模式(scheme)'管理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

模式(scheme)是uWSGI uri的前缀部分。当你这样

.. code-block:: sh

   uwsgi --ini http://foobar.local:9090/test.ini
   
http://就是模式，发信号给uWSGI，告诉它必须通过http下载配置文件。

直到现在，那些'模式(scheme)'都是硬编码的。现在，它们作为茶具被公开出来，因此你可以添加更多的模式(scheme)了 (或者覆盖默认的)。

新的系统也已被应用到PSGI插件了 (抱歉啦，我们确保只有perl开发者才会明白那种诗意 :P)，因此，你可以做这样的事：

.. code-block:: sh

   uwsgi --http-socket :1717 --psgi http://yourapps.local/dancer.pl
   
或者

.. code-block:: sh

   ./uwsgi --binary-append-data yourapp.pl > blob001
   cat blob001 >> ./uwsgi
   ./uwsgi --http-socket :1717 --psgi data://0

挂载点检查
^^^^^^^^^^^^^^^^^^

可能很难理解为什么一个应用服务器应该检查挂载点。

正如几年前，理解如何在用户空间内写入文件系统是件傻事。

因此，看看这篇关于用uWSGI管理Fuse文件系统的文章： https://uwsgi-docs.readthedocs.io/en/latest/tutorials/ReliableFuse.html

初步libffi插件
^^^^^^^^^^^^^^^^^^^^^^^^^

随着为公开的钩子嵌入c库变得越来越普遍，我们开始致力于libffi集成，允许传给钩子安全（而理智）的参数。很快就会有相关内容了。

对kFreeBSD的官方支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

官方支持Debian/kFreeBSD。

你甚至可以也使用FreeBSD jails了!!!

:doc:`FreeBSDJails`

可用性
************

uWSGI 1.9.18于2013年10月11日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.18.tar.gz
