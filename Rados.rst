RADOS插件
====================

自uWSGI 1.9.16起可用，自uWSGI 2.0.6起稳定

官方modifier1: 28

作者：Javier Guerra, Marcin Deranek, Roberto De Ioris, Sokolov Yura 又名funny_falcon

'rados'插件让你可以使用librados API，直接提供存储在一个Ceph集群中的对象。

注意，不是CephFS文件系统，也不是'radosgw' S3/Swift兼容层；RADOS是裸对象存储层。


第1步：Ceph集群和内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你想尝试最小Ceph安装，那么你可以遵循这个指导：http://ceph.com/docs/master/start/。注意，你只需要OSD和MON守护进程，MDS只在CephFS文件系统的时候是必须的。

一旦运行了它，你应该有一个配置文件 (默认在/etc/ceph/ceph.con上)，并且应该能够使用 `rados` 功能。

.. code-block:: sh

   rados lspools

默认情况下，你应该至少拥有'data', 'metadata'和'rbd'池。现在添加一些内容到'data'池。例如，如果你有一个'list.html'文件和子目录'imgs/'下的图片'first.jpeg', 'second.jpeg'：

.. code-block:: sh

   rados -p data put list.html list.html
   rados -p data put imgs/first.jpeg imgs/first.jpeg
   rados -p data put imgs/second.jpeg imgs/second.jpeg
   rados -p data ls -

注意，RADOS没有目录的概念，但是对象名可以包含斜线。


第2步：uWSGI
^^^^^^^^^^^^

有一个名为'rados'的构建配置文件可以用，因此你可以简单这样做：

.. code-block:: sh

   make PROFILE=rados
   
或者

.. code-block:: sh

   python uwsgiconfig.py --build rados
   
或者使用安装程序

.. code-block:: sh

   # this will create a binary called /tmp/radosuwsgi that you will use instead of 'uwsgi'
   curl http://uwsgi.it/install | bash -s rados /tmp/radosuwsgi

显然，你可以将rados支持当成插件构建

.. code-block:: sh

   uwsgi --build-plugin plugins/rados/

或者旧式的：

.. code-block:: sh

   python uwsgiconfig.py --plugin plugins/rados/

现在，你可以启动一个HTTP服务器来提供RADOS对象服务了：

.. code-block:: ini

   [uwsgi]
   ; bind on port 9090
   http-socket = :9090
   ; set the default modifier1 to the rados one
   http-socket-modifier1 = 28
   ; mount our rados pool
   rados-mount = mountpoint=/rad/,pool=data,config=/etc/ceph/ceph.conf
   ; spawn 30 threads
   threads = 30

'rados-mount'参数接收多个子参数：

 - mountpoint: 必需，RADOS对象将会出现的URL前缀。
 - pool: 必需，提供服务的RADOS池。
 - config: 可选，ceph配置文件的路径。
 - timeout: 可选，设置操作的超时时间，以秒为单位
 - allow_put: 允许调用 ``PUT`` HTTP方法来存储新的对象
 - allow_delete: 允许调用 ``DELETE`` HTTP方法来移除对象
 - allow_mkcol: 允许调用 ``MKCOL`` HTTP方法来创建新的池
 - allow_propfind: (要求版本uWSGI 2.1) 允许调用WebDAV ``PROPFIND`` 方法
 - buffer_size:  ``GET`` 请求的最大缓冲大小，以字节为单位 (最小8192，最大的16777216，默认是131072)
 - put_buffer_size:  ``PUT`` 请求的最大缓冲大小 (默认为buffer_size)

在这个例子中，你的内容地址将会是http://localhost:9090/rad/list.html, http://localhost:9090/rad/imgs/first.jpeg和http://localhost:9090/rad/imgs/second.jpeg。


高可用性
^^^^^^^^^^^^^^^^^

RADOS存储系统是全分布的，只要使用相同的'ceph.conf'，在多个机器上启动几个uWSGI worker，那么所有的worker都能看到同一个池。如果它们都在相同的挂载点上提供服务，那么你会得到一个抗故障的RADOS-HTTP网关。


多挂载点
^^^^^^^^^^^^^^^^^^^^

你可以声明多个'rados-mount'项，每一个会定义一个新的挂载点。通过这种方式，你可以在不同的URL上公开不同的RADOS池。

HTTP方法
^^^^^^^^^^^^

支持以下方法：

