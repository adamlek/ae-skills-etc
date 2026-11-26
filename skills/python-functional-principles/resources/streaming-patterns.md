# Streaming patterns

Use eager collections for small data when simplest. Prefer lazy iteration or batching when datasets are large or the input is unbounded.

```python
def filter_valid(documents: Iterable[Document]) -> Iterator[Document]:
    return (document for document in documents if document.valid)


def normalize_documents(documents: Iterable[Document]) -> Iterator[Document]:
    return (normalize_document(document) for document in documents)
```

Compose lazy stages without materializing every intermediate. Iterators may be single-use; generator bodies run when consumed, not necessarily when created. Keep resources alive for the entire consumption period.

```python
def process_file(source: Path, destination: Path) -> None:
    with source.open(encoding='utf-8') as input_file:
        documents = (parse_document(line) for line in input_file)
        valid = filter_valid(documents)
        normalized = normalize_documents(valid)
        write_documents(normalized, destination)
```

The writer must consume the iterator before the file closes. Do not use the same source and destination unless the implementation explicitly handles safe replacement.

Materialize when sorting, indexing, repeated traversal, complete batches, size or independent ownership are required. Batch according to resource limits and downstream contracts; avoid unbounded queues.

## Avoid repeated work

- Reuse initialized clients and parsed data.
- Build indices for repeated lookups.
- Reconcile actual changes instead of rescanning or reapplying unchanged state when reliable state tracking exists.
- Use incremental processing only with a defined invalidation and recovery model; account for external drift when it matters.
- Cache only when reuse justifies added state and invalidation.
- Avoid quadratic copying and unnecessary collection conversions.

Measure real bottlenecks when possible. Prefer algorithmic and data-flow improvements to speculative micro-optimizations. Preserve correctness and failure guarantees while optimizing.
