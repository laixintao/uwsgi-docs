uWSGI 1.9.2
===========

更新日志 20130326

错误修复
********

修复python3响应头管理问题 (错误的refcnt)

修复当postbuffering在合适的位置的时候，请求体上的readline()

修复ruby fiber插件

新特性
********

route-run以及cachestore路由操作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在，你可以在uWSGI缓存中自动存储响应：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   ; ensure the sweeper thread will run
   master = true
   cache2 = name=pippo2,items=10
   module = werkzeug.testapp:test_app
   route-run = cache:key=${REQUEST_URI},name=pippo2
   route-run = cachestore:key=${REQUEST_URI},expires=30,name=pippo2

这个例子检查每一个请求在缓存'pippo2'中是否可用。如果不可用，那么请求插件 (werkzeug test app)将会正常运行，然后其输出将会存储在缓存中 (只有在它返回HTTP 200状态的时候才会存储)

``--route-run`` 是一个新的选项，允许你直接调用路由动作，而无需检查特定的条件 (是哒，这是一个优化)

路由访问至cookie和查询字符串
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

查看更新文档 :doc:`InternalRouting`

空内部路由条件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

查看更新文档 :doc:`InternalRouting`

Geoip插件
^^^^^^^^^^^^^^^^

查看官方文档 docs :doc:`GeoIP`

SSI插件 (测试版)
^^^^^^^^^^^^^^^^^^^^^

查看官方文档 docs :doc:`SSI`

可用性
************

uWSGI 1.9.2于2013年03月26日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.2.tar.gz
