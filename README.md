# Agent Clone

Universal AI Agent migration tool. Clone your agent's complete identity to any new environment.

## Supported Frameworks

| Framework | Config | Persona | Memory | Skills | Sessions | Credentials |
|-----------|--------|---------|--------|--------|----------|-------------|
| **Hermes** | config.yaml | SOUL.md | memory_store.db + memories/ | skills/ | state.db + sessions/ | .env + auth.json |
| **OpenClaw** | openclaw.json | workspace/SOUL.md | memory/main.sqlite | workspace/skills/ | agents/main/sessions/ | credentials/ + identity/ |
| **Claude Code** | settings.json | CLAUDE.md | memory.md | skills/ | sessions/ | (env vars) |

## Quick Start

### Pack (on source machine)

```bash
# Pack a single framework
tar czf hermes-clone.tar.gz \
  .hermes/config.yaml .hermes/SOUL.md .hermes/.env .hermes/auth.json \
  .hermes/memories/ .hermes/memory_store.db .hermes/state.db \
  .hermes/skills/ .hermes/sessions/

# Or pack ALL frameworks at once
cd ~ && tar czf agent-backup.tar.gz \
  .hermes/config.yaml .hermes/SOUL.md .hermes/.env .hermes/memories/ \
  .hermes/memory_store.db .hermes/state.db .hermes/skills/ .hermes/sessions/ \
  .openclaw/openclaw.json .openclaw/workspace/ .openclaw/memory/ \
  .openclaw/credentials/ .openclaw/identity/ \
  .claude/settings.json .claude/CLAUDE.md .claude/skills/ .claude/projects/ \
  2>/dev/null
```

### Deploy (on target machine)

```bash
tar xzf agent-backup.tar.gz
# Adapt config for new environment (see SKILL.md for details)
```

## Features

- **Cross-architecture support**: x86_64 to ARM (Raspberry Pi, etc.)
- **Lean or full packaging**: Exclude large binaries, or copy everything
- **Security guidance**: Sensitive data checklist, encrypted transfer
- **Troubleshooting**: Common issues and fixes for each framework
- **Cross-framework migration**: Move data between different agent frameworks

## Documentation

Full documentation is in [SKILL.md](./SKILL.md) — including:
- Complete file path mapping for all 3 frameworks
- Architecture compatibility matrix
- Version compatibility notes
- Step-by-step deploy instructions
- Post-deploy verification checklist
- Security and privacy guidelines

## Use as a Hermes Skill

```bash
# Install into your Hermes skills directory
mkdir -p ~/.hermes/skills/devops/agent-clone
curl -o ~/.hermes/skills/devops/agent-clone/SKILL.md \
  https://raw.githubusercontent.com/gegewu81/agent-clone/main/SKILL.md
```

## License

MIT

