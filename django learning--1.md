# Django入门

- [Django入门](#django%E5%85%A5%E9%97%A8)
  - [1. 创建项目](#1-%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE)
  - [2. 创建应用](#2-%E5%88%9B%E5%BB%BA%E5%BA%94%E7%94%A8)
  - [3. 编写第一个视图](#3-%E7%BC%96%E5%86%99%E7%AC%AC%E4%B8%80%E4%B8%AA%E8%A7%86%E5%9B%BE)
  
## 1. 创建项目

打开终端(命令行)，切换到代码的目录，然后运行一下命令：

```shell
$django-admin startproject mysite
```

startproject命令创建了:

```shell
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

这些目录和文件的用处如下：

- >最外层的mysite/根目录只是你项目的容器，名称可以随意指定。
  
- >mange.py是一个管理Django项目的命令行工具[django-admin and manage.py](#2-数据库配置)获取所有manage.py的细节。
- >里面一层的mysite/目录包含你的项目，这是一个纯Python包。包名就是引用它内部任何东西需要用到的(mysite.urls)。
- >mysite\/\__init__.py：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包。如果你是 Python 初学者，阅读官方文档中的 更多关于包的知识。
- >mysite/settings.py：Django 项目的配置文件。如果你想知道这个文件是如何工作的，请查看 [Django settings](#) 了解细节。
- >mysite/urls.py：Django 项目的 URL 声明，就像你网站的“目录”。阅读 [URL调度器]() 文档来获取更多关于 URL 的内容。
- >mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。阅读 [如何使用 WSGI 进行部署]() 了解更多细节。

在主目录mysite/下运行下面的命令：

```shell
$python manage.py runserver
```

就可以启动Django自带的用于开发的简易服务器。默认情况下，runserver命令会将服务器设置为监听
本机内部的IP的8000端口。如果需要更改服务器端口，使用如下命令：

```shell
$python manage.py runserver 8080
```

## 2. 创建应用

现在项目已经配置妥当，接着创建应用。应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库，或者简单的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用。应用可以被很多个项目使用。

创建应用命令：

```shell
$python manage.py startapp polls
```

这将会创建一个polls目录，它的目录结构大致如下：

```shell
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

## 3. 编写第一个视图

打开polls/views.py，填入一下代码：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#polls/views.py

from django.http import HttpResponse

def index(request):
    return HttpResponse('Hello world')
```

这样就可以创建一个最简单的视图。接着在polls目录中新建一个urls.py，并输入以下代码:

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#polls/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

下一步是要在根 URLconf 文件中指定我们创建的 polls.urls 模块。在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include()， 如下：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

#mysite/urls.py

from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

__函数include()__:

函数include()允许引用其它URLconfs。每当 Django 遇到 :func：~django.urls.include 时，它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理。

我们设计 include() 的理念是使其可以即插即用。因为投票应用有它自己的 URLconf( polls/urls.py )，他们能够被放在 "/polls/" ， "/fun_polls/" ，"/content/polls/"，或者其他任何路径下，这个应用都能够正常工作。

__函数path()__:

函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name。现在，是时候来研究这些参数的含义了。

route:是一个匹配URL的准则(类似正则表达式)。当Django响应一个请求时，它会从urlpatterns的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项。

view:当Django找到一个匹配的准则，就会调用这个特定的视图函数(比如polls.urls.py中调用的views.index)，并传入一个HttpRequest对象作为第一个参数，被捕获的参数以关键字参数的形式传入。

kwargs:可以作为一个字典传递给目标视图函数。这里不做描述。

name:为URL取名能使你的Django的任意地方唯一的引用它，尤其是在模板中。这个有用的特性允许你只改变一个文件就能全局的修改某个URL模式。
