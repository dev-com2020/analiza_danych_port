## Rozwiązania – 12 Pandas vs Polars

```python
# Benchmark i przykładowe rozwiązania ćwiczeń
import time
import pandas as pd
import polars as pl

# 1) Benchmark CSV vs Parquet
def bench_pd_csv():
    t0 = time.perf_counter()
    df = pd.read_csv("data/huge.csv", usecols=["order_date","Quantity","Price"], parse_dates=["order_date"])  # noqa
    df["revenue"] = df["Quantity"] * df["Price"]
    res = df.set_index("order_date").resample("M")["revenue"].sum()
    return time.perf_counter() - t0

def bench_pl_csv():
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

def bench_pd_parquet():
    t0 = time.perf_counter()
    df = pd.read_parquet("data/huge.parquet", columns=["order_date","Quantity","Price"])  # noqa
    df["order_date"] = pd.to_datetime(df["order_date"])  # jeśli nie datetime
    df["revenue"] = df["Quantity"] * df["Price"]
    res = df.set_index("order_date").resample("M")["revenue"].sum()
    return time.perf_counter() - t0

def bench_pl_parquet():
    t0 = time.perf_counter()
    lf = (
        pl.scan_parquet("data/huge.parquet")
        .select(["order_date","Quantity","Price"])  # pushdown
        .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
        .group_by_dynamic(index_column="order_date", every="1mo")
        .agg(pl.col("revenue").sum())
    )
    lf.collect(streaming=True)
    return time.perf_counter() - t0

print({
    "pd_csv": bench_pd_csv(),
    "pl_csv": bench_pl_csv(),
    "pd_parquet": bench_pd_parquet(),
    "pl_parquet": bench_pl_parquet(),
})

# 2) Pipeline filtr->join->agregacja
orders_pd = pd.read_parquet("data/orders.parquet")
customers_pd = pd.read_parquet("data/customers.parquet")
res_pd = (
    orders_pd[orders_pd["order_date"] >= "2024-01-01"]
    .merge(customers_pd[["customer_id","segment"]], on="customer_id", how="left")
    .assign(revenue=lambda d: d["Quantity"] * d["Price"])
    .groupby([pd.Grouper(key="order_date", freq="M"), "segment"])['revenue']
    .sum()
    .reset_index()
)

orders_pl = pl.scan_parquet("data/orders.parquet")
customers_pl = pl.scan_parquet("data/customers.parquet").select(["customer_id","segment"])  # projection
res_pl = (
    orders_pl
    .filter(pl.col("order_date") >= pl.datetime(2024,1,1))
    .join(customers_pl, on="customer_id", how="left")
    .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
    .group_by_dynamic(index_column="order_date", every="1mo", by="segment")
    .agg(pl.col("revenue").sum().alias("revenue"))
).collect()

# 3) Pandas redukcja pamięci
df_small = pd.read_csv(
    "data/huge.csv",
    usecols=["customer_id","city","Quantity","Price"],
    dtype={"customer_id":"Int32","city":"category","Quantity":"Int16","Price":"Float32"},
)
print(df_small.dtypes)
```
