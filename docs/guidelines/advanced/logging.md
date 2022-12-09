# Logging

在开发中，通常会使用 `print` 输出一些信息，或者诊断信息。在信息的完整性和信息格式上都不能简便且灵活控制。此时使用 `logging` 是个更好的选择，
而且也鼓励开发人员尽可能的优先选用打印日志的方式在控制台输出信息。

日志是对软件执行时所发生事件的一种追踪方式。软件开发人员对他们的代码添加日志调用，借此来指示某事件的发生。
一个事件通过一些包含变量数据的描述信息来描述（比如：每个事件发生时的数据都是不同的）。开发者还会区分事件的重要性，
重要性也被称为 **等级** 或 **严重性**。有一个好的日志实践，能让开发调试流程更顺畅，出现问题能更快速精准定位。

本文不会以最基础的方式讲述 Python logging 的使用，而是以当前总结的实践方式结合实际操作案例展示 Logging 的使用，所以在阅读文章钱，
你应该提前了解 [日志 HOWTO](https://docs.python.org/zh-cn/3/howto/logging.html) 和
 [Python 的日志记录工具](https://docs.python.org/zh-cn/3/library/logging.html) 两篇文档。

## 1. 简单使用

在一般开发中，对于临时开发的项目，可能为了快速完成任务，项目中大量使用了 `print` 将调试信息输出到控制台。
项目后期就会出现调试困难等问题。本节会提供在简单环境下快速使用日志方式。

### 1.1 单文件使用

对于单文件的使用，直接使用根日志对象即可。由于默认的日志级别为 `WARNING` ，所以需要使用更低级别的日志是无法显示的。

```python
"""Simple logging"""
import logging

logging.warning('I love you ~')
```

如果后续开发有要控制日志级别的需求，直接在开始初始化日志配置就可以了；

```python
"""Simple logging"""
import logging


logging.basicConfig(level=logging.DEBUG)

logging.warning('I love you ~')
logging.debug('I love you too ~')
```

当需要输出更详细的日志信息，如执行时间 、 日志级别 、 线程或进程信息，都可以很方便的控制。

```python
"""Simple logging"""
import logging

logging.basicConfig(
    level=logging.DEBUG,
    # format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    # format='%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(process)d %(thread)d - %(message)s',
    format='%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(process)d %(thread)d - %(pathname)s:%(lineno)d %(message)s',
    datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
)

logging.warning('I love you ~')
logging.debug('I love you too ~')

```

虽然使用 `print` 能更快速的在控制太输出想要看到的内容，但从上面的示例来看，直接使用默认的日志输出也是很方便的，唯一的区别可能
就是需要导入了。而使用日志的话，想要在后续增加输出更精确的信息就显得比较灵活。

日志格式所支持的字段请参考 [LogRecord 属性](https://docs.python.org/zh-cn/3/library/logging.html#logrecord-attributes) 。

这也是为什么建议优先使用日志的原因。

### 1.2 一般项目中使用

对于一般的项目，可能会有多个模块或者包，在每个模块中都初始化一下日志配置显然违反了 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 原则。
所以最好是有一个公共的日志模块，在该模块中初始化日志。

**log.py** ：

```python
"""Log config"""
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
)

logger = logging.getLogger()

```

在其他模块中直接导入 `from log import logger` 即可使用。

???+ node "提示"

    在 `log.py` 中初始化日志配置后，虽然可以在其他模块中直接使用 `logging` 打印日志，但如果启动脚本的时候没有导入该模块，
    该模块中的内容是不会加载的，也就是所日志配置并没有实际执行。

    此时可以重构代码：

    ```python
    """Log config"""
    import logging


    def config_logging() -> None:
        """Config logging"""
        logging.basicConfig(
            level=logging.DEBUG,
            format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
        )

    ```

    然后在项目实际执行之前调用方法初始化配置就可以了。

对于需要灵活控制日志配置的，可以将日志级别放到配置中，通过工厂方式初始化：

```python
"""Log config"""
import logging
from typing import Optional


def config_logging(
        level: Optional[int, str] = logging.DEBUG,
        log_format: Optional[str] = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
) -> None:
    """
    Config logging
    :param level:
    :param log_format:
    :return:
    """
    logging.basicConfig(
        level=level,
        format=log_format,
        datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
    )

```

在逻辑执行之前调用方法通过参数工厂化日志配置。

## 2. 通用实践

本节内容是在通用项目中可以使用的一般实践方法。

### 2.1 使用 `ini` 格式文件配置日志

在项目中新建一个 `log.ini` 的配置文件：

**log.ini** ：

```ini
[loggers]
keys = root,simpleExample

[handlers]
keys = consoleHandler

[formatters]
keys = simpleFormatter

[logger_root]
level = DEBUG
handlers = consoleHandler

[logger_simpleExample]
level = DEBUG
handlers = consoleHandler
qualname = simpleExample
propagate = 0

[handler_consoleHandler]
class = StreamHandler
level = DEBUG
formatter = simpleFormatter
args = (sys.stdout,)

[formatter_simpleFormatter]
format = %(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt =

```

新建一个 `log.py` 的包：

**log.py** ：

```python
from logging.config import fileConfig


def init_ini_log() -> None:
    fileConfig('log.ini')

```

最后在程序执行前，初始化日志配置。

logging 默认使用 [configparser](https://docs.python.org/zh-cn/3/library/configparser.html#module-configparser) 解析 `ini` 格式文件。
如果你想使用其他格式，如 `toml` 或者 `yaml` ，则需要自己手动读取并解析文件内容为 Dict 即可。

### 2.2 使用 `yaml` 格式文件配置日志

新建 `log.yml` 文件：

**log.yml** ：

```yaml
version: 1
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: simple
    stream: ext://sys.stdout
loggers:
  simpleExample:
    level: DEBUG
    handlers:
      - console
    propagate: false
root:
  level: DEBUG
  handlers:
    - console
```

[YAML](https://yaml.org/) 格式解析工具有三个，都在文档中有相应地址。在这里选用 [PyYAML](https://pyyaml.org/) 。

安装依赖：

```bash
pip install pyyaml
```

新建 `log.py` 模块：

**log.py** ：

```python
from logging.config import dictConfig
from yaml import load

try:
    from yaml import CLoader as Loader, CDumper as Dumper
except ImportError:
    from yaml import Loader, Dumper


def init_yml_log() -> None:
    with open('log.yml', mode='r') as obj:
        logging_config = load(obj, Loader=Loader)
    dictConfig(logging_config)

```

最后在程序执行前，初始化日志配置。

### 2.3 注意事项

#### 2.3.1 优化

根据文档中 [优化](https://docs.python.org/zh-cn/3/howto/logging.html#optimization) 一节内容描述，日志中的参数化消息，
应该延迟加载。这么做是为了减少在计算日志参数时所消耗的资源，因为如果日志记录非丢弃，则不需要消耗这部分资源。所以在
日志记录上，应采用 `%` 的方式，而不是其他字符串格式化。

关于性能的讨论可以参考 [W1202 - logging-fstring-interpolation is not useful](https://github.com/PyCQA/pylint/issues/2395) 。
