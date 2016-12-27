uWSGI 2.0.13
============

[20160510]

改动
-------

- 修复使用GCC 6编译的问题
- 远程rpc修复 (Darvame)
- Musl支持！ (Natanael Copa, Matt Dainty, Riccardo Magliocchetti)
- 在spooler目录不存在的时候创建它 (Alexandre Bonnetain)
- 修复大端linux上的编译 (Riccardo Magliocchetti)
- 大量的缓存修复 (Darvame)
- 使得在一个不同的目录中编译插件更简单(Jakub Jirutka)
- 添加wheel包机制 (Matt Robenolt)
- 将EPOLLEXCLUSIVE用于读取，有助于惊群问题 (在linux 4.5+上) (INADA Naoki)
- 修复与unix socket的apache 2.4集成 (Alexandre Rossi)
- 添加HTTP/2支持到apache 2 proxy (Michael Fladischer, OGAWA Hirofumi)
- 修复apache 2.4.20的apache mod proxy编译 (Mathieu Arnold)
- 默认使用clang作为MacOS X上的默认编译器 (Riccardo Magliocchetti)
- 添加--cgi-close-stdin-on-eof (Roberto De Ioris)


可用性
------------

你可以从http://projects.unbit.it/downloads/uwsgi-2.0.13.tar.gz下载uWSGI 2.0.13
