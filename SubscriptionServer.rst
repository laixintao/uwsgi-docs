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

The subscription system is currently available for cluster joining (当多播/广播不能用的时候), the Fastrouter, the HTTP/HTTPS/SPDY router, the rawrouter and the sslrouter.

那就是说，你可以很快创建一个evented/fast_as_hell HTTP load balancer in no time.

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --master

Now simply subscribe your nodes to the HTTP subscription server.

You can check the subscription server stats and/or the subscribed nodes through the ``http-stats-server`` option.

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --http-stats-server 192.168.0.100:5004 --master

You can also forward subscription requests to another server with the option ``http-resubscribe``.

.. code-block:: sh

    uwsgi --http :80 --http-subscription-server 192.168.0.100:2626 --http-resubscribe 192.168.0.101:2627 --master

Securing the Subscription System
--------------------------------

The subscription system is meant for "trusted" networks. All of the nodes in your network can potentially make a total mess with it.

If you are building an infrastructure for untrusted users or you simply need more control over who can subscribe to a Subscription Server you can use openssl rsa public/private key pairs for "signing" you subscription requests.

.. code-block:: sh

    # First, create the private key for the subscriber. DO NOT SET A PASSPHRASE FOR THIS KEY.
    openssl genrsa -out private.pem
    # Generate the public key for the subscription server:
    openssl rsa -pubout -out test.uwsgi.it_8000.pem -in private.pem

The keys must be named after the domain/key we are subscribing to serve, plus the .pem extension.

.. note:: If you're subscribing to a pool for an application listening on a specified port you need to use the ``domain_port.pem`` scheme for your key files. Generally all of the DNS-allowed chars are supported, all of the others are mapped to an underscore.

An example of an RSA protected server looks like this:

.. code-block:: ini

    [uwsgi]
    master = 1
    http = :8000
    http-subscription-server = 127.0.0.1:2626
    subscriptions-sign-check = SHA1:/etc/uwsgi/keys

The last line tells uWSGI that public key files will be stored in /etc/uwsgi/keys.

At each subscription request the server will check for the availability of the public key file and use it, if available, to verify the signature of the packet. Packets that do not correctly verify are rejected.

On the client side you need to pass your private key along with other ``subscribe-to`` options. Here's an example:

.. code-block:: ini

    [uwsgi]
    socket = 127.0.0.1:8080
    subscribe-to = 127.0.0.1:2626:test.uwsgi.it:8000,5,SHA1:/home/foobar/private.pem
    psgi = test.psgi

Let's analyze the ``subscribe-to`` usage:

* ``127.0.0.1:2626`` is the subscription server we want to subscribe to.
* ``test.uwsgi.it:8000`` is the subscription key.
* ``5`` is the modifier1 value for our psgi app
* ``SHA1:/home/private/test.uwsgi.it_8000.pem`` is the <digest>:<rsa> couple for authenticating to the server (the <rsa> field is the private key path).

.. note:: Please make sure you're using the same digest method (SHA1 in the examples above) both on the server and on the client.

To avoid replay attacks, each subscription packet has an increasing number (normally the unix time) avoiding the allowance of duplicated packets.
Even if an attacker manages to sniff a subscription packet it will be unusable as it is already processed previously.
Obviously if someone manages to steal your private key he will be able to build forged packets.

使用SSH密钥
**************

SSH-formatted keys are generally loved by developers (well, more than classic PEM files).

Both --subscribe-to and --subscribe2 (see below) support SSH private keys, while for the server part you have the encode the public key in pkcs8:

.. code-block:: sh

   ssh-keygen -f chiavessh001.pub -e -m pkcs8
   
--subscribe2
------------

This is the keyval version of --subscribe-to. It supports more tricks and a (generally) more readable syntax:

.. code-block:: sh

   uwsgi --socket 127.*:0 --subscribe2 server=127.0.0.1:7171,key=ubuntu64.local:9090,sign=SHA1:chiavessh001
   
   
支持的字段是：

* ``server`` the address of the subscription server
* ``key`` the key to subscribe (generally the domain name)
* ``addr`` the address to subscribe (the value of the item)
* ``socket`` the socket number (zero-based), this is like 'addr' by take the uWSGI internal socket number
* ``weight`` the load balancing value
* ``modifier1`` and ``modifier2``
* ``sign`` <algo>:<file> the signature for the secured system
* ``check`` it takes a file as argument. If it exists the packet is sent, otherwise it is skipped
* ``sni_key`` set the keyfile to use for SNI proxy management
* ``sni_crt`` set the crt file to use for SNI proxy management
* ``sni_ca`` set the ca file to use for SNI proxy management
* ``algo`` (uWSGI 2.1) set the load balancing algorithm to use (they are pluggable, included are wrr, lrc, wlrc and iphash)
* ``proto`` (uWSGI 2.1) the protocol to use, by default it is 'uwsgi'
* ``backup`` (uWSGI 2.1) set the backup level (change meaning based on algo)

通知
-------------

When you subscribe to a server, you can ask it to "acknowledge" the acceptance of your request.

Just add ``--subscription-notify-socket <addr>`` pointing to a datagram (Unix or UDP) address, on which your instance will bind and the subscription server will send acknowledgements to.

挂载点 (uWSGI 2.1)
-----------------------

Generally you subscribe your apps to specific domains.

Thanks to the mountpoint support introduced in uWSGI 2.1, you can now subscribe each node to a specific directory (you need to specify how much levels you want to support):

First of all you need to tell the subscription server to accept (and manage) mountpoint requests:

.. code-block:: sh

   uwsgi --master --http :8080 --http-subscription-server 127.0.0.1:4040 --subscription-mountpoints 1
   
Then you can start subscribing to mountpoints.
   
.. code-block:: sh

   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/foo
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/bar
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it/foo
   uwsgi --socket 127.0.0.1:0 --subscribe2 server=127.0.0.1:4040,key=mydomain.it

The first and the third instance will answer to all of the requests for /foo, the second will answer for /bar and the last one will manage all of the others.

For the secured subscription system, you only need to use the domain key (you do not need to generate a certificate for each mountpoint).

If you want to support mountpoints in the form /one/two instead of /one, just pass '2' to --subscription-mountpoints and so on. For performance reason you need to choose how much elements your path can support, and you cannot mix them (read: if --subscription-mountpoints is 2 you can support /one/two or /foo/bar but not /foobar)
