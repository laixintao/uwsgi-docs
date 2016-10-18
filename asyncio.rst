asyncio循环引擎 (CPython >= 3.4, uWSGI >= 2.0.4)
========================================================

.. 警告::

  状态：实验中，有许多隐含式，特别是关于WSGI标准

 ``asyncio`` 插件公开了一个建立在 ``asyncio`` CPython API (https://docs.python.org/3.4/library/asyncio.html#module-asyncio)顶部的一个循环引擎。

由于uWSGI并不是基于回调的，因此你需要一个挂起引擎(目前只支持“greenlet”)来管理WSGI callable。

为什么不把WSGI callable映射到一个协程呢？
*********************************************

理由很简单：这会以各种可能的方式终端。(这里就不深入细节了。)

出于这个原因，每个uWSGI核被映射到一个greenlet (运行WSGI回调)上。

这个greenlet在asyncio事件循环中注册事件和协程。

Callable VS. 协程
***********************

当开始使用asyncio时，你可能会对Callable和协程之间感到困惑。

当一个特定的事件引发的时候（例如，当一个文件描述符准备用于读的时候），会执行Callable。它们基本上是在主greenlet中执行的标准函数 (最终，它们可以切换控制回特定的uWSGI核上)。

协程更复杂：它们很接近greenlet，但在内部，它们运行在Python帧之上，而不是C堆栈上。自一个Python程序员看来，协程是非常特别的生成器，你的WSGI callable可以生成协程。

利用asyncio支持构建uWSGI
***********************************

可以在官方源代码树(也将构建greenlet支持)中找到一个'asyncio'构建配置文件。

.. code-block:: sh

   CFLAGS="-I/usr/local/include/python3.4" make PYTHON=python3.4 asyncio
   
或者

.. code-block:: sh

   CFLAGS="-I/usr/local/include/python3.4" UWSGI_PROFILE="asyncio" pip3 install uwsgi
   
一定要使用Python 3.4+作为Python版本，并且添加greenlet include目录到 ``CFLAGS`` (如果你从发行包中安装了greenlet支持，那么这可能并不需要)。

第一个例子：一个简单的回调
************************************

让我们从一个简单的WSGI callable开始，它在该 callable返回之后2秒后触发一个函数（神奇！）。

.. code-block:: python

   import asyncio
   
   def two_seconds_elapsed():
       print("Hello 2 seconds elapsed")
   
   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       asyncio.get_event_loop().call_later(2, two_seconds_elapsed)
       return [b"Hello World"]
       
一旦被调用，那么该应用函数将在asyncio事件循环中注册一个 callable，然后会返回到客户端。

在2秒后，事件循环将会运行该函数。

你可以这样运行这个例子：

.. code-block:: sh

   uwsgi --asyncio 10 --http-socket :9090 --greenlet --wsgi-file app.py
   
``--asyncio`` 是一个快捷方式，它启用10个uWSGI异步核，让你能够用一个单一的进程就可以管理多达10个并发请求。
   
但是，如何在WSGI callable中等待一个回调完成呢？我们可以使用greenlet来挂起我们的WSGI函数 (记住，我们的WSGI callable是封装在一个greenlet中的):

.. code-block:: python

   import asyncio
   import greenlet
   
   def two_seconds_elapsed(me):
       print("Hello 2 seconds elapsed")
       # back to WSGI  callable
       me.switch()
   
   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       myself = greenlet.getcurrent()
       asyncio.get_event_loop().call_later(2, two_seconds_elapsed, myself)
       # back to event loop
       myself.parent.switch()
       return [b"Hello World"]
       
我们可以更进一步，为WSGI生成器滥用uWSGI支持：

.. code-block:: python

   import asyncio
   import greenlet

   def two_seconds_elapsed(me):
       print("Hello 2 seconds elapsed")
       me.switch()

   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       myself = greenlet.getcurrent()
       asyncio.get_event_loop().call_later(2, two_seconds_elapsed, myself)
       myself.parent.switch()
       yield b"One"
       asyncio.get_event_loop().call_later(2, two_seconds_elapsed, myself)
       myself.parent.switch()
       yield b"Two"

另一个例子：Future与协程
***************************************

你可以使用 ``asyncio.Task`` 从你的 WSGI callable中生成协程：

.. code-block:: python

   import asyncio
   import greenlet

   @asyncio.coroutine
   def sleeping(me):
       yield from asyncio.sleep(2)
       # back to callable
       me.switch()

   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       myself = greenlet.getcurrent()
       # enqueue the coroutine
       asyncio.Task(sleeping(myself))
       # suspend to event loop
       myself.parent.switch()
       # back from event loop
       return [b"Hello World"]

有了Future，我们甚至可以从协程中获取结果……

.. code-block:: python

   import asyncio
   import greenlet

   @asyncio.coroutine
   def sleeping(me, f):
       yield from asyncio.sleep(2)
       f.set_result(b"Hello World")
       # back to callable
       me.switch()


   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       myself = greenlet.getcurrent()
       future = asyncio.Future()
       # enqueue the coroutine with a Future
       asyncio.Task(sleeping(myself, future))
       # suspend to event loop
       myself.parent.switch()
       # back from event loop
       return [future.result()]
       
一个更高级的使用 ``aiohttp`` 模块的例子 (记住执行 ``pip install aiohttp`` 来安装它，它并不是一个标准库模块)

.. code-block:: python

   import asyncio
   import greenlet
   import aiohttp

   @asyncio.coroutine
   def sleeping(me, f):
       yield from asyncio.sleep(2)
       response = yield from aiohttp.request('GET', 'http://python.org')
       body = yield from response.read_and_close()
       # body is a byterray !
       f.set_result(body)
       me.switch()


   def application(environ, start_response):
       start_response('200 OK', [('Content-Type','text/html')])
       myself = greenlet.getcurrent()
       future = asyncio.Future()
       asyncio.Task(sleeping(myself, future))
       myself.parent.switch()
       # this time we use yield, just for fun...
       yield bytes(future.result())

状态
******

* 该插件被认为是实验性的 (WSGI中使用asyncio的影响目前尚未清楚)。未来，当检测到Python >= 3.4的时候，可能会默认构建。
* 虽然（或多或少）技术上是可行的，但是在不久的将来，并不期望将一个WSGI callable映射到一个Python 3协程上。
* 该插件为非阻塞的读/写和定时器注册钩子。这意味着，你可以自动使用uWSGI API和asyncio。看看 https://github.com/unbit/uwsgi/blob/master/tests/websockets_chat_asyncio.py 这个例子。
