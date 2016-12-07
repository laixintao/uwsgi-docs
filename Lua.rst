联合uWSGI使用Lua/WSAPI
==========================

为uWSGI 2.0进行了更新

构建插件
-------------------

lua插件是官方的uWSGI发行版的一部分 (官方的modifier 6)，你可以在plugins/lua目录下找到它。

该插件支持lua 5.1, lua 5.2和luajit。

默认情况下，假设使用lua 5.1

如往常一样，有多种方法来构建和安装Lua支持：

从源代码目录：

.. code-block:: sh

   make lua
   
通过安装程序 (结果二进制文件将会出现在/tmp/uwsgi)

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s lua /tmp/uwsgi
   
或者你可以把它当成一个插件进行构建

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/lua
   
或者 (如果你已经有了一个uwsgi二进制文件)

.. code-block:: sh

   uwsgi --build-plugin plugins/lua
   
构建系统 (查看plugins/lua目录中的uwsgiplugin.py以获取更多信息) 使用pkg-config来查找头文件和库。

你可以指定pkg-config模块来使用UWSGICONFIG_LUAPC环境变量。

例如

.. code-block:: sh

   UWSGICONFIG_LUAPC=lua5.2 make lua
   
将会为lua 5.2构建一个uwsgi二进制文件

以及

.. code-block:: sh

   UWSGICONFIG_LUAPC=luajit make lua
   
将会为luajit构建一个uwsgi二进制文件

如果你不想依赖于pkg-config工具，那么你可以通过以下环境变量手动指定包含和库目录，以及lib名字：

.. code-block:: sh

   UWSGICONFIG_LUAINC=<directory>
   UWSGICONFIG_LUALIBPATH=<directory>
   UWSGICONFIG_LUALIB=<name>
   
为什么是Lua ?
---------

如果你是从其他面向对象的语言来的，那么你可能会发现对于web开发来说，lua是一个奇怪的选择。

那么，在探索Lua的时候，你必须考虑一件事：它很快，真的很快，并且消耗非常少的资源。

uWSGI插件允许你用lua编写web应用，但是另一个目的（如果不是主要目的）是使用Lua来扩展使用信号框架、rpc子系统或者简单的钩子引擎的uWSGI服务器 (和你的应用)。

如果你的代码中存在慢区域（独立于语言使用），那么考虑用Lua (在用C处理之前) 重写它们，并且使用uWSGI来安全调用它们。

你的第一个WSAPI应用
----------------------------

我们将使用官方的WSAPI例子，让我们称之为 :file:`pippo.lua` ：

.. code-block:: lua

  function hello(wsapi_env)
    local headers = { ["Content-type"] = "text/html" }
    local function hello_text()
      coroutine.yield("<html><body>")
      coroutine.yield("<p>Hello Wsapi!</p>")
      coroutine.yield("<p>PATH_INFO: " .. wsapi_env.PATH_INFO .. "</p>")
      coroutine.yield("<p>SCRIPT_NAME: " .. wsapi_env.SCRIPT_NAME .. "</p>")
      coroutine.yield("</body></html>")
    end
    return 200, headers, coroutine.wrap(hello_text)
  end
  
  return hello

现在用 ``lua`` 选项来运行uWSGI (如果你将其当做插件使用，那么记得添加 ``--plugins lua`` 作为第一个命令行选项)

.. code-block:: sh

  ./uwsgi --http :8080 --http-modifier1 6 --lua pippo.lua

这个命令行启动了一个http路由器，它转发请求到单个加载了pippo.lua的worker。

正如你可以见到的那样，强制使用modifier 6。

显然，你可以直接附加uWSGI到你的前线web服务器（例如nginx）上，并将其绑定到一个uwsgi socket：

.. code-block:: sh

  ./uwsgi --socket 127.0.0.1:3031 --lua pippo.lua

(记得在你选择的web服务器中设置modifier1为6)

并发
-----------

基本上，在所有支持的uWSGI并发模型中，都能用Lua

你可以用多进程：

.. code-block:: sh

  ./uwsgi --socket 127.0.0.1:3031 --lua pippo.lua --processes 8 --master
  
  
或者多线程：

