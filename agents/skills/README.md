# skills/

This directory contains stack-specific reusable instructions for the agent.

Each subfolder contains a **`SKILL.md`** — a focused guide the agent loads
only when a task requires that specific capability.

---

## Structure

```
agents/skills/
  _template/
    SKILL.md          ← Template for new skills (copy to start a new one)
  ui-components/
    SKILL.md
  data-access/
    SKILL.md
  auth/
    SKILL.md
  ...
```

---

## How to create a skill

1. Copy `_template/SKILL.md` into a new folder named after the capability
2. Fill in all sections — especially `## When to use this skill`
3. The agent scans skills with: `grep -A 8 "## When to use" agents/skills/*/SKILL.md`

> The "When to use this skill" section is what the agent reads to decide whether to load the full file. Be specific — vague descriptions reduce detection accuracy.

---

## Recommended skills for most projects

| Folder | Purpose |
|---|---|
| `ui-components/` | Patterns for creating UI components (framework-specific) |
| `data-access/` | ORM/query patterns, naming conventions, anti-patterns |
| `auth/` | Authentication and authorization patterns |
| `validation/` | Input validation approach and reusable validators |
| `api-integration/` | Patterns for calling external APIs |
| `error-handling/` | Error classes, response format, logging conventions |

---

## SKILL.md structure

```markdown
# Skill: [Capability Name]

## When to use this skill
[Describe which types of tasks should load this skill — be specific]

## Patterns
[Code examples of the correct approach]

## Anti-patterns
[Code examples of what NOT to do, with reason]

## Checklist
- [ ] Item agents must verify before finishing the task

## Constraints
- [Hard rule specific to this capability]
```