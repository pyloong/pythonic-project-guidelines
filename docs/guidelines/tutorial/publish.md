# 打包发布

项目开发测试完成后，可以将项目发布到 Pypi 仓库中，然后在其他地方通过 Pip 命令即可
安装使用。对于一些工具包比较方便。

## 打包

根据 PEP 517 规范，新的打包机制可通过 `poetry` 工具来操作。

安装打包工具：

```bash
pip install poetry
```

然后运行打包命令：

```bash
poetry build
```

## 发布

```bash
poetry publish
```

输入账号密码后，即可将项目发布到 Pypi

> 注意：由于 Pypi 上的项目名称是唯一的， `example_etl` 名称已经被使用，所以你在测试时，需要使用其他项目名称。

## 安装测试

项目正常发布后，可以通过 pip 安装到本地使用：

```bash
pip install example-etl
```

输出结果如下：

```text
❯ pip install example-etl
Collecting example-etl
  Downloading example_etl-0.0.1.dev0-py3-none-any.whl (14 kB)
Collecting dynaconf==3.1.7
  Downloading dynaconf-3.1.7-py2.py3-none-any.whl (200 kB)
     |████████████████████████████████| 200 kB 850 kB/s            
Collecting stevedore==3.5.0
  Downloading stevedore-3.5.0-py3-none-any.whl (49 kB)
     |████████████████████████████████| 49 kB 747 kB/s            
Collecting click==8.0.3
  Downloading click-8.0.3-py3-none-any.whl (97 kB)
     |████████████████████████████████| 97 kB 554 kB/s            
Collecting pbr!=2.1.0,>=2.0.0
  Downloading pbr-5.8.0-py2.py3-none-any.whl (112 kB)
     |████████████████████████████████| 112 kB 1.9 MB/s            
Installing collected packages: pbr, stevedore, dynaconf, click, example-etl
Successfully installed click-8.0.3 dynaconf-3.1.7 example-etl-0.0.1.dev0 pbr-5.8.0 stevedore-3.5.0

```
