# 测试

测试是保障安全上线一个重要的步骤，编写良好的测试，可以在发布之前尽可能避免 BUG 出现。
在修改功能后，也可以通过回归测试，检查现有功能的稳定性。

编写单元测试过程，和开发顺序一直，现测试三个模块，再测试 `manage` 模块，最后测试调用逻辑。

测试时，使用的是 `pytest` 工具，而不是使用 `unittest` 。

## 测试 extractor

在 `tests` 包中新建 `test_extractor.py` 内容如下：

```python
"""Test extractor"""
import pytest

from example_etl.extractor.base import BaseExtractor
from example_etl.extractor.file import FileExtractor


def test_base_source(mocker):
    """Test base extractor"""
    close_mock = mocker.patch.object(BaseExtractor, 'close')
    with pytest.raises(NotImplementedError):
        with BaseExtractor(mocker.MagicMock()) as base:
            base.extract()
    assert close_mock.called_once()


def test_file_source(mocker, foo_file):
    """Test file extractor"""
    extractor = FileExtractor(mocker.MagicMock())
    extractor.settings.FILE_EXTRACTOR_PATH = foo_file
    data = list(extractor.extract())
    assert data == ['foo']

```

测试代码中，分别测试了 `BaseExtractor` 和 `FileExtractor` 的逻辑。

在测试逻辑中使用了 `mocker` 功能，可以在测试单元逻辑时，将其依赖的东西 mock 掉。在 `pytest` 测试框架中，
使用 `pytest-mock` 扩展可以很方便的使用 `mocker` 对象。

安装 `pytest-mock` ：

```bash
poetry add -D pytest-mock
```

> 这里使用了 `poetry add -D` ，意思是将 `pytest-mock` 安装到开发环境依赖中。
> 当在一个新环境 `poetry install` 安装时，安装所有非可选组的依赖项。

测试代码中同时使用了 `foo_file` 的 fixture ，它定义在 `conftest.py` 中，内容如下：

```python
"""Test config"""
import tempfile

import pytest
from click.testing import CliRunner


@pytest.fixture()
def clicker():
    """clicker fixture"""
    yield CliRunner()


@pytest.fixture()
def foo_file():
    """foo file"""
    with tempfile.NamedTemporaryFile(mode='w') as file:
        file.write('foo')
        file.flush()
        yield file.name

```

然后在命令行中运行 `pytest` ，测试刚刚编写的测试代码。可以看到如下输出：

```text
❯ pytest
================================================================= test session starts =================================================================
platform linux -- Python 3.10.0, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: /tmp/test/example_etl, configfile: setup.cfg, testpaths: tests
plugins: cov-3.0.0, mock-3.6.1
collected 12 items                                                                                                                                    

tests/test_cmdline.py .....                                                                                                                     [ 41%]
tests/test_extcator.py ..                                                                                                                       [ 58%]
tests/test_log.py ....                                                                                                                          [ 91%]
tests/tests.py .                                                                                                                                [100%]

================================================================= 12 passed in 0.18s ==================================================================
```

测试成功。

## 测试 transformer

在 `tests` 包中创建 `test_transformer.py` 内容如下：

```python
"""Test transformer"""
import pytest

from example_etl.transformer.base import BaseTransformer
from example_etl.transformer.strip import StripTransformer


def test_base_process(mocker):
    """Test base transformer"""
    process = BaseTransformer(mocker.MagicMock())
    with pytest.raises(NotImplementedError):
        process.transform('foo')


@pytest.mark.parametrize(
    'data, expect_value',
    [
        ('xx ', 'xx'),
        (' xx ', 'xx'),
        ('xx', 'xx'),
    ]
)
def test_strip_process(mocker, data, expect_value):
    """Test strip transformer"""
    processor = StripTransformer(mocker.MagicMock())
    res = processor.transform(data)
    assert res == expect_value

```

在测试 `test_strip_process` 时，使用了 pytest 的参数化测试。可以在一个测试逻辑中，测试不同的输入输出值。

再次运行 `pytest` 命令，检测测试是否正确。

## 测试 loader

在 `tests` 包中创建 `test_loader.py` ，内容如下：

```python
"""Test loader"""
import tempfile
from pathlib import Path

import pytest

from example_etl.loader.base import BaseLoader
from example_etl.loader.file import FileLoader


def test_base_dest(mocker):
    """Test base loader"""
    close_mock = mocker.patch.object(BaseLoader, 'close')
    with BaseLoader(mocker.MagicMock()) as base:
        with pytest.raises(NotImplementedError):
            base.load('foo')
    assert close_mock.called_once()


def test_file_dest(mocker):
    """Test file loader"""
    with tempfile.NamedTemporaryFile() as file:
        settings_mock = mocker.MagicMock()
        settings_mock.FILE_LOADER_PATH = file.name
        with FileLoader(settings_mock) as loader:
            loader.load('foo')
        file = Path(file.name)
        stat = file.stat()
        assert stat.st_size == 3

```

在测试 `test_file_dest` 时，使用了一个临时文件作为目标写入，使用了 `with` 关键字打开文件，
在测试完成后，会自动删除临时文件。

再次运行 `pytest` 检查测试结果。

## 测试 manage

`manage` 的逻辑同样需要测试，在 `tests` 包中创建 `test_manage.py` 文件，内容如下：

