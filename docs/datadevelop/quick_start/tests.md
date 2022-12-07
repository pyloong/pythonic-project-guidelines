# 测试

## 单元测试

单元测试（unit test）就是编写测试来验证某一模块的功能正确性。一般会指定输入，验证输出是否符合预期，可以帮助我们很快准确的定位到问题的位置，出现问题的模块和单元。
我们将使用[Pytest](/pythonic-project-guidelines/guidelines/advanced/test/#12-pytest)进行测试。

### 配置文件测试

`test_settings` ：测试`dynaconf`配置是否使用`testing`环境配置

```python
@pytest.fixture()
def context():
    """Fixture context for the tests"""
    Context().environment = 'testing'
    ctx = Context()
    return ctx

def test_settings(context):
    """Test: Setting init by "testing" env"""
    settings = context.settings

    assert settings.message == 'This is in testing env'

    # tmp path
    assert settings.input_path == '../tmp/input/car_price.csv'
    assert settings.output_path == '../tmp/output'

    # spark configs
    assert settings.spark_master == 'local[*]'
    assert settings.spark_config.spark.driver.memory == '3G'
    assert settings.spark_config.spark.executor.memory == '16G'
    assert settings.spark_config.spark.sql.debug.maxToStringFields == 100
```

### 任务Extract测试

`test_extract` ：测试`car_price.csv`文件是否被正确读取

```python
def test_extract(context):
    """Test: Read CSV file return DataFrame"""
    task = AutomotiveDataTask()
    df = task._extract()

    assert df.count() == 205

    return df
```

### 任务Transform测试

`test_transform_filter_price` ：测试`automotive_data_etl`转换过程，是否过滤`price`字段`10000`以上的数据

```python
def test_transform_filter_price(test_extract):
    """Test: Filter results price > 10000"""
    transform = AutomotiveDataTransform()
    df = transform._filter_price(test_extract)

    assert df.filter(col('price') <= 10000).count() == 0

    return df
```

`test_transform_process_car_name` ：测试`CarName`列是否将`[dirty tmp]`字符串清除

```python
def test_transform_process_car_name(test_transform_filter_price):
    """Test: Clean [dirty tmp] from CarName"""
    transform = AutomotiveDataTransform()
    df = transform._process_car_name(test_transform_filter_price)

    assert df.filter(col('CarName').contains('[dirty tmp]')).count() == 0

    return df
```

`test_transform_select_final_columns` ：测试最终结果`Columns`是否为：`car_id`, `car_name`, `symboling`,  `price`

```python
def test_transform_select_final_columns(test_transform_process_car_name):
    """Test: Final columns is ['car_id', 'car_name', 'symboling',  'price']"""
    final_columns = ['car_id', 'car_name', 'symboling', 'price']
    transform = AutomotiveDataTransform()
    df = transform._select_final_columns(test_transform_process_car_name)
    names = df.schema.names

    assert names.sort() == final_columns.sort()

    return df
```

### 任务Load测试

`test_load` ：测试结果数据是否可以正确被写入`JSON`文件

```python
def test_load(test_transform_select_final_columns):
    """Test: Load csv file"""
    task = AutomotiveDataTask()
    task._load(test_transform_select_final_columns)
```

### 任务完整流程测试

创建测试文件`tests/test_task.py`

```python
"""Test log"""
import pytest
from pyspark.sql.functions import col

from automotive_data_etl.context import Context
from automotive_data_etl.tasks.automotive_task.automotive_transform import AutomotiveDataTransform
from automotive_data_etl.tasks.automotive_task.task import AutomotiveDataTask


@pytest.fixture()
def context():
    """Fixture context for the tests"""
    Context().environment = 'testing'
    ctx = Context()
    return ctx


def test_settings(context):
    """Test: Setting init by "testing" env"""
    settings = context.settings

    assert settings.message == 'This is in testing env'

    # tmp path
    assert settings.input_path == '../tmp/input/car_price.csv'
    assert settings.output_path == '../tmp/output'

    # spark configs
    assert settings.spark_master == 'local[*]'
    assert settings.spark_config.spark.driver.memory == '3G'
    assert settings.spark_config.spark.executor.memory == '16G'
    assert settings.spark_config.spark.sql.debug.maxToStringFields == 100


@pytest.fixture()
def test_extract(context):
    """Test: Read CSV file return DataFrame"""
    task = AutomotiveDataTask()
    df = task._extract()

    assert df.count() == 205

    return df


@pytest.fixture()
def test_transform_filter_price(test_extract):
    """Test: Filter results price > 10000"""
    transform = AutomotiveDataTransform()
    df = transform._filter_price(test_extract)

    assert df.filter(col('price') <= 10000).count() == 0

    return df


@pytest.fixture()
def test_transform_process_car_name(test_transform_filter_price):
    """Test: Clean [dirty tmp] from CarName"""
    transform = AutomotiveDataTransform()
    df = transform._process_car_name(test_transform_filter_price)

    assert df.filter(col('CarName').contains('[dirty tmp]')).count() == 0

    return df


@pytest.fixture()
def test_transform_select_final_columns(test_transform_process_car_name):
    """Test: Final columns is ['car_id', 'car_name', 'symboling',  'price']"""
    final_columns = ['car_id', 'car_name', 'symboling', 'price']
    transform = AutomotiveDataTransform()
    df = transform._select_final_columns(test_transform_process_car_name)
    names = df.schema.names

    assert names.sort() == final_columns.sort()

    return df


def test_load(test_transform_select_final_columns):
    """Test: Load csv file"""
    task = AutomotiveDataTask()
    task._load(test_transform_select_final_columns)

```

## 数据测试

### Step 1: 数据输入测试

1. 数据资源应该被验证，来确保正确的数据被加载进系统

2. 任务需求使用CSV文件数据源，加载内容与源数据进行匹配

### Step 2: Transform 阶段测试

1. 测试转换规则可正确执行

2. 转换数据量与被转换数据量是否匹配

### Step 3: 输出结果测试

1. 检查转换(Transformation)规则被正确应用

2. 通过其他方式计算源数据量进行比较
