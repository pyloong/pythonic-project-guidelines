# 快速上手

本文通过一个包含主要知识点的简单项目，向开发者展示一个通用、规范和易于理解的Pyspark ETL的项目开发流程。
示例项目是一个将本地文件进行预处理，并将结果导出文件的演示程序。

## 初始化项目

### 创建项目骨架

使用 [cookiecutter](https://github.com/cookiecutter/cookiecutter) 加载项目模板。通过交互操作，可以选择使用的功能。

```bash
cookiecutter https://github.com/JingyuanR/cookiecutter-pyspark-etl-project
```

### 创建虚拟环境

切换到`pyspark-etl-template`
文件路径下，项目使用 [poetry](/pythonic-project-guidelines/introduction/virtualenv/#25-poetry)
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

#### ETL任务开发

ETL任务放在`Tasks`目录下，实现`AbstractTransform`和`AbstractTask`

- `AbstractTask`: Task任务抽象类，`executor`执行器实例化`插件Task`，执行`AbstractTask`抽象父类的`run`方法
    - `run`: 执行Task任务流程
    - `_input`: 数据源抽取(抽象方法)
    - `_transform`: 数据转换(抽象方法)，执行转换流程，调用`AbstractTransform`子类
    - `_output`: 数据加载(抽象方法)
- `AbstractTransform`: Transform抽象类，提供`AbstractTask`中`_transform`使用，同一个Task可以实现多个`_transform`
    - `_transform`: 数据转换(抽象方法)，指定转换流程，处理输入数据(DataFrame)

ETL任务完成后需要注册插件：

因为项目默认使用[poetry](/pythonic-project-guidelines/introduction/virtualenv/#25-poetry)
管理虚拟环境，则需要在`pyproject.toml` 文件增加中增加如下内容：

```toml
[tool.poetry.plugins."poetry.plugin"]
demo = "poetry_demo_plugin.plugin:MyPlugin"
```

使用如下命令将项目插件更新到环境中：

```shell
poetry install
```

## 开发实践

### 任务需求

现有汽车信息数据[car_price.csv](../../assets/data/car_price.csv)

- 对`CarName`字段包含`[dirty data]`数据进行纠正，去除字符串`[dirty data]`

- 删除`price`小于`10000`的汽车数据

- 最终结果Schema：`car_id`, `symboling`, `car_name`, `price`, 将结果导出`json`文件

### Task类

创建汽车数据ETL任务`CarDataTask`类，`src/etl_project/tasks/car_etl_example/task.py`

```py title="task.py"
"""Processing car data task."""
from pyspark.sql import DataFrame

from pyspark_etl_example.tasks.abstract.task import AbstractTask
from pyspark_etl_example.tasks.car_etl_example.car_transform import \
    CarTransform


class CarDataTask(AbstractTask):
    """Processing car data task."""

    def _input(self) -> DataFrame:
        """Read CSV file return DataFrame"""
        df: DataFrame = self.spark.read.csv(
            self.settings.input_path,
            encoding='utf-8',
            header=True,
            inferSchema=True
        )
        self.logger.info(f'Extract data from {self.settings.input_path}')
        return df

    def _transform(self, df: DataFrame) -> DataFrame:
        """execute CarTransform transform function"""
        return CarTransform().transform(df)

    def _output(self, df: DataFrame) -> None:
        """Load final data to output path"""
        df.write.json(self.settings.output_path, mode='overwrite', encoding='utf-8')
        self.logger.info(f'Load data to {self.settings.output_path}')
```

每一个`Task`任务都会经过“输入”、“转换”和“输出”的过程，实现`AbstractTask`中的`_input`、`_transform`、`_output`抽象方法

- `_input`：读取`example_data/input`下csv文件
- `_transform`：执行将实现的`Transform`类的`transform`方法
- `_output`：将DataFrame以Json格式写入`example_data/output`目录下

### Transform类

创建汽车数据`CarTransform`类，`src/etl_project/tasks/car_etl_example/car_transform.py`

```py title="car_transform.py"
"""Car data Transformation."""
from functools import reduce

from pyspark.sql import DataFrame
from pyspark.sql.functions import col
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

from pyspark_etl_example.tasks.abstract.transform import AbstractTransform


class CarTransform(AbstractTransform):
    """Car data Transformation."""
    def transform(self, df: DataFrame) -> DataFrame:
        """Execute the transform process"""
        transformations = (
            self._filter_price,
            self._process_car_name,
            self._select_final_columns,
        )
        return reduce(DataFrame.transform, transformations, df)

    @staticmethod
    def _filter_price(df: DataFrame) -> DataFrame:
        """Filter results price > 1000"""
        return df.filter(col('price') > 1000)

    @staticmethod
    def _process_car_name(df: DataFrame) -> DataFrame:
        """Clean [dirty data] from CarName"""
        res_df = df.withColumn('CarName', _name_replace_udf(col('CarName')).alias('CarName'))
        return res_df

    @staticmethod
    def _select_final_columns(df: DataFrame) -> DataFrame:
        """Car price data dataframe select final columns"""
        return df.select(
            col('car_ID').alias('car_id'),
            col('symboling'),
            col('CarName').alias('car_name'),
            col('price'),
        )


@udf(returnType=StringType())
def _name_replace_udf(car_name):
    """Clean [dirty data] udf"""
    if not car_name:
        return None
    err_str = '[dirty data]'
    if err_str not in car_name:
        return car_name
    car_name = car_name.replace(err_str, '')
    return car_name
```

`CarTransform`类，实现以下方法：

- `_filter_price`的功能是筛选 `price` > 10000 的数据
- `_process_car_name`的功能是使用udf方法`_name_replace_udf`，将`CarName`字段中包含脏数据`[dirty_data]`的内容进行处理
- `_select_final_columns`方法是查询并返回`car_id`,`symboling`,`car_name`,`price`数据
- `transform`的功能是执行处理流程，返回数据结果，至此完成汽车数据转换过程

### 配置

将如下配置更新到配置文件中，因为项目默认使用dev环境配置，则需要在`configs/dev.toml`中增加如下内容：

```toml
# example path
input_path = '../../data/input/car_price.csv'
output_path = '../../data/output'
```

### 注册Task

将`main`命令行入口和上述实现的`Task`类注册到命名空间中。

编辑`pyproject.toml`文件，增加[poetry插件](https://python-poetry.org/docs/plugins/)如下内容：

```toml
[tool.poetry.plugins.console_scripts]
pyspark_etl_template = "pyspark_etl_template.app:main"

[tool.poetry.plugins."etl_tasks"]
car_etl_example = "pyspark_etl_template.tasks.car_etl_example.task:CarDataTask"
```

这么做的目的是将`CarDataTask`注册到`entry_points`中， 然后在程序中使用`importlib.metadata`
根据名称空间查找。而 `stevedore` 则是封装了查找的复杂逻辑，让使用插件更简单。

将项目以可编辑模式安装到当前环境：

```shell
poetry install
```

可以在 `Python Console` 下查看注册插件信息：

```bash
>>> from importlib.metadata import entry_points

>>> entry_points(group='etl_tasks')

[EntryPoint(name='car_etl_example', value='pyspark_etl_template.tasks.car_etl_example.car_transform:CarTransform', group='etl_tasks')]
```

将本地项目以可编辑方式安装到当前 Python 环境：

```shell
pip install -e .
```

### 运行Task

然后通过命令行的方式运行`Task`：

```shell
pyspark_etl_template --env=development --task=car_etl_example
```

