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

You can use the application dictionary mechanism to avoid setting up your application in your configuration.

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


Passing this Python module name (that is, it should be importable and without the ``.py`` extension) to uWSGI's ``module`` / ``wsgi`` option, uWSGI will search the ``uwsgi.applications`` dictionary for the URL prefix/callable mappings.
  
The value of every item can be a callable, or its name as a string.

.. TODO: Where is the string looked up from?

.. _Virtualenv:

Virtualenv支持
------------------

`virtualenv <virtualenvwww_>`_ is a mechanism that lets you isolate one (or more) Python applications' libraries (and interpreters, when not using uWSGI) from each other.
Virtualenvs should be used by any respectable modern Python application.

.. _virtualenvwww: http://www.virtualenv.org/

快速入门
^^^^^^^^^^

1. Create your virtualenv::

    $ virtualenv myenv
    New python executable in myenv/bin/python
    Installing setuptools...............done.
    Installing pip.........done.

2. Install all the modules you need (using `Flask <http://flask.pocoo.org/>`_ as an example)::

    $ ./myenv/bin/pip install flask
    $ # Many modern Python projects ship with a `requirements.txt` file that you can use with pip like this:
    $ ./myenv/bin/pip install -r requirements.txt

3. Copy your WSGI module into this new environment (under :file:`lib/python2.{x}` if you do not want to modify your ``PYTHONPATH``).
  
  .. note:: It's common for many deployments that your application will live outside the virtualenv. How to configure this is not quite documented yet, but it's probably very easy.
  .. TODO: Document that.

  Run the uwsgi server using the ``home``/``virtualenv`` option (``-H`` for short)::

    $ uwsgi -H myenv -s 127.0.0.1:3031 -M -w envapp


.. _Python3:

Python 3
--------

The WSGI specification was updated for Python 3 as PEP3333_. 

One major change is that applications are required to respond only with ``bytes`` instances, not (Unicode) strings, back to the WSGI stack.

.. _pep3333: http://www.python.org/dev/peps/pep-3333/

You should encode strings or use bytes literals:

.. code-block:: python

  def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    yield 'Hello '.encode('utf-8')
    yield b'World\n'

.. _PythonPaste:

Paste支持
-------------

If you are a user or developer of Paste-compatible frameworks, such as Pyramid_, Pylons_ and Turbogears_ or applications using them, you can use the uWSGI ``--paste`` option to conveniently deploy your application.

.. _pyramid: http://docs.pylonsproject.org/projects/pyramid/en/latest/
.. _turbogears: http://turbogears.org
.. _pylons: http://www.pylonsproject.org 

For example, if you have a virtualenv in :file:`/opt/tg2env` containing a Turbogears app called ``addressbook`` configured in :file:`/opt/tg2env/addressbook/development.ini`::

  uwsgi --paste config:/opt/tg2env/addressbook/development.ini --socket :3031 -H /opt/tg2env

That's it! No additional configuration or Python modules to write.

.. warning::

  If you setup multiple process/workers (:term:`master` mode) you will receive an error::

    AssertionError: The EvalException middleware is not usable in a multi-process environment

  in which case you'll have to set the ``debug`` option in your paste configuration file to False -- or revert to single process environment.


.. _PythonPecan:

Pecan支持
-------------

If you are a user or developer of the Pecan_ WSGI framework, you can use the uWSGI ``--pecan`` option to conveniently deploy your application.

.. _pecan: http://pecanpy.org

For example, if you have a virtualenv in :file:`/opt/pecanenv` containing a Pecan app called ``addressbook`` configured in :file:`/opt/pecanenv/addressbook/development.py`::

  uwsgi --pecan /opt/pecanenv/addressbook/development.py --socket :3031 -H /opt/pecanenv

.. warning::

  If you setup multiple process/workers (:term:`master` mode) you will receive an error::

    AssertionError: The DebugMiddleware middleware is not usable in a multi-process environment

  in which case you'll have to set the ``debug`` option in your pecan configuration file to False -- or revert to single process environment.

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

