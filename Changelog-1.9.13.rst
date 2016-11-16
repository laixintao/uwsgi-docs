uWSGI 1.9.13
============

更新日志[20130622]

错误修复
^^^^^^^^

- 修复了当启用卸载时，没有请求插件被加载的极端情况
- 修复当存在多条规则时的harakiri路由 (返回NEXT而不是CONTINUE)
- 修复在慢DNS响应上curl崩溃的master问题 (Łukasz Mierzwa)
- 移除uwsgi.h中的PTRACE检查 (自uWSGI 1.0起不再需要它了)
- 修复--print-sym
- 在--cflags中添加一个新行
- 改进python3检测和编译
- 修复Coro::AnyEvent循环引擎 (John Berthels)
- Rack api函数现在是静态的了
- 对大文件上传更好的fastcgi处理
- 为Python非apple构建改进在Darwin的GCC使用
- 修复rawrouter中的XCLIENT使用
- 当使用CC=clang的时候，用clang预处理器代替硬编码的'cpp'
- 当请求更高的值时，设置16位选项为65535
- 修复虚拟主机 (它现在与1.4配置兼容)

新特性
^^^^^^^^^^^^

PyPy性能和功能改进
****************************************

PyPy插件已经被大大改进。C代码的数量已减少了70%，英寸，现在，该插件的绝大多数代码是用Python写的。移除了c helper，允许python部分通过cffi直接调用原生uWSGI函数。

已添加对PyPy continulets (及其greenlet抽象)的支持 (同时等待一个用于pypy的稳定的gevent移植)，并且已经有了一个聊天样例
(使用uwsgi异步api):

https://github.com/unbit/uwsgi/tree/master/t/pypy

https://github.com/unbit/uwsgi/blob/master/contrib/pypy/uwsgi_pypy_greenlets.py

改进了pypy uwsgi api，现在，你可以使用uwsgidecorators模块了 (即使spooler子系统仍然缺失)


分块输入api
*****************

在过去的日子里，已经有了一堆关于如何正确管理分块输入的讨论。由于基本上并未可用的标准以一种“明确的”方式来支持它，因此我们定义了一个低层次的api (所以，可以未来我们可以很容易的调整它)。

该api公开了两个函数：

uwsgi.chunked_read()

和

uwsgi.chunked_read_nb()

一个非阻塞的聊天例子：

.. code-block:: py

   import uwsgi
   def application(e, sr):
       while True:
           uwsgi.wait_fd_read(uwsgi.connection_fd())
           uwsgi.suspend()
           msg = uwsgi.chunked_read_nb()
           if msg: print "core %d" % e['uwsgi.core'], msg


变得更好的第三方插件管理：--dot-h选项
****************************************************************

正如--cflags选项显示将CFLAGS用来构建服务器一样，--dot-h选项显示了uwsgi.h的内容

这意味着uwsgi.h的内容现在被嵌入到二进制文件中了 (如果有压缩的话)。

这可能看起来是个奇怪的选择，但是其目的是使得从uwsgi源代码中编译出插件变得容易
(在2.0中，肯定会有一些会能用的)

setmethod, seturi 和setpathinfo路由动作
************************************************

我们继续扩展路由API

添加了3个新的动作来动态修改请求

UWSGI_INCLUDES
**************

你现在可以用这个环境变量来覆盖include搜索路径了 (在构建uWSGI的时候)。

改进的set_user_harakiri api函数
***************************************

现在，uwsgi.set_user_harakiri自动识别mule和spooler。已将它添加到了ruby/rack, pypy和perl/psgi插件中

--add-cache-item [cache ]KEY=VALUE
**********************************

这是一个商品选项(commodity option) (主要用于测试)，允许你在启动的时候将一个项存储到uWSGI缓存中

router_xmldir插件
************************

这是旨在强调转换api的概念插件的证明。

它基本上生成目录的一个xml表示。这对于实现apache式的目录索引可能有用：

看看这个使用xslt的例子：

https://github.com/unbit/uwsgi/issues/271#issuecomment-19820204

为@spool*装饰器实现 __call__
*****************************************

多亏了'anaconda'，你现在可以直接调用映射到spooler的函数了，因此，取代

.. code-block:: py

    myfunc.spool(args)
    
你可以直接这样：

.. code-block:: py

    myfunc(args)
    
显然还是支持旧的方式的

uwsgi[lq]路由变量
*************************

这个路由变量表示listen_queue的当前大小：

.. code-block:: ini

   [uwsgi]
   ...
   route-if = higher:${uwsgi[lq]};70 break:503 Server Overload
   ...

--use-abort
***********

在一些系统上，SEGV信号处理器在uWSGI回溯后不能被正确地恢复。

如果你想生成一个core文件，那么你或许想要在回溯之后立即触发SIGABRT。

可用性
^^^^^^^^^^^^

uWSGI 1.9.13于2013年06月22日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.13.tar.gz
