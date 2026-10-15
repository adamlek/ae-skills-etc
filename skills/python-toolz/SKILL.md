---
name: python-toolz
description: Use when working with the Python toolz package (functional utilities extending itertools). Helps implement functional programming patterns, compose pipelines, and perform data transformations efficiently.
---

# Python Toolz Skill

Use this skill when the user is working with the `toolz` package - a Python functional programming library that extends `itertools` with higher-level utilities.

## When to Use Toolz

Trigger this skill when the user:
- Mentions `toolz`, `cytoolz`, or functional programming patterns in Python
- Wants to compose functions or build data pipelines
- Needs to group, partition, or transform data in functional style
- Asks about itertools alternatives or more powerful iteration tools
- Wants to use currying, partial application, or function composition
- Is working with streaming/lazy data processing

## Key Toolz Modules

Understand these core modules:

### `toolz.itertoolz` - Iteration Patterns
Common functions to use:
- `groupby(key, seq)` - Group items by a key function (unlike itertools.groupby, doesn't require sorted input)
- `partition(n, seq)` / `partition_all(n, seq)` - Split sequence into chunks (exact or at end)
- `take(n, seq)` / `drop(n, seq)` - Get/drop N items from an iterable
- `concat(seqs)` - Flatten nested iterables
- `unique(seq, key=...)` - Remove duplicates (stable, preserves order)
- `pluck(index, seqs)` / `pluck(key, seqs)` - Extract field/index from collection of dicts/tuples
- `mapcat(func, seqs)` - Apply function then concatenate results
- `sliding_window(n, seq)` - Create overlapping windows (like moving average)
- `reduceby(key, func, seq)` - Group-reduce in one pass

### `toolz.functoolz` - Function Utilities
- `compose(*funcs)` - Compose functions right-to-left (f(g(x)))
- `pipe(data, *funcs)` - Thread data through functions
- `curry(func)` / `curried` namespace - Create curried versions of functions
- `partial(func, *args, **kwargs)` - Partial application (also in stdlib functools)
- `memoize(func)` - Cache function results
- `juxt(*funcs)` - Create a function that applies multiple funcs to the same argument

### `toolz.dicttoolz` - Dictionary Operations
- `merge(*dicts)` / `merge_with(func, *dicts)` - Merge multiple dicts
- `valmap(func, d)` / `keymap(func, d)` - Apply function to values or keys
- `itemmap(func, d)` - Apply function to (key, value) pairs
- `assoc(d, key, value)` / `dissoc(d, key)` - Generic dict operations
- `get_in(keys, coll)` - Nested dictionary access

## Implementation Patterns

### Functional Pipelines
Prefer `pipe()` or `compose()` for readable pipelines:

```python
from toolz import pipe, map, filter

result = pipe(
    data,
    filter(lambda x: x > 0),
    map(lambda x: x * 2),
    list
)
```

### Grouping and Aggregation
Use `groupby` or `reduceby` for group-by operations:

```python
from toolz import groupby, reduceby
from operator import add

# Group by key (no sort required)
groups = groupby(lambda x: x['category'], items)

# Group and aggregate in one pass (more efficient)
aggregated = reduceby(
    lambda x: x['category'],
    lambda acc, x: acc + x['value'],
    items,
    0
)
```

### Currying
Use currying for creating reusable, composable functions:

```python
from toolz import curry

@curry
def add(a, b):
    return a + b

add_5 = add(5)
result = add_5(3)  # 8
```

### Windowed Operations
Use `sliding_window` for moving operations:

```python
from toolz import sliding_window

# Moving average
data = [1, 2, 3, 4, 5]
windows = list(sliding_window(3, data))
# [(1, 2, 3), (2, 3, 4), (3, 4, 5)]

averages = [sum(w)/len(w) for w in windows]
```

## Best Practices

1. **Lazy evaluation**: Toolz iterables are lazy by default. Use `list()` or `consume()` only when needed.
2. **Memory efficiency**: Prefer toolz functions for large datasets - they work on generators without loading everything into memory.
3. **Composition over loops**: Replace complex loops with composed functions for clarity.
4. **Functional purity**: Keep functions pure when possible to enable memoization.
5. **Use `merge_with` for aggregation**: When merging dicts with conflicting keys, use `merge_with` with an aggregation function.
6. **Consider `cytoolz`**: For performance-critical code, `cytoolz` is a drop-in replacement with C-accelerated functions.

## Importing Toolz

```python
import toolz  # or from toolz import flow, curried, etc.

# Common patterns
from toolz import pipe, compose, groupby, pluck, merge
from toolz.curried import map, filter, reduce  # Returns curried versions
```

## Common Use Cases

### ETL/Data Processing
```python
from toolz import groupby, pluck, valfilter, merge_with

def process_pipeline(raw_data):
    return pipe(
        raw_data,
        filter(lambda d: d.get('active')),
        groupby(lambda d: d['category']),
        valfilter(lambda k: k in ['sales', 'marketing']),
        lambda g: {k: merge_with(add, *v) for k, v in g.items()}
    )
```

### API Response Transformation
```python
from toolz import pluck, assoc, mapcat

def transform_responses(responses):
    return pipe(
        responses,
        pluck('items'),  # Extract 'items' from each response
        mapcat(list),    # Flatten the nested items
        map(lambda x: assoc(x, 'processed', True))
    )
```

## When NOT to Use Toolz

- Simple operations are more readable with list comprehensions
- When extreme performance is needed (consider numpy/pandas for numerical work)
- When tight coupling with existing OO design patterns is required

## Debugging Tips

- Use `take(n, seq)` to peek at the first n items of a lazy pipeline
- Use `list(seq)` or `consume(seq)` to force evaluation when needed
- `toolz.trace` can help visualize data flow through a pipeline
- Remember that iterables are consumed when iterated - store in a list if needed multiple times

## Integration Considerations

- Toolz works well with `pandas`, `numpy`, and standard Python data structures
- Compatible with async code (but iterables will be consumed differently)
- Can be used alongside standard library `itertools`, `functools`, and `operator`
