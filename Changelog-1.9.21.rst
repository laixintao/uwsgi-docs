uWSGI 1.9.21
============

2.0(定于2013年12月30日)之前的最后一个1.9版本

从现在起，所有的版本都将-rc (将不会增加新功能)

一个描述从 (非常过时) 1.2 和 1.4版本起升级的注意事项的文档正在进行中。

这个版本包括一个直接嵌入到uWSGI二进制文件中的新的简化插件构建器子系统。

一个报告第三方插件的页面在这里: :doc:`ThirdPartyPlugins` (请随意添加你的插件)

然后现在……

更新日志 [20131211]

错误修复
********

- croak if the psgi streamer fails
- 允许在raspberrypi上构建coroae
- do not wait for write availability until strictly required
- avoid segfault when async mode api is called without async mode
- fixed plain (without suspend engine) async mode
- do not spit errors on non x86 timerfd_create
- 支持 __arm__ 上的timerfd_create/timerfd_settime

优化
*************

用于第一个块的writev()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inernally when the first response body is sent, uWSGI check if response headers have been sent too, and eventually send them with an additional write() call.

This new optimizations allows uWSGI to send both headers and the first body chunk with single writev() syscall.

If the writev() returns with an incomplete write on the second vector, the system will fallback to simple write().

use a single buffer for websockets outgoing packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before this patch every single websocket packet required to allocate a memory chunk.

This patch forces the reuse of a single dynamic buffer. For games this should result in a pretty good improvement in responsiveness.

新特性
********

已移除zeromq api
^^^^^^^^^^^^^^^^^^

The zeromq api (a single function indeed) has been removed. Each plugin rquiring zeromq cam simply call zmq_init() insteadd of uwsgi_zeromq_init().

The mongrel2 support has been moved to a 'mongrel2' plugin.

To pair uWSGI with mongrel2 the same options as before can be used, just remember to load (and build) the mongrel2 plugin

The new sharedarea
^^^^^^^^^^^^^^^^^^

The shared area subsystem has been rewritten (it is incompatible with the old api as it requires a new argument as it now supports multiple memory areas).

Check updated docs: :doc:`SharedArea`

report request data in writers and readers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

every error when reading and writing to/from the client will report current request's data.

This should simplify debugging a lot.

Modular logchunks management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The uWSGI api has been extended to allow plugins to define their log-request vars.

Check: :doc:`LogFormat`

tmsecs and tmicros, werr, rerr, ioerr, var.XXX
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

6 new request logging variables are available:

tmsecs: report the current unix time in milliseconds

tmicros: report the current unix time in microseconds

werr: report the number of write errors for the current request

rerr: report the number of read errors for the current request

ioerr: the sum of werr and rerr

var.XXX: report the context of the request var XXX (like var.PATH_INFO)

mountpoints and mules support for symcall
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The symcall plugin has been improved to support mules and mountpoints.

To run a C function in a mule just specify it as ``--mule=foobar()`` when the mule finds an argument ending
with () it will consider it a function symbol.

read2 and wait_milliseconds async hooks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This two non-blocking hooks adds new capabilities to the non-blocking system.

The first one allows to wait on two file descriptors with the same call (currently implemented only in plain async mode)

The second one is used to have a millisecond resolution sleep. (this is currently used only by the sharedarea waiting system)

websockets binary messages
^^^^^^^^^^^^^^^^^^^^^^^^^^

You can now send websocket binary message. Just use ``uwsgi.websocket_send_binary()`` instead of ``uwsgi.websocket_send()``

the 'S' master fifo command
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sending 'S' to the master fifo, enable/disable the sending of subscription packets

as-mule hook
^^^^^^^^^^^^

this new custom hooks allows you to execute custom code in every mule:

.. code-block:: ini

   [uwsgi]
   hook-as-mule = exec:myscript.sh
   ...


accepting hook and improved chain reloading
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The chain reloading subsystem has been improved to take in account when a worker is really ready to accept() requests.

This specific state is announced to the Emperor too.

Check this article for more infos: https://uwsgi-docs.readthedocs.io/en/latest/articles/TheArtOfGracefulReloading.html

--after-request-call
^^^^^^^^^^^^^^^^^^^^

this option allows you to call specific C functions (in chains) after each request. While you should use the framework/interface features for this kind of job, sometimes it is not possible to execute
code after the logging phase. In such a case feel free to abuse this option.

错误页面
^^^^^^^^^^^

3个新的选项允许你自定义错误页面 (仅限html):

``--error-page-403 <file>``                     add an error page (html) for managed 403 response

``--error-page-404 <file>``                     add an error page (html) for managed 404 response

``--error-page-500 <file>``                     add an error page (html) for managed 500 response

简化插件构建器
^^^^^^^^^^^^^^^^^^^^^^^^^^

构建uWSGI插件现在超级简单：

.. code-block:: sh

   uwsgi --build-plugin <directory>
   
this option will create a sane environment based on the current binary (no need to fight with build profiles and #ifdef) and will build the plugin.

No external files (included uwsgi.h) are needed as the uWSGI binary embeds them.


2.0任务清单
************

- implement websockets and sharedarea support in Lua
- complete sharedarea api for CPython, Perl, Ruby and PyPy
- implement read2 and wait_milliseconds hook in all of the available loop engines

可用性
************

uWSGI 1.9.21于2013年12月11日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.21.tar.gz
