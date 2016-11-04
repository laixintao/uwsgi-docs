钩子(hook)
===========

(更新至uWSGI 1.9.16)

uWSGI主要的指导思想是变得“模块化”。它大量的特性作为插件暴露出来，既允许用户优化其构建，又鼓励开发者对其进行扩展。

写插件可能是一件烦人的任务，特别是当你只需要改变/实现一个单一的功能的时候。

对于简单的任务，uWSGI公开了一个钩子API，你可以任性地使用它来修改uWSGI的内部行为。

"可用钩子"的uWSGI阶段
^^^^^^^^^^^^^^^^^^^^^^^^^^^

在准备管理请求之前，uWSGI经历各种“阶段”。你可以将一个或多个“钩子”附加到这些阶段。

每个阶段都可能是“致命的”，如果是这样，那么一个失败的钩子将意味着整个uWSGI实例的失败 (通常调用 ``exit(1)``)。

目前 (2013年9月)，有以下阶段可供选择：

* ``asap`` 在解析完配置文件之后，完成一切事情之前直接运行。它是致命的。
* ``pre-jail`` 在对放弃特权的任何尝试，或者将进程放到任何形式的jail之前运行。它是致命的。
* ``post-jail`` 在任何jail之后立即运行，但会在删除权限之前运行。如果jail需要fork()，那么由父进程运行这个阶段。这是致命的。
* ``in-jail`` 在jail之后立即运行，但在post-jail之后运行。如果jail需要fork()，那么由子进程运行这个阶段。这是致命的。
* ``as-root`` 在删除权限之前立即运行 (作为root运行的最后机会)。 这是致命的。
* ``as-user`` 在删除权限之后立即运行。这是致命的。
* ``pre-app`` 在应用加载之前立即运行。这是致命的。
* ``post-app`` 在应用加载之后立即运行。这是致命的。
* ``accepting`` 在每个worker开始接收请求之前运行 (从uWSGI 1.9.21开始可用)。
* ``accepting1`` 在第一个worker开始接收请求之前运行 (从uWSGI 1.9.21开始可用)。
* ``accepting-once`` 在每个worker开始接收请求之前运行 (从uWSGI 1.9.21开始可用，每个实例运行一次)。
* ``accepting1-once`` 在第一个worker开始接收请求之前运行 (从uWSGI 1.9.21开始可用，每个实例运行一次)。
* ``as-user-atexit`` 在实例开始关闭之前运行。这不是致命的。
* ``as-emperor`` 在Emperor进程中生成vassal之后立即运行。这不是致命的。
* ``as-vassal`` 执行uwsgi二进制文件之前在vassal中运行。这是致命的。

"硬编码"钩子
^^^^^^^^^^^^^^^^^^^^^

如前所述，钩子子系统的目的在于允许附加“钩子”到不同的uWSGI阶段。

有两种钩子。简单的那种是所谓的"硬编码"钩子。它们以多样性为代价，公开了常见的模式。

目前 (2013年九月)，有以下可用的“硬编码”钩子 (它们按照它们在下面显示的顺序运行):


``mount`` —— 文件系统挂载
******************************

参数：<filesystem> <src> <mountpoint> [flags]

公开的标志是可用于操作系统的那些。例如，在Linux上，你会选择bind, recursive, readonly等等。

``umount`` —— 文件系统解除挂载
*********************************

参数：<mountpoint> [flags]

``exec`` 运行shell命令
***************************

参数：<command> [args...]

通过 ``/bin/sh`` 运行命令。

如果出于某些原因，你不想要使用 ``/bin/sh`` 作为运行的shell，那么你可以使用 ``--binsh`` 选项来覆盖它。你可以指定多个 ``--binsh`` 选项 —— 将会尝试使用它们，直到找到一个有效的shell。

``call`` 在当前进程地址空间内调用函数
************************************************************

参数：<symbol> [args...]

一般来说，会忽略参数 (唯一例外是emperor/vassal阶段，见下)，因为系统期望不带参数调用符号。

