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

``help``: bind to the specified UNIX/TCP socket using persistent uwsgi protocol (puwsgi)



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

``help``: search for text file (vassal name + extension) containing the on demand socket name



emperor-on-demand-ext
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 搜索包含即需socket名的文本文件 (vassal名 + 扩展)



emperor-on-demand-directory
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 启用在即需模式下，绑定到enable on demand mode binding to the unix socket in the specified directory named like the vassal + .socket



emperor-on-demand-dir
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: enable on demand mode binding to the unix socket in the specified directory named like the vassal + .socket



emperor-on-demand-exec
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: use the output of the specified command as on demand socket name (the vassal name is passed as the only argument)



emperor-extra-extension
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: allows the specified extension in the Emperor (vassal will be called with --config)



emperor-extra-ext
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: allows the specified extension in the Emperor (vassal will be called with --config)



emperor-no-blacklist
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable Emperor blacklisting subsystem



emperor-use-clone
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: use clone() instead of fork() passing the specified unshare() flags



emperor-use-fork-server
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: connect to the specified fork server instead of using plain fork() for new vassals



vassal-fork-base
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: use plain fork() for the specified vassal (instead of a fork-server)



emperor-subreaper
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force the Emperor to be a sub-reaper (if supported)



emperor-cap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: set vassals capability



vassals-cap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: set vassals capability



vassal-cap
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_emperor_cap

``help``: set vassals capability



emperor-collect-attribute
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: collect the specified vassal attribute from imperial monitors



emperor-collect-attr
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: collect the specified vassal attribute from imperial monitors



emperor-fork-server-attr
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the vassal's attribute to get when checking for fork-server



emperor-wrapper-attr
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the vassal's attribute to get when checking for fork-wrapper



emperor-chdir-attr
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the vassal's attribute to get when checking for chdir



imperial-monitor-list
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled imperial monitors



imperial-monitors-list
**********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled imperial monitors



vassals-inherit
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add config templates to vassals config (uses --inherit)



vassals-include
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: include config templates to vassals config (uses --include instead of --inherit)



vassals-inherit-before
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add config templates to vassals config (uses --inherit, parses before the vassal file)



vassals-include-before
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: include config templates to vassals config (uses --include instead of --inherit, parses before the vassal file)



vassals-start-hook
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command before each vassal starts



vassals-stop-hook
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command after vassal's death



vassal-sos
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: ask emperor for reinforcement when overloaded



vassal-sos-backlog
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: ask emperor for sos if backlog queue has more items than the value specified



vassals-set
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: automatically set the specified option (via --set) for every vassal



vassal-set
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: automatically set the specified option (via --set) for every vassal



heartbeat
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: announce healthiness to the emperor



zeus
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: enable Zeus mode



reload-mercy
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum time (in seconds) we wait for workers and other processes to die during reload/shutdown



worker-reload-mercy
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum time (in seconds) a worker can take to reload/shutdown (default is 60)



mule-reload-mercy
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum time (in seconds) a mule can take to reload/shutdown (default is 60)



exit-on-reload
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force exit even if a reload is requested



die-on-term
***********
``argument``: 无参数

``parser``: uwsgi_opt_deprecated

``help``: exit instead of brutal reload on SIGTERM (no more needed)



force-gateway
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force the spawn of the first registered gateway without a master



help
****
``argument``: 无参数

``shortcut``: -h

``parser``: uwsgi_help

``flags``: UWSGI_OPT_IMMEDIATE

``help``: show this help



usage
*****
``argument``: 无参数

``shortcut``: -h

``parser``: uwsgi_help

``flags``: UWSGI_OPT_IMMEDIATE

``help``: show this help



print-sym
*********
``argument``: 必需参数

``parser``: uwsgi_print_sym

``flags``: UWSGI_OPT_IMMEDIATE

``help``: print content of the specified binary symbol



print-symbol
************
``argument``: 必需参数

``parser``: uwsgi_print_sym

``flags``: UWSGI_OPT_IMMEDIATE

``help``: print content of the specified binary symbol



reaper
******
``argument``: 无参数

``shortcut``: -r

``parser``: uwsgi_opt_true

``help``: call waitpid(-1,...) after each request to get rid of zombies



max-requests
************
``argument``: 必需参数

``shortcut``: -R

``parser``: uwsgi_opt_set_64bit

``help``: reload workers after the specified amount of managed requests



max-requests-delta
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: add (worker_id * delta) to the max_requests value of each worker



min-worker-lifetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: number of seconds worker must run before being reloaded (default is 60)



max-worker-lifetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: reload workers after the specified amount of seconds (default is disabled)



socket-timeout
**************
``argument``: 必需参数

``shortcut``: -z

``parser``: uwsgi_opt_set_int

``help``: set internal sockets timeout



no-fd-passing
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable file descriptor passing



locks
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: create the specified number of shared locks



lock-engine
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the lock engine



ftok
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the ipcsem key via ftok() for avoiding duplicates



persistent-ipcsem
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not remove ipcsem's on shutdown



sharedarea
**********
``argument``: 必需参数

``shortcut``: -A

``parser``: uwsgi_opt_add_string_list

``help``: create a raw shared memory area of specified pages (note: it supports keyval too)

``reference``: :doc:`SharedArea`



safe-fd
*******
``argument``: 必需参数

``parser``: uwsgi_opt_safe_fd

``help``: do not close the specified file descriptor



fd-safe
*******
``argument``: 必需参数

``parser``: uwsgi_opt_safe_fd

``help``: do not close the specified file descriptor



cache
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: create a shared cache containing given elements



cache-blocksize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set cache blocksize



cache-store
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable persistent cache to disk



cache-store-sync
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set frequency of sync for persistent cache



cache-no-expire
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable auto sweep of expired items



cache-expire-freq
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the frequency of cache sweeper scans (default 3 seconds)



cache-report-freed-items
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: constantly report the cache item freed by the sweeper (use only for debug)



cache-udp-server
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: bind the cache udp server (used only for set/update/delete) to the specified socket



cache-udp-node
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: send cache update/deletion to the specified cache udp server



cache-sync
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: copy the whole content of another uWSGI cache server on server startup



cache-use-last-modified
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: update last_modified_at timestamp on every cache item modification (default is disabled)



add-cache-item
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an item in the cache



load-file-in-cache
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a static file in the cache



load-file-in-cache-gzip
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a static file in the cache with gzip compression



cache2
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: create a new generation shared cache (keyval syntax)



queue
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable shared queue



queue-blocksize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set queue blocksize



queue-store
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable persistent queue to disk



queue-store-sync
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set frequency of sync for persistent queue



spooler
*******
``argument``: 必需参数

``shortcut``: -Q

``parser``: uwsgi_opt_add_spooler

``flags``: UWSGI_OPT_MASTER

``help``: run a spooler on the specified directory



spooler-external
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_spooler

``flags``: UWSGI_OPT_MASTER

``help``: map spoolers requests to a spooler directory managed by an external instance



spooler-ordered
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: try to order the execution of spooler tasks



spooler-chdir
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: chdir() to specified directory before each spooler task



spooler-processes
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_IMMEDIATE

``help``: set the number of processes for spoolers



spooler-quiet
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not be verbose with spooler tasks



spooler-max-tasks
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of tasks to run before recycling a spooler



spooler-harakiri
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set harakiri timeout for spooler tasks



spooler-frequency
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set spooler frequency, default 30 seconds



spooler-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set spooler frequency, default 30 seconds



mule
****
``argument``: 可选参数

``parser``: uwsgi_opt_add_mule

``flags``: UWSGI_OPT_MASTER

``help``: add a mule



mules
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_mules

``flags``: UWSGI_OPT_MASTER

``help``: add the specified number of mules



farm
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_farm

``flags``: UWSGI_OPT_MASTER

``help``: add a mule farm



mule-msg-size
*************
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set mule message buffer size



signal
******
``argument``: 必需参数

``parser``: uwsgi_opt_signal

``flags``: UWSGI_OPT_IMMEDIATE

``help``: send a uwsgi signal to a server



signal-bufsize
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set buffer size for signal queue



signals-bufsize
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set buffer size for signal queue



signal-timer
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add a timer (syntax: <signal> <seconds>)



timer
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add a timer (syntax: <signal> <seconds>)



signal-rbtimer
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add a redblack timer (syntax: <signal> <seconds>)



rbtimer
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add a redblack timer (syntax: <signal> <seconds>)



rpc-max
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: maximum number of rpc slots (default: 64)



disable-logging
***************
``argument``: 无参数

``shortcut``: -L

``parser``: uwsgi_opt_false

``help``: disable request logging



flock
*****
``argument``: 必需参数

``parser``: uwsgi_opt_flock

``flags``: UWSGI_OPT_IMMEDIATE

