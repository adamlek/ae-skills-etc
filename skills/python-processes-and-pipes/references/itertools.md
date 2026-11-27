# itertools quick reference

All functions return lazy iterators. Official docs: https://docs.python.org/3/library/itertools.html

Import the names you use: `from itertools import chain, islice`.

## Infinite

| Function | Yields |
|---|---|
| `count(start=0, step=1)` | `start, start+step, ...` |
| `cycle(xs)` | items of `xs` repeated forever (stores a copy) |
| `repeat(x, times=None)` | `x` forever, or `times` times |

## Finite, one input

| Function | Yields |
|---|---|
| `accumulate(xs, func=add, *, initial=None)` | running results |
| `batched(xs, n, *, strict=False)` | tuples of up to `n` items (3.12+; `strict` 3.13+) |
| `compress(xs, selectors)` | items where the selector is truthy |
| `dropwhile(pred, xs)` | items after `pred` first fails |
| `filterfalse(pred, xs)` | items where `pred` is false; `pred=None` keeps falsy items |
| `groupby(xs, key=None)` | `(key, group)` for each run of equal keys |
| `islice(xs, stop)` / `islice(xs, start, stop, step)` | a slice; no negative indices |
| `pairwise(xs)` | overlapping pairs `(a, b), (b, c), ...` (3.10+) |
| `starmap(func, xs)` | `func(*item)` for each item |
| `takewhile(pred, xs)` | items until `pred` first fails |
| `tee(xs, n=2)` | `n` independent iterators (buffers the difference) |

## Finite, several inputs

| Function | Yields |
|---|---|
| `chain(*iterables)` | items of each iterable in turn |
| `chain.from_iterable(iterables)` | same, from an iterable of iterables |
| `zip_longest(*iterables, fillvalue=None)` | tuples, padding shorter inputs |

Built-in `zip(*iterables, strict=False)` stops at the shortest; `strict=True` raises on mismatch.

## Combinatoric

| Function | Yields |
|---|---|
| `product(*iterables, repeat=1)` | Cartesian product |
| `permutations(xs, r=None)` | ordered arrangements |
| `combinations(xs, r)` | unordered selections, no repeats |
| `combinations_with_replacement(xs, r)` | unordered selections, with repeats |

## Useful recipes

From the itertools docs, or close to them. Copy them rather than adding a dependency. The `more-itertools` package contains these and many more.

```python
from collections import deque
from itertools import chain, islice, groupby

def take(n, iterable):
    return list(islice(iterable, n))

def nth(iterable, n, default=None):
    return next(islice(iterable, n, None), default)

def tail(n, iterable):
    return iter(deque(iterable, maxlen=n))

def consume(iterator, n=None):
    if n is None:
        deque(iterator, maxlen=0)
    else:
        next(islice(iterator, n, n), None)

def flatten(list_of_lists):
    return chain.from_iterable(list_of_lists)

def all_equal(iterable, key=None):
    return len(take(2, groupby(iterable, key))) <= 1

def first_true(iterable, default=False, pred=None):
    return next(filter(pred, iterable), default)

def unique_justseen(iterable, key=None):
    """Drop consecutive duplicates: AAAABBBCCDAABBB -> A B C D A B"""
    return (next(group) for _, group in groupby(iterable, key))
```
