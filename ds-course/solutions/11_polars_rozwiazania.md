## Rozwiązania – 11 Polars

```python
import polars as pl

# Lazy agregacja miesięczna i zapis Parquet
lazy = (
    pl.scan_csv("data/sales.csv")
    .with_columns((pl.col("Quantity") * pl.col("Price")).alias("revenue"))
    .with_columns(pl.col("order_date").str.strptime(pl.Datetime, fmt=None, strict=False))
    .group_by_dynamic(index_column="order_date", every="1mo")
    .agg(pl.col("revenue").sum().alias("rev"))
)
monthly = lazy.collect()
monthly.write_parquet("artifacts/revenue_by_month.parquet")

# z-score
df = pl.DataFrame({"x": [1.0, 2.0, 3.0, 4.0]})
df_z = df.with_columns(((pl.col("x") - pl.col("x").mean()) / pl.col("x").std()).alias("z"))
print(df_z)

# join i pivot
customers = pl.DataFrame({"id": [1,2,3], "name": ["Ana","Ben","Cem"]})
orders = pl.DataFrame({
    "order_id": [101,102,103,104],
    "customer_id": [1,1,3,2],
    "value": [50,70,30,90],
    "order_month": ["2024-01","2024-02","2024-02","2024-01"],
})

joined = orders.join(customers, left_on="customer_id", right_on="id", how="left")
pivot = joined.pivot(index="name", columns="order_month", values="value", aggregate_fn="sum")
print(pivot)
```
