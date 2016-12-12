GlusterFS插件
====================

自uWSGI 1.9.15起可用

官方modifier1: 27

'glusterfs'插件允许你使用自GlusterFS 3.4起可用的glusterfs api，直接提供存储在glusterfs文件系统中的文件

这个方法 (与通过fuse或者nfs提供相比) 在性能和易于部署方面有许多优势。


第1步：安装glusterfs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

我们从官方渠道构建glusterfs，在3个节点上(192.168.173.1, 192.168.173.2, 192.168.173.3)，把它安装在/opt/glusterfs。

.. code-block:: sh

   ./configure --prefix=/opt/glusterfs
   make
   make install
   
现在，这样启动配置/控制守护进程：

.. code-block:: sh

   /opt/glusterfs/sbin/glusterd
   
从现在起，我们可以配置我们的集群

第2步：第一个集群
^^^^^^^^^^^^^^^^^^^^^^^^

运行控制客户端来访问glusterfs shell:

.. code-block:: sh

   /opt/glusterfs/sbin/gluster
   
第一步是“发现”其他节点：

.. code-block:: sh

   # do not run on node1 !!!
   peer probe 192.168.173.1
   # do not run on node2 !!!
   peer probe 192.168.173.2
   # do not run on node3 !!!
   peer probe 192.168.173.3

记住，你无需对运行着glusterfs控制台的机器的相同的地址运行"peer probe"。你必须在集群的每个节点上重复这个过程。

现在，我们可以创建一个副本卷 (每个节点上都必须存在/exports/brick001目录):

.. code-block:: sh

   volume create unbit001 replica 3 192.168.173.1:/exports/brick001 192.168.173.2:/exports/brick001 192.168.173.3:/exports/brick001
   
并启动它：

.. code-block:: sh

   volume start unbit001
   
现在，你应该可以挂载你的glusterfs文件系统，并开始在其中写文件了 (你可以使用nfs或者fuse)

第3步：uWSGI
^^^^^^^^^^^^

一个名为'glusterfs'的构建配置文件已经可以用了，所以你可以简单这样做：

.. code-block:: sh

   PKG_CONFIG_PATH=/opt/glusterfs/lib/pkgconfig/ UWSGI_PROFILE=glusterfs make
   
这个配置文件目前禁用了'matheval'支持，因为glusterfs库使用带相同函数前缀的bison/yacc (引发命名冲突)。


现在，你可以启动你的HTTP服务来快速提供glusterfs文件 (记住，不涉及任何nfs或者fuse):

.. code-block:: ini

   [uwsgi]
   ; bind on port 9090
   http-socket = :9090
   ; set the default modifier1 to the glusterfs one
   http-socket-modifier1 = 27
   ; mount our glusterfs filesystem
   glusterfs-mount = mountpoint=/,volume=unbit001,server=192.168.173.1:0
   ; spawn 30 threads
   threads = 30
   

高可用性
^^^^^^^^^^^^^^^^^

主要的GlusterFS卖点是高可用性。使用前面的设置，我们引入了一个带控制守护进程的SPOF。

'server'选项允许你指定多个控制守护进程 (尝试它们直到有一个响应)

.. code-block:: ini

   [uwsgi]
   ; bind on port 9090
   http-socket = :9090
   ; set the default modifier1 to the glusterfs one
   http-socket-modifier1 = 27
   ; mount our glusterfs filesystem
   glusterfs-mount = mountpoint=/,volume=unbit001,server=192.168.173.1:0;192.168.173.2:0;192.168.173.3:0
   ; spawn 30 threads
   threads = 30
   
'0'端口是glusterfs惯例，它意味着“默认端口” (一般是24007)。你可以指定任何你需要/想要的端口。

多个挂载点
^^^^^^^^^^^^^^^^^^^^

如果你的web服务器 (像nginx或者uWSGI http路由器) 能够设置协议变量 (像SCRIPT_NAME或者UWSGI_APPID)，那么你可以在同一个实例中挂载多个glusterfs文件系统：

.. code-block:: ini

   [uwsgi]
   ; bind on port 9090
   http-socket = :9090
   ; set the default modifier1 to the glusterfs one
   http-socket-modifier1 = 27
   ; mount our glusterfs filesystem
   glusterfs-mount = mountpoint=/,volume=unbit001,server=192.168.173.1:0;192.168.173.2:0;192.168.173.3:0
   glusterfs-mount = mountpoint=/foo,volume=unbit002,server=192.168.173.1:0;192.168.173.2:0;192.168.173.3:0
   glusterfs-mount = mountpoint=/bar,volume=unbit003,server=192.168.173.1:0;192.168.173.2:0;192.168.173.3:0
   ; spawn 30 threads
   threads = 30
   
多进程 VS 多线程
^^^^^^^^^^^^^^^^^^^^^^^^^^^

目前，混合两者将会提供最佳性能和可用性。

异步支持还在进行中

内部路由
^^^^^^^^^^^^^^^^

 :doc:`InternalRouting` 让你可以重写请求来改变请求文件。目前，glusterfs插件只使用PATH_INFO，因此，你可以通过'setpathinfo'指令来改变它

也支持缓存。看看教程 (链接在主页上) ，获得一些很酷的点子吧


使用capabilities (在Linux上)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你的集群要求客户端绑定到特权端口 (<1024) ，而你不想要改动它 (而显然，你不想要作为root运行uWSGI)，那么你或许想要给予你的uWSGI实例NET_BIND_SERVICE capability。只需确保你有一个启用了capabilities的uWSGI，然后添加

.. code-block:: sh

   ... --cap net_bind_service ...
   
到所有你想要连接到glusterfs的实例

注意:
^^^^^^

该插件自动启用了mime type引擎。

无目录索引支持
