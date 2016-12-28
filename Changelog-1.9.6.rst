uWSGI 1.9.6
===========

更新日志20130409

错误修复
********

* 使用gcc 4.8构建python插件的解决方法

抱歉，这并不是一个正在的错误修复，但是一个发布没有错误修复似乎是不对的……

新特性
************

Sqlite和LDAP插件化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

将配置存储在sqlite数据库或者LDAP树中是一个非常“罕见的”配置uWSGI实例的方式。出于这样的原因，已将它们转移到专用插件。

如果你将配置存储到一个sqlite数据库库中，只需添加--plugin sqlite3。对于LDAP，仅需条件--plugin ldap：

.. code-block:: sh

   uwsgi --plugin sqlite --sqlite config.db

配置带内部路由的动态应用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

直至现在，你需要配置你的web服务器动态加载应用。

添加了三条新指令来按需加载应用。

看看这个例子：

.. code-block:: ini

   [uwsgi]

   http-socket = :9090

   route = ^/foo chdir:/tmp
   route = ^/foo log:SCRIPT_NAME=${SCRIPT_NAME}
   route = ^/foo log:URI=${REQUEST_URI}
   route = ^/foo sethome:/var/uwsgi/venv001
   route = ^/foo setfile:/var/uwsgi/app001.py
   route = ^/foo break:

   route = ^/bar chdir:/var
   route = ^/bar addvar:SCRIPT_NAME=/bar
   route = ^/bar sethome:/var/uwsgi/venv002
   route = ^/bar setfile:/var/uwsgi/app002.py
   route = ^/bar break:

正如你所看到的，现在重写SCRIPT_NAME非常容易。sethome指令目前仅适用于python应用
(它表示'virtualenv')

Carbon平均值计算 (作者：Łukasz Mierzwa)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在，你可以配置carbon插件在尚未管理任何请求的时候，如何发送响应平均数。

有三种方式：

   --carbon-idle-avg none - 如果没有请求，则不要推送任何avg_rt值

   --carbon-idle-avg last - 使用最后一次计算的avg_rt值（默认）

   --carbon-idle-avg zero - 如果没有请求，则推送0



内部路由的数值检查
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有了新的检查：

ishigher 或者 '>'

islower 或者 '<'

ishigherequal 或者 '>='

islowerequal 或者 '<='

例如：

.. code-block:: ini

   [uwsgi]
   route-if = ishigher:${CONTENT_LENGTH};1000 break:403 Forbidden


内部路由子系统中的math和time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你构建具有matheval（debian/ubuntu上是matheval-dev）支持的uWSGI，那么你将通过'math'路由变量，在你的路由系统中获得math支持。

目前，添加了'time'路由变量，它只导出'unix'字段，返回纪元。

看看这个疯狂的例子：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route-run = addvar:TEMPO=${time[unix]}
   route-run = log:inizio = ${TEMPO}
   route-run = addvar:TEMPO=${math[TEMPO+1]}
   route-run = log:tempo = ${TEMPO}


正如你所见，路由子系统可以存储值到请求变量中 (这里，我们创建了一个'TEMPO'变量，然后你将能够访问它，甚至是在你的应用请求变量中也可以访问它)

'math'运算可以引用请求变量

查看matheval文档，看看支持的运算：http://matheval.sourceforge.net/docs/index.htm

