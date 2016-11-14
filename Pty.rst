Pty插件
==============

-自uWSGI 1.9.15起可用，Linux, OpenBSD, FreeBSD和OSX皆支持。

这个插件允许你附加伪终端到你的应用上。

目前，只能在第一个worker上附加（并通过网络公开）伪终端服务器
(未来会移除该限制)。

该插件还公开了一个客户端模式 (避免你把它跟netcat, telnet或者screen设置混在一起)


构建它
***********

默认构建配置文件中并不包含这个插件，因此，你必须手工构建它：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/pty [profile]
   
(如果你不使用默认的构建配置文件，那么记得指定它)

例子1：Rack应用共享调试
********************************************

.. code-block:: sh

   UWSGI_PROFILE=ruby2 UWSGI_EMBED_PLUGINS=pty make
   
.. code-block:: sh

   ./uwsgi --rbshell="require 'pry';binding.pry" --socket /tmp/foo.socket --master --pty-socket :5000
   
.. code-block:: sh

   ./uwsgi --pty-connect :5000

例子2：IPython控制线程
*********************************

.. code-block:: py

   import IPython
   from uwsgidecorators import *

   # only worker 1 has the pty attached
   @postfork(1)
   @thread
   def tshell():
       while True:
           IPython.embed()
           

