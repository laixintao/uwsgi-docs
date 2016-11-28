将OpenBSD httpd作为代理使用
============================

从版本5.7起，OpenBSD就包含了一个带有FastCGI支持的最小化（真正最小化）的web服务器

(http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/httpd.8?query=httpd&sec=8)

启用它的第一步是编写它的配置文件 ```/etc/httpd.conf```

.. code-block:: c

   server "default" {
       listen on 0.0.0.0 port 80
   
       fastcgi socket ":3031"
   }

然后通过 ```rcctl``` 工具来启用并启动它：

.. code-block:: sh

   rcctl enable httpd
   rcctl start httpd

这个最小化配置将会在端口80上生成一个chroot的web服务器，作为用户'www'运行，并且使用FastCGI协议将每个请求转发到地址127.0.0.1:3031上。


现在，你只需在FastCGI地址上生成uWSGI：

.. code-block:: ini

   [uwsgi]
   fastcgi-socket = 127.0.0.1:3031
   ; a simple python app (eventually remember to load the python plugin)
   wsgi-file = app.py


显然，你可以将uWSGI作为一个全功能的CGI服务器使用 (当然，实际上，它比现有的任何一个cgi服务器都具有更多的特性 :P)，仅需记住强制modifier1为'9'：

.. code-block:: ini

   [uwsgi]
   fastcgi-socket = 127.0.0.1:3031
   fastcgi-modifier1 = 9
   ; a simple cgi-bin directory (eventually remember to load the cgi plugin)
   cgi = /var/www/cgi-bin

现在，你可以将你的cgi脚本放到/var/www/cgi-bin下了 (记得给它们可执行权限)

你也可以使用UNIX域socket，仅需记住，httpd服务器运行的根目录是/var/www，因此你必须在其下的一个目录中绑定uWSGI socket：

.. code-block:: ini

   [uwsgi]
   fastcgi-socket = /var/www/run/uwsgi.socket
   fastcgi-modifier1 = 9
   ; a simple cgi-bin directory
   cgi = /var/www/cgi-bin

.. code-block:: c

   server "default" {
       listen on 0.0.0.0 port 80
   
       fastcgi socket "/run/uwsgi.socket"
   }


如果你想只转发指定的路径到uWSGI，那么可以使用一个location指令：

.. code-block:: c

   server "default" {
       listen on 0.0.0.0 port 80
   
       location "/foo/*" {
           fastcgi socket ":3031"
       }
       
       location "/cgi-bin/*" {
           fastcgi socket ":3032"
       }
   }
   
注意
=====

目前 (2015年五月)，httpd仅能连接到unix域socket和绑定到地址127.0.0.1上的tcp fastcgi socket
