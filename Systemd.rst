Systemd
=======

uWSGI是用于 `systemd <http://www.freedesktop.org/wiki/Software/systemd>`_ 的一个新型守护进程。

它可以通知状态改变和准备好使用。

当uWSGI检测到它正运行于systemd下时，启用通知系统。

添加Emperor到systemd
*****************************

将uWSGI应用与你的初始化系统集成的一个方法是使用 :doc:`Emperor<Emperor>` 。

你的初始化系统将只会与统治所有的应用的Emperor进行通信。

创建一个systemd服务文件 (你可以将其保存为/etc/systemd/system/emperor.uwsgi.service)

.. note::
   小心使用一些systemd版本 (例如，Debian Jessie的215)，因为SIGQUIT信号将会毁掉systemd服务。在那里，使用KillSignal=SIGTERM + "die-on-term" UWSGI选项。

.. code-block:: ini

   [Unit]
   Description=uWSGI Emperor
   After=syslog.target

   [Service]
   ExecStart=/root/uwsgi/uwsgi --ini /etc/uwsgi/emperor.ini
   # Requires systemd version 211 or newer
   RuntimeDirectory=uwsgi
   Restart=always
   KillSignal=SIGQUIT
   Type=notify
   StandardError=syslog
   NotifyAccess=all

   [Install]
   WantedBy=multi-user.target

然后运行它

.. code-block:: sh

   systemctl start emperor.uwsgi.service

检查它的状态。

.. code-block:: sh

   systemctl status emperor.uwsgi.service

你会看到Emperor报告管理的vassal数目给systemd (和你)。

.. code-block:: sh

   emperor.uwsgi.service - uWSGI Emperor
    Loaded: loaded (/etc/systemd/system/emperor.uwsgi.service)
	  Active: active (running) since Tue, 17 May 2011 08:51:31 +0200; 5s ago
   Main PID: 30567 (uwsgi)
	  Status: "The Emperor is governing 1 vassals"
	  CGroup: name=systemd:/system/emperor.uwsgi.service
		  ├ 30567 /root/uwsgi/uwsgi --ini /etc/uwsgi/emperor.ini
		  ├ 30568 /root/uwsgi/uwsgi --ini werkzeug.ini
		  └ 30569 /root/uwsgi/uwsgi --ini werkzeug.ini


你可以这样停止Emperor (以及它管理的所有应用)

.. code-block:: sh

   systemctl stop emperor.uwsgi.service

一个简单的 ``emperor.ini`` 可以看起来像这样 (www-data只是一个匿名用户)

注意：不要守护Emperor (或者master)，除非你知道你在干什么！！！

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   uid = www-data
   gid = www-data

如果你想要允许每个vassal在不同的特权下允许，那么将 ``uid`` 和 ``gid`` 选项从emperor配置中移除 (请阅读Emperor文档！)

日志记录
*******

使用前面的服务文件，所有的Emperor消息都会到syslog中。你可以通过移除 ``StandardError=syslog`` 指令来避免它。

如果你那样做，那么确保在你的Emperor配置中设置 ``--logto`` 选项，否则，你所有的日子都将会丢失！

将socket放到/run/
************************

在一个现代系统上，/run/是作为一个tmpfs挂载的，并且是存放socket和pid文件的正确之地。要让systemd自动创建一个带有正确的user/group的/run/uwsgi/子目录，并且当守护进程停止的时候清理该目录，那么添加

.. code-block:: ini

   RuntimeDirectory=uwsgi

到你的systemd uwsgi单元文件的[Service]部分。这个 ``RuntimeDirectory`` 参数要求systemd版本211或者以上。对于systemd较老的版本，创建一个systemd-tmpfiles配置文件 (你可以将其存为/etc/tmpfiles.d/emperor.uwsgi.conf):

.. code-block:: ini

   d /run/uwsgi 0755 www-data www-data -

Socket激活
*****************

从uWSGI 0.9.8.3起，socket激活就可以用了。你可以设置systemd只在第一个socket连接之后才生成uWSGI实例。

创建所需的emperor.uwsgi.socket (在 ``/etc/systemd/system/emperor.uwsgi.socket``)。注意，*.socket文件名必须匹配*.service文件名。

.. code-block:: ini

   [Unit]
   Description=Socket for uWSGI Emperor

   [Socket]
   # Change this to your uwsgi application port or unix socket location
   ListenStream=/tmp/uwsgid.sock

   [Install]
   WantedBy=sockets.target

然后禁用服务，并且启用socket单元。

.. code-block:: sh

   # systemctl disable emperor.uwsgi.service
   # systemctl enable emperor.uwsgi.socket
   
当使用Systemd socket激活的时候，你无需在你的uWSGI配置中指定任何socket；实例将会从Systemd继承socket。

要拥有Systemd socket激活之下的uWSGI服务的HTTP (取代二进制uwsgi协议)，则设置 ``protocol`` 为 ``http`` ；例如，在一个INI中，这样：

.. code-block:: ini
   
   [uwsgi]
   protocol = http
   wsgi = ...
   ...
   
在systemd中，每个应用一个服务
******************************

另一个方法是让systemd处理启动单独的应用，同时利用systemd模板单元文件，当然还有socket激活。每个应用将会在其自己的用户下允许。

``/etc/systemd/system/uwsgi-app@.socket``:

.. code-block:: ini

  [Unit]
  Description=Socket for uWSGI app %i

  [Socket]
  ListenStream=/var/run/uwsgi/%i.socket
  SocketUser=www-%i
  SocketGroup=www-data
  SocketMode=0660

  [Install]
  WantedBy=sockets.target

``/etc/systemd/system/uwsgi-app@.service``:

.. code-block:: ini

  [Unit]
  Description=%i uWSGI app
  After=syslog.target

  [Service]
  ExecStart=/usr/bin/uwsgi \
          --ini /etc/uwsgi/apps-available/%i.ini \
          --socket /var/run/uwsgi/%i.socket
  User=www-%i
  Group=www-data
  Restart=on-failure
  KillSignal=SIGQUIT
  Type=notify
  StandardError=syslog
  NotifyAccess=all

现在，添加一个新的应用到你的系统中就只是创建合适的用户并启用socket和服务的事情了。例如，如果要配置cgit:

.. code-block:: sh

  adduser www-cgit --disabled-login --disabled-password \
    --ingroup www-data --home /var/lib/www/cgit --shell /bin/false
  systemctl enable uwsgi-app@cgit.socket
  systemctl enable uwsgi-app@cgit.service
  systemctl start uwsgi-app@cgit.socket

然后配置ini文件 ``/etc/uwsgi/apps-available/cgit.ini``:

.. code-block:: ini

  [uwsgi]
  master = True
  cheap = True
  idle = 600
  die-on-idle = True # If app is not used often, it will exit and be launched
                     # again by systemd requested by users.

  manage-script-name = True

  plugins = 0:cgi
  cgi = /usr/lib/cgit/cgit.cgi

最后，如果适用，如往常一样配置你的HTTP服务器。
