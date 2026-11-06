# pyhouse

**A Python query builder for ClickHouse.** Define entities and compose analytical SQL with filters, joins, aggregation and pagination.

This is an early implementation by [Juliano Sig](https://github.com/julianosig), useful for exploring reusable data-access abstractions. The example below generates SQL without contacting a database.

## Quick start

Requires Python 3.12 or later.

```sh
git clone https://github.com/julianosig/pyhouse.git
cd pyhouse
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -e .
```

```python
from pyhouse.entities import Entity
from pyhouse.fields import Type
from pyhouse.operators import EQ
from pyhouse.query import Query


class Event(Entity):
    name = Type.String()


query = (
    Query(Event)
    .read(Event.name)
    .where(EQ(Event.name, "signup"))
    .max(10)
)
print(query.query())
```

Output (whitespace normalized):

```sql
SELECT Event.name AS name FROM Event
WHERE (Event.name = 'signup') LIMIT 10 OFFSET 0
```

`.query()` returns SQL; `.fetch()` executes it against ClickHouse and maps rows to dictionaries. Executing requires a reachable database and an existing table matching the entity.

## Implementation map

| Module | Responsibility |
| :--- | :--- |
| [entities.py](pyhouse/entities.py) | Entity metadata and create/search/save entry points |
| [fields.py](pyhouse/fields.py) | ClickHouse field types and descriptors |
| [operators.py](pyhouse/operators.py) | Equality, ranges, membership and string filters |
| [query.py](pyhouse/query.py) | Fluent selection, joins, grouping, ordering and pagination |
| [connection.py](pyhouse/connection.py) | ClickHouse driver connection and cursor |
| [paginator.py](pyhouse/paginator.py) | Pagination helpers |

`Query.sum()` builds aggregations; `combine()` adds joins; `order_by()`, `offset()` and `max()` control the result window. See the implementation for the current API.

## Configuration and current scope

Connection settings are loaded at import time from the environment and an optional local `.env` file:

| Variable | Default |
| :--- | :--- |
| `CLICKHOUSE_HOST` | `localhost` |
| `CLICKHOUSE_PORT` | `9000` (native protocol) |
| `CLICKHOUSE_USERNAME` | `default` |
| `CLICKHOUSE_PASSWORD` | `password` |
| `CLICKHOUSE_DATABASE` | `default` |

Set credentials locally to match your database before importing the package. The SQL-only example does not execute a query or require a running ClickHouse server.

SQL values are currently interpolated into strings. Use trusted inputs while experimenting; parameter binding and escaping need to be addressed before accepting untrusted values. The repository's test directory is currently a placeholder.

## Contributing

Useful next steps are configurable connection handling, parameterized queries, documented aggregate examples and integration tests against ClickHouse. Please include a small reproducer with bug reports.

## License

[MIT](LICENSE).
