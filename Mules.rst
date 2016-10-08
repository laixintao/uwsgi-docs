uWSGI Mule
===========

Mule是活在uWSGI栈中的worker进程，但通过socket连接是不能访问的，它可以作为一种通用子系统使用，以卸载任务。你可以将它们看成一个比较原始的 :doc:`spooler<Spooler>`。

它们可以访问整个 uWSGI API，可以管理信号，并且可以通过一个简单的基于字符串的消息系统来进行通信。

要启动一个mule (你可以启动无限个它们)，需要多少次，就使用多少次 ``mule`` 选项。

Mule有两种模式，

* 纯信号模式（默认模式）。在这种模式下，mule像正常的worker那样加载你的应用。它们只能响应 :doc:`uWSGI signals<Signals>`。
* 编程模式。在这种模式下，mule与你的应用分开加载一个程序。见 ProgrammedMules_.

默认情况下，每个mule以纯信号模式启动。

.. code-block:: sh

    uwsgi --socket :3031 --mule --mule --mule --mule

.. code-block:: xml

    <uwsgi>
        <socket>:3031</socket>
        <mule/>
        <mule/>
        <mule/>
        <mule/>
    </uwsgi>

基本使用
-----------

.. code-block:: py

    import uwsgi
    from uwsgidecorators import timer, signal, filemon
    
    # run a timer in the first available mule
    @timer(30, target='mule')
    def hello(signum):
        print "Hi! I am responding to signal %d, running on mule %d" % (signum, uwsgi.mule_id())
    
    # map signal 17 to mule 2
    @signal(17, target='mule2')
    def i_am_mule2(signum):
        print "Greetings! I am running in mule number two."
    
    # monitor /tmp and arouse all of the mules on modifications
    @filemon('/tmp', target='mules')
    def tmp_modified(signum):
        print "/tmp has been modified. I am mule %d!" % uwsgi.mule_id()
    

.. _ProgrammedMules:

赋予mule智慧
-----------------------

如前所述，可以对mule进行编程。要赋予一个mule自定义逻辑，则将脚本名传递给 ``mule`` 选项。

.. code-block:: sh

    uwsgi --socket :3031 --mule=somaro.py --mule --mule --mule

这将会运行4个mule，3个处于纯信号模式，一个运行 :file:`somaro.py`。

.. code-block:: py

    # somaro.py
    from threading import Thread
    import time
    
    def loop1():
        while True:
            print "loop1: Waiting for messages... yawn."
            message = uwsgi.mule_get_msg()
            print message
    
    def loop2():
        print "Hi! I am loop2."
        while True:
            time.sleep(2)
            print "This is a thread!"
    
    t = Thread(target=loop2)
    t.daemon = True
    t.start()
    
    if __name__ == '__main__':
        loop1()

因此，正如你可以从这个例子看到的那样，你可以在一个编程mule中使用 :py:meth:`mule_get_msg` 来接收消息。相同编程mule中的多个线程会等待消息。

如果你想阻塞一个mule，以等待一个uWSGI信号，而不是消息，那么你可以使用 :py:meth:`uwsgi.signal_wait`。

使用 :py:meth:`uwsgi.mule_msg` 来发送一个消息给编程mule。可以从uWSGI栈中的任何一个地方发送mule消息，包括但不限制于worker, spooler, 另一个mule。

.. code-block:: py

    # Send the string "ciuchino" to mule1.
    # If you do not specify a mule ID, the message will be processed by the first available programmed mule.
    uwsgi.mule_msg("ciuchino", 1)

由于你可以生成无限个mule，因此你或许需要某些形式的同步 —— 例如，如果你正在开发一个任务管理子系统，并且不希望两个mule能够同时启动相同的任务。你很幸运 —— 见 :doc:`Locks`。