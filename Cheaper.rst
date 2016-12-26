uWSGI cheaper子系统 ——  适应性进程生成
========================================================

uWSGI通过可插拔算法，提供了动态扩展运行的worker数量的能力。使用 ``uwsgi --cheaper-algos-list`` 来获取可用算法的列表。

用法
-----

要启用cheaper模式，则添加 ``cheaper = N`` 选项到uWSGI配置文件中，其中，N是uWSGI可以运行的worker的最小数目。
``cheaper`` 必须小于配置的worker最大数目
(``workers`` 或者 ``processes`` 选项)。

.. code-block:: ini

   # set cheaper algorithm to use, if not set default will be used
   cheaper-algo = spare

   # minimum number of workers to keep at all times
   cheaper = 2
   
   # number of workers to spawn at startup
   cheaper-initial = 5

   # maximum number of workers that can be spawned
   workers = 10

   # how many workers should be spawned at a time
   cheaper-step = 1


这个配置将会告诉WSGI负载之下最多运行10个worker。如果应用处于idle状态，那么uWSGI将会停止worker，但它总是会让至少2个worker在运行。使用 ``cheaper-initial`` ，你可以控制在启动的时候应该生成几个worker。如果你的平均负载要求比最小数量的worker还要多，那么你可以让它们立即生成，然后在负载足够低的情况下，"省省" (杀死它们)。当cheaper算法决定它需要更多的worker时，它会生成它们的 ``cheaper-step`` 。这在你有一个高的最大worker数的时候有用 —— 否则在突然尖峰负载的情况下，它会花费大量的时间来一个一个生成足够的worker。

设置内存限制
---------------------

自1.9.16起，可以设置rss内存限制，从而在没有达到进程数量限制，但是所有worker使用的rss内存的总数量达到指定限制的时候停止cheapter生成新worker。

.. code-block:: ini

   # soft limit will prevent cheaper from spawning new workers
   # if workers total rss memory is equal or higher
   # we use 128MB soft limit below (values are in bytes)
   cheaper-rss-limit-soft = 134217728

   # hard limit will force cheaper to cheap single worker
   # if workers total rss memory is equal or higher
   # we use 160MB hard limit below (values are in bytes)
   cheaper-rss-limit-hard = 167772160

注意：

- 硬限制是可选的，可以单独使用软限制。
- 硬限制值必须比软限制值高，这两个值都不应该彼此太接近。
- 硬限制值应该是软限制值 + 至少是用于给定应用的平均worker内存使用。
- 软限制值是cheaper的限制器，它不会生成更多的worker，但已运行中的worker的内存使用可能会增长，要处理它，也可以设置reload-on-rss。建议为应用内存使用cgroups设置牢不可破的限制。


``spare`` cheaper算法
---------------------------

这是默认的算法。如果所有的worker在 ``cheaper_overload`` 秒内都处于忙碌状态，那么uWSGI将会生成新的worker。当负载消失的时候，它将开始一次停止一个进程。


``spare2`` cheaper算法
----------------------------

这个算法与spare类似，但适用于更快增长worker数，并且较慢降低worker数的大规模。

当idle状态的worker数比 ``cheaper`` 指定的数量小的时候，它生成 (``cheaper`` -
idle状态的worker数) worker。一次生成worker的数量的最大值可以通过 ``cheaper-step`` 来限制。例如。 ``cheaper`` 是4，有2个idle状态的worker，并且 ``cheaper-step`` 为1，它会生成1个worker。

当idle状态的worker数比 ``cheaper`` 大时，它会增加内部计数器。当idle状态的worker数小于或等于 ``cheaper`` 时，会重置计数器。
当计数器等于 ``cheaper-idle`` 时，省掉一个worker，然后重置计数器。

样例配置：

.. code-block:: ini

   workers = 64          # maximum number of workers

   cheaper-algo = spare2
   cheaper = 8           # tries to keep 8 idle workers
   cheaper-initial = 8   # starts with minimal workers
   cheaper-step = 4      # spawn at most 4 workers at once
   cheaper-idle = 60     # cheap one worker per minute while idle


