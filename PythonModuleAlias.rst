Aliasing Python modules
=======================

拥有一个Python包/模块/文件的多个版本是非常常见的。

操作PYTHONPATH，或者使用virtualenv是在无需修改代码的情况下使用不同版本的一种方式。

但是亲爱的，为什么不使用一个别名系统，以让你随心所欲的将模块名映射到文件呢？这就是为嘛我们有 ``pymodule-alias`` 选项的原因了！


案例1 - 映射一个简单的文件到一个虚拟模块
--------------------------------------------------

假设 ``swissknife.py`` 包含了许多有用的类和函数。

It's imported in gazillions of places in your app. Now, we'll want to modify it, but keep the original file intact for whichever reason, and call it ``swissknife_mk2``.

你的选择是：

1) to modify all of your code to import and use swissknife_mk2 instead of swissknife. Yeah, no, not's going to happen.
2) modify the first line of all your files to read ``import swissknife_mk2 as swissknife``. A lot better but you make software for money... and time is money, so why the fuck not use something more powerful?

So don't touch your files -- just remap!

.. code-block:: sh

    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=swissknife_mk2
    # Kapow! uWSGI one-two ninja punch right there!
    # You can put the module wherever you like, too:
    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=/mnt/floppy/KNIFEFAC/SWISSK~1.PY
    # Or hey, why not use HTTP?
    ./uwsgi -s :3031 -w myproject --pymodule-alias swissknife=http://uwsgi.it/modules/swissknife_extreme.py

You can specify multiple ``pymodule-alias`` directives.

.. code-block:: yaml

    uwsgi:
      socket: :3031
      module: myproject
      pymodule-alias: funnymodule=/opt/foo/experimentalfunnymodule.py
      pymodule-alias: uglymodule=/opt/foo/experimentaluglymodule.py


案例2 - 映射一个包到目录
------------------------------------------

You have this shiny, beautiful Django project and something occurs to you: Would it work with Django trunk? On to set up a new virtualenv... nah. Let's just use ``pymodule-alias``!

.. code-block:: py

  ./uwsgi -s :3031 -w django_uwsgi --pymodule-alias django=django-trunk/django


案例3 - 覆盖特定的子模块
-------------------------------------

你有一个Werkzeug项目，你想要覆盖它 - 出于随意什么理由 - ``werkzeug.test_app`` with one of your own devising. Easy, of course!

.. code-block:: python

    ./uwsgi -s :3031 -w werkzeug.testapp:test_app() --pymodule-alias werkzeug.testapp=mytestapp