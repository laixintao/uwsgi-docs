使用uWSGI提供静态文件 (更新至1.9)
================================================

不幸的是，你不能无需通过一些协议 (HTTP, SPDY或者别的什么的) 提供静态文件服务。

幸运的是，对于提供静态文件，uWSGI有一系列广泛的选项了微优化。

一般来说，你所选的web服务器 (Nginx, Mongrel2等) 将高效快速地提供静态文件，并且会直接把动态请求转发到uWSGI后端节点。

uWSGI项目的主要目标是ISPs和PaaS (即，托管市场)，其中，一般你会想要避免在中央服务器上生成磁盘I/O，并且让每个用户专有的区域自己处理（负责）它。更重要的是，你想要让客户在无需打扰你的系统管理员的情况下自定义他们提供静态资产服务的方式。

模式1：在传递请求给你的应用之前检查静态资源
**************************************************************************

在web应用中管理静态文件是一种相当常见的方式。诸如Ruby on Rails这样的关键和许多PHP应用已使用这种方法好多年了。

假设你的静态资产位于 ``/customers/foobar/app001/public`` 之下。你想要在传递请求给你的动态应用之前检查在那个目录下请求都有一个对应的文件。 ``--check-static`` 选项就是为你而设的：

.. code-block:: sh

   --check-static /customers/foobar/app001/public

如果uWSGI接收到一个对 ``/foo.png`` 的请求，那么它将首先检查 ``/customers/foobar/app001/public/foo.png`` 是否存在，如果不存在，那么它会调用你的应用。

你可以多次指定 ``--check-static`` ，来指定多个可能的根路径。

.. code-block:: sh

   --check-static /customers/foobar/app001/public --check-static /customers/foobar/app001/static

uWSGI将首先检查 ``/customers/foobar/app001/public/foo.png`` ；如果它找不到它，那么它会在最后委托请求到你的应用之前尝试 ``/customers/foobar/app001/static/foo.png`` 。

模式2：信任前端DOCUMENT_ROOT
**************************************

如果你的前端 (一个web服务器，一个uWSGI核心路由器……) 设置了 ``DOCUMENT_ROOT`` 值，那么你可以使用 ``--check-static-docroot`` 选项，指示uWSGI将其作为一个检查静态文件的有效目录。

模式3：使用静态文件挂载点
**************************************

一个更一般的方法是“映射”指定请求前缀到你的文件系统上的物理目录。

 ``--static-map mountpoint=path`` 选项可以实现此目的。

.. code-block:: sh

   --static-map /images=/var/www/img

如果你接收到一个对 ``/images/logo.png`` 的请求，并且 ``/var/www/img/logo.png`` 存在，那么将会提供它。否则，你的应用会管理这个请求。

你可以多次指定 ``--static-map`` 选项，即使是对于同一个挂载点。

.. code-block:: sh

   --static-map /images=/var/www/img --static-map /images=/var/www/img2 --static-map /images=/var/www/img3

在找到文件之前会在每个目录中进行搜索，如果找不到，这个请求就会交由你的应用管理。

在一些特定的情况下，你或许想要以一种不同的方式构建内部路径，保留请求的原始路径部分。 ``--static-map2`` 选项会完成此项工作。

.. code-block:: sh

   --static-map2 /images=/var/www/img

对 ``/images/logo.png`` 的请求将会被当成 ``/var/www/img/images/logo.png`` 。

你可以同时 ``map`` (或者 ``map2``) 目录和文件。

.. code-block:: sh

   --static-map /images/logo.gif=/tmp/oldlogo.gif
   # (psst: put favicons here)


模式4：使用高级内部路由
***************************************

当映射不够的时候，高级内部路由 (自1.9起可用) 将是你最后的手段。

多亏了强大的正则表达式，你将能够构建非常复杂的映射。

.. code-block:: ini

   [uwsgi]
   route = /static/(.*)\.png static:/var/www/images/pngs/$1/highres.png
   route = *\.jpg static:/var/www/always_the_same_photo.jpg

设置首页
**********************

