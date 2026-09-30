# Part 12: Tuples & Named Tuples

## บทนำ (Introduction)

**Tuple** เป็น ordered collection ที่ **immutable** (ไม่สามารถเปลี่ยนแปลงได้หลังสร้าง) ใน Python
คล้าย List แต่มีความต่างที่สำคัญคือ ความเปลี่ยนแปลงไม่ได้ทำให้ Tuple มีประโยชน์ในหลายกรณี

### Tuple vs List เปรียบเทียบ
| คุณสมบัติ | Tuple | List |
|-----------|-------|------|
| Syntax | `(1, 2, 3)` | `[1, 2, 3]` |
| Mutable | ไม่ได้ | ได้ |
| Methods | น้อยกว่า (2) | มาก (11) |
| Performance | เร็วกว่า | ช้ากว่า |
| Memory | ใช้น้อยกว่า | ใช้มากกว่า |
| Hashable | ได้ (ถ้าไม่มี mutable) | ไม่ได้ |
| Use case | ข้อมูลคงที่ | ข้อมูลเปลี่ยนแปลงได้ |

---

## 1. Tuples พื้นฐาน

### 1.1 การสร้าง Tuple

```python
# วิธีที่ 1: ใช้วงเล็บ
empty_tuple = ()
single = (42,)      # ต้องมี comma! ไม่งั้นเป็น int
point = (3, 4)
rgb = (255, 128, 0)
person = ("Alice", 25, "Bangkok")

print(empty_tuple)  # ()
print(single)       # (42,)
print(point)        # (3, 4)
print(type(single)) # <class 'tuple'>

# วิธีที่ 2: ไม่มีวงเล็บ (tuple packing)
coordinates = 10, 20, 30
print(coordinates)      # (10, 20, 30)
print(type(coordinates))# <class 'tuple'>

# วิธีที่ 3: tuple() constructor
from_list = tuple([1, 2, 3, 4, 5])
from_string = tuple("Python")
from_range = tuple(range(1, 6))

print(from_list)    # (1, 2, 3, 4, 5)
print(from_string)  # ('P', 'y', 't', 'h', 'o', 'n')
print(from_range)   # (1, 2, 3, 4, 5)
```

### 1.2 Single-element Tuple

```python
# สำคัญมาก! ต้องมี comma
not_tuple = (42)     # นี่คือ int ไม่ใช่ tuple
is_tuple = (42,)     # นี่คือ tuple
also_tuple = 42,     # ก็เป็น tuple เช่นกัน

print(type(not_tuple))  # <class 'int'>
print(type(is_tuple))   # <class 'tuple'>
print(type(also_tuple)) # <class 'tuple'>

# ตรวจสอบ
print(not_tuple == 42)          # True
print(is_tuple == (42,))        # True
print(is_tuple[0] == 42)        # True
```

### 1.3 Accessing Tuple Elements

```python
person = ("Alice", 25, "Bangkok", "Engineer")

# Positive indexing
print(person[0])   # Alice
print(person[2])   # Bangkok

# Negative indexing
print(person[-1])  # Engineer
print(person[-2])  # Bangkok

# Slicing
print(person[1:3]) # (25, 'Bangkok')
print(person[:2])  # ('Alice', 25)
print(person[::2]) # ('Alice', 'Bangkok')

# length
print(len(person))  # 4

# นับและหาตำแหน่ง
colors = ("red", "blue", "green", "blue", "red")
print(colors.count("blue"))  # 2
print(colors.index("green")) # 2
```

### 1.4 Tuple is Immutable

