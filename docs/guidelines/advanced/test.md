# Test - 测试

测试是软件开发中一个不可避免的环节，在代码级别进行测试，能够在部署钱尽早发现程序中的异常，增强软件的健壮性。

在遵循 [TDD](http://agiledata.org/essays/tdd.html#:~:text=Test%2Ddriven%20development%20(TDD),Agile%20DBAs%20for%20database%20development.)
原则来开发，能有效提高代码的设计。

## 1. 测试工具

在 Python 中除了有语言内置的测试框架之外，还有许多第三方测试框架，一些非测试框架内部也会内置测试框架。其目的都是在内置测试框架的基础上
增加了一些特性，让编写测试更加方便，测试过程更加顺畅。

为了方便测试框架查找测试用例，在编写测试时应遵循一定的规范：

- 测试模块要以 `test_` 开头
- 测试方法要以 `test_` 开头
- 测试类名要以 `Test` 开头

### 1.1 内置测试框架

Python 内置测试框架是 [unittest](https://docs.python.org/zh-cn/3/library/unittest.html) ，是受到了 JUnit 的启发，并且在使用上与其他语言的
单元测试框架类似。

面向对象的方式所支持的几个概念：

- 测试脚手架： `test fixture` 表示为了展开一项或多项测试所需要准备的工作，以及相关的清理工作
- 测试用例：一个测试用例是一个独立的单元测试。
- 测试套件：是一系列的测试用例，或测试套件。
- 测试运行器：用于执行和输出测试结果。

下面是一个最简单的测试用例：

```python
# Test
import unittest
from unittest import TestCase


class TestSum(TestCase):

    def test_sum(self):
        """Test sum"""
        assert sum([1, 1]) == 2


if __name__ == '__main__':
    unittest.main()

```

可以通过运行该文件运行测试，也可以用 `python -m test_xxx.py` 运行测试模块。

组织测试的测试代码：

```python
# Test
from csv import DictReader
import unittest
from unittest import TestCase
from tempfile import NamedTemporaryFile


class TestSum(TestCase):

    def test_sum(self):
        """Test sum"""
        assert sum([1, 1]) == 2


class TestCsv(TestCase):

    def setUp(self) -> None:
        self.temp_file = NamedTemporaryFile(suffix='csv')
        self.filename = self.temp_file.name
        with open(self.filename, 'w') as obj:
            obj.writelines([
                'name,age\n',
                'foo,12\n',
                'bar,15\n'
            ])

    def test_csv(self):
        with open(self.filename, 'r') as obj:
            reader = DictReader(obj)
            data = list(reader)
        self.assertEqual(len(data), 2)

    def tearDown(self) -> None:
        self.temp_file.close()


def suite():
    _suite = unittest.TestSuite()
    _suite.addTest(TestSum())
    _suite.addTest(TestCsv())
    return _suite


if __name__ == '__main__':
    runner = unittest.TextTestRunner()
    runner.run(suite())

```

使用 `TestSuite` 和 `TextTestRunner` 组织测试，可以让代码的逻辑更加清晰。

#### 1.1.1 Mock 对象

在进行单元测试的时候，难免会遇到依赖具体的对象或资源的情况。为了只测试具体单元的逻辑，就需要模拟单元逻辑所依赖的内容。

使用 [unittest.mock](https://docs.python.org/zh-cn/3/library/unittest.mock-examples.html) 可以很好解决这中问题。

如下例子：

```python
# Test
import unittest
from typing import Any
from unittest import TestCase
from unittest.mock import Mock


class Session:

    def close(self, connection: Any):
        connection.close()


class TestSession(TestCase):

    def setUp(self) -> None:
        self.session = Session()

    def test_close(self):
        mock = Mock()
        self.session.close(mock)
        mock.close.assert_called_with()


if __name__ == '__main__':
    unittest.main()

```

在测试 `Session.close` 这个单元逻辑的时候，依赖一个 `connection` 对象。因为单元测试仅关注单元内部逻辑是否正确，即给定输入，测试其内部逻辑。
所以使用一个 `Mock` 对象模拟入参，然后判断入参是否在逻辑内被调用。

除了模拟对象，还可以模拟类：

```python
# Test
import unittest
from unittest import TestCase
from unittest.mock import patch


class Session:

    def exist(self):
        """"""

    def delete(self):
        if self.exist():
            return True
        return False


class TestSession(TestCase):

    def setUp(self) -> None:
        self.session = Session()

    def test_delete(self):
        with patch.object(Session, 'exist', return_value=True) as mock_session:
            session = Session()
            self.assertTrue(session.delete())
        mock_session.assert_called_once()


if __name__ == '__main__':
    unittest.main()

```

案例中，测试的是 `Session.delete` 方法，内部逻辑依赖 `Session.exist` 。因为仅测试单元逻辑所以将它依赖
调用的 `Session.exist` 模拟掉。

### 1.2 Pytest

[Pytest](https://docs.pytest.org/en/latest/) 是在 [unittest](https://docs.python.org/zh-cn/3/library/unittest.html) 的基础上
增加了大量语法糖，让测试更加简便和灵活。并且带有插件功能，方便集成其他功能。

**由于 Pytest 能兼容其他大多数测试框架，而且它也具有强大的功能，所以推荐使用 Pytest 作为主要测试框架使用。**

安装：

```bash
pip install pytest
```

编写测试文件：

```python
# content of test_sample.py
def inc(x):
    return x + 1


def test_answer():
    assert inc(3) == 5

```

在终端运行 `pytest` 即可自动发现测试，并运行。

???+ note "提示"

    `pytest` 会自动发现所有以 `test_` 开头和 `_test.py` 结尾的文件，并加载所有以 `test_` 开头的方法和 `Test` 开头的类。

#### 1.2.2 目录结构的选择

在项目结构上，推荐使用 `src` 目录放源代码，同级的 `tests` 放测试模块，测试模块的组织和 `src` 的包结构一致，测试的功能
相对应。

```text
setup.py
src/
    mypkg/
        __init__.py
        app.py
        view.py
tests/
    __init__.py
    foo/
        __init__.py
        test_view.py
    bar/
        __init__.py
        test_view.py
```

其他风格的使用可以参考 [Choosing a test layout / import rules](https://docs.pytest.org/en/latest/explanation/goodpractices.html#choosing-a-test-layout-import-rules)

#### 1.2.1 fixture

Pytest 的 [fixture](https://docs.pytest.org/en/latest/explanation/fixtures.html#about-fixtures) 可以为测试提供一定的环境。

```python
import pytest


class Fruit:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return self.name == other.name


@pytest.fixture
def my_fruit():
    return Fruit("apple")


@pytest.fixture
def fruit_basket(my_fruit):
    return [Fruit("banana"), my_fruit]


def test_my_fruit_in_basket(my_fruit, fruit_basket):
    assert my_fruit in fruit_basket

```

在测试是，公共的 `fixture` 一般推荐放在 `conftest.py` 中。

更复杂的 `fixture` ：

```python
import pytest


@pytest.fixture
def order():
    return []


@pytest.fixture
def a(order):
    order.append("a")


@pytest.fixture
def b(a, order):
    order.append("b")


@pytest.fixture
def c(a, b, order):
    order.append("c")


@pytest.fixture
def d(c, b, order):
    order.append("d")


@pytest.fixture
def e(d, b, order):
    order.append("e")


@pytest.fixture
def f(e, order):
    order.append("f")


@pytest.fixture
def g(f, c, order):
    order.append("g")


def test_order(g, order):
    assert order == ["a", "b", "c", "d", "e", "f", "g"]

```

#### 1.2.3 参数化测试

在针对同一个逻辑有多种不同输入进行测试时，直接想到的处理方式就是做成工厂，然后在测试方法中
构造参数列表传递给工厂。这在 unittest 中称作复用测试逻辑。否则就需要为测试逻辑编写不同参数
的测试方法。

但在 Pytest 中可以使用[参数化测试](https://docs.pytest.org/en/latest/how-to/parametrize.html)，
轻松解决这种问题。

```python
"""Test log"""
import pytest


def update_log_level(debug: bool, level: str) -> str:
    if debug:
        return 'DEBUG'
    return level


@pytest.mark.parametrize(
    ['debug', 'level', 'expect_value'],
    [
        (True, '', 'DEBUG'),
        (True, 'INFO', 'DEBUG'),
        (False, 'DEBUG', 'DEBUG'),
        (False, 'INFO', 'INFO'),
    ]
)
def test_log_level(debug: bool, level: str, expect_value):
    """Test log level"""
    log_level_name = update_log_level(debug, level)
    assert log_level_name == expect_value

```

参数化测试带来的好处非常直观，而且测试编写也变得简单。

#### 1.2.4 插件

Pytest 拥有大量的[插件](https://docs.pytest.org/en/latest/reference/plugin_list.html) ，而且基本上是安装即可和 Pytest 无缝
集成，轻松使用。

下面列举几个常用的插件

##### 1.2.4.1 pytest-asyncio

[pytest-asyncio](https://pypi.org/project/pytest-asyncio/) 可以轻松测试 `asyncio` 方法

```python
@pytest.mark.asyncio
async def test_some_asyncio_code():
    res = await library.do_something()
    assert b'expected result' == res

```

##### 1.2.4.2 pytest-mock

[pytest-mock](https://pypi.org/project/pytest-mock/) 可以像使用 `fixture` 一样使用 `mock` ，而不必导入 `unittest.mock`

```python
import os

class UnixFS:

    @staticmethod
    def rm(filename):
        os.remove(filename)

def test_unix_fs(mocker):
    mocker.patch('os.remove')
    UnixFS.rm('file')
    os.remove.assert_called_once_with('file')

```

##### 1.2.4.3 pytest-cov

[pytest-cov](https://pypi.org/project/pytest-cov/) 让 [coverage](https://coverage.readthedocs.io/en/coverage-5.5/) 和 Pytest 集成，
方便使用。

### 1.3 框架自带测试

本节内容主要简单描述一些框架自带的测试的使用。 Pytest 也都能兼容这些测试。所以如果使用框架推荐的写法来写测试，在使用 Pytest 运行
也是没有问题的。

#### 1.3.1 Django

Django 的[单元测试](https://docs.djangoproject.com/en/3.2/topics/testing/)也是基于 `unittest` 来做的，
只不过增加了一些框架上的内容，便于在测试时，附带框架功能。

测试用例：

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

运行测试 `./manage.py test` 。

在运行时，内部逻辑依然是通过 `unittest` 来自动查找测试类。

#### 1.3.2 Fastapi

[Fastapi](https://fastapi.tiangolo.com/tutorial/testing/) 仅仅提供了带有框架功能的 `TestClient` 。初始化的实例
方便测试 API 接口，而不是真正启动一个 Web 服务。

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}

```

运行 `pytest` 。
