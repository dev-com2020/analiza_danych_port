## Rozwiązania – 04 Data Cleaning

```python
import pandas as pd

df = pd.read_csv("data/raw.csv")

# usunięcie kolumn >30% braków
na_ratio = df.isna().mean()
cols_to_drop = na_ratio[na_ratio > 0.3].index.tolist()
df = df.drop(columns=cols_to_drop)

# melt/pivot
wide = pd.DataFrame({"city":["K","K","W","W"],"Q1":[1,2,3,4],"Q2":[2,3,4,5]})
long = wide.melt(id_vars=["city"], var_name="quarter", value_name="revenue")
back = long.pivot_table(index="city", columns="quarter", values="revenue", aggfunc="sum")

# sort + rank
df_rank = df.sort_values(["score","user"], ascending=[False, True]).copy()
df_rank["rank"] = df_rank["score"].rank(ascending=False, method="dense")

# łączenie trzech ramek
orders = pd.read_csv("data/orders.csv")
customers = pd.read_csv("data/customers.csv")
products = pd.read_csv("data/products.csv")

oc = orders.merge(customers, left_on="customer_id", right_on="customer_id", how="left")
ocp = oc.merge(products, left_on="product_id", right_on="product_id", how="left")
ocp["basket_value"] = ocp["quantity"] * ocp["price"]
summary = ocp.groupby("order_id")["basket_value"].sum().reset_index()
```
