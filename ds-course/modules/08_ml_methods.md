## Metody uczenia maszynowego – przegląd

Cele modułu:
- Zrozumieć główne klasy problemów: regresja, klasyfikacja, grupowanie, redukcja wymiarów.
- Poznać przykładowe algorytmy i metryki.

### Regresja
```python
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import make_pipeline

X, y = fetch_california_housing(return_X_y=True, as_frame=False)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Regresja liniowa
lin = LinearRegression().fit(X_train, y_train)
y_pred = lin.predict(X_test)
print("MSE:", mean_squared_error(y_test, y_pred), "R2:", r2_score(y_test, y_pred))

# Regresja wielomianowa (feature engineering)
poly_model = make_pipeline(PolynomialFeatures(degree=2, include_bias=False), LinearRegression())
poly_model.fit(X_train, y_train)
print("R2 poly:", r2_score(y_test, poly_model.predict(X_test)))
```

### Regresja logistyczna (klasyfikacja binarna)
```python
from sklearn.datasets import load_breast_cancer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, roc_auc_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

X, y = load_breast_cancer(return_X_y=True)
clf = make_pipeline(StandardScaler(), LogisticRegression(max_iter=2000, random_state=42))
clf.fit(X, y)
proba = clf.predict_proba(X)[:, 1]
print("ACC:", accuracy_score(y, clf.predict(X)), "ROC-AUC:", roc_auc_score(y, proba))
```

### Klasyfikacja – inne modele
```python
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(n_estimators=200, random_state=42)
rf.fit(X, y)
```

### Grupowanie danych (uczenie nienadzorowane)
```python
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans

X, _ = make_blobs(n_samples=500, centers=3, random_state=42, cluster_std=1.2)
kmeans = KMeans(n_clusters=3, n_init=10, random_state=42)
labels = kmeans.fit_predict(X)
```

### Redukcja wymiarów
```python
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE

pca = PCA(n_components=2, random_state=42)
X2 = pca.fit_transform(X)

tsne = TSNE(n_components=2, random_state=42, init="pca", perplexity=30)
X2_tsne = tsne.fit_transform(X)
```

Wybór metryki: regresja – MSE/RMSE/MAE/R2; klasyfikacja – Accuracy/Precision/Recall/F1/ROC-AUC; grupowanie – silhouette score; redukcja – wyjaśniona wariancja.

### Ćwiczenia
- Na `fetch_california_housing` porównaj regresję liniową i wielomianową (stopnie 2–3).
- Dla `breast_cancer` porównaj LogisticRegression i RandomForest – raport metryk.
- Przeprowadź KMeans dla danych 2D i narysuj klastry; policz silhouette score dla k=2..6.
- Zastosuj PCA do 2D i TSNE do wizualizacji klastrów – porównaj.
