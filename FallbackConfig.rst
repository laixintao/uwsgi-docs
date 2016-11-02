后备配置
======================

(从1.9.15-dev起可用)

如果你需要一个"重置到出厂默认设置"，或者"在用户把配置弄得一团糟的时候显示一个欢迎页面"的场景，那么后备配置就是你的银弹。

简单的例子
***********

一个非常普遍的问题是screwing-up the port on which the instance is listening.

为了模拟这样的错误，我们试着以非特权用户来绑定80端口:

.. code-block:: sh

   uwsgi --uid 1000 --http-socket :80
   
uWSGI将会退出:

.. code-block:: sh

   bind(): Permission denied [core/socket.c line 755]
   
Internally (from the kernel point of view) the instance exited with status 1

Now we want to allow the instance to automatically bind on port 8080 when the user supplied config fails.

Let's define a fallback config (you can save it as safe.ini):

.. code-block:: ini

   [uwsgi]
   print = Hello i am the fallback config !!!
   http-socket = :8080
   wsgi-file = welcomeapp.wsgi
   
Now we can re-run the (broken) instance:

.. code-block:: sh

   uwsgi --fallback-config safe.ini --uid 1000 --http-socket :80


Your error will be now something like:

.. code-block:: sh

   bind(): Permission denied [core/socket.c line 755]
   Thu Jul 25 21:55:39 2013 - !!! /home/roberto/uwsgi/uwsgi (pid: 7409) exited with status 1 !!!
   Thu Jul 25 21:55:39 2013 - !!! Fallback config to safe.ini !!!
   [uWSGI] getting INI configuration from safe.ini
   *** Starting uWSGI 1.9.15-dev-a0cb71c (64bit) on [Thu Jul 25 21:55:39 2013] ***
   ...
   
As you can see, the instance has detected the exit code 1 and has binary patched itself with a new config (without changing the pid, or calling fork())


已破坏应用
***********

Another common problem is the inability to load an application, but instead of bringing down the whole site we want to load
an alternate application:

.. code-block:: sh

   uwsgi --fallback-config safe.ini --need-app --http-socket :8080 --wsgi-file brokenapp.py
   
Here the key is --need-app. It will call exit(1) if the instance has not been able to load at least one application.

多种回退层次
************************

Your fallback config file can specify a fallback-config directive too, allowing multiple fallback levels. BEWARE OF LOOPS!!!

它是如何工作的
***************

The objective is catching the exit code of a process before the process itself is destroyed (we do not want to call another fork(), or destroy already opened file descriptors)

uWSGI makes heavy usage of atexit() hooks, so we only need to register the fallback handler as the first one (hooks are executed in reverse order).

In addition to this we need to get the exit code in our atexit() hook, something is not supported by default (the on_exit() function is now deprecated).

The solution is "patching" exit(x) with uwsgi_exit(x) that is a simple wrapper setting uwsgi.last_exit_code memory pointer.

Now the hook only needs to check for uwsgi.last_exit_code == 1 and eventually execve() the binary again passing the fallback config to it

.. code-block:: c

   char *argv[3];
   argv[0] = uwsgi.binary_path;
   argv[1] = uwsgi.fallback_config;
   argv[2] = NULL;
   execvp(uwsgi.binary_path, argv);
   
小抄
*****

Try to place --fallback-config as soon as possible in your config tree. The various config parsers may fail (calling exit(1)) before the fallback file is registered


