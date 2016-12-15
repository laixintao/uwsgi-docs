在uWSGI中运行PHP脚本
============================

You can safely run PHP scripts using 你可以通过uWSGI的 :doc:`CGI<CGI>` 支持安全运行PHP脚本。这个方法的缺点是每次请求生成新的PHP解释器引发的延迟。

要获得超级不错的性能，你会想要嵌入PHP解释器到uWSGI核心中，并使用PHP插件。

构建
--------

一堆发行版 (例如Fedora, Red Hat和CentOS) 包含了一个 ``php-embedded`` 包。安装它，以及 ``php-devel`` ，然后你应该能够构建php插件：

.. code-block:: sh

    python uwsgiconfig.py --plugin plugins/php
    # You can set the path of the php-config script with UWSGICONFIG_PHPPATH.
    UWSGICONFIG_PHPPATH=/opt/php53/bin/php-config python uwsgiconfig.py --plugin plugins/php
    # or directly specify the directory in which you have installed your php environment
    UWSGICONFIG_PHPDIR=/opt/php53 python uwsgiconfig.py --plugin plugins/php

如果有链接问题 (例如找不到库)，那么安装那些缺失的包 (``ncurses-devel``, ``gmp-devel``, ``pcre-devel``...)，但是要警告下你，如果你添加了修改uWSGI核心行为的开发包 (``pcre`` 就是其中一个)，那么你也 _需要_ 重新编译uWSGI服务器，否则会引发奇怪的问题。

对于那些不提供一个libphp包的发行版 (例如，所有基于Debian的发行版)，你必须在 ``./configure`` 中带上 ``--enable-embed`` 标志来重新构建PHP：

.. code-block:: sh

    ./configure --prefix=/usr/local --with-mysql --with-mysqli --with-pdo-mysql --with-gd --enable-mbstring --enable-embed
    # That's a good starting point

Ubuntu 10.04 (较新的版本包括官方嵌入libphp的sapi)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sh

    # Add ppa with libphp5-embed package
    sudo add-apt-repository ppa:l-mierzwa/lucid-php5
    # Update to use package from ppa
    sudo apt-get update
    # Install needed dependencies
    sudo apt-get install php5-dev libphp5-embed libonig-dev libqdbm-dev
    # Compile uWSGI PHP plugin
    python uwsgiconfig --plugin plugins/php

多个PHP版本
^^^^^^^^^^^^^^^^^^^^^

有时 (如果你是ISP，那么总是) 你或许在系统中安装了多个PHP版本。在这种情况下，你会需要对每个PHP版本使用一个uWSGI插件：

.. code-block:: sh

    UWSGICONFIG_PHPDIR=/opt/php51 python uwsgiconfig.py --plugin plugins/php default php51
    UWSGICONFIG_PHPDIR=/opt/php52 python uwsgiconfig.py --plugin plugins/php default php52
    UWSGICONFIG_PHPDIR=/opt/php53 python uwsgiconfig.py --plugin plugins/php default php53

'default'是你的服务器核心的构建配置文件。如果你不带一个指定的配置文件构建uWSGI，那么它将会是'default'。

然后，你可以使用 ``plugins php51`` 加载一个指定的插件，等等。你不能在同一个uWSGI进程内加载多个PHP版本。

用nginx运行PHP应用
---------------------------

如果你有简单的应用 (基于文件扩展名)，那么你可以使用像这样的东东：

.. code-block:: nginx

    location ~ \.php$ {
        root /your_document_root;
        include uwsgi_params;
        uwsgi_modifier1 14;
        uwsgi_pass 127.0.0.1:3030;
    }
    
或许你想要检查所有包含字符串 ``.php`` 的URI：

.. code-block:: nginx

    location ~ \.php {
        root /your_document_root;
        include uwsgi_params;
        uwsgi_modifier1 14;
        uwsgi_pass 127.0.0.1:3030;
    }
    
现在，只需运行带一堆进程的uWSGI服务器：

.. code-block:: sh

    uwsgi -s :3030 --plugin php -M -p 4
    # Or abuse the adaptive process spawning with the --cheaper option
    uwsgi -s :3030 --plugin php -M -p 40 --cheaper 4

这将允许多达40个并发php请求，但只会在需要的时候试着生成（或摧毁）worker，维持一个包含4个进程的最小池。

高级配置
----------------------

默认情况下，PHP插件将会愉悦地执行任何你传给它的脚本。你或许想要用 ``php-allowed-ext`` 选项限制到一个扩展名子集。

