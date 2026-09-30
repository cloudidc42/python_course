# Part 10 - Functions: Advanced

## สารบัญ

1. [Keyword-only Arguments](#1-keyword-only-arguments)
2. [Positional-only Arguments (Python 3.8+)](#2-positional-only-arguments-python-38)
3. [Function Annotations (Type Hints)](#3-function-annotations-type-hints)
4. [Recursive Functions](#4-recursive-functions)
5. [Closures](#5-closures)
6. [Higher-order Functions](#6-higher-order-functions)
7. [Partial Functions (functools.partial)](#7-partial-functions-functoolspartial)
8. [Function Caching (functools.lru_cache)](#8-function-caching-functoolslru_cache)
9. [Memoization](#9-memoization)
10. [Pure Functions vs Side Effects](#10-pure-functions-vs-side-effects)
11. [ตัวอย่างโปรแกรมจริง](#11-ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. Keyword-only Arguments

### ความหมาย

Keyword-only arguments คือ arguments ที่ต้องส่งด้วย keyword เท่านั้น ไม่สามารถส่งแบบ positional ได้ กำหนดโดยใส่ `*` ไว้ก่อนพวกมันใน signature

### Syntax

```python
def function(pos_arg, *, keyword_only_arg):
    pass
```

### ตัวอย่างที่ 1: Keyword-only Arguments พื้นฐาน

```python
# ตัวอย่างที่ 1
def create_connection(host, port, *, timeout=30, ssl=True, retry=3):
    """สร้าง connection
    
    host, port: positional arguments
    timeout, ssl, retry: keyword-only arguments (หลัง *)
    """
    print(f"เชื่อมต่อ: {host}:{port}")
    print(f"  timeout={timeout}s, ssl={ssl}, retry={retry}")

# เรียกแบบถูกต้อง
create_connection("example.com", 8080)
create_connection("db.server.com", 5432, timeout=60, ssl=False)
create_connection("api.service.com", 443, retry=5)

# ❌ Error: keyword-only args ต้องส่งด้วย keyword
try:
    create_connection("host.com", 80, 60)  # 60 ไปที่ timeout ไม่ได้!
except TypeError as e:
    print(f"\n✗ Error: {e}")
```

**Output:**
```
เชื่อมต่อ: example.com:8080
  timeout=30s, ssl=True, retry=3
เชื่อมต่อ: db.server.com:5432
  timeout=60s, ssl=False, retry=3
เชื่อมต่อ: api.service.com:443
  timeout=30s, ssl=True, retry=5

✗ Error: create_connection() takes 2 positional arguments but 3 were given
```

### ตัวอย่างที่ 2: Keyword-only กับ *args

```python
# ตัวอย่างที่ 2: รวม *args กับ keyword-only
def print_items(*items, separator=", ", end_char="\n", uppercase=False):
    """แสดงรายการสิ่งของ
    
    items: positional args (จำนวนใดก็ได้)
    separator, end_char, uppercase: keyword-only
    """
    output = [item.upper() if uppercase else item for item in items]
    print(separator.join(str(i) for i in output), end=end_char)

# ทดสอบ
print_items("apple", "banana", "cherry")
print_items("a", "b", "c", separator=" | ")
print_items("hello", "world", separator="-", uppercase=True)
print_items(1, 2, 3, 4, 5, separator=" + ", end_char=" = ?\n")
```

### ตัวอย่างที่ 3: Keyword-only ใน API Design

```python
# ตัวอย่างที่ 3: API design ที่ดี
def send_email(to, subject, body, *, cc=None, bcc=None, html=False, priority="normal"):
    """ส่งอีเมล
    
    to, subject, body: ข้อมูลหลัก (positional)
    cc, bcc, html, priority: ตัวเลือกเพิ่มเติม (keyword-only เพื่อความชัดเจน)
    """
    print(f"\nส่งอีเมล:")
    print(f"  To: {to}")
    print(f"  Subject: {subject}")
    print(f"  Body: {body[:50]}...")
    if cc: print(f"  CC: {cc}")
    if bcc: print(f"  BCC: {bcc}")
    print(f"  HTML: {html}, Priority: {priority}")
    return True

# เรียกแบบชัดเจน
send_email(
    "user@example.com",
    "สวัสดี",
    "นี่คือข้อความทดสอบ",
    cc="manager@example.com",
    priority="high"
)

send_email(
    "team@example.com",
    "ประกาศ",
    "<h1>ข่าวสำคัญ</h1>",
    html=True,
    bcc="archive@example.com"
)
```

---

## 2. Positional-only Arguments (Python 3.8+)

### ความหมาย

Positional-only arguments คือ arguments ที่ต้องส่งแบบ positional เท่านั้น ไม่สามารถใช้ keyword ได้ กำหนดโดยใส่ `/` ไว้หลังพวกมัน

### Syntax

```python
def function(pos_only_arg, /, normal_arg, *, keyword_only_arg):
    pass
```

### ตัวอย่างที่ 4: Positional-only Arguments

```python
# ตัวอย่างที่ 4
def pow_custom(base, exponent, /):
    """คำนวณ base^exponent
    
    base, exponent: positional-only (ก่อน /)
    """
    return base ** exponent

# ✅ ถูกต้อง
print(pow_custom(2, 10))    # 1024
print(pow_custom(3, 3))     # 27

# ❌ Error: ไม่สามารถใช้ keyword ได้
try:
    print(pow_custom(base=2, exponent=10))
except TypeError as e:
    print(f"✗ Error: {e}")

# ตัวอย่างที่ 5: รวมทั้ง 3 ประเภท
def comprehensive(pos_only, /, normal, *, kw_only):
    """Function ที่มีทุกประเภทของ parameters"""
    print(f"pos_only={pos_only}, normal={normal}, kw_only={kw_only}")

# เรียกที่ถูกต้อง
comprehensive(1, 2, kw_only=3)          # ✅
comprehensive(1, normal=2, kw_only=3)   # ✅

# ❌ Error cases
try:
    comprehensive(pos_only=1, normal=2, kw_only=3)  # pos_only ใช้ keyword ไม่ได้
except TypeError as e:
    print(f"✗ {e}")

try:
    comprehensive(1, 2, 3)  # kw_only ต้องส่งด้วย keyword
except TypeError as e:
    print(f"✗ {e}")
```

### ตัวอย่างที่ 6: ประโยชน์ของ Positional-only

```python
# ตัวอย่างที่ 6: ป้องกันการพึ่งพา parameter name
def parse_coordinates(x, y, /):
    """แปลง x, y เป็น tuple
    
    ใช้ positional-only เพราะ x, y เป็นชื่อทั่วไป
    ถ้าเปลี่ยนชื่อ parameter ในอนาคต จะไม่กระทบ caller
    """
    return (float(x), float(y))

# ถ้าไม่มี / caller อาจเรียกว่า parse_coordinates(x=1, y=2)
# ซึ่งทำให้เปลี่ยนชื่อ parameter ไม่ได้โดยไม่ทำให้ code เก่าพัง
print(parse_coordinates(3, 4))
print(parse_coordinates(10.5, -2.3))

# ตัวอย่าง built-in ที่ใช้ positional-only
# int(x=10, base=2) ← ❌ ใช้ไม่ได้
# int(10, 2) ← ✅ ต้องแบบนี้
print(int("1010", 2))  # binary 1010 = 10
```

---

## 3. Function Annotations (Type Hints)

### ความหมาย

Type hints คือการระบุประเภทของ parameters และ return value ช่วยให้:
- โค้ดอ่านง่ายขึ้น
- IDE แสดง autocomplete และ error ได้ดีขึ้น
- สามารถใช้ tools เช่น mypy ตรวจสอบ type

### ตัวอย่างที่ 7: Type Hints พื้นฐาน

```python
# ตัวอย่างที่ 7: Type hints พื้นฐาน
def add(a: int, b: int) -> int:
    """บวกสองจำนวนเต็ม"""
    return a + b

def greet(name: str, times: int = 1) -> str:
    """สร้างข้อความทักทาย"""
    return f"สวัสดี {name}! " * times

def is_adult(age: int) -> bool:
    """ตรวจสอบว่าบรรลุนิติภาวะหรือไม่"""
    return age >= 18

# Type hints ไม่ enforce ที่ runtime (แค่ hints)
print(add(3, 5))         # 8
print(greet("Alice", 2))  # สวัสดี Alice! สวัสดี Alice! 
print(is_adult(20))      # True

# ดู annotations
print(f"\nAnnotations ของ add: {add.__annotations__}")
```

### ตัวอย่างที่ 8: Type Hints ขั้นสูง

```python
from typing import List, Dict, Tuple, Optional, Union, Any, Callable

# ตัวอย่างที่ 8: Type hints ขั้นสูง
def process_scores(
    scores: List[float],
    weights: Optional[List[float]] = None
) -> Dict[str, float]:
    """คำนวณสถิติคะแนน"""
    if not scores:
        return {}
    
    if weights is None:
        weights = [1.0] * len(scores)
    
    total_weight = sum(weights)
    weighted_avg = sum(s * w for s, w in zip(scores, weights)) / total_weight
    
    return {
        "count": len(scores),
        "mean": sum(scores) / len(scores),
        "weighted_mean": weighted_avg,
        "min": min(scores),
        "max": max(scores),
    }

result = process_scores([85.0, 92.0, 78.0, 95.0])
for key, value in result.items():
    print(f"  {key}: {value:.2f}")

# Union type
def parse_number(value: Union[str, int, float]) -> float:
    """แปลงค่าต่างๆ เป็น float"""
    return float(value)

print(f"\n{parse_number('3.14')}")
print(f"{parse_number(42)}")
print(f"{parse_number(2.718)}")
```

### ตัวอย่างที่ 9: Type Hints สมัยใหม่ (Python 3.10+)

```python
# Python 3.10+ ใช้ | แทน Union
# Python 3.9+ ใช้ list, dict, tuple แทน List, Dict, Tuple

def calculate(
    values: list[int | float],
    operation: str = "sum"
) -> int | float | None:
    """คำนวณตาม operation ที่กำหนด"""
    if not values:
        return None
    
    match operation:
        case "sum":
            return sum(values)
        case "product":
            result = 1
            for v in values:
                result *= v
            return result
        case "mean":
            return sum(values) / len(values)
        case _:
            return None

print(calculate([1, 2, 3, 4, 5]))              # sum
print(calculate([1, 2, 3, 4, 5], "product"))   # product
print(calculate([10, 20, 30], "mean"))         # mean

# Callable type hint
from typing import Callable

def apply_func(
    values: list[int],
    func: Callable[[int], int]
) -> list[int]:
    """ใช้ func กับทุก element"""
    return [func(v) for v in values]

print(apply_func([1, 2, 3, 4], lambda x: x**2))
print(apply_func([1, 2, 3, 4], lambda x: x*2))
```

---

## 4. Recursive Functions

### ความหมาย

Recursive function คือ function ที่เรียกตัวเอง ทุก recursive function ต้องมี:
1. **Base case**: เงื่อนไขหยุด
2. **Recursive case**: เรียกตัวเองด้วยข้อมูลที่เล็กลง

### ตัวอย่างที่ 10: Factorial

```python
# ตัวอย่างที่ 10: Factorial
def factorial(n: int) -> int:
    """คำนวณ n! (n factorial)
    
    Base case: n == 0 หรือ 1 -> คืน 1
    Recursive: n! = n * (n-1)!
    """
    if n < 0:
        raise ValueError("n ต้องไม่เป็นลบ")
    if n <= 1:  # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case

# ทดสอบ
for n in range(11):
    print(f"{n}! = {factorial(n):,}")

# แสดงขั้นตอน
def factorial_verbose(n: int, depth: int = 0) -> int:
    indent = "  " * depth
    print(f"{indent}factorial({n})")
    
    if n <= 1:
        print(f"{indent}= 1 (base case)")
        return 1
    
    sub_result = factorial_verbose(n - 1, depth + 1)
    result = n * sub_result
    print(f"{indent}= {n} × {sub_result} = {result}")
    return result

print("\nขั้นตอน factorial(5):")
result = factorial_verbose(5)
```

### ตัวอย่างที่ 11: Fibonacci Recursive

```python
# ตัวอย่างที่ 11: Fibonacci
def fib_recursive(n: int) -> int:
    """Fibonacci แบบ recursive (ช้า เพราะคำนวณซ้ำ)"""
    if n <= 1:
        return n
    return fib_recursive(n - 1) + fib_recursive(n - 2)

def fib_memo(n: int, memo: dict = None) -> int:
    """Fibonacci แบบ memoization (เร็ว)"""
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]

import time

# เปรียบเทียบความเร็ว
print("เปรียบเทียบ Fibonacci:")
print(f"{'n':>4} {'recursive':>12} {'memoized':>10}")
print("-" * 30)

for n in [10, 20, 30]:
    start = time.time()
    r1 = fib_recursive(n)
    t1 = time.time() - start
    
    start = time.time()
    r2 = fib_memo(n)
    t2 = time.time() - start
    
    print(f"{n:>4} {t1*1000:>10.3f}ms {t2*1000:>8.3f}ms")
    assert r1 == r2, "ผลลัพธ์ไม่ตรงกัน!"

print("ผลลัพธ์ตรงกันทั้งหมด ✓")
```

### ตัวอย่างที่ 12: Tree Traversal

```python
# ตัวอย่างที่ 12: Tree Traversal
def build_tree():
    return {
        "value": 1,
        "children": [
            {
                "value": 2,
                "children": [
                    {"value": 4, "children": []},
                    {"value": 5, "children": []},
                ]
            },
            {
                "value": 3,
                "children": [
                    {"value": 6, "children": []},
                    {
                        "value": 7,
                        "children": [
                            {"value": 8, "children": []},
                        ]
                    },
                ]
            },
        ]
    }

def tree_sum(node: dict) -> int:
    """บวกค่าทุก node ใน tree"""
    if not node:
        return 0
    total = node["value"]
    for child in node["children"]:
        total += tree_sum(child)
    return total

def tree_depth(node: dict) -> int:
    """หาความลึกของ tree"""
    if not node["children"]:
        return 0
    return 1 + max(tree_depth(child) for child in node["children"])

def tree_print(node: dict, prefix: str = "", is_last: bool = True) -> None:
    """แสดง tree แบบ pretty print"""
    connector = "└── " if is_last else "├── "
    print(prefix + connector + str(node["value"]))
    
    new_prefix = prefix + ("    " if is_last else "│   ")
    children = node["children"]
    for i, child in enumerate(children):
        tree_print(child, new_prefix, i == len(children) - 1)

tree = build_tree()
print("Tree Structure:")
print(tree["value"])
children = tree["children"]
for i, child in enumerate(children):
    tree_print(child, "", i == len(children) - 1)

print(f"\nผลรวมทุก node: {tree_sum(tree)}")
print(f"ความลึกสูงสุด: {tree_depth(tree)}")
```

### ตัวอย่างที่ 13: Quick Sort

```python
# ตัวอย่างที่ 13: Quick Sort
def quick_sort(arr: list) -> list:
    """Quick Sort แบบ recursive"""
    if len(arr) <= 1:  # Base case
        return arr
    
    pivot = arr[len(arr) // 2]  # เลือก pivot
    left = [x for x in arr if x < pivot]    # น้อยกว่า pivot
    middle = [x for x in arr if x == pivot]  # เท่ากับ pivot
    right = [x for x in arr if x > pivot]   # มากกว่า pivot
    
    return quick_sort(left) + middle + quick_sort(right)

def merge_sort(arr: list) -> list:
    """Merge Sort แบบ recursive"""
    if len(arr) <= 1:  # Base case
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])   # เรียง left half
    right = merge_sort(arr[mid:])  # เรียง right half
    
    return merge(left, right)

def merge(left: list, right: list) -> list:
    """รวม 2 sorted lists"""
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# ทดสอบ
import random
random.seed(42)

data = [random.randint(1, 100) for _ in range(15)]
print(f"ก่อนเรียง: {data}")
print(f"Quick Sort: {quick_sort(data)}")
print(f"Merge Sort: {merge_sort(data)}")
print(f"Built-in:  {sorted(data)}")
print(f"✓ ผลตรงกัน: {quick_sort(data) == merge_sort(data) == sorted(data)}")
```

---

## 5. Closures

### ความหมาย

Closure คือ function ที่ "จำ" environment ที่มันถูกสร้างขึ้น รวมถึงตัวแปรจาก enclosing scope

### ตัวอย่างที่ 14: Closure พื้นฐาน

```python
# ตัวอย่างที่ 14: Closure พื้นฐาน
def make_adder(n: int):
    """สร้าง function ที่บวก n ให้กับ input"""
    def adder(x: int) -> int:
        return x + n  # n ถูก "capture" ใน closure
    return adder

add5 = make_adder(5)
add10 = make_adder(10)
add100 = make_adder(100)

print(f"add5(3) = {add5(3)}")      # 8
print(f"add10(3) = {add10(3)}")    # 13
print(f"add100(3) = {add100(3)}")  # 103

# Closure "จำ" ค่าของ n ของแต่ละ instance
print(f"\nadd5.__closure__[0].cell_contents = {add5.__closure__[0].cell_contents}")
print(f"add10.__closure__[0].cell_contents = {add10.__closure__[0].cell_contents}")
```

### ตัวอย่างที่ 15: Closure ใช้งานจริง

```python
# ตัวอย่างที่ 15: Counter using Closure
def make_counter(start: int = 0, step: int = 1):
    """สร้าง counter ด้วย closure"""
    count = [start]  # ใช้ list เพื่อ mutability
    
    def increment():
        count[0] += step
        return count[0]
    
    def decrement():
        count[0] -= step
        return count[0]
    
    def reset():
        count[0] = start
    
    def current():
        return count[0]
    
    return increment, decrement, reset, current

# สร้าง counter instances ต่างๆ
inc1, dec1, rst1, cur1 = make_counter(0, 1)
inc2, dec2, rst2, cur2 = make_counter(100, 10)

print("Counter 1 (start=0, step=1):")
print(f"  Inc: {inc1()}, {inc1()}, {inc1()}")
print(f"  Dec: {dec1()}")
print(f"  Current: {cur1()}")

print("\nCounter 2 (start=100, step=10):")
print(f"  Inc: {inc2()}, {inc2()}, {inc2()}")
print(f"  Dec: {dec2()}")
print(f"  Current: {cur2()}")

print(f"\nCounter 1 ยังคงเป็น: {cur1()}")  # ไม่ถูกกระทบจาก counter 2
```

### ตัวอย่างที่ 16: Closure สำหรับ Decorator

```python
# ตัวอย่างที่ 16: Decorator ด้วย Closure
import functools
import time

def retry(max_attempts: int = 3, delay: float = 0.1):
    """Decorator สำหรับ retry เมื่อ exception"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_error = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_error = e
                    print(f"  ครั้งที่ {attempt} ล้มเหลว: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise last_error
        return wrapper
    return decorator

def log_calls(func):
    """Decorator บันทึกการเรียก function"""
    call_count = [0]  # Closure variable
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        call_count[0] += 1
        print(f"[Log] {func.__name__} ถูกเรียก (ครั้งที่ {call_count[0]})")
        result = func(*args, **kwargs)
        print(f"[Log] {func.__name__} คืนค่า: {result}")
        return result
    
    wrapper.call_count = lambda: call_count[0]  # เพิ่ม method
    return wrapper

@log_calls
def add(a, b):
    return a + b

print("Test log_calls:")
add(1, 2)
add(3, 4)
add(5, 6)
print(f"เรียกทั้งหมด: {add.call_count()} ครั้ง")

# Test retry
import random
random.seed(100)

@retry(max_attempts=4, delay=0)
def unreliable_function():
    """Function ที่ล้มเหลวแบบสุ่ม"""
    if random.random() < 0.6:  # 60% โอกาสล้มเหลว
        raise RuntimeError("Connection failed!")
    return "Success!"

print("\nTest retry:")
try:
    result = unreliable_function()
    print(f"ผลลัพธ์: {result}")
except RuntimeError:
    print("ล้มเหลวทุกครั้ง")
```

---

## 6. Higher-order Functions

### ความหมาย

Higher-order function คือ function ที่รับ function เป็น argument หรือคืน function เป็น return value

### Built-in Higher-order Functions

```python
# ตัวอย่างที่ 17: map, filter, reduce

from functools import reduce

numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# map: ใช้ function กับทุก element
squares = list(map(lambda x: x**2, numbers))
print(f"map(square): {squares}")

# filter: กรอง element ตามเงื่อนไข
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(f"filter(even): {evens}")

# reduce: รวม elements เป็นค่าเดียว
total = reduce(lambda acc, x: acc + x, numbers)
print(f"reduce(sum): {total}")

product = reduce(lambda acc, x: acc * x, numbers[:5])
print(f"reduce(product of 1-5): {product}")

# รวมกัน
result = reduce(
    lambda acc, x: acc + x,
    filter(
        lambda x: x % 2 == 0,
        map(lambda x: x**2, numbers)
    )
)
print(f"\nผลรวมของ square ของเลขคู่: {result}")
```

### ตัวอย่างที่ 18: Custom Higher-order Functions

```python
# ตัวอย่างที่ 18: Custom HOF
from typing import Callable, TypeVar, List, Any

T = TypeVar('T')

def compose(*funcs: Callable) -> Callable:
    """รวม functions เข้าด้วยกัน (right to left)"""
    def composed(x):
        for func in reversed(funcs):
            x = func(x)
        return x
    return composed

def pipe(*funcs: Callable) -> Callable:
    """รวม functions เข้าด้วยกัน (left to right)"""
    def piped(x):
        for func in funcs:
            x = func(x)
        return x
    return piped

def apply_all(value, *funcs: Callable) -> List[Any]:
    """ใช้ทุก function กับ value เดียวกัน"""
    return [func(value) for func in funcs]

# ทดสอบ compose
double = lambda x: x * 2
add_one = lambda x: x + 1
square = lambda x: x ** 2

# compose: square(add_one(double(x)))
transform = compose(square, add_one, double)
print(f"compose(square, add_one, double)(3) = {transform(3)}")
# double(3)=6, add_one(6)=7, square(7)=49

# pipe: double(add_one(square(x)))
transform2 = pipe(double, add_one, square)
print(f"pipe(double, add_one, square)(3) = {transform2(3)}")
# double(3)=6, add_one(6)=7, square(7)=49 (same with different order)

# apply_all
results = apply_all(5, double, add_one, square, lambda x: x-1)
print(f"\napply_all(5, ...): {results}")
```

### ตัวอย่างที่ 19: Currying

```python
# ตัวอย่างที่ 19: Currying (แบ่ง function หลาย args เป็น chain ของ function 1 arg)

def curry(func: Callable) -> Callable:
    """Currying decorator"""
    import inspect
    n_params = len(inspect.signature(func).parameters)
    
    def curried(*args):
        if len(args) >= n_params:
            return func(*args[:n_params])
        return lambda *more_args: curried(*args, *more_args)
    
    return curried

@curry
def add(a, b, c):
    return a + b + c

@curry
def multiply(a, b):
    return a * b

# ใช้แบบปกติ
print(f"add(1, 2, 3) = {add(1, 2, 3)}")

# ใช้แบบ curried
add1 = add(1)          # partially applied
add1_2 = add1(2)       # partially applied ต่อ
result = add1_2(3)     # complete!
print(f"add(1)(2)(3) = {result}")

# ใช้กับ map
double = multiply(2)
print(f"\ndouble = multiply(2)")
numbers = [1, 2, 3, 4, 5]
doubled = list(map(double, numbers))
print(f"map(double, {numbers}) = {doubled}")
```

---

## 7. Partial Functions (functools.partial)

### ความหมาย

`functools.partial` ช่วย "freeze" บางส่วนของ arguments ของ function สร้าง function ใหม่ที่มี arguments บางตัวถูกกำหนดไว้แล้ว

### ตัวอย่างที่ 20: partial พื้นฐาน

```python
from functools import partial

# ตัวอย่างที่ 20
def power(base, exponent):
    return base ** exponent

# สร้าง function ใหม่ที่ fix exponent
square = partial(power, exponent=2)
cube = partial(power, exponent=3)
square_root = partial(power, exponent=0.5)

print(f"square(4) = {square(4)}")           # 4^2 = 16
print(f"cube(3) = {cube(3)}")               # 3^3 = 27
print(f"square_root(16) = {square_root(16)}")  # 16^0.5 = 4.0

# partial กับ positional args
def multiply(a, b):
    return a * b

double = partial(multiply, b=2)   # fix b=2
triple = partial(multiply, b=3)   # fix b=3

nums = [1, 2, 3, 4, 5]
print(f"\nDoubled: {list(map(double, nums))}")
print(f"Tripled: {list(map(triple, nums))}")
```

### ตัวอย่างที่ 21: partial ใช้งานจริง

```python
from functools import partial
import json

# ตัวอย่างที่ 21: partial ใน real use case

# 1. Custom print function
print_error = partial(print, "[ERROR]", sep=" ")
print_info = partial(print, "[INFO]", sep=" ")
print_warning = partial(print, "[WARN]", sep=" ")

print_info("โปรแกรมเริ่มทำงาน")
print_warning("หน่วยความจำใกล้เต็ม")
print_error("ไม่สามารถเชื่อมต่อฐานข้อมูลได้")

# 2. Custom JSON serializer
compact_json = partial(json.dumps, separators=(',', ':'))
pretty_json = partial(json.dumps, indent=2, ensure_ascii=False)

data = {"name": "Alice", "age": 25, "city": "กรุงเทพ"}
print(f"\nCompact: {compact_json(data)}")
print(f"Pretty:\n{pretty_json(data)}")

# 3. การสร้าง URL templates
def build_url(protocol, domain, path, params=None):
    url = f"{protocol}://{domain}/{path}"
    if params:
        query = "&".join(f"{k}={v}" for k, v in params.items())
        url += f"?{query}"
    return url

https_api = partial(build_url, "https", "api.example.com")
http_dev = partial(build_url, "http", "localhost:8000")

print(f"\n{https_api('users')}")
print(f"{https_api('products', params={'page': 1, 'limit': 20})}")
print(f"{http_dev('test')}")
```

---

## 8. Function Caching (functools.lru_cache)

### ความหมาย

`functools.lru_cache` เป็น decorator ที่เก็บผลลัพธ์ของ function calls ไว้ใน cache (LRU = Least Recently Used)

### ตัวอย่างที่ 22: lru_cache พื้นฐาน

```python
from functools import lru_cache
import time

# ตัวอย่างที่ 22
@lru_cache(maxsize=128)  # เก็บผลลัพธ์ได้ 128 entries
def fibonacci_cached(n: int) -> int:
    """Fibonacci ที่มี cache"""
    if n <= 1:
        return n
    return fibonacci_cached(n - 1) + fibonacci_cached(n - 2)

def fibonacci_no_cache(n: int) -> int:
    """Fibonacci แบบปกติ (ช้า)"""
    if n <= 1:
        return n
    return fibonacci_no_cache(n - 1) + fibonacci_no_cache(n - 2)

# เปรียบเทียบความเร็ว
print("เปรียบเทียบ Fibonacci with/without cache:")
print(f"{'n':>4} {'no cache (ms)':>15} {'cached (ms)':>12} {'speedup':>10}")
print("-" * 45)

for n in [10, 20, 30, 35]:
    start = time.time()
    r1 = fibonacci_no_cache(n)
    t_no_cache = time.time() - start
    
    # clear cache แล้วทดสอบ
    fibonacci_cached.cache_clear()
    start = time.time()
    r2 = fibonacci_cached(n)
    t_cached = time.time() - start
    
    speedup = t_no_cache / t_cached if t_cached > 0 else float('inf')
    print(f"{n:>4} {t_no_cache*1000:>14.3f}ms {t_cached*1000:>11.3f}ms {speedup:>9.1f}x")

# Cache info
print(f"\nCache info: {fibonacci_cached.cache_info()}")
```

### ตัวอย่างที่ 23: cache กับ @cache (Python 3.9+)

```python
from functools import cache  # ไม่จำกัด maxsize (Python 3.9+)
import time

@cache
def expensive_calculation(n: int) -> int:
    """การคำนวณที่ใช้เวลามาก (จำลอง)"""
    time.sleep(0.001)  # จำลองการทำงาน
    return n ** 2 + n

print("Test @cache:")
start = time.time()
for i in range(5):
    result = expensive_calculation(i)
    print(f"  expensive_calculation({i}) = {result}")
t1 = time.time() - start

print(f"\nครั้งที่ 1 ใช้เวลา: {t1*1000:.1f}ms")

# เรียกซ้ำ (ได้จาก cache)
start = time.time()
for i in range(5):
    result = expensive_calculation(i)
t2 = time.time() - start

print(f"ครั้งที่ 2 ใช้เวลา: {t2*1000:.1f}ms (จาก cache)")
print(f"เร็วขึ้น: {t1/t2:.0f}x")
print(f"Cache info: {expensive_calculation.cache_info()}")
```

---

## 9. Memoization

### ความหมาย

Memoization คือเทคนิคการเก็บผลลัพธ์ที่คำนวณแล้วเพื่อใช้ซ้ำ คล้ายกับ caching แต่ implement เอง

### ตัวอย่างที่ 24: Manual Memoization

```python
# ตัวอย่างที่ 24: Manual Memoization
def memoize(func):
    """Memoization decorator แบบ manual"""
    cache = {}
    
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    wrapper.cache = cache
    wrapper.cache_size = lambda: len(cache)
    wrapper.__wrapped__ = func
    wrapper.__name__ = func.__name__
    return wrapper

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# ทดสอบ
for i in range(15):
    print(f"  F({i:2}) = {fibonacci(i)}")

print(f"\nCache size: {fibonacci.cache_size()}")
print(f"Cached values (first 5): {list(fibonacci.cache.items())[:5]}")
```

### ตัวอย่างที่ 25: Memoization สำหรับ Expensive Computations

```python
# ตัวอย่างที่ 25: Memoization สำหรับการคำนวณซับซ้อน
import time

def memoize_with_ttl(ttl_seconds: float = 60):
    """Memoization พร้อม Time-to-Live (TTL)"""
    def decorator(func):
        cache = {}
        
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            now = time.time()
            
            # ตรวจสอบ cache
            if key in cache:
                result, timestamp = cache[key]
                if now - timestamp < ttl_seconds:
                    print(f"  [Cache HIT] {func.__name__}{args}")
                    return result
                else:
                    print(f"  [Cache EXPIRED] {func.__name__}{args}")
            else:
                print(f"  [Cache MISS] {func.__name__}{args}")
            
            # คำนวณใหม่
            result = func(*args, **kwargs)
            cache[key] = (result, now)
            return result
        
        def clear_cache():
            cache.clear()
        
        wrapper.clear_cache = clear_cache
        wrapper.__name__ = func.__name__
        return wrapper
    return decorator

@memoize_with_ttl(ttl_seconds=5)
def fetch_exchange_rate(from_currency: str, to_currency: str) -> float:
    """จำลองการดึงอัตราแลกเปลี่ยน (ช้า)"""
    time.sleep(0.1)  # จำลอง API call
    rates = {
        ("USD", "THB"): 35.5,
        ("EUR", "THB"): 38.2,
        ("JPY", "THB"): 0.24,
    }
    return rates.get((from_currency, to_currency), 0)

print("Test Memoization with TTL:")
fetch_exchange_rate("USD", "THB")  # MISS
fetch_exchange_rate("USD", "THB")  # HIT
fetch_exchange_rate("EUR", "THB")  # MISS
fetch_exchange_rate("USD", "THB")  # HIT
fetch_exchange_rate("EUR", "THB")  # HIT
```

---

## 10. Pure Functions vs Side Effects

### Pure Functions

Pure function คือ function ที่:
1. ให้ผลลัพธ์เดิมเสมอสำหรับ input เดิม
2. ไม่มี side effects (ไม่แก้ไขสิ่งใดนอก function)

### Side Effects

Side effects ได้แก่:
- แก้ไข global variable
- แก้ไข input argument (mutation)
- I/O operations (print, file, network)
- การเปลี่ยนแปลงสถานะ (state)

### ตัวอย่างที่ 26: Pure vs Impure Functions

```python
# ตัวอย่างที่ 26
# ❌ Impure functions
total = 0

def add_to_total_impure(n):  # แก้ global state
    global total
    total += n
    return total

items_list = []

def append_to_list_impure(item):  # แก้ argument หรือ global state
    items_list.append(item)
    return items_list

# ✅ Pure functions
def add_pure(a, b):  # ไม่แก้ state ใด
    return a + b

def append_pure(lst, item):  # ไม่แก้ input list
    return lst + [item]  # คืน list ใหม่

# ทดสอบ
print("=== Pure Functions ===")
lst1 = [1, 2, 3]
lst2 = append_pure(lst1, 4)
print(f"lst1 (ไม่เปลี่ยน): {lst1}")
print(f"lst2 (ใหม่): {lst2}")

print("\n=== Impure Functions ===")
print(f"total ก่อน: {total}")
add_to_total_impure(5)
add_to_total_impure(10)
print(f"total หลัง: {total}")
```

### ตัวอย่างที่ 27: ประโยชน์ของ Pure Functions

```python
# ตัวอย่างที่ 27
# Pure functions ทดสอบได้ง่าย
def calculate_tax(income: float, rate: float = 0.20) -> float:
    """Calculate tax - pure function"""
    return income * rate

def calculate_discount(price: float, discount_pct: float) -> float:
    """Calculate discount - pure function"""
    return price * (1 - discount_pct / 100)

def calculate_total(
    items: list[dict],
    discount_pct: float = 0,
    tax_rate: float = 0.07
) -> dict:
    """Calculate order total - pure function
    
    ไม่แก้ไข items และคืน dict ใหม่เสมอ
    """
    subtotal = sum(item["price"] * item["qty"] for item in items)
    discount_amount = subtotal * (discount_pct / 100)
    after_discount = subtotal - discount_amount
    tax_amount = calculate_tax(after_discount, tax_rate)
    total = after_discount + tax_amount
    
    return {
        "subtotal": subtotal,
        "discount": discount_amount,
        "after_discount": after_discount,
        "tax": tax_amount,
        "total": total
    }

# ทดสอบได้แม่นยำ เพราะผลลัพธ์แน่นอน
cart = [
    {"name": "Apple", "price": 30, "qty": 5},
    {"name": "Banana", "price": 20, "qty": 3},
    {"name": "Cherry", "price": 80, "qty": 2},
]

result = calculate_total(cart, discount_pct=10, tax_rate=0.07)
print("สรุปคำสั่งซื้อ:")
print(f"  Subtotal:      {result['subtotal']:>10,.2f} บาท")
print(f"  Discount (10%): {result['discount']:>9,.2f} บาท")
print(f"  After Discount: {result['after_discount']:>9,.2f} บาท")
print(f"  Tax (7%):       {result['tax']:>9,.2f} บาท")
print(f"  Total:          {result['total']:>9,.2f} บาท")

# ทดสอบความบริสุทธิ์
print(f"\ncart ไม่เปลี่ยนแปลง: {len(cart)} items")
result2 = calculate_total(cart, discount_pct=10, tax_rate=0.07)
print(f"ผลลัพธ์เหมือนกัน: {result == result2}")
```

---

## 11. ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Recursive Algorithms

```python
"""
Recursive Algorithm ต่างๆ
"""

def tower_of_hanoi(n: int, source: str, target: str, auxiliary: str) -> list:
    """Tower of Hanoi
    
    คืน list ของ moves ที่ต้องทำ
    """
    moves = []
    
    def hanoi(disks, src, tgt, aux):
        if disks == 1:
            moves.append(f"ย้ายจาก {src} ไป {tgt}")
            return
        hanoi(disks - 1, src, aux, tgt)
        moves.append(f"ย้ายจาก {src} ไป {tgt}")
        hanoi(disks - 1, aux, tgt, src)
    
    hanoi(n, source, target, auxiliary)
    return moves

# ทดสอบ Tower of Hanoi
print("Tower of Hanoi (3 disks):")
moves = tower_of_hanoi(3, "A", "C", "B")
for i, move in enumerate(moves, 1):
    print(f"  {i:2}. {move}")
print(f"รวม {len(moves)} moves (ต้องใช้ {2**3 - 1} moves)")

print()

# Binary Search แบบ recursive
def binary_search_recursive(arr: list, target: int, left: int, right: int) -> int:
    """Binary Search แบบ recursive"""
    if left > right:
        return -1
    
    mid = (left + right) // 2
    
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)

sorted_arr = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
print("Binary Search:")
for target in [23, 91, 50]:
    idx = binary_search_recursive(sorted_arr, target, 0, len(sorted_arr) - 1)
    print(f"  ค้นหา {target}: {'index '+str(idx) if idx != -1 else 'ไม่พบ'}")

# Flatten nested structure แบบ recursive
def deep_flatten(data) -> list:
    """Flatten ข้อมูลซ้อนกันลึกแค่ไหนก็ได้"""
    result = []
    for item in data:
        if isinstance(item, (list, tuple)):
            result.extend(deep_flatten(item))
        else:
            result.append(item)
    return result

nested = [1, [2, [3, [4, [5]]]], [6, 7], 8, [9, [10]]]
print(f"\nFlat: {deep_flatten(nested)}")
```

### โปรแกรมที่ 2: Functional Programming Patterns

```python
"""
Functional Programming Patterns ใน Python
"""
from functools import reduce, partial
from typing import Callable, TypeVar, List, Any

T = TypeVar('T')
R = TypeVar('R')

# Pipeline Pattern
def pipeline(*funcs: Callable) -> Callable:
    """สร้าง pipeline ของ transformations"""
    return reduce(lambda f, g: lambda x: g(f(x)), funcs)

# Transformation functions
def normalize(text: str) -> str:
    """ทำให้ข้อความเป็นมาตรฐาน"""
    return text.lower().strip()

def remove_punctuation(text: str) -> str:
    """ลบ punctuation"""
    import string
    return "".join(c for c in text if c not in string.punctuation)

def tokenize(text: str) -> list:
    """แบ่งเป็น tokens"""
    return text.split()

def remove_stopwords(words: list) -> list:
    """ลบ stop words"""
    stopwords = {"the", "a", "an", "is", "in", "on", "at", "to", "for", "of", "and"}
    return [w for w in words if w not in stopwords]

def count_words(words: list) -> dict:
    """นับความถี่คำ"""
    freq = {}
    for word in words:
        freq[word] = freq.get(word, 0) + 1
    return freq

def sort_by_frequency(word_freq: dict) -> list:
    """เรียงตามความถี่"""
    return sorted(word_freq.items(), key=lambda x: x[1], reverse=True)

# สร้าง text analysis pipeline
analyze_text = pipeline(
    normalize,
    remove_punctuation,
    tokenize,
    remove_stopwords,
    count_words,
    sort_by_frequency
)

# ทดสอบ
text = """
Python is a high-level, general-purpose programming language. 
Its design philosophy emphasizes code readability and simplicity. 
Python is dynamically-typed and has a large standard library.
Python is great for data science, web development, and automation.
"""

print("Text Analysis Pipeline:")
results = analyze_text(text)
print(f"\nTop 10 คำที่ใช้บ่อย:")
for word, count in results[:10]:
    bar = "█" * count
    print(f"  {word:<15} {count:>3} {bar}")

# Functional data processing
students = [
    {"name": "Alice", "score": 92, "dept": "CS"},
    {"name": "Bob", "score": 75, "dept": "Math"},
    {"name": "Charlie", "score": 88, "dept": "CS"},
    {"name": "Diana", "score": 68, "dept": "Physics"},
    {"name": "Eve", "score": 95, "dept": "CS"},
    {"name": "Frank", "score": 82, "dept": "Math"},
]

# Functional pipeline
def get_cs_students(students):
    return filter(lambda s: s["dept"] == "CS", students)

def get_high_achievers(students, threshold=80):
    return filter(lambda s: s["score"] >= threshold, students)

def get_names(students):
    return map(lambda s: s["name"], students)

def get_avg_score(students):
    scores = [s["score"] for s in students]
    return sum(scores) / len(scores) if scores else 0

# ใช้ pipeline
cs_students = list(get_cs_students(students))
cs_high = list(get_high_achievers(cs_students))
cs_high_names = list(get_names(cs_high))

print(f"\n\nCS students: {[s['name'] for s in cs_students]}")
print(f"CS high achievers (>=80): {cs_high_names}")
print(f"Average CS score: {get_avg_score(cs_students):.1f}")
```

---

## 12. แบบฝึกหัด

### แบบฝึกหัดข้อที่ 1: Recursive Power Function

```
จงเขียน power(base, exp) แบบ recursive
รองรับ exp ที่เป็นลบด้วย
```

**เฉลย:**

```python
def power(base: float, exp: int) -> float:
    """คำนวณ base^exp แบบ recursive"""
    if exp == 0:
        return 1
    if exp < 0:
        return 1 / power(base, -exp)
    if exp % 2 == 0:
        half = power(base, exp // 2)
        return half * half  # Fast power: a^(2n) = (a^n)^2
    return base * power(base, exp - 1)

# ทดสอบ
tests = [(2, 10), (3, 5), (2, -3), (5, 0), (10, -2)]
for base, exp in tests:
    result = power(base, exp)
    expected = base ** exp
    match = "✓" if abs(result - expected) < 0.0001 else "✗"
    print(f"{base}^{exp:>3} = {result:>12.6f} (ตรวจสอบ: {expected:>12.6f}) {match}")
```

---

### แบบฝึกหัดข้อที่ 2: Decorator สำหรับ Input Validation

```
จงเขียน decorator ที่ validate type ของ arguments ตาม type hints
```

**เฉลย:**

```python
import functools
import inspect

def validate_types(func):
    """Decorator ตรวจสอบ type ของ arguments ตาม type hints"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        hints = func.__annotations__
        sig = inspect.signature(func)
        params = list(sig.parameters.keys())
        
        # ตรวจสอบ positional args
        for i, (param_name, value) in enumerate(zip(params, args)):
            if param_name in hints and hints[param_name] != type(None):
                expected_type = hints[param_name]
                if not isinstance(value, expected_type):
                    raise TypeError(
                        f"Parameter '{param_name}' ต้องเป็น {expected_type.__name__} "
                        f"ไม่ใช่ {type(value).__name__}"
                    )
        
        return func(*args, **kwargs)
    return wrapper

@validate_types
def calculate_bmi(weight: float, height: float) -> float:
    return weight / (height ** 2)

# ทดสอบ
try:
    print(f"BMI: {calculate_bmi(70.0, 1.75):.2f}")
    print(f"BMI: {calculate_bmi(60, 1.65):.2f}")   # int แทน float - อาจผ่านเพราะ int เป็น subtype
    calculate_bmi("70", 1.75)  # ❌ Error
except TypeError as e:
    print(f"✗ {e}")
```

---

### แบบฝึกหัดข้อที่ 3: Memoization สำหรับ Recursive

```
จงเขียน recursive function หา Catalan number
พร้อม memoization
```

**เฉลย:**

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def catalan(n: int) -> int:
    """Catalan number C(n) = sum(C(i)*C(n-1-i)) for i=0..n-1"""
    if n <= 1:
        return 1
    return sum(catalan(i) * catalan(n - 1 - i) for i in range(n))

print("Catalan Numbers:")
for i in range(12):
    print(f"  C({i:2}) = {catalan(i):,}")

print(f"\nCache info: {catalan.cache_info()}")

# ใช้งาน: C(n) นับจำนวน balanced parentheses expressions ที่ยาว 2n
def generate_parentheses(n: int) -> list:
    """Generate balanced parentheses"""
    result = []
    def generate(s, open_count, close_count):
        if len(s) == 2 * n:
            result.append(s)
            return
        if open_count < n:
            generate(s + "(", open_count + 1, close_count)
        if close_count < open_count:
            generate(s + ")", open_count, close_count + 1)
    
    generate("", 0, 0)
    return result

for n in range(1, 5):
    parens = generate_parentheses(n)
    print(f"n={n}: {len(parens)} ways = C({n}) = {catalan(n)}")
    print(f"  {parens}")
```

---

### แบบฝึกหัดข้อที่ 4: Function Composition

```
จงสร้าง function pipeline ที่:
- รับรายการ numbers
- กรองเฉพาะเลขบวก
- ยกกำลัง 2
- บวก 10
- เรียงจากน้อยไปมาก
```

**เฉลย:**

```python
from functools import reduce
from typing import Callable, List

def compose(*funcs: Callable) -> Callable:
    """Compose functions (right to left)"""
    def composed(x):
        return reduce(lambda v, f: f(v), reversed(funcs), x)
    return composed

# Define transformations
filter_positive: Callable = lambda nums: [n for n in nums if n > 0]
square_all: Callable = lambda nums: [n**2 for n in nums]
add_ten: Callable = lambda nums: [n + 10 for n in nums]
sort_asc: Callable = lambda nums: sorted(nums)

# สร้าง pipeline (right to left execution)
process = compose(sort_asc, add_ten, square_all, filter_positive)

# ทดสอบ
data = [-5, 3, -2, 7, -1, 4, 0, -8, 6, 2]
print(f"Input:          {data}")
print(f"After filter:   {filter_positive(data)}")
print(f"After square:   {square_all(filter_positive(data))}")
print(f"After add 10:   {add_ten(square_all(filter_positive(data)))}")
print(f"After sort:     {process(data)}")
```

---

### แบบฝึกหัดข้อที่ 5: Cached Fibonacci with Stats

```
จงเขียน Fibonacci ที่:
- ใช้ lru_cache
- นับ cache hits vs misses
- แสดง performance stats
```

**เฉลย:**

```python
import time

class FibonacciCalculator:
    """Fibonacci Calculator พร้อม caching stats"""
    
    def __init__(self):
        self.cache = {}
        self.hits = 0
        self.misses = 0
        self.call_count = 0
    
    def calculate(self, n: int) -> int:
        """คำนวณ Fibonacci พร้อมเก็บ stats"""
        self.call_count += 1
        
        if n in self.cache:
            self.hits += 1
            return self.cache[n]
        
        self.misses += 1
        
        if n <= 1:
            result = n
        else:
            result = self.calculate(n - 1) + self.calculate(n - 2)
        
        self.cache[n] = result
        return result
    
    def stats(self):
        total = self.hits + self.misses
        hit_rate = self.hits / total * 100 if total > 0 else 0
        print(f"\nCache Stats:")
        print(f"  Total calls:  {self.call_count:,}")
        print(f"  Cache hits:   {self.hits:,} ({hit_rate:.1f}%)")
        print(f"  Cache misses: {self.misses:,}")
        print(f"  Cache size:   {len(self.cache):,}")
    
    def reset_stats(self):
        self.hits = 0
        self.misses = 0
        self.call_count = 0

calc = FibonacciCalculator()

print("Calculate F(0) to F(20):")
start = time.time()
for n in range(21):
    result = calc.calculate(n)
    if n % 5 == 0:
        print(f"  F({n:2}) = {result:,}")
elapsed = time.time() - start

print(f"\nใช้เวลา: {elapsed*1000:.3f}ms")
calc.stats()

# เรียกซ้ำ
calc.reset_stats()
print("\nCalculate F(5) to F(15) ซ้ำ:")
for n in range(5, 16):
    calc.calculate(n)
calc.stats()
```

---

### แบบฝึกหัดข้อที่ 6: Partial Application Chain

```
จงสร้าง function ที่สร้าง greeting ต่างๆ
โดยใช้ partial application
```

**เฉลย:**

```python
from functools import partial

def create_greeting(
    prefix: str,
    name: str,
    *,
    suffix: str = "",
    language: str = "en",
    formal: bool = False
) -> str:
    """สร้าง greeting"""
    if language == "th":
        formal_prefix = "คุณ" if formal else ""
        return f"{prefix} {formal_prefix}{name}{suffix}"
    else:
        honorific = "Mr./Ms." if formal else ""
        return f"{prefix} {honorific} {name}{suffix}".strip()

# สร้าง specialized greetings
hello = partial(create_greeting, "Hello")
hi = partial(create_greeting, "Hi")
welcome = partial(create_greeting, "Welcome,", suffix="!")
formal_greeting = partial(create_greeting, "Good day,", formal=True)

sawasdi = partial(create_greeting, "สวัสดี", language="th")
formal_thai = partial(create_greeting, "สวัสดีครับ", language="th", formal=True)

# ทดสอบ
names = ["Alice", "Bob", "Charlie"]

print("English Greetings:")
for name in names:
    print(f"  hello: {hello(name)}")
    print(f"  formal: {formal_greeting(name)}")

print("\nThai Greetings:")
for name in ["สมชาย", "สมหญิง"]:
    print(f"  informal: {sawasdi(name)}")
    print(f"  formal: {formal_thai(name)}")
```

---

### แบบฝึกหัดข้อที่ 7-10 (สรุปสั้น)

```python
# ข้อ 7: Recursive Flatten with depth limit
def flatten(data, max_depth=None, current_depth=0):
    """Flatten nested list พร้อม depth limit"""
    result = []
    for item in data:
        if isinstance(item, list) and (max_depth is None or current_depth < max_depth):
            result.extend(flatten(item, max_depth, current_depth + 1))
        else:
            result.append(item)
    return result

data = [1, [2, [3, [4, [5]]]]]
print(f"depth=None: {flatten(data)}")
print(f"depth=1:    {flatten(data, max_depth=1)}")
print(f"depth=2:    {flatten(data, max_depth=2)}")

# ข้อ 8: Function Pipeline Builder
def build_pipeline(*steps):
    """สร้าง pipeline จาก steps"""
    def run(data):
        result = data
        for step in steps:
            result = step(result)
        return result
    return run

normalize = lambda s: s.lower().strip()
tokenize = lambda s: s.split()
count = lambda words: len(words)

word_counter = build_pipeline(normalize, tokenize, count)
print(f"\nจำนวนคำ: {word_counter('  Hello World Python  ')}")

# ข้อ 9: Memoized LCS (Longest Common Subsequence)
from functools import lru_cache

def lcs(s1: str, s2: str) -> int:
    """Longest Common Subsequence"""
    @lru_cache(maxsize=None)
    def dp(i, j):
        if i == 0 or j == 0:
            return 0
        if s1[i-1] == s2[j-1]:
            return 1 + dp(i-1, j-1)
        return max(dp(i-1, j), dp(i, j-1))
    
    return dp(len(s1), len(s2))

pairs = [("ABCBDAB", "BDCAB"), ("AGGTAB", "GXTXAYB")]
for s1, s2 in pairs:
    print(f"\nLCS('{s1}', '{s2}') = {lcs(s1, s2)}")

# ข้อ 10: Higher-order Statistics
print("\n(ข้อ 10: ลองเขียน function สร้าง custom aggregators)")
```

---

## สรุป Part 10

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| Keyword-only args | `*` บังคับใช้ keyword |
| Positional-only args | `/` บังคับใช้ positional |
| Type hints | annotations ช่วย IDE และ docs |
| Recursive | base case + recursive case |
| Closures | function จำ enclosing scope |
| Higher-order | รับ/คืน function |
| partial | freeze บาง arguments |
| lru_cache | caching อัตโนมัติ |
| Memoization | เก็บผลลัพธ์ใช้ซ้ำ |
| Pure functions | ไม่มี side effects |

### Parameter Types สรุป

```python
def f(pos_only, /, normal, *args, kw_only, **kwargs):
    pass
#   pos_only: positional-only (ก่อน /)
#   normal: positional หรือ keyword
#   *args: extra positional (tuple)
#   kw_only: keyword-only (หลัง *)
#   **kwargs: extra keyword (dict)
```

### Key Takeaways:
1. **Keyword-only** ช่วยป้องกันการส่ง argument ผิดลำดับ
2. **Type hints** ไม่ enforce ที่ runtime แต่ช่วย documentation
3. Recursive ต้องมี **base case** เสมอ
4. **Closure** เก็บ state ใน enclosing scope
5. **lru_cache** ช่วยเร็วขึ้นอย่างมากสำหรับ pure functions ที่เรียกซ้ำ
6. **Pure functions** ทดสอบง่ายและ predictable กว่า

---

*Part 10 จบแล้ว ยินดีด้วย! คุณเรียนรู้ Functions ทั้งพื้นฐานและขั้นสูงแล้ว*

*ไปต่อที่ [Part 11](../part11/README.md)*
