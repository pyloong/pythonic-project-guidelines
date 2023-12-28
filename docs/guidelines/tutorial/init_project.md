# 初始化项目

在本章节，你讲学习到一下内容：

- 使用 cookiecutter 初始化一个项目结构
- 查看初始化项目里面的主要文件内容
- 对项目进行 git 初始化和内容提交
- 使用 poetry 初始化项目的 python 环境
- 使用 tox 自动化测试项目，检查初始化的项目有没有问题

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
project_name [My Project]: example-etl
project_slug [example_etl]: 
project_description [My Awesome Project!]: This is my first etl project.
author_name [Author]: test
author_email [test@example.com]: test@example.com
version [0.1.0]: 
Select python_version:
1 - 3.10
2 - 3.11
Choose from 1, 2 [1]: 
use_src_layout [y]: 
use_poetry [y]: 
use_docker [n]: 
Select ci_tools:
1 - none
2 - Gitlab
3 - Github
Choose from 1, 2, 3 [1]: 
init_skeleton [n]: y

```

然后使用 vscode 打开项目：

```bash
code example_etl
```

建议在项目开始的时候就初始化 git 仓库，并在后续及时提交功能修改。

```bash
git init
git config user.name test
git config user.email test@example.com
git commit -m "feat: init project."
```

### 项目内容

在 IDE 中查看项目，可以看到目录结构如下：

```text
❯ tree example_etl
example_etl
├── .editorconfig
├── .gitignore
├── .pre-commit-config.yaml
├── LICENSE
├── README.md
├── docs
│   └── development.md
├── pyproject.toml
├── src
│   └── example_etl
│       ├── __init__.py
│       ├── cmdline.py
│       ├── config
│       │   ├── __init__.py
│       │   └── settings.yml
│       └── log.py
├── tests
│   ├── __init__.py
│   ├── conftest.py
│   ├── settings.yml
│   ├── test_cmdline.py
│   ├── test_log.py
│   └── test_version.py
└── tox.ini

6 directories, 19 files
```

#### pyproject.toml

`pyproject.toml` 是项目的打包配置文件。文件前面 `tool.poetry` 中设置了项目的基本信息。 `tool.poetry.dependencies` 中配置了此项目的依赖库。

文件内容如下：

```toml
[tool.poetry]
name = "example_etl"
version = "0.1.0"
description = "This is my first etl project."
readme = "README.md"
authors = ["test <test@example.com>"]
license = "MIT"
classifiers = [
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3.10",
]

[tool.poetry.dependencies]
python = "^3.10"
dynaconf = "^3.1.12"
click = "^8.1.3"

[tool.poetry.group.dev.dependencies]
pylint = "^2.17.4"
isort = "^5.12.0"
pytest = "^7.3.1"
tox = "^4.5.2"
mkdocs = "^1.4.3"
mkdocs-material = "^8.5.11"
pytest-pylint = "^0.19.0"
pre-commit = "^3.3.2"

[tool.poetry.scripts]
example_etl = "example_etl.cmdline:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.pytest.ini_options]
testpaths = "tests"
python_files = "tests.py test_*.py *_tests.py"

[tool.pylint.design]
max-line-length = 120

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

优先级从从上到下依次增大，优先级高的会覆盖优先级低的配置。

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

项目使用 `poetry` 管理虚拟环境，运行命令自动创建虚拟环境，同时安装开发环境依赖

### 安装项目默认依赖

在项目目录执行 `poetry install -v` 以可视化过程安装依赖：