``backlog`` cheaper算法
-----------------------------

.. note:: ``backlog`` 只适用于Linux以及TCP socket (不是UNIX域的socket)。

如果socket的监听队列有超过 ``cheaper_overload`` 个请求在等待处理，那么uWSGI将会生成新的worker。如果积压降低，它将会开始一次杀掉一个进程。

``busyness`` cheaper算法
------------------------------

.. note:: 这个算法是可选的，只有在编译并加载了 ``cheaper_busyness`` 插件的情况下，才可以用它。

该插件实现了这样一个算法，基于给定时间周期内的平均利用率来添加或移除worker。它的目标是保持在任意有比需要的最小值的worker数还多的worker可用，这样，应用将总是能够处理新的请求。如果你只想运行最小数量的worker，那么使用spare或者backlog算法。

使用该插件主要是因为spare和backlog插件工作的方式引发非常激进的缩放行为。如果你设置一个低的 ``cheaper`` 值
(例如，1)，那么uWSGI将会一直只运行1个worker，然后只在运行中的那个worker过载的时候才生成新的worker。如果应用要求更多的worker，那么uWSGI将会一直生成停止worker。只有在非常低的负载期间，最小数量的worker才够。

Busyness算法试着与其相反：按需生成worker，然后只有在很有肯能不需要它们的时候才停止一些。这应该会使得worker数量更加稳定，并且更少进行重新生成。由于大部分的时间里，我们比实际需要的拥有更多的worker，因此平均应用响应时间应该比使用其他插件更低。

选项：

cheaper-overload
****************

指定窗口，以秒为单位，用于追踪worker的平均busyness。例如：

.. code-block:: ini

   cheaper-overload = 30

这个选项将会每30秒检查busyness。如果在上一个30秒期间，所有worker都是运行3秒，并且在剩下的27秒内出于idle状态，那么计算所得的busyness将会是10% (3/30)。这个值将会决定uWSGI可以多快响应负载尖峰。至少每 ``cheaper-overload`` 秒会生成新的worker (除非你在Linux上运行uWSGI —— 详情请见
``cheaper-busyness-backlog-alert`` )。

如果你想要更快地对负载尖峰进行响应，那么为这个值取一个小的值，这样，就会更频繁地计算busyness。记住，这可能会导致比需要的更频繁地启动/停止worker，因为每一个小的尖峰都会生成新的worker。使用一个高的 ``cheaper-overload`` 值，则worker数量将会更少发生改变，因为较长的周期将会吞掉所有负载短尖峰和极端值。默认是3，对于busyness插件，最好使用较高的值 (10-30)。

cheaper-step
************

当算法决策需要worker的时候，要生成worker的数目。默认是1。

cheaper-initial
***************

启动应用的时候，启动的worker数。在应用启动之后，如果需要的话，算法可以停止或者启动worker。

cheaper-busyness-max
********************

这是允许的最大busyness。每次上一个 ``cheaper-overload`` 秒计算的busyness比这个值高的时候，uWSGI将会生成
``cheaper-step`` 新worker。默认是50.

cheaper-busyness-min
********************

这是最小的busyness。如果当前的busyness位于该值之下，那么应用就会被认为处在一个“idle周期”中，而uWSGI将会开始对它们进行计数。一旦到达idle周期的所需数目，uWSGI将会杀掉一个worker。默认是25.

cheaper-busyness-multiplier
***************************

这个选项告诉uWSGI在停止一个worker之前，我们需要多少个idle周期。
在到达这个限制之后，uWSGI将会停止一个worker，并且设置这个计数器。

例如：

.. code-block:: ini
   
   cheaper-overload = 10
   cheaper-busyness-multiplier = 20
   cheaper-busyness-min = 25

