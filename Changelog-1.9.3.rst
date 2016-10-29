uWSGI 1.9.3
===========

更新日志 20130328


错误修复
********

fixed imports in the JVM build system when virtualenvs are used (Ryan Kaskel)

fixed mod_proxy_uwsgi with apache 2.4

fixed php headers generation when Status is created from the php app itself


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

In a recent uWSGI maling-list thread, the need to not rely on file system permissions for the tyrant mode emerged.

Albeit it is the most secure approach, two new options --immediate-uid and --immediate-gid have been added.

Setting them on top of your vassal file will force the instance to setuid()/setgid() as soon as possible and without the (theoretical) possibility to override them.

The word "theoretical" here is the key, you always need to remember that a security bug in uWSGI could allow a malicious user to change privileges, so if you really
care security (or do not trust uWSGI developers ;) always drop privileges before the vassal/instance is spawned (like in standard tyrant mode)

Honouring symlinks in tyrant mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The option --emperor-tyrant-nofollow has been added forcing the emperor to now follow symlinks when searching for uid/gid in tyrant mode.

This option allows the sysadmin to simply symlink configurations and just change the uid/gid of the symlink it self (remember to
pass the -h option to chown !!!)

The "rpcret" routing action (or usa Lua to write advanced rules)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :doc:`InternalRouting` continue to be improved.

You can already call rpc function for the routing system (to generate response bypassing WSGI/PSGI/Rack/... engines):

.. code-block:: ini

   [uwsgi]
   lua-load = myrpcfunctions.lua
   route = ^/foo/(.+)/call rpc:hello_world ${REMOTE_ADDR} $1

the hello_world rpc function is defined (and registered) in the myrpcfunctions.lua taking two arguments.

The function is called when the routing regexp matches, and its output sent to the client.

The "rpcret" works in similar way, but instead generating a response, you generate a routing return code:

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

and the uWSGI config:

.. code-block:: ini

   [uwsgi]
   route-run = rpcret:choose ${REQUEST_URI} ${REMOTE_ADDR}
   route-run = break

   route-label = topogigio
   route-run = log:i am topogigio !!!

The 'choose' rpc function will be invoked at every request passing REQUEST_URI and REMOTE_ADDR as its argument.

The return string of the function will be used to know what to do next (from the internal ruting point of view).

Currently supported return strings are:

``next`` move to the next rule

``continue`` pass the request to the request handler

``goon`` move to the next rule with a different action

``break`` close the connection with an optional status code

``goto <label>`` goto to the specified label


Obviously rpc functions for rpcret can be written in any language/platform supported by uWSGI, but we strongly suggest to go with Lua for performance reasons
(the inpact compared to pure C code is pretty irrelevant). If you are lucky and can use LuaJit you will experiment even better performance as for this kind of job
a JIT compiler is the best approach.


可用性
************

uWSGI 1.9.3 has been released on 20130328 and can be downloaded from:

http://projects.unbit.it/downloads/uwsgi-1.9.3.tar.gz
