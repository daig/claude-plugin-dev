# LSP Servers

LSP (Language Server Protocol) servers provide code intelligence features like completions, diagnostics, go-to-definition, and hover information.

## Overview

LSP servers enable Claude to:
- Get language-specific completions
- See diagnostic errors and warnings
- Navigate code (go-to-definition, find references)
- Understand type information

## Configuration Format

### In plugin.json

```json
{
  "name": "my-plugin",
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ts": "typescript",
        ".tsx": "typescriptreact",
        ".js": "javascript",
        ".jsx": "javascriptreact"
      }
    }
  }
}
```

### In .lsp.json

Create `.claude-plugin/.lsp.json`:

```json
{
  "lspServers": {
    "python": {
      "command": "pyright-langserver",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".py": "python"
      }
    }
  }
}
```

## Configuration Schema

```json
{
  "server-name": {
    "command": "language-server",    // Required: Server executable
    "args": ["--stdio"],             // Optional: Arguments
    "extensionToLanguage": {         // Required: File extension mapping
      ".ext": "language-id"
    },
    "transport": "stdio",            // Optional: stdio (default)
    "timeout": 30000,                // Optional: Startup timeout
    "restartOnCrash": true           // Optional: Auto-restart
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `command` | string | LSP server executable |
| `extensionToLanguage` | object | Maps file extensions to language IDs |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `args` | string[] | `[]` | Command arguments |
| `transport` | string | `"stdio"` | Communication method |
| `timeout` | number | `30000` | Startup timeout (ms) |
| `restartOnCrash` | boolean | `false` | Auto-restart on crash |
| `env` | object | `{}` | Environment variables |

## Extension to Language Mapping

The `extensionToLanguage` field maps file extensions to LSP language identifiers:

```json
{
  "extensionToLanguage": {
    ".ts": "typescript",
    ".tsx": "typescriptreact",
    ".js": "javascript",
    ".jsx": "javascriptreact",
    ".mjs": "javascript",
    ".cjs": "javascript"
  }
}
```

### Common Language IDs

| Language | ID | Extensions |
|----------|-----|------------|
| TypeScript | `typescript` | `.ts` |
| TSX | `typescriptreact` | `.tsx` |
| JavaScript | `javascript` | `.js`, `.mjs`, `.cjs` |
| JSX | `javascriptreact` | `.jsx` |
| Python | `python` | `.py` |
| Go | `go` | `.go` |
| Rust | `rust` | `.rs` |
| Java | `java` | `.java` |
| C/C++ | `c`, `cpp` | `.c`, `.cpp`, `.h` |
| C# | `csharp` | `.cs` |
| Ruby | `ruby` | `.rb` |
| PHP | `php` | `.php` |

## Common LSP Servers

### TypeScript/JavaScript

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".ts": "typescript",
      ".tsx": "typescriptreact",
      ".js": "javascript",
      ".jsx": "javascriptreact"
    }
  }
}
```

Install: `npm install -g typescript-language-server typescript`

### Python (Pyright)

```json
{
  "python": {
    "command": "pyright-langserver",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".py": "python"
    }
  }
}
```

Install: `npm install -g pyright`

### Python (Pylsp)

```json
{
  "python": {
    "command": "pylsp",
    "extensionToLanguage": {
      ".py": "python"
    }
  }
}
```

Install: `pip install python-lsp-server`

### Go (gopls)

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

Install: `go install golang.org/x/tools/gopls@latest`

### Rust (rust-analyzer)

```json
{
  "rust": {
    "command": "rust-analyzer",
    "extensionToLanguage": {
      ".rs": "rust"
    }
  }
}
```

Install: Via rustup or package manager

### Java (jdtls)

```json
{
  "java": {
    "command": "jdtls",
    "extensionToLanguage": {
      ".java": "java"
    },
    "env": {
      "JAVA_HOME": "${JAVA_HOME}"
    }
  }
}
```

### C/C++ (clangd)

```json
{
  "cpp": {
    "command": "clangd",
    "extensionToLanguage": {
      ".c": "c",
      ".cpp": "cpp",
      ".h": "c",
      ".hpp": "cpp"
    }
  }
}
```

Install: Via package manager or LLVM

### Ruby (Solargraph)

```json
{
  "ruby": {
    "command": "solargraph",
    "args": ["stdio"],
    "extensionToLanguage": {
      ".rb": "ruby"
    }
  }
}
```

Install: `gem install solargraph`

## Multiple Language Servers

Configure multiple servers for different languages:

```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ts": "typescript",
        ".tsx": "typescriptreact"
      }
    },
    "python": {
      "command": "pyright-langserver",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".py": "python"
      }
    },
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go"
      }
    }
  }
}
```

## Features Provided

LSP servers can provide these capabilities:

| Feature | Description |
|---------|-------------|
| Completions | Code suggestions as you type |
| Diagnostics | Errors, warnings, hints |
| Hover | Type info and documentation |
| Go to Definition | Navigate to symbol definition |
| Find References | Find all symbol usages |
| Document Symbols | Outline of file structure |
| Workspace Symbols | Search symbols across project |
| Rename | Rename symbol across files |
| Formatting | Code formatting |
| Code Actions | Quick fixes and refactors |

## Best Practices

### 1. Use Global Installs

Install LSP servers globally for consistent behavior:

```bash
npm install -g typescript-language-server typescript
```

### 2. Handle Missing Servers Gracefully

Document installation requirements:

```markdown
## Prerequisites

Install the TypeScript language server:
```bash
npm install -g typescript-language-server typescript
```
```

### 3. Set Appropriate Timeouts

Slow servers need longer timeouts:

```json
{
  "java": {
    "command": "jdtls",
    "timeout": 60000  // 60 seconds for JVM startup
  }
}
```

### 4. Enable Auto-Restart

For stability:

```json
{
  "typescript": {
    "restartOnCrash": true
  }
}
```

### 5. Use Environment Variables

For configurable paths:

```json
{
  "java": {
    "env": {
      "JAVA_HOME": "${JAVA_HOME}"
    }
  }
}
```

## Troubleshooting

### Server Not Starting

```bash
# Verify server is installed and in PATH
which typescript-language-server

# Test server manually
typescript-language-server --stdio
```

### Wrong Language Detection

Verify `extensionToLanguage` mapping:

```json
{
  "extensionToLanguage": {
    ".ts": "typescript",  // Not "ts" or "TypeScript"
  }
}
```

### No Completions/Diagnostics

1. Check server logs (stderr)
2. Verify project has proper config (tsconfig.json, pyrightconfig.json)
3. Ensure dependencies are installed

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "command not found" | Server not installed | Install server globally |
| No diagnostics | Missing project config | Add tsconfig.json, etc. |
| Slow startup | JVM-based server | Increase timeout |
| Crashes frequently | Memory issues | Restart on crash |
