通过订阅的按需vassals
===================================

生成一个带有一个命令socket (它是一个通道，允许外部进程管理vassals) 的Emperor，并且指示它不要生成vassals，直到收到了一个“spawn”命令。

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-command-socket = /run/emperor.socket
   emperor-wait-for-command = true
   
生成一个带有监听端口:4041的订阅服务器，并且监听端口 :3031 的Fastrouter，并且指示它每当方向一个不活动的实例时联系特定的emperor socket：

.. code-block:: ini

   [uwsgi]
   master = true
   fastrouter = :3031
   fastrouter-subscription-server = 127.0.0.1:4041
   ; the emperor socket to contact
   fastrouter-emperor-socket = /tmp/emperor.socket
   fastrouter-stats-server = 127.0.0.1:4040
   

把你的vassals放在/etc/uwsgi/vassals中，确保它们每个都正确地订阅这个fastrouter (要求这个订阅将'不活动的订阅'标记为'活动'

现在，你可以开始使用原始数据报，将“不活动的”订阅放在fastrouter中。使用--subscribe2方式，一个不活动订阅包例子可以是这样的：

.. code-block:: sh

   addr=127.0.0.1:3036,vassal=one.ini,inactive=1,server=127.0.0.1:4041,key=127.0.0.1:9090

这将会指示fastrouter映射主机127.0.0.1:9090到将运行在127.0.0.1:3036上的不活动uWSGI服务器，并且在第一个请求上运行one.ini vassal。

