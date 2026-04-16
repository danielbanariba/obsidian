---
tags: [aprendizaje, polars, memoria, python]
tema: polars
fecha: 2026-04-03
---

# Polars COW y clone() innecesarios

## Qué aprendí
Polars usa Copy-on-Write (COW). Cuando hacés `df.with_columns(...)`, retorna un DataFrame NUEVO que comparte los buffers de las columnas no modificadas con el original. No necesitás hacer `df.clone()` antes de `with_columns()`.

## Por qué es importante
Cada `clone()` duplica la memoria del DataFrame completo. En un DataFrame de 1GB, dos clones = 3GB de pico. Eso fue lo que causó el OOM en Demanda Limpia.

## Ejemplo práctico
```python
# MAL — desperdicia memoria
df = df.clone().with_columns(
    pl.col("float_col").fill_nan(None)
)

# BIEN — with_columns ya retorna nuevo DF
df = df.with_columns(
    pl.col("float_col").fill_nan(None)
)
```

## Links
- [[OOM Demanda Limpia]]
- Polars docs: https://docs.pola.rs/user-guide/concepts/data-types-and-structures/
