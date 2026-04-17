# Agent Clone — AI Agent 通用迁移工具

将你的 AI Agent 完整身份（配置、人格、记忆、技能、会话）迁移到任意新环境。

**[English](./README.md)** | **中文**

---

## 支持的框架

| 框架 | 配置文件 | 人格文件 | 记忆 | 技能 | 会话 | 凭证 |
|------|---------|---------|------|------|------|------|
| **Hermes** | config.yaml | SOUL.md | memory_store.db + memories/ | skills/ | state.db + sessions/ | .env + auth.json |
| **OpenClaw** | openclaw.json | workspace/SOUL.md | memory/main.sqlite | workspace/skills/ | agents/main/sessions/ | credentials/ + identity/ |
| **Claude Code** | settings.json | CLAUDE.md | memory.md | skills/ | sessions/ | (环境变量) |

---

## 快速开始

### 1. 打包（源机器上执行）

```bash
# 打包单个框架 — Hermes
cd ~
tar czf hermes-clone.tar.gz \
  .hermes/config.yaml \
  .hermes/SOUL.md \
  .hermes/.env \
  .hermes/auth.json \
  .hermes/memories/ \
  .hermes/memory_store.db \
  .hermes/state.db \
  .hermes/skills/ \
  .hermes/sessions/ \
  2>/dev/null

# 打包单个框架 — OpenClaw
tar czf openclaw-clone.tar.gz \
  .openclaw/openclaw.json \
  .openclaw/workspace/ \
  .openclaw/memory/ \
  .openclaw/agents/main/sessions/ \
  .openclaw/credentials/ \
  .openclaw/identity/ \
  2>/dev/null

# 打包单个框架 — Claude Code
tar czf claude-code-clone.tar.gz \
  .claude/settings.json \
  .claude/CLAUDE.md \
  .claude/memory.md \
  .claude/skills/ \
  .claude/projects/ \
  2>/dev/null

# 一键打包全部框架
tar czf agent-backup-$(date +%Y%m%d).tar.gz \
  .hermes/config.yaml .hermes/SOUL.md .hermes/.env .hermes/auth.json \
  .hermes/memories/ .hermes/memory_store.db .hermes/state.db \
  .hermes/skills/ .hermes/sessions/ \
  .openclaw/openclaw.json .openclaw/workspace/ .openclaw/memory/ \
  .openclaw/credentials/ .openclaw/identity/ \
  .claude/settings.json .claude/CLAUDE.md .claude/skills/ .claude/projects/ \
  2>/dev/null
```

### 2. 传输到目标机器

```bash
# 方式A：局域网 SCP
scp agent-backup.tar.gz 目标机器:~

# 方式B：加密传输（推荐，通过公网时）
tar czf - <文件列表> | gpg --symmetric --cipher-algo AES256 -o backup.tar.gz.gpg
# 目标机器解密：
gpg -d backup.tar.gz.gpg | tar xzf -
```

### 3. 部署（目标机器上执行）

```bash
cd ~
tar xzf agent-backup.tar.gz
# 然后根据下面的框架专属步骤进行适配
```

---

## 框架专属部署步骤

### Hermes

```bash
# 1. 先安装 Hermes（创建初始目录结构）
pip install hermes-agent

# 2. 停止运行中的实例
pkill -f hermes 2>/dev/null

# 3. 解压备份
tar xzf hermes-clone.tar.gz

# 4. 跨架构适配
ARCH=$(uname -m)
if [ "$ARCH" != "x86_64" ]; then
  echo "检测到非 x86_64 架构 ($ARCH)，禁用 tirith"
  sed -i 's/tirith_enabled: true/tirith_enabled: false/' ~/.hermes/config.yaml
fi

# 5. 重建 Python 虚拟环境（跨架构必须）
cd ~/.hermes/hermes-agent && python3 -m venv venv && ./venv/bin/pip install -e .

# 6. 创建符号链接
mkdir -p ~/.local/bin
ln -sf ~/.hermes/hermes-agent/venv/bin/hermes ~/.local/bin/hermes

# 7. 验证
hermes doctor
```