``help``: lock the specified file before starting, exit if locked



flock-wait
**********
``argument``: 必需参数

``parser``: uwsgi_opt_flock_wait

``flags``: UWSGI_OPT_IMMEDIATE

``help``: lock the specified file before starting, wait if locked



flock2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: lock the specified file after logging/daemon setup, exit if locked



flock-wait2
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_IMMEDIATE

``help``: lock the specified file after logging/daemon setup, wait if locked



pidfile
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: create pidfile (before privileges drop)



pidfile2
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: create pidfile (after privileges drop)



safe-pidfile
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: create safe pidfile (before privileges drop)



safe-pidfile2
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: create safe pidfile (after privileges drop)



chroot
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: chroot() to the specified directory



pivot-root
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: pivot_root() to the specified directories (new_root and put_old must be separated with a space)



pivot_root
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: pivot_root() to the specified directories (new_root and put_old must be separated with a space)



uid
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_uid

``help``: setuid to the specified user/uid



gid
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_gid

``help``: setgid to the specified group/gid



add-gid
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add the specified group id to the process credentials



immediate-uid
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_immediate_uid

``flags``: UWSGI_OPT_IMMEDIATE

``help``: setuid to the specified user/uid IMMEDIATELY



immediate-gid
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_immediate_gid

``flags``: UWSGI_OPT_IMMEDIATE

``help``: setgid to the specified group/gid IMMEDIATELY



no-initgroups
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable additional groups set via initgroups()



cap
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_cap

``help``: set process capability



unshare
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: unshare() part of the processes and put it in a new namespace



unshare2
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_unshare

``help``: unshare() part of the processes and put it in a new namespace after rootfs change



setns-socket
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: expose a unix socket returning namespace fds from /proc/self/ns



setns-socket-skip
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: skip the specified entry when sending setns file descriptors



setns-skip
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: skip the specified entry when sending setns file descriptors



setns
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: join a namespace created by an external uWSGI instance



setns-preopen
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: open /proc/self/ns as soon as possible and cache fds



fork-socket
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: suspend the execution after early initialization and fork() at every unix socket connection



fork-server
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: suspend the execution after early initialization and fork() at every unix socket connection



jailed
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: mark the instance as jailed (force the execution of post_jail hooks)



jail
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: put the instance in a FreeBSD jail



jail-ip4
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an ipv4 address to the FreeBSD jail



jail-ip6
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an ipv6 address to the FreeBSD jail



jidfile
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: save the jid of a FreeBSD jail in the specified file



jid-file
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: save the jid of a FreeBSD jail in the specified file



jail2
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an option to the FreeBSD jail



libjail
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an option to the FreeBSD jail



jail-attach
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: attach to the FreeBSD jail



refork
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again after privileges drop. Useful for jailing systems



re-fork
*******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again after privileges drop. Useful for jailing systems



refork-as-root
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again before privileges drop. Useful for jailing systems



re-fork-as-root
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again before privileges drop. Useful for jailing systems



refork-post-jail
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again after jailing. Useful for jailing systems



re-fork-post-jail
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fork() again after jailing. Useful for jailing systems



hook-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook as soon as possible



hook-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before jailing



hook-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after jailing



hook-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook in jail after initialization



hook-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before privileges drop



hook-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after privileges drop



hook-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before app exit and reload



hook-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before app loading



hook-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after app loading



hook-post-fork
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after each fork



hook-accepting
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after each worker enter the accepting phase



hook-accepting1
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after the first worker enters the accepting phase



hook-accepting-once
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after each worker enter the accepting phase (once per-instance)



hook-accepting1-once
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook after the first worker enters the accepting phase (once per instance)



hook-master-start
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the Master starts



hook-touch
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the specified file is touched (syntax: <file> <action>)



hook-emperor-start
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the Emperor starts



hook-emperor-stop
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the Emperor send a stop message



hook-emperor-reload
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the Emperor send a reload message



hook-emperor-lost
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook when the Emperor connection is lost



hook-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before exec()ing the vassal



hook-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook in the emperor after the vassal has been started



hook-as-on-demand-vassal
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook whenever a vassal enters on-demand mode



hook-as-on-config-vassal
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook whenever the emperor detects a config change for an on-demand vassal



hook-as-emperor-before-vassal
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook before the new vassal is spawned



hook-as-vassal-before-drop
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook into vassal, before dropping its privileges



hook-as-emperor-setns
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook in the emperor entering vassal namespace



hook-as-mule
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook in each mule



hook-as-gateway
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified hook in each gateway



after-request-hook
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified function/symbol after each request



after-request-call
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified function/symbol after each request



exec-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command as soon as possible



exec-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command before jailing



exec-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command after jailing



exec-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command in jail after initialization



exec-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command before privileges drop



exec-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command after privileges drop



exec-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command before app exit and reload



exec-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command before app loading



exec-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command after app loading



exec-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command before exec()ing the vassal



exec-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: run the specified command in the emperor after the vassal has been started



mount-asap
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem as soon as possible



mount-pre-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem before jailing



mount-post-jail
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem after jailing



mount-in-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem in jail after initialization



mount-as-root
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem before privileges drop



mount-as-vassal
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem before exec()ing the vassal



mount-as-emperor
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: mount filesystem in the emperor after the vassal has been started



umount-asap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem as soon as possible



umount-pre-jail
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem before jailing



umount-post-jail
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem after jailing



umount-in-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem in jail after initialization



umount-as-root
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem before privileges drop



umount-as-vassal
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem before exec()ing the vassal



umount-as-emperor
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unmount filesystem in the emperor after the vassal has been started



wait-for-interface
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified network interface to come up before running root hooks



wait-for-interface-timeout
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the timeout for wait-for-interface



wait-interface
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified network interface to come up before running root hooks



wait-interface-timeout
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the timeout for wait-for-interface



wait-for-iface
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified network interface to come up before running root hooks



wait-for-iface-timeout
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the timeout for wait-for-interface



wait-iface
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified network interface to come up before running root hooks



wait-iface-timeout
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the timeout for wait-for-interface



wait-for-fs
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified filesystem item to appear before running root hooks



wait-for-file
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified file to appear before running root hooks



wait-for-dir
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified directory to appear before running root hooks



wait-for-mountpoint
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: wait for the specified mountpoint to appear before running root hooks



wait-for-fs-timeout
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the timeout for wait-for-fs/file/dir



call-asap
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function as soon as possible



call-pre-jail
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function before jailing



call-post-jail
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function after jailing



call-in-jail
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function in jail after initialization



call-as-root
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function before privileges drop



call-as-user
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function after privileges drop



call-as-user-atexit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function before app exit and reload



call-pre-app
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function before app loading



call-post-app
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function after app loading



call-as-vassal
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function() before exec()ing the vassal



call-as-vassal1
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function before exec()ing the vassal



call-as-vassal3
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function(char *, uid_t, gid_t) before exec()ing the vassal



call-as-emperor
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function() in the emperor after the vassal has been started



call-as-emperor1
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function in the emperor after the vassal has been started



call-as-emperor2
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function(char *, pid_t) in the emperor after the vassal has been started



call-as-emperor4
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified function(char *, pid_t, uid_t, gid_t) in the emperor after the vassal has been started



ini
***
``argument``: 必需参数

``parser``: uwsgi_opt_load_ini

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from ini file



yaml
****
``argument``: 必需参数

``shortcut``: -y

``parser``: uwsgi_opt_load_yml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from yaml file



yml
***
``argument``: 必需参数

``shortcut``: -y

``parser``: uwsgi_opt_load_yml

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from yaml file



json
****
``argument``: 必需参数

``shortcut``: -j

``parser``: uwsgi_opt_load_json

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from json file



js
**
``argument``: 必需参数

``shortcut``: -j

``parser``: uwsgi_opt_load_json

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from json file



weight
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: weight of the instance (used by clustering/lb/subscriptions)



auto-weight
***********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: set weight of the instance (used by clustering/lb/subscriptions) automatically



no-server
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force no-server mode



command-mode
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_IMMEDIATE

``help``: force command mode



no-defer-accept
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable deferred-accept on sockets



tcp-nodelay
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable TCP NODELAY on each request



so-keepalive
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable TCP KEEPALIVEs



so-send-timeout
***************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: set SO_SNDTIMEO



socket-send-timeout
*******************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: set SO_SNDTIMEO



so-write-timeout
****************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: set SO_SNDTIMEO



socket-write-timeout
********************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: set SO_SNDTIMEO



socket-sndbuf
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set SO_SNDBUF



socket-rcvbuf
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set SO_RCVBUF



limit-as
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``help``: limit processes address space/vsz



limit-nproc
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: limit the number of spawnable processes



reload-on-as
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MEMORY

