# 项目结构

从哪些地方描述：

- 分别描述两种目录结构
- 两种目录结构的比较与区别
- 当前采用的结构

由于 Python 简单易用，很多开始使用 Python 的人都是从一个脚本文件开始，逐步形成多个 Python 文件组成的程序。也正因为如此大部分人并没以一个项目或工程的概念去看待自己的程序。而现在社区中的流行项目也存在两种不同的目录结构。

## 1 简单结构

[Python 项目打包](https://packaging.python.org/tutorials/packaging-projects/) 文章中以一个简单项目结构演示了如何打包一个 Python 项目

```txt
packaging_tutorial
├── LICENSE
├── README.md
├── example_pkg
│   └── __init__.py
├── setup.py
└── tests
```

项目结构以根目录开始，作为项目的环境。因为，为了在开发中正常导入 `example_pkg` 中所有的东西，就需要将项目根目录添加到 `sys.path` 中。这也就让项目根目录下的所有包都变成了可导入。当有多个同级包时，它们都是扁平的散落在项目根目录。项目根目录下可能还存在其他非包目录，如 `data` 、 `docs` 等。如果需要本地引用第三方库，也需要放到根目录，但第三方包并不是项目的子包，而是它的一个引用。这样做会造成职责混乱。

比如这样的一个项目：

```txt
tutorial
├── LICENSE
├── README.md
├── data
|   └── user.json
├── docs
│   └── history.md
├── user
│   └── __init__.py
├── views
│   └── __init__.py
├── requests            # 这是需要本地打包的第三方包
│   └── __init__.py
├── setup.py
└── tests
```

当多个目录扁平的分布在项目根目录时，它们扮演者不同的功能，在开发上，会带了一定的混乱。而且在打包和测试上也会带来一些不便。

在打包上，需要提供更多的配置排除不必要的目录，如 `docs` 或者其他不需要打包仅项目中的东西。

当使用可编辑安装（ `pip install -e .` ） 时，会将项目根目录中的所有东西安装到环境中，包括一些不需要的。

使用自动化测试 `tox` 工具无法检测安装之后的问题，因为这种目录环境可以直接使用环境中的包（项目根目录被添加到 `sys.path` 中了）。

## 2 src 结构

[Pypa 维护的示例项目](https://github.com/pypa/sampleproject) 中采用了一种更推荐的结构 `src` 结构。

```txt
sampleproject
├── data
├── src
|   └── sample
|       └── __init__.py
├── setup.py
└── tests
```

六年前的这篇文章 [Packaging a python library](https://blog.ionelmc.ro/2014/05/25/python-packaging/#the-structure) 就详细阐述了使用 `src` 结构比简单结构的诸多有点。而现在也逐渐被社区作为一个标准遵循。虽然社区中有大量老的项目依然采用简单布局，但新项目推荐使用 `src` 结构。

如下面这个示例项目结构：

```txt
sampleproject
├── data
│   └── user.json
├── docs
│   └── history.md
├── setup.cfg
├── setup.py
├── src
│   ├── requests
│   │   └── __init__.py
│   └── sample
│       ├── __init__.py
│       ├── user
│       │   └── __init__.py
│       └── views
│           └── __init__.py
├── tests
│   ├── __init__.py
│   ├── user
│   │   └── __init__.py
│   └── views
│       └── __init__.py
└── tox.ini
```

项目的包结构很清晰，在环境中只需要引入 `src` 目录，就可以轻松导入项目源代码。通过 `pip install -e .` 可编辑安装，也只会安装 `src` 中的包。管理起来更加清晰。

## 3 实践

下面以一个简单真实的项目来演示使用 `src` 组织项目

### 3.1 创建项目

**创建项目:**

```bash
mkdir sampleproject
cd sampleproject
```

**初始化版本管理：**

```bash
git init
# 如果没有全局用户名和邮箱，需要先配置
git config user.email example@example.com
git config user.name example
```

**创建项目自述文件：**

```bash
touch README.md
```

### 3.2 编写项目源代码

**创建项目包：**

```bash
mkdir src/sample_project
touch src/sample_project/__init__.py
```

**初始化版本号：**

`src/sample_project/__init__.py`

```python
__version__ = '0.1.0'
```

**安装依赖：**

```bash
pip install click
```

**创建命令入口文件：**

`src/sample_project/cmdline.py`

```python
import click

@click.command()
def main():
    click.echo('Hello world!')


if __name__ == "__main__":
    main()
```

### 3.3 编写测试

**创建测试目录：**

```bash
mkdir -p tests/sample_project
touch tests/sample_project/__init__.py
```

**安装依赖：**

```bash
pip install pytest
```

**创建测试文件：**

`tests/sample_project/test_cmdline.py`

```python
from click.testing import CliRunner

from sample_project import cmdline


def test_main():
    runner = CliRunner()
    result = runner.invoke(cmdline.main)
    assert 'Hello world!' in result.output
```

**运行测试：**

```bash
pip install -e .  # 以可编辑安装方式到环境中
pytest
```

测试运行成功，说明功能正确

### 3.4 初始化打包配置


**编写打包配置：**

`setup.py`

```python
import setuptools

setuptools.setup()
```

`setup.cfg`

```ini
[metadata]
name = sample_project
version = attr: sample_project.__version__
author = example
author_email = example@example.com
description = Sample Project
keywords = ssl_manager
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.7

[options]
python_requires > = 3.7
include_package_data = True
packages = find:
package_dir =
    = src
install_requires =
    click

[options.entry_points]
console_scripts =
    ssl_manager = sample_project.cmdline:main

[options.packages.find]
where = src

[tool:pytest]
testpaths = tests
python_files = tests.py test_*.py *_tests.py
```

**打包：**

```bash
python setup.py bdist_wheel
```


### 3.5 总结

至此，一个项目开发完成，完整项目结构如下：

```txt
├── build
│   ├── bdist.linux-x86_64
│   └── lib
│       └── sample_project
│           ├── cmdline.py
│           └── __init__.py
├── dist
│   └── sample_project-0.1.0.linux-x86_64.tar.gz
├── setup.cfg
├── setup.py
├── src
│   ├── sample_project
│   │   ├── cmdline.py
│   │   ├── __init__.py
│   └── sample_project.egg-info
│       ├── dependency_links.txt
│       ├── entry_points.txt
│       ├── PKG-INFO
│       ├── requires.txt
│       ├── SOURCES.txt
│       └── top_level.txt
└── tests
    ├── __init__.py
    └── sample_project
        ├── __init__.py
        └── test_cmdline.py
```
