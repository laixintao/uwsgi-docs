格式化uWSGI请求日志
==============================

uWSGI有一个 ``--logformat`` 选项，用来构建自定义的请求日志行。语法很简单：

.. code-block:: ini

   [uwsgi]
   logformat = i am a logline reporting "%(method) %(uri) %(proto)" returning with status %(status)

所有用 ``%()`` 标记的变量都可以使用特定的规则来替代。定义了三种日志变量 ("offsetof", 函数和用户定义)。

offsetof
********

这些是一味从当前请求内部的 ``wsgi_request`` 结构获取的。

* ``%(uri)`` -> REQUEST_URI
* ``%(method)`` -> REQUEST_METHOD
* ``%(user)`` -> REMOTE_USER
* ``%(addr)`` -> REMOTE_ADDR
* ``%(host)`` -> HTTP_HOST
* ``%(proto)`` -> SERVER_PROTOCOL
* ``%(uagent)`` -> HTTP_USER_AGENT (自1.4.5起)
* ``%(referer)`` -> HTTP_REFERER (自1.4.5起)

函数
*********

这些是简单的函数，调用来生成日志变量值：

* ``%(status)`` -> HTTP响应状态码
* ``%(micros)`` -> 响应时间，以微秒为单位
* ``%(msecs)`` -> 响应时间，以毫秒为单位
* ``%(time)`` -> 请求开始的时间戳
* ``%(ctime)`` -> 请求开始的ctime
* ``%(epoch)`` -> Unix格式的当前时间
* ``%(size)`` -> 响应体大小 + 响应头大小 (自1.4.5起)
* ``%(ltime)`` -> 人类可读(Apache风格)的请求时间 (自1.4.5起)
* ``%(hsize)`` -> 响应头大小 (自1.4.5起)
* ``%(rsize)`` -> 响应体大小 (自1.4.5起)
* ``%(cl)`` -> 请求内容体大小 (自1.4.5起)
* ``%(pid)`` -> 处理请求的worker的pid (自1.4.6起)
* ``%(wid)`` -> 处理请求的worker的id (自1.4.6起)
* ``%(switches)`` -> 异步切换数 (自1.4.6起)
* ``%(vars)`` -> 请求中的CGI变量数 (自1.4.6起)
* ``%(headers)`` -> 已生成的响应头数 (自1.4.6起)
* ``%(core)`` -> 运行请求的核心 (自1.4.6起)
* ``%(vsz)`` -> 地址空间/虚拟内存使用 (单位为字节) (自1.4.6起)
* ``%(rss)`` -> RSS内存使用 (单位为字节)  (自1.4.6起)
* ``%(vszM)`` -> 地址空间/虚拟内存使用 (单位为MB) (自1.4.6起)
* ``%(rssM)`` -> RSS内存使用 (单位为MB) (自1.4.6起)
* ``%(pktsize)`` -> 内部的请求uwsgi包大小 (自1.4.6起)
* ``%(modifier1)`` -> 请求的modifier1 (自1.4.6起)
* ``%(modifier2)`` -> 请求的modifier2 (自1.4.6起)
* ``%(metric.XXX)`` -> 访问XXX度量值 (见 :doc:`Metrics`)
* ``%(rerr)`` -> 请求的读错误数 (自1.9.21起)
* ``%(werr)`` -> 请求的写错误数 (自1.9.21起)
* ``%(ioerr)`` -> 请求的读写错误数 (自1.9.21起)
* ``%(tmsecs)`` -> 请求开始时间戳，自纪元起，单位为毫秒 (自1.9.21起)
* ``%(tmicros)`` -> 请求开始时间戳，自纪元起，单位为微秒 (自1.9.21起)
* ``%(var.XXX)`` -> 请求变量XXX的内容 (例如var.PATH_INFO，自1.9.21起可用)

用户定义的日志变量
********************

你可以在你的请求处理函数中定义日志变量。这些变量只存活在每个请求中。

.. code-block:: python

   import uwsgi
   def application(env, start_response):
       uwsgi.set_logvar('foo', 'bar')
       # returns 'bar'
       print uwsgi.get_logvar('foo')
       uwsgi.set_logvar('worker_id', str(uwsgi.worker_id()))
       ...

使用以下日志格式，你将能够访问代码定义的日志变量：

.. code-block:: sh

   uwsgi --logformat 'worker id = %(worker_id) for request "%(method) %(uri) %(proto)" test = %(foo)'

Apache风格相结合的请求日志记录
*************************************

要生成兼容Apache的日志：

.. code-block:: ini

   [uwsgi]
   ...
   log-format = %(addr) - %(user) [%(ltime)] "%(method) %(uri) %(proto)" %(status) %(size) "%(referer)" "%(uagent)"
   ...

hack日志格式
*****************

(更新至1.9.21)

你可以这样注册新的"logchunk" (为每个日志格式符号调用的函数)

.. code-block:: c

   struct uwsgi_logchunk *uwsgi_register_logchunk(char *name, ssize_t (*func)(struct wsgi_request *, char **), int need_free);

* ``name`` -- 符号名
* ``need_free`` -- 如果是1，表示由 ``func`` 设置的指针必须释放（free()）
* ``func`` -- 日志处理函数中调用的函数

.. code-block:: c

   static ssize_t uwsgi_lf_foobar(struct wsgi_request *wsgi_req, char **buf) {
           *buf = uwsgi_num2str(wsgi_req->status);
           return strlen(*buf);
   }

   static void register_logchunks() {
           uwsgi_register_logchunk("foobar", uwsgi_lf_foobar, 1);
   }
   
   struct uwsgi_plugin foobar_plugin = {
           .name = "foobar",
           .on_load = register_logchunks,
   };
   
现在，如果你加载foobar插件，那么，你将能够使用 %(foobar) 请求日志变量 (它会报告请求状态)。