.. code-block:: sh

  ./uwsgi --socket 127.0.0.1:3031 --lua pippo.lua --threads 8 --master
  
或者同时使用

.. code-block:: sh

  ./uwsgi --socket 127.0.0.1:3031 --lua pippo.lua --processes 4 --threads 8 --master
  
你可以以协程模式 (见下) 运行它，使用 :doc:`uGreen` 作为挂起引擎

.. code-block:: sh

  ./uwsgi --socket 127.0.0.1:3031 --lua pippo.lua --async 1000 --ugreen
  
线程和异步模式每个都将初始化lua状态 (你可以将其视为一个完整的独立lua VM)

尽情使用协程
------------------

Lua最让人兴奋的特性之一是协程（协作多线程）支持。uWSGI使用它的异步引擎，可以从中受益。Lua插件将会为每个异步核心初始化一个 ``lua_State`` 。我们将使用我们的pippo.lua的CPU密集型版本来测试它：

.. code-block:: lua

  function hello(wsapi_env)
    local headers = { ["Content-type"] = "text/html" }

    local function hello_text()
      coroutine.yield("<html><body>")
      coroutine.yield("<p>Hello Wsapi!</p>")
      coroutine.yield("<p>PATH_INFO: " .. wsapi_env.PATH_INFO .. "</p>")
      coroutine.yield("<p>SCRIPT_NAME: " .. wsapi_env.SCRIPT_NAME .. "</p>")
      for i=0, 10000, 1 do
          coroutine.yield(i .. "<br/>")
      end
      coroutine.yield("</body></html>")
    end

    return 200, headers, coroutine.wrap(hello_text)
  end

  return hello

并且运行带有8个异步核心的uWSGI……

.. code-block:: sh

  ./uwsgi --socket :3031 --lua pippo.lua --async 8

就这样，你可以在单个worker中管理8个并发请求！

Lua协程并非在C栈之上运行的 (意味着你不能用你的C代码来管理它们)，但多亏了 :doc:`uGreen` (uWSGI官方协程/绿色线程引擎)，你可以绕过这个限制。

多亏了uGreen，你可以在你的Lua应用中使用uWSGI异步API，获得一个非常高层次的并发。


.. code-block:: lua

   uwsgi.async_connect
   uwsgi.wait_fd_read
   uwsgi.wait_fd_write
   uwsgi.is_connected
   uwsgi.send
   uwsgi.recv
   uwsgi.close
   uwsgi.ready_fd

线程例子
-----------------

Lua插件是“线程安全的”，因为uWSGI因为一个lua_State到每个内部pthread。例如，你可以非常容易地运行Sputnik_ wiki引擎。使用
LuaRocks_ 来安装Sputnik和 ``versium-sqlite3`` 。要求有一个数据库备份存储，因为默认的文件系统存储并不支持多个解释器并发访问。创建一个wsapi兼容文件：

.. code-block:: lua

    require('sputnik')
    return sputnik.wsapi_app.new{
      VERSIUM_STORAGE_MODULE = "versium.sqlite3", 
      VERSIUM_PARAMS = {'/tmp/sputnik.db'},
      SHOW_STACK_TRACE = true,
      TOKEN_SALT = 'xxx',
      BASE_URL       = '/',
    }

并运行你的线程化uWSGI服务器

.. code-block:: sh

  ./uwsgi --plugins lua --lua sputnik.ws --threads 20 --socket :3031

.. _Sputnik: http://sputnik.freewisdom.org/
.. _LuaRocks: http://www.luarocks.org/

内存的注意事项
----------------

我们都知道，uWSGI对内存的吝啬。内存是一种珍贵的字眼。不要信任那些不关心你的内存的软件！在每个请求之后，（默认）会自动调用Lua垃圾收集器。

你可以使用 ``--lua-gc-freq <n>`` 选项来调整GC调用的频率，其中，n是将会调用GC之后的请求数目：

.. code-block:: ini

   [uwsgi]
   plugins = lua
   socket = 127.0.0.1:3031
   processes = 4
   master = true
   lua = foobar.lua
   ; run the gc every 10 requests
   lua-gc-freq = 10
   
RPC和信号
---------------

Lua shell
-------------

把Lua当成一个“配置器”使用
---------------------------

uWSGI api状态
----------------
