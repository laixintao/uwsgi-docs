The Chunked input API
=====================

An API for managing HTTP chunked input requests has been added in uWSGI 1.9.13.

The API is very low-level to allow easy integration with standard apps.

只公开了两个函数：

* ``chunked_read([timeout])``

* ``chunked_read_nb()``

This API is supported (从uWSGI 1.9.20起) on CPython, PyPy and Perl.

Reading chunks
**************

To read a chunk (blocking) just run

.. code-block:: perl

   my $msg = uwsgi::chunked_read
   
如果未指定超时时间，则会使用默认值。If you do not get a chunk in time, the function will croak (or raise an exception when under Python).

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
