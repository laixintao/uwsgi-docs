管理标志
================

.. warning:: 这个特性当前可能已暂停或弃用。

使用管理标志（Management Flag）系统，你可以远程修改uWSGI栈的某些方面的行为，而无需下线服务器。

.. note:: 一个更综合的重设置系统可能正在开发中。

所有的标志都是用一个无符号的32位值（因此块大小总是4），它包含为这个标志设置的值。如果你不指定该值，那么仅发送uWSGI头，服务型将会把它当成一种读请求。


====    =============== ===========
标志    动作            描述
====    =============== ===========
0       logging         启用/禁用日志记录
1       max_requests    设置每个worker的最大请求数
2       socket_timeout  修改内部socket超时
3       memory_debug    启用/禁用内存调试/报告
4       master_interval 设置master进程检查间隔
5       harakiri        设置/取消设置harakiri超时
6       cgi_mode        启用/禁用cgi模式
7       threads         启用/禁用线程（当前未实现）
8       reaper          启用/禁用进程reaper   
9       log-zero        启用/禁用记录响应大小为零的请求
10      log-slow        设置/取消设置记录缓慢请求
11      log-4xx         启用/禁用记录4xx响应状态的请求
12      log-5xx         启用/禁用记录5xx响应状态的请求
13      log-big         设置/取消设置记录具有大的响应大小的请求
14      log-sendfile    设置/取消设置记录sendfile请求
15      backlog-status  报告backlog队列的当前大小（Linux仅限tcp）    
16      backlog-errors  报告backlog队列中的错误数（Linux仅限tcp）   
====    =============== ===========

myadmin工具
------------

包含一个简单（丑陋）的脚本， ``myadmin`` ，用来远程更改管理标志：

.. code-block:: sh

    # disable logging on the uWSGI server listening on 192.168.173.17 port 3031
    ./uwsgi --no-server -w myadmin --pyargv "192.168.173.17:3031 0 0"
    # re-enable logging
    ./uwsgi --no-server -w myadmin --pyargv "192.168.173.17:3031 0 1"
    # read a value:
    ./uwsgi --no-server -w myadmin --pyargv "192.168.173.17:3031 15"
