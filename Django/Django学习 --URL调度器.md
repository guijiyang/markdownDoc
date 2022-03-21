# URL调度器

- [URL调度器](#url%E8%B0%83%E5%BA%A6%E5%99%A8)
  - [1. 概况](#1-%E6%A6%82%E5%86%B5)
  - [2. Django如何处理一个请求](#2-django%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%E4%B8%80%E4%B8%AA%E8%AF%B7%E6%B1%82)
  - [3. 路径转换器](#3-%E8%B7%AF%E5%BE%84%E8%BD%AC%E6%8D%A2%E5%99%A8)
  - [4.自定义路径转换器](#4%E8%87%AA%E5%AE%9A%E4%B9%89%E8%B7%AF%E5%BE%84%E8%BD%AC%E6%8D%A2%E5%99%A8)
  - [5. 使用正则表达式](#5-%E4%BD%BF%E7%94%A8%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)

## 1. 概况

Django创建一个URL配置的Python模块，用于一个应用的URLs设置。这个模块将URL路径表达式映射到Python函数(views)。

Django 还提供根据当前语言翻译URL 的一种方法。更多信息参见 国际化文档。

## 2. Django如何处理一个请求

请求一个页面时，Django系统决定执行哪个Python代码的算法：

- >通常Django通过 `ROOT_URLCONF` 的值来确定使用的根URL配置模块(一个python文件)，但是如果输入的HttpRequest对象有urlconf属性(由中间层设置)，将根据这个值确定URL配置模块。
- >Django加载相应模块并找寻变量urlpatterns。这必须是django.urls.path()和/或django.urls.re_path()的实例。
- >Django依次匹配每个URL模式，遇到第一个匹配的URL时停下。
- >一旦匹配成功，Django导入并调用指定的view，这通常是一个简单的Python方程(或者基于类的view)。view通常传递以下几个参数：
  - >一个HttpRequest实例;
  - >如果匹配的URL模式没有返回命令的簇(groups)，那么正则表达式匹配的值讲作为位置参数。
  - >关键字参数由匹配路径表达式的任何名称部分构成， 替代django.urls.path()中的kwargs部分的参数。
- >如果没有URL模式匹配，或在这个过程中产生一个exception，Django触发一个合适的错误处理view。

例如下面的URL配置:

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

from django.urls import path
from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
]
```

可以看到：

- 使用尖括号从URL中捕获值。
- 捕获的值可以包含一个转换器类型，如\<int:name>捕获一个integer参数。如果不指定int这个类型，任意的string都匹配。
- urlpatterns开始不需要/，因为每个URL都有/。
  
## 3. 路径转换器

以下的路径转换默认可用：
  
- str:匹配素有非空字符串，当然其中没有/，这个是默认的转换器。
- int:匹配非负integer。返回一个int。
- slug:匹配所有带有-和_的字符串。
- uuid:匹配素有格式化的UUID。为防止大量的URLs映射到相同的页面，需要使用破折号和小写字母。
- path:匹配非空字符串，并包含/。这个可用于匹配整个URL路径。

## 4.自定义路径转换器

创建一个传唤器类，包含：

- 一个regex类属性，作为一个string。
- 一个to_python(self, value)方法，用于转换匹配的字符串到指定的类型用于view函数。如果转换失败引起ValueError异常。
- 一个to_url(self, value)方法，转换Python类型为string用于URL中。

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-
class FourDigitYearConverter:
    regex = '[0-9]{4}'

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return '%04d' % value
```

使用方法 `register_converter()` 将转换器类注册到URLconf中：

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

from django.urls import path, register_converter

from . import converters, views

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<yyyy:year>/', views.year_archive),
    ...
]
```

## 5. 使用正则表达式

如果路径和转换器语法不满足需要的URL模式，还可以使用正则表达式。这时就需要使用 `re_path()` 。

在Python语法中一个正则表达式簇是(?P\<name>pattern)，name表示簇的名称，pattern表示匹配的模式。
如下所示一个利用正则表达式的URLconf:

```python
#! /usr/bin/python3
# -*-coding:utf8 -*-

from django.urls import path, re_path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
    re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<slug>[\w-]+)/$', views.article_detail),
]
```

组要注意这里捕获的参数都是string，不管具体的正则表达式匹配过程。所以当`path()`切换到`re_path()`需要注意到参数的变化。

也可以使用无名正则表达式簇，如 `([0-9]{4})`。