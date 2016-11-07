使用inetd/xinetd进行socket激活
===================================

Inetd和Xinetd是两个用来按需启动网络进程的守护进程。你也可以在uWSGI中使用。

Inetd
-----


.. code-block:: inetd

    127.0.0.1:3031 stream tcp wait root /usr/bin/uwsgi uwsgi -M -p 4 --wsgi-file /root/uwsgi/welcome.py --log-syslog=uwsgi

通过这个配置，一旦进行了第一个连接，就会在端口3031上运行uWSGI。注意：第一个参数 (紧跟着/usr/bin/uwsgi后的那一个)会被映射到 ``argv[0]`` 。不要忘了这个 -- 如果你想确定的话，总是将其设置为 ``uwsgi`` 。

Xinetd 
------

.. code-block:: xinetd

    service uwsgi
    	{
    	        disable         = no
    	        id              = uwsgi-000
    	        type            = UNLISTED
    	        socket_type     = stream
    	        server          = /root/uwsgi/uwsgi
    	        server_args     = --chdir /root/uwsgi/ --module welcome --logto /tmp/uwsgi.log
    	        port            = 3031
    	        bind            = 127.0.0.1
    	        user            = root
    	        wait            = yes
    	}

再次，你无需在uWSGI中指定socket，因为xinetd会将其传递给服务器。
