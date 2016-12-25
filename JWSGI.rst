JWSGI接口
===================

.. note:: JWSGI并不是一个标准，然而，如果你喜欢JWSGI，那么何不发送一个RFC给uWSGI邮件列表。我们对标准没有特别的兴趣，但是谁知道呢……

JWSGI是考虑Java的WSGI/PSGI/Rack方式的一个口子。

如果，由于某种模糊的原因，你想要用JVM语言开发应用，并且你不想要部署一个巨大的servlet栈，那么JWSGI应该与你同行。

它是一个非常简单的协议：调用一个公共方法，它接收一个 ``HashMap`` 作为它唯一的参数。这个HashMap包含了CGI风格的变量，而 ``jwsgi.input`` 包含了一个Java InputStream对象。

这个函数必须返回一个包含3个对象的数字：

* ``status`` (java.lang.Integer) (例如：200)
* ``headers`` (HashMap) (例如：{"Content-type": "text/html", "Server":
  "uWSGI", "Foo": ["one","two"]})
* ``body`` (可能是一个String，一个String数组，一个File或者一个InputStream对象)

例子
-------

一个简单的JWSGI应用看起来像这样：

.. code-block:: java

   import java.util.*;
   public class MyApp {

       public static Object[] application(HashMap env) {

           int status = 200;

           HashMap<String,Object> headers = new HashMap<String,Object>();
           headers.put("Content-type", "text/html");
           // a response header can have multiple values
           String[] servers = {"uWSGI", "Unbit"};
           headers.put("Server", servers);

           String body = "<h1>Hello World</h1>" + env.get("REQUEST_URI");

           Object[] response = { status, headers, body };

           return response;
       }
   }



如何使用它？
***************

你同时需要'jvm'插件和'jwsgi'插件。项目中有一个名为'jwsgi'的构建配置文件可以用，允许你实现带jvm+jwsgi的单片构建：

.. code-block:: sh

   UWSGI_PROFILE=jwsgi make


1. 用 ``javac`` 编译你的类。

   .. code-block:: sh

      javac MyApp.java

4. 运行uWSGI，并且指定要运行的方法 (使用格式class:method)

   .. code-block:: sh

      ./uwsgi --socket /tmp/uwsgi.socket --plugins jvm,jwsgi --jwsgi
      MyApp:application --threads 40

  这将会在UNIX socket /tmp/uwsgi.socket上运行一个JWSGI应用，使用40个线程。

读取请求体
********************

``jwsgi.input`` 项是一个 ``uwsgi.RequestBody`` 对象 (java/io/InputStream的子类). 通过它来访问请求体。

.. code-block:: java

   import java.util.*;
   public class MyApp {

       public static Object[] application(HashMap env) {

           int status = 200;

           HashMap<String,Object> headers = new HashMap<String,Object>();
           headers.put("Content-type", "text/plain");

           int body_len = Integer.parseInt((String) env.get("CONTENT_LENGTH"));
           byte[] chunk = new byte[body_len];

           uwsgi.RequestBody input = (uwsgi.RequestBody) env.get("jwsgi.input");

           int len = input.read(chunk);

           System.out.println("read " + len + " bytes");

           String body = new String(chunk, 0, len);

           Object[] response = { status, headers, body };

           return response;
       }
   }

注意用 ``read(byte[])`` 来代替传统的 ``read()`` 。后者低效地每次读取一个字节，而前者一次读取一个更大的块。

JWSGI和Groovy
****************

因为它是低级别的，因此，JWSGI标准可以原样用于JVM上运行的其他语言。例如，这是一个"Hello World" Groovy样例：

.. code-block:: groovy

   static def Object[] application(java.util.HashMap env) {
        def headers = ["Content-Type":"text/html", "Server":"uWSGI"]
        return [200, headers, "<h1>Hello World</h1"]
   }

一个提供静态文件服务的例子：

.. code-block:: groovy

   static def Object[] application(java.util.HashMap env) {
        def headers = ["Content-Type":"text/plain", "Server":"uWSGI"]
        return [200, headers, new File("/etc/services")]
   }

第二个方法非常有效，因为它将滥用uWSGI内部功能。例如，如果你启用了卸载，那么你的worker线程将会被突然释放。要加载Groovy代码，记得编译它：

.. code-block:: sh

   groovyc Foobar.groovy

然后运行它：

.. code-block:: sh

   ./uwsgi --socket /tmp/uwsgi.socket --plugins jvm,jwsgi --jwsgi Foobar:application --threads 40

JWSGI和Scala
***************

就像Groovy，你可以用Scala编写JWSGI应用。你只需要入口点函数来使用原生的Java对象：

.. code-block:: scala

   object HelloWorld {
        def application(env:java.util.HashMap[String, Object]): Array[Object] = {
                var headers = new java.util.HashMap[String, Object]()
                headers.put("Content-Type", "text/html")
                headers.put("Server", "uWSGI")
                return Array(200:java.lang.Integer, headers , "Hello World")
        }
   }

或者一种更Scala的方式：

.. code-block:: scala

   object HelloWorld {
        def application(env:java.util.HashMap[String, Object]): Array[Object] = {
                val headers = new java.util.HashMap[String, Object]() {
                        put("Content-Type", "text/html")
                        put("Server", Array("uWSGI", "Unbit"))
                }
                return Array(200:java.lang.Integer, headers , "Hello World")
        }
   }

一旦用 ``scalac <filename>`` 编译好了，你可以像这样运行：

.. code-block:: sh

   ./uwsgi --socket /tmp/uwsgi.socket --plugins jvm,jwsgi --jwsgi HelloWorld:application --threads 40
