uWSGI服务器中的JVM (更新至1.9)
========================================

.. toctree::
   :maxdepth: 1

   JWSGI
   Ring

介绍
************  

自uWSGI 1.9起，你可以拥有一个嵌入在核心中的完整、线程安全的通用JVM。所有的插件可以通过 :doc:`RPC subsystem<RPC>` 或者使用uWSGI :doc:`Signals` 来调用JVM函数 (用Java, JRuby, Jython, Clojure, 任何JVM可以运行的新式语言编写的)。JVM插件本身可以实现请求处理器来托管基于JVM的web应用。目前，支持 :doc:`JWSGI` 和 :doc:`Ring` (Clojure)应用。长期目标是支持servlets，但它将需要大量的赞助和资金 (请随意到info@unbit.it询问更多有关项目的信息)。

构建JVM支持
************************

首先，确保安装了一个完整的JDK发行版。uWSGI构建系统将试着检测常见的JDK设置 (Debian, Ubuntu, Centos, OSX...)，但是，如果它没法找到一个JDK安装，那么它会需要一些来自用户的信息（见下）。要构建JVM插件，只需运行：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/jvm default

如果需要的话，修改'default'为你的构建配置文件。例如，如果你有一个Perl/PSGI单片构建，只需运行

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/jvm psgi

或者对于一个完整的模块化构建

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/jvm core

如果一切顺利，将会构建jvm_plugin。如果构建系统不能找到一个JDK案子，那么你将需要指定头文件目录（包含jni.h文件的目录）和lib目录（包含libjvm.so的目录）的地址。例如，如果jni.h在/opt/java/includes中，而libjvm.so在/opt/java/lib/jvm/i386中，那么这样运行构建系统：

.. code-block:: sh

   UWSGICONFIG_JVM_INCPATH=/opt/java/includes UWSGICONFIG_JVM_LIBPATH=/opt/java/lib/jvm/i386 python uwsgiconfig --plugin plugins/jvm


成功构建之后，你会获得uwsgi.jar文件的路径。那个jarball包含了访问uWSGI API的类，而你应该将它拷贝到你的CLASSPATH中，或者至少手工从uWSGI的配置加载它。

通过RPC子系统公开函数
****************************************

在这个例子中，我们将公开一个"hello" Java函数 (返回一个字符串)，并且将从一个Python WSGI应用中调用它。这是我们的基本配置 (假设使用模块化构建)。

.. code-block:: ini

   [uwsgi]
   plugins = python,jvm
   http = :9090
   wsgi-file = myapp.py
   jvm-classpath = /opt/uwsgi/lib/uwsgi.jar

``jvm-classpath`` 是一个由JVM插件公开的选项，它允许你添加目录或者jar文件到你的classpath中。你可以指定任意多的 ``jvm-classpath`` 选项。这里，我们手工添加 ``uwsgi.jar`` ，因为我们并没有将它拷贝到我们的CLASSPATH中。这是我们的WSGI样例脚本。

.. code-block:: py

   import uwsgi
   
   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       yield "<h1>"
       yield uwsgi.call('hello')
       yield "</h1>"

这里，我们使用 ``uwsgi.call()`` 来取代 ``uwsgi.rpc()`` ，作为一种快捷方式 (选项解析时会有小的性能提升)。现在，我们创建我们的Foobar.java类。它的 ``static void main()`` 函数将会在启动的时候由uWSGI运行。

.. code-block:: java

   public class Foobar {
      static void main() {

          // create an anonymous function
          uwsgi.RpcFunction rpc_func = new uwsgi.RpcFunction() { 
              public String function(String... args) {
                  return "Hello World";
              }
          };

          // register it in the uWSGI RPC subsystem
          uwsgi.register_rpc("hello", rpc_func);
      }
   }


``uwsgi.RpcFunction`` 接口允许你轻松编写uWSGI兼容的RPC函数。现在，编译Foobar.java文件：

.. code-block:: sh

   javac Foobar.java

(最终，修正classpath，或者用-cp选项传递uwsgi.jar路径)
你现在有了一个Foobar.class，它可以由uWSGI加载。让我们完成配置……

.. code-block:: ini

   [uwsgi]
   plugins = python,jvm
   http = :9090
   wsgi-file = myapp.py
   jvm-classpath = /opt/uwsgi/lib/uwsgi.jar
   jvm-main-class = Foobar

