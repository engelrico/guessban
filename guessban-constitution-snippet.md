# Guessban — Constitution Snippet

Copy the block below into your project's `constitution.md`.
This tells the agent how to read and apply your `.guessban.yaml` policy.

---

## Agent Clarification Policy (Guessban)

Before starting any implementation task:

1. Read `.guessban.yaml` in the project root.
2. Read `guessban-definitions.md` for the meaning of each `stop_on` trigger.
3. Check if any `stop_on` condition applies to the current task.

**If a `stop_on` condition is met:**
Ask up to `max_questions` clarifying questions before proceeding.
Wait for answers before writing any code.

**After reaching `max_questions`, apply `default_if_silent`:**

- `proceed` — continue implementation, make internal assumptions silently
- `warn` — continue implementation, but first output an explicit list of every assumption you are making and why
- `fail_task` — stop completely, do not write any code.
  Output the message "🙋 GUESSBAN: Cannot proceed without clarification."
  followed by a numbered list of every unresolved question.

**If no `.guessban.yaml` is found:**
Apply default behavior: max 2 questions, stop on `missing_data_schema`
or `conflicting_requirements`, then `warn`.

Never guess silently. A short question beats wrong code.
