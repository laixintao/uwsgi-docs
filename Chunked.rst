块输入API
=====================

已在uWSGI 1.9.13中添加了一个用于管理HTTP块输入请求的API。

这个API是非常低层次的，允许与标准的应用进行集成。

只公开了两个函数：

* ``chunked_read([timeout])``

* ``chunked_read_nb()``

(从uWSGI 1.9.20起) 在CPython, PyPy和Perl上都支持这个API

读取块
**************

要读取一个块（阻塞），只需运行

.. code-block:: perl

   my $msg = uwsgi::chunked_read
   
如果未指定超时时间，则会使用默认值。如果你没有及时获得一个块，那么这个函数将会嚷嚷 (或者在Python下，引发一个异常)。

在非阻塞/异步引擎下，你或许想要使用

.. code-block:: perl

   my $msg = uwsgi::chunked_read_nb
   
如果没有可用的块（并且错误时抛出了异常），那么该函数将会立即返回 ``undef`` (或者在Python上是 ``None``)。

一个完整的PSGI流回显例子：

.. code-block:: perl

   # simple PSGI echo app reading chunked input
   sub streamer {
        $responder = shift;
        # generate the headers and start streaming the response
        my $writer = $responder->( [200, ['Content-Type' => 'text/plain']]);

        while(1) {
                my $msg = uwsgi::chunked_read;
                last unless $msg;
                $writer->write($msg);
        }

        $writer->close;
   }

   my $app = sub {
        return \&streamer;
   };


调整块缓冲
************************

在开始读取块之前，uWSGI会分配一个固定的缓冲来存储块。

所有的消息总是会存储在相同的缓冲中。如果接收到了比缓冲大的消息，那么会引发一个异常。

默认情况下，缓冲限制为1MB。你可以用 ``--chunked-input-limit`` 选项来调整它 (单位是字节)。


与代理集成
************************

如果你计划把uWSGI放在代理/路由器之后，那么确保它支持块输入请求 (或者一般原始HTTP请求)。

当使用uWSGI HTTP路由器时，只需添加--http-raw-body来支持块输入。

HAProxy属于开箱即用。

Nginx >= 1.4支持块输入。

选项
*******

* ``--chunked-input-limit``: 一个块消息的限制 (以字节为单位，默认是1MB)
* ``--chunked-input-timeout``: 阻塞chunked_read的默认超时时间 (以秒为单位，默认与--socket-timeout值相同，4秒)

小抄
*****

* 在消耗了请求体哪怕是一字节之后调用块API函数是错误的 (这包含 ``--post-buffering``)。
* 块API函数可以被"Transfer-Encoding: chunked" 头的存在独立调用。
