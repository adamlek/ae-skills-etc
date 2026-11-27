# Pitfalls

Common bugs in data-processing code, with the symptom and the fix.

## Iterators

**Consuming an iterator twice.**
Symptom: the second loop, `sum`, or `list` is empty.
Fix: do the work in one pass, or `list()` once. See [streaming and laziness](streaming-and-laziness.md).

**A function that takes `Iterable` but loops twice.**
Symptom: works in tests with lists, returns wrong results with generators.
Fix: type the parameter as `Sequence` and materialize at the call site, or rewrite as one pass.

**Testing lazy output without materializing it.**
Symptom: `assert result == [1, 2]` fails because `result` is a generator.
Fix: `assert list(result) == [1, 2]`.

**Laziness hides errors.**
Symptom: the traceback points at the sink, but the bug is in a transform.
Fix: read the whole generator chain in the traceback, or wrap stages in `list()` temporarily.

**Lazy evaluation captures late-bound variables.**
Symptom: every generator in a list uses the last value of a loop variable.

```python
gens = [(x * n for x in xs) for n in range(3)]   # all use n == 2 when consumed
```

Fix: bind the value with a function parameter, or materialize inside the loop.

## Grouping

**`itertools.groupby` on unsorted data.**
Symptom: the same key appears in several groups.
Fix: sort by the same key first, or use `toolz.groupby` / `defaultdict(list)`.

**Using a group after advancing `itertools.groupby`.**
Symptom: saved groups are empty.

```python
groups = {k: g for k, g in groupby(data, key)}   # every g is now empty
```

Fix: `{k: list(g) for k, g in groupby(data, key)}`.

**Confusing the two `groupby` signatures.**
`itertools.groupby(data, key)` versus `toolz.groupby(key, data)`. Swapping them raises `TypeError` or, worse, runs with the wrong arguments.

## Chunking and windows

**`toolz.partition` drops the final short chunk.**
Symptom: the last few records are silently lost.
Fix: use `partition_all` or `itertools.batched`.

**Plain `zip` truncates silently.**
Symptom: records missing when two streams have different lengths.
Fix: `zip(a, b, strict=True)`, or `zip_longest` if padding is intended.

**`sliding_window(n, xs)` on short input yields nothing.**
Handle the "fewer than *n* items" case explicitly if downstream code expects at least one window.

## toolz

**Importing `map` / `filter` from `toolz` instead of `toolz.curried`.**
`toolz.map` is the built-in. `pipe(xs, filter(pred))` raises `TypeError: filter expected 2 arguments`.

**Calling a non-curried toolz function with partial arguments inside `pipe`.**
`toolz.groupby(key)` raises `TypeError`. Use `toolz.curried.groupby(key)`.

**`valfilter` versus `keyfilter`.**
`valfilter` tests values, `keyfilter` tests keys. Filtering a grouped dict by its key names needs `keyfilter`.

**`merge_with` with a binary function.**
`merge_with(operator.add, a, b)` raises `TypeError`, because the function receives a single list of all values for a key. Use `merge_with(sum, a, b)`.

**Mutable `init` in `reduceby`.**
`reduceby(key, binop, xs, [])` shares one list across all keys if `binop` mutates it. Pass `list` (a callable) or make `binop` return a new value.

## itertools details

**`filterfalse(None, xs)` is valid.** It keeps the falsy items. To keep truthy items, use `filter(None, xs)`.

**`takewhile` swallows the first failing item.** It is consumed and not yielded. Use a loop if you need it.

**`cycle` stores a full copy** of its input. Do not cycle over a huge stream.

**`tee` can use unbounded memory** when one copy runs far ahead of the other.

## Design

**I/O inside a transform.** A transform that logs, reads config, or calls an API is hard to test and reuse. Move the effect to a source, sink, or injected dependency.

**Mutating input records.** A step that changes a dict in place affects every other reference to it, including earlier stages if the data was shared. Return new records instead.

**Swallowing errors.** `except Exception: continue` hides bugs in your own code as well as bad input. Catch the specific exception, and record what was skipped.

**Sorting a stream that does not fit in memory.** `sorted` materializes everything. Sort at the source (database `ORDER BY`, presorted files), or use an external sort.

**A long `pipe` nobody can debug.** If you need to inspect intermediate values, use named variables instead.
