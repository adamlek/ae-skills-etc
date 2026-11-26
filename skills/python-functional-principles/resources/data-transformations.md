# Data transformations

Prefer idiomatic Python over strict functional syntax. Choose the operation that communicates intent and has suitable complexity.

```python
names = [user.name for user in users]
active_users = [user for user in users if user.active]
active_names = [user.name for user in users if user.active]
users_by_id = {user.id: user for user in users}
categories = {document.category for document in documents}
total = sum(item.price * item.quantity for item in items)
has_admin = any(user.is_admin for user in users)
all_valid = all(document.valid for document in documents)
user = next((user for user in users if user.id == target_id), None)
```

Dictionary indexing by ID silently overwrites duplicates; use explicit validation or grouping if duplicates matter. `all` returns True for an empty iterable. Use an explicit nonempty check when required by the domain. Sets discard order guarantees; preserve order when needed.

Use `map(parse_document, raw_documents)` when applying an existing function is clearer. Use generator expressions when lazy evaluation helps. Avoid complex nested comprehensions and `reduce` when a built-in or loop is clearer.

## Grouping and aggregation

```python
def count_categories(documents: Iterable[Document]) -> Counter[str]:
    return Counter(document.category for document in documents)
```

Use defaultdict for grouping, Counter for frequencies, sum/min/max for aggregation and itertools for meaningful iterator composition. `itertools.groupby` groups consecutive keys, not all equal keys; sort first only when appropriate.

Use loops for multiple related operations, early termination, effects, accumulation and error handling. Do not replace a clear loop with a custom functional abstraction.

## Avoid repeated work

Build a lookup once when doing repeated searches. Compute each expensive transformation once and pass the result onward. Do not materialize intermediate collections merely to preserve a preferred syntax. Match the project's supported Python version and dependencies.
