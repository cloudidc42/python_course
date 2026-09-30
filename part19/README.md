# Part 19: Lambda, map, filter, reduce & Functional Programming

## สารบัญ
1. [Lambda Functions](#1-lambda-functions)
2. [map() Function](#2-map-function)
3. [filter() Function](#3-filter-function)
4. [reduce() (functools.reduce)](#4-reduce-functoolsreduce)
5. [sorted() with key parameter](#5-sorted-with-key-parameter)
6. [zip() และ unzip](#6-zip-และ-unzip)
7. [any() และ all()](#7-any-และ-all)
8. [enumerate()](#8-enumerate)
9. [Functional Programming Concepts](#9-functional-programming-concepts)
10. [Pure Functions](#10-pure-functions)
11. [Immutability](#11-immutability)
12. [functools Module](#12-functools-module)
13. [operator Module](#13-operator-module)
14. [ตัวอย่างโปรแกรมจริง](#14-ตัวอย่างโปรแกรมจริง)
15. [แบบฝึกหัด](#15-แบบฝึกหัด)

---

## 1. Lambda Functions

Lambda function คือ anonymous function (ฟังก์ชันไม่มีชื่อ) ที่เขียนในบรรทัดเดียว

### รูปแบบพื้นฐาน

```python
# รูปแบบ:
# lambda parameters: expression

# เทียบกับ def
def square(x):
    return x ** 2

square_lambda = lambda x: x ** 2

print(square(5))        # 25
print(square_lambda(5)) # 25
```

```python
# ตัวอย่างที่ 2: lambda กับหลาย parameters
add = lambda x, y: x + y
multiply = lambda x, y, z: x * y * z

print(add(3, 4))         # 7
print(multiply(2, 3, 4)) # 24
```

```python
# ตัวอย่างที่ 3: lambda กับ default parameter
greet = lambda name, greeting="Hello": f"{greeting}, {name}!"
print(greet("Alice"))           # Hello, Alice!
print(greet("Bob", "สวัสดี"))  # สวัสดี, Bob!
```

```python
# ตัวอย่างที่ 4: lambda ใน higher-order functions
def apply(func, value):
    """ใช้ฟังก์ชันกับค่า"""
    return func(value)

print(apply(lambda x: x**2, 5))         # 25
print(apply(lambda x: x.upper(), "hello")) # HELLO
print(apply(lambda x: len(x), "Python"))   # 6
```

```python
# ตัวอย่างที่ 5: lambda กับ conditional expression
classify_number = lambda n: "positive" if n > 0 else "negative" if n < 0 else "zero"

for num in [-3, 0, 5]:
    print(f"{num}: {classify_number(num)}")
```

```python
# ตัวอย่างที่ 6: lambda ใน dictionary
operations = {
    "add": lambda x, y: x + y,
    "subtract": lambda x, y: x - y,
    "multiply": lambda x, y: x * y,
    "divide": lambda x, y: x / y if y != 0 else None,
}

for op_name, op_func in operations.items():
    result = op_func(10, 2)
    print(f"{op_name}(10, 2) = {result}")
```

```python
# ตัวอย่างที่ 7: lambda สำหรับ key extraction
records = [
    {"name": "Charlie", "age": 30},
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 35},
]

sorted_by_name = sorted(records, key=lambda r: r["name"])
sorted_by_age = sorted(records, key=lambda r: r["age"])

print("Sorted by name:")
for r in sorted_by_name:
    print(f"  {r['name']}: {r['age']}")

print("\nSorted by age:")
for r in sorted_by_age:
    print(f"  {r['name']}: {r['age']}")
```

### เมื่อไหร่ควรและไม่ควรใช้ Lambda

```python
# ✅ ควรใช้: ฟังก์ชันง่ายๆ ใช้ครั้งเดียว
items = [1, -3, 2, -1, 4]
positive = list(filter(lambda x: x > 0, items))
print(positive)  # [1, 2, 4]

# ✅ ควรใช้: เป็น key function
pairs = [(2, 'b'), (1, 'a'), (3, 'c')]
sorted_pairs = sorted(pairs, key=lambda p: p[0])
print(sorted_pairs)

# ❌ ไม่ควรใช้: logic ซับซ้อน - ใช้ def แทน
# process = lambda x: x**2 + 2*x + 1 if x > 0 else -x**2  # ยากอ่าน

# ✅ ดีกว่า
def process(x):
    if x > 0:
        return x**2 + 2*x + 1
    return -x**2

# ❌ ไม่ควรใช้: assign lambda ให้ตัวแปร (ใช้ def แทน)
# double = lambda x: x * 2  # ไม่ดี
def double(x):  # ดีกว่า
    return x * 2
```

---

## 2. map() Function

`map()` ใช้ฟังก์ชันกับทุก element ใน iterable

### รูปแบบ

```python
# map(function, iterable)
# map(function, iterable1, iterable2, ...)  # หลาย iterables
```

```python
# ตัวอย่างที่ 1: map พื้นฐาน
numbers = [1, 2, 3, 4, 5]

# แบบ map
squared = list(map(lambda x: x**2, numbers))
print(f"Squared: {squared}")

# แบบ map กับ def function
def double(x):
    return x * 2

doubled = list(map(double, numbers))
print(f"Doubled: {doubled}")
```

```python
# ตัวอย่างที่ 2: map กับ built-in functions
str_numbers = ["1", "2", "3", "4", "5"]
integers = list(map(int, str_numbers))
print(f"Integers: {integers}")

floats = list(map(float, str_numbers))
print(f"Floats: {floats}")

words = ["hello", "world", "python"]
upper_words = list(map(str.upper, words))
print(f"Upper: {upper_words}")
```

```python
# ตัวอย่างที่ 3: map กับ 2 iterables
a = [1, 2, 3, 4, 5]
b = [10, 20, 30, 40, 50]

sums = list(map(lambda x, y: x + y, a, b))
products = list(map(lambda x, y: x * y, a, b))

print(f"Sums: {sums}")
print(f"Products: {products}")

# เทียบกับ zip
sums_zip = [x + y for x, y in zip(a, b)]
print(f"Sums (zip): {sums_zip}")
```

```python
# ตัวอย่างที่ 4: map กับ objects
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
    
    def get_grade(self):
        if self.score >= 90: return "A"
        elif self.score >= 80: return "B"
        elif self.score >= 70: return "C"
        elif self.score >= 60: return "D"
        return "F"

students = [
    Student("Alice", 85),
    Student("Bob", 72),
    Student("Charlie", 91),
]

names = list(map(lambda s: s.name, students))
grades = list(map(lambda s: s.get_grade(), students))
grade_pairs = list(map(lambda s: (s.name, s.get_grade()), students))

print(f"Names: {names}")
print(f"Grades: {grades}")
print(f"Grade pairs: {grade_pairs}")
```

```python
# ตัวอย่างที่ 5: map เปรียบเทียบกับ list comprehension
data = [" Alice ", " Bob ", " Charlie "]

# map
cleaned_map = list(map(str.strip, data))

# list comprehension
cleaned_comp = [s.strip() for s in data]

print(f"Map: {cleaned_map}")
print(f"Comp: {cleaned_comp}")
# ผลเหมือนกัน - แต่ list comprehension อ่านง่ายกว่าในหลายกรณี
```

```python
# ตัวอย่างที่ 6: map pipeline
data = ["  3.14  ", "  2.71  ", "  1.41  "]

# Pipeline: strip -> float -> round -> str
result = list(
    map(
        lambda x: f"{round(x, 1):.1f}",
        map(float, map(str.strip, data))
    )
)
print(f"Pipeline result: {result}")

# แบบอ่านง่ายกว่า
def process_value(s):
    return f"{round(float(s.strip()), 1):.1f}"

result2 = list(map(process_value, data))
print(f"Process result: {result2}")
```

---

## 3. filter() Function

`filter()` กรอง elements ที่ผ่านเงื่อนไข

### รูปแบบ

```python
# filter(function, iterable)
# function ต้อง return True/False
```

```python
# ตัวอย่างที่ 1: filter พื้นฐาน
numbers = range(-5, 6)

positives = list(filter(lambda x: x > 0, numbers))
print(f"Positives: {positives}")

evens = list(filter(lambda x: x % 2 == 0, range(20)))
print(f"Evens: {evens}")
```

```python
# ตัวอย่างที่ 2: filter กับ None (กรอง truthy values)
mixed = [0, 1, "", "hello", None, [], [1, 2], False, True]
truthy = list(filter(None, mixed))
print(f"Truthy values: {truthy}")

# กรอง falsy values
falsy = list(filter(lambda x: not x, mixed))
print(f"Falsy values: {falsy}")
```

```python
# ตัวอย่างที่ 3: filter objects
users = [
    {"name": "Alice", "age": 25, "active": True},
    {"name": "Bob", "age": 17, "active": True},
    {"name": "Charlie", "age": 30, "active": False},
    {"name": "Diana", "age": 22, "active": True},
    {"name": "Eve", "age": 16, "active": False},
]

# Active adult users
active_adults = list(filter(
    lambda u: u["active"] and u["age"] >= 18,
    users
))
print(f"Active adults: {[u['name'] for u in active_adults]}")

# Users ที่ inactive
inactive = list(filter(lambda u: not u["active"], users))
print(f"Inactive: {[u['name'] for u in inactive]}")
```

```python
# ตัวอย่างที่ 4: filter กับ string
words = ["python", "java", "javascript", "ruby", "perl", "php", "go"]

# คำที่ขึ้นต้นด้วย 'p'
p_words = list(filter(lambda w: w.startswith('p'), words))
print(f"P words: {p_words}")

# คำที่ยาวกว่า 4 ตัวอักษร
long_words = list(filter(lambda w: len(w) > 4, words))
print(f"Long words: {long_words}")
```

```python
# ตัวอย่างที่ 5: รวม filter และ map
data = [1, -2, 3, -4, 5, -6, 7, -8, 9, -10]

# กรองบวก แล้ว square
positive_squares = list(
    map(lambda x: x**2, filter(lambda x: x > 0, data))
)
print(f"Positive squares: {positive_squares}")

# เทียบกับ list comprehension (อ่านง่ายกว่า)
positive_squares_comp = [x**2 for x in data if x > 0]
print(f"Using comprehension: {positive_squares_comp}")
```

---

## 4. reduce() (functools.reduce)

`reduce()` ลด iterable เป็นค่าเดียวโดยใช้ฟังก์ชันสะสม

```python
from functools import reduce

# ตัวอย่างที่ 1: sum ด้วย reduce
numbers = [1, 2, 3, 4, 5]

total = reduce(lambda acc, x: acc + x, numbers)
print(f"Total: {total}")  # 15

# เทียบกับ sum() built-in
print(f"sum(): {sum(numbers)}")  # 15

# reduce ที่แสดง step
def show_reduce(acc, x):
    result = acc + x
    print(f"  {acc} + {x} = {result}")
    return result

print("Steps:")
total = reduce(show_reduce, numbers)
```

```python
# ตัวอย่างที่ 2: product ด้วย reduce
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda acc, x: acc * x, numbers)
print(f"Product: {product}")  # 120

# เทียบ
import math
print(f"math.prod(): {math.prod(numbers)}")
```

```python
# ตัวอย่างที่ 3: max/min ด้วย reduce
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

max_val = reduce(lambda acc, x: acc if acc > x else x, numbers)
min_val = reduce(lambda acc, x: acc if acc < x else x, numbers)

print(f"Max: {max_val}")  # 9
print(f"Min: {min_val}")  # 1
```

```python
# ตัวอย่างที่ 4: reduce กับ initial value
numbers = [1, 2, 3, 4, 5]

# initial value = 100
result = reduce(lambda acc, x: acc + x, numbers, 100)
print(f"Sum starting from 100: {result}")  # 115

# กรณี empty list (จำเป็นต้องมี initial value)
empty = []
try:
    result_empty = reduce(lambda acc, x: acc + x, empty)
except TypeError as e:
    print(f"Error: {e}")

result_empty = reduce(lambda acc, x: acc + x, empty, 0)
print(f"Empty list sum: {result_empty}")  # 0
```

```python
# ตัวอย่างที่ 5: flatten ด้วย reduce
nested = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
flat = reduce(lambda acc, x: acc + x, nested, [])
print(f"Flat: {flat}")

# สร้าง sentence จาก words
words = ["Python", "is", "a", "great", "language"]
sentence = reduce(lambda acc, w: acc + " " + w, words)
print(f"Sentence: {sentence}")
```

```python
# ตัวอย่างที่ 6: compose functions ด้วย reduce
def compose(*functions):
    """รวมหลาย functions เป็นหนึ่งเดียว"""
    return reduce(lambda f, g: lambda x: f(g(x)), functions)

# ฟังก์ชันต่างๆ
add_one = lambda x: x + 1
double = lambda x: x * 2
square = lambda x: x ** 2

# compose จากขวาไปซ้าย: square -> double -> add_one
transform = compose(add_one, double, square)
result = transform(3)  # add_one(double(square(3))) = add_one(double(9)) = add_one(18) = 19
print(f"compose(add_one, double, square)(3) = {result}")
```

---

## 5. sorted() with key parameter

```python
# sorted() vs list.sort()
# sorted() - สร้าง list ใหม่ (ไม่เปลี่ยนต้นฉบับ)
# list.sort() - เปลี่ยน in-place (ไม่ return ค่า)

numbers = [3, 1, 4, 1, 5, 9, 2, 6]
sorted_nums = sorted(numbers)  # สร้างใหม่
numbers.sort()                  # เปลี่ยน in-place

print(f"sorted(): {sorted_nums}")
print(f"after sort(): {numbers}")
```

```python
# ตัวอย่างที่ 2: sort กับ key function
words = ["banana", "apple", "cherry", "date", "elderberry"]

# Sort โดย length
by_length = sorted(words, key=len)
print(f"By length: {by_length}")

# Sort โดย last character
by_last = sorted(words, key=lambda w: w[-1])
print(f"By last char: {by_last}")

# Sort reverse
by_length_rev = sorted(words, key=len, reverse=True)
print(f"By length (desc): {by_length_rev}")
```

```python
# ตัวอย่างที่ 3: sort objects
students = [
    {"name": "Charlie", "grade": 85, "age": 22},
    {"name": "Alice", "grade": 92, "age": 20},
    {"name": "Bob", "grade": 85, "age": 21},
    {"name": "Diana", "grade": 78, "age": 23},
]

# Sort โดย grade (high to low)
by_grade = sorted(students, key=lambda s: s["grade"], reverse=True)
print("By grade (desc):")
for s in by_grade:
    print(f"  {s['name']}: {s['grade']}")

# Sort หลาย keys: grade ก่อน แล้ว age
by_grade_age = sorted(students, key=lambda s: (-s["grade"], s["age"]))
print("\nBy grade (desc) then age (asc):")
for s in by_grade_age:
    print(f"  {s['name']}: grade={s['grade']}, age={s['age']}")
```

```python
# ตัวอย่างที่ 4: sort ด้วย operator.itemgetter (เร็วกว่า lambda)
from operator import itemgetter, attrgetter

data = [("Alice", 25), ("Bob", 30), ("Charlie", 22)]

# ใช้ itemgetter
sorted_by_age = sorted(data, key=itemgetter(1))
print(f"By age: {sorted_by_age}")

# sort list of dicts
dicts = [{"name": "Charlie", "age": 30}, {"name": "Alice", "age": 25}]
sorted_dicts = sorted(dicts, key=itemgetter("name"))
print(f"By name: {sorted_dicts}")
```

```python
# ตัวอย่างที่ 5: stable sort
# Python's sort เป็น stable sort - ถ้า key เท่ากัน จะรักษา order เดิม
data = [
    ("Alice", 25), ("Bob", 25), ("Charlie", 30),
    ("Diana", 25), ("Eve", 30)
]

sorted_data = sorted(data, key=lambda x: x[1])
print("Sorted (stable):")
for item in sorted_data:
    print(f"  {item}")
# Alice, Bob, Diana จะยังอยู่ในลำดับเดิม (age=25)
```

---

## 6. zip() และ unzip

```python
# zip() รวม iterables เป็น pairs/tuples
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
cities = ["Bangkok", "London", "Paris"]

# zip 2 iterables
paired = list(zip(names, ages))
print(f"Paired: {paired}")

# zip 3 iterables
tripled = list(zip(names, ages, cities))
print(f"Tripled: {tripled}")
```

```python
# ตัวอย่างที่ 2: zip กับ different lengths
list1 = [1, 2, 3, 4, 5]
list2 = ['a', 'b', 'c']

# zip หยุดที่ iterable ที่สั้นที่สุด
result = list(zip(list1, list2))
print(f"zip (shortest): {result}")  # [(1, 'a'), (2, 'b'), (3, 'c')]

# zip_longest - ใช้ค่า default
from itertools import zip_longest
result_longest = list(zip_longest(list1, list2, fillvalue="X"))
print(f"zip_longest: {result_longest}")
```

```python
# ตัวอย่างที่ 3: unzip (zip กับ *)
pairs = [(1, 'a'), (2, 'b'), (3, 'c')]

# unzip
numbers, letters = zip(*pairs)
print(f"Numbers: {numbers}")
print(f"Letters: {letters}")
```

```python
# ตัวอย่างที่ 4: zip สำหรับ parallel iteration
keys = ["name", "age", "email"]
values = ["Alice", 25, "alice@example.com"]

record = dict(zip(keys, values))
print(f"Record: {record}")

# สร้าง dictionary จาก 2 lists
d = {k: v for k, v in zip(keys, values)}
print(f"Dict: {d}")
```

```python
# ตัวอย่างที่ 5: zip สำหรับ matrix operations
matrix_a = [[1, 2, 3], [4, 5, 6]]
matrix_b = [[7, 8, 9], [10, 11, 12]]

# บวก matrices ด้วย zip
matrix_sum = [
    [a + b for a, b in zip(row_a, row_b)]
    for row_a, row_b in zip(matrix_a, matrix_b)
]
print(f"Matrix sum: {matrix_sum}")

# Dot product ของ 2 vectors
v1 = [1, 2, 3, 4]
v2 = [5, 6, 7, 8]
dot_product = sum(a * b for a, b in zip(v1, v2))
print(f"Dot product: {dot_product}")
```

```python
# ตัวอย่างที่ 6: zip สำหรับ sliding window
data = [1, 2, 3, 4, 5, 6, 7, 8]
window_size = 3

# Sliding window ด้วย zip
windows = list(zip(*[data[i:] for i in range(window_size)]))
print(f"Sliding windows (size {window_size}): {windows}")

# Moving average
moving_avg = [sum(w) / window_size for w in windows]
print(f"Moving average: {moving_avg}")
```

---

## 7. any() และ all()

```python
# any() - True ถ้ามีอย่างน้อย 1 element เป็น truthy
# all() - True ถ้าทุก elements เป็น truthy

# พื้นฐาน
nums = [1, 2, 3, 4, 5]
print(f"any([1,2,3]): {any(nums)}")   # True
print(f"all([1,2,3]): {all(nums)}")   # True

mixed = [0, 1, 2, 3]
print(f"any([0,1,2]): {any(mixed)}")  # True (มี 1, 2, 3)
print(f"all([0,1,2]): {all(mixed)}")  # False (มี 0)

empty = []
print(f"any([]): {any(empty)}")  # False
print(f"all([]): {all(empty)}")  # True (vacuous truth)
```

```python
# ตัวอย่างที่ 2: any/all กับ generator
numbers = [10, 20, 30, -5, 40]

# ตรวจสอบว่ามีค่าลบหรือไม่
has_negative = any(n < 0 for n in numbers)
print(f"มีค่าลบ: {has_negative}")

# ตรวจสอบว่าทุกค่าเป็นบวก
all_positive = all(n > 0 for n in numbers)
print(f"ทุกค่าเป็นบวก: {all_positive}")
```

```python
# ตัวอย่างที่ 3: any/all กับ string
def validate_password(password):
    """ตรวจสอบรหัสผ่าน"""
    checks = {
        "length >= 8": len(password) >= 8,
        "has uppercase": any(c.isupper() for c in password),
        "has lowercase": any(c.islower() for c in password),
        "has digit": any(c.isdigit() for c in password),
        "has special": any(c in "!@#$%^&*" for c in password),
    }
    
    for check, passed in checks.items():
        status = "✓" if passed else "✗"
        print(f"  {status} {check}")
    
    return all(checks.values())

passwords = ["python", "Python1", "Python1!", "Py1!"]
for pwd in passwords:
    print(f"\nPassword: '{pwd}'")
    valid = validate_password(pwd)
    print(f"Valid: {valid}")
```

```python
# ตัวอย่างที่ 4: any/all กับ lists of dicts
employees = [
    {"name": "Alice", "has_insurance": True, "has_retirement": True},
    {"name": "Bob", "has_insurance": True, "has_retirement": False},
    {"name": "Charlie", "has_insurance": False, "has_retirement": True},
]

# มีพนักงานที่มีครบทุกสวัสดิการหรือไม่
any_full_benefits = any(
    emp["has_insurance"] and emp["has_retirement"]
    for emp in employees
)
print(f"มีพนักงานที่มีครบ: {any_full_benefits}")

# พนักงานทุกคนมีประกันหรือไม่
all_insured = all(emp["has_insurance"] for emp in employees)
print(f"ทุกคนมีประกัน: {all_insured}")
```

---

## 8. enumerate()

```python
# enumerate() เพิ่ม index ให้กับ iterable

# แบบ loop ปกติ
fruits = ["apple", "banana", "cherry"]
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")

# แบบ enumerate (ดีกว่า)
print()
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
```

```python
# ตัวอย่างที่ 2: enumerate กับ start parameter
for i, fruit in enumerate(fruits, start=1):  # เริ่มจาก 1
    print(f"{i}. {fruit}")
```

```python
# ตัวอย่างที่ 3: enumerate กับ zip
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]

for i, (name, score) in enumerate(zip(names, scores), start=1):
    print(f"{i}. {name}: {score}")
```

```python
# ตัวอย่างที่ 4: enumerate สร้าง dict
colors = ["red", "green", "blue"]

# สร้าง color->index mapping
color_to_index = {color: i for i, color in enumerate(colors)}
index_to_color = {i: color for i, color in enumerate(colors)}

print(f"Color to index: {color_to_index}")
print(f"Index to color: {index_to_color}")
```

```python
# ตัวอย่างที่ 5: enumerate หา position
text = "Hello World Python"
words = text.split()

# หา index ของ word ที่ขึ้นต้นด้วย 'P'
p_positions = [i for i, word in enumerate(words) if word.startswith('P')]
print(f"Words starting with P at positions: {p_positions}")

# หา position ของ element ที่ match condition
numbers = [10, 25, 5, 35, 15, 40]
first_above_30 = next(
    (i for i, n in enumerate(numbers) if n > 30),
    None
)
print(f"First number > 30 at index: {first_above_30}")
```

---

## 9. Functional Programming Concepts

Functional Programming (FP) เป็นแนวคิดการเขียนโปรแกรมที่:
- ใช้ functions เป็นหน่วยพื้นฐาน
- หลีกเลี่ยง shared state และ side effects
- เน้น data transformation

```python
# หลักการ FP หลัก:
# 1. Pure Functions - ผลลัพธ์ขึ้นอยู่กับ input เท่านั้น
# 2. Immutability - ไม่เปลี่ยนแปลงข้อมูลต้นฉบับ
# 3. First-class Functions - functions เป็น values
# 4. Higher-order Functions - functions ที่รับ/return functions
# 5. Function Composition - รวม functions เป็นหนึ่ง

# First-class Functions
def add(x, y):
    return x + y

# ส่ง function เป็น argument
def apply(func, a, b):
    return func(a, b)

print(apply(add, 3, 4))  # 7

# เก็บ function ใน variable
my_func = add
print(my_func(5, 6))  # 11

# เก็บ functions ใน list
math_ops = [
    lambda x, y: x + y,
    lambda x, y: x - y,
    lambda x, y: x * y,
    lambda x, y: x / y if y != 0 else None,
]

for op in math_ops:
    print(op(10, 3))
```

### Higher-order Functions

```python
# Higher-order functions รับหรือ return functions

# ตัวอย่างที่ 1: function factory
def multiplier(factor):
    """สร้าง function ที่คูณด้วย factor"""
    return lambda x: x * factor

double = multiplier(2)
triple = multiplier(3)
hundred = multiplier(100)

print(double(5))   # 10
print(triple(5))   # 15
print(hundred(5))  # 500
```

```python
# ตัวอย่างที่ 2: decorator pattern (HOF)
def timer(func):
    """วัดเวลาการทำงานของ function"""
    import time
    
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} ใช้เวลา {(end-start)*1000:.2f} ms")
        return result
    
    return wrapper

@timer
def slow_computation(n):
    return sum(i**2 for i in range(n))

result = slow_computation(100000)
print(f"Result: {result}")
```

```python
# ตัวอย่างที่ 3: currying
def curry(func):
    """แปลง function เป็น curried function"""
    import inspect
    
    def curried(*args):
        params = inspect.signature(func).parameters
        if len(args) >= len(params):
            return func(*args)
        return lambda *more_args: curried(*(args + more_args))
    
    return curried

@curry
def add_three(a, b, c):
    return a + b + c

# ใช้งาน
print(add_three(1, 2, 3))     # 6
print(add_three(1)(2)(3))     # 6
print(add_three(1, 2)(3))     # 6

add_5 = add_three(5)
add_5_6 = add_5(6)
print(add_5_6(7))             # 18
```

---

## 10. Pure Functions

Pure function คือ function ที่:
1. ผลลัพธ์ขึ้นอยู่กับ input เท่านั้น (deterministic)
2. ไม่มี side effects (ไม่เปลี่ยน state ภายนอก)

```python
# ✅ Pure function
def add_pure(a, b):
    return a + b  # ขึ้นอยู่กับ a, b เท่านั้น

def calculate_tax_pure(income, rate):
    return income * rate  # ไม่มี side effects

# ✅ Pure - คืนค่าใหม่ ไม่เปลี่ยนต้นฉบับ
def append_item_pure(lst, item):
    return lst + [item]  # สร้าง list ใหม่

original = [1, 2, 3]
new_list = append_item_pure(original, 4)
print(f"Original: {original}")  # ไม่เปลี่ยน
print(f"New: {new_list}")
```

```python
# ❌ Impure functions - มี side effects
counter = 0  # global state

def increment_impure():
    global counter
    counter += 1  # เปลี่ยน global state!
    return counter

def log_and_add(a, b):
    print(f"Adding {a} + {b}")  # side effect: print
    return a + b

def get_current_time():
    from datetime import datetime
    return datetime.now()  # ผลลัพธ์เปลี่ยนแต่ละครั้ง

# ✅ Pure versions
def add_with_counter(a, b, current_count):
    return a + b, current_count + 1

count = 0
result, count = add_with_counter(3, 4, count)
print(f"Result: {result}, Count: {count}")
```

```python
# Pure functions ทดสอบง่ายกว่า
def calculate_bmi(weight_kg, height_m):
    """คำนวณ BMI"""
    return weight_kg / (height_m ** 2)

def classify_bmi(bmi):
    """จัดประเภท BMI"""
    if bmi < 18.5: return "Underweight"
    elif bmi < 25.0: return "Normal"
    elif bmi < 30.0: return "Overweight"
    else: return "Obese"

# ทดสอบง่าย - deterministic
test_cases = [
    (50, 1.70),
    (65, 1.70),
    (85, 1.70),
    (100, 1.70),
]

for weight, height in test_cases:
    bmi = calculate_bmi(weight, height)
    category = classify_bmi(bmi)
    print(f"{weight}kg/{height}m: BMI={bmi:.1f} ({category})")
```

---

## 11. Immutability

Immutability หมายถึงการไม่เปลี่ยนแปลงข้อมูลหลังสร้าง

```python
# Python มี immutable types: int, float, str, tuple, frozenset

# ✅ Immutable
x = 5
y = x  # y คือค่าเดิม
x = 10  # สร้าง int ใหม่
print(f"x: {x}, y: {y}")  # x=10, y=5 (แยกกัน)

# ✅ Strings เป็น immutable
s = "hello"
new_s = s.upper()  # สร้าง string ใหม่
print(f"Original: {s}, New: {new_s}")

# ✅ Tuple เป็น immutable
t = (1, 2, 3)
# t[0] = 10  # TypeError!
new_t = t + (4,)  # สร้าง tuple ใหม่
print(f"Original: {t}, New: {new_t}")
```

```python
# ❌ Mutable - แชร์ reference
a = [1, 2, 3]
b = a          # b ชี้ไปที่ list เดียวกัน!
b.append(4)
print(f"a: {a}")  # [1, 2, 3, 4] เปลี่ยนด้วย!
print(f"b: {b}")  # [1, 2, 3, 4]

# ✅ Copy อย่างถูกต้อง
a = [1, 2, 3]
b = a.copy()   # shallow copy
c = a[:]       # shallow copy
import copy
d = copy.deepcopy(a)  # deep copy (สำหรับ nested)

b.append(4)
print(f"a: {a}")  # [1, 2, 3] ไม่เปลี่ยน
print(f"b: {b}")  # [1, 2, 3, 4]
```

```python
# Pattern: สร้าง data ใหม่แทนการเปลี่ยนแปลง

# ❌ Mutable approach
def add_item_mutable(cart, item):
    cart.append(item)  # เปลี่ยน cart เดิม!
    return cart

# ✅ Immutable approach
def add_item_immutable(cart, item):
    return cart + [item]  # สร้าง cart ใหม่

cart = ["apple", "banana"]
new_cart = add_item_immutable(cart, "cherry")
print(f"Original cart: {cart}")    # ไม่เปลี่ยน
print(f"New cart: {new_cart}")

# ✅ ใช้ tuple สำหรับ immutable data
def add_item_tuple(cart, item):
    return cart + (item,)  # tuple immutable

cart_tuple = ("apple", "banana")
new_cart_tuple = add_item_tuple(cart_tuple, "cherry")
print(f"Original: {cart_tuple}")
print(f"New: {new_cart_tuple}")
```

---

## 12. functools Module

```python
import functools

# 1. partial() - สร้าง partial function
def power(base, exponent):
    return base ** exponent

square = functools.partial(power, exponent=2)
cube = functools.partial(power, exponent=3)

print(f"square(5) = {square(5)}")  # 25
print(f"cube(3) = {cube(3)}")     # 27

# partial กับ built-ins
from functools import partial

# สร้าง logger ที่กำหนด prefix
def log_message(level, prefix, message):
    print(f"[{level}] {prefix}: {message}")

error_log = partial(log_message, "ERROR", "APP")
info_log = partial(log_message, "INFO", "APP")

error_log("Connection failed")
info_log("Server started")
```

```python
# 2. lru_cache() - cache ผลลัพธ์
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# ครั้งแรกคำนวณจริง
print(fibonacci(30))  # 832040
print(fibonacci(40))  # 102334155

# ดู cache info
print(fibonacci.cache_info())

# เปรียบเทียบ performance
import time

def fib_no_cache(n):
    if n < 2:
        return n
    return fib_no_cache(n - 1) + fib_no_cache(n - 2)

start = time.perf_counter()
fibonacci(35)  # cached
cached_time = time.perf_counter() - start

start = time.perf_counter()
fib_no_cache(30)  # ไม่ cache (เล็กกว่าเพื่อไม่ให้นานเกิน)
no_cache_time = time.perf_counter() - start

print(f"With cache: {cached_time*1000:.3f} ms")
print(f"No cache: {no_cache_time*1000:.3f} ms")
```

```python
# 3. reduce()
from functools import reduce

# สร้าง pipeline ด้วย reduce
def pipeline(*functions):
    """รวม functions เป็น pipeline"""
    def apply(value, func):
        return func(value)
    return lambda x: reduce(apply, functions, x)

# สร้าง text processing pipeline
normalize = pipeline(
    str.strip,
    str.lower,
    lambda s: s.replace("  ", " "),
)

texts = ["  Hello World  ", "  PYTHON IS GREAT  "]
for text in texts:
    print(f"'{text}' -> '{normalize(text)}'")
```

```python
# 4. wraps() - preserve function metadata
from functools import wraps

def my_decorator(func):
    @wraps(func)  # ถ้าไม่มี @wraps metadata จะหาย
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    """ทักทายผู้ใช้"""
    return f"Hello, {name}!"

print(greet("Alice"))
print(f"Function name: {greet.__name__}")  # greet (ไม่ใช่ wrapper)
print(f"Docstring: {greet.__doc__}")       # ทักทายผู้ใช้
```

```python
# 5. total_ordering - เพิ่ม comparison methods
from functools import total_ordering

@total_ordering
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
    
    def __eq__(self, other):
        return self.celsius == other.celsius
    
    def __lt__(self, other):
        return self.celsius < other.celsius
    
    # total_ordering จะสร้าง __le__, __gt__, __ge__ ให้อัตโนมัติ
    
    def __repr__(self):
        return f"Temperature({self.celsius}°C)"

temps = [Temperature(30), Temperature(20), Temperature(25)]
sorted_temps = sorted(temps)
print(f"Sorted: {sorted_temps}")
print(f"Max: {max(temps)}")
print(f"Min: {min(temps)}")
```

---

## 13. operator Module

```python
import operator

# operator module มี functions สำหรับ built-in operators

# ตัวอย่างที่ 1: arithmetic operators
a, b = 10, 3

print(f"add: {operator.add(a, b)}")       # 13
print(f"sub: {operator.sub(a, b)}")       # 7
print(f"mul: {operator.mul(a, b)}")       # 30
print(f"truediv: {operator.truediv(a, b):.2f}")  # 3.33
print(f"floordiv: {operator.floordiv(a, b)}")   # 3
print(f"mod: {operator.mod(a, b)}")       # 1
print(f"pow: {operator.pow(a, b)}")       # 1000
```

```python
# ตัวอย่างที่ 2: comparison operators
print(f"eq: {operator.eq(5, 5)}")         # True
print(f"ne: {operator.ne(5, 6)}")         # True
print(f"lt: {operator.lt(3, 5)}")         # True
print(f"le: {operator.le(5, 5)}")         # True
print(f"gt: {operator.gt(7, 5)}")         # True
print(f"ge: {operator.ge(5, 5)}")         # True
```

```python
# ตัวอย่างที่ 3: itemgetter และ attrgetter
from operator import itemgetter, attrgetter

# itemgetter สำหรับ dict/list
records = [
    {"name": "Charlie", "score": 85},
    {"name": "Alice", "score": 92},
    {"name": "Bob", "score": 78},
]

# sort โดยใช้ itemgetter (เร็วกว่า lambda)
sorted_by_score = sorted(records, key=itemgetter("score"), reverse=True)
for r in sorted_by_score:
    print(f"  {r['name']}: {r['score']}")

# itemgetter หลาย keys
get_name_score = itemgetter("name", "score")
for r in records:
    print(get_name_score(r))  # ('Charlie', 85)
```

```python
# ตัวอย่างที่ 4: attrgetter สำหรับ objects
class Product:
    def __init__(self, name, price, category):
        self.name = name
        self.price = price
        self.category = category
    
    def __repr__(self):
        return f"Product({self.name}, {self.price})"

products = [
    Product("Widget", 9.99, "Electronics"),
    Product("Gadget", 29.99, "Electronics"),
    Product("Tool", 14.99, "Tools"),
    Product("Gear", 5.99, "Tools"),
]

# sort โดย price
sorted_by_price = sorted(products, key=attrgetter("price"))
print("Sorted by price:")
for p in sorted_by_price:
    print(f"  {p.name}: ${p.price}")

# sort หลาย attributes
sorted_multi = sorted(products, key=attrgetter("category", "price"))
print("\nSorted by category, price:")
for p in sorted_multi:
    print(f"  {p.category}/{p.name}: ${p.price}")
```

```python
# ตัวอย่างที่ 5: ใช้ operator กับ reduce
from functools import reduce
import operator

numbers = [1, 2, 3, 4, 5]

# sum
total = reduce(operator.add, numbers)
print(f"Sum: {total}")

# product
product = reduce(operator.mul, numbers)
print(f"Product: {product}")

# max
maximum = reduce(operator.gt.__call__ or max, numbers)  # ใช้ max แทน
maximum = reduce(lambda a, b: a if a > b else b, numbers)
print(f"Max: {maximum}")
```

---

## 14. ตัวอย่างโปรแกรมจริง

### 14.1 Data Processing Pipeline

```python
from functools import reduce, partial
from operator import itemgetter
import statistics

# ข้อมูลดิบ
raw_sales_data = [
    {"product": "Widget A", "region": "North", "amount": 1200, "units": 10},
    {"product": "Widget B", "region": "South", "amount": 800, "units": 8},
    {"product": "Widget A", "region": "South", "amount": 1500, "units": 12},
    {"product": "Gadget X", "region": "North", "amount": 3000, "units": 5},
    {"product": "Widget B", "region": "North", "amount": 600, "units": 6},
    {"product": "Gadget X", "region": "South", "amount": 2500, "units": 4},
    {"product": "Widget A", "region": "North", "amount": 900, "units": 7},
]

# Pipeline functions
def filter_by_region(region):
    return partial(filter, lambda x: x["region"] == region)

def select_fields(*fields):
    """เลือกเฉพาะ fields ที่ต้องการ"""
    return partial(map, lambda x: {f: x[f] for f in fields})

def calculate_revenue_per_unit(records):
    """คำนวณราคาต่อหน่วย"""
    return map(
        lambda r: {**r, "price_per_unit": r["amount"] / r["units"]},
        records
    )

def group_and_sum(records, key, value_key):
    """จัดกลุ่มและรวม"""
    groups = {}
    for record in records:
        group = record[key]
        if group not in groups:
            groups[group] = 0
        groups[group] += record[value_key]
    return groups

# สร้าง pipeline
def create_sales_report(data, region=None):
    """สร้าง sales report"""
    
    # Step 1: กรองตาม region (ถ้ากำหนด)
    if region:
        filtered = list(filter(lambda x: x["region"] == region, data))
    else:
        filtered = data
    
    # Step 2: คำนวณ metrics
    with_metrics = list(calculate_revenue_per_unit(filtered))
    
    # Step 3: Group by product
    by_product = group_and_sum(with_metrics, "product", "amount")
    
    # Step 4: สถิติ
    amounts = [r["amount"] for r in filtered]
    
    report = {
        "region": region or "All",
        "total_records": len(filtered),
        "total_revenue": sum(amounts),
        "avg_revenue": statistics.mean(amounts) if amounts else 0,
        "top_product": max(by_product, key=by_product.get) if by_product else None,
        "revenue_by_product": dict(sorted(by_product.items(), key=itemgetter(1), reverse=True)),
    }
    
    return report

# รัน pipeline
print("=== Sales Report ===")
for region in [None, "North", "South"]:
    report = create_sales_report(raw_sales_data, region)
    print(f"\nRegion: {report['region']}")
    print(f"  Total Records: {report['total_records']}")
    print(f"  Total Revenue: ฿{report['total_revenue']:,}")
    print(f"  Avg Revenue: ฿{report['avg_revenue']:,.0f}")
    print(f"  Top Product: {report['top_product']}")
    print(f"  Revenue by Product:")
    for product, revenue in report['revenue_by_product'].items():
        print(f"    {product}: ฿{revenue:,}")
```

### 14.2 Sorting Algorithms (Functional Style)

```python
from functools import reduce

def quicksort_functional(lst):
    """Quicksort แบบ functional"""
    if len(lst) <= 1:
        return lst
    
    pivot = lst[len(lst) // 2]
    left = list(filter(lambda x: x < pivot, lst))
    middle = list(filter(lambda x: x == pivot, lst))
    right = list(filter(lambda x: x > pivot, lst))
    
    return quicksort_functional(left) + middle + quicksort_functional(right)

def mergesort_functional(lst):
    """Mergesort แบบ functional"""
    if len(lst) <= 1:
        return lst
    
    mid = len(lst) // 2
    left = mergesort_functional(lst[:mid])
    right = mergesort_functional(lst[mid:])
    
    return merge(left, right)

def merge(left, right):
    """Merge 2 sorted lists"""
    result = []
    i, j = 0, 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    return result + left[i:] + right[j:]

import random
data = [random.randint(1, 100) for _ in range(20)]
print(f"Original: {data}")
print(f"Quicksort: {quicksort_functional(data)}")
print(f"Mergesort: {mergesort_functional(data)}")
print(f"Built-in sort: {sorted(data)}")
```

---

## 15. แบบฝึกหัด

### ข้อที่ 1: Lambda Calculator

```python
# เฉลย
import math

math_functions = {
    "sin": lambda x: math.sin(math.radians(x)),
    "cos": lambda x: math.cos(math.radians(x)),
    "tan": lambda x: math.tan(math.radians(x)),
    "sqrt": lambda x: math.sqrt(x) if x >= 0 else None,
    "abs": lambda x: abs(x),
    "factorial": lambda x: math.factorial(int(x)) if x >= 0 else None,
    "log2": lambda x: math.log2(x) if x > 0 else None,
    "log10": lambda x: math.log10(x) if x > 0 else None,
}

test_inputs = [
    ("sin", 30), ("cos", 60), ("sqrt", 16),
    ("factorial", 5), ("log2", 8), ("sqrt", -4)
]

for func_name, value in test_inputs:
    func = math_functions[func_name]
    result = func(value)
    result_str = f"{result:.4f}" if isinstance(result, float) else str(result)
    print(f"{func_name}({value}) = {result_str}")
```

### ข้อที่ 2: map/filter/reduce pipeline

```python
# เฉลย: ประมวลผล transaction data
transactions = [
    {"id": "T001", "amount": 150.0, "type": "credit", "category": "food"},
    {"id": "T002", "amount": 500.0, "type": "debit", "category": "electronics"},
    {"id": "T003", "amount": 75.0, "type": "credit", "category": "food"},
    {"id": "T004", "amount": 1200.0, "type": "debit", "category": "rent"},
    {"id": "T005", "amount": 89.99, "type": "credit", "category": "clothing"},
    {"id": "T006", "amount": 45.0, "type": "debit", "category": "food"},
]

from functools import reduce

# 1. กรองเฉพาะ credit transactions
credits = list(filter(lambda t: t["type"] == "credit", transactions))

# 2. แปลงเป็น amounts
credit_amounts = list(map(lambda t: t["amount"], credits))

# 3. หา total ด้วย reduce
total_credits = reduce(lambda acc, x: acc + x, credit_amounts, 0)

# 4. กรองเฉพาะ debits และหา total
debits = filter(lambda t: t["type"] == "debit", transactions)
total_debits = reduce(lambda acc, t: acc + t["amount"], debits, 0)

print(f"Total credits: ฿{total_credits:.2f}")
print(f"Total debits: ฿{total_debits:.2f}")
print(f"Net: ฿{total_credits - total_debits:.2f}")

# 5. จัดกลุ่มตาม category
category_totals = {}
for t in transactions:
    cat = t["category"]
    if cat not in category_totals:
        category_totals[cat] = 0
    category_totals[cat] += t["amount"]

print("\nSpending by category:")
for cat, total in sorted(category_totals.items(), key=lambda x: x[1], reverse=True):
    print(f"  {cat}: ฿{total:.2f}")
```

### ข้อที่ 3: Function Composition

```python
# เฉลย
from functools import reduce

def compose(*functions):
    """Compose functions (right to left)"""
    def composed(x):
        return reduce(lambda v, f: f(v), reversed(functions), x)
    return composed

def pipe(*functions):
    """Pipe functions (left to right)"""
    def piped(x):
        return reduce(lambda v, f: f(v), functions, x)
    return piped

# สร้าง text processing pipeline
text_pipeline = pipe(
    str.strip,
    str.lower,
    lambda s: s.replace("-", " "),
    lambda s: " ".join(s.split()),  # normalize spaces
    lambda s: s.title(),
)

test_texts = [
    "  hello-world  ",
    "  PYTHON   IS    GREAT  ",
    "  functional-programming-in-python  "
]

for text in test_texts:
    result = text_pipeline(text)
    print(f"'{text}' -> '{result}'")
```

### ข้อที่ 4-15

```python
# ข้อที่ 4: Memoization ด้วย functools.lru_cache
from functools import lru_cache

@lru_cache(maxsize=128)
def count_ways(n, coins):
    """นับวิธีการจ่ายเงิน n บาท ด้วยเหรียญ"""
    if n == 0:
        return 1
    if n < 0:
        return 0
    return sum(count_ways(n - coin, coins) for coin in coins)

coins = (1, 5, 10, 25)
for amount in [10, 25, 50, 100]:
    ways = count_ways(amount, coins)
    print(f"Ways to make {amount}¢: {ways}")
```

```python
# ข้อที่ 5: sorted ด้วย multiple keys
employees = [
    {"name": "Alice", "dept": "Engineering", "salary": 85000},
    {"name": "Bob", "dept": "Marketing", "salary": 70000},
    {"name": "Charlie", "dept": "Engineering", "salary": 90000},
    {"name": "Diana", "dept": "Marketing", "salary": 75000},
    {"name": "Eve", "dept": "Engineering", "salary": 80000},
]

# Sort: dept ascending, salary descending
sorted_employees = sorted(
    employees,
    key=lambda e: (e["dept"], -e["salary"])
)

print("Sorted (dept asc, salary desc):")
for emp in sorted_employees:
    print(f"  {emp['dept']}/{emp['name']}: ฿{emp['salary']:,}")
```

```python
# ข้อที่ 6: zip สำหรับ matrix transpose
def transpose(matrix):
    return list(map(list, zip(*matrix)))

m = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print("Original:")
for row in m:
    print(row)
print("Transposed:")
for row in transpose(m):
    print(row)
```

```python
# ข้อที่ 7: any/all สำหรับ validation
def validate_data(records, required_fields, validators):
    """Validate records"""
    errors = []
    
    for i, record in enumerate(records):
        # ตรวจสอบ required fields
        missing = [f for f in required_fields if f not in record or record[f] is None]
        if missing:
            errors.append(f"Record {i}: missing fields: {missing}")
            continue
        
        # ตรวจสอบแต่ละ validator
        for field, validator_func in validators.items():
            if field in record and not validator_func(record[field]):
                errors.append(f"Record {i}: invalid {field}: {record[field]}")
    
    return errors

records = [
    {"name": "Alice", "age": 25, "email": "alice@example.com"},
    {"name": "", "age": 17, "email": "invalid-email"},
    {"name": "Charlie", "age": 30, "email": "charlie@test.com"},
    {"age": 25},  # missing name and email
]

validators = {
    "name": lambda x: bool(x) and len(x) >= 2,
    "age": lambda x: isinstance(x, int) and 18 <= x <= 120,
    "email": lambda x: "@" in x and "." in x.split("@")[-1],
}

errors = validate_data(records, ["name", "age", "email"], validators)
if errors:
    print("Validation errors:")
    for error in errors:
        print(f"  - {error}")
else:
    print("All records valid!")
```

```python
# ข้อที่ 8: Partial functions สำหรับ formatting
from functools import partial

def format_number(number, prefix="", suffix="", decimal_places=2, thousands_sep=True):
    """จัดรูปแบบตัวเลข"""
    if thousands_sep:
        formatted = f"{number:,.{decimal_places}f}"
    else:
        formatted = f"{number:.{decimal_places}f}"
    return f"{prefix}{formatted}{suffix}"

# สร้าง specialized formatters ด้วย partial
format_baht = partial(format_number, prefix="฿", decimal_places=2)
format_usd = partial(format_number, prefix="$", decimal_places=2)
format_percent = partial(format_number, suffix="%", decimal_places=1)
format_integer = partial(format_number, decimal_places=0, thousands_sep=True)

test_values = [1234.5678, 9999999.99, 0.125, 1000000]
for val in test_values:
    print(f"{val}")
    print(f"  Baht:    {format_baht(val)}")
    print(f"  USD:     {format_usd(val)}")
    print(f"  Percent: {format_percent(val)}")
    print(f"  Integer: {format_integer(val)}")
```

```python
# ข้อที่ 9: enumerate สำหรับ text formatter
def format_numbered_list(items, start=1, indent=0):
    """จัดรูปแบบ numbered list"""
    spaces = " " * indent
    return "\n".join(
        f"{spaces}{i}. {item}"
        for i, item in enumerate(items, start=start)
    )

def format_table(headers, rows, sep=" | "):
    """จัดรูปแบบ table"""
    col_widths = [
        max(len(str(headers[i])), max(len(str(row[i])) for row in rows))
        for i in range(len(headers))
    ]
    
    header_row = sep.join(
        str(h).ljust(w) for h, w in zip(headers, col_widths)
    )
    separator = "-+-".join("-" * w for w in col_widths)
    data_rows = [
        sep.join(str(cell).ljust(w) for cell, w in zip(row, col_widths))
        for row in rows
    ]
    
    return "\n".join([header_row, separator] + data_rows)

items = ["Python", "JavaScript", "Java", "Go", "Rust"]
print(format_numbered_list(items))
print()

headers = ["Name", "Age", "Score"]
rows = [
    ["Alice", 25, 85.5],
    ["Bob", 30, 72.0],
    ["Charlie", 28, 91.5],
]
print(format_table(headers, rows))
```

```python
# ข้อที่ 10: Functional approach สำหรับ statistics
from functools import reduce
import math

def mean(data):
    return reduce(lambda acc, x: acc + x, data) / len(data)

def variance(data):
    m = mean(data)
    return mean(list(map(lambda x: (x - m)**2, data)))

def std_dev(data):
    return math.sqrt(variance(data))

def median(data):
    sorted_data = sorted(data)
    n = len(sorted_data)
    if n % 2 == 0:
        return (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2
    return sorted_data[n//2]

def mode(data):
    from collections import Counter
    freq = Counter(data)
    max_freq = max(freq.values())
    return [k for k, v in freq.items() if v == max_freq]

data = [4, 7, 2, 9, 4, 3, 7, 4, 1, 8, 5, 4]
print(f"Data: {sorted(data)}")
print(f"Mean: {mean(data):.2f}")
print(f"Median: {median(data):.2f}")
print(f"Mode: {mode(data)}")
print(f"Variance: {variance(data):.2f}")
print(f"Std Dev: {std_dev(data):.2f}")
print(f"Min: {reduce(lambda a, b: a if a < b else b, data)}")
print(f"Max: {reduce(lambda a, b: a if a > b else b, data)}")
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

| เครื่องมือ | ใช้เพื่อ | เมื่อไหร่ |
|-----------|---------|----------|
| `lambda` | Anonymous function | ใช้ครั้งเดียว, เป็น argument |
| `map()` | แปลงทุก element | ทำงานกับทุกตัว |
| `filter()` | กรอง elements | เลือกตามเงื่อนไข |
| `reduce()` | รวมเป็นค่าเดียว | accumulate |
| `sorted()` | เรียงลำดับ | กับ key function |
| `zip()` | รวม iterables | parallel iteration |
| `any()/all()` | ตรวจสอบ conditions | boolean check |
| `enumerate()` | เพิ่ม index | วน loop ที่ต้องการ index |
| `functools` | HOF utilities | partial, lru_cache, reduce |
| `operator` | Built-in operators | เป็น functions |

### Functional vs OOP vs Imperative

| แนวทาง | จุดแข็ง | เหมาะกับ |
|--------|--------|---------|
| Functional | ทดสอบง่าย, ไม่มี side effects | data transformation |
| OOP | จัดการ state ดี, modular | complex systems |
| Imperative | เข้าใจง่าย, control flow ชัด | procedural tasks |

### ขั้นต่อไป

ในบทถัดไป (Part 20) เราจะนำความรู้ทั้งหมดมาสร้าง **Project: Calculator App & Quiz Game** จริงๆ
