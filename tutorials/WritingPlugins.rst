编写uWSGI插件
=====================

本教程将向你介绍uWSGI hacking。需要一点C知识和UNIX理论。

本教程中使用的简化（安全）的构建系统已添加到uWSGI 1.9.21中，在旧版本中，你需要原始程序（在教程结束时有描述）

uWSGI插件是什么？
************************

uWSGI插件是标准的共享库（带有经典的.so扩展名），公开了一个特定的名为"uwsgi_plugin"的C结构。

这个结构公开了一堆便利的信息（例如插件的名字）和“钩子”。

钩子是简单函数，注册来在特定的服务器阶段运行。

你可以写的最小的插件是像这样的 ('foobar'插件)：

.. code-block:: c

   #include <uwsgi.h>
   
   struct uwsgi_plugin foobar_plugin = {
           .name ="foobar",
   };
   
它将自己声明为'foobar'，并且不公开任何钩子（是哒，它是这最没用的钩子，除了增加了uWSGI小小的内存使用）。

不要求插件定义钩子 —— 它们可以简单地公开可以使用uWSGI高级功能调用的函数 (阅读： :doc:`Hooks`)。

为什么（以及何时）使用插件？
***********************

即使uWSGI可以直接加载共享库 (使用 ``--dlopen``) ，并且将它们的函数称之为钩子，但是有时候你想要与uWSGI内部结构进行交互。

第一个插件
****************

我们第一个插件将是一个简单的"Hello world":

.. code-block:: c

   #include <uwsgi.h>
   
   static int foo_init() {
         uwsgi_log("Hello World\n");
         return 0;
   }
   
   struct uwsgi_plugin foobar_plugin = {
           .name = "foobar",
           .init = foo_init,
   };
   
将其保存为 ``foobar.c`` 。

构建它：

.. code-block:: sh

   uwsgi --build-plugin foobar.c
   
你将会获得一个 ``foobar_plugin.so`` ，你可以将它加载到你的uWSGI二进制中。

.. code-block:: sh

   uwsgi --plugin ./foobar_plugin.so
   
如果一切顺利，那么在uWSGI错误退出（因为没有定义任何socket）之前，你应该会在终端上看到"Hello World"。

uwsgiplugin.py文件
***********************

魔法解密
**************************

正如所看到的那样，uwsgi二进制本身能够在不强迫用户/开发者关心构建配置文件，#ifdef或者平台特定的配置的情况下构建插件。

这是有可能的，因为uwsgi二进制本身包含了原始的'uwsgi.h'文件以及'uwsgiconfig.py'脚本。

除了这个，还存储了构建二进制时使用的CFLAGS。

有了这3个组件，你就可以放心地安全地构建为你的uwsgi二进制调整的uWSGI插件了。

一般的插件 VS 请求插件
**********************************

wsgi_request结构
***********************

header, body和sendfile
**************************

Offloading
**********

可用钩子
***************

定义选项
****************

使用C++
*********

使用Objective-C
*****************

socket I/O
**********

每当你在一个socket上进行I/O操作时，你必须确保不阻塞当前运行的线程/核/worker。

uwsgi API公开了一些函数，用来在处理I/O的时候保证安全性。它们将在这里记录，但目前还木有。

