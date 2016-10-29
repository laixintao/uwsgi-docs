支持的语言和平台
=================================

.. list-table:: 
    :header-rows: 1
    
    * - 技术
      - 初始可用版本
      - 注意事项
      - 状态
    * - Python
      - 0.9.1
      - 第一个可用插件，支持WSGI (:pep:`333`, :pep:`3333`),
        Web3 (from version 0.9.7-dev) 和Pump (自0.9.8.4起)。 与
        :doc:`Virtualenv` ，多个Python解析器 :doc:`Python3` 配合良好，并且具有独特的特性，例如 :doc:`PythonModuleAlias`,
        :doc:`DynamicVirtualenv` 和 :doc:`uGreen` 。源代码发布版本中有一个可用的uWSGI API模块导出
        :doc:`decorators<PythonDecorators>` 。自1.3，支持PyPy :doc:`is supported<PyPy>` 1.3添加了
        :doc:`Tracebacker` 。
      - 稳定, 100%的uWSGI API支持
    * - Lua
      - 0.9.5
      - 支持 :doc:`LuaWSAPI` ，协程和线程
      - 稳定, 60%的uWSGI API支持
    * - Perl
      - 0.9.5
      - :doc:`Perl` (PSGI)支持。支持多种解析器，线程和异步模式
      - 稳定, 60%的uWSGI API支持
    * - Ruby
      - 0.9.7-dev
      - :doc:`Ruby` 支持。有一个用于 :doc:`Ruby 1.9
        fibers<FiberLoop>` 的循环引擎，以及一个便捷的 :doc:`DSL <RubyDSL>` 模块可用。
      - 稳定, 80%的uWSGI API支持
    * - :doc:`Erlang`
      - 0.9.5
      - 允许uWSGI和Erlang节点之间的消息交换。
      - 稳定，没有uWSGI API支持
    * - :doc:`CGI`
      - 1.0-dev
      - Run CGI scripts
      - 稳定，没有uWSGI API支持
    * - :doc:`PHP`
      - 1.0-dev
      - 运行PHP脚本
      - 自1.1起稳定，5%的uWSGI API支持
    * - :doc:`Go`
      - 1.4-dev
      - 允许与Go语言的集成
      - 15%的uWSGI API支持
    * - :doc:`JVM`
      - 1.9-dev
      - uWSGI和Java虚拟机
        :doc:`JWSGI<JWSGI>` 之间的集成，以及
         :doc:`Clojure/Ring<Ring>` 处理程序是可用的。
      - 稳定
    * - :doc:`Mono`
      - 0.9.7-dev
      - 允许uWSGI和Mono之间的集成，以及ASP.NET应用的执行。
      - 稳定
    * - :doc:`V8`
      - 1.9.4
      - 运行uWSGI和V8 JavaScript引擎之间的集成
      - 开发初期
