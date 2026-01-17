# Hooks

Hooks are event handlers that run at specific points in Claude's execution. They enable validation, transformation, logging, and other cross-cutting concerns.

## Hook Events

| Event | When It Fires | Use Case |
|-------|---------------|----------|
| `PreToolUse` | Before any tool runs | Validate, transform, or block tool calls |
| `PostToolUse` | After any tool completes | Log, validate output, trigger follow-up |
| `SessionStart` | When session begins | Initialize, set up environment |
| `Stop` | When session ends | Cleanup, save state |
| `Notification` | On system notifications | React to events |

### PreToolUse

Fires before a tool executes. Can:
- Validate parameters
- Transform inputs
- Block execution
- Add context

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "type": "prompt",
      "prompt": "Ensure this command is safe and follows security guidelines"
    }
  ]
}
```

### PostToolUse

Fires after a tool completes. Can:
- Validate outputs
- Log results
- Trigger follow-up actions
- Transform results

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "type": "command",
      "command": "prettier --write $FILE"
    }
  ]
}
```

### SessionStart

Fires when a Claude Code session starts:

```json
{
  "SessionStart": [
    {
      "type": "command",
      "command": "echo 'Session started at $(date)' >> ~/.claude/session.log"
    }
  ]
}
```

### Stop

Fires when the session ends:

```json
{
  "Stop": [
    {
      "type": "command",
      "command": "echo 'Session ended' >> ~/.claude/session.log"
    }
  ]
}
```

## Hook Types

### command

Runs a shell command:

```json
{
  "type": "command",
  "command": "npm run lint -- $FILE",
  "timeout": 30000
}
```

**Variables in commands:**
- `$FILE` - The file path (for file-related tools)
- `$TOOL` - The tool name
- `${CLAUDE_PLUGIN_ROOT}` - Plugin root directory

### prompt

Sends a prompt to Claude for processing:

```json
{
  "type": "prompt",
  "prompt": "Review this change for security issues. If any are found, explain them."
}
```

### agent

Launches a subagent:

```json
{
  "type": "agent",
  "agent": "security-reviewer",
  "prompt": "Review this for security concerns"
}
```

## Matcher Syntax

Matchers determine which tool calls trigger the hook.

### Match Specific Tool

```json
{
  "matcher": "Bash"
}
```

### Match Multiple Tools

```json
{
  "matcher": ["Bash", "Write", "Edit"]
}
```

### Match with Pattern

```json
{
  "matcher": {
    "tool": "Bash",
    "pattern": "rm|delete|drop"
  }
}
```

### Match All Tools

```json
{
  "matcher": "*"
}
```

### Match File Extensions

```json
{
  "matcher": {
    "tool": "Write",
    "extension": [".ts", ".tsx"]
  }
}
```

## Hook Configuration

### In plugin.json

```json
{
  "name": "my-plugin",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "prompt",
        "prompt": "Validate command safety"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "type": "command",
        "command": "prettier --write $FILE"
      }
    ]
  }
}
```

### In hooks.json

Create `.claude-plugin/hooks.json`:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "type": "prompt",
      "prompt": "Check command safety"
    }
  ],
  "PostToolUse": [
    {
      "matcher": "Write",
      "type": "command",
      "command": "eslint --fix $FILE"
    }
  ]
}
```

### In hooks/ Directory

Create `hooks/validate-bash.md`:

```markdown
---
event: PreToolUse
matcher: Bash
type: prompt
---

Before running this bash command, verify:
1. No destructive operations without confirmation
2. No access to sensitive files
3. No network calls to untrusted hosts
```

## Hook Options

### once

Run only on first match:

```json
{
  "matcher": "Bash",
  "type": "prompt",
  "prompt": "Acknowledge bash usage",
  "once": true
}
```

### timeout

Set command timeout (milliseconds):

```json
{
  "type": "command",
  "command": "npm run lint",
  "timeout": 60000
}
```

### continue_on_error

Continue even if hook fails:

```json
{
  "type": "command",
  "command": "optional-check.sh",
  "continue_on_error": true
}
```

## Complete Examples

### Security Validation Hook

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "type": "prompt",
      "prompt": "Review this bash command for security:\n\n1. Check for command injection risks\n2. Verify no sensitive data exposure\n3. Ensure no destructive operations\n\nIf safe, proceed. If not, suggest a safer alternative."
    },
    {
      "matcher": {
        "tool": "Write",
        "extension": [".env", ".key", ".pem"]
      },
      "type": "prompt",
      "prompt": "STOP: This appears to be a sensitive file. Verify this operation is intended."
    }
  ]
}
```

