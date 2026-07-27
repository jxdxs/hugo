---
title: "ZeroClaw 入门：从安装到运行第一个 AI 助手"
date: 2026-07-27T17:50:00+08:00
draft: false
description: "面向新手的 ZeroClaw 安装、模型配置、命令行对话、Gateway 与安全使用教程。"
categories: [Agent]
tags: [ZeroClaw, AI, 教程]
---

ZeroClaw 是一个使用 Rust 编写的轻量级 AI Agent 运行时。它以单个二进制程序运行，可以连接 OpenAI、Anthropic、Ollama 等多种模型服务，也支持 Telegram、Discord、Matrix、邮件、Webhook 和命令行等多种交互渠道。

与普通的聊天客户端不同，ZeroClaw 不只是调用大模型回答问题，还可以通过工具访问 Shell、浏览器、HTTP 服务、硬件设备和自定义 MCP 服务。

本文将带你完成：

1. 了解 ZeroClaw 的基本工作方式；
2. 安装 ZeroClaw；
3. 使用 Quickstart 初始化配置；
4. 配置模型服务；
5. 运行第一次命令行对话；
6. 注册后台服务；
7. 了解安全配置和常见问题。

> 本文根据 ZeroClaw 官方仓库和官方文档整理。ZeroClaw 仍在快速迭代，命令、配置字段和功能可能随版本变化。实际使用时，请以官方文档为准。

官方项目地址：<https://github.com/zeroclaw-labs/zeroclaw>

官方文档：<https://docs.zeroclawlabs.ai/>

## 一、ZeroClaw 是什么

ZeroClaw 可以理解为一个运行在本机上的 AI Agent 框架。

它的大致工作流程如下：

```text
用户
  ↓
命令行、Telegram、Discord、Webhook 等渠道
  ↓
ZeroClaw Agent Runtime
  ├── 模型提供商
  ├── 工具系统
  ├── 记忆系统
  ├── 安全策略
  └── Gateway / Dashboard
  ↓
AI 助手执行任务并返回结果
```

ZeroClaw 的主要特点包括：

- 单个 Rust 二进制程序；
- 支持多种模型提供商；
- 支持本地 Ollama 等模型服务；
- 支持多个聊天渠道；
- 支持 Shell、浏览器、HTTP 等工具；
- 支持 Gateway 和 Web Dashboard；
- 支持记忆、定时任务和 SOP 工作流；
- 支持 Linux、macOS、Windows、FreeBSD、NixOS 和 Docker；
- 默认强调权限控制、审批和沙箱隔离。

需要注意的是，ZeroClaw 不是某个具体模型。它本身不提供大模型，而是负责连接和管理你选择的模型服务。

## 二、安装前准备

ZeroClaw 支持多种操作系统。新手通常可以选择 Linux、macOS、Windows 或 Docker。使用 Linux 时，建议准备一个普通用户账号、稳定的网络连接，以及至少一个模型服务的 API Key，或者已经在本地运行 Ollama。

Linux 用户可以使用下面的命令查看 CPU 架构：

```bash
uname -m
```

常见结果包括：

```text
x86_64
aarch64
arm64
```

如果官方 Release 提供了对应平台的预编译版本，优先使用预编译版本，不需要额外安装 Rust 编译环境。

## 三、安装 ZeroClaw

官方提供了安装脚本。最简单的安装方式是：

```bash
curl -fsSL https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/master/install.sh | bash
```

安装脚本会优先尝试使用预编译二进制。如果当前平台没有匹配的预编译版本，则可能回退到源码构建。

安装完成后，检查版本：

```bash
zeroclaw --version
```

如果终端显示版本号，说明程序已经安装成功。还可以查看帮助：

```bash
zeroclaw --help
```

### 使用 Git 仓库安装

