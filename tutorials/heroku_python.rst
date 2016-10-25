使用uWSGI在Heroku上运行python web应用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

前提条件：一个Heroku账户 (在cedar平台上)，git (本地系统跟上) 以及heroku toolbelt。

注意：需要uWSGI version >= 1.4.6来确保正确运行python应用。较老的版本可能可用，但并不支持。

准备环境
*************************

在你的本地系统上，为项目准备一个目录：

.. code-block:: sh

   mkdir uwsgi-heroku
   cd uwsgi-heroku
   git init .
   heroku create

最后一个命令将会创建一个新的heroku应用 (你可以在网页界面上检查它)。

在我们的例子中，我们会运行Werkzeug WSGI testapp，因此，除了uWSGI之外，我们还需要安装werkzeug包。

第一步是创建一个requirements.txt文件，然后用git来跟踪它。

该文件的内容很简单

.. code-block:: sh

   uwsgi
   werkzeug

现在，用git来跟踪

.. code-block:: sh

   git add requirements.txt

创建uWSGI配置文件
******************************

现在，可以创建我们的uWSGI配置文件了。基本上，在heroku上可以使用所有特性

.. code-block:: ini

   [uwsgi]
   http-socket = :$(PORT)
   master = true
   processes = 4
   die-on-term = true
   module = werkzeug.testapp:test_app
   memory-report = true

正如你所看到的，这是一个非常标准的配置。heroku必备的选项只用 --http-socket 和 --die-on-term。

需要第一个来将uWSGI socket绑定到由Heroku系统请求的端口 (通过环境变量PORT导出，我们可以通过$(PORT)访问)

需要第二个(--die-on-term)来修改uWSGI接收到一个SIGTERM时的默认行为 (粗暴的重载，而Heroku期望是关机)

memory-report选项 (因为我们处于内存受限的环境中)是个不错的东东。

记得跟踪这个文件

.. code-block:: sh

   git add uwsgi.ini

准备第一次commit/push
***********************************

现在，需要最后一步：创建Procfile。

Procfile是一个描述启动哪个命令的文件。一般来说(对于其他部署系统)，对于每个由你的应用请求的额外进程，你都将使用它 (例如memcached, redis, celery...)，但在uWSGI之下，你可以继续使用它的高级功能来管理它们。

因此，Procfile是需要启动你的uWSGI实例：

.. code-block:: sh

   web: uwsgi uwsgi.ini

跟踪它：

.. code-block:: sh

   git add Procfile

最后，提交所有东西：

.. code-block:: sh

   git commit -a -m "first commit"

以及，push它 (即：部署)到Heroku:

.. code-block:: sh

    git push heroku master

第一次的适合，将需要一点时间，因为需要准备你的virtualenv以及编译uWSGI。

之后的push将会快得多。

检查你的应用
*****************

运行 ``heroku logs`` ，你将能够访问uWSGI日志。你应该获取所有你熟悉的信息，以及发生问题的情况下的一些最终提示。

使用python的另一个版本
*******************************

Heroku支持不同的python版本。默认情况下 (目前是2013年二月份)，允许Python 2.7.3。

如果你需要另一个版本，那么在你的仓库中创建一个runtime.txt，然后在里面写上如下字符串：

.. code-block:: sh

   python-2.7.2

来使用python 2.7.2

记得在仓库中add/commit它。

每当你的修改python版本，就会构建一个新的uWSGI二进制。

多进程还是多线程？
*****************************

这显然取决于你的应用。但由于我们在一个内存受限的环境中，因此使用线程可以期望获得更好的内存使用。

除此之外，如果你计划将生产应用放在Heroku上，那么确保了解Dynos和它们的代理是如何工作的(这很重要。真的)

异步/绿色线程/协程？
*****************************

像往常一样，不要相信那些让你总是使用某种类型的异步模式(例如gevent)的人。如果你的应用是异步友好型的，那么显然，你可以使用gevent (在近期的uWSGI发布版本中，会默认构建它)，但如果你不知道，那么保持使用多进程（或多线程）。

Harakiri
********

如之前所述，如果你计划将生产应用放在heroku上，那么确保了解dynos和它们的代理是如何工作的。基于此，试着总是为你的应用将harakiri参数设置成一个不错的值。 (不要要求默认值，它取决于应用)

静态文件
************

一般来讲，在Heroku上提供静态文件并不是一个好主意 (主要从设计的角度来看)。你当然可以有此需求。在这种情况下，记得使用uWSGI功能，特别是卸载（offloading）是在提供大文件时留出worker的最好方法 (另外，记得必须使用git来跟踪你的静态文件)

自适应进程生成
*************************

对于Heroku方法，没有好的支持的算法，并且很可能，在这样一个平台上使用一个动态进程数并没有什么意义。

日志记录
*******

如果你计划在生产环境上使用heroku，那么记住在一个外部服务器上（有持续存储）发送你的日志(例如，通过udp)。

检查uWSGI可用的记录器。当然，会有一个满足你的需要的。(重视安全性，因为日志会记录明文)。

更新：一个具有crypto特性的udp记录器正在开发中。

告警
******

所有的告警插件应该工作正常

Spooler
***********

由于你的应用运行在一个非持久化的文件系统上，因此使用Spooler是个糟糕的主意 (你会很容易丢失任务)。

Mule
*****

它们可以正常使用

信号 (定时器、文件监控器、cron……)
****************************************

它们都能用，但不要依赖于cron功能，因为heroku每时每刻都能杀掉/摧毁/重启你的实例。

外部守护进程
****************

 --attach-daemon 选项及其 --smart 变量可以正常使用。只是记住，你处于一个不稳定的文件系统中，并且你无法任意如你所愿的绑定端口/地址

监控你的应用（高级/hack）
*************************************

尽管Heroku和newrelic服务工作良好，但是你总是需要监控你的uWSGI实例内部。

一般来说，作为客户端，你使用诸如uwsgitop这样的工具启动stats子系统。

你可以简单的添加uwsgitop到你的requirements.txt中

.. code-block:: sh

   uwsgi
   uwsgitop
   werkzeug

并在一个TCP端口上启动stats服务器 (unix socket将不能用，因为运行uwsgitop的实例并不在同一个服务器上！！！):

.. code-block:: ini

   [uwsgi]
   http-socket = :$(PORT)
   master = true
   processes = 4
   die-on-term = true
   module = werkzeug.testapp:test_app
   memory-report = true
   stats = :22222

现在，我们有个问题：如果访问我们的实例？

我们需要知道物理运行我们的实例的机器的LAN地址。要完成它，一个原始的技巧是在uWSGI启动的适合运行ifconfig：

.. code-block:: ini

   [uwsgi]
   http-socket = :$(PORT)
   master = true
   processes = 4
   die-on-term = true
   module = werkzeug.testapp:test_app
   memory-report = true
   stats = :22222
   exec-pre-app = /sbin/ifconfig eth0

现在，有了 ``heroku logs`` 命令，你就可以知道你的stats服务器在哪里了

.. code-block:: sh

   heroku run uwsgitop 10.x.x.x:22222

将x.x.x修改成发现的地址，然后记住，你不能绑定到端口22222上，因此，相应修改它。

为了监控，值得搞得一团糟吗？如果你在上线之前测试你的应用，那么这是一个好主意，但如果你计划购买更多的dynos，那么一切都变得太复杂，此时最好使用一些heroku相关技术 (如果有的话)
