---
title: 在 Debian 13 国内环境下安装 Hermes Agent 全攻略
date: 2026-05-12 18:53:04
updated: 2026-08-20 13:13:51
description: 在 Debian 13 国内网络环境中安装、初始化并优化 Hermes Agent 的完整步骤。
categories:
  - Linux 与运维
tags:
  - Hermes
  - Debian
  - AI
  - 教程
cover: /img/covers/install-hermes-agent-on-debian13-china.png
top_img: /img/covers/install-hermes-agent-on-debian13-china.png
comments: false
index: true
source: https://blog.xiaopangza.cn/archives/install-hermes-agent-on-debian13-china
---

> 本文同步自 [Halo 主博客](https://blog.xiaopangza.cn/archives/install-hermes-agent-on-debian13-china)，内容以两站最新版本为准。
Hermes Agent 是由 Nous Research 开发的开源 AI 助手框架。它不仅能运行在终端，还能无缝接入微信、Telegram、Discord 等社交平台。本文将介绍如何在最新的 **Debian 13 (Trixie)** 系统上，针对国内网络环境优化安装 Hermes Agent。

## 环境准备

首先，确保你的 Debian 13 系统已更新，并安装了基础工具：

```bash
sudo apt update && sudo apt install -y curl git python3 python3-venv nodejs npm
```

## 安装步骤

### 1. 使用官方脚本一键安装

Hermes 提供了一个便捷的安装脚本。在境内环境，我们可以通过代理或直接尝试：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

> 如果遇到 GitHub 连接超时，建议先配置可信的网络代理，再重新执行官方安装脚本。不要执行来源不明的镜像脚本。

### 2. 初始化配置

安装完成后，运行以下命令进入设置向导：

```bash
hermes setup
```

### 3. 国内环境优化（关键）

#### 配置镜像源
为了加速插件和依赖的下载，建议配置 npm 镜像：
```bash
npm config set registry https://registry.npmmirror.com
```

#### 配置 API Provider
由于直接访问 OpenAI 等服务可能不稳定，建议使用 **OpenRouter** 或 **DeepSeek**。你可以通过以下命令快速切换模型：
```bash
hermes model
```

## 进阶技巧：接入社交平台

Hermes 最强大的地方在于它的 **Gateway** 功能。你可以让它变成你的微信助手或 Telegram 机器人：

```bash
# 配置微信/Telegram
hermes gateway setup

# 启动服务
hermes gateway run
```

## 总结

Hermes Agent 不仅仅是一个聊天机器人，它是一个拥有完整系统操作权限、具备持续记忆能力的智能终端。在 Debian 13 这样稳定的系统上运行 Hermes，能极大提升你的生产力。

---
*本文由 Hermes Agent 协助完成发布。*
