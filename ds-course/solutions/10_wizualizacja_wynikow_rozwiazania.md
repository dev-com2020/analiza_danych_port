## Rozwiązania – 10 Wizualizacja wyników

```python
import matplotlib.pyplot as plt
from sklearn.metrics import ConfusionMatrixDisplay, RocCurveDisplay, PrecisionRecallDisplay
import numpy as np

# Załóżmy, że mamy clf, X_test, y_test oraz importances
fig, axes = plt.subplots(2, 2, figsize=(10,8))
ConfusionMatrixDisplay.from_estimator(clf, X_test, y_test, ax=axes[0,0])
RocCurveDisplay.from_estimator(clf, X_test, y_test, ax=axes[0,1])
PrecisionRecallDisplay.from_estimator(clf, X_test, y_test, ax=axes[1,0])

imp = importances
idx = np.argsort(imp)[-15:]
axes[1,1].barh(range(len(idx)), imp[idx])
axes[1,1].set_yticks(range(len(idx)))
axes[1,1].set_yticklabels(idx)
plt.tight_layout(); plt.show()

# Porównanie ROC trzech modeli
fig, ax = plt.subplots(figsize=(5,4))
for name, model in [("LR", lr), ("RF", rf), ("SVC", svc)]:
    RocCurveDisplay.from_estimator(model, X_test, y_test, ax=ax, name=name)
plt.show()
```
