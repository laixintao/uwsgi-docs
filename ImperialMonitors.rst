Imperial监控器
=================


``dir://`` -- 扫描目录中的uWSGI配置文件
-----------------------------------------------------

简单将你所有的配置文件放到一个目录中，然后将uWSGI
emperor指向它。Emperor将会开始扫描这个目录。当它发现了一个有效的配置文件的时候，它将生成一个新的uWSGI实例。

在我们的例子中，我们部署一个 Werkzeug_ 测试应用，一个 Trac_ 实例，一个Ruby
on Rails应用和一个 Django_ 应用。

werkzeug.xml

.. code-block:: xml

  <uwsgi>
      <module>werkzeug.testapp:test_app</module>
      <master/>
      <processes>4</processes>
      <socket>127.0.0.1:3031</socket>
  </uwsgi>

trac.ini

.. code-block:: ini

  [uwsgi]
  master = true
  processes = 2
  module = trac.web.main:dispatch_request
  env = TRAC_ENV=/opt/project001
  socket = 127.0.0.1:3032

rails.yml

.. code-block:: yaml

  uwsgi:
      plugins: rack
      rack: config.ru
      master: 1
      processes: 8
      socket: 127.0.0.1:3033
      post-buffering: 4096
      chdir: /opt/railsapp001

django.ini

.. code-block:: ini

  [uwsgi]
  socket = 127.0.0.1:3034
  threads = 40
  master = 1
  env = DJANGO_SETTINGS_MODULE=myapp.settings
  module = django.core.handlers.wsgi:WSGIHandler()
  chdir = /opt/djangoapp001

把这四个文件放到一个目录中，例如我们例子中的 :file:`/etc/uwsgi/vassals` ，然后生成Emperor:

.. code-block:: sh

  uwsgi --emperor /etc/uwsgi/vassals

