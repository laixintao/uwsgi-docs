动态添加应用
===============================

注意：这并不是托管多个应用的最好的方法。你最好为每个应用运行一个uWSGI实例。

你可以在没有配置应用的情况下启动uWSGI服务器。

要加载一个新的应用，可以使用uwsgi包中的这些变量：

* ``UWSGI_SCRIPT`` -- 传递定义了一个 ``application`` 可调用对象的WSGI脚本名
* 或者 ``UWSGI_MODULE`` 和 ``UWSGI_CALLABLE`` -- 模块名 (可导入路径) 以及可调用对象的名字来从该模块调用

在Cherokee, Nginx, Apache, cgi_dynamic上有对动态应用的官方支持。可以很容易添加它们到Tomcat和Twisted处理器中。

定义带动态应用的VirtualEnv
-------------------------------------

Virtualenvs是基于 ``Py_SetPythonHome()`` 函数的。只有在 ``Py_Initialize()`` 之前调用这个函数才有用，因此不能将其用于动态应用。

要定义一个带DynamicApps的VirtualEnv，hack是唯一的解决方法。

首先，你必须告诉python不要导入 ``site`` 模块。这个模块添加所有的 ``site-packages`` 到 ``sys.path`` 中。要模拟virtualenvs，我们必须只在子解析器初始化后加载site模块。跳过第一次 ``import site`` ，我们现在可以在动态应用加载时简单设置 ``sys.prefix`` 和
``sys.exec_prefix`` ，然后调用

.. code-block:: c

   PyImport_ImportModule("site");
   // Some users would want to not disable initial site module loading, so the site module must be reloaded:
   PyImport_ReloadModule(site_module);

现在，我们可以使用 ``UWSGI_PYHOME`` 变量动态设置VirtualEnv：


.. code-block:: c

   location / {
     uwsgi_pass 192.168.173.5:3031;
     include uwsgi_params;
     uwsgi_param UWSGI_SCRIPT mytrac;
     uwsgi_param UWSGI_PYHOME /Users/roberto/uwsgi/VENV2;
   }
   


 
 
