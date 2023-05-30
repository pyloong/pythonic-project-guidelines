# 打包发布

本章节，你可以学到：

- 使用 poetry 构建项目
- 使用 poetry 将项目发布到 pypi
- 安装并使用你发布到 pypi 的项目

项目开发测试完成后，可以将项目发布到 Pypi 仓库中，然后在其他地方通过 Pip 命令即可
安装使用。对于一些工具包比较方便。

## 打包

根据 PEP 517 规范，新的打包机制可通过 `poetry` 工具来操作。

执行构建命令：

```bash
$ poetry build
Building example_etl (0.1.0)
  - Building sdist
  - Built example_etl-0.1.0.tar.gz
  - Building wheel
  - Built example_etl-0.1.0-py3-none-any.whl

```

## 发布

本项目为测试项目，以下操作可以将项目发布到 [https://test.pypi.org/](https://test.pypi.org/) 。

首先在 [https://test.pypi.org/](https://test.pypi.org/) 注册账号。

然后配置 poetry 发布的仓库：

```bash
poetry config repositories.testpypi https://test.pypi.org/legacy/
```

然后填写根据你的用户名密码修改如下命令后，将项目发布到 testpypi 上。

```bash
poetry publish --repository=testpypi --username=USERNAME --password=PASSWORD
```

> 注意：不建议将测试项目发布都 [https://pypi.org/](https://pypi.org/) 上。在 [https://pypi.org/](https://pypi.org/) 上的项目名称是全局唯一的，
> 所以如果你考虑到将项目发布到 [https://pypi.org/](https://pypi.org/) 上前，应定一个不存在名称。

## 安装测试

项目正常发布后，可以通过 pip 安装到本地使用：

```bash
pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple example-etl
```

由于我们使用的是测试仓库，所以需要指定 `--index-url https://test.pypi.org/simple/` 参数，用来安装发布到 [https://test.pypi.org/](https://test.pypi.org/) 的包
同时使用 `--extra-index-url https://pypi.org/simple` 参数，来安装 `example-etl` 依赖的包。

输出结果如下：

```text
❯ pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple example-etl
Looking in indexes: https://test.pypi.org/simple/, https://pypi.org/simple
Collecting example-etl
  Downloading https://test-files.pythonhosted.org/packages/f3/51/8cea9e34ae2f0e48abb6a0aa58cdf29d4d2900bdd97e45b8d4ee24b357f0/example_etl-0.1.0-py3-none-any.whl (9.5 kB)
Collecting stevedore<6.0.0,>=5.1.0
  Downloading stevedore-5.1.0-py3-none-any.whl (49 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 49.6/49.6 kB 195.3 kB/s eta 0:00:00
Collecting click<9.0.0,>=8.1.3
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 389.6 kB/s eta 0:00:00
Collecting dynaconf<4.0.0,>=3.1.12
  Downloading dynaconf-3.1.12-py2.py3-none-any.whl (211 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 211.8/211.8 kB 878.8 kB/s eta 0:00:00
Collecting pbr!=2.1.0,>=2.0.0
  Downloading pbr-5.11.1-py2.py3-none-any.whl (112 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 112.7/112.7 kB 1.7 MB/s eta 0:00:00
Installing collected packages: pbr, dynaconf, click, stevedore, example-etl
Successfully installed click-8.1.3 dynaconf-3.1.12 example-etl-0.1.0 pbr-5.11.1 stevedore-5.1.0
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
