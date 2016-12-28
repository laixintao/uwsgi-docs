uWSGI选项
^^^^^^^^^^^^^

这是一份自动生成的uWSGI选项参考列表。

它与你可以通过 ``--help`` 选项获得的输出相同。

对于新手而言，本页可能是了解uWSGI最糟糕的方法。如果你还在学习该项目的工作方式，那么你应该阅读各种快速入门和教程。

每一个选项都有以下属性:

* argument: 它是结构选项 (由getopt()/getopt_long()使用) has_arg元素。可以是'必须的', '无参数'或者'可选参数'
* shortcut: 有些选项可以用短形式指定 (一个破折号，后面跟着单个字母)
* parser: 这是uWSGI如何解析参数。有几十种方式，最常见的方式是'uwsgi_opt_set_str' 何时接收简单的字符串，'uwsgi_opt_set_int' 何时接收32位数字，'uwsgi_opt_add_string_list' 何时可以指定该参数多次来构建一个列表。
* help: 帮助消息，与你从 ``uwsgi --help`` 获得的相同
* reference: 一个到让你更好理解并且提供选项上下文的文档页面

你可以添加更多详细信息到这个页面上，编辑https://github.com/unbit/uwsgi-docs/blob/master/optdefs.pl (拜托，在发送一个pull请求之前，再三检查)

uWSGI核心
==========
socket
******
``argument``: 必需参数

``shortcut``: -s

``parser``: uwsgi_opt_add_socket

``help``: 使用默认协议绑定到指定UNIX/TCP socket上



uwsgi-socket
************
``argument``: 必需参数

``shortcut``: -s

``parser``: uwsgi_opt_add_socket

``help``: 使用uwsgi协议绑定到指定UNIX/TCP socket上



suwsgi-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_ssl_socket

``help``: 通过SSL使用uwsgi协议绑定到指定UNIX/TCP socket上



ssl-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_ssl_socket

``help``: 使用SSL之上的uwsgi协议绑定到指定的UNIX/TCP socket



http-socket
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用HTTP协议绑定到指定UNIX/TCP socket上



http-socket-modifier1
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用HTTP协议时，强制指定的modifier1



http-socket-modifier2
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用HTTP协议时，强制指定的modifier2



http11-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用HTTP 1.1 (Keep-Alive)协议绑定到指定UNIX/TCP socket上



https-socket
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_ssl_socket

``help``: 使用HTTPS协议绑定到指定UNIX/TCP socket上



https-socket-modifier1
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用HTTPS协议时，强制指定的modifier1



https-socket-modifier2
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用HTTPS协议时，强制指定的modifier2



fastcgi-socket
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用FastCGI协议绑定到指定UNIX/TCP socket上



fastcgi-nph-socket
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用FastCGI协议绑定到指定UNIX/TCP socket上 (nph模式)



fastcgi-modifier1
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用FastCGI协议时，强制指定的modifier1



fastcgi-modifier2
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用FastCGI协议时，强制指定的modifier2



scgi-socket
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用SCGI协议绑定到指定的UNIX/TCP socket



scgi-nph-socket
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用SCGI协议绑定到指定UNIX/TCP socket上 (nph模式)



scgi-modifier1
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用SCGI协议时，强制指定的modifier1



scgi-modifier2
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用SCGI协议时，强制指定的modifier2



raw-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket_no_defer

``help``: 使用RAW协议绑定到指定UNIX/TCP socket上



raw-modifier1
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用RAW协议时，强制指定的modifier1



raw-modifier2
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 当使用RAW协议时，强制指定的modifier2



puwsgi-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_socket

``help``: 使用持久性uwsgi协议(puwsgi)绑定到指定的UNIX/TCP socket上



protocol
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 对默认socket强制使用指定的协议



socket-protocol
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 对默认socket强制使用指定的协议



shared-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_shared_socket

``help``: 为高级jailing或者ipc创建一个共享socket



undeferred-shared-socket
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_shared_socket

``help``: 为高级jailing或者ipc创建一个共享socket (undeferred模式)



processes
*********
``argument``: 必需参数

``shortcut``: -p

``parser``: uwsgi_opt_set_int

``help``: 生成指定数目的worker/进程



workers
*******
``argument``: 必需参数

``shortcut``: -p

``parser``: uwsgi_opt_set_int

``help``: 生成指定数目的worker/进程



thunder-lock
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 序列化accept()使用 (如果可能的话)

``reference``: :doc:`articles/SerializingAccept`



harakiri
********
``argument``: 必需参数

``shortcut``: -t

``parser``: uwsgi_opt_set_int

``help``: 设置harakiri超时



harakiri-verbose
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 对harakiri启用verbose模式



harakiri-no-arh
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在after-request-hook期间不启用harakiri



no-harakiri-arh
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在after-request-hook期间不启用harakiri



no-harakiri-after-req-hook
**************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在after-request-hook期间不启用harakiri



backtrace-depth
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置回溯深度



mule-harakiri
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为mule任务设置harakiri超时时间



xmlconfig
*********
``argument``: 必需参数

``shortcut``: -x

``parser``: uwsgi_opt_load_xml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从xml文件加载配置



xml
***
``argument``: 必需参数

``shortcut``: -x

``parser``: uwsgi_opt_load_xml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从xml文件加载配置



config
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_config

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 使用可插拔系统加载配置



fallback-config
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 当退出码为1的时候，使用指定的配置重新执行uwsgi



strict
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 启用strict模式 (不能使用占位符)



skip-zero
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 跳过对文件描述符0的检查



skip-atexit
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 跳过atexit钩子 (由master忽略)



set
***
``argument``: 必需参数

``shortcut``: -S

``parser``: uwsgi_opt_set_placeholder

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置一个占位符或者选项



set-placeholder
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_placeholder

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置一个占位符



set-ph
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_placeholder

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置一个占位符



get
***
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_NO_INITIAL

``help``: 打印指定的选项值并退出



declare-option
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_custom_option

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 声明一个新的uWSGI自定义选项

``reference``: :doc:`CustomOptions`



declare-option2
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_custom_option

``help``: 声明一个新的uWSGI自定义选项 (非立即)



resolve
*******
``argument``: 必需参数

``parser``: uwsgi_opt_resolve

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 把dns查询的结果放到指定的占位符上，语法：placeholder=name (立即选项)



for
***
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) for循环



for-glob
********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) for循环 (扩展glob)



for-times
*********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) for循环 (扩展指定的数字到一个从1开始的列表)



for-readline
************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) for循环 (扩展指定文件到一个行列表)



endfor
******
``argument``: 可选参数

``parser``: uwsgi_opt_noop

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) end for循环



end-for
*******
``argument``: 可选参数

``parser``: uwsgi_opt_noop

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) end for循环



if-opt
******
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查选项



if-not-opt
**********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查选项



if-env
******
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查环境变量



if-not-env
**********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查环境变量



ifenv
*****
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查环境变量



if-reload
*********
``argument``: 无参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查重载



if-not-reload
*************
``argument``: 无参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查重载



if-hostname
***********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查主机名



if-not-hostname
***************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查主机名



if-hostname-match
***********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 尝试匹配主机名到一个正则表达式



if-not-hostname-match
***************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 尝试匹配主机名到一个正则表达式



if-exists
*********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查文件/目录存在性



if-not-exists
*************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查文件/目录存在性



ifexists
********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查文件/目录存在性



if-plugin
*********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查插件



if-not-plugin
*************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查插件



ifplugin
********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查插件



if-file
*******
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查文件存在性



if-not-file
***********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查文件存在性



if-dir
******
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查目录存在性



if-not-dir
**********
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查目录存在性



ifdir
*****
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查目录存在性



if-directory
************
``argument``: 必需参数

``parser``: uwsgi_opt_logic

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) 检查目录存在性



endif
*****
``argument``: 可选参数

``parser``: uwsgi_opt_noop

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) end if



end-if
******
``argument``: 可选参数

``parser``: uwsgi_opt_noop

``flags``: UWSGI_OPT_IMMEDIATE

``help``: (选择逻辑) end if



blacklist
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置选项黑名单上下文



end-blacklist
*************
``argument``: 无参数

``parser``: uwsgi_opt_set_null

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 清除选项黑名单上下文



whitelist
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置选项白名单上下文



end-whitelist
*************
``argument``: 无参数

``parser``: uwsgi_opt_set_null

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 清除选项白名单上下文



ignore-sigpipe
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不报告 (烦人的) SIGPIPE



ignore-write-errors
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不报告 (烦人的) write()/writev()错误



write-errors-tolerance
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置允许写错误的最大数 (默认：不容忍)



write-errors-exception-only
***************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 写错误时只引发一个异常，把控制权交给应用自身



disable-write-exception
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用write()/writev()上的异常生成



inherit
*******
``argument``: 必需参数

``parser``: uwsgi_opt_load

``help``: 使用指定文件作为配置模板



include
*******
``argument``: 必需参数

``parser``: uwsgi_opt_load

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 包含指定的文件作为立即配置



inject-before
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在配置文件之前注入一个文本文件 (高级模板)



inject-after
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在配置文件之后注入一个文本文件 (高级模板)



daemonize
*********
``argument``: 必需参数

``shortcut``: -d

``parser``: uwsgi_opt_set_str

``help``: 守护uWSGI



daemonize2
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 应用加载后守护uWSGI



stop
****
``argument``: 必需参数

``parser``: uwsgi_opt_pidfile_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 停止一个实例



reload
******
``argument``: 必需参数

``parser``: uwsgi_opt_pidfile_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 重载一个实例



pause
*****
``argument``: 必需参数

``parser``: uwsgi_opt_pidfile_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 暂停一个实例



suspend
*******
``argument``: 必需参数

``parser``: uwsgi_opt_pidfile_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 挂起一个实例



resume
******
``argument``: 必需参数

``parser``: uwsgi_opt_pidfile_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 恢复一个实例



connect-and-read
****************
``argument``: 必需参数

``parser``: uwsgi_opt_connect_and_read

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 连接到一个socket，并且等待来自它的数据



extract
*******
``argument``: 必需参数

``parser``: uwsgi_opt_extract

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 抽取/转储任何支持的地址到标准输出



listen
******
``argument``: 必需参数

``shortcut``: -l

``parser``: uwsgi_opt_set_int

``help``: 设置socket监听队列大小



max-vars
********
``argument``: 必需参数

``shortcut``: -v

``parser``: uwsgi_opt_max_vars

``help``: 设置内部iovec/vars结构的数量



max-apps
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置per-worker应用的最大数目



buffer-size
***********
``argument``: 必需参数

``shortcut``: -b

``parser``: uwsgi_opt_set_64bit

``help``: 设置内部缓冲大小



设置请求的最大大小 (排除request-body)，这一般映射到请求头的大小。默认情况下，它是4k。如果你接收到了一个更大的请求 (例如，带有大cookies或者查询字符串)，那么你也许需要增加它。它也是一个安全度量，所以调整为你的应用需要，而不是最大输出。

memory-report
*************
``argument``: 无参数

``shortcut``: -m

``parser``: uwsgi_opt_true

``help``: 启用内存报告



profiler
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 启用指定的profiler



cgi-mode
********
``argument``: 无参数

``shortcut``: -c

``parser``: uwsgi_opt_true

``help``: 对支持CGI模式的插件，强制使用CGI模式



abstract-socket
***************
``argument``: 无参数

``shortcut``: -a

``parser``: uwsgi_opt_true

``help``: 强制在abstract中使用UNIX socket (仅Linux)



chmod-socket
************
``argument``: 可选参数

``shortcut``: -C

``parser``: uwsgi_opt_chmod_socket

``help``: chmod-socket



chmod
*****
``argument``: 可选参数

``shortcut``: -C

``parser``: uwsgi_opt_chmod_socket

``help``: chmod-socket



chown-socket
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 修改unix socket的所有者



umask
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_umask

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置umask



freebind
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 把socket置于freebind模式



为每个由uWSGI创建的socket设置IP_FREEBIND标识。这类socket可以绑定到不存在的ip地址上。它的主要目的是为了高可用性 (仅Linux)

map-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 映射socket到指定的worker



enable-threads
**************
``argument``: 无参数

``shortcut``: -T

``parser``: uwsgi_opt_true

``help``: 启用线程



no-threads-wait
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在退出/重载的时候，不等待线程取消



auto-procname
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 自动设置进程名到某些有意义的名字



procname-prefix
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_PROCNAME

``help``: 添加一个前缀到进程名



procname-prefix-spaced
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str_spaced

``flags``: UWSGI_OPT_PROCNAME

``help``: 添加一个空间前缀到进程名



procname-append
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_PROCNAME

``help``: 附加一个字符串到进程名



procname
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_PROCNAME

``help``: 设置进程名



procname-master
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_PROCNAME

``help``: 设置master进程名



single-interpreter
******************
``argument``: 无参数

``shortcut``: -i