```python
"""Test manage"""
import pytest

from example_etl.exceptions import PluginNotFoundError
from example_etl.extractor.file import FileExtractor
from example_etl.manage import Manage, get_extension


def test_get_extension():
    """Test get extension"""
    plugin = get_extension('example_etl.extractor', 'file')
    assert plugin is FileExtractor


def test_get_extension_error():
    """Test get extension error"""
    with pytest.raises(PluginNotFoundError):
        get_extension('example_etl.extractor', 'xxx')


def test_manage_run(mocker):
    """Test manage run"""
    mocker.patch('example_etl.manage.get_extension')
    process_mock = mocker.patch.object(Manage, 'transform')
    manage = Manage()

    manage.run()
    assert process_mock.called_once()


def test_manage_transform(mocker):
    """Test manage transform"""
    magic_mock = mocker.MagicMock()
    manage = Manage()
    manage.transformer = magic_mock
    magic_mock.extract.return_value = [1, 2]
    manage.transform(magic_mock, magic_mock)

    assert magic_mock.extract.called_once()
    assert magic_mock.load.call_count == 2
    assert magic_mock.transform.call_count == 2

```

在测试时，需要保证配置文件中存在之前在代码中使用的变量。

在 `src/example_etl/config/settings.yml` 文件中加入如下内容：

```yaml
extractor_name: file
transformer_name: strip
loader_name: file
```

配置程序默认使用三个已经实现的逻辑。

再次运行 `pytest` 检查测试结果。

## 检查测试覆盖率

测试覆盖率指示编写的单元测试，覆盖了多少源代码。能够通过测试覆盖率查看还有哪些内容没有被测试到。

运行 `pytest --cov` 查看测试覆盖率。

```text
❯ pytest --cov
================================================================= test session starts =================================================================
platform linux -- Python 3.10.0, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: /tmp/test/example_etl, configfile: pyproject.toml, testpaths: tests
plugins: cov-3.0.0, mock-3.6.1
collected 24 items                                                                                                                                    

tests/test_cmdline.py .....                                                                                                                     [ 20%]
tests/test_extcator.py ..                                                                                                                       [ 29%]
tests/test_loader.py ..                                                                                                                         [ 37%]
tests/test_log.py ......                                                                                                                        [ 62%]
tests/test_manage.py ....                                                                                                                       [ 79%]
tests/test_transformer.py ....                                                                                                                  [ 95%]
tests/tests.py .                                                                                                                                [100%]

---------- coverage: platform linux, python 3.10.0-final-0 -----------
Name                                      Stmts   Miss  Cover
-------------------------------------------------------------
src/example_etl/__init__.py                   1      0   100%
src/example_etl/cmdline.py                   23      0   100%
src/example_etl/config/__init__.py            8      0   100%
src/example_etl/constants.py                  1      0   100%
src/example_etl/exceptions.py                10      2    80%
src/example_etl/extractor/__init__.py         0      0   100%
src/example_etl/extractor/base.py            13      0   100%
src/example_etl/extractor/file.py            12      0   100%
src/example_etl/loader/__init__.py            0      0   100%
src/example_etl/loader/base.py               12      0   100%
src/example_etl/loader/file.py               15      0   100%
src/example_etl/log.py                       19      0   100%
src/example_etl/manage.py                    33      0   100%
src/example_etl/transformer/__init__.py       0      0   100%
src/example_etl/transformer/base.py           5      0   100%
src/example_etl/transformer/strip.py          7      0   100%
tests/__init__.py                             0      0   100%
tests/conftest.py                            12      0   100%
tests/test_cmdline.py                        10      0   100%
tests/test_exceptions.py                      0      0   100%
tests/test_extcator.py                       14      0   100%
tests/test_loader.py                         20      0   100%
tests/test_log.py                            10      0   100%
tests/test_manage.py                         25      0   100%
tests/test_transformer.py                    12      0   100%
tests/tests.py                                3      0   100%
-------------------------------------------------------------
TOTAL                                       265      2    99%


================================================================= 24 passed in 0.41s ==================================================================

```

通过覆盖率可以看到 `src/example_etl/exceptions.py` 的逻辑还有没测试的。

## 完善其他测试

在 `tests` 模块中创建 `test_exceptions.py` 文件，内容如下：

```python
"""Test exception"""
from example_etl.exceptions import PluginNotFoundError


def test_plugin_not_found_error():
    """test plugin not found error"""
    error = PluginNotFoundError('foo', 'bar')
    assert str(error) == 'Can not found "bar" plugin in foo'

```

再次运行 `pytest --cov` 可以看到覆盖率 100% 。

## 代码风格检测

为了让开发风格达到统一，使用代码格式化工具检测。

使用 `isort` 将导包部分格式化为统一格式，使用 `pylint` 检测代码是否符合 PEP8 规范，同时还能检测
一些不标准的的语法，并给出修改建议。

执行 `isort . --check-only --diff` 检测代码风格，并仅输出不符合规范的导包，执行 `isort` 会自动格式
化代码。

运行 `pylint src tests` 检查 `src` 目录和 `tests` 目录下的 Python 代码。会输出不符合规范的内容，然后
根据建议修改即可。

## 自动化测试

项目默认带有 `tox` 自动化配置。当开发完成后，直接运行 `tox` ，会自动在模拟环境中测试代码。测试时，会
创建独立的虚拟环境，然后将项目打包安装到环境中，再进行测试。

`tox` 会自动执行 `pytest` 测试， 导包检测，代码风格检测。
