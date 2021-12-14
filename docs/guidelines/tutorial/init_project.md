# 初始化项目

初始化项目时，使用 cookiecutter 加载 [项目模板](https://github.com/pyloong/cookiecutter-pythonic-project) 创建。
通过交互操作，可以选择使用的功能。

## 创建项目骨架

在终端运行命令：

```bash
cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project
```

然后根据交互提示，选择需要的内容。最终输入如下：

```text
❯ cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project
You've downloaded /home/kevin/.cookiecutters/cookiecutter-pythonic-project before. Is it okay to delete and re-download it? [yes]: yes
project_name [My Project]: example-etl
project_slug [example_etl]: 
project_description [My Awesome Project!]: Example etl tools
author_name [Author]: huagang
author_email [huagang@example.com]: huagang517@126.com
version [0.1.0]: 
Select python_version:
1 - 3.7
2 - 3.8
3 - 3.9
4 - 3.10
Choose from 1, 2, 3, 4 [1]: 4
use_src_layout [y]: y
use_pipenv [y]: y
Select index_server:
1 - none
2 - aliyun
Choose from 1, 2 [1]: 
use_docker [n]: y
Select ci_tools:
1 - none
2 - Gitlab
3 - Github
Choose from 1, 2, 3 [1]: 3
init_skeleton [n]: y

```

然后使用 vscode 打开项目：

```bash
code example_etl
```

### 项目内容

在 IDE 中查看项目，可以看到目录结构如下：

```text
❯ tree example_etl
example_etl
├── Dockerfile
├── docs
│   └── development.md
├── LICENSE
├── MANIFEST.in
├── Pipfile
├── pyproject.toml
├── README.md
├── setup.cfg
├── src
│   └── example_etl
│       ├── cmdline.py
│       ├── config
│       │   ├── __init__.py
│       │   └── settings.yml
│       ├── __init__.py
│       └── log.py
├── tests
│   ├── conftest.py
│   ├── __init__.py
│   ├── test_cmdline.py
│   ├── test_log.py
│   └── tests.py
└── tox.ini

5 directories, 19 files

```

#### setup.cfg

`setup.cfg` 是项目的打包配置文件。文件前面 `metadata` 中设置了项目的基本信息。 `options` 中配置了此项目必须使用的一些信息。
最后几行还包含了 `pytest` 的配置，这是因为他们可以共用这些配置。

文件内容如下：

```ini
[metadata]
name = example_etl
version = attr: example_etl.__version__
author = huagang
author_email = huagang517@126.com
description = Example etl tools
keywords = example-etl
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.10

[options]
python_requires > = 3.10
include_package_data = True
packages = find:
package_dir =
    = src
install_requires =
    dynaconf
    click

[options.packages.find]
where = src
exclude =
    tests*
    docs

# https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html
[options.entry_points]
console_scripts =
    example_etl = example_etl.cmdline:main

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

[tool:pytest]
testpaths = tests
python_files = tests.py test_*.py
```

#### pyproject.toml

`pyproject.toml` 是 PEP 517 的打包规范所需要的配置文件，用来指示使用哪种打包工具。

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

```

#### MANIFEST.in

`MANIFEST.in` 指示打包时需要包含或排除的文件。可以通过正则等配置。

```text
include LICENSE
include MANIFEST.in
include *.md
graft docs
graft tests
graft src/example_etl
recursive-include src/example_etl/config/settings.yml
global-exclude __pycache__
global-exclude *.py[co]
```

#### src/example_etl/cmdline.py

`src/example_etl/cmdline.py` 是使用 `click` 编写的一个命令行入口文件，通过一些自定义命令和参数来控制程序的逻辑。

```python
"""Command line"""
import click
from click import Context

from example_etl import __version__
from example_etl.config import settings
from example_etl.log import init_log


@click.group(invoke_without_command=True)
@click.pass_context
@click.option(
    '-V',
    '--version',
    is_flag=True,
    help='Show version and exit.'
)  # If it's true, it will override `settings.VERBOSE`
@click.option('-v', '--verbose', is_flag=True, help='Show more info.')
@click.option(
    '--debug',
    is_flag=True,
    help='Enable debug.'
)  # If it's true, it will override `settings.DEBUG`
def main(ctx: Context, version: str, verbose: bool, debug: bool):
    """Main commands"""
    if version:
        click.echo(__version__)
    elif ctx.invoked_subcommand is None:
        click.echo(ctx.get_help())
    else:
        if verbose:
            settings.set('VERBOSE', True)
        if debug:
            settings.set('DEBUG', True)


@main.command()
def run():
    """Run command"""
    init_log()
    click.echo('run......')

```

#### src/example_etl/log.py

`src/example_etl/log.py` 是预定义日志配置文件，当项目启动时，会自动初始化默认的日志配置。

```python
"""Log"""
import logging
import os
from logging.config import dictConfig

from example_etl.config import settings

os.makedirs(settings.LOGPATH, exist_ok=True)


def verbose_formatter(verbose: int) -> str:
    """formatter factory"""
    if verbose is True:
        return 'verbose'
    return 'simple'


def update_log_level(debug: bool, level: str) -> str:
    """update log level"""
    if debug is True:
        level_num = logging.DEBUG
    else:
        level_num = logging.getLevelName(level)
    settings.set('LOGLEVEL', logging.getLevelName(level_num))
    return settings.LOGLEVEL


def init_log() -> None:
    """Init log config."""
    log_level = update_log_level(settings.DEBUG, str(settings.LOGLEVEL).upper())

    log_config = {
        "version": 1,
        "disable_existing_loggers": False,
        "formatters": {
            'verbose': {
                'format': '%(asctime)s %(levelname)s %(name)s %(process)d %(thread)d %(message)s',
            },
            'simple': {
                'format': '%(asctime)s %(levelname)s %(name)s %(message)s',
            },
        },
        "handlers": {
            "console": {
                "formatter": verbose_formatter(settings.VERBOSE),
                'level': 'DEBUG',
                "class": "logging.StreamHandler",
            },
            'file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'level': 'DEBUG',
                'formatter': verbose_formatter(settings.VERBOSE),
                'filename': os.path.join(settings.LOGPATH, 'all.log'),
                'maxBytes': 1024 * 1024 * 1024 * 200,  # 200M
                'backupCount': '5',
                'encoding': 'utf-8'
            },
        },
        "loggers": {
            '': {'level': log_level, 'handlers': ['console']},
        }
    }

    dictConfig(log_config)

