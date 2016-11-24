日志编码器
============

uWSGI 1.9.16开始就有了“日志编码”功能。

一个编码器接收一个日志行，然后返回它的一个“转换”。

可以通过插件添加编码器，并可以链式（一个编码器的输出将是随后的编码器的输入，依次递推）启动编码器。

.. code-block:: ini

   [uwsgi]
   ; send logs to udp address 192.168.173.13:1717
   logger = socket:192.168.173.13:1717
   ; before sending a logline to the logger encode it in gzip
   log-encoder = gzip
   ; after gzip add a 'clear' prefix to easy decode
   log-encoder = prefix i am gzip encoded
   ...


   
   
使用这个配置，日志服务器将会接收 "i am gzip encoded"字符串，后面跟着以gzip编码的真正的日志消息

日志编码器的语法如下：

log-encoder = <encoder>[ args]

而args (如果有的话)由单个空格分隔

请求日志 VS 标准输出/标准错误
*****************************

--log-encoder选项只编码标准输出/标准错误日志。

如果你想要编码请求日志，则使用--log-req-encoder：

.. code-block:: ini

   [uwsgi]
   ; send request logs to udp address 192.168.173.13:1717
   req-logger = socket:192.168.173.13:1717
   ; before sending a logline to the logger encode it in gzip
   log-req-encoder = gzip
   ; after gzip add a 'clear' prefix to easy decode
   log-req-encoder = prefix i am gzip encoded
   ...
   
路由编码器
****************

日志路由允许基于正则表达式发送每个日志行到不同的日志引擎。你也可以带编码器使用相同的系统：

.. code-block:: ini

   [uwsgi]
   ; by default send logs to udp address 192.168.173.13:1717
   logger = socket:192.168.173.13:1717
   ; an alternative logger using the same address
   logger = secondlogger socket:192.168.173.13:1717
   ; use 'secondlogger' for the logline containing 'uWSGI'
   log-route = secondlogger uWSGI
   ; before sending a logline to the 'secondlogger' logger encode it in gzip
   log-encoder = gzip:secondlogger
   ...

   
``Core`` 编码器
*****************

在uwsgi的'core'中，由以下可用的编码器：

``prefix`` 添加一个原始前缀到每个日志消息

``suffix`` 添加一个原始后缀到每个日志消息

``nl`` 添加一个换行符字符到每个日志消息

``gzip`` 用gzip压缩每个消息 (需要zlib)

``compress`` 用zlib压缩每个消息 (需要zlib)

``format`` 应用指定的格式到每个日志消息：

.. code-block:: ini

   [uwsgi]
   ...
   log-encoder = format [FOO ${msg} BAR]
   ...
   
``json`` 像 ``format`` ，但是每个变量都是json转义的

.. code-block:: ini

   [uwsgi]
   ...
   log-encoder = json {"unix":${unix}, "msg":"${msg}"}
   ...
   
可用以下变量 (用于format和json)：

``${msg}`` 原始日志消息 (移除换行符)

``${msgnl}`` 原始日志消息 (带换行符)

``${unix}`` 当前unix时间

``${micros}`` 当前unix时间，以微秒为单位

``${strftime:xxx}`` strftime，使用xxx格式：



.. code-block:: ini

   [uwsgi]
   ...
   ; we need to escape % to avoid magic vars nameclash
   log-encoder = json {"unix":${unix}, "msg":"${msg}", "date":"${strftime:%%d/%%m/%%Y %%H:%%M:%%S}"}
   ...

  
 ``msgpack`` 编码器
***********************

这是官方添加到uWSGI源的第一个日志编码器插件。它允许以msgpack格式来编码日志行 (http://msgpack.org/)。

其语法是相当多变的，因为它被开发来添加任何信息到单一的包中

``log-encoder = msgpack <format>``

格式相当复杂，因为它是整个包中的单个项列表。

例如，如果你想要编码 {'foo':'bar', 'test':17} 字典，那么需要这样读取它：

a map of 2 items | the string foo | the string bar | the string test | the integer 17

总共5项。

一个更复杂的结构 {'boo':30, 'foo':'bar', 'test': [1,3,3,17.30,nil,true,false]}

将是

a map of 3 items | the string boo | the number 30| the string foo| the string bar | the string test | an array of 7 items | the integer 1 | the integer 3 | the integer 3 | the float 17.30 | a nil | a true | a false

<format>字符串是这种方式的一种表示：

.. code-block:: sh
   
   map:2|str:foo|str:bar|str:test|int:17

|是每个项的分隔符。冒号前的字符串是项类型，后面跟着可选的参数

支持以下项类型：

``map`` 一个字典，参数是项个数

``array`` 一个数组，参数是项个数

``str`` 一个字符串，参数是字符串本身

``bin`` 一个字节数组，参数是二进制流本身

``int`` 一个整型，参数是数字

``float`` 一个浮点数，参数是数字

``nil`` undefined/NULL

``true`` 布尔类型 TRUE

``false`` 布尔类型 FALSE

除了msgpack类型之外，可以用一系列的动态类型：

``msg`` 将日志行转换成移除换行符的msgpack字符串

``msgbin`` 将日志行转换成移除换行符的msgpack字节数组

``msgnl`` 将日志行转换成msgpack字符串 (包含换行符)

``msgbin`` 将日志行转换成msgpack字节数组 (包含换行符)

``unix`` 转换成unix时间的整数格式

``micros`` 转换成unix时间的整数格式，以微秒为单位

``strftime`` 使用strftime语法转换成一个字符串。strftime格式就是参数

下面是一个通过udp发送日志行到logstash服务器的例子：


(logstash调试配置):

.. code-block:: c

   input {
        udp {
                codec =>   msgpack {}
                port => 1717
        }
   }
   output {
        stdout { debug => true }
        elasticsearch { embedded => true }
   }


.. code-block:: ini

   [uwsgi]
   logger = socket:192.168.173.13:1717
   log-encoder = msgpack map:4|str:message|msg|str:hostname|str:%h|str:version|str:%V|str:appname|str:myapp
   ...
   
这将生成以下结构：

.. code-block:: js

   {
      "message": "*** Starting uWSGI 1.9.16-dev-29d80ce (64bit) on [Sat Sep  7 15:04:32 2013] ***",
      "hostname": "unbit.it",
      "version": "1.9.16-dev",
      "appname": "myapp"
   }
   
它将被存储到elasticsearch

注意事项
*********

编码器自动启用--log-master

要获得最佳性能，考虑使用--threaded-logger为日志发送分配一个线程
