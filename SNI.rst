SNI - 服务器名称识别 (SSL节点的虚拟主机)
================================================================

uWSGI 1.9 (代号为"ssl as p0rn") 添加了对SNI (服务器名称识别，Server Name Identification) 的支持，贯穿整个SSL子系统。HTTPS路由器，SPDY路由器和SSL路由器都可以透明使用它。

SNI是SSL标准的一个扩展，它允许客户端为它想要的资源指定一个“名字”。名字通常是请求主机名，因此你可以像使用HTTP ``Host:`` 头部那样实现类虚拟主机行为，而无需而外的IP地址等。

在uWSGI中，一个SNI对象是由名字和值组成的。名字是服务器名/主机名，而值是“SSL上下文” (你可以把它想成特定领域的证书、密钥和密码的总和)。

添加SNI对象
******************

要添加一个SNI对象，只需使用 ``--sni`` 选项：

.. code-block:: sh

   --sni <name> crt,key[,ciphers,client_ca]

例如：

.. code-block:: sh

   --sni "unbit.com unbit.crt,unbit.key"

或者对于基于客户端的SSL鉴权和OpenSSL HIGH加密级别

.. code-block:: sh

   --sni "secure.unbit.com unbit.crt,unbit.key,HIGH,unbit.ca"

添加复杂SNI对象
**************************

有时，对于你的SNI对象，你需要更复杂的密钥 (例如当使用通配符证书时)

如果你构建了带PCRE/正则表达式支持的uWSGI (你应该会)，那么你可以使用 ``--sni-regexp`` 选项。

.. code-block:: sh

   --sni-regexp "*.unbit.com unbit.crt,unbit.key,HIGH,unbit.ca"

海量SNI托管
*******************

uWSGI的主要目的之一是海量托管，因此，不支持这样的SSL将会让人相当郁闷。

如果你有映射到相同的IP地址的几十个 (或者对于这个问题，几百个) 证书，那么你可以简单将它们放在一个目录中 (遵循一个简单的约定，我们将复杂一点) ，然后每当需要为一个域名查找环境的时候，让uWSGI扫描它。

要添加一个目录，仅需使用

.. code-block:: sh

   --sni-dir <path>

例如

.. code-block:: sh

   --sni-dir /etc/customers/certificates

现在，如果你有 ``unbit.com`` 和 ``example.com`` 证书= (.crt) ，以及密钥 (.key)，那么仅需把它们放在那里，遵循以下命名规则：

* ``/etc/customers/certificates/unbit.com.crt``
* ``/etc/customers/certificates/unbit.com.key``
* ``/etc/customers/certificates/unbit.com.ca``
* ``/etc/customers/certificates/example.com.crt``
* ``/etc/customers/certificates/example.com.key``

正如你所看到的， ``example.com`` 没有.ca文件，因此将会对其禁用客户端鉴权。

如果你想要强制设置一个默认的密码到SNI上下文中，那么使用

.. code-block:: sh

   --sni-dir-ciphers HIGH

(或者任何其他你需要的值)

注：不支持卸载SNI对象。一旦将其加载到内存，就会一直持有它们，直到重载。

订阅系统和SNI
***************************

uWSGI 2.0在订阅系统中添加了对SNI的支持。

https/spdy路由器和sslrouter可以动态加载来自一个订阅包指定的路径的证书和密钥：

.. code-block:: sh

   uwsgi --subscribe2 key=mydomain.it,socket=0,sni_key=/foo/bar.key,sni_crt=/foo/bar.crt
   
   
这个路由器将会基于指定的文件（确保路由器可以访问它们）创建一个新的SSL上下文，并将当最后一个节点断开连接的时候会销毁它。

这对于海量托管有用，其中，客户的证书位于家目录下，并且你想要他们不用找你就可以修改/更新那些文件。

.. note::

   我们明白，直接在订阅包中封装密钥和证书会更有用得多，但是从安全的角度来看，密钥的网络传输真的有点蠢。我们正在研究将它与安全订阅系统（其中，每个包都被加密）结合起来是否是一个解决方法。
