uWSGI告警子系统 (自1.3起) 
====================================

从1.3版本开始，uWSGI有了一个告警系统。这个子系统允许开发者/系统管理员通过各种通道“宣布”应用的特殊条件。例如，你也许想要通过一个全监听队列的Jabber/XMPP，或者harakiri条件收到通知。告警系统基于两个组件：事件监控器和事件操作。

事件监控器是等待特定条件（例如，文件描述符或者特定日志信息上的一个事件）的东东。

一旦条件为真，那么就会引发一个操作（例如，发送邮件）。

嵌入式事件监控器
***********************

事件监控器可通过插件添加，uWSGI核心包括以下事件监控器：

* ``log-alarm`` 在指定的正则表达式匹配到一个日志行的时候，触发一个告警
* ``alarm-fd`` 在指定的文件描述符准备好的时候，触发一个告警（这是这相当底层，并且是大多数告警插件的基础）
* ``alarm-backlog`` 在socket backlog队列满的时候，触发一个告警
* ``alarm-segfault`` (自1.9.9起) 在uWSGI出现段错误的时候触发一个告警
* ``alarm-cheap`` 为每个基于curl的告警使用主要的告警线程，而不是创建专用线程

定义告警
*****************

你可以定义无数个告警。每个告警都具有唯一的名字。

目前，在主要发布版本中，可用的告警操作如下：

.. parsed-literal::
   'cmd' - 运行一个命令，将日志行传递给标准输入(stdin)
   'signal' -  生成一个uWSGI信号
   'mule' - 发送日志行给一个mule
   'curl' - 传递一个日志行给一个curl url (支持http,https和smtp)
   'xmpp' - 通过XMPP/jabber发送日志行

要定义一个告警，使用选项 ``--alarm`` 。

.. parsed-literal::
   --alarm "<name> <plugin>:<opts>"

记住，只有当你在命令行定义告警的时候，才引用。

.. code-block:: ini
   
   [uwsgi]
   alarm = mailme cmd:mail -s 'uWSGI alarm' -a 'From: foobar@example.com' admin@example.com
   alarm = cachefull signal:17

这里，我们定义了两个告警： ``mailme`` 和 ``cachefull`` 。第一个调用 ``mail`` 二进制来发送日志行给一个邮件地址；第二个生成一个uWSGI信号。现在，我们需要添加规则以触发告警：

.. code-block:: ini
   
   [uwsgi]
   alarm = mailme cmd:mail -s 'uWSGI alarm' -a 'From: foobar@example.com' admin@example.com
   alarm = cachefull signal:17
   log-alarm = cachefull,mailme uWSGI listen queue of socket
   log-alarm = mailme HARAKIRI ON WORKER

log-alarm的语法如下

.. parsed-literal::
   --log-alarm "<name> <regexp>"


在我们前面的例子中，我们使用应用到日志行的正则表达式定义了两个条件。第一个在监听队列满的时候会触发这两个告警，而第二个在一个worker被harakiri灭掉（commit harakiri）的时候，只会调用'mailme'。

该死，这……这是我见过的最原始的东东……
*****************************************************

你也许是对的。但是如果你暂时丢掉你那本“如何成为一个有大量基友但是没啥钱的炫酷程序员”，那么你会意识到有了这么一个简单的系统，你可以做些什么。想要看个例子？

.. code-block:: ini
   
   [uwsgi]
   alarm = jabber xmpp:foobar@jabber.xxx;mysecretpassword;admin@jabber.xxx,admin2@jabber.xxx
   log-alarm = jabber ^TERRIBLE ALARM

现在，在你的应用中，仅需添加

.. code-block:: python

   print "TERRIBLE ALARM! The world exploded!!!"

来发送一个Jabber信息给 ``admin@jabber.xxx`` 和 ``admin2@jabber.xxx``
而无需添加任何显著开销到你的应用上 (因为告警是由master进程中一个或多个线程触发的，而跟worker无关)。

再看看另一个例子？

看看这个Rack中间件：

.. code-block:: rb

   class UploadCheck
     def initialize(app)
       @app = app       
     end                
   
     def call(env)
       if env['REQUEST_METHOD'] == 'POST' and env['PATH_INFO'] == '/upload'
         puts "TERRIBLE ALARM! An upload has been made!"
       end   
       @app.call(env)   
     end                
   end               


不受糟糕的规则之害
*************************

这样一个多功能的系统易受到许多丑陋的错误干扰，主要是无限循环。因此，尽量精心构造你的正则表达式。内嵌的反循环子系统应防止告警插件错误的产生日志行。这个系统并不完美，因此，请再三检查你的正则表达式。

如果你正在写一个插件，那么确保在你的日志消息前附加'[uwsgi-alarm'字符串。这样的行将会被跳过，直接传递给日志子系统。有一个方便的可用API函数： ``uwsgi_log_alarm()``.

log-alarm是如何工作的呢？
************************

