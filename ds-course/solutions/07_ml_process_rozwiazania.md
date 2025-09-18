## Rozwiązania – 07 Proces ML

```python
from sklearn.model_selection import train_test_split, cross_val_score, learning_curve
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.metrics import roc_auc_score
from sklearn.datasets import load_breast_cancer
import numpy as np

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

pipe_lr = make_pipeline(StandardScaler(), LogisticRegression(max_iter=2000, random_state=42))
pipe_svc = make_pipeline(StandardScaler(), SVC(probability=True, random_state=42))

cv_lr = cross_val_score(pipe_lr, X_train, y_train, cv=5, scoring="roc_auc").mean()
cv_svc = cross_val_score(pipe_svc, X_train, y_train, cv=5, scoring="roc_auc").mean()
print(cv_lr, cv_svc)

pipe_best = pipe_lr if cv_lr >= cv_svc else pipe_svc
pipe_best.fit(X_train, y_train)
proba = pipe_best.predict_proba(X_test)[:,1]
print("ROC-AUC test:", roc_auc_score(y_test, proba))

sizes, train_scores, val_scores = learning_curve(pipe_best, X, y, cv=5, train_sizes=np.linspace(0.1,1.0,5))
print(train_scores.mean(axis=1), val_scores.mean(axis=1))
```
