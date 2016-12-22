玩转Perl, Eyetoy和RaspberryPi
=====================================

作者: Roberto De Ioris

时间: 2013-12-07

.. image:: https://raw.github.com/unbit/uwsgi-capture/master/rpi-examples/rpi_eyetoy.jpg

简介
*****

这篇文章是在2.0版本发布之前，旨在提高uWSGI在各种领域的性能和可用性的各种实验的产物。

要跟着这篇文章，你需要：

* 一个树莓派 (任何model)，上面安装了Linux发行版 (我使用的是标准的Raspbian)
* 一个PS3 Eyetoy网络摄像头
* 一个可用websocket的浏览器 (基本上任何正经的浏览器都可以)
* 一点点Perl知识 (真的只要一点点，少于10行Perl代码 ;)
* 耐心 (在RPI上构建uWSGI + PSGI + coroae需要13分钟)

uWSGI子系统和插件
****************************

这个项目利用以下uWSGI子系统和插件：

* :doc:`../WebSockets`
* :doc:`../SharedArea` (用来存储帧)
* :doc:`../Mules` (用来收集帧)
* :doc:`../Symcall`
* :doc:`../Perl`
* :doc:`../Async` (可选，我们使用 ``Coro::Anyevent`` ，但是你可以依靠标准进程，虽然这将需要更多内存)

我们想要完成什么呢
**************************

我们想要我们的RPI收集来自Eyetoy的帧，然后使用wesockets将其流化至各种已连接的客户端，使用一个HTML5 canvas元素来展示它们。

整个系统必须尽可能使用少的内存，尽可能少的CPU周期，并且它应该支持大量的客户端 (... 好吧，即使是10个客户端对树莓派硬件而言也是个胜利 ;)

技术背景
********************

Eyetoy以YUYV格式 (称为YUV 4:2:2) 捕获帧。这意味着，对于2个像素，我们需要4个字节。

默认情况下，分辨率被设置为640x480，因此每个帧将需要614,400字节。

一旦我们有了帧，我们就需要将其解码为RGBA，以允许HTML5 canvas展示它。

YUYV和RGBA之间的转换对RPI来说是重量的 (特别是若你需要为每个已连接客户端进行转换的时候)，因此我们会在浏览器中使用Javascript来进行转换。(我们可以使用其他的方式，看看文章末尾以获取相关信息。)

uWSGI栈是由一个从Eyetoy收集帧，并将这些帧写到uWSGI的共享区域的mule组成的。

worker不断地从那个共享区域读取帧，并将它们作为websockets消息发送出去。

让我们开始吧：uwsgi-capture插件
*************************************

uWSGI 1.9.21引入了一个构建uWSGI插件的简化（并且安全）的过程。(期待稍后会有更多的第三方插件！)

位于 https://github.com/unbit/uwsgi-capture 的项目展示了一个使用Video4Linux 2 API来收集帧的非常简单的插件。

每一帧都会被写入到由插件自身初始化的共享区域中。

第一步是获取uWSGI，然后用'coroae'配置文件来构建它：

.. code-block:: sh

   sudo apt-get install git build-essential libperl-dev libcoro-perl
   git clone https://github.com/unbit/uwsgi
   cd uwsgi
   make coroae
   
这个过程需要大约13分钟。如果一切顺利，那么你可以克隆uwsgi-capture插件，然后构建它。

.. code-block:: sh

   git clone https://github.com/unbit/uwsgi-capture
   ./uwsgi --build-plugin uwsgi-capture
   
现在，在你的uwsgi目录中就有了capture_plugin.so文件。

把你的Eyetoy插入到RPI上的USB口中，然后看看它是否工作：

.. code-block:: sh

   ./uwsgi --plugin capture --v4l-capture /dev/video0
   
( ``--v4l-capture`` 选项是由capture插件公开的)

如果一切顺利，那么你应该在uWSGI启动日志中看到以下行：

.. code-block:: sh

   /dev/video0 detected width = 640
   /dev/video0 detected height = 480
   /dev/video0 detected format = YUYV
   sharedarea 0 created at 0xb6935000 (150 pages, area at 0xb6936000)
   /dev/video0 started streaming frames to sharedarea 0
   
(这个共享区域内存指针显然将会不一样)

uWSGI进程将会在此之后立即退出，因为我们并没有告诉它要做什么。 :)

 ``uwsgi-capture`` 插件公开了2个函数：

* ``captureinit()``, 作为插件的init()钩子的映射，将会由uWSGI自动调用。如果指定了--v4l-capture选项，那么这个函数将会初始化指定设备，并且将其映射到一个uWSGI共享区域。
* ``captureloop()`` 是收集帧，并将它们写入到共享区域的函数。这个函数应该不断运行 (即使没有客户端读取帧)

