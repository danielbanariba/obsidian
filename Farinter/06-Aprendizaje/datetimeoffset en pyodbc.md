---
tags: [aprendizaje, pyodbc, sql-server, datetimeoffset]
tema: sql-server
fecha: 2026-04-05
---

# datetimeoffset en pyodbc

## Concepto
pyodbc no soporta `datetimeoffset` (SQL type -155) nativamente. Al leer una columna con ese tipo, tira `ODBC SQL type -155 is not yet supported`.

## Por qué es importante
Muchas tablas de Dagster (como `client_to_call`) usan `datetimeoffset` para timestamps. Sin el fix, no podés leer esas tablas via MCP.

## Ejemplo práctico
```python
import struct

def _convert_datetimeoffset(raw: bytes) -> str:
    # El valor llega como struct binario de 20 bytes
    parts = struct.unpack("<6hIhh", raw)
    y, mon, d, h, mi, s, frac, tz_h, tz_m = parts
    us = frac // 1000
    sign = "+" if tz_h >= 0 else "-"
    return f"{y:04d}-{mon:02d}-{d:02d} {h:02d}:{mi:02d}:{s:02d}.{us:06d}{sign}{abs(tz_h):02d}:{abs(tz_m):02d}"

conn = pyodbc.connect(conn_str)
conn.add_output_converter(-155, _convert_datetimeoffset)
```

NO funciona: `val.decode("utf-8")` — el valor NO es UTF-8, es un struct binario.

## Links
- [[MCP Improvements]]
