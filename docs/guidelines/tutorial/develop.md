# 功能开发

根据前面的系统设计， ETL 项目总共有三个核心模块，分别是 `extractor` 、 `transformer` 和 `loader` 。为了
能运行逻辑，还需要一个 `manage` 模块用来编排三个模块的逻辑。然后会在命令行中注册一个入口方法，调用 `mange` 的逻辑。

## extractor

`extractor` 的作用是从源目标提取数据，目标可以是文件、数据库、消息队列等。这典型是一个多实现的情况，同时也
为了统一其他开发人员编写自己的 `extractor` ，就需要对 `extractor` 做出一个抽象设计。我们使用 `BaseExtractor` 类
做一个抽象基类。

### extractor 基类

创建 `extractor` 包，并在里面新建一个 `base.py` 文件，文件内容如下：

> 注意：Python 的包是一个文件夹，里面包含一个 `__init__.py` 文件。只是一个空文件夹，不是合法的 Python 包。

`src/example_etl/extractor/base.py`

```python
"""Base extractor."""
from typing import Iterable


class BaseExtractor:
    """Base extractor"""

    def __init__(self, settings):
        self.settings = settings
        self.setup()

    def setup(self):
        """Setup something when init extractor"""

    def extract(self) -> Iterable[str]:
        """Extract data."""
        raise NotImplementedError()

    def close(self):
        """Close something."""

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

```

`BaseExtractor` 有一个抽象方法 `extract` ，需要实现时，继承该类，并实现这个方法即可。 `BaseExtractor` 同时默认实现了
`__enter__` 和 `__exit__` 两个方法，目的是让实现类可以通过 `with` 关键字调用，并自动管理 `close` 方法。这对于数据库
连接的实现很有帮助。

`BaseExtractor` 接收一个 `settings` 对象，这个对象其实就是 `example_etl.config.settings` 对象，这里通过调用者传递。

`extract` 的返回值是一个可迭代的对象，迭代内容为 `str` 。

### extractor 的 file 实现

基于 `BaseExtractor` 做一个文件提取其实现。

在 `extractor` 包中创建文件 `file.py` ，并增加如下内容：

`src/example_etl/extractor/file.py`

```python
"""
File extractor

extract data from file.
"""
import logging
from typing import Iterable

from example_etl.constants import DEFAULT_ENCODING
from example_etl.extractor.base import BaseExtractor

logger = logging.getLogger(__name__)


class FileExtractor(BaseExtractor):
    """File extractor"""

    def extract(self) -> Iterable[str]:
        """Open and read file"""
        extractor_path = self.settings.FILE_EXTRACTOR_PATH
        logger.info('Extract data from %s', extractor_path)
        with open(extractor_path, 'r', encoding=DEFAULT_ENCODING) as file:
            for i in file:
                yield i

```

在实现的 `extract` 方法中，从 `FileExtractor.settings` 对象中获取了一个 `FILE_EXTRACTOR_PATH` 变量，这个变量是从
配置文件中获取的。使用时，需要在配置文件中增加 `file_extractor_path: /tmp/foo.txt` 的值。

`extract` 方法中直接可以通过返回迭代对象的方式自动管理文件读对象。

注意一点的是，打开文件时使用了默认字符集的常量值 `DEFAULT_ENCODING` 。所以还要创建 ``src/example_etl/constants.py``，
并加入如下内容：

```python
"""Constants"""
DEFAULT_ENCODING = 'utf-8'

```

`file.py` 文件中还创建了一个全局 `logger` 对象，对象名称使用了 `__name__` 获取该包的名称。在打印日志时，显示的包名
为 `example_etl.extractor.file` 。在 `extract` 方法中打印一条执行记录。

## transformer

`transformer` 模块的功能是转换读取到的逻辑。在这个过程中，通过接收 `extractor` 读取到的文本，处理后传递给 `loader` 。

此过程可以执行去除空格、删减字符等操作。

为了方便实现，创建一个基类 `BaseTransformer` 。

### transformer 基类

首先创建 `transformer` 包，然后新建 `BaseTransformer.py` 文件：

`src/example_etl/transformer/BaseTransformer.py`

```python
"""Base transformer"""


class BaseTransformer:
    """Base transformer"""

    def __init__(self, settings):
        self.settings = settings

    def transform(self, data: str) -> str:
        """Transform data"""
        raise NotImplementedError()

```

`BaseTransformer` 同样接收一个 `settings` 对象。其抽象方法 `transform` 接收一个字符串类型的 `data` 并返回 `str` 类型
的数据。

### tansformer 去空格实现

`BaseTransformer` 实现一个可以删除文本前后空格的实现 `StripTransformer` ：

创建 `strip.py`

`src/example_etl/transformer/strip.py`

```python
"""Transform data and remove blank of data star and end."""
import logging

from example_etl.transformer.base import BaseTransformer

logger = logging.getLogger(__name__)


class StripTransformer(BaseTransformer):
    """
    Transform data and remove blank of data star and end.
    """
    def transform(self, data: str) -> str:
        """Remove blank of data star and end."""
        logger.debug('Strip data: "%s"', data)
        return data.strip()

```

