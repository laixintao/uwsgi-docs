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
     - snmp特定
     - snmp特定
     - 标识一个SNMP请求/响应 (主要通过UDP)
   
   * - 72
     - chr(TT)
     - chr(P)
     - 相对于'HTTP'字符串，标志这是一个原始的HTTP响应。
   
   * - 73
     - 宣告消息大小 (完备性检查)
     - 宣告类型(0 = 主机名)
     - 宣告消息
   
   * - 74
     - 多播消息大小 (完备性检查)
     - 0
     - 字符数组；一个自定义的多播消息，由 ``uwsgi.multicast_manager`` 管理
   
   * - 95
     - 集群成员字典大小
     - ``action``
     - 从一个机器添加/移除/启用/禁用节点。操作可以是 0 = 添加, 1 = 移除, 2 = 启用, 3 = 禁用。添加操作要求一个至少包含3个键的字典： ``hostname``, ``address`` 和 ``workers``
   
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
     - 选项字典的大小 (如果响应)
     - 0, 1
     - 来自一个uwsgi节点的请求配置数据 (即使通过多播)
   
   * - 100
     - 0
     - 0, 1
     - PING- PONG 如果modifier2为0，那么它是一个PING请求，否则它是一个PONG (一个响应)。对于集群健康检查有用
   
   * - 101
     - 包大小
     - 0
     - ECHO服务

   * - 109
     - 干净的有效负荷的大小
     - 0 to 255
     - legion消息 (UDP, 请求体被加密) 
   
   * - 110
     - 负载大小
     - 0 to 255
     - ``uwsgi_signal`` 框架 (有效负荷是可选的), modifier2是信号数字
   
   * - 111
     - 包大小
     - 0, 1, 2, 3
     - 缓存操作。0: 读, 1: 写, 2: 删除, 3: 基于字典

   * - 123
     - 包大小
     - -
     - 用于通知核心路由器特殊条件的特殊modifier
   
   * - 173
     - 包大小
     - 0, 1
     - RPC。包时一个uwsgi数组，其中，第一个项是函数名，而接下来是参数 (如果 ``modifier2`` 是1，那么RPC将会是'raw'，并且所有的响应将会被返回给应用，包含uwsgi头，如果可用的话。
   
   * - 200
     - 0
     - 0
     - 用于持久性连接的关闭标志
   
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
