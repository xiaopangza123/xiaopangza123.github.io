---
title: IntelliJ IDEA 深度集成指南：通过 ACP 协议接管你的 Hermes Agent
date: 2026-05-12 18:58:12
updated: 2026-08-20 13:13:46
description: 通过 ACP 将 Hermes Agent 接入 IntelliJ IDEA，统一使用项目上下文、工程工具与 MCP 能力。
categories:
  - AI 与智能体
tags:
  - Hermes
  - AI
  - 教程
  - IntelliJ IDEA
  - ACP
cover: /img/covers/connect-hermes-agent-to-intellij-idea-via-acp.png
top_img: /img/covers/connect-hermes-agent-to-intellij-idea-via-acp.png
comments: false
index: true
source: https://blog.xiaopangza.cn/archives/connect-hermes-agent-to-intellij-idea-via-acp
---

> 本文同步自 [Halo 主博客](https://blog.xiaopangza.cn/archives/connect-hermes-agent-to-intellij-idea-via-acp)，内容以两站最新版本为准。
作为一名开发者，如果你还在 IDE 和终端之间来回切换复制粘贴代码，那么这篇指南将彻底改变你的工作流。通过 **ACP (Agent Control Protocol)** 协议，我们可以将 **Hermes Agent** 深度嵌入到 IntelliJ IDEA 中，让它直接感知项目上下文，并代劳繁琐的编码任务。

---

## 💡 为什么选择 ACP 模式？

相比传统的 CLI 模式，在 IDEA 中使用 ACP 接入 Hermes 有以下显著优势：
- **零摩擦读写**：Hermes 可以直接修改当前编辑器中的文件。
- **全栈工具箱**：在 IDE 内直接调用 Hermes 强大的技能（Skills）和工具（Tools）。
- **统一管理**：所有的模型配置、记忆和 MCP 服务在终端和 IDE 间完美同步。

---

## 🛠️ 配置全流程

### 1. 基础环境
确保你的系统中已安装 Hermes 及其 ACP 适配组件：
```bash
pip install -e '.[acp]'
```
运行 `hermes acp` 确认服务能够正常进入监听状态。

### 2. 在 IDEA 中注册自定义智能体 (核心)
在 IntelliJ IDEA 的 ACP 插件中，最优雅的接入方式是通过修改 `acp.json` 来添加自定义智能体。

- **操作步骤**：
  1. 找到插件的配置存储文件 `acp.json`。
  2. 在 `agents` 数组中手动注入以下配置：

```json
{
  "name": "Hermes Agent",
  "command": "hermes",
  "args": ["acp"],
  "env": {
    "PATH": "/usr/local/bin:/usr/bin:/bin"
  }
}
```
*提示：如果 `hermes` 未在默认 PATH 中，请替换为绝对路径。*

---

## ⚡ 进阶：让 Hermes 指挥你的 IDEA (MCP 原生能力)

这是 ACP 集成的“杀手锏”功能：你可以通过 MCP 服务，让 Hermes 获得直接操作 IDEA 的权限。这意味着你可以直接在对话框中发令：

> “Hermes，帮我构建这个项目并启动应用。”

### 1. 核心操作
通过在 Hermes 中注册并启用 IDEA 提供的 MCP 接口，Hermes 将解锁以下原生能力：
- **项目构建 (Build)**：调用 IDEA 的编译引擎进行增量或全量构建。
- **运行/调试 (Run/Debug)**：一键启动当前配置好的运行实例。
- **版本控制 (VCS)**：通过 IDEA 的 Git 插件执行 Commit、Push 或查看 Diff。

### 2. 配置策略：归口管理
**重要提醒**：所有的 MCP 服务（无论是外部的还是 IDEA 原生的）**应当在 Hermes 中统一注册**，而不是在 IDEA 插件中单独配置。

- **操作方式**：
  ```bash
  # 在终端通过 Hermes CLI 注册 MCP 服务
  hermes mcp add idea-service --command "npx @some/idea-mcp-server"
  ```
- **原理**：IDEA 通过 ACP 连接到 Hermes，而 Hermes 作为“智能中枢”管理所有的 MCP 工具。这样你无论是在终端还是 IDE 里，都能使用完全一致的工具集。

---

## 🎯 极致体验技巧

- **智能审批**：对于危险命令（如重置分支），Hermes 会直接在 IDEA 中弹出审批窗口，安全且高效。
- **自动 Commit**：在完成功能修复后，直接对它说：“按照 conventional commit 规范提交代码”，它会调用 IDEA 的 VCS 接口帮你写好 Commit Message 并提交。

---

> **结语**：将 Hermes Agent 接入 IDEA 后，它不再只是一个聊天机器人，而是成为了一个拥有你所有 IDE 权限、真正理解工程细节的“虚拟副驾驶”。

---
*本文由已通过 ACP 深度集成至 IntelliJ IDEA 的 Hermes Agent 协作完成优化发布。*
