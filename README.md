# Python 项目工程化实践指南

指南主要包含以下标题：

- 项目工程化
  - [ ] 项目结构
  - [ ] 代码风格
  - [ ] 开发规范
  - [ ] 类型标注
  - [ ] 配置管理
  - [ ] 日志使用
  - [ ] 自动化测试
  - [ ] 打包与发布

- 开发实践
  - [ ] 通用项目实践
  - [ ] 信号的使用
  - [ ] 插件化机制
  - [ ] Scrapy 开发实践
  - [ ] SQLAlchemy 实践
  - [ ] Fastapi 实践
  - [ ] Django 实践

> 如果您对文档有任何建议或意见，欢迎提交 ISSUE 。当然我们更期待与您共同协作开发，让文档变得更加完善。

## 使用方式

### 1. 克隆项目

```bash
git clone xxx
```

### 2. 初始化环境

项目预览所启动的 server 是需要安装 Python 依赖的，强烈建议使用 Python 3.6+ 的版本。如果本地没有 Python 环境，也可以在 [Docker 中启动预览服务器](https://squidfunk.github.io/mkdocs-material/creating-your-site/#creating-your-site)。

#### 2.1 本地初始化

使用 Pipenv 或者其他虚拟环境管理工具

```bash
pipenv install
```

#### 2.2 使用 Docker 初始化

```bash
docker pull squidfunk/mkdocs-material
```

### 3. 预览

#### 3.1 本地预览

```bash
mkdocs serve
```

#### 3.2 使用 Docker 预览

**uinx**:

```bash
docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```

**Windows**:

```bash
docker run --rm -it -p 8000:8000 -v "%cd%":/docs squidfunk/mkdocs-material
```

## 协作规范

文档采用使用 Markdown 编写，使用 [mkdocs](https://www.mkdocs.org/) 配合 [mkdocs-material](https://squidfunk.github.io/mkdocs-material-insiders/) 主题构建。

- Fork
- 编码
- PR
- Review
- 合并
