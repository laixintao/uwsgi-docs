uWSGI内部路由
======================

``更新至1.9``

自uWSGI 1.9起，提供了一个可编程的内部路由子系统。 (1.1版本之后的旧版本有一个不大起眼的版本)。你可以使用内部路由子系统来动态改变处理请求的方式。例如，你可以用它来触发对于指定URL的301重定向，或者在指定条件下提供来自于缓存的内容。这个内部路由子系统的灵感来源于Apache的 ``mod_rewrite`` 和Linux的 ``iptables`` 命令。
在喷它是凌乱、不优雅或者不图灵完备之前，请记住，它必须快并且只是快。如果你需要优雅以及更多的复杂度，那么你可以在自己的代码中实现。

路由链
******************

在请求周期内，会经过不同的“链”。每个链包含一个路由表 (见下)。

链可以是“递归的”，一个“递归”链在一个请求周期内可以被多次调用。

这是链的顺序：

``request`` 在请求被传递给插件之前应用

``error`` 一个生成了一个HTTP状态码就会被立即应用（递归链）

``response`` 在生成了最后一个响应头后被应用 (仅在发送响应体之前)

``final`` 在已经发送响应到客户端之后被应用

 ``request`` 链是 (为了方便起见) 是“默认的”链，因此它的选项没有前缀，而其他的链需要前缀。

例如：

``route-user-agent`` -> 用于request链

而

``response-route-uri`` -> 用于response链

内部路由表
**************************

内部路由表是一个接一个（也允许向前跳跃）执行的“规则”序列。每个规则由一个''subject（标题）''，一个''condition（条件）''和一个''action（动作）''组成。''条件''通常是一个应用到标题的PCRE正则表达式：如果匹配上了，那么就会触发相应的动作。标题是请求的变量。目前支持以下标题：

* ``host`` (检查HTTP_HOST)
* ``uri`` (检查REQUEST_URI)
* ``qs`` (检查QUERY_STRING)
* ``remote-addr`` (检查REMOTE_ADDR)
* ``remote-user`` (检查REMOTE_USER)
* ``referer`` (检查HTTP_REFERER)
* ``user-agent`` (检查HTTP_USER_AGENT)
* ``status`` (检查HTTP响应状态码，在request链中不可用)
* ``default`` (默认标题，映射到PATH_INFO)

除此之外，有一个可用的低层次条件可插拔系统。你可以使用 ``--route-if`` 选项来访问此系统。目前，支持以下检查：

* ``exists`` (检查标题是否存在于文件系统)
* ``isfile`` (检查标题是否是一个文件)
* ``isdir`` (检查标题是否是一个目录)
* ``isexec`` (检查标题是否是一个可执行文件)
* ``equal``/``isequal``/``eq``/``==`` (检查标题是否等于指定模式)
* ``ishigherequal``/``>=``
* ``ishigher``/``>``
* ``islower``/``<``
* ``islowerequal``/``<=``
* ``startswith`` (检查标题是否以指定模式开头)
* ``endswith`` (检查标题是否以指定模式结尾)
* ``regexp``/re (检查标题是否匹配指定的正则表达式)
* ``empty`` (检查标题是否为空)
* ``contains``

当一次检查需要一个模式(例如使用'equal'或者'regexp')时，你可以使用分号将它从标题分隔开：

.. code-block:: ini

   ; never matches
   route-if = equal:FOO;BAR log:never here
   ; matches
   route-if = regexp:FOO;^F log:starts with F


动作是当一个规则匹配的时候要运行的汗水。这些动作由插件导出，并有一个返回值。

动作返回值
********************

每个动作都有一个返回值，这个返回值告诉路由引擎接下来要做什么。支持以下返回码：

* ``NEXT`` (继续下一个规则)
* ``CONTINUE`` (停止扫描内部路由表并运行请求)
* ``BREAK`` (停止扫描内部路由表并关闭请求)
* ``GOTO x`` (跳到规则 ``x``)

当一个规则不匹配的时候，则假设 ``NEXT`` 。

第一个例子
*****************

.. code-block:: ini

   [uwsgi]
   route-user-agent = .*curl.* redirect:http://uwsgi.it
   route-remote-addr = ^127\.0\.0\.1$ break:403 Forbidden
   route = ^/test log:someone called /test
   route = \.php$ rewrite:/index.php
   route = .* addheader:Server: my uWSGI server
   route-host = ^localhost$ logvar:local=1
   route-uri = ^/foo/(.*)\.jpg$ cache:key=$1.jpg
   route-if = equal:${PATH_INFO};/bad break:500 Internal Server Error

