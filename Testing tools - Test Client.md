# Django中的测试工具

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-20~2016-03-30
- 中文译名：Django中的测试工具 之 测试客户端
- 原标题和链接：[Testing tools](https://docs.djangoproject.com/en/1.9/topics/testing/tools/)
- 说明：`Django1.9`官方文档翻译，为保证翻译后语义通顺便于理解，在不影响把握原文所表中心的前提下，部分原文语句有省略、补充。翻译本文纯属兴趣爱好，不敢保证与原文所表100%一致，但必字句斟酌。能力有限，如有异译或觉不妥，欢迎交流。误导必致歉，查错必纠正。
- 版权声明：未经本人许可，禁止转载发布

-------------------------------

当你编写测试时，`Django`提供一系列“开箱即用”的工具来供你调用。

##测试客户端

测试客户端是一个模拟浏览器的`Python`类，它可以辅助你测试你的视图函数，并且可以和你的`Django`应用进行交互。

通过测试客户端，你可以做到以下这些事：

- 模拟某个`URL`的`GET`和`POST`请求，然后来观察响应，响应中包括从底层`HTTP`(响应头和状态码)到页面内容的所有内容。
- 浏览重定向链(如果存在的话)来核对`URL`和重定向链每一步的状态码。
- 测试一个给定的由`Django`模板来渲染的，包含模板上下文和某些变量值的请求。

测试客户端并不打算成为[**Selenium**](http://seleniumhq.org/)或者其他浏览器模拟框架的替代品。`Django`的测试客户端与这些浏览器模拟框架有不一样的侧重点。简短来说：

- `Django`测试客户端侧重来测试被渲染的模板是否正确以及模板是否被传递了正确的模板上下文数据。
- 类似[**Selenium**](http://seleniumhq.org/)的浏览器模拟框架侧重于测试已经被渲染的`HTML`和网页的一些行为（也就是`JavaScript`的一些功能函数）。`Django`也为这些框架提供了专门的支持，更多详细信息可以查看[`LiveServerTestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.LiveServerTestCase)章节。

一个全面的测试套件应当综合使用这两种测试类型。

###概述和一个简单的小例子

如果要使用测试客户端，实例化`django.test.Client`来获取网页：

```python
>>> from django.test import Client
>>> c = Client()
>>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
>>> response.status_code
200
>>> response = c.get('/customer/details/')
>>> response.content
b'<!DOCTYPE html...'
```

正如这个例子展示的，你可以从`Python`的交互式解释器中来实例化一个`Client`。


那么测试客户端如何工作呢，了解这些很重要：

- 测试客户端并不需要事先让`Web`服务器跑起来。事实上，它完全脱离于`Web`服务器执行并且执行的很好。这是因为它避免了`HTTP`的开销直接与`Django`框架交互。这让单元测试执行的很快。

- 当请求网页的时候，只需要指出网页的相对`URL`路径而不需要整个域名。例如，这样是正确的：

```python
>>> c.get('/login/')
```

 不正确的：

```python
>>> c.get('https://www.example.com/login/')
```

 测试客户端不能直接访问你的`Django`项目之外的网页。如果你需要访问项目之外的网页，使用`Python`标准库，比如[`urllib`](https://docs.python.org/3/library/urllib.html#module-urllib)。

- 测试客户端根据配置文件中[`ROOT_URLCONF`](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-ROOT_URLCONF)指定的`URLconf`来解析`URL`。 
- 尽管上边的例子能够在`Python`交互解释器中执行，但是有一些测试客户端的功能函数，尤其是模板相关的功能函数，只有在测试运行的时候是可以调用执行。

 这样的原因是`Django`的测试`runner`执行了一点黑魔法来决定给定的视图会加载哪个模板。而这个黑魔法（其实就是内存中`Django`模板系统）只在测试`runner`跑的时候被执行。

- 默认的测试客户端将关闭所有对你站点上的`CSRF`的检查。
  
  如果你想要你的测试客户端去执行`CSRF`检查，你可以在实例化测试客户端的时候传递`enforce_csrf_checks`参数：
  
```python
>>> from django.test import Client
>>> csrf_client = Client(enforce_csrf_checks=True)
```   



-----------------------------------------------

###发起请求

使用`django.test.Client`类来发起请求。

`class Client(enforce_csrf_checks=False, **defaults)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client)

实例化的时候这个类不需要任何参数，但是你可以通过传递参数来指定一些默认的报文头，比如，下面这样写可以在每个`HTTP`请求头上加上一个`User-Agent`：

```python
>>> c = Client(HTTP_USER_AGENT='Mozilla/5.0')
```

有一些在`Client`的[`get()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.get)或者[`post()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.post)方法中传递的参数优先于(会覆盖掉)在实例化`Client`时传递的相同概念的参数。`enforce_csrf_checks`参数可以被用于测试`CSRF`保护（见上）。

一旦你实例化了`Client`，你就可以调用以下的方法：

####`get(path, data=None, follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.get)
 
 发起一个指定**path**的请求，然后返回一个**Response**对象。响应对象在下方会有详细文档。
 
 **data**参数传递的键值对可以被用来创建一个有数据的`GET`请求。比如：
 
```python
>>> c = Client()
>>> c.get('/customers/details/', {'name': 'fred', 'age': 7})
```

这样发起的`GET`请求就等价于

```plain
/customers/details/?name=fred&age=7
```

**extra**参数可以被用于指定请求头的信息。比如：

```python
>>> c = Client()
>>> c.get('/customers/details/', {'name': 'fred', 'age': 7},
...       HTTP_X_REQUESTED_WITH='XMLHttpRequest')
```

这样将发送一个带有`HTTP_X_REQUESTED_WITH`请求头的请求，这对于使用`django.http.HttpRequest.is_ajax()`方法的测试来说有好处。

>**CGI标准**

>通过****extra**传递的报文头参数应该遵循[`CGI`](https://www.w3.org/CGI/)的标准。例如，模拟一个从浏览器发送到服务器，但是`Host`报文头不同的`HTTP`请求，应该传递的参数的名称是`HTTP_HOST`。

如果你已经把`GET`参数写到了`URL`里，那你就不需要再传入一遍这些参数了。例如，上边的`GET`请求也可以写成：

```python
>>> c = Client()
>>> c.get('/customers/details/?name=fred&age=7')
```

如果你把参数写到了`URL`里，同时也在`get`方法中传入了参数，那么传入的参数将覆盖掉`URL`中的参数。

如果你将**follow**设置为**True**，测试客户端将追踪重定向，返回的响应对象的**redirect_chain**属性是一个元组，包含所有中间`URL`以及对应的状态码。

比如，如果你发起请求访问**/redirect_me/**，这个链接重定向到**/next/**，再重定向到**/final/**，这个时候你就会看到：

```python
>>> response = c.get('/redirect_me/', follow=True)
>>> response.redirect_chain
[('http://testserver/next/', 302), ('http://testserver/final/', 302)]
```

如果你将**secure**设置为**True**，测试客户端会模拟**HTTPS**的请求。


####`post(path, data=None, content_type=MULTIPART_CONTENT, follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.post)

发起一个指定**path**的`POST`请求，返回一个**Response**对象，有关该对象下边会详细说明。

参数**data**是`POST`请求提交的数据，一般为键值对。例如：


```python
>>> c = Client()
>>> c.post('/login/', {'name': 'fred', 'passwd': 'secret'})
```

这样发起的`POST`请求就等价于

```plain
/login/
```

其中包含的`POST`数据为：

```plian
name=fred&passwd=secret
```

如果你在调用`post`方法时指定了**content_type**参数（比如`XML`对应的`text/xml`），那么你的发出去的`POST`请求报文就包含了数据和你指定的**content_type**请求头。

如果你没有指定**content_type**参数，那么**content_type**将默认为`multipart/form-data`。在上边的例子里，**data**中的数据将被按照`multipart/form`的格式编码。

有时需要提交一个单键多值的字典——比如指定`<select multiple>`的选项时，一个键会有多个值与之对应。举个栗子，**choice**键有三个值组成一个元组作为选项：

```python
{'choices': ('a', 'b', 'd')}
```

提交文件是一种特殊的情况。需要**POST**一个文件时，你只需要用一个文件字段名作为键和一个指向你想上传的文件的指针作为对应的值。例如：

```python
>>> c = Client()
>>> with open('wishlist.doc') as fp:
...     c.post('/customers/wishes/', {'name': 'fred', 'attachment': fp})
```

（注意这里的键名**attachment**不是固定的，任意键名都是可以的。）

与`get()`方法类似，你同样也可以通过提供一个文件指针来上传类文件对象（比如[**StringIO**](https://docs.python.org/3/library/io.html#io.StringIO)）或者[**BytesIO**](https://docs.python.org/3/library/io.html#io.BytesIO)。

>**Django 1.8 新特性**

>使用类文件对象的功能已经添加。


注意，假如你打算在多个`post()`方法中使用同一个文件指针，需要手动的在每次**post**之前手动重置这个文件指针。最简单的办法就是用上边代码展示的那样，在每次把文件指针传给`post()`之后，手动关闭掉这个文件流。


你应当确保你的文件以一种能够被正常读出的方式打开。比如文件是一张图片，其中包含了二进制数据，这意味着你需要以**`rb`**(`read binary`)方式打开这个文件。


其他参数的指定方法都类似[**`Client.get()`**](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.get)。

如果你发起一个`URL`的**POST**请求，其中包含一些已经编码好的参数，这些参数将能够在`request.GET`字典中获取到。举个栗子，如果你发起了这样一个请求：

```python
>>> c.post('/login/?visitor=true', {'name': 'fred', 'passwd': 'secret'})
```

视图处理请求的时候能够根据`request.POST`获取用户名和密码，也能根据`request.GET`来确定当前用户是否是一个`visitor`。

和`get()`方法一样，如果你将**follow**设置为**True**，测试客户端将追踪重定向，返回的响应对象的**redirect_chain**属性是一个元组，包含所有中间`URL`以及对应的状态码。

如果你将**secure**设置为**True**，测试客户端会模拟**HTTPS**的请求。

####`head(path, data=None, follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.head)

发起对**path**访问的`HEAD`请求然后返回一个**Response**对象。这个函数调用方法和[`Client.get()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.get)基本上一致，也包含了**follow**,**secure**,**extra**等参数，但是`HEAD`请求并不会给你返回一个消息体。

####`options(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.options)

发起对**path**访问的`OPTIONS`请求然后返回一个**Response**对象。在**RESTful**接口中很常见的请求类型。

**data**参数的值会作为请求体发送，请求头中的**Content-Type**由参数**content_type**决定。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。

####`put(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.put)

发起对**path**访问的`PUT`请求然后返回一个**Response**对象。在**RESTful**接口中很常见的请求类型。

**data**参数的值会作为请求体发送，请求头中的**Content-Type**由参数**content_type**决定。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。

####`patch(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.patch)

发起对**path**访问的`PATCH`请求然后返回一个**Response**对象。在**RESTful**接口中很常见的请求类型。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。

####`delete(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.delete)

发起对**path**访问的`DELETE`请求然后返回一个**Response**对象。在**RESTful**接口中很常见的请求类型。

**data**参数的值会作为请求体发送，请求头中的**Content-Type**由参数**content_type**决定。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。

####`trace(path, follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.trace)

>**Django 1.8的新特性**

发起对**path**访问的`TRACE`请求然后返回一个**Response**对象。这个请求通常应用于模拟诊断探测。

区别于其他的请求类型，根据[`RFC 2616`](https://tools.ietf.org/html/rfc2616.html)的约定，**TRACE**请求不应该包含实体，因此`trace()`方法并不提供**data**参数。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。

####`login(**credentials)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.login)

如果你的网站使用了`Django`的[认证系统](https://docs.djangoproject.com/en/1.9/topics/auth/)来处理用户登录，你可以使用测试客户端的**`login()`**方法去模拟用户登录的效果。

由于这个方法和使用了[**`AuthenticationForm`**](https://docs.djangoproject.com/en/1.9/topics/auth/default/#django.contrib.auth.forms.AuthenticationForm)的[**`login()`**](https://docs.djangoproject.com/en/1.9/topics/auth/default/#django.contrib.auth.login)视图是等效的，因此没被激活的用户（[`is_active=False`](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User.is_active)）默认是没有权限登录的。

在你调用了这个方法成功登录之后，测试客户端将利用必需的`cookies`和`session`的数据去通过那些基于用户登录信息的测试。

**credentials**参数的形式取决于你在配置文件中[**`AUTHENTICATION_BACKENDS`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-AUTHENTICATION_BACKENDS)配置好的[认证后端](https://docs.djangoproject.com/en/1.9/topics/auth/customizing/#authentication-backends)。如果你正在使用`Django`标准的认证后端（**ModelBackend**），**credentials**应当由用户的用户名和密码关键字组成。

```python
>>> c = Client()
>>> c.login(username='fred', password='secret')

# 现在你就可以访问已登录用户才能访问到的视图.
```

如果你正在使用其他的认证后端，这个方法可能需要不同的**credentials**。它需要的参数和你使用的后端的`authenticate()`方法是一致的。

如果`credentials`参数被接收而且认证通过，`login()`返回`True`。

最后要提醒的是，你要确保在你调用这个方法之前已经创建了对应的用户。正如之前介绍的，测试程序执行的时候使用的是默认没有用户数据的测试用数据库。因此，生产站点上合法的用户在测试的时候是无法使用的。你需要在测试套件中先创建新用户——手动使用`Django`的`model`的`API`或者使用其他的测试工具。另外注意一点，如果你像让你的测试用户拥有一个密码，你不能通过密码属性值来设置用户密码，而是应该使用[`set_password()`](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User.set_password)方法去存储一个哈希加密过的正确过的密码。当然你也可以使用[`create_user`](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.UserManager.create_user)方法去创建一个新用户。


####`force_login(user, backend=None)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.force_login)

>**Django 1.9 新特性**

如果你的站点正在使用`Django`的[认证系统](https://docs.djangoproject.com/en/1.9/topics/auth/)，你可以使用`force_login()`方法去模拟用户登录的效果。当用户登录的详细信息不重要时，可以考虑使用**`force_login()`**而不是 [ `login()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.login)。

不同于`login()`的是，`force_login()`方法跳过了认证的过程：没被激活的用户([`is_active=False`](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User.is_active))也允许登录，而且用户的认证信息(`credentials`参数)也不需要提供。

用户将拥有**backend**属性，由传入的参数`backend`决定（应当是一个`Python`的点路径字符串），或者当这个参数没有被传入时，由**settings.AUTHENTICATION_BACKENDS[0]**决定。由 [ `login()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.login)调用的[`authenticate()`](https://docs.djangoproject.com/en/1.9/topics/auth/default/#django.contrib.auth.authenticate)函数通常通过这个来标识用户。

####`logout()`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.logout)

如果你的站点正在使用`Django`的[认证系统](https://docs.djangoproject.com/en/1.9/topics/auth/)，你可以使用`logout()`方法去模拟用户注销的效果。

在你调用了这个方法之后，测试客户端会将所有的`cookies`和`session`数据还原到默认状态。随后的请求将来自一个[匿名用户](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.AnonymousUser)。

-----------------

###测试响应

`get()`和`post()`方法都会返回**Response**对象。这个**Response**对象和`Django`视图所需要返回的**HttpResponse**不一样，测试所返回的**Response**对象多了一些对测试代码实现的功能很有用处的数据。

一个**Response**对象含有的属性包括：

####`class Response`

#####**client**

用于发起请求并产生这个响应对象的测试客户端。

#####**content**

响应的实体，是一个字节串。要么是视图渲染之后形成的最终的页面内容，要么是一些错误信息。

#####**context**

模板的上下文实例，用来渲染模板最终形成响应对象的**content**属性。

如果被渲染的网页使用了多个模板，那么**context**属性将会是一个按照渲染顺序排序的**Context**对象列表。

不论在渲染的过程中使用了多少个模板，你可以使用操作符`[]`来检索上下文的值。举个栗子，上下文变量**name**可以通过下边的方法检索：
  
```python
>>> response = client.get('/foo/')
>>> response.context['name']
'Arthur'
```

>**没在使用`Django`原生的模板？**

>**context**这个属性只有在使用[`DjangoTemplates`](https://docs.djangoproject.com/en/1.9/topics/templates/#django.template.backends.django.DjangoTemplates)时是有效的，当你使用其他模板引擎时，对应属性名是[**context_data**](https://docs.djangoproject.com/en/1.9/ref/template-response/#django.template.response.SimpleTemplateResponse.context_data)而不是**context**。

#####`json(**kwargs)`

>**Django 1.9 新特性**
 
将响应体按照`JSON`格式解析，该方法调用时的其他参数可以参考[`json.loads()`](https://docs.python.org/3/library/json.html#json.loads)方法，两者是一样的。但是请注意，如果响应头中**Content-Type**并不是**"application/json"**，那么在尝试调用这个方法时会抛出[**ValueError**](https://docs.python.org/3/library/exceptions.html#ValueError)异常。最后，举个调用的例子：
 
```python
>>> response = client.get('/foo/')
>>> response.json()['name']
'Arthur'
```


#####**request**

响应对应的请求数据。

#####**wsgi_request**

由生成响应的测试处理器生成的**WSGIRequest**实例。

#####**status_code**

响应的**HTTP状态**，值为一个整数。查看[**RFC 2616 第十章节**](https://tools.ietf.org/html/rfc2616.html#section-10)来获取**HTTP**状态码详细列表。

#####**templates**

由用来渲染最终**content**属性内容的模板实例组成的列表，列表中实例的顺序即模板渲染顺序。对于列表中的每一个模板，如果是从文件中加载，可以使用`template.name`去获取模板文件的名称（名称是一个字符串，比如`'admin/index.html'`）。

>**没在使用`Django`原生的模板？**

>**templates.name**这个属性只有在使用[`DjangoTemplates`](https://docs.djangoproject.com/en/1.9/topics/templates/#django.template.backends.django.DjangoTemplates)时是有效的，当你使用其他模板引擎时，如果你只需要模板的名称，可以直接用[**template_name**](https://docs.djangoproject.com/en/1.9/ref/template-response/#django.template.response.SimpleTemplateResponse.template_name)。

#####**resolver_match**

>**Django 1.8 新特性**

>响应对象的一个[**ResolverMatch**](https://docs.djangoproject.com/en/1.9/ref/urlresolvers/#django.core.urlresolvers.ResolverMatch)实例。你可以使用这个实例的[**func**](https://docs.djangoproject.com/en/1.9/ref/urlresolvers/#django.core.urlresolvers.ResolverMatch.func)属性去回溯这个响应对应的视图。例如：

```python
# 这里的 my_view 是一个视图函数
self.assertEqual(response.resolver_match.func, my_view)
# 基于类的视图判定相同需要比较函数名称(__name__)
# 直接拿as_view() 生成的函数和func比较不会相等。
self.assertEqual(response.resolver_match.func.__name__, MyView.as_view().__name__)
```

>如果对应的`URL`不存在，直接获取这个属性的值的话将抛出[**Resolver404**](https://docs.djangoproject.com/en/1.9/ref/exceptions/#django.core.urlresolvers.Resolver404)异常。


你也可以使用字典的相关语法去获取或者设置**HTTP**响应头中的相关信息的值。比如你可以通过使用**`response['Content-Type']`**去决定响应内容的类型。

--------------------

###异常

测试客户端访问某个视图时抛出异常，这些异常都是可以捕获到的。你可以使用标准的`try...except`代码块或者[`assertRaises()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaises)去测试异常。

但是不是所有的异常都是可以捕获的，像[`Http404`](https://docs.djangoproject.com/en/1.9/topics/http/views/#django.http.Http404)、[`PermissionDenied`](https://docs.djangoproject.com/en/1.9/ref/exceptions/#django.core.exceptions.PermissionDenied)、[`SystemExit`](https://docs.python.org/3/library/exceptions.html#SystemExit)和[`SuspiciousOperation`](https://docs.djangoproject.com/en/1.9/ref/exceptions/#django.core.exceptions.SuspiciousOperation)都是无法捕获的。`Django`在内部捕获这些异常并直接将其转换为对应的**HTTP**状态码返回。你可以在测试里通过检查`response.status_code`来判断异常。

-----------------------

###持久化状态

测试客户端是有状态的。如果一个响应返回了`cookies`，那么`cookies`将被存储到测试客户端然后在之后的`get()`或`post()`请求中发送出去。

这些`cookies`并不遵循超时策略。如果你想让一个`cookie`超时，要么手动删除它，要么创建一个新的**Client**实例（迅速删掉了所有的`cookies`）。

一个测试客户端有两个用来保存持久化状态信息的属性。你可以在测试时访问这些属性的内容。

####`Client.cookies`

一个`Python`的[`SimpleCookie`](https://docs.python.org/3/library/http.cookies.html#http.cookies.SimpleCookie)对象，包含了测试客户端中所有当前`cookies`的值。查看[`http.cookies`](https://docs.python.org/3/library/http.cookies.html#module-http.cookies)模块的文档获取更多详细信息。

####`Client.session`

一个包含会话信息的类似字典的对象。查看[会话文档](https://docs.djangoproject.com/en/1.9/topics/http/sessions/)获取详细信息。

如果你想要修改会话信息并且保存，必须首先把`Client.session`保存在一个变量里（因为每一次访问这个属性的时候都会有一个新的**SessionStore**被创建）。

```python
def test_something(self):
    session = self.client.session
    session['somekey'] = 'test'
    session.save()
```

--------------------

###例子

下边是一个使用测试客户端的简单的单元测试：

```python
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def setUp(self):
        # Every test needs a client.
        self.client = Client()

    def test_details(self):
        # Issue a GET request.
        response = self.client.get('/customer/details/')

        # Check that the response is 200 OK.
        self.assertEqual(response.status_code, 200)

        # Check that the rendered context contains 5 customers.
        self.assertEqual(len(response.context['customers']), 5)
```

>也可以看看

>[`django.test.RequestFactory`](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/#django.test.RequestFactory)

