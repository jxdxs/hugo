---
title: "PicoClaw 入门：从安装到运行第一个 AI 助手"
date: 2026-07-27T16:00:00+08:00
draft: false
description: "一篇面向新手的 PicoClaw 安装、模型配置、命令行使用与 Telegram 接入教程。"
tags: [PicoClaw, AI, 教程]
categories: [技术]
---

PicoClaw 是由 Sipeed 发起、使用 Go 语言编写的轻量级个人 AI 助手。它采用单二进制文件，支持 x86_64、ARM64、MIPS、RISC-V 等多种架构，适合部署在普通 Linux 服务器、树莓派和资源有限的开发板上。

本文以 Linux/macOS 的命令行为主，带你完成下载、初始化、配置模型，并运行第一个 AI 助手。

> 本文根据 PicoClaw 官方仓库和官方文档整理。PicoClaw 仍处于快速迭代阶段，配置字段可能随版本变化，请以官方文档为准。

## 一、准备工作

开始前准备好：

- 一台 Linux、macOS 或 Windows 电脑
- 一个可用的大模型 API Key
- 能访问 GitHub Releases 和模型服务的网络环境
- Linux 用户确认自己的 CPU 架构

Linux 查看架构：

```bash
uname -m
```

常见对应关系如下：

- `x86_64`：普通 Intel/AMD 64 位电脑和服务器
- `aarch64` 或 `arm64`：多数 64 位 ARM 设备
- `armv7l`：部分 32 位 ARM 设备
- `riscv64`：64 位 RISC-V 开发板

PicoClaw 的预编译版本不要求另外安装 Go。只有从源码构建时，才需要 Go 1.25+；如果还要构建 Web UI，则需要 Node.js 22+ 和 pnpm 10.33.0+。

## 二、安装方式一：下载官方预编译版本

这是最适合新手的方式。

### 1. 从官方网站下载

打开 PicoClaw 官方网站：

<https://picoclaw.io/>

官方网站会根据平台提供下载选项。也可以从官方 GitHub Releases 页面下载：

<https://github.com/sipeed/picoclaw/releases>

截至本文发布时，官方最新稳定版本为 `v0.3.1`。版本更新较快，下载时请选择 Releases 页面显示的最新版本，并确认文件与自己的系统架构匹配。

### 2. Linux 命令行下载示例

下面以 Linux ARM64 为例。请把版本号和文件名替换为 Releases 页面中的实际名称：

```bash
wget https://github.com/sipeed/picoclaw/releases/latest/download/picoclaw_Linux_arm64.tar.gz
tar xzf picoclaw_Linux_arm64.tar.gz
chmod +x picoclaw
sudo install -m 755 picoclaw /usr/local/bin/picoclaw
```

如果是普通 Intel/AMD 64 位 Linux，通常选择 `Linux_amd64`；如果是 RISC-V 设备，则选择对应的 `Linux_riscv64` 构建包。

确认安装成功：

```bash
picoclaw --version
```

如果终端提示 `command not found`，请检查 `/usr/local/bin` 是否在 PATH 中，或者直接使用当前目录下的 `./picoclaw` 命令。

### 3. 校验下载文件

Release 页面通常会提供校验和文件。生产环境或公网下载建议校验 SHA-256：

```bash
sha256sum picoclaw_Linux_arm64.tar.gz
```

将输出结果与官方 `checksums.txt` 中对应文件的值进行比较。不要从来历不明的第三方网站下载 PicoClaw，也不要运行无法确认来源的二进制文件。

## 三、安装方式二：从源码构建

如果你需要开发、修改源码，或者官方 Release 暂时没有适合你设备的构建版本，可以从源码构建：

```bash
git clone https://github.com/sipeed/picoclaw.git
cd picoclaw
make deps
make build
```

构建结果通常会出现在项目的 `build/` 目录中。需要构建 Web UI Launcher 时，再执行：

```bash
(cd web/frontend && pnpm install --frozen-lockfile)
make build-launcher
```

如果只想构建当前平台，使用 `make build` 即可；如果需要构建多个由 Makefile 管理的平台，可以使用：

```bash
make build-all
```

新手没有开发需求时，不建议一开始就从源码构建，直接使用官方预编译版本更省事。

## 四、初始化 PicoClaw

安装完成后，运行初始化向导：

```bash
picoclaw onboard
```

首次运行会创建 PicoClaw 的配置目录和工作区，通常位于：

```text
~/.picoclaw/
```

其中常见文件包括：

```text
~/.picoclaw/config.json       # 常规配置
~/.picoclaw/.security.yml     # API Key、Token 等敏感配置
~/.picoclaw/workspace/        # 助手工作区
```

不同版本的初始化提示可能略有不同。完成后，可以查看帮助：

```bash
picoclaw --help
picoclaw onboard --help
```

## 五、配置模型和 API Key

PicoClaw 的模型配置位于 `~/.picoclaw/config.json`。配置文件中的 `model_name` 应与默认 Agent 使用的模型名称对应；模型标识通常使用 `协议/模型名` 的形式，例如：

```json
{
  "agents": {
    "defaults": {
      "model_name": "my-model"
    }
  },
  "model_list": [
    {
      "model_name": "my-model",
      "model": "openai/gpt-4o",
      "api_base": "https://api.openai.com/v1"
    }
  ]
}
```

API Key 不建议直接写进 `config.json`。较新版本支持把敏感配置单独放入 `~/.picoclaw/.security.yml`：