`StripTransformer` 实现是通过字符串方法 `strip` 删除接收到字符串数据前后空格，并返回结果。

`strip.py` 文件中同样初始化一个 `logger` 对象，在 `transform` 中打印一条记录。需要注意的是，这里使用了
`debug` 方法，打印的日志为 `DEBUG` 级别。当日志级别设置在 `INFO` 时，这里的执行记录是不会打印的。对于
关注低的记录，可以使用 `DEBUG` 。

## loader

`loader` 模块用来将 `transformer` 转换的数据加载到目标位置。目标可以是文件、数据库、消息队列等。

同样的，对 `loader` 做出抽象类 `BaseLoader` 。

### loader 基类

在 `loader` 包中创建 `base.py` 文件，文件内容如下：

`src/example_etl/loader/base.py`

```python
"""Base loader"""


class BaseLoader:
    """Base loader"""

    def __init__(self, settings):
        self.settings = settings
        self.setup()

    def setup(self):
        """Setup something when init loader."""

    def load(self, data: str):
        """Write data to loader"""
        raise NotImplementedError()

    def close(self):
        """Close something"""

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

    def __enter__(self):
        return self

```

在 `BaseLoader` 中有一个 `load` 的抽象方法，用来给继承类实现。默认的 `setup` 方法可以在初始化
时做一些逻辑，比如打开文件、创建数据库连接等。 `close` 用来关闭这些逻辑。 `__exit__` 和 `__enter__` 可以
让 `BaseLoader` 通过 `with` 关键字使用。

需要注意的是， `BaseLoader` 的 `load` 方法中不能有创建连接对象的逻辑，因为 `load` 会出现在循环中的。

### loader 的 file 实现

默认实现一个将数据写入文件的实现类 `FileLoader` 。

在 `loader` 包中创建 `file.py` 文件，文件内容如下：

`src/example_etl/loader/file.py`

```python
"""
File loader

Write data to loader file.
"""
import logging

from example_etl.constants import DEFAULT_ENCODING
from example_etl.loader.base import BaseLoader

logger = logging.getLogger(__name__)


class FileLoader(BaseLoader):
    """
    File loader
    """
    file = None

    def setup(self):
        """Open a file when init loader."""
        loader_path = self.settings.FILE_LOADER_PATH
        logger.info('Write data to %s', loader_path)
        self.file = open(loader_path, 'w', encoding=DEFAULT_ENCODING)  # pylint: disable=consider-using-with

    def load(self, data: str):
        """Write data to a file."""
        self.file.write(data)
        self.file.flush()

    def close(self):
        """Close file object when task done."""
        self.file.close()

```

该类在 `setup` 方法中打开文件对象，并在 `close` 方法中关闭文件。 `load` 方法会写入数据，并立即
将内容刷新到文件中。

## 插件注册

三个基础模块使用插件机制自动发现，并通过配置文件指定需要使用的具体实现。在后续使用中，基于抽象基类
开发的其他实现也是通过这种来做。

