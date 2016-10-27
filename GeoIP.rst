GeoIP插件
================

``geoip`` 插件添加新的路由变量到你的内部路由子系统中。
GeoIP变量前缀为"geoip"标签。要构建geoip插件，你需要官方的GeoIP C库和它的头文件。支持的数据库是country和city，它们会在启动的时候完全加载到内存中。

可以访问country数据库的以下变量：

* ``${geoip[country_code]}``
* ``${geoip[country_code3]}``
* ``${geoip[country_name]}``

而以为存储数据而增长的内存使用为代价，city数据库提供更多变量

* ``${geoip[continent]}``
* ``${geoip[country_code]}``
* ``${geoip[country_code3]}``
* ``${geoip[country_name]}``
* ``${geoip[region]}``
* ``${geoip[region_name]}``
* ``${geoip[city]}``
* ``${geoip[postal_code]}``
* ``${geoip[latitude]}`` (``${geoip[lat]}``)
* ``${geoip[longitude]}`` (``${geoip[lon]}``)
* ``${geoip[dma]}``
* ``${geoip[area]}``

启用geoip查询
*********************

要启用GeoIP查询系统，你需要加载至少一个数据库。在加载完geoip插件后，你会获得2个新的选项：

* ``--geoip-country`` 指定一个country数据库
* ``--geoip-city`` 指定一个city数据库

如果你没有至少指定一个数据库，那么，该系统将会总是返回空字符串。

一个例子
**********

.. code-block:: ini

   [uwsgi]
   plugin = geoip
   http-socket = :9090
   ; load the geoip city database
   geoip-city = GeoLiteCity.dat
   module = werkzeug.testapp:test_app
   ; first some debug info (addvar will ad WSGI variables you will see in the werkzeug testapp)
   route-run = log:${geoip[country_name]}/${geoip[country_code3]}
   route-run = addvar:COUNTRY=${geoip[country_name]}
   route-run = log:${geoip[city]}/${geoip[region]}/${geoip[continent]}
   route-run = addvar:COORDS=${geoip[lon]}/${geoip[lat]}
   route-run = log:${geoip[region_name]}
   route-run = log:${geoip[dma]}/${geoip[area]}

   ; then something more useful
   ; block access to all of the italians (hey i am italian do not start blasting me...)
   route-if = equal:${geoip[country_name]};Italy break:403 Italians cannot see this site :P
   ; try to serve a specific page translation
   route = ^/foo/bar/test.html static:/var/www/${geoip[country_code]}/test.html

内存使用
************

country数据库很小，因此你在使用它的时候一般没有问题。反而，city数据库能很大 (从20MB到大于40MB)。如果你有大量使用GeoIP的city数据库的实例，并且是使用最近的
Linux系统，那么考虑使用 :doc:`KSM` 来减少内存使用。
GeoIP数据库使用的所有内存可以由所有实例共享。
