---
layout: post
title: '从零到一：阿里云 200M 带宽服务器的高级全栈基建（1Panel + Docker + GitHub Actions）'
subtitle: '服务器搭建'
date: 2026-02-25
categories: server
author: yucol
tags: [服务器, GitHub Actions, blog]
cover: 'https://pub-e58ead495ed24701b28423dc43c97973.r2.dev/2026/02/badd3f192479846afeaa60f358e947eb.webp'
cover_author: 'Yucol'
cover_author_link: 'https://yucol.top'
---


# 1. 前言：为什么放弃“手动搬运”？

在拥有了自己的服务器后，第一反应是手动上传文件。但这存在三个弊端：

* **效率低**：改一个字也要开一遍 FTP。
* **易出错**：容易漏传或传错目录。
* **无版本控制**：无法回溯历史。
本篇记录如何利用 **GitHub Actions** 打造一条自动化流水线，实现“本地代码一推，服务器即刻更新”。

---

# 2. 环境说明

* **服务器**：阿里云 2核2G / 200M 带宽 / Ubuntu 系统。
* **运维面板**：1Panel（基于 Docker 的现代化管理面板）。
* **静态引擎**：Jekyll (Ruby 3.4.4)。
* **部署通道**：SSH + Rsync。

---

# 3. 核心步骤详解

## 第一步：服务器基建 (1Panel)

1. 安装 1Panel 并在应用商店安装 **OpenResty**（Nginx 的增强版）。
2. 在“网站”菜单下创建一个静态网站。
* **路径**：`/opt/1panel/apps/openresty/openresty/www/sites/test/index`（此路径为后期同步的目标 `TARGET`）。


3. 确保服务器防火墙已开启 80/443 端口，以及 SSH 通讯所需的 22 端口。

## 第二步：建立“信任关系” (SSH Key)

为了让 GitHub 机器人能直接登录我们的服务器，需要配置免密登录：

1. **本地生成密钥对**：
`ssh-keygen -t rsa -b 4096 -f id_rsa_github`
2. **分发公钥**：将 `id_rsa_github.pub` 的内容追加到服务器的 `/root/.ssh/authorized_keys` 中。
3. **权限加固**：确保 `.ssh` 目录权限为 `700`，`authorized_keys` 权限为 `600`。

## 第三步：配置 GitHub Secrets

在 GitHub 仓库的 `Settings -> Secrets and variables -> Actions` 中配置四个关键变量：

* `REMOTE_HOST`: 服务器公网 IP。
* `REMOTE_USER`: `root`。
* `SSH_PRIVATE_KEY`: `id_rsa_github` 文件里的完整私钥内容。
* `REMOTE_TARGET`: 服务器上的静态目录路径。

## 第四步：编写自动化流水线 (YAML)

在 `.github/workflows/` 下创建 `jekyll.yml`。核心逻辑是利用 `easingthemes/ssh-deploy` 插件执行 `rsync`。

```yaml
# 关键配置摘要
- name: Deploy to Aliyun Server
  uses: easingthemes/ssh-deploy@v5.1.0
  with:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
    ARGS: "-rltzvi --delete" # 增量同步，删除服务器多余文件
    SOURCE: "_site/"         # Jekyll 构建后的产物目录
    REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
    REMOTE_USER: ${{ secrets.REMOTE_USER }}
    TARGET: ${{ secrets.REMOTE_TARGET }}

```

---

# 4. 外部资源集成：

为了追求极致的性能和稳定性，我并没有将所有功能都死磕在自己的服务器上，而是引入了更成熟的云原生方案：

## A. 评论系统：Waline + Vercel + Neon

为了不让评论数据的存储和处理占用阿里云的 2G 内存，我采用了 **Serverless（无服务器）** 架构：

* **后端部署 (Vercel)**：利用 Vercel 的免费额度托管 Waline 后端程序。
* **数据库 (Neon)**：所有的评论内容存储在 Neon 的云数据库中，安全且免费。
* **集成方式**：在 Jekyll 模板中引入 Waline 的 JS 脚本，通过 API 与 Vercel 通信。
* **深度心得**：这种方案实现了“数据随人走”，即便我以后重装服务器系统，评论数据也不会丢失。

## B. 极致图床：Cloudflare R2 + PicGo

虽然服务器有 200M 带宽，但为了节省宝贵的公网流量以及应对未来可能的并发访问，我搭建了专业的对象存储图床：

* **存储底座 (Cloudflare R2)**：利用 Cloudflare 的 R2 存储（S3 兼容模式），免流量费且自带全球 CDN 加速。
* **自动化工具 (PicGo)**：在本地配置 PicGo 插件，实现“截图 -> 上传 -> 自动生成 Markdown 链接”的一键流。
* **安全加固 (V2 签名)**：使用 V2 签名机制确保上传接口的安全，防止图床被恶意盗刷。

---

# 5. 未来扩展：

目前的基建仅完成了博客的部署。后续计划：

- [ ] **后端 API**：在 1Panel 中使用 Docker 部署 Python (FastAPI) 容器。
- [ ] **移动端联调**：Kotlin 开发的安卓 App 通过 HTTP 请求调用服务器 IP 上的 Python 接口，实现数据上云。
