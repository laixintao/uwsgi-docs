钩子(hook)
=====

(更新至uWSGI 1.9.16)

uWSGI主要的指导思想是变得“模块化”。它大量的特性作为插件暴露出来，既允许用户优化其构建，又鼓励开发者对其进行扩展。

写插件可能是一件烦人的任务，特别是当你只需要改变/实现一个单一的功能的时候。

对于简单的任务，uWSGI公开了一个钩子API，你可以任性地使用它来修改uWSGI的内部行为。

The "hookable" uWSGI phases
^^^^^^^^^^^^^^^^^^^^^^^^^^^

在准备管理请求之前，uWSGI经历各种“阶段”。你可以将一个或多个“钩子”附加到这些阶段。

每个阶段都可能是“致命的”，如果是这样，那么一个失败的钩子将意味着整个uWSGI实例的失败 (通常调用 ``exit(1)``)。

目前 (2013年9月)，有以下阶段可供选择：

* ``asap`` 在解析完配置文件之后，完成一切事情之前直接运行。它是致命的。
* ``pre-jail`` 在对放弃特权的任何尝试，或者将进程放到任何形式的jail之前运行。它是致命的。
* ``post-jail`` run soon after any jailing, but before privileges drop. If jailing requires fork(), the parent process run this phase. it is fatal.
* ``in-jail`` run soon after jayling, but after post-jail. If jailing requires fork(), the chidlren run this phase. it is fatal.
* ``as-root`` run soon before privileges drop (last chance to run something as root). it is fatal.
* ``as-user`` run soon after privileges drop. it is fatal.
* ``pre-app`` run before applications loading. it is fatal.
* ``post-app`` run after applications loading. it is fatal.
* ``accepting`` run before the each worker starts accepting requests (available from uWSGI 1.9.21).
* ``accepting1`` run before the first worker starts accepting requests (available from uWSGI 1.9.21).
* ``accepting-once`` run before the each worker starts accepting requests (available from uWSGI 1.9.21, runs one time per instance).
* ``accepting1-once`` run before the first worker starts accepting requests (available from uWSGI 1.9.21, runs one time per instance).
* ``as-user-atexit`` run before shutdown of the instance. it is non-fatal.
* ``as-emperor`` run soon after the spawn of a vassal in the Emperor process. it is non-fatal.
* ``as-vassal`` run in the vassal before executing the uwsgi binary. it is fatal.

"硬编码"钩子
^^^^^^^^^^^^^^^^^^^^^

As said before, the purpose of the hook subsystem is to allow attaching "hooks" to the various uWSGI phases.

There are two kind of hooks. The simple ones are the so-called "hardcoded" ones. They expose common patterns at the cost of versatility.

Currently (September 2013) the following "hardcoded" hooks are available (they run in the order they are shown below):


``mount`` —— 文件系统挂载
******************************

Arguments: <filesystem> <src> <mountpoint> [flags]

The exposed flags are the ones available for the operating system. As an example on Linux you will options like bind, recursive, readonly etc.

``umount`` —— 文件系统解除挂载
*********************************

Arguments: <mountpoint> [flags]

``exec`` 运行shell命令
***************************

Arguments: <command> [args...]

Run the command under ``/bin/sh``.

If for some reason you do not want to use ``/bin/sh`` as the running shell, you can override it with the ``--binsh`` option. You can specify multiple ``--binsh`` options -- they will be tried until one valid shell is found.

``call`` 在当前进程地址空间内调用函数
************************************************************

Arguments: <symbol> [args...]

Generally the arguments are ignored (the only exceptions are the emperor/vassal phases, see below) as the system expects to call the symbol without arguments.

``<symbol>`` can be any symbol currently available in the process's address space.

This allows some interesting tricks when combined with the ``--dlopen`` uWSGI option:

.. code-block:: c

   // foo.c
   #include <stdio.h>
   void foo_hello() {
           printf("I am the foo_hello function called by a hook!\n");
   }
   
Build this as a shared library:

.. code-block:: sh

   gcc -o foo.so -shared -fPIC foo.c
   
and load it into the uWSGI symbol table.

.. code-block:: sh

   uwsgi --dlopen ./foo.so ...
   
From now on, the "foo_hello" symbol is available in the uWSGI symbol table, ready to be called by the 'call' hooks.

.. 警告::

   As --dlopen is a wrapper for the ``dlopen()`` function, beware of absolute paths and library search paths. If you do not want headaches, use always absolute paths when dealing with shared libraries.

附加“硬编码”钩子
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each hardcoded hook exposes a set of options for each phase (with some exceptions).

Each option is composed by the name of the hook and its phase, so to run a command in the ``as-root`` phase you will use ``--exec-as-root``, or ``--exec-as-user`` for the ``as-user`` phase.

Remember, you can attach all of the hooks you need to a hook-phase pair.

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
   
The only exception to the rule are the `as-emperor` and `as-vassal` phases. For various reasons they expose a bunch of handy variants -- see below.

“高级”钩子
^^^^^^^^^^^^^^^^^^^^

A problem that limits their versatility (a big no-no in the uWSGI state of mind) with hardcoded hooks, is that you cannot control the order of the whole chain (as each phase executes each hooks grouped by type). If you want more control, "advanced" hooks are the best choice.

Each phase has a single chain in which you specify the hook the call and which handler.

Handlers specify how to run hooks. New handlers can be registered by plugins.

Currently the handlers exposed by the core are:

* ``exec`` - same as the 'exec' hardcoded options
* ``call`` - call the specified symbol ignoring return value
* ``callret`` - call the specified symbol expecting an int return. anything != 0 means failure
* ``callint`` - call the specified symbol parsing the argument as an int
* ``callintret`` - call the specified symbol parsing the argument as an int and expecting an int return.
* ``mount`` - same as 'mount' hardcoded options
* ``umount`` - same as 'umount' hardcoded options
* ``cd`` - convenience handler, same as ``call:chdir <directory>``
* ``exit`` - convenience handler, same as ``callint:exit [num]``
* ``print`` - convenience handler, same as calling the ``uwsgi_log`` symbol
* ``write`` - (from uWSGI 1.9.21), write a string to the specified file using write:<file> <string>
* ``writefifo`` - (from uWSGI 1.9.21), write a string to the specified FIFO using writefifo:<file> <string>
* ``unlink`` - (from uWSGI 1.9.21), unlink the specified file

.. code-block:: ini

   [uwsgi]
   ...
   hook-as-root = mount:proc none /proc
   hook-as-root = exec:cat /proc/self/mounts
   hook-pre-app = callint:putenv PATH=bin:$(PATH)
   hook-post-app = call:uwsgi_log application has been loaded
   hook-as-user-atexit = print:goodbye cruel world
   ...