``parser``: uwsgi_opt_true

``help``: 不要使用多个解释器 (如果可用)



need-app
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果没有可以加载的应用，则退出



master
******
``argument``: 无参数

``shortcut``: -M

``parser``: uwsgi_opt_true

``help``: 启用master进程



honour-stdin
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不要重新映射标准输入到/dev/null



emperor
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 运行Emperor

``reference``: :doc:`Emperor`



Emperor是一个特殊的uWSGI实例，旨在管理其他uWSGI进程 (即：vassal)。默认情况下，配置它来监控一个包含有效uWSGI配置文件的目录，每当创建了一个文件，就会生成一个新的实例，当动了这个文件，相应的实例就会被重载，当这个文件被移除，相应的实例就会被销毁。可以扩展以支持更多的范例

emperor-proxy-socket
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 强制vassal成为一个Emperor代理



emperor-wrapper
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为vassal设置一个二进制封装器



emperor-nofollow
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 当检查mtime的时候，不遵循符号链接



emperor-procname
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置Emperor进程名



emperor-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置Emperor扫描频率 (默认是3秒)



emperor-required-heartbeat
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置Emperor对心跳的容忍度



emperor-curse-tolerance
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置Emperor对于被诅咒的vassal的容忍



emperor-pidfile
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 将Emperor pid写入到指定文件中



emperor-tyrant
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 将Emperor置于Tyrant模式



emperor-tyrant-nofollow
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 当在Tyrant模式中检查uid/gid时不要遵循符号链接



emperor-tyrant-initgroups
*************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在Tyrant模式下，通过initgroups()添加而外的组设置



emperor-stats
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行Emperor统计信息服务器



emperor-stats-server
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行Emperor统计信息服务器



early-emperor
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 尽快生成emperor



emperor-broodlord
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 在BroodLord模式下运行emperor



emperor-throttle
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为糟糕的vassal设置节流层次 (以毫秒为单位)  (默认是1000)



emperor-max-throttle
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为糟糕的vassal设置最大节流层次 (以毫秒为单位)  (默认是3分钟)



emperor-magic-exec
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果vassal配置文件有可执行位，则为它们添加exec://前缀



emperor-on-demand-extension
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 搜索包含按需socket名的文本文件 (vassal名+扩展名)



emperor-on-demand-ext
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 搜索包含即需socket名的文本文件 (vassal名 + 扩展)



emperor-on-demand-directory
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在按需模式下启用，绑定到指定目录下名字像vassal + .socket的unix socket



emperor-on-demand-dir
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在按需模式下启用，绑定到指定目录下名字像vassal + .socket的unix socket



emperor-on-demand-exec
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 使用指定命令的输出作为按需socket名 (传递vassal名作为唯一的参数)



emperor-extra-extension
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 允许在Emperor使用指定的扩展名（将会用--config调用vassal）



emperor-extra-ext
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 允许在Emperor使用指定的扩展名（将会用--config调用vassal）



emperor-no-blacklist
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用Emperor黑名单子系统



emperor-use-clone
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: 使用clone()而不是fork()来传递指定的unshare()标志



emperor-use-fork-server
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 对于新的vassal，连接到指定的fork服务器，而不是使用简单的fork()



vassal-fork-base
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 对指定的vassal使用简单的fork() (而不是一个fork服务器)



emperor-subreaper
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 强制Emperor成为一个子reaper (如果支持的话)



emperor-cap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: 设置vassal的capability



vassals-cap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: 设置vassal的capability



vassal-cap
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: 设置vassal的capability



emperor-collect-attribute
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 收集来自帝国监控器的指定vassal属性



emperor-collect-attr
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 收集来自帝国监控器的指定vassal属性



emperor-fork-server-attr
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 当检查fork服务器时，设置获取的vassal属性



emperor-wrapper-attr
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 当检查fork封装器时，设置获取的vassal属性



emperor-chdir-attr
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 当检查chdir时，设置获取的vassal属性



imperial-monitor-list
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出启用的帝国监控器



imperial-monitors-list
**********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出启用的帝国监控器



vassals-inherit
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加配置模板到vassal配置中 (使用--inherit)



vassals-include
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 包含配置模板到vassal配置 (使用--include而不是--inherit)



vassals-inherit-before
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加配置模板到vassal配置中 (使用--inherit，在vassal文件之前解析)



vassals-include-before
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 包含配置模板到vassal配置 (使用--include而不是--inherit，在vassal文件之前解析)



vassals-start-hook
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在每个vassal启动之前，运行指定的命令



vassals-stop-hook
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在vassal死掉之后，运行指定命令



vassal-sos
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 当过载时，要求emperor增援



vassal-sos-backlog
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 当backlog队列拥有比指定的值多的项的时候，要求emperor紧急救援



vassals-set
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 对每个vassal自动设置指定选项 (通过--set)



vassal-set
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 对每个vassal自动设置指定选项 (通过--set)



heartbeat
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 向emperor宣告健康



zeus
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 启用Zeus模式



reload-mercy
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置the maximum time (以秒为单位) we wait for workers and other processes to die during reload/shutdown



worker-reload-mercy
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置一个worker重载/关闭可以花费的最大时间 (以秒为单位，默认是60)



mule-reload-mercy
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置一个mule重载/关闭可以花费的最大时间 (以秒为单位，默认是60)



exit-on-reload
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 强制退出，即使请求重载



die-on-term
***********
``argument``: 无参数

``parser``: uwsgi_opt_deprecated

``help``: 对于SIGTERM，退出而不是粗鲁重载 (不再需要)



force-gateway
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 强制第一个注册的网关在无需master的情况下生成



help
****
``argument``: 无参数

``shortcut``: -h

``parser``: uwsgi_help

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 显示帮助



usage
*****
``argument``: 无参数

``shortcut``: -h

``parser``: uwsgi_help

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 显示帮助



print-sym
*********
``argument``: 必需参数

``parser``: uwsgi_print_sym

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 打印指定二进制符号的内容



print-symbol
************
``argument``: 必需参数

``parser``: uwsgi_print_sym

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 打印指定二进制符号的内容



reaper
******
``argument``: 无参数

``shortcut``: -r

``parser``: uwsgi_opt_true

``help``: 每个请求之后调用waitpid(-1,...)，以摆脱僵尸进程



max-requests
************
``argument``: 必需参数

``shortcut``: -R

``parser``: uwsgi_opt_set_64bit

``help``: 在指定数量的管理请求后重载worker



max-requests-delta
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 添加(worker_id * delta)到每个worker的max_requests值



min-worker-lifetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 在重载之前必须运行的秒数 (默认是60)



max-worker-lifetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 在指定秒数后重载worker (默认禁止)



socket-timeout
**************
``argument``: 必需参数

``shortcut``: -z

``parser``: uwsgi_opt_set_int

``help``: 设置间隔socket超时时间



no-fd-passing
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用文件描述符传递



locks
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 创建指定数目的共享锁



lock-engine
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置锁引擎



ftok
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 通过ftok()设置ipcsem键，以避免重复



persistent-ipcsem
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 关闭时不移除ipcsem



sharedarea
**********
``argument``: 必需参数

``shortcut``: -A

``parser``: uwsgi_opt_add_string_list

``help``: 创建指定页的原始共享内存区域 (注意：它也支持键值)

``reference``: :doc:`SharedArea`



safe-fd
*******
``argument``: 必需参数

``parser``: uwsgi_opt_safe_fd

``help``: 不关闭指定的文件描述符



fd-safe
*******
``argument``: 必需参数

``parser``: uwsgi_opt_safe_fd

``help``: 不关闭指定的文件描述符



cache
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 创建一个包含给定元素的共享缓存



cache-blocksize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置缓存块大小



cache-store
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 启用到磁盘的持久缓存



cache-store-sync
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置持久化缓存的同步频率



cache-no-expire
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用对过期项的自动清除



cache-expire-freq
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置缓存清道夫扫描频率（默认3秒）



cache-report-freed-items
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不断报告由清道夫释放的缓存项 (只用于调试)



cache-udp-server
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 绑定缓存udp服务器 (只用于设置/更新/删除) 到指定socket



cache-udp-node
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 发送缓存更新/删除到指定的缓存udp服务器



cache-sync
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在服务器启动时拷贝另一个uWSGI缓存服务器的完整内容



cache-use-last-modified
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在每次缓存项修改时更新last_modified_at时间戳 (默认禁止)



add-cache-item
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个项到缓存



load-file-in-cache
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载缓存中的一个静态文件



load-file-in-cache-gzip
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 使用gzip压缩加载缓存中的一个静态文件



cache2
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 创建一个新生代共享缓存（键值语法）



queue
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用共享队列



queue-blocksize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置队列块大小



queue-store
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 启用到磁盘的持久化队列



queue-store-sync
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为持久化队列设置同步频率



spooler
*******
``argument``: 必需参数

``shortcut``: -Q

``parser``: uwsgi_opt_add_spooler

``flags``: UWSGI_OPT_MASTER

``help``: 在指定目录上运行一个spooler



spooler-external
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_spooler

``flags``: UWSGI_OPT_MASTER

``help``: 映射spooler请求到由外部实例管理的spooler目录



spooler-ordered
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 尝试排序spooler任务的执行



spooler-chdir
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 每个spooler任务之前chdir()到指定目录



spooler-processes
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 为spooler设置进程数



spooler-quiet
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不要打印spooler任务的冗余信息



spooler-max-tasks
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 在循环使用一个spooler之前设置运行的最大任务数



spooler-harakiri
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为spooler任务设置harakiri超时时间



spooler-frequency
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置spooler频率，默认是30秒



spooler-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置spooler频率，默认是30秒



mule
****
``argument``: 可选参数

``parser``: uwsgi_opt_add_mule

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个mule



mules
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_mules

``flags``: UWSGI_OPT_MASTER

``help``: 添加指定数目的mule



farm
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_farm

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个mule farm



mule-msg-size
*************
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置mule消息缓冲大小



signal
******
``argument``: 必需参数

``parser``: uwsgi_opt_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 发送一个uwsgi信号到服务器



signal-bufsize
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置信号队列的缓冲大小



signals-bufsize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置信号队列的缓冲大小



signal-timer
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 设置定时器 (语法： <signal> <seconds>)



timer
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 设置定时器 (语法： <signal> <seconds>)



signal-rbtimer
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个红黑定时器 (语法： <signal> <seconds>)



rbtimer
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个红黑定时器 (语法： <signal> <seconds>)



rpc-max
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: rpc槽的最大数（默认：64）



disable-logging
***************
``argument``: 无参数

``shortcut``: -L

``parser``: uwsgi_opt_false

``help``: 禁用请求日志记录



flock
*****
``argument``: 必需参数

``parser``: uwsgi_opt_flock

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在启动前锁住指定文件，如果已经被锁了，则退出



flock-wait
**********
``argument``: 必需参数

``parser``: uwsgi_opt_flock_wait

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在启动前锁住指定文件，如果已经被锁了，则等待


flock2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在日志记录/守护设置之后锁住指定文件，如果已经被锁了，则退出



flock-wait2
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在日志记录/守护设置之后锁住指定文件，如果已经被锁了，则等待



pidfile
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 创建pid文件 (在移除特权之前)



pidfile2
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 创建pid文件 (在移除特权之后)



safe-pidfile
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 创建安全的pid文件 (在移除特权之前)



safe-pidfile2
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 创建安全的pid文件 (在移除特权之后)



chroot
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: chroot()到指定目录



pivot-root
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: pivot_root()到指定目录 (必须用一个空格来分隔new_root和put_old)



pivot_root
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: pivot_root()到指定目录 (必须用一个空格来分隔new_root和put_old)



uid
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_uid

``help``: setuid to the specified user/uid



gid
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_gid

``help``: setgid到指定的group/gid



add-gid
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加指定group id到进程凭证



immediate-uid
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_immediate_uid

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 立即setuid到指定的user/uid



immediate-gid
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_immediate_gid

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 立即setgid到指定的group/gid



no-initgroups
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 通过initgroups()禁用额外的组设置



cap
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_cap

``help``: 设置进程capability



unshare
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: unshare()部分进程，然后将其放在一个新的名字空间中



unshare2
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: 在rootfs改动之后，unshare()部分进程，然后将其放在一个新的名字空间中




setns-socket
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 公开一个返回来自/proc/self/ns的名字空间fds的 unix socket



setns-socket-skip
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当发送setns文件描述符时，跳过指定项



setns-skip
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当发送setns文件描述符时，跳过指定项



setns
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加入一个由外部uWSGI实例创建的名字空间



setns-preopen
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 尽快打开/proc/self/ns，并缓存fds



fork-socket
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在早期初始化之后挂起执行，并且在每次unix socket连接时fork()



fork-server
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在早期初始化之后挂起执行，并且在每次unix socket连接时fork()



jailed
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 标记实例为jailed状态 (强制执行post_jail钩子)



