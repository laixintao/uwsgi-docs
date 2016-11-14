uWSGI 1.9.5
===========

更新日志20130404

错误修复
********

* 修复cachestore路由指令的内存泄漏 (Riccardo Magliocchetti)
* 修复carbon插件中的内存泄漏 (Riccardo Magliocchetti)
* 修复cgi插件中的内存泄漏 (Riccardo Magliocchetti)
* 修复旧式python动态应用
* 强制emperor对vassal遵循--max-fd
* 改进的使用post缓存的PSGI查找
* 修复kvlist转义


新特性
********

GridFS插件
^^^^^^^^^^^^^^^^^

一个导出GridFS特性的插件已推出，查看官方文档: :doc:`GridFS`

V8改进
^^^^^^^^^^^^^^^

V8插件继续改进。JSGI 3.0初步支持，以及多线程已经可用。

已实现'require' commonjs标准。

编写commonjs规格说明将是一个非常漫长的工作，因此，也许与像teajs (老的v8cgi)这样的项目合作将是一个更好的选择。

与此同时，我们正在输出文档： :doc:`V8`

'cgi'路由指令
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你可以直接从 :doc:`InternalRouting` 调用CGI脚本

.. code-block:: ini

   [uwsgi]
   plugin = cgi
   route = ^/cgi-bin/(.+) cgi:/usr/lib/cgi-bin/$1


可用性
************

uWSGI 1.9.5自20130404起可用，你可以在下面的url中找到它

http://projects.unbit.it/downloads/uwsgi-1.9.5.tar.gz
