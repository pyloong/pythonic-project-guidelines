# 快速上手

这是一个快速上手的开发指南，本文通过一个包含主要知识点的简单项目，向开发者展示一个更符合 Python 规范和风格（Pythonic）的项目开发流程。

示例项目是一个单词统计的演示程序，如果你想查看完整示例，可以浏览 [Word Count](https://github.com/pyloong/pythonic-project-samples/tree/feature/word_count) 项目源码。

## 1. 开发环境搭建

### 1.1 Python 开发环境

本项目使用 Python 3.10 。具体版本的 Python 环境可以在[官网](https://www.python.org/downloads/)下载。

### 1.2 开发工具

推荐使用 [Pycharm](https://www.jetbrains.com/pycharm/) 开发工具，可以选择免费的社区版本。

[Visual Studio Code](https://code.visualstudio.com/) 是微软开发的一款免费轻量级文本编辑器，通过安装插件可以自定义成一款功能强大的 IDE 开发工具。目前支持 Python 的插件体系已经较为完善，此方案也可以作为备用。

### 1.3 虚拟环境工具

推荐使用 [Poetry](https://python-poetry.org/) ，既包含了虚拟环境管理工具也支持打包发布等功能。

在安装好 Python 环境后，应该在全局环境中安装 [Poetry](https://python-poetry.org/) 。

```bash
sudo python -m pip install -U pip
sudo pip install -U poetry
```

### 1.4 初始化项目

[cookiecutter](https://cookiecutter.readthedocs.io/en/1.7.2/README.html) 是一个通过项目模板创建项目的命令行工具。

安装 cookiecutter

```bash
sudo pip3 install -U cookiecutter
```

初始化项目

```bash
cd workspace
cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project
```

运行命令后会出现下面的配置过程，如果你不清楚配置的具体用途，可以直接按回车使用默认配置，默认配置使用项目模板初始值。

```bash
❯ cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project
project_name [My Project]: Word Count
project_slug [word_count]: 
project_description [My Awesome Project!]: Word Count Project.
author_name [Author]: test
author_email [author@example.com]: test@example.com
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
init_skeleton [n]:

```

如果你在使用项目模板过程中有任何问题或疑问，可以通过发起 [issues](https://github.com/pyloong/cookiecutter-pythonic-project/issues) 进行反馈。

生成后的项目结构如下：

```txt
word_count
├── .editorconfig
├── .gitignore
├── .pre-commit-config.yaml
├── LICENSE
├── README.md
├── docs
│   └── development.md
├── pyproject.toml
├── src
│   └── word_count
│       └── __init__.py
├── tests
│   ├── __init__.py
│   ├── conftest.py
│   ├── settings.yml
│   └── test_version.py
└── tox.ini

5 directories, 13 files
```

生成项目的 `src` 目录下有一个项目模块，用来存放项目源代码， `tests` 目录用来编写模块的相关测试代码。

`pyproject.toml` 包含项目初始依赖，和项目的描述信息，`tox.ini` 定义了任务自动化执行逻辑。

### 1.5 初始化项目环境

使用 poetry 初始化一个虚拟环境。

```bash
cd word_count
poetry install -v
```

初始化完成后，会生成一个 `poetry.lock`，可以用来锁定生产环境安装包的版本和依赖信息。

### 1.6 初始化 Git

推荐使用 [Git](https://git-scm.com/) 对项目进行版本管理。所以需要提前安装 Git ，并熟悉常用的 Git 概念和 Git 命令。

```bash
git init
git config user.name test
git config user.email test@example.com

# 初始化项目提交
git add .
git commit -m "feat: 初始化项目提交"
```

### 1.7 会用到的其他工具

在生成的 `pyproject.toml` 文件中，默认添加了一些开发环境中常用的工具。

- `isort`: [isort](https://pycqa.github.io/isort/) 是一个自动格式化导入工具
- `pylint`: [pylint](https://www.pylint.org/) 是一个检测代码风格工具
- `pytest`: [pytest](https://docs.pytest.org/en/stable/) 是一个更加易用的测试框架，兼容 `unittest` 测试框架
- `pytest-cov`: [pytest-cov](https://github.com/pytest-dev/pytest-cov) 是 `pytest` 的 [Coverage](https://coverage.readthedocs.io/en/latest/) 插件，用来统计测试覆盖率
- `mkdocs`: [mkdocs](https://www.mkdocs.org/) 是一个项目文档构建工具，使用 markdown 编写内容，构建生成文档页面。
- `mkdocs-material`: [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) 是基于 [mkdocs](https://www.mkdocs.org/) 构建文档，并提供现代化主题的库。
- `tox`: [tox](https://tox.readthedocs.io/en/latest/) 是一个任务自动化工具

如果想要了解相关的功能，可以阅读对应的技术说明文档。

## 2. 功能开发

首先将项目以可编辑方式安装到环境中：

```bash
poetry install
```

这样做的目的是将 `src` 下的包安装到 Python 环境中，否则无法正常导入包中的模块。

### 2.1 功能需求

提供一个从文本文件读取数据，数据以空格分割单词，然后统计文件中的单词数量，并将结果写入到目标文件中。

### 2.2 编写计数器

在 `src/word_count/` 下创建 `counter.py` 文件，同时加入如下内容：

```python
"""Count a file """
import logging
from collections.abc import Generator
from pathlib import Path

# Config root logger
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)


def count(source_file: str, dest_file: str):
    """
    Count source
    :param source_file:
    :param dest_file:
    :return:
    """
    words = read_from_file(Path(source_file))

    total = 0

    for _ in words:
        total += 1

    write_to_file(Path(dest_file), total)


def read_from_file(source_file: Path) -> Generator[str, None, None]:
    """

    :param source_file:
    :return:
    """
    # Read source_file
    logging.debug('Read file: %s', source_file)
    with open(source_file, 'r', encoding='utf-8') as source_obj:
        for line in source_obj:
            for word in line.split(' '):
                yield word


def write_to_file(dest_file: Path, total_words: int) -> None:
    """
    Write result to file
    :param dest_file:
    :param total_words:
    :return:
    """
    logging.debug('Count %s words, write to %d', dest_file, total_words)
    with open(dest_file, 'w', encoding='utf-8') as dest_obj:
        dest_obj.write(f'Total count: {total_words}')

```

#### 2.2.1 导入格式化

在项目根目录运行 [isort](https://pycqa.github.io/isort/) 对导入进行格式化。

```bash
isort .
```

此操作会自动修改代码，将导入的包格式化。如果想查看区别，可以运行如下命令：

```bash
isort . --check-only --diff
```

#### 2.2.2 代码风格检查

在项目根目录运行 [pylint](https://www.pylint.org/) 检查代码是否规范，是否符合 [PEP8](https://www.python.org/dev/peps/pep-0008/) 标准。

```bash
pylint tests src
```

此操作会列出代码中不符合规范的部分，并显示对应的规范名称。可以在[这里](https://pylint.readthedocs.io/en/latest/technical_reference/features.html)找到所有规则。

在完成修改后再次运行两个命令，直到都没有异常输出为止。

#### 2.2.3 测试

> 如果你使用的是 Pycharm 开发，可以通过点击  `File` --> `Settings` --> `Tools` --> `Python Integrated Tools` --> `Testing` --> `Default runner` 选择测试框架，推荐使用 `pytest`。

为了方便使用 `mock` 需要安装 `pytest-mock` 模块，可以在 `pytest` 的 `fixture` 特性上使用 `mock`。

安装开发环境依赖：

```bash
poetry add --group dev pytest-mock
```

添加测试配置，在 `tests/conftest.py` 中加入：

```python
"""Test config"""
from pathlib import Path
from tempfile import TemporaryDirectory

import pytest


@pytest.fixture
def mock_path() -> Path:
    """Mock a path, and clean when unit test done."""
    with TemporaryDirectory() as temp_path:
        yield Path(temp_path)

```

在 `tests/` 下添加与 `src/word_count` 目录中文件名相同的文件，并在文件名前添加 `test_` 前缀。

添加文件 `tests/test_counter.py`：

```python
"""Test counter"""
from pathlib import Path

import pytest

from word_count.counter import count, read_from_file, write_to_file


@pytest.fixture(name='mock_source_file')
def fixture_mock_source_file(mock_path) -> Path:
    """mock source_file, this file has two words."""
    words = ['hello', ' ', 'words']
    source_file = mock_path / 'source.txt'
    with open(source_file, 'w', encoding='utf-8') as obj:
        obj.write(''.join(words))
    yield source_file


def test_read_from_file(mock_source_file):
    """Test read_from_file"""
    result = read_from_file(mock_source_file)
    assert sum(1 for _ in result) == 2


def test_write_to_file(mock_path):
    """Test write_to_file"""
    dest_file = mock_path / 'dest.txt'
    write_to_file(dest_file, 100)
    with open(dest_file, 'r', encoding='utf-8') as obj:
        txt = obj.read()
        assert 'Total count: 100' in txt


def test_count(mocker, mock_path, mock_source_file):
    """Test count"""
    mock_read_from_file = mocker.patch(
        'word_count.counter.read_from_file',
        return_value=list(range(10))
    )
    mock_write_to_file = mocker.patch(
        'word_count.counter.write_to_file'
    )
    dest_file = mock_path / 'dest.txt'
    count(str(mock_source_file), str(dest_file))
    mock_read_from_file.assert_called_once_with(mock_source_file)
    mock_write_to_file.assert_called_once_with(dest_file, 10)

```

运行 `pytest` ，让测试正确运行。如果测试用例失败，需要根据出错堆栈找到问题原因，解决掉后再次运行测试命令，直到代码测试通过。

然后运行 `isort` 和 `pylint src tests` 格式化代码并检查代码风格。

#### 2.2.4 提交代码

一个功能特性开发完成后，需要提交代码来保存记录，避免意外操作。

```bash
git add .
git commit -m "feat(counter): 增加 Counter 逻辑，并完成测试。"
```

### 2.3 编写命令行入口

在 `src/word_count/` 目录下，创建 `cmdline.py` 文件，加入如下内容：

```python
"""Cmdline"""
import argparse
import sys

from word_count.counter import count


def init_args() -> argparse.Namespace:
    """Init argument and parse"""
    parser = argparse.ArgumentParser()
    parser.add_argument('-s', '--source', required=True, help='Source file used for count.')
    parser.add_argument('-d', '--dest', required=True, help='Dest file used for count result')
    return parser.parse_args(sys.argv[1:])


def main():
    """Execute"""
    args = init_args()
    count(args.source, args.dest)


if __name__ == '__main__':
    main()

```

运行 `isort` 和 `pylint` 格式化代码并检查代码风格。

#### 2.3.1 测试

创建 `tests/test_cmdline.py` 文件，加入如下内容：

```python
"""Test cmdline"""
import sys

import pytest

from word_count import cmdline


def test_help(mocker, capsys):
    """test help command"""
    args = ['word_count', '-h']
    mocker.patch.object(sys, 'argv', args)
    with pytest.raises(SystemExit) as ex:
        cmdline.main()

    assert ex.value.code == 0
    outerr = capsys.readouterr()
    assert '-s SOURCE' in outerr.out
    assert '-d DEST' in outerr.out

def test_only_pass_source(mocker, capsys):
    """test only pass -s """
    args = ['word_count', '-s', 'foo']
    mocker.patch.object(sys, 'argv', args)
    with pytest.raises(SystemExit) as ex:
        cmdline.main()

    assert ex.value.code == 2
    outerr = capsys.readouterr()
    assert 'the following arguments are required: -d' in outerr.err

def test_only_pass_dest(mocker, capsys):
    """test only pass -d"""
    args = ['word_count', '-d', 'foo']
    mocker.patch.object(sys, 'argv', args)
    with pytest.raises(SystemExit) as ex:
        cmdline.main()

    assert ex.value.code == 2
    outerr = capsys.readouterr()
    assert 'the following arguments are required: -s' in outerr.err

def test_main(mocker):
    """test cmdline, and everything is fine."""
    args = ['word_count', '-s', 'foo', '-d', 'bar']
    mocker.patch.object(sys, 'argv', args)
    mock_count = mocker.patch('word_count.cmdline.count')
    cmdline.main()
    mock_count.assert_called_once()

```

运行 `pytest`，让测试正确运行。

运行 `isort` 和 `pylint` 格式化代码并检查代码风格。

#### 2.3.2 提交代码

```bash
git add .
git commit -m "feat(cmdline): 增加 cmdline 逻辑，并完成测试。"
```

### 2.4 总结

至此，我们的功能已经开发完成。在整个开发过程中，我们遵循了 “添加功能特性” => “代码风格检查” => “单元测试” 的开发流程。

如果感觉每次运行多个命令比较繁琐，可以在项目根目录中运行 `tox` 自动化完成代码测试、导包检查和代码风格检查。

```bash
tox
```

现在可以在终端中运行单词统计：

```bash
python src/word_count/cmdline.py -s LICENSE -d /tmp/res.txt
```

### 2.5 打包发布

如果希望别人能更方便的使用项目，可以将项目打包发布到 [pypi](https://pypi.org/) 中，然后在需要使用的地方运行 `pip install -U word-count`。

但是安装到环境后去运行 `cmdline.py` 会比较麻烦，所以需要将 `cmdline.py` 注册成可执行命令。

修改 `pyproject.toml` ，增加如下内容：

```toml
[tool.poetry.plugins.console_scripts]
word_count = "word_count.cmdline:main"
```

当使用 `pip` 命令将项目包安装到环境后，会自动注册一个 `word_count` 的可执行命令。

再次将本地项目以可编辑方式安装到当前 Python 环境：

```bash
poetry install
```

然后就可以正常使用 `word_count` 命令：

```txt
$ word_count -h
usage: word_count [-h] -s SOURCE -d DEST

optional arguments:
  -h, --help            show this help message and exit
  -s SOURCE, --source SOURCE
                        Source file used for count.
  -d DEST, --dest DEST  Dest file used for count result

```

#### 2.5.1 打包

运行打包命令：

```bash
poetry build 
```

`sdist` 会将项目打包成源码包， `bdist_wheel` 会将项目打包成编译后的二进制包。

打包后的文件在 `dist` 目录中。可以直接在其他地方运行 `pip install word_count.wheel` 安装。

#### 2.5.2 发布

将开发好的项目发布到索引仓库，或内网的私有仓库。

```bash
poetry publish
```

默认会将项目发布到 [pypi](https://pypi.org/) 中，所以需要有对应的登录账号。
