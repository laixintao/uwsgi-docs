安装uWSGI
================

从发布包安装
--------------------------------------

在一些操作系统/发布版本中，uWSGI已作为软件包提供。

通过源码包安装
----------------------

要构建uWSGI，你需要Python和一个C编译器 (支持 ``gcc`` 和 ``clang`` )。取决于你想要支持的语言，你将需要它们的开发头文件。在Debian/Ubuntu系统上，你可以这样安装它们（以及构建软件所需的其余的基础设施）：

.. code-block:: sh

   apt-get install build-essential python

而如果你想要构建一个带有Python/wsgi支持的二进制文件的话 (举个例子)

.. code-block:: sh

   apt-get install python-dev

在Fedora/Redhat系统上，你可以这样安装它们：

.. code-block:: sh

   yum groupinstall "Development Tools"
   yum install python

而对于python/wsgi支持：

.. code-block:: sh

   yum install python-devel


如果你的系统中有了可用的 `make` 变体，那么你可以简单运行 `make` 。如果没有 `make` (或者想有更多的控制权)，那么运行：

.. code-block:: sh

   python uwsgiconfig.py --build

你还可以使用pip来安装uWSGI (它将会构建一个带python支持的二进制文件)。

.. code-block:: sh

   # Install the latest stable release:
   pip install uwsgi
   # ... or if you want to install the latest LTS (long term support) release,
   pip install http://projects.unbit.it/downloads/uwsgi-lts.tar.gz

或者你可以使用ruby gems (它将会构建一个带ruby/rack支持的二进制文件)。

.. code-block:: sh

   # Install the latest stable release:
   gem install uwsgi


在构建的最后，你将会获得启用特性的报告。如果缺少了任何你需要的，那么仅需添加开发头文件，然后重新运行构建。例如，要构建带ssl和perl正则表达式支持的uWSGI，你需要libssl-dev和pcre头文件。

可替换构建配置文件
--------------------------

出于历史原因，当你运行'make'的时候，会构建把Python作为唯一支持的语言的uWSGI。你可以使用构建配置文件来自定义uWSGI服务器，配置文件位于 `buildconf/` 目录中。你可以这样使用一个特定的配置文件：

.. code-block:: sh

   python uwsgiconfig.py --build <profile>

或者你可以通过一个环境变量传递它：

.. code-block:: sh

   UWSGI_PROFILE=lua make
   # ... or even ...
   UWSGI_PROFILE=gevent pip install uwsgi


模块化构建
--------------

这是你的发行版应该遵循的方法，也是在你想要在uWSGI之上构建一个商用服务必须遵循的方法（见下）。大部分的uWSGI特性是作为插件提供的。可以使用--plugin选项来加载插件。如果你想提供给用户最大量的灵活性，让他们只使用最少的资源，那么仅需创建一个模块化构建。一个名为"core"的构建配置文件是可用的。

.. code-block:: sh

   python uwsgiconfig.py --build core

这将会狗偶家一个不带插件的uWSGi二进制文件。这被称为"服务器核心"。现在，你可以开始构建所需的所有的插件。检查源代码发布中的plugins/ 目录，以获取完整列表。

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/psgi core
   python uwsgiconfig.py --plugin plugins/rack core
   python uwsgiconfig.py --plugin plugins/python core
   python uwsgiconfig.py --plugin plugins/lua core
   python uwsgiconfig.py --plugin plugins/corerouter core
   python uwsgiconfig.py --plugin plugins/http core
   ...

记得要传递构建配置文件 (这里是'core') 作为第三个参数。
   
