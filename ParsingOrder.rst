uWSGI是如何解析配置文件的
=============================

直到uWSGI 1.1，解析顺序都是不“稳定”或者“可靠的”。

从uWSGI 1.1开始 (多亏了它新的选项子系统)，我们有了一个一般的规则：从上到下，尽快扩展。

从上到下意味着选项根据它们解析的顺序内部派系，而“尽快扩展”表示注入一个请求的配置文件的选项，中断当前解析的配置文件：

注意， ``inherit`` 选项与其他包含选项不同：它是在变量扩展 *之后* 进行扩展的，因此，任何环境变量，外部文件和占位符并 *未* 被扩展。魔术变量 (*例如* ``%n``) 会被正常进行扩展。

file1.ini (从命令行请求的那个文件)


.. code-block:: ini

   [uwsgi]
   socket = :3031
   ini = file2.ini
   socket = :3032
   chdir = /var/www
   
file2.ini

.. code-block:: ini

   [uwsgi]
   master = true
   memory-report = true
   processes = 4
   
内部将会被组装成：


.. code-block:: ini

   [uwsgi]
   socket = :3031
   ini = file2.ini
   master = true
   memory-report = true
   processes = 4
   socket = :3032
   chdir = /var/www
   
一个更复杂的例子：

file1.ini (从命令行请求的那个文件)

.. code-block:: ini

   [uwsgi]
   socket = :3031
   ini = file2.ini
   socket = :3032
   chdir = /var/www
   
file2.ini

.. code-block:: ini

   [uwsgi]
   master = true
   xml = file3.xml
   memory-report = true
   processes = 4
   
file3.xml

.. code-block:: xml

   <uwsgi>
     <plugins>router_uwsgi</plugins>
     <route>^/foo uwsgi:127.0.0.1:4040,0,0</route>
   </uwsgi>
   
结果将会是：

.. code-block:: ini

   [uwsgi]
   socket = :3031
   ini = file2.ini
   master = true
   xml = file3.xml
   plugins = router_uwsgi
   route = ^/foo uwsgi:127.0.0.1:4040,0,0
   memory-report = true
   processes = 4
   socket = :3032
   chdir = /var/www
   

扩展变量/占位符
********************************

在组装了内部配置树之后，将会应用变量和占位符替换。

第一步是用环境变量的值VALUE来替换所有出现的 $(VALUE) 。

.. code-block:: ini

   [uwsgi]
   foobar = $(PATH)
   
foobar值将会是shell的PATH变量的内容

第二步将会扩展在 @(FILENAME) 中括起来的文本文件。

.. code-block:: ini

   [uwsgi]
   nodename = @(/etc/hostname)
   
nodename值将会是/etc/hostname的内容

最后一步是占位符替换。一个占位符是另一个选项的引用：

.. code-block:: ini

   [uwsgi]
   socket = :3031
   foobar = %(socket)
   

foobar的内容将会被映射为socket的内容。

魔术变量说明
*************************

配置文件，支持另一种形式的变量，被称为“魔术”变量。当它们指向配置文件本身的时候，它们将会被尽快解析：


.. code-block:: ini

   [uwsgi]
   my_config_file = %p
   

my_config_file的内容一旦被解析，它将会被设置为%p的值 (当前文件的绝对路径)。这意味着%p (或者任何你需要的魔术变量)在当前解析的配置文件中将永远是一致的。
