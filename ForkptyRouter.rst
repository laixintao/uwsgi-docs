Forkpty路由器
==================

处理容器现在是一个常见的部署模式。处理jails/名字空间时的最烦人的任务之一是‘附加’到已经运行的实例。

forkpty路由器旨在简化这个过程，提供一个伪终端服务器给你的uWSGI实例。

一个客户端连接到由forkpty路由器公开的socket，然后获得一个连接到进程的新的伪终端 (一般是一个shell，但是可以是任何你想要的)

uwsgi模式 VS 原始模式
**********************

连接到forkpty路由器的客户端可以用两个协议来进行数据交换：uwsgi和原始模式。

原始模式简单映射socket到pty，出于这样一个原因，你将不能够调整你的终端大小，或者发送特定的信号。这个模式的优点在于性能：每个字符没有开销。

uwsgi模式把每个指令 (stdin, signals, window changes) 封装在一个uwsgi包中。这是非常类似于ssh的工作方式的，所以，如果你计划对shell会话使用forkpty路由器，那么uwsgi模式是最佳选择 (在用户体验方面)。

uwsgi协议的开销 (最差情况) 是每个stdin事件的5个字节 (单个字符)

运行forkpty路由器
**************************

这个插件不是默认内建的，因此你必须编译它：

.. code-block:: sh

	uwsgi --build-plugin plugins/forkptyrouter

或者，使用老的插件构建系统：

.. code-block:: sh

	python uwsgiconfig.py --plugin plugins/forkptyrouter

一般来说，也需要编译pty插件 (为了客户端访问)

.. code-block:: sh

   uwsgi --build-plugin plugins/pty

或者再次，使用老的构建系统：

.. code-block:: sh

	python uwsgiconfig.py --plugin plugins/pty

又或者，你可以一次性构建：

.. code-block:: sh

   UWSGI_EMBED_PLUGINS=pty,forkptyrouter make

现在，你可以将forkptyrouter作为一个标准网关运行 (我们使用UNIX socket，因为我们想要一个与jails的通信信道，并且不共享uts名字空间，以提供一个新的主机名)

.. code-block:: ini

   [uwsgi]
   master = true
   unshare = uts
   exec-as-root = hostname iaminajail
   uid = kratos
   gid = kratos
   forkpty-router = /tmp/fpty.socket

并且用pty客户端连接：

.. code-block:: sh

   uwsgi --pty-connect /tmp/fpty.socket
   

现在，在uWSGI实例中，你有了一个shell (默认是/bin/sh)。运行 ``hostname`` 将会给你'iaminajail'

最终，你可以避免使用uWSGI附加到pty上，取而代之，你可以依赖于这个简单的python脚本：

.. code-block:: py

   import socket
   import sys
   import os
   import select
   import copy
   from termios import *
   import atexit
   
   s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
   s.connect(sys.argv[1])
   
   tcattr = tcgetattr(0)
   orig_tcattr = copy.copy(tcattr)
   atexit.register(tcsetattr, 0, TCSANOW, orig_tcattr)
   
   tcattr[0] |= IGNPAR
   tcattr[0] &= ~(ISTRIP | IMAXBEL | BRKINT | INLCR | IGNCR | ICRNL | IXON | IXANY | IXOFF);
   tcattr[0] &= ~IUCLC;
   tcattr[3] &= ~(ICANON | ECHO | ECHOE | ECHOK | ECHONL);
   tcattr[3] &= ~IEXTEN;
   tcattr[1] &= ~OPOST;
   tcattr[6][VMIN] = 1;
   tcattr[6][VTIME] = 0;
   
   tcsetattr(0, TCSANOW, tcattr);
   
   while True:
       (rl, wl, xl) = select.select([0, s], [], [])
       if s in rl:
           buf = s.recv(4096)
           if not buf: break
           os.write(1, buf)
       if 0 in rl:
           buf = os.read(0, 4096)
           if not buf: break
           s.send(buf)
           



前一个例子使用原始模式，如果调整客户端客户端大小，你将看不到任何更新。

要使用'uwsgi'模式，则添加一个'u'：

.. code-block:: ini

   [uwsgi]
   master = true
   unshare = uts
   exec-as-root = hostname iaminajail
   uid = kratos
   gid = kratos
   forkpty-urouter = /tmp/fpty.socket


.. code-block:: sh

   uwsgi --pty-uconnect /tmp/fpty.socket

单个实例可以在不同的socket上公开两个协议

.. code-block:: ini

   [uwsgi]
   master = true
   unshare = uts
   exec-as-root = hostname iaminajail
   uid = kratos
   gid = kratos
   forkpty-router = /tmp/raw.socket
   forkpty-urouter = /tmp/uwsgi.socket

修改默认命令
****************************

默认情况下，forkpty路由器在新的连接上运行/bin/sh。

你可以使用--forkptyrouter-command修改命令

.. code-block:: ini

   [uwsgi]
   master = true
   unshare = uts
   exec-as-root = hostname iaminajail
   uid = kratos
   gid = kratos
   forkpty-router = /tmp/raw.socket
   forkpty-urouter = /tmp/uwsgi.socket
   forkptyrouter-command= /bin/zsh