如果连续20次检查，每10秒执行一次（总共200秒），平均worker busyness低于25%，那么将会停止一个worker。如果平均busyness跳到 ``cheaper-busyness-max`` 以上，idle周期计数器将会被重置，并且我们生成新的worker。如果在idle周期计数期间，平均busyness跳到 ``cheaper-busyness-min`` 以上，但仍然低于 ``cheaper-busyness-max`` ，那么idle周期计数器将会被调整，而我们需要等待另外一个idle周期。如果在idle周期计数期间，平均busyness跳到 ``cheaper-busyness-min`` 以上，但仍然低于 ``cheaper-busyness-max`` 连续3次，那么会重置idle周期计数器。

cheaper-busyness-penalty
************************

当worker因为足够的idle周期停止，然后快速(比worker所需的同等时间少)又生成回来的时候，uWSGI将会自动调整停止worker所需的idle周期数，然后我们会增加 ``cheaper-busyness-multiplier`` 值这个值。默认是1.

例如：

.. code-block:: ini

   cheaper-overload = 10
   cheaper-busyness-multiplier = 20
   cheaper-busyness-min = 25
   cheaper-busyness-penalty = 2

如果连续20次检查，每10秒执行一次（总共200秒），平均worker busyness低于25%，那么将会停止一个worker。如果新的worker在少于200秒内生成 (时间从我们生成它之前的最后一个worker算起)，那么 ``cheaper-busyness-multiplier`` 值将会被增加到22 (20+2)。现在，我们将需要等待220秒 (22*10)来cheap另一个worker。这个选项用于防止worker一直启动和停止，因为一旦我们停止了一个worker，busyness也许会提供以致于够着 ``cheaper-busyness-max`` 。没有这个，或者如果被不充分地调整，我们会陷入一个停止/启动反馈循环。

cheaper-busyness-verbose
************************

这个选项启用 ``cheaper_busyness`` 插件的调试日志。

cheaper-busyness-backlog-alert
******************************

这个选项只有在Linux上能用。它用于允许快速响应负载峰值，即使使用了高的 ``cheaper-overload`` 值。在每个uWSGI master周期（默认是1秒）中，会检查当前的监听队列。如果它比这个值高，那么会生成一个紧急worker。当使用这个选项的时候，使用高的 ``cheaper-overload`` 值以拥有更平滑的worker数缩放是安全的。默认是33.

cheaper-busyness-backlog-multiplier
***********************************

这个选项只有在Linux上能用。它就像 ``cheaper-busyness-multiplier`` ，除了它只用于监听队列高于 ``cheaper-busyness-backlog-alert`` 时的紧急worker生成。

紧急worker是在大负载峰值下生成的，用以防止当前运行的worker过载。有时，负载峰值是随机并且短暂的，这会生成大量的紧急worker。在这种情况下，获得那些worker之前，我们会需要等待几个周期。这提供了一个交替的multiplier来更快的获得这些进程。默认是3.

cheaper-busyness-backlog-step
*****************************

这个选项只有在Linux上能用。 它设置当监听队列高于 ``cheaper-busyness-backlog-alert`` 时生成的紧急worker数。默认是1.

cheaper-busyness-backlog-nonzero
********************************

这个选项只有在Linux上能用。如果超过N秒，请求监听队列都>0，那么它将会生成新的紧急worker。它用来保护服务器不受这样的边缘情况之害：只有单个worker运行，并且这个worker在处理一个长时间运行的请求。如果uWSGI收到了新的请求，那么它们将会待在请求队列中，直到那个长时间运行请求完成。使用这个选项，我们可以检测这样一个条件，并生成新的worker来防止入队请求超时。默认是60.

关于Busyness的一些注意事项
**************************

