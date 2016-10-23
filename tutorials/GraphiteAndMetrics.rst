使用Metrics子系统，在Ubuntu上构建Graphite
=========================================================

本教程将指导你安装一个多应用服务器，其中，每个应用发送度量到一个中央graphite/carbon服务器。

Graphite可以在这里找到： http://graphite.wikidot.com/

uWSGI Metrics子系统的相关文档在这里 :doc:`../Metrics`

本教程假设是在amd64的Ubuntu Saucy (13.10)版本

而对于Graphite，我们将使用Ubuntu官方包，并且将从官方渠道下载和安装uWSGI内核和插件

安装Graphite及其他需要的包
**************************************************

.. code-block:: sh

   sudo apt-get install python-dev ruby-dev bundler build-essential libpcre3-dev graphite-carbon graphite-web
   
python-dev和ruby-dev是必须的，因为我们想要同时支持WSGI和Rack应用。

pcre开发头文件运行你构建带内部路由支持（你总是想要的东东）的uWSGI

初始化Graphite
*********************

第一步僵尸启用Carbon服务器。

Graphite项目由三个子系统组成：whisper, carbon和前端

Whisper是一种数据存储格式(类似于rrdtool)

Carbon是一个服务器，用来收集度量并将其存储在whisper文件中 (好吧，它做的东西会更多些，但这是它的主要用途)

而web前端用来可视化根据carbon服务器收集到的数据构建的图表。

要启用carbon服务器，编辑 ``/etc/default/graphite-carbon`` 并且设置CARBON_CACHE_ENABLED为true

在启动carbon服务器之前，我们需要建立其搜索索引

仅需运行：

.. code-block:: sh

   sudo /usr/bin/graphite-build-search-index

然后启动carbon服务器 (在下一次重启的时候，将会自动启动它)

.. code-block:: sh

   sudo /etc/init.d/carbon-cache start

构建和安装uWSGI
*****************************

下载最新的稳定的uWSGI安装包

.. code-block:: sh

   wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
   
解压缩并在创建的目录下运行：

.. code-block:: sh

   python uwsgiconfig.py --build core
   
这将会构建uWSGI "core"二进制文件。

现在，我们想要构建python, rack和carbon插件：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/python core
   python uwsgiconfig.py --plugin plugins/rack core
   python uwsgiconfig.py --plugin plugins/carbon core
   
   
现在，我们有了 ``uwsgi``, ``python_plugin.so``, ``rack_plugin.so`` 和 ``carbon_plugin.so``

将它们拷贝到系统目录下：

.. code-block:: sh

   sudo mkdir /etc/uwsgi
   sudo mkdir /usr/lib/uwsgi
   sudo cp uwsgi /usr/bin/uwsgi
   sudo cp python_plugin.so /usr/lib/uwsgi
   sudo cp rack_plugin.so /usr/lib/uwsgi
   sudo cp carbon_plugin.so /usr/lib/uwsgi

设置uWSGI Emperor
****************************

创建一个upstart配置文件，用以启动 :doc:`../Emperor`

.. code-block:: sh

   # Emperor uWSGI script

   description "uWSGI Emperor"
   start on runlevel [2345]
   stop on runlevel [06]

   exec /usr/bin/uwsgi --emperor /etc/uwsgi
   
将其保存为 ``/etc/init/emperor.conf`` ，然后启动Emperor:

.. code-block:: sh

   start emperor
   
   
从现在起，要启动uWSGI实例，仅需把它们的配置文件放到/etc/uwsgi中

生成Graphite web界面
***********************************

在启动graphite web界面 (它是一个Django应用)前，我们需要初始化它的数据库。

运行：

.. code-block:: sh

   sudo graphite-manage syncdb
   
这是manage.py的一个标准的django syncdb命令。仅需回答问题来创建一个admin用户即可。

现在，我们准备好创建一个uWSGI vassal了：

.. code-block:: ini

   [uwsgi]
   plugins-dir = /usr/lib/uwsgi
   plugins = python
   uid = _graphite
   gid = _graphite
   wsgi-file = /usr/share/graphite-web/graphite.wsgi
   http-socket = :8080
   
将其保存为 ``/etc/uwsgi/graphite.ini``
   
 _graphite 用户 (和组)由graphite ubuntu包创建。我们的uWSGI vassal将会运行在此权限下运行。

web界面将会监听服务器的8080端口，使用HTTP。如果你更喜欢代理，那么仅需修改 ``http-socket`` 为 ``http`` ，或者将其放在一个诸如nginx这样的完整的web服务器之后 (本教程中并不涵盖此步骤)


生成vassals，用以发送度量给Graphite
********************************************

现在，已经准备好发送应用的度量给carbon/graphite服务器了。

对于/etc/uwsgi中的每一个vassal文件，仅需确保添加以下选项：

.. code-block:: ini

   [uwsgi]
   ...
   plugins = carbon
   enable-metrics = true
   carbon-use-metrics = true
   carbon-id = %n
   carbon = 127.0.0.1:2003
   ...

 ``carbon-id`` 给每个度量设置了一个有意义的前缀 (%n自动转换成不带扩展名的vassal文件名)。

 ``carbon`` 选项设置carbon服务器的地址以接收度量值 (默认情况下，carbon服务器绑定在2003端口，但是你可以通过编辑 ``/etc/carbon/carbon.conf`` 然后重启carbon服务器来修改它)

将Graphiti (基于Ruby/Sinatra) 作为替代前端
***********************************************************

Graphiti是来自Graphite的一个替代web面板/前端，用Sinatra (一个Ruby/Rack框架)编写。

Graphiti需要redis，因此，确保你的系统中运行着一个redis服务器。

运行：

.. code-block:: sh

   sudo apt-get install redis-server
   
就够了

第一步是clone这个graphiti应用 (将其放在任何你想要/需要的地方)：

.. code-block:: sh

   git clone https://github.com/paperlesspost/graphiti.git
   
然后运行bundler工具 (如果你对ruby世界没信心，那么（我告诉你），它是一个用来管理依赖的工具)

.. code-block:: sh

   bundle install

.. 注意:: 如果eventmachine gem安装失败，那么在Gemfile中添加"gem 'eventmachine'"以作为第一个gem，然后运行bundle update。这会确保安装最新的eventmachine版本

在bundle安装了所有的gem之后，你必须拷贝graphiti样例配置：

.. code-block:: sh

   cp config/settings.yml.example config/settings.yml
   
编辑它，设置graphite_base_url为graphite web界面(django那个)运行的url。

现在，我们可以在uWSGI上部署它了

.. code-block:: ini

   [uwsgi]
   plugins-dir = /usr/lib/uwsgi
   plugins = rack
   chdir = <path_to_graphiti>
   rack = config.ru
   rbrequire = bundler/setup
   http-socket = :9191
   uid = _graphite
   gid = _graphite
   
将其保存为 ``/etc/uwsgi/graphiti.ini`` ，让Emperor部署它

现在，你可以连接到9191端口来管理你收集的度量了。

与往常一样，你可以随意将实例放在代理之后。

小抄
*****

默认情况下，carbon服务器监听公有地址。除非你清楚你正在做什么，否则你应该将其指向一个本地地址 (例如127.0.0.1)

uWSGI导出大量的度量 (更多是有计划的)，不要害怕使用它们

应用和carbon服务器之间不具备安全性，任何应用都能写入度量值。如果你托管不信任的应用，那么最好使用其他方法 (例如，给系统中的每个用户一个graphite实例)

对于redis同样适用，如果你允许不受信任的应用，那么从安全的角度来看，一个共享的redis实例绝对不是一个好的选择。
