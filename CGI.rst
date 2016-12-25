在uWSGI上运行CGI脚本
============================

CGI插件提供了使用uWSGI服务器运行CGI脚本的能力。

Web服务器/客户端／负载均衡器使用modifier ``9`` 发送请求给uWSGI服务器。然后，uWSGI使用从客户端传过来的变量作为CGI变量（有时会修复它们），调用对应的脚本／可执行文件，再转发其输出到客户端。

该插件会尝试模仿apache的行为，允许你即使在那种原生不支持CGI的服务器，例如nginx上，也能运行CGI脚本。

启用插件
-------------------

默认情况下，不会将CGI插件构建到核心部分中。你需要构建一个嵌入了cgi的二进制文件，或者构建cgi插件。

要构建带CGI支持的单个二进制文件：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s cgi /tmp/uwsgi

要将其作为插件编译，

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/cgi
   
或者，在源代码目录：

.. code-block:: sh

   make PROFILE=cgi

配置CGI模式
--------------------

``cgi <[mountpoint=]path>`` 选项是配置你的CGI环境的主要入口点。

``path`` 可以是一个目录或者一个可执行文件。如果是一个目录，CGI插件将会使用该URI来查找脚本路径。如果传递了一个可执行文件，那么将会运行它，并且会在其环境中设置 ``SCRIPT_NAME``, ``SCRIPT_FILENAME`` 和 ``PATH_INFO`` 。

``mountpoint`` 是可选的。你可以用它来映射不同的URI到不同的CGI目录／脚本。


注意
-----

* 记得让你的CGI应用使用uWSGI的资源限制和jailing技术 (名字空间，chroot, capability, unshare....)，来限制它们可能会造成的损害。
* 从uWSGI 2.0.2开始，你可以通过使用异步模式，拥有甚至更加便宜的并发。
* 如果没有映射到一个辅助函数，那么每个CGI脚本必须拥有读取和执行权限。

例子
--------

例子1:启用CGI的哑目录
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: ini

  [uwsgi]
  plugins = cgi
  socket = uwsgi.sock
  cgi = /var/www/cgi-bin

每个请求将会搜索 :file:`/var/www/cgi-bin` 中的指定文件，并执行它。

到 ``http://example.com/foo.cgi`` 的请求会运行 ``/var/www/cgi-bin/foo.cgi`` 。

例子2:老式cgi-bin目录
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: ini

  [uwsgi]
  plugins = cgi
  socket = uwsgi.sock
  cgi = /cgi-bin=/var/lib/cgi-bin

对 ``http://example.com/cgi-bin/foo`` 的调用将会运行 ``/var/lib/cgi-bin/foo`` 。

例子3:限制使用某些扩展
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们只想执行.cgi和.pl文件：

.. code-block:: ini
  
  [uwsgi]
  plugins = cgi
  socket = uwsgi.sock
  cgi = /cgi-bin=/var/lib/cgi-bin
  cgi-allowed-ext = .cgi
  cgi-allowed-ext = .pl

例子4:使用脚本扩展名，映射脚本到解释器
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们想要通过 ``php5-cgi`` 二进制文件，运行目录 ``/var/www`` 中的以 ``.php`` 结尾的文件：

.. code-block:: ini

  [uwsgi]
  plugins = cgi
  socket = uwsgi.sock
  cgi = /var/www
  cgi-allowed-ext = .php
  cgi-helper = .php=php5-cgi

如果用辅助函数运行一个文件，那么运行的文件将不需要执行权限。当然，辅助函数是需要的。

扩展名比较是不区分大小写的。

例子5:通过nginx，将PHP脚本作为CGI运行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

配置Nginx来传递.php请求到uWSGI，使用 :file:`/var/www/foo` 作为文档根目录。

.. code-block:: nginx

  location ~ .php$ {
    include uwsgi_params;
    uwsgi_param REDIRECT_STATUS 200; # required by php 5.3
    uwsgi_modifier1 9;
    uwsgi_pass 127.0.0.1:3031;
  }

并且像这样配置uWSGI：

.. code-block:: ini

  [uwsgi]
  plugins = cgi
  socket = 127.0.0.1:3031
  cgi = /var/www/foo
  cgi-allowed-ext = .php
  cgi-helper = .php=php5-cgi