``help``: reload if address space is higher than specified megabytes



reload-on-rss
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MEMORY

``help``: reload if rss memory is higher than specified megabytes



evil-reload-on-as
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_MEMORY

``help``: force the master to reload a worker if its address space is higher than specified megabytes



evil-reload-on-rss
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_megabytes

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_MEMORY

``help``: force the master to reload a worker if its rss memory is higher than specified megabytes



reload-on-fd
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: reload if the specified file descriptor is ready



brutal-reload-on-fd
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: brutal reload if the specified file descriptor is ready



ksm
***
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``help``: enable Linux KSM



pcre-jit
********
``argument``: 无参数

``parser``: uwsgi_opt_pcre_jit

``flags``: UWSGI_OPT_IMMEDIATE

``help``: enable pcre jit (if available)



never-swap
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: lock all memory pages avoiding swapping



touch-reload
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: reload uWSGI if the specified file is modified/touched



touch-workers-reload
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: trigger reload of (only) workers if the specified file is modified/touched



touch-chain-reload
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: trigger chain reload if the specified file is modified/touched



touch-logrotate
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: trigger logrotation if the specified file is modified/touched



touch-logreopen
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: trigger log reopen if the specified file is modified/touched



touch-exec
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: run command when the specified file is modified/touched (syntax: file command)



touch-signal
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: signal when the specified file is modified/touched (syntax: file signal)



fs-reload
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: graceful reload when the specified filesystem object is modified



fs-brutal-reload
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: brutal reload when the specified filesystem object is modified



fs-signal
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise a uwsgi signal when the specified filesystem object is modified (syntax: file signal)



check-mountpoint
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: destroy the instance if a filesystem is no more reachable (useful for reliable Fuse management)



mountpoint-check
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: destroy the instance if a filesystem is no more reachable (useful for reliable Fuse management)



check-mount
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: destroy the instance if a filesystem is no more reachable (useful for reliable Fuse management)



mount-check
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: destroy the instance if a filesystem is no more reachable (useful for reliable Fuse management)



propagate-touch
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: over-engineering option for system with flaky signal management



limit-post
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: limit request body



no-orphans
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically kill workers if master dies (can be dangerous for availability)



prio
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_rawint

``help``: set processes/threads priority



cpu-affinity
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set cpu affinity



post-buffering
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: enable post buffering



post-buffering-bufsize
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set buffer size for read() in post buffering mode



body-read-warning
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set the amount of allowed memory allocation (in megabytes) for request body before starting printing a warning



upload-progress
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: enable creation of .json files in the specified directory during a file upload



no-default-app
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not fallback to default app



manage-script-name
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically rewrite SCRIPT_NAME and PATH_INFO



ignore-script-name
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: ignore SCRIPT_NAME



catch-exceptions
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: report exception as http output (discouraged, use only for testing)



reload-on-exception
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: reload a worker when an exception is raised



reload-on-exception-type
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: reload a worker when a specific exception type is raised



reload-on-exception-value
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: reload a worker when a specific exception value is raised



reload-on-exception-repr
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: reload a worker when a specific exception type+value (language-specific) is raised



exception-handler
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add an exception handler



enable-metrics
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: enable metrics subsystem



metric
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: add a custom metric



metric-threshold
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: add a metric threshold/alarm



metric-alarm
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: add a metric threshold/alarm



alarm-metric
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: add a metric threshold/alarm



metrics-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: export metrics as text files to the specified directory



metrics-dir-restore
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: restore last value taken from the metrics dir



metric-dir
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: export metrics as text files to the specified directory



metric-dir-restore
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: restore last value taken from the metrics dir



metrics-no-cores
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_METRICS|UWSGI_OPT_MASTER

``help``: disable generation of cores-related metrics

``reference``: :doc:`Metrics`



Do not expose metrics of async cores.

udp
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: run the udp server on the specified address



stats
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable the stats server on the specified address



stats-server
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable the stats server on the specified address



stats-http
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: prefix stats server json output with http headers



stats-minified
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: minify statistics json output



stats-min
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: minify statistics json output



stats-push
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER|UWSGI_OPT_METRICS

``help``: push the stats json to the specified destination



stats-pusher-default-freq
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the default frequency of stats pushers



stats-pushers-default-freq
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the default frequency of stats pushers



stats-no-cores
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: disable generation of cores-related stats

``reference``: :doc:`Metrics`



Do not expose the information about cores in the stats server.

stats-no-metrics
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: do not include metrics in stats output

``reference``: :doc:`Metrics`



Do not expose the metrics at all in the stats server.

multicast
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: subscribe to specified multicast group



multicast-ttl
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set multicast ttl



multicast-loop
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set multicast loop (default 1)



master-fifo
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: enable the master fifo



notify-socket
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable the notification socket



subscription-notify-socket
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: set the notification socket for subscriptions



subscription-mountpoints
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: enable mountpoints support for subscription system



subscription-mountpoint
***********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: enable mountpoints support for subscription system



legion
******
``argument``: 必需参数

``parser``: uwsgi_opt_legion

``flags``: UWSGI_OPT_MASTER

``help``: became a member of a legion



legion-mcast
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_mcast

``flags``: UWSGI_OPT_MASTER

``help``: became a member of a legion (shortcut for multicast)



legion-node
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_node

``flags``: UWSGI_OPT_MASTER

``help``: add a node to a legion



legion-freq
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the frequency of legion packets



legion-tolerance
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the tolerance of legion subsystem



legion-death-on-lord-error
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: declare itself as a dead node for the specified amount of seconds if one of the lord hooks fails



legion-skew-tolerance
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the clock skew tolerance of legion subsystem (default 30 seconds)



legion-lord
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on Lord election



legion-unlord
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on Lord dismiss



legion-setup
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on legion setup



legion-death
************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on legion death (shutdown of the instance)



legion-join
***********
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on legion join (first time quorum is reached)



legion-node-joined
******************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call on new node joining legion



legion-node-left
****************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_hook

``flags``: UWSGI_OPT_MASTER

``help``: action to call node leaving legion



legion-quorum
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_quorum

``flags``: UWSGI_OPT_MASTER

``help``: set the quorum of a legion



legion-scroll
*************
``argument``: 必需参数

``parser``: uwsgi_opt_legion_scroll

``flags``: UWSGI_OPT_MASTER

``help``: set the scroll of a legion



legion-scroll-max-size
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: set max size of legion scroll buffer



legion-scroll-list-max-size
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set max size of legion scroll list buffer



subscriptions-sign-check
************************
``argument``: 必需参数

``parser``: uwsgi_opt_scd

``flags``: UWSGI_OPT_MASTER

``help``: set digest algorithm and certificate directory for secured subscription system



subscriptions-sign-check-tolerance
**********************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the maximum tolerance (in seconds) of clock skew for secured subscription system



subscriptions-sign-skip-uid
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: skip signature check for the specified uid when using unix sockets credentials



subscriptions-credentials-check
*******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: add a directory to search for subscriptions key credentials



subscriptions-use-credentials
*****************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable management of SCM_CREDENTIALS in subscriptions UNIX sockets



subscription-algo
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_ssa

``help``: set load balancing algorithm for the subscription system



subscription-dotsplit
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: try to fallback to the next part (dot based) in subscription key



subscribe-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: subscribe to the specified subscription server



st
**
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: subscribe to the specified subscription server



subscribe
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: subscribe to the specified subscription server



subscribe2
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: subscribe to the specified subscription server using advanced keyval syntax



subscribe-freq
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: send subscription announce at the specified interval



subscription-tolerance
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set tolerance for subscription servers



unsubscribe-on-graceful-reload
******************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: force unsubscribe request even during graceful reload



start-unsubscribed
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: configure subscriptions but do not send them (useful with master fifo)



subscribe-with-modifier1
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: force the specififed modifier1 when subscribing



snmp
****
``argument``: 可选参数

``parser``: uwsgi_opt_snmp

``help``: enable the embedded snmp server



snmp-community
**************
``argument``: 必需参数

``parser``: uwsgi_opt_snmp_community

``help``: set the snmp community string



ssl-verbose
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: be verbose about SSL errors



ssl-sessions-use-cache
**********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: use uWSGI cache for ssl sessions storage



ssl-session-use-cache
*********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: use uWSGI cache for ssl sessions storage



ssl-sessions-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set SSL sessions timeout (default: 300 seconds)



ssl-session-timeout
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set SSL sessions timeout (default: 300 seconds)



sni
***
``argument``: 必需参数

``parser``: uwsgi_opt_sni

``help``: add an SNI-governed SSL context



sni-dir
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: check for cert/key/client_ca file in the specified directory and create a sni/ssl context on demand



sni-dir-ciphers
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set ssl ciphers for sni-dir option



ssl-enable3
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable SSLv3 (insecure)