jail
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 把实例放到一个FreeBSD jail中



jail-ip4
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an ipv4 address to the FreeBSD jail



jail-ip6
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个ipv6地址到FreeBSD jail



jidfile
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 保存一个FreeBSD jail的jid到指定文件中



jid-file
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 保存一个FreeBSD jail的jid到指定文件中



jail2
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个选项到FreeBSD jail



libjail
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个选项到FreeBSD jail



jail-attach
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 附加到FreeBSD jail



refork
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之后再次fork()。对jailing系统有用



re-fork
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之后再次fork()。对jailing系统有用



refork-as-root
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之前再次fork()。对jailing系统有用



re-fork-as-root
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之前再次fork()。对jailing系统有用



refork-post-jail
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之后再次fork()。对jailing系统有用



re-fork-post-jail
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 移除特权之后再次fork()。对jailing系统有用



hook-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 尽快运行指定钩子



hook-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: jailing之前运行指定钩子



hook-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: jailing之后运行指定钩子



hook-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 初始化之后在jail中运行指定的钩子



hook-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在移除特权之前运行指定的钩子



hook-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在移除特权之后运行指定的钩子



hook-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用退出和重载之前运行指定的钩子



hook-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之前运行指定的钩子



hook-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之后运行指定的钩子



hook-post-fork
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每次fork之后运行指定的钩子



hook-accepting
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个worker进入接收阶段之后运行指定的钩子



hook-accepting1
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在第一个worker进入接收阶段之后运行指定的钩子



hook-accepting-once
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个worker进入接收阶段之后运行指定的钩子 (每个实例一次)



hook-accepting1-once
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在第一个worker进入接收阶段之后运行指定的钩子 (每个实例一次)



hook-master-start
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当Master启动时运行指定的钩子



hook-touch
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当碰了指定文件时运行指定的钩子 (语法：<file> <action>)



hook-emperor-start
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当Emperor启动时运行指定的钩子



hook-emperor-stop
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当Emperor发送一个停止消息时运行指定的钩子



hook-emperor-reload
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当Emperor发送一个重载消息时运行指定的钩子



hook-emperor-lost
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当失去Emperor连接时运行指定的钩子



hook-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前运行指定的钩子



hook-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在启动vassal之后，在emperor中运行指定的钩子



hook-as-on-demand-vassal
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 每当vassal进入按需模式时运行指定的钩子



hook-as-on-config-vassal
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 每当vassal检测到按需vassal的配置改动时运行指定的钩子



hook-as-emperor-before-vassal
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在生成新的vassal之前运行指定的钩子



hook-as-vassal-before-drop
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在移除vassal特权之前，运行指定的钩子



hook-as-emperor-setns
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在emperor中运行指定的钩子，进入vassal名字空间



hook-as-mule
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个mule中运行指定的钩子



hook-as-gateway
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个网关中运行指定的钩子



after-request-hook
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个请求之后，运行指定的函数/符号



after-request-call
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在每个请求之后，运行指定的函数/符号


exec-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 尽快运行指定命令



exec-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之前运行指定命令



exec-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之后运行指定命令



exec-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 初始化之后，在jail中运行指定命令



exec-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之前运行指定命令



exec-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之后运行指定命令



exec-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用退出和重载之前运行指定命令



exec-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之前运行指定命令



exec-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之后运行指定命令



exec-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前运行指定命令



exec-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中运行指定命令



mount-asap
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 尽快挂载文件系统



mount-pre-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之前挂载文件系统



mount-post-jail
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之后挂载文件系统



mount-in-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 初始化后，在jail中挂载文件系统



mount-as-root
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之前挂载文件系统



mount-as-vassal
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前挂载文件系统



mount-as-emperor
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中挂载文件系统



umount-asap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 尽快取消挂载文件系统



umount-pre-jail
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之前取消挂载文件系统



umount-post-jail
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之后取消挂载文件系统



umount-in-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 初始化后，在jail中取消挂载文件系统



umount-as-root
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之前取消挂载文件系统



umount-as-vassal
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前取消挂载文件系统



umount-as-emperor
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中取消挂载文件系统



wait-for-interface
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的网络接口出现



wait-for-interface-timeout
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置wait-for-interface的超时时间



wait-interface
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的网络接口出现



wait-interface-timeout
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置wait-for-interface的超时时间



wait-for-iface
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的网络接口出现



wait-for-iface-timeout
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置wait-for-interface的超时时间



wait-iface
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的网络接口出现



wait-iface-timeout
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置wait-for-interface的超时时间



wait-for-fs
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的文件系统项出现



wait-for-file
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的文件出现



wait-for-dir
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的目录出现



wait-for-mountpoint
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在运行root钩子之前等待指定的挂载点出现


wait-for-fs-timeout
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为wait-for-fs/file/dir设置超时时间



call-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 尽快调用指定函数



call-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之前调用指定函数



call-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在jailing之后调用指定函数



call-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 初始化后，在jail中调用指定函数



call-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之前调用指定函数



call-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在移除特权之后调用指定函数



call-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用退出和重载之前调用指定函数



call-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之前调用指定函数



call-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在应用加载之后调用指定函数



call-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前调用指定函数()



call-as-vassal1
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前调用指定函数



call-as-vassal3
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在exec() vassal之前调用指定函数(char *, uid_t, gid_t)



call-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中调用指定函数()



call-as-emperor1
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中调用指定函数



call-as-emperor2
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中调用指定函数(char *, pid_t)



call-as-emperor4
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 启动了vassal之后，在emperor中调用指定函数(char *, pid_t, uid_t, gid_t)



ini
***
``argument``: 必需参数

``parser``: uwsgi_opt_load_ini

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从ini文件加载配置



yaml
****
``argument``: 必需参数

``shortcut``: -y

``parser``: uwsgi_opt_load_yml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从yaml文件加载配置



yml
***
``argument``: 必需参数

``shortcut``: -y

``parser``: uwsgi_opt_load_yml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从yaml文件加载配置



json
****
``argument``: 必需参数

``shortcut``: -j

``parser``: uwsgi_opt_load_json

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从json文件加载配置



js
**
``argument``: 必需参数

``shortcut``: -j

``parser``: uwsgi_opt_load_json

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从json文件加载配置



weight
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 实例的权重 (由集群/lb/订阅使用)



auto-weight
***********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 自动设置实例的权重 (由集群/lb/订阅使用)



no-server
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 强制使用非服务器模式



command-mode
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 强制使用命令模式



no-defer-accept
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用socket上的deferred-accept



tcp-nodelay
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在每个请求上启用TCP NODELAY



so-keepalive
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用TCP KEEPALIVE



so-send-timeout
***************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: 设置SO_SNDTIMEO



socket-send-timeout
*******************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: 设置SO_SNDTIMEO



so-write-timeout
****************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: 设置SO_SNDTIMEO



socket-write-timeout
********************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: 设置SO_SNDTIMEO



socket-sndbuf
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置SO_SNDBUF



socket-rcvbuf
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置SO_RCVBUF



limit-as
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``help``: 限制进程地址空间/vsz



limit-nproc
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 限制可生成进程的数目



reload-on-as
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MEMORY

``help``: 如果地址空间比指定百万字节高，那么重载



reload-on-rss
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MEMORY

``help``: 如果rss内存比指定百万字节高，那么重载



evil-reload-on-as
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_MEMORY

``help``: 如果worker的地址空间比指定百万字节高，那么强制master重载这个worker



evil-reload-on-rss
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_MEMORY

``help``: 如果worker的rss内存比指定百万字节高，那么强制master重载这个worker



reload-on-fd
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果指定的文件描述符准备好了，则重载



brutal-reload-on-fd
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果指定的文件描述符准备好了，则粗鲁重载



ksm
***
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``help``: 启用Linux KSM



pcre-jit
********
``argument``: 无参数

``parser``: uwsgi_opt_pcre_jit

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 启用pcre jit (如果可用)



never-swap
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 锁定所有的内存页，避免交换



touch-reload
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果修改/碰了指定的文件，那么重载uWSGI



touch-workers-reload
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果修改/碰了指定的文件，那么触发（仅）worker的重载



touch-chain-reload
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果修改/碰了指定的文件，那么触发链式重载



touch-logrotate
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 如果修改/碰了指定的文件，那么触发日志循环



touch-logreopen
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 如果修改/碰了指定的文件，那么触发日志的重新打开



touch-exec
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当修改/碰了指定的文件时，运行命令 (语法：file command)



touch-signal
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当修改/碰了指定的文件时，发送信号 (语法：file signal)



fs-reload
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当修改了指定的文件系统对象时，优雅重载



fs-brutal-reload
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当修改了指定的文件系统对象时，粗鲁重载



fs-signal
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当修改了指定的文件系统对象时，引发一个uwsgi信号 (语法：file signal)



check-mountpoint
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果一个文件系统不再可达，则摧毁实例（对于可靠Fuse管理有用）



mountpoint-check
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果一个文件系统不再可达，则摧毁实例（对于可靠Fuse管理有用）



check-mount
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果一个文件系统不再可达，则摧毁实例（对于可靠Fuse管理有用）



mount-check
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 如果一个文件系统不再可达，则摧毁实例（对于可靠Fuse管理有用）



propagate-touch
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 用于带怪怪的信号管理的系统的过度设计选项



limit-post
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 限制请求体



no-orphans
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果master死掉了，自动杀死worker (对于可用性可能是危险的)



prio
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_rawint

``help``: 设置进程/线程优先权



cpu-affinity
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置cpu关系



post-buffering
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 启用post buffering



post-buffering-bufsize
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置post buffering模式下，read()的缓冲大小



body-read-warning
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置开始打印警告之前对请求体的内存分配允许量（以百万字节为单位）



upload-progress
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 启用文件上传期间，指定目录中的.json文件创建



no-default-app
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不要回退到默认应用



manage-script-name
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 自动重写SCRIPT_NAME和PATH_INFO



ignore-script-name
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 忽略SCRIPT_NAME



catch-exceptions
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 将异常作为http输出报告 (不鼓励，只用于测试)



reload-on-exception
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 当触发了一个异常时，重载worker



reload-on-exception-type
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当引发了一个指定的异常类型时，重载worker



reload-on-exception-value
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当引发了一个指定的异常值时，重载worker



reload-on-exception-repr
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 当引发了一个指定的异常类型+值（语言特定）时，重载worker



exception-handler
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个异常处理器



enable-metrics
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 启用度量子系统



metric
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 添加一个自定义的度量



metric-threshold
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 添加一个度量阈值/告警



metric-alarm
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 添加一个度量阈值/告警



alarm-metric
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 添加一个度量阈值/告警



metrics-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 将度量作为文本文件导出到指定目录



metrics-dir-restore
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 恢复从度量目录获取的最后值



metric-dir
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 将度量作为文本文件导出到指定目录



metric-dir-restore
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 恢复从度量目录获取的最后值



metrics-no-cores
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: 禁用核心相关度量的生成

``reference``: :doc:`Metrics`



不要公开异步核心的度量。

udp
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 在指定的地址上运行udp服务器



stats
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 在指定的地址上启用统计信息服务器



stats-server
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 在指定的地址上启用统计信息服务器



stats-http
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 给统计信息服务器的json输出加上http头部前缀



stats-minified
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 最小化统计信息json输出



stats-min
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 最小化统计信息json输出



stats-push
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER|UWSGI_OPT_METRICS

``help``: 推送统计信息json到指定的目的地



stats-pusher-default-freq
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置统计信息推送器的默认频率



stats-pushers-default-freq
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置统计信息推送器的默认频率



stats-no-cores
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 禁用核心相关的统计信息的生成

``reference``: :doc:`Metrics`



不要在统计信息服务器中公开关于核心的信息。

stats-no-metrics
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 不要在统计信息输出中包含度量

``reference``: :doc:`Metrics`



一点也不要在统计信息服务器公开度量。

multicast
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 订阅到指定的多播组



multicast-ttl
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置多播ttl



multicast-loop
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置多播循环（默认1）



master-fifo
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 启用master fifo



notify-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 启用通知socket



subscription-notify-socket
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 为订阅设置通知socket



subscription-mountpoints
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 为订阅系统启用挂载点支持



subscription-mountpoint
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 为订阅系统启用挂载点支持


legion
******
``argument``: 必需参数

``parser``: uwsgi_opt_legion

``flags``: UWSGI_OPT_MASTER

``help``: 成为一个legion的成员



legion-mcast
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_mcast

``flags``: UWSGI_OPT_MASTER

``help``: 成为一个legion的成员 (multicast的快捷方式)



legion-node
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_node

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个节点到legion



legion-freq
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置legion包的频率



legion-tolerance
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置legion子系统的容许度



legion-death-on-lord-error
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 如果lord钩子中的一个失败的话，则宣告自身在指定秒数内都是一个死掉的节点



