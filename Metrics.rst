度量（Metrics）子系统
=====================

自1.9.19起可用。

uWSGI度量子系统让你可以管理来自应用的“数字”。

虽然在1.9开发周期内，缓存子系统获得了一些计算能力，但是度量子系统在设计上进行了优化，以存储数字并在其上应用函数。因此，与缓存子系统相比，它是一种更快的方式，并且需要消耗一小部分内存。

在启用后，度量子系统配置了大量的度量 (例如每核请求，内存使用，等等) ，但是，除此之外，你可以配置自己的度量，例如活跃用户数，或者，比方说，特定URL的访问数，以及应用或整个服务器的内存消耗。

要启用度量子系统，只需添加 ``--enable-metrics`` 到选项中，或者配置一个统计数据推送器 (见下)。

度量子系统是完全线程安全的。

默认情况下，uWSGI会创建许多度量 (大部分是已经已计划的)，因此，在添加你自己的度量之前，确保uWSGI并没有公开你所需要的度量。

度量名和oid
*********************

每个度量都必须要有一个名字 (只包含数字、字母、下划线、破折号和点) 和一个可选的 ``oid`` (用于映射一个度量到 :doc:`SNMP`)。

度量类型
************

在处理度量之前，你需要了解每个度量表示的各种类型:


COUNTER (type 0)
^^^^^^^^^^^^^^^^

这是一个一般增长的数字 (如请求数)。

GAUGE (type 1)
^^^^^^^^^^^^^^

这是一个可以动态增长或减少的数字 (如worker的使用内存，或者CPU负载)。

ABSOLUTE (type 2)
^^^^^^^^^^^^^^^^^

这是一个绝对数字，如整个服务器的内存，或者磁盘的大小。

ALIAS (type 3)
^^^^^^^^^^^^^^

这是一个指向另一个度量的虚拟度量。你可以用它来为已存在的度量赋予不同的名字。

度量收集器
*****************

一旦你定义了一个度量类型，那么你需要告诉uWSGI如何“收集”特定的度量。

有各种可用的收集器 (可以通过插件添加更多收集器)。

* ``ptr`` -- 该值是从内存指针收集来的
* ``file`` -- 该值是从文件收集来的
* ``sum`` -- 该值是其他度量的总和
* ``avg`` -- 计算孩子的算术平均值 (在1.9.20中添加)
* ``accumulator`` -- 总是增加孩子的值的总和到最后的值。看看下面这个例子。

   Round 1: child1 = 22, child2 = 17 -> metric_value = 39
   Round 2: child1 = 26, child2 = 30 -> metric_value += 56

* ``multiplier`` - 用指定的参数 (arg1n) 乘以孩子的值的总和。

   child1 = 22, child2 = 17, arg1n = 3 -> metric_value = (22+17)*3

* ``func`` - 每次调用指定的函数计算该值。
* ``manual`` - NULL收集器。必须使用度量API，从应用手动更新该值。

自定义度量
**************

你可以在你的应用中定义额外的度量。

``--metric`` 选项允许你添加更多度量。

它有两种语法：“简化的”和“键值”。

.. code-block:: sh

   uwsgi --http-socket :9090 --metric foobar
   
将会创建一个度量'foobar'，它的类型是“counter”，手工收集，并且无oid。

要创建高级度量，那么你需要键值方式:

.. code-block:: sh

   uwsgi --http-socket :9090 --metric name=foobar,type=gauge,oid=100.100.100
   
可以使用以下的键：

