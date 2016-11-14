uWSGI 2.0.14
============

[20161003]

维护版本

改动
-------

- 回迁gevent早期猴子补丁 (jianbin-wei)
- 修复OpenBSD版本检查 (Pavel Korovin)
- PSGI/Perl缓存api修复 (Alexander Demenshin)
- 在router_rewrite插件中挣钱解码PATH_INFo (Ben Hearsum)
- 为链式重载 + worker覆盖组合添加uwsgi.accepting() (enkore)
- 修复cheapter模式下的worker杀死 (shoham-stratoscale)
- 添加--cgi-safe选项 (nnnn20430)
- 已为COROAE插件实现优雅重载 (aleksey-mashanov)
- 添加--php-fallback2, --php-fallback-qs (Felicity unixwitch)
- 添加ipv4in和ipv6in路由规则 (Felicity unixwitch)
- 修复在交互式工作时，python3中的readline支持 (Anthony Sottile)
- 为mule和spooler实现touch重载（touch-reloading） (Alexandre Bonnetain)
- 添加request_start时间戳到统计数据中 (Ben Plotnick)
- 修复uwsgi_routing_func_rewrite中的双重释放 (William Orr)
- 各种mod_proxy_uwsgi修复 (Ya-Lin Huang)
- 支持PSGI中的'no-answer' (Anton Petrusevich)
- 添加php-constant选项 (Дамјан Георгиевски [gdamjan])
- 添加标准输入输出日志记录器 (Дамјан Георгиевски [gdamjan])
- spooler: 修复读取不一致数据 (Pavel Patrin)
- 从构建过程删除-WError (Riccardo Magliocchetti, 由Ian Denhardt提议)
- 常量基于coverity的修复 (Riccardo Magliocchetti)

可用性
------------

你可以从http://projects.unbit.it/downloads/uwsgi-2.0.14.tar.gz下载uWSGI 2.0.14