新增非标准的seek()和tell()到wsgi.input (要求post-buffering)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在测试'Klaus'项目 (https://github.com/jonashaag/klaus)的'smart模式'时，我们注意到当处于smart模式时，它违反了WSGI的标准调用seek()和tell()。

当post-buffering启用的时候，我们添加了对这两种方法的支持。

记住：它们违反了WSGI标准，因此，试着避免使用它们 (如果可以的话)。有更好的方式可以来完成相同的事。

Pyshell改进，亦称Welcome IPython (想法：C Anthony Risinger)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在使用--pyshell的时候，你可以调用ipython shell而不是默认的：

.. code-block:: sh

   uwsgi -s :3031 --pyshell="from IPython import embed; embed()"

显然，你可以传递任何代码给--pyshell

'rpcraw'路由指令
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

另一个强大但是极度危险的动作。它会调用一个rpc函数，直接发送其返回值给客户端  (不进行进一步的处理)。

空的返回值表示“进入下一条路由规则”。

返回值必须是有效的HTTP：

.. code-block:: js

   uwsgi.register_rpc('myrules', function(uri) {
        if (uri == '/foo') {
                return "HTTP/1.0 200 OK\r\nContent-Type: text/plain\r\nServer: uWSGI\r\nFoo: bar\r\n\r\nCiao Ciao";
        }
        return "";
   });

.. code-block:: ini

   [uwsgi]
   plugin = v8
   v8-load = rules.js
   route = ^/foo rpcraw:myrules ${REQUEST_URI}


HTTP Range头的初步支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

range请求头允许只请求资源的一部分 (例如，一个静态文件的有限字节集合)。

当提供静态文件服务的时候，可以使用该系统，但默认禁用它。仅需添加--honour-range来启用它。

未来，它将用于文件封装 (例如wsgi.file_wrapper) 和 :doc:`GridFS` (这就是默认不启用它的原因，因为你很有可能已经在你的应用中实现了range管理)


'lord'路由条件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们在努力地实现一个使用 :doc:`Legion` 的真正令人惊奇的集群子系统。

现在，当一个实例是lord的时候，你可以执行内部路由规则了：

.. code-block:: ini

   [uwsgi]
   ...
   route-if = lord:mylegion log:I AM THE LORD !!!

这个"I AM THE LORD !!!"日志行只有在实例是legion 'mylegion'的lord时才会打印

GridFS认证
^^^^^^^^^^^^^^^^^^^^^

现在，当使用 :doc:`GridFS` 的时候，你可以连接到已认证的MongoDB服务器了。

只需添加用户名和密码参数到挂载定义中

--for-times配置逻辑
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你可以使用--for-times来运行多次（指定数目）uWSGI选项：

.. code-block:: ini

   [uwsgi]
   for-times = 8
      mule = true
   endfor =

这将会生成8个mule

'uwsgi'路由变量
^^^^^^^^^^^^^^^^^^^^^^^

在定义路由规则的时候访问uWSGI内部参数将会很方便。'uwsgi'路由变量就是这样的变量的容器。

目前，它导出'wid' (运行该规则的worker的id) 和'pid' (运行该规则的worker的pid)

.. code-block:: ini

   [uwsgi]
   master = true
   processes = 4
   ; stupid rule... break connections to the worker 4
   route-if = ishigher:${uwsgi[wid]};3 break:403 Forbidden

'alarm'路由动作
^^^^^^^^^^^^^^^^^^^^^^^^^^

现在，你可以从路由子系统触发告警：

.. code-block:: ini

   [uwsgi]

   alarm = pippo cmd:cat

   route = ^/help alarm:pippo ${uwsgi[wid]} ${uwsgi[pid]}
   http-socket = :9090

当请求/help的时候，会触发'pippo'告警，传递wid和pid作为消息

欢迎来到ruby shell
^^^^^^^^^^^^^^^^^^^^^^^^^

和--pyshell一样，现在，我们有ruby shell了：

.. code-block:: sh

   uwsgi --rbshell -s :3031

或者

.. code-block:: sh

   uwsgi --rbshell="require 'pry';binding.pry" -s :3031

关于使用pry shell: http://pryrepl.org/

... 以及欢迎来到Lua shell
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

跟python和ruby一样，甚至Lua都有其shell了。仅需添加--lua-shell

再见了，老的（没用的）探测子系统
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

探测子系统是在0.9开发周期中添加的，但是它设计糟糕，基本上不能用了。

已经明确删除它了 (跳过了弃用阶段，因为1.9并不是一个LTS版本，而1.4仍然支持它)


Legion子系统的改进 (作者：Łukasz Mierzwa)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

添加了两个新的钩子：--legion-node-joined和--legion-node-left

更多的微调
^^^^^^^^^^^^^^^^

添加了--socket-sndbuf和--socket-rcvbuf，从而允许调整发送uWSGI socket的接收缓存 (小心使用)

V8改进和TeaJS集成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:doc:`V8` 插件正继续改进中。主要的目标仍然是 :doc:`InternalRouting` ，但是JSGI支持基本完成了，我们正在致力于TeaJS (老的v8cgi)集成：http://code.google.com/p/teajs/

更多内容很快就会实现……


可用性
************

uWSGI 1.9.6自20130409起可用，你可以在下面的url中找到它：

http://projects.unbit.it/downloads/uwsgi-1.9.6.tar.gz