Emperor将会在那个目录中查找uWSGI实例( ``dir://`` 插件是隐式声明的)，然后启动所需守护进程来运行它们。

.. _Werkzeug: http://werkzeug.pocoo.org/
.. _Trac: http://trac.edgewall.org/
.. _Django: http://djangoproject.com/

``glob://`` -- 监控一个shell模式
--------------------------------------

``glob://`` 与 ``dir://`` 类似，但是必须指定一个glob表达式：

.. code-block:: sh

 uwsgi --emperor "/etc/vassals/domains/*/conf/uwsgi.xml"
 uwsgi --emperor "/etc/vassals/*.ini"

.. note:: 记得用引号将模式括住，否则你的shell将很有可能解析它，然后在调用的时候对其进行展开，这并不是你想要的。

由于Emperor可以搜索子目录层次的配置文件，因此你可以使用一个像这样的结构：

.. code-block:: sh

  /opt/apps/app1/app1.xml
  /opt/apps/app1/...all the app files...
  /opt/apps/app2/app2.ini
  /opt/apps/app2/...all the app files...

然后这样运行uWSGI：

.. code-block:: sh

  uwsgi --emperor /opt/apps/app*/app*.*


``pg://`` -- 扫描一个用于配置的PostgreSQL表
------------------------------------------------------

你可以针对PostgreSQL数据库来运行一个指定查询。它的结果必须是定义一个vassal的由3到6个字段组成的列表：

1. 实例名，包含一个有效的uWSGI配置文件扩展名。 (例如
   ``django-001.ini``)
2. 一个 ``TEXT`` 块，包含该vassal的配置，它的格式基于字段1的扩展。
3. 一个数字，表示这一行的修改时间，使用UNIX格式
   (自纪元起，以秒为单位)。
4. vassal实例的UID。只在 :ref:`Tyrant` 模式下是必须的。
5. vassal实例的GID。只在 :ref:`Tyrant` 模式下是必须的。
6. 用于按需vassal激活的socket。如果指定该字段，那么就会在按需模式下运行vassal。如果省略或者为空，那么将会正常运行vassal。查看
   :doc:`OnDemandVassals` 以获取更多信息。

.. code-block:: sh

  uwsgi --plugin emperor_pg --emperor "pg://host=127.0.0.1 user=foobar dbname=emperor;SELECT name,config,ts FROM vassals"

* 每当添加了一个新的元组，就会创建一个新的实例，并且使用第二个字段中指定的配置来生成它。
* 每当修改时间字段发生了改变，就会重载该实例。
* 如果移除了一个元组，那么对应的vassal也将会被销毁。


``mongodb://`` -- 扫描用于配置的MongoDB集合
------------------------------------------------------------

.. code-block:: sh

  uwsgi --plugin emperor_mongodb --emperor "mongodb://127.0.0.1:27107,emperor.vassals,{enabled:1}"

这将会扫描字段 ``enabled`` 设置为1的 ``emperor.vassals`` 集合中所有的document。一个兼容Emperor的document必须定义三个字段： ``name``, ``config`` 和 ``ts`` 。在 :ref:`Tyrant` 模式下，需要2个额外的字段。对于按需vassal模式，也有可选的 ``socket`` 字段。

* ``name`` (字符串) 是vassal的名字 (记得给它一个有效的扩展名，例如.ini)
* ``config`` (多行字符串) 是vassal配置，该配置的格式由 ``name`` 的扩展名所描述。
* ``ts`` (日期) 是配置的时间戳 (注意：MongoDB内部将时间戳以毫秒为单位进行存储。)
* ``uid`` (数字) vassal实例的UID。仅在 :ref:`Tyrant` 模式下是必须的。
* ``gid`` (数字) vassal实例的GID。仅在 :ref:`Tyrant` 模式下是必须的。
* ``socket`` (字符串) 用于按需vassal激活的Socket。如果指定，那么
  vassal将会运行在按需模式。如果省略或者为空，那么vassal将会正常运行。查看 :doc:`OnDemandVassals` 以获得更多信息。

``amqp://`` -- 使用一个AMQP兼容的消息队列来宣告事件
---------------------------------------------------------------------

把你的AMQP (例如，RabbitMQ) 服务器地址作为--emperor参数的值设置：

.. code-block:: sh

  uwsgi --plugin emperor_amqp --emperor amqp://192.168.0.1:5672

现在，Emperor将会等待 ``uwsgi.emperor`` 交换机中的消息。这应该是一个类型为 `fanout` 的交换机，但是你可以根据你的特殊需求使用其他系统。消息是包含一个有效的uWSGI配置文件的绝对路径的简单字符串。

.. code-block:: python

  # The pika module is used in this example, but you're free to use whatever adapter you like.
  import pika
  # connect to RabbitMQ server
  connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.0.1'))
  # get the channel
  channel = connection.channel()
  # create the exchange (if not already available)
  channel.exchange_declare(exchange='uwsgi.emperor', type='fanout')
  # publish a new config file
  channel.basic_publish(exchange='uwsgi.emperor', routing_key='', body='/etc/vassals/mydjangoapp.xml')

你第一次启动脚本的时候，emperor将会添加新的实例 (如果该配置文件可用)。从那时开始，每当你重发布消息，应用将会被重载。当你移除配置文件的时候，应用也会被移除。

.. tip::

  你可以订购不同服务器上的所有emperor到这个交换机，从而实现集群同步的重载/部署。

使用HTTP的AMQP
^^^^^^^^^^^^^^

uWSGI :ref:`is capable of loading configuration files over
HTTP<LoadingConfig>`. 这是一种非常方便的为大量主机动态生成配置文件的方式。简单在AMQP消息中声明配置文件的HTTP URL。记住，它必须以有效配置文件的扩展名结尾，但在钩子之下，它可以由脚本生成。如果该HTTP URL返回一个非200状态码，那么将会移除该实例。

.. code-block:: python

  channel.basic_publish(exchange='uwsgi.emperor', routing_key='', body='http://example.com/confs/trac.ini')

直接的AMQP配置
^^^^^^^^^^^^^^^^^^^^^^^^^

也可能直接通过AMQP提供配置文件。 ``routing_key``
将是（虚拟）配置文件名，而消息将是配置文件的内容。

.. code-block:: python

  channel.basic_publish(
    exchange='uwsgi.emperor', 
    routing_key='mydomain_trac_config.ini', 
    body="""
  [uwsgi]
  socket=:3031
  env = TRAC_ENV=/accounts/unbit/trac/uwsgi
  module = trac.web.main:dispatch_request
  processes = 4""")

前一个模式的相同的重载规则是有效的。当你想要移除一个实例的时候，则简单将一个空的body作为“配置”进行设置。

.. code-block:: python

  channel.basic_publish(exchange='uwsgi.emperor', routing_key='mydomain_trac_config.ini', body='')

``zmq://`` -- ZeroMQ
--------------------

Emperor把自己绑定到一个ZeroMQ PULL socket，准备好接收命令。

.. code-block:: sh

   uwsgi --plugin emperor_zeromq --emperor zmq://tcp://127.0.0.1:5252

每个命令都是一条通过PUSH zmq socket发送的多部分消息。一个命令至少由两部分组成： ``command`` 和 ``name`` 。 ``command`` 是要执行的动作，而 ``name`` 是vassal的名字。可以指定4个可选部分。

* ``config`` (一个包含vassal配置的字符串)
* ``uid`` (tyrant模式下移除特权的用户id)
* ``gid`` (tyrant模式下移除特权的组id)
* ``socket`` (用于按需vassal激活的socket。如果指定，如果指定，那么
  vassal将会运行在按需模式。如果省略或者为空，那么vassal将会正常运行。查看 :doc:`OnDemandVassals` 以获得更多信息。)

有两类命令 (目前):

* ``touch``
* ``destroy``

第一个用于创建和重载实例，而第二个用于销毁。如果你不指定一个配置字符串，那么Emperor将会假设你指的是Emperor当前目录下的一个可用的静态文件。

.. code-block:: python

   import zmq
   c = zmq.Context()
   s = zmq.Socket(c, zmq.PUSH)
   s.connect('tcp://127.0.0.1:5252')
   s.send_multipart(['touch','foo.ini',"[uwsgi]\nsocket=:4142"])


``zoo://`` -- Zookeeper
-----------------------

目前正在开发中。

``ldap://`` -- LDAP
-------------------

目前正在开发中。