例子6:并发
^^^^^^^^^^^^^^^^^^^^^^

默认情况下，每个uWSGI worker将能够运行单个CGI脚本。这意味着，使用一个进程将会阻塞传入的请求，直到结束了第一个请求。

添加更多的worker将会缓解这个问题，但是也会消耗大量内存。

线程是个更好的选择。让我们配置每个worker进程运行20个worker线程，这样，并发运行20个CGI脚本。

.. code-block:: ini

  [uwsgi]
  plugins = cgi
  threads = 20
  socket = 127.0.0.1:3031
  cgi = /var/www/foo
  cgi-allowed-ext = .php
  cgi-helper = .php=php5-cgi
  
  
使用异步模式来得到甚至更廉价的并发：


.. code-block:: ini

  [uwsgi]
  plugins = cgi
  async = 200
  ugreen = true
  socket = 127.0.0.1:3031
  cgi = /var/www/foo
  cgi-allowed-ext = .php
  cgi-helper = .php=php5-cgi
  
这将会生成200个协程，每个能够管理一个CGI脚本（使用几k内存）
  

例子7:nginx之后到Mailman web接口
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: nginx

  location /cgi-bin/mailman {
      include uwsgi_params;
      uwsgi_modifier1 9;
      uwsgi_pass 127.0.0.1:3031;
  }


.. code-block:: ini

  [uwsgi]
  plugins = cgi
  threads = 20
  socket = 127.0.0.1:3031
  cgi = /cgi-bin/mailman=/usr/lib/cgi-bin/mailman
  cgi-index = listinfo

``cgi-index`` 指令指定了当请求一个以斜杠结尾的路径时，运行哪个脚本。这样， ``/cgi-bin/mailman/`` 将会被映射到 ``/cgi-bin/mailman/listinfo`` 脚本。

例子8:在一个子目录中作为CGI的Viewvc
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用挂载点选项。
.. code-block:: ini

  [uwsgi]
  plugins = cgi
  threads = 20
  socket = 127.0.0.1:3031
  cgi = /viewvc=/usr/lib/cgi-bin/viewvc.cgi

例子9:使用uWSGI HTTP路由器和 ``check-static`` 选项
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是一个非常全栈的解决方案，只使用运行在8080端口上的uWSGI。

.. code-block:: ini

  [uwsgi]
  plugins = http, cgi
  
  ; bind on port 8080 and use the modifier 9
  http = :8080
  http-modifier1 = 9
  
  ; set the document_root as a placeholder
  my_document_root = /var/www
  
  ; serve static files, skipping .pl and .cgi files
  check-static = %(my_document_root)
  static-skip-ext = .pl
  static-skip-ext = .cgi
  
  ; run cgi (ending in .pl or .cgi) in the document_root
  cgi = %(my_document_root)
  cgi-index = index.pl
  cgi-index = index.cgi
  cgi-allowed-ext = .pl
  cgi-allowed-ext = .cgi

例子10:优化CGI (高级)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你可以避免在每次请求上重新运行解释器的开销，启动时加载解释器（们），并且调用它们中的函数，而不是 ``execve()`` 解释器本身。

源代码发行版中的 :file:`contrib/cgi_python.c` 文件是关于如何优化Python CGI脚本的一个小小的例子。

启动时加载Python解释器，并且在每次 ``fork()`` 后，调用 ``uwsgi_cgi_run_python`` 。

要编译该库，你可以使用像这样的命令：

.. code-block:: sh

  gcc -shared -o cgi_python.so -fPIC -I /usr/include/python2.7/ cgi_python.c -lpython2.7

然后映射 ``.py`` 文件到 ``uwsgi_cgi_run_python`` 函数。


.. code-block:: ini

  [uwsgi]
  plugins = cgi
  
  cgi = /var/www
  cgi-loadlib = ./cgi_python.so:uwsgi_cgi_load_python
  cgi-helper = .py=sym://uwsgi_cgi_run_python

}}}

记得在辅助函数中给符号加上 ``sym://`` 前缀，让uWSGI把它当成一个已加载符号进行搜索，而不是一个磁盘文件。
