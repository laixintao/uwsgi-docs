Ruby API支持
================

状态
------

用于Ruby的uWSGI API仍然未完成 (QueueFramework, SharedArea, 自定义路由和SNMP是最缺乏的一员)。一旦各种API调用准备好了，将会立即扩展DSL。

当前可用的API函数和常量 (可用在UWSGI ruby模块中找到) 是


* UWSGI.suspend
* UWSGI.masterpid
* UWSGI.async_sleep
* UWSGI.wait_fd_read
* UWSGI.wait_fd_write
* UWSGI.async_connect
* UWSGI.signal
* UWSGI.register_signal
* UWSGI.register_rpc
* UWSGI.signal_registered
* UWSGI.signal_wait
* UWSGI.signal_received
* UWSGI.add_cron
* UWSGI.add_timer
* UWSGI.add_rb_timer
* UWSGI.add_file_monitor
* UWSGI.cache_get
* UWSGI.cache_get!
* UWSGI.cache_exists
* UWSGI.cache_exists?
* UWSGI.cache_del
* UWSGI.cache_set
* UWSGI.cache_set
* UWSGI.cache_set!
* UWSGI.cache_update
* UWSGI.cache_update!
* UWSGI.setprocname
* UWSGI.set_warning_message
* UWSGI.lock
* UWSGI.unlock
* UWSGI.mem
* UWSGI.mule_get_msg
* UWSGI.request_id
* UWSGI.mule_id
* UWSGI.mule_msg
* UWSGI.worker_id
* UWSGI.log
* UWSGI.logsize
* UWSGI.i_am_the_spooler
* UWSGI.send_to_spooler
* UWSGI.spool
* UWSGI::OPT
* UWSGI::VERSION
* UWSGI::HOSTNAME
* UWSGI::NUMPROC
* UWSGI::PIDFILE
* UWSGI::SPOOL_OK
* UWSGI::SPOOL_RETRY
* UWSGI::SPOLL_IGNORE

uWSGI DSL
---------

平行于uWSGI API Python装饰器，有一个可用的用于Ruby的DSL，允许优雅访问 uWSGI API。

这个模块作为 :file:`uwsgidsl.rb` ，可以在源代码发行版中找到。你可以将这个代码放到你的 :file:`config.ru` 文件中，或者使用 ``rbrequire`` 选项来自动包含它。

timer(n, block)
---------------

定期执行代码。

.. code-block:: ruby

  timer 30 do |signum|
    puts "30 seconds elapsed"
  end

rbtimer(n, block)
-----------------

就像timer，但是使用一个红黑树计时器。

.. code-block:: ruby

  rbtimer 30 do |signum|
    puts "30 seconds elapsed"
  end

filemon(path, block)
--------------------

在文件修改的时候执行代码。


.. code-block:: ruby

  filemon '/tmp' do |signum|
    puts "/tmp has been modified"
  end

cron(hours, mins, dom, mon, dow, block)
---------------------------------------

使用 :doc:`CronInterface` 周期性执行一个任务

.. code-block:: ruby

  cron 20,16,-1,-1,-1 do |signum|
    puts "It's time for tea."
  end

signal(signum, block)
---------------------

为 :doc:`SignalFramework` 将代码注册为信号处理器。

.. code-block:: ruby

  signal 17 do |signum|
    puts "Signal #{signum} was invoked."
  end

postfork(block)
---------------

每次 ``fork()`` 之后执行代码。

.. code-block:: ruby

  postfork do
    puts "uWSGI server called fork()"
  end

rpc(name, block)
----------------

将代码作为 :doc:`RPC` 函数注册。

.. code-block:: ruby
  
  rpc 'helloworld' do
      return "Hello World"
  end
  
  rpc 'advancedhelloworld' do |x,y|
      return "x = #{x}, y = #{y}"
  end

mule(id?, block)
----------------

将代码作为 :doc:`Mule <Mules>` brain注册。

.. code-block:: ruby
  
  mule 1 do # Run in mule 1
    puts "I am the mule #{UWSGI.mule_id}"
  end

  mule do # Run in first available mule
    puts "I am the mule #{UWSGI.mule_id}"
  end

在函数返回之后，mule将会是无brain的。要避免这个问题，请将这个代码放到一个循环中，或者使用 ``muleloop`` 。

muleloop(id?, block)
--------------------

在一个循环上下文中，在Mule中执行代码。

.. code-block:: ruby
  
  muleloop 3 do
    puts "I am the mule #{UWSGI.mule_id}"
    sleep(2)
  end

SpoolProc
---------

 ``Proc`` 的子类，允许你定义一个在 :doc:`Spooler<Spooler>` 中执行的任务。

.. code-block:: ruby

  # define the function
  my_long_running_task = SpoolProc.new {|args|
    puts "I am a task"
    UWSGI::SPOOL_OK
  }

  # spool it
  my_long_running_task.call({'foo' => 'bar', 'one' => 'two'})

MuleFunc
--------

从任意进程（例如一个worker）中调用一个函数，但是在mule中执行

.. code-block:: ruby

  i_am_a_long_running_function = MuleFunc.new do |pippo, pluto|
    puts "i am mule #{UWSGI.mule_id} #{pippo}, #{pluto}"
  end
  
  i_am_a_long_running_function.call("serena", "alessandro")

这个worker调用 ``i_am_a_long_running_function()`` ，但是函数将会在第一个可用mule中异步执行。

如果你想在一个指定的mule中运行函数，那么添加一个ID参数。以下代码将会只使用mule #5。

.. code-block:: ruby

  i_am_a_long_running_function = MuleFunc.new 5 do |pippo,pluto|
    puts "i am mule #{UWSGI.mule_id} #{pippo}, #{pluto}"
  end

  i_am_a_long_running_function.call("serena", "alessandro")

真实世界的使用
----------------

一个简单的Sinatra应用，每30秒打印消息：

.. code-block:: ruby

  # This is config.ru

  require 'rubygems'
  require 'sinatra'
  require 'uwsgidsl'
  
  timer 30 do |signum|
    puts "30 seconds elapsed"
  end
  
  get '/hi' do
    "Hello World!"
  end
  
  run Sinatra::Application

或者你可以将你的代码放到一个专有的文件中 (这里是 :file:`mytasks.rb`)

.. code-block:: ruby
  
  require 'uwsgidsl'
  
  timer 30 do |signum|
    puts "30 seconds elapsed"
  end
  
  timer 60 do |signum|
    puts "60 seconds elapsed"
  end

然后这样加载它

.. code-block:: sh

  uwsgi --socket :3031 --rack config.ru --rbrequire mytasks.rb --master --processes 4
