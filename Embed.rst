在uWSGI中嵌入一个应用
=================================

从uWSGI 0.9.8.2开始，你可以在服务器二进制中嵌入文件。这些可以是任何文件类型，包括配置文件。你也可以嵌入目录，因此，通过拦截Python模块加载器，你也可以明确导入包。在这个例子中，我们将嵌入一个完整的Flask项目。

第1步：创建构建配置文件
----------------------------------

我们假设你已经准备好了uWSGI源代码。

在 ``buildconf`` 目录下，定义你的配置文件 —— 让我们称之为flask.ini:

.. code-block:: ini

    [uwsgi]
    inherit = base
    main_plugin = python
    bin_name = myapp
    embed_files = bootstrap.py,myapp.py

``myapp.py`` 是一个简单的flask应用。

.. code-block:: py

    from flask import Flask
    app = Flask(__name__)
    app.debug = True
    
    @app.route('/')
    def index():
        return "Hello World"

``bootstrap.py`` 在源代码发布版中。它会扩展python的import子系统，来使用内嵌在uWSGI中的文件。

现在，编译你饱含应用的服务器。文件将会作为可执行符号潜入。文件中的点和连接号等会被转换成下划线。

.. code-block:: xxx

    python uwsgiconfig.py --build flask

由于 ``bin_name`` 是 ``myapp`` ，现在你可以运行

.. code-block:: sh

    ./myapp --socket :3031 --import sym://bootstrap_py --module myapp:app

 ``sym://`` 伪协议让uWSGI能够访问二进制文件的嵌入符号和数据，在这种情况下，直接从二进制镜像中导入bootstrap.py。

第2步：嵌入配置文件
---------------------------------

我们想要让我们的二进制文件自动加载我们的Flask应用，而无需传递一个长长地命令行。

让我们创建配置 -- flaskconfig.ini:

.. code-block:: ini

    [uwsgi]
    socket = 127.0.0.1:3031
    import = sym://bootstrap_py
    module = myapp:app

然后将其作为一个配置文件添加到构建配置文件中。

.. code-block:: ini

    [uwsgi]
    inherit = default
    bin_name = myapp
    embed_files = bootstrap.py,myapp.py
    embed_config = flaskconfig.ini

然后，在你重新构建服务器后

.. code-block:: sh

    python uwsgiconfig.py --build flask

你现在可以简单加载

.. code-block:: sh

    ./myapp
    # Remember that this new binary continues to be able to take parameters and config files:
    ./myapp --master --processes 4

第3步：嵌入flask自身
------------------------------

现在，我们准备好玩转uWSGI的忍者神功了。我们想要单个二进制文件，它嵌入所有的Flask模块，包括Werkzeug和Jinja2，Flask的依赖。我们需要拥有这些包的目录，然后在构建配置文件中指定它们。

.. code-block:: ini

    [uwsgi]
    inherit = default
    bin_name = myapp
    embed_files = bootstrap.py,myapp.py,werkzeug=site-packages/werkzeug,jinja2=site-packages/jinja2,flask=site-packages/flask
    embed_config = flaskconfig.ini

.. note:: 这次，我们使用了"name=directory"形式，来强制符号使用指定的名字，以避免获得像 ``site_packages_flask___init___py`` 这样恶心的名字。

重新构建并重新运行。当运行以向你展示加载嵌入模块时，我们添加了--no-site。

.. code-block:: sh

    python uwsgiconfig.py --build flask
    ./myapp --no-site --master --processes 4

第4步：添加模板
------------------------

仍然不满意？好吧，你也不应该满意。

.. code-block:: ini

    [uwsgi]
    inherit = default
    bin_name = myapp
    embed_files = bootstrap.py,myapp.py,werkzeug=site-packages/werkzeug,jinja2=site-packages/jinja2,flask=site-packages/flask,templates
    embed_config = flaskconfig.ini

模板将会被添加到二进制文件中……但是我们会需要通过创建一个自定义的Jinja2模板加载器，指示Flask如何从二进制镜像中加载模板。

.. code-block:: py

    from flask import Flask, render_template
    from flask.templating import DispatchingJinjaLoader
    
    class SymTemplateLoader(DispatchingJinjaLoader):
    
        def symbolize(self, name):
            return name.replace('.','_').replace('/', '_').replace('-','_')
    
        def get_source(self, environment, template):
            try:
                import uwsgi
                source = uwsgi.embedded_data("templates_%s" % self.symbolize(template))
                return source, None, lambda: True
            except:
                pass
            return super(SymTemplateLoader, self).get_source(environment, template)
    
    app = Flask(__name__)
    app.debug = True
    
    app.jinja_env.loader = SymTemplateLoader(app)
    
    @app.route('/')
    def index():
        return render_template('hello.html')
    
    @app.route('/foo')
    def foo():
        return render_template('bar/foo.html')

POW! BIFF! NINJA AWESOMENESS.
