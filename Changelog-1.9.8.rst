uWSGI 1.9.8
===========

更新日志[20130423]

Note: this is an "emergency" release fixing 2 regressions causing a crash during reloads and when using async+uGreen

错误修复
********

 - fixed a crash when reloading the master
 - fixed a crash in async mode + uGreen
 - the 'mime' routing var requires a request var (not a raw string)

可用性
************

You can download uWSGi 1.9.8 from http://projects.unbit.it/downloads/uwsgi-1.9.8.tar.gz
