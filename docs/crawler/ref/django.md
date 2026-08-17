# Web 前后端开发及 Django 简介

本次大作业需要使用 Python Web 后端搭建音乐信息网站。本文以 Django 和一个简易博客为例介绍基本开发流程；根据个人兴趣，也允许使用 Django 之外的 Python 后端框架。Django 是一个免费开源的高级 Python Web 框架，它鼓励快速开发和简洁、实用的设计。



#### 前后端交互

**前端**即是你打开网站能看到的东西。一般分为前端设计和前端开发，前端设计一般可以理解为网站的视觉设计，前端开发则是网站的前台代码实现，包括基本的 HTML 和 CSS 以及 JavaScript/ajax。

HTML：主要负责数据显示

CSS：主要是负责外观，主要有 bootstrap

JavaScript：主要实现动态效果。一般是直接用 js 框架，使用比较多的是 jquery



**后端**指的是运行在后台并且控制着前端的内容，它主要负责程序的逻辑部分的实现（一些简单的逻辑也可以在前端用js实现），管理数据库等。一般来说就是接受前端请求、处理数据、把数据返回前端。它涉及到的动态语言如 PHP、Java、Python 等。



#### 安装Django

Django的官方发行版可以直接从pip（Python包管理器）安装，只需要安装pip后在terminal中运行

```shell
$ pip install Django
```



#### 创建项目

安装Django后，我们可使用`django-admin startproject <项目名>` 来创建一个新项目，创建后项目的文件树是以下的结构：

```
.
├── manage.py  # 管理 Django 项目的命令行工具
└── <Project Name>
    ├── __init__.py  
    ├── settings.py  # Django 项目的配置文件
    ├── urls.py  # 主路由入口
    ├── asgi.py  # 以 asgi 方式进行部署的配置文件
    └── wsgi.py  # 以 wsgi 方式进行部署的配置文件
```



