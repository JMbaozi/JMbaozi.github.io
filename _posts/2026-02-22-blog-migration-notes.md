---
layout: post
title: '从 H2O 到 H2O-ac：我的博客迁移与填坑笔记'
subtitle: '一次略微不顺的博客主题更换历程'
date: 2026-02-22
categories: blog
author: yucol
tags: [Jekyll, GitHub Actions, 填坑, blog]
cover: 'https://images.unsplash.com/photo-1506104489822-562ca25152fe?w=1600&q=900'
cover_author: 'Redd Francisco'
cover_author_link: 'https://unsplash.com/@reddfrancisco'

pin: true
---

## 前言

博客最初的采用的模板是由[kaeyleo](https://github.com/kaeyleo)基于[jekyll](https://jekyllrb.com/)开发的[H2O](https://github.com/kaeyleo/jekyll-theme-H2O)主题。我在此基础上添加了评论区、友链、归档、音乐播放器、访客记录、目录等功能。但是限于能力有限，写的代码一堆BUG。最近我浏览到了一个基于[H2O](https://github.com/kaeyleo/jekyll-theme-H2O)修改的[H2O-ac](https://github.com/zhonger/jekyll-theme-H2O-ac)主题，其内容丰富，视觉效果和阅读体验非常好。

所以我决定对博客进行了一次“大手术”，将原本一直使用的H2O模板替换成了目前视觉效果非常出色的 **H2O-ac** 主题。

虽然 H2O-ac 带来了极佳的阅读体验和学术风界面，但在迁移和配置自动化的过程中，我遇到了不少令人头大的 Bug。

为了方便日后查阅，也为了给同样使用该主题的小伙伴提供参考，特写下这篇笔记。

## 迁移过程：从手动到全自动

最初的迁移并不顺利，旧模板留下的残留文件（如 Azure 的部署脚本等）频繁导致构建失败。

### 1. 自动化 CI/CD 的重建
为了实现“推送即发布”，我弃用了原本混乱的部署脚本，重新编写了 `.github/workflows/jekyll.yml`。通过以下逻辑确保了环境的纯净：
* **环境初始化**：使用 Ruby 3.4.4 环境。
* **强制清理**：在编译前运行 `bundle exec jekyll clean`。
* **强制覆盖**：发布到 `gh-pages` 分支时使用 `force_orphan: true`，彻底杜绝旧文件缓存。

### 2. 版权与开源协议
由于 H2O-ac 是基于 H2O 主题的 fork 版本，遵循 **MIT 开源协议**。在迁移过程中，我保留了原作者的版权信息。

## 遇到的问题与 Bug 排查（重点）

这次迁移最核心的挑战在于：文章在标签（Tags）页能看，但在首页（Blog）和归档（Archives）页却神秘失踪。经过反复排查，我定位到了以下几个“元凶”：

### Bug 1：`pin: false` 逻辑陷阱
这是最难发现的一个点。H2O-ac 主题自带了置顶功能，但我发现：
* 当我在文章头部显式写下 `pin: false` 时，首页和归档页会完全无法显示该文章。
* **解决方案**：普通文章直接**删除 `pin` 属性**即可。如果不置顶，就不要写这一行，写成 `false` 反而会被主题逻辑误判并过滤掉。

### Bug 2：分支挂载错位
在 GitHub Pages 设置中，如果将部署源选为 `master`，可能无法看到经过 Actions 机器人编译后的成品。
* **解决方法**：将 GitHub Pages 的部署分支切换到 **`gh-pages`**，这才是机器人存放 HTML 成品的地方。

## 参考文档

* [H2O-ac主题仓库](https://github.com/zhonger/jekyll-theme-H2O-ac)
* [H2O-ac主题介绍](https://lisz.me/tech/new-theme-h2o-ac)
* [H2O-ac主题文档](https://h2o-ac-doc.lisz.me/)

## 结语

感谢 H2O 系列主题的作者们提供的优秀代码。
