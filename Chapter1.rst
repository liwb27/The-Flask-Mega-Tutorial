.. _Chapter1:

第一章: Hello, World!
=====================

欢迎！你正在开始学习如何使用Python和Flask框架搭建网络应用。在这一章里，你将会学习如何生成一个Flask工程。在这一章结束后你会搭建一个运行在你自己计算机上的简单的Flask网络应用！

这是本章的GitHub链接:`Browse <https://github.com/miguelgrinberg/microblog/tree/v0.1>`_, `Zip <https://github.com/miguelgrinberg/microblog/archive/v0.1.zip>`_, `Diff <https://github.com/miguelgrinberg/microblog/compare/v0.0...v0.1>`_。

安装Python
----------

你可以从Python官方网站上下载安装包。如果你时Windows系统并且使用WSL或者Cygwin，注意你将不能使用原生Windows版本的Python，而是从Ubuntu（如果你在使用WSL）或者Cygwin获取一个Unix的版本。

为了确保你的Python工作正常，你可以打开一个控制台终端然后输入python3，如果不行则输入python。你将会看到下面的结果::

    $ python3
    Python 3.5.2 (default, Nov 17 2016, 17:05:23)
    [GCC 5.4.0 20160609] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> _

Python解释器现在正在等待一个交互式的输入，这里你输入python语句。你可以输入exit()然后Enter退出。在Linux和Mac OS X版本的Python中，你可以输入Ctrl-D推出。在Windows中快捷键时Ctrl-Z加Enter。

配置虚拟环境
------------

安装Flask前需要配置python的虚拟环境，为了给不同的应用准备不同版本的包，Python使用了虚拟环境的概念。虚拟环境是一个Python解释器的完全拷贝。当你将包安装到虚拟环境中后，只有虚拟环境中的拷贝收到影响。因为每个应用程序都使用了不同的虚拟环境，因此可以完全自由的安装各种版本的包。虚拟环境的另一个好处是创建者拥有完全所有权，所以不需要管理员权限。

现在我们来创建一个虚拟环境。目录名叫microblog，同时也是应用程序的名字::

    $ mkdir microblog
    $ cd microblog

如果你使用的是Python3，默认支持虚拟环境，所以你可以这样来创建::

    $ python3 -m venv venv

通过这个命令，python执行venv包，这个包创建了名叫venv的虚拟环境。命令中的第一个venv是Python中的包名，第二个是虚拟环境名。你可以将第二个venv替换为任何其他名字。通常我都把虚拟环境命名为venv，这样每当打开一个工程立刻就能知道他使用的虚拟环境。

注意，有些操作系统你需要python替换为python3。有些安装使用python表示Python 2.x，python3表示Python 3.x。

当命令执行完，你会得到一个名叫venv的目录，里面存储着虚拟环境文件。

当你使用Python 3.4以前的版本（包括Python 2.7），默认不支持虚拟环境。这些版本的Python需要下载名叫virtualenv的第三方工具。当virtualenv安装好后，你可以用以下命令来创建虚拟环境::

    $ virtualenv venv

不论你用哪种方法创建了虚拟环境，现在你要告诉系统你要使用它。执行下列命令来启用虚拟环境::

    $ source venv/bin/activate
    (venv) $ _

如果你使用Microsoft Windows的命令提示符窗口（Windows command prompt window），启用方法略有不同::

    $ venv\Scripts\activate
    (venv) $ _ 
    
**笔记：** Scripts文件夹下提供了3个activate，activate.bat是cmd下用的，activate.ps1是powershell下用的。powershell下默认不启用脚本执行权限，可以使用如下命令启用::

    Set-ExecutionPolicy RemoteSigned

当你启动了一个虚拟环境，终端会运行虚拟环境中的python。而且终端提示符也会被修改为虚拟环境。这一对终端的更改局限在这一终端进程中，当你关闭终端后虚拟环境也会关闭。如果你同时开启了多个终端，最好的办法是每个终端启用单独的虚拟环境。

安装Flask
---------

下一步是安装Flask，Python提供了一个安装工具叫做pip（在python2.7中没有自带这个工具，需要单独安装）。

你可以使用pip来安装一个包到你的机器上，像这样::

    $ pip install <package-name>

如果你的Python解释器是对计算机上所有用户全局安装的，那么可能你的常用用户并没有修改它的权限，所以唯一的办法是用管理员账户运行pip。因此在虚拟环境中安装就会省去这个麻烦。

在虚拟环境终端中安装Flask::

    (venv) $ pip install flask

当你想确认虚拟环境中Flask以经装好，你可以在Python解释器中import Flask来确认::

    >>> import flask
    >>> _

如果没有提示错误，那么Flask就以经安装成功。

一个"Hello, World" Flask 程序
-----------------------------

