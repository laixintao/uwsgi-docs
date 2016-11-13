uWSGI 1.9
=========


这是一个将会指向LTS 2.0点版本。它引入了许多内部改动，并且移除了大量基本未使用、不能用或者太丑的功能。


1.0.x弃用的选项已经被永久删除。

一切皆非阻塞
********************

从现在开始，所有的请求差距，都需要是非阻塞的。已经添加了一个新的
C/C++/Obj-C api集合，来帮助用户／开发者安全编写非阻塞代码。诸如RPC这样的插件已经使用新api进行重写了，这让你能够通过像Gevent或者Coro::Anyevent这样的引擎来使用它。重写了异步模式，从而更好地配合新规则。你可以在 :doc:`Async` 找到更多信息。新的异步模式要求某些形式的协程/绿色线程/挂起引擎正确工作。再次，看看
:doc:`Async`

Coro::AnyEvent
**************

这个Perl/PSGI插件是uWSGI项目最老的插件之一，但过去它并不支持高级方式下的异步模式。

多亏了新的 :doc:`Async` 模式，才添加了一个Coro::Anyevent (coroae)循环引擎。

要构建它，你需要Coro::Anyevent包 (可以使用cpanm来获取它)，然后添加--coroae <n> 到你的选项中，其中，<n>是要生成的异步核心数。


JVM插件
**************

在uWSGI 1.9中，我们终于有了一个真正可用的JVM基础设施了。看看 :doc:`JVM` 中的文档吧。改进的 :doc:`JWSGI` 支持，以及新的Clojure :doc:`Ring` 插件都可以使用了。

Mono ASP.NET插件
***********************

第一次Mono插件尝试 (在2010年) 就是一次彻底的失败。现在，我们有了一个新的闪亮实现。

看看这个文档： :doc:`Mono`

语言独立的HTTP体管理
*****************************************

写uWSGI请求插件最恼人的任务之一就是每次都要重新实现http体读取器的管理。

新的非阻塞api添加了3个简单通用的C/C++/Obj-C函数来以一种语言独立的方式处理它：


.. code-block:: c

   char *uwsgi_request_body_read(struct wsgi_request *wsgi_req, ssize_t hint, ssize_t *rlen);
   char *uwsgi_request_body_readline(struct wsgi_request *wsgi_req, ssize_t hint, ssize_t *rlen);
   void uwsgi_request_body_seek(struct wsgi_request *wsgi_req, off_t pos); 


它们自动管理post缓存，非阻塞和上传进展。

所有的请求插件已更新到新的API。



更快的uwsgi/HTTP/FastCGI/SCGI原生socket
*********************************************

所有的--socket协议解析器已经全部被重写，从而更快 (更少的使用系统调用)，并且使用更少的内存。现在，它们更复杂了，但是你应该注意到 (在已加载站点上) 每个请求的系统调用数的减少。


添加了对SCGI协议的支持，并且已实现了NPH fastcgi模式 (这里，输出是HTTP，而不是cgi)。

FastCGI协议现在支持真正的sendfile()使用

为HTTP和FastCGI将请求体存储在一个临时文件中的这种老做法已经被移除 (除非你使用post缓存)。这意味着，除了uwsgi之外，你现在可以使用其他协议进行上传了。


请求日志记录 VS 错误日志记录
******************************

较老的uWSGI发布版本的最恼人的问题之一是缺乏方便地把请求日志从错误日志中分割出来的能力。现在，你可以创建一个记录器，然后让它只记录请求：

.. code-block:: ini

   [uwsgi]
   req-logger = syslog
   ...

举个例子，你也许想要发送请求日志到syslog和redis，发送错误日志到mongodb (放到foo.bar集合中):

.. code-block:: ini

   [uwsgi]
   req-logger = syslog
   req-logger = redislog:127.0.0.1:6269
   logger = mongodblog:127.0.0.1:9090,foo.bar
   ...

或者只是使用 (无聊到) 文件

.. code-block:: ini

   [uwsgi]
   req-logger = file:/tmp/reqlog
   logger = file:/tmp/errlog
   ...

链式重载
***************

当位于lazy/lazy_apps模式时，你可以简单地销毁一个worker，以迫使它重载应用代码。

一个新的名为“链式重载”的重载系统，允许你有时重载一个worker (与批量销毁所有worker这种标准方式相对)

只能通过"touch"来触发链式重载: --touch-chain-reload <file>

卸载改进
***********************

卸载是在uWSGI 1.4出现的，并且是最受喜爱的特性之一。在1.9种，我们添加了一个新的引擎： "write"，它允许你卸载磁盘上文件的写。有一个通用函数api uwsgi.offload()，它允许应用访问卸载引擎。所有发送静态文件的uWSGI部分 (包括语言特定的实现，像WSGI
wsgi.file_wrapper) 都得到了扩展，以在可用的情况下自动使用卸载。这意味着，你可以使用你的框架的方式来提供静态文件服务，而不会损失太多的性能，并且(更重要的是)不会阻塞你的worker。


