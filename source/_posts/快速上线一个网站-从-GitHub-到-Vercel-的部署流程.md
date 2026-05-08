---
title: 快速上线一个网站：从 GitHub 到 Vercel 的部署流程
date: 2026-05-08 16:45:00
categories:
  - 建站笔记
tags:
  - Vercel
  - GitHub
  - Cloudflare
  - DNS
excerpt: 记录一次把网站快速上线的完整流程，包括代码托管、Vercel 部署、自定义域名接入、Cloudflare DNS 配置以及 SSL 证书的基本原理。
---

最近在整理个人博客和一些小网站的上线流程，发现真正影响效率的往往不是写代码本身，而是“怎么把它稳定地挂到网上”。如果只是本地跑起来，事情很快就结束了；但一旦要接入自己的域名、启用 HTTPS、顺便让访问更稳定，部署链路就会一下子变长。

这篇文章参考一份“快速上线一个网站”的流程清单，顺手把背后的思路也一起整理一下。整体路线可以概括为：

```text
本地代码
  -> GitHub 托管
  -> Vercel 自动部署
  -> 自定义域名
  -> Cloudflare DNS 与代理
  -> SSL/TLS 配置
```

如果你正在做个人博客、作品集、文档站或者一个轻量级产品官网，这套流程基本都适用。

## 事先要准备什么

开始之前，至少需要准备这几样东西：

```text
1. GitHub 账号
2. 可以正常运行的网站代码
3. Vercel 账号
4. 一个域名（如果你想使用自定义域名）
5. Cloudflare 账号（如果你想把 DNS 和代理交给 Cloudflare）
```

如果只是先把网站发出去，其实前三项就够了。域名和 Cloudflare 是第二阶段的事情，不影响你先把站点上线。

## 第一步：把代码放到 GitHub

无论你的网站是用 Hexo、Next.js、Vue，还是一套纯静态 HTML 页面，第一步都建议先把源码放到 GitHub 仓库里。

这样做有几个直接的好处：

```text
1. 代码有版本记录，修改不会乱。
2. Vercel 可以直接从仓库拉代码并自动部署。
3. 以后每次 push，都可以触发新一轮构建。
```

如果是 Hexo 这种静态站点，我更推荐把“源码工程”放到 GitHub，而不是只上传生成后的 `public/`。因为真正需要长期维护的是：

```text
Markdown 文章
站点配置
主题配置
图片与静态资源
```

这些内容才是网站的工作区。生成后的网页只是构建产物，交给 Vercel 去处理更合适。

## 第二步：在 Vercel 上导入仓库

代码进了 GitHub 之后，就可以登录 Vercel，把 GitHub 账号关联过去，然后选择刚才的仓库进行部署。

Vercel 的优点在于，它把“拉代码、构建、发布、生成预览链接”这一串动作都自动化了。对于个人站点来说，体验非常省心：

```text
GitHub push 一次
    -> Vercel 自动检测提交
    -> 自动构建
    -> 自动发布
```

如果项目是 Hexo，通常只需要确认两件事：

```text
Build Command: npm run build
Output Directory: public
```

以我这个博客项目为例，`package.json` 里已经有：

```json
{
  "scripts": {
    "build": "hexo generate"
  }
}
```

所以 Vercel 在构建时执行 `npm run build`，本质上就是执行 `hexo generate`，最后把 `public/` 作为发布目录。

如果只是想尽快把网站发出去，到这里其实已经完成了。此时你会拿到一个类似下面这样的默认访问地址：

```text
https://your-project-name.vercel.app
```

## 第三步：接入自定义域名

默认的 `vercel.app` 域名适合预览和测试，但如果准备长期使用，还是建议绑定自己的域名。

一个比较常见的流程是：

```text
域名注册商购买域名
    -> 把 DNS 托管交给 Cloudflare
    -> 在 Vercel 中绑定域名
    -> 按提示补齐 DNS 记录
```

域名注册商不一定非得是哪一家，只要后台支持修改 DNS 服务器即可。参考材料里提到的是 `Spaceship`，它的优点是对国内用户也比较友好，支持支付宝。

如果只是从操作角度理解，这一步做的事情其实很简单：

```text
买一个域名
告诉这个域名以后由 Cloudflare 负责解析
再告诉 Vercel：这个域名是我的，请把它指到我的站点
```

## 第四步：为什么要把 DNS 交给 Cloudflare

很多人第一次接触 Cloudflare，会觉得它只是“顺手做个解析”。其实它扮演的角色比普通 DNS 面板更大一些。

当你把域名的名称服务器改成 Cloudflare 提供的 NS 之后，含义是：

```text
以后这个域名的解析规则，不再由原注册商决定
而是由 Cloudflare 作为权威 DNS 提供商来决定
```

从网络原理上说，DNS 的作用就是把域名转换成可访问的服务器地址。用户在浏览器里输入：

```text
example.com
```

浏览器并不知道网站在哪台机器上，它会先查 DNS，拿到对应的记录之后，才知道请求该发到哪里。