* ``name`` -- 设置度量名
* ``oid`` -- 设置度量oid
* ``type`` -- 设置度量类型，可以是 ``counter``, ``gauge``, ``absolute``, ``alias``
* ``initial_value`` -- 启动时设置度量为一个指定的值。
* ``freq`` -- 设置收集频率，以秒为单位 (默认为1)
* ``reset_after_push`` -- 在度量被推到后端后，重置度量为0 (或者配置的 ``initial_value``) (因此是每 ``freq`` 秒)
* ``children`` -- 映射孩子到该度量 (见下)
* ``alias`` -- 该度量将会是指定的度量的一个简单别名 (--metric name=foobar,alias=worker.0.requests,type=alias)
* ``arg1`` 到 ``arg3`` -- 基于字符串的参数 (见下)
* ``arg1n`` 到 ``arg3n`` -- 基于数字的参数 (见下)
* ``collector`` 设置收集器，可以是 ``ptr``, ``file``, ``sum``, ``func`` 或者任何插件公开的收集器。如果没有指定收集器，那么说明该度量是手工收集的 (你的应用需要更新它)。

当前未实现ptr，而其他收集器则需要一点额外的配置：

``collector=file`` 需要 ``arg1`` 作为文件名，以及一个可选的 ``arg1n`` 作为所谓的分割值。

.. code-block:: sh

   uwsgi --metric name=loadavg,type=gauge,collector=file,arg1=/proc/loadavg,arg1n=1,freq=3
   
这将会添加一个 `loadavg` 度量，类型为gauge，使用 ``/proc/loadavg`` 的内容每3秒更新一次。内容会被分割 (使用\\n, \\t, spaces, \\r和0作为分隔符)，并且第一项 (返回数组是基于0的) 会被当成返回值使用。

分割符是非常强大的，它使得从更复杂的文件（例如 ``/proc/meminfo`` ）中收集信息成为可能。

.. code-block:: sh

   uwsgi --metric name=memory,type=gauge,collector=file,arg1=/proc/meminfo,arg1n=4,freq=3
   
一旦分割， ``/proc/meminfo`` 在第4个位置上保存的是MemFree值。

``collector=sum`` 要求度量列表必须加起来。每个度量都有“孩子”的概念。总和收集器将会对它所有的孩子的值进行求和：

.. code-block:: sh

   uwsgi --metric name=reqs,collector=sum,children=worker.1.requests;worker.2.requests
   
这将会每秒计算worker.1.requests和worker.2.requests值的总和。

``collector=func`` 是一个方便的收集器，避免你为了添加一个新的收集器而去编写整个插件。

让我们定义一个C函数 (称这个文件为mycollector.c或者任何你想要的名字):

.. code-block:: c

   int64_t my_collector(void *metric) {
           return 173;
   }
   
然后把它作为一个共享库进行构建……

.. code-block:: sh

   gcc -shared -o mycollector.so mycollector.c
   
现在，运行uWSGI来加载库……

.. code-block:: sh

   uwsgi --dlopen ./mycollector.so --metric name=mine,collector=func,arg1=my_collector,freq=10
   
这将会每10秒调用C函数my_collector，并且将会设置度量'mine'的值为该函数的返回值。

这个函数必须返回一个 ``int64_t`` 值。它接收的参数是一个 ``uwsgi_metric`` 指针。一般来说，你不需要解析这个度量，因此，只要把它转换成void，就能避免很多糟心的事。

度量目录
*********************

UNIX系统管理员超爱文本文件。它们一般就是系统管理员们大部分时间必须处理的东东。如果你想取悦一个UNIX系统管理员，那么只需给他/她一些文本文件。 (或者一些咖啡，又或者也许是些威士忌，这取决于他们的口味。但一般来说，文本文件应该就可以了。)

度量子系统可以将它所有的度量，以文本文件的形式公开到一个目录中:

.. code-block:: uwsgi

   uwsgi --metrics-dir mymetrics ...
   
这个目录必须预先存在。

这将在'mymetrics'目录中，为每个度量创建一个文本文件。每个文件的内容是度量的值 (实时更新)。

每个文件被映射到进程的地址空间，因此，如果你的虚拟内存稍有增加，也不用担心。

恢复度量（持久化度量）
**************************************

当你重启一个uWSGI实例的时候，会重置它所有的度量。

这一般是最好的，但如果你想要，你可以使用之前存储在定义的度量目录中的值来恢复先前的状态。