**需要适配的配置项：**
- `tirith_enabled` — 非 x86_64 架构设为 false
- `mcp_servers` — 移除目标环境不存在的 MCP 服务（如 windows-mcp）
- `custom_providers.ollama` — 目标无 ollama 或内存不足时删除
- `.env` — 确认 API Key 在新网络可用
- Node.js — 确保 npx 在 PATH 中

### OpenClaw

```bash
# 1. 停止运行中的实例
pkill -f openclaw 2>/dev/null

# 2. 解压备份
tar xzf openclaw-clone.tar.gz

# 3. 设置权限
chmod 600 ~/.openclaw/credentials/*.json 2>/dev/null
chmod 600 ~/.openclaw/identity/*.json 2>/dev/null
chmod 700 ~/.openclaw/

# 4. 验证记忆数据库
sqlite3 ~/.openclaw/memory/main.sqlite "SELECT count(*) FROM chunks;"

# 5. 启动测试
openclaw --version
```

**需要适配的配置项：**
- `openclaw.json` — 更新硬编码路径
- `credentials/` — 部分平台凭证可能绑定设备，需重新授权
- `extensions/` — 确认扩展路径有效

### Claude Code

```bash
# 1. 解压备份
tar xzf claude-code-clone.tar.gz

# 2. 更新 settings.json 中的环境变量
#    - ANTHROPIC_AUTH_TOKEN 是否有效
#    - ANTHROPIC_BASE_URL 是否可达
#    - 插件兼容性

# 3. 项目级 CLAUDE.md 需放回对应项目目录

# 4. 启动测试
claude --version
claude
```

---

## 数据目录完整对照表

### Hermes 数据结构

```
~/.hermes/
├── config.yaml              # 主配置（模型、压缩、MCP 等）
├── SOUL.md                  # Agent 人格定义
├── .env                     # API Key、环境变量
├── auth.json                # 认证令牌
├── channel_directory.json   # 通道目录
├── memories/
│   ├── MEMORY.md            # Agent 工作记忆（Markdown）
│   └── USER.md              # 用户画像（Markdown）
├── memory_store.db          # 结构化记忆（SQLite / fact_store）
├── state.db                 # 会话状态（SQLite）
├── sessions/                # 会话归档
├── skills/                  # 技能目录（SKILL.md 文件树）
├── context_length_cache.yaml # 上下文压缩缓存
├── pairing/                 # 通道配对信息
├── hermes-agent/            # 源码 + venv（架构相关，不跨平台）
└── bin/tirith               # 安全沙箱二进制（架构相关）
```

### OpenClaw 数据结构

```
~/.openclaw/
├── openclaw.json            # 主配置（模型、MCP、扩展、通道、凭证）
├── workspace/
│   ├── SOUL.md              # Agent 人格
│   ├── IDENTITY.md          # Agent 身份文档
│   ├── USER.md              # 用户偏好
│   ├── AGENTS.md            # Agent 描述
│   ├── TOOLS.md             # 工具描述
│   ├── HEARTBEAT.md         # 心跳配置
│   ├── config/mcporter.json # MCP 服务器配置
│   ├── memory/              # 工作记忆
│   └── skills/              # 技能定义
├── memory/
│   └── main.sqlite          # 长期记忆（FTS5 + 向量嵌入）
├── agents/main/sessions/    # 会话记录（JSONL）
├── credentials/             # 平台认证（飞书、微信等）
├── identity/                # 设备认证
│   ├── device.json
│   └── device-auth.json
├── cron/                    # 定时任务
├── extensions/              # 已安装扩展
├── browser/                 # 浏览器数据（通常不迁移）
└── logs/                    # 日志（通常不迁移）
```

### Claude Code 数据结构

```
~/.claude/
├── settings.json            # 全局设置（API Key、插件、语言等）
├── CLAUDE.md                # 全局指令（如存在）
├── memory.md                # Agent 记忆（如存在）
├── sessions/                # 会话记录
├── projects/                # 项目级设置和记忆
│   └── -path-to-project/
│       ├── CLAUDE.md        # 项目级指令
│       └── ...              # 项目记忆
├── skills/                  # 自定义技能
├── history.jsonl            # 命令历史
├── todos/                   # 待办事项
├── plugins/                 # 插件
├── stats-cache.json         # 使用统计（可选）
└── transcripts/             # 完整对话记录
```