把解析权交给 Cloudflare 之后，后续你就可以在 Cloudflare 后台控制：

```text
A 记录
CNAME 记录
代理开关
缓存策略
HTTPS 行为
```

这也是为什么很多人会把 Cloudflare 放在 Vercel 前面。整体链路可以理解成：

```text
用户
  -> Cloudflare
  -> Vercel
```

## 第五步：A 记录、CNAME 记录到底在干什么

当你在 Vercel 里绑定域名时，它通常会提示你配置若干条 DNS 记录。最常见的是：

```text
A 记录：根域名指向一个 IP
CNAME 记录：www 等子域名指向一个别名
```

例如：

```text
A      @      76.76.21.21
CNAME  www    cname.vercel-dns.com
```

它们各自的含义可以这样理解：

```text
A 记录：
把一个域名直接指向某个 IP 地址。

CNAME 记录：
把一个域名别名指向另一个域名，让后者继续完成解析。
```

所以在接入 Vercel 时，常见做法是：

```text
根域名 example.com 用 A 记录
www.example.com 用 CNAME 记录
```

如果此时你在 Cloudflare 里打开代理模式，请求就不会直接从用户飞到 Vercel，而是先经过 Cloudflare 的边缘节点，再由 Cloudflare 转发给 Vercel。

这一步带来的价值包括：

```text
1. 隐藏源站信息
2. 提供基础的安全防护
3. 借助边缘节点改善访问体验
4. 为后续 HTTPS 和缓存策略留出控制空间
```

## 第六步：SSL/TLS 为什么经常出问题

网站一旦挂上自定义域名，下一件事往往就是 HTTPS。很多部署流程真正卡住的地方，也恰好在这里。

常见现象包括：

```text
打开后反复重定向
浏览器提示证书错误
ERR_TOO_MANY_REDIRECTS
```

背后原因通常不是“证书没申请到”，而是链路上有两段 HTTPS 配置没有对齐：

```text
用户 <-> Cloudflare
Cloudflare <-> Vercel
```

比较稳妥的方式，是在 Cloudflare 中使用：

```text
SSL/TLS Mode: Full (strict)
```

它的含义是：

```text
用户访问 Cloudflare 时，使用 Cloudflare 的证书加密；
Cloudflare 去访问 Vercel 时，也要求对方提供可信证书。
```

这样做的好处有两个：

```text
1. 整条链路都保持加密。
2. 避免因为一端 HTTP、一端 HTTPS 而出现重定向循环。
```

如果把这件事说得再直白一点，可以理解为：

```text
不是“网站有证书”就够了，
而是链路上的每一段都要知道自己该用什么方式通信。
```

## 一套更容易记住的上线顺序

如果不想每次部署都把自己绕晕，我现在更习惯按下面的顺序操作：

1. 先保证本地代码能正常运行。
2. 把源码推到 GitHub。
3. 在 Vercel 上导入仓库，先拿到一个可访问的默认域名。
4. 网站内容确认无误后，再绑定自定义域名。
5. 用 Cloudflare 接管 DNS。
6. 补齐 A 记录和 CNAME 记录。
7. 最后检查 SSL/TLS 是否处于 `Full (strict)`。

这个顺序的好处在于：每一步都只解决一个问题。

```text
先解决“能不能发布”
再解决“能不能用自己的域名”
最后解决“稳不稳定、安不安全”
```

这样比一上来同时折腾代码、域名、DNS、证书，要轻松得多。

## 如果网站本身是 Hexo，需要特别注意什么

对 Hexo 站点来说，还有一个很容易被忽略的小点：部署目标会影响 `_config.yml` 里的 `url` 和 `root`。

如果部署到 Vercel 的根域名，通常应该写成：

```yaml
url: https://your-domain.com
root: /
```

如果部署到 GitHub Pages 的项目子路径，例如：

```text
https://username.github.io/blogdu
```

则通常要写成：

```yaml
url: https://username.github.io/blogdu
root: /blogdu/
```

这两个值的意义并不只是“看起来像网址”：

```text
url 影响 canonical、og:url、RSS 等绝对链接
root 影响静态资源路径和站内链接前缀
```

所以如果后续决定从 GitHub Pages 切到 Vercel，最好把这两个值一起调整，避免页面资源路径仍然带着旧的子目录。

## 结尾

回头看这条上线链路，其实每一环都不算复杂，复杂的是它们刚好横跨了代码托管、构建部署、DNS、代理和 HTTPS 五个不同层面。把这些东西一次性堆在一起看，很容易让人误以为“部署网站特别难”。

但如果换个视角：

```text
GitHub 管源码
Vercel 管构建和发布
Cloudflare 管域名解析和代理
SSL/TLS 管通信加密
```

每一层的职责就清楚很多了。

对个人博客或轻量站点来说，这已经是一条相当成熟的路线。先让网站上线，再逐步补齐域名、证书和访问体验，通常会比一开始追求“全都配满”更高效。