只需增加 ``--metrics-dir-restore`` 选项，来强制度量子系统在开始收集值之前，从度量目录读回度量值。

API
***

你的语言插件应该至少公开以下API函数。目前，它们已经在Perl, CPython, PyPy和Ruby中实现了。

* ``metric_get(name)``
* ``metric_set(name, value)``
* ``metric_set_max(name, value)`` -- 只有当给定的 *value* 比当前存储的值大的时候，才设置度量 *name* 
* ``metric_set_min(name, value)`` -- 只有当给定的 *value* 比当前存储的值小的时候，才设置度量 *name* 

    当你需要将一个度量设置为最大值或最小值的时候，``metric_set_max`` 和 ``metric_set_min`` 可以被用来避免必须调用 ``metric_get`` 。另一个简单的用例时使用 ``avg`` 收集器来收集一些被设置成 *max* 和 *min* 的度量之间的平均值。

* ``metric_inc(name[, delta])``
* ``metric_dec(name[, delta])``
* ``metric_mul(name[, delta])``
* ``metric_div(name[, delta])``
* metrics (度量键的元组/数组，应该是不可变并且不可调用的，当前未实现)

统计信息推送器
***************

可以将已收集的度量发送到外部系统，用于分析或者图表生成。

统计信息推送器是旨在发送度量给那些系统的插件。

目前，有两种类型的统计信息推送器：JSON和raw

JSON统计信息推送器发送整个JSON统计信息块 (与你从统计信息服务器获取的相同)，而'raw'则户发送度量列表。

目前可用的统计信息推送器：

rrdtool
^^^^^^^

* 类型: raw
* 插件: rrdtool (默认内置)
* 需要（运行时）: librrd.so
* 语法: ``--stats-push rrdtool:my_rrds ...``

这将会为每个度量存储一个rrd文件到指定的目录中。每个rrd文件都有一个单一的数据来源，名为'metric'。

用法：

.. code-block:: sh

   uwsgi --rrdtool my_rrds ...
   # or
   uwsgi --stats-push rrdtool:my_rrds ...
   
默认情况下，每300秒更新RRD文件。你可以使用 ``--rrdtool-freq`` 来调整这个值

在运行时检测librrd.so库。如果你需要，那么你可以使用 ``--rrdtool-lib`` 来指定它的绝对路径。

statsd
^^^^^^

* 类型: raw
* 插件: stats_pusher_statsd
* 语法: ``--stats-push statsd:address[,prefix]``

推送度量给一个statsd服务器。

用法：

.. code-block:: sh

    uwsgi --stats-push statsd:127.0.0.1:8125,myinstance ...

carbon
^^^^^^

* 类型: raw
* 插件: carbon (built-in by default)
* 见： :doc:`Carbon`

zabbix
^^^^^^

* 类型: raw
* 插件: zabbix
* 语法: ``--stats-push zabbix:address[,prefix]``

推送度量给一个zabbix服务器。

该插件公开了一个 ``--zabbix-template`` 选项，它将会生成一个zabbix模板 (在标准输出或者在指定的文件中)，该模板包含所有公开的度量作为捕获项。

.. note:: 在一些Zabbix版本，你将需要授权获准推送的IP地址。

用法： 

.. code-block:: sh

   uwsgi --stats-push zabbix:127.0.0.1:10051,myinstance ...
   
mongodb
^^^^^^^

* 类型: json
* 插件: stats_pusher_mongodb
* 需要（运行时）: libmongoclient.so
* 语法（键值）: ``--stats-push mongodb:addr=<addr>,collection=<db>,freq=<freq>``

推送统计信息 (JSON格式) 到指定的MongoDB数据库。

file
^^^^

* 类型: json
* 插件: stats_pusher_file

例子插件，将统计信息JSON存储到一个文件中。

socket
^^^^^^

* 类型: raw
* 插件: stats_pusher_socket (builtin by default)
* 语法: ``--stats-push socket:address[,prefix]``

