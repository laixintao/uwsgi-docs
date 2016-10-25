uWSGI Spooler
=================

更新至uWSGI 2.0.1

支持语言：Perl, Python, Ruby

Spooler是内置于uWSGI的队列管理器，它的工作方式像打印/邮件系统。

你可以排队大量的邮件发送、图像处理、视频编码等等，并且让spooler在后台为你努力工作，同时用户的请求会被正常的worker处理。

spooler通过定义"spooler文件"将会写入的目录来工作，每次spooler在它的目录下找到一个文件，它就会解析它，然后运行一个特定的函数。

你可以让多个spooler映射到不同的目录，甚至可以让多个spooler映射到相同的目录。

``--spooler <directory>``选项允许你生成一个spooler进程，而``--spooler-processes <n>``允许你设置为每个spooler生成多少个进程。

spooler也能够管理uWSGI信号量，因此，你可以把它当成你的处理器的目标使用。

这个配置将为你的实例生成一个spooler (myspool目录必须存在)

.. code-block:: ini

   [uwsgi]
   spooler = myspool
   ...
   
而这个将创建两个spooler：

.. code-block:: ini

   [uwsgi]
   spooler = myspool
   spooler = myspool2
   ...

拥有多个spooler使你能够把任务区分优先次序（甚至对其并行处理）

spool文件
-----------

spool文件是序列化的字符串哈希/字典。spooler将对其进行解析，然后将得到的哈希/字典传递给spooler函数（见下文）。

序列化格式与'uwsgi'协议使用的格式相同，因此，最多只能64k (即使有窍门传递更大的值，见下面的'body'魔法键)。用于spooler包的modifier1是17, 因此，一个{'hello' => 'world'}哈希将会被编码成：

========= ============== ==============
header    key1           value1
========= ============== ==============
17|14|0|0 |5|0|h|e|l|l|o |5|0|w|o|r|l|d
========= ============== ==============

一个锁定系统允许你在出现问题的时候安全地手工移除spool文件，或者在两个spooler目录之间移动。

允许跨NFS的spool目录，但是如果你在适当的位置上没有合适的NFS锁，那么请避免映射相同的spooler NFS目录到不同机器上的spooler。

设置spooler函数/调用
-------------------------------------

由于有几十种不同的方式来排队spooler请求，因此我们将首先涵盖接收请求。

想要有个全面运作的spooler，那么你需要定义一个"spooler函数/调用"来处理请求。

无论配置的spooler数目是多少，都会执行相同的函数。由开发者指示它识别任务。如果你不处理请求，那么spool目录就会只是填满。

这个函数必须返回一个整数值：

* -2 (SPOOL_OK) —— 任务已完成，将会移除spool文件
* -1 (SPOOL_RETRY) —— 暂时错误，在下一个spooler迭代将会重试该任务。
* 0 (SPOOL_IGNORE) —— 忽略此任务，如果实例中加载了多个语言，那么它们所有都会竞争管理该任务。这个返回值允许你跳过特定的语言的任务。

任何其他值都会被解析成-1 (重试)。

每个语言插件都有它自己定义spooler函数的方法：

Perl:

.. code-block:: pl

   uwsgi::spooler(
       sub {
           my ($env) = @_;
           print $env->{foobar};
           return uwsgi::SPOOL_OK;
       }
   );
   # hint - uwsgi:: is available when running using perl-exec= or psgi= 
   # no don't need to use "use" or "require" it, it's already there.
   
Python:

.. code-block:: py

   import uwsgi
   
   def my_spooler(env):
       print env['foobar']
       return uwsgi.SPOOL_OK
       
   uwsgi.spooler = my_spooler
    
Ruby:

.. code-block:: rb

   module UWSGI
        module_function
        def spooler(env)
                puts env.inspect
                return UWSGI::SPOOL_OK
        end
   end


spooler函数必须在master进程中定义，因此，如果是在lazy-apps模式下，那么确保将其放到一个文件中，该文件要在服务器设置之初被解析。(在Python中，你可以使用--shared-import，在Ruby中，使用--shared-require，在Perl中，使用--perl-exec)。

Python支持使用 ``--spooler-python-import`` 选项，直接在spooler中导入代码。

排队请求到一个spooler
--------------------------------

'spool' api函数允许你排队一个哈希/目录到实例指定的spooler：

.. code-block:: ini

   # add this to your instance .ini file
   spooler=/path/to/spooler
   # that's it! now use one of the code blocks below to send requests
   # note: you'll still need to register some sort of receiving function (specified above)

.. code-block:: py

   # python
   import uwsgi
   uwsgi.spool({'foo': 'bar', 'name': 'Kratos', 'surname': 'the same of Zeus'})
   # or
   uwsgi.spool(foo='bar', name='Kratos', surname='the same of Zeus')
   # for python3 use bytes instead of strings !!!


.. code-block:: pl

   # perl 
   uwsgi::spool({foo => 'bar', name => 'Kratos', surname => 'the same of Zeus'})
   # the uwsgi:: functions are available when executed within psgi or perl-exec

.. code-block:: rb

   # ruby
   UWSGI.spool(foo => 'bar', name => 'Kratos', surname => 'the same of Zeus')
   
