Nginx支持
=============

自0.8.40版本起，Nginx本身就包含了对使用 :doc:`uwsgi protocol<Protocol>` 的上游服务器的的支持。


配置Nginx
-----------------

一般来说，你只需包含uwsgi_params文件 (包含在nginx发行版本中)，使用uwsgi_pass指令来设置uWSGI socket的地址。

::

    uwsgi_pass unix:///tmp/uwsgi.sock;
    include uwsgi_params;

—— 或者如果你使用的是TCP socket，

::

    uwsgi_pass 127.0.0.1:3031;
    include uwsgi_params;


然后，只需重载Nginx，你就准备好了经过Nginx的，由uWSGI驱动的应用。

 ``uwsgi_params`` 文件是啥？
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

它就是为了方便，仅此而已！为了让你阅读愉快，自uWSGI 1.3起，该文件的内容是::

  uwsgi_param QUERY_STRING $query_string;
  uwsgi_param REQUEST_METHOD $request_method;
  uwsgi_param CONTENT_TYPE $content_type;
  uwsgi_param CONTENT_LENGTH $content_length;
  uwsgi_param REQUEST_URI $request_uri;
  uwsgi_param PATH_INFO $document_uri;
  uwsgi_param DOCUMENT_ROOT $document_root;
  uwsgi_param SERVER_PROTOCOL $server_protocol;
  uwsgi_param REMOTE_ADDR $remote_addr;
  uwsgi_param REMOTE_PORT $remote_port;
  uwsgi_param SERVER_ADDR $server_addr;
  uwsgi_param SERVER_PORT $server_port;
  uwsgi_param SERVER_NAME $server_name;

.. seealso:: :doc:`Vars`

集群
----------

对于所有的上游处理程序，Nginx支持漂亮的集群集成。

添加一个 `upstream` 指令到server配置块外::

    upstream uwsgicluster {
      server unix:///tmp/uwsgi.sock;
      server 192.168.1.235:3031;
      server 10.0.0.17:3017;
    } 


然后修改你的uwsgi_pass指令::

    uwsgi_pass uwsgicluster;

这样，你的请求将会在配置的uWSGI服务器之间进行均衡。


动态应用
------------

当传递特殊变量的使用，uWSGI服务器可以按需加载应用。

可以在不传递任何应用配置的情况下启动uWSGI::

  ./uwsgi -s /tmp/uwsgi.sock


如果请求设置了 ``UWSGI_SCRIPT`` 变量，那么服务器将会加载指定的模块::

  location / {
    root html;
    uwsgi_pass uwsgicluster;
    uwsgi_param UWSGI_SCRIPT testapp;
    include uwsgi_params;
  }

你甚至还可以在每个location内配置多个应用::

  location / {
    root html;
    uwsgi_pass uwsgicluster;
    uwsgi_param UWSGI_SCRIPT testapp;
    include uwsgi_params;
  }

  location /django {
    uwsgi_pass uwsgicluster;
    include uwsgi_params;
    uwsgi_param UWSGI_SCRIPT django_wsgi;
  }  
        

在同一个进程中托管多个应用 (亦称管理SCRIPT_NAME和PATH_INFO)
----------------------------------------------------------------------------------

WSGI标准决定了 ``SCRIPT_NAME`` 是一个用来选择特定应用的变量。不幸的是，
nginx不能够根据SCRIPT_NAME重写PATH_INFO。出于这样的原因，你需要指示uWSGI在所谓的“挂载点”中映射特定的应用，并且自动重写SCRIPT_NAME和PATH_INFO：

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   ; mount apps
   mount = /app1=app1.py
   mount = /app2=app2.py
   ; rewrite SCRIPT_NAME and PATH_INFO accordingly
   manage-script-name = true
   
   
   
考虑到应用本身 (最终使用WSGI/Rack/PSGI中间件) 可以重写SCRIPT_NAME和PATH_INFO。

