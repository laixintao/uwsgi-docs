Fork服务器 (由Intellisurvey赞助)
============================================

这是一个由Intellisurvey.com赞助开发的真正高级（且复杂）的特性。

如果你在相同的代码库之上构建了数以上百个应用，那么你可以设置你的Emperor来从已经运行的vassal（加载了应用核心部分）那里fork vassal。

目前，只有在PSGI插件中才支持该特性，并且要求Linux kernel >= 3.4。

它如何工作
------------

当处于 `fork-server` 模式时，Emperor区分两种类型的vassal：base vassal和adopted vassal。

"base" vassal是非常经典的vassal；由Emperor的 `fork()` + `execve()` 生成。唯一的区别在于，假定尽可能快的加载尽可能多的应用代码，然后挂起它们，以等待UNIX socket的连接。

一个"base" vassal大概像这样

.. code-block:: ini

   [uwsgi]
   ; load myapp.pl as soon as possible
   early-psgi = myapp.pl
   ; suspend and execution and bind on UNIX socket /tmp/fork_server.socket
   fork-server = /tmp/fork_server.socket
   
"adopted" vassal是真正的“新玩意”。

一旦请求了一个adopted vassal，Emperor就会连接到指定的fork服务器 (而不是自己调用 `fork()` + `execve()` )。

Emperor传递新vassal的命令行选项的uwsgi序列化数组和多达3个文件描述符 (由于UNIX socket允许将文件描述符从一个进程传递给另一个)。

那3个文件描述符是：

* 1 -> 和Emperor的通信管道 (必要)
* 2 -> 配置管道 (可选)
* 3 -> 按需socket (可选)

在这一点上，fork服务器 `fork()` 自身两次，然后使用所提供的参数数组继续uWSGI启动。

Emperor怎会 `wait()` 一个外部进程，然后呢？

这就是为什么要求>= 3.4的内核，因为有了 `prctl(PR_SET_CHILD_SUBREAPER, 1)` 调用，我们可以告诉vassal在它们的父亲死掉的时候，重新归于另一个父亲 (实际上，fork服务器fork两次，因此，vassal并没有活着的父亲，可怜的孩子)。

现在，Emperor有了一个新的孩子和一个通信管道。就这样。

为fork服务器模式配置Emperor
---------------------------------------------

你只需要两个新的选项： ``--emperor-use-fork-serve <addr>`` 和 ``--vassal-fork-base <name>``

让我们从一个缓慢加载的 (10秒) Perl应用开始：

.. code-block:: pl

   # myapp.pl
   print "I am the App\n";
   sleep(10);
   my $app = sub {
        return [200, ['Content-Type'=>'text/html'], ["Hello World"]];
   };

将其保存为myapp.pl，然后将其加载到perlbase.ini vassal文件中 (这是一个base vassal):

.. code-block:: ini

   [uwsgi]
   early-psgi = myapp.pl
   fork-server = /var/run/fork_server.socket

现在，创建两个vassal (one.ini和two.ini)，它们将从base vassal那里 `fork()` ：

.. code-block:: ini

   [uwsgi]
   ; one.ini
   http-socket = :8181
   processes = 4
   uid = 1001
   gid = 1001
   
.. code-block:: ini

   [uwsgi]
   ; two.ini
   http-socket = :8282
   processes = 8
   uid = 1002
   gid = 1002
   
正如你可以见到的那样，它们是相当不同的，即使在特权部分。

现在，在fork服务器模式下生成Emperor，让perlbase.ini作为一个"base" vassal：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-fork-server = /var/run/fork_server.socket
   vassal-fork-base = perlbase.ini
   emperor-stats = 127.0.0.1:5000
   
这个Emperor将开始把perlbase.ini当成一个标准的vassal运行，而对于非base vassal，它会从base vassal `fork()` ，在那里，已经加载了应用。

你会注意到，你的vassal将会立即启动，而不是等待10秒。很酷吧？
