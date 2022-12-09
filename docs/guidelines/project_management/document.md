# 文档管理

项目文档用来说明和记录项目的信息，有助于开发人员、管理人员、使用者的交流和沟通。在 Python 项目中
一般通过 [Mkdocs](https://www.mkdocs.org/) 和 [sphinx](https://www.sphinx-doc.org/en/master/) 来
构建项目文档。两者都支持 markdown 标记的文件，但后者也支持 [reStructuredText](https://docutils.sourceforge.io/rst.html) 标记文件。

## mkdocs

[Mkdocs](https://www.mkdocs.org/) 是一个快速、简单的静态站点生成工具。可以通过指定目录中的 markdown 标记文件，来生成静态网页。
使用 YAML 格式配置文件。

特点：

- YAML 单文件配置
- 生成静态站点
- 支持 markdown
- 支持自定义主题
- 支持 markdown 扩展标记
- 支持插件

## sphinx

[sphinx](https://www.sphinx-doc.org/en/master/) 是使用 [reStructuredText](https://docutils.sourceforge.io/rst.html) 标记编写文档，并
生成静态站点的工具。

特点：

- 单个 Python 文件配置
- 生成 HTML 、 ePub 等多种格式
- 支持 markdown 和 reStructuredText
- 支持自定义主题
- 支持扩展

## 实践

在开发实践中，推荐使用 [Mkdocs](https://www.mkdocs.org/) ，因为它简单上手，并且有许多优秀的第三方主题。

### 实践案例

首先创建一个 `example-doc` 的目录，然后初始化项目虚拟环境，安装环境依赖：

```text
❯ mkdir example-doc
❯ cd example-doc
❯ poetry init
Package name [example-doc]: 
Version [0.1.0]: 
Description []: 
Author [doc <doc@example.com>, n to skip]: 
License []: 
Compatible Python versions [^3.10]: 

Would you like to define your main dependencies interactively? (yes/no) [yes]
You can specify a package in the following forms:
  - A single name (requests): this will search for matches on PyPI
  - A name and a constraint (requests@^2.23.0)
  - A git url (git+https://github.com/python-poetry/poetry.git)
  - A git url with a revision (git+https://github.com/python-poetry/poetry.git#develop)
  - A file path (../my-package/my-package.whl)
  - A directory (../my-package/)
  - A url (https://example.com/packages/my-package-0.1.0.tar.gz)

Package to add or search for (leave blank to skip):

Would you like to define your development dependencies interactively? (yes/no) [yes]
Package to add or search for (leave blank to skip):

Generated file

[tool.poetry]
name = "example-doc"
version = "0.1.0"
description = ""
authors = ["doc <doc@example.com>"]
readme = "README.md"
packages = [{include = "example_doc"}]

[tool.poetry.dependencies]
python = "^3.10"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"


Do you confirm generation? (yes/no) [yes]

❯ poetry shell
Creating virtualenv example-doc-DN2_2NFH-py3.10 in C:\Users\qiang.xie\AppData\Local\pypoetry\Cache\virtualenvs
Spawning shell within C:\Users\qiang.xie\AppData\Local\pypoetry\Cache\virtualenvs\example-doc-DN2_2NFH-py3.10
❯ poetry add mkdocs
Using version ^1.4.2 for mkdocs

Updating dependencies
Resolving dependencies...

Writing lock file

Package operations: 14 installs, 0 updates, 0 removals

  • Installing six (1.16.0)
  • Installing colorama (0.4.6)
  • Installing markupsafe (2.1.1)
  • Installing python-dateutil (2.8.2)
  • Installing pyyaml (6.0)
  • Installing click (8.1.3)
  • Installing ghp-import (2.1.0)
  • Installing jinja2 (3.1.2)
  • Installing packaging (22.0)
  • Installing pyyaml-env-tag (0.1)
  • Installing watchdog (2.2.0)
  • Installing mergedeep (1.3.4)
  • Installing markdown (3.3.7)
  • Installing mkdocs (1.4.2)
```

初始化文档配置：

```text
❯ mkdocs new .
INFO     -  Writing config file: ./mkdocs.yml
INFO     -  Writing initial docs: ./docs/index.md
❯ ls
docs  mkdocs.yml  pyproject.toml  poetry.lock

```

然后启动 mkdocs 的本地服务器：

```text
❯ mkdocs serve
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.05 seconds
INFO     -  [11:00:22] Serving on http://127.0.0.1:8000/

```

然后浏览器打开 [http://127.0.0.1:8000] 访问生成的文档站点。

站点使用默认主题，风格有点复古。可以使用 [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) 让站点更好看：

安装 `mkdocs-material` ：

```bash
poetry add mkdocs-material
```

修改配置文件 `mkdocs.yml` ，增加如下内容：

```yaml
theme:
  name: material
```

重新启动 `mkdocs serve` ，即可看到注意已经改变。

对于 Mkdocs 的更多使用细节可以参考文档：

- [Mkdocs 快速开始](https://www.mkdocs.org/getting-started/#getting-started-with-mkdocs)
- [Mkdocs 配置](https://www.mkdocs.org/user-guide/configuration/)
- [mkdocs-material 主题](https://squidfunk.github.io/mkdocs-material/)