<!-- markdownlint-disable MD013 MD033-->
```bash
$ poetry install -v
Creating virtualenv example-etl-B-7RVLBy-py3.10 in /Users/kevin/Library/Caches/pypoetry/virtualenvs
Using virtualenv: /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10
Updating dependencies
Resolving dependencies... (7.5s)

Finding the necessary packages for the current system

Package operations: 52 installs, 1 update, 0 removals

  • Installing six (1.16.0)
  • Installing lazy-object-proxy (1.9.0)
  • Installing markupsafe (2.1.2)
  • Installing python-dateutil (2.8.2)
  • Installing pyyaml (6.0)
  • Installing typing-extensions (4.6.2)
  • Installing wrapt (1.15.0)
  • Installing astroid (2.15.5)
  • Installing certifi (2023.5.7)
  • Installing charset-normalizer (3.1.0)
  • Installing click (8.1.3)
  • Installing dill (0.3.6)
  • Installing filelock (3.12.0)
  • Installing exceptiongroup (1.1.1)
  • Installing ghp-import (2.1.0)
  • Installing idna (3.4)
  • Installing distlib (0.3.6)
  • Installing iniconfig (2.0.0)
  • Installing isort (5.12.0)
  • Installing jinja2 (3.1.2)
  • Installing markdown (3.3.7)
  • Installing mccabe (0.7.0)
  • Installing mergedeep (1.3.4)
  • Installing packaging (23.1)
  • Installing platformdirs (3.5.1)
  • Installing pyyaml-env-tag (0.1)
  • Installing pluggy (1.0.0)
  • Updating setuptools (67.7.2 -> 67.8.0)
  • Installing tomli (2.0.1)
  • Installing urllib3 (2.0.2)
  • Installing tomlkit (0.11.8)
  • Installing watchdog (3.0.0)
  • Installing cachetools (5.3.1)
  • Installing cfgv (3.3.1)
  • Installing chardet (5.1.0)
  • Installing colorama (0.4.6)
  • Installing identify (2.5.24)
  • Installing mkdocs (1.4.3)
  • Installing mkdocs-material-extensions (1.1.1)
  • Installing nodeenv (1.8.0)
  • Installing pygments (2.15.1)
  • Installing pylint (2.17.4)
  • Installing pymdown-extensions (10.0.1)
  • Installing pyproject-api (1.5.1)
  • Installing pytest (7.3.1)
  • Installing requests (2.31.0)
  • Installing toml (0.10.2)
  • Installing virtualenv (20.23.0)
  • Installing dynaconf (3.1.12)
  • Installing mkdocs-material (8.5.11)
  • Installing pre-commit (3.3.2)
  • Installing pytest-pylint (0.19.0)
  • Installing tox (4.5.2)

Writing lock file

Installing the current project: example_etl (0.1.0)
```

<!-- markdownlint-restore -->

然后执行 `poetry shell` 进入到虚拟环境。

在使用 vscode 的时候，可以运行 `Ctrl + Shift + p` 打开指令，输入 `> Python: Select Interpreter` 选择刚刚创建的虚拟环境。
如果看不到，只需要点击旁边的刷新按钮即可。然后重新打开一个新的终端，会自动进入虚拟环境。

## 运行测试

为了保证初始化项目是正常的，在进行后续步骤之前，运行自动化测试逻辑：

```bash
tox
```

可以看到最后输出如下：

<!-- markdownlint-disable MD013 MD033-->

