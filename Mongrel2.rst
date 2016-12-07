附加uWSGI到Mongrel2
===========================

Mongrel2_ 是下下一代的web服务器，关注于现代web应用。

就像uWSGI，它完全语言无关，集群友好，并且富有争议 :)

它使用令人惊奇的 ZeroMQ_ 库来通信，允许可靠容易的消息队列，以及免配置的可扩展性。

从版本0.9.8-dev起，uWSGI可以被用作Mongrel2处理器。

.. _Mongrel2: http://mongrel2.org/
.. _ZeroMQ: http://www.zeromq.org/

要求
------------

要在uWSGI中启用对ZeroMQ/Mongrel2的支持，你需要zeromq库 library (2.1+)和uuid库。

Mongrel2可以使用JSON或者tnetstring来将数据 (例如头部和各种其他信息) 传递给处理器。uWSGI默认支持tnetstring，但需要 `Jansson <http://www.digip.org/jansson/>`_ 库来解析JSON数据。如果你没安装jansson，或者不想使用JSON，那么确保你在Mongrel2中的Handler部分指定了 ``protocol='tnetstring'`` ，因为默认是使用JSON的。这会导致uWSGI日志中一条相当模糊的“JSON支持未启用。跳过请求”。

配置Mongrel2
--------------------

find你可以发现uWSGI源代码配备了 ``mongrel2-uwsgi.conf`` 。你可以以这个文件为基础来配置Mongrel2。


.. code-block:: python

  main = Server(
      uuid="f400bf85-4538-4f7a-8908-67e313d515c2",
      access_log="/logs/access.log",
      error_log="/logs/error.log",
      chroot="./",
      default_host="192.168.173.11",
      name="test",
      pid_file="/run/mongrel2.pid",
      port=6767,
      hosts = [
          Host(name="192.168.173.11", routes={
              '/': Handler(send_spec='tcp://192.168.173.11:9999',
                      send_ident='54c6755b-9628-40a4-9a2d-cc82a816345e', 
                      recv_spec='tcp://192.168.173.11:9998', recv_ident='',
                      protocol='tnetstring')
          })
      ]
  )

  settings = {'upload.temp_store':'tmp/mongrel2.upload.XXXXXX'}
  servers = [main]

这是一个相当标准的Mongrel2配置，启动了上传流。

为Mongrel2配置uWSGI
------------------------------

要将uWSGI附加到Mongrel2，只需简单实用 :ref:`OptionZeromq` 选项：

.. code-block:: sh

  uwsgi --zeromq tcp://192.168.173.11:9999,tcp://192.168.173.11:9998

你可以生成多个进程 (每个都会订阅到Mongrel2，并使用一个不同的uuid)

.. code-block:: sh
 
  uwsgi --zeromq tcp://192.168.173.11:9999,tcp://192.168.173.11:9998 -p 4

你也可以使用线程。每个线程将订阅Mongrel2队列，但是响应者socket将会由所有线程共享，并由互斥锁包含。

.. code-block:: sh

  uwsgi --zeromq tcp://192.168.173.11:9999,tcp://192.168.173.11:9998 -p 4 --threads 8
  # This will spawn 4 processes with 8 threads each, totaling 32 threads.

一起测试
-------------

添加一个应用到uWSGI (一如既往，我们将使用werkzeug.testapp)

.. code-block:: sh

  uwsgi --zeromq tcp://192.168.173.11:9999,tcp://192.168.173.11:9998 -p 4 --threads 8 --module werkzeug.testapp:test_app

现在，在所有你想要的服务器上启动命令，Mongrel2将会自动分发请求给它们。

异步模式
----------

.. warning::

  对ZeroMQ的异步支持仍然在开发中，因为ZeroMQ使用边沿触发事件，这些事件会复杂化uWSGI异步架构中的东东。

Chroot
------

默认情况下，Mongrel2会 ``chroot()`` 。这对于安全是件好事，但是对于文上传流会让人头疼。记住，Mongrel2将会在它自己的chroot jail中上传文件，因此，如果你的uWSGI实例并不处于同一个chroot jail中，那么你将必须小心选择路径。在这个例子中，我们的Mongrel2配置文件使用了一个相对路径来轻松让uWSGI访问文件。

性能
-----------

Mongrel2即使是在巨大负载的情况下也是非常快且可靠的。tnetstring和JSON是基于文本的 (因此，它们比二进制 :doc:`uwsgi protocol <Protocol>` 稍微低效率点。然而，因为Mongrel2不需要昂贵的一请求一连接方法，因此与(例如) :doc:`Nginx<Nginx>` + uWSGI 方法相比，你应该会得到几乎相同的（如果不是更高的）的结果。

uWSGI集群 + ZeroMQ
-------------------------

你可以轻松地将uWSGI :doc:`clustering<Clustering>` 和ZeroMQ混在一起。

选择主节点，然后运行

.. code-block:: sh

  uwsgi --zeromq tcp://192.168.173.11:9999,tcp://192.168.173.11:9998 -p 4 --threads 8 --module werkzeug.testapp:test_app --cluster 225.1.1.1:1717

然后在所有其他节点上，简单运行


.. code-block:: sh
  
  uwsgi --cluster 225.1.1.1:1717

用ZeroMQ混合标准socket
-----------------------------------

除了ZeroMQ，你还可以添加uwsgi/:doc:`HTTP<HTTP>`/FastCGI/... socket到你的uWSGI服务器中，但如果你这样做，记得禁用线程！这个限制未来可能会被修复。

通过ZeroMQ进行日志记录
------------------

.. seealso:: :doc:`ZeroMQLogging`