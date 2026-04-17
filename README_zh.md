# Agent Clone — AI Agent 通用迁移工具

将你的 AI Agent 完整身份（配置、人格、记忆、技能、会话）迁移到任意新环境。

**[English](./README.md)** | **中文**

## 支持的框架

| 框架 | 配置文件 | 人格文件 | 记忆 | 技能 | 会话 | 凭证 |
|------|---------|---------|------|------|------|------|
| **Hermes** | config.yaml | SOUL.md | memory_store.db + memories/ | skills/ | state.db + sessions/ | .env + auth.json |
| **OpenClaw** | openclaw.json | workspace/SOUL.md | memory/main.sqlite | workspace/skills/ | agents/main/sessions/ | credentials/ + identity/ |
| **Claude Code** | settings.json | CLAUDE.md | memory.md | skills/ | sessions/ | (环境变量) |

## 快速开始

### 打包（源机器）

```bash
# 打包单个框架 — Hermes
cd ~ && tar czf hermes-clone.tar.gz \
  .hermes/config.yaml .hermes/SOUL.md .hermes/.env .hermes/auth.json \
  .hermes/memories/ .hermes/memory_store.db .hermes/state.db \
  .hermes/skills/ .hermes/sessions/ 2>/dev/null

# 打包单个框架 — OpenClaw
tar czf openclaw-clone.tar.gz \
  .openclaw/openclaw.json .openclaw/workspace/ .openclaw/memory/ \
  .openclaw/agents/main/sessions/ .openclaw/credentials/ .openclaw/identity/ \
  2>/dev/null

# 打包单个框架 — Claude Code
tar czf claude-code-clone.tar.gz \
  .claude/settings.json .claude/CLAUDE.md .claude/memory.md \
  .claude/skills/ .claude/projects/ 2>/dev/null

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

### 传输 & 部署（目标机器）

```bash
scp agent-backup.tar.gz 目标机器:~
ssh 目标机器
tar xzf agent-backup.tar.gz
# 按框架专属步骤适配（详见 SKILL.md）
```

## 特性

- **跨架构支持**：x86_64 ↔ ARM（树莓派、Mac M 系列等）
- **精简/完整打包**：可排除大型二进制文件，也可全量拷贝
- **安全指引**：敏感数据清单、加密传输方案
- **常见问题排查**：每个框架的典型问题及解决方案
- **跨框架迁移**：在 Hermes / OpenClaw / Claude Code 之间迁移人格和记忆

## 完整文档

👉 **[SKILL.md](./SKILL.md)** — 完整参考手册，包含：

- 三框架详细文件路径映射
- 跨架构兼容性矩阵（x86 ↔ arm64）
- 版本兼容说明
- 各框架逐步部署指南
- 部署后验证清单
- 安全与隐私指引
- 跨框架迁移对照表

## 安装为 Hermes Skill

```bash
mkdir -p ~/.hermes/skills/devops/agent-clone
curl -o ~/.hermes/skills/devops/agent-clone/SKILL.md \
  https://raw.githubusercontent.com/gegewu81/agent-clone/master/SKILL.md
```

## 许可证

MIT
