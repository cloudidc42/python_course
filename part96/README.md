# Part 96: Performance Optimization & Profiling

## สารบัญ

1. [Introduction to Performance Optimization](#introduction)
2. [Performance Profiling Tools](#profiling-tools)
3. [Benchmarking with timeit](#benchmarking)
4. [Algorithm Complexity (Big O)](#big-o)
5. [Python Performance Tips](#performance-tips)
6. [List vs Generator Performance](#list-vs-generator)
7. [NumPy Vectorization](#numpy-vectorization)
8. [Cython เบื้องต้น](#cython)
9. [ctypes และ cffi](#ctypes-cffi)
10. [Numba JIT Compilation](#numba)
11. [PyPy](#pypy)
12. [Memory Optimization](#memory-optimization)
13. [String Formatting Performance](#string-formatting)
14. [Dictionary vs if-elif Chains](#dict-vs-ifelif)
15. [Database Query Optimization](#database-optimization)
16. [API Response Time Optimization](#api-optimization)
17. [แบบฝึกหัด](#exercises)

---

## 1. Introduction to Performance Optimization <a name="introduction"></a>

การ optimize performance ของโปรแกรม Python เป็นทักษะสำคัญที่นักพัฒนาระดับสูงต้องมี หลักการสำคัญคือ:

> **"Premature optimization is the root of all evil"** - Donald Knuth

ขั้นตอนที่ถูกต้องในการ optimize:
1. **Measure first** - วัดก่อนเสมอ ไม่เดาสุ่ม
2. **Profile** - หา bottleneck ที่แท้จริง
3. **Optimize** - แก้ไขเฉพาะจุดที่มีปัญหา
4. **Verify** - ตรวจสอบว่า optimize แล้วดีขึ้นจริง

```python
# ตัวอย่าง 1: วัด performance แบบง่าย
import time

def measure_time(func):
    """Decorator สำหรับวัดเวลาการทำงานของฟังก์ชัน"""
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.6f} seconds")
        return result
    return wrapper

@measure_time
def slow_function():
    total = 0
    for i in range(1_000_000):
        total += i
    return total

@measure_time
def fast_function():
    return sum(range(1_000_000))

result1 = slow_function()
result2 = fast_function()
print(f"Results match: {result1 == result2}")
```

---

## 2. Performance Profiling Tools <a name="profiling-tools"></a>

### 2.1 cProfile - The Built-in Profiler

`cProfile` เป็น built-in profiler ที่มากับ Python บอกว่าฟังก์ชันไหนใช้เวลานานแค่ไหน

```python
# ตัวอย่าง 2: การใช้ cProfile
import cProfile
import pstats
import io

def fibonacci(n):
    """คำนวณ Fibonacci แบบ recursive (จงใจให้ช้า)"""
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

def matrix_multiply(size=100):
    """คูณ matrix แบบ naive"""
    A = [[i + j for j in range(size)] for i in range(size)]
    B = [[i * j for j in range(size)] for i in range(size)]
    C = [[0] * size for _ in range(size)]
    
    for i in range(size):
        for j in range(size):
            for k in range(size):
                C[i][j] += A[i][k] * B[k][j]
    return C

def main():
    # ทำงานหลายอย่าง
    fib_result = fibonacci(30)
    matrix_result = matrix_multiply(50)
    return fib_result, matrix_result

# วิธีที่ 1: Profile แบบง่าย
# cProfile.run('main()')

# วิธีที่ 2: Profile พร้อม statistics
profiler = cProfile.Profile()
profiler.enable()

main()

profiler.disable()

# แสดงผลแบบ sorted
stream = io.StringIO()
stats = pstats.Stats(profiler, stream=stream)
stats.sort_stats('cumulative')  # เรียงตาม cumulative time
stats.print_stats(20)  # แสดง 20 อันดับแรก
print(stream.getvalue())
```

```python
# ตัวอย่าง 3: Profile เฉพาะบางส่วนของโค้ด
import cProfile
import pstats

class ProfileContext:
    """Context manager สำหรับ profiling"""
    
    def __init__(self, name="profile", sort_by='cumulative', lines=15):
        self.name = name
        self.sort_by = sort_by
        self.lines = lines
        self.profiler = cProfile.Profile()
    
    def __enter__(self):
        self.profiler.enable()
        return self
    
    def __exit__(self, *args):
        self.profiler.disable()
        stats = pstats.Stats(self.profiler)
        stats.sort_stats(self.sort_by)
        print(f"\n=== Profile: {self.name} ===")
        stats.print_stats(self.lines)

# การใช้งาน
def process_data(data):
    """ประมวลผลข้อมูล"""
    result = []
    for item in data:
        # simulate processing
        processed = item ** 2 + item ** 0.5
        result.append(processed)
    return result

def sort_and_filter(data):
    """เรียงและกรองข้อมูล"""
    sorted_data = sorted(data, reverse=True)
    filtered = [x for x in sorted_data if x > 100]
    return filtered

with ProfileContext("data processing"):
    data = list(range(10000))
    processed = process_data(data)
    filtered = sort_and_filter(processed)
    print(f"Processed {len(processed)} items, filtered to {len(filtered)}")
```

### 2.2 line_profiler - Profile ทีละบรรทัด

```python
# ตัวอย่าง 4: line_profiler (ต้องติดตั้ง: pip install line_profiler)
# การใช้งาน: kernprof -l -v script.py

# ไฟล์: profile_example.py
from line_profiler import LineProfiler

def process_list(items):
    """ฟังก์ชันที่ต้องการ profile ทีละบรรทัด"""
    result = []                           # บรรทัดนี้เร็ว
    
    for item in items:                    # loop หลัก
        # การคำนวณที่ต้องการวัด
        squared = item ** 2              # บรรทัดนี้ใช้เวลาเท่าไร?
        sqrt_val = item ** 0.5           # บรรทัดนี้ล่ะ?
        combined = squared + sqrt_val    # และบรรทัดนี้?
        
        if combined > 1000:              # condition check
            result.append(combined)     # append operation
    
    return sorted(result)               # sort ทั้งหมด

# Profile ด้วย LineProfiler
profiler = LineProfiler()
profiler.add_function(process_list)

# รัน function
test_data = list(range(10000))
profiler.runcall(process_list, test_data)

# แสดงผล
profiler.print_stats()
```

### 2.3 memory_profiler - วัดการใช้ Memory

```python
# ตัวอย่าง 5: memory_profiler (pip install memory_profiler)
from memory_profiler import memory_usage, profile
import tracemalloc

# วิธีที่ 1: ใช้ @profile decorator
# @profile
def memory_intensive_function():
    """ฟังก์ชันที่ใช้ memory มาก"""
    # สร้าง list ขนาดใหญ่
    big_list = [i for i in range(1_000_000)]          # ~8MB
    
    # สร้าง dict ขนาดใหญ่
    big_dict = {i: i**2 for i in range(100_000)}      # ~10MB
    
    # ประมวลผล
    result = sum(big_list)
    
    # ลบ reference เพื่อ free memory
    del big_list
    del big_dict
    
    return result

# วิธีที่ 2: ใช้ tracemalloc (built-in)
def track_memory_with_tracemalloc():
    """ติดตาม memory ด้วย tracemalloc"""
    tracemalloc.start()
    
    # สร้างข้อมูล
    data = {i: [j for j in range(100)] for i in range(1000)}
    
    # ดู snapshot
    snapshot = tracemalloc.take_snapshot()
    top_stats = snapshot.statistics('lineno')
    
    print("\nTop 5 memory allocations:")
    for stat in top_stats[:5]:
        print(f"  {stat}")
    
    current, peak = tracemalloc.get_traced_memory()
    print(f"\nCurrent memory: {current / 1024:.1f} KB")
    print(f"Peak memory: {peak / 1024:.1f} KB")
    
    tracemalloc.stop()
    del data

track_memory_with_tracemalloc()

# วิธีที่ 3: วัด memory usage ของฟังก์ชัน
mem_usage = memory_usage(memory_intensive_function)
print(f"\nMemory usage: {max(mem_usage):.1f} MB (peak)")
print(f"Memory usage: {min(mem_usage):.1f} MB (min)")
```

```python
# ตัวอย่าง 6: ใช้ objgraph สำหรับ memory leak detection
# pip install objgraph

import gc
import sys

def check_object_sizes():
    """ตรวจสอบขนาดของ objects ต่างๆ"""
    
    # ขนาดของ data types ต่างๆ
    data_types = {
        'int (0)': 0,
        'int (large)': 10**100,
        'float': 3.14,
        'str (empty)': '',
        'str (hello)': 'hello',
        'list (empty)': [],
        'list (100 items)': list(range(100)),
        'dict (empty)': {},
        'dict (100 items)': {i: i for i in range(100)},
        'tuple (empty)': (),
        'tuple (100 items)': tuple(range(100)),
        'set (empty)': set(),
        'set (100 items)': set(range(100)),
    }
    
    print("Object sizes:")
    print("-" * 40)
    for name, obj in data_types.items():
        size = sys.getsizeof(obj)
        print(f"  {name:30s}: {size:8d} bytes")

check_object_sizes()
```

---

## 3. Benchmarking with timeit <a name="benchmarking"></a>

```python
# ตัวอย่าง 7: timeit - เครื่องมือ benchmark มาตรฐาน
import timeit
from functools import lru_cache

# เปรียบเทียบวิธีการต่างๆ ในการสร้าง list
def benchmark_list_creation():
    
    # วิธีที่ 1: for loop
    setup1 = "data = []"
    stmt1 = """
for i in range(1000):
    data.append(i)
"""
    
    # วิธีที่ 2: list comprehension
    stmt2 = "data = [i for i in range(1000)]"
    
    # วิธีที่ 3: list()
    stmt3 = "data = list(range(1000))"
    
    # วิธีที่ 4: generator to list
    stmt4 = "data = [*range(1000)]"
    
    n = 10000
    
    t1 = timeit.timeit(stmt1, setup=setup1, number=n)
    t2 = timeit.timeit(stmt2, number=n)
    t3 = timeit.timeit(stmt3, number=n)
    t4 = timeit.timeit(stmt4, number=n)
    
    print("List Creation Benchmarks (10,000 iterations):")
    print(f"  for loop + append:    {t1:.3f}s")
    print(f"  list comprehension:   {t2:.3f}s")
    print(f"  list(range()):        {t3:.3f}s")
    print(f"  [*range()]:           {t4:.3f}s")
    
    fastest = min(t1, t2, t3, t4)
    print(f"\nRelative speeds (vs fastest):")
    print(f"  for loop:   {t1/fastest:.2f}x")
    print(f"  comprehension: {t2/fastest:.2f}x")
    print(f"  list():     {t3/fastest:.2f}x")
    print(f"  unpack:     {t4/fastest:.2f}x")

benchmark_list_creation()
```

```python
# ตัวอย่าง 8: เปรียบเทียบ string concatenation methods
import timeit

def benchmark_string_ops():
    n = 5000
    size = 100
    
    # วิธีที่ 1: + operator
    stmt1 = f"""
result = ''
for i in range({size}):
    result += str(i)
"""
    
    # วิธีที่ 2: join
    stmt2 = f"result = ''.join(str(i) for i in range({size}))"
    
    # วิธีที่ 3: f-string in list then join
    stmt3 = f"""
parts = [str(i) for i in range({size})]
result = ''.join(parts)
"""
    
    # วิธีที่ 4: io.StringIO
    stmt4 = f"""
import io
buf = io.StringIO()
for i in range({size}):
    buf.write(str(i))
result = buf.getvalue()
"""
    
    t1 = timeit.timeit(stmt1, number=n)
    t2 = timeit.timeit(stmt2, number=n)
    t3 = timeit.timeit(stmt3, number=n)
    t4 = timeit.timeit(stmt4, number=n)
    
    print("\nString Concatenation Benchmarks:")
    print(f"  + operator:      {t1:.3f}s")
    print(f"  join(generator): {t2:.3f}s")
    print(f"  list + join:     {t3:.3f}s")
    print(f"  StringIO:        {t4:.3f}s")

benchmark_string_ops()
```

```python
# ตัวอย่าง 9: การใช้ timeit.repeat สำหรับ statistical benchmarks
import timeit
import statistics

def statistical_benchmark(stmt, setup="pass", repeat=7, number=10000):
    """Benchmark พร้อม statistical analysis"""
    times = timeit.repeat(stmt, setup=setup, repeat=repeat, number=number)
    
    mean = statistics.mean(times)
    stdev = statistics.stdev(times)
    minimum = min(times)
    maximum = max(times)
    
    print(f"  Statement: {stmt[:50]}...")
    print(f"  Mean:   {mean*1000:.3f} ms")
    print(f"  StDev:  {stdev*1000:.3f} ms")
    print(f"  Min:    {minimum*1000:.3f} ms")
    print(f"  Max:    {maximum*1000:.3f} ms")
    print(f"  Coefficient of variation: {(stdev/mean)*100:.1f}%")
    return mean

# เปรียบเทียบ dict lookup vs attribute access
print("Dict lookup vs Attribute access:")
print("-" * 50)

setup_dict = "d = {'x': 1, 'y': 2, 'z': 3}"
setup_obj = """
class Point:
    def __init__(self):
        self.x, self.y, self.z = 1, 2, 3
p = Point()
"""

print("\nDict lookup:")
t1 = statistical_benchmark("v = d['x']", setup=setup_dict)

print("\nAttribute access:")
t2 = statistical_benchmark("v = p.x", setup=setup_obj)

print(f"\nDict is {t1/t2:.2f}x {'slower' if t1>t2 else 'faster'} than attribute access")
```

---

## 4. Algorithm Complexity (Big O) <a name="big-o"></a>

### ความซับซ้อนของ Algorithm

| Complexity | Name | Example |
|-----------|------|---------|
| O(1) | Constant | Dictionary lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Bubble sort |
| O(2ⁿ) | Exponential | Recursive fibonacci |
| O(n!) | Factorial | Permutations |

```python
# ตัวอย่าง 10: Big O examples ใน Python
import time
import random

def demonstrate_complexity():
    """แสดงความแตกต่างของ time complexity"""
    
    sizes = [100, 1000, 5000, 10000]
    
    print("Time Complexity Demonstration")
    print("=" * 60)
    
    for n in sizes:
        data = list(range(n))
        random.shuffle(data)
        
        # O(1) - Dictionary lookup
        d = {i: i for i in range(n)}
        start = time.perf_counter()
        _ = d[n // 2]
        t_o1 = (time.perf_counter() - start) * 1_000_000
        
        # O(log n) - Binary search (sorted data)
        sorted_data = list(range(n))
        start = time.perf_counter()
        lo, hi = 0, n - 1
        target = n // 2
        while lo <= hi:
            mid = (lo + hi) // 2
            if sorted_data[mid] == target:
                break
            elif sorted_data[mid] < target:
                lo = mid + 1
            else:
                hi = mid - 1
        t_ologn = (time.perf_counter() - start) * 1_000_000
        
        # O(n) - Linear search
        start = time.perf_counter()
        _ = target in data
        t_on = (time.perf_counter() - start) * 1_000_000
        
        # O(n²) - Bubble sort (small n only)
        if n <= 1000:
            test_data = data[:100]
            start = time.perf_counter()
            for i in range(len(test_data)):
                for j in range(len(test_data) - 1 - i):
                    if test_data[j] > test_data[j+1]:
                        test_data[j], test_data[j+1] = test_data[j+1], test_data[j]
            t_on2 = (time.perf_counter() - start) * 1_000_000
            n2_str = f"{t_on2:.2f}"
        else:
            n2_str = "too slow"
        
        print(f"\nn={n:6d}: O(1)={t_o1:.3f}μs, O(logn)={t_ologn:.3f}μs, "
              f"O(n)={t_on:.2f}μs, O(n²)={n2_str}μs")

demonstrate_complexity()
```

```python
# ตัวอย่าง 11: การเลือก Data Structure ที่เหมาะสม
import time
import random

def benchmark_data_structures():
    """เปรียบเทียบ list vs set vs dict สำหรับ contains check"""
    
    n = 100_000
    data = random.sample(range(n * 10), n)
    search_items = random.sample(range(n * 10), 1000)
    
    # สร้าง data structures
    data_list = data
    data_set = set(data)
    data_dict = {x: True for x in data}
    
    # Test list (O(n))
    start = time.perf_counter()
    found = sum(1 for item in search_items if item in data_list)
    t_list = time.perf_counter() - start
    
    # Test set (O(1) average)
    start = time.perf_counter()
    found = sum(1 for item in search_items if item in data_set)
    t_set = time.perf_counter() - start
    
    # Test dict (O(1) average)
    start = time.perf_counter()
    found = sum(1 for item in search_items if item in data_dict)
    t_dict = time.perf_counter() - start
    
    print(f"\n'contains' check for {len(search_items)} items in {n:,} element collection:")
    print(f"  list:  {t_list:.4f}s  ({t_list/t_set:.0f}x slower than set)")
    print(f"  set:   {t_set:.4f}s  (baseline)")
    print(f"  dict:  {t_dict:.4f}s  ({t_dict/t_set:.2f}x vs set)")
    
    print(f"\nMemory usage:")
    import sys
    print(f"  list: {sys.getsizeof(data_list):,} bytes")
    print(f"  set:  {sys.getsizeof(data_set):,} bytes")
    print(f"  dict: {sys.getsizeof(data_dict):,} bytes")

benchmark_data_structures()
```

---

## 5. Python Performance Tips <a name="performance-tips"></a>

```python
# ตัวอย่าง 12: Local variables vs Global variables
import timeit

# Python lookup: local > enclosing > global > built-in (LEGB)
# Local variables เร็วกว่า global เพราะ LOAD_FAST vs LOAD_GLOBAL

def global_loop():
    """ใช้ global variable ใน loop"""
    import math
    result = 0
    for i in range(1000):
        result += math.sqrt(i)  # แต่ละครั้งต้อง lookup 'math' และ 'sqrt'
    return result

def local_loop():
    """Cache ฟังก์ชันใน local variable"""
    from math import sqrt  # เก็บใน local
    result = 0
    for i in range(1000):
        result += sqrt(i)  # LOAD_FAST แทน LOAD_GLOBAL
    return result

def cached_loop():
    """Cache ทั้ง function ใน local variable"""
    import math
    sqrt = math.sqrt  # cache เข้า local
    result = 0
    for i in range(1000):
        result += sqrt(i)
    return result

t1 = timeit.timeit(global_loop, number=5000)
t2 = timeit.timeit(local_loop, number=5000)
t3 = timeit.timeit(cached_loop, number=5000)

print("Local vs Global Variable Performance:")
print(f"  global math.sqrt:   {t1:.3f}s")
print(f"  from math import:   {t2:.3f}s ({t1/t2:.2f}x faster)")
print(f"  cached sqrt = ...:  {t3:.3f}s ({t1/t3:.2f}x faster)")
```

```python
# ตัวอย่าง 13: map/filter vs comprehensions vs loops
import timeit

def compare_iteration_methods():
    """เปรียบเทียบวิธีการ iterate ต่างๆ"""
    
    n = 10000
    data = list(range(n))
    
    # filter + map (functional)
    stmt_map_filter = f"""
result = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, range({n}))))
"""
    
    # list comprehension
    stmt_comprehension = f"""
result = [x**2 for x in range({n}) if x % 2 == 0]
"""
    
    # for loop
    stmt_loop = f"""
result = []
for x in range({n}):
    if x % 2 == 0:
        result.append(x**2)
"""
    
    # generator expression
    stmt_gen = f"""
result = list(x**2 for x in range({n}) if x % 2 == 0)
"""
    
    repeats = 1000
    
    t1 = timeit.timeit(stmt_map_filter, number=repeats)
    t2 = timeit.timeit(stmt_comprehension, number=repeats)
    t3 = timeit.timeit(stmt_loop, number=repeats)
    t4 = timeit.timeit(stmt_gen, number=repeats)
    
    print("\nIteration Method Performance:")
    print(f"  map+filter:          {t1:.3f}s")
    print(f"  list comprehension:  {t2:.3f}s")
    print(f"  for loop:            {t3:.3f}s")
    print(f"  generator expr:      {t4:.3f}s")

compare_iteration_methods()
```

```python
# ตัวอย่าง 14: ใช้ built-in functions แทน manual loops
import timeit
import operator
from functools import reduce

def sum_manual(data):
    total = 0
    for x in data:
        total += x
    return total

def sum_builtin(data):
    return sum(data)

def max_manual(data):
    maximum = data[0]
    for x in data[1:]:
        if x > maximum:
            maximum = x
    return maximum

def max_builtin(data):
    return max(data)

# Test
data = list(range(100000))

t1 = timeit.timeit(lambda: sum_manual(data), number=100)
t2 = timeit.timeit(lambda: sum_builtin(data), number=100)
t3 = timeit.timeit(lambda: max_manual(data), number=100)
t4 = timeit.timeit(lambda: max_builtin(data), number=100)

print("\nBuilt-in vs Manual:")
print(f"  sum manual:   {t1:.3f}s")
print(f"  sum built-in: {t2:.3f}s ({t1/t2:.1f}x faster)")
print(f"  max manual:   {t3:.3f}s")
print(f"  max built-in: {t4:.3f}s ({t3/t4:.1f}x faster)")
```

```python
# ตัวอย่าง 15: ใช้ slots สำหรับ class ที่สร้าง instance จำนวนมาก
import sys
import timeit

class PointWithDict:
    """Class ปกติ - มี __dict__"""
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

class PointWithSlots:
    """Class ที่ใช้ __slots__ - ไม่มี __dict__"""
    __slots__ = ['x', 'y', 'z']
    
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

# เปรียบเทียบ memory
p1 = PointWithDict(1, 2, 3)
p2 = PointWithSlots(1, 2, 3)

print("\n__slots__ Memory Comparison:")
print(f"  PointWithDict:  {sys.getsizeof(p1.__dict__) + sys.getsizeof(p1)} bytes")
print(f"  PointWithSlots: {sys.getsizeof(p2)} bytes")

# เปรียบเทียบ speed
n = 1_000_000

t1 = timeit.timeit(
    "PointWithDict(1.0, 2.0, 3.0)",
    globals=globals(),
    number=n
)
t2 = timeit.timeit(
    "PointWithSlots(1.0, 2.0, 3.0)",
    globals=globals(),
    number=n
)

print(f"\nCreating {n:,} instances:")
print(f"  PointWithDict:  {t1:.3f}s")
print(f"  PointWithSlots: {t2:.3f}s ({t1/t2:.2f}x faster)")

# เปรียบเทียบ attribute access
points_dict = [PointWithDict(i, i, i) for i in range(10000)]
points_slots = [PointWithSlots(i, i, i) for i in range(10000)]

t3 = timeit.timeit(
    "sum(p.x + p.y + p.z for p in points_dict)",
    globals={'points_dict': points_dict},
    number=1000
)
t4 = timeit.timeit(
    "sum(p.x + p.y + p.z for p in points_slots)",
    globals={'points_slots': points_slots},
    number=1000
)

print(f"\nAttribute access (10,000 objects):")
print(f"  With dict:   {t3:.3f}s")
print(f"  With slots:  {t4:.3f}s ({t3/t4:.2f}x faster)")
```

---

## 6. List vs Generator Performance <a name="list-vs-generator"></a>

```python
# ตัวอย่าง 16: List comprehension vs Generator expressions
import sys
import timeit

# Memory comparison
list_comp = [x**2 for x in range(1_000_000)]
gen_exp = (x**2 for x in range(1_000_000))

print("\nList vs Generator Memory:")
print(f"  List comprehension: {sys.getsizeof(list_comp):,} bytes")
print(f"  Generator:          {sys.getsizeof(gen_exp):,} bytes")
print(f"  Ratio: {sys.getsizeof(list_comp) / sys.getsizeof(gen_exp):.0f}x more memory for list")

# Speed comparison สำหรับ use cases ต่างๆ
def sum_with_list(n):
    return sum([x**2 for x in range(n)])

def sum_with_generator(n):
    return sum(x**2 for x in range(n))

def find_first_with_list(n, target=500000):
    return next(iter([x for x in range(n) if x**2 > target]))

def find_first_with_generator(n, target=500000):
    return next(x for x in range(n) if x**2 > target)

n = 100_000

t1 = timeit.timeit(lambda: sum_with_list(n), number=100)
t2 = timeit.timeit(lambda: sum_with_generator(n), number=100)
t3 = timeit.timeit(lambda: find_first_with_list(n), number=100)
t4 = timeit.timeit(lambda: find_first_with_generator(n), number=100)

print(f"\nSum all elements (n={n:,}):")
print(f"  List:      {t1:.3f}s")
print(f"  Generator: {t2:.3f}s")

print(f"\nFind first matching (early termination):")
print(f"  List:      {t3:.3f}s (builds entire list first)")
print(f"  Generator: {t4:.3f}s ({t3/t4:.0f}x faster - lazy evaluation!)")
```

```python
# ตัวอย่าง 17: Generator pipeline สำหรับ data processing
import time

def read_data(n=1_000_000):
    """Simulate reading data"""
    for i in range(n):
        yield {'id': i, 'value': i * 2.5, 'category': i % 5}

def filter_category(data, category):
    """Filter by category"""
    for item in data:
        if item['category'] == category:
            yield item

def transform_value(data):
    """Transform values"""
    for item in data:
        yield {**item, 'transformed': item['value'] ** 0.5}

def aggregate(data, limit=None):
    """Aggregate results"""
    total = 0
    count = 0
    for item in data:
        total += item['transformed']
        count += 1
        if limit and count >= limit:
            break
    return total, count

# Generator pipeline - ประมวลผลแบบ lazy
print("\nGenerator Pipeline:")
start = time.perf_counter()

pipeline = read_data(1_000_000)
filtered = filter_category(pipeline, category=2)
transformed = transform_value(filtered)
total, count = aggregate(transformed)

elapsed = time.perf_counter() - start
print(f"  Processed {count:,} items out of 1,000,000")
print(f"  Total: {total:.2f}")
print(f"  Time: {elapsed:.3f}s")
print(f"  Memory: O(1) - เก็บแค่ 1 item ต่อครั้ง!")
```

---

## 7. NumPy Vectorization <a name="numpy-vectorization"></a>

```python
# ตัวอย่าง 18: NumPy vs Pure Python
import timeit

def numpy_vs_python():
    """เปรียบเทียบ NumPy กับ Pure Python"""
    try:
        import numpy as np
    except ImportError:
        print("NumPy not installed. Run: pip install numpy")
        return
    
    n = 1_000_000
    data_list = list(range(n))
    data_np = np.arange(n, dtype=np.float64)
    
    # Test 1: Sum
    t1 = timeit.timeit(lambda: sum(data_list), number=10)
    t2 = timeit.timeit(lambda: np.sum(data_np), number=10)
    print(f"Sum {n:,} elements:")
    print(f"  Python sum:  {t1:.3f}s")
    print(f"  NumPy sum:   {t2:.3f}s ({t1/t2:.0f}x faster)")
    
    # Test 2: Element-wise square root
    t3 = timeit.timeit(lambda: [x**0.5 for x in data_list], number=10)
    t4 = timeit.timeit(lambda: np.sqrt(data_np), number=10)
    print(f"\nSqrt {n:,} elements:")
    print(f"  Python:  {t3:.3f}s")
    print(f"  NumPy:   {t4:.3f}s ({t3/t4:.0f}x faster)")
    
    # Test 3: Dot product
    a_list = list(range(1000))
    b_list = list(range(1000))
    a_np = np.array(a_list, dtype=np.float64)
    b_np = np.array(b_list, dtype=np.float64)
    
    t5 = timeit.timeit(lambda: sum(x*y for x, y in zip(a_list, b_list)), number=10000)
    t6 = timeit.timeit(lambda: np.dot(a_np, b_np), number=10000)
    print(f"\nDot product (n=1000):")
    print(f"  Python:  {t5:.3f}s")
    print(f"  NumPy:   {t6:.3f}s ({t5/t6:.0f}x faster)")

numpy_vs_python()
```

```python
# ตัวอย่าง 19: NumPy broadcasting และ vectorized operations
try:
    import numpy as np
    import timeit

    def vectorized_operations():
        """NumPy vectorization examples"""
        
        # สร้าง arrays
        x = np.linspace(0, 2 * np.pi, 1_000_000)
        
        # Vectorized math (ทำงานกับทุก element พร้อมกัน)
        sin_x = np.sin(x)
        cos_x = np.cos(x)
        
        # Broadcasting - คำนวณ outer product โดยไม่ต้อง loop
        a = np.arange(100).reshape(100, 1)  # column vector
        b = np.arange(100).reshape(1, 100)  # row vector
        matrix = a + b  # 100x100 matrix ด้วย broadcasting
        
        print("NumPy Vectorization:")
        print(f"  sin+cos of 1M elements: done")
        print(f"  Broadcasting result shape: {matrix.shape}")
        
        # Conditional operations (vectorized if)
        data = np.random.randn(1_000_000)
        
        # Python way
        t1 = timeit.timeit(
            lambda: [abs(x) if x < 0 else x for x in data.tolist()],
            number=5
        )
        
        # NumPy way
        t2 = timeit.timeit(
            lambda: np.where(data < 0, -data, data),
            number=5
        )
        
        print(f"\nConditional operation on 1M elements:")
        print(f"  Python loop: {t1:.3f}s")
        print(f"  np.where:    {t2:.3f}s ({t1/t2:.0f}x faster)")
        
        # Boolean indexing
        t3 = timeit.timeit(
            lambda: [x for x in data.tolist() if x > 0],
            number=5
        )
        t4 = timeit.timeit(
            lambda: data[data > 0],
            number=5
        )
        
        print(f"\nBoolean filtering:")
        print(f"  Python list comp: {t3:.3f}s")
        print(f"  NumPy boolean:    {t4:.3f}s ({t3/t4:.0f}x faster)")
    
    vectorized_operations()
    
except ImportError:
    print("NumPy not available")
```

---

## 8. Cython เบื้องต้น <a name="cython"></a>

Cython คือ superset ของ Python ที่ compile เป็น C เพื่อเพิ่มความเร็ว

```python
# ตัวอย่าง 20: Cython setup และ usage
# ไฟล์: fast_math.pyx (Cython source)
"""
# fast_math.pyx
# cython: language_level=3

def cy_sum(list data):
    cdef double total = 0.0
    cdef int i
    cdef int n = len(data)
    for i in range(n):
        total += data[i]
    return total

def cy_fibonacci(int n):
    cdef int a = 0, b = 1, i
    for i in range(n):
        a, b = b, a + b
    return a

# ใช้ typed memoryview สำหรับ NumPy arrays
import numpy as np
cimport numpy as np

def cy_dot_product(
    np.ndarray[np.float64_t, ndim=1] a,
    np.ndarray[np.float64_t, ndim=1] b
):
    cdef int n = len(a)
    cdef double result = 0.0
    cdef int i
    for i in range(n):
        result += a[i] * b[i]
    return result
"""

# ไฟล์: setup.py
"""
from setuptools import setup
from Cython.Build import cythonize
import numpy as np

setup(
    ext_modules=cythonize("fast_math.pyx"),
    include_dirs=[np.get_include()]
)
"""

# คำสั่ง compile:
# python setup.py build_ext --inplace

# การใช้งาน:
# import fast_math
# result = fast_math.cy_fibonacci(1000)

print("""
Cython Workflow:
1. เขียนไฟล์ .pyx พร้อม type annotations
2. สร้าง setup.py
3. compile: python setup.py build_ext --inplace
4. import และใช้งานเหมือน Python module ทั่วไป

Performance gain ทั่วไป:
- Pure Python loops: 10-100x faster
- NumPy operations: 2-10x faster  
- Parallel computation (prange): additional speedup
""")

# Pure Python fibonacci สำหรับเปรียบเทียบ
def py_fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a

import timeit
t = timeit.timeit(lambda: py_fibonacci(1000), number=100000)
print(f"Pure Python fibonacci(1000): {t:.3f}s")
print("Cython version would be ~50-100x faster")
```

---

## 9. ctypes และ cffi <a name="ctypes-cffi"></a>

```python
# ตัวอย่าง 21: ctypes - เรียก C functions โดยตรง
import ctypes
import ctypes.util
import timeit

def ctypes_examples():
    """ตัวอย่างการใช้ ctypes"""
    
    # หา C math library
    libm_name = ctypes.util.find_library('m')
    if not libm_name:
        print("C math library not found")
        return
    
    try:
        libm = ctypes.CDLL(libm_name)
    except Exception as e:
        print(f"Cannot load math library: {e}")
        return
    
    # กำหนด return type และ argument types
    libm.sqrt.restype = ctypes.c_double
    libm.sqrt.argtypes = [ctypes.c_double]
    
    libm.sin.restype = ctypes.c_double
    libm.sin.argtypes = [ctypes.c_double]
    
    # เรียกใช้ C functions
    result_sqrt = libm.sqrt(2.0)
    result_sin = libm.sin(3.14159 / 2)
    
    print(f"C sqrt(2.0) = {result_sqrt:.10f}")
    print(f"C sin(π/2)  = {result_sin:.10f}")
    
    # เปรียบเทียบ speed
    import math
    
    t1 = timeit.timeit(lambda: math.sqrt(2.0), number=1_000_000)
    t2 = timeit.timeit(lambda: libm.sqrt(2.0), number=1_000_000)
    
    print(f"\nPerformance:")
    print(f"  math.sqrt:  {t1:.3f}s")
    print(f"  ctypes.sqrt:{t2:.3f}s")

ctypes_examples()
```

```python
# ตัวอย่าง 22: ctypes - สร้าง shared library จาก C
"""
# fast_ops.c
#include <math.h>
#include <stdlib.h>

// คำนวณ sum ของ array
double array_sum(double* data, int n) {
    double total = 0.0;
    for (int i = 0; i < n; i++) {
        total += data[i];
    }
    return total;
}

// คำนวณ moving average
void moving_average(double* data, double* result, int n, int window) {
    double sum = 0.0;
    for (int i = 0; i < window; i++) sum += data[i];
    result[0] = sum / window;
    for (int i = window; i < n; i++) {
        sum += data[i] - data[i - window];
        result[i - window + 1] = sum / window;
    }
}

// compile: gcc -shared -fPIC -O3 -o fast_ops.so fast_ops.c -lm
"""

# การเรียกใช้งาน
import ctypes
import ctypes.util

# ถ้ามี library ให้ load ได้:
# lib = ctypes.CDLL('./fast_ops.so')
# lib.array_sum.restype = ctypes.c_double
# lib.array_sum.argtypes = [ctypes.POINTER(ctypes.c_double), ctypes.c_int]

# สร้าง array สำหรับส่งให้ C
# n = 1000000
# arr = (ctypes.c_double * n)(*range(n))
# result = lib.array_sum(arr, n)

print("""
ctypes Workflow:
1. เขียน C code
2. Compile เป็น shared library (.so หรือ .dll)
3. Load ด้วย ctypes.CDLL()
4. กำหนด argtypes และ restype
5. เรียกใช้เหมือน Python function

cffi ทางเลือก (pip install cffi):
- API ที่ clean กว่า
- รองรับ inline C definitions
- สามารถ compile จาก Python
""")
```

---

## 10. Numba JIT Compilation <a name="numba"></a>

```python
# ตัวอย่าง 23: Numba - Python JIT compiler
# pip install numba

def numba_examples():
    try:
        from numba import jit, njit, prange
        import numpy as np
        import timeit
        
        # ฟังก์ชัน Python ปกติ
        def py_sum_squares(n):
            total = 0.0
            for i in range(n):
                total += i * i
            return total
        
        # ฟังก์ชัน Numba JIT compiled
        @jit(nopython=True)
        def nb_sum_squares(n):
            total = 0.0
            for i in range(n):
                total += i * i
            return total
        
        # Parallel version
        @njit(parallel=True)
        def nb_parallel_sum(arr):
            total = 0.0
            for i in prange(len(arr)):
                total += arr[i] ** 2
            return total
        
        n = 1_000_000
        
        # Warm up Numba (first call compiles)
        nb_sum_squares(100)
        
        t1 = timeit.timeit(lambda: py_sum_squares(n), number=10)
        t2 = timeit.timeit(lambda: nb_sum_squares(n), number=10)
        
        print("Numba JIT Performance:")
        print(f"  Python:  {t1:.3f}s")
        print(f"  Numba:   {t2:.3f}s ({t1/t2:.0f}x faster)")
        
        # NumPy operations
        arr = np.random.randn(n)
        
        t3 = timeit.timeit(lambda: np.sum(arr**2), number=100)
        nb_parallel_sum(arr[:100])  # warm up
        t4 = timeit.timeit(lambda: nb_parallel_sum(arr), number=100)
        
        print(f"\nArray sum of squares ({n:,} elements):")
        print(f"  NumPy:           {t3:.3f}s")
        print(f"  Numba parallel:  {t4:.3f}s")
        
    except ImportError:
        print("Numba not installed. Run: pip install numba")
        print("Showing conceptual example instead...")
        
        print("""
Numba Decorators:
@jit(nopython=True)  - compile to machine code, no Python objects
@njit               - same as nopython=True, shorter syntax
@njit(parallel=True) - enable parallel execution with prange
@vectorize         - create ufunc for NumPy arrays
@cuda.jit          - compile for NVIDIA GPU

Best use cases:
- Numerical loops that can't be vectorized with NumPy
- Complex algorithms with many branches
- Performance-critical inner loops
""")

numba_examples()
```

---

## 11. PyPy <a name="pypy"></a>

```python
# ตัวอย่าง 24: PyPy - Alternative Python Implementation
"""
PyPy คือ Python implementation ที่มี JIT compiler ในตัว
รันได้เร็วกว่า CPython 4-10x สำหรับ pure Python code

การติดตั้ง:
1. Download จาก https://www.pypy.org/download.html
2. หรือใช้ conda: conda install -c conda-forge pypy3.9

Code ที่ได้ประโยชน์จาก PyPy:
- Pure Python loops
- Recursive algorithms
- String processing
- Object-oriented code

Code ที่ PyPy ไม่ได้เปรียบ:
- NumPy operations (NumPy ถูก optimize สำหรับ CPython แล้ว)
- C extension calls
- I/O bound tasks
"""

# ตัวอย่างที่ PyPy เก่งกว่า CPython มาก
def pi_approximation(n):
    """คำนวณ π ด้วย Leibniz formula"""
    pi = 0.0
    for i in range(n):
        pi += ((-1) ** i) / (2 * i + 1)
    return pi * 4

def prime_sieve(n):
    """Sieve of Eratosthenes"""
    sieve = [True] * (n + 1)
    sieve[0] = sieve[1] = False
    
    for i in range(2, int(n**0.5) + 1):
        if sieve[i]:
            for j in range(i*i, n + 1, i):
                sieve[j] = False
    
    return [i for i, is_prime in enumerate(sieve) if is_prime]

import timeit

t1 = timeit.timeit(lambda: pi_approximation(1_000_000), number=5)
t2 = timeit.timeit(lambda: prime_sieve(100_000), number=50)

print("CPython Performance (PyPy would be ~5-10x faster):")
print(f"  π approximation (1M iter): {t1:.3f}s")
print(f"  Prime sieve (100K):        {t2:.3f}s")
print("\nTo run with PyPy: pypy3 your_script.py")
```

---

## 12. Memory Optimization <a name="memory-optimization"></a>

```python
# ตัวอย่าง 25: Memory optimization strategies
import sys
import gc

def memory_optimization_examples():
    """เทคนิคการลดการใช้ memory"""
    
    # 1. ใช้ generators แทน lists
    def large_data_generator(n):
        for i in range(n):
            yield i * 2
    
    # ไม่ดี: สร้าง list ทั้งหมดในครั้งเดียว
    bad = [i * 2 for i in range(1_000_000)]
    size_list = sys.getsizeof(bad)
    del bad
    
    # ดี: ใช้ generator
    good = large_data_generator(1_000_000)
    size_gen = sys.getsizeof(good)
    del good
    
    print("Memory: List vs Generator:")
    print(f"  List (1M ints):  {size_list:,} bytes")
    print(f"  Generator:       {size_gen:,} bytes")
    
    # 2. ใช้ array แทน list สำหรับ numeric data
    import array
    
    data_list = list(range(1_000_000))
    data_array = array.array('i', range(1_000_000))  # int32
    
    print(f"\nList vs array.array (1M integers):")
    print(f"  list:  {sys.getsizeof(data_list):,} bytes")
    print(f"  array: {sys.getsizeof(data_array):,} bytes")
    
    # 3. ใช้ __slots__
    class BigClass:
        def __init__(self, a, b, c, d, e):
            self.a, self.b, self.c, self.d, self.e = a, b, c, d, e
    
    class SlottedClass:
        __slots__ = ['a', 'b', 'c', 'd', 'e']
        def __init__(self, a, b, c, d, e):
            self.a, self.b, self.c, self.d, self.e = a, b, c, d, e
    
    big = BigClass(1, 2, 3, 4, 5)
    slotted = SlottedClass(1, 2, 3, 4, 5)
    
    print(f"\n__slots__ memory:")
    print(f"  Regular class:  {sys.getsizeof(big) + sys.getsizeof(big.__dict__)} bytes")
    print(f"  Slotted class:  {sys.getsizeof(slotted)} bytes")
    
    # 4. ใช้ namedtuple หรือ dataclass แทน dict
    from collections import namedtuple
    from dataclasses import dataclass
    
    # dict
    point_dict = {'x': 1.0, 'y': 2.0, 'z': 3.0}
    
    # namedtuple
    Point = namedtuple('Point', ['x', 'y', 'z'])
    point_nt = Point(1.0, 2.0, 3.0)
    
    print(f"\ndict vs namedtuple:")
    print(f"  dict:       {sys.getsizeof(point_dict)} bytes")
    print(f"  namedtuple: {sys.getsizeof(point_nt)} bytes")

memory_optimization_examples()
```

```python
# ตัวอย่าง 26: Garbage Collection tuning
import gc
import timeit

def gc_optimization():
    """การ tune Garbage Collector"""
    
    print("\nGarbage Collector Settings:")
    print(f"  Enabled: {gc.isenabled()}")
    print(f"  Thresholds: {gc.get_thresholds()}")
    print(f"  Stats: {gc.get_stats()}")
    
    # ปิด GC สำหรับ compute-intensive sections
    class NoGC:
        """Context manager ปิด GC ชั่วคราว"""
        def __enter__(self):
            gc.disable()
            return self
        
        def __exit__(self, *args):
            gc.enable()
            gc.collect()  # clean up manually
    
    # Benchmark
    def create_objects():
        return [{'id': i, 'data': [j for j in range(10)]} 
                for i in range(10000)]
    
    t1 = timeit.timeit(create_objects, number=50)
    
    with NoGC():
        t2 = timeit.timeit(create_objects, number=50)
    
    print(f"\nObject creation benchmark:")
    print(f"  With GC:     {t1:.3f}s")
    print(f"  Without GC:  {t2:.3f}s ({(t1-t2)/t1*100:.1f}% faster)")
    
    # เก็บ memory ด้วย weak references
    import weakref
    
    class ExpensiveObject:
        def __init__(self, id):
            self.id = id
            self.data = list(range(1000))  # simulate large data
    
    # Cache ด้วย weak references
    cache = {}
    
    def get_object(id):
        if id in cache:
            obj = cache[id]()  # dereference weak ref
            if obj is not None:
                return obj
        
        obj = ExpensiveObject(id)
        cache[id] = weakref.ref(obj)  # store weak reference
        return obj
    
    obj1 = get_object(1)
    obj2 = get_object(1)  # จาก cache
    print(f"\nWeak reference cache: same object? {obj1 is obj2}")

gc_optimization()
```

---

## 13. String Formatting Performance <a name="string-formatting"></a>

```python
# ตัวอย่าง 27: String formatting methods comparison
import timeit

def benchmark_string_formatting():
    """เปรียบเทียบวิธีการ format string"""
    
    name = "Alice"
    age = 30
    score = 98.5
    
    n = 500000
    
    # วิธีที่ 1: % formatting (เก่า)
    t1 = timeit.timeit(
        "s = 'Name: %s, Age: %d, Score: %.1f' % (name, age, score)",
        globals={'name': name, 'age': age, 'score': score},
        number=n
    )
    
    # วิธีที่ 2: .format()
    t2 = timeit.timeit(
        "s = 'Name: {}, Age: {}, Score: {:.1f}'.format(name, age, score)",
        globals={'name': name, 'age': age, 'score': score},
        number=n
    )
    
    # วิธีที่ 3: f-string (Python 3.6+)
    t3 = timeit.timeit(
        "s = f'Name: {name}, Age: {age}, Score: {score:.1f}'",
        globals={'name': name, 'age': age, 'score': score},
        number=n
    )
    
    # วิธีที่ 4: Template string
    t4 = timeit.timeit(
        "s = Template('Name: $name, Age: $age').substitute(name=name, age=age)",
        setup="from string import Template",
        globals={'name': name, 'age': age},
        number=n
    )
    
    print("String Formatting Performance:")
    print(f"  % formatting:  {t1:.3f}s")
    print(f"  .format():     {t2:.3f}s")
    print(f"  f-string:      {t3:.3f}s (fastest)")
    print(f"  Template:      {t4:.3f}s (slowest)")
    
    print(f"\nf-string is {t2/t3:.2f}x faster than .format()")
    print(f"f-string is {t1/t3:.2f}x faster than % formatting")

benchmark_string_formatting()
```

---

## 14. Dictionary vs if-elif Chains <a name="dict-vs-ifelif"></a>

```python
# ตัวอย่าง 28: Dictionary dispatch vs if-elif
import timeit
import random

def benchmark_dispatch():
    """เปรียบเทียบ dict dispatch กับ if-elif"""
    
    # ฟังก์ชัน handler
    def handle_create(): return "created"
    def handle_read(): return "read"
    def handle_update(): return "updated"
    def handle_delete(): return "deleted"
    def handle_list(): return "listed"
    def handle_search(): return "searched"
    def handle_export(): return "exported"
    def handle_import(): return "imported"
    
    commands = ['create', 'read', 'update', 'delete', 
                'list', 'search', 'export', 'import']
    
    # วิธีที่ 1: if-elif chain
    def process_if_elif(command):
        if command == 'create':
            return handle_create()
        elif command == 'read':
            return handle_read()
        elif command == 'update':
            return handle_update()
        elif command == 'delete':
            return handle_delete()
        elif command == 'list':
            return handle_list()
        elif command == 'search':
            return handle_search()
        elif command == 'export':
            return handle_export()
        elif command == 'import':
            return handle_import()
        else:
            raise ValueError(f"Unknown command: {command}")
    
    # วิธีที่ 2: dict dispatch
    dispatch_table = {
        'create': handle_create,
        'read': handle_read,
        'update': handle_update,
        'delete': handle_delete,
        'list': handle_list,
        'search': handle_search,
        'export': handle_export,
        'import': handle_import,
    }
    
    def process_dict(command):
        handler = dispatch_table.get(command)
        if handler is None:
            raise ValueError(f"Unknown command: {command}")
        return handler()
    
    # Test data - mix of commands (worst case: 'import' is last in if-elif)
    test_commands = random.choices(commands, k=100000)
    
    t1 = timeit.timeit(
        lambda: [process_if_elif(cmd) for cmd in test_commands],
        number=10
    )
    t2 = timeit.timeit(
        lambda: [process_dict(cmd) for cmd in test_commands],
        number=10
    )
    
    print("Dict dispatch vs if-elif:")
    print(f"  if-elif: {t1:.3f}s")
    print(f"  dict:    {t2:.3f}s ({t1/t2:.2f}x faster)")
    print("\nDict dispatch advantages:")
    print("  - O(1) lookup vs O(n) if-elif scan")
    print("  - Easier to extend (add new handlers)")
    print("  - Cleaner code structure")

benchmark_dispatch()
```

---

## 15. Database Query Optimization <a name="database-optimization"></a>

```python
# ตัวอย่าง 29: SQLite query optimization
import sqlite3
import time
import random
import string

def create_test_db():
    """สร้าง test database"""
    conn = sqlite3.connect(':memory:')
    cursor = conn.cursor()
    
    # สร้าง table
    cursor.execute('''
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL,
            age INTEGER,
            city TEXT,
            score REAL
        )
    ''')
    
    # Insert test data
    cities = ['Bangkok', 'Chiang Mai', 'Phuket', 'Pattaya', 'Khon Kaen']
    users = []
    for i in range(100000):
        name = f"User{i}"
        email = f"user{i}@example.com"
        age = random.randint(18, 80)
        city = random.choice(cities)
        score = round(random.uniform(0, 100), 2)
        users.append((name, email, age, city, score))
    
    cursor.executemany(
        'INSERT INTO users (name, email, age, city, score) VALUES (?,?,?,?,?)',
        users
    )
    conn.commit()
    return conn

conn = create_test_db()

# Test 1: Query without index
start = time.perf_counter()
for _ in range(100):
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE age = 25")
    result = cursor.fetchall()
t_no_index = time.perf_counter() - start

# สร้าง index
conn.execute("CREATE INDEX idx_age ON users(age)")
conn.execute("CREATE INDEX idx_city ON users(city)")
conn.execute("CREATE INDEX idx_city_age ON users(city, age)")

# Test 2: Query with index
start = time.perf_counter()
for _ in range(100):
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE age = 25")
    result = cursor.fetchall()
t_with_index = time.perf_counter() - start

print("Database Query Optimization:")
print(f"  Without index: {t_no_index:.3f}s")
print(f"  With index:    {t_with_index:.3f}s ({t_no_index/t_with_index:.1f}x faster)")

# Test 3: N+1 problem vs JOIN
# N+1 way (slow)
conn.execute('''
    CREATE TABLE orders (
        id INTEGER PRIMARY KEY,
        user_id INTEGER,
        amount REAL,
        FOREIGN KEY (user_id) REFERENCES users(id)
    )
''')

orders = [(random.randint(1, 10000), round(random.uniform(10, 1000), 2)) 
          for _ in range(50000)]
conn.executemany('INSERT INTO orders (user_id, amount) VALUES (?,?)', orders)
conn.commit()

user_ids = [random.randint(1, 10000) for _ in range(100)]

# N+1 query (ไม่ดี)
start = time.perf_counter()
results = []
for user_id in user_ids:
    user = conn.execute("SELECT * FROM users WHERE id=?", (user_id,)).fetchone()
    user_orders = conn.execute("SELECT * FROM orders WHERE user_id=?", (user_id,)).fetchall()
    results.append((user, user_orders))
t_n_plus_1 = time.perf_counter() - start

# JOIN query (ดี)
start = time.perf_counter()
for user_id in user_ids:
    result = conn.execute('''
        SELECT u.*, o.amount 
        FROM users u 
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE u.id = ?
    ''', (user_id,)).fetchall()
t_join = time.perf_counter() - start

print(f"\nN+1 vs JOIN (100 users with orders):")
print(f"  N+1 queries: {t_n_plus_1:.3f}s ({len(user_ids)*2} queries)")
print(f"  JOIN:        {t_join:.3f}s ({len(user_ids)} queries)")
```

```python
# ตัวอย่าง 30: Connection pooling และ batch operations
import sqlite3
import time

def batch_operations_demo():
    """แสดงประโยชน์ของ batch operations"""
    
    conn = sqlite3.connect(':memory:')
    conn.execute('CREATE TABLE items (id INTEGER, value REAL)')
    
    n = 10000
    data = [(i, i * 1.5) for i in range(n)]
    
    # วิธีที่ 1: Insert ทีละ record
    start = time.perf_counter()
    for item in data:
        conn.execute('INSERT INTO items VALUES (?,?)', item)
    conn.commit()
    t_single = time.perf_counter() - start
    
    conn.execute('DELETE FROM items')
    conn.commit()
    
    # วิธีที่ 2: executemany (batch insert)
    start = time.perf_counter()
    conn.executemany('INSERT INTO items VALUES (?,?)', data)
    conn.commit()
    t_batch = time.perf_counter() - start
    
    conn.execute('DELETE FROM items')
    conn.commit()
    
    # วิธีที่ 3: executemany ใน single transaction
    start = time.perf_counter()
    with conn:
        conn.executemany('INSERT INTO items VALUES (?,?)', data)
    t_transaction = time.perf_counter() - start
    
    print(f"\nBatch Insert ({n:,} records):")
    print(f"  Single inserts:    {t_single:.3f}s")
    print(f"  executemany:       {t_batch:.3f}s ({t_single/t_batch:.0f}x faster)")
    print(f"  with transaction:  {t_transaction:.3f}s ({t_single/t_transaction:.0f}x faster)")

batch_operations_demo()
```

---

## 16. API Response Time Optimization <a name="api-optimization"></a>

```python
# ตัวอย่าง 31: API caching strategies
import time
import functools
import hashlib
import json
from typing import Any, Dict, Optional

class APICache:
    """Simple in-memory API cache"""
    
    def __init__(self, ttl: float = 300):
        self.cache: Dict[str, tuple] = {}
        self.ttl = ttl
        self.hits = 0
        self.misses = 0
    
    def _make_key(self, func_name: str, args: tuple, kwargs: dict) -> str:
        key_data = json.dumps({
            'func': func_name,
            'args': str(args),
            'kwargs': str(sorted(kwargs.items()))
        }, sort_keys=True)
        return hashlib.md5(key_data.encode()).hexdigest()
    
    def get(self, key: str) -> Optional[Any]:
        if key in self.cache:
            value, timestamp = self.cache[key]
            if time.time() - timestamp < self.ttl:
                self.hits += 1
                return value
            else:
                del self.cache[key]
        self.misses += 1
        return None
    
    def set(self, key: str, value: Any):
        self.cache[key] = (value, time.time())
    
    def stats(self):
        total = self.hits + self.misses
        hit_rate = self.hits / total * 100 if total > 0 else 0
        return {
            'hits': self.hits,
            'misses': self.misses,
            'total': total,
            'hit_rate': f"{hit_rate:.1f}%",
            'cached_items': len(self.cache)
        }

# Global cache instance
api_cache = APICache(ttl=60)

def cached_api_call(func):
    """Decorator สำหรับ cache API calls"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        key = api_cache._make_key(func.__name__, args, kwargs)
        result = api_cache.get(key)
        if result is None:
            result = func(*args, **kwargs)
            api_cache.set(key, result)
        return result
    return wrapper

# Simulate expensive API call
@cached_api_call
def fetch_user_data(user_id: int) -> dict:
    """Simulate API call to user service"""
    time.sleep(0.01)  # simulate network latency
    return {
        'id': user_id,
        'name': f'User {user_id}',
        'score': user_id * 1.5
    }

# Test caching
print("\nAPI Caching Demo:")

# First calls (cache miss)
start = time.perf_counter()
for i in range(5):
    fetch_user_data(i)
t1 = time.perf_counter() - start
print(f"  5 unique calls (no cache): {t1:.3f}s")

# Repeated calls (cache hit)
start = time.perf_counter()
for i in range(5):
    fetch_user_data(i)
t2 = time.perf_counter() - start
print(f"  5 cached calls:            {t2:.5f}s ({t1/t2:.0f}x faster)")

print(f"\nCache stats: {api_cache.stats()}")
```

```python
# ตัวอย่าง 32: Async API calls with asyncio
import asyncio
import time

async def fetch_item(item_id: int, delay: float = 0.05) -> dict:
    """Simulate async API call"""
    await asyncio.sleep(delay)  # simulate I/O wait
    return {'id': item_id, 'data': f'item_{item_id}'}

async def fetch_sequential(ids):
    """Fetch items sequentially"""
    results = []
    for id in ids:
        result = await fetch_item(id)
        results.append(result)
    return results

async def fetch_concurrent(ids):
    """Fetch items concurrently"""
    tasks = [fetch_item(id) for id in ids]
    return await asyncio.gather(*tasks)

async def main():
    ids = list(range(20))
    
    # Sequential
    start = time.perf_counter()
    results_seq = await fetch_sequential(ids)
    t_seq = time.perf_counter() - start
    
    # Concurrent
    start = time.perf_counter()
    results_conc = await fetch_concurrent(ids)
    t_conc = time.perf_counter() - start
    
    print("\nAsync API Performance:")
    print(f"  Sequential ({len(ids)} calls, 50ms each): {t_seq:.3f}s")
    print(f"  Concurrent:                               {t_conc:.3f}s")
    print(f"  Speedup: {t_seq/t_conc:.1f}x")
    
    # Batch with semaphore (control concurrency)
    async def fetch_with_semaphore(ids, max_concurrent=5):
        sem = asyncio.Semaphore(max_concurrent)
        
        async def bounded_fetch(id):
            async with sem:
                return await fetch_item(id)
        
        tasks = [bounded_fetch(id) for id in ids]
        return await asyncio.gather(*tasks)
    
    start = time.perf_counter()
    results_batched = await fetch_with_semaphore(ids, max_concurrent=5)
    t_batch = time.perf_counter() - start
    print(f"  Concurrent (max=5):                       {t_batch:.3f}s")

asyncio.run(main())
```

```python
# ตัวอย่าง 33: Response compression และ serialization
import json
import gzip
import zlib
import sys
import timeit

def benchmark_serialization():
    """เปรียบเทียบ serialization formats"""
    
    # สร้าง test data
    data = {
        'users': [
            {'id': i, 'name': f'User{i}', 'email': f'user{i}@ex.com',
             'scores': [j * 1.5 for j in range(20)]}
            for i in range(1000)
        ]
    }
    
    # JSON
    json_str = json.dumps(data)
    
    # Compressed JSON
    json_compressed = gzip.compress(json_str.encode())
    
    try:
        import msgpack
        # MessagePack (pip install msgpack)
        mp_bytes = msgpack.packb(data)
        has_msgpack = True
    except ImportError:
        has_msgpack = False
    
    print("\nSerialization Comparison:")
    print(f"  JSON size:           {len(json_str):,} bytes")
    print(f"  gzip JSON size:      {len(json_compressed):,} bytes ({len(json_compressed)/len(json_str)*100:.0f}% of original)")
    
    if has_msgpack:
        print(f"  MessagePack size:    {len(mp_bytes):,} bytes ({len(mp_bytes)/len(json_str)*100:.0f}% of original)")
    
    # Speed comparison
    n = 1000
    t_json_enc = timeit.timeit(lambda: json.dumps(data), number=n)
    t_json_dec = timeit.timeit(lambda: json.loads(json_str), number=n)
    
    print(f"\nJSON serialize:   {t_json_enc:.3f}s")
    print(f"JSON deserialize: {t_json_dec:.3f}s")

benchmark_serialization()
```

---

## 17. Advanced Examples <a name="advanced"></a>

```python
# ตัวอย่าง 34: LRU Cache สำหรับ expensive computations
from functools import lru_cache
import timeit

@lru_cache(maxsize=128)
def expensive_computation(n):
    """Fibonacci ด้วย LRU cache"""
    if n < 2:
        return n
    return expensive_computation(n - 1) + expensive_computation(n - 2)

def no_cache_fibonacci(n):
    """Fibonacci ไม่มี cache"""
    if n < 2:
        return n
    return no_cache_fibonacci(n - 1) + no_cache_fibonacci(n - 2)

print("\nLRU Cache Performance:")

# First call (compute)
start = __import__('time').perf_counter()
result = expensive_computation(35)
t1 = __import__('time').perf_counter() - start
print(f"  fibonacci(35) first call: {t1:.3f}s")

# Subsequent calls (cached)
start = __import__('time').perf_counter()
for _ in range(10000):
    result = expensive_computation(35)
t2 = __import__('time').perf_counter() - start
print(f"  fibonacci(35) x10000 calls: {t2:.5f}s (cached!)")

print(f"\nCache info: {expensive_computation.cache_info()}")
```

```python
# ตัวอย่าง 35: Profiling-guided optimization workflow
import cProfile
import pstats
import io
import time

def unoptimized_code(data):
    """โค้ดที่ยังไม่ได้ optimize"""
    result = []
    for item in data:
        # ปัญหา: แปลง int เป็น string แล้วกลับ
        s = str(item)
        if len(s) > 2:
            val = int(s) * 2
        else:
            val = int(s) * 3
        result.append(val)
    
    # ปัญหา: sort ซ้ำ
    sorted_result = sorted(result)
    filtered = [x for x in sorted_result if x % 2 == 0]
    final = sorted(filtered, reverse=True)
    
    return final

def optimized_code(data):
    """โค้ดที่ optimize แล้ว"""
    result = []
    for item in data:
        # แก้: ไม่ต้องแปลงเป็น string
        if item > 99:
            val = item * 2
        else:
            val = item * 3
        result.append(val)
    
    # แก้: sort ครั้งเดียว, filter ก่อน
    filtered = [x for x in result if x % 2 == 0]
    return sorted(filtered, reverse=True)

def even_better_code(data):
    """โค้ดที่ optimize สุด"""
    # One-pass solution
    return sorted(
        (item * 2 if item > 99 else item * 3 
         for item in data 
         if (item * 2 if item > 99 else item * 3) % 2 == 0),
        reverse=True
    )

data = list(range(1000))

t1 = timeit.timeit(lambda: unoptimized_code(data), number=1000)
t2 = timeit.timeit(lambda: optimized_code(data), number=1000)
t3 = timeit.timeit(lambda: even_better_code(data), number=1000)

print("\nOptimization Progress:")
print(f"  Unoptimized: {t1:.3f}s (baseline)")
print(f"  Optimized:   {t2:.3f}s ({t1/t2:.1f}x faster)")
print(f"  Even better: {t3:.3f}s ({t1/t3:.1f}x faster)")
```

---

## 18. แบบฝึกหัด <a name="exercises"></a>

### แบบฝึกหัดที่ 1: Profile และ Optimize

**โจทย์:** Profile ฟังก์ชันต่อไปนี้และปรับปรุงให้เร็วขึ้นอย่างน้อย 3x

```python
def find_common_words(text1, text2, min_length=4):
    """หาคำที่เหมือนกันใน 2 texts"""
    words1 = []
    for word in text1.split():
        word = word.lower().strip('.,!?;:')
        if len(word) >= min_length:
            words1.append(word)
    
    words2 = []
    for word in text2.split():
        word = word.lower().strip('.,!?;:')
        if len(word) >= min_length:
            words2.append(word)
    
    common = []
    for word in words1:
        if word in words2 and word not in common:
            common.append(word)
    
    return sorted(common)

# Test
import random
import string

def generate_text(n_words=10000):
    words = ['the', 'quick', 'brown', 'fox', 'jumps', 'over', 'lazy', 'dog',
             'python', 'programming', 'performance', 'optimization', 'algorithm']
    return ' '.join(random.choices(words, k=n_words))

text1 = generate_text()
text2 = generate_text()
```

**เฉลย:**

```python
import re
import timeit

def find_common_words_slow(text1, text2, min_length=4):
    words1 = []
    for word in text1.split():
        word = word.lower().strip('.,!?;:')
        if len(word) >= min_length:
            words1.append(word)
    
    words2 = []
    for word in text2.split():
        word = word.lower().strip('.,!?;:')
        if len(word) >= min_length:
            words2.append(word)
    
    common = []
    for word in words1:
        if word in words2 and word not in common:
            common.append(word)
    
    return sorted(common)

def find_common_words_fast(text1, text2, min_length=4):
    # ใช้ set แทน list สำหรับ O(1) lookup
    pattern = re.compile(r'[.,!?;:]')
    
    def get_words(text):
        return {pattern.sub('', w.lower()) 
                for w in text.split() 
                if len(pattern.sub('', w.lower())) >= min_length}
    
    # Set intersection หาคำที่เหมือนกัน
    words1 = get_words(text1)
    words2 = get_words(text2)
    
    return sorted(words1 & words2)

import random
def generate_text(n_words=10000):
    words = ['the', 'quick', 'brown', 'fox', 'jumps', 'over', 'lazy', 'dog',
             'python', 'programming', 'performance', 'optimization', 'algorithm']
    return ' '.join(random.choices(words, k=n_words))

text1 = generate_text()
text2 = generate_text()

t1 = timeit.timeit(lambda: find_common_words_slow(text1, text2), number=50)
t2 = timeit.timeit(lambda: find_common_words_fast(text1, text2), number=50)

print(f"Slow: {t1:.3f}s")
print(f"Fast: {t2:.3f}s ({t1/t2:.1f}x faster)")
assert sorted(find_common_words_slow(text1, text2)) == find_common_words_fast(text1, text2)
print("Results match!")
```

### แบบฝึกหัดที่ 2: Memory-Efficient Data Processing

**โจทย์:** เขียนโปรแกรมที่อ่านไฟล์ CSV ขนาดใหญ่ (simulate ด้วย generator) และคำนวณสถิติโดยไม่โหลดข้อมูลทั้งหมดเข้า memory

```python
# เฉลย
import statistics
from typing import Iterator, Dict

def generate_large_csv(n_rows: int = 1_000_000) -> Iterator[dict]:
    """Simulate large CSV file"""
    import random
    categories = ['A', 'B', 'C', 'D']
    for i in range(n_rows):
        yield {
            'id': i,
            'category': random.choice(categories),
            'value': random.gauss(100, 15)
        }

def compute_stats_efficient(data_iter: Iterator[dict]) -> Dict:
    """คำนวณสถิติแบบ single-pass"""
    count = 0
    total = 0.0
    min_val = float('inf')
    max_val = float('-inf')
    category_counts = {}
    
    # Welford's online algorithm for variance
    M2 = 0.0
    
    for row in data_iter:
        count += 1
        value = row['value']
        cat = row['category']
        
        total += value
        min_val = min(min_val, value)
        max_val = max(max_val, value)
        
        # Category counting
        category_counts[cat] = category_counts.get(cat, 0) + 1
        
        # Online variance (Welford's method)
        delta = value - (total / count)
        M2 += delta * delta * (count - 1) / count
    
    variance = M2 / count if count > 1 else 0
    
    return {
        'count': count,
        'mean': total / count,
        'std': variance ** 0.5,
        'min': min_val,
        'max': max_val,
        'categories': category_counts
    }

import time
start = time.perf_counter()
stats = compute_stats_efficient(generate_large_csv(100_000))
elapsed = time.perf_counter() - start

print(f"\nStats computed in {elapsed:.3f}s:")
print(f"  Count: {stats['count']:,}")
print(f"  Mean:  {stats['mean']:.2f}")
print(f"  Std:   {stats['std']:.2f}")
print(f"  Min:   {stats['min']:.2f}")
print(f"  Max:   {stats['max']:.2f}")
print(f"  Categories: {stats['categories']}")
print("Memory used: O(1) - constant regardless of data size!")
```

### แบบฝึกหัดที่ 3: Caching Strategy

**โจทย์:** Implement multi-level cache (L1: in-memory, L2: disk) สำหรับ expensive API calls

```python
# เฉลย
import json
import os
import time
import hashlib
import functools
from pathlib import Path

class MultiLevelCache:
    """Multi-level cache: L1 (memory) + L2 (disk)"""
    
    def __init__(self, l1_size=100, l2_dir=None, ttl=3600):
        self.l1: dict = {}  # in-memory cache
        self.l1_order = []  # LRU order
        self.l1_size = l1_size
        self.l2_dir = Path(l2_dir or '/tmp/cache')
        self.l2_dir.mkdir(parents=True, exist_ok=True)
        self.ttl = ttl
        self.stats = {'l1_hits': 0, 'l2_hits': 0, 'misses': 0}
    
    def _key(self, func_name, args, kwargs):
        data = f"{func_name}:{args}:{sorted(kwargs.items())}"
        return hashlib.sha256(data.encode()).hexdigest()[:16]
    
    def get(self, key):
        # L1 lookup
        if key in self.l1:
            value, ts = self.l1[key]
            if time.time() - ts < self.ttl:
                self.l1_order.remove(key)
                self.l1_order.append(key)
                self.stats['l1_hits'] += 1
                return value
            else:
                del self.l1[key]
                self.l1_order.remove(key)
        
        # L2 lookup (disk)
        l2_path = self.l2_dir / f"{key}.json"
        if l2_path.exists():
            try:
                with open(l2_path) as f:
                    cached = json.load(f)
                if time.time() - cached['ts'] < self.ttl:
                    self._set_l1(key, cached['value'])
                    self.stats['l2_hits'] += 1
                    return cached['value']
            except Exception:
                l2_path.unlink(missing_ok=True)
        
        self.stats['misses'] += 1
        return None
    
    def _set_l1(self, key, value):
        if len(self.l1) >= self.l1_size:
            oldest = self.l1_order.pop(0)
            del self.l1[oldest]
        self.l1[key] = (value, time.time())
        self.l1_order.append(key)
    
    def set(self, key, value):
        self._set_l1(key, value)
        l2_path = self.l2_dir / f"{key}.json"
        try:
            with open(l2_path, 'w') as f:
                json.dump({'value': value, 'ts': time.time()}, f)
        except Exception:
            pass

cache = MultiLevelCache(l1_size=50, ttl=60)

def cached(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        key = cache._key(func.__name__, args, kwargs)
        result = cache.get(key)
        if result is None:
            result = func(*args, **kwargs)
            cache.set(key, result)
        return result
    return wrapper

@cached
def expensive_api_call(user_id):
    time.sleep(0.05)
    return {'id': user_id, 'name': f'User{user_id}'}

# Test
print("\nMulti-level Cache Demo:")
ids = list(range(10)) * 3

start = time.perf_counter()
for uid in ids:
    expensive_api_call(uid)
elapsed = time.perf_counter() - start

print(f"  {len(ids)} calls completed in {elapsed:.3f}s")
print(f"  Stats: {cache.stats}")
```

### แบบฝึกหัดที่ 4: NumPy Optimization

**โจทย์:** แปลง Pure Python image processing เป็น NumPy

```python
# เฉลย
import timeit
try:
    import numpy as np
    
    def blur_python(image, kernel_size=3):
        """Blur image ด้วย Pure Python"""
        h, w = len(image), len(image[0])
        k = kernel_size // 2
        result = [[0] * w for _ in range(h)]
        
        for i in range(k, h - k):
            for j in range(k, w - k):
                total = 0
                count = 0
                for di in range(-k, k + 1):
                    for dj in range(-k, k + 1):
                        total += image[i + di][j + dj]
                        count += 1
                result[i][j] = total // count
        
        return result
    
    def blur_numpy(image, kernel_size=3):
        """Blur image ด้วย NumPy"""
        k = kernel_size // 2
        result = np.zeros_like(image)
        
        for di in range(-k, k + 1):
            for dj in range(-k, k + 1):
                result[k:-k, k:-k] += np.roll(np.roll(image, di, 0), dj, 1)[k:-k, k:-k]
        
        result[k:-k, k:-k] //= (kernel_size ** 2)
        return result
    
    # Create test image
    size = 100
    image_list = [[i * j % 256 for j in range(size)] for i in range(size)]
    image_np = np.array(image_list, dtype=np.int32)
    
    t1 = timeit.timeit(lambda: blur_python(image_list), number=10)
    t2 = timeit.timeit(lambda: blur_numpy(image_np), number=100)
    
    print(f"\nImage blur ({size}x{size}):")
    print(f"  Pure Python: {t1:.3f}s")
    print(f"  NumPy:       {t2:.3f}s ({t1/t2:.1f}x faster)")
    
except ImportError:
    print("NumPy not available for exercise 4")
```

### แบบฝึกหัดที่ 5: Async Optimization

**โจทย์:** Optimize การดึงข้อมูลจากหลาย APIs พร้อมกัน

```python
# เฉลย
import asyncio
import time
import random

async def fetch_product(product_id: int) -> dict:
    await asyncio.sleep(random.uniform(0.05, 0.15))
    return {'id': product_id, 'name': f'Product {product_id}', 'price': product_id * 9.99}

async def fetch_inventory(product_id: int) -> dict:
    await asyncio.sleep(random.uniform(0.03, 0.08))
    return {'product_id': product_id, 'stock': random.randint(0, 100)}

async def fetch_reviews(product_id: int) -> list:
    await asyncio.sleep(random.uniform(0.04, 0.10))
    return [{'rating': random.randint(1, 5)} for _ in range(random.randint(1, 5))]

async def get_product_details_sequential(product_id: int) -> dict:
    product = await fetch_product(product_id)
    inventory = await fetch_inventory(product_id)
    reviews = await fetch_reviews(product_id)
    avg_rating = sum(r['rating'] for r in reviews) / len(reviews)
    return {**product, **inventory, 'avg_rating': avg_rating}

async def get_product_details_concurrent(product_id: int) -> dict:
    product, inventory, reviews = await asyncio.gather(
        fetch_product(product_id),
        fetch_inventory(product_id),
        fetch_reviews(product_id)
    )
    avg_rating = sum(r['rating'] for r in reviews) / len(reviews)
    return {**product, **inventory, 'avg_rating': avg_rating}

async def main():
    product_ids = list(range(10))
    
    # Sequential
    start = time.perf_counter()
    results = [await get_product_details_sequential(pid) for pid in product_ids]
    t_seq = time.perf_counter() - start
    
    # Concurrent per product
    start = time.perf_counter()
    results = [await get_product_details_concurrent(pid) for pid in product_ids]
    t_conc1 = time.perf_counter() - start
    
    # Fully concurrent (all products at once)
    start = time.perf_counter()
    results = await asyncio.gather(*[
        get_product_details_concurrent(pid) for pid in product_ids
    ])
    t_conc2 = time.perf_counter() - start
    
    print(f"\nAsync Optimization ({len(product_ids)} products):")
    print(f"  Fully sequential:           {t_seq:.3f}s")
    print(f"  Concurrent per product:     {t_conc1:.3f}s ({t_seq/t_conc1:.1f}x faster)")
    print(f"  All concurrent:             {t_conc2:.3f}s ({t_seq/t_conc2:.1f}x faster)")

asyncio.run(main())
```

### แบบฝึกหัดที่ 6: Database Optimization

**โจทย์:** Optimize slow database queries

```python
# เฉลย
import sqlite3
import time
import random

def setup_db():
    conn = sqlite3.connect(':memory:')
    conn.executescript('''
        CREATE TABLE products (
            id INTEGER PRIMARY KEY,
            name TEXT, category TEXT,
            price REAL, stock INTEGER
        );
        CREATE TABLE orders (
            id INTEGER PRIMARY KEY,
            product_id INTEGER, quantity INTEGER,
            total REAL, created_at TEXT
        );
    ''')
    
    products = [(f'Product{i}', random.choice(['A','B','C']), 
                 round(random.uniform(10,500),2), random.randint(0,100))
                for i in range(10000)]
    conn.executemany('INSERT INTO products (name,category,price,stock) VALUES(?,?,?,?)', products)
    
    orders = [(random.randint(1,10000), random.randint(1,10),
               round(random.uniform(10,5000),2), f'2024-0{random.randint(1,9)}-01')
              for _ in range(100000)]
    conn.executemany('INSERT INTO orders (product_id,quantity,total,created_at) VALUES(?,?,?,?)', orders)
    conn.commit()
    return conn

conn = setup_db()

# Slow query
start = time.perf_counter()
for _ in range(100):
    conn.execute('''
        SELECT p.category, AVG(o.total)
        FROM orders o JOIN products p ON o.product_id = p.id
        WHERE p.category = 'A'
        GROUP BY p.category
    ''').fetchall()
t_slow = time.perf_counter() - start

# Add index
conn.executescript('''
    CREATE INDEX idx_prod_cat ON products(category);
    CREATE INDEX idx_order_pid ON orders(product_id);
''')

start = time.perf_counter()
for _ in range(100):
    conn.execute('''
        SELECT p.category, AVG(o.total)
        FROM orders o JOIN products p ON o.product_id = p.id
        WHERE p.category = 'A'
        GROUP BY p.category
    ''').fetchall()
t_fast = time.perf_counter() - start

print(f"\nDatabase Optimization:")
print(f"  Without index: {t_slow:.3f}s")
print(f"  With index:    {t_fast:.3f}s ({t_slow/t_fast:.1f}x faster)")
```

### แบบฝึกหัดที่ 7: String Processing Optimization

**โจทย์:** Optimize การประมวลผล text จำนวนมาก

```python
# เฉลย
import re
import timeit

def process_text_slow(texts):
    results = []
    for text in texts:
        # ไม่ดี: compile regex ใหม่ทุกครั้ง
        text = re.sub(r'[^\w\s]', '', text)
        text = re.sub(r'\s+', ' ', text)
        text = text.lower().strip()
        words = text.split()
        results.append(words)
    return results

# Compile regex ครั้งเดียว
PUNCT_RE = re.compile(r'[^\w\s]')
SPACE_RE = re.compile(r'\s+')

def process_text_fast(texts):
    results = []
    for text in texts:
        text = PUNCT_RE.sub('', text)
        text = SPACE_RE.sub(' ', text).lower().strip()
        results.append(text.split())
    return results

texts = [f"Hello, World! This is text #{i}. Python is amazing!!!" for i in range(10000)]

t1 = timeit.timeit(lambda: process_text_slow(texts), number=10)
t2 = timeit.timeit(lambda: process_text_fast(texts), number=10)

print(f"\nText Processing:")
print(f"  Compile each time: {t1:.3f}s")
print(f"  Pre-compiled:      {t2:.3f}s ({t1/t2:.1f}x faster)")
```

### แบบฝึกหัดที่ 8: Comprehensive Optimization

**โจทย์:** วิเคราะห์และ optimize โปรแกรมต่อไปนี้ให้เร็วขึ้น 5x

```python
# โค้ดต้นฉบับ (slow)
def analyze_data_slow(records):
    unique_users = []
    for r in records:
        if r['user_id'] not in unique_users:
            unique_users.append(r['user_id'])
    
    user_totals = {}
    for user_id in unique_users:
        user_totals[user_id] = 0
        for r in records:
            if r['user_id'] == user_id:
                user_totals[user_id] += r['amount']
    
    result = []
    for user_id, total in user_totals.items():
        avg = total / sum(1 for r in records if r['user_id'] == user_id)
        result.append({'user_id': user_id, 'total': total, 'avg': avg})
    
    return sorted(result, key=lambda x: x['total'], reverse=True)

# เฉลย (fast)
from collections import defaultdict
import timeit

def analyze_data_fast(records):
    # Single pass O(n)
    user_data = defaultdict(lambda: {'total': 0.0, 'count': 0})
    
    for r in records:
        uid = r['user_id']
        user_data[uid]['total'] += r['amount']
        user_data[uid]['count'] += 1
    
    result = [
        {'user_id': uid, 'total': d['total'], 'avg': d['total'] / d['count']}
        for uid, d in user_data.items()
    ]
    
    return sorted(result, key=lambda x: x['total'], reverse=True)

# Test data
import random
records = [{'user_id': random.randint(1, 100), 'amount': random.uniform(10, 1000)}
           for _ in range(10000)]

t1 = timeit.timeit(lambda: analyze_data_slow(records), number=10)
t2 = timeit.timeit(lambda: analyze_data_fast(records), number=100)

print(f"\nComprehensive Optimization:")
print(f"  Slow (O(n²) loops): {t1:.3f}s")
print(f"  Fast (O(n) single pass): {t2:.3f}s ({t1/t2:.0f}x faster!)")

# Verify results match
r1 = analyze_data_slow(records)
r2 = analyze_data_fast(records)
assert len(r1) == len(r2)
assert abs(r1[0]['total'] - r2[0]['total']) < 0.001
print("Results verified correct!")
```

---

## สรุป

| หัวข้อ | เครื่องมือ | ประโยชน์ |
|--------|-----------|---------|
| CPU Profiling | cProfile, line_profiler | หา CPU bottleneck |
| Memory Profiling | memory_profiler, tracemalloc | หา memory leak |
| Benchmarking | timeit | วัดเวลาอย่างแม่นยำ |
| Vectorization | NumPy | 10-100x สำหรับ numeric |
| JIT Compilation | Numba, PyPy | 5-100x สำหรับ loops |
| C Extensions | Cython, ctypes, cffi | Maximum speed |
| Caching | functools.lru_cache | ลด redundant computation |
| Async I/O | asyncio | แก้ I/O bottleneck |
| Data Structures | set, dict | O(1) vs O(n) lookup |
| Memory Layout | __slots__, array | ลด memory overhead |

**หลักการสำคัญ:**
1. **Measure first** - อย่า optimize โดยไม่มีข้อมูล
2. **Profile** - หา 20% ของโค้ดที่ใช้ 80% ของเวลา
3. **Algorithm first** - การเลือก algorithm ที่ดีสำคัญกว่า micro-optimization
4. **Readability** - อย่า sacrifice readability เพื่อ performance ที่ไม่จำเป็น
5. **Test** - ตรวจสอบว่า optimize แล้วได้ผลลัพธ์ถูกต้อง

---

*Part 96 - Performance Optimization & Profiling | Python Course*
