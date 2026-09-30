# Part 71 - NumPy: Fundamentals (พื้นฐาน NumPy)

## สารบัญ
1. [NumPy คืออะไร และทำไมต้องใช้](#1-numpy-คืออะไร-และทำไมต้องใช้)
2. [ndarray Object](#2-ndarray-object)
3. [Creating Arrays](#3-creating-arrays)
4. [Array Shapes and Dimensions](#4-array-shapes-and-dimensions)
5. [Indexing และ Slicing](#5-indexing-และ-slicing)
6. [Boolean Indexing](#6-boolean-indexing)
7. [Fancy Indexing](#7-fancy-indexing)
8. [Array Operations (Element-wise)](#8-array-operations-element-wise)
9. [Broadcasting](#9-broadcasting)
10. [Array Math Operations](#10-array-math-operations)
11. [Statistical Functions](#11-statistical-functions)
12. [Array Manipulation](#12-array-manipulation)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. NumPy คืออะไร และทำไมต้องใช้

### ความเป็นมาของ NumPy

**NumPy** (Numerical Python) คือ library หลักสำหรับการคำนวณทางตัวเลขใน Python
ถูกพัฒนาโดย Travis Oliphant ในปี 2005 โดย build บน Numeric และ Numarray libraries ก่อนหน้า

NumPy เป็นรากฐานของ ecosystem การวิเคราะห์ข้อมูล Python ทั้งหมด:
- **Pandas** ใช้ NumPy arrays เป็น backend
- **Scikit-learn** ใช้ NumPy สำหรับ ML computations
- **TensorFlow / PyTorch** ได้รับแรงบันดาลใจจาก NumPy API
- **SciPy** extend NumPy สำหรับ scientific computing

### ทำไม NumPy ถึงเร็วกว่า Python Lists?

Python lists มีปัญหาหลายอย่างสำหรับ numerical computation:

1. **Dynamic typing overhead**: Python ต้องตรวจสอบ type ของแต่ละ element ทุกครั้ง
2. **Memory fragmentation**: List เก็บ pointers ไปยัง objects ที่กระจัดกระจายใน memory
3. **No vectorized operations**: ต้องใช้ loop ซึ่งช้า

NumPy แก้ปัญหาเหล่านี้ด้วย:
1. **Contiguous memory**: elements เก็บติดกันใน memory (cache-friendly)
2. **Fixed type**: ทุก element มี type เดียวกัน ไม่ต้องตรวจสอบทุกครั้ง
3. **Vectorized C operations**: ใช้ optimized C/Fortran code ทำงานกับ arrays ทั้งหมดพร้อมกัน
4. **SIMD instructions**: ใช้ CPU hardware parallelism

```python
# ตัวอย่างที่ 1: เปรียบเทียบความเร็ว Python List vs NumPy
import numpy as np
import time

# สร้างข้อมูล 10 ล้านตัวเลข
n = 10_000_000

# Python List
python_list = list(range(n))

# NumPy Array
numpy_array = np.arange(n)

# วัดเวลา: บวกทุกตัวเลขด้วย 1
start = time.time()
python_result = [x + 1 for x in python_list]
python_time = time.time() - start
print(f"Python List time: {python_time:.3f} seconds")

start = time.time()
numpy_result = numpy_array + 1
numpy_time = time.time() - start
print(f"NumPy time: {numpy_time:.4f} seconds")
print(f"NumPy เร็วกว่า {python_time/numpy_time:.0f}x")

# Output ตัวอย่าง:
# Python List time: 0.850 seconds
# NumPy time: 0.012 seconds
# NumPy เร็วกว่า 70x
```

```python
# ตัวอย่างที่ 2: เปรียบเทียบ memory usage
import sys
import numpy as np

n = 1_000_000

# Python List of integers
python_list = list(range(n))
python_memory = sys.getsizeof(python_list) + sum(sys.getsizeof(x) for x in python_list[:10]) * n // 10
print(f"Python List memory (approx): {python_memory / 1024**2:.1f} MB")

# NumPy Array
numpy_array = np.arange(n, dtype=np.int64)
numpy_memory = numpy_array.nbytes
print(f"NumPy Array memory: {numpy_memory / 1024**2:.1f} MB")

# Python List ใช้ memory มากกว่า NumPy ประมาณ 5-10 เท่า
# เพราะแต่ละ element ใน Python List คือ full Python object (28 bytes)
# ขณะที่ NumPy int64 ใช้แค่ 8 bytes ต่อ element
```

### การติดตั้ง NumPy

```bash
# ติดตั้งด้วย pip
pip install numpy

# ติดตั้งด้วย conda (แนะนำสำหรับ data science)
conda install numpy

# ตรวจสอบ version
python -c "import numpy as np; print(np.__version__)"
```

```python
# ตัวอย่างที่ 3: import NumPy (convention)
import numpy as np  # ใช้ alias 'np' เป็น convention มาตรฐาน

# ตรวจสอบข้อมูล NumPy
print(np.__version__)           # version
print(np.show_config())         # แสดง compilation config
```

---

## 2. ndarray Object

### ndarray คืออะไร?

**ndarray** (N-dimensional array) คือ core data structure ของ NumPy
เป็น grid ของ values ที่มี type เดียวกัน (homogeneous) และถูก index โดยใช้ tuple ของ integers

คุณสมบัติสำคัญของ ndarray:
- **ndim**: จำนวน dimensions (axes)
- **shape**: tuple แสดงขนาดของแต่ละ dimension
- **size**: จำนวน elements ทั้งหมด
- **dtype**: ชนิดข้อมูลของ elements
- **itemsize**: ขนาดของแต่ละ element ใน bytes
- **data**: buffer ที่เก็บ actual array elements

```python
# ตัวอย่างที่ 4: ทำความรู้จัก ndarray attributes
import numpy as np

# สร้าง 2D array
arr = np.array([[1, 2, 3],
                [4, 5, 6],
                [7, 8, 9]])

print("Array:")
print(arr)
print()
print(f"ndim (จำนวน dimensions): {arr.ndim}")      # 2
print(f"shape (รูปร่าง): {arr.shape}")             # (3, 3)
print(f"size (จำนวน elements): {arr.size}")        # 9
print(f"dtype (ชนิดข้อมูล): {arr.dtype}")          # int64
print(f"itemsize (bytes ต่อ element): {arr.itemsize}")  # 8
print(f"nbytes (bytes ทั้งหมด): {arr.nbytes}")     # 72
```

### Data Types (dtype) ใน NumPy

NumPy รองรับ data types หลายประเภท:

| dtype | คำอธิบาย | ขนาด (bytes) | ช่วงค่า |
|-------|-----------|--------------|---------|
| `int8` | Integer 8-bit | 1 | -128 ถึง 127 |
| `int16` | Integer 16-bit | 2 | -32,768 ถึง 32,767 |
| `int32` | Integer 32-bit | 4 | -2.1B ถึง 2.1B |
| `int64` | Integer 64-bit | 8 | -9.2E18 ถึง 9.2E18 |
| `uint8` | Unsigned Integer 8-bit | 1 | 0 ถึง 255 |
| `float16` | Float 16-bit (half) | 2 | ~±65,504 |
| `float32` | Float 32-bit (single) | 4 | ~±3.4E38 |
| `float64` | Float 64-bit (double) | 8 | ~±1.8E308 |
| `complex64` | Complex number | 8 | - |
| `complex128` | Complex number | 16 | - |
| `bool` | Boolean | 1 | True/False |
| `str_` | Unicode string | variable | - |
| `object_` | Python object | variable | - |

```python
# ตัวอย่างที่ 5: การกำหนด dtype
import numpy as np

# สร้าง array ด้วย dtype ต่างๆ
arr_int32 = np.array([1, 2, 3], dtype=np.int32)
arr_float64 = np.array([1.0, 2.5, 3.7], dtype=np.float64)
arr_bool = np.array([True, False, True], dtype=np.bool_)
arr_complex = np.array([1+2j, 3+4j], dtype=np.complex128)
arr_str = np.array(['hello', 'world'], dtype=np.str_)

print(f"int32: {arr_int32}, dtype={arr_int32.dtype}")
print(f"float64: {arr_float64}, dtype={arr_float64.dtype}")
print(f"bool: {arr_bool}, dtype={arr_bool.dtype}")
print(f"complex128: {arr_complex}, dtype={arr_complex.dtype}")
print(f"str: {arr_str}, dtype={arr_str.dtype}")

# การแปลง dtype (type casting)
arr_float = np.array([1.7, 2.9, 3.1])
arr_int = arr_float.astype(np.int32)  # truncate (ไม่ปัดเศษ)
print(f"\nFloat: {arr_float}")
print(f"Cast to int32: {arr_int}")  # [1, 2, 3]
```

```python
# ตัวอย่างที่ 6: Overflow และ Underflow
import numpy as np

# int8 มีค่าสูงสุด 127
arr = np.array([127], dtype=np.int8)
print(f"int8 max: {arr[0]}")         # 127
arr_overflow = arr + 1
print(f"int8 max + 1 (overflow): {arr_overflow[0]}")  # -128 (overflow!)

# ใช้ dtype ที่เหมาะสมเพื่อหลีกเลี่ยง overflow
arr_safe = np.array([127], dtype=np.int16)
print(f"int16 max+1: {arr_safe + 1}")  # [128] (ปลอดภัย)
```

---

## 3. Creating Arrays

### วิธีสร้าง Arrays แบบต่างๆ

```python
# ตัวอย่างที่ 7: np.array() - สร้างจาก Python sequences
import numpy as np

# จาก list
arr1d = np.array([1, 2, 3, 4, 5])
print("1D:", arr1d)

# จาก nested list (2D)
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print("2D:", arr2d)

# จาก nested nested list (3D)
arr3d = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print("3D shape:", arr3d.shape)  # (2, 2, 2)
print("3D:\n", arr3d)

# จาก tuple
arr_tuple = np.array((10, 20, 30))
print("From tuple:", arr_tuple)

# กำหนด dtype ขณะสร้าง
arr_float = np.array([1, 2, 3], dtype=float)
print("With dtype=float:", arr_float)  # [1. 2. 3.]
```

```python
# ตัวอย่างที่ 8: np.zeros() และ np.ones()
import numpy as np

# zeros - สร้าง array ที่เต็มไปด้วย 0
zeros_1d = np.zeros(5)                    # 1D, 5 elements
zeros_2d = np.zeros((3, 4))               # 2D, 3x4
zeros_3d = np.zeros((2, 3, 4))            # 3D, 2x3x4
zeros_int = np.zeros((3, 3), dtype=int)   # zeros ของ integer

print("zeros_1d:", zeros_1d)
print("zeros_2d:\n", zeros_2d)
print("zeros_3d shape:", zeros_3d.shape)

# ones - สร้าง array ที่เต็มไปด้วย 1
ones_2d = np.ones((3, 3))
print("\nones_2d:\n", ones_2d)

# full - สร้าง array ที่เต็มไปด้วยค่าที่กำหนด
fives = np.full((3, 3), 5)
pi_arr = np.full((2, 4), np.pi)
print("\nfull of 5s:\n", fives)
print("full of pi:\n", pi_arr)
```

```python
# ตัวอย่างที่ 9: np.arange() - สร้าง array แบบ sequence
import numpy as np

# arange(stop)
arr1 = np.arange(10)
print("arange(10):", arr1)  # [0 1 2 3 4 5 6 7 8 9]

# arange(start, stop)
arr2 = np.arange(2, 10)
print("arange(2, 10):", arr2)  # [2 3 4 5 6 7 8 9]

# arange(start, stop, step)
arr3 = np.arange(0, 20, 2)
print("arange(0, 20, 2):", arr3)  # [0 2 4 6 8 10 12 14 16 18]

# arange กับ float step
arr4 = np.arange(0, 1, 0.1)
print("arange(0, 1, 0.1):", arr4)  # ระวัง floating point precision!

# arange กับค่าลบ
arr5 = np.arange(10, 0, -1)
print("arange(10, 0, -1):", arr5)  # [10 9 8 7 6 5 4 3 2 1]
```

```python
# ตัวอย่างที่ 10: np.linspace() - สร้าง evenly spaced values
import numpy as np

# linspace(start, stop, num) - กำหนดจำนวน points (รวม endpoint)
arr1 = np.linspace(0, 1, 11)
print("linspace(0, 1, 11):", arr1)
# [0.  0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1. ]

# ไม่รวม endpoint
arr2 = np.linspace(0, 1, 10, endpoint=False)
print("linspace without endpoint:", arr2)

# linspace ดีกว่า arange สำหรับ float เพราะควบคุม precision ได้ดีกว่า
arr3 = np.linspace(0, 2*np.pi, 100)  # 100 points สำหรับ sine wave
print(f"\n100 points 0 to 2π: shape={arr3.shape}, first={arr3[0]:.4f}, last={arr3[-1]:.4f}")
```

```python
# ตัวอย่างที่ 11: np.logspace() - logarithmically spaced
import numpy as np

# logspace(start, stop, num) - 10^start ถึง 10^stop
arr1 = np.logspace(0, 3, 4)  # 10^0=1, 10^1=10, 10^2=100, 10^3=1000
print("logspace(0, 3, 4):", arr1)  # [   1.   10.  100. 1000.]

# เปลี่ยน base
arr2 = np.logspace(0, 8, 9, base=2)  # 2^0=1 ถึง 2^8=256
print("logspace base=2:", arr2)  # [  1.   2.   4.  8.  16.  32.  64. 128. 256.]
```

```python
# ตัวอย่างที่ 12: Random arrays
import numpy as np

# ตั้ง seed เพื่อให้ผลลัพธ์ reproducible
rng = np.random.default_rng(42)  # แนะนำ: ใช้ new-style Generator

# uniform distribution [0, 1)
rand_uniform = rng.random((3, 4))
print("Random uniform:\n", rand_uniform)

# normal distribution (mean=0, std=1)
rand_normal = rng.standard_normal((3, 3))
print("\nRandom normal:\n", rand_normal)

# integers
rand_int = rng.integers(1, 100, size=(2, 5))  # 1 ถึง 99
print("\nRandom integers:\n", rand_int)

# choice - สุ่มจาก array
choices = rng.choice([10, 20, 30, 40, 50], size=5, replace=False)
print("\nRandom choice:", choices)

# shuffle - สลับลำดับ
arr = np.arange(10)
rng.shuffle(arr)
print("Shuffled:", arr)
```

```python
# ตัวอย่างที่ 13: Special arrays
import numpy as np

# identity matrix (เมทริกซ์เอกลักษณ์)
identity = np.eye(4)
print("Identity matrix:\n", identity)

# diagonal matrix
diag = np.diag([1, 2, 3, 4])
print("\nDiagonal matrix:\n", diag)

# เอา diagonal จาก matrix
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
diag_values = np.diag(matrix)
print("\nDiagonal values:", diag_values)  # [1 5 9]

# empty - สร้าง array โดยไม่ initialize (เร็วกว่า zeros แต่ค่าไม่แน่นอน)
empty = np.empty((3, 3))
print("\nEmpty array (uninitialized):\n", empty)

# zeros_like, ones_like - สร้าง array shape เดียวกับที่กำหนด
original = np.array([[1.5, 2.3], [3.7, 4.1]])
zeros_like = np.zeros_like(original)
ones_like = np.ones_like(original)
print("\nzeros_like:\n", zeros_like)
print("ones_like:\n", ones_like)
```

---

## 4. Array Shapes and Dimensions

### ทำความเข้าใจ Shape และ Dimensions

```python
# ตัวอย่างที่ 14: เข้าใจ shape และ dimensions
import numpy as np

# 0D array (scalar)
scalar = np.array(42)
print(f"0D: value={scalar}, shape={scalar.shape}, ndim={scalar.ndim}")
# shape=(), ndim=0

# 1D array (vector)
vector = np.array([1, 2, 3, 4, 5])
print(f"1D: shape={vector.shape}, ndim={vector.ndim}")
# shape=(5,), ndim=1

# 2D array (matrix)
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(f"2D: shape={matrix.shape}, ndim={matrix.ndim}")
# shape=(2, 3), ndim=2  -> 2 rows, 3 columns

# 3D array (tensor)
tensor = np.zeros((4, 3, 2))
print(f"3D: shape={tensor.shape}, ndim={tensor.ndim}")
# shape=(4, 3, 2), ndim=3 -> 4 planes, 3 rows, 2 columns

# 4D array (ใช้ใน CNN: batch, channels, height, width)
batch = np.zeros((32, 3, 224, 224))
print(f"4D (image batch): shape={batch.shape}, ndim={batch.ndim}")
```

```python
# ตัวอย่างที่ 15: reshape - เปลี่ยน shape โดยไม่เปลี่ยนข้อมูล
import numpy as np

arr = np.arange(24)
print("Original:", arr.shape)  # (24,)

# reshape เป็นต่างๆ (ต้องได้ขนาดเท่าเดิม)
arr_2d = arr.reshape(4, 6)    # 4x6 = 24 ✓
arr_3d = arr.reshape(2, 3, 4) # 2x3x4 = 24 ✓
arr_back = arr.reshape(24)    # กลับเป็น 1D

print("Reshaped to (4,6):\n", arr_2d)
print("Reshaped to (2,3,4) shape:", arr_3d.shape)

# ใช้ -1 ให้ NumPy คำนวณขนาดอัตโนมัติ
arr_auto = arr.reshape(6, -1)  # -1 = 24/6 = 4 -> shape (6, 4)
print("reshape(6, -1):", arr_auto.shape)  # (6, 4)

arr_auto2 = arr.reshape(-1, 8)  # -1 = 24/8 = 3 -> shape (3, 8)
print("reshape(-1, 8):", arr_auto2.shape)  # (3, 8)
```

```python
# ตัวอย่างที่ 16: เพิ่ม/ลด dimensions
import numpy as np

arr = np.array([1, 2, 3, 4])
print("Original shape:", arr.shape)  # (4,)

# เพิ่ม dimension ด้วย np.newaxis
row_vector = arr[np.newaxis, :]  # เพิ่ม axis ที่ 0 -> shape (1, 4)
col_vector = arr[:, np.newaxis]  # เพิ่ม axis ที่ 1 -> shape (4, 1)
print("Row vector shape:", row_vector.shape)  # (1, 4)
print("Col vector shape:", col_vector.shape)  # (4, 1)

# expand_dims - เพิ่ม dimension ที่ตำแหน่งที่กำหนด
arr_exp = np.expand_dims(arr, axis=0)   # (1, 4)
arr_exp2 = np.expand_dims(arr, axis=1)  # (4, 1)
print("expand_dims axis=0:", arr_exp.shape)
print("expand_dims axis=1:", arr_exp2.shape)

# squeeze - ลบ dimensions ที่มีขนาด 1
arr_squeezed = np.squeeze(arr_exp)  # (4,)
print("After squeeze:", arr_squeezed.shape)
```

---

## 5. Indexing และ Slicing

### การเข้าถึง Elements ใน Arrays

```python
# ตัวอย่างที่ 17: 1D Indexing และ Slicing
import numpy as np

arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90, 100])
#               0    1    2    3    4    5    6    7    8    9
#             -10   -9   -8   -7   -6   -5   -4   -3   -2   -1

# Positive indexing
print("arr[0]:", arr[0])   # 10 (first element)
print("arr[4]:", arr[4])   # 50
print("arr[9]:", arr[9])   # 100 (last element)

# Negative indexing (นับจากท้าย)
print("arr[-1]:", arr[-1])  # 100 (last)
print("arr[-3]:", arr[-3])  # 80

# Slicing: arr[start:stop:step]
print("\narr[2:7]:", arr[2:7])      # [30 40 50 60 70] (index 2 ถึง 6)
print("arr[:5]:", arr[:5])          # [10 20 30 40 50] (จาก 0 ถึง 4)
print("arr[5:]:", arr[5:])          # [60 70 80 90 100] (จาก 5 ถึงท้าย)
print("arr[::2]:", arr[::2])        # [10 30 50 70 90] (ทุก 2 ตัว)
print("arr[1::2]:", arr[1::2])      # [20 40 60 80 100] (เริ่มจาก index 1, ทุก 2 ตัว)
print("arr[::-1]:", arr[::-1])      # [100 90 80 70 60 50 40 30 20 10] (reverse)
print("arr[2:8:2]:", arr[2:8:2])   # [30 50 70] (index 2,4,6)
```

```python
# ตัวอย่างที่ 18: 2D Indexing และ Slicing
import numpy as np

matrix = np.array([[1,  2,  3,  4],
                   [5,  6,  7,  8],
                   [9, 10, 11, 12],
                   [13, 14, 15, 16]])

# เข้าถึง element เดียว: matrix[row, col]
print("matrix[0, 0]:", matrix[0, 0])   # 1 (top-left)
print("matrix[2, 3]:", matrix[2, 3])   # 12
print("matrix[-1, -1]:", matrix[-1, -1])  # 16 (bottom-right)

# เข้าถึงทั้ง row
print("\nRow 0:", matrix[0])          # [1 2 3 4]
print("Row 2:", matrix[2])            # [9 10 11 12]
print("Last row:", matrix[-1])        # [13 14 15 16]

# เข้าถึงทั้ง column
print("\nColumn 0:", matrix[:, 0])    # [1 5 9 13]
print("Column 2:", matrix[:, 2])      # [3 7 11 15]
print("Last column:", matrix[:, -1])  # [4 8 12 16]

# 2D Slicing: matrix[row_slice, col_slice]
print("\nTop-left 2x2:", matrix[:2, :2])
# [[1 2]
#  [5 6]]

print("\nBottom-right 2x2:", matrix[2:, 2:])
# [[11 12]
#  [15 16]]

print("\nRows 1-2, Cols 1-3:", matrix[1:3, 1:4])
# [[ 6  7  8]
#  [10 11 12]]

# step ใน 2D
print("\nEvery other row and col:", matrix[::2, ::2])
# [[ 1  3]
#  [ 9 11]]
```

```python
# ตัวอย่างที่ 19: 3D Indexing
import numpy as np

# สร้าง 3D array shape (2, 3, 4)
# คิดเป็น 2 "pages", แต่ละ page มี 3 rows x 4 cols
tensor = np.arange(24).reshape(2, 3, 4)
print("3D array:\n", tensor)
print("Shape:", tensor.shape)

# เข้าถึง page (axis 0)
print("\nPage 0:\n", tensor[0])      # [[0 1 2 3], [4 5 6 7], [8 9 10 11]]
print("Page 1:\n", tensor[1])       # [[12 13 14 15], ...]

# เข้าถึง row ใน page
print("\nPage 0, Row 1:", tensor[0, 1])    # [4 5 6 7]

# เข้าถึง element เดี่ยว
print("tensor[1, 2, 3]:", tensor[1, 2, 3])  # 23

# 3D Slicing
print("\nAll pages, Row 0, All cols:", tensor[:, 0, :])
# [[ 0  1  2  3]
#  [12 13 14 15]]

print("\nPage 0, All rows, Col 2:", tensor[0, :, 2])  # [2 6 10]
```

```python
# ตัวอย่างที่ 20: View vs Copy - ความสำคัญมากในการแก้ไขข้อมูล
import numpy as np

original = np.array([1, 2, 3, 4, 5])

# Slice สร้าง VIEW (ไม่ copy ข้อมูล)
view = original[1:4]
print("Before modification:")
print("original:", original)
print("view:", view)

view[0] = 99  # แก้ไข view
print("\nAfter view[0] = 99:")
print("original:", original)  # [1 99 3 4 5] - original เปลี่ยนด้วย!
print("view:", view)          # [99 3 4]

# ตรวจสอบว่าเป็น view หรือ copy
print("\nview.base is original:", view.base is original)  # True = เป็น view

# Copy - ใช้ .copy() เพื่อสร้างสำเนาอิสระ
original2 = np.array([1, 2, 3, 4, 5])
copy_arr = original2[1:4].copy()  # สร้าง copy

copy_arr[0] = 99
print("\nAfter copy[0] = 99:")
print("original2:", original2)  # [1 2 3 4 5] - ไม่เปลี่ยน!
print("copy_arr:", copy_arr)    # [99 3 4]
```

---

## 6. Boolean Indexing

### การเลือกข้อมูลด้วยเงื่อนไข

Boolean indexing ช่วยให้เลือก elements ที่ตรงตาม condition โดยไม่ต้องใช้ loop

```python
# ตัวอย่างที่ 21: Boolean Indexing พื้นฐาน
import numpy as np

arr = np.array([10, 25, 3, 47, 15, 8, 62, 33, 18, 55])

# สร้าง boolean mask
mask = arr > 20
print("Array:", arr)
print("mask (> 20):", mask)  # [False True False True ...]

# ใช้ mask เพื่อเลือก elements
selected = arr[mask]
print("Elements > 20:", selected)  # [25 47 62 33 55]

# เขียนสั้นลงได้
print("Elements > 20 (direct):", arr[arr > 20])

# conditions ต่างๆ
print("\nElements >= 15 and <= 50:", arr[(arr >= 15) & (arr <= 50)])
print("Elements < 10 or > 50:", arr[(arr < 10) | (arr > 50)])
print("Elements != 15:", arr[arr != 15])
```

```python
# ตัวอย่างที่ 22: Boolean Indexing กับ 2D arrays
import numpy as np

matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

# เลือก elements ที่มีค่า > 5
print("Elements > 5:", matrix[matrix > 5])  # [6 7 8 9] (flatten)

# สร้าง boolean mask 2D
mask_2d = matrix > 5
print("Boolean mask:\n", mask_2d)

# แก้ไขค่าด้วย boolean mask
matrix[matrix > 5] = 0
print("After setting >5 to 0:\n", matrix)

# row-based condition
data = np.array([[1, 5, 3],
                 [2, 8, 4],
                 [6, 2, 9]])
row_mask = data[:, 1] > 4  # เลือก rows ที่ column 1 > 4
print("\nRows where col[1] > 4:\n", data[row_mask])
```

```python
# ตัวอย่างที่ 23: np.where() - conditional replacement
import numpy as np

arr = np.array([1, -2, 3, -4, 5, -6, 7, -8])

# np.where(condition, value_if_true, value_if_false)
result = np.where(arr > 0, arr, 0)  # เก็บค่าบวก, ค่าลบให้เป็น 0
print("Replace negatives with 0:", result)  # [1 0 3 0 5 0 7 0]

result2 = np.where(arr > 0, 'positive', 'negative')
print("Sign labels:", result2)

# np.where กับเงื่อนไขหลายชั้น (ใช้ nested np.where)
# หรือใช้ np.select
conditions = [arr > 0, arr < 0, arr == 0]
choices = ['positive', 'negative', 'zero']
result3 = np.select(conditions, choices)
print("With np.select:", result3)
```

---

## 7. Fancy Indexing

### การ Index ด้วย Arrays

```python
# ตัวอย่างที่ 24: Fancy Indexing พื้นฐาน
import numpy as np

arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

# ใช้ list of indices
indices = [0, 2, 5, 8]
print("Fancy indexing:", arr[indices])  # [10 30 60 90]

# สามารถซ้ำ index ได้
print("Repeated indices:", arr[[0, 0, 3, 3, 7]])  # [10 10 40 40 80]

# Fancy indexing สร้าง COPY (ต่างจาก slicing ที่สร้าง view)
copy = arr[[0, 2, 4]]
copy[0] = 999
print("Original unchanged:", arr[0])  # 10 (ไม่เปลี่ยน)
```

```python
# ตัวอย่างที่ 25: Fancy Indexing กับ 2D arrays
import numpy as np

matrix = np.array([[1,  2,  3,  4],
                   [5,  6,  7,  8],
                   [9, 10, 11, 12],
                   [13, 14, 15, 16]])

# เลือก rows ด้วย list
print("Rows 0, 2:", matrix[[0, 2]])
# [[ 1  2  3  4]
#  [ 9 10 11 12]]

# เลือก elements เฉพาะ: matrix[[rows], [cols]]
rows = [0, 1, 2, 3]
cols = [0, 1, 2, 3]
print("Diagonal:", matrix[rows, cols])  # [ 1  6 11 16] (diagonal)

# เลือก elements ที่ตำแหน่งต่างๆ
print("Specific elements:", matrix[[0,1,2], [3,2,1]])  # [4 7 10]
# element (0,3)=4, (1,2)=7, (2,1)=10

# การเรียงลำดับ rows ด้วย fancy indexing
reorder = matrix[[2, 0, 3, 1]]  # เรียง rows ใหม่
print("\nReordered matrix:\n", reorder)
```

```python
# ตัวอย่างที่ 26: np.take() และ np.put()
import numpy as np

arr = np.array([100, 200, 300, 400, 500])

# np.take - เหมือน fancy indexing แต่ใช้กับ multidimensional ได้ง่ายกว่า
taken = np.take(arr, [1, 3, 4])
print("np.take:", taken)  # [200 400 500]

# np.put - แทนที่ค่าที่ index ที่กำหนด
arr_copy = arr.copy()
np.put(arr_copy, [0, 2], [999, 888])
print("After np.put:", arr_copy)  # [999 200 888 400 500]
```

---

## 8. Array Operations (Element-wise)

### การคำนวณ Element-wise

```python
# ตัวอย่างที่ 27: Arithmetic operations
import numpy as np

a = np.array([1, 2, 3, 4, 5])
b = np.array([10, 20, 30, 40, 50])

# Operations ทำ element-wise (แต่ละ element คู่กัน)
print("a + b:", a + b)    # [11 22 33 44 55]
print("a - b:", a - b)    # [-9 -18 -27 -36 -45]
print("a * b:", a * b)    # [10 40 90 160 250]
print("a / b:", a / b)    # [0.1 0.1 0.1 0.1 0.1]
print("a // b:", a // b)  # [0 0 0 0 0] (floor division)
print("a % b:", a % b)    # [1 2 3 4 5] (modulo)
print("a ** 2:", a ** 2)  # [1 4 9 16 25] (power)

# Operations กับ scalar
print("\na + 100:", a + 100)   # [101 102 103 104 105]
print("a * 3:", a * 3)         # [3 6 9 12 15]
print("a / 2:", a / 2)         # [0.5 1.  1.5 2.  2.5]

# Comparison operations (return boolean array)
print("\na > 3:", a > 3)        # [False False False True True]
print("a == 3:", a == 3)        # [False False True False False]
```

```python
# ตัวอย่างที่ 28: Universal Functions (ufuncs)
import numpy as np

arr = np.array([0, np.pi/6, np.pi/4, np.pi/3, np.pi/2])

# Trigonometric functions
print("sin:", np.sin(arr))
print("cos:", np.cos(arr))
print("tan:", np.tan(arr[:4]))  # avoid tan(π/2) = infinity

# Exponential and logarithm
x = np.array([1, 2, 4, 8, 16])
print("\nnp.exp:", np.exp(np.array([0, 1, 2])))   # [1, e, e²]
print("np.log:", np.log(x))         # natural log
print("np.log2:", np.log2(x))       # log base 2
print("np.log10:", np.log10(x))     # log base 10

# Rounding
floats = np.array([1.2, 2.7, -1.5, -2.3, 0.5])
print("\nnp.round:", np.round(floats))     # [1. 3. -2. -2.  0.] (banker's rounding)
print("np.floor:", np.floor(floats))       # [ 1.  2. -2. -3.  0.]
print("np.ceil:", np.ceil(floats))         # [ 2.  3. -1. -2.  1.]
print("np.trunc:", np.trunc(floats))       # [ 1.  2. -1. -2.  0.]
print("np.abs:", np.abs(floats))           # [1.2 2.7 1.5 2.3 0.5]
```

---

## 9. Broadcasting

### กฎของ Broadcasting

Broadcasting คือกลไกที่ NumPy ใช้ทำ operations กับ arrays ที่มี shape ต่างกัน
โดยไม่ต้องสร้างสำเนาข้อมูลจริงๆ

**กฎของ Broadcasting:**
1. หาก arrays มี ndim ต่างกัน ให้เพิ่ม leading 1s ให้ array ที่มี ndim น้อยกว่า
2. Arrays ที่มีขนาด 1 ใน dimension ใด จะถูก "stretched" ให้ match กับ array อื่น
3. หาก size ไม่ compatible และไม่มีขนาด 1 จะเกิด error

```python
# ตัวอย่างที่ 29: Broadcasting พื้นฐาน
import numpy as np

# scalar กับ array (1D)
arr = np.array([1, 2, 3, 4, 5])
result = arr + 10  # scalar 10 ถูก broadcast ไปทุก element
print("arr + 10:", result)

# 1D กับ 2D
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
row = np.array([10, 20, 30])  # shape (3,)

# row ถูก broadcast ทุก row ของ matrix
result = matrix + row
print("\nmatrix + row:\n", result)
# [[11 22 33]
#  [14 25 36]
#  [17 28 39]]
```

```python
# ตัวอย่างที่ 30: Broadcasting 2D - column vector
import numpy as np

matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

# column vector shape (3, 1)
col = np.array([[10], [20], [30]])  # shape (3, 1)

# col ถูก broadcast ทุก column
result = matrix + col
print("matrix + column vector:\n", result)
# [[11 12 13]
#  [24 25 26]
#  [37 38 39]]
```

```python
# ตัวอย่างที่ 31: Broadcasting rules - การทำความเข้าใจ
import numpy as np

# Shapes ที่ compatible กัน:
# (4, 3) + (3,)   -> (4, 3)  [row ถูก broadcast]
# (4, 1) + (1, 3) -> (4, 3)  [ทั้งสองถูก broadcast]
# (4, 3) + (1,)   -> (4, 3)  [scalar broadcast]

a = np.ones((4, 3))        # shape (4, 3)
b = np.arange(3)           # shape (3,) -> treated as (1, 3)
c = np.arange(4)[:, None]  # shape (4, 1)

print("a.shape:", a.shape)  # (4, 3)
print("b.shape:", b.shape)  # (3,)
print("c.shape:", c.shape)  # (4, 1)

result_ab = a + b
result_ac = a + c
result_bc = b + c  # (3,) + (4,1) -> (4, 3)

print("\na + b shape:", result_ab.shape)  # (4, 3)
print("a + c shape:", result_ac.shape)  # (4, 3)
print("b + c shape:", result_bc.shape)  # (4, 3)
print("b + c:\n", result_bc)

# Error: shapes ที่ไม่ compatible
try:
    d = np.ones((3, 4))
    e = np.ones((4, 3))
    result = d + e  # Error! (3,4) + (4,3) ไม่ compatible
except ValueError as err:
    print(f"\nError: {err}")
```

```python
# ตัวอย่างที่ 32: Broadcasting use case จริง - normalization
import numpy as np

# Normalize matrix: (x - mean) / std สำหรับแต่ละ column
data = np.array([[1.0, 2.0, 3.0],
                 [4.0, 5.0, 6.0],
                 [7.0, 8.0, 9.0],
                 [10.0, 11.0, 12.0]])

# คำนวณ mean และ std ของแต่ละ column
col_mean = data.mean(axis=0)  # shape (3,)
col_std = data.std(axis=0)    # shape (3,)

print("Column means:", col_mean)
print("Column stds:", col_std)

# Normalize ด้วย broadcasting
normalized = (data - col_mean) / col_std  # (4,3) - (3,) -> broadcast
print("\nNormalized data:\n", normalized)
print("Normalized means (should be ~0):", normalized.mean(axis=0))
print("Normalized stds (should be ~1):", normalized.std(axis=0))
```

---

## 10. Array Math Operations

### การคำนวณทางคณิตศาสตร์

```python
# ตัวอย่างที่ 33: Aggregate functions
import numpy as np

arr = np.array([[1, 2, 3],
                [4, 5, 6],
                [7, 8, 9]])

# Global aggregation (ทั้ง array)
print("Sum:", arr.sum())          # 45
print("Mean:", arr.mean())        # 5.0
print("Max:", arr.max())          # 9
print("Min:", arr.min())          # 1
print("Std:", arr.std())          # 2.581...
print("Var:", arr.var())          # 6.666...

# Aggregation ตาม axis
print("\nSum axis=0 (per column):", arr.sum(axis=0))   # [12 15 18]
print("Sum axis=1 (per row):", arr.sum(axis=1))         # [6 15 24]
print("Mean axis=0:", arr.mean(axis=0))                  # [4. 5. 6.]
print("Max axis=1:", arr.max(axis=1))                    # [3 6 9]
```

```python
# ตัวอย่างที่ 34: Cumulative functions
import numpy as np

arr = np.array([1, 2, 3, 4, 5])

print("cumsum:", np.cumsum(arr))   # [1 3 6 10 15] (1, 1+2, 1+2+3, ...)
print("cumprod:", np.cumprod(arr)) # [1 2 6 24 120] (1, 1*2, 1*2*3, ...)

# 2D cumsum
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print("\nmatrix cumsum axis=0:\n", np.cumsum(matrix, axis=0))
print("matrix cumsum axis=1:\n", np.cumsum(matrix, axis=1))
```

```python
# ตัวอย่างที่ 35: Dot product และ Matrix multiplication
import numpy as np

# Dot product (1D vectors)
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
dot = np.dot(a, b)  # 1*4 + 2*5 + 3*6 = 4 + 10 + 18 = 32
print("Dot product:", dot)

# Matrix multiplication (2D)
A = np.array([[1, 2], [3, 4]])  # shape (2, 2)
B = np.array([[5, 6], [7, 8]])  # shape (2, 2)

# Matrix multiply (ไม่ใช่ element-wise!)
result1 = np.dot(A, B)
result2 = A @ B  # operator @ (Python 3.5+)
print("\nMatrix A:\n", A)
print("Matrix B:\n", B)
print("A @ B:\n", result2)
# [[19 22]
#  [43 50]]

# matmul vs dot - ต่างกันตอน batch operations
print("\nnp.matmul(A, B):\n", np.matmul(A, B))  # same as @
```

```python
# ตัวอย่างที่ 36: Special math operations
import numpy as np

arr = np.array([4.0, 9.0, 16.0, 25.0])

print("np.sqrt:", np.sqrt(arr))     # [2. 3. 4. 5.]
print("np.cbrt:", np.cbrt(arr))     # cube root: [1.587 2.08 2.52 2.924]
print("np.square:", np.square(arr)) # [16. 81. 256. 625.]

# Trigonometric (in radians)
angles = np.array([0, np.pi/4, np.pi/2, np.pi])
print("\nnp.sin:", np.sin(angles).round(4))  # [0. 0.7071 1. 0.]
print("np.cos:", np.cos(angles).round(4))    # [1. 0.7071 0. -1.]

# Degree conversion
degrees = np.array([0, 45, 90, 180, 270, 360])
radians = np.deg2rad(degrees)
print("\ndegrees to radians:", radians.round(4))

# Hyperbolic
x = np.array([0, 1, 2])
print("\nnp.sinh:", np.sinh(x))
print("np.cosh:", np.cosh(x))
```

---

## 11. Statistical Functions

### ฟังก์ชันทางสถิติ

```python
# ตัวอย่างที่ 37: Basic statistics
import numpy as np

data = np.array([23, 45, 12, 67, 34, 89, 56, 78, 11, 90])

print(f"Mean: {np.mean(data):.2f}")        # 50.5
print(f"Median: {np.median(data):.2f}")    # 50.5
print(f"Std: {np.std(data):.2f}")          # 27.99
print(f"Var: {np.var(data):.2f}")          # 783.85
print(f"Min: {np.min(data)}")              # 11
print(f"Max: {np.max(data)}")              # 90
print(f"Range: {np.max(data) - np.min(data)}")  # 79

# Percentiles
print(f"\n25th percentile: {np.percentile(data, 25):.2f}")
print(f"50th percentile (median): {np.percentile(data, 50):.2f}")
print(f"75th percentile: {np.percentile(data, 75):.2f}")

# Multiple percentiles at once
p = np.percentile(data, [0, 25, 50, 75, 100])
print(f"Quartiles: {p}")
```

```python
# ตัวอย่างที่ 38: argmin, argmax - หา index ของค่า min/max
import numpy as np

arr = np.array([10, 25, 3, 47, 15, 8, 62, 33])

print("Max value:", np.max(arr))           # 62
print("Index of max:", np.argmax(arr))     # 6
print("Min value:", np.min(arr))           # 3
print("Index of min:", np.argmin(arr))     # 2

# 2D argmax/argmin
matrix = np.array([[3, 9, 2],
                   [1, 7, 5],
                   [8, 4, 6]])

print("\nGlobal argmax:", np.argmax(matrix))  # index ใน flattened array
print("argmax axis=0:", np.argmax(matrix, axis=0))  # index ของ max ใน แต่ละ column
print("argmax axis=1:", np.argmax(matrix, axis=1))  # index ของ max ใน แต่ละ row
```

```python
# ตัวอย่างที่ 39: Correlation และ Covariance
import numpy as np

# สร้างข้อมูล 2 variables
x = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
y = 2 * x + np.random.default_rng(42).normal(0, 1, 10)  # y ≈ 2x + noise

# Correlation coefficient (-1 ถึง 1)
# 1 = perfect positive correlation, -1 = perfect negative, 0 = no correlation
corr = np.corrcoef(x, y)
print("Correlation matrix:")
print(corr)
print(f"Correlation coefficient: {corr[0, 1]:.4f}")

# Covariance matrix
cov = np.cov(x, y)
print("\nCovariance matrix:")
print(cov)
```

---

## 12. Array Manipulation

### การจัดการ Arrays

```python
# ตัวอย่างที่ 40: reshape, flatten, ravel
import numpy as np

arr = np.arange(12)
matrix = arr.reshape(3, 4)

# flatten - สร้าง 1D copy
flat_copy = matrix.flatten()
flat_copy[0] = 999
print("Original matrix[0,0]:", matrix[0, 0])  # 0 (ไม่เปลี่ยน)
print("flatten creates copy:", flat_copy[:5])

# ravel - สร้าง 1D view (ถ้าเป็นไปได้)
flat_view = matrix.ravel()
flat_view[0] = 999
print("\nOriginal matrix[0,0] after ravel:", matrix[0, 0])  # 999 (เปลี่ยน!)
```

```python
# ตัวอย่างที่ 41: transpose
import numpy as np

# Transpose สลับ rows และ columns
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
print("Original shape:", matrix.shape)  # (2, 3)

transposed = matrix.T  # หรือ np.transpose(matrix)
print("Transposed shape:", transposed.shape)  # (3, 2)
print("Transposed:\n", transposed)

# Transpose 3D array
tensor = np.arange(24).reshape(2, 3, 4)
print("\n3D tensor shape:", tensor.shape)  # (2, 3, 4)

t_default = tensor.T  # reverse all axes
print("Transposed (default):", t_default.shape)  # (4, 3, 2)

t_custom = np.transpose(tensor, (1, 0, 2))  # กำหนด order ของ axes
print("Transposed (1,0,2):", t_custom.shape)  # (3, 2, 4)
```

```python
# ตัวอย่างที่ 42: concatenate, stack
import numpy as np

a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# concatenate - ต่อกันตาม axis ที่มีอยู่แล้ว
concat_0 = np.concatenate([a, b], axis=0)  # ต่อ rows (vertical)
concat_1 = np.concatenate([a, b], axis=1)  # ต่อ cols (horizontal)
print("concat axis=0:\n", concat_0)  # shape (4, 2)
print("\nconcat axis=1:\n", concat_1)  # shape (2, 4)

# vstack, hstack - shortcuts
print("\nvstack:\n", np.vstack([a, b]))   # เหมือน concat axis=0
print("\nhstack:\n", np.hstack([a, b]))   # เหมือน concat axis=1

# stack - สร้าง new axis
stacked = np.stack([a, b], axis=0)  # shape (2, 2, 2)
print("\nstack axis=0 shape:", stacked.shape)
stacked_1 = np.stack([a, b], axis=1)  # shape (2, 2, 2)
print("stack axis=1 shape:", stacked_1.shape)
```

```python
# ตัวอย่างที่ 43: split - แบ่ง array
import numpy as np

arr = np.arange(12)
print("Original:", arr)

# split เป็น 3 ส่วนเท่าๆ กัน
parts = np.split(arr, 3)
print("Split into 3:", parts)  # [array([0,1,2,3]), array([4,5,6,7]), ...]

# split ตาม indices
parts2 = np.split(arr, [3, 7])  # แบ่งที่ index 3 และ 7
print("Split at [3,7]:", parts2)  # [arr[:3], arr[3:7], arr[7:]]

# array_split - สำหรับการแบ่งที่ไม่ลงตัว
parts3 = np.array_split(arr, 5)  # แบ่งเป็น 5 ส่วน (ไม่เท่ากันได้)
print("array_split into 5:", [len(p) for p in parts3])
```

```python
# ตัวอย่างที่ 44: sort และ argsort
import numpy as np

arr = np.array([3, 1, 4, 1, 5, 9, 2, 6, 5, 3])

# sort - เรียงลำดับ (สร้าง copy)
sorted_arr = np.sort(arr)
print("Sorted:", sorted_arr)  # [1 1 2 3 3 4 5 5 6 9]

# argsort - return indices ที่จะทำให้ array เรียงลำดับ
indices = np.argsort(arr)
print("Argsort indices:", indices)
print("Verify:", arr[indices])  # เหมือน sorted_arr

# sort descending
sorted_desc = np.sort(arr)[::-1]
print("Sorted descending:", sorted_desc)

# sort 2D array
matrix = np.array([[3, 1, 4],
                   [1, 5, 9],
                   [2, 6, 5]])
print("\nSorted axis=0 (column-wise):\n", np.sort(matrix, axis=0))
print("Sorted axis=1 (row-wise):\n", np.sort(matrix, axis=1))
```

```python
# ตัวอย่างที่ 45: unique - หาค่าที่ไม่ซ้ำ
import numpy as np

arr = np.array([3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5])

# หาค่า unique
unique_vals = np.unique(arr)
print("Unique values:", unique_vals)  # [1 2 3 4 5 6 9]

# พร้อม count
unique_vals, counts = np.unique(arr, return_counts=True)
print("Unique values:", unique_vals)
print("Counts:", counts)
for val, count in zip(unique_vals, counts):
    print(f"  {val}: {count} times")

# หา index ของ unique values ใน original array
unique_vals, indices = np.unique(arr, return_index=True)
print("\nFirst occurrence indices:", indices)
```

```python
# ตัวอย่างที่ 46: tile และ repeat
import numpy as np

arr = np.array([1, 2, 3])

# repeat - ทำซ้ำแต่ละ element
rep = np.repeat(arr, 3)
print("repeat by 3:", rep)  # [1 1 1 2 2 2 3 3 3]

# repeat ต่างกันในแต่ละ element
rep2 = np.repeat(arr, [1, 2, 3])
print("repeat [1,2,3]:", rep2)  # [1 2 2 3 3 3]

# tile - ทำซ้ำ array ทั้งหมด
tiled = np.tile(arr, 3)
print("tile by 3:", tiled)  # [1 2 3 1 2 3 1 2 3]

# tile ใน 2D
tiled_2d = np.tile(arr, (2, 3))  # ทำซ้ำ 2 รอบตาม row, 3 รอบตาม col
print("tile (2,3):\n", tiled_2d)
```

---

## 13. แบบฝึกหัด

### ข้อ 1: Array Creation และ Properties
สร้าง array ต่อไปนี้และแสดงข้อมูล (shape, dtype, size, ndim):
- Array 1D ของเลข 1-20 เฉพาะเลขคี่
- Array 2D (5x5) ที่มีค่า 0 ยกเว้น diagonal ที่เป็น 1, 2, 3, 4, 5
- Array 3D (2x3x4) ที่เติมด้วย random floats ระหว่าง 0-100

```python
# เฉลยข้อ 1
import numpy as np

# Array 1D เลขคี่ 1-20
odd_arr = np.arange(1, 21, 2)
print("Odd numbers 1-20:", odd_arr)
print(f"  shape={odd_arr.shape}, dtype={odd_arr.dtype}, size={odd_arr.size}")

# Array 2D 5x5 diagonal
diag_arr = np.diag([1, 2, 3, 4, 5])
print("\nDiagonal 5x5:\n", diag_arr)
print(f"  shape={diag_arr.shape}, ndim={diag_arr.ndim}")

# Array 3D random
rng = np.random.default_rng(42)
rand_arr = rng.uniform(0, 100, size=(2, 3, 4))
print("\nRandom 3D array shape:", rand_arr.shape)
print("  Min:", rand_arr.min().round(2), "Max:", rand_arr.max().round(2))
```

### ข้อ 2: Slicing
กำหนดให้ matrix เป็น array 8x8 (ค่า 0-63) ให้ extract:
- แถวที่ 3 และ 5
- คอลัมน์ที่ 2, 4, 6
- Sub-matrix 4x4 ตรงกลาง (rows 2-5, cols 2-5)
- Checkerboard pattern (elements ที่ sum ของ indices เป็นเลขคู่)

```python
# เฉลยข้อ 2
import numpy as np

matrix = np.arange(64).reshape(8, 8)
print("Matrix:\n", matrix)

# แถวที่ 3 และ 5
rows_35 = matrix[[3, 5]]
print("\nRows 3 and 5:\n", rows_35)

# คอลัมน์ที่ 2, 4, 6
cols_246 = matrix[:, [2, 4, 6]]
print("\nColumns 2, 4, 6:\n", cols_246)

# Sub-matrix 4x4 ตรงกลาง
center = matrix[2:6, 2:6]
print("\nCenter 4x4:\n", center)

# Checkerboard pattern
rows, cols = np.indices((8, 8))
checkerboard = matrix[(rows + cols) % 2 == 0]
print("\nCheckerboard elements:", checkerboard)
```

### ข้อ 3: Boolean Indexing
กำหนดให้ grades เป็น array คะแนน 30 คน (random 40-100) ให้:
- นับจำนวนนักเรียนที่สอบผ่าน (>= 60)
- หา index ของนักเรียนที่ได้ A (>= 80)
- แทนที่คะแนน < 50 ด้วย 50 (pass by substitution)
- คำนวณ mean เฉพาะคนที่สอบผ่าน

```python
# เฉลยข้อ 3
import numpy as np

rng = np.random.default_rng(123)
grades = rng.integers(40, 101, size=30)
print("Grades:", grades)

# นับจำนวนที่ผ่าน
passed = grades >= 60
print(f"\nPassed: {passed.sum()} students out of {len(grades)}")

# index ของนักเรียนที่ได้ A
a_indices = np.where(grades >= 80)[0]
print("A grade indices:", a_indices)
print("A grade scores:", grades[a_indices])

# แทนที่คะแนน < 50
grades_adjusted = grades.copy()
grades_adjusted[grades_adjusted < 50] = 50
print("\nAdjusted grades (min 50):", grades_adjusted)

# mean ของคนที่ผ่าน
pass_mean = grades[passed].mean()
print(f"Mean of passing students: {pass_mean:.2f}")
```

### ข้อ 4: Broadcasting
เขียนฟังก์ชัน `distance_matrix(points)` ที่รับ array shape (n, 2) ของ coordinates
และ return matrix (n, n) ที่แต่ละ element [i,j] คือ Euclidean distance ระหว่าง point i และ j
(ไม่ใช้ loop, ใช้ broadcasting)

```python
# เฉลยข้อ 4
import numpy as np

def distance_matrix(points):
    """
    คำนวณ pairwise Euclidean distance ด้วย broadcasting
    points: shape (n, 2)
    return: shape (n, n)
    """
    # points[:, np.newaxis, :] -> shape (n, 1, 2)
    # points[np.newaxis, :, :] -> shape (1, n, 2)
    # diff = (n, n, 2) โดย broadcasting
    diff = points[:, np.newaxis, :] - points[np.newaxis, :, :]
    # sum of squares ตาม axis=2, แล้ว sqrt
    return np.sqrt(np.sum(diff**2, axis=2))

# ทดสอบ
points = np.array([[0, 0], [3, 4], [1, 1], [6, 8]])
dist = distance_matrix(points)
print("Distance matrix:")
print(dist.round(2))
# point 0 ถึง point 1: sqrt(9+16) = 5.0 ✓
print(f"\nDistance from (0,0) to (3,4): {dist[0, 1]:.2f}")  # 5.0
```

### ข้อ 5: Statistical Analysis
กำหนดให้ sales_data เป็น matrix (12, 5) แทน sales รายเดือน (rows) ของ 5 สาขา (cols)
ให้:
- หา total sales ของแต่ละสาขา
- หา best month สำหรับแต่ละสาขา
- หา worst performing branch (total sales ต่ำสุด)
- Normalize ข้อมูลแต่ละสาขา (min-max scaling ระหว่าง 0-1)

```python
# เฉลยข้อ 5
import numpy as np

rng = np.random.default_rng(42)
sales_data = rng.integers(100, 500, size=(12, 5)).astype(float)
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
          'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
branches = ['Branch A', 'Branch B', 'Branch C', 'Branch D', 'Branch E']

# Total sales ของแต่ละสาขา
total_sales = sales_data.sum(axis=0)
for branch, total in zip(branches, total_sales):
    print(f"{branch}: {total:.0f}")

# Best month สำหรับแต่ละสาขา
best_month_idx = sales_data.argmax(axis=0)
for branch, month_idx in zip(branches, best_month_idx):
    print(f"{branch} best month: {months[month_idx]}")

# Worst performing branch
worst_branch_idx = total_sales.argmin()
print(f"\nWorst branch: {branches[worst_branch_idx]}")

# Min-max normalization
col_min = sales_data.min(axis=0)   # shape (5,)
col_max = sales_data.max(axis=0)   # shape (5,)
normalized = (sales_data - col_min) / (col_max - col_min)  # broadcasting
print("\nNormalized min:", normalized.min(axis=0).round(4))
print("Normalized max:", normalized.max(axis=0).round(4))
```

### ข้อ 6: Array Manipulation
ทำงานกับ images (แทนด้วย arrays):
- สร้าง grayscale image (100x100) random
- Flip horizontally และ vertically
- Crop ส่วนกลาง 50x50
- สร้าง thumbnail โดย average ทุก 2x2 block เป็น 1 pixel (downscale 50x50)

```python
# เฉลยข้อ 6
import numpy as np

rng = np.random.default_rng(42)
image = rng.integers(0, 256, size=(100, 100), dtype=np.uint8)
print("Image shape:", image.shape)

# Flip horizontally (left-right)
flipped_h = np.fliplr(image)
print("Flipped horizontally shape:", flipped_h.shape)

# Flip vertically (up-down)
flipped_v = np.flipud(image)
print("Flipped vertically shape:", flipped_v.shape)

# Crop center 50x50
start_r = (100 - 50) // 2  # = 25
start_c = (100 - 50) // 2  # = 25
cropped = image[start_r:start_r+50, start_c:start_c+50]
print("Cropped shape:", cropped.shape)

# Downscale 2x average pooling -> 50x50
# reshape เป็น (50, 2, 50, 2) แล้ว mean ตาม axis 1 และ 3
thumbnail = image.reshape(50, 2, 50, 2).mean(axis=(1, 3)).astype(np.uint8)
print("Thumbnail shape:", thumbnail.shape)
print("Original mean:", image.mean().round(2))
print("Thumbnail mean:", thumbnail.mean().round(2))
```

### ข้อ 7: Fancy Indexing
กำหนดให้ student_scores เป็น matrix (50, 5) (50 นักเรียน, 5 วิชา) ให้:
- หา top 10 นักเรียนตาม total score
- แสดง scores ของ top 10 นักเรียน
- หา rank ของแต่ละนักเรียน (1 = คะแนนรวมสูงสุด)

```python
# เฉลยข้อ 7
import numpy as np

rng = np.random.default_rng(42)
scores = rng.integers(40, 101, size=(50, 5))
print("Scores shape:", scores.shape)

# Total score ของแต่ละนักเรียน
totals = scores.sum(axis=1)  # shape (50,)

# หา top 10 ด้วย argsort (เรียงจากมากไปน้อย = reverse)
sorted_indices = np.argsort(totals)[::-1]
top10_indices = sorted_indices[:10]

print("\nTop 10 student indices:", top10_indices)
print("Top 10 total scores:", totals[top10_indices])
print("\nTop 10 scores by subject:\n", scores[top10_indices])

# Rank (1 = highest)
# argsort ของ argsort ให้ rank
ranks = np.argsort(np.argsort(totals)[::-1]) + 1
print("\nRanks (first 10 students):", ranks[:10])
```

### ข้อ 8: Broadcasting Application - Pairwise Comparison
เขียนฟังก์ชันที่รับ 1D array และ return boolean matrix (n,n)
โดย matrix[i,j] = True ถ้า arr[i] > arr[j]

```python
# เฉลยข้อ 8
import numpy as np

def pairwise_greater(arr):
    """Return boolean matrix where [i,j] = True if arr[i] > arr[j]"""
    # arr[:, None] shape (n, 1), arr[None, :] shape (1, n)
    # broadcasting -> (n, n)
    return arr[:, np.newaxis] > arr[np.newaxis, :]

arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
result = pairwise_greater(arr)
print("Array:", arr)
print("Pairwise greater matrix:")
print(result.astype(int))

# นับว่าแต่ละ element ชนะกี่ element (rank-like)
wins = result.sum(axis=1)
print("\nWins per element:", wins)
print("(This is equivalent to rank from 0)")
```

### ข้อ 9: NumPy ทำ Image Processing เบื้องต้น
ทำ RGB image processing:
- สร้าง RGB image 200x200x3 (random noise)
- Swap R and B channels (effect คล้าย BGR to RGB)
- แปลงเป็น grayscale ด้วย formula: 0.299R + 0.587G + 0.114B
- Apply threshold: pixels > 128 เป็น 255, อื่นๆ เป็น 0

```python
# เฉลยข้อ 9
import numpy as np

rng = np.random.default_rng(42)
image = rng.integers(0, 256, size=(200, 200, 3), dtype=np.uint8)
print("RGB image shape:", image.shape)

# Swap R and B channels (index 0 และ 2)
bgr_image = image.copy()
bgr_image[:, :, 0] = image[:, :, 2]  # R = B
bgr_image[:, :, 2] = image[:, :, 0]  # B = R (original)
# หรือใช้ fancy indexing
bgr_image2 = image[:, :, [2, 1, 0]]   # reverse channels
print("BGR image shape:", bgr_image2.shape)

# แปลงเป็น grayscale
weights = np.array([0.299, 0.587, 0.114])
# dot product ตาม axis ที่ 2 (channels)
grayscale = (image * weights).sum(axis=2).astype(np.uint8)
print("Grayscale shape:", grayscale.shape)

# Apply threshold
binary = np.where(grayscale > 128, 255, 0).astype(np.uint8)
print("Binary shape:", binary.shape)
print("Binary values:", np.unique(binary))  # [0, 255]
print(f"White pixels: {(binary == 255).sum()}, Black pixels: {(binary == 0).sum()}")
```

### ข้อ 10: Performance Optimization
เปรียบเทียบ 3 วิธีคำนวณ moving average ของ array ขนาด 1 ล้านตัว:
1. Python loop + list
2. NumPy loop (ยังคง loop แต่ใช้ NumPy operations)
3. Vectorized NumPy (ไม่มี loop)

```python
# เฉลยข้อ 10
import numpy as np
import time

n = 1_000_000
window = 5
data = np.random.default_rng(42).random(n)

# Method 1: Pure Python loop
def moving_avg_python(data, window):
    result = []
    data_list = data.tolist()
    for i in range(len(data_list) - window + 1):
        result.append(sum(data_list[i:i+window]) / window)
    return result

# Method 2: NumPy loop
def moving_avg_numpy_loop(data, window):
    n = len(data) - window + 1
    result = np.empty(n)
    for i in range(n):
        result[i] = np.mean(data[i:i+window])
    return result

# Method 3: Vectorized NumPy (ใช้ cumsum trick)
def moving_avg_vectorized(data, window):
    cumsum = np.cumsum(np.insert(data, 0, 0))
    return (cumsum[window:] - cumsum[:-window]) / window

# วัดเวลา (method 1 ช้ามาก ข้ามไป)
print("Testing with 10000 elements...")
data_small = data[:10000]

t0 = time.time()
r1 = moving_avg_python(data_small, window)
t1 = time.time()
print(f"Python loop: {t1-t0:.4f}s")

t0 = time.time()
r2 = moving_avg_numpy_loop(data_small, window)
t1 = time.time()
print(f"NumPy loop: {t1-t0:.4f}s")

t0 = time.time()
r3 = moving_avg_vectorized(data_small, window)
t1 = time.time()
print(f"Vectorized NumPy: {t1-t0:.6f}s")

# Full dataset
print("\nTesting with 1,000,000 elements (vectorized only)...")
t0 = time.time()
r_full = moving_avg_vectorized(data, window)
t1 = time.time()
print(f"Vectorized on 1M elements: {t1-t0:.4f}s")
print(f"Result shape: {r_full.shape}")
print(f"First 5 values: {r_full[:5]}")
```

---

## สรุป Part 71

ใน Part นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| NumPy basics | ทำไม NumPy เร็วกว่า Python list, การติดตั้ง |
| ndarray | attributes (shape, dtype, size, ndim), data types |
| Creating arrays | zeros, ones, arange, linspace, random, eye, diag |
| Shapes | reshape, expand_dims, squeeze, newaxis |
| Indexing | 1D, 2D, 3D indexing และ slicing |
| View vs Copy | slice = view, copy() = สำเนาอิสระ |
| Boolean indexing | mask, np.where, np.select |
| Fancy indexing | integer array indexing, สร้าง copy |
| Operations | element-wise arithmetic, ufuncs |
| Broadcasting | กฎ broadcasting, use cases |
| Statistics | mean, std, percentile, argmax, corrcoef |
| Manipulation | reshape, transpose, concatenate, split, sort |

### Key Concepts ที่สำคัญ

1. **NumPy เร็วเพราะ**: contiguous memory + fixed type + vectorized C operations
2. **View vs Copy**: slice ให้ view (เปลี่ยน view = เปลี่ยน original!), ใช้ `.copy()` ถ้าต้องการ copy
3. **Broadcasting**: arrays ที่ shape compatible จะ "stretch" อัตโนมัติ ประหยัด memory
4. **Vectorization**: หลีกเลี่ยง Python loops ใช้ NumPy operations แทนเสมอ
5. **dtype ที่เหมาะสม**: เลือก dtype ให้เหมาะกับข้อมูลเพื่อประหยัด memory

---

**ต่อไป**: [Part 72 - NumPy Advanced Operations](../part72/README.md)
