uWSGI 1.9.1
===========

1.9版本树的第一个小版本。

错误修复
********

修复优雅重载后的--req-logger

修复carbon插件的一种崩溃

修复多worker以及适当的写时复制时的信号处理

修复Rack插件中的异常处理

XSLT插件
***************

添加了 :doc:`XSLT<XSLT>` 插件。它让我们通过请求插件或者 :doc:`InternalRouting` 应用XML转换。

Legion scrolls api
******************

scrolls是附加到 :doc:`Legion<Legion>` 集群中的每个成员上的文本。我们正在慢慢地定义一个API，以允许开发者直接在他们的应用和配置中使用legion子系统。1.9.1中添加的是uwsgi.scrolls(legion)函数，它返回由整个集群定义的当前scrolls的一个列表/数组。这还不是完全可用（有用）的，后续还会有更多的函数……

按需vassals
*****************

对于大规模托管的更好的资源使用的另一步。你现在可以告诉 :doc:`Emperor<Emperor>` 只在对指定的socket进行的第一个请求之后才启动vassal。与 --idle/--die-on-idle 选项相结合，就可以拥有完全按需的应用了。

要为每个vassal定义等待的socket，你有3个选项：

--emperor-on-demand-extension <ext>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这将会指示Emperor去检查一个名为<vassal>+<ext>的文件，如果该文件可用，那么将会读取，并把其内容作为socket等待的内容：

.. code-block:: sh

   uwsgi --emperor /etc/uwsgi/vassals --emperor-on-demand-extension .socket

假设有一个在/etc/uwsgi/vassals中的myapp.ini文件，那么将会搜索/etc/uwsgi/vassals/myapp.ini.socket (而它的内容会用作这个socket的名字)

在第一次连接的时候，会生成vassal，并且会将socket作为文件描述符0进行传递。文件描述符0总是会被uWSGI检查，因此你并不需要在vassal文件中指定一个--socket选项。这自动对uwsgi socket有效，如果你使用其他协议 (例如http或者fastcgi)，那么你必须使用--protocol选项来指定它

--emperor-on-demand-directory <dir>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是较不通用的方法，只支持UNIX socket。基本上，vassal的名字 (不带扩展名和路径)会被附加到指定的目录，并带上.socket扩展名，然后作为按需 socket使用：

.. code-block:: sh

   uwsgi --emperor /etc/uwsgi/vassals --emperor-on-demand-directory /var/tmp

使用前一个例子，socket /var/tmp/myapp.socket将会被自动绑定

--emperor-on-demand-exec <cmd>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

只是你（非常有可能）用在很大的部署中。每次添加了一个新的vassal，就会运行提供的命令，并把vassal名当做第一个参数传递。命令的STDOUT作为socket名使用。

--exec-post-app钩子
************************

除了其他--exec-*选项 (用来在不同的服务器阶段运行命令)，添加了一个新的选项，以允许你在加载一个应用之后运行命令。

pyring构建配置文件
************************

这是一个非常特别的构建配置文件，允许你自动构建一个带完整python支持和模块化jvm + ring执行的虚拟环境（注：原文是jvm + ring honouring virtualenvs）的uWSGI栈。

缓存路由器插件
***********************

这已经被改善了，在下一个发布版本中，我们应该能够只使用内部路由子系统就可以直接将响应存储在uWSGI缓存中

稍后将会放出文档

crypto记录器
*****************

如果你在云服务上托管你的应用，并且没有永久存储，那么你可能想要发送日志到外部系统。忧伤的是，日志常常包含敏感信息，你不应该明文传输它们。新的crypto记录器试着解决这个问题，它允许你加密每个日志包，然后通过udp将它发送给能够解密它的服务器。

在下面的例子中

.. code-block:: sh

   uwsgi --plugin logcrypto --logger crypto:addr=192.168.173.22:1717,algo=bf-cbc,secret=ciaociao -M -p 4 -s :3031

将会发送每个日志包给192.168.173.22:1717指定的udp服务器，根据'ciaociao'密钥，使用blowfish cbc算法来加密文本。

这里是一个可用的样例服务器：

https://github.com/unbit/uwsgi/blob/master/contrib/cryptologger.rb


rpc内部路由指令
************************************

添加了"rpc"路由指令，允许你直接从路由子系统调用rpc函数，并将它们的输出转发到客户端。

看看下面的例子：

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   route = ^/foo addheader:Content-Type: text/html
   route = ^/foo rpc:hello ${REQUEST_URI} ${HTTP_USER_AGENT}
   route = ^/bar/(.+)$ rpc:test $1 ${REMOTE_ADDR} uWSGI %V
   route = ^/pippo/(.+)$ rpc:test@127.0.0.1:4141 $1 ${REMOTE_ADDR} uWSGI %V
   import = funcs.py

carbon插件对名字解析的初步支持
***********************************************************

你可以使用主机名来指定carbon服务器。当前的代码是非常简单的。以后的更新将会支持轮询。

新的路由条件
**********************

添加了新的路由条件 (equal,startswith,endswith,regexp)，看看更新的文档：

https://uwsgi-docs.readthedocs.io/en/latest/InternalRouting.html#the-internal-routing-table

'V'魔术变量
*****************

你可以在配置中使用%V魔术变量来引用uWSGI版本字符串

'mongodb'一般插件
****************************

这是一个用于不能访问共享libmongoclient的packager的商品插件。这基本上在一个可以被其他mongodb插件使用的新的共享对象中链接它。

通过网络构建profile
***************************

现在，你可以使用url（支持http, https和ftp）来引用构建配置文件了：

.. code-block:: sh

   UWSGI_PROFILE=http://uwsgi.it/psgi.ini make


获取它
******

uWSGI 1.9.1于20130324发布

你可以从下面地址下载它：

http://projects.unbit.it/downloads/uwsgi-1.9.1.tar.gz



