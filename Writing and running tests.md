# Django中编写和执行测试

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-02~2016-03-09
- 中文译名：Django中编写和执行测试
- 原标题和链接：[Writing and running tests](https://docs.djangoproject.com/en/1.9/topics/testing/overview/)
- 说明：`Django1.9`官方文档翻译，为保证翻译后语义通顺便于理解，在不影响把握原文所表中心的前提下，部分原文语句有省略、补充。翻译本文纯属兴趣爱好，不敢保证与原文所表100%一致，但必字句斟酌。能力有限，如有异译或觉不妥，欢迎交流。误导必致歉，查错必纠正。
- 版权声明：未经本人许可，禁止转载发布

-------------------------------

>相关文档链接：

>- [测试教程](https://docs.djangoproject.com/en/1.9/intro/tutorial05/)

>- [测试工具参考](https://docs.djangoproject.com/en/1.9/topics/testing/tools/)

>- [进阶测试专题](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/)


这篇文档分为两个主要的章节。首先我们会介绍如何利用`Django`编写测试，然后再介绍怎样让测试跑起来。

##编写测试

`Django`的单元测试使用的是`Python`的标准库模块：[**`unittest`**](https://docs.python.org/3/library/unittest.html#module-unittest)。这个模块定义了一些基于类的方法实现的测试。

下面是一个继承了[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)的例子。[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)同时也是[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)的子类，[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)把每个测试在单独的事务中执行来实现没有依赖的测试。

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

>默认的[`starapp`](https://docs.djangoproject.com/en/1.9/ref/django-admin/#django-admin-startapp)命令模板会在新的应用中创建一个`tests.py`脚本。当你只需要数量不多的测试时，直接写在这个文件里没有问题。但是随着你的测试越来越多时，你很可能想去把它们放到一个`test package`中，这样你就可以将你的测试分离成多个子模块，比如`test_models.py`，`test_views.py`，`test_forms.py`等等。任性选择你喜欢的模块组织结构吧。

>另请参阅[使用Django test runner去测试可重复利用的应用](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/#testing-reusable-applications)

------------------------------------

>**注意**：

>如果你的测试依赖于数据库，比如创建或者查询`models`，请确保你的测试类继承了[`django.test.TestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)而不是[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)。

>使用[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)避免了每次在一个事务中执行一个测试的开销，并且也不会影响数据库，但是如果你的测试需要直接与数据库交互，测试的行为和结果将可能因为测试被执行的顺序而变得不同。这将导致的结果是：当单独执行一个个测试单元时测试通过，但是当执行一个测试套件时测试失败。

----------------------------------

##执行测试

当你写好了测试后，利用你项目中的`manage.py`工具脚本使用[**`test`**](https://docs.djangoproject.com/en/1.9/ref/django-admin/#django-admin-test)命令：

```bash
$ ./manage.py test
```

对测试文件的识别是基于单元测试模块的内置的测试发现功能。默认情况下，这条命令将发现当前工作目录下所有以`test`开头的脚本文件中的测试代码。

你可以通过新添任意数量的“测试标签”到`./manage.py test`命令之后去指定特定的测试运行。每一个测试标签可以是一个`Python`包、模块、测试用例子类或者测试方法以点分隔的全路径。例如：

```bash
# Run all the tests in the animals.tests module
$ ./manage.py test animals.tests

# Run all the tests found within the 'animals' package
$ ./manage.py test animals

# Run just one test case
$ ./manage.py test animals.tests.AnimalTestCase

# Run just one test method
$ ./manage.py test animals.tests.AnimalTestCase.test_animals_can_speak
```


你也可以提供一个目录的路径去发现这个路径下的测试：

```bash
$ ./manage.py test animals/
```

如果你的测试文件没被命名为`test*.py`模式的文件名，你可以通过`-p`选项（或者`--pattern`）来自定义一个文件模式匹配名来匹配测试文件。例如：

```bash
$ ./manage.py test --pattern="tests_*.py"
```

当测试正在运行的时候，如果你按下`Ctrl-C`，它会等待当前正在跑的测试单元执行完毕然后优雅的退出，同时会输出所有测试失败的有关细节信息，报告有多少个测试用例被执行，其中有多少个失败、多少个错误，然后照常清除掉测试用的数据库。有时候，有一些测试会意外的出错，你可能想在所有的测试都跑完前获取一些有关这些错误的细节，因此如果你忘记给测试命令加上[`--failfast`](https://docs.djangoproject.com/en/1.9/ref/django-admin/#cmdoption-test--failfast)选项的话，按下`Ctrl-C`非常有效。

如果你不想等到当前正在跑的测试单元结束，你可以再按下一次`Ctrl-C`，然后测试会立即停止，但是这样很不优雅。所有在测试中断前的详细信息都不会被报告出来，而且所有由这次测试创建的数据库都不会被清除掉。

>**打开警告功能**

>在跑你的测试时，建议打开`Python`的警告功能：`python -Wall manage.py test`。这个`-Wall`标记告诉`Python`去显示一些警告消息。`Django`和`Python`很多其他的库一样，当一些功能方法已经不支持时，会使用警告去标记它们。它也可以标记出你代码中那些虽然严格来讲没有错，但是可以优化的地方。

###测试用数据库

当你的测试需要数据库支持时，换句话说，进行模型测试时，不会使用你真实的（生产）数据库，而是为测试新建一个全新的空的数据库（我在下文称之为**测试用数据库**）。

无论测试通过与否，在测试正常执行完毕时，都会清除掉测试用数据库。

>**`Django`1.8的新特性**

>你可以使用[`test --keepdb`](https://docs.djangoproject.com/en/1.9/ref/django-admin/#cmdoption-test--keepdb)选项来让测试用数据库不被清除。这将让测试用数据库一直保留。如果数据库不存在，它将首先被创建。此外，对测试用数据库的变动也会被及时更新。

默认情况下，测试用数据库的名字会以配置文件中的[**DATABASES**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASES)中[**NAME**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-NAME)
为基准，在它前边加上`test_`前缀。当在使用**SQLite**的时候，测试将默认使用内存数据库（也就是说，数据库将被在内存中创建，完全绕过文件系统）。在[**DATABASES**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASES)配置字典中，你可以添加一个[**`TEST`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASE-TEST)字典来配置测试用数据库。举个栗子，如果你想使用不同于默认名的测试用数据库名，可以在[**DATABASES**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASES)中每个给出的数据库的[**`TEST`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASE-TEST)字典中指定它们对应的数据库名[**NAME**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-TEST_NAME)。

使用**PostgreSQL**时，数据库配置中的[**USER**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-USER)用户还需要**postgres**数据库的访问读权限。

除了使用一个独立分离的数据库之外，测试数据库会同样使用你配置文件中所有数据库相关配置：[**ENGINE**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DATABASE-ENGINE)，[**USER**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-USER)，[**HOST**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-HOST)等等。测试用数据库是由配置中的[**USER**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-USER)用户创建，因此你需要确保该用户有足够的权限在操作系统中创建一个新的数据库。


如果你需要对数据库的编码进行细粒度的控制，可以在**`TEST`**配置字典中使用[**CHARSET**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-TEST_CHARSET)选项。如果你正在使用**MySQL**，你还可以使用[**COLLATION**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-TEST_COLLATION)选项去指定测试用数据库的排序规则。请参阅[**setting 文档**](https://docs.djangoproject.com/en/1.9/ref/settings/)获取更多进阶的配置的细节。

如果将**SQLite**内存数据库（3.7.13+）和`Python`（3.4+）搭配使用，共享高速缓存`Cache`会被启用，也就是说你可以编写能够在线程间共享数据库的测试了。

>**`Django`1.8的新特性**

>上边提到的有关**SQLite**共享高速缓存的功能已经实现。

--------------

>**执行测试的时候需要生产数据库中的数据？**

>如果你的代码模块在被编译后试图去访问生产数据库，这将导致测试用数据库创建之前的一些潜在的意外的结果。举个栗子，如果你在模块级(`module-level`)的代码中写了一个在生产数据库的查询，并且数据库的确存在，生产数据可能破坏你的测试。无论如何我们都不推荐在你的代码中有类似的`import-time`数据库查询。

>这条原则同样适用于[`ready()`](https://docs.djangoproject.com/en/1.9/ref/applications/#django.apps.AppConfig.ready)的自定义实现。

>**另请参阅**

>[多数据库测试进阶](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/#topics-testing-advanced-multidb)

-----------------------------

###测试执行的顺序

为了确保所有的测试用例代码开始执行的时候都使用没有脏数据的数据库，`Django`按照以下原则来给测试重新排序：

- 所有[**TestCase**](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TestCase)的子类首先执行
- 然后，执行所有其他基于`Django`的测试（继承自[`SimpleTestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.SimpleTestCase)或者[`TransactionTestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TransactionTestCase)的子类），但是执行的时候既不会保证也不会强制对它们进行排序。
- 最后执行所有其他的继承自[`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)的测试（包括文档测试），这些测试可能会修改数据库但是并不会在测试结束后恢复数据库到测试之前的状态。


>**注解**

>测试执行的新顺序可以暴露出一些你想象不到的测试对于执行顺序的依赖。当存在依赖于数据库状态的[`TransactionTestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.TransactionTestCase)类测试时，它们必须被及时更新以确保各个测试能够独立运行。

---------------------------

>**`Django`1.8的新特性**

>通过使用`test --reverse`选项，你可以倒序执行所有测试。这将帮助你确保你的测试之间是相互独立的。

-------------------

###仿真回滚

任何在`migrations`中加载的初始数据只有基于`TestCase`的测试可以获取，基于`TransactionTestCase`则不能获取，而且只在支持事务的后端上能够获取（`MyISAM`不支持事务处理）。对于那些继承了`TransactionTestCase`的测试类比如[` LiveServerTestCase`](https://docs.djangoproject.com/en/1.9/topics/testing/tools/#django.test.LiveServerTestCase)和[`StaticLiveServerTestCase`](https://docs.djangoproject.com/en/1.9/ref/contrib/staticfiles/#django.contrib.staticfiles.testing.StaticLiveServerTestCase)同样也适用于这条规则。

通过在`TestCase`或者`TransactionTestCase`的子类定义体中设置`serialized_rollback`选项为`True`，`Django`就可以为每一个测试用例重新从数据库载入数据，但是请注意这将减慢测试套件三倍左右的运行速度。

第三方应用或者那些依赖于`MyISAM`的开发将会需要设置上边提到的。但是一般情况下你使用的应该是支持事务的数据库而且大部分测试都继承`TestCase`，因此不需要这个设置。

初始序列化通常很快，但是如果你希望序列化时排除掉某些应用，或者希望稍微加快点测试执行速度，你可能需要添加那些你想排除掉的应用到[**`TEST_NON_SERIALIZED_APPS`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-TEST_NON_SERIALIZED_APPS)。

为了防止序列化后的数据被重复加载，可以定义`serialized_rollback=True`来在数据库刷新的时候关闭[`post_migrate`](https://docs.djangoproject.com/en/1.9/ref/signals/#django.db.models.signals.post_migrate)信号。

----------------------------

###其他测试情况

无论你的`settings`配置文件中[**`DEBUG`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DEBUG)变量的值是什么，所有`Django`的测试都按[**`DEBUG`**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-DEBUG)=`False`来执行。这是为了确保你的代码的输出能够和生产环境配置下的输出相匹配。

每个测试执行结束后高速缓存`Cache`不会被清空，如果你在生产环境下执行`manage.py test fooapp`，可以把数据从测试代码中插入到正在跑的系统的高速缓存中，这一点和数据库不同，不会有单独的一个“测试用高速缓存”被创建。但是这个有可能在将来有所[**改变**](https://code.djangoproject.com/ticket/11505)。

---------------------------

###读懂测试输出

当你执行你的测试时，你将看到大量的测试执行的准备信息。你可以通过命令选项`verbosity`去控制这些信息详细程度：

```django
Creating test database...
Creating table myapp_animal
Creating table myapp_mineral
```

上边是在告诉你测试当前正在创建一个测试用的数据库，就像上边的章节讲的那样。

一旦测试用数据库创建完毕，`Django`将开始执行你的测试。如果所有的测试执行正常且全部通过，你将看到像这样的输出：

```django
----------------------------------------------------------------------
Ran 22 tests in 0.221s

OK
```

如果存在一些测试失败，你将看到有关失败的测试的全部的细节信息：

```django
======================================================================
FAIL: test_was_published_recently_with_future_poll (polls.tests.PollMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/dev/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_poll
    self.assertEqual(future_poll.was_published_recently(), False)
AssertionError: True != False

----------------------------------------------------------------------
Ran 1 test in 0.003s

FAILED (failures=1)
```

有关这些错误输出的详细的说明已经超过这篇文档的范围，但是输出的信息其实很直观。你可以通过查阅`Python`的[**unittest**](https://docs.python.org/3/library/unittest.html#module-unittest)库的文档来获取更多详细信息。

另外请注意，如果测试中出现失败的测试或者报错，测试脚本的返回代码是1。如果测试全部通过，返回代码是0。如果你在一个`shell`脚本中使用测试脚本，并且你需要在`shell`脚本中获取测试成功与否的信息，这个特性就非常有用了。


-------------------

###加速测试

在较新的`Django`版本中，默认的密码哈希被设计的相当缓慢。如果在你的测试中需要认证很多用户，你可以使用一个定制的`settings`文件，然后将[**PASSWORD_HASHERS**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-PASSWORD_HASHERS)配置上一个更快的哈希算法:

```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]
```

最后，不要忘记添加`fixtures`中用到的相应的哈希算法到[**PASSWORD_HASHERS**](https://docs.djangoproject.com/en/1.9/ref/settings/#std:setting-PASSWORD_HASHERS)中，如果你用到`fixtures`的话。