前面的规则构建以下表：

* 如果 ``HTTP_USER_AGENT`` 变量包含'curl'，那么重定向请求到
  http://uwsgi.it (状态码为302，动作返回BREAK)
* 如果 ``REMOTE_ADDR`` 是'127.0.0.1'，则返回403 Forbidden (动作返回
  BREAK)
* 如果 ``PATH_INFO`` 以/test开头，则在日志中打印字符串'someone called /test' (动作返回NEXT)
* 如果 ``PATH_INFO`` 以'.php'结尾，那么将其重写至/index.php (动作返回
  NEXT)
* 对于所有的 ``PATH_INFO`` ，添加HTTP头'Server: my uWSGI server'到响应中 (动作返回NEXT)
* 如果 ``HTTP_HOST`` 是localhost，那么添加日志变量'local'，并将其设置为'1'
* 如果 ``REQUEST_URI`` 以/foo开头，并且以.jpg结尾，那么使用附带的键（通过正则表达式分组构建）从uWSGI缓存中获取它 (动作返回
  BREAK)
* 如果 ``PATH_INFO`` 等于/bad，那么抛出500错误

访问请求变量
**********************

除了PCRE占位符/组 (使用$1到$9)，你可以使用${VAR}语法来访问请求变量 (PATH_INFO, SCRIPT_NAME, REQUEST_METHOD...)。

.. code-block:: ini

   [uwsgi]
   route-user-agent = .*curl.* redirect:http://uwsgi.it${REQUEST_URI}

访问cookie
*****************

你可以使用${cookie[name]}语法来访问一个cookie值：

.. code-block:: ini

   [uwsgi]
   route = ^/foo log:${cookie[foobar]}

这将会在日志中记录当前请求的'foobar' cookie内容

访问查询字符串
****************************

你可以使用${qs[name]}语法访问HTTP请求字符串的值：

.. code-block:: ini

   [uwsgi]
   route = ^/foo log:${qs[foobar]}

这将会在日志中记录当前请求的查询字符串的'foobar'项的内容

可插拔路由变量
***************************

cookie和qs变量都是所谓的“路由变量”。它们是可插拔的，因此外部插件可以添加新的变量来添加新特性到你的应用上。
(看看 :doc:`GeoIP` 插件，以获取关于这个的一个例子。) 还有一些可用的嵌入式路由变量。

* ``mime`` -- 返回指定变量的mime类型：${mime[REQUEST_URI]}
  
  .. code-block:: ini
  
     [uwsgi]
     route = ^/images/(.+) addvar:MYFILE=$1.jpg
     route = ^/images/ addheader:Content-Type: ${mime[MYFILE]}

* ``time`` -- 以多种形式返回时间/日期。唯一支持（目前）的是time[unix]，返回时代
* ``httptime`` -- 返回http日期，添加数值参数 (如果指定) 来获得当前时间 (使用空参数来获取当前的服务器时间)

.. code-block:: ini
  
     [uwsgi]
     ; add Date header
     route-run = addheader:Date ${httptime[]}


* ``math`` -- 需要matheval支持。例如：math[CONTENT_LENGTH+1]
* ``base64`` -- 将指定的变量用base64编码
* ``hex`` -- 将指定的变量用hex编码
* ``upper`` -- 将指定变量转换成大写
* ``lower`` -- 将指定变量转换成小写
* ``uwsgi`` -- 返回内部的uWSGI信息，目前支持uwsgi[wid], uwsgi[pid], uwsgi[uuid]和uwsgi[status]

--route-if不够吗？为什么还要--route-uri和其他小伙伴？
******************************************************

这是个好问题。你只需要一直记住，uWSGI是关于通用性和 *性能* 的。可以循环利用总是好的。
``--route-if`` 选项，虽然通用，但是不能够被优化，因为在处理每一个请求的时候都必须重新计算它所有的部分。这显然非常快，但是
``--route-uri`` 选项 (和其他小伙伴)可以被预先优化 (在启动期间) 来直接映射到请求内存区域，因此，如果你可以使用它们，那么你绝对应该用它们。 :)