```python
point = (3, 4)

# ลองแก้ไข → จะ Error
try:
    point[0] = 10
except TypeError as e:
    print(f"Error: {e}")
# Error: 'tuple' object does not support item assignment

# ลองลบ → จะ Error
try:
    del point[0]
except TypeError as e:
    print(f"Error: {e}")
# Error: 'tuple' object doesn't support item deletion

# แต่ถ้า tuple มี mutable object ข้างใน สามารถแก้ไข object นั้นได้
mixed = ([1, 2, 3], "hello", 42)
mixed[0].append(4)      # ได้! แก้ไข list ข้างใน
print(mixed)  # ([1, 2, 3, 4], 'hello', 42)

# แต่แทน element ไม่ได้
try:
    mixed[0] = [10, 20]
except TypeError as e:
    print(f"Error: {e}")
```

---

## 2. Tuple Methods

### 2.1 count() และ index()

```python
numbers = (1, 2, 3, 2, 1, 4, 2, 5, 2)

# count() - นับจำนวน
print(numbers.count(2))   # 4
print(numbers.count(1))   # 2
print(numbers.count(9))   # 0

# index() - หาตำแหน่งแรก
print(numbers.index(3))   # 2
print(numbers.index(2))   # 1 (ตัวแรก)
print(numbers.index(2, 2))   # 3 (เริ่มหาจาก index 2)
print(numbers.index(2, 4))   # 6 (เริ่มหาจาก index 4)

try:
    print(numbers.index(9))
except ValueError:
    print("ไม่พบ 9 ใน tuple")
```

---

## 3. Tuple Unpacking

### 3.1 Basic Unpacking

```python
# Unpack tuple เป็นตัวแปร
point = (3, 4)
x, y = point
print(x, y)  # 3 4

# Unpack หลายตัวแปร
person = ("Alice", 25, "Bangkok")
name, age, city = person
print(name, age, city)  # Alice 25 Bangkok

# Unpack ใน for loop
coordinates = [(1, 2), (3, 4), (5, 6)]
for x, y in coordinates:
    print(f"Point: ({x}, {y})")

# Unpack จาก function return
def get_dimensions():
    return 1920, 1080

width, height = get_dimensions()
print(f"{width}x{height}")  # 1920x1080
```

### 3.2 Star Unpacking (*)

```python
# เก็บค่าที่เหลือด้วย *
first, *rest = (1, 2, 3, 4, 5)
print(first)  # 1
print(rest)   # [2, 3, 4, 5]  ← กลายเป็น list!

*start, last = (1, 2, 3, 4, 5)
print(start)  # [1, 2, 3, 4]
print(last)   # 5

first, *middle, last = (1, 2, 3, 4, 5)
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5

# ใช้ _ สำหรับค่าที่ไม่ต้องการ
name, _, city = ("Alice", 25, "Bangkok")  # ไม่ต้องการ age
print(name, city)  # Alice Bangkok

# ใช้ *_ สำหรับหลายค่า
first, *_, last = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
print(first, last)  # 1 10
```

### 3.3 Nested Unpacking

```python
# Unpack nested tuple
nested = ((1, 2), (3, 4))
(a, b), (c, d) = nested
print(a, b, c, d)  # 1 2 3 4

# Unpack ใน for loop
data = [("Alice", (85, 92, 78)), ("Bob", (70, 65, 88))]
for name, (test1, test2, test3) in data:
    avg = (test1 + test2 + test3) / 3
    print(f"{name}: avg={avg:.1f}")
```

---

## 4. Swapping Variables

```python
# Python ทำได้ง่ายมากด้วย tuple unpacking
a, b = 10, 20
print(f"Before: a={a}, b={b}")  # a=10, b=20

a, b = b, a  # swap
print(f"After:  a={a}, b={b}")  # a=20, b=10

# ใน Python นี้คือ tuple assignment จริงๆ:
# 1. สร้าง tuple (b, a) = (20, 10)
# 2. Unpack เป็น a=20, b=10

# Swap 3 ตัวแปร
x, y, z = 1, 2, 3
x, y, z = z, x, y
print(x, y, z)  # 3 1 2

# ใช้ใน sorting algorithm (เช่น bubble sort)
def bubble_sort(lst):
    n = len(lst)
    arr = list(lst)  # copy เพื่อไม่แก้ต้นฉบับ
    for i in range(n):
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]  # swap
    return arr

print(bubble_sort([64, 34, 25, 12, 22, 11, 90]))
```

