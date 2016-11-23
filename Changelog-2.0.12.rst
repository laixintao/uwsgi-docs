uWSGI 2.0.12
============

[20151230]

错误修复
--------

- 'rpcvar'路由动作在空响应时正确返回NEXT
- uwsgiconfig: 修复python3中的空键处理 (Simone Basso)
- plugins/alarm_speech: 修复AppKit拼写，以支持大小写敏感的文件系统 (Andrew Janke)
- 修复继承INET地址0.0.0.0 (INADA Naoki)
- core/xmlconf: 正确初始化libxml2 (Riccardo Magliocchetti)
- python插件中传递LIBDIR给linker (Borys Pierov)
- 对于pty, forkptyrouter和mono插件的平台相关的构建修复 (Jonas Smedegaard和Riccardo Magliocchetti)

新的特性和反向移植
--------------------------

自定义worker api
*********************

你终于可以覆盖uWSGI处理模型，来获得它的完全控制权。这与你可以在诸如gunicorn（以及它与tornado或者gevent的集成）这样的项目中可以做的事情非常类似。显然，原生的插件仍然是最佳方法 (它们允许与uWSGI api和states的集成)，但在一些情况下，你也许想要使用uWSGI进程管理功能，让你的应用做剩下的工作。

目前，只有python插件支持对worker的“覆盖”，有一个aiohttp (asyncio)样例模块：

https://github.com/unbit/uwsgi-docs/blob/master/WorkerOverride.rst


--wsgi-disable-file-wrapper
***************************

这个选项禁用WSGI标准的wsgi.file_wrapper优化。在一些边缘情况中，这是避免错误的唯一技巧。

官方PHP 7支持
**********************

php插件中现在官方支持PHP 7。


uwsgi.spooler_get_task api (关于作者：Alexandre Bonnetain)
*********************************************************

这个补丁允许你轻松解析spooler文件。

看看这里的例子/测试：

https://github.com/unbit/uwsgi/blob/master/t/spooler/read.py

--if-hostname-match (关于作者：Alexandre Bonnetain)
**************************************************

这个选项用于配置逻辑，允许你只有在关于主机名的正则表达式匹配上时才定义选项：

.. code-block:: ini

   [uwsgi]
   if-hostname-match = ^prod
     threads = 20
   endif =
   


可用性
------------

你可以从http://projects.unbit.it/downloads/uwsgi-2.0.12.tar.gz下载uWSGI 2.0.12
