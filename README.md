# 🚫 Guessban

> Your AI agent is guessing. Guessban stops that.

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Works with SpecKit](https://img.shields.io/badge/works%20with-SpecKit-8A2BE2)](https://github.com/github/spec-kit)
[![YAML](https://img.shields.io/badge/config-.guessban.yaml-green)](examples/)

---

## The Problem

Your AI coding agent hits an ambiguous requirement.
It doesn't ask. It guesses. It ships wrong code.
You review it, rewrite it, lose an hour.

**Repeat.**

---

## The Fix

Drop one file into your repo. Define exactly when your agent must stop and ask
instead of silently hallucinating a solution.

```yaml
# .guessban.yaml
clarification_policy:
  allowed: true
  max_questions: 2
  stop_on:
    - missing_data_schema
    - conflicting_requirements
    - undefined_auth_model
    - no_spec_reference
  default_if_silent: fail_task
```

That's it. No install. No dependencies. No magic.

---

## How It Works

Guessban works as a **SpecKit companion**. Add the policy snippet to your
`constitution.md` once — and every subsequent `/speckit.implement` call
respects your project's clarification rules.

```
Without Guessban           With Guessban
──────────────────         ──────────────────────────────
Agent hits ambiguity  →    Agent hits ambiguity
       ↓                          ↓
  Keeps going           Checks .guessban.yaml
       ↓                          ↓
 Wrong code shipped       stop_on: missing_data_schema
       ↓                          ↓
  You fix it             🛑 BLOCKED: "What's the schema
       ↓                    for the users table?"
 30 min wasted                    ↓
                           You answer in 10 seconds
                                  ↓
                            Correct code shipped ✅
```

---

## Setup (2 minutes)

**Step 1** — Copy `.guessban.yaml` into your project root:

```bash
curl -O https://raw.githubusercontent.com/YOUR_USERNAME/guessban/main/examples/minimal.yaml
mv minimal.yaml .guessban.yaml
```

**Step 2** — Add this block to your `constitution.md`:

```markdown
## Agent Clarification Policy
Before implementing any task, read `.guessban.yaml` in the project root.
If a `stop_on` condition is met: output BLOCKED + reason, do not proceed.
If `allowed: true`: ask max `max_questions` questions before starting.
If no `.guessban.yaml` exists: ask max 2 questions, stop on missing
schema or conflicting requirements.
Never guess silently. A short question beats wrong code.
```

**Step 3** — Run `/speckit.implement` as usual. Done.

---

## Config Options

| Field | Type | Default | Description |
|---|---|---|---|
| `allowed` | boolean | `true` | Agent may ask clarifying questions |
| `max_questions` | integer (1–5) | `2` | Max questions before proceeding |
| `stop_on` | string[] | see below | Conditions that trigger a hard stop |
| `default_if_silent` | enum | `warn` | Behavior if no config found: `fail_task` / `warn` / `proceed` |

**Default `stop_on` triggers:**
- `missing_data_schema`
- `conflicting_requirements`
- `undefined_auth_model`
- `no_spec_reference`
- `missing_api_contract`

---

## Examples

| File | Use Case |
|---|---|
| [`examples/minimal.yaml`](examples/minimal.yaml) | Quick start, sensible defaults |
| [`examples/production.yaml`](examples/production.yaml) | Strict mode, `fail_task` on any ambiguity |
| [`examples/greenfield.yaml`](examples/greenfield.yaml) | Relaxed mode for early exploration |

---

## Why Not Just Use SpecKit's `/clarify`?

SpecKit's built-in clarify mode runs **before** the spec is written.
Guessban runs **during implementation** — where most hallucinations happen.

| | SpecKit `/clarify` | Guessban |
|---|---|---|
| Active in specify step | ✅ | ✅ |
| Active in implement step | ❌ | ✅ |
| Project-specific config | ❌ | ✅ |
| Configurable stop triggers | ❌ | ✅ |
| Hard fail on ambiguity | ❌ | ✅ |
| Versioned in Git | ❌ | ✅ |

---

## Contributing

Ideas, examples for other tools (Cursor, Copilot Chat, Claude Code),
or edge cases you've hit — all welcome. Open an issue or PR.

---

## License

MIT © 2026 riCo
