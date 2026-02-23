# Guessban

[![SpecKit Compatible](https://img.shields.io/badge/SpecKit-compatible-blue)](https://github.com/github/spec-kit)
[![Mode: pre_analyze](https://img.shields.io/badge/mode-pre__analyze-orange)](https://github.com/engelrico/guessban)
[![Mode: mid_impl](https://img.shields.io/badge/mode-mid__impl-orange)](https://github.com/engelrico/guessban)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Works with Copilot](https://img.shields.io/badge/works%20with-GitHub%20Copilot-blue)](https://github.com/features/copilot)
[![Works with Cursor](https://img.shields.io/badge/works%20with-Cursor-blue)](https://cursor.com)
[![Works with Claude](https://img.shields.io/badge/works%20with-Claude-blue)](https://claude.ai)

**Guessban** is a policy-based guard for Spec-Driven Development with [SpecKit](https://github.com/github/spec-kit).

It prevents AI agents from implementing code when the spec is not ready,
and catches drift between spec and code during implementation.

---

## The Problem

AI coding agents are fast. Too fast.

`/speckit.analyze` finds problems. But nothing stops the agent from
implementing anyway – ignoring CRITICAL findings, skipping missing schemas,
or generating code that contradicts the spec.

Guessban enforces the stop.

---

## How It Works

Guessban runs as a policy gate at two points in the SpecKit workflow:

```
/speckit.analyze
      ↓
 [GUESSBAN pre_analyze]   ← blocks if spec is not ready
      ↓
/speckit.implement (task)
      ↓
 [GUESSBAN mid_impl]      ← blocks if code drifts from spec
      ↓
 next task...
```

You define which findings are blocking in `.guessban.yaml`.
The agent constitution snippet tells your AI agent to enforce the policy.

---

## Setup: Files to Copy Into Your Project

### Step 1 – Policy config (required)

```bash
curl -o .guessban.yaml \
  https://raw.githubusercontent.com/engelrico/guessban/main/.guessban.yaml
```

Edit `.guessban.yaml` to match your project's needs.

---

### Step 2 – Agent instructions (required)

```bash
curl -o guessban-constitution-snippet.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-constitution-snippet.md
```

Then paste the relevant section into your agent's instructions file:

| Agent | Instructions file |
|-------|------------------|
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules` or `.cursorrules` |
| Claude (Projects) | Project instructions field |
| Windsurf | `.windsurf/instructions.md` |

---

### Step 3 – Trigger definitions (required)

Your agent needs this file to understand what each trigger means.

```bash
curl -o guessban-definitions.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-definitions.md
```

---

### Step 4 – Workflow guide (required)

Explains which mode to use and how to resolve blocks. Keep it in your project
so your team and your agent can reference it.

```bash
curl -o guessban-workflow.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-workflow.md
```

---

### All at once

```bash
curl -o .guessban.yaml \
  https://raw.githubusercontent.com/engelrico/guessban/main/.guessban.yaml

curl -o guessban-constitution-snippet.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-constitution-snippet.md

curl -o guessban-definitions.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-definitions.md

curl -o guessban-workflow.md \
  https://raw.githubusercontent.com/engelrico/guessban/main/guessban-workflow.md
```

---

## Quick Start

**1.** Copy all four files into your project root (see above).

**2.** Edit `.guessban.yaml` to configure your policy:

```yaml
version: "1.0"
mode: pre_analyze

pre_analyze:
  stop_on:
    - missing_data_schema
    - undefined_auth_model
    - conflicting_requirements
    - no_spec_reference
```

**3.** Paste the relevant section from `guessban-constitution-snippet.md`
into your agent's instructions file.

**4.** Run your SpecKit workflow as usual:

```
/speckit.clarify → /speckit.plan → /speckit.tasks → /speckit.analyze
```

Guessban activates automatically when the agent reads the constitution snippet.

---

## Modes

| Mode | When it runs | Blocks |
|------|-------------|--------|
| `pre_analyze` | After `/speckit.analyze`, before `/speckit.implement` | If spec is not ready |
| `mid_impl` | After each implemented task | If code drifts from spec |
| `both` | Both points | Both cases |

See `guessban-workflow.md` for a detailed decision guide.

---

## Triggers

Guessban ships with built-in triggers covering the most common spec quality issues:

**Pre-Analyze:**
- `missing_data_schema` – No DB schema for core entities
- `undefined_auth_model` – Auth mentioned but roles missing
- `conflicting_requirements` – Contradictory requirements
- `no_spec_reference` – Task without spec coverage
- `breaking_change_detected` – Incompatible API change

**Mid-Impl:**
- `breaking_spec_change` – Code contradicts spec
- `new_missing_schema` – Schema dependency surfaced during impl
- `task_spec_deviation` – Task does not implement what spec says
- `security_pattern_violated` – Auth/security pattern ignored

Full definitions and severities: see `guessban-definitions.md`.

---

## Custom Triggers

```yaml
custom_triggers:
  - id: no_i18n_keys
    severity: HIGH
    description: "A UI string was hardcoded instead of using i18n keys."
    match: "hardcoded_string_in_ui"
```

---

## Files in This Repo

| File | Purpose |
|------|---------|
| `.guessban.yaml` | Policy configuration |
| `guessban-definitions.md` | All trigger definitions and severities |
| `guessban-constitution-snippet.md` | Agent instructions for pre_analyze and mid_impl |
| `guessban-workflow.md` | When to use which mode and how to resolve blocks |
| `examples/` | Ready-to-use config examples |

---

## License

MIT