我们想要一个mule来运行 ``captureloop()`` 函数。

.. code-block:: sh

   ./uwsgi --plugin capture --v4l-capture /dev/video0 --mule="captureloop()" --http-socket :9090
   
这次，我们绑定uWSGI到HTTP端口9090，并且带有一个映射到"captureloop()"函数的mule。这个mule语法是由symcall插件公开的，这个插件控制每一个由"()"结尾的mule参数 (引号是必须的，用来避免shell搞乱括号)。

如果一切顺利，那么你应该看到你的uWSGI服务器生成一个master，一个mule和一个worker。

第二步：PSGI应用
********************

是时候来写我们发送Eyetoy帧的websocket服务器了 (你可以在这里找到这个例子的源代码：https://github.com/unbit/uwsgi-capture/tree/master/rpi-examples)。

这个PSGI应用将会非常简单：

.. code-block:: pl

   use IO::File;
   use File::Basename;

   my $app = sub {
        my $env = shift;

        # websockets connection happens on /eyetoy
        if ($env->{PATH_INFO} eq '/eyetoy') {
                # complete the handshake
                uwsgi::websocket_handshake($env->{HTTP_SEC_WEBSOCKET_KEY}, $env->{HTTP_ORIGIN});
                while(1) {
                        # wait for updates in the sharedarea
                        uwsgi::sharedarea_wait(0, 50);
                        # send a binary websocket message directly from the sharedarea
                        uwsgi::websocket_send_binary_from_sharedarea(0, 0)
                }
        }
        # other requests generate the html
        else {
                return [200, ['Content-Type' => 'text/html'], new IO::File(dirname(__FILE__).'/eyetoy.html')];
        }
   }

唯一有趣的部分是：

.. code-block:: pl

   uwsgi::sharedarea_wait(0, 50);
   
这个函数挂起当前请求，直到指定的共享区域 ('zero'那个) 得到更新。由于这个函数基本上是一个频繁循环的poll，因此第二个参数指定了poll的频率，以毫秒为单位。50毫秒就能有不错的结果了(随意尝试其他值)。

.. code-block:: pl

   uwsgi::websocket_send_binary_from_sharedarea(0, 0)
   
这是一个特别的功能性函数，直接发送一个来自于共享区域（是哒，zero拷贝）的websocket二进制消息。第一个参数是共享区域id ('zero'那个)，而第二个是共享区域中开始读取的位置 (再次是0，因为我们想要一个完整的帧)。

第三步：HTML5
*************

HTML部分 (好吧，说是Javascript部分更恰当些) 是非常简单的，除了从YUYV到RGB(A)转换。

.. code-block:: html

   <html>
        <body>
                <canvas id="mystream" width="640" height="480" style="border:solid 1px red"></canvas>

                <script>


                        var canvas = document.getElementById('mystream');
                        var width = canvas.width;
                        var height = canvas.height;
                        var ctx = canvas.getContext("2d");
                        var rgba = ctx.getImageData(0, 0, width, height);

                        // fill alpha (optimization)
                        for(y = 0; y< height; y++) {
                                for(x = 0; x < width; x++) {
                                        pos = (y * width * 4) + (x * 4) ;
                                        rgba.data[pos+3] = 255;
                                }
                        }

                        // connect to the PSGI websocket server
                        var ws = new WebSocket('ws://' + window.location.host + '/eyetoy');
                        ws.binaryType = 'arraybuffer';
                        ws.onopen = function(e) {
                                console.log('ready');
                        };

                        ws.onmessage = function(e) {
                                var x, y;
                                var ycbcr = new Uint8ClampedArray(e.data);
                                // convert YUYV to RGBA
                                for(y = 0; y< height; y++) {
                                        for(x = 0; x < width; x++) {
                                                pos = (y * width * 4) + (x * 4) ;
                                                var vy, cb, cr;
                                                if (x % 2 == 0) {
                                                        ycbcr_pos = (y * width * 2) + (x * 2);
                                                        vy = ycbcr[ycbcr_pos];
                                                        cb = ycbcr[ycbcr_pos+1];
                                                        cr = ycbcr[ycbcr_pos+3];
                                                }
                                                else {
                                                        ycbcr_pos = (y * width * 2) + ((x-1) * 2);
                                                        vy = ycbcr[ycbcr_pos+2];
                                                        cb = ycbcr[ycbcr_pos+1];
                                                        cr = ycbcr[ycbcr_pos+3];
                                                }
                                                var r = (cr + ((cr * 103) >> 8)) - 179;
                                                var g = ((cb * 88) >> 8) - 44 + ((cr * 183) >> 8) - 91;
                                                var b = (cb + ((cb * 198) >> 8)) - 227;
                                                rgba.data[pos] = vy + r;
                                                rgba.data[pos+1] = vy + g;
                                                rgba.data[pos+2] = vy + b;
                                        }
                                }                
                                // draw pixels
                                ctx.putImageData(rgba, 0, 0);
                        };
                        ws.onclose = function(e) { alert('goodbye');}
                        ws.onerror = function(e) { alert('oops');}
                </script>

        </body>
   </html>
   
这里没啥特别的。绝大部分的代码是关于YUYV->RGBA转换。注意设置websocket通信为“二进制”模式 (binaryType = 'arraybuffer'就够了)，并且一定要使用Uint8ClampedArray (否则性能将会很糟糕)

准备观看
**************

.. code-block:: sh

   ./uwsgi --plugin capture --v4l-capture /dev/video0 --http-socket :9090 --psgi uwsgi-capture/rpi-examples/eyetoy.pl --mule="captureloop()"

连接你的浏览器到你的树莓派到TCP端口9090，然后开始看看。

并发性
***********

当你看你的websocket流时，你或许想要启动另一个浏览器窗口来看看你的视频的第二份拷贝。不幸的是，你生成的uWSGI只有一个worker，因此只有一个客户端才能获取到流。

你可以轻松添加多个worker：

.. code-block:: sh

   ./uwsgi --plugin capture --v4l-capture /dev/video0 --http-socket :9090 --psgi uwsgi-capture/rpi-examples/eyetoy.pl --mule="captureloop()" --processes 10

就像这个，最多支持10个人观看视频流。

但是对于像这样的绑定I/O应用，协程是更好的方式 (并且更便宜)：

.. code-block:: sh

   ./uwsgi --plugin capture --v4l-capture /dev/video0 --http-socket :9090 --psgi uwsgi-capture/rpi-examples/eyetoy.pl --mule="captureloop()" --coroae 10
   
现在，奇妙的是，我们能够只用一个进程来管理10个客户端！树莓派上的内存将会对你心存感激。

零拷贝所有的东西
************************

为什么我们要使用共享区域？

共享区域是uWSGI最高级的特性之一。如果你看看uwsgi-capture插件，那么你会看到它是如何轻松创建一个指向一个mmap()区域的共享区域的。基本上，每个worker，线程（但是在Perl中请千万不要使用线程）或者协程将会以一种并发安全的方式访问那个内存。

除此之外，多亏了websocket/共享区域合作API，你可以直接发送来自于一个共享区域的websocket包，而无需拷贝内存 (除了结果websocket包)。

这是比下面这样更快的方式：

.. code-block:: pl

   my $chunk = uwsgi::sharedarea_read(0, 0)
   uwsgi::websocket_send_binary($chunk)
   
我们需要每次迭代的时候为$chunk分配内存，拷贝共享区域内容到它里面，最后在一个websocket消息中封装它。

有了共享区域，你移除了不断分配（和释放）内存，以及将其从共享区域拷贝到Perl VM的需求。

其他方法
**********************

显然你还可以使用其他方法。

你可以破解uwsgi-capture，分配直接写入RGBA帧的第二个共享区域。

JPEG编码是相当快的，你可以尝试在RPI中编码帧，然后将其作为MJPEG帧发送 (而不是使用websockets):

.. code-block:: pl

   my $writer = $responder->( [200, ['Content-Type' => 'multipart/x-mixed-replace; boundary=uwsgi_mjpeg_frame']]);
   $writer->write("--uwsgi_mjpeg_frame\r\n");
   while(1) {
       uwsgi::sharedarea_wait(0);
       my $chunk = uwsgi::sharedarea_read(0, 0);
       $writer->write("Content-Type: image/jpeg\r\n");
       $writer->write("Content-Length: ".length($chunk)."\r\n\r\n");
       $writer->write($chunk);
       $writer->write("\r\n--uwsgi_mjpeg_frame\r\n");
   }

其他语言
***************

在写这篇文章的时候，uWSGI PSGI插件是唯一一个为websockets+sharedarea公开了额外API的插件。其他语言插件将在不久后进行更新。


更多的hack
************

捣鼓RPI板子是相当有趣的，而uWSGI则是它的一个不错的伴侣 (特别是它的低层次API函数)。

.. note::

  留给读者的一个练习：记住，你可以mmap()地址0x20200000来访问Raspberry PI GPIO控制器……准备好写一个uwsgi-gpio插件了吗？
  