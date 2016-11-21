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

After literally years of bug reports and corrupted data and other general badness, `mod_proxy_uwsgi` is finally stable.

On modern Apache2 releases it supports UNIX sockets too.

Updated docs: https://uwsgi-docs.readthedocs.io/en/latest/Apache.html#mod-proxy-uwsgi

uwsgi[rsize]路由变量
************************

The new `uwsgi[rsize]` routing variable (meaningful only in the 'final' chain) exposes the response size of the request.

`callint`模式
********************

This scheme allows you to generate blobs from functions exposed by your uWSGI instance:

.. code-block:: ini

   [uwsgi]
   uid = @(callint://get_my_uid)
   gid = @(callint://get_my_gid)
   
--fastrouter-fallback-on-no-key
*******************************

The corerouter's fallback procedure requires that a valid key (domain name) has been requested.

This option forces the various routers to trigger the fallback procedure even if a key has not been found.

PHP 5.5 opcode caching via --php-sapi-name
******************************************

For mysterious reasons the PHP 5.5+'s opcode caching is not enabled in the "embed" SAPI. This option allows you to fake the SAPI name -- `apache` is a good option -- to force the opcode caching engine to turn on.

改进链式重载
************************

多亏了Marko Tiikkaja，链式重载过程正确地在cheaper模式下工作，并且更加详细。

添加'chdir'键值到--attach-daemon2
****************************************

You can now set where attached daemons need to chdir().

可用性
^^^^^^^^^^^^

uWSGI 2.0.6已于20140701发布

你可以从这里下载它：

http://projects.unbit.it/downloads/uwsgi-2.0.6.tar.gz
