与Graphite/Carbon集成
================================

`Graphite <http://graphite.wikidot.com/>`_ 是一个强大的实时图形应用，构建于3个组件之上： 

- Whisper -- 一个数据存储系统
- Carbon -- 一个用来接收数据的服务器
- 用于图形渲染和管理的Python web应用。

uWSGI Carbon插件允许你发送uWSGI的内部统计数据到一个或多个Carbon服务器上。自uWSGI 1.0起，默认编译它，尽管它也可以作为插件构建。

快速开始
----------

为了说明起见，假设你的Carbon服务器正监听
``127.0.0.1:2003`` ，而你的uWSGI实例位于机器 ``debian32`` 之上，有4个进程监听 ``127.0.0.1:3031`` 。通过添加 ``--carbon`` 选项到你的uWSGI中，你将指示它定时发送它的统计数据到Carbon服务器。默认的周期是60秒。

.. code-block:: sh

    uwsgi --socket 127.0.0.1:3031 --carbon 127.0.0.1:2003 --processes 4 

度量的命名类似于 ``uwsgi.<hostname>.<id>.requests`` 和
``uwsgi.<hostname>.<id>.worker<n>.requests`` ，其中：

* ``hostname`` -- 机器的主机名
* ``id`` -- 第一个uWSGI socket的名字 (会用下划线来替换点)
* ``n`` -- worker进程的数目 (基于1)。

uWSGI生成的Carbon度量的名字举例：

* ``uwsgi.debian32.127_0_0_1:3031.requests``
  (``uwsgi.<hostname>.<id>.requests``)
* ``uwsgi.debian32.127_0_0_1:3031.worker1.requests``
  (``uwsgi.<hostname>.<id>.worker<n>.requests``)
* ``uwsgi.debian32.127_0_0_1:3031.worker2.requests``
  (``uwsgi.<hostname>.<id>.worker<n>.requests``)
* ``uwsgi.debian32.127_0_0_1:3031.worker3.requests``
  (``uwsgi.<hostname>.<id>.worker<n>.requests``)
* ``uwsgi.debian32.127_0_0_1:3031.worker4.requests``
  (``uwsgi.<hostname>.<id>.worker<n>.requests``).

.. seealso:: :doc:`tutorials/GraphiteAndMetrics`

