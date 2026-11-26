---
name: python-functional-principles
description: >
  Apply simplicity through composition when designing, writing, modifying,
  reviewing, or refactoring Python code. Construct programs from small,
  meaningful functions with explicit inputs and outputs. Compose functions
  into pipelines and pipelines into programs. Reuse existing operations,
  isolate side effects, preserve data, and introduce complexity only when
  the problem requires it. Prefer idiomatic Python and practical functional
  programming over unnecessary abstractions, architectural layers, and
  strict functional purity.
---

# Python Principles

## Philosophy

Write small, meaningful operations with explicit inputs and outputs. Compose these operations into pipelines, and compose pipelines into programs. Reuse existing operations, isolate side effects, preserve data, and introduce complexity only when the problem requires it.

**Simplicity through composition, rather than functional purity alone.**

Use functional programming as a means to achieve clarity, predictability and reuse. Do not sacrifice correctness, readability, maintainability or performance to satisfy a paradigm. Optimize for the effort required to understand and change the program, not for the fewest lines or the greatest number of abstractions.

## Mental model

- Function: an operation that performs one coherent task.
- Pipeline: a composition of functions that performs a larger task.
- Program: a composition of pipelines and functions that solves a use case.
- Pure function: an operation whose result depends only on its inputs and has no observable side effects.
- Boundary: a point where the program interacts with external state or resources.

Understand a function without understanding the entire program; a pipeline without inspecting every function; a program without inspecting every pipeline. These are conceptual levels, not classes or frameworks that must be implemented.

## Core rules

1. Solve the actual problem using the simplest correct implementation.
2. Extract meaningful operations, not arbitrary fragments. Keep trivial expressions inline.
3. Make inputs, outputs and dependencies explicit. Use meaningful names and appropriate type annotations.
4. Compose ordinary function calls; name intermediate results when they clarify data flow.
5. Inspect and reuse existing operations before introducing new ones. Share behavior with the same meaning, not merely similar syntax.
6. Separate computation and decisions from external interactions. Prefer pure computations and explicit I/O boundaries.
7. Preserve caller-owned data unless mutation is part of the contract. Permit encapsulated local mutation.
8. Prefer Python built-ins, comprehensions, generators and readable loops over custom machinery.
9. Introduce classes, configuration objects, interfaces and modules only for concrete responsibilities.
10. Validate at appropriate boundaries, handle real failure modes, and preserve necessary safeguards.
11. Avoid unnecessary rescans, repeated work, allocation and resource initialization.
12. Preserve existing contracts and unrelated behavior when modifying code. Remove obsolete paths when safe.

## Composition example

```python
def process_dataset(
    source: Path,
    destination: Path,
    client: LLMClient,
) -> None:
    documents = read_data(source)
    valid_documents = filter_valid_documents(documents)
    groups = group_by_category(valid_documents)
    prompts = create_prompts(groups)
    responses = generate_responses(prompts, client)
    save_results(responses, destination)
```

Let the pipeline describe what happens and in which order. Let individual operations describe how. Do not introduce a Pipeline class, shared mutable context, generic stage registry or composition framework unless actual requirements justify it. Keep a small program in one file when that is sufficient.

## Implementation workflow

1. Understand the required behavior, available inputs, outputs, constraints and external interactions.
2. Inspect relevant code, callers and existing contracts. Identify reusable operations.
3. Identify meaningful computations, transformations, decisions and effects. Define their contracts.
4. Express the workflow through ordinary function composition. Keep orchestration visible.
5. Implement only missing behavior. Use existing Python constructs and project conventions.
6. Simplify: remove duplicated logic, unnecessary wrappers, speculative options, excessive defensive checks and repeated work.
7. Verify the required behavior, failure modes, resource ownership and compatibility. Run relevant tests when available; disclose unexecuted tests and unresolved limitations.

For an existing program, modify the operation responsible for the requested behavior rather than accumulating patches, alternate paths or compatibility layers without a demonstrated need. Do not rewrite unrelated code simply to enforce this skill.

## Resources

Read only the resources relevant to the current task. The core rules above apply independently.

| Resource | Read when |
|---|---|
| [Function design](resources/function-design.md) | Designing signatures, deciding granularity, naming or typing operations |
| [Composition patterns](resources/composition-patterns.md) | Constructing pipelines, organizing programs, reusing stages or planning changes |
| [Immutability catalog](resources/immutability-catalog.md) | Updating collections, domain models or shared state |
| [Data transformations](resources/data-transformations.md) | Mapping, filtering, grouping, aggregation or choosing loops |
| [Execution boundaries](resources/execution-boundaries.md) | Working with I/O, dependencies, configuration, resources or concurrency |
| [Error handling](resources/error-handling.md) | Designing validation, domain outcomes, exceptions or retries |
| [Streaming patterns](resources/streaming-patterns.md) | Processing large datasets, avoiding repeated work or managing lazy iteration |
| [Anti-patterns](resources/anti-patterns.md) | Reviewing code for overengineering, unnecessary indirection or duplication |
| [Testing and verification](resources/testing-verification.md) | Testing functions, pipelines, integrations or refactoring existing code |

## Final review

- Can each function be understood independently, and does it represent a meaningful operation?
- Is the workflow visible without reading every implementation detail?
- Are dependencies, side effects, state ownership and data flow explicit?
- Does each new abstraction serve an actual requirement?
- Can any component be removed or replaced with a simpler existing operation without losing necessary behavior?
- Are correctness, security, existing contracts and resource guarantees preserved?

Functions describe operations. Pipelines describe workflows. Programs describe use cases. Achieve simplicity through composition.