更好的静态文件管理/服务
**************************************

uWSGI 1.9在提供静态文件服务方面有了许多改进。

你或许想要看看： :doc:`StaticFiles`

对于系统管理员来说，最有趣的新特性之一是能够使用
uWSGI新一代的缓存 (见下) 来存储请求 -> 绝对路径映射

新一代的缓存子系统 (cache2)
*******************************************

uWSGI缓存子系统已经被完全重写，变成一个更通用的内存键/值存储。旧的缓存子系统已在它之上进行了重建，现在更多是一个通用的“web缓存”系统。新的缓存子系统允许你控制内存存储的所有方面，从哈希算法到块的数量。

现在，对于每个实例，你可以有多个缓存 (由名称标识)
选项

.. code-block:: ini

   [uwsgi]
   cache2 = name=mycache,items=100
   cache2 = name=faster,items=200,hash=murmur2,keysize=100,blocksize=4096
   cache2 = name=fslike,items=1000,keysize=256,bitmap=1,blocks=2000,blocksize=8192
   ...


在这个例子中，我们创建了3个缓存：mycache, faster和fslike.

第一个是标准的旧式缓存，能够存储100个项，最大是64k，键的大小限制为2048字节，使用djb33x哈希算法。第二个使用murmur2哈希算法，每个键最大可以是100字节，可以存储200个项，最大是4k。最后一个就像文件系统一样，其中，每个项可以跨多个块。这意味着，fslike缓存可以为不同大小的对象节省大量的内存 (但它会比基于非bitmap的缓存更慢)

你在cache2中可以指定的选项如下：

``name`` 缓存名 (必须唯一) 必要

``items/max_items/maxitems`` 设置缓存剋存储的最大项数。必要

``blocksize`` 设置单个块的大小

``blocks`` 设置块数 (只在bitmap模式使用)

``hash`` 设置哈希算法，目前支持：djbx33盒murmur2

``hashsize/hash_size`` 设置哈希表大小 (默认为65536个项)

``keysize/key_size`` 设置键大小

``store`` 设置永久化存储缓存的文件名

``store_sync/storesync`` 设置频率 (以秒为单位)，其中，调用msync()来把缓存刷新至磁盘 (当在永久化模式时)

``node/nodes`` 新的缓存子系统可以通过udp包发送缓存更新。使用这个选项，设置一或多个 (使用 `;` 分隔) 发送更新的udp地址

``sync`` 将其设置为缓存服务器的地址。它的全部内容将会被拷贝到新的缓存 (将其用于初始化同步)

``udp/udp_servers/udp_server/udpservers/udpserver`` 绑定到特定的udp地址 (使用 `;` 分隔) ，监听缓存更新

``bitmap`` 启用bitmap模式 (设置其为1)

如果你问，为啥会存在这样低层次的调整，那么你必须考虑到，新的缓存子系统被用在大量的地方，因此，对于不同的需求，你或许想要不同的调整。以 :doc:`SSLScaling` 为例

旧的--cache-server选项已被移除。0.9.8添加的线程缓存服务器已经完全被新的非阻塞基础设施所代替。如果你加载"缓存（cache）"插件 (单件构建时默认启用)，那么将提供一个缓存服务器，它由worker管理。


更新文档可以看看这里 :doc:`Caching`

Legion子系统
********************

Legion子系统是uWSGI项目全新增加的一个系统。它取代了旧的集群子系统 (已在1.9删除)。它实现了一个quorum系统来管理集群环境中的共享资源。文档已经有了： :doc:`Legion`

Cygwin (windows) 支持
************************

通过cygwin POSIX仿真系统，可以在windows机器上变异uWSGI。事件系统使用简单的poll() (映射到cygwin上的select())，而锁引擎使用windows的mutex。尽管从我们的测试看来，它相当的稳固，但是我们认为该移植仍然是“实验性的”


高级异常子系统
*****************************

与语言独立的请求体管理一样，添加了一个异常管理系统。目前仅Python和
Ruby插件支持，允许对异常情况(例如对指定的异常进行重载)的语言无关处理。已对--catch-exception选项进行改进，以显示大量的有用信息。试试看 (尚在开发中!!!)。未来的开发将允许发送异常到诸如
Sentry或者Airbrake这样的系统。

SPDY, SSL和SNI
*****************

已经添加了令人兴奋的新特性到SSL系统和HTTP路由器中。

SPDY支持 (当前只是版本3) 将会受到大量用户关注，而SNI子系统则会受到系统管理员青睐。