如果你打开Flask官方网站，你会看到一个仅由5行代码组成的简单例子。相比于重复那个简单的例子，我会给你准备一个更加丰富的例子，而且能更好的适应日后的大型程序。

程序会存在一个包（package）中。在Python中，一个包含__init__.py的文件被认为是一个包，而且可以被导入。当你导入一个包时__init__.py文件会被执行，并由它来决定哪些变量会被外界使用。

我们创建一个叫app的包，用来承载程序。在microblog目录下运行下列命令::

    (venv) $ mkdir app

app包中的 __init__.py 包含下列代码::

    from flask import Flask

    app = Flask(__name__)

    from app import routes

上述代码从flask包中导入了Flask类，并创建了一个该类的实例。传递给Flask类的__name__变量是一个Python预定义的变量，它的值是当前使用的模块的名称。当Flask需要使用相关资源文件，例如模板文件时，使用当前模块的位置作为初始路径，这一点会在 :ref:`Chapter2` 中涉及。事实上，传递__name__变量是配置几乎所有Flask项目的正确的方法。程序还会导入routes模块，尽管它现在还不存在。

有两个实体叫做app，在一开始可能引起困惑。app包是通过app目录和__init__.py文件定义的，在 ``from app import routes`` 这句代码中被引用。app变量是在__init__.py中定义的一个实例，是app包的成员。

另一个值得注意的是routes模块是在代码最后被导入的，而不是通常的在代码开头导入。底部导入为了避免循环导入的一个方法，这在Flask程序中非常常见。你会发现routes模块需要导入在__init__.py中定义的app变量，所以将循环导入中的一个放置在代码最后，可以避免两个文件之间的引用冲突。

那么routes模块里都有什么？路由（routes）是由程序实现的一组不同的链接（URL）。在Flask中，每一个路由的处理程序都是一个Python函数，被称为视图函数（view function）。视图函数会被映射到一个或者多个路由URL上，这样在客户端请求某个URL时，Flask就能知道如何响应。

你需要把下列代码写在一个新模块里，并保存为app/routes.py，这将是你的第一个视图函数::

    from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        return "Hello, World!"

这个视图函数非常简单，只返回了一个问候字符串。两行 ``@app.route`` 代码是函数修饰器，一个Python语言的特性。修饰器会修改被修饰的函数。通常修饰器用来作为一些特定事件的回调函数。在这个例子中，``@app.route`` 将URL和函数进行了关联。在例子中有两个修饰器，分别把/和/index 两个URL关联到这个函数上。这意味着当浏览器请求上述任意一个URL时，Flask都会调用这个函数，并将函数结果返回给浏览器。如果现在你还是搞不清楚，那么当你运行这个程序时就能更清楚一点。

你还需要一个顶层的Python代码，来定义Flask程序的实例。把这个代码文件命名为microblog.py，它只包含了一行代码，来导入程序实例::

    from app import app

还记得那两个app？这里你就可以在同一个语句里看到他们。Flask程序实例叫做app，它同时也是app包的成员。``from app import app statement`` 语句导入了app包中的app变量。如果你觉得这有点绕，你可以把其中一个app改成其他名字。

为了确保一切正确，这里是到目前位置的项目结构::

    microblog/
    venv/
    app/
        __init__.py
        routes.py
    microblog.py

现在程序的第一个版本已经完成了，在运行之前还需要设置Flask环境变量FLASK_APP::

    (venv) $ export FLASK_APP=microblog.py

如果你使用的是windows系统，将上述代码中的 ``export`` 替换为 ``set``。

**笔记：** 
    如果是windows环境下的话，cmd和powershell的环境变量设置方法不同。
    cmd下和文中的方式相同，powershell下需要用 ``$env:FLASK_APP="microblog.py"``。

准备好了么？现在就可以用下面的命令运行你的第一个网络应用了::

    (venv) $ flask run
    * Serving Flask app "microblog"
    * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

当服务初始化完成后，就会等待客户端连接。``flask run`` 的输出结果表明服务运行在127.0.0.1的IP地址上。这个地址代表你的当前计算机，它还有一个更简短的名字：localhost。网络服务会监听特定的网络端口。通常在网络服务器上部署的程序监听443端口，或者在未加密情况下监听80端口，但这些端口都需要管理员权限。但这里程序运行在开发服务器上，Flask使用了空闲的端口5000。现在打开浏览器并输入下列URL::

 http://localhost:5000/

或者也可以输入下面的URL::

 http://localhost:5000/index

发现路由映射了么？第一个URL被映射到了 /，第二个映射到了 /index。两个路由都被关联到了同一个视图函数上，所以产生了相同的输出，这些输出来自于同一个函数的返回值。如果你输入其他URL会得到一个错误，因为只有上述两个URL可以被程序识别。

你可以输入 ``Ctrl-C`` 来停止服务。

恭喜，你现在已经完成了成为一个网络开发者的第一步！