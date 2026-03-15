---
name: arrow-ipc-transport
description: "Serves Apache Arrow IPC binary streams over HTTP for zero-copy columnar data transfer between backend and frontend"
category: integrations
difficulty: advanced
tags: [apache-arrow, ipc, binary-transport, columnar, zero-copy, pyarrow]
stack: [python-3.12, pyarrow, apache-arrow-js]
---

# Arrow IPC Transport

You are a data engineer. When implementing Arrow IPC as a binary transport layer:

## Backend: Zero-Copy from DuckDB to Arrow IPC

```python
import pyarrow as pa
import pyarrow.ipc as ipc


def execute_to_arrow(sql: str) -> pa.Table:
    """Execute SQL and get Arrow table (zero-copy from DuckDB)."""
    conn = get_read_connection()
    return conn.execute(sql).to_arrow_table()


def execute_to_arrow_ipc(sql: str) -> bytes:
    """Serialize Arrow table to IPC stream bytes."""
    table = execute_to_arrow(sql)
    sink = pa.BufferOutputStream()
    writer = ipc.new_stream(sink, table.schema)
    writer.write_table(table)
    writer.close()
    return sink.getvalue().to_pybytes()
```

## Backend: Serve Binary Response

```python
from litestar import Controller, post, Response

@post("/api/arrow/from-query")
async def from_query(self, data: QueryRequest) -> Response:
    raw = execute_to_arrow_ipc(data.sql)
    return Response(
        content=raw,
        media_type="application/vnd.apache.arrow.stream",
        headers={"Content-Length": str(len(raw))},
    )
```

## Frontend: Decode Arrow IPC in Browser

```javascript
import { tableFromIPC } from 'apache-arrow'

async function fetchArrowData(sql) {
  const response = await fetch('/api/arrow/from-query', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ sql }),
  })
  const buffer = await response.arrayBuffer()
  const table = tableFromIPC(new Uint8Array(buffer))

  // Access columns
  const column = table.getChild('column_name')
  for (let i = 0; i < table.numRows; i++) {
    console.log(column.get(i))
  }
  return table
}
```

## Arrow Table Introspection

```python
def get_table_info(table_name: str) -> dict:
    arrow_table = execute_to_arrow(f'SELECT * FROM "{table_name}"')
    return {
        "num_rows": arrow_table.num_rows,
        "num_columns": arrow_table.num_columns,
        "total_bytes": arrow_table.nbytes,
        "columns": [{
            "name": f.name,
            "type": str(f.type),
            "nullable": f.nullable,
            "null_count": arrow_table.column(i).null_count,
            "nbytes": arrow_table.column(i).nbytes,
        } for i, f in enumerate(arrow_table.schema)],
    }
```

## Buffer Layout Visualization

```python
def get_buffer_layout(table_name: str) -> dict:
    arrow_table = execute_to_arrow(f'SELECT * FROM "{table_name}"')
    columns = []
    for i, field in enumerate(arrow_table.schema):
        col = arrow_table.column(i)
        chunks = []
        for j, chunk in enumerate(col.chunks):
            buffers = [{"index": k, "size": b.size if b else 0, "is_null": b is None}
                       for k, b in enumerate(chunk.buffers())]
            chunks.append({"index": j, "length": len(chunk), "buffers": buffers})
        columns.append({"name": field.name, "type": str(field.type), "chunks": chunks})
    return {"columns": columns}
```

## Performance Benchmarking: Arrow IPC vs JSON

```python
import time, json

def benchmark(sql: str, iterations: int = 10) -> dict:
    # Arrow IPC timing
    arrow_times = []
    for _ in range(iterations):
        t0 = time.perf_counter()
        raw = execute_to_arrow_ipc(sql)
        arrow_times.append((time.perf_counter() - t0) * 1000)

    # JSON timing
    json_times = []
    for _ in range(iterations):
        t0 = time.perf_counter()
        data = json.dumps(execute_query(sql), default=str).encode()
        json_times.append((time.perf_counter() - t0) * 1000)

    return {
        "arrow_avg_ms": sum(arrow_times) / len(arrow_times),
        "json_avg_ms": sum(json_times) / len(json_times),
        "arrow_bytes": len(raw),
        "json_bytes": len(data),
        "speedup": (sum(json_times) / len(json_times)) / (sum(arrow_times) / len(arrow_times)),
    }
```

## Why Arrow IPC over JSON?

| Metric | Arrow IPC | JSON |
|--------|-----------|------|
| Format | Binary columnar | Text row-oriented |
| Size | 3-5x smaller | Larger (string encoding) |
| Speed | 3-5x faster | Slower (parse overhead) |
| Types | Preserved (int32, float64, date) | Everything is string/number |
| Zero-copy | Yes (DuckDB → Arrow) | No (serialize → parse) |

## npm dependency

```json
{ "dependencies": { "apache-arrow": "^18.0.0" } }
```