legion-skew-tolerance
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置legion子系统的时钟偏差容许度 (默认是30秒)



legion-lord
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: Lord选举时要调用的动作



legion-unlord
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: 开除Lord时要调用的动作



legion-setup
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: legion设置时要调用的动作



legion-death
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: legion死掉时要调用的动作 (实例的关闭)



legion-join
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: legion加入时要调用的动作 (第一次到达法定数量时)



legion-node-joined
******************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: 新节点加入legion时要调用的动作



legion-node-left
****************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: 节点离开legion时要调用的动作



legion-quorum
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_quorum

``flags``: UWSGI_OPT_MASTER

``help``: 设置一个Legion的法定数量



legion-scroll
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_scroll

``flags``: UWSGI_OPT_MASTER

``help``: 设置一个legion的scroll



legion-scroll-max-size
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: 设置legion scroll缓冲的最大大小



legion-scroll-list-max-size
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置legion scroll列表缓冲的最大大小



subscriptions-sign-check
************************
``argument``: 必需参数

``parser``: uwsgi_opt_scd

``flags``: UWSGI_OPT_MASTER

``help``: 为安全的订阅系统设置digest算法和证书目录



subscriptions-sign-check-tolerance
**********************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 为安全的订阅系统设置时钟偏差的最大容忍度（以秒为单位）



subscriptions-sign-skip-uid
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当使用unix socket凭证的时候，对指定的uid跳过签名检查



subscriptions-credentials-check
*******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个用以搜索订阅密钥凭证的目录



subscriptions-use-credentials
*****************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用订阅UNIX socket中的SCM_CREDENTIALS管理



subscription-algo
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_ssa

``help``: 为订阅系统设置负载均衡算法



subscription-dotsplit
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 尝试回退到订阅键中的下一个部分（基于点）



subscribe-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 订阅到指定的订阅服务器



st
**
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 订阅到指定的订阅服务器



subscribe
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 订阅到指定的订阅服务器



subscribe2
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 使用高级的键值语法订阅到指定的订阅服务器



subscribe-freq
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 以指定间隔发送订阅宣告



subscription-tolerance
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为订阅服务器设置容忍度



unsubscribe-on-graceful-reload
******************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 强制取消订阅请求，即使是在优雅重载期间



start-unsubscribed
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 配置订阅，但是不要发送它们 (对master fifo有用)



subscribe-with-modifier1
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 订阅时，强制使用指定的modifier1



snmp
****
``argument``: 可选参数

``parser``: uwsgi_opt_snmp

``help``: 启用嵌入的snmp服务器



snmp-community
**************
``argument``: 必需参数

``parser``: uwsgi_opt_snmp_community

``help``: 设置snmp community字符串



ssl-verbose
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 输出SSL错误详细信息



ssl-sessions-use-cache
**********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 将uWSGI缓存用于ssl会话存储



ssl-session-use-cache
*********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 将uWSGI缓存用于ssl会话存储



ssl-sessions-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置SSL会话超时时间 (默认：300秒)



ssl-session-timeout
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置SSL会话超时时间 (默认：300秒)



sni
***
``argument``: 必需参数

``parser``: uwsgi_opt_sni

``help``: 添加一个管理SNI的SSL上下文



sni-dir
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 检查指定目录中的cert/key/client_ca文件，并按需创建一个sni/ssl上下文



sni-dir-ciphers
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为sni-dir设置ssl密码



ssl-enable3
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用SSLv3 (不安全的)



ssl-option
**********
``argument``: 无参数

``parser``: uwsgi_opt_add_string_list

``help``: 设置一个原始的ssl选项 (数值)



sni-regexp
**********
``argument``: 必需参数

``parser``: uwsgi_opt_sni

``help``: 添加一个管理SNI的SSL上下文 (键是一个正则表达式)



ssl-tmp-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在指定的目录中存储ssl相关的临时文件



check-interval
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置master检查的间隔 (以秒为单位)



forkbomb-delay
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 当检查到一个forkbomb时，休眠指定秒数



binary-path
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 强制使用二进制路径



privileged-binary-patch
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 用一个新的命令给uwsgi二进制文件打补丁 (在移除特权之前)



unprivileged-binary-patch
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 用一个新的命令给uwsgi二进制文件打补丁 (在移除特权之后)



privileged-binary-patch-arg
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 用一个新的命令和参数给uwsgi二进制文件打补丁 (在移除特权之前)



unprivileged-binary-patch-arg
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 用一个新的命令和参数给uwsgi二进制文件打补丁 (在移除特权之后)



async
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用使用指定核心的异步模式



disable-async-warn-on-queue-full
********************************
``argument``: 无参数

``parser``: uwsgi_opt_false

``help``: 禁止打印'async queue is full'警告信息。



max-fd
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置文件描述符的最大数目 (需要root权限)



logto
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置logfile/udp地址



logto2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在移除特权之后记录日志到指定的文件或者udp地址



log-format
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为请求日志记录设置高级格式



logformat
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为请求日志记录设置高级格式



logformat-strftime
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 应用strftime到logformat输出



log-format-strftime
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 应用strftime到logformat输出



logfile-chown
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: chown日志文件



logfile-chmod
*************
``argument``: 必需参数

``parser``: uwsgi_opt_logfile_chmod

``help``: chmod日志文件



log-syslog
**********
``argument``: 可选参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 记录日志到syslog



log-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 发送日志到指定的socket



req-logger
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_req_logger

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: 设置/附加一个请求记录器



logger-req
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_req_logger

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: 设置/附加一个请求记录器



logger
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 设置/附加一个记录器



logger-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的记录器



loggers-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的记录器



threaded-logger
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 卸载日志写入到一个线程中



log-encoder
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 添加一个项到日志编码器链中



log-req-encoder
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 添加一个项到日志请求编码器链中



log-drain
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 排除（不要显示）匹配到指定正则表达式的日志行



log-filter
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 只显示匹配到指定正则表达式的日志行



log-route
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_custom_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 如果应用到日志行的正则表达式匹配上的话，则记录日志到指定名字的记录器



log-req-route
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_custom_list

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: 如果应用到日志行的正则表达式匹配上的话，则记录请求到指定名字的记录器



use-abort
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在segfault/fpe上调用abort()，对于生成一个core dump可能有用



alarm
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 创建一个新的告警，语法：<alarm> <plugin:args>



alarm-cheap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 对基于curl的告警使用主要的告警线程，而不是创建专用线程



alarm-freq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 调整反循环告警系统 (默认3秒)



alarm-fd
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当一个fd准备好读取的时候，引发指定的告警 (默认情况下，它读取1个字节，为eventfd设置8)



alarm-segfault
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当执行了段错误处理器时，引发指定的告警



segfault-alarm
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当执行了段错误处理器时，引发指定的告警



alarm-backlog
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



backlog-alarm
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



lq-alarm
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



alarm-lq
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



alarm-listen-queue
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



listen-queue-alarm
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 当socket backlog队列满时，引发指定的告警



log-alarm
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 当日志行匹配到指定的正则表达式时，引发指定的告警，语法： <alarm>[,alarm...] <regexp>



alarm-log
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 当日志行匹配到指定的正则表达式时，引发指定的告警，语法： <alarm>[,alarm...] <regexp>



not-log-alarm
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list_custom

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 当日志行匹配到指定的正则表达式时，跳过指定的告警，语法： <alarm>[,alarm...] <regexp>



not-alarm-log
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list_custom

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: 当日志行匹配到指定的正则表达式时，跳过指定的告警，语法：<alarm>[,alarm...] <regexp>



alarm-list
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用告警



alarms-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用告警



alarm-msg-size
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置告警消息的最大大小 (默认：8192)



log-master
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER|UWSGI_OPT_LOG_MASTER

``help``: 委托日志记录给master进程



log-master-bufsize
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 为master记录器设置缓冲大小。更大的日志消息将会被截短



log-master-stream
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 将master logpipe作为SOCK_STREAM创建



log-master-req-stream
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 将master请求logpipe作为SOCK_STREAM创建



log-reopen
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 重新加载后重新打开日志



log-truncate
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启动时清除日志



log-maxsize
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER|UWSGI_OPT_LOG_MASTER

``help``: 设置最大日志文件大小



log-backupname
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置日志循环后的日志文件名



logdate
*******
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: 给日志加上日期或者一个strftime字符串前缀



log-date
********
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: 给日志加上日期或者一个strftime字符串前缀



log-prefix
**********
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: 给日志加上一个字符串前缀



log-zero
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 记录没有body的响应



log-slow
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 记录慢于指定毫秒数的请求



log-4xx
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 记录响应为4xx的请求



log-5xx
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 记录响应为5xx的请求



log-big
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 记录大于指定大小的请求



log-sendfile
************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 记录sendfile请求



log-ioerror
***********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 记录带io错误的日志请求



log-micros
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 以微秒，而不是毫秒为单位报告响应时间



log-x-forwarded-for
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 使用来自X-Forwarded-For头部而不是REMOTE_ADDR头部的ip



master-as-root
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 让master进程作为root运行



drop-after-init
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 插件初始化之后运行特权移除



drop-after-apps
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 应用加载后运行特权移除



force-cwd
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 强制使用指定的初始工作目录



binsh
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 覆盖/bin/sh (exec钩子使用，总是回退到/bin/sh)



chdir
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 应用加载前chdir到指定目录



chdir2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 应用加载之后chdir到指定目录



lazy
****
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 设置lazy模式 (在worker而不是master中加载应用)



lazy-apps
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在每个worker而不是master中加载应用



cheap
*****
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 设置cheap模式 (只在第一个请求后生成worker)



cheaper
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 设置cheaper模式 (适应性进程生成)



cheaper-initial
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 设置cheaper模式下生成的初始进程数



cheaper-algo
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 选择一个用于适应性进程生成的算法



cheaper-step
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 在每次超载时生成的额外进程数



cheaper-overload
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 在指定过载后增加worker


cheaper-idle
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 在指定idle后减少worker (算法：spare2) (默认：10)


cheaper-algo-list
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的cheaper算法



cheaper-algos-list
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的cheaper算法



cheaper-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的cheaper算法



cheaper-rss-limit-soft
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 如果所有worker总的常驻内存使用高于这个限制，则不要生成新的worker



cheaper-rss-limit-hard
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: 如果总的worker常驻内存使用更高，则试着停止worker



idle
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 设置idle模式 (在静止后，将uWSGI置于cheap模式)



die-on-idle
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 当idle时，关闭uWSGI



mount
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在挂载点下加载应用



worker-mount
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在指定worker中，或者worker生成后，在挂载点下加载应用



threads
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: 在prethreaded模式下运行每个worker，使用指定数目的线程



thread-stacksize
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: 设置线程栈大小



threads-stacksize
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: 设置线程栈大小



thread-stack-size
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: 设置线程栈大小



threads-stack-size
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: 设置线程栈大小



vhost
*****
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用虚拟主机模式 (基于SERVER_NAME变量)



vhost-host
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_VHOST

``help``: 启用虚拟主机模式 (基于HTTP_HOST变量)



route
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个路由



route-host
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于Host头部，添加一个路由


route-uri
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REQUEST_URI添加一个路由



route-qs
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于QUERY_STRING添加一个路由



route-remote-addr
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_ADDR添加一个路由



route-user-agent
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_USER_AGENT添加一个路由



route-remote-user
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_USER添加一个路由



route-referer
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_REFERER添加一个路由



route-label
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个路由标签 (用于和goto一起使用)



route-if
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个路由



route-if-not
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个路由 (否定版本)



route-run
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 总是运行指定的路由动作



final-route
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个最终路由



final-route-status
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 为指定状态添加一个最终路由



final-route-host
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于Host头部添加一个最终路由



final-route-uri
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REQUEST_URI添加一个最终路由



final-route-qs
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于QUERY_STRING添加一个最终路由



final-route-remote-addr
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_ADDR添加一个最终路由



final-route-user-agent
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_USER_AGENT添加一个最终路由



final-route-remote-user
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_USER添加一个最终路由



final-route-referer
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_REFERER添加一个最终路由



final-route-label
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个最终路由标签 (用于和goto一起使用)



final-route-if
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个最终路由



final-route-if-not
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个最终路由 (否定版本)



final-route-run
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 总是运行指定的最终路由动作



error-route
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个错误路由



error-route-status
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 为指定状态添加一个错误路由



error-route-host
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于Host头部添加一个错误路由



error-route-uri
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REQUEST_URI添加一个错误路由



error-route-qs
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于QUERY_STRING添加一个错误路由



error-route-remote-addr
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_ADDR添加一个错误路由



error-route-user-agent
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_USER_AGENT添加一个错误路由



error-route-remote-user
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_USER添加一个错误路由



error-route-referer
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_REFERER添加一个错误路由



error-route-label
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个错误路由标签 (用于和goto一起使用)



error-route-if
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个错误路由



error-route-if-not
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个错误路由 (否定版本)



