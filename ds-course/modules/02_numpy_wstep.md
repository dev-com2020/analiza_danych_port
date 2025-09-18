## Wstęp do NumPy

Cele modułu:
- Tworzenie wektorów i macierzy (`ndarray`).
- Przekształcenia i operacje: typy, kształty, ucinanie/slicing, maski.
- Wektoryzacja i broadcasting.
- Elementy arytmetyki i algebry liniowej, rozwiązywanie równań liniowych.

### Tworzenie tablic
```python
import numpy as np

v = np.array([1, 2, 3], dtype=np.float64)  # wektor
M = np.array([[1, 2], [3, 4]], dtype=np.int64)  # macierz 2x2
z = np.zeros((3, 4))
o = np.ones(5)
r = np.arange(0, 10, 2)  # 0..8 co 2
l = np.linspace(0, 1, 5)  # 5 punktów z [0,1]

# Losowe
np.random.seed(42)
rn = np.random.randn(3, 3)  # N(0,1)
ru = np.random.rand(2, 4)   # U(0,1)
```

### Kształty i typy
```python
print(v.shape, v.ndim, v.dtype)
print(M.T)         # transpozycja
print(M.reshape(1, 4))
print(M.astype(np.float32))
```

### Wybieranie/slicing/maski
```python
a = np.arange(10)
print(a[2:7])        # 2..6
print(a[::2])        # co 2
print(a[::-1])       # odwrócenie

B = np.arange(12).reshape(3, 4)
print(B[1, 2])       # element
print(B[0, :])       # pierwszy wiersz
print(B[:, 1:3])     # kolumny 1-2

mask = B % 2 == 0
print(B[mask])       # tylko parzyste
```

### Wektoryzacja i broadcasting
```python
x = np.array([1, 2, 3])
y = np.array([10, 20, 30])
print(x + y)            # wektoryzacja
print(x * 10)           # skalar rozszerzany (broadcasting)

X = np.ones((3, 1))
Y = np.arange(3).reshape(1, 3)
print(X + Y)            # kształty (3,1) + (1,3) -> (3,3)
```

### Arytmetyka i algebra liniowa
```python
A = np.array([[3., 2.], [1., 4.]])
b = np.array([6., 5.])

# Mnożenie macierzowe
print(A @ A)           # lub np.matmul(A, A)

# Normy, ślady, wyznaczniki
print(np.linalg.norm(A))
print(np.trace(A))
print(np.linalg.det(A))

# Wartości własne i wektory własne
w, V = np.linalg.eig(A)
print(w)
print(V)

# Rozwiązywanie równań liniowych: A x = b
x = np.linalg.solve(A, b)
print(x)
```

Uwagi wydajnościowe: preferuj operacje na całych tablicach zamiast pętli w czystym Pythonie.

### Ćwiczenia
- Utwórz wektor liczb od 0 do 99 i wyciągnij co piątą wartość.
- Zbuduj macierz 10x10 z liczb 0..99 i odwróć kolejność kolumn.
- Wykorzystaj maskę do wybrania elementów podzielnych przez 3 i 5.
- Zademonstruj broadcasting: dodaj wektor długości 10 do każdej kolumny macierzy 10x10.
- Rozwiąż układ równań: `2x + y = 5`, `x - y = 1` przy użyciu `np.linalg.solve`.