``<symbol>`` 可以是当前在进程的地址空间中任何可用的符号。

当与 ``--dlopen`` uWSGI选项结合使用的时候，可以实现一些有趣的技巧：

.. code-block:: c

   // foo.c
   #include <stdio.h>
   void foo_hello() {
           printf("I am the foo_hello function called by a hook!\n");
   }
   
将其作为共享库进行构建：

.. code-block:: sh

   gcc -o foo.so -shared -fPIC foo.c
   
然后将其加载到uWSGI符号表里。

.. code-block:: sh

   uwsgi --dlopen ./foo.so ...
   
从现在开始，"foo_hello"符号就存在uWSGI符号表里了，它准备好呗“call”钩子调用了。

.. warning::

   由于--dlopen是 ``dlopen()`` 函数的一个封装，因此谨防使用绝对路径和库搜索路径。如果你不想给自己找麻烦，那么在处理共享库的时候，总是使用绝对路径。

附加“硬编码”钩子
^^^^^^^^^^^^^^^^^^^^^^^^^^^

每个硬编码钩子都为每个阶段公开了一组选项 (有一些例外)。

每个选项都是由钩子的名字和它的阶段组成的，因此要在 ``as-root`` 阶段运行一个命令，使用 ``--exec-as-root``, 或者为 ``as-user`` 阶段使用 ``--exec-as-user`` 。

记住，你可以将所有你需要的钩子附加到一个钩子-阶段对。

.. code-block:: ini

   [uwsgi]
   ...
   exec-as-root = cat /proc/cpuinfo
   exec-as-root = echo 1 > /proc/sys/net/ipv4/ip_forward
   
   exec-as-user = ls /tmp
   exec-as-user-at-exit = rm /tmp/foobar
   
   dlopen = ./foo.so
   call-as-user = foo_hello
   ...
   
这个规则的唯一例外是 `as-emperor` 和 `as-vassal` 阶段。由于种种原因，它们公开了一堆方便的变体 —— 见下。

“高级”钩子
^^^^^^^^^^^^^^^^^^^^

一个限制硬编码钩子的多样性（uWSGI中的一个大大的禁忌）的问题是，你不能控制整个链的顺序 (每个阶段执行按类型分组的每个钩子)。如果你想获得更多的控制权，那么“高级”钩子是最好的选择。

每个阶段都有一个单一的链，你在其中指定调用的钩子和使用哪个处理器。

处理器指定了如何运行钩子。可以通过插件注册新的处理器。

目前，核心公开的处理器是：

* ``exec`` - 与'exec'硬编码选项相同
* ``call`` - 调用指定的符号，忽略返回值
* ``callret`` - 调用指定的符号，期望一个int类型的返回值。任何!= 0的返回都表示失败
* ``callint`` - 调用指定的符号，将参数当成一个int来解析
* ``callintret`` - 调用指定的符号，将参数当成一个int来解析，并期待返回一个int。
* ``mount`` - 与'mount'硬编码选项相同
* ``umount`` - 与'umount'硬编码选项相同
* ``cd`` - 便捷处理器，与 ``call:chdir <directory>`` 相同
* ``exit`` - 便捷处理器，与 ``callint:exit [num]`` 相同
* ``print`` - 便捷处理器，与调用 ``uwsgi_log`` 符号相同
* ``write`` - (从uWSGI 1.9.21起)，使用write:<file> <string>将一个字符串写入到指定文件中。
* ``writefifo`` - (从uWSGI 1.9.21起)，使用writefifo:<file> <string>将一个字符串写入到指定FIFO中。
* ``unlink`` - (从uWSGI 1.9.21起), 取消链接指定的文件

.. code-block:: ini

   [uwsgi]
   ...
   hook-as-root = mount:proc none /proc
   hook-as-root = exec:cat /proc/self/mounts
   hook-pre-app = callint:putenv PATH=bin:$(PATH)
   hook-post-app = call:uwsgi_log application has been loaded
   hook-as-user-atexit = print:goodbye cruel world
   ...