### Auto-Format Hook

```json
{
  "PostToolUse": [
    {
      "matcher": {
        "tool": "Write",
        "extension": [".ts", ".tsx", ".js", ".jsx"]
      },
      "type": "command",
      "command": "prettier --write $FILE"
    },
    {
      "matcher": {
        "tool": "Write",
        "extension": [".py"]
      },
      "type": "command",
      "command": "black $FILE"
    }
  ]
}
```

### Logging Hook

```json
{
  "PreToolUse": [
    {
      "matcher": "*",
      "type": "command",
      "command": "echo \"$(date): $TOOL\" >> ~/.claude/tool-usage.log"
    }
  ]
}
```

### Session Hooks

```json
{
  "SessionStart": [
    {
      "type": "command",
      "command": "${CLAUDE_PLUGIN_ROOT}/scripts/init-session.sh"
    },
    {
      "type": "prompt",
      "prompt": "Session starting. Remind user of project conventions."
    }
  ],
  "Stop": [
    {
      "type": "command",
      "command": "${CLAUDE_PLUGIN_ROOT}/scripts/cleanup-session.sh"
    }
  ]
}
```

### Multi-Stage Validation

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "type": "command",
      "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate-command.sh \"$COMMAND\""
    },
    {
      "matcher": "Bash",
      "type": "prompt",
      "prompt": "Double-check this bash command is appropriate for the task."
    }
  ]
}
```

## Hook Execution Order

1. Plugin hooks run in order defined
2. Multiple plugins: hooks run in plugin load order
3. Multiple hooks for same event: run sequentially
4. Hook failure can block tool execution (unless `continue_on_error`)

## Best Practices

### 1. Keep Hooks Focused

Each hook should do one thing:

```json
// Good: Single responsibility
{ "type": "command", "command": "prettier --write $FILE" }

// Avoid: Multiple responsibilities
{ "type": "command", "command": "prettier --write $FILE && eslint $FILE && git add $FILE" }
```

### 2. Use Appropriate Timeouts

```json
{
  "type": "command",
  "command": "npm run typecheck",
  "timeout": 120000  // 2 minutes for slow operations
}
```

### 3. Handle Failures Gracefully

```json
{
  "type": "command",
  "command": "optional-lint.sh $FILE",
  "continue_on_error": true
}
```

### 4. Be Specific with Matchers

```json
// Good: Specific to relevant files
{
  "matcher": {
    "tool": "Write",
    "extension": [".ts", ".tsx"]
  }
}

// Avoid: Too broad
{ "matcher": "Write" }
```

### 5. Use once for One-Time Checks

```json
{
  "matcher": "Bash",
  "type": "prompt",
  "prompt": "Acknowledge that bash commands will be used",
  "once": true
}
```

## Debugging Hooks

### Check Hook Execution

Add logging to verify hooks run:

```json
{
  "PreToolUse": [
    {
      "matcher": "*",
      "type": "command",
      "command": "echo 'Hook fired for $TOOL' >> /tmp/hook-debug.log"
    }
  ]
}
```

### Test Matchers

Test your matcher logic:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "type": "command",
      "command": "echo 'Bash matcher worked'"
    }
  ]
}
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Hook not firing | Wrong matcher | Check tool name spelling |
| Command fails | Wrong path | Use `${CLAUDE_PLUGIN_ROOT}` |
| Timeout | Slow command | Increase timeout value |
| Blocking execution | No `continue_on_error` | Add flag if hook is optional |
