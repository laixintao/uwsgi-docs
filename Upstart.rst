通过Upstart运行uWSGI
=========================

Upstart是类Ubuntu发行版本的init系统。

它基于声明式配置文件，而非昔日的shell脚本，配置文件放在 :file:`/etc/init` 目录中。

一个简单的脚本 (/etc/init/uwsgi.conf)
--------------------------------------

.. code-block:: upstart

    # simple uWSGI script
    
    description "uwsgi tiny instance"
    start on runlevel [2345]
    stop on runlevel [06]
    
    respawn
    
    exec uwsgi --master --processes 4 --die-on-term --socket :3031 --wsgi-file /var/www/myapp.wsgi
    
使用Emperor
-----------------

.. seealso:: :doc:`Emperor`

对于每个应用而言，比init文件更好的一种方法是通过Upstart只启动一个Emperor，然后让这个Emperor处理剩下的工作。

.. code-block:: upstart

    # Emperor uWSGI script
    
    description "uWSGI Emperor"
    start on runlevel [2345]
    stop on runlevel [06]
    
    respawn
    
    exec uwsgi --emperor /etc/uwsgi

如果你想在master进程下运行Emperor（出于访问高级特性的目的），那么记住添加--die-on-term


.. code-block:: upstart

    # Emperor uWSGI script
    
    description "uWSGI Emperor"
    start on runlevel [2345]
    stop on runlevel [06]
    
    respawn
    
    exec uwsgi --master --die-on-term --emperor /etc/uwsgi
    
什么是--die-on-term？
----------------------

默认情况下，uWSGI将SIGTERM信号映射到一个“一个粗暴的加载过程”。

然而，Upstart使用SIGTERM来完全关闭进程。 ``die-on-term`` 反转了SIGTERM和SIGQUIT对uWSGI的意义。

前者将关闭整个栈，而后者将粗暴的加载它。

socket激活 (自Ubuntu 12.04起)
-------------------------------------

较新版本的Upstart具有一个类似于Inetd的功能，它在连接到特定socket的时候让进程启动。

只有当一个客户端（或者web服务器）第一次连接到uWSGI的时候，你才可以使用这个功能来启动uWSGI。

'start on socket'指令将触发这种行为。

你无需在uWSGI中指定这个socket，因为Upstart自己将会传递它。

.. code-block:: upstart

    # simple uWSGI script
    
    description "uwsgi tiny instance"
    start on socket PROTO=inet PORT=3031
    stop on runlevel [06]
    
    respawn
    
    exec uwsgi --master --processes 4 --die-on-term --wsgi-file /var/www/myapp.wsgi

