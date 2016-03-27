# Django中的测试工具

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-20~2016-03-21
- 中文译名：Django中的测试工具
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

有时需要提交一个单键多值的字典——比如指定**<select multiple>**的选项时，一个键会有多个值与之对应。举个栗子，**choice**键有三个值组成一个元组作为选项：

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

发起对**path**访问的`OPTIONS`请求然后返回一个**Response**对象。在**RESTful**接口中很常见的请求。

**data**参数的值会作为请求体发送，请求头中的**Content-Type**由参数**content_type**决定。

**follow**,**secure**,**extra**等参数的作用方式与**`Client.get()`**相同。