* 通过实验确定设置。对于每个人而言，没有哪个万金油值应该被使用。测试并挑选对你来说的最佳值。监控uWSGI统计数据 (例如，通过Carbon) 会使得决定使用哪个值简单一些。
* 不要指望busyness恒久不变。它会经常改变。最终，真正的用户是以非常随机的方式与你的应用进行交互的。推荐使用更长的--cheaper-overload值 (>=30) ，使得尖峰更少。
* 如果你想要运行这个插件的一些基准，那么你应该使用添加随机性到工作负载的工具。
* 使用少量的worker (2-3)，启动新的worker，或者停止一个worker或许会大大影响busyness。如果你有2个worker，带50%的busyness，那么停止其中一个将会增加busyness到100%。记住，当挑选最小和最大程度的时候，如果大部分时间只有少量的worker在运行，那么max应该超过min的两倍，否则，每当停止一个worker的时候，它将会增加busyness到超过max程度。
* 使用少量的worker (1-4) 和默认设置，期望这个插件将会保持平静busyness低于最小程度；调整程度来补偿。
* 使用处理负载所需的更多数量的worker，worker计数器将会稳定在接近最小busyness程度的某处，在这个值附近上下波动。
* 在对这个插件进行实验的时候，建议启用 ``--cheaper-busyness-verbose`` ，从而知道它正在做什么。一个样例日志如下。

  .. code-block:: python

     # These messages are logged at startup to show current settings
     [busyness] settings: min=20%, max=60%, overload=20, multiplier=15, respawn penalty=3
     [busyness] backlog alert is set to 33 request(s)

     # With --cheaper-busyness-verbose enabled You can monitor calculated busyness
     [busyness] worker nr 1 20s average busyness is at 11%
     [busyness] worker nr 2 20s average busyness is at 11%
     [busyness] worker nr 3 20s average busyness is at 20%
     [busyness] 20s average busyness of 3 worker(s) is at 14%

     # Average busyness is under 20%, we start counting idle cycles
     # we have overload=20 and multiplier=15 so we need to wait 300 seconds before we can stop worker
     # cycle we just had was counted as idle so we need to wait another 280 seconds
     # 1 missing second below is just from rounding, master cycle is every 1 second but it also takes some time, this is normal
     [busyness] need to wait 279 more second(s) to cheap worker

     # We waited long enough and we can stop one worker
     [busyness] worker nr 1 20s average busyness is at 6%
     [busyness] worker nr 2 20s average busyness is at 22%
     [busyness] worker nr 3 20s average busyness is at 19%
     [busyness] 20s average busyness of 3 worker(s) is at 15%
     [busyness] 20s average busyness is at 15%, cheap one of 3 running workers

     # After stopping one worker average busyness is now higher, which is no surprise
     [busyness] worker nr 2 20s average busyness is at 36%
     [busyness] worker nr 3 20s average busyness is at 24%
     [busyness] 20s average busyness of 2 worker(s) is at 30%
     # 30% is above our minimum (20%), but it's still far from our maximum (60%)
     # since this is not idle cycle uWSGI will ignore it when counting when to stop worker
     [busyness] 20s average busyness is at 30%, 1 non-idle cycle(s), adjusting cheaper timer

     # After a while our average busyness is still low enough, so we stop another worker
     [busyness] 20s average busyness is at 3%, cheap one of 2 running workers

     # With only one worker running we won't see per worker busyness since it's the same as total average
     [busyness] 20s average busyness of 1 worker(s) is at 16%
     [busyness] 20s average busyness of 1 worker(s) is at 17%

     # Shortly after stopping second worker and with only one running we have load spike that is enough to hit our maximum level
     # this was just few cycles after stopping worker so uWSGI will increase multiplier
     # now we need to wait extra 3 cycles before stopping worker
     [busyness] worker(s) respawned to fast, increasing cheaper multiplier to 18 (+3)

     # Initially we needed to wait only 300 seconds, now we need to have 360 subsequent seconds when workers busyness is below minimum level
     # 10*20 + 3*20 = 360
     [busyness] worker nr 1 20s average busyness is at 9%
     [busyness] worker nr 2 20s average busyness is at 17%
     [busyness] worker nr 3 20s average busyness is at 17%
     [busyness] worker nr 4 20s average busyness is at 21%
     [busyness] 20s average busyness of 4 worker(s) is at 16%
     [busyness] need to wait 339 more second(s) to cheap worker
