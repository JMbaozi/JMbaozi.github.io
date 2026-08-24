---
layout: post
title: '目录点不动了：一次 TOC 锚点失灵的排查与修复'
subtitle: 'kramdown 锚点 id 生成规则的填坑笔记'
date: 2026-08-24
categories: blog
author: yucol
tags: [Jekyll, kramdown, Liquid, 填坑, blog]
cover: 'https://img.yucol.uk/2026/08/a377711300cffa4926bb759155881e60.webp'
cover_author: 'Fotis Fotopoulos'
cover_author_link: 'https://unsplash.com/@ffstop'
---

## 前言

博客迁移到 H2O-ac 主题之后，一直有个小毛病懒得管：文章右侧的目录（TOC），有的条目点了能跳，有的点了毫无反应。起初以为是网络卡顿，后来发现规律相当稳定——**标题是纯文字的能跳转，标题里带符号的就不能**。比如《OAuth2（含密码哈希）与带 JWT 令牌的 Bearer 认证》这篇里，「核心安全工具函数」点得动，「数据模型定义 (Pydantic)」就点不动。

今天下决心把这个 bug 修掉，顺带把搁置已久的本地预览环境搭了起来。以后改主题不再「盲改—推送—在线抽奖」，而是本地看过了再 push。

## 定位：点不动的时候，浏览器在报错

打开开发者工具的 Console，点击那个点不动的目录项，报错立刻现形：

```text
Uncaught TypeError: Cannot read properties of null (reading 'getBoundingClientRect')
```

`getBoundingClientRect` 出现在主题的平滑滚动函数里。看一眼 `_layouts/default.html`：

```js
function scrollToAdjust(id){
  var element = document.getElementById(id);
  var headerOffset = 90;
  var elementPosition = element.getBoundingClientRect().top;
  ...
}
```

也就是说，目录点击后是靠 `document.getElementById(id)` 找到标题元素再平滑滚动过去的（偏移 90px 是为了不被吸顶的导航栏挡住）。`element` 是 `null`，说明**传进来的 id 在页面上根本不存在**。

再看目录这边的 HTML，点击的并不是一个普通的锚点链接：

```html
<a onclick="scrollToAdjust('数据模型定义-(pydantic)')">数据模型定义 (Pydantic)</a>
```

而文章里这个标题真实渲染出来的标签是：

```html
<h2 id="数据模型定义-pydantic">数据模型定义 (Pydantic)</h2>
```

一边是 `数据模型定义-(pydantic)`，一边是 `数据模型定义-pydantic`。括号没被处理掉，`getElementById` 自然找不到人。问题清楚了，但为什么会差这么一点？

## 根因：两套各说各话的 id 生成规则

