日志记录
==============

.. seealso:: :doc:`LogFormat`

基本的日志记录
-------------

uWSGI中最基本的日志记录的形式是将请求、错误和信息消息写到标准输出/标准错误。这是默认配置中使用的方式。日志重定向的最基本的形式是 ``--logto`` /
``--logto2`` / ``--daemonize`` 选项，它们允许你重定向日志到文件中。

基本记录日志到文件
^^^^^^^^^^^^^^^^^^^^^^

要将日志写到文件中，而不是标准输出/标准错误，则使用 ``--logto`` ，或者同时守护uWSGI， ``--daemonize`` 。

.. code-block:: sh

    ./uwsgi -s :3031 -w simple_app --daemonize /tmp/mylog.log
    ./uwsgi -s :3031 -w simple_app --logto /tmp/mylog.log
    # logto2 only opens the log file after privileges have been dropped to the specified uid/gid.
    ./uwsgi -s :3031 -w simple_app --uid 1001 --gid 1002 --logto2 /tmp/mylog.log

基本日志记录 (连接UDP模式)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用UDP日志记录，你可以集中集群日志记录，或者重定向日志的持久化到其他机器上，以卸载磁盘I/O。UDP日志记录可以工作在守护模式和互动模式下。UDP日志记录是在连接socket模式下进行的，因此，UDP服务器必须在uWSGI启动前可用。对于更原始的方法（在未连接模式下工作），见socket日志记录部分。

要启用连接UDP模式，则传递UDP服务器的地址到 ``--daemonize``/``--logto`` 选项：

.. code-block:: sh

    ./uwsgi -s :3031 -w simple_app --daemonize 192.168.0.100:1717
    ./uwsgi -s :3031 -w simple_app --logto 192.168.0.100:1717

这将会重定向所有的标准输出/标准错误数据到192.168.0.100，端口1717上的UDP socket。现在，你需要一个UDP服务器，它将管理你的UDP消息。你可以使用netcat，甚至是uWSGI：

.. code-block:: sh

    nc -u -p 1717 -s 192.168.0.100 -l
    ./uwsgi --udp 192.168.0.100:1717

第二种方式更有用一点，因为它将打印每条消息的来源 (ip:port)。在多个uWSGI服务器将日志记录在同一个UDP服务器上的情况下，它将让你区分服务器。当然，你可以编写你自己的应用来管理/过滤/保存通过UDP接收到的日志。


可插拔记录器
-----------------

uWSGI还支持可插拔记录器，这让你在何处以及如何记录更具灵活性。取决于于你的uWSGI构建的配置，一些记录器可能/可能没有用。一些也许要求作为插件加载。要找出你的构建中哪些插件可用，请带 ``--logger-list`` 调用uWSGI。要设置一个可插拔记录器，则使用 ``--logger`` 或者
``--req-logger`` 选项。 ``--logger`` 将为每条消息设置一个记录器，而 ``--req-logger`` 将会为请求信息消息设置一个记录器。

这是语法：

.. code-block:: sh

    --logger <plugin>[:options]
    --logger "<name> <plugin>[:options]" # The quotes are only required on the command line -- config files don't use them

只要你喜欢，你可以设置尽可能多的记录器。命名插件用于日志路由，下面是使用纯文本文件分割请求/错误日志记录的一个非常简单的例子。

.. code-block:: ini

    [uwsgi]
    req-logger = file:/tmp/reqlog
    logger = file:/tmp/errlog

日志路由
-----------

默认情况下，所有的日志行会被发送到所有声明的记录器。如果你不想要这样，那么你可以使用 ``--log-route`` (以及用于请求记录器的 ``--log-req-route`` )，指定一个正则表达式来路由某些日志消息到不同的目的地。

例如：

.. code-block:: ini

    [uwsgi]
    logger = mylogger1 syslog
    logger = theredisone redislog:127.0.0.1:6269
    logger = theredistwo redislog:127.0.0.1:6270
    logger = file:/tmp/foobar # This logger will log everything else, as it's not named
    logger = internalservererror file:/tmp/errors
    # ...
    log-route = internalservererror (HTTP/1.\d 500)
    log-route = mylogger1 uWSGI listen queue of socket .* full

这将会记录每个500错误到/tmp/errors，而监听队列满错误将会出现在/tmp/foobar中。这有点类似于 :doc:`AlarmSubsystem` ，虽然告警通常更重，应该只用于紧急情况。

写日志到文件中
----------------

``logfile`` 插件 —— 默认嵌入。

.. code-block::

    uwsgi --socket :3031 --logger file:/tmp/uwsgi.log

写日志到socket
------------------

``logsocket`` 插件 —— 默认嵌入。

你可以使用 ``--logger socket:...`` (或者 ``--log-socket ...``) 写日志到一个未连接UNIX/UDP socket。

.. code-block:: sh

    uwsgi --socket :3031 --logger socket:/tmp/uwsgi.logsock

将会发送日志项到Unix socket ``/tmp/uwsgi.logsock``.

.. code-block:: sh

    uwsgi --socket :3031 --logger socket:192.168.173.19:5050

将会发送日志数据报到UDP地址192.168.173.19，端口是5050.你也可以通过传递多播地址，多播日志到多个日志服务器上：

.. code-block:: sh

    uwsgi --socket :3031 --logger socket:225.1.1.1:1717

写日志到syslog
-----------------

``logsyslog`` 插件 —— 默认嵌入。

 ``logsyslog`` 插件路由日志到Unix标准的syslog中。你可以传递一个用于发送的可选的ID，作为日志项的"facility"参数。

.. code-block:: sh

    uwsgi --socket :3031 --logger syslog:uwsgi1234

或者

.. code-block:: sh

    uwsgi --socket :3031 --logger syslog:uwsgi1234,local6

