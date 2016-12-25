Apache支持
===============

目前，有三个可用的uwsgi协议相关的apache2模块。

mod_uwsgi
---------------

这是原始模块。它可靠，但是却令人难以置信地丑，并且不遵循许多apache编码规范。

可以以两种方式使用 ``mod_uwsgi`` ：

* "assbackwards"方式 (默认的方式)。它是最快的方式，但是离Apache2 API有点远。如果你不对uWSGI生成的内容使用Apache2过滤器 (包括gzip)，那么使用这个模式。
* "cgi"模式。这个相对有点慢，但是更Apache更好的集成。要使用CGI模式，则传递 ``-C`` 给uWSGI服务器。

选项
^^^^^^^

.. note::
   
   所有的选项都可以被设置在每个host或者每个location上。


**uWSGISocket <path> [timeout]**   uwsgi服务器socket的绝对路径和可选超时时间（以秒为单位）。

**uWSGISocket2 <path>**            容错uwsgi服务器socket的绝对路径

**uWSGIServer <host:port>**        一个UWSGI服务器的地址和端口 (例如，localhost:4000)

**uWSGIModifier1 <int>**           设置uWSGI modifier1  

**uWSGIModifier2 <int>**           设置uWSGI modifier2

**uWSGIForceScriptName <value>**   强制使用 ``SCRIPT_NAME`` (应用名)

**uWSGIForceCGIMode <on/off>**     强制uWSGI CGI模式，以与apache过滤器完美契合

**uWSGIForceWSGIScheme <value>**   强制使用WSGI scheme变量 (默认设置为"http")

**uWSGIMaxVars  <int>**            设置uwsgi协议变量的最大允许数量 (默认是128)


要传递自定义变量，请使用 ``SetEnv`` 指令：

..
  
  SetEnv UWSGI_SCRIPT yourapp


mod_proxy_uwsgi
---------------


这是最新的模块，并且可能是未来最好的选择。它是一个“代理”模块，因此你会获得由mod_proxy公开的所有特性。它是完全“兼容apache API”的，因此应该很容易将其与可用模块集成。使用它是很方便的；只是记得在你的apache配置中加载mod_proxy和
mod_proxy_uwsgi模块。

.. parsed-literal::

   ProxyPass /foo uwsgi://127.0.0.1:3032/
   ProxyPass /bar uwsgi://127.0.0.1:3033/
   ProxyPass / uwsgi://127.0.0.1:3031/

头两个分别设置到/foo和/bar的SCRIPT_NAME，而最后一个使用一个空的SCRIPT_NAME。你可以使用SetEnv指令设置额外的uwsgi变量，使用mod_proxy_balancer设置负载均衡请求。

.. parsed-literal::

   <Proxy balancer://mycluster>
      BalancerMember uwsgi://192.168.1.50:3031/
      BalancerMember uwsgi://192.168.1.51:3031/
   </Proxy>
   ProxyPass / balancer://mycluster

注意成员/节点定义中的最后一个斜杠。对于非空的SCRIPT_NAME/挂载点，它是可选的，但是对于挂载在域的根上的应用，则是必须的。目前，该模块缺乏设置modifier的能力，虽然这将会很快被修复。一个替代方法是设置你想要的插件作为第一个（0）使用：

.. parsed-literal::

   plugins = 0:php

.. note::

   `mod_proxy_uwsgi` is considered stable starting from uWSGI 2.0.6

.. note::

  如果你想要使用这个模块 (并且帮助uWSGI项目)，那么报告任何你找到的错误，而不是回到古老(并且丑陋)的mod_uwsgi
  
从Apache 2.4.9起，添加了对Unix socket的支持。语法相当简单：

.. parsed-literal::

  ProxyPass / unix:/var/lib/uwsgi/app1.sock|uwsgi://uwsgi-uds-app1/
  ProxyPass / unix:/var/lib/uwsgi/app2.sock|uwsgi://uwsgi-uds-app2/

.. note::

  根据Apache文档，代理地址的主机名部分会被忽略。然而，如果有人想要在同一个apache实例内，对不同的应用使用不同的socket，那么代理地址会被apache用来标识不同的代理worker：如果代理地址相同，那么只有第一个声明的unix域socket会获得一个worker，而第二个声明的应用将不会拥有worker，并且不可达。关于这个apache限制（问题？）的一个变通是使用不同的主机名。

mod_Ruwsgi
----------

这个模块基于SCGI模块，作者是Roger Florkowski。

.. note::

  该模块当前未公开。