error-route-run
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 总是运行指定的错误路由动作



response-route
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个响应路由



response-route-status
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于指定状态添加一个响应路由



response-route-host
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于Host头部添加一个响应路由



response-route-uri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REQUEST_URI添加一个响应路由



response-route-qs
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于QUERY_STRING添加一个响应路由



response-route-remote-addr
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_ADDR添加一个响应路由



response-route-user-agent
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_USER_AGENT添加一个响应路由



response-route-remote-user
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于REMOTE_USER添加一个响应路由



response-route-referer
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于HTTP_REFERER添加一个响应路由



response-route-label
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 添加一个响应路由标签 (用于和goto一起使用)



response-route-if
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个响应路由



response-route-if-not
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 基于条件添加一个响应路由 (否定版本)



response-route-run
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: 总是运行指定的响应路由动作



router-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用路由器



routers-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用路由器



error-page-403
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个错误页面 (html) 来管理403响应



error-page-404
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个错误页面 (html) 来管理404响应



error-page-500
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个错误页面 (html) 来管理500响应



websockets-ping-freq
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置websockets自动ping包的频率 (以秒为单位) 



websocket-ping-freq
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置websockets自动ping包的频率 (以秒为单位) 



websockets-pong-tolerance
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置websockets ping/pong子系统的容忍度 (以秒为单位)



websocket-pong-tolerance
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置websockets ping/pong子系统的容忍度 (以秒为单位)



websockets-max-size
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置websocket消息的最大允许大小 (以KB为单位，默认是1024)



websocket-max-size
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置websocket消息的最大允许大小 (以KB为单位，默认是1024)



chunked-input-limit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置块输入部分的最大大小 (默认是1MB，以字节为单位)



chunked-input-timeout
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为块输入设置默认超时时间



clock
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置一个时钟源



clock-list
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用时钟



clocks-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用时钟



add-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 自动添加HTTP头部到响应



rem-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 自动从响应移除指定的HTTP头部



del-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 自动从响应移除指定的HTTP头部



collect-header
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 存储指定的响应头部到一个请求变量中 (语法：header var)



response-header-collect
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 存储指定的响应头部到一个请求变量中 (语法：header var)



pull-header
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 存储指定的响应头部到一个请求变量中，并将其从响应中移除 (语法：header var)



check-static
************
``argument``: 必需参数

``parser``: uwsgi_opt_check_static

``flags``: UWSGI_OPT_MIME

``help``: 检查指定的目录中的静态文件



check-static-docroot
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: 检查请求的DOCUMENT_ROOT中的静态文件



static-check
************
``argument``: 必需参数

``parser``: uwsgi_opt_check_static

``flags``: UWSGI_OPT_MIME

``help``: 检查指定的目录中的静态文件



static-map
**********
``argument``: 必需参数

``parser``: uwsgi_opt_static_map

``flags``: UWSGI_OPT_MIME

``help``: 映射挂载点到静态目录（或者文件）



static-map2
***********
``argument``: 必需参数

``parser``: uwsgi_opt_static_map

``flags``: UWSGI_OPT_MIME

``help``: 和static-map类似，但是完全附加请求资源到docroot



static-skip-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 静态文件检查时跳过指定的扩展名



static-index
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 如果请求一个目录，则搜索指定的文件



static-safe
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 如果文件位于指定路径下，则跳过安全性检查



static-cache-paths
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MIME|UWSGI_OPT_MASTER

``help``: 将已解析的路径放到uWSGI缓存中，存储指定秒数



static-cache-paths-name
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME|UWSGI_OPT_MASTER

``help``: 将指定缓存用于静态路径



mimefile
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 设置mime types文件路径 (默认/etc/apache2/mime.types)



mime-file
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 设置mime types文件路径 (默认/etc/apache2/mime.types)



mimefile
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 设置mime types文件路径 (默认/etc/mime.types)



mime-file
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 设置mime types文件路径 (默认/etc/mime.types)



static-expires-type
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于内容类型设置Expires头部



static-expires-type-mtime
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于内容类型和文件mtime设置Expires头部



static-expires
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于文件名正则表达式设置Expires头部



static-expires-mtime
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于文件名正则表达式和文件mtime设置Expires头部



static-expires-uri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于REQUEST_URI正则表达式设置Expires头部



static-expires-uri-mtime
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于REQUEST_URI正则表达式和文件mtime设置Expires头部



static-expires-path-info
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于PATH_INFO正则表达式设置Expires头部



static-expires-path-info-mtime
******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: 基于PATH_INFO正则表达式和文件mtime设置Expires头部



static-gzip
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MIME

``help``: 如果提供的正则表达式匹配静态文件转换，那么它将搜索一个gzip版本



static-gzip-all
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: 检查所有请求的静态文件的gzip版本



static-gzip-dir
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 检查指定目录/前缀中所有请求的静态文件的gzip版本



static-gzip-prefix
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 检查指定目录/前缀中所有请求的静态文件的gzip版本



static-gzip-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 检查带指定扩展名/后缀的所有请求的静态文件的gzip版本



static-gzip-suffix
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 检查带指定扩展名/后缀的所有请求的静态文件的gzip版本



honour-range
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用对HTTP Range头部的支持



offload-threads
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置要生成的卸载线程数 (每个worker，默认是0)



offload-thread
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置要生成的卸载线程数 (每个worker，默认是0)



file-serve-mode
***************
``argument``: 必需参数

``parser``: uwsgi_opt_fileserve_mode

``flags``: UWSGI_OPT_MIME

``help``: 设置静态文件服务模式



fileserve-mode
**************
``argument``: 必需参数

``parser``: uwsgi_opt_fileserve_mode

``flags``: UWSGI_OPT_MIME

``help``: 设置静态文件服务模式



disable-sendfile
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用sendfile()，并依赖无聊的read()/write()



check-cache
***********
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 检查指定缓存中的响应数据（空则表示默认缓存）



close-on-exec
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在连接socket上设置close-on-exec (对于在请求中生成进程可能是必须的)



close-on-exec2
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在服务器socket上设置close-on-exec (对于在请求中生成进程可能是必须的)



mode
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置uWSGI custom模式



env
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_env

``help``: 设置环境变量



ienv
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_env

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 设置环境变量 (立即版本)



envdir
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载一个守护工具兼容的envdir



early-envdir
************
``argument``: 必需参数

``parser``: uwsgi_opt_envdir

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 尽快加载一个守护工具兼容的envdir



unenv
*****
``argument``: 必需参数

``parser``: uwsgi_opt_unset_env

``help``: 取消设置环境变量



vacuum
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 尝试移除所有生成的文件/socket



file-write
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 移除特权之前将指定内容写入到指定文件中 (语法：file=value) 



cgroup
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 将进程放到指定的cgroup中



cgroup-opt
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在指定的cgroup选项中设置值



cgroup-dir-mode
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为cgroup目录设置权限（默认是700）



namespace
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在指定的rootfs下的一个新的名字空间中运行



namespace-keep-mount
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在你的名字空间中保持指定的挂载点



ns
**
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 在指定的rootfs下的一个新的名字空间中运行



namespace-net
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 添加network名字空间



ns-net
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 添加network名字空间



enable-proxy-protocol
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用PROXY1协议支持 (仅用于http解析器)



reuse-port
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在socket上启用REUSE_PORT标志 (仅BSD)



tcp-fast-open
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 在TCP socket上启用TCP_FASTOPEN标志，使用指定的qlen值



tcp-fastopen
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 在TCP socket上启用TCP_FASTOPEN标志，使用指定的qlen值



tcp-fast-open-client
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果支持的话，使用sendto(..., MSG_FASTOPEN, ...)来代替connect()



tcp-fastopen-client
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果支持的话，使用sendto(..., MSG_FASTOPEN, ...)来代替connect()



zerg
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 附加到一个zerg服务器



zerg-fallback
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果zerg服务器不可用，则回退到正常的socket



zerg-server
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: 在指定的UNIX socket上启用zerg服务器



cron
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务



cron2
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron2

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务 (key=val语法)



unique-cron
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_cron

``flags``: UWSGI_OPT_MASTER

``help``: 添加一个唯一的cron任务



cron-harakiri
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置等待cron命令完成的最大时间 (以秒为单位)



legion-cron
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，该任务只有在实例是一个指定legion的lord时才能运行



cron-legion
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，该任务只有在实例是一个指定legion的lord时才能运行



unique-legion-cron
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个唯一的cron任务，该任务只有在实例是一个指定legion的lord时才能运行



unique-cron-legion
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个唯一的cron任务，该任务只有在实例是一个指定legion的lord时才能运行



loop
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 选择uWSGI循环引擎



loop-list
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的循环引擎



loops-list
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的循环引擎



worker-exec
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 作为worker运行指定命令



worker-exec2
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 作为worker运行指定命令 (post_fork钩子之后)



attach-daemon
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 附加一个命令/守护进程到master进程（命令必须不跑到后台）



attach-control-daemon
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 附加一个命令/守护进程到master进程（命令必须不跑到后台），当守护进程死掉时，master也会死掉



smart-attach-daemon
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 附加一个命令/守护进程到由pid文件管理的master进程（命令必须被守护）



smart-attach-daemon2
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 附加一个命令/守护进程到由pid文件管理的master进程（命令必须不被守护）



legion-attach-daemon
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 与--attach-daemon相同，但是守护进程只在legion lord节点上运行



legion-smart-attach-daemon
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 与--smart-attach-daemon相同，但是守护进程只在legion lord节点上运行



legion-smart-attach-daemon2
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: 与--smart-attach-daemon2相同，但是守护进程只在legion lord节点上运行



daemons-honour-stdin
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: 不要讲外部守护进程的标准输入修改为/dev/null



attach-daemon2
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon2

``flags``: UWSGI_OPT_MASTER

``help``: attach-daemon键值变体 (也支持smart模式)



plugins
*******
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 加载uWSGI插件



plugin
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 加载uWSGI插件



need-plugins
************
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 加载uWSGI插件 (错误时退出)



need-plugin
***********
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 加载uWSGI插件 (错误时退出)



plugins-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 新增一个目录到uWSGI插件搜索路径



plugin-dir
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 新增一个目录到uWSGI插件搜索路径



plugins-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的插件



plugin-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 列出已启用的插件



autoload
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 当找到未知选项时，尝试自动加载插件



dlopen
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_dl

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 盲目加载一个共享库



allowed-modifiers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 允许的modifier的以逗号分隔的列表



remap-modifier
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 将请求modifier从一个id重新映射到另一个



dump-options
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: dump可用选项的完整列表



show-config
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 显示重新格式化为ini的当前配置



binary-append-data
******************
``argument``: 必需参数

