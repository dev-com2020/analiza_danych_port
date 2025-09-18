## Pandas vs Polars – porównanie pracy z danymi

Cele modułu:
- Zrozumieć różnice architektonalne i ich wpływ na wydajność i ergonomię.
- Porównać typowe zadania: wczytywanie, filtrowanie, agregacja, join, pivot.
- Poznać wzorce: kiedy wybrać Pandas, kiedy Polars, a kiedy łączyć oba.

### Różnice architektonalne (skrót)
- Pandas: in-memory, wierszocentryczne operacje, bogate API, ogromny ekosystem; od Pandas 2.x wsparcie backendu Arrow.
- Polars: kolumnowy silnik, wyrażenia, lazy plan z optymalizatorem (projection/predicate pushdown), wielowątkowość default.

### API porównanie – przykłady
```python
# Wczytanie i podstawowa agregacja
# Pandas
import pandas as pd
df_pd = pd.read_csv("data/sales.csv", usecols=["order_date","Quantity","Price"], parse_dates=["order_date"])
df_pd["revenue"] = df_pd["Quantity"] * df_pd["Price"]
by_month_pd = df_pd.set_index("order_date").resample("M")["revenue"].sum().reset_index()

# Polars (lazy, pushdown)
import polars as pl
by_month_pl = (
    pl.scan_csv("data/sales.csv")
    .select(["order_date","Quantity","Price"])  # projection pushdown
    .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
    .group_by_dynamic(index_column="order_date", every="1mo")
    .agg(pl.col("revenue").sum().alias("revenue"))
).collect()
```

```python
# Join
# Pandas
orders = pd.read_csv("data/orders.csv")
customers = pd.read_csv("data/customers.csv")
joined_pd = orders.merge(customers, left_on="customer_id", right_on="customer_id", how="left")

# Polars
orders_pl = pl.scan_csv("data/orders.csv")
customers_pl = pl.scan_csv("data/customers.csv")
joined_pl = orders_pl.join(customers_pl, left_on="customer_id", right_on="customer_id", how="left").collect()
```

### Benchmark – wzorzec pomiaru
Uwaga: wyniki zależą od wersji bibliotek, sprzętu, formatu danych i rozmiaru. Mierz lokalnie.
```python
import time
import pandas as pd
import polars as pl

def bench_pandas():
    t0 = time.perf_counter()
    df = pd.read_csv("data/huge.csv", usecols=["order_date","Quantity","Price"], parse_dates=["order_date"])  # noqa
    df["revenue"] = df["Quantity"] * df["Price"]
    res = df.set_index("order_date").resample("M")["revenue"].sum()
    return time.perf_counter() - t0

def bench_polars():
    t0 = time.perf_counter()
    lf = (
        pl.scan_csv("data/huge.csv")
        .select(["order_date","Quantity","Price"])  # pushdown
        .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
        .group_by_dynamic(index_column="order_date", every="1mo")
        .agg(pl.col("revenue").sum())
    )
    lf.collect(streaming=True)
    return time.perf_counter() - t0

tp = bench_pandas(); tl = bench_polars()
print({"pandas_s": tp, "polars_s": tl, "speedup_x": tp/max(tl, 1e-9)})
```

### Kiedy Pandas, kiedy Polars?
- Pandas: bogate API, kompatybilność bibliotek, ad-hoc analiza w Notebookach, mniejsze dane, prace na `Excel`.
- Polars: duże dane, przetwarzanie wsadowe, pipeline’y i ETL, szybkie agregacje, optymalizacje pushdown.
- Hybryda: wczytaj i przetwórz w Polars (lazy), a wynik przekonwertuj do Pandas do dalszych narzędzi (`to_pandas`).

### Transfer danych między Pandas i Polars
```python
import pandas as pd, polars as pl
df_pd = pd.read_parquet("data/part.parquet")
df_pl = pl.from_pandas(df_pd)  # -> Polars
df_back = df_pl.to_pandas()    # -> Pandas
```

### Ćwiczenia
- Uruchom benchmark na swoim sprzęcie dla CSV i Parquet (porównaj wyniki).
- Zaimplementuj pipeline: filtr -> join -> agregacja w obu bibliotekach i porównaj czas.
- Zmniejsz zużycie pamięci w Pandas (`category`, `Float32`) i zbadaj wpływ na czas.
