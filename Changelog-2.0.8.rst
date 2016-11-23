uWSGI 2.0.8
===========

注意：这是第一个默认禁用SSL3的版本，如果你需要它，可以通过 ``--ssl-enable3`` 选项来重新启用

错误修复
--------

* 修复当 ``--php-app`` 存在时的PHP `SCRIPT_NAME` 使用
* 允许不带第二个参数的"appendn"钩子
* 修复Carbon插件中的堆损坏 (关于作者：Nigel Heron)
* 修复 `getifaddrs()` 内存管理
* 修复 `tcsetattr()` 使用
* 修复返回值的kevent使用 (关于作者：Adriano Di Luzio)
* 确保PSGI响应头格式正确
* 修复附加守护进程的重载
* 修复SSL/TLS关闭
* 修复不以/结尾的路径的挂载逻辑 (关于作者： Adriano Di Luzio)
* 修复spooler装饰器的Python3支持 (关于作者：Adriano Di Luzio)

新特性
********

从2.1移植过来的RTSP和分块输入，用于HTTP路由器
*************************************************************

 ``--http-manage-rtsp`` 和 ``--http-chunked-input` 已从2.1移植过来，允许HTTP路由器自动检测RTSP和分块请求。这对于即将到来的https://github.com/unbit/uwsgi-realtime插件有用。

--hook-post-fork
****************

这个自定义钩子允许你在每次 `fork()` 之后调用动作。

对asyncio插件，回退到trollius
***************************************

如果你对python2构建asyncio插件，那么将会尝试回退到 `trollius <https://trollius.readthedocs.io/>`_ 模块。

这个特性基本零测试覆盖，因此欢迎每个报告 (错误或者成功都可以)。

添加sweep_on_full, clear_on_full 和 no_expire 到 ``--cache2``
****************************************************************

已添加3个用于 ``--cache2`` 的新选项，用来改进缓存过期策略：

* ``sweep_on_full`` 一旦缓存变满，就进行扫除 (删除所有过期项)
* ``clear_on_full`` 一旦缓存满了，就完全清理缓存
* ``no_expire`` 强制缓存不要生成缓存扫除线程，委托项移除给前两个选项

从2.1移植wait-for-fs/mountpoints
*******************************************

* ``--wait-for-fs <path>`` 挂起uWSGI启动，直到文件/目录可用
* ``--wait-for-file <path>`` 挂起uWSGI启动，直到文件可用
* ``--wait-for-dir <path>`` 挂起uWSGI启动，直到目录可用
* ``--wait-for-mountpoint <path>`` 挂起uWSGI启动，直到挂载点可用

改进卸载api (backport from 2.1)
********************************************

uWSGI 2.0.8与即将到来的https://github.com/unbit/uwsgi-realtime插件兼容，该插件使用uWSGI卸载引擎 + Redis发布/订阅（publish/subscribe），允许实时特性
(例如websockets或者音频/视频流)的使用。

允许将来自远程源的插件作为嵌入插件构建
*******************************************************

已扩展UWSGI_EMBED_PLUGINS环境变量，支持远程插件。例如，你可以使用Avahi和实时插件，这样构建一个单片uwsgi二进制文件：

.. code-block:: sh

   UWSGI_EMBED_PLUGINS="avahi=https://github.com/20tab/uwsgi-avahi,realtime=https://github.com/unbit/uwsgi-realtime" make

自动管理HTTP_X_FORWARDED_PROTO
*******************************************

虽然在HTTP世界里，对于转发会话有了一个新的标准 (http://tools.ietf.org/html/rfc7239)，但是这个版本添加了对X-Forwarded-Proto头的支持，自动检测相应的请求模式(scheme)。

可用性
------------

uWSGI 2.0.8已于20141026发布。在这下载它：

http://projects.unbit.it/downloads/uwsgi-2.0.8.tar.gz
