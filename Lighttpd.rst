Lighttpd支持
================

.. note:: Lighttpd支持是实验性的。

用于Lighttpd的uwsgi处理器位于uWSGI发行版的 ``/lighttpd`` 目录下。

构建模块
-------------------

首先下载lighttpd的源代码，然后解压缩它。然后拷贝uWSGI发行版中的
``lighttpd/mod_uwsgi.c`` 文件到Lighttpd的
``/src`` 目录下。添加以下内容到lighttpd ``src/Makefile.am`` 文件中的accesslog块之后：

::

  lib_LTLIBRARIES += mod_uwsgi.la
  mod_uwsgi_la_SOURCES = mod_uwsgi.c
  mod_uwsgi_la_LDFLAGS = -module -export-dynamic -avoid-version -no-undefined
  mod_uwsgi_la_LIBADD = $(common_libadd)

然后启动

::

  autoreconf -fi

接着和往常一样，

::

  ./configure && make && make install

配置Lighttpd
--------------------

修改你的配置文件：

::
  
  server.modules = (
    ...
    "mod_uwsgi",
    ...
  )

  # ...

  uwsgi.server = (
    "/pippo" => (( "host" => "192.168.173.15", "port" => 3033 )),
    "/" => (( "host" => "127.0.0.1", "port" => 3031 )),
  )

如果你在相同的虚拟路径/URI之下指定多个主机，那么负载均衡将会被激活，并且使用"Fair"算法。