.. code-block:: sh

    uwsgi --plugin php --master --socket :3030 --processes 4 --php-allowed-ext .php --php-allowed-ext .inc

无前端服务器运行PHP应用
--------------------------------------

这是一个样例配置，有一个“公用的”uWSGI实例，它运行一个PHP应用，并提供静态文件。对于例子而言，它有点复杂，但对于棘手配置而言，应该是一个不错的开始点。

.. code-block:: ini

    [uwsgi]
    ; load the required plugins, php is loaded as the default (0) modifier
    plugins = http,0:php
    
    ; bind the http router to port 80
    http = :80
    ; leave the master running as root (to allows bind on port 80)
    master = true
    master-as-root = true
    
    ; drop privileges
    uid = serena
    gid = serena
    
    ; our working dir
    project_dir = /var/www
    
    ; chdir to it (just for fun)
    chdir = %(project_dir)
    ; check for static files in it
    check-static = %(project_dir)
    ; ...but skip .php and .inc extensions
    static-skip-ext = .php
    static-skip-ext = .inc
    ; search for index.html when a dir is requested
    static-index = index.html
    
    ; jail our php environment to project_dir
    php-docroot = %(project_dir)
    ; ... and to the .php and .inc extensions
    php-allowed-ext = .php
    php-allowed-ext = .inc
    ; and search for index.php and index.inc if required
    php-index = index.php
    php-index = index.inc
    ; set php timezone
    php-set = date.timezone=Europe/Rome
    
    ; disable uWSGI request logging
    disable-logging = true
    ; use a max of 17 processes
    processes = 17
    ; ...but start with only 2 and spawn the others on demand
    cheaper = 2
    
一个更极端的例子，混合了 :doc:`CGI <CGI>` 和PHP，使用 :doc:`internal routing <InternalRouting>` 和一点 :doc:`configuration logic <ConfigLogic>` 。

.. code-block:: ini

    [uwsgi]
    ; load plugins
    plugins-dir = /proc/unbit/uwsgi
    plugins = cgi,php,router_uwsgi
    
    ; set the docroot as a config placeholder
    docroot = /accounts/unbit/www/unbit.it
    
    ; reload whenever this config file changes
    ; %p is the full path of the current config file
    touch-reload = %p
    
    ; set process names to something meaningful
    auto-procname = true
    procname-prefix-spaced = [unbit.it]
    
    ; run with at least 2 processes but increase up to 8 when needed
    master = true
    processes = 8
    cheaper = 2
    
    ; check for static files in the docroot
    check-static = %(docroot)
    ; check for cgi scripts in the docroot
    cgi = %(docroot)
    
    logto = /proc/unbit/unbit.log
    ;rotate logs when filesize is higher than 20 megs
    log-maxsize = 20971520
    
    ; a funny cycle using 1.1 config file logic
    for = .pl .py .cgi
      static-skip-ext = %(_)
      static-index = index%(_)
      cgi-allowed-ext = %(_)
    endfor =
    
    ; map cgi modifier and helpers
    ; with this trick we do not need to give specific permissions to cgi scripts
    cgi-helper = .pl=perl
    route = \.pl$ uwsgi:,9,0
    cgi-helper = .cgi=perl
    route = \.cgi$ uwsgi:,9,0
    cgi-helper = .py=python
    route = \.py$ uwsgi:,9,0
    
    ; map php modifier as the default
    route = .* uwsgi:,14,0
    static-skip-ext = .php
    php-allowed-ext = .php
    php-allowed-ext = .inc
    php-index = index.php
    
    ; show config tree on startup, just to see
    ; how cool is 1.1 config logic
    show-config = true

uWSGI API支持
-----------------

对一些uWSGI API的初期支持已经在1.1版本添加了。这是支持函数的列表：

* uwsgi_version()
* uwsgi_setprocname($name)
* uwsgi_worker_id()
* uwsgi_masterpid()
* uwsgi_signal($signum)
* uwsgi_rpc($node, $func, ...)
* uwsgi_cache_get($key)
* uwsgi_cache_set($key, $value)
* uwsgi_cache_update($key, $value)
* uwsgi_cache_del($key)

是哒，这意味着你可以使用RPC，从PHP调用Python函数。

.. code-block:: py
    
    from uwsgidecorators import *
    
    # define a python function exported via uwsgi rpc api
    @rpc('hello')
    def hello(arg1, arg2, arg3):
        return "%s-%s-%s" (arg3, arg2, arg1)

.. code-block:: php

    Python says the value is <? echo uwsgi_rpc("", "hello", "foo", "bar", "test"); ?>

