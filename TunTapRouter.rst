TunTap路由器
=================

TunTap路由器是为运行在一个专用网络名字空间中的linux进程提供网络连接的ad-hoc方案。好吧，显然它有其他用途，但是非常有可能这是最有趣的，也是为什么会开发它的原因。目前，它只构建在Linux平台上。


TunTap路由器默认不编译。


要一次性拥有它：

.. code-block:: sh

   UWSGI_EMBED_PLUGINS=tuntap make
   
(是哒，这个插件只命名为'tuntap'，因为它有效公开了各种tuntap设备特性)

使用它的最佳方式是将其绑定到一个unix socket上，允许新的名字空间中的进程访问它 (一般来说，unix socket是用于linux名字空间的最佳通信通道)。


第一个配置
****************

我们想要让我们的vassal使用192.168.0.0/24网络，并且将192.168.0.1作为默认网关。

默认网关 (也就是说，tuntap路由器) 是由Emperor自身管理的

.. code-block:: ini

   [uwsgi]
   ; create the tun device 'emperor0' and bind it to a unix socket
   tuntap-router = emperor0 /tmp/tuntap.socket
   ; give it an ip address
   exec-as-root = ifconfig emperor0 192.168.0.1 netmask 255.255.255.0 up
   ; setup nat
   exec-as-root = iptables -t nat -F
   exec-as-root = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   ; enable linux ip forwarding
   exec-as-root = echo 1 >/proc/sys/net/ipv4/ip_forward
   ; force vassals to be created in a new network namespace
   emperor-use-clone = net
   emperor = /etc/vassals
   
由这个Emperor生成的vassal将在无网络连接的情况下产生。

要让它们访问公共网络，我们创建一个新的tun设备 (它将只存在于vassal网络名字空间中)，指示它路由流量到Emperor的tuntap unix socket：

.. code-block:: ini

   [uwsgi]
   ; we need it as the vassal have no way to know it is jailed
   ; without it post_jail plugin hook would be never executed
   jailed = true
   ; create uwsgi0 tun interface and force it to connect to the Emperor exposed unix socket
   tuntap-device = uwsgi0 /tmp/tuntap.socket
   ; bring up loopback
   exec-as-root = ifconfig lo up
   ; bring up interface uwsgi0
   exec-as-root = ifconfig uwsgi0 192.168.0.2 netmask 255.255.255.0 up
   ; and set the default gateway
   exec-as-root = route add default gw 192.168.0.1
   ; classic options
   uid = customer001
   gid = customer001
   socket = /var/www/foobar.socket
   psgi-file = foobar.pl
   ...

内嵌防火墙
*********************

TunTap路由器由一个非常简单的防火墙，用来管理vassal的流量

防火墙基于2个链 (入链和出链)，而每条规则是由3个参数组成的：<action> <src> <dst>

会将这个防火墙应用于从客户端到tuntap设备(out)以及反过来(in)的流量


第一条匹配的规则终止这个链，如果没有规则应用上，那么策略就是"allow"

以下规则允许从vassal到互联网的访问，但是阻塞vassal的内部通信

.. code-block:: ini

   [uwsgi]
   tuntap-router = emperor0 /tmp/tuntap.socket
   
   tuntap-router-firewall-out = allow 192.168.0.0/24 192.168.0.1
   tuntap-router-firewall-out = deny 192.168.0.0/24 192.168.0.0/24
   tuntap-router-firewall-out = allow 192.168.0.0/24 0.0.0.0
   tuntap-router-firewall-out = deny
   tuntap-router-firewall-in = allow 192.168.0.1 192.168.0.0/24
   tuntap-router-firewall-in = deny 192.168.0.0/24 192.168.0.0/24
   tuntap-router-firewall-in = allow 0.0.0.0 192.168.0.0/24
   tuntap-router-firewall-in = deny
   
   exec-as-root = ifconfig emperor0 192.168.0.1 netmask 255.255.255.0 up
   ; setup nat
   exec-as-root = iptables -t nat -F
   exec-as-root = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   ; enable linux ip forwarding
   exec-as-root = echo 1 >/proc/sys/net/ipv4/ip_forward
   ; force vassals to be created in a new network namespace
   emperor-use-clone = net
   emperor = /etc/vassals
   
安全性
********

TunTap路由器的“切换”部分 (即，映射ip地址到vassal)是灰常简单的：TunTap路由器接收到的第一个来自于vassal的包为vassal注册那个ip地址。一个好方法 (从安全的角度来说) 是在vassal中的网络设置之后立即发送一个ping包：

.. code-block:: ini

   [uwsgi]
   ; create uwsgi0 tun interface and force it to connect to the Emperor exposed unix socket
   tuntap-device = uwsgi0 /tmp/tuntap.socket
   ; bring up loopback
   exec-as-root = ifconfig lo up
   ; bring up interface uwsgi0
   exec-as-root = ifconfig uwsgi0 192.168.0.2 netmask 255.255.255.0 up
   ; and set the default gateway
   exec-as-root = route add default gw 192.168.0.1
   
   ; ping something to register
   exec-as-root = ping -c 1 192.168.0.1
   
   ; classic options
   ...
   
在注册了一个vassal/ip对之后，只有那个组合才是有效的 (因此，其他vassal将不能够使用那个地址，直到持有该地址的vassal死掉)
   
   
未来
**********

这正成为unbit.it网络栈的一个非常重要的部分。我们现在致力于：

- 动态防火墙规则 (luajit搞了一个用于编写快速网络规则的很棒的工具)

- tuntap路由器的联合/代理 (tuntaprouter可以在一个tcp连接上多路传输vassal网络到一个外部的tuntap路由器 [这就是为什么你可以绑定一个tuntap路由器到一个tcp地址上])

- vassal鉴权 (或许老的UNIX配套凭证就够了)

- 用于网络统计数据(rx/tx/errors)的统计数据服务器

- 基于blastbeat项目的bandwidth shaper

