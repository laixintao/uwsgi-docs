Clojure/Ring JVM请求处理器
====================================

幸好自1.9起， :doc:`JVM` 插件就能用了，可以在uWSGI上运行Clojure web应用。

支持的网关标准是Ring, https://github.com/ring-clojure/ring。它完整的详细说明在这里： https://github.com/ring-clojure/ring/blob/master/SPEC

有一个名为"ring"的uWSGI构建配置文件可以用来生成同时带有JVM和Ring插件的单片构建。

从uWSGI源代码：

.. code-block:: sh

   UWSGI_PROFILE=ring make

构建系统将会试着根据不同预设检测你的JDK安装 (例如，在CentOS上，你可以 ``yum install 
java-1.6.0-openjdk.x86_64-devel`` 或者 ``java-1.7.0-openjdk-devel.x86_64`` ，或者在Debian/Ubuntu上 ``openjdk-6-jdk`` 等等……)。

也会搜索OSX/Xcode默认路径。

成功构建之后，你将会得到uwsgi二进制文件和一个uwsgi.jar文件，你应该拷贝到你的CLASSPATH中 (或者每次记住在uwsgi配置中设置它)。

.. seealso:: 对于更多关于JVM插件的信息，看看 :doc:`JVM`

我们第一个Ring应用
******************

一个基本的Clojure/Ring应用可能如下 (将其保持为myapp.clj)：

.. code-block:: Clojure

   (ns myapp)

   (defn handler [req]
        {:status 200
         :headers { "Content-Type" "text/plain" , "Server" "uWSGI" }
         :body (str "<h1>The requested uri is " (get req :uri) "</h1>")
        }
   )

该代码定义了一个名为'myapp'的新的名字空间，其中，'handler'函数是Ring的入口点 (每个web请求都会调用的函数)

现在，我们可以构建一个配置，在HTTP路由器上提供该应用，端口为9090 (称之为config.ini)：

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 8
   http-modifier2 = 1

   jvm-classpath = plugins/jvm/uwsgi.jar
   jvm-classpath = ../.lein/self-installs/leiningen-2.0.0-standalone.jar

   clojure-load = myapp.clj
   ring-app = myapp:handler

运行uWSGI:

.. code-block:: sh

   ./uwsgi config.ini

现在，连接到端口9090，你应该看到该应用响应。

你可以注意到，我们手工添加了uwsgi.jar和Leiningen的standalone.jar (它包括整个Clojure发行版) 到我们的classpath中。

显然，如果你不想要使用Leiningen，那么只需添加Clojure jar到你的classpath。

``clojure-load`` 选项在JVM中加载了一个Clojure对象 (非常类似于 ``jvm-class`` 和基本的jvm插件做的事)。

 ``ring-app`` 指定了搜索ring函数入口点的类/名字空间。

在我们的例子中，该函数位于'myapp'名字空间，叫做'handler' (你可以理解语法为namespace:function)

注意modifier配置。JVM插件将自己注册为8，而Ring则将自己注册为modifier 2 #1，产生有效配置"modifier1 8, modifier2 1"。

使用Leiningen
***************

Leiningen是一个用来管理Clojure项目的很不错的工具。如果你使用Clojure，那么你非常有可能是一个Leiningen用户。

Leiningen的一个巨大的优势在于轻松生成单个JAR发行版。这意味着，你可以用单个文件部署整个应用。

让我们用 ``lein`` 命令创建一个新的"helloworld" Ring应用。

.. code-block:: sh

   lein new helloworld

把它移到刚刚创建的'helloworld'目录，然后编辑project.clj文件

.. code-block:: Clojure

   (defproject helloworld "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.Clojure/Clojure "1.4.0"]])

我们想要添加 ``ring-core`` 包到我们的依赖中 (它包含了一组类/模块来简化ring应用的编写) ，而显然，我们需要修改描述和URL：

.. code-block:: Clojure

   (defproject helloworld "0.1.0-SNAPSHOT"
  :description "My second uWSGI ring app"
  :url "https://uwsgi-docs.readthedocs.io/en/latest/Ring.html"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.Clojure/Clojure "1.4.0"] [ring/ring-core "1.2.0-beta1"]])

现在，保存并运行……

.. code-block:: sh

   lein repl

这将会安装所有我们需要的jar，并且带我们到Clojure控制台 (目前从中退出即可)。

