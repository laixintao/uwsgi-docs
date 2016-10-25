Python/WSGI应用快速入门
=======================================

这个快速入门将会告诉你如何部署简单的WSGI应用和常见的web框架。

这里，Python指的是CPython，对于PyPy，你需要使用特定的插件： :doc:`PyPy`，Jython支持正在开发中。

.. note::

    要遵循此快速入门，你至少需要uWSGI 1.4。更老的版本不再维护，并且有灰常多bug！

安装uWSGI以及Python支持
************************************

.. 小技巧::

    当你开始学习uWSGI时，试着从官方来源进行构建：使用发布版本提供的包可能会让你很头疼。当你清楚了解uWSGI时，你可以使用模块化构建 (就像你的发布版本中可用的那个一样)。

uWSGI是一个（大的）C应用，因此，你需要一个C编译器 (例如gcc或者clang)，以及Python开发头文件。

在基于Debian的发行版上，

.. code-block:: sh

   apt-get install build-essential python-dev

就够了。

你有多种方式为Python安装uWSGI：

* via pip

  .. code-block:: sh

      pip install uwsgi

* 使用网络安装器

  .. code-block:: sh

      curl http://uwsgi.it/install | bash -s default /tmp/uwsgi

  (这将会把uWSGI二进制安装到 ``/tmp/uwsgi`` ，你可以随意修改它)。

* 通过下载源tarball文件，然后执行"make"命令

  .. code-block:: sh

      wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
      tar zxvf uwsgi-latest.tar.gz
      cd <dir>
      make

  (构建之后，当前目录下，你会获得一个 ``uwsgi`` 二进制文件)。

未涵盖通过包发布来安装(不可能取悦所有人)，但是所有的一般规则都适用。

当使用发行版提供的包来测试这个快速入门时，你也许想要考虑一件事，就是非常有可能你的发行版本以模块化的方式构建了uWSGI (每个特性都是一个必须加载的不同插件)。要完成这个快速入门，你必须在第一个系列的例子前面加上 ``--plugin python,http`` ，以及当移除了HTTP路由器时加上 ``--plugin python``  (如果这对你没意义，那就请继续往下读)。

第一个WSGI应用
**************************

让我们从一个简单的"Hello World"开始：

.. code-block:: python

   def application(env, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       return [b"Hello World"]

(将其保存为 ``foobar.py``)。

正如你所见，它由一个简单的Python函数组成。之所以称之为 "application"，是因为它是uWSGI Python加载器将会搜索的默认函数 (但你显然可以自定义)。

将其部署在HTTP端口9090
***************************

现在，启动uWSGI来运行一个HTTP服务器/路由器，它会传递请求到你的WSGI应用：

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py

就这样。

.. note:: 当你有一个前端web服务器，或者你正进行某些形式的基准时，不要使用 ``--http`` ，使用 ``--http-socket`` 。继续阅读快速入门来了解原因。


添加并发和监控
*********************************

你想进行的第一个调整可能是增加并发性 (默认情况下，uWSGI启动一个单一的进程和一个单一的线程)。

你可以用 ``--processes`` 选项添加更多的进程，或者使用 ``--threads`` 选项添加更多的线程 (或者可以同时添加)。

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py --master --processes 4 --threads 2

这将会生成4个进程 (每个进程有2个线程)，一个master进程 (在Inc死掉的时候会生成它们) 和HTTP路由器 (见前面)。

一个重要的任务是监控。在生产部署上，了解正在发生的事情是至关重要的。stats子系统允许你将uWSGI的内部统计数据作为JSON导出：

.. code-block:: sh

   uwsgi --http :9090 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

对你的应用进行几次请求，然后telnet到端口9191，你会获得大量有趣的信息。你可能想要使用"uwsgitop" (仅需 ``pip install`` 来安装它)，这是一个类似于top的工具，用来监控实例。

.. note::

    绑定stats socket到一个私有地址 (除非你知道你在做什么)，否则，每个人都能够访问它！

将它放在一个完整的web服务器之后
*******************************

即使uWSGI HTTP路由器是稳定并且高性能的，但是你或许想要将你的应用放在一个全功能的web服务器之后。

uWSGI原生支持HTTP, FastCGI, SCGI及其特定的名为"uwsgi"的协议 (是哒，错误的命名选择)。最好的协议显然是uwsgi，nginx和Cherokee已经支持它了 (虽然有各种Apache模块可用)

一个常用的nginx配置如下：

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
   }

这表示“传递每一个请求给绑定到3031端口并使用uwsgi协议的服务器”。

现在，我们可以生成uWSGI来本地使用uwsgi协议：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

如果你要运行 ``ps aux`` ，那么你会看到一个进程。已经移除了HTTP路由器，因为我们的“worker” (被分配给uWSGI的进程) 本地使用uwsgi协议。

