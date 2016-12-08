在uWSGI中使用Linux KSM
========================

`Kernel Samepage Merging <http://www.linux-kvm.org/page/KSM>`_ 是Linux 内核 >= 2.6.32 的一个特性，允许进程共享具有相同内容的内存页。这是由一个内核守护进程周期性执行扫描，对比，以及在可能的情况下，对特定内存区域进行合并来完成的。KVM生而为增强，它可以被用于使用公共数据的进程 (例如，带有语言解释器和标准库的uWSGI进程)。

如果你幸运的话，使用KSM可能会成倍减少你的uWSGI实例的内存使用。特别是在大量的 :doc:`Emperor<Emperor>` 部署中：为每个vassal启用KSM可能会节省大量的内存。uWSGI中的KSM是 `Asidev s.r.l.
<http://www.asidev.com/en/company.html>`_ 的Giacomo Bagnoli的想法。非常感谢他。

启用KSM守护进程
-----------------------

要启用KSM守护进程 (``ksmd``)，只需设置 ``/sys/kernel/mm/ksm/run`` 为1，如下所示：

.. code-block:: sh

    echo 1 > /sys/kernel/mm/ksm/run

.. note::

    记得在机器启动的时候做这个，因为KSM守护进程并不是默认运行的。

.. note::

    KSM是一个可选功能，必须由进程明确要求，因此只是启用KSM将不会拯救你的机器上的一切东西。

在uWSGI中启用KSM支持
-----------------------------

如果你已经在一个内核中编译了带KSM支持的uWSGI了，那么你将能够使用 ``ksm`` 选项。这个选项将会指示uWSGI在每个请求或者master周期后注册经常内存映射 (通过 ``madvise`` 系统调用)。如果自上一次扫描后，并没有页面映射发生改变，那么就不会使用昂贵的系统调用。

性能影响
------------------

检查进程映射要求在每次请求之后解析 ``/proc/self/maps`` 文件。在一些设置中，这将降低性能。你可以通过传递一个参数给 ``ksm`` 选项来调整uWSGI页面扫描的频率。

.. code-block:: sh

    # Scan for process mappings every 10 requests (or 10 master cycles)
    ./uwsgi -s :3031 -M -p 8 -w myapp --ksm=10

检查KSM是否工作良好
----------------------------

``/sys/kernel/mm/ksm/pages_shared`` 和 ``/sys/kernel/mm/ksm/pages_sharing`` 文件包含了关于KSM效率的统计信息。值越高，你的uWSGI实例消耗的内存越少。

使用collectd的KSM统计
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一个像这样的简单的Bash脚本可以用来密切关注KSM的效率：

.. code-block:: sh

    #!/bin/bash
    
    export LC_ALL=C
    
    if [ -e /sys/kernel/mm/ksm/pages_sharing ]; then
        pages_sharing=`cat /sys/kernel/mm/ksm/pages_sharing`;
        page_size=`getconf PAGESIZE`;
        saved=$(echo "scale=0;$pages_sharing * $page_size"|bc);
        echo "PUTVAL <%= cn %>/ksm/gauge-saved interval=60 N:$saved"
    fi

在你的 `collectd <http://collectd.org/>`_ 配置中，添加诸如以下的东西：

.. code-block:: ini

    LoadPlugin exec
    <Plugin exec>
       Exec "nobody" "/usr/local/bin/ksm_stats.sh"
    </Plugin>
