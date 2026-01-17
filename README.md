# Claude Code Plugin Development Skill

A comprehensive skill for learning how to develop Claude Code plugins. Covers all plugin components including skills, commands, agents, hooks, MCP servers, LSP servers, and marketplace publishing.

## Quick Start

Add this plugin to your Claude Code settings:

```json
{
  "plugins": ["./path/to/claude-plugin-dev"]
}
```

Then ask Claude:
- "How do I create a Claude Code plugin?"
- "What's the SKILL.md frontmatter format?"
- "How do I add MCP servers to my plugin?"

## What's Included

This skill teaches you about all 7 Claude Code plugin components:

| Component | Documentation | Purpose |
|-----------|---------------|---------|
| **Skills** | [skills.md](skills/plugin-dev/skills.md) | Agent capabilities Claude can invoke |
| **Commands** | [commands.md](skills/plugin-dev/commands.md) | User slash commands |
| **Agents** | [agents.md](skills/plugin-dev/agents.md) | Specialized subagents |
| **Hooks** | [hooks.md](skills/plugin-dev/hooks.md) | Event handlers |
| **MCP Servers** | [mcp.md](skills/plugin-dev/mcp.md) | External tool integrations |
| **LSP Servers** | [lsp.md](skills/plugin-dev/lsp.md) | Code intelligence |
| **Marketplace** | [marketplace.md](skills/plugin-dev/marketplace.md) | Distribution |

## Documentation Structure

```
skills/plugin-dev/
├── SKILL.md           # Main entry point and quick start
├── structure.md       # Plugin structure and plugin.json schema
├── skills.md          # Writing skills and SKILL.md format
├── commands.md        # Slash commands
├── agents.md          # Subagents
├── hooks.md           # Event hooks
├── mcp.md             # MCP server integration
├── lsp.md             # LSP server integration
├── marketplace.md     # Publishing and distribution
└── troubleshooting.md # Common issues and solutions
```

## Creating Your First Plugin

Minimal plugin structure:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```

**plugin.json:**

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My first Claude Code plugin",
  "skills": ["../skills"]
}
```

**SKILL.md:**

```markdown
---
name: my-skill
description: Does something useful
---

# My Skill

Instructions for Claude go here.
```

## Key Lessons

From real-world experience building plugins:

1. **Source format in marketplace.json**: Use `{"source": "github", ...}` NOT `{"type": "github", ...}`

2. **plugin.json location**: Must be in `.claude-plugin/` directory, not plugin root

3. **Skills location**: Must be at plugin root in `skills/`, NOT inside `.claude-plugin/`

4. **Path references**: In plugin.json, paths are relative to `.claude-plugin/`:
   ```json
   { "skills": ["../skills"] }  // Goes up to plugin root
   ```

## License

MIT
