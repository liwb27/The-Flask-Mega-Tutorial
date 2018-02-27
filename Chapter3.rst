.. _Chapter3:

第三章: Web表单(Web Forms)
==========================
Web表单是网络应用程序的基本组成，通过使用Web表单，可以实现发表blog，登陆网站等功能。

这是本章的GitHub链接:`Browse <https://github.com/miguelgrinberg/microblog/tree/v0.3>`_, `Zip <https://github.com/miguelgrinberg/microblog/archive/v0.3.zip>`_, `Diff <https://github.com/miguelgrinberg/microblog/compare/v0.2...v0.3>`_。

介绍Flask-WTF
---------------
Flask-WTF插件是WTForms包的一个轻量级封装，并且可以完美的和Flask结合，我们使用Flask-WTF插件来处理Web表单。这是我们介绍的第一个Flask插件，插件在Flask生态环境中占据重要位置，它们实现了Flask所未实现的功能。

Flask插件是可以通过pip安装的Python包，你可以用下列命令将Flask-WTF安装到你的虚拟环境中::

    (venv) $ pip install flask-wtf

配置
-------
到目前位置，程序十分的简单，因此并不需要考虑配置的问题。但对于复杂的程序，Flask提供了一些灵活配置的选项。

进行配置的方法有很多中，最基本的是直接在app.config中定义变量，变量是以字典的形式存储的，例如::

    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'you-will-never-guess'
    # ... add more variables here as needed

为了贯彻一种叫做关注点分离(separation of concerns)的理念，我们需要将配置信息存放在单独的文件中，和程序文件一起放置在程序目录里。

用一个类来存储所有的配置变量是一种我喜欢的方式，因为它非常的灵活。下面是一个配置类的代码，存储在顶层目录的config.py中::

    import os

    class Config(object):
        SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

很简单是吧？配置信息被定义未类的成员变量。当程序需要更多配置的时候，可以向这个类里增加成员变量。

SECRET_KEY变量是Flask程序的一个重要组成。Flask和一些插件使用这个参数作为加密密钥来生成签名或者令牌。Flask-WTF使用这个参数来防止Web表单受到CSRF(`Cross-Site Request Forgery <http://en.wikipedia.org/wiki/Cross-site_request_forgery>`_)攻击。正如变量名称所示，secret key就应该是秘密的，不应轻易让外人知道。

SECRET_KEY默认读取系统环境变量中的值，如果不存在，则使用一个预定的字符串。这种模式在许多配置中使用，在开发时安全需求较低，可以使用预定的字符串，但是当部署到产品服务器中，就应该为它配置一个系统环境变量。

现在我们有了配置文件，我们还需要告诉Flask去读取并使用它。可以通过 ``app.config.from_object()`` 函数来应用配置::

    from flask import Flask
    from config import Config

    app = Flask(__name__)
    app.config.from_object(Config)

    from app import routes

下面我们可以在Python解释器中看一下SECRET_KEY的值时什么::

    >>> from microblog import app
    >>> app.config['SECRET_KEY']
    'you-will-never-guess'

用户登陆窗体
-------------
Flask-WTF插件用Python类来代表Web表单。类的每一个成员都是表单的一个字段。我们使用一个新的app/forms.py文件来存储所有的web表单。首先，来定义一个用户登陆表单::

    from flask_wtf import FlaskForm
    from wtforms import StringField, PasswordField, BooleanField, SubmitField
    from wtforms.validators import DataRequired

    class LoginForm(FlaskForm):
        username = StringField('Username', validators=[DataRequired()])
        password = PasswordField('Password', validators=[DataRequired()])
        remember_me = BooleanField('Remember Me')
        submit = SubmitField('Sign In')

导入Flask-WTF的类时需要使用flask_wtf的名称，这里我们导入了表单的基类FlaskForm。之后我们导入了4个字段类，这些类时直接从WTForms包中导入的，每个字段都包含了一个描述作为第一个输入变量。validators是可选的参数，它可以将验证器关联到字段上。

