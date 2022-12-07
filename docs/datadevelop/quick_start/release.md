# 发布

## 本地运行

### 注册Task

将`main`命令行入口和上述实现的`Task`类注册到命名空间中。

编辑`pyproject.toml`文件，增加[poetry插件](https://python-poetry.org/docs/plugins/)如下内容：

```toml
[tool.poetry.plugins.console_scripts]
automotive_data_etl = "automotive_data_etl.cmdline:main"

[tool.poetry.plugins."etl_tasks"]
automotive_task = "automotive_data_etl.tasks.automotive_task.task:AutomotiveDataTask"
```

这么做的目的是将`AutomotiveDataTask`注册到`entry_points`中， 然后在程序中使用`importlib.metadata`
根据名称空间查找。而 `stevedore` 则是封装了查找的复杂逻辑，让使用插件更简单。

将项目以可编辑模式安装到当前环境：

```shell
poetry install
```

可以在 `Python Console` 下查看注册插件信息：

```bash
>>> from importlib.metadata import entry_points

>>> entry_points(group='etl_tasks')

[EntryPoint(name='automotive_task', value='automotive_data_etl.tasks.automotive_task.task:AutomotiveDataTask', group='etl_tasks')]
```

将本地项目以可编辑方式安装到当前 Python 环境：

```shell
pip install -e .
```

### 运行Task

然后通过命令行的方式运行`Task`，通过命令行参数的方式更新输入输出路径：

```shell
automotive_data_etl \
  --env=development \
  --task=automotive_task \
  --input=tmp/input/car_price.csv \
  --output=tmp/output/
```

<!--- TODO
## Spark运行

打包好一个应用程序之后，就可以使用 `bin/spark-submit` 脚本来提交它。这个脚本会负责设置 `Spark` 及其依赖的 `classpath`
，同时它可以支持 `Spark` 所支持的所有不同类型的集群管理器和部署模式。

将`automotive_data_etl`程序打包，在用户目录下执行：

```shell
poetry build
```

打包后生成`dist`目录，目录下包含`automotive_data_etl-0.1.0.tar.gz`和`automotive_data_etl-0.1.0-py3-none-any.whl`文件。

接下来可以使用`spark-submit`运行程序：

```shell
./bin/spark-submit --master k8s://HOST:PORT --py-files automotive_data_etl-0.1.0-py3-none-any.whl src/automotive_data_etl/cmdline.py \
  --env=prod \
  --task=automotive_task \
  --input=tmp/input/car_price.csv \
  --output=tmp/output/
```
-->