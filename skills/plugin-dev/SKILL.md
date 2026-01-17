---
name: plugin-dev
description: Guide to developing Claude Code plugins - covers skills, commands, agents, hooks, MCP servers, LSP servers, and marketplace publishing
triggers:
  - plugin
  - skill
  - marketplace
  - mcp server
  - lsp server
  - claude code plugin
  - create plugin
  - write plugin
  - hook
  - subagent
---

# Claude Code Plugin Development Guide

This skill teaches you how to create Claude Code plugins. Plugins extend Claude Code with custom skills, commands, agents, hooks, and integrations.

## Quick Start: Create a Minimal Plugin

Create this structure to get started:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```

**Minimal plugin.json:**

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My first Claude Code plugin",
  "skills": ["../skills"]
}
```

**Minimal SKILL.md:**

```markdown
---
name: my-skill
description: Does something useful
---

# My Skill

Instructions for Claude go here.
```

Install locally: Add the plugin path to your `.claude/settings.json`:

```json
{
  "plugins": ["./path/to/my-plugin"]
}
```

## Plugin Components Overview

| Component | Location | Purpose | When to Use |
|-----------|----------|---------|-------------|
| **Skills** | `skills/` | Agent capabilities Claude can invoke | Teaching Claude specialized knowledge or workflows |
| **Commands** | `commands/` | User slash commands | Quick actions users invoke with `/command` |
| **Agents** | `agents/` | Specialized subagents | Complex isolated tasks needing their own context |
| **Hooks** | `hooks/` or inline | Event handlers | Pre/post processing of tool calls, session events |
| **MCP Servers** | `.mcp.json` or inline | External tool integrations | Connecting databases, APIs, external services |
| **LSP Servers** | `.lsp.json` or inline | Code intelligence | Language-specific completions, diagnostics |
| **Output Styles** | `outputStyles/` | Custom formatting | Changing how Claude formats responses |

## Decision Tree: Which Component Do I Need?

```
Do you want to...

├─ Add knowledge/instructions for Claude? → SKILL
│   └─ Should it auto-trigger on certain phrases? → Add triggers to SKILL.md frontmatter
│
├─ Let users invoke an action with /command? → COMMAND
│
├─ Run isolated multi-step tasks? → AGENT
│
├─ React to events (tool calls, session start)? → HOOK
│   ├─ Before a tool runs? → PreToolUse hook
│   ├─ After a tool runs? → PostToolUse hook
│   └─ At session start/stop? → SessionStart/Stop hook
│
├─ Connect to external services? → MCP SERVER
│   ├─ Database queries? → MCP Server with DB tools
│   ├─ API integration? → MCP Server with API tools
│   └─ File system access? → MCP Server with FS tools
│
└─ Add code intelligence? → LSP SERVER
    └─ Completions, diagnostics, go-to-definition? → Configure LSP
```

## Core Concepts

### Plugin Root vs .claude-plugin Directory

```
my-plugin/                  ← Plugin root (where skills/, commands/ live)
├── .claude-plugin/         ← Configuration directory
│   ├── plugin.json         ← Required: plugin manifest
│   └── marketplace.json    ← Optional: for distribution
├── skills/                 ← Skills directory (at root, NOT in .claude-plugin)
├── commands/               ← Commands directory
├── agents/                 ← Agents directory
└── hooks/                  ← Hooks directory
```

**Critical**: Skills, commands, agents, and hooks directories must be at the plugin root, NOT inside `.claude-plugin/`.

### Path Configuration in plugin.json

Paths in `plugin.json` are relative to the `.claude-plugin/` directory:

```json
{
  "skills": ["../skills"],      // Goes up to plugin root, then into skills/
  "commands": ["../commands"],
  "agents": ["../agents"],
  "hooks": ["../hooks"]
}
```

### Environment Variables

- `${CLAUDE_PLUGIN_ROOT}` - Absolute path to the plugin root directory
- Useful in hook commands and MCP server configurations

## Detailed Documentation

For comprehensive information on each component:

- **[Plugin Structure](structure.md)** - Directory layout, plugin.json schema, installation
- **[Skills](skills.md)** - SKILL.md format, frontmatter options, auto-discovery
- **[Commands](commands.md)** - Slash command format, arguments, namespacing
- **[Agents](agents.md)** - Subagent configuration, capabilities, when to use
- **[Hooks](hooks.md)** - All hook events, matchers, configuration
- **[MCP Servers](mcp.md)** - External integrations, transport types, examples
- **[LSP Servers](lsp.md)** - Code intelligence configuration
- **[Marketplace](marketplace.md)** - Publishing, distribution, marketplace.json
- **[Troubleshooting](troubleshooting.md)** - Common issues and solutions

## Common Patterns

### Skill with Tool Restrictions

```yaml
---
name: safe-skill
description: A skill that only uses read operations
allowed-tools:
  - Read
  - Glob
  - Grep
---
```

### Skill with Auto-trigger

```yaml
---
name: react-expert
description: React development best practices
triggers:
  - react
  - jsx
  - component
---
```

### Command with Arguments

```markdown
---
name: deploy
description: Deploy to environment
---

Deploy to $1 environment with options: $ARGUMENTS
```

### Hook for Validation

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "prompt",
        "prompt": "Validate this bash command is safe"
      }
    ]
  }
}
```

## Best Practices

1. **Clear descriptions** - Write descriptions that help Claude understand when to use your skill
2. **Specific triggers** - Use unique trigger phrases to avoid conflicts with other skills
3. **Minimal permissions** - Use `allowed-tools` to restrict skills to only needed tools
4. **Test locally first** - Install as a local plugin before publishing
5. **Document well** - Include examples and edge cases in your skill instructions

## Example: Complete Plugin

See the structure of this plugin itself for a complete working example:

```
claude-plugin-dev/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
└── skills/
    └── plugin-dev/
        ├── SKILL.md          # This file
        ├── structure.md
        ├── skills.md
        ├── commands.md
        ├── agents.md
        ├── hooks.md
        ├── mcp.md
        ├── lsp.md
        ├── marketplace.md
        └── troubleshooting.md
```
