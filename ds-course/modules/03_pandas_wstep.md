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

### Excel w Pandas (odczyt i zapis)
- Silniki: dla .xlsx zalecane `openpyxl` (odczyt/zapis), alternatywnie `xlsxwriter` (zapis). Zainstaluj: `pip install openpyxl xlsxwriter`.
- Dla legacy .xls możesz użyć `xlrd==1.2.0` (nowsze xlrd nie wspiera .xlsx).
```python
# odczyt wybranych arkuszy i kolumn
df_xlsx = pd.read_excel(
    "data/report.xlsx",
    sheet_name=["Sales", "Expenses"],  # lista lub nazwa
    usecols="A:D",
    dtype={"CustomerID": "Int64"},
    engine="openpyxl",
)

# praca z wieloma arkuszami via ExcelFile
with pd.ExcelFile("data/report.xlsx", engine="openpyxl") as xls:
    sales = pd.read_excel(xls, sheet_name="Sales")
    expenses = pd.read_excel(xls, sheet_name="Expenses")

# zapis do wielu arkuszy z formatowaniem przez xlsxwriter
with pd.ExcelWriter("artifacts/summary.xlsx", engine="xlsxwriter") as writer:
    sales.to_excel(writer, sheet_name="Sales", index=False)
    expenses.to_excel(writer, sheet_name="Expenses", index=False)
    # przykładowe formatowanie
    workbook = writer.book
    worksheet = writer.sheets["Sales"]
    money_fmt = workbook.add_format({"num_format": "#,##0.00"})
    worksheet.set_column("C:C", 12, money_fmt)
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

### Ćwiczenia
- Wczytaj zbiór `tips` z URL i sprawdź rozkład braków.
- Zamień typ kolumny `smoker` na kategoriczny i policz średni `tip` wg `smoker` i `day`.
- Z pliku CSV wczytaj tylko wybrane kolumny (`usecols`) i ustaw właściwe `dtype`.
- Połącz dane z bazy (np. `orders`) i z CSV (`customers`) po kluczach.
