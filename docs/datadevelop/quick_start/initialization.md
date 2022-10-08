# 初始化项目

本文通过一个包含主要知识点的简单项目，向开发者展示一个通用、规范和易于理解的ETL的项目开发流程。
示例项目使用`Pyspark`将本地文件进行预处理，并将结果导出文件的演示程序。

## 工程化开发

### 创建项目骨架

使用 [cookiecutter](https://github.com/cookiecutter/cookiecutter) 加载项目模板。通过交互操作，可以选择使用的功能。

```bash
cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project-bigdata-etl
```

### 创建虚拟环境

切换到项目根目录下，项目使用 [poetry](/pythonic-project-guidelines/introduction/virtualenv/#25-poetry)
管理虚拟环境，运行命令自动创建虚拟环境，同时安装开发环境依赖

```bash
poetry install
```

### IDE项目初始化

#### 加载虚拟环境

使用Pycharm打开项目 `File` | `Settings` | `Project` | `Python Interpreter`

选择 `Poetry Environment`

添加刚才创建的虚拟环境(选择`Existing`)

[![pycharm_add_interpreter-1](../../assets/images/pycharm/pycharm_add_interpreter-1.png)](../../assets/images/pycharm/pycharm_add_interpreter-1.png)

[![pycharm_add_interpreter-2](../../assets/images/pycharm/pycharm_add_interpreter-2.png)](../../assets/images/pycharm/pycharm_add_interpreter-2.png)

#### 修改项目结构

使用Pycharm打开项目  `File` | `Settings` | `Project` | `Project Structure`

将`src`和`tests`目录设置为`Sources`源代码路径

[![add_project_structure](../../assets/images/pycharm/add_project_structure.png)](../../assets/images/pycharm/add_project_structure.png)

#### ETL项目结构概述

[//]: # (TODO 详细介绍项目工程化结构，并附加代码块)

ETL任务放在`Tasks`目录下，实现`AbstractTransform`和`AbstractTask`

- `AbstractTask`: Task任务抽象类，`executor`执行器实例化`插件Task`，执行`AbstractTask`抽象父类的`run`方法
    - `run`: 执行Task任务流程
    - `_extract`: 数据源抽取(抽象方法)
    - `_transform`: 数据转换(抽象方法)，执行转换流程，调用`AbstractTransform`子类
    - `_load`: 数据加载(抽象方法)
- `AbstractTransform`: Transform抽象类，提供`AbstractTask`中`_transform`使用，同一个Task可以实现多个`_transform`
    - `_transform`: 数据转换(抽象方法)，指定转换流程，处理输入数据(DataFrame)

ETL任务完成后需要注册插件：

因为项目默认使用[poetry](/pythonic-project-guidelines/introduction/virtualenv/#25-poetry)
管理虚拟环境，则需要在`pyproject.toml` 文件增加中增加如下内容：

```toml
[tool.poetry.plugins."etl_tasks"]
task_name = "{task_class_path}:TaskExample"
```

使用如下命令将项目插件更新到环境中：

```shell
poetry install
```
