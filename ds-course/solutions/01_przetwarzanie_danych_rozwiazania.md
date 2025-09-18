## Rozwiązania – 01 Przetwarzanie danych

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("data/transactions.csv")

# typy i braki
print(df.dtypes)
print(df.isna().sum())

# data na datetime i filtrowanie braków
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
df = df.dropna(subset=["order_date", "quantity", "price"])

# przychód i agregacja miesięczna
df["revenue"] = df["quantity"] * df["price"]
by_month = df.set_index("order_date").resample("M")["revenue"].sum()
by_month.to_csv("artifacts/revenue_by_month.csv")

# wykres
by_month.plot(title="Przychód miesięczny")
plt.tight_layout(); plt.show()
```
