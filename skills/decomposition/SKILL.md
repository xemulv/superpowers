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

## Step 2: Identify and classify called functions

List every function the skeleton calls that does not yet have a real implementation. Classify each as:

- **`[INTEGRATION]`** — interfaces with an external system: file I/O, database, HTTP, external library (e.g. tree-sitter, ORM, SDK). Its real output defines data shapes the rest of the system must handle.
- **`[MUSCLE]`** — pure business logic with no external dependencies. Can be tested and implemented using synthetic data.

## Step 3: Write stubs

The goal: after this step the top-level skeleton must be **runnable end-to-end and produce visible output** (in a browser, terminal, or file). Every stub must return realistic hardcoded data — never `None`, never an empty structure — so the pipeline does not crash before reaching the output.

For each called function, create a stub in the appropriate module file. Match field names and structure to what the skeleton actually uses downstream.

```python
def fetch_user(user_id: int) -> dict:
    return {"id": user_id, "name": "stub", "email": "stub@example.com"}

def get_permissions(user: dict) -> dict:
    return {"roles": ["admin"]}

def format_response(user: dict, perms: dict) -> dict:
    return {"status": 200, "body": {"user": user, "perms": perms}}
```

**`[INTEGRATION]` stubs — pipeline blocker rule:**

If an INTEGRATION function returns an external-library object that cannot be realistically faked (e.g. a tree-sitter `Node`, a DB cursor, an HTTP response object), do NOT write `return None`. Instead, mark it explicitly as a **pipeline blocker**:

```python
def parse_file(php_file):
    raise NotImplementedError("BLOCKER: returns tree-sitter Node — cannot be faked. Run inspect_php() first.")
```

A blocker means the skeleton cannot run until this function is implemented. Report the blocker to the user immediately at the end of Step 5. Do not proceed to the next decomposition without resolving it.

## Step 4: Update the decomposition document

Find or create `docs/superpowers/decompositions/YYYY-MM-DD-<feature>.md`.

Add a section for this function. Use the literal text `[SKELETON]`, `[MUSCLE]`, or `[INTEGRATION]`:

```markdown
### handle_get_user [SKELETON]
- load_from_db [INTEGRATION]
- fetch_user [MUSCLE]
- get_permissions [MUSCLE]
- format_response [MUSCLE]
```

If a child was already listed under another parent, update that parent's entry to `[SKELETON]`.

Rules:
- All children start as `[MUSCLE]` or `[INTEGRATION]` when first listed.
- A function changes to `[SKELETON]` only when `decomposition` is run for it — never earlier.
- `[INTEGRATION]` functions must be implemented before `[MUSCLE]` functions — their real output defines data shapes that MUSCLE stubs cannot guess.

## Step 5: Ask what's next

Read the full decomposition document. Find all functions marked `[MUSCLE]` or `[INTEGRATION]` that do not appear as a `[SKELETON]` header. Show the list grouped by type:

> "INTEGRATION functions (implement first): `load_from_db`. MUSCLE functions not yet decomposed: `fetch_user`, `get_permissions`, `format_response`. Which do you want to decompose next, or shall we stop?"

Wait for the user's response. Do not proceed until the user replies.
