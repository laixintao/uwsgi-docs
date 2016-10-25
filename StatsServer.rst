uWSGI Stats服务器
======================

除了 :doc:`SNMP<SNMP>` ，uWSGI还支持一个Stats服务器机制，它将uWSGI状态作为一个JSON对象导出到一个socket。

只需使用 ``stats`` 选项，后面跟着一个有效的socket地址。如果你想通过HTTP提供stats，那么还需要添加 ``stats-http`` 选项。

.. code-block:: sh

    --stats 127.0.0.1:1717
    --stats /tmp/statsock
    --stats :5050
    --stats @foobar
    # Any of the above socket types can also return stats using HTTP
    --stats 127.0.0.1:1717 --stats-http

如果客户端连接到一个指定的socket，那么在连接结束之前，它将获得一个包含uWSGI内部统计数据的JSON对象。

.. code-block:: sh

    uwsgi --socket :3031 --stats :1717 --module welcome --master --processes 8

然后

.. code-block:: sh

    nc 127.0.0.1 1717
    # or for convenience...
    uwsgi --connect-and-read 127.0.0.1:1717

将返回像这样的东东：

.. code-block:: js

    {
      "workers": [{
        "id": 1,
        "pid": 31759,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 2,
        "pid": 31760,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 3,
        "pid": 31761,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 4,
        "pid": 31762,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 5,
        "pid": 31763,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 6,
        "pid": 31764,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 7,
        "pid": 31765,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }, {
        "id": 8,
        "pid": 31766,
        "requests": 0,
        "exceptions": 0,
        "status": "idle",
        "rss": 0,
        "vsz": 0,
        "running_time": 0,
        "last_spawn": 1317235041,
        "respawn_count": 1,
        "tx": 0,
        "avg_rt": 0,
        "apps": [{
          "id": 0,
          "modifier1": 0,
          "mountpoint": "",
          "requests": 0,
          "exceptions": 0,
          "chdir": ""
        }]
      }]
    }
        

uwsgitop
--------

``uwsgitop`` 是一个类似于top的命令，它使用stats服务器。可以在PyPI找到，因此使用 ``easy_install`` 或者 ``pip`` 来安装它 (自然而然地，包名是 ``uwsgitop``)。

可以在Github找到其源码。https://github.com/unbit/uwsgitop

