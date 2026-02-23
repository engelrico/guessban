# Guessban Trigger Definitions

This document defines all triggers recognized by Guessban.
Triggers are referenced in `.guessban.yaml` under `stop_on`, `warn_on`, and `ignore`.

---

## Pre-Analyze Triggers

These triggers are evaluated after `/speckit.analyze` completes,
before `/speckit.implement` is allowed to start.

### `missing_data_schema`
**Severity:** CRITICAL  
**Description:** A core entity referenced in the spec has no corresponding data schema defined.  
**Example:** Spec mentions an `Order` entity but no DB model, type definition, or schema file exists.  
**Fix:** Define the schema before implementation starts.

---

### `undefined_auth_model`
**Severity:** CRITICAL  
**Description:** The spec references authentication or authorization but no roles, permissions, or auth strategy are defined.  
**Example:** Spec says "only admins can delete" but no role model exists.  
**Fix:** Add an auth model to the spec before implementation starts.

---

### `conflicting_requirements`
**Severity:** CRITICAL  
**Description:** Two or more requirements in the spec contradict each other.  
**Example:** One task requires a public API endpoint, another requires all endpoints to be authenticated.  
**Fix:** Resolve the conflict in the spec before implementation starts.

---

### `no_spec_reference`
**Severity:** CRITICAL  
**Description:** A task has no traceable reference to a spec section or acceptance criterion.  
**Example:** Task says "implement search" but no spec describes search behavior, inputs, or expected outputs.  
**Fix:** Add a spec section that covers this task.

---

### `breaking_change_detected`
**Severity:** CRITICAL  
**Description:** The spec or a task introduces a change that is incompatible with an existing contract (API, DB schema, interface).  
**Example:** A field is renamed in the spec but existing consumers depend on the old name.  
**Fix:** Add a migration plan or versioning strategy to the spec.

---

### `missing_error_handling`
**Severity:** HIGH  
**Description:** The spec does not define error cases or failure behavior for a feature.  
**Example:** A payment flow spec has no definition of what happens on timeout or declined card.  
**Fix:** Add error scenarios to the acceptance criteria.

---

### `performance_not_addressed`
**Severity:** HIGH  
**Description:** No performance criteria or non-functional requirements are defined for a latency- or load-sensitive feature.  
**Example:** A search endpoint spec has no mention of response time expectations or pagination.  
**Fix:** Add performance requirements to the spec.

---

### `incomplete_acceptance_criteria`
**Severity:** HIGH  
**Description:** A task has acceptance criteria but they are vague, untestable, or incomplete.  
**Example:** "The UI should look good" is not a testable criterion.  
**Fix:** Rewrite criteria as concrete, verifiable conditions.

---

### `style_inconsistency`
**Severity:** LOW  
**Description:** Naming conventions or formatting in the spec are inconsistent but do not affect correctness.  
**Example:** Mixed use of camelCase and snake_case for field names across spec files.  
**Fix:** Optional – normalize naming for clarity.

---

### `minor_naming_convention`
**Severity:** LOW  
**Description:** A field, entity, or endpoint name deviates from the project's naming convention.  
**Example:** `getUserData` instead of `getUser` per convention.  
**Fix:** Optional – align with convention if consistency matters.

---

## Mid-Impl Triggers

These triggers are evaluated after each task is implemented,
by scanning the generated code against the spec.

---

### `breaking_spec_change`
**Severity:** CRITICAL  
**Description:** The generated code introduces behavior that contradicts the spec as written.  
**Example:** Spec says `POST /orders` returns `201 Created`, but generated code returns `200 OK`.  
**Fix:** Align the implementation with the spec, or update the spec intentionally and re-analyze.

---

### `new_missing_schema`
**Severity:** CRITICAL  
**Description:** A schema dependency surfaced during implementation that was not caught in the pre-analyze phase.  
**Example:** Implementation of `OrderService` requires a `Discount` model that was never defined.  
**Fix:** Define the missing schema and re-run the task.

---

### `task_spec_deviation`
**Severity:** CRITICAL  
**Description:** The implemented task does not match what the spec describes for that task.  
**Example:** Task says "filter by status", but generated code filters by date only.  
**Fix:** Re-implement the task according to spec, or update the spec and re-run analyze.

---

### `security_pattern_violated`
**Severity:** CRITICAL  
**Description:** The generated code ignores or bypasses an auth or security pattern defined in the spec.  
**Example:** Spec requires JWT validation on all endpoints, but a new endpoint was generated without it.  
**Fix:** Apply the required security pattern before continuing.

---

### `partial_spec_coverage`
**Severity:** HIGH  
**Description:** The task was implemented but only covers part of what the spec requires.  
**Example:** Spec defines three filter options; only two were implemented.  
**Fix:** Complete the implementation or split the task and track the remainder explicitly.

---

### `undocumented_side_effect`
**Severity:** HIGH  
**Description:** The implementation introduced changes outside the task scope that are not reflected in the spec.  
**Example:** Implementing `createOrder` also modified `updateInventory` logic without a corresponding task or spec change.  
**Fix:** Document the side effect in the spec or revert the unscoped change.

---

## Severity Reference

| Severity | Default Behavior | Configurable |
|----------|-----------------|--------------|
| CRITICAL | Always blocks | Yes – can downgrade to `warn_on` |
| HIGH | Warns by default | Yes – can upgrade to `stop_on` |
| LOW | Ignored by default | Yes – can upgrade to `warn_on` or `stop_on` |

---

## Adding Custom Triggers

Custom triggers can be defined in `.guessban.yaml` under `custom_triggers`:

```yaml
custom_triggers:
  - id: no_i18n_keys
    severity: HIGH
    description: "A UI string was hardcoded instead of using i18n keys."
    match: "hardcoded_string_in_ui"
