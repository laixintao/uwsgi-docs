别名化Python模块
=======================

拥有一个Python包/模块/文件的多个版本是非常常见的。

操作PYTHONPATH，或者使用virtualenv是在无需修改代码的情况下使用不同版本的一种方式。

但是亲爱的，为什么不使用一个别名系统，以让你随心所欲的将模块名映射到文件呢？这就是为嘛我们有 ``pymodule-alias`` 选项的原因了！


案例1 - 映射一个简单的文件到一个虚拟模块
--------------------------------------------------

假设 ``swissknife.py`` 包含了许多有用的类和函数。

在你的应用中成千上万的地方都对其进行了导入。现在，我们想要修改它，但是出于任何原因，我们想要保留原文件完好，称其为 ``swissknife_mk2`` 。

你的选择是：

1) 修改你所有的代码来导入和使用swissknife_mk2，而不是swissknife。是啊，不，这不会发生。
2) 修改你所有文件的第一行： ``import swissknife_mk2 as swissknife`` 。好多了，但是你写软件是为了赚钱的……而时间就是金钱，所以不特么用些更强大的东西呢？

所以，不要碰你的文件 —— 只需重新映射！

.. code-block:: sh

    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=swissknife_mk2
    # Kapow! uWSGI one-two ninja punch right there!
    # You can put the module wherever you like, too:
    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=/mnt/floppy/KNIFEFAC/SWISSK~1.PY
    # Or hey, why not use HTTP?
    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=http://uwsgi.it/modules/swissknife_extreme.py

你可以指定多个 ``pymodule-alias`` 指令。

.. code-block:: yaml

    uwsgi:
      socket: :3031
      module: myproject
      pymodule-alias: funnymodule=/opt/foo/experimentalfunnymodule.py
      pymodule-alias: uglymodule=/opt/foo/experimentaluglymodule.py


案例2 - 映射一个包到目录
------------------------------------------

你拥有这个金光闪闪的漂亮的Django项目，但是有些事情发生在你身上：它能和Django主干完美配合吗？创建一个新的虚拟环境……还是别吧。让我们只使用 ``pymodule-alias``!

.. code-block:: py

  ./uwsgi -s :3031 -w django_uwsgi --pymodule-alias django=django-trunk/django


案例3 - 覆盖特定的子模块
-------------------------------------

你有一个Werkzeug项目，你想要覆盖它 - 出于随意什么理由 - 带你其中一个自己的设计的 ``werkzeug.test_app`` 。当然，这很容易！

.. code-block:: python

    ./uwsgi -s :3031 -w werkzeug.testapp:test_app() --pymodule-alias werkzeug.testapp=mytestapp