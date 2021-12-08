# 代码检测

代码检测是使用一些工具检查代码知否符合 Python 相关规范。

当前主流的代码检测规范包括

- [black](https://github.com/psf/black)
- [flake8](https://flake8.pycqa.org/)
- [pylint](https://www.pylint.org/)
- [yapf](https://github.com/google/yapf/)

## 代码检测工具

### black

[black](https://github.com/psf/black) 是 [PSF](https://www.python.org/psf/github/) 组织下的一个代码格式化工具。
其特点是强制格式化代码，使代码保持一致性。但缺点是会自动调整代码格式。

特点：

- 符合 PEP 8 标准
- 支持自定义规则
- 自动格式化代码
- IDE 插件
- psf 社区维护

### flake8

[flake8](https://flake8.pycqa.org/) 是 [pycqa](http://meta.pycqa.org/) 组织下的一个代码检测工具。它遵循 PEP 8 规范，
指示出不符合规范的代码。

特点：

- 符合 PEP 8 规范
- 集合使用 [pycodestyle](https://github.com/PyCQA/pycodestyle) ， [pyflakes](https://github.com/PyCQA/pyflakes) ， [mccabe](https://github.com/PyCQA/mccabe) 等第三方插件。
- 支持自定义规则
- 提示不符合规范的内容
- IDE 插件
- git 或 Mercurial 扩展
- pycoa 社区维护

### pylint

[pylint](https://www.pylint.org/) 是 [pycqa](http://meta.pycqa.org/) 组织下维护的工具。它不仅仅是一款代码检测工具，还可以发现变成错误，代码异常，并提供简单的重构建议。

特点：

- 符合 PEP 8 规范
- 支持自定义规则
- 错误检测
- 重构建议
- IDE 插件
- pycoa 社区维护

### yapf

[yapf](https://github.com/google/yapf/) 是 [Google](https://opensource.google/) 维护的一个代码检测工具。它和上述工具不同，
使用基于 [clang-format](https://clang.llvm.org/docs/ClangFormat.html) 的算法将代码重新格式化为复合风格指南的最佳格式。类似于 Golang 的 `gofmt` 工具。
所以它和 [black](https://github.com/psf/black) 工具有点类似。

特点：

- 符合 PEP 8 规范
- 支持自定义规则
- 自动格式化代码
- IDE 插件
- google 社区维护

## 使用实践

虽然代码检测工具有很多，但是它们的初衷都是为了让 Python 代码符合一致的风格和规范。只不过是有的工具更激进而已。具有良好编码习惯的开发人员，写出的代码，
无论使用哪种工具，都能轻松通过。所以代码检测工具的最终目的是告知开发人员尽可能遵守一致的风格来编写代码。

考虑到代码检测工具的指导性，和功能性，推荐使用 [pylint](https://www.pylint.org/) 作为首选检测工具。在实践中发现由于某些库和 pylint 的兼容性问题，当
使用 [pylint](https://www.pylint.org/) 有问题时，可以使用 [flake8](https://flake8.pycqa.org/)  作为替代的检测工具。

### 使用

在实际使用过程中，可以将代码检测逻辑放在自动化工具中运行。将逻辑放在 `tox` 中，可以在本地开发时方便使用。在 CI 阶段只需要调用 tox 就可以了。

#### tox

```ini
# tox (https://tox.readthedocs.io/) is a tool for running tests
# in multiple virtualenvs. This configuration file will run the
# test suite on all supported python versions. To use it, "pip install tox"
# and then run "tox" from this directory.

[tox]
isolated_build = True
envlist =
    py{37,38,39,310}
    isort
    lint

[testenv]
deps =
    pipenv
usedevelop = true
commands =
    pipenv sync -d
    pytest --cov=src

[testenv:isort]
deps =
    isort
commands =
    isort . --check-only --diff

[testenv:lint]
deps =
    pipenv
changedir = {toxinidir}
commands =
    pipenv sync -d
    pylint src tests
```

#### github action

```yaml
# This workflow will install Python dependencies, run tests and lint with a single version of Python
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-python-with-github-actions

name: main

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os  }}

    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-20.04]
        python: ["3.7", "3.8", "3.9", "3.10"]

    steps:
      - uses: actions/checkout@v2

      - name: Set up Python ${{ matrix.python }} on ${{ matrix.os }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install tox

      - name: Test with tox
        run: |
          tox -e py

  linting:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2

      - run: pip install tox
      - run: |
          tox -e isort
          tox -e lint
```

#### gitlab-ci

```yaml
default:
  image: python:3.9
  before_script:
    - pip install -U pip

.base_test:
  stage: test
  script:
    - pip install -U tox
    - tox -e py

stages:
  - test
  - build
  - upload

# Due to gitlab ci not support matrix build. So use YAML anchors:
# https://forum.gitlab.com/t/matrix-builds-in-ci/9629
test:py37:
  image: python:3.7
  extends:
    - .base_test

test:py38:
  image: python:3.8
  extends:
    - .base_test

test:py39:
  image: python:3.9
  extends:
    - .base_test

test:py310:
  image: python:3.10
  extends:
    - .base_test

test:lint:
  stage: test
  script:
    - pip install -U tox
    - tox -e isort
    - tox -e lint

```