你也可以使用内部路由子系统来重写请求变量。特别是对于动态应用，这会是一种不错的方法。

注意：古老的uWSGI版本习惯支持所谓的"uwsgi_modifier1 30"方法。不要这样做。它实际上是一种丑陋的hack


SCRIPT_NAME是一个方便的惯例，但是允许你使用任何“映射方法”，例如，可以使用UWSGI_APPID变量在挂载点表中设置一个键。


.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   ; mount apps
   mount = the_app1=app1.py
   mount = the_app2=app2.py


.. code-block::

   location /app1 {
    root html;
    uwsgi_pass uwsgicluster;
    uwsgi_param UWSGI_APPID the_app1;
    include uwsgi_params;
   }
   
   location /app2 {
    root html;
    uwsgi_pass uwsgicluster;
    uwsgi_param UWSGI_APPID the_app2;
    include uwsgi_params;
   }
  
  
还记得吗，你可以使用nginx变量作为变量值，因此你可以使用Host头来实行某种形式的应用路由:

.. code-block::

   location / {
    root html;
    uwsgi_pass uwsgicluster;
    uwsgi_param UWSGI_APPID $http_host;
    include uwsgi_params;
   }
   
   
现在，只需在uWSGI挂载你的应用，将域名作为挂载键

.. code-block:: ini

   [uwsgi]
   socket = 127.0.0.1:3031
   ; mount apps
   mount = example.com=app1.py
   mount = foobar.it=app2.py

静态文件
------------

为了最佳性能和安全性，记得配置Nginx来提供静态文件服务，而不是让你可怜的应用自己来处理它。

uWSGI服务器可以完美提供静态文件服务，但是并不如一个专用的web服务器，例如Nginx，那么快速有效。

例如，可以像这样映射Django的 ``/media`` 路径::

  location /media {
    alias /var/lib/python-support/python2.6/django/contrib/admin/media;
  }

只有在请求的文件名不存在的时候，一些应用需要传递控制权给UWSGI服务器::

  if (!-f $request_filename) {
    uwsgi_pass uwsgicluster;
  }


.. admonition:: WARNING

  如果使用不当，那么像这样的配置可能会引发安全性问题。理智考虑，请务必三番五次检查你的应用文件、配置文件和其他敏感文件位于静态文件的根目录之外。

虚拟主机
---------------

你可以使用Nginx虚拟主机，这并无什么特殊问题。

如果你运行“不可信的”web应用 (如果你恰巧是ISP，那么就如你那些客户一样)，那么你应该限制它们的内存/地址空间使用，并且对每个主机/应用使用不同的 `uid` ::

    server {
      listen 80;
      server_name customersite1.com;
      access_log /var/log/customersite1/access_log;
      location / {
        root /var/www/customersite1;
        uwsgi_pass 127.0.0.1:3031;
    	include uwsgi_params;
      }
    }

    server {
      listen 80;
      server_name customersite2.it;
      access_log /var/log/customersite2/access_log;
      location / {
        root /var/www/customersite2;
        uwsgi_pass 127.0.0.1:3032;
        include uwsgi_params;
      }
    }

    server {
      listen 80;
      server_name sivusto3.fi;
      access_log /var/log/customersite3/access_log;
      location / {
        root /var/www/customersite3;
        uwsgi_pass 127.0.0.1:3033;
        include uwsgi_params;
      }
    }    


现在，可以在为每个socket使用不同的uid和受限（如果你想的话）地址空间来运行客户应用 (使用你选择的进程管理器，例如 `rc.local`, :doc:`Upstart`, `Supervisord` 或者任何激起你想象的工具) ::

  uwsgi --uid 1001 -w customer1app --limit-as 128 -p 3 -M -s 127.0.0.1:3031
  uwsgi --uid 1002 -w customer2app --limit-as 128 -p 3 -M -s 127.0.0.1:3032
  uwsgi --uid 1003 -w django3app --limit-as 96 -p 6 -M -s 127.0.0.1:3033

