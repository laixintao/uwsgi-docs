uWSGI Legion子系统
==========================

自uWSGI 1.9-dev起，已经添加了一个用于集群的新的子系统：Legion子系统。一个Legion是一组不断争取主导的uWSGI节点。每个节点都有一个武力值 (如果可能的话，彼此间该值不同)。具有最高武力值的节点就是这个Legion的Lord (或者如果你喜欢不大游戏的说法，嗯，更工程师点的术语是：master)。这种不断的斗争会生成7种事件：

1. ``setup`` - legion子系统在一个节点上启动的时候
2. ``join`` - 在第一次到达仲裁时，只有在新加入的节点上
3. ``lord`` - 当该节点成为lord
4. ``unlord`` - 当这个节点失去了lord称号
5. ``death`` - 当legion子系统关闭的时候
6. ``node-joined`` - 当任何新节点加入到我们的legion的时候
7. ``node-left`` - 当任何节点离开我们的legion的时候

每当这样的一个事件发生的时候，你都可以触发操作。

**注意**：要构建带Legion支持的uWSGI，则必须安装 ``openssl`` 头文件。

IP接管
^^^^^^^^^^^

这对于集群环境来说，是一个非常常见的配置。IP地址是一种必须只由一个节点拥有的资源。对于这个例子而言，那个节点就是我们的Lord。如果我们正确的配置了一个Legion (记住，一个uWSGI实例可以是所有你需要的legion的成员)，那么我们可以很容易实现IP接管。

.. code-block:: ini

   [uwsgi]

   legion = clusterip 225.1.1.1:4242 98 bf-cbc:hello
   legion-node = clusterip 225.1.1.1:4242

   legion-lord = clusterip cmd:ip addr add 192.168.173.111/24 dev eth0
   legion-lord = clusterip cmd:arping -c 3 -S 192.168.173.111 192.168.173.1

   legion-setup = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0
   legion-unlord = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0
   legion-death = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0

在这个例子中，我们加入了一个名为 ``clusterip`` 的legion。要从其他节点接收消息，那么绑定到多播地址225.1.1.1:4242。这个节点的武力值会是98，而每一条消息将会使用CBC中的Blowfish算法，使用共享密钥 ``hello`` 进行加密。 ``legion-node`` 选项制定了我们宣布消息的目的地。由于我们使用多播，因此只需要制定一个单一的“节点”。最后的选项是在该集群的不同阶段触发的动作。对于一个IP接管解决方案，我们简单依赖于Linux的 ``iproute`` 命令来设置/取消设置ip地址，发送一个额外的ARP消息来宣告这种改变。显然，这个特定的例子需要root选项，或者 ``CAP_NET_ADMIN`` Linux能力。因此，确保不要在与管理IP接管相同的uWSGI实例上运行不可信应用。

仲裁集
**********

要选择一个Lord，Legion中的每个成员都要进行投票。当一个legion中所有活动成员都同意选择一个Lord的时候，那么这个Lord就会当选，而老的Lord会被降级。每当加入一个新的节点，或者有节点离开的时候，就会重新计算仲裁集，然后被记录到整个集群中。

选择Lord
*****************

一般来讲，具有较高武力值的节点会被选为Lord，但是可能会出现多个节点具有相同武力值的情况。当启动一个节点的时候，会给它分配一个UUID。如果两个节点具有相同的武力值，那么拥有较高字典序的UUID值的那个节点将会胜出。

分裂的集群
***********

虽然Legion中的每个成员必须发送它内部集群成员的校验和，但是系统仍然易受“分裂的集群”问题的攻击。如果一个节点失去了与集群的连接，那么它可能会相信它是唯一可用的节点，然后开始Lord模式。

对于许多场景来说，这并不是最佳的。如果在一个legion中有超过2个节点，那么你可能想要考虑调整仲裁集水平。仲裁集水平是选举一个lord所需的票数（与节点数相对）。
``legion-quorum`` 是用于这项任务的选项。你可以通过让Legion子系统检查至少2票来解决分裂集群问题：

.. code-block:: ini

   [uwsgi]

   legion = clusterip 225.1.1.1:4242 98 bf-cbc:hello
   legion-node = clusterip 225.1.1.1:4242

   legion-quorum = clusterip 2

   legion-lord = clusterip cmd:ip addr add 192.168.173.111/24 dev eth0
   legion-lord = clusterip cmd:arping -c 3 -S 192.168.173.111 192.168.173.1

   legion-setup = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0
   legion-unlord = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0
   legion-death = clusterip cmd:ip addr del 192.168.173.111/24 dev eth0

由1.9.7开始，你可以使用武力值为0的节点 (概念类似于MongoDB的
Arbiter节点)，在检查仲裁集的时候会考虑这样的节点，但是这样的节点将永远不会成为Lord。这在你只需要一对节点，但是要对抗分裂集群的时候很有用。

动作
*******

一个legion的四个阶段都能触发动作。动作系统是模块化的，因此你可以添加新类型的动作。目前支持的动作是：

