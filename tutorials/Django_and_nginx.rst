使用uWSGI和nginx来设置Django和你的web服务器
==========================================================

.. highlight:: bash

本教程针对那些想要设置一个生产web服务器的Django用户。它介绍了设置Django以使得其与uWSGI和nginx工作良好的必要步骤。它涵盖了所有三个组成部分，提供了一个web应用和服务器软件的完整栈。

Django_ 是一个高层次的Python Web框架，鼓励快速开发和干净实用的设计。

.. _Django: http://djangoproject.com/

nginx_ (发音为 *engine-x*) 是一个免费开源并且高性能的HTTP服务器和反向代理，还是一个IMAP/POP3代理服务器。

.. _nginx: http://nginx.org/


本教程的一些注释
------------------------------

.. admonition:: 注意

    这是一个 **教程** 。它并不打算提供一个参考指南，对于部署主题，不要想着有一个详尽的参考。

对于Django部署而言，nginx和uWSGI是不错的选择，但它们并非唯一的选择，也不是“官方”选择。对于它们两个，都有不错的替代品，因此鼓励你去详细研究一下。

我们这里部署Django的方式是种不错的方式，但它 **不是** *唯一* 的方式；
对于某些目的，它甚至也许不是最好的方式。

然而，它是一种可靠而简单的方式，而这里所涉及的材料将会向你介绍无论你用什么软件来部署Django都会熟悉的概念和过程。通过为你提供一个可用的步骤，以及向你预演完成此目标的必须步骤，它将会为你提供探索其他做到这点的方法的基础。

.. admonition:: 注意

	本教程对你所使用的系统做了一些假设。

假设你正使用类Unix系统，并且它有一个类似于包管理器的功能。然而，如果你需要问类似于“那么在Mac OS X上的等价物是啥呢”这样的问题，那么你将能够非常简单地找到帮助。

虽然这个教程假设你使用Django 1.4或更高的版本，这将会自动在你的新工程里创建一个wsgi模块，但是，这些指示对低于Django 1.4的版本同样适用。虽然，你需要自己获得Django wsgi模块，并且你可能会发现Django工厂目录结构有点不一样。


概念
-------

一个web服务器面对的是外部世界。它能直接从文件系统提供文件 (HTML, 图像， CSS等等)。然而，它无法 *直接*与Django应用通信；它需要借助一些工具的帮助，这些东西会运行运用，接收来自web客户端（例如浏览器）的请求，然后返回响应。

一个Web服务器网关接口（Web Server Gateway Interface） - WSGI - 就是干这活的。 WSGI_ 是一种Python标准。

.. _WSGI: http://wsgi.org/

uWSGI是一种WSGI实现。在这个教程中，我们将设置uWSGI，让它创建一个Unix socket，并且通过WSGI协议提供响应到web服务器。最后，我们完整的组件栈看起来将是这样的::

    the web client <-> the web server <-> the socket <-> uwsgi <-> Django

在你开始设置uWSGI之前
---------------------------------

virtualenv
^^^^^^^^^^

确保你正处在用来安装所需软件的虚拟机中 (稍后，我们将描述如何安装一个系统范围的uwsgi):

.. code-block:: bash

    virtualenv uwsgi-tutorial
    cd uwsgi-tutorial
    source bin/activate

Django
^^^^^^

将Django装到你的虚拟机中，创建一个新的项目，然后 ``cd`` 到该项目中:

.. code-block:: bash

    pip install Django
    django-admin.py startproject mysite 
    cd mysite

关于域名和端口
^^^^^^^^^^^^^^^^^^^^^^^^^

在这个教程中，我们将假设你的域名为 ``example.com`` 。用你自己的FQDN或者IP地址来代替。


从头到尾，我们将使用8000端口作为web服务器的公开端口，就像Django runserver默认的那样。当然，你可以使用任何你想要的端口，但是我已经选了这个，因此，它不会与web服务器可能已经选择的任何端口冲突。

基本的uWSGI安装和配置
------------------------------------------

