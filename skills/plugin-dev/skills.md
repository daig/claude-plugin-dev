# Skills

Skills are knowledge and instructions that Claude can use when helping users. They're the most common plugin component, teaching Claude specialized knowledge, workflows, and best practices.

## SKILL.md File Format

Each skill is defined by a `SKILL.md` file in a dedicated directory:

```
skills/
└── my-skill/
    ├── SKILL.md        # Required: Skill definition
    ├── reference.md    # Optional: Additional documentation
    └── examples/       # Optional: Example files
        └── example.ts
```

## Frontmatter Schema

The SKILL.md file uses YAML frontmatter to configure the skill:

```yaml
---
name: my-skill
description: A clear description of what this skill does
triggers:
  - keyword1
  - keyword2
allowed-tools:
  - Read
  - Glob
  - Grep
model: sonnet
context:
  - reference.md
  - examples/
agent: explore
hooks:
  PreToolUse:
    - matcher: Bash
      type: prompt
      prompt: Verify command safety
user-invocable: true
disable-model-invocation: false
---

# Skill Instructions

Your instructions for Claude go here...
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique identifier for the skill |
| `description` | string | Clear description used for auto-discovery |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `triggers` | string[] | `[]` | Phrases that auto-activate this skill |
| `allowed-tools` | string[] | all | Tools Claude can use (restricts access) |
| `model` | string | default | Model to use: "haiku", "sonnet", "opus" |
| `context` | string[] | `[]` | Additional files to include |
| `agent` | string | none | Run as subagent type |
| `hooks` | object | none | Skill-specific hooks |
| `user-invocable` | boolean | `true` | Can be manually invoked with /skill |
| `disable-model-invocation` | boolean | `false` | Prevent auto-triggering |

## Field Details

### name

The skill identifier. Used for:
- Manual invocation: `/skills/my-skill`
- Internal references
- Conflict detection

```yaml
name: react-patterns
```

**Naming conventions:**
- Use lowercase with hyphens
- Be descriptive but concise
- Avoid generic names like "helper" or "utils"

### description

The description is **critical for auto-discovery**. Claude uses this to decide when to activate the skill.

```yaml
description: Best practices for React component development including hooks, state management, and performance optimization
```

**Writing good descriptions:**
- Be specific about the skill's domain
- Include key technologies/concepts
- Mention primary use cases
- Keep under 200 characters for readability

### triggers

Phrases that automatically activate the skill when the user mentions them:

```yaml
triggers:
  - react component
  - useState
  - useEffect
  - react hooks
```

**Trigger best practices:**
- Use specific, unique phrases
- Avoid overly common words
- Include variations users might use
- Test for conflicts with other skills

### allowed-tools

Restricts which tools Claude can use when this skill is active:

```yaml
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - Write
```

**Available tools:**
- `Read` - Read files
- `Write` - Create/overwrite files
- `Edit` - Edit existing files
- `Glob` - Find files by pattern
- `Grep` - Search file contents
- `Bash` - Run shell commands
- `Task` - Launch subagents
- `WebFetch` - Fetch web content
- `WebSearch` - Search the web
- `NotebookEdit` - Edit Jupyter notebooks
- `AskUserQuestion` - Ask user questions

**Use cases:**
- Read-only skills: `[Read, Glob, Grep]`
- Safe editing: `[Read, Glob, Grep, Edit]`
- Full access: omit the field entirely

### model

Override the default model for this skill:

```yaml
model: opus
```

**Options:**
- `haiku` - Fast, lightweight tasks
- `sonnet` - Balanced (default)
- `opus` - Complex reasoning tasks

### context

Include additional files when the skill is activated:

```yaml
context:
  - reference.md           # Single file
  - examples/              # Entire directory
  - ../shared/common.md    # Relative path
```

**Context files are:**
- Loaded when skill activates
- Available to Claude as reference
- Useful for examples, schemas, templates

### agent

Run the skill as a specific subagent type:

```yaml
agent: explore
```

**Agent types:**
- `explore` - Codebase exploration
- `plan` - Implementation planning
- `general-purpose` - General tasks

### hooks

Define skill-specific hooks:

```yaml
hooks:
  PreToolUse:
    - matcher: Bash
      type: prompt
      prompt: Ensure this command follows our security guidelines
  PostToolUse:
    - matcher: Write
      type: command
      command: ./scripts/lint.sh $FILE
