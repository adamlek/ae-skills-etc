# Streaming and laziness

Lazy pipelines produce one item at a time, only when the next step asks for it. Memory use stays flat no matter how large the input is.

## Lazy versus eager

| Lazy (returns an iterator) | Eager (builds a whole collection) |
|---|---|
| generator functions (`yield`) | `list(...)`, `tuple(...)`, `set(...)`, `dict(...)` |
| generator expressions `(x for x in xs)` | list/set/dict comprehensions |
| `map`, `filter`, `zip`, `enumerate`, `reversed` | `sorted(...)` |
| almost all of `itertools` | `len(...)`, `sum(...)`, `max(...)`, `"".join(...)` (consume fully) |
| `toolz`: `take`, `drop`, `concat`, `mapcat`, `unique`, `partition_all`, `sliding_window`, `pluck` | `toolz`: `groupby`, `reduceby`, `frequencies`, `countby` (return dicts) |

## When to materialize

Turn a stream into a list only when a step needs one of these:

- **The length** (`len`), or the last item.
- **More than one pass** over the data.
- **Sorting** or random access by index.
- **A stable snapshot** before an external source changes.
- **Debugging**, temporarily.

If the data is small and fits easily in memory, using lists throughout is fine and often clearer. Laziness is a tool for large or unbounded inputs, not a rule.

## Generator functions as steps

A generator function is the most flexible lazy step. Use it when a step needs state, setup, or several statements.

```python
def dedupe_consecutive(lines: Iterable[str]) -> Iterator[str]:
    previous = None
    for line in lines:
        if line != previous:
            yield line
        previous = line
```

Use `yield from` to delegate to another iterable:

```python
def read_all(paths: Iterable[Path]) -> Iterator[str]:
    for path in paths:
        yield from read_lines(path)
```

## Iterators are single-use

Once consumed, an iterator is empty. This is the most common streaming bug.

```python
errors = (line for line in lines if "ERROR" in line)
count = sum(1 for _ in errors)
first = next(errors, None)       # None: errors is already exhausted
```

Fixes, in order of preference:

1. **Do both things in one pass** with a loop.
2. **Materialize once** with `list()` if the data fits in memory.
3. **`itertools.tee`** if consumers advance roughly together. If one consumer runs far ahead, `tee` buffers everything in between, so it can use as much memory as a list.

Calling `iter()` on a list gives a fresh iterator each time. Calling `iter()` on an iterator returns the same iterator. A function that takes an `Iterable` and loops over it twice works on lists but silently fails on generators.

## Resource lifetime

A generator that opens a file keeps it open until the generator finishes or is closed.

```python
def read_lines(path: Path) -> Iterator[str]:
    with path.open(encoding="utf-8") as f:
        yield from f
```

This is fine when the consumer reads to the end. If the consumer stops early (`islice`, `takewhile`, `break`), the file stays open until the generator is garbage collected. When that matters, close explicitly:

```python
from contextlib import closing

with closing(read_lines(path)) as lines:
    header = next(lines)
```

Or open the resource in the caller and pass the open file into the step.

## Bounded batches

Batching keeps memory bounded while still allowing per-batch work such as bulk inserts or one LLM call per batch.

```python
from itertools import batched   # Python 3.12+

for batch in batched(records, 500):
    db.insert_many(batch)
```

Before Python 3.12, use `toolz.partition_all(500, records)`. Both yield a final short batch. `toolz.partition` drops the short final batch, which is rarely what you want.

## Peeking

To look at the first item without losing it:

```python
from itertools import chain

def peek(iterable):
    it = iter(iterable)
    for first in it:
        return first, chain([first], it)
    raise ValueError("empty input")
```

`toolz.peek(seq)` does the same and raises `StopIteration` on empty input.

## Debugging lazy pipelines

- Nothing runs until something consumes the stream. A pipeline with no sink does no work.
- Errors appear at consumption time, often far from where the bug is. The traceback shows the generator chain.
- Peek at an intermediate stream with `list(islice(stream, 5))`, but remember this consumes those items.
- Temporarily wrap a stage in `list()` to locate a failing step, then remove it.

## Performance notes

- Generator expressions and itertools have low per-item overhead. They are rarely the bottleneck.
- For numerical work on large arrays, use NumPy, pandas, or Polars instead of item-by-item Python.
- `cytoolz` is a drop-in, faster implementation of `toolz` for hot loops.
- Measure before optimizing. A clear loop is better than a clever chain that saves nothing measurable.
