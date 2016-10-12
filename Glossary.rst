词汇表
========

.. glossary::
  :sorted:

  harakiri
      uWSGI的一个特性，中止那些过长时间处理请求的worker。使用选项的 ``harakiri`` 族来配置。每一个会耗费比harakiri timeout指定的秒数长的请求都将会被丢弃，而对应的worker会被回收。

  master
      uWSGI内置的预派生+线程 多worker管理模式，通过打开 ``master`` 开关激活。对于所有实际的服务部署，*不* 使用master模式，真的不是个好主意。
