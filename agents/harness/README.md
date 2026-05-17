# Harness Configurations

This folder contains **optional, harness-specific enforcement configs**.
They are not part of the core SDD/RPI workflow — the workflow is defined in `agents/AGENTS.md`,
`agents/RULES.md`, and `agents/PROJECT.md`.

These files enforce the rules at the harness level (blocking tool calls, requiring approval),
complementing the behavioral rules in `RULES.md`.

---

## Files

| File | Harness | What it does |
|---|---|---|
| `claude-code/settings.json` | Claude Code | Blocks git commands via `permissions.deny` |
| `opencode/opencode.json` | OpenCode | Blocks git commands via `permission.bash` |

---

## How to use

### Claude Code
Copy `claude-code/settings.json` to `.claude/settings.json` in the project root:

```bash
cp agents/harness/claude-code/settings.json .claude/settings.json
```

### OpenCode
Copy `opencode/opencode.json` to `opencode.json` in the project root:

```bash
cp agents/harness/opencode/opencode.json opencode.json
```

---

## Why separate?

The core workflow (`AGENTS.md`, `RULES.md`) works with any harness by relying on the agent's
understanding of the rules. Harness configs add a second enforcement layer — but only for
harnesses that support it. This keeps the workflow agnostic while allowing per-harness hardening.