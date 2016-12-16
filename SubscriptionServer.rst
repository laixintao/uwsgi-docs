uWSGI订阅服务器
=========================

uWSGI栈的一些部分要求有一个键值映射系统。

例如， :doc:`Fastrouter` 需要知道对于一个特定的请求，要联系哪个服务器。

在拥有大量的节点的大网络中，手工管理这些配置可能就是地狱般可怕。uWSGI实现了一个订阅系统，其中，节点本身向订阅服务器宣告它的存在，这将反过来填入它们的内部字典。

.. code-block:: sh

    uwsgi --fastrouter :1717 --fastrouter-subscription-server 192.168.0.100:2626

这将会在端口1717上运行一个uWSGI fastrouter，并且创建一个空字典，其中，主机名是键，而uwsgi地址则是值。

要填充这个字典，你可以联系192.168.0.100:2626，这是订阅服务器的地址。

对于每个键，可以存在多个地址，启用负载均衡 (各种算法也能用)。

一个节点可以使用 ``subscribe-to`` 或者 ``subscribe2`` 选项来向订阅服务器宣告它的存在。

.. code-block:: sh

    uwsgi --socket 192.168.0.10:3031 --wsgi myapp -M --subscribe-to 192.168.0.100:2626:uwsgi.it

FastRouter将会映射每个对uwsgi.it的请求到192.168.0.10:3031。

现在，为uwsgi.it添加第二个节点，简单运行它并订阅：

.. code-block:: xxx

    uwsgi --socket 192.168.0.11:3031 --wsgi myapp --master --subscribe-to 192.168.0.100:2626:uwsgi.it

会自动将死掉的节点从池中移除。

 ``subscribe2`` 的语法是类似的，但是它允许更多的控制，因为它允许指定额外的选项，例如所有的请求应该被转发到哪个地址。它的值语法是一个带"key=value"对的字符串，每个由一个逗号分隔。

.. code-block:: sh

    uwsgi -s 192.168.0.10:3031 --wsgi myapp --master --subscribe2 server=192.168.0.100:2626,key=uwsgi.it,addr=192.168.0.10:3031

可用 ``subscribe2`` 键的列表，见下。

该订阅系统目前对集群加入（cluster joining） (当多播/广播不能用的时候)、Fastrouter、HTTP/HTTPS/SPDY路由器、rawrouter和sslrouter可用。

那就是说，你可以很快创建一个事件化/fast_as_hell的HTTP负载均衡器。

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --master

现在，简单订阅你的节点到HTTP订阅服务器。

你可以通过 ``http-stats-server`` 选项检查订阅服务器统计信息以及/或者订阅节点。

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --http-stats-server 192.168.0.100:5004 --master

你还可以使用 ``http-resubscribe`` 选项转发订阅请求到其他服务器。

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --http-resubscribe 192.168.0.101:2627 --master

防护订阅系统
--------------------------------

订阅服务器意味着“可信任”网络。你网络中的所有节点可能潜在会制造大混乱。

如果你正为不可信任用户构建基础设施，或者你只是对可以订阅到订阅服务器的用户需要更多的控制权，那么你可以使用openssl rsa 公钥/密钥对，来“签名”你的订阅请求。

.. code-block:: sh

    # First, create the private key for the subscriber. DO NOT SET A PASSPHRASE FOR THIS KEY.
    openssl genrsa -out private.pem
    # Generate the public key for the subscription server:
    openssl rsa -pubout -out test.uwsgi.it_8000.pem -in private.pem

密钥必须以我们订阅到服务器的域名/键命名，加上.pem扩展名。

.. note:: 如果你正订阅到一个池，使得应用监听到一个指定的端口，那么你需要为你的密钥文件使用 ``domain_port.pem`` 模式。一般而言，支持所有DNS允许的字符，其他都会被映射到一个下划线。

一个RSA保护的服务器如下：

.. code-block:: ini

    [uwsgi]
    master = 1
    http = :8000
    http-subscription-server = 127.0.0.1:2626
    subscriptions-sign-check = SHA1:/etc/uwsgi/keys

最后一行告诉uWSGI，公钥文件将会存储在/etc/uwsgi/keys中。

对于每个订阅请求，服务器将会检查公钥文件的可用性，如果可用，则会用它来验证包的签名。拒绝不能正确验证的包。

在客户端，你需要传递你的私钥，以及其他 ``subscribe-to`` 选项。这是一个例子：

