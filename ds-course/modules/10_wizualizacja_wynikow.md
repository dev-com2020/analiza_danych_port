## Wizualizowanie wyników modeli

Cele modułu:
- Ocena modeli: krzywe ROC/PR, macierze pomyłek, porównanie metryk.
- Krzywe uczenia i walidacji, ważność cech.

### Macierz pomyłek i krzywe ROC/PR
```python
from sklearn.metrics import ConfusionMatrixDisplay, RocCurveDisplay, PrecisionRecallDisplay
import matplotlib.pyplot as plt

# Załóż, że mamy model clf, dane X_test, y_test (binarne)
fig, ax = plt.subplots(1, 3, figsize=(12,4))
ConfusionMatrixDisplay.from_estimator(clf, X_test, y_test, ax=ax[0])
RocCurveDisplay.from_estimator(clf, X_test, y_test, ax=ax[1])
PrecisionRecallDisplay.from_estimator(clf, X_test, y_test, ax=ax[2])
plt.tight_layout(); plt.show()
```

### Krzywe uczenia i walidacji
```python
from sklearn.model_selection import learning_curve, validation_curve
import numpy as np

train_sizes, train_scores, val_scores = learning_curve(clf, X, y, cv=5, train_sizes=np.linspace(0.1, 1.0, 5))
train_mean = train_scores.mean(axis=1)
val_mean = val_scores.mean(axis=1)

plt.plot(train_sizes, train_mean, label="train")
plt.plot(train_sizes, val_mean, label="val")
plt.legend(); plt.xlabel("rozmiar próby"); plt.ylabel("skuteczność"); plt.show()
```

### Ważność cech i SHAP (zarys)
```python
import numpy as np

# Dla modeli z feature_importances_
importances = clf.feature_importances_
idx = np.argsort(importances)[-20:]
plt.barh(range(len(idx)), importances[idx]); plt.yticks(range(len(idx)), idx); plt.show()

# Do wyjaśnialności: pakiet shap (wymaga instalacji) – zarys użycia
# import shap
# explainer = shap.Explainer(clf, X_train)
# shap_values = explainer(X_test)
# shap.plots.beeswarm(shap_values)
```

Wizualizacja to narzędzie komunikacji – dostosuj wykresy do odbiorcy i problemu.

### Ćwiczenia
- Zbuduj panel 2x2: macierz pomyłek, krzywa ROC, krzywa PR oraz ważność cech.
- Dla trzech modeli na tym samym zbiorze narysuj na jednym wykresie 3 krzywe ROC.
- Zastosuj SHAP (opcjonalnie) do zinterpretowania 10 najwyższych ważności cech.
