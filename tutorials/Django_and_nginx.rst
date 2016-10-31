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

This should serve a 'hello world' message directly to the browser on port 8000.
访问::

    http://example.com:8000

to check. If so, it means the following stack of components works::

    the web client <-> uWSGI <-> Python

测试你的Django项目
^^^^^^^^^^^^^^^^^^^^^^^^

Now we want uWSGI to do the same thing, but to run a Django site instead of the
``test.py`` module.

If you haven't already done so, make sure that your ``mysite`` project actually works::

    python manage.py runserver 0.0.0.0:8000

And if it that works, run it using uWSGI::

    uwsgi --http :8000 --module mysite.wsgi

* ``module mysite.wsgi``: load the specified wsgi module

Point your browser at the server; if the site appears, it means uWSGI is able to
serve your Django application from your virtualenv, and this stack operates
correctly::

    the web client <-> uWSGI <-> Django

Now normally we won't have the browser speaking directly to uWSGI. That's a job
for the webserver, which will act as a go-between.

基本的nginx
-----------

安装nginx
^^^^^^^^^^^^^

.. code-block:: bash

    sudo apt-get install nginx  
    sudo /etc/init.d/nginx start    # start nginx

And now check that the nginx is serving by visiting it in a web browser on port
80 - you should get a message from nginx: "Welcome to nginx!". That means these
components of the full stack are working together::

    the web client <-> the web server

If something else *is* already serving on port 80 and you want to use nginx
there, you'll have to reconfigure nginx to serve on a different port. For this
tutorial though, we're going to be using port 8000.

为你的站点配置nginx
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You will need the ``uwsgi_params`` file, which is available in the ``nginx``
directory of the uWSGI distribution, or from
https://github.com/nginx/nginx/blob/master/conf/uwsgi_params

Copy it into your project directory. In a moment we will tell nginx to refer to
it.

Now create a file called mysite_nginx.conf, and put this in it::

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

This conf file tells nginx to serve up media and static files from the
filesystem, as well as handle requests that require Django's intervention. For a
large deployment it is considered good practice to let one server handle
static/media files, and another handle Django applications, but for now, this
will do just fine.

Symlink to this file from /etc/nginx/sites-enabled so nginx can see it:

.. code-block:: bash

    sudo ln -s ~/path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/

部署静态文件
^^^^^^^^^^^^^^^^

Before running nginx, you have to collect all Django static files in the static 
folder. First of all you have to edit mysite/settings.py adding:

.. code-block:: Python

    STATIC_ROOT = os.path.join(BASE_DIR, "static/")

and then run

.. code-block:: bash

    python manage.py collectstatic



基本的nginx测试
^^^^^^^^^^^^^^^^

重启nginx:

.. code-block:: bash

    sudo /etc/init.d/nginx restart

To check that media files are being served correctly, add an image called
``media.png`` to the ``/path/to/your/project/project/media directory``, then
visit http://example.com:8000/media/media.png - if this works, you'll know at
least that nginx is serving files correctly.

It is worth not just restarting nginx, but actually stopping and then starting
it again, which will inform you if there is a problem, and where it is.

nginx和uWSGI以及test.py
---------------------------

Let's get nginx to speak to the "hello world" ``test.py`` application.

.. code-block:: bash

    uwsgi --socket :8001 --wsgi-file test.py

This is nearly the same as before, except this time one of the options is
different:

* ``socket :8001``: use protocol uwsgi, port 8001 

nginx meanwhile has been configured to communicate with uWSGI on that port, and
with the outside world on port 8000. Visit:

http://example.com:8000/

to check. And this is our stack::

    the web client <-> the web server <-> the socket <-> uWSGI <-> Python

Meanwhile, you can try to have a look at the uswgi output at
http://example.com:8001 - but quite probably, it won't work because your browser
speaks http, not uWSGI, though you should see output from uWSGI in your
terminal.

使用Unix socket而不是端口
-----------------------------------

So far we have used a TCP port socket, because it's simpler, but in fact it's
better to use Unix sockets than ports - there's less overhead.

Edit ``mysite_nginx.conf``, changing it to match:: 

    server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    # server 127.0.0.1:8001; # for a web port socket (we'll use this first) 

and restart nginx.

Run uWSGI again:

.. code-block:: bash

    uwsgi --socket mysite.sock --wsgi-file test.py 

This time the ``socket`` option tells uWSGI which file to use.

Try http://example.com:8000/ in the browser.

如果那不行
^^^^^^^^^^^^^^^^^^^^

