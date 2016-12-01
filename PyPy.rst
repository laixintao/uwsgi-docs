PyPy插件
===============

要求uWSGI >= 2.0.9

介绍
^^^^^^^^^^^^

想法/设计：Maciej Fijalkowski

贡献者：Alex Gaynor, Armin Rigo

自uWSGI 1.9.11起，就有了一个基于cffi的PyPy插件。老且慢的基于cpyext的已经从版本树中移除了。

目前，只在Linux系统上支持该插件。接下来的发布版本也会支持其他平台。

这个插件在启动的时候加载 ``libpypy-c.so`` ，设置PyPy安装目录，并且执行一个实现插件逻辑的特别的Python模块。所以是哒，大多数的插件都是用Python实现的，并且理论上讲，这个方法将允许除了C, C++和Objective-C之外，直接用Python编写uWSGI插件。

截至2014年12月，已经合并了到PyPy的所有所需插件，因此你可以获取一个官方最新版本 (或者一个于2014年12月后发布的稳定版本)，然后与uWSGI一起使用它。


安装带PyPy支持的uWSGI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

与往常一样，使用uWSGI，你有不同的方法基于你的需求安装uWSGI。

如果你已在PyPy家目录安装了pip，那么可以运行

.. code-block:: sh

   pip install uwsgi
  
这个uwsgi setup.py文件将会识别PyPy环境，并且构建一个仅PyPy的uWSGI二进制文件。

使用相同的方式，你可以执行uWSGI源码中提供的setup.py：

.. code-block:: sh

    pypy setup.py install
    

(这两种方法将会把pypy家目录硬编码到uWSGI二进制文件中，因此，你无需在你的选项中设置pypy-home)


或者你可以手工编译：

.. code-block:: sh

   UWSGI_PROFILE=pypy make
   
或者可以使用网络安装程序：

.. code-block:: sh

   curl https://uwsgi.it/install | bash -s pypy /tmp/uwsgi
   
这将会在 ``/tmp/uwsgi`` 中构建一个uWSGI + PyPy二进制文件

或者你可以把PyPy支持当成插件构建。

.. code-block:: sh

   uwsgi --build-plugin plugins/pypy

或者(旧式)

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/pypy
   
PyPy家目录
^^^^^^^^^^^^^

uWSGI Python插件 (更准确的说，是CPython插件) 通过链接 ``libpython`` 工作。这意味着，对于Python的每一个不同的版本，你都需要重新构建这个插件。而这个PyPy插件不同，因为libpypy-c是在启动时加载的，而它的符号在运行时解决。这允许你动态迁移到一个不同的PyPy版本。

这个方法的“缺点”是，你需要在运行时通知uWSGI你的PyPy安装在哪里 (除非你是通过pip或者使用setup.py脚本来安装uwsgi的，在这样的情况下，会自动找到家目录)

假设你的PyPy位于 ``/opt/pypy`` ，你可以这样启动uWSGI：

.. code-block:: sh

   uwsgi --http-socket :9090 --pypy-home /opt/pypy
  
通过这个命令行，uWSGI将会搜索 ``/opt/pypy/bin/libpypy-c.so`` ，如果找到，它将会设置那个路径为PyPy家目录。

如果你的 ``libpypy-c.so`` 位于PyPy家目录之外 (并且位于一个动态链接器不可达的目录中)，那么你可以使用 ``--pypy-lib`` 选项。

.. code-block:: sh

   uwsgi --http-socket :9090 --pypy-home /opt/pypy --pypy-lib /opt/libs/libpypy-c.so
   
通过这个方法，你可以使用来自一个指定PyPy构建的库，以及来自其他的家目录。

.. note:: 如果你想指向当前目录下的一个.so文件，那么记得给--pypy-lib加上前缀./！

PyPy设置文件
^^^^^^^^^^^^^^^^^^^

如前所述，大多数的uWSGI PyPy插件都是用Python写的。该代码于运行时加载，你也可以自定义它。

是哒，这就意味着你无需重新构建uWSGI就可以改变这个插件的工作方式。

``pypy_setup.py`` 文件的默认版本被嵌入到uWSGI二进制文件中，而它自动在启动时加载。

如果你想改变它，那么仅需通过 ``--pypy-setup`` 选项传递另一个文件名。

.. code-block:: sh

   uwsgi --http-socket :9090 --pypy-home /opt/pypy --pypy-lib /opt/libs/libpypy-c.so --pypy-setup /home/foobar/foo.py
   
这个Python模块实现了uWSGI钩子和用于访问来自你的应用的uWSGI API的虚拟 ``uwsgi`` python模块。

如果你想要检索嵌入的pypy_setup.py文件的内容，那么你可以通过使用 ``print-sym`` 这个便捷选项来从二进制符号中读取它。

