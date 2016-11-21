uWSGI 2.0.4
===========

更新日志 [20140422]

错误修复
--------

- 修复"mime"路由变量 (Steve Stagg)
- http解析器允许重复的http头
- 更快的即需Emperor管理
- 修复UWSGI_ADDITIONAL_SOURCES构建选项
- 当启用SPDY时，合并重复的头 (Łukasz Mierzwa)
- 为未命名记录器修复段错误
- lazy-apps模式下可用--need-app
- 修复致命钩子管理


新特性
------------

实验性asyncio循环引擎 (CPython >= 3.4)
*****************************************************

asyncio (也被称为'tulip')是使用Python 3编写基于非阻塞/异步/回调代码的新的基础结构。

这个 (实验性的) 插件允许你将asyncio当成uWSGI循环引擎使用。

文档：https://uwsgi-docs.readthedocs.io/en/latest/asyncio.html

httprouter高级超时时间管理
**************************************

HTTP路由器有2个新的特定超时时间：

* `--http-headers-timeout <n>`: 定义等待http头的超时时间
* `--http-connect-timeout <n>`: 定义连接到后端实例时的超时时间

这应该能帮助系统管理员提高安全性和可用性。

关于作者：Łukasz Mierzwa

允许在--cache2禁用缓存告警
******************************************

作者：Łukasz Mierzwa

已添加'ignore_full'键值到cache2。这将会在缓存满时禁用告警。

清理LRU缓存特性，来自Yu Zhao (getcwd)
*******************************************

这个新模式允许你配置一个缓存，来自动终止最近最少使用 (LRU) 的元素，从而在空间耗尽的时候腾出空间。

仅需添加 `purge_lru=1` 到你的cache2指令中。

支持FreeBSD上的嵌入配置
**********************************

现在，你也可以在FreeBSD系统上，把配置文件嵌入到二进制文件中：

https://uwsgi-docs.readthedocs.io/en/latest/Embed.html#step-2-embedding-the-config-file

RPC钩子
********

已添加两个新的钩子：

* 'rpc' -> 调用指定的RPC函数 (错误时失败)
* 'rpcretry' -> 调用指定的RPC函数 (错误时重试)

`setmodifier1` 和 `setmodifier2` 路由动作
*************************************************

只是为了设置modifier而必须加载'uwsgi'路由插件真的真的很烦。

这两个新的选项让你可以动态设置请求modifier。

用于静态路由器的 `no_headers` 选项
*************************************

基于键值的静态路由动作现在可以避免重写响应头了 (对X-Sendfile有用)，仅需添加no_headers=1到你的键值选项上。

可用性
------------

uWSGI 2.0.4已于20140422发布，你可以从这里下载它：

http://projects.unbit.it/downloads/uwsgi-2.0.4.tar.gz


