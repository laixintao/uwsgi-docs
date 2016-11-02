uWSGI Emperor —— 多应用部署
=========================================

如果你需要在单个服务器，或者一组服务器上部署大量的应用，那么Emperor模式就是你的最佳选择。它是一个特殊的uWSGI实例，会监控特定的事件，并且会按需生成/停止/重载实例 (当被一个Emperor管理的时候，被称为 :term:`vassals<vassal>`)。

默认情况下，Emperor将会扫描特定目录搜索支持的(.ini,
.xml, .yml, .json, 等等) uWSGI配置文件，但可以使用
:term:`imperial monitor` 插件来对其进行扩展。 ``dir://`` 和 ``glob://`` 插件是嵌入到核心中的，因此，不需要加载它们，它们会被自动检测到。 ``dir://`` 是默认插件。

* 每当一个imperial监控器检测到一个新的配置文件的时候，将会根据该配置文件生成一个新的uWSGI实例。
* 每当一个配置文件被修改的时候 (它的修改时间发生了改变，因此 ``touch --no-dereference`` 或许会是你的好伙伴)，就会重载相应的应用。
* 每当一个配置文件被删除的时候，就会停止相应的应用。
* 如果emperor死掉了，那么所有的vassal都会死掉。
* 如果出于某种原因，一个vassal死掉了，那么emperor将会重新生成它。

可以通过多次指定 ``--emperor`` 来监控配置的多个来源。

.. seealso::

  见 :doc:`ImperialMonitors` 以获得uWSGI出厂的Imperial Monitor插件列表，以及如何使用它们。

.. toctree::

  ImperialMonitors
  EmperorProtocol
  OnDemandVassals

特殊配置变量
-------------------------------

将 :ref:`Placeholders` 与 :ref:`MagicVars` 和Emperor一起使用可能会节省你大量的时间，并且让你的配置更DRY。假设在/opt/apps中，只有 Django_ 应用。/opt/apps/app.skel (
.skel扩展名并不是uWSGI已知的配置文件类型，因此将会被跳过)

.. code-block:: ini

  [uwsgi]
  chdir = /opt/apps/%n
  threads = 20
  socket = /tmp/sockets/%n.sock
  env = DJANGO_SETTINGS_MODULE=%n.settings
  module = django.core.handlers.wsgi:WSGIHandler()

然后，为每个应用，创建一个符号链接::

  ln -s /opt/apps/app.skel /opt/apps/app1.ini
  ln -s /opt/apps/app.skel /opt/apps/app2.ini

最后，使用 ``--emperor-nofollow`` 选项启动Emperor。现在，你可以使用以下命令来分别重载每个vassal了::

  touch --no-dereference $INI_FILE

.. _Django: http://djangoproject.com

传递配置参数给所有的vassal
-----------------------------------------------

从1.9.19开始，你可以使用 ``--vassal-set`` 来传递选项了

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   vassal-set = processes=8
   vassal-set = enable-metrics=1
   
这将会添加 ``--set processes=8`` 和 ``--set enable-metrics=1`` 到每个vassal中

你还可以强制Emperor使用环境变量传递选项给uWSGI实例。每个形式为  ``UWSGI_VASSAL_xxx`` 的环境变量将会在新的实例中重写为 ``UWSGI_xxx`` ，并带有一般的
:ref:`configuration implications<ConfigEnv>`.  

例如::

  UWSGI_VASSAL_SOCKET=/tmp/%n.sock uwsgi --emperor /opt/apps

会让你避免在配置文件中指定socket选项。

另外，你可以使用 ``--vassals-include`` 选项，让每个
vassal自动包含一个完整的配置文件::

  uwsgi --emperor /opt/apps --vassals-include /etc/uwsgi/vassals-default.ini

注意，如果你这样做，被包含的文件中的 ``%n`` (及其他魔术变量) 将被解析为被包含的文件的名字，而不是原始的vassal配置文件的名字。如果你想要使用vassal名在被包含的文件中设置选项，那么你必须使用占位符。例如，在vassal配置中，这样写::

  [uwsgi]
  vassal_name = %n
  ... more options

在 ``vassal-defaults.ini`` 中，这样写::

  [uwsgi]
  socket = /tmp/sockets/%(vassal_name).sock

.. _Tyrant:

Tyrant模式 (安全的多用户托管)
---------------------------------------

