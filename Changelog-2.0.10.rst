uWSGI 2.0.10
============

[20150317]

错误修复
--------

* 不因gcc 4.9的使用而降低安全标准 (Riccardo Magliocchetti)
* Perl/PSGI确保至少有两个参数传递给xs_input_seek (Ivan Kruglov)
* Per/PSGI修复多个解析器使用
* spooler: 修复scandir使用
* 修复异常处理器参数管理
* 修复'log-master' + 'daemonize2'禁用所有日志记录
* 修复http Range头管理


新特性
********

safeexec钩子
**************

这类似'exec'，但不会在错误时退出，即使执行的命令返回一个非零值

移植--emperor-wrapper-fallback和--emperor-wrapper-override
********************************************************************

--emperor-wrapper-fallback选项允许你在运行vassal并且找不到默认的binary_path（或者返回错误）时，指定一个替代二进制文件来执行。 (你可以多次指定它)

--emperor-wrapper-override类似，但是“覆盖”默认的封装器 (你可以多次指定它)

添加对UNIX socket的支持到rsyslog
*****************************************

此rsyslog记录器现在可以把一个unix socket当成地址 (以斜线开头的参数会被识别成一个unix路径)

forcecl转换
**********************

这个转换就像'fixcl'，但它生成Content-Length头，即使Content-Length已处于删除列表中。


可用性
------------

你可以在这里下载uWSGI 2.0.10：

http://projects.unbit.it/downloads/uwsgi-2.0.10.tar.gz
