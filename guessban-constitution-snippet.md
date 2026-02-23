# Guessban – Agent Constitution Snippet

Copy the relevant section into your agent's system prompt or
constitution file (e.g. `.github/copilot-instructions.md`).
Use the Pre-Analyze snippet, the Mid-Impl snippet, or both.

---

## Pre-Analyze Mode

Add this to your agent constitution to enforce Guessban before implementation starts.

---

### Guessban Pre-Analyze Policy

Before calling any implementation tool or generating any code, you MUST run
`/speckit.analyze` and evaluate the output against the Guessban policy defined
in `.guessban.yaml`.

**Rules:**
- Load `.guessban.yaml` from the project root.
- Read the `pre_analyze.stop_on` list.
- If `/speckit.analyze` returns any finding whose trigger ID matches an entry
  in `stop_on`, you MUST stop immediately.
- Do NOT proceed to `/speckit.implement` or generate any code.
- Output a block message in this format:

```
🚫 GUESSBAN BLOCKED [pre_analyze]

Trigger : {trigger_id}
Finding : {finding.description}
Severity: {finding.severity}
Task    : {task.id} – {task.title}

Resolution required before implementation can continue.
Update the spec or resolve the finding, then re-run /speckit.analyze.
```

- If findings match `warn_on` only, output a warning but allow implementation to proceed.
- If no findings match `stop_on` or `warn_on`, output:

```
✅ GUESSBAN PASSED [pre_analyze] – No blocking findings. Proceeding to implementation.
```

---

## Mid-Impl Mode

Add this to your agent constitution to enforce Guessban after each implemented task.

---

### Guessban Mid-Impl Policy

After each task is implemented and before starting the next task, you MUST
evaluate the generated code against the Guessban policy defined in `.guessban.yaml`.

**Rules:**
- Load `.guessban.yaml` from the project root.
- Read the `mid_impl.stop_on` list.
- After each task completes (based on `mid_impl.check_after: task`), scan the
  generated code against the current spec.
- If any new finding matches an entry in `stop_on`, you MUST stop immediately.
- Do NOT start the next task.
- Output a block message in this format:

```
🚫 GUESSBAN BLOCKED [mid_impl]

Trigger : {trigger_id}
Finding : {finding.description}
Severity: {finding.severity}
Task    : {task.id} – {task.title}

Resolution required before the next task can start.
Fix the implementation or update the spec, then re-run this task.
```

- If findings match `warn_on` only, output a warning but allow the next task to start.
- If no findings match `stop_on` or `warn_on`, output:

```
✅ GUESSBAN PASSED [mid_impl] – Task {task.id} clean. Proceeding to next task.
```

---

## Both Modes

To use both modes, include both snippets in your agent constitution.
Pre-Analyze runs first. Mid-Impl runs after each task.
A passing Pre-Analyze does not skip Mid-Impl checks.
