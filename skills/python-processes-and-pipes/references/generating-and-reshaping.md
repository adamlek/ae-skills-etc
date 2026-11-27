# Generating and reshaping

Change the shape of a stream: slice it, combine streams, flatten, chunk, window, or produce combinations. All functions here are lazy unless noted.

## Slicing a stream

```python
from itertools import islice
from toolz import take, drop

first_ten  = list(islice(stream, 10))
rows_10_20 = list(islice(stream, 10, 20))
every_3rd  = list(islice(stream, 0, None, 3))

first_ten  = list(take(10, stream))      # toolz: argument order fits currying
without_header = drop(1, rows)
```

`islice` works on any iterable but cannot take negative indices. For the last *n* items, use `collections.deque(stream, maxlen=n)` or `toolz.tail(n, stream)`.

Use slicing syntax (`xs[10:20]`) on lists; it is clearer.

## Counters and repeats

```python
from itertools import count, cycle, repeat

for line_no, line in zip(count(1), lines): ...    # prefer enumerate(lines, start=1)
colors = cycle(["red", "green", "blue"])           # infinite
zeros = repeat(0, 10)
```

`count` and `cycle` are infinite. Always pair them with something finite (`zip`, `islice`, `takewhile`).

## Combining streams

```python
from itertools import chain, zip_longest

all_rows = chain(rows_2023, rows_2024)                  # one after another
pairs    = zip(names, ages, strict=True)                # raise if lengths differ (3.10+)
padded   = zip_longest(names, ages, fillvalue=None)     # pad the shorter one
```

Use `strict=True` when lengths must match. A silent truncation by plain `zip` is a common data bug.

Merge several *sorted* streams into one sorted stream:

```python
import heapq
merged = heapq.merge(log_a, log_b, key=lambda e: e.timestamp)
```

Alternate between streams (round robin):

```python
from toolz import interleave
list(interleave([[1, 2, 3], "ab"]))   # [1, 'a', 2, 'b', 3]
```

## Flattening

```python
from itertools import chain
from toolz import concat, mapcat

flat  = chain.from_iterable([[1, 2], [3], [4, 5]])       # one level
flat  = concat([[1, 2], [3], [4, 5]])                     # same, toolz
words = mapcat(str.split, lines)                          # map, then flatten
words = (word for line in lines for word in line.split()) # same, plain Python
```

To flatten arbitrary depth, write a recursive generator and decide explicitly which types count as containers. Strings are iterable, so naive recursion on them never ends.

```python
def flatten(items):
    for item in items:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item
```

## Chunking

| Need | Tool | Final partial chunk |
|---|---|---|
| Fixed-size batches | `itertools.batched(xs, n)` (3.12+) | kept |
| Fixed-size batches, older Python | `toolz.partition_all(n, xs)` | kept |
| Exact-size chunks only | `toolz.partition(n, xs)` | **dropped** |
| Exact-size chunks, padded | `toolz.partition(n, xs, pad=None)` | padded |
| Split by a condition | `itertools.groupby` on a key, or a generator | — |

Since Python 3.13, `batched(xs, n, strict=True)` raises if the last batch is short.

Split a stream into sections at marker lines:

```python
def split_sections(lines: Iterable[str]) -> Iterator[list[str]]:
    section = []
    for line in lines:
        if line.startswith("#") and section:
            yield section
            section = []
        section.append(line)
    if section:
        yield section
```

## Windows

```python
from itertools import pairwise              # 3.10+
from toolz import sliding_window

deltas = [b - a for a, b in pairwise(readings)]
moving_avg = [sum(w) / 3 for w in sliding_window(3, readings)]
```

`sliding_window(n, xs)` yields nothing if `xs` has fewer than `n` items.

## Combinatorics

```python
from itertools import product, combinations, combinations_with_replacement, permutations

grid    = product(["S", "M", "L"], ["red", "blue"])  # replaces nested loops
pairs   = combinations(players, 2)                    # unordered, no repeats
orders  = permutations(stages, 3)                     # ordered
dice    = product(range(1, 7), repeat=2)
```

These grow quickly. `permutations` of 10 items yields 3.6 million tuples. Keep them lazy and filter early.

## Splitting one stream into several

```python
from itertools import tee
a, b = tee(stream, 2)
```

See [streaming and laziness](streaming-and-laziness.md) for when `tee` is a bad idea. To split by a predicate into two lists, a single loop is clearer:

```python
valid, invalid = [], []
for record in records:
    (valid if is_valid(record) else invalid).append(record)
```