把uWSGI安装到你的virtualenv中
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    pip install uwsgi
    
当然，有其他安装uWSGI的方式，但这种方式如其他方式一样棒。记住，你将需要安装Python开发包。对于Debian，或者Debian衍生系统，例如Ubuntu，你需要安装的是 ``pythonX.Y-dev`` ，其中，X.Y是你Python的版本。

基础测试
^^^^^^^^^^

创建一个名为 ``test.py`` 文件::

    # test.py
    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return [b"Hello World"] # python3
	#return ["Hello World"] # python2

.. note:: 需要考虑到，对于Python 3，需要 ``bytes()`` 。

运行uWSGI:

.. code-block:: bash

    uwsgi --http :8000 --wsgi-file test.py

选项表示:

* ``http :8000``: 使用http协议，端口8000 

* ``wsgi-file test.py``: 加载指定的文件，test.py

当浏览器访问8000端口时，这将直接提供一个'hello world'消息。
访问::

    http://example.com:8000

来看一看。如果是这样，那么意味着以下的组件栈正常::

    the web client <-> uWSGI <-> Python

测试你的Django项目
^^^^^^^^^^^^^^^^^^^^^^^^

现在，我们想让uWSGI做同样的事，但是返回一个Django站点而不是
``test.py`` 模块。

如果你还没有这样做，那么请确保你的 ``mysite`` 项目实际上正常工作::

    python manage.py runserver 0.0.0.0:8000

而如果正常，则使用uWSGI来运行它::

    uwsgi --http :8000 --module mysite.wsgi

* ``module mysite.wsgi``: 加载指定的wsgi模块

将你的浏览器指向该服务器；如果站点出现，那么意味着uWSGI可以为你虚拟环境中的Django应用服务，而这个栈工作正常::

    the web client <-> uWSGI <-> Django

现在，通常我们不会让浏览器直接与uWSGI通信。那是web服务器的工作，这是个穿针引线的活。

基本的nginx
-----------

安装nginx
^^^^^^^^^^^^^

.. code-block:: bash

    sudo apt-get install nginx  
    sudo /etc/init.d/nginx start    # start nginx

现在，通过在一个web浏览器上通过端口80访问它，来检查nginx是否正常 - 你应该会从nginx获得一个消息："Welcome to nginx!". 那意味着整个栈的这些模块都能一起正常工作::

    the web client <-> the web server

如果有其他的东东已经提供端口80的服务了，并且你想要在那里使用nginx，那么你将必须重新配置nginx来提供另一个端口的服务。但是，在这个教程中，我们将使用端口8000。

为你的站点配置nginx
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你会需要 ``uwsgi_params`` 文件，可用在uWSGI发行版本的 ``nginx`` 目录下，或者从
https://github.com/nginx/nginx/blob/master/conf/uwsgi_params
找到。

将其拷贝到你的项目目录中。一会儿，我们将告诉nginx引用它。

现在，创建一个名为mysite_nginx.conf的文件，然后将这个写入到它里面::

    # mysite_nginx.conf

    # the upstream component nginx needs to connect to
    upstream django {
        # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
        server 127.0.0.1:8001; # for a web port socket (we'll use this first) 
    }
 
    # configuration of the server
    server {
        # the port your site will be served on
        listen      8000;
        # the domain name it will serve for
        server_name .example.com; # substitute your machine's IP address or FQDN
        charset     utf-8;
   
        # max upload size
        client_max_body_size 75M;   # adjust to taste

        # Django media
        location /media  {
            alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
        }
   
        location /static {
            alias /path/to/your/mysite/static; # your Django project's static files - amend as required
        }
   
        # Finally, send all non-media requests to the Django server.
        location / {
            uwsgi_pass  django;
            include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed
        }
    }

这个配置文件告诉nginx提供来自文件系统的媒体和静态文件，以及处理那些需要Django干预的请求。对于一个大型部署，让一台服务器处理静态/媒体文件，让另一台处理Django应用，被认为是一种很好的做法，但是现在，这样就好了。

