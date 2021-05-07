# 配置

配置是一个项目的核心驱动，可以在不更改源代码或减少源代码修改的情况下快速调整项目的运行。
使用中心配置驱动项目，能让项目的使用更加灵活，运维工作更轻松。

例如 Django 框架会自带一个 `settings.py` 文件，在 `settings.py` 中的配置项都会覆盖框架
级别的默认配置，方便用户自定义修改。在代码中，可以使用 `django.settings` 对象获取所有
配置项。 Scrapy 框架同样也有这种机制。

这些成熟的框架增加了中心配置，就是为了通过开放出来的配置项来灵活控制框架所支持的内容。
而在一般项目中，也可以参照这种设计，让项目部署更加灵活。

## 一般做法

常见增加中心配置的做法是在项目中增加一个 `settings.py` 文件，该文件中模块级别常量定义配置项。
在使用时，通过导入模块中的内容使用。

例如：

在项目中创建一个 `settings.py` 文件，在文件中定义模块级常量

```python
## Settings

# File config
SOURCE_FILE = '/tmp/foo.txt'

# Log config
LOG_LEVEL = 'DEBUG'
LOG_FORMAT = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
```

创建 `app.py` 文件，在文件中导入 `settings` 模块，并使用该模块中的常量。

```python
"""Count a file """
import logging
from pathlib import Path  

import settings


# Config root logger
logging.basicConfig(
    level=settings.LOG_LEVEL,
    format=settings.LOG_FORMAT,
)


def count_word(source_file: Path) -> None:
    """
    :param source_file:
    :return:    None
    """
    total_words = 0
    # Read source_file
    logging.debug('Read file: %s', source_file)
    with open(source_file, mode='r', encoding='utf-8') as source_obj:
        for line in source_obj.readlines():
            total_words += len(line.split(' '))
    logging.info('File has %s words', total_words)


def main():
    count_word(Path(settings.SOURCE_FILE))


if __name__ == '__main__':
    main()

```

## 动态配置

Dynaconf 是一个灵活的中心配置管理工具，底层设计和 Django 一致，会延迟加载配置。

其具有如下特点：

- 加载多个配置源
- 配置分层
- Django Flask 扩展
- 支持 Redis 和 Vault

```python

```
