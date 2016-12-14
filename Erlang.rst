集成uWSGI和Erlang
=============================

.. warning:: 自1.9.20起，Erlang支持已经不能用了。新方案正在开发中。

uWSGI服务器可以作为一个Erlang C-Node，并且和Erlang节点交换数据以及进行RPC。

构建
--------

首先，你需要 ``ei`` 库和头文件。在官方的Erlang包中可以找到它们。如果你在Debian/Ubuntu之上，那么安装
``erlang-dev`` 包。可以嵌入Erlang支持，或者将其当成一个插件进行构建。要嵌入，则添加 ``erlang`` 和 ``pyerl`` 插件到你的构建配置文件中。

.. code-block:: ini

    embedded_plugins = python, ping, nagios, rpc, fastrouter, http, ugreen, erlang, pyerl

或者将其作为插件构建

.. code-block:: sh

    python uwsgiconfig --plugin plugins/erlang
    python uwsgiconfig --plugin plugins/pyerl

Erlang插件将允许uWSGI成为一个Erlang C-Node。 ``pyerl`` 插件将添加Erlang函数到Python插件中。

激活Erlang支持
-------------------------

你只需要设置两个选项来在你的启用Erlang的uWSGI构建中启用Erlang支持。 ``erlang`` 选项设置了你的uWSGI服务器的Erlang节点名。它可以以简单或扩展格式指定：

* ``nodename@ip``
* ``nodename@address``
* ``nodename``

 ``erlang-cookie`` 选项为内部节点通信设置cookie。如果你不指定它，那么会从 :file:`~/.erlang.cookie` 文件获取这个值。

要运行带启用Erlang的uWSGI：

.. code-block:: sh

    uwsgi --socket :3031 --erlang testnode@192.168.173.15 --erlang-cookie UUWSGIUWSGIU -p 2

一个简单的RPC hello world例子
--------------------------------

* 定义一个新的Erlang模块，它只导出一个简单的函数。

  .. code-block:: erlang
      
      -module(uwsgitest).
      -export([hello/0]).
      
      hello() ->
          'hello world !'.
  
* 启动 ``erl`` shell，指定节点名和（最终） (eventually) cookie:
  
  .. code-block:: sh
  
      erl -name testnode@192.168.173.1
  
* 编译uwsgitest Erlang模块
  
  .. code-block:: erlang
  
      c(uwsgitest).
      {ok,uwsgitest}
  
* ... 然后试着运行 ``hello`` 函数：
  
  .. code-block:: erlang
  
      uwsgitest:hello().
      'hello world !'

不错 - 现在，我们的Erlang模块就能用了，我们准备好RPC了！返回你的uWSGI服务器机器，然后定义一个新的WSGI模块 —— 让我们称之为
:file:`erhello.py`.

.. code-block:: py

    import uwsgi
    
    def application(env, start_response):
        testnode = uwsgi.erlang_connect("testnode@192.168.173.1")
        start_response('200 OK', [('Content-Type', 'text/plain')])
        yield uwsgi.erlang_rpc(testnode, "uwsgitest", "hello", [])
        uwsgi.erlang_close(testnode)

或者快速方式

.. code-block:: py

    import uwsgi
    
    def application(env, start_response):
        start_response('200 OK', [('Content-Type', 'text/plain')])
        yield uwsgi.erlang_rpc("testnode@192.168.173.1", "uwsgitest", "hello", [])

现在，使用这个新的模块启动uWSGI服务器：

.. code-block:: xxx

    uwsgi --socket :3031 --erlang testnode@192.168.173.15 --erlang-cookie UUWSGIUWSGIU -p 2 -w erhello

在你的浏览器中访问启用了uWSGI的web服务器，你应该会看到你的Erlang RPC调用的输出。

Python-Erlang映射
----------------------

uWSGI服务器试着根据下面这张表将Erlang类型转换成Python对象。

==========  ====== ====
Python      Erlang 注意
==========  ====== ====
str         binary
unicode     atom   受内部atom大小限制
int/long    int
list        list
tuple       tuple
3-tuple     pid
==========  ====== ====

发送消息给Erlang节点
--------------------------------

Erlang的最强大的特性之一是节点间消息传递系统。uWSGI也可以与Erlang节点进行通信。让我们定义一个新的Erlang模块，它简单回显任何我们发送给它的东西。

.. code-block:: erlang

    -module(uwsgiecho).
    -export([start/0, loop/0, echo/1]).
    
    echo(Message) ->
            {i_am_echo , Message}.
    
    loop() ->
            receive
                    Message1 ->
                            io:format("received a message~n"),
                            { useless, 'testnode@192.168.173.15' } ! echo(Message1)
            end,
            loop().
    
    start() ->
            register(echoer, spawn(uwsgiecho, loop, [])).

记得用Erlang ``register`` 函数注册你的进程。使用pid来标志进程是有问题的。现在，你可以用
:py:meth:`uwsgi.erlang_send_message` 来发送消息。

.. code-block:: py

    uwsgi.erlang_send_message(node, "echoer", "Hello echo server !!!" )

第二个参数是注册进程名。如果你不指定名字，那么传递一个Python的3元素元组会被解析为一个Pid。如果你的Erlang服务器返回消息给你的请求，那么你可以用
:py:meth:`uwsgi.erlang_recv_message` 来接收它们。记住，即使Erlang需要一个进程名/pid来发送消息，但是它们将被uWSGI所忽略。


接收Erlang消息
-------------------------

有时，你想直接从一个Erlang节点发送消息给uWSGI服务器。要接收Erlang消息，你必须在你的Python代码中注册"Erlang进程"。

.. code-block:: py

    import uwsgi
    
    def erman(arg):
        print "received an erlang message:", arg
    
    uwsgi.erlang_register_process("myprocess", erman)

现在，在Erlang中，你可发送消息给你注册的"myprocess"进程了：

.. code-block:: erlang

    { myprocess, 'testnode@192.168.173.15' } ! "Hello".


RPC
---

你可以直接从Erlang调用uWSGI :doc:`RPC` 函数。

.. code-block:: erlang

    rpc:call('testnode@192.168.173.15', useless, myfunction, []).

这将会调用一个配置为Erlang节点的uWSGI服务器上的"myfunction" uWSGI RPC函数。

连接持久化
----------------------

在高负载站点上，为每一个Erlang交互打开和关闭连接是很过分的。在你的应用初始化过程中用
:meth:`uwsgi.erlang_connect` 打开一个连接，并让文件描述符保持。

Mnesia又如何？
------------------

当你需要一个高可用性站点的时候，我们建议你使用 Mnesia_ 。构建一个Erlang模块来公开所有你需要的数据库交互，并且使用
:py:meth:`uwsgi.erlang_rpc` 来与之交互。

.. _Mnesia: http://en.wikipedia.org/wiki/Mnesia


我可以在uWSGI之上运行 EWGI_ 应用吗？
---------------------------------------------

现在，是不行的。最好的方式是开发一个插件，并且为EWGI应用分配一个特殊的modifier。

那在此之前，你可以在Python代码中封装即将到来的请求为EWGI形式，并使用 :py:meth:`uwsgi.erlang_rpc` 来调用你的Erlang应用。

.. _EWGI: http://code.google.com/p/ewgi/wiki/EWGISpecification
