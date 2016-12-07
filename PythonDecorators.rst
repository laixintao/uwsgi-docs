uWSGI API - Python装饰器
=============================

 :doc:`The uWSGI API<PythonModule>` 非常底层，因为它必须是语言无关的。

也就是说，对于许多语言，例如Python，太底层并不是一件好事。

以吾之愚见，装饰器是Python较为牛逼的特性之一，因此，在uWSG源代码树中，你会发现有个模块导出了一堆的装饰器，它们涵盖了很大一部分uWSGI API。




小抄
-----

基于信号量的装饰器在第一个可用worker中执行信号处理器。如果你已经启动了spooler，那么你可以在其中执行信号处理器，让worker不用管它，只要管理正常的请求即可。简单传递 ``target='spooler'`` 给装饰器。

.. code-block:: py

    @timer(3, target='spooler')
    def hello(signum):
        print("hello")


例子：一个Django会话清理器和视频解码器
---------------------------------------------------

让我们定义一个 :file:`task.py` 模块，然后将其放进Django项目目录中。

.. code-block:: py

    from uwsgidecorators import *
    from django.contrib.sessions.models import Session
    import os

    @cron(40, 2, -1, -1, -1)
    def clear_django_session(num):
        print("it's 2:40 in the morning: clearing django sessions")
        Session.objects.all().delete()

    @spool
    def encode_video(arguments):
        os.system("ffmpeg -i \"%s\" image%%d.jpg" % arguments['filename'])

这个会话清理器在每天的2:40会执行一次，要排队一个视频编码器，我们只需简单地在某个地方spool一下。

.. code-block:: py

    from task import encode_video

    def index(request):
        # launching video encoding
        encode_video.spool(filename=request.POST['video_filename'])
        return render_to_response('enqueued.html')

现在，启用spooler，运行uWSGI：

.. code-block:: ini

    [uwsgi]
    ; a couple of placeholder
    django_projects_dir = /var/www/apps
    my_project = foobar
    ; chdir to app project dir and set pythonpath
    chdir = %(django_projects_dir)/%(my_project)
    pythonpath = %(django_projects_dir)
    ; load django
    module = django.core.handlers:WSGIHandler()
    env = DJANGO_SETTINGS_MODULE=%(my_project).settings
    ; enable master
    master = true
    ; 4 processes should be enough
    processes = 4
    ; enable the spooler (the mytasks dir must exist!)
    spooler = %(chdir)/mytasks
    ; load the task.py module
    import = task
    ; bind on a tcp socket
    socket = 127.0.0.1:3031

唯一一个特别重要的选项是 ``import`` 这个。它的工作方式与 ``module`` 相同，但跳过了WSGI的可调用搜索。你可以用它在加载WSGI应用之前预加载模块。你可以指定无限数目的 '''import''' 指令。

例子：web2py + spooler + timer
---------------------------------

首先，定义你的spooler和timer函数 (我们将称其为:file:``mytasks.py``)

.. code-block:: py
    
    from uwsgidecorators import *
    
    @spool
    def a_long_task(args):
        print(args)
        
    @spool
    def a_longer_task(args)
        print("longer.....")
    
    @timer(3)
    def three_seconds(signum):
        print("3 seconds elapsed")
    
    @timer(10, target='spooler')
    def ten_seconds_in_the_spooler(signum):
        print("10 seconds elapsed in the spooler")

现在，运行web2py。

.. code-block:: sh

    uwsgi --socket :3031 --spooler myspool --master --processes 4 --import mytasks --module web2py.wsgihandler

一旦加载了应用，你就会在日志中看到两个运行着的timer。

现在，我们想要从我们的web2py控制器排队任务。

编辑它们其中一个，然后添加

.. code-block:: py

    import mytasks # be sure mytasks is importable!

    def index(): # this is a web2py action
        mytasks.a_long_task.spool(foo='bar')
        return "Task enqueued"


uwsgidecorators API参考
-----------------------------

.. default-domain:: py

.. module:: uwsgidecorators

.. function:: postfork(func)

   uWSGI是一个预启动 (或者说是"尽情使用fork")的服务器，因此，你可能需要在每次 ``fork()`` 之后执行一个修正任务。这就是 ``postfork`` 装饰器的用武之处。你可以声明多个 ``postfork`` 任务。每个被装饰器装饰的函数将在每个 ``fork()`` 之后依次执行。

   .. code-block:: py

      @postfork
      def reconnect_to_db():
          myfoodb.connect()
      
      @postfork
      def hello_world():
          print("Hello World")

.. function:: spool(func)

   uWSGI的 :doc:`spooler<Spooler>` 是非常有用的。与Celery或其他队列相比，它非常“原始”。 ``spool`` 装饰器会帮到你！

   .. code-block:: py

      @spool
      def a_long_long_task(arguments):
          print(arguments)
          for i in xrange(0, 10000000):
              time.sleep(0.1)
      
      @spool
      def a_longer_task(args):
          print(args)
          for i in xrange(0, 10000000):
              time.sleep(0.5)
      
      # enqueue the tasks
      a_long_long_task.spool(foo='bar',hello='world')
      a_longer_task.spool({'pippo':'pluto'})

   上面的函数将会自动返回 ``uwsgi.SPOOL_OK`` ，因此，根据其返回状态，它们将独立执行一次。

.. XXX: What does the above mean?