``parser``: uwsgi_opt_binary_append_data

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 返回一个资源的内容到标准输出，用于附加到一个uwsgi二进制文件中 (用于data://)



print
*****
``argument``: 必需参数

``parser``: uwsgi_opt_print

``help``: 简单打印



iprint
******
``argument``: 必需参数

``parser``: uwsgi_opt_print

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 简单打印 (立即版本)



exit
****
``argument``: 可选参数

``parser``: uwsgi_opt_exit

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 强制实例的exit()



cflags
******
``argument``: 无参数

``parser``: uwsgi_opt_cflags

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 报告uWSGI CFLAGS (对构建外部插件有用)



dot-h
*****
``argument``: 无参数

``parser``: uwsgi_opt_dot_h

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 转储用于构建核心的uwsgi.h  (对构建外部插件有用)



config-py
*********
``argument``: 无参数

``parser``: uwsgi_opt_config_py

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 转储用于构建核心的uwsgiconfig.py  (对构建外部插件有用)



build-plugin
************
``argument``: 必需参数

``parser``: uwsgi_opt_build_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 为当前二进制文件构建一个uWSGI插件



version
*******
``argument``: 无参数

``parser``: uwsgi_opt_print

``help``: 打印uWSG版本




plugin: airbrake
================

plugin: alarm_curl
==================

plugin: alarm_speech
====================

plugin: alarm_xmpp
==================

plugin: asyncio
===============
asyncio
*******
``argument``: 必需参数

``parser``: uwsgi_opt_setup_asyncio

``flags``: UWSGI_OPT_THREADS

``help``: 一个快捷方式，启用带有指定异步核心数和优化参数的asyncio循环引擎




plugin: cache
=============

plugin: carbon
==============
carbon
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: 推送统计信息到指定的carbon服务器



carbon-timeout
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置carbon连接超时时间，以秒为单位（默认是3） 



carbon-freq
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置carbon推送频率，以秒为单位（默认是60）



carbon-id
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置carbon id



carbon-no-workers
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用单个worker度量的生成



carbon-max-retry
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置连接错误下的最大重试数 (默认是1)



carbon-retry-delay
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置连接重试延迟，以秒为单位（默认是7）



carbon-root
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置carbon度量根节点 (默认是'uwsgi')



carbon-hostname-dots
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置主机名中用以替换点的字符 (默认不替换点)



carbon-name-resolve
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 允许将主机名用作carbon服务器地址 (默认禁用)



carbon-resolve-names
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 允许将主机名用作carbon服务器地址 (默认禁用)



carbon-idle-avg
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: idle期间（无请求）的平均值，可以是"last", "zero", "none" (默认是last)



carbon-use-metrics
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不要计算所有的统计信息，使用度量子系统数据来代替 (警告！键值将不同)




plugin: cgi
===========
cgi
***
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi

``help``: 添加一个cgi挂载点/目录/脚本



cgi-map-helper
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi_maphelper

``help``: 添加一个cgi map-helper



cgi-helper
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi_maphelper

``help``: 添加一个cgi map-helper



cgi-from-docroot
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在DOCUMENT_ROOT中盲目启用cgi



cgi-buffer-size
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置cgi缓冲大小



cgi-timeout
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置cgi脚本超时时间



cgi-index
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个cgi索引文件



cgi-allowed-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 允许的cgi扩展名



cgi-unset
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 取消设置指定的环境变量



cgi-loadlib
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载一个cgi共享库/优化器



cgi-optimize
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用cgi realpath()优化器



cgi-optimized
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用cgi realpath()优化器



cgi-path-info
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 禁用cgi脚本中的PATH_INFO管理



cgi-do-not-kill-on-error
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 发生错误时，不要发送SIGKILL给cgi脚本



cgi-async-max-attempts
**********************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: cgi异步模式下的最大waitpid()尝试（默认是10）




plugin: cheaper_backlog2
========================

plugin: cheaper_busyness
========================

plugin: clock_monotonic
=======================

plugin: clock_realtime
======================

plugin: corerouter
==================

plugin: coroae
==============
coroae
******
``argument``: 必需参数

``parser``: uwsgi_opt_setup_coroae

``help``: 一个快捷方式，启用 Coro::AnyEvent 循环引擎，带指定数量的异步核心和优化参数




plugin: cplusplus
=================

plugin: curl_cron
=================
curl-cron
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url



cron-curl
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url



legion-curl-cron
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url，只有在实例是指定legion的lord时才可运行



legion-cron-curl
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url，只有在实例是指定legion的lord时才可运行



curl-cron-legion
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url，只有在实例是指定legion的lord时才可运行



cron-curl-legion
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: 新增一个cron任务，通过CURL调用指定的url，只有在实例是指定legion的lord时才可运行




plugin: dumbloop
================
dumbloop-modifier1
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 为code_string设置modifier1



dumbloop-code
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为code_string设置加载的脚本



dumbloop-function
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为code_string设置运行的函数




plugin: dummy
=============

plugin: echo
============

plugin: emperor_amqp
====================

plugin: emperor_mongodb
=======================

plugin: emperor_pg
==================

plugin: emperor_zeromq
======================

plugin: example
===============

plugin: exception_log
=====================

plugin: fastrouter
==================
fastrouter
**********
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter

``help``: 在指定的端口上运行fastrouter

``reference``: :doc:`Fastrouter`



fastrouter-processes
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数量的fastrouter进程



fastrouter-workers
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数量的fastrouter进程



fastrouter-zerg
***************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: 附加fastrouter到一个zerg服务器



fastrouter-use-cache
********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 使用uWSGI缓存作为fastrouter的hostname->server映射器



fastrouter-use-pattern
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: 将一个模式用于fastrouter hostname->server映射



fastrouter-use-base
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: 将一个基本目录用于fastrouter hostname->server映射



fastrouter-fallback
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 错误情况下，回退到指定的节点



fastrouter-use-code-string
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: 将code string当成fastrouter的hostname->server映射器



fastrouter-use-socket
*********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: 转发请求到指定的uwsgi socket



fastrouter-to
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 转发请求到指定的uwsgi服务器 (你可以多次指定它以实现负载均衡)



fastrouter-gracetime
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 在指定秒数之后，重试连接到死掉的静态节点



fastrouter-events
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置并发事件的最大数



fastrouter-quiet
****************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 不要报告失败的连接给实例



fastrouter-cheap
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在cheap模式下运行fastrouter



fastrouter-subscription-server
******************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: 在指定地址上运行fastrouter订阅服务器



fastrouter-subscription-slot
****************************
``argument``: 必需参数

``parser``: uwsgi_opt_deprecated

``help``: *** 已弃用 ***



fastrouter-timeout
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置fastrouter超时时间



fastrouter-post-buffering
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 启用fastrouter post buffering



fastrouter-post-buffering-dir
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 将fastrouter缓存的文件到指定的目录中 (空操作，使用TMPDIR env)



fastrouter-stats
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行fastrouter统计信息服务器



fastrouter-stats-server
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行fastrouter统计信息服务器



fastrouter-ss
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行fastrouter统计信息服务器



fastrouter-harakiri
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用fastrouter harakiri



fastrouter-uid
**************
``argument``: 必需参数

``parser``: uwsgi_opt_uid

``help``: 移除fastrouter特权到指定的uid



fastrouter-gid
**************
``argument``: 必需参数

``parser``: uwsgi_opt_gid

``help``: 移除fastrouter特权到指定的gid



fastrouter-resubscribe
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 转发订阅到指定的订阅服务器



fastrouter-resubscribe-bind
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 当重新订阅时，绑定到指定的地址



fastrouter-buffer-size
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置内部缓冲大小 (默认：页大小)



fastrouter-fallback-on-no-key
*****************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 即使找不到订阅键，也要移动到回退节点



fastrouter-force-key
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 跳过uwsgi解析，直接设置一个键




plugin: fiber
=============
fiber
*****
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ruby fiber，作为挂起引擎




plugin: forkptyrouter
=====================
forkptyrouter
*************
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: 在指定地址上运行forkptyrouter



forkpty-router
**************
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: 在指定地址上运行forkptyrouter



forkptyurouter
**************
``argument``: 必需参数

``parser``: uwsgi_opt_forkpty_urouter

``help``: 在指定地址上运行forkptyrouter



forkpty-urouter
***************
``argument``: 必需参数

``parser``: uwsgi_opt_forkpty_urouter

``help``: 在指定地址上运行forkptyrouter



forkptyrouter-command
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 每次连接都运行指定命令 (默认：/bin/sh)



forkpty-router-command
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 每次连接都运行指定命令 (默认：/bin/sh)



forkptyrouter-cmd
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 每次连接都运行指定命令 (默认：/bin/sh)



forkpty-router-cmd
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 每次连接都运行指定命令 (默认：/bin/sh)



forkptyrouter-rows
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: 设置forkptyrouter默认pty窗口行



forkptyrouter-cols
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: 设置forkptyrouter默认pty窗口列



forkptyrouter-processes
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的forkptyrouter进程



forkptyrouter-workers
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的forkptyrouter进程



forkptyrouter-zerg
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: 附加forkptyrouter到一个zerg服务器



forkptyrouter-fallback
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 错误情况下，回退到指定的节点



forkptyrouter-events
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置concufptyent事件的最大数



forkptyrouter-cheap
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在cheap模式下运行forkptyrouter



forkptyrouter-timeout
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置forkptyrouter超时时间



forkptyrouter-stats
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行forkptyrouter 统计信息服务器



forkptyrouter-stats-server
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行forkptyrouter 统计信息服务器



forkptyrouter-ss
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行forkptyrouter 统计信息服务器



forkptyrouter-harakiri
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用forkptyrouter harakiri




plugin: gccgo
=============
go-load
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在进程地址空间中加载一个go共享库，最终为main.main和__go_init_main打补丁



gccgo-load
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在进程地址空间中加载一个go共享库，最终为main.main和__go_init_main打补丁



go-args
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置go命令行参数



gccgo-args
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置go命令行参数



goroutines
**********
``argument``: 必需参数

``parser``: uwsgi_opt_setup_goroutines

``flags``: UWSGI_OPT_THREADS

``help``: 一个快捷方式，为基于goroutine的应用设置优化选项，接收要生成的最大goroutine数为参数




plugin: geoip
=============
geoip-country
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载指定的geoip国家数据库



geoip-city
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载指定的geoip城市数据库



geoip-use-disk
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 不要将geoip数据库缓存在内存中




plugin: gevent
==============
gevent
******
``argument``: 必需参数

``parser``: uwsgi_opt_setup_gevent

``flags``: UWSGI_OPT_THREADS

``help``: 一个快捷方式，启用gevent循环引擎，使用指定数目的异步核心数和优化参数



gevent-monkey-patch
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启动时自动调用gevent.monkey.patch_all()



gevent-early-monkey-patch
*************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在应用加载之前自动调用gevent.monkey.patch_all()



gevent-wait-for-hub
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 等待gevent hub死掉，而不是控制greenlet




plugin: glusterfs
=================
glusterfs-mount
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 在一个uri上虚拟挂载指定的glusterfs卷



glusterfs-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: glusterfs异步模式的超时时间




plugin: graylog2
================

plugin: greenlet
================
greenlet
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用greenlet作为挂起引擎




plugin: gridfs
==============
gridfs-mount
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 在指定挂载点上挂载一个gridfs数据库



gridfs-debug
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: 对每个请求报告gridfs挂载点和项名称 (调试)




plugin: http
============
http
****
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter

``help``: 在指定地址上添加一个http路由器/服务器



httprouter
**********
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter

``help``: 在指定地址上添加一个http路由器/服务器



https
*****
``argument``: 必需参数

``parser``: uwsgi_opt_https

``help``: 在指定地址上添加一个https路由器/服务器，使用指定的证书和密钥



https2
******
``argument``: 必需参数

``parser``: uwsgi_opt_https2

``help``: 使用键值选项添加一个https/spdy路由器/服务器



https-export-cert
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 导出包含原始客户端证书的uwsgi变量HTTPS_CC



https-session-context
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置会话id上下文为指定值



http-to-https
*************
``argument``: 必需参数

``parser``: uwsgi_opt_http_to_https

``help``: 在指定地址上添加一个http路由器/服务器，并重定向所有请求到https



http-processes
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置要生成的http进程数



http-workers
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置要生成的http进程数



http-var
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个key=value项到已生成的uwsgi包



http-to
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 转发请求到指定的节点 (你可以为lb多次指定)



http-zerg
*********
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: 附加http路由器到一个zerg服务器



http-fallback
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 发生错误时，回退到指定节点



http-modifier1
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置uwsgi协议modifier1



http-modifier2
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置uwsgi协议modifier2



http-use-cache
**************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 使用uWSGI缓存作为key->value虚拟机映射器



http-use-pattern
****************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: 使用指定模式来映射请求到unix socket



http-use-base
*************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: 将指定的base用于映射请求到unix socket



http-events
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置并发http异步事件的数目



http-subscription-server
************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: 启用订阅服务器



http-timeout
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置内部http socket超时时间



http-manage-expect
******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_64bit

``help``: 管理Expect HTTP请求头部 (可选地检查Content-Length)



http-keepalive
**************
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``help``: HTTP 1.1 keepalive支持 (非流水) 请求



http-auto-chunked
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在HTTP 1.1 keepalive期间，自动将输出转换成块编码 (如果需要的话)



http-auto-gzip
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果设置uWSGI-Encoding头部为gzip，则自动gzip压缩内容，但是不指定内容大小 (Content-Length/Transfer-Encoding)和Content-Encoding



http-raw-body
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 盲目发送HTTP请求体到后端 (要求后端支持WebSockets和Icecast)



http-websockets
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 自动检测websockets连接，并将会话置于原始模式



http-chunked-input
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 自动检测块输入请求，并将会话置于原始模式



http-use-code-string
********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: 将code string作为hostname->server映射器，用于http路由器



http-use-socket
***************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: 转发请求到指定的uwsgi socket



http-gracetime
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 指定秒数之后，重试连接死掉的静态节点



http-quiet
**********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 不要向实例报告失败的连接



http-cheap
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在cheap模式下运行http路由器



http-stats
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行http路由器统计信息服务器



http-stats-server
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行http路由器统计信息服务器



http-ss
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行http路由器统计信息服务器



http-harakiri
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用http路由器harakiri



http-stud-prefix
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_addr_list

``help``: expect a stud prefix (1byte family + 4/16 bytes address) on connections from the specified address



http-uid
********
``argument``: 必需参数

``parser``: uwsgi_opt_uid

``help``: drop http router privileges to the specified uid



http-gid
********
``argument``: 必需参数

``parser``: uwsgi_opt_gid

``help``: drop http router privileges to the specified gid



http-resubscribe
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward subscriptions to the specified subscription server



http-buffer-size
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置内部缓冲大小 (默认：页大小)



http-server-name-as-http-host
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: force SERVER_NAME to HTTP_HOST



http-headers-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置internal http socket timeout for headers



http-connect-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置internal http socket timeout for backend connections



http-manage-source
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: manage the SOURCE HTTP method placing the session in raw mode



http-enable-proxy-protocol
**************************
``argument``: 可选参数

``parser``: uwsgi_opt_true

``help``: manage PROXY protocol requests



http-backend-http
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use plain http protocol instead of uwsgi for backend nodes



http-manage-rtsp
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: manage RTSP sessions



0x1f
****
``argument``: 0x8b

``shortcut``: -Z_DEFLATED

``help``: 0




plugin: jvm
===========
jvm-main-class
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load the specified class and call its main() function



jvm-opt
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add the specified jvm option



jvm-class
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load the specified class



jvm-classpath
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add the specified directory to the classpath




plugin: jwsgi
=============
jwsgi
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified JWSGI application (syntax class:method)




plugin: ldap
============
ldap
****
``argument``: 必需参数

``parser``: uwsgi_opt_load_ldap

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load configuration from ldap server



ldap-schema
***********
``argument``: 无参数

``parser``: uwsgi_opt_ldap_dump

``flags``: UWSGI_OPT_IMMEDIATE

``help``: dump uWSGI ldap schema



ldap-schema-ldif
****************
``argument``: 无参数

``parser``: uwsgi_opt_ldap_dump_ldif

``flags``: UWSGI_OPT_IMMEDIATE

``help``: dump uWSGI ldap schema in ldif format




plugin: legion_cache_fetch
==========================

plugin: libffi
==============

plugin: libtcc
==============

plugin: logcrypto
=================

plugin: logfile
===============

plugin: logpipe
===============

plugin: logsocket
=================

plugin: logzmq
==============
log-zeromq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: send logs to a zeromq server




plugin: lua
===========
lua
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load lua wsapi app



lua-load
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a lua file



lua-shell
*********
``argument``: 无参数

``parser``: uwsgi_opt_luashell

``help``: run the lua interactive shell (debug.debug())



luashell
********
``argument``: 无参数

``parser``: uwsgi_opt_luashell

``help``: run the lua interactive shell (debug.debug())



lua-gc-freq
***********
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: 设置the lua gc frequency (default: 0, runs after every request)




plugin: matheval
================

plugin: mongodb
===============

plugin: mongodblog
==================

plugin: mongrel2
================
zeromq
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_lazy_socket

``help``: create a mongrel2/zeromq pub/sub pair



zmq
***
``argument``: 必需参数

``parser``: uwsgi_opt_add_lazy_socket

``help``: create a mongrel2/zeromq pub/sub pair



zeromq-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_lazy_socket

``help``: create a mongrel2/zeromq pub/sub pair



zmq-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_lazy_socket

``help``: create a mongrel2/zeromq pub/sub pair



mongrel2
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_lazy_socket

``help``: create a mongrel2/zeromq pub/sub pair




plugin: mono
============
mono-app
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a Mono asp.net app from the specified directory



mono-gc-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: run the Mono GC every <n> requests (default: run after every request)



mono-key
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: select the ApplicationHost based on the specified CGI var



mono-version
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the Mono jit version



mono-config
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the Mono config file



mono-assembly
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified main assembly (default: uwsgi.dll)



mono-exec
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: exec the specified assembly just 在应用加载之前



mono-index
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an asp.net index file




plugin: msgpack
===============

plugin: nagios
==============
nagios
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_NO_INITIAL

``help``: nagios check




plugin: notfound
================
notfound-log
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: log requests to the notfound plugin




plugin: objc_gc
===============

plugin: pam
===========
pam
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the pam service name to use



pam-user
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置a fake user for pam




plugin: php
===========
php-ini
*******
``argument``: 必需参数

``parser``: uwsgi_opt_php_ini

``help``: 设置php.ini path



php-config
**********
``argument``: 必需参数

``parser``: uwsgi_opt_php_ini

``help``: 设置php.ini path



php-ini-append
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 设置php.ini path (append mode)



php-config-append
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 设置php.ini path (append mode)



php-set
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 设置a php config directive



php-index
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: list the php index files



php-docroot
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: force php DOCUMENT_ROOT



php-allowed-docroot
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: list the allowed document roots



php-allowed-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: list the allowed php file extensions



php-allowed-script
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: list the allowed php scripts (require absolute path)



php-server-software
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: force php SERVER_SOFTWARE



php-app
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: force the php file to run at each request



php-app-qs
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: when in app mode force QUERY_STRING to the specified value + REQUEST_URI



php-fallback
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified php script when the request one does not exist



php-app-bypass
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``help``: if the regexp matches the uri the --php-app is bypassed



php-var
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add/overwrite a CGI variable at each request



php-dump-config
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: dump php config (if modified via --php-设置or append options)



php-exec-before
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run specified php code before the requested script



php-exec-begin
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run specified php code before the requested script



php-exec-after
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run specified php code after the requested script



php-exec-end
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run specified php code after the requested script



php-sapi-name
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: hack the sapi name (required for enabling zend opcode cache)



early-php
*********
``argument``: 无参数

``parser``: uwsgi_opt_early_php

``flags``: UWSGI_OPT_IMMEDIATE

``help``: initialize an early perl interpreter shared by all loaders



early-php-sapi-name
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: hack the sapi name (required for enabling zend opcode cache)




plugin: ping
============
ping
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_NO_INITIAL | UWSGI_OPT_NO_SERVER

``help``: ping specified uwsgi host



ping-timeout
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置ping timeout




plugin: psgi
============
psgi
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a psgi app



psgi-enable-psgix-io
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用psgix.io support



perl-no-die-catch
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not catch $SIG{__DIE__}



perl-local-lib
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置perl locallib path



perl-version
************
``argument``: 无参数

``parser``: uwsgi_opt_print

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 打印perl版本



perl-args
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: add items (space separated) to @ARGV



perl-arg
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an item to @ARGV



perl-exec
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: exec the specified perl file before fork()



perl-exec-post-fork
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: exec the specified perl file after fork()



perl-auto-reload
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: 启用perl auto-reloader with the specified frequency



perl-auto-reload-ignore
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: ignore the specified files when auto-reload is enabled



plshell
*******
``argument``: 可选参数

``parser``: uwsgi_opt_plshell

``help``: run a perl interactive shell



plshell-oneshot
***************
``argument``: 无参数

``parser``: uwsgi_opt_plshell

``help``: run a perl interactive shell (one shot)



perl-no-plack
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force the use of do instead of Plack::Util::load_psgi



early-perl
**********
``argument``: 必需参数

``parser``: uwsgi_opt_early_perl

``flags``: UWSGI_OPT_IMMEDIATE

``help``: initialize an early perl interpreter shared by all loaders



early-psgi
**********
``argument``: 必需参数

``parser``: uwsgi_opt_early_psgi

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a psgi app soon after uWSGI initialization



early-perl-exec
***************
``argument``: 必需参数

``parser``: uwsgi_opt_early_exec

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a perl script soon after uWSGI initialization




plugin: pty
===========
pty-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: bind the pty server on the specified address



pty-log
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: send stdout/stderr to the log engine too



pty-input
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: read from original stdin in addition to pty



pty-connect
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_NO_INITIAL

``help``: connect the current terminal to a pty server



pty-uconnect
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_NO_INITIAL

``help``: connect the current terminal to a pty server (using uwsgi protocol)



pty-no-isig
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable ISIG terminal attribute in client mode



pty-exec
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行指定命令 soon after the pty thread is spawned




plugin: pypy
============
pypy-lib
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the path/name of the pypy library



pypy-setup
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the path of the python setup script



pypy-home
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the home of pypy library



pypy-wsgi
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a WSGI module



pypy-wsgi-file
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a WSGI/mod_wsgi file



pypy-ini-paste
**************
``argument``: 必需参数

``parser``: uwsgi_opt_pypy_ini_paste

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a paste.deploy config file containing uwsgi section



pypy-paste
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a paste.deploy config file



pypy-eval
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: evaluate pypy code before fork()



pypy-eval-post-fork
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: evaluate pypy code soon after fork()



pypy-exec
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: execute pypy code from file before fork()



pypy-exec-post-fork
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: execute pypy code from file soon after fork()



pypy-pp
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an item to the pythonpath



pypy-python-path
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an item to the pythonpath



pypy-pythonpath
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an item to the pythonpath




plugin: python
==============
wsgi-file
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load .wsgi file



file
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load .wsgi file



eval
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: eval python code



module
******
``argument``: 必需参数

``shortcut``: -w

``parser``: uwsgi_opt_set_str

``help``: 加载一个WSGI模块



wsgi
****
``argument``: 必需参数

``shortcut``: -w

``parser``: uwsgi_opt_set_str

``help``: 加载一个WSGI模块



callable
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置default WSGI callable name



test
****
``argument``: 必需参数

``shortcut``: -J

``parser``: uwsgi_opt_set_str

``help``: 测试模块导入



home
****
``argument``: 必需参数

``shortcut``: -H

``parser``: uwsgi_opt_set_str

``help``: 设置PYTHONHOME/virtualenv



virtualenv
**********
``argument``: 必需参数

``shortcut``: -H

``parser``: uwsgi_opt_set_str

``help``: 设置PYTHONHOME/virtualenv



venv
****
``argument``: 必需参数

``shortcut``: -H

``parser``: uwsgi_opt_set_str

``help``: 设置PYTHONHOME/virtualenv



pyhome
******
``argument``: 必需参数

``shortcut``: -H

``parser``: uwsgi_opt_set_str

``help``: 设置PYTHONHOME/virtualenv



py-programname
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置Python程序名



py-program-name
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置Python程序名



pythonpath
**********
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``help``: 添加目录(或者glob)到pythonpath



python-path
***********
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``help``: 添加目录(或者glob)到pythonpath



pp
**
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``help``: 添加目录(或者glob)到pythonpath



pymodule-alias
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a python alias module



post-pymodule-alias
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a python module alias after uwsgi module initialization



import
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入一个python模块



pyimport
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入一个python模块



py-import
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入一个python模块



python-import
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入一个python模块



shared-import
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在所有进程中导入一个python模块



shared-pyimport
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在所有进程中导入一个python模块



shared-py-import
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在所有进程中导入一个python模块



shared-python-import
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在所有进程中导入一个python模块



pyargv
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: manually 设置sys.argv



optimize
********
``argument``: 必需参数

``shortcut``: -O

``parser``: uwsgi_opt_set_int

``help``: 设置python optimization level



pecan
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a pecan config file



paste
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a paste.deploy config file



paste-logger
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用paste fileConfig logger



web3
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a web3 app



pump
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a pump app



wsgi-lite
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a wsgi-lite app



ini-paste
*********
``argument``: 必需参数

``parser``: uwsgi_opt_ini_paste

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a paste.deploy config file containing uwsgi section



ini-paste-logged
****************
``argument``: 必需参数

``parser``: uwsgi_opt_ini_paste

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a paste.deploy config file containing uwsgi section (load loggers too)



reload-os-env
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force reload of os.environ at each request



no-site
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not import site module



pyshell
*******
``argument``: 可选参数

``parser``: uwsgi_opt_pyshell

``help``: run an interactive python shell in the uWSGI environment



pyshell-oneshot
***************
``argument``: 可选参数

``parser``: uwsgi_opt_pyshell

``help``: run an interactive python shell in the uWSGI environment (one-shot variant)



python
******
``argument``: 必需参数

``parser``: uwsgi_opt_pyrun

``help``: run a python script in the uWSGI environment



py
**
``argument``: 必需参数

``parser``: uwsgi_opt_pyrun

``help``: run a python script in the uWSGI environment



pyrun
*****
``argument``: 必需参数

``parser``: uwsgi_opt_pyrun

``help``: run a python script in the uWSGI environment



py-tracebacker
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: 启用the uWSGI python tracebacker



py-auto-reload
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: monitor python modules mtime to trigger reload (use only in development)



py-autoreload
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: monitor python modules mtime to trigger reload (use only in development)



python-auto-reload
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: monitor python modules mtime to trigger reload (use only in development)



python-autoreload
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: monitor python modules mtime to trigger reload (use only in development)



py-auto-reload-ignore
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_THREADS|UWSGI_OPT_MASTER

``help``: ignore the specified module during auto-reload scan (can be specified multiple times)



wsgi-env-behaviour
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the strategy for allocating/deallocating the WSGI env



wsgi-env-behavior
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置the strategy for allocating/deallocating the WSGI env



start_response-nodelay
**********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: send WSGI http headers as soon as possible (PEP violation)



wsgi-strict
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: try to be fully PEP compliant disabling optimizations



wsgi-accept-buffer
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 除了string/byte外，接受CPython 兼容buffer对象作为WSGI响应



wsgi-accept-buffers
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 除了string/byte外，接受CPython 兼容buffer对象作为WSGI响应



python-version
**************
``argument``: 无参数

``parser``: uwsgi_opt_pyver

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 报告python版本



python-raw
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载一个python文件来管理原始请求



py-sharedarea
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 从指定大小的python bytearray对象创建一个共享区域



py-call-osafterfork
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 让子进程能够运行cpython来拦截OS信号



early-python
************
``argument``: 无参数

``parser``: uwsgi_early_python

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 尽快加载python VM (对fork服务器有用)



early-pyimport
**************
``argument``: 必需参数

``parser``: uwsgi_early_python_import

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在早期导入一个python模块



early-python-import
*******************
``argument``: 必需参数

``parser``: uwsgi_early_python_import

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 在早期导入一个python模块



early-pythonpath
****************
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 添加目录(或者glob)到pythonpath (立即版本)



early-python-path
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 添加目录(或者glob)到pythonpath (立即版本)




plugin: pyuwsgi
===============

plugin: rack
============
rails
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_POST_BUFFERING

``help``: 加载一个rails <= 2.x的应用



rack
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_POST_BUFFERING

``help``: 加载一个rack应用



ruby-gc-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置ruby GC频率



rb-gc-freq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置ruby GC频率



rb-lib
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个目录到ruby libdir搜索路径



ruby-lib
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个目录到ruby libdir搜索路径



rb-require
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本



ruby-require
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本



rbrequire
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本



rubyrequire
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本



require
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本



shared-rb-require
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本 (共享)



shared-ruby-require
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本 (共享)



shared-rbrequire
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本 (共享)



shared-rubyrequire
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本 (共享)



shared-require
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 导入/需要一个ruby模块/脚本 (共享)



gemset
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载指定gem设置(rvm)



rvm
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载指定gem设置(rvm)



rvm-path
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 在指定的目录中搜索rvm



rbshell
*******
``argument``: 可选参数

``parser``: uwsgi_opt_rbshell

``help``: 运行ruby/irb shell



rbshell-oneshot
***************
``argument``: 无参数

``parser``: uwsgi_opt_rbshell

``help``: 设置ruby/irb shell (一次搞定)




plugin: rados
=============
rados-mount
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 虚拟挂载uri中的指定rados卷



rados-timeout
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 异步操作的超时时间




plugin: rawrouter
=================
rawrouter
*********
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: 在指定端口上运行rawrouter



rawrouter-processes
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的rawrouter进程



rawrouter-workers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的rawrouter进程



rawrouter-zerg
**************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: 附加rawrouter到一个zerg服务器



rawrouter-use-cache
*******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 将uWSGI缓存作为hostname->server映射器用于rawrouter



rawrouter-use-pattern
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: 将一个模式用于rawrouter hostname->server映射



rawrouter-use-base
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: 将一个基本目录用于rawrouter hostname->server映射



rawrouter-fallback
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 发生错误时，回退到指定节点



rawrouter-use-code-string
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: 将code string作为hostname->server映射器，用于rawrouter



rawrouter-use-socket
********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: 转发请求到指定的uwsgi socket



rawrouter-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 转发请求到指定的uwsgi服务器 (你可以为负载均衡多次指定它)



rawrouter-gracetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 指定秒数hour，重试连接到死亡静态节点



rawrouter-events
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置并发事件的最大数



rawrouter-max-retries
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置重试/回退到其他节点的最大次数



rawrouter-quiet
***************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 不要向实例报告失败的连接



rawrouter-cheap
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在cheap模式下运行rawrouter



rawrouter-subscription-server
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: 在指定的地址上运行rawrouter订阅服务器



rawrouter-subscription-slot
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_deprecated

``help``: *** 已弃用 ***



rawrouter-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置rawrouter超时时间



rawrouter-stats
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行rawrouter统计信息服务器



rawrouter-stats-server
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行rawrouter统计信息服务器



rawrouter-ss
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行rawrouter统计信息服务器



rawrouter-harakiri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用rawrouter harakiri



rawrouter-xclient
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 使用xclient协议来传递客户端地址



rawrouter-buffer-size
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置内部缓冲大小 (默认：页大小)




plugin: rbthreads
=================
rbthreads
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ruby原生线程



rb-threads
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ruby原生线程



rbthread
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ruby原生线程



rb-thread
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ruby原生线程




plugin: redislog
================

plugin: ring
============
ring-load
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载指定的clojure脚本



clojure-load
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载指定的clojure脚本



ring-app
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 映射指定的ring应用 (语法：namespace:function)




plugin: router_access
=====================

plugin: router_basicauth
========================

plugin: router_cache
====================

plugin: router_expires
======================

plugin: router_hash
===================

plugin: router_http
===================

plugin: router_memcached
========================

plugin: router_metrics
======================

plugin: router_radius
=====================

plugin: router_redirect
=======================

plugin: router_redis
====================

plugin: router_rewrite
======================

plugin: router_spnego
=====================

plugin: router_static
=====================

plugin: router_uwsgi
====================

plugin: router_xmldir
=====================

plugin: rpc
===========

plugin: rrdtool
===============
rrdtool
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER|UWSGI_OPT_METRICS

``help``: 在指定的目录中存储rrd文件



rrdtool-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置收集频率



rrdtool-lib
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置rrd库的名称 (默认：librrd.so)




plugin: rsyslog
===============
rsyslog-packet-size
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置syslog消息的最大包大小 (默认1024) 警告！使用包大小 > 1024会违反RFC 3164 (#4.1)



rsyslog-split-messages
**********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 如果大消息比允许的包大小大，则将它分割到多个块 (默认是false)




plugin: ruby19
==============

plugin: servlet
===============

plugin: signal
==============

plugin: spooler
===============

plugin: sqlite3
===============
sqlite3
*******
``argument``: 必需参数

``parser``: uwsgi_opt_load_sqlite3

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从文件加载配置sqlite3 db



sqlite
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_sqlite3

``flags``: UWSGI_OPT_IMMEDIATE

``help``: 从文件加载配置sqlite3 db




plugin: ssi
===========

plugin: sslrouter
=================
sslrouter
*********
``argument``: 必需参数

``parser``: uwsgi_opt_sslrouter

``help``: 在指定端口上运行sslrouter



sslrouter2
**********
``argument``: 必需参数

``parser``: uwsgi_opt_sslrouter2

``help``: 在指定端口上运行sslrouter (基于键值)



sslrouter-session-context
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 设置会话id上下文到指定的值



sslrouter-processes
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的sslrouter进程



sslrouter-workers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork指定数目的sslrouter进程



sslrouter-zerg
**************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: 附加sslrouter到一个zerg服务器



sslrouter-use-cache
*******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 将uWSGI缓存作为hostname->server映射器用于sslrouter



sslrouter-use-pattern
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: 对sslrouter hostname->server映射使用一个模式



sslrouter-use-base
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: 对sslrouter hostname->server映射使用一个基本目录



sslrouter-fallback
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 发生错误时，回退到指定节点



sslrouter-use-code-string
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: 将code string作为hostname->server映射器，用于sslrouter



sslrouter-use-socket
********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: 转发请求到指定的uwsgi socket



sslrouter-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 转发请求到指定的uwsgi服务器 (你可以为负载均衡多次指定它)



sslrouter-gracetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 指定秒数hour，重试连接到死亡静态节点



sslrouter-events
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置并发时间的最大数



sslrouter-max-retries
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 对其他节点设置最大重试/回退数



sslrouter-quiet
***************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: 不要向实例报告失败的连接



sslrouter-cheap
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 在cheap模式下运行sslrouter



sslrouter-subscription-server
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: 在指定地址上运行sslrouter订阅服务器



sslrouter-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置sslrouter超时时间



sslrouter-stats
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行sslrouter统计信息服务器



sslrouter-stats-server
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行sslrouter统计信息服务器



sslrouter-ss
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行sslrouter统计信息服务器



sslrouter-harakiri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 启用sslrouter harakiri



sslrouter-sni
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 使用SNI来路由请求



sslrouter-buffer-size
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置内部缓冲大小 (默认：页大小)




plugin: stackless
=================
stackless
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 使用stackless作为挂起引擎




plugin: stats_pusher_file
=========================

plugin: stats_pusher_mongodb
============================

plugin: stats_pusher_socket
===========================

plugin: stats_pusher_statsd
===========================

plugin: symcall
===============
symcall
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 将指定的C符号作为symcall请求处理器 (也支持<mountpoint=func>)



symcall-use-next
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 搜索符号时使用RTLD_NEXT



symcall-register-rpc
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 将指定的C符号作为RPC函数加载 (语法：name function)



symcall-post-fork
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 每次fork()之后调用指定的C符号




plugin: syslog
==============

plugin: systemd_logger
======================

plugin: tornado
===============
tornado
*******
``argument``: 必需参数

``parser``: uwsgi_opt_setup_tornado

``flags``: UWSGI_OPT_THREADS

``help``: 一个快捷方式，用指定核心数和优化参数启用tornado循环引擎




plugin: transformation_chunked
==============================

plugin: transformation_gzip
===========================

plugin: transformation_offload
==============================

plugin: transformation_template
===============================

plugin: transformation_tofile
=============================

plugin: transformation_toupper
==============================

plugin: tuntap
==============
tuntap-router
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 运行tuntap路由器 (语法：<device> <socket> [stats] [gateway])



tuntap-device
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 添加一个tuntap设备到实例 (语法：<device>[ <socket>])



tuntap-use-credentials
**********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: 启用对tuntap客户端/服务器的SCM_CREDENTIALS检查



tuntap-router-firewall-in
*************************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_firewall

``help``: 添加一个防火墙规则到tuntap路由器 (语法：<action> <src/mask> <dst/mask>)



tuntap-router-firewall-out
**************************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_firewall

``help``: 添加一个防火墙规则到tuntap路由器 (语法：<action> <src/mask> <dst/mask>)



tuntap-router-route
*******************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_route

``help``: 新增一个路由规则到tuntap路由器 (语法：<src/mask> <dst/mask> <gateway>)



tuntap-router-stats
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 运行tuntap路由器统计信息服务器



tuntap-device-rule
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 新增一个tuntap设备规则 (语法：<direction> <src/mask> <dst/mask> <action> [target])




plugin: ugreen
==============
ugreen
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: 启用ugreen协程子系统



ugreen-stacksize
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 设置ugreen栈大小，以页为单位




plugin: v8
==========
v8-load
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 加载一个javascript文件



v8-preemptive
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: 指定频率下将v8置于preemptive move (单个隔离) 



v8-gc-freq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: 设置v8垃圾回收频率



v8-module-path
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 设置v8模块搜索路径



v8-jsgi
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 加载指定的JSGI 3.0应用




plugin: webdav
==============
webdav-mount
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 映射一个文件系统目录作为webdav仓库



webdav-css
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 为自动webdav目录列表添加一个css url



webdav-javascript
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 为自动webdav目录列表添加一个javascript url



webdav-js
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 为自动webdav目录列表添加一个javascript url



webdav-class-directory
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME

``help``: 为自动webdav目录列表设置css目录类



webdav-div
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME

``help``: 为自动webdav目录列表设置div id



webdav-lock-cache
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME

``help``: 为webdav锁设置使用的缓存



webdav-principal-base
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME

``help``: 使用指定的基础启用WebDAV当前主要扩展



webdav-add-option
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV标准到OPTIONS响应中



webdav-add-prop
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有资源中



webdav-add-collection-prop
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有集合中



webdav-add-object-prop
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有对象中



webdav-add-prop-href
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有资源中 (href值)



webdav-add-collection-prop-href
*******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有集合中 (href值)



webdav-add-object-prop-href
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有对象中 (href值)



webdav-add-prop-comp
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有资源中 (xml值)



webdav-add-collection-prop-comp
*******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有集合中 (xml值)



webdav-add-object-prop-comp
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV属性到所有对象中 (xml值)



webdav-add-rtype-prop
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV资源类型属性到所有资源中



webdav-add-rtype-collection-prop
********************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV资源类型属性到所有集合中



webdav-add-rtype-object-prop
****************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 添加一个WebDAV资源类型属性到所有对象中



webdav-skip-prop
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: 如果指定的prop在资源xattr中可用的话，不要添加它




plugin: xattr
=============

plugin: xslt
============
xslt-docroot
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 为xslt处理添加一个document_root



xslt-ext
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 搜索带指定扩展名的xslt样式表



xslt-var
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 从指定的请求变量中获取xslt样式表路径



xslt-stylesheet
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 如果找不到xslt样式表，那么使用指定的



xslt-content-type
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 为xslt结果设置ontent-type (默认是：text/html)




plugin: zabbix
==============
zabbix-template
***************
``argument``: 可选参数

``parser``: uwsgi_opt_zabbix_template

``flags``: UWSGI_OPT_METRICS

``help``: 为当前度量设置打印（或者存储到文件）zabbix模板




plugin: zergpool
================
zergpool
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 为指定的地址启动一个指定地址上的zergpool



zerg-pool
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: 为指定的地址启动一个指定地址上的zergpool