ssl-option
**********
``argument``: 无参数

``parser``: uwsgi_opt_add_string_list

``help``: set a raw ssl option (numeric value)



sni-regexp
**********
``argument``: 必需参数

``parser``: uwsgi_opt_sni

``help``: add an SNI-governed SSL context (the key is a regexp)



ssl-tmp-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: store ssl-related temp files in the specified directory



check-interval
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set the interval (in seconds) of master checks



forkbomb-delay
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: sleep for the specified number of seconds when a forkbomb is detected



binary-path
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: force binary path



privileged-binary-patch
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: patch the uwsgi binary with a new command (before privileges drop)



unprivileged-binary-patch
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: patch the uwsgi binary with a new command (after privileges drop)



privileged-binary-patch-arg
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: patch the uwsgi binary with a new command and arguments (before privileges drop)



unprivileged-binary-patch-arg
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: patch the uwsgi binary with a new command and arguments (after privileges drop)



async
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable async mode with specified cores



disable-async-warn-on-queue-full
********************************
``argument``: 无参数

``parser``: uwsgi_opt_false

``help``: Disable printing 'async queue is full' warning messages.



max-fd
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set maximum number of file descriptors (requires root privileges)



logto
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set logfile/udp address



logto2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: log to specified file or udp address after privileges drop



log-format
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set advanced format for request logging



logformat
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set advanced format for request logging



logformat-strftime
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: apply strftime to logformat output



log-format-strftime
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: apply strftime to logformat output



logfile-chown
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: chown logfiles



logfile-chmod
*************
``argument``: 必需参数

``parser``: uwsgi_opt_logfile_chmod

``help``: chmod logfiles



log-syslog
**********
``argument``: 可选参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: log to syslog



log-socket
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: send logs to the specified socket



req-logger
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_req_logger

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: set/append a request logger



logger-req
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_req_logger

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: set/append a request logger



logger
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_logger

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: set/append a logger



logger-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled loggers



loggers-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled loggers



threaded-logger
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: offload log writing to a thread



log-encoder
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: add an item in the log encoder chain



log-req-encoder
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: add an item in the log req encoder chain



log-drain
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: drain (do not show) log lines matching the specified regexp



log-filter
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: show only log lines matching the specified regexp



log-route
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_custom_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: log to the specified named logger if regexp applied on logline matches



log-req-route
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_custom_list

``flags``: UWSGI_OPT_REQ_LOG_MASTER

``help``: log requests to the specified named logger if regexp applied on logline matches



use-abort
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: call abort() on segfault/fpe, could be useful for generating a core dump



alarm
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: create a new alarm, syntax: <alarm> <plugin:args>



alarm-cheap
***********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: use main alarm thread rather than create dedicated threads for curl-based alarms



alarm-freq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: tune the anti-loop alam system (default 3 seconds)



alarm-fd
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when an fd is read for read (by default it reads 1 byte, set 8 for eventfd)



alarm-segfault
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the segmentation fault handler is executed



segfault-alarm
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the segmentation fault handler is executed



alarm-backlog
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



backlog-alarm
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



lq-alarm
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



alarm-lq
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



alarm-listen-queue
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



listen-queue-alarm
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: raise the specified alarm when the socket backlog queue is full



log-alarm
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: raise the specified alarm when a log line matches the specified regexp, syntax: <alarm>[,alarm...] <regexp>



alarm-log
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: raise the specified alarm when a log line matches the specified regexp, syntax: <alarm>[,alarm...] <regexp>



not-log-alarm
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list_custom

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: skip the specified alarm when a log line matches the specified regexp, syntax: <alarm>[,alarm...] <regexp>



not-alarm-log
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list_custom

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_LOG_MASTER

``help``: skip the specified alarm when a log line matches the specified regexp, syntax: <alarm>[,alarm...] <regexp>



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

``help``: delegate logging to master process



log-master-bufsize
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set the buffer size for the master logger. bigger log messages will be truncated



log-master-stream
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: create the master logpipe as SOCK_STREAM



log-master-req-stream
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: create the master requests logpipe as SOCK_STREAM



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

``help``: set logfile name after rotation



logdate
*******
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: prefix logs with date or a strftime string



log-date
********
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: prefix logs with date or a strftime string



log-prefix
**********
``argument``: 可选参数

``parser``: uwsgi_opt_log_date

``help``: prefix logs with a string



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

``help``: log requests with io errors



log-micros
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: report response time in microseconds instead of milliseconds



log-x-forwarded-for
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use the ip from X-Forwarded-For header instead of REMOTE_ADDR



master-as-root
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: leave master process running as root



drop-after-init
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run privileges drop after plugin initialization



drop-after-apps
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run privileges drop after apps loading



force-cwd
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: force the initial working directory to the specified value



binsh
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: override /bin/sh (used by exec hooks, it always fallback to /bin/sh)



chdir
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: 应用加载前chdir到指定目录



chdir2
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: chdir to specified directory after apps loading



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

``help``: 设置cheap模式 (spawn workers only after the first request)



cheaper
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: set cheaper mode (adaptive process spawning)



cheaper-initial
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: set the initial number of processes to spawn in cheaper mode



cheaper-algo
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: choose to algorithm used for adaptive process spawning



cheaper-step
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: number of additional processes to spawn at each overload



cheaper-overload
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: increase workers after specified overload


cheaper-idle
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: decrease workers after specified idle (algo: spare2) (default: 10)


cheaper-algo-list
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled cheapers algorithms



cheaper-algos-list
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled cheapers algorithms



cheaper-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled cheapers algorithms



cheaper-rss-limit-soft
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: don't spawn new workers if total resident memory usage of all workers is higher than this limit



cheaper-rss-limit-hard
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``flags``: UWSGI_OPT_MASTER | UWSGI_OPT_CHEAPER

``help``: if total workers resident memory usage is higher try to stop workers



idle
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MASTER

``help``: set idle mode (put uWSGI in cheap mode after inactivity)



die-on-idle
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: shutdown uWSGI when idle



mount
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load application under mountpoint



worker-mount
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load application under mountpoint in the specified worker or after workers spawn



threads
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: run each worker in prethreaded mode with the specified number of threads



thread-stacksize
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: set threads stacksize



threads-stacksize
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: set threads stacksize



thread-stack-size
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: set threads stacksize



threads-stack-size
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_THREADS

``help``: set threads stacksize



vhost
*****
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable virtualhosting mode (based on SERVER_NAME variable)



vhost-host
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_VHOST

``help``: enable virtualhosting mode (based on HTTP_HOST variable)



route
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route



route-host
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on Host header



route-uri
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on REQUEST_URI



route-qs
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on QUERY_STRING



route-remote-addr
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on REMOTE_ADDR



route-user-agent
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on HTTP_USER_AGENT



route-remote-user
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on REMOTE_USER



route-referer
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on HTTP_REFERER



route-label
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a routing label (for use with goto)



route-if
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on condition



route-if-not
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a route based on condition (negate version)



route-run
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: always run the specified route action



final-route
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route



final-route-status
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route for the specified status



final-route-host
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on Host header



final-route-uri
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on REQUEST_URI



final-route-qs
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on QUERY_STRING



final-route-remote-addr
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on REMOTE_ADDR



final-route-user-agent
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on HTTP_USER_AGENT



final-route-remote-user
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on REMOTE_USER



final-route-referer
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on HTTP_REFERER



final-route-label
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final routing label (for use with goto)



final-route-if
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on condition



final-route-if-not
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a final route based on condition (negate version)



final-route-run
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: always run the specified final route action



error-route
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route



error-route-status
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route for the specified status



error-route-host
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on Host header



error-route-uri
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on REQUEST_URI



error-route-qs
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on QUERY_STRING



error-route-remote-addr
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on REMOTE_ADDR



error-route-user-agent
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on HTTP_USER_AGENT



error-route-remote-user
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on REMOTE_USER



error-route-referer
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on HTTP_REFERER



error-route-label
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error routing label (for use with goto)



error-route-if
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on condition



error-route-if-not
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add an error route based on condition (negate version)



error-route-run
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: always run the specified error route action



response-route
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route



response-route-status
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route for the specified status



response-route-host
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on Host header



response-route-uri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on REQUEST_URI



response-route-qs
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on QUERY_STRING



response-route-remote-addr
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on REMOTE_ADDR



response-route-user-agent
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on HTTP_USER_AGENT



response-route-remote-user
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on REMOTE_USER



response-route-referer
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on HTTP_REFERER



response-route-label
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response routing label (for use with goto)



response-route-if
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on condition



response-route-if-not
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: add a response route based on condition (negate version)



response-route-run
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_route

``help``: always run the specified response route action



router-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled routers



routers-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled routers



error-page-403
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an error page (html) for managed 403 response