发送到local6 facility


写日志到远程syslog
------------------------

``logrsyslog`` 插件 —— 默认嵌入。


``logrsyslog`` 插件路由日志到位于远程服务器的Unix标准syslog上。除了远程syslog服务器的address+port外，你可以传递一个用于发送的可选的ID，作为日志项的"facility"参数。

.. code-block:: sh

    uwsgi --socket :3031 --logger rsyslog:12.34.56.78:12345,uwsgi1234

Redis记录器
------------

``redislog`` 插件 —— 默认嵌入。

默认情况下， ``redislog`` 插件将会“发布”每个日志行到一个redis pub/sub队列中。该记录器插件的语法如下：

.. code-block:: sh

    --logger redislog[:<host>,<command>,<prefix>]

默认， ``host`` 会被映射到 ``127.0.0.1:6379`` ， ``command`` 会被映射到"publish uwsgi" 并且 ``prefix`` 是空的。要发布到一个名为foobar的队列，则使用 ``redislog:127.0.0.1:6379,publish foobar`` 。Redis日记记录不只限于pub/sub。例如，你可以推送项到一个列表中，如下所示。

.. code-block:: sh

    --logger redislog:/tmp/redis.sock,rpush foo,example.com

当将一个日志行写入到一个远程服务器的时候，一个错误的场景会导致阻塞master，好的办法是使用 ``--threaded-logger`` 来卸载日志写入到第二个线程中。

MongoDB记录器
--------------

``mongodblog`` 插件 —— 默认嵌入。.

MongoDB日志记录(``mongodblog``)的记录器语法是

.. code-block:: sh

    --logger mongodblog[:<host>,<collection>,<node>]

其中， ``host`` 是MongoDB实例的地址 (默认 ``127.0.0.1:27017``)， ``collection`` 命名要写入日志行的集合 (默认是 ``uwsgi.logs``)，而 ``node`` 是用于实例发送日志的身份字符串 (默认是server hostname)。

.. code-block:: sh

    --logger mongodblog

将会用默认值运行记录器，而

.. code-block:: sh

    --logger mongodblog:127.0.0.1:9090,foo.bar

将会写入日志到mongodb服务器127.0.0.1:9090的集合 ``foo.bar`` 中，使用默认的节点名。就如Redis记录器一样，卸载日志写入到一个专用的线程是个好的选择。

.. code-block:: ini

    [uwsgi]
    threaded-logger = true
    logger = mongodblog:127.0.0.1:27017,uwsgi.logs_of_foobar
    # As usual, you could have multiple loggers:
    # logger = mongodblog:192.168.173.22:27017,uwsgi.logs_of_foobar
    socket = :3031

ZeroMQ日志记录
--------------

就如UDP日志记录一样，你可以通过ZeroMQ集中/分布日志记录。使用 ``ZMQ_PULL`` socket构建你的日志记录守护程序：

.. code-block:: python

    import zmq

    ctx = zmq.Context()

    puller = ctx.socket(zmq.PULL)
    puller.bind("tcp://192.168.173.18:9191")

    while True:
        message = puller.recv()
        print message,

现在，运行你的uWSGI服务器：

.. code-block:: sh

    uwsgi --logger zeromq:tcp://192.168.173.18:9191 --socket :3031 --module werkzeug.testapp:test_app

(``--log-zeromq`` 是这个记录器的一个别名。)


Crypto记录器 (插件)
----------------------

如果你在云服务商托管你的应用，并且不使用持久化存储，那么你也许想要发送你的日志到一个外部系统。然而，日志通常包含敏感信息，并应该将那些信息明文传输。
``logcrypto`` 插件记录器试图通过在发送之前加密每个日志包，然后通过UDP发送到一个能够解密它的服务器，从而解决这个问题。下一个例子将会发送每个日志包到192.168.173.22:1717上的一个UDP服务器，每个日志包都会用CBC模式中的Blowfish算法，通过密钥 ``ciaociao`` 进行加密。


.. code-block:: sh

   uwsgi --plugin logcrypto --logger crypto:addr=192.168.173.22:1717,algo=bf-cbc,secret=ciaociao -M -p 4 -s :3031

一个样例服务器如下：
https://github.com/unbit/uwsgi/blob/master/contrib/cryptologger.rb

Graylog2记录器 (插件)
------------------------

``graylog2`` 插件 —— 非默认编译。

这个插件将会发送日志的到一个Graylog2服务器上，使用Graylog2的原生GELF格式。

.. code-block:: sh

    uwsgi --plugin graylog2 --logger graylog2:127.0.0.1:1234,dsfargeg

Systemd记录器 (插件)
-----------------------

``systemd_logger`` 插件 —— 非默认编译。

这个插件将会将日志项写入到Systemd journal中。

.. code-block:: sh

    uwsgi --plugin systemd_logger --logger systemd


编写你自己的日志记录插件
-------------------------------

这个插件， ``foolog.c`` 将会把你的消息写入到由--logto/--daemonize指定的文件中，带有一个简单前缀，并使用vector IO。

.. code-block:: c

    #include <uwsgi.h>

    ssize_t uwsgi_foolog_logger(struct uwsgi_logger *ul, char *message, size_t len) {

            struct iovec iov[2];

            iov[0].iov_base = "[foo] ";
            iov[0].iov_len = 6;

            iov[1].iov_base = message;
            iov[1].iov_len = len;

            return writev(uwsgi.original_log_fd, iov, 2);
    }

    void uwsgi_foolog_register() {
            uwsgi_register_logger("syslog", uwsgi_syslog_logger);
    }

    struct uwsgi_plugin foolog_plugin = {
        .name = "foolog",
        .on_load = uwsgi_foolog_register,
    };
