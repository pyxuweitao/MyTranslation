# Django中的测试工具

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-10~2016-03-09
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

##概述和一个简单的小例子

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

##发起请求

使用`django.test.Client`类来发起请求。

`class Client(enforce_csrf_checks=False, **defaults)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client)

实例化的时候这个类不需要任何参数，但是你可以通过传递参数来指定一些默认的报文头，比如，下面这样写可以在每个`HTTP`请求头上加上一个`User-Agent`：

```python
>>> c = Client(HTTP_USER_AGENT='Mozilla/5.0')
```

有一些在`Client`的[`get()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.get)或者[`post()`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.Client.post)方法中传递的参数优先于在实例化`Client`时传递的参数。`enforce_csrf_checks`参数可以被用于测试`CSRF`保护（见上）。

一旦你实例化了`Client`，你就可以调用以下的方法：

- `get(path, data=None, follow=False, secure=False, **extra)`[[source]](https://docs.djangoproject.com/en/1.9/_modules/django/test/client/#Client.get)
 
 发起一个指定**path**的请求，然后返回一个**Response**对象。响应对象在下方会有详细文档。
 
 **data**参数传递的键值对可以被用来创建一个有数据的`GET`请求。比如：
 
```python
>>> c = Client()
>>> c.get('/customers/details/', {'name': 'fred', 'age': 7})
```

这样发起的`GET`请求就等价于

```javascript
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