error-page-404
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an error page (html) for managed 404 response



error-page-500
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add an error page (html) for managed 500 response



websockets-ping-freq
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the frequency (in seconds) of websockets automatic ping packets



websocket-ping-freq
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the frequency (in seconds) of websockets automatic ping packets



websockets-pong-tolerance
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the tolerance (in seconds) of websockets ping/pong subsystem



websocket-pong-tolerance
************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the tolerance (in seconds) of websockets ping/pong subsystem



websockets-max-size
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set the max allowed size of websocket messages (in Kbytes, default 1024)



websocket-max-size
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set the max allowed size of websocket messages (in Kbytes, default 1024)



chunked-input-limit
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set the max size of a chunked input part (default 1MB, in bytes)



chunked-input-timeout
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set default timeout for chunked input



clock
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set a clock source



clock-list
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled clocks



clocks-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled clocks



add-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: automatically add HTTP headers to response



rem-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: automatically remove specified HTTP header from the response



del-header
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: automatically remove specified HTTP header from the response



collect-header
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: store the specified response header in a request var (syntax: header var)



response-header-collect
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: store the specified response header in a request var (syntax: header var)



pull-header
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: store the specified response header in a request var and remove it from the response (syntax: header var)



check-static
************
``argument``: 必需参数

``parser``: uwsgi_opt_check_static

``flags``: UWSGI_OPT_MIME

``help``: check for static files in the specified directory



check-static-docroot
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: check for static files in the requested DOCUMENT_ROOT



static-check
************
``argument``: 必需参数

``parser``: uwsgi_opt_check_static

``flags``: UWSGI_OPT_MIME

``help``: check for static files in the specified directory



static-map
**********
``argument``: 必需参数

``parser``: uwsgi_opt_static_map

``flags``: UWSGI_OPT_MIME

``help``: map mountpoint to static directory (or file)



static-map2
***********
``argument``: 必需参数

``parser``: uwsgi_opt_static_map

``flags``: UWSGI_OPT_MIME

``help``: like static-map but completely appending the requested resource to the docroot



static-skip-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: skip specified extension from staticfile checks



static-index
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: search for specified file if a directory is requested



static-safe
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: skip security checks if the file is under the specified path



static-cache-paths
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``flags``: UWSGI_OPT_MIME|UWSGI_OPT_MASTER

``help``: put resolved paths in the uWSGI cache for the specified amount of seconds



static-cache-paths-name
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MIME|UWSGI_OPT_MASTER

``help``: use the specified cache for static paths



mimefile
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: set mime types file path (default /etc/apache2/mime.types)



mime-file
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: set mime types file path (default /etc/apache2/mime.types)



mimefile
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: set mime types file path (default /etc/mime.types)



mime-file
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: set mime types file path (default /etc/mime.types)



static-expires-type
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on content type



static-expires-type-mtime
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on content type and file mtime



static-expires
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on filename regexp



static-expires-mtime
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on filename regexp and file mtime



static-expires-uri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on REQUEST_URI regexp



static-expires-uri-mtime
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on REQUEST_URI regexp and file mtime



static-expires-path-info
************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on PATH_INFO regexp



static-expires-path-info-mtime
******************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_dyn_dict

``flags``: UWSGI_OPT_MIME

``help``: set the Expires header based on PATH_INFO regexp and file mtime



static-gzip
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_regexp_list

``flags``: UWSGI_OPT_MIME

``help``: if the supplied regexp matches the static file translation it will search for a gzip version



static-gzip-all
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: check for a gzip version of all requested static files



static-gzip-dir
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: check for a gzip version of all requested static files in the specified dir/prefix



static-gzip-prefix
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: check for a gzip version of all requested static files in the specified dir/prefix



static-gzip-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: check for a gzip version of all requested static files with the specified ext/suffix



static-gzip-suffix
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: check for a gzip version of all requested static files with the specified ext/suffix



honour-range
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable support for the HTTP Range header



offload-threads
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the number of offload threads to spawn (per-worker, default 0)



offload-thread
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the number of offload threads to spawn (per-worker, default 0)



file-serve-mode
***************
``argument``: 必需参数

``parser``: uwsgi_opt_fileserve_mode

``flags``: UWSGI_OPT_MIME

``help``: set static file serving mode



fileserve-mode
**************
``argument``: 必需参数

``parser``: uwsgi_opt_fileserve_mode

``flags``: UWSGI_OPT_MIME

``help``: set static file serving mode



disable-sendfile
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable sendfile() and rely on boring read()/write()



check-cache
***********
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: check for response data in the specified cache (empty for default cache)



close-on-exec
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: set close-on-exec on connection sockets (could be required for spawning processes in requests)



close-on-exec2
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: set close-on-exec on server sockets (could be required for spawning processes in requests)



mode
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set uWSGI custom mode



env
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_env

``help``: set environment variable



ienv
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_env

``flags``: UWSGI_OPT_IMMEDIATE

``help``: set environment variable (IMMEDIATE version)



envdir
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a daemontools compatible envdir



early-envdir
************
``argument``: 必需参数

``parser``: uwsgi_opt_envdir

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load a daemontools compatible envdir ASAP



unenv
*****
``argument``: 必需参数

``parser``: uwsgi_opt_unset_env

``help``: unset environment variable



vacuum
******
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: try to remove all of the generated file/sockets



file-write
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: write the specified content to the specified file (syntax: file=value) before privileges drop



cgroup
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: put the processes in the specified cgroup



cgroup-opt
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: set value in specified cgroup option



cgroup-dir-mode
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set permission for cgroup directory (default is 700)



namespace
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run in a new namespace under the specified rootfs



namespace-keep-mount
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: keep the specified mountpoint in your namespace



ns
**
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run in a new namespace under the specified rootfs



namespace-net
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: add network namespace



ns-net
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: add network namespace



enable-proxy-protocol
*********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable PROXY1 protocol support (only for http parsers)



reuse-port
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable REUSE_PORT flag on socket (BSD only)



tcp-fast-open
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable TCP_FASTOPEN flag on TCP sockets with the specified qlen value



tcp-fastopen
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable TCP_FASTOPEN flag on TCP sockets with the specified qlen value



tcp-fast-open-client
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use sendto(..., MSG_FASTOPEN, ...) instead of connect() if supported



tcp-fastopen-client
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use sendto(..., MSG_FASTOPEN, ...) instead of connect() if supported



zerg
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: attach to a zerg server



zerg-fallback
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: fallback to normal sockets if the zerg server is not available



zerg-server
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_MASTER

``help``: enable the zerg server on the specified UNIX socket



cron
****
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task



cron2
*****
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron2

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task (key=val syntax)



unique-cron
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a unique cron task



cron-harakiri
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum time (in seconds) we wait for cron command to complete



legion-cron
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task runnable only when the instance is a lord of the specified legion



cron-legion
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task runnable only when the instance is a lord of the specified legion



unique-legion-cron
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a unique cron task runnable only when the instance is a lord of the specified legion



unique-cron-legion
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_unique_legion_cron

``flags``: UWSGI_OPT_MASTER

``help``: add a unique cron task runnable only when the instance is a lord of the specified legion



loop
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: select the uWSGI loop engine



loop-list
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled loop engines



loops-list
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled loop engines



worker-exec
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command as worker



worker-exec2
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command as worker (after post_fork hook)



attach-daemon
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: attach a command/daemon to the master process (the command has to not go in background)



attach-control-daemon
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: attach a command/daemon to the master process (the command has to not go in background), when the daemon dies, the master dies too



smart-attach-daemon
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: attach a command/daemon to the master process managed by a pidfile (the command has to daemonize)



smart-attach-daemon2
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: attach a command/daemon to the master process managed by a pidfile (the command has to NOT daemonize)



legion-attach-daemon
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: same as --attach-daemon but daemon runs only on legion lord node



legion-smart-attach-daemon
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: same as --smart-attach-daemon but daemon runs only on legion lord node



legion-smart-attach-daemon2
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon

``flags``: UWSGI_OPT_MASTER

``help``: same as --smart-attach-daemon2 but daemon runs only on legion lord node



daemons-honour-stdin
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MASTER

``help``: do not change the stdin of external daemons to /dev/null



attach-daemon2
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_daemon2

``flags``: UWSGI_OPT_MASTER

``help``: attach-daemon keyval variant (supports smart modes too)



plugins
*******
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load uWSGI plugins



plugin
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load uWSGI plugins



need-plugins
************
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load uWSGI plugins (exit on error)



need-plugin
***********
``argument``: 必需参数

``parser``: uwsgi_opt_load_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load uWSGI plugins (exit on error)



plugins-dir
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: add a directory to uWSGI plugin search path



plugin-dir
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_IMMEDIATE

``help``: add a directory to uWSGI plugin search path



plugins-list
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled plugins



