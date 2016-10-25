使用uWSGI在Heroku上运行Ruby/Rack
==============================================

前提条件：一个Heroku账户 (在cedar平台上)，git (本地系统跟上) 以及heroku toolbelt (或者老的/已弃用的heroku gem)

注意：需要uWSGI version >= 1.4.8来确保正确运行ruby/rack应用。较老的版本可能可用，但并不支持。

准备环境(一个Sinatra应用)
*************************************************

在你的本地系统上，为你的sinatra应用准备结构

.. code-block:: sh

   mkdir uwsgi-heroku
   cd uwsgi-heroku
   git init .
   heroku create --stack cedar
   

最后一个命令将会创建一个新的heroku应用 (你可以在网页界面上检查它)。

下一步是创建我们的Gemfile (这个文件包含应用所需的gem)

.. code-block:: rb

   source 'https://rubygems.org'

   gem "uwsgi"
   gem "sinatra"

现在，需要运行 ``bundle install`` 来创建Gemfile.lock文件

用git来跟踪这两个：

.. code-block:: sh

   git add Gemfile
   git add Gemfile.lock

最后，创建一个包含Sinatra样例应用的config.ru文件

.. code-block:: rb

   require 'sinatra'

   get '/hi' do
     return "ciao"
   end

   run Sinatra::Application

并且，跟踪它

.. code-block:: sh

   git add config.ru

创建uWSGI配置文件
******************************

现在，准备好创建uWSGI配置了 (我们将使用.ini文件格式，称为uwsgi.ini)。

heroku的最小化步骤如下 (解释请查看文件中的注释)

.. code-block:: ini

   [uwsgi]
   ; bind to the heroku required port
   http-socket = :$(PORT)
   ; force the usage of the ruby/rack plugin for every request (7 is the official numbero for ruby/rack)
   http-socket-modifier1 = 7
   ; load the bundler subsystem
   rbrequire = bundler/setup
   ; load the application
   rack = config.ru
   ; when the app receives the TERM signal let's destroy it (instead of brutal reloading)
   die-on-term = true

但是，更好的安装将是

.. code-block:: ini

   [uwsgi]
   ; bind to the heroku required port
   http-socket = :$(PORT)
   ; force the usage of the ruby/rack plugin for every request (7 is the official numbero for ruby/rack)
   http-socket-modifier1 = 7
   ; load the bundler subsystem
   rbrequire = bundler/setup
   ; load the application
   rack = config.ru
   ; when the app receives the TERM signal let's destroy it (instead of brutal reloading)
   die-on-term = true
   ; enable the master process
   master = true
   ; spawn 4 processes to increase concurrency
   processes = 4
   ; report memory usage after each request
   memory-report = true
   ; reload if the rss memory is higher than 100M
   reload-on-rss = 100

跟踪它：

.. code-block:: sh

   git add uwsgi.ini

部署到heroku
*******************

需要创建最后一个文件 (Heroku要求的)。它就是Procfile，用来指示Heroku系统为web应用启动哪个进程。

我们想使用uwsgi.ini配置文件来生成uwsgi (通过bundler作为gem安装)

.. code-block:: sh

   web: bundle exec uwsgi uwsgi.ini

跟踪它：

.. code-block:: sh

   git add Procfile

提交所有：

.. code-block:: sh

   git commit -a -m "first attempt"

然后push到heroku:

.. code-block:: sh

   git push heroku master

如果一切顺利，你将在/hi路径中的应用url下看到你的页面

记得运行 ``heroku logs`` 来检查看看是否一切正常。

fork()小白指南
******************

uWSGI允许你选择在应用中如何使用fork() syscall。

默认情况下，办法是在master进程中加载进程，然后fork()到worker，这样，worker将会继承master进程的一个拷贝。

这个方法加速了启动，并且可能会消耗更少的内存。真相是你常常(对于ruby垃圾回收工作的方式)会获得更少的内存增益。真正的优势是在性能上，因为应用启动花费的大部分时间是花在了（缓慢地）文件搜索上。使用fork() 早期方法，你可以为worker避免重复一次那个缓慢的过程。

显然，uWSGI的准则是“做任何你需要做的事，如果不能，那这它就是一个uWSGI错误”，因此，如果你的应用不是fork()友好的，那你可以添加 ``lazy-apps = true`` 选项，这将会在每个worker中加载你的应用一次。

ruby GC
***********

默认情况下，uWSGI在每次请求你后，调用ruby的垃圾收集器。这确保了内存的优化使用 (记住，在Heroku上，你的内存受限) 。你不应该动默认的方式，但如果性能下降，那么你或许想要使用 ``ruby-gc-freq = n`` 选项进行调试，这里，n是调用GC后的请求数。

并发性
***********

尽管uWSGI支持并发性的大量不同的范例，但是对于大部分的ruby/rack应用来说，建议使用多进程。

基本上，所有流行的ruby框架的依赖于多进程。记住，你的应用受限，因此，生成许多进程会适合你的Heroku dyno。

自uWSGI 1.9.14起，添加了原生ruby 1.9/2.x线程支持。Rails4 (只在生产模式！！) 支持它们：

.. code-block:: ini

   [uwsgi]
   ...
   ; spawn 8 threads per-process
   threads = 8
   ; maps them as ruby threads
   rbthreads = true
   ; do not forget to set production mode for rails4 apps !!!
   env = RAILS_ENV=production
   ...

Harakiri
********

如果你计划将生产应用放在heroku上，那么确保了解dynos和它们的代理是如何工作的。基于此，试着总是为你的应用将harakiri参数设置成一个不错的值。 (不要要求默认值，它取决于应用)

Harakiri，是一个单一的请求在被master摧毁之前可以运行的最长时间

静态文件
************

一般来讲，在Heroku上提供静态文件并不是一个好主意 (主要从设计的角度来看)。你当然可以有此需求。在这种情况下，记得使用uWSGI功能，特别是卸载（offloading）是在提供大文件时留出worker的最好方法 (另外，记得必须使用git来跟踪你的静态文件)

避免在ruby/rack代码中提供静态文件。这将会非常慢（与使用uWSGI功能相比），并且还会让你的worker忙于传输文件

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
