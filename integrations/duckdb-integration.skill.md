---
name: duckdb-integration
description: "Integrates DuckDB as an embedded OLAP database with thread-safe connection management, auto-seeding, and schema introspection"
category: integrations
difficulty: intermediate
tags: [duckdb, olap, embedded-database, columnar, analytics]
stack: [python-3.12, duckdb]
---

# DuckDB Integration

You are a data engineer. When integrating DuckDB as an embedded analytical database:

## Connection Manager (Singleton)

```python
import os
import threading
import duckdb

DB_PATH = os.environ.get("DUCKDB_PATH", "app.duckdb")

_lock = threading.Lock()
_connection: duckdb.DuckDBPyConnection | None = None


def get_connection() -> duckdb.DuckDBPyConnection:
    global _connection
    if _connection is None:
        with _lock:
            if _connection is None:
                _connection = duckdb.connect(DB_PATH)
                _seed_demo_data(_connection)
    return _connection


def get_read_connection() -> duckdb.DuckDBPyConnection:
    """Return a cursor for read operations (supports concurrent reads)."""
    return get_connection().cursor()
```

## Auto-Seeding

```python
def _seed_demo_data(conn: duckdb.DuckDBPyConnection) -> None:
    tables = [r[0] for r in conn.execute(
        "SELECT table_name FROM information_schema.tables WHERE table_schema='main'"
    ).fetchall()]
    if "my_table" in tables:
        return  # already seeded

    conn.execute("""
        CREATE TABLE my_table AS
        SELECT i AS id,
               CASE (i % 3) WHEN 0 THEN 'A' WHEN 1 THEN 'B' ELSE 'C' END AS category,
               round(random() * 100, 2)::DOUBLE AS value
        FROM generate_series(1, 1000) t(i)
    """)
```

## Schema Introspection

```python
def get_schemas() -> list[dict]:
    conn = get_read_connection()
    tables = conn.execute("""
        SELECT table_name FROM information_schema.tables
        WHERE table_schema = 'main' AND table_name NOT LIKE '\\_%' ESCAPE '\\'
        ORDER BY table_name
    """).fetchall()

    result = []
    for (name,) in tables:
        cols = conn.execute("""
            SELECT column_name, data_type, is_nullable
            FROM information_schema.columns
            WHERE table_name = ? ORDER BY ordinal_position
        """, [name]).fetchall()
        row_count = conn.execute(f'SELECT count(*) FROM "{name}"').fetchone()[0]
        result.append({
            "name": name,
            "row_count": row_count,
            "columns": [{"name": c[0], "type": c[1], "nullable": c[2] == "YES"} for c in cols],
        })
    return result
```

## CSV/Parquet Import via PyArrow

```python
import pyarrow.csv as pa_csv
import pyarrow.parquet as pq
import pyarrow as pa

def import_file(filename: str, data: bytes, table_name: str) -> dict:
    conn = get_connection()
    buf = pa.py_buffer(data)
    reader = pa.BufferReader(buf)

    if filename.endswith(".parquet"):
        arrow_table = pq.read_table(reader)
    elif filename.endswith(".csv"):
        arrow_table = pa_csv.read_csv(reader)
    else:
        return {"error": "Unsupported file type"}

    conn.execute(f'DROP TABLE IF EXISTS "{table_name}"')
    conn.execute(f'CREATE TABLE "{table_name}" AS SELECT * FROM arrow_table')
    row_count = conn.execute(f'SELECT count(*) FROM "{table_name}"').fetchone()[0]
    return {"table": table_name, "row_count": row_count}
```

## DuckDB-Specific SQL Features

```sql
-- Window function with QUALIFY
SELECT * FROM sales QUALIFY ROW_NUMBER() OVER (PARTITION BY category ORDER BY amount DESC) = 1;

-- PIVOT
PIVOT sales ON category USING SUM(amount);

-- Nested types
SELECT [1,2,3] AS list_col, {'a': 1} AS struct_col;

-- generate_series for synthetic data
SELECT * FROM generate_series(1, 10000);
```

## Thread Safety Notes

- DuckDB supports one writer at a time — protect writes with `threading.Lock`
- Read cursors via `.cursor()` can run concurrently
- DuckDB is in-process — no network, zero-copy with Arrow via `to_arrow_table()`
