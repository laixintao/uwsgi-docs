uWSGI 2.0.5
===========

更新日志 [20140601]

错误修复
--------

- 修复Lua插件中对重复header的支持 (关于作者：tizoc)
- 修复OpenBSD和NetBSD上对嵌入配置的支持
- 基于cURL的插件的多个修复 (关于作者：Yu Zhao)
- 修复基于毫秒的等待
- 修复共享区域的poller
- 修复统计信息服务器中的JSON编码器
- 修复FastCGI解析器并实现EOF管理 (关于作者： Jeff Trawick)
- improved fast on-demand mode
- 排除对静态文件的avg_rt计算
- 修复uwsgi内部路由器中的变量支持
- 修复websockets + keepalive顺序
- 禁用基于协程的循环引擎中的SIGPIPE管理
- 修复32位系统中的64位共享区域管理
- honor chmod/chown-socket in fd0 mode
- hack to avoid Safari on iOS making a mess with keepalive
- 当同时使用--logto和--log2时的日志设置 (关于作者：Łukasz Mierzwa)
- 修复mule_get_msg EAGAIN
- signal_pidfile返回正确的错误码code
- 修复OSX上的asyncio

新特性
------------

mule进程的优雅重载 (关于作者：Paul Egan)
******************************************************

现在会发送 `SIGHUP` 给mule，而不是直接杀死它们。
你可以随意在代码中捕获该信号。
如果一个mule在允许的“宽恕期” (`--mule-reload-mercy`, 默认是60秒)内没有死掉，那么将发送SIGKILL。

`return` 路由动作 (关于作者：Yu Zhao)
******************************************

这个新的动作将允许用户编写简化的"break"子句。

例如，"return:403"等价于"break:403 Forbidden"，带有响应体"Forbidden"。

这个响应体对于告诉终端用户出了什么事是非常有用的。

--emperor-no-blacklist
**********************

这个新的选项完全禁用Emperor的黑名单子系统。

Icecast2协议辅助器
*************************

即将到来的unbit.com项目之一是一个基于uWSGI的音频/视频流媒体服务器。

这个插件 (应在Europython 2014期间发布) 已支持Icecast2协议。

已添加了一堆的补丁到这个HTTP路由器，来支持Icecast2协议。

例如， ``--http-manage-source`` 选项运行HTTP路由器遵循 `SOURCE` 方法请求，自动将它们置于原始模式。

--metrics-no-cores, --stats-no-cores, --stats-no-metrics
********************************************************

当你拥有上百（或上千）个异步核时，公开它们的度量也许会非常地慢。

已添加了3个新的选项，允许你禁用核相关的度量的生成，以及最终导致禁用它们在统计数据服务器上的使用。

共享区域改进
***********************

共享区域API在持续改进中。最后的补丁包括直接从命令行进行内存映射文件（或设备） (`mmap`) 的支持。

一个测试该特性的有趣的方式是映射树莓派的BCM2835内存。这个小小的例子让你读取RPi系统的计时器。

.. code-block:: sh

   uwsgi --sharedarea file=/dev/mem,offset=0x20003000,size=4096 ...
   
现在，你可以从第一个（基于0）共享区域读取一个64位的值：

.. code-block:: python

   # read 64bit from 0x20003004
   timer = uwsgi.sharedarea_read64(0, 0x04)
   
(显然，在读取和写入树莓派内存时，要小心。一个错误很可能就把整个系统搞挂了！)

UWSGI_GO_CHEAP_CODE
*******************

这个退出码 (15) 可以由worker引发，来告诉master不要重新生成它。

对http路由器的PROXY1支持 (关于作者：bgglenn)
*****************************************************

选项 ``--http-enable-proxy-protocol`` 允许HTTP路由器理解PROXY1协议请求，例如那些由Haproxy或者 or Amazon Elastic Load Balancer (ELB)发起的请求。

用于度量的reset_after_push (关于作者：Babacar Tall)
****************************************************

这个度量属性确保度量被推送到外部系统(例如Carbon或者StatsD)后，度量值被重置为0 (或者它硬编码的 `initial_value`)。

setremoteaddr
*************

这个新的路由选项允许你完全覆盖由协议处理器检测到的 `REMOTE_ADDR` ：

.. code-block:: ini

   [uwsgi]
   ; treat all requests as local
   route-run = setremoteaddr:127.0.0.1

`resolve` 选项
********************

有些uWSGI选项 (或者插件) 并不会自动解析DNS名到IP地址。这个选项允许你映射一个占位符到一个字符串的DNS解析结果：

.. code-block:: ini

   [uwsgi]
   ; place the dns resolution of 'example.com' in the 'myserver' placeholder
   resolve = myserver=example.com
   ; %(myserver) would now be 93.184.216.119
   subscribe2 = server=%(myserver),key=foobar

可用性
-------------

uWSGI 2.0.5已于[20140601]发布，你可以从这里下载：

http://projects.unbit.it/downloads/uwsgi-2.0.5.tar.gz
