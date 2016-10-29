使用Nagios监控uWSGI
============================

官方的uWSGI发行版包含了一个插件，用来添加 Nagios_\ -友好型输出。

要通过Nagios监控，并最终获得告警消息，请使用以下命令，其中， ``node`` 是要监控的socket (UNIX或者TCP)。

.. code-block:: sh

  uwsgi --socket <node> --nagios

设置告警消息
------------------------

你可以使用 :func:`uwsgi.set_warning_message` 函数，直接从你的应用设置一个告警消息。所有的ping响应 (Nagios也使用) 将会报告这个消息。

.. _Nagios: http://www.nagios.com/