默认情况下，对“目录” (例如/或者/foo)的请求会被跳过 (如果未使用高级内部路由)。

如果你想要映射指定文件到一个“目录”请求 (如久负盛名的index.html) ，那么仅需使用 ``--static-index`` 选项。

.. code-block:: sh

   --static-index index.html --static-index index.htm --static-index home.html

正如其他选项一样，第一个匹配将会终止整个链。

MIME类型
**********

你的对静态文件的HTTP/SPDY/随便什么响应应该总是返回特定文件正确的mime类型，从而让用户代理正确地处理它们。

默认情况下，uWSGI从 ``/etc/mime.types`` 文件构建它的MIME类型列表。你可以使用 ``--mime-file`` 选项加载额外的文件。

.. code-block:: sh

   --mime-file /etc/alternatives.types --mime-file /etc/apache2/mime.types

所有的文件将会被组合成单个自动优化的链表。

跳过指定扩展
****************************

一些平台/语言，最值得注意的是基于CGI的，例如PHP，是以一种非常简单的方式进行部署的。

你只需把它们放在文档的根目录下，然后每当你调用它们的时候，就会执行。

当与静态文件服务组合的时候，这个方法需要你注意一下，避免你的CGI/PHP/其他什么的会像静态文件一样被提供。

 ``--static-skip-ext`` 将会达到理想效果。

CGI和PHP部署上的一个非常常见的模式是这个：

.. code-block:: sh

   --static-skip-ext .php --static-skip-ext .cgi --static-skip-ext .php4


设置Expires头部
***************************

当提供静态文件的时候，滥用客户端浏览器缓存是明智之选。默认情况下，uWSGI会添加一个 ``Last-Modified`` 头部到所有的静态响应中，并且将遵循 ``If-Modified-Since`` 请求头部。

这对于高流量站点或许不够。你可以使用以下选项之一来添加自动 ``Expires`` 头部：

* ``--static-expires-type`` 将会对指定的MIME类型设置Expires头部为指定秒数。
* ``--static-expires-type-mtime`` 类似，但是基于文件修改时间，而非当前时间。
* ``--static-expires`` (和 ``-mtime``) 将会为所有匹配指定正则表达式的文件名（在结束映射到文件系统之后）设置Expires头部。
* ``--static-expires-uri`` (和 ``-mtime``) 针对 ``REQUEST_URI`` 匹配正则表达式
* ``--static-expires-path-info`` (和 ``-mtime``) 针对 ``PATH_INFO`` 匹配正则表达式

.. code-block:: sh

   # Expire an hour from now
   --static-expires-type text/html=3600
   # Expire an hour from the file's modification time
   --static-expires-type-mtime text/html=3600
   # Same as static-expires-type, but based on a regexp:
   --static-expires /var/www/static/foo*\.jpg 3600

传输模式
**************

如果你开发了一个异步/非阻塞应用，那么直接从uWSGI提供静态文件就不是一个大问题。

所有的传输都是以异步方式进行管理的，因此在此期间，你的应用都不会阻塞。

在多进程/多线程模式下，你的进程（或者线程）将会在文件的整个传输期间阻塞。

对于较小的文件，这不是个问题，但是对于较大的，将它们的传输卸载到其他什么的会是一个不错的想法。

你有多种方式做到这点：

X-Sendfile
^^^^^^^^^^

如果你的web服务器支持X-Sendfile头部，并且访问了你想要发送的文件 (例如，它位于你的应用所在的机器上，或者可以通过NFS访问它)，那么你可以用 ``--file-serve-mode x-sendfile`` 选项避免从你的应用中传输这个文件。

使用这个，uWSGI将只会生成响应头，并且将会委托web服务器传输物理文件。

X-Accel-Redirect
^^^^^^^^^^^^^^^^

目前（2013年1月）只有Nginx支持。与X-Sendfile的工作方式相同，唯一的区别在于选项参数。

.. code-block:: sh

   --file-serve-mode x-accel-redirect

卸载
^^^^^^^^^^ 

如果你的前端服务器不能访问静态文件的话，这是最好的方法。它使用 :doc:`OffloadSubsystem` 来委托文件传输给非阻塞线程池。

