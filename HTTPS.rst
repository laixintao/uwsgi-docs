HTTPS支持 (自1.3起)
============================

使用 ``https <socket>,<certificate>,<key>`` 选项。这个选项可能会被多次指定。首先使用OpenSSL工具生成你服务器的密钥，证书签名请求，以及自注册证书:

.. note:: 你会想要一个用于生产的真正的SSL证书的。

::
  
  openssl genrsa -out foobar.key 2048
  openssl req -new -key foobar.key -out foobar.csr
  openssl x509 -req -days 365 -in foobar.csr -signkey foobar.key -out foobar.crt

然后使用刚刚生成的SSL证书和密钥启动服务器::

  uwsgi --master --https 0.0.0.0:8443,foobar.crt,foobar.key

由于端口443（这个端口通常用于HTTPS）是特权端口 (即非root进程可能不会绑定它)，因此你可以使用共享socket机制，像这样绑定后去除特权::

  uwsgi --shared-socket 0.0.0.0:443 --uid roberto --gid roberto --https =0,foobar.crt,foobar.key

uWSGI将会绑定到任何IP的443端口，然后删除那些 ``roberto`` 的特权，接着讲共享socket 0 (``=0``) 用于HTTPS。

.. note:: =0语法目前未公开。

设置SSL/TLS加密
-----------------------

``https`` 选项的第四个参数是可选的，你可以用它来指定OpenSSL加密套件。

.. code-block:: ini

   [uwsgi]
   master = true
   shared-socket = 0.0.0.0:443
   uid = www-data
   gid = www-data
   
   https = =0,foobar.crt,foobar.key,HIGH
   http-to = /tmp/uwsgi.sock


这将会为你的SSL/TLS事务（尽可能）设置所有的 **HIGHest（最高的）** 密码。

客户端证书认证
---------------------------------

``https`` 还可以有可选的第五个参数。你可以用它来指定一个CA证书来对你的客户端进行鉴权。生成你的CA密钥和证书 (这次，密钥将会是4096位，并且有密保)::

  openssl genrsa -des3 -out ca.key 4096
  openssl req -new -x509 -days 365 -key ca.key -out ca.crt

生成服务器密钥和CSR (如前)::

  openssl genrsa -out foobar.key 2048
  openssl req -new -key foobar.key -out foobar.csr

使用你的新CA注册服务器证书::

  openssl x509 -req -days 365 -in foobar.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out foobar.crt

为你的客户端创建一个密钥和一个CSR，使用CA进行对其进行注册，然后将其打包为
PKCS#12。为每个客户端重复这些步骤。

::

  openssl genrsa -des3 -out client.key 2048
  openssl req -new -key client.key -out client.csr
  openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
  openssl pkcs12 -export -in client.crt -inkey client.key -name "Client 01" -out client.p12


然后为证书客户端鉴权配置uWSGI

.. code-block:: ini

  [uwsgi]
  master = true
  shared-socket = 0.0.0.0:443
  uid = www-data
  gid = www-data
  https = =0,foobar.crt,foobar.key,HIGH,!ca.crt
  http-to = /tmp/uwsgi.sock

.. note:: 如果你不想要强制客户端证书鉴权，那么移除https选项中ca.crt之前的'!'。
