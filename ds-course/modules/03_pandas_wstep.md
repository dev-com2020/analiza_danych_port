## Wstęp do Pandas

Cele modułu:
- Poznać `Series` i `DataFrame`.
- Pozyskiwać dane z plików, zasobów WWW i baz danych.

### Series i DataFrame
```python
import pandas as pd

s = pd.Series([10, 20, 30], index=["a", "b", "c"], name="score")
print(s["b"])          # indeksowanie po etykiecie

df = pd.DataFrame({
    "city": ["Kraków", "Warszawa", "Gdańsk"],
    "pop": [0.77, 1.86, 0.47],
})
print(df.head())
print(df.dtypes)
```

### Pozyskiwanie danych – pliki
```python
df_csv = pd.read_csv("data/customers.csv")
df_excel = pd.read_excel("data/sales.xlsx", sheet_name="Q1")
df_parquet = pd.read_parquet("data/events.parquet")

# parametry przydatne w read_csv
df_csv2 = pd.read_csv(
    "data/sales.csv",
    sep=",",
    parse_dates=["order_date"],
    dtype={"customer_id": "Int64"},
    na_values=["", "NA", "null"],
)
```

### Pozyskiwanie danych – zasoby w internecie
```python
import pandas as pd

url = "https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv"
tips = pd.read_csv(url)
print(tips.head())
```

### Pozyskiwanie danych – bazy danych (SQL)
```python
import pandas as pd
from sqlalchemy import create_engine, text

# PostgreSQL (przykład DSN): postgresql+psycopg2://user:pass@host:5432/db
engine = create_engine("postgresql+psycopg2://user:pass@localhost:5432/analytics")

with engine.connect() as conn:
    df_orders = pd.read_sql(text("SELECT * FROM orders WHERE order_date >= :d"), conn, params={"d": "2024-01-01"})

print(df_orders.shape)
```

W praktyce kluczowe są poprawne typy kolumn, jawne traktowanie `NaN` oraz kontrola pamięci (`usecols`, `dtype`, chunking przez `chunksize`).
