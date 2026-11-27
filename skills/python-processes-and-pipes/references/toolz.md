# toolz

`toolz` extends `itertools` and `functools` with composition helpers, extra sequence operations, and immutable dict operations. `cytoolz` is a faster, drop-in C implementation with the same API.

Use toolz when it makes a step clearer than plain Python. Do not add the dependency for one function that a comprehension can replace.

## Argument order

toolz puts the data **last**: `take(n, seq)`, `groupby(key, seq)`, `valmap(f, d)`. This lets you curry the leading arguments and pass the result to `pipe`.

The itertools equivalents put the data **first**: `islice(seq, n)`, `groupby(seq, key)`. Check the order when switching between them.

## `pipe` and `compose`

```python
from toolz import pipe, compose

pipe(x, f, g, h)          # h(g(f(x))), reads left to right
compose(h, g, f)(x)       # same, reads right to left, returns a function
compose_left(f, g, h)(x)  # same, reads left to right, returns a function
```

Every step in `pipe` must take exactly one argument. For steps that need more, use the curried namespace, `functools.partial`, or a small named function.

## The curried namespace

`toolz.curried` provides versions of toolz functions (and `map`, `filter`, `sorted`, `reduce`) that return a function when called with fewer arguments than they need.

```python
from toolz import pipe
from toolz.curried import filter, map, groupby, valmap, take

report = pipe(
    orders,
    filter(lambda o: o.status == "paid"),
    groupby(lambda o: o.region),
    valmap(lambda os: sum(o.total for o in os)),
)
```

**Common mistake:** `from toolz import map, filter` gives you the *built-in* `map` and `filter`. Inside `pipe`, `filter(pred)` then raises `TypeError`. Always import them from `toolz.curried`.

Importing `map` and `filter` from `toolz.curried` shadows the built-ins for the whole module. If that is confusing, import the namespace instead:

```python
import toolz.curried as tc
pipe(rows, tc.filter(is_valid), tc.map(parse), list)
```

## `curry`

```python
from toolz import curry

@curry
def scale(factor: float, value: float) -> float:
    return factor * value

pipe(values, tc.map(scale(2.0)), list)
```

Put configuration parameters first and data last so partial application fills the configuration. Curried functions produce confusing errors when called with wrong arguments, so use `curry` sparingly in shared code. `functools.partial` is explicit and often enough.

## Other function helpers

| Function | Does | Example |
|---|---|---|
| `juxt(f, g, h)` | Calls several functions on the same input, returns a tuple | `juxt(min, max, len)(values)` |
| `complement(pred)` | Negates a predicate | `filter(complement(is_empty), xs)` |
| `identity` | Returns its argument | default key function |
| `memoize(f)` | Caches results; prefer `functools.cache` | — |
| `do(f, x)` | Calls `f(x)` for its side effect, returns `x` | logging inside a `pipe` |
| `excepts(Exc, f, handler)` | Calls `handler` instead of raising `Exc` | `excepts(ValueError, int, lambda _: None)` |
| `thread_first`, `thread_last` | Like `pipe`, but with `(f, extra_args)` tuples | `thread_last(xs, (map, inc), sum)` |

To log the data flowing through a `pipe`, insert `do(print)` or `do(logger.debug)` as a step (with `toolz.curried.do`).

## Sequence functions (itertoolz)

Lazy unless noted.

| Function | Does |
|---|---|
| `take(n, xs)`, `drop(n, xs)`, `tail(n, xs)` | First *n*, skip *n*, last *n* (`tail` is eager) |
| `first`, `second`, `nth(n, xs)`, `last` | Single items |
| `peek(xs)` | Returns `(first, iterator_with_first_restored)` |
| `concat(seqs)`, `mapcat(f, seqs)` | Flatten one level; map then flatten |
| `interleave(seqs)` | Round robin |
| `unique(xs, key=)` | Dedupe, keeps order |
| `partition(n, xs)`, `partition_all(n, xs)` | Chunks; `partition` drops a short last chunk |
| `sliding_window(n, xs)` | Overlapping windows |
| `pluck(key, xs)` | Field from each item; `key` may be a list of keys |
| `groupby(key, xs)` | **Eager**, returns `dict[key, list]` |
| `reduceby(key, binop, xs, init)` | **Eager**, group and fold in one pass |
| `frequencies(xs)`, `countby(key, xs)` | **Eager**, counts |
| `topk(k, xs, key=)` | **Eager**, the *k* largest |
| `isdistinct(xs)`, `isiterable(x)` | Checks |
| `accumulate(binop, xs)` | Running fold (data last, unlike itertools) |
| `join(leftkey, left, rightkey, right)` | Relational join of two sequences |

## Dict functions (dicttoolz)

All return new dicts.

| Function | Does |
|---|---|
| `merge(*dicts)` | Combine, last wins |
| `merge_with(fn, *dicts)` | Combine, `fn(list_of_values)` resolves collisions |
| `valmap`, `keymap`, `itemmap` | Transform values, keys, or `(key, value)` pairs |
| `valfilter`, `keyfilter`, `itemfilter` | Keep entries by value, key, or pair |
| `assoc(d, k, v)`, `dissoc(d, *ks)` | Copy with a key set or removed |
| `assoc_in(d, keys, v)`, `update_in(d, keys, f)` | Copy with a nested key set or updated |
| `get_in(keys, d, default=)` | Nested lookup |

## When not to use toolz

- One-off transforms that a comprehension expresses directly.
- Codebases where the team does not know toolz. Curried pipelines are harder to read for newcomers.
- Numerical work on large arrays. Use NumPy, pandas, or Polars.
- When a long `pipe` hides intermediate values that would make debugging easier. Named variables are free.
