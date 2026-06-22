---
name: walking-skeleton
description: Use when starting a feature top-down — orchestrates the decomposition loop, writes integration and unit tests, adds comments, and generates a plan for muscle functions.
---

# Walking Skeleton

Use at the start of a feature to design top-down: write the orchestration logic first, verify the system works end-to-end with stubs, then drive muscle implementation through TDD.

**Announce:** "Using walking-skeleton skill."

## Phase 1: Decomposition Loop

1. Ask: "What is the entry point function? Does it already exist in the codebase, or should I write it?"
2. Invoke the `superpowers:decomposition` skill for the entry point.
3. After each decomposition call, read the decomposition document and show the current tree.
4. Ask: "Which MUSCLE function do you want to decompose next, or shall we stop?"
5. If the user names a function → invoke `superpowers:decomposition` for that function. Return to step 3.
6. If the user says stop → proceed to Phase 2.

**Do not stop the loop on your own judgment.** Only the user decides when decomposition is complete.

## Phase 1.5: Resolve INTEGRATION functions

If the decomposition document contains any `[INTEGRATION]` functions, do this before writing tests:

1. Write a separate `inspect_<system>()` utility function that dumps real output from the external system for a sample input. Place it in the appropriate module (e.g. `parser_php.py`). This function is kept — it may be reused for debugging.
2. Run it on a real fixture and show the output to the user.
3. Hardcode the real output shape into the INTEGRATION function stubs.
4. Reclassify those functions from `[INTEGRATION]` to `[MUSCLE]` in the decomposition document.

After this phase, no `[INTEGRATION]` entries remain. All further steps work only with `[MUSCLE]`.

## Phase 2: Tests

### Integration test

Ask the user:

> "What should `<entry_point>` return for a typical real input? Give me an example: what input do you pass in, and what is the correct expected output?"

Write one integration test using the user's answer. Place it in `tests/test_<feature>_integration.py`:

```python
def test_<entry_point>_end_to_end():
    result = <entry_point>(<real_input>)
    assert result == <correct_expected_output>
```

This test uses the **correct** expected output — not the hardcoded stub output. It will fail immediately. That is intentional: it is the acceptance spec for the agent.

### Unit tests for MUSCLE functions

For each function marked `[MUSCLE]` in the decomposition document:

1. Read the stub to understand the return shape and parameter types.
2. Write a unit test in `tests/test_<function_name>.py`. If the correct behavior is ambiguous from the stub alone, ask the user: "What should `<function>` return for input `<example>`?"

```python
def test_<muscle_function>_returns_correct_result():
    result = <muscle_function>(<input>)
    assert result == <correct_expected_output>
```

All unit tests will fail. That is correct. They define what the agent must implement.

## Phase 3: Comments

For every function in the codebase (both skeleton and muscle), add a one-line comment directly above the `def` line:

- **Skeleton**: describe what it orchestrates and what its children must produce.
- **Muscle**: describe input types, return shape, and key behavior.

```python
# Orchestrates user fetch: loads user, resolves permissions, returns formatted response.
def handle_get_user(user_id: int) -> Response:
    ...

# Returns user dict {id, name, email} from DB by id. Raises NotFound if missing.
def fetch_user(user_id: int) -> dict:
    return {"id": user_id, "name": "stub", "email": "stub@example.com"}
```

Do not write multi-line comments. One line per function.

## Phase 4: Plan

Invoke the `superpowers:writing-plans` skill.

Brief it with:
- The list of all MUSCLE functions from the decomposition document and their file paths.
- The path to the integration test.
- The paths to all unit tests.
- The instruction: "Each muscle function has a failing unit test. There is also a failing integration test. Write a TDD plan: each task implements one muscle function until its unit test passes. The final task runs all tests and verifies the integration test is green."