.. code-block:: sh

   uwsgi --print-sym uwsgi_pypy_setup

WSGI支持
^^^^^^^^^^^^

该插件实现了PEP 333和PEP 3333。你可以同时加载WSGI模块和 ``mod_wsgi`` 样式的 ``.wsgi`` 文件。

要加载一个WSGI模块 (它必须位于你的Python路径中)：

.. code-block:: sh

   uwsgi --http-socket :9090 --pypy-home /opt/pypy --pypy-wsgi myapp
   
要加载一个WSGI文件：

.. code-block:: sh

   uwsgi --http-socket :9090 --pypy-home /opt/pypy --pypy-wsgi-file /var/www/myapp/myapp.wsgi
   
RPC支持
^^^^^^^^^^^

你可以使用 :func:`uwsgi.register_rpc` API函数来注册RPC函数，就像你可以使用普通的Python插件一样。

.. code-block:: py

   import uwsgi
   
   def hello():
       return "Hello World"
       
   uwsgi.register_rpc('hello', hello)
   
要调用RPC函数， :func:`uwsgi.rpc` 和 :func:`uwsgi.call` 都可以用。

.. code-block:: py

   import uwsgi
   
   uwsgi.rpc('192.168.173.100:3031', 'myfunc', 'myarg')
   uwsgi.call('myfunc', 'myarg')
   uwsgi.call('myfunc@192.168.173.100:3031', 'myarg')
   
   
(与本地RPC) 集成已经在PyPy和PyPy, PyPy和JVM, 以及PyPy和Lua之前测试过了。所有这些都工作得十分完美……因此，那意味着你可以调用来自PyPy的Java函数。

IPython技巧
^^^^^^^^^^^^^

用一个用于测试的运行中的shell是非常非常棒的。你可以使用IPython来实现。

.. code-block:: sh

   uwsgi --socket :3031 --pypy-home /opt/pypy --pypy-eval "import IPython; IPython.embed()" --honour-stdin
   
   
uWSGI API状态
^^^^^^^^^^^^^^^^

自20130526起，支持以下API函数、钩子和属性。

* :py:data:`uwsgi.opt`
* :py:data:`uwsgi.post_fork_hook`
* :func:`uwsgi.add_cron()`
* :func:`uwsgi.setprocname()`
* :func:`uwsgi.alarm()`
* :func:`uwsgi.signal_registered()`
* :func:`uwsgi.mule_id()`
* :func:`uwsgi.worker_id()`
* :func:`uwsgi.masterpid()`
* :func:`uwsgi.lock()`
* :func:`uwsgi.unlock()`
* :func:`uwsgi.add_file_monitor()`
* :func:`uwsgi.add_timer()`
* :func:`uwsgi.add_rb_timer()`
* :func:`uwsgi.cache_get()`
* :func:`uwsgi.cache_set()`
* :func:`uwsgi.cache_update()`
* :func:`uwsgi.cache_del()`
* :func:`uwsgi.signal()`
* :func:`uwsgi.call()`
* :func:`uwsgi.rpc()`
* :func:`uwsgi.register_rpc()`
* :func:`uwsgi.register_signal()`
  
选项
^^^^^^^


* ``pypy-lib`` - 加载指定的libpypy-s.so
* ``pypy-setup`` - 加载指定的pypy_setup脚本文件
* ``pypy-home`` - 设置pypy家目录
* ``pypy-wsgi`` - 加载一个WSGI模块
* ``pypy-wsgi-file`` - 加载一个mod_wsgi兼容的.wsgi文件
* ``pypy-eval`` - 在 ``fork()`` 之前执行指定的字符串
* ``pypy-eval-post-fork`` - 在每个 ``fork()`` 之后执行指定的字符串
* ``pypy-exec`` - 在 ``fork()`` 之前执行指定的python脚本
* ``pypy-exec-post-fork`` - 在每个 ``fork()`` 之后执行指定的python脚本
* ``pypy-pp/pypy-python-path/pypy-pythonpath`` - 添加指定项到pythonpath
* ``pypy-paste`` - 加载一个paste.deploy .ini配置
* ``pypy-ini-paste`` - 加载一个paste.deploy .ini配置，并使用它的[uwsgi]部分


注意事项
^^^^^^^^

* 混合libpython和libpypy-c是明确禁止的。pypy插件中的检查阻止你做这种地狱般的事情。
* 从一个Python程序员的观点来看，PyPy插件一般较为“正统”，而CPython在许多领域也许有点亵渎。我们已经能够进行抉择了，因为我们无需向后兼容较老的uWSGI发行版。
* uWSGI API仍然未完成
* WSGI加载器并不更新uWSGI内部应用列表，因此，诸如 ``--need-app`` 这样的东东并不能用。即使应用已成功被加载，服务器也会在启动时报告"dynamic mode"。这将会很快被修复。
