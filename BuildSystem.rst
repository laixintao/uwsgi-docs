uWSGI构建系统
======================

- 更新至1.9.13

本页面描述了uWSGI构建系统是如何工作的，以及如何对其进行定制。

uwsgiconfig.py
**************

这是一个旨在调用不同编译/链接阶段的python脚本。

在2009年期间，当开始定义uWSGI指导方针 (以及准则) 时，人们一致认为，autotools, cmake和它们的小伙伴们不受许多系统管理员喜爱。尽管它们相当标准，但是需要包数量，以及它们之前的不兼容（特别是在autotools的世界中）对于一个具有快速开发/演进的项目来说，是个问题，其中“从源代码编译”曾经，现在是，并且可能将是从产品获得最好的最好的方式。除此之外，构建过程一定要快 (在入门级的x86上少于1分钟是主要规则)

出于这样的原因，要编译uWSGI，你只需要有一个c编译套件 (gcc, clang...) 和一个python解释器。有人可能会说，perl可能是一个更好的选择，也许这是事实 (默认情况下，许多操作系统一般都会安装它)，但是我们还是决定主要用python，因为，在uWSGI项目启动的时候，它是一个仅支持python的应用。(显然，如果你想要开发一个可替换的构建系统，请随意)

基本上，uwsgiconfig.py检测系统中的可用特性，并使用所谓的“构建配置文件”构建一个uwsgi二进制（及其插件）

构建配置文件
**************

第一个例子
*************

CC和CPP
**********

这2个环境变量告诉uwsgiconfig.py使用一个替换的C编译器和C预处理器。

如果不定义它们，那么过程如下：

对于CC -> 尝试从运行uwsgiconfig.py的python二进制中获得CC config_var，回退到'gcc'

对于CPP -> 回退到'cpp'


例如，在一个同时拥有gcc和clang的系统上，你会得到

.. code-block:: sh

   CC=clang CPP=clang-cpp python uwsgiconfig.py --build

CPUCOUNT
********

在“即使在生产系统上，也要轻松快速构建”的精神下，uwsgiconfig.py试着使用你所有的cpu核来生成c编译器的多个实例 (每个核一个)。

你可以使用CPUCOUNT环境变量来覆盖这个系统，强制已检测的CPU核数目 (设为1将会禁用并行构建)。

.. code-block:: sh

   CPUCOUNT=2 python uwsgiconfig.py --build

UWSGI_FORCE_REBUILD
*******************

插件和uwsgiplugin.py
**************************


一个uWSGI插件是一个导出<name>_plugin符号的共享库。其中，<name>是插件名。

例如，psgi插件将会导出psgi_plugin符号，而pypy将会导出pypy_plugin符号，以此类推。

这个符号是一个uwsgi_plugin C结构，定义插件的钩子。

当你让uWSGI加载一个插件的时候，它简单调用dlopen()，并通过dlsym()获得uwsgi_plugin结构。

uWSGI项目的绝大部分是作为插件开发的，这保证了配置的模块化，以及明显理智的发展方式。

系统管理员可以随意嵌入每个插件在服务器二进制文件中，或将每个插件当成一个外部共享库构建。

已嵌入的插件在构建配置文件的'embedded_plugins'指令中定义。你可以使用UWSGI_EMBED_PLUGINS环境变量，从命令行添加更多嵌入插件（见下）。

事实上，如果你想要把一个插件作为共享库构建，那么只需带--plugin选项运行uwsgiconfig.py

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/psgi
   
这将会构建plugins/psgi中的插件为psgi_plugin.so文件

要在构建插件的时候，指定一个构建配置文件，你可以把这个配置文件当成一个额外的参数传递

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/psgi mybuildprofile

UWSGI_INCLUDES
**************

- 这个已在1.9.13添加

在启动时，运行CPP二进制文件来检测默认的包含路径。你可以使用UWSGI_INCLUDES环境变量来添加更多路径

.. code-block:: sh

   UWSGI_INCLUDES=/usr/local/include,/opt/dev/include python uwsgiconfig.py --build

UWSGI_EMBED_PLUGINS
*******************

UWSGI_EMBED_CONFIG
******************

允许嵌入指定的.ini文件到服务器二进制文件中 (目前仅支持Linux)

在启动的时候，服务器立即解析嵌入的文件。

嵌入的配置中自定义的选项将如标准选项一般可用。

UWSGI_BIN_NAME
**************

CFLAGS和LDFLAGS
******************

用于插件的 UWSGICONFIG_* 
*************************

libuwsgi.so
***********

uwsgibuild.log
**************

uwsgibuild.lastcflags
*********************

cflags和uwsgi.h magic
************************

嵌入文件
***************

fake make
*************
