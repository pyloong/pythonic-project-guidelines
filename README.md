# Python 项目工程化开发指南

![GitHub Workflow Status (branch)](https://img.shields.io/github/workflow/status/pyloong/pythonic-project-guidelines/gh-page/main?label=gh-page&logo=github&style=flat-square)

文档目标：

以通俗易懂结构清晰的文档向读者展示如何做 Python 工程化

受众目标：

- Python 初学者
- Python 初级开发
- Python 中级开发

指南主要包含以下主题：


- [x] 快速上手（一个最通用，最初级的示例项目）
- [x] 开发前准备
    - [x] Python 环境的安装
    - [x] 虚拟环境管理
    - [x] IDE 的选择
- [x] Python 规范
    - [x] 风格规范
    - [x] 语言规范
- [x] 项目工程化
    - [x] 初级教程(一个包含完整开发流程的示例项目)
        - [x] 初始化项目
        - [x] 功能开发
        - [x] 测试
        - [x] 打包发布
    - [x] 进阶教程
        - [x] 类型标注
        - [x] 使用配置系统
        - [x] 如何用好日志
        - [x] 异常管理
        - [x] 如何更好得测试
        - [x] 用信号解耦逻辑
        - [x] 支持插件化
    - [x] 项目管理
        - [x] 代码检测
        - [x] 项目结构
        - [x] 文档管理
        - [x] 打包发布
- [ ] 开发实践
    - [ ] Web
        - [x] Fastapi
        - [ ] Django
        - [ ] Flask
    - [ ] 爬虫
        - [ ] Scrapy
        - [ ] aiohttp
    - [ ] 数据库
        - [ ] SQLALchemy

> 如果您对文档有任何建议或意见，欢迎提交 [issues](https://github.com/pyloong/pythonic-project-guidelines/issues) 进行讨论。当然我们更期待与您共同协作开发，让文档变得更加完善。

## 使用方式

### 1. 克隆项目

```bash
git clone https://github.com/pyloong/pythonic-project-guidelines
```

### 2. 初始化环境

项目预览需要安装 Python 环境来启动 server，强烈建议使用 Python 3.9+ 的版本。如果本地没有 Python 环境，也可以使用 [Docker预览服务器](https://squidfunk.github.io/mkdocs-material/creating-your-site/#creating-your-site) 来启动。

#### 2.1 本地初始化

创建虚拟环境：

```
python3 -m venv .venv
source .venv/bin/activate
```

安装依赖：

```bash
pip install -r requirements.txt
```

#### 2.2 使用 Docker 初始化

```bash
docker pull squidfunk/mkdocs-material:8.4.1
```

### 3. 预览

#### 3.1 本地预览

```bash
mkdocs serve
```

#### 3.2 使用 Docker 预览

**uinx**:

```bash
docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material:8.4.1
```

**Windows**:

```bash
docker run --rm -it -p 8000:8000 -v "%cd%":/docs squidfunk/mkdocs-material:8.4.1
```

## 协作规范

文档使用 Markdown 编写，使用 [mkdocs](https://www.mkdocs.org/) 配合 [mkdocs-material](https://squidfunk.github.io/mkdocs-material/) 主题构建。

- form
- code
- pr