GOTO
****

是哒，整个信息技术产业（和历史）最具争议的结构就是这里。你可以向前 (只能向前！)跳到内部路由表的特定点。你可以设置标签来标记路由表的特定点，或者如果你勇敢 (或者蠢)，直接跳到一个规则变化。规则编号会在服务器启动的时候打印处理，但请用标签。

.. code-block:: ini

   [uwsgi]

   route-host = ^localhost$ goto:localhost
   route-host = ^sid\.local$ goto:sid.local
   route = .* last:
  
   route-label = sid.local
   route-user-agent = .*curl.* redirect:http://uwsgi.it
   route-remote-addr = ^192\.168\..* break:403 Forbidden
   route = ^/test log:someone called /test
   route = \.php$ rewrite:/index.php
   route = .* addheader:Server: my sid.local server
   route = .* logvar:local=0
   route-uri = ^/foo/(.*)\.jpg$ cache:key=$1.jpg
   route = .* last:

   route-label = localhost
   route-user-agent = .*curl.* redirect:http://uwsgi.it
   route-remote-addr = ^127\.0\.0\.1$ break:403 Forbidden
   route = ^/test log:someone called /test
   route = \.php$ rewrite:/index.php
   route = .* addheader:Server: my uWSGI server
   route = .* logvar:local=1
   route-uri = ^/foo/(.*)\.jpg$ cache:key=$1.jpg
   route = .* last:

这个例子像前面那个，但域之间存在些不同。看看"last:"的时候，来终端路由表扫描。你可以将前2个规则写成一个：

.. code-block:: ini

   [uwsgi]

   route-host = (.*) goto:$1
   
收集响应头
***************************

正如我们已经看到的，每个uWSGI请求都有一组相关的变量。它们一般是由web服务器传递的CGI变量，但是你也可以用其他变量来扩展它们 (看看'addvar'动作)。

uWSGI 1.9.16添加了一个新特性，允许你存储一个响应头的内容到一个请求变量中。这让编写更高级的规则变得更简单。

例如，你可能想要gzip所有的text/html响应：

.. code-block:: ini

   [uwsgi]
   ; store Content-Type response header in MY_CONTENT_TYPE var
   collect-header = Content-Type MY_CONTENT_TYPE
   ; if response is text/html, and client supports it, gzip it
   response-route-if = equal:${MY_CONTENT_TYPE};text/html goto:gzipme
   response-route-run = last:
   
   response-route-label = gzipme
   ; gzip only if the client support it
   response-route-if = contains:${HTTP_ACCEPT_ENCODING};gzip gzip:
   
可用动作
*********************

``continue``/``last``
^^^^^^^^^^^^^^^^^^^^^

返回值： ``CONTINUE``

停止扫描内部路由表，并继续到选定的请求处理器。

``break``
^^^^^^^^^

返回值： ``BREAK``

停止扫描内部路由表，并关闭该请求。可以选择返回指定的HTTP状态码：

.. code-block:: ini

   [uwsgi]
   route = ^/notfound break:404 Not Found
   route = ^/bad break:
   route = ^/error break:500

注意： ``break`` 并不支持请求变量，因为它旨在通知浏览器有关错误，而不是通知终端用户。也就是说，我们可以判定以下代码将会发送它读取的内容给浏览器 (即 ``${REMOTE_ADDR}`` 并不会被转换成远程IP地址)。

.. code-block:: ini

   [uwsgi]
   route-remote-addr = ^127\.0\.0\.1$ break:403 Forbidden for ip ${REMOTE_ADDR}

如果你真的想要做些古古怪怪的东西，那么可以看看 ``clearheaders`` 。

``return``/``break-with-status``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

返回值： ``BREAK``

``return`` 使用uWSGI内置的状态码并同时返回状态码和消息体。它类似于 ``break`` ，但正如上面提到的， ``break``
并没有错误消息体。 ``return:403`` 等价于：

.. code-block:: ini

   [uwsgi]
    route-run = clearheaders:403 Forbidden
    route-run = addheader:Content-Type: text/plain
    route-run = addheader:Content-Length: 9
    route-run = send:Forbidden
    route-run = break:

``log``
^^^^^^^

返回值： ``NEXT``

在日志中打印指定消息。

.. code-block:: ini

   [uwsgi]
   route = ^/logme/(.) log:hey i am printing $1