这些线程每一个都能并发管理数以千计的文件传输。

要启用文件传输卸载，只需使用选项 ``--offload-threads`` ，指定要生成线程的数目 (试着将其设为CPU核心数，来利用SMP)。

GZIP (uWSGI 1.9)
****************

uWSGI 1.9可以检测一个静态文件的 ``*.gz`` 变体。

许多用户/系统管理员低估了即使Gzip编码的CPU影响。

每次都压缩文件 (除非你对web服务器以某种方式对它们进行缓存) 将使用CPU，而你将不能够使用高级的 (零拷贝) 技术，例如 ``sendfile()`` 。对一个非常满载的站点（或网络），这可能会是个问题（特别是当gzip编码对于一个更好更灵敏的用户体验而言是必须的时候）。

虽然uWSGI能够即时压缩内容 (例如，这用于HTTP/HTTPS/SPDY路由器)，但是提供gzip压缩的静态文件最好的方法是“手动” (但请使用脚本，而不是让实习生来做这件事) 生成它们，然后让uWSGI选择每次最好提供压缩的还是未压缩的。

用这种方式，提供gzip内容将与提供标准静态文件 (sendfile, 卸载……) 没啥区别

要触发这个行为，你有多种选择：

* ``static-gzip <regexp>`` 对所有匹配到指定正则表达式（该正则表达式被应用到文件的完整文件系统路径）的请求文件检测.gz变体
* ``static-gzip-dir <dir>``/``static-gzip-prefix <prefix>`` 对位于指定目录下的所有的文件检测.gz变体
* ``static-gzip-ext <ext>``/``static-gzip-suffix <suffix>`` 对所有具有指定扩展/后缀的文件检测.gz变体
* ``static-gzip-all`` 对所有请求的静态文件检测.gz变体

所以基本上，如果你有 ``/var/www/uwsgi.c`` 和 ``/var/www/uwsgi.c.gz`` ，那么接收gzip作为它们的Content-Encoding的客户端将会透明地提供gzip压缩版本。

安全
********

每一个静态映射都将完整转换成“正在的”路径 (因此也会转换符号链接)。

如果结果路径并不位于选项中指定的路径下，那么将会触发一个安全性错误，并且拒绝该请求。

如果你信任你的UNIX技能，并且知道你在做什么，那么你可以添加“安全”路径列表。如果一个转换路径并不位于一个配置目录下，但位于一个安全路径下，那么仍然会提供。

例如：

.. code-block:: sh

   --static-map /foo=/var/www/

``/var/www/test.png`` 是到 ``/tmp/foo.png`` 的符号链接

在 ``/foo/test.png`` 转换后，uWSGI会引发一个安全性错误，因为 ``/tmp/foo.png`` 并不位于 ``/var/www/`` 下。

使用

.. code-block:: sh

   --static-map /foo=/var/www/ --static-safe /tmp

将绕过那个限制。

你可以多次指定 ``--static-safe`` 选项。

缓存路径映射/解析
**********************************

静态文件服务的瓶颈之一是恒定大量的 ``stat()`` 系统调用。

你可以使用uWSGI缓存系统来存储从URI到文件系统路径的映射。

.. code-block:: sh

   --static-cache-paths 30

将会缓存每个静态文件转换在uWSGI缓存中，时间为30秒。

从uWSGI 1.9起，添加了一个更新的缓存子系统，运行你创建多个缓存。如果你想要存储转换到一个指定的缓存中，那么你可以使用 ``--static-cache-paths-name <cachename>`` 。

福利：在缓存中存储静态文件
**********************************************

你可以使用选项 ``--load-file-in-cache <filename>`` (可以多次指定它) 在启动期间直接把一个静态文件存储到uWSGI缓存中。该文件的内容将会被存储在键<filename>之下。

所以请注意 —— ``load-file-in-cache ./foo.png`` 将会把这个项作为 ``./foo.png`` 存储，而非它的完整路径。

注意事项
*********

* 静态文件服务子系统自动遵循If-Modified-Since HTTP请求头
