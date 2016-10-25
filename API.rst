uWSGI API
=============

语言插件应该公开以下API。每个语言都有自己的惯例，局限和优点。

当移植API到特定的语言时，请尽量保持该语言的风格。

这是uWSGI API公开的函数的“官方”列表，未在此处出现的函数时不规范的，因此在未来的uWSGI版本中，也许会更改它们的语义。


这里提出的语法是python的，每个语言要以它们自己特定的风格公开每个函数。

当使用not_defined的时候，意味着语言特定的NULL表示 (例如python中的None，或者perl中的undef)

uwsgi.signal(signum)
********************

引发指定的uwsgi信号

uwsgi.register_rpc(name, func, argc=0)
**************************************

将具有指定"name"的函数"func"作为RPC进行注册

argc将会强制接收参数数量

uwsgi.rpc(node, name, *args)
****************************

带提供的args（如果有的话），在服务器"node"上调用uWSGI RPC函数

如果接电视not_defined，那会调用一个“本地的”RPC

uwsgi.call(name, *args)
***********************

带提供的args（如果有的话）本地调用uWSGI RPC函数"name"

uwsgi.cache_get(key[, cache])
*****************************

从指定的"cache"中获取"key"。如果"cache"是not_defined，那么会默认使用本地缓存。

要从远程缓冲区获取一个项，那么"cache"值的语法为cachename@addr:port

如果未发现此项，或者发生了错误，那么返回not_defined

uwsgi.add_timer(signum, secs)
*****************************

注册一个每"secs"引发"signum"的定时器

uwsgi.add_rb_timer(signum, secs[, iterations])
**********************************************

注册一个每"secs"引发"signum"的红黑定时器

uwsgi.lock(num=0)
*****************

请求指定用户锁

uwsgi.unlock(num=0)
*******************

释放指定用户锁

uwsgi.masterpid()
*****************

返回master进程的当前pid

uwsgi.alarm(alarm, msg)
***********************

引发特定的"alarm"，该告警带有告警信息"msg"

uwsgi.suspend()
***************

挂起当前的异步核心，然后把CPU交给调度链中的下一个核心

uwsgi.async_sleep(secs)
***********************

将当前的异步核心挂起指定的时间(时间单位为秒)

(需要uwsgi.suspend()或者提交"yield"形式)

uwsgi.connection_fd()
*********************

返回用客户端打开的连接的文件描述符

uwsgi.async_connect(addr)
*************************

返回到指定“addr”的非堵塞连接的文件描述符

发生错误时会引发一个异常

uwsgi.wait_fd_read(fd[, timeout])
*********************************

(需要uwsgi.suspend()或者提交"yield"形式)

发生错误时会引发一个异常

uwsgi.wait_fd_write(fd[, timeout])
**********************************

(需要uwsgi.suspend()或者提交"yield"形式)

发生错误时会引发一个异常

uwsgi.ready_fd()
****************

after resume from 从suspend()恢复过来后，返回当前可用的文件描述符，或者对于超时导致的恢复，返回-1

uwsgi.send([fd,] data)
**********************

发送指定的"data"给文件描述符"fd"。

如果未指定"fd"，那么将会使用uwsgi.connection_fd()的输出

参数的位置有点奇怪，但这会使得与POSIX write()的集成更容易。

uwsgi.recv([fd,] len)
*********************

从指定的"fd"接收最多"len"字节

如果未指定"fd"，那么将会使用uwsgi.connection_fd()的输出

参数的位置有点奇怪，但这会使得与POSIX read()的集成更容易。

uwsgi.close(fd)
***************

关闭指定的文件描述符

uwsgi.setprocname(name)
***********************

设置调用进程的名字

uwsgi.add_cron(signum, minute, hour, day, month, week)
******************************************************

注册一个引发uwsgi信号"signum"的cron


uwsgi.disconnect()
******************

与客户端断连，而无需停止请求处理器


uwsgi.worker_id()
*****************

返回当前的worder in（整数）

0表示调用进程不是一个worker

uwsgi.mule_id()
*****************

返回当前的mule id（整数）

0表示调用进程不是一个mule

uwsgi.signal_registered(signum)
*******************************

检查是否注册了"signum"

返回布尔值

uwsgi.opt
*********

这是实例的所有指定选项的哈希/字典 (包括已注册的和虚拟的)

uwsgi.version
*************

uWSGI版本字符串

uwsgi.hostname
**************

服务器主机名

uwsgi.register_signal(signum, kind, handler)
********************************************

注册映射到"handler"的指定的"kind"的uwsgi信号"signum"

错误时引发一个Exception

uwsgi.set_user_harakiri(sec)
****************************

(为worker，mule和spooler)设置用户harakiri

值为0，则重置定时器
