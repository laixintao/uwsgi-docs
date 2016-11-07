uWSGI项目
=================

uWSGI项目旨在为构建托管服务开发全栈。

使用通用的API和通用的配置风格来实现应用服务器 (对于各种编程语言和协议)，代理，进程管理器和监控器。

由于其可插拔架构，可以对其扩展以支持更多的平台和语言。

目前，你可以用C, C++和Objective-C来编写插件。

名字中的"WSGI"部分归功于同名Python标准，因为它是该项目第一个开发的插件。

通用性、高性能、低资源使用和可靠性是该项目的强项（也是唯一遵循的法则）。

包含的组件（更新至最新的稳定版本）
======================================================

核心 (实现配置、进程管理、socket创建、监控、日志记录、共享内存块、ipc、集群成员和 :doc:`SubscriptionServer`)

请求插件 (为各种语言和平台实现应用服务器接口：WSGI, PSGI, Rack, Lua WSAPI, CGI, PHP, Go ...)

网关 (实现负载均衡器、代理和路由器)

:doc:`Emperor <Emperor>` (实现大量实例管理和监控)

循环引擎 (实现事件和并发，组件可以在reforking, threaded, asynchronous/evented和green thread/coroutine模式下运行。支持多种技术，包括uGreen, Greenlet, Stackless, :doc:`Gevent <Gevent>`, Coro::AnyEvent, :doc:`Tornado <Tornado>`, Goroutines和Fibers)

.. note::

  uWSGI是一个具有快速发布周期的非常活跃的项目。出于这个原因，代码和文档可能不总是同步。我们试着尽最大的努力维护好文档，但这是一个艰苦的工作。为此表示歉意。如果你碰到问题了，那么邮件列表是寻求有关uWSGI的帮助的最佳来源。欢迎文档（除代码之外）贡献者。


快速入门
===========

.. toctree::
   :maxdepth: 1

   WSGIquickstart
   PSGIquickstart
   RackQuickstart
   Snippets


目录
=================

.. toctree::
   :maxdepth: 1

   Download
   Install
   BuildSystem
   Management
   LanguagesAndPlatforms
   SupportedPlatforms
   WebServers
   FAQ
   ThingsToKnow
   Configuration
   FallbackConfig
   ConfigLogic
   Options
   CustomOptions
   ParsingOrder
   Vars
   Protocol
   AttachingDaemons
   MasterFIFO
   Inetd
   Upstart
   Systemd
   Circus
   Embed
   Logging
   LogFormat
   LogEncoders
   Hooks
   WorkerOverride
   Glossary
   ThirdPartyPlugins

教程
=========

.. toctree::
   :maxdepth: 1

   tutorials/CachingCookbook
   tutorials/Django_and_nginx
   tutorials/dreamhost
   tutorials/heroku_python
   tutorials/heroku_ruby
   tutorials/ReliableFuse
   tutorials/DynamicProxying
   tutorials/GraphiteAndMetrics


文章
========

.. toctree::
   :maxdepth: 1

   articles/SerializingAccept
   #articles/MassiveHostingWithEmperorAndNamespaces
   articles/TheArtOfGracefulReloading
   articles/FunWithPerlEyetoyRaspberrypi
   articles/OffloadingWebsocketsAndSSE



uWSGI子系统
================

.. toctree::
   :maxdepth: 1

   AlarmSubsystem
   Caching
   WebCaching
   Cron
   Fastrouter
   InternalRouting
   Legion
   Locks
   Mules
   OffloadSubsystem
   Queue
   RPC
   SharedArea
   Signals
   Spooler
   SubscriptionServer
   StaticFiles
   SNI
   GeoIP
   Transformations
   WebSockets
   Metrics
   Chunked

使用uWSGI进行扩大化
==================

.. toctree::
   :maxdepth: 1

   Cheaper
   Emperor
   Broodlord
   Zerg
   DynamicApps
   SSLScaling

安全的uWSGI
==============

.. toctree::
   :maxdepth: 1

   Capabilities
   Cgroups
   KSM
   Namespaces
   FreeBSDJails
   ForkptyRouter
   TunTapRouter


密切关注你的应用
===========================

.. toctree::
   :maxdepth: 1

   Nagios
   SNMP
   PushingStats
   Carbon
   StatsServer
   Metrics


异步和循环引擎
======================

.. toctree::
   :maxdepth: 1

   Async
   Gevent
   Tornado
   uGreen
   asyncio




Web服务器支持
==================

.. toctree::
   :maxdepth: 1

   Apache
   Cherokee
   HTTP
   HTTPS
   SPDY
   Lighttpd
   Mongrel2
   Nginx
   OpenBSDhttpd


语言支持
==================

.. toctree::
   :maxdepth: 2

   Python
   PyPy
   PHP
   Perl
   Ruby
   Lua
   JVM
   Mono
   CGI
   GCCGO
   Symcall
   XSLT
   SSI
   V8
   GridFS
   GlusterFS
   Rados

其他插件
=============

.. toctree::
   :maxdepth: 1

   Pty
   SPNEGO
   LDAP


中断/弃用特性
==========================

.. toctree::
   :maxdepth: 1

   Erlang
   ManagementFlag
   Go


发布说明
=============

稳定版本
---------------

.. toctree::
   :maxdepth: 1

   Changelog-2.0.13.1
   Changelog-2.0.13
   Changelog-2.0.12
   Changelog-2.0.11.2
   Changelog-2.0.11.1
   Changelog-2.0.11
   Changelog-2.0.10
   Changelog-2.0.9
   Changelog-2.0.8
   Changelog-2.0.7
   Changelog-2.0.6
   Changelog-2.0.5
   Changelog-2.0.4
   Changelog-2.0.3
   Changelog-2.0.2
   Changelog-2.0.1
   Changelog-2.0
   Changelog-1.9.21
   Changelog-1.9.20
   Changelog-1.9.19
   Changelog-1.9.18
   Changelog-1.9.17
   Changelog-1.9.16
   Changelog-1.9.15
   Changelog-1.9.14
   Changelog-1.9.13
   Changelog-1.9.12
   Changelog-1.9.11
   Changelog-1.9.10
   Changelog-1.9.9
   Changelog-1.9.8
   Changelog-1.9.7
   Changelog-1.9.6
   Changelog-1.9.5
   Changelog-1.9.4
   Changelog-1.9.3
   Changelog-1.9.2
   Changelog-1.9.1
   Changelog-1.9



联系
=======

================== =
邮件列表       http://lists.unbit.it/cgi-bin/mailman/listinfo/uwsgi
Gmane镜像       http://dir.gmane.org/gmane.comp.python.wsgi.uwsgi.general
IRC                #uwsgi @ irc.freenode.org. 该channel的所有者是 `unbit`.
Twitter            http://twitter.com/unbit
商业支持 http://unbit.com/
================== =

.

商业支持
==================

你可以从http://unbit.com购买商业支持。

捐赠
======

uWSGI的发展是由意大利ISP `Unbit <http://unbit.it/>`_ 及其客户赞助的。你可以购买商业支持和许可。如果你不是一个Unbit客户，或者你不能/不想要购买一个商业uWSGI证书，那么可以考虑进行捐赠。当然，请在你的捐赠中随意要求新特性。

我们将信任任何想要赞助新特性的人。

访问该捐赠链接 http://unbit.it/uwsgi_donate 。你也可以通过 `GitTip <https://www.gittip.com/unbit/>`_ 来捐赠。

索引和表
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
