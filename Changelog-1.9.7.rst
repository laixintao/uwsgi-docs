uWSGI 1.9.7
===========


错误修复
********

- 修复teajs引擎构建

- 修复卸载状态码 (当卸载请求的时候，设置为202)

- 在60秒分辨率内执行cron任务，而不是61秒

- 修复websocket代理

- 检查python3 unicode编码 (而不是崩溃……)

- 修复重载时的ipcsem移除

- 修复OpenBSD, NetBSD和DragonFlyBSD上的kqueue定时器

- 修复/重新实现perl的uwsgi::register_rpc

- 修复sendfile()错误时的fd泄漏

- 修复在使用gzip文件变量时的Content-Length

- 允许非请求插件注册rpc函数

- 对于cgroups更健壮的错误检查

- 当挂载多个perl应用的时候，遵循PSGI插件中更多SCRIPT_NAME


新特性
********


Legion cron
^^^^^^^^^^^

当应用的多个实例在运行的时候，一个共同的需求是强制只有其中一个实例运行cron任务。新的--legion-cron使用 :doc:`Legion` 来做到这点：

.. code-block:: ini

   [uwsgi]
   ; use the new legion-mcast shortcut (with a valor 90)
   legion-mcast = mylegion 225.1.1.1:9191 90 bf-cbc:mysecret
   ; run the script only if the instance is the lord of the legion "mylegion"
   legion-cron = mylegion -1 -1 -1 -1 -1 my_script.sh


Curl cron
^^^^^^^^^

添加了curl_cron插件，允许cron子系统调用url (通过libcurl) 而非unix命令：

.. code-block:: ini

   [uwsgi]
   ; call http://uwsgi.it every minute
   curl-cron = -1 -1 -1 -1 -1 http://uwsgi.it/

请求的输出会被记录在日志中

UWSGI_EMBED_PLUGINS构建变量
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在在构建阶段可以动态嵌入插件了。看看这个例子：

.. code-block:: sh

   UWSGI_EMBED_PLUGINS=gridfs,rack UWSGI_PROFILE=psgi make

这将会构建一个单片二进制文件，对psgi使用默认的配置文件，以及带有gridfs和rack插件 (都嵌入到二进制文件中)


Gzip缓存
^^^^^^^^^^^^

cachestore路由功能现在可以直接将项以gzip格式存储了。

看看CachingCookbook: https://uwsgi-docs.readthedocs.io/en/latest/tutorials/CachingCookbook.html

--skip-atexit
^^^^^^^^^^^^^

mongodb客户端库中的一个错误可能会导致uWSGI服务器在关机/重载期间崩溃。这个选项避免了调用atexit()钩子。如果你在构建一个 :doc:`GridFS` 基础设施，那么你也许想要使用这个选项，同时让MongoDB小伙伴解决问题。

proxyhttp和proxyuwsgi
^^^^^^^^^^^^^^^^^^^^^^^^

http和uwsgi路由指令现在更智能了。你可以缓存它们的输出，并且在日志中获取正确的状态码。

这要求你不要使用卸载。如果卸载就在那里，并且对于这两个路由器不想使用卸载，那么使用以proxy为前缀的变量，这将会跳过卸载。

现在，你可以做些很酷的事情了，例如：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   ; create a cache of 100 items
   cache = 100
   ; check if a cached value is available
   route-run = cache:key=${REQUEST_URI}
   ; proxy all request to http://unbit.it
   route-run = http:81.174.68.52:80,unbit.it
   ; and cache them for 5 minutes
   route-run = cachestore:key=${REQUEST_URI},expires=300

The transformation api
^^^^^^^^^^^^^^^^^^^^^^

已经添加了一个用于操作响应的通用API (cachestore使用它)

看看 :doc:`Transformations`

--alarm-fd
^^^^^^^^^^

我们正在改进 :doc:`AlarmSubsystem` ，让它更少依赖日志行。现在，当fd准备好读的时候，你可以触发告警了。

这对于与Linux的eventfd()功能集成是非常有用的。

例如，当你的cgroup正在运行OOM-Killer的时候，你可以监控（以及抛出一个异常）

.. code-block:: ini

   [uwsgi]
   ; define an 'outofmemory' alarm that simply print the alarm in the logs
   alarm = outofmemory log:
   ; raise the alarm (with the specified message) when fd is ready (this is an eventfd se we read 8 bytes from the fd)
   alarm-fd = outofmemory $(CGROUP_OOM_FD):8 OUT OF MEMORY !!!

在这个例子中，CGROUP_OOM_FD是一个环境变量，映射到从某些启动脚本继承过来的一个eventfd()文件描述符数目。或许 (在不久的将来)，我们会能够直接在uWSGI中定义这类型的监控。

更多关于eventfd() + cgroup集成在这里：https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt

一个样例perl启动脚本：

.. code-block:: pl

   use Linux::FD;
   use POSIX;

   my $foo = Linux::FD::Event->new(0);
   open OOM,'/sys/fs/cgroup/uwsgi/memory.oom_control';
   # we dup() the file as Linux::FD::Event set the CLOSE_ON_EXEC bit (why ???)
   $ENV{'CGROUP_OOM_FD'} = dup(fileno($foo)).'';

   open CONTROL,'>/sys/fs/cgroup/uwsgi/cgroup.event_control';
   print CONTROL fileno($foo).' '.fileno(OOM)."\n";
   close CONTROL;

   exec 'uwsgi','mem.ini';

默认编译的spooler服务器插件和cheaper busyness算法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在极端高负载场景中，busyness cheaper算法 (来自Łukasz Mierzwa) 在过去就是一个银弹，它允许基于实时使用时间，考虑性能和响应时间，自适应生成进程。出于这个原因，现在默认内置了这个插件。

除此之外，还在默认构建配置文件中添加了远程spooler插件 (允许外部进程排队作业) 。


可用性
************

uWSGI 1.9.7自20130422其可用，你可以在下面的url中找到它：

http://projects.unbit.it/downloads/uwsgi-1.9.7.tar.gz
