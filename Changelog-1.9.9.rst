uWSGI 1.9.9
===========

更新日志[20130508]

特别警告！！！
*******************

router_basicauth插件已经更改了它的默认行为，当鉴权失败的时候，会返回"break"。

"basicauth-next"动作，使用老行为 (返回"next")

这个新方法应该减少由错误配置引发的安全性问题

错误修复
********

* 不为“不负责的”插件增加"tx"统计数据计数
* 修复--backtrace-depth
* 修复cache-sync解析
* 修复mule farm初始化
* 修复当使用正则表达式条件路由时的多线程问题
* 修复psgi插件中的default-app使用
* 修复python动态模式 + 线程
* 修复重试出现时corerouter中的错误报告
* 为网关正确报告harakiri条件

新特性
************

WebDav插件
^^^^^^^^^^^^^^^^^

WebDav是该项目要求得挺多的特性之一。现在，我们有了一个测试版本的插件，已经支持诸如carddav这样的其他标准：

https://github.com/unbit/uwsgi/blob/master/t/webdav/carddav.ini

官方的modifier是35，并且要把一个简单的目录作为webdav共享（适用于windows, gnome....）进行挂载，你仅需指定--webdav-mount选项：

.. code-block:: ini

   [uwsgi]
   plugin = webdav
   http-socket = :9090
   http-socket-modifier1 = 35
   webdav-mount = /home/foobar

记得保护共享：

.. code-block:: ini

   [uwsgi]
   plugin = webdav,router_basicauth
   http-socket = :9090
   http-socket-modifier1 = 35
   route-run = basicauth:CardDav uWSGI server,unbit:unbit
   webdav-mount = /home/foobar

WebDav属性是作为文件系统xattr进行存储的，因此，一定要使用支持它的文件系统 (ext4, xfs, hfs+...)

LOCK/UNLOCK支持仍然是不完整的

稍后会有官方文档。

支持Go 1.1 (或多或少，对于go用户而言，是个坏消息……)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

虽然你可以成功地把go 1.1应用嵌入到uWSGI中，但是go 1.1将是完全fork()不安全的。

那意味着，你不能使用多进程，master，mule等等。

基本上，一半的uWSGI特性在go应用中都不怎么能用。

以后可能会有所改变，但目前，我们的目标是与gccgo项目更好的集成。

将会继续支持Go 1.0.x (除非gccgo显示了自己是一个更好的选择)

将会实现更多的特性。

改进的异步模式
^^^^^^^^^^^^^^^^^^^^

已经更新Stackless, Greenlet和Fiber支持，以支持新的异步特性

radius插件
^^^^^^^^^^^^^^^^^

现在，你可以使用router_radius插件来鉴权radius服务器了：

.. code-block:: ini

   [uwsgi]
   plugin = webdav,router_radius
   http-socket = :9090
   http-socket-modifier1 = 35
   route-run = radius:realm=CardDav uWSGI server,server=127.0.0.1:1812
   webdav-mount = /home/foobar

SPNEGO插件
^^^^^^^^^^^^^^^^^

另一个鉴权后端，使用SPNEGO (kerberos)

.. code-block:: ini

   [uwsgi]
   plugin = webdav,router_spnego
   http-socket = :9090
   http-socket-modifier1 = 35
   route-run = spnego:HTTP@localhost
   webdav-mount = /home/foobar

这个插件还在测试阶段，因为它会内存泄漏 (看起来是MIT-kerberos中的一个问题)，并Heimdal实现并没有用。

欢迎更多的报告

ldap认证
^^^^^^^^^^^^^^^^^^^^^^

(作者：Łukasz Mierzwa)

目前，它缺乏SASL支持。很快会对其进行改善。

.. code-block:: ini

   [uwsgi]
   ...
   plugins = router_ldapauth
   route = ^/a ldapauth:LDAP realm,url=ldap://ldap.domain,com;basedn=ou=users,dc=domain.com;binddn=uid=proxy,dc=domain,dc=com;bindpw=password



新的内部路由特性
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们移除了GOON动作，因为它乱糟糟的，并且基本上对新的鉴权方法没用