运行 `$ python manage.py runserver` 可以在本地的 8000 端口启动服务，此时通过浏览器访问 [http://127.0.0.1:8000/](http://127.0.0.1:8000/) 可以看到一个默认的祝贺网页。



#### 创建一个简易的博客页面

在项目目录下，可以用startapp命令创建一个博客应用

```
$ python manage.py startapp blog
```

**项目与应用**：应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库。项目则是一个网站使用的配置和应用的集合。这之后就可以看到

```
.
├── blog  # 你创建的应用
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations  # 应用对数据库表及表属性的修改历史
│   │   └── __init__.py
│   ├── models.py  # 应用的“模型”定义
│   ├── tests.py  # 应用的“单元测试”定义
│   └── views.py  # 应用的“视图”定义
├── manage.py
└── <Project Name>
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

这时，你一般需要在 `<project>/settings.py` 中的 `INSTALLED_APPS` 字段中注册该应用。


#### 模型（Model）

模型是用来描述数据的，包含了数据对应的字段和行为，一个模型一般对应数据库中的一张表。如果你对数据库的结构（主键、外键、索引）并不熟悉，可以查看这篇文章（[MySQL 主键、外键、索引 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/64368422)） 。

我们在`<app>/models.py`中加入如下的代码：

```python
from django.db import models

class Blog(models.Model):
    title = models.CharField(max_length=30)
    blog_content = models.CharField(max_length=2000)


class Comment(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)  # Blog 删除时级联删除 Comment
    user = models.CharField(max_length=10)
    comment_content = models.CharField(max_length=1000)
```

> 默认主键：数据库中的每个表都必须有一个unique的主键，如果我们没有在创建class时显式的定义主键，Django会创建一个名为`id`的默认主键，`id`从1开始按照数据的插入顺序递增。



修改完后，使用makemigrations去生成修改数据表结构和属性的语句

```
python manage.py makemigrations blog
```

之后每次在server部署之前，使用migrate命令将makemigrations生成的语句应用到数据库的表中

```
python manage.py migrate
```



##### 使用shell往表中加入数据

Django默认使用SQLite作为默认数据库，在执行了上面的命令中后实际已经在本地创建了一个db.sqlite3文件，其中有两个空表Blog和Comment。我们可以使用Django自带的shell工具往表填入数据，我们现在加入第一条博客：

```shell
$ python manage.py shell
In [1]: from blog.models import Blog

In [2]: Blog.objects.all()
Out[2]: <QuerySet []>

In [3]: blog = Blog(title="title", blog_content="content")

In [4]: blog.save()

In [5]: blog.id
Out[5]: 1

In [6]: Blog.objects.all()
Out[6]: <QuerySet [<Blog: Blog object (1)>]>

In [7]: exit
```



#### 视图（View）

视图函数是后端逻辑的主入口，其接受经过路由之后的 [`HttpRequest`](https://docs.djangoproject.com/zh-hans/stable/ref/request-response/#django.http.HttpRequest) 类型的请求作为参数，并返回一个 [`HttpResponse`](https://docs.djangoproject.com/zh-hans/stable/ref/request-response/#django.http.HttpResponse) 类型的对象作为响应。你可以在上述链接中查找这两个类分别有哪些成员变量可以供你使用。

在 `blog/views.py` 中：

```python
from .models import Blog, Comment
from django.template import loader
from django.http import HttpResponse, HttpResponseRedirect

def show_blog(request, id): # 这里 request 是 HttpRequest 类型的对象
    blog = Blog.objects.get(id=id) # 数据库查询操作
    template = loader.get_template('blog/index.html')
    context = {
        'blog_id': id,
        'blog_title': blog.title,
        'blog_content': blog.blog_content,
        'comments': blog.comment_set.all() # 通过外键反向查询
    }
    return HttpResponse(template.render(context, request)) # 返回渲染好的html

def comment(request, id): 
    data = request.POST
    # 将新的消息添加到数据库中
    user = data['user']
    comment_content = data['content']
    blog = Blog.objects.get(id=id)
    obj = Comment(blog=blog, user=user, comment_content=comment_content)
    obj.full_clean() #对数据进行验证
    obj.save() #存储在表中
    return HttpResponseRedirect(f'/index/blog/{id}') # 将页面重定向到博客的url
```



#### HTML模板

我们使用 Django 的模板系统，只要创建一个视图，就可以将页面的设计从代码中分离出来。

首先，在你的 `blog` 目录里创建一个 `templates` 目录。Django 将会默认在这个目录里查找模板文件。

在你刚刚创建的 `templates` 目录里，再创建一个目录 `blog`，然后在其中新建一个文件 `index.html` 。换句话说，你的模板文件的路径应该是 `blog/templates/blog/index.html` 。



> **模板命名空间**
>
> 虽然我们现在可以将模板文件直接放在 `blog/templates` 文件夹中（而不是再建立一个 `blog` 子文件夹），但是这样做不太好。Django 将会选择第一个匹配的模板文件，如果你有一个模板文件正好和另一个应用中的某个模板文件重名，Django 没有办法区分它们。我们需要帮助 Django 选择正确的模板，最好的方法就是把他们放入各自的 *命名空间* 中，也就是把这些模板放入一个和自身应用重名的子文件夹里



```html
<h1><a href="/index/blog/1"> {{ blog_title }} </a></h1>
<p>
    {{ blog_content }}
</p>
{% if comments %}
    <ul>
    {% for comment in comments %}
        <li>{{ comment.user }} : {{ comment.comment_content }}</li>
    {% endfor %}
    </ul>
{% else %}
    <p>No comment.</p>
{% endif %}
<!-- 表单，用于提交评论 -->
<form action="/index/comment/{{ blog_id }}" method="post">
    {% csrf_token %}
    <!-- name 对应 request 中的键值（key） -->
    user:<input type="text" name="user">
    comment:<input type="text" name="content">
    <input type="submit" value="comment">
</form>
```


#### 路由（Routing）

首先，我们来解决后端收到请求时，后端会将请求交给哪个应用的哪个视图函数处理的问题。和这个功能有关的文件主要包括 `<Project Name>/urls.py` 和 `<app>/urls.py`。

假如我们的后端部署在 `http://localhost:6011/`，我们在访问 `http://localhost:6011/index/blog/1` 时，后端会首先在 `<Project Name>/urls.py` 中以 `index/blog/1` 开始搜索。假设 `<Project Name>/urls.py` 中配置为：

```python
from django.urls import path, include

urlpatterns = [
    path('index/', include("blog.urls")),
]
```



我们会匹配掉字符串 `'index/'`，然后将剩余路径交给 `blog/urls.py` 处理，这也是这里 `include` 的作用，将请求转发给子应用的路由表处理。

然后，假设我们在 `blog/urls.py` 中配置为：

```python
from django.urls import path, include
import blog.views as views

urlpatterns = [
    path('blog/<int:id>', views.show_blog),
    path('comment/<int:id>', views.comment)
]
```

这时剩余请求 `blog/1` 匹配到第一条规则后，交由 `blog/views.py` 中的 `show_blog(request=request, id=1)` 函数进行处理，即后端会帮助我们调用这个函数，并把请求体（和请求有关的信息，包括请求方法、请求数据等等）作为参数传给这个函数。

之后就可以访问http://127.0.0.1:8000/index/blog/1来查看第一篇博客和提交评论了！

此外还有更多和路由有关的功能请阅读[官方文档](https://docs.djangoproject.com/zh-hans/stable/topics/http/urls/)。


---

示例代码地址：[python-course / django-example · GitLab (tsinghua.edu.cn)](https://git.tsinghua.edu.cn/python-course/django-example)

**参考资料**

+ [Django 文档](https://docs.djangoproject.com/zh-hans/stable/)
