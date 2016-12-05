SSI (服务器端包含，Server Side Includes) 插件
=================================

服务器端包含是一种编写动态web页面的“老式”方法。

通常把它当成一个模板系统，而不是一个全功能语言。

uWSGI SSI插件的主要目的是拥有一个可以访问uWSGI API的快速的模板系统。

在写这篇文章的时候，即2013年3月，这个插件还处于测试版本，实现了少于30%的SSI标准，关注点在于把uWSGI API当成SSI命令公开。

把它当成一个请求处理器使用
*****************************

这个插件有一个官方的modifier1，数字19。

.. code-block:: ini

   [uwsgi]
   plugin = ssi
   http = :9090
   http-modifier1 = 19
   http-var = DOCUMENT_ROOT=/var/www

该插件把文件名作为 ``DOCUMENT_ROOT``+``PATH_INFO`` 构建。然后，将这个文件作为服务器端包含文档解析。

 ``DOCUMENT_ROOT`` 和 ``PATH_INFO`` 都是必须的，否则将会返回一个500错误。

Nginx的一个样例配置如下：

.. code-block:: c

   location ~ \.shtml$ {
       root /var/www;
       include uwsgi_params;
       uwsgi_pass 127.0.0.1:3031;
       uwsgi_modifier1 19;
   }

而对于uWSGI大概像这样……

.. code-block:: ini

   [uwsgi]
   plugin = ssi
   socket = 127.0.0.1:3031

把SSI作为一个路由动作使用
*****************************

更通用的方式是把SSI解析器作为一个路由动作使用。

.. code-block:: ini

   [uwsgi]
   plugin = ssi
   http-socket = :9090
   route = ^/(.*) ssi:/var/www/$1.shtml

.. warning:: 对于所有的路由动作，文件路径上并没有运行更高层次自定义的检查。如果你传递不信任路径给SSI动作，那么你应该清理它们 (你可以再次使用路由，检查..的存在，或者其他危险符号)。

并且考虑到上述告诫，当作为路由动作使用时， ``DOCUMENT_ROOT`` 或者 ``PATH_INFO`` 并不是必须的，因为传递的参数包含了完整的文件系统路径。

支持的SSI命令
**********************

这是支持的命令列表 (及其参数)。如果一个命令并非SSI标准的一部分 (也就是说，它是uWSGI特有的)，那么将会报告它。

echo
^^^^

参数： ``var``

打印指定请求变量的内容。

printenv
^^^^^^^^

打印所有请求变量的列表。

include
^^^^^^^

参数： ``file``

包含指定文件 (相对于当前目录)。

cache
^^^^^

.. note:: 这是uWSGI特有的/非标准的。

参数： ``key`` ``name``

打印命名缓存中指定缓存键的值。

状态
******

* 这个插件是完全线程安全的，并且非常快。
* 非常少的命令可用，不久会添加更多命令。
