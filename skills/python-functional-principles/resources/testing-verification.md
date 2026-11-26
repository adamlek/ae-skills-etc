# Testing and verification

Test meaningful behavior and invariants rather than incidental implementation details. Pure functions should normally require no external resources, elaborate mocks or application startup.

```python
def test_grant_permission():
    user = User(name='Alice', permissions=frozenset({'read'}))
    updated = grant_permission(user, 'write')
    assert updated.permissions == frozenset({'read', 'write'})
    assert user.permissions == frozenset({'read'})
```

## Levels

- Functions: test individual contracts, edge cases, expected outcomes and input preservation where required.
- Pipelines: test composed behavior, ordering, failure propagation and side-effect decisions.
- Boundaries: test resource cleanup, actual external interactions and failure semantics. Use integration tests for guarantees that mocks cannot establish.
- Programs: test the intended use case end to end, including necessary safety and operational constraints.

For plan/apply workflows, test that dry-runs and actual execution use the same decision logic and dry-runs do not mutate external state. For streaming and concurrency, test single-pass inputs, resource lifetime, bounded work and cancellation where relevant.

## Refactoring

Inspect callers and established contracts before changing implementation. Preserve observable behavior unless a change is required or requested. Include regression coverage for the defect or requested change. Remove tests only when their contract is intentionally obsolete.

Use existing test tools and conventions. Run relevant tests, linting or type checks when available. Report failures, unexecuted checks and limitations accurately. Do not claim mocked tests establish real external-system behavior or that unexecuted tests passed.
