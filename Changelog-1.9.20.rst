uWSGI 1.9.20
============

更新日志 [20131117]

为2.0进行的第一轮弃用和清除
************************************************

* Go茶具现在被认为是“损坏的”，并且已移出 ``plugins`` 目录。在uWSGI中运行Go应用的推荐方法是使用 :doc:`GCCGO` 插件。
* 已移出 ``--auto-snapshot`` 选项，现在可以通过 :doc:`MasterFIFO` 来进行实例的高级管理。
* 已移除matheval支持，而推出了一个通用的"matheval"插件 (用于内部路由) (但默认不编译)。看看下面在配置文件中做“算术”的新方法。
* "erlang"和"pyerl"插件不能用了，并且已移出 ``plugins`` 目录。将会在2.0发布之后完全重写Erlang支持。

下一次计划的弃用和清除
****************************************

ZeroMQ API (实际上是一个简单的函数) 将会被移除。每个使用ZeroMQ的插件江湖创建其自己的 ``zmq`` 上下文 (无需共享)。这意味着在uWSGI核心二进制文件中不再链接libzmq。

Mongrel2协议支持将会移到一个"mongrel2"插件上，而不是内嵌到核心中。

错误修复
********

* 修复在lazy模式下进行优雅重载时的master中止。
* 修复 ``default_app`` 使用。
* 另一轮coverity修复，由Riccardo Magliocchetti完成。
* 修复读取请求体时的 ``EAGAIN`` 管理。

新特性
********

RPC子系统的64位返回值
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在此版本之前，每个RPC响应都被限制到64K大小 (16位)。

现在，RPC协议自动检测是否需要更多的空间，并且可以扩展到64位。

该方法的另一个优势在于，只会分配每个响应需要的内存大小，而不是每次盲目创建一个64k大小的块。

新的GCCGO插件
^^^^^^^^^^^^^^^^^^^^

看看官方文档： :doc:`GCCGO`

这个插件处于开发的初期，但它已经非常稳定了。

配置文件中简单计算
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如前所见，我们移除了matheval支持，取而代之的是一个简化的接口：

https://uwsgi-docs.readthedocs.io/en/latest/Configuration.html#placeholders-math-from-uwsgi-1-9-20-dev

例如，现在，你可以自动设置线程数为：

.. code-block:: ini

   [uwsgi]
   ; %k is a magic var translated to the number of cpu cores
   threads = %(%k * 3)
   ...

(``%k * 3`` 是 ``number_of_cpu_cores * 3``).

新的魔术变量
^^^^^^^^^^^^^^

``%t``
    Unix时间 (以秒为单位，实例启动时收集)。

``%T``
    Unix时间 (以微秒为单位，实例启动时收集)。

``%k``
    检查CPU核数。

Perl/PSGI改进
^^^^^^^^^^^^^^^^^^^^^^

* :doc:`Chunked`.
* ``psgix.io`` 是一个映射到连接文件描述符的 ``Socket::IO`` 对象 (你需要用 ``--psgi-enable-psgix-io`` 来启用它)。
* 来自API的 ``uwsgi::rpc`` 和 ``uwsgi::connection_fd`` 。
* ``--plshell`` 江湖调用一个交互式shell (基于 ``Devel::REPL``)。

新的原生协议： ``--https-socket`` 和 ``--ssl-socket``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当带SSL支持构建时，uWSGI公开了两个原生socket协议：HTTPS和通过SSL的uwsgi。

这两个选项都以以下值为参数： ``<addr>,<cert>,<key>[,ciphers,ca]`` 

.. code-block:: ini

   [uwsgi]
   https-socket = :8443,foobar.crt,foobar.key
   ...
   
目前，没有主流的web服务器支持SSL上的uwsgi，在未来的几小时内，将会把一个nginx补丁送交审核。

PROXY (version1)协议支持
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

最近，Amazon ELB增加了对HAProxy PROXY (version 1)协议的支持。这个简单的协议允许前端传递真正的客户端IP到后端。

增加 ``--enable-proxy-protocol`` 将会强制 ``--http-socket`` 检查用于设置 ``REMOTE_ADDR`` 和 ``REMOTE_PORT`` 字段的PROXY协议请求。

新的度量收集器
^^^^^^^^^^^^^^^^^^^^^^

``avg``
    计算孩子的算术平均： ``--metric name=foobar,collector=avg,children=metric1;metric2``.

``accumulator``
    总是添加指定孩子的值到最终值中。

``multiplier``
    n用 ``arg1n`` 中指定的值乘以指定孩子的总和。

看看 :doc:`Metrics`.

可用性
************

uWSGI 1.9.20于2013年11月17日发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.20.tar.gz.