# SDD Workflow

A structured workflow for AI-assisted development using **Spec Driven Development (SDD)** and the **Research → Plan → Implement (RPI)** methodology.

Designed for agentic IDEs with filesystem access (Claude Code, Antigravity, OpenCode, Codex CLI, and others), where the agent acts as a pair programmer: controlled, traceable, and spec-driven.

---

## Contents

1. [Core Concepts](#1-core-concepts)
2. [Prerequisites](#2-prerequisites)
3. [Directory Structure](#3-directory-structure)
4. [Initial Setup](#4-initial-setup)
5. [The RPI Cycle](#5-the-rpi-cycle)
6. [Skills System](#6-skills-system)
7. [Utility Prompts](#7-utility-prompts)
8. [Behavioral Rules](#8-behavioral-rules)
9. [Best Practices](#9-best-practices)

---

## 1. Core Concepts

### Spec Driven Development (SDD)

Every feature starts with a spec. No code is written without a `SPEC.md` and a `TASK.md`. This prevents scope creep, preserves architectural decisions, and makes AI-assisted development auditable and reproducible.

### Research → Plan → Implement (RPI)

Each feature goes through three isolated phases. Each phase runs in its own agent session and produces a single artifact that becomes the sole context for the next phase.

```
RESEARCH → PLAN → IMPLEMENT
```

| Phase | Input | Output |
|---|---|---|
| Research | Requirements (PRD, description, or existing code) | `RESEARCH.md` |
| Plan | `RESEARCH.md` | `SPEC.md` + `TASK.md` + `PROGRESS.md` + `TEST.md` |
| Implement | `SPEC.md` + `TASK.md` + `PROGRESS.md` | Code + updated `TASK.md` + updated `PROGRESS.md` |

### Why isolated sessions?

Each session starts fresh. Passing only the artifact from the previous phase — instead of the entire conversation history — keeps context lean, reduces token waste, and avoids context rot across long development cycles.

---

## 2. Prerequisites

- An agentic IDE with **filesystem read/write access** (Claude Code, Antigravity, OpenCode, Codex CLI, Cursor, Windsurf, or equivalent)
- The agent must be able to read and write files within the project directory
- No specific language or framework required — the workflow is stack-agnostic

---

## 3. Directory Structure

```
agents/
  AGENTS.md                   ← Agent role and SDD cycle definition
  RULES.md                    ← Mandatory process rules (git, tasks, decisions)
  PROJECT.md                  ← Stack, architecture, and stack-specific rules
  DECISIONS.md                ← Append-only architectural decisions log
  SETUP.md                    ← Initial setup guide (delete after setup)
  prompts/
    rpi-research.md           ← Phase 1: Research prompt
    rpi-plan.md               ← Phase 2: Plan prompt
    rpi-implement.md          ← Phase 3: Implement prompt
    task-create.md            ← Utility: create a task manually
    skill-call.md             ← Utility: invoke a specific skill
    conventional-commit.md    ← Utility: generate a conventional commit message
    pr-template.md            ← Utility: generate a pull request description
  skills/
    [one .md file per capability]
  specs/
    001-feature-name/
      RESEARCH.md
      SPEC.md
      TASK.md
      PROGRESS.md
      DECISIONS.md
      TEST.md
```

An optional `docs/` directory at the project root (outside `agents/`) can be used to
centralize reference documents that serve as input to the workflow:

```
docs/                         ← optional, at project root
  prd.md                      ← product requirements
  roadmap.md                  ← feature prioritization
  architecture.md             ← high-level architectural decisions
  business-rules.md           ← domain-specific rules
  api-contracts/              ← external API references
  rfcs/                       ← architectural proposals
  research/                   ← technical spikes and evaluations
```

These documents are not part of the workflow itself — they are consulted as input
during Research and Plan phases when relevant.

---

### Files read on every session

`AGENTS.md`, `PROJECT.md`, `RULES.md`

> Keep these files dense and scannable. Every token counts.

### Files read per task (on demand)

`SPEC.md`, `TASK.md`, `PROGRESS.md`, and relevant skills from `agents/skills/`

---

## 4. Initial Setup

### Step 1 — Copy the template

Copy **only the `agents/` directory** into the root of your project. No other files from
this repository are required — the `README.md` and any other root-level files stay behind.

### Step 2 — Run the setup

Open a new agent session and instruct the agent to follow the setup file:

> *"Read and follow the instructions in `agents/SETUP.md`"*

The agent will read the file directly from the filesystem and configure the workflow automatically based on what you provide. No need to copy or paste its contents.

The setup file instructs the agent to:

```
Read agents/AGENTS.md, agents/RULES.md, and agents/SETUP.md.

Your job is to configure this project's SDD workflow by producing three outputs:
agents/PROJECT.md, agents/DECISIONS.md, and one skill file per relevant capability
in agents/skills/.

## Input sources (use what is available, in this order of priority)

1. PRD or requirements document — if provided, extract stack, architecture, and rules from it
2. Existing project files — scan the codebase: package.json, pyproject.toml, tsconfig.json,
   README, src/ structure, config files, and any existing documentation
3. Ask me — if neither is available or if critical information is missing, ask before generating

## Output 1 — agents/PROJECT.md

Produce a dense, scannable file. Prefer tables over paragraphs. Include:

- Project name and purpose (1–2 sentences)
- Runtime, framework, and key libraries (actual versions if detectable)
- Persistence layer and access method
- Frontend stack (if any)
- File and module structure (table: path → responsibility)
- Architecture patterns in use (e.g. layered, repository, MVC)
- Stack Rules: rules already enforced in the codebase
  (naming conventions, forbidden patterns, mandatory abstractions, etc.)

## Output 2 — agents/DECISIONS.md

For new projects: leave the header only, no entries yet.
For existing projects: document architectural decisions already present in the codebase.
  Look for: chosen libraries over alternatives, folder organization rationale,
  patterns used consistently, anything that would be a "why did we do it this way" question.
  Format each entry as:
  ### [YYYY-MM-DD] Decision title
  **Context:** why this decision existed
  **Decision:** what was chosen
  **Consequences:** what this implies going forward

## Output 3 — agents/skills/*.md

Identify which capability domains exist in this project and create one skill file per domain.
Common domains: ui-components, data-access, auth, validation, api-integration, error-handling.
Only create skills that are actually relevant to this project — do not create generic placeholders.

Each skill file must follow this structure:

# Skill: [Capability Name]

## When to use this skill
[Specific task types that should load this skill]

## Patterns
[Correct approach with concise code examples from this project's stack]

## Anti-patterns
[What NOT to do, with reason]

## Checklist
- [ ] Items the agent must verify before finishing a task using this skill

## Constraints

- Produce only the three outputs above — no other files
- Do not create specs or implement any feature
- Do not run git commands
- If any critical information is missing and cannot be inferred, ask before generating
- After delivering all files, wait for my review and approval
```

---

### What to provide alongside the setup instruction

| Scenario | What to send |
|---|---|
| New project + PRD | Reference the file path, describe requirements inline, or attach the document |
| Existing project | Just send the instruction — the agent scans the codebase |
| Existing project + PRD | Provide the PRD alongside — the agent combines both sources |

> The agent accepts requirements in any form: a file path (`docs/prd.md`), an inline
> description in the prompt, or an attached document. Use whatever is most convenient.

### Step 3 — Review and delete SETUP.md

After the agent generates the files, review them and confirm:

- [ ] `PROJECT.md` has no `[FILL: ...]` placeholders remaining
- [ ] `DECISIONS.md` reflects existing decisions (existing projects) or is empty (new projects)
- [ ] At least one skill file exists in `agents/skills/`
- [ ] `AGENTS.md` and `RULES.md` are unmodified

Then delete `agents/SETUP.md`.

---

## 5. The RPI Cycle

Each feature follows the same three-phase cycle, always in isolated sessions.

### Phase 1 — Research

**Goal:** understand the scope. Produce `RESEARCH.md`.

1. Create the spec folder: `agents/specs/NNN-feature-name/`
2. Open a new agent session
3. Instruct the agent: *"Read and follow `agents/prompts/rpi-research.md`"*
4. Attach or describe your requirements
5. Review the generated `RESEARCH.md` and approve before proceeding

> If there are open questions in `RESEARCH.md`, answer them before moving to Plan.

### Phase 2 — Plan

**Goal:** define what will be built and how. Produce `SPEC.md`, `TASK.md`, `PROGRESS.md`, and `TEST.md`.

1. Open a new agent session
2. Instruct the agent: *"Read and follow `agents/prompts/rpi-plan.md`"*
3. The agent reads `RESEARCH.md` and produces the plan artifacts
4. Review all files before proceeding

> `TASK.md` is the implementation backlog. Each `[ ]` is one atomic task.

### Phase 3 — Implement

**Goal:** implement one task at a time. Produce code and update `TASK.md` and `PROGRESS.md`.

1. Open a new agent session
2. Instruct the agent: *"Read and follow `agents/prompts/rpi-implement.md`"*
3. The agent identifies the first unchecked `[ ]` in `TASK.md` and implements it
4. Review the output and confirm before the agent moves to the next task

> The agent stops after each task and waits for your confirmation. This is by design.

### Phase transitions

Each phase ends when you review and approve the generated artifacts. To move to the next
phase, open a **new agent session** and reference the spec folder — the artifacts on the
filesystem are the only context the agent needs.

**Research → Plan:**
> *"Read and follow `agents/prompts/rpi-plan.md`. The spec is at `agents/specs/NNN-feature-name/`"*

**Plan → Implement:**
> *"Read and follow `agents/prompts/rpi-implement.md`. The spec is at `agents/specs/NNN-feature-name/`"*

> The spec folder path is the only additional context required. Without it, the agent
> cannot determine which spec is active if more than one exists.

### Spec folder after a complete cycle

```
agents/specs/001-feature-name/
  RESEARCH.md     ← compressed feature understanding
  SPEC.md         ← functional requirements and business rules
  TASK.md         ← task checklist (all [x] when done)
  PROGRESS.md     ← execution history and observations
  DECISIONS.md    ← feature-local architectural decisions
  TEST.md         ← acceptance test cases (given/when/then)
```

---

## 6. Skills System

Skills are focused instruction files the agent loads **only when a task requires that specific capability**. They encode your project's patterns, anti-patterns, and conventions so the agent produces consistent code without needing to re-explain the same context every session.

### How the agent decides which skills to load

Before implementing a task, the agent scans `agents/skills/` and reads the **"When to use this skill"** section of each file. If the section matches the current task, the skill is loaded into context.

> The quality of skill detection depends directly on how specific and accurate the "When to use" section is.

### Forcing a skill for a specific task

Use `agents/prompts/skill-call.md` when you want to guarantee a skill is applied, regardless of auto-detection.

### Skill file structure

```markdown
# Skill: [Capability Name]

## When to use this skill
[Specific task types that trigger this skill]

## Patterns
[Correct approach with code examples]

## Anti-patterns
[What NOT to do, with reason]

## Checklist
- [ ] Items to verify before finishing the task
```

### Common skills to create

| File | Create when... |
|---|---|
| `ui-components.md` | Project has a UI layer |
| `data-access.md` | Project uses a database or persistence layer |
| `auth.md` | Project has authentication or authorization |
| `validation.md` | Project validates user or external input |
| `api-integration.md` | Project calls external APIs |
| `error-handling.md` | Project has a defined error format or logging strategy |

---

## 7. Utility Prompts

These prompts are used manually by the developer — they are not part of the RPI cycle.

| Prompt | When to use |
|---|---|
| `task-create.md` | Add a new task to an existing `TASK.md` without running the full Plan phase |
| `skill-call.md` | Force a specific skill to be loaded for a task |
| `conventional-commit.md` | Generate a conventional commit message after implementation |
| `pr-template.md` | Generate a pull request description from the spec and completed tasks |

### How to use utility prompts

Open a new agent session and instruct the agent to read the prompt file directly:
> *"Read and follow `agents/prompts/conventional-commit.md`"*

Replace the prompt filename with the one you need. Replace any `<PLACEHOLDERS>` mentioned in the file with the relevant values.

---

## 8. Behavioral Rules

These rules are defined in `AGENTS.md` and `RULES.md` and apply to every session:

| Rule | Description |
|---|---|
| R01 | No code without `SPEC.md` and `TASK.md` |
| R02 | One task per session — never implement multiple tasks at once |
| R03 | Update `PROGRESS.md` on every task completion |
| R04 | Mark `[x]` in `TASK.md` when a task is done |
| R05 | Record architectural decisions in `DECISIONS.md` |
| R06 | Never run git commands (commit, push, branch, tag) |
| R07 | Prefer simple and readable code — no unnecessary abstractions |
| R08 | No new dependencies without justification in `DECISIONS.md` |
| R09 | No auto-generated comments or JSDoc unless explicitly requested |
| R11 | No refactoring outside the current task scope |
| R12 | No anticipating future tasks |
| R13 | Follow KISS, DRY, Clean Code, and SOLID |

---

## 9. Best Practices

**Keep PROJECT.md lean**
It is read on every single session. Every unnecessary line costs tokens. Use tables, avoid paragraphs.

**One spec per feature, one task per session**
Resist the urge to bundle. Small, atomic tasks produce more predictable and reviewable output.

**Review before confirming**
The agent stops after each task and waits for your approval. Use this checkpoint — review the diff before confirming the next task.

**Skills over repeated instructions**
If you find yourself explaining the same pattern in multiple sessions, it belongs in a skill file.

**Decisions are permanent context**
Every entry in `DECISIONS.md` prevents the agent from questioning or re-deciding something already settled. Keep it up to date.

**Don't modify AGENTS.md or RULES.md**
Stack-specific rules belong in `PROJECT.md` under the Stack Rules section. Modifying the core files breaks the workflow contract.
