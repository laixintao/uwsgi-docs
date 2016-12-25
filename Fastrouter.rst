uWSGI FastRouter
====================

对于高级设置，uWSGI包含了"fastrouter"插件，它是一个代理/负载均衡器/路由器，使用uwsgi协议。默认内建它。你可以将其放在你的web服务器和真正的uWSGI实例之间，以获取对于HTTP请求到你的应用服务器的路由的更多控制。

开始
---------------

首先，你必须运行fastrouter，将它绑定到一个指定的地址上。也支持多个地址。

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter /tmp/uwsgi.sock --fastrouter @foobar

.. note:: 这是世界上最没用的Fastrouter设置。

恭喜你！你刚刚运行了世界上最没用的Fastrouter设置。简单将fastrouter绑定到一堆地址上并不会指示它如何路由请求。要赋予它智能，你必须告诉它如何路由请求。

方法1：--fastrouter-use-base
----------------------------

这个选项将会告诉fastrouter连接到一个UNIX socket，这个socket的名字与指定目录中的请求主机的名字相同。

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-use-base /tmp/sockets/

如果你接收到一个对 ``example.com`` 的请求，那么fastrouter将会转发请求到 ``/tmp/sockets/example.com`` 。

方法2：--fastrouter-use-pattern
-------------------------------

与前面的设置相同，但是你将能够使用模式，其中， ``%s`` 映射到请求的键/主机名。

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-use-pattern /tmp/sockets/%s/uwsgi.sock

对 ``example.com`` 的请求将会被映射到
``/tmp/sockets/example.com/uwsgi.sock``.

方法3：--fastrouter-use-cache
-----------------------------

你可以在 :doc:`uWSGI cache <Caching>` 中存储键值映射。选择一种方式去填充缓存，例如，一个像这样的Python脚本……

.. code-block:: py

    import uwsgi
    # Requests for example.com on port 8000 will go to 127.0.0.1:4040
    uwsgi.cache_set("example.com:8000", "127.0.0.1:4040")
    # Requests for unbit.it will go to 127.0.0.1:4040 with the modifier1 set to 5 (perl/PSGI)
    uwsgi.cache_set("unbit.it", "127.0.0.1:4040,5")

然后运行你启用了Fastrouter的服务器，告诉它首先运行脚本。

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-use-cache --cache 100 --file foobar.py

方法4：--fastrouter-subscription-server
---------------------------------------

这可能是用于大量自动扩展托管的最好的方式之一。它使用 :doc:`subscription server <SubscriptionServer>` 来允许实例宣告自己，并订阅到fastrouter。

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-subscription-server 192.168.0.100:7000
    
这将会在地址192.168.0.100，端口7000上生成一个订阅服务器

现在，你可以生成订阅到fastrouter的实例了：

.. code-block:: sh

    uwsgi --socket :3031 -M --subscribe-to 192.168.0.100:7000:example.com
    uwsgi --socket :3032 -M --subscribe-to 192.168.0.100:7000:unbit.it,5 --subscribe-to 192.168.0.100:7000:uwsgi.it

正如你可能注意到的，你可以订阅多个fastrouter，使用多个键。具有相同的键的订阅到相同fastrouter的多个实例将会自动负载均衡，并被监控。很方便，不是吗？就像缓存键/值存储，可以用一个逗号来设置 ``modifier1`` 。(上面是 ``,5`` ) 订阅系统的另一个特性是避免选择端口。你可以将实例绑定到随机端口，而订阅系统将会发送真实的值到订阅服务器上。

.. code-block:: sh

    uwsgi --socket 192.168.0.100:0 -M --subscribe-to 192.168.0.100:7000:example.com


映射文件
^^^^^^^^^^^^^

如果你需要指定大量的键，那么你可以使用一个映射文件来取代。

.. code-block:: plain

    # mappings.txt
    unbit.it
    unbit.it:8000,5
    uwsgi.it
    projects.unbit.it

.. code-block:: sh

    uwsgi --socket :3031 -M --subscribe-to 192.168.0.100:7000:@mappings.txt

方法5：--fastrouter-use-code-string
-----------------------------------

如果Darth Vader穿着一个带有你的脸的T恤 (还要其他一些边界情况)，那么你可以使用代码驱动的映射来自定义fastrouter。选择一个支持uWSGI的语言 (就像Python或者Lua)，然后定义你的映射函数。

.. code-block:: py

    def get(key):
        return '127.0.0.1:3031'

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-use-code-string 0:mapper.py:get

这将会指示fastrouter使用插件(modifier1) 0加载脚本 ``mapper.py`` ，并且调用'get'全局函数，传给它key值。在前面的例子中，你会总是路由请求到127.0.0.1:3031。让我们创建一个更高级的系统，只是为了好玩！

