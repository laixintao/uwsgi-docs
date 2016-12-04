WebCaching框架
===========================

.. note::

    这是旧的缓存子系统到记录在这里 :doc:`Caching` 的新的uWSGI缓存API的移植。
    使用这里的选项将会创建一个名为“default”的新型缓存。


要启用web缓存，使用 ``cache`` 选项为你的项分配槽。以下的命令行将会创建一个包含至少1000个项的缓存。

.. code-block:: sh

   ./uwsgi --socket 127.0.0.1:3031 --module mysimpleapp --master --processes 4 --cache 1000

要在应用中使用这个缓存，

.. code-block:: python

   uwsgi.cache_set("foo_key", "foo_value") # set a key
   value = uwsgi.cache_get("foo_key") # get a key.


持久化存储
------------------

你可以把缓存数据存储在一个后备存储文件来实现持久化。简单添加 ``cache-store <filename>`` 选项。每个内核都会以一个不同的速率提交数据到磁盘。你可以使用 ``cache-store-sync <n>`` 设置是否/何时强制，其中， ``n`` 是每次磁盘同步之前等待的master周期数。

缓存清道夫
-------------

自uWSGI 1.2起，缓存项过期是由 :term:`master` 进程中的一个线程进行管理的，以减少死锁的风险。这个线程可以通过 ``cache-no-expire`` 选项禁用 (让缓存项过期成为空操作)。

可以用 ``cache-expire-freq <seconds>`` 设置缓存清道夫线程的频率。你可以通过使用 ``cache-report-freed-items`` 来让清道夫记录可用项数目。

直接从你的web服务器访问缓存
-------------------------------------------------

.. code-block:: nginx

   location / {
    uwsgi_pass 127.0.0.1:3031;
    uwsgi_modifier1 111;
    uwsgi_modifier2 3;
    uwsgi_param key $request_uri;
   }

就这样啦！Nginx现在会从一个远程uwsgi协议兼容的服务器获取HTTP响应。虽然老实说，它并不非常有用，因为如果缓存未命中，那么你将会看到你一个空白页。

一个会回退到真正的uwsgi请求的更好的系统是

.. code-block:: nginx

   location / {
     uwsgi_pass 192.168.173.3:3032;
     uwsgi_modifier1 111;
     uwsgi_modifier2 3;
     uwsgi_param key $request_uri;
     uwsgi_pass_request_headers off;
     error_page 502 504 = @real;
   }

   location @real {
     uwsgi_pass 192.168.173.3:3032;
     uwsgi_modifier1 0;
     uwsgi_modifier2 0;
     include uwsgi_params;
   }
   
Django缓存后端
--------------------

如果你运行着Django，那么有一个已经可以使用的名为 ``django-uwsgi-cache`` 的应用。它由Ionel Cristian Mărieș维护，位于https://github.com/ionelmc/django-uwsgi-cache，并且在pypi上。

.. _caching configuration: https://docs.djangoproject.com/en/dev/topics/cache/?from=olddocs#the-per-site-cache
