uwsgi协议魔术变量
==============================

你可以通过使用web服务器（或一般使用一个uwsgi兼容的客户端）传递的专用的变量来动态调整或配置uWSGI服务器的各个方面。

* 对于Nginx，使用 ``uwsgi_param <name> <value>;`` 指令。
* 对于Apache，使用 ``SetEnv <name> <value>`` 指令。

``UWSGI_SCHEME``
----------------

当不能可靠确定时，设置URL方案。例如，这可以用来强制使用HTTPS (使用值 ``https``)。

``UWSGI_SCRIPT``
----------------

将指定的脚本作为一个映射到 ``SCRIPT_NAME`` 的新应用加载。该应用将明显只加载一次，而不是在每个请求都加载。

::

  uwsgi_param UWSGI_SCRIPT werkzeug.testapp:test_app;
  uwsgi_param SCRIPT_NAME  /testapp;


``UWSGI_MODULE`` 和 ``UWSGI_CALLABLE``
---------------------------------------

加载一个新的应用 (定义为 ``module:callable``)，将其映射到 ``SCRIPT_NAME``.

::

  uwsgi_param UWSGI_MODULE werkzeug.testapp;
  uwsgi_param UWSGI_CALLABLE test_app;
  uwsgi_param SCRIPT_NAME  /testapp;


``UWSGI_PYHOME``
----------------

为一个 :doc:`dynamic application<DynamicApps>` 动态设置Python :ref:`Virtualenv`。

.. seealso:: :ref:`DynamicVirtualenv`

``UWSGI_CHDIR``
---------------

在管理请求之前 ``chdir()`` 到指定的目录。

``UWSGI_FILE``
--------------

将指定的文件作为一个新的动态应用加载。

``UWSGI_TOUCH_RELOAD``
----------------------

当指定的文件的修改时间自最后一个请求后发生改变时，重载uWSGI栈。

::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_TOUCH_RELOAD /tmp/touchme.foo;
    uwsgi_pass /tmp/uwsgi.sock;
  }

``UWSGI_CACHE_GET``
-------------------

.. seealso:: :doc:`Caching`

为特定的键查看uWSGI换成。如果找到该键，它将会作为原始的HTTP输出返回，而不是请求的一般处理。

::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_CACHE_GET $request_uri;
    uwsgi_pass 127.0.0.1:3031;
  }


``UWSGI_SETENV``
----------------

为一个新的动态应用设置指定的环境变量。

.. note:: 要在Python应用中使用这个功能，你需要启用 ``reload-os-env`` uWSGI选项。

在不使用一个WSGI文件/模块的情况下动态加载一个Django应用::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_SCRIPT django.core.handlers.wsgi:WSGIHandler();
    uwsgi_param UWSGI_CHDIR /mydjangoapp_path;
    uwsgi_param UWSGI_SETENV DJANGO_SETTINGS_MODULE=myapp.settings;
  }


``UWSGI_APPID``
---------------

.. note:: 自0.9.9起可用。

绕过 ``SCRIPT_NAME`` 和 :doc:`VirtualHosting` ，从而让用户在没有限制（或者不头疼）的情况下选择挂载点。

这个概念是非常通用的: ``UWSGI_APPID`` 是一个应用的标识符。如果在应用的内部列表中找不到它，那么要加载它。

::

  server {
      server_name server001;
      location / {
          include uwsgi_params;
          uwsgi_param UWSGI_APPID myfunnyapp;
          uwsgi_param UWSGI_FILE /var/www/app1.py
      }
  }
  
  server {
      server_name server002;
      location / {
          include uwsgi_params;
          uwsgi_param UWSGI_APPID myamazingapp;
          uwsgi_param UWSGI_FILE /var/www/app2.py
      }
  }

