# OpenClaw 多实例配置实战教程

_从零开始配置三个独立运行的 OpenClaw 实例，实现飞书多 Bot 协作_

![Version](https://img.shields.io/badge/version-1.4-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.3.13-orange)
![Platform](https://img.shields.io/badge/platform-Linux-lightgrey)

**版本**: 1.4（最新版本）  
**最后更新**: 2026-03-19  
**OpenClaw 版本**: 2026.3.13  
**作者**: OpenClaw 团队（🦞 Main + 💻 Coder + ✍️ Writer + 🎨 Artist）

---

## 📖 目录

### 第一部分：基础篇
- [第 1 章：认识 OpenClaw](#第 1 章认识 openclaw)
- [第 2 章：环境准备](#第 2 章环境准备)

### 第二部分：单实例配置
- [第 3 章：创建第一个实例（Main）](#第 3 章创建第一个实例 main)

### 第三部分：多实例配置（Profile 模式）
- [第 4 章：Agent 注册机制](#第 4 章 agent 注册机制)
- [第 5 章：Default 实例的特殊地位](#第 5 章 default 实例的特殊地位)
- [第 6 章：Profile 机制详解](#第 6 章 profile 机制详解)
- [第 7 章：创建 Coder 实例](#第 7 章创建 coder 实例)
- [第 8 章：创建 Writer 实例](#第 8 章创建 writer 实例)
- [第 9 章：创建 Artist 实例](#第 9 章创建 artist 实例)

### 第四部分：架构模式对比 ⭐
- [第 10 章：Profile 实例 vs Agent 模式](#第 10 章 profile 实例 vs agent 模式)

### 第五部分：认证与同步
- [第 11 章：认证配置](#第 11 章认证配置)
- [第 12 章：常见问题排查](#第 12 章常见问题排查)

### 第六部分：实例间通信
- [第 13 章：通信机制](#第 13 章通信机制)
- [第 14 章：任务下达方式](#第 14 章任务下达方式)
- [第 15 章：状态监控](#第 15 章状态监控)

### 第七部分：飞书集成
- [第 16 章：飞书应用配置](#第 16 章飞书应用配置)
- [第 17 章：飞书路由绑定](#第 17 章飞书路由绑定)
- [第 18 章：配对授权流程](#第 18 章配对授权流程)

### 第八部分：运维与管理
- [第 19 章：日常运维](#第 19 章日常运维)
- [第 20 章：配置验证清单](#第 20 章配置验证清单)

### 第九部分：实战案例
- [第 21 章：ClawClock 项目协作](#第 21 章 clawclock 项目协作)
- [第 22 章：配置错误修复实录](#第 22 章配置错误修复实录)

### 第十部分：附录
- [附录 A：命令速查表](#附录 a 命令速查表)
- [附录 B：配置文件模板](#附录 b 配置文件模板)
- [附录 C：故障排除](#附录 c 故障排除)
- [附录 D：版本历史](#附录 d 版本历史)

---

## 第一部分：基础篇

### 第 1 章：认识 OpenClaw

#### 1.1 OpenClaw 是什么

**OpenClaw** 是一个开源的 AI 助手框架，支持：

- 🤖 **多模型接入** - 阿里云百炼、智谱、月之暗面等
- 💬 **多通道通信** - 飞书、Telegram、WhatsApp、Discord 等
- 🔄 **多实例运行** - 同一台机器并行运行多个独立实例
- 🧠 **持久化记忆** - 支持长期记忆和每日日志
- 🛠️ **可扩展技能** - 通过技能系统扩展功能

**定位**：个人/团队的 AI 助手基础设施

**适用场景**：
- 个人 AI 助手
- 客服机器人
- 开发助手
- 写作助手
- 设计助手
- 多角色协作系统

---

#### 1.2 核心概念

| 概念 | 说明 | 示例 |
|------|------|------|
| **实例 (Instance)** | 独立运行的 OpenClaw 进程 | Main、Coder、Writer、Artist |
| **Agent** | 实例内的 AI 代理 | main、coder、writer、artist |
| **工作空间 (Workspace)** | Agent 的文件目录 | `~/.openclaw/workspace/` |
| **通道 (Channel)** | 通信渠道 | feishu、telegram |
| **Profile** | 实例配置标识 | `--profile coder` |
| **会话 (Session)** | 对话历史记录 | `*.jsonl` 文件 |
| **路由 (Binding)** | 消息分发规则 | `feishu:main-bot → main` |

---

#### 1.3 架构概览

**单实例模式**：
```
用户 → 飞书 Bot → Main 实例 → 回复
```

**多实例模式**（本教程）：
```
                    ┌─────────────┐
                    │   飞书用户   │
                    └──────┬──────┘
                           │
         ┌─────────────────┼──────────────────┐
         │                 │                 │
   ┌─────▼─────┐   ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
   │ main-bot  │   │ coder-bot   │   │ writer-bot  │   │ artist-bot  │
   │ 🦞        │   │  💻         │   │  ✍️         │   │  🎨         │
   └─────┬─────┘   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
         │                │                 │                 │
   ┌─────▼─────┐   ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
   │  Main     │   │   Coder     │   │   Writer    │   │   Artist    │
   │  18789    │   │   18989     │   │   18689     │   │   19889     │
   └───────────┘   └─────────────┘   └─────────────┘   └─────────────┘
```

---

### 第 2 章：环境准备

#### 2.1 系统要求

**操作系统**：
- ✅ Linux（Debian/Ubuntu/Arch/Fedora）
- ⚠️ macOS（部分功能受限）
- ❌ Windows（需 WSL2）

**软件版本**：
- Node.js ≥ 18.x
- npm ≥ 9.x
- Python 3.8+（可选，用于技能）

**硬件要求**：
- CPU：2 核心以上
- 内存：2GB 以上（每实例约 500MB）
- 磁盘：10GB 可用空间

---

#### 2.2 安装 OpenClaw

```bash
# 安装 OpenClaw
npm install -g openclaw

# 验证安装
openclaw --version

# 查看帮助
openclaw --help
```

**输出示例**：
```
OpenClaw CLI version 2026.3.13
Usage: openclaw <command> [options]

Commands:
  gateway      启动网关服务
  agents       管理 Agents
  models       管理模型
  pairing      配对授权
```

---

#### 2.3 飞书应用创建

**步骤 1：登录飞书开放平台**
- 访问 https://open.feishu.cn
- 使用企业账号登录

**步骤 2：创建应用**
1. 点击「创建应用」
2. 选择「企业自建应用」
3. 填写应用名称（如：OpenClaw-Main）
4. 点击「创建」

**步骤 3：获取凭证**
- 进入应用管理页面
- 记录 **App ID**（格式：`cli_xxxxxxxxxxxxxxxx`）
- 点击「查看 Secret」，记录 **App Secret**

**步骤 4：配置机器人**
1. 点击「添加机器人」
2. 配置机器人名称和头像
3. 启用「消息接收」权限

**步骤 5：重复创建 4 个应用**
| 应用名称 | 用途 | 端口 |
|---------|------|------|
| OpenClaw-Main | 通用助手 | 18789 |
| OpenClaw-Coder | 编码专家 | 18989 |
| OpenClaw-Writer | 写作专家 | 18689 |
| OpenClaw-Artist | 设计专家 | 19889 |

---

#### 2.4 准备工作目录

```bash
# 创建基础目录
mkdir -p ~/.openclaw/workspace/{memory,config,skills}

# 设置权限
chmod 700 ~/.openclaw
```

---

## 第二部分：单实例配置

### 第 3 章：创建第一个实例（Main）

#### 3.1 初始化配置

```bash
# 进入工作目录
cd ~/.openclaw/workspace

# 初始化（可选）
openclaw init
```

**初始化会创建以下文件**：
- `SOUL.md` - 人格定义
- `IDENTITY.md` - 身份元数据
- `USER.md` - 用户信息
- `AGENTS.md` - 行为准则
- `MEMORY.md` - 长期记忆

---

#### 3.2 配置文件结构

```
~/.openclaw/
├── openclaw.json              # 全局配置 ⭐
├── identity/
│   └── device.json            # 设备身份
├── credentials/
│   ├── feishu-pairing.json    # 飞书配对
│   └── feishu-default-allowFrom.json  # 授权列表
├── agents/
│   └── main/
│       ├── agent/
│       │   ├── auth.json
│       │   ├── auth-profiles.json
│       │   └── models.json
│       └── sessions/          # 会话历史
└── workspace/                 # 工作空间
    ├── SOUL.md
    ├── IDENTITY.md
    ├── USER.md
    ├── AGENTS.md
    ├── MEMORY.md
    ├── memory/                # 每日日志
    └── config/                # 扩展配置
```

---

#### 3.3 配置飞书通道

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_YOUR_MAIN_APP_ID",
      "appSecret": "YOUR_MAIN_APP_SECRET",
      "connectionMode": "websocket",
      "domain": "feishu",
      "groupPolicy": "open"
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "YOUR_MAIN_GATEWAY_TOKEN"
    }
  }
}
```

**关键字段说明**：
- `appId`：飞书应用 ID
- `appSecret`：飞书应用密钥
- `connectionMode`：连接模式（websocket 或 http）
- `gateway.auth.token`：网关访问令牌

---

#### 3.4 启动服务

**方式 A：直接启动**
```bash
openclaw gateway
```

**方式 B：使用 systemd（推荐）**

创建服务文件 `~/.config/systemd/user/openclaw-gateway.service`：

```ini
[Unit]
Description=OpenClaw Gateway - Main Instance
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/linuxbrew/.linuxbrew/bin/node \
  "/home/linuxbrew/.linuxbrew/lib/node_modules/openclaw/dist/index.js" \
  gateway --port 18789
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/lenovo
Environment=OPENCLAW_GATEWAY_PORT=18789
Environment=OPENCLAW_GATEWAY_TOKEN=YOUR_MAIN_GATEWAY_TOKEN
Environment=OPENCLAW_SERVICE_MARKER=openclaw
Environment=OPENCLAW_SERVICE_KIND=gateway

[Install]
WantedBy=default.target
```

**启动命令**：
```bash
# 重新加载 systemd 配置
systemctl --user daemon-reload

# 启动服务
systemctl --user start openclaw-gateway.service

# 查看状态
systemctl --user status openclaw-gateway.service

# 启用开机自启
systemctl --user enable openclaw-gateway.service
```

---

#### 3.5 飞书配对授权

**步骤 1：发送测试消息**
- 在飞书中给 main-bot 发送任意消息

**步骤 2：获取配对码**
Bot 会回复：
```
OpenClaw: access not configured.
Your Feishu user id: ou_YOUR_FEISHU_USER_ID
Pairing code: U7Y3A3FB
Ask the bot owner to approve with:
openclaw pairing approve feishu U7Y3A3FB
```

**步骤 3：执行授权**
```bash
openclaw pairing approve feishu U7Y3A3FB
```

**输出**：
```
Approved feishu sender ou_YOUR_FEISHU_USER_ID.
```

---

#### 3.6 验证运行

```bash
# 检查服务状态
systemctl --user status openclaw-gateway.service

# 检查端口监听
ss -tlnp | grep 18789

# 健康检查
curl http://localhost:18789/health

# 查看日志
journalctl --user -u openclaw-gateway.service --since "5 minutes ago"
```

**预期输出**：
```
● openclaw-gateway.service - OpenClaw Gateway - Main Instance
     Active: active (running)
     Main PID: 102307
     Memory: 1.3G
```

---

## 第三部分：多实例配置

### 第 4 章：Agent 注册机制

#### 4.1 什么是 Agent 注册

**Agent 注册**是将一个新实例添加到 OpenClaw 系统的过程。执行 `openclaw agents add <name>` 后，系统会自动完成以下配置：

---

#### 4.2 注册过程详解

**执行命令**：
```bash
openclaw agents add artist --workspace ~/.openclaw/workspace-artist
```

**系统自动完成的操作**：

1. **创建 Agent 目录**
```bash
~/.openclaw/agents/artist/
├── agent/          # Agent 配置文件
│   ├── auth.json
│   ├── auth-profiles.json
│   └── models.json
└── sessions/       # 会话历史目录
```

2. **复制认证配置**
- 从 `~/.openclaw/agents/main/agent/` 复制：
  - `auth.json` - 基础认证信息
  - `auth-profiles.json` - OAuth Token
  - `models.json` - 模型配置

3. **更新全局配置**
- 编辑 `~/.openclaw/openclaw.json`
- 在 `agents.list` 中添加新 Agent：
```json
{
  "id": "artist",
  "workspace": "/home/lenovo/.openclaw/workspace-artist"
}
```

4. **设置身份配置**
```bash
openclaw agents set-identity --agent artist --from-identity
```
- 读取 `~/.openclaw/workspace-artist/IDENTITY.md`
- 配置 Agent 的名称、表情符号、主题

---

#### 4.3 注册后更改的文件清单

| 文件路径 | 更改内容 | 作用 |
|---------|---------|------|
| `~/.openclaw/openclaw.json` | 添加 `agents.list` 条目 | 注册新 Agent |
| `~/.openclaw/agents/artist/agent/auth.json` | 新建（从 main 复制） | 基础认证 |
| `~/.openclaw/agents/artist/agent/auth-profiles.json` | 新建（从 main 复制） | OAuth Token |
| `~/.openclaw/agents/artist/agent/models.json` | 新建（从 main 复制） | 模型配置 |
| `~/.openclaw/agents/artist/sessions/` | 创建目录 | 会话存储 |

---

#### 4.4 重要说明

**目录结构区别**：
- **Agent 注册目录**：`~/.openclaw/agents/artist/` - 路由配置
- **Profile 状态目录**：`~/.openclaw-artist/` - 运行时状态
- **工作空间**：`~/.openclaw/workspace-artist/` - 工作文件

**认证同步**：
- 注册时自动从 Main 实例复制认证配置
- 如后续更新认证，需手动同步到各实例

---

### 第 5 章：Default 实例的特殊地位

#### 9.1 什么是 Default 实例

**Default 实例**（默认实例）是第一个创建的 OpenClaw 实例，也是整个多实例架构的核心。

**识别方法**：
- 启动命令不使用 `--profile` 参数
- 状态目录：`~/.openclaw/`
- 工作空间：`~/.openclaw/workspace/`

---

#### 9.2 Default 实例的特殊地位

##### 1. 中央管理角色

**Default 实例是协调者**：
- 管理所有路由规则（`bindings`）
- 存储完整的 Agent 列表（`agents.list`）
- 可以向其他实例下达指令
- 监控所有实例状态

**配置示例**（`~/.openclaw/openclaw.json`）：
```json
{
  "agents": {
    "list": [
      { "id": "main", "workspace": "/home/lenovo/.openclaw/workspace" },
      { "id": "coder", "workspace": "/home/lenovo/.openclaw/workspace-coder" },
      { "id": "writer", "workspace": "/home/lenovo/.openclaw/workspace-writer" },
      { "id": "artist", "workspace": "/home/lenovo/.openclaw/workspace-artist" }
    ]
  },
  "bindings": [
    { "agentId": "main", "match": { "accountId": "main-bot" } },
    { "type": "route", "agentId": "coder", "match": { "accountId": "coder-bot" } },
    { "type": "route", "agentId": "writer", "match": { "accountId": "writer-bot" } },
    { "type": "route", "agentId": "artist", "match": { "accountId": "artist-bot" } }
  ]
}
```

---

##### 2. 路由规则存储

**只有 Default 实例存储完整路由**：
- Main → main-bot
- Coder → coder-bot
- Writer → writer-bot
- Artist → artist-bot

**Profile 实例只存储自己的路由**：
```json
// ~/.openclaw-coder/openclaw.json
{
  "bindings": [
    {
      "agentId": "main",
      "match": { "accountId": "coder-bot" }  // 只绑定自己
    }
  ]
}
```

---

##### 3. 跨实例访问权限

**只有 Default 实例需要配置**：
```json
{
  "tools": {
    "sessions": {
      "visibility": "all"  // 可以访问所有实例的会话
    }
  }
}
```

**Profile 实例不需要**：
- 它们只执行任务
- 不需要访问其他实例
- 遵循最小权限原则

---

#### 9.3 Default 实例 vs Profile 实例

| 特性 | Default 实例 | Profile 实例 |
|------|------------|-------------|
| **启动命令** | `openclaw gateway` | `openclaw --profile <name> gateway` |
| **状态目录** | `~/.openclaw/` | `~/.openclaw-<name>/` |
| **工作空间** | `~/.openclaw/workspace/` | `~/.openclaw/workspace-<name>/` |
| **agents.list** | 完整（所有实例） | 简化（只有自己） |
| **bindings** | 多个（所有路由） | 单一（只绑定自己） |
| **visibility** | `all`（需要） | 无（不需要） |
| **角色** | 协调者/管理者 | 任务执行者 |

---

#### 9.4 配置文件对比

**Default 实例配置**（`~/.openclaw/openclaw.json`）：
```json
{
  "agents": {
    "list": [
      { "id": "main", ... },
      { "id": "coder", ... },
      { "id": "writer", ... },
      { "id": "artist", ... }
    ]
  },
  "bindings": [
    { "agentId": "main", ... },
    { "type": "route", "agentId": "coder", ... },
    { "type": "route", "agentId": "writer", ... },
    { "type": "route", "agentId": "artist", ... }
  ],
  "tools": {
    "sessions": { "visibility": "all" }
  }
}
```

**Profile 实例配置**（`~/.openclaw-coder/openclaw.json`）：
```json
{
  "agents": {
    "list": [
      { "id": "main", "workspace": "/home/lenovo/.openclaw-coder/workspace" }
    ]
  },
  "bindings": [
    { "agentId": "main", "match": { "accountId": "coder-bot" } }
  ]
  // 没有 tools.sessions.visibility 配置
}
```

---

#### 9.5 实例间关系

```
┌──────────────────────────────────────────────────────────────┐
│  Default 实例 (🦞 Main) - 协调者/管理者                       │
│  ~/.openclaw/                                                │
│  - 存储所有路由规则                                           │
│  - 完整的 agents.list                                        │
│  - visibility=all（可访问所有实例）                            │
│  - 可以向其他实例下达指令                                      │
└─────────────────────┬────────────────────────────────────────┘
                      │ 单向访问/管理
         ┌────────────┼──────────────┐
         │            │              │
         ▼            ▼              ▼
┌────────────┐  ┌────────────┐  ┌────────────┐
│   Coder    │  │   Writer   │  │   Artist   │
│  执行者    │  │  执行者    │  │  执行者    │
│  ~/.openclaw-coder/       │  │  ~/.openclaw-artist/       │
│  只绑定 coder-bot         │  │  只绑定 artist-bot         │
└────────────┘  └────────────┘  └────────────┘
```

**指挥链**：
```
陛下 → Default 实例（副手）→ Profile 实例（Coder/Writer/Artist）
```

---

#### 9.6 重要原则

1. **只有一个 Default 实例** - 第一个创建的实例
2. **所有新实例都注册到 Default** - 使用 `openclaw agents add`
3. **Default 管理路由** - 所有路由规则存储在 Default
4. **Profile 实例独立运行** - 有独立的状态目录和工作空间
5. **认证需要手动同步** - 从 Default 同步到各 Profile 实例

---

### 第 6 章：Profile 机制详解

#### 8.1 什么是 Profile

**Profile** 是 OpenClaw 的多实例隔离机制，允许：
- 同一台机器运行多个独立实例
- 每个实例有独立的状态目录、工作空间、配置
- 通过 `--profile <name>` 参数区分

**命名规则**：
- 默认实例：不使用 `--profile`
- Profile 实例：`--profile coder`、`--profile writer`、`--profile artist`

---

#### 4.2 目录结构对比

| 实例类型 | 状态目录 | 工作空间 |
|---------|---------|---------|
| **默认** | `~/.openclaw/` | `~/.openclaw/workspace/` |
| **Profile** | `~/.openclaw-<name>/` | `~/.openclaw/workspace-<name>/` |

**示例**：
```
# Main（默认）
~/.openclaw/
├── openclaw.json
└── workspace/

# Coder（--profile coder）
~/.openclaw-coder/
├── openclaw.json
└── workspace-coder/

# Writer（--profile writer）
~/.openclaw-writer/
├── openclaw.json
└── workspace-writer/

# Artist（--profile artist）
~/.openclaw-artist/
├── openclaw.json
└── workspace-artist/
```

---

#### 4.3 启动命令对比

```bash
# 默认实例（Main）
openclaw gateway --port 18789

# Coder 实例
openclaw --profile coder gateway --port 18989

# Writer 实例
openclaw --profile writer gateway --port 18689

# Artist 实例
openclaw --profile artist gateway --port 19889
```

**环境变量**：
```bash
# Main
OPENCLAW_STATE_DIR=~/.openclaw/
OPENCLAW_WORKSPACE=~/.openclaw/workspace/

# Coder
OPENCLAW_PROFILE=coder
OPENCLAW_STATE_DIR=~/.openclaw-coder/
OPENCLAW_WORKSPACE=~/.openclaw/workspace-coder/

# Writer
OPENCLAW_PROFILE=writer
OPENCLAW_STATE_DIR=~/.openclaw-writer/
OPENCLAW_WORKSPACE=~/.openclaw/workspace-writer/

# Artist
OPENCLAW_PROFILE=artist
OPENCLAW_STATE_DIR=~/.openclaw-artist/
OPENCLAW_WORKSPACE=~/.openclaw/workspace-artist/
```

---

### 第 7 章：创建 Coder 实例

#### 9.1 创建工作空间

```bash
# 创建独立的工作空间目录
mkdir -p ~/.openclaw/workspace-coder/{memory,config,skills}

# 复制基础配置文件
cp ~/.openclaw/workspace/{AGENTS,SOUL,USER}.md ~/.openclaw/workspace-coder/
```

---

#### 9.2 创建身份文件

创建 `~/.openclaw/workspace-coder/IDENTITY.md`：

```markdown
# IDENTITY.md - Who Am I?

- **Name:** OpenClaw Coder
- **Creature:** AI coding specialist
- **Vibe:** Technical, precise, code-focused
- **Emoji:** 💻
- **Avatar:** (same as main)

---

This is the coding specialist instance, focused on development tasks.
```

创建 `~/.openclaw/workspace-coder/SOUL.md`：

```markdown
# SOUL.md - Coder Instance

_You are the coding specialist of the OpenClaw family._

## Core Identity

- **Focus**: Code, development, debugging, architecture
- **Style**: Precise, technical, efficient
- **Tools**: Full access to all coding agents

## Behavior

- Jump straight into code - no filler
- Prefer solutions that are maintainable and well-documented
- When in doubt, show code examples
- Test before committing

## Boundaries

- Don't modify production systems without explicit confirmation
- Keep a backup before major refactors
- Respect the main instance's workspace

---

_You are the builder. Make things work._
```

---

#### 9.3 注册 Agent

```bash
# 创建 agent 目录
mkdir -p ~/.openclaw/agents/coder/{agent,sessions}

# 复制认证配置
cp ~/.openclaw/agents/main/agent/auth*.json ~/.openclaw/agents/coder/agent/
cp ~/.openclaw/agents/main/agent/models.json ~/.openclaw/agents/coder/agent/

# 注册 Agent
openclaw agents add coder --workspace ~/.openclaw/workspace-coder

# 设置身份
openclaw agents set-identity --agent coder --from-identity --workspace ~/.openclaw/workspace-coder
```

---

#### 9.4 配置 systemd 服务

创建 `~/.config/systemd/user/openclaw-coder.service`：

```ini
[Unit]
Description=OpenClaw Gateway - Coder Instance (v2026.3.13)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/linuxbrew/.linuxbrew/bin/node \
  "/home/linuxbrew/.linuxbrew/lib/node_modules/openclaw/dist/index.js" \
  --profile coder gateway --port 18989
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/lenovo
Environment=OPENCLAW_PROFILE=coder
Environment=OPENCLAW_STATE_DIR=/home/lenovo/.openclaw-coder
Environment=OPENCLAW_WORKSPACE=/home/lenovo/.openclaw/workspace-coder
Environment=OPENCLAW_GATEWAY_PORT=18989
Environment=OPENCLAW_GATEWAY_TOKEN=YOUR_CODER_GATEWAY_TOKEN
Environment=OPENCLAW_SERVICE_MARKER=openclaw-coder
Environment=OPENCLAW_SERVICE_KIND=gateway
Environment=OPENCLAW_SERVICE_VERSION=2026.3.13

[Install]
WantedBy=default.target
```

**关键环境变量**：
- `OPENCLAW_PROFILE=coder` - Profile 名称
- `OPENCLAW_STATE_DIR` - 独立状态目录
- `OPENCLAW_WORKSPACE` - 独立工作空间
- `OPENCLAW_GATEWAY_TOKEN` - 独立访问令牌

---

#### 9.5 绑定飞书路由

```bash
# 将 coder-bot 账号绑定到 Coder 实例
openclaw agents bind --agent coder --bind feishu:coder-bot

# 验证路由配置
openclaw agents bindings
```

**预期输出**：
```
Routing bindings:
- main <- feishu accountId=main-bot
- coder <- feishu accountId=coder-bot
```

---

#### 9.6 启动并验证

```bash
# 重新加载 systemd 配置
systemctl --user daemon-reload

# 启动 Coder 实例
systemctl --user start openclaw-coder.service

# 查看状态
systemctl --user status openclaw-coder.service

# 验证模型可用性
openclaw --profile coder agent -m "测试" --session-id test-coder-1
```

**预期输出**：
```
● openclaw-coder.service - OpenClaw Gateway - Coder Instance
     Active: active (running)
     Main PID: 131571
     Memory: 752.7M
```

**飞书配对**：
- 在飞书中给 coder-bot 发送消息
- 获取配对码并授权：
```bash
openclaw --profile coder pairing approve feishu <CODE>
```

---

### 第 8 章：创建 Writer 实例

#### 8.1 重复 Coder 的配置流程

```bash
# 创建工作空间
mkdir -p ~/.openclaw/workspace-writer/{memory,config,skills}
cp ~/.openclaw/workspace/{AGENTS,SOUL,USER}.md ~/.openclaw/workspace-writer/

# 创建 agent 目录
mkdir -p ~/.openclaw/agents/writer/{agent,sessions}
cp ~/.openclaw/agents/main/agent/auth*.json ~/.openclaw/agents/writer/agent/

# 注册 Agent
openclaw agents add writer --workspace ~/.openclaw/workspace-writer
openclaw agents set-identity --agent writer --from-identity --workspace ~/.openclaw/workspace-writer
```

---

#### 8.2 身份定位

创建 `~/.openclaw/workspace-writer/IDENTITY.md`：

```markdown
# IDENTITY.md - Who Am I?

- **Name:** OpenClaw Writer
- **Creature:** AI writing specialist
- **Vibe:** Creative, articulate, expressive
- **Emoji:** ✍️
- **Avatar:** (same as main)

---

This is the writing specialist instance, focused on content creation and editing.
```

---

#### 8.3 配置 systemd 服务

创建 `~/.config/systemd/user/openclaw-writer.service`：

```ini
[Unit]
Description=OpenClaw Gateway - Writer Instance (v2026.3.13)
After=network-online.target

[Service]
ExecStart=/home/linuxbrew/.linuxbrew/bin/node \
  "/home/linuxbrew/.linuxbrew/lib/node_modules/openclaw/dist/index.js" \
  --profile writer gateway --port 18689
Restart=always
Environment=OPENCLAW_PROFILE=writer
Environment=OPENCLAW_STATE_DIR=/home/lenovo/.openclaw-writer
Environment=OPENCLAW_WORKSPACE=/home/lenovo/.openclaw/workspace-writer
Environment=OPENCLAW_GATEWAY_PORT=18689
Environment=OPENCLAW_GATEWAY_TOKEN=YOUR_WRITER_GATEWAY_TOKEN
Environment=OPENCLAW_SERVICE_MARKER=openclaw-writer

[Install]
WantedBy=default.target
```

---

#### 8.4 配置检查清单

启动前务必检查：

- [ ] 工作空间路径：`~/.openclaw/workspace-writer/`
- [ ] 端口：18689（不与 Main/Coder 冲突）
- [ ] 飞书 App ID：writer-bot 的 App ID
- [ ] 飞书 App Secret：writer-bot 的 Secret
- [ ] Gateway Token：独立的 token
- [ ] 路由绑定：`openclaw agents bind --agent writer --bind feishu:writer-bot`

---

### 第 9 章：创建 Artist 实例

#### 9.1 创建工作空间

```bash
# 创建独立的工作空间目录
mkdir -p ~/.openclaw/workspace-artist/{memory,config,skills}

# 复制基础配置文件
cp ~/.openclaw/workspace/{AGENTS,SOUL,USER}.md ~/.openclaw/workspace-artist/
```

---

#### 9.2 创建身份文件

创建 `~/.openclaw/workspace-artist/IDENTITY.md`：

```markdown
# IDENTITY.md - Who Am I?

- **Name:** OpenClaw Artist
- **Creature:** AI design specialist
- **Vibe:** Creative, visual, aesthetic-focused
- **Emoji:** 🎨
- **Avatar:** (same as main)

---

This is the design specialist instance, focused on UI/UX design, visual aesthetics, and creative work.
```

创建 `~/.openclaw/workspace-artist/SOUL.md`：

```markdown
# SOUL.md - Artist Instance

_You are the design specialist of the OpenClaw family._

## Core Identity

- **Focus**: UI/UX design, visual aesthetics, color theory, layout design
- **Style**: Creative, visual, detail-oriented, user-centered
- **Tools**: Design recommendations, color palettes, layout suggestions

## Behavior

- Think visually - describe designs clearly
- Consider user experience in every suggestion
- Provide concrete examples and mockups when possible
- Balance aesthetics with usability

## Boundaries

- Don't make design decisions without user input
- Respect brand guidelines when provided
- Consider accessibility in all designs

## Collaboration

- Work alongside Main, Coder, Writer instances
- Main handles: coordination, general tasks
- Coder handles: implementation, technical details
- Writer handles: documentation, content
- You handle: design, visuals, user experience

---

_You are the artist. Make things beautiful._
```

---

#### 9.3 注册 Agent

```bash
# 创建 agent 目录
mkdir -p ~/.openclaw/agents/artist/{agent,sessions}

# 复制认证配置
cp ~/.openclaw/agents/main/agent/auth*.json ~/.openclaw/agents/artist/agent/
cp ~/.openclaw/agents/main/agent/models.json ~/.openclaw/agents/artist/agent/

# 注册 Agent
openclaw agents add artist --workspace ~/.openclaw/workspace-artist

# 设置身份
openclaw agents set-identity --agent artist --from-identity --workspace ~/.openclaw/workspace-artist
```

---

#### 9.4 配置 systemd 服务

创建 `~/.config/systemd/user/openclaw-artist.service`：

```ini
[Unit]
Description=OpenClaw Gateway - Artist Instance (v2026.3.13)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/linuxbrew/.linuxbrew/bin/node \
  "/home/linuxbrew/.linuxbrew/lib/node_modules/openclaw/dist/index.js" \
  --profile artist gateway --port 19889
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/lenovo
Environment=OPENCLAW_PROFILE=artist
Environment=OPENCLAW_STATE_DIR=/home/lenovo/.openclaw-artist
Environment=OPENCLAW_WORKSPACE=/home/lenovo/.openclaw/workspace-artist
Environment=OPENCLAW_GATEWAY_PORT=19889
Environment=OPENCLAW_GATEWAY_TOKEN=YOUR_ARTIST_GATEWAY_TOKEN
Environment=OPENCLAW_SERVICE_MARKER=openclaw-artist
Environment=OPENCLAW_SERVICE_KIND=gateway
Environment=OPENCLAW_SERVICE_VERSION=2026.3.13

[Install]
WantedBy=default.target
```

---

#### 9.5 创建 openclaw.json

创建 `~/.openclaw-artist/openclaw.json`：

```json
{
  "meta": {
    "lastTouchedVersion": "2026.3.13",
    "lastTouchedAt": "2026-03-18T00:00:00.000Z"
  },
  "models": {
    "mode": "merge",
    "providers": {
      "bailian": {
        "baseUrl": "https://coding.dashscope.aliyuncs.com/v1",
        "apiKey": "sk-sp-YOUR_BAILIAN_API_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3.5-plus",
            "name": "qwen3.5-plus",
            "reasoning": false,
            "input": ["text", "image"],
            "cost": { "input": 0, "output": 0 },
            "contextWindow": 1000000,
            "maxTokens": 65536
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "bailian/qwen3.5-plus"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "/home/lenovo/.openclaw/workspace-artist"
      }
    ]
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "accountId": "artist-bot"
      }
    }
  ],
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_YOUR_ARTIST_APP_ID",
      "appSecret": "YOUR_ARTIST_APP_SECRET",
      "connectionMode": "websocket",
      "domain": "feishu",
      "groupPolicy": "open"
    }
  },
  "gateway": {
    "mode": "local",
    "port": 19889,
    "auth": {
      "mode": "token",
      "token": "YOUR_ARTIST_GATEWAY_TOKEN"
    }
  },
  "plugins": {
    "entries": {
      "feishu": {
        "enabled": true
      }
    }
  }
}
```

**注意**：Profile 实例需要独立的 `openclaw.json` 文件在 `~/.openclaw-artist/` 目录下。

---

#### 9.6 绑定飞书路由

```bash
# 将 artist-bot 账号绑定到 Artist 实例
openclaw agents bind --agent artist --bind feishu:artist-bot

# 验证路由配置
openclaw agents bindings
```

**预期输出**：
```
Routing bindings:
- main <- feishu accountId=main-bot
- coder <- feishu accountId=coder-bot
- writer <- feishu accountId=writer-bot
- artist <- feishu accountId=artist-bot
```

---

#### 9.7 启动并验证

```bash
# 重新加载 systemd 配置
systemctl --user daemon-reload

# 启动 Artist 实例
systemctl --user start openclaw-artist.service

# 查看状态
systemctl --user status openclaw-artist.service

# 验证模型可用性
openclaw --profile artist agent -m "测试" --session-id test-artist-1
```

**预期输出**：
```
● openclaw-artist.service - OpenClaw Gateway - Artist Instance
     Active: active (running)
     Main PID: 195318
     Memory: 173.9M
```

**飞书配对**：
- 在飞书中给 artist-bot 发送消息
- 获取配对码并授权：
```bash
openclaw --profile artist pairing approve feishu <CODE>
```

---
---

## 第四部分：架构模式对比 ⭐

### 第 10 章：Profile 实例 vs Agent 模式

OpenClaw 支持两种多实例/多 Agent 架构模式。理解它们的区别对于选择合适的部署方案至关重要。

---

#### 9.5.1 两种模式概览

| 维度 | **Profile 实例模式** | **Agent 模式** |
|------|---------------------|---------------|
| **定义** | 独立的 Gateway 进程 | 同一 Gateway 下的隔离"大脑" |
| **配置** | 独立的 `openclaw.json` | 共享 Gateway 配置，独立 `agentDir` |
| **端口** | 独立端口（如 18789、18989） | 共享同一端口 |
| **进程** | 独立 systemd 服务 | 同一进程内的逻辑隔离 |
| **资源占用** | 高（完整 Gateway 开销） | 低（仅 workspace 隔离） |
| **启动命令** | `OPENCLAW_PROFILE=work openclaw gateway` | `openclaw agents add coding` |

---

#### 9.5.2 Profile 实例模式（本教程采用）

**启动方式**：
```bash
# 每个实例是独立的 Gateway 进程
systemctl --user start openclaw-coder
systemctl --user start openclaw-writer
systemctl --user start openclaw-artist
```

**特点**：
- ✅ **完全独立** — 每个实例有自己的 `openclaw.json`
- ✅ **独立 Channel 账号** — 可以用不同的飞书机器人、Telegram Bot
- ✅ **独立认证 Token** — 每个 Gateway 有自己的 auth token
- ✅ **独立网络端口** — 18789, 18989, 18689, 19889
- ✅ **可以部署在不同机器上** — 灵活性高
- ❌ **资源开销大** — 每个实例都是一个完整 Gateway（约 600MB 内存）

**目录结构**：
```
/home/lenovo/
├── .openclaw/                    # main 实例
│   ├── openclaw.json             # 独立配置
│   ├── workspace/                # main 工作区
│   └── agents/main/              # main agent
│
├── .openclaw-coder/              # coder 实例
│   ├── openclaw.json             # 独立配置
│   ├── workspace/                # coder 工作区
│   └── agents/main/              # coder 的 main agent
│
├── .openclaw-writer/             # writer 实例
│   ├── openclaw.json             # 独立配置
│   ├── workspace/                # writer 工作区
│   └── agents/main/              # writer 的 main agent
│
└── .openclaw-artist/             # artist 实例
    ├── openclaw.json             # 独立配置
    └── workspace-artist/         # artist 工作区
```

**systemd 服务配置**：
```ini
# openclaw-coder.service
Environment=OPENCLAW_PROFILE=coder
Environment=OPENCLAW_STATE_DIR=/home/lenovo/.openclaw-coder
Environment=OPENCLAW_WORKSPACE=/home/lenovo/.openclaw/workspace-coder
Environment=OPENCLAW_GATEWAY_PORT=18989
```

**适用场景**：
1. **不同的 Channel 账号** — 个人 WhatsApp vs 公司 WhatsApp
2. **完全隔离的需求** — 不同信任级别的任务
3. **多机器部署** — 家里一台、公司一台、VPS 一台
4. **测试/开发环境** — 不影响生产配置

---

#### 9.5.3 Agent 模式

**启动方式**：
```bash
# 在同一 Gateway 下添加 Agent
openclaw agents add coder
openclaw agents add writer
openclaw agents add artist
```

**特点**：
- ✅ **共享 Gateway** — 一个进程服务所有 Agent
- ✅ **共享 Channel 账号** — 通过 bindings 路由到不同 Agent
- ✅ **统一端口** — 只用一个端口（如 18789）
- ✅ **资源效率高** — 一个 Gateway (~600MB) + 多个 Agent (每个 ~50MB)
- ✅ **管理简单** — 统一配置、统一监控
- ❌ **需要配置 bindings** — 消息路由更复杂
- ❌ **共享 Channel 账号** — 默认所有 Agent 用同一个飞书机器人

**目录结构**：
```
/home/lenovo/.openclaw/
├── openclaw.json                 # 单一配置，包含所有 Agent
├── workspace/                    # main 工作区
└── agents/
    ├── main/                     # main agent
    ├── coder/                    # coder agent
    ├── writer/                   # writer agent
    └── artist/                   # artist agent
```

**配置示例**（`openclaw.json`）：
```json
{
  "agents": {
    "list": [
      { id: "coder", workspace: "~/.openclaw/workspace-coder" },
      { id: "writer", workspace: "~/.openclaw/workspace-writer" },
    ],
  },
  "bindings": [
    {
      agentId: "coder",
      match: { channel: "feishu", peer: { kind: "direct", id: "+852111" } },
    },
    {
      agentId: "writer",
      match: { channel: "feishu", peer: { kind: "direct", id: "+852222" } },
    },
  ],
}
```

**适用场景**：
1. **多人格/多角色** — 编程助手、写作助手、艺术助手
2. **多用户共享 Gateway** — Alice、Bob、Charlie 共用一个 Gateway
3. **按任务类型路由** — dev、ops、support
4. **资源受限环境** — 节省内存和 CPU

---

#### 9.5.4 详细对比表

| 特性 | Profile 实例 | Agent |
|------|-------------|-------|
| **隔离级别** | 进程级（最高） | 逻辑级（workspace 隔离） |
| **配置独立性** | 完全独立 | 部分独立 |
| **Channel 账号** | 可不同账号 | 默认共享（可配置多账号） |
| **内存占用** | ~600MB/实例 | ~50MB/Agent |
| **启动时间** | ~5 秒 | 即时 |
| **消息路由** | 端口分离 | bindings 路由 |
| **会话隔离** | 完全隔离 | 完全隔离 |
| **技能安装** | 独立 `skills/` | 独立 `skills/` + 共享 `~/.openclaw/skills` |
| **认证配置** | 独立 | 独立（`auth-profiles.json`） |
| **适合场景** | 多环境/多机器 | 多人格/多用户 |

---

#### 9.5.5 陛下的实际架构（4 个 Profile 实例）

根据系统进程检查，当前运行了 **4 个独立的 Gateway 实例**：

| 实例名 | 端口 | 配置目录 | Workspace | Feishu 账号 | systemd 服务 |
|--------|------|----------|-----------|-------------|--------------|
| **main** (我) | 18789 | `~/.openclaw/` | `~/.openclaw/workspace` | 默认 | `openclaw-gateway.service` |
| **coder** | 18989 | `~/.openclaw-coder/` | `~/.openclaw-coder/workspace` | `coder-bot` | `openclaw-coder.service` |
| **writer** | 18689 | `~/.openclaw-writer/` | `~/.openclaw-writer/workspace` | `writer-bot` | `openclaw-writer.service` |
| **artist** | 19889 | `~/.openclaw-artist/` | `~/.openclaw/workspace-artist` | `artist-bot` | `openclaw-artist.service` |

**资源占用**：
- 总内存：约 2.4GB (4 × 600MB)
- 4 个独立 systemd 服务
- 4 个独立的飞书机器人

---

#### 9.5.6 技能安装的差异

**Profile 实例模式**（需要分别安装）：
```bash
# 安装到 main 实例
cd ~/.openclaw/workspace/skills
skillhub install agent-backup-transfer

# 安装到 coder 实例
cd ~/.openclaw-coder/workspace/skills
skillhub install agent-backup-transfer

# 安装到 writer 实例
cd ~/.openclaw-writer/workspace/skills
skillhub install agent-backup-transfer

# 安装到 artist 实例
cd ~/.openclaw-artist/workspace/skills
skillhub install agent-backup-transfer
```

**Agent 模式**（一次安装，共享使用）：
```bash
# 所有 Agent 共享技能
cd ~/.openclaw/workspace/skills
skillhub install agent-backup-transfer
```

---

#### 9.5.7 推荐建议

**官方推荐**：优先使用 **Agent 模式**

**理由**：
1. ✅ **资源效率高** — 一个 Gateway 进程服务多个 Agent
2. ✅ **管理简单** — 统一配置、统一监控
3. ✅ **灵活路由** — 可以通过 bindings 精细控制消息流向
4. ✅ **隔离足够** — workspace、sessions、auth 都是独立的
5. ✅ **技能共享** — 安装一次，所有 Agent 可用

**何时使用 Profile 实例**：
1. **不同机器部署** — 家里一台、公司一台、VPS 一台
2. **完全独立的 Channel 账号** — 个人 WhatsApp vs 公司 WhatsApp
3. **测试/开发环境** — 不影响生产配置
4. **安全隔离需求** — 不同信任级别的任务

---

#### 9.5.8 迁移方案（如需切换）

**从 Profile 实例 → Agent 模式**：

```bash
# 1. 备份现有配置
cp -r ~/.openclaw-coder ~/.openclaw-coder.bak
cp -r ~/.openclaw-writer ~/.openclaw-writer.bak

# 2. 在 main 实例中添加 Agent
openclaw agents add coder --workspace ~/.openclaw-coder/workspace
openclaw agents add writer --workspace ~/.openclaw-writer/workspace

# 3. 配置 bindings
openclaw agents bind --agent coder --bind feishu:coder-bot
openclaw agents bind --agent writer --bind feishu:writer-bot

# 4. 停止独立实例
systemctl --user stop openclaw-coder.service
systemctl --user stop openclaw-writer.service

# 5. 重启 main 实例
systemctl --user restart openclaw-gateway.service

# 6. 验证路由
openclaw agents bindings
```

**注意事项**：
- 迁移前务必备份所有配置
- 需要重新配置飞书路由绑定
- 所有 Agent 将共享同一个飞书机器人
- 技能需要重新安装到 main 实例

---
## 第五部分：认证与同步

### 第 11 章：认证配置

#### 11.1 认证文件类型

| 文件 | 作用 | 位置 |
|------|------|------|
| `auth.json` | 基础认证信息 | `~/.openclaw/agents/main/agent/` |
| `auth-profiles.json` | OAuth Token | `~/.openclaw/agents/main/agent/` |
| `models.json` | 模型配置 | `~/.openclaw/agents/main/agent/` |

---

#### 8.2 同步认证配置

**为什么需要同步**：
- Profile 实例使用独立的状态目录
- 认证信息不会自动共享
- 需要手动从 Main 实例同步

**同步命令**：
```bash
# Main → Coder
cp ~/.openclaw/agents/main/agent/auth-profiles.json \
   ~/.openclaw-coder/agents/main/agent/

cp ~/.openclaw/agents/main/agent/models.json \
   ~/.openclaw-coder/agents/main/agent/

# Main → Writer
cp ~/.openclaw/agents/main/agent/auth-profiles.json \
   ~/.openclaw-writer/agents/main/agent/

cp ~/.openclaw/agents/main/agent/models.json \
   ~/.openclaw-writer/agents/main/agent/

# Main → Artist
cp ~/.openclaw/agents/main/agent/auth-profiles.json \
   ~/.openclaw-artist/agents/main/agent/

cp ~/.openclaw/agents/main/agent/models.json \
   ~/.openclaw-artist/agents/main/agent/
```

**重启实例**：
```bash
systemctl --user restart openclaw-coder.service
systemctl --user restart openclaw-writer.service
systemctl --user restart openclaw-artist.service
```

---

#### 8.3 更新 Bailian API Key

所有实例共用同一个 Bailian API Key：

```bash
# 检查当前配置
grep -A2 '"bailian"' ~/.openclaw/openclaw.json
grep -A2 '"bailian"' ~/.openclaw-coder/openclaw.json
grep -A2 '"bailian"' ~/.openclaw-writer/openclaw.json
grep -A2 '"bailian"' ~/.openclaw-artist/openclaw.json

# 如需更新，编辑 openclaw.json
# "apiKey": "sk-sp-YOUR_BAILIAN_API_KEY"
```

---

### 第 12 章：常见问题排查

#### 12.1 Token 过期

**症状**：
```
HTTP 401: invalid access token or token expired (auth)
```

**原因**：
- OAuth Token 过期
- 认证文件未同步

**解决**：
```bash
# 从 Main 实例同步最新认证
cp ~/.openclaw/agents/main/agent/auth-profiles.json \
   ~/.openclaw-coder/agents/main/agent/

# 重启实例
systemctl --user restart openclaw-coder.service
```

---

#### 9.2 API Key 不匹配

**症状**：
- 所有模型返回 401 错误
- 无法调用任何模型

**原因**：
- 各实例 `openclaw.json` 中的 API Key 不一致

**解决**：
```bash
# 检查 API Key
cat ~/.openclaw/openclaw.json | grep bailian.apiKey
cat ~/.openclaw-coder/openclaw.json | grep bailian.apiKey

# 统一 API Key（编辑 openclaw.json）
```

---

#### 9.3 工作空间路径错误

**症状**：
- 实例读取了错误的工作空间
- 配置文件不生效

**原因**：
- `openclaw.json` 中 `agents.list[].workspace` 路径错误

**检查**：
```bash
jq '.agents.list[].workspace' ~/.openclaw-writer/openclaw.json
jq '.agents.list[].workspace' ~/.openclaw-artist/openclaw.json
# 应该是：/home/lenovo/.openclaw-writer/workspace
# 错误示例：/home/lenovo/.openclaw-coder/workspace
```

**解决**：
```bash
# 编辑 openclaw.json，修正 workspace 路径
```

---

#### 9.4 路由未生效

**症状**：
- 消息未分发到对应实例
- 所有消息都由 Main 实例处理

**检查**：
```bash
openclaw agents bindings
```

**解决**：
```bash
# 重新绑定路由
openclaw agents bind --agent coder --bind feishu:coder-bot
openclaw agents bind --agent writer --bind feishu:writer-bot
openclaw agents bind --agent artist --bind feishu:artist-bot
```

---

## 第六部分：实例间通信

### 第 13 章：通信机制

#### 13.1 通信架构

```
┌──────────────────────────────────────────────────────────────┐
│  Main 实例 (🦞) - 协调者/管理者                               │
│  ✅ 需要 visibility=all                                       │
│  - 可以向 Coder/Writer/Artist 下达任务                         │
│  - 可以监控所有实例状态                                        │
└─────────────────────┬────────────────────────────────────────┘
                      │ 单向访问
         ┌────────────┼──────────────┐
         │            │              │
         ▼            ▼              ▼
┌────────────┐  ┌────────────┐  ┌────────────┐
│   Coder    │  │   Writer   │  │   Artist   │
│  ❌ 不需要  │  │  ❌ 不需要  │  │  ❌ 不需要  │
│  visibility │  │  visibility │  │  visibility │
└────────────┘  └────────────┘  └────────────┘
```

---

#### 10.2 配置跨实例访问

**仅 Main 实例需要配置**：

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "tools": {
    "sessions": {
      "visibility": "all"
    }
  }
}
```

**作用**：
- Main 实例可以访问 Coder/Writer/Artist 的会话
- Coder/Writer/Artist 不需要此配置（它们只执行任务）

**重启 Main 实例**：
```bash
systemctl --user restart openclaw-gateway.service
```

---

#### 10.3 会话键格式

| 实例 | 会话键 |
|------|--------|
| Main | `agent:main:main` |
| Coder | `agent:coder:coder` |
| Writer | `agent:writer:writer` |
| Artist | `agent:artist:artist` |

**使用示例**：
```javascript
sessions_send({
  sessionKey: "agent:artist:artist",
  message: "请设计一个 UI 配色方案"
})
```

---

### 第 14 章：任务下达方式

#### 14.1 sessions_send（直接会话）

**适用场景**：多轮对话、持续协作

```javascript
sessions_send({
  sessionKey: "agent:artist:artist",
  message: `【任务】设计 ClawClock 新主题

要求：
1. 赛博朋克风格
2. 主色调：紫色 + 蓝色
3. 考虑可读性

完成后请展示配色方案。`
})
```

**特点**：
- ✅ 向持久会话发送消息
- ✅ 保留完整对话历史
- ✅ 适合多步骤协作

---

#### 11.2 sessions_spawn（子代理）

**适用场景**：一次性任务

```javascript
sessions_spawn({
  task: "设计一个现代 UI 配色方案",
  agentId: "artist",
  mode: "run",
  streamTo: "parent"
})
```

**特点**：
- ✅ 创建临时子代理
- ✅ 任务完成后自动销毁
- ✅ 结果自动推送回 Main

---

#### 11.3 方式对比

| 方式 | 适用场景 | 生命周期 | 上下文保留 |
|------|----------|----------|-----------|
| `sessions_send` | 多轮对话、持续协作 | 持久 | ✅ 保留 |
| `sessions_spawn` | 一次性任务 | 临时 | ❌ 不保留 |

---

### 第 15 章：状态监控

#### 15.1 查询会话列表

```javascript
sessions_list({
  limit: 20
})
```

**输出示例**：
```json
{
  "sessions": [
    { "key": "agent:main:main", "updatedAt": 1773807491684 },
    { "key": "agent:coder:coder", "updatedAt": 1773806515052 },
    { "key": "agent:writer:writer", "updatedAt": 1773805178885 },
    { "key": "agent:artist:artist", "updatedAt": 1773809338000 }
  ]
}
```

---

#### 12.2 查看子代理状态

```javascript
subagents({
  action: "list",
  recentMinutes: 60
})
```

**输出**：
```
active subagents:
- coder: 代码优化中...
- writer: 文档编写中...
- artist: UI 设计进行中...
```

---

#### 12.3 读取会话历史

```javascript
sessions_history({
  sessionKey: "agent:artist:artist",
  limit: 50,
  includeTools: true
})
```

---

#### 12.4 系统级监控

```bash
# 查看所有实例状态
systemctl --user status openclaw-gateway.service openclaw-coder.service openclaw-writer.service openclaw-artist.service

# 查看实时日志
journalctl --user -u openclaw-artist.service -f

# 查看会话文件数量
ls -la ~/.openclaw-artist/agents/main/sessions/*.jsonl | wc -l
# 输出：2 个会话
```

---

## 第七部分：飞书集成

### 第 16 章：飞书应用配置

#### 16.1 创建飞书应用

1. 访问 https://open.feishu.cn
2. 登录企业账号
3. 点击「创建应用」
4. 选择「企业自建应用」
5. 填写应用名称
6. 点击「创建」

---

#### 13.2 配置机器人权限

**必需权限**：
- ✅ 消息发送权限
- ✅ 群组读取权限
- ✅ 事件订阅权限
- ✅ 机器人进入群聊权限

**配置步骤**：
1. 进入应用管理页面
2. 点击「权限管理」
3. 搜索并添加所需权限
4. 点击「发布」

---

#### 13.3 配置事件订阅

**方式 A：WebSocket 模式（推荐）**
- 无需配置 Request URL
- 自动接收消息事件
- 适合个人/小规模使用

**方式 B：HTTP 模式**
- 需要公网 URL
- 配置 Request URL
- 适合企业级部署

---

### 第 17 章：飞书路由绑定

#### 17.1 查看当前路由

```bash
openclaw agents bindings
```

**输出**：
```
Routing bindings:
- main <- feishu accountId=main-bot
- coder <- feishu accountId=coder-bot
- writer <- feishu accountId=writer-bot
- artist <- feishu accountId=artist-bot
```

---

#### 14.2 添加路由绑定

```bash
# Main 实例
openclaw agents bind --agent main --bind feishu:main-bot

# Coder 实例
openclaw agents bind --agent coder --bind feishu:coder-bot

# Writer 实例
openclaw agents bind --agent writer --bind feishu:writer-bot

# Artist 实例
openclaw agents bind --agent artist --bind feishu:artist-bot
```

---

#### 14.3 路由规则

路由规则存储在 `~/.openclaw/openclaw.json`：

```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "accountId": "main-bot"
      }
    },
    {
      "type": "route",
      "agentId": "coder",
      "match": {
        "channel": "feishu",
        "accountId": "coder-bot"
      }
    },
    {
      "type": "route",
      "agentId": "writer",
      "match": {
        "channel": "feishu",
        "accountId": "writer-bot"
      }
    },
    {
      "type": "route",
      "agentId": "artist",
      "match": {
        "channel": "feishu",
        "accountId": "artist-bot"
      }
    }
  ]
}
```

---

### 第 18 章：配对授权流程

#### 18.1 用户发送消息触发配对

飞书用户给 Bot 发送第一条消息时：

```
Bot 回复：
OpenClaw: access not configured.
Your Feishu user id: ou_YOUR_FEISHU_USER_ID
Pairing code: ABCD1234
Ask the bot owner to approve with:
openclaw pairing approve feishu ABCD1234
```

---

#### 15.2 执行授权命令

```bash
# Main 实例
openclaw pairing approve feishu ABCD1234

# Coder 实例
openclaw --profile coder pairing approve feishu ABCD1234

# Writer 实例
openclaw --profile writer pairing approve feishu ABCD1234

# Artist 实例
openclaw --profile artist pairing approve feishu ABCD1234
```

---

#### 15.3 查看授权状态

```bash
cat ~/.openclaw/credentials/feishu-default-allowFrom.json
```

**输出**：
```json
{
  "version": 1,
  "allowFrom": [
    "ou_YOUR_FEISHU_USER_ID"
  ]
}
```

---

## 第八部分：运维与管理

### 第 19 章：日常运维

#### 19.1 服务管理

```bash
# 启动所有实例
systemctl --user start openclaw-gateway.service
systemctl --user start openclaw-coder.service
systemctl --user start openclaw-writer.service
systemctl --user start openclaw-artist.service

# 停止所有实例
systemctl --user stop openclaw-*.service

# 重启单个实例
systemctl --user restart openclaw-artist.service

# 查看状态
systemctl --user status openclaw-*.service
```

---

#### 16.2 日志查看

```bash
# 实时日志（按 Ctrl+C 退出）
journalctl --user -u openclaw-gateway.service -f

# 最近 1 小时日志
journalctl --user -u openclaw-artist.service --since "1 hour ago"

# 查看错误日志
journalctl --user -u openclaw-writer.service --since "1 hour ago" | grep -i error

# 导出日志
journalctl --user -u openclaw-gateway.service > openclaw-main.log
```

---

#### 16.3 配置备份

```bash
# 备份配置文件
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak.$(date +%Y%m%d)
cp -r ~/.openclaw/workspace ~/.openclaw/workspace.bak.$(date +%Y%m%d)

# 备份 systemd 服务文件
cp ~/.config/systemd/user/openclaw-*.service ~/.config/systemd/user/backup/
```

---

### 第 20 章：配置验证清单

#### 20.1 创建新实例后的检查

- [ ] `gateway.port` - 端口是否正确（不冲突）
- [ ] `gateway.auth.token` - Token 是否独立
- [ ] `channels.feishu.appId` - App ID 是否正确
- [ ] `channels.feishu.appSecret` - App Secret 是否正确
- [ ] `bindings[0].accountId` - Bot 账号是否正确
- [ ] `agents.list[].workspace` - 工作空间路径是否正确 ⚠️
- [ ] `agents.list[].agentDir` - Agent 目录是否正确
- [ ] 认证配置已同步（auth-profiles.json、models.json）
- [ ] 路由已绑定（`openclaw agents bindings`）
- [ ] 飞书配对已授权

---

#### 17.2 验证命令

```bash
# 对比配置差异
diff -u ~/.openclaw-coder/openclaw.json ~/.openclaw-writer/openclaw.json
diff -u ~/.openclaw-coder/openclaw.json ~/.openclaw-artist/openclaw.json

# 检查关键字段
jq '.agents.list[].workspace' ~/.openclaw-writer/openclaw.json
jq '.agents.list[].workspace' ~/.openclaw-artist/openclaw.json

# 测试模型可用性
openclaw --profile artist agent -m "测试" --session-id test-123

# 检查路由
openclaw agents bindings

# 检查服务状态
systemctl --user status openclaw-artist.service
```

---

## 第九部分：实战案例

### 第 21 章：ClawClock 项目协作

#### 21.1 项目背景

**ClawClock** 是一个图形化时钟应用：
- Python + Tkinter 实现
- 支持模拟时钟和数字时钟
- 25 时区支持
- 4 种主题
- 100+ 自动化测试

**GitHub**: https://github.com/lyy3431/clawclock

---

#### 18.2 任务分工

| 实例 | 任务 |
|------|------|
| **Main** | 项目协调、进度监控、GIF 制作 |
| **Coder** | 代码开发、测试、Bug 修复 |
| **Writer** | 文档编写、README 优化 |
| **Artist** | UI 设计、配色方案、图标设计 |

---

#### 18.3 协作流程

```
1. Main → sessions_send → Coder: "优化代码"
2. Main → sessions_send → Writer: "更新文档"
3. Main → sessions_send → Artist: "设计新主题"
4. Coder → 自动回复 → Main: "代码完成"
5. Writer → 自动回复 → Main: "文档完成"
6. Artist → 自动回复 → Main: "设计完成"
7. Main → 用户汇报：任务完成
```

---

### 第 22 章：配置错误修复实录

#### 22.1 Writer 实例配置错误

**发现时间**：2026-03-18 12:08

**错误配置**：
```json
// ~/.openclaw-writer/openclaw.json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "workspace": "/home/lenovo/.openclaw-coder/workspace"  // ❌ 错误！
      }
    ]
  }
}
```

**正确配置**：
```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "workspace": "/home/lenovo/.openclaw-writer/workspace"  // ✅ 正确
      }
    ]
  }
}
```

---

#### 19.2 修复步骤

```bash
# 1. 发现错误（对比配置文件）
diff -u ~/.openclaw-coder/openclaw.json ~/.openclaw-writer/openclaw.json

# 2. 编辑修复
# 编辑 ~/.openclaw-writer/openclaw.json

# 3. 重启服务
systemctl --user restart openclaw-writer.service

# 4. 验证功能
openclaw --profile writer agent -m "测试" --session-id test-fix
```

---

#### 19.3 Artist 实例配置问题（长连接失败）

**发现时间**：2026-03-18 19:42

**症状**：
- 飞书后台检测不到 Artist 的长连接
- WebSocket 连接失败
- 日志报错：`system busy` 和 `Cannot read properties of undefined`

---

##### 问题 1：App Secret 拼写错误

**错误配置**：
```json
{
  "channels": {
    "feishu": {
      "appId": "cli_YOUR_ARTIST_APP_ID",
      "appSecret": "YOUR_ARTIST_APP_SECRET"  // ❌ 示例：拼写错误
    }
  }
}
```

**正确配置**：
```json
{
  "channels": {
    "feishu": {
      "appId": "cli_YOUR_ARTIST_APP_ID",
      "appSecret": "YOUR_ARTIST_APP_SECRET"  // ✅ 正确（示例）
    }
  }
}
```

**修复方法**：
```bash
# 编辑配置文件
nano ~/.openclaw-artist/openclaw.json

# 修正 App Secret（删除多余的 E）

# 重启服务
systemctl --user restart openclaw-artist.service
```

---

##### 问题 2：缺少 commands 配置

**错误配置**：
```json
{
  // Artist 配置中没有 commands 部分 ❌
  "channels": { ... },
  "gateway": { ... }
}
```

**正确配置**：
```json
{
  "commands": {                    // ✅ 添加 commands 配置
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "channels": { ... },
  "gateway": { ... }
}
```

**修复方法**：
```bash
# 从 Coder 实例复制 commands 配置
# 编辑 ~/.openclaw-artist/openclaw.json
# 在 channels 之前添加 commands 部分

# 重启服务
systemctl --user restart openclaw-artist.service
```

---

##### 问题 3：飞书长连接未启用

**症状**：
```
[error]: receive events or callbacks through persistent connection only available 
in self-build & Feishu app, Configured in:
  Developer Console(开发者后台) 
    -> Events and Callbacks(事件与回调)
    -> Mode of event/callback subscription(订阅方式)
    -> Receive events/callbacks through persistent connection(使用长连接接收事件/回调)
```

**原因**：
- 飞书开发者后台未开启长连接接收事件
- 需要在飞书后台手动配置

**解决步骤**：

1. **登录飞书开放平台**
   - 访问：https://open.feishu.cn

2. **进入 Artist 应用管理**
   - App ID: `cli_YOUR_ARTIST_APP_ID`

3. **配置事件与回调**
   - 点击「事件与回调」（Events and Callbacks）

4. **启用长连接**
   - 找到「订阅方式」（Mode of event/callback subscription）
   - 选择：✅ **使用长连接接收事件/回调**
   - (Receive events/callbacks through persistent connection)

5. **保存配置**
   - 点击保存按钮

6. **验证连接**
```bash
# 查看日志确认 WebSocket 连接成功
journalctl --user -u openclaw-artist.service -f

# 成功日志：
# [feishu] feishu[default]: WebSocket client started
# [info]: [ '[ws]', 'ws client ready' ]  ✅
```

---

##### 验证日志

**成功连接的日志**：
```log
Mar 18 19:43:12 lenovo2026 node[196009]: 
2026-03-18T19:43:12.259+08:00 [feishu] feishu[default]: WebSocket client started
Mar 18 19:43:12 lenovo2026 node[196009]: 
2026-03-18T19:43:12.493+08:00 [info]: [ '[ws]', 'ws client ready' ]
```

**失败日志**（修复前）：
```log
Mar 18 19:42:39 lenovo2026 node[195525]: 
2026-03-18T19:42:39.706+08:00 [error]: [ '[ws]', 'code: 1000040345, system busy' ]
Mar 18 19:42:39 lenovo2026 node[195525]: 
2026-03-18T19:42:39.716+08:00 [error]: [ '[ws]', 'Cannot read properties of undefined' ]
Mar 18 19:42:39 lenovo2026 node[195525]: 
2026-03-18T19:42:39.724+08:00 [info]: [ 'ws', 'unable to connect to the server after trying 6 times")' ]
```

---

#### 19.4 经验总结

**教训**：
1. 复制模板配置时要逐项检查所有字段
2. 特别关注路径、端口、密钥等关键配置
3. App Secret 等凭证要仔细核对，避免拼写错误
4. 配置完成后必须验证
5. 飞书长连接需要在开发者后台手动开启
6. 记录错误以便后续参考

**改进措施**：
- 在 MEMORY.md 中记录「变更验证原则」
- 创建配置验证清单（见第 17 章）
- 编写自动化验证脚本（待完成）
- 添加 Artist 配置问题到故障排除章节

---

#### 19.5 配置检查清单（Artist 实例）

创建 Artist 实例后务必检查：

- [ ] **App ID**：`cli_YOUR_ARTIST_APP_ID`（格式：`cli_xxxxxxxxxxxxxxxx`）
- [ ] **App Secret**：`YOUR_ARTIST_APP_SECRET`（无拼写错误）
- [ ] **commands 配置**：已添加 commands 部分
- [ ] **端口**：19889（不与其他实例冲突）
- [ ] **WebSocket**：日志显示 `ws client ready`
- [ ] **飞书后台**：已开启长连接接收事件
- [ ] **路由绑定**：`openclaw agents bindings` 显示 artist-bot
- [ ] **飞书配对**：已授权用户访问

---

## 第十部分：附录

### 附录 A：命令速查表

#### Agents 管理
```bash
openclaw agents list                          # 列出所有 agents
openclaw agents bindings                      # 查看路由绑定
openclaw agents bind --agent artist --bind feishu:artist-bot  # 添加路由
openclaw agents unbind --agent writer --bind feishu:writer-bot  # 移除路由
```

#### 服务管理
```bash
systemctl --user status openclaw-*.service    # 查看所有实例状态
systemctl --user restart openclaw-artist.service  # 重启 Artist
journalctl --user -u openclaw-artist.service -f  # 实时日志
```

#### 配对授权
```bash
openclaw pairing approve feishu <CODE>        # Main 实例
openclaw --profile coder pairing approve feishu <CODE>  # Coder 实例
openclaw --profile writer pairing approve feishu <CODE>  # Writer 实例
openclaw --profile artist pairing approve feishu <CODE>  # Artist 实例
```

#### 模型管理
```bash
openclaw models list                          # 查看模型列表
openclaw --profile artist models list         # 查看 Artist 的模型
```

#### 测试连接
```bash
openclaw --profile artist agent -m "测试" --session-id test-123
```

---

### 附录 B：配置文件模板

#### B.1 openclaw.json 模板

```json
{
  "meta": {
    "lastTouchedVersion": "2026.3.13",
    "lastTouchedAt": "2026-03-18T00:00:00.000Z"
  },
  "models": {
    "mode": "merge",
    "providers": {
      "bailian": {
        "baseUrl": "https://coding.dashscope.aliyuncs.com/v1",
        "apiKey": "sk-sp-YOUR_BAILIAN_API_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3.5-plus",
            "name": "qwen3.5-plus",
            "reasoning": false,
            "input": ["text", "image"],
            "cost": { "input": 0, "output": 0 },
            "contextWindow": 1000000,
            "maxTokens": 65536
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "bailian/qwen3.5-plus"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "/home/lenovo/.openclaw/workspace"
      }
    ]
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "feishu",
        "accountId": "main-bot"
      }
    }
  ],
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_YOUR_APP_ID",
      "appSecret": "YOUR_APP_SECRET",
      "connectionMode": "websocket",
      "domain": "feishu",
      "groupPolicy": "open"
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "YOUR_GATEWAY_TOKEN"
    }
  },
  "plugins": {
    "entries": {
      "feishu": {
        "enabled": true
      }
    }
  }
}
```

---

#### B.2 systemd 服务文件模板

```ini
[Unit]
Description=OpenClaw Gateway - {INSTANCE} Instance (v2026.3.13)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/linuxbrew/.linuxbrew/bin/node \
  "/home/linuxbrew/.linuxbrew/lib/node_modules/openclaw/dist/index.js" \
  {PROFILE} gateway --port {PORT}
Restart=always
RestartSec=5
KillMode=process
Environment=HOME=/home/lenovo
Environment=OPENCLAW_PROFILE={PROFILE}
Environment=OPENCLAW_STATE_DIR=/home/lenovo/.openclaw-{PROFILE}
Environment=OPENCLAW_WORKSPACE=/home/lenovo/.openclaw/workspace-{PROFILE}
Environment=OPENCLAW_GATEWAY_PORT={PORT}
Environment=OPENCLAW_GATEWAY_TOKEN={GATEWAY_TOKEN}
Environment=OPENCLAW_SERVICE_MARKER=openclaw-{PROFILE}
Environment=OPENCLAW_SERVICE_KIND=gateway
Environment=OPENCLAW_SERVICE_VERSION=2026.3.13

[Install]
WantedBy=default.target
```

**变量替换**：
- `{INSTANCE}`: Main/Coder/Writer/Artist
- `{PROFILE}`: 空（Main）/ `--profile coder` / `--profile writer` / `--profile artist`
- `{PORT}`: 18789 / 18989 / 18689 / 19889
- `{GATEWAY_TOKEN}`: 独立的 token

---

#### B.3 IDENTITY.md 模板

```markdown
# IDENTITY.md - Who Am I?

- **Name:** {实例名称}
- **Creature:** {生物类型}
- **Vibe:** {性格特点}
- **Emoji:** {表情符号}
- **Avatar:** (same as main)

---

{简短描述}
```

---

#### B.4 SOUL.md 模板

```markdown
# SOUL.md - {实例名称}

_You are the {角色描述} of the OpenClaw family._

## Core Identity

- **Focus**: {专注领域}
- **Style**: {行为风格}
- **Tools**: {工具权限}

## Behavior

- {行为准则 1}
- {行为准则 2}
- {行为准则 3}

## Boundaries

- {边界 1}
- {边界 2}

---

_{座右铭}_
```

---

### 附录 C：故障排除

#### C.1 常见问题 FAQ

**Q: 实例启动失败**
```bash
# 查看详细错误
journalctl --user -u openclaw-artist.service --since "5 minutes ago"

# 检查端口占用
ss -tlnp | grep 19889

# 检查配置文件
cat ~/.openclaw-artist/openclaw.json | jq .
```

**Q: 飞书消息不响应**
```bash
# 检查路由绑定
openclaw agents bindings

# 检查配对授权
cat ~/.openclaw/credentials/feishu-default-allowFrom.json

# 检查日志
journalctl --user -u openclaw-gateway.service | grep feishu
```

**Q: 模型调用失败**
```bash
# 检查 API Key
grep apiKey ~/.openclaw/openclaw.json

# 测试模型
openclaw agent -m "测试" --session-id test-1

# 查看认证文件
cat ~/.openclaw/agents/main/agent/auth-profiles.json
```

---

#### C.2 错误代码对照表

| 错误代码 | 含义 | 解决方案 |
|---------|------|----------|
| 401 | 认证失败 | 检查 API Key、Token |
| 403 | 权限不足 | 检查飞书权限配置 |
| 404 | 资源不存在 | 检查路径、路由 |
| 500 | 服务器错误 | 查看日志、重启服务 |
| EADDRINUSE | 端口被占用 | 更换端口或停止占用进程 |

---

#### C.3 日志分析技巧

```bash
# 提取错误信息
journalctl --user -u openclaw-gateway.service | grep -i error

# 提取飞书消息
journalctl --user -u openclaw-gateway.service | grep feishu

# 按时间过滤
journalctl --user -u openclaw-gateway.service --since "2026-03-18 10:00:00" --until "2026-03-18 12:00:00"

# 导出日志分析
journalctl --user -u openclaw-gateway.service > main.log
grep -E "error|fail|exception" main.log
```

---

### 附录 D：版本历史

| 日期 | 事件 | 备注 |
|------|------|------|
| 2026-03-04 | Main 实例初始化 | OpenClaw 诞生 |
| 2026-03-07 | 配置 Bailian 模型 | 8 个模型可用 |
| 2026-03-14 | ClawClock 项目启动 | 公司第一个项目 |
| 2026-03-17 09:00 | Coder 实例配置完成 | 独立运行 |
| 2026-03-17 11:20 | Writer 实例配置完成 | 三实例架构成型 |
| 2026-03-17 12:00 | 飞书路由绑定完成 | 三个 Bot 独立接收消息 |
| 2026-03-18 11:25 | 配置 `tools.sessions.visibility=all` | 实现实例间通信 |
| 2026-03-18 12:08 | 修复 Writer 实例配置错误 | workspace 路径错误 |
| 2026-03-18 12:15 | 记录「变更验证原则」 | 写入 MEMORY.md |
| 2026-03-18 18:49 | 编写本教程 | 版本 1.0 完成 |
| 2026-03-18 19:27 | Artist 实例配置完成 | 四实例协作架构 |
| 2026-03-18 19:30 | 更新教程添加 Artist 章节 | 版本 1.1 完成 |
| 2026-03-18 19:32 | 修正 Artist 端口为 19889 | 端口更正 |
| 2026-03-18 19:42 | 修复 Artist 长连接问题 | App Secret + commands + 飞书配置 |
| 2026-03-18 19:44 | 补充 Artist 配置问题到教程 | 版本 1.2 完成 |
| 2026-03-19 07:00 | 安装 7 个新技能 | agent-backup-transfer, agent-hush, agent-error-logger-new, agent-advisor, productivity, personal-assistant, clawchat-p2p |
| 2026-03-19 09:00 | 修复 GitHub Token 泄露 | 撤销旧 Token，切换 SSH 认证 |
| 2026-03-19 14:00 | 发现 4 个 Profile 实例 | main, coder, writer, artist |
| 2026-03-19 15:00 | 添加架构模式对比章节 | Profile 实例 vs Agent 模式详解，版本 1.3 |
| 2026-03-19 17:00 | 重新编排章节结构 | 优化目录逻辑，所有章节顺序编号，版本 1.4 |

---

## 📚 后记

本教程基于真实配置经验编写，所有命令和配置均已验证。

**编写团队**：
- 🦞 Main - 总体架构、通信机制
- 💻 Coder - 技术细节、命令验证
- ✍️ Writer - 文档编写、排版优化
- 🎨 Artist - 艺术设计、视觉优化
- 🎨 Artist - UI 设计、视觉优化

**反馈与建议**：
欢迎通过 GitHub Issues 或飞书联系作者。

---

_© 2026 OpenClaw Team. MIT License._
