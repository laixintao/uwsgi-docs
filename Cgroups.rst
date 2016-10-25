在Linux CGroup中运行uWSGI
===============================

Linux cgroup是在最近的Linux内核中提供的一个惊人的特性。它们允许你在约束环境中，使用有限的CPU, 内存,调度优先级, IO等"jail"你的进程……

.. note:: 要使用cgroup，uWSGI必须作为root运行。 ``uid`` 和 ``gid`` 是非常非常必要的。

启用cgroup
----------------

首先，你需要在你的系统中启用cgroup支持。创建/cgroup目录，然后添加这个到你的/etc/fstab中：

.. code-block:: sh

    none /cgroup cgroup cpu,cpuacct,memory

然后，挂载/cgroup，这样，你就会拥有带有已控制CPU和内存使用的jail了。有一些其他的Cgroup子系统，但是CPU和内存使用时最有用的约束。

让我们在一个cgroup中运行uWSGI：

.. code-block:: sh

    ./uwsgi -M -p 8 --cgroup /cgroup/jail001 -w simple_app -m --http :9090

Cgroup是简单的目录。使用这个命令，你的uWSGI服务器和它的worker就会“限制”在'cgroup/jail001'这个cgroup中。如果你对服务器发起一堆请求，那么，你将会看到cgroup目录中的使用计数器 —— cpuacct.*和memoryfiles.*增长。通过制定一个已经存在的目录，你还可以使用预先存在的cgroup。

一个真实的例子：为你的客户计划QoS
-------------------------------------------------------

假设你为4个客户托管应用。其中两个每个月支付你$100，一个支付$200，而最后一个支付$400。为了拥有一个好的服务质量实现，$100的应用应该获得1/8，或者12.5%的CPU功率，$200的应用应该获得1/4 (25%)，而最后一个应该获得50%。要实现这点，我们必须创建4个cgroup，一个应用一个，然后限制其调度权重。

.. code-block:: sh

    ./uwsgi --uid 1001 --gid 1001 -s /tmp/app1 -w app1 --cgroup /cgroup/app1 --cgroup-opt cpu.shares=125
    ./uwsgi --uid 1002 --gid 1002 -s /tmp/app2 -w app1 --cgroup /cgroup/app2 --cgroup-opt cpu.shares=125
    ./uwsgi --uid 1003 --gid 1003 -s /tmp/app3 -w app1 --cgroup /cgroup/app3 --cgroup-opt cpu.shares=250
    ./uwsgi --uid 1004 --gid 1004 -s /tmp/app4 -w app1 --cgroup /cgroup/app4 --cgroup-opt cpu.shares=500
    
    
 ``cpu.shares``值是简单地根据彼此进行计算的，因此，你可以使用任何你喜欢的方案，例如 (125, 125, 250, 500)或者甚至是(1, 1, 2, 4)。处理了CPU，我们接着限制内存。让我们使用和之前一样的方案，它们中最大是2GB。
    
.. code-block:: sh

    ./uwsgi --uid 1001 --gid 1001 -s /tmp/app1 -w app1 --cgroup /cgroup/app1 --cgroup-opt cpu.shares=125 --cgroup-opt memory.limit_in_bytes=268435456
    ./uwsgi --uid 1002 --gid 1002 -s /tmp/app2 -w app1 --cgroup /cgroup/app2 --cgroup-opt cpu.shares=125 --cgroup-opt memory.limit_in_bytes=268435456
    ./uwsgi --uid 1003 --gid 1003 -s /tmp/app3 -w app1 --cgroup /cgroup/app3 --cgroup-opt cpu.shares=250 --cgroup-opt memory.limit_in_bytes=536870912
    ./uwsgi --uid 1004 --gid 1004 -s /tmp/app4 -w app1 --cgroup /cgroup/app4 --cgroup-opt cpu.shares=500 --cgroup-opt memory.limit_in_bytes=1067459584
