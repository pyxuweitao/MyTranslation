# Django中编写和执行测试

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-03~2016-03-05
- 中文译名：Django中编写和执行测试
- 原标题和链接：[Writing and running tests](https://docs.djangoproject.com/en/1.9/topics/testing/overview/)
- 说明：`Django1.9`官方文档翻译
- 版权声明：未经本人许可，禁止转载发布


-------------------------------

>相关文档链接：

>- [测试教程](https://docs.djangoproject.com/en/1.9/intro/tutorial05/)

>- [测试工具参考](https://docs.djangoproject.com/en/1.9/topics/testing/tools/)

>- [进阶测试专题](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/)


这篇文档分为两个主要的章节。首先我们会介绍如何利用`Django`编写测试，然后再介绍怎样让测试跑起来。

##编写测试

`Django`的单元测试使用的是`Python`的标准库模块：[**`unittest`**](https://docs.python.org/3/library/unittest.html#module-unittest)。这个模块定义了一些基于类的方法实现的测试。

下面是一个继承了[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)的例子。[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)同时也是[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)的子类，[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)通过将每个测试在单独的事务中执行来实现孤立。

```python
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def setUp(self):
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")

    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')
```


当你开始执行测试时，测试工具会默认的从所有文件名以**`test`**打头的文件中寻找所有的测试用例（即所有[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)的子类），并自动根据那些测试用例创建一个测试套件，然后执行这个测试套件。

想要知道更多有关[`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest)的细节，请查看`Python`的文档。


---------------------------------

>**测试文件应该放在什么地方？**

>默认的[`starapp`](https://docs.djangoproject.com/en/1.9/ref/django-admin/#django-admin-startapp)命令模板会在新的应用中创建一个`tests.py`文件。当你只需要数量不多的测试时，直接写在这个文件里没有问题。但是随着你的测试越来越多时，你很可能想去把它们放到一个`test package`中，这样你就可以将你的测试分离成多个子模块，比如`test_models.py`，`test_views.py`，`test_forms.py`等等。任性选择任意你喜欢的模块组织结构吧。

>另请参阅[使用Django test runner去测试可重复利用的应用](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/#testing-reusable-applications)

------------------------------------

>**注意**：

>如果你的测试依赖于数据库，比如创建或者查询`models`，请确保你的测试类继承了[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)而不是[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)。

>使用[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)避免了每次在一个事务中执行一个测试的开销，并且也不会影响数据库，但是如果你的测试需要直接与数据库交互，测试的行为和结果将会因为测试被执行的顺序而变得不同。这将导致的结果是：当单独执行一个个测试单元时测试通过，但是当执行一个测试套件时测试失败。

----------------------------------

##执行测试

当你写好了测试后，利用你项目中的`manage.py`工具脚本使用[**`test`**](https://docs.djangoproject.com/en/1.9/ref/django-admin/#django-admin-test)命令：

```bash
$ ./manage.py test
```