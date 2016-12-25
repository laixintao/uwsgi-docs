管理外部守护进程/服务
==================================

uWSGI可以轻松地监控外部进程，允许你提高你的多层应用的可靠性和可用性。例如，你可以管理诸如Memcached, Redis, Celery, Ruby delayed_job, 甚至是专门的PostgreSQL实例。

各种各样的服务
*****************

目前，uWSGI支持三种类型的进程：

* ``--attach-daemon`` -- 直接附加到非守护式进程
* ``--smart-attach-daemon`` -- 管理pid文件 (同时包括前台和守护)
* ``--smart-attach-daemon2`` -- 带守护管理的管理pid文件

第一类允许你直接附加进程到uWSGI master。当master死掉或者重载时，会销毁这些进程。这对于那些每当重启应用时必须被清除的服务来说，是最好的选项。

管理pid文件进程可以在死亡或者master的重载下存活下来，只要它们的pid文件可用，并且包含的pid匹配到一个运行中的pid。这对那些要求更长的持久性的进程，以及粗鲁的杀死可能意味着诸如数据库数据的遗失的情况来说，这是最好的选择。

最后一个目录是第二个的一个超集。如果你的进程不支持守护，或者写入到一个pid文件中，那么你可以让master来管理。非常少的守护进程／应用需要这个特性，但是对于微小的原型应用，或者只是设计不佳的应用来说，会是有用的。

自uWSGI 2.0起，第四个选项， ``--attach-daemon2`` 已被添加来进行高级的配置（见下）。

例子
********

在'dumb'模式下管理一个 **memcached** 实例。每当停止或者重载uWSGI，就会销毁memcached。

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   attach-daemon = memcached -p 11311 -u roberto

在'smart'模式下管理一个 **memcached** 实例。memcached会在uWSGI停止和重载下存活。

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   smart-attach-daemon = /tmp/memcached.pid memcached -p 11311 -d -P /tmp/memcached.pid -u roberto

在smart模式下管理2个 **mongodb** 实例：

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   smart-attach-daemon = /tmp/mongo1.pid mongod --pidfilepath /tmp/mongo1.pid --dbpath foo1 --port 50001
   smart-attach-daemon = /tmp/mongo2.pid mongod --pidfilepath /tmp/mongo2.pid --dbpath foo2 --port 50002

管理 **PostgreSQL** 专用实例 (/db/foobar1中的集群):

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   smart-attach-daemon = /db/foobar1/postmaster.pid /usr/lib/postgresql/9.1/bin/postgres -D /db/foobar1

管理 **celery**:

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   smart-attach-daemon = /tmp/celery.pid celery -A tasks worker --pidfile=/tmp/celery.pid

管理 **delayed_job**:

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   env = RAILS_ENV=production
   rbrequire = bundler/setup
   rack = config.ru
   chdir = /var/apps/foobar
   smart-attach-daemon = %(chdir)/tmp/pids/delayed_job.pid %(chdir)/script/delayed_job start

管理 **dropbear**:


.. code-block:: ini

   [uwsgi]
   namespace = /ns/001/:testns
   namespace-keep-mount = /dev/pts
   socket = :3031
   exec-as-root = chown -R www-data /etc/dropbear
   attach-daemon = /usr/sbin/dropbear -j -k -p 1022 -E -F -I 300

当使用namespace选项时，你可以附加一个dropbear守护进程，以允许直接访问指定名字空间内的系统。这要求
*/dev/pts* 文件系统被挂载在该名字空间内，并且你的worker运行时使用的用户可以访问该名字空间内的 */etc/dropbear* 目录。

Legion支持
**************

自uWSGI 1.9.9起，使用 :doc:`Legion` 子系统来进行守护进程管理成为了可能。Legion守护将只在legion lord节点上执行，因此在每个legion中，将总是只有单个守护实例运行。一旦lord死掉，就会在另一个节点上生成守护进程。要添加一个legion守护，请使用--legion-attach-daemon, --legion-smart-attach-daemon和
--legion-smart-attach-daemon2选项，它们拥有与普通的守护进程选项相同的语法。不同在于，需要添加legion名作为第一个参数。

例子：

管理 **celery beat**:

.. code-block:: ini

   [uwsgi]
   master = true
   socket = :3031
   legion-mcast = mylegion 225.1.1.1:9191 90 bf-cbc:mysecret
   legion-smart-attach-daemon = mylegion /tmp/celery-beat.pid celery beat --pidfile=/tmp/celery-beat.pid
   
   
--attach-daemon2
****************

已在uWSGI 2.0中添加这个选项，以允许高级配置。它是一个键值选项，接收以下键：

* ``command``/``cmd``/``exec``: 要执行的命令行
* ``freq``: 在认为一个守护进程“损坏”之前的最大尝试次数
* ``pidfile``: 要检查的pid文件 (启用smart模式)
* ``control``: 如果设置了，那么守护进程则成为一个‘控制’者：如果它死掉了，整个uWSGI实例都会挂掉
* ``daemonize``/``daemon``: 守护进程 (启用smart2模式)
* ``touch`` 用以检查的分号分隔带文件列表：每当‘碰了’它们，就会重启守护进程
* ``stopsignal``/``stop_signal``: 当停止uWSGI时，发送给守护进程的信号数
* ``reloadsignal``/``reload_signal``: 当重载uWSGI时，发送给守护进程的信号数
* ``stdin``: 如果设置了，文件描述符0将不会重新映射到/dev/null
* ``uid``: 移除特权到指定uid (要求master作为root运行)
* ``gid``: 移除特权到指定gid (要求master作为root运行)
* ``ns_pid``: 在新的pid名字空间内生成进程 (要求master作为root运行，仅限Linux)
* ``chdir``: 运行命令之前，chdir()到指定的目录 (在uWSGI 2.0.6中添加)


例如：

.. code-block:: ini

   [uwsgi]
   attach-daemon2 = cmd=my_daemon.sh,pidfile=/tmp/my.pid,uid=33,gid=33,stopsignal=3