---

## 5. Tuple as Return Value

### 5.1 Return หลายค่า

```python
def min_max(numbers):
    """คืนค่าน้อยสุดและมากสุด"""
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 4, 1, 5, 9, 2, 6])
print(f"Min: {low}, Max: {high}")  # Min: 1, Max: 9

def divmod_custom(a, b):
    """คืนผลหารและเศษ"""
    return a // b, a % b

quotient, remainder = divmod_custom(17, 5)
print(f"17 ÷ 5 = {quotient} เศษ {remainder}")  # 17 ÷ 5 = 3 เศษ 2

# Python มี built-in divmod()
q, r = divmod(17, 5)
print(q, r)  # 3 2
```

### 5.2 ตัวอย่างจริง: Statistics

```python
import math

def statistics(data):
    """คำนวณ statistics และคืนหลายค่า"""
    n = len(data)
    mean = sum(data) / n
    
    variance = sum((x - mean) ** 2 for x in data) / n
    std_dev = math.sqrt(variance)
    
    sorted_data = sorted(data)
    if n % 2 == 0:
        median = (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2
    else:
        median = sorted_data[n//2]
    
    return mean, median, std_dev, min(data), max(data)

data = [23, 45, 12, 67, 34, 89, 56, 78, 90, 11]
mean, median, std, minimum, maximum = statistics(data)
print(f"Mean: {mean:.2f}")
print(f"Median: {median:.2f}")
print(f"Std Dev: {std:.2f}")
print(f"Range: {minimum} - {maximum}")
```

---

## 6. Named Tuples (collections.namedtuple)

### 6.1 การสร้าง Named Tuple

```python
from collections import namedtuple

# สร้าง namedtuple class
Point = namedtuple("Point", ["x", "y"])
Color = namedtuple("Color", ["red", "green", "blue"])
Person = namedtuple("Person", "name age city")  # string ก็ได้

# สร้าง instance
p = Point(3, 4)
red = Color(255, 0, 0)
alice = Person("Alice", 25, "Bangkok")

# เข้าถึงด้วยชื่อ
print(p.x, p.y)           # 3 4
print(red.red)             # 255
print(alice.name, alice.age)  # Alice 25

# เข้าถึงด้วย index (ยังทำได้)
print(p[0], p[1])          # 3 4
print(alice[0])            # Alice
```

### 6.2 Named Tuple Methods

```python
from collections import namedtuple

Student = namedtuple("Student", ["name", "grade", "gpa"])
s = Student("Bob", 12, 3.85)

# _fields - ดู field names
print(Student._fields)  # ('name', 'grade', 'gpa')

# _asdict() - แปลงเป็น dict
print(s._asdict())
# {'name': 'Bob', 'grade': 12, 'gpa': 3.85}

# _replace() - สร้าง instance ใหม่ด้วยค่าที่เปลี่ยน
updated = s._replace(gpa=3.95)
print(updated)    # Student(name='Bob', grade=12, gpa=3.95)
print(s)          # Student(name='Bob', grade=12, gpa=3.85)  ไม่เปลี่ยน!

# _make() - สร้างจาก iterable
data = ["Charlie", 11, 3.70]
charlie = Student._make(data)
print(charlie)  # Student(name='Charlie', grade=11, gpa=3.70)
```

### 6.3 Named Tuple กับ Default Values (Python 3.6.1+)

