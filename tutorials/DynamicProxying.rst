使用RPC和内部路由构建一个动态代理
====================================================

正在进行中 (要求uWSGI 1.9.14，我们使用PyPy作为引擎)

第一步：构建你的映射函数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用hostname作为映射 (你可以使用任何你需要的)

.. code-block:: py

    import uwsgi
    
    def my_mapper(hostname):
        return "127.0.0.1:3031"
        
    uwsgi.register_rpc('the_mapper', my_mapper)
    
将其保持为myfuncs.py
    

第二步：构建一个路由表
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: ini

   [uwsgi]
   ; enable the pypy engine
   pypy-home = /opt/pypy
   ; execute the myfuncs.py file (the 'the_mapper' rpc function will be registered)
   pypy-exec = myfuncs.py
   
   ; bind to a port
   http-socket = :9090
   
   ; let's define our routing table
   
   ; at every request (route-run execute the action without making check, use it instead of --route .*) run the_mapper passing HTTP_HOST as argument
   ; and place the result in the MYNODE variable
   route-run = rpcvar:MYNODE the_mapper ${HTTP_HOST}
   ; print the MYNODE variable (just for fun)
   route-run = log:${MYNODE}
   ; proxy the request to the chosen backend node
   route-run = http:${MYNODE}
   
   ; enable offloading for automagic non-blocking behaviour
   ; a good value for offloading is the number of cpu cores
   offload-threads = 2
   
