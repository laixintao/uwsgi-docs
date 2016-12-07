为你的实例定义新选项
=======================================

有时，内置选项并不够。例如，你可能需要给予你的客户自定义选项，让他们在你的平台上配置他们的应用。或者，你需要配置辣么多的实例，因此你想要简化一些选项，例如per-datacenter或者per-server-type。为你的配置文件/命令行声明新的选项是达到这些目标的一个好方法。

要定义一个新选项，使用 ``--declare-option``::

  --declare-option <option_name>=<option1=value1>[;<option2=value2>;<option3=value3>...]


一个有用的例子可以是定义一个"redirect"选项，使用内部路由子系统中的重定向插件::

  --declare-option "redirect=route=\$1 redirect:\$2"


这将会声明一个新的选项，名为 ``redirect`` ，它接收2个参数。将会使用$-prefixed变量对那些参数进行扩展。就像shell脚本一样， *需要使用反斜杠，从而让你的shell不扩展这些值* 。

现在，你就能在你的配置文件中定义重定向了：

.. code-block:: sh

  uwsgi --declare-option "redirect=route=\$1 redirect:\$2" --ini config.ini

Config.ini:

.. code-block:: ini

  [uwsgi]
  socket = :3031
  ; define my redirects
  redirect = ^/foo http://unbit.it
  redirect = \.jpg$ http://uwsgi.it/test
  redirect = ^/foo/bar/ /test

或者直接在命令行：

.. code-block:: sh

  uwsgi --declare-option "redirect=route=\$1 redirect:\$2" --socket :3031 --redirect "^/foo http://unbit.it" --redirect "\.jpg$ http://uwsgi.it/test" --redirect "^/foo/bar/ /test"

更好玩：一堆快捷方式
------------------------------

现在，我们将为常用应用定义新选项。

Shortcuts.ini:

.. code-block:: ini

  [uwsgi]
  ; let's define a shortcut for trac (new syntax: trac=<path_to_trac_instance>)
  declare-option = trac=plugin=python;env=TRAC_ENV=$1;module=trac.web.main:dispach_request
  ; one for web2py (new syntax: web2py=<path_to_web2_py_dir>)
  declare-option = web2py=plugin=python;chdir=$1;module=wsgihandler
  ; another for flask (new syntax: flask=<path_to_your_app_entry_point>)
  declare-option = flask=plugin=python;wsgi-file=$1;callable=app

要把一个Trac实例连接到/var/www/trac/fooenv:

.. code-block:: ini

  [uwsgi]
  ; include new shortcuts
  ini = shortcuts.ini
  
  ; classic options
  http = :8080
  master = true
  threads = 4
  
  ; our new option
  trac = /var/www/trac/fooenv

一个用于Web2py的配置，以XML格式：

.. code-block:: xml

  <uwsgi>
    <!-- import shortcuts -->
    <ini>shortcuts.ini</ini>
    <!-- run the https router with HIGH ciphers -->
    <https>:443,test.crt,test.key,HIGH</https>
  
    <master/>
    <processes>4</processes>
  
    <!-- load web2py from /var/www/we2py -->
    <web2py>/var/www/we2py</web2py>
  </uwsgi>

Emperor的一个小技巧：自动为你的vassal导入快捷方式
------------------------------------------------------------------------

如果你使用 :doc:`Emperor<Emperor>` 来管理你的客户/用户，那么你可以配置它来自动在每个vassal中导入你的快捷方式。

.. code-block:: sh

  uwsgi --emperor /etc/uwsgi/vassals --vassals-include /etc/uwsgi/shortcuts.ini


对于多个快捷方式，使用：

.. code-block:: sh

  uwsgi --emperor /etc/uwsgi/vassals --vassals-include /etc/uwsgi/shortcuts.ini --vassals-include /etc/uwsgi/shortcuts2.ini --vassals-include /etc/uwsgi/shortcuts3.ini

或者 (使用一点 :doc:`configuration logic magic<ConfigLogic>`):

.. code-block:: ini

  [uwsgi]
  emperor = /etc/uwsgi/vassals
  
  for = shortcuts shortcuts2 shortcuts3
    vassals-include = /etc/uwsgi/%(_).ini
  endfor =

一个绝招：嵌入快捷方式到你的uWSGI二进制文件中
-----------------------------------------------------------

uWSGI的构建系统允许你嵌入文件，无论是普通的文件还是配置，到服务器二进制文件中。尽情使用此特性将让你可以嵌入新的选项快捷方式到服务器二进制文件中，自动允许用户使用它们。要嵌入你的快捷方式文件，则编辑你的构建配置文件 (例如
:file:`buildconf/base.ini`) ，然后设置 ``embed_config`` 为快捷方式文件的路径。重新构建你的服务器，然后你新的选项就可以用了。

.. seealso:: :doc:`BuildConf`
