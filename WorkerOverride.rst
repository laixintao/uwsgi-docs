覆盖worker
==================

多亏了暴露给插件的"worker"钩子，你可以覆盖每个uWSGI worker运行的代码。

目前，python插件是唯一一个暴露它的插件：

.. code-block:: ini

   [uwsgi]
   ; create a bunch of sockets
   socket = 127.0.0.1:3031
   socket = 127.0.0.1:3032
   ; spawn the master
   master = true
   ; spawn 4 processes
   processes = 4
   ; load a python script as the worker code
   python-worker-override = aioserver.py


该python脚本可以访问uwsgi模块，因此它可以控制/改变内部逻辑。

下面的例子展示了aiohttp的使用 (需要python 3.5)

.. code-block:: python

   import asyncio
   from aiohttp import web

   import uwsgi
   import socket
   import sys
   import signal

   async def handle(request):
      name = request.match_info.get('name', "Anonymous")
      text = "Hello, " + name
      return web.Response(body=text.encode('utf-8'))

   async def wshandler(request):
      ws = web.WebSocketResponse()
      await ws.prepare(request)

      async for msg in ws:
        if msg.tp == web.MsgType.text:
            ws.send_str("Hello, {}".format(msg.data))
        elif msg.tp == web.MsgType.binary:
            ws.send_bytes(msg.data)
        elif msg.tp == web.MsgType.close:
            break

      return ws

    async def init(loop, fd):
       app = web.Application(loop=loop)
       app.router.add_route('GET', '/echo', wshandler)
       app.router.add_route('GET', '/{name}', handle)

       srv = await loop.create_server(app.make_handler(),
                                      sock=socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM))
       print("asyncio server started on uWSGI {0}".format(uwsgi.version))
       return srv

   def destroy():
      print("destroy worker {0}".format(uwsgi.worker_id()))
      sys.exit(0)

   def graceful_reload():
      print("graceful reload for worker {0}".format(uwsgi.worker_id()))
      # TODO do somethign meaningful
      sys.exit(0)

   loop = asyncio.get_event_loop()
   loop.add_signal_handler(signal.SIGINT, destroy)
   loop.add_signal_handler(signal.SIGHUP, graceful_reload)
   # spawn a handler for every uWSGI socket
   for fd in uwsgi.sockets:
      loop.run_until_complete(init(loop, fd))
   uwsgi.accepting()
   loop.run_forever()


在这个例子中 (来自官方的aiohttp文档)，我们看到uwsgi.sockets列表 (保存uWSGI socket文件描述符列表)，以及对SIGINT和SIGHUP的覆盖，以支持重新加载 (SIGHUP应该调整以支持等待所有的入队请求)

调用 :py:func:`uwsgi.accepting()` 以通知master，worker正在接收请求，这对于touch-chain-reload正常工作是必须的。

应该扩展该脚本，在每个请求后调用uwsgi.log(...)，以及（最后）更新一些度量
