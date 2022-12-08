# 构建与发布

作为项目的最后一环，分发至关重要。有良好的分发流程，便于使用。基于 Python 自带的分发机制显然是
更好的选择。

本文将以一个数据导出的项目讲述。

## 1. 项目准备

因为本文的重点是对打包分发，所以项目的功能开发就不作为重点。

项目源代码可以在 [pythonic-project-samples](https://github.com/whg517/pythonic-project-samples/tree/example/file2mongo) 中获取。

项目采用 `src` 目录结构，项目描述信息都在 `pyproject.toml` 中定义。

## 2. 项目打包

### 2.1 打包工具

由于历史原因， Python 的打包走了很长一段路了，但和其他语言的打包工具相比，为了还是有很长一段路要走。

在 [PEP-517](https://www.python.org/dev/peps/pep-0517/) 中，提到了当前 Python 构建系统的不足和响应的解决方案。
其主要就是解决让 Python 支持更加灵活的构建系统。
[PEP-518](https://www.python.org/dev/peps/pep-0518/) 则提出为项目指定一个最小的构建系统。

#### 2.1.1 setuptools

[Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 是当前使用最为广泛的构建工具，现在绝大多数工具
都在使用 `setuptools` 构建项目。它是 `disutils` 的增强版。

[Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 可以说是现在最成熟的构建工具了，支持常用特性如下：

- [支持打包资源文件](https://setuptools.readthedocs.io/en/latest/userguide/distribution.html#generating-source-distributions)
- [支持打包数据文件](https://setuptools.readthedocs.io/en/latest/userguide/datafiles.html)
- [支持 CPython 编译器](https://setuptools.readthedocs.io/en/latest/userguide/distribution.html#distributing-extensions-compiled-with-cython)
- [支持 zip 压缩选项](https://setuptools.readthedocs.io/en/latest/userguide/miscellaneous.html#setting-the-zip-safe-flag)
- [支持包命名空间](https://setuptools.readthedocs.io/en/latest/userguide/package_discovery.html)
- [支持安装依赖](https://setuptools.readthedocs.io/en/latest/userguide/dependency_management.html#package-requirement)
- [支持可选安装依赖](https://setuptools.readthedocs.io/en/latest/userguide/dependency_management.html#optional-dependencies)
- [支持指定 Python 版本](https://setuptools.readthedocs.io/en/latest/userguide/dependency_management.html#python-requirement)
- [支持注册 Setuptools 子命令](https://setuptools.readthedocs.io/en/latest/userguide/extension.html#adding-commands)
- [支持入口点（Entry Points）](https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html)

由于 [Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 是最成熟的构建工具，所以与其它构建工具对比来看，它的缺点可能就是现在的配置仍需要定义在 `setup.cfg` 或者 `setup.py` 文件中，而不是定义在 `pyproject.toml` 文件中。

缺点：

- 不支持在 `pyproject.toml` 中定义配置
- 不支持发布，需要配合 `twine` 。

##### 2.1.1.1 示例配置

**pyproject.toml** :

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

增加 `setup.py` 或者 `setup.cfg` 两种有其一即可。然后在文件中定义配置。

推荐使用 `setup.cfg` 。

=== "setup.cfg"

    ```cfg
    [metadata]
    name = mypackage
    version = 0.0.1

    [options]
    packages = mypackage
    install_requires =
            requests
        importlib; python_version == "3.7"
    ```

=== "setup.py"

    ```python
    from setuptools import setup

    setup(
        name='mypackage',
        version='0.0.1',
        packages=['mypackage'],
        install_requires=[
            'requests',
            'importlib; python_version == "2.6"',
        ],
    )
    ```

此时你的项目结构应在是这样的：


```text
~/mypackage/
    pyproject.toml
    setup.cfg # or setup.py
    mypackage/__init__.py
```

##### 2.1.1.2 通用方式构建

安装 PEP-517 规范的包生成器 [build](https://pypi.org/project/build/) 和 [setuptools](https://pypi.org/project/setuptools/)，`pip install build setuptools` 。

然后开始构建 `python -m build wheel` 。

##### 2.1.1.3 Setuptools 构建

此方法是在不安装 [build](https://pypi.org/project/build/) 的情况下使用的。

如果你只是用了 `setup.cfg` 配置的情况下，你还需要增加一个 `setup.py` 文件，内容如下：

**setup.py** ：

```python
from setuptools import setup

setup()
```

如果你仅使用了 `setup.py` 配置 [Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 的话，可以不需要 `setup.cfg` 文件。

安装依赖 `pip install setuptools` ，然后进行构建 `python setup.py bdist_wheel` 。

##### 2.1.1.4 twine 发布

由于 setuptools 不支持发布功能，所以需要借助其他工具将包发布中央仓库。

[Twine](https://twine.readthedocs.io/en/latest/) 是 [Pypa](https://www.pypa.io/en/latest/) 团队维护的一个将 Python 包发布到 [Pypi](https://pypi.org/) 的工具。

安装依赖： `pip install twine` 。

使用 [Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 构建项目，构建结果默认是放在项目根目录的 `./dist` 下面 。

发布项目到 [Pypi](https://pypi.org/) ：

```bash
twine upload dist/*
```

#### 2.1.2 flit

[Flit](https://flit.readthedocs.io/en/latest/) 是一个轻量简单的 Python 构建工具，它的出现也可以说是划时代的，因为它的出现促进了新标准的发现，
如 [PEP-517](https://www.python.org/dev/peps/pep-0517) 和 [PEP-518](https://www.python.org/dev/peps/pep-0518) 。

`Flit` 具有如下特点：

- [简单轻量](https://flit.readthedocs.io/en/latest/rationale.html#why-use-flit)
- [支持发布](https://flit.readthedocs.io/en/latest/cmdline.html#flit-publish)
- [支持打包数据文件](https://flit.readthedocs.io/en/latest/pyproject_toml.html#sdist-section)
- [支持子包](https://flit.readthedocs.io/en/latest/rationale.html?highlight=Subpackages#why-use-flit)
- [支持复制构建](https://flit.readthedocs.io/en/latest/reproducible.html)
- [支持安装依赖](https://flit.readthedocs.io/en/latest/pyproject_toml.html?highlight=requires#metadata-section)
- [支持可选安装依赖](https://flit.readthedocs.io/en/latest/pyproject_toml.html?highlight=requires-extra#metadata-section)
- [支持指定 Python 版本](https://flit.readthedocs.io/en/latest/pyproject_toml.html?highlight=requires-python#metadata-section)
- [支持注册 Flit 子命令](https://flit.readthedocs.io/en/latest/pyproject_toml.html?highlight=requires-python#scripts-section)
- [支持入口点（Entry Points）](https://flit.readthedocs.io/en/latest/pyproject_toml.html?highlight=requires-python#entry-points-sections)
- [支持 `pyproject.toml` 文件定义配置](https://flit.readthedocs.io/en/latest/pyproject_toml.html)

缺点：

- 不支持 CPython 编译
- 不支持 zip 压缩选项


##### 2.1.2.1 示例配置

**pyproject.toml** ：

```toml
[build-system]
requires = ["flit_core >=2,<4"]
build-backend = "flit_core.buildapi"

[tool.flit.metadata]
module = "foobar"
author = "Sir Robin"
author-email = "robin@camelot.uk"
home-page = "https://github.com/sirrobin/foobar"

```

##### 2.1.2.2 通用方式构建

安装 PEP-517 规范的包生成器  [build](https://pypi.org/project/build/) 和 [flit](https://pypi.org/project/flit/) ，`pip install build flit` 。

然后开始构建 `python -m build wheel` 。

##### 2.1.2.3 flit 构建

此方法是在不安装 [build](https://pypi.org/project/build/) 的情况下使用的。

安装依赖： `pip install flit` ，然后进行构建 `flit build --format wheel` 。

##### 2.1.2.4 flit 发布

`flit` 的发布命令会自行先构建 `wheel` 和 `sdist` 包，然后上传到 [Pypi](https://pypi.org/) 或者其他仓库。

```bash
flit build --format wheel
```

#### 2.1.3 poetry

[python-poetry](https://python-poetry.org/docs/) 是一个更高级的工具。它是一个构建工具的同时也是一个依赖管理工具。
在构建上，同样遵循了 [PEP-517](https://www.python.org/dev/peps/pep-0517) 规范，在依赖管理上，有点类似于 [Pipenv](https://pipenv.pypa.io/en/latest/) 。

[python-poetry](https://python-poetry.org/docs/) 具有如下特点：

- [支持环境管理](https://python-poetry.org/docs/managing-environments/)
- [支持发布](https://python-poetry.org/docs/cli/#publish)
- [支持 shell 插件，如 `bash` 、 `Fish` 、 `Zsh`](https://python-poetry.org/docs/#enable-tab-completion-for-bash-fish-or-zsh)
- [支持打包数据文件](https://python-poetry.org/docs/pyproject/#include-and-exclude)
- [支持注册 poetry 子命令](https://python-poetry.org/docs/pyproject/#scripts)
- [支持 Setuptools 的入口点（Entry Points）](https://python-poetry.org/docs/pyproject/#plugins)
- [支持包命名空间](https://python-poetry.org/docs/pyproject/#packages)
- [支持安装依赖](https://python-poetry.org/docs/pyproject/#dependencies-and-dev-dependencies)
- [支持可选安装依赖](https://python-poetry.org/docs/pyproject/#extras)
- [支持在 `pyproject.toml` 中定义配置](https://python-poetry.org/docs/pyproject/)

缺点：

- [不支持 CPython 编译](https://github.com/Vlek/roll/issues/29)
- 不支持 zip 压缩选项

##### 2.1.3.1 示例配置

```toml
[build-system]
requires = ["poetry_core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.poetry]
name = "poetry-demo"
version = "0.1.0"
description = ""
authors = ["Sébastien Eustace <sebastien@eustace.io>"]

[tool.poetry.dependencies]
python = "^3.10"

[tool.poetry.dev-dependencies]
pytest = "^3.4"

```

### 2.2 打包构建（poetry）

现阶段选用 `poetry` 作为构建工具。

为项目指定所需要使用的构建工具。创建 `pyproject.toml` 文件，增加如下内容：

```toml
[tool.poetry]
name = "file2mongo"
version = "0.1.0"
description = "File data to MongoDB"
readme = "README.md"
authors = ["demo <demo@example.com>"]
license = "MIT"
classifiers = [
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3.10",
]

[tool.poetry.dependencies]
python = "^3.10"
dynaconf = "^3.1.9"
click = "^8.1.3"
pymongo = "^4.3.3"

[tool.poetry.dev-dependencies]
pylint = "^2.14.5"
isort = "^5.10.1"
pytest = "^7.1.2"
mkdocs = "^1.3.1"
mkdocs-material = "^8.4.1"

[tool.poetry.plugins."scripts"]
file2mongo = "file2mongo.cmdline:main"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

```

#### 2.2.1 项目基本信息

[tool.poetry](https://python-poetry.org/docs/pyproject/) 是项目基本描述信息，有项目名称，版本号，作者相关信息等。

为了让别人更精准的获取你的包的信息，应该尽量包含如下：

- `name` ： 项目名称。必需字段
- `version` ： 版本号。必需字段
- `description` : 项目简短描述。必需字段
- `license` ： 许可证。可选字段，建议添加
- `author` ： 项目作者。必需字段
- `maintainers` : 维护者。这是一个维护者列表，应该与作者区分开来。 可能包含一个电子邮件，格式为 name <email>。可选字段
- `readme` : 项目的 README 文件或相对应的路径或路径列表。可选字段
- `homepage` : 项目网站的 URL。可选字段
- `keywords` ： 项目关键字，有助于模糊搜索匹配。可选字段

#### 2.2.2 options

此节点内容虽然为可选，但为了项目的完整性，有些内容还是需要的。

- `tool.poetry.dependencies` : 项目使用过程中依赖的包
- `tool.poetry.scripts` : 安装包时将安装的脚本或可执行文件
- `tool.poetry.extras` :可选依赖项，增强包，但不是必需的，可选依赖项的集群。
- `tool.poetry.plugins` : 插件，可通过 `importlib.metadata`导入
- `tool.poetry.urls` : 自定义 url，发布pypi后展示
- `build-system` : 构建系统引用部分

##### 2.2.2.1 入口点 EntryPoints

[Entry-points](https://packaging.python.org/specifications/entry-points/) 可以很方便的注册命令行脚本，或者提供一种插件加载机制。

**注册命令行** ：

例如，要创建名为 `foo` 的控制台脚本，`pyproject.toml`文件添加如下示例：

```toml
[tool.poetry.scripts]
foo = "my_package.some_module:main_func"

```

#### 2.2.3 构建

当配置完成后，就可以开始构建了。

运行命令：

```bash
poetry build
```

运行完成后，会在项目根目录的 `./dist` 中生成两个分发文件。一个是 `.tar.gz` 结尾的源码压缩包，一个是 `.whl` 结尾的二进制包。

## 3 分发

打包后的文件可以通过分发手段给其他人使用。

### 3.1 手动分发

手动分发，即自己管理这些软件包，如通过复制、 `ftp` 或者网络发送等方式。
使用时，下载所需要的版本分发包，然后使用 Pip 安装 `pip install foo.whl` 即可。

### 3.2 使用仓库分发

Python 所用公开包都存放在 [Pypi](https://pypi.org/) ，当我们使用 `pip install requests` 的时候，默认会从 [Pypi](https://pypi.org/) 中查找最新版本的分发包，找到了就先下载到本地，然后安装到环境中。除了官方仓库，还支持私有仓库。

要发布到 [Pypi](https://pypi.org/) ，首先需要注册账号，如果是要测试，则可以使用测试仓库[Test-Pypi](https://test.pypi.org/) 。对于私有仓库，可以参考具体文档[poetry-publish](https://python-poetry.org/docs/cli/#publish)，但使用方法基本一致，
只需要替换一下仓库地址。

上传到 [Test-Pypi](https://test.pypi.org/) ：

```bash
poetry publish -r https://test.pypi.org/ -u username -p password
```

填写用户名和密码即可上传。

上传到 [Pypi](https://pypi.org/) ：

```bash
poetry publish -u username -p password
```
