Symcall插件
==================

symcall插件 (modifier 18) 是一个便利的插件，允许你在不需要开发一个完整的uWSGI插件的情况下编写原生uWSGI请求处理器。

你告诉它在启动时加载那个时候符号，然后它就会在每个请求上运行它。

.. note::

   在标准的构建配置文件中，默认内建"symcall"插件。

第一步：准备环境
*********************************

uWSGI二进制自身允许你在无需外部开发包或者头文件的情况下开发插件和库。

第一步是获取 ``uwsgi.h`` C/C++头文件：

.. code-block:: sh

   uwsgi --dot-h > uwsgi.h
   
现在，在当前目录下，我们有了一个准备被包含的新鲜的uwsgi.h。

第二步：第一个请求处理器：
**********************************

我们的C处理器将会打印REMOTE_ADDR值以及一对HTTP头。

(称其为mysym.c或者随便你)

.. code-block:: c

   #include "uwsgi.h"

   int mysym_function(struct wsgi_request *wsgi_req) {
   
        // read request variables
        if (uwsgi_parse_vars(wsgi_req)) {
                return -1;
        }
        
        // get REMOTE_ADDR
        uint16_t vlen = 0;
        char *v = uwsgi_get_var(wsgi_req, "REMOTE_ADDR", 11, &vlen);
        
        // send status
        if (uwsgi_response_prepare_headers(wsgi_req, "200 OK", 6)) return -1;
        // send content_type
        if (uwsgi_response_add_content_type(wsgi_req, "text/plain", 10)) return -1;
        // send a custom header
        if (uwsgi_response_add_header(wsgi_req, "Foo", 3, "Bar", 3)) return -1;
        
        // send the body
        if (uwsgi_response_write_body_do(wsgi_req, v, vlen)) return -1;
        
        return UWSGI_OK;
   }

第三步：将我们的代码作为共享库进行构建
*********************************************

uwsgi.h文件是一个ifdef地狱 (所以可能最好不要看得太深入)。

幸运的是，uwsgi二进制通过--cflags选项公开了所有所需的CFLAGS。

我们可以一次到位构建我们的库：

.. code-block:: c

   gcc -fPIC -shared -o mysym.so `uwsgi --cflags` mysym.c

现在，你准备好了在uWSGI中加载的mysym.so库了

最后一步：映射symcall插件到 ``mysym_function`` 符号
*******************************************************************

.. code-block:: sh

   uwsgi --dlopen ./mysym.so --symcall mysym_function --http-socket :9090 --http-socket-modifier1 18
   
通过 ``--dlopen`` ，我们在uWSGI进程地址空间中加载了一个共享库。

 ``--symcall`` 选项允许我们当有modifier1 18的时候，指定调用哪个符号

我们绑定实例到HTTP socket 9090，强制modifier1 18。


钩子和symcall释放：一个TCL处理器
******************************************

我们想要编写一个每次运行以下TCL脚本 (foo.tcl) 的请求处理器：

.. code-block:: tcl

   # call it foo.tcl
   proc request_handler { remote_addr path_info query_string } {
        set upper_pathinfo [string toupper $path_info]
        return "Hello $remote_addr $upper_pathinfo $query_string"
   }
   
   
我们将定义一个初始化TCL解析器和传递脚本的函数。将会在启动时，于移除权限之后立即调用这个函数。

最后，我们定义引用这个TCL过程并传递参数的请求处理器

.. code-block:: c


   #include <tcl.h>
   #include "uwsgi.h"

   // global interpreter
   static Tcl_Interp *tcl_interp;

   // the init function
   void ourtcl_init() {
        // create the TCL interpreter
        tcl_interp = Tcl_CreateInterp() ;
        if (!tcl_interp) {
                uwsgi_log("unable to initialize TCL interpreter\n");
                exit(1);
        }

        // initialize the interpreter
        if (Tcl_Init(tcl_interp) != TCL_OK) {
                uwsgi_log("Tcl_Init error: %s\n", Tcl_GetStringResult(tcl_interp));
                exit(1);
        }

        // parse foo.tcl
        if (Tcl_EvalFile(tcl_interp, "foo.tcl") != TCL_OK) {
                uwsgi_log("Tcl_EvalFile error: %s\n", Tcl_GetStringResult(tcl_interp));
                exit(1);
        }

        uwsgi_log("TCL engine initialized");
   }

   // the request handler
   int ourtcl_handler(struct wsgi_request *wsgi_req) {

        // get request vars
        if (uwsgi_parse_vars(wsgi_req)) return -1;

        Tcl_Obj *objv[4];
        // the proc name
        objv[0] = Tcl_NewStringObj("request_handler", -1);
        // REMOTE_ADDR
        objv[1] = Tcl_NewStringObj(wsgi_req->remote_addr, wsgi_req->remote_addr_len);
        // PATH_INFO
        objv[2] = Tcl_NewStringObj(wsgi_req->path_info, wsgi_req->path_info_len);
        // QUERY_STRING
        objv[3] = Tcl_NewStringObj(wsgi_req->query_string, wsgi_req->query_string_len);

        // call the proc
        if (Tcl_EvalObjv(tcl_interp, 4, objv, TCL_EVAL_GLOBAL) != TCL_OK) {
                // ERROR, report it to the browser
                if (uwsgi_response_prepare_headers(wsgi_req, "500 Internal Server Error", 25)) return -1;
                if (uwsgi_response_add_content_type(wsgi_req, "text/plain", 10)) return -1;
                char *body = (char *) Tcl_GetStringResult(tcl_interp);
                if (uwsgi_response_write_body_do(wsgi_req, body, strlen(body))) return -1;
                return UWSGI_OK;
        }

        // all fine
        if (uwsgi_response_prepare_headers(wsgi_req, "200 OK", 6)) return -1;
        if (uwsgi_response_add_content_type(wsgi_req, "text/plain", 10)) return -1;

        // write the result
        char *body = (char *) Tcl_GetStringResult(tcl_interp);
        if (uwsgi_response_write_body_do(wsgi_req, body, strlen(body))) return -1;
        return UWSGI_OK;
   }

   
你可以这样构建它：

.. code-block:: sh

   gcc -fPIC -shared -o ourtcl.so `./uwsgi/uwsgi --cflags` -I/usr/include/tcl ourtcl.c -ltcl
   
与前一个例子唯一的差异在于，-I和-l用于添加TCL头文件和库。

所以，让我们允许它：

.. code-block:: sh

   uwsgi --dlopen ./ourtcl.so --hook-as-user call:ourtcl_init --http-socket :9090 --symcall ourtcl_handler --http-socket-modifier1 18
   
这里，唯一新的用户是 ``--hook-as-user call:ourtcl_init`` ，它在移除特权后引用指定的函数。


.. note::

   代码非线程安全的！如果你想改进这个tcl库来支持多线程，那么最好的方法将是对每个pthread而非对全局使用一个TCL解析器。
   
注意事项
**************

自uWSGI 1.9.21起，多亏了 ``--build-plugin`` 选项，开发uWSGI插件变得相当简单。

symcall插件是用于小的代码库／片段，对于更大的需求，考虑开发一个完整的插件。

我们前面已经看到的tcl例子或许是“错误”使用的正确例子 ;)
