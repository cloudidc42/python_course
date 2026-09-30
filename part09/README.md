# Part 09 - Functions: Basics

## สารบัญ

1. [การประกาศและเรียกใช้ Function](#1-การประกาศและเรียกใช้-function)
2. [Parameters vs Arguments](#2-parameters-vs-arguments)
3. [Default Parameters](#3-default-parameters)
4. [*args และ **kwargs](#4-args-และ-kwargs)
5. [Return Values](#5-return-values)
6. [Docstrings](#6-docstrings)
7. [Variable Scope (LEGB Rule)](#7-variable-scope-legb-rule)
8. [Global and Local Variables](#8-global-and-local-variables)
9. [Global Keyword](#9-global-keyword)
10. [Functions as First-Class Objects](#10-functions-as-first-class-objects)
11. [ตัวอย่างโปรแกรมจริง](#11-ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. การประกาศและเรียกใช้ Function

### ความหมาย

Function คือ block ของโค้ดที่ถูกตั้งชื่อ สามารถเรียกใช้ซ้ำได้หลายครั้ง ช่วยทำให้โค้ด:
- **Reusable** - ใช้ซ้ำได้
- **Readable** - อ่านง่ายขึ้น
- **Maintainable** - ดูแลรักษาง่าย
- **DRY** - Don't Repeat Yourself

### Syntax

```python
def function_name(parameters):
    """docstring"""
    # function body
    return value  # optional
```

### ตัวอย่างที่ 1: Function พื้นฐาน

```python
# Function ที่ไม่รับ parameter และไม่ return ค่า
def greet():
    print("สวัสดี!")
    print("ยินดีต้อนรับสู่ Python")

# เรียกใช้ function
greet()
greet()  # เรียกซ้ำได้
print(f"greet() คืนค่า: {greet()}")  # None เพราะไม่มี return
```

**Output:**
```
สวัสดี!
ยินดีต้อนรับสู่ Python
สวัสดี!
ยินดีต้อนรับสู่ Python
สวัสดี!
ยินดีต้อนรับสู่ Python
greet() คืนค่า: None
```

### ตัวอย่างที่ 2: Function พื้นฐานพร้อม parameter

```python
# Function ที่รับ parameter
def greet_person(name):
    print(f"สวัสดี {name}!")

def add(a, b):
    result = a + b
    return result

# เรียกใช้
greet_person("Alice")
greet_person("Bob")

sum_result = add(3, 5)
print(f"3 + 5 = {sum_result}")

# สามารถใช้ผลลัพธ์โดยตรง
print(f"10 + 20 = {add(10, 20)}")
```

### ตัวอย่างที่ 3: ทำไมต้องใช้ Function

```python
# ❌ ไม่ใช้ function (ซ้ำซ้อน)
print("=== นักศึกษา 1 ===")
name1 = "Alice"
score1 = 85
if score1 >= 80:
    grade1 = "A"
elif score1 >= 70:
    grade1 = "B"
else:
    grade1 = "C"
print(f"ชื่อ: {name1}, คะแนน: {score1}, เกรด: {grade1}")

print("\n=== นักศึกษา 2 ===")
name2 = "Bob"
score2 = 72
if score2 >= 80:
    grade2 = "A"
elif score2 >= 70:
    grade2 = "B"
else:
    grade2 = "C"
print(f"ชื่อ: {name2}, คะแนน: {score2}, เกรด: {grade2}")

# ✅ ใช้ function (DRY - Don't Repeat Yourself)
def show_student_info(name, score):
    if score >= 80:
        grade = "A"
    elif score >= 70:
        grade = "B"
    else:
        grade = "C"
    print(f"ชื่อ: {name}, คะแนน: {score}, เกรด: {grade}")

print("\n=== ใช้ Function ===")
show_student_info("Alice", 85)
show_student_info("Bob", 72)
show_student_info("Charlie", 91)
show_student_info("Diana", 65)
```

---

## 2. Parameters vs Arguments

### ความแตกต่าง

- **Parameter** คือตัวแปรที่ประกาศใน function definition
- **Argument** คือค่าที่ส่งให้ function เมื่อเรียกใช้

```python
def greet(name):  # name คือ Parameter
    print(f"Hello, {name}")

greet("Alice")  # "Alice" คือ Argument
```

### ประเภทของ Arguments

#### Positional Arguments

```python
# ตัวอย่างที่ 4: Positional Arguments
def describe_person(name, age, city):
    print(f"{name} อายุ {age} ปี อยู่ที่ {city}")

# ส่งตามลำดับ position
describe_person("Alice", 25, "Bangkok")
describe_person("Bob", 30, "Chiang Mai")

# ✗ ลำดับผิดจะได้ผลผิด
describe_person(25, "Alice", "Bangkok")  # ผิด! age กับ name สลับกัน
```

#### Keyword Arguments

```python
# ตัวอย่างที่ 5: Keyword Arguments
def describe_person(name, age, city):
    print(f"{name} อายุ {age} ปี อยู่ที่ {city}")

# ใช้ชื่อ parameter ระบุ
describe_person(name="Alice", age=25, city="Bangkok")

# ลำดับไม่สำคัญเมื่อใช้ keyword
describe_person(city="Phuket", name="Charlie", age=28)

# ผสม positional และ keyword (positional ต้องมาก่อน)
describe_person("Diana", age=22, city="Pattaya")
```

---

## 3. Default Parameters

### ความหมาย

Default parameter คือ parameter ที่มีค่าเริ่มต้น ถ้าไม่ส่งค่ามา จะใช้ค่า default แทน

### ตัวอย่างที่ 6: Default Parameters

```python
# ประกาศ function พร้อม default values
def greet(name, greeting="สวัสดี", language="Thai"):
    if language == "Thai":
        print(f"{greeting} คุณ{name}!")
    elif language == "English":
        print(f"Hello, {name}!")
    else:
        print(f"{greeting} {name}")

# เรียกแบบต่างๆ
greet("Alice")                          # ใช้ default ทั้งหมด
greet("Bob", "ดีสวัสดี")               # เปลี่ยน greeting
greet("Charlie", "Hi", "English")      # เปลี่ยนทั้งหมด
greet(name="Diana", language="English") # keyword + default
```

### ตัวอย่างที่ 7: Default Parameter ที่ซับซ้อน

```python
def create_user(
    username,
    email,
    role="user",
    is_active=True,
    max_login_attempts=3,
    timezone="Asia/Bangkok"
):
    """สร้าง user object"""
    return {
        "username": username,
        "email": email,
        "role": role,
        "is_active": is_active,
        "max_login_attempts": max_login_attempts,
        "timezone": timezone
    }

# สร้าง users แบบต่างๆ
user1 = create_user("alice", "alice@example.com")
user2 = create_user("admin", "admin@example.com", role="admin", max_login_attempts=5)
user3 = create_user("bob", "bob@example.com", is_active=False)

for user in [user1, user2, user3]:
    print(f"User: {user['username']} | Role: {user['role']} | Active: {user['is_active']}")
```

### ⚠️ ข้อควรระวัง: Mutable Default Arguments

```python
# ❌ อย่าใช้ mutable object เป็น default
def add_item_bad(item, items=[]):  # ❌ อันตราย!
    items.append(item)
    return items

print(add_item_bad("apple"))   # ["apple"]
print(add_item_bad("banana"))  # ["apple", "banana"] ← ผิด!
print(add_item_bad("cherry"))  # ["apple", "banana", "cherry"] ← ผิด!

# ✅ ใช้ None แล้วสร้าง list ใน function
def add_item_good(item, items=None):  # ✅ ถูกต้อง
    if items is None:
        items = []
    items.append(item)
    return items

print("\n✅ แบบที่ถูกต้อง:")
print(add_item_good("apple"))   # ["apple"]
print(add_item_good("banana"))  # ["banana"]
print(add_item_good("cherry"))  # ["cherry"]

# ส่ง list ที่มีอยู่แล้ว
my_list = ["existing"]
print(add_item_good("new", my_list))  # ["existing", "new"]
```

---

## 4. *args และ **kwargs

### *args - Positional Arguments แบบไม่จำกัดจำนวน

`*args` รับ positional arguments จำนวนใดก็ได้ เก็บเป็น tuple

```python
# ตัวอย่างที่ 8: *args
def sum_all(*numbers):
    """บวกตัวเลขทั้งหมดที่ส่งมา"""
    total = 0
    for num in numbers:
        total += num
    return total

print(sum_all(1, 2))           # 3
print(sum_all(1, 2, 3, 4, 5)) # 15
print(sum_all())               # 0
print(type(sum_all.__code__.co_varnames))

# ตัวอย่างอื่น
def print_info(title, *items):
    """แสดงรายการ"""
    print(f"\n{title}:")
    for i, item in enumerate(items, 1):
        print(f"  {i}. {item}")

print_info("ผลไม้", "apple", "banana", "cherry")
print_info("สี", "red", "green", "blue", "yellow", "purple")
```

### **kwargs - Keyword Arguments แบบไม่จำกัดจำนวน

`**kwargs` รับ keyword arguments จำนวนใดก็ได้ เก็บเป็น dictionary

```python
# ตัวอย่างที่ 9: **kwargs
def create_profile(**info):
    """สร้างโปรไฟล์จาก keyword arguments"""
    print("โปรไฟล์:")
    for key, value in info.items():
        print(f"  {key}: {value}")

create_profile(name="Alice", age=25, city="Bangkok", job="Engineer")
create_profile(name="Bob", email="bob@example.com", active=True)

# ตัวอย่าง: function configuration
def connect_database(**config):
    """เชื่อมต่อ database"""
    host = config.get("host", "localhost")
    port = config.get("port", 5432)
    database = config.get("database", "mydb")
    
    print(f"เชื่อมต่อ: {host}:{port}/{database}")
    return f"postgresql://{host}:{port}/{database}"

connect_database()
connect_database(host="db.example.com", port=5433, database="production")
```

### การรวม *args และ **kwargs

```python
# ตัวอย่างที่ 10: รวม *args และ **kwargs
def flexible_function(required, *args, **kwargs):
    """Function ที่ยืดหยุ่น"""
    print(f"Required: {required}")
    print(f"*args: {args} (ประเภท: {type(args).__name__})")
    print(f"**kwargs: {kwargs} (ประเภท: {type(kwargs).__name__})")

flexible_function("hello", 1, 2, 3, name="Alice", age=25)

print()

# Unpacking เมื่อเรียก function
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result1 = add(*numbers)  # Unpack list เป็น positional args
print(f"add(*{numbers}) = {result1}")

data = {"a": 10, "b": 20, "c": 30}
result2 = add(**data)    # Unpack dict เป็น keyword args
print(f"add(**{data}) = {result2}")
```

### ลำดับของ Parameters

```python
# ลำดับที่ถูกต้อง
def correct_order(pos1, pos2, *args, keyword_only, **kwargs):
    print(f"pos1={pos1}, pos2={pos2}")
    print(f"args={args}")
    print(f"keyword_only={keyword_only}")
    print(f"kwargs={kwargs}")

correct_order(1, 2, 3, 4, 5, keyword_only="required", extra="value")
```

---

## 5. Return Values

### return Statement

```python
# ตัวอย่างที่ 11: return ค่าเดียว
def square(n):
    return n ** 2

def cube(n):
    return n ** 3

print(f"5^2 = {square(5)}")
print(f"3^3 = {cube(3)}")

# ใช้ return value ต่อได้
result = square(4) + cube(2)
print(f"4^2 + 2^3 = {result}")
```

### Multiple Return Values

```python
# ตัวอย่างที่ 12: return หลายค่า
def min_max(numbers):
    """คืนค่าต่ำสุดและสูงสุด"""
    return min(numbers), max(numbers)

def divide_with_remainder(a, b):
    """คืนผลหารและเศษ"""
    quotient = a // b
    remainder = a % b
    return quotient, remainder

# รับหลายค่า
nums = [3, 7, 1, 9, 4, 6]
minimum, maximum = min_max(nums)
print(f"ต่ำสุด: {minimum}, สูงสุด: {maximum}")

q, r = divide_with_remainder(17, 5)
print(f"17 ÷ 5 = {q} เศษ {r}")

# หรือรับเป็น tuple
result = min_max(nums)
print(f"ผลลัพธ์เป็น tuple: {result}")
print(f"ประเภท: {type(result)}")
```

### ตัวอย่างที่ 13: Return dictionary

```python
def analyze_text(text):
    """วิเคราะห์ข้อความและคืน dict"""
    words = text.split()
    sentences = text.count('.') + text.count('!') + text.count('?')
    
    return {
        "char_count": len(text),
        "word_count": len(words),
        "sentence_count": sentences,
        "avg_word_length": sum(len(w) for w in words) / len(words) if words else 0,
        "unique_words": len(set(words)),
    }

text = "Python is great. Python is easy to learn! Python is popular."
stats = analyze_text(text)

print("สถิติข้อความ:")
for key, value in stats.items():
    print(f"  {key}: {value:.2f}" if isinstance(value, float) else f"  {key}: {value}")
```

### Early Return Pattern

```python
# ตัวอย่างที่ 14: Early Return
def validate_email(email):
    """ตรวจสอบ email ด้วย early return"""
    if not email:
        return False, "email ว่างเปล่า"
    
    if "@" not in email:
        return False, "ไม่มีเครื่องหมาย @"
    
    parts = email.split("@")
    if len(parts) != 2:
        return False, "มีเครื่องหมาย @ มากกว่าหนึ่งตัว"
    
    local, domain = parts
    
    if not local:
        return False, "ไม่มีส่วน local"
    
    if "." not in domain:
        return False, "domain ไม่ถูกต้อง"
    
    domain_parts = domain.split(".")
    if any(len(p) == 0 for p in domain_parts):
        return False, "domain ไม่ถูกต้อง"
    
    return True, "email ถูกต้อง"

# ทดสอบ
emails = [
    "user@example.com",
    "",
    "notanemail",
    "user@@example.com",
    "@example.com",
    "user@",
    "user@.com",
]

for email in emails:
    is_valid, message = validate_email(email)
    print(f"'{email}': {'✓' if is_valid else '✗'} {message}")
```

---

## 6. Docstrings

### ความหมาย

Docstring คือ string ที่อยู่บรรทัดแรกของ function ใช้อธิบายการทำงาน สามารถเข้าถึงได้ผ่าน `.__doc__`

### Docstring Formats

```python
# ตัวอย่างที่ 15: Docstring styles

# Style 1: One-liner
def add(a, b):
    """บวกสองตัวเลขเข้าด้วยกัน"""
    return a + b

# Style 2: Google Style
def calculate_bmi(weight_kg, height_m):
    """คำนวณ Body Mass Index (BMI)
    
    Args:
        weight_kg (float): น้ำหนักเป็น กิโลกรัม
        height_m (float): ส่วนสูงเป็น เมตร
    
    Returns:
        float: ค่า BMI
    
    Raises:
        ValueError: ถ้าน้ำหนักหรือส่วนสูงเป็น 0 หรือน้อยกว่า
    
    Examples:
        >>> calculate_bmi(70, 1.75)
        22.86
        >>> calculate_bmi(60, 1.60)
        23.44
    """
    if weight_kg <= 0:
        raise ValueError("น้ำหนักต้องมากกว่า 0")
    if height_m <= 0:
        raise ValueError("ส่วนสูงต้องมากกว่า 0")
    
    return weight_kg / (height_m ** 2)

# เข้าถึง docstring
print(calculate_bmi.__doc__)

# ใช้ help()
# help(calculate_bmi)

# ทดสอบ
print(f"\nBMI (70kg, 1.75m): {calculate_bmi(70, 1.75):.2f}")
```

### ตัวอย่างที่ 16: Docstring สำหรับโปรแกรมจริง

```python
def sort_students(students, key="score", reverse=True):
    """เรียงรายชื่อนักศึกษาตามเกณฑ์ที่กำหนด
    
    Args:
        students (list[dict]): รายการนักศึกษา แต่ละคนมี 'name' และ 'score'
        key (str): เกณฑ์การเรียง ('name' หรือ 'score') default='score'
        reverse (bool): True=จากมากไปน้อย, False=น้อยไปมาก default=True
    
    Returns:
        list[dict]: รายการนักศึกษาที่เรียงแล้ว
    
    Examples:
        >>> students = [{"name": "Bob", "score": 85}, {"name": "Alice", "score": 92}]
        >>> sort_students(students, key='score')
        [{'name': 'Alice', 'score': 92}, {'name': 'Bob', 'score': 85}]
    """
    if key not in ("name", "score"):
        raise ValueError(f"key ต้องเป็น 'name' หรือ 'score' ไม่ใช่ '{key}'")
    
    return sorted(students, key=lambda s: s[key], reverse=reverse)

# ทดสอบ
students = [
    {"name": "Charlie", "score": 78},
    {"name": "Alice", "score": 92},
    {"name": "Bob", "score": 85},
    {"name": "Diana", "score": 88},
]

sorted_by_score = sort_students(students)
print("เรียงตามคะแนน (มาก->น้อย):")
for s in sorted_by_score:
    print(f"  {s['name']}: {s['score']}")

sorted_by_name = sort_students(students, key="name", reverse=False)
print("\nเรียงตามชื่อ (ก-ฮ):")
for s in sorted_by_name:
    print(f"  {s['name']}: {s['score']}")
```

---

## 7. Variable Scope (LEGB Rule)

### LEGB Rule

Python ค้นหาตัวแปรตามลำดับ:
- **L** - Local: ภายใน function
- **E** - Enclosing: ใน function ที่ครอบอยู่ (สำหรับ nested functions)
- **G** - Global: ระดับ module
- **B** - Built-in: Python built-ins เช่น print, len, range

### ตัวอย่างที่ 17: LEGB Scope

```python
# Built-in scope
x = len("hello")  # len เป็น built-in

# Global scope
global_var = "ฉันอยู่ใน Global Scope"

def outer_function():
    # Enclosing scope
    enclosing_var = "ฉันอยู่ใน Enclosing Scope"
    
    def inner_function():
        # Local scope
        local_var = "ฉันอยู่ใน Local Scope"
        
        # LEGB: ค้นหาตามลำดับ L -> E -> G -> B
        print(f"Local: {local_var}")
        print(f"Enclosing: {enclosing_var}")  # จาก Enclosing scope
        print(f"Global: {global_var}")        # จาก Global scope
        print(f"Built-in: {len('test')}")     # Built-in function
    
    inner_function()

outer_function()
```

### ตัวอย่างที่ 18: ชื่อตัวแปรซ้ำกัน (Shadowing)

```python
# ตัวแปรใน local scope บัง global scope
x = "global"

def show_x():
    x = "local"    # สร้าง local x ใหม่ (ไม่แก้ global)
    print(f"ใน function: x = '{x}'")

show_x()
print(f"นอก function: x = '{x}'")  # global ยังคงเดิม

print()

# ตัวอย่างที่ซับซ้อนกว่า
value = 100  # global

def process():
    value = 200  # local (ไม่ใช่ global)
    print(f"ใน process(): value = {value}")

process()
print(f"หลัง process(): value = {value}")  # ยังคงเป็น 100
```

---

## 8. Global and Local Variables

### ตัวอย่างที่ 19: Global vs Local

```python
# Global variable
total_count = 0  # Global
items = []       # Global

def add_item(item):
    """เพิ่มสินค้า (ไม่แก้ global โดยตรง)"""
    # items.append(item)  # ทำได้! เพราะแก้ภายใน object
    # แต่ไม่สามารถ reassign items = [] โดยไม่ใช้ global
    items.append(item)  # แก้ object ที่ global ชี้อยู่

def get_count():
    """คืนจำนวนสินค้า"""
    local_count = len(items)  # local variable
    return local_count

add_item("apple")
add_item("banana")
add_item("cherry")

print(f"สินค้า: {items}")
print(f"จำนวน: {get_count()}")

# Function ไม่สามารถ reassign global list โดยไม่ใช้ global keyword
def clear_items():
    # items = []  # สร้าง local ใหม่ ไม่ได้แก้ global
    items.clear()  # แก้ global object

clear_items()
print(f"หลัง clear: {items}")
```

---

## 9. Global Keyword

### ตัวอย่างที่ 20: global keyword

```python
# ใช้ global keyword เพื่อแก้ไข global variable
counter = 0  # global

def increment():
    global counter  # บอก Python ว่าใช้ global counter
    counter += 1

def reset():
    global counter
    counter = 0

print(f"เริ่มต้น: counter = {counter}")
increment()
increment()
increment()
print(f"หลัง increment 3 ครั้ง: counter = {counter}")
reset()
print(f"หลัง reset: counter = {counter}")

# ตัวอย่างที่ใช้งานจริง
total_sales = 0.0

def process_sale(amount):
    global total_sales
    total_sales += amount
    print(f"  ขาย {amount:.2f} บาท (รวม: {total_sales:.2f} บาท)")

print("\nประมวลผลการขาย:")
process_sale(150.00)
process_sale(320.50)
process_sale(75.00)
print(f"ยอดขายรวม: {total_sales:.2f} บาท")
```

### ตัวอย่างที่ 21: nonlocal keyword

```python
# nonlocal สำหรับ nested functions
def make_counter(start=0):
    """สร้าง counter function"""
    count = start
    
    def increment(step=1):
        nonlocal count  # ใช้ count จาก enclosing scope
        count += step
        return count
    
    def decrement(step=1):
        nonlocal count
        count -= step
        return count
    
    def get():
        return count  # อ่านได้โดยไม่ต้อง nonlocal
    
    def reset():
        nonlocal count
        count = start
    
    return increment, decrement, get, reset

# สร้าง counter
inc, dec, get_val, rst = make_counter(10)

print(f"เริ่มต้น: {get_val()}")
print(f"increment: {inc()}")
print(f"increment(5): {inc(5)}")
print(f"decrement: {dec()}")
print(f"reset: ", end="")
rst()
print(get_val())
```

---

## 10. Functions as First-Class Objects

### ความหมาย

ใน Python, functions เป็น "first-class objects" หมายความว่า:
- เก็บในตัวแปรได้
- ส่งเป็น argument ให้ function อื่นได้
- คืนค่าจาก function ได้
- เก็บใน data structure ได้

### ตัวอย่างที่ 22: เก็บ function ในตัวแปร

```python
# เก็บ function ในตัวแปร
def add(a, b): return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b

# เก็บใน dictionary
operations = {
    "add": add,
    "subtract": subtract,
    "multiply": multiply,
}

# เรียกผ่าน dictionary
for name, func in operations.items():
    result = func(10, 3)
    print(f"operations['{name}'](10, 3) = {result}")

print()

# เก็บใน list
func_list = [add, subtract, multiply]
for func in func_list:
    print(f"{func.__name__}(5, 2) = {func(5, 2)}")
```

### ตัวอย่างที่ 23: ส่ง function เป็น argument

```python
# Higher-order function: รับ function เป็น argument
def apply_operation(numbers, operation):
    """ใช้ operation กับแต่ละตัวเลข"""
    return [operation(n) for n in numbers]

def square(n): return n ** 2
def cube(n): return n ** 3
def double(n): return n * 2
def is_even(n): return n % 2 == 0

numbers = [1, 2, 3, 4, 5]
print(f"Numbers: {numbers}")
print(f"Squared: {apply_operation(numbers, square)}")
print(f"Cubed:   {apply_operation(numbers, cube)}")
print(f"Doubled: {apply_operation(numbers, double)}")
print(f"IsEven:  {apply_operation(numbers, is_even)}")
```

### ตัวอย่างที่ 24: Lambda Functions

```python
# Lambda: anonymous function
square_lambda = lambda x: x ** 2
print(f"lambda x: x**2 => {square_lambda(5)}")

# ใช้กับ sorted
students = [
    {"name": "Charlie", "score": 78, "age": 20},
    {"name": "Alice", "score": 92, "age": 22},
    {"name": "Bob", "score": 85, "age": 19},
]

# เรียงตาม score
by_score = sorted(students, key=lambda s: s["score"], reverse=True)
print("\nเรียงตาม score:")
for s in by_score:
    print(f"  {s['name']}: {s['score']}")

# เรียงตาม name
by_name = sorted(students, key=lambda s: s["name"])
print("\nเรียงตาม name:")
for s in by_name:
    print(f"  {s['name']}")

# Lambda กับ map, filter
nums = range(1, 11)
evens = list(filter(lambda x: x % 2 == 0, nums))
squares = list(map(lambda x: x**2, evens))
print(f"\nเลขคู่: {evens}")
print(f"ยกกำลัง 2: {squares}")
```

### ตัวอย่างที่ 25: Function ที่ return Function

```python
# Function ที่สร้างและคืน function
def make_multiplier(factor):
    """สร้าง function ที่คูณด้วย factor"""
    def multiplier(x):
        return x * factor
    return multiplier

# สร้าง multiplier functions
double = make_multiplier(2)
triple = make_multiplier(3)
times_ten = make_multiplier(10)

print(f"double(5) = {double(5)}")
print(f"triple(4) = {triple(4)}")
print(f"times_ten(7) = {times_ten(7)}")

# ใช้กับ map
numbers = [1, 2, 3, 4, 5]
doubled_nums = list(map(double, numbers))
print(f"\n{numbers} doubled: {doubled_nums}")
```

---

## 11. ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Utility Functions Library

```python
"""
ชุด Utility Functions สำหรับใช้งานทั่วไป
"""

# === String Utilities ===
def truncate(text, max_length, suffix="..."):
    """ตัด string ให้ไม่เกิน max_length
    
    Args:
        text: ข้อความต้นฉบับ
        max_length: ความยาวสูงสุด
        suffix: ส่วนท้ายที่เพิ่ม (default: "...")
    
    Returns:
        str: ข้อความที่ตัดแล้ว
    """
    if len(text) <= max_length:
        return text
    return text[:max_length - len(suffix)] + suffix

def slugify(text):
    """แปลง text เป็น URL-friendly slug
    
    Example: "Hello World!" -> "hello-world"
    """
    import re
    text = text.lower().strip()
    text = re.sub(r'[^\w\s-]', '', text)
    text = re.sub(r'[\s_-]+', '-', text)
    text = re.sub(r'^-+|-+$', '', text)
    return text

def is_palindrome(text):
    """ตรวจสอบว่าข้อความเป็น palindrome หรือไม่"""
    cleaned = "".join(c.lower() for c in text if c.isalnum())
    return cleaned == cleaned[::-1]

def count_words(text):
    """นับจำนวนคำในข้อความ"""
    return len(text.split())

def title_case(text):
    """แปลงเป็น Title Case แบบ smart"""
    small_words = {"a", "an", "the", "and", "but", "or", "for", "nor", "in", "on", "at"}
    words = text.lower().split()
    result = []
    for i, word in enumerate(words):
        if i == 0 or word not in small_words:
            result.append(word.capitalize())
        else:
            result.append(word)
    return " ".join(result)

# === Number Utilities ===
def clamp(value, min_val, max_val):
    """จำกัดค่าให้อยู่ระหว่าง min_val และ max_val"""
    return max(min_val, min(max_val, value))

def percentage(part, whole, decimals=2):
    """คำนวณเปอร์เซ็นต์"""
    if whole == 0:
        return 0
    return round((part / whole) * 100, decimals)

def format_number(number, prefix="", suffix="", decimal_places=2):
    """จัดรูปแบบตัวเลข"""
    formatted = f"{number:,.{decimal_places}f}"
    return f"{prefix}{formatted}{suffix}"

# === List Utilities ===
def flatten(nested_list):
    """ทำให้ nested list เป็น flat list"""
    result = []
    for item in nested_list:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

def chunk(lst, size):
    """แบ่ง list เป็น chunks ขนาด size"""
    return [lst[i:i+size] for i in range(0, len(lst), size)]

def remove_duplicates(lst):
    """ลบ duplicate โดยรักษาลำดับ"""
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

# ทดสอบทุก functions
print("=== String Utilities ===")
print(truncate("Hello World, this is a long text", 20))
print(slugify("Hello World! This is Python 3.10"))
print(is_palindrome("racecar"), is_palindrome("hello"))
print(count_words("The quick brown fox"))
print(title_case("the lord of the rings"))

print("\n=== Number Utilities ===")
print(clamp(15, 0, 10))
print(clamp(-5, 0, 10))
print(clamp(5, 0, 10))
print(percentage(75, 200))
print(format_number(1234567.89, prefix="฿ ", suffix=" บาท"))

print("\n=== List Utilities ===")
print(flatten([1, [2, 3], [4, [5, 6]], 7]))
print(chunk([1,2,3,4,5,6,7,8,9], 3))
print(remove_duplicates([1, 2, 3, 2, 4, 3, 5, 1]))
```

### โปรแกรมที่ 2: Math Functions

```python
"""
ชุด Math Functions
"""
import math

def is_prime(n):
    """ตรวจสอบว่า n เป็นจำนวนเฉพาะ"""
    if n < 2: return False
    if n == 2: return True
    if n % 2 == 0: return False
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        if n % i == 0: return False
    return True

def prime_factors(n):
    """หา prime factors ของ n"""
    factors = []
    d = 2
    while d * d <= n:
        while n % d == 0:
            factors.append(d)
            n //= d
        d += 1
    if n > 1:
        factors.append(n)
    return factors

def gcd(a, b):
    """Greatest Common Divisor (Euclidean algorithm)"""
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    """Least Common Multiple"""
    return abs(a * b) // gcd(a, b)

def fibonacci(n):
    """สร้าง Fibonacci sequence n ตัว"""
    if n <= 0: return []
    if n == 1: return [0]
    seq = [0, 1]
    while len(seq) < n:
        seq.append(seq[-1] + seq[-2])
    return seq

def power_of_two(n):
    """ตรวจสอบว่า n เป็นกำลังสองหรือไม่"""
    return n > 0 and (n & (n - 1)) == 0

def digits_sum(n):
    """บวกหลักของตัวเลข"""
    return sum(int(d) for d in str(abs(n)))

# ทดสอบ
print("=== Prime Numbers ===")
primes = [n for n in range(2, 50) if is_prime(n)]
print(f"จำนวนเฉพาะถึง 50: {primes}")

print("\n=== Prime Factors ===")
for n in [12, 36, 100, 97, 360]:
    factors = prime_factors(n)
    print(f"  {n} = {'×'.join(map(str, factors))}")

print("\n=== GCD & LCM ===")
pairs = [(12, 18), (100, 75), (7, 13)]
for a, b in pairs:
    print(f"  GCD({a},{b})={gcd(a,b)}, LCM({a},{b})={lcm(a,b)}")

print("\n=== Fibonacci ===")
fib = fibonacci(10)
print(f"  F(10) = {fib}")

print("\n=== Power of 2 ===")
for n in [1, 2, 4, 8, 16, 15, 100]:
    print(f"  {n}: {'✓' if power_of_two(n) else '✗'}")
```

### โปรแกรมที่ 3: String Processing

```python
"""
String Processing Functions
"""

def reverse_words(sentence):
    """กลับลำดับคำในประโยค"""
    return " ".join(sentence.split()[::-1])

def count_char_types(text):
    """นับจำนวนตัวอักษรแต่ละประเภท"""
    return {
        "uppercase": sum(1 for c in text if c.isupper()),
        "lowercase": sum(1 for c in text if c.islower()),
        "digits": sum(1 for c in text if c.isdigit()),
        "spaces": sum(1 for c in text if c.isspace()),
        "special": sum(1 for c in text if not c.isalnum() and not c.isspace()),
        "total": len(text)
    }

def compress_string(text):
    """Run-length encoding compression"""
    if not text:
        return ""
    
    result = []
    count = 1
    
    for i in range(1, len(text)):
        if text[i] == text[i-1]:
            count += 1
        else:
            result.append(f"{text[i-1]}{count if count > 1 else ''}")
            count = 1
    
    result.append(f"{text[-1]}{count if count > 1 else ''}")
    compressed = "".join(result)
    return compressed if len(compressed) < len(text) else text

def find_all_occurrences(text, pattern):
    """หาตำแหน่งทั้งหมดที่ pattern ปรากฏ"""
    positions = []
    start = 0
    while True:
        pos = text.find(pattern, start)
        if pos == -1:
            break
        positions.append(pos)
        start = pos + 1
    return positions

def word_wrap(text, width=40):
    """จัดข้อความให้ไม่เกิน width ตัวอักษรต่อบรรทัด"""
    words = text.split()
    lines = []
    current_line = []
    current_length = 0
    
    for word in words:
        if current_length + len(word) + (1 if current_line else 0) <= width:
            current_line.append(word)
            current_length += len(word) + (1 if len(current_line) > 1 else 0)
        else:
            if current_line:
                lines.append(" ".join(current_line))
            current_line = [word]
            current_length = len(word)
    
    if current_line:
        lines.append(" ".join(current_line))
    
    return "\n".join(lines)

# ทดสอบ
print("=== String Processing ===")

print("\nreverse_words:")
print(f"  '{reverse_words('Hello World Python')}'")

print("\ncount_char_types:")
text = "Hello, World! 123"
stats = count_char_types(text)
for key, val in stats.items():
    print(f"  {key}: {val}")

print("\ncompress_string:")
for s in ["aabbbcccc", "aaabbb", "abcde", "aaaaaa"]:
    compressed = compress_string(s)
    print(f"  '{s}' -> '{compressed}'")

print("\nfind_all_occurrences:")
text = "the cat sat on the mat by the gate"
positions = find_all_occurrences(text, "the")
print(f"  'the' ใน '{text}'")
print(f"  ตำแหน่ง: {positions}")

print("\nword_wrap:")
long_text = "Python is a high-level general-purpose programming language that emphasizes code readability and simplicity."
print(word_wrap(long_text, 40))
```

---

## 12. แบบฝึกหัด

### แบบฝึกหัดข้อที่ 1: Temperature Converter

```
จงเขียน function แปลงอุณหภูมิระหว่าง Celsius, Fahrenheit, Kelvin
```

**เฉลย:**

```python
def celsius_to_fahrenheit(c):
    return (c * 9/5) + 32

def fahrenheit_to_celsius(f):
    return (f - 32) * 5/9

def celsius_to_kelvin(c):
    return c + 273.15

def kelvin_to_celsius(k):
    return k - 273.15

def convert_temperature(value, from_unit, to_unit):
    """แปลงอุณหภูมิ
    
    Args:
        value: ค่าอุณหภูมิ
        from_unit: หน่วยต้นทาง ('C', 'F', 'K')
        to_unit: หน่วยปลายทาง ('C', 'F', 'K')
    """
    # แปลงเป็น Celsius ก่อน
    if from_unit == "C":
        celsius = value
    elif from_unit == "F":
        celsius = fahrenheit_to_celsius(value)
    elif from_unit == "K":
        celsius = kelvin_to_celsius(value)
    else:
        raise ValueError(f"หน่วยไม่รู้จัก: {from_unit}")
    
    # แปลงจาก Celsius ไปหน่วยปลายทาง
    if to_unit == "C":
        return celsius
    elif to_unit == "F":
        return celsius_to_fahrenheit(celsius)
    elif to_unit == "K":
        return celsius_to_kelvin(celsius)
    else:
        raise ValueError(f"หน่วยไม่รู้จัก: {to_unit}")

# ทดสอบ
conversions = [
    (100, "C", "F"),
    (212, "F", "C"),
    (0, "C", "K"),
    (273.15, "K", "C"),
    (37, "C", "F"),
]

for value, from_u, to_u in conversions:
    result = convert_temperature(value, from_u, to_u)
    print(f"{value}°{from_u} = {result:.2f}°{to_u}")
```

---

### แบบฝึกหัดข้อที่ 2: Statistics Functions

```
จงเขียน function คำนวณสถิติ: mean, variance, std_dev, z_score
```

**เฉลย:**

```python
def mean(data):
    """คำนวณค่าเฉลี่ย"""
    return sum(data) / len(data) if data else 0

def variance(data, population=True):
    """คำนวณ variance"""
    if not data:
        return 0
    avg = mean(data)
    n = len(data) if population else len(data) - 1
    if n == 0:
        return 0
    return sum((x - avg) ** 2 for x in data) / n

def std_dev(data, population=True):
    """คำนวณ standard deviation"""
    return variance(data, population) ** 0.5

def z_score(value, data):
    """คำนวณ z-score ของค่าหนึ่งในชุดข้อมูล"""
    avg = mean(data)
    sd = std_dev(data)
    if sd == 0:
        return 0
    return (value - avg) / sd

# ทดสอบ
scores = [85, 92, 78, 95, 88, 72, 90, 83, 87, 91]
print(f"Data: {scores}")
print(f"Mean: {mean(scores):.2f}")
print(f"Variance (pop): {variance(scores):.2f}")
print(f"Std Dev (pop): {std_dev(scores):.2f}")
print("\nZ-scores:")
for score in scores:
    z = z_score(score, scores)
    print(f"  {score}: z = {z:.2f}")
```

---

### แบบฝึกหัดข้อที่ 3: List Operations

```
จงเขียน function จัดการ list: merge_sorted, binary_search, rotate
```

**เฉลย:**

```python
def merge_sorted(list1, list2):
    """รวม 2 sorted lists เป็น 1 sorted list"""
    result = []
    i = j = 0
    
    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1
    
    result.extend(list1[i:])
    result.extend(list2[j:])
    return result

def binary_search(sorted_list, target):
    """ค้นหาด้วย Binary Search
    
    Returns:
        int: index ของ target หรือ -1 ถ้าไม่พบ
    """
    left, right = 0, len(sorted_list) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if sorted_list[mid] == target:
            return mid
        elif sorted_list[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

def rotate(lst, k):
    """หมุน list ไป k ตำแหน่ง (บวก=ขวา, ลบ=ซ้าย)"""
    if not lst:
        return lst
    k = k % len(lst)
    return lst[-k:] + lst[:-k] if k else lst[:]

# ทดสอบ
print("=== merge_sorted ===")
a = [1, 3, 5, 7, 9]
b = [2, 4, 6, 8, 10]
print(f"{a} + {b} = {merge_sorted(a, b)}")

print("\n=== binary_search ===")
sorted_list = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
for target in [23, 16, 91, 50]:
    idx = binary_search(sorted_list, target)
    print(f"  search {target}: {'index '+str(idx) if idx != -1 else 'ไม่พบ'}")

print("\n=== rotate ===")
lst = [1, 2, 3, 4, 5]
for k in [0, 1, 2, -1, 7]:
    print(f"  rotate({lst}, {k}) = {rotate(lst, k)}")
```

---

### แบบฝึกหัดข้อที่ 4: Validators

```
จงเขียน validation functions สำหรับข้อมูลต่างๆ
```

**เฉลย:**

```python
import re

def validate_phone(phone):
    """ตรวจสอบเบอร์โทรศัพท์ไทย"""
    pattern = r'^(0[689]\d{8}|0\d{8,9})$'
    clean = phone.replace("-", "").replace(" ", "")
    if re.match(pattern, clean):
        return True, "เบอร์โทรถูกต้อง"
    return False, "เบอร์โทรไม่ถูกต้อง (ต้องขึ้นต้นด้วย 0 และมี 9-10 หลัก)"

def validate_thai_id(id_number):
    """ตรวจสอบเลขบัตรประชาชนไทย"""
    id_num = str(id_number).replace("-", "").replace(" ", "")
    
    if len(id_num) != 13:
        return False, "ต้องมี 13 หลัก"
    
    if not id_num.isdigit():
        return False, "ต้องเป็นตัวเลขเท่านั้น"
    
    # Checksum
    total = sum(int(id_num[i]) * (13 - i) for i in range(12))
    check_digit = (11 - (total % 11)) % 10
    
    if check_digit == int(id_num[12]):
        return True, "เลขบัตรถูกต้อง"
    return False, "check digit ผิด"

def validate_url(url):
    """ตรวจสอบ URL"""
    pattern = r'^https?://[^\s/$.?#].[^\s]*$'
    if re.match(pattern, url, re.IGNORECASE):
        return True, "URL ถูกต้อง"
    return False, "URL ไม่ถูกต้อง"

# ทดสอบ
print("=== Phone Validation ===")
phones = ["0812345678", "0912345678", "081234567", "1234567890"]
for phone in phones:
    valid, msg = validate_phone(phone)
    print(f"  {phone}: {'✓' if valid else '✗'} {msg}")

print("\n=== URL Validation ===")
urls = [
    "https://www.example.com",
    "http://test.co.th/path?query=1",
    "not-a-url",
    "ftp://invalid.com",
]
for url in urls:
    valid, msg = validate_url(url)
    print(f"  {url[:30]}: {'✓' if valid else '✗'} {msg}")
```

---

### แบบฝึกหัดข้อที่ 5: Decorator Function

```
จงเขียน decorator function สำหรับ:
- วัดเวลาการทำงานของ function
- จำกัดจำนวนครั้งที่เรียกใช้
```

**เฉลย:**

```python
import time
import functools

def timer(func):
    """Decorator วัดเวลาการทำงาน"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} ใช้เวลา {(end-start)*1000:.3f} ms")
        return result
    return wrapper

def call_limit(max_calls):
    """Decorator จำกัดจำนวนครั้งที่เรียก"""
    def decorator(func):
        calls = [0]  # ใช้ list เพื่อ mutability
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if calls[0] >= max_calls:
                raise RuntimeError(f"{func.__name__} เรียกได้สูงสุด {max_calls} ครั้งเท่านั้น")
            calls[0] += 1
            print(f"  (ครั้งที่ {calls[0]}/{max_calls})")
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

@timer
def slow_calculation(n):
    """จำลอง calculation ช้า"""
    total = 0
    for i in range(n):
        total += i ** 2
    return total

@call_limit(3)
def limited_function(x):
    return x * 2

# ทดสอบ timer
result = slow_calculation(100000)
print(f"Result: {result:,}")

# ทดสอบ call_limit
print("\nTest call_limit(3):")
for i in range(4):
    try:
        result = limited_function(i)
        print(f"  limited_function({i}) = {result}")
    except RuntimeError as e:
        print(f"  Error: {e}")
```

---

### แบบฝึกหัดข้อที่ 6 - 10 (สรุปสั้น)

```python
# ข้อ 6: Number Formatter
def format_currency(amount, currency="THB", decimals=2):
    """จัดรูปแบบสกุลเงิน"""
    symbols = {"THB": "฿", "USD": "$", "EUR": "€", "GBP": "£"}
    symbol = symbols.get(currency, currency)
    return f"{symbol}{amount:,.{decimals}f}"

print(format_currency(1234567.89))
print(format_currency(9999.99, "USD"))

# ข้อ 7: Text Analyzer
def text_readability(text):
    """วิเคราะห์ความยากของข้อความ (Flesch-Kincaid approximation)"""
    words = text.split()
    sentences = text.count('.') + text.count('!') + text.count('?')
    if sentences == 0: sentences = 1
    
    syllables = sum(max(1, len([c for c in w.lower() if c in 'aeiou'])) for w in words)
    
    score = 206.835 - 1.015 * (len(words)/sentences) - 84.6 * (syllables/len(words))
    score = max(0, min(100, score))
    
    if score >= 90: level = "ง่ายมาก"
    elif score >= 70: level = "ง่าย"
    elif score >= 60: level = "ปานกลาง"
    elif score >= 50: level = "ยากพอสมควร"
    else: level = "ยากมาก"
    
    return score, level

score, level = text_readability("Python is easy to learn. It has clean syntax.")
print(f"\nReadability: {score:.1f} ({level})")

# ข้อ 8-10: เพิ่มเติมในการฝึก
print("\n(ข้อ 8-10: ฝึกเขียนเอง)")
```

---

## สรุป Part 09

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| def | ประกาศ function |
| Parameters/Arguments | Positional, Keyword |
| Default params | ค่าเริ่มต้น (ระวัง mutable) |
| *args/**kwargs | รับ args ไม่จำกัด |
| return | คืนค่า (หลายค่าได้) |
| Docstrings | เอกสาร function |
| LEGB Rule | ลำดับการค้นหา scope |
| global/nonlocal | แก้ไขตัวแปรนอก scope |
| First-class | function ใน variable, arg, return |

### Key Takeaways:
1. Function ช่วยให้โค้ด **DRY** (Don't Repeat Yourself)
2. ใช้ **None** เป็น default แทน mutable object
3. **LEGB rule**: Local → Enclosing → Global → Built-in
4. Python functions เป็น **first-class objects**
5. **Docstring** คือวิธีเขียน documentation ที่ถูกต้อง
6. `*args` = tuple, `**kwargs` = dict

---

*Part 09 จบแล้ว ไปต่อที่ [Part 10 - Functions: Advanced](../part10/README.md)*