.. code-block:: ini

    [uwsgi]
    socket = 127.0.0.1:8080
    subscribe-to = 127.0.0.1:2626:test.uwsgi.it:8000,5,SHA1:/home/foobar/private.pem
    psgi = test.psgi

让我们分析 ``subscribe-to`` 使用：

* ``127.0.0.1:2626`` 是我们想要订阅的订阅服务器。
* ``test.uwsgi.it:8000`` 是订阅键。
* ``5`` 是用于我们的psgi应用的modifier1值
* ``SHA1:/home/private/test.uwsgi.it_8000.pem`` 是用来鉴权服务器的 <digest>:<rsa> 对 (<rsa>字段是私钥地址)。

.. note:: 请确保你在服务器和客户端都使用相同的摘要方法 (上面的例子中是SHA1)。

为了避免重放攻击，每个订阅包都有一个增量数字 (一般是Unix时间)，避免允许重复包。即使攻击者试图嗅探一个订阅包，它也会是不可用的，因为之前已经处理它了。显然，如果有人试图窃取你的私钥，他将能够构建伪造数据包。

使用SSH密钥
**************

SSH格式的密钥一般受到开发者喜爱 (嗯，比经典的PEM文件更受欢迎)。

--subscribe-to和--subscribe2 (见下) 都支持SSH密钥，而对于服务器部分，你要用pkcs8编码公钥：

.. code-block:: sh

   ssh-keygen -f chiavessh001.pub -e -m pkcs8
   
--subscribe2
------------

这是--subscribe-to的键值版本。它支持更多技巧，以及（一般）更可读的语法：

.. code-block:: sh

   uwsgi --socket 127.*:0 --subscribe2 server=127.0.0.1:7171,key=ubuntu64.local:9090,sign=SHA1:chiavessh001
   
   
支持的字段是：

* ``server`` 订阅服务器的地址
* ``key`` 订阅的键 (一般是域名)
* ``addr`` 订阅的地址 (项的值)
* ``socket`` socket数字 (基于0)，这就像'addr'，通过接收uWSGI内部socket值
* ``weight`` 负载均衡值
* ``modifier1`` 和 ``modifier2``
* ``sign`` <algo>:<file> 安全系统的签名
* ``check`` 它接收一个文件作为参数。如果存在，则发送包，否则，跳过它
* ``sni_key`` 为SNI代理管理设置密钥文件
* ``sni_crt`` 为SNI代理管理设置crt文件
* ``sni_ca`` 为SNI代理管理设置ca文件
* ``algo`` (uWSGI 2.1) 设置使用的负载均衡算法 (它们是可插拔的，包含wrr, lrc, wlrc和iphash)
* ``proto`` (uWSGI 2.1) 使用的协议，默认是'uwsgi'
* ``backup`` (uWSGI 2.1) 设置备份层次 (基于算法改变意义)

通知
-------------

当你订阅到一个服务器的时候，你可以让它“确认”你的请求的接受情况。

只需添加 ``--subscription-notify-socket <addr>`` ，指向一个数据报 (Unix或者UDP) 地址，你的实例将会绑定到上面，并且订阅服务器将会发送确认到这个地址。

挂载点 (uWSGI 2.1)
-----------------------

一般来说，你订阅你的应用到指定域上。

幸好有了uWSGI 2.1中引入的挂载点支持，你现在可以订阅每个节点到一个指定的目录了 (需要指定你想要支持多少层次)：

首先，你需要告诉订阅服务器支持 (和管理) 挂载点请求：

.. code-block:: sh

   uwsgi --master --http :8080 --http-subscription-server 127.0.0.1:4040 --subscription-mountpoints 1
   
然后，你可以开始订阅到挂载点。
   
.. code-block:: sh

   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/foo
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/bar
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/foo
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it

第一个和第三个实例将会响应所有对/foo的请求，而第二个将会响应/bar，最后一个会管理所有其他的请求。

对于安全的订阅系统，你只需要使用域名键（不需要为每个挂载点生成证书）。

如果你想以/one/two的形式，而不是/one的形式来支持挂载点，那么只需传递'2'到--subscription-mountpoints，等等。出于性能考虑，你需要选择你的路径可以支持多少个元素，并且不能弄混它们 (说明：如果--subscription-mountpoints是2，那么你可以支持/one/two或者/foo/bar，但不能支持/foobar)
