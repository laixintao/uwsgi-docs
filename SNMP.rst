内嵌SNMP服务器
========================

uWSGI服务器内嵌了一个小小的SNMP服务器，你可以用它来把你的web应用和监控基础设施集成在一起。

要启用SNMP支持，你必须运行uWSGI UDP服务器，然后选择一个SNMP community字符串 (这是SNMP使用的基本鉴权系统)。

.. code-block:: sh

  ./uwsgi -s :3031 -w staticfilesnmp --udp 192.168.0.1:2222 --snmp --snmp-community foo
  # or the following. Using the SNMP option to pass the UDP address is a lot more elegant. ;)
  ./uwsgi -s :3031 -w myapp --master --processes 4 --snmp=192.168.0.1:2222 --snmp-community foo

这将会在TCP端口3031和UDP端口2222上运行uWSGI服务器，启用SNMP，并使用"foo"作为community字符串。

请注意，SNMP服务器是在移除特权之后，在master进程中启动的。如果你想要它监听一个特权端口，那么你可以在linux上使用 :doc:`Capabilities<Capabilities>` ，或者使用 ``master-as-root`` 选项，作为root运行master经常。发行版中包含了 :file:`staticfilesnmp.py` 文件，它是一个通过SNMP公开了一个计数器的简单应用。

uWSGI SNMP服务器公开了2组信息：

* 由uWSGI服务器自身管理的一般信息。访问uWSGI SNMP信息的基本OID是 ``1.3.6.1.4.1.35156.17`` (``iso.org.dod.internet.private.enterprise.unbit.uwsgi``)。常规选项被映射到 ``1.3.6.1.4.1.35156.17.1.x`` 。
* 自定义信息由应用管理，并且通过 ``1.3.6.1.4.1.35156.17.2.x`` 访问。

所以，要获取uWSGI服务器管理的请求数，你可以

.. code-block:: sh

  snmpget -v2c -c foo 192.168.0.1:2222 1.3.6.1.4.1.35156.17.1.1 # 1.1 corresponds to ``general.requests``

导出自定义值
-----------------------

要从你的应用管理自定义值，你有这些Python函数，

* :py:func:`uwsgi.snmp_set_counter32`
* :py:func:`uwsgi.snmp_set_counter64`
* :py:func:`uwsgi.snmp_set_gauge`
* :py:func:`uwsgi.snmp_incr_counter32`
* :py:func:`uwsgi.snmp_incr_counter64`
* :py:func:`uwsgi.snmp_incr_gauge`
* :py:func:`uwsgi.snmp_decr_counter32`
* :py:func:`uwsgi.snmp_decr_counter64`
* :py:func:`uwsgi.snmp_decr_gauge`

所以，如果你想要将当前登录用户数量（这是一个计量，因为它可以降低）作为自定义OID 40公开，那么调用

.. code-block:: python

  users_logged_in = random.randint(0, 1024) # a more predictable source of information would be better.
  uwsgi.snmp_set_gauge(40, users_logged_in)

然后查询它，

.. code-block:: sh

  snmpget -v2c -c foo 192.168.0.1:2222 1.3.6.1.4.1.35156.17.2.40

可以配置系统snmp守护进程 (net-snmp) 来代理SNMP请求到uwsgi。这允许你同时运行系统守护进程和uwsgi，并且首先通过系统守护进程运行所有SNMP请求。要配置系统snmp守护进程 (net-snmp) 来代理连接到uwsgi，则添加这些行到/etc/snmp/snmpd.conf底部，然后重启守护进程：

.. code-block:: sh

   proxy -v 2c -c foo 127.0.0.1:2222 .1.3.6.1.4.1.35156.17
   view    systemview    included   .1.3.6.1.4.1.35156.17

用uwsgi中配置的community和端口替换'foo'和'2222'。

