---
title: "OpenClaw 完全新手指南：从安装到打造个人 AI 助手"
date: 2026-07-27T18:12:00+08:00
draft: false
description: "面向零基础用户的 OpenClaw 完整上手教程，涵盖安装、Gateway、工作区、模型、渠道、技能、多智能体与安全设置。"
categories: [Agent]
tags: [OpenClaw, AI, 入门教程]
---

OpenClaw 是一个运行在自己设备上的个人 AI 助手网关。它可以把大模型连接到 Telegram、WhatsApp、Discord、Slack、Signal、WebChat 等聊天渠道，也能配合工作区、记忆、浏览器、文件和自动化工具完成更复杂的任务。

你可以把它理解成三部分的组合：

- **模型**负责理解问题和生成回答；
- **Gateway**负责连接渠道、管理会话和调度任务；
- **工具与工作区**让助手能够读取资料、处理文件和执行经过授权的操作。

本教程适合第一次接触 OpenClaw 的用户，将从最小可用配置开始，逐步完成安装、初始化、对话和安全加固。

> OpenClaw 正在持续更新。不同版本的命令、配置字段和渠道选项可能存在差异，执行时请先用 `openclaw --help` 检查当前版本的帮助信息。

## 一、OpenClaw 能做什么

安装并配置好 OpenClaw 后，你可以通过聊天软件向自己的助手发送消息，让它完成例如：

- 总结文章、会议记录和项目文档；
- 整理工作区中的文件；
- 查询天气、网页和公开资料；
- 协助编写、检查和测试代码；
- 通过定时任务发送提醒；
- 连接手机或其他节点执行特定操作；
- 为不同项目使用不同的 Agent 和工作目录。

它的基本结构可以简化为：

```text
聊天渠道 / Control UI / 命令行
              ↓
           Gateway
              ↓
   Agent + 模型 + 会话上下文
              ↓
     工作区、记忆、技能和工具
```

Gateway 是整个系统的核心。渠道负责把消息送进来，Gateway 决定消息交给哪个 Agent 和会话，Agent 再根据权限调用模型或工具。

## 二、安装前准备

### 系统要求

通常需要：

- macOS、Linux，或 Windows 上的 WSL2；
- Node.js 22 LTS 或更新版本；
- 一个可用的大模型服务账号或本地模型服务；
- 能够访问 npm 和模型 API 的网络环境。

先查看 Node.js 版本：

```bash
node --version
npm --version
```

如果 Node.js 版本过旧，建议先升级。OpenClaw 依赖现代 Node.js 运行时，版本不匹配可能导致安装成功但启动失败。

## 三、安装 OpenClaw

### macOS / Linux 安装脚本

官方安装脚本可以自动完成安装：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

执行脚本前，建议先查看脚本内容和目标环境，尤其是在生产服务器上安装时。也可以选择使用 npm：

```bash
npm install -g openclaw@latest
```

### Windows

Windows 用户可以在 PowerShell 中使用：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

如果你更熟悉 Linux 工具链，也可以使用 WSL2，再按照 Linux 的安装方式操作。

### 检查安装

```bash
openclaw --version
openclaw --help
```

如果能看到版本号和命令帮助，说明主程序已经进入 PATH。

## 四、运行初始化向导

OpenClaw 提供初始化向导，用于设置模型、工作区、Gateway 和基础服务：

```bash
openclaw onboard
```

如果希望同时安装后台服务，可以使用：

```bash
openclaw onboard --install-daemon
```

向导通常会引导你完成以下内容：

1. 选择模型提供商；
2. 设置认证信息；
3. 指定 Agent 工作区；
4. 配置 Gateway；
5. 选择需要连接的渠道；
6. 创建或安装后台服务。

第一次配置时不必一次开启所有功能。推荐先只配置一个模型和命令行或 Control UI，确认基本对话成功后，再添加 Telegram 等外部渠道。

## 五、启动 Gateway 和 Control UI

查看 Gateway 状态：

```bash
openclaw gateway status
```

前台启动适合调试：

```bash
openclaw gateway --port 18789 --verbose
```

如果已经通过向导安装了后台服务，可以使用对应的 Gateway 管理命令启动或重启服务：

```bash
openclaw gateway start
openclaw gateway restart
```

打开浏览器控制面板：

```bash
openclaw dashboard
```

本机常见访问地址是：

```text
http://127.0.0.1:18789/
```

Control UI 适合进行第一次对话、查看会话和确认 Gateway 是否正常。远程访问时，不要直接把未经认证的端口暴露到公网，应使用 SSH 隧道、Tailscale 或其他经过身份认证的私有访问方式。

## 六、Workspace：助手的工作目录

Workspace 是 Agent 的主要工作区域。它既可以保存项目文件，也可以提供每次对话需要的背景资料。

