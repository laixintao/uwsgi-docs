Emperor协议
====================

从1.3开始，你可以通过 :doc:`Emperor<Emperor>` 生成自定义应用。

非uWSGI Vassal应该永不进行守护，以维持与Emperor的链接。如果你想/需要与Emperor更好的集成，那么实现Emperor协议。

协议
------------

环境变量 ``UWSGI_EMPEROR_FD`` 会被传递给每个vassal，包含一个文件描述符号。

.. code-block:: python

  import os
  has_emperor = os.environ.get('UWSGI_EMPEROR_FD')
  if has_emperor:
     print "I'm a vassal snake!"

或者在Perl中，

.. code-block:: python

  my $has_emperor = $ENV{'UWSGI_EMPEROR_FD'}
  if ($has_emperor) {
    print "I am a vassal.\n"
  }

或者在C中，

.. code-block:: c

  int emperor_fd = -1;
  char *has_emperor = getenv("UWSGI_EMPEROR_FD");
  if (has_emperor) {
      emperor_fd = atoi(has_emperor);
      fprintf(stderr, "I am a vassal.\n");
  }

从现在开始，你可以通过这个文件描述符从Emperor接收信息（发送信息到Emperor）。

消息时字节大小 (0-255)，并且每个数字 (字节) 都有其含义。

== ==
0  由Emperor发送，用来停止一个vassal
1  由Emperor发送，用来重载一个vassal / 当vassal已经生成时，由其发送
2  由vassal发送，用来向Emperor请求配置块
5  当vassal准备好接收请求时，由其发送
17 在第一个宣告loyalty的请求后，由vassal发送
22 由vassal发送，用来通知Emperor自愿的关闭
26 vassal发送的心跳。在第一个接收到的心跳之后，Emperor将会期待更多来自该vassal的心跳。
30 由vassal发送，用来请求 :doc:`Broodlord` 模式。
== ==
