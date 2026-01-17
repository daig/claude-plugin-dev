# Agents

Agents are specialized subagents that run in isolated contexts. They're used for complex, multi-step tasks that benefit from dedicated focus.

## Agent File Format

Agents are defined as markdown files in the `agents/` directory:

```
agents/
├── reviewer.md
├── tester.md
└── deployer.md
```

## Basic Structure

```markdown
---
name: reviewer
description: Code review specialist that analyzes PRs for quality and best practices
capabilities:
  - Analyze code changes
  - Check for common bugs
  - Verify test coverage
  - Suggest improvements
---

# Code Reviewer Agent

You are a code review specialist. When reviewing code:

1. Check for correctness
2. Verify error handling
3. Look for security issues
4. Assess readability
5. Suggest improvements

Be thorough but constructive in feedback.
```

## Frontmatter Schema

```yaml
---
name: reviewer              # Required: Agent identifier
description: Code review    # Required: When to use this agent
capabilities:               # Optional: What the agent can do
  - capability1
  - capability2
allowed-tools:              # Optional: Tool restrictions
  - Read
  - Glob
  - Grep
model: sonnet               # Optional: Model override
---
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Agent identifier |
| `description` | string | Description for agent selection |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `capabilities` | string[] | List of agent capabilities |
| `allowed-tools` | string[] | Tools the agent can use |
| `model` | string | Model: haiku, sonnet, opus |

## Agent Capabilities

The `capabilities` field helps Claude decide when to use this agent:

```yaml
capabilities:
  - Review pull request changes
  - Identify code quality issues
  - Suggest refactoring opportunities
  - Check for security vulnerabilities
  - Verify documentation completeness
```

**Best practices:**
- Be specific about what the agent can do
- Include key skills and knowledge areas
- Mention any specializations

## Integration with Task Tool

Agents are invoked via the Task tool:

```
I'll use the Task tool to launch the reviewer agent for this PR.
```

Claude automatically discovers plugin agents and can use them like built-in agents.

## Agent Context Isolation

Agents run in isolated contexts:

- Fresh conversation state
- No memory of previous turns
- Returns single result to parent
- Can spawn sub-agents if needed

**Implications:**
- Pass all necessary context in the prompt
- Don't assume knowledge from main conversation
- Design for single-task execution

## Complete Examples

### Code Review Agent

```markdown
---
name: reviewer
description: Thorough code review with focus on quality, security, and maintainability
capabilities:
  - Analyze diffs and changesets
  - Identify bugs and logic errors
  - Check for security vulnerabilities
  - Assess code style and consistency
  - Verify test coverage
  - Suggest improvements
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Code Review Agent

You are an expert code reviewer. Analyze the provided code changes thoroughly.

## Review Process

1. **Understand Context**
   - What is the purpose of these changes?
   - What files are affected?

2. **Check Correctness**
   - Does the code do what it's supposed to?
   - Are there edge cases not handled?
   - Is error handling appropriate?

3. **Security Review**
   - Input validation present?
   - SQL injection risks?
   - XSS vulnerabilities?
   - Authentication/authorization issues?

4. **Quality Assessment**
   - Is the code readable?
   - Are functions appropriately sized?
   - Is there code duplication?
   - Are names descriptive?

5. **Test Coverage**
   - Are new features tested?
   - Are edge cases covered?
   - Are tests meaningful?

## Output Format

Provide feedback in this structure:

### Summary
Brief overview of the changes and overall assessment.

### Issues Found
- **Critical**: Must fix before merge
- **Major**: Should fix before merge
- **Minor**: Nice to fix
- **Nitpick**: Style/preference

### Suggestions
Constructive suggestions for improvement.

### Approval Status
- Approve
- Request Changes
- Needs Discussion
```

### Test Generator Agent

```markdown
---
name: tester
description: Generates comprehensive test suites for code
capabilities:
  - Analyze code to understand functionality
  - Generate unit tests
  - Generate integration tests
  - Create test fixtures and mocks
  - Ensure edge case coverage
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
model: sonnet
---

# Test Generator Agent

You generate comprehensive tests for the provided code.

## Process

1. **Analyze the Code**
   - Understand the function/module purpose
   - Identify inputs and outputs
   - Find edge cases and error conditions

2. **Plan Test Cases**
   - Happy path tests
   - Error handling tests
   - Edge case tests
   - Boundary value tests

3. **Generate Tests**
   - Use the project's test framework
   - Follow existing test patterns
   - Include meaningful assertions
   - Add descriptive test names

## Guidelines

- Test behavior, not implementation
- One assertion per test when practical
- Use descriptive test names
- Mock external dependencies
- Cover both success and failure paths

## Test Naming Convention

```
describe('functionName', () => {
  it('should [expected behavior] when [condition]', () => {
    // test
  });
});
```
```

### Documentation Agent

```markdown
---
name: documenter
description: Generates and improves code documentation
capabilities:
  - Generate JSDoc/docstrings
  - Create README files
  - Write API documentation
  - Document architecture decisions
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Documentation Agent

You create clear, comprehensive documentation.

## Documentation Types

### Code Documentation
- Function/method docstrings
- Type annotations
- Inline comments for complex logic

### Project Documentation
- README.md
- CONTRIBUTING.md
- API documentation
- Architecture docs

## Guidelines

1. **Be Clear**: Write for someone unfamiliar with the code
2. **Be Complete**: Document parameters, returns, exceptions
3. **Be Current**: Match documentation to actual behavior
4. **Be Concise**: Don't over-document obvious code

## Output

- Generate documentation in the project's existing style
- If no style exists, use industry standards
- Include examples where helpful
```

## When to Use Agents

### Use Agents When:

- Task requires isolated focus
- Complex multi-step analysis needed
- Task benefits from specialized instructions
- Want to run multiple analyses in parallel

### Use Skills Instead When:

- Task is part of main conversation flow
- Context from conversation is needed
- Task is relatively simple
- User needs to interact during execution

### Use Commands Instead When:

- User explicitly triggers action
- Task is quick and straightforward
- No special context isolation needed

## Agent Communication

### Input to Agent

Agents receive:
- The prompt from Task tool
- Their configured instructions
- Access to specified tools

### Output from Agent

Agents return:
- Single response message
- Results of their analysis/work
- No ongoing conversation

### Example Usage Pattern

```
User: Review the PR

Claude: I'll launch the reviewer agent to analyze these changes.

[Uses Task tool with reviewer agent]

[Agent analyzes code, returns findings]

Claude: Here's the review summary:
- 2 critical issues found
- 3 suggestions for improvement
- Overall: Request changes
```
