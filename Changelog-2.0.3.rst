uWSGI 2.0.3
===========

更新日志 20140317

错误修复
********

* 修复spooler的'at'键使用
* 使用即需Emperor socket修复内存和fd泄漏
* 在__APPLE__上，为syslog插件使用LOG_NOTICE
* 修复mongrel2支持
* hack以避免libmongoclient在处理损坏的指针时崩溃
* 日志告警现在是一个uwsgi_log_verbose()封装器
* 修复tuntap路由器内存损坏
* 独立于DHE参数设置ECDHE曲线 (Hynek Schlawack)
* 在检查每一个waitpid之前，不等待整个Emperor循环
* 修复caller()的一个回退，不代表启动*.psgi程序 (Ævar Arnfjörð Bjarmason)

新特性
********

Emperor SIGWINCH和SIGURG
---------------------------

Emperor现在响应两种新的信号：

SIGWINCH: 强制emperor重新扫描vassal

SIGURG: 清理Emperor状态 (目前，它只清理它的黑名单)

从git仓库，实时构建插件
-------------------------------------------------

现在，你可以构建存储在git服务器上的插件了：

.. code-block:: sh

   uwsgi --build-plugin https://github.com/unbit/uwsgi-bonjour
   
或者

.. code-block:: sh

   UWSGI_EMBED_PLUGINS="bonjour=https://github.com/unbit/uwsgi-bonjour" pip install uwsgi

uwsgi.add_var(key, value)
-------------------------

现在，你可以直接从你的应用设置请求变量，以更好地与内部路由子系统集成

.. code-block:: pl

   my $app = sub {
        uwsgi::add_var("newvar","newvalue");
        return [200, ['Content-Type' => 'text/html'], ["Hello"]];
   }
   
.. code-block:: sh

   uwsgi --http-socket :9090 --psgi hello.pl --response-route-run "log:\${newvar}"
   
add_var已在CPython和Perl插件中实现

'disableheaders'路由动作
-------------------------------

这个新的动作禁用响应头的发送，独立于当前的请求状态

糟糕的条件下更智能的Emperor
---------------------------------

现在，Emperor在它不可能正确地杀死一个损坏的vassal时（无论是不一致的Emperor状态，还是由于内部系统问题），会完全销毁内部vassal相关的结构

可用性
************

你可以从这里下载uWSGI 2.0.3：http://projects.unbit.it/downloads/uwsgi-2.0.3.tar.gz
