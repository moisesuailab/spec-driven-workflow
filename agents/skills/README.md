# skills/

This directory contains stack-specific reusable instructions for the agent.

Each `.md` file in this directory is a **skill** — a focused guide the agent loads
only when a task requires that specific capability.

---

## How to create a skill

Create one file per capability. The agent will load it only when relevant.

### Recommended skill files for most projects

| File | Purpose |
|---|---|
| `ui-components.md` | Patterns for creating UI components (framework-specific) |
| `data-access.md` | ORM/query patterns, naming conventions, anti-patterns |
| `auth.md` | Authentication and authorization patterns |
| `validation.md` | Input validation approach and reusable validators |
| `api-integration.md` | Patterns for calling external APIs |
| `error-handling.md` | Error classes, response format, logging conventions |

---

## Skill file structure (recommended)

```markdown
# Skill: [Capability Name]

## When to use this skill
[Describe which types of tasks should load this skill]

## Patterns
[Code examples of the correct approach]

## Anti-patterns
[Code examples of what NOT to do, with reason]

## Checklist
- [ ] Item agents must verify before finishing the task
```
