# 构建与发布

作为项目的最后一环，分发至关重要。有良好的分发流程，便于使用。基于 Python 自带的分发机制显然是
更好的选择。

本文将以一个数据导出的项目讲述。

## 1. 项目准备

因为本文的重点是对打包分发，所以项目的功能开发就不作为重点。

项目源代码可以在 [pythonic-project-samples](https://github.com/whg517/pythonic-project-samples/tree/example/file2mongo) 中获取。

项目采用 `src` 目录结构，项目描述信息都在 `setup.cfg` 中定义。

## 2. 项目打包

### 2.1 打包工具

由于历史原因， Python 的打包走了很长一段路了，但和其他语言的打包工具相比，为了还是有很长一段路要走。

在 [PEP-517](https://www.python.org/dev/peps/pep-0517/) 中，提到了当前 Python 构建系统的不足和响应的解决方案。
其主要就是解决让 Python 支持更加灵活的构建系统。
[PEP-518](https://www.python.org/dev/peps/pep-0518/) 则提出为项目指定一个最小的构建系统。

#### 2.1.1 `setuptools`

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
python = "*"

[tool.poetry.dev-dependencies]
pytest = "^3.4"

```

### 2.2 打包构建

考虑到新工具的功能性，现阶段仍然选用 `setuptools` 作为构建工具。

为项目指定所需要使用的构建工具。创建 `pyproject.toml` 文件，增加如下内容：

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

```

创建 `setup.cfg` 文件，增加 [Setuptools](https://setuptools.readthedocs.io/en/latest/index.html) 配置。

```ini
[metadata]
name = file2mongo
version = attr: file2mongo.__version__
author = demo
author_email = demo@example.com
description = File data to MongoDB
keywords = file2mongo
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
    dynaconf
    click
    pymongo

[options.packages.find]
where = src
exclude =
    tests*
    docs

# https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html
[options.entry_points]
console_scripts =
    file2mongo = file2mongo.cmdline:main

# Packaging project data in module file2mongo.
# https://setuptools.readthedocs.io/en/latest/userguide/datafiles.html?highlight=package_data
[options.package_data]
file2mongo.config =
    settings.yml

# Copy data for user from project when pip install.
# The relative path is prefix `sys.prefix` . eg: `/usr/local/`.
# Path and data will remove When pip uninstall.
# https://docs.python.org/3/distutils/setupscript.html#installing-additional-files
[options.data_files]
etc/file2mongo =
    src/file2mongo/config/settings.yml
```

#### 2.2.1 metadata

[metadata](https://setuptools.readthedocs.io/en/latest/userguide/declarative_config.html?highlight=metadata#metadata) 是项目基本描述信息，有项目名称，版本号，作者相关信息，证书等。

为了让别人更精准的获取你的包的信息，应该至少包含如下：

- `name` ： 项目名称
- `version` ： 版本号，支持字符串，文件内容，或者项目包中的变量。如果需要使用包中的变量，则 Setuptools 的版本号需要大于等于 `39.2.0`
- `url` ： 项目地址
- `author` ： 项目作者
- `author_email` ： 作者邮箱
- `classifiers` ： 分类。有助于在 Pypi 中使用条件过滤查找项目。
- `license` ： 许可证。可以保护你的知识产权。
- `description` ： 项目简要描述。
- `long_description` ： 项目描述，支持字符串和文件(Markdown 文件或 RST 文件)
- `keywords` ： 项目关键字，有助于模糊搜索匹配。

##### 2.2.1.1 版本号

Python 项目版本规范可以参考 [PEP 440 -- Version Identification and Dependency Specification](https://www.python.org/dev/peps/pep-0440/) ，
Setuptools 本身已经支持多种版本号控制方案。但推荐使用 [语义化版本方案](https://www.python.org/dev/peps/pep-0440/#semantic-versioning) 。
对于 Setuptools 支持的版本方案可以参考 [Specifying Your Project’s Version](https://setuptools.readthedocs.io/en/latest/userguide/distribution.html#specifying-your-project-s-version) 。

**标记和 ”每日构建“ 或 ”快照“ 版本** ：

当一组相关的项目正在开发时，跟踪比通常用于“稳定”发行版的更细粒度版本增量可能很重要。而稳定版本可以用带点链接的数字和 alpha/beta/etc 来表示。
状态代码、项目的开发版本通常需要通过修订、构建号甚至构建日期来跟踪。当开发中的项目需要相互引用时，这一点尤其正确，因此可能确实需要某个东西的最新版本!

为了支持这些场景，Setuptools 允许您通过在项目的“官方”版本标识符中添加一个或多个以下内容来“标记”源代码和egg发行版:

- 手动指定的预发布标记，例如“build”或“dev”，或手动指定的发布后标记，例如构建或修订号（`--tag-build=STRING`, `-bSTRING`）
- 用 8 个字符表示的构建日期（`--tag-date`， `-d` ）作为 postrelease 标记

可以在生成每日构建或快照的sdist或bdist命令之前添加egg_info和所需的选项来添加这些标签。有关更多详细信息，请参见 [egg_info](https://setuptools.readthedocs.io/en/latest/userguide/commands.html#egg-info) 命令的以下部分。

（另外，为了确保依赖项处理工具将知道您项目的哪个版本比其他版本新，在发布项目之前，请确保参阅上面的 [指定项目的版本](https://setuptools.readthedocs.io/en/latest/userguide/distribution.html#specifying-your-project-s-version) 部分。）

最后，如果您经常创建构建，并且在可下载的位置构建它们，或者将它们复制到分发服务器，那么您可能还应该检执行[rotate](https://setuptools.readthedocs.io/en/latest/userguide/commands.html#rotate)
 命令，该命令会自动删除除 N 个最近修改的与 glob 模式匹配的发行版之外的所有发行版。操作如下:

```bash
setup.py egg_info -rbDEV bdist_egg rotate -m.egg -k3
```

构建一个 egg ，其版本信息包括“DEV-rNNNN”(其中NNNN是资源树的最新修订)，然后从发行目录中删除除最近构建的三个egg文件之外的所有egg文件。

如果您必须管理多个包的自动化构建，每个包都有不同的标记和滚动策略，那么您可能还需要运行 [alias](https://setuptools.readthedocs.io/en/latest/setuptools.html#alias) 命令，
该命令允许每个包定义一个别名，就像daily 一样，它将执行必要的标记、构建和滚动命令。然后，一个简单的脚本或 cron 作业就可以每天在每个项目目录中运行 setup.py 。
(您还可以定义站点范围或每个用户的每日别名的默认版本，这样没有定义自己的项目就可以使用适当的默认值。)

**制作官方（非快照）版本** ：

当您发布正式版本，创建源代码或二进制发行版时，需要覆盖setup中的标记设置。这样就不用注册像 `foobar-0.7a1.dev-r34832` 这样的版本了。如果主分支上进行开发，
并且为发行版使用标记或分支，那么就很容易做到只在分支或标记发行版之后更改 setup.cfg，这样主分支仍然会生成开发快照。

另外，如果没有为发布版本进行分支，您可以在命令行上覆盖默认的版本选项，使用如下方法:

```bash
python setup.py egg_info -Db "" sdist bdist_egg
```

该命令的第一部分( `egg_info -Db ""` )将在创建源和二进制 egg 之前覆盖已配置的标记信息。因此，这些命令将使用 `setup.py` 中的普通版本，而不添加构建指定字符串。

当然，如果您经常这样做，您可能希望为这个操作创建一个个人别名，例如:

```bash
python setup.py alias -u release egg_info -Db ""
```

然后你可以像这样使用它：

```bash
python setup.py release sdist bdist_egg
```

当然，也可以创建更复杂的别名来完成上述所有操作。有关更多信息，请参阅 [egg_info](https://setuptools.readthedocs.io/en/latest/userguide/commands.html?highlight=alias#egg-info-create-egg-metadata-and-set-build-tags) 和
 [alias](https://setuptools.readthedocs.io/en/latest/userguide/commands.html?highlight=alias#alias-define-shortcuts-for-commonly-used-commands) 命令下面的部分。

#### 2.2.2 options

此节点内容虽然为可选，但为了项目的完整性，有些内容还是需要的。

- `install_requires` ： 使用 Pip 安装时需要的依赖，即项目使用过程中依赖的包
- `extras_require` ： 使用 Pip 安装时的可选依赖，一般是在提供扩展或者可选功能所欲要的包
- `python_requires` ： 项目使用时所需要的 Python 版本。
- `entry_points` ： 入口点内容，如生成命令行脚本，或者注册模块
- `packages` ： 项目的包，或者使用 `find:` 在 `package_dir` 中自动加载
- `package_dir` ： 项目包所在的位置
- `include_package_data` ： 是否打包数据文件
- `package_dir` ： 包含数据文件的包所在位置
- `data_files` ： 在安装时，写入到系统其他位置的数据文件
- `exclude_package_data` ： 排除的数据文件

##### 2.2.2.1 入口点 EntryPoints

[Entry-points](https://packaging.python.org/specifications/entry-points/) 可以很方便的注册命令行脚本，或者提供一种插件加载机制。

**注册命令行** ：

Setuptools 可以通过使用正确的扩展名自动生成脚本，甚至会在 Windows 上创建一个 `.exe` 文件。使用此特性的方法是在配置文件中定义 `entry points` ，
指出生成的脚本应该导入和运行什么函数。例如，要创建两个名为 `foo` 和 `bar` 的控制台脚本和一个名为 `baz` 的 GUI 脚本，

如下示例：

```python
setup(
    # other arguments here...
    entry_points={
        'console_scripts': [
            'foo = my_package.some_module:main_func',
            'bar = other_module:some_func',
        ],
        'gui_scripts': [
            'baz = my_package_gui:start_func',
        ]
    }
)
```

当该项目安装在非 Windows 平台上(使用 `setup.py install` 、`setup.py develop` 或使用 `pip install` )时，将安装一组 `foo` 、 `bar` 和 `baz` 脚本，
从指定的模块导入 `main_func` 和 `some_func` 。指定的函数在没有参数的情况下被调用，并且返回值被传递给 `sys.exit()` ，因此可以返回一个错误级别日志或一条消息来打印到 `stderr` 。

在 Windows 平台上，将创建 `foo.exe` ， `bar.exe` 和 `baz.exe` 启动程序，和 `foo.py` ， `bar.py` 和 `baz.pyw` 文件。`.exe` 程序通过查找正确版本的 Python 来运行 `.py` 和 `.pyw` 文件。

可以根据需求定义任意数量的 `console script` 和 `gui script` 入口点，每个入口点都可以指定所依赖的 “附加功能” ，这些依赖会在运行时添加到 `sys.path` 中。
更多 “附加功能” 信息请参考 [声明附加内容](https://setuptools.readthedocs.io/en/latest/userguide/dependency_management.html?highlight=declaring-extras#optional-dependencies) 部分。
 `entry points` 的更多内容请参考 [服务和插件的动态发现](https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html#dynamic-discovery-of-services-and-plugins)的部分。

##### 2.2.2.2 打包数据文件

Setuptools 支持[打包数据文件](https://setuptools.readthedocs.io/en/latest/userguide/datafiles.html?highlight=package_data#data-files-support) ，这样可以
很方便的把包中的数据文件预先打包，在分发的时候直接使用。常见的文件就是配置了。

**方法一** ：

```ini
[options]
include_package_data = True
```

`include_package_data` 会告诉 Setuptools 通过 `distutils` 的 `MANIFEST.in` 文件指定数据文件。

**方法二** ：

```ini
[options.package_data]
"" =
    "*.txt"
    "*.rst"
"hello" = 
    "*.msg"
```

使用 `package_data` 可以更细粒度的控制索要包含的数据文件。第一个配置是告诉 Setuptools 在通过全局模式递归匹配以 `.txt` 和 `.rst` 结尾
的所有文件，第二个配置是查找 `hello` 下所有 `.msg` 结尾的文件，并在打包是包含匹配到的所有数据文件。

**方法三（SRC 模式）** ：

```ini
[options]
packages = find:
package_dir =
    = src

[options.packages.find]
where = src

[options.package_data]
"" =
    "*.txt"
    "*.rst"
"hello" = 
    "*.msg"
```

当使用 [SRC 项目结构](https://pyloong.github.io/pythonic-project-guidelines/guidelines/project_structure/#3) 时，需要采用上述配置。

`packages = find:` 中告诉 Setuptools 采用 `setuptools.find_packages` 方法查找项目包结构所在位置， `options.packages.find` 中会配置 `setuptools.find_packages` 方法的参数， `where = src` 则指定项目的包在 `src` 下。

`package_dir` 告诉 `distutils` 包在 `src` 下，因此 `options.package_data` 中的配置都会在 `src` 下的包中查找。

##### 2.2.2.3 安装附加文件

`data_files` 可以指定需要安装到系统中的其他文件，如配置文件、消息目录，数据文件等。

```ini
[options]
packages = find:
package_dir =
    = src

[options.packages.find]
where = src

[options.package_data]
my_package.config = settings.yml
my_package.data = users.csv

[options.data_files]
/etc/my_package =
    src/my_package/config/settings.yml
share/my_package/data = 
    src/my_package/data/users.csv

```

`options.data_files` 之前的几项配置告诉 Setuptools 在构建的时候，将需要的数据文件打包。
它本身的配置内容是指定项目在使用 Pip 安装时，将项目包中 `src/my_package/config/settings.yml` 复制到系统目录 `/etc/my_package/settings.yml` 位置。
并将项目包中 `src/my_package/data/users.csv` 复制到相对于解释器的 `<sys.preifox>/share/my_package/data` 目录中。如果是系统的 Python 解释器，则是相对于 `sys.prefix` 一般是 `/usr` 或者 `/usr/local` ，如果是用户安装，则一般是 `~/.local` 。

更多细节请参考 [安装附加文件](https://setuptools.readthedocs.io/en/latest/deprecated/distutils/setupscript.html?highlight=data_files#installing-additional-files) 。

#### 2.2.3 构建

当配置完成后，就可以开始构建了。

运行命令：

```bash
python -m build
```

运行完成后，会在项目根目录的 `./dist` 中生成两个分发文件。一个是 `.tar.gz` 结尾的源码压缩包，一个是 `.whl` 结尾的二进制包。

## 3 分发

打包后的文件可以通过分发手段给其他人使用。

### 3.1 手动分发

手动分发，即自己管理这些软件包，如通过复制、 `ftp` 或者网络发送等方式。
使用时，下载所需要的版本分发包，然后使用 Pip 安装 `pip install foo.whl` 即可。

### 3.2 使用仓库分发

Python 所用公开包都存放在 [Pypi](https://pypi.org/) ，当我们使用 `pip install requests` 的时候，默认会从 [Pypi](https://pypi.org/) 中查找最新版本的分发包，找到了就先下载到本地，然后安装到环境中。除了官方仓库，还支持私有仓库。

要发布到 [Pypi](https://pypi.org/) ，首先需要注册账号，如果是要测试，则可以使用测试仓库[Test-Pypi](https://test.pypi.org/) 。对于私有仓库，可以参考具体文档，但使用方法基本一致，
只需要替换一下仓库地址。

安装 [Twine](https://twine.readthedocs.io/en/latest/) 工具。

```bash
pip install twine
```

上传到 [Test-Pypi](https://test.pypi.org/) ：

```bash
twine upload -r testpypi dist/*
```

填写用户名和密码即可上传。

上传到 [Pypi](https://pypi.org/) ：

```bash
twine upload dist/*
```

#### 3.2.1 Twine 配置

Twine 提供了配置，可以避免每次输密码或者仓库地址的麻烦。

**配置文件** ：

Twine 可以使用 [.pypirc](https://packaging.python.org/specifications/pypirc/) 中的配置。默认读取位置是 `~/.pypirc`

```ini
[distutils]
index-servers =
    pypi
    testpypi
    private-repository

[pypi]
username = __token__
password = <PyPI token>

[testpypi]
username = __token__
password = <TestPyPI token>

[private-repository]
repository = <private-repository URL>
username = <private-repository username>
password = <private-repository password>
```

上述配置中有三个仓库，分别为 `pypi` 、 `test-pypi` 、 `private-repository` 。在使用时，默认是 `pypi` ，也可以使用 `-r` 参数指定。其中 `username` 、 `password` 为对应仓库用户名和密码。

???+ danger "警告"

    配置文件中以文本形式存储了密码，应注意安全隐患。

    可以使用 [keyring](https://pypi.org/project/keyring/) 或调整文件权限 `chmod 600 ~/.pypirc` 。


**使用环境变量** ：


- `TWINE_USERNAME` ： 仓库用户名
- `TWINE_PASSWORD` ： 仓库密码
- `TWINE_REPOSITORY` ： 仓库地址。既可以是完整地址，也可以是 `.pypirc` 中的仓库名称。

**使用 keyring 管理密码** ：

使用 [keyring](https://pypi.org/project/keyring/) 可以很安全的管理密码，在上传包的时候，输入用户名后即可从 [keyring](https://pypi.org/project/keyring/) 中读取配置的密码。

```bash
keyring set https://upload.pypi.org/legacy/ your-username
```

???+ note "注意"

    [keyring](https://pypi.org/project/keyring/) 在没有桌面环境的地方使用起来会复杂一点。具体使用可以参考
     [Using Keyring on headless Linux systems](https://keyring.readthedocs.io/en/latest/#using-keyring-on-headless-linux-systems) 和
     [Using Keyring on headless Linux systems in a Docker container](https://keyring.readthedocs.io/en/latest/#using-keyring-on-headless-linux-systems-in-a-docker-container)

**禁用 Keyring** ：

在一些特殊情况下，可能并不想使用 [keyring](https://pypi.org/project/keyring/) ，或者 [keyring](https://pypi.org/project/keyring/) 的提示会造成系统流程出现意外，比如 CI 情况下。所以可以通过如下方式禁用：

```bash
keyring --disable
```

