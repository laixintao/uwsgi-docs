uWSGI 2.0.13
============

[20160510]

改动
-------

- Fix compilation with GCC 6
- 远程rpc修复 (Darvame)
- Musl支持！ (Natanael Copa, Matt Dainty, Riccardo Magliocchetti)
- 在spooler目录不存在的时候创建它 (Alexandre Bonnetain)
- Fix compilation on big endian linux (Riccardo Magliocchetti)
- 大量的缓存修复 (Darvame)
- 使得在一个不同的目录中编译插件更简单(Jakub Jirutka)
- 添加wheel包机制 (Matt Robenolt)
- Use EPOLLEXCLUSIVE for reading, helps with the thundering herd problem (on linux 4.5+) (INADA Naoki)
- Fix apache 2.4 integration with unix sockets (Alexandre Rossi)
- 添加HTTP/2支持到apache 2 proxy (Michael Fladischer, OGAWA Hirofumi)
- Fix apache mod proxy compilation with apache 2.4.20 (Mathieu Arnold)
- Default to clang as default compiler on MacOS X (Riccardo Magliocchetti)
- 添加--cgi-close-stdin-on-eof (Roberto De Ioris)


可用性
------------

You can download uWSGI 2.0.13 from http://projects.unbit.it/downloads/uwsgi-2.0.13.tar.gz
