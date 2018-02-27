.. _Chapter4:

第四章:数据库(Database)
==========================
这一章的内容十分重要，因为对于大多数应用程序，都需要一个高效的存取数据的手段，而这正是数据库所擅长的。

这是本章的GitHub链接:`Browse <https://github.com/miguelgrinberg/microblog/tree/v0.4>`_, `Zip <https://github.com/miguelgrinberg/microblog/archive/v0.4.zip>`_, `Diff <https://github.com/miguelgrinberg/microblog/compare/v0.3...v0.4>`_。

在Flask中使用数据库
--------------------
Flask原生并不支持数据库，因此你可以自由的选择最适合你的应用的数据库。

数据库可以分为两大类：一类符合关系模型，叫做关系数据库；另一类不符合，叫做非关系数据库(NoSQL)。我认为，关系数据库可以更好的适应结构化的数据例如用户、发表的日志等等，而非关系数据库更适合非规整的数据。在本文中，两种数据库都可以适用，但是我准备使用关系数据库。

在 :ref:`Chapter2` 中我们使用了第一个Flask插件。在本章中我们要使用两个新插件，第一个是 `Flask-SQLAlchemy <http://packages.python.org/Flask-SQLAlchemy>`_，它提供了 `SQLAlchemy <http://www.sqlalchemy.org/>`_  包的FLask封装。SQLAlchemy是一个对象关系映射器(`Object Relational Mapper <http://en.wikipedia.org/wiki/Object-relational_mapping>`_，简称ORM)。ORM允许用户使用类来操作数据库，将数据表和查询语句映射为对象和方法，ORM的任务是将对类对象的操作转换为数据库命令。

SQLAlchemy最棒的一点是，它是许多关系数据库的ORM，包括MySQL、PostgreSQL和SQLite。这是非常强大的功能，因为你可以在开发时是哦那个一个轻量级的数据库例如SQLite，当发布产品时可以切换到MySQL或PostgreSQL上，而不需要更改程序代码。

进入虚拟环境后安装Flask-SQLAlchemy::

    (venv) $ pip install flask-sqlalchemy

数据库迁移
--------------
我所见过的大多数数据库教程都涉及到数据库的创建和使用，但在应用程序需要更改或增长时，没有充分解决对现有数据库进行更新的问题。这是一个很困难的问题，因为关系数据库是围绕结构化的数据建立的，所以当数据的结构变化时，已经存储在数据库中的数据就很难迁移到新的结构上去。

下面要介绍第二个插件 `Flask-Migrate <https://github.com/miguelgrinberg/flask-migrate>`_，这是SQLAlchemy迁移工具 `Alembic <https://bitbucket.org/zzzeek/alembic>`_ 的Flask封装。使用这个工具需要在数据库建立之前做一些准备工作，但会使得你的数据库在之后的开发过程中稳定性更好。

安装Flask-Migrate和之前其他的插件类似::

    (venv) $ pip install flask-migrate

Flask-SQLAlchemy配置
------------------------------
在开发过程中，我准备使用SQLite，这是开发小型应用的最佳选择。它的数据库存储在单个文件中，不需要运行服务程序。

下面我们有两个新的配置要加入配置文件config.py中::

    import os
    basedir = os.path.abspath(os.path.dirname(__file__))

    class Config(object):
        # ...
        SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
            'sqlite:///' + os.path.join(basedir, 'app.db')
        SQLALCHEMY_TRACK_MODIFICATIONS = False


Flask-SQLAlchemy插件从SQLALCHEMY_DATABASE_URI变量中获取数据库的位置。The SQLALCHEMY_TRACK_MODIFICATIONS被设为False，这样可以在我操作数据库时屏蔽Flask-SQLAlchemy发出的信息。

数据库在程序中需要一个实例来表示，数据库迁移引擎也需要一个实例，这些都添加在app/__init__.py文件中::

    from flask import Flask
    from config import Config
    from flask_sqlalchemy import SQLAlchemy
    from flask_migrate import Migrate

    app = Flask(__name__)
    app.config.from_object(Config)
    db = SQLAlchemy(app)
    migrate = Migrate(app, db)

    from app import routes, models

