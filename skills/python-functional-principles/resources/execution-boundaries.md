# Execution boundaries

## Functional core, imperative shell

Obtain external state, compute a decision, then apply it. Pass external information such as time or configuration into computations.

```python
def is_expired(item: Item, now: datetime) -> bool:
    return item.expires_at <= now
```

Obtain `now` at the boundary. An explicit effectful dependency makes dependencies visible but does not make a function pure.

## Dependencies and ownership

Initialize expensive clients and resources once at appropriate boundaries; reuse them where safe.

```python
def generate_responses(prompts: Iterable[str], client: LLMClient) -> list[Response]:
    return [client.generate(prompt) for prompt in prompts]


def main() -> None:
    config = load_config()
    with create_client(config) as client:
        process_dataset(config.source, config.destination, client)
```

Use ordinary parameter passing before dependency injection frameworks. Do not pass dependencies through operations that do not need them. The acquirer owns resource cleanup unless ownership is explicitly transferred. Do not close caller-owned resources. Use context managers for files, connections and clients where supported.

Group related settings in a configuration dataclass only when they form a meaningful reusable contract. Pass individual values to small computations rather than entire application settings. Avoid options for hypothetical requirements.

## Concurrency

Keep execution policies separate from transformations. Introduce concurrency only when the workload benefits. For async clients, expose async functions rather than running event loops inside helpers.

```python
async def generate_response(prompt: str, client: AsyncLLMClient) -> Response:
    return await client.generate(prompt)
```

Start an event loop only at an appropriate synchronous entry point. Use structured concurrency (e.g. TaskGroup on Python 3.11+) when suitable, bound concurrency and pending work, manage cancellation and failure, preserve required ordering, and release resources after their users finish. Async syntax does not make blocking calls non-blocking. Choose threads, processes or async I/O according to workload and resource constraints.

## External consistency

Multiple side effects are not automatically atomic. Respect existing transaction, delivery, idempotency and concurrency guarantees. Do not invent retry or rollback semantics. For destructive operations, preserve necessary confirmation, authorization, dry-run and audit safeguards.