如果你的代理/web服务器/路由器使用HTTP，那么你必须告诉uWSGI本地使用http协议 (这与会自己生成一个代理的--http不同):

.. code-block:: sh

   uwsgi --http-socket 127.0.0.1:3031 --wsgi-file foobar.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

开机自动启动uWSGI
************************************

如果你在想打开vi，然后写一个init.d脚本来生成uWSGI，那么请坐下(并且冷静下来)，先确保你的系统没有提供一个更好（更现代）的方法。

每个发行版本都选择了一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...) ，并且有大量可用的进程管理器 (supervisord, god, monit, circus...)。

uWSGI会跟它们都很好地集成 (我们希望是这样的)，但如果你计划部署大量的应用，那么看看uWSGI :doc:`Emperor<Emperor>` - 它或多或少是每个devops工程师的梦想。

部署Django
****************

Django大概是最常使用的Python web框架了。部署它是相当容易的 (我们继续配置4个进程，每个进程有2个线程)。

假设Django工程位于 ``/home/foobar/myproject``:

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --chdir /home/foobar/myproject/ --wsgi-file myproject/wsgi.py --master --processes 4 --threads 2 --stats 127.0.0.1:9191

(使用 ``--chdir`` ，我们移到指定的目录下)。在Django中，需要使用它来正确加载模块。

哎呀！这是什么鬼？！是哒，你是对的，对的……处理这样长的命令行并不实际，并且愚蠢而易于犯错。不要害怕！uWSGI支持多种配置风格。在这个入门中，我们会使用.ini文件。

.. code-block:: ini

    [uwsgi]
    socket = 127.0.0.1:3031
    chdir = /home/foobar/myproject/
    wsgi-file = myproject/wsgi.py
    processes = 4
    threads = 2
    stats = 127.0.0.1:9191

好得多了！

仅需运行：

.. code-block:: sh

   uwsgi yourfile.ini

如果文件 ``/home/foobar/myproject/myproject/wsgi.py`` (或者任何你的工程的名字) 并不存在，那么你很有可能使用的是Django的一个老(< 1.4)版本。在这种情况下，你需要多一点配置：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --chdir /home/foobar/myproject/ --pythonpath .. --env DJANGO_SETTINGS_MODULE=myproject.settings --module "django.core.handlers.wsgi:WSGIHandler()" --processes 4 --threads 2 --stats 127.0.0.1:9191

或者，使用.ini文件：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   chdir = /home/foobar/myproject/
   pythonpath = ..
   env = DJANGO_SETTINGS_MODULE=myproject.settings
   module = django.core.handlers.wsgi:WSGIHandler()
   processes = 4
   threads = 2
   stats = 127.0.0.1:9191

更老的(< 1.4)Django发布版本需要设置 ``env``, ``module`` 和 ``pythonpath`` (``..`` 允许我们访问 ``myproject.settings`` 模块)。


部署Flask
***************

Flask是一个流行的Python web微框架。

将下面例子保存为 ``myflaskapp.py``:

.. code-block:: python

   from flask import Flask

   app = Flask(__name__)

   @app.route('/')
   def index():
       return "<span style='color:red'>I am app 1</span>"

Flask将其WSGI函数 (就是那个在这个快速入门开始的地方我们称为"application"的那个)导出为"app"，因此，我们需要指示uWSGI使用它。我们仍然使用4个进程/2个线程，以及uwsgi socket：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --wsgi-file myflaskapp.py --callable app --processes 4 --threads 2 --stats 127.0.0.1:9191

(唯一添加的是 ``--callable`` 选项)。

部署web2py
****************

又一个流行选择。解压缩web2py源发布版本到所选的目录下，然后编写一个uWSGI配置文件：

.. code-block:: ini

   [uwsgi]
   http = :9090
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

.. note::

    在近期的web2py发布版本中，你可能需要将 ``wsgihandler.py`` 脚本拷贝出 ``handlers`` 目录。

再次使用HTTP路由器。仅需在浏览器中访问端口9090，你就能看到web2py欢迎页面。

点击管理员界面，然后……哎哟，不能用，因为需要HTTPS。不要担心，uWSGI路由器是可以使用HTTPS的 (确保你有OpenSSL开发头文件：安装它们，然后重新构建uWSGI，构建系统将会自动检测到它)。

首先，生成密钥和证书：

.. code-block:: sh

   openssl genrsa -out foobar.key 2048
   openssl req -new -key foobar.key -out foobar.csr
   openssl x509 -req -days 365 -in foobar.csr -signkey foobar.key -out foobar.crt

现在，你有了2个文件 (好吧，算上 ``foobar.csr`` ，是3个)， ``foobar.key`` 和 ``foobar.crt`` 。修改uWSGI配置：

.. code-block:: ini

   [uwsgi]
   https = :9090,foobar.crt,foobar.key
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