添加了"setscriptname"动作来覆盖内部计算的SCRIPT_NAME (不仅是变量)

"donotlog"动作强制uWSGI不要记录当前请求

改进了"regexp"路由条件，以允许分组。现在，你可以很容易地操作字符串，并把它们当成新的请求变量进行添加：

.. code-block:: ini

   [uwsgi]
   ...
   route-if = regexp:${REQUEST_URI};^/(.)oo addvar:PIPPO=$1
   route-run = log:PIPPO IS ${PIPPO}

这将会获取foo的第一个字符，然后将其放到PIPPO请求变量中

Gevent atexit钩子
^^^^^^^^^^^^^^^^^^

uwsgi.atexit钩子现在由gevent插件兑现 (作者：André Cruz)


流转换
^^^^^^^^^^^^^^^^^^^^^^^^^

可以在运行时应用转换 (不涉及缓冲)。

看看已更新的文档： :doc:`Transformations`

xattr插件
^^^^^^^^^^^^^^^^

xattr插件让你在内部路由子系统中应用文件扩展属性：

.. code-block:: ini

   [uwsgi]
   ...
   route-run = addvar:MYATTR=user.uwsgi.foo.bar
   route-run = log:The attribute is ${xattr[/tmp/foo:MYATTR]}


或者 (带2个变量的变种)

.. code-block:: ini

   [uwsgi]
   ...
   route-run = addvar:MYFILE=/tmp/foo
   route-run = addvar:MYATTR=user.uwsgi.foo.bar
   route-run = log:The attribute is ${xattr2[MYFILE:MYATTR]}


airbrake插件
^^^^^^^^^^^^^^^^^^^

(作者：Łukasz Mierzwa)

目前处于开发的早期阶段，允许发送uWSGI异常和高级给airbrake服务器。

稍后将会有官方文档。

Legion守护进程
^^^^^^^^^^^^^^

(作者：Łukasz Mierzwa)

不，这不是一个黑金属乐队，它是 :doc:`Legion` 的一个新特性，允许你只在实例是lord的时候运行外部进程：

.. code-block:: ini

   [uwsgi]

   master = true
   http = :8081
   stats = :2101
   wsgi-file = tests/staticfile.py

   logdate = true

   legion = legion1 225.1.1.1:19678 100 bf-cbc:abc
   legion-node = legion1 225.1.1.1:19678

   legion-attach-daemon = legion1 memcached -p 10001

   legion-smart-attach-daemon = legion1 /tmp/memcached.pid memcached -p 10002 -d -P /tmp/memcached.pid


--touch-exec
^^^^^^^^^^^^

有一个新的"touch"选项 (像--touch-reload)可以用了，触发一条命令的执行：

.. code-block:: ini

   [uwsgi]
   ...
   touch-exec = /tmp/foobar run_my_script.sh
   touch-exec = /var/test/foo.txt run_my_second_script.sh arg1 arg2


用于缓存的数学运算
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在，你可以使用缓存子系统来存储64位有符号数，并在其上应用原子操作。

已使用5个新函数扩展了uwsgi api (目前仅有python插件公开)：

*uwsgi.cache_num(key[,cache]) ->从指定项获取64位数

*uwsgi.cache_inc(key[,amount=1,expires,cache]) -> 增加指定键指定值

*uwsgi.cache_dec(key[,amount=1,expires,cache]) -> 减少指定键指定值

*uwsgi.cache_mul(key[,amount=2,expires,cache]) -> 乘以指定键指定值

*uwsgi.cache_div(key[,amount=2,expires,cache]) -> 除以指定键指定值

已将该新api暴露给路由子系统了，允许你事先高级模式，例如请求限制器：

https://github.com/unbit/uwsgi/blob/master/t/routing/limiter.ini

这个例子显示了热限制一个单一的IP的请求为每30秒10个

这个新功能的长期目标是作为即将到来的度量子系统的基础

可用性
************

uWSGI 1.9.9自20130508起可用，你可以在下面的url中找到它

http://projects.unbit.it/downloads/uwsgi-1.9.9.tar.gz

