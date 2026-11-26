# Immutability catalog

Preserve caller-owned data by default. Local mutation is allowed when the function owns the object and no caller-owned data is changed. Do not use immutability to justify unnecessary allocation or copying.

| Operation | Mutates input | Return-new alternative |
|---|---|---|
| Sort list | `items.sort()` | `sorted(items)` |
| Reverse list | `items.reverse()` | `items[::-1]` |
| Add list item | `items.append(x)` | `[*items, x]` |
| Extend list | `items.extend(other)` | `[*items, *other]` |
| Update dict | `record.update(changes)` | `record | changes` (Python 3.9+) |
| Remove dict key | `record.pop(key)` | `{k: v for k, v in record.items() if k != key}` |
| Add set member | `members.add(x)` | `members | {x}` |
| Update dataclass | field assignment | `dataclasses.replace(obj, field=value)` |

Choose the operation appropriate to ownership. Return-new operations are generally shallow; shared nested objects remain shared. Do not repeatedly copy growing collections inside loops.

## Domain data

```python
@dataclass(frozen=True)
class User:
    name: str
    permissions: frozenset[str]


def grant_permission(user: User, permission: str) -> User:
    return replace(user, permissions=user.permissions | {permission})
```

Frozen dataclasses prevent normal field reassignment, not mutation of nested lists or dictionaries. Use tuples and frozensets when their semantics fit. `Sequence` and `Mapping` express read-only interfaces to type checkers, not runtime immutability.

## Nested updates

Copy only the affected paths when sharing unchanged values is safe:

```python
def update_quantity(order: dict, item_id: str, quantity: int) -> dict:
    return {
        **order,
        'items': [
            {**item, 'quantity': quantity} if item['id'] == item_id else item
            for item in order['items']
        ],
    }
```

Use deeper copying or immutable domain models when independent ownership is required. Avoid indiscriminate deepcopy.

## Local mutation

```python
def group_documents(documents: Iterable[Document]) -> dict[str, list[Document]]:
    groups = defaultdict(list)
    for document in documents:
        groups[document.category].append(document)
    return dict(groups)
```

This mutates only a locally created accumulator; document objects themselves are shared. Use classes for meaningful state, invariants, identity or resource lifetime. Preserve documented in-place APIs when refactoring unless changing the contract explicitly.