```

See [hooks.md](hooks.md) for complete hook documentation.

### user-invocable

Controls whether users can manually invoke the skill:

```yaml
user-invocable: true   # Can use /skills/my-skill
user-invocable: false  # Only auto-triggered
```

### disable-model-invocation

Prevents auto-triggering (only manual invocation):

```yaml
disable-model-invocation: true  # Must be explicitly invoked
disable-model-invocation: false # Can auto-trigger (default)
```

## Skill Instructions

After the frontmatter, write instructions for Claude:

```markdown
---
name: api-design
description: REST API design best practices
---

# API Design Guidelines

When designing REST APIs, follow these principles:

## URL Structure

- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural nouns: `/users` not `/user`
- Nest for relationships: `/users/{id}/orders`

## HTTP Methods

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET    | Read    | Yes        |
| POST   | Create  | No         |
| PUT    | Replace | Yes        |
| PATCH  | Update  | No         |
| DELETE | Remove  | Yes        |

## Response Codes

- 200: Success
- 201: Created
- 400: Bad request
- 404: Not found
- 500: Server error

## Example Implementation

When creating an endpoint, structure it like:

\`\`\`typescript
router.get('/users/:id', async (req, res) => {
  const user = await db.users.find(req.params.id);
  if (!user) return res.status(404).json({ error: 'Not found' });
  return res.json(user);
});
\`\`\`
```

### Instruction Best Practices

1. **Structure clearly** - Use headers, lists, tables
2. **Include examples** - Show code, inputs, outputs
3. **Be specific** - Avoid vague guidance
4. **Keep focused** - One skill = one domain
5. **Reference files** - Use `context` for large examples

## File Referencing

### Inline References

Reference files within instructions:

```markdown
See the configuration format in `config.schema.json`.

Example usage from `examples/basic.ts`:
```

### Context Loading

Use the `context` field to auto-load files:

```yaml
context:
  - schema.json
  - examples/
```

### Dynamic References

Reference user's project files:

```markdown
Check the user's `package.json` for dependencies.
Look for existing patterns in their `src/` directory.
```

## Progressive Disclosure

Structure skills from simple to complex:

```markdown
---
name: testing
description: Testing best practices
---

# Testing Guide

## Quick Start

Run tests with:
\`\`\`bash
npm test
\`\`\`

## Writing Tests

Basic test structure:
\`\`\`typescript
describe('feature', () => {
  it('should work', () => {
    expect(true).toBe(true);
  });
});
\`\`\`

## Advanced Topics

### Mocking

[Detailed mocking instructions...]

### Integration Tests

[Detailed integration test instructions...]
```

## Complete Example

```markdown
---
name: typescript-strict
description: TypeScript strict mode configuration and best practices for type safety
triggers:
  - typescript strict
  - strict mode
  - type safety
  - tsconfig strict
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - Write
context:
  - tsconfig.example.json
---

# TypeScript Strict Mode

This skill helps configure and work with TypeScript's strict mode options.

## Enabling Strict Mode

Add to `tsconfig.json`:

\`\`\`json
{
  "compilerOptions": {
    "strict": true
  }
}
\`\`\`

## Strict Options Explained

| Option | Purpose |
|--------|---------|
| `strictNullChecks` | Null/undefined are distinct types |
| `strictFunctionTypes` | Stricter function type checking |
| `strictBindCallApply` | Check bind/call/apply |
| `strictPropertyInitialization` | Class properties must be initialized |
| `noImplicitAny` | Error on implicit `any` |
| `noImplicitThis` | Error on implicit `this` |
| `alwaysStrict` | Emit "use strict" |

## Migration Strategy

1. Enable `strict: true`
2. Fix errors file by file
3. Use `// @ts-expect-error` temporarily for complex cases
4. Remove workarounds as you fix root causes

## Common Patterns

### Handling Nullable Values

\`\`\`typescript
// Before (error with strictNullChecks)
function greet(name: string | null) {
  console.log(name.toUpperCase()); // Error!
}

// After
function greet(name: string | null) {
  if (name) {
    console.log(name.toUpperCase());
  }
}
\`\`\`
```
