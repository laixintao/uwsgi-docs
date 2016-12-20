ruby/Rack应用快速入门
=====================================

以下指导将带你安装和运行一个基于Ruby的uWSGI发行版 ，旨在运行Rack应用。

安装带Ruby支持的uWSGI
**********************************

要构建uWSGI，你需要一个C编译器 (支持gcc和clang) 和Python二进制文件 (用来运行uwsgiconfig.py脚本，它将执行各种编译步骤)。

因为我们正在构建一个带Ruby支持的uWSGI二进制文件，因此我们也需要Ruby开发头文件 (基于Debian发行版上的 ``ruby-dev`` 包)。

你可以手工构建uWSGI —— 这些都是等价的：

.. code-block:: sh

   make rack
   UWSGI_PROFILE=rack make
   make PROFILE=rack
   python uwsgiconfig.py --build rack
   
但是如果你懒癌犯了，那么你可以一次性下载、构建和安装一个uWSGI + Ruby二进制文件：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s rack /tmp/uwsgi
   
或者以一种更“Ruby友好型”方式：

.. code-block:: sh

   gem install uwsgi
   
这些方法都构建一个“单片”uWSGI二进制文件。uWSGI项目是由许多插件注册的。你可以选择构建服务器核心，并为每个特性使用一个插件（需要时加载），或者你可以构建一个带有所有你需要的特性的单个二进制文件。后者这种构建称之为“单片”。

这个快速入门假设使用一个单片二进制文件 (因此你无需加载插件)。如果你更喜欢使用你的包发行版 (而不是从官方来源构建uWSGI)，那么见下。

关于发布包的注意事项
************************

你的发行版非常有可能包含一个uWSGI包集合。那些uWSGI包趋向于高模块化的 (并且偶尔高度过时)，因此，除了核心部分之外，你需要安装所需插件。必须在你的uWSGI配置中加载插件。在学习阶段，我们强烈建议不要使用发行版包，从而轻松跟着文档和教程。

一旦你适应了“uWSGI方式”，你就可以为你的部署选择最佳方式。

例如，本教程利用"http"和"rack"插件。如果你正使用模块化构建，那么确保通过 ``--plugins http,rack`` 选项加载它们。

你的第一个Rack应用
*******************

Rack是编写Ruby web应用的标准方式。

这是一个标准的Rack Hello world脚本 (称之为app.ru)：

.. code-block:: rb

   class App

     def call(environ)
       [200, {'Content-Type' => 'text/html'}, ['Hello']]
     end
     
   end
   
   run App.new
   
扩展名 ``.ru`` 表示"rackup"，这是Rack发行版中包含的部署工具。Rackup使用了一点DSL，因此，要将其用到uWSGI中，你需要安装rack gem：

.. code-block:: sh

   gem install rack
   
现在，我们准备好部署uWSGI了：

.. code-block:: sh

   uwsgi --http :8080 --http-modifier1 7 --rack app.ru

(如果‘uwsgi’并不在你当前的$PATH中，记得替换它)

或者如果你正使用的是一个模块化构建 (就像你的发行版的那个)

.. code-block:: sh

   uwsgi --plugins http,rack --http :8080 --http-modifier1 7 --rack app.ru
   
使用这个命令行，我们生成了一个HTTP代理，路由每个请求到一个进程 (名为'worker')，进程管理它并把响应发送回HTTP路由器 (接着路由器发送回客户端)。

如果你对为什么生成两个进程有疑问，那么（我告诉你）是因为这是你将会在生产上使用的正常架构 (一个带有一个后端应用服务器的前线web服务器)。

如果你不想要生成HTTP代理，而是直接强制worker应答HTTP请求，那么仅需将命令行修改为

.. code-block:: sh

   uwsgi --http-socket :8080 --http-socket-modifier1 7 --rack app.ru
   
现在，你有了单个管理请求的进程了 (但记住，直接将应用服务器公开一般是危险的，并且并不通用)。

那个'--http-modifier1 7'东东是啥？
****************************************

uWSGI支持多种语言和平台。当服务器接收到一个请求的时候，它必须知道将其“路由” 到哪里。

