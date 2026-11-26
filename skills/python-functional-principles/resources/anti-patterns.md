# Anti-patterns

Review total complexity, not just duplication or line count. Each abstraction should hide meaningful detail, establish a useful contract or consolidate a real responsibility.

| Signal | Prefer | Exception |
|---|---|---|
| Stateless class used as namespace | Module functions | Required existing interface |
| Wrapper only forwarding arguments | Call original operation | Stable public or integration boundary |
| Generic pipeline engine | Ordinary function composition | Dynamic workflows, durable execution or scheduling requirements |
| Universal mutable context | Explicit data and dependencies | Genuine shared domain or execution contract |
| Helper for every expression | Inline simple expressions | Meaningful reusable predicate |
| Generic function with many callbacks and flags | Concrete domain operations | Stable demonstrated shared contract |
| Speculative provider interfaces | Existing concrete client | Actual multiple implementations or integration needs |
| Generic utils/services/managers folders | Cohesive responsibility-based modules | Established project organization with clear meaning |
| Repeated defensive checks | Validation at appropriate boundaries | Changed state, separate trust boundaries or real risk |
| Patch or parallel execution path | Modify responsible operation | Required migration or compatibility contract |
| Strict immutability via repeated copies | Local owned mutation or suitable data structure | Explicit persistent-data requirements |

## Example: speculative generalization

Avoid `process_items(items, transform, predicate=None, aggregator=None, postprocess=None)` when callers merely need unrelated concrete operations.

```python
def extract_active_users(users: Iterable[User]) -> list[User]:
    return [user for user in users if user.active]


def calculate_total_weight(documents: Iterable[Document]) -> int:
    return sum(document.weight for document in documents)
```

Generalize when actual usage reveals the same responsibility and stable contract. Do not unify different domain rules merely because they look alike.

## Simplification review

Identify each added function, class, dependency, module, configuration option and compatibility branch. Ask what current requirement it serves and whether the same behavior can be expressed using an existing construct. Remove obsolete or redundant code only after checking callers and guarantees. Preserve necessary complexity for correctness, security, performance and operations.
