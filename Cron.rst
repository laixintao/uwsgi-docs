uWSGI类cron接口
=============================

uWSGI的 :term:`master` 拥有一个内部类cron的工具，它可以在预设的时间生成事件。你可以这样用它

* 通过uWSGI API，在这种情况下，cron事件将会生成uWSGI信号
* 直接通过选项，在这种情况下，事件将会运行shell命令

基于uWSGI信号
------------------

 :meth:`uwsgi.add_cron` 函数是uWSGI信号cron工具的接口。语法如下

.. code-block:: py

    uwsgi.add_cron(signal, minute, hour, day, month, weekday)

后面5个参数与标准的crontab工作原理相似，但是用-1代替"*"，用-2，-3等代替"*/2", "*/3"等。

.. code-block:: py

    import uwsgi
    
    def five_o_clock_on_the_first_day_of_the_month(signum):
        print "It's 5 o'clock of the first day of the month."
    
    uwsgi.register_signal(99, "", five_o_clock_on_the_first_day_of_the_month)
    uwsgi.add_cron(99, 0, 5, 1, -1, -1)


定时器 VS. cron
---------------

没有设置到指定日期的周期性事件应该使用定时器/红黑定时器。但你对指定日期/小时感兴趣的时候，使用cron。

例如：

.. code-block:: py

    uwsgi.add_cron(99, -1, -1, -1, -1, -1) # ugly, bad and inefficient way to run signal 99 every minute :(
    uwsgi.add_timer(99, 60) # much better.

注意
-----

* ``day`` 和 ``weekday`` 像原始crontab那样进行或运算。
* 默认情况下，你可以为每个master最多定义64个基于信号的cron任务。可以在 :file:`uwsgi.h` 中增加这个值。

基于选项的cron
-----------------

你可以使用 ``cron`` 选项，在配置中直接定义cron任务。可以指定无数个基于选项的cron记录。语法与基于信号的cron相同。

例如：

.. code-block:: ini

    [uwsgi]
    cron = 59 2 -1 -1 -1 /usr/bin/backup_my_home --recursive
    cron = 9 11 -1 -1 2 /opt/dem/bin/send_reminders

.. code-block:: xml

    <uwsgi>
        <cron>59 2 -1 -1 -1 /usr/bin/backup_my_home --recursive</cron>
        <cron>9 11 -1 -1 2 /opt/dem/bin/send_reminders</cron>
    </uwsgi>

.. code-block:: ini

   [uwsgi]
   ; every two hours
   cron = -1 -2 -1 -1 -1 /usr/bin/backup_my_home --recursive

Legion cron
************

当你的实例是 :doc:`Legion` 的一部分时，你可以配置它只在它是指定Legion的Lord时才运行cron。

.. code-block:: ini

   [uwsgi]
   legion = mycluster 225.1.1.1:1717 100 bf-cbc:hello
   legion-node = mycluster 225.1.1.1:1717
   ; every two hours
   legion-cron = mycluster -1 -2 -1 -1 -1 /usr/bin/backup_my_home --recursive

Unique crons
************

.. note:: 这个特性自1.9.11起可用。


一些命令可能需要较长的时间来完成，或者只是挂在那里执行它们的任务。有时，这没问题，但还是有一些情况，运行相同命令的多个实例可能是危险的。

对于这种情况，增加了 ``unique-cron`` 和 ``unique-legion-cron`` 选项。语法与 ``cron`` 和 ``legion-cron`` 相同，但是不同在于，uWSGI将会跟踪执行状态，并且在它完成之前，并不会再次执行cronjob。

例子：

.. code-block:: ini

   [uwsgi]
   cron = -1 -1 -1 -1 -1 sleep 70

这会在每分钟执行 ``sleep 70`` ，但是sleep命令将会比我们的执行时间间隔还要长，最后，将会得到一个数目增长的sleep进程。要修复这个问题，我们可以简单的用 ``unique-cron`` 替换 ``cron`` ，uWSGI会确保只有一个sleep进程在运行。一个新的进程会在前一个结束的时候立即启动。

Harakiri
********

.. note:: 自1.9.11起可用。

``--cron-harakiri`` 将会强制命令执行的时间。如果有命令花费更长的时间，那么将会杀死这个命令。

.. code-block:: ini

   [uwsgi]

   cron = sleep 30
   cron-harakiri = 10

这会在10秒后杀死cron命令。注意， ``cron-harakiri`` 是全局限制，它影响的是所有的cron命令。要设置一个用单一命令上的限制，使用 ``cron2`` 选项 (见下)。

cron选项的新语法
***************************

.. note:: 自1.9.11起可用

为了对cron更好的控制，一个新的选项被添加到uWSGI：

.. code-block:: ini

   [uwsgi]
   cron2 = option1=value,option2=value command to execute

例子：

.. code-block:: ini

   [uwsgi]

   cron2 = minute=-2,unique=1 sleep 130

将会每2分钟生成一个唯一的cron命令 ``sleep 130`` 。

选项列表是可选的，每个cron可用的选项如下：

* ``minute`` - crontab的分钟部分，默认是-1 (解析为*)
* ``hour`` - crontab的小时部分，默认是-1 (解析为*)
* ``day`` - crontab的天部分，默认是-1 (解析为*)
* ``month`` - crontab的月部分，默认是-1 (解析为*)
* ``week`` - crontab的周部分，默认是-1 (解析为*)
* ``unique`` - 将cron命令标记为唯一 (见上)，默认是0 (不唯一)
* ``harakiri`` - 为该cron命令设置harakiri超时时间 (以秒为单位)，默认是0 (不使用harakiri)
* ``legion`` - 为这个cron命令设置使用的legion名，cron legion只有在legion lord节点上才会执行。

注意，你不能在选项列表中使用空格。(``minute=1, hour=2`` 将不会工作，但是 ``minute=1,hour=2`` 则是可以的。)
如果缺失了任何选项，那么就会使用默认值。

.. code-block:: ini

   [uwsgi]
   # execute ``my command`` every minute (-1 -1 -1 -1 -1 crontab).
   cron2 = my command
   # execute unique command ``/usr/local/bin/backup.sh`` at 5:30 every day.
   cron2 = minute=30,hour=5,unique=1 /usr/local/bin/backup.sh


.. code-block:: ini

   [uwsgi]
   legion = mycluster 225.1.1.1:1717 100 bf-cbc:hello
   legion-node = mycluster 225.1.1.1:1717
   cron2 = minute=-10,legion=mycluster my command

这会对 ``my command`` 禁用harakiri，但是其他的cron命令仍然会在10秒后被杀死：

.. code-block:: ini

   [uwsgi]
   cron-harakiri = 10
   cron2 = harakiri=0 my command
   cron2 = my second command