每个uWSGI插件都有一个分配的数字 (即modifier)，ruby/rack这个使用的是7。因此， ``--http-modifier1 7`` 意思是“路由到rack插件”。

虽然uWSGI也有一个更加“人类友好型”的 :doc:`internal routing system <InternalRouting>` ，但是，使用modifier是最快的方式，所以，尽可能使用它们。

使用一个完整的web服务器：nginx
*****************************

提供的HTTP路由器，是 (是哒，真的已经够了) 只是一个路由器。你可以将它当成一个负载均衡器或者代理使用，但是如果你需要一个完整的web服务器 (用于有效提供静态文件，或者一个web服务器所擅长的所有那些任务)，那么你可以摆脱uwsgi HTTP路由器 (如果你正使用模块化构建，那么记得修改--plugins http,rack为--plugins rack)，然后将你的应用放在Nginx之后。

为了与Nginx通信，uWSGI可以使用多个协议：HTTP, uwsgi, FastCGI, SCGI, 等等。

最有效的是uwsgi。Nginx默认支持uwsgi协议。

在一个uwsgi socket上运行你的rack应用：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru

然后在你的nginx配置中添加一个location节

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 7;
   }

重载你的nginx服务器，然后它应该开始代理请求到你的uWSGI实例。

注意，你不需要配置uWSGI来设置一个指定的modifier，nginx将会使用 ``uwsgi_modifier1 5;`` 指令来实现它。

添加并发性
******************

通过前面的例子，你部署了一个能够服务单个请求的栈。

要增加并发性，你需要添加更多的进程。如果你希望有一个魔法数学公式可以找到要生成的正确进程数，嗯……很抱歉，我们没有。你需要进行实验，并且监控你的应用，以找到正确的值。考虑到每个进程是你的应用的一个完全拷贝，因此，必须考虑内存使用。

要添加更多的进程，只需使用 `--processes <n>` 选项：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes 8
   
将生成8个进程。

Ruby 1.9/2.0引入了一个改进的线程支持，uWSGI通过'rbthreads'插件支持它。当你编译uWSGI+ruby (>=1.9)单片二进制文件的时候，会自动内建这个插件。

要添加更多线程：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --rbthreads 4
   
或者线程+进程

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes --rbthreads 4
   
有其他（一般更高级/复杂）的方式来提高并发性 (例如，'fiber')，但是，大多时候，你最终会得到一个普通的旧式多进程或者多线程模型。如果你感兴趣，那么请查看 :doc:`Rack` 之上的完整文档。

添加健壮性：Master进程
*************************************

It is highly recommended to have the uWSGI master process always running on productions apps.

It will constantly monitor your processes/threads and will add fun features like the :doc:`StatsServer`.

To enable the master simply add ``--master``

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --rack app.ru --processes 4 --master
   
使用配置文件
******************

uWSGI有数百个选项 (但一般来说，你不会使用超过数十个)。通过命令行处理它们有点蠢，因此，试着总是使用配置文件。

uWSGI支持各种标准 (XML, INI, JSON, YAML, 等等)。从一个移到另一个相当简单。你可以通过命令行使用的相同的选项也可以在配置文件中使用，只需将前缀 ``--`` 移除：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = app.ru
   processes = 4
   master = true
   
或者xml:

.. code-block:: xml

   <uwsgi>
     <socket>127.0.0.1:3031</socket>
     <rack>app.ru</rack>
     <processes>4</processes>
     <master/>
   </uwsgi>
   
要通过配置文件运行uWSGI，只需将其当做一个参数指定：

.. code-block:: sh

   uwsgi yourconfig.ini
   
如果出于某些原因，你的配置不以期望的扩展名 (.ini, .xml, .yml, .js) 结尾，那么你可以通过这种方式，强制二进制文件使用一个指定的解析器：

.. code-block:: sh

   uwsgi --ini yourconfig.foo
   
.. code-block:: sh

   uwsgi --xml yourconfig.foo

.. code-block:: sh

   uwsgi --yaml yourconfig.foo

等等等等。

你甚至可以通过管道使用配置 (使用破折号强制从标准输入中读取)：

.. code-block:: sh

   ruby myjsonconfig_generator.rb | uwsgi --json -
   
当你生成多个进程时的fork()问题
****************************************************