将这个文件链接到/etc/nginx/sites-enabled，这样nginx就可以看到它了:

.. code-block:: bash

    sudo ln -s ~/path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/

部署静态文件
^^^^^^^^^^^^^^^^

在运行nginx之前，你必须收集所有的Django静态文件到静态文件夹里。首先，必须编辑mysite/settings.py，添加:

.. code-block:: Python

    STATIC_ROOT = os.path.join(BASE_DIR, "static/")

然后运行

.. code-block:: bash

    python manage.py collectstatic



基本的nginx测试
^^^^^^^^^^^^^^^^

重启nginx:

.. code-block:: bash

    sudo /etc/init.d/nginx restart

要检查是否正确的提供了媒体文件服务，添加一个名为
``media.png`` 的图像到 ``/path/to/your/project/project/media directory`` 中，然后访问http://example.com:8000/media/media.png - 如果这能正常工作，那么至少你知道nginx正在正确的提供文件服务。

值得不仅仅是重启nginx，而是实际停止然后再次启动它，这将会通知是否有问题，以及问题在哪里。

nginx和uWSGI以及test.py
---------------------------

让nginx对 ``test.py`` 应用说句"hello world"吧。

.. code-block:: bash

    uwsgi --socket :8001 --wsgi-file test.py

这几乎与之前相同，除了这次有一个选项不同：

* ``socket :8001``: 使用uwsgi协议，端口为8001 

同时，已经配置了nginx在那个端口与uWSGI通信，而对外使用8000端口。访问:

http://example.com:8000/

来检查。而这是我们的栈::

    the web client <-> the web server <-> the socket <-> uWSGI <-> Python

同时，你可以试着看看在http://example.com:8001的uswgi输出 - 但很有可能它不会正常工作，因为你的浏览器使用http，而不是uWSGI，但你应该能够在终端上看到来自uWSGI的输出。

使用Unix socket而不是端口
-----------------------------------

目前，我们使用了一个TCP端口socket，因为它简单些，但事实上，使用Unix socket会比端口更好 - 开销更少。

编辑 ``mysite_nginx.conf``, 修改它以匹配:: 

    server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    # server 127.0.0.1:8001; # for a web port socket (we'll use this first) 

然后重启nginx.

再次运行uWSGI:

.. code-block:: bash

    uwsgi --socket mysite.sock --wsgi-file test.py 

这次， ``socket`` 选项告诉uWSGI使用哪个文件。

在浏览器中尝试访问http://example.com:8000/。

如果那不行
^^^^^^^^^^^^^^^^^^^^

检查nginx错误日志(/var/log/nginx/error.log)。如果你看到像这样的信息::

    connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission
    denied)

那么可能你需要管理这个socket上的权限，从而允许nginx使用它。

尝试::

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 # (very permissive)

或者::

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=664 # (more sensible) 

你可能还必须添加你的用户到nginx的组 (可能是
www-data)，反之亦然，这样，nginx可以正确地读取或写入你的socket。

值得保留nginx日志的输出在终端窗口中滚动，这样，在解决问题的时候，你就可以容易的参考它们了。

使用uwsgi和nginx运行Django应用
---------------------------------------------------

运行我们的Django应用：

.. code-block:: bash

    uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664

现在，uWSGI和nginx应该不仅仅可以为一个"Hello World"模块服务，还可以为你的Django项目服务。

配置uWSGI以允许.ini文件
-----------------------------------------

我们可以将用在uWSGI上的相同的选项放到一个文件中，然后告诉
uWSGI使用该文件运行。这使得管理配置更容易。

创建一个名为 ```mysite_uwsgi.ini``` 的文件::

    # mysite_uwsgi.ini file
    [uwsgi]

    # Django-related settings
    # the base directory (full path)
    chdir           = /path/to/your/project 
    # Django's wsgi file
    module          = project.wsgi
    # the virtualenv (full path) 
    home            = /path/to/virtualenv

    # process-related settings
    # master
    master          = true
    # maximum number of worker processes
    processes       = 10
    # the socket (use the full path to be safe
    socket          = /path/to/your/project/mysite.sock 
    # ... with appropriate permissions - may be needed
    # chmod-socket    = 664
    # clear environment on exit
    vacuum          = true           