```python
from collections import namedtuple

# กำหนด defaults
Employee = namedtuple("Employee", ["name", "department", "salary"])
Employee.__new__.__defaults__ = ("Unknown", 30000)  # default สำหรับ 2 ค่าหลัง

emp1 = Employee("Alice", "Engineering", 80000)
emp2 = Employee("Bob")  # ใช้ default
emp3 = Employee("Charlie", "Marketing")

print(emp1)  # Employee(name='Alice', department='Engineering', salary=80000)
print(emp2)  # Employee(name='Bob', department='Unknown', salary=30000)
print(emp3)  # Employee(name='Charlie', department='Marketing', salary=30000)
```

### 6.4 ทำไมใช้ Named Tuple?

```python
from collections import namedtuple

# แบบ tuple ธรรมดา - ไม่ชัดเจน
student_raw = ("Alice", 85, "A", "Mathematics")
print(student_raw[0])  # ต้องจำว่า index 0 = ชื่อ

# แบบ Named Tuple - ชัดเจนมาก
Student = namedtuple("Student", ["name", "score", "grade", "subject"])
student = Student("Alice", 85, "A", "Mathematics")
print(student.name)     # อ่านเข้าใจทันที
print(student.score)
print(student.grade)

# ใช้แทน class เล็กๆ ที่ไม่ต้องการ methods
# ประหยัด memory กว่า class ธรรมดา
# immutable = thread-safe
```

### 6.5 ตัวอย่างจริง: Database Records

```python
from collections import namedtuple

# สร้าง "schema"
Product = namedtuple("Product", ["id", "name", "price", "stock", "category"])
Order = namedtuple("Order", ["order_id", "product", "quantity", "total"])

# "Database" records
products = [
    Product(1, "Laptop", 25000, 10, "Electronics"),
    Product(2, "Mouse", 500, 50, "Accessories"),
    Product(3, "Keyboard", 1200, 30, "Accessories"),
    Product(4, "Monitor", 8500, 15, "Electronics"),
    Product(5, "Headset", 2000, 20, "Accessories"),
]

# Query operations
def find_by_category(products, category):
    return [p for p in products if p.category == category]

def get_total_value(products):
    return sum(p.price * p.stock for p in products)

def find_affordable(products, max_price):
    return sorted(
        [p for p in products if p.price <= max_price],
        key=lambda p: p.price
    )

# Electronics
electronics = find_by_category(products, "Electronics")
print("Electronics:")
for p in electronics:
    print(f"  {p.name}: ฿{p.price:,} (stock: {p.stock})")

# Inventory value
print(f"\nมูลค่า inventory: ฿{get_total_value(products):,}")

# สินค้าไม่เกิน 2000
affordable = find_affordable(products, 2000)
print(f"\nสินค้าราคาไม่เกิน ฿2,000:")
for p in affordable:
    print(f"  {p.name}: ฿{p.price:,}")
```

---

## 7. Packing and Unpacking ขั้นสูง

### 7.1 Tuple Concatenation

```python
t1 = (1, 2, 3)
t2 = (4, 5, 6)
combined = t1 + t2
print(combined)  # (1, 2, 3, 4, 5, 6)

# Repetition
repeated = (1, 2) * 3
print(repeated)  # (1, 2, 1, 2, 1, 2)

# ไม่สามารถ += แบบ list ได้ (แต่ทำได้ด้วยการสร้างใหม่)
t = (1, 2, 3)
t += (4, 5)   # สร้าง tuple ใหม่ ไม่ใช่ modify ของเดิม
print(t)       # (1, 2, 3, 4, 5)
```

### 7.2 Tuple เป็น Dictionary Key

```python
# List ไม่สามารถเป็น key ได้ (unhashable)
# แต่ Tuple สามารถ (hashable)

# coordinates เป็น key
grid = {
    (0, 0): "origin",
    (1, 0): "right",
    (0, 1): "up",
    (-1, 0): "left",
    (0, -1): "down"
}

print(grid[(0, 0)])   # origin
print(grid[(1, 0)])   # right

# ตัวอย่าง: chess board
chess = {}
chess[("a", 1)] = "White Rook"
chess[("e", 1)] = "White King"
chess[("e", 8)] = "Black King"

for pos, piece in chess.items():
    col, row = pos
    print(f"{col}{row}: {piece}")
```

