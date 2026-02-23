# Guessban Workflow

This document explains when to use which Guessban mode and how
the two modes fit into a Spec-Driven Development workflow with SpecKit.

---

## Overview

```
/speckit.clarify
      ↓
/speckit.plan
      ↓
/speckit.tasks
      ↓
/speckit.analyze
      ↓
 [GUESSBAN pre_analyze] ← blocks here if policy violated
      ↓
/speckit.implement (task 1)
      ↓
 [GUESSBAN mid_impl] ← blocks here if policy violated
      ↓
/speckit.implement (task 2)
      ↓
 [GUESSBAN mid_impl]
      ↓
      ...
```

---

## Mode: `pre_analyze`

**When to use:**
- You want a hard gate before any code is generated
- You are working in a team and need consistent enforcement of spec quality
- You want to catch structural problems (missing schemas, auth gaps) early

**How it works:**
1. `/speckit.analyze` runs and produces a findings report
2. Guessban reads `.guessban.yaml` and checks `pre_analyze.stop_on`
3. If any finding matches → implementation is blocked
4. Developer fixes the spec or finding, then re-runs `/speckit.analyze`
5. If no blocking findings remain → implementation proceeds

**Best for:** Teams, greenfield projects, critical production services

---

## Mode: `mid_impl`

**When to use:**
- You want to catch issues that only surface during code generation
- You are working in a brownfield codebase with incomplete spec coverage
- You want per-task safety checks without a full pre-flight gate

**How it works:**
1. `/speckit.implement` runs for a single task
2. Guessban scans the generated code against the spec
3. If a new violation is found → next task is blocked
4. Developer fixes the implementation or updates the spec
5. If clean → next task starts

**Best for:** Solo developers, brownfield projects, iterative workflows

---

## Mode: `both`

**When to use:**
- Maximum coverage: catch spec problems before AND during implementation
- Brownfield projects where pre-analyze alone is insufficient
- High-stakes projects where rework is expensive

**How it works:**
- Pre-Analyze gate runs first → blocks if spec is not ready
- Mid-Impl gate runs after each task → blocks if code drifts from spec
- Both gates must pass for the workflow to complete cleanly

**Best for:** Brownfield + team combination, regulated environments, complex specs

---

## Choosing a Mode

| Situation | Recommended Mode |
|-----------|-----------------|
| Solo, greenfield, disciplined | `pre_analyze` |
| Solo, brownfield, iterative | `mid_impl` |
| Team, greenfield | `pre_analyze` |
| Team, brownfield | `both` |
| High-stakes production service | `both` |
| Prototyping / low risk | `pre_analyze` or none |

---

## Resolving a Block

### Pre-Analyze Block
1. Read the block message – note the `trigger_id` and `finding.description`
2. Open the relevant spec file
3. Fix the issue (add schema, resolve conflict, add spec reference)
4. Re-run `/speckit.analyze`
5. If Guessban passes → proceed to `/speckit.implement`

### Mid-Impl Block
1. Read the block message – note which task was blocked and why
2. Either:
   - Fix the generated code to match the spec, or
   - Update the spec to reflect the intended implementation, then re-run `/speckit.analyze`
3. Re-run the blocked task
4. If Guessban passes → proceed to the next task

---

## Report Files

When `output.save_report: true` is set in `.guessban.yaml`, Guessban saves
a report after each run:

```
.guessban/
  reports/
    2026-02-23-pre_analyze.md
    2026-02-23-mid_impl-task-03.md
```

Reports include all findings, their severities, and whether the run was blocked or passed.
