# MCP Servers

MCP (Model Context Protocol) servers provide Claude with access to external tools and resources. They enable database queries, API integrations, file system access, and other external capabilities.

## Overview

MCP servers run as separate processes that communicate with Claude Code. They provide:
- Custom tools Claude can invoke
- Resource access (databases, APIs, files)
- Persistent connections to external services

## Configuration Format

### In plugin.json

```json
{
  "name": "my-plugin",
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/my-server.js"],
      "env": {
        "API_KEY": "${API_KEY}",
        "DB_URL": "${DB_URL}"
      }
    }
  }
}
```

### In .mcp.json

Create `.claude-plugin/.mcp.json`:

```json
{
  "mcpServers": {
    "database": {
      "command": "node",
      "args": ["./servers/db-server.js"]
    },
    "api": {
      "command": "python",
      "args": ["./servers/api-server.py"]
    }
  }
}
```

## Server Configuration Schema

```json
{
  "server-name": {
    "command": "node",           // Required: Executable
    "args": ["path/to/server"],  // Required: Arguments
    "env": {                     // Optional: Environment variables
      "VAR": "value"
    },
    "transport": "stdio",        // Optional: stdio (default) or http
    "timeout": 30000,            // Optional: Startup timeout (ms)
    "restartOnCrash": true       // Optional: Auto-restart
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `command` | string | Executable to run |
| `args` | string[] | Command arguments |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `env` | object | `{}` | Environment variables |
| `transport` | string | `"stdio"` | Transport type |
| `timeout` | number | `30000` | Startup timeout (ms) |
| `restartOnCrash` | boolean | `false` | Auto-restart on crash |

## Transport Types

### stdio (Default)

Server communicates via stdin/stdout:

```json
{
  "my-server": {
    "command": "node",
    "args": ["server.js"],
    "transport": "stdio"
  }
}
```

### HTTP

Server exposes HTTP endpoint:

```json
{
  "my-server": {
    "transport": "http",
    "url": "http://localhost:3000/mcp"
  }
}
```

### SSE (Server-Sent Events)

Server uses SSE for streaming:

```json
{
  "my-server": {
    "transport": "sse",
    "url": "http://localhost:3000/events"
  }
}
```

## Environment Variables

### Plugin Variables

Use `${CLAUDE_PLUGIN_ROOT}` for paths:

```json
{
  "my-server": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"],
    "env": {
      "CONFIG": "${CLAUDE_PLUGIN_ROOT}/config.json"
    }
  }
}
```

### User Environment Variables

Reference user's environment:

```json
{
  "my-server": {
    "env": {
      "API_KEY": "${API_KEY}",
      "DB_PASSWORD": "${DATABASE_PASSWORD}"
    }
  }
}
```

### Required vs Optional Variables

Document required variables in your README:

```markdown
## Required Environment Variables

- `API_KEY`: Your API key (get from dashboard)
- `DB_URL`: Database connection string
```

## Auto-Start Behavior

MCP servers start automatically when:
1. Plugin is enabled
2. Claude Code session starts
3. Server crashes (if `restartOnCrash: true`)

Servers stop when:
1. Session ends
2. Plugin is disabled
3. Server crashes (if `restartOnCrash: false`)

## Complete Examples

### Database Server

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-postgres", "${DB_URL}"],
      "env": {
        "DB_URL": "${DATABASE_URL}"
      },
      "timeout": 10000
    }
  }
}
```

### API Client Server

```json
{
  "mcpServers": {
    "github-api": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/github-mcp.js"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      },
      "restartOnCrash": true
    }
  }
}
```

### File System Server

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-filesystem", "/allowed/path"],
      "timeout": 5000
    }
  }
}
```

### Multiple Servers

```json
{
  "mcpServers": {
    "database": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/db.js"],
      "env": {
        "DB_URL": "${DATABASE_URL}"
      }
    },
    "cache": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/redis.js"],
      "env": {
        "REDIS_URL": "${REDIS_URL}"
      }
    },
    "search": {
      "command": "python",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/search.py"],
      "env": {
        "ELASTIC_URL": "${ELASTICSEARCH_URL}"
      }
    }
  }
}
```

### External HTTP Server

```json
{
  "mcpServers": {
    "remote-api": {
      "transport": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

## Writing MCP Servers

### Basic Node.js Server

```javascript
// servers/my-server.js
import { Server } from '@anthropic/mcp';

const server = new Server({
  name: 'my-server',
  version: '1.0.0'
});

// Define a tool
server.addTool({
  name: 'my_tool',
  description: 'Does something useful',
  parameters: {
    type: 'object',
    properties: {
      input: { type: 'string', description: 'Input value' }
    },
    required: ['input']
  },
  handler: async ({ input }) => {
    return { result: `Processed: ${input}` };
  }
});

// Start server
server.listen();
```

### Basic Python Server

```python
# servers/my_server.py
from mcp import Server, Tool

server = Server(name='my-server', version='1.0.0')

@server.tool('my_tool', description='Does something useful')
async def my_tool(input: str) -> dict:
    return {'result': f'Processed: {input}'}

if __name__ == '__main__':
    server.run()
```

## Tools Provided by MCP Servers

MCP servers can provide:

### Query Tools

```javascript
server.addTool({
  name: 'query_database',
  description: 'Run a read-only database query',
  parameters: {
    type: 'object',
    properties: {
      sql: { type: 'string', description: 'SQL query' }
    }
  },
  handler: async ({ sql }) => {
    const results = await db.query(sql);
    return { rows: results };
  }
});
```

### Action Tools

```javascript
server.addTool({
  name: 'create_issue',
  description: 'Create a GitHub issue',
  parameters: {
    type: 'object',
    properties: {
      title: { type: 'string' },
      body: { type: 'string' }
    }
  },
  handler: async ({ title, body }) => {
    const issue = await github.createIssue({ title, body });
    return { issue_url: issue.html_url };
  }
});
```

### Resource Tools

```javascript
server.addResource({
  name: 'config',
  description: 'Application configuration',
  handler: async () => {
    return JSON.parse(fs.readFileSync('./config.json'));
  }
});
```

## Best Practices

### 1. Secure Credentials

Never hardcode credentials:

```json
// Good
{ "env": { "API_KEY": "${API_KEY}" } }

// Bad
{ "env": { "API_KEY": "sk-1234..." } }
```

### 2. Handle Startup Failures

Set appropriate timeouts:

```json
{
  "timeout": 15000,  // 15 seconds for slow servers
  "restartOnCrash": true
}
```

### 3. Document Requirements

In your README:

```markdown
## Prerequisites

1. Node.js 18+
2. Environment variables:
   - `DATABASE_URL`: PostgreSQL connection string
   - `API_KEY`: Service API key
```

### 4. Provide Health Checks

In your server:

```javascript
server.addTool({
  name: 'health_check',
  description: 'Check server health',
  handler: async () => ({ status: 'healthy' })
});
```

### 5. Log Appropriately

```javascript
console.error('Server starting...');  // Logs to stderr (visible)
console.log('Debug info');            // Part of MCP protocol
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Server not starting | Wrong command/args | Check paths with `which node` |
| Missing env vars | Variable not set | Export in shell or use .env |
| Connection timeout | Server too slow | Increase timeout |
| Crashes on startup | Missing dependencies | Run `npm install` in server dir |
| Tools not appearing | Server error | Check stderr output |