* GET -> 检索资源
* HEAD -> 和GET一样，但是没有请求体
* OPTIONS -> (要求版本uWSGI 2.1) 返回允许的HTTP方法和WebDAV支持的列表
* PUT -> 要求mountpoint选项中有allow_put，存储资源到ceph：curl -T /etc/services http://localhost:8080/services
* MKCOL -> 要求mountpoint选项中有allow_mkcol，创建一个新的池：curl -X MKCOL http://localhost:8080/anewpool (将会创建池'anewpool')
* DELETE -> 要求mountpoint选项中有allow_delete，移除一个对象
* PROPFIND -> 要求mountpoint选项中有allow_propfind (uWSGI 2.1+)，实现WebDAV PROPFIND方法

特性
^^^^^^^^

* 支持多进程
* 异步支持功能齐全，ugreen挂起引擎是唯一支持的引擎：


.. code-block:: ini

   [uwsgi]
   ; bind on port 9090
   http-socket = :9090
   ; set the default modifier1 to the rados one
   http-socket-modifier1 = 28
   ; mount our rados pool
   rados-mount = mountpoint=/rad/,pool=data,config=/etc/ceph/ceph.conf
   ; spawn 1000 async cores
   async = 1000
   ; required !!!
   ugreen = true

缓存样例
^^^^^^^^^^^^^^^

强烈建议使用缓存来改进性能和减少Ceph集群上的负载。这是一个好例子：

.. code-block:: ini

   [uwsgi]
   ; create a bitmap cache with max 1000 items storable in 10000 4k blocks
   cache2 = name=radoscache,items=1000,blocks=10000,blocksize=4096,bitmap=1
   
   ; check every object ending with .html in the 'radoscache' cache
   route = \.html$ cache:key=${PATH_INFO},name=radoscache,content_type=text/html
   ; if not found, store it at the end of the request for 3600 seconds (this will automatically enable Expires header)
   route = \.html$ cachestore:key=${PATH_INFO},name=radoscache,expires=3600
   
   ; general options
   
   ; master is always a good idea
   master = true
   ; bind on http port 9090 (better to use a uwsgi socket behind a proxy like nginx)
   http-socket = :9090
   ; set the default modifier1 to the rados one
   http-socket-modifier1 = 28
   ; mount our rados 'htmlpages' pool
   rados-mount = mountpoint=/,pool=htmlpages
   
   ; spawn multiple processes and threads
   processes = 4
   threads = 8

要测试缓存行为，诸如uwsgicachetop (https://pypi.python.org/pypi/uwsgicachetop) 这样的工具将非常有用。

更多关于缓存的信息在这里： :doc:`CachingCookbook`

安全注意事项
^^^^^^^^^^^^^

启用MKCOL, PUT和DELETE可能会有很高的安全风险。

将它们与内部路由框架结合，以添加鉴权/认证策略：

.. code-block:: ini

   [uwsgi]
   master = true
   ; bind on http port 9090 (better to use a uwsgi socket behind a proxy like nginx)
   http-socket = :9090
   ; set the default modifier1 to the rados one
   http-socket-modifier1 = 28
   ; mount our rados 'htmlpages' pool
   rados-mount = mountpoint=/,pool=htmlpages,allow_put=1,allow_mkcol=1
   
   ; spawn multiple processes and threads
   processes = 4
   threads = 8
   
   ; permit PUT only to authenticated 'foo' user
   route-if = equal:${REQUEST_METHOD};PUT basicauth:my secret area,foo:bar
   
   ; allow MKCOL only from 127.0.0.1
   route-if = equal:${REQUEST_METHOD};MKCOL goto:check_localhost
   ; end of the chain
   route-run = last:
   
   route-label = check_localhost
   ; if REMOTE_ADDR = 127.0.0.1 -> continue to rados plugin
   route-remote-addr = ^127\.0\.0\.1$ continue:
   ; otherwise break with 403
   route-run = break:403 Forbidden
   


注意事项
^^^^^^^^^^

* 这个插件自动启用MIME type引擎。
* 无目录索引支持。在rados/ceph上下文中，它并无意义。
* 你应该在你的uWSGI实例中移除特权，因此，确保你把正确的权限给予了ceph keyring。
* 如果你用它来获取/存储大对象，那么考虑提高 ``buffer_size`` 。4194304是个非常高性能的值，如果你想节约内存，那么1048576也不错。
* 支持放入Erasure编码的池。会自动调整 ``put_buffer_size`` 来满足池对齐要求。
