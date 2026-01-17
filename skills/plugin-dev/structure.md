# Plugin Structure

This document covers the directory layout, `plugin.json` schema, and installation methods for Claude Code plugins.

## Directory Layout

A complete plugin structure:

```
my-plugin/
├── .claude-plugin/           # Required: Configuration directory
│   ├── plugin.json           # Required: Plugin manifest
│   ├── marketplace.json      # Optional: For marketplace distribution
│   ├── hooks.json            # Optional: Hooks configuration
│   ├── .mcp.json             # Optional: MCP server configuration
│   └── .lsp.json             # Optional: LSP server configuration
│
├── skills/                   # Skills directory (at plugin root)
│   ├── skill-one/
│   │   ├── SKILL.md          # Skill entry point
│   │   └── reference.md      # Additional skill documentation
│   └── skill-two/
│       └── SKILL.md
│
├── commands/                 # Commands directory
│   ├── deploy.md
│   └── test.md
│
├── agents/                   # Agents directory
│   ├── reviewer.md
│   └── tester.md
│
├── hooks/                    # Hooks directory (alternative to hooks.json)
│   └── validate.md
│
├── outputStyles/             # Output styles directory
│   └── concise.md
│
└── README.md                 # Plugin documentation
```

### Critical Layout Rules

1. **`.claude-plugin/` must exist** - This directory identifies a folder as a Claude Code plugin
2. **`plugin.json` must be inside `.claude-plugin/`** - Not at the plugin root
3. **Component directories at root** - `skills/`, `commands/`, `agents/`, `hooks/` must be at the plugin root, NOT inside `.claude-plugin/`
4. **SKILL.md naming** - Each skill folder must contain a `SKILL.md` file (case-sensitive)

## plugin.json Schema

The `plugin.json` file is the plugin manifest. It defines metadata and component locations.

### Complete Schema

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "A description of what the plugin does",
  "author": "Your Name",
  "license": "MIT",
  "homepage": "https://github.com/you/my-plugin",
  "repository": "https://github.com/you/my-plugin",
  "bugs": "https://github.com/you/my-plugin/issues",

  "skills": ["../skills"],
  "commands": ["../commands"],
  "agents": ["../agents"],
  "hooks": ["../hooks"],
  "outputStyles": ["../outputStyles"],

  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/mcp-server.js"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  },

  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ts": "typescript",
        ".tsx": "typescriptreact"
      }
    }
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "prompt",
        "prompt": "Validate command safety"
      }
    ]
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Plugin identifier (lowercase, hyphens allowed) |
| `version` | string | Semantic version (e.g., "1.0.0") |
| `description` | string | Brief description of the plugin |

### Optional Metadata Fields

| Field | Type | Description |
|-------|------|-------------|
| `author` | string | Plugin author name |
| `license` | string | License identifier (e.g., "MIT", "Apache-2.0") |
| `homepage` | string | URL to plugin homepage |
| `repository` | string | URL to source repository |
| `bugs` | string | URL for bug reports |

### Component Path Fields

All paths are **relative to the `.claude-plugin/` directory**.

| Field | Type | Description |
|-------|------|-------------|
| `skills` | string[] | Paths to skill directories |
| `commands` | string[] | Paths to command directories |
| `agents` | string[] | Paths to agent directories |
| `hooks` | string[] | Paths to hook directories |
| `outputStyles` | string[] | Paths to output style directories |

**Path examples:**

```json
{
  "skills": ["../skills"],           // Plugin root's skills/ directory
  "skills": ["../skills/subset"],    // Specific subdirectory
  "skills": ["../skills", "../more-skills"]  // Multiple directories
}
```

### Inline Configuration Fields

| Field | Type | Description |
|-------|------|-------------|
| `mcpServers` | object | MCP server configurations (see mcp.md) |
| `lspServers` | object | LSP server configurations (see lsp.md) |
| `hooks` | object | Hook configurations (see hooks.md) |

## Environment Variables

### Available Variables

| Variable | Description |
|----------|-------------|
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to the plugin root directory |

### Usage in plugin.json

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

### Usage in Hook Commands

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
      }
    ]
  }
}
```

## Installation Methods

### 1. Local Plugin (Development)

Add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["./path/to/my-plugin"]
}
```

Or user-wide in `~/.claude/settings.json`:

```json
{
  "plugins": ["/absolute/path/to/my-plugin"]
}
```

### 2. From Marketplace

Add marketplace and enable plugin:

```json
{
  "marketplaces": [
    {
      "source": "github",
      "repo": "owner/marketplace-repo"
    }
  ],
  "plugins": ["marketplace-name/plugin-name"]
}
```

### 3. Direct from GitHub

```json
{
  "plugins": [
    {
      "source": "github",
      "repo": "owner/plugin-repo"
    }
  ]
}
```

### 4. Direct from Git URL

```json
{
  "plugins": [
    {
      "source": "url",
      "url": "https://gitlab.com/owner/plugin-repo.git"
    }
  ]
}
```

## Installation Scopes

| Scope | Location | Use Case |
|-------|----------|----------|
| Project | `.claude/settings.json` | Project-specific plugins |
| User | `~/.claude/settings.json` | Personal plugins across all projects |
| System | System config location | Organization-wide plugins |

### Scope Precedence

Project settings override user settings, which override system settings. Plugins from all scopes are loaded (not replaced).

## Validation

### Check Plugin Structure

```bash
# Verify .claude-plugin directory exists
ls -la my-plugin/.claude-plugin/

# Verify plugin.json is valid JSON
cat my-plugin/.claude-plugin/plugin.json | jq .

# Verify skills directory structure
find my-plugin/skills -name "SKILL.md"
```

### Common Validation Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Plugin not found" | Missing `.claude-plugin/` directory | Create the directory |
| "Invalid plugin.json" | Malformed JSON | Check JSON syntax |
| "Skills not loading" | Wrong path in `skills` field | Use `["../skills"]` not `["./skills"]` |

## Minimal Examples

### Skill-Only Plugin

```
minimal-skill/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```

```json
{
  "name": "minimal-skill",
  "version": "1.0.0",
  "description": "A minimal skill plugin",
  "skills": ["../skills"]
}
```

### Command-Only Plugin

```
minimal-command/
├── .claude-plugin/
│   └── plugin.json
└── commands/
    └── hello.md
```

```json
{
  "name": "minimal-command",
  "version": "1.0.0",
  "description": "A minimal command plugin",
  "commands": ["../commands"]
}
```

### MCP Server Plugin

```
mcp-plugin/
├── .claude-plugin/
│   └── plugin.json
└── servers/
    └── my-server.js
```

```json
{
  "name": "mcp-plugin",
  "version": "1.0.0",
  "description": "An MCP server plugin",
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/my-server.js"]
    }
  }
}
```
