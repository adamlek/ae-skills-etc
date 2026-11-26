# Composition patterns

## Design at multiple levels

Identify the required workflow top-down, then implement and reuse its operations bottom-up. Each level should hide implementation detail while preserving the information necessary to understand its behavior.

```python
def prepare_dataset(path: Path) -> list[DocumentGroup]:
    documents = read_data(path)
    normalized = normalize_documents(documents)
    valid = filter_valid_documents(normalized)
    return group_by_category(valid)


def generate_dataset(path: Path, client: LLMClient) -> list[Response]:
    groups = prepare_dataset(path)
    prompts = create_prompts(groups)
    return generate_responses(prompts, client)


def process_dataset(source: Path, destination: Path, client: LLMClient) -> None:
    responses = generate_dataset(source, client)
    save_results(responses, destination)
```

Illustrative domain types and operations above belong to the application. Do not extract a stage merely to rename another function. Inline a simple composition such as `sorted(set(values))` when clearer.

## Explicit flow

Prefer passing data between operations rather than mutating a shared PipelineContext. A structured object is appropriate if it represents genuine related domain data. Branches, loops and early exits are valid pipeline orchestration; not all workflows are linear.

## Plan and apply

Compute decisions once and reuse them for dry-runs, execution and reporting.

```python
def reconcile_access(desired: AccessPolicy, repository: PolicyRepository, *, dry_run: bool) -> list[Change]:
    current = repository.read()
    changes = calculate_changes(current, desired)
    if not dry_run:
        repository.apply(changes)
    return changes
```

Keep `calculate_changes` independent of I/O. A dry-run must not mutate external state. Real application may require revalidation, locking or transactions if state can change between planning and applying.

## Program organization

Start with cohesive code; split modules when stable responsibilities emerge. Example: models.py, storage.py, transformations.py, generation.py, pipelines.py, main.py. This is not a required template. Initialize dependencies at entry points and keep orchestration visible. Avoid generic folders or architecture layers without concrete meaning.

## Reuse and modification

Inspect existing operations and callers. Reuse equivalent contracts, extend an operation when the behavior belongs to it, and create a new function for a distinct responsibility. Do not unify independent domain rules merely because their implementations look similar. Modify the responsible operation instead of creating a parallel execution path; preserve public contracts and remove superseded code only when safe.
