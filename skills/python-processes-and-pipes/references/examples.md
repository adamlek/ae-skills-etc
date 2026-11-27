# Worked examples

Each example starts with its outline, then the code. Adapt the pattern rather than copying the details.

## 1. Large CSV: filter and aggregate in one pass

Outline: `read rows -> parse -> keep paid -> total by region -> print`

```python
import csv
from collections.abc import Iterable, Iterator
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class Sale:
    region: str
    status: str
    amount: float


def read_sales(path: Path) -> Iterator[Sale]:
    with path.open(newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            yield Sale(row["region"], row["status"], float(row["amount"]))


def is_paid(sale: Sale) -> bool:
    return sale.status == "paid"


def total_by_region(sales: Iterable[Sale]) -> dict[str, float]:
    totals: dict[str, float] = {}
    for sale in sales:
        totals[sale.region] = totals.get(sale.region, 0.0) + sale.amount
    return totals


def report(path: Path) -> dict[str, float]:
    sales = read_sales(path)
    paid = filter(is_paid, sales)
    return total_by_region(paid)
```

Memory stays flat: only the `totals` dict grows, and only with the number of regions.

## 2. Sorted log file: per-session summaries with `itertools.groupby`

Outline: `read lines -> parse -> group consecutive by session -> summarize -> write`

The log is already ordered by session, so `itertools.groupby` streams one session at a time.

```python
from itertools import groupby
from operator import attrgetter


def summarize_session(session_id: str, events: Iterable[Event]) -> SessionSummary:
    events = list(events)                     # one session fits in memory
    return SessionSummary(
        session_id=session_id,
        start=events[0].timestamp,
        end=events[-1].timestamp,
        errors=sum(e.level == "ERROR" for e in events),
    )


def summarize_log(source: Path, destination: Path) -> None:
    events = read_events(source)
    sessions = groupby(events, key=attrgetter("session_id"))
    summaries = (summarize_session(sid, group) for sid, group in sessions)
    write_jsonl(summaries, destination)
```

If the file were not ordered by session, `itertools.groupby` would split sessions. Then you would either sort first (if it fits in memory) or aggregate into a dict.

## 3. Paginated API: flatten, deduplicate, reshape

Outline: `fetch pages -> flatten items -> dedupe by id -> keep active -> reshape -> collect`

```python
from collections.abc import Callable, Iterator
from toolz import unique


def fetch_all(get_page: Callable[[str | None], dict]) -> Iterator[dict]:
    cursor = None
    while True:
        page = get_page(cursor)
        yield from page["items"]
        cursor = page.get("next_cursor")
        if not cursor:
            return


def to_user(raw: dict) -> User:
    return User(id=raw["id"], email=raw["contact"]["email"].lower())


def active_users(get_page: Callable[[str | None], dict]) -> list[User]:
    items = fetch_all(get_page)
    distinct = unique(items, key=lambda raw: raw["id"])
    active = (raw for raw in distinct if raw.get("status") == "active")
    return [to_user(raw) for raw in active]
```

`get_page` is injected. In tests it can return pages from a list:

```python
pages = {None: {"items": [a, b], "next_cursor": "2"}, "2": {"items": [b, c]}}
assert active_users(pages.__getitem__) == [...]
```

## 4. The same transform as a `toolz.pipe`

Outline: `keep paid -> group by region -> sum totals -> keep large`

```python
from toolz import pipe
from toolz.curried import filter, groupby, valmap, valfilter


def large_regions(orders: Iterable[Order], *, threshold: float) -> dict[str, float]:
    return pipe(
        orders,
        filter(lambda o: o.status == "paid"),
        groupby(lambda o: o.region),
        valmap(lambda region_orders: sum(o.total for o in region_orders)),
        valfilter(lambda total: total >= threshold),
    )
```

Equivalent with plain Python:

```python
def large_regions(orders: Iterable[Order], *, threshold: float) -> dict[str, float]:
    totals: dict[str, float] = {}
    for order in orders:
        if order.status == "paid":
            totals[order.region] = totals.get(order.region, 0.0) + order.total
    return {region: total for region, total in totals.items() if total >= threshold}
```

Both are fine. The plain version streams without building groups. The `pipe` version reads closer to the outline. Pick one per codebase and stay consistent.

## 5. Batch LLM calls with error capture

Outline: `read docs -> batch -> build prompt -> generate -> pair with ids -> write ok and failed`

```python
from itertools import batched


def run_batches(
    docs: Iterable[Doc],
    generate: Callable[[str], str],
    *,
    batch_size: int,
) -> Iterator[Result]:
    for batch in batched(docs, batch_size):
        ids = [doc.id for doc in batch]
        try:
            output = generate(build_prompt(batch))
        except TransientAPIError as exc:
            yield Result(ids=ids, output=None, error=str(exc))
        else:
            yield Result(ids=ids, output=output, error=None)


def process(source: Path, ok_path: Path, failed_path: Path, generate, *, batch_size=20) -> None:
    results = list(run_batches(read_docs(source), generate, batch_size=batch_size))
    write_jsonl((r for r in results if r.error is None), ok_path)
    write_jsonl((r for r in results if r.error is not None), failed_path)
```

Only the expected API error is caught, and every failure is recorded with its ids, so failed batches can be retried later. `results` is materialized because it is read twice. For very large runs, write both outputs from one loop instead.