H2O-ac 的目录基于 [allejo/jekyll-toc](https://github.com/allejo/jekyll-toc) 这段 Liquid 实现。原版逻辑其实很干净：从编译后的 HTML 里把标题标签上的 `id="..."` 原样抠出来，拼成 `href="#id"`。**id 是 kramdown 生成什么就用什么，天然不会错位**。

主题作者为了让目录点击变成「带偏移的平滑滚动」，改掉了链接的生成方式——不再用现成的 id，而是拿标题文字自己重新推导一遍。`_includes/layouts/toc.html` 里关键的两行（修改前）：

{% raw %}
```liquid
{% assign anchorBody2 = anchorBody | remove: "：" | replace: " ", "-" | downcase %}
{% capture listItem %}<a onclick="scrollToAdjust('{{ anchorBody2 }}')">{{ anchorBody }}</a>{% endcapture %}
```
{% endraw %}

这套推导只做了三件事：删掉全角冒号、空格换成 `-`、转小写。

而 kramdown 开启 `auto_ids`（默认开启）后，给标题生成 id 走的是另一套规则。拿本站文章实测归纳一下：**ASCII 字母转小写、空格转 `-`、汉字保留、其余符号（`（）、：？！` 等）一律删除**。

两套规则一对照，问题就完全对上了：

| 标题原文 | kramdown 生成的真实 id | Liquid 推导出的字符串 | 结果 |
| --- | --- | --- | --- |
| FastAPI 教程示例 | fastapi-教程示例 | fastapi-教程示例 | 能跳 |
| 总结：核心知识点图谱 | 总结核心知识点图谱 | 总结核心知识点图谱 | 能跳（全角冒号恰好被照顾到） |
| 数据模型定义 (Pydantic) | 数据模型定义-pydantic | 数据模型定义-(pydantic) | 找不到元素 |
| API 路由 (Endpoints) | api-路由-endpoints | api-路由-(endpoints) | 找不到元素 |

也就是说，主题作者显然是踩过「总结：xxx」这类标题的坑，于是专门补了 `remove: "："`——但这只是打补丁，没有对齐 kramdown 的完整规则。任何带括号、顿号、问号的标题依然会失配。与其继续枚举符号打补丁，不如釜底抽薪。

## 修复：别自己算，用现成的

回到 jekyll-toc 的原始思路：id 不要推导，直接用标题标签上那个。这段 Liquid 本来就把真实 id 提取成了 `htmlID` 变量（原版 `href` 用的就是它），改回去即可——同时保留主题的平滑滚动体验：

{% raw %}
```liquid
{% capture listItem %}<a{{ anchorAttributes }} onclick="scrollToAdjust('{{ htmlID }}'); return false;">{{ anchorBody }}</a>{% endcapture %}
```
{% endraw %}

改动只有一行（删掉原来那行 `anchorBody2` 的推导）。说明两点：

- `anchorAttributes` 是这段 Liquid 里现成的变量，包含 `href="#真实id"`，顺带恢复了链接语义——中键新开、JS 失效时的浏览器原生跳转都回来了；
- `return false` 阻止 `href` 的默认瞬时跳转，避免和 `scrollToAdjust` 的平滑滚动打起来。

另外给 `scrollToAdjust` 补了个判空，以后即使再出现失配，也只是安静地不滚动，而不是在 Console 里抛异常：

```js
function scrollToAdjust(id){
  var element = document.getElementById(id);
  if (!element) return;
  var headerOffset = 90;
  ...
}
```


## 顺带：把本地预览环境搭了

修 bug 的过程中还了另一桩心愿：在这台 Windows 机器上跑起 `jekyll serve`。这里也记几笔，都是实际踩到的坑。

**装 Ruby。** 去 [RubyInstaller](https://rubyinstaller.org/) 装 3.4 的 base 版即可，Jekyll 本体不需要 DevKit。

**native 扩展会要 MSYS2。** `bundle install` 装到 `eventmachine`、`racc` 这类带 C 扩展的 gem 时会报 `MSYS2 could not be found`。Ruby 对 MSYS2 的探测顺序里有一条：**优先找 Ruby 安装目录下的 `msys64`**。所以把 MSYS2 静默安装到 `C:\Ruby34-x64\msys64`，什么都不用配置，RubyGems 自己就能发现它，连 `ridk install` 都省了。

**gem 源的证书问题。** Gemfile 里写的 `https://gems.ruby-china.com/` 在我的网络下握手直接报 `hostname mismatch`，但官方源反而畅通。不动 Gemfile（CI 还要靠它），用 bundler 的本地配置把源重定向：

```bash
bundle config set --local mirror.https://gems.ruby-china.com/ https://rubygems.org/
bundle config set --local path vendor/bundle
```

这两条落在 `.bundle/config` 里，只对本机生效。同时记得把 `.bundle` 和 `vendor` 加进 `.gitignore`，免得哪天手滑提交了几百兆依赖。

之后就是常规操作：

```bash
bundle install
bundle exec jekyll serve
# http://127.0.0.1:4000
```

改 Markdown、改模板，保存即热更新，浏览器里当场看效果。

## 结语

这次修复的本质其实是个很朴素的道理：**当系统已经生成了一份权威数据（kramdown 的锚点 id）时，任何「按自己理解再算一遍」的代码都是在制造第二个真相源**。两套规则只要有一条对不上，bug 就来了。删掉那行自作聪明的 Liquid，让目录老老实实用标题自带的 id，问题就再也没有复现的机会。

新的工作流也顺了：本地 `jekyll serve` 预览 → 浏览器验证 → 满意了再 push 触发 Actions 构建。