# agents/

This directory contains the SDD (Spec Driven Development) workflow for this project.
The agent uses these files to act as a controlled, traceable pair programmer.

---

## How it works

Every feature follows the **RPI cycle** in isolated agent sessions:

```
RESEARCH → PLAN → IMPLEMENT
```

Each phase produces a single artifact that becomes the sole context for the next phase.
No code is written without a `SPEC.md` and a `TASK.md`.

---

## Entry point

The root `AGENTS.md` (at the project root) is the file auto-loaded by any harness.
It bootstraps the session and references the files below.

---

## Directory map

| Path | Purpose | Read when |
|---|---|---|
| `AGENTS.md` | Full workflow definition and SDD cycle | Every session (via root `AGENTS.md`) |
| `RULES.md` | Mandatory process rules | Every session |
| `PROJECT.md` | Stack, architecture, stack-specific rules | Every session |
| `DECISIONS.md` | Append-only architectural decisions log | Every session |
| `SETUP.md` | Initial setup guide — **delete after setup** | Setup only |
| `harness/` | Optional enforcement configs per harness — see `harness/README.md` | Setup only |
| `prompts/` | Phase and utility prompts — see `prompts/README.md` | Per phase / on demand |
| `skills/` | Stack-specific capability instructions — see `skills/README.md` | Per task, on demand |
| `specs/` | One folder per feature, auto-populated during development | Per feature session |

---

## Starting a new feature

1. Create the spec folder: `agents/specs/NNN-feature-name/`
2. Open a new agent session
3. Instruct the agent: *"Read and follow `agents/prompts/rpi-research.md`"*
4. Provide your requirements (path to a file, inline description, or attached document)
5. Follow the RPI cycle through Plan and Implement

---

## Files to never modify

- `AGENTS.md` (root, project root)
- `agents/AGENTS.md`
- `agents/RULES.md`
- All files inside `agents/prompts/`
- `agents/skills/_template/SKILL.md`

Stack-specific rules belong in `agents/PROJECT.md` under the **Stack Rules** section.