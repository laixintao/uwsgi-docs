Cherokee支持
================

.. note::

  Cherokee的最新官方版本有一个uWSGI配置向导。如果你想要使用它，那么你必须在系统的 ``PATH`` 所包含的目录下安装uWSGI。

* 为你的目标设置UWSGI处理器。
* 如果你使用的是默认的目标 (``/``)，那么记得取消检查 ``check_file`` 属性。
* 配置类型"Remote"的“信息源”，指定uWSGI的socket名。如果你的uWSGI支持TCP，那么你可以通过在一个不同的机器上生成uWSGI服务器来构建集群。

.. note::

  记得用合适的处理器为你所有包含静态文件 (例如
  /media /images ...) 的URI添加目标

动态应用
------------

如果你想要热添加应用，则在uWSGI的handler选项中指定 ``UWSGI_SCRIPT`` 变量：

* 在 `Add new custom environment variable` 中指定 ``UWSGI_SCRIPT`` 名，并且将你的WSGI脚本名 (不带.py扩展) 作为值。

你的应用将会在第一个请求中自动被加载。
