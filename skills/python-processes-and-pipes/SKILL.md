---
name: python-processes-and-pipes
description: >
  Process data and build pipelines in Python. Use when writing, reviewing, or
  refactoring code that reads, filters, transforms, groups, aggregates,
  chunks, or writes data: ETL jobs, scripts, log/CSV/JSON processing, API
  response shaping, and batch LLM workflows. Covers pipeline design (pure
  transforms, I/O at the edges, thin entry points), lazy streaming with
  generators, and choosing between comprehensions, built-ins, itertools,
  and toolz.
---

# Python Processes and Pipes

Data flows from a **source**, through a sequence of named **transforms**, into a **sink**. Write code so that this flow is visible.

```
read -> parse -> keep valid -> group by user -> summarize -> save
```

If you can write the task as a line like this, you can write the program. Each step becomes a function. The pipeline calls them in order.

## Mental model

| Part | Job | Side effects |
|---|---|---|
| **Source** | Read from files, APIs, databases. Yield records. | Yes: input |
| **Transform** | Take records, return records. Map, filter, group, reduce, reshape. | No |
| **Sink** | Consume records. Write, send, print. | Yes: output |
| **Pipeline** | Connect a source, transforms, and a sink in order. | Only through its parts |
| **Program** | Parse arguments, build clients, call the pipeline, handle top-level errors. | Yes |

These are roles, not classes. A small program is often one file with a few functions.

## Core rules

1. **Start from the data flow.** Write the steps as a line of verbs before you write code. Name functions after those verbs.
2. **Keep I/O at the edges.** Sources and sinks touch the outside world. Transforms do not. You can then test the transforms with plain lists.
3. **Make the pipeline read like an outline.** It connects steps. Implementation details belong inside the steps.
4. **Stay lazy until you must materialize.** Pass iterators between steps. Call `list()`, `sorted()`, or `dict()` only when a step needs the whole dataset: length, sorting, multiple passes, or random access.
5. **Use the simplest tool that expresses the step.** Prefer, in order: a comprehension or loop, a built-in, `itertools`, then `toolz`. Move down the list only when it makes the step clearer.
6. **Pass configuration and dependencies in.** Thresholds, paths, batch sizes, and clients are parameters. Do not read them from globals inside transforms.
7. **Validate where data enters.** Parse and check raw input in the source. Downstream steps can then trust their input. Catch errors where you can act on them; never swallow them silently.
8. **Test small things with data, and big things with fakes.** Test transforms with literal inputs. Test pipelines with fake sources, sinks, and clients.

## Choosing a tool

| Task | First choice | When it gets harder |
|---|---|---|
| Transform each item | comprehension / generator expression | `map` with a named function |
| Keep some items | comprehension with `if` | `filter`, `itertools.filterfalse` |
| Stop or start at a condition | — | `itertools.takewhile`, `dropwhile` |
| First *n* items of a stream | `itertools.islice` | `toolz.take` / `drop` |
| Concatenate / flatten one level | `itertools.chain`, `chain.from_iterable` | `toolz.concat`, `toolz.mapcat` |
| Fixed-size chunks | `itertools.batched` (3.12+) | `toolz.partition_all` |
| Overlapping windows | `itertools.pairwise` (size 2) | `toolz.sliding_window` |
| Group unsorted data | `collections.defaultdict(list)` | `toolz.groupby` |
| Group sorted runs / huge streams | `itertools.groupby` | — |
| Group and aggregate in one pass | loop with a `dict` | `toolz.reduceby` |
| Count things | `collections.Counter` | `toolz.frequencies` |
| Deduplicate, keep order | `dict.fromkeys(items)` | `toolz.unique(key=...)` |
| Running totals | `itertools.accumulate` | — |
| Nested dict lookup | chained `.get()` | `toolz.get_in` |
| Chain many one-argument steps | named intermediate variables | `toolz.pipe` / `compose` |

Details for each row are in the references.

## The shape of a pipeline