```

#### src/example_etl/config/\_\_init\_\_.py

`src/example_etl/config/__init__.py` 是使用 `dynaconf` 初始化的配置中心，项目所有的配置都是
从 `settings` 对象中获取，它会读取项目级别的默认配置文件，也会读取自定义配置文件。

默认情况下，加载的配置文件如下：

- `src/example_etl/config/settings.yml` ：项目默认配置文件
- `src/example_etl/config/settings.local.yml` ：这个在项目中是不会 git 追踪的，属于本地自定义配置
- `<sys.prefix>/etc/example_etl/settings.yml` ：操作系统外部配置文件。默认这个配置文件和项目默认配置文件的内容一致。
- 使用 `EXAMPLE_ETL_<name>=<value>` 环境变量传递

优先级从从上倒下依次增大，优先级高的会覆盖优先级低的配置。

```python
"""
Configuration center.
Use https://www.dynaconf.com/
"""""
import os
import sys
from pathlib import Path

from dynaconf import Dynaconf


_base_dir = Path(__file__).parent.parent

_settings_files = [
    # All config file will merge.
    Path(__file__).parent / 'settings.yml',  # Load default config.
]

# User configuration. It will be created automatically by the pip installer .
_external_files = [
    Path(sys.prefix, 'etc', 'example_etl', 'settings.yml')
]


settings = Dynaconf(
    # Set env `EXAMPLE_ETL_FOO='bar'`，use `settings.FOO` .
    envvar_prefix='EXAMPLE_ETL',
    settings_files=_settings_files,  # load user configuration.
    # environments=True,  # Enable multi-level configuration，eg: default, development, production
    load_dotenv=True,  # Enable load .env
    # env_switcher='EXAMPLE_ETL_ENV',
    lowercase_read=False,  # If true, can't use `settings.foo`, but can only use `settings.FOO`
    includes=_external_files,  # Customs settings.
    base_dir=_base_dir,  # `settings.BASE_DIR`
)

```

## 初始化环境

项目使用 `pipenv` 管理虚拟环境，运行命令自动创建虚拟环境，同时安装开发环境依赖

```bash
pipenv install -d
```

输入如下：

```text
❯ pipenv install -d
Creating a virtualenv for this project...
Pipfile: /tmp/test/example_etl/Pipfile
Using /usr/local/bin/python3.10 (3.10.0) to create virtualenv...
⠸ Creating virtual environment...created virtual environment CPython3.10.0.final.0-64 in 201ms
  creator CPython3Posix(dest=/home/kevin/.virtualenvs/example_etl-H6lUq1jb, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/kevin/.local/share/virtualenv)
    added seed packages: pip==21.3.1, setuptools==58.5.3, wheel==0.37.0
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator

✔ Successfully created virtual environment! 
Virtualenv location: /home/kevin/.virtualenvs/example_etl-H6lUq1jb
Installing dependencies from Pipfile.lock (96296b)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 26/26 — 00:00:14
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

然后执行 `pipenv shell` 进入到虚拟环境。

在使用 vscode 的时候，可以运行 `Ctrl + Shift + p` 打开指令，输入 `> Python: Select Interpreter` 选择刚刚创建的虚拟环境。
如果看不到，只需要点击旁边的刷新按钮即可。然后重新打开一个新的终端，会自动进入虚拟环境。

## 运行测试

为了保证初始化项目是正常的，在进行后续步骤之前，运行自动化测试逻辑：

```bash
tox
```

可以看到最后输出如下：

```text
_______________________________________________________________________ summary _______________________________________________________________________
  py310: commands succeeded
  isort: commands succeeded
  pylint: commands succeeded
  congratulations :)
```

即一切正常。
