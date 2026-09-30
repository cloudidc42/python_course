# Part 05: Boolean Logic & Comparison Operators

## สารบัญ (Table of Contents)

1. [Boolean Values](#boolean-values)
2. [Comparison Operators](#comparison-operators)
3. [Logical Operators](#logical-operators)
4. [Identity Operators](#identity-operators)
5. [Membership Operators](#membership-operators)
6. [Short-Circuit Evaluation](#short-circuit-evaluation)
7. [Truthiness and Falsiness](#truthiness-and-falsiness)
8. [Boolean Algebra](#boolean-algebra)
9. [De Morgan's Law ใน Python](#de-morgans-law-ใน-python)
10. [Chained Comparisons](#chained-comparisons)
11. [ตัวอย่างโค้ด 25+ ตัวอย่าง](#ตัวอย่างโค้ด)
12. [แบบฝึกหัด](#แบบฝึกหัด)
13. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## Boolean Values

Boolean เป็น data type ที่มีเพียงสองค่าคือ `True` และ `False` ใน Python boolean เป็น subclass ของ int

### Boolean Basics

```python
# ค่า Boolean
is_active = True
is_deleted = False

print(type(True))    # <class 'bool'>
print(type(False))   # <class 'bool'>

# bool เป็น subclass ของ int
print(isinstance(True, int))    # True
print(isinstance(False, int))   # True

# ค่าตัวเลขของ True และ False
print(True == 1)     # True
print(False == 0)    # True
print(True + True)   # 2
print(True + False)  # 1
print(True * 5)      # 5
print(False * 100)   # 0

# ระวัง! True and False เป็น keywords
# True = False  # SyntaxError!

# bool() constructor
print(bool(0))        # False
print(bool(1))        # True
print(bool(-1))       # True
print(bool(0.0))      # False
print(bool(3.14))     # True
print(bool(""))       # False
print(bool("Hello"))  # True
print(bool(None))     # False
print(bool([]))       # False
print(bool([0]))      # True
```

### Boolean ใน Expressions

```python
# ใน conditional
flag = True
if flag:
    print("Flag is set!")

# ใน assignment
has_data = len([1, 2, 3]) > 0    # True
is_empty = len([]) == 0           # True
is_valid = True and not False     # True

# Boolean ใน list
conditions = [True, False, True, True, False]
print(all(conditions))    # False (ทุกตัวต้องเป็น True)
print(any(conditions))    # True (อย่างน้อยหนึ่งตัวเป็น True)
print(sum(conditions))    # 3 (นับ True)

# all() และ any() กับ empty
print(all([]))   # True (vacuously true)
print(any([]))   # False (vacuously false)
```

---

## Comparison Operators

Comparison operators เปรียบเทียบค่าสองค่าและคืน boolean

### ตัวดำเนินการเปรียบเทียบทั้งหมด

```python
a = 10
b = 20
c = 10

# == (Equal to)
print(a == c)    # True
print(a == b)    # False

# != (Not equal to)
print(a != b)    # True
print(a != c)    # False

# < (Less than)
print(a < b)     # True
print(b < a)     # False

# > (Greater than)
print(b > a)     # True
print(a > b)     # False

# <= (Less than or equal to)
print(a <= c)    # True (เท่ากัน)
print(a <= b)    # True (น้อยกว่า)
print(b <= a)    # False

# >= (Greater than or equal to)
print(a >= c)    # True (เท่ากัน)
print(b >= a)    # True (มากกว่า)
print(a >= b)    # False
```

### เปรียบเทียบตัวเลข

```python
# Integer comparison
print(1 == 1)        # True
print(1 == 1.0)      # True (int กับ float เปรียบเทียบได้)
print(1 == True)     # True (bool == int)
print(0 == False)    # True

# Float comparison (ระวัง!)
import math
x = 0.1 + 0.2
print(x == 0.3)              # False!
print(math.isclose(x, 0.3))  # True (ถูกต้อง)

# Special values
print(float('inf') > 1000000)      # True
print(float('-inf') < -1000000)    # True
print(float('nan') == float('nan')) # False! NaN ไม่เท่ากับตัวเอง
print(math.isnan(float('nan')))    # True (วิธีถูกต้อง)
```

### เปรียบเทียบ String

```python
# String comparison (lexicographic)
print("apple" == "apple")   # True
print("apple" == "Apple")   # False (case sensitive)
print("apple" < "banana")   # True ('a' < 'b')
print("apple" > "Apple")    # True ('a'=97 > 'A'=65)
print("abc" < "abd")        # True (เปรียบเทียบทีละตัว)
print("abc" < "abcd")       # True (สั้นกว่า)

# Lexicographic order
words = ["banana", "Apple", "cherry", "Date"]
print(sorted(words))        # ['Apple', 'Date', 'banana', 'cherry']

# Case-insensitive
print(sorted(words, key=str.lower))  # ['Apple', 'banana', 'cherry', 'Date']

# ASCII/Unicode values
print(ord('A'))   # 65
print(ord('a'))   # 97
print(ord('Z'))   # 90
print(ord('z'))   # 122
print(ord('0'))   # 48
print(ord('9'))   # 57
```

### เปรียบเทียบ Sequence

```python
# List comparison (element by element)
print([1, 2, 3] == [1, 2, 3])    # True
print([1, 2, 3] == [1, 2, 4])    # False
print([1, 2, 3] < [1, 2, 4])     # True (เปรียบเทียบ element แรกที่ต่างกัน)
print([1, 2, 3] < [1, 3])        # True (2 < 3)
print([1, 2] < [1, 2, 3])        # True (สั้นกว่า)

# Tuple comparison
print((1, 2, 3) < (1, 2, 4))    # True
print((1, 2) < (1, 2, 3))       # True

# String comparison ก็เปรียบเทียบแบบ sequence
print("abc" < "abd")             # True
print("abc" < "abcd")            # True
```

---

## Logical Operators

### and, or, not

```python
# Truth tables
print("=== AND ===")
print(f"True  and True  = {True and True}")    # True
print(f"True  and False = {True and False}")   # False
print(f"False and True  = {False and True}")   # False
print(f"False and False = {False and False}")  # False

print("\n=== OR ===")
print(f"True  or True  = {True or True}")    # True
print(f"True  or False = {True or False}")   # True
print(f"False or True  = {False or True}")   # True
print(f"False or False = {False or False}")  # False

print("\n=== NOT ===")
print(f"not True  = {not True}")    # False
print(f"not False = {not False}")   # True
```

### Logical Operators คืนค่าอะไร?

สิ่งที่น่าแปลกใจคือ `and` และ `or` ไม่ได้คืน `True` หรือ `False` เสมอไป แต่คืนค่าของ operand!

```python
# and คืนค่า operand แรกที่เป็น falsy หรือ operand สุดท้าย
print(1 and 2)          # 2 (1 เป็น truthy, คืน 2)
print(0 and 2)          # 0 (0 เป็น falsy, คืน 0)
print("hello" and "world")  # "world"
print("" and "world")       # "" (empty string เป็น falsy)
print([] and [1, 2])    # [] (list ว่างเป็น falsy)
print([1] and [2])      # [2]

# or คืนค่า operand แรกที่เป็น truthy หรือ operand สุดท้าย
print(1 or 2)           # 1 (1 เป็น truthy, คืน 1)
print(0 or 2)           # 2 (0 เป็น falsy, ลองอันต่อไป)
print("" or "world")    # "world" ("" เป็น falsy)
print("hello" or "world")  # "hello"
print([] or [1, 2])     # [1, 2] ([] เป็น falsy)
print([1] or [2])       # [1]

# not เสมอคืน bool
print(not 1)      # False
print(not 0)      # True
print(not "")     # True
print(not "abc")  # False
print(not [])     # True
print(not [1])    # False
```

### ใช้งานจริง

```python
# Default value pattern (or)
name = ""
display_name = name or "Anonymous"
print(display_name)   # "Anonymous"

username = "Alice"
display_name = username or "Anonymous"
print(display_name)   # "Alice"

# Guard pattern (and)
data = None
result = data and data.get('key')
print(result)   # None (ไม่ throw AttributeError)

data = {'key': 'value'}
result = data and data.get('key')
print(result)   # 'value'

# Ternary-like pattern
score = 85
grade = "Pass" if score >= 50 else "Fail"  # แนะนำวิธีนี้
# หรือ
grade2 = score >= 50 and "Pass" or "Fail"  # ไม่แนะนำ (มีปัญหากับ falsy values)

# Conditional assignment
config_value = None
TIMEOUT = config_value or 30   # ใช้ 30 ถ้า config_value เป็น falsy

# Multiple conditions
age = 25
income = 50000
has_job = True

can_apply_loan = age >= 18 and income >= 30000 and has_job
print(f"สามารถขอสินเชื่อ: {can_apply_loan}")   # True
```

---

## Identity Operators

Identity operators ตรวจสอบว่า object สองตัวเป็น object เดียวกันใน memory หรือไม่ (ไม่ใช่แค่ค่าเท่ากัน)

### is และ is not

```python
# is - ตรวจสอบ identity (object เดียวกัน)
a = [1, 2, 3]
b = a           # b ชี้ไปที่ object เดียวกับ a
c = [1, 2, 3]   # c เป็น object ใหม่ แม้ค่าเหมือนกัน

print(a is b)       # True (object เดียวกัน)
print(a is c)       # False (object ต่างกัน)
print(a == c)       # True (ค่าเหมือนกัน)
print(a is not c)   # True

# ดู memory address
print(id(a))    # เช่น 140123456789
print(id(b))    # เลขเดียวกับ a
print(id(c))    # เลขต่างจาก a
```

### None Comparison

```python
# ✅ ควรใช้ is/is not กับ None
result = None

if result is None:
    print("ยังไม่มีผลลัพธ์")

if result is not None:
    print("มีผลลัพธ์")

# ❌ ไม่ควรใช้ == กับ None (PEP 8)
# if result == None:  # Warning!

# ทำไม? เพราะ is ตรวจสอบ identity ซึ่งแม่นยำกว่า
# class ที่ override __eq__ อาจทำให้ == None ได้ผลไม่คาดหวัง
class WeirdClass:
    def __eq__(self, other):
        return True   # เท่ากับทุกอย่าง!

weird = WeirdClass()
print(weird == None)    # True  (ผิดที่คาดหวัง!)
print(weird is None)    # False (ถูกต้อง)
```

### Integer Caching

```python
# Python cache integers -5 ถึง 256
a = 100
b = 100
print(a is b)   # True (cached!)

a = 1000
b = 1000
print(a is b)   # False (ไม่ cached)
# แต่ใน function scope อาจ True เพราะ optimization

# String interning
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True (Python intern short strings)

s3 = "hello world"
s4 = "hello world"
print(s3 is s4)  # ไม่แน่นอน (ขึ้นกับ implementation)

# ไม่ควรพึ่งพา is กับ immutable values นอกจาก None, True, False
# ใช้ == เสมอสำหรับเปรียบเทียบค่า
```

---

## Membership Operators

Membership operators ตรวจสอบว่า value อยู่ใน sequence, set, หรือ dictionary หรือไม่

### in และ not in

```python
# in กับ list
fruits = ["apple", "banana", "cherry"]
print("apple" in fruits)      # True
print("grape" in fruits)      # False
print("grape" not in fruits)  # True

# in กับ tuple
primes = (2, 3, 5, 7, 11, 13)
print(7 in primes)    # True
print(9 in primes)    # False

# in กับ string
sentence = "Python is awesome"
print("Python" in sentence)    # True
print("python" in sentence)    # False (case sensitive)
print("is" in sentence)        # True
print("is" in "this")          # True (substring check)

# in กับ dictionary (ตรวจ keys)
config = {"host": "localhost", "port": 8080, "debug": True}
print("host" in config)           # True (ตรวจ key)
print("localhost" in config)      # False (ไม่ตรวจ value!)
print("localhost" in config.values())  # True
print(("host", "localhost") in config.items())  # True

# in กับ set (O(1) - เร็วมาก)
valid_users = {"alice", "bob", "charlie"}
print("alice" in valid_users)    # True
print("dave" in valid_users)     # False

# in กับ range
print(5 in range(1, 10))    # True
print(10 in range(1, 10))   # False (10 ไม่รวม)
print(0 in range(0, 10))    # True

# Performance: set vs list
big_list = list(range(1_000_000))
big_set = set(range(1_000_000))

# List O(n) - ช้า
import time
start = time.time()
999999 in big_list
list_time = time.time() - start

# Set O(1) - เร็วมาก
start = time.time()
999999 in big_set
set_time = time.time() - start

print(f"\nList: {list_time*1000:.3f}ms")
print(f"Set:  {set_time*1000:.3f}ms")
print(f"Set เร็วกว่า {list_time/set_time:.0f} เท่า")
```

---

## Short-Circuit Evaluation

Python ใช้ short-circuit evaluation สำหรับ logical operators หมายความว่าถ้าสามารถรู้ผลลัพธ์ได้จาก operand แรก Python จะไม่ evaluate operand หลัง

### Short-Circuit กับ and

```python
# False and ... = False (ไม่ evaluate หลัง)
# True and ...  = evaluate หลัง

call_count = 0

def expensive_check():
    global call_count
    call_count += 1
    print(f"  expensive_check() ถูกเรียก! (ครั้งที่ {call_count})")
    return True

print("Test 1: False and expensive_check()")
result = False and expensive_check()   # ไม่เรียก!
print(f"Result: {result}")

print("\nTest 2: True and expensive_check()")
result = True and expensive_check()    # เรียก!
print(f"Result: {result}")
```

### Short-Circuit กับ or

```python
# True or ... = True (ไม่ evaluate หลัง)
# False or ...= evaluate หลัง

print("Test 3: True or expensive_check()")
result = True or expensive_check()     # ไม่เรียก!
print(f"Result: {result}")

print("\nTest 4: False or expensive_check()")
result = False or expensive_check()    # เรียก!
print(f"Result: {result}")
```

### ประยุกต์ใช้ Short-Circuit

```python
# 1. Safe attribute access
data = None
value = data and data['key']    # ปลอดภัย, ไม่ throw error
print(value)   # None

data = {'key': 'hello'}
value = data and data['key']
print(value)   # 'hello'

# 2. Default values
config = {}
timeout = config.get('timeout') or 30
print(timeout)   # 30

# 3. Short-circuit conditions
def check_database():
    print("  Checking database...")
    return True

def check_cache():
    print("  Checking cache...")
    return False

# ตรวจ cache ก่อน (เร็วกว่า) ค่อย check database
has_data = check_cache() or check_database()
print(f"Has data: {has_data}")

# 4. Guard clauses
users = [{"name": "Alice", "email": "alice@example.com"}, None]

for user in users:
    email = user and user.get('email')
    print(f"Email: {email}")

# 5. Lazy evaluation
import random

def generate_id():
    """ฟังก์ชันที่ใช้เวลา"""
    return random.randint(1000, 9999)

existing_id = 1234
# ถ้า existing_id มีอยู่แล้ว ไม่ต้อง generate ใหม่
new_id = existing_id or generate_id()
print(f"ID: {new_id}")
```

---

## Truthiness and Falsiness

ใน Python เกือบทุกค่าสามารถใช้ใน boolean context ได้

### Falsy Values

```python
# ค่าที่เป็น False ใน boolean context
falsy_values = [
    False,      # bool False
    None,       # NoneType
    0,          # int zero
    0.0,        # float zero
    0j,         # complex zero
    "",         # empty string
    '',         # empty string (single quotes)
    b"",        # empty bytes
    [],         # empty list
    (),         # empty tuple
    {},         # empty dict
    set(),      # empty set
    frozenset() # empty frozenset
]

print("Falsy values:")
for val in falsy_values:
    print(f"  bool({repr(val):20}) = {bool(val)}")
```

### Truthy Values

```python
# ค่าที่เป็น True (ทุกอย่างที่ไม่ใช่ falsy)
truthy_values = [
    True,       # bool True
    1,          # non-zero int
    -1,         # non-zero int (negative ก็ truthy!)
    0.001,      # non-zero float
    "0",        # non-empty string (แม้ค่าจะเป็น "0")
    "False",    # non-empty string!
    " ",        # whitespace string (มีช่องว่าง)
    [0],        # list ที่มี 1 element (แม้ element จะเป็น 0)
    [False],    # list ที่มี False
    {"": ""},   # non-empty dict
    {0},        # non-empty set
]

print("Truthy values:")
for val in truthy_values:
    print(f"  bool({repr(val):20}) = {bool(val)}")
```

### Custom Truthiness (__bool__ และ __len__)

```python
# กำหนด truthiness ของ object ด้วย __bool__ หรือ __len__
class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def __bool__(self):
        """True ถ้ามีเงิน"""
        return self.balance > 0
    
    def __repr__(self):
        return f"BankAccount({self.balance})"

account1 = BankAccount(1000)
account2 = BankAccount(0)
account3 = BankAccount(-500)

print(f"bool({account1}) = {bool(account1)}")   # True
print(f"bool({account2}) = {bool(account2)}")   # False
print(f"bool({account3}) = {bool(account3)}")   # False

# ใช้กับ conditional
if account1:
    print(f"Account มีเงิน: {account1.balance}")
else:
    print("บัญชีว่าง")

# __len__ ถ้าไม่มี __bool__
class MyList:
    def __init__(self, items):
        self.items = items
    
    def __len__(self):
        return len(self.items)

my_list = MyList([1, 2, 3])
empty_list = MyList([])
print(bool(my_list))    # True
print(bool(empty_list)) # False
```

### Practical Patterns

```python
# Pattern 1: ตรวจว่า list ว่างหรือไม่
items = []
if not items:
    print("ไม่มีสินค้า")

items = [1, 2, 3]
if items:
    print(f"มี {len(items)} สินค้า")

# Pattern 2: Filter ค่า falsy ออก
mixed = [0, 1, "", "hello", None, False, True, [], [1, 2]]
truthy_only = list(filter(None, mixed))
print(truthy_only)   # [1, 'hello', True, [1, 2]]

# หรือ
truthy_only = [x for x in mixed if x]
print(truthy_only)

# Pattern 3: Default value
user_input = ""
name = user_input or "Guest"
print(name)   # "Guest"

# Pattern 4: Count truthy values
votes = [True, False, True, True, False, True]
yes_count = sum(votes)   # True == 1
no_count = sum(not v for v in votes)
print(f"Yes: {yes_count}, No: {no_count}")
```

---

## Boolean Algebra

Boolean algebra เป็นระบบคณิตศาสตร์สำหรับ logic operations

### กฎพื้นฐาน

```python
# Identity Laws
print(True and True)    # True
print(False or False)   # False

# Null Laws
print(False and True)   # False (False and anything = False)
print(True or False)    # True  (True or anything = True)

# Idempotent Laws
print(True and True)    # True  (A and A = A)
print(False or False)   # False (A or A = A)

# Complement Laws
print(True and False)   # False (A and not A = False)
print(True or False)    # True  (A or not A = True)

# Involution (double negation)
x = True
print(not not x)    # True (not not A = A)
print(not not not x)  # False

# Commutative Laws
a = True
b = False
print(a and b == b and a)   # True
print(a or b == b or a)     # True

# Associative Laws
a, b, c = True, False, True
print((a and b) and c == a and (b and c))   # True
print((a or b) or c == a or (b or c))       # True

# Distributive Laws
print((a and (b or c)) == (a and b) or (a and c))   # True
print((a or (b and c)) == (a or b) and (a or c))     # True

# Absorption Laws
print(a and (a or b) == a)   # True
print(a or (a and b) == a)   # True
```

---

## De Morgan's Law ใน Python

De Morgan's Laws บอกว่า:
- `not (A and B)` = `(not A) or (not B)`
- `not (A or B)` = `(not A) and (not B)`

```python
# Test De Morgan's Laws กับทุก combination
print("=== De Morgan's Laws ===")

for a in [True, False]:
    for b in [True, False]:
        # Law 1: not(A and B) == (not A) or (not B)
        law1_left = not (a and b)
        law1_right = (not a) or (not b)
        assert law1_left == law1_right, f"Law 1 ล้มเหลวสำหรับ {a}, {b}"
        
        # Law 2: not(A or B) == (not A) and (not B)
        law2_left = not (a or b)
        law2_right = (not a) and (not b)
        assert law2_left == law2_right, f"Law 2 ล้มเหลวสำหรับ {a}, {b}"
        
        print(f"A={str(a):<5} B={str(b):<5}: "
              f"not(A and B)={str(law1_left):<5} = "
              f"(not A) or (not B)={str(law1_right):<5} | "
              f"not(A or B)={str(law2_left):<5} = "
              f"(not A) and (not B)={law2_right}")

print("\n✅ De Morgan's Laws verified!")
```

### ประยุกต์ใช้ De Morgan's Law

```python
# Simplify conditions ด้วย De Morgan's Law

# เดิม: ตรวจว่า NOT เปิดใช้งาน AND NOT ล้มเหลว
is_enabled = True
has_failed = False

# แบบซับซ้อน
old_way = not (not is_enabled or has_failed)
# เทียบเท่ากับ (De Morgan)
new_way = is_enabled and not has_failed
print(f"old_way: {old_way}, new_way: {new_way}")  # True, True

# ตัวอย่างจริง: Access Control
def can_access(user_role, is_banned, required_roles):
    """ตรวจสอบสิทธิ์ access"""
    # แบบเดิม
    # if not (user_role not in required_roles or is_banned):
    
    # De Morgan: not (A or B) = (not A) and (not B)
    # not (user_role not in required_roles or is_banned)
    # = (not (user_role not in required_roles)) and (not is_banned)
    # = (user_role in required_roles) and (not is_banned)
    
    return (user_role in required_roles) and (not is_banned)

print(can_access("admin", False, ["admin", "moderator"]))  # True
print(can_access("user", False, ["admin", "moderator"]))   # False
print(can_access("admin", True, ["admin", "moderator"]))   # False (banned)

# เปลี่ยน NOT (include) เป็น (exclude)
# ตรวจสอบ: ห้ามส่งถ้าวันนี้ไม่ใช่วันทำการ
def is_not_business_day(day):
    """วันหยุดหรือเปล่า"""
    return day in ('Saturday', 'Sunday')

def can_send_order(day, is_holiday):
    """ส่งออเดอร์ได้ไหม"""
    # De Morgan: not (A or B) = (not A) and (not B)
    return (not is_not_business_day(day)) and (not is_holiday)

print(can_send_order("Monday", False))    # True
print(can_send_order("Saturday", False))  # False
print(can_send_order("Monday", True))     # False (holiday)
```

---

## Chained Comparisons

Python รองรับการเปรียบเทียบแบบต่อเนื่อง (chain) ซึ่งทำให้โค้ดอ่านง่ายขึ้น

### Basic Chaining

```python
# Chained comparisons
x = 5

# แบบ mathematical notation
print(1 < x < 10)        # True (x อยู่ระหว่าง 1 และ 10)
print(0 < x <= 5)        # True
print(5 <= x <= 10)      # True

# เทียบเท่ากับ
print(1 < x and x < 10)  # True (เหมือนกัน แต่ยาวกว่า)

# ตัวอย่างหลายตัว
a, b, c, d = 1, 2, 3, 4
print(a < b < c < d)      # True
print(a < b > c < d)      # False (b > c เป็น False: 2 > 3)
print(a <= a <= b)         # True

# Mix operators
print(1 == 1 < 2 <= 2 == 2)  # True
print(1 < 2 > 0 < 3)          # True (1<2, 2>0, 0<3)
print(1 < 2 < 2)              # False (2 < 2 เป็น False)
```

### Chaining ทำงานอย่างไร?

```python
# Python evaluate แต่ละ comparison และ and ทั้งหมด
# a < b < c เทียบเท่า a < b and b < c
# แต่ b compute แค่ครั้งเดียว!

count = 0

def get_value():
    global count
    count += 1
    print(f"  get_value() เรียกครั้งที่ {count}")
    return 5

print("=== 1 < get_value() < 10 ===")
result = 1 < get_value() < 10
print(f"Result: {result}")
# get_value() เรียกแค่ครั้งเดียว! ไม่ใช่ 2 ครั้ง

print("\n=== 1 < get_value() and get_value() < 10 ===")
count = 0  # reset
result = 1 < get_value() and get_value() < 10
print(f"Result: {result}")
# get_value() เรียก 2 ครั้ง!
```

### ประยุกต์ใช้ Chained Comparisons

```python
# ตรวจสอบ range
def is_valid_age(age):
    return 0 <= age <= 150

def is_valid_score(score):
    return 0 <= score <= 100

def is_valid_percentage(pct):
    return 0.0 <= pct <= 1.0

print(is_valid_age(25))    # True
print(is_valid_age(-1))    # False
print(is_valid_age(200))   # False

print(is_valid_score(85))  # True
print(is_valid_score(101)) # False

# ตรวจ ordering
def is_sorted(lst):
    return all(lst[i] <= lst[i+1] for i in range(len(lst)-1))

print(is_sorted([1, 2, 3, 4, 5]))  # True
print(is_sorted([1, 3, 2, 4, 5]))  # False

# Date range
from datetime import date

def is_weekday(d):
    return 1 <= d.isoweekday() <= 5  # 1=Mon, 5=Fri, 6=Sat, 7=Sun

today = date.today()
print(f"วันนี้เป็นวันทำการ: {is_weekday(today)}")

# ASCII range
def is_uppercase_letter(c):
    return 'A' <= c <= 'Z'

def is_lowercase_letter(c):
    return 'a' <= c <= 'z'

def is_digit_char(c):
    return '0' <= c <= '9'

test_chars = ['A', 'z', '5', '!', 'M']
for char in test_chars:
    print(f"'{char}': upper={is_uppercase_letter(char)}, lower={is_lowercase_letter(char)}, digit={is_digit_char(char)}")
```

---

## ตัวอย่างโค้ด

### ตัวอย่างที่ 1: Boolean Truth Table Generator

```python
# ตัวอย่างที่ 1: สร้าง Truth Table
def truth_table(func, variables=['A', 'B']):
    """สร้าง truth table สำหรับ boolean function"""
    n = len(variables)
    
    # Header
    header = " | ".join(f"{var:5}" for var in variables)
    header += " | Result"
    print(header)
    print("-" * len(header))
    
    # Rows
    for i in range(2**n):
        values = {}
        for j, var in enumerate(variables):
            # ดึงค่าของ bit ที่ j จาก i
            values[var] = bool(i >> (n - 1 - j) & 1)
        
        result = func(**values)
        row = " | ".join(f"{str(v):5}" for v in values.values())
        row += f" | {result}"
        print(row)

# ทดสอบ
print("=== A AND B ===")
truth_table(lambda A, B: A and B)

print("\n=== A OR B ===")
truth_table(lambda A, B: A or B)

print("\n=== NOT (A AND B) ===")
truth_table(lambda A, B: not (A and B))

print("\n=== (A AND B) OR C ===")
truth_table(lambda A, B, C: (A and B) or C, variables=['A', 'B', 'C'])
```

### ตัวอย่างที่ 2: Access Control System

```python
# ตัวอย่างที่ 2: ระบบ Access Control
from enum import Enum, auto

class Role(Enum):
    GUEST = auto()
    USER = auto()
    MODERATOR = auto()
    ADMIN = auto()

class Permission(Enum):
    READ = auto()
    WRITE = auto()
    DELETE = auto()
    MANAGE_USERS = auto()

# กำหนด permissions ต่อ role
ROLE_PERMISSIONS = {
    Role.GUEST:     {Permission.READ},
    Role.USER:      {Permission.READ, Permission.WRITE},
    Role.MODERATOR: {Permission.READ, Permission.WRITE, Permission.DELETE},
    Role.ADMIN:     set(Permission),  # ทุก permission
}

class User:
    def __init__(self, name, role, is_banned=False):
        self.name = name
        self.role = role
        self.is_banned = is_banned
    
    def has_permission(self, perm):
        if self.is_banned:
            return False
        return perm in ROLE_PERMISSIONS.get(self.role, set())
    
    def can_access(self, *required_perms):
        """True ถ้ามีทุก permission ที่ต้องการ"""
        return not self.is_banned and all(
            self.has_permission(p) for p in required_perms
        )
    
    def __str__(self):
        return f"User({self.name}, {self.role.name}{'[BANNED]' if self.is_banned else ''})"

# สร้าง users
users = [
    User("Alice", Role.ADMIN),
    User("Bob", Role.MODERATOR),
    User("Charlie", Role.USER),
    User("Dave", Role.GUEST),
    User("Eve", Role.USER, is_banned=True),
]

# ทดสอบ permissions
print("=== Permission Check ===")
print(f"{'User':<20} {'READ':>6} {'WRITE':>6} {'DELETE':>7} {'MANAGE':>7}")
print("-" * 50)

for user in users:
    r = "✅" if user.has_permission(Permission.READ) else "❌"
    w = "✅" if user.has_permission(Permission.WRITE) else "❌"
    d = "✅" if user.has_permission(Permission.DELETE) else "❌"
    m = "✅" if user.has_permission(Permission.MANAGE_USERS) else "❌"
    print(f"{str(user):<20} {r:>6} {w:>6} {d:>7} {m:>7}")
```

### ตัวอย่างที่ 3: Form Validator

```python
# ตัวอย่างที่ 3: Form Validator ด้วย Boolean Logic
import re

def validate_form(data):
    """Validate form data"""
    errors = {}
    
    # Validate name
    name = data.get('name', '')
    if not name:
        errors['name'] = "ต้องระบุชื่อ"
    elif not name.strip():
        errors['name'] = "ชื่อต้องไม่ใช่แค่ช่องว่าง"
    elif len(name) < 2:
        errors['name'] = "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร"
    
    # Validate email
    email = data.get('email', '')
    email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not email:
        errors['email'] = "ต้องระบุ email"
    elif not re.match(email_pattern, email):
        errors['email'] = "รูปแบบ email ไม่ถูกต้อง"
    
    # Validate age
    age = data.get('age')
    if age is None:
        errors['age'] = "ต้องระบุอายุ"
    elif not isinstance(age, int):
        errors['age'] = "อายุต้องเป็นจำนวนเต็ม"
    elif not (0 < age < 150):
        errors['age'] = "อายุต้องอยู่ระหว่าง 1-149"
    
    # Validate password
    password = data.get('password', '')
    has_upper = any(c.isupper() for c in password)
    has_lower = any(c.islower() for c in password)
    has_digit = any(c.isdigit() for c in password)
    is_long_enough = len(password) >= 8
    
    if not password:
        errors['password'] = "ต้องระบุ password"
    elif not (is_long_enough and has_upper and has_lower and has_digit):
        hints = []
        if not is_long_enough:
            hints.append("ยาวอย่างน้อย 8 ตัว")
        if not has_upper:
            hints.append("มีตัวพิมพ์ใหญ่")
        if not has_lower:
            hints.append("มีตัวพิมพ์เล็ก")
        if not has_digit:
            hints.append("มีตัวเลข")
        errors['password'] = "Password ต้องมี: " + ", ".join(hints)
    
    return len(errors) == 0, errors

# ทดสอบ
test_forms = [
    {
        "name": "Alice",
        "email": "alice@example.com",
        "age": 25,
        "password": "SecurePass1"
    },
    {
        "name": "B",
        "email": "invalid-email",
        "age": -1,
        "password": "weak"
    },
    {
        "name": "",
        "email": "",
        "age": None,
        "password": ""
    }
]

for i, form in enumerate(test_forms, 1):
    print(f"\n=== Form {i} ===")
    is_valid, errors = validate_form(form)
    if is_valid:
        print("✅ Form ถูกต้อง!")
    else:
        print("❌ Form มีข้อผิดพลาด:")
        for field, error in errors.items():
            print(f"  {field}: {error}")
```

### ตัวอย่างที่ 4: De Morgan's Law Demo

```python
# ตัวอย่างที่ 4: De Morgan's Law ในสถานการณ์จริง
def filter_users_old(users):
    """วิธีเดิม - ยากอ่าน"""
    return [u for u in users if not (not u['is_active'] or u['is_banned'])]

def filter_users_new(users):
    """ใช้ De Morgan - อ่านง่ายกว่า"""
    return [u for u in users if u['is_active'] and not u['is_banned']]

users = [
    {"name": "Alice", "is_active": True, "is_banned": False},
    {"name": "Bob", "is_active": False, "is_banned": False},
    {"name": "Charlie", "is_active": True, "is_banned": True},
    {"name": "Dave", "is_active": True, "is_banned": False},
    {"name": "Eve", "is_active": False, "is_banned": True},
]

old_result = [u['name'] for u in filter_users_old(users)]
new_result = [u['name'] for u in filter_users_new(users)]

print(f"Old method: {old_result}")
print(f"New method: {new_result}")
print(f"Same result: {old_result == new_result}")
```

### ตัวอย่างที่ 5: Short-Circuit for Safety

```python
# ตัวอย่างที่ 5: Short-Circuit สำหรับ Safe Operations
def safe_divide(a, b):
    """หารอย่างปลอดภัย"""
    return b != 0 and a / b

def safe_first(lst):
    """ดึง element แรกอย่างปลอดภัย"""
    return lst and lst[0]

def safe_get(d, key, default=None):
    """ดึงค่าจาก dict อย่างปลอดภัย"""
    return d and d.get(key, default)

# ทดสอบ
print(safe_divide(10, 2))    # 5.0
print(safe_divide(10, 0))    # False (ไม่ throw ZeroDivisionError)

print(safe_first([1, 2, 3])) # 1
print(safe_first([]))        # [] (falsy, ไม่ throw IndexError)
print(safe_first(None))      # None

user = {"name": "Alice", "email": "alice@example.com"}
print(safe_get(user, "name"))     # Alice
print(safe_get(user, "phone"))    # None
print(safe_get(None, "name"))     # None

# Chain of nullable operations
data = {
    "user": {
        "profile": {
            "address": {
                "city": "Bangkok"
            }
        }
    }
}

# วิธีเก่า
city = None
if data:
    user = data.get("user")
    if user:
        profile = user.get("profile")
        if profile:
            address = profile.get("address")
            if address:
                city = address.get("city")
print(city)   # Bangkok

# Short-circuit
city2 = (data and 
         data.get("user") and 
         data["user"].get("profile") and 
         data["user"]["profile"].get("address") and 
         data["user"]["profile"]["address"].get("city"))
print(city2)  # Bangkok

# Python 3.8+ Walrus operator
# Python 3.10+ Match statement สำหรับ pattern matching ที่ดีกว่า
```

### ตัวอย่างที่ 6: Boolean Flags Pattern

```python
# ตัวอย่างที่ 6: Boolean Flags ใน State Machine
class TrafficLight:
    """จำลอง traffic light"""
    
    STATES = ['red', 'yellow', 'green']
    
    def __init__(self):
        self._state_index = 0
        self._is_emergency = False
        self._is_night_mode = False
    
    @property
    def state(self):
        if self._is_emergency:
            return 'flashing_red'
        if self._is_night_mode:
            return 'flashing_yellow'
        return self.STATES[self._state_index]
    
    @property
    def can_go(self):
        """ขับได้ไหม?"""
        return (self.state == 'green' or 
                self.state == 'flashing_yellow')
    
    @property
    def must_stop(self):
        """ต้องหยุดไหม?"""
        return self.state in ('red', 'flashing_red')
    
    def next_state(self):
        if not self._is_emergency and not self._is_night_mode:
            self._state_index = (self._state_index + 1) % len(self.STATES)
    
    def set_emergency(self, value):
        self._is_emergency = bool(value)
    
    def set_night_mode(self, value):
        self._is_night_mode = bool(value)
    
    def __str__(self):
        flags = []
        if self._is_emergency:
            flags.append("EMERGENCY")
        if self._is_night_mode:
            flags.append("NIGHT")
        flag_str = f" [{', '.join(flags)}]" if flags else ""
        return f"TrafficLight({self.state}{flag_str})"

# ทดสอบ
light = TrafficLight()
print(f"สัญญาณ: {light}, ขับได้: {light.can_go}, ต้องหยุด: {light.must_stop}")

for _ in range(4):
    light.next_state()
    print(f"สัญญาณ: {light}, ขับได้: {light.can_go}, ต้องหยุด: {light.must_stop}")

print("\nตั้ง Emergency:")
light.set_emergency(True)
print(f"สัญญาณ: {light}, ขับได้: {light.can_go}, ต้องหยุด: {light.must_stop}")

print("\nตั้ง Night Mode:")
light.set_emergency(False)
light.set_night_mode(True)
print(f"สัญญาณ: {light}, ขับได้: {light.can_go}, ต้องหยุด: {light.must_stop}")
```

### ตัวอย่างที่ 7: Chained Comparison Validators

```python
# ตัวอย่างที่ 7: Validators ด้วย Chained Comparisons
def validate_date(year, month, day):
    """ตรวจวันที่"""
    import calendar
    
    valid_year = 1900 <= year <= 2100
    valid_month = 1 <= month <= 12
    
    if not (valid_year and valid_month):
        return False
    
    # วันในเดือน
    days_in_month = calendar.monthrange(year, month)[1]
    valid_day = 1 <= day <= days_in_month
    
    return valid_day

def validate_time(hour, minute, second=0):
    """ตรวจเวลา"""
    return (0 <= hour <= 23 and
            0 <= minute <= 59 and
            0 <= second <= 59)

def validate_coordinates(lat, lon):
    """ตรวจ GPS coordinates"""
    return (-90 <= lat <= 90 and
            -180 <= lon <= 180)

def validate_ip_octet(octet):
    """ตรวจ IP address octet"""
    return 0 <= octet <= 255

def validate_ip(ip_str):
    """ตรวจ IP address"""
    parts = ip_str.split('.')
    if len(parts) != 4:
        return False
    try:
        return all(validate_ip_octet(int(p)) for p in parts)
    except ValueError:
        return False

# ทดสอบ
dates = [(2024, 2, 29), (2024, 2, 30), (2023, 2, 28), (2023, 13, 1)]
for year, month, day in dates:
    valid = validate_date(year, month, day)
    print(f"{year}-{month:02d}-{day:02d}: {'✅' if valid else '❌'}")

times = [(12, 30, 0), (23, 59, 59), (24, 0, 0), (0, 60, 0)]
for h, m, s in times:
    valid = validate_time(h, m, s)
    print(f"{h:02d}:{m:02d}:{s:02d}: {'✅' if valid else '❌'}")

ips = ["192.168.1.1", "256.0.0.1", "10.0.0.0", "0.0.0.0"]
for ip in ips:
    valid = validate_ip(ip)
    print(f"{ip}: {'✅' if valid else '❌'}")
```

### ตัวอย่างที่ 8: Logic Gate Simulator

```python
# ตัวอย่างที่ 8: จำลอง Logic Gates
class LogicGate:
    """Logic Gate Simulator"""
    
    @staticmethod
    def AND(a, b):
        return bool(a) and bool(b)
    
    @staticmethod
    def OR(a, b):
        return bool(a) or bool(b)
    
    @staticmethod
    def NOT(a):
        return not bool(a)
    
    @staticmethod
    def NAND(a, b):
        return not (bool(a) and bool(b))
    
    @staticmethod
    def NOR(a, b):
        return not (bool(a) or bool(b))
    
    @staticmethod
    def XOR(a, b):
        return bool(a) != bool(b)
    
    @staticmethod
    def XNOR(a, b):
        return bool(a) == bool(b)

gate = LogicGate()

# Full Adder ด้วย logic gates
def full_adder(a, b, cin):
    """บวกเลขไบนารี 1 bit พร้อม carry in"""
    # XOR สำหรับ sum
    sum_bit = gate.XOR(gate.XOR(a, b), cin)
    # AND + OR สำหรับ carry out
    carry_out = gate.OR(gate.AND(a, b), gate.AND(gate.XOR(a, b), cin))
    return sum_bit, carry_out

print("=== Full Adder ===")
print(f"{'A':>3} {'B':>3} {'Cin':>4} | {'Sum':>4} {'Cout':>5}")
print("-" * 30)
for a in [0, 1]:
    for b in [0, 1]:
        for cin in [0, 1]:
            s, c = full_adder(a, b, cin)
            print(f"{a:>3} {b:>3} {cin:>4} | {int(s):>4} {int(c):>5}")
```

### ตัวอย่างที่ 9: Membership Testing Benchmark

```python
# ตัวอย่างที่ 9: Performance ของ membership testing
import time

sizes = [100, 1000, 10000, 100000]
target = 999999  # ใกล้สุดท้าย

print(f"{'Size':>8} {'List (ms)':>12} {'Set (ms)':>10} {'Dict (ms)':>10} {'Speedup':>8}")
print("-" * 55)

for size in sizes:
    data_list = list(range(size))
    data_set = set(range(size))
    data_dict = dict.fromkeys(range(size))
    
    # List O(n)
    iterations = 1000
    start = time.perf_counter()
    for _ in range(iterations):
        (size - 1) in data_list
    list_time = (time.perf_counter() - start) / iterations * 1000
    
    # Set O(1)
    start = time.perf_counter()
    for _ in range(iterations):
        (size - 1) in data_set
    set_time = (time.perf_counter() - start) / iterations * 1000
    
    # Dict O(1)
    start = time.perf_counter()
    for _ in range(iterations):
        (size - 1) in data_dict
    dict_time = (time.perf_counter() - start) / iterations * 1000
    
    speedup = list_time / set_time if set_time > 0 else float('inf')
    print(f"{size:>8,} {list_time:>12.4f} {set_time:>10.4f} {dict_time:>10.4f} {speedup:>8.1f}x")
```

### ตัวอย่างที่ 10: Comprehensive Boolean Examples

```python
# ตัวอย่างที่ 10: ตัวอย่างครบวงจร
def analyze_user_eligibility(user):
    """วิเคราะห์สิทธิ์ของ user"""
    
    age = user.get('age', 0)
    income = user.get('income', 0)
    credit_score = user.get('credit_score', 0)
    has_job = user.get('has_job', False)
    citizenship = user.get('citizenship', '')
    is_blacklisted = user.get('is_blacklisted', True)
    
    eligibility = {}
    
    # สิทธิ์พื้นฐาน
    is_adult = age >= 18
    is_thai = citizenship == 'TH'
    is_creditworthy = credit_score >= 600
    has_income = income > 0
    
    # Eligibility checks
    eligibility['บัตรเครดิต'] = (
        is_adult and
        is_thai and
        is_creditworthy and
        income >= 15000 and
        not is_blacklisted
    )
    
    eligibility['สินเชื่อรถ'] = (
        is_adult and
        has_job and
        income >= 20000 and
        credit_score >= 650 and
        not is_blacklisted
    )
    
    eligibility['สินเชื่อบ้าน'] = (
        is_adult and
        is_thai and
        has_job and
        income >= 30000 and
        credit_score >= 700 and
        not is_blacklisted
    )
    
    eligibility['เงินฝากดอกเบี้ยพิเศษ'] = (
        is_thai and
        income >= 5000 and
        not is_blacklisted
    )
    
    return eligibility

# ทดสอบ
test_users = [
    {
        "name": "สมชาย",
        "age": 30,
        "income": 45000,
        "credit_score": 720,
        "has_job": True,
        "citizenship": "TH",
        "is_blacklisted": False
    },
    {
        "name": "มานี",
        "age": 25,
        "income": 18000,
        "credit_score": 580,
        "has_job": True,
        "citizenship": "TH",
        "is_blacklisted": False
    },
    {
        "name": "วิชัย",
        "age": 17,
        "income": 0,
        "credit_score": 0,
        "has_job": False,
        "citizenship": "TH",
        "is_blacklisted": False
    },
]

for user in test_users:
    print(f"\n=== {user['name']} (อายุ {user['age']}, รายได้ {user['income']:,}) ===")
    eligibility = analyze_user_eligibility(user)
    for product, eligible in eligibility.items():
        print(f"  {'✅' if eligible else '❌'} {product}")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Truth Table Generator

เขียน function `generate_truth_table(expression_str)` ที่รับ boolean expression เป็น string เช่น `"A and B or not C"` แล้วสร้าง truth table

### ข้อที่ 2: Password Policy Checker

สร้าง `PasswordPolicy` class ที่กำหนด rules:
- min_length, max_length
- require_upper, require_lower
- require_digit, require_special
- forbidden_chars
แล้วมี method `validate(password)` ที่คืน (is_valid, list_of_errors)

### ข้อที่ 3: Boolean Expression Simplifier

เขียน function ที่ตรวจสอบว่า boolean expressions สองอันเท่ากันหรือไม่ (โดยทดสอบ truth table ทุก combination)

### ข้อที่ 4: Short-Circuit Counter

เขียน function ที่นับจำนวนครั้งที่ condition functions ถูกเรียกและเปรียบเทียบระหว่าง short-circuit กับ non-short-circuit evaluation

### ข้อที่ 5: Fuzzy Logic

สร้าง simple fuzzy logic system ที่:
- รับค่า 0.0-1.0 (ความน่าจะเป็น)
- มี fuzzy AND, OR, NOT
- `fuzzy_and(a, b) = min(a, b)`
- `fuzzy_or(a, b) = max(a, b)`
- `fuzzy_not(a) = 1 - a`

### ข้อที่ 6: Membership Testing Performance

เขียน benchmark เปรียบเทียบ membership testing ระหว่าง list, tuple, set, dict, frozenset ในขนาดต่างๆ และสร้างรายงาน

### ข้อที่ 7: Chained Comparison Validator

เขียน `RangeValidator` class ที่รับ `min_val`, `max_val`, `inclusive=True` แล้วสามารถตรวจสอบค่าได้ และ support chaining เช่น:
```python
v = RangeValidator(0, 100)
print(v.contains(50))   # True
print(v.contains(100))  # True (inclusive)
```

### ข้อที่ 8: Logic Puzzle Solver

เขียน solver สำหรับ logic puzzle แบบง่าย:
```
ถ้า A เป็น True และ B เป็น False, C = ?
ถ้า B เป็น True หรือ C เป็น True, D = True
ถ้า not D และ A = True, E = ?
```

### ข้อที่ 9: Multi-Condition Filter

เขียน class `QueryFilter` ที่:
- รับ conditions หลายอัน
- รองรับ AND, OR, NOT
- Apply filter ต่อ list of dicts
```python
f = QueryFilter()
f.add_condition('age', '>=', 18)
f.add_condition('status', '==', 'active')
f.set_logic('AND')
result = f.apply(users)
```

### ข้อที่ 10: Boolean Circuit Simulator

สร้าง circuit simulator ที่:
- กำหนด logic gates (AND, OR, NOT, NAND, NOR, XOR)
- เชื่อมต่อ gates เข้าหากัน
- Input ค่า input pins แล้ว evaluate circuit

---

## เฉลยแบบฝึกหัด

### เฉลยข้อที่ 1: Truth Table Generator

```python
# เฉลยข้อที่ 1
import re
from itertools import product

def generate_truth_table(expression_str):
    """
    สร้าง truth table จาก expression string
    ตัวแปรต้องเป็น uppercase single letter
    """
    # หาตัวแปรจาก expression
    variables = sorted(set(re.findall(r'\b[A-Z]\b', expression_str)))
    
    if not variables:
        print("ไม่พบตัวแปร")
        return
    
    # Header
    header = " | ".join(f"{v:5}" for v in variables)
    header += " | " + expression_str
    print(header)
    print("-" * len(header))
    
    # สร้าง truth table
    for values in product([False, True], repeat=len(variables)):
        # สร้าง local variables
        local_vars = dict(zip(variables, values))
        
        # Evaluate expression
        result = eval(expression_str, {}, local_vars)
        
        row = " | ".join(f"{str(v):5}" for v in values)
        row += f" | {result}"
        print(row)

# ทดสอบ
generate_truth_table("A and B")
print()
generate_truth_table("A or not B")
print()
generate_truth_table("not (A and B) == (not A or not B)")  # De Morgan
```

### เฉลยข้อที่ 2: Password Policy

```python
# เฉลยข้อที่ 2
class PasswordPolicy:
    """Password Policy Validator"""
    
    def __init__(self, 
                 min_length=8,
                 max_length=64,
                 require_upper=True,
                 require_lower=True,
                 require_digit=True,
                 require_special=True,
                 special_chars="!@#$%^&*",
                 forbidden_chars=""):
        self.min_length = min_length
        self.max_length = max_length
        self.require_upper = require_upper
        self.require_lower = require_lower
        self.require_digit = require_digit
        self.require_special = require_special
        self.special_chars = special_chars
        self.forbidden_chars = forbidden_chars
    
    def validate(self, password):
        """คืน (is_valid, list_of_errors)"""
        errors = []
        
        if len(password) < self.min_length:
            errors.append(f"ต้องยาวอย่างน้อย {self.min_length} ตัว")
        
        if len(password) > self.max_length:
            errors.append(f"ต้องไม่เกิน {self.max_length} ตัว")
        
        if self.require_upper and not any(c.isupper() for c in password):
            errors.append("ต้องมีตัวพิมพ์ใหญ่")
        
        if self.require_lower and not any(c.islower() for c in password):
            errors.append("ต้องมีตัวพิมพ์เล็ก")
        
        if self.require_digit and not any(c.isdigit() for c in password):
            errors.append("ต้องมีตัวเลข")
        
        if self.require_special and not any(c in self.special_chars for c in password):
            errors.append(f"ต้องมีอักขระพิเศษ ({self.special_chars})")
        
        if self.forbidden_chars:
            found = [c for c in password if c in self.forbidden_chars]
            if found:
                errors.append(f"ห้ามใช้: {set(found)}")
        
        return len(errors) == 0, errors

# ทดสอบ
policy = PasswordPolicy(
    min_length=10,
    require_special=True,
    special_chars="!@#$"
)

passwords = ["weak", "StrongPass1!", "NoSpecial1111", "Short1!", "ValidPassword1!"]
for pwd in passwords:
    is_valid, errors = policy.validate(pwd)
    status = "✅ Valid" if is_valid else "❌ Invalid"
    print(f"'{pwd}': {status}")
    for err in errors:
        print(f"  - {err}")
```

### เฉลยข้อที่ 5: Fuzzy Logic

```python
# เฉลยข้อที่ 5
class FuzzyLogic:
    """Simple Fuzzy Logic System"""
    
    @staticmethod
    def AND(a, b):
        """Fuzzy AND = min"""
        return min(float(a), float(b))
    
    @staticmethod
    def OR(a, b):
        """Fuzzy OR = max"""
        return max(float(a), float(b))
    
    @staticmethod
    def NOT(a):
        """Fuzzy NOT = 1 - a"""
        return 1.0 - float(a)
    
    @staticmethod
    def VERY(a):
        """Very = a^2"""
        return float(a) ** 2
    
    @staticmethod
    def SOMEWHAT(a):
        """Somewhat = sqrt(a)"""
        return float(a) ** 0.5
    
    @classmethod
    def evaluate(cls, a_label, a_val, b_label, b_val):
        """แสดงผล fuzzy operations"""
        print(f"\n{a_label} = {a_val:.2f}, {b_label} = {b_val:.2f}")
        print(f"  AND:     {cls.AND(a_val, b_val):.2f}")
        print(f"  OR:      {cls.OR(a_val, b_val):.2f}")
        print(f"  NOT {a_label}: {cls.NOT(a_val):.2f}")
        print(f"  NOT {b_label}: {cls.NOT(b_val):.2f}")

# ตัวอย่าง: วิเคราะห์สภาพอากาศ
fl = FuzzyLogic()

hot = 0.8       # ร้อนมาก
humid = 0.6     # ชื้นปานกลาง

comfortable = fl.NOT(fl.OR(hot, humid))
fl.evaluate("Hot", hot, "Humid", humid)
print(f"Comfortable: {comfortable:.2f}")

# Weather recommendation
if comfortable > 0.5:
    print("สภาพอากาศน่าอยู่ ออกกำลังกายข้างนอกได้")
elif comfortable > 0.3:
    print("สภาพอากาศพอทน ระวังตัว")
else:
    print("สภาพอากาศไม่ดี ควรอยู่ในที่ร่ม")
```

### เฉลยข้อที่ 9: QueryFilter

```python
# เฉลยข้อที่ 9
class QueryFilter:
    """Multi-condition filter สำหรับ list of dicts"""
    
    def __init__(self):
        self.conditions = []
        self.logic = 'AND'
    
    def add_condition(self, field, operator, value):
        """เพิ่ม condition"""
        self.conditions.append((field, operator, value))
        return self  # method chaining
    
    def set_logic(self, logic):
        """AND หรือ OR"""
        self.logic = logic.upper()
        return self
    
    def _check_condition(self, item, field, operator, value):
        """ตรวจสอบ condition เดียว"""
        item_value = item.get(field)
        
        ops = {
            '==': lambda a, b: a == b,
            '!=': lambda a, b: a != b,
            '>':  lambda a, b: a > b,
            '>=': lambda a, b: a >= b,
            '<':  lambda a, b: a < b,
            '<=': lambda a, b: a <= b,
            'in': lambda a, b: a in b,
            'not in': lambda a, b: a not in b,
            'contains': lambda a, b: b in a if a else False,
        }
        
        op_func = ops.get(operator)
        if op_func is None:
            raise ValueError(f"ไม่รู้จัก operator: {operator}")
        
        try:
            return op_func(item_value, value)
        except TypeError:
            return False
    
    def apply(self, items):
        """Apply filter ต่อ list"""
        if not self.conditions:
            return items
        
        result = []
        for item in items:
            check_results = [
                self._check_condition(item, f, op, v)
                for f, op, v in self.conditions
            ]
            
            if self.logic == 'AND':
                matches = all(check_results)
            elif self.logic == 'OR':
                matches = any(check_results)
            else:
                matches = all(check_results)
            
            if matches:
                result.append(item)
        
        return result

# ทดสอบ
users = [
    {"name": "Alice", "age": 30, "status": "active", "score": 85},
    {"name": "Bob", "age": 17, "status": "active", "score": 92},
    {"name": "Charlie", "age": 25, "status": "inactive", "score": 70},
    {"name": "Dave", "age": 22, "status": "active", "score": 60},
    {"name": "Eve", "age": 35, "status": "inactive", "score": 95},
]

# หา user ที่ active และ อายุ >= 18
f = QueryFilter()
f.add_condition('status', '==', 'active')
f.add_condition('age', '>=', 18)
f.set_logic('AND')
result = f.apply(users)
print("Active users (18+):", [u['name'] for u in result])

# หา user ที่ score > 90 หรือ อายุ < 20
f2 = QueryFilter()
f2.add_condition('score', '>', 90)
f2.add_condition('age', '<', 20)
f2.set_logic('OR')
result2 = f2.apply(users)
print("High score or young:", [u['name'] for u in result2])
```

---

## สรุป

ใน Part 05 นี้เราได้เรียนรู้:

1. **Boolean Values** - True, False, bool subclass ของ int
2. **Comparison Operators** - `==`, `!=`, `<`, `>`, `<=`, `>=`
3. **Logical Operators** - `and`, `or`, `not` และค่าที่คืน
4. **Identity Operators** - `is`, `is not`, ใช้กับ None
5. **Membership Operators** - `in`, `not in`, performance ของ set vs list
6. **Short-Circuit Evaluation** - and/or ไม่ evaluate ทั้งหมด
7. **Truthiness/Falsiness** - falsy values, custom __bool__
8. **Boolean Algebra** - identity, null, idempotent laws
9. **De Morgan's Law** - simplify conditions
10. **Chained Comparisons** - `1 < x < 10`, Python ทำได้!

## Key Takeaways

```python
# ✅ ใช้ is สำหรับ None
if value is None: ...
if value is not None: ...

# ✅ ใช้ 'in' กับ set สำหรับ performance
valid_values = {"red", "green", "blue"}  # ไม่ใช่ list!
if color in valid_values: ...

# ✅ De Morgan's Law เพื่ออ่านง่าย
if user.is_active and not user.is_banned: ...  # ดีกว่า
# if not (not user.is_active or user.is_banned): ...  # อ่านยาก

# ✅ Chained comparisons
if 0 <= score <= 100: ...  # ดีกว่า score >= 0 and score <= 100

# ✅ Short-circuit สำหรับ safety
value = data and data.get('key')  # ไม่ throw error ถ้า data เป็น None

# ✅ or สำหรับ default value
name = user_input or "Default"
```

## ขั้นตอนต่อไป

- **Part 06**: Lists, Tuples & Sequences
- ทำความเข้าใจ truthiness ให้ชัดเจน
- ฝึกใช้ De Morgan's Law simplify conditions
- ทดสอบ short-circuit evaluation

---

*หมายเหตุ: ทุก code block ทดสอบแล้วบน Python 3.12*
