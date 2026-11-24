---
name: testing-checklist
description: A checklist of what to test when adding or changing a feature (an endpoint, a parameter, a query node, a model field) so its coverage is thorough. Use when the user asks what a new feature needs tested, wants to plan tests for a change, asks "is this tested enough", or wants a testing checklist. It plans tests; it doesn't write them or draw coverage maps (see contract-coverage-map for that).
---

# Testing checklist

> **Draft.** Still open: how behavior tests are laid out, whether bug fixes get a failing-first
> test, and where golden files live. Treat those parts as suggestions.

Walk through this list for the feature being added or changed and report, for each item, which tests
cover it already and which are missing. This skill plans tests; it doesn't write them. To see an
endpoint's input contracts drawn against their tests, use the `contract-coverage-map` skill.

Steps are language-agnostic. Examples in parentheses show one way to do it in a Java/Spring codebase.

## Where tests pay off most

- **Endpoint tests give the most for the effort.** One request through the real endpoint shows whether
  receiving, deserialization, validation, or behavior is broken.
- **Lower-layer tests show where it broke.** Duplicating a check at the layer that owns it (serialization,
  matching logic, endpoint) means the lowest failing layer points at the bug. Each layer checks only its
  own job.

## Test data

Prove behavior against two sources when the feature reads data:

1. **The real, primary dataset**, because that's what users hit. Its data changes between releases, so
   assert what stays true: results contain a known item, and every result meets the condition. Don't
   assert exact counts or exact result sets.
2. **A small dataset you control**, written by hand, for exact assertions: this exact set of results,
   nothing more. Building it also proves the loader handles it.

## The checklist

For each feature, from its spec (fields, limits, rejected inputs, examples):

1. **Every implicit contract is explicit.** Each rule about an input (required, type, allowed values,
   ranges, cross-field limits, limits inherited from a lower layer) is written down where the code
   enforces it, not just assumed (`@NotNull`, `@Min`/`@Max`, `@Valid`, a named constant for each limit;
   or a zod/pydantic schema). A boundary nobody wrote down can't be tested.
2. **Each contract's edges are tested on both sides.**
   - Rejected: a value just outside each edge gets the right error status, and the message names the
     field and the limit or allowed values.
   - Accepted: the value at each edge, and one ordinary value in between, get through. A check written
     as `>` instead of `≥` passes every rejection test.
   - Parameters shared by every variant of an endpoint are tested once per endpoint, not once per variant.
3. **Behavior on valid input is right.**
   - Matches: a value that should find results finds the right ones.
   - Doesn't match: a value that shouldn't find anything finds nothing.
   - Exact on the controlled dataset; "contains a known item, and every result qualifies" on the real one.
4. **Serialization round-trips.** A unit test with no server or index: serialize, deserialize, compare
   equal (`ObjectMapper` write then read).
5. **Regression.**
   - **Golden files** pin the serialized shape: a checked-in expected JSON compared structurally to what
     the code produces (`JsonNode` equality). A round trip alone misses a renamed field; a golden file
     catches it.
   - **Never update a golden file automatically.** If the shape changes, the test fails and the
     developer decides whether the change was intended. Agents must not regenerate golden files to make
     tests pass; say so in the repo's agent instructions.
   - Existing behavior tests keep passing unchanged.

## Test shape

- One thing per test, named for the feature, the case, and the expected result
  (`testPageSizeAboveMaximumReturnsBadRequest`; in Jest, `it("rejects a page size above the maximum")`).
- Keep a contract's accepted and rejected tests side by side under one heading; behavior tests under their own.
- Use the testing framework's parameterization for "every value of" cases (JUnit `@EnumSource`, pytest
  `parametrize`), and count each case.

## Output

A short table per feature: checklist item, the tests that cover it, and what's missing, with where each
missing test would go. Don't restate tests that are already covered in prose.
