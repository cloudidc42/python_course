# Part 26: Decorators - From Basics to Advanced

## สารบัญ
1. [Functions as First-Class Objects](#functions-as-first-class-objects)
2. [Decorator Concept และ Syntax](#decorator-concept-และ-syntax)
3. [Simple Decorator](#simple-decorator)
4. [Decorator with Arguments](#decorator-with-arguments)
5. [Class Decorators](#class-decorators)
6. [Stacking Decorators](#stacking-decorators)
7. [functools.wraps](#functoolswraps)
8. [Built-in Decorators](#built-in-decorators)
9. [@functools.lru_cache](#functoolslru_cache)
10. [@dataclass เบื้องต้น](#dataclass-เบื้องต้น)
11. [Practical Decorators](#practical-decorators)
12. [Decorator Patterns ในการทำงานจริง](#decorator-patterns-ในการทำงานจริง)
13. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Functions as First-Class Objects

ใน Python ฟังก์ชันเป็น **first-class objects** หมายความว่าฟังก์ชันสามารถ:
- ถูกเก็บในตัวแปร
- ถูกส่งเป็น argument ให้ฟังก์ชันอื่น
- ถูก return จากฟังก์ชัน
- ถูกเก็บใน data structure (list, dict, ฯลฯ)

แนวคิดนี้เป็นรากฐานของ decorators

### ตัวอย่าง 1: ฟังก์ชันเก็บในตัวแปร

```python
def greet(name):
    return f"สวัสดี, {name}!"

# เก็บฟังก์ชันในตัวแปร (ไม่ใช่เรียกใช้งาน ไม่มีวงเล็บ)
say_hello = greet
print(say_hello("สมชาย"))   # สวัสดี, สมชาย!
print(greet("สมหญิง"))      # สวัสดี, สมหญิง!

# ดูว่าทั้งสองชี้ไปที่ object เดียวกัน
print(say_hello is greet)    # True
print(id(say_hello) == id(greet))  # True
```

### ตัวอย่าง 2: ฟังก์ชันเป็น Argument

```python
def apply_operation(x, y, operation):
    """รับฟังก์ชันเป็น argument และเรียกใช้งาน"""
    return operation(x, y)

def add(x, y):
    return x + y

def multiply(x, y):
    return x * y

def power(x, y):
    return x ** y

# ส่งฟังก์ชันเป็น argument
result1 = apply_operation(3, 4, add)       # 7
result2 = apply_operation(3, 4, multiply)  # 12
result3 = apply_operation(3, 4, power)     # 81

print(result1, result2, result3)

# ใช้ lambda
result4 = apply_operation(10, 3, lambda x, y: x - y)  # 7
print(result4)
```

### ตัวอย่าง 3: ฟังก์ชัน Return ฟังก์ชัน (Higher-Order Functions)

```python
def make_multiplier(factor):
    """สร้างฟังก์ชันที่คูณด้วย factor"""
    def multiplier(number):
        return number * factor
    return multiplier  # return ฟังก์ชัน ไม่ใช่ผลลัพธ์

# สร้างฟังก์ชันต่างๆ
double = make_multiplier(2)
triple = make_multiplier(3)
times_ten = make_multiplier(10)

print(double(5))     # 10
print(triple(5))     # 15
print(times_ten(5))  # 50

# ฟังก์ชันที่ถูกสร้างมา "จำ" ค่า factor (closure)
print(double.__closure__[0].cell_contents)  # 2
```

### ตัวอย่าง 4: Closure และ Scope

```python
def outer(message):
    """outer function"""
    
    def inner():
        # inner สามารถเข้าถึง variable ของ outer (enclosing scope)
        print(f"ข้อความ: {message}")
    
    return inner

# สร้าง closure
hello_func = outer("สวัสดีโลก!")
bye_func = outer("ลาก่อน!")

hello_func()  # ข้อความ: สวัสดีโลก!
bye_func()    # ข้อความ: ลาก่อน!

# แต่ละ closure มีสำเนา environment ของตัวเอง
print(hello_func.__closure__[0].cell_contents)  # สวัสดีโลก!
print(bye_func.__closure__[0].cell_contents)    # ลาก่อน!
```

### ตัวอย่าง 5: ฟังก์ชันใน Data Structure

```python
# เก็บฟังก์ชันใน dict - pattern นี้ใช้บ่อยมากในการทำ dispatch table
operations = {
    'add': lambda x, y: x + y,
    'sub': lambda x, y: x - y,
    'mul': lambda x, y: x * y,
    'div': lambda x, y: x / y if y != 0 else "Error: division by zero"
}

def calculator(op, x, y):
    if op in operations:
        return operations[op](x, y)
    return "Unknown operation"

print(calculator('add', 10, 5))   # 15
print(calculator('mul', 3, 7))    # 21
print(calculator('div', 10, 0))   # Error: division by zero
print(calculator('pow', 2, 8))    # Unknown operation

# เก็บฟังก์ชันใน list
transformations = [str.upper, str.strip, str.title]
text = "  hello world  "
result = text
for transform in transformations:
    result = transform(result)
print(result)  # Hello World
```

---

## Decorator Concept และ Syntax

**Decorator** คือฟังก์ชันที่รับฟังก์ชันเป็น argument, เพิ่ม/แก้ไข behavior บางอย่าง, แล้ว return ฟังก์ชันใหม่ (หรือฟังก์ชันเดิมที่ถูก wrapped)

แนวคิดหลักคือ: **"Wrap" ฟังก์ชันเพื่อเพิ่ม behavior โดยไม่แก้ไขโค้ดเดิม**

```
ฟังก์ชันเดิม → Decorator → ฟังก์ชันใหม่ที่มี behavior เพิ่มเติม
```

### ตัวอย่าง 6: Decorator แบบ Manual (ก่อนใช้ @ syntax)

```python
def my_decorator(func):
    """นี่คือ decorator - รับฟังก์ชัน, return ฟังก์ชันใหม่"""
    def wrapper():
        print("ก่อนเรียกฟังก์ชัน")
        func()  # เรียกฟังก์ชันเดิม
        print("หลังเรียกฟังก์ชัน")
    return wrapper

def say_hello():
    print("สวัสดี!")

# ใช้ decorator แบบ manual
say_hello = my_decorator(say_hello)
say_hello()
# Output:
# ก่อนเรียกฟังก์ชัน
# สวัสดี!
# หลังเรียกฟังก์ชัน
```

### ตัวอย่าง 7: @ Syntax (Syntactic Sugar)

```python
def my_decorator(func):
    def wrapper():
        print("ก่อนเรียกฟังก์ชัน")
        func()
        print("หลังเรียกฟังก์ชัน")
    return wrapper

# @ syntax ทำงานเหมือน: say_hello = my_decorator(say_hello)
@my_decorator
def say_hello():
    print("สวัสดี!")

say_hello()
# Output:
# ก่อนเรียกฟังก์ชัน
# สวัสดี!
# หลังเรียกฟังก์ชัน
```

---

## Simple Decorator

### ตัวอย่าง 8: Decorator พื้นฐานพร้อม Arguments และ Return Value

```python
import functools

def simple_logger(func):
    """Decorator สำหรับ log การเรียกใช้ฟังก์ชัน"""
    @functools.wraps(func)  # รักษา metadata ของฟังก์ชันเดิม
    def wrapper(*args, **kwargs):
        print(f"กำลังเรียก: {func.__name__}")
        print(f"  Arguments: {args}, Kwargs: {kwargs}")
        result = func(*args, **kwargs)
        print(f"  ผลลัพธ์: {result}")
        return result
    return wrapper

@simple_logger
def add(x, y):
    """บวกเลขสองตัว"""
    return x + y

@simple_logger
def greet(name, greeting="สวัสดี"):
    """สร้างการทักทาย"""
    return f"{greeting}, {name}!"

result = add(3, 4)
print("---")
msg = greet("สมชาย", greeting="ดีครับ")
```

### ตัวอย่าง 9: *args และ **kwargs ใน Decorator

```python
import functools

def verbose(func):
    """Decorator ที่รองรับฟังก์ชันทุกชนิด"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # *args รับ positional arguments ทั้งหมด
        # **kwargs รับ keyword arguments ทั้งหมด
        print(f"เรียก {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"  → {result}")
        return result
    return wrapper

@verbose
def no_args():
    return "ไม่มี args"

@verbose
def one_arg(x):
    return x * 2

@verbose
def many_args(a, b, c=10, d=20):
    return a + b + c + d

no_args()
one_arg(5)
many_args(1, 2, d=30)
```

---

## Decorator with Arguments

บางครั้งเราต้องการส่ง parameter ให้ decorator เอง เช่น `@repeat(3)` - ต้องการให้ฟังก์ชันทำงาน 3 ครั้ง

วิธีทำคือสร้าง **decorator factory** - ฟังก์ชันที่ return decorator อีกที

```
decorator_factory(args) → decorator → wrapper
```

### ตัวอย่าง 10: Decorator Factory พื้นฐาน

```python
import functools

def repeat(times):
    """Decorator factory - รับจำนวนครั้งที่จะทำซ้ำ"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(times):
                result = func(*args, **kwargs)
            return result  # return ผลลัพธ์ครั้งสุดท้าย
        return wrapper
    return decorator  # return decorator ไม่ใช่ wrapper

@repeat(3)  # ทำงาน 3 ครั้ง
def say_hello(name):
    print(f"สวัสดี {name}!")

@repeat(1)
def do_once():
    print("ทำแค่ครั้งเดียว")

say_hello("โลก")
print("---")
do_once()
```

### ตัวอย่าง 11: Decorator with Prefix/Suffix

```python
import functools

def add_text(prefix="", suffix=""):
    """เพิ่ม prefix และ suffix ให้ return value"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            return f"{prefix}{result}{suffix}"
        return wrapper
    return decorator

@add_text(prefix="[INFO] ", suffix=" ✓")
def get_message():
    return "การทำงานสำเร็จ"

@add_text(prefix=">>> ")
def get_code():
    return "print('hello')"

print(get_message())  # [INFO] การทำงานสำเร็จ ✓
print(get_code())     # >>> print('hello')
```

### ตัวอย่าง 12: Retry Decorator พร้อม Parameters

```python
import functools
import time
import random

def retry(max_attempts=3, delay=1.0, exceptions=(Exception,)):
    """
    Decorator สำหรับ retry เมื่อเกิด exception
    
    Args:
        max_attempts: จำนวนครั้งสูงสุดที่จะ retry
        delay: เวลารอระหว่างการ retry (วินาที)
        exceptions: tuple ของ exception ที่จะ retry
    """
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt < max_attempts:
                        print(f"  ครั้งที่ {attempt} ล้มเหลว: {e}. รอ {delay}s แล้ว retry...")
                        time.sleep(delay)
                    else:
                        print(f"  ครั้งที่ {attempt} ล้มเหลว: {e}. หมดจำนวน retry แล้ว!")
            raise last_exception
        return wrapper
    return decorator

# จำลองฟังก์ชันที่ล้มเหลวบ้างบางครั้ง
attempt_count = 0

@retry(max_attempts=3, delay=0.1, exceptions=(ValueError,))
def unstable_function():
    global attempt_count
    attempt_count += 1
    if attempt_count < 3:
        raise ValueError(f"Error ครั้งที่ {attempt_count}")
    return "สำเร็จ!"

result = unstable_function()
print(f"ผลลัพธ์: {result}")
```

---

## Class Decorators

นอกจากใช้ฟังก์ชันเป็น decorator ยังสามารถใช้ **คลาส** เป็น decorator ได้ โดยต้องมี `__call__` method

### ตัวอย่าง 13: Class-based Decorator

```python
import functools
import time

class Timer:
    """Class decorator สำหรับวัดเวลา"""
    
    def __init__(self, func):
        # เก็บฟังก์ชันเดิมไว้
        functools.update_wrapper(self, func)
        self.func = func
        self.total_time = 0
        self.call_count = 0
    
    def __call__(self, *args, **kwargs):
        start = time.perf_counter()
        result = self.func(*args, **kwargs)
        end = time.perf_counter()
        
        elapsed = end - start
        self.total_time += elapsed
        self.call_count += 1
        
        print(f"{self.func.__name__} ใช้เวลา {elapsed:.6f}s")
        return result
    
    def stats(self):
        if self.call_count > 0:
            avg = self.total_time / self.call_count
            print(f"สถิติ: เรียก {self.call_count} ครั้ง, "
                  f"รวม {self.total_time:.6f}s, "
                  f"เฉลี่ย {avg:.6f}s")

@Timer
def slow_function(n):
    """ฟังก์ชันที่ช้า"""
    total = sum(range(n))
    return total

slow_function(100000)
slow_function(200000)
slow_function(300000)
slow_function.stats()
```

### ตัวอย่าง 14: Class Decorator พร้อม Arguments

```python
import functools

class validate_types:
    """Class decorator สำหรับ validate ประเภท arguments"""
    
    def __init__(self, **type_map):
        self.type_map = type_map
    
    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            for param_name, value in bound.arguments.items():
                if param_name in self.type_map:
                    expected_type = self.type_map[param_name]
                    if not isinstance(value, expected_type):
                        raise TypeError(
                            f"Parameter '{param_name}' ต้องเป็น {expected_type.__name__}, "
                            f"แต่ได้รับ {type(value).__name__}"
                        )
            return func(*args, **kwargs)
        return wrapper

@validate_types(name=str, age=int)
def create_user(name, age):
    return f"User: {name}, อายุ: {age}"

# ถูกต้อง
print(create_user("สมชาย", 25))

# ผิดประเภท - จะ raise TypeError
try:
    print(create_user("สมหญิง", "ยี่สิบห้า"))
except TypeError as e:
    print(f"Error: {e}")
```

---

## Stacking Decorators

สามารถใช้หลาย decorator บนฟังก์ชันเดียวได้ โดย Python จะใช้งานจาก **ล่างขึ้นบน** (bottom to top) ตอน apply แต่ทำงานจาก **บนลงล่าง** ตอนเรียกใช้

```
@decorator_a
@decorator_b
@decorator_c
def func():
    pass

# เทียบเท่ากับ:
func = decorator_a(decorator_b(decorator_c(func)))
```

### ตัวอย่าง 15: ลำดับการทำงานของ Stacked Decorators

```python
import functools

def decorator_1(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator 1: ก่อน")
        result = func(*args, **kwargs)
        print("Decorator 1: หลัง")
        return result
    return wrapper

def decorator_2(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator 2: ก่อน")
        result = func(*args, **kwargs)
        print("Decorator 2: หลัง")
        return result
    return wrapper

def decorator_3(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator 3: ก่อน")
        result = func(*args, **kwargs)
        print("Decorator 3: หลัง")
        return result
    return wrapper

@decorator_1
@decorator_2
@decorator_3
def my_function():
    print("  ฟังก์ชันหลัก")

my_function()
# Output:
# Decorator 1: ก่อน
# Decorator 2: ก่อน
# Decorator 3: ก่อน
#   ฟังก์ชันหลัก
# Decorator 3: หลัง
# Decorator 2: หลัง
# Decorator 1: หลัง
```

### ตัวอย่าง 16: ตัวอย่างจริงของ Stacking

```python
import functools
import time

def log_call(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"[LOG] เรียก {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

def measure_time(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"[TIME] {func.__name__}: {elapsed:.4f}s")
        return result
    return wrapper

def validate_positive(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        for arg in args:
            if isinstance(arg, (int, float)) and arg < 0:
                raise ValueError(f"ค่าต้องเป็นบวก, ได้รับ: {arg}")
        return func(*args, **kwargs)
    return wrapper

@log_call
@measure_time
@validate_positive
def calculate(n):
    return sum(range(n))

result = calculate(1000000)
print(f"ผลลัพธ์: {result}")

try:
    calculate(-1)
except ValueError as e:
    print(f"Error: {e}")
```

---

## functools.wraps

เมื่อเราสร้าง decorator ฟังก์ชัน `wrapper` จะแทนที่ข้อมูล metadata ของฟังก์ชันเดิม เช่น `__name__`, `__doc__`, `__module__` 

`functools.wraps` แก้ปัญหานี้โดยคัดลอก metadata จากฟังก์ชันเดิมมาให้ `wrapper`

### ตัวอย่าง 17: ปัญหาโดยไม่ใช้ functools.wraps

```python
# ไม่ใช้ functools.wraps
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_important_function():
    """นี่คือ docstring ที่สำคัญมาก"""
    pass

# Metadata หายไป!
print(my_important_function.__name__)  # wrapper (ผิด!)
print(my_important_function.__doc__)   # None (ผิด!)

import inspect
print(inspect.getdoc(my_important_function))  # None
```

### ตัวอย่าง 18: แก้ปัญหาด้วย functools.wraps

```python
import functools

# ใช้ functools.wraps
def good_decorator(func):
    @functools.wraps(func)  # คัดลอก metadata จาก func ไปยัง wrapper
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@good_decorator
def my_important_function():
    """นี่คือ docstring ที่สำคัญมาก"""
    pass

# Metadata ถูกต้อง!
print(my_important_function.__name__)  # my_important_function ✓
print(my_important_function.__doc__)   # นี่คือ docstring ที่สำคัญมาก ✓

# Attributes ที่ functools.wraps คัดลอก:
# __module__, __name__, __qualname__, __annotations__,
# __doc__, __dict__, __wrapped__

# __wrapped__ ให้เข้าถึงฟังก์ชันเดิม
print(my_important_function.__wrapped__)  # <function my_important_function at 0x...>
```

---

## Built-in Decorators

Python มี built-in decorators หลายตัวที่ใช้บ่อยมาก

### @property

`@property` ทำให้ method เรียกใช้เหมือน attribute ป้องกัน direct access ไปยัง internal state

### ตัวอย่าง 19: @property

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius  # _ บอกว่าเป็น "internal" attribute
    
    @property
    def radius(self):
        """Getter: อ่านค่า radius"""
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """Setter: กำหนดค่า radius พร้อม validation"""
        if value < 0:
            raise ValueError("radius ต้องมีค่าบวก")
        self._radius = value
    
    @radius.deleter
    def radius(self):
        """Deleter: ลบ radius"""
        print("กำลังลบ radius")
        del self._radius
    
    @property
    def diameter(self):
        """Computed property - คำนวณจาก radius"""
        return self._radius * 2
    
    @property
    def area(self):
        """Computed property - คำนวณพื้นที่"""
        import math
        return math.pi * self._radius ** 2

c = Circle(5)
print(f"radius: {c.radius}")     # 5 (เรียกเหมือน attribute ไม่ใส่วงเล็บ)
print(f"diameter: {c.diameter}") # 10
print(f"area: {c.area:.2f}")     # 78.54

c.radius = 10  # ใช้ setter
print(f"radius ใหม่: {c.radius}")

try:
    c.radius = -1  # Validation จะ raise
except ValueError as e:
    print(f"Error: {e}")
```

### ตัวอย่าง 20: @classmethod

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_string):
        """Alternative constructor จาก string 'YYYY-MM-DD'"""
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)  # cls คือ class Date เอง
    
    @classmethod
    def today(cls):
        """สร้าง Date object ของวันนี้"""
        import datetime
        today = datetime.date.today()
        return cls(today.year, today.month, today.day)
    
    def __repr__(self):
        return f"Date({self.year}, {self.month}, {self.day})"
    
    def __str__(self):
        return f"{self.year:04d}-{self.month:02d}-{self.day:02d}"

# สร้างแบบปกติ
d1 = Date(2024, 1, 15)

# สร้างผ่าน alternative constructor
d2 = Date.from_string("2024-06-20")
d3 = Date.today()

print(d1)   # 2024-01-15
print(d2)   # 2024-06-20
print(d3)   # วันปัจจุบัน
```

### ตัวอย่าง 21: @staticmethod

```python
class MathUtils:
    """Class ที่รวม utility functions ทางคณิตศาสตร์"""
    
    @staticmethod
    def is_prime(n):
        """ตรวจสอบว่า n เป็นจำนวนเฉพาะหรือไม่
        ไม่ต้องการ self หรือ cls เลย"""
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True
    
    @staticmethod
    def gcd(a, b):
        """หาตัวหารร่วมมาก"""
        while b:
            a, b = b, a % b
        return a
    
    @staticmethod
    def fibonacci(n):
        """สร้าง Fibonacci sequence n ตัว"""
        if n <= 0:
            return []
        elif n == 1:
            return [0]
        fibs = [0, 1]
        for _ in range(n - 2):
            fibs.append(fibs[-1] + fibs[-2])
        return fibs

# เรียกผ่าน class (ไม่ต้องสร้าง instance)
print(MathUtils.is_prime(17))      # True
print(MathUtils.is_prime(20))      # False
print(MathUtils.gcd(48, 18))       # 6
print(MathUtils.fibonacci(10))     # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# เรียกผ่าน instance ก็ได้ (แต่ไม่ได้รับ self)
utils = MathUtils()
print(utils.is_prime(13))          # True
```

### ตัวอย่าง 22: เปรียบเทียบ instance method, classmethod, staticmethod

```python
class Animal:
    species_count = 0  # class variable
    
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound
        Animal.species_count += 1
    
    def make_sound(self):
        """Instance method: เข้าถึง self (instance)"""
        return f"{self.name} พูดว่า: {self.sound}!"
    
    @classmethod
    def get_species_count(cls):
        """Class method: เข้าถึง cls (class) ไม่ใช่ instance"""
        return f"มีสัตว์ทั้งหมด {cls.species_count} ชนิด"
    
    @staticmethod
    def is_animal(obj):
        """Static method: ไม่ต้องการ self หรือ cls"""
        return isinstance(obj, Animal)

dog = Animal("หมา", "โฮ่ง")
cat = Animal("แมว", "เมี้ยว")

print(dog.make_sound())              # instance method
print(Animal.get_species_count())    # class method
print(Animal.is_animal(dog))         # static method -> True
print(Animal.is_animal("not animal")) # static method -> False
```

---

## @functools.lru_cache

`lru_cache` ย่อมาจาก **Least Recently Used Cache** - เก็บผลลัพธ์ของฟังก์ชันไว้ใน memory เพื่อไม่ต้องคำนวณซ้ำ

### ตัวอย่าง 23: lru_cache พื้นฐาน

```python
import functools
import time

# โดยไม่ใช้ cache - ช้ามาก
def fibonacci_slow(n):
    if n < 2:
        return n
    return fibonacci_slow(n-1) + fibonacci_slow(n-2)

# ใช้ lru_cache - เร็วมาก
@functools.lru_cache(maxsize=None)  # maxsize=None = ไม่จำกัด
def fibonacci_fast(n):
    if n < 2:
        return n
    return fibonacci_fast(n-1) + fibonacci_fast(n-2)

# เปรียบเทียบความเร็ว
start = time.perf_counter()
result_slow = fibonacci_slow(35)
print(f"Slow: {result_slow}, เวลา: {time.perf_counter()-start:.3f}s")

start = time.perf_counter()
result_fast = fibonacci_fast(35)
print(f"Fast: {result_fast}, เวลา: {time.perf_counter()-start:.6f}s")

# ดู cache info
print(fibonacci_fast.cache_info())
# CacheInfo(hits=33, misses=36, maxsize=None, currsize=36)

# ล้าง cache
fibonacci_fast.cache_clear()
print(fibonacci_fast.cache_info())
```

### ตัวอย่าง 24: lru_cache กับ maxsize

```python
import functools

@functools.lru_cache(maxsize=128)  # เก็บได้สูงสุด 128 ค่า
def expensive_computation(x, y):
    """จำลองการคำนวณที่หนัก"""
    import time
    time.sleep(0.001)  # จำลองความช้า
    return x ** 2 + y ** 2

# ครั้งแรก - คำนวณจริง
result1 = expensive_computation(3, 4)
result2 = expensive_computation(3, 4)  # ดึงจาก cache

print(expensive_computation.cache_info())
# Hits จะเป็น 1 (ครั้งที่ 2 มาจาก cache)

# หมายเหตุ: arguments ต้องเป็น hashable!
# ไม่รองรับ list, dict (ต้องแปลงเป็น tuple ก่อน)
@functools.lru_cache(maxsize=32)
def process_items(items_tuple):  # รับ tuple แทน list
    return sum(items_tuple)

result = process_items((1, 2, 3, 4, 5))
print(result)  # 15
```

---

## @dataclass เบื้องต้น

`@dataclass` จาก module `dataclasses` สร้าง class สำหรับเก็บข้อมูลโดยอัตโนมัติ ลด boilerplate code ได้มาก

### ตัวอย่าง 25: dataclass พื้นฐาน

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0  # ค่า default

@dataclass
class Student:
    name: str
    age: int
    grades: List[float] = field(default_factory=list)  # mutable default
    student_id: str = field(default="", repr=False)    # ไม่แสดงใน repr
    
    def average_grade(self):
        if not self.grades:
            return 0.0
        return sum(self.grades) / len(self.grades)
    
    def add_grade(self, grade):
        self.grades.append(grade)

# สร้าง instance
p1 = Point(1.0, 2.0)
p2 = Point(3.0, 4.0, 5.0)

print(p1)  # Point(x=1.0, y=2.0, z=0.0)
print(p2)  # Point(x=3.0, y=4.0, z=5.0)

# Equality comparison ถูกสร้างอัตโนมัติ
p3 = Point(1.0, 2.0)
print(p1 == p3)  # True

s = Student("สมชาย", 20)
s.add_grade(85.0)
s.add_grade(90.0)
s.add_grade(78.5)
print(s)
print(f"เกรดเฉลี่ย: {s.average_grade():.2f}")
```

### ตัวอย่าง 26: dataclass options

```python
from dataclasses import dataclass, field
import dataclasses

@dataclass(frozen=True)  # immutable - เหมือน namedtuple
class ImmutablePoint:
    x: float
    y: float
    
    def distance_to_origin(self):
        return (self.x**2 + self.y**2)**0.5

@dataclass(order=True)  # รองรับ <, >, <=, >=
class Temperature:
    value: float
    unit: str = field(default="C", compare=False)  # ไม่ใช้ใน comparison
    
    def to_celsius(self):
        if self.unit == "F":
            return (self.value - 32) * 5/9
        elif self.unit == "K":
            return self.value - 273.15
        return self.value

p = ImmutablePoint(3.0, 4.0)
print(p.distance_to_origin())  # 5.0

try:
    p.x = 10  # จะ raise FrozenInstanceError
except dataclasses.FrozenInstanceError as e:
    print(f"ไม่สามารถแก้ไข: {e}")

temps = [Temperature(100), Temperature(37), Temperature(0), Temperature(25)]
temps.sort()  # sort ได้เพราะ order=True
print([t.value for t in temps])  # [0, 25, 37, 100]
```

---

## Practical Decorators

### ตัวอย่าง 27: Timing Decorator

```python
import functools
import time
from contextlib import contextmanager

def timing(unit='ms'):
    """วัดเวลาการทำงาน
    
    Args:
        unit: หน่วยเวลา ('s', 'ms', 'μs', 'ns')
    """
    units = {'s': 1, 'ms': 1e3, 'μs': 1e6, 'ns': 1e9}
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter_ns()
            result = func(*args, **kwargs)
            elapsed_ns = time.perf_counter_ns() - start
            
            factor = units.get(unit, 1e3)
            elapsed = elapsed_ns / (1e9 / factor)
            
            print(f"{func.__name__} ใช้เวลา: {elapsed:.3f} {unit}")
            return result
        return wrapper
    return decorator

@timing('ms')
def sort_large_list():
    import random
    data = [random.random() for _ in range(100000)]
    return sorted(data)

@timing('μs')
def simple_calculation():
    return sum(range(1000))

sort_large_list()
simple_calculation()
```

### ตัวอย่าง 28: Logging Decorator สมบูรณ์

```python
import functools
import logging
import traceback
from datetime import datetime

def log_execution(logger=None, level=logging.INFO, log_args=True, log_result=True):
    """Decorator สำหรับ log การทำงานของฟังก์ชัน
    
    Args:
        logger: logging.Logger object (ถ้าไม่ระบุจะสร้างใหม่)
        level: logging level
        log_args: log arguments หรือไม่
        log_result: log return value หรือไม่
    """
    def decorator(func):
        nonlocal logger
        if logger is None:
            logger = logging.getLogger(func.__module__)
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            func_name = f"{func.__module__}.{func.__qualname__}"
            
            # Log เมื่อเริ่มทำงาน
            if log_args:
                args_repr = [repr(a) for a in args]
                kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
                signature = ", ".join(args_repr + kwargs_repr)
                logger.log(level, f"เรียก {func_name}({signature})")
            else:
                logger.log(level, f"เรียก {func_name}")
            
            start_time = datetime.now()
            
            try:
                result = func(*args, **kwargs)
                elapsed = (datetime.now() - start_time).total_seconds()
                
                if log_result:
                    logger.log(level, f"{func_name} สำเร็จใน {elapsed:.4f}s → {result!r}")
                else:
                    logger.log(level, f"{func_name} สำเร็จใน {elapsed:.4f}s")
                
                return result
                
            except Exception as e:
                elapsed = (datetime.now() - start_time).total_seconds()
                logger.error(
                    f"{func_name} ล้มเหลวหลังจาก {elapsed:.4f}s: "
                    f"{type(e).__name__}: {e}",
                    exc_info=True
                )
                raise
        
        return wrapper
    return decorator

# ตั้งค่า logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

@log_execution(level=logging.DEBUG)
def divide(a, b):
    return a / b

@log_execution(log_args=False)
def get_user_data(user_id):
    return {"id": user_id, "name": "สมชาย"}

result = divide(10, 2)
data = get_user_data(42)

try:
    divide(10, 0)
except ZeroDivisionError:
    pass
```

### ตัวอย่าง 29: Retry Decorator ขั้นสูง

```python
import functools
import time
import logging
from typing import Type, Tuple, Optional, Union

logger = logging.getLogger(__name__)

def retry_with_backoff(
    max_retries: int = 3,
    initial_delay: float = 1.0,
    backoff_factor: float = 2.0,
    max_delay: float = 60.0,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    on_retry: Optional[callable] = None
):
    """
    Retry decorator with exponential backoff
    
    Args:
        max_retries: จำนวนครั้งสูงสุดที่ retry
        initial_delay: เวลา delay เริ่มต้น (วินาที)
        backoff_factor: ตัวคูณเวลา delay ในแต่ละครั้ง
        max_delay: เวลา delay สูงสุด
        exceptions: ประเภท exception ที่จะ retry
        on_retry: callback function เมื่อ retry (รับ exception, attempt_number)
    """
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            delay = initial_delay
            
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_retries:
                        logger.error(f"หมดจำนวน retry สำหรับ {func.__name__}: {e}")
                        raise
                    
                    if on_retry:
                        on_retry(e, attempt + 1)
                    
                    actual_delay = min(delay, max_delay)
                    logger.warning(
                        f"Retry {attempt + 1}/{max_retries} สำหรับ {func.__name__}: "
                        f"{type(e).__name__}: {e}. รอ {actual_delay:.1f}s"
                    )
                    time.sleep(actual_delay)
                    delay *= backoff_factor
        
        wrapper.max_retries = max_retries
        return wrapper
    return decorator

# ตัวอย่างการใช้งาน
call_count = 0

@retry_with_backoff(
    max_retries=3,
    initial_delay=0.1,
    backoff_factor=2.0,
    exceptions=(ConnectionError, TimeoutError),
    on_retry=lambda e, n: print(f"กำลัง retry ครั้งที่ {n}...")
)
def connect_to_server():
    global call_count
    call_count += 1
    if call_count < 3:
        raise ConnectionError(f"Connection failed (attempt {call_count})")
    return "Connected!"

result = connect_to_server()
print(f"ผลลัพธ์: {result}")
```

### ตัวอย่าง 30: Input Validation Decorator

```python
import functools

def validate(**validators):
    """
    Decorator สำหรับ validate input parameters
    
    Usage:
        @validate(name=lambda x: len(x) > 0, age=lambda x: 0 <= x <= 150)
        def create_user(name, age):
            ...
    """
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            errors = []
            for param_name, validator in validators.items():
                if param_name in bound.arguments:
                    value = bound.arguments[param_name]
                    try:
                        if not validator(value):
                            errors.append(f"'{param_name}' ไม่ผ่าน validation (value={value!r})")
                    except Exception as e:
                        errors.append(f"'{param_name}' validator error: {e}")
            
            if errors:
                raise ValueError("Validation errors:\n" + "\n".join(f"  - {e}" for e in errors))
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate(
    name=lambda x: isinstance(x, str) and len(x.strip()) > 0,
    age=lambda x: isinstance(x, int) and 0 <= x <= 150,
    email=lambda x: '@' in x and '.' in x.split('@')[-1]
)
def register_user(name, age, email):
    return f"ลงทะเบียน: {name}, อายุ {age}, email: {email}"

# ถูกต้อง
print(register_user("สมชาย", 25, "somchai@example.com"))

# ผิด
try:
    register_user("", 25, "invalid-email")
except ValueError as e:
    print(f"Error:\n{e}")
```

### ตัวอย่าง 31: Authentication/Authorization Decorator

```python
import functools
from typing import List, Optional

# จำลอง user session
current_user = {"id": 1, "username": "admin", "roles": ["admin", "user"]}

class AuthError(Exception):
    pass

class PermissionError(Exception):
    pass

def require_auth(func):
    """ตรวจสอบว่า user ล็อกอินอยู่หรือไม่"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if not current_user:
            raise AuthError("กรุณาล็อกอินก่อน")
        return func(*args, **kwargs)
    return wrapper

def require_roles(*required_roles: str):
    """ตรวจสอบว่า user มี role ที่ต้องการหรือไม่"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if not current_user:
                raise AuthError("กรุณาล็อกอินก่อน")
            
            user_roles = current_user.get("roles", [])
            if not any(role in user_roles for role in required_roles):
                raise PermissionError(
                    f"ต้องการ role: {required_roles}, "
                    f"แต่มีเพียง: {user_roles}"
                )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_auth
@require_roles("admin")
def delete_user(user_id):
    return f"ลบ user ID {user_id} สำเร็จ"

@require_auth
@require_roles("admin", "moderator")
def edit_content(content_id, new_content):
    return f"แก้ไข content {content_id}: {new_content}"

@require_auth
def view_profile(user_id):
    return f"ดูโปรไฟล์ user {user_id}"

# ทดสอบ
try:
    result = delete_user(42)
    print(result)
except (AuthError, PermissionError) as e:
    print(f"Error: {e}")

# เปลี่ยน role เป็น user ธรรมดา
current_user["roles"] = ["user"]
try:
    delete_user(42)
except PermissionError as e:
    print(f"PermissionError: {e}")
```

---

## Decorator Patterns ในการทำงานจริง

### ตัวอย่าง 32: Rate Limiting Decorator

```python
import functools
import time
from collections import deque

def rate_limit(calls_per_second=1):
    """จำกัดจำนวน calls ต่อวินาที"""
    min_interval = 1.0 / calls_per_second
    last_called = [0.0]
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            wait_time = min_interval - elapsed
            
            if wait_time > 0:
                print(f"Rate limit: รอ {wait_time:.3f}s")
                time.sleep(wait_time)
            
            last_called[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls_per_second=2)  # สูงสุด 2 calls ต่อวินาที
def call_api(endpoint):
    return f"Response from {endpoint}"

start = time.time()
for i in range(4):
    result = call_api(f"/api/endpoint_{i}")
    print(f"  {result} ({time.time()-start:.2f}s)")
```

### ตัวอย่าง 33: Caching Decorator ขั้นสูง (Custom)

```python
import functools
import time
from typing import Any, Optional

def cache_with_ttl(ttl_seconds=60, maxsize=128):
    """Cache พร้อม Time-To-Live"""
    
    def decorator(func):
        cache = {}  # {args_key: (result, timestamp)}
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # สร้าง cache key
            key = (args, tuple(sorted(kwargs.items())))
            
            # ตรวจสอบ cache
            if key in cache:
                result, timestamp = cache[key]
                if time.time() - timestamp < ttl_seconds:
                    print(f"  [CACHE HIT] {func.__name__}")
                    return result
                else:
                    print(f"  [CACHE EXPIRED] {func.__name__}")
                    del cache[key]
            
            # คำนวณและเก็บ cache
            print(f"  [CACHE MISS] {func.__name__}")
            result = func(*args, **kwargs)
            
            # จำกัดขนาด cache
            if len(cache) >= maxsize:
                oldest_key = min(cache.keys(), key=lambda k: cache[k][1])
                del cache[oldest_key]
            
            cache[key] = (result, time.time())
            return result
        
        def cache_clear():
            cache.clear()
        
        def cache_info():
            return {
                'size': len(cache),
                'maxsize': maxsize,
                'ttl': ttl_seconds
            }
        
        wrapper.cache_clear = cache_clear
        wrapper.cache_info = cache_info
        return wrapper
    
    return decorator

@cache_with_ttl(ttl_seconds=5, maxsize=10)
def fetch_user_data(user_id):
    time.sleep(0.1)  # จำลอง API call
    return {"id": user_id, "name": f"User {user_id}"}

# ทดสอบ cache
data1 = fetch_user_data(1)  # CACHE MISS
data2 = fetch_user_data(1)  # CACHE HIT
data3 = fetch_user_data(2)  # CACHE MISS

print(fetch_user_data.cache_info())
```

### ตัวอย่าง 34: Singleton Decorator

```python
import functools
import threading

def singleton(cls):
    """ทำให้ class เป็น Singleton (สร้าง instance ได้แค่ตัวเดียว)"""
    instances = {}
    lock = threading.Lock()
    
    @functools.wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            with lock:  # thread-safe
                if cls not in instances:
                    instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    get_instance._instances = instances
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self, host="localhost", port=5432):
        print(f"เชื่อมต่อ database ที่ {host}:{port}")
        self.host = host
        self.port = port
    
    def query(self, sql):
        return f"Query result for: {sql}"

# จะสร้าง instance แค่ครั้งเดียว
db1 = DatabaseConnection("db.example.com", 5432)
db2 = DatabaseConnection("other-db.com", 3306)  # arguments จะถูกละเว้น

print(db1 is db2)       # True - instance เดียวกัน
print(db1.host)          # db.example.com
print(db2.host)          # db.example.com (เหมือนกัน)
```

### ตัวอย่าง 35: Deprecation Warning Decorator

```python
import functools
import warnings

def deprecated(reason="", version="", alternative=""):
    """Decorator สำหรับ mark function ว่า deprecated"""
    def decorator(func):
        message_parts = [f"{func.__name__} เลิกใช้แล้ว"]
        if version:
            message_parts.append(f"ตั้งแต่ version {version}")
        if reason:
            message_parts.append(f"เหตุผล: {reason}")
        if alternative:
            message_parts.append(f"ใช้ {alternative} แทน")
        
        message = ". ".join(message_parts) + "."
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            warnings.warn(message, DeprecationWarning, stacklevel=2)
            return func(*args, **kwargs)
        
        wrapper.__deprecated__ = True
        wrapper.__deprecated_message__ = message
        return wrapper
    return decorator

@deprecated(
    reason="ใช้ทรัพยากรมาก",
    version="2.0",
    alternative="process_data_v2()"
)
def process_data_v1(data):
    """วิธีเดิมในการประมวลผลข้อมูล"""
    return list(data)

def process_data_v2(data):
    """วิธีใหม่ที่ดีกว่า"""
    return [x for x in data if x is not None]

# จะแสดง warning
import warnings
with warnings.catch_warnings(record=True) as w:
    warnings.simplefilter("always")
    result = process_data_v1([1, 2, 3])
    if w:
        print(f"Warning: {w[0].message}")
```

---

## แบบฝึกหัด

### ข้อ 1: สร้าง `@memoize` decorator

สร้าง decorator ที่ cache ผลลัพธ์ของฟังก์ชัน (ง่ายกว่า lru_cache)

**คำตอบ:**

```python
import functools

def memoize(func):
    """Simple memoization decorator"""
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    wrapper.cache = cache
    wrapper.cache_clear = lambda: cache.clear()
    return wrapper

@memoize
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(10))   # 3628800
print(factorial(5))    # 120 (จาก cache)
print(factorial.cache) # {(1,): 1, (2,): 2, ...}
```

### ข้อ 2: สร้าง `@type_check` decorator

สร้าง decorator ที่ตรวจสอบประเภทของ arguments และ return value โดยใช้ type hints

**คำตอบ:**

```python
import functools
import inspect
import typing

def type_check(func):
    """Decorator ที่ validate types ตาม type hints"""
    hints = typing.get_type_hints(func)
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        
        # ตรวจสอบ argument types
        for param_name, value in bound.arguments.items():
            if param_name in hints:
                expected = hints[param_name]
                if not isinstance(value, expected):
                    raise TypeError(
                        f"Argument '{param_name}': "
                        f"คาดหวัง {expected.__name__}, "
                        f"ได้รับ {type(value).__name__}"
                    )
        
        result = func(*args, **kwargs)
        
        # ตรวจสอบ return type
        if 'return' in hints:
            expected_return = hints['return']
            if expected_return is not type(None) and not isinstance(result, expected_return):
                raise TypeError(
                    f"Return type: คาดหวัง {expected_return.__name__}, "
                    f"ได้รับ {type(result).__name__}"
                )
        
        return result
    return wrapper

@type_check
def greet(name: str, times: int) -> str:
    return f"สวัสดี {name}! " * times

print(greet("สมชาย", 2))

try:
    greet("สมชาย", "สอง")  # TypeError
except TypeError as e:
    print(f"Error: {e}")
```

### ข้อ 3: สร้าง `@trace` decorator

สร้าง decorator ที่แสดง call stack และ return values เหมาะสำหรับ debug recursive functions

**คำตอบ:**

```python
import functools

def trace(func):
    """Decorator สำหรับ trace recursive functions"""
    func._depth = 0
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        indent = "  " * func._depth
        args_str = ", ".join(repr(a) for a in args)
        print(f"{indent}→ {func.__name__}({args_str})")
        
        func._depth += 1
        result = func(*args, **kwargs)
        func._depth -= 1
        
        print(f"{indent}← {func.__name__}({args_str}) = {result!r}")
        return result
    return wrapper

@trace
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)

fib(4)
```

### ข้อ 4: สร้าง `@count_calls` decorator

สร้าง decorator ที่นับจำนวนครั้งที่ฟังก์ชันถูกเรียก

**คำตอบ:**

```python
import functools

def count_calls(func):
    """นับจำนวนครั้งที่ฟังก์ชันถูกเรียก"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        return func(*args, **kwargs)
    
    wrapper.calls = 0
    wrapper.reset = lambda: setattr(wrapper, 'calls', 0)
    return wrapper

@count_calls
def api_request(url):
    return f"Response from {url}"

for i in range(5):
    api_request(f"https://api.example.com/{i}")

print(f"เรียก API ทั้งหมด {api_request.calls} ครั้ง")
api_request.reset()
print(f"หลัง reset: {api_request.calls} ครั้ง")
```

### ข้อ 5: สร้าง `@enforce_immutability` decorator

สร้าง decorator สำหรับ class method ที่ห้ามแก้ไข attributes บางตัว

**คำตอบ:**

```python
import functools

def read_only_attrs(*attr_names):
    """Decorator สำหรับ class ที่ทำให้ attributes บางตัวเป็น read-only"""
    def decorator(cls):
        original_setattr = cls.__setattr__ if hasattr(cls, '__setattr__') else object.__setattr__
        
        @functools.wraps(cls.__init__ if hasattr(cls, '__init__') else lambda: None)
        def new_setattr(self, name, value):
            if hasattr(self, '_initialized') and name in attr_names:
                raise AttributeError(f"Cannot modify read-only attribute: {name}")
            original_setattr(self, name, value)
        
        original_init = cls.__init__
        
        def new_init(self, *args, **kwargs):
            original_init(self, *args, **kwargs)
            object.__setattr__(self, '_initialized', True)
        
        cls.__setattr__ = new_setattr
        cls.__init__ = new_init
        return cls
    return decorator

@read_only_attrs('id', 'created_at')
class User:
    def __init__(self, user_id, name):
        self.id = user_id
        self.name = name
        import datetime
        self.created_at = datetime.datetime.now()

user = User(1, "สมชาย")
user.name = "สมหญิง"  # OK - name ไม่ใช่ read-only
print(user.name)

try:
    user.id = 99  # Error! id เป็น read-only
except AttributeError as e:
    print(f"Error: {e}")
```

### ข้อ 6: สร้าง `@circuit_breaker` decorator

Circuit Breaker pattern - หยุด call service เมื่อ error มากเกินไป

**คำตอบ:**

```python
import functools
import time

def circuit_breaker(failure_threshold=3, reset_timeout=10):
    """
    Circuit Breaker Pattern
    - CLOSED: ทำงานปกติ
    - OPEN: หยุดรับ request เมื่อ failure มากเกินไป
    - HALF-OPEN: ทดสอบว่า service กลับมาแล้วหรือยัง
    """
    def decorator(func):
        state = {'status': 'CLOSED', 'failures': 0, 'last_failure': None}
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            
            if state['status'] == 'OPEN':
                if now - state['last_failure'] > reset_timeout:
                    state['status'] = 'HALF-OPEN'
                    print(f"Circuit HALF-OPEN: ทดสอบ service...")
                else:
                    raise Exception(f"Circuit OPEN: service ไม่พร้อม")
            
            try:
                result = func(*args, **kwargs)
                if state['status'] == 'HALF-OPEN':
                    state['status'] = 'CLOSED'
                    state['failures'] = 0
                    print("Circuit CLOSED: service กลับมาแล้ว")
                return result
            except Exception as e:
                state['failures'] += 1
                state['last_failure'] = now
                
                if state['failures'] >= failure_threshold:
                    state['status'] = 'OPEN'
                    print(f"Circuit OPEN: failures={state['failures']}")
                raise
        
        wrapper.state = state
        return wrapper
    return decorator

fail_count = 0

@circuit_breaker(failure_threshold=3, reset_timeout=1)
def call_external_service():
    global fail_count
    fail_count += 1
    if fail_count <= 3:
        raise ConnectionError("Service unavailable")
    return "Service response"

for i in range(5):
    try:
        result = call_external_service()
        print(f"สำเร็จ: {result}")
    except Exception as e:
        print(f"Error: {e}")
```

### ข้อ 7-10: แบบฝึกหัดเพิ่มเติม (ให้นักเรียนทำเอง)

**ข้อ 7**: สร้าง `@timeout(seconds)` decorator ที่จะ raise `TimeoutError` ถ้าฟังก์ชันทำงานนานเกินกำหนด

**ข้อ 8**: สร้าง `@run_async` decorator ที่ทำให้ฟังก์ชัน synchronous ทำงานใน thread ใหม่

**ข้อ 9**: สร้าง `@cache_to_file(filepath)` decorator ที่ save cache ลงไฟล์ และโหลดกลับมาเมื่อ restart

**ข้อ 10**: สร้าง `@event_hook` decorator system ที่ allow ผู้ใช้ subscribe ไปยัง before/after events ของฟังก์ชัน

---

## สรุป

| Concept | ใช้เมื่อไหร่ |
|---------|-------------|
| Simple decorator | เพิ่ม behavior ง่ายๆ ให้ฟังก์ชัน |
| Decorator with args | ต้องการ parameterize behavior |
| Class decorator | ต้องการ maintain state ระหว่าง calls |
| Stacking | รวม behaviors หลายอย่าง |
| @property | controlled attribute access |
| @classmethod | alternative constructors, factory methods |
| @staticmethod | utility functions ที่ไม่ต้องการ instance |
| @lru_cache | cache expensive computations |
| @dataclass | data containers ที่ไม่ต้องการ boilerplate |

Decorators เป็น pattern ที่ทรงพลังมากสำหรับ:
- **Separation of Concerns**: แยก business logic ออกจาก cross-cutting concerns
- **DRY Principle**: ไม่เขียนโค้ดซ้ำ
- **Open/Closed Principle**: เพิ่ม behavior โดยไม่แก้ไขโค้ดเดิม