plugin-list
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: list enabled plugins



autoload
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_IMMEDIATE

``help``: try to automatically load plugins when unknown options are found



dlopen
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_dl

``flags``: UWSGI_OPT_IMMEDIATE

``help``: blindly load a shared library



allowed-modifiers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: comma separated list of allowed modifiers



remap-modifier
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: remap request modifier from one id to another



dump-options
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: dump the full list of available options



show-config
***********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: show the current config reformatted as ini



binary-append-data
******************
``argument``: 必需参数

``parser``: uwsgi_opt_binary_append_data

``flags``: UWSGI_OPT_IMMEDIATE

``help``: return the content of a resource to stdout for appending to a uwsgi binary (for data:// usage)



print
*****
``argument``: 必需参数

``parser``: uwsgi_opt_print

``help``: simple print



iprint
******
``argument``: 必需参数

``parser``: uwsgi_opt_print

``flags``: UWSGI_OPT_IMMEDIATE

``help``: simple print (immediate version)



exit
****
``argument``: 可选参数

``parser``: uwsgi_opt_exit

``flags``: UWSGI_OPT_IMMEDIATE

``help``: force exit() of the instance



cflags
******
``argument``: 无参数

``parser``: uwsgi_opt_cflags

``flags``: UWSGI_OPT_IMMEDIATE

``help``: report uWSGI CFLAGS (useful for building external plugins)



dot-h
*****
``argument``: 无参数

``parser``: uwsgi_opt_dot_h

``flags``: UWSGI_OPT_IMMEDIATE

``help``: dump the uwsgi.h used for building the core  (useful for building external plugins)



config-py
*********
``argument``: 无参数

``parser``: uwsgi_opt_config_py

``flags``: UWSGI_OPT_IMMEDIATE

``help``: dump the uwsgiconfig.py used for building the core  (useful for building external plugins)



build-plugin
************
``argument``: 必需参数

``parser``: uwsgi_opt_build_plugin

``flags``: UWSGI_OPT_IMMEDIATE

``help``: build a uWSGI plugin for the current binary



version
*******
``argument``: 无参数

``parser``: uwsgi_opt_print

``help``: print uWSGI version




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

``help``: a shortcut enabling asyncio loop engine with the specified number of async cores and optimal parameters




plugin: cache
=============

plugin: carbon
==============
carbon
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MASTER

``help``: push statistics to the specified carbon server



carbon-timeout
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set carbon connection timeout in seconds (default 3)



carbon-freq
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set carbon push frequency in seconds (default 60)



carbon-id
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set carbon id



carbon-no-workers
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable generation of single worker metrics



carbon-max-retry
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set maximum number of retries in case of connection errors (default 1)



carbon-retry-delay
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set connection retry delay in seconds (default 7)



carbon-root
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set carbon metrics root node (default 'uwsgi')



carbon-hostname-dots
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set char to use as a replacement for dots in hostname (dots are not replaced by default)



carbon-name-resolve
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: allow using hostname as carbon server address (default disabled)



carbon-resolve-names
********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: allow using hostname as carbon server address (default disabled)



carbon-idle-avg
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: average values source during idle period (no requests), can be "last", "zero", "none" (default is last)



carbon-use-metrics
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: don't compute all statistics, use metrics subsystem data instead (warning! key names will be different)




plugin: cgi
===========
cgi
***
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi

``help``: add a cgi mountpoint/directory/script



cgi-map-helper
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi_maphelper

``help``: add a cgi map-helper



cgi-helper
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cgi_maphelper

``help``: add a cgi map-helper



cgi-from-docroot
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: blindly enable cgi in DOCUMENT_ROOT



cgi-buffer-size
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set cgi buffer size



cgi-timeout
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set cgi script timeout



cgi-index
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a cgi index file



cgi-allowed-ext
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: cgi allowed extension



cgi-unset
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: unset specified environment variables



cgi-loadlib
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a cgi shared library/optimizer



cgi-optimize
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable cgi realpath() optimizer



cgi-optimized
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable cgi realpath() optimizer



cgi-path-info
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: disable PATH_INFO management in cgi scripts



cgi-do-not-kill-on-error
************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not send SIGKILL to cgi script on errors



cgi-async-max-attempts
**********************
``argument``: 无参数

``parser``: uwsgi_opt_set_int

``help``: max waitpid() attempts in cgi async mode (default 10)




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

``help``: a shortcut enabling Coro::AnyEvent loop engine with the specified number of async cores and optimal parameters




plugin: cplusplus
=================

plugin: curl_cron
=================
curl-cron
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL



cron-curl
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL



legion-curl-cron
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL runnable only when the instance is a lord of the specified legion



legion-cron-curl
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL runnable only when the instance is a lord of the specified legion



curl-cron-legion
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL runnable only when the instance is a lord of the specified legion



cron-curl-legion
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_legion_cron_curl

``flags``: UWSGI_OPT_MASTER

``help``: add a cron task invoking the specified url via CURL runnable only when the instance is a lord of the specified legion




plugin: dumbloop
================
dumbloop-modifier1
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the modifier1 for the code_string



dumbloop-code
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the script to load for the code_string



dumbloop-function
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the function to run for the code_string




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

``help``: run the fastrouter on the specified port

``reference``: :doc:`Fastrouter`



fastrouter-processes
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of fastrouter processes



fastrouter-workers
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of fastrouter processes



fastrouter-zerg
***************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: attach the fastrouter to a zerg server



fastrouter-use-cache
********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: use uWSGI cache as hostname->server mapper for the fastrouter



fastrouter-use-pattern
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: use a pattern for fastrouter hostname->server mapping



fastrouter-use-base
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: use a base dir for fastrouter hostname->server mapping



fastrouter-fallback
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: fallback to the specified node in case of error



fastrouter-use-code-string
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: use code string as hostname->server mapper for the fastrouter



fastrouter-use-socket
*********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: forward request to the specified uwsgi socket



fastrouter-to
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward requests to the specified uwsgi server (you can specify it multiple times for load balancing)



fastrouter-gracetime
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: retry connections to dead static nodes after the specified amount of seconds



fastrouter-events
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of concurrent events



fastrouter-quiet
****************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: do not report failed connections to instances



fastrouter-cheap
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run the fastrouter in cheap mode



fastrouter-subscription-server
******************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: run the fastrouter subscription server on the specified address



fastrouter-subscription-slot
****************************
``argument``: 必需参数

``parser``: uwsgi_opt_deprecated

``help``: *** deprecated ***



fastrouter-timeout
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set fastrouter timeout



fastrouter-post-buffering
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: enable fastrouter post buffering



fastrouter-post-buffering-dir
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: put fastrouter buffered files to the specified directory (noop, use TMPDIR env)



fastrouter-stats
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the fastrouter stats server



fastrouter-stats-server
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the fastrouter stats server



fastrouter-ss
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the fastrouter stats server



fastrouter-harakiri
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable fastrouter harakiri



fastrouter-uid
**************
``argument``: 必需参数

``parser``: uwsgi_opt_uid

``help``: drop fastrouter privileges to the specified uid



fastrouter-gid
**************
``argument``: 必需参数

``parser``: uwsgi_opt_gid

``help``: drop fastrouter privileges to the specified gid



fastrouter-resubscribe
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward subscriptions to the specified subscription server



fastrouter-resubscribe-bind
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: bind to the specified address when re-subscribing



fastrouter-buffer-size
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set internal buffer size (default: page size)



fastrouter-fallback-on-no-key
*****************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: move to fallback node even if a subscription key is not found



fastrouter-force-key
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: skip uwsgi parsing and directly set a key




plugin: fiber
=============
fiber
*****
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable ruby fiber as suspend engine




plugin: forkptyrouter
=====================
forkptyrouter
*************
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: run the forkptyrouter on the specified address



forkpty-router
**************
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: run the forkptyrouter on the specified address



forkptyurouter
**************
``argument``: 必需参数

``parser``: uwsgi_opt_forkpty_urouter

``help``: run the forkptyrouter on the specified address



forkpty-urouter
***************
``argument``: 必需参数

``parser``: uwsgi_opt_forkpty_urouter

``help``: run the forkptyrouter on the specified address



forkptyrouter-command
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command on every connection (default: /bin/sh)



forkpty-router-command
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command on every connection (default: /bin/sh)



forkptyrouter-cmd
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command on every connection (default: /bin/sh)



forkpty-router-cmd
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the specified command on every connection (default: /bin/sh)



forkptyrouter-rows
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: set forkptyrouter default pty window rows



forkptyrouter-cols
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_16bit

``help``: set forkptyrouter default pty window cols



forkptyrouter-processes
***********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of forkptyrouter processes



forkptyrouter-workers
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of forkptyrouter processes



forkptyrouter-zerg
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: attach the forkptyrouter to a zerg server



forkptyrouter-fallback
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: fallback to the specified node in case of error



forkptyrouter-events
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of concufptyent events



forkptyrouter-cheap
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run the forkptyrouter in cheap mode



forkptyrouter-timeout
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set forkptyrouter timeout



forkptyrouter-stats
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the forkptyrouter stats server



forkptyrouter-stats-server
**************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the forkptyrouter stats server



forkptyrouter-ss
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the forkptyrouter stats server



forkptyrouter-harakiri
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable forkptyrouter harakiri




plugin: gccgo
=============
go-load
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a go shared library in the process address space, eventually patching main.main and __go_init_main



gccgo-load
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load a go shared library in the process address space, eventually patching main.main and __go_init_main



go-args
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set go commandline arguments



gccgo-args
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set go commandline arguments



goroutines
**********
``argument``: 必需参数

``parser``: uwsgi_opt_setup_goroutines

``flags``: UWSGI_OPT_THREADS

``help``: a shortcut setting optimal options for goroutine-based apps, takes the number of max goroutines to spawn as argument




plugin: geoip
=============
geoip-country
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified geoip country database



geoip-city
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified geoip city database



geoip-use-disk
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not cache geoip databases in memory




plugin: gevent
==============
gevent
******
``argument``: 必需参数

``parser``: uwsgi_opt_setup_gevent

``flags``: UWSGI_OPT_THREADS

``help``: a shortcut enabling gevent loop engine with the specified number of async cores and optimal parameters



gevent-monkey-patch
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: call gevent.monkey.patch_all() automatically on startup



gevent-early-monkey-patch
*************************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: call gevent.monkey.patch_all() automatically before app loading



gevent-wait-for-hub
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: wait for gevent hub's death instead of the control greenlet




plugin: glusterfs
=================
glusterfs-mount
***************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: virtual mount the specified glusterfs volume in a uri



glusterfs-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: timeout for glusterfs async mode




plugin: graylog2
================

plugin: greenlet
================
greenlet
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable greenlet as suspend engine




plugin: gridfs
==============
gridfs-mount
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: mount a gridfs db on the specified mountpoint



gridfs-debug
************
``argument``: 无参数

``parser``: uwsgi_opt_true

``flags``: UWSGI_OPT_MIME

``help``: report gridfs mountpoint and itemname for each request (debug)




plugin: http
============
http
****
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter

``help``: add an http router/server on the specified address



httprouter
**********
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter

``help``: add an http router/server on the specified address



https
*****
``argument``: 必需参数

``parser``: uwsgi_opt_https

``help``: add an https router/server on the specified address with specified certificate and key



https2
******
``argument``: 必需参数

``parser``: uwsgi_opt_https2

``help``: add an https/spdy router/server using keyval options



https-export-cert
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: export uwsgi variable HTTPS_CC containing the raw client certificate



https-session-context
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the session id context to the specified value



http-to-https
*************
``argument``: 必需参数

``parser``: uwsgi_opt_http_to_https

``help``: add an http router/server on the specified address and redirect all of the requests to https



http-processes
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the number of http processes to spawn



http-workers
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the number of http processes to spawn



http-var
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a key=value item to the generated uwsgi packet



http-to
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward requests to the specified node (you can specify it multiple time for lb)



http-zerg
*********
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: attach the http router to a zerg server



http-fallback
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: fallback to the specified node in case of error



http-modifier1
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set uwsgi protocol modifier1



http-modifier2
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set uwsgi protocol modifier2



http-use-cache
**************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: use uWSGI cache as key->value virtualhost mapper



http-use-pattern
****************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: use the specified pattern for mapping requests to unix sockets



http-use-base
*************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: use the specified base for mapping requests to unix sockets



http-events
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the number of concurrent http async events



http-subscription-server
************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: enable the subscription server



http-timeout
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set internal http socket timeout



http-manage-expect
******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_64bit

``help``: manage the Expect HTTP request header (optionally checking for Content-Length)



http-keepalive
**************
``argument``: 可选参数

``parser``: uwsgi_opt_set_int

``help``: HTTP 1.1 keepalive support (non-pipelined) requests



http-auto-chunked
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically transform output to chunked encoding during HTTP 1.1 keepalive (if needed)



http-auto-gzip
**************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically gzip content if uWSGI-Encoding header is set to gzip, but content size (Content-Length/Transfer-Encoding) and Content-Encoding are not specified



http-raw-body
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: blindly send HTTP body to backends (required for WebSockets and Icecast support in backends)



http-websockets
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically detect websockets connections and put the session in raw mode



http-chunked-input
******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: automatically detect chunked input requests and put the session in raw mode



http-use-code-string
********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: use code string as hostname->server mapper for the http router



http-use-socket
***************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: forward request to the specified uwsgi socket



http-gracetime
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: retry connections to dead static nodes after the specified amount of seconds



http-quiet
**********
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: do not report failed connections to instances



http-cheap
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run the http router in cheap mode



http-stats
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the http router stats server



http-stats-server
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the http router stats server



http-ss
*******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the http router stats server



http-harakiri
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable http router harakiri



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

``help``: set internal buffer size (default: page size)



http-server-name-as-http-host
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: force SERVER_NAME to HTTP_HOST



http-headers-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set internal http socket timeout for headers



http-connect-timeout
********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set internal http socket timeout for backend connections



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

``help``: set the lua gc frequency (default: 0, runs after every request)




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

``help``: set the Mono jit version



mono-config
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the Mono config file



mono-assembly
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified main assembly (default: uwsgi.dll)



mono-exec
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: exec the specified assembly just before app loading



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

``help``: set the pam service name to use



pam-user
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set a fake user for pam




plugin: php
===========
php-ini
*******
``argument``: 必需参数

``parser``: uwsgi_opt_php_ini

``help``: set php.ini path



php-config
**********
``argument``: 必需参数

``parser``: uwsgi_opt_php_ini

``help``: set php.ini path



php-ini-append
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: set php.ini path (append mode)



php-config-append
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: set php.ini path (append mode)



php-set
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: set a php config directive



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

``help``: dump php config (if modified via --php-set or append options)



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

``help``: set ping timeout




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

``help``: enable psgix.io support



perl-no-die-catch
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: do not catch $SIG{__DIE__}



perl-local-lib
**************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set perl locallib path



perl-version
************
``argument``: 无参数

``parser``: uwsgi_opt_print

``flags``: UWSGI_OPT_IMMEDIATE

``help``: print perl version



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

``help``: enable perl auto-reloader with the specified frequency



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

``help``: run the specified command soon after the pty thread is spawned




plugin: pypy
============
pypy-lib
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the path/name of the pypy library



pypy-setup
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the path of the python setup script



pypy-home
*********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the home of pypy library



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

``help``: set default WSGI callable name



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

``help``: add directory (or glob) to pythonpath



python-path
***********
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``help``: add directory (or glob) to pythonpath



pp
**
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``help``: add directory (or glob) to pythonpath



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

``help``: manually set sys.argv



optimize
********
``argument``: 必需参数

``shortcut``: -O

``parser``: uwsgi_opt_set_int

``help``: set python optimization level



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

``help``: enable paste fileConfig logger



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

``help``: enable the uWSGI python tracebacker



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

``help``: set the strategy for allocating/deallocating the WSGI env



wsgi-env-behavior
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the strategy for allocating/deallocating the WSGI env



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

``help``: accept CPython buffer-compliant objects as WSGI response in addition to string/bytes



wsgi-accept-buffers
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: accept CPython buffer-compliant objects as WSGI response in addition to string/bytes



python-version
**************
``argument``: 无参数

``parser``: uwsgi_opt_pyver

``flags``: UWSGI_OPT_IMMEDIATE

``help``: report python version



python-raw
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load a python file for managing raw requests



py-sharedarea
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: create a sharedarea from a python bytearray object of the specified size



py-call-osafterfork
*******************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable child processes running cpython to trap OS signals



early-python
************
``argument``: 无参数

``parser``: uwsgi_early_python

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load the python VM as soon as possible (useful for the fork server)



early-pyimport
**************
``argument``: 必需参数

``parser``: uwsgi_early_python_import

``flags``: UWSGI_OPT_IMMEDIATE

``help``: import a python module in the early phase



early-python-import
*******************
``argument``: 必需参数

``parser``: uwsgi_early_python_import

``flags``: UWSGI_OPT_IMMEDIATE

``help``: import a python module in the early phase



early-pythonpath
****************
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``flags``: UWSGI_OPT_IMMEDIATE

``help``: add directory (or glob) to pythonpath (immediate version)



early-python-path
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_pythonpath

``flags``: UWSGI_OPT_IMMEDIATE

``help``: add directory (or glob) to pythonpath (immediate version)




plugin: pyuwsgi
===============

plugin: rack
============
rails
*****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_POST_BUFFERING

``help``: load a rails <= 2.x app



rack
****
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``flags``: UWSGI_OPT_POST_BUFFERING

``help``: load a rack app



ruby-gc-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set ruby GC frequency



rb-gc-freq
**********
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set ruby GC frequency



rb-lib
******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a directory to the ruby libdir search path



ruby-lib
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a directory to the ruby libdir search path



rb-require
**********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script



ruby-require
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script



rbrequire
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script



rubyrequire
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script



require
*******
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script



shared-rb-require
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script (shared)



shared-ruby-require
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script (shared)



shared-rbrequire
****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script (shared)



shared-rubyrequire
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script (shared)



shared-require
**************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: import/require a ruby module/script (shared)



gemset
******
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified gemset (rvm)



rvm
***
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: load the specified gemset (rvm)



rvm-path
********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: search for rvm in the specified directory



rbshell
*******
``argument``: 可选参数

``parser``: uwsgi_opt_rbshell

``help``: run  a ruby/irb shell



rbshell-oneshot
***************
``argument``: 无参数

``parser``: uwsgi_opt_rbshell

``help``: set ruby/irb shell (one shot)




plugin: rados
=============
rados-mount
***********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``flags``: UWSGI_OPT_MIME

``help``: virtual mount the specified rados volume in a uri



rados-timeout
*************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: timeout for async operations




plugin: rawrouter
=================
rawrouter
*********
``argument``: 必需参数

``parser``: uwsgi_opt_undeferred_corerouter

``help``: run the rawrouter on the specified port



rawrouter-processes
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of rawrouter processes



rawrouter-workers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of rawrouter processes



rawrouter-zerg
**************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: attach the rawrouter to a zerg server



rawrouter-use-cache
*******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: use uWSGI cache as hostname->server mapper for the rawrouter



rawrouter-use-pattern
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: use a pattern for rawrouter hostname->server mapping



rawrouter-use-base
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: use a base dir for rawrouter hostname->server mapping



rawrouter-fallback
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: fallback to the specified node in case of error



rawrouter-use-code-string
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: use code string as hostname->server mapper for the rawrouter



rawrouter-use-socket
********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: forward request to the specified uwsgi socket



rawrouter-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward requests to the specified uwsgi server (you can specify it multiple times for load balancing)



rawrouter-gracetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: retry connections to dead static nodes after the specified amount of seconds



rawrouter-events
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of concurrent events



rawrouter-max-retries
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of retries/fallbacks to other nodes



rawrouter-quiet
***************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: do not report failed connections to instances



rawrouter-cheap
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run the rawrouter in cheap mode



rawrouter-subscription-server
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: run the rawrouter subscription server on the spcified address



rawrouter-subscription-slot
***************************
``argument``: 必需参数

``parser``: uwsgi_opt_deprecated

``help``: *** deprecated ***



rawrouter-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set rawrouter timeout



rawrouter-stats
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the rawrouter stats server



rawrouter-stats-server
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the rawrouter stats server



rawrouter-ss
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the rawrouter stats server



rawrouter-harakiri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable rawrouter harakiri



rawrouter-xclient
*****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use the xclient protocol to pass the client address



rawrouter-buffer-size
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set internal buffer size (default: page size)




plugin: rbthreads
=================
rbthreads
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable ruby native threads



rb-threads
**********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable ruby native threads



rbthread
********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable ruby native threads



rb-thread
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: enable ruby native threads




plugin: redislog
================

plugin: ring
============
ring-load
*********
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load the specified clojure script



clojure-load
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load the specified clojure script



ring-app
********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: map the specified ring application (syntax namespace:function)




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

``help``: store rrd files in the specified directory



rrdtool-freq
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set collect frequency



rrdtool-lib
***********
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the name of rrd library (default: librrd.so)




plugin: rsyslog
===============
rsyslog-packet-size
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set maximum packet size for syslog messages (default 1024) WARNING! using packets > 1024 breaks RFC 3164 (#4.1)



rsyslog-split-messages
**********************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: split big messages into multiple chunks if they are bigger than allowed packet size (default is false)




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

``help``: load config from sqlite3 db



sqlite
******
``argument``: 必需参数

``parser``: uwsgi_opt_load_sqlite3

``flags``: UWSGI_OPT_IMMEDIATE

``help``: load config from sqlite3 db




plugin: ssi
===========

plugin: sslrouter
=================
sslrouter
*********
``argument``: 必需参数

``parser``: uwsgi_opt_sslrouter

``help``: run the sslrouter on the specified port



sslrouter2
**********
``argument``: 必需参数

``parser``: uwsgi_opt_sslrouter2

``help``: run the sslrouter on the specified port (key-value based)



sslrouter-session-context
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: set the session id context to the specified value



sslrouter-processes
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of sslrouter processes



sslrouter-workers
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: prefork the specified number of sslrouter processes



sslrouter-zerg
**************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_zerg

``help``: attach the sslrouter to a zerg server



sslrouter-use-cache
*******************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: use uWSGI cache as hostname->server mapper for the sslrouter



sslrouter-use-pattern
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_pattern

``help``: use a pattern for sslrouter hostname->server mapping



sslrouter-use-base
******************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_use_base

``help``: use a base dir for sslrouter hostname->server mapping



sslrouter-fallback
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: fallback to the specified node in case of error



sslrouter-use-code-string
*************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_cs

``help``: use code string as hostname->server mapper for the sslrouter



sslrouter-use-socket
********************
``argument``: 可选参数

``parser``: uwsgi_opt_corerouter_use_socket

``help``: forward request to the specified uwsgi socket



sslrouter-to
************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: forward requests to the specified uwsgi server (you can specify it multiple times for load balancing)



sslrouter-gracetime
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: retry connections to dead static nodes after the specified amount of seconds



sslrouter-events
****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of concurrent events



sslrouter-max-retries
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set the maximum number of retries/fallbacks to other nodes



sslrouter-quiet
***************
``argument``: 必需参数

``parser``: uwsgi_opt_true

``help``: do not report failed connections to instances



sslrouter-cheap
***************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: run the sslrouter in cheap mode



sslrouter-subscription-server
*****************************
``argument``: 必需参数

``parser``: uwsgi_opt_corerouter_ss

``help``: run the sslrouter subscription server on the spcified address



sslrouter-timeout
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: set sslrouter timeout



sslrouter-stats
***************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the sslrouter stats server



sslrouter-stats-server
**********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the sslrouter stats server



sslrouter-ss
************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the sslrouter stats server



sslrouter-harakiri
******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_int

``help``: enable sslrouter harakiri



sslrouter-sni
*************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use SNI to route requests



sslrouter-buffer-size
*********************
``argument``: 必需参数

``parser``: uwsgi_opt_set_64bit

``help``: set internal buffer size (default: page size)




plugin: stackless
=================
stackless
*********
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use stackless as suspend engine




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

``help``: load the specified C symbol as the symcall request handler (supports <mountpoint=func> too)



symcall-use-next
****************
``argument``: 无参数

``parser``: uwsgi_opt_true

``help``: use RTLD_NEXT when searching for symbols



symcall-register-rpc
********************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: load the specified C symbol as an RPC function (syntax: name function)



symcall-post-fork
*****************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: call the specified C symbol after each fork()




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

``help``: a shortcut enabling tornado loop engine with the specified number of async cores and optimal parameters




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

``help``: run the tuntap router (syntax: <device> <socket> [stats] [gateway])



tuntap-device
*************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a tuntap device to the instance (syntax: <device>[ <socket>])



tuntap-use-credentials
**********************
``argument``: 可选参数

``parser``: uwsgi_opt_set_str

``help``: enable check of SCM_CREDENTIALS for tuntap client/server



tuntap-router-firewall-in
*************************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_firewall

``help``: add a firewall rule to the tuntap router (syntax: <action> <src/mask> <dst/mask>)



tuntap-router-firewall-out
**************************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_firewall

``help``: add a firewall rule to the tuntap router (syntax: <action> <src/mask> <dst/mask>)



tuntap-router-route
*******************
``argument``: 必需参数

``parser``: uwsgi_tuntap_opt_route

``help``: add a routing rule to the tuntap router (syntax: <src/mask> <dst/mask> <gateway>)



tuntap-router-stats
*******************
``argument``: 必需参数

``parser``: uwsgi_opt_set_str

``help``: run the tuntap router stats server



tuntap-device-rule
******************
``argument``: 必需参数

``parser``: uwsgi_opt_add_string_list

``help``: add a tuntap device rule (syntax: <direction> <src/mask> <dst/mask> <action> [target])




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

``help``: set ugreen stack size in pages




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

``help``: put v8 in preemptive move (单个隔离) with the specified frequency



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



