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

## Phase 1.5: Run the skeleton end-to-end

After the decomposition loop, run the top-level function with a real input. The goal is **visible output** — something rendered in a browser, printed to a terminal, or written to a file. Do not proceed to Phase 2 until you see it.

Repeat until the pipeline produces visible output:

1. Run the skeleton with a real input.
2. If visible output appears → done, go to Phase 2.
3. If it crashes or produces nothing, find the blocker:
   - **INTEGRATION blocker** (external-library object that cannot be faked):
     1. Write `inspect_<system>()` — a utility that dumps real output from the external system on a sample input. Keep it; it may be reused for debugging.
     2. Run it on a real fixture, show the output to the user.
     3. Save the output to `tests/fixtures/<fixture>_<ext>.out` (e.g. `sample.php` → `sample_php.out`). This is the permanent record of the API shape — future agents read it instead of re-running the utility.
     4. Implement the function using the real API. These are thin wrappers — write the real code now.
     5. Reclassify `[INTEGRATION]` → `[MUSCLE]` in the decomposition document.
   - **MUSCLE blocker** (stub raises `NotImplementedError("DEPENDS ON: X")`):
     - If X is **not yet implemented** → implement X first (treat as INTEGRATION blocker above).
     - If X is **already implemented or stubbed** → write a proper hardcoded stub for this function using X's real output shape.
4. Return to step 1.

After this phase, the skeleton runs end-to-end and produces visible output.

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
2. Write a unit test in `tests/test_<function_name>.py` using **structural assertions** — check types, shapes, and invariants. Do not assert exact field values; at this stage only the structure is known.

```python
def test_<muscle_function>_returns_correct_structure():
    result = <muscle_function>(<input>)
    assert isinstance(result, <ExpectedType>)
    assert <structural_invariant>  # e.g. len(result.nodes) > 0
```

3. Run the tests — they must be **green** (stubs return correct structure).
4. Replace each stub body with `return None`.
5. Run the tests again — they must now be **red**.

The unit tests are now failing and ready to drive TDD implementation. The integration test is the primary acceptance spec.

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