启用log-alarm自动让uWSGI实例进入 :term:`log-master模式` ，委托日志写入到master中。master只在传递日志行给日志插件之前执行告警子系统。堵塞的告警插件应运行在一个线程中 (例如curl和xmpp这些)，而简单的告警插件 (例如signal和cmd)可以直接运行在master中。

可用插件及其语法
**********************************

cmd
^^^

运行一个shell命令，传递命令行给它的标准输入：

.. parsed-literal::
   cmd:<command>
 

signal
^^^^^^

引发一个uWSGI信号。

.. parsed-literal::
   signal:[signum]

.. 又见:: :doc:`Signals`

mule
^^^^

发送日志行给一个mule，等待消息。

.. parsed-literal::
   mule:[mule_id]

.. 又见:: :doc:`Mules`

curl
^^^^

发送日志行给可cURL的URL。这个告警插件默认情况下不编译，因此如果需要构建它，仅需运行：

.. parsed-literal::
   python uwsgiconfig.py --plugin plugins/alarm_curl

.. parsed-literal::
   curl:<url>[;opt1=val1;opt2=val2]

``url`` 是任意标准的cURL URL，而当前公开的选项如下

* "auth_pass"
* "auth_user"
* "conn_timeout"
* "mail_from"
* "mail_to"
* "method"
* "ssl"
* "subject"
* "timeout"
* "url"
* "ssl_insecure"

因此，要通过SMTP AUTH发送邮件：

.. code-block:: ini

   [uwsgi]
   plugins = alarm_curl
   alarm = test curl:smtp://mail.example.com;mail_to=admin@example.com;mail_from=uwsgi@example.com;auth_user=uwsgi;auth_pass=secret;subject=alarm from uWSGI !!!

或者，我们可以使用Gmail来发送告警：

.. code-block:: ini

   [uwsgi]
   plugins = alarm_curl
   alarm = gmail curl:smtps://smtp.gmail.com;mail_to=admin@example.com;auth_user=uwsgi@gmail.com;auth_pass=secret;subject=alarm from uWSGI !!!

或者将日志行 `PUT` 到由基本身份验证保护的HTTP服务器：

.. code-block:: ini

   [uwsgi]
   plugins = alarm_curl
   alarm = test2 curl:http://192.168.173.6:9191/argh;auth_user=topogigio;auth_pass=foobar

或者将日志行 `POST` 到一个有自生成SSL证书的HTTPS服务器。

.. code-block:: ini

   [uwsgi]
   plugins = alarm_curl
   alarm = test3 curl:https://192.168.173.6/argh;method=POST;ssl_insecure=true

xmpp
^^^^

可能是内建的那一堆插件中最有趣的一个了。你需要 ``libgloox`` 包来构建XMPP告警插件 (在Debian/Ubuntu上，运行 ``apt-get install gloox-dev``).

.. code-block:: shell

   python uwsgiconfig.py --plugin plugins/alarm_xmpp

.. code-block:: shell

   xmpp:<jid>;<password>;<recipients>

你可以将“,”当成分隔符来设置多个收件人。

.. code-block:: ini

   [uwsgi]
   plugins = alarm_xmpp
   alarm = jabber xmpp:app@example.it;secret1;foo1@foo.it,foo2@foo.it

一个仍然关于XMPP插件的更有趣的事情是，当你的应用死掉的时候，你会看到你的应用的Jabber账户。 :-)

一些XMPP服务器 (最明显的是OSX服务器) 要求你绑定到资源。你可以通过附加 ``/resource`` 到JID来这样做。

.. code-block:: ini

   [uwsgi]
   plugins = alarm_xmpp
   alarm = jabber xmpp:max@server.local/uWSGI;secret1;foo1@foo.it,foo2@foo.it

speech
^^^^^^

用于OSX的一个玩具插件，主要用于，卖弄uWSGI和Objective-C的集成。它只使用OSX语音合成器来“宣告”告警。

.. code-block:: shell

   python uwsgiconfig.py --plugin plugins/alarm_speech

.. code-block:: ini

   [uwsgi]
   plugins = alarm_speech
   http-socket = :8080
   alarm = say speech:
   log-alarm = say .*

打开你的扩音器，运行uWSGI，然后开始听……

airbrake
^^^^^^^^

从1.9.9开始，uWSGI包含了 ``--alarm-segfault`` 选项，在uWSGI段错误的适合引发告警。

 ``airbrake`` 插件可以用来发送段错误回溯给airbrake兼容的服务器。例如Airbrake自己，以及它的开源克隆errbit
(https://github.com/errbit/errbit)，Airbrake支持是实验性的，它也许不会在所有情况下都可用。

.. code-block:: ini

   plugins = airbrake
   alarm = errbit airbrake:http://errbit.domain.com/notifier_api/v2/notices;apikey=APIKEY;subject=uWSGI segfault
   alarm-segfault = errbit

注意，alarm-segfault不需要Airbrake插件。使用其他告警插件，也可以发送回溯。