uWSGI某种程度上是“Perl化的”，我们没法隐藏它。它的大多数抉择 (从“不止一种实现的方式”开始) 来自于Perl世界 (更一般来说，是来自于传统的UNIX系统管理员方法)。

当应用到其他语言/平台上的时候，这个方法有时会导致意外行为。

当你开始学习uWSGI的时候，你会面对的其中一个“问题”是它的 ``fork()`` 使用。

默认情况下，uWSGI在第一个生成的进程中加载你的应用，然后多次 ``fork()`` 自身。

这意味着，你的应用被单次加载，然后被拷贝。

虽然这个方法加速了服务器的启动，但是有些应用在这项技术下会出问题 (特别是那些在启动时初始化db连接的应用，因为将会在子进程中继承连接的文件描述符)。

如果你对uWSGI使用的粗暴的preforking不确定，那么只需使用 ``--lazy-apps`` 选项来禁用它。它将会强制uWSGI在每个worker中完全加载你的应用一次。

部署Sinatra
*****************

让我们忘掉fork()，回到有趣的事情上来。这次，我们部署一个Sinatra应用：

.. code-block:: rb

   require 'sinatra'

   get '/hi' do
     "Hello World"
   end

   run Sinatra::Application
   
将其另存为 ``config.ru`` ，然后如前所见那样运行：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   
.. code-block:: sh

   uwsgi yourconf.ini
   
好吧，或许你已经注意到，基本与前面的app.ru例子没啥区别。

这是因为，基本上，每个现代的Rack应用都将其自身作为一个.ru文件（一般称为config.ru）公开，因此，无需使用多个选项来加载应用 (例如，像Python/WSGI世界中的那样)。

部署RubyOnRails >= 3
**************************

从3.0起，Rails就是完全兼容Rack的，并且它公开了一个你可以直接加载的config.ru文件 (就像我们用Sinatra那样)。

与Sinatra的唯一不同是，你的项目有一个特定的布局/约定，期望你当前的工作目录是包含项目的那个目录，因此，让我们添加一个chdir选项：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   chdir = <path_to_your_rails_app>
   env = RAILS_ENV=production
   
.. code-block:: sh

   uwsgi yourconf.ini
   
除了chdir之外，我们添加了'env'选项，它设置 ``RAILS_ENV`` 环境变量。

从4.0起，Rails支持多线程 (只适用于ruby 2.0)：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   rbthreads = 2
   lazy-apps = true
   chdir = <path_to_your_rails_app>
   env = RAILS_ENV=production

部署更老的RubyOnRails
***************************

较老的Rails版本并非完全Rack兼容的。出于这样的原因，uWSGI中有一个特定的选项，用来加载较老的Rails应用 (你也会需要'thin'这个gem)。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 4
   lazy-apps = true
   rails = <path_to_your_rails_app>
   env = RAILS_ENV=production
   
所以，长话短说，指定 ``rails`` 选项，将rails应用的目录（而不是Rackup文件）作为参数传递。

Bundler和RVM
***************

Bundler是用来管理依赖的标准的事实上的Ruby工具。基本上，你在Gemfile文本文件中指定你的应用所需的gem，然后启动bundler来安装它们。

要让uWSGI遵守bundler安装，你只需要添加：

.. code-block:: ini

   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>

(第一个并非ruby 1.9/2.x要求的必须节。)

基本上，这些行强制uWSGI加载bundler引擎，并且使用在 ``BUNDLE_GEMFILE`` 环境变量中指定的Gemfile。

当使用Bundler的时候 (就像现代框架所做的那样)，你的常见部署配置将会是：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   
除了Bundler之外，RVM是另一个常见的工具。

它允许你在单个系统中安装多个 (独立的) Ruby版本 (带自己的gemset)。

要指示uWSGI使用一个指定RVM版本的gemset，只需使用 `--gemset` 选项：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   gemset = ruby-2.0@foobar
   
只是注意，对于每个Ruby版本（是Ruby版本，不是gemset！），你都需要一个uWSGI二进制文件（或者如果你使用模块化构建，则是一个插件）。

如果你有兴趣，那么这里是构建uWSGI核心+每个rvm中安装的Ruby版本1个插件的命令列表。

