uWSGI 2.0.11
============

[20150701]

错误修复
********

- [pypy] 修复ffi.string的错误使用
- 修复gcc 5检测 (jimfunk)
- 修复用于网关的共享socket
- [psgi] 将abs更改为labs，因为offset被声明为long类型 (Peter H. Ezetta)
- 添加null结束符号到uwsgi_get_dot_h()和uwsgi_config_py() (Jay Oster)
- 修复停止/重启期间的线程等待 (Kaiwen Xu)
- 修复链式重载信息显示
- [python] 修复spooler任务引用计数 (Curtis Maloney)
- 多种静态分析改进 (Riccardo Magliocchetti)
- 修复对于非常大范围的共享区域支持
- 修复大小为0的响应的gzip转换 (Curtis Maloney)
- 修复https客户端证书认证管理 (Vladimir Didenko)
- 修复OpenBSD构建
- 修复TMPFILE权限


新特性
************

mem_collector线程
^^^^^^^^^^^^^^^^^^^^^^^^

Evil内存监控器 (例如--evil-reload-on-rss) 现在是由专有线程异步管理。

这解决了失控进程不被master捕获到的问题。

fixpathinfo路由动作
^^^^^^^^^^^^^^^^^^^^^^^^^^

这是移除臭名昭著的uwsgi_modifier1 30遗留物需要的另一步。

这个路由动作假设PATH_INFO cgi变量包含了SCRIPT_NAME部分。

这个动作允许你在nginx中设置SCRIPT_NAME，而无需重写PATH_INFO (这是nginx无法承担的)

.. code-block:: ini

   [uwsgi]
   ; blindly assumes PATH_INFO is clobbered with SCRIPT_NAME
   route-run = fixpathinfo:

uwsgi[sor]和time[micros]路由变量
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这两个新的变量公开了当前请求的开始时间 (以微秒为单位) 和当前时间 (再次，以微秒为单位)

.. code-block:: ini

   [uwsgi]
   route-run = log:request started at ${uwsgi[sor]}
   route-run = log:current micros time is ${time[micros]}

wait-for-socket
^^^^^^^^^^^^^^^

这与wait-for-fs/iface/file/dir类似

挂起实例的生成，直到特定的tcp/unix socket准备好了。

你可以用它来同步vassal生成 (例如，停止一个vassal，直到已生成一个postgresql服务器)

wait_for hooks
^^^^^^^^^^^^^^

所有的wait-for-*函数现在可以当成钩子使用：

.. code-block:: ini

   [uwsgi]
   ; wait for postgres soon after privileges drop
   hook-as-user = wait_for_socket:127.0.0.1:5432

可用性
************

你可以从这里下载uWSGI 2.0.11：

http://projects.unbit.it/downloads/uwsgi-2.0.11.tar.gz
