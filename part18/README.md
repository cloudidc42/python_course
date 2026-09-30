# Part 18: Comprehensions

## สารบัญ
1. [List Comprehension พื้นฐาน](#1-list-comprehension-พื้นฐาน)
2. [List Comprehension กับ Condition](#2-list-comprehension-กับ-condition)
3. [Nested List Comprehension](#3-nested-list-comprehension)
4. [Dictionary Comprehension](#4-dictionary-comprehension)
5. [Set Comprehension](#5-set-comprehension)
6. [Generator Expressions](#6-generator-expressions)
7. [Comprehension vs for loop (Performance)](#7-comprehension-vs-for-loop-performance)
8. [Complex Comprehensions](#8-complex-comprehensions)
9. [เมื่อไหร่ควรและไม่ควรใช้](#9-เมื่อไหร่ควรและไม่ควรใช้)
10. [Walrus Operator (:=) ใน Comprehensions](#10-walrus-operator--ใน-comprehensions)
11. [ตัวอย่างโปรแกรมจริง](#11-ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. List Comprehension พื้นฐาน

List comprehension คือวิธีสร้าง list ในรูปแบบที่กระชับและ Pythonic

### รูปแบบพื้นฐาน

```python
# รูปแบบ:
# [expression for item in iterable]

# แบบ for loop ปกติ
squares_loop = []
for x in range(10):
    squares_loop.append(x ** 2)

# แบบ list comprehension
squares_comp = [x ** 2 for x in range(10)]

print(squares_loop)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(squares_comp)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
print(squares_loop == squares_comp)  # True
```

```python
# ตัวอย่างที่ 2: แปลง string เป็น uppercase
names = ["alice", "bob", "charlie", "diana"]

# แบบ loop
upper_loop = []
for name in names:
    upper_loop.append(name.upper())

# แบบ comprehension
upper_comp = [name.upper() for name in names]

print(upper_comp)  # ['ALICE', 'BOB', 'CHARLIE', 'DIANA']
```

```python
# ตัวอย่างที่ 3: แปลงหลาย types
numbers = ["1", "2", "3", "4", "5"]
integers = [int(n) for n in numbers]
print(integers)  # [1, 2, 3, 4, 5]

# คำนวณ
pi_values = [round(3.14159 * i, 2) for i in range(1, 6)]
print(pi_values)  # [3.14, 6.28, 9.42, 12.57, 15.71]
```

```python
# ตัวอย่างที่ 4: ใช้กับ method calls
words = ["  hello  ", "  world  ", "  python  "]
stripped = [word.strip() for word in words]
lengths = [len(word.strip()) for word in words]

print(stripped)  # ['hello', 'world', 'python']
print(lengths)   # [5, 5, 6]
```

```python
# ตัวอย่างที่ 5: สร้าง list จาก tuple
points = [(1, 2), (3, 4), (5, 6), (7, 8)]

# ดึง x coordinates
x_coords = [point[0] for point in points]
y_coords = [point[1] for point in points]

# คำนวณ distance จาก origin
import math
distances = [math.sqrt(x**2 + y**2) for x, y in points]

print(f"X: {x_coords}")
print(f"Y: {y_coords}")
print(f"Distances: {[round(d, 2) for d in distances]}")
```

```python
# ตัวอย่างที่ 6: flatten nested list
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

## 2. List Comprehension กับ Condition

เพิ่ม condition ในการกรองข้อมูล

### if condition (filtering)

```python
# รูปแบบ:
# [expression for item in iterable if condition]

# กรองเลขคู่
numbers = range(20)
even_numbers = [n for n in numbers if n % 2 == 0]
print(even_numbers)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

```python
# ตัวอย่างที่ 2: กรองจำนวนเฉพาะ
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

primes = [n for n in range(2, 50) if is_prime(n)]
print(primes)  # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
```

```python
# ตัวอย่างที่ 3: กรองข้อความ
words = ["apple", "banana", "cherry", "date", "elderberry", "fig"]

# คำที่มีความยาวมากกว่า 4 ตัวอักษร
long_words = [w for w in words if len(w) > 4]
print(long_words)  # ['apple', 'banana', 'cherry', 'elderberry']

# คำที่ขึ้นต้นด้วย vowel
vowels = "aeiou"
vowel_words = [w for w in words if w[0].lower() in vowels]
print(vowel_words)  # ['apple', 'elderberry']
```

```python
# ตัวอย่างที่ 4: กรอง dict values
students = [
    {"name": "Alice", "grade": 85, "passed": True},
    {"name": "Bob", "grade": 72, "passed": True},
    {"name": "Charlie", "grade": 55, "passed": False},
    {"name": "Diana", "grade": 91, "passed": True},
    {"name": "Eve", "grade": 63, "passed": False},
]

# นักเรียนที่ผ่าน
passed_students = [s["name"] for s in students if s["passed"]]
print(f"ผ่าน: {passed_students}")

# นักเรียนที่ได้เกรด >= 80
high_achievers = [s["name"] for s in students if s["grade"] >= 80]
print(f"เกรด >= 80: {high_achievers}")
```

### if-else expression (ternary)

```python
# รูปแบบ:
# [true_value if condition else false_value for item in iterable]

numbers = range(1, 11)

# เลขคู่/คี่
labels = ["even" if n % 2 == 0 else "odd" for n in numbers]
print(labels)  # ['odd', 'even', 'odd', 'even', ...]

# ค่าบวก/ลบ
values = [-3, -1, 0, 2, 4, -2, 5]
abs_signs = [f"+{v}" if v >= 0 else str(v) for v in values]
print(abs_signs)  # ['-3', '-1', '+0', '+2', '+4', '-2', '+5']
```

```python
# ตัวอย่างที่ 6: categorize grades
grades = [45, 72, 88, 55, 91, 67, 83, 38, 76, 94]
grade_labels = [
    "A" if g >= 90 else
    "B" if g >= 80 else
    "C" if g >= 70 else
    "D" if g >= 60 else
    "F"
    for g in grades
]
print(list(zip(grades, grade_labels)))
```

```python
# ตัวอย่างที่ 7: แทนค่า None ด้วย default
raw_data = [1, None, 3, None, 5, 6, None, 8]
cleaned = [v if v is not None else 0 for v in raw_data]
print(cleaned)  # [1, 0, 3, 0, 5, 6, 0, 8]
```

---

## 3. Nested List Comprehension

### 3.1 สร้าง Matrix

```python
# สร้าง matrix 3x3
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]
for row in matrix:
    print(row)

# Output:
# [1, 2, 3]
# [2, 4, 6]
# [3, 6, 9]
```

```python
# ตัวอย่างที่ 2: Transpose matrix
original = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

transposed = [[row[i] for row in original] for i in range(3)]
print("Original:")
for row in original:
    print(row)

print("\nTransposed:")
for row in transposed:
    print(row)

# หรือใช้ zip
transposed_zip = [list(row) for row in zip(*original)]
print("\nTransposed (zip):")
for row in transposed_zip:
    print(row)
```

```python
# ตัวอย่างที่ 3: Flatten nested list
nested = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# nested 3 levels
deep_nested = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]]
flat_2d = [[item for item in sublist] for nested in deep_nested for sublist in nested]
print(flat_2d)  # [[1, 2], [3, 4], [5, 6], [7, 8]]
```

```python
# ตัวอย่างที่ 4: Multiplication table
size = 5
table = [[f"{i}x{j}={i*j:2d}" for j in range(1, size+1)] for i in range(1, size+1)]
for row in table:
    print("  ".join(row))
```

```python
# ตัวอย่างที่ 5: สร้าง chess board
chess_board = [
    ["W" if (i + j) % 2 == 0 else "B" for j in range(8)]
    for i in range(8)
]

print("Chess Board:")
for row in chess_board:
    print(" ".join(row))
```

```python
# ตัวอย่างที่ 6: Cartesian product
colors = ["red", "green", "blue"]
shapes = ["circle", "square"]

combinations = [f"{color} {shape}" for color in colors for shape in shapes]
print(combinations)
# ['red circle', 'red square', 'green circle', 'green square', 'blue circle', 'blue square']
```

---

## 4. Dictionary Comprehension

```python
# รูปแบบพื้นฐาน:
# {key: value for item in iterable}

# สร้าง dict จาก list
squares_dict = {n: n**2 for n in range(1, 6)}
print(squares_dict)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

```python
# ตัวอย่างที่ 2: invert dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}
```

```python
# ตัวอย่างที่ 3: กรอง dictionary
student_grades = {
    "Alice": 85, "Bob": 72, "Charlie": 55,
    "Diana": 91, "Eve": 63
}

# เฉพาะนักเรียนที่ผ่าน (>= 60)
passing = {name: grade for name, grade in student_grades.items() if grade >= 60}
print(f"ผ่าน: {passing}")

# แปลงเกรดเป็น letter grade
def to_letter_grade(score):
    if score >= 90: return "A"
    elif score >= 80: return "B"
    elif score >= 70: return "C"
    elif score >= 60: return "D"
    else: return "F"

letter_grades = {name: to_letter_grade(grade) for name, grade in student_grades.items()}
print(f"Letter grades: {letter_grades}")
```

```python
# ตัวอย่างที่ 4: สร้างจาก 2 lists
keys = ["name", "age", "city"]
values = ["Alice", 25, "Bangkok"]
person = {k: v for k, v in zip(keys, values)}
print(person)  # {'name': 'Alice', 'age': 25, 'city': 'Bangkok'}
```

```python
# ตัวอย่างที่ 5: group items
words = ["apple", "banana", "avocado", "blueberry", "cherry", "apricot"]

# จัดกลุ่มตามอักษรแรก
word_groups = {}
for word in words:
    first_letter = word[0]
    if first_letter not in word_groups:
        word_groups[first_letter] = []
    word_groups[first_letter].append(word)

# แบบ comprehension (ซับซ้อนกว่า แต่ก็ทำได้)
from collections import defaultdict
word_groups_comp = {
    letter: [w for w in words if w.startswith(letter)]
    for letter in set(w[0] for w in words)
}
print(dict(sorted(word_groups_comp.items())))
```

```python
# ตัวอย่างที่ 6: nested dictionary comprehension
# สร้าง multiplication table เป็น dict
mult_table = {
    i: {j: i * j for j in range(1, 6)}
    for i in range(1, 6)
}

# ดูผลลัพธ์
for i, row in mult_table.items():
    print(f"{i}: {row}")
```

```python
# ตัวอย่างที่ 7: แปลง list of dicts
users = [
    {"id": 1, "name": "Alice", "role": "admin"},
    {"id": 2, "name": "Bob", "role": "user"},
    {"id": 3, "name": "Charlie", "role": "user"},
]

# สร้าง lookup dict by id
user_by_id = {user["id"]: user for user in users}
print(f"User 2: {user_by_id[2]}")

# สร้าง name lookup
name_by_id = {user["id"]: user["name"] for user in users}
print(f"Names: {name_by_id}")
```

---

## 5. Set Comprehension

```python
# รูปแบบพื้นฐาน:
# {expression for item in iterable}

# สร้าง set จาก list (ลบ duplicates อัตโนมัติ)
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique_numbers = {n for n in numbers}
print(unique_numbers)  # {1, 2, 3, 4}
```

```python
# ตัวอย่างที่ 2: set ของ characters
text = "Hello World"
unique_chars = {c.lower() for c in text if c.isalpha()}
print(sorted(unique_chars))  # ['d', 'e', 'h', 'l', 'o', 'r', 'w']
```

```python
# ตัวอย่างที่ 3: set ของ email domains
emails = [
    "alice@gmail.com",
    "bob@yahoo.com",
    "charlie@gmail.com",
    "diana@hotmail.com",
    "eve@gmail.com"
]

domains = {email.split("@")[1] for email in emails}
print(f"Domains: {domains}")  # {'gmail.com', 'yahoo.com', 'hotmail.com'}
```

```python
# ตัวอย่างที่ 4: กรอง set
numbers = range(-10, 11)
positive_squares = {n**2 for n in numbers if n > 0 and n**2 < 50}
print(sorted(positive_squares))  # [1, 4, 9, 16, 25, 36, 49]
```

```python
# ตัวอย่างที่ 5: หาตัวร่วมระหว่าง sets
list1 = [1, 2, 3, 4, 5, 6]
list2 = [4, 5, 6, 7, 8, 9]

common = {x for x in list1 if x in list2}
print(f"Common: {common}")  # {4, 5, 6}

# หรือใช้ set intersection
common2 = set(list1) & set(list2)
print(f"Common (intersection): {common2}")

unique_to_list1 = {x for x in list1 if x not in list2}
print(f"Only in list1: {unique_to_list1}")
```

---

## 6. Generator Expressions

Generator expressions คล้ายกับ list comprehension แต่ไม่สร้าง list ทั้งหมดในหน่วยความจำ

```python
# รูปแบบ:
# (expression for item in iterable)  # ใช้ () แทน []

# List comprehension - สร้าง list ทั้งหมดทันที
list_comp = [x**2 for x in range(10)]
print(f"List: {list_comp}")
print(f"Type: {type(list_comp)}")

# Generator expression - สร้างทีละตัวเมื่อต้องการ
gen_exp = (x**2 for x in range(10))
print(f"\nGenerator: {gen_exp}")
print(f"Type: {type(gen_exp)}")

# ดูค่าด้วย next()
print(f"First: {next(gen_exp)}")
print(f"Second: {next(gen_exp)}")

# หรือแปลงเป็น list
gen_exp2 = (x**2 for x in range(10))
print(f"All: {list(gen_exp2)}")
```

```python
# ตัวอย่างที่ 2: ใช้กับ sum, max, min
numbers = range(1, 1001)

# sum ด้วย generator (ประหยัด memory)
total = sum(n**2 for n in numbers)
print(f"Sum of squares 1-1000: {total}")

max_even = max(n for n in numbers if n % 2 == 0)
print(f"Max even 1-1000: {max_even}")

# any() และ all()
has_prime = any(n for n in range(2, 100) if all(n % i != 0 for i in range(2, n)))
print(f"มีจำนวนเฉพาะ 2-100: {has_prime}")

all_positive = all(n > 0 for n in [1, 2, 3, 4, 5])
print(f"ทุกตัวเป็นบวก: {all_positive}")
```

```python
# ตัวอย่างที่ 3: generator ประหยัด memory มากแค่ไหน
import sys

# รายชื่อไฟล์จำนวนมาก (จำลอง)
n = 1_000_000

# List comprehension - ใช้ memory มาก
list_comp = [i for i in range(n)]
list_size = sys.getsizeof(list_comp)

# Generator - ใช้ memory น้อยมาก
gen_exp = (i for i in range(n))
gen_size = sys.getsizeof(gen_exp)

print(f"List size: {list_size:,} bytes ({list_size / 1024 / 1024:.1f} MB)")
print(f"Generator size: {gen_size} bytes")
print(f"Memory ratio: {list_size / gen_size:,.0f}x")
```

```python
# ตัวอย่างที่ 4: generator pipeline
import re

# Pipeline สำหรับประมวลผลข้อมูล
log_data = [
    "2024-01-15 10:30:00 ERROR: Connection failed",
    "2024-01-15 10:31:00 INFO: User logged in",
    "2024-01-15 10:32:00 ERROR: Database timeout",
    "2024-01-15 10:33:00 WARNING: High memory usage",
    "2024-01-15 10:34:00 ERROR: File not found",
]

# Step 1: กรองเฉพาะ ERROR lines
errors = (line for line in log_data if "ERROR" in line)

# Step 2: แยก timestamp และ message
parsed = (
    {"time": parts[0] + " " + parts[1], "message": " ".join(parts[3:])}
    for line in errors
    for parts in [line.split()]
)

# Step 3: ดึงแค่ messages
messages = (item["message"] for item in parsed)

# ใช้งาน (lazy evaluation - ประมวลผลเฉพาะเมื่อต้องการ)
for msg in messages:
    print(f"Error: {msg}")
```

```python
# ตัวอย่างที่ 5: infinite generator
def infinite_counter(start=0, step=1):
    """Generator ที่นับไม่สิ้นสุด"""
    current = start
    while True:
        yield current
        current += step

# ใช้กับ islice เพื่อจำกัดจำนวน
from itertools import islice

counter = infinite_counter(0, 2)  # เลขคู่
first_10_evens = list(islice(counter, 10))
print(f"Even numbers: {first_10_evens}")

fib_counter = (lambda: (
    lambda: (a := 0, b := 1) and 
    None or
    (yield from iter(lambda: None, None))  # ซับซ้อนเกินไป
))()

# วิธีที่ถูกต้องกว่า
def fibonacci_gen():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci_gen()
first_15_fib = list(islice(fib, 15))
print(f"Fibonacci: {first_15_fib}")
```

---

## 7. Comprehension vs for loop (Performance)

```python
import time

# ทดสอบ performance
def benchmark(func, *args, iterations=3):
    """วัดเวลาทำงาน"""
    times = []
    for _ in range(iterations):
        start = time.perf_counter()
        result = func(*args)
        end = time.perf_counter()
        times.append(end - start)
    
    avg_time = sum(times) / len(times)
    return result, avg_time

N = 100_000

# Test 1: สร้าง list
def with_loop(n):
    result = []
    for i in range(n):
        result.append(i**2)
    return result

def with_comprehension(n):
    return [i**2 for i in range(n)]

def with_map(n):
    return list(map(lambda i: i**2, range(n)))

_, loop_time = benchmark(with_loop, N)
_, comp_time = benchmark(with_comprehension, N)
_, map_time = benchmark(with_map, N)

print(f"For loop:          {loop_time*1000:.2f} ms")
print(f"List comprehension:{comp_time*1000:.2f} ms")
print(f"map():             {map_time*1000:.2f} ms")
print(f"Comprehension speedup: {loop_time/comp_time:.2f}x")
```

```python
# ทดสอบ sum
import sys

N = 1_000_000

# sum กับ generator (ประหยัด memory)
start = time.perf_counter()
total_gen = sum(i**2 for i in range(N))
gen_time = time.perf_counter() - start

# sum กับ list comprehension (ใช้ memory มากกว่า)
start = time.perf_counter()
total_list = sum([i**2 for i in range(N)])
list_time = time.perf_counter() - start

print(f"\nSum N={N:,}:")
print(f"Generator: {gen_time*1000:.2f} ms")
print(f"List comp: {list_time*1000:.2f} ms")
print(f"Same result: {total_gen == total_list}")
```

```python
# เปรียบเทียบ memory usage
import sys

def memory_comparison():
    N = 10_000
    
    # List comprehension
    list_comp = [i**2 for i in range(N)]
    list_mem = sys.getsizeof(list_comp)
    
    # Generator expression
    gen_exp = (i**2 for i in range(N))
    gen_mem = sys.getsizeof(gen_exp)
    
    print(f"N = {N:,}")
    print(f"List comprehension: {list_mem:,} bytes")
    print(f"Generator expression: {gen_mem:,} bytes")
    print(f"Memory saved: {(list_mem - gen_mem) / list_mem * 100:.1f}%")

memory_comparison()
```

```python
# กฎทั่วไปสำหรับ performance

# ✅ ใช้ list comprehension เมื่อ:
# 1. ต้องการ list จริงๆ (random access, หลาย iteration, len())
# 2. ขนาดข้อมูลเล็ก-กลาง (< 100K items)
squares = [x**2 for x in range(1000)]
print(f"List - 5th item: {squares[4]}")  # random access

# ✅ ใช้ generator เมื่อ:
# 1. ต้องการวนซ้ำครั้งเดียว
# 2. ข้อมูลใหญ่ (> 1M items)
# 3. ข้อมูลไม่สิ้นสุด
big_sum = sum(x**2 for x in range(1_000_000))
print(f"Generator sum: {big_sum}")

# ✅ ใช้ generator กับ early termination
def find_first_prime_above(n):
    return next(
        (x for x in range(n, n*2) if all(x % i != 0 for i in range(2, int(x**0.5)+1))),
        None
    )

print(f"First prime above 100: {find_first_prime_above(100)}")
```

---

## 8. Complex Comprehensions

### 8.1 Multiple conditions

```python
# หลาย conditions
numbers = range(-20, 21)

# เลขที่หารด้วย 3 ลงตัว และ ไม่ใช่ศูนย์ และ เป็นบวก
result = [n for n in numbers if n % 3 == 0 and n != 0 and n > 0]
print(result)  # [3, 6, 9, 12, 15, 18]
```

```python
# ตัวอย่างที่ 2: nested comprehension พร้อม condition
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12]
]

# ดึงค่าที่ > 5 จาก matrix
large_values = [val for row in matrix for val in row if val > 5]
print(large_values)  # [6, 7, 8, 9, 10, 11, 12]

# หาตำแหน่งของค่าที่ > 7
positions = [(i, j) for i, row in enumerate(matrix) for j, val in enumerate(row) if val > 7]
print(positions)  # [(1, 3), (2, 0), (2, 1), (2, 2), (2, 3)]
```

```python
# ตัวอย่างที่ 3: comprehension ซับซ้อนสำหรับ text processing
import re

texts = [
    "Python 3.12 is amazing",
    "Java 17 released",
    "JavaScript 2023",
    "Python 3.11 also good",
]

# ดึง Python versions
python_versions = [
    float(re.search(r'Python (\d+\.\d+)', text).group(1))
    for text in texts
    if re.search(r'Python (\d+\.\d+)', text)
]
print(f"Python versions: {python_versions}")

# ดึงตัวเลขทั้งหมดจากทุก text
all_numbers = [
    float(num)
    for text in texts
    for num in re.findall(r'\d+\.?\d*', text)
]
print(f"All numbers: {all_numbers}")
```

```python
# ตัวอย่างที่ 4: data transformation pipeline
employees = [
    {"name": "Alice", "dept": "Engineering", "salary": 80000, "years": 5},
    {"name": "Bob", "dept": "Marketing", "salary": 65000, "years": 3},
    {"name": "Charlie", "dept": "Engineering", "salary": 90000, "years": 8},
    {"name": "Diana", "dept": "HR", "salary": 70000, "years": 6},
    {"name": "Eve", "dept": "Engineering", "salary": 75000, "years": 4},
]

# คำนวณ bonus สำหรับ Engineering ที่ทำงาน > 4 ปี
bonuses = {
    emp["name"]: emp["salary"] * 0.15
    for emp in employees
    if emp["dept"] == "Engineering" and emp["years"] > 4
}
print(f"Bonuses: {bonuses}")

# สร้าง salary report
salary_report = [
    {
        "name": emp["name"],
        "dept": emp["dept"],
        "total": emp["salary"] + (emp["salary"] * 0.1 if emp["years"] >= 5 else 0),
        "seniority": "Senior" if emp["years"] >= 5 else "Junior"
    }
    for emp in employees
]

for report in salary_report:
    print(f"{report['name']} ({report['seniority']}): ฿{report['total']:,.0f}")
```

```python
# ตัวอย่างที่ 5: สร้าง graph adjacency list
edges = [(1, 2), (1, 3), (2, 3), (3, 4), (4, 5), (2, 5)]
vertices = set(v for edge in edges for v in edge)

# Adjacency list ด้วย dict comprehension
graph = {
    v: [u for (a, b) in edges for u in ([b] if a == v else [a] if b == v else [])]
    for v in vertices
}

print("Graph adjacency list:")
for vertex, neighbors in sorted(graph.items()):
    print(f"  {vertex}: {sorted(neighbors)}")
```

---

## 9. เมื่อไหร่ควรและไม่ควรใช้

### ✅ ควรใช้ comprehension

```python
# 1. การแปลงข้อมูลง่ายๆ
numbers = [1, 2, 3, 4, 5]
doubled = [n * 2 for n in numbers]

# 2. การกรองข้อมูล
even = [n for n in range(20) if n % 2 == 0]

# 3. แปลง types
str_nums = ["1", "2", "3"]
ints = [int(n) for n in str_nums]

# 4. สร้าง dict lookup
users = [("Alice", 1), ("Bob", 2), ("Charlie", 3)]
user_dict = {name: id for name, id in users}

# 5. เมื่อโค้ดยังอ่านง่าย
words = ["hello", "world"]
result = [word.upper() for word in words if len(word) > 3]
```

### ❌ ไม่ควรใช้ comprehension

```python
# 1. ซับซ้อนเกินไป - ยากอ่าน
# ❌ ไม่ดี
result = [process(x) for xs in data for x in xs if validate(x) and transform(x) > threshold]

# ✅ ดีกว่า
result = []
for xs in data:
    for x in xs:
        if validate(x) and transform(x) > threshold:
            result.append(process(x))

# 2. มีผลข้างเคียง (side effects)
# ❌ ไม่ดี - comprehension ไม่ควรมี side effects
processed = [print(x) or x*2 for x in numbers]  # print เป็น side effect

# ✅ ดีกว่า
for x in numbers:
    print(x)
processed = [x * 2 for x in numbers]

# 3. ต้องการ multiple statements
# ❌ ไม่สามารถทำได้
# [x = compute(n); y = transform(x); x + y for n in numbers]

# ✅ ใช้ loop แทน
result = []
for n in numbers:
    x = compute_something(n)
    y = transform_something(x)
    result.append(x + y)

# 4. เมื่อต้องการ error handling
# ❌ ไม่ดี
def safe_divide_bad(numbers, divisors):
    return [a/b for a, b in zip(numbers, divisors)]  # อาจเกิด ZeroDivisionError

# ✅ ดีกว่า
def safe_divide_good(numbers, divisors):
    result = []
    for a, b in zip(numbers, divisors):
        try:
            result.append(a / b)
        except ZeroDivisionError:
            result.append(None)
    return result

# ทดสอบ
print(safe_divide_good([10, 20, 30], [2, 0, 5]))

def compute_something(n):
    return n * 2

def transform_something(x):
    return x + 1

numbers = [1, 2, 3, 4, 5]
result = []
for n in numbers:
    x = compute_something(n)
    y = transform_something(x)
    result.append(x + y)
print(result)
```

### Readability Guidelines

```python
# กฎทั่วไป: comprehension ที่ดีควรอ่านในบรรทัดเดียวได้

# ✅ อ่านง่าย (1 บรรทัด)
evens = [n for n in range(20) if n % 2 == 0]

# ✅ ยอมรับได้ (2-3 บรรทัด ถ้าอ่านง่าย)
filtered_sorted = [
    item.strip().lower()
    for item in raw_data
    if item.strip()
]

# ❌ ยากอ่านเกินไป
# ถ้า comprehension ยาวกว่า 2-3 บรรทัด ให้ใช้ loop แทน
raw_data = ["  Hello  ", "  World  ", "", "  Python  "]
filtered_sorted = [
    item.strip().lower()
    for item in raw_data
    if item.strip()
]
print(filtered_sorted)
```

---

## 10. Walrus Operator (:=) ใน Comprehensions

Walrus operator (`:=`) Python 3.8+ ช่วยให้กำหนดค่าและใช้งานพร้อมกันได้

```python
# รูปแบบ: variable := expression

# ตัวอย่างที่ 1: ไม่ต้องคำนวณซ้ำ
# ❌ คำนวณ len(x) สองครั้ง
result_bad = [x for x in ["hi", "hello", "hey", "world"] if len(x) > 3]

# ✅ คำนวณครั้งเดียวและเก็บไว้
result_good = [
    (x, n)  # ใช้ทั้ง x และ n
    for x in ["hi", "hello", "hey", "world"]
    if (n := len(x)) > 3  # เก็บ len(x) ไว้ใน n
]
print(result_good)  # [('hello', 5), ('world', 5)]
```

```python
# ตัวอย่างที่ 2: process และ filter ในขั้นตอนเดียว
import re

texts = [
    "Email: alice@example.com",
    "No email here",
    "Contact: bob@domain.org",
    "Just text",
    "Email: charlie@test.net"
]

# ❌ ต้องทำ regex สองครั้ง
emails_bad = [
    re.search(r'\w+@\w+\.\w+', text).group()
    for text in texts
    if re.search(r'\w+@\w+\.\w+', text)
]

# ✅ ทำครั้งเดียวด้วย walrus
emails_good = [
    match.group()
    for text in texts
    if (match := re.search(r'\w+@\w+\.\w+', text))
]
print(f"Emails: {emails_good}")
```

```python
# ตัวอย่างที่ 3: while loop กับ walrus
import random

# ✅ สั้นกว่าด้วย walrus
random.seed(42)
data_stream = iter([random.randint(1, 100) for _ in range(50)])

# หาค่าแรกที่ > 80 จาก stream
large_values = []
while (val := next(data_stream, None)) is not None:
    if val > 80:
        large_values.append(val)
    if len(large_values) >= 3:  # เก็บแค่ 3 ค่าแรก
        break

print(f"First 3 values > 80: {large_values}")
```

```python
# ตัวอย่างที่ 4: ใช้กับ comprehension ที่มี expensive computation
def expensive_computation(x):
    """จำลองการคำนวณที่ใช้เวลา"""
    return x ** 3 - 2 * x ** 2 + x

numbers = range(-10, 11)

# ✅ คำนวณครั้งเดียว ใช้ทั้งใน filter และ expression
results = [
    (x, result)
    for x in numbers
    if (result := expensive_computation(x)) > 100
]
print(f"Results > 100: {results}")
```

```python
# ตัวอย่างที่ 5: nested list comprehension กับ walrus
matrix = [
    [1, 5, 3, 7, 2],
    [8, 4, 6, 1, 9],
    [3, 7, 2, 8, 5]
]

# หา row ที่มีค่า max > 7 และบอกว่า max เท่าไหร่
rows_with_large_max = [
    (i, max_val, row)
    for i, row in enumerate(matrix)
    if (max_val := max(row)) > 7
]

for row_idx, max_val, row in rows_with_large_max:
    print(f"Row {row_idx}: max={max_val}, data={row}")
```

---

## 11. ตัวอย่างโปรแกรมจริง

### 11.1 Data Transformation Pipeline

```python
# ระบบประมวลผลข้อมูลนักเรียน

students_raw = [
    {"id": "S001", "name": "Alice Smith", "scores": [85, 92, 78, 88, 90]},
    {"id": "S002", "name": "Bob Johnson", "scores": [72, 65, 80, 75, 70]},
    {"id": "S003", "name": "Charlie Brown", "scores": [95, 98, 92, 96, 94]},
    {"id": "S004", "name": "Diana Prince", "scores": [60, 55, 68, 62, 58]},
    {"id": "S005", "name": "Eve Wilson", "scores": [88, 85, 90, 87, 92]},
]

# Step 1: คำนวณสถิติพื้นฐาน
def calculate_stats(scores):
    avg = sum(scores) / len(scores)
    return {
        "avg": round(avg, 2),
        "min": min(scores),
        "max": max(scores),
        "range": max(scores) - min(scores)
    }

# Step 2: กำหนด grade
def get_grade(avg):
    if avg >= 90: return "A"
    elif avg >= 80: return "B"
    elif avg >= 70: return "C"
    elif avg >= 60: return "D"
    else: return "F"

# Step 3: สร้าง report ด้วย comprehension
student_reports = [
    {
        "id": s["id"],
        "name": s["name"],
        "first_name": s["name"].split()[0],
        **calculate_stats(s["scores"]),
        "grade": get_grade(sum(s["scores"]) / len(s["scores"])),
        "passed": sum(s["scores"]) / len(s["scores"]) >= 60
    }
    for s in students_raw
]

# Step 4: Analysis ด้วย comprehensions
passing_students = [r["name"] for r in student_reports if r["passed"]]
grade_distribution = {
    grade: [r["name"] for r in student_reports if r["grade"] == grade]
    for grade in ["A", "B", "C", "D", "F"]
}
top_students = sorted(student_reports, key=lambda x: x["avg"], reverse=True)[:3]

# แสดงผล
print("=== Student Reports ===")
for report in student_reports:
    print(f"{report['id']}: {report['name']}")
    print(f"  Average: {report['avg']}, Grade: {report['grade']}")
    print(f"  Min: {report['min']}, Max: {report['max']}")

print("\n=== Grade Distribution ===")
for grade, names in grade_distribution.items():
    if names:
        print(f"Grade {grade}: {names}")

print(f"\n=== Top 3 Students ===")
for i, student in enumerate(top_students, 1):
    print(f"{i}. {student['name']}: {student['avg']:.2f}")

# Class statistics
all_averages = [r["avg"] for r in student_reports]
class_stats = {
    "class_avg": round(sum(all_averages) / len(all_averages), 2),
    "pass_rate": f"{len(passing_students)}/{len(student_reports)}",
    "highest": max(all_averages),
    "lowest": min(all_averages)
}
print(f"\n=== Class Statistics ===")
for key, value in class_stats.items():
    print(f"  {key}: {value}")
```

### 11.2 Matrix Operations

```python
# Matrix operations ด้วย comprehensions

def create_matrix(rows, cols, value=0):
    """สร้าง matrix ด้วย default value"""
    return [[value] * cols for _ in range(rows)]

def matrix_add(a, b):
    """บวก matrices"""
    return [
        [a[i][j] + b[i][j] for j in range(len(a[0]))]
        for i in range(len(a))
    ]

def matrix_multiply(a, b):
    """คูณ matrices"""
    rows_a, cols_a = len(a), len(a[0])
    rows_b, cols_b = len(b), len(b[0])
    
    if cols_a != rows_b:
        raise ValueError(f"ขนาดไม่ตรง: {cols_a} != {rows_b}")
    
    return [
        [sum(a[i][k] * b[k][j] for k in range(cols_a)) for j in range(cols_b)]
        for i in range(rows_a)
    ]

def matrix_transpose(m):
    """Transpose matrix"""
    return [[m[j][i] for j in range(len(m))] for i in range(len(m[0]))]

def matrix_scalar_mult(m, scalar):
    """คูณด้วย scalar"""
    return [[val * scalar for val in row] for row in m]

def flatten_matrix(m):
    """Flatten เป็น 1D list"""
    return [val for row in m for val in row]

def matrix_filter(m, condition):
    """กรองค่าใน matrix"""
    return [[val for val in row if condition(val)] for row in m]

# ทดสอบ
print("=== Matrix Operations ===")

A = [[1, 2, 3], [4, 5, 6]]
B = [[7, 8, 9], [10, 11, 12]]
C = [[1, 2], [3, 4], [5, 6]]

print(f"A = {A}")
print(f"B = {B}")
print(f"C = {C}")

result_add = matrix_add(A, B)
print(f"\nA + B = {result_add}")

result_mult = matrix_multiply(A, C)
print(f"\nA × C = {result_mult}")

result_transpose = matrix_transpose(A)
print(f"\nA^T = {result_transpose}")

result_scalar = matrix_scalar_mult(A, 2)
print(f"\n2A = {result_scalar}")

flattened = flatten_matrix(A)
print(f"\nFlatten A = {flattened}")

filtered = matrix_filter(A, lambda x: x % 2 == 0)
print(f"\nEven values = {filtered}")
```

### 11.3 Text Processing

```python
import re
from collections import Counter

def analyze_text(text):
    """วิเคราะห์ข้อความด้วย comprehensions"""
    
    # Tokenize
    words = re.findall(r'\b[a-zA-Z]+\b', text.lower())
    
    # Word frequency
    word_freq = Counter(words)
    
    # Unique words (sorted)
    unique_words = sorted({word for word in words})
    
    # Words by length
    words_by_length = {
        length: sorted({w for w in words if len(w) == length})
        for length in range(1, max(len(w) for w in words) + 1)
        if any(len(w) == length for w in words)
    }
    
    # Sentences
    sentences = [s.strip() for s in re.split(r'[.!?]', text) if s.strip()]
    
    # Sentence lengths
    sent_lengths = [len(s.split()) for s in sentences]
    
    return {
        "total_words": len(words),
        "unique_words": len(unique_words),
        "avg_word_length": sum(len(w) for w in words) / len(words) if words else 0,
        "most_common": word_freq.most_common(5),
        "words_by_length": {
            k: v for k, v in words_by_length.items() if len(v) <= 5
        },
        "sentences": len(sentences),
        "avg_sentence_length": sum(sent_lengths) / len(sent_lengths) if sent_lengths else 0,
        "unique_words_list": unique_words[:10],  # แสดงแค่ 10 คำแรก
    }

# ทดสอบ
sample_text = """
Python is a versatile programming language. Python is used for web development, 
data science, and machine learning. Python is easy to learn and powerful.
Many companies use Python for various applications.
"""

analysis = analyze_text(sample_text)

print("=== Text Analysis ===")
print(f"Total words: {analysis['total_words']}")
print(f"Unique words: {analysis['unique_words']}")
print(f"Avg word length: {analysis['avg_word_length']:.2f}")
print(f"Sentences: {analysis['sentences']}")
print(f"Avg sentence length: {analysis['avg_sentence_length']:.1f} words")
print(f"\nMost common words: {analysis['most_common']}")
print(f"\nFirst 10 unique words: {analysis['unique_words_list']}")
print(f"\nWords by length (some):")
for length, words in sorted(analysis['words_by_length'].items()):
    if 3 <= length <= 6:
        print(f"  {length} letters: {words}")
```

---

## 12. แบบฝึกหัด

### ข้อที่ 1

```python
# โจทย์: สร้าง list ของจำนวนเฉพาะ 1-100 ด้วย list comprehension

# เฉลย
def is_prime(n):
    if n < 2:
        return False
    return all(n % i != 0 for i in range(2, int(n**0.5) + 1))

primes = [n for n in range(2, 101) if is_prime(n)]
print(f"จำนวนเฉพาะ 1-100: {primes}")
print(f"จำนวน: {len(primes)}")
```

### ข้อที่ 2

```python
# โจทย์: แปลง list of dicts เป็น dict โดยใช้ id เป็น key
products = [
    {"id": "P001", "name": "Widget", "price": 9.99},
    {"id": "P002", "name": "Gadget", "price": 24.99},
    {"id": "P003", "name": "Doohickey", "price": 4.99},
]

# เฉลย
product_by_id = {p["id"]: p for p in products}
print(f"Product P002: {product_by_id['P002']}")

# หา products ที่ราคา < 10
affordable = {p["id"]: p["name"] for p in products if p["price"] < 10}
print(f"Affordable products: {affordable}")
```

### ข้อที่ 3

```python
# โจทย์: สร้าง Fibonacci ด้วย generator expression และเปรียบเทียบ memory

import sys
from itertools import islice

def fibonacci_gen():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# เฉลย
n = 30

# List comprehension
fib_gen = fibonacci_gen()
fib_list = [next(fib_gen) for _ in range(n)]
list_size = sys.getsizeof(fib_list)

# Generator
fib_gen2 = fibonacci_gen()
gen_expr = islice(fib_gen2, n)
gen_size = sys.getsizeof(gen_expr)

print(f"Fibonacci list ({n} items): {fib_list}")
print(f"List size: {list_size} bytes")
print(f"Generator size: {gen_size} bytes")
```

### ข้อที่ 4

```python
# โจทย์: สร้าง matrix 5x5 ของ multiplication table และ filter ค่าที่ > 12

# เฉลย
mult_table = [[i * j for j in range(1, 6)] for i in range(1, 6)]

print("Multiplication Table:")
for row in mult_table:
    print([f"{n:2d}" for n in row])

large_values = [val for row in mult_table for val in row if val > 12]
print(f"\nValues > 12: {sorted(set(large_values))}")
```

### ข้อที่ 5-10

```python
# ข้อที่ 5: Anagram finder
words = ["listen", "silent", "enlist", "hello", "world", "inlets"]

# เฉลย
def get_sorted_chars(word):
    return "".join(sorted(word))

anagram_groups = {}
for word in words:
    key = get_sorted_chars(word)
    if key not in anagram_groups:
        anagram_groups[key] = []
    anagram_groups[key].append(word)

anagram_groups_comp = {
    get_sorted_chars(w): [x for x in words if get_sorted_chars(x) == get_sorted_chars(w)]
    for w in words
}

# ดู groups ที่มีมากกว่า 1 คำ
anagram_sets = [group for group in set(
    tuple(sorted(v)) for v in anagram_groups.values()
) if len(group) > 1]

print("Anagram groups:")
for group in anagram_sets:
    print(f"  {list(group)}")
```

```python
# ข้อที่ 6: CSV parser ด้วย comprehension
csv_data = """name,age,score,city
Alice,25,85.5,Bangkok
Bob,30,72.0,Chiang Mai
Charlie,28,91.5,Bangkok
Diana,22,68.0,Phuket
"""

def parse_csv(csv_string):
    lines = [line for line in csv_string.strip().split('\n') if line]
    headers = lines[0].split(',')
    
    records = [
        {
            key: (float(val) if val.replace('.', '').isdigit() else val)
            for key, val in zip(headers, line.split(','))
        }
        for line in lines[1:]
    ]
    
    return records

parsed = parse_csv(csv_data)
print("Parsed CSV:")
for record in parsed:
    print(f"  {record}")

# กรอง
bangkok_students = [r["name"] for r in parsed if r["city"] == "Bangkok"]
print(f"\nBangkok students: {bangkok_students}")

high_scores = {r["name"]: r["score"] for r in parsed if r["score"] >= 80}
print(f"High scorers: {high_scores}")
```

```python
# ข้อที่ 7: สร้าง spiral matrix
def create_spiral(n):
    """สร้าง n×n spiral matrix"""
    matrix = create_matrix(n, n)
    num = 1
    top, bottom, left, right = 0, n-1, 0, n-1
    
    while top <= bottom and left <= right:
        for i in range(left, right + 1):
            matrix[top][i] = num
            num += 1
        top += 1
        
        for i in range(top, bottom + 1):
            matrix[i][right] = num
            num += 1
        right -= 1
        
        for i in range(right, left - 1, -1):
            matrix[bottom][i] = num
            num += 1
        bottom -= 1
        
        for i in range(bottom, top - 1, -1):
            matrix[i][left] = num
            num += 1
        left += 1
    
    return matrix

def create_matrix(rows, cols, value=0):
    return [[value] * cols for _ in range(rows)]

spiral = create_spiral(4)
print("Spiral Matrix 4×4:")
for row in spiral:
    print([f"{n:3d}" for n in row])
```

```python
# ข้อที่ 8: Set operations ด้วย comprehension
class SetAnalyzer:
    def __init__(self, set_a, set_b):
        self.a = set(set_a)
        self.b = set(set_b)
    
    @property
    def union(self):
        return {x for x in self.a | self.b}
    
    @property
    def intersection(self):
        return {x for x in self.a if x in self.b}
    
    @property
    def difference_a(self):
        """ใน A แต่ไม่ใน B"""
        return {x for x in self.a if x not in self.b}
    
    @property
    def difference_b(self):
        """ใน B แต่ไม่ใน A"""
        return {x for x in self.b if x not in self.a}
    
    @property
    def symmetric_difference(self):
        """ใน A หรือ B แต่ไม่ใช่ทั้งคู่"""
        return {x for x in self.a | self.b if x not in self.a or x not in self.b}

a = {1, 2, 3, 4, 5}
b = {3, 4, 5, 6, 7}

analyzer = SetAnalyzer(a, b)
print(f"A = {a}, B = {b}")
print(f"Union: {sorted(analyzer.union)}")
print(f"Intersection: {sorted(analyzer.intersection)}")
print(f"A - B: {sorted(analyzer.difference_a)}")
print(f"B - A: {sorted(analyzer.difference_b)}")
print(f"Symmetric diff: {sorted(analyzer.symmetric_difference)}")
```

```python
# ข้อที่ 9: frequency analysis ด้วย comprehension
import re
from collections import Counter

def frequency_analysis(text, top_n=10):
    """วิเคราะห์ความถี่คำ"""
    # Tokenize
    words = re.findall(r'\b[a-zA-Zก-๙]+\b', text.lower())
    
    # Stopwords (คำที่ไม่สำคัญ)
    stopwords = {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'is', 'are', 'was', 'were', 'be', 'been', 'have', 'has', 'do', 'does', 'for', 'of', 'it', 'this', 'that'}
    
    # กรอง stopwords
    meaningful_words = [w for w in words if w not in stopwords]
    
    # นับความถี่
    freq = Counter(meaningful_words)
    top_words = freq.most_common(top_n)
    
    # สถิติ
    unique = set(meaningful_words)
    
    return {
        "top_words": top_words,
        "total_words": len(meaningful_words),
        "unique_words": len(unique),
        "vocabulary_richness": len(unique) / len(meaningful_words) if meaningful_words else 0
    }

text = """
Python is a high-level programming language. Python supports multiple programming 
paradigms. Python is widely used in data science, web development, automation, 
and artificial intelligence. Python code is readable and easy to understand.
"""

analysis = frequency_analysis(text)
print("=== Frequency Analysis ===")
print(f"Total meaningful words: {analysis['total_words']}")
print(f"Unique words: {analysis['unique_words']}")
print(f"Vocabulary richness: {analysis['vocabulary_richness']:.2%}")
print("\nTop words:")
for word, count in analysis['top_words']:
    bar = "█" * count
    print(f"  {word:15s}: {count:2d} {bar}")
```

```python
# ข้อที่ 10: ระบบ inventory ด้วย comprehension
inventory = [
    {"id": "A001", "name": "Widget A", "category": "Electronics", "stock": 50, "price": 29.99, "reorder_point": 10},
    {"id": "A002", "name": "Widget B", "category": "Electronics", "stock": 8, "price": 49.99, "reorder_point": 15},
    {"id": "B001", "name": "Gadget X", "category": "Tools", "stock": 0, "price": 99.99, "reorder_point": 5},
    {"id": "B002", "name": "Gadget Y", "category": "Tools", "stock": 25, "price": 79.99, "reorder_point": 8},
    {"id": "C001", "name": "Doohickey", "category": "Misc", "stock": 100, "price": 4.99, "reorder_point": 20},
]

# 1. สินค้าที่ stock ต่ำกว่า reorder point
low_stock = [
    {"id": item["id"], "name": item["name"], "stock": item["stock"]}
    for item in inventory
    if item["stock"] <= item["reorder_point"]
]
print("Low stock items:")
for item in low_stock:
    print(f"  {item['id']}: {item['name']} (stock: {item['stock']})")

# 2. มูลค่า inventory แยกตาม category
inventory_value = {
    category: sum(
        item["price"] * item["stock"]
        for item in inventory
        if item["category"] == category
    )
    for category in {item["category"] for item in inventory}
}
print(f"\nInventory value by category:")
for cat, value in sorted(inventory_value.items()):
    print(f"  {cat}: ฿{value:,.2f}")

# 3. สรุป
total_value = sum(item["price"] * item["stock"] for item in inventory)
out_of_stock = [item["name"] for item in inventory if item["stock"] == 0]
print(f"\nTotal inventory value: ฿{total_value:,.2f}")
print(f"Out of stock: {out_of_stock}")
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

| Comprehension | รูปแบบ | ใช้เพื่อ |
|--------------|--------|---------|
| List | `[expr for x in iter]` | สร้าง list |
| List + Filter | `[expr for x in iter if cond]` | กรองและแปลง |
| Nested List | `[expr for x in a for y in b]` | flatten, Cartesian product |
| Dict | `{k: v for x in iter}` | สร้าง/แปลง dictionary |
| Set | `{expr for x in iter}` | สร้าง set ไม่ซ้ำ |
| Generator | `(expr for x in iter)` | ประหยัด memory |

### เมื่อไหร่ใช้อะไร

| สถานการณ์ | ใช้ |
|----------|-----|
| ต้องการ list | List comprehension |
| ข้อมูลใหญ่/ไม่สิ้นสุด | Generator expression |
| สร้าง dict จาก data | Dict comprehension |
| ลบ duplicates | Set comprehension |
| Logic ซับซ้อน | for loop |
| Side effects | for loop |

### ขั้นต่อไป

ในบทถัดไป (Part 19) เราจะเรียนรู้เกี่ยวกับ **Lambda, map, filter, reduce & Functional Programming** ซึ่งเป็นแนวคิดการเขียนโปรแกรมแบบ functional
