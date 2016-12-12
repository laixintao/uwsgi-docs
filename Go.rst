uWSGI Go支持 (只有1.4)
===========================

.. warning:: 自1.9.20起，Go插件已经被 :doc:`GCCGO` 插件取代了。

从uWSGI 1.4-dev起，你可以在你的uWSGI栈中托管Go web应用了。你可以从http://golang.org/下载Go。目前，支持Linux
i386/x86_64, FreeBSD i386/x86_64和OSX。对于OSX支持，你需要Go 1.0.3+，或者你将需要应用http://code.google.com/p/go/source/detail?r=62b7ebe62958上的可用补丁。goroutine目前只在Linux i386/x86_64上支持。

构建带Go支持的uWSGI
******************************

可以作为嵌入部件或者插件构建Go支持。与其他语言的设置主要的不同是，这次，我们会构建一个uwsgi库，而不是一个uwsgi二进制文件。这个库将会被名为uwsgi.go的一个Go包使用，你可以链接你的应用。不要怕，因为在uWSGI发行版中，已经有一个构建配置文件，你可以用它来构建一个内置Go支持的完整的（单片）发行版。在构建过程的最后，你会得到一个libuwsgi.so共享库和一个uwsgi.a Go包。

要构建uWSGI+go，仅需运行 (在uWSGI源代码目录)

.. code-block:: sh

   UWSGI_PROFILE=go make

或者如果Python并不在你的系统目录中，或者你需要使用一个指定python版本：

.. code-block:: sh

   /usr/local/bin/python uwsgiconfig.py --build go

(或者你自定义的Python的位置)

在构建过程的最后，你会得到一个libuwsgi.so文件 (拷贝或者链接它到一个库目录，例如/usr/local/lib或者/usr/lib，最后需要的话，运行ldconfig)，和一个plugins/go/pkg子目录（基于你的架构/操作系统）中的uwsgi.a文件。

.. important::

   构建过程的最后的消息报告了你在构建uWSGI Go应用的时候应该使用的 ``GOPATH``  (在某个地方拷贝/记住/注释该值)。

   如果你已经知道Go导入系统是如何工作的，那么随意拷贝你的系统范围的GOPATH中的uwsgi.a。

编写第一个Go应用
********************************

默认情况下，uWSGI Go插件支持 ``http.DefaultServeMux`` 处理器，所以，如果你的应用已经基于此了，那么在uWSGI中运行它应该非常简单。

.. code-block:: go

   package main

   import (
           "uwsgi"
           "net/http"
           "fmt"
   )

   func oneHandler(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintf(w, "<h1>One</h1>")
   }


   func twoHandler(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintf(w, "<h2>Two</h2>")
   }

   func main() {
           http.HandleFunc("/one/", oneHandler)
           http.HandleFunc("/two/", twoHandler)
           uwsgi.Run()
   }

可以看到，唯一与标准的基于 ``net/http`` 的应用不同的地方在于，需要 ``import "uwsgi"`` ，以及调用 ``uwsgi.Run()`` 函数，这将会运行整个uWSGI服务器。如果你想用你自己的请求处理器来取代 ``http.DefaultServeMux`` ，那么使用``uwsgi.Handler(http.Handler)`` 或者 ``uwsgi.RequestHandler(func(http.ResponseWriter, *http.Request))`` 来设置它。

.. code-block:: go

   func myHandler(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintf(w, "<h2>Two</h2>")
   }

   func main() {
           uwsgi.RequestHandler(myHandler)
           uwsgi.Run()
   }

构建你的第一个应用
***********************

假设你的应用是helloworld.go，仅需运行以下。

.. code-block:: sh

   GOPATH=/home/foobar/uwsgi/plugins/go go build helloworld.go

将GOPATH改为你从构建过程获得的值，或者你安装/拷贝uwsgi.a的目录。如果一切顺利，你会得到一个'helloworld'可执行文件。那个可执行文件是一个完整的uWSGI服务器 (是的，真的是)。