在init脚本中，我做了三件事：一是增加了一个db对象表示数据库，二是添加了另一个对象migrate表示数据库迁移引擎，最后导入了一个新的models模块，这个模块用来定义数据库结构。

数据库模型
-----------
用类来表示需要被存储在数据库中的数据，ORM层会把从类结构翻译成数据库模型。下面我们使用 `WWW SQL Designer tool <http://ondras.zarovi.cz/sql/demo/>`_ 来创建一个表示用户的模型

.. image:: /_static/ch04-users.png

id字段是主键，每个用户都会有一个不同的值。通常主键的值是数据库自动生成的，所以我们只需要将这个字段标记为主键即可

username、email、password_hash三个字段是string类型（数据库中通常叫做varchar），并且定义了最大长度，这样数据库可以优化存储空间。username、email字段的含义不言自明。password_hash是用户密码的hash值，为了安全考虑，我们没有直接存储用户明文密码。直接存储明文密码的坏处是，一旦数据库被破解，黑客就能直接得到用户密码，这对用户来说是灾难性的。存储用户密码的hash值可以极大的提高密码安全，这将在后续章节中讨论。

现在我们把上述模型用python类来实现，代码存储在app/models.py中::

    from app import db

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(64), index=True, unique=True)
        email = db.Column(db.String(120), index=True, unique=True)
        password_hash = db.Column(db.String(128))

        def __repr__(self):
            return '<User {}>'.format(self.username)

上面创建的User类继承自db.Model，db.Model是Flask-SQLAlchemy中所有模型的基类。User类定义了一系列字段作为类成员。这些字段都是db.Column类的实例，这个类接受字段类型作为输入，还可以接受其他可选参数，例如我们可以将一个字段标识为唯一(unique)和索引(indexed)，正确使用这些标识可以使数据库更加高效。

__repr__函数告诉Python如何显示(print)这个类，这对调试十分有帮助，你可以在Python解释器中测试这个函数的用法::

    >>> from app.models import User
    >>> u = User(username='susan', email='susan@example.com')
    >>> u
    <User susan>

创建迁移存储库(Migration Repository)
--------------------------------------
上一节中创建的模型定义了本应用程序的初始数据库结构。但是当程序持续开发后，很有可能需要更改初始的结构，比如增加新字段、修改删除老字段等。Alembic会将这些改动一一记录为脚本，在需要时可以重新构建整个数据库。

为了完成这项工作，Alembic会维护一个迁移存储库，这是一个存储迁移脚本的文件夹。每当数据库结构发生改变时，存储库中就会添加一个迁移脚本，详细记录了改动内容。当需要迁移数据库时，顺序执行所有的迁移脚本即可完成新数据库的创建。

Flask-Migrate将命令集成到了flask命令下。你已经使用过了 ``flask run``，这是一个flask原生的命令。``flask db`` 命令是Flask-Migrate插件添加的，可以完成所有和数据库迁移有关的工作。下面我们看一下如何创建迁移存储库:

.. code-block:: shell

    (venv) $ flask db init
        Creating directory /home/miguel/microblog/migrations ... done
        Creating directory /home/miguel/microblog/migrations/versions ... done
        Generating /home/miguel/microblog/migrations/alembic.ini ... done
        Generating /home/miguel/microblog/migrations/env.py ... done
        Generating /home/miguel/microblog/migrations/README ... done
        Generating /home/miguel/microblog/migrations/script.py.mako ... done
        Please edit configuration/connection/logging settings in
        '/home/miguel/microblog/migrations/alembic.ini' before proceeding.

记住flask命令需要FLASK_APP环境变量，在这个程序中，你需要设置 ``FLASK_APP=microblog.py``，具体可以参见 :ref:`Chapter1`。

当你运行完上述命令，你会看到一个新的迁移文件夹，里面有一些新文件和一个版本子目录。从现在起，这些文件都是当前项目的一部分，而且应该被添加到版本控制软件中。

第一个数据库迁移
------------------
在迁移存储库建立好之后，我们来创建第一个迁移记录，包含了数据库中的User表。有两种创建迁移记录的方法：手动或者自动。当自动生成时，Alembic比较由给定的数据库模型(database models)定义的数据库模式(database schema)，和实际存在的数据库模式。然后根据变化生成迁移脚本，使得数据库模式和程序中定义的数据库模型相对应。在这里，因为之前不存在数据库，自动迁移将会将整个User模型添加到迁移脚本中。命令如下:

