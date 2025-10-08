# Unit Test Naming Convention and Ratio Rule

This document defines a project-local convention for naming and generating unit tests with a strict 1:5 positive-to-negative ratio per function to strengthen failure-path coverage.

## Overview

The convention requires every test case name to follow the regex pattern `^\d+_(positive|negative)$` to ensure consistent, sortable identifiers across suites.

The generation policy enforces exactly one positive test for every five negative tests per function under test, targeting robust boundary and error handling verification.

## Naming Convention

- Test names must match `^\d+_(positive|negative)$` to keep automated tooling deterministic and collision-free.
- The leading integer should increment monotonically within a file or suite to preserve ordering and avoid duplicate names.
- Optional zero-padding (e.g., `001_positive`) may be adopted if lexicographic sort order must align with numeric order.

## Ratio Policy

- Maintain a per-function ratio of 1:5 for positive to negative tests to emphasize edge and error scenarios in complex systems.
- Generate tests in blocks of 6 per function: 1 positive and 5 negative to make the ratio trivially verifiable.
- If total tests are not a multiple of 6, keep positives â‰¤ âŒˆN/6âŒ‰ and negatives â‰¥ âŒŠ5N/6âŒ‹ while staying as close to 1:5 as possible.

## Cline Rule Definition

The following snippet can be placed into a Cline ruleset file to enforce naming and ratio during generation or edits.

```yaml
rules:
  - id: unit-test-naming-and-ratio
    name: Enforce unit test naming and 1:5 ratio
    when: generating or editing unit tests
    must:
      - "Each test case name MUST match the pattern: ^\\d+_(positive|negative)$"
      - "For every function under test, maintain a positive:negative ratio of 1:5 (i.e., for every 1 positive test, generate 5 negative tests)."
      - "Create tests in blocks of 6 per function: 1 positive and 5 negative."
      - "If total tests are not a multiple of 6, keep positives <= ceil(total/6) and negatives >= floor(5*total/6) while staying as close to 1:5 as possible."
      - "Increment the leading integer monotonically within the file or suite to avoid duplicate names."
    validate:
      name_regex: "^\\d+_(positive|negative)$"
      ratio:
        scope: per_function
        positive_to_negative: "1:5"
        tolerance:
          max_positive_per_6: 1
          min_negative_per_6: 5
```

## Example Layout

This illustrates how a function `foo()` would be covered with one positive and five negative tests in a single block of six.

```text
tests/
  foo_tests.rs
    001_positive
    002_negative
    003_negative
    004_negative
    005_negative
    006_negative
```

## Generation Guidance

- Start numbering at 001 within each file or use continuous numbering across the suite based on repository preference.
- Keep positive tests focused on the canonical successful path with minimal mocks and maximal determinism.
- Distribute negative tests across validation failures, boundary limits, resource faults, and protocol or contract violations.

## Validation Checklist

- All test names match `^\d+_(positive|negative)$` and are unique within the suite.
- For each function, total positives equal the number of six-test blocks and negatives are five times that count.
- Edge cases are explicitly represented among the five negative tests, not clustered into a single failure mode.

## Notes for Rust, Kotlin, and JNI Projects

Repositories spanning Rust, Kotlin, and JNI integrations benefit from heavier negative coverage due to FFI boundaries and async/concurrency failure modes.

Prioritize negative scenarios that mimic real-world failures such as:
- Type mismatches across language boundaries
- Nullability/Option misuse
- Panics/exceptions at FFI boundaries
- Timeouts or cancellations in async operations
- Thread safety violations

## Example Prompts

Below are prompt patterns that align with the rule and help generators produce conformant tests.

```text
- "Generate 6 unit tests for function `parse_config`: one positive and five negative, 
  named as <int>_positive or <int>_negative, continuing numbering from the last test."

- "For `connect_socket`, produce the next block of 6 tests with 1 success path and 
  5 error paths (timeout, DNS fail, invalid port, permission denied, closed fd), 
  enforcing ^\d+_(positive|negative)$."

- "Create only negative tests for `validate_payload` to complete the 1:5 ratio for 
  this function, continuing numbering without gaps."
```

## Compatibility Considerations

- If a framework restricts test identifiers to alphanumerics, the rule may be adapted by replacing underscores with permitted separators while preserving the integer-first ordering.
- When test discovery relies on file-level grouping, keep a stable mapping between function names and dedicated test files to simplify ratio verification.

## Maintenance Tips

- Track the next available integer in a per-file header comment or generator state to avoid collisions in parallel work.
- Validate ratios in CI with a lightweight script that counts positives and negatives per function and fails on deviation.
- Periodically refactor negative tests to ensure diversity across input domains, concurrency states, and resource conditions.
