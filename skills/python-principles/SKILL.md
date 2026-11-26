---
name: python-principles
description: Design, write, review, and refactor Python programs as focused functions composed into readable pipelines and thin entry points. Use for nontrivial Python scripts, data processing, ETL, automations, and LLM workflows where modularity, reuse, clear data flow, and testability matter. Favor minimal abstractions, explicit dependencies, and separation of transformations from side effects.
---

# Python Principles

Build programs from small, meaningful operations that are easy to understand, test, reuse, and rearrange. Prefer straightforward composition over clever or elaborate architecture.

## Conceptual layers

- **Function (operation):** Perform one cohesive task with clearly defined inputs, outputs, and behavior. Make it small enough to understand without tracing unrelated code; do not impose an arbitrary line limit.
- **Pipeline:** Compose operations to accomplish a workflow. Show the order of steps and how results move between them. Keep implementation details inside the operations.
- **Program:** Configure and coordinate one or more pipelines. Manage entry points, arguments, dependencies, output destinations, and top-level errors.

A program can contain one pipeline or many. Do not create layers merely to satisfy these labels.

## Design rules

1. **Start from the data flow.** Describe the task as a short sequence of domain actions, such as `read -> filter -> group -> build prompt -> generate -> save`. Give each meaningful action a descriptive name.
2. **Give functions one reason to change.** Group closely related statements within a function. Extract a function when it represents a distinct responsibility, is useful independently, or makes the caller substantially clearer. Avoid helpers that merely forward a single call.
3. **Make inputs and outputs explicit.** Pass required values as parameters and return results. Prefer type hints and meaningful data structures for nontrivial data. Avoid hidden globals, implicit mutation, and functions whose outputs depend on unrelated external state.
4. **Keep transformations pure where practical.** Filtering, grouping, parsing, and formatting should usually return new values without I/O or hidden side effects. Put file access, network calls, LLM requests, database operations, and persistence behind clearly named boundary functions.
5. **Make pipelines read like an outline.** A pipeline should mostly connect operations, not contain their detailed implementation. Use plain Python calls, loops, and comprehensions; do not introduce a pipeline framework unless it solves a concrete problem.
6. **Do things once, reuse later.** Centralize shared rules and genuinely repeated behavior. Extract reusable functions around stable concepts, not superficial code similarity. Avoid speculative abstraction and premature general-purpose utilities.
7. **Separate configuration from behavior.** Pass thresholds, paths, model choices, and other settings explicitly. Keep hard-coded environment details out of reusable transformations.
8. **Handle failures at the right boundary.** Validate externally supplied data where it enters the program. Raise informative exceptions or return explicitly modeled failures. Catch, log, retry, or recover where the program has enough context to act; do not silently swallow errors.
9. **Keep resource use proportional.** Prefer iterators, generators, or bounded batches if input data may be large. Avoid loading entire datasets into memory without a reason. Do not sacrifice clarity for speculative optimization.
10. **Test at the smallest useful level.** Unit-test deterministic operations using simple input/output cases. Test pipelines with controlled or fake external dependencies. Use integration tests for files, APIs, databases, and LLM clients.
11. **Keep organization proportional to complexity.** Start with a single readable module when sufficient. Split modules by stable responsibility only when this helps navigation or ownership. Avoid unnecessary classes, base classes, managers, registries, factories, and indirection.

## Preferred workflow for writing or refactoring code

1. Identify the program's inputs, outputs, side effects, and sequence of operations.
2. Define the essential data shapes and function signatures.
3. Implement and test the independent transformations first.
4. Implement I/O boundaries and pass external services as dependencies when this improves testability.
5. Compose the operations into a pipeline with visible data flow.
6. Add a thin program entry point that supplies configuration and runs the pipeline.
7. Simplify: remove duplicate logic, unnecessary wrappers, hidden state, and abstractions without demonstrated value.

## Example: dataset-to-LLM pipeline

```python
import json
from collections.abc import Callable, Iterable, Iterator, Sequence
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class Item:
    text: str
    score: float


def read_items(path: Path) -> list[Item]:
    items = []
    with path.open(encoding="utf-8") as source:
        for line in source:
            if not line.strip():
                continue
            row = json.loads(line)
            items.append(Item(text=row["text"], score=row["score"]))
    return items


def filter_items(items: Iterable[Item], min_score: float) -> list[Item]:
    return [item for item in items if item.score >= min_score]


def batch_items(items: Sequence[Item], size: int) -> Iterator[list[Item]]:
    if size < 1:
        raise ValueError("Batch size must be positive")
    for start in range(0, len(items), size):
        yield list(items[start : start + size])


def build_prompt(items: Sequence[Item]) -> str:
    lines = "\n".join(f"- {item.text}" for item in items)
    return f"Summarize these items:\n{lines}"


def run_pipeline(
    path: Path,
    generate: Callable[[str], str],
    *,
    min_score: float = 0.7,
    batch_size: int = 10,
) -> list[str]:
    items = read_items(path)
    selected = filter_items(items, min_score)

    results = []
    for batch in batch_items(selected, batch_size):
        prompt = build_prompt(batch)
        results.append(generate(prompt))
    return results
```

`read_items` owns file I/O; `filter_items`, `batch_items`, and `build_prompt` can be tested without external systems. `run_pipeline` shows the workflow and accepts `generate` as a dependency, allowing a real LLM client in production and a fake implementation in tests. A program entry point can parse arguments, construct the LLM client, call `run_pipeline`, and save results.

The list-based approach is suitable for modest datasets. For large inputs, replace it with streaming or bounded processing while keeping the same separation of responsibilities.

## Sanity check
Before introducing a new function, class, abstraction, or module, determine what concrete responsibility it serves. If it does not improve readability, reuse, testability, or separation of concerns, prefer the simpler implementation.


## Review checklist

- Can the pipeline's intent be understood by reading only its function calls?
- Does each function do one cohesive job, without being artificially fragmented?
- Can transformations be tested without files, networks, or an LLM?
- Are dependencies, data structures, and parameter names explicit?
- Is shared logic defined once, without speculative generalization?
- Are error handling, resource use, and module structure as simple as the task allows?

When generating or revising Python code, apply these principles by default. Explain tradeoffs briefly when a simpler implementation intentionally deviates from them.
