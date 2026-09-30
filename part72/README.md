# Part 72 - NumPy: Advanced Operations (NumPy ขั้นสูง)

## สารบัญ
1. [Linear Algebra (np.linalg)](#1-linear-algebra-nplinalg)
2. [Matrix Operations](#2-matrix-operations)
3. [Eigenvalues and Eigenvectors](#3-eigenvalues-and-eigenvectors)
4. [SVD (Singular Value Decomposition)](#4-svd-singular-value-decomposition)
5. [Fourier Transforms (np.fft)](#5-fourier-transforms-npfft)
6. [Random Number Generation](#6-random-number-generation)
7. [Polynomial Math (np.poly)](#7-polynomial-math-nppoly)
8. [Sorting and Searching](#8-sorting-and-searching)
9. [Set Operations](#9-set-operations)
10. [Structured Arrays](#10-structured-arrays)
11. [Memory Layout and Strides](#11-memory-layout-and-strides)
12. [Vectorization Techniques](#12-vectorization-techniques)
13. [NumPy with Files](#13-numpy-with-files)
14. [แบบฝึกหัด](#14-แบบฝึกหัด)

---

## 1. Linear Algebra (np.linalg)

### ทำไม Linear Algebra สำคัญ?

Linear Algebra เป็นรากฐานของ:
- **Machine Learning**: regression, neural networks, PCA
- **Computer Vision**: image transformations, 3D graphics
- **Signal Processing**: Fourier analysis
- **Physics & Engineering**: solving systems of equations

```python
# ตัวอย่างที่ 1: Matrix determinant และ rank
import numpy as np

A = np.array([[2, 1, 3],
              [1, 4, 2],
              [3, 2, 5]])

# Determinant - บอกว่า matrix เป็น singular หรือไม่
det = np.linalg.det(A)
print(f"Determinant: {det:.4f}")
# ถ้า det ≈ 0 แสดงว่า matrix เป็น singular (ไม่มี inverse)

# Rank - จำนวน linearly independent rows/columns
rank = np.linalg.matrix_rank(A)
print(f"Rank: {rank}")  # ค่าสูงสุดคือ min(rows, cols)

# Trace - ผลรวม diagonal elements
trace = np.trace(A)
print(f"Trace: {trace}")

# Norm - ขนาดของ matrix/vector
frobenius_norm = np.linalg.norm(A)           # Frobenius norm (default)
inf_norm = np.linalg.norm(A, ord=np.inf)     # Maximum absolute row sum
one_norm = np.linalg.norm(A, ord=1)          # Maximum absolute column sum
print(f"Frobenius norm: {frobenius_norm:.4f}")
print(f"Infinity norm: {inf_norm:.4f}")
```

```python
# ตัวอย่างที่ 2: Matrix inverse
import numpy as np

A = np.array([[2.0, 1.0],
              [5.0, 3.0]])

# Inverse - A^(-1) โดย A * A^(-1) = I (identity matrix)
A_inv = np.linalg.inv(A)
print("Matrix A:")
print(A)
print("\nInverse of A:")
print(A_inv)
print("\nVerify A @ A_inv (should be identity):")
print((A @ A_inv).round(10))

# Condition number - บอกว่า matrix numerically stable แค่ไหน
# ค่าสูง = ill-conditioned (ผลลัพธ์ไม่ reliable)
cond = np.linalg.cond(A)
print(f"\nCondition number: {cond:.4f}")
```

```python
# ตัวอย่างที่ 3: Solving linear systems
import numpy as np

# แก้ระบบสมการ: Ax = b
# 2x + y = 5
# 5x + 3y = 13
A = np.array([[2.0, 1.0],
              [5.0, 3.0]])
b = np.array([5.0, 13.0])

# Method 1: linalg.solve (เร็วกว่า inv)
x = np.linalg.solve(A, b)
print("Solution x:", x)  # x=2, y=1

# Verify: Ax should equal b
print("Verify Ax:", A @ x)  # [5. 13.]
print("Residual:", np.linalg.norm(A @ x - b))  # ≈ 0

# Method 2: lstsq (least-squares solution, รองรับ overdetermined systems)
x_ls, residuals, rank, sv = np.linalg.lstsq(A, b, rcond=None)
print("\nLeast-squares solution:", x_ls)
```

```python
# ตัวอย่างที่ 4: Solving overdetermined system (least squares)
import numpy as np

# 3 equations, 2 unknowns (overdetermined)
A = np.array([[1.0, 1.0],
              [1.0, 2.0],
              [1.0, 3.0],
              [1.0, 4.0],
              [1.0, 5.0]])
b = np.array([2.5, 4.0, 5.5, 7.2, 8.9])  # noisy observations

# lstsq หาค่า x ที่ minimize ||Ax - b||²
x, residuals, rank, singular_values = np.linalg.lstsq(A, b, rcond=None)
print(f"Coefficients: intercept={x[0]:.4f}, slope={x[1]:.4f}")
print(f"Residuals sum: {residuals}")
print(f"Singular values: {singular_values}")

# Predicted values
b_pred = A @ x
print(f"\nActual vs Predicted:")
for actual, pred in zip(b, b_pred):
    print(f"  {actual:.1f} vs {pred:.4f}")
```

---

## 2. Matrix Operations

```python
# ตัวอย่างที่ 5: Matrix multiplication methods
import numpy as np

A = np.array([[1, 2, 3],
              [4, 5, 6]])  # shape (2, 3)
B = np.array([[7,  8],
              [9,  10],
              [11, 12]])  # shape (3, 2)

# Matrix multiplication: (2,3) @ (3,2) = (2,2)
C1 = A @ B               # Python 3.5+
C2 = np.matmul(A, B)     # equivalent
C3 = np.dot(A, B)        # also works for 2D

print("A @ B:")
print(C1)
print("\nnp.matmul(A, B):")
print(C2)

# Element-wise multiplication (Hadamard product)
X = np.array([[1, 2], [3, 4]])
Y = np.array([[5, 6], [7, 8]])
hadamard = X * Y  # element-wise
print("\nElement-wise X * Y:")
print(hadamard)

# Outer product
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
outer = np.outer(a, b)  # shape (3, 3)
print("\nOuter product:")
print(outer)
```

```python
# ตัวอย่างที่ 6: Batch matrix operations
import numpy as np

# Batch multiplication (matmul ทำงานกับ arrays > 2D ได้)
# batch of 5 matrices, each 2x3
A_batch = np.random.default_rng(42).random((5, 2, 3))
# batch of 5 matrices, each 3x4
B_batch = np.random.default_rng(43).random((5, 3, 4))

# Multiply ทุก pair พร้อมกัน
C_batch = np.matmul(A_batch, B_batch)  # shape (5, 2, 4)
print("Batch matmul shape:", C_batch.shape)

# np.einsum - Einstein summation notation (ยืดหยุ่นมาก)
# ij,jk->ik คือ matrix multiplication
C_einsum = np.einsum('ij,jk->ik', A_batch[0], B_batch[0])
print("einsum result shape:", C_einsum.shape)
print("Same as matmul:", np.allclose(C_einsum, C_batch[0]))
```

```python
# ตัวอย่างที่ 7: Special matrix operations
import numpy as np

# QR Decomposition: A = Q @ R
# Q: orthogonal matrix (Q^T @ Q = I)
# R: upper triangular matrix
A = np.array([[12, -51, 4],
              [6,  167, -68],
              [-4, 24, -41]], dtype=float)

Q, R = np.linalg.qr(A)
print("Q (orthogonal):\n", Q.round(4))
print("R (upper triangular):\n", R.round(4))
print("Verify Q @ R ≈ A:", np.allclose(Q @ R, A))
print("Verify Q^T @ Q ≈ I:", np.allclose(Q.T @ Q, np.eye(3), atol=1e-10))

# Cholesky decomposition: A = L @ L^T (สำหรับ symmetric positive-definite matrix)
# ใช้ใน statistics, optimization
B = np.array([[4, 2], [2, 3]], dtype=float)
L = np.linalg.cholesky(B)
print("\nCholesky L:\n", L)
print("Verify L @ L^T ≈ B:", np.allclose(L @ L.T, B))
```

---

## 3. Eigenvalues and Eigenvectors

### ทำความเข้าใจ Eigenvalue/Eigenvector

**Definition**: สำหรับ matrix A, eigenvector v และ eigenvalue λ ที่สอดคล้องกัน:
`A @ v = λ * v`

**Applications**:
- **PCA**: หา principal components (eigenvectors ของ covariance matrix)
- **PageRank**: eigenvector ของ link matrix
- **Stability analysis**: eigenvalues บอกถึง stability ของ system

```python
# ตัวอย่างที่ 8: Eigenvalues และ Eigenvectors พื้นฐาน
import numpy as np

A = np.array([[4, 2],
              [1, 3]], dtype=float)

eigenvalues, eigenvectors = np.linalg.eig(A)
print("Eigenvalues:", eigenvalues)       # [5. 2.]
print("Eigenvectors (columns):\n", eigenvectors)

# Verify: A @ v = λ * v
for i, (lam, v) in enumerate(zip(eigenvalues, eigenvectors.T)):
    Av = A @ v
    lambda_v = lam * v
    print(f"\nEigenvector {i+1}: {v}")
    print(f"A @ v = {Av.round(6)}")
    print(f"λ * v = {lambda_v.round(6)}")
    print(f"Match: {np.allclose(Av, lambda_v)}")
```

```python
# ตัวอย่างที่ 9: PCA ด้วย Eigenvalues
import numpy as np

# สร้าง correlated data
rng = np.random.default_rng(42)
x = rng.standard_normal(100)
y = 2 * x + 0.5 * rng.standard_normal(100)  # y มี correlation กับ x

data = np.column_stack([x, y])  # shape (100, 2)

# Step 1: Center data
data_centered = data - data.mean(axis=0)

# Step 2: Covariance matrix
cov_matrix = np.cov(data_centered.T)  # shape (2, 2)
print("Covariance matrix:\n", cov_matrix.round(4))

# Step 3: Eigendecomposition
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# Step 4: Sort by eigenvalue (ลดลง)
idx = np.argsort(eigenvalues)[::-1]
eigenvalues = eigenvalues[idx]
eigenvectors = eigenvectors[:, idx]

print("\nEigenvalues:", eigenvalues.round(4))
print("Variance explained:")
variance_ratio = eigenvalues / eigenvalues.sum()
for i, (ev, vr) in enumerate(zip(eigenvalues, variance_ratio)):
    print(f"  PC{i+1}: {vr*100:.1f}%")

# Step 5: Project data onto principal components
projected = data_centered @ eigenvectors
print("\nProjected data shape:", projected.shape)
print("PC1 variance:", projected[:, 0].var().round(4))
print("PC2 variance:", projected[:, 1].var().round(4))
```

```python
# ตัวอย่างที่ 10: eigh - Eigenvalues สำหรับ symmetric/Hermitian matrix (เร็วกว่า eig)
import numpy as np

# Symmetric matrix (ใช้ eigh แทน eig เมื่อรู้ว่า symmetric)
A = np.array([[6, 3, 1],
              [3, 4, 2],
              [1, 2, 5]], dtype=float)

# eigh return eigenvalues เรียงลำดับจากน้อยไปมาก
eigenvalues_h, eigenvectors_h = np.linalg.eigh(A)
print("Eigenvalues (eigh):", eigenvalues_h.round(4))
print("Eigenvectors (eigh):\n", eigenvectors_h.round(4))

# Verify orthogonality: V^T @ V = I
print("\nV^T @ V (should be identity):")
print((eigenvectors_h.T @ eigenvectors_h).round(10))
```

---

## 4. SVD (Singular Value Decomposition)

### SVD คืออะไร?

SVD แตกสลาย matrix A ออกเป็น 3 matrices: `A = U @ Σ @ V^T`
- **U**: left singular vectors (m × m orthogonal)
- **Σ**: diagonal matrix of singular values (m × n)
- **V**: right singular vectors (n × n orthogonal)

**Applications**:
- Image compression
- Recommendation systems (Collaborative Filtering)
- Noise reduction
- Pseudo-inverse

```python
# ตัวอย่างที่ 11: SVD พื้นฐาน
import numpy as np

A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9],
              [10, 11, 12]], dtype=float)  # shape (4, 3)

# full_matrices=False = compact SVD (เร็วกว่า)
U, s, Vt = np.linalg.svd(A, full_matrices=False)
print("A shape:", A.shape)          # (4, 3)
print("U shape:", U.shape)          # (4, 3)
print("s (singular values):", s.round(4))  # [25.46 1.29 0.]
print("Vt shape:", Vt.shape)        # (3, 3)

# Reconstruct A
Sigma = np.diag(s)                  # convert s to diagonal matrix
A_reconstructed = U @ Sigma @ Vt
print("\nReconstruction error:", np.linalg.norm(A - A_reconstructed))

# rank ของ matrix = จำนวน non-zero singular values
rank = np.sum(s > 1e-10)
print("Effective rank:", rank)       # 2 (เนื่องจาก A มี rank 2)
```

```python
# ตัวอย่างที่ 12: Low-rank approximation (Image compression)
import numpy as np

# สร้าง grayscale image จำลอง
rng = np.random.default_rng(42)
# สร้าง low-rank image + noise
U_true = rng.standard_normal((50, 3))
V_true = rng.standard_normal((3, 50))
image = U_true @ V_true + 0.5 * rng.standard_normal((50, 50))

print("Original image shape:", image.shape)
print("Original data points:", image.size)  # 2500

# SVD
U, s, Vt = np.linalg.svd(image, full_matrices=False)
print("Singular values (first 10):", s[:10].round(3))

# Low-rank approximation ด้วย k singular values
def low_rank_approx(U, s, Vt, k):
    return U[:, :k] @ np.diag(s[:k]) @ Vt[:k, :]

# ทดสอบ k ต่างๆ
for k in [1, 3, 5, 10]:
    approx = low_rank_approx(U, s, Vt, k)
    error = np.linalg.norm(image - approx) / np.linalg.norm(image)
    compression = (k * (50 + 50 + 1)) / (50 * 50)
    print(f"k={k:2d}: error={error:.4f}, compression ratio={compression:.4f} "
          f"({compression*100:.1f}% of original)")
```

```python
# ตัวอย่างที่ 13: Moore-Penrose Pseudo-inverse
import numpy as np

# pinv ใช้ SVD ในการคำนวณ pseudo-inverse
# สำหรับ matrix ที่ไม่เป็น square หรือ singular matrix

A = np.array([[1, 2, 3],
              [4, 5, 6]])  # non-square, shape (2, 3)

A_pinv = np.linalg.pinv(A)  # shape (3, 2)
print("A shape:", A.shape)
print("A_pinv shape:", A_pinv.shape)

# Property: A @ A_pinv @ A ≈ A
print("A @ A_pinv @ A ≈ A:", np.allclose(A @ A_pinv @ A, A))

# ใช้ pseudo-inverse ใน least squares regression
# หา x ที่ minimize ||Ax - b||
b = np.array([1.0, 2.0])
x_pinv = A_pinv @ b
print("\nLeast squares solution:", x_pinv.round(6))
print("Residual:", np.linalg.norm(A @ x_pinv - b))
```

---

## 5. Fourier Transforms (np.fft)

### Fourier Transform คืออะไร?

Fourier Transform แปลง signal จาก **time domain** เป็น **frequency domain**
ทำให้เห็นว่า signal ประกอบด้วย frequencies อะไรบ้าง

**Applications**:
- Audio processing: เสียง, speech recognition
- Image processing: filtering, compression (JPEG ใช้ DCT ซึ่งเกี่ยวข้องกัน)
- Signal analysis: vibration, EEG
- Solving PDEs

```python
# ตัวอย่างที่ 14: FFT พื้นฐาน
import numpy as np

# สร้าง signal: ผสม 2 frequencies
sample_rate = 1000  # 1000 samples ต่อวินาที
duration = 1.0      # 1 วินาที
t = np.linspace(0, duration, int(sample_rate * duration), endpoint=False)

# Signal = 50 Hz + 120 Hz
freq1, freq2 = 50, 120
amplitude1, amplitude2 = 1.0, 0.5
signal = amplitude1 * np.sin(2 * np.pi * freq1 * t) + \
         amplitude2 * np.sin(2 * np.pi * freq2 * t)

print(f"Signal shape: {signal.shape}")
print(f"Sample rate: {sample_rate} Hz")
print(f"Duration: {duration} s")

# FFT
fft_result = np.fft.fft(signal)
frequencies = np.fft.fftfreq(len(t), d=1/sample_rate)

# Magnitude spectrum (เฉพาะ positive frequencies)
n = len(t)
positive_freq_idx = frequencies > 0
freq_pos = frequencies[positive_freq_idx]
magnitude = np.abs(fft_result[positive_freq_idx]) * 2 / n

# หา peaks
peak_indices = np.where(magnitude > 0.1)[0]
print("\nDetected frequencies:")
for idx in peak_indices:
    print(f"  {freq_pos[idx]:.1f} Hz, amplitude ≈ {magnitude[idx]:.3f}")
```

```python
# ตัวอย่างที่ 15: FFT สำหรับ 2D (images)
import numpy as np

# สร้าง image จำลอง (checkerboard pattern)
n = 64
image = np.zeros((n, n))
for i in range(0, n, 8):
    for j in range(0, n, 8):
        if (i//8 + j//8) % 2 == 0:
            image[i:i+8, j:j+8] = 1

# 2D FFT
fft2d = np.fft.fft2(image)
fft2d_shifted = np.fft.fftshift(fft2d)  # move zero-frequency to center
magnitude_spectrum = np.log(1 + np.abs(fft2d_shifted))  # log scale

print("Image shape:", image.shape)
print("FFT2D shape:", fft2d.shape)
print("Magnitude spectrum range:", magnitude_spectrum.min().round(2), "to", magnitude_spectrum.max().round(2))

# Inverse FFT - reconstruct image
image_reconstructed = np.fft.ifft2(fft2d).real
print("Reconstruction error:", np.max(np.abs(image - image_reconstructed)))
```

```python
# ตัวอย่างที่ 16: Low-pass filter ด้วย FFT
import numpy as np

# Signal + noise
sample_rate = 1000
t = np.linspace(0, 1, sample_rate, endpoint=False)
clean_signal = np.sin(2 * np.pi * 10 * t)  # 10 Hz clean signal
noise = 0.5 * np.random.default_rng(42).standard_normal(sample_rate)
noisy_signal = clean_signal + noise

# FFT
fft_noisy = np.fft.fft(noisy_signal)
frequencies = np.fft.fftfreq(sample_rate, 1/sample_rate)

# Low-pass filter: zero out frequencies > 20 Hz
cutoff = 20
fft_filtered = fft_noisy.copy()
fft_filtered[np.abs(frequencies) > cutoff] = 0

# Inverse FFT
filtered_signal = np.fft.ifft(fft_filtered).real

# Compare SNR
snr_noisy = np.var(clean_signal) / np.var(noisy_signal - clean_signal)
snr_filtered = np.var(clean_signal) / np.var(filtered_signal - clean_signal)
print(f"SNR (noisy): {10*np.log10(snr_noisy):.2f} dB")
print(f"SNR (filtered): {10*np.log10(snr_filtered):.2f} dB")
print("Noise reduction:", f"{snr_filtered/snr_noisy:.1f}x improvement")
```

---

## 6. Random Number Generation

### NumPy Random (New Style API)

```python
# ตัวอย่างที่ 17: New-style random generator
import numpy as np

# สร้าง Generator (แนะนำใช้แทน np.random.* ที่เป็น legacy)
rng = np.random.default_rng(seed=42)  # seed ทำให้ reproducible

# Distributions ต่างๆ
print("Uniform [0,1):", rng.random(5))
print("Uniform [a,b):", rng.uniform(2, 10, 5))
print("Normal (μ=0, σ=1):", rng.standard_normal(5).round(4))
print("Normal (μ=5, σ=2):", rng.normal(5, 2, 5).round(4))
print("Integer [1,100):", rng.integers(1, 100, 5))
print("Binomial (n=10, p=0.3):", rng.binomial(10, 0.3, 5))
print("Poisson (λ=3):", rng.poisson(3, 5))
print("Exponential (λ=1):", rng.exponential(1, 5).round(4))
print("Beta (α=2, β=5):", rng.beta(2, 5, 5).round(4))
print("Gamma (shape=2):", rng.gamma(2, 1, 5).round(4))
```

```python
# ตัวอย่างที่ 18: Multivariate distributions
import numpy as np

rng = np.random.default_rng(42)

# Multivariate Normal
mean = [0, 0]
cov = [[1, 0.8], [0.8, 1]]  # correlation = 0.8
samples = rng.multivariate_normal(mean, cov, size=1000)
print("Multivariate Normal shape:", samples.shape)
print("Sample mean:", samples.mean(axis=0).round(4))
print("Sample covariance:\n", np.cov(samples.T).round(4))

# Dirichlet distribution (สร้าง probabilities ที่รวมเป็น 1)
alpha = [1, 2, 3]  # concentration parameters
dir_samples = rng.dirichlet(alpha, size=5)
print("\nDirichlet samples:")
print(dir_samples.round(4))
print("Sum of each row:", dir_samples.sum(axis=1))  # ทุก row รวมเป็น 1
```

```python
# ตัวอย่างที่ 19: Monte Carlo simulation
import numpy as np

# ประมาณค่า π ด้วย Monte Carlo
rng = np.random.default_rng(42)
n_samples = 10_000_000  # 10 ล้าน samples

# สุ่มจุด (x,y) ใน square [0,1]²
x = rng.random(n_samples)
y = rng.random(n_samples)

# จุดที่อยู่ใน quarter circle: x² + y² ≤ 1
inside = (x**2 + y**2) <= 1.0

# π/4 = fraction of points inside circle
pi_estimate = 4 * inside.mean()
print(f"Estimated π = {pi_estimate:.6f}")
print(f"Actual π   = {np.pi:.6f}")
print(f"Error: {abs(pi_estimate - np.pi):.6f}")
```

---

## 7. Polynomial Math (np.poly)

```python
# ตัวอย่างที่ 20: Polynomial operations
import numpy as np

# Polynomial: coefficients จาก highest degree
# p = 3x³ - 2x² + 5x - 1
p = np.array([3, -2, 5, -1])

# Evaluate polynomial ที่ x = 2
value_at_2 = np.polyval(p, 2)
print(f"p(2) = {value_at_2}")  # 3(8) - 2(4) + 5(2) - 1 = 24-8+10-1 = 25

# Evaluate ที่หลายค่า
x = np.linspace(-2, 2, 100)
y = np.polyval(p, x)
print(f"y range: [{y.min():.2f}, {y.max():.2f}]")

# Polynomial arithmetic
p1 = np.array([1, 2, 3])    # x² + 2x + 3
p2 = np.array([1, -1])      # x - 1

product = np.polymul(p1, p2)
print(f"\np1 * p2 = {product}")  # coefficients of (x²+2x+3)(x-1)

quotient, remainder = np.polydiv(p1, p2)
print(f"p1 / p2: quotient={quotient}, remainder={remainder}")

# Roots of polynomial
roots = np.roots(p)
print(f"\nRoots of p: {roots}")
# Verify
for root in roots:
    val = np.polyval(p, root)
    print(f"  p({root:.4f}) = {val:.10f}")
```

```python
# ตัวอย่างที่ 21: Polynomial fitting
import numpy as np

# สร้างข้อมูล noisy polynomial
rng = np.random.default_rng(42)
x = np.linspace(-3, 3, 30)
y_true = 2*x**3 - x**2 + 3*x - 1  # true polynomial
y_noisy = y_true + 10 * rng.standard_normal(30)

# Fit polynomial of degree 3
coeffs = np.polyfit(x, y_noisy, deg=3)
print("Fitted coefficients:", coeffs.round(4))
print("True coefficients: [2, -1, 3, -1]")

# Evaluate fitted polynomial
y_fitted = np.polyval(coeffs, x)

# R² score
ss_res = np.sum((y_noisy - y_fitted)**2)
ss_tot = np.sum((y_noisy - y_noisy.mean())**2)
r_squared = 1 - ss_res/ss_tot
print(f"R² = {r_squared:.4f}")

# Poly1d object - interface ที่ใช้งานง่ายกว่า
poly = np.poly1d(coeffs)
print(f"\nPoly1d: {poly}")
print(f"poly(0) = {poly(0):.4f}")
print(f"poly(1) = {poly(1):.4f}")

# Derivative and integral
poly_deriv = poly.deriv()
poly_integ = poly.integ()
print(f"\nDerivative: {poly_deriv}")
print(f"Integral: {poly_integ}")
```

---

## 8. Sorting and Searching

```python
# ตัวอย่างที่ 22: Advanced sorting
import numpy as np

# Structured sorting
data = np.array([(3, 'Charlie', 85.5),
                 (1, 'Alice', 92.0),
                 (2, 'Bob', 78.3),
                 (4, 'David', 92.0)],
                dtype=[('id', int), ('name', 'U10'), ('score', float)])

# Sort by score (descending)
sorted_by_score = np.sort(data, order='score')[::-1]
print("Sorted by score (desc):")
for row in sorted_by_score:
    print(f"  ID={row['id']}, {row['name']}, {row['score']}")

# Sort by multiple fields (score desc, then name asc)
# NumPy sort by multiple keys: sort by secondary first, then primary
# Sort by name first
temp = np.sort(data, order='name')
# Then by score descending
final = temp[np.argsort(-temp['score'], kind='stable')]
print("\nSorted by score desc, name asc:")
for row in final:
    print(f"  {row['name']}: {row['score']}")
```

```python
# ตัวอย่างที่ 23: Searching functions
import numpy as np

# searchsorted - binary search (array must be sorted)
arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

idx_left = np.searchsorted(arr, 35)       # 'left' = smallest index where 35 could go
idx_right = np.searchsorted(arr, 35, side='right')
print(f"searchsorted(35, 'left'): {idx_left}")   # 3 (between 30 and 40)
print(f"searchsorted(35, 'right'): {idx_right}")  # 3

# หาว่าค่าอยู่ตรงไหน
values = np.array([25, 40, 75])
positions = np.searchsorted(arr, values)
print(f"\nPositions of {values}: {positions}")

# nonzero - หา indices ของ elements ที่ไม่เป็น 0
arr2 = np.array([0, 3, 0, 7, 0, 1, 0, 8])
nonzero_indices = np.nonzero(arr2)
print(f"\nNon-zero indices: {nonzero_indices[0]}")
print(f"Non-zero values: {arr2[nonzero_indices]}")

# argwhere - เหมือน nonzero แต่ return (n, ndim) array
matrix = np.array([[0, 1, 0], [2, 0, 3], [0, 4, 0]])
positions_2d = np.argwhere(matrix > 0)
print(f"\nPositions of non-zero elements in 2D:")
for pos in positions_2d:
    print(f"  [{pos[0]}, {pos[1]}] = {matrix[pos[0], pos[1]]}")
```

```python
# ตัวอย่างที่ 24: Partition - partial sort
import numpy as np

arr = np.array([7, 3, 1, 9, 4, 2, 8, 5, 6])

# partition(arr, k) - ทำให้ element ที่ k คือ element ที่ควรอยู่ตรงนั้นถ้าเรียง
# elements ก่อน k จะน้อยกว่า arr[k], elements หลัง k จะมากกว่า
partitioned = np.partition(arr, 3)  # element ที่ index 3 คือ element อันดับที่ 4
print("Partitioned:", partitioned)
print("4th smallest:", partitioned[3])  # 4 (element อันดับ 4)

# หา top-k elements อย่างมีประสิทธิภาพ (ไม่ต้อง full sort)
k = 3
# หา k smallest
k_smallest = np.partition(arr, k)[:k]
print(f"\n{k} smallest (unordered): {k_smallest}")
print(f"{k} smallest (ordered): {np.sort(k_smallest)}")

# หา k largest
k_largest = np.partition(arr, -k)[-k:]
print(f"{k} largest (unordered): {k_largest}")
print(f"{k} largest (ordered): {np.sort(k_largest)[::-1]}")
```

---

## 9. Set Operations

```python
# ตัวอย่างที่ 25: Set operations บน 1D arrays
import numpy as np

a = np.array([1, 2, 3, 4, 5, 6, 7])
b = np.array([3, 4, 5, 6, 7, 8, 9])

# Union - รวมทุก elements (ไม่ซ้ำ)
union = np.union1d(a, b)
print("Union:", union)  # [1 2 3 4 5 6 7 8 9]

# Intersection - elements ที่มีใน a และ b
intersection = np.intersect1d(a, b)
print("Intersection:", intersection)  # [3 4 5 6 7]

# Difference - elements ใน a แต่ไม่ใน b
difference = np.setdiff1d(a, b)
print("a - b:", difference)  # [1 2]

difference_ba = np.setdiff1d(b, a)
print("b - a:", difference_ba)  # [8 9]

# Symmetric difference - elements ใน a หรือ b แต่ไม่ทั้งคู่
sym_diff = np.setxor1d(a, b)
print("Symmetric difference:", sym_diff)  # [1 2 8 9]

# in1d / isin - ตรวจสอบว่า element อยู่ใน set
test = np.array([2, 5, 8, 11])
is_in_a = np.isin(test, a)
print("\ntest:", test)
print("In a:", is_in_a)  # [True True False False]
```

---

## 10. Structured Arrays

### Structured Array คืออะไร?

Structured Array คือ array ที่แต่ละ element เป็น record มีหลาย fields
เปรียบเสมือน table ที่มี typed columns

```python
# ตัวอย่างที่ 26: Creating structured arrays
import numpy as np

# กำหนด dtype ด้วย list of tuples: (field_name, dtype)
dt = np.dtype([('name', 'U20'),     # Unicode string max 20 chars
               ('age', np.int32),
               ('height', np.float64),
               ('active', np.bool_)])

# Method 1: จาก list of tuples
people = np.array([('Alice', 30, 1.65, True),
                   ('Bob', 25, 1.80, True),
                   ('Charlie', 35, 1.75, False),
                   ('Diana', 28, 1.68, True)],
                  dtype=dt)

print("Structured array:")
print(people)
print("\nDtype:", people.dtype)

# เข้าถึง fields
print("\nNames:", people['name'])
print("Ages:", people['age'])
print("Heights:", people['height'])
print("Active:", people['active'])

# Filter
active_people = people[people['active'] == True]
print("\nActive people:")
for p in active_people:
    print(f"  {p['name']}, age={p['age']}, height={p['height']}")
```

```python
# ตัวอย่างที่ 27: Nested structured arrays
import numpy as np

# Nested dtype
point_dt = np.dtype([('x', float), ('y', float)])
shape_dt = np.dtype([('center', point_dt),
                     ('radius', float),
                     ('label', 'U10')])

shapes = np.array([((0.0, 0.0), 5.0, 'circle_1'),
                   ((3.0, 4.0), 2.5, 'circle_2'),
                   ((1.0, 1.0), 1.0, 'circle_3')],
                  dtype=shape_dt)

print("Shapes:", shapes)
print("\nCenters:", shapes['center'])
print("Center x values:", shapes['center']['x'])

# Sort by radius
sorted_shapes = np.sort(shapes, order='radius')
print("\nSorted by radius:")
for s in sorted_shapes:
    print(f"  {s['label']}: center=({s['center']['x']}, {s['center']['y']}), r={s['radius']}")
```

---

## 11. Memory Layout and Strides

### ทำความเข้าใจ Memory Layout

```python
# ตัวอย่างที่ 28: C-order vs F-order
import numpy as np

# C-order (row-major, default): เก็บ elements ตาม row
arr_c = np.array([[1, 2, 3],
                  [4, 5, 6]], order='C')

# F-order (column-major, Fortran): เก็บ elements ตาม column
arr_f = np.array([[1, 2, 3],
                  [4, 5, 6]], order='F')

print("C-order strides:", arr_c.strides)   # (24, 8) bytes per step
print("F-order strides:", arr_f.strides)   # (8, 16) bytes per step
print("C-order flags:\n", arr_c.flags)

# รับ underlying memory bytes
print("\nC-order buffer:")
print(list(arr_c.data.tobytes()))  # เก็บเป็น [1,2,3,4,5,6]

print("\nF-order buffer:")
print(list(arr_f.data.tobytes()))  # เก็บเป็น [1,4,2,5,3,6]
```

```python
# ตัวอย่างที่ 29: Strides และ views
import numpy as np

arr = np.arange(16)
print("Original:", arr)
print("Strides:", arr.strides)  # (8,) - 8 bytes per element

# reshape ใช้ strides ไม่สร้าง copy
matrix = arr.reshape(4, 4)
print("\nMatrix strides:", matrix.strides)  # (32, 8) - 32 bytes per row, 8 per col

# ใช้ as_strided สำหรับ advanced operations
from numpy.lib.stride_tricks import as_strided

# สร้าง sliding window view ด้วย strides
arr = np.arange(10)
window_size = 3
shape = (len(arr) - window_size + 1, window_size)
strides = (arr.strides[0], arr.strides[0])  # step 1 element สำหรับทั้ง axes
windows = as_strided(arr, shape=shape, strides=strides)
print("\nSliding window (size=3):")
print(windows)
```

```python
# ตัวอย่างที่ 30: Memory-efficient operations
import numpy as np
import sys

# ตรวจสอบว่าเป็น contiguous หรือไม่
arr = np.arange(16).reshape(4, 4)
print("C-contiguous:", arr.flags['C_CONTIGUOUS'])    # True
print("F-contiguous:", arr.flags['F_CONTIGUOUS'])    # False

# Transpose ทำให้ไม่ contiguous
arr_t = arr.T
print("Transposed C-contiguous:", arr_t.flags['C_CONTIGUOUS'])  # False

# สร้าง contiguous copy
arr_t_contiguous = np.ascontiguousarray(arr_t)
print("After ascontiguousarray:", arr_t_contiguous.flags['C_CONTIGUOUS'])  # True

# memory usage
print("\nMemory usage:")
print(f"  int8: {np.zeros(1000, dtype=np.int8).nbytes} bytes")
print(f"  int32: {np.zeros(1000, dtype=np.int32).nbytes} bytes")
print(f"  float32: {np.zeros(1000, dtype=np.float32).nbytes} bytes")
print(f"  float64: {np.zeros(1000, dtype=np.float64).nbytes} bytes")
```

---

## 12. Vectorization Techniques

### เทคนิคการเขียน Vectorized Code

```python
# ตัวอย่างที่ 31: np.vectorize - แปลง Python function เป็น ufunc
import numpy as np

# Python function ที่ทำงานกับ scalar เท่านั้น
def categorize(score):
    if score >= 90:
        return 'A'
    elif score >= 80:
        return 'B'
    elif score >= 70:
        return 'C'
    elif score >= 60:
        return 'D'
    else:
        return 'F'

# vectorize ทำให้ apply กับ array ได้
vectorized_cat = np.vectorize(categorize)

scores = np.array([95, 82, 74, 65, 55, 88, 91, 73])
grades = vectorized_cat(scores)
print("Scores:", scores)
print("Grades:", grades)

# NOTE: np.vectorize ยังใช้ loop อยู่ ไม่ได้เร็วขึ้นจริง
# แต่สำหรับ complex logic ที่เขียน vectorized ไม่ได้ก็ใช้ได้
```

```python
# ตัวอย่างที่ 32: np.apply_along_axis
import numpy as np

data = np.array([[1, 2, 3, 4, 5],
                 [6, 7, 8, 9, 10],
                 [11, 12, 13, 14, 15]])

# Apply function ตาม axis
def normalize_row(row):
    """Normalize ให้ sum = 1"""
    return row / row.sum()

# apply ตาม axis=1 (ต่อ row)
normalized = np.apply_along_axis(normalize_row, axis=1, arr=data)
print("Normalized rows:")
print(normalized)
print("Row sums:", normalized.sum(axis=1))  # ทุก row รวมเป็น 1

# แต่ vectorized approach นี้เร็วกว่ามาก:
normalized_v = data / data.sum(axis=1, keepdims=True)
print("\nVectorized normalized:")
print(normalized_v)
```

```python
# ตัวอย่างที่ 33: Custom ufunc ด้วย np.frompyfunc
import numpy as np

# สร้าง ufunc จาก Python function
def clamp(x, low, high):
    """Clamp value between low and high"""
    return max(low, min(high, x))

# frompyfunc: nin=input args, nout=output args
clamp_ufunc = np.frompyfunc(clamp, 3, 1)

arr = np.array([-2, -1, 0, 1, 2, 3, 4, 5, 6])
result = clamp_ufunc(arr, 0, 4).astype(float)
print("Original:", arr)
print("Clamped [0,4]:", result)

# Vectorized version (เร็วกว่า)
def clamp_vectorized(arr, low, high):
    return np.clip(arr, low, high)  # numpy built-in

print("np.clip:", np.clip(arr, 0, 4))
```

```python
# ตัวอย่างที่ 34: Numba JIT compilation (ถ้าติดตั้ง)
# Numba compile Python+NumPy code เป็น machine code ที่เร็วมาก
try:
    from numba import jit
    import numpy as np
    import time

    # ฟังก์ชันที่ต้องการ JIT
    @jit(nopython=True)
    def sum_squares_numba(arr):
        result = 0.0
        for x in arr:
            result += x * x
        return result

    # Pure NumPy
    def sum_squares_numpy(arr):
        return np.sum(arr ** 2)

    arr = np.random.default_rng(42).random(10_000_000)

    # Warm up Numba (first call compiles)
    sum_squares_numba(arr[:100])

    t0 = time.time()
    r1 = sum_squares_numba(arr)
    t1 = time.time()
    print(f"Numba: {t1-t0:.4f}s, result={r1:.2f}")

    t0 = time.time()
    r2 = sum_squares_numpy(arr)
    t1 = time.time()
    print(f"NumPy: {t1-t0:.4f}s, result={r2:.2f}")

except ImportError:
    print("Numba not installed. Install with: pip install numba")
    print("Numba สามารถ speed up loops ได้ 10-100x บน NumPy-heavy code")
```

---

## 13. NumPy with Files

### บันทึกและโหลด Arrays

```python
# ตัวอย่างที่ 35: Binary format (.npy, .npz)
import numpy as np
import os

# สร้างข้อมูลตัวอย่าง
arr1 = np.random.default_rng(42).random((100, 100))
arr2 = np.arange(1000).reshape(20, 50)

# บันทึก single array (.npy)
np.save('/tmp/array1.npy', arr1)
print("Saved array1.npy")

# โหลดกลับ
loaded_arr1 = np.load('/tmp/array1.npy')
print("Loaded shape:", loaded_arr1.shape)
print("Data preserved:", np.allclose(arr1, loaded_arr1))

# บันทึกหลาย arrays (.npz)
np.savez('/tmp/arrays.npz', data=arr1, indices=arr2)
print("\nSaved arrays.npz")

# โหลด .npz (lazy loading)
npz_file = np.load('/tmp/arrays.npz')
print("Keys in npz:", list(npz_file.files))
loaded_data = npz_file['data']
loaded_indices = npz_file['indices']
print("data shape:", loaded_data.shape)
print("indices shape:", loaded_indices.shape)
npz_file.close()

# savez_compressed - บีบอัด (ไฟล์เล็กลง แต่ช้ากว่าเล็กน้อย)
np.savez_compressed('/tmp/arrays_compressed.npz', data=arr1, indices=arr2)

# เปรียบเทียบขนาดไฟล์
size_npz = os.path.getsize('/tmp/arrays.npz')
size_npz_c = os.path.getsize('/tmp/arrays_compressed.npz')
print(f"\nFile sizes:")
print(f"  .npz: {size_npz:,} bytes")
print(f"  .npz compressed: {size_npz_c:,} bytes")
print(f"  Compression ratio: {size_npz/size_npz_c:.2f}x")
```

```python
# ตัวอย่างที่ 36: Text format (savetxt, loadtxt)
import numpy as np

# สร้างข้อมูล
data = np.array([[1.23456, 2.34567, 3.45678],
                 [4.56789, 5.67890, 6.78901],
                 [7.89012, 8.90123, 9.01234]])

# savetxt - บันทึกเป็น text
np.savetxt('/tmp/data.txt', data, delimiter=',',
           fmt='%.4f',        # format: 4 decimal places
           header='col1,col2,col3',
           comments='')       # ไม่เพิ่ม # ก่อน header

# แสดงไฟล์ที่สร้าง
with open('/tmp/data.txt', 'r') as f:
    print("Saved file:")
    print(f.read())

# loadtxt - โหลดกลับ
loaded = np.loadtxt('/tmp/data.txt', delimiter=',', skiprows=1)
print("Loaded data:")
print(loaded)
print("Data preserved:", np.allclose(data, loaded, atol=0.0001))
```

```python
# ตัวอย่างที่ 37: CSV handling
import numpy as np

# สร้าง CSV data ที่มี mixed types
csv_content = """name,age,score,passed
Alice,25,85.5,True
Bob,30,72.3,True
Charlie,22,55.1,False
Diana,28,90.0,True
"""

with open('/tmp/students.csv', 'w') as f:
    f.write(csv_content)

# loadtxt กับ dtype ที่กำหนด
data = np.loadtxt('/tmp/students.csv',
                  delimiter=',',
                  skiprows=1,    # skip header
                  dtype={'names': ('name', 'age', 'score', 'passed'),
                         'formats': ('U20', 'i4', 'f8', 'U5')})

print("CSV data:")
for row in data:
    print(f"  {row['name']}: age={row['age']}, score={row['score']}, passed={row['passed']}")

# genfromtxt - รองรับ missing values
csv_with_missing = """id,value1,value2
1,10.5,20.3
2,,15.7
3,8.9,
4,12.1,18.4
"""

with open('/tmp/missing.csv', 'w') as f:
    f.write(csv_with_missing)

data_missing = np.genfromtxt('/tmp/missing.csv',
                              delimiter=',',
                              skip_header=1,
                              filling_values=np.nan)
print("\nData with missing values (as NaN):")
print(data_missing)
print("NaN positions:", np.argwhere(np.isnan(data_missing)))
```

```python
# ตัวอย่างที่ 38: Memory-mapped files สำหรับ large datasets
import numpy as np
import os

# Memory mapping ช่วยให้ทำงานกับไฟล์ขนาดใหญ่โดยไม่ต้อง load ทั้งหมดลง RAM

# สร้างไฟล์ binary ขนาดใหญ่
filename = '/tmp/large_array.bin'
shape = (10000, 1000)  # 10M elements = 80MB float64

# สร้าง memmap สำหรับ writing
mmap_write = np.memmap(filename, dtype='float64',
                       mode='w+',    # create/overwrite
                       shape=shape)

# เขียนข้อมูลทีละส่วน (ไม่ต้องโหลดทั้งหมดลง RAM)
rng = np.random.default_rng(42)
chunk_size = 1000
for i in range(0, shape[0], chunk_size):
    mmap_write[i:i+chunk_size] = rng.random((chunk_size, shape[1]))

mmap_write.flush()  # บังคับ write ลงดิสก์
del mmap_write

print(f"Created file: {os.path.getsize(filename):,} bytes")

# โหลดด้วย memory mapping (ไม่โหลด RAM จนกว่าจะเข้าถึง)
mmap_read = np.memmap(filename, dtype='float64',
                      mode='r',     # read-only
                      shape=shape)

# เข้าถึงเฉพาะส่วนที่ต้องการ
sample = mmap_read[100:110, :5]
print("\nSample from memmap:")
print(sample.round(4))

# Statistics บน slice เท่านั้น (ไม่โหลดทั้งหมด)
col_mean = mmap_read[:, 0].mean()
print(f"\nMean of first column: {col_mean:.6f}")

del mmap_read
```

---

## 14. แบบฝึกหัด

### ข้อ 1: Linear Algebra
แก้ระบบสมการ 4 ตัวแปร ด้วย `np.linalg.solve`:
```
2x + y - z + w = 3
x + 3y + 2z - w = 1
-x + y + 4z + 2w = 5
3x - y + z - 3w = 2
```

```python
# เฉลยข้อ 1
import numpy as np

A = np.array([[2, 1, -1, 1],
              [1, 3, 2, -1],
              [-1, 1, 4, 2],
              [3, -1, 1, -3]], dtype=float)
b = np.array([3, 1, 5, 2], dtype=float)

x = np.linalg.solve(A, b)
print("Solution:")
print(f"x={x[0]:.4f}, y={x[1]:.4f}, z={x[2]:.4f}, w={x[3]:.4f}")

# Verify
print("\nVerify Ax = b:")
print("Ax:", (A @ x).round(10))
print("b:", b)
print("Match:", np.allclose(A @ x, b))

print(f"\nDeterminant: {np.linalg.det(A):.4f}")
print(f"Rank: {np.linalg.matrix_rank(A)}")
```

### ข้อ 2: SVD Image Compression
Load หรือสร้าง grayscale image 100x100 แล้วทำ SVD-based compression
แสดง compression ratio และ reconstruction error สำหรับ k = 1, 5, 10, 20, 50

```python
# เฉลยข้อ 2
import numpy as np

# สร้าง grayscale image ที่มี structure (ไม่ใช่ pure noise)
rng = np.random.default_rng(42)
x = np.linspace(0, 10, 100)
y = np.linspace(0, 10, 100)
X, Y = np.meshgrid(x, y)
image = (np.sin(X) * np.cos(Y) + np.sin(2*X) * np.cos(3*Y)).astype(float)
image = (image - image.min()) / (image.max() - image.min())  # normalize 0-1

print(f"Image shape: {image.shape}")
print(f"Original data points: {image.size}")

# SVD decomposition
U, s, Vt = np.linalg.svd(image)

print(f"\nU shape: {U.shape}")
print(f"s shape: {s.shape}")
print(f"Vt shape: {Vt.shape}")

# Total variance explained
total_energy = np.sum(s**2)

print(f"\n{'k':>4} | {'Compression':>12} | {'Error':>10} | {'Energy%':>8}")
print("-" * 42)
for k in [1, 5, 10, 20, 50]:
    # Reconstruct
    approx = U[:, :k] @ np.diag(s[:k]) @ Vt[:k, :]

    # Metrics
    n_params = k * (100 + 100 + 1)  # U cols + Vt rows + s values
    compression = n_params / image.size
    error = np.sqrt(np.mean((image - approx)**2))  # RMSE
    energy = np.sum(s[:k]**2) / total_energy

    print(f"{k:>4} | {compression:>12.4f} | {error:>10.6f} | {energy*100:>7.2f}%")
```

### ข้อ 3: Fourier Analysis
วิเคราะห์ signal ที่ประกอบด้วย frequencies ไม่รู้จัก:
- สร้าง signal: 3 Hz (amplitude 1), 7 Hz (amplitude 0.5), 15 Hz (amplitude 0.8) + noise
- ใช้ FFT เพื่อหา dominant frequencies
- Filter เฉพาะ frequencies < 10 Hz และ reconstruct signal

```python
# เฉลยข้อ 3
import numpy as np

sample_rate = 500  # Hz
duration = 2.0
t = np.linspace(0, duration, int(sample_rate * duration), endpoint=False)

# สร้าง signal
rng = np.random.default_rng(42)
signal = (1.0 * np.sin(2 * np.pi * 3 * t) +
          0.5 * np.sin(2 * np.pi * 7 * t) +
          0.8 * np.sin(2 * np.pi * 15 * t) +
          0.2 * rng.standard_normal(len(t)))  # noise

print(f"Signal: {len(t)} samples, {duration}s duration")

# FFT
fft_result = np.fft.fft(signal)
freqs = np.fft.fftfreq(len(t), 1/sample_rate)
magnitude = np.abs(fft_result) * 2 / len(t)

# หา positive frequencies เท่านั้น
pos_mask = freqs > 0
pos_freqs = freqs[pos_mask]
pos_mag = magnitude[pos_mask]

# หา peaks (magnitude > threshold)
threshold = 0.15
peaks = pos_freqs[pos_mag > threshold]
peak_mags = pos_mag[pos_mag > threshold]

print("\nDetected frequencies:")
for f, m in sorted(zip(peaks, peak_mags)):
    print(f"  {f:.1f} Hz, amplitude ≈ {m:.3f}")

# Low-pass filter (cutoff = 10 Hz)
fft_filtered = fft_result.copy()
fft_filtered[np.abs(freqs) > 10] = 0

signal_filtered = np.fft.ifft(fft_filtered).real
print(f"\nLow-pass filtered (cutoff=10Hz)")
print(f"Original RMS: {np.sqrt(np.mean(signal**2)):.4f}")
print(f"Filtered RMS: {np.sqrt(np.mean(signal_filtered**2)):.4f}")
```

### ข้อ 4: Monte Carlo Integration
ใช้ Monte Carlo method ประมาณค่า integral:
∫∫ (x² + y²) dxdy โดย x ∈ [0,1], y ∈ [0,1]
(ค่าจริง = 2/3)

```python
# เฉลยข้อ 4
import numpy as np

rng = np.random.default_rng(42)

true_value = 2/3
print(f"True integral value: {true_value:.6f}")

# Monte Carlo integration
for n in [1000, 10000, 100000, 1000000]:
    x = rng.uniform(0, 1, n)
    y = rng.uniform(0, 1, n)
    f_values = x**2 + y**2          # function to integrate
    area = 1.0 * 1.0                 # area of integration region
    estimate = area * f_values.mean()

    error = abs(estimate - true_value)
    print(f"n={n:>8,}: estimate={estimate:.6f}, error={error:.6f}")
```

### ข้อ 5: Polynomial Regression
fit polynomial degree 2, 4, 8 กับข้อมูล noisy sin wave
คำนวณ training error และ overfitting

```python
# เฉลยข้อ 5
import numpy as np

rng = np.random.default_rng(42)
x_train = np.linspace(0, 2*np.pi, 20)
y_train = np.sin(x_train) + 0.2 * rng.standard_normal(20)

x_test = np.linspace(0, 2*np.pi, 100)
y_test = np.sin(x_test)

print("Polynomial Regression Results:")
print(f"{'Degree':>8} | {'Train MSE':>12} | {'Test MSE':>12}")
print("-" * 38)

for degree in [1, 2, 4, 8, 12]:
    # Fit
    coeffs = np.polyfit(x_train, y_train, degree)

    # Evaluate
    y_train_pred = np.polyval(coeffs, x_train)
    y_test_pred = np.polyval(coeffs, x_test)

    train_mse = np.mean((y_train - y_train_pred)**2)
    test_mse = np.mean((y_test - y_test_pred)**2)

    print(f"{degree:>8} | {train_mse:>12.6f} | {test_mse:>12.6f}")
```

### ข้อ 6: Structured Array Database
สร้าง "mini database" ของ products ด้วย structured array และทำ queries ต่างๆ

```python
# เฉลยข้อ 6
import numpy as np

# กำหนด schema
product_dt = np.dtype([
    ('id', np.int32),
    ('name', 'U30'),
    ('category', 'U20'),
    ('price', np.float64),
    ('stock', np.int32),
    ('rating', np.float32)
])

# สร้างข้อมูล
products = np.array([
    (1,  'Laptop Pro 15',    'Electronics',  45999.0, 50,  4.5),
    (2,  'Wireless Mouse',   'Electronics',   1299.0, 200, 4.2),
    (3,  'Python Book',      'Books',          599.0, 100, 4.8),
    (4,  'Coffee Maker',     'Appliances',   3499.0,  30, 3.9),
    (5,  'USB Hub',          'Electronics',   799.0, 150, 4.1),
    (6,  'Standing Desk',    'Furniture',   12999.0,  20, 4.7),
    (7,  'Data Science Book','Books',          799.0,  80, 4.9),
    (8,  'Blender',          'Appliances',   2299.0,  45, 4.0),
    (9,  'Monitor 27"',      'Electronics', 15999.0,  35, 4.6),
    (10, 'Ergonomic Chair',  'Furniture',    8999.0,  25, 4.3)
], dtype=product_dt)

# Queries
print("=== Product Database ===\n")

# Query 1: ทุก Electronics ที่ราคา < 5000
elec_cheap = products[(products['category'] == 'Electronics') &
                       (products['price'] < 5000)]
print("Electronics < 5000 THB:")
for p in elec_cheap:
    print(f"  {p['name']}: {p['price']:.2f} THB")

# Query 2: Top 3 products by rating
top3 = products[np.argsort(products['rating'])[::-1][:3]]
print("\nTop 3 by rating:")
for p in top3:
    print(f"  {p['name']}: ★{p['rating']:.1f}")

# Query 3: Value ของ inventory แต่ละ category
for cat in np.unique(products['category']):
    cat_mask = products['category'] == cat
    cat_products = products[cat_mask]
    total_value = (cat_products['price'] * cat_products['stock']).sum()
    print(f"\n{cat}: total inventory value = {total_value:,.0f} THB")
```

### ข้อ 7: Memory Optimization
สร้าง large dataset และ optimize memory usage ด้วย dtype selection

```python
# เฉลยข้อ 7
import numpy as np

rng = np.random.default_rng(42)
n = 1_000_000

# สร้าง dataset
ids = rng.integers(1, 100001, n)      # 1-100000
ages = rng.integers(18, 80, n)        # 18-79
scores = rng.uniform(0, 100, n)       # 0-100
labels = rng.integers(0, 5, n)        # 0-4

# Version 1: Default dtypes
data_default = {
    'ids': ids.astype(np.int64),
    'ages': ages.astype(np.int64),
    'scores': scores.astype(np.float64),
    'labels': labels.astype(np.int64)
}

# Version 2: Optimized dtypes
data_optimized = {
    'ids': ids.astype(np.int32),      # int32 เพียงพอสำหรับ 100000
    'ages': ages.astype(np.int8),     # int8 พอสำหรับ 18-79
    'scores': scores.astype(np.float32),  # float32 ประหยัดกว่า float64
    'labels': labels.astype(np.int8) # int8 พอสำหรับ 0-4
}

def total_memory(data_dict):
    return sum(arr.nbytes for arr in data_dict.values())

mem_default = total_memory(data_default)
mem_optimized = total_memory(data_optimized)

print(f"Default dtypes: {mem_default/1024**2:.1f} MB")
print(f"Optimized dtypes: {mem_optimized/1024**2:.1f} MB")
print(f"Memory saved: {(1 - mem_optimized/mem_default)*100:.1f}%")

# ตรวจสอบว่าค่ายังถูกต้อง
print(f"\nAge range: [{data_optimized['ages'].min()}, {data_optimized['ages'].max()}]")
print(f"Score range: [{data_optimized['scores'].min():.2f}, {data_optimized['scores'].max():.2f}]")
```

### ข้อ 8: Set Operations สำหรับ Recommendation System
ใช้ set operations ทำ simple recommendation:
กำหนด users กับ items ที่ชอบ หา:
- Items ที่ user A และ user B ชอบเหมือนกัน (Jaccard similarity)
- Items ที่ user A ชอบแต่ user B ยังไม่ได้ดู (items to recommend)
- Items ที่ popular (ทุกคนชอบ)

```python
# เฉลยข้อ 8
import numpy as np

# สร้างข้อมูล user preferences (item IDs)
user_likes = {
    'Alice':   np.array([1, 2, 3, 5, 8, 10, 13, 15]),
    'Bob':     np.array([2, 3, 4, 6, 8, 11, 12, 15]),
    'Charlie': np.array([1, 3, 5, 7, 9, 10, 13, 14]),
    'Diana':   np.array([2, 4, 5, 8, 10, 12, 15, 16])
}

# Jaccard similarity: |A∩B| / |A∪B|
def jaccard_similarity(set1, set2):
    intersection = np.intersect1d(set1, set2)
    union = np.union1d(set1, set2)
    return len(intersection) / len(union)

# คำนวณ pairwise Jaccard similarity
users = list(user_likes.keys())
print("Jaccard Similarity Matrix:")
print(f"{'':>10}", end='')
for u in users:
    print(f"{u:>10}", end='')
print()

for u1 in users:
    print(f"{u1:>10}", end='')
    for u2 in users:
        sim = jaccard_similarity(user_likes[u1], user_likes[u2])
        print(f"{sim:>10.3f}", end='')
    print()

# Recommendations: Items ที่ Alice ชอบแต่ Bob ยังไม่ได้ดู
recommendations = np.setdiff1d(user_likes['Alice'], user_likes['Bob'])
print(f"\nRecommend to Bob (Alice liked but Bob hasn't): {recommendations}")

# Popular items: ชอบอย่างน้อย 3 คน
all_items = np.union1d(*user_likes.values())
popularity = {}
for item in all_items:
    count = sum(item in likes for likes in user_likes.values())
    popularity[item] = count

popular_threshold = 3
popular_items = np.array([item for item, cnt in popularity.items() if cnt >= popular_threshold])
print(f"Popular items (liked by ≥{popular_threshold} users): {np.sort(popular_items)}")
```

### ข้อ 9: Vectorized Statistics
สร้างฟังก์ชัน compute_stats ที่ vectorized (ไม่มี loop):
รับ 2D array (n_samples, n_features) และ return dict ของ stats

```python
# เฉลยข้อ 9
import numpy as np

def compute_stats(data):
    """
    Compute comprehensive statistics for 2D array.
    Fully vectorized, no Python loops.

    Args:
        data: ndarray shape (n_samples, n_features)
    Returns:
        dict with per-feature statistics
    """
    stats = {}

    # Basic stats (ตาม axis=0 = per feature/column)
    stats['mean'] = np.mean(data, axis=0)
    stats['median'] = np.median(data, axis=0)
    stats['std'] = np.std(data, axis=0, ddof=1)    # sample std
    stats['var'] = np.var(data, axis=0, ddof=1)     # sample var
    stats['min'] = np.min(data, axis=0)
    stats['max'] = np.max(data, axis=0)
    stats['range'] = stats['max'] - stats['min']

    # Percentiles
    stats['q1'] = np.percentile(data, 25, axis=0)
    stats['q3'] = np.percentile(data, 75, axis=0)
    stats['iqr'] = stats['q3'] - stats['q1']       # interquartile range

    # Skewness: E[(x-μ)³] / σ³
    centered = data - stats['mean']
    stats['skewness'] = np.mean(centered**3, axis=0) / (stats['std']**3 + 1e-10)

    # Kurtosis: E[(x-μ)⁴] / σ⁴ - 3
    stats['kurtosis'] = np.mean(centered**4, axis=0) / (stats['std']**4 + 1e-10) - 3

    # Outlier detection (>3 std from mean)
    z_scores = np.abs(centered) / (stats['std'] + 1e-10)
    stats['n_outliers'] = np.sum(z_scores > 3, axis=0)

    # Count valid (non-NaN) values
    stats['n_valid'] = np.sum(~np.isnan(data), axis=0)
    stats['n_missing'] = np.sum(np.isnan(data), axis=0)

    return stats

# ทดสอบ
rng = np.random.default_rng(42)
data = rng.normal(0, 1, (1000, 5))
data[rng.random((1000, 5)) < 0.02] = np.nan  # 2% missing values

stats = compute_stats(data)

print("Feature Statistics (5 features, 1000 samples):")
print(f"\n{'Metric':>12} | " + " | ".join(f"F{i+1:>10}" for i in range(5)))
print("-" * 70)
for key in ['mean', 'std', 'min', 'max', 'q1', 'q3', 'skewness', 'n_outliers', 'n_missing']:
    values = stats[key]
    print(f"{key:>12} | " + " | ".join(f"{v:>10.3f}" for v in values))
```

### ข้อ 10: End-to-End NumPy Pipeline
ทำ complete data analysis pipeline:
1. สร้าง sales data (12 months × 5 products × 3 regions)
2. คำนวณ total, mean, growth rate ในแต่ละมิติ
3. ทำ SVD เพื่อหา dominant patterns
4. บันทึกและโหลดกลับ

```python
# เฉลยข้อ 10
import numpy as np

rng = np.random.default_rng(42)

# 1. สร้าง data: shape (12, 5, 3) = months × products × regions
months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']
products = ['P1', 'P2', 'P3', 'P4', 'P5']
regions = ['North', 'Central', 'South']

# สร้าง realistic sales data with trends
base_sales = rng.integers(100, 500, (5, 3)).astype(float)
trend = np.linspace(1.0, 1.3, 12)  # 30% growth over year

sales = np.zeros((12, 5, 3))
for m in range(12):
    seasonal = 1 + 0.2 * np.sin(2 * np.pi * m / 12)  # seasonal pattern
    noise = rng.uniform(0.9, 1.1, (5, 3))
    sales[m] = base_sales * trend[m] * seasonal * noise

print("Sales data shape:", sales.shape)
print("(months=12, products=5, regions=3)\n")

# 2. Analysis
# Total sales per month (across all products and regions)
monthly_total = sales.sum(axis=(1, 2))  # shape (12,)
print("Monthly totals:")
for month, total in zip(months, monthly_total):
    print(f"  {month}: {total:.0f}")

# Monthly growth rate
growth_rate = (monthly_total[1:] / monthly_total[:-1] - 1) * 100
print(f"\nAvg monthly growth: {growth_rate.mean():.2f}%")

# Best product per region
product_region_total = sales.sum(axis=0)  # shape (5, 3)
best_product_per_region = np.argmax(product_region_total, axis=0)
print("\nBest product per region:")
for region, best_idx in zip(regions, best_product_per_region):
    print(f"  {region}: {products[best_idx]}")

# 3. SVD ใน 2D reshape
sales_2d = sales.reshape(12, 15)  # (months, products*regions)
U, s, Vt = np.linalg.svd(sales_2d, full_matrices=False)

# Energy explained by each component
energy = s**2 / np.sum(s**2)
cumulative_energy = np.cumsum(energy)
print("\nSVD Energy explained:")
for i, (e, ce) in enumerate(zip(energy[:5], cumulative_energy[:5])):
    print(f"  PC{i+1}: {e*100:.1f}% (cumulative: {ce*100:.1f}%)")

# 4. บันทึกและโหลดกลับ
np.savez('/tmp/sales_analysis.npz',
         sales=sales,
         monthly_total=monthly_total,
         growth_rate=growth_rate,
         singular_values=s)

print("\nSaved to sales_analysis.npz")

# โหลดกลับ
loaded = np.load('/tmp/sales_analysis.npz')
print("Loaded keys:", list(loaded.files))
print("Sales data preserved:", np.allclose(sales, loaded['sales']))
loaded.close()
```

---

## สรุป Part 72

ใน Part นี้เราได้เรียนรู้ advanced NumPy operations:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| **Linear Algebra** | solve, inv, det, rank, norm, lstsq |
| **Matrix Ops** | matmul, batch ops, QR, Cholesky, einsum |
| **Eigenvalues** | eig, eigh, PCA implementation |
| **SVD** | decomposition, compression, pseudo-inverse |
| **FFT** | 1D/2D FFT, frequency analysis, filtering |
| **Random** | new-style Generator, distributions, Monte Carlo |
| **Polynomial** | polyfit, polyval, roots, derivative |
| **Sorting** | partition, argsort, searchsorted |
| **Set Ops** | union, intersect, difference, isin |
| **Structured** | dtype definitions, field access, sorting |
| **Strides** | memory layout, view tricks, memmap |
| **Vectorization** | vectorize, frompyfunc, numba |
| **File I/O** | npy, npz, savetxt, genfromtxt, memmap |

### Performance Tips

1. **ใช้ `np.linalg.solve` แทน `inv`**: เร็วกว่าและ numerically stable กว่า
2. **SVD สำหรับ low-rank problems**: ดีกว่า matrix inverse สำหรับ ill-conditioned matrices
3. **FFT complexity**: O(n log n) แทน O(n²) ของ direct convolution
4. **Memory mapping**: ใช้กับ datasets ที่ใหญ่กว่า RAM
5. **Partition ดีกว่า Sort**: เมื่อต้องการ top-k ไม่ต้อง full sort

---

**ก่อนหน้า**: [Part 71 - NumPy Fundamentals](../part71/README.md)
**ต่อไป**: [Part 73 - Pandas Data Manipulation](../part73/README.md)
