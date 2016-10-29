配置逻辑
===================

自1.1起，可以使用某些逻辑构建。

目前支持以下语句：

* ``for`` .. ``endfor``
* ``if-dir`` / ``if-not-dir``
* ``if-env`` / ``if-not-env``
* ``if-exists`` / ``if-not-exists``
* ``if-file`` / ``if-not-file``
* ``if-opt`` / ``if-not-opt``
* ``if-reload`` / ``if-not-reload`` —— 未公开

以上每个语句都导出了一个上下文值，你可以通过特殊的占位符 ``%(_)`` 来访问它们。例如，"for"语句设置 ``%(_)`` 为当前的迭代值。

.. warning:: 不支持递归逻辑，它会导致uWSGI即刻退出。

for
---

for迭代由空格隔开的字符串。以下三个代码块是等价的。

.. code-block:: ini

  [uwsgi]
  master = true
  ; iterate over a list of ports
  for = 3031 3032 3033 3034 3035
  socket = 127.0.0.1:%(_)
  endfor =
  module = helloworld


.. code-block:: xml

  <uwsgi>
    <master/>
    <for>3031 3032 3033 3034 3035</for>
      <socket>127.0.0.1:%(_)</socket>
    <endfor/>
    <module>helloworld</module>
  </uwsgi>


.. code-block:: sh

  uwsgi --for="3031 3032 3033 3034 3035" --socket="127.0.0.1:%(_)" --endfor --module helloworld

注意，for循环是分别被应用到块中的每一行的，而不是应用到块整体上。例如，这个：

.. code-block:: ini

  [uwsgi]
  for = a b c
  socket = /var/run/%(_).socket
  http-socket = /var/run/%(_)-http.socket
  endfor =

展开为：

.. code-block:: ini

  [uwsgi]
  socket = /var/run/a.socket
  socket = /var/run/b.socket
  socket = /var/run/c.socket
  http-socket = /var/run/a-http.socket
  http-socket = /var/run/b-http.socket
  http-socket = /var/run/c-http.socket

if-env
------

检查是否定义了一个环境变量，将其值放在上下文占位符中。

.. code-block:: ini

  [uwsgi]
  if-env = PATH
  print = Your path is %(_)
  check-static = /var/www
  endif =
  socket = :3031

if-exists
---------

检查一个文件或目录是否存在。设置上下文占位符为找到的文件名。

.. code-block:: ini

  [uwsgi]  
  http = :9090
  ; redirect all requests if a file exists
  if-exists = /tmp/maintenance.txt
  route = .* redirect:/offline
  endif =

.. note:: 上述例子使用 :doc:`InternalRouting`.

if-file
-------

检查给定的路径是否存在，是否为一个常规的文件。设置上下文占位符为找到的文件名。

.. code-block:: xml

  <uwsgi>
    <plugins>python</plugins>
    <http-socket>:8080</http-socket>
    <if-file>settings.py</if-file>
      <module>django.core.handlers.wsgi:WSGIHandler()</module>
    <endif/>
  </uwsgi>

if-dir
------

检查给定的路径是否存在，是否为一个目录。设置上下文占位符为找到的文件名。

.. code-block:: yaml

  uwsgi:
    socket: 4040
    processes: 2
    if-dir: config.ru
    rack: %(_)
    endif:

if-opt
------
检查是否设置了给定选项，或者给定选项是否具有一个给定的值。设置上下文占位符为选项引用的值。

检查是否设置了一个选项，仅需将选项名传递给 ``if-opt`` 。

.. code-block:: yaml

  uwsgi:
    cheaper: 3
    if-opt: cheaper
    print: Running in cheaper mode, with initially %(_) processes
    endif:

要检查给定选项是否具有一个给定的值，则将
``option-name=value`` 传递给 ``if-opt`` 。

.. code-block:: yaml

  uwsgi:
    # Set busyness parameters if it was chosen
    if-opt: cheaper-algo=busyness
    cheaper-busyness-max: 25
    cheaper-busyness-min: 10
    endif:

由于uWSGI解析其配置文件的方式，你只能使用uWSGI之前解析到的选项。特别是，这意味着：

* 只考虑在 ``if-opt`` 选项之上设置的选项。这包含了任何由前面 ``include`` (或者特定类型的包含，例如 ``ini``) 选项设置的任何选项，但不要包含由前面 ``inherit`` 选项设置的选项)。
* ``if-opt`` 是在展开魔术变量之后，展开占位符和其他变量之前进行处理的。因此，如果你使用 ``if-opt`` 来比较一个选项的值，那么只使用魔术变量来核对该值和配置文件中一致。 

  如果你在 ``if-opt`` 块内使用上下文占位符 ``%(_)`` ，那么应该没问题：任何占位符稍后将会被展开。
* 如果多次指定一个选项，那么 ``if-opt`` 只能看到第一个的值。
* 只会看到显式设置的值，而不会看到隐式默认值。

.. seealso:: :doc:`ParsingOrder`
