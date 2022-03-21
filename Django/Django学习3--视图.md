# 视图

- [视图](#%E8%A7%86%E5%9B%BE)
  - [1. 概况](#1-%E6%A6%82%E5%86%B5)
  - [2. 编写更多视图](#2-%E7%BC%96%E5%86%99%E6%9B%B4%E5%A4%9A%E8%A7%86%E5%9B%BE)
  - [3. 一个真正的视图](#3-%E4%B8%80%E4%B8%AA%E7%9C%9F%E6%AD%A3%E7%9A%84%E8%A7%86%E5%9B%BE)
  - [4. 抛出404错误](#4-%E6%8A%9B%E5%87%BA404%E9%94%99%E8%AF%AF)
  - [5. 使用模板系统](#5-%E4%BD%BF%E7%94%A8%E6%A8%A1%E6%9D%BF%E7%B3%BB%E7%BB%9F)
  - [6. 去除模板中的硬编码URL](#6-%E5%8E%BB%E9%99%A4%E6%A8%A1%E6%9D%BF%E4%B8%AD%E7%9A%84%E7%A1%AC%E7%BC%96%E7%A0%81url)
  - [7. 为URL名称添加命名空间](#7-%E4%B8%BAurl%E5%90%8D%E7%A7%B0%E6%B7%BB%E5%8A%A0%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4)
  
## 1. 概况

Django中的视图的概念是一类具有相同功能和模板的网页的集合。比如，在一个博客应用中，你可能创建如下几个视图：

- 博客首页--展示最近的吉祥内容。
- 内容详情页--详细展示某项内容。
- 以年、月、天为单位的归档页--展示选中的年、月、天中的所创建的内容。
- 评论处理器--用于响应为一项内容添加评论的操作。

而在我们的投票应用中，我们需要下列几个视图：

- 问题索引页--展示最近的几个投票问题。
- 问题详情页--展示某个投票的问题和不带结果的选项列表。
- 问题结果页--展示某个投票的结果。
- 投票处理器--用于响应用户为某个问题的特定选项投票的操作。

django中每一个视图表现为一个简单的Python函数(或方法，如果在基于类的视图中的话)。django会根据URL域名之后的部分来选择使用哪个视图。

## 2. 编写更多视图

现在向polls/views.py中添加视图：

```Python
#! /usr/bin/python3
# -*-coding:utf8 -*-

# polls/views.py

def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

并在polls.urls模块中添加这些视图：

```Python
#! /usr/bin/python3
# -*-coding:utf8 -*-

# polls/urls.py

from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

然后在浏览器中转到`/polls/34/`，Django将会运行detail()方法并且展示URL提供的问题。再试试 `/polls/34/result/` 和 `/polls/34/vote/` ——你将会看到暂时用于占位的结果和投票页。

## 3. 一个真正的视图

每个视图必须要做的只有两件事：返回一个包含被请求页面内容的HttpResponse对象，或者跑出一个异常，比如Http404。

在index()函数中插入一些新内容，让它展示最近的5个投票问题，以空格分割：

```Python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

这里有个问题：页面的设计写死在视图函数的代码里的。如果你想改变页面的样子，你需要编辑 Python 代码。所以让我们使用 Django 的模板系统，只要创建一个视图，就可以将页面的设计从代码中分离出来。

首先，在你的 polls 目录里创建一个 templates 目录。Django 将会在这个目录里查找模板文件。项目的 `TEMPLATES` 配置项描述了 Django 如何载入和渲染模板。默认的设置文件设置了 DjangoTemplates 后端，并将 APP_DIRS 设置成了 True。这一选项将会让 DjangoTemplates 在每个 INSTALLED_APPS 文件夹中寻找 "templates" 子目录。这就是为什么尽管我们没有像在第二部分中那样修改 DIRS 设置，Django 也能正确找到 polls 的模板位置的原因。

在你刚刚创建的 templates 目录里，再创建一个目录 polls，然后在其中新建一个文件 index.html 。换句话说，你的模板文件的路径应该是 polls/templates/polls/index.html 。因为 Django 会寻找到对应的 app_directories ，所以你只需要使用 polls/index.html 就可以引用到这一模板了。

**polls/templates/polls/index.html**:

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

修改polls/views.py中的index视图来使用模板：

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

Django提供了一个快捷函数render()实现载入模板，填充上下文，再返回由它生成的 HttpResponse 对象，重写index()视图:

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

这样就不再需要loader和HttpResponse。但是其他函数需要用到HttpResponse的地方还是要导入。

## 4. 抛出404错误

下面是一个抛出404错误的视图代码：

```python
rom django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

这里如果question_id没有对应的数据，这个视图抛出一个Http404异常。

Django提供了一个快捷函数get_object_or_404()尝试获取一个对象，如果失败就抛出Http404异常。那么上面的代码就可以简化为：

```python
rom django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

## 5. 使用模板系统

前面的 detail() 视图中向模板传递了上下文变量 question 。下面是 polls/detail.html 模板里正式的代码：

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

查看 [模板指南]() 可以了解关于模板的更多信息。

## 6. 去除模板中的硬编码URL

我们在 polls/index.html 里编写投票链接时，链接是硬编码的：

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

但是这个强耦合的链接，对于一个包含很多应用的项目来说，修改起来十分困难。然而，因为你在 polls.urls 的 url() 函数中通过 name 参数为 URL 定义了名字，你可以使用 {% url %} 标签代替它：

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

如果你想改变投票详情视图的 URL，比如想改成 polls/specifics/12/ ，你不用在模板里修改任何东西（包括其它模板），只要在 polls/urls.py 里稍微修改一下就行：

```python
path('specifics/<int:question_id>/', views.detail, name='detail'),
```

## 7. 为URL名称添加命名空间

在一个真实的 Django 项目中，可能会有五个，十个，二十个，甚至更多应用。Django 如何分辨重名的 URL 呢？举个例子，polls 应用有 detail 视图，可能另一个博客应用也有同名的视图。Django 如何知道 {% url %} 标签到底对应哪一个应用的 URL 呢？

答案是：在根 URLconf 中添加命名空间。在 polls/urls.py 文件中稍作修改，加上 app_name 设置命名空间：

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

修改模板的详细视图：

```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

{% url 'polls:detail' question.id %}
{% url 'polls:index' question.id %}