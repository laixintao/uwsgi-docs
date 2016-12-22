Fork Server和CPython
========================

我们的"base"应用 (/var/www中的myforkbase.py):

.. code-block:: py

   # you should see this message only in the base instance
   print "I AM THE MODULE"

   def application(e, sr):
       sr('200 OK',[('Content-Type','text/plain')])
       return ['Hello World']
       
       
base vassal (在/etc/forkvassals/base.ini中)

.. code-block:: ini

   [uwsgi]
   ; add /var/www to the PYTHONPATH (asap)
   early-python-path = /var/www
   ; import myforkbase.py (asap)
   early-pyimport = myforkbase
   ; spawn the fork server and suspend the vassal
   fork-server = /run/forkme
   

然后现在是两个继承自base vassal的vassal

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
   
`--emperor-collect-attr` 选项告诉Emperor在[emperor]部分搜索一个'myfork-base'属性，而 `--emperor-fork-server-attr` 指示它把参数当成fork服务器来连接。

待办事项
====

--emperor-collect-attr可以由emperor-fork-server-attr隐含
