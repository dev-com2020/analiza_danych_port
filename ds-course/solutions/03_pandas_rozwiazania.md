## Rozwiązania – 03 Pandas

```python
import pandas as pd
from sqlalchemy import create_engine, text

# tips z URL i braki
url = "https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv"
tips = pd.read_csv(url)
print(tips.isna().mean())

# smoker jako kategoria i agregacja
tips["smoker"] = tips["smoker"].astype("category")
agg = tips.groupby(["smoker", "day"])['tip'].mean().reset_index()
print(agg)

# CSV z usecols i dtype
df_csv = pd.read_csv(
    "data/customers.csv",
    usecols=["customer_id", "city", "age"],
    dtype={"customer_id": "Int64", "city": "string", "age": "Int64"}
)

# Join DB + CSV (przykład, wymaga działającej bazy)
# engine = create_engine("postgresql+psycopg2://user:pass@localhost:5432/analytics")
# with engine.connect() as conn:
#     orders = pd.read_sql(text("SELECT order_id, customer_id, value FROM orders"), conn)
# merged = orders.merge(df_csv, left_on="customer_id", right_on="customer_id", how="left")
# print(merged.head())
```
