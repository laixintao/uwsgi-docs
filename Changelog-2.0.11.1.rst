uWSGI 2.0.11.1
==============

[20150719]

错误修复
********

* 修复HTTPS路由器资源释放和文件描述符泄漏
* 当errno为0的时候，不吐出ssl错误

新特性
************

unix_signal钩子
^^^^^^^^^^^^^^^^^^^^

现在，你可以重新映射UNIX信号到指定的函数符号：

.. code-block:: c

   #include <stdio.h>

   void hello_world(int signum) {
           printf("Hello World\n");
   }
   
.. code-block:: sh

   gcc -o libhello.so -shared hello.c
   uwsgi --dlopen ./libhello.so --hook-master-start "unix_signal:1 hello_world" ...
   
每当引发信号1 (SIGHUP)时，就会运行函数hello_world

可用性
************

你可以从这里下载uWSGI 2.0.11.1：

http://projects.unbit.it/downloads/uwsgi-2.0.11.1.tar.gz
