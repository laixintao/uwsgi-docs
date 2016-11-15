uWSGI 1.9.10
============

更新日志[20130511]

错误修复
********

* 修复重载期间的告警线程
* 修复--touch-*选项中的未初始化内存
* 修复--attach-daemon中的回归

新特性
************

欢迎来到gccgo
^^^^^^^^^^^^^^^^

gcc 4.8中的Go支持是惊人的，多亏了堆栈分割特性，现在，你可以使用goroutine，而无需分配一整个pthread。

Go 1.1将不再与uWSGI兼容，而gccgo将会成为运行go应用的官方方式。

gccgo插件在开发的早期阶段，但是已经可以在preforking模式中运行了。

我们在努力地开发一个"goroutines"循环引擎。敬请关注。

最后的路由
^^^^^^^^^^^^

现在，你可以在一个请求之后运行路由规则了。显然，请求之后并非所有公开的操作都有意义，但你应该可以编写甚至更复杂的设置。

看看这个基于HTTP响应状态的请求限制器 (一个你只能在请求之后获取的值):

https://github.com/unbit/uwsgi/blob/master/t/routing/errorlimiter.ini

可用性
************

uWSGI 1.9.10于20130511发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.10.tar.gz
