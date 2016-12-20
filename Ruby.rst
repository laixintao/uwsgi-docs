Ruby支持
============

.. toctree::
   :maxdepth: 1

   RubyAPI

从版本0.9.7-dev起，一个Ruby (Rack/Rails) 插件正式发布。官方用于Ruby应用的modifier数字是7，因此，记得在你的web服务器配置中设置它。

插件可以被嵌入到uWSGI核心中，或者作为一个动态加载的插件构建。

插件仍然不支持一些uWSGI标准的特性，例如：

* UDP请求管理
* :doc:`SharedArea` (support on the way)
* :doc:`Queue`

见 :doc:`RubyAPI` 页，以获取当前支持的特性列表。



为Ruby支持构建uWSGI
-------------------------------

你可以在 :file:`buildconf` 目录中找到 :file:`rack.ini` 。这个配置将会构建一个嵌入了Ruby解释器的uWSGI。要用这个配置构建uWSGI，你会需要Ruby头文件/开发包。

.. code-block:: sh

   python uwsgiconfig.py --build rack

所得uWSGI二进制文件可以运行Ruby应用。

也存在一个 :file:`rackp.ini` 构建配置；这将会构建一个Ruby作为插件支持的uWSGI；在这个例子中，记得带 ``plugins=rack`` 选项调用uWSGI。

关于内存消费的一个注意事项
-----------------------------------

默认情况下，这个插件的内存管理是非常积极的 (因为Ruby可以轻松消耗掉内存，就像它正变得过时一样)。会默认在每个请求之后调用Ruby垃圾回收器。如果你的应用在每次请求都创建大量的对象的话，这或许会损害你的性能。你可以使用 :ref:`OptionRubyGcFreq` 选项来调整回收的频率。像往常一样，这并没有一本万利的值，因此，实验一下吧。

如果你的应用在不受控制的情况下内存泄漏，那么在使用 ``max-requests`` 选项重启之前考虑限制一个worker可以管理的请求数。使用 ``limit-as`` 也有用。

关于线程和fiber的一个注意事项
-----------------------------------

添加Ruby 1.8中的线程支持在讨论之外。这个版本中的线程支持实际上对一个像uWSGI这样的服务器而言是没用的。Ruby 1.9有一个非常类似于Python的线程模式，它的支持从uWSGI 1.9.14起，就可以使用"rbthreads"插件来实现。

fiber是Ruby 1.9的一个新特性。它们是协程/绿色线程/停止恢复/合作多线程，或者任何你喜欢的对这类有趣技术的称呼的实现。见 :doc:`FiberLoop` 。

在uWSGI上运行Rack应用
----------------------------------

这个例子向你展示如何在uWSGI上运行一个Sinatra应用。


:file:`config.ru`

.. code-block:: ruby
    
    require 'rubygems'
    require 'sinatra'
    
    get '/hi' do
    "Hello World!"
    end
    
    run Sinatra::Application

然后调用uWSGI (如果你将Ruby支持作为插件构建，那么使用 ``--plugins`` )：

.. code-block:: sh

    ./uwsgi -s :3031 -M -p 4 -m --post-buffering 4096 --rack config.ru
    ./uwsgi --plugins rack -s :3031 -M -p 4 -m --post-buffering 4096 --rack config.ru

.. note:: 根据Rack规范， ``post-buffering`` 是必须的。

.. note:: 由于Sinatra有一个内建的日志记录系统，因此你或许希望用 ``disable-logging`` 选项来禁用uWSGI对请求的日志记录。


在uWSGI上运行Ruby on Rails应用
-------------------------------------------

由于编写正式的文档并不十分有趣，这里是几个uWSGI上的Rails应用的例子。

运行typo
^^^^^^^^^^^^

.. code-block:: sh
  
  sudo gem install typo
  typo install /tmp/mytypo
  ./uwsgi -s :3031 --lazy-apps --master --processes 4 --memory-report --rails /tmp/mytypo --post-buffering 4096 --env RAILS_ENV=production

--lazy-apps这里是至关重要的，因为typo (就像许多应用一样) 并非fork友好型的 (它不期望在master中加载，然后调用fork())。使用这个选项，应用会在每个worker中完全加载一次。

Nginx配置：

.. code-block:: nginx

    location / {
      root "/tmp/mytypo/public";
      include "uwsgi_params";
      uwsgi_modifier1 7;
      if (!-f $request_filename) {
        uwsgi_pass 127.0.0.1:3031;
      }
    }

运行Radiant
^^^^^^^^^^^^^^^

.. code-block:: sh

  sudo gem install radiant
  radiant /tmp/myradiant
  cd /tmp/myradiant
  # (edit config/database.yml to fit)
  rake production db:bootstrap
  ./uwsgi -s :3031 -M -p 2 -m --rails /tmp/myradiant --post-buffering 4096 --env RAILS_ENV=production

Apache配置 (直接映射静态路径)：

.. code-block:: apache

  DocumentRoot /tmp/myradiant/public

  <Directory /tmp/myradiant/public>
    Allow from all
  </Directory>

  <Location />
    uWSGISocket 127.0.0.1:3032
    SetHandler uwsgi-handler
    uWSGIForceScriptName /
    uWSGImodifier1 7
  </Location>

  <Location /images>
    SetHandler default-handler
  </Location>
  
  <Location /stylesheets>
    SetHandler default-handler
  </Location>
  
  <Location /javascripts>
    SetHandler default-handler
  </Location>

Rails和SSL
^^^^^^^^^^^^^

你或许希望使用 ``HTTPS`` / ``UWSGI_SCHEME https`` uwsgi协议参数来告知应用它运行在HTTPS之下。

对于Nginx：

.. code-block:: nginx
    
    uwsgi_param HTTPS on; # Rails 2.x apps
    uwsgi_param UWSGI_SCHEME https; # Rails 3.x apps