一些键有特殊含义：

* 'spooler' => 指定必须管理这个任务的spooler的绝对路径
* 'at' => 必须执行该任务的unix时间 (读：该任务将不会运行，直到过去'at'时间)
* 'priority' => this will be the subdirectory in the spooler directory in which the task will be placed, you can use that trick to give a good-enough prioritization to tasks (for better approach use multiple spoolers)
* 'body' => use this key for objects bigger than 64k, the blob will be appended to the serialzed uwsgi packet and passed back to the spooler function as the 'body' argument

.. note::

   Spool arguments must be strings (or bytes for python3). The API functions will try to cast non-string values to strings/bytes, but do not rely on that functionality!

外部spooler
-----------------

You could want to implement a centralized spooler for your server across many uWSGI instances.

A single instance will manage all of the tasks enqueued by multiple uWSGI instances.

To accomplish this setup, each uWSGI instance has to know which spooler directories are valid (consider it a form of security).

To add an external spooler directory use the ``--spooler-external <directory>`` option, then add to it using the spool function.

The spooler locking subsystem will avoid any messes that you might think could occur.

.. code-block:: ini

   [uwsgi]
   spooler-external = /var/spool/uwsgi/external
   ...

.. code-block:: py

   # python
   import uwsgi
   uwsgi.spool({'foo': 'bar',  'spooler': '/var/spool/uwsgi/external'})
   # or
   uwsgi.spool(foo='bar', spooler='/var/spool/uwsgi/external')
   # for python3 use bytes instead of strings !!!



Networked spoolers
------------------

You can even enqueue tasks over the network (be sure the 'spooler' plugin is loaded in your instance, but generally it is built in by default).

As we have already seen, spooler packets have modifier1 17, you can directly send those packets to an uWSGI socket of an instance with a spooler enabled.

We will use the Perl ``Net::uwsgi`` module (exposing a handy uwsgi_spool function) in this example (but feel free to use whatever you want to write the spool files).

.. code-block:: perl

   #!/usr/bin/perl
   use Net::uwsgi;
   uwsgi_spool('localhost:3031', {'test'=>'test001','argh'=>'boh','foo'=>'bar'});
   uwsgi_spool('/path/to/my.sock', {'test'=>'test001','argh'=>'boh','foo'=>'bar'});
   
.. code-block:: ini

   [uwsgi]
   socket = /path/to/my.sock
   socket = localhost:3031
   spooler = /path/for/files
   spooler-processes=1
   perl-exec = /path/for/script-which-registers-spooler-sub.pl  
   ...
   
(thanks brianhorakh for the example)

优先级
----------

We have already seen that you can use the 'priority' key to give order in spooler parsing.

While having multiple spoolers would be an extremely better approach, on system with few resources 'priorities' are a good trick.

They works only if you enable the ``--spooler-ordered`` option. This option allows the spooler to scan directories entry in alphabetical order.

If during the scan a directory with a 'number' name is found, the scan is suspended and the content of this subdirectory will be explored for tasks.

.. code-block:: sh

   /spool
   /spool/ztask
   /spool/xtask
   /spool/1/task1
   /spool/1/task0
   /spool/2/foo
   
With this layout the order in which files will be parsed is:

.. code-block:: sh

   /spool/1/task0
   /spool/1/task1
   /spool/2/foo
   /spool/xtask
   /spool/ztask
   
Remember, priorities only work for subdirectories named as 'numbers' and you need the ``--spooler-ordered`` option.

The uWSGI spooler gives special names to tasks so the ordering of enqueuing is always respected.

选项
-------
``spooler=directory``
run a spooler on the specified directory

``spooler-external=directory``
map spoolers requests to a spooler directory managed by an external instance

``spooler-ordered``
try to order the execution of spooler tasks (uses scandir instead of readdir)

``spooler-chdir=directory``
call chdir() to specified directory before each spooler task

``spooler-processes=##``
set the number of processes for spoolers

``spooler-quiet``
do not be verbose with spooler tasks

``spooler-max-tasks=##``
set the maximum number of tasks to run before recycling a spooler (to help alleviate memory leaks)

``spooler-signal-as-task``
combined use with ``spooler-max-tasks``. enable this, spooler will treat signal events as task.
run signal handler will also increase the spooler task count.

``spooler-harakiri=##``
set harakiri timeout for spooler tasks, see [harakiri] for more information.

``spooler-frequency=##``
set the spooler frequency

``spooler-python-import=???``
import a python module directly in the spooler

技巧和窍门
---------------

You can re-enqueue a spooler request by returning ``uwsgi.SPOOL_RETRY`` in your callable:

.. code-block:: py

    def call_me_again_and_again(env):
        return uwsgi.SPOOL_RETRY
    
You can set the spooler poll frequency using the ``--spooler-frequency <secs>`` option (default 30 seconds).

You could use the :doc:`Caching` or :doc:`SharedArea` to exchange memory structures between spoolers and workers.

Python (uwsgidecorators.py) and Ruby (uwsgidsl.rb) exposes higher-level facilities to manage the spooler, try to use them instead of the low-level approach described here.

When using a spooler as a target for a uWSGI signal handler you can specify which one to route signal to using its ABSOLUTE directory name.
