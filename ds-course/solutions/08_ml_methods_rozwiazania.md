## Rozwiązania – 08 Metody ML

```python
from sklearn.datasets import fetch_california_housing, load_breast_cancer, make_blobs
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.metrics import mean_squared_error, r2_score, silhouette_score, classification_report
from sklearn.preprocessing import PolynomialFeatures, StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.ensemble import RandomForestClassifier
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE

# regresja
X, y = fetch_california_housing(return_X_y=True)
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, random_state=42)
lin = LinearRegression().fit(Xtr, ytr)
poly2 = make_pipeline(PolynomialFeatures(2, include_bias=False), LinearRegression()).fit(Xtr, ytr)
print(r2_score(yte, lin.predict(Xte)), r2_score(yte, poly2.predict(Xte)))

# klasyfikacja
Xc, yc = load_breast_cancer(return_X_y=True)
pipe_log = make_pipeline(StandardScaler(), LogisticRegression(max_iter=2000))
rf = RandomForestClassifier(n_estimators=300, random_state=42)
pipe_log.fit(Xc, yc); rf.fit(Xc, yc)
print(classification_report(yc, pipe_log.predict(Xc)))
print(classification_report(yc, rf.predict(Xc)))

# grupowanie i silhouette
Xb, _ = make_blobs(n_samples=800, centers=4, random_state=42, cluster_std=1.1)
for k in range(2,7):
    km = KMeans(n_clusters=k, n_init=10, random_state=42).fit(Xb)
    print(k, silhouette_score(Xb, km.labels_))

# redukcja wymiarów
pca = PCA(n_components=2, random_state=42).fit_transform(Xb)
tsne = TSNE(n_components=2, random_state=42, init="pca", perplexity=30).fit_transform(Xb)
```
