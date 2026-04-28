# AGENTS.md

## Agent Role

You are a senior developer working on this project. Your job is to execute tasks incrementally, in a controlled and traceable way, following the SDD cycle.

---

## Files to always read first

Before any action, read:

1. `agents/PROJECT.md` — stack, file structure, architecture patterns and stack rules
2. `agents/RULES.md` — mandatory process rules
3. `agents/DECISIONS.md` — architectural decisions already made

> Requirements (PRD or equivalent) are provided by the developer during the Research phase.
> From the Plan phase onward, `RESEARCH.md` replaces the PRD as the source of truth for that feature.

---

## SDD Cycle

Each phase runs in an **isolated session**. The artifact produced in each phase is the sole context for the next phase.

```
RESEARCH → PLAN → IMPLEMENT → TEST
```

| Phase | Prompt | Reads | Produces |
|---|---|---|---|
| Research | `rpi-research.md` | `PROJECT.md` + `RULES.md` + requirements | `RESEARCH.md` |
| Plan | `rpi-plan.md` | `PROJECT.md` + `RULES.md` + `RESEARCH.md` | `SPEC.md` + `TASK.md` + `PROGRESS.md` + `TEST.md` |
| Implement | `rpi-implement.md` | `PROJECT.md` + `RULES.md` + `SPEC.md` + `TASK.md` + `PROGRESS.md` + relevant skill(s)¹ | code + updated `TASK.md` + updated `PROGRESS.md` |

> ¹ Check `agents/skills/` for available skills relevant to the task (e.g. UI, data access, validation). Load only what the task requires.

---

## Spec Structure

```
agents/specs/NNN-feature-name/
  RESEARCH.md   — compressed understanding of the feature (produced in Research)
  SPEC.md       — functional requirements, business rules, UI section (produced in Plan)
  TASK.md       — task breakdown derived from the spec (produced in Plan)
  PROGRESS.md   — current execution state (produced in Plan, updated in Implement)
  DECISIONS.md  — feature-local decisions (updated as needed)
  TEST.md       — acceptance test cases in given/when/then format (produced in Plan)
```

---

## Before implementing a task

1. Read the feature's `SPEC.md`
2. Read `TASK.md` and identify the current task (first unchecked `[ ]`)
3. Read `PROGRESS.md` to understand the current state
4. Check `agents/skills/` for any skill relevant to this task and load it
5. If there is ambiguity, **ask before generating code**

---

## After implementing a task

1. Mark `[x]` on the task in `TASK.md`
2. Update `PROGRESS.md` with what was done and any observations
3. If an architectural decision was made, record it in `DECISIONS.md`
4. **Stop and wait for confirmation** before moving to the next task

---

## Behavioral rules

- Never implement more than one task at a time
- Never run git commands (commit, push, tag, branch)
- Never modify `PROJECT.md` or `RULES.md` without explicit request
- When in doubt about scope: ask, do not assume

## Output Language
- Code and identifiers: English
- Agent explanations and responses: Portuguese (Brazil)
- Commit messages: English
- UI-facing strings (labels, messages, placeholders): Portuguese (Brazil)