emperor通常以root运行，在每个实例的配置中设置UID和GID。然后，vassal实例在处理请求之前删除特权。在这个模式下，如果你的用户可以访问自己的uWSGI配置文件，那么你不能信任他们会设置正确的 ``uid`` 和
``gid`` 。你可以以非特权用户运行emperor (使用 ``uid`` 和
``gid``)，但是所有的vassal之后将会运行在相同的用户之下，因因为非特权用户不能自己切换成其他用户。对于这种情况，可以使用Tyrant模式 —— 仅需添加 ``emperor-tyrant`` 选项。

在Tyrant模式下，Emperor会使用vassal配置文件（或者对于其他Imperial监控器，通过一些配置的其他方法）的UID/GID来运行vassal。如果使用了Tyrant模式，那么vassal配置文件必须是UID/GID > 0。如果UID或者GID为0，或者一个已经在运行的vassal的配置的UID或GID发生了变化，那么将会出现一个错误。


适用于偏执型系统管理员的Tyrant模式 (仅Linux)
***********************************************

如果你构建了一个启用了 :doc:`Capabilities` 选项的uWSGI版本，那么你可以以非特权用户运行Emperor，但是维持应用到tyrant模式所需的最小的root能力。

.. code-block:: ini

   [uwsgi]
   uid = 10000
   gid = 10000
   emperor = /tmp
   emperor-tyrant = true
   cap = setgid,setuid


Loyalty
-------

一旦一个vassal管理了一个请求，那么它将变得“loyal”。Emperor使用这个状态来识别行为不良的vassal并惩罚它们。

节流
----------

每当在同一秒中生成了两个或以上的vassal，Emperor就会启动一个节流子系统来避免 `fork bombing`_ 。每当这种情况发生的时候，该系统就会增加一个节流差 (通过 :ref:`OptionEmperorThrottle` 选项指定，以毫秒为单位)，然后在生成一个新的vassal之前等待该节流差指定的时间。每当一个新的vassal的生成不触发节流的时候，当前的节流持续时间就会减半。

.. _fork bombing: http://en.wikipedia.org/wiki/Fork_bomb

黑名单系统
----------------

每当一个不loyal的vassal死掉的时候，它就会被放在一个耻辱黑名单中。当vassal在黑名单的时候，它就会被节流至一个最大值 (可以通过
:ref:`OptionEmperorMaxThrottle` 调整)，从默认的节流差3开始。每当一个位于黑名单中的vassal死掉的时候，它的节流值就会增加节流差指定的数值 (:ref:`OptionEmperorThrottle`)。

你也可以通过发送信号SIGURG给emperor进程来清空黑名单。这将会重置节流值。

心跳系统
----------------

vassal可以自愿让Emperor监控它们的状态。启用了心跳的vassal的worker会发送"heartbeat"消息给Emperor。如果超过N(默认
30, :ref:`OptionEmperorRequiredHeartbeat`)秒后，
Emperor都没有接收到来自一个实例的心跳，那么就会认为那个实例挂起，因此会对它进行重载。要在一个vassal中启动发送心跳包，则添加 :ref:`OptionHeartbeat` 选项。

.. important::

  如果你所有的worker都卡在处理完全合法的请求，例如缓慢的大文件上传，那么Emperor将会触发重载，仿佛worker都挂起了。触发的重载是优雅地重载，因此你可以为理智的行为调整你的配置/超时时间/容忍度。

.. TODO: Clarify the above admonition

将Linux命名空间用于vassal
----------------------------------

在Linux上，你可以告诉Emperor在“非共享”上下文中运行vassal。那意味着你可以使用文件系统、ipc、uts、网络、pid和uid的专用视图来运行每个vassal。

你通常使用诸如 ``lxc`` 或者它的抽象，例如 ``docker`` ，来做的事在uWSGI中都是自然而然的。

例如，如果你想在一个新的名字空间内运行每个vassal：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = fs,net,ipc,pid,uts
   
现在，每个vassal将能够修改文件系统布局，网络，主机名等等等等，而不会损坏主系统。

uWSGI发布版本中包含了几个辅助守护进程，用来简化监禁vassal的管理。最值得注意的是， :doc:`TunTapRouter` 允许jail中的完整用户空间网络，而 ``forkpty router`` 允许jail中的虚拟终端分配。

无需在你的vassal中取消共享所有的子系统，有时候，你只想要把专用的ipc和主机名给一个vassal，并从进程列表中隐藏:

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-use-clone = fs,ipc,pid,uts
   
vassal可以是:

