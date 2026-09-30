# Part 07 - Loops: for Loop

## สารบัญ

1. [for Loop พื้นฐาน](#1-for-loop-พื้นฐาน)
2. [Iterating over Collections](#2-iterating-over-collections)
3. [range() Function](#3-range-function)
4. [enumerate() Function](#4-enumerate-function)
5. [zip() Function](#5-zip-function)
6. [break, continue, else ใน for Loop](#6-break-continue-else-ใน-for-loop)
7. [Nested for Loops](#7-nested-for-loops)
8. [for Loop กับ List Comprehension](#8-for-loop-กับ-list-comprehension)
9. [itertools Module เบื้องต้น](#9-itertools-module-เบื้องต้น)
10. [ตัวอย่างโปรแกรมจริง](#10-ตัวอย่างโปรแกรมจริง)
11. [แบบฝึกหัด](#11-แบบฝึกหัด)

---

## 1. for Loop พื้นฐาน

### ความหมายและการทำงาน

`for` loop ใน Python ใช้สำหรับวนซ้ำผ่าน **iterable** (สิ่งที่สามารถวนซ้ำได้) เช่น list, string, tuple, dict, set, range และอื่นๆ

### Syntax

```python
for variable in iterable:
    # โค้ดที่จะทำงานซ้ำ
    statement
```

### ตัวอย่างที่ 1: for loop พื้นฐาน

```python
# วนซ้ำผ่าน list
fruits = ["apple", "banana", "cherry", "date"]

print("รายการผลไม้:")
for fruit in fruits:
    print(f"  - {fruit}")

print(f"\nจำนวนผลไม้ทั้งหมด: {len(fruits)} ชนิด")
```

**Output:**
```
รายการผลไม้:
  - apple
  - banana
  - cherry
  - date

จำนวนผลไม้ทั้งหมด: 4 ชนิด
```

### ตัวอย่างที่ 2: for loop กับ string

```python
# วนซ้ำผ่านตัวอักษรใน string
word = "Python"

print(f"ตัวอักษรในคำว่า '{word}':")
for char in word:
    print(f"  '{char}'")

# นับจำนวนสระ
vowels = "aeiouAEIOU"
count = 0
for char in word:
    if char in vowels:
        count += 1
print(f"\nจำนวนสระในคำว่า '{word}': {count} ตัว")
```

### ตัวอย่างที่ 3: for loop กับ tuple

```python
# วนซ้ำผ่าน tuple
coordinates = (10, 20, 30, 40, 50)

total = 0
for value in coordinates:
    total += value
    print(f"  ค่า: {value}, ผลรวมสะสม: {total}")

print(f"\nผลรวมทั้งหมด: {total}")
print(f"ค่าเฉลี่ย: {total / len(coordinates):.2f}")
```

### ตัวอย่างที่ 4: for loop กับตัวเลข

```python
# คำนวณ factorial
def factorial(n):
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

# ทดสอบ
for n in range(1, 11):
    print(f"{n}! = {factorial(n):,}")
```

**Output:**
```
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
6! = 720
7! = 5,040
8! = 40,320
9! = 362,880
10! = 3,628,800
```

---

## 2. Iterating over Collections

### 2.1 Iterating over List

```python
# ตัวอย่างที่ 5: วนซ้ำผ่าน list หลายรูปแบบ
students = [
    {"name": "Alice", "score": 85},
    {"name": "Bob", "score": 72},
    {"name": "Charlie", "score": 91},
    {"name": "Diana", "score": 68},
]

# วนซ้ำแบบง่าย
print("=== รายชื่อนักศึกษา ===")
for student in students:
    name = student["name"]
    score = student["score"]
    grade = "A" if score >= 80 else "B" if score >= 70 else "C"
    print(f"  {name}: {score} คะแนน (เกรด {grade})")
```

### 2.2 Iterating over Dictionary

```python
# ตัวอย่างที่ 6: วนซ้ำผ่าน dictionary
person = {
    "name": "Alice",
    "age": 25,
    "city": "Bangkok",
    "job": "Engineer"
}

print("=== วนผ่าน keys ===")
for key in person:              # หรือ person.keys()
    print(f"  Key: {key}")

print("\n=== วนผ่าน values ===")
for value in person.values():
    print(f"  Value: {value}")

print("\n=== วนผ่าน key-value pairs ===")
for key, value in person.items():
    print(f"  {key}: {value}")
```

### 2.3 Iterating over Set

```python
# ตัวอย่างที่ 7: วนซ้ำผ่าน set
unique_colors = {"red", "green", "blue", "yellow", "orange"}

print("สีทั้งหมด (ลำดับไม่แน่นอน):")
for color in unique_colors:
    print(f"  - {color}")

# Set ไม่มีลำดับ จะได้ผลลัพธ์ต่างกันแต่ละครั้ง
# ถ้าต้องการลำดับให้ใช้ sorted()
print("\nสีทั้งหมด (เรียงตามตัวอักษร):")
for color in sorted(unique_colors):
    print(f"  - {color}")
```

### 2.4 Iterating over String Methods

```python
# ตัวอย่างที่ 8: ประมวลผล string
sentence = "The quick brown fox jumps over the lazy dog"
words = sentence.split()

# นับความถี่ของตัวอักษร
char_freq = {}
for char in sentence.lower():
    if char.isalpha():
        char_freq[char] = char_freq.get(char, 0) + 1

# แสดงผล 5 ตัวอักษรที่พบมากที่สุด
sorted_chars = sorted(char_freq.items(), key=lambda x: x[1], reverse=True)
print("ตัวอักษรที่พบมากที่สุด 5 อันดับ:")
for char, count in sorted_chars[:5]:
    bar = "█" * count
    print(f"  '{char}': {count:2d} ครั้ง {bar}")
```

---

## 3. range() Function

### รูปแบบต่างๆ ของ range()

```python
# Syntax:
# range(stop)
# range(start, stop)
# range(start, stop, step)
```

### ตัวอย่างที่ 9: range() แบบต่างๆ

```python
# range(stop) - เริ่มจาก 0 ถึง stop-1
print("range(5):", list(range(5)))

# range(start, stop)
print("range(2, 8):", list(range(2, 8)))

# range(start, stop, step)
print("range(0, 20, 2):", list(range(0, 20, 2)))   # เลขคู่
print("range(1, 20, 2):", list(range(1, 20, 2)))   # เลขคี่

# range แบบถอยหลัง
print("range(10, 0, -1):", list(range(10, 0, -1)))  # 10 ถึง 1
print("range(10, -1, -1):", list(range(10, -1, -1)))  # 10 ถึง 0

# range ขนาดใหญ่ (ไม่สร้าง list จริงๆ จนกว่าจะ iterate)
big_range = range(1, 1_000_001)
print(f"\nrange(1, 1000001) มีสมาชิก: {len(big_range):,} ตัว")
print(f"ค่าแรก: {big_range[0]}, ค่าสุดท้าย: {big_range[-1]}")
```

### ตัวอย่างที่ 10: การใช้ range() กับ for

```python
# คำนวณผลรวม 1 ถึง n
def sum_to_n(n):
    total = 0
    for i in range(1, n + 1):
        total += i
    return total

# สูตร: n*(n+1)/2
for n in [10, 100, 1000]:
    calc = sum_to_n(n)
    formula = n * (n + 1) // 2
    print(f"Sum(1 to {n:4d}) = {calc:,} (formula: {formula:,}) - {'✓' if calc == formula else '✗'}")

print()

# สร้างตาราง
print("ตาราง multiplication:")
print("    ", end="")
for i in range(1, 6):
    print(f"{i:4}", end="")
print()
print("    " + "-" * 20)

for i in range(1, 6):
    print(f"{i:2} |", end="")
    for j in range(1, 6):
        print(f"{i*j:4}", end="")
    print()
```

### ตัวอย่างที่ 11: range กับ index

```python
# ใช้ range เพื่อเข้าถึง index
colors = ["red", "green", "blue", "yellow", "purple"]

# เข้าถึงแบบ index
print("เข้าถึงด้วย index:")
for i in range(len(colors)):
    print(f"  colors[{i}] = {colors[i]}")

# แก้ไขค่าใน list
numbers = [1, 2, 3, 4, 5]
print(f"\nก่อน: {numbers}")
for i in range(len(numbers)):
    numbers[i] = numbers[i] ** 2  # ยกกำลัง 2
print(f"หลัง (ยกกำลัง 2): {numbers}")
```

---

## 4. enumerate() Function

### ความหมาย

`enumerate()` ใช้เมื่อต้องการทั้ง **index** และ **value** ในการวนซ้ำ

### Syntax

```python
enumerate(iterable, start=0)
```

### ตัวอย่างที่ 12: enumerate() พื้นฐาน

```python
# แบบไม่ใช้ enumerate (ไม่แนะนำ)
fruits = ["apple", "banana", "cherry"]
print("แบบไม่ใช้ enumerate:")
for i in range(len(fruits)):
    print(f"  {i + 1}. {fruits[i]}")

# แบบใช้ enumerate (แนะนำ)
print("\nแบบใช้ enumerate:")
for index, fruit in enumerate(fruits):
    print(f"  {index}. {fruit}")

# กำหนด start index
print("\nเริ่มนับที่ 1:")
for index, fruit in enumerate(fruits, start=1):
    print(f"  {index}. {fruit}")
```

### ตัวอย่างที่ 13: enumerate() ใน use cases จริง

```python
# หาตำแหน่งที่ค่าเงื่อนไขเป็นจริง
numbers = [3, 7, 2, 8, 1, 9, 4, 6, 5, 10]

print("ตัวเลขที่มากกว่า 5:")
for idx, num in enumerate(numbers):
    if num > 5:
        print(f"  numbers[{idx}] = {num}")

# สร้าง numbered list
tasks = [
    "อ่านหนังสือ",
    "ทบทวนโค้ด",
    "ทำแบบฝึกหัด",
    "ดูวิดีโอ",
    "ทำ project"
]

print("\nรายการสิ่งที่ต้องทำ:")
for i, task in enumerate(tasks, 1):
    checkbox = "[ ]"
    print(f"  {i}. {checkbox} {task}")

# แก้ไข list ด้วย enumerate
data = ["  hello  ", "  world  ", "  python  ", "  code  "]
print(f"\nก่อน: {data}")
for i, item in enumerate(data):
    data[i] = item.strip()
print(f"หลัง (strip): {data}")
```

### ตัวอย่างที่ 14: enumerate() กับ dict comprehension

```python
# สร้าง dictionary จาก list ด้วย enumerate
menu_items = ["ข้าวผัด", "ต้มยำ", "แกงเขียวหวาน", "ผัดไทย", "ยำวุ้นเส้น"]
prices = [80, 120, 150, 100, 90]

# สร้าง menu dict
menu = {item: price for item, price in zip(menu_items, prices)}

print("เมนูอาหาร:")
for num, (item, price) in enumerate(menu.items(), 1):
    print(f"  {num:2d}. {item:<15} {price:>5} บาท")

print(f"\nราคาเฉลี่ย: {sum(menu.values())/len(menu):.2f} บาท")
```

---

## 5. zip() Function

### ความหมาย

`zip()` ใช้รวม (combine) iterable หลายตัวเข้าด้วยกัน สร้าง tuple pairs

### ตัวอย่างที่ 15: zip() พื้นฐาน

```python
names = ["Alice", "Bob", "Charlie", "Diana"]
ages = [25, 30, 22, 28]
cities = ["Bangkok", "Chiang Mai", "Phuket", "Pattaya"]

# zip รวม 2 lists
print("ชื่อ + อายุ:")
for name, age in zip(names, ages):
    print(f"  {name}: {age} ปี")

# zip รวม 3 lists
print("\nชื่อ + อายุ + เมือง:")
for name, age, city in zip(names, ages, cities):
    print(f"  {name} อายุ {age} ปี อยู่ที่ {city}")
```

### ตัวอย่างที่ 16: zip() กับ lists ความยาวต่างกัน

```python
# zip หยุดที่ iterable ที่สั้นที่สุด
list1 = [1, 2, 3, 4, 5]
list2 = ["a", "b", "c"]

print("zip ปกติ (หยุดที่ตัวสั้น):")
for x, y in zip(list1, list2):
    print(f"  {x}, {y}")

# ใช้ itertools.zip_longest เพื่อไม่ตัดทิ้ง
from itertools import zip_longest
print("\nzip_longest (เติม None):")
for x, y in zip_longest(list1, list2):
    print(f"  {x}, {y}")

print("\nzip_longest (เติมค่า default):")
for x, y in zip_longest(list1, list2, fillvalue="N/A"):
    print(f"  {x}, {y}")
```

### ตัวอย่างที่ 17: zip() สร้าง dictionary

```python
# สร้าง dict จาก 2 lists
keys = ["name", "age", "city", "job"]
values = ["Alice", 25, "Bangkok", "Engineer"]

# แบบ 1: dict constructor
person_dict = dict(zip(keys, values))
print("Person dict:", person_dict)

# แบบ 2: dict comprehension
person_dict2 = {k: v for k, v in zip(keys, values)}
print("Person dict2:", person_dict2)

# Unzip (แยก)
pairs = [(1, "a"), (2, "b"), (3, "c"), (4, "d")]
numbers, letters = zip(*pairs)  # * คือ unpack
print(f"\nNumbers: {numbers}")
print(f"Letters: {letters}")
```

### ตัวอย่างที่ 18: zip() กับ enumerate()

```python
# รวม zip กับ enumerate
subjects = ["คณิตศาสตร์", "ภาษาไทย", "วิทยาศาสตร์", "ประวัติศาสตร์"]
scores = [85, 90, 78, 92]
max_scores = [100, 100, 100, 100]

print(f"{'No.':<4} {'วิชา':<20} {'คะแนน':>8} {'เต็ม':>6} {'%':>6}")
print("-" * 50)

for i, (subject, score, max_score) in enumerate(zip(subjects, scores, max_scores), 1):
    percentage = score / max_score * 100
    bar = "█" * int(percentage / 10)
    print(f"{i:<4} {subject:<20} {score:>8} {max_score:>6} {percentage:>5.1f}%")

avg = sum(scores) / len(scores)
print("-" * 50)
print(f"{'เฉลี่ย':>34} {avg:>8.1f}")
```

---

## 6. break, continue, else ใน for Loop

### break - หยุดการวนซ้ำ

```python
# ตัวอย่างที่ 19: break
print("=== break ===")
numbers = [3, 7, 2, 8, 5, 1, 9, 4]

print("ค้นหาตัวเลขแรกที่มากกว่า 6:")
for num in numbers:
    print(f"  ตรวจสอบ {num}...", end="")
    if num > 6:
        print(f" พบ! หยุดการค้นหา")
        found = num
        break
    print(" ไม่ผ่าน")
else:
    found = None
    print("ไม่พบตัวเลข")

print(f"ผลลัพธ์: {found}")
```

### continue - ข้ามการวนซ้ำในรอบนั้น

```python
# ตัวอย่างที่ 20: continue
print("\n=== continue ===")
numbers = range(1, 11)

print("เลขคี่ระหว่าง 1-10:")
for num in numbers:
    if num % 2 == 0:  # ข้ามเลขคู่
        continue
    print(f"  {num}", end="")
print()

# กรองข้อมูล
data = ["apple", "", "banana", None, "cherry", "", "date", None]
print("\nกรองข้อมูลว่าง:")
valid_items = []
for item in data:
    if not item:  # ข้ามค่าว่าง
        continue
    valid_items.append(item)
print(f"ก่อนกรอง: {data}")
print(f"หลังกรอง: {valid_items}")
```

### else ใน for Loop

```python
# ตัวอย่างที่ 21: for...else
# else จะทำงานเมื่อ loop จบครบโดยไม่มี break

print("\n=== for...else ===")

def find_prime(numbers_list):
    """หาตัวเลขแรกที่เป็นจำนวนเฉพาะ"""
    for num in numbers_list:
        is_prime = True
        if num < 2:
            continue
        for i in range(2, int(num**0.5) + 1):
            if num % i == 0:
                is_prime = False
                break
        if is_prime:
            return num
    return None

test_lists = [
    [4, 6, 8, 10, 7, 12],
    [4, 6, 8, 10, 12],
]

for lst in test_lists:
    prime = find_prime(lst)
    if prime:
        print(f"{lst}: พบจำนวนเฉพาะ {prime}")
    else:
        print(f"{lst}: ไม่พบจำนวนเฉพาะ")

# for...else แบบชัดเจน
print()
target = 7
search_list = [1, 3, 5, 7, 9, 11]

for item in search_list:
    if item == target:
        print(f"พบ {target} ในรายการ!")
        break
else:
    print(f"ไม่พบ {target} ในรายการ")
```

---

## 7. Nested for Loops

### ความหมาย

Nested for loop คือ for loop ที่อยู่ภายใน for loop อีกอัน ใช้สำหรับประมวลผลข้อมูล 2 มิติหรือการสร้าง combination

### ตัวอย่างที่ 22: ตารางสูตรคูณ

```python
# สร้างตารางสูตรคูณ
size = 10

print("ตารางสูตรคูณ 10x10")
print("    ", end="")
for i in range(1, size + 1):
    print(f"{i:4}", end="")
print()
print("    " + "----" * size)

for i in range(1, size + 1):
    print(f"{i:2} |", end="")
    for j in range(1, size + 1):
        print(f"{i*j:4}", end="")
    print()
```

### ตัวอย่างที่ 23: ลวดลายตัวเลข

```python
# สร้างลวดลายต่างๆ
n = 5

print("สามเหลี่ยมตัวเลข:")
for i in range(1, n + 1):
    for j in range(1, i + 1):
        print(j, end=" ")
    print()

print("\nสามเหลี่ยมกลับหัว:")
for i in range(n, 0, -1):
    for j in range(1, i + 1):
        print(j, end=" ")
    print()

print("\nสามเหลี่ยมดาว:")
for i in range(1, n + 1):
    print(" " * (n - i) + "* " * i)

print("\nรูปเพชร:")
for i in range(1, n + 1):
    print(" " * (n - i) + "* " * i)
for i in range(n - 1, 0, -1):
    print(" " * (n - i) + "* " * i)
```

### ตัวอย่างที่ 24: ประมวลผล Matrix

```python
# ประมวลผล 2D list (matrix)
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print("Matrix:")
for row in matrix:
    for val in row:
        print(f"{val:3}", end="")
    print()

# คำนวณผลรวมของแต่ละแถว
print("\nผลรวมแต่ละแถว:")
for i, row in enumerate(matrix):
    row_sum = sum(row)
    print(f"  แถว {i+1}: {row} -> ผลรวม = {row_sum}")

# Transpose (สลับแถวและคอลัมน์)
rows = len(matrix)
cols = len(matrix[0])
transposed = [[0] * rows for _ in range(cols)]

for i in range(rows):
    for j in range(cols):
        transposed[j][i] = matrix[i][j]

print("\nMatrix Transposed:")
for row in transposed:
    for val in row:
        print(f"{val:3}", end="")
    print()
```

---

## 8. for Loop กับ List Comprehension

### List Comprehension คืออะไร

List comprehension คือการสร้าง list ใหม่จาก iterable ในบรรทัดเดียว ทำให้โค้ดกระชับและ Pythonic มากขึ้น

### Syntax

```python
[expression for item in iterable]
[expression for item in iterable if condition]
[expression for item in iterable if condition else other_expression]
```

### ตัวอย่างที่ 25: List Comprehension พื้นฐาน

```python
# แบบ for loop ปกติ
squares_normal = []
for i in range(1, 11):
    squares_normal.append(i ** 2)
print(f"for loop: {squares_normal}")

# แบบ List Comprehension
squares_comp = [i ** 2 for i in range(1, 11)]
print(f"comprehension: {squares_comp}")

# เลขคู่ยกกำลัง 2
even_squares = [i ** 2 for i in range(1, 11) if i % 2 == 0]
print(f"เลขคู่ยกกำลัง 2: {even_squares}")

# แปลงข้อมูล
names = ["alice", "bob", "charlie", "diana"]
upper_names = [name.upper() for name in names]
print(f"ชื่อพิมพ์ใหญ่: {upper_names}")

# กรองและแปลง
words = ["hello", "world", "python", "programming", "is", "fun"]
long_words_upper = [w.upper() for w in words if len(w) > 4]
print(f"คำยาวกว่า 4 ตัวอักษร: {long_words_upper}")
```

### ตัวอย่างที่ 26: Nested List Comprehension

```python
# Flatten nested list
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# แบบ for loop
flattened_loop = []
for sublist in nested:
    for item in sublist:
        flattened_loop.append(item)
print(f"for loop flatten: {flattened_loop}")

# แบบ List Comprehension
flattened_comp = [item for sublist in nested for item in sublist]
print(f"comprehension flatten: {flattened_comp}")

# สร้าง matrix
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]
print(f"\nMatrix 3x3:")
for row in matrix:
    print(f"  {row}")
```

### ตัวอย่างที่ 27: Dictionary และ Set Comprehension

```python
# Dict Comprehension
numbers = [1, 2, 3, 4, 5]
squares_dict = {n: n**2 for n in numbers}
print(f"squares_dict: {squares_dict}")

# กรองใน dict comprehension
even_squares_dict = {n: n**2 for n in numbers if n % 2 == 0}
print(f"even squares dict: {even_squares_dict}")

# Invert dictionary
original = {"a": 1, "b": 2, "c": 3, "d": 4}
inverted = {v: k for k, v in original.items()}
print(f"Original: {original}")
print(f"Inverted: {inverted}")

# Set Comprehension
data = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique_evens = {n for n in data if n % 2 == 0}
print(f"\nunique evens: {unique_evens}")
```

---

## 9. itertools Module เบื้องต้น

### ความหมาย

`itertools` เป็น module ที่ built-in มาพร้อม Python มี function ต่างๆ สำหรับการทำงานกับ iterables อย่างมีประสิทธิภาพ

### ตัวอย่างที่ 28: itertools พื้นฐาน

```python
import itertools

# count() - นับต่อเนื่องไม่มีที่สิ้นสุด
print("count(1, 2) แสดง 5 ตัวแรก:")
for i, val in enumerate(itertools.count(1, 2)):
    print(f"  {val}", end="")
    if i >= 4:
        break
print()

# cycle() - วนซ้ำ iterable ไม่มีที่สิ้นสุด
print("\ncycle(['A', 'B', 'C']) แสดง 7 ตัวแรก:")
for i, val in enumerate(itertools.cycle(['A', 'B', 'C'])):
    print(f"  {val}", end="")
    if i >= 6:
        break
print()

# repeat() - ทำซ้ำค่า
print("\nrepeat('Hello', 4):")
for val in itertools.repeat("Hello", 4):
    print(f"  {val}", end="")
print()
```

### ตัวอย่างที่ 29: itertools สำหรับ Combinations

```python
import itertools

items = ['A', 'B', 'C', 'D']

# product() - Cartesian product
print("product(['A','B'], ['1','2']):")
for combo in itertools.product(['A', 'B'], ['1', '2']):
    print(f"  {combo}", end="")
print()

# permutations() - การเรียงสับเปลี่ยน
print("\npermutations(['A','B','C'], 2):")
perms = list(itertools.permutations(['A', 'B', 'C'], 2))
print(f"  จำนวน: {len(perms)}")
for p in perms:
    print(f"  {''.join(p)}", end=" ")
print()

# combinations() - การรวมกัน (ไม่สนลำดับ)
print("\ncombinations([1,2,3,4], 2):")
combos = list(itertools.combinations([1, 2, 3, 4], 2))
print(f"  จำนวน: {len(combos)}")
for c in combos:
    print(f"  {c}", end=" ")
print()

# combinations_with_replacement()
print("\ncombinations_with_replacement(['A','B','C'], 2):")
cwrs = list(itertools.combinations_with_replacement(['A', 'B', 'C'], 2))
for c in cwrs:
    print(f"  {''.join(c)}", end=" ")
print()
```

### ตัวอย่างที่ 30: itertools สำหรับ Data Processing

```python
import itertools

# chain() - รวม iterables
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]

print("chain:")
for val in itertools.chain(list1, list2, list3):
    print(val, end=" ")
print()

# groupby() - จัดกลุ่ม
data = [
    ("Alice", "Engineering"),
    ("Bob", "Marketing"),
    ("Charlie", "Engineering"),
    ("Diana", "HR"),
    ("Eve", "Marketing"),
    ("Frank", "Engineering"),
]
data.sort(key=lambda x: x[1])  # ต้อง sort ก่อน

print("\nจัดกลุ่มตามแผนก:")
for dept, employees in itertools.groupby(data, key=lambda x: x[1]):
    emp_list = [e[0] for e in employees]
    print(f"  {dept}: {', '.join(emp_list)}")

# islice() - slice iterable
print("\nislice(range(100), 5, 15, 2):")
for val in itertools.islice(range(100), 5, 15, 2):
    print(val, end=" ")
print()

# takewhile() และ dropwhile()
numbers = [1, 3, 5, 2, 8, 7, 9, 4]
print("\ntakewhile (น้อยกว่า 6):", list(itertools.takewhile(lambda x: x < 6, numbers)))
print("dropwhile (น้อยกว่า 6):", list(itertools.dropwhile(lambda x: x < 6, numbers)))
```

---

## 10. ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Number Patterns

```python
def print_number_patterns():
    """สร้างลวดลายตัวเลขแบบต่างๆ"""
    
    n = 5
    
    print("Pattern 1: สามเหลี่ยมตัวเลขเพิ่มขึ้น")
    for i in range(1, n + 1):
        for j in range(1, i + 1):
            print(j, end=" ")
        print()
    
    print("\nPattern 2: สามเหลี่ยมตัวเลขเดียวกัน")
    for i in range(1, n + 1):
        for j in range(i):
            print(i, end=" ")
        print()
    
    print("\nPattern 3: Floyd's Triangle")
    num = 1
    for i in range(1, n + 1):
        for j in range(i):
            print(f"{num:3}", end="")
            num += 1
        print()
    
    print("\nPattern 4: Pascal's Triangle")
    def pascal_row(n):
        row = [1]
        for k in range(1, n + 1):
            row.append(row[-1] * (n - k + 1) // k)
        return row
    
    for i in range(n):
        row = pascal_row(i)
        spaces = " " * (n - i - 1)
        row_str = " ".join(f"{x:3}" for x in row)
        print(spaces + row_str)
    
    print("\nPattern 5: ตาราง x ยกกำลัง y")
    print("    ", end="")
    for j in range(1, 6):
        print(f"{j:6}", end="")
    print()
    print("    " + "------" * 5)
    for i in range(1, 6):
        print(f"{i:3} |", end="")
        for j in range(1, 6):
            print(f"{i**j:6}", end="")
        print()

print_number_patterns()
```

### โปรแกรมที่ 2: ตารางสูตรคูณแบบ Interactive

```python
def multiplication_table(n=10):
    """สร้างตารางสูตรคูณ"""
    
    # Header
    header_width = 5 * (n + 1) + 3
    print("=" * header_width)
    print(f"{'ตารางสูตรคูณ ' + str(n) + 'x' + str(n):^{header_width}}")
    print("=" * header_width)
    
    # Column headers
    print(f"{'x':>5}", end="")
    for i in range(1, n + 1):
        print(f"{i:5}", end="")
    print()
    print("-" * header_width)
    
    # Rows
    for i in range(1, n + 1):
        print(f"{i:>5}", end="")
        for j in range(1, n + 1):
            result = i * j
            print(f"{result:5}", end="")
        print()
    
    print("=" * header_width)

multiplication_table(10)
```

### โปรแกรมที่ 3: Data Processing System

```python
def analyze_sales_data():
    """วิเคราะห์ข้อมูลการขาย"""
    
    # ข้อมูลการขาย
    sales_data = [
        {"date": "2024-01-01", "product": "Apple", "qty": 50, "price": 25},
        {"date": "2024-01-01", "product": "Banana", "qty": 30, "price": 15},
        {"date": "2024-01-02", "product": "Apple", "qty": 40, "price": 25},
        {"date": "2024-01-02", "product": "Cherry", "qty": 20, "price": 50},
        {"date": "2024-01-03", "product": "Banana", "qty": 60, "price": 15},
        {"date": "2024-01-03", "product": "Apple", "qty": 35, "price": 25},
        {"date": "2024-01-04", "product": "Cherry", "qty": 15, "price": 50},
        {"date": "2024-01-04", "product": "Date", "qty": 25, "price": 40},
        {"date": "2024-01-05", "product": "Apple", "qty": 45, "price": 25},
        {"date": "2024-01-05", "product": "Banana", "qty": 55, "price": 15},
    ]
    
    # คำนวณยอดขายรวม
    total_revenue = 0
    product_revenue = {}
    date_revenue = {}
    
    for sale in sales_data:
        revenue = sale["qty"] * sale["price"]
        total_revenue += revenue
        
        product = sale["product"]
        product_revenue[product] = product_revenue.get(product, 0) + revenue
        
        date = sale["date"]
        date_revenue[date] = date_revenue.get(date, 0) + revenue
    
    print("=" * 50)
    print("รายงานยอดขาย")
    print("=" * 50)
    
    # ยอดขายแต่ละวัน
    print("\nยอดขายรายวัน:")
    for date, revenue in sorted(date_revenue.items()):
        bar = "█" * int(revenue / 200)
        print(f"  {date}: {revenue:>8,} บาท {bar}")
    
    # ยอดขายแต่ละสินค้า
    print("\nยอดขายแต่ละสินค้า:")
    sorted_products = sorted(product_revenue.items(), key=lambda x: x[1], reverse=True)
    for rank, (product, revenue) in enumerate(sorted_products, 1):
        percentage = revenue / total_revenue * 100
        bar = "█" * int(percentage / 2)
        print(f"  {rank}. {product:<10} {revenue:>8,} บาท ({percentage:5.1f}%) {bar}")
    
    print(f"\nยอดขายรวมทั้งหมด: {total_revenue:,} บาท")
    
    # หาวันที่ขายดีที่สุด
    best_date = max(date_revenue, key=date_revenue.get)
    print(f"วันที่ขายดีที่สุด: {best_date} ({date_revenue[best_date]:,} บาท)")
    
    # หาสินค้าขายดีที่สุด
    best_product = max(product_revenue, key=product_revenue.get)
    print(f"สินค้าขายดีที่สุด: {best_product} ({product_revenue[best_product]:,} บาท)")

analyze_sales_data()
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดข้อที่ 1: FizzBuzz

```
จงเขียนโปรแกรม FizzBuzz:
- พิมพ์ตัวเลข 1 ถึง n
- ถ้าหารด้วย 3 ลงตัว พิมพ์ "Fizz"
- ถ้าหารด้วย 5 ลงตัว พิมพ์ "Buzz"
- ถ้าหารด้วยทั้ง 3 และ 5 ลงตัว พิมพ์ "FizzBuzz"
```

**เฉลย:**

```python
def fizzbuzz(n):
    result = []
    for i in range(1, n + 1):
        if i % 15 == 0:
            result.append("FizzBuzz")
        elif i % 3 == 0:
            result.append("Fizz")
        elif i % 5 == 0:
            result.append("Buzz")
        else:
            result.append(str(i))
    return result

output = fizzbuzz(30)
for i, val in enumerate(output, 1):
    print(f"{i:2}: {val}")
```

---

### แบบฝึกหัดข้อที่ 2: หาตัวเลขเฉพาะ (Prime Numbers)

```
จงเขียนโปรแกรมหาจำนวนเฉพาะทั้งหมดที่น้อยกว่าหรือเท่ากับ n
```

**เฉลย:**

```python
def find_primes(n):
    """หาจำนวนเฉพาะด้วย Sieve of Eratosthenes"""
    if n < 2:
        return []
    
    # สร้าง boolean array
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    
    for i in range(2, int(n**0.5) + 1):
        if is_prime[i]:
            for j in range(i*i, n+1, i):
                is_prime[j] = False
    
    return [i for i in range(2, n+1) if is_prime[i]]

primes = find_primes(100)
print(f"จำนวนเฉพาะถึง 100 ({len(primes)} ตัว):")
for i, prime in enumerate(primes, 1):
    print(f"{prime:4}", end=("" if i % 10 != 0 else "\n"))
print()
```

---

### แบบฝึกหัดข้อที่ 3: เรียงตัวเลข Bubble Sort

```
จงเขียน Bubble Sort algorithm โดยใช้ nested for loop
```

**เฉลย:**

```python
def bubble_sort(arr):
    """Bubble Sort - แสดงขั้นตอนทีละรอบ"""
    n = len(arr)
    arr = arr.copy()  # ไม่แก้ไข original
    
    print(f"ก่อนเรียง: {arr}")
    
    for i in range(n - 1):
        swapped = False
        for j in range(n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        
        print(f"รอบที่ {i+1}: {arr}")
        
        if not swapped:  # ถ้าไม่มีการสลับ แสดงว่าเรียงแล้ว
            print("  (เรียงแล้ว หยุดเร็ว)")
            break
    
    return arr

data = [64, 34, 25, 12, 22, 11, 90]
sorted_data = bubble_sort(data)
print(f"หลังเรียง: {sorted_data}")
```

---

### แบบฝึกหัดข้อที่ 4: สถิติข้อมูล

```
จงเขียนโปรแกรมคำนวณสถิติพื้นฐาน:
mean, median, mode, min, max, range จากรายการตัวเลข
```

**เฉลย:**

```python
def calculate_statistics(data):
    """คำนวณสถิติพื้นฐาน"""
    if not data:
        return None
    
    n = len(data)
    sorted_data = sorted(data)
    
    # Mean
    mean = sum(data) / n
    
    # Median
    mid = n // 2
    if n % 2 == 0:
        median = (sorted_data[mid - 1] + sorted_data[mid]) / 2
    else:
        median = sorted_data[mid]
    
    # Mode
    freq = {}
    for val in data:
        freq[val] = freq.get(val, 0) + 1
    max_freq = max(freq.values())
    modes = [k for k, v in freq.items() if v == max_freq]
    
    # Min, Max, Range
    minimum = min(data)
    maximum = max(data)
    data_range = maximum - minimum
    
    # Standard Deviation
    variance = sum((x - mean) ** 2 for x in data) / n
    std_dev = variance ** 0.5
    
    print(f"ข้อมูล: {data}")
    print(f"จำนวน: {n}")
    print(f"ค่าเฉลี่ย (Mean): {mean:.2f}")
    print(f"มัธยฐาน (Median): {median:.2f}")
    print(f"ฐานนิยม (Mode): {modes} (ความถี่: {max_freq})")
    print(f"ค่าต่ำสุด: {minimum}")
    print(f"ค่าสูงสุด: {maximum}")
    print(f"พิสัย (Range): {data_range}")
    print(f"ส่วนเบี่ยงเบนมาตรฐาน: {std_dev:.2f}")

data = [4, 7, 2, 9, 1, 5, 7, 3, 8, 6, 7, 4, 5, 2, 8]
calculate_statistics(data)
```

---

### แบบฝึกหัดข้อที่ 5: สร้าง Word Frequency

```
จงเขียนโปรแกรมนับความถี่ของคำในข้อความ และแสดง Top N คำที่ใช้บ่อยที่สุด
```

**เฉลย:**

```python
def word_frequency(text, top_n=10):
    """นับความถี่ของคำ"""
    import string
    
    # ทำความสะอาดข้อความ
    text = text.lower()
    for char in string.punctuation:
        text = text.replace(char, " ")
    
    words = text.split()
    
    # นับความถี่
    freq = {}
    for word in words:
        if len(word) > 2:  # ข้ามคำสั้น
            freq[word] = freq.get(word, 0) + 1
    
    # เรียงตามความถี่
    sorted_words = sorted(freq.items(), key=lambda x: x[1], reverse=True)
    
    print(f"จำนวนคำทั้งหมด: {len(words)}")
    print(f"คำไม่ซ้ำ: {len(freq)}")
    print(f"\nTop {top_n} คำที่ใช้บ่อย:")
    print(f"{'อันดับ':>6} {'คำ':<20} {'ความถี่':>8} {'แถบ'}")
    print("-" * 50)
    
    for rank, (word, count) in enumerate(sorted_words[:top_n], 1):
        bar = "█" * count
        print(f"{rank:>6}. {word:<20} {count:>8} {bar}")

sample_text = """
Python is an interpreted high-level general-purpose programming language.
Its design philosophy emphasizes code readability with the use of significant indentation.
Python is dynamically typed and garbage-collected. It supports multiple programming paradigms,
including structured procedural object-oriented and functional programming.
Python was created by Guido van Rossum and first released in 1991. Python consistently ranks
as one of the most popular programming languages.
"""

word_frequency(sample_text, 10)
```

---

### แบบฝึกหัดข้อที่ 6: เกมทายตัวเลขแบบให้คำใบ้

```
จงเขียนโปรแกรมทายตัวเลข 0-100 โดยระบบช่วยเดา:
- ให้ผู้ใช้คิดตัวเลขในใจ
- โปรแกรมเดาตัวเลขโดยใช้ Binary Search
- ผู้ใช้ตอบ: higher/lower/correct
```

**เฉลย:**

```python
def binary_search_game():
    """เกม Binary Search ทายตัวเลข"""
    print("=" * 40)
    print("เกมทายตัวเลข (Binary Search)")
    print("=" * 40)
    print("คิดตัวเลขระหว่าง 1-100 ไว้ในใจ")
    print("โปรแกรมจะช่วยเดา")
    print()
    
    low, high = 1, 100
    attempts = 0
    
    guesses_log = []
    
    while low <= high:
        guess = (low + high) // 2
        attempts += 1
        guesses_log.append(guess)
        
        print(f"รอบที่ {attempts}: โปรแกรมเดา {guess}")
        print(f"  (ช่วงที่เหลือ: {low}-{high})")
        answer = input("  ตอบ (h=สูงกว่า, l=ต่ำกว่า, c=ถูกต้อง): ").strip().lower()
        
        if answer == 'c':
            print(f"\nโปรแกรมทายถูก! ตัวเลขคือ {guess}")
            print(f"ใช้ {attempts} ครั้ง")
            print(f"ลำดับการเดา: {' -> '.join(map(str, guesses_log))}")
            return
        elif answer == 'h':
            low = guess + 1
        elif answer == 'l':
            high = guess - 1
        else:
            print("  กรุณาตอบ h, l หรือ c")
            attempts -= 1  # ไม่นับรอบนี้
    
    print("มีบางอย่างผิดพลาด (ตัวเลขต้องอยู่ระหว่าง 1-100)")

# binary_search_game()  # uncomment เพื่อเล่น
print("(จำลองการทำงาน - uncomment เพื่อเล่นจริง)")
```

---

### แบบฝึกหัดข้อที่ 7: Transpose Matrix

```
จงเขียนโปรแกรม transpose matrix (สลับแถวและคอลัมน์)
```

**เฉลย:**

```python
def transpose_matrix(matrix):
    """Transpose matrix"""
    if not matrix:
        return []
    
    rows = len(matrix)
    cols = len(matrix[0])
    
    # สร้าง transposed matrix
    transposed = [[0] * rows for _ in range(cols)]
    
    for i in range(rows):
        for j in range(cols):
            transposed[j][i] = matrix[i][j]
    
    return transposed

def print_matrix(matrix, title="Matrix"):
    print(f"\n{title}:")
    for row in matrix:
        print("  [" + "  ".join(f"{x:3}" for x in row) + "]")

# ทดสอบ
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12]
]

print_matrix(matrix, "Original (3x4)")
transposed = transpose_matrix(matrix)
print_matrix(transposed, "Transposed (4x3)")

# ตรวจสอบ
print(f"\nขนาดเดิม: {len(matrix)}x{len(matrix[0])}")
print(f"ขนาดหลัง transpose: {len(transposed)}x{len(transposed[0])}")
```

---

### แบบฝึกหัดข้อที่ 8: สร้างปฏิทิน

```
จงเขียนโปรแกรมสร้างปฏิทินรายเดือน
```

**เฉลย:**

```python
def print_calendar(year, month):
    """สร้างปฏิทิน"""
    import calendar
    
    months_thai = [
        "", "มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน",
        "พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม",
        "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"
    ]
    
    days_thai = ["จ", "อ", "พ", "พฤ", "ศ", "ส", "อา"]
    
    # หาวันแรกของเดือนและจำนวนวัน
    first_weekday, num_days = calendar.monthrange(year, month)
    
    print(f"\n{'='*30}")
    print(f"  {months_thai[month]} {year + 543} (ค.ศ. {year})")
    print(f"{'='*30}")
    
    # Header วัน
    for day in days_thai:
        print(f"{day:^4}", end="")
    print()
    print("-" * 28)
    
    # เว้นวางสำหรับวันเริ่มต้น
    current_day = 1
    print("    " * first_weekday, end="")
    
    for day in range(1, num_days + 1):
        weekday = (first_weekday + day - 1) % 7
        print(f"{day:>4}", end="")
        if weekday == 6:  # วันอาทิตย์
            print()
    print()
    print("=" * 30)

# แสดงปฏิทิน
print_calendar(2024, 1)
print_calendar(2024, 2)
print_calendar(2024, 12)
```

---

### แบบฝึกหัดข้อที่ 9: Caesar Cipher

```
จงเขียนโปรแกรม Caesar Cipher (การเข้ารหัสโดยเลื่อนตัวอักษร)
encrypt และ decrypt
```

**เฉลย:**

```python
def caesar_cipher(text, shift, mode="encrypt"):
    """Caesar Cipher"""
    result = []
    
    if mode == "decrypt":
        shift = -shift
    
    for char in text:
        if char.isalpha():
            # กำหนด base (A=65, a=97)
            base = ord('A') if char.isupper() else ord('a')
            # เลื่อนและวนกลับด้วย modulo 26
            shifted = (ord(char) - base + shift) % 26 + base
            result.append(chr(shifted))
        else:
            result.append(char)  # ไม่ใช่ตัวอักษรคงเดิม
    
    return "".join(result)

# ทดสอบ
messages = [
    "Hello World",
    "Python is Amazing",
    "The quick brown fox",
]

for msg in messages:
    for shift in [3, 13, 25]:
        encrypted = caesar_cipher(msg, shift, "encrypt")
        decrypted = caesar_cipher(encrypted, shift, "decrypt")
        match = "✓" if decrypted == msg else "✗"
        print(f"Shift {shift:2}: '{msg}' -> '{encrypted}' -> '{decrypted}' {match}")
    print()
```

---

### แบบฝึกหัดข้อที่ 10: ระบบคะแนนนักเรียน

```
จงเขียนโปรแกรมจัดการคะแนนนักเรียน:
- รับรายชื่อและคะแนนจากหลาย subject
- คำนวณ GPA
- แสดง ranking
- หาคนที่ได้คะแนนสูงสุดและต่ำสุดในแต่ละวิชา
```

**เฉลย:**

```python
def student_grade_system():
    """ระบบจัดการคะแนนนักเรียน"""
    
    students = {
        "Alice": {"Math": 92, "Science": 88, "English": 95, "History": 78},
        "Bob": {"Math": 75, "Science": 82, "English": 70, "History": 85},
        "Charlie": {"Math": 98, "Science": 95, "English": 88, "History": 92},
        "Diana": {"Math": 65, "Science": 70, "English": 75, "History": 68},
        "Eve": {"Math": 88, "Science": 90, "English": 85, "History": 87},
    }
    
    subjects = list(next(iter(students.values())).keys())
    
    # คำนวณ GPA แต่ละคน
    gpas = {}
    for name, scores in students.items():
        avg = sum(scores.values()) / len(scores)
        gpas[name] = avg
    
    # แสดงตารางคะแนน
    print("=" * 75)
    print(f"{'ชื่อ':<12}", end="")
    for subject in subjects:
        print(f"{subject:>10}", end="")
    print(f"{'GPA':>10} {'Rank':>6}")
    print("=" * 75)
    
    # เรียงตาม GPA
    ranking = sorted(gpas.items(), key=lambda x: x[1], reverse=True)
    rank_dict = {name: rank for rank, (name, _) in enumerate(ranking, 1)}
    
    for name, scores in students.items():
        print(f"{name:<12}", end="")
        for subject in subjects:
            score = scores[subject]
            print(f"{score:>10}", end="")
        gpa = gpas[name]
        rank = rank_dict[name]
        print(f"{gpa:>10.1f} {rank:>6}")
    
    print("=" * 75)
    
    # หาคะแนนสูงสุดและต่ำสุดแต่ละวิชา
    print("\nสถิติรายวิชา:")
    print(f"{'วิชา':<12} {'สูงสุด':<20} {'ต่ำสุด':<20} {'เฉลี่ย':>8}")
    print("-" * 65)
    
    for subject in subjects:
        subject_scores = [(name, scores[subject]) for name, scores in students.items()]
        best = max(subject_scores, key=lambda x: x[1])
        worst = min(subject_scores, key=lambda x: x[1])
        avg = sum(s for _, s in subject_scores) / len(subject_scores)
        
        print(f"{subject:<12} {best[0]+' ('+str(best[1])+')':<20} {worst[0]+' ('+str(worst[1])+')':<20} {avg:>8.1f}")
    
    # Ranking
    print("\nการจัดอันดับ:")
    for rank, (name, gpa) in enumerate(ranking, 1):
        medal = "🥇" if rank == 1 else "🥈" if rank == 2 else "🥉" if rank == 3 else "  "
        print(f"  {rank}. {medal} {name:<12} GPA: {gpa:.2f}")

student_grade_system()
```

---

## สรุป Part 07

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| for loop พื้นฐาน | วนซ้ำผ่าน iterable |
| Collections | list, dict, set, string, tuple |
| range() | range(n), range(start,stop), range(start,stop,step) |
| enumerate() | ได้ทั้ง index และ value |
| zip() | รวม iterables หลายตัว |
| break/continue/else | ควบคุม flow ใน loop |
| Nested loops | ประมวลผลข้อมูล 2 มิติ |
| List Comprehension | สร้าง list แบบกระชับ |
| itertools | tools สำหรับ iteration ขั้นสูง |

### Key Takeaways:
1. **for loop** ใน Python ทำงานกับ iterable ทุกชนิด
2. ใช้ **enumerate()** แทน `range(len())` เสมอ
3. **zip()** สะดวกมากสำหรับการรวม lists
4. **List Comprehension** ทำให้โค้ดกระชับและ Pythonic
5. **break/continue** ควบคุม flow ใน loop
6. **else** ใน for loop ทำงานเมื่อ loop ไม่ถูก break

---

*Part 07 จบแล้ว ไปต่อที่ [Part 08 - Loops: while Loop](../part08/README.md)*
