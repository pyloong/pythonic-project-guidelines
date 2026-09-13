# 虚拟环境

文章要点：

- 介绍 Python 的虚拟环境
- 介绍并使用 Python 中常见的虚拟环境
- 总结开发中的虚拟环境的使用实践

## 1. 概述

Python 应用程序通常会使用不在标准库内的软件包和模块。应用程序有时需要特定版本的库，因为应用程序可能需要修复特定的错误，或者可以使用库的过时版本的接口编写应用程序。

这意味着一个 Python 环境可能无法满足每个应用程序的要求。如果应用程序 A 需要特定模块的 1.0 版本，但应用程序 B 需要 2.0 版本，则需求存在冲突，安装版本 1.0 或 2.0 将导致某一个应用程序无法运行。

这个问题的解决方案是创建一个 [virtual environment](https://docs.python.org/zh-cn/3/glossary.html#term-virtual-environment) ，一个目录树，其中安装有特定Python版本，以及许多其他包。

然后，不同的应用将可以使用不同的虚拟环境。 要解决先前面例子中的冲突，应用程序 A 可以拥有自己的安装了 1.0 版本的虚拟环境，而应用程序 B 则拥有安装了 2.0 版本的另一个虚拟环境。 如果应用程序 B 要求将某个库升级到 3.0 版本，也不会影响应用程序 A 的环境。

## 2. 虚拟环境管理工具

现在 Python 的虚拟环境管理工具越来越强大。常见的虚拟环境管理工具如下：

- [`venv`](https://docs.python.org/zh-cn/3/library/venv.html#module-venv) ： Python 标准库中的虚拟环境管理工具
- [`conda`](https://docs.conda.io/en/latest/) ： Anaconda 下的管理工具
- [`Virtualenv`](https://virtualenv.pypa.io/en/latest/) ： 第三方的虚拟环境管理工具，现在在 Pypa 中维护。
- [`Pipenv`](https://pipenv.pypa.io/en/latest/) ： 第三方的虚拟环境管理工具，现在在 Pypa 中维护。
- [`poetry`](https://python-poetry.org/) ： 第三方的虚拟环境管理工具。

### 2.1 venv

[`venv`](https://docs.python.org/zh-cn/3/library/venv.html#module-venv) 是 Python 标准库中的一个模块。如果系统中有多个版本的 Python 环境，可以创建指定版本的虚拟环境。

**创建虚拟环境：**

在当前目录创建一个名为 `demo` 的虚拟环境目录：

```bash
python3 -m venv demo
```

如果 `demo` 不存在，就会创建该目录，同时在里面创建 Python 解释器，标准库和各种支持文件的副本目录。

通常创建以点开头的 `.venv` 目录。既可以做到隐藏目录的效果，也可以和常见的 `.env` 环境变量定义文件区分。

**使用虚拟环境：**

下面激活环境变量

Windows:

```cmd
demo\Scripts\activate.bat
```

Unix 或 MacOs 上：

```bash
source demo/bin/active
```

激活后就可以在终端中使用创建的虚拟环境了。

```text
$ source demo/bin/activate
(demo)  $ python
Python 3.7.3 (default, Oct 28 2020, 14:33:53)
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.version
'3.7.3 (default, Oct 28 2020, 14:33:53) \n[GCC 8.3.0]'
>>> sys.path
['', '/usr/lib/python37.zip', '/usr/lib/python3.7', '/usr/lib/python3.7/lib-dynload', '/tmp/test/demo/lib/python3.7/site-packages']
```

**退出虚拟环境：**

```bash
deactive
```

### 2.2 Conda

Conda 是在 Windows， macOS 和 Linux 上运行的开源软件包管理系统和环境管理系统。 Conda 快速安装，运行和更新软件包及其依赖项。Conda 可以轻松地在本地计算机上的环境中创建，保存，加载和切换。它是为 Python 程序创建的，但可以打包和分发适用于任何语言的软件。

Conda 作为软件包管理器可以帮助您查找和安装软件包。如果您需要一个需要使用其他版本的 Python 的软件包，则无需切换到其他环境管理器，因为 Conda 也是环境管理器。仅需几个命令，您就可以设置一个完全独立的环境来运行该不同版本的Python，同时继续在正常环境中运行您通常的 Python 版本。

在默认配置下，Conda 可以安装和管理在 [repo.anaconda.com](https://repo.anaconda.com/) 上，由 Anaconda® 审查和维护的上千个软件包。

Conda可以与 Travis CI 和 AppVeyor 等持续集成系统结合使用，以提供频繁，自动的代码测试。

所有版本的 Anaconda 和 Miniconda 中都包含 conda 软件包和环境管理器。

**操作前提：**

请确保 Python 环境是由 Anaconda 或 Miniconda 提供的。

**创建虚拟环境：**

创建一个名为 `demo` 目录的虚拟环境

```bash
conda create --name demo
```

**使用虚拟环境：**

<!-- markdownlint-disable MD013 -->

```text
C:\Users\test>conda activate demo

(demo) C:\Users\test>python
Python 3.8.3 (default, Jul  2 2020, 17:30:36) [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', 'C:\\ProgramData\\Anaconda3\\python38.zip', 'C:\\ProgramData\\Anaconda3\\DLLs', 'C:\\ProgramData\\Anaconda3\\lib', 'C:\\ProgramData\\Anaconda3', 'C:\\ProgramData\\Anaconda3\\lib\\site-packages', 'C:\\ProgramData\\Anaconda3\\lib\\site-packages\\win32', 'C:\\ProgramData\\Anaconda3\\lib\\site-packages\\win32\\lib', 'C:\\ProgramData\\Anaconda3\\lib\\site-packages\\Pythonwin']
>>> sys.version
'3.8.3 (default, Jul  2 2020, 17:30:36) [MSC v.1916 64 bit (AMD64)]'
```
<!-- markdownlint-restore -->
**退出虚拟环境：**

```bash
deactivate
```

### 2.3 Virtualenv

[Virtualenv](https://virtualenv.pypa.io/en/latest/index.html) 是一个第三方库，现在由 Pypa 管理。其具有比 `venv` 更强大的功能，但现在 Virtualenv 的一些功能也在慢慢是配到 `venv` 上。

**安装：**

```bash
pip install -U virtualenv
```

Virtualenv 在 Conda 环境下会有 Bug 。[^virtualenv-bug]

**创建虚拟环境：**

创建一个名为 `venv` 目录的虚拟环境

```bash
virtualenv venv
```

**使用虚拟环境：**

```bash
source venv/bin/activate
```

**好用的工具：**

搭配 [VirtualenvWrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) 可以更方便的使用和管理虚拟环境。

Linux 安装：

```bash
pip install virtualenvwrapper
# 执行 virtualvnewrapper 初始化脚本。可以将下面这一行加入到 `~/.bashrc` 中，方便当前用户使用，或者加入到 `/etc/profile` 中方便所有用户使用
source /usr/local/bin/virtualenvwrapper.sh
```

创建虚拟环境：

```bash
# 执行命令，默认会在 `~/.virtualenvs` 下创建对应名称的虚拟环境目录，同时初始化虚拟环境。
# 所有虚拟环境都会集中存放在这里。避免了项目根目录下有虚拟环境目录。
mkvirtualenv venv
```

使用虚拟环境：

```bash
workon venv
```

删除虚拟环境：

```bash
rmvirtualenv venv
```

Windows 安装：

```bash
pip install virtualenvwrapper-win
```

设置环境变量 `WORKON_HOME=D:/virtualenvs`

使用的方法和上面一致。

### 2.4 Pipenv

[Pipenv](https://pipenv.pypa.io/en/latest/) 是一个更高级的虚拟环境管理工具，其依赖 `Virtualenv` ，并在之上做了许多其他功能。正如其官网中所说，它的目的是要把所有最好的包管理（ `bundler` , `composer` , `npm` ， `yarn` 等）引入到 Python 中。

Pipenv 具有如下特点

- 集中存储虚拟环境，如果不存在则直接创建。可以通过 `WORKON_HOME` 环境变量配置。
- 生成 `Pipfile` 和 `Pipfile.lock` 。前者记录依赖项、安装源、要使用的 Python 版本，后者记录所安装的的版本的 Hash 值等信息。
- 自动安装卸载依赖，自动清除无用的依赖。
- 自动加载 `.env` 文件。
- 能根据依赖树的关系检测依赖冲突。

**安装：**

```bash
pip install pipenv
```

**创建虚拟环境：**

在项目根目录执行 `pipenv install` ：

<!-- markdownlint-disable MD009 -->
```text
root@b2e8a92bace7:~/demo# pipenv install
Creating a virtualenv for this project...
Pipfile: /root/demo/Pipfile
Using /usr/local/bin/python3 (3.7.7) to create virtualenv...
⠸ Creating virtual environment...created virtual environment CPython3.7.7.final.0-64 in 175ms
  creator CPython3Posix(dest=/root/.virtualenvs/demo-xfYnOzmm, clear=False, no_vcs_ignore=False, global=False)                                                              
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)                                     
    added seed packages: pip==20.2.4, setuptools==50.3.2, wheel==0.35.1                                                                                                     
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator                                                                 
                                                                                                                                                                            
✔ Successfully created virtual environment! 
Virtualenv location: /root/.virtualenvs/demo-xfYnOzmm
Creating a Pipfile for this project...
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Updated Pipfile.lock (a65489)!
Installing dependencies from Pipfile.lock (a65489)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```
<!-- markdownlint-restore -->

**使用虚拟环境：**

单次使用

```bash
# 查看虚拟环境的 Python 版本
pipenv run python --version
```

进入虚拟环境

```bash
pipenv shell
```

**安装依赖：**

```bash
pipenv install tox
```

依赖安装完成后，会更新 `Pipfile` 文件，同时更新 `Pipfile.lock` 文件，记录安装的版本和对应 HASH 值。

### 2.5 Poetry (推荐使用)

[Poetry](https://python-poetry.org/) 是后期之秀，它的雄心不仅仅是做 Pipenv 的事，它还想把 Python 的打包管理一并做了，并消除 `setup.py` 文件。它使用基于 [PEP517](https://www.python.org/dev/peps/pep-0517/) 规范的 `pyproject.toml` 文件记录信息，并打包。
具体内容可以参考 [PEP 517 -- A build-system independent format for source trees](https://www.python.org/dev/peps/pep-0517/) 。
当前基于 PEP517 的构建模式已经完全可用。在 pip 的发行记录中，最早是在 [18.1 (2018-10-05)](https://pip.pypa.io/en/stable/news/#id438) 就引入了 PEP517 的 `0.2` 版本。

在使用上，Poetry 给人的感觉更现代化。

**安装：**

Poetry 自身应安装在独立虚拟环境中，不要安装到系统 Python 或 Poetry 将要管理的项目虚拟环境。
按照 [Poetry 官方安装说明](https://python-poetry.org/docs/#installation)，推荐使用 [pipx](https://pipx.pypa.io/stable/) 安装；pipx 会为 Poetry 创建和维护隔离环境，同时让 `poetry` 命令可在各项目中使用。

```bash
pipx install poetry
poetry --version
```

**使用：**

```bash
# 使用前需要先初始化项目的基本信息，生成 `pyproject.toml` 文件
poetry init

# 安装依赖
poetry add tox

# 进入虚拟环境
poetry shell

# 构建项目
poetry build

# 发布项目
poetry publish
```

## 3. 虚拟环境实践(Poetry)

众多的虚拟环境，和对应的工具，在选择时难免有点困惑，要选择一个好用的工具，最佳途径就是自己都尝试一遍。

上述几个主流虚拟环境工具除了 `venv` 是内置库，其他几个都是基于 `Virtualenv` 再次开发，并提供了其他功能。但是 `Virtualenv` 没有提供依赖检测的功能，而且依赖包的管理还是需要使用 `pip` 命令，依赖项需要通过 `requirements.txt` 。

当你需要管理不同开发环境下的依赖时，就需要两个或更多个 `requirements.txt` 。例如 `requirements-devlopment.txt` ， `requirements-production.txt` 或这 `requirements-test.txt` 。

`Poetry` 是一个更酷的工具，无论是交互地输出，还是它基于 PEP517 的特性。它允许您声明您的项目所依赖的库，它将为您管理（安装/更新）它们。Poetry 提供了一个锁定文件以确保可重复安装，并可以构建您的项目以供分发。
poetry 通过配置文件 pyproject.toml 来完成依赖管理、环境配置、基本信息配置等功能，相当于把 Python 项目中的 Pipfile、setup.py、setup.cfg、requirements.txt、MANIFEST.in 融合到一起。

综合来看 `Poetry` 就显得更加合适，支持多种环境管理，提供依赖关系校验，和依赖的 poetry.lock 文件，也有自动管理依赖的操作。同时可以用于 Python 工程打包和发布。

下面以一个项目的生命周期描述如何更好的使用 `Poetry` 。

### 3.1 初始化项目

初始化项目，使用 `Poetry` 在项目根目录创建当前项目的虚拟环境。

```bash
poetry init
```

进入当前项目的虚拟环境。

```bash
poetry shell
```

### 3.2 安装项目依赖

当需要区分开发环境和普通环境时，就可以通过 `add -D` 选项安装开发环境依赖

```bash
poetry add -D pytest tox
```

一般的依赖直接安装即可。

```bash
poetry add django requests scrapy sqlalchemy
```

### 3.3 清理依赖

当需要从环境中清除不在需要的依赖时，可以使用命令卸载

```bash
poetry remove scrapy
```

或者直接修改 `pyproject.toml` 文件，删除不再需要的内容，然后通过 `poetry lock` 更新 `poetry.lock` 文件。

### 3.4 部署

在部署时，强烈推荐使用 `poetry install` 安装在 `pyproject.toml` 文件中依赖包。

### 3.5 生成 `requirements.txt`

使用 `poetry show` 可以看到所有依赖列表。

```bash
# 查看所有依赖
poetry show
# 仅所有开发依赖
poetry show --only dev 
```

```bash
poetry export -f requirements.txt --output --without-hashes
```

[^virtualenv-bug]: 如果你的是 Conda 环境，请使用 `20.0.34` 之前的 Virtualenv 。具体请参考 [virtualenv==20.0.34 not compatible with python on windows #12094](https://github.com/ContinuumIO/anaconda-issues/issues/12094) 和 [conda support - Windows 3.7+ #1986](https://github.com/pypa/virtualenv/issues/1986) 。如果这个问题已经修复，请忽略。