```yaml
model_list:
  my-model:
    api_keys:
      - "替换为你的模型 API Key"
```

然后限制文件权限：

```bash
chmod 600 ~/.picoclaw/.security.yml
```

注意：上面的 `my-model` 必须与 `config.json` 中的 `model_name` 完全一致。不同模型服务商需要使用对应的 `model`、`api_base` 和协议格式，请参考官方配置示例。

完成配置后，用一次性命令测试模型：

```bash
picoclaw agent -m "你好，请用一句话介绍自己。"
```

如果不带 `-m`，可以进入交互模式：

```bash
picoclaw agent
```

## 六、启动 Gateway

如果只需要在终端聊天，`picoclaw agent` 就够了；如果要连接 Telegram 等聊天渠道，则需要启动 Gateway：

```bash
picoclaw gateway
```

Gateway 默认面向本机运行。保持这个终端窗口打开，然后在另一个终端中检查进程或发送测试消息。

如果需要让 Gateway 在后台运行，建议使用 systemd、Docker 或其他受控的服务管理方式，并同时配置日志、自动重启和访问控制。不要为了“方便访问”而直接把未配置认证的 Gateway 暴露到公网。

## 七、接入 Telegram

接入 Telegram 通常需要以下步骤：

1. 在 Telegram 中找到 BotFather 并创建机器人
2. 保存 Bot API Token
3. 在 PicoClaw 中启用 Telegram 频道
4. 将 Token 放到 `.security.yml`，不要写入 Git 仓库
5. 配置自己的 Telegram 用户 ID 白名单
6. 重启 Gateway 并发送测试消息

敏感 Token 可以放在：

```yaml
channels:
  telegram:
    token: "替换为你的 Telegram Bot Token"
```

频道的普通开关和白名单等非敏感设置放在 `config.json`。配置结构会因 PicoClaw 版本而变化，建议先查看仓库中的示例配置和官方频道文档，不要直接照搬其他版本的字段。

最重要的一点是：不要把允许访问的用户列表留空。空白名单可能导致陌生人也能调用你的机器人，进而消耗 API 额度，甚至触发工具操作。

## 八、使用 Web UI Launcher（可选）

PicoClaw 也提供 Web UI Launcher。下载对应的 Launcher 后可以直接运行：

```bash
picoclaw-launcher
```

然后在浏览器打开：

<http://localhost:18800>

通常的配置顺序是：

1. 配置 Provider，并填写模型 API Key
2. 配置聊天频道，例如 Telegram
3. 启动 Gateway
4. 发送测试消息

在远程服务器、虚拟机或 Docker 环境中，如果确实需要从其他机器访问 Web UI，才考虑使用 `-public`；同时必须配合防火墙、反向代理和认证。不要把管理界面直接暴露在公网。

## 九、常见问题

### 1. 架构下载错了

如果运行时出现 `Exec format error`，通常是二进制架构与设备不匹配。重新执行：

```bash
uname -m
```

然后下载对应架构的 Release 文件。

### 2. 没有执行权限

Linux 下执行：

```bash
chmod +x picoclaw
```

### 3. 模型请求失败

依次检查：

- API Key 是否正确、是否过期
- `model_name` 是否一致
- `model` 和 `api_base` 是否匹配服务商
- 服务器能否访问模型 API
- 账户是否有余额或调用额度

不要把包含完整 API Key 的日志直接发到群组或提交到 GitHub。

### 4. Telegram 没有回复

检查以下项目：

- Gateway 是否正在运行
- Bot Token 是否正确
- Telegram 用户 ID 是否在白名单中
- 模型 API 是否可用
- 配置修改后是否重启了 Gateway

### 5. 配置文件被提交到 Git

立即停止继续分享该文件，并到对应服务商后台撤销或轮换 API Key、Bot Token。随后把密钥移到 `.security.yml`，并在项目的 `.gitignore` 中加入：

```gitignore
.security.yml
config.json
```

## 十、安全建议

PicoClaw 很轻量，但它仍然是一个可以调用模型和工具的 AI Agent，轻量不等于可以忽略安全：

- 只从 `picoclaw.io` 或 `github.com/sipeed/picoclaw` 下载
- API Key、Bot Token、密码和私钥不要提交到 Git
- 使用 `.security.yml` 分离敏感配置，并设置 `chmod 600`
- Telegram、QQ 等外部频道必须设置用户白名单
- Gateway 和 Web UI 默认只监听本机；公网访问前先配置认证和防火墙
- 定期检查官方 Release 和安全公告
- PicoClaw 仍在快速开发阶段，正式生产部署前应先进行隔离和权限测试

## 结语

PicoClaw 的入门路径可以概括为：

```text
确认 CPU 架构
  → 下载官方二进制
  → picoclaw onboard
  → 配置模型和 API Key
  → picoclaw agent 测试
  → picoclaw gateway
  → 按需接入 Telegram 或 Web UI
```

建议先在本机完成命令行对话，再接入 Telegram 等外部频道。这样出现问题时，可以快速判断到底是模型配置、Gateway，还是聊天渠道本身出了问题。

官方资源：

- 项目仓库：<https://github.com/sipeed/picoclaw>
- 官方网站：<https://picoclaw.io/>
- 官方文档：<https://docs.picoclaw.io/>
- Release 下载：<https://github.com/sipeed/picoclaw/releases>
