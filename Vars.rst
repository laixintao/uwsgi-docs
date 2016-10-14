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

Load a new app (defined as ``module:callable``) mapped into ``SCRIPT_NAME``.

::

  uwsgi_param UWSGI_MODULE werkzeug.testapp;
  uwsgi_param UWSGI_CALLABLE test_app;
  uwsgi_param SCRIPT_NAME  /testapp;


``UWSGI_PYHOME``
----------------

Dynamically set the Python :ref:`Virtualenv` for a :doc:`dynamic application<DynamicApps>`.

.. seealso:: :ref:`DynamicVirtualenv`

``UWSGI_CHDIR``
---------------

``chdir()`` to the specified directory before managing the request.

``UWSGI_FILE``
--------------

Load the specified file as a new dynamic app.

``UWSGI_TOUCH_RELOAD``
----------------------

Reload the uWSGI stack when the specified file's modification time has changed since the last request.

::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_TOUCH_RELOAD /tmp/touchme.foo;
    uwsgi_pass /tmp/uwsgi.sock;
  }

``UWSGI_CACHE_GET``
-------------------

.. seealso:: :doc:`Caching`

Check the uWSGI cache for a specified key. If the value is found, it will be returned as raw HTTP output instead of the usual processing of the request.

::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_CACHE_GET $request_uri;
    uwsgi_pass 127.0.0.1:3031;
  }


``UWSGI_SETENV``
----------------

Set the specified environment variable for a new dynamic app.

.. 注意:: To allow this in Python applications you need to enable the ``reload-os-env`` uWSGI option.

Dynamically load a Django app without using a WSGI file/module::

  location / {
    include uwsgi_params;
    uwsgi_param UWSGI_SCRIPT django.core.handlers.wsgi:WSGIHandler();
    uwsgi_param UWSGI_CHDIR /mydjangoapp_path;
    uwsgi_param UWSGI_SETENV DJANGO_SETTINGS_MODULE=myapp.settings;
  }


``UWSGI_APPID``
---------------

.. 注意:: Available since 0.9.9.

Bypass ``SCRIPT_NAME`` and :doc:`VirtualHosting` to let the user choose the mountpoint without limitations (or headaches).

The concept is very generic: ``UWSGI_APPID`` is the identifier of an application. If it is not found in the internal list of apps, it will be loaded.

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