.. code-block:: sh

   # build the core
   make nolang
   # build plugin for 1.8.7
   rvm use 1.8.7
   ./uwsgi --build-plugin "plugins/rack rack187"
   # build for 1.9.2
   rvm use 1.9.2
   ./uwsgi --build-plugin "plugins/rack rack192"
   # and so on...
   
然后，如果你想要使用ruby 1.9.2和@oops gemset：

.. code-block:: ini

   [uwsgi]
   plugins = ruby192
   socket = 127.0.0.1:3031
   rack = config.ru
   master = true
   processes = 4
   lazy-apps = true
   rbrequire = rubygems
   rbrequire = bundler/setup
   env = BUNDLE_GEMFILE=<path_to_your_Gemfile>
   gemset = ruby-1.9.2@oops

开机时自动启动uWSGI
************************************

如果你正想着打开vi，然后编写一个init.d脚本来生成uWSGI，那么坐下（并且淡定），首先确保你的系统并没有提供一个更好（更现代）的方法。

每个发行版都选择了一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...)，并且有大量可用的进程管理器 (supervisord, god, monit, circus...)。

uWSGI将与它们完美集成（我们希望），但是如果你计划部署大量的应用，那么看看uWSGI :doc:`Emperor<Emperor>` - 它或多或少是每个devops工程师的梦想。

安全性和可用性
*************************

总是避免以root运行你的uWSGI实例。你可以使用uid和gid选项来移除特权。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   rack = app.ru
   master = true
   processes = 8


web应用部署的一个常见问题是“卡住的请求”。你所有的线程/worker都卡住了 (请求阻塞) ，而你的应用无法接收更多的请求。

要避免这个问题，你可以设置一个 ``harakiri`` 定时器。它是一个监控器 (由master进程管理)，会摧毁那些卡住超过指定秒数的进程。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   rack = app.ru
   master = true
   processes = 8
   harakiri = 30

这将会摧毁那些阻塞超过30秒的worker。小心选择 ``harakiri`` 值！

除此之外，自uWSGI 1.9起，stats服务器导出了全部请求变量，因此，你可以（实时）看到你的实例正在做什么 (对于每个worker, thread 或者异步核)。

启用统计信息服务器是很简单的：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_to_your_app
   rack = app.ru
   master = true
   processes = 8
   harakiri = 30
   stats = 127.0.0.1:5000
   
只需将其绑定到一个地址 (UNIX或者TCP) 上，并且只需连接 (你也可以使用telnet) 到它上面，来接收你的实例的一个JSON内容。

 ``uwsgitop`` 应用 (你可以在官方的github仓库中找到它) 是一个使用统计信息服务器来实现类top实时监控的工具的例子 (有颜色！！！)

内存使用
************

低内存使用是整个uWSGI项目的卖点之一。

不幸的是，默认对内存的积极态度也许（看好：是也许）会导致某些性能问题。

默认情况下，uWSGI Rack插件在每次请求之后调用Ruby GC (垃圾回收器)。如果你想要减少这个频率，只需添加``--rb-gc-freq <n>`` 选项，其中，n是调用GC之后的请求数。

如果你计划进行uWSGI的基准 (或者把它与其他方法对比)，那么考虑它对GC的使用。

Ruby可以是一个真正的内存吞噬者，因此，我们更喜欢默认积极对待内存，而不是取悦hello-world基准点。

卸载
**********

:doc:`OffloadSubsystem` 允许你在某些特定的模式匹配的时候尽快释放你的worker，并且能够委托给一个纯c线程。例如，发送来自文件系统的静态文件，从网络传输数据到客户端，等等。

卸载是非常复杂的，但是它的使用对终端用户是透明的。如果你想要试一试，那么只需添加--offload-threads <n>，其中，<n>是要生成的线程数 (一个cpu一个线程是个不错的值)。

当启用了卸载线程时，所有可以被优化的部分将会被自动检测到。


以及现在
*********

有了这些许概念，你应该已经可以上生产了，但是uWSGI是个巨大的项目，它有数百个特性和配置。如果你想要成为一个更好的系统管理员，那么请求继续阅读完整的文档。

欢迎进入uWSGI的世界！