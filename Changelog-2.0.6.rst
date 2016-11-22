uWSGI 2.0.6
===========

更新日志 [20140701]


错误修复
^^^^^^^^

* 修复订阅系统中的内存泄漏
* 修复ssl-socket快捷方式
* 修复Apache2 mod_proxy_uwsgi。现在，它对所有的Apache MPM引擎都是稳定的。
* 修复PHP插件中的SCRIPT_NAME和PATH_TRANSLATED生成 (感谢Matthijs Kooijman)
* 重建时将老的FIFO socket从事件队列移除 (感谢Marko Tiikkaja)


新特性
^^^^^^^^^^^^

新的Rados插件
*********************

关于作者：Marcin Deranek

Rados插件已经被改进和加固，现在对生产来说，可以算是稳定和可用的了。

异步模式和多线程正常工作。

已添加对上传对象（通过PUT）和创建新的池 (MKCOL) 的支持。

期望在uWSGI 2.1支持WebDAV。

文档已更新：https://uwsgi-docs.readthedocs.io/en/latest/Rados.html

--if-hostname
*************

这是配置逻辑，用来只有当系统的主机名匹配到一个给定的值的时候，才包含选项。

.. code-block:: ini

   [uwsgi]
   if-hostname = node1.local
     socket = /tmp/socket1.socket
   endif =
   
   if-hostname = node2.local
     socket = /var/run/foo.socket
   endif = 
   
Apache2 `mod_proxy_uwsgi` 固化
***************************************

经过真真多年的问题报告和损坏的数据，以及其他一般糟糕的事， `mod_proxy_uwsgi` 终于稳定了。

在现代的Apache2版本中，它也支持UNIX socket。

已更新文档： https://uwsgi-docs.readthedocs.io/en/latest/Apache.html#mod-proxy-uwsgi

uwsgi[rsize]路由变量
************************

新的 `uwsgi[rsize]` 路由变量 (只有在“最终”链中才有意义) 公开了请求的响应大小。

`callint`模式
********************

该模式允许你从由你的uWSGI实例公开的函数生成blob：

.. code-block:: ini

   [uwsgi]
   uid = @(callint://get_my_uid)
   gid = @(callint://get_my_gid)
   
--fastrouter-fallback-on-no-key
*******************************

这个核心路由器的回退过程要求已请求一个有效键（域名）。

这个选项强制各个路由器触发回退过程，即使尚未发现一个键。

通过--php-sapi-name进行PHP 5.5指令缓存
******************************************

出于神秘的原因，并未在“嵌入的”SAPI中启用PHP 5.5+的指令缓存。这个选项允许你伪造SAPI名 -- `apache` 是个不错的选项 -- 来强制指令缓存引擎打开。

改进链式重载
************************

多亏了Marko Tiikkaja，链式重载过程正确地在cheaper模式下工作，并且更加详细。

添加'chdir'键值到--attach-daemon2
****************************************

现在，你可以设置附加的守护进程需要chdir()到哪里。

可用性
^^^^^^^^^^^^

uWSGI 2.0.6已于20140701发布

你可以从这里下载它：

http://projects.unbit.it/downloads/uwsgi-2.0.6.tar.gz
