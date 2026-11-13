---
name: python-f-iter
description: Efficiently generate, combine, sequence, and chunk iterable data using Python's itertools and toolz libraries. Use when users need to handle infinite sequences, Cartesian products, flattening nested structures, chunking data, or windowed iterations.
---

# Python Iteration & Sequencing Tools

This skill focuses on efficiently generating, structuring, and iterating over data using Python's standard `itertools` and the functional `toolz` library. 

## Core Principles

- **Lazy Evaluation**: Both `itertools` and `toolz` generate values on-demand. Only consume memory when converted to a `list()` or fully iterated.
- **Compose over Loop**: Chain iterators together instead of writing heavily nested `for` loops or creating intermediate lists.
- **Exhaustion**: Remember that iterators are consumed after a single pass. Use `itertools.tee` or convert to a list if you need multiple passes.

## 1. Generation and Subsetting

**Infinite Iterators (`itertools`)**
```python
from itertools import count, cycle, repeat

# count: infinite counter (great for zip)
for i, item in zip(count(start=100, step=10), items):
    pass

# cycle: repeat sequence infinitely
colors = cycle(['red', 'green', 'blue'])
next(colors) # 'red'

# repeat: repeat single value N times
zeros = repeat(0, 10)

```

**Slicing and Taking (`itertools` & `toolz`)**

```python
from itertools import islice
from toolz import take, drop

# Get elements 10-20 without loading entire iterable
middle = list(islice(data_stream, 10, 20))

# Get every 3rd element
every_third = list(islice(data_stream, 0, None, 3))

# toolz alternatives for head/tail operations
first_five = list(take(5, data_stream))
skip_first_five = list(drop(5, data_stream))

```

## 2. Combining and Flattening

**Flattening Iterables**

```python
from itertools import chain
from toolz import concat, mapcat

# Combine multiple iterables
combined = chain(list1, list2, list3)

# Flatten a list of lists
matrix = [[1, 2], [3, 4], [5, 6]]
flat_itertools = list(chain.from_iterable(matrix))
flat_toolz = list(concat(matrix))

# Apply function then flatten
transformed = list(mapcat(lambda x: [x, x*2], [1, 2, 3]))
# [1, 2, 2, 4, 3, 6]

```

**Zipping with Different Lengths**

```python
from itertools import zip_longest

names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30]

# Fills missing values from the shorter iterable
for name, age in zip_longest(names, ages, fillvalue='Unknown'):
    print(name, age)

```

## 3. Combinatorics and Windowing

**Cartesian Products and Permutations (`itertools`)**

```python
from itertools import product, combinations, permutations

# Nested loop alternative
for color, size in product(['red', 'blue'], ['S', 'M', 'L']):
    print(color, size)

# All unique pairs (order doesn't matter)
pairs = list(combinations(['A', 'B', 'C', 'D'], 2))

# All orderings
orderings = list(permutations(['A', 'B', 'C'], 2))

```

**Sliding Windows (`toolz`)**

```python
from toolz import sliding_window

# Great for moving averages or sequence analysis
data = [1, 2, 3, 4, 5]
windows = list(sliding_window(3, data))
# [(1, 2, 3), (2, 3, 4), (3, 4, 5)]

```

## 4. Chunking and Splitting

**Partitioning Data (`toolz`)**

```python
from toolz import partition, partition_all

data = [1, 2, 3, 4, 5, 6, 7]

# Exact chunks (drops remainder)
exact_chunks = list(partition(3, data))
# [(1, 2, 3), (4, 5, 6)]

# Keep remainder
all_chunks = list(partition_all(3, data))
# [(1, 2, 3), (4, 5, 6), (7,)]

```

**Splitting an Iterator (`itertools`)**

```python
from itertools import tee

# Create independent copies of an iterator
it1, it2, it3 = tee(data_stream, 3)

```

## Best Practices

1. Avoid `islice(count(), N)`. While it works lazily, `range(N)` is simpler.
2. Use generator expressions alongside these tools for maximum memory efficiency.
3. For performance-critical code doing heavy windowing or combinatorics, consider `cytoolz`.
