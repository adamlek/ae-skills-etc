# Function design

## Meaningful granularity

Extract an operation when it has independent domain meaning, is reused, needs independent testing, or hides detail that distracts from a caller. Do not extract every expression or impose arbitrary line limits.

Prefer:

```python
def filter_positive(values: Iterable[int]) -> list[int]:
    return [value for value in values if value > 0]
```

Do not extract `is_positive` unless it represents a meaningful reusable predicate. Conversely, do not inline substantial domain calculations simply because they have one caller.

## Explicit contracts

```python
def calculate_total(
    prices: Iterable[Decimal],
    discount: Decimal,
) -> Decimal:
    subtotal = sum(prices, Decimal('0'))
    return subtotal * (1 - discount)
```

Pass required information rather than reading hidden configuration, mutating global variables or passing a universal context. Use `Iterable` for traversal, `Sequence` for indexing, and concrete types when their capabilities are required. Read-only interfaces do not imply runtime immutability. Use existing domain types; avoid speculative generics and protocols.

Use keyword-only options when arguments would otherwise be ambiguous. Group settings only when they form a coherent reusable configuration.

```python
def generate_response(prompt: str, client: LLMClient, *, temperature: float = 0.2) -> str:
    return client.generate(prompt, temperature=temperature)
```

## Readability

Name operations after intent: `calculate_changes`, `filter_valid_documents`, `group_by_category`. Name intermediate values according to what they represent. Prefer guard clauses when they clarify the main path. Do not replace ordinary decisions with strategy classes or dispatch registries without a concrete need.

```python
subtotal = sum(item.price * item.quantity for item in items)
discounted_total = subtotal * (1 - discount)
total = discounted_total * (1 + tax_rate)
```

Document contracts, units, invariants, ownership, ordering and non-obvious reasons. Do not narrate syntax. Keep public signatures stable unless the requested behavior requires changing them.
