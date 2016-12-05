XSLT插件
===============

自uWSGI 1.9.1起，有了一个新的名为"xslt"的插件，它同时作为请求处理器和路由指令实现了XML样式表。

想成功应用转换，你需要一个“文档” (一个XML文档) 和一个样式表 (XSLT文件)。

此外，你可以应用全局参数，并设置一个指定的内容类型 (默认情况下，生成的输出被设置为text/html)。

请求处理器
*******************

Modifier1 23已被赋给XSLT请求处理器。

文档路径是通过将 ``PATH_INFO`` 附加到 ``DOCUMENT_ROOT`` 来创建的。

样式表路径根据以下步骤创建：

* 如果设置了一个指定的CGI变量 (通过 ``--xslt-var``)，那么将把它当成样式表路径使用。
* 如果命名像文档加上指定的扩展名 (默认情况下，搜索 ``.xsl`` 和 ``.xslt``) 的文件存在，那么将会把它当成样式表路径使用。
* 最后，尝试一系列静态XSLT文件 (由 ``--xslt-stylesheet`` 指定)。

例如：

.. code-block:: sh

   uwsgi --http-socket :9090 --http-socket-modifier1 23 --xslt-ext .bar

如果请求/foo.xml (并且文件存在)，那么 ``DOCUMENT_ROOT``+``foo.xml.bar`` 将会作为xslt文件被搜索。

.. code-block:: sh

   uwsgi --http-socket :9090 --http-socket-modifier1 23 --xslt-stylesheet /var/www/myfile1.xslt --xslt-stylesheet /var/www/myfile2.xslt

如果请求/foo.xml (并且文件存在)，那么将会试着用 ``/var/www/myfile1.xslt`` 。如果它不存在，那么将会用 ``/var/www/myfile2.xslt`` 来替代。

.. code-block:: sh

   uwsgi --http-socket :9090 --http-socket-modifier1 23 --xslt-var UWSGI_XSLT

如果请求/foo.xml (并且文件存在)，那么 ``UWSGI_XSLT`` 变量 (你可以从web服务器对其进行设置) 的内容就会被当成样式表路径使用。

如果 ``QUERY_STRING`` 可用，那么它的项将会作为全局参数传递给样式表。

例如，如果你请求 ``/foo.xml?foo=bar&test=uwsgi`` ，那么"foo" (值为"bar"）和 "test" (值为"uwsgi")将会作为全局变量被传递：

.. code-block:: xml

   <xsl:value-of select="$foo"/>
   <xsl:value-of select="$test"/>

路由指令
***********************

该插件将自身注册为名为"xslt"的内部路由指令。它或许比请求插件更通用得多。

它的语法相当简单：

.. code-block:: ini

   [uwsgi]
   plugin = xslt
   route = ^/foo xslt:doc=${DOCUMENT_ROOT}/${PATH_INFO}.xml,stylesheet=/var/www/myfunction.xslt,content_type=text/html,params=foo=bar&test=unbit

这将会应用 ``/var/www/myfunction.xslt`` 转换到 ``foo.xml`` ，并将其作为 ``text/html`` 返回。

该路由指令唯一需要的参数是 ``doc`` 和 ``stylesheet`` 。