```python
import json
from collections.abc import Callable, Iterable, Iterator
from dataclasses import dataclass
from itertools import batched
from pathlib import Path


@dataclass(frozen=True)
class Item:
    text: str
    score: float


# Source: owns file access and parsing.
def read_items(path: Path) -> Iterator[Item]:
    with path.open(encoding="utf-8") as lines:
        for line in lines:
            if line.strip():
                row = json.loads(line)
                yield Item(text=row["text"], score=float(row["score"]))


# Transforms: pure, testable with plain lists.
def keep_confident(items: Iterable[Item], min_score: float) -> Iterator[Item]:
    return (item for item in items if item.score >= min_score)


def build_prompt(items: Iterable[Item]) -> str:
    bullets = "\n".join(f"- {item.text}" for item in items)
    return f"Summarize these items:\n{bullets}"


# Sink: owns output.
def write_lines(lines: Iterable[str], path: Path) -> None:
    with path.open("w", encoding="utf-8") as out:
        for line in lines:
            out.write(json.dumps(line) + "\n")


# Pipeline: reads like the outline.
def summarize_file(
    source: Path,
    destination: Path,
    generate: Callable[[str], str],
    *,
    min_score: float = 0.7,
    batch_size: int = 10,
) -> None:
    items = read_items(source)
    confident = keep_confident(items, min_score)
    prompts = (build_prompt(batch) for batch in batched(confident, batch_size))
    summaries = (generate(prompt) for prompt in prompts)
    write_lines(summaries, destination)
```

Why it is built this way:

- Each step is lazy, so memory use stays flat even for large files. Nothing runs until `write_lines` starts pulling.
- `generate` is a parameter. Production code passes a real LLM client; tests pass a function that returns a fixed string.
- `keep_confident` and `build_prompt` need no files or network to test.
- A program entry point parses arguments, builds the client, and calls `summarize_file`. It holds no processing logic.

## Two composition styles

**Named intermediates (default).** Each line names what the data *is* at that point. This style is easy to debug, works with any function signature, and needs no library.

**`toolz.pipe` (optional).** Useful when the steps are a long chain of one-argument functions and the intermediate names add nothing:

```python
from toolz import pipe
from toolz.curried import filter, map

totals = pipe(
    records,
    filter(is_active),
    map(to_amount),
    sum,
)
```

Use `toolz.curried` inside `pipe`. Plain `toolz.map` and `toolz.filter` are the built-ins and do not curry. See [toolz](references/toolz.md).

Do not build a `Pipeline` class, a stage registry, or a shared mutable context object unless a concrete requirement needs one.

## References

Read only the references that apply to the current task.

| Reference | Read when |
|---|---|
| [Pipeline design](references/pipeline-design.md) | Structuring a program, splitting steps, placing I/O, config, errors, and tests |
| [Streaming and laziness](references/streaming-and-laziness.md) | Large inputs, generators, iterator exhaustion, when to materialize, resource cleanup |
| [Generating and reshaping](references/generating-and-reshaping.md) | Slicing, combining, flattening, chunking, windowing, combinatorics |
| [Filtering, grouping, reducing](references/filtering-grouping-reducing.md) | Predicates, dedup, group-by, aggregation, dict transforms |
| [toolz](references/toolz.md) | Using `pipe`, `compose`, `curry`, `juxt`, dicttoolz, or cytoolz |
| [itertools quick reference](references/itertools.md) | Looking up an itertools function or recipe |
| [Pitfalls](references/pitfalls.md) | Debugging a pipeline or reviewing data-processing code |
| [Worked examples](references/examples.md) | Starting a new pipeline from a known pattern |

For general function design, immutability, and error modelling, see the `python-functional-principles` skill.

## Review checklist

- Can you describe the pipeline as a line of verbs, and does the code match it?
- Do sources and sinks hold all the I/O? Are transforms free of it?
- Is data materialized only where a step needs the whole dataset?
- Is any iterator consumed twice by accident?
- Does each step use the simplest tool that is clear?
- Are thresholds, paths, and clients passed in rather than hard-coded?
- Can each transform be tested with a literal list?
