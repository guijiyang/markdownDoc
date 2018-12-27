# Django数据库配置和模型搭建

- [Django数据库配置和模型搭建](#django%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E5%92%8C%E6%A8%A1%E5%9E%8B%E6%90%AD%E5%BB%BA)
  - [1. 数据库配置](#1-%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE)
  - [2. 创建模型](#2-%E5%88%9B%E5%BB%BA%E6%A8%A1%E5%9E%8B)
  - [3. 激活模型](#3-%E6%BF%80%E6%B4%BB%E6%A8%A1%E5%9E%8B)
  - [4. 初试API](#4-%E5%88%9D%E8%AF%95api)
  - [5. Django管理员页面](#5-django%E7%AE%A1%E7%90%86%E5%91%98%E9%A1%B5%E9%9D%A2)
    - [5.1. 创建一个管理员账号](#51-%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%AE%A1%E7%90%86%E5%91%98%E8%B4%A6%E5%8F%B7)
    - [5.2. 添加应用](#52-%E6%B7%BB%E5%8A%A0%E5%BA%94%E7%94%A8)
  
## 1. 数据库配置

打开 mysite/settings.py 。这是个包含了 Django 项目设置的 Python 模块。

通常，这个配置文件使用 SQLite 作为默认数据库。如果你想使用其他数据库，你需要安装合适的 database bindings ，然后改变设置文件中 DATABASES 'default' 项目中的一些键值：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#mysite/settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'webDB',
        'USER': 'webuser',
        'PASSWORD': '19910707',
        'OPTIONS':
        {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'"
        }
    }
}
```

- >ENGINE--可选值有'django.db.backends.sqlite3'，'django.db.backends.postgresql'，'django.db.backends.mysql'，或 'django.db.backends.oracle'。其它 可用后端。

- >NAME--数据库名称。如使用SQLite，这将是电脑上的一个文件，这种情况下NAME应该是此文件的绝对路径，包括文件名。默认值 os.path.join(BASE_DIR, 'db.sqlite3') 将会把数据库文件储存在项目的根目录。

如果不适用SQLite，则必须添加一些额外设置，比如USER、PASSWORD、HOST等，具体请查看 [DATABASES]()。

编辑 mysite/settings.py 文件前，先设置 TIME_ZONE 为你自己时区。
此外，INSTANLLED__APPS设置项包括了项目中启用的素有Django应用。

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#mysite/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

__通常， INSTALLED_APPS 默认包括了以下 Django 的自带应用__:

- django.contrib.admin -- 管理员站点， 你很快就会使用它。
  
- django.contrib.auth -- 认证授权系统。
- django.contrib.contenttypes -- 内容类型框架。
- django.contrib.sessions -- 会话框架。
- django.contrib.messages -- 消息框架。
- django.contrib.staticfiles -- 管理静态文件的框架。

默认开启的某些应用需要至少一个数据表，所以，在使用他们之前需要在数据库中创建一些表。请执行以下命令：

```shell
$python manage.py migrate
```

这个命令检查 INSTALLED_APPS 设置，为其中每个应用创建需要的数据表。

> 如果不需要某个应用，可以在运行migrate前从 INSTALLED_APPS 中删除或注释掉。这样 migrate 命令就不会对该应用进行数据库迁移。

## 2. 创建模型

Django创建数据库驱动的WEB应用首先需要定义模型-也就是进行数据库结构设计和附加的其他元数据。

以官方教程的投票系统为例。需要创建两个模型：问题 Question 和选项 Choice。Question 模型包括问题描述和发布时间。Choice 模型有两个字段，选项描述和当前得票数。每个选项属于一个问题。

这些概念可以通过一个简单的python类来描述。编辑polls/models.py文件：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#polls/models.py

from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

每一个模型都是django.db.models.Model类的子类。每一个模型有些类变量，表示的是模型里的一个数据库字段。

每一个字段都是 [Field]() 类的实例(字符字段CharField，日期时间字段DateTimeField)。这些字段表示要处理的数据类型。

每一个Field类实例变量的名字(question_text或pub_date)也是字段名。数据库会将它们作为列名。

你可以使用可选的选项来为 Field 定义一个人类可读的名字。这个功能在很多 Django 内部组成部分中都被使用了，而且作为文档的一部分。如果某个字段没有提供此名称，Django 将会使用对机器友好的名称，也就是变量名。在上面的例子中，我们只为 Question.pub_date 定义了对人类友好的名字。对于模型内的其它字段，它们的机器友好名也会被作为人类友好名使用。

在最后，这里使用 [ForeignKey]() 定义了一个关系。这将告诉 Django，每个 Choice 对象都关联到一个 Question 对象。Django 支持所有常用的数据库关系：多对一、多对多和一对一。

## 3. 激活模型

创建完模型后，首先需要把polls应用安装到我们的项目中。我们需要在配置类 INSTALLED_APPS 中添加设置。添加如下所示：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#mysite/settings.py

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'blogs.apps.BlogsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

现在项目中已经包含了polls应用(和blog应用)。运行下面的命令：

```shell
$python manage.py makemigrations polls
```

通过运行 makemigrations 命令，Django 会检测你对模型文件的修改（在这种情况下，你已经取得了新的），并且把修改的部分储存为一次 迁移。模型的迁移数据，它被储存在 polls/migrations/0001_initial.py 里。

移命令会执行哪些 SQL 语句。sqlmigrate 命令接收一个迁移的名称，然后返回对应的 SQL：

```shell
python manage.py sqlmigrate polls 0001
```

可以看到下面输出：

```mysql
BEGIN;
--
-- Create model Choice
--
CREATE TABLE `polls_choice` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `choice_text` varchar(200) NOT NULL, `votes` integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE `polls_question` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `question_text` varchar(200) NOT NULL, `pub_date` datetime(6) NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE `polls_choice` ADD COLUMN `question_id` integer NOT NULL;
ALTER TABLE `polls_choice` ADD CONSTRAINT `polls_choice_question_id_c5b4b260_fk_polls_question_id` FOREIGN KEY (`question_id`) REFERENCES `polls_question` (`id`);
COMMIT;
```

现在再次运行 ```python manage.py migrate``` 命令，在数据库里创建新定义的模型的数据表
,以下是输出信息：

```shell
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

这个 migrate 命令选中所有还没有执行过的迁移（Django 通过在数据库中创建一个特殊的表 django_migrations 来跟踪执行过哪些迁移）并应用在数据库上 - 也就是将你对模型的更改同步到数据库结构上。

迁移是非常强大的功能，它能让你在开发过程中持续的改变数据库结构而不需要重新删除和创建表 - 它专注于使数据库平滑升级而不会丢失数据。我们会在后面的教程中更加深入的学习这部分内容，现在，你只需要记住，改变模型需要这三步：

编辑models.py文件，改变模型

- *编辑 `models.py` 文件，改变模型。*
- *运行 `python manage.py makemigrations` 为模型的改变生成迁移文件。*
- *运行 `python manage.py migrate` 来应用数据库迁移。*

## 4. 初试API

现在交互式尝试下Django创建的各种API，通过一下命令打开：

`python manage.py shell`

```python shell
>>> from polls.models import Choice, Question 
>>> Question.objects.all()
<QuerySet []>
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())
>>> q.save()
>>> q.id
1
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2018, 12, 17, 2, 47, 56, tzinfo=<UTC>)
>>> q.question_text = "What's up?"
>>> q.save()
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>
```

接下来，给 Question 和 Choice 增加 \__str__() 方法。

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

# polls/models.py

from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

除去常规的Python方法之外，我们可以自定义方法。

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

# polls/models.py

import datetime

from django.db import models
from django.utils import timezone

class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

## 5. Django管理员页面

### 5.1. 创建一个管理员账号

运行创建管理员账号的命令 `python manage.py createsuperuser`，
依次输入名称、邮件地址和密码。然后就可以启动开发服务器 `python manage.py runserver`。

现在打开浏览器的admin目录，就可以看见管理员登录界面。

### 5.2. 添加应用

但是我们的投票应用在哪呢？它没在索引页面里显示。

只需要做一件事：我们得告诉管理页面，问题 Question 对象需要被管理。打开 polls/admin.py 文件，把它编辑成下面这样：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

# polls/admin.py
from django.contrib import admin
from .models import Question
# Register your models here.

admin.site.register(Question))
```

打开admin管理界面，显示如下：

![django-learning-2](https://github.com/guijiyang/markdownDoc/raw/master/img/django%20learning--2.png "django-learning")

在这里可以添加、修改和删除访问管理员站点的用户，也可以添加投票的问题，如果包含了blog应用，也可以添加blog信息。