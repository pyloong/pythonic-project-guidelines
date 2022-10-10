# 应用开发

提供ETL工程化的项目示例，帮助初学者快速理解和学习ETL完整的工程化开发。

## 任务需求

现有汽车信息数据[car_price.csv](../../assets/data/car_price.csv)

- 对`CarName`字段包含`[dirty data]`数据进行纠正，去除字符串`[dirty data]`

- 删除`price`小于`10000`的汽车数据

- 最终结果Schema：`car_id`, `symboling`, `car_name`, `price`, 将结果导出`json`文件

在命令行使用`cookiecutter`创建项目骨架:

```text
❯ cookiecutter https://github.com/pyloong/cookiecutter-pythonic-project-bigdata-etl
project_name [My Project]: Automotive Data Etl
project_slug [automotive_data_etl]:
project_description [My Awesome Project!]: This is my first etl package, i love it.
author_name [Author]: ming
author_email [ming@example.com]: ming@gmail.com
version [0.1.0]:
Select python_version:
1 - 3.10
2 - 3.9
Choose from 1, 2 [1]:
use_src_layout [y]:
use_poetry [y]:
use_docker [n]:
Select ci_tools:
1 - none
2 - Gitlab
3 - Github
Choose from 1, 2, 3 [1]:
Select use_framework:
1 - none
2 - pyspark
Choose from 1, 2 [1]: 2
```

## Task类

创建汽车数据ETL任务`AutomotiveDataTask`类，`src/automotive_data_etl/tasks/automotive_task/task.py`

```py title="task.py"
"""Processing car data task."""
import logging

from pyspark.sql import DataFrame

from automotive_data_etl.tasks.abstract.task import AbstractTask
from automotive_data_etl.tasks.automotive_task.automotive_transform import AutomotiveDataTransform


class AutomotiveDataTask(AbstractTask):
    """Processing car data task."""

    def __init__(self):
        super().__init__()
        self.spark = self.ctx.get_spark_session()

    def _extract(self) -> DataFrame:
        """Read CSV file return DataFrame"""
        df: DataFrame = self.spark.read.csv(
            self.settings.input_path,
            encoding='utf-8',
            header=True,
            inferSchema=True
        )
        this.logger.info(f'Extract data from {self.settings.input_path}')
        return df

    def _transform(self, df: DataFrame) -> DataFrame:
        """execute CarTransform transform function"""
        return AutomotiveDataTransform().transform(df)

    def _load(self, df: DataFrame) -> None:
        """Load final data to output path"""
        df.write.json(self.settings.output_path, mode='overwrite', encoding='utf-8')
        this.logger.info(f'Load data to {self.settings.output_path}')

```

每一个`Task`任务都会经过“输入”、“转换”和“输出”的过程，实现`AbstractTask`中的`_extract`、`_transform`、`_load`抽象方法

- `_extract`：读取`tmp/input/car_price.csv`CSV文件
- `_transform`：执行将实现的`Transform`类的`transform`方法
- `_load`：将DataFrame以Json格式写入`tmp/output`目录下

## Transform类

创建汽车数据`AutomotiveDataTransform`类，`src/automotive_data_etl/tasks/automotive_task/automotive_transform.py`

```py title="automotive_transform.py"
"""Car data Transformation."""
from functools import reduce

from pyspark.sql import DataFrame
from pyspark.sql.functions import col
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

from automotive_data_etl.tasks.abstract.transform import AbstractTransform


class AutomotiveDataTransform(AbstractTransform):
    """Car data Transformation."""

    def transform(self, df: DataFrame) -> DataFrame:
        """Execute the transform process"""
        transformations = (
            self._filter_price,
            self._process_car_name,
            self._select_final_columns,
        )
        return reduce(DataFrame.transform, transformations, df)  # type: ignore

    @staticmethod
    def _filter_price(df: DataFrame) -> DataFrame:
        """Filter results price > 10000"""
        return df.filter(col('price') > 10000)

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

`AutomotiveDataTransform`类，实现以下方法：

- `_filter_price`的功能是筛选 `price` > 10000 的数据
- `_process_car_name`的功能是使用udf方法`_name_replace_udf`，将`CarName`字段中包含脏数据`[dirty_data]`的内容进行处理
- `_select_final_columns`方法是查询并返回`car_id`,`symboling`,`car_name`,`price`数据
- `transform`的功能是执行处理流程，返回数据结果，至此完成汽车数据转换过程

## 配置

将如下配置更新到配置文件中，因为项目默认使用dev环境配置，则需要在`configs/dev.toml`中增加如下内容：

```toml
# spark configs
spark_master = 'local[*]'
spark_config.spark.driver.memory = '3G'
spark_config.spark.executor.memory = '16G'
spark_config.spark.sql.debug.maxToStringFields = 100
```
