# Error handling

Distinguish normal domain outcomes from failures to fulfill a contract, and from cancellation or interruption.

```python
def find_user(users: Iterable[User], user_id: str) -> User | None:
    return next((user for user in users if user.id == user_id), None)
```

Use ordinary values for expected outcomes when the distinction is simple. Use structured outcomes when callers need different data for different results.

```python
@dataclass(frozen=True)
class Accepted:
    order: Order

@dataclass(frozen=True)
class Rejected:
    reason: str


def validate_order(order: Order) -> Accepted | Rejected:
    if not order.items:
        return Rejected('Order contains no items')
    return Accepted(order)
```

Do not introduce generic Result or monadic frameworks for every operation. Follow established project contracts.

## Exceptions

Catch exceptions only to recover, translate to a meaningful contract, add useful context, retry under a defined policy, or perform required cleanup. Prefer specific exception types. Preserve causes with `raise ... from exc` when translating. Let unexpected failures reach the boundary responsible for reporting them. Do not log and re-raise at every layer, swallow errors, or return ambiguous defaults.

## Validation and safeguards

Validate untrusted data at the appropriate boundary. Internal operations may rely on established invariants, but preserve checks at trust boundaries and where state may change. Do not remove necessary authorization, validation, safety checks, integrity checks or resource limits in the name of simplicity.

Avoid speculative compatibility branches, repeated checks of already validated invariants, fallback values that conceal faults, and unsupported retry behavior. Retry only appropriate transient failures with bounded attempts and timeouts; account for idempotency and cancellation. Never silently retry a destructive or non-idempotent action without a justified policy.