``logvar``
^^^^^^^^^^

返回值： ``NEXT``

添加指定的日志变量。

.. code-block:: ini

   [uwsgi]
   route = ^/logme/(.) logvar:item=$1

``goto``
^^^^^^^^

返回值： ``NEXT``

向前跳到指定的标签或者规则位置。

``addvar``
^^^^^^^^^^

返回值： ``NEXT``

添加指定的CGI (环境) 变量到请求中。

.. code-block:: ini

   [uwsgi]
   route = ^/foo/(.) addvar:FOOVAR=prefix$1suffix

``addheader``
^^^^^^^^^^^^^

返回值： ``NEXT``

添加指定的HTTP头到响应中。

.. code-block:: ini

   [uwsgi]
   route = ^/foo/(.) addheader:Foo: Bar

``delheader``//``remheader``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

返回值： ``NEXT``

把指定的HTTP头从响应中移除。


.. code-block:: ini

   [uwsgi]
   route = ^/foo/(.) delheader:Foo

``signal``
^^^^^^^^^^

返回值： ``NEXT``

引发指定的uwsgi信号。

``send``
^^^^^^^^

返回值： ``NEXT``

非常高级 (且危险) 的函数，允许你添加原始数据到响应中。

.. code-block:: ini

   [uwsgi]
   route = ^/foo/(.) send:destroy the world

``send-crnl``
^^^^^^^^^^^^^

返回值： ``NEXT``

非常高级 (且危险) 的函数，允许你添加原始数据到响应中，以\r\n作为后缀。

.. code-block:: ini

   [uwsgi]
   route = ^/foo/(.) send-crnl:HTTP/1.0 100 Continue

``redirect``/``redirect-302``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

返回值： ``BREAK``

插件： ``router_redirect``

返回一个HTTP 302 Redirect给指定的URL。

``redirect-permanent``/``redirect-301``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

返回值： ``BREAK``

插件： ``router_redirect``

返回一个HTTP 3301 Permanent Redirect给指定的URL。

``rewrite``
^^^^^^^^^^^

返回值： ``NEXT``

插件： ``router_rewrite``

一个重写引擎，灵感来自于Apache mod_rewrite。在请求被派发给请求处理函数之前，根据指定的规则重建PATH_INFO和
QUERY_STRING。

.. code-block:: ini

   [uwsgi]
   route-uri = ^/foo/(.*) rewrite:/index.php?page=$1.php

``rewrite-last``
^^^^^^^^^^^^^^^^

rewrite的别名，但是返回值是 ``CONTINUE``, 直接传递请求到请求处理器next。

``uwsgi``
^^^^^^^^^

返回值： ``BREAK``

插件： ``router_uwsgi``

重写一个请求的modifier1, modifier2和可选的 ``UWSGI_APPID`` 值，或者路由请求到一个外部的uwsgi服务器。

.. code-block:: ini

   [uwsgi]
   route = ^/psgi uwsgi:127.0.0.1:3031,5,0

这个配置路由所有以 ``/psgi`` 开始的请求到在127.0.0.1:3031上运行的uwsgi服务器，设置modifier1为5，设置modifier2为0.如果你只想修改modifier1/modifier2，而不想将请求路由到一个外部服务器，那么使用以下语法。

.. code-block:: ini

   [uwsgi]
   route = ^/psgi uwsgi:,5,0

要设置一个指定的 ``UWSGI_APPID`` 值，在其后附加。

.. code-block:: ini

   [uwsgi]
   route = ^/psgi uwsgi:127.0.0.1:3031,5,0,fooapp

子请求是异步型的 (支持诸如gevent或者ugreen这样的引擎)，如果offload线程可用，那么将会使用它们。

``http``
^^^^^^^^

返回值： ``BREAK``

插件： ``router_http``

将请求路由到外部HTTP服务器。

.. code-block:: ini

   [uwsgi]
   route = ^/zope http:127.0.0.1:8181

你可以使用以下语法来替换一个替代的Host头：

.. code-block:: ini

   [uwsgi]
   route = ^/zope http:127.0.0.1:8181,myzope.uwsgi.it

``static``
^^^^^^^^^^

返回值： ``BREAK``

插件： ``router_static``

提供来自指定物理路径的静态文件。

