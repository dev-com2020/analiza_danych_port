## Wstęp do uczenia maszynowego – proces

Cele modułu:
- Zrozumieć kroki: eksploracja, wybór modelu, przygotowanie danych, podział na zbiory, trening, walidacja, przeciwdziałanie przeuczeniu, redukcja wymiarów.

### Eksploracja danych i przygotowanie cech
- Oczyszczanie braków, kodowanie kategorii (`OneHotEncoder`), skalowanie (`StandardScaler`, `MinMaxScaler`).
- Tworzenie cech pochodnych i walidacja założeń.

### Podział danych i walidacja
```python
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

pipe = make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000, random_state=42))
cv_scores = cross_val_score(pipe, X_train, y_train, cv=5, scoring="roc_auc")
print(cv_scores.mean(), cv_scores.std())
```

### Unikanie przeuczenia
- Regularizacja (L1/L2), wczesne zatrzymanie, ograniczenie złożoności modelu.
- Więcej danych, augmentacja, sensowne cechy.

### Redukcja wymiarowości
```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2, random_state=42)
X_2d = pca.fit_transform(X_train)
print(pca.explained_variance_ratio_)
```

### Wybór modelu
- Użyj siatki hiperparametrów i walidacji krzyżowej.
```python
from sklearn.model_selection import GridSearchCV

param_grid = {"logisticregression__C": [0.1, 1.0, 10.0]}
grid = GridSearchCV(pipe, param_grid=param_grid, cv=5, scoring="roc_auc")
grid.fit(X_train, y_train)
print(grid.best_params_, grid.best_score_)
```

### Ćwiczenia
- Dla wybranego zbioru danych przygotuj pipeline: skalowanie + model, z walidacją krzyżową.
- Porównaj 2 modele (np. logistyczna vs. SVM) na tych samych cechach – wybierz lepszy po ROC-AUC.
- Narysuj krzywą uczenia i zinterpretuj, czy problem to high bias czy high variance.
