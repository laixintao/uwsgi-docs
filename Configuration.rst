配置uWSGI
=================

可以使用一些不同的方法对uWSGI进行配置。在相同的uWSGI调用中，所有的配置方法或许是混合搭配的。

.. note:: 一些配置方法或许需要特定的插件 (例如，sqlite和ldap)。

.. seealso:: :doc:`ConfigLogic`

配置系统是统一的，因此，每一个命令行选项与配置文件中的项1:1对应。

例子：

.. code-block:: sh

   uwsgi --http-socket :9090 --psgi myapp.pl
   
可以写成

.. code-block:: ini

   [uwsgi]
   http-socket = :9090
   psgi = myapp.pl


.. _LoadingConfig:

加载配置文件
---------------------------

除了简单的磁盘文件，uWSGI支持通过多种方式加载配置文件::

  uwsgi --ini http://uwsgi.it/configs/myapp.ini # HTTP
  uwsgi --xml - # standard input
  uwsgi --yaml fd://0 # file descriptor
  uwsgi --json 'exec://nc 192.168.11.2:33000' # arbitrary executable

.. note::

  还能使用更多晦涩的文件源，例如 :doc:`Emperor<Emperor>`, 嵌入式结构 (两种), 动态库符号和ELF部分。

.. _MagicVars:

魔术变量
---------------

uWSGI配置文件可以包含“魔术”变量，以百分号%作为前缀。目前，定义了如下魔术变量 (你可以通过 :py:data:`uwsgi.magic_table` 在Python中访问它们)。

======== ==
%v       vassals目录 (pwd)
%V       uWSGI版本
%h       主机名
%o       原始配置文件名，在命令行指定
%O       与%o相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%p       配置文件的绝对路径
%P       与%p相同，但指的是第一个非模板配置文件 ，但指的是第一个非模板配置文件
         (1.9.18版本)
%s       配置文件名
%S       与%s相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%d       包含配置文件的目录的绝对路径
%D       与%d相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%e       配置文件的扩展名
%E       与%e相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%n       不带扩展名的文件名
%N       与%n相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%c       包含配置文件的目录名 (1.3+版本)
%C       与%c相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%t       unix时间 (以秒为单位，在实例启动时收集) (1.9.20-dev+版本)
%T       unix时间 (以微秒为单位，在实例启动时收集) (version 1.9.20-dev+)
%x       当前段的标识符，例如， `config.ini:section` (1.9-dev+版本)
%X       与%x相同，但指的是第一个非模板配置文件
         (1.9.18版本)
