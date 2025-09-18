## Sztuczne Sieci Neuronowe i łączenie klasyfikatorów

Cele modułu:
- Zarys sieci neuronowych w sklearn/keras.
- Ensembling: bagging, boosting, stacking.
- Wizualizowanie wyników i monitorowanie.

### Sieci neuronowe (zarys, sklearn MLPClassifier)
```python
from sklearn.neural_network import MLPClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
from sklearn.datasets import load_digits

X, y = load_digits(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

mlp = MLPClassifier(hidden_layer_sizes=(100,100), activation="relu", max_iter=300, random_state=42)
mlp.fit(X_train, y_train)
print(classification_report(y_test, mlp.predict(X_test)))
```

### Ensembling
```python
from sklearn.ensemble import BaggingClassifier, RandomForestClassifier, GradientBoostingClassifier, StackingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC

# Bagging / Random Forest
rf = RandomForestClassifier(n_estimators=300, random_state=42)

# Boosting
gb = GradientBoostingClassifier(random_state=42)

# Stacking
base_estimators = [("svc", SVC(probability=True, random_state=42)), ("rf", rf)]
stack = StackingClassifier(estimators=base_estimators, final_estimator=LogisticRegression(max_iter=1000))
```

### Wizualizowanie wyników modeli
```python
import matplotlib.pyplot as plt
from sklearn.metrics import ConfusionMatrixDisplay, RocCurveDisplay

fig, ax = plt.subplots(1, 2, figsize=(10,4))
ConfusionMatrixDisplay.from_estimator(mlp, X_test, y_test, ax=ax[0])
RocCurveDisplay.from_estimator(stack, X_test, (y_test==1).astype(int), ax=ax[1])  # przykład binarny
plt.tight_layout()
plt.show()
```

W praktyce do sieci głębokich użyj `tensorflow/keras` lub `pytorch` i GPU.