### 7.3 zip() กับ Tuples

```python
# zip คืน iterator ของ tuples
names = ("Alice", "Bob", "Charlie")
ages = (25, 30, 28)
cities = ("Bangkok", "Chiang Mai", "Phuket")

# รวม
combined = list(zip(names, ages, cities))
print(combined)
# [('Alice', 25, 'Bangkok'), ('Bob', 30, 'Chiang Mai'), ('Charlie', 28, 'Phuket')]

# Unzip
unzipped_names, unzipped_ages, unzipped_cities = zip(*combined)
print(unzipped_names)   # ('Alice', 'Bob', 'Charlie')
print(unzipped_ages)    # (25, 30, 28)
```

---

## 8. Performance Comparison

### 8.1 Tuple เร็วกว่า List

```python
import sys
import timeit

# ขนาด Memory
my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
my_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

print(f"List size:  {sys.getsizeof(my_list)} bytes")
print(f"Tuple size: {sys.getsizeof(my_tuple)} bytes")
# Tuple ใช้ memory น้อยกว่า

# Speed (creation)
list_time = timeit.timeit("[1, 2, 3, 4, 5]", number=1000000)
tuple_time = timeit.timeit("(1, 2, 3, 4, 5)", number=1000000)

print(f"\nList creation:  {list_time:.4f} seconds")
print(f"Tuple creation: {tuple_time:.4f} seconds")
# Tuple สร้างเร็วกว่า (Python cache tuple literals)
```

---

## 9. โปรแกรมจริง (Real-world Examples)

### 9.1 Coordinate System

```python
from collections import namedtuple
import math

Point = namedtuple("Point", ["x", "y"])
Point3D = namedtuple("Point3D", ["x", "y", "z"])

def distance(p1, p2):
    """คำนวณระยะห่างระหว่าง 2 จุด"""
    return math.sqrt((p2.x - p1.x)**2 + (p2.y - p1.y)**2)

def midpoint(p1, p2):
    """จุดกึ่งกลาง"""
    return Point((p1.x + p2.x) / 2, (p1.y + p2.y) / 2)

def slope(p1, p2):
    """ความชัน"""
    if p2.x == p1.x:
        return float('inf')
    return (p2.y - p1.y) / (p2.x - p1.x)

def perimeter(points):
    """คำนวณเส้นรอบรูป polygon"""
    n = len(points)
    total = sum(
        distance(points[i], points[(i+1) % n])
        for i in range(n)
    )
    return total

# ทดสอบ
A = Point(0, 0)
B = Point(3, 4)
C = Point(6, 0)

print(f"A = {A}")
print(f"B = {B}")
print(f"Distance A-B: {distance(A, B):.2f}")
print(f"Midpoint A-B: {midpoint(A, B)}")
print(f"Slope A-B: {slope(A, B):.2f}")
print(f"Triangle Perimeter: {perimeter([A, B, C]):.2f}")
```

### 9.2 Function Returning Multiple Values