表单模板
---------
接下来需要将表单加入到HTML模板中。由于表单类会自动的将自己渲染为HTML代码，因此这项工作十分的简单，下面是将表单插入到app/templates/login.html的代码::

    {% extends "base.html" %}

    {% block content %}
        <h1>Sign In</h1>
        <form action="" method="post">
            {{ form.hidden_tag() }}
            <p>
                {{ form.username.label }}<br>
                {{ form.username(size=32) }}
            </p>
            <p>
                {{ form.password.label }}<br>
                {{ form.password(size=32) }}
            </p>
            <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
            <p>{{ form.submit() }}</p>
        </form>
    {% endblock %}

现在这个模板会需要一个名叫form的表单类作为输入，稍后这些代码会添加到login视图函数中。

HTML代码中的<form>标签是web表单的容器。<form>的action属性是告诉浏览器当用户提交信息时所使用的Http请求方式，默认使用的时GET方式，但这样会在URL中增加许多请求的信息，因此我们在这里使用了POST方式。

form.hidden_tag()函数生成了一个隐藏字段，用于防止csrf攻击。你需要做的仅是配置程序的SECRET_KEY，并且在将form.hidden_tag()添加到表单里，剩下的工作Flask-WTF会帮你完成。

在使用wtforms时，你不需要写任何的HTML字段代码，只需要将 ``{{ form.<field_name>.label }}`` 放置在你需要的位置，他们会自动的被渲染成正确的HTML代码，同时你还可以将CSS类应用到这些字段上。

表单视图
----------
最后一步时写一个新的视图函数来渲染这个表单。我们将新的视图函数仍然添加到app/routes.py文件中::

    from flask import render_template
    from app import app
    from app.forms import LoginForm

    # ...

    @app.route('/login')
    def login():
        form = LoginForm()
        return render_template('login.html', title='Sign In', form=form)

首先从app.forms中导入了之前写好的LoginForm类，然后生成一个LoginForm类的实例，最后再将其作为参数传递给模板页面。``form=form`` 代码中第一个form是指模板页面中的变量form，第二个form指刚刚实例化的LoginForm类。

我们再修改base.html，在导航条中增加一个Login连接::

    <div>
        Microblog:
        <a href="/index">Home</a>
        <a href="/login">Login</a>
    </div>

接收表单数据
---------------
如果你现在点击login按钮，浏览器会返回一个"Method Not Allowed"错误。这是因为我们的login函数还没有配置可以接受Post数据，下面我们改进login函数使之可以接受用户提交的数据::

    from flask import render_template, flash, redirect

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            flash('Login requested for user {}, remember_me={}'.format(
                form.username.data, form.remember_me.data))
            return redirect('/index')
        return render_template('login.html', title='Sign In', form=form)

函数修饰符的改变告诉Flask这个函数可以处理GET和POST请求，默认情况下视图函数只能处理GET请求。HTTP协议声明，GET请求是向客户端返回信息时使用的，POST请求是客户端向服务器发送数据时使用的（事实上GET请求也可以向服务器发送数据，但不推荐这么做）。之前提示的"Method Not Allowed"错误就是因为浏览器发送了一个POST请求，而服务器端没有配置为可以接受POST请求。

**笔记：** 修改login.html使用Get方式时，页面提交后URL会变成下面的样子::

    http://127.0.0.1:5000/login?csrf_token=ImJiYjQ5ZjVmMTczMWY5MTI4MDk3OGRlOGJhMGE4YmYwYmJkMmFjMWEi.DV6gmQ.SBs1Amg-cWNL2mGBIO6dqDHOvjw&username=xxx&password=xxx&submit=Sign+In

这里包含了表单中所输入的所有字段信息，以及一个csrf_token，这个token是由form.hidden_tag()生成的。

form.validate_on_submit()函数完成了表单处理的工作。当服务器使用GET方法时，这个函数会返回False，因此我们可以用这个函数来跳过处理用户提交数据的代码。

当用户提交表单时form.validate_on_submit()函数会收集所有提交的数据，并且运行所有字段上的验证器。如果一切正常它会返回True，但如果出现任意的验证未通过，函数会返回False，这样页面又会向GET请求时那样被返回给用户，稍后我们还会在页面个上添加验证失败时的错误提示信息。

当form.validate_on_submit()验证通过时，login函数会调用flash()函数。flash()函数是一种向用户显示信息的常用方法，这里我们暂时先使用这种方法。另外一个使用的函数是redirect()，这个函数会让浏览器自动跳转到一个新的URL，在这里我们把用户跳转到了/index。 