```bash
$ tox
py310: install_deps> python -I -m pip install poetry
.pkg: install_requires> python -I -m pip install poetry-core
.pkg: _optional_hooks> python /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pyproject_api/_backend.py True poetry.core.masonry.api
.pkg: get_requires_for_build_sdist> python /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pyproject_api/_backend.py True poetry.core.masonry.api
.pkg: prepare_metadata_for_build_wheel> python /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pyproject_api/_backend.py True poetry.core.masonry.api
.pkg: build_sdist> python /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pyproject_api/_backend.py True poetry.core.masonry.api
py310: install_package_deps> python -I -m pip install 'click<9.0.0,>=8.1.3' 'dynaconf<4.0.0,>=3.1.12'
py310: install_package> python -I -m pip install --force-reinstall --no-deps /private/tmp/example_etl/.tox/.tmp/package/1/example_etl-0.1.0.tar.gz
py310: commands[0]> poetry install -v
Using virtualenv: /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10
Installing dependencies from lock file

Finding the necessary packages for the current system

Package operations: 0 installs, 0 updates, 0 removals, 53 skipped

  • Installing astroid (2.15.5): Skipped for the following reason: Already installed
  • Installing identify (2.5.24): Skipped for the following reason: Already installed
  • Installing chardet (5.1.0): Skipped for the following reason: Already installed
  • Installing charset-normalizer (3.1.0): Skipped for the following reason: Already installed
  • Installing click (8.1.3): Skipped for the following reason: Already installed
  • Installing colorama (0.4.6): Skipped for the following reason: Already installed
  • Installing distlib (0.3.6): Skipped for the following reason: Already installed
  • Installing isort (5.12.0): Skipped for the following reason: Already installed
  • Installing jinja2 (3.1.2): Skipped for the following reason: Already installed
  • Installing idna (3.4): Skipped for the following reason: Already installed
  • Installing certifi (2023.5.7): Skipped for the following reason: Already installed
  • Installing iniconfig (2.0.0): Skipped for the following reason: Already installed
  • Installing lazy-object-proxy (1.9.0): Skipped for the following reason: Already installed
  • Installing mkdocs-material (8.5.11): Skipped for the following reason: Already installed
  • Installing dynaconf (3.1.12): Skipped for the following reason: Already installed
  • Installing mkdocs-material-extensions (1.1.1): Skipped for the following reason: Already installed
  • Installing markdown (3.3.7): Skipped for the following reason: Already installed
  • Installing packaging (23.1): Skipped for the following reason: Already installed
  • Installing mergedeep (1.3.4): Skipped for the following reason: Already installed
  • Installing mkdocs (1.4.3): Skipped for the following reason: Already installed
  • Installing pre-commit (3.3.2): Skipped for the following reason: Already installed
  • Installing nodeenv (1.8.0): Skipped for the following reason: Already installed
  • Installing filelock (3.12.0): Skipped for the following reason: Already installed
  • Installing mccabe (0.7.0): Skipped for the following reason: Already installed
  • Installing markupsafe (2.1.2): Skipped for the following reason: Already installed
  • Installing pytest (7.3.1): Skipped for the following reason: Already installed
  • Installing pytest-pylint (0.19.0): Skipped for the following reason: Already installed
  • Installing ghp-import (2.1.0): Skipped for the following reason: Already installed
  • Installing pylint (2.17.4): Skipped for the following reason: Already installed
  • Installing pyyaml-env-tag (0.1): Skipped for the following reason: Already installed
  • Installing requests (2.31.0): Skipped for the following reason: Already installed
  • Installing dill (0.3.6): Skipped for the following reason: Already installed
  • Installing platformdirs (3.5.1): Skipped for the following reason: Already installed
  • Installing exceptiongroup (1.1.1): Skipped for the following reason: Already installed
  • Installing python-dateutil (2.8.2): Skipped for the following reason: Already installed
  • Installing pygments (2.15.1): Skipped for the following reason: Already installed
  • Installing pyproject-api (1.5.1): Skipped for the following reason: Already installed
  • Installing cfgv (3.3.1): Skipped for the following reason: Already installed
  • Installing six (1.16.0): Skipped for the following reason: Already installed
  • Installing virtualenv (20.23.0): Skipped for the following reason: Already installed
  • Installing pyyaml (6.0): Skipped for the following reason: Already installed
  • Installing cachetools (5.3.1): Skipped for the following reason: Already installed
  • Installing tomlkit (0.11.8): Skipped for the following reason: Already installed
  • Installing tox (4.5.2): Skipped for the following reason: Already installed
  • Installing urllib3 (2.0.2): Skipped for the following reason: Already installed
  • Installing setuptools (67.8.0): Skipped for the following reason: Already installed
  • Installing toml (0.10.2): Skipped for the following reason: Already installed
  • Installing wrapt (1.15.0): Skipped for the following reason: Already installed
  • Installing typing-extensions (4.6.2): Skipped for the following reason: Already installed
  • Installing pymdown-extensions (10.0.1): Skipped for the following reason: Already installed
  • Installing watchdog (3.0.0): Skipped for the following reason: Already installed
  • Installing tomli (2.0.1): Skipped for the following reason: Already installed
  • Installing pluggy (1.0.0): Skipped for the following reason: Already installed

Installing the current project: example_etl (0.1.0)
py310: commands[1]> poetry run pytest tests
================================================================================================================================= test session starts =================================================================================================================================
platform darwin -- Python 3.10.11, pytest-7.3.1, pluggy-1.0.0
cachedir: .tox/py310/.pytest_cache
rootdir: /private/tmp/example_etl
configfile: pyproject.toml
plugins: pylint-0.19.0
collected 10 items                                                                                                                                                                                                                                                                    

tests/test_cmdline.py .....                                                                                                                                                                                                                                                     [ 50%]
tests/test_exceptions.py .                                                                                                                                                                                                                                                      [ 50%]
tests/test_log.py .....                                                                                                                                                                                                                                                         [ 91%]
tests/test_version.py .                                                                                                                                                                                                                                                         [100%]


================================================================================================================================== warnings summary ===================================================================================================================================
../../../Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pkg_resources/__init__.py:121
  /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pkg_resources/__init__.py:121: DeprecationWarning: pkg_resources is deprecated as an API
    warnings.warn("pkg_resources is deprecated as an API", DeprecationWarning)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
============================================================================================================================ 10 passed, 1 warning in 0.01s ============================================================================================================================
py310: OK ✔ in 14.05 seconds
isort: install_deps> python -I -m pip install isort
isort: install_package_deps> python -I -m pip install 'click<9.0.0,>=8.1.3' 'dynaconf<4.0.0,>=3.1.12'
isort: install_package> python -I -m pip install --force-reinstall --no-deps /private/tmp/example_etl/.tox/.tmp/package/2/example_etl-0.1.0.tar.gz
isort: commands[0]> isort . --check-only --diff
Skipped 1 files
isort: OK ✔ in 3.05 seconds
pylint: install_deps> python -I -m pip install poetry
pylint: install_package_deps> python -I -m pip install 'click<9.0.0,>=8.1.3' 'dynaconf<4.0.0,>=3.1.12'
pylint: install_package> python -I -m pip install --force-reinstall --no-deps /private/tmp/example_etl/.tox/.tmp/package/3/example_etl-0.1.0.tar.gz
pylint: commands[0]> poetry install -v
Using virtualenv: /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10
Installing dependencies from lock file

Finding the necessary packages for the current system

Package operations: 0 installs, 0 updates, 0 removals, 53 skipped

  • Installing astroid (2.15.5): Skipped for the following reason: Already installed
  • Installing certifi (2023.5.7): Skipped for the following reason: Already installed
  • Installing cfgv (3.3.1): Skipped for the following reason: Already installed
  • Installing cachetools (5.3.1): Skipped for the following reason: Already installed
  • Installing dill (0.3.6): Skipped for the following reason: Already installed
  • Installing click (8.1.3): Skipped for the following reason: Already installed
  • Installing colorama (0.4.6): Skipped for the following reason: Already installed
  • Installing chardet (5.1.0): Skipped for the following reason: Already installed
  • Installing distlib (0.3.6): Skipped for the following reason: Already installed
  • Installing markdown (3.3.7): Skipped for the following reason: Already installed
  • Installing filelock (3.12.0): Skipped for the following reason: Already installed
  • Installing identify (2.5.24): Skipped for the following reason: Already installed
  • Installing idna (3.4): Skipped for the following reason: Already installed
  • Installing isort (5.12.0): Skipped for the following reason: Already installed
  • Installing charset-normalizer (3.1.0): Skipped for the following reason: Already installed
  • Installing dynaconf (3.1.12): Skipped for the following reason: Already installed
  • Installing markupsafe (2.1.2): Skipped for the following reason: Already installed
  • Installing ghp-import (2.1.0): Skipped for the following reason: Already installed
  • Installing mergedeep (1.3.4): Skipped for the following reason: Already installed
  • Installing iniconfig (2.0.0): Skipped for the following reason: Already installed
  • Installing mkdocs-material (8.5.11): Skipped for the following reason: Already installed
  • Installing nodeenv (1.8.0): Skipped for the following reason: Already installed
  • Installing exceptiongroup (1.1.1): Skipped for the following reason: Already installed
  • Installing pyproject-api (1.5.1): Skipped for the following reason: Already installed
  • Installing pluggy (1.0.0): Skipped for the following reason: Already installed
  • Installing python-dateutil (2.8.2): Skipped for the following reason: Already installed
  • Installing jinja2 (3.1.2): Skipped for the following reason: Already installed
  • Installing pyyaml-env-tag (0.1): Pending...
  • Installing pymdown-extensions (10.0.1): Skipped for the following reason: Already installed
  • Installing packaging (23.1): Skipped for the following reason: Already installed
  • Installing mccabe (0.7.0): Skipped for the following reason: Already installed
  • Installing pytest-pylint (0.19.0): Skipped for the following reason: Already installed
  • Installing pyyaml (6.0): Skipped for the following reason: Already installed
  • Installing pygments (2.15.1): Skipped for the following reason: Already installed
  • Installing pymdown-extensions (10.0.1): Skipped for the following reason: Already installed
  • Installing packaging (23.1): Skipped for the following reason: Already installed
  • Installing mccabe (0.7.0): Skipped for the following reason: Already installed
  • Installing pytest-pylint (0.19.0): Skipped for the following reason: Already installed
  • Installing pyyaml (6.0): Skipped for the following reason: Already installed
  • Installing pygments (2.15.1): Skipped for the following reason: Already installed
  • Installing pyyaml-env-tag (0.1): Skipped for the following reason: Already installed
  • Installing pymdown-extensions (10.0.1): Skipped for the following reason: Already installed
  • Installing packaging (23.1): Skipped for the following reason: Already installed
  • Installing mccabe (0.7.0): Skipped for the following reason: Already installed
  • Installing pytest-pylint (0.19.0): Skipped for the following reason: Already installed
  • Installing pyyaml (6.0): Skipped for the following reason: Already installed
  • Installing pygments (2.15.1): Skipped for the following reason: Already installed
  • Installing pylint (2.17.4): Skipped for the following reason: Already installed
  • Installing tox (4.5.2): Skipped for the following reason: Already installed
  • Installing setuptools (67.8.0): Skipped for the following reason: Already installed
  • Installing platformdirs (3.5.1): Skipped for the following reason: Already installed
  • Installing toml (0.10.2): Skipped for the following reason: Already installed
  • Installing watchdog (3.0.0): Skipped for the following reason: Already installed
  • Installing tomlkit (0.11.8): Skipped for the following reason: Already installed
  • Installing mkdocs-material-extensions (1.1.1): Skipped for the following reason: Already installed
  • Installing typing-extensions (4.6.2): Skipped for the following reason: Already installed
  • Installing urllib3 (2.0.2): Skipped for the following reason: Already installed
  • Installing lazy-object-proxy (1.9.0): Skipped for the following reason: Already installed
  • Installing six (1.16.0): Skipped for the following reason: Already installed
  • Installing tomli (2.0.1): Skipped for the following reason: Already installed
  • Installing mkdocs (1.4.3): Skipped for the following reason: Already installed
  • Installing requests (2.31.0): Skipped for the following reason: Already installed
  • Installing pytest (7.3.1): Skipped for the following reason: Already installed
  • Installing virtualenv (20.23.0): Skipped for the following reason: Already installed
  • Installing wrapt (1.15.0): Skipped for the following reason: Already installed
  • Installing pre-commit (3.3.2): Skipped for the following reason: Already installed

Installing the current project: example_etl (0.1.0)
pylint: commands[1]> poetry run pylint tests src

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

.pkg: _exit> python /Users/kevin/Library/Caches/pypoetry/virtualenvs/example-etl-B-7RVLBy-py3.10/lib/python3.10/site-packages/pyproject_api/_backend.py True poetry.core.masonry.api
  py310: OK (14.05=setup[9.47]+cmd[3.84,0.75] seconds)
  isort: OK (3.05=setup[2.67]+cmd[0.38] seconds)
  pylint: OK (12.92=setup[7.86]+cmd[2.91,2.15] seconds)
  congratulations :) (30.10 seconds)

```

<!-- markdownlint-restore -->

至此，项目环境初始化完成。一切正常。

## 提交代码

在开发时，需要养成及时提交代码的好习惯。

```bash
git add .
git commit -m "feat: init project env."
```
