WebSocket支持
=================

在uWSGI 1.9中，添加了一个高性能的websocket (RFC 6455) 实现。

虽然对于WebSockets，存在许多不同的解决方案，但是其中大多数依赖于更高级的语言实现，对于诸如游戏或者流这样的主题很少有足够好的。

默认编译uWSGI websockets实现。

Websocket支持是由20Tab S.r.l. http://20tab.com/ 赞助的。

它们发布了一个完整的游戏 (基于uWSGI websockets api的bomberman复制): https://github.com/20tab/Bombertab

一个echo服务器
**************

这是一个uWSGI websockets应用的样子：

.. code-block:: python

   def application(env, start_response):
       # complete the handshake
       uwsgi.websocket_handshake(env['HTTP_SEC_WEBSOCKET_KEY'], env.get('HTTP_ORIGIN', ''))
       while True:
           msg = uwsgi.websocket_recv()
           uwsgi.websocket_send(msg) 

你无需操心保持连接，或者拒绝死掉的对端。 ``uwsgi.websocket_recv()`` 将会在后台为你做一切脏活。

握手
***********

握手是一个websocket连接的第一个阶段。

要发送一个完整的握手响应，你可以使用 ``uwsgi.websocket_handshake([key,origin, proto])`` 函数。没有正确的握手，连接将会永远不会完成。

在1.9系列，key参数是必须的。在2.0+，你可以在不传递参数的情况下调用websocket_handshake (将会更加请求日期自动构建响应)。

发送
*******

发送数据给浏览器是相当容易的。 ``uwsgi.websocket_send(msg)`` -- 没有更多的了。

接收
*********

这是整个实现真正核心的部分。

这个函数实际上隐瞒了它的实际目的。它确实返回一个websocket消息，但它实际上也保持连接打开 (使用ping/pong子系统)，并且监控流状态。

``msg = uwsgi.websocket_recv()``

这个函数可以从一个命名通道（见下）接收消息，并且自动将其转发到你的websocket连接。

它将总是只返回从浏览器发送的websocket消息 -- 任何其他通信则在后台进行。

也有一个非阻塞变体 -- ``msg = uwsgi.websocket_recv_nb()`` 。见： https://github.com/unbit/uwsgi/blob/master/tests/websockets_chat_async.py

PING/PONG
*********

要保持一个websocket连接打开，你应该不断发送ping (或者pong，见下文) 到浏览器，并期望它响应。如果来自浏览器/客户端的响应不及时到达，那么就会关闭连接 (``uwsgi.websocket_recv()`` 将会引发一个异常)。除了ping之外， ``uwsgi.websocket_recv()`` 函数发送所谓的 '无偿pong'。它们被用来通知客户端服务端可用。

所有这些任务都发生在后台，你无需管理它们！

可用代理
*****************

不幸的是，并非所有的HTTP web服务器/代理都与websockets工作得很好。

* uWSGI HTTP/HTTPS/SPDY路由器完美支持它们。只需记得添加 ``--http-websockets`` 选项。

  .. code-block:: sh

   uwsgi --http :8080 --http-websockets --wsgi-file myapp.py
   
或者

.. code-block:: sh

   uwsgi --http :8080 --http-raw-body --wsgi-file myapp.py
   
这有点更“原始”，但支持诸如块输入这样的东东。

* Haproxy正常工作。

* nginx >= 1.4 正常工作，并且无需额外的配置。

语言支持
****************

* Python https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.py
* Perl https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.pl
* PyPy https://github.com/unbit/uwsgi/blob/master/tests/websockets_chat_async.py
* Ruby https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.ru
* Lua https://github.com/unbit/uwsgi/blob/master/tests/websockets_echo.lua

支持的并发模型
****************************

* 多进程
* 多线程
* uWSGI原生异步api
* Coro::AnyEvent
* gevent
* Ruby fibers + uWSGI async
* Ruby threads
* greenlet + uWSGI async
* uGreen + uWSGI async
* PyPy continulets

wss:// (基于https的websockets)
******************************

uWSGI HTTPS路由器可用websockets。只需记得在你的客户端代码中使用wss://作为连接scheme。

基于SPDY的Websockets
********************

n/a

路由
*******

http代理内部路由器直接支持websocket (假设你的前线代理已经支持它们了)

.. code-block:: ini

   [uwsgi]
   route = ^/websocket uwsgi:127.0.0.1:3032,0,0
   
或者

.. code-block:: ini

   [uwsgi]
   route = ^/websocket http:127.0.0.1:8080

Api
***

uwsgi.websocket_handshake([key, origin, proto])

uwsgi.websocket_recv()

uwsgi.websocket_send(msg)

uwsgi.websocket_send_binary(msg) (在1.9.21中添加，以支持二进制消息)

uwsgi.websocket_recv_nb()

uwsgi.websocket_send_from_sharedarea(id, pos) (在1.9.21中添加，允许直接从 :doc:`SharedArea` 发送)

uwsgi.websocket_send_binary_from_sharedarea(id, pos) (在1.9.21中添加，允许直接从 :doc:`SharedArea` 发送)
