Cherokee支持
================

.. note::

  Recent official versions of Cherokee have an uWSGI configuration wizard. If
  you want to use it you have to install uWSGI in a directory included in your
  system ``PATH``.

* Set the UWSGI handler for your target.
* 如果你使用的是默认的目标 (``/``)，那么记得uncheck the ``check_file`` property.
* Configure an "information source" of type "Remote", specifying the socket name of uWSGI. If your uWSGI has TCP support, you can build a cluster by spawning the uWSGI server on a different machine.

.. note::

  Remember to add a target for all of your URI containing static files (ex.
  /media /images ...) using an appropriate handler

动态应用
------------

如果你想要热添加应用，则在uWSGI的handler选项中指定 ``UWSGI_SCRIPT`` 变量：

* In the section: `Add new custom environment variable` specify ``UWSGI_SCRIPT`` as name and the name of your WSGI script (without the .py extension) as the value.

你的应用将会在第一个请求中自动被加载。
