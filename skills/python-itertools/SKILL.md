---
name: python-itertools
description: Efficiently iterate, group, and filter data using Python's itertools module. Use when user mentions itertools, needs to work with iterables, wants to avoid manual loops for grouping/chaining/filtering, or asks about optimizing iteration patterns.
---

# Python itertools for Efficient Iteration

This skill provides guidance for using Python's `itertools` module to write more efficient, readable, and Pythonic code when working with iterables.

## Core Principles

- **Prefer itertools over manual loops** when possible - it's C-optimized and more readable
- **Lazy evaluation** - most itertools functions return iterators that generate values on-demand
- **Compose operations** - chain multiple itertools operations together for complex processing
- **Import specific functions** - `from itertools import chain, groupby, islice` is cleaner than `import itertools`

## Common Patterns

### 1. Iteration and Subsetting

**islice** - slice iterators without creating intermediate lists
```python
# Get first 5 elements
from itertools import islice
first_five = list(islice(data_stream, 5))

# Get elements 10-20 without loading entire list
middle = list(islice(data_stream, 10, 20))

# Get every nth element
every_third = list(islice(data_stream, 0, None, 3))
```

**count** - infinite counter
```python
from itertools import count

for i, item in zip(count(), items):
    # i is 0, 1, 2, ...
    print(i, item)

# Start from a specific value
for i in count(start=100, step=10):
    if i > 150:
        break
```

**cycle** - repeat an iterable infinitely
```python
from itertools import cycle

colors = cycle(['red', 'green', 'blue'])
for i in range(5):
    print(next(colors))  # red, green, blue, red, green
```

**repeat** - repeat a value
```python
from itertools import repeat

# Repeat a single value
for x in repeat(42, 3):  # Only repeat 3 times
    print(x)  # 42, 42, 42
```

### 2. Combining Iterables

**chain** - combine multiple iterables
```python
from itertools import chain

# Simple concatenation
combined = chain(list1, list2, list3)

# Or use from_iterable for list of iterables
lists = [[1,2], [3,4], [5,6]]
all_items = chain.from_iterable(lists)
```

**zip_longest** - zip iterables of different lengths
```python
from itertools import zip_longest

names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30]  # Shorter
default_name = 'Unknown'
default_age = 0

for name, age in zip_longest(names, ages, fillvalue=(default_name, default_age)):
    print(f"{name}: {age}")
```

**product** - Cartesian product (like nested loops)
```python
from itertools import product

colors = ['red', 'blue']
sizes = ['S', 'M', 'L']

for color, size in product(colors, sizes):
    print(f"{color} shirt size {size}")
```

**combinations** - all subsets of a specific size (order-independent)
```python
from itertools import combinations

teams = ['Alice', 'Bob', 'Charlie', 'Diana']
pairs = list(combinations(teams, 2))
# [('Alice', 'Bob'), ('Alice', 'Charlie'), ...]
```

**permutations** - all orderings
```python
from itertools import permutations

items = ['A', 'B', 'C']
ordering = list(permutations(items, 2))
# [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
```

### 3. Grouping and Filtering

**groupby** - group consecutive items (key function required, sort first!)
```python
from itertools import groupby

# IMPORTANT: itertools.groupby only groups CONSECUTIVE items
# Sort data first if you want groups by value
transactions = sorted([
    {'type': 'debit', 'amount': 10},
    {'type': 'credit', 'amount': 20},
    {'type': 'credit', 'amount': 15},
], key=lambda x: x['type'])

for trans_type, group in groupby(transactions, key=lambda x: x['type']):
    total = sum(t['amount'] for t in group)
    print(f"{trans_type}: {total}")
```

**filterfalse** - opposite of filter()
```python
from itertools import filterfalse

# Filter out falsy values
truthy = list(filterfalse(lambda x: not x, data))

# Alternative to list comprehension: [x for x in data if x]
```

**dropwhile** - drop items while predicate is true
```python
from itertools import dropwhile

# Skip header rows
headers = ['Name', 'Age', 'Score']
data = ['John', 25, 89, 'Jane', 30, 92]
valid_data = dropwhile(lambda x: x in headers, data)
```

**takewhile** - take items while predicate is true
```python
from itertools import takewhile

# Read up to first blank line
lines = ['line1', 'line2', '', 'line4']
content = list(takewhile(lambda x: x.strip() != '', lines))
# ['line1', 'line2']
```

**compress** - filter by booleans
```python
from itertools import compress

data = ['a', 'b', 'c', 'd', 'e']
selectors = [True, False, True, False, Truefiltered = list(compress(data, selectors))
# ['a', 'c', 'e']
```

### 4. Accumulation and Reductions

