SPDY路由器 (uWSGI 1.9)
===========================

自uWSGI 1.9起，HTTPS路由器已被扩展，支持SPDY协议的版本3。

要运行带SPDY支持的HTTPS路由器，则使用 ``--https2`` 选项：

.. code-block:: sh

   uwsgi --https2 addr=0.0.0.0:8443,cert=foobart.crt,key=foobar.key,spdy=1 --module werkzeug.testapp:test_app

这将会在端口8443上启动一个带有哦SPDY支持的HTTPS路由器，转发请求到实例正在运行的Werkzeug测试应用。如果你通过一个启用了SPDY的浏览器访问https://address:8443/，那么你会看到由
`Werkzeug <http://werkzeug.pocoo.org/>`_ 报告的额外的WSGI变量：

* ``SPDY`` -- ``on``
* ``SPDY.version`` -- 协议版本 (一般是 ``3``)
* ``SPDY.stream`` -- 流标识符 (一个奇数)。

作为非root用户打开特权端口将需要使用 `shared-socket` 选项，以及一个稍微不同的语法：

.. code-block:: sh

   uwsgi --shared-socket :443 --https2 addr==0,cert=foobart.crt,key=foobar.key,spdy=1 --module werkzeug.testapp:test_app --uid user

可以同时使用HTTP和HTTPS (`=0` 和 `=1` 是由 `shared-socket` 命令打开的特权端口的引用):

.. code-block:: sh

   uwsgi --shared-socket :80 --shared-socket :443 --http =0 --https2 addr==1,cert=foobart.crt,key=foobar.key,spdy=1 --module werkzeug.testapp:test_app --uid user


注意事项
********

* 要使用SPDY，你需要至少OpenSSL 1.x (所有现代的Linux发行版都应该有它)。
* 在上传期间，窗口大小会被不断更新。
*  ``--http-timeout`` 指令被用来设置SPDY超时时间。这是在SPDY连接被关闭之后的最大数量的不活跃时间。
* 来自浏览器的 ``PING`` 请求 **全都** 被确认。
* 连接时，SPDY路由器发送一个带最优值的设置包给客户端。
* 如果流以某种灾难性方式失败了，那么整个连接会被硬关闭。
* 总是遵循 ``RST`` 消息。

待办事项
********

* 添加旧的SPDY v2支持 (值得吗？)
* 允许推送来自uWSGI缓存的资源
* 允许调整内部缓冲