.. code-block:: ini

   [uwsgi]
   route = ^/logo static:/var/www/logo.png

``basicauth``
^^^^^^^^^^^^^

返回值： ``NEXT`` or ``BREAK 401`` on failed authentication

插件： ``router_basicauth``

支持四种语法。

* ``basicauth:realm,user:password`` – 一个简单的用户名:密码映射
* ``basicauth:realm,user:`` – 只鉴权用户名
* ``basicauth:realm,htpasswd`` – 使用一个类htpasswd文件。支持所有的POSIX
  crypt()算法。这与Apache传统的htpasswd文件 _不_ 同，因此，使用htpasswd工具的 ``-d`` 标志来创建兼容文件。
* ``basicauth:realm,`` – 用来立即引起一个HTTP 401响应。
  由于路由是从上至下解析的，因此，你也许想要引发这个来避免绕过规则。

例如：

.. code-block:: ini

   [uwsgi]
   route = ^/foo basicauth-next:My Realm,foo:bar
   route = ^/foo basicauth:My Realm,foo2:bar2
   route = ^/bar basicauth:Another Realm,kratos:

例如： 对Trac使用basicauth

.. code-block:: ini

   [uwsgi]
   ; load plugins (if required)
   plugins = python,router_basicauth

   ; bind to port 9090 using http protocol
   http-socket = :9090

   ; set trac instance path
   env = TRAC_ENV=myinstance
   ; load trac
   module = trac.web.main:dispatch_request

   ; trigger authentication on /login
   route = ^/login basicauth-next:Trac Realm,pippo:pluto
   route = ^/login basicauth:Trac Realm,foo:bar

   ;high performance file serving
   static-map = /chrome/common=/usr/local/lib/python2.7/dist-packages/trac/htdocs

``basicauth-next``
^^^^^^^^^^^^^^^^^^

与 ``basicauth`` 相同，但在鉴权失败的时候返回 ``NEXT`` 。

``ldapauth``
^^^^^^^^^^^^

返回值：鉴权失败的时候返回 ``NEXT`` 或者 ``BREAK 401`` 

插件： ``ldap``

这个鉴权路由器是LDAP插件的一部分，因此为了让这个能用，必须加载它。它就像basicauth路由器，但是使用一个LDAP服务器来进行鉴权，语法： ``ldapauth:realm,options`` 

可用选项：

* ``url`` - LDAP服务器URI (必填)
* ``binddn`` - 用于绑定的DN。如果LDAP服务器不允许匿名搜索，那么该选项必填。
* ``bindpw`` -  ``binddn`` 用户的密码
* ``basedn`` - 搜索用户时使用的基本DN (必填)
* ``filter`` - 搜索用户时使用的过滤器 (默认是
  "(objectClass=*)")
* ``attr`` - LDAP属性，保存用户登录 (默认是"uid")
* ``loglevel`` - 0 - 不记录任何绑定，1 - 记录鉴权错误，2 -
  同时记录成功和失败的绑定

例如：

.. code-block:: ini

   route = ^/protected ldapauth:LDAP auth realm,url=ldap://ldap.domain.com;basedn=ou=users,dc=domain;binddn=uid=proxy,ou=users,dc=domain;bindpw=password;loglevel=1;filter=(objectClass=posixAccount)

``ldapauth-next``
^^^^^^^^^^^^^^^^^

与ldapauth相同，但是鉴权失败的时候返回 ``NEXT`` 。

``cache``
^^^^^^^^^

返回值： ``BREAK``

插件： ``router_cache``

``cachestore``/``cache-store``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``cachevar``
^^^^^^^^^^^^

``cacheset``
^^^^^^^^^^^^

``memcached``
^^^^^^^^^^^^^

``rpc``
^^^^^^^

"rpc"路由指令让你可以直接调用来自路由子系统的uWSGI RPC函数，并且将它们的输出转发到客户端。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route = ^/foo addheader:Content-Type: text/html
   route = ^/foo rpc:hello ${REQUEST_URI} ${HTTP_USER_AGENT}
   route = ^/bar/(.+)$ rpc:test $1 ${REMOTE_ADDR} uWSGI %V
   route = ^/pippo/(.+)$ rpc:test@127.0.0.1:4141 $1 ${REMOTE_ADDR} uWSGI %V
   import = funcs.py



``call``
^^^^^^^^^

插件： ``rpc``

