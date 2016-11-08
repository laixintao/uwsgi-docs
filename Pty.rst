Pty插件
==============

-自uWSGI 1.9.15起可用，Linux, OpenBSD, FreeBSD和OSX皆支持。

This plugin allows you to attach pseudo terminals to your applications.

Currently the pseudoterminal server can be attached (and exposed over network) only on the first worker
(this limit will be removed in the future).

该插件还公开了一个客户端模式 (避免你把它跟netcat, telnet或者screen设置混在一起)


Building it
***********

The plugin is not included in the default build profiles, so you have to build it manually:

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/pty [profile]
   
(remember to specify the build profile if you are not using the default one)

Example 1: Rack application shared debugging
********************************************

.. code-block:: sh

   UWSGI_PROFILE=ruby2 UWSGI_EMBED_PLUGINS=pty make
   
.. code-block:: sh

   ./uwsgi --rbshell="require 'pry';binding.pry" --socket /tmp/foo.socket --master --pty-socket :5000
   
.. code-block:: sh

   ./uwsgi --pty-connect :5000

Example 2: IPython control thread
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
           

