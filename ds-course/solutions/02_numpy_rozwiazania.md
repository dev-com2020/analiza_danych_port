## Rozwiązania – 02 NumPy

```python
import numpy as np

# co piąta wartość 0..99
a = np.arange(100)
print(a[::5])

# macierz 10x10 i odwrócone kolumny
M = np.arange(100).reshape(10, 10)
print(M[:, ::-1])

# maska podzielne przez 3 i 5
mask = (a % 3 == 0) & (a % 5 == 0)
print(a[mask])

# broadcasting: dodanie wektora do kolumn
v = np.arange(10)
print(M + v)

# układ równań
A = np.array([[2., 1.], [1., -1.]])
b = np.array([5., 1.])
x = np.linalg.solve(A, b)
print(x)
```