如果你希望查看源码，或者准备参与开发，也可以克隆仓库：

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh
```

如果只希望安装程序，不运行交互式初始化，可以使用：

```bash
./install.sh --skip-quickstart
```

### 常用安装参数

```bash
./install.sh --prebuilt                 # 强制使用预编译版本
./install.sh --source                   # 强制从源码构建
./install.sh --preset minimal           # 使用最小功能预设
./install.sh --list-features            # 查看可用功能
./install.sh --prefix /tmp/zeroclaw     # 安装到自定义目录
./install.sh --dry-run --prebuilt       # 只预览，不执行安装
```

## 四、使用 Quickstart 初始化

安装完成后，可以运行：

```bash
zeroclaw quickstart
```

Quickstart 会引导你完成基本配置，例如选择模型提供商、设置 API Key、选择模型名称、创建默认 Agent 和进行连接测试。

ZeroClaw 默认使用 TOML 配置文件：

```text
~/.zeroclaw/config.toml
```

初始化后，可以查看配置目录：

```bash
ls -la ~/.zeroclaw
```

如果你只想先安装程序，之后再初始化，也可以稍后手动运行 `zeroclaw quickstart`。

> API Key 属于敏感凭据，不要把它提交到 Git 仓库，也不要粘贴到公开论坛或截图中。

## 五、配置模型服务

ZeroClaw 支持 OpenAI、Anthropic、Ollama、OpenAI Compatible API 等多种模型提供商。配置通常需要包含：

- 模型提供商类型；
- 模型别名；
- 模型名称；
- API Key 或认证方式；
- Agent 使用的模型别名。

官方配置结构采用提供商、Agent 和安全策略等多个区块。不同版本的字段可能变化，因此建议优先通过 `zeroclaw quickstart` 生成配置，而不是完全手写。

如果使用兼容 OpenAI API 的服务，通常还需要配置 API 地址、API Key、模型名称和 API 协议类型。配置完成后，可以使用：

```bash
zeroclaw auth status
```

查看当前认证状态。

如果模型请求失败，可以先检查：

1. API Key 是否正确；
2. API 地址是否正确；
3. 模型名称是否存在；
4. 当前账号是否有调用权限；
5. 网络是否能访问模型服务。

## 六、运行第一次 AI 对话

初始化完成后，可以直接启动交互式 Agent：

```bash
zeroclaw agent
```

如果配置中有多个 Agent，可以通过别名选择：

```bash
zeroclaw agent -a <alias>
```

例如：

```bash
zeroclaw agent -a assistant
```

启动后，可以输入：

```text
你好，请介绍一下你自己。
```

也可以测试一个简单任务：

```text
请帮我列出当前目录中的文件。
```

如果 ZeroClaw 获得了 Shell 工具权限，它可能会先请求批准，再执行对应操作。建议第一次测试时使用无害、只读的任务，例如：

```text
当前系统是什么操作系统？
```

或者：

```text
请告诉我当前工作目录。
```

不要一开始就让 Agent 执行删除文件、修改系统配置或操作生产环境。

## 七、Agent 与工具

ZeroClaw 的 Agent 不只是文本问答程序，它还可以调用工具完成任务。常见工具类型包括：

- Shell 命令；
- 浏览器操作；
- HTTP 请求；
- 文件读写；
- 数据库或记忆存储；
- MCP 服务；
- GPIO、I2C、SPI 和 USB 硬件接口。

例如，你可以让 Agent：

```text
读取当前项目的 README，并总结主要功能。
```

或者：

```text
检查当前目录下最近修改的文件。
```

涉及修改文件或运行高风险命令时，ZeroClaw 的安全策略可能要求人工确认。这类审批机制很重要，因为 Agent 可能具有实际的系统操作能力。建议先熟悉工具权限，再逐步扩大授权范围。

## 八、启动 Gateway

除了命令行交互，ZeroClaw 还支持 Gateway，用于提供 HTTP、WebSocket 和 Dashboard 能力。可以先查看帮助：

```bash
zeroclaw gateway --help
```

Gateway 通常可以用于：

- 为 Web 客户端提供接口；
- 管理 Agent；
- 查看记忆；
- 编辑配置；
- 管理定时任务；
- 检查工具调用；
- 对接其他客户端。

如果 Gateway 监听在公网地址，必须额外配置认证、防火墙和访问控制。不建议直接把未保护的 Gateway 端口暴露到互联网。

## 九、注册为系统服务

如果希望 ZeroClaw 在后台持续运行，可以使用：

```bash
zeroclaw service install
```

启动服务：

```bash
zeroclaw service start
```

查看服务状态：

```bash
zeroclaw service status
```

停止服务：

```bash
zeroclaw service stop
```

不同操作系统的服务管理方式可能不同：Linux 通常使用 systemd，macOS 通常使用 launchd，Windows 使用 Windows Service。如果服务启动失败，应查看系统服务日志，并检查模型认证和 Gateway 配置。

## 十、接入 Telegram

ZeroClaw 支持 Telegram 等聊天渠道。接入流程通常包括：

1. 在 Telegram 中通过 BotFather 创建机器人；
2. 获取 Bot Token；
3. 在 ZeroClaw 配置中启用 Telegram；
4. 设置允许访问的用户或聊天；
5. 启动 Gateway 或后台服务；
6. 在 Telegram 中发送测试消息。

Bot Token 属于高敏感凭据。不要把它提交到博客仓库、GitHub Issue 或公开日志中。

建议同时配置白名单，只允许自己的 Telegram 用户 ID 使用机器人。如果不配置白名单，陌生用户可能也能向 Bot 发送消息，甚至诱导 Agent 调用工具。因此正式使用前，务必限制允许访问的用户。

Telegram 配置字段可能随 ZeroClaw 版本变化。建议使用 Quickstart 或当前版本官方 Channels 文档，不要直接照抄旧版本示例。

## 十一、安全配置建议

### 1. 不要使用过度权限的系统账号

不建议让 Agent 以 root 或 Administrator 身份运行。更好的做法是创建专用系统用户、使用独立工作目录、只授予必要的文件权限，并避免让 Agent 访问 SSH 私钥、密码和生产配置。

### 2. 配置工作区边界

尽量让 Agent 只访问专门的工作目录，例如：

```text
/home/kevin/zeroclaw-workspace
```

不要默认让它访问整个用户目录。

### 3. 谨慎开启高风险工具

Shell、浏览器、文件写入和网络请求工具都可能带来风险。初次使用时建议开启审批，对删除、覆盖、发布等操作要求人工确认，不要在生产服务器上直接测试高权限功能。

### 4. API Key 不要写入 Git

配置敏感信息时，可以把 ZeroClaw 配置目录加入忽略列表，或者使用环境变量和专门的凭据管理方式。提交前检查：

```bash
git status
```

### 5. Gateway 不要裸奔

如果启用了 Gateway，不要直接暴露到公网。应使用身份认证、防火墙、HTTPS 和可信网络访问控制，并定期检查访问日志。

### 6. 谨慎使用 YOLO 模式

ZeroClaw 提供了面向开发环境的 YOLO 模式，可以跳过部分安全确认。它只适合临时测试、隔离的开发机器或没有重要数据的容器。不建议在包含个人文件、SSH 密钥、生产数据的机器上使用。

## 十二、常见问题

### 找不到 `zeroclaw` 命令

可能是安装目录没有加入 PATH。可以先查看：

```bash
which zeroclaw
```

如果没有输出，检查安装脚本提示的安装路径，并将该目录加入 PATH。

### Quickstart 无法连接模型

依次检查：

```bash
zeroclaw auth status
```

然后确认 API Key 没有多余空格、模型名称和 API 地址正确、服务商账号可用，以及本机网络正常。

### 模型流式请求失败

部分服务商可能不支持流式响应，或者中间代理不兼容。ZeroClaw 可能会自动回退到非流式请求。此时应先检查认证状态和服务商配置。

### Agent 不允许执行命令

这通常是安全策略在起作用。建议查看当前 Agent 的风险配置，检查工具是否启用，并根据提示人工批准安全的操作。不要为了省事直接关闭所有安全检查。

### Telegram Bot 没有回应

可以检查 Bot Token、Telegram 渠道、用户白名单、Gateway 或后台服务状态，以及 ZeroClaw 日志中的报错。同时确认 Bot 没有被其他程序同时使用。

## 十三、卸载 ZeroClaw

如果需要卸载 ZeroClaw，可以使用：

```bash
./install.sh --uninstall
```

如果已经注册为系统服务，应先停止并卸载服务，再删除配置目录。删除配置前请确认其中没有需要保留的记忆、工作文件或凭据。

## 十四、总结

ZeroClaw 是一个轻量、可自托管、面向多平台的 AI Agent 运行时。最基本的使用流程是：

```bash
# 安装
curl -fsSL https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/master/install.sh | bash

