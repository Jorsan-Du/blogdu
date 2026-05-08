---
title: Git 常用命令：从第一次上传到日常更新
date: 2026-05-08 17:10:00
categories:
  - 建站笔记
tags:
  - Git
  - GitHub
  - Hexo
  - Vercel
excerpt: 整理一套适合日常使用的 Git 命令流程，包括第一次上传项目、后续提交更新、拉取远程代码、查看状态，以及 Hexo 博客部署到 Vercel 时的常用操作。
---

平时写博客或者维护一个小项目时，Git 命令不需要记得特别多，但有几组命令一定要熟。它们分别对应几个常见场景：

```text
第一次把本地项目上传到 GitHub
后续修改后提交并推送
只提交某一个文件
从远程仓库拉取最新代码
查看当前仓库状态
配合 Hexo 和 Vercel 自动部署
```

这篇文章就把这些命令整理成一套可以反复查阅的流程。以后写完文章、改完配置、推送到 GitHub，再等 Vercel 自动部署，基本就靠这些命令完成。

## 第一次上传项目前的准备

如果是第一次在这台电脑上使用 Git，需要先配置用户名和邮箱。这个配置通常只需要做一次：

```powershell
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

这两个值会写入 Git 提交记录，用来标识这次提交是谁做的。可以用下面的命令检查当前配置：

```powershell
git config --global --list
```

如果输出里能看到类似下面的内容，就说明配置成功了：

```text
user.name=你的GitHub用户名
user.email=你的GitHub邮箱
```

## 第一次把项目上传到 GitHub

进入项目根目录后，先初始化本地 Git 仓库：

```powershell
git init
```

然后把默认分支改成 `main`：

```powershell
git branch -M main
```

接着把当前项目里的文件加入暂存区：

```powershell
git add .
```

提交一次初始版本：

```powershell
git commit -m "Initial commit"
```

然后把本地仓库和 GitHub 上的新仓库关联起来：

```powershell
git remote add origin https://github.com/你的用户名/你的仓库名.git
```

最后推送到 GitHub：

```powershell
git push -u origin main
```

这里的 `-u origin main` 表示把当前本地分支和远程的 `main` 分支建立默认关联。以后再推送时，就可以直接使用：

```powershell
git push
```

## 检查远程仓库地址

如果不确定项目有没有关联 GitHub 仓库，可以查看远程地址：

```powershell
git remote -v
```

正常情况下，会看到类似这样的结果：

```text
origin  https://github.com/你的用户名/你的仓库名.git (fetch)
origin  https://github.com/你的用户名/你的仓库名.git (push)
```

如果发现地址写错了，不需要重新初始化仓库，直接修改远程地址即可：

```powershell
git remote set-url origin https://github.com/你的用户名/正确的仓库名.git
```

修改后再检查一次：

```powershell
git remote -v
```

## 日常更新代码

日常最常用的是三步：

```powershell
git add .
git commit -m "这次修改的说明"
git push
```

例如写完一篇博客，可以这样提交：

```powershell
git add .
git commit -m "更新博客文章"
git push
```

这三步分别对应：

```text
git add .       把当前修改加入暂存区
git commit      生成一次本地提交
git push        把提交推送到 GitHub
```

如果项目已经接入 Vercel，那么 `git push` 之后，Vercel 通常会自动拉取最新代码并重新部署。

## 查看当前修改了什么

提交之前，建议先看一下当前仓库状态：

```powershell
git status
```

它会告诉你哪些文件被修改了、哪些文件还没有加入暂存区、当前分支是否领先或落后远程分支。

如果只想看文件级别的简略状态，可以用：

```powershell
git status --short
```

常见标记大概是：

```text
M  表示文件被修改
A  表示新增文件
D  表示删除文件
?? 表示未被 Git 跟踪的新文件
```

## 只提交某一个文件

有时候不想把所有改动都提交上去，只想提交某一篇文章或某一个配置文件，可以指定文件路径：

```powershell
git add source/_posts/你的文章.md
git commit -m "新增一篇文章"
git push
```

也可以只提交配置文件：

```powershell
git add _config.yml
git commit -m "调整站点配置"
git push
```

这种方式适合在工作区有多处修改，但只想先发布其中一部分内容的时候使用。

## 拉取远程最新代码

如果在另一台电脑上改过代码，或者 GitHub 上的仓库比本地更新，需要先拉取远程代码：

```powershell
git pull
```

如果想明确指定远程仓库和分支，也可以写成：

```powershell
git pull origin main
```

日常建议在开始修改之前先执行一次：

```powershell
git pull
```

这样可以减少本地代码和远程代码不一致导致的冲突。

## 查看提交记录

查看最近的提交记录：

```powershell
git log --oneline
```

输出会类似这样：

```text
27ebeef 调整 Hexo 配置适配 Vercel
72a3af7 更新博客内容
```

这对确认 Vercel 部署的是不是最新提交很有帮助。比如 Vercel 后台会显示本次部署对应的 commit id，可以和 `git log --oneline` 里的记录对照。

## 适合 Hexo 博客的日常流程

如果项目是 Hexo 博客，并且部署到 Vercel，我现在更推荐下面这套流程：

```powershell
hexo.cmd clean
hexo.cmd generate
git status
git add .
git commit -m "更新博客内容"
git push
```

这里 `hexo.cmd clean` 和 `hexo.cmd generate` 主要用于本地验证，确认文章和配置可以正常生成。真正部署到 Vercel 时，Vercel 会在云端执行构建命令。

对于这个博客项目来说，`package.json` 里有：

```json
{
  "scripts": {
    "build": "hexo generate"
  }
}
```

所以 Vercel 的构建过程大致是：

```text
GitHub 收到 push
  -> Vercel 拉取最新代码
  -> 执行 npm run build
  -> 生成 public/
  -> 发布 public/ 目录