.. code-block:: shell

    (venv) $ flask db migrate -m "users table"
    INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
    INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
    INFO  [alembic.autogenerate.compare] Detected added table 'user'
    INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
    INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
    Generating /home/miguel/microblog/migrations/versions/e517276bb1c2_users_table.py ... done

命令行输出会告诉你Alembic添加了什么到迁移脚本中。前两行可以忽略，然后它说发现了user表和两个索引，并且标明了它写在了迁移记录的哪里。``e517276bb1c2`` 是自动生成的唯一标识，``-m`` 选项添加的注释是可选的，它是一个迁移记录的简单注释。

生成号的迁移记录是工程的一部分，需要添加到版本控制中。你可以查看一下脚本的内容，你会发现它包含两个函数分别是upgrade()和downgrade()，分别用来升级和降级数据库。

``flask db migrate`` 命令并没有对数据库做任何的修改，它只是将改动写入了迁移记录。为了应用这些改动我们还需要执行 ``flask db upgrade`` 命令:

.. code-block:: shell

    (venv) $ flask db upgrade
    INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
    INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
    INFO  [alembic.runtime.migration] Running upgrade  -> e517276bb1c2, users table

因为本项目使用了SQLite，``upgrade`` 命令会检测到数据库文件不存在，并且自动创建它。当使用数据库服务例如MySQL或者PostgreSQL时，你需要在运行 ``upgrate`` 之前创建好数据库。

数据库升级和降级工作流
------------------------------
尽管现在整个应用程序还处在它的婴儿期，但我们仍然可以先讨论一下数据库迁移的策咯。假设你的应用程序正在开发机上进行开发，同时还部署在一台正在使用中的服务器上。

如果你现在想对下一版发布的应用程序的数据库模型进行一些修改，比如增加一个表之类的。在没有迁移之前你需要考虑如何对数据库进行修改，并且这个改动要同时应用在开发机和服务器上，这是一个很复杂的工作。

在有了数据库迁移工具之后，每当你修改了数据库结构，你可以生成一个新的迁移脚本(``flask db migrate``)，在检查过脚本正确性后就可以将其应用在开发数据库上(``flask db updated``)，然后将迁移脚本添加到源代码管理中。

当你准备发布这个新版本到产品服务器上时，你需要做的仅仅是下载最新版本的程序，这之中包含了迁移脚本，然后运行 ``flask db upgrade``。Alembic会检测服务器上的数据库不是最新版本，并且自动运行所需的迁移脚本。

当然，还有一个 ``flask db downgrade`` 命令，用来撤销上一次的迁移。这个操作在产品服务器上很少使用，但在程序开发过程中却很使用。当你应用了一个迁移脚本之后，发现你并不需要这些改动时，你可以使用降级命令来回退到之前的数据库版本，然后重新进行开发。

数据库关系
------------
关系数据库擅长存储相互关联的数据。在用户发表一片blog时，在users表中会有一条用户记录，在posts表中会有这篇blog的记录。最有效的存储方式是将两条记录进行关联。

当一个用户到blog的关联建立后，数据库就可以响应关于这个关联的查询。最常见的查询是你想查找某篇blog的作者。还有更复杂的查询，比如你想知道某个用户发表的所有blog。Flask-SQLAlchemy可以帮助我们解决上述问题。

让我们来扩展一下当前数据库，来存储发表的blog，这是新的数据库模式:

.. image:: /_static/ch04-users-posts.png

posts表包括id，body(日志内容)，和一个时间戳。除此之外还有一个user_id字段，关联到它的作者。每个用户都有唯一的id作为主键，将blog和用户关联起来的方法是添加一个用户id的引用，也就是user_id字段，这个字段叫做外键。上面的数据库图表中外键用一个连线表示。这种关系是“一对多”关系，因为一个用户可以写很多blog。

