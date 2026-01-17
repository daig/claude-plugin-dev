# Marketplace

This document covers how to publish Claude Code plugins to marketplaces for distribution and discovery.

## Overview

Marketplaces are repositories that list multiple plugins. Users can:
- Browse available plugins
- Install plugins by name
- Get automatic updates

## marketplace.json Schema

Create `.claude-plugin/marketplace.json` for distribution:

```json
{
  "name": "my-marketplace",
  "description": "A collection of useful Claude Code plugins",
  "owner": "my-org",
  "plugins": [
    {
      "name": "plugin-one",
      "description": "First plugin description",
      "source": {
        "source": "github",
        "repo": "my-org/plugin-one"
      }
    },
    {
      "name": "plugin-two",
      "description": "Second plugin description",
      "source": {
        "source": "github",
        "repo": "my-org/plugin-two"
      }
    }
  ]
}
```

### Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Marketplace identifier |
| `description` | string | Yes | Marketplace description |
| `owner` | string | Yes | Owner/organization name |
| `plugins` | array | Yes | List of plugin entries |

### Plugin Entry Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Plugin name |
| `description` | string | Yes | Plugin description |
| `source` | object | Yes | Source configuration |

## Source Formats

**Critical**: The source configuration uses a nested `source` key. This is a common source of errors.

### GitHub Repository

```json
{
  "source": {
    "source": "github",
    "repo": "owner/repo-name"
  }
}
```

### Git URL

```json
{
  "source": {
    "source": "url",
    "url": "https://github.com/owner/repo.git"
  }
}
```

### GitLab

```json
{
  "source": {
    "source": "url",
    "url": "https://gitlab.com/owner/repo.git"
  }
}
```

### Relative Path (Same Repository)

```json
{
  "source": "./plugins/my-plugin"
}
```

## Common Mistakes

### Wrong Source Format

```json
// WRONG - uses "type" instead of "source"
{
  "source": {
    "type": "github",
    "repo": "owner/repo"
  }
}

// CORRECT - uses "source"
{
  "source": {
    "source": "github",
    "repo": "owner/repo"
  }
}
```

### Missing Nested Source

```json
// WRONG - flat structure
{
  "name": "plugin",
  "source": "github",
  "repo": "owner/repo"
}

// CORRECT - nested source object
{
  "name": "plugin",
  "source": {
    "source": "github",
    "repo": "owner/repo"
  }
}
```

## Distribution Methods

### 1. GitHub Marketplace Repository

Create a repository specifically for your marketplace:

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── README.md
```

Users add to settings:

```json
{
  "marketplaces": [
    {
      "source": "github",
      "repo": "my-org/my-marketplace"
    }
  ]
}
```

### 2. Self-Hosted Marketplace

Host marketplace.json on any Git server:

```json
{
  "marketplaces": [
    {
      "source": "url",
      "url": "https://git.example.com/claude-plugins/marketplace.git"
    }
  ]
}
```

### 3. Single Plugin Distribution

For a single plugin, users can install directly:

```json
{
  "plugins": [
    {
      "source": "github",
      "repo": "my-org/my-plugin"
    }
  ]
}
```

### 4. Monorepo with Multiple Plugins

Include multiple plugins in one repository:

```
my-plugins/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── plugin-a/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   └── plugin-b/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
└── README.md
```

marketplace.json:

```json
{
  "name": "my-plugins",
  "plugins": [
    {
      "name": "plugin-a",
      "source": "./plugins/plugin-a"
    },
    {
      "name": "plugin-b",
      "source": "./plugins/plugin-b"
    }
  ]
}
```

## Complete Examples

### Organization Marketplace

```json
{
  "name": "acme-claude-plugins",
  "description": "Official Claude Code plugins from ACME Corporation",
  "owner": "acme-corp",
  "plugins": [
    {
      "name": "acme-deploy",
      "description": "Deploy to ACME Cloud infrastructure",
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-deploy-plugin"
      }
    },
    {
      "name": "acme-db",
      "description": "ACME Database integration with MCP server",
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-db-plugin"
      }
    },
    {
      "name": "acme-testing",
      "description": "ACME testing framework skill",
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-testing-plugin"
      }
    }
  ]
}
```

### Community Marketplace

```json
{
  "name": "community-plugins",
  "description": "Community-contributed Claude Code plugins",
  "owner": "claude-community",
  "plugins": [
    {
      "name": "react-patterns",
      "description": "React development patterns and best practices",
      "source": {
        "source": "github",
        "repo": "contributor1/react-claude-skill"
      }
    },
    {
      "name": "docker-helper",
      "description": "Docker and container management assistance",
      "source": {
        "source": "github",
        "repo": "contributor2/docker-claude-plugin"
      }
    }
  ]
}
```

### Mixed Source Marketplace

```json
{
  "name": "mixed-marketplace",
  "description": "Plugins from various sources",
  "owner": "my-org",
  "plugins": [
    {
      "name": "github-plugin",
      "description": "From GitHub",
      "source": {
        "source": "github",
        "repo": "owner/github-plugin"
      }
    },
    {
      "name": "gitlab-plugin",
      "description": "From GitLab",
      "source": {
        "source": "url",
        "url": "https://gitlab.com/owner/gitlab-plugin.git"
      }
    },
    {
      "name": "local-plugin",
      "description": "In same repository",
      "source": "./plugins/local-plugin"
    }
  ]
}
```

## User Installation

### Adding a Marketplace

Users add marketplaces to their settings:

```json
{
  "marketplaces": [
    {
      "source": "github",
      "repo": "my-org/my-marketplace"
    }
  ]
}
```

### Installing from Marketplace

Once marketplace is added, install plugins by name:

```json
{
  "plugins": ["my-marketplace/plugin-name"]
}
```

### Direct Installation (No Marketplace)

Install plugins directly without marketplace:

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

## Reserved Names

Some names are reserved and cannot be used:

- `claude`
- `anthropic`
- `official`
- Names of built-in skills/commands

## Best Practices

### 1. Clear Naming

```json
{
  "name": "acme-deploy",        // Good: clear, namespaced
  "name": "deploy"             // Bad: too generic
}
```

### 2. Descriptive Entries

```json
{
  "name": "typescript-strict",
  "description": "TypeScript strict mode configuration and migration help"
}
```

### 3. Consistent Structure

Keep all plugins in your marketplace following the same patterns.

### 4. Version Information

Include version in plugin.json (inside each plugin):

```json
{
  "name": "my-plugin",
  "version": "1.2.0"
}
```

### 5. Documentation

Include README.md in:
- Marketplace repository
- Each plugin repository

### 6. Validation

Test your marketplace.json:

```bash
# Validate JSON syntax
cat .claude-plugin/marketplace.json | jq .

# Check source format
jq '.plugins[].source' .claude-plugin/marketplace.json
```

## Troubleshooting

### "Invalid schema: plugins.0.source"

Wrong source format. Fix:

```json
// Change this:
{ "type": "github", "repo": "..." }

// To this:
{ "source": "github", "repo": "..." }
```

### "Plugin not found"

1. Check plugin name matches exactly
2. Verify source repository exists
3. Check marketplace is properly added

### "Failed to fetch marketplace"

1. Verify repository URL is correct
2. Check repository is public (or credentials are configured)
3. Verify network access
