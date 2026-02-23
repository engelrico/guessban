# Guessban

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

## Quick Start

**1. Add `.guessban.yaml` to your project root:**

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

**2. Add the constitution snippet to your agent:**

Copy the relevant section from `guessban-constitution-snippet.md` into your
agent's system prompt or instructions file
(e.g. `.github/copilot-instructions.md`).

**3. Run your SpecKit workflow as usual:**

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

## Files

| File | Purpose |
|------|---------|
| `.guessban.yaml` | Policy configuration |
| `guessban-definitions.md` | All trigger definitions and severities |
| `guessban-constitution-snippet.md` | Agent instructions for pre_analyze and mid_impl |
| `guessban-workflow.md` | When to use which mode and how to resolve blocks |
| `examples/` | Ready-to-use config examples |

---

## Examples

See the `examples/` folder for ready-to-use configurations:

- `examples/pre_analyze/` – Minimal gate before implementation
- `examples/mid_impl/` – Per-task guard during implementation
- `examples/both/` – Full coverage for brownfield + team projects

---

## License

MIT
