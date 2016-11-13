uWSGI 1.9.3
===========

更新日志 20130328


错误修复
********

修复当使用虚拟环境的时候，在JVM构建系统中的导入问题 (Ryan Kaskel)

修复apache 2.4的mod_proxy_uwsgi问题

修复当Status是由php应用自身创建的时候的php头部问题


新特性
********

可插拔配置系统 (带Lua支持)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

从此版本起，你将能够将配置器 (例如已经可用的xml, ini, yaml, json, ldap, sqlite...) 作为插件实现。

第一个可用的配置器是Lua (由lua插件提供)。

这是一个用Lua写的样例配置：

.. code-block:: lua

   config = {}

   config['immediate-uid'] = 'roberto'
   config['immediate-gid'] = 'roberto'
   config['http-socket'] = ':9090'
   config['env'] = { 'FOO=bar', 'TEST=topogigio' }
   config['module'] = 'werkzeug.testapp:test_app'

   return config

你可以这样加载它：

.. code-block:: sh

   uwsgi --plugin lua --config config.lua

--config选项是加载可插拔配置器的方式。你甚至可以使用自己的版本来覆盖已经可用的内嵌的配置器。

Emperor已被扩展来支持可插拔配置器：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-extra-extension = .lua
   emperor-extra-extension = .foo

增加emperor-extra-extension将会允许emperor搜索指定的扩展名，使用--config选项传递配置文件给vassal。

即用setuid和setgid
^^^^^^^^^^^^^^^^^^^^^^^^^^^

在最近的uWSGI邮件列表支线中，对于tyrant模式，不依赖于文件系统权限的需求应运而生。

尽管它是最安全的方法，但是还是添加了两个新的选项--immediate-uid和--immediate-gid。

在你的vassal文件顶部设置它们将会强制实例尽快setuid()/setgid()，并且（理论上来讲）不可能覆盖掉它们。

这里，“理论上来讲”这个词是关键，你需要记住，uWSGI中的一个安全性问题可能允许恶意用户修改权限，因此，如果你真的在意安全性 (或者不信任uWSGI开发者;) ，那么就要在生成vassal/实例之前删除权限 (就像在标准的tyrant模式中)

tyrant模式中的遵循符号链接（Honouring symlinks）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

已经添加了选项--emperor-tyrant-nofollow，用来在tyrant模式中搜索uid/gid的时候，迫使emperor遵循符号链接。

该选项允许系统管理员简单的符号连接配置，并且只是修改符号连接自身的uid/gid (记得把-h选项传递给chown!!!)

"rpcret"路由动作 (或者使用Lua编写高级规则)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:doc:`InternalRouting` 在持续改善中。

你已经可以为路由系统调用rpc函数了 (生成绕过WSGI/PSGI/Rack/...引擎的响应):

.. code-block:: ini

   [uwsgi]
   lua-load = myrpcfunctions.lua
   route = ^/foo/(.+)/call rpc:hello_world ${REMOTE_ADDR} $1

hello_world rpc函数是在myrpcfunctions.lua中定义（以及注册）的，它接收两个参数。

当路由正则表达式匹配上的时候，会调用该函数，而它的输出会被发送给客户端。

"rpcret"工作方式相似，但不是生成响应，而是生成一个路由返回码：

.. code-block:: lua

   function choose(request_uri, remote_addr)
      print( 'REQUEST_URI is ' ..request_uri.. ' (from Lua)')
      if request_uri == '/topogigio' then
        return "goto topogigio"
      end
      return "break 500 Internal server Error !!!"
   end

   print('Hello Hello')
   uwsgi.register_rpc('choose', choose)

以及uWSGI配置：

.. code-block:: ini

   [uwsgi]
   route-run = rpcret:choose ${REQUEST_URI} ${REMOTE_ADDR}
   route-run = break

   route-label = topogigio
   route-run = log:i am topogigio !!!

这个'choose' rpc函数将会在每个请求中被调用，传递REQUEST_URI和REMOTE_ADDR作为它的参数。

函数的返回字符串将会被用来指示下一步做什么 (从内部路由的角度)。

目前支持的返回字符串是：

``next`` 移到下一条规则

``continue`` 将请求传递给请求处理器

``goon`` 移动到下一个有着不同动作的规则

``break`` 关闭连接，有一个可选的状态码

``goto <label>`` 跳到指定的标签


显然，可以在任何uWSGI支持的语言/平台中编写用于rpcret的rpc函数，但是我们强烈建议出于性能原因，使用Lua
(与纯C相比，影响是非常不相关的)。如果你幸运，可以使用LuaJit，你会体验到甚至更好的性能，因为对于这种任务，JIT编译器是最好的方法。


可用性
************

uWSGI 1.9.3已经在20130328发布，你可以从下面的地址中下载：

http://projects.unbit.it/downloads/uwsgi-1.9.3.tar.gz