```python
from collections import namedtuple

# ใช้ namedtuple เป็น return type
SearchResult = namedtuple("SearchResult", ["found", "index", "value"])
ValidationResult = namedtuple("ValidationResult", ["valid", "errors"])

def binary_search(arr, target):
    """Binary search ที่คืน namedtuple"""
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return SearchResult(found=True, index=mid, value=arr[mid])
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return SearchResult(found=False, index=-1, value=None)

def validate_user(username, password, email):
    """Validate user input"""
    errors = []
    
    if len(username) < 3:
        errors.append("Username ต้องมีอย่างน้อย 3 ตัวอักษร")
    if len(password) < 8:
        errors.append("Password ต้องมีอย่างน้อย 8 ตัวอักษร")
    if "@" not in email:
        errors.append("Email ไม่ถูกต้อง")
    
    return ValidationResult(valid=len(errors) == 0, errors=errors)

# ทดสอบ binary search
sorted_list = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
result = binary_search(sorted_list, 23)
print(f"Found: {result.found}, Index: {result.index}, Value: {result.value}")

result2 = binary_search(sorted_list, 50)
print(f"Found: {result2.found}")

# ทดสอบ validation
v = validate_user("Al", "pass", "not-an-email")
if not v.valid:
    print("\nข้อผิดพลาด:")
    for error in v.errors:
        print(f"  - {error}")

v2 = validate_user("alice", "password123", "alice@example.com")
print(f"\nValid: {v2.valid}")
```

### 9.3 เปรียบเทียบ Tuple กับ Class

```python
from collections import namedtuple
import sys

# วิธีที่ 1: Tuple ธรรมดา
point_raw = (3, 4)

# วิธีที่ 2: Named Tuple
Point = namedtuple("Point", ["x", "y"])
point_named = Point(3, 4)

# วิธีที่ 3: Class
class PointClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

point_class = PointClass(3, 4)

# Memory comparison
print(f"Tuple:       {sys.getsizeof(point_raw)} bytes")
print(f"Named Tuple: {sys.getsizeof(point_named)} bytes")
print(f"Class:       {sys.getsizeof(point_class)} bytes")

# Readability
print(f"\nTuple access: {point_raw[0]}, {point_raw[1]}")
print(f"Named access: {point_named.x}, {point_named.y}")
print(f"Class access: {point_class.x}, {point_class.y}")

# Named tuple ยังเป็น iterable
for coord in point_named:
    print(coord)
```

---

## 10. เมื่อไหรควรใช้ Tuple

```python
# 1. ข้อมูลที่ไม่ควรเปลี่ยน (constants)
RGB_RED = (255, 0, 0)
RGB_GREEN = (0, 255, 0)
SCREEN_SIZE = (1920, 1080)
PI_DIGITS = (3, 1, 4, 1, 5, 9, 2, 6)

# 2. Return หลายค่า
def get_location():
    return 13.7563, 100.5018  # lat, lng Bangkok

lat, lng = get_location()

# 3. Dictionary key (List ทำไม่ได้)
cache = {}
cache[(1, 2)] = "result 1"
cache[(3, 4)] = "result 2"

# 4. ข้อมูลที่เหมาะเป็น record
WEEKDAYS = ("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")

# 5. ป้องกันการ modify โดยไม่ตั้งใจ
def get_config():
    return ("localhost", 5432, "mydb", "postgres")  # ค่า config ที่ไม่ควรเปลี่ยน

# 6. Performance-critical code (tuple เร็วกว่า list)
# เมื่อสร้างและอ่านบ่อยๆ
```

---

## แบบฝึกหัด (Exercises)

### ข้อ 1: Tuple Statistics
```python
def tuple_stats(data):
    """คืน tuple (min, max, sum, avg)"""
    return min(data), max(data), sum(data), sum(data)/len(data)

result = tuple_stats((5, 3, 8, 1, 9, 2, 7, 4, 6))
minimum, maximum, total, average = result
print(f"Min={minimum}, Max={maximum}, Sum={total}, Avg={average:.2f}")
```

### ข้อ 2: Swap Without Temp
```python
def swap_all(*values):
    """สลับค่าทุกตัว"""
    lst = list(values)
    lst.reverse()
    return tuple(lst)

print(swap_all(1, 2, 3, 4, 5))  # (5, 4, 3, 2, 1)
```