``rpcret``
^^^^^^^^^

插件： ``rpc``

`rpcret` 调用指定的rpc函数，并将其返回值当成动作返回码 (next, continue, goto, 等等)


``rpcblob``//``rpcnext``
^^^^^^^^^^^^^^^^^^^^^^^^

插件： ``rpc``

`rpcnext/rpcblob` 调用指定的RPC函数，发送响应到客户端，并继续下一条规则。


``rpcraw``
^^^^^^^^^

插件： ``rpc``


``rpcvar``
^^^^^^^^^

插件： ``rpc``

调用指定的rpc函数，并将其返回值赋给指定的CGI环境变量。


``access``
^^^^^^^^^^

``spnego``
^^^^^^^^^^

开发中……

``radius``
^^^^^^^^^^

开发中……

``xslt``
^^^^^^^^

.. seealso:: :doc:`XSLT`

ssi
^^^

.. seealso:: :doc:`SSI`

gridfs
^^^^^^

.. seealso:: :doc:`GridFS`

``donotlog``
^^^^^^^^^


``chdir``
^^^^^^^^^


``seturi``
^^^^^^^^^

更新 `REQUEST_URI`


``setapp``
^^^^^^^^^


``setuser``
^^^^^^^^^


``sethome``
^^^^^^^^^


``setfile``
^^^^^^^^^


``setscriptname``
^^^^^^^^^


``setprocname``
^^^^^^^^^


``alarm``
^^^^^^^^^


``flush``
^^^^^^^^^


``fixcl``
^^^^^^^^^


``cgi``
^^^^^^^^^

插件： ``cgi``


``cgihelper``
^^^^^^^^^

插件： ``cgi``


``access``
^^^^^^^^^

插件： ``router_access``


``cache-continue``
^^^^^^^^^

插件： ``router_cache``


``cachevar``
^^^^^^^^^

插件： ``router_cache``


``cacheinc``
^^^^^^^^^

插件： ``router_cache``


``cachedec``
^^^^^^^^^

插件： ``router_cache``


``cachemul``
^^^^^^^^^

插件： ``router_cache``


``cachediv``
^^^^^^^^^

插件： ``router_cache``


``proxyhttp``
^^^^^^^^^

插件： ``router_http``


``memcached``
^^^^^^^^^

插件： ``router_memcached``


``memcached-continue``
^^^^^^^^^

插件： ``router_memcached``

``memcachedstore``
^^^^^^^^^

插件： ``router_memcached``

``memcached-store``
^^^^^^^^^

插件： ``router_memcached``



``redis``
^^^^^^^^^

插件： ``router_redis``


``redis-continue``
^^^^^^^^^

插件： ``router_redis``

``redisstore``
^^^^^^^^^

插件： ``router_redis``

``redis-store``
^^^^^^^^^

插件： ``router_redis``


``proxyuwsgi``
^^^^^^^^^^^^^^

插件： ``router_uwsgi``

``harakiri``
^^^^^^^^^^^^

为当前请求设置harakiri。

``file``
^^^^^^^^

在 *不* 使用加速（sendfile, offloading等等）的情况下直接转移指定的文件名。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route-run = file:filename=/var/www/${PATH_INFO}
   
``clearheaders``
^^^^^^^^^^^^^^^^

清除响应头，设置一个新的HTTP状态码，可用于重置响应

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   response-route = ^/foo goto:foobar
   response-route-run = last:
   
   response-route-label = foobar
   response-route-run = clearheaders:404 Not Found
   response-route-run = addheader:Content-Type: text/html
   
``resetheaders``
^^^^^^^^^^^^^^^^

clearheaders的别名

``xattr``
^^^^^^^^^

插件： ``xattr``

xattr插件允许你在内部路由子系统中引用文件扩展属性：

.. code-block:: ini

   [uwsgi]
   ...
   route-run = addvar:MYATTR=user.uwsgi.foo.bar
   route-run = log:The attribute is ${xattr[/tmp/foo:MYATTR]}


或者 (带2个变量的变体)

.. code-block:: ini

   [uwsgi]
   ...
   route-run = addvar:MYFILE=/tmp/foo
   route-run = addvar:MYATTR=user.uwsgi.foo.bar
   route-run = log:The attribute is ${xattr2[MYFILE:MYATTR]}

仅在Linux平台上可用。