**accumulate** - running totals, products, or custom reductions
```python
from itertools import accumulate

# Running sum
numbers = [1, 2, 3, 4, 5]
running_sum = list(accumulate(numbers))
# [1, 3, 6, 10, 15]

# Running product (need import operator)
import operator
running_product = list(accumulate(numbers, operator.mul))
# [1, 2, 6, 24, 120]

# Custom reduction with max
running_max = list(accumulate([3, 1, 4, 1, 5], max))
# [3, 3, 4, 4, 5]
```

**tee** - split an iterator
```python
from itertools import tee

# iterator can only be consumed once normally
# tee creates n independent copies
iterator = range(10)
it1, it2, it3 = tee(iterator, 3)

list(it1)  # consumes first copy
list(it2)  # still works
list(it3)  # still works
```

## When to Use itertools vs Other Approaches

| Situation | Use itertools | Alternative |
|-----------|---------------|-------------|
| Simple iteration over list | No | `for x in items:` |
| Need first N items from large dataset | Yes: `islice` | `items[:N]` |
| Need to group related data | Yes: `groupby` | `defaultdict(list)` |
| Combine multiple iterables | Yes: `chain` | `list1 + list2 + list3` |
| Need Cartesian product | Yes: `product` | Nested loops |
| All combinations of N items | Yes: `combinations` | Manual recursion |
| Filter by complex predicate | Yes: `filterfalse`/`dropwhile` | List comprehension |
| Running totals | Yes: `accumulate` | Manual loop with accumulator |
| Infinite counter | Yes: `count` | `while True:` with manual increment |

## Common Pitfalls

1. **groupby requires sorted data**: `itertools.groupby` only groups consecutive elements. If your data isn't sorted, you'll get multiple groups for the same key. `sorted(data, key=func)` before using `groupby`.

2. **Only consume iterator once**: After you iterate through an itertools, it's exhausted. Use `list()` or `tee()` if you need multiple passes.

3. **islice on infinite iterators**: `islice(range(1000000), 10)` is fine (10 elements), but `islice(count(), 10)` creates a lazy iterator that consumes memory only when needed.

4. **filterfalse requires predicate**: `filterfalse(lambda x: x)` filters falsy values, but `filterfalse(None, iterable)` is not valid.

## Real-World Examples

### Processing large CSV
```python
from itertools import islice, groupby
import csv

def process_large_csv(filepath, rows_per_batch=1000):
    with open(filepath) as f:
        reader = csv.DictReader(f)
        
        # Process in batches to avoid loading entire file
        batch_num = 0
        while True:
            batch = list(islice(reader, rows_per_batch))
            if not batch:
                break
            
            # Sort by category for grouping
            batch.sort(key=lambda x: x['category'])
            
            for category, group in groupby(batch, key=lambda x: x['category']):
                # Process group
                print(f"Batch {batch_num} - {category}: {len(list(group))} items")
            
            batch_num += 1
```

### Flattening nested structures
```python
from itertools import chain

# Flatten lists of lists
matrix = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
flat = list(chain.from_iterable(matrix))

# More general: flatten any depth
def flatten(iterable):
    for item in iterable:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item
```

### Round-robin scheduling
```python
from itertools import cycle, islice

def round_robin(*iterables):
    """Round-robin schedule from multiple iterables"""
    iterators = [iter(it) for it in iterables]
    for items in cycle(iterators):
        item = next(items, None)
        if item is None:
            remaining = [it for it in iterators if list(it)]
            if not remaining:
                break
        else:
            yield item

teams = ['TeamA', 'TeamB', 'TeamC']
venues = ['Venue1', 'Venue2', 'Venue3']

for team, venue in zip(round_robin(*teams), round_robin(*venues)):
    print(f"{team} at {venue}")
```

## Optimization Tips

1. **Compose operations**: Chain multiple itertools operations instead of intermediate lists
   ```python
   # Good - lazy composition
   result = filter(
       lambda x: x > 5,
       map(lambda x: x * 2, range(100))
   )

   # Bad - creates intermediate lists
   doubled = [x * 2 for x in range(100)]
   filtered = [x for x in doubled if x > 5]
   ```

2. **Use generator expressions** instead of list comprehensions when possible with itertools

3. **Profile before optimizing**: `itertools` is fast, but readability matters. Use simple loops when they're clearer.

## Testing itertools Code

When testing code that uses itertools, remember that iterators can only be consumed once:

```python
# Bad - fails on second assert
my_iterator = groupby(data, key=func)
list(my_iterator)  # exhaust iterator
assert len(list(my_iterator)) == n  # Empty!

# Good - convert to list first
my_list = list(groupby(data, key=func))
assert len(my_list) == n
```

## Module Reference

Always check the official docs: https://docs.python.org/3/library/itertools.html

Key categories:
- Infinite iterators: `count`, `cycle`, `repeat`
- Terminating on shortest: `accumulate`, `chain`, `compress`, `dropwhile`, `filterfalse`, `groupby`, `islice`, `starmap`, `takewhile`, `tee`, `zip_longest`
- Combinatoric: `product`, `permutations`, `combinations`, `combinations_with_replacement`