安装插件框架 [stevedore](https://docs.openstack.org/stevedore/latest/index.html) ：

```bash
pipenv install stevedore
```

将 `stevedore` 依赖加入到项目安装依赖列表中。

编辑 `setup.cfg` 文件，在 `options` 下的 `install_requires` 下增加 `stevedore` ：

```ini
[options]
python_requires > = 3.9
include_package_data = True
packages = find:
package_dir =
    = src
install_requires =
    click
    dynaconf
    stevedore

```

### 注册插件

将上述实现的三个类注册到命名空间中。

编辑 `setup.cfg` 文件，在 `[options.entry_points]` 中增加如下内容：

```ini

example_etl.extractor =
    file = example_etl.extractor.file:FileExtractor

example_etl.loader =
    file = example_etl.loader.file:FileLoader

example_etl.transformer =
    strip = example_etl.transformer.strip:StripTransformer
```

修改完后的文件内容如下：

```ini
[metadata]
name = example_etl
version = attr: example_etl.__version__
author = huagang
author_email = huagang517@126.com
description = Etl tools
keywords = etl
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.9
    Programming Language :: Python :: 3.10

[options]
python_requires > = 3.9
include_package_data = True
packages = find:
package_dir =
    = src
install_requires =
    click == 8.0.3
    dynaconf == 3.1.7
    stevedore == 3.5.0

[options.packages.find]
where = src
exclude =
    tests*
    docs

# https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html
[options.entry_points]
console_scripts =
    example_etl = example_etl.cmdline:main

example_etl.extractor =
    file = example_etl.extractor.file:FileExtractor

example_etl.loader =
    file = example_etl.loader.file:FileLoader

example_etl.transformer =
    strip = example_etl.transformer.strip:StripTransformer

# Packaging project data in module example_etl.
# https://setuptools.readthedocs.io/en/latest/userguide/datafiles.html?highlight=package_data
[options.package_data]
example_etl.config =
    settings.yml

# Copy data for user from project when pip install.
# The relative path is prefix `sys.prefix` . eg: `/usr/local/`.
# Path and data will remove When pip uninstall.
# https://docs.python.org/3/distutils/setupscript.html#installing-additional-files
[options.data_files]
etc/example_etl =
    src/example_etl/config/settings.yml

```

这么做的目的是将 `FileExtractor` 、 `FileLoader` 、 `StripTransformer` 分别注册到 `entry_points` 中，
然后在程序中使用 `import.metadata` 根据名称空间查找。而 `stevedore` 则是封装了查找的复杂逻辑，让使用
更简单。

将项目以可编辑模式安装到当前环境：

```bash
pip install -e .
```

可以在 `src/example_etl.egg-info/entry_points.txt` 文件中查看打包后的注册信息：

```text
[console_scripts]
example_etl = example_etl.cmdline:main

[example_etl.extractor]
file = example_etl.extractor.file:FileExtractor

[example_etl.loader]
file = example_etl.loader.file:FileLoader

[example_etl.transformer]
strip = example_etl.transformer.strip:StripTransformer


```

## manage

`manage` 模块是用来编排前面三个模块的逻辑。

创建 `manage.py` ，文件内容如下：

```python
"""Manage"""
import logging
from typing import Type

from stevedore import ExtensionManager

from example_etl.config import settings
from example_etl.exceptions import PluginNotFoundError
from example_etl.extractor.base import BaseExtractor
from example_etl.loader.base import BaseLoader
from example_etl.transformer.base import BaseTransformer

logger = logging.getLogger(__name__)


class Manage:
    """Manager"""

    def __init__(self):
        self.extractor_kls: Type[BaseExtractor] = get_extension(
            'example_etl.extractor',
            settings.EXTRACTOR_NAME,
        )
        self.loader_kls: Type[BaseLoader] = get_extension(
            'example_etl.loader',
            settings.LOADER_NAME,
        )
        self.transformer_kls: Type[BaseTransformer] = get_extension(
            'example_etl.transformer',
            settings.TRANSFORMER_NAME,
        )

        self.transformer: BaseTransformer = self.transformer_kls(settings)

    def run(self):
        """Run manage"""
        with self.extractor_kls(settings) as extractor:
            with self.loader_kls(settings) as loader:
                self.transform(extractor, loader)
        logger.info('Exit example_etl.')

    def transform(self, extractor: BaseExtractor, loader: BaseLoader):
        """Transform data from extractor to loader."""
        logger.info('Start transformer data ......')
        for i in extractor.extract():
            data = self.transformer.transform(i)
            loader.load(data)

        logger.info('Data processed.')


def get_extension(namespace: str, name: str):
    """Get extension by name from namespace."""
    extension_manager = ExtensionManager(namespace=namespace, invoke_on_load=False)
    for ext in extension_manager.extensions:
        if ext.name == name:
            logger.info('Load plugin: %s in namespace "%s"', ext.plugin, namespace)
            return ext.plugin

    raise PluginNotFoundError(namespace=namespace, name=name)

```

在 `manage` 中封装了一个通过名称空间和名称两个参数查找插件的方法 `get_extension` 。当找不到对应
的插件时，会抛出 `PluginNotFoundError` 异常。

在 `Manage` 类的 `__init__` 方法中，分别从三个名称空间查找实现类，查找的名字则是通过配置文件的
变量获取的，这样就可以通过配置灵活地调整需要使用的具体实现了。

`run` 方法中使用 `with` 关键字分别初始化 `extractor` 和 `loader` ，在逻辑结束时，可以自动管理
在 `close` 中关闭的对象。

`transform` 方法中调用 `extractor.extract` 方法遍历读取的数据，并在转换后将数据通过 `loader.load` 写入
目标位置。

最后创建一个 `exceptions.py` 文件，内容如下：

```python
"""Exception"""


class EtlError(Exception):
    """Etl error"""


class PluginNotFoundError(EtlError):
    """PluginNotFoundError"""

    def __init__(self, namespace: str, name: str):
        super().__init__()
        self._namespace = namespace
        self._name = name

    def __repr__(self):
        return f'Can not found "{self._name}" plugin in {self._namespace}'

    def __str__(self):
        return self.__repr__()

```

在 `exceptions.py` 文件中首先创建了一个全局异常类 `EtlError` ， `PluginNotFoundError` 异常继承它。
当需要捕获所以项目异常时，可以通过 `EtlError` 捕获。

## 增加命令行调用

编辑 `cmdline.py` 文件，修改 `rum` 方法，内容如下：

```python
@main.command()
def run():
    """Run command"""
    init_log()
    manage = Manage()
    manage.run()

```

在使用命令 `example_etl` 调用时，可以通过传递 `run` 指令运行。