常见文件包括：

```text
~/.openclaw/workspace/
├── AGENTS.md
├── SOUL.md
├── USER.md
├── IDENTITY.md
├── TOOLS.md
├── MEMORY.md
└── skills/
```

这些文件可以分别承担不同职责：

- `AGENTS.md`：工作规则、操作约定和项目说明；
- `SOUL.md`：助手的语气、边界和行为风格；
- `USER.md`：用户偏好和长期背景；
- `IDENTITY.md`：助手名称、表情和身份信息；
- `TOOLS.md`：本机工具、设备和环境说明；
- `MEMORY.md`：适合长期保留的稳定记忆；
- `skills/`：当前工作区可使用的技能。

不要把 API Key、Bot Token、SSH 私钥、密码或其他敏感凭据放进这些文件。也不要把整个 `~/.openclaw` 目录直接提交到公开仓库。

建议为每个重要项目建立独立子目录，明确告诉 Agent 哪些文件可以读取和修改。工作区越清晰，Agent 越容易得到准确上下文。

## 七、模型配置

OpenClaw 本身不是模型，而是模型的运行和连接层。你需要配置至少一个模型提供商，例如：

- OpenAI；
- Anthropic；
- Google；
- OpenAI Compatible API；
- 本地 Ollama 或其他本地服务。

配置模型时，通常需要确定：

- 提供商名称；
- 模型 ID；
- API 地址；
- API Key 或其他认证方式；
- 主模型与备用模型。

如果当前版本提供认证或配置检查命令，可以使用：

```bash
openclaw auth status
openclaw config get agents.defaults.model.primary
```

具体字段以当前版本 Schema 为准。不要把网上旧版本的 JSON 示例直接覆盖到现有配置上。正确做法是先备份，再只合并需要修改的字段。

一个稳妥的模型策略是：

```text
主模型：日常质量较高的模型
备用模型：主模型不可用时自动切换
本地模型：适合隐私或离线测试
```

第一次测试建议使用一个响应稳定的主模型，等基础链路正常后，再配置备用模型和本地模型。

## 八、完成第一次对话

除了 Control UI，也可以通过命令行调用 Agent：

```bash
openclaw agent --message "你好，请介绍一下你自己。"
```

还可以测试一个只读任务：

```bash
openclaw agent --message "请告诉我当前工作目录，并列出其中的文件。"
```

第一次测试建议遵循以下顺序：

1. 让 Agent 回答一个普通问题；
2. 让 Agent 读取一份无敏感信息的文本；
3. 让 Agent 总结工作区内容；
4. 最后再测试文件修改或 Shell 工具。

涉及写文件、安装软件、发送消息或执行系统命令时，应保留人工确认。不要一开始就使用高权限账号或让 Agent 访问整个用户目录。

## 九、接入 Telegram 等聊天渠道

### Telegram 的基本流程

1. 使用 BotFather 创建 Bot；
2. 保存 Bot Token；
3. 在 OpenClaw 中启用 Telegram；
4. 设置私聊策略和用户白名单；
5. 启动 Gateway；
6. 从自己的 Telegram 账号发送测试消息。

Telegram 配置示例可以表达为以下思路：

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:your-user-id"],
    },
  },
}
```

这是示意结构，不同版本的字段名称可能不同，请以当前版本配置 Schema 为准。

### 私聊安全策略

常见策略包括：

- `pairing`：未知用户先获得配对码，批准后才能使用；
- `allowlist`：只允许明确列出的用户；
- `open`：允许所有用户私聊，风险最高；
- `disabled`：关闭私聊入口。

个人 Bot 推荐使用配对或白名单。不要为了测试方便长期使用公开 DM。

### 群聊安全

群聊中建议同时启用：

- 群组白名单；
- 只有提及 Bot 时才回应；
- 限制高风险工具；
- 独立的群聊会话；
- 必要时使用专门的低权限 Agent。

陌生群组中的消息属于不可信输入，不要让它们直接触发文件删除、外部发布、Shell 或 Gateway 管理操作。

## 十、会话与多智能体

### 会话隔离

不同渠道、发送者、群组和线程可以使用不同会话。开启合理的隔离后，工作聊天不会轻易带入私人聊天的上下文。

如果多人共用一个 Bot，建议使用按渠道和用户隔离的 DM 策略，并为群聊设置独立会话。涉及私人资料的任务，使用专用 Agent 和专用工作区更稳妥。

### 多智能体

当任务类型差异较大时，可以配置多个 Agent，例如：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        description: "通用个人助手",
        workspace: "~/.openclaw/workspace",
      },
      {
        id: "coder",
        description: "编程和代码审查助手",
        workspace: "~/.openclaw/workspace-coder",
      },
    ],
  },
}
```

