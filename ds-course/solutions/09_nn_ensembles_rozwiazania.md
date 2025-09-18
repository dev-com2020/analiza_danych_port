## Rozwiązania – 09 NN i ensembling

```python
from sklearn.neural_network import MLPClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, ConfusionMatrixDisplay
from sklearn.datasets import load_digits
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, StackingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
import matplotlib.pyplot as plt

X, y = load_digits(return_X_y=True)
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

mlp = MLPClassifier(hidden_layer_sizes=(100,100), activation="relu", max_iter=400, random_state=42)
mlp.fit(Xtr, ytr)
print(classification_report(yte, mlp.predict(Xte)))

rf = RandomForestClassifier(n_estimators=300, random_state=42)
gb = GradientBoostingClassifier(random_state=42)
stack = StackingClassifier(estimators=[("svc", SVC(probability=True, random_state=42)), ("rf", rf)], final_estimator=LogisticRegression(max_iter=1000))
for model in [rf, gb, stack]:
    model.fit(Xtr, ytr)
    print(model.__class__.__name__, "\n", classification_report(yte, model.predict(Xte)))

ConfusionMatrixDisplay.from_estimator(stack, Xte, yte)
plt.show()
```
