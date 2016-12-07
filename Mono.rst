Mono ASP.NET插件
=======================

uWSGI 1.9添加了对Mono平台的支持，特别是对于ASP.NET基础设施。

The most common way to deploy 部署Mono ASP.NET应用的最常见方式是使用XSP项目，它是一个简单的web服务器网关，实现了HTTP和FastCGI协议。

通过Mono插件，你将能够给直接在uWSGI中托管ASP.net应用，在你的应用中免费获得它所有的特性。

正如所有其他uWSGI插件一样，你可以使用 :doc:`RPC` 子系统调用所有从其他语言导出的函数。

构建uWSGI + Mono
*********************

你可以作为插件或者在一个单片构建中构建Mono支持。

有一个名为"mono"的构建配置文件可用，让这个任务相当简单。

确保你的系统中安装了mono。你需要Mono头部， ``mcs`` 编译器和System.Web组合。在标准的mono发行版中都可以找到它们。

在最近的Debian/Ubuntu系统中，你可以使用

.. code-block:: sh

    apt-get install build-essential python mono-xsp4 asp.net-examples

``mono-xsp4`` 是一次性安装所有我们需要的东东的窍门，而ASP.net样例将会被用来测试我们的安装。

我们可以构建一个嵌入了Mono的单片uWSGI发行版：

.. code-block:: sh

   UWSGI_PROFILE=mono make

在这个过程的最后 (如果一切顺利的话)，你将会得到到 ``uwsgi.dll`` 组合的路径。

你或许想要在你的GAC中安装它 (使用gacutil -i <path>)，以避免每次都要指定其路径。这个库让你访问来自Mono应用的uWSGI api。

启动服务器
*******************

Mono插件有一个官方的 ``modifier1``, 15。

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-app = /usr/share/asp.net-demos
   mono-index = index.asp

前一个步骤假设已在GAC中安装了uwsgi.dll，如果你并非如此，那么你可以这样强制它的路径使用：

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-app = /usr/share/asp.net-demos
   mono-index = index.asp
   mono-assembly = /usr/lib/uwsgi/uwsgi.dll

``/usr/share/asp.net-demos`` 是包含Mono的样例ASP.net应用的目录。

如果启动uWSGI，那么你会得到一个关于不能找到uwsgi.dll的错误，你可以这样强制使用一个特定的路径

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-app = /usr/share/asp.net-demos
   mono-index = index.asp
   mono-assembly = /usr/lib/uwsgi/uwsgi.dll
   env = MONO_PATH=/usr/lib/uwsgi/

或者你可以简单将uwsgi.dll拷贝到你的站点目录的 (在这个情况下，是 ``/usr/share/asp.net-demos`` )  ``/bin`` 目录下。

``mono-index`` 选项是用来设置当请求一个目录时的搜索文件的。你可以多次指定它。

钩子之下：mono键
****************************

前一个例子应该会完美运行，但是在内部，做了大量的假设。

整个mono插件依赖于"键"概念。一个键是一个.net应用的唯一标识。在这个例子中，该应用的键是"/usr/share/asp.net-demos"。这是一种键1:1映射到虚拟机的情况。要将一个虚拟机路径映射到一个指定的键，你可以使用

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-app = /foobar=/usr/share/asp.net-demos

现在/foobar键映射到/usr/share/asp.net-demos .net应用。

默认情况下，请求键会被映射到DOCUMENT_ROOT变量。因此，在这个新的情况下，/foobar应该是DOCUMENT_ROOT的值。

但是uWSGI http路由器并没有DOCUMENT_ROOT的概念，因此前一个工作要怎么工作呢？这是因为第一个加载的应用通常是默认应用，因此mono插件，不能够查找到一个应用，则返回默认的。

把DOCUMENT_ROOT当成键使用会相当受限的。因此可用--mono-key选项。让我们使用uWSGI内部路由来构建一个海量虚拟主机栈

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-key = MONO_APP
   route-run = addvar:MONO_APP=/var/www/asp/${HTTP_HOST}
   
MONO_APP并不是mono插件会用来搜索应用的变量 (取代DOCUMENT_ROOT)。

多亏了内部路径，我们（动态）设置它为指定主机应用的根目录，因此，到example.com的请求将会被映射到/var/www/asp/example.com


并发和fork()不友好
**************************************

由于Mono VM并非 ``fork()`` 友好的，因此会为每个worker生成一个新的VM。这确保了你可以在多进程模式下运行你的应用。

Mono拥有相当稳定的多线程支持，并且它与uWSGI的线程支持工作良好。

.. code-block:: ini

   [uwsgi]
   http = :9090
   http-modifier1 = 15
   mono-app = /usr/share/asp.net-demos
   mono-index = index.asp
   mono-assembly = /usr/lib/uwsgi/uwsgi.dll
   env = MONO_PATH=/usr/lib/uwsgi/
  
   master = true
   processes = 4
   threads = 20

有了这个而设置，你会生成4个进程，每个进程有20个线程。试着不要依赖于单个进程。尽管在所谓的“企业环境”下，这是一个常见的设置，但是拥有多进程确保了更好的可用性 (多亏了master工作)。这个规则（例如）甚至应用到了 :doc:`JVM` 插件上。

API访问
**********

这是一个在进行中的工作。目前，只导出了几个函数。 :doc:`RPC` 和信号子系统，以及 :doc:`Caching` 框架将是高优先级的。

技巧
******

如往常一样，uWSGI试着优化（可能的话）你的应用的“常见”操作。提供静态文件服务会被自动加速 (或者当卸载启用的时候进行卸载)，并且会缓存所有的路径解析。