```

这意味着如果使用 Vercel，一般不需要执行：

```powershell
hexo.cmd deploy
```

`hexo deploy` 更适合 GitHub Pages 这类由 Hexo 主动推送静态产物的部署方式；Vercel 则是从 GitHub 拉源码后自动构建。

## 哪些文件不应该提交

Hexo 项目里有些目录通常不应该提交到 GitHub，比如：

```text
node_modules/
public/
db.json
.deploy*/
```

原因是：

```text
node_modules/ 是依赖安装目录，可以通过 npm install 重新生成。
public/ 是 Hexo 构建产物，可以通过 hexo generate 重新生成。
db.json 是 Hexo 本地缓存，不是源码。
.deploy*/ 是部署过程中生成的临时目录。
```

这些内容应该写进 `.gitignore`。这样 GitHub 仓库里保存的是源码，而不是一堆可以自动生成的文件。

## 常用检查命令速查

查看当前状态：

```powershell
git status
```

查看简略状态：

```powershell
git status --short
```

查看当前分支：

```powershell
git branch
```

查看远程仓库：

```powershell
git remote -v
```

查看最近提交：

```powershell
git log --oneline
```

查看某个文件的修改：

```powershell
git diff _config.yml
```

查看已经加入暂存区的修改：

```powershell
git diff --cached
```

## 一个完整示例

假设我今天写完一篇博客，准备发布到 Vercel，完整流程可以是：

```powershell
hexo.cmd clean
hexo.cmd generate
git status
git add .
git commit -m "新增 Git 常用命令笔记"
git push
```

推送完成后，去 Vercel 后台看最新部署。如果部署状态是 `Ready`，并且对应的提交信息是刚刚这次 commit，就说明 Vercel 已经拿到了最新代码。

如果网页内容更新了，但样式丢失，就优先检查 Hexo 的 `_config.yml`：

```yaml
url: https://你的域名
root: /
```

如果部署在 Vercel 根域名下，`root` 通常应该是 `/`。如果仍然写成 GitHub Pages 项目站点常用的 `/仓库名/`，页面就可能能打开，但 CSS、JS 和图片路径全部失效。

## 结尾

Git 的日常使用并不需要一开始就背很多命令。真正高频的，其实就是：

```powershell
git status
git add .
git commit -m "说明"
git push
```

再配合 Hexo 的：

```powershell
hexo.cmd clean
hexo.cmd generate
```

基本就能覆盖个人博客的大部分维护场景。等这套流程熟了以后，再去理解分支、回滚、合并冲突这些内容，会自然很多。
