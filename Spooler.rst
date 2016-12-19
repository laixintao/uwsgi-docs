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
* 'priority' => 这将是spooler目录中的子目录，任务将会被放置在其中，你可以使用哪个技巧来赋予任务足够好的优先权 (更好的方法是使用多个spooler)
* 'body' => 为大于64k的对象使用这个键，这个blob将会被附加到序列化的uwsgi包上，然后作为'body'参数传回给spooler函数

.. note::

   Spool arguments must be strings (or bytes for python3). The API functions will try to cast non-string values to strings/bytes, but do not rely on that functionality!

外部spooler
-----------------

你可能想要为你的服务器实现一个跨多个uWSGI实例的集中式spooler。

单个实例将会管理由多个uWSGI实例入队的所有任务。

要完成这个配置，每个uWSGI实例必须知道哪个spooler目录是有效的 (将其当成一种形式的安全来考虑)。

要添加一个外部spooler目录，使用 ``--spooler-external <directory>`` 选项，然后使用spool函数来添加。

spooler锁子系统将会避免你认为可能会出现的任何混乱。

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



网络spooler
------------------

你甚至可以通过网络入队任务 (确保在你的实例中加载了'spooler'插件，但是一般来说，是默认内置的)。

正如我们已经看到的那样，spooler包使用modifier1 17，你可以直接发送那些包到一个启用了spooler的实例的uWSGI socket上。

在这个例子中，我们会使用Perl的 ``Net::uwsgi`` 模块 (公开了一个方便的uwsgi_spool函数) (但随意使用任何你想要的模块来写spool文件)。

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
   
(感谢brianhorakh提供这个例子)

优先级
----------

我们已经看到，你可以使用'priority'键来赋予spooler解析次序。

虽然使用多个spooler也许是一个更好的方法，但是在一个资源不多的系统上，‘优先权’是个好技巧。

只有你启动了 ``--spooler-ordered`` 选项，它们才能用。这个选项允许spooler以字母序扫描目录项。

如果在扫描期间，发现了一个具有‘数字’名的目录，那么扫描就会被挂起，然后将会探索这个子目录的内容以查找任务。

.. code-block:: sh

   /spool
   /spool/ztask
   /spool/xtask
   /spool/1/task1
   /spool/1/task0
   /spool/2/foo
   
使用这个布局，文件解析的次序将是：

.. code-block:: sh

   /spool/1/task0
   /spool/1/task1
   /spool/2/foo
   /spool/xtask
   /spool/ztask
   
记住，优先级只对命名为“数字”的子目录有用，并且你需要 ``--spooler-ordered`` 选项。

uWSGI spooler为任务赋予了特殊的名字，因此，入队的次序总是会被遵循的。

选项
-------
``spooler=directory``
在指定的目录上运行一个spooler

``spooler-external=directory``
映射spooler请求到一个由外部实例管理的spooler目录

``spooler-ordered``
试着排序spooler任务的执行 (使用scandir来取代readdir)

``spooler-chdir=directory``
在每个spooler任务之前，调用chdir()到指定的目录

``spooler-processes=##``
为spooler设置进程数

``spooler-quiet``
不要打印spooler任务的冗余信息

``spooler-max-tasks=##``
设置循环利用一个spooler之前运行的最大任务数 (以帮助减轻内存泄漏)

``spooler-signal-as-task``
与 ``spooler-max-tasks`` 组合使用。启用这个，spooler将会把信号事件当成任务。运行信号处理器也将会增加spooler任务数。

``spooler-harakiri=##``
为spooler任务设置harakiri超时时间，见[harakiri]以获取更多信息。

``spooler-frequency=##``
设置spooler频率

``spooler-python-import=???``
直接在spooler中导入一个python模块

技巧和窍门
---------------

你可以通过在你的可回调对象中返回 ``uwsgi.SPOOL_RETRY`` 来重新入队一个spooler请求：

.. code-block:: py

    def call_me_again_and_again(env):
        return uwsgi.SPOOL_RETRY
    
你可以使用 ``--spooler-frequency <secs>`` 选项来设置spooler poll频率 (默认是30秒)。

你可以使用 :doc:`Caching` 或者 :doc:`SharedArea` 来在spooler和worker之间交换内存结构。

Python (uwsgidecorators.py)和Ruby (uwsgidsl.rb)公开了高层次的功能来管理spooler，试着使用它们来取代这里描述的低层次方法。

当把一个spooler当成uWSGI信号处理器的目标使用的时候，你可以使用绝对目录名来指定路由信号到哪个。