然后使用这个文件运行uswgi：

.. code-block:: bash

    uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file

再次，测试Django站点是否如预期工作。

系统级安装uWSGI
-------------------------

目前，uWSGI只装在我们的虚拟环境中；出于部署需要，我们将需要让它安装在系统范围中。

停用你的虚拟环境::

    deactivate

然后在系统范围中安装uWSGI::

    sudo pip install uwsgi
                                                             
    # Or install LTS (long term support).
    pip install http://projects.unbit.it/downloads/uwsgi-lts.tar.gz

uWSGI的wiki描述了几种 `installation procedures`_. 在系统级安装
uWSGI之前，值得考虑下要选择哪个版本，以及最合适的安装方法。

.. _installation procedures: http://projects.unbit.it/uwsgi/wiki/Install
 
再次检查你是否仍然能如之前那样运行uWSGI:
    
.. code-block:: bash

    uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file

Emperor模式
------------

uWSGI可以运行在'emperor'模式。在这种模式下，它会监控uWSGI配置文件目录，然后为每个它找到的配置文件生成实例 ('vassals')。

每当修改了一个配置文件，emperor将会自动重启
vassal.

.. code-block:: bash

    # create a directory for the vassals
    sudo mkdir /etc/uwsgi
    sudo mkdir /etc/uwsgi/vassals
    # symlink from the default config directory to your config file
    sudo ln -s /path/to/your/mysite/mysite_uwsgi.ini /etc/uwsgi/vassals/
    # run the emperor
    uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

你或许需要使用sudo来运行uWSGI:

.. code-block:: bash

    sudo uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

选项表示:

* ``emperor``: 查找vassals (配置文件)的地方
* ``uid``: 进程一旦启动后的用户id
* ``gid``: 进程一旦启动后的组id
 
检查站点；它应该在运行。

系统启动时运行uWSGI
----------------------------------------

最后一步是让这一切在系统启动的时候自动发生。

对于许多系统来说，最简单 (如果不是最好的)的方式是使用 ``rc.local`` 文件。

编辑 ``/etc/rc.local`` 然后在"exit 0"行前添加::

    /usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data --daemonize /var/log/uwsgi-emperor.log


应该就这样！

进一步的配置
---------------------

理解这就是一个让你开始的 *教程* 很重要。你 **真的真的** 需要读一读nginx和uWSGI文档，并在将部署到生产环境之前学习可用的选项。

nginx和uWSGI都从友好的社区中获益，这些社区都能提供关于配置和使用的宝贵建议。

nginx
^^^^^

然而，nginx的一般配置并不在这篇教程的范围之内，对于一个生产网站，你可能会想要让它监听80端口，而不是8000。

你应该也为提供非Django的文件服务配置一个独立的nginx location块。例如，通过uWSGI提供静态文件服务并不有效。相反，直接从Nginx对其提供服务，并且完全绕过uWSGI会更有效。

uWSGI
^^^^^

uWSGI支持多种配置方式。见 `uWSGI's documentation`_ 和
`examples`_.

.. _uWSGI's documentation: https://uwsgi-docs.readthedocs.io
.. _examples: http://projects.unbit.it/uwsgi/wiki/Example

在这篇教程中，已经提到了一些uWSGI选项；对于生产上的部署，其他你应该看一看的选项包括 (通过样例设置列在这里)::

    env = DJANGO_SETTINGS_MODULE=mysite.settings # set an environment variable
    safe-pidfile = /tmp/project-master.pid # create a pidfile
    harakiri = 20 # respawn processes taking more than 20 seconds
    limit-as = 128 # limit the project to 128 MB
    max-requests = 5000 # respawn processes after serving 5000 requests
    daemonize = /var/log/uwsgi/yourproject.log # background the process & log
