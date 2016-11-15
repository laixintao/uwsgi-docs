uWSGI 1.9.12
============

更新日志[20130605]

错误修复
^^^^^^^^

- 卸载缓存写入将会反悔正确的状态码，而不是202
- 你现在可以设置TMPDIR环境变量，来控制临时文件的路径了 (这为那些没有/tmp路径控制权的用户修复了一个老问题)
- 修复amqp imperial监控器上的一个编译错误
- 当cron被报告到统计信息服务器的时候，会正确的被转义
- 修复上传大文件的时候fastcgi解析器的极端情况
- 修复对最新的cygwin的支持

新特性
^^^^^^^^^^^^

卸载响应
********************

看看下面的WSGI应用：

.. code-block:: python

   def application(environ, start_response):
       start_response('200 OK', [('Content-Type', 'text/plain')])
       return ['u' * 100000000]
       
它将生成大约100兆数据。worker花费在该请求上的98%的时间都是数据传输。由于整个响应是紧随请求之后的，因此，我们可以卸载该数据写入到一个线程里，然后立即释放worker (这样，它就可以处理新的请求了)。

100兆很大，但甚至是1MB都可以引发十几个poll()/write()系统调用，这会阻塞你的worker不少毫秒

多亏了在1.9.11中添加的“内存卸载”功能，实现它变得非常简单。

通过 :doc:`Transformations` 实现卸载

.. code-block:: ini

   [uwsgi]
   socket = :3031
   wsgi-file = myapp.py
   ; offload all of the application writes
   route-run = offload:
   
默认情况下，会将响应缓存至内存，直到它大小为1MB为止。之后，将会将其缓存至磁盘，接着卸载引擎会使用sendfile()。

你可以设置磁盘缓存后的限制 (以字节为单位)，传递一个参数给offload:

.. code-block:: ini

   [uwsgi]
   socket = :3031
   wsgi-file = myapp.py
   ; offload all of the application writes (buffer to disk after 1k)
   route-run = offload:1024
   
"offload"必须是该恋中的最后一个转换

.. code-block:: ini

   [uwsgi]
   socket = :3031
   wsgi-file = myapp.py
   ; gzip the response
   route-run = gzip:
   ; offload all of the application writes (buffer to disk after 1k)
   route-run = offload:1024
   
   
JWSGI和JVM改进
**************************

已扩展了JVM插件，来支持更多的扶助对象 (例如ArrayList)，而JWSGI现在可以被当成一个低级别的层次使用，来天津队更多基于JVM语言的支持。

JRuby集成是这类应用的第一个尝试。我们刚刚发布了一个JWSGI到Rack适配器，允许你在JRUBY之上适配运行
Ruby/Rack应用：

https://github.com/unbit/jwsgi-rack


关于Jython的一个类似的方法正在开发中

--touch-signal
**************

添加了一个新的touch选项，允许当touch一个文件的时候，引发一个uwsgi信号：

.. code-block:: ini

   [uwsgi]
   ...
   ; raise signal 17 on /tmp/foobar modifications
   touch-signal = /tmp/foobar 17
   ...

"pipe"卸载引擎
*************************

添加了一个允许从socket转移到客户端的新的卸载引擎。

它将自动用于新的router_memacached和router_redis插件


memcached路由改进
*****************************


你现在可以把响应存储到memcached中了 (正如你已经可以用uWSGI缓存做到的那样)

.. code-block:: ini

   [uwsgi]
   ...
   route = ^/cacheme memcachedstore:addr=127.0.0.1:11211,key=${REQUEST_URI}
   route = ^/cacheme2 memcachedstore:addr=192.168.0.1:11211,key=${REQUEST_URI}foobar
   ...
   
显然，你也可以获取它们

.. code-block:: ini

   [uwsgi]
   ...
   route-run = memcached:addr=127.0.0.1:11211,key=${REQUEST_URI}
   ...
   
当前，会在默认配置文件中内置memcached路由器

新的redis路由器
********************

基于memcached路由器，添加了一个redis路由器。它的工作方式相同：


.. code-block:: ini

   [uwsgi]
   ...
   route = ^/cacheme redisstore:addr=127.0.0.1:6379,key=${REQUEST_URI}
   route = ^/cacheme2 redisstore:addr=192.168.0.1:6379,key=${REQUEST_URI}foobar
   ...
   
... 以及获取值

.. code-block:: ini

   [uwsgi]
   ...
   route-run = redis:addr=127.0.0.1:6379,key=${REQUEST_URI}
   ...

默认内置redis路由器

"hash"路由器
*****************

这个特别的路由动作允许你哈希一个字符串，并从列表中返回一个值 (由哈希键索引)。

看看下面的列表：

127.0.0.1:11211

192.168.0.1:11222

192.168.0.2:22122

192.168.0.4:11321

以及一个字符串：

/foobar

我们使用djb33x算法来对字符串/foobar进行哈希，然后应用模4 (项列表的大小) 到结果中。

结果为"1"，所以，将会获得列表中的第二个项 (显然，索引是从零开始的)。

你认出这个模式了吗？

是的，它是一种标准的将项分布到多个服务器上的方式 (例如，memcached客户端用它已经有很长的一段时间了)。

这个哈希路由器公开了这个系统，允许你分布项到你的redis/memcached服务器上，或者做其他有趣的事。

这是redis的用法示例：

.. code-block:: ini

   [uwsgi]
   ...
   ; hash the list of servers and return the value in the MYNODE var
   route = ^/cacheme_as/(.*) hash:items=127.0.0.1:11211;192.168.0.1:11222;192.168.0.2:22122;192.168.0.4:11321,key=$1,var=MYNODE
   ; log the result
   route = ^/cacheme_as/(.*) log:${MYNODE} is the choosen memcached server !!!
   ; use MYNODE as the server address
   route = ^/cacheme_as/(.*) memcached:addr=${MYNODE},key=$1
   ...
   
你甚至可以从uWSGI支持的那些哈希算法中选择使用的算法

.. code-block:: ini

   [uwsgi]
   ...
   ; hash the list of servers with murmur2 and return the value in the MYNODE var
   route = ^/cacheme_as/(.*) hash:algo=murmur2,items=127.0.0.1:11211;192.168.0.1:11222;192.168.0.2:22122;192.168.0.4:11321,key=$1,var=MYNODE
   ; log the result
   route = ^/cacheme_as/(.*) log:${MYNODE} is the choosen memcached server !!!
   ; use MYNODE as the server address
   route = ^/cacheme_as/(.*) memcached:addr=${MYNODE},key=$1
   ...

默认会编译该router_hash插件

可用性
^^^^^^^^^^^^

uWSGI 1.9.12于20130605发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.12.tar.gz