更改后的app/models.py文件内容如下::

    from datetime import datetime
    from app import db

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(64), index=True, unique=True)
        email = db.Column(db.String(120), index=True, unique=True)
        password_hash = db.Column(db.String(128))
        posts = db.relationship('Post', backref='author', lazy='dynamic')

        def __repr__(self):
            return '<User {}>'.format(self.username)

    class Post(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        body = db.Column(db.String(140))
        timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
        user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

        def __repr__(self):
            return '<Post {}>'.format(self.body)

新的Post类表示用户写的blog。timestamp字段添加了索引，在按照时间顺序获取所有blog时很有用，同时还添加了datetime.utcnow作为默认值，当默认值是一个函数时，SQLAlchemy会调用这个函数（注意函数后并没有 ``()``）。通常在服务器应用上都医用UTC时间和日期，这样能保证不论用户在哪个时区都使用统一的时间，显示时时间戳会转换成用户当前时区。

user_id字段初始化成一个关联到 ``user.id`` 的外键，这表明它会从user表中引用一个值。这里user是数据库表的名字，Flask-SQLAlchemy会自动将模型类的名字转换为小写。User类还新增了一个posts字段，初始化为 ``db.relationship`` 类型。这不是一个真实的数据库字段，它是一个在用户和日志之间的高层次视图，因此没有在数据库图表中出现。对于一个“一对多”的关系，``db.relationship`` 字段通常定义在“一”这端，作为获取“多”端数据的快捷手段。例如，一个用户数据存储在变量 ``u`` 中，``u.posts`` 表达式会运行一个数据库查询来获取所有这个用户发表的日志。``db.relationship`` 的第一个参数指明了“多”端。``backref`` 参数是一个用来添加到“多”端的字段名，这个字段指向“一”端的对象。它添加了一个 ``post.author`` 表达式，返回当前日志的作者。``lazy`` 变量定义了这个关系的数据库查询如何执行，这在以后会讨论。在本文结尾会有一个例子来解释上述文字。

在完成了数据库更新后，还需要添加一个迁移脚本:

.. code-block:: shell

    (venv) $ flask db migrate -m "posts table"
    INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
    INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
    INFO  [alembic.autogenerate.compare] Detected added table 'post'
    INFO  [alembic.autogenerate.compare] Detected added index 'ix_post_timestamp' on '['timestamp']'
    Generating /home/miguel/microblog/migrations/versions/780739b227a7_posts_table.py ... done

然后将其应用到数据库上:

.. code-block:: shell

    (venv) $ flask db upgrade
    INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
    INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
    INFO  [alembic.runtime.migration] Running upgrade e517276bb1c2 -> 780739b227a7, posts table

记得把迁移脚本添加到源代码管理中。

游戏时间
----------
前面我们花了很长时间来定义数据库，但并没有展示它是如何工作的。由于程序中还没有关于数据库的操作代码，我们现在先使用Python解释器来熟悉一下数据库操作。确保你正处在虚拟环境中，然后运行 ``python``。

进入到Python提示符后，我们首先导入数据库实例和模型::

    >>> from app import db
    >>> from app.models import User, Post

然后创建一个新用户::

    >>> u = User(username='john', email='john@example.com')
    >>> db.session.add(u)
    >>> db.session.commit()

对数据库的所有操作都是在一个会话(session)中完成的，可以通过 ``db.session`` 来获取会话对象。在一个session中可以应用多项改动，当所有改动都执行完后，只需要执行一次 ``db.session.commit()`` 就可将所有改动自动保存到数据库中。如果在session中出现了一个错误，那么程序会自动调用 ``db.session.rollback()`` 来撤销这个会话中的所有改动。记住只有当执行 ``db.session.commit()`` 时数据库的改动才会被应用到数据库上。会话保证了数据库中的内容和程序代码的执行不会出现不同步的情况。

让我们再添加一个用户::

    >>> u = User(username='susan', email='susan@example.com')
    >>> db.session.add(u)
    >>> db.session.commit()

我们可以执行一个查询来得到所有的用户::

    >>> users = User.query.all()
    >>> users
    [<User john>, <User susan>]
    >>> for u in users:
    ...     print(u.id, u.username)
    ...
    1 john
    2 susan
    
每个模型都有一个 ``query`` 属性，这是数据库查询的入口。最基本的查询是返回所有的这个类的条目，这个函数叫做 ``all()``。注意，我们之前添加用户时，这些用户的id被自动设成了1和2。

下面是另外一种查询，当你知道了一个用户的id，你可以使用下面的方法得到这个用户::

    >>> u = User.query.get(1)
    >>> u
    <User john>

现在我们再添加一个用户日志::

    >>> u = User.query.get(1)
    >>> p = Post(body='my first post!', author=u)
    >>> db.session.add(p)
    >>> db.session.commit()

我没有给 ``timestamp`` 字段赋值，因为这个字段有默认值，你可以模型定义的部分找到。那么 ``user_id`` 字段是如何赋值的呢？回想一下我们在创建 ``User`` 类时添加的 ``db.relationship``，我们给每个用户添加了一个 ``posts`` 属性，还在给每个日志添加了一个 ``author`` 属性。我使用了虚拟字段 ``author`` 来给一个日志赋值，而没有去处理用户ID，SQLAlchemy会自动处理这些，它提供了一个关系和外键的高层抽象。

我们再来看一些其他查询::

    >>> # get all posts written by a user
    >>> u = User.query.get(1)
    >>> u
    <User john>
    >>> posts = u.posts.all()
    >>> posts
    [<Post my first post!>]

    >>> # same, but with a user that has no posts
    >>> u = User.query.get(2)
    >>> u
    <User susan>
    >>> u.posts.all()
    []

    >>> # print post author and body for all posts 
    >>> posts = Post.query.all()
    >>> for p in posts:
    ...     print(p.id, p.author.username, p.body)
    ...
    1 john my first post!

    # get all users in reverse alphabetical order
    >>> User.query.order_by(User.username.desc()).all()
    [<User susan>, <User john>]

`Flask-SQLAlchemy文档 <http://packages.python.org/Flask-SQLAlchemy/index.html>`_ 是学习各类查询命令的最好的方法。

在结束这一节之前，我们把之前添加的用户的日志全部清除，这样数据库就是干净的，可以供下一章只用::

    >>> users = User.query.all()
    >>> for u in users:
    ...     db.session.delete(u)
    ...
    >>> posts = Post.query.all()
    >>> for p in posts:
    ...     db.session.delete(p)
    ...
    >>> db.session.commit()

Shell Context
-------------
记得上一节我们打开Python解释器后干的第一件事么，我们首先执行了一些 ``import``::

    >>> from app import db
    >>> from app.models import User, Post

当你在编写你的程序时，会经常用到Python shell，所以每次都执行 ``import`` 就会十分繁琐。``flask shell`` 命令是另外一个十分有用的工具。``shell`` 命令是排在 ``run`` 命令之后Flask第二重要的命令。这个命令的目的是在程序上下文（context）中启动一个Python解释器。下面是一个例子::

    (venv) $ python
    >>> app
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    NameError: name 'app' is not defined
    >>>

    (venv) $ flask shell
    >>> app
    <Flask 'app'>

在一般的解释器会话中，``app``变量是无法被识别的，但使用了 ``flask shell`` 后，会自动导入当前flask程序的实例。除此之外，你还可以配置一个“shell上下文”（shell context），其中包含了一系列需要导入的变量。

下面一个函数写在microblog.py中，创建了一个shell context，并且添加了一些数据库的模型和实例::

    from app import app, db
    from app.models import User, Post

    @app.shell_context_processor
    def make_shell_context():
        return {'db': db, 'User': User, 'Post': Post}

``app.shell_context_processor`` 装饰器将当前函数注册为shell context函数。当使用 ``flask shell`` 命令时，会自动调用这个函数并且将它返回的变量注册到shell会话中。返回值使用字典而不是列表的原因是，每一个变量都有一个在shell中引用的名称，这个名称通过字典的key定义。

添加完shell context处理函数后，你就可以在不导入数据库实体的前提下进行数据库操作了::

    (venv) $ flask shell
    >>> db
    <SQLAlchemy engine=sqlite:////Users/migu7781/Documents/dev/flask/microblog2/app.db>
    >>> User
    <class 'app.models.User'>
    >>> Post
    <class 'app.models.Post'>