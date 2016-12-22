uWSGI 2.0.9
===========

[20141230]

错误修复
--------

* 修复非阻塞模式下的mod_proxy_uwsgi (非常感谢Joe cuchac)
* 修复master-fifo + cheaper
* 修复bind_to_unix上的错误时泄漏 (Riccardo Magliocchetti)
* atexit钩子也可以工作在cheaped worker上
* atexit钩子在关闭期间也可以工作在gevent模式上
* 修复重载后的carbon命令行选项
* 第一次运行时不遵循Emperor节流
* 修复Mono插件
* 修复corerouter中的对端名
* 修复守护进程的停止信号
* https/spdy路由器中的多个ssl/tls修复
* 修复python3 --py-auto-reload-ignore
* 修复corerouter中的modifier
* 支持yajl (OSX)
* psgi: 确保我们在psgix.harakiri.commit上调用任何DESTROY钩子 (Ævar Arnfjörð Bjarmason)
* systemdlogger: 使用-Werror=format-security修复编译 (Riccardo Magliocchetti)
* 修复无掩码websockets
* perl修复潜在的refcounting问题 (Mattia Barbon)

新特性
********

改进Linux的PyPy支持
*******************************

PyPy团队已开始在他们的官方版本中构建libpypy-c.so了。现在，通过uWSGI使用pypy应该更容易了：

https://uwsgi-docs.readthedocs.io/en/latest/PyPy.html

Fastrouter post-buffering
*************************

fastrouter拥有post-buffering功能：

https://uwsgi-docs.readthedocs.io/en/latest/Fastrouter.html#post-buffering-mode-uwsgi-2-0-9

Perl uwsgi::opt
***************

psgi/perl插件公开了uwsgi::opt哈希值，报告整个实例的键值配置

--pull-header
*************

这像--collect-header，但并不返回已收集的头给客户端

active-workers信号目标
****************************

这就像'workers'目标，但是只将信号转发给非cheaper的worker

httpdumb路由动作
***********************

http内部路由器公开了一个名为'httpdumb'的新模式，此模式下，在转发请求之前不修改请求头

可用性
------------

uWSGI 2.0.9已于20141230发布。

你可以从这里下载它：

http://projects.unbit.it/downloads/uwsgi-2.0.9.tar.gz
