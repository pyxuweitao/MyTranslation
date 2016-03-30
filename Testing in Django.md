# 在Django项目中进行测试

------------------------------

- 翻译人：徐维涛
- 翻译周期：2016-03-01~2016-03-01
- 中文译名：在Django项目中进行测试
- 原标题和链接：[Testing in Django](https://docs.djangoproject.com/en/1.9/topics/testing/)
- 说明：`Django1.9`官方文档翻译
- 版权声明：未经本人许可，禁止转载发布

-------------------------------

对现代`Web`开发者来说，自动化测试是一个非常有用的`bug-killing`工具。你可以使用一系列测试（也就是一个测试套件`test suite`）来解决或者避免代码中许多问题：

- 当你在写新的代码时，你可以使用测试去验证你的代码是否按照你所想象的那样执行。
- 当你在重构或者修改之前的代码时，你可以使用测试去确保你的修改不会影响到你的应用正常运行。

对`Web`应用进行测试是一个复杂的工作，因为`Web`应用是由多层逻辑构成——从`HTTP`级的请求处理，到表单验证和解析，再到模板渲染。使用`Django`的测试执行框架和各式各样的辅助工具，你可以模拟客户端请求、插入测试数据、检验你应用的输出是否正确等，总之就是去核实你的代码在做它应该做的事情。

最棒的地方是，在`Django`中进行测试真的很简单。

在`Django`中编写测试，我们推荐使用`Python`标准库中内建的 [**unittest**](https://docs.python.org/3/library/unittest.html#module-unittest) 模块。如何使用它在 **[Django中编写和执行测试](https://docs.djangoproject.com/en/1.9/topics/testing/overview/)** 文档中有详细说明。

你也可以使用任何其他的`Python`测试框架，`Django`提供了一个`API`和一些工具来帮助你把其他测试框架集成到`Django`中。具体做法可以参考文档**[Advanced testing topics](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/)**中的**[使用其他测试框架](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/#other-testing-frameworks)**章节。

----------------

目录章节：

- [Django中编写和执行测试](http://xuweitao.me/djangozhong-bian-xie-he-zhi-xing-ce-shi.html)
- [测试工具 - 测试客户端](http://xuweitao.me/djangoce-shi-gong-ju-zhi-ce-shi-ke-hu-duan.html)
- [Advanced testing topics](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/)

