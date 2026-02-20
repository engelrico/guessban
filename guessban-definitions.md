# Guessban Trigger Definitions

This file defines what each `stop_on` trigger means.
Place it in your project root alongside `.guessban.yaml`.

The agent reads this file to understand when to stop and ask
instead of proceeding with an assumption.

---

## Data & Schema

**`missing_data_schema`**
No database schema, entity model, or data structure is defined for the
entities this task operates on.
> *"Save user preferences"* — but no preferences table or model is specified.

**`missing_api_contract`**
An external API or service is used but no contract, schema, or
documentation is referenced in the spec.
> *"Fetch weather data"* — but no API endpoint or response format is defined.

**`undefined_return_type`**
A function or endpoint has no defined response structure or return type.
> *"Return the result"* — but what shape is the result?

---

## Requirements & Spec

**`conflicting_requirements`**
Two or more requirements in the spec directly contradict each other.
> *"The endpoint must be public"* AND *"All endpoints require authentication."*

**`no_spec_reference`**
The task has no corresponding entry in the spec. The agent is asked to
build something that was never specified.

**`missing_acceptance_criteria`**
No definition of done exists for this task. The agent cannot determine
when the implementation is correct.

**`out_of_scope`**
The task goes beyond what the spec describes. The agent would need to
make architectural decisions that belong to the planning phase.

---

## Security & Auth

**`undefined_auth_model`**
The task involves authentication or authorization but no auth model,
roles, or permissions are defined.
> *"Only admins can delete"* — but no admin role is defined anywhere.

**`pii_without_policy`**
The task handles personally identifiable information (PII) but no
data protection or retention policy is referenced.

---

## Architecture & Side Effects

**`undocumented_side_effects`**
The implementation would trigger actions not mentioned in the spec —
such as sending emails, emitting events, or modifying unrelated data.

**`breaking_change_detected`**
The task would change an existing interface, API contract, or data
structure in a way that affects other parts of the system.

**`undefined_dependency`**
The task requires an external package, service, or module that is
not listed or approved in the spec or constitution.

---

## How to Extend

Add your own triggers in `.guessban.yaml` and define them here.
Keep definitions short: one sentence + one concrete example is enough.
