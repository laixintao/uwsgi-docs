uWSGI 2.0.7
===========

更新日志 [20140905]

错误修复
********

- 修复Statsd插件中的计数器 (Joshua C. Forest)
- 修复PHP插件中的缓存 (Andrew Bevitt)
- 修复对以数字开头的系统用户的管理
- 使用 `memmove` 来代替 `memcpy` 进行请求体readline(Andrew Wason)
- 忽略 `setns` 中的“user”名字空间 (仍然是问题源)
- 修复Python3 RPC 字节/字符串混乱 (结果：我们两种都支持)
- 当挂载钩子失败时不销毁Emperor
- 修复OS X上的Mono插件中的符号查找错误 (Ventero)
- 修复缓冲区溢出时的FastCGI和SCGI协议错误
- 修复Solaris/SmartOS I/O管理
- 修复RPC子系统中的两种内存泄漏 (Riccardo Magliocchetti)
- 修复Rados插件的PUT方法 (Martin Mlynář)
- 修复cow模式下使用多个线程的多个Python挂载点
- 统计信息UNIX socket现在由 `vacuum` 删除
- 修复缓存LRU模式中的离一(off-by-one)崩溃
- 强制Cygwin中的单CPU构建 (Guido Notari)

新特性和改进
*****************************

允许从每个CPython上下文中调用spooler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在Europython 2014, Ultrabug (一个uWSGI贡献者和packager) 要求直接从greenlet spool任务的可能。

已完成。

store_delete cache2选项
^^^^^^^^^^^^^^^^^^^^^^^^^^

作者：goir

`--cache2` 选项的 `store_delete` 标识允许你强制缓存引擎自动移除无效的备份存储文件，而不是坚定拒绝启动。

文件日志器循环
^^^^^^^^^^^^^^^^^^^^

作者：Riccardo Magliocchetti

 `file` 日志器已扩展，允许使用循环 (非插件式 `--logto` 使用相同的系统)。

https://github.com/unbit/uwsgi/commit/0324e5965c360dccfb873ffe351dec88ddab59c5

Vassal插件钩子
^^^^^^^^^^^^^^^^^^^

这个插件钩子API已扩展了两个新的钩子： `vassal` 和 `vassal_before_exec` 。

它们允许在vassal进程已创建后立即对vassal自定义。

第一个使用它的第三方插件是'apparmor': https://github.com/unbit/uwsgi-apparmor

这允许你应用Apparmor配置文件到vassal。


Broodlord改进
^^^^^^^^^^^^^^^^^^^^^^

已改进Broodlord子系统，它有一个新选项： `--vassal-sos` ，当实例的所有worker都处于忙碌状态时，自动要求加固。

除此之外，系统管理员现在可以手动要求加固，发送'B'命令到一个实例的master FIFO。

可用性
************

uWSGI 2.0.7已于20140905发布，现在你可以从下面的地址下载它

http://projects.unbit.it/downloads/uwsgi-2.0.7.tar.gz
