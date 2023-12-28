# 配置

配置是一个项目的核心驱动，可以在不更改源代码或减少源代码修改的情况下快速调整项目的运行。
使用中心配置驱动项目，能让项目的使用更加灵活，运维工作更轻松。

例如 Django 框架会自带一个 `settings.py` 文件，在 `settings.py` 中的配置项都会覆盖框架
级别的默认配置，方便用户自定义修改。在代码中，可以使用 `django.settings` 对象获取所有
配置项。 Scrapy 框架同样也有这种机制。

这些成熟的框架增加了中心配置，就是为了通过开放出来的配置项来灵活控制框架所支持的内容。
而在一般项目中，也可以参照这种设计，让项目部署更加灵活。

## 1. 一般做法

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

## 2. 动态配置示例

Dynaconf 是一个灵活的中心配置管理工具，底层设计和 Django 一致，会延迟加载配置。

其具有如下特点：

- 加载多个配置源
- 配置分层
- Django Flask 扩展
- 支持 Redis 和 Vault

在项目中新建配置文件 `settings.yml`

**settings.yml** ：

```yaml
foo: 1
bar: 2

```

新建配置模块 `config.py`

**config.py** ：

```python
from dynaconf import Dynaconf

settings = Dynaconf(
    settings_files=['settings.yml'],
)

```

新建一个 `app.py` 文件，使用配置

**app.py** ：

```python
from config import settings

print(settings.FOO)
print(settings.BAR)

```

然后运行 `python app.py` 可以看到已经能够自动获取 `settings.yml` 配置文件中的值。

增加本地配置文件 `settings.local.yml`

**settings.local.yml** :

```yml
foo: 10
bar: 20

```

再次运行 `python app.py` ，程序会自动获取 `settings.local.yml` 。

这是因为 `Dynaconf` 在初始化是传入了配置文件格式为 `settings.yml` ，在加载配置时，会同时查找 `settings.local.yml` 的配置文件。
并将两个配置文件的内容合并，如果存在相同变量， `settings.local.yml` 会覆盖 `settings.yml` 中的配置。

## 项目实践

### Django 项目

Dynaconf 可以搭配 Django 一起使用。虽然 Django 有自己的配置模块，但是并不灵活。

搭配 Dynaconf ，可以启动层级配置，例如支持 `Dev` 、 `prod` 和 `test` 多种环境的配置，而且可以通过环境变量很方便的
修改配置，包括加载其他地方的配置。


在 Django 项目的 `settings.py` 文件最后添加如下内容：

```python
import dynaconf  # pylint: disable=wrong-import-position

settings = dynaconf.DjangoDynaconf(
    __name__,
    envvar_prefix='BLOG',
    settings_files=[
        BASE_DIR / 'settings.local.yml'
    ],
    environments=False,
    load_dotenv=True,
    ENVVAR_FOR_DYNACONF='BLOG_SETTINGS',
    includes=[
        Path(sys.prefix, 'etc', 'blog', 'settings.yml'),
    ]
)
```

当 Django 加载 `settings.py` 模块的时候，会初始化 Dynaconf 。 Dynaconf 会将 Django 的 `settings` 对象中的配置加载到 Dynaconf 中，
然后将自身的所有配置再重新加载到 Django 的 `settings` 对象中。

Dynaconf 不仅会加载配置文件，也会加载以 `BLOG_` 开头的环境变量。


#### 使用配置文件

在初始化 Dynaconf 时，会加载项目根目录的 `settings.local.yml` 配置文件，此文件一般是开发时使用的本地配置，并且不应该被 Git 追踪。
在不同的开发人员使用或者不同的环境中，可以使用多样化的本地配置。对于需要统一的默认配置，直接放在 `settings.py` 中就可以了。

同时还会读取 `<sys.prefix>/etc/blog/settings.yml` 。这个一般作为生产环境的系统配置。如果使用的是系统 Python 环境，可能目录是在
`/usr/local/etc/blog/settings.yml` ，如果是虚拟环境，则可能是 `/home/foo/.virtualenvs/blog-fxage/etc/blog/settings.yml` 。

如果想自定义配置文件位置，可以通过设置环境变量 `BLOG_SETTINGS=/tmp/settings.yml` 指定 Dynaconf 加载文件的位置。

```yaml
DEBUG: true

ALLOWED_HOSTS:
  - '*'

INSTALLED_APPS:
 - dynaconf_merge_unique   # 指示 Dynaconf 将 INSTALLED_APPS 与默认配置合并而不是覆盖，并且进行去重
 - debug_toolbar    # 指示 Dynaconf 将 debug_toolbar 添加到 INSTALLED_APPS 列表中

MIDDLEWARE:
 - dynaconf_merge_unique
 - debug_toolbar.middleware.DebugToolbarMiddleware

DATABASES:
  default:
    ENGINE: 'django.db.backends.mysql'
    NAME: blo
    USER: root
    PASSWORD: '000000'
    HOST: 127.0.0.1
    PORT: 3306

REST_FRAMEWORK:
  # 指示 Dynaconf 将 REST_FRAMEWORK 与默认配置合并，而不是覆盖
  dynaconf_merge_unique: true
  # 指示 Dynaconf 只修改 PAGE_SIZE 的值，其他不变
  PAGE_SIZE: 10
}
```

上述配置在 Dynaconf 读取后，可以覆盖 `settings.py` 中的默认配置。其中有几个点需要注意：

- 如果直接文件中定义配置，会覆盖默认配置。
- 如果需要和默认配置合并，可以使用 `dynaconf_merge` 。

#### 使用环境变量

Dynaconf 支持加载环境变量，也可以使用 .env 文件。

在使用环境变量时，同样和配置文件一样，支持完全覆盖，和自动合并。

需要额外强调一点的是， Dynaconf 初始化的时候，使用了 `envvar_prefix=BLOG` 。 Dynaconf 会自动加载以 `BLOG_` 开头的
环境变量。包括 `ENVVAR_FOR_DYNACONF='BLOG_SETTINGS'` 配置的 Dynaconf 加载配置文件的环境变量 `BLOG_SETTINGS` 。

所以在使用环境变量的时候，不要错误的将 `BLOG_SETTINGS` 环境变量指定其他内容，而造成不必要的错误。

```shell
# 使用环境变量配置单值
export BLOG_DEBUG='True'

# 使用环境变量配置对象
export BLOG_DATABASES="{'default'={'ENGINE'='django.db.backends.mysql', 'NAME'='blog', 'USER'='root', 'PASSWORD'='000000', 'HOST'='localhost', 'POST'=3306}}"

# 使用环境变量配置合并内容
export BLOG_MIDDLEWARE='["dynaconf_merge_unique", "debug_toolbar.middleware.DebugToolbarMiddleware"]'   # 使用 dynaconf_merge_unique 合并并去重
export BLOG_MIDDLEWARE='@merge ["debug_toolbar.middleware.DebugToolbarMiddleware"]' # 使用 merge 关键字
export BLOG_MIDDLEWARE='@merge debug_toolbar.middleware.DebugToolbarMiddleware' # 简写

export BLOG_REST_FRAMEWORK='{PAGE_SIZE=10, dynaconf_merge=true}'    # 使用 dynaconf_merge 合并
export BLOG_REST_FRAMEWORK='@merge {PAGE_SIZE=10}'  # 使用 merge 关键字
export BLOG_REST_FRAMEWORK='@merge PAGE_SIZE=10'    # 简写

export BLOG_DATABASES__default__PASSWORD='123456'   # 使用两个下划线 (__) 作为子级
```