.. code-block:: sh

   ./helloworld --http :8080 --http-modifier1 11

仅需将你的浏览器指向端口8080，然后检查/one/和/two/。你可以开始添加进程和一个master：

.. code-block:: sh

   ./helloworld --http :8080 --http-modifier1 11 --master --processes 8

注意：官方分配了modifier1 11给Go。

上生产
*******************

在一个生产环境中，你可能会把一个web服务器/代理放在你的应用前面。因此，你的nginx配置看起来会是这样：：

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 11;
   }

而你的uWSGI配置将会差不多是这样……

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 4

最后，简单运行你的应用：

.. code-block:: sh

   ./helloworld config.ini

goroutine (目前仅Linux/FreeBSD)
*****************************************

goroutine非常可能是Go平台最有趣的特性。当你构建带go插件的uWSGI时，一个用于goroutine的uWSGI循环引擎自动嵌入到uWSGI库中。要在每个uWSGI进程中生成goroutine，只需添加 ``goroutines = N`` 选项，其中，N是要生成的goroutine数目。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 4
   goroutines = 100

使用这个配置，你将为每个uWSGI进程生成100个goroutine，总共生成400个goroutine (!) 就uWSGI看来，goroutine对应pthread，但你将也能够从你的应用生成基于coroutine的任务。

uWSGI api
*********

从你的Go应用访问uWSGI API是相当简单的。要这样做，请引用uwsgi包导出的函数：

.. code-block:: go

   package main

   import (
      "fmt"
      "uwsgi"
   )

   func hello2(signum int) {
           fmt.Println("I am an rb_timer running on mule", uwsgi.MuleId())
   }

   func hello(signum int) {
           fmt.Println("Ciao, 3 seconds elapsed")
   }

   func postinit() {
           uwsgi.RegisterSignal(17, "", hello)
           uwsgi.AddTimer(17, 3)

           uwsgi.RegisterSignal(30, "mule1", hello2)
           uwsgi.AddRbTimer(30, 5)
   }

   func foofork() {
      fmt.Println("fork() has been called")
   }

   func main() {
           uwsgi.PostInit(postinit)
           uwsgi.PostFork(foofork)
           uwsgi.Run()
   }


PostInit()函数设置在完成Go初始化之后，调用的“钩子”。PostFork()设置在每次fork()之后调用的“钩子”。在postinit钩子中，我们注册了两个uwsgi信号，第二个运行在一个mule (mule1) 中。要运行这个代码，只需像上面那样构建你新的应用，然后执行它

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 2
   goroutines = 20
   mules = 2
   memory-report = true

这次，我们添加了memory-report，试一试，看看Go应用可以多省内存。

从Emperor运行
************************

如果你运行在Emperor模式下，那么你可以通过 ``privileged-binary-patch`` 选项运行uWSGI-Go应用。你的vassal配置应该差不多像这样。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   master = true
   processes = 2
   goroutines = 20
   mules = 2
   memory-report = true
   uid = foobar
   gid = foobar
   privileged-binary-patch = /tmp/bin/helloworld

(显然，修改``/tmp/bin/helloworld`` 为任何你应用所在的地方……)

注意事项
********

* 你可以在uWSGI源代码发行版的 ``t/go`` 目录中找到一系列有趣的go样例。
* 目前不可能在不修改go核心的情况下修改进程名
* 你不能和Go一起使用uWSGI原生线程 (只使用--goroutines)
* 目前只公开了一小部分uWSGI API。如果你想要hack下，或者需要更多，那么只需编辑plugins/go/src/uwsgi目录中的uwsgi.go文件
* goroutine需要异步模式 (如果你正自定义你的uWSGI二进制文件，那么记得总是包含它)
* 似乎可能甚至在goroutines模式下无问题加载Python, Lua和PSGI插件 (需要更多的测试)