Check your nginx error log(/var/log/nginx/error.log). If you see something like::

    connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission
    denied)

then probably you need to manage the permissions on the socket so that nginx is
allowed to use it.

Try::

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 # (very permissive)

or::

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=664 # (more sensible) 

You may also have to add your user to nginx's group (which is probably
www-data), or vice-versa, so that nginx can read and write to your socket
properly. 

It's worth keeping the output of the nginx log running in a terminal window so
you can easily refer to it while troubleshooting.

使用uwsgi和nginx运行Django应用
---------------------------------------------------

运行我们的Django应用：

.. code-block:: bash

    uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664

Now uWSGI and nginx should be serving up not just a "Hello World" module, but
your Django project.

配置uWSGI以允许.ini文件
-----------------------------------------

We can put the same options that we used with uWSGI into a file, and then ask
uWSGI to run with that file. It makes it easier to manage configurations.

Create a file called ```mysite_uwsgi.ini```::

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

And run uswgi using this file:

.. code-block:: bash

    uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file

Once again, test that the Django site works as expected.

系统级安装uWSGI
-------------------------

So far, uWSGI is only installed in our virtualenv; we'll need it installed
system-wide for deployment purposes.

Deactivate your virtualenv::

    deactivate

and install uWSGI system-wide::

    sudo pip install uwsgi
                                                             
    # Or install LTS (long term support).
    pip install http://projects.unbit.it/downloads/uwsgi-lts.tar.gz

The uWSGI wiki describes several `installation procedures`_. Before installing
uWSGI system-wide, it's worth considering which version to choose and the most
apppropriate way of installing it.

.. _installation procedures: http://projects.unbit.it/uwsgi/wiki/Install
 
Check again that you can still run uWSGI just like you did before:
    
.. code-block:: bash

    uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file

Emperor模式
------------

uWSGI can run in 'emperor' mode. In this mode it keeps an eye on a directory of
uWSGI config files, and will spawn instances ('vassals') for each one it finds.

Whenever a config file is amended, the emperor will automatically restart the
vassal.

.. code-block:: bash

    # create a directory for the vassals
    sudo mkdir /etc/uwsgi
    sudo mkdir /etc/uwsgi/vassals
    # symlink from the default config directory to your config file
    sudo ln -s /path/to/your/mysite/mysite_uwsgi.ini /etc/uwsgi/vassals/
    # run the emperor
    uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

You may need to run uWSGI with sudo:

.. code-block:: bash

    sudo uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

The options mean:

* ``emperor``: where to look for vassals (config files)
* ``uid``: the user id of the process once it's started
* ``gid``: the group id of the process once it's started
 
Check the site; it should be running.

系统启动时运行uWSGI
----------------------------------------

The last step is to make it all happen automatically at system startup time.

For many systems, the easiest (if not the best) way to do this is to use the ``rc.local`` file.

Edit ``/etc/rc.local`` and add::

    /usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data --daemonize /var/log/uwsgi-emperor.log

before the line "exit 0".

And that should be it!

进一步的配置
---------------------

It is important to understand that this has been a *tutorial*, to get you
started. You **do** need to read the nginx and uWSGI documentation, and study
the options available before deployment in a production environment.

Both nginx and uWSGI benefit from friendly communities, who are able to offer
invaluable advice about configuration and usage.

nginx
^^^^^

General configuration of nginx is not within the scope of this tutorial though
you'll probably want it to listen on port 80, not 8000, for a production
website.

You should also configure a separate nginx location block for serving non-Django
files. For example, it's inefficient to serve static files via uWSGI. Instead, 
serve them directly from Nginx and completely bypass uWSGI.

uWSGI
^^^^^

uWSGI supports multiple ways to configure it. See `uWSGI's documentation`_ and
`examples`_.

.. _uWSGI's documentation: https://uwsgi-docs.readthedocs.io
.. _examples: http://projects.unbit.it/uwsgi/wiki/Example

Some uWSGI options have been mentioned in this tutorial; others you ought to
look at for a deployment in production include (listed here with example
settings)::

    env = DJANGO_SETTINGS_MODULE=mysite.settings # set an environment variable
    safe-pidfile = /tmp/project-master.pid # create a pidfile
    harakiri = 20 # respawn processes taking more than 20 seconds
    limit-as = 128 # limit the project to 128 MB
    max-requests = 5000 # respawn processes after serving 5000 requests
    daemonize = /var/log/uwsgi/yourproject.log # background the process & log
