使用Emperor和Linux名字空间进行大规模“安全”托管，亦称"改进unbit.it和pythonanywhere.com"
===============================================================================================================

作者：Roberto De Ioris

*** 正在进行中 ***

免责声明
**********

在以下的介绍中，我会提到两家公司：Unbit和pythonanywhere.com。我在这两家工作 (实际上，我拥有第一家公司 :P)。

如果你觉得我是在为这两个公司打广告，好吧，你是对的。

简介
*****

自2005年起，我就作为意大利ISP Unbit (http://unbit.it)的首席系统管理员，以及全球各种托管公司的顾问。

Unbit是一个面向开发者的服务，我们允许托管基本任何你想要的东西，而不强制你使用VPS，只需尽情使用Linux内核设施 (它与眼下Heroku做的非常相似，但是早于Heroku 5年出现 ;)

在2009年，我们开始uWSGI项目，最初是作为一个WSGI服务器，然后，我们慢慢意识到，它的范式可以被应用到我们所有的基础设施上，因此，现在它正在成为各种语言的一种“托管平台”。我们计划到2014年，对于整个Unbit托管栈只使用uWSGI。

在你激动之前，我想告诉你，Unbit只接受意大利客户 (我们不是种族主义者，这是避免与我们工作的其他托管公司发生法律问题的政策) ，并且我们的价格相当高，因为我们不作任何形式的过度抛售 (而更重要的是，我们不提供免费账户 ;)

在超过8年间，我和我的同事经历了数千个问题 (是哒，如果你想要进入互联网服务市场，那么准备好奉献你绝大多数的时间来解决用户产生的问题，并且别想获得作为人的最小尊重 ;) 因此，你在整个uWSGI项目所见的，就是这些年的头疼与未眠之夜 (以及来自客户的羞辱)

在2013年的夏天，我跟pythonanywhere.com的小伙伴 (主要是和Harry Percival) 工作了一会。

他们大量地将uWSGI特性用于他们的服务，因此他们帮助我激发我脑中的新想法和解决方法。

在uWSGI 1.9.15，已合并了大量用于高级Linux名字空间使用的新补丁，这多亏了与pythonanywhere.com伙伴们的合作。

基于这两个公司的经验，这篇文章将会展示其中一种你可以按着来构建用于托管不可靠的web应用的服务的方法 (是哒，即使你拥有和平用户的最大集合，他们也必须被视为“不可靠”和“邪恶的”，否则，你就不是一个好的系统管理员)。

它不是一步一步的教程，但是是为你提供对你的需要改进和调整概念的一些基础的某种类型的菜谱。

我们想要允许我们的用户做什么
**********************************

- 部署WSGI,PSGI和RACK应用 (无CGI和php，虽然技术上可行，但是如果你想你可以通过php托管赚钱，那么你应该开始寻找第二份工作)
- 运行cron脚本
- 运行私有服务 (redis, beanstalkd, memcached...)
- 应用可以连接到互联网
- 多个域名可以映射到同一个实例

...以及，我们想要禁止什么
*****************************

- 用户不能看到同个机器上的其他账户的进程。他们的初始进程必须是uWSGI master
- 用户不能看到同个机器上的其他账户的文件
- 用户不能连接到同个机器上的其他账户的私有服务 (memcached, redis...)
- 用户不能读／写同个机器上的其他账户的ipc信号量，共享内存和消息队列
- 用户不能分配比他们付费的量更多的内存
- 用户不能使用比他们付费的量更多的CPU

操作系统
********************

web服务器
*************

由于我们无需担心php和.htaccess文件等滥用，因此可以选择我们想要的任何服务器。

我们更喜欢用nginx (即使我们 [Unbit] 正慢慢地移到uWSGI http/https/spdy 路由器，因为我们只需要一个带有动态路由的最小的代理，但是对于某些而言，更复杂的nginx就是正途)，但是你可以使用任何你喜欢的。

“控制面板”
*******************

这是你需要开发的东东，你的面板越可用和强大，你的用户将会越开心。

你的控制面板可能就是那种会让你的托管公司成功的东西。

你的控制面板的目标是生成"vassal文件" (见下)。vassal文件可以是.ini, xml, yaml和json (除非你又特别的理由要使用其他格式)。

vassal文件包含客户微系统的整个结构。一旦创建了一个vassal文件，就会部署它 (而当它被修改时，将会重载它)

uWSGI '语言'插件
************************

我们想要支持多种应用。较好的方法是有一个单一的uWSGI二进制文件，以及一系列的“语言插件” (一个用于一个你想要支持的语言)。

你可以支持同一个语言的多个版本。仅需构建对应的插件。

在Unbit，我们做了一个非常模块化的uWSGi发行版 (基本上所有都是插件)。这是必须的，因为我们考虑内存的任何MB，因此，我们允许用户只启用所需的特性，来尽可能的获得多内存。

如果你尚未是uWSGI管理的黑带，那么我建议你从包含'nolang'构建配置文件开始。

它将会构建一个不带任何内置语言的标准uwsgi二进制文件。

...

Lazy应用 VS prefork
********************

uWSGI的有争议的设计选择之一是“默认prefork”。

它意味着你的应用会在启动时加载，然后对每个worker调用fork()。

虽然这是UNIX世界的常见方法，并且它对Perl开发者来说是期望的行为
(历史上更接近UNIX世界)，但是对于Python（也许还有Ruby），它完全未知和未期望。

所以，当你在构建一个基于uWSGI的服务的时候，你需要做的选择之一就是如何管理fork()行为。

如果你不确定，那么让我告诉你一件事：使用prefork行为，你会让一些用户非常愉悦，以及完全丧失大量用户。使用--lazy-apps，你将让你所有的用户完全不在意。相信我，当你也有愤怒的用户的时候，几个愉悦的用户也不能让你开心。

因此，uWSGI默认的fork()行为对于大规模托管而言一般是错误的，因此，添加--lazy-apps，并最终给予高级用户需要时改变它的自由。


文件系统布局
*********************

发行版的升级始终是一个大屠杀。这是一个非常乐观的分析。相信我。

但“光阴似箭”，因此，迟早会有客户开始要求更新的包……

你可以更新，但是你会自动将你绝大多数的用户置于狂怒模式，因为非常有可能他们的应用将不再工作。

一个让所有人都开心的方法是在你的系统中拥有不同的发行版 (是哒，这听起来很傻，但是请继续阅读)。

Debbotstrap是个不错的工具。让我们在/distros文件夹下创建我们的发行版集合：

.. code-block:: sh

   debootstrap lucid /distros/lucid
   debootstrap etch /distros/etch
   debootstrap precise /distros/precise
   debootstrap saucy /distros/saucy
   ...
   
每个用户将能攻选择 (和变更) 它的发行版，因为多亏了我们的设置（见下），它的根文件系统将会是其中一个可用发行版的只读挂载。

最终的布局将是：

* / (rootfs, 只读映射到/distros的目录之一)
* /proc (展示进程和获取系统信息需要)
* /tmp (每个用户应该拥有一个专门的/tmp)
* /dev (应该至少包含zero和null，但是也可以绑定挂载到系统的/dev)
* /dev/pts (伪终端所需，由所有的vassal共享 [直到将发布linux的pts名字空间])
* /var/run (所有的socket都将绑定在这里，并且被主要的rootfs进行符号链接，以进行nginx和ssh访问)
* /opt (这可能是一个绑定挂载，由所有包含发行版独立文件的用户共享)


Linux名字空间
****************

这是限制用户的第一步。

对于这一步，我们将使用5个名字空间：文件系统，sysv ipc, uts, 网络和pid

文件系统 (fs)
^^^^^^^^^^^^^^^

这允许改变文件系统布局 (挂载点)。

我们将使用--pivot-root选项（Linux特有的）来代替在每个vassal中chroot()，它与挂载名字空间结合，允许文件系统布局的细粒度配置。

sysv ipc (ipc)
^^^^^^^^^^^^^^

sysv ipc公开了3个原语：信号量，共享内存和消息队列。

取消共享它会创建这3个特性的一个专用组

uts (uts)
^^^^^^^^^

这个名字空间允许你有一个专用的主机名

网络 (net)
^^^^^^^^^^^^^^^^

当你停止共享主要的网络名字空间，你将失去对接口地址的访问。将会分配一个新的回环。

进程 (pid)
^^^^^^^^^^^^^^^

这个名字空间允许你对用户隐藏不是用户名字空间本身的部分的进程。

uWSGI master进程将会是用户pid为1的进程。

Emperor使用名字空间
^^^^^^^^^^^^^^^^^^^^^^^

--emperor-use-clone选项允许Emperor在一个新的名字空间中直接生成vassal。

我们的配置将如下：

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-user-clone = fs,ipc,uts,net,pid
  
而每个vassal将是

.. code-block:: ini

   [uwsgi]
   ; set the hostname
   exec-as-root = hostname foobar
   ; bring up loopback
   exec-as-root = ifconfig lo up

Linux cgroups
*************

uWSGI Emperor 和 vassal
*************************

网络
**********

这可能是最复杂的部分。给予网络一个jail设置的“正统”方式是使用veth或者macvlan。

第一个是由两个虚拟接口组成的“网络管道”。在创建名字空间后，你将其中一个管道的尾部移到名字空间中。

相反，Macvlan通过将一个额外的mac地址赋给物理接口来工作。

这两种方法对于类VPS设置而言都不错，但是这里，我们需要网络只连接到外部服务 (入站连接是由http代理管理的)。

veth和macvlan方法都难以正确管理，而在1.9.15中，我们引入了大量的特性来简化所需步骤，在1.9.16中，我们决定创建一个基于tuntap设备的ad-hoc解决方法。

基本上，对于每个vassal，我们创建一个tun设备 (它是一个虚拟网络接口，可以通过用户空间来管理)，将其连接 (通过 unix socket) 到主名字空间中的另一个tun设备。

这个tuntap路由器是一个基于软件的ip路由器，它主要从tuntap设备获取包，然后将其转发给unix socket (反之亦然)。

这个方法极度简化了整个设置，并且，作为一个杀手级特性，一个超级简单的防火墙被嵌入到进程中，用来配置外部规则。

这个tuntap路由器应运行在Emperor中 (它是一个uWSGI网关，因此这次我们需要master进程):

.. code-block:: ini

   [uwsgi]
   emperor = /etc/uwsgi/vassals
   emperor-user-clone = fs,ipc,uts,net,pid
   master = true
   ; create the tun interface 'emperor0' reachable by /var/run/tuntap.socket
   tuntap-router = emperor0 /var/run/tuntap.socket
   ; give an internal ip address to 'emperor0'
   exec-as-root = ifconfig emperor0 192.168.0.1 netmask 255.255.255.0
   ; configure NAT for vassals
   exec-as-root = iptables -t nat -F
   exec-as-root = iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j MASQUERADE
   exec-as-root = echo 1 > /proc/sys/net/ipv4/ip_forward
   
   ; configure the internal firewall to disallow communication between vassals
   tuntap-router-firewall-out = allow 192.168.0.0/24 192.168.0.1
   tuntap-router-firewall-out = deny 192.168.0.0/24 192.168.0.0/24
   tuntap-router-firewall-out = allow 192.168.0.0/24 0.0.0.0
   ; we need this rule as default policy is 'allow'
   tuntap-router-firewall-out = deny
   tuntap-router-firewall-in = allow 192.168.0.1 192.168.0.0/24
   tuntap-router-firewall-in = deny 192.168.0.0/24 192.168.0.0/24
   tuntap-router-firewall-in = allow 0.0.0.0 192.168.0.0/24
   ; we need this rule as default policy is 'allow'
   tuntap-router-firewall-in = deny
   
和一个vassal

.. code-block:: ini

   [uwsgi]
   master = true
   ; set the hostname
   exec-as-root = hostname foobar
   ; bring up loopback
   exec-as-root = ifconfig lo up
   ; bring up the tuntap device and connect to the emperor
   tuntap-device = uwsgi0 /var/run/tuntap.socket
   ; configure the 'uwsgi0' interface
   exec-as-root = ifconfig uwsgi0 192.168.0.2 netmask 255.255.255.0
   ; use the tuntap router as default gw
   exec-as-root = route add default gw 192.168.0.1
   ...

Cron
****

Cron任务被添加到vassal文件中，语法与经典的crontab有点不同，我们不使用*和,，只使用数字
(是哒，它比经典的cron功能较少，但是uWSGI配置文件允许循环和其他结构)

.. code-block:: ini

   [uwsgi]
   ; run at 23:59 every day
   cron = 59 23 -1 -1 -1 myscript arg1
   ; run every five minutes on saturday
   cron = -5 -1 -1 -1 6

静态文件服务
*******************

额外的守护进程
******************

SSH
***

用名字空间设置来管理ssh可能确实棘手。Linux系统调用"setns"允许“附加”到一个已运行的名字空间上。

它一般能用，但现在我会告诉你为什么我不想将其用于我的服务的一个技术原因：我不喜欢它。就是这么一回事。

我们已经看到，unix socket作为一个名字空间之间的通信渠道工作得非常好，那么，为什么不用它们来“进入”一个已经运行的名字空间呢？

如果你作为unix系统管理员，那么你不能忽略伪终端 (或一般来说，终端)。它是unix世界中最古老的（以及最原始的）API之一，根据工作，根据年龄。并且它们工作得不错。

uWSGI发行版带了2个pty相关的插件：pty和forkptyrouter。

第一个简单将单个伪终端附加到你的worker上，并且绑定到一个网络地址。连接到这个地址可以访问到这个伪终端。这个技巧使得诸如共享调试这样的技术成为可能。pty插件也公开了客户端部分，因此你可以使用uwsgi二进制文件自身来连接到这个pty。

这对于我们的ssh访问可有用？不是这样的。

我们现在需要的是forkptyrouter (或者是更具可读性的forkpty-router)。它跟pty服务器非常相似，不同在于，它为每个连接生成一个新的pty。正如ssh做的那样。

forkpty-router运行到名字空间中，因此任何附加到它上面的进程将有效地运行在名字空间本身。

你现在应该明白了这点：我们的客户通过ssh作为非名字空间账户登陆，但并不是给予他们默认的shell，而死强制他们连接到pty-router。

这种方法的“缺点”是，我们需要两个用于每个ssh对端的pty (一个用于client -> ssh，而另一个用于ssh -> namespace)。

要强制ssh服务器运行一个指定的命令，则在sshd_config中使用ForceCommand指令


福利：KSM
**********

缺少什么
***************

- 计算网络使用
- 扩展到多台机器
