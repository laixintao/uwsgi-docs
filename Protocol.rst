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
     - size of WSGI block vars (HTTP请求体除外)
     - 0
     - Standard WSGI request followed by the HTTP request body
   
   * - 1
     - reserved for UNBIT
     -
     -
   
   * - 2
     - reserved for UNBIT
     -
     -

   * - 3
     - reserved for UNBIT
     -
     -
        
   * - 5
     - size of :doc:`PSGI <Perl>` block vars (HTTP request body excluded)
     - 0
     - Standard :doc:`PSGI <Perl>` request followed by the HTTP request body
   
   * - 6
     - size of LUA WSAPI block vars (HTTP request body excluded)
     - 0
     - Standard LUA/WSAPI request followed by the HTTP request body
   
   * - 7
     - size of RACK block vars (HTTP request body excluded)
     - 0
     - Standard RACK request followed by the HTTP request body
   
   * - 8
     - size of JWSGI/Ring block vars (HTTP request body excluded)
     - 0
     - Standard JVM request for :doc:`JWSGI` and :doc:`Ring` followed by the HTTP request body
   
   * - 9
     - size of CGI block vars (HTTP request body excluded)
     - 0
     - Standard :doc:`CGI` request followed by the HTTP request body
   
   * - 10
     - size of block vars
     - 0- 255
     -  Management interface request: setup flag specified by modifier2. For a list of management flag look at ManagementFlag
   
   * - 14
     - size of CGI block vars (HTTP request body excluded)
     - 0
     - Standard :doc:`PHP` request followed by the HTTP request body

   * - 15
     - size of Mono ASP.NET block vars (HTTP request body excluded)
     - 0
     - Standard :doc:`Mono` request followed by the HTTP request body
   
   * - 17
     - size of Spooler block vars
     - 0- 255
     - :doc:`Spooler` request, the block vars is converted to a dictionary/hash/table and passed to the spooler callable. The second modifier is currently ignored.

   * - 18
     - size of CGI block vars
     - 0-255
     - direct call to c-like symbols   

   * - 22
     - size of code string
     - 0- 255
     - Raw Code evaluation. The interpreter is chosen by the modifier2. 0 is Python, 5 is Perl.
       It does not return a valid uwsgi response, but a raw string (that may be an HTTP response)

   * - 23
     - size of CGI vars
     - 0- 255
     - invoke the :doc:`XSLT`

   * - 24
     - size of CGI vars
     - 0- 255
     - invoke the :doc:`V8`

   * - 25
     - size of CGI vars
     - 0- 255
     - invoke the :doc:`GridFS`
   
   * - 26
     - size of CGI vars
     - 0- 255
     - invoke the :doc:`GlusterFS`
     
   * - 27
     - 0
     - 0- 255
     - call the :doc:`FastFuncs` specified by the modifier2 field
   
   * - 28
     - 0
     - 0- 255
     - invoke the :doc:`Rados`
   
   * - 30
     - size of WSGI block vars (HTTP request body excluded)
     - 0 (if defined the size of the block vars is 24bit le, for now none of the webserver handlers support this feature)
     - Standard WSGI request followed by the HTTP request body. The PATH_INFO is automatically modified, removing the SCRIPT_NAME from it
   
   * - 31
     - size of block vars
     - 0- 255
     - Generic message passing (reserved)
   
   * - 32
     - size of char array
     - 0- 255
     - array of char passing (reserved)
   
   * - 33
     - size of marshal object
     - 0- 255
     - marshalled/serialzed object passing (reserved)
   
   * - 48
     - snmp specific
     - snmp specific
     - identify a SNMP request/response (mainly via UDP)
   
   * - 72
     - chr(TT)
     - chr(P)
     - Corresponds to the 'HTTP' string and signals that this is a raw HTTP response.
   
   * - 73
     - announce message size (for sanity check)
     - announce type (0 = hostname)
     - announce message
   
   * - 74
     - multicast message size (for sanity check)
     - 0
     - array of chars; a custom multicast message managed by ``uwsgi.multicast_manager``
   
   * - 95
     - cluster membership dict size
     - ``action``
     - add/remove/enable/disable node from a cluster. Action may be 0 = add, 1 = remove, 2 = enable, 3 = disable. Add action requires a dict of at least 3 keys: ``hostname``, ``address`` and ``workers``
   
   * - 96
     - log message size
     - 0
     - Remote logging (clustering/multicast/unicast)
   
   * - 97
     - 0
     - 0, 1
     - brutal reload request (0 request -  1 confirmation)
   
   * - 98
     - 0
     - 0, 1
     - graceful reload request (0 request -  1 confirmation)
   
   * - 99
     - size of options dictionary (if response)
     - 0, 1
     - request configuration data from a uwsgi node (even via multicast)
   
   * - 100
     - 0
     - 0, 1
     - PING- PONG if modifier2 is 0 it is a PING request otherwise it is a PONG (a response). Useful for cluster health- check
   
   * - 101
     - size of packet
     - 0
     - ECHO service

   * - 109
     - size of clean payload
     - 0 to 255
     - legion msg (UDP, the body is encrypted) 
   
   * - 110
     - size of payload
     - 0 to 255
     - ``uwsgi_signal`` framework (payload is optional), modifier2 is the signal num 
   
   * - 111
     - size of packet
     - 0, 1, 2, 3
     - Cache operations. 0: read, 1: write, 2: delete, 3: dict_based

   * - 123
     - size of packet
     - -
     - special modifier for signaling corerouters about special conditions
   
   * - 173
     - size of packet
     - 0, 1
     - RPC. The packet is an uwsgi array where the first item is the name of the function and the following are the args (if ``modifier2`` is 1 the RPC will be 'raw' and all of the response will be returned to the app, uwsgi header included, if available.
   
   * - 200
     - 0
     - 0
     - Close mark for persistent connections
   
   * - 224
     - size of packet
     - 0
     - Subscription packet. see SubscriptionServer
   
   * - 255
     - 0
     - 0- 255
     - Generic response. Request dependent. For example a spooler response set 0 for a failed spool or 1 for a successful one

The uwsgi vars
--------------

The uwsgi block vars represent a dictionary/hash. 每个键值对都以这种方式进行编码：

.. code-block:: c

    struct uwsgi_var {
        uint16_t key_size;
        uint8_t key[key_size];
        uint16_t val_size;
        uint8_t val[val_size];
    }
