uWSGI 2.0.1
===========

更新日志 [20140209]

错误修复和改进
*************************

- 由于错误的原型声明，无SSL构建uWSGI会导致编译错误。此问题已得到修复。
- 已修复PyPy插件中防止大量线程使用的竞争条件
- 只有在心跳子系统已启用的情况下才检查心跳状态
- 改进心跳代码，以支持各种边缘情况
- 改进psgi.input以支持read()中的偏移
- 修复 (和简化) perl堆栈跟踪使用情况
- 修复sni安全订阅
- CGI插件不再需要了，Status头部是第一个 (Andjelko Horvat)
- 修复CPython mule_msg_get超时时间解析
- 允许通过绝对路径嵌入配置文件
- 修复symcall rpc
- 修复CPython spooler api中的内存泄漏 (xiaost)
- --no-orphans加固已经回来了 (目前仅Linux)
- 改进dotsplit路由器模式，以减少DOS风险
- 子Emperor现在默认是loyal的
- 修复非共享ruby 1.8.7支持
- 修复harakiri CPython对tracebacker
- 请求变量现在正确由统计信息服务器公开
- 对logfile-chown支持log-master
- 改进legion重载
- 修复tuntap网络掩码
- 修复busyness插件，无需度量子系统

新特性
********

uWSGI 2.0是一个LTS分支，因此，别期待有灰常多的新特性。2.0.1是第一个维护发布版本，因此，你仍然可以获得一堆新特性
(主要特性并未在2.0完成)


Perl原生Spooler支持
---------------------------

Perl最终获得了对Spooler子系统的支持。在2.0中，我们添加了服务器支持，而在2.0.1中，我们也完成了客户端支持。

.. code-block:: pl

   use Data::Dumper;

   uwsgi::spooler(sub {
        my $env = shift;
        print Dumper($env);
        return uwsgi::SPOOL_OK;
   });

   uwsgi::spool({'foo' => 'bar', 'arg2' => 'test2'})


--alarm-backlog
---------------

当监听队列满的时候，引发指定告警

.. code-block:: ini

   [uwsgi]
   alarm = myalarm cmd:mail -s 'ALARM ON QUEUE' admin@example.com
   alarm-backlog = myalarm

--close-on-exec2
----------------

关于作者：Kaarle Ritvanen

这个标志在所有服务器socket上应用CLOSE_ON_EXEC socket标志。如果你不想要你请求生成进程继承服务器描述符，那么使用它。

注意：--close-on-exec在所有的socket（客户端和服务器）上应用此标志

简单通知子系统
------------------------------

订阅的一个烦人问题是，客户端不知道它是否已正确被订阅至服务器了。

该通知子系统允许你添加订阅包到一个数据报地址 (udp或者unix)，服务器将会发送回消息（例如成功的订阅）到该地址

.. code-block:: ini

   [uwsgi]
   ; enable the notification socket
   notify-socket = /tmp/notify.socket
   ; pass it in subscriptions
   subscription-notify-socket = /tmp/notify.socket
   ...
   
该通知子系统是真正通用的。期待未来会有更多的子系统使用它。

用于守护进程的pid名字空间 (仅Linux)
--------------------------------------

这是一个仅Linux的实验性特性，允许你在一个新的pid名字空间内生成守护进程。这个特性要求master作为root运行。

看看： :doc:`AttachingDaemons`

重订阅
---------------

fastrouter和http/https/spdy路由器现在支持"重订阅"。

你可以指定一个数据报地址 (udp或unix)，所有的订阅请求将会被转发到该地址 (明显改变节点地址到这个路由器地址)

这个系统在构建“联合”设置时会有用：

.. code-block:: ini

   [uwsgi]
   fastrouter = 192.168.0.1:3031
   fastrouter-subscription-server = 127.0.0.1:5000
   fastrouter-resubscribe = 192.168.0.2:5000
   
使用这个设置，192.168.0.2上的fastrouter会拥有目的被设置为192.168.0.1:3031的所有192.168.0.1的记录。

文件系统监控API
----------------------

该实时文件通知API已被标准化，现在可以通过插件使用它了。注册一个监控器的原型是：

.. code-block:: c

   struct uwsgi_fsmon *uwsgi_register_fsmon(char *path, void (*func) (struct uwsgi_fsmon *), void *data) {
   
它将会在“path”上注册一个监控器，触发函数"func"，并将"data"作为参数传递给该函数。

记住，这与"touch" api不同，它是基于poll的，并且只能监控文件。(而fsmon还可以监控文件夹)

对yajl 1.0的支持
--------------------

2.0添加了yajl JSON解析器 (version 2)的支持。2.0.1也添加了对1.0的支持。

for-readline
------------

一个配置逻辑迭代器，它生成（yield）文件行：

.. code-block:: ini

   [uwsgi]
   for-readline = /etc/myenvs
     env = %(_)
   end-for =

%i和%j魔术变量
--------------------

%i -> 返回当前解析文件的inode

%j -> 返回当前解析绝对文件名的32位djb33x哈希的十六进制表示

--inject-before 和 --inject-after
----------------------------------

这两个新的选项应该为每个人使配置模板系统完整。

它们基本上前置和附加'blobs'到一个配置文件。

是哒，这听起来有点扯淡。

看看下面的例子：

header.xml:

.. code-block:: xml

   <uwsgi>
       <socket>:3031</socket>
       
footer.xml:

.. code-block:: xml

   <master/>
       </uwsgi>
       
和body.xml:

.. code-block:: xml

   <processes>8</processes>
   
你可以这样构建一个单一的配置树：

.. code-block:: sh

   uwsgi --show-config --inject-before header.xml --inject-after footer.xml --xml body.xml
   
这个方法，虽然原始，但是让你以更高级的方式使用魔术变量（因为你可以使用它们控制文件的上下文）

注意：顺序很重要，--inject-before和--inject-after必须在相关配置选项之前指定。

--http-server-name-as-http-host
-------------------------------

一些Ruby/Rack中间件在SERVER_NAME/HTTP_HOST检查的时候进行可疑检查。

这个标志允许http路由器自动映射SERVER_NAME到HTTP_HOST，而不是指示你的uWSGI实例来做这件事。

更好的Emperor的Ragnarok (关闭过程)
----------------------------------------------

'Ragnarok'是Emperor阶段，当你让它关闭时，会执行此阶段。

在2.0.1之前，这个过程简单发送KILL给vassal，来粗暴销毁它们。

而这个新的Ragnarok则是一种更加仁慈的方式，让vassal优雅关闭。

Emperor对vassal不关闭的容忍可以使用--reload-mercy来调整 (默认30秒)

PyPy粘贴支持
------------------

已添加两个用于PyPy插件的新选项，用于粘贴支持：

--pypy-paste <config>

--pypy-ini-paste <ini>

它们都1:1映射到CPython变量，但与其相反，它们自动修复日志记录

可用性
************

你可以从这里下载uWSGI 2.0.1： http://projects.unbit.it/downloads/uwsgi-2.0.1.tar.gz
