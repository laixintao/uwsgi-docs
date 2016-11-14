推送统计信息 (从1.4起)
=============================

重要：度量子系统提供了对以下概念更好的介绍。见 :doc:`Metrics`

从uWSGI 1.4起，你可以通过各种系统(称之为统计信息推送器)推送统计数据 (与你通过 :doc:`StatsServer` 获得的相同的JSON块)。

会定期 (默认是3秒) 对统计数据进行推送。

'file'统计信息推送器
***********************

默认情况下，直到1.9.18，就可以使用'file' 统计信息推送器了。从1.9.19起，它可以作为一个插件 (stats_pusher_file) 使用。

它允许你保存json数据块到一个文件中 (以附加模式打开)

.. code-block:: ini

   [uwsgi]
   socket = :3031
   module = foobar
   master = true
   stats-push = file:path=/tmp/foobar,freq=10

这个配置将会每10秒附加JSON到/tmp/foobar文件


'mongodb'统计信息推送器
**************************

这是第一个开发的统计信息推送器插件，它允许你直接在一个mongodb集合上存储JSON数据

.. code-block:: ini

   [uwsgi]
   plugins = stats_pusher_mongodb
   socket = :3031
   module = foobar
   master = true
   stats-push = mongodb:addr=127.0.0.1:5151,collection=uwsgi.mystats,freq=4

这个配置将会每4秒把JSON数据插入到mongodb服务器127.0.0.1:5151上的集合uwsgi.mystats中。

要构建这个插件，你需要mongodb开发头文件 (Debian/Ubuntu上是mongodb-dev)

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/stats_pusher_mongodb

将会达到目的


小抄
*****

你可以配置所有你需要的统计数据推送器，只需指定多个stats-push选项

.. code-block:: ini

   [uwsgi]
   plugins = stats_pusher_mongodb
   socket = :3031
   module = foobar
   master = true
   stats-push = mongodb:addr=127.0.0.1:5151,collection=uwsgi.mystats,freq=4
   stats-push = mongodb:addr=127.0.0.1:5152,collection=uwsgi.mystats,freq=4
   stats-push = mongodb:addr=127.0.0.1:5153,collection=uwsgi.mystats,freq=4
   stats-push = mongodb:addr=127.0.0.1:5154,collection=uwsgi.mystats,freq=4

