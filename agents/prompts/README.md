# agents/prompts/

Prompt files for the SDD workflow. Each file contains instructions the agent follows
when invoked for a specific phase or utility task.

**How to use:** open a new agent session and instruct it to read the file directly.
> *"Read and follow `agents/prompts/rpi-research.md`"*

Do not modify these files. They are intentionally stack-agnostic.

---

## RPI Cycle Prompts

Used in sequence, one per isolated session, to develop a feature end-to-end.

| File | Phase | Produces |
|---|---|---|
| `rpi-research.md` | Research | `RESEARCH.md` |
| `rpi-plan.md` | Plan | `SPEC.md` + `TASK.md` + `PROGRESS.md` + `TEST.md` |
| `rpi-implement.md` | Implement | Code + updated `TASK.md` + updated `PROGRESS.md` |

---

## Utility Prompts

Used on demand by the developer, outside the RPI cycle.

| File | When to use |
|---|---|
| `task-create.md` | Add a task to an existing `TASK.md` without running the full Plan phase |
| `skill-call.md` | Force a specific skill to be loaded for the current task |
| `conventional-commit.md` | Generate a conventional commit message after implementation |
| `pr-template.md` | Generate a pull request description from the spec and completed tasks |