现在，我们想要写我们的Ring应用，只需编辑文件src/helloworld/core.clj，将以下内容放到它里面：

.. code-block:: Clojure

   (ns helloworld.core
    (:use ring.util.response))

   (defn handler [request]
    (-> (response "Hello World")
    (content-type "text/plain")))


然后重新编辑project.clj，来指示Leiningen在哪个名字空间之上构建：

.. code-block:: Clojure

   (defproject helloworld "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}

  :aot [helloworld.core]

  :dependencies [[org.Clojure/Clojure "1.4.0"] [ring/ring-core "1.2.0-beta1"]])

正如你所见，我们在 ``:aot`` 关键字中添加了helloworld.core。

现在，让我们编译我们的代码：

.. code-block:: sh

   lein compile

然后构建完整的jar (uberjar):

.. code-block:: sh

   lein uberjar

如果一切顺利，你应该会在这个过程的最后看到像这样的消息：

.. code-block:: sh

   Created /home/unbit/helloworld/target/helloworld-0.1.0-SNAPSHOT-standalone.jar

记下这个地址，接着我们可以配置uWSGI来运行我们的应用。

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 8
   http-modifier2 = 1

   jvm-classpath = plugins/jvm/uwsgi.jar
   jvm-classpath = /home/unbit/helloworld/target/helloworld-0.1.0-SNAPSHOT-standalone.jar

   jvm-class = helloworld/core__init

   ring-app = helloworld.core:handler

这次，我们不加载Clojure代码，而是直接加载一个JVM类。

注意：当你指定过一个JVM类的时候，你必须使用'/'形式，而不是常见的点形式。

当编译你的应用的时候，Clojure系统会自动添加__init后缀。

``ring-app`` 设置helloworld.core名字空间的入口点和函数'handler'。

当我们用 ``jvm-class`` 加载它的时候，我们可以访问那个名字空间。

并发
***********

和所有JVM插件请求处理器一样，多线程是达到并发的最好的方式。

JVM中的线程时相当稳定的，不要怕使用它们 (即使你也可以生成多个进程)

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 8
   http-modifier2 = 1

   jvm-classpath = plugins/jvm/uwsgi.jar
   jvm-classpath = /home/unbit/helloworld/target/helloworld-0.1.0-SNAPSHOT-standalone.jar

   jvm-class = helloworld/core__init

   ring-app = helloworld.core:handler

   master = true
   processes = 4
   threads = 8

这个设置会生成4个uWSGI进程 (worker) ，每个进程有8个线程 (总共32个线程)。

访问uWSGI api
***********************

Clojure也可以调用原生的Java类，因此它可以访问由JVM插件公开的uWSGI API。

下面的例子展示了如何通过Clojure调用一个函数 (用python写的)：

.. code-block:: Clojure

   (ns myapp
    (import uwsgi)
   )

   (defn handler [req]
     {:status 200
      :headers { "Content-Type" "text/html" , "Server" "uWSGI" }
      :body (str "<h1>The requested uri is " (get req :uri) "</h1>" "<h2>reverse is " (uwsgi/rpc (into-array ["" "reverse" (get req :uri)])) "</h2>" )
     }
   )

从一个Python模块注册了"reverse"函数：

.. code-block:: python
 
   from uwsgidecorators import *

   @rpc('reverse')
   def contrario(arg):
       return arg[::-1]

这是使用的配置：

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 8
   http-modifier2 = 1 
   jvm-classpath = plugins/jvm/uwsgi.jar
   jvm-classpath = /usr/share/java/Clojure-1.4.jar
   Clojure-load = myapp.clj
   plugin = python
   import = pyrpc.py
   ring-app = myapp:handler
   master = true

另一个有用的特性是访问uwsgi换成。记住，缓存键是字符串，而值是字节。

uWSGI实现对于响应，除了字符串，还支持字节数组。这显然违反了标准，但是避免了每次都要重新编码字节 (但显然，如果你喜欢的话，你可以随意每次重新编码)。

注意事项和状态
****************

* 有一个允许加载已编译代码和指定ring应用的快捷选项将会很酷。
* 正如 :doc:`JWSGI` 处理器，自动使用所有的uWSGI性能特性 (就像当发送静态文件或者缓存输入的时候)
* 这个插件是在Mingli Yuan的合作和贡献想法之下开发出来的。灰常感谢！
