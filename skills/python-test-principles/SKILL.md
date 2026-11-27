---
name: test-principles
description: Design reusable, behavior-oriented tests by separating the case being evaluated from the function, service, or process that produces the observed value. Apply when writing, reviewing, refactoring, or extending unit tests, integration tests, regression tests, API tests, and evaluation suites. Define general cases with explicit inputs and expected behavior, then compose them with the relevant producers. Prefer small, independent evaluation functions and thin test orchestration over duplicated assertions or tests coupled to implementation details.
---

# Test Principles

## Core principle: separate the case from the producer

Define what should be true before deciding how to obtain the value to test. A case describes a behavioral requirement, not the implementation that satisfies it.

Treat the output of a function, process, API, or pipeline as the input to a case. The case must not need to know where that value came from. Any producer that satisfies the same contract should be testable using the same case.

Keep three responsibilities separate:

- **Scenario:** Define the stimulus, initial conditions, and relevant expectations.
- **Producer:** Execute the operation being tested and obtain an observable result.
- **Case:** Evaluate that result against an explicit expectation or invariant.

Compose these responsibilities in a test. Do not put producer-specific setup, execution, or branching inside a reusable case.

## 1. Define the general case first

Start with the behavior that matters. Define the smallest meaningful predicate or assertion that captures it. Give it a name that describes the requirement rather than the implementation.

```python
from collections.abc import Callable
from typing import TypeVar

T = TypeVar("T")


def case(value: T, case_f: Callable[[T], bool]) -> bool:
    return case_f(value)


def is_six_digit_string(value: object) -> bool:
    return (
        isinstance(value, str)
        and len(value) == 6
        and value.isascii()
        and value.isdigit()
    )


assert case("123456", is_six_digit_string)
assert not case("12345", is_six_digit_string)
assert not case("abcdef", is_six_digit_string)
```

The case evaluates the observed value. It does not generate it or import the function that generates it. Here the six-digit requirement is illustrative; define the actual requirement from the contract, not from a convenient example.

Do not introduce a generic `case` wrapper merely to forward arguments when calling the predicate directly is clearer. The separation of responsibilities is the principle; the wrapper is not mandatory.

## 2. Compose cases with producers

Execute the producer outside the evaluation function. Pass the observed result into the same case regardless of its origin.

```python
def test_local_identifier():
    observed = generate_local_identifier()
    assert is_six_digit_string(observed), repr(observed)


def test_remote_identifier():
    observed = request_remote_identifier()
    assert is_six_digit_string(observed), repr(observed)
```

If multiple producers share the same contract and setup is compatible, parameterize the test instead of duplicating it:

```python
import pytest


@pytest.mark.parametrize(
    "produce",
    [generate_local_identifier, request_remote_identifier],
    ids=["local", "remote"],
)
def test_identifier_contract(produce):
    observed = produce()
    assert is_six_digit_string(observed), repr(observed)
```

Keep producer-specific unit and integration tests in appropriate suites when their environments or failure modes differ. Reuse the case across those suites without making a unit test depend on a remote service.

## 3. Make expectations explicit and independent

Distinguish the input sent to the producer from the observed value passed to the case. Provide the expected result, relevant context, or tolerance explicitly when the behavior cannot be evaluated from the observed value alone.

```python
def assert_matches_expected(observed, expected):
    assert observed == expected


@pytest.mark.parametrize(
    "source, expected",
    [("  hello  ", "hello"), ("", "")],
)
def test_normalize(source, expected):
    observed = normalize(source)
    assert_matches_expected(observed, expected)
```

Do not compute expected results by calling the same implementation being tested or by duplicating its algorithm. Derive expectations from the specification, known examples, independently calculated results, or meaningful invariants. Use direct equality when the helper adds no semantic value.

## 4. Keep cases independent and composable

- Prefer small cases that each express one coherent behavioral requirement.
- Keep evaluation functions deterministic and free of unnecessary side effects. Supply clocks, reference data, or other context explicitly when needed.
- Separate data preparation, fixtures, mocks, network requests, and cleanup from evaluation.
- Reuse an existing case when its semantics match. Do not generalize unrelated requirements merely because their assertions look similar.
- Combine relevant cases at the test orchestration level. Avoid large validators that hide which requirement failed.
- Keep test code simpler than production code. Use ordinary functions, assertions, and parameterization before introducing classes, registries, or custom test frameworks.

## 5. Test the contract, not incidental implementation details

Evaluate observable behavior: return values, errors, state changes, emitted events, or other required effects. For non-value behavior, obtain the relevant observation (such as a captured event or state snapshot) outside the case and evaluate it against the contract. Capture expected exceptions at the execution boundary; check their relevant properties separately when useful.

Avoid asserting private methods, intermediate steps, exact call sequences, or data representations unless they are explicitly part of the required behavior. When interactions themselves form the contract, capture and evaluate those interactions rather than ignoring them.

Do not confuse a reusable case with a complete test. A test must still identify the producer, control relevant conditions, execute the operation, and report failure with sufficient context.

## 6. Select meaningful scenarios

For each requirement, select representative valid inputs, boundary conditions, invalid inputs, and relevant regressions. Test both acceptance and rejection when a case acts as a validator. Do not add redundant examples that exercise the same behavior without increasing confidence.

Assert the strongest justified expectation. A format check does not prove correctness of the content, and a non-empty result does not prove that the intended operation succeeded.

Use parameterization for variations of the same behavior. Keep distinct requirements separately named so failures remain actionable. Never weaken an assertion simply to accommodate a failing implementation.

## Review before finishing

- Can the case evaluate an equivalent observation from another producer without modification?
- Is the required behavior clear independently of the implementation?
- Are stimulus, observed result, and expected behavior distinguishable?
- Does a failing case actually fail the test, with useful diagnostic context?
- Are the expectations independent of the implementation under test?
- Are the cases as simple as possible, without needless wrappers or abstractions?

If a test cannot be reused because it verifies a genuinely producer-specific contract, retain that specificity. Do not force generalization where no shared behavior exists.