设置 ``uwsgi_rpc`` 的第一个参数为空，将会触发本地rpc。

或者你可以共享uWSGI :doc:`cache <Caching>`...

.. code-block:: py
    
    uwsgi.cache_set("foo", "bar")

.. code-block:: php

    <? echo uwsgi_cache_get("foo"); ?>
    
    
uWSGI缓存之上的会话 (uWSGI >=2.0.4)
------------------------------------------

从uWSGI 2.0.4起，你可以将PHP会话存储在uWSGI缓存中。

.. code-block:: ini

   [uwsgi]
   plugins = php
   http-socket = :9090
   http-socket-modifier1 = 14
   ; create a cache with 1000 items named 'mysessions'
   cache2 = name=mysessions,items=1000
   ; set the 'uwsgi' session handler
   php-set = session.save_handler=uwsgi
   ; use the 'mysessions' cache for storing sessions
   php-set = session.save_path=mysessions
   
   ; or to store sessions in remote caches...
   ; use the 'foobar@192.168.173.22:3030' cache for storing sessions
   php-set = session.save_path=foobar@192.168.173.22:3030

Zend Opcode Cache (uWSGI >= 2.0.6)
----------------------------------

由于某些神秘的原因，在嵌入SAPI中，Opcode Cache是禁用的。

你可以通过告诉PHP引擎运行在apache SAPI之下(使用 ``php-sapi-name`` 选项)来绕过这个问题：

.. code-block:: ini

   [uwsgi]
   plugins = php
   php-sapi-name = apache
   http-socket = :9090
   http-socket-modifier1 = 14

ForkServer (uWSGI >= 2.1)
-------------------------

:doc:`ForkServer` 是2.1分支的主要特性之一。它允许你从指定的父亲那里继承你的vassal，而不是Emperor。

PHP插件已被扩展，来支持fork服务器，所以你可以拥有一个php基本实例池，其中，vassal可以 `fork()` 。这意味着，你可以共享opcode cache以及做其他花样。

多亏了uWSGI 2.1中的vassal属性，我们可以选择一个vassal将从哪个父亲中调用fork()。

.. note::

    你需要Linux内核 >= 3.4 (这个特性要求 ``PR_SET_CHILD_SUBREAPER``) 以获得“稳定”使用。否则，你的Emperor将不能够正确wait()孩子(children) (这将会减缓你的vassal的重新生成，并且会导致各种形式的竞争条件)。

在下面的例子中，我们将会生成3个vassal，一个 (称为base.ini) 将会初始化一个PHP引擎，而其他两个将会从第一个 `fork()` 。

.. code-block:: ini

   [uwsgi]
   ; base.ini
   
   ; force the sapi name to 'apache', this will enable the opcode cache
   early-php-sapi-name = apache
   ; load a php engine as soon as possible
   early-php = true
   
   ; ... and wait for fork() requests on /run/php_fork.socket
   fork-server = /run/php_fork.socket
   
然后2个vassal

.. code-block:: ini

   [emperor]
   ; tell the emperor the address of the fork server
   fork-server = /run/php_fork.socket

   [uwsgi]
   ; bind to port :4001
   socket = 127.0.0.1:4001
   ; force all requests to be mapped to php
   socket-modifier1 = 14
   ; enforce a DOCUMENT_ROOT
   php-docroot = /var/www/one
   ; drop privileges
   uid = one
   gid = one


   
.. code-block:: ini

   [emperor]
   ; tell the emperor the address of the fork server
   fork-server = /run/php_fork.socket

   [uwsgi]
   ; bind to port :4002
   socket = 127.0.0.1:4002
   ; force all requests to be mapped to php
   socket-modifier1 = 14
   ; enforce a DOCUMENT_ROOT
   php-docroot = /var/www/two
   ; drop privileges
   uid = two
   gid = two
   
这两个vassal是完全无关的 (即使它们是从同一个父亲那里fork过来的)，所以你可以移除特权，使用不同的进程策略，等等。

现在生成Emperor：
 
 .. code-block:: sh
 
    uwsgi --emperor phpvassals/ --emperor-collect-attr fork-server --emperor-fork-server-attr fork-server
    
 ``--emperor-collect-attr`` 迫使Emperor在vassal文件的[emperor]部分搜索'fork-server'属性，而 ``--emperor-fork-server-attr`` 告诉它使用这个参数作为fork服务器的地址。

显然，如果一个vassal不公开这么一个属性，那么它将会正常地从Emperor fork()。
