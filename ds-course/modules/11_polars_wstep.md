## Wstęp do Polars

Polars to wydajna biblioteka do analizy danych w Rust/Python z API inspirowanym `pandas`, ale oparte na silniku kolumnowym i wyrażeniach. Wspiera tryb eager i lazy, optymalizacje query i wielowątkowość.

Cele modułu:
- Podstawy `pl.DataFrame`, `Series`, `Expr` i różnice względem `pandas`.
- Tryb lazy (`LazyFrame`), wyrażenia, optymalizacje.
- Wczytywanie/zapisywanie danych (CSV/Parquet), grupowania, złączenia, pivot, selekcje kolumnowe.

Instalacja: `pip install polars pyarrow`.

### Podstawy – eager API
```python
import polars as pl

df = pl.DataFrame({
    "city": ["Kraków", "Warszawa", "Gdańsk"],
    "pop": [0.77, 1.86, 0.47],
})
print(df)
print(df.select(pl.col("pop").mean()))
print(df.with_columns((pl.col("pop") * 1000).alias("pop_k")))
```

### Selekcja i filtrowanie (wyrażenia)
```python
df2 = df.select(
    pl.all(),
    (pl.col("pop") * 1000).alias("pop_k"),
).filter(pl.col("pop") > 0.5)
```

### Groupby i agregacje
```python
sales = pl.DataFrame({
    "city": ["K", "K", "W", "W"],
    "q": ["Q1", "Q2", "Q1", "Q2"],
    "rev": [100, 120, 200, 210],
})

agg = sales.group_by(["city", "q"]).agg(
    pl.col("rev").sum().alias("rev_sum"),
    pl.col("rev").mean().alias("rev_mean"),
)
```

### Join i pivot
```python
customers = pl.DataFrame({"id": [1,2,3], "name": ["Ana","Ben","Cem"]})
orders = pl.DataFrame({"id": [101,102,103], "customer_id": [1,1,3], "value": [50,70,30]})

joined = orders.join(customers, left_on="customer_id", right_on="id", how="left")

pivoted = sales.pivot(index="city", columns="q", values="rev")
```

### Lazy API i optymalizacje
```python
lazy = (
    pl.scan_csv("data/sales.csv")
    .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
    .group_by_dynamic(index_column="order_date", every="1mo")
    .agg(pl.col("revenue").sum().alias("rev"))
)

result = lazy.collect()  # wykonanie po optymalizacji
```

### IO i typy
```python
df_csv = pl.read_csv("data/customers.csv", dtypes={"customer_id": pl.Int64, "city": pl.Utf8})
df_parq = pl.read_parquet("data/events.parquet")
df_csv.write_parquet("artifacts/customers.parquet")
```

### Ćwiczenia
- Wczytaj CSV do `LazyFrame`, dodaj kolumnę `revenue = Quantity*Price`, zagreguj miesięcznie i zapisz do Parquet.
- Użyj wyrażeń do policzenia `zscore` dla kolumny liczbowej (średnia i std z `pl.mean`, `pl.std`).
- Połącz ramki `orders` i `customers`, następnie zrób pivot po miesiącach i policz sumę wartości.