最后一个选项 (``jvm-main-class``) 将会加载一个java类，并且执行它的 ``main()`` 方法。现在，我们可以访问localhost:9090，并且应该看到Hello World消息。

注册信号处理器
***************************

用与RPC子系统相同的方式，你可以注册信号处理器。你将能够在时间事件、文件修改、cron等上调用Java函数

我们的Sigbar.java:

.. code-block:: java

   public class Sigbar {
      static void main() {

          // create an anonymous function
          uwsgi.SignalHandler sh = new uwsgi.SignalHandler() { 
              public void function(int signum) {
                  System.out.println("Hi, i am the signal " + signum);
              }
          };

          // register it in the uWSGI signal subsystem
          uwsgi.register_signal(17, "", sh);
      }
   }

``uwsgi.SignalHandler`` 是用于信号处理器的接口。

每当触发了信号17，将会运行对应的JVM。记得编译这个文件，在uWSGI中加载它，并启用master进程 (没有它，信号子系统将不能用)。


fork()问题和多线程
*************************************

JVM并不是 ``fork()`` 友好的。如果你在master中加载一个虚拟机，然后fork() (就像你通常在其他语言中做的那样) 子JVM将会失败 (这主要是因为JVM需要的线程并不会继承)。出于这样的原因，会为每个worker、mule和spooler生成一个JVM。幸运的是，与其他绝大部分平台不同，JVM带有真正强大的多线程支持。uWSGI支持它，因此，如果你想要运行请求处理器 (JWSGI, Clojure/Ring) 中的一个，那么只需记得使用 ``--threads`` 选项生成一批线程。

它是如何工作的呢？
*****************

uWSGI使用JNI接口嵌入JVM。不幸的是，我们不能依赖JVM的自动垃圾回收器，因此我们必须自动解引用所有分配的对象。这从性能和使用的角度来说并不是个问题，但是会让插件的开发与其他基于JVM的产品相比更难一点。幸运的是，当前的API简化了那个任务。

传递选项到JVM
**************************

你可以使用 ``--jvm-opt`` 选项传递指定的选项给JVM。

例如，要把堆使用限制为10兆字节：

.. code-block:: ini

   [uwsgi]
   ...
   jvm-opt = -Xmx10m

加载类 (不带main方法)
*************************************

我们已经看到了如何加载类和在启动时运行它们的 ``main()`` 方法。通常来说，你会想要只在添加它们到JVM的时候加载类 (运行访问外部模块需要用到它们)。要加载一个类，你可以使用 ``--jvm-class`` 。

.. code-block:: ini

   [uwsgi]
   ...
   jvm-class = Foobar
   jvm-class = org/unbit/Unbit

记住，类名必须使用'/'格式，而不是点！这个规则也应用到 ``--jvm-main-class`` 。

请求处理器
****************

虽然Java(TM)的世界有它自己的J2EE环境来部署web应用，你可以响应使用一个不同的方法。uWSGI项目实现了J2EE没有的大量特性 (并且没有实现大量J2EE强大的特性)，因此，你可能会发现它的方法更适合你的设置（或者口味，又或者技能）。

JVM插件公开了一个API，允许hook web请求。这个方法与uWSGI工作的“传统”方式有点托尼盖。JVM插件将自己注册为一个处理器，使用modifier1==8，但将会看看modifier2的值，以确定使用它的哪一个请求处理器来处理它。例如， :doc:`Ring` 插件在JVM插件中注册，使用modifier2数为'1'。因此，要传递请求给它，你需要类似于这样的：

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 8
   http-modifier2 = 1

或者使用nginx:

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_modifier1 8;
       uwsgi_modifier2 1;
       uwsgi_pass /tmp/uwsgi.socket;
   }


目前，有2个JVM请求处理器可用：

* :doc:`JWSGI`
* :doc:`Ring` (for Clojure)

前面已经说了，开发一个servlet请求处理器的想法是有的，但是它将需要赞助（也就是说，银子），因为这会需要相当大的努力。

注意事项
**********

* 要使用UNIX socket，无需特殊的jar文件 —— JVM插件可以访问所有的uWSGI特性。
* 你可能会沉迷于log4j模块。这没啥不对，但是看看uWSGI的日志记录功能 (需要更少的资源，更少的配置，以及更少的进取心)
* uWSGI API访问仍然未完成 (将会在1.9后更新)
* JVM在受限地址空间的环境中无法愉快使用。如果你在你的实例中加载JVM，那么避免使用 ``--limit-as`` 。
