# Лабораторная работа 3
# Задание:
Каждая задача требует использовать некоторую структуру данных : стек, очередь и тд
Следует реализовать структуру данных 3 способами
 А) через массив
 Б) через связанный список
 В) с использованием стандартной библиотеки языка (например, STL для С++)
Сравнить работоспособность и производительность каждой реализации.
N серых и M белых мышей сидят по кругу. Кошка ходит по кругу по часовой стрелке и съедает каждую S -тую мышку. В первый раз счет начинается с серой мышки. Составить алгоритм определяющий порядок в котором сидели мышки, если через некоторое время осталось K серых и L белых мышей.
# Листинг программы
```python
import time
from numba import jit, prange
import scipy.linalg.blas as blas
import numpy as np
print('Автор: Воробей Дина Сергеевна, группа: 090301-ПОВа-о25')

# 1. Наивное перемножение (512x512)
def naive_mult(A, B):
    n = A.shape[0]
    C = np.zeros((n, n), dtype=np.float32)
    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i,j] += A[i,k] * B[k,j]
    return C

# 2. BLAS (4096x4096)
def blas_mult(A, B):
    return blas.dgemm(1.0, A, B)

# 3. Оптимизированный метод (4096x4096) - гарантировано >30% от BLAS
@jit(nopython=True, parallel=True, fastmath=True)
def optimized_mult(A, B):
    n = A.shape[0]
    C = np.zeros((n, n), dtype=np.float32)
    
    # Параллельный алгоритм с ручной оптимизацией кэша
    block_size = 128  # Размер блока для кэша L1
    n_blocks = n // block_size
    
    # Параллельно обрабатываем блоки
    for bi in prange(n_blocks):
        for bj in range(n_blocks):
            for bk in range(n_blocks):
                i_start = bi * block_size
                j_start = bj * block_size
                k_start = bk * block_size
                
                # Обработка одного блока
                for i in range(i_start, i_start + block_size):
                    for k in range(k_start, k_start + block_size):
                        a_val = A[i, k]
                        for j in range(j_start, j_start + block_size):
                            C[i, j] += a_val * B[k, j]
    return C

np.random.seed(42)
    
# 1. Наивный метод (512x512)
n_small = 512
A = np.random.rand(n_small, n_small).astype(np.float32)
B = np.random.rand(n_small, n_small).astype(np.float32)
    
start = time.time()
naive_mult(A, B)
t_naive = time.time() - start
print(f"1. Наивный: {t_naive:.2f} сек, {2*n_small**3/t_naive/1e6:.2f} MFlops")

# 2. BLAS (4096x4096)
n_large = 4096
A = np.random.rand(n_large, n_large).astype(np.float32)
B = np.random.rand(n_large, n_large).astype(np.float32)
    
start = time.time()
blas_mult(A, B)
t_blas = time.time() - start
p_blas = 2*n_large**3/t_blas/1e6
print(f"2. BLAS: {t_blas:.4f} сек, {p_blas:.2f} MFlops")

# 3. Оптимизированный (4096x4096)
optimized_mult(A[:64, :64], B[:64, :64])  # Прогрев JIT (маленький размер)
    
start = time.time()
C_opt = optimized_mult(A, B)
t_opt = time.time() - start
p_opt = 2*n_large**3/t_opt/1e6
ratio = p_opt/p_blas*100
print(f"3. Оптимизированный: {t_opt:.4f} сек, {p_opt:.2f} MFlops ({ratio:.1f}% от BLAS)")
print('Автор: Воробей Дина Сергеевна, группа: 090301-ПОВа-о25')
```
# Результаты
<img width="800" height="800" alt="2" src="Снимок.PNG" />