---

## 跨架构兼容性矩阵

| 数据类型 | x86→x86 | x86→arm64 | arm64→x86 | arm64→arm64 |
|----------|---------|-----------|-----------|-------------|
| 配置文件（YAML/JSON/MD） | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 |
| SQLite 数据库 | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 |
| 文本技能（*.md, *.py） | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 | ✅ 直接拷贝 |
| Python venv | ✅ 直接拷贝 | ❌ 必须重建 | ❌ 必须重建 | ✅ 直接拷贝 |
| 原生二进制（tirith 等） | ✅ 直接拷贝 | ❌ 不可用 | ❌ 不可用 | ✅ 直接拷贝 |
| Node 原生模块 | ✅ 直接拷贝 | ⚠️ 需检查 | ⚠️ 需检查 | ✅ 直接拷贝 |

**核心原则**：纯文本/数据文件可移植；编译产物（venv、二进制、原生模块）必须在目标环境重建。

---

## 安全注意事项

### 绝对不要提交到公开仓库的文件

| 框架 | 文件 | 包含内容 |
|------|------|---------|
| Hermes | `.env` | API Key、Token |
| Hermes | `auth.json` | 认证令牌 |
| OpenClaw | `credentials/` | 平台认证（微信、飞书等） |
| OpenClaw | `identity/device-auth.json` | 设备认证 |
| Claude Code | `settings.json` | `ANTHROPIC_AUTH_TOKEN` |

### 传输安全建议

- 局域网内用 `scp` 即可
- 跨公网建议用 GPG 加密：`tar czf - <文件> | gpg --symmetric -o backup.gpg`
- 部分平台的凭证（如 OpenClaw 飞书/微信）绑定设备，迁移后可能需要重新授权

---

## 常见问题

### Hermes: "state.db schema 不兼容"
不同版本可能数据库结构不同。建议：让新版本创建新的 state.db，只迁移 config、SOUL.md、memories/、skills/。

### Hermes: "tirith 段错误"
二进制文件架构不匹配。在 config.yaml 中设 `tirith_enabled: false`。

### OpenClaw: "记忆数据库锁定"
确保没有其他 OpenClaw 实例在运行。拷贝时排除 `-shm` 和 `-wal` 文件。

### Claude Code: "API Key 失效"
部分 Key 绑定 IP 或机器，需要在新环境重新生成。

### 任何框架: "Python venv 损坏"
跨架构不能拷贝 venv，必须重建：
```bash
cd ~/.hermes/hermes-agent
rm -rf venv
python3 -m venv venv
./venv/bin/pip install -e .
```

---

## 跨框架迁移

想从框架 A 迁移到框架 B？人格和记忆文本是最可移植的部分：

| 方向 | 人格 | 记忆 | 技能 |
|------|------|------|------|
| Hermes → OpenClaw | SOUL.md → workspace/SOUL.md | MEMORY.md → workspace/memory/ | 手动移植 |
| OpenClaw → Hermes | workspace/SOUL.md → SOUL.md | 提取为文本 → memories/ | 手动移植 |
| Claude Code → Hermes | CLAUDE.md → SOUL.md | memory.md → memories/ | 手动移植 |
| Hermes → Claude Code | SOUL.md → CLAUDE.md | memories/ → memory.md | 手动改写 |

跨框架迁移无法自动完成——框架架构差异太大。但人格（Markdown 文本）和记忆（文本内容）是最容易迁移的元素。

---

## 安装为 Hermes Skill

```bash
mkdir -p ~/.hermes/skills/devops/agent-clone
curl -o ~/.hermes/skills/devops/agent-clone/SKILL.md \
  https://raw.githubusercontent.com/gegewu81/agent-clone/master/SKILL.md
```

## 许可证

MIT
