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

首先创建一个 `example-doc` 的目录，然后初始化项目虚拟环境：

```text
❯ mkdir example-doc
❯ cd example-doc
❯ pipenv install mkdocs
Creating a virtualenv for this project...
Pipfile: /tmp/test/example-doc/Pipfile
Using /usr/local/bin/python3.9 (3.9.7) to create virtualenv...
⠸ Creating virtual environment...created virtual environment CPython3.9.7.final.0-64 in 140ms
  creator CPython3Posix(dest=/home/kevin/.virtualenvs/example-doc-dabLH6DG, clear=False, no_vcs_ignore=False, global=False)                                        
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/kevin/.local/share/virtualenv)                      
    added seed packages: pip==21.3.1, setuptools==58.5.3, wheel==0.37.0                                                                                            
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator                                                      
                                                                                                                                                                   
✔ Successfully created virtual environment! 
Virtualenv location: /home/kevin/.virtualenvs/example-doc-dabLH6DG
Creating a Pipfile for this project...
Installing mkdocs...
Adding mkdocs to Pipfile's [packages]...
✔ Installation Succeeded 
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success! 
Updated Pipfile.lock (493bb6)!
Installing dependencies from Pipfile.lock (493bb6)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

进入到虚拟环境后，初始化文档配置：

```text
❯ mkdocs new .
INFO     -  Writing config file: ./mkdocs.yml
INFO     -  Writing initial docs: ./docs/index.md
❯ ls
docs  mkdocs.yml  Pipfile  Pipfile.lock

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
pipenv install mkdocs-material
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
