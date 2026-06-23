---
name: review-doc
description: Use when a task is complete and needs a human-readable review document — after implementing a function, fixing a bug, or completing any unit of work that another person will review.
---

# Review Doc

Generate a review document for a completed unit of work. Structured as a journalist's pyramid: the reader enters at the top and descends only as deep as needed.

**Announce:** "Using review-doc skill."

## Modes

- **`new`** — a new function or feature was added
- **`fix`** — existing code was changed (bugfix, refactor, patch)

If mode is not provided, infer it from context: if the function did not exist before → `new`; if it existed and was modified → `fix`.

## Output File

Write to `docs/review/YYYY-MM-DD-<feature>.md`. If the file already exists (previous tasks in the same plan), **append** a new section — do not overwrite.

## Document Structure

### Mode: `new`

```markdown
## <Function or Feature Name>

### Helicopter View
**What:** <one sentence — what this is>
**Why:** <one sentence — why it exists, what problem it solves>
**How:** <one sentence — how it works at the highest level>

### General Details
**What:** <2-4 sentences with more context>
**Why:** <2-4 sentences — motivation, constraints, trade-offs>
**How:** <2-4 sentences — key steps, data flow, important decisions>

### Full Details
**`<function_name>(<params>) → <return_type>`**
<What it does, key logic, edge cases, non-obvious decisions. Code snippets only if they clarify something that text cannot.>

[Repeat for each function implemented in this task]
```

### Mode: `fix`

```markdown
## <Function or Feature Name> (fix)

### Helicopter View
**What:** <what was changed>
**Before:** <how it worked previously, one sentence>
**After:** <how it works now, one sentence>

### General Details
**What:** <more detail on the change>
**Before:** <previous behavior with context>
**After:** <new behavior with context, why this is better>

### Full Details
**`<function_name>`**
<What was wrong, what changed, why. Code before/after only if it clarifies something text cannot.>

[Repeat for each changed function]
```

## Rules

- Write for someone who has **no context** — they were not present during implementation.
- Helicopter View: max 3 sentences total. If you need more, the view is not high enough.
- Full Details: be specific. Mention types, return shapes, edge cases handled.
- Never copy-paste the plan task description as the review — derive from the actual code written.
- If a function was a `DEPENDS ON` stub that got replaced with a real implementation, note it explicitly in Full Details.