### ข้อ 3: Named Tuple สำหรับ Playing Card
```python
from collections import namedtuple

Card = namedtuple("Card", ["suit", "rank"])

def create_deck():
    suits = ("Spades", "Hearts", "Diamonds", "Clubs")
    ranks = ("2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A")
    return [Card(suit, rank) for suit in suits for rank in ranks]

deck = create_deck()
print(f"Total cards: {len(deck)}")
print(f"First card: {deck[0]}")
print(f"Last card: {deck[-1]}")
# หา Aces
aces = [c for c in deck if c.rank == "A"]
print(f"Aces: {aces}")
```

### ข้อ 4: Unpack nested tuple
```python
def process_student_data(data):
    """
    data = (name, (score1, score2, score3))
    Return: (name, average, grade)
    """
    name, scores = data
    avg = sum(scores) / len(scores)
    grade = "A" if avg >= 90 else "B" if avg >= 80 else "C" if avg >= 70 else "D"
    return name, avg, grade

students = [
    ("Alice", (95, 88, 92)),
    ("Bob", (72, 68, 75)),
    ("Charlie", (85, 90, 87)),
]

for student_data in students:
    name, avg, grade = process_student_data(student_data)
    print(f"{name}: {avg:.1f} ({grade})")
```

### ข้อ 5: Tuple Compression
```python
def run_length_encode(data):
    """
    Run-length encoding: (1,1,1,2,2,3,3,3,3) → ((1,3),(2,2),(3,4))
    """
    if not data:
        return ()
    
    result = []
    current = data[0]
    count = 1
    
    for item in data[1:]:
        if item == current:
            count += 1
        else:
            result.append((current, count))
            current = item
            count = 1
    result.append((current, count))
    
    return tuple(result)

def run_length_decode(encoded):
    result = []
    for value, count in encoded:
        result.extend([value] * count)
    return tuple(result)

data = (1, 1, 1, 2, 2, 3, 3, 3, 3, 1, 1)
encoded = run_length_encode(data)
print(f"Encoded: {encoded}")  # ((1, 3), (2, 2), (3, 4), (1, 2))

decoded = run_length_decode(encoded)
print(f"Decoded: {decoded}")
print(f"Match: {data == decoded}")
```

### ข้อ 6: Named Tuple สำหรับ Employee
```python
from collections import namedtuple

Employee = namedtuple("Employee", ["id", "name", "department", "salary"])

employees = [
    Employee(1, "Alice", "Engineering", 75000),
    Employee(2, "Bob", "Marketing", 55000),
    Employee(3, "Charlie", "Engineering", 85000),
    Employee(4, "Diana", "HR", 50000),
    Employee(5, "Eve", "Engineering", 90000),
]

# 1. หาพนักงาน Engineering ทั้งหมด
eng = [e for e in employees if e.department == "Engineering"]
print("Engineering team:")
for e in eng:
    print(f"  {e.name}: ฿{e.salary:,}")

# 2. เงินเดือนเฉลี่ยของ Engineering
eng_avg = sum(e.salary for e in eng) / len(eng)
print(f"Avg salary: ฿{eng_avg:,.0f}")

# 3. พนักงานเงินเดือนสูงสุด
top = max(employees, key=lambda e: e.salary)
print(f"Highest paid: {top.name} (฿{top.salary:,})")
```

### ข้อ 7: Tuple ใน Set
```python
# Tuple สามารถใส่ใน set ได้ (เพราะ hashable)
visited_positions = set()

def move(pos, direction):
    x, y = pos
    moves = {"up": (0, 1), "down": (0, -1), "left": (-1, 0), "right": (1, 0)}
    dx, dy = moves[direction]
    return (x + dx, y + dy)

current = (0, 0)
visited_positions.add(current)

directions = ["up", "right", "right", "down", "left", "up", "left"]
for d in directions:
    current = move(current, d)
    visited_positions.add(current)

print(f"Current position: {current}")
print(f"Unique positions visited: {len(visited_positions)}")
print(f"Positions: {sorted(visited_positions)}")
```

