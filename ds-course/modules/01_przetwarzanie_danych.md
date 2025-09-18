## Przetwarzanie danych

Cele modułu:
- Zrozumieć etapy przepływu danych: pozyskiwanie, wstępne czyszczenie, transformacje, walidacja, zapis.
- Poznać podstawowe narzędzia: `numpy`, `pandas` oraz dobre praktyki pracy.

### Pipeline przetwarzania danych (wysoki poziom)
- Pozyskanie: pliki (CSV/Parquet/Excel), API/HTTP, bazy danych (SQL/NoSQL).
- Walidacja i profilowanie: typy, brakujące wartości, rozkłady, wartości odstające.
- Transformacje: filtrowanie, łączenie, agregacje, zmianę kształtu (reshape/pivot), standaryzację/normalizację.
- Zapisywanie artefaktów: dane przetworzone, raporty, modele.

### Dobre praktyki
- Pracuj na kopii danych surowych, zapisuj kroki (skrypty/Notebook).
- Ustal typy kolumn i jawnie obsługuj `NaN`/wartości brakujące.
- Mierz wydajność (wektoryzacja, unikanie pętli w Pythonie) i pamięć.
- Utrzymuj deterministyczność (ziarno losowe) i wersjonuj dane/model.

### Minimalny przykład E2E
```python
import pandas as pd

# 1) Pozyskanie
df = pd.read_csv("data/sales.csv")

# 2) Walidacja (prosty przegląd)
print(df.info())
print(df.describe(include="all"))

# 3) Czyszczenie/transformacje
df = df.rename(columns={"Order Date": "order_date"})
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
df = df.dropna(subset=["order_date", "Quantity", "Price"])  # wymagane
df["revenue"] = df["Quantity"] * df["Price"]

# 4) Agregacja
by_month = df.set_index("order_date").resample("M")["revenue"].sum()

# 5) Zapis
by_month.to_csv("artifacts/revenue_by_month.csv")
```

Następne moduły pogłębiają techniki NumPy, Pandas i ML.

### Ćwiczenia
- Załaduj plik CSV z transakcjami, wylistuj typy kolumn i liczbę wartości brakujących.
- Przekonwertuj kolumnę daty na `datetime`, odfiltruj wiersze z brakami w kluczowych kolumnach.
- Dodaj kolumnę przychodu jako iloczyn ilości i ceny, policz przychód miesięczny.
- Zapisz wynik do `artifacts/revenue_by_month.csv` oraz narysuj prosty wykres liniowy.
