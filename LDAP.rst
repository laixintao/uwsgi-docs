使用LDAP配置uWSGI
===========================

可以使用LDAP配置uWSGI。LDAP是一种集中uWSGI服务器的大型集群的配置的灵活方法。

.. note::

  必须在 :doc:`building<Build>` uWSGI时启用LDAP支持。要求
  `libldap` 库。


导入uWSGIConfig模式
--------------------------------

带 `--ldap-schema` 或者 `--ldap-schema-ldif` 参数运行uWSGI会让它输出一个标准的LDAP模式 (或者一个LDIF文件)，你可以将其导入到你的服务器中。

LDIF dump的一个例子
--------------------

This is an 这是带有 `uWSGIConfig` 项的一个OpenLDAP服务器的LDIF dump，运行着一个Trac实例。

.. code-block:: ldif

  dn: dc=projects,dc=unbit,dc=it
  objectclass: uWSGIConfig
  objectclass: domain
  dc: projects
  uWSGIsocket: /var/run/uwsgi/projects.unbit.it.sock
  uWSGIhome: /accounts/unbit/tracvenv
  uWSGImodule: trac.web.main:dispatch_request
  uWSGImaster: TRUE
  uWSGIprocesses: 4
  uWSGIenv: TRAC_ENV=/accounts/unbit/trac/uwsgi

使用
-----

你仅需传递一个有效的LDAP url给 `--ldap` 选项。只有返回的第一个项才会被当成配置使用。

..
  
  uwsgi --ldap ldap://ldap.unbit.it/dc=projects,dc=unbit,dc=it


如果你想要一个带有子范围的过滤器 (这将会返回树 `dc=projects,dc=unbit,dc=it` 下 `ou=Unbit` 的第一条记录):

..

  uwsgi --ldap ldap://ldap.unbit.it/dc=projects,dc=unbit,dc=it?sub?ou=Unbit


.. attention:
  
  目前不支持鉴权。
