## Przygotowywanie i czyszczenie danych – Operacje i przekształcenia DataFrame

Cele modułu:
- Usuwanie kolumn i wierszy, zarządzanie brakami danych.
- Zmiana wymiarów (reshaping), pivoting.
- Rangowanie i sortowanie.
- Łączenie ramek: `concat`, `merge`, `join`.

### Usuwanie kolumn i wierszy
```python
import pandas as pd

df = pd.DataFrame({"a": [1, 2, None], "b": ["x", "y", "z"], "c": [10, 20, 30]})

df = df.drop(columns=["b"])           # usuń kolumny
df = df.dropna(subset=["a"])          # usuń wiersze z brakami w a
df = df[df["c"] > 10]                  # filtr wierszy
```

### Zmiana wymiarów – reshaping i pivoting
```python
sales = pd.DataFrame({
    "city": ["Kraków", "Kraków", "Warszawa", "Warszawa"],
    "quarter": ["Q1", "Q2", "Q1", "Q2"],
    "revenue": [100, 120, 200, 210],
})

wide = sales.pivot(index="city", columns="quarter", values="revenue")
long = wide.reset_index().melt(id_vars=["city"], var_name="quarter", value_name="revenue")

# Multiindeks i agregacje
pivot_tbl = pd.pivot_table(sales, index="city", columns="quarter", values="revenue", aggfunc="sum", margins=True)
```

### Rangowanie i sortowanie danych
```python
df = pd.DataFrame({"team": ["A", "B", "C"], "score": [88, 95, 90]})
df["rank"] = df["score"].rank(ascending=False, method="dense")
df_sorted = df.sort_values(["score", "team"], ascending=[False, True])
```

### Łączenie ramek danych
```python
import pandas as pd

customers = pd.DataFrame({"id": [1, 2, 3], "name": ["Ana", "Ben", "Cem"]})
orders = pd.DataFrame({"id": [101, 102, 103], "customer_id": [1, 1, 3], "value": [50, 70, 30]})

# concat (sklejanie wierszy/kolumn)
df_rows = pd.concat([customers, customers], axis=0, ignore_index=True)

# merge/join
df_join = orders.merge(customers, left_on="customer_id", right_on="id", how="left", suffixes=("_ord", "_cust"))

# join po indeksie
orders_idx = orders.set_index("customer_id")
customers_idx = customers.set_index("id")
df_join2 = orders_idx.join(customers_idx, how="left")
```

Pamiętaj o kontroli duplikatów kluczy oraz typów podczas łączenia ramek.

### Ćwiczenia
- Usuń kolumny o wysokim udziale braków (>30%) i uzasadnij wybór.
- Użyj `melt` i `pivot_table`, aby przejść między formą szeroką a długą.
- Posortuj dane po dwóch kolumnach i dodaj ranking z remisami (`method="dense"`).
- Połącz trzy ramki: `orders`, `customers`, `products` i policz wartość koszyka.