``cmd:<command>``
^^^^^^^^^^^^^

运行一个shell命令。

``signal:<num>``
^^^^^^^^^^^^^^^^

引发一个uWSGI信号。

``log:<msg>``
^^^^^^^^^^^^^

记录一个消息。例如，你可以将日志动作与告警系统结合在一起，从而获得免费的集群监控。

``Multicast, broadcast and unicast``
************************************

即使多播可能是实现集群最简单的方式，但是它不是适用于所有网络的。如果多播不可选，那么你可以依赖于正常的IP地址。仅需绑定到一个地址上，然后发送所有你需要的legion-node选项：

.. code-block:: ini

   [uwsgi]

   legion = mycluster 192.168.173.17:4242 98 bf-cbc:hello
   legion-node = mycluster 192.168.173.22:4242
   legion-node = mycluster 192.168.173.30:4242
   legion-node = mycluster 192.168.173.5:4242

这是用于一个拥有4个节点的集群(这个节点 + 3个其他节点)

多Legion
****************

你可以把多个legion加入到相同的实例中。只是要记住，为每个legion使用不同的地址 (多播情况下则是端口)。

.. code-block:: ini

   [uwsgi]

   legion = mycluster 192.168.173.17:4242 98 bf-cbc:hello
   legion-node = mycluster 192.168.173.22:4242
   legion-node = mycluster 192.168.173.30:4242
   legion-node = mycluster 192.168.173.5:4242

   legion = mycluster2 225.1.1.1:4243 99 aes-128-cbc:secret
   legion-node = mycluster2 225.1.1.1:4243

   legion = anothercluster 225.1.1.1:4244 91 aes-256-cbc:secret2
   legion-node = anothercluster 225.1.1.1:4244

安全性
********

Legion子系统发送的每个包都是使用特定的加密方法，一个预先共享的密钥以及一个可选的IV（初始化向量）进行加密的。取决于加密方法，IV可能是一个必要参数。要获取支持的加密方法列表，运行 ``openssl enc -h`` 。

.. important:: Legion的每个节点必须使用相同的加密参数。

要指定IV，仅需添加另一个参数到 **legion** 选项。

.. code-block:: ini

   [uwsgi]

   legion = mycluster 192.168.173.17:4242 98 bf-cbc:hello thisistheiv
   legion-node = mycluster 192.168.173.22:4242
   legion-node = mycluster 192.168.173.30:4242
   legion-node = mycluster 192.168.173.5:4242

要减少基于重放攻击的影响，拒绝带低于30秒的时间戳的包。这是一个可调参数。如果你对所有节点的时间无法控制，那么你可以增加时钟偏差公差。

调整和时间偏差
*********************

当前，你可以调整三个参数。这些调整项影响系统中的所有Legion。每个包发送的频率 (以秒为单位)
(**legion-freq <secs>**)，节点不发送包后经过多少时间会被认为已经死亡（该时间以秒为单位） (**legion-tolerance <secs>**)，以及节点之间的时间偏差数 (**legion-skew-tolerance <secs>**)。Legion子系统要求严格的时间同步，因此强烈建议使用NTP或者类似的工具。默认情况下，每3秒发送每个包，一个节点会在（不发送包后经过）15秒后认为死亡，而允许30秒的时间偏差。减少偏差容忍度应该加大对重放工具的安全保护。

Lord scroll (即将推出)
*************************

Legion子系统可以用于各种各样的目的，从master选举到节点自动发现或者简单的监控。一个例子是将"一组数据" (一个scroll)赋给每个节点。这个的一种使用时传递重新配置的参数给你的应用，或者记录特定的消息。目前，scroll系统正在加以改进之中，因此，如果你有任何想法，那么请加入到我们的邮件列表或者IRC频道中吧。

Legion API
**********

你可以通过简单调用以下函数来判断实例是否是Legion的Lord：

.. code-block:: c

   int uwsgi_legion_i_am_the_lord(char *legion_name);

如果当前实例时特定Legion的Lord的话，则返回1。

* Python插件将其公开为 ``uwsgi.i_am_the_lord(name)``
* PSGI插件将其公开为 ``uwsgi::i_am_the_lord(name)``
* Rack插件将其公开为 ``UWSGI::i_am_the_lord(name)``

显然，未来将会添加更多的API函数，你可以自由公开你的想法。

统计信息
********

可以在 :doc:`StatsServer` 中找到Legion的信息。确保理解“节点”和“成员”之间的不同。节点是你用 **legion-node** 选项配置的对端，而成员是加入到集群的有效的节点。

老的集群子系统
****************************

在0.9开发周期中，添加了一个集群子系统 (基于多播)。它非常原始不可靠，并且很有可能没人把它当回事。新的方法是使用可以使用不同后端的一般的API来改变它。Legion子系统可以是这些后端中的一员，以及像corosync或者redhat集群套件这样的项目（也可以是这些后端）。
