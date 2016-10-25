Fork Server with CPython
========================

我们的"base"应用 (/var/www中的myforkbase.py):

.. code-block:: py

   # you should see this message only in the base instance
   print "I AM THE MODULE"

   def application(e, sr):
       sr('200 OK',[('Content-Type','text/plain')])
       return ['Hello World']
       
       
The base vassal (in /etc/forkvassals/base.ini)

.. code-block:: ini

   [uwsgi]
   ; add /var/www to the PYTHONPATH (asap)
   early-python-path = /var/www
   ; import myforkbase.py (asap)
   early-pyimport = myforkbase
   ; spawn the fork server and suspend the vassal
   fork-server = /run/forkme
   

and now two vassals inheriting from the base

.. code-block:: ini

   [emperor]
   ; vassal's attribute
   myfork-base = /run/forkme

   [uwsgi]
   http-socket = :9090
   processes = 4
   ; the python VM will find myappfork already loaded
   wsgi = myappfork
   
   
.. code-block:: ini

   [emperor]
   ; vassal's attribute
   myfork-base = /run/forkme

   [uwsgi]
   http-socket = :9091
   processes = 8
   threads = 2
   ; the python VM will find myappfork already loaded
   wsgi = myappfork
   
   
最后，运行Emperor

.. code-block:: sh

   uwsgi --emperor /etc/forkvassals --emperor-collect-attr myfork-base --emperor-fork-server-attr myfork-base
   
the `--emperor-collect-attr` option tells the Emperor to search for a 'myfork-base' attribute in the [emperor] section, while `--emperor-fork-server-attr`
instruct it to use the parameter as the fork-server to connect to.

待办事项
====

The --emperor-collect-attr could be implicited by emperor-fork-server-attr
