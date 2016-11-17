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

The patch proposed by yihuang suggests the use of the low-level buffer protocol exposed by the CPython C api. Strings (in python2) and bytes (in python3) support the buffer protocol, so its use is transparent
and backward compatibility is granted too. (for the CPython C api experts: yes we support both old and new buffer protocol)

这是一个非标行为，你必须通过--wsgi-accept-buffer来自愿启用它。

Use with care as it could mask errors and/or wrong behaviours.

Note: if you tried 1.9.18-dev you may note this option was enabled by default. It was an error. Thanks to Graham Dumpleton (mod_wsgi author) for pointing it out.

Emperor和配置改进
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Credits: Matthijs Kooijman

The config system has been improved to be even more consistent in respect to strict mode (remainder: with --strict you basically check your config files for unknown options
avoiding headaches caused by typos).

New magic vars have been added exposing the name of the original config file (this simplify templating when in Emperor mode), check them at https://github.com/unbit/uwsgi-docs/blob/master/Configuration.rst#magic-variables

The Emperor got support for Linux capabilities using the --emperor-cap option. The option takes the list of capability you want to maintain
for your vassals when they start as root:

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-cap = setuid,net_bind_service
   
with this setup your vassal will be only able to drop privileges and bind to ports < 1024

Its best friend is the CLONE_NEWUSER flag of linux namespaces that is now fully supported on uWSGI:

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = user
   emperor-cap = setuid,net_bind_service
   
this will create a new root user for the vassal with fewer privileges (CLONE_NEWUSER is pretty hard to understand, but the best thing
to catch it is seeing it as a new root user with dedicated capabilities)

构建系统改进
^^^^^^^^^^^^^^^^^^^^^^^^^

The build system has been improved to link custom sources on the fly. This works great for low-level hooks:

.. code-block:: c

   // embed_me.c
   #include <stdio.h>
   
   void hello_i_am_foobar() {
           printf("I Am foobar");
   }

Now we can link this file to the main uWSGI binary in one shot:


.. code-block:: sh

   UWSGI_ADDITIONAL_SOURCES=embed_me.c make

and you will automatically get access for your hooks:

.. code-block:: sh

   uwsgi --http-socket :9090 --call-asap hello_i_am_foobar
   
Finally, Riccardo Magliocchetti rewrote the build script to use optparse instead of raw/old-fashioned sys.argv parsing


插件化'schemes'管理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

schemes are the prefix part of uWSGI uri's. When you do

.. code-block:: sh

   uwsgi --ini http://foobar.local:9090/test.ini
   
the http:// is the scheme, signalling uWSGI it has to download the config file via http.

Til now those 'schemes' were hardcoded. Now they are exposed as plugins, so you can add more of them (or override the default one).

The new system has been applied to the PSGI plugin too (sorry we are sure only perl developers will understand that kind of poetry :P) so you can do things like:

.. code-block:: sh

   uwsgi --http-socket :1717 --psgi http://yourapps.local/dancer.pl
   
or

.. code-block:: sh

   ./uwsgi --binary-append-data yourapp.pl > blob001
   cat blob001 >> ./uwsgi
   ./uwsgi --http-socket :1717 --psgi data://0

mountpoints checks
^^^^^^^^^^^^^^^^^^

It could be hard to understand why an application server should check for mountpoints.

In the same way understanding how writing filesystem in userspace was silly few years ago.

So, check the article about managing Fuse filesystem with uWSGI: https://uwsgi-docs.readthedocs.io/en/latest/tutorials/ReliableFuse.html

初步libffi插件
^^^^^^^^^^^^^^^^^^^^^^^^^

As embedding c libraries for exposing hooks is becoming more common, we have started working on libffi integration, allowing
safe (and sane) argument passing to hooks. More to came soon.

对kFreeBSD的官方支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Debian/kFreeBSD is officially supported.

You can even use FreeBSD jails too !!!

:doc:`FreeBSDJails`

可用性
************

uWSGI 1.9.18于2013年10月11日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.18.tar.gz