初步文档可用

:doc:`SPDY`

:doc:`SNI`

HTTP路由器保持连接，自动分块，自动gzip和透明的websockets
***************************************************************************

许多用户已经开始在生产上使用HTTP/HTTPS/SPDY路由器了，因此，我们开始对其添加特性。记住，这只是一个路由器／代理，不允许任何I/O，因此你可能不能够丢掉你老的，但是工作良好的web服务器。

新选项：

``--http-keepalive`` 启用HTTP/1.1保持连接（keepalive）连接

``--http-auto-chunked`` 对于没有内容长度的后端响应 (或者已经启用块编码)，在块模式下转换输出，以维持keepalive连接

``--http-auto-gzip`` 如果uWSGI-Encoding头被设置为gzip，则自动gzip压缩内容，但是内容大小 (Content-Length/Transfer-Encoding) 和Content-Encoding不会被指定

``--http-websockets`` 自动检测websockets连接，以把请求处理器置于原始（raw）模式

SSL路由器 (sslrouter)
**************************

添加了一个新的corerouter，它与rawrouter的工作方式相同，但是它会终止ssl连接。该sslrouter可以使用sni来实现虚拟主机 (使用--sslrouter-sni选项)

Websockets api
**************

20Tab S.r.l. (一个开发HTML5浏览器游戏的公司) 为uWSGI赞助了一个快速的语言无关的websockets api的开发。该api目前处于一个非常良好的状态(也许比其他实现更快)。仍然需要完成文档，但是你可以看看下面的例子 (一个简单的echo):

https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.pl (perl)

https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.py (python)

https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.ru (ruby)

新的内部路由 (图灵完备？)
****************************************

内部路由子系统已经被重写，现在它是“可编程的”。你可以把它看成使用steroids (和goto ;)的apache mod_rewrite。仍然需要移植文档，但是这个新的系统允许你在线修改/过滤CGI变量和HTTP头，以及管理HTTP鉴权和缓存。

已更新的文档在这里 (仍在进行中) :doc:`InternalRouting`

Emperor ZMQ插件
******************

添加了一个新的imperial监控器，允许通过zeromq消息管理vassal：

https://uwsgi-docs.readthedocs.io/en/latest/ImperialMonitors.html#zmq-zeromq

通过统计信息服务器的完全检查
****************************************

现在，统计信息服务器为每个核导出了当前运行中的请求的所有请求变量，因此，它也工作在多线程模式下。这是检查你的实例在做什么，以及如何做的一种棒棒哒的方式。在未来，会扩展uwsgitop，从而实时显示当前运行的请求。

Nagios插件
*************

使用nagios插件发送Ping请求将不再算到应用请求统计数据中。这意味着，如果应用使用--idle选项，那么启用的nagios
ping将不再阻止应用变成idle状态，因此，从1.9起，当使用nagios插件的时候，应该禁用
--idle。否则，应用可能刚好在nagios ping请求之前置于idle状态，当ping抵达的时候，它需要从idle中醒来，而这可能花费的时间会比ping超时时间更长，从而引发nagios告警。

移除和弃用特性
*******************************

- 已移除--app选项。要在指定的挂载点加载应用，请使用--mount选项

- 已移除--static-offload-to-thread选项。使用更灵活的--offload-threads

- 已移除grunt模式。要实现相同的行为，只需使用线程，或者直接调用fork()和uwsgi.disconnect()

- 已移除send_message/recv_message api (使用语言提供的函数)

进行中的，问题和回归
***********************************

对于大量预期特性，我们错过了时间：

- SPNEGO支持，这是一个内部路由指令，用来实现SPNEGO认证支持

- Ruby 1.9 fibers支持已经被重写了，但是需要测试

- Erlang支持并为获得所需关注，非常有可能会延迟到2.0

- 异步休眠API未完成

- 仍然未实现SPDY推送

- RADIUS和LDAP内部路由指令未实现

- channel子系统 (为方便的websockets通信所需) 仍然未实现

除此之外，我们还有一些将会在接下来的小版本中解决的问题：

- --lazy模式没用了，现在它像--lazy-apps，但是在SIGHUP上只有workers-reload策略

- 看起来JVM与协程引擎不能很好的工作，或许我们应该为它添加一个检查

- Solaris和类Solaris系统并未获得严格测试

特别鸣谢
**************

许多用户/开发者在1.9开发周期中提供了帮助。我们想要特别感谢：

Łukasz Mierzwa (fastrouters伸缩性测试)

Guido Berhoerster (让内部路由新天网)

Riccardo Magliocchetti (静态分析)

André Cruz (HTTPS和gevent battle测试)

Mingli Yuan (Clojure/Ring支持和测试套件)