多 Agent 的价值在于隔离，而不是简单增加数量。不同 Agent 可以拥有不同的模型、工作区、工具和安全策略。

适合拆分的场景包括：

- 通用助手处理日常问答；
- 编程 Agent 只访问代码目录；
- 写作 Agent 只处理内容工作区；
- 研究 Agent 负责读取网页并整理资料。

## 十一、安装和管理技能

技能是可复用的能力模块。它们可以帮助 OpenClaw 扩展天气查询、图像处理、GitHub 操作、视频处理等功能。

如果使用 ClawHub 生态，常见流程是：

```bash
npm install -g clawhub
clawhub search "weather"
clawhub install weather
clawhub list
```

安装技能前应检查：

- 技能来源是否可信；
- `SKILL.md` 是否说明了会执行什么操作；
- 是否需要 Shell、网络或凭据权限；
- 是否包含可疑的安装脚本；
- 是否与当前 OpenClaw 版本兼容。

不要因为技能名称看起来熟悉就直接安装。技能本质上可能包含可执行流程，权限应与普通软件安装同样谨慎。

## 十二、常用 CLI 操作

可以从这些命令开始熟悉 OpenClaw：

```bash
# 查看版本和帮助
openclaw --version
openclaw --help

# 查看整体状态
openclaw status

# Gateway 状态和日志
openclaw gateway status
openclaw logs --follow

# 打开控制面板
openclaw dashboard

# 运行诊断
openclaw doctor
```

配置读取和修改命令是否可用，取决于当前版本。修改前可以先读取目标值：

```bash
openclaw config get agents.defaults.workspace
```

确认字段后再执行修改，并立即做一次状态检查。不要在不清楚字段层级时使用整文件覆盖。

## 十三、Heartbeat 与定时任务

Heartbeat 适合周期性检查，例如定期查看待办事项、项目状态或即将到来的日程；Cron 更适合严格的时间点和一次性提醒。

设计自动化时，先从只读任务开始：

```text
读取状态 → 生成摘要 → 发给我确认
```

确认稳定后，再考虑有限的自动操作。以下动作建议始终保留人工确认：

- 对外发送邮件或消息；
- 发布博客和社交媒体内容；
- 删除或覆盖文件；
- 修改服务器配置；
- 操作支付、生产和权限系统。

自动化不是越多越好。每个定时任务都应该能说明触发时间、读取哪些数据、会产生什么副作用，以及失败后如何发现。

## 十四、安全设置与故障排查

### 推荐的安全基线

```text
Gateway：默认只监听本机或可信私有网络
私聊：使用 pairing 或 allowlist
群聊：限制群组并要求提及
工作区：只开放专用目录
Shell：默认需要确认
浏览器：只给确实需要的 Agent
外部发布：始终人工确认
凭据：不写入博客、Git 和普通记忆
```

### 配置错误

如果出现配置校验失败：

1. 备份当前配置；
2. 查看具体错误字段；
3. 用当前版本文档或 Schema 核对字段；
4. 只修正目标字段；
5. 重新运行 `openclaw doctor` 或状态检查。

### Gateway 无法启动

检查：

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

同时确认端口没有被占用、配置文件可读、服务用户有权访问工作区，以及 Node.js 版本符合要求。

### Bot 没有回应

重点检查渠道是否启用、凭据是否正确、用户是否被允许、Gateway 是否运行，以及是否存在另一个程序同时使用同一个 Bot Token。

### Session not found

会话可能已经过期、重置或被清理。可以从 Control UI 创建新会话，或根据当前渠道支持的命令重新开始对话。

## 十五、推荐的新手成长路线

不要一次配置全部功能，可以按照这条路线逐步扩展：

1. 安装 OpenClaw 并完成模型配置；
2. 用 Control UI 发送第一条消息；
3. 整理 Workspace 中的身份、用户和工具说明；
4. 接入 Telegram，并配置白名单；
5. 学习会话隔离和记忆管理；
6. 安装一个经过检查的技能；
7. 创建一个只读的定时任务；
8. 配置专用 Agent 和专用工作区；
9. 最后再开放浏览器、Shell、节点和外部自动化。

每完成一步，都先做小范围测试并保留回滚方式。

## 总结

OpenClaw 的核心并不只是“连接一个大模型”，而是把模型、渠道、会话、工作区和工具组织成一个可以长期使用的个人助手系统。

真正稳定的配置通常遵循四个原则：

- **先简单后复杂**：先跑通一个渠道，再扩展功能；
- **先隔离后共享**：不同用户、项目和群组不要默认共享上下文；
- **先确认后自动化**：高风险操作保留人工审批；
- **先备份后修改**：配置、记忆和工作区都应可恢复。

当你掌握这些基础之后，OpenClaw 才能从“会回答问题的 Bot”逐渐变成可靠的个人 AI 工作台。🦞
