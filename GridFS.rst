GridFS插件
=================

从uWSGI 1.9.5开始，发布了一个"GridFS"插件。它同时公开了一个请求处理器和一个内部路由函数。它的官方modifier是'25'。路由指令则是"gridfs"。该插件是用C++写的。

必要条件和安装
************************

要构建这个插件，你需要 ``libmongoclient`` 头文件 (和一个正常运作的C ++编译器)。在一个类似于Debian的系统中，你可以这样做。

.. code-block:: sh

   apt-get install mongodb-dev g++

有一个用于gridfs的构建配置文件可用：

.. code-block:: sh

   UWSGI_PROFILE=gridfs make

或者你可以将它作为插件构建：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/gridfs

要快速安装单片构建，你可以使用网络安装程序：

.. code-block:: sh

   curl http://uwsgi.it/install | bash -s gridfs /tmp/uwsgi

这将会安装一个启用了gridfs的uwsgi二进制文件。


独立快速入门
*********************

这是一个独立的配置，它盲目映射传入的 ``PATH_INFO`` 到名为"test"的GridFS db中的项：

.. code-block:: ini

   [uwsgi]
   ; you can remove the plugin directive if you are using a uWSGI gridfs monolithic build
   plugin = gridfs
   ; bind to http port 9090
   http-socket = :9090
   ; force the modifier to be the 25th
   http-socket-modifier1 = 25
   ; map gridfs requests to the "test" db
   gridfs-mount = db=test

假设在你的GridFS中存储了myfile.txt文件，即"/myfile.txt"，那么运行以下命令：

.. code-block:: sh

   curl -D /dev/stdout http://localhost:9090/myfile.txt

然后你应该可以获取到这个文件。

最初斜杠的问题
*************************

一般来说， ``PATH_INFO`` 都会带一个'/'前缀。如果你并不使用绝对路径名来存储项，那么这在GridFS路径解析中会引发问题。为了解决这个问题，你可以让 ``gridfs`` 插件跳过最初的斜杠：

.. code-block:: ini

   [uwsgi]
   ; you can remove the plugin directive if you are using a uWSGI gridfs monolithic build
   plugin = gridfs
   ; bind to http port 9090
   http-socket = :9090
   ; force the modifier to be the 25th
   http-socket-modifier1 = 25
   ; map gridfs requests to the "test" db
   gridfs-mount = db=test,skip_slash=1

现在，它将会搜索"myfile.txt"，以取代搜索/myfile.txt。

多挂载点 (和服务器)
**********************************

你可以在不同SCRIPT_NAME (或者UWSGI_APPID)下挂载不同的GridFS数据库。如果你的web服务器能够正确管理 ``SCRIPT_NAME`` 变量，那么你不需要任何额外的设置 (除了--gridfs-mount)。否则，不要忘了添加--manage-script-name选项

.. code-block:: ini

   [uwsgi]
   ; you can remove the plugin directive if you are using a uWSGI gridfs monolithic build
   plugin = gridfs
   ; bind to http port 9090
   http-socket = :9090
   ; force the modifier to be the 25th
   http-socket-modifier1 = 25
   ; map gridfs requests to the "test" db
   gridfs-mount = db=test,skip_slash=1
   ; map /foo to db "wolverine" on server 192.168.173.17:4040
   gridfs-mount = mountpoint=/foo,server=192.168.173.17:4040,db=wolverine
   ; map /bar to db "storm" on server 192.168.173.30:4040
   gridfs-mount = mountpoint=/bar,server=192.168.173.30:4040,db=storm
   ; force management of the SCRIPT_NAME variable
   manage-script-name = true

.. code-block:: sh

    curl -D /dev/stdout http://localhost:9090/myfile.txt
    curl -D /dev/stdout http://localhost:9090/foo/myfile.txt
    curl -D /dev/stdout http://localhost:9090/bar/myfile.txt

这样，每个请求将会映射到一个不同的GridFS服务器。

副本集合
************

如果你正使用一个副本集合，那么你可以通过这个语法在你的uWSGI配置中使用它：<replica>server1,server2,serverN...

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   gridfs-mount = server=rs0/ubuntu64.local\,raring64.local\,mrspurr-2.local,db=test

注意用来转义服务器列表的反斜杠。

前缀
********

和移除初始斜杠一样，你或许需要给每个项的名字加前缀：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   gridfs-mount = server=rs0/ubuntu64.local\,raring64.local\,mrspurr-2.local,db=test,prefix=/foobar___

对/test.txt的请求将会被映射到/foobar___/test.txt

而 

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   gridfs-mount = server=rs0/ubuntu64.local\,raring64.local\,mrspurr-2.local,db=test,prefix=/foobar___,skip_slash=1

将会映射到/foobar___test.txt

MIME类型和文件名
************************

默认情况下，文件的MIME类型是由GridFS中存储的文件名而来的。这个文件名可能不会映射到有效请求的URI，或者你可能不想要为你的响应设置一个 ``content_type`` 。或者你也许想要允许一些其他系统设置它。如果你想要禁用MIME类型生成，那么只需添加 ``no_mime=1`` 到挂载选项。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,no_mime=1

如果你想要你的响应使用原始值来设置文件名 (存储在GridFS中的那个)，则添加 ``orig_filename=1``

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,no_mime=1,orig_filename=1

超时
********

你可以通过添加 ``timeout=N`` 到选项中，来设置低层次MongoDB操作的超时时间：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   ; set a 3 seconds timeout
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,timeout=3

MD5和ETag头部
********************

GridFS存储每个文件的MD5哈希值。你可以添加这个信息到你的响应头，作为ETag (十六进制格式的MD5)或者Content-MD5 (in Base64)的值。对于添加ETag头部，使用
``etag=1`` ，而对于添加Content-MD5，则使用 ``md5=1`` 。没有什么可以阻止你同时添加这两个头到响应中。

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   ; set a 3 seconds timeout
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,timeout=3,etag=1,md5=1

多线程
**************

这个插件是完全线程安全的，因此考虑使用多线程来提高并发：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   http-socket-modifier1 = 25
   ; set a 3 seconds timeout
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,timeout=3,etag=1,md5=1
   master = true
   processes = 2
   threads = 8

这将会生成2个由master监控的进程，每个进程有8个线程，总共16个线程。

与Nginx组合
********************

这与其他插件没有什么不同：

.. code-block:: c

   location / {
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 25;
   }

这是确保设置 ``uwsgi_modifier1`` 值来保证所有的请求都被路由到GridFS。

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   gridfs-mount = server=ubuntu64.local,db=test,skip_slash=1,timeout=3,etag=1,md5=1
   master = true
   processes = 2
   threads = 8

'gridfs'内部路由动作
************************************

这个插件公开了一个'gridfs'动作，它简单返回一个项：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   route = ^/foo/(.+).jpg gridfs:server=192.168.173.17,db=test,itemname=$1.jpg

选项和request插件的选项是一样的，只有"itemname"是例外。它指定了GridFS db中的对象名。

注意事项
*********

* 如果你不指定一个服务器地址，就会假设使用127.0.0.1:27017。
* 在异步模式下使用这个插件并非官方支持的，但是也许能用。
* 如果你不明白为嘛请求并不提供你的GridFS项，那么考虑添加 ``--gridfs-debug`` 选项。它将会在uWSGI日志中打印被请求项。