.. function:: spoolforever(func)

   当你想要持续的执行一个spool任务时，使用 ``spoolforever`` 。一个 ``@spoolforever`` 任务将总是返回 ``uwsgi.SPOOL_RETRY`` 。

   .. code-block:: py

     @spoolforever
     def a_longer_task(args):
         print(args)
         for i in xrange(0, 10000000):
             time.sleep(0.5)
     
     # enqueue the task
     a_longer_task.spool({'pippo':'pluto'})



.. function:: spoolraw(func)

  高级用户也许想要控制一个任务的返回值。


   .. code-block:: py
      
      @spoolraw
      def a_controlled_task(args):
          if args['foo'] == 'bar':
              return uwsgi.SPOOL_OK
          return uwsgi.SPOOL_RETRY
      
      a_controlled_task.spool(foo='bar')

.. function:: rpc("name", func)

   uWSGI的 :doc:`RPC` 是远程调用uWSGI实例中托管的应用中的函数最快的方式。使用@rpc装饰器，你可以容易地定义导出函数。

   .. code-block:: py
      
      @rpc('helloworld')
      def ciao_mondo_function():
          return "Hello World"

.. function:: signal(num)(func)

   你可以轻松地为 :doc:`信号框架<Signals>` 注册信号。

   .. code-block:: py
      
       @signal(17)
       def my_signal(num):
           print("i am signal %d" % num)

.. function:: timer(interval, func)

   定期执行一个函数。

   .. code-block:: py
      
      @timer(3)
      def three_seconds(num):
          print("3 seconds elapsed")

.. function:: rbtimer(interval, func)

   像@timer，但是使用红黑定时器。

.. XXX: What the hell does _that_ mean? 我也想知道这是神马？

.. function:: cron(min, hour, day, mon, wday, func)

      
   为 :doc:`CronInterface` 轻松注册函数。

   .. code-block:: py

      @cron(59, 3, -1, -1, -1)
      def execute_me_at_three_and_fiftynine(num):
          print("it's 3:59 in the morning")

   从1.2起，支持一种新的语法来模拟类 ``crontab`` 间隔 (每个第N分钟，等等。)。在uWSGI中，可以像这样指定 ``*/5 * * * *`` ：

   .. code-block:: py

      @cron(-5, -1, -1, -1, -1)
      def execute_me_every_five_min(num):
          print("5 minutes, what a long time!")

.. function:: filemon(path, func)

   每次一个文件/目录被修改的适合，执行一个函数。

   .. code-block:: py

        @filemon("/tmp")
        def tmp_has_been_modified(num):
            print("/tmp directory has been modified. Great magic is afoot")

.. function:: erlang(process_name, func)

   将一个函数映射为一个 :doc:`Erlang<Erlang>` 进程。

   .. code-block:: py

        @erlang('foobar')
        def hello():
            return "Hello"


.. function:: thread(func)

    标记函数在一个单独的线程中执行。

    .. 重要:: 必须在uWSGI中使用 ``enable-threads`` 或者 ``threads <n>`` 选项来启用线程。

    .. code-block:: py

        @thread
        def a_running_thread():
            while True:
                time.sleep(2)
                print("i am a no-args thread")

        @thread
        def a_running_thread_with_args(who):
            while True:
                time.sleep(2)
                print("Hello %s (from arged-thread)" % who)

        a_running_thread()
        a_running_thread_with_args("uWSGI")

    你也可以将 ``@thread`` 和 ``@postfork`` 结合在一起，从而在一个新生成的worker中的一个新线程里生成postfork处理器。

    .. code-block:: py

        @postfork
        @thread
        def a_post_fork_thread():
            while True:
                time.sleep(3)
                print("Hello from a thread in worker %d" % uwsgi.worker_id())

.. function:: lock(func)

    这个装饰器将会在一个完全锁定的环境中执行一个函数，从而阻止其他worker或者线程(或者是master，如果你够蠢或者够勇敢的话)同时运行它。显然，这也可以跟@postfork组合在一起。

    .. code-block:: py

        @lock
        def dangerous_op():
            print("Concurrency is for fools!")


.. function:: mulefunc([mulespec], func)

    卸载函数的执行到 :doc:`mule<Mules>` .当卸载函数被调用，它将会立即返回，而执行将会被委托给一个mule。

    .. code-block:: py

        @mulefunc
        def i_am_an_offloaded_function(argument1, argument2):
            print argument1,argument2

    你也可以指定一个mule ID或者mule farm来运行该函数。请务必记住用一个uwsgi import配置选项来注册你的函数。

    .. code-block:: py
    
        @mulefunc(3)
        def on_three():
            print "I'm running on mule 3."

        @mulefunc('old_mcdonalds_farm')
        def on_mcd():
            print "I'm running on a mule on Old McDonalds' farm."

.. function:: harakiri(time, func)

    从uWSGI 1.3-dev开始，添加了一个可定制的二次 :term:`harakiri` 子系统。如果一个给定的调用执行时间太长，那么你可以使用这个装饰器去灭掉一个worker。

    .. code-block:: py

        @harakiri(10)
        def slow_function(foo, bar):
            for i in range(0, 10000):
                for y in range(0, 10000):
                    pass

        # or the alternative lower level api

        uwsgi.set_user_harakiri(30) # you have 30 seconds. fight!
        slow_func()
        uwsgi.set_user_harakiri(0) # clear the timer, all is well