%i       文件的inode数字 (2.0.1版本)
%I       与%i相同，但指的是第一个非模板配置文件
%0..%9   包含配置文件的目录的全路径的一个特定的组件 (1.3+版本)
%[       ANSI转义"\\033" (对于打印颜色有用)
%k       检测到的cpu核数 (1.9.20-dev+版本)
%u       运行进程的用户的uid (2.0版本)
%U       运行进程的用户的用户名 (如果可用，否则回退到uid) (2.0版本)
%g       运行进程的用户的gid (2.0版本)
%G       运行进程的用户所在组名 (如果可用，否则回退到gid)  (2.0版本)
%j       完整配置路径的djb33x哈希的HEX表示
%J       与%j相同，但指的是第一个非模板配置文件

======== ==

注意，它们大多数指的是它们所在的文件，即使那个文件被包含在其他文件中。

一个例外是大部分的大写版本，指的是加载的第一个非模板配置文件。这意味着，第一个配置文件并不是通过 ``--include`` 或者 ``--inherit`` 加载的，而是通过例如 ``--ini``, ``--yaml`` 或者 ``--config`` 加载的。这些是为了使用emperor，来指向用 ``--vassals-include`` 或者 ``--vassals-inherit`` 包含的实际的vassal配置文件，而不是模板。

例如，这里是 :file:`funnyapp.ini` 。

.. code-block:: ini

  [uwsgi]
  socket = /tmp/%n.sock
  module = werkzeug.testapp:test_app
  processes = 4
  master = 1

``%n`` 将会使用配置文件的名字来替代，不加扩展名，因此，这个情况下的结果将会是

.. code-block:: ini

  [uwsgi]
  socket = /tmp/funnyapp.sock
  module = werkzeug.testapp:test_app
  processes = 4
  master = 1

.. _Placeholders:

占位符
------------

占位符是通过设置你自己发明的一个新的配置变量，在配置时间定义的自定义的魔术变量。

.. code-block:: ini

  [uwsgi]
  ; These are placeholders...
  my_funny_domain = uwsgi.it
  set-ph = max_customer_address_space=64
  set-placeholder = customers_base_dir=/var/www
  ; And these aren't.
  socket = /tmp/sockets/%(my_funny_domain).sock
  chdir = %(customers_base_dir)/%(my_funny_domain)
  limit-as = %(max_customer_address_space)

可以直接指定占位符，或者使用 ``set-placeholder``
/ ``set-ph`` 选项。后者在以下情况下有用：

* 更明显的表示出你在设置的是占位符，而不是常规选项。
* 在命令行设置选项，因为诸如
  ``--foo=bar`` 这样的未知选项会被拒绝，但是 ``--set-placeholder foo=bar`` 是可以的。
* 当启用strict模式的时候，设置占位符。

占位符是可访问的，像任何uWSGI选项一样，在你的应用代码中，通过 :py:data:`uwsgi.opt` 进行访问。

.. code-block:: python

  import uwsgi
  print uwsgi.opt['customers_base_dir']

这个特性可以被（滥）用来减少你的应用所需的配置文件的数目。

类似地，可以使用 `$(ENV_VAR)` 和 `@(file_name)` 语法来包含环境变量和外部文本文件的内容。又见
:doc:`ParsingOrder`.

占位符数学 (自uWSGI 1.9.20-dev起)
-----------------------------------------

你可以使用这种特殊的语法，应用数学公式到占位符上：

.. code-block:: ini

   [uwsgi]
   foo = 17
   bar = 30
   ; total will be 50
   total = %(foo + bar + 3)
   
运算符之间不要忘了空格。

在管道中（而不是一般的数学格式）执行运算：

.. code-block:: ini

   [uwsgi]
   foo = 17
   bar = 30
   total = %(foo + bar + 3 * 2)
   
'total' 将会被计算为100:

 (((foo + bar) + 3) * 2)
 
可以用递增和递减快捷方式

.. code-block:: ini

   [uwsgi]
   foo = 29
   ; remember the space !!!
   bar = %(foo ++)

bar将会是30

如果在两个项之间你不指定一个运算符，那么会假设使用“字符串连接”：

.. code-block:: ini

   [uwsgi]
   foo = 2
   bar = 9
   ; remember the space !!!
   bar = %(foo bar ++)
   
头两个项将会被计算为'29' (不是11，因为未指定任何数学运算)

'@'魔法
-------------

我们已经看到了可以使用@(filename)形式来包含一个文件的内容

.. code-block:: ini

   [uwsgi]
   foo = @(/tmp/foobar)
   
真相是，'@'可以读取来自所有支持的uwsgi方案的内容

.. code-block:: ini

   [uwsgi]
   ; read from a symbol
   foo = @(sym://uwsgi_funny_function)
   ; read from binary appended data
   bar = @(data://0)
   ; read from http
   test = @(http://example.com/hello)
   ; read from a file descriptor
   content = @(fd://3)
   ; read from a process stdout
   body = @(exec://foo.pl)
   ; call a function returning a char *
   characters = @(call://uwsgi_func)


命令行参数
----------------------

例子::

  uwsgi --socket /tmp/uwsgi.sock --socket 127.0.0.1:8000 --master --workers 3

.. _ConfigEnv:

环境变量
---------------------

当作为环境变量传递时，会把选项首字母大写，并加上
`UWSGI_` 前缀，而破折号会被下划线替代。

.. note::

   该方法并不支持相同配置变量的一些值。

例子::

   UWSGI_SOCKET=127.0.0.1 UWSGI_MASTER=1 UWSGI_WORKERS=3 uwsgi

INI文件
---------

.INI文件实际上是一种标准的配置格式，用在许多应用中。它由 ``[section]`` 和 ``key=value`` 对组成。

一个样例uWSGI INI配置：

.. code-block:: ini

  [uwsgi]
  socket = /tmp/uwsgi.sock
  socket = 127.0.0.1:8000
  workers = 3
  master = true

默认情况下，uWSGI使用 ``[uwsgi]`` 段，但是你可以在使用 ``filename:section`` 语法加载INI文件的时候指定另一个段名，也就是::

  uwsgi --ini myconf.ini:app1

作为选择，你可以通过省略文件名并只指定段名来从相同的文件中加载另一个段。注意，技术上来讲，这会从上个加载的.ini文件中加载命名段，而不是从当前的文件中加载，因此，当包含其他文件的时候，小心为上。

.. code-block:: ini

  [uwsgi]
  # This will load the app1 section below
  ini = :app1
  # This will load the defaults.ini file
  ini = defaults.ini
  # This will load the app2 section from the defaults.ini file!
  ini = :app2

  [app1]
  plugin = rack

  [app2]
  plugin = php

* 行内空格是不重要的。
* 以分号 (``;``) 或者一个哈希/井号 (``#``) 开头的行会被当成注释忽略。
* 可以在没有值那个部分的情况下设置布尔型的值。因此，简单的 ``master`` 等价于 ``master=true`` 。这或许与其他INI解析器，例如 ``paste.deploy`` ，并不兼容。
* 为了方便，uWSGI特殊识别裸 ``.ini`` 参数，因此，调用 ``uwsgi myconf.ini``  等价于 ``uwsgi --ini myconf.ini`` 。

XML文件
---------

根节点应该是 ``<uwsgi>`` ，而选项值是节点文本。


一个例子：

.. code-block:: xml

  <uwsgi>
    <socket>/tmp/uwsgi.sock</socket>
    <socket>127.0.0.1:8000</socket>
    <master/>
    <workers>3</workers>
  </uwsgi>

文件中还可以有多个 ``<uwsgi>`` 节，由不同的 ``id`` 属性标记。要选择使用的节，则在 ``xml`` 选项中的文件名之后指定其id，使用冒号作为分隔符。当使用这种 `id` 模式时，文件的根节点可以是任何你喜欢的。这会允许你将 ``uwsgi`` 配置节点嵌入到其他XML文件中。

.. code-block:: xml

  <i-love-xml>
    <uwsgi id="turbogears"><socket>/tmp/tg.sock</socket></uwsgi>
    <uwsgi id="django"><socket>/tmp/django.sock</socket></uwsgi>
  </i-love-xml>

* 可在没有文本值的情况下设置布尔类型的值。
* 为了方便起见，uWSGI特殊识别裸 ``.ini`` 参数，因此，调用 ``uwsgi myconf.xml``  等价于 ``uwsgi --xml myconf.xml`` 。

JSON 文件
----------

JSON应该使用一个键值对来表示一个对象，键是
`"uwsgi"` ，而值是配置变量的一个对象。支持原生JSON列表、布尔型和数字。

一个例子：

.. code-block:: json

  {"uwsgi": {
    "socket": ["/tmp/uwsgi.sock", "127.0.0.1:8000"],
    "master": true,
    "workers": 3
  }}

再次，可以使用文件名后冒号来加载一个命名段。

.. code-block:: json

  {"app1": {
    "plugin": "rack"
  }, "app2": {
    "plugin": "php"
  }}

然后这样加载::

  uwsgi --json myconf.json:app2

.. note::

   The `Jansson`_ library is required during uWSGI build time to enable JSON
   support.  By default the presence of the library will be auto-detected and
   JSON support will be automatically enabled, but you can force JSON support
   to be enabled or disabled by editing your build configuration.

   .. seealso:: :doc:`Install`

.. _Jansson: http://www.digip.org/jansson/

YAML 文件
----------

根元素应该是 `uwsgi` 。布尔型选项要设置成 `true` 或 `1` 。

一个例子：

.. code-block:: yaml

  uwsgi:
    socket: /tmp/uwsgi.sock
    socket: 127.0.0.1:8000
    master: 1
    workers: 3

另外，一个命名段可以使用文件名后的冒号进行加载。

.. code-block:: yaml

  app1:
    plugin: rack
  app2:
    plugin: php

然后这样加载它::

  uwsgi --yaml myconf.yaml:app2


SQLite配置
--------------------

.. note::

  构建中……

LDAP 配置
------------------

LDAP是集中uWSGI服务器大型集群配置的一种灵活的方式。配置它是一个负责的主题。见 :doc:`LDAP` 以获得更多信息。
