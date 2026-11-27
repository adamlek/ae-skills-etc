# Pipeline design

How to structure a data-processing program: what goes in each step, where I/O lives, and how to test it.

## Workflow

1. **List inputs, outputs, and side effects.** What does the program read? What does it write? What does it call?
2. **Write the outline.** One line of verbs: `fetch -> parse -> dedupe -> enrich -> write`.
3. **Define the record shape.** A frozen `dataclass` or `NamedTuple` for anything with more than two fields. Raw `dict`s are fine at the source boundary and for short scripts.
4. **Write the transforms first.** They are pure and easiest to test.
5. **Write sources and sinks.** Keep them thin: read and parse, or format and write.
6. **Connect them in a pipeline function.**
7. **Add a thin entry point.** It parses arguments, builds clients, calls the pipeline, and reports errors.
8. **Simplify.** Remove wrappers that only forward a call, helpers used once that do not clarify, and options nobody asked for.

## What makes a good step

- **One reason to change.** "Parse a log line" and "decide if a line is an error" are separate steps if either may change on its own.
- **Named after what it does to the data.** `keep_recent`, `group_by_user`, `to_csv_row`. Avoid `process`, `handle`, `do_stuff`.
- **Explicit inputs and outputs.** Values come in as parameters and leave as return values. No reading globals, no mutating arguments.
- **Not artificially small.** A step that wraps a single expression used once should usually stay inline.

## Signatures

Accept the most general type you need, and return the most specific type you produce.

```python
from collections.abc import Iterable, Iterator, Sequence

def keep_errors(lines: Iterable[str]) -> Iterator[str]:       # streams
    return (line for line in lines if "ERROR" in line)

def median(values: Sequence[float]) -> float:                   # needs len and indexing
    ordered = sorted(values)
    mid = len(ordered) // 2
    return ordered[mid] if len(ordered) % 2 else (ordered[mid - 1] + ordered[mid]) / 2
```

`Iterable` in a signature promises a single pass. `Sequence` tells the caller you need the whole thing.

Put configuration after the data, as keyword-only arguments:

```python
def keep_recent(events: Iterable[Event], *, since: datetime) -> Iterator[Event]: ...
```

## Sources and sinks

A **source** opens a resource, parses raw data into records, and validates it. It is the only step that should know about file formats, HTTP, or SQL.

A **sink** takes records and writes them out. It is the only step that should know about output formats.

```python
def read_events(path: Path) -> Iterator[Event]:
    with path.open(encoding="utf-8") as f:
        for line_no, line in enumerate(f, start=1):
            try:
                yield Event.from_json(line)
            except (ValueError, KeyError) as exc:
                raise ValueError(f"{path}:{line_no}: bad event") from exc
```

## Dependencies

Pass external services into the pipeline as parameters. A plain callable is usually enough; you do not need an interface class.

```python
def enrich_users(users: Iterable[User], lookup: Callable[[str], Profile]) -> Iterator[EnrichedUser]:
    return (EnrichedUser(user, lookup(user.id)) for user in users)
```

In tests, `lookup` can be a `dict.__getitem__` or a lambda.

## Configuration

- Read config (CLI arguments, environment, files) once, in the entry point.
- Pass individual values to the pipeline. Pass a small frozen config object only when there are many related values.
- Transforms never read environment variables or files themselves.

## Errors

Decide per pipeline what a bad record means:

| Policy | When | How |
|---|---|---|
| **Fail fast** | Bad input means a bug or corrupt data | Raise in the source with context (file, line, id) |
| **Skip and report** | Some bad records are expected | Split into good and bad streams; log or write the bad ones |
| **Retry** | Transient failures at a boundary | Retry inside the source or sink, never inside a pure transform |

Skip-and-report without losing the errors:

```python
def parse_all(lines: Iterable[str]) -> tuple[list[Event], list[tuple[str, str]]]:
    good, bad = [], []
    for line in lines:
        try:
            good.append(Event.from_json(line))
        except ValueError as exc:
            bad.append((line, str(exc)))
    return good, bad
```

Never use a bare `except:` or `except Exception: pass` inside a pipeline. It hides bugs in later steps.

## Program entry point

```python
def main() -> None:
    args = parse_args()
    client = make_llm_client(args.model)
    summarize_file(args.source, args.destination, client.generate,
                   min_score=args.min_score, batch_size=args.batch_size)

if __name__ == "__main__":
    main()
```

The entry point holds no processing logic. If you deleted it, the pipeline would still be usable from another script or a test.

## Organization

- Start with one module. Split only when a file becomes hard to navigate, and split by responsibility (`sources.py`, `transforms.py`, `sinks.py`), not by layer names.
- Avoid managers, registries, factories, base classes, and pipeline frameworks unless a concrete requirement needs them.

## Testing

| Level | What | How |
|---|---|---|
| Transform | Pure function | Literal input, assert on output. Wrap lazy output in `list()`. |
| Pipeline | Steps wired correctly | Fake source (a list), fake client (a lambda), sink to a list or `tmp_path` |
| Boundary | Real files, APIs, DBs | A few integration tests; mark them so they can be skipped |

```python
def test_keep_confident():
    items = [Item("a", 0.9), Item("b", 0.1)]
    assert list(keep_confident(items, min_score=0.5)) == [Item("a", 0.9)]

def test_summarize_file(tmp_path):
    src = tmp_path / "in.jsonl"
    src.write_text('{"text": "a", "score": 0.9}\n')
    dst = tmp_path / "out.jsonl"
    summarize_file(src, dst, generate=lambda prompt: "SUMMARY")
    assert dst.read_text() == '"SUMMARY"\n'
```

Test edge cases that pipelines commonly get wrong: empty input, a single item, a final partial batch, and duplicate keys.
