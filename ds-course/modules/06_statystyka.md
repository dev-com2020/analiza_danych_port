## Podstawy analizy statystycznej i wnioskowanie

Cele modułu:
- Miary położenia i rozproszenia, korelacje.
- Rozkłady, centralne twierdzenie graniczne (intuicyjnie).
- Wnioskowanie: estymacja, przedziały ufności, testy A/B (zarys).

### Miary opisowe i korelacje
```python
import pandas as pd

df = pd.read_csv("data/tips.csv")
print(df["total_bill"].mean(), df["total_bill"].median(), df["total_bill"].std())
print(df[["total_bill", "tip", "size"]].corr(method="pearson"))
```

### Przedziały ufności (przykład z bootstrapem)
```python
import numpy as np

x = df["tip"].dropna().to_numpy()
rng = np.random.default_rng(42)
boot_means = [rng.choice(x, size=len(x), replace=True).mean() for _ in range(5000)]
ci_low, ci_high = np.percentile(boot_means, [2.5, 97.5])
print(ci_low, ci_high)
```

### Testy hipotez – przykład (różnica średnich, z-score w uproszczeniu)
```python
from scipy import stats

group_m = df[df["sex"] == "Male"]["tip"].dropna()
group_f = df[df["sex"] == "Female"]["tip"].dropna()

t_stat, p_value = stats.ttest_ind(group_m, group_f, equal_var=False)
print(t_stat, p_value)
```

Uwaga: dobór testu zależy od rozkładu danych, liczebności i założeń (normalność, wariancje).