# 初始化
zeroclaw quickstart

# 查看认证状态
zeroclaw auth status

# 启动命令行 Agent
zeroclaw agent

# 注册后台服务
zeroclaw service install
zeroclaw service start
```

初学者建议按照以下顺序使用：

1. 先安装 ZeroClaw；
2. 使用 Quickstart 配置模型；
3. 测试命令行对话；
4. 熟悉工具审批机制；
5. 再启用 Gateway；
6. 最后接入 Telegram 等外部渠道；
7. 全程使用白名单、最小权限和独立工作目录。

ZeroClaw 的能力越强，越需要重视权限和数据安全。不要把它当作普通聊天机器人，而应该把它视为一个可以执行实际操作的软件助手。

> ZeroClaw 仍处于持续开发阶段。安装命令、配置格式和部分功能可能发生变化，部署前请查看官方仓库和官方文档的最新说明。

## 参考资料

- ZeroClaw 官方仓库：<https://github.com/zeroclaw-labs/zeroclaw>
- ZeroClaw 官方文档：<https://docs.zeroclawlabs.ai/>
- ZeroClaw Quickstart：<https://github.com/zeroclaw-labs/zeroclaw/blob/master/docs/book/src/getting-started/quickstart.md>
- ZeroClaw 配置文档：<https://github.com/zeroclaw-labs/zeroclaw/tree/master/docs/book/src/providers>
