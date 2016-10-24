Web服务器集成
======================

uWSGI支持多种集成web服务器的方法。它自己也能够为HTTP请求服务。

Nginx
------------

.. seealso:: :doc:`Nginx`

自Nginx官方发行版本0.8.40起就包含了uWSGI模块。uWSGI包里维护了一个支持Nginx 0.7.x的版本。

这是一个稳定的处理程序，由Unbit提供商业支持。


Apache
------

.. seealso:: :doc:`Apache`

Apache2 `mod_uwsgi` 模块式第一个为uWSGI开发的web服务器集成模块。它稳定，但可以更好地与Apache API集成。

由Unbit提供商业支持。

自uWSGI 0.9.6-dev起，包含了一个名为 `mod_Ruwsgi` 的第二个Apache2模块。它更加Apache API友好。 *Unbit未对mod_Ruwsgi提供商业支持。*

在1.2开发周期中，添加了另一个名为 `mod_proxy_uwsgi` 模块。不久，这应该是基于Apache部署的最好的选择。


Lighttpd (试验性)
-----------------------

该模块是最新开发的，但是在官方的inclusion发布版本中其包含已经被拒了，因为主作者认为 :doc:`uwsgi protocol<Protocol>` 是“重复发明轮子” ，而建议使用一个FastCGI方法。我们尊重这种态度。这个模块会继续存在uWSGI源代码树中，但是当前未对其进行维护。

对于这个处理程序，当前并无商业支持。我们认为这个模块是“试验性的”。


Twisted
-------

这是一个"商用"处理程序，主要在不安装一个完整的web服务器的情况下测试应用的时候有用。如果你想要开发一个uWSGI服务器，那么看看这个模块。 :doc:`Twisted`.


Tomcat
------

可以使用包含的服务小程序来从Tomcat转发请求到uWSGI服务器。它稳定，但当前缺乏文档。

对于这个处理程序，当前并无商业支持。


CGI
---

CGI处理程序是用于“偷懒”安装的。不鼓励在生产环境上使用它们。


Cherokee (废弃)
-------------------

.. seealso:: :doc:`Cherokee`

Cherokee web服务器官方支持uWSGI。
Cherokee是快而且轻量的，拥有一个漂亮的admin界面，以及一个不错的社区。从一开始，他们对uWSGI的支持就很棒，在大多数的情况下，我们都推荐使用它。
Cherokee uWSGI处理程序的用户群可能是最大的。Cherokee uWSGI处理程序由Unbit提供商业支持。


Mongrel2 (废弃)
--------

.. seealso:: :doc:`Mongrel2`


自0.9.8-dev起，对 `Mongrel2 Project <http://mongrel2.org/>`_ 的支持通过 :doc:`ZeroMQ` 协议插件可用。

在我们的测试中，Mongrel2几乎通过了我们发送的所有负载。

非常棒和稳定的项目。试试吧 :) 
