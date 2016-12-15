perl/PSGI应用快速入门
=====================================


以下指导将会代理安装和运行一个基于perl的uWSGI发行版，旨在运行PSGI应用。


安装带Perl支持的uWSGI
**********************************

要构建uWSGI，你需要一个c编译器 (支持gcc和clang) 和python二进制文件 (它将只运行uwsgiconfig.py脚本，这个脚本将会执行各种编译步骤)。由于我们要构建一个带perl支持的uWSGI二进制文件，因此我们也需要perl开发头文件 (基于debian发行版的libperl-dev包)

你可以手工构建uWSGI：

.. code-block:: sh

   python uwsgiconfig.py --build psgi
   
这与下面相同


.. code-block:: sh

   UWSGI_PROFILE=psgi make
   
或者使用网络安装程序：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s psgi /tmp/uwsgi
   
那会在/tmp/uwsgi中创建一个uWSGI二进制文件 (随意修改路径为任意你想要的)

发行版包的注意事项
************************

你的发行版非常有可能包含了一个uWSGI包集，所以，除了核心部分之外，你需要安装所需插件。必须在你的配置中加载插件。在学习阶段，我们强烈建议不要使用发行版包，才能轻松遵循文档和教程。

一旦你适应了“uWSGI方式”，你就可以为你的部署选择最佳方法。

你第一个PSGI应用
*******************

将其保存为一个名为myapp.pl的文件

.. code-block:: pl

   my $app = sub {
        my $env = shift;
        return [
                '200',
                [ 'Content-Type' => 'text/html' ],
                [ "<h1>Hello World</h1>" ],
        ];
   };

然后通过uWSGI，在http模式下运行它：

.. code-block:: sh

   uwsgi --http :8080 --http-modifier1 5 --psgi myapp.pl

(如果'uwsgi'不在你当前的$PATH中，那么记得替换它)

或者如果你使用模块化构建 (就像你发行版那样)

.. code-block:: sh

   uwsgi --plugins http,psgi --http :8080 --http-modifier1 5 --psgi myapp.pl
   
.. note:: 当你有一个前端web服务器的时候，不要使用--http，使用--http-socket。接着读快速入门以了解为什么。

那个'--http-modifier1 5'是啥鬼东西？？？
*******************************************

uWSGI支持各种的语言和平台。当服务器接收到了一个请求的时候，它必须知道要把它“路由”到哪里。

每个uWSGI插件都被分配了一个数字 (modifier)，perl/psgi这个分配到的是5。所以--http-modifier1 5表示"路由到psgi插件"

尽管uWSGI有一个更加"人类友好的" :doc:`internal routing system <InternalRouting>` ，但是使用modifier是最快的方式，所以，如果有可能的话，总是使用它们吧。


使用一个完整的web服务器：nginx
*******************************

提供的http路由器，只是 (对哒，令人难以置信的) 一个路由器。你可以将它作为一个负载均衡器或者代理使用，但是如果你需要一个完整的web服务器 (来有效地提供静态文件，或者所有web服务器擅长的那些任务)，那么你可以摆脱uwsgi http路由器 (如果你使用模块化构建，那么记得修改--plugins http,psgi为--plugins psgi)，然后将你的应用放在nginx之后。

要与nginx通信，uWSGI可以使用各种协议：http, uwsgi, fastcgi, scgi...

最有效的是uwsgi那个。Nginx默认包含uwsgi协议支持。

在一个uwsgi socket之上运行你的psgi应用：

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl

然后在你的nginx配置中添加一个location节


.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 5;
   }

重载你的nginx服务器，它应该开始代理请求到你的uWSGI实例

注意，你不需要配置uWSGI来设置一个指定的modifier，nginx将会使用 ``uwsgi_modifier1 5;`` 指令来做到这点

如果你的代理/web服务器/路由器使用HTTP，那么你必须告诉uWSGI原生使用http协议 (这与--http不同，后者自己会生成一个代理)：

.. code-block:: sh

   uwsgi --http-socket 127.0.0.1:3031 --http-socket-modifier1 5 --psgi myapp.pl
   
正如你所见，我们需要指定使用的modifier1，因为http协议不能够携带这类信息


添加并发
******************

你可以通过多进程、多线程或者各种异步模式为你的应用提供并发性。

要生成额外的进程，使用--processes选项

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --processes 4

要拥有额外的线程，使用--threads

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --threads 8

或者两者一起来

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --threads 8 --processes 4
   
perl世界中一个非常常见的非阻塞/协程库是Coro::AnyEvent。uWSGI可以简单包括 ``coroae`` 插件来使用它 (甚至与多进程结合在一起)。

要构建一个带 ``coroae`` 支持的uWSGI二进制文件，只需运行

.. code-block:: sh

   UWSGI_PROFILE=coroae make
   
