---
title: 使用 CDN 加速你的 GitHub Pages 网站
date: 2020-02-05 20:28:57
updated: 2020-02-16 20:28:57
tags:
  - CDN
  - 笔记
  - GitHub Pages
categories:
  - 云游的小笔记
---

内容分发网络 Content Delivery Network

<!-- more -->

## 前言

除去 WordPress, 许多博客网站是托管于 GitHub Pages 上的，但其在国内的速度实在是难以称道。
所以使用国内服务商的 CDN 对其进行加速不失为一个好办法。

> 内容分发网络（Content Delivery Network，CDN）通过将站点内容发布至遍布全国的海量加速节点，使其用户可就近获取所需内容，避免网络拥堵、地域、运营商等因素带来的访问延迟问题，有效提升下载速度、降低响应时间，提供流畅的用户体验。

曾将我使用的策略是在国内托管 [Coding Pages](https://coding.net/)，以及为了让百度能抓取到博客内容，还做了一番配置。

> [让百度收录你的 GitHub Pages 博客](https://yunyoujun.cn/note/baidu-seo-about-github-pages/)

但是 Coding 的服务并不稳定，且经常变动一些策略。
现在更是下线整合到静态部署中，且似乎因为 API 的问题实名认证总是不能通过，所以暂时都无法使用。

所以便干脆使用 CDN 来进行加速。并且也可以轻松被百度抓取了。

因为我域名购置于腾讯云，且腾讯云 CDN 每月赠送免费 10G 流量。
所以我直接采用腾讯云的 CDN 来实现。

> 注意：域名需要备案（按需签发 SSL 证书）

## 步骤

首先开通[腾讯云 - 内容分发网络](<(https://cloud.tencent.com/product/cdn)>)。

### 添加自己的域名

![add-domain-for-cdn](https://cos.yunyoujun.cn/blog/add-domain-for-cdn.png)

### 设置源站

管理 > 基本配置

![config-source-site](https://cos.yunyoujun.cn/blog/config-source-site.png)

这里是 GitHub Pages 提供的 IP 地址，可以添加多行。

> https://help.github.com/en/github/working-with-github-pages/about-custom-domains-and-github-pages

---

> 可选：建议前往 `高级配置` 开启 HTTPS 配置

### 回源协议

证书管理 > 编辑 > 协议跟随 (如果没开启 HTTPS，默认的 HTTP 也可以。)

![set-back-source-protocol](https://cos.yunyoujun.cn/blog/set-back-source-protocol.png)

### 设置 CNAME

前往 [域名解析](https://console.cloud.tencent.com/cns)

根据需要将 CDN 提供的 CNAME 线路类型设置为 `境内`。
`境外` 则仍默认解析回 GitHub Pages。

![set-different-cname-for-domain](https://cos.yunyoujun.cn/blog/set-different-cname-for-domain.png)

### 配置缓存

默认的缓存时间非常长，不配置的话就会导致 CDN 的文件长时间没有更新。

可以参见腾讯云文档 [缓存配置问题](https://cloud.tencent.com/document/product/228/2968#.E7.BC.93.E5.AD.98.E9.85.8D.E7.BD.AE.E9.97.AE.E9.A2.98)

也可以在 [刷新预热](https://console.cloud.tencent.com/cdn/refresh) 处手动刷新。

## 后话

测试发现首页基本可以秒开，速度确实不错。
至于流量万一不够用怎么办，emm，大概等这里真有这么大访问量的时候，就不至于还要在这样各处薅羊毛了吧。

## FAQ

### CNAME 与 MX 记录冲突导致邮件丢失

值得注意的是，设置 CDN 的方式是使用 CNAME 重定向到 CDN 域名。
如果你同时将裸域名（yunyoujun.cn）作为博客域名和域名邮箱（比如我的邮箱：me@yunyoujun.cn），那么你可能会遇到 CNAME 与 MX 记录冲突问题。

如果你的运营商没有这么提示你，那也最好不要这么做，因为这会导致域名邮箱发生邮件丢失。

在过去解析尚未规范时，部分运营商是允许同时在裸域名上设置 CNAME 和 MX 记录的。
但如今按照 RFC 标准协议，CNAME 优先级最高，所以在解析请求过程中，会优先返回 CNAME 解析记录结果。
这样设置的结果就是导致无法解析到用户设置的 MX 记录（设置权重也没有用），影响邮箱的正常收发。

现在，大部分运营商会提示 CNAME 与 MX 记录发生冲突，来避免这种情况。

> 更多信息请参阅 [RFC1034](https://www.rfc-editor.org/rfc/rfc1034.txt?spm=a2c4g.11186623.2.13.59ef4054LkHX23&file=rfc1034.txt) 和 [RFC2181](https://www.rfc-editor.org/rfc/rfc2181.txt?spm=a2c4g.11186623.2.14.59ef4054LkHX23&file=rfc2181.txt)。
> [记录冲突的规则](https://help.aliyun.com/knowledge_detail/39787.html#h2-u8BB0u5F55u51B2u7A81u7684u89C4u52193)

我此前之所以使用 GitHub Pages 托管，却仍然能够使用域名邮箱，是因为我使用了 GitHub 提供的 A 记录解析。

```txt
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

> [Managing a custom domain for your GitHub Pages site](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)

而如今加了 CDN 又回到了这一两难的局面。

最后想着长痛不如短痛，下定决定将博客主域名更换为 `www.yunyoujun.cn`。

裸域名仍旧使用 A 记录和 MX 记录。
设置 A 记录的作用是用户访问 `yunyoujun.cn` 时（GitHub Pages 的 CNAME 文件提前设置为 `www.yunyoujun.cn`），那么 GitHub Pages 会自动从 `yunyoujun.cn` 跳转为 `www.yunyoujun.cn`。

此外，谷歌浏览器会自动隐藏 www 域名前缀，所以一定程度上减少观感的影响。

以及我发现一些企业的网站都采取的裸域名跳转 www 域名的方式。

譬如：

- 语雀：<yuque.com>,
- JetBrains（著名的 IDE 软件开发商）：<jetbrains.com>（我在几年前的视频里发现他们留的还是裸域名的网址，而现在则是跳转 www 链接。）

当然如果你对域名邮箱没有需求，且域名非常短又很酷，使用裸域名也并非不可。

> PS. 怎么感觉最近说话都有点翻译腔了。