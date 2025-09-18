## Rozwiązania – 06 Statystyka

```python
import pandas as pd
import numpy as np
from scipy import stats

tips = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv")

# bootstrap CI 95%
x = tips["tip"].dropna().to_numpy()
rng = np.random.default_rng(42)
boot = [rng.choice(x, size=len(x), replace=True).mean() for _ in range(5000)]
ci = np.percentile(boot, [2.5, 97.5])
print(ci)

# t-test
g_yes = tips[tips["smoker"] == "Yes"]["tip"].dropna()
g_no = tips[tips["smoker"] == "No"]["tip"].dropna()
print(stats.ttest_ind(g_yes, g_no, equal_var=False))

# korelacje
print(tips[["total_bill","tip"]].corr(method="pearson"))
print(tips[["total_bill","tip"]].corr(method="spearman"))
```
