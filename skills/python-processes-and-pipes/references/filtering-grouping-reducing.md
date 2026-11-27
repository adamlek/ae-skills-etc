# Filtering, grouping, reducing

Keep the records you want, organize them by key, and combine them into results.

## Predicates

A predicate is a function that returns `True` or `False`. Keep predicates pure and give non-trivial ones a name:

```python
def is_billable(event: Event) -> bool:
    return event.kind == "usage" and not event.internal

billable = (e for e in events if is_billable(e))
```

A named predicate can be reused, tested, and negated without duplicating logic.

## Filtering

```python
from itertools import filterfalse, takewhile, dropwhile, compress

kept     = (x for x in xs if pred(x))       # default
kept     = filter(pred, xs)                  # when pred is already a named function
rejected = filterfalse(pred, xs)             # the complement, without writing `not`
truthy   = filter(None, xs)                  # drop falsy values ("", 0, None, [])

body     = takewhile(lambda l: l.strip(), lines)       # up to the first blank line
after    = dropwhile(lambda l: l.startswith("#"), lines)  # skip leading comments
selected = compress(rows, mask)              # keep rows where mask is truthy
```

`takewhile` consumes the first failing item and discards it. If you need that item, write a loop.

## Deduplicating

```python
unique_ids = list(dict.fromkeys(ids))                  # keeps first-seen order, hashable items

from toolz import unique
unique_users = unique(users, key=lambda u: u.email)    # lazy, by key
```

`set(xs)` loses order. `toolz.unique` keeps a set of seen keys, so memory grows with the number of distinct keys.

## Grouping

The two `groupby` functions have the same name and different behavior.

| | `itertools.groupby(xs, key)` | `toolz.groupby(key, xs)` |
|---|---|---|
| Groups | **consecutive** equal keys only | all equal keys |
| Input | must be sorted by key for full grouping | any order |
| Output | lazy iterator of `(key, group_iterator)` | `dict[key, list]` |
| Memory | one group at a time | everything |
| Use for | sorted streams, runs, huge files | small to medium unsorted data |

```python
from itertools import groupby
from operator import attrgetter

by_day = attrgetter("day")
for day, events in groupby(sorted(all_events, key=by_day), key=by_day):
    print(day, sum(e.amount for e in events))
```

Each group iterator is invalidated when the outer iterator advances. Consume or `list()` the group inside the loop.

Without toolz, use `defaultdict`:

```python
from collections import defaultdict

by_user = defaultdict(list)
for order in orders:
    by_user[order.user_id].append(order)
```

## Aggregating

Plain loop with a dict (the default):

```python
totals: dict[str, float] = {}
for sale in sales:
    totals[sale.region] = totals.get(sale.region, 0) + sale.amount
```

`toolz.reduceby` does the same in one call without building groups first:

```python
from toolz import reduceby

totals = reduceby(
    lambda sale: sale.region,
    lambda acc, sale: acc + sale.amount,
    sales,
    0,
)
```

If `init` is mutable (a list or dict), pass a zero-argument callable such as `list` so each key gets its own copy.

Counting:

```python
from collections import Counter

by_status = Counter(r.status for r in responses)
top_three = by_status.most_common(3)
```

`toolz.frequencies(xs)` and `toolz.countby(key, xs)` return plain dicts with the same information.

Reductions over the whole stream:

| Need | Tool |
|---|---|
| Sum, min, max, any, all | built-ins (`max` and `min` accept `key=` and `default=`) |
| Running totals | `itertools.accumulate(xs)`, `accumulate(xs, max)` |
| Custom fold to one value | `functools.reduce(fn, xs, init)`, but a loop is often clearer |
| Several aggregates in one pass | a loop, or `toolz.juxt` over a materialized list |

## Extracting fields

```python
from operator import itemgetter, attrgetter
from toolz import pluck, get_in

names  = [row["name"] for row in rows]                 # default
names  = pluck("name", rows)                           # lazy, toolz
pairs  = map(itemgetter("id", "name"), rows)           # tuples of several fields
status = get_in(["response", "meta", "status"], payload, default="unknown")
```

## Transforming dicts

| Goal | Plain Python | toolz |
|---|---|---|
| Change values | `{k: f(v) for k, v in d.items()}` | `valmap(f, d)` |
| Change keys | `{f(k): v for k, v in d.items()}` | `keymap(f, d)` |
| Keep by value | `{k: v for k, v in d.items() if p(v)}` | `valfilter(p, d)` |
| Keep by key | `{k: v for k, v in d.items() if p(k)}` | `keyfilter(p, d)` |
| Merge, last wins | `a \| b` | `merge(a, b)` |
| Merge, combine collisions | loop | `merge_with(sum, a, b)` |
| Copy with one key changed | `{**d, "k": v}` | `assoc(d, "k", v)` |
| Copy with a nested key changed | — | `assoc_in(d, ["a", "b"], v)`, `update_in(d, ["a", "b"], f)` |

`merge_with` passes the combining function a *list* of colliding values, so use functions like `sum` or `max`, not binary ones like `operator.add`.

All toolz dict functions return new dicts. They never mutate their input.
