## Analiza danych i wizualizacje

Cele modułu:
- Podstawowe eksploracje i agregacje w `pandas`.
- Wprowadzenie do `matplotlib` i generowania wykresów z poziomu `pandas`.
- `seaborn` i wygodne wizualizacje statystyczne.

### Szybkie eksploracje w pandas
```python
import pandas as pd

df = pd.read_csv("data/tips.csv")
print(df.info())
print(df.describe(include="all"))
print(df.isna().mean())

# Grupowanie i agregacje
by_day = df.groupby("day")["total_bill"].agg(["count", "mean", "median"]).reset_index()
print(by_day)
```

### Wprowadzenie do matplotlib
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 2*np.pi, 200)
y = np.sin(x)
plt.figure(figsize=(6,3))
plt.plot(x, y, label="sin(x)")
plt.title("Wykres funkcji sin")
plt.xlabel("x")
plt.ylabel("y")
plt.legend()
plt.tight_layout()
plt.show()
```

### Wykresy z poziomu pandas
```python
ax = by_day.plot(kind="bar", x="day", y="mean", title="Średni rachunek wg dnia", legend=False)
ax.set_xlabel("Dzień")
ax.set_ylabel("Średni rachunek")
plt.tight_layout()
plt.show()
```

### Seaborn i wykresy statystyczne
```python
import seaborn as sns

tips = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv")

# wykres rozrzutu z estymacją regresji liniowej
sns.lmplot(data=tips, x="total_bill", y="tip", hue="time")
plt.show()

# rozkłady
plt.figure()
sns.histplot(tips["total_bill"], kde=True)
plt.show()

# porównanie grup
plt.figure()
sns.boxplot(data=tips, x="day", y="total_bill", hue="sex")
plt.show()
```

W praktyce pamiętaj o kontekście: dobieraj typ wykresu do pytania analitycznego.
