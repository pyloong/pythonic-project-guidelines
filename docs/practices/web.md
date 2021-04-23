# 快速上手

这是一个快速上手的示例项目，旨在通过一个尽可能包含主要知识点的简单项目，来向使用者展示一个更 Python 化的项目开发流程。

示例项目是一个使用异步微 Web 框架 [Fastapi](https://fastapi.tiangolo.com/) 开发的博客系统。项目业务功能比较简单，但完整体现了一个项目从环境搭建，到开发，最后测试发布的完整流程。

## 1. 开发环境搭建

### 1.1 Python 环境

鉴于官方已经停止对 Python 2 的支持 [^1] ，我们不推荐再使用 Python 2 进行开发。根据当前 Python 版本使用情况，推荐使用 Python 3.7+ 。

具体的版本的 Python 环境可以在 [官网](https://www.python.org/downloads/) 下载。为了使用便利性，可以选择 Anaconda [^2] 。

### 1.2 开发工具

推荐使用 [Pycharm](https://www.jetbrains.com/pycharm/) 作为主要开发工具，可以选择社区版本免费使用。

[Visual Studio Code](https://code.visualstudio.com/) 是微软开发的一款免费轻量文本编辑器，通过安装插件可以自定义成一款功能强大的 IDE 。在对 Python 的支持上，已经有了较为完善的插件体系，此方案也可以作为备用。

### 1.3 虚拟环境工具

推荐使用 [pipenv](https://pipenv.pypa.io/en/latest/) 。Pipenv 相比使用 `requirements.txt` 管理依赖列表，更加强大。它支持同时管理开发生产环境依赖，自动查找虚拟环境，生成依赖锁定文件等其他特性。

在安装好 Python 环境后，应该在全局环境中安装 pipenv 。

### 1.4 Git 使用

推荐使用 [Git](https://git-scm.com/) 对项目进行版本管理。所以需要提前安装 Git ，并熟悉常用 Git 的概念和常用 Git 命令。

## 2. 项目初始化

### 2.1 初始化项目结构

项目结构采用 `src` 目录结构，详见 [pypa/sampleproject](https://github.com/pypa/sampleproject) 。

创建项目目录结构：

```txt
.
├── README.md
├── setup.cfg
├── setup.py
├── src
│   └── example_blog
│       └── __init__.py
└── tests
    └── __init__.py
```

初始化项目虚拟环境：

```bash
pipenv install
```

安装完成后，项目目录会自动生成 `Pipfile` 和 `Pipfile.lock` 两个文件。

### 2.2 初始化项目基本信息

编辑 `setup.py` 文件：

```python
import setuptools

setuptools.setup()

```

编辑 `setup.cfg` 文件，配置项目描述信息：

```ini
[metadata]
name = example_blog
version = attr: example_blog.__version__
author = huagang
author_email = huagang517@126.com
description = This is example blog system.
keywords = blog example
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.7

[options]
python_requires > = 3.7
include_package_data = True
packages = find:
package_dir =
    =src
install_requires =

[options.packages.find]
where = src

```

编辑 `src/example_blog/__init__.py` ，创建初始版本号：

```python
__version__ = '0.1.0'
```

### 2.3 增加项目自述文件

编写 `README.md` 文件

```markdown
# 一个简单博客系统示例.

此项目是一个简单的博客系统，提供一些用户管理和博客文章管理。目的是演示如何做一个更加 Pythonic 的项目。

如果您有任何意见和建议，欢迎开启 ISSUE 发起讨论。期待与您打造更加完美的 Python 示例。

## 协作开发

- Fork 仓库
- 编写代码，测试，提交
- 发起 PR
- 审核通过后合并，协作完成

```

### 2.4 增加 `.gitignore`

```txt
# Created by .ignore support plugin (hsz.mobi)
### Python template
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
cover/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
.pybuilder/
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
# .python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock

# PEP 582; used by e.g. github.com/David-OConnor/pyflow
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# pytype static type analyzer
.pytype/

# Cython debug symbols
cython_debug/

### Windows template
# Windows thumbnail cache files
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db

# Dump file
*.stackdump

# Folder config file
[Dd]esktop.ini

# Recycle Bin used on file shares
$RECYCLE.BIN/

# Windows Installer files
*.cab
*.msi
*.msix
*.msm
*.msp

# Windows shortcuts
*.lnk

### Linux template
*~

# temporary files which can be created if a process still has a handle open of a deleted file
.fuse_hidden*

# KDE directory preferences
.directory

# Linux trash folder which might appear on any partition or disk
.Trash-*

# .nfs files are created when an open file is removed but is still being accessed
.nfs*

### macOS template
# General
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon

# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

.vscode
.idea
```

### 2.6 初始 Git 提交

```bash
git init
git config user.name example
git config user.email example@example.com
git add .
git commit -m "feat: First commit!"
```

### 2.5 安装开发包

将项目以编辑方式安装到环境中：

```bash
pip install -e .
```

!!! warning "注意"
    这里不要使用 `pipenv` 命令，否则会写入 `Pipfile` 。

## 3. 项目功能开发

### 3.1 创建命令行入口

命令行入口是启动项目的主入口，常见的做法是使用一个 `__main__` 函数，调用启动代码，然后使用 `python` 命令启动该文件。但对于多级命令参数的情况就比较麻烦，推荐使用 [click](https://click.palletsprojects.com/en/7.x/) 工具编写入口逻辑。

安装依赖：

```bash
pipenv install click
```

编辑 `setup.cfg` ，将增加安装依赖：

```ini
install_requires =
    click
```

创建 `src/example_blog/cmdline.py` 文件：

```python
@click.group(invoke_without_command=True)
@click.pass_context
@click.option('-V', '--version', is_flag=True, help='Show version and exit.')
def main(ctx, version):
    if version:
        click.echo(__version__)
    elif ctx.invoked_subcommand is None:
        click.echo(ctx.get_help())

```

编辑 `setup.cfg` ，将命令行入口注册到项目描述文件中：

```ini
[options.entry_points]
console_scripts =
    example_blog = example_blog.cmdline:main
```

重新安装项目：

```bash
pip install -e .
```

测试命令是否生效：

```bash
example_blog -V
```

提交代码：

```bash
git add .
git commit -m "feat: Add commline."
```

### 3.2 引入项目配置系统

项目的配置系统是一个项目的核心驱动，使用配置系统便于管理散落在各处的配置参数，也方便在启动前通过调整配置，改变系统行为。

[Dynaconf](https://www.dynaconf.com/) 是一个高度灵活的配置管理工具，支持多环境分层，多种配置导入等有点。在项目开发中，推荐使用如下实践。

安装依赖：

```bash
pipenv install dynaconf
```

编辑 `setup.cfg` ，将增加安装依赖：

```ini
install_requires =
    click
    dynaconf
```

建立配置包，和配置文件：

```bash
mkdir src/example_blog/config
touch src/example_blog/config/__init__.py
touch src/example_blog/config/settings.yml
```

编辑 `src/example_blog/config/__init__.py` ， 初始化全局配置对象：

```python
import os
import sys
from pathlib import Path

from dynaconf import Dynaconf

_BASE_DIR = Path(__file__).parent.parent

settings_files = [
    Path(__file__).parent / 'settings.yml',
]  # 指定绝对路径加载默认配置

settings = Dynaconf(
    envvar_prefix="EXAMPLE_BLOG",  # 环境变量前缀。设置`EXAMPLE_BLOG_FOO='bar'`，使用`settings.FOO`
    settings_files=settings_files,
    environments=False,  # 启用多层次日志，支持 dev, pro
    load_dotenv=True,  # 加载 .env
    env_switcher="EXAMPLE_BLOG_ENV",  # 用于切换模式的环境变量名称 EXAMPLE_BLOG_ENV=production
    lowercase_read=False,  # 禁用小写访问， settings.name 是不允许的
    includes=[os.path.join(sys.prefix, 'etc', 'example_blog', 'settings.yml')],  # 自定义配置覆盖默认配置
    base_dir=_BASE_DIR,  # 编码传入配置
)

```

编辑 `src/example_blog/config/settings.yml` ，初始化配置：

```yaml
LOG_LEVEL: INFO
```

编辑 `src/example_blog/config/settings.local.yml` ，增加本地开发配置：

```yaml
LOG_LEVEL: DEBUG
```

根据 [Dynaconf](https://www.dynaconf.com/) 规则， `settings.local.yml` 的配置为本地配置，且优先级比 `settings.yml` 低，所以本地配置会在后面加载，覆盖之前的配置。

编辑 `.gitignore` ，将所有本地配置排除版本控制之外。

```txt
**/settings.local.yml
```

编辑 `setup.cfg` ，将默认配置文件包含打包范围之内：

```ini
[options.package_data]
example_blog.config = settings.yml
```

编辑 `setup.cfg` ，增加外部配置路径：

```ini
[options.data_files]
etc/example_blog = src/example_blog/config/settings.yml
```

**注意：**

`data_files` 中的相对路径会写入到 `sys.prefix` 位置下。当安装后，会在该目录下生成 `etc/example_blog/settings.yml` 的配置文件。可以通过修改该文件驱动项目配置。此文件属于用户级别配置。会覆盖默认配置。

提交代码。

```bash
git add .
git commit -m "feat: Add config."
```

### 3.3 引入日志

创建 `src/example_blog/log.py` ，初始化 log ：

```python
from logging.config import dictConfig

from example_blog.config import settings


def init_log():
    log_config = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'sample': {'format': '%(asctime)s %(levelname)s %(message)s'},
            'verbose': {'format': '%(asctime)s %(levelname)s %(name)s %(process)d %(thread)d %(message)s'},
            "access": {
                "()": "uvicorn.logging.AccessFormatter",
                "fmt": '%(asctime)s %(levelprefix)s %(client_addr)s - "%(request_line)s" %(status_code)s',
            },
        },
        'handlers': {
            "console": {
                "formatter": 'verbose',
                'level': 'DEBUG',
                "class": "logging.StreamHandler",
            },
        },
        'loggers': {
            '': {'level': settings.LOG_LEVEL, 'handlers': ['console']},
        },
    }

    dictConfig(log_config)

```

提交代码：

```bash
git add .
git commit -m "feat: Add log"
```

### 3.4 数据访问

数据层是应用的最底层，和数据存储打交道。使用 [sqlalchemy](https://www.sqlalchemy.org/) 作底层数据模型建模和数据访问操作。

安装依赖：

```bash
pipenv install sqlalchemy mysqlclient
```

编辑 `setup.cfg` ，将增加安装依赖：

```ini
install_requires =
    click
    dynaconf
    sqlalchemy
    mysqlclient
```

编写 `src/example_blog/config/settings.yml` ，增加数据库配置信息：

```yaml
# ######################################################################################################
# # https://docs.sqlalchemy.org/en/13/core/engines.html
DATABASE:
  DRIVER: mysql
  NAME: example_blog
  HOST: 127.0.0.1
  PORT: 3306
  USERNAME: root
  PASSWORD: root
  QUERY:
    charset: utf8mb4
```

!!! danger "警告"
    `settings.yml` 为系统默认配置，会被 git 追踪管理，不要填写真正的数据库连接信息。真实配置信息可以写在 `settings.local.yml` 文件中，会覆盖默认配置。

新建 `src/example_blog/db.py` ，创建 [sqlalchemy](https://www.sqlalchemy.org/) 访问对象：

```python
"""Database connections"""

from sqlalchemy.engine import create_engine
from sqlalchemy.engine.base import Engine
from sqlalchemy.engine.url import URL
from sqlalchemy.orm import scoped_session, sessionmaker

from example_blog.config import settings

url = URL(
    drivername=settings.DATABASE.DRIVER,
    username=settings.DATABASE.get('USERNAME', None),
    password=settings.DATABASE.get('PASSWORD', None),
    host=settings.DATABASE.get('HOST', None),
    port=settings.DATABASE.get('PORT', None),
    database=settings.DATABASE.get('NAME', None),
    query=settings.DATABASE.get('QUERY', None),
)

engine: Engine = create_engine(url, echo=True)

SessionFactory = sessionmaker(bind=engine, autocommit=False, autoflush=True)

ScopedSession = scoped_session(SessionFactory)

```

创建 `src/example_blog/models.py` ，创建数据模型：

```python
"""Models"""

from datetime import datetime

from sqlalchemy import Column, DateTime, Integer, String, Text
from sqlalchemy.ext.declarative import declarative_base, declared_attr


class CustomBase:
    """https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/mixins.html"""

    @declared_attr
    def __tablename__(cls):
        return cls.__name__.lower()

    __table_args__ = {
        'mysql_engine': 'InnoDB',
        'mysql_collate': 'utf8mb4_general_ci'
    }

    id = Column(Integer, primary_key=True, autoincrement=True)


BaseModel = declarative_base(cls=CustomBase)


class Article(BaseModel):
    """Article table"""
    title = Column(String(500))
    body = Column(Text(), nullable=True)
    create_time = Column(DateTime, default=datetime.now, nullable=False)
    update_time = Column(DateTime, default=datetime.now, onupdate=datetime.now, nullable=False)

```

为了在应用中更方便的使用数据模型对象，引入 [pydantic](https://pydantic-docs.helpmanual.io/) 来定义一些对象模型的基本信息。

安装依赖：

```bash
pipenv install pydantic
```

编辑 `setup.cfg` ，将增加安装依赖：

```ini
install_requires =
    click
    dynaconf
    sqlalchemy
    mysqlclient
    pydantic
```

创建 `src/example_blog/schemas.py` ，创建对象模型：

```python
from datetime import datetime
from typing import Optional, TypeVar

from pydantic import BaseModel, constr

from example_blog.models import BaseModel as DBModel

ModelType = TypeVar('ModelType', bound=DBModel)
CreateSchema = TypeVar('CreateSchema', bound=BaseModel)
UpdateSchema = TypeVar('UpdateSchema', bound=BaseModel)


class InDBMixin(BaseModel):
    id: int

    class Config:
        orm_mode = True


class BaseArticle(BaseModel):
    title: constr(max_length=500)
    body: Optional[str] = None


class ArticleSchema(BaseArticle, InDBMixin):
    create_time: datetime
    update_time: datetime


class CreateArticleSchema(BaseArticle):
    pass


class UpdateArticleSchema(BaseArticle):
    title: Optional[constr(max_length=500)] = None

```

创建 `src/example_blog/dao.py` ，创建数据访问层：

```python
from typing import Generic, List

from fastapi.encoders import jsonable_encoder
from sqlalchemy.orm import Session

from example_blog.models import Article
from example_blog.schemas import CreateSchema, ModelType, UpdateSchema, CreateArticleSchema, UpdateArticleSchema


class BaseDAO(Generic[ModelType, CreateSchema, UpdateSchema]):
    model: ModelType

    def get(self, session: Session, offset=0, limit=10) -> List[ModelType]:
        result = session.query(self.model).offset(offset).limit(limit).all()
        return result

    def get_by_id(self, session: Session, pk: int, ) -> ModelType:
        return session.query(self.model).get(pk)

    def create(self, session: Session, obj_in: CreateSchema) -> ModelType:
        """Create"""
        obj = self.model(**jsonable_encoder(obj_in))
        session.add(obj)
        session.commit()
        return obj

    def patch(self, session: Session, pk: int, obj_in: UpdateSchema) -> ModelType:
        """Patch"""
        obj = self.get_by_id(session, pk)
        update_data = obj_in.dict(exclude_unset=True)
        for key, val in update_data.items():
            setattr(obj, key, val)
        session.add(obj)
        session.commit()
        session.refresh(obj)
        return obj

    def delete(self, session: Session, pk: int) -> None:
        """Delete"""
        obj = self.get_by_id(session, pk)
        session.delete(obj)
        session.commit()

    def count(self, session: Session):
        return session.query(self.model).count()


class ArticleDAO(BaseDAO[Article, CreateArticleSchema, UpdateArticleSchema]):
    model = Article

```

提交代码：

```bash
git add .
git commit -m "feat: Add models and DAO"
```

### 3.5 服务层

创建 `src/example_blog/services.py` ，创建服务：

```python
"""Service"""
from typing import Generic, List

from sqlalchemy.orm import Session

from example_blog.dao import ArticleDAO, BaseDAO
from example_blog.models import Article
from example_blog.schemas import CreateSchema, ModelType, UpdateSchema


class BaseService(Generic[ModelType, CreateSchema, UpdateSchema]):
    dao: BaseDAO

    def get(self, session: Session, offset=0, limit=10) -> List[ModelType]:
        """"""
        return self.dao.get(session, offset=offset, limit=limit)

    def total(self, session: Session) -> int:
        return self.dao.count(session)

    def get_by_id(self, session: Session, pk: int) -> ModelType:
        """Get by id"""
        return self.dao.get_by_id(session, pk)

    def create(self, session: Session, obj_in: CreateSchema) -> ModelType:
        """Create a object"""
        return self.dao.create(session, obj_in)

    def patch(self, session: Session, pk: int, obj_in: UpdateSchema) -> ModelType:
        """Update"""
        return self.dao.patch(session, pk, obj_in)

    def delete(self, session: Session, pk: int) -> None:
        """Delete a object"""
        return self.dao.delete(session, pk)


class ArticleService(BaseService[Article, CreateSchema, UpdateSchema]):
    dao = ArticleDAO()

```

提交代码：

```bash
git add .
git commit -m "feat: Add services."
```

### 3.6 引入 Fastapi

[Fastapi](https://fastapi.tiangolo.com/) 是一个轻量的 Web 框架，现在引入，使其作为 API 层

安装依赖：

```bash
pipenv install fastapi uvicorn
```

编辑 `setup.cfg` ，增加安装依赖：

```ini
install_requires =
    click
    dynaconf
    sqlalchemy
    mysqlclient
    pydantic
    fastapi
    uvicorn
```

创建 `src/examp.e_blog/views.py` ，创建视图：

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from example_blog.dependencies import CommonQueryParams, get_db
from example_blog.schemas import (ArticleSchema, CreateArticleSchema,
                                  UpdateArticleSchema)
from example_blog.services import ArticleService

router = APIRouter()

_service = ArticleService()


@router.get('/articles')
def get(
        session: Session = Depends(get_db),
        commons: CommonQueryParams = Depends()
):
    return _service.get(session, offset=commons.offset, limit=commons.limit)


@router.get('/articles/{pk}')
def get_by_id(
        pk: int,
        session: Session = Depends(get_db)
):
    return _service.get_by_id(session, pk)


@router.post('/articles', response_model=ArticleSchema)
def create(
        obj_in: CreateArticleSchema,
        session: Session = Depends(get_db),
):
    return _service.create(session, obj_in)


@router.patch('/articles/{pk}', response_model=ArticleSchema)
def patch(
        pk: int,
        obj_in: UpdateArticleSchema,
        session: Session = Depends(get_db)
):
    return _service.patch(session, pk, obj_in)


@router.delete('/articles/{pk}')
def delete(
        pk: int,
        session: Session = Depends(get_db)
):
    return _service.delete(session, pk)

```

创建 `src/example_blog/middlewares.py` ，创建数据库会话中间件：

```python
from typing import Callable

from fastapi import FastAPI, Request, Response

from example_blog.db import SessionFactory


async def db_session_middleware(request: Request, call_next: Callable) -> Response:
    response = Response('Internal server error', status_code=500)
    try:
        request.state.db = SessionFactory()
        response = await call_next(request)
    finally:
        request.state.db.close()

    return response


def init_middleware(app: FastAPI) -> None:
    app.middleware('http')(db_session_middleware)

```

创建 `src/example_blog/dependencies.py` ，创建 Fastapi 的依赖项：

```python
from fastapi import Request
from sqlalchemy.orm import Session


def get_db(request: Request) -> Session:
    return request.state.db


class CommonQueryParams:
    def __init__(self, offset: int = 1, limit: int = 10):
        self.offset = offset - 1
        if self.offset < 0:
            self.offset = 0
        self.limit = limit

        if self.limit < 0:
            self.limit = 10
```

创建 `src/example_blog/routes.py` ，创建路由：

```python
from fastapi import APIRouter, FastAPI

from example_blog import views


def router_v1():
    router = APIRouter()
    router.include_router(views.router, tags=['Article'])
    return router


def init_routers(app: FastAPI):
    app.include_router(router_v1(), prefix='/api/v1', tags=['v1'])

```

创建 `src/example_blog/server.py` ，创建服务启动逻辑：

```python
"""server"""
import uvicorn
from fastapi import FastAPI

from example_blog import middlewares, routes
from example_blog.config import settings
from example_blog.log import init_log


class Server:

    def __init__(self):
        init_log()
        self.app = FastAPI()

    def init_app(self):
        middlewares.init_middleware(self.app)
        routes.init_routers(self.app)

    def run(self):
        self.init_app()
        uvicorn.run(
            app=self.app,
            host=settings.HOST,
            port=settings.PORT,
        )

```

修改 `src/example_blog/config/settings.yml` ，增加服务配置：

```yml
HOST: 127.0.0.1
PORT: 8000
```

提交代码：

```bash
git add .
git commit -m "feat: Add api service."
```

### 3.7 编写启动命令

编辑 `src/example_blog/cmdline.py` ，增加启动 Server 逻辑：

```python
@main.command()
@click.option('-h', '--host', show_default=True, help=f'Host IP. Default: {settings.HOST}')
@click.option('-p', '--port', show_default=True, type=int, help=f'Port. Default: {settings.PORT}')
@click.option('--level', help='Log level')
def server(host, port, level):
    """Start server."""
    kwargs = {
        'LOGLEVEL': level,
        'HOST': host,
        'PORT': port,
    }
    for name, value in kwargs.items():
        if value:
            settings.set(name, value)

    Server().run()

```

提交代码：

```bash
git add .
git commit -m "feat: Add server cmdline."
```

### 3.8 启动 Server

命令行运行：

```bash
example_blog server
```

可以看到如下输出：

```txt
INFO:     Started server process [21687]
2020-12-28 18:11:56,341 INFO uvicorn.error 21687 139772921304768 Started server process [21687]
INFO:     Waiting for application startup.
2020-12-28 18:11:56,341 INFO uvicorn.error 21687 139772921304768 Waiting for application startup.
INFO:     Application startup complete.
2020-12-28 18:11:56,341 INFO uvicorn.error 21687 139772921304768 Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
2020-12-28 18:11:56,341 INFO uvicorn.error 21687 139772921304768 Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

浏览器打开 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 即可查看接口文档。

提交代码

### 3.9 引入迁移工具

为了便于数据模型变更，引入 [alembic](https://alembic.sqlalchemy.org/en/latest/) 做数据库迁移。

安装依赖：

```bash
pipenv install alembic
```

编辑 `setup.cfg` ，将增加安装依赖：

```ini
install_requires =
    click
    dynaconf
    sqlalchemy
    mysqlclient
    pydantic
    fastapi
    uvicorn
    alembic
```

初始化 alembic ：

```bash
alembic init migration
mv alembic.ini src/example_blog/migration
```

将 alembic 的相关文件全部放到 `src/example_blog/migration` 目录中

修改 `src/example_blog/migration/alembic.ini` ：

```ini
# A generic, single database configuration.

[alembic]
# path to migration scripts
;script_location = src/example_blog/migration
script_location = .

# template used to generate migration files
# file_template = %%(rev)s_%%(slug)s

# timezone to use when rendering the date
# within the migration file as well as the filename.
# string value is passed to dateutil.tz.gettz()
# leave blank for localtime
# timezone =

# max length of characters to apply to the
# "slug" field
# truncate_slug_length = 40

# set to 'true' to run the environment during
# the 'revision' command, regardless of autogenerate
# revision_environment = false

# set to 'true' to allow .pyc and .pyo files without
# a source .py file to be detected as revisions in the
# versions/ directory
# sourceless = false

# version location specification; this defaults
# to src/example_blog/migration/versions.  When using multiple version
# directories, initial revisions must be specified with --version-path
# version_locations = %(here)s/bar %(here)s/bat src/example_blog/migration/versions

# the output encoding used when revision files
# are written from script.py.mako
# output_encoding = utf-8

;sqlalchemy.url = driver://user:pass@localhost/dbname


[post_write_hooks]
# post_write_hooks defines scripts or Python functions that are run
# on newly generated revision scripts.  See the documentation for further
# detail and examples

# format using "black" - use the console_scripts runner, against the "black" entrypoint
# hooks=black
# black.type=console_scripts
# black.entrypoint=black
# black.options=-l 79

# Logging configuration
[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S

```

修改 `src/example_blog/migration/env.py` ：

```python
from logging.config import fileConfig

from alembic import context
from sqlalchemy import engine_from_config, pool

from example_blog import db
from example_blog.models import BaseModel

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
# This line sets up loggers basically.
fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
# target_metadata = None

target_metadata = BaseModel.metadata


# other values from the config, defined by the needs of env.py,
# can be acquired:
# my_important_option = config.get_main_option("my_important_option")
# ... etc.


def run_migrations_offline():
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well.  By skipping the Engine creation
    we don't even need a DBAPI to be available.

    Calls to context.execute() here emit the given string to the
    script output.

    """
    context.configure(
        url=db.url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online():
    """Run migrations in 'online' mode.

    In this scenario we need to create an Engine
    and associate a connection with the context.

    """
    configuration = config.get_section(config.config_ini_section)
    configuration['sqlalchemy.url'] = str(db.url)
    connectable = engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()

```

编写 `src/example_blog/cmdline.py` ，创建迁移命令：

```python
from pathlib import Path

from alembic import config
from click import Context


@main.command()
@click.pass_context
@click.option('-h', '--help', is_flag=True)
@click.argument('args', nargs=-1)
def migrate(ctx: Context, help, args):
    """usage migrate -- arguments    """
    with utils.chdir(Path(__file__).parent / 'migration'):
        argv = list(args)
        if help:
            argv.append('--help')
        config.main(prog=ctx.command_path, argv=argv)

```

创建 `utils.py` ：

```python
"""Utils"""

import contextlib
import os
from os import PathLike
from typing import Union


@contextlib.contextmanager
def chdir(path: Union[str, PathLike]):
    cwd = os.getcwd()
    os.chdir(path)
    yield
    os.chdir(cwd)

```

!!! info "提示"
    由于使用了 click 包装了 alembic 命令，在使用上会有点不同，默认应该使用 `migrate --` 后加 alembic 的其他参数，否则多参数的情况下会无法识别。

为了将 `src/example_blog/migration` 打包到项目中，需要将其变成 Python 包。

创建 `src/example_blog/migration/__init__.py` 和 `src/example_blog/migration/versions/__init__.py`

编辑 `setup.cfg` ，将迁移脚本配置信息加入打包系统：

```ini
[options.package_data]
example_blog.config = settings.yml
example_blog.migration =
    alembic.ini
    README
    script.py.mako
```

创建空白数据库迁移版本：

```bash
example_blog migrate -- revision -m "init"
```

执行迁移：

```bash
example_blog migrate -- upgrade head
```

创建第一个数据库迁移版本：

```bash
example_blog migrate -- revision --autogenerate -m "init_table"
```

执行迁移：

```bash
example_blog migrate -- upgrade head
```

提交代码：

```bash
git add .
git commit -m "Add alembic migrate."
```

## 4. 测试和优化代码

测试是软件开发中重要的一环，能够在发布之前检查出更多可能出现的异常情况。

测试框架选用比较常用的 [pytest](https://docs.pytest.org/en/stable/) ，它具有强大的功能和很好的兼容性。

安装依赖：

```bash
pipenv install -d pytest
```

创建 `tests/settings.yml` ，初始化测试配置：

```yml
DEBUG: false
LOG_LEVEL: INFO

HOST: 127.0.0.1
PORT: 8000

DATABASE:
  DRIVER: mysql
  NAME: example_blog
  HOST: 127.0.0.1
  PORT: 3306
  USERNAME: root
  PASSWORD: root
  QUERY:
    charset: utf8mb4
```

编辑 `tests/__init__.py` ，加载测试配置：

```python
import os

from example_blog.config import settings

settings.load_file(os.path.join(os.path.dirname(__file__), 'settings.yml'))
settings.load_file(os.path.join(os.path.dirname(__file__), 'settings.local.yml'))

```

虽然本地开发配置可以临时调整，但对于开发环境和测试环境依然有些不一样。从上面代码中可以看到加载了两个测试配置，和 Dynaconf 规则一样， `settings.local.yml` 配置为本地配置，不会被代码追踪，只不过这里是手动实现的。

提交代码：

```bash
git add .
git commit -m "test: Init test."
```

### 4.1 测试数据访问层

编写测试配置：

新建 `tests/conftest.py` ，创建测试配置：

```python
"""Test config"""

import os
from pathlib import Path

import pytest
from alembic import command, config

from sqlalchemy.orm import Session

from example_blog import migration
from example_blog.config import settings
from example_blog.db import SessionFactory
from example_blog.models import Article


@pytest.fixture()
def migrate():
    """Re-init database when run a test."""
    os.chdir(Path(migration.__file__).parent)
    alembic_config = config.Config('./alembic.ini')
    alembic_config.set_main_option('script_location', os.getcwd())
    print('\n----- RUN ALEMBIC MIGRATION: -----\n')
    command.downgrade(alembic_config, 'base')
    command.upgrade(alembic_config, 'head')
    try:
        yield
    finally:
        command.downgrade(alembic_config, 'base')
        db_name = settings.DATABASE.get('NAME')
        if settings.DATABASE.DRIVER == 'sqlite' and os.path.isfile(db_name):
            try:
                os.remove(db_name)
            except FileNotFoundError:
                pass


@pytest.fixture()
def session(migrate) -> Session:
    """session fixture"""
    _s = SessionFactory()
    yield _s
    _s.close()


@pytest.fixture()
def init_article(session):
    """Init article"""
    a_1 = Article(title='Hello world', body='Hello world, can you see me?')
    a_2 = Article(title='Love baby', body='I love you everyday, and i want with you.')
    a_3 = Article(title='Tomorrow', body='When the sun rises, this day is fine day, cheer up.')
    session.add_all([a_1, a_2, a_3])
    session.commit()

```

编写数据访问层用例：

```python
import pytest

from example_blog.dao import ArticleDAO
from example_blog.models import Article
from example_blog.schemas import CreateArticleSchema, UpdateArticleSchema


class TestArticle:

    @pytest.fixture()
    def dao(self, init_article):
        yield ArticleDAO()

    def test_get(self, dao, session):
        users = dao.get(session)
        assert len(users) == 3
        users = dao.get(session, limit=2)
        assert len(users) == 2
        users = dao.get(session, offset=4)
        assert not users

    def test_get_by_id(self, dao, session):
        user = dao.get_by_id(session, 1)
        assert user.id == 1

    def test_create(self, dao, session):
        origin_count = session.query(dao.model).count()
        obj_in = CreateArticleSchema(title='test')
        dao.create(session, obj_in)
        count = session.query(dao.model).count()
        assert origin_count + 1 == count

    def test_patch(self, dao, session):
        obj: Article = session.query(dao.model).first()
        body = obj.body
        obj_in = UpdateArticleSchema(body='test')
        updated_obj: Article = dao.patch(session, obj.id, obj_in)
        assert body != updated_obj.body

    def test_delete(self, dao, session):
        origin_count = session.query(dao.model).count()
        dao.delete(session, 1)
        count = session.query(dao.model).count()
        assert origin_count - 1 == count

    def test_count(self, dao, session):
        count = dao.count(session)
        assert count == 3

```

运行测试：

```bash
pytest tests/test_dao.py
```

如果运行成功，则测试正确。

提交代码：

```bash
git add .
git commit -m "test: Add dao test."
```

### 4.2 测试服务层

创建 `tests/test_services.py` ，创建测试用例：

```python
import pytest

from example_blog.schemas import CreateArticleSchema, UpdateArticleSchema
from example_blog.services import ArticleService


class TestArticleService:

    @pytest.fixture()
    def service(self, init_article):
        yield ArticleService()

    def test_get(self, service, session):
        objs = service.get(session)
        assert len(objs) == 3
        objs = service.get(session, limit=2)
        assert len(objs) == 2
        objs = service.get(session, offset=5)
        assert not objs

    def test_total(self, service, session):
        total = service.total(session)
        assert total == 3

    def test_by_id(self, service, session):
        __obj = session.query(service.dao.model).first()
        obj = service.get_by_id(session, __obj.id)
        assert obj.id == __obj.id

    def test_create(self, service, session):
        origin_count = service.total(session)
        obj_in = CreateArticleSchema(title='test')
        service.create(session, obj_in)
        count = service.total(session)
        assert origin_count + 1 == count

    def test_patch(self, service, session):
        origin_obj = session.query(service.dao.model).first()
        body = origin_obj.body
        obj_in = UpdateArticleSchema(body='test')
        obj = service.patch(session, origin_obj.id, obj_in)
        assert body != obj.body

    def test_delete(self, service, session):
        origin_count = service.total(session)
        obj = session.query(service.dao.model).first()
        service.delete(session, obj.id)
        count = service.total(session)
        assert origin_count - 1 == count

```

运行测试：

```bash
pytest tests/test_services.py
```

如果运行成功，则测试正确。

提交代码：

```bash
git add .
git commit -m "test: Add service test."
```

### 4.3 测试试图层

编辑 `tests/conftest.py` ，创建测试配置：

```python
from fastapi.testclient import TestClient

from example_blog import migration, server



@pytest.fixture
def client():
    """Fast api test client factory"""
    _s = server.Server()
    _s.init_app()
    _c = TestClient(app=_s.app)
    yield _c

```

由于 Fastapi 的 `TestClient` 依赖 `requests` ，所以需要先安装：

```bash
pipenv install -d requests
```

创建 `tests/test_views.py` ，测试试图：

```python
import pytest
from fastapi.encoders import jsonable_encoder
from fastapi.responses import Response

from example_blog.models import Article
from example_blog.schemas import ModelType


def test_docs(client):
    """Test view"""
    response = client.get('/docs')
    assert response.status_code == 200


class BaseTest:
    version = 'v1'
    base_url: str
    model: ModelType

    @pytest.fixture()
    def init_data(self):
        pass

    def url(self, pk: int = None) -> str:
        url_split = ['api', self.version, self.base_url]
        if pk:
            url_split.append(str(pk))
        return '/'.join(url_split)

    def assert_response_ok(self, response: Response):
        assert response.status_code == 200

    def test_get(self, client, session, init_data):
        count = session.query(self.model).count()
        response = client.get(self.url())
        self.assert_response_ok(response)
        assert count == len(response.json())

    def test_get_by_id(self, client, session, init_data):
        obj = session.query(self.model).first()
        response = client.get(self.url(obj.id))
        self.assert_response_ok(response)
        assert jsonable_encoder(obj) == response.json()

    def test_delete(self, client, session, init_data):
        count = session.query(self.model).count()
        session.close()
        response = client.delete(self.url(1))
        self.assert_response_ok(response)
        after_count = session.query(self.model).count()
        assert after_count == 2
        assert count - 1 == after_count


class TestArticle(BaseTest):
    model = Article
    base_url = 'articles'

    @pytest.fixture()
    def init_data(self, init_article):
        pass

    def test_create(self, client, session, init_data):
        response = client.post(
            self.url(),
            json={'title': 'xxx'}
        )
        self.assert_response_ok(response)
        assert response.json().get('title') == 'xxx'

    def test_patch(self, client, session, init_data):
        obj = session.query(Article).first()
        response = client.patch(self.url(obj.id), json={'body': 'xxx'})
        self.assert_response_ok(response)
        assert response.json().get('body') != obj.body

```

运行测试：

```bash
pytest tests/test_views.py
```

如果运行成功，则测试正确。

提交代码：

```bash
git add .
git commit -m "test: Add view test."
```

### 4.4 测试命令行

编辑 `tests/conftest.py` ，创建测试配置：

```python
from click.testing import CliRunner



@pytest.fixture
def cli():
    runner = CliRunner(echo_stdin=True, mix_stderr=False)
    yield runner

```

创建 `tests/test_cmdline.py` ，创建测试用例：

```python
import uvicorn
from alembic import config

import example_blog
from example_blog import cmdline


def test_main(cli):
    result = cli.invoke(cmdline.main)
    assert result.exit_code == 0
    result = cli.invoke(cmdline.main, '-V')
    assert result.exit_code == 0
    assert str(result.output).strip() == example_blog.__version__


def test_run(cli, mocker):
    mock_run = mocker.patch.object(uvicorn, 'run')
    result = cli.invoke(cmdline.main, ['server', '-h', '127.0.0.1', '-p', '8080'])
    assert result.exit_code == 0
    mock_run.assert_called_once_with(app=mocker.ANY, host='127.0.0.1', port=8080)


def test_migrate(cli, mocker):
    mock_main = mocker.patch.object(config, 'main')
    cli.invoke(cmdline.main, ['migrate', '--help'])
    mock_main.assert_called_once()

```

因为单元测试中使用了 [mock](https://docs.python.org/3/library/unittest.mock.html) ，所以安装配合 pytest 使用的 [pytest-mock](https://github.com/pytest-dev/pytest-mock/)

```bash
pipenv install -d pytest-mock
```

运行测试：

```bash
pytest tests/test_views.py
```

如果运行成功，则测试正确。

提交代码：

```bash
git add .
git commit -m "test: Add cmdline test."
```

### 4.5 其他测试

创建 `tests/test_dependencies.py` ，创建测试用例：

```python
import pytest

from example_blog.dependencies import CommonQueryParams


@pytest.mark.parametrize(
    ['args', 'expect_value'],
    [
        ((), (0, 10)),
        ((0,), (0, 10)),
        ((-10, -10), (0, 10)),
        ((5, 100), (4, 100)),
    ]
)
def test_common_query_params(args, expect_value):
    params = CommonQueryParams(*args)
    assert params.offset == expect_value[0]
    assert params.limit == expect_value[1]

```

创建 `tests/test_utils.py` ，创建测试用例：

```python
import os

from example_blog.utils import chdir


def test_chdir():
    path = '/tmp'
    cwd = os.getcwd()
    with chdir(path):
        assert path == os.getcwd()
    assert cwd == os.getcwd()

```

运行测试：

```bash
pytest
```

如果运行成功，则测试正确。

提交代码：

```bash
git add .
git commit -m "test: Add other test."
```

至此，所有测试运行完毕，除了 `src/example_blog/migration` 之外的包的测试已经可以全部覆盖。

### 4.6 优化代码

代码风格和代码规范是一个开发人员开发修养的体现，好的代码能够让人眼前一亮。为了规范，社区开发许多工具用于检测代码。

#### 4.6.1 优化导入

[isort](https://pycqa.github.io/isort/) 是一个自动格式化导入的工具。

安装依赖：

```bash
pipenv install -d isort
```

格式化代码：

```bash
isort .
```

此时可以不用先急着提交，在后面对代码风格检测的时候可能还会再次格式化代码。

#### 4.6.2 优化代码风格

[flake8](https://flake8.pycqa.org/en/latest/) 是一个遵循 [PEP8](https://www.python.org/dev/peps/pep-0008/) 规范检测代码的工具。使用该工具，可以检测出哪些代码不符合 PEP8 规范。

安装依赖：

```bash
pipenv install -d flake8
```

检测代码：

```bash
flake8
```

根据输出提示，参照 [flake8 规则](https://www.flake8rules.com/) 进行调整，直至完全符合为止。

提交代码：

```bash
git add .
git commit -m "feat: Lint code"
```

## 5. 打包发布

到这一步， `setup.cfg` 文件应该是这样的：

```ini
[metadata]
name = example_blog
version = attr: example_blog.__version__
author = huagang
author_email = huagang517@126.com
description = This is example blog system.
keywords = blog example
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Operating System :: OS Independent
    Programming Language :: Python :: 3.7

[options]
python_requires > = 3.7
include_package_data = True
packages = find:
package_dir =
    =src
install_requires =
    click
    dynaconf
    sqlalchemy
    mysqlclient
    pydantic
    fastapi
    uvicorn
    alembic

[options.packages.find]
where = src

[options.entry_points]
console_scripts =
    example_blog = example_blog.cmdline:main

[options.package_data]
example_blog.config = settings.yml
example_blog.migration =
    alembic.ini
    README
    script.py.mako

[options.data_files]
etc/example_blog = src/example_blog/config/settings.yml
```

在整个开发过程中，是逐步丰富此文件的。这是项目的描述文件，描述了打包的配置信息。

### 5.1 打包

```bash
python setup.py sdist bdist_wheel
```

在 `dist` 目录中可以看到两个文件，一个是 `.tar.gz` 的源码打包文件，一个是 `.whl` 的二进制文件。

### 5.2 发布

将开发好的项目发布到索引仓库，或内网的私有仓库。

使用 [twine](https://twine.readthedocs.io/en/latest/) 上传：

```bash
twine upload dist/*
```

[^1]: https://www.python.org/doc/sunset-python-2/
[^2]: 现在 Anaconda / Miniconda 在 Windows 上使用虚拟环境工具 [Virtualenv](https://virtualenv.pypa.io/en/latest/) 存在一些兼容问题，而且 Pipenv 是依赖这个工具的。请参考 [conda support - Windows 3.7+ #1986](https://github.com/pypa/virtualenv/issues/1986) 和 [virtualenv==20.0.34 not compatible with python on windows #12094](https://github.com/ContinuumIO/anaconda-issues/issues/12094)
