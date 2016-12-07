uWSGI Perl支持 (PSGI)
=========================

在Perl世界中， :term:`PSGI` 等价于 :term:`WSGI` 。

* http://plackperl.org/
* https://github.com/plack/psgi-specs/blob/master/PSGI.pod

官方支持PSGI插件，并且官方分配给它一个uwsgi modifier, ``5`` 。因此，照例，如果你要将请求派发给Perl应用，那么在你的web服务器配置中设置 ``modifier1`` 值为5。

编译PSGI插件
-------------------------

你可以使用提供的 :file:`buildconf/psgi.ini` 文件来构建一个仅PSGI的uWSGI服务器。在构建PSGI插件之前，确保安装了 ``ExtUtils::Embed`` 模块和它的先决条件。

.. code-block:: sh

    python uwsgiconfig.py --build psgi
    # or compile it as a plugin...
    python uwsgiconfig.py --plugin plugins/psgi
    # and if you have not used the default configuration
    # to build the uWSGI core, you have to pass
    # the configuration name you used while doing that:
    python uwsgiconfig.py --plugin plugins/psgi core
    
或者 (如常)，你可以使用网络安装程序：

.. code-block:: sh

    curl http://uwsgi.it/install | bash -s psgi /tmp/uwsgi
    
这样，在/tmp/uwsgi中，你就有了一个带有perl支持的单个uwsgi二进制文件。

使用
-----

这个插件只导出了一个选项： ``psgi <app>``

你可以这样简单加载应用

.. code-block:: sh

    ./uwsgi -s :3031 -M -p 4 --psgi myapp.psgi -m
    # or when compiled as a plugin,
    ./uwsgi --plugins psgi -s :3031 -M -p 4 --psgi myapp.psgi -m


测试PSGI框架/应用
-----------------------------------

以下框架/应用已经用uWSGI测试过了：

* MojoMojo_
* Mojolicious_
* Mojolicious+perlbrew+uWSGI+nginx_ 安装包

.. _MojoMojo: http://mojomojo.org/
.. _Mojolicious: http://mojolicio.us/
.. _Mojolicious+perlbrew+uWSGI+nginx: https://github.com/kraih/mojo/wiki/nginx-&-uwsgi(psgi)-&-perlbrew-&-mojolicious

多应用支持
-----------------

你可以使用 ``mount`` 选项或者使用 ``UWSGI_SCRIPT``/``UWSGI_FILE`` 请求变量在同一个uWSGI进程中加载多个几乎相互隔离的应用。

.. code-block:: ini

    [uwsgi]
    
    mount = app1=foo1.pl
    mount = app2=foo2.psgi
    mount = app3=foo3.pl

.. code-block:: nginx

    server {
      server_name example1.com;
      location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:3031;
        uwsgi_param UWSGI_APPID app1;
        uwsgi_param UWSGI_SCRIPT foo1.pl;
        uwsgi_modifier1 5;
      }
    }
    
    server {
      server_name example2.com;
      location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:3031;
        uwsgi_param UWSGI_APPID app2;
        uwsgi_param UWSGI_SCRIPT foo2.psgi;
        uwsgi_modifier1 5;
      }
    }
    
    server {
      server_name example3.com;
      location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:3031;
        uwsgi_param UWSGI_APPID app3;
        uwsgi_param UWSGI_SCRIPT foo3.pl;
        uwsgi_modifier1 5;
      }
    }

自动重载器 (自uWSGI 1.9.18起)
-------------------------------------

选项 --perl-auto-reload <n> 允许你指示uWSGI监控由perl vm导入的每一个单一的模块。

每当其中一个模块改变，整个实例将会被（优雅）加载。

监控器通过在处理一个请求并且经过指定的秒数（自最后一次运行起）（这个秒数是选项的值）之后，迭代%INC工作的。

这可能看起来是次优的 (你会获得从以下请求起的新内容)，但对于perl的工作方式而言，它是一种更稳定（安全）的方式。

如果你想从监控跳过指定文件，只需使用--perl-auto-reload-ignore来添加它们。

记住，总会扫描%INC中的模块，如果你想要监控你的.psgi文件，那么你需要使用经典的--touch-reload选项来指定它们。

注意事项
---------

* 应该默认支持异步。
* 在启用了ithreads的perl构建中，支持线程。对于每个应用，会为每个线程创建一个新的解释器。这应该不会与一个简单的多进程基于fork()的子系统有太大的不同。
* 存在目前未知的内存泄漏。


真实世界例子： `HTML::Mason`
---------------------------------

1. 安装来自CPAN的HTML::Mason PSGI处理器，并为你的站点创建一个目录。
   
   .. code-block:: sh
      
      cpan install HTML::Mason::PSGIHandler
      mkdir mason

2. 创建 ``mason/index.html``:

   .. code-block:: html
   
       % my $noun = 'World';
       % my $ua = $r->headers_in;
       % foreach my $hh (keys %{$ua}) {
        <% $hh %><br/>
       % }
       Hello <% $noun %>!<br/>
       How are ya?<br/>
       Request <% $r->method %> <% $r->uri %><br/>

3. 创建PSGI文件 (``mason.psgi``):

   .. code-block:: perl
   
       use HTML::Mason::PSGIHandler;
       
       my $h = HTML::Mason::PSGIHandler->new(
    	      comp_root => "/Users/serena/uwsgi/mason", # required
       );
       
       my $handler = sub {
    	      my $env = shift;
    	      $h->handle_psgi($env);
       };
    
   注意 ``comp_root`` ，它必须是一个绝对路径！

4. 现在运行uWSGI:

   .. code-block:: sh

    ./uwsgi -s :3031 -M -p 8 --psgi mason.psgi -m

5. 然后在你的浏览器中访问 ``/index.html`` 。
