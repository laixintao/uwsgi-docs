回退配置
======================

(从1.9.15-dev起可用)

如果你需要一个"重置到出厂默认设置"，或者"在用户把配置弄得一团糟的时候显示一个欢迎页面"的场景，那么回退配置就是你的银弹。

简单的例子
***********

一个非常普遍的问题是screwing-up the port on which the instance is listening.

为了模拟这样的错误，我们试着以非特权用户来绑定80端口:

.. code-block:: sh

   uwsgi --uid 1000 --http-socket :80
   
uWSGI将会退出:

.. code-block:: sh

   bind(): Permission denied [core/socket.c line 755]
   
在内部 (从内核的角度)，实例退出，状态为1

现在，我们想让实例在用户提供的配置失败的时候自动绑定到端口8080.

让我们来定义一个回退配置 (你可以将其保存为safe.ini):

.. code-block:: ini

   [uwsgi]
   print = Hello i am the fallback config !!!
   http-socket = :8080
   wsgi-file = welcomeapp.wsgi
   
现在，我们可以重新运行（损坏的）实例：

.. code-block:: sh

   uwsgi --fallback-config safe.ini --uid 1000 --http-socket :80


现在，错误将如下：

.. code-block:: sh

   bind(): Permission denied [core/socket.c line 755]
   Thu Jul 25 21:55:39 2013 - !!! /home/roberto/uwsgi/uwsgi (pid: 7409) exited with status 1 !!!
   Thu Jul 25 21:55:39 2013 - !!! Fallback config to safe.ini !!!
   [uWSGI] getting INI configuration from safe.ini
   *** Starting uWSGI 1.9.15-dev-a0cb71c (64bit) on [Thu Jul 25 21:55:39 2013] ***
   ...
   
正如你所见，实例检测到退出码1，并且用一个新的配置对自身打了二进制补丁 (无需更改pid，或者调用fork())


已破坏应用
***********

另一个常见的问题是不能加载应用，但并非让整个站点挂掉，而是想要加载一个备用应用：

.. code-block:: sh

   uwsgi --fallback-config safe.ini --need-app --http-socket :8080 --wsgi-file brokenapp.py
   
这里，键是--need-app。如果实例一直未能加载至少一个应用，那么它将会调用exit(1)。

多种回退层次
************************

你的回退配置文件也可以指定一个fallback-config指令，允许多种回退层次。当心循环！！！

它是如何工作的
***************

目标是在进程本身被销毁之前捕获到其退出码 (不想要调用另一个fork()，或者销毁已打开的文件描述符)

uWSGI大量使用atexit()钩子，因此我们仅需将回退处理器作为第一个进行注册 (钩子是以相反的顺序执行的)。

除此之外，我们需要在我们的atexit()钩子中获取退出码，这个默认不支持 (on_exit()函数现在已弃用)。

解决方法是用uwsgi_exit(x)对exit(x)“打补丁”，它是一个简单的设置uwsgi.last_exit_code内存指针的封装器。

现在，钩子仅需检测uwsgi.last_exit_code == 1，并最终再次execve()二进制文件，将回退配置传递给它

.. code-block:: c

   char *argv[3];
   argv[0] = uwsgi.binary_path;
   argv[1] = uwsgi.fallback_config;
   argv[2] = NULL;
   execvp(uwsgi.binary_path, argv);
   
注意事项
**********

试着尽可能快地将--fallback-config放到你的配置树中。在回退文件被注册之前，各种配置解析器可能会失败 (调用exit(1))


