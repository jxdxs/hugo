---
title: "SSHTerm：用 Tauri 2 写了一个不到 5MB 的桌面 SSH/SFTP 客户端"
date: 2026-07-31T08:47:00+08:00
draft: false
description: "一个开源项目，Rust 后端 + 原生 Web 前端，就能做出原生 SSH 终端和 SFTP 文件管理器。聊聊它的技术选型、架构和代码设计。"
categories: [工具]
tags: [SSH, SFTP, Tauri, Rust, 桌面应用, 开源]
---

最近在 GitHub 上翻到一个有意思的项目——[SSHTerm](https://github.com/jxdxs/sshterm)，一个用 **Tauri 2** 构建的桌面原生 SSH/SFTP 客户端。

用了几天之后，我觉得它很值得聊聊。不是因为功能有多全（毕竟还是个 v0.1.0），而是因为它展示了**用 Tauri 2 做工具类桌面应用**的清爽路径。

## 为什么值得看

现在市面上 SSH 客户端不少：FinalShell、Termius、Xshell、PuTTY……但 SSHTerm 的定位很明确——**轻量、原生、开源、免费**。

几个关键数据：

- 安装包约 **5MB**（Tauri 的招牌优势）
- 后端 **Rust**，前端 **原生 HTML/CSS/JS + xterm.js**
- 支持 **Linux (.deb / .AppImage) + Windows (.msi)**
- **MIT 协议**，随便改

没有 Electron 的几百 MB 体积，不需要 Java 运行时，不需要 .NET 框架。一个 Rust 二进制 + 一个 WebView，搞定。

## 技术栈一览

从 `Cargo.toml` 和 `package.json` 可以清楚看到选型逻辑：

| 层 | 技术 | 为什么选它 |
| --- | --- | --- |
| 窗口框架 | Tauri 2.0 | 体积小、性能好、Rust 原生 |
| SSH 协议 | ssh2 (rust) | libssh2 绑定，成熟稳定 |
| 本地存储 | rusqlite | 嵌入式 SQLite，零配置 |
| 终端渲染 | xterm.js | 浏览器终端的事实标准 |
| 前端构建 | Vite 6 | 极快的 HMR 和构建 |
| CI/CD | GitHub Actions | 自动编译跨平台安装包 |

这里面最有意思的组合是 **Rust 处理 SSH/SFTP 协议 + WebView 渲染界面**。Tauri 2 的 `invoke` 机制让前端可以直接调用 Rust 函数，不需要额外的 HTTP 服务或 WebSocket 桥接。

## 代码结构

```
sshterm/
├── src-tauri/           # Rust 后端
│   ├── src/
│   │   ├── main.rs      # Tauri 入口 + 16 个命令注册
│   │   ├── ssh.rs       # SSH 连接 + SFTP 操作
│   │   └── store.rs     # SQLite 主机持久化
│   ├── Cargo.toml
│   └── tauri.conf.json
├── frontend/            # 前端 UI
│   ├── index.html       # 单页应用
│   ├── style.css        # Tokyo Night 暗色主题
│   └── app.js           # ~400 行前端逻辑
├── .github/workflows/
└── README.md
```

大概就是三个 Rust 源文件 + 三个前端文件，一个完整可用的 SSH 客户端。这种文件量在 Electron 项目里连配置文件都写不完。

## 深入看看代码

### Rust 后端：SSH 连接（src-tauri/src/ssh.rs）

核心是一个 `SshSession` 结构体，封装了 `ssh2::Session` 的所有操作：

```rust
pub struct SshSession {
    session: Mutex<Session>,
    sftp: Mutex<Option<Sftp>>,
}
```

连接逻辑很标准：TCP 建连 → SSH 握手 → 认证（密码或密钥）。但有几个细节做得不错：

1. **超时控制**：读/写超时都设了 30 秒，避免网络卡死 UI
2. **错误链完整**：每个错误都保留了上下文（`TCP connect failed: ...`、`Password auth failed: ...`），调试时能直接定位
3. **认证方式枚举**：`auth_type` 字段支持 `"password"` 和 `"key"`，密钥还可以带 passphrase

SFTP 操作也很完整——`list_dir`、`read_file`、`write_file`、`create_dir`、`remove`、`rename`、`stat`，常规文件管理全覆盖。

值得一提的是 `list_dir` 对隐藏文件的处理：

```rust
if name.starts_with('.') { continue; }
```

虽然简单，但避免了远端 `.` 和 `..` 特殊目录在前端显示出来造成导航混乱。不过 `.gitignore`、`.env` 等有意义的前缀点文件也会被跳过，如果需要管理隐藏文件，这里可以加个开关。

### Rust 后端：数据持久化（src-tauri/src/store.rs）

主机信息存在 SQLite 里，`Store` 结构体用 `Mutex<Connection>` 保证线程安全：

```rust
pub struct Store {
    conn: Mutex<Connection>,
}
```

数据库文件放在系统数据目录（`dirs::data_dir()/sshterm/hosts.db`），不需要用户操心路径。只有一张表 `hosts`，字段清晰：

```
id, name, hostname, port, username, auth_type,
password, key_path, key_passphrase,
group_name, color, notes,
created_at, updated_at
```

支持分组和颜色标记，前端渲染时按组聚合，左侧看起来井井有条。结构上看，后面如果要加标签、收藏、最近连接等特性，直接加字段就行，扩展性不错。

### Rust 后端：命令注册（src-tauri/src/main.rs）

Tauri 2 的命令注册方式：

```rust
fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_shell::init())
        .manage(AppState {
            store,
            active_sessions: Mutex::new(HashMap::new()),
        })
        .invoke_handler(tauri::generate_handler![
            list_hosts, get_groups, add_host, update_host, delete_host,
            ssh_connect, ssh_disconnect, ssh_exec,
            sftp_list_dir, sftp_read_file, sftp_write_file,
            sftp_create_dir, sftp_remove, sftp_rename, sftp_stat,
        ])
        .run(tauri::generate_context!())
        .expect("Error while running SSHTerm");
}
```

一共 16 个命令，分成三组：主机 CRUD、SSH 操作、SFTP 文件操作。每个命令都是纯函数式的签名——入参出参靠 serde 序列化，前端调用时像调本地 async 函数一样自然。

`active_sessions: Mutex<HashMap<String, SshSession>>` 这个设计值得说一下：每次 SSH 连接都分配一个 UUID，前端和后端通过 sessionId 关联，保证了多标签页场景下会话互不干扰。

### 前端：仅原生 JS，不到 400 行

前端没有用 React、Vue 或 Svelte，就是**原生 HTML + CSS + JavaScript**。这在 Tauri 项目里其实是很合理的选择——不需要虚拟 DOM，不需要组件化框架，100% 控制渲染性能。

xterm.js 的配置值得一看，Tokyo Night 配色方案是通过 `theme` 字段逐一配置的：

```javascript
theme: {
    background: '#000000',
    foreground: '#c0caf5',
    cursor: '#7aa2f7',
    red: '#f7768e',
    green: '#9ece6a',
    // ... 完整的 Tokyo Night palette
}
```

还用到了三个 xterm addon：
- **FitAddon**：自动适应容器大小
- **WebglAddon**：GPU 加速渲染（降级到 canvas）
- **SearchAddon**：终端内搜索

配色用的是 **Tokyo Night** 风格，长时间看 SSH 终端眼睛不容易疲劳。

### 前端 SFTP 文件管理

`getFileIcon()` 函数根据文件扩展名返回对应的 emoji 图标，体验做得挺用心：

```javascript
const icons = {
    pdf: '📕', zip: '📦', mp3: '🎵', mp4: '🎬',
    jpg: '🖼', txt: '📄', md: '📄', json: '📋',
    js: '⚙️', py: '⚙️', rs: '⚙️',
    html: '🌐', css: '🎨',
};
```

双栏文件浏览器布局，左侧远程、右侧本地，中间放上传下载按钮。标准的 SFTP 客户端交互模式，没有画蛇添足。

## 值得改进的地方（作为一个 v0.1.0）

看完源码，有几个可以继续优化的点：

1. **实时终端流**：目前用的是 `ssh_exec` 命令执行模式，一次执行完返回结果。真正的交互式 shell 需要 Tauri events 或 WebSocket 桥来推送 SSH Channel 的输出流。代码里已经有 WebSocket 变量和 resize 处理逻辑的框架，但流式终端还没完全接上。

2. **密码存储**：密码和密钥 passphrase 目前明文存在 SQLite 里。虽然本地应用不像服务器那么敏感，但加上系统密钥链集成（macOS Keychain、Linux Secret Service、Windows Credential Manager）会更让人放心。

3. **本地 SFTP 文件浏览**：目前本地文件面板只有基础框架，真正的本地文件系统浏览还没实现。`state.sftp.localPath` 硬编码为 `/home`。

4. **SSH 密钥生成**：没有集成密钥对生成功能，新用户可能需要先用 `ssh-keygen`。

5. **会话恢复**：重启应用后不会自动恢复上次的连接和标签页状态。加个最近会话历史应该能让体验连贯不少。

## 构建体验

项目文档里的构建步骤很直接：

```bash
cargo install tauri-cli --version "^2.0"
npm install
cargo tauri dev    # 开发模式
cargo tauri build  # 发布构建
```

GitHub Actions CI 配置已经写好，推送后自动编译 Linux .deb/.AppImage 和 Windows .msi。

## 一点想法

SSHTerm 这种项目恰恰是 **Tauri 最擅长的场景**：一个清晰的工具类应用，不需要复杂的前端交互，但需要接近原生的网络性能和系统集成。

对比 Electron 方案：

- **体积**：5MB vs 200MB+
- **内存**：Rust 的 SSH 栈 vs 整个 Chromium
- **启动速度**：瞬开 vs 加载 V8 引擎

而开发体验上，Tauri 2 的 `invoke` 机制让前后端通信像调本地函数一样直接，不需要写 REST API 或 RPC 协议。Rust 后端直接调用 libssh2，出错路径清晰，SSH 超时、认证失败都能精确控制。

如果你也在考虑做一个命令行工具的桌面壳子、服务器管理面板或者运维工具，用 Tauri 2 + Rust 后端 + 原生 Web 前端这套组合，值得认真考虑。

[SSHTerm GitHub 仓库 →](https://github.com/jxdxs/sshterm)
