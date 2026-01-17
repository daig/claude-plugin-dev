# Troubleshooting

Common issues when developing Claude Code plugins and how to resolve them.

## Schema and Configuration Errors

### "Invalid schema: plugins.0.source"

**Symptom**: Error when loading marketplace or installing plugin.

**Cause**: Wrong source format in marketplace.json.

**Fix**:

```json
// WRONG
{
  "source": {
    "type": "github",
    "repo": "owner/repo"
  }
}

// CORRECT
{
  "source": {
    "source": "github",
    "repo": "owner/repo"
  }
}
```

Note the nested `source.source` - it's confusing but required.

### "Plugin not found"

**Symptom**: Plugin doesn't appear after installation.

**Causes and fixes**:

1. **Missing `.claude-plugin/` directory**
   ```bash
   mkdir -p my-plugin/.claude-plugin
   ```

2. **plugin.json in wrong location**
   ```
   # Wrong
   my-plugin/plugin.json

   # Correct
   my-plugin/.claude-plugin/plugin.json
   ```

3. **Invalid JSON syntax**
   ```bash
   cat .claude-plugin/plugin.json | jq .
   ```

### "Invalid plugin.json"

**Symptom**: Plugin fails to load with schema error.

**Common causes**:

1. **Missing required fields**
   ```json
   {
     "name": "my-plugin",      // Required
     "version": "1.0.0",       // Required
     "description": "..."      // Required
   }
   ```

2. **Skills directory in wrong place**
   ```
   # Wrong - inside .claude-plugin
   .claude-plugin/skills/

   # Correct - at plugin root
   my-plugin/skills/
   ```

## Skills Not Loading

### Skill not appearing in /skills

**Causes and fixes**:

1. **SKILL.md not found**
   ```
   # Required structure
   skills/
   └── my-skill/
       └── SKILL.md    # Case-sensitive!
   ```

2. **Skills directory in wrong location**
   ```
   # Wrong
   .claude-plugin/skills/

   # Correct
   my-plugin/skills/     # At plugin root
   ```

3. **Auto-discovery not working**
   - Ensure `skills/` is at plugin root (not inside `.claude-plugin/`)
   - Each skill needs `skills/skill-name/SKILL.md`

### Skill not auto-triggering

**Causes and fixes**:

1. **Missing triggers**
   ```yaml
   ---
   name: my-skill
   triggers:           # Add trigger phrases
     - my keyword
     - another phrase
   ---
   ```

2. **`disable-model-invocation: true`**
   ```yaml
   ---
   disable-model-invocation: false  # Allow auto-trigger
   ---
   ```

3. **Poor description**
   ```yaml
   # Bad - too vague
   description: Helper skill

   # Good - specific
   description: React component development with hooks, state management, and TypeScript
   ```

### Skill context files not loading

**Cause**: Wrong path in context field.

**Fix**: Paths are relative to SKILL.md location:

```yaml
---
context:
  - reference.md          # Same directory as SKILL.md
  - examples/basic.ts     # Subdirectory
  - ../shared/common.md   # Parent directory
---
```

## Hooks Not Triggering

### PreToolUse hook not firing

**Causes and fixes**:

1. **Wrong matcher**
   ```json
   // Wrong - typo in tool name
   { "matcher": "bash" }

   // Correct - exact tool name
   { "matcher": "Bash" }
   ```

2. **Hook in wrong location**
   ```json
   // In plugin.json
   {
     "hooks": {
       "PreToolUse": [...]  // Correct location
     }
   }
   ```

### Hook command not executing

**Causes and fixes**:

1. **Wrong command path**
   ```json
   // Use CLAUDE_PLUGIN_ROOT for plugin paths
   {
     "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
   }
   ```

2. **Script not executable**
   ```bash
   chmod +x scripts/validate.sh
   ```

3. **Missing dependencies**
   ```bash
   # Check script runs manually
   ./scripts/validate.sh
   ```

### Hook fires too many times

**Fix**: Use `once: true` for one-time hooks:

```json
{
  "matcher": "Bash",
  "type": "prompt",
  "prompt": "Acknowledge bash usage",
  "once": true
}
```

## Commands Not Working

### Command not appearing in /help

**Causes**:

1. **Missing frontmatter**
   ```markdown
   ---
   name: my-command
   description: Does something
   ---
   ```

2. **commands/ in wrong location**
   ```
   # Correct
   my-plugin/commands/my-command.md
   ```

3. **Wrong path in plugin.json**
   ```json
   { "commands": ["../commands"] }
   ```

### Arguments not substituting

**Cause**: Wrong placeholder syntax.

**Fix**:

```markdown
# Use $1, $2 or $ARGUMENTS
Deploy to $1 environment
Full args: $ARGUMENTS
```

## MCP Server Issues

### Server not starting

**Causes and fixes**:

1. **Command not found**
   ```bash
   # Verify command exists
   which node
   ```

2. **Wrong path**
   ```json
   {
     "command": "node",
     "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"]
   }
   ```

3. **Missing dependencies**
   ```bash
   cd servers && npm install
   ```

### Server crashes on startup

**Causes**:

1. **Missing environment variables**
   ```bash
   export API_KEY=your-key
   ```

2. **Port already in use**
   ```bash
   lsof -i :3000  # Check port
   ```

3. **Increase timeout**
   ```json
   { "timeout": 60000 }
   ```

### Tools not appearing

**Cause**: Server error during initialization.

**Debug**:

```bash
# Run server manually to see errors
node servers/my-server.js
```

## LSP Server Issues

### No completions/diagnostics

**Causes and fixes**:

1. **Server not installed**
   ```bash
   npm install -g typescript-language-server
   ```

2. **Wrong extensionToLanguage**
   ```json
   {
     "extensionToLanguage": {
       ".ts": "typescript"  // Not "ts" or "TypeScript"
     }
   }
   ```

3. **Missing project config**
   - TypeScript: needs `tsconfig.json`
   - Python: may need `pyrightconfig.json`

## Path Resolution Issues

### Relative paths not working

**Remember**: Skills/commands/agents/hooks are auto-discovered at plugin root. For MCP servers, use `${CLAUDE_PLUGIN_ROOT}`:

```json
{
  "mcpServers": {
    "my-server": {
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"]
    }
  }
}
```

### Environment variable not expanding

**Fix**: Use `${}` syntax:

```json
{
  "env": {
    "API_KEY": "${API_KEY}",              // User's env var
    "CONFIG": "${CLAUDE_PLUGIN_ROOT}/config.json"  // Plugin path
  }
}
```

## Debugging Checklist

### Quick Validation

```bash
# 1. Check directory structure
ls -la my-plugin/.claude-plugin/
ls -la my-plugin/skills/*/

# 2. Validate JSON files
cat my-plugin/.claude-plugin/plugin.json | jq .
cat my-plugin/.claude-plugin/marketplace.json | jq .

# 3. Check SKILL.md exists
find my-plugin -name "SKILL.md"

# 4. Verify scripts are executable
ls -la my-plugin/scripts/
```

### Enable Debug Logging

Add logging hooks to trace execution:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "echo \"$(date): $TOOL\" >> /tmp/claude-debug.log"
      }
    ]
  }
}
```

### Test Components Independently

1. **Test skill**: Add to settings, ask Claude about it
2. **Test command**: Run `/command`
3. **Test hook**: Trigger the matching tool
4. **Test MCP server**: Run server manually
5. **Test LSP**: Open a file of that type

## Getting Help

If you're still stuck:

1. Check the error message carefully
2. Verify file locations match expected paths
3. Test each component independently
4. Check JSON syntax with `jq`
5. Look at working plugin examples
