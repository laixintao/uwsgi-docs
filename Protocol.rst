uwsgi协议
==================

uwsgi协议是uWSGI服务器使用的本地协议。

它是一个二进制协议，可以携带任何类型的数据。一个uwsgi分组的头4个字节描述了这个分组包含的数据类型。

**每个uwsgi请求生成一个uwsgi格式的响应。**

甚至是web服务器处理程序也遵循这个规则，因为一个HTTP响应是一个有效的uwsgi包 (看下 ``modifier1`` = 72)。

该协议主要通过TCP工作，但是master经常可以为 :doc:`SNMP` 或者集群管理/消息请求绑定到一个UDP单播/组播。

SCTP支持正在开发中。

uwsgi包头
-------------------

.. code-block:: c

    struct uwsgi_packet_header {
        uint8_t modifier1;
        uint16_t datasize;
        uint8_t modifier2;
    };


除非另有指定的 ``datasize`` 值包含了包体大小 (小端的16位)。

包描述
-------------------


.. list-table::
   :header-rows: 1

   * - ``modifier1``
     - ``datasize``
     - ``modifier2``
     - ``packet type``
   
   * - 0
     - WSGI块变量大小 (HTTP请求体除外)
     - 0
     - 标准WSGI请求，后面跟着HTTP请求体
   
   * - 1
     - 为UNBIT保留
     -
     -
   
   * - 2
     - 为UNBIT保留
     -
     -

   * - 3
     - 为UNBIT保留
     -
     -
        
   * - 5
     -  :doc:`PSGI <Perl>` 块变量 (HTTP请求体除外)的大小
     - 0
     - 标准 :doc:`PSGI <Perl>` 请求，后面跟着HTTP请求体
   
   * - 6
     - LUA WSAPI块变量 (HTTP请求体除外) 大小
     - 0
     - 标准LUA/WSAPI请求，后面跟着HTTP请求体
   
   * - 7
     - RACK块变量 (HTTP请求体除外) 大小
     - 0
     - 标准RACK请求，后面跟着HTTP请求体
   
   * - 8
     - JWSGI/Ring块变量 (HTTP请求体除外) 大小
     - 0
     - 用于 :doc:`JWSGI` 和 :doc:`Ring` 的标准JVM请求，后面跟着HTTP请求体
   
   * - 9
     - CGI块变量 (HTTP请求体除外) 大小
     - 0
     - 标准 :doc:`CGI` 请求，后面跟着HTTP请求体
   
   * - 10
     - 块变量大小
     - 0- 255
     -  管理接口请求：由modifier2指定的setup标志。关于管理标志列表，查看ManagementFlag
   
   * - 14
     - CGI块变量 (HTTP请求体除外) 大小
     - 0
     - 标准 :doc:`PHP` 请求，后面跟着HTTP请求体

   * - 15
     - Mono ASP.NET块变量 (HTTP请求体除外) 大小
     - 0
     - 标准 :doc:`Mono` 请求，后面跟着HTTP请求体
   
   * - 17
     - Spooler块变量大小
     - 0- 255
     - :doc:`Spooler` 请求，块变量会被转换成一个字典/哈希/表，然后被传递给spooler可调用对象。当前忽略第二个modifier.

   * - 18
     - CGI块变量的大小
     - 0-255
     - 直接调用到类C符号

   * - 22
     - 代码字符串大小
     - 0- 255
     - 原始代码评估。由modifier2选定解释器。0是Python, 5是Perl。它不会返回一个有效的uwsgi响应，但会返回一个原始字符串 (可能是一个HTTP响应)

   * - 23
     - CGI变量大小
     - 0- 255
     - 调用 :doc:`XSLT`

   * - 24
     - CGI变量大小
     - 0- 255
     - 调用 :doc:`V8`

   * - 25
     - CGI变量大小
     - 0- 255
     - 调用 :doc:`GridFS`
   
   * - 26
     - CGI变量大小
     - 0- 255
     - 调用 :doc:`GlusterFS`
     
   * - 27
     - 0
     - 0- 255
     - 调用modifier2字段指定的 :doc:`FastFuncs` 
   
   * - 28
     - 0
     - 0- 255
     - 调用 :doc:`Rados`
   
   * - 30
     - WSGI块变量 (HTTP请求体除外) 大小
     - 0 (如果定义，那么块变量的大小是24位，目前，没有web服务器处理器支持这个特性)
     - 标准WSGI请求，后面跟着HTTP请求体。会自动修改PATH_INFO，将SCRIPT_NAME从中删除
   
   * - 31
     - 块变量大小
     - 0- 255
     - 一般消息传递 (保留)
   
   * - 32
     - 字符数组大小
     - 0- 255
     - 字符数组传递 (保留)
   
   * - 33
     - marshal对象大小
     - 0- 255
     - 编组/序列化对象传递 (保留)
   
   * - 48
     - snmp specific
     - snmp specific
     - identify a SNMP request/response (mainly via UDP)
   
   * - 72
     - chr(TT)
     - chr(P)
     - Corresponds to the 'HTTP' string and signals that this is a raw HTTP response.
   
   * - 73
     - announce message size (完备性检查)
     - announce type (0 = hostname)
     - announce message
   
   * - 74
     - 多播消息大小 (完备性检查)
     - 0
     - array of chars; a custom multicast message managed by ``uwsgi.multicast_manager``
   
   * - 95
     - cluster membership dict size
     - ``action``
     - add/remove/enable/disable node from a cluster. Action may be 0 = add, 1 = remove, 2 = enable, 3 = disable. Add action requires a dict of at least 3 keys: ``hostname``, ``address`` and ``workers``
   
   * - 96
     - 日志消息大小
     - 0
     - 远程日志记录 (集群/多播/单播)
   
   * - 97
     - 0
     - 0, 1
     - 粗鲁重载请求 (0 请求 - 1 确认)
   
   * - 98
     - 0
     - 0, 1
     - 优雅重载请求 (0 请求 - 1 确认)
   
   * - 99
     - size of options dictionary (if response)
     - 0, 1
     - request configuration data from a uwsgi node (even via multicast)
   
   * - 100
     - 0
     - 0, 1
     - PING- PONG if modifier2 is 0 it is a PING request otherwise it is a PONG (a response). Useful for cluster health- check
   
   * - 101
     - 包大小
     - 0
     - ECHO服务

   * - 109
     - size of clean payload
     - 0 to 255
     - legion msg (UDP, the body is encrypted) 
   
   * - 110
     - 负载大小
     - 0 to 255
     - ``uwsgi_signal`` framework (payload is optional), modifier2 is the signal num 
   
   * - 111
     - 包大小
     - 0, 1, 2, 3
     - Cache operations. 0: read, 1: write, 2: delete, 3: dict_based

   * - 123
     - 包大小
     - -
     - special modifier for signaling corerouters about special conditions
   
   * - 173
     - 包大小
     - 0, 1
     - RPC. The packet is an uwsgi array where the first item is the name of the function and the following are the args (if ``modifier2`` is 1 the RPC will be 'raw' and all of the response will be returned to the app, uwsgi header included, if available.
   
   * - 200
     - 0
     - 0
     - Close mark for persistent connections
   
   * - 224
     - 包大小
     - 0
     - 订阅包。见SubscriptionServer
   
   * - 255
     - 0
     - 0- 255
     - 一般响应。请求相关。例如，一个spooler响应为一个失败的spool设置0，而为一个成功的spool设置1

uwsgi变量
--------------

uwsgi块变量表示一个字典/哈希。hash. 每个键值对都以这种方式进行编码：

.. code-block:: c

    struct uwsgi_var {
        uint16_t key_size;
        uint8_t key[key_size];
        uint16_t val_size;
        uint8_t val[val_size];
    }
