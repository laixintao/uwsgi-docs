Python支持
==============

.. toctree::
  :maxdepth: 1

  PythonModule
  PythonDecorators
  PythonPump
  Tracebacker
  PythonModuleAlias

.. 也可以看看:: :ref:`Python配置选项 <OptionsPython>`

.. _PythonAppDict:

应用字典
----------------------

你可以使用应用字典机制来避免在配置中设置你的应用。

.. code-block:: python

  import uwsgi
  import django.core.handlers.wsgi

  application = django.core.handlers.wsgi.WSGIHandler()

  def myapp(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    yield 'Hello World\n'

  uwsgi.applications = {
    '': application,
    '/django': 'application',
    '/myapp': myapp
  }


将这个Python模块名 (即，它应该是可导入的，并且没有 ``.py`` 扩展名)传递给uWSGI的 ``module`` / ``wsgi`` 选项，uWSGI将会为URL前缀/可回调映射搜索 ``uwsgi.applications`` 字典。
  
每一个项的值可以使一个可回调对象，或者字符串类型的名字。

.. TODO: Where is the string looked up from?

.. _Virtualenv:

Virtualenv支持
------------------

`virtualenv <virtualenvwww_>`_ 是一种机制，它允许你彼此隔离一个 (或多个) Python应用的库 (当不使用uWSGI时，还有解释器)。任何可敬的现代Python应用都应该使用virtualenv。

.. _virtualenvwww: http://www.virtualenv.org/

快速入门
^^^^^^^^^^

1. 创建你的virtualenv::

    $ virtualenv myenv
    New python executable in myenv/bin/python
    Installing setuptools...............done.
    Installing pip.........done.

2. 安装所有所需的模块 (以 `Flask <http://flask.pocoo.org/>`_ 为例)::

    $ ./myenv/bin/pip install flask
    $ # Many modern Python projects ship with a `requirements.txt` file that you can use with pip like this:
    $ ./myenv/bin/pip install -r requirements.txt

3. 将你的WSGI模块拷贝到这个新环境中 (如果你不想要修改你的 ``PYTHONPATH``，那就是在 :file:`lib/python2.{x}` 之下)。
  
  .. note:: 对于许多部署而言，应用运行在virtualenv之外是常见的。如何配置它尚未有文档说明，但是它可能非常容易。
  .. TODO: Document that.

  使用 ``home``/``virtualenv`` 选项 (简称 ``-H``)来运行uwsgi服务器::

    $ uwsgi -H myenv -s 127.0.0.1:3031 -M -w envapp


.. _Python3:

Python 3
--------

WSGI规范随着 PEP3333_ 为Python 3进行了更新。

一个主要的改变时应用必须响应 ``bytes`` 实例，而非 (Unicode) 字符串到WSGI栈。

.. _pep3333: http://www.python.org/dev/peps/pep-3333/

你应该对字符串进行编码，或者使用bytes literal:

.. code-block:: python

  def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    yield 'Hello '.encode('utf-8')
    yield b'World\n'

.. _PythonPaste:

Paste支持
-------------

If you are a user or developer of 如果你是Paste兼容的框架（例如 Pyramid_, Pylons_ 和 Turbogears_  或者使用它们的应用）的用户或开发者，那么你可以使用uWSGI ``--paste`` 选项来方便地部署应用。

.. _pyramid: http://docs.pylonsproject.org/projects/pyramid/en/latest/
.. _turbogears: http://turbogears.org
.. _pylons: http://www.pylonsproject.org 

例如，如果你有一个位于 :file:`/opt/tg2env` 的虚拟环境，包含一个名为 ``addressbook`` 的Turbogears应用，并把它配置在了 :file:`/opt/tg2env/addressbook/development.ini`::

  uwsgi --paste config:/opt/tg2env/addressbook/development.ini --socket :3031 -H /opt/tg2env

就这样！无需编写额外的配置或者Python模块。

.. warning::

  如果你设置多个进程/worker (:term:`master` mode) ，那么你将收到一个错误::

    AssertionError: The EvalException middleware is not usable in a multi-process environment

  在这种情况下，你将必须把你的paste配置文件中的 ``debug`` 选项设置为False —— 或者恢复到单进程环境。


.. _PythonPecan:

Pecan支持
-------------

如果你是 Pecan_ WSGI框架的用户或开发者，那么你可以使用uWSGI的 ``--pecan`` 选项来方便地部署应用。

.. _pecan: http://pecanpy.org

例如，如果你有一个位于 :file:`/opt/pecanenv` 的虚拟环境，包含一个名为 ``addressbook`` 的Pecan应用，并把它配置在了 :file:`/opt/pecanenv/addressbook/development.py`::

  uwsgi --pecan /opt/pecanenv/addressbook/development.py --socket :3031 -H /opt/pecanenv

.. warning::

  如果你设置多个进程/worker (:term:`master` 模式)，那么你将收到一个错误::

    AssertionError: The DebugMiddleware middleware is not usable in a multi-process environment

  在这种情况下，你将必须把你的Pecan配置文件中的 ``debug`` 选项设置为False —— 或者恢复到单进程环境。

使用Django应用django-uwsgi
--------------------------------

首先，你需要从https://github.com/unbit/django-uwsgi获取``django_uwsgi`` app (一旦它在发行的``django``目录中)。

通过``pip install django-uwsgi``安装，并且将其添加到你的``INSTALLED_APPS``中。

.. code-block:: py

    INSTALLED_APPS = (
        # ...
        'django.contrib.admin',
        'django_uwsgi',
        # ...
    )
    
然后相应地修改``urls.py``。例如：

.. code-block:: py

    # ...
    url(r'^admin/uwsgi/', include('django_uwsgi.urls')),
    url(r'^admin/', include(admin.site.urls)),
    # ...

确保将django_uwsgid的URL模式放在admin site的模式*之前*，否则将永远匹配不上它。

然后，``/admin/uwsgi/``将提供uWSGI静态文件，并且有一个优雅重载服务器（当运行在Master之下时）的按钮。注意，只有在启用了``memory-report``选项的情况下，才会报告内存使用情况。

`阅读django-uwsgi位于rtfd.org <http://django-uwsgi.rtfd.org/>的文档`_

