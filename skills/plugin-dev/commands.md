# Commands

Commands are user-invocable actions triggered with `/command`. They provide quick shortcuts for common tasks.

## Command File Format

Commands are defined as markdown files in the `commands/` directory:

```
commands/
├── deploy.md
├── test.md
└── lint.md
```

Each file defines one command with the filename (minus `.md`) as the command name.

## Basic Structure

```markdown
---
name: deploy
description: Deploy the application to an environment
---

Deploy the application to the $1 environment.

Use the following deployment checklist:
1. Run tests
2. Build the application
3. Deploy to $1
4. Verify deployment health
```

## Frontmatter Schema

```yaml
---
name: deploy              # Required: Command name
description: Deploy app   # Required: Shown in /help
allowed-tools:            # Optional: Restrict tools
  - Bash
  - Read
---
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Command identifier (used as `/name`) |
| `description` | string | Brief description shown in `/help` |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `allowed-tools` | string[] | Tools Claude can use |
| `model` | string | Override model (haiku/sonnet/opus) |

## Argument Placeholders

Commands support argument substitution:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `$ARGUMENTS` | All arguments as single string | `/cmd foo bar` → "foo bar" |
| `$1` | First argument | `/cmd foo bar` → "foo" |
| `$2` | Second argument | `/cmd foo bar` → "bar" |
| `$3`, `$4`... | Subsequent arguments | Same pattern |

### Example with Arguments

```markdown
---
name: create
description: Create a new component
---

Create a new $1 component named $2.

Additional options: $ARGUMENTS
```

Usage:
- `/create react Button --typescript`
- `$1` = "react"
- `$2` = "Button"
- `$ARGUMENTS` = "react Button --typescript"

## Namespacing

When installed from a plugin, commands are namespaced:

```
/plugin-name:command
```

For example, a `deploy` command from `my-tools` plugin:

```
/my-tools:deploy production
```

**Note:** If there's no conflict, the short form may also work:

```
/deploy production
```

## Command Visibility

Commands appear in `/help` output:

```
Available commands:
  /deploy      - Deploy the application to an environment
  /test        - Run the test suite
  /lint        - Lint and fix code issues
```

## Complete Examples

### Simple Command

```markdown
---
name: test
description: Run the test suite
---

Run the test suite for this project.

1. Detect the test framework (jest, vitest, pytest, etc.)
2. Run all tests
3. Report results summary
```

### Command with Required Argument

```markdown
---
name: migrate
description: Run database migrations (up/down/status)
---

Run database migration command: $1

Valid commands:
- up: Apply pending migrations
- down: Rollback last migration
- status: Show migration status

If no argument provided, show status.
```

### Command with Multiple Arguments

```markdown
---
name: scaffold
description: Generate boilerplate code
---

Generate a $1 named $2 with the following options:

Type: $1 (component, hook, service, util)
Name: $2
Options: $ARGUMENTS

Follow the project's existing patterns for file structure and naming.
```

### Command with Tool Restrictions

```markdown
---
name: analyze
description: Analyze code without making changes
allowed-tools:
  - Read
  - Glob
  - Grep
---

Analyze the codebase for:
- Code quality issues
- Potential bugs
- Performance concerns
- Security vulnerabilities

Report findings but do not modify any files.
```

### Command with Specific Model

```markdown
---
name: architect
description: Design system architecture
model: opus
---

Design the architecture for: $ARGUMENTS

Consider:
- Scalability requirements
- Technology constraints
- Team expertise
- Timeline

Provide detailed technical recommendations with trade-off analysis.
```

## Best Practices

### 1. Clear Descriptions

```yaml
# Good
description: Deploy to production with zero-downtime rolling update

# Bad
description: Deploy
```

### 2. Validate Arguments

```markdown
---
name: env
description: Switch environment (dev/staging/prod)
---

Switch to the $1 environment.

Valid environments:
- dev
- staging
- prod

If $1 is not one of these, ask for clarification.
```

### 3. Provide Defaults

```markdown
---
name: build
description: Build the project (default: production)
---

Build the project for $1 environment.

If no environment specified, default to production.
```

### 4. Include Examples in Description

```yaml
description: "Create a component (usage: /create ComponentName)"
```

### 5. Keep Commands Focused

Each command should do one thing well. For complex workflows, create multiple commands or use a skill instead.

## Commands vs Skills vs Agents

| Feature | Command | Skill | Agent |
|---------|---------|-------|-------|
| User invokes with | `/command` | `/skill` or auto | Task tool |
| Auto-triggers | No | Yes (with triggers) | No |
| Isolated context | No | No | Yes |
| Best for | Quick actions | Knowledge/workflows | Complex multi-step |

### When to Use Commands

- Quick, specific actions
- User-initiated operations
- Simple workflows with clear inputs
- Shortcuts for common tasks

### When to Use Skills Instead

- Complex instructions
- Auto-triggered behavior
- Multiple related operations
- Reference documentation
