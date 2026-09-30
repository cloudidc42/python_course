# Part 02: Variables, Data Types & Operators

## สารบัญ (Table of Contents)

1. [Variables คืออะไร?](#variables-คืออะไร)
2. [การตั้งชื่อตามหลัก PEP 8](#การตั้งชื่อตามหลัก-pep-8)
3. [Data Types ทั้งหมด](#data-types-ทั้งหมด)
4. [int - จำนวนเต็ม](#int---จำนวนเต็ม)
5. [float - จำนวนทศนิยม](#float---จำนวนทศนิยม)
6. [complex - จำนวนเชิงซ้อน](#complex---จำนวนเชิงซ้อน)
7. [str - ข้อความ](#str---ข้อความ)
8. [bool - ค่าตรรกะ](#bool---ค่าตรรกะ)
9. [None - ค่าว่าง](#none---ค่าว่าง)
10. [Type Checking](#type-checking)
11. [Type Conversion](#type-conversion)
12. [Operators ทุกประเภท](#operators-ทุกประเภท)
13. [Operator Precedence](#operator-precedence)
14. [Multiple Assignment](#multiple-assignment)
15. [Constants](#constants)
16. [ตัวอย่างโค้ด](#ตัวอย่างโค้ด)
17. [แบบฝึกหัด](#แบบฝึกหัด)
18. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## Variables คืออะไร?

Variable (ตัวแปร) คือชื่อที่ใช้อ้างอิงไปยังค่าที่เก็บไว้ใน memory ใน Python ตัวแปรเป็น **dynamically typed** หมายความว่าไม่ต้องประกาศชนิดล่วงหน้า

```python
# Python - dynamically typed
x = 10          # x เป็น int
x = "hello"     # x เปลี่ยนเป็น str ได้ทันที
x = [1, 2, 3]  # x เปลี่ยนเป็น list ได้อีก

# C/Java - statically typed
# int x = 10;     # กำหนดชนิดตั้งแต่ประกาศ
# x = "hello";    # ERROR! ชนิดไม่ตรง
```

### วิธีสร้าง Variable

```python
# รูปแบบ: variable_name = value
age = 25
name = "สมชาย"
price = 99.99
is_active = True

# Python สร้าง object ใน memory แล้วให้ variable ชี้ไปที่ object นั้น
# ไม่ใช่ "เก็บค่า" แต่เป็น "ชี้ไปที่ object"
```

### Memory Model ของ Python

```
ตัวแปร x = 10

Stack (Variables)    Heap (Objects)
+----------+         +-----------+
|  x ----→|------→  |  int: 10  |
+----------+         +-----------+

ตัวแปร y = x (ทั้งคู่ชี้ object เดียวกัน)

Stack                Heap
+----------+         +-----------+
|  x ----→|------→  |  int: 10  |
|  y ----→|------↗  +-----------+
+----------+
```

---

## การตั้งชื่อตามหลัก PEP 8

**PEP 8** (Python Enhancement Proposal 8) คือ style guide มาตรฐานของ Python

### กฎการตั้งชื่อ Variable

```python
# ✅ ถูกต้อง
user_name = "Alice"          # snake_case (แนะนำ)
age = 25                     # lowercase ธรรมดา
total_price = 100.50         # หลายคำใช้ underscore
item1 = "apple"              # ตัวเลขท้ายชื่อได้
_private_var = "secret"      # underscore นำหน้า = private
MAX_SIZE = 100               # UPPER_CASE = constants
__dunder__ = "special"       # double underscore = dunder

# ❌ ผิด
1number = 10          # ขึ้นต้นด้วยตัวเลขไม่ได้
my-name = "Bob"       # ใช้ - ไม่ได้ (ใช้ underscore)
class = "Python"      # ห้ามใช้ reserved words
for = 10              # ห้ามใช้ keywords
```

### Reserved Keywords ของ Python

```python
# keywords เหล่านี้ห้ามใช้เป็น variable name
import keyword
print(keyword.kwlist)

# ผลลัพธ์:
# ['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await',
#  'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except',
#  'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is',
#  'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return',
#  'try', 'while', 'with', 'yield']
```

### Naming Conventions ตาม PEP 8

| ประเภท | Convention | ตัวอย่าง |
|--------|-----------|----------|
| Variables | `snake_case` | `user_name`, `total_price` |
| Functions | `snake_case` | `get_user()`, `calculate_total()` |
| Classes | `PascalCase` | `UserProfile`, `ShoppingCart` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_SIZE`, `PI_VALUE` |
| Private | `_leading_underscore` | `_private_method` |
| Name mangled | `__double_underscore` | `__very_private` |
| Magic methods | `__dunder__` | `__init__`, `__str__` |
| Modules | `lowercase` | `utils.py`, `helper.py` |
| Packages | `lowercase` | `mypackage/` |

### ตัวอย่างชื่อที่ดีและไม่ดี

```python
# ❌ ชื่อที่ไม่ดี
d = 7             # ไม่รู้ว่าคืออะไร
lst = [1, 2, 3]  # ไม่ชัดเจน
n = input()       # n คืออะไร?

# ✅ ชื่อที่ดี
days_until_deadline = 7
fruit_list = [1, 2, 3]
user_name = input()

# ✅ ชื่อที่ดี (สำหรับ loop variable)
for i in range(10):          # i ยอมรับได้
    print(i)

for index, item in enumerate(items):  # ดีกว่า i, v
    print(f"{index}: {item}")
```

---

## Data Types ทั้งหมด

Python มี built-in data types หลายประเภท:

```
Built-in Types
├── Numeric
│   ├── int (จำนวนเต็ม)
│   ├── float (ทศนิยม)
│   └── complex (เชิงซ้อน)
├── Text
│   └── str (ข้อความ)
├── Boolean
│   └── bool (True/False)
├── None
│   └── NoneType (ค่าว่าง)
├── Sequence
│   ├── list [แก้ไขได้]
│   ├── tuple (ไม่แก้ไข)
│   └── range
├── Set
│   ├── set {ไม่ซ้ำ}
│   └── frozenset (ไม่แก้ไข)
└── Mapping
    └── dict {key: value}
```

---

## int - จำนวนเต็ม

Python `int` ไม่มีขีดจำกัดขนาด (arbitrary precision)!

```python
# การสร้าง int
a = 42
b = -17
c = 0

# Python int ใหญ่ได้ไม่จำกัด
huge_number = 999999999999999999999999999999
print(huge_number)  # แสดงได้ครบ!
print(type(huge_number))  # <class 'int'>

# int ในระบบเลขต่างๆ
decimal = 255      # เลขฐาน 10 (ปกติ)
binary = 0b11111111   # เลขฐาน 2 (binary) = 255
octal = 0o377         # เลขฐาน 8 (octal) = 255
hexadecimal = 0xFF    # เลขฐาน 16 (hexadecimal) = 255

print(decimal, binary, octal, hexadecimal)
# 255 255 255 255

# แปลงกลับ
print(bin(255))    # 0b11111111
print(oct(255))    # 0o377
print(hex(255))    # 0xff
```

---

## float - จำนวนทศนิยม

```python
# การสร้าง float
pi = 3.14159
negative = -0.5
scientific = 1.5e3    # 1500.0
small = 2.5e-4        # 0.00025

print(type(pi))       # <class 'float'>

# Float precision issue (ปัญหาความแม่นยำ)
print(0.1 + 0.2)      # 0.30000000000000004 ไม่ใช่ 0.3!
print(0.1 + 0.2 == 0.3)  # False!

# วิธีแก้ด้วย round()
result = round(0.1 + 0.2, 1)
print(result)         # 0.3
print(result == 0.3)  # True

# Special float values
import math
print(math.inf)       # Infinity (บวก)
print(-math.inf)      # -Infinity (ลบ)
print(math.nan)       # NaN (Not a Number)

print(math.isinf(math.inf))   # True
print(math.isnan(math.nan))   # True
print(math.isfinite(3.14))    # True

# Float limits
print(float('inf'))   # inf
print(float('-inf'))  # -inf
print(float('nan'))   # nan
```

---

## complex - จำนวนเชิงซ้อน

```python
# การสร้าง complex number
c1 = 3 + 4j      # 3 เป็นส่วนจริง, 4j เป็นส่วนจินตภาพ
c2 = complex(2, -1)  # 2 + (-1)j

print(c1)          # (3+4j)
print(type(c1))    # <class 'complex'>

# Properties
print(c1.real)     # 3.0 (ส่วนจริง)
print(c1.imag)     # 4.0 (ส่วนจินตภาพ)
print(c1.conjugate())  # (3-4j) (conjugate)

# การคำนวณ
c3 = c1 + c2
print(c3)          # (5+3j)

c4 = c1 * c2
print(c4)          # (10+5j)

# Absolute value (modulus)
import cmath
print(abs(c1))     # 5.0 (sqrt(3^2 + 4^2))
print(cmath.phase(c1))  # มุม (radians)
```

---

## str - ข้อความ

```python
# การสร้าง str
s1 = 'single quotes'
s2 = "double quotes"
s3 = '''triple
single quotes
multiline'''
s4 = """triple
double quotes
multiline"""

# Empty string
empty = ""
also_empty = str()

print(type(s1))    # <class 'str'>

# Strings เป็น immutable (ไม่แก้ไขได้)
name = "Python"
# name[0] = "J"  # ERROR! TypeError

# แต่สร้างใหม่ได้
name = "J" + name[1:]
print(name)  # Jython

# ความยาว
print(len("Hello"))    # 5
print(len("สวัสดี"))   # 6
```

---

## bool - ค่าตรรกะ

```python
# Boolean values
is_active = True
is_deleted = False

print(type(True))   # <class 'bool'>
print(type(False))  # <class 'bool'>

# bool เป็น subclass ของ int!
print(isinstance(True, int))   # True
print(True == 1)               # True
print(False == 0)              # True
print(True + True)             # 2
print(True + False)            # 1
print(True * 5)                # 5

# Truthy and Falsy values
# Falsy: False, None, 0, 0.0, 0j, "", [], (), {}, set()
# Truthy: ทุกอย่างที่ไม่ใช่ falsy

print(bool(0))       # False
print(bool(1))       # True
print(bool(""))      # False
print(bool("Hello")) # True
print(bool([]))      # False
print(bool([1, 2]))  # True
print(bool(None))    # False
```

---

## None - ค่าว่าง

```python
# None แทนค่าที่ไม่มี/ว่างเปล่า
result = None
user = None

print(type(None))   # <class 'NoneType'>
print(result)       # None

# การตรวจสอบ None ที่ถูกต้อง
# ✅ ใช้ 'is' หรือ 'is not'
if result is None:
    print("ยังไม่มีค่า")

if result is not None:
    print("มีค่า")

# ❌ อย่าใช้ == กับ None (ยังทำงานได้ แต่ไม่แนะนำ)
if result == None:  # PEP 8 บอกว่าอย่าทำแบบนี้
    print("ยังไม่มีค่า")

# Function ที่ไม่มี return statement return None โดย default
def do_nothing():
    pass

result = do_nothing()
print(result)  # None
```

---

## Type Checking

### ใช้ type()

```python
values = [42, 3.14, "hello", True, None, [1, 2], (3, 4), {5: 6}]

for val in values:
    print(f"{repr(val):20} → {type(val).__name__}")

# ผลลัพธ์:
# 42                   → int
# 3.14                 → float
# 'hello'              → str
# True                 → bool
# None                 → NoneType
# [1, 2]               → list
# (3, 4)               → tuple
# {5: 6}               → dict
```

### ใช้ isinstance()

```python
x = 42
print(isinstance(x, int))         # True
print(isinstance(x, float))       # False
print(isinstance(x, (int, float))) # True (เป็นอันใดอันหนึ่ง)

# isinstance ดีกว่า type() เพราะรองรับ inheritance
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()
print(type(dog) == Animal)        # False
print(isinstance(dog, Animal))    # True ✅ (Dog สืบทอดจาก Animal)
print(isinstance(dog, Dog))       # True
```

### ตรวจสอบ Type แบบอื่น

```python
# hasattr - ตรวจสอบว่ามี attribute หรือไม่
x = [1, 2, 3]
print(hasattr(x, 'append'))       # True
print(hasattr(x, 'keys'))         # False

# callable - ตรวจสอบว่าเรียกเป็น function ได้ไหม
print(callable(print))            # True
print(callable(42))               # False

# __class__ attribute
print(x.__class__)                # <class 'list'>
print(x.__class__.__name__)       # list
```

---

## Type Conversion

### Implicit Conversion (อัตโนมัติ)

```python
# Python แปลงให้อัตโนมัติในบางกรณี
result = 10 + 3.5    # int + float = float
print(result)         # 13.5
print(type(result))   # <class 'float'>

result = True + 1    # bool + int = int
print(result)         # 2
print(type(result))   # <class 'int'>
```

### Explicit Conversion (ด้วยตนเอง)

```python
# int() - แปลงเป็น int
print(int(3.9))       # 3 (ตัดทศนิยมทิ้ง ไม่ปัด)
print(int("42"))      # 42
print(int(True))      # 1
print(int(False))     # 0
# print(int("3.14"))  # ERROR! ValueError
# print(int("hello")) # ERROR! ValueError

# float() - แปลงเป็น float
print(float(10))      # 10.0
print(float("3.14"))  # 3.14
print(float("1e3"))   # 1000.0
print(float(True))    # 1.0

# str() - แปลงเป็น str
print(str(42))        # "42"
print(str(3.14))      # "3.14"
print(str(True))      # "True"
print(str(None))      # "None"
print(str([1, 2, 3])) # "[1, 2, 3]"

# bool() - แปลงเป็น bool
print(bool(0))        # False
print(bool(1))        # True
print(bool(-1))       # True (ทุกตัวเลขที่ไม่ใช่ 0 = True)
print(bool(""))       # False
print(bool("False"))  # True! (string ที่ไม่ว่าง = True)
print(bool(None))     # False
print(bool([]))       # False
print(bool([0]))      # True

# list(), tuple(), set() - แปลงระหว่าง sequence types
my_list = [1, 2, 3, 2, 1]
my_tuple = tuple(my_list)     # (1, 2, 3, 2, 1)
my_set = set(my_list)         # {1, 2, 3} (ลบซ้ำ)
back_to_list = list(my_set)   # [1, 2, 3]

print(my_tuple)
print(my_set)
```

### ตารางสรุป Type Conversion

| ต้นทาง | ปลายทาง | ตัวอย่าง | ผลลัพธ์ |
|--------|---------|---------|---------|
| str "42" | int | `int("42")` | 42 |
| str "3.14" | float | `float("3.14")` | 3.14 |
| int 42 | str | `str(42)` | "42" |
| float 3.9 | int | `int(3.9)` | 3 |
| int 10 | float | `float(10)` | 10.0 |
| bool True | int | `int(True)` | 1 |
| int 1 | bool | `bool(1)` | True |
| list | tuple | `tuple([1,2])` | (1, 2) |
| tuple | list | `list((1,2))` | [1, 2] |
| list | set | `set([1,2,2])` | {1, 2} |

---

## Operators ทุกประเภท

### 1. Arithmetic Operators (เลขคณิต)

```python
a = 17
b = 5

print(f"a + b  = {a + b}")    # 22 (บวก)
print(f"a - b  = {a - b}")    # 12 (ลบ)
print(f"a * b  = {a * b}")    # 85 (คูณ)
print(f"a / b  = {a / b}")    # 3.4 (หาร - ได้ float เสมอ)
print(f"a // b = {a // b}")   # 3 (Floor division - ได้ int)
print(f"a % b  = {a % b}")    # 2 (Modulo - เศษจากการหาร)
print(f"a ** b = {a ** b}")   # 1419857 (ยกกำลัง)
print(f"-a     = {-a}")       # -17 (เปลี่ยนเครื่องหมาย)

# หมายเหตุ: // กับ % กับเลขติดลบ
print((-7) // 2)   # -4 (ปัดลง ไม่ใช่ตัด!)
print(7 // (-2))   # -4
print((-7) % 2)    # 1 (สัญญาณตาม divisor)
```

### 2. Comparison Operators (เปรียบเทียบ)

```python
x = 10
y = 20

print(x == y)    # False (เท่ากัน)
print(x != y)    # True  (ไม่เท่ากัน)
print(x < y)     # True  (น้อยกว่า)
print(x > y)     # False (มากกว่า)
print(x <= y)    # True  (น้อยกว่าหรือเท่ากัน)
print(x >= y)    # False (มากกว่าหรือเท่ากัน)

# เปรียบเทียบ string
s1 = "apple"
s2 = "banana"
print(s1 < s2)   # True (เปรียบเทียบตาม ASCII/Unicode)
print("Z" < "a") # True ('Z' = 90, 'a' = 97)

# Chained comparisons
age = 25
print(18 <= age < 65)     # True (Python อนุญาต!)
print(0 < x < y < 100)   # True

# ระวัง: เปรียบเทียบ float
print(0.1 + 0.2 == 0.3)    # False!
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True ✅
```

### 3. Logical Operators (ตรรกะ)

```python
# and - True ถ้าทั้งสองเป็น True
print(True and True)   # True
print(True and False)  # False
print(False and True)  # False
print(False and False) # False

# or - True ถ้าอย่างน้อยหนึ่งเป็น True
print(True or True)    # True
print(True or False)   # True
print(False or True)   # True
print(False or False)  # False

# not - กลับค่า
print(not True)   # False
print(not False)  # True

# ตัวอย่างจริง
age = 25
has_id = True

can_buy_alcohol = age >= 18 and has_id
print(f"ซื้อได้: {can_buy_alcohol}")   # True

is_student = True
is_employed = False
has_income = is_student or is_employed
print(f"มีรายได้: {has_income}")  # True

# Short-circuit evaluation
def check():
    print("check() ถูกเรียก!")
    return True

# False and ... = False (ไม่เรียก check())
print(False and check())   # False (check ไม่ถูกเรียก)

# True or ... = True (ไม่เรียก check())
print(True or check())     # True (check ไม่ถูกเรียก)
```

### 4. Bitwise Operators (Bit)

```python
a = 0b1010  # 10
b = 0b1100  # 12

print(f"a & b  = {a & b} = {bin(a & b)}")   # 8 = 0b1000 (AND)
print(f"a | b  = {a | b} = {bin(a | b)}")   # 14 = 0b1110 (OR)
print(f"a ^ b  = {a ^ b} = {bin(a ^ b)}")   # 6 = 0b0110 (XOR)
print(f"~a     = {~a} = {bin(~a)}")          # -11 (NOT)
print(f"a << 2 = {a << 2} = {bin(a << 2)}") # 40 = 0b101000 (Left shift)
print(f"a >> 1 = {a >> 1} = {bin(a >> 1)}") # 5 = 0b101 (Right shift)

# ตัวอย่างประยุกต์
# ตรวจสอบว่าเลขคู่หรือคี่
def is_even(n):
    return (n & 1) == 0

print(is_even(4))   # True
print(is_even(7))   # False

# คูณ/หาร 2 ด้วย bit shift
x = 10
print(x << 1)   # 20 (คูณ 2)
print(x >> 1)   # 5  (หาร 2)
```

### 5. Assignment Operators (การกำหนดค่า)

```python
# = (basic assignment)
x = 10
print(x)    # 10

# += (add and assign)
x += 5      # x = x + 5
print(x)    # 15

# -= (subtract and assign)
x -= 3      # x = x - 3
print(x)    # 12

# *= (multiply and assign)
x *= 2      # x = x * 2
print(x)    # 24

# /= (divide and assign)
x /= 4      # x = x / 4
print(x)    # 6.0

# //= (floor divide and assign)
x = 17
x //= 5     # x = x // 5
print(x)    # 3

# %= (modulo and assign)
x = 17
x %= 5      # x = x % 5
print(x)    # 2

# **= (power and assign)
x = 2
x **= 8     # x = x ** 8
print(x)    # 256

# &=, |=, ^=, <<=, >>= (bitwise assignment)
a = 0b1010
a &= 0b1100
print(bin(a))   # 0b1000

# := (Walrus Operator - Python 3.8+)
# กำหนดค่าพร้อม expression
numbers = [1, 2, 3, 4, 5]
if (n := len(numbers)) > 3:
    print(f"List ยาว {n} items")  # List ยาว 5 items

# ใช้ใน while loop
import re
data = "Hello World 123"
if match := re.search(r'\d+', data):
    print(f"พบตัวเลข: {match.group()}")  # พบตัวเลข: 123
```

### 6. Identity Operators (ตัวตน)

```python
# is - ตรวจสอบว่าเป็น object เดียวกันใน memory หรือไม่
# is not - ตรวจสอบว่าไม่ใช่ object เดียวกัน

x = [1, 2, 3]
y = x           # y ชี้ไปที่ object เดียวกับ x
z = [1, 2, 3]   # z เป็น object ใหม่ แม้ค่าเหมือนกัน

print(x is y)           # True (object เดียวกัน)
print(x is z)           # False (object ต่างกัน)
print(x == z)           # True (ค่าเหมือนกัน)
print(x is not z)       # True

# id() แสดง memory address
print(id(x))    # เช่น 140234567890
print(id(y))    # เลขเดียวกับ x
print(id(z))    # เลขต่างจาก x

# None ควรตรวจด้วย is
result = None
print(result is None)       # ✅ ถูกต้อง
print(result == None)       # ใช้งานได้ แต่ไม่แนะนำ

# Integer caching (optimization ของ Python)
a = 100
b = 100
print(a is b)   # True (Python cache integers -5 ถึง 256)

a = 1000
b = 1000
print(a is b)   # False (นอกช่วง cache)
```

### 7. Membership Operators (การเป็นสมาชิก)

```python
# in - ตรวจสอบว่ามีอยู่ใน sequence หรือไม่
# not in - ตรวจสอบว่าไม่มีอยู่

fruits = ["apple", "banana", "cherry"]
print("apple" in fruits)        # True
print("grape" in fruits)        # False
print("grape" not in fruits)    # True

# ใช้กับ string
sentence = "Python is awesome"
print("Python" in sentence)     # True
print("java" in sentence)       # False

# ใช้กับ dictionary (ตรวจสอบ keys)
config = {"host": "localhost", "port": 8080}
print("host" in config)         # True
print("password" in config)     # False
print("localhost" in config.values())  # True (ตรวจค่า)

# ใช้กับ set (เร็วกว่า list)
valid_colors = {"red", "green", "blue"}
user_input = "red"
print(user_input in valid_colors)   # True

# Performance: O(1) สำหรับ set/dict, O(n) สำหรับ list
```

---

## Operator Precedence

ลำดับการคำนวณ (จากสูงสุดไปต่ำสุด):

| ลำดับ | Operator | คำอธิบาย |
|-------|---------|---------|
| 1 (สูงสุด) | `()` | วงเล็บ |
| 2 | `**` | ยกกำลัง |
| 3 | `+x`, `-x`, `~x` | Unary (บวก/ลบ/NOT) |
| 4 | `*`, `/`, `//`, `%` | คูณ, หาร, floor div, modulo |
| 5 | `+`, `-` | บวก, ลบ |
| 6 | `<<`, `>>` | Bit shift |
| 7 | `&` | Bitwise AND |
| 8 | `^` | Bitwise XOR |
| 9 | `\|` | Bitwise OR |
| 10 | `==`, `!=`, `<`, `>`, `<=`, `>=`, `is`, `is not`, `in`, `not in` | Comparison |
| 11 | `not` | Logical NOT |
| 12 | `and` | Logical AND |
| 13 (ต่ำสุด) | `or` | Logical OR |

```python
# ตัวอย่าง Precedence
result = 2 + 3 * 4    # = 2 + 12 = 14 (ไม่ใช่ 20)
print(result)          # 14

result = (2 + 3) * 4  # = 5 * 4 = 20 (ใช้วงเล็บบังคับ)
print(result)          # 20

result = 2 ** 3 ** 2  # = 2 ** 9 = 512 (** เป็น right-associative)
print(result)          # 512

result = (2 ** 3) ** 2  # = 8 ** 2 = 64
print(result)            # 64

# ตัวอย่างซับซ้อน
x = 5
result = x > 3 and x < 10 or x == 5
# = (5 > 3) and (5 < 10) or (5 == 5)
# = True and True or True
# = True or True
# = True
print(result)  # True
```

---

## Multiple Assignment

```python
# 1. Assign ค่าเดียวกันให้หลายตัวแปร
a = b = c = 0
print(a, b, c)   # 0 0 0

# 2. Tuple unpacking (parallel assignment)
x, y = 10, 20
print(x, y)      # 10 20

# 3. Swap ค่าโดยไม่ต้องใช้ตัวแปรช่วย
x, y = y, x
print(x, y)      # 20 10

# 4. Extended unpacking (Python 3)
first, *rest = [1, 2, 3, 4, 5]
print(first)     # 1
print(rest)      # [2, 3, 4, 5]

first, *middle, last = [1, 2, 3, 4, 5]
print(first)     # 1
print(middle)    # [2, 3, 4]
print(last)      # 5

# 5. Unpack จาก function return
def get_coordinates():
    return 10.5, 20.3   # Return tuple

lat, lon = get_coordinates()
print(f"Lat: {lat}, Lon: {lon}")  # Lat: 10.5, Lon: 20.3

# 6. Augmented assignment
count = 0
count += 1
count += 1
print(count)  # 2

# 7. Chained assignment (ระวัง! mutable objects)
a = b = []    # a และ b ชี้ object เดียวกัน!
a.append(1)
print(b)      # [1] อันตราย!

# วิธีที่ถูกต้อง
a = []
b = []    # แยก object กัน
a.append(1)
print(b)  # [] ถูกต้อง
```

---

## Constants

Python ไม่มี constant จริงๆ แต่ใช้ convention:

```python
# Constants โดย convention ใช้ UPPER_CASE
PI = 3.14159265358979323846
MAX_CONNECTIONS = 100
DATABASE_URL = "postgresql://localhost:5432/mydb"
APP_VERSION = "1.2.3"

# ใช้ Final type hint (Python 3.8+) เพื่อบอก type checker
from typing import Final

MAX_SIZE: Final = 100
APP_NAME: Final[str] = "MyApp"

# ใน class ใช้ class attribute
class Config:
    DEBUG = False
    HOST = "localhost"
    PORT = 8080
    MAX_RETRIES = 3

# enum สำหรับ constants ที่เป็นกลุ่ม (แนะนำที่สุด)
from enum import Enum, auto

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

class Status(Enum):
    PENDING = auto()
    ACTIVE = auto()
    INACTIVE = auto()
    DELETED = auto()

print(Color.RED)          # Color.RED
print(Color.RED.value)    # 1
print(Color.RED.name)     # RED

print(Status.ACTIVE)      # Status.ACTIVE
print(Status.ACTIVE.value)  # 2 (auto ใส่เลขให้)
```

---

## ตัวอย่างโค้ด

### ตัวอย่างที่ 1: Variable Types Explorer

```python
# ตัวอย่างที่ 1: สำรวจ Variable Types
def show_variable_info(name, value):
    """แสดงข้อมูล variable"""
    print(f"Variable: {name}")
    print(f"  Value:   {repr(value)}")
    print(f"  Type:    {type(value).__name__}")
    print(f"  ID:      {id(value)}")
    print()

# ทดสอบทุก basic types
show_variable_info("integer", 42)
show_variable_info("float", 3.14)
show_variable_info("complex", 1+2j)
show_variable_info("string", "Hello")
show_variable_info("boolean", True)
show_variable_info("none", None)
```

### ตัวอย่างที่ 2: Type Conversion ครบวงจร

```python
# ตัวอย่างที่ 2: Type Conversion
print("=== int to others ===")
n = 42
print(f"int:     {n}")
print(f"float:   {float(n)}")
print(f"str:     {str(n)}")
print(f"bool:    {bool(n)}")
print(f"complex: {complex(n)}")
print(f"bin:     {bin(n)}")
print(f"oct:     {oct(n)}")
print(f"hex:     {hex(n)}")

print("\n=== str to numbers ===")
s = "123"
print(f"str:     {s}")
print(f"int:     {int(s)}")
print(f"float:   {float(s)}")

# Handle errors
try:
    bad = int("hello")
except ValueError as e:
    print(f"Error: {e}")
```

### ตัวอย่างที่ 3: All Arithmetic Operations

```python
# ตัวอย่างที่ 3: ดำเนินการทางเลขคณิตทั้งหมด
def arithmetic_demo(a, b):
    """แสดงผลการคำนวณทั้งหมด"""
    print(f"a = {a}, b = {b}")
    print(f"  a + b  = {a + b}")
    print(f"  a - b  = {a - b}")
    print(f"  a * b  = {a * b}")
    print(f"  a / b  = {a / b:.4f}")
    print(f"  a // b = {a // b}")
    print(f"  a % b  = {a % b}")
    print(f"  a ** b = {a ** b}")
    print(f"  abs(a) = {abs(a)}")
    print()

arithmetic_demo(17, 5)
arithmetic_demo(-17, 5)
arithmetic_demo(2, 10)
```

### ตัวอย่างที่ 4: Comparison Deep Dive

```python
# ตัวอย่างที่ 4: Comparison ลึกๆ
# เปรียบเทียบตัวเลข
print("=== Numeric Comparison ===")
print(f"1 == 1.0: {1 == 1.0}")      # True
print(f"1 is 1.0: {1 is 1.0}")      # False
print(f"True == 1: {True == 1}")    # True
print(f"True is 1: {True is 1}")    # False

# เปรียบเทียบ string
print("\n=== String Comparison ===")
s1 = "Python"
s2 = "Python"
s3 = "python"

print(f"'Python' == 'Python': {s1 == s2}")     # True
print(f"'Python' == 'python': {s1 == s3}")     # False
print(f"'A' < 'a': {'A' < 'a'}")              # True (ASCII)
print(f"'apple' < 'banana': {'apple' < 'banana'}")  # True

# String interning
print(f"s1 is s2: {s1 is s2}")  # True หรือ False ขึ้นกับ implementation
```

### ตัวอย่างที่ 5: Logical Short-Circuit

```python
# ตัวอย่างที่ 5: Short-Circuit Evaluation
def expensive_check():
    """จำลอง function ที่ใช้เวลานาน"""
    print("  (expensive_check ถูกเรียก!)")
    return True

def safe_divide(a, b):
    """หารอย่างปลอดภัยด้วย short-circuit"""
    return b != 0 and a / b  # ไม่หารถ้า b เป็น 0

print("=== Short-circuit AND ===")
print("False and expensive_check():")
result = False and expensive_check()  # ไม่เรียก expensive_check
print(f"  = {result}")

print("\nTrue and expensive_check():")
result = True and expensive_check()   # เรียก expensive_check
print(f"  = {result}")

print("\n=== Short-circuit OR ===")
print("True or expensive_check():")
result = True or expensive_check()    # ไม่เรียก expensive_check
print(f"  = {result}")

print("\nFalse or expensive_check():")
result = False or expensive_check()   # เรียก expensive_check
print(f"  = {result}")

print("\n=== Safe Divide ===")
print(f"safe_divide(10, 2) = {safe_divide(10, 2)}")
print(f"safe_divide(10, 0) = {safe_divide(10, 0)}")
```

### ตัวอย่างที่ 6: Bitwise Operations

```python
# ตัวอย่างที่ 6: Bitwise Operations ประยุกต์ใช้งาน
print("=== Permission System using Bits ===")
# สร้างระบบ permissions ด้วย bitwise
READ    = 0b001  # 1
WRITE   = 0b010  # 2
EXECUTE = 0b100  # 4

# กำหนด permissions ด้วย OR
user_perms = READ | WRITE         # 0b011 = 3
admin_perms = READ | WRITE | EXECUTE  # 0b111 = 7
readonly_perms = READ              # 0b001 = 1

def check_permission(user_perms, permission):
    return bool(user_perms & permission)

print(f"User permissions:  {bin(user_perms)}")
print(f"Admin permissions: {bin(admin_perms)}")
print()

print(f"User can read:     {check_permission(user_perms, READ)}")
print(f"User can write:    {check_permission(user_perms, WRITE)}")
print(f"User can execute:  {check_permission(user_perms, EXECUTE)}")
print()
print(f"Admin can read:    {check_permission(admin_perms, READ)}")
print(f"Admin can execute: {check_permission(admin_perms, EXECUTE)}")

# เพิ่ม permission
user_perms |= EXECUTE
print(f"\nหลังเพิ่ม execute: {bin(user_perms)}")

# ลบ permission
user_perms &= ~WRITE    # AND กับ NOT WRITE
print(f"หลังลบ write: {bin(user_perms)}")
```

### ตัวอย่างที่ 7: Multiple Assignment

```python
# ตัวอย่างที่ 7: Multiple Assignment Tricks
# Swap แบบ Python
x, y = 10, 20
print(f"ก่อน swap: x={x}, y={y}")
x, y = y, x
print(f"หลัง swap: x={x}, y={y}")

# Unpack list
coordinates = [10.5, 20.3, 35.1]
lat, lon, alt = coordinates
print(f"Lat: {lat}, Lon: {lon}, Alt: {alt}")

# Unpack จาก function
def min_max(numbers):
    return min(numbers), max(numbers)

minimum, maximum = min_max([3, 1, 4, 1, 5, 9, 2, 6])
print(f"Min: {minimum}, Max: {maximum}")

# Extended unpacking
first, *rest = range(1, 6)
print(f"First: {first}, Rest: {rest}")

*init, last = range(1, 6)
print(f"Init: {init}, Last: {last}")

head, *body, tail = "Hello World Python"
print(f"Head: {head}, Tail: {tail}")
print(f"Body: {''.join(body)}")
```

### ตัวอย่างที่ 8: Walrus Operator

```python
# ตัวอย่างที่ 8: Walrus Operator (:=) Python 3.8+
import re

# ตัวอย่าง 1: ใน while loop
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
index = 0
while (value := data[index]) < 7:
    print(f"Processing: {value}")
    index += 1
print(f"หยุดที่: {value}")

# ตัวอย่าง 2: ใน if statement
text = "Error: connection refused on port 8080"
if match := re.search(r'port (\d+)', text):
    port = match.group(1)
    print(f"พบ port number: {port}")

# ตัวอย่าง 3: ลดการ compute ซ้ำ
numbers = [1, 2, 3, 4, 5]
# แบบเดิม
total = sum(numbers)
if total > 10:
    print(f"Total {total} มากกว่า 10")

# แบบ walrus
if (total := sum(numbers)) > 10:
    print(f"Total {total} มากกว่า 10")
```

### ตัวอย่างที่ 9: Operator Precedence ลึก

```python
# ตัวอย่างที่ 9: ทำความเข้าใจ Operator Precedence
expressions = [
    ("2 + 3 * 4", 2 + 3 * 4),
    ("(2 + 3) * 4", (2 + 3) * 4),
    ("2 ** 3 ** 2", 2 ** 3 ** 2),      # right-associative
    ("(2 ** 3) ** 2", (2 ** 3) ** 2),
    ("-2 ** 2", -2 ** 2),              # = -(2**2) = -4
    ("(-2) ** 2", (-2) ** 2),          # = 4
    ("4 + 2 > 5 and 3 < 4", 4 + 2 > 5 and 3 < 4),
    ("not True or False", not True or False),
    ("not (True or False)", not (True or False)),
]

for expr, result in expressions:
    print(f"{expr:<35} = {result}")
```

### ตัวอย่างที่ 10: Integer Bases Conversion

```python
# ตัวอย่างที่ 10: เลขฐานต่างๆ
def show_number_bases(n):
    """แสดงตัวเลขในทุกฐาน"""
    print(f"\nตัวเลข: {n} (decimal)")
    print(f"  Binary (ฐาน 2):  {bin(n):>12}  = {n:>5} (decimal)")
    print(f"  Octal (ฐาน 8):   {oct(n):>12}  = {n:>5} (decimal)")
    print(f"  Decimal (ฐาน 10):  {n:>10}  = {n:>5} (decimal)")
    print(f"  Hex (ฐาน 16):    {hex(n):>12}  = {n:>5} (decimal)")
    print(f"  Binary string:   {n:08b}  (8-bit)")

show_number_bases(42)
show_number_bases(255)
show_number_bases(16)

# แปลงจากฐานอื่นกลับมา decimal
print("\nแปลงกลับมา decimal:")
print(f"0b101010 = {int('0b101010', 2)}")   # หรือ int('101010', 2)
print(f"0o52     = {int('0o52', 8)}")        # หรือ int('52', 8)
print(f"0x2a     = {int('0x2a', 16)}")       # หรือ int('2a', 16)
```

### ตัวอย่างที่ 11: Constants ด้วย Enum

```python
# ตัวอย่างที่ 11: Constants ด้วย Enum
from enum import Enum, IntFlag, auto

class Weekday(Enum):
    MONDAY = 1
    TUESDAY = 2
    WEDNESDAY = 3
    THURSDAY = 4
    FRIDAY = 5
    SATURDAY = 6
    SUNDAY = 7
    
    def is_weekend(self):
        return self in (Weekday.SATURDAY, Weekday.SUNDAY)
    
    @classmethod
    def workdays(cls):
        return [day for day in cls if not day.is_weekend()]

# ใช้งาน
today = Weekday.WEDNESDAY
print(f"วันนี้: {today.name}")
print(f"วันหยุด: {today.is_weekend()}")

print("\nวันทำงาน:")
for day in Weekday.workdays():
    print(f"  {day.name} ({day.value})")
```

### ตัวอย่างที่ 12: isinstance ขั้นสูง

```python
# ตัวอย่างที่ 12: isinstance กับ abstract types
from collections.abc import Sequence, Mapping, Iterable

def describe_type(obj):
    """บอก properties ของ object"""
    print(f"\nObject: {repr(obj)}")
    print(f"  Type:     {type(obj).__name__}")
    print(f"  Iterable: {isinstance(obj, Iterable)}")
    print(f"  Sequence: {isinstance(obj, Sequence)}")
    print(f"  Mapping:  {isinstance(obj, Mapping)}")
    print(f"  Number:   {isinstance(obj, (int, float, complex))}")

describe_type(42)
describe_type("hello")
describe_type([1, 2, 3])
describe_type((1, 2, 3))
describe_type({"key": "value"})
describe_type({1, 2, 3})
```

### ตัวอย่างที่ 13: Type Annotation และ Type Check

```python
# ตัวอย่างที่ 13: Type Annotations
from typing import Union, Optional

def add_numbers(a: Union[int, float], b: Union[int, float]) -> float:
    """บวกตัวเลขสองจำนวน"""
    return float(a + b)

def find_user(user_id: int) -> Optional[str]:
    """ค้นหา user"""
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)

# Python 3.10+ syntax (ใช้ | แทน Union)
def process(value: int | float | str) -> str:
    """แปลงทุกอย่างเป็น string"""
    return str(value)

print(add_numbers(3, 4.5))       # 7.5
print(find_user(1))              # Alice
print(find_user(99))             # None
print(process(42))               # "42"
print(process(3.14))             # "3.14"
print(process("hello"))          # "hello"
```

---

## แบบฝึกหัด

### ข้อที่ 1: Variable Explorer

สร้างตัวแปรแต่ละชนิด (int, float, str, bool, None) แล้วแสดงข้อมูล type, value, id ของแต่ละตัว

### ข้อที่ 2: Type Conversion Calculator

เขียน function `safe_convert(value, target_type)` ที่แปลง value เป็น type ที่ต้องการ และ return (result, success_bool) ถ้า conversion ล้มเหลวให้ return (None, False)

### ข้อที่ 3: Temperature Converter

เขียนโปรแกรมแปลงอุณหภูมิระหว่าง Celsius, Fahrenheit, Kelvin:
- C to F: `F = C * 9/5 + 32`
- F to C: `C = (F - 32) * 5/9`
- C to K: `K = C + 273.15`

### ข้อที่ 4: Bitwise Calculator

เขียน function ที่รับตัวเลข 2 ตัวแล้วแสดงผล bitwise operations ทั้งหมด (AND, OR, XOR, NOT, LEFT SHIFT, RIGHT SHIFT)

### ข้อที่ 5: Constants สำหรับร้านอาหาร

สร้าง Enum class `MenuCategory` ที่มี: APPETIZER, MAIN_COURSE, DESSERT, DRINK และ Enum class `OrderStatus` ที่มี: PENDING, PREPARING, READY, DELIVERED, CANCELLED

### ข้อที่ 6: Multiple Assignment Practice

ใช้ multiple assignment เพื่อ:
- สร้างตัวแปร 3 ตัวพร้อมกัน
- Swap ค่า 3 ตัวแปร (a, b, c เป็น b, c, a)
- Unpack list เป็น first, middle_items, last

### ข้อที่ 7: Operator Precedence Test

คำนวณค่าของ expressions ต่อไปนี้โดยไม่รันโค้ด แล้วตรวจสอบด้วย Python:
1. `3 + 4 * 2 - 1`
2. `2 ** 3 + 4 * 2`
3. `10 > 5 and 3 < 2 or True`
4. `not False and True or False`
5. `5 > 3 > 1`

### ข้อที่ 8: Truthy/Falsy Explorer

เขียน function `check_truthiness(values)` ที่รับ list ของค่าต่างๆ แล้วแสดงว่าแต่ละค่าเป็น truthy หรือ falsy

### ข้อที่ 9: Integer Bases Converter

เขียน function `convert_base(n, from_base, to_base)` ที่แปลงตัวเลขจากฐานหนึ่งไปอีกฐาน รองรับ base 2, 8, 10, 16

### ข้อที่ 10: Grade Calculator

เขียนโปรแกรมรับคะแนน 5 วิชา แล้ว:
- คำนวณค่าเฉลี่ย
- หาคะแนนสูงสุดและต่ำสุด
- ตัดเกรด: A(90+), B(80+), C(70+), D(60+), F(<60)
- ใช้ comparison operators, arithmetic operators, และ type conversion

---

## เฉลยแบบฝึกหัด

### เฉลยข้อที่ 1: Variable Explorer

```python
# เฉลยข้อที่ 1
def show_info(name, value):
    print(f"{'='*30}")
    print(f"Name:  {name}")
    print(f"Value: {repr(value)}")
    print(f"Type:  {type(value).__name__}")
    print(f"ID:    {id(value)}")

variables = [
    ("count", 42),
    ("pi", 3.14159),
    ("message", "สวัสดีโลก"),
    ("is_valid", True),
    ("empty", None),
]

for name, value in variables:
    show_info(name, value)
```

### เฉลยข้อที่ 2: Safe Convert

```python
# เฉลยข้อที่ 2
def safe_convert(value, target_type):
    """แปลง type อย่างปลอดภัย"""
    try:
        result = target_type(value)
        return result, True
    except (ValueError, TypeError):
        return None, False

# ทดสอบ
tests = [
    ("42", int),
    ("3.14", float),
    ("hello", int),
    (True, str),
    (None, int),
    ("100", bool),
]

for value, target in tests:
    result, success = safe_convert(value, target)
    status = "✅" if success else "❌"
    print(f"{status} convert({repr(value)}, {target.__name__}) = {repr(result)}")
```

### เฉลยข้อที่ 3: Temperature Converter

```python
# เฉลยข้อที่ 3
def celsius_to_fahrenheit(c):
    return c * 9/5 + 32

def fahrenheit_to_celsius(f):
    return (f - 32) * 5/9

def celsius_to_kelvin(c):
    return c + 273.15

def kelvin_to_celsius(k):
    return k - 273.15

def convert_temperature(value, from_unit, to_unit):
    """แปลงอุณหภูมิ"""
    # แปลงเป็น Celsius ก่อน
    if from_unit == "F":
        celsius = fahrenheit_to_celsius(value)
    elif from_unit == "K":
        celsius = kelvin_to_celsius(value)
    else:
        celsius = value
    
    # แปลงจาก Celsius เป็น target
    if to_unit == "F":
        return celsius_to_fahrenheit(celsius)
    elif to_unit == "K":
        return celsius_to_kelvin(celsius)
    else:
        return celsius

# ทดสอบ
print(f"100°C = {convert_temperature(100, 'C', 'F'):.2f}°F")
print(f"32°F = {convert_temperature(32, 'F', 'C'):.2f}°C")
print(f"0°C = {convert_temperature(0, 'C', 'K'):.2f}K")
print(f"300K = {convert_temperature(300, 'K', 'C'):.2f}°C")
```

### เฉลยข้อที่ 4: Bitwise Calculator

```python
# เฉลยข้อที่ 4
def bitwise_calc(a, b):
    """แสดง bitwise operations ทั้งหมด"""
    print(f"a = {a} ({bin(a)})")
    print(f"b = {b} ({bin(b)})")
    print(f"{'─'*40}")
    print(f"AND:         a & b  = {a & b} ({bin(a & b)})")
    print(f"OR:          a | b  = {a | b} ({bin(a | b)})")
    print(f"XOR:         a ^ b  = {a ^ b} ({bin(a ^ b)})")
    print(f"NOT a:       ~a     = {~a}")
    print(f"Left shift:  a << 2 = {a << 2} ({bin(a << 2)})")
    print(f"Right shift: a >> 1 = {a >> 1} ({bin(a >> 1)})")

bitwise_calc(10, 12)
```

### เฉลยข้อที่ 5: Restaurant Constants

```python
# เฉลยข้อที่ 5
from enum import Enum, auto

class MenuCategory(Enum):
    APPETIZER    = auto()
    MAIN_COURSE  = auto()
    DESSERT      = auto()
    DRINK        = auto()

class OrderStatus(Enum):
    PENDING    = auto()
    PREPARING  = auto()
    READY      = auto()
    DELIVERED  = auto()
    CANCELLED  = auto()
    
    def is_active(self):
        return self in (OrderStatus.PENDING, OrderStatus.PREPARING, OrderStatus.READY)

# ทดสอบ
print("Menu Categories:")
for cat in MenuCategory:
    print(f"  {cat.name}: {cat.value}")

print("\nOrder Statuses:")
for status in OrderStatus:
    active = "(active)" if status.is_active() else ""
    print(f"  {status.name}: {status.value} {active}")
```

### เฉลยข้อที่ 6: Multiple Assignment Practice

```python
# เฉลยข้อที่ 6
# สร้าง 3 ตัวแปรพร้อมกัน
x, y, z = 10, 20, 30
print(f"x={x}, y={y}, z={z}")

# Rotate: a, b, c → b, c, a
a, b, c = 1, 2, 3
print(f"ก่อน: a={a}, b={b}, c={c}")
a, b, c = b, c, a
print(f"หลัง: a={a}, b={b}, c={c}")

# Unpack list
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
first, *middle_items, last = data
print(f"First: {first}")
print(f"Middle: {middle_items}")
print(f"Last: {last}")
```

### เฉลยข้อที่ 7: Operator Precedence Test

```python
# เฉลยข้อที่ 7
tests = [
    ("3 + 4 * 2 - 1", 3 + 4 * 2 - 1),           # 10
    ("2 ** 3 + 4 * 2", 2 ** 3 + 4 * 2),           # 16
    ("10 > 5 and 3 < 2 or True", 10 > 5 and 3 < 2 or True),  # True
    ("not False and True or False", not False and True or False),  # True
    ("5 > 3 > 1", 5 > 3 > 1),                     # True (chained)
]

print("Expression คำตอบ การคำนวณ")
for expr, result in tests:
    print(f"{expr:<40} = {result}")
```

### เฉลยข้อที่ 8: Truthy/Falsy Explorer

```python
# เฉลยข้อที่ 8
def check_truthiness(values):
    """ตรวจสอบ truthy/falsy"""
    print(f"{'Value':<20} {'Type':<12} {'Truthy/Falsy'}")
    print("─" * 50)
    for val in values:
        is_truthy = bool(val)
        label = "✅ Truthy" if is_truthy else "❌ Falsy"
        type_name = type(val).__name__
        print(f"{repr(val):<20} {type_name:<12} {label}")

test_values = [
    0, 1, -1, 0.0, 0.1,
    "", "hello", "False",
    [], [0], (), (0,),
    {}, {"key": "val"},
    None, True, False
]

check_truthiness(test_values)
```

### เฉลยข้อที่ 9: Base Converter

```python
# เฉลยข้อที่ 9
def convert_base(number_str, from_base, to_base):
    """
    แปลงตัวเลขจากฐานหนึ่งไปอีกฐาน
    
    Args:
        number_str: ตัวเลขในรูป string
        from_base: ฐานต้นทาง (2, 8, 10, 16)
        to_base: ฐานปลายทาง (2, 8, 10, 16)
    
    Returns:
        str: ตัวเลขในฐานปลายทาง
    """
    # แปลงเป็น decimal ก่อน
    decimal = int(number_str, from_base)
    
    # แปลงจาก decimal ไปฐานที่ต้องการ
    if to_base == 2:
        return bin(decimal)[2:]   # ตัด 0b ออก
    elif to_base == 8:
        return oct(decimal)[2:]   # ตัด 0o ออก
    elif to_base == 10:
        return str(decimal)
    elif to_base == 16:
        return hex(decimal)[2:].upper()  # ตัด 0x ออก
    else:
        raise ValueError(f"ไม่รองรับฐาน {to_base}")

# ทดสอบ
print(f"255 (decimal) → binary:  {convert_base('255', 10, 2)}")
print(f"255 (decimal) → octal:   {convert_base('255', 10, 8)}")
print(f"255 (decimal) → hex:     {convert_base('255', 10, 16)}")
print(f"FF (hex) → decimal:      {convert_base('FF', 16, 10)}")
print(f"1010 (binary) → decimal: {convert_base('1010', 2, 10)}")
```

### เฉลยข้อที่ 10: Grade Calculator

```python
# เฉลยข้อที่ 10
def get_grade(score):
    """ตัดเกรดจากคะแนน"""
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

def calculate_grades():
    """รับคะแนน 5 วิชาและคำนวณผล"""
    subjects = ["คณิตศาสตร์", "ภาษาไทย", "วิทยาศาสตร์", "ภาษาอังกฤษ", "สังคม"]
    scores = []
    
    print("=== ระบบตัดเกรด ===")
    for subject in subjects:
        while True:
            try:
                score = float(input(f"คะแนน{subject} (0-100): "))
                if 0 <= score <= 100:
                    scores.append(score)
                    break
                else:
                    print("คะแนนต้องอยู่ระหว่าง 0-100")
            except ValueError:
                print("กรุณาพิมพ์ตัวเลข")
    
    # คำนวณ
    average = sum(scores) / len(scores)
    max_score = max(scores)
    min_score = min(scores)
    
    # แสดงผล
    print("\n" + "="*40)
    print("ผลการเรียน")
    print("-"*40)
    for subject, score in zip(subjects, scores):
        grade = get_grade(score)
        print(f"{subject:<15}: {score:>6.1f}  เกรด {grade}")
    
    print("-"*40)
    print(f"{'ค่าเฉลี่ย':<15}: {average:>6.1f}  เกรด {get_grade(average)}")
    print(f"{'คะแนนสูงสุด':<15}: {max_score:>6.1f}")
    print(f"{'คะแนนต่ำสุด':<15}: {min_score:>6.1f}")
    print("="*40)

# ทดสอบแบบไม่ต้อง input
scores = [85, 72, 90, 68, 78]
subjects = ["คณิตศาสตร์", "ภาษาไทย", "วิทยาศาสตร์", "ภาษาอังกฤษ", "สังคม"]

average = sum(scores) / len(scores)
print(f"ค่าเฉลี่ย: {average:.2f}  เกรด: {get_grade(average)}")
print(f"สูงสุด: {max(scores)}, ต่ำสุด: {min(scores)}")
for subject, score in zip(subjects, scores):
    print(f"  {subject}: {score} ({get_grade(score)})")
```

---

## สรุป

ใน Part 02 นี้เราได้เรียนรู้:

1. **Variables** - ตัวแปร, การตั้งชื่อตาม PEP 8, memory model
2. **Data Types** - int, float, complex, str, bool, None
3. **Type Checking** - `type()`, `isinstance()`, `hasattr()`
4. **Type Conversion** - implicit/explicit, ตาราง conversion
5. **Operators** - arithmetic, comparison, logical, bitwise, assignment, identity, membership
6. **Operator Precedence** - ลำดับการคำนวณ
7. **Multiple Assignment** - tuple unpacking, extended unpacking, swap
8. **Constants** - convention, Final, Enum

## ขั้นตอนต่อไป

- **Part 03**: Strings & String Methods
- ฝึกใช้ operator ทุกประเภท
- เข้าใจ truthy/falsy values
- ลอง type conversion ในสถานการณ์จริง

---

*หมายเหตุ: ทุก code block ทดสอบแล้วบน Python 3.12*
