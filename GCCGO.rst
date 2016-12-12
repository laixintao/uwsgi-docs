GCCGO插件
================

uWSGI 1.9.20官方使用一个基于GCCGO的新插件替换了老的 :doc:`Go` 插件。

GCCGO的使用允许更多的特性，并且与uWSGI部署方式更好地集成起来。

GCC 套件 >= 4.8 is expected (强烈建议)。

它是如何工作的
************

当启用插件的时候，会在每次 ``fork()`` 之后初始化一个新的go运行时(runtime)。

如果一个 ``main`` Go函数在进程地址空间内可用，那么它将会在Go运行时中执行，否则，控制权会回到uWSGI循环引擎。

为什么不使用纯Go？
*********************

不幸的是，标准的Go运行时当前是不可嵌入的，并且不支持作为共享库编译代码。

而这两个都是有意义的uWSGI集成所需的。

从GCC 4.8.2起，大大改进了它的 ``libgo`` ，并且构建共享库以及初始化Go运行时工作得异常成功 (即使它需要一点……不非常优雅的hack)。

构建插件
*******************

有一个构建配置文件可用，允许你构建一个uWSGI+gccgo二进制文件，准备好加载Go共享库：

.. code-block:: sh

   make gccgo

第一个应用
*************

你不需要改变你在Go中编写web应用的方式。可以无缝使用 ``net/http`` 包：

.. code-block:: go

   package main

   import "uwsgi"
   import "net/http"
   import "fmt"



   func viewHandler(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "<h1>Hello World</h1>")
   }

   func main() {
        http.HandleFunc("/view/", viewHandler)
        uwsgi.Run()
   }

唯一不同之处在于调用 ``uwsgi.Run()`` ，而不是初始化Go HTTP服务器。

要把代码作为共享库进行构建，只要简单运行：

.. code-block:: sh

   gcc -fPIC -shared -o myapp.so myapp.go
   
如果你得到一个关于gcc不能够解析uWSGI符号的错误，那么只需添加 ``-I<path_to_uwsgi_binary>`` 到命令行 (见下):

.. code-block:: sh

   gcc -fPIC -shared -I/usr/bin -o myapp.so myapp.go
   
现在，在uWSGI之下运行它：

.. code-block:: sh

   uwsgi --http-socket :9090 --http-socket-modifier1 11 --go-load ./myapp.so
   
gccgo插件将其自身注册为 ``modifier1`` 11，所以，记得设置它来运行Go代码。

uwsgi.gox
*********

默认情况下，当构建gccgo配置文件的时候，会创建一个uwsgi.gox文件。在使用uWSGI API构建go应用的时候，可以使用它，来解析符号。

记住，如果你添加包含uwsgi二进制（如前所示）的目录到gcc的包含 (``-I path``) 路径，那么二进制文件本身将会被用于解析符号。

共享库 VS 单片二进制文件
***************************************

对于许多开发者来说，Go的主要卖点在于“静态多合一”的二进制方法。

基本上，一个Go应用并没有依赖，因此，一半的常见部署文件就这样自动消失了。

托管Go应用的uWSGI友好方式是让一个uWSGI二进制文件以库的形式加载一个指定的Go应用。

如果这不可接受，那么你可以同时用uWSGI和该Go应用构建单个二进制文件：

.. code-block:: sh

   CFLAGS=-DUWSGI_GCCGO_MONOLITHIC UWSGI_ADDITIONAL_SOURCES=myapp.go UWSGI_PROFILE=gccgo make


协程
**********

多亏了新的GCC split stack特性，在gccgo中，goroutine被健全 (例如，不需要一个完整的pthread) 实现。

在插件自身中，有一个将每个uWSGI核心映射到一个goroutine上的循环引擎可用。

要以goroutine模式启动uWSGI，仅需添加 ``--goroutines <n>`` ，其中，<n>是要生成的并发goroutine的最大数。

就像 :doc:`Gevent` ，uWSGI信号处理器是在一个专用的goroutine中执行的。

除此之外，所有的阻塞调用利用 ``netpoll`` Go api。这意味着，你可以在一个goroutine中运行内部路由动作，包括rpc。

选项
*******

* ``--go-load <path>`` 在进程地址空间中加载指定的go共享库
* ``--gccgo-load <path>`` go-load的别名
* ``--go-args <arg1> <arg2> <argN>`` 设置传递给虚拟go命令行的参数
* ``--gccgo-args <arg1> <arg2> <argN>`` go-args的别名
* ``--goroutines <n>`` 启用goroutine循环引擎，并指定异步核数目

uWSGI API
*********

.. note:: 这部分可能，或者可能不过时。谁知道呢！

不幸的是，只有一点点uWSGI API被移植到了gccgo插件中。对于uWSGI 2.0，会及时添加更多的特性。

当前公开的API函数：

* ``uwsgi.CacheGet(key string, cache string) string``
* ``uwsgi.RegisterSignal(signum uint8, receiver string, handler func(uint8)) bool``

注意事项
*********

* 拜托拜托，不要启用多线程，它将不能用，并且可能永远不能用。
* 所有的uWSGI原生特性 (像内部路由) 是工作在goroutine模式下的。然而，别指望近期诸如Python或者Perl这样的语言会在其上工作。
