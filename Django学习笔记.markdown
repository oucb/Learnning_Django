# Django使用

标签（空格分隔）： web

---
###**初始化**
**1.`python -c "import django; print(django.get_version())"`**
检查django版本
**2.进入相应目录执行：`django-admin startproject mysite`**
创建如下目录与文件：
```
mysite/
    manage.py
    mysite/
        __init__.py:这是一个空的脚本,用来告诉Python编译器这个目录是一个Python包.
        settings.py:用来存储Django项目设置的文件.
        urls.py:用来存储项目里的URL模式.
        wsgi.py:用来帮助你运行开发服务,同时可以帮助部署你的生产环境.
```
creating a startproject of django named mysite.This will create a mysite directory in your current directory,you can find a mysite directory and a manage.py file in the directory.
**3.进入manage.py所在目录执行：`python manage.py runserver`**
You’ve started the Django development server, a lightweight Web server written purely in Python. 
**4.进入manage.py所在目录执行`python manage.py startapp polls`**
创建如下目录与文件
```
polls/
    __init__.py
    admin.py:在这里你可以向Django注册你的模型,它会为你创建Django的管理界面.
    apps.py
    migrations/
        __init__.py
    models.py:一个存储你的应用中数据模型的地方 - 在这里描述数据的实体和关系.
    tests.py
    views.py:在这里处理用户请求和响应.
```
creating a directory polls，a django app。
**5.`python manage.py migrate`**
The migrate command looks at the INSTALLED_APPS setting and creates any necessary database tables according to the database settings in your mysite/settings.py file and the database migrations shipped with the app

###**models**
A model is the single, definitive source of truth about your data. It contains the essential fields and behaviors of the data you’re storing.
> 1.修改models.py文件后，在settings.py文件INSTALL_APPS中添加应用设置。
2.`python manage.py makemigrations polls`
By running makemigrations, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a migration.
`python manage.py sqlmigrate polls 0001`获取名称为0001的migration并返回其sql内容
3.`python manage.py migrate`
create model tables in your database
```
Change your models (in models.py).
Run python manage.py makemigrations to create migrations for those changes
Run python manage.py migrate to apply those changes to the database.
```
**It's important to add __str__() methods to your models, not only for your own convenience when dealing with the interactive prompt, but also because objects' representations are used throughout Django's automatically-generated admin.**
> `question_text__startswith`
`pub_date__year`
**注意text、date后面的是"double underscore"线，谨记**

###**Admin**
**1.`python manage.py createsuperuser`**
创建管理员账户
**2.Make the poll app modifiable in the admin**
在应用polls下的admin.py中增加如下代码
```python
from .models import Question
admin.site.register(Question)
```

###**views&urls**
**1.`url(r'^polls/', include('polls.urls'))`**
工程url匹配文件urls.py中包含应用polls的url匹配文件，**注意polls后面不需要`$`来匹配任意字符，只需要`/`**
**2.捕获内容传递给参数使用**
The question_id='34' part comes from `(?P<question_id>[0-9]+)`. Using `parentheses` around a pattern “captures” the text matched by that pattern and sends it as an argument to the view function; `?P<question_id>` defines the name that will be used to identify the matched pattern; and [0-9]+ is a regular expression to match a sequence of digits
**3.`A shortcut: render()`**
The render() function takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument. It returns an HttpResponse object of the given template rendered with the given context.
**4.A shortcut: `get_object_or_404()`**
The `get_object_or_404()` function takes a Django model as its first argument and an arbitrary number of keyword arguments, which it passes to the get() function of the model’s manager. It raises Http404 if the object doesn’t exist.

###**提交form**
**1.Whenever you create a form that alters data server-side, use method="post". This tip  isn’t specific to Django; it’s just good Web development practice.**

**2.`request.POST`**

* `request.POST` is a dictionary-like object that lets you access submitted data by key name. In this case, request.POST['choice'] returns the ID of the selected choice, as a string. request.POST values are always strings.
Note that Django also provides request.GET for accessing GET data in the same way – but we’re explicitly using request.POST in our code, to ensure that data is only altered via a POST call.
* `request.POST['choice']` will raise KeyError if choice wasn't provided in POST data. The above code checks for KeyError and redisplays the question form with an error message if choice isn’t given.
* After incrementing the choice count, the code returns an `HttpResponseRedirect` rather than a normal HttpResponse. HttpResponseRedirect takes a single argument: the URL to which the user will be redirected (see the following point for how we construct the URL in this case).
`As the Python comment above points out, you should always return an HttpResponseRedirect after` `successfully dealing with POST data. This tip isn’t specific to Django; it’s just good Web development` `practice.`
* We are using the `reverse()` function in the `HttpResponseRedirect` constructor in this example. This function helps `avoid having to hardcode a URL in the view function`. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view. In this case, using the URLconf we set up in Tutorial 3, this reverse() call will return a string like
```
'/polls/3/results/'
# where the 3 is the value of question.id. This redirected URL will then call the 'results' view to display the final page.
```
**3.race condition**
If you are interested, you can read Avoiding race conditions using F() to learn how you can solve this issue.

###**generic views**
1.The DetailView generic view expects the primary key value captured from the URL to be called "pk"
```
url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
改为：
url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
```
**注意：上述更改中将`question_id`改为`pk`**

###**Customize the admin form**
**1.控制数据模型显示的内容项及顺序**
create a model admin class, then pass it as the second argument to admin.site.register() – any time you need to change the admin options for an model.
```python
class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']
admin.site.register(Question, QuestionAdmin)
```
列表fields中的项是要显示的内容，将依照列表中位置的前后顺序依次显示

**2.设置要显示的内容项为分栏显示并添加分栏标题**
The first element of each tuple in fieldsets is the title of the fieldset。
```
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]
```
**3.添加关联对象**
`Django knows that a ForeignKey should be represented in the admin as a <select> box`
```python
class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3
# with the TabularInline, the related objects are displayed in a more compact, table-based format；即第一列显示关联对象，后面每列显示对象的相应属性内容。 the StackedInline,垂直展开显示关联对象的属性内容。

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
        # 'classed': ['collapse']提供显示隐藏相应栏的按钮
    ]
    inlines = [ChoiceInline]
    # This tells Django：Choice objects are edited on the Question admin page. By default, provide enough fields for 3 choices.
```

**4.list_display**
the list_display admin option, which is a tuple of field names to display, as columns, on the change list page for the object:
```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
    #You can click on the column headers to sort by those values
```
模型对象列表页显示对象的其它属性栏

**5.list_filter**
```python
list_filter = ['pub_date']
```
That adds a “Filter” sidebar that lets people filter the change list by the pub_date field

**6.search_fields**
```python
search_fields = ['question_text']
```

###**从浏览器获取输入**
1.通过在url正则表达式中使用括号来捕获url地址中的内容，括号里的内容将被捕获传递给相关函数使用，或是直接在正则表达式通过`(?P<name>content)`将conten内容传递给指定变量name

2.通过表单的request.POST['x']或是request.GET['x']来获取name为’x‘的input输入字符




