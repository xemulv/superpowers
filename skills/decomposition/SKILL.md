---
name: decomposition
description: Use when decomposing a skeleton function — reads or writes the function body, creates stubs for all called functions, and updates the decomposition tree document.
---

# Decomposition

Use when you need to decompose one function into a skeleton with child stubs.
Can be called standalone or from within `superpowers:walking-skeleton`.

**Announce:** "Using decomposition skill for `<function name>`."

## Step 1: Get the skeleton function

Check if the function already exists in the codebase.

- If it **exists**: read it. Use it as-is — do not modify.
- If it **does not exist**: write it now. Real logic, real calls, real return values. This is NOT a stub — it is the orchestration code.

Example of a skeleton function (Python):

```python
def handle_get_user(user_id: int) -> Response:
    user = fetch_user(user_id)
    perms = get_permissions(user)
    return format_response(user, perms)
```

## Step 2: Identify called functions

List every function the skeleton calls that does not yet have a real implementation (only stubs or does not exist at all).

## Step 3: Write stubs

For each called function, create a stub that returns hardcoded data of the correct type. Place stubs in the same file as the skeleton, or in the appropriate module file per the project's conventions.

The hardcoded data must match the type the skeleton expects. Match field names and structure to what the skeleton code actually uses.

Example stubs:

```python
def fetch_user(user_id: int) -> dict:
    return {"id": user_id, "name": "stub", "email": "stub@example.com"}

def get_permissions(user: dict) -> dict:
    return {"roles": ["admin"]}

def format_response(user: dict, perms: dict) -> dict:
    return {"status": 200, "body": {"user": user, "perms": perms}}
```

## Step 4: Update the decomposition document

Find or create `docs/superpowers/decompositions/YYYY-MM-DD-<feature>.md`.

Add a section for this function. Use the literal text `[SKELETON]` and `[MUSCLE]`:

```markdown
### handle_get_user [SKELETON]
- fetch_user [MUSCLE]
- get_permissions [MUSCLE]
- format_response [MUSCLE]
```

If a child was already listed as `[MUSCLE]` under another parent in the document, update that parent's entry to `[SKELETON]`.

Rules:
- All children start as `[MUSCLE]` when first listed.
- A function changes from `[MUSCLE]` to `[SKELETON]` only when `decomposition` is run for it — never earlier.

## Step 5: Ask what's next

Read the full decomposition document. Find all functions marked `[MUSCLE]` that do not appear as a `[SKELETON]` header. Show the list:

> "MUSCLE functions not yet decomposed: `fetch_user`, `get_permissions`, `format_response`. Which do you want to decompose next, or shall we stop?"

Wait for the user's response. Do not proceed until the user replies.
