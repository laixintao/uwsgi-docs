Python Tracebacker
==================

.. versionadded:: 1.3-dev

通常，如果你想要获取你的app的实时回溯，那么你必须修改你的代码，为其添加一个hook或者入口，正如:doc:`TipsAndTricks`页面上描述的。

从1.3-dev开始，uWSGI包括了一个类似的技术，允许你通过一个UNIX socket获取实时回溯。

要启用这个回溯器，你需要添加选项``py-tracebacker=<socket>``,，其中，``<socket>``是已创建UNIX socket的_basename_。

如果你有4个uWSGI worker，并且添加了``py-tracebacker=/tmp/tbsocket``，那么将会创建名字从``/tmp/tbsocket1``到``/tmp/tbsocket4``的4个socket。

连接到其中任意一个都将会返回worker中运行的线程的当前回溯。你可以使用你最喜欢的应用或方法来连接到那些socket，但是uWSGI有一个供你使用的方便的选项``connect-and-read``：

  uwsgi --connect-and-read /tmp/tbsocket1

一个例子
----------

让我们写一个名为``slow.py``的蠢蠢的测试应用：

.. code-block:: python

  import time  
  
  def dormi():
    time.sleep(60)
  
  def dormi2():
    dormi()
  
  def dormi3():
    dormi2()

  def dormi4():
    dormi3()
  
  def dormi5():
    dormi4()
  
  def application(e, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    dormi5()
    return "hello"

然后运行它::

  uwsgi --http :8080 -w slow --master --processes 2 --threads 4 --py-tracebacker /tmp/tbsocket.

然后创建一堆到它的请求::

  curl http://localhost:8080 &
  curl http://localhost:8080 &
  curl http://localhost:8080 &
  curl http://localhost:8080 &

现在，当这些请求运行的时候 (每个都会花费几乎一分钟来完成)，你就可以检索回溯，比方说，前两个worker::

  ./uwsgi --connect-and-read /tmp/tbsocket.1
  ./uwsgi --connect-and-read /tmp/tbsocket.2

回溯器的输出将会是这样的::

  *** uWSGI Python tracebacker output ***
  
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 22 function = application line = dormi5()
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 14 function = dormi5 line = def dormi5(): dormi4()
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 13 function = dormi4 line = def dormi4(): dormi3()
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 12 function = dormi3 line = def dormi3(): dormi2()
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 11 function = dormi2 line = def dormi2(): dormi()
  thread_id = uWSGIWorker1Core1 filename = ./slow.py lineno = 9 function = dormi line = time.sleep(60)
  
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 22 function = application line = dormi5()
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 14 function = dormi5 line = def dormi5(): dormi4()
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 13 function = dormi4 line = def dormi4(): dormi3()
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 12 function = dormi3 line = def dormi3(): dormi2()
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 11 function = dormi2 line = def dormi2(): dormi()
  thread_id = uWSGIWorker1Core3 filename = ./slow.py lineno = 9 function = dormi line = time.sleep(60)
  
  thread_id = MainThread filename = ./slow.py lineno = 22 function = application line = dormi5()
  thread_id = MainThread filename = ./slow.py lineno = 14 function = dormi5 line = def dormi5(): dormi4()
  thread_id = MainThread filename = ./slow.py lineno = 13 function = dormi4 line = def dormi4(): dormi3()
  thread_id = MainThread filename = ./slow.py lineno = 12 function = dormi3 line = def dormi3(): dormi2()
  thread_id = MainThread filename = ./slow.py lineno = 11 function = dormi2 line = def dormi2(): dormi()
  thread_id = MainThread filename = ./slow.py lineno = 9 function = dormi line = time.sleep(60)

将回溯器与Harakiri组合在一起
---------------------------------------

如果一个请求由于:term:`harakiri<Harakiri>`特性而被杀掉，那么在Harakiri阶段，会自动记录回溯。