.. code-block:: py
    
    domains = {}
    domains['example.com'] = {'nodes': ('127.0.0.1:3031', '192.168.0.100:3032'), 'node': 0}
    domains['unbit.it'] = {'nodes': ('127.0.0.1:3035,5', '192.168.0.100:3035,5'), 'node': 0}
    
    DEFAULT_NODE = '192.168.0.1:1717'
    
    def get(key):
        if key not in domains:
            return DEFAULT_NODE
    
        # get the node to forward requests to
        nodes = domains[key]['nodes']
        current_node = domains[key]['node']
        value = nodes[current_node]
    
        # round robin :P
        next_node = current_node + 1
        if next_node >= len(nodes):
            next_node = 0
    
        domains[key]['node'] = next_node
    
        return value

.. code-block:: sh

    uwsgi --fastrouter 127.0.0.1:3017 --fastrouter-use-code-string 0:megamapper.py:get

只需短短几行，我们就实现了一个带有回退节点的循环负载均衡。Pow! 你可以添加某些形式的节点监控，在脚本中启动线程，或者其他疯狂的东东。 (确保将其添加到文档中！)

.. attention:: 记住，不要在你的函数中写阻塞代码。fastrouter是完全非阻塞的，不要毁了它！

Cheap模式和共享socket
-----------------------------

一个常见的设置是让一个web服务器/代理连接到一个fastrouter，并且让一系列的uWSGI实例订阅到它上面。通常请情况下，你会将web服务器节点当成一个uWSGI实例节点使用。这个节点将会订阅到本地fastrouter。好吧……不要在上面浪费时间周期！共享socket是一种在各种uWSGI部件之间共享socket的方式。让我们使用它在fastrouter和uWSGI实例之间共享socket。

.. code-block:: ini

    [uwsgi]
    ;create a shared socket (the webserver will connect to it)
    shared-socket = 127.0.0.1:3031
    
    ; bind the fastrouter to the shared socket
    fastrouter = =0
    ; bind an instance to the same socket
    socket = =0
    
    ; having a master is always a good thing...
    master = true
    ; our subscription server
    fastrouter-subscription-server = 192.168.0.100:4040
    ; our app
    wsgi-file = /var/www/myheavyapp.wsgi
    ; a bunch of processes
    processes = 4
    ; and put the fastrouter in cheap mode
    fastrouter-cheap = true
    

使用这个设置，你的请求将会直接到达你的应用（无代理开销），或者到fastrouter (传递请求给远程节点)。当fastrouter处于cheap模式的时候，它将不会响应请求，直到节点可用。这意味着，当没有订阅的节点的时候，只有你本地的应用会响应。当所有的节点都挂掉的时候，fastrouter将会回到cheap模式。看到套路了吗？另一个到棒棒哒的自动缩放的设置。


Post-buffering模式 (uWSGI >= 2.0.9)
------------------------------------

fastrouter (默认情况下) 是一个流代理。这意味着，一旦解析了uwsgi包 (即，请求头部)，它就会被转发到后端（们）。

现在，如果你的web代理也是一个流代理 (例如apache，或者uWSGI http路由器)，那么你的应用在带有请求体的请求下会被阻塞很长一段时间。说得更明白一点：

* 客户端启动请求，发送http头部
* web代理接收它，然后发送给fastrouter
* fastrouter接收它，然后发送给后端
* 客户端开始发送请求体块 (例如文件上传)
* web代理接收它们，然后转发给fastrouter
* fastrouter接收它们，然后转发给后端，以此类推

现在，想象有10个并发的客户端在做这件事，你将会得到10个处于忙碌状态不确定时间的应用服务器worker（或者线程）。(注意，这个问题会被这样一个事实放大：一般来说，线程/进程的数目是非常受限的，甚至是处于异步模式也是如此，因此，你有一个有限的并发请求，但是它通常如此之高，以至于这个问题没那么相关)

像nginx这样的web代理是会“缓冲”的，因此它们会等待，直到整个请求（及其请求体）读完，然后再将其发送到后端。

你可以用 ``--fastrouter-post-buffering <n>`` 选项指示fastrouter像nginx一样，其中，<n>是指，请求体到多大之后，将会被存储到磁盘（作为临时文件），而不是内存：

.. code-block:: ini

   [uwsgi]
   fastrouter = 127.0.0.1:3031
   fastrouter-to = /var/run/app.socket
   fastrouter-post-buffering = 8192
   
将会把fastrouter置于缓冲模式，每当请求体大于8192字节，就会把它存储到一个临时文件中，而当小于（或等于）时，则会存在内存里

记住，post-buffering，并非一本万利的解决方法 (否则，会默认使用它)，启用它会破坏websockets，块输入，上传过程等等等等。只在需要的时候启用它。

注意
-----

* fastrouter使用以下变量 (按优先顺序) 来选择使用的键：

  * ``UWSGI_FASTROUTER_KEY`` - 最通用的，因为它不依赖于以任何方式的请求
  * ``HTTP_HOST``
  * ``SERVER_NAME``

* 你可以使用--fastrouter-events增加fastrouter可以管理的异步事件数目 (默认情况下，它是依赖于系统的)

你可以用--fastrouter-timeout修改默认超时时间。默认情况下，当通过unix socket使用的时候，fastrouter会设置fd socket passing。如果你不想要它，那么添加--no-fd-passing