使用以下格式将度量推送到一个UDP服务器： ``<metric> <type> <value>`` (<type> 是前面报告的数字形式)。

例子：

.. code-block:: sh

   uwsgi --stats-push socket:127.0.0.1:8125,myinstance ...

告警/阈值
*****************

你可以为每个度量配置一个或多个“阈值”。

一旦到达了这个限制，那么就会触发指定的告警 (见 :doc:`AlarmSubsystem`)。

一旦传递了该告警，你可以选择重置计数器为一个指定的值 (一般是0)，或者继续以特定的比率触发告警。

.. code-block:: ini

   [uwsgi]
   ...
   metric-alarm = key=worker.0.avg_response_time,value=2000,alarm=overload,rate=30
   metric-alarm = key=loadavg,value=3,alarm=overload,rate=120
   metric-threshold = key=mycounter,value=1000,reset=0
   ...
   
不需要指定告警。使用阈值来自动重置一个度量是非常有效的。
   
注意： ``--metric-threshold`` 和 ``--metric-alarm`` 是同个选项的别名。

SNMP集成
****************

 :doc:`SNMP` 服务器从1.3.6.1.4.1.35156.17.3 OID开始，公开了度量。

例如，要获得 ``worker.0.requests`` 的值：

.. code-block:: sh

   snmpget -v2c -c <snmp_community> <snmp_addr>:<snmp_port> 1.3.6.1.4.1.35156.17.3.0.1
   
记住：只有具有相关的OID的度量可以通过SNMP使用。

内部路由集成
****************************

 ''router_metrics'' 插件 (默认内置) 添加了一系列的动作到内部路由子系统。

* ``metricinc:<metric>[,value]`` increase the <metric>
* ``metricdec:<metric>[,value]`` decrease the <metric>
* ``metricmul:<metric>[,value]`` multiply the <metric>
* ``metricdiv:<metric>[,value]`` divide the <metric>
* ``metricset:<metric>,<value>`` set <metric> to <value>

除了动作之外，还添加了一个名为"metric"的路由变量。

例子：

.. code-block:: ini

   [uwsgi]
   metric = mymetric
   route = ^/foo metricinc:mymetric
   route-run = log:the value of the metric 'mymetric' is ${metric[mymetric]}
   log-format = %(time) - %(metric.mymetric)

请求日志记录
***************

你可以使用 %(metric.xxx) 占位符，在你的请求日志格式中访问度量值：

.. code-block:: ini

   [uwsgi]
   log-format = [hello] %(time) %(metric.worker.0.requests)

官方已注册度量
*****************************

这是一项正在进行的工作。

要知道公开了哪个默认度量的最好的方式是启用统计信息服务器，然后查询它 (或者添加 ``--metrics-dir`` 选项)。

* worker/3 (公开关于worker的信息，例如 worker.1.requests [或者3.1.1] 报告了由worker 1提供服务的请求数)
* plugin/4 (用于由插件自动添加的度量的命名空间，例如plugins.foo.bar)
* core/5 (用于一般实例信息的命名空间)
* router/6 (用于核心路由器的命名空间，例如router.http.active_sessions)
* socket/7 (用于socket的命名空间，例如socket.0.listen_queue)
* mule/8 (用于mule的命名空间，例如mule.1.signals)
* spooler/9 (用于spooler的命名空间，例如spooler.1.signals)
* system/10 (用于系统度量的命名空间，例如loadavg或者空闲内存)
 
为插件分配OID
*************************

如果你想要编写一个会公开度量的插件，那么请添加OID命名空间，你将会用到下面的列表的OID命名空间，并且先进行pull请求。

这将确保所有的插件都使用唯一的OID命名空间。

给所有的插件度量名加上插件名前缀，来确保当相同的键用在多个插件中的时候没有冲突 (example plugin.myplugin.foo.bar, worker.1.plugin.myplugin.foo.bar)

 * (3|4).100.1 - cheaper_busyness

外部工具
**************

看看： https://github.com/unbit/unbit-bars