### ข้อ 8: Fibonacci ด้วย Tuple Unpacking
```python
def fibonacci_gen(n):
    """Generate Fibonacci ด้วย tuple unpacking"""
    sequence = []
    a, b = 0, 1
    for _ in range(n):
        sequence.append(a)
        a, b = b, a + b  # tuple unpacking ทำ swap ได้สวยงาม
    return tuple(sequence)

fib = fibonacci_gen(15)
print(fib)  # (0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377)
```

### ข้อ 9: Named Tuple สำหรับ Color
```python
from collections import namedtuple

Color = namedtuple("Color", ["name", "r", "g", "b"])

COLORS = (
    Color("Red", 255, 0, 0),
    Color("Green", 0, 255, 0),
    Color("Blue", 0, 0, 255),
    Color("Yellow", 255, 255, 0),
    Color("Cyan", 0, 255, 255),
    Color("Magenta", 255, 0, 255),
    Color("White", 255, 255, 255),
    Color("Black", 0, 0, 0),
)

def to_hex(color):
    return f"#{color.r:02X}{color.g:02X}{color.b:02X}"

def brightness(color):
    return (color.r * 299 + color.g * 587 + color.b * 114) / 1000

for color in COLORS:
    bright = brightness(color)
    print(f"{color.name:10} {to_hex(color)}  brightness={bright:.0f}")
```

### ข้อ 10: Flight Information
```python
from collections import namedtuple

Flight = namedtuple("Flight", ["flight_no", "origin", "destination", "departure", "duration", "price"])

flights = [
    Flight("TG301", "BKK", "NRT", "08:00", 6.5, 15000),
    Flight("TG302", "BKK", "NRT", "14:00", 6.5, 12000),
    Flight("TG201", "BKK", "SIN", "07:00", 2.5, 5000),
    Flight("TG202", "BKK", "SIN", "16:00", 2.5, 4500),
    Flight("TG401", "BKK", "LHR", "23:00", 11.5, 35000),
    Flight("TG501", "BKK", "LAX", "22:30", 16.0, 45000),
]

def search_flights(destination):
    return [f for f in flights if f.destination == destination]

def cheapest_flight(destination):
    options = search_flights(destination)
    if not options:
        return None
    return min(options, key=lambda f: f.price)

def flights_under(max_price):
    return sorted(
        [f for f in flights if f.price <= max_price],
        key=lambda f: f.price
    )

print("เที่ยวบินไป Tokyo (NRT):")
for f in search_flights("NRT"):
    print(f"  {f.flight_no}: {f.departure} - ฿{f.price:,}")

cheapest = cheapest_flight("NRT")
print(f"\nถูกสุดไป Tokyo: {cheapest.flight_no} ฿{cheapest.price:,}")

print("\nเที่ยวบินราคาไม่เกิน ฿10,000:")
for f in flights_under(10000):
    print(f"  {f.flight_no} → {f.destination}: ฿{f.price:,}")
```

---

## สรุป (Summary)

### Tuple vs List

| เมื่อไหร่ | Tuple | List |
|-----------|-------|------|
| ข้อมูลไม่เปลี่ยน | ✓ | |
| ข้อมูลเปลี่ยนได้ | | ✓ |
| เป็น dict key | ✓ | |
| ใส่ใน set | ✓ | |
| Return หลายค่า | ✓ | |
| เพิ่ม/ลบ element | | ✓ |
| Memory น้อย | ✓ | |
| Performance สูง | ✓ | |

### Named Tuple ใช้เมื่อ
- ต้องการ tuple ที่อ่านโค้ดเข้าใจง่าย
- แทน class เล็กๆ ที่ไม่มี method
- เป็น return type ของ function
- ต้องการ immutable data structure ที่ self-documenting

> **หมายเหตุ:** Part ต่อไปจะเรียน Dictionaries ซึ่งเป็นโครงสร้างข้อมูล key-value ที่ใช้กันมากที่สุด