.. code-block:: ini

   [uwsgi]
   ; set the hostname
   exec-as-root = hostname foobar
   ; umount /proc and remount to hide processes
   ; as we are in the 'fs' namespace umounting /proc does not interfere with the main one
   exec-as-root = umount /proc
   exec-as-root = mount -t proc none /proc
   ; drop privileges
   uid = foobar
   gid = foobar
   ; bind to the socket
   socket = /tmp/myapp.socket
   psgi = myapp.pl


Imperial统计局
---------------------------------

你可以添加
:ref:`OptionEmperorStats` 选项，将用一个TCP地址作为该选项的值，来为Emperor启用统计/状态服务。通过连接到该地址，你将获得一个JSON格式的统计信息块。

.. _BinaryPatch:

将非uWSGI应用当成vassal运行或者把替代的uWSGI当成vassal使用
-------------------------------------------------------------

你可以使用 ``privileged-binary-patch``/``unprivileged-binary-patch`` 选项，把一个不同的二进制文件当成你的vassal ``exec()`` 。第一个选项在socket继承和共享socket初始化后对二进制文件打补丁（这样你就可以使用uWSGI定义的socket了）。第二个选项在特权删除后对二进制文件打补丁。用这种方式，你将能够使用uWSGI的UID/GID/chroot/namespace/jailing选项。使用与Emperor传递给vassal的相同的参数来调用该二进制文件。

.. code-block:: ini

  ; i am a special vassal calling a different binary in a new linux network namespace
  [uwsgi]
  uid = 1000
  gid = 1000
  unshare = net
  unprivileged-binary-patch = /usr/bin/myfunnyserver

.. important::

  *不要守护* 你的应用。如果你这样做，那么Emperor将会失去与它们的连接。

会给新的二进制文件传递uWSGI参数。如果你不喜欢这种行为 (或者需要传递自定义参数)，那么添加 ``-arg`` 到二进制补丁选项:

.. code-block:: ini

  ; i am a special vassal calling a different binary in a new linux network namespace
  ; with custom options
  [uwsgi]
  uid = 1000
  gid = 1000
  unshare = net
  unprivileged-binary-patch-arg = ps aux

或者

.. code-block:: ini

  ;nginx example
  [uwsgi]
  privileged-binary-patch-arg = nginx -g "daemon off;"

.. seealso::

  你自定义的vassal应用也可以使用 :doc:`emperor protocol <EmperorProtocol>` 与emperor通信。

将Emperor和FastRouter集成在一起
-------------------------------------------

FastRouter是一个使用 :doc:`Protocol` 的代理/负载均衡器/路由器。来自  `Lincoln Loop`_  的Yann
Malet发布了 `a draft about massive Emperor +
Fastrouter deployment`_ (PDF)，使用 :doc:`Caching` 作为socket映射存储的主机名。

.. _Lincoln Loop: http://lincolnloop.com/

.. _`a draft about massive Emperor + Fastrouter deployment`: http://projects.unbit.it/uwsgi/raw-attachment/wiki/Emperor/lincolnloop.pdf

注意
-----

* 启动时，emperor ``chdir()`` 到vassal目录。所有的vassal实例将从这里启动。
* 如果uwsgi二进制文件不在你的系统路径中，那么你可以使用 ``binary-path`` 来强制其路径::
    
    ./uwsgi --emperor /opt/apps --binary-path /opt/uwsgi/uwsgi

* 发送 ``SIGUSR1`` 到emperor将会在其日志中打印vassal状态。
* 停止 (``SIGINT``/``SIGTERM``/``SIGQUIT``) Emperor将会调用
  Ragnarok，并杀死所有vassal。
* 发送 ``SIGHUP`` 到Emperor将会重载所有的vassal。
* 发送 ``SIGURG`` 到Emperor将会把所有的vassal从黑名单中移除。
* 一般来讲，不应该带 ``--master`` 运行emperor，除非有特别需要的master特性，例如高级日志记录。
* 一般来讲，emperor应该在服务器启动时启动，并且保持不动，不重载/重启，除非uWSGI升级；emperor重载有点猛，会一次性重载所有的vassal。相反，vassal应该在需要的时候单独重载，以使用imperial监控器的方式。

待办事项
--------

* 文档待办事项：使用非文件系统监控器理清"启动时更改目录"行为做了什么。
* 导出更多魔术变量
* 增加对xml/ini/yaml文件中多部分的支持 (这将允许将单个配置文件用于多个实例中)
