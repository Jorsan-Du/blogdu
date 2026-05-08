---
title: 使用 Hexo 框架搭建个人博客的笔记
date: 2026-05-08 15:04:54
categories:
  - 建站笔记
tags:
  - Hexo
  - Redefine
  - 个人博客
excerpt: 记录一次从零搭建个人博客的过程，包括 Hexo 项目初始化、Redefine 主题接入、目录结构理解、本地预览和后续写作流程。
---

最近准备重新整理一个自己的个人博客。之前手上有一份已经生成好的静态博客目录，里面全是 `index.html`、`css`、`js` 和图片资源，可以直接部署访问，但不适合继续长期维护。真正适合写博客的方式，还是保留一份源码工程：文章用 Markdown 写，样式和站点信息通过配置管理，最后再由 Hexo 生成静态页面。

这篇文章记录一下我使用 Hexo 搭建个人博客的过程，也顺便把一些容易混淆的概念理清楚。

## 技术选型

这次使用的框架和主题如下：

```text
Hexo: 静态博客生成器
Redefine: Hexo 主题
Node.js: Hexo 运行环境
Markdown: 文章写作格式
```

Hexo 的核心思路很简单：

```text
Markdown 文章 + 站点配置 + 主题模板
              ↓
           Hexo 构建
              ↓
        public 静态网页
```

也就是说，平时写作和修改都应该发生在源码目录里，最终生成出来的 `public/` 只是部署产物，不建议直接手动修改。

## 初始化项目

先在博客目录下新建一个 Hexo 工程。我这里把项目命名为 `blogdu`：

```powershell
cd "E:\研究生材料\个人博客"

npm.cmd install -g hexo-cli
hexo.cmd init blogdu
cd ".\blogdu"
npm.cmd install
```

这里有一个 Windows 下的小细节：PowerShell 直接运行 `npm` 有时会被执行策略拦住，所以我习惯使用 `npm.cmd`、`hexo.cmd` 这类命令。

初始化完成后，项目结构大概是这样：

```text
blogdu/
├─ _config.yml
├─ package.json
├─ scaffolds/
├─ source/
│  └─ _posts/
├─ themes/
└─ node_modules/
```

其中最重要的是：

```text
_config.yml       站点主配置
source/_posts/   博客文章目录
scaffolds/       新建文章时使用的模板
public/          构建后的静态文件目录
```

## 安装 Redefine 主题

默认 Hexo 会使用 `landscape` 主题。为了让博客外观更完整，这里安装 Redefine：

```powershell
npm.cmd install hexo-theme-redefine
```

然后在 `_config.yml` 里把主题切换成：

```yaml
theme: redefine
```

Redefine 主题的配置我放在项目根目录的 `_config.redefine.yml` 中。这样后续修改主题时，不需要直接动 `node_modules` 里的主题源码。

这点很重要：

```text
不要直接修改 node_modules/hexo-theme-redefine/
```

因为以后执行 `npm install` 或升级主题时，`node_modules` 里的内容可能会被覆盖。主题个性化最好通过 `_config.redefine.yml`、`source/images/` 或自定义注入来完成。

## 修改站点配置

站点基础信息在 `_config.yml` 中配置，例如：

```yaml
title: blogdu
subtitle: '个人博客'
description: '记录技术、学习和生活的个人博客'
author: Jorson_Du
language: zh-CN
timezone: Asia/Shanghai
```

本地开发时可以先写：

```yaml
url: http://localhost:4000
```

以后如果部署到 GitHub Pages，再改成正式域名，例如：

```yaml
url: https://yourname.github.io
root: /
```

文章链接格式由 `permalink` 控制：

```yaml
permalink: :year/:month/:day/:title/
```

这会生成类似下面这样的文章地址：

```text
/2026/05/08/使用-Hexo-框架搭建个人博客的笔记/
```

如果想让链接更短，也可以改成：

```yaml
permalink: posts/:title/
```

## 理解 source 和 public

搭建 Hexo 博客时，最容易混淆的是 `source/` 和 `public/`。

`source/` 是源码目录，平时应该改这里：

```text
source/_posts/       文章 Markdown
source/images/       图片、头像、封面图
source/about/        关于页面
```

`public/` 是生成目录，只用于部署：

```text
public/index.html
public/css/
public/js/
public/images/
```

每次执行：

```powershell
hexo.cmd clean
hexo.cmd generate
```

Hexo 都会重新生成 `public/`。所以如果直接修改 `public/` 里的文件，下一次构建时很可能会丢失。

我的理解是：

```text
source 是工作区，public 是成品区。
```

## 新建文章

新建一篇文章使用：

```powershell
hexo.cmd new post "使用 Hexo 框架搭建个人博客的笔记"
```

Hexo 会在 `source/_posts/` 下生成一个 Markdown 文件。文章开头的这一段叫 front matter：

```yaml
---
title: 使用 Hexo 框架搭建个人博客的笔记
date: 2026-05-08 15:04:54
categories:
  - 建站笔记
tags:
  - Hexo
  - Redefine
  - 个人博客
---
```

front matter 用来描述文章的元数据，包括标题、时间、分类和标签。正文则直接写在 front matter 后面。

## 本地预览

写完文章后，可以启动本地服务：

```powershell
hexo.cmd server -p 4000
```

然后浏览器打开：

```text
http://127.0.0.1:4000/
```

如果页面没有更新，可以重新构建：

```powershell
hexo.cmd clean
hexo.cmd generate
hexo.cmd server -p 4000
```

这次搭建时还遇到一个小问题：如果 `source/_posts/` 里没有任何文章，Redefine 主题在当前配置下可能不会生成首页 `index.html`，访问首页时会看到 404。所以我保留了一篇起始文章，后续再逐步替换成正式内容。

## 常用命令

日常写博客基本只需要记住这几个命令：

```powershell
# 新建文章
hexo.cmd new post "文章标题"

# 本地预览
hexo.cmd server -p 4000

# 清理缓存和生成目录
hexo.cmd clean

# 生成静态文件
hexo.cmd generate

# 完整重新构建
hexo.cmd clean
hexo.cmd generate
```

如果想通过 npm scripts 执行，也可以用：

```powershell
npm.cmd run server
npm.cmd run build
npm.cmd run clean
```

## 后续计划

目前博客已经可以正常运行，下一步准备继续做几件事：

1. 修改头像、favicon 和首页 banner。
2. 调整导航栏，增加“归档”“分类”“标签”“关于我”。
3. 整理文章模板，让每篇新文章自动带上分类、标签和摘要字段。
4. 配置本地搜索和 RSS。
5. 最后再部署到 GitHub Pages 或自己的服务器。

搭建博客本身并不复杂，真正关键的是把工作流理清楚：文章写在 `source/_posts/`，站点信息写在 `_config.yml`，主题外观写在 `_config.redefine.yml`，生成结果在 `public/`。只要这几个边界清楚，后续维护起来就会轻松很多。