或者

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s coroae /tmp/uwsgi
   
你将得到一个同时包含了 ``psgi`` 和 ``coroae`` 插件的uWSGI二进制文件。

现在，以Coro::AnyEvent模式运行你的应用：


.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --coroae 1000 --processes 4
   
它将运行4个进程，每个都能够管理多达1000个协程 (或者Coro微线程)。


添加鲁棒性：Master进程
*************************************

高度推荐在生产应用上总是运行master进程。

它将不断监控你的进程/线程，并且会添加有趣的特性，例如 :doc:`StatsServer`

要启用master，只需添加--master

.. code-block:: sh

   uwsgi --socket 127.0.0.1:3031 --psgi myapp.pl --processes 4 --master
   
使用配置文件
******************

uWSGI有数百个选项。通过命令行与它们打交道基本上是种愚蠢的行为，所以试着一直使用配置文件吧。uWSGI支持各种标准 (xml, .ini, json, yaml...)。从一种移到另一种是相当简单的。你通过命令行可以使用的选项也一样可以在配置文件中使用，只需要移除 ``--`` 前缀即可：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   psgi = myapp.pl
   processes = 4
   master = true
   
或者xml:

.. code-block:: xml

   <uwsgi>
     <socket>127.0.0.1:3031</socket>
     <psgi>myapp.pl</psgi>
     <processes>4</processes>
     <master/>
   </uwsgi>
   
要使用配置文件来运行uWSGI，只需将其作为一个参数指定：

.. code-block:: sh

   uwsgi yourconfig.ini
   
如果出于某些原因，你的配置不能以期望的扩展名 (.ini, .xml, .yml, .js) 结尾，那么你可以这样强制二进制文件使用一个指定的解析器：

.. code-block:: sh

   uwsgi --ini yourconfig.foo
   
.. code-block:: sh

   uwsgi --xml yourconfig.foo

.. code-block:: sh

   uwsgi --yaml yourconfig.foo

等等等等

你甚至可以使用管道配置 (使用破折号来强制从标准输入读取):

.. code-block:: sh

   perl myjsonconfig_generator.pl | uwsgi --json -


自动在开机启动uWSGI
************************************

如果你在考虑编写一些init.d脚本来生成uWSGI，那么只需坐下（并且冷静下来），检查看看你的系统是否没给你提供一个更好的（更现代的）方法。

每个发行版都选择了一个启动系统 (:doc:`Upstart<Upstart>`, :doc:`Systemd`...)，并且有大量可用的进程管理器 (supervisord, god...).

uWSGI将会和它们很好的集成（我们希望是），但是如果你计划部署大量的应用，那么看看uWSGI :doc:`Emperor<Emperor>` ，它是每个devops的梦想。

安全性和可用性
*************************

总是避免将你的uWSGI实例作为root运行。你可以使用uid和gid选项来移除特权

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8


web应用的部署的一个常见问题是“卡住请求”。你所有的线程/worker都卡在一个请求上了，导致你的应用不能再接收请求。

要避免那个问题，你可以设置一个 ``harakiri`` 定时器。它是一个监控器 (由master进程管理)，它将销毁那些卡住超过指定秒数的进程

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8
   harakiri = 30

将会摧毁那些阻塞超过30秒的worker。小心选择harakiri的值！！！

除此之外，自uWSGI 1.9起，统计信息服务器导出了整套请求变量，所以你可以看到（实时）你的实例在做什么 (对每个worker，线程或者异步核)

启用统计信息服务器是很容易的：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   uid = foo
   gid = bar
   chdir = path_toyour_app
   psgi = myapp.pl
   master = true
   processes = 8
   harakiri = 30
   stats = 127.0.0.1:5000
   
只需将其绑定到一个地址 (UNIX或者TCP)，然后连接上 (你也可以使用telnet) 它，来接收你的实例的一个JSON表示。

 ``uwsgitop`` 应用 (你可以在官方的github仓库中找到它) 是一个使用统计信息服务器来实现类top实时监控的工具的例子 (有颜色！！！)


卸载
**********

:doc:`OffloadSubsystem` 允许你在某些特定的模式匹配的时候尽快释放你的worker，并且能够委托给一个纯c线程。例如，发送来自文件系统的静态文件，从网络传输数据到客户端，等等。

卸载是非常复杂的，但是它的使用对终端用户是透明的。如果你想要试一试，那么只需添加--offload-threads <n>，其中，<n>是要生成的线程数 (一个cpu一个线程是个不错的值)。

当启用了卸载线程时，所有可以被优化的部分将会被自动检测到。


以及现在
*********

有了这些许概念，你应该已经可以上生产了，但是uWSGI是个巨大的项目，它有数百个特性和配置。如果你想要成为一个更好的系统管理员，那么请求继续阅读完整的文档。