当你调用flash()时，Flask会存储这一信息，但是被存储的信息并不会在浏览器中显示，我们还需要修改页面模板来显示这一信息。下面我准备将flash信息显示到base.html上::

    <html>
        <head>
            {% if title %}
            <title>{{ title }} - microblog</title>
            {% else %}
            <title>microblog</title>
            {% endif %}
        </head>
        <body>
            <div>
                Microblog:
                <a href="/index">Home</a>
                <a href="/login">Login</a>
            </div>
            <hr>
            {% with messages = get_flashed_messages() %}
            {% if messages %}
            <ul>
                {% for message in messages %}
                <li>{{ message }}</li>
                {% endfor %}
            </ul>
            {% endif %}
            {% endwith %}
            {% block content %}{% endblock %}
        </body>
    </html>

这里我使用了 ``with`` 语句将get_flashed_messages()的结果赋值给了messages变量。get_flashed_messages()函数是Flask自带的，它将所有被flash()存储的信息以列表的形式返回。后面的语句是判断如果messages非空则循环显示所有messages中的信息。

另外值得注意的一点是调用get_flashed_messages后会将返回的信息从flash的存储中删除，也就是说，所有被flash的信息都只能调用一次。

改进字段验证
-------------
验证器是用来防止提交非法数据的手段。通常验证失败时都会重新显示原表单，让用户重新修改错误的字段。

如果你之前尝试提交非法数据，你会发现验证机制工作的十分正常，但是页面上缺少了错误提示，来告诉用户哪里不对。事实上，验证器已经生成了这些错误提示，只是我们还没有把它们放置在页面模板里。下面的代码添加了username和password的验证信息::

    {% extends "base.html" %}

    {% block content %}
        <h1>Sign In</h1>
        <form action="" method="post">
            {{ form.hidden_tag() }}
            <p>
                {{ form.username.label }}<br>
                {{ form.username(size=32) }}<br>
                {% for error in form.username.errors %}
                <span style="color: red;">[{{ error }}]</span>
                {% endfor %}
            </p>
            <p>
                {{ form.password.label }}<br>
                {{ form.password(size=32) }}<br>
                {% for error in form.password.errors %}
                <span style="color: red;">[{{ error }}]</span>
                {% endfor %}
            </p>
            <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
            <p>{{ form.submit() }}</p>
        </form>
    {% endblock %}


我们在username和password字段后面增加了一个循环来显示所有的错误信息。通常来说，所有带有验证器的验证器的字段的错误信息都可以从form.<field_name>.errors中获得，这是一个列表，因为字段可以由多个验证器，因此错误信息也可能不只一个。

现在如果你再尝试提交空的用户名或密码，你或得到一个红色的错误提示。

生成链接
----------
现在登陆表单已经基本完成了，在结束本章前我们再来讨论一下如何正确的再模板和重定向时使用链接。到目前为止所有的链接都是直接定义的，例如再base模板的导航条中::

    <div>
        Microblog:
        <a href="/index">Home</a>
        <a href="/login">Login</a>
    </div>

以及在login函数使用重定向时::

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            # ...
            return redirect('/index')
        # ...

这样直接定义链接的坏处是如果有一天你像重新命名你的链接，那么你就需要到各个地方去逐一修改这些链接名了。

为了更好的管理这些链接，Flask提供了一个函数叫做url_for()，它使用内部的视图函数到URL的映射来生成URL。例如，url_for('login')会返回/login，url_for('index')返回/index。url_for()的输入是视图函数的名字。

你可能会问为什么这样会更方便，事实上相比于视图函数的名字，URL有更大的可能性被改变。另外一个原因是，有些URL可能会使用到动态的内容，这样一来生成这些URL就会需要一些其他的元素，并且变得十分麻烦，而url_for()可以自动生成这些动态URL。

因此，从现在开始当我需要使用URL时，我都会用url_for()来生成，导航条的代码变成了::

        <div>
            Microblog:
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('login') }}">Login</a>
        </div>

下面是login视图函数的代码::

    from flask import render_template, flash, redirect, url_for

    # ...

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            # ...
            return redirect(url_for('index'))
        # ...