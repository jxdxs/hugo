---
title: "OpenClaw 入门：从安装到第一次对话"
date: 2026-07-27T16:45:00+08:00
draft: false
description: "面向新手的 OpenClaw 安装、初始化、网关验证与安全配置教程。"
tags: [OpenClaw, AI, 教程]
categories: [Agent]
---

OpenClaw 是一个可以连接大模型、工具和聊天渠道的个人 AI 助手框架。本文以 Linux/macOS 为例，带你从零完成安装，并运行第一次对话。

> 本文根据 OpenClaw 官方文档整理。不同版本的命令和配置字段可能会变化，遇到差异时请优先参考官方文档。

## 一、准备工作

开始前准备好以下内容：

- 一台 Linux 或 macOS 电脑；Windows 用户可以使用 PowerShell 或 WSL2
- Node.js 22.22.3+、24.15+ 或 25.9+，官方推荐 Node.js 24
- 一个大模型服务商的 API Key，例如 OpenAI、Anthropic 或 Google
- 能够正常访问相关软件源和模型服务的网络环境

先检查 Node.js 版本：

```bash
node --version
```

如果尚未安装 Node.js，可以参考 OpenClaw 官方的 Node.js 安装说明。不要使用过旧的 Node.js 版本，否则可能遇到依赖或运行时错误。

## 二、安装 OpenClaw

在 Linux 或 macOS 终端执行官方安装脚本：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装完成后，确认命令可用：

```bash
openclaw --version
```

Windows PowerShell 用户可以执行：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

如果你更偏好手动管理依赖，也可以使用 npm、Docker、Nix 等安装方式。新手建议先使用官方安装脚本，流程更简单。

## 三、运行初始化向导

安装完成后运行：

```bash
openclaw onboard --install-daemon
```

初始化向导会引导你完成：

1. 选择模型服务商
2. 填写 API Key
3. 配置 Gateway
4. 安装后台服务
5. 完成基础工作区设置

API Key 属于敏感凭据，不要发布到 GitHub、博客、截图或聊天群组中。如果暂时不想配置某个可选功能，可以先跳过，之后再运行：

```bash
openclaw configure
```

## 四、检查 Gateway 状态

初始化完成后，查看网关是否正常运行：

```bash
openclaw gateway status
```

正常情况下，你会看到 Gateway 正在监听本机端口。Gateway 是 OpenClaw 的核心服务，负责协调模型、工具、频道和控制界面。

如果服务没有启动，可以先查看诊断信息：

```bash
openclaw status
```

配置发生变化后，使用重启命令让新配置生效：

```bash
openclaw gateway restart
```

## 五、打开控制面板并发送第一条消息

运行：

```bash
openclaw dashboard
```

命令会打开浏览器中的 Control UI。进入页面后，在聊天窗口发送一条简单消息，例如：

```text
你好，请介绍一下你自己。
```

如果能够收到模型回复，说明基础安装、认证和 Gateway 都已经正常工作。

## 六、连接 Telegram 等聊天渠道

OpenClaw 支持 Telegram、Discord、Slack、Signal、WhatsApp 等多个渠道。以 Telegram 为例，通常需要：

1. 在 Telegram 中通过 BotFather 创建机器人
2. 保存 Bot API Token
3. 在 OpenClaw 中配置 Telegram 频道
4. 设置允许访问的用户白名单
5. 重启 Gateway 并发送测试消息

建议始终配置白名单，只允许自己的 Telegram 用户 ID 访问。不要把 `allow_from` 留空，否则可能让任何人都能调用你的助手。

## 七、常见问题排查

### 1. `openclaw` 命令不存在

重新打开终端，或检查安装目录是否已经加入 PATH：

```bash
command -v openclaw
```

如果仍然找不到命令，重新运行安装流程，并检查安装过程中的错误信息。

### 2. Gateway 没有运行

先查看状态：

```bash
openclaw gateway status
```

再查看整体诊断信息：

```bash
openclaw status
```

修改配置后记得重启 Gateway：

```bash
openclaw gateway restart
```

### 3. API Key 无效

确认 API Key 没有多余空格、没有过期，并且对应服务商账户有可用额度。不要把完整 Key 粘贴到公开日志中。

### 4. 频道能收到消息但没有回复

重点检查：

- Gateway 是否正在运行
- 频道 Token 是否正确
- 当前用户是否在白名单中
- 模型 API 是否可用
- 日志中是否出现认证或网络错误

## 八、安全建议

OpenClaw 可以调用浏览器、命令行和其他工具，因此建议从一开始就做好基本安全设置：

- API Key 和 Bot Token 只保存在本机安全配置中
- 不要把配置文件、工作区记忆和密钥提交到 Git 仓库
- 聊天频道使用用户白名单
- Gateway 默认只监听本机，不要在没有认证和防火墙保护的情况下暴露到公网
- 定期更新 OpenClaw 和 Node.js
- 对陌生消息中的命令、链接和文件保持谨慎

## 结语

最短的入门路径可以概括为：

```text
安装 Node.js
  → 安装 OpenClaw
  → 运行 onboard
  → 检查 Gateway
  → 打开 dashboard
  → 发送第一条消息
```

先让本地控制面板正常工作，再逐步接入 Telegram 等外部频道，会比一次性配置所有功能更容易排错。

官方文档：<https://docs.openclaw.ai/start/getting-started>
