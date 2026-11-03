---
name: python-f-filter
description: Use when users need to conditionally extract, group, reduce, or filter data streams in Python. Leverages itertools and toolz for functional data processing, dictionary mapping, and complex predicate filtering.
---

# Python Filtering, Grouping & Reduction Tools

This skill focuses on conditionally processing data—filtering out noise, grouping related items, extracting specific fields, and reducing sequences using Python's `itertools` and `toolz` libraries.

## Core Principles

- **Predicates**: Many of these functions rely on a predicate (a function returning `True` or `False`). Keep predicates pure (no side effects).
- **Pipelines**: Complex filtering logic is best expressed as a composed pipeline (`toolz.pipe`) rather than deeply nested function calls.
- **The Groupby Rule**: `itertools.groupby` requires sorted data. `toolz.groupby` does not, but returns a dictionary and stores all data in memory.

## 1. Sequence Filtering

**Predicate Filtering (`itertools` & `toolz`)**
```python
from itertools import filterfalse, takewhile, dropwhile
from toolz import unique

# Filter out elements where predicate is True (opposite of filter)
truthy_only = list(filterfalse(lambda x: not x, data))

# Take items UNTIL predicate becomes False
# e.g., read up to first blank line
valid_lines = list(takewhile(lambda x: x.strip() != '', lines))

# Drop items UNTIL predicate becomes False, then keep the rest
# e.g., skip header rows
data_only = list(dropwhile(lambda x: x in headers, rows))

# Remove duplicates while preserving order
unique_items = list(unique(data, key=lambda x: x['id']))

```

**Boolean Masking (`itertools`)**

```python
from itertools import compress

data = ['a', 'b', 'c', 'd', 'e']
selectors = [True, False, True, False, True]
filtered = list(compress(data, selectors))
# ['a', 'c', 'e']

```

## 2. Grouping Data

There is a critical difference between grouping approaches:

**`itertools.groupby` (Memory Efficient, Requires Sorting)**
Groups *consecutive* matching items. Must be sorted first. Returns an iterator of iterators.

```python
from itertools import groupby

# MUST SORT FIRST
transactions.sort(key=lambda x: x['type'])

for trans_type, group in groupby(transactions, key=lambda x: x['type']):
    print(f"{trans_type}: {len(list(group))} items")

```

**`toolz.groupby` (Convenient, Memory Heavy)**
Groups *all* matching items regardless of order. Returns a dictionary of lists.

```python
from toolz import groupby

# No sorting required
groups = groupby(lambda x: x['type'], transactions)
# {'debit': [...], 'credit': [...]}

```

## 3. Reductions and Accumulations

**Running Totals (`itertools`)**

```python
from itertools import accumulate
import operator

# Running sum: [1, 3, 6, 10]
running_sum = list(accumulate([1, 2, 3, 4]))

# Running product: [1, 2, 6, 24]
running_prod = list(accumulate([1, 2, 3, 4], operator.mul))

```

**Group and Aggregate (`toolz`)**

```python
from toolz import reduceby

# Group by category, and sum the amounts in one pass
category_totals = reduceby(
    key=lambda x: x['category'],
    binop=lambda acc, x: acc + x['amount'],
    seq=transactions,
    init=0
)

```

## 4. Dictionary and Field Extraction

**Extracting Fields (`toolz`)**

```python
from toolz import pluck, get_in

# Extract specific keys from a list of dictionaries
names = list(pluck('name', user_records))

# Safe nested dictionary access
status = get_in(['response', 'metadata', 'status'], api_payload, default='unknown')

```

**Filtering Dictionaries (`toolz`)**

```python
from toolz import keymap, valmap, itemmap, valfilter

# Filter dict by values
high_scores = valfilter(lambda v: v > 90, scores_dict)

# Transform values while keeping keys
doubled = valmap(lambda v: v * 2, scores_dict)

```

## 5. Functional Pipelines for ETL

For complex filtering and extraction, use `toolz.pipe` to thread data through multiple operations readably:

```python
from toolz import pipe, valfilter, merge_with
from toolz.curried import filter, map
from operator import add

def process_sales(raw_data):
    return pipe(
        raw_data,
        filter(lambda d: d.get('active')),           # Filter active records
        toolz.groupby(lambda d: d['department']),    # Group by dept
        valfilter(lambda k: k in ['sales', 'tech']), # Filter specific depts
        lambda g: {k: merge_with(add, *v) for k, v in g.items()} # Reduce
    )

```

## Best Practices

1. **Choose the right groupby**: Use `itertools.groupby` for massive, pre-sorted streams (like log files). Use `toolz.groupby` for smaller, unordered data (like API responses).
2. Use `filterfalse` instead of complex list comprehensions when the negative predicate is already defined.
3. Keep data flowing lazily as long as possible; only use `list()` or `dict()` at the very end of your filtering pipeline.

```

```
