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
   
该函数将会立即返回 ``undef`` (或者在Python上是 ``None``) if no chunks are available (and croak/raise an exception on error).

A full PSGI streaming echo example:

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


Tuning the chunks buffer
************************

Before starting to read chunks, uWSGI allocates a fixed buffer for storing chunks.

All of the messages are always stored in the same buffer. If a message bigger than the buffer is received, an exception will be raised.

By default the buffer is limited to 1 MB. You can tune it with the ``--chunked-input-limit`` option (bytes).


与代理集成
************************

If you plan to put uWSGI behind a proxy/router be sure it supports chunked input requests (or generally raw HTTP requests).

When using the uWSGI HTTP router just add --http-raw-body to support chunked input.

HAProxy works out of the box.

Nginx >= 1.4 supports chunked input.

选项
*******

* ``--chunked-input-limit``: the limit (in bytes) of a chunk message (default 1MB)
* ``--chunked-input-timeout``: the default timeout (in seconds) for blocking chunked_read (default to the same --socket-timeout value, 4 seconds)

小抄
*****

* Calling chunked API functions after having consumed even a single byte of the request body is wrong (this includes ``--post-buffering``).
* Chunked API functions can be called independently by the presence of "Transfer-Encoding: chunked" header.
