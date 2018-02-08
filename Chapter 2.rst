.. _Chapter2:

第二章: 模板(Templates)
==========================

第一章完成后，你的工程的目录结构如下所示::

    microblog\
    venv\
    app\
        __init__.py
        routes.py
    microblog.py

通过配置 ``FLASK_APP=microblog.py`` 然后执行 ``flask run`` 来启动网络服务程序。在http://localhost:5000/等待客户端连接。

本章会学习如何使用模板来生成更复杂结构和动态组件的网页。

这是本章的GitHub链接:`Browse <https://github.com/miguelgrinberg/microblog/tree/v0.2>`_, `Zip <https://github.com/miguelgrinberg/microblog/archive/v0.2.zip>`_, `Diff <https://github.com/miguelgrinberg/microblog/compare/v0.1...v0.2>`_。


什么是模板
------------

如果我想在页面上增加一个欢迎用户的标题，首先我需要用python的字典来存储一个用户名::

    user = {'username': 'Miguel'}

然后使用下列代码进行html输出::

    from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        user = {'username': 'miguel'}
        return '''
    <html>
        <head>
            <title>Home Page - Microblog</title>
        </head>
        <body>
            <h1>Hello, ''' + user['username'] + '''!</h1>
        </body>
    </html>'''

上述方法显然是一种很笨的方法，仅仅添加一个简单的功能就会使view函数变得无比复杂。如果可以使程序逻辑从网页布局中脱离，那么整个代码就会变得更加可读。

模板可以使得业务逻辑和页面布局分离。在Flask中，模板作为单独文件存储在\Templates目录下，所以当你想要使用模板的时候，首先要建立一个templates目录::

    (venv) $ mkdir app/templates

下面是一个模板文件的内容，它存储在app/templates/index.html文件总::

    <html>
        <head>
            <title>{{ title }} - Microblog</title>
        </head>
        <body>
            <h1>Hello, {{ user.username }}!</h1>
        </body>
    </html>

它和一般的html文件类似，只是多了一些动态内容的占位符，占位符用 ``{{ ... }}`` 表示，占位符表示一些仅在运行时才被解析的变量。

在使用模板将页面布局分离出来后，视图函数就变得简单了许多::

    from flask import render_template
    from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        user = {'username': 'Miguel'}
        return render_template('index.html', title='Home', user=user)

试着运行一下新程序，检查浏览器中的html源代码，并将它和模板中的html代码做一下比较，就会发现模板是如何工作的。

把模板转换为完整的html页面的过程叫渲染(rendering)。我们需要使用Flask自带的 ``render_template()`` 函数来进行渲染。这个函数使用模板文件名和一系列参数变量作为输入，它会将占位符替换为对应变量的值后，返回渲染完成的页面。

``render_template()`` 函数调用了Jinjia2模板引擎。Jinjia2将 ``{{ ... }}`` 中的内容替换为对应的变量的值。

条件语句
----------

除了替换占位符为变量值之外，Jinjia2还有许多更为强大的功能。例如，Jinjia2还支持条件语句，语法格式是 ``{% ... %}``，下面是一个新版本的index.html模板文件::

    <html>
        <head>
            {% if title %}
            <title>{{ title }} - Microblog</title>
            {% else %}
            <title>Welcome to Microblog!</title>
            {% endif %}
        </head>
        <body>
            <h1>Hello, {{ user.username }}!</h1>
        </body>
    </html>

现在模板文件变得更加智能，如果视图函数忘记传递title变量，那么它就会显示一个默认的欢迎语句。你可以在调用 ``render_template()`` 时去掉title变量，来看一下模板引擎时如何处理的。

循环语句
----------

已登陆的用户可能想看到他们发表的blog列表，下面我们改进程序来支持这一功能。

首先生成一些测试用的用户和他们的blog::

from flask import render_template
from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        user = {'username': 'Miguel'}
        posts = [
            {
                'author': {'username': 'John'},
                'body': 'Beautiful day in Portland!'
            },
            {
                'author': {'username': 'Susan'},
                'body': 'The Avengers movie was so cool!'
            }
        ]
        return render_template('index.html', title='Home', user=user, posts=posts)

``posts`` 字典包含两部分内容，一是author表示作者，二是body表示blog的内容。由于posts的数量不确定，我们需要使用循环来显示所有的post。

Jinjia2提供了for语句来支持循环::

    <html>
        <head>
            {% if title %}
            <title>{{ title }} - Microblog</title>
            {% else %}
            <title>Welcome to Microblog</title>
            {% endif %}
        </head>
        <body>
            <h1>Hi, {{ user.username }}!</h1>
            {% for post in posts %}
            <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
            {% endfor %}
        </body>
    </html>

很简单吧？现在可以看一下最终渲染出的页面效果。

模板继承
----------

通常网站的顶部都会有一个带有常用连接的导航条，我们可以很简单的增加一个导航条到index.html中，但是当网站页面变多时，我就需要给每一个页面单独手动添加并进行维护，这就变成了一个极为繁琐的工作了。

Jinjia2提供了模板继承特性来解决这一难题。你所要做的仅仅是将所有模板中公用的部分提取出来供它们继承。

首先定义一个基模板，其中包含了一个导航条和标题栏，你需要把文件保存到app/templates/base.html中::

    <html>
        <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog</title>
        {% endif %}
        </head>
        <body>
            <div>Microblog: <a href="/index">Home</a></div>
            <hr>
            {% block content %}{% endblock %}
        </body>
    </html>

在这个模板中我们使用了 ``block`` 语句来定义子模板在继承时将自身放置的位置。每一个 ``block`` 都有一个唯一的名字，这样子模板在继承时就知道该将内容放置到哪个block中。

基模板做好后，我们通过继承base.html来简化index.html::

    {% extends "base.html" %}

    {% block content %}
        <h1>Hi, {{ user.username }}!</h1>
        {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
        {% endfor %}
    {% endblock %}

base.html中处理了页面的总体布局，所以在index.html中就可以将这部分去除。``extends`` 语句建立了两个模板之间的继承关系。两个模板中都有名叫content的block，这样Jinjia2就知道如何将两个模板合并在一起。现在如果我还需要添加更多的页面，我就可以从base.html中继承出一些子模板。