重新运行uWSGI，并且在浏览器中使用 ``https://`` 访问9090端口。

关于Python线程的注意事项
************************

如果你在不使用线程的情况下启动uWSGI，那么Python GIL将不会启动，因此，你的应用生成的线程将不会运行。你或许不会喜欢这个选择，但是记住，uWSGI是一个语言无关的服务器，因此它的大多数选择都是为了维护它的“不可知论”。

但是，不用担心，uWSGI开发者做的选择基本上没有选项不能改变的。（也就是说，基本上你可以通过选项改变它）

如果你想要维护Python线程支持，而不为你的应用启动多线程，那么仅需添加 ``--enable-threads`` 选项 (或者在ini风格的文件中添加 ``enable-threads = true`` )。

virtualenv
***********

可以配置uWSGI在指定的virtualenv中搜索Python模块。

仅需添加 ``virtualenv = <path>`` 到你的选项中。

安全性和可用性
*************************

**总是** 避免以root用户运行你的uWSGI实例。你可以使用 ``uid`` 和 ``gid`` 选项来去除权限：

.. code-block:: ini

   [uwsgi]
   https = :9090,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

如果你需要绑定到特许端口 (例如用于HTTPS的443)，那么使用共享socket。它们在去除权限之前创建，并且可以通过 ``=N`` 语法引用，其中， ``N`` 是socket号 (从0开始)：

.. code-block:: ini

   [uwsgi]
   shared-socket = :443
   https = =0,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8

web应用部署的一个常见问题是“卡住的请求”。你所有的线程/worker都卡住了 (请求阻塞) ，而你的应用无法接收更多的请求。要避免这个问题，你可以设置一个 ``harakiri`` 定时器。它是一个监控器 (由master进程管理)，会摧毁那些卡住超过指定秒数的进程 (小心选择 ``harakiri`` 值)。例如，你也许想要摧毁那些阻塞超过30秒的worker：

.. code-block:: ini

   [uwsgi]
   shared-socket = :443
   https = =0,foobar.crt,foobar.key
   uid = foo
   gid = bar
   chdir = path_to_web2py
   module = wsgihandler
   master = true
   processes = 8
   harakiri = 30

除此之外，自uWSGI 1.9起，stats服务器导出了全部请求变量，因此，你可以（实时）看到你的实例正在做什么 (对于每个worker, thread 或者异步核)。


卸载
**********

:doc:`OffloadSubsystem` 允许你在特定模式匹配的适合尽快地释放你的worker，并且可以被委派给一个纯C线程。举几个栗子，从文件系统发送静态文件，从网络传输数据到客户端，等等。

卸载是非常复杂的，但它的使用对最终用户是透明的。如果你想试一试，那么仅需添加 ``--offload-threads <n>`` ，其中，<n>是要生成的线程数 (每个CPU1个是开始的一个不错的值)。

当启用了卸载线程，将会自动检测所有可以被优化的部分。

彩蛋：对不同Python版本使用相同的uWSGI二进制文件
*********************************************************

正如我们所见的，uWSGI是由一个小核心和各种插件组成的。插件可以嵌入到二进制文件中，或者动态加载。当你为Python构建uWSGI的时候，一系列的插件，加上Python本身将会嵌入到最后的二进制文件中。

如果你想要支持多个Python版本，而不想为每个版本构建一个二进制文件的适合，这会是个问题。

最好的方法可能是使用一个小的二进制文件，其中内建语言无关的特性，然后为每个Python版本准备一个插件，按需加载。

在uWSGI源代码目录中：

.. code-block:: sh

   make PROFILE=nolang
   
这将会构建一个uwsgi二进制文件，它内建了除了Python之外所有默认的插件。

现在，在相同的目录下，我们开始构建Python插件：

.. code-block:: sh

   PYTHON=python3.4 ./uwsgi --build-plugin "plugins/python python34"
   PYTHON=python2.7 ./uwsgi --build-plugin "plugins/python python27"
   PYTHON=python2.6 ./uwsgi --build-plugin "plugins/python python26"

最后，你将获得三个文件： ``python34_plugin.so``, ``python27_plugin.so``, ``python26_plugin.so`` 。拷贝这些到你想要的目录下。 (默认情况下，uWSGI会在当前工作目录下搜索插件。)

现在，在你的配置文件中，你可以简单地添加 (在最顶部)  `plugins-dir` 和 `plugin` 指令。

.. code-block:: ini

   [uwsgi]
   plugins-dir = <path_to_your_plugin_directory>
   plugin = python26
   
这将会从目录下加载 ``python26_plugin.so`` 插件库到你拷贝插件到的那里。

现在……
**********

有了这些概念，你应该已经能够投入生产了，但是uWSGI是一个庞大的项目，它有数百种特性和配置。如果你想成为一个更好的系统管理员，那么继续阅读整个文档吧。
