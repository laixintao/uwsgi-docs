Gevent循环引擎
======================

`Gevent`_ 是一个令人惊奇的非阻塞Python网络库，构建在
``libev`` 和 ``greenlet`` 之上。虽然uWSGI支持Greenlet作为挂起-恢复/绿色线程/协程库，但是还需要大量的努力和代码修改才能对gevent起作用。gevent插件要求gevent
1.0.0 和 :doc:`Async` 模式。

.. _Gevent: http://www.gevent.org

注意
-----

*  :doc:`SignalFramework` 完全对Gevent模式有效。每个处理函数将会在一个专用的greenlet中执行。看看 :file:`tests/ugevent.py` 这个例子。
* uWSGI多线程模式 (``threads`` 选项) 对Gevent无效。支持在你的应用中运行Python多线程。
* 把uWSGI的异步API和gevent混在一起是**明确禁止的**。

构建插件 (uWSGI >= 1.4)
----------------------------------

当使用默认配置文件的时候，会默认编译gevent插件。
执行以下命令将会安装python插件，以及gevent：

.. code-block:: sh

   pip install uwsgi


构建插件 (uWSGI < 1.4)
---------------------------------

可以在 :file:`buildconf` 目录中找到一个"gevent"构建配置文件。

.. code-block:: sh

  python uwsgiconfig --build gevent
  # or...
  UWSGI_PROFILE=gevent make
  # or...
  UWSGI_PROFILE=gevent pip install git+git://github.com/unbit/uwsgi.git
  # or...
  python uwsgiconfig --plugin plugins/gevent # external plugin

以gevent模式运行uWSGI
----------------------------

.. code-block:: sh

   uwsgi --gevent 100 --socket :3031 --module myapp

或者对于模块化构建：

.. code-block:: sh

   uwsgi --plugins python,gevent --gevent 100 --socket :3031 --module myapp

--gevent的参数是要生成的异步核数


一个疯狂的例子
---------------

以下例子显示如何在请求中休眠，如何发起异步网络请求，以及如何在一个请求已经被关闭之后继续进行一些逻辑处理。

.. code-block:: python

  import gevent
  import gevent.socket
  
  def bg_task():
      for i in range(1,10):
          print "background task", i
          gevent.sleep(2)
  
  def long_task():
      for i in range(1,10):
          print i
          gevent.sleep()
  
  def application(e, sr):
      sr('200 OK', [('Content-Type','text/html')])
      t = gevent.spawn(long_task)
      t.join()
      yield "sleeping for 3 seconds...<br/>"
      gevent.sleep(3)
      yield "done<br>"
      yield "getting some ips...<br/>"
      urls = ['www.google.com', 'www.example.com', 'www.python.org', 'projects.unbit.it']
      jobs = [gevent.spawn(gevent.socket.gethostbyname, url) for url in urls]
      gevent.joinall(jobs, timeout=2)
  
      for j in jobs:
          yield "ip = %s<br/>" % j.value
  
      gevent.spawn(bg_task) # this task will go on after request end

猴子补丁
---------------

uWSGI使用原生gevent api，因此，并不需要猴子补丁。即便如此，你的代码也可能需要它，因此，记得在你的应用的开头调用 ``gevent.monkey.patch_all()`` 。自uWSGI 1.9起，便利的
``--gevent-monkey-patch`` 选项将会为你完成这个工作。
请注意，uWSGI是在你的应用 **启动** 的时候进行猴子补丁的，而不是在你的应用 **加载** 之前。因此，如果你在加载应用的时候加载其他模块，那么你或许仍然需要自己调用
``gevent.monkey.patch_all()`` 。

一个常见的例子是将 ``psycopg2_gevent`` 用于django。Django会为每个线程对postgres发起连接 (将其存储在线程变量中)。

随着uWSGI gevent插件运行在一个单一的线程中，这个方法将会导致psycopg中的死锁。启用猴子补丁将会让你映射线程局部变量到greenlet (虽然你可以避免完全猴子补丁，并只调用
``gevent.monkey.patch_thread()``) ，然后解决这个问题：

.. code-block:: python 

   import gevent.monkey
   gevent.monkey.patch_thread()
   import gevent_psycopg2
   gevent_psycopg2.monkey_patch()

或者 (为所有东西打上猴子补丁)

.. code-block:: python 

   import gevent.monkey
   gevent.monkey.patch_all()
   import gevent_psycopg2
   gevent_psycopg2.monkey_patch()

客户端和前端的一些注意事项
------------------------------

* 如果你正在测试一个生成一连串数据的WSGI应用，那么你应该知道，默认情况下 ``curl`` 缓存数据知道一个新行出现。因此，你要么确保使用 ``-N`` 标记来禁用curl的缓存，要么在你的输出中确保有常规的新行。
* 如果你在uWSGI之前使用Nginx，并且希望从你的应用流数据，那么你可能会想要禁用Nginx的缓存。
  
.. code-block:: nginx
  
    uwsgi_buffering off;
