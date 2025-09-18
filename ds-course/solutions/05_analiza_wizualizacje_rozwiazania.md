## Rozwiązania – 05 Analiza i wizualizacje

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

tips = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/tips.csv")

# histogram + KDE
sns.histplot(tips["total_bill"], kde=True)
plt.show()

# boxplot wg dnia i płci
sns.boxplot(data=tips, x="day", y="tip", hue="sex")
plt.show()

# scatter z kolorami/markerami
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="smoker", style="sex")
plt.show()

# grupowanie i barplot z błędami
agg = tips.groupby(["day", "time"]).agg(mean_tip=("tip", "mean"), std_tip=("tip", "std")).reset_index()
sns.barplot(data=agg, x="day", y="mean_tip", hue="time")
plt.show()
```
