uWSGI 2.0.2
===========


更新日志 20140226


错误修复
--------

* 修复在较老的编译器/libc上的python3支持
* 允许以仅spooler模式启动
* 修复缓存bitmap支持，并添加测试套件 (关于作者：Danila Shtan)
* 修复ftime日志变量
* 添加异步远程信号管理
* 修复end-for和end-if
* 修复内部路由响应链中的循环
* 修复pypy execute_source使用
* logpipe: 不setsid()两次 (关于作者：INADA Naoki)

新特性和改进
-----------------------------

CGI插件
**********

已改进此插件以支持流式传输。

除此之外，long-awaited异步支持最终准备好了。现在，你可以拥有CGI并发，而无需生成许许多多昂贵的线程/进程

看看： :doc:`CGI`

PSGI加载改进
*************************

PSGI加载器现在尝试使用Plack::Util::load_psgi()函数，而不是简单的eval。这解决了环境中各种不一致 (例如psgi脚本的双次解析／编译／执行)。

如果Plack模块不可用，那么会使用一个简单的基于do的代码 (非常类似于load_psgi)

非常感谢booking.com的Ævar Arnfjörð Bjarmason，他发现了这个问题


可用性
************

你可以从以下地址下载uWSGI 2.0.2：
http://projects.unbit.it/downloads/uwsgi-2.0.2.tar.gz




