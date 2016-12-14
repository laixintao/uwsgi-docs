uWSGI RPC栈
===============

uWSGI包含了一个快速、简单、平稳跨平台的RPC栈。

虽然，你或许会爱上这个子系统，但是，答应我，只在你需要的时候才用它，好吗？对于绝大部分常见，有大量更适合的高层次RPC技术可以用。

也就是说，uWSGI RPC子系统闪光点在于它的性能和内存使用。例如，如果你需要将一个请求的负载拆分到多个服务器上，那么uWSGI RPC是个不错的选择，因为它允许你不怎么费力就可以卸载任务。

它最大的限制在于它的“无类型”方法。

RPC函数可以接收多达254个参数。每个参数必须是一个字符串，最大大小为16位 (65535字节)，而返回值必须是一个字符串 (这次是64位，所以这不是一个实际的限制)。

.. warning:: 64位响应长度只在uWSGI 1.9.20中实现，较老的版本则是16位响应长度限制。

.. note:: RPC函数以二进制字符串的形式接收参数，所以每个RPC可导出函数必须假设每个参数都是一个字符串。每个RPC函数返回一个0或者拥有更多字符的二进制字符串。

所以，如果你需要“优雅的”或者强类型，那么就看看其他的吧 (或者，在uWSGI RPC之上自己处理，或许……)。

从1.9起，RPC子系统就是完全异步友好型的，因此，你可以把它跟gevent和Coro::AnyEvent等一起使用。

通过实例学习
-------------------

让我们从一个简单的RPC调用开始，这个调用从 ``10.0.0.1:3031`` 到 ``10.0.0.2:3031`` 。

所以，让我们导出 ``.2`` 上的一个"hello"函数

.. code-block:: py

    import uwsgi

    def hello_world():
        return "Hello World"

    uwsgi.register_rpc("hello", hello_world)

这使用了 :py:meth:`uwsgi.register_rpc` 来声明一个名为"hello"的函数，以备导出。我们会用 ``--socket :3031`` 启动这个服务器。

在调用者那端，在 ``10.0.0.1`` 之上，让我们声明这个世界（第二个）最简单的WSGI应用。

.. code-block:: py

    import uwsgi

    def application(env, start_response):
        start_response('200 Ok', [('Content-Type', 'text/html')])
        return uwsgi.rpc('10.0.0.2:3031', 'hello')

就这样！

你需要Perl？
^^^^^^^^^^^^^^

.. code-block:: perl
    #!/usr/bin/perl

    sub hello_world {
        return("hello world -- the time is: ".time());
        }

    uwsgi::register_rpc("hello",\&hello_world);

或者也许你想要从单个perl脚本中调用一个RPC函数？

.. code-block:: perl
    #!/usr/bin/perl
    use Net::uwsgi;
    print Dumper(Net::uwsgi::uwsgi_rpc('127.0.0.1:3031','hello'));


那么，比方说，Lua呢？
^^^^^^^^^^^^^^^^^^^^^^^^^^^

很高兴你问了。如果你想要导出Lua中的函数，简单这样：

.. code-block:: lua

    function hello_with_args(arg1, arg2)
        return 'args are '..arg1..' '..arg2
    end

    uwsgi.register_rpc('hellolua', hello_with_args)

而在你的Python WSGI应用中：

.. code-block:: py

    import uwsgi

    def application(env, start_response):
        start_response('200 Ok', [('Content-Type', 'text/html')]
        return uwsgi.rpc('10.0.0.2:3031', 'hellolua', 'foo', 'bar')

其他语言/平台？
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

查看语言特定文档，基本上，它们所有都应该支持注册和调用RPC函数。

你可以基本不花啥力气就能构建多语言应用，并且将会愉快地惊讶于调用来自于Perl，JavaScript，Python等等的 :doc:`Java<JVM>` 函数是有多容易。


本地进行RPC
-----------------

本地进行RPC听起来可能有点蠢，但如果你需要从Python调用一个Lua函数，并且使用绝对最少的开销，那么uWSGI RPC就是你的人啦。

如果你想要调用定义在同一台服务器（由同一个master管理，等）上的RPC，只需设置 ``uwsgi.rpc`` 的第一个参数为None或者nil，或者使用方便函数 :py:meth:`uwsgi.call` 。

从内部路由子系统进行RPC
---------------------------------------------

RPC插件导出了一堆内部路由动作：

* `rpc` 调用指定的rpc函数，并将响应发送给客户端
* `rpcnext/rpcblob` 调用指定的rpc函数，发送响应给客户端，并继续执行下一条规则
* `rpcret` 调用指定的rpc函数，并将其返回值当成动作返回码 (next, continue, goto ...)

.. code-block:: ini

   [uwsgi]
   route = ^/foo rpc:hello ${REQUEST_URI} ${REMOTE_ADDR}
   ; call on remote nodes
   route = ^/multi rpcnext:part1@192.168.173.100:3031
   route = ^/multi rpcnext:part2@192.168.173.100:3031
   route = ^/multi rpcnext:part3@192.168.173.100:3031


从nginx进行RPC
--------------------

由于Nginx支持对发送到上游uWSGI服务器的uwsgi包的低层次操作，所以你可以直接通过它进行RPC。疯了！

.. code-block:: nginx

    location /call {
        uwsgi_modifier1 173;
        uwsgi_modifier2 1;

        uwsgi_param hellolua foo
        uwsgi_param bar ""

        uwsgi_pass 10.0.0.2:3031;

        uwsgi_pass_request_headers off;
        uwsgi_pass_request_body off;
    }

大小为0的字符串将会被uWSGI数组解析器忽略，所以当参数数目+function_name不是偶数的时候，你可以安全使用它们。

Modifier2被设为1，来通知接收到了原始字符串 (在这个例子中，是HTTP响应)。否则，RPC子系统会将输出封装到一个uwsgi协议包中，而nginx还没智能到读取它们。


HTTP PATH_INFO -> RPC bridge
----------------------------

XML-RPC -> RPC bridge
---------------------
