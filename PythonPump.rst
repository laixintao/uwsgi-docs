Pump支持
============

.. 注意:: Pump并不是PEP，也不是标准。

Pump_ 是一个新项目，旨在“更好的”WSGI。

为你方便起见，一个样例Pump应用：

.. code-block:: python

    def app(req):
        return {
            "status": 200,
            "headers": {"content_type": "text/html"},
            "body": "<h1>Hello!</h1>"
        }

要加载一个Pump应用，只需使用 ``pump`` 选项来声明回调。

.. code-block:: sh

    uwsgi --http-socket :8080 -M -p 4 --pump myapp:app

``myapp`` 是模块的名字（必须是可导入的！），而app是回调。回调部分是可选的，默认情况下，uWSGI将会搜索一个名为'application'的回调。

.. _Pump: http://adeel.github.com/pump/