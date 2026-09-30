# Part 23 - OOP: Polymorphism & Duck Typing

## สารบัญ
1. [Polymorphism คืออะไร](#polymorphism-คืออะไร)
2. [Method Overriding as Polymorphism](#method-overriding-as-polymorphism)
3. [Duck Typing ใน Python](#duck-typing-ใน-python)
4. [Operator Overloading (Dunder Methods)](#operator-overloading-dunder-methods)
5. [__add__, __sub__, __mul__, __truediv__](#arithmetic-operators)
6. [__eq__, __lt__, __gt__](#comparison-operators)
7. [__len__, __getitem__, __setitem__, __contains__](#container-operators)
8. [__call__ Method](#__call__-method)
9. [__iter__ และ __next__](#__iter__-และ-__next__)
10. [Protocol-based Polymorphism](#protocol-based-polymorphism)
11. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Polymorphism คืออะไร

**Polymorphism** (พหุสัณฐาน) มาจากภาษากรีก: "poly" = หลาย, "morph" = รูปร่าง หมายถึง ความสามารถที่วัตถุต่างชนิดกันสามารถตอบสนองต่อ interface เดียวกันได้ โดยมี behavior ที่แตกต่างกัน

### ทำไม Polymorphism สำคัญ?

```python
# ===== ไม่มี Polymorphism =====
# ต้องเช็ค type ทุกที่ - โค้ดยุ่งเหยิง

def calculate_area_BAD(shape):
    if type(shape).__name__ == "Circle":
        import math
        return math.pi * shape.radius ** 2
    elif type(shape).__name__ == "Rectangle":
        return shape.width * shape.height
    elif type(shape).__name__ == "Triangle":
        return 0.5 * shape.base * shape.height
    else:
        raise ValueError(f"ไม่รู้จัก shape type: {type(shape).__name__}")


# ===== มี Polymorphism =====
# แต่ละ shape รับผิดชอบ area() ของตัวเอง - โค้ดสะอาด

def calculate_area_GOOD(shape):
    return shape.area()  # ไม่สนใจว่า shape เป็น class อะไร

# เพิ่ม shape ใหม่ไม่ต้องแก้ calculate_area_GOOD เลย!
```

### รูปแบบของ Polymorphism ใน Python

```python
# 1. Subtype Polymorphism (Inheritance-based)
# 2. Duck Typing
# 3. Operator Overloading
# 4. Protocol-based (typing.Protocol)
```

---

## Method Overriding as Polymorphism

รูปแบบพื้นฐานของ Polymorphism - subclasses override methods ของ parent

### ตัวอย่างที่ 1: Polymorphism ผ่าน Method Overriding

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        """Base method - subclasses จะ override"""
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return f"{self.name}: โฮ่ง!"

class Cat(Animal):
    def speak(self):
        return f"{self.name}: เมี๊ยว!"

class Duck(Animal):
    def speak(self):
        return f"{self.name}: แก้ก!"

class Cow(Animal):
    def speak(self):
        return f"{self.name}: มู้ว!"

# Polymorphism ในงาน: function เดียวทำงานได้กับทุก type
def make_noise(animal):
    print(animal.speak())  # ไม่สนใจว่า animal เป็น class อะไร

# ใช้กับ list ของสัตว์หลายชนิด
animals = [
    Dog("บักโกง"),
    Cat("มะหมา"),
    Duck("โดนัลด์"),
    Cow("แม่วัว"),
    Dog("ไฟ"),
]

print("เสียงสัตว์ต่างๆ:")
for animal in animals:
    make_noise(animal)  # แต่ละตัวส่งเสียงต่างกัน!

# ไม่สนใจ type ก็หาผลรวมได้
```

### ตัวอย่างที่ 2: Polymorphism กับ Functions

```python
class Renderer:
    """Base renderer"""
    def render(self, content):
        raise NotImplementedError

class HTMLRenderer(Renderer):
    def render(self, content):
        return f"<p>{content}</p>"

class MarkdownRenderer(Renderer):
    def render(self, content):
        return f"**{content}**"

class PlainTextRenderer(Renderer):
    def render(self, content):
        return content

class JSONRenderer(Renderer):
    def render(self, content):
        import json
        return json.dumps({"text": content})

# Function ที่รับ renderer ใดๆ ก็ได้
def publish_content(content_list, renderer):
    """เผยแพร่เนื้อหาผ่าน renderer ที่กำหนด"""
    for item in content_list:
        print(renderer.render(item))

content = ["สวัสดี Python", "OOP เป็นเรื่องสนุก", "Polymorphism ดีมาก"]

print("=== HTML ===")
publish_content(content, HTMLRenderer())

print("\n=== Markdown ===")
publish_content(content, MarkdownRenderer())

print("\n=== JSON ===")
publish_content(content, JSONRenderer())
```

---

## Duck Typing ใน Python

> "If it walks like a duck and quacks like a duck, it must be a duck."

**Duck Typing** คือ Python ไม่สนใจว่า object เป็น class อะไร แต่สนใจว่า object นั้นมี method/attribute ที่ต้องการหรือไม่

### ตัวอย่างที่ 3: Duck Typing ในแบบ Python

```python
# Python ไม่ต้องการ inheritance เพื่อ polymorphism

class Duck:
    def quack(self):
        return "เป็ดจริง: แก้ก!"
    
    def walk(self):
        return "เป็ดเดิน..."

class Person:
    def quack(self):
        return "คนเลียนเสียงเป็ด: แก้ก!"
    
    def walk(self):
        return "คนเดิน..."

class RobotDuck:
    def quack(self):
        return "หุ่นยนต์เป็ด: BEEP QUACK BEEP"
    
    def walk(self):
        return "หุ่นยนต์เดิน..."

class Cat:
    def meow(self):
        return "แมว: เมี๊ยว!"
    # ไม่มี quack และ walk


def duck_test(thing):
    """ทดสอบว่าสิ่งนี้เป็น 'เป็ด' หรือไม่"""
    try:
        print(thing.quack())
        print(thing.walk())
        print(f"  --> {type(thing).__name__} ผ่านการทดสอบเป็ด!\n")
    except AttributeError as e:
        print(f"  --> {type(thing).__name__} ไม่ใช่เป็ด: {e}\n")


# ทดสอบ - ไม่สนใจ type
duck_test(Duck())
duck_test(Person())
duck_test(RobotDuck())
duck_test(Cat())
```

### ตัวอย่างที่ 4: Duck Typing กับ Built-in Functions

```python
# Python built-in functions ใช้ duck typing ตลอด

class CustomIterable:
    """ทำให้ class iterate ได้ - ไม่ต้อง inherit จาก list"""
    
    def __init__(self, data):
        self._data = data
        self._index = 0
    
    def __iter__(self):  # รองรับ for loop
        return self
    
    def __next__(self):  # รองรับ next()
        if self._index >= len(self._data):
            raise StopIteration
        value = self._data[self._index]
        self._index += 1
        return value


class CustomSizable:
    """ทำให้ class มี len() ได้"""
    
    def __init__(self, items):
        self._items = items
    
    def __len__(self):  # รองรับ len()
        return len(self._items)


class CustomStringifiable:
    """ทำให้ class แปลงเป็น string ได้"""
    
    def __init__(self, value):
        self.value = value
    
    def __str__(self):
        return f"Custom: {self.value}"
    
    def __format__(self, spec):
        if spec == 'upper':
            return str(self.value).upper()
        return str(self.value)


# ทดสอบ
ci = CustomIterable([1, 2, 3, 4, 5])
for item in ci:  # ใช้ __iter__ และ __next__
    print(item, end=" ")
print()

cs = CustomSizable(["a", "b", "c", "d"])
print(f"ขนาด: {len(cs)}")  # ใช้ __len__

cstr = CustomStringifiable("hello")
print(cstr)               # ใช้ __str__
print(f"{cstr:upper}")    # ใช้ __format__
print(f"{cstr}")          # ใช้ __format__ ปกติ
```

### ตัวอย่างที่ 5: EAFP vs LBYL

```python
# Python style: EAFP = Easier to Ask Forgiveness than Permission
# (ลองก่อน แล้วค่อยจัดการ exception)

# ===== LBYL style (Look Before You Leap) =====
# ตรวจสอบก่อน - ไม่ใช่ Pythonic

def process_LBYL(obj):
    if hasattr(obj, 'process') and callable(getattr(obj, 'process')):
        return obj.process()
    else:
        return None


# ===== EAFP style =====
# ลองทำ แล้วค่อยดักข้อผิดพลาด - Pythonic

def process_EAFP(obj):
    try:
        return obj.process()
    except AttributeError:
        return None


class Processor:
    def process(self):
        return "ประมวลผลแล้ว"

class NotProcessor:
    pass

p = Processor()
n = NotProcessor()

print(process_EAFP(p))  # ประมวลผลแล้ว
print(process_EAFP(n))  # None
```

---

## Operator Overloading (Dunder Methods)

Python ให้เรากำหนดพฤติกรรมของ operators (+, -, *, /, ==, <, >, len, [], in, etc.) สำหรับ custom classes ผ่าน **dunder methods** (double underscore methods)

---

## Arithmetic Operators

### ตัวอย่างที่ 6: __add__, __sub__, __mul__, __truediv__

```python
class Vector2D:
    """Vector 2 มิติ พร้อม arithmetic operators"""
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # ===== Arithmetic =====
    
    def __add__(self, other):
        """v1 + v2"""
        if isinstance(other, Vector2D):
            return Vector2D(self.x + other.x, self.y + other.y)
        elif isinstance(other, (int, float)):
            return Vector2D(self.x + other, self.y + other)
        return NotImplemented
    
    def __radd__(self, other):
        """other + v (reverse add)"""
        # รองรับ: 5 + vector
        return self.__add__(other)
    
    def __sub__(self, other):
        """v1 - v2"""
        if isinstance(other, Vector2D):
            return Vector2D(self.x - other.x, self.y - other.y)
        return NotImplemented
    
    def __mul__(self, scalar):
        """v * scalar"""
        if isinstance(scalar, (int, float)):
            return Vector2D(self.x * scalar, self.y * scalar)
        elif isinstance(scalar, Vector2D):
            # Dot product
            return self.x * scalar.x + self.y * scalar.y
        return NotImplemented
    
    def __rmul__(self, scalar):
        """scalar * v (reverse mul)"""
        return self.__mul__(scalar)
    
    def __truediv__(self, scalar):
        """v / scalar"""
        if scalar == 0:
            raise ZeroDivisionError("หารด้วย 0 ไม่ได้")
        return Vector2D(self.x / scalar, self.y / scalar)
    
    def __floordiv__(self, scalar):
        """v // scalar"""
        if scalar == 0:
            raise ZeroDivisionError("หารด้วย 0 ไม่ได้")
        return Vector2D(self.x // scalar, self.y // scalar)
    
    def __neg__(self):
        """-v"""
        return Vector2D(-self.x, -self.y)
    
    def __abs__(self):
        """abs(v) = magnitude"""
        return (self.x**2 + self.y**2) ** 0.5
    
    # ===== In-place operators =====
    
    def __iadd__(self, other):
        """v += other"""
        if isinstance(other, Vector2D):
            self.x += other.x
            self.y += other.y
            return self
        return NotImplemented
    
    def __isub__(self, other):
        """v -= other"""
        if isinstance(other, Vector2D):
            self.x -= other.x
            self.y -= other.y
            return self
        return NotImplemented
    
    def __str__(self):
        return f"({self.x}, {self.y})"
    
    def __repr__(self):
        return f"Vector2D({self.x}, {self.y})"


# ทดสอบ
v1 = Vector2D(1, 2)
v2 = Vector2D(3, 4)

print(f"v1 = {v1}")
print(f"v2 = {v2}")
print(f"v1 + v2 = {v1 + v2}")    # (4, 6)
print(f"v1 - v2 = {v1 - v2}")    # (-2, -2)
print(f"v1 * 3 = {v1 * 3}")      # (3, 6)
print(f"3 * v1 = {3 * v1}")      # (3, 6) - reverse mul
print(f"v2 / 2 = {v2 / 2}")      # (1.5, 2.0)
print(f"-v1 = {-v1}")            # (-1, -2)
print(f"|v2| = {abs(v2):.4f}")   # 5.0000
print(f"v1·v2 = {v1 * v2}")     # dot product = 11

# In-place
v3 = Vector2D(1, 1)
v3 += v1
print(f"v3 += v1: {v3}")  # (2, 3)
```

---

## Comparison Operators

### ตัวอย่างที่ 7: __eq__, __lt__, __gt__, __le__, __ge__, __ne__

```python
from functools import total_ordering

@total_ordering  # สร้าง comparison methods ที่เหลือจาก __eq__ และ __lt__
class Temperature:
    """อุณหภูมิ - ใช้เปรียบเทียบได้"""
    
    def __init__(self, celsius):
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    
    def __eq__(self, other):
        if isinstance(other, Temperature):
            return abs(self._celsius - other._celsius) < 0.001
        elif isinstance(other, (int, float)):
            return abs(self._celsius - other) < 0.001
        return NotImplemented
    
    def __lt__(self, other):
        if isinstance(other, Temperature):
            return self._celsius < other._celsius
        elif isinstance(other, (int, float)):
            return self._celsius < other
        return NotImplemented
    
    def __hash__(self):
        return hash(round(self._celsius, 3))
    
    def __str__(self):
        return f"{self._celsius}°C"
    
    def __repr__(self):
        return f"Temperature({self._celsius})"


# ทดสอบ
t1 = Temperature(100)
t2 = Temperature(0)
t3 = Temperature(37)
t4 = Temperature(100)

print(f"t1 ({t1}) == t4 ({t4}): {t1 == t4}")  # True
print(f"t1 ({t1}) == t2 ({t2}): {t1 == t2}")  # False
print(f"t1 ({t1}) > t3 ({t3}): {t1 > t3}")    # True
print(f"t2 ({t2}) < t3 ({t3}): {t2 < t3}")    # True
print(f"t1 ({t1}) >= t4 ({t4}): {t1 >= t4}")  # True

# ใช้ใน sorted()
temps = [Temperature(100), Temperature(37), Temperature(0), Temperature(-10)]
print("\nเรียงจากเย็นไปร้อน:")
for t in sorted(temps):
    print(f"  {t}")

# ใช้ใน min/max
print(f"\nต่ำสุด: {min(temps)}")
print(f"สูงสุด: {max(temps)}")

# ใช้ใน set (ต้องมี __hash__)
temp_set = {Temperature(100), Temperature(100), Temperature(37)}
print(f"\nSet: {temp_set}")  # ลบ duplicate
```

---

## Container Operators

### ตัวอย่างที่ 8: __len__, __getitem__, __setitem__, __contains__

```python
class NumberList:
    """Custom list ที่ทำงานเหมือน list แต่เก็บเฉพาะตัวเลข"""
    
    def __init__(self, *args):
        self._data = []
        for item in args:
            self.append(item)
    
    def append(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"ต้องเป็นตัวเลข ไม่ใช่ {type(value).__name__}")
        self._data.append(value)
    
    def __len__(self):
        """len(obj) -> int"""
        return len(self._data)
    
    def __getitem__(self, index):
        """obj[index] หรือ obj[start:stop]"""
        if isinstance(index, slice):
            return NumberList(*self._data[index])
        if not isinstance(index, int):
            raise TypeError("index ต้องเป็น int หรือ slice")
        return self._data[index]
    
    def __setitem__(self, index, value):
        """obj[index] = value"""
        if not isinstance(value, (int, float)):
            raise TypeError(f"ต้องเป็นตัวเลข")
        self._data[index] = value
    
    def __delitem__(self, index):
        """del obj[index]"""
        del self._data[index]
    
    def __contains__(self, value):
        """value in obj"""
        return value in self._data
    
    def __iter__(self):
        """for item in obj"""
        return iter(self._data)
    
    def __reversed__(self):
        """reversed(obj)"""
        return reversed(self._data)
    
    def __add__(self, other):
        """obj + other"""
        if isinstance(other, NumberList):
            return NumberList(*self._data, *other._data)
        return NotImplemented
    
    # Statistical methods
    def sum(self):
        return sum(self._data)
    
    def mean(self):
        if not self._data:
            raise ValueError("List ว่างเปล่า")
        return self.sum() / len(self)
    
    def max(self):
        return max(self._data)
    
    def min(self):
        return min(self._data)
    
    def __str__(self):
        return f"NumberList({self._data})"
    
    def __repr__(self):
        return f"NumberList({', '.join(str(x) for x in self._data)})"


# ทดสอบ
nl = NumberList(1, 2, 3, 4, 5)

print(f"ขนาด: {len(nl)}")         # 5
print(f"element แรก: {nl[0]}")    # 1
print(f"element สุดท้าย: {nl[-1]}") # 5
print(f"slice [1:3]: {nl[1:3]}") # NumberList([2, 3])

nl[0] = 10
print(f"หลัง set nl[0]=10: {nl}")

print(f"3 ใน nl: {3 in nl}")      # True
print(f"99 ใน nl: {99 in nl}")    # False

print(f"sum: {nl.sum()}")
print(f"mean: {nl.mean()}")

# For loop
print("Items:", end=" ")
for item in nl:
    print(item, end=" ")
print()

# Reversed
print("Reversed:", end=" ")
for item in reversed(nl):
    print(item, end=" ")
print()

# ลอง error
try:
    nl.append("hello")
except TypeError as e:
    print(f"Error: {e}")
```

### ตัวอย่างที่ 9: __getattr__, __setattr__, __delattr__

```python
class FlexObject:
    """Object ที่รับ attributes ใดๆ ก็ได้"""
    
    def __init__(self, **kwargs):
        self._data = kwargs
    
    def __getattr__(self, name):
        """เรียกเมื่อ attribute ไม่พบใน normal lookup"""
        if name.startswith('_'):
            raise AttributeError(name)
        return self._data.get(name)
    
    def __setattr__(self, name, value):
        """เรียกทุกครั้งที่ set attribute"""
        if name.startswith('_'):
            # attributes ที่ขึ้นต้นด้วย _ ใช้ normal way
            super().__setattr__(name, value)
        else:
            # attributes อื่นๆ เก็บใน _data
            self._data[name] = value
    
    def __delattr__(self, name):
        """del obj.attr"""
        if name in self._data:
            del self._data[name]
        else:
            raise AttributeError(f"ไม่มี attribute '{name}'")
    
    def __contains__(self, name):
        return name in self._data
    
    def keys(self):
        return self._data.keys()
    
    def __repr__(self):
        items = ', '.join(f"{k}={v!r}" for k, v in self._data.items())
        return f"FlexObject({items})"


# ทดสอบ
obj = FlexObject(name="สมชาย", age=30)
print(obj.name)    # สมชาย
print(obj.age)     # 30

obj.email = "somchai@example.com"  # เพิ่ม attribute ใหม่
print(obj.email)   # somchai@example.com

print('name' in obj)   # True
print('phone' in obj)  # False

del obj.age
print(obj.age)  # None (ไม่มีแล้ว)

print(obj)
```

---

## __call__ Method

`__call__` ทำให้ object สามารถ "เรียก" เหมือนฟังก์ชันได้

### ตัวอย่างที่ 10: __call__ Method

```python
class Multiplier:
    """Object ที่เรียกได้เหมือนฟังก์ชัน"""
    
    def __init__(self, factor):
        self.factor = factor
        self.call_count = 0
    
    def __call__(self, value):
        """เรียกได้เหมือน function: multiplier(10)"""
        self.call_count += 1
        return value * self.factor
    
    def __repr__(self):
        return f"Multiplier(factor={self.factor}, calls={self.call_count})"


# สร้าง callable objects
double = Multiplier(2)
triple = Multiplier(3)

# เรียกเหมือนฟังก์ชัน
print(double(5))   # 10
print(triple(5))   # 15
print(double(10))  # 20

print(double)  # Multiplier(factor=2, calls=2)

# ตรวจสอบว่า callable
print(callable(double))     # True
print(callable(lambda x: x)) # True
print(callable(42))         # False
```

### ตัวอย่างที่ 11: __call__ สำหรับ Decorator Class

```python
import functools
import time

class Timer:
    """Decorator class ที่วัดเวลาการทำงานของฟังก์ชัน"""
    
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.total_time = 0
        self.call_count = 0
    
    def __call__(self, *args, **kwargs):
        start = time.time()
        result = self.func(*args, **kwargs)
        elapsed = time.time() - start
        
        self.total_time += elapsed
        self.call_count += 1
        
        print(f"{self.func.__name__}() ใช้เวลา: {elapsed:.4f}s "
              f"(เรียกแล้ว {self.call_count} ครั้ง)")
        return result
    
    @property
    def avg_time(self):
        return self.total_time / self.call_count if self.call_count > 0 else 0


@Timer
def slow_function(n):
    """จำลองฟังก์ชันที่ใช้เวลา"""
    total = 0
    for i in range(n):
        total += i ** 2
    return total

result1 = slow_function(100000)
result2 = slow_function(200000)
result3 = slow_function(50000)

print(f"\nสถิติ: เรียก {slow_function.call_count} ครั้ง")
print(f"เวลาเฉลี่ย: {slow_function.avg_time:.4f}s")
```

### ตัวอย่างที่ 12: __call__ สำหรับ Memoization

```python
class Memoize:
    """Cache ผลลัพธ์ของ function"""
    
    def __init__(self, func):
        self.func = func
        self.cache = {}
        functools.update_wrapper(self, func)
    
    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]
    
    def cache_info(self):
        return f"cache size: {len(self.cache)}"


@Memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

import functools

# ทดสอบ - ครั้งแรกคำนวณ ครั้งต่อไปดึงจาก cache
print(fibonacci(10))   # 55
print(fibonacci(20))   # 6765
print(fibonacci(30))   # 832040
print(fibonacci.cache_info())
```

---

## __iter__ และ __next__

ทำให้ object สามารถ iterate ได้ (ใช้ใน for loop)

### ตัวอย่างที่ 13: Iterable Object

```python
class CountDown:
    """นับถอยหลัง - iterable object"""
    
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        """เรียกเมื่อ for loop เริ่มต้น
        ควร return iterator object (ในที่นี้ return ตัวเอง)
        """
        self.current = self.start
        return self
    
    def __next__(self):
        """เรียกทุก iteration"""
        if self.current < 0:
            raise StopIteration  # บอกว่าหมดแล้ว
        value = self.current
        self.current -= 1
        return value


# ทดสอบ
print("นับถอยหลัง:")
for num in CountDown(5):
    print(num, end=" ")
print()

# ใช้ list(), tuple(), sum() กับ iterable ได้
print(list(CountDown(10)))
print(sum(CountDown(100)))  # 0+1+2+...+100 = 5050

# ใช้ next() ด้วยตัวเอง
cd = CountDown(3)
it = iter(cd)   # เรียก __iter__
print(next(it)) # 3  - เรียก __next__
print(next(it)) # 2
print(next(it)) # 1
print(next(it)) # 0
try:
    print(next(it))  # StopIteration
except StopIteration:
    print("หมดแล้ว!")
```

### ตัวอย่างที่ 14: Iterator vs Iterable

```python
class NumberRange:
    """Iterable - มี __iter__ แต่ไม่มี __next__"""
    
    def __init__(self, start, stop, step=1):
        self.start = start
        self.stop = stop
        self.step = step
    
    def __iter__(self):
        """Return iterator ใหม่ทุกครั้ง"""
        return NumberRangeIterator(self.start, self.stop, self.step)
    
    def __len__(self):
        return max(0, (self.stop - self.start + self.step - 1) // self.step)


class NumberRangeIterator:
    """Iterator - มีทั้ง __iter__ และ __next__"""
    
    def __init__(self, start, stop, step):
        self.current = start
        self.stop = stop
        self.step = step
    
    def __iter__(self):
        """Iterator ควร return ตัวเอง"""
        return self
    
    def __next__(self):
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += self.step
        return value


# Iterable สามารถ iterate ได้หลายครั้ง
r = NumberRange(0, 10, 2)
print("ครั้งที่ 1:", list(r))
print("ครั้งที่ 2:", list(r))  # ได้ผลเหมือนกัน!

# Iterator iterate ได้ครั้งเดียว
it = iter(r)
print("Iterator ครั้งที่ 1:", list(it))
print("Iterator ครั้งที่ 2:", list(it))  # ว่างเปล่า! exhausted

print(f"ขนาด range: {len(r)}")
```

### ตัวอย่างที่ 15: Infinite Iterator

```python
class FibonacciIterator:
    """Iterator ที่สร้าง Fibonacci sequence ไม่รู้จบ"""
    
    def __init__(self):
        self.a = 0
        self.b = 1
    
    def __iter__(self):
        return self
    
    def __next__(self):
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        return value


# ใช้ itertools.islice เพื่อ limit infinite iterator
from itertools import islice

fib = FibonacciIterator()
print("Fibonacci 10 ตัวแรก:", list(islice(fib, 10)))

# สร้างใหม่
fib2 = FibonacciIterator()
first_100 = list(islice(fib2, 100))
print(f"Fibonacci ตัวที่ 100: {first_100[-1]}")
print(f"ผลรวม 10 ตัวแรก: {sum(list(islice(FibonacciIterator(), 10)))}")
```

---

## Protocol-based Polymorphism

Python 3.8+ มี `typing.Protocol` ที่ให้กำหนด interface แบบ structural typing

### ตัวอย่างที่ 16: typing.Protocol

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    """Protocol: สิ่งที่ draw ได้"""
    
    def draw(self) -> str:
        ...

@runtime_checkable
class Resizable(Protocol):
    """Protocol: สิ่งที่ resize ได้"""
    
    def resize(self, factor: float) -> None:
        ...


# Classes ไม่ต้อง inherit จาก Protocol
class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    def draw(self) -> str:
        return f"วาดวงกลม radius={self.radius}"
    
    def resize(self, factor: float) -> None:
        self.radius *= factor


class Square:
    def __init__(self, side):
        self.side = side
    
    def draw(self) -> str:
        return f"วาดสี่เหลี่ยม side={self.side}"
    
    def resize(self, factor: float) -> None:
        self.side *= factor


class Text:
    def __init__(self, content):
        self.content = content
    
    def draw(self) -> str:
        return f"วาดข้อความ: {self.content}"
    # ไม่มี resize!


def render(drawable: Drawable):
    """รับ Drawable ใดๆ"""
    print(drawable.draw())


def scale_up(resizable: Resizable, factor=2):
    """รับ Resizable ใดๆ"""
    resizable.resize(factor)


# ทดสอบ - ไม่ต้อง inherit จาก Drawable
shapes = [Circle(5), Square(4), Text("Hello")]

for shape in shapes:
    render(shape)  # ทุก shape draw ได้

print()
for shape in [Circle(5), Square(4)]:
    scale_up(shape, 1.5)
    render(shape)

# Runtime check ด้วย isinstance (ต้องใช้ @runtime_checkable)
c = Circle(3)
print(f"\nCircle เป็น Drawable: {isinstance(c, Drawable)}")
print(f"Circle เป็น Resizable: {isinstance(c, Resizable)}")

t = Text("Hello")
print(f"Text เป็น Drawable: {isinstance(t, Drawable)}")
print(f"Text เป็น Resizable: {isinstance(t, Resizable)}")
```

### ตัวอย่างที่ 17: Protocol กับ Type Hints

```python
from typing import Protocol, List, Optional

class Sortable(Protocol):
    """Protocol สำหรับ object ที่เรียงได้"""
    def __lt__(self, other) -> bool: ...

class Printable(Protocol):
    """Protocol สำหรับ object ที่ print ได้"""
    def __str__(self) -> str: ...

def sort_and_print(items: List[Sortable]) -> None:
    """ทำงานกับ Sortable ใดๆ"""
    sorted_items = sorted(items)
    for item in sorted_items:
        print(item)

class Score:
    def __init__(self, name, value):
        self.name = name
        self.value = value
    
    def __lt__(self, other):
        return self.value < other.value
    
    def __str__(self):
        return f"{self.name}: {self.value}"

class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    def __lt__(self, other):
        return (self.year, self.month, self.day) < (other.year, other.month, other.day)
    
    def __str__(self):
        return f"{self.year:04d}-{self.month:02d}-{self.day:02d}"

# ทดสอบ
scores = [Score("Alice", 95), Score("Bob", 80), Score("Charlie", 90)]
sort_and_print(scores)

print()

dates = [Date(2024, 3, 15), Date(2023, 12, 1), Date(2024, 1, 5)]
sort_and_print(dates)
```

---

## ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Vector Class ครบถ้วน

```python
import math
from functools import total_ordering

@total_ordering
class Vector:
    """Vector class ที่รองรับ operations ต่างๆ"""
    
    def __init__(self, *components):
        if not components:
            raise ValueError("Vector ต้องมีอย่างน้อย 1 component")
        if not all(isinstance(c, (int, float)) for c in components):
            raise TypeError("ทุก component ต้องเป็นตัวเลข")
        self._components = tuple(components)
    
    @property
    def dimensions(self):
        return len(self._components)
    
    @property
    def x(self):
        return self._components[0]
    
    @property
    def y(self):
        return self._components[1] if self.dimensions > 1 else 0
    
    @property
    def z(self):
        return self._components[2] if self.dimensions > 2 else 0
    
    # ===== Arithmetic =====
    
    def _check_same_dimensions(self, other):
        if self.dimensions != other.dimensions:
            raise ValueError(
                f"Dimension mismatch: {self.dimensions} vs {other.dimensions}"
            )
    
    def __add__(self, other):
        if isinstance(other, Vector):
            self._check_same_dimensions(other)
            return Vector(*[a + b for a, b in zip(self._components, other._components)])
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Vector):
            self._check_same_dimensions(other)
            return Vector(*[a - b for a, b in zip(self._components, other._components)])
        return NotImplemented
    
    def __mul__(self, other):
        if isinstance(other, (int, float)):
            return Vector(*[c * other for c in self._components])
        elif isinstance(other, Vector):
            # Dot product
            self._check_same_dimensions(other)
            return sum(a * b for a, b in zip(self._components, other._components))
        return NotImplemented
    
    def __rmul__(self, other):
        return self.__mul__(other)
    
    def __truediv__(self, scalar):
        if scalar == 0:
            raise ZeroDivisionError
        return Vector(*[c / scalar for c in self._components])
    
    def __neg__(self):
        return Vector(*[-c for c in self._components])
    
    def __abs__(self):
        """magnitude ของ vector"""
        return math.sqrt(sum(c**2 for c in self._components))
    
    # ===== Comparison =====
    
    def __eq__(self, other):
        if isinstance(other, Vector):
            return self._components == other._components
        return NotImplemented
    
    def __lt__(self, other):
        """เปรียบเทียบโดยใช้ magnitude"""
        if isinstance(other, Vector):
            return abs(self) < abs(other)
        return NotImplemented
    
    def __hash__(self):
        return hash(self._components)
    
    # ===== Container =====
    
    def __len__(self):
        return self.dimensions
    
    def __getitem__(self, index):
        return self._components[index]
    
    def __iter__(self):
        return iter(self._components)
    
    def __contains__(self, value):
        return value in self._components
    
    # ===== String =====
    
    def __str__(self):
        return f"({', '.join(str(c) for c in self._components)})"
    
    def __repr__(self):
        return f"Vector{self._components}"
    
    def __format__(self, spec):
        if spec == '.2f':
            return f"({', '.join(f'{c:.2f}' for c in self._components)})"
        return str(self)
    
    # ===== Methods =====
    
    def magnitude(self):
        return abs(self)
    
    def normalize(self):
        """Unit vector"""
        mag = self.magnitude()
        if mag == 0:
            raise ValueError("ไม่สามารถ normalize zero vector")
        return self / mag
    
    def dot(self, other):
        """Dot product"""
        return self * other
    
    def cross(self, other):
        """Cross product (3D เท่านั้น)"""
        if self.dimensions != 3 or other.dimensions != 3:
            raise ValueError("Cross product ใช้ได้กับ 3D vector เท่านั้น")
        return Vector(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )
    
    def angle_to(self, other):
        """มุมระหว่าง vectors (radians)"""
        cos_angle = self.dot(other) / (self.magnitude() * other.magnitude())
        cos_angle = max(-1, min(1, cos_angle))  # clamp ไม่ให้เกิน -1 ถึง 1
        return math.acos(cos_angle)
    
    def project_onto(self, other):
        """Projection ของ self ลงบน other"""
        return other * (self.dot(other) / other.dot(other))
    
    @classmethod
    def zero(cls, dimensions=3):
        return cls(*([0] * dimensions))
    
    @classmethod
    def from_list(cls, lst):
        return cls(*lst)


# =================== ทดสอบ Vector ===================

print("=== Vector Operations ===\n")

v1 = Vector(1, 2, 3)
v2 = Vector(4, 5, 6)

print(f"v1 = {v1}")
print(f"v2 = {v2}")
print(f"v1 + v2 = {v1 + v2}")
print(f"v1 - v2 = {v1 - v2}")
print(f"v1 * 2 = {v1 * 2}")
print(f"3 * v1 = {3 * v1}")
print(f"v1 / 2 = {v1 / 2}")
print(f"-v1 = {-v1}")
print(f"|v1| = {abs(v1):.4f}")
print(f"|v2| = {abs(v2):.4f}")

print(f"\nDot product v1·v2 = {v1 * v2}")
cross = v1.cross(v2)
print(f"Cross product v1×v2 = {cross}")

angle = v1.angle_to(v2)
print(f"มุมระหว่าง v1 และ v2 = {math.degrees(angle):.2f}°")

unit = v1.normalize()
print(f"Unit vector ของ v1 = {unit:.2f}")
print(f"|unit| = {abs(unit):.6f}")

# Container operations
print(f"\nv1[0] = {v1[0]}")
print(f"2 ใน v1: {2 in v1}")
print(f"99 ใน v1: {99 in v1}")
print(f"len(v1) = {len(v1)}")

print("\nComponents:")
for i, comp in enumerate(v1):
    print(f"  [{i}] = {comp}")

# Comparison
vectors = [v2, v1, Vector(0, 0, 1), Vector(10, 0, 0)]
print(f"\nเรียงตาม magnitude: {sorted(vectors)}")
```

### โปรแกรมที่ 2: Matrix Class

```python
class Matrix:
    """Matrix class พร้อม operations ต่างๆ"""
    
    def __init__(self, data):
        """
        Parameters:
            data: list of lists (rows x cols)
        """
        if not data or not data[0]:
            raise ValueError("Matrix ต้องไม่ว่างเปล่า")
        
        rows = len(data)
        cols = len(data[0])
        
        if not all(len(row) == cols for row in data):
            raise ValueError("ทุกแถวต้องมีจำนวน columns เท่ากัน")
        
        # Validate values
        for row in data:
            for val in row:
                if not isinstance(val, (int, float)):
                    raise TypeError("ทุก element ต้องเป็นตัวเลข")
        
        self._data = [row[:] for row in data]  # deep copy
        self._rows = rows
        self._cols = cols
    
    @property
    def rows(self):
        return self._rows
    
    @property
    def cols(self):
        return self._cols
    
    @property
    def shape(self):
        return (self._rows, self._cols)
    
    @property
    def is_square(self):
        return self._rows == self._cols
    
    # ===== Container =====
    
    def __len__(self):
        """จำนวนแถว"""
        return self._rows
    
    def __getitem__(self, key):
        if isinstance(key, tuple):
            row, col = key
            return self._data[row][col]
        return self._data[key]
    
    def __setitem__(self, key, value):
        if isinstance(key, tuple):
            row, col = key
            self._data[row][col] = value
        else:
            self._data[key] = value
    
    def __iter__(self):
        return iter(self._data)
    
    # ===== Arithmetic =====
    
    def __add__(self, other):
        if self.shape != other.shape:
            raise ValueError(f"Shape ไม่ตรงกัน: {self.shape} vs {other.shape}")
        result = []
        for r in range(self._rows):
            result.append([self._data[r][c] + other._data[r][c] 
                          for c in range(self._cols)])
        return Matrix(result)
    
    def __sub__(self, other):
        if self.shape != other.shape:
            raise ValueError(f"Shape ไม่ตรงกัน: {self.shape} vs {other.shape}")
        result = []
        for r in range(self._rows):
            result.append([self._data[r][c] - other._data[r][c] 
                          for c in range(self._cols)])
        return Matrix(result)
    
    def __mul__(self, other):
        if isinstance(other, (int, float)):
            # Scalar multiplication
            return Matrix([[val * other for val in row] for row in self._data])
        elif isinstance(other, Matrix):
            # Matrix multiplication
            if self._cols != other._rows:
                raise ValueError(
                    f"ไม่สามารถคูณ {self.shape} กับ {other.shape}"
                )
            result = []
            for r in range(self._rows):
                row = []
                for c in range(other._cols):
                    val = sum(self._data[r][k] * other._data[k][c]
                             for k in range(self._cols))
                    row.append(val)
                result.append(row)
            return Matrix(result)
        return NotImplemented
    
    def __rmul__(self, other):
        if isinstance(other, (int, float)):
            return self.__mul__(other)
        return NotImplemented
    
    def __eq__(self, other):
        if isinstance(other, Matrix):
            return self._data == other._data
        return NotImplemented
    
    def __neg__(self):
        return Matrix([[-val for val in row] for row in self._data])
    
    # ===== Methods =====
    
    def transpose(self):
        """สลับแถวและคอลัมน์"""
        return Matrix([[self._data[r][c] for r in range(self._rows)]
                      for c in range(self._cols)])
    
    def trace(self):
        """ผลรวม diagonal (square matrix เท่านั้น)"""
        if not self.is_square:
            raise ValueError("trace ใช้ได้กับ square matrix เท่านั้น")
        return sum(self._data[i][i] for i in range(self._rows))
    
    def determinant(self):
        """คำนวณ determinant (2x2 เท่านั้นในตัวอย่างนี้)"""
        if not self.is_square:
            raise ValueError("determinant ต้องเป็น square matrix")
        if self._rows == 1:
            return self._data[0][0]
        if self._rows == 2:
            return (self._data[0][0] * self._data[1][1] - 
                    self._data[0][1] * self._data[1][0])
        # สำหรับ n>2 ใช้ Laplace expansion
        det = 0
        for c in range(self._cols):
            minor = [[self._data[r][cc] for cc in range(self._cols) if cc != c]
                    for r in range(1, self._rows)]
            det += ((-1)**c) * self._data[0][c] * Matrix(minor).determinant()
        return det
    
    def __str__(self):
        max_width = max(len(str(val)) 
                       for row in self._data 
                       for val in row)
        lines = []
        for row in self._data:
            formatted = [f"{val:>{max_width}}" for val in row]
            lines.append(f"  [{', '.join(formatted)}]")
        return "Matrix(\n" + "\n".join(lines) + "\n)"
    
    def __repr__(self):
        return f"Matrix({self._data!r})"
    
    @classmethod
    def identity(cls, n):
        """สร้าง identity matrix n×n"""
        data = [[1 if i == j else 0 for j in range(n)] for i in range(n)]
        return cls(data)
    
    @classmethod
    def zeros(cls, rows, cols):
        return cls([[0] * cols for _ in range(rows)])
    
    @classmethod
    def ones(cls, rows, cols):
        return cls([[1] * cols for _ in range(rows)])


# =================== ทดสอบ Matrix ===================

A = Matrix([[1, 2], [3, 4]])
B = Matrix([[5, 6], [7, 8]])

print("A =", A)
print("B =", B)
print("A + B =", A + B)
print("A - B =", A - B)
print("A * 2 =", A * 2)
print("A * B =", A * B)  # Matrix multiplication
print("A^T =", A.transpose())

print(f"\ntrace(A) = {A.trace()}")
print(f"det(A) = {A.determinant()}")

# Identity matrix
I = Matrix.identity(3)
print(f"\nIdentity 3x3:\n{I}")

# Test I * A = A (สำหรับ 2x2)
I2 = Matrix.identity(2)
print(f"I2 * A = I * A:\n{I2 * A}")

# Container operations
print(f"\nA[0] = {A[0]}")
print(f"A[0, 1] = {A[0, 1]}")
print(f"len(A) = {len(A)}")

for row in A:
    print(row)
```

### โปรแกรมที่ 3: Custom Container Class

```python
class SortedList:
    """Sorted list ที่ maintain ลำดับเสมอ"""
    
    def __init__(self, iterable=None, key=None, reverse=False):
        self._key = key or (lambda x: x)
        self._reverse = reverse
        self._data = []
        
        if iterable:
            for item in iterable:
                self.add(item)
    
    def add(self, item):
        """เพิ่ม item พร้อม maintain sort order"""
        # Binary search หาตำแหน่ง
        lo, hi = 0, len(self._data)
        key_val = self._key(item)
        
        while lo < hi:
            mid = (lo + hi) // 2
            mid_key = self._key(self._data[mid])
            
            if self._reverse:
                if key_val > mid_key:
                    hi = mid
                else:
                    lo = mid + 1
            else:
                if key_val < mid_key:
                    hi = mid
                else:
                    lo = mid + 1
        
        self._data.insert(lo, item)
    
    def remove(self, item):
        self._data.remove(item)
    
    def pop(self, index=-1):
        return self._data.pop(index)
    
    # Container protocols
    def __len__(self):
        return len(self._data)
    
    def __contains__(self, item):
        return item in self._data
    
    def __getitem__(self, index):
        return self._data[index]
    
    def __iter__(self):
        return iter(self._data)
    
    def __reversed__(self):
        return reversed(self._data)
    
    def __add__(self, other):
        result = SortedList(self, key=self._key, reverse=self._reverse)
        for item in other:
            result.add(item)
        return result
    
    def __eq__(self, other):
        if isinstance(other, SortedList):
            return self._data == other._data
        if isinstance(other, list):
            return self._data == other
        return NotImplemented
    
    def __str__(self):
        return f"SortedList({self._data})"
    
    def __repr__(self):
        return f"SortedList({self._data!r})"
    
    def min(self):
        return self._data[0] if not self._reverse else self._data[-1]
    
    def max(self):
        return self._data[-1] if not self._reverse else self._data[0]
    
    def bisect(self, item):
        """หา index ที่จะ insert item"""
        lo, hi = 0, len(self._data)
        key_val = self._key(item)
        while lo < hi:
            mid = (lo + hi) // 2
            if key_val <= self._key(self._data[mid]):
                hi = mid
            else:
                lo = mid + 1
        return lo


# ทดสอบ
import random

# สร้าง sorted list จาก random numbers
numbers = SortedList([5, 2, 8, 1, 9, 3, 7, 4, 6])
print(f"Sorted: {numbers}")
print(f"Min: {numbers.min()}, Max: {numbers.max()}")

numbers.add(0)
numbers.add(10)
print(f"หลัง add 0 และ 10: {numbers}")

print(f"\n5 ใน list: {5 in numbers}")
print(f"99 ใน list: {99 in numbers}")
print(f"ความยาว: {len(numbers)}")

# Sorted list ของ strings (sorted ตาม len)
words = SortedList(
    ["Python", "is", "a", "great", "programming", "language"],
    key=len
)
print(f"\nเรียงตามความยาว: {words}")

# Descending order
desc = SortedList([3, 1, 4, 1, 5, 9, 2, 6], reverse=True)
print(f"Descending: {desc}")

# Iteration
print("Items:")
for item in numbers[:5]:
    print(f"  {item}")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Money Class

```python
# สร้าง Money class ที่รองรับ:
# - __add__, __sub__ (บวกเงิน)
# - __mul__, __truediv__ (คูณด้วย scalar)
# - __eq__, __lt__ (เปรียบเทียบ)
# - __str__, __repr__
# - สกุลเงินที่แตกต่างกัน (THB, USD, EUR)
# - ป้องกัน: ไม่บวก currencies ต่างกันได้

# เฉลย
class Money:
    EXCHANGE_RATES = {
        ('USD', 'THB'): 35.0,
        ('EUR', 'THB'): 38.5,
        ('THB', 'USD'): 1/35.0,
        ('THB', 'EUR'): 1/38.5,
        ('USD', 'EUR'): 35.0/38.5,
        ('EUR', 'USD'): 38.5/35.0,
    }
    
    def __init__(self, amount, currency="THB"):
        self.amount = round(float(amount), 2)
        self.currency = currency.upper()
    
    def convert_to(self, target_currency):
        if self.currency == target_currency:
            return Money(self.amount, target_currency)
        rate = self.EXCHANGE_RATES.get((self.currency, target_currency))
        if rate is None:
            raise ValueError(f"ไม่รู้อัตราแลกเปลี่ยน {self.currency} -> {target_currency}")
        return Money(self.amount * rate, target_currency)
    
    def __add__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                # แปลง other เป็น currency เดียวกัน
                other = other.convert_to(self.currency)
            return Money(self.amount + other.amount, self.currency)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                other = other.convert_to(self.currency)
            result = self.amount - other.amount
            if result < 0:
                raise ValueError("เงินไม่พอ")
            return Money(result, self.currency)
        return NotImplemented
    
    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Money(self.amount * scalar, self.currency)
        return NotImplemented
    
    def __rmul__(self, scalar):
        return self.__mul__(scalar)
    
    def __truediv__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Money(self.amount / scalar, self.currency)
        return NotImplemented
    
    def __eq__(self, other):
        if isinstance(other, Money):
            if self.currency == other.currency:
                return abs(self.amount - other.amount) < 0.01
            other = other.convert_to(self.currency)
            return abs(self.amount - other.amount) < 0.01
        return NotImplemented
    
    def __lt__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                other = other.convert_to(self.currency)
            return self.amount < other.amount
        return NotImplemented
    
    def __le__(self, other):
        return self < other or self == other
    
    def __gt__(self, other):
        if isinstance(other, Money):
            return other < self
        return NotImplemented
    
    def __ge__(self, other):
        return self > other or self == other
    
    def __hash__(self):
        # แปลงเป็น THB เพื่อ hash
        thb = self.convert_to('THB')
        return hash(round(thb.amount, 2))
    
    def __str__(self):
        return f"{self.amount:,.2f} {self.currency}"
    
    def __repr__(self):
        return f"Money({self.amount}, {self.currency!r})"

# ทดสอบ
price1 = Money(1000, "THB")
price2 = Money(500, "THB")
price_usd = Money(10, "USD")

print(price1 + price2)          # 1,500.00 THB
print(price1 - price2)          # 500.00 THB
print(price1 + price_usd)       # ??? THB (แปลง USD เป็น THB)
print(price1 * 1.1)             # 1,100.00 THB
print(price1 / 2)               # 500.00 THB

print(price1 > price2)          # True
print(price_usd.convert_to('THB'))  # 350.00 THB
```

### ข้อที่ 2 - 10: โจทย์ฝึกหัดเพิ่มเติม

```python
# ข้อที่ 2: Polynomial class
# - __add__, __sub__, __mul__ สำหรับ polynomials
# - __call__(x) คำนวณค่า f(x)
# - __str__ แสดง: 3x^2 + 2x + 1

# ข้อที่ 3: Queue class (FIFO) แบบ iterable
# - enqueue(), dequeue(), peek()
# - __len__, __iter__, __contains__, __str__
# - __add__ (รวม 2 queues)

# ข้อที่ 4: Interval class
# - Interval(start, end)
# - __contains__ (x in interval)
# - __and__ (intersection), __or__ (union)
# - __len__ (ความยาว interval)

# ข้อที่ 5: Color class
# - Color(r, g, b) หรือ Color.from_hex("#FF8000")
# - __add__ (mix colors), __mul__ (darken/lighten)
# - __eq__, __str__, to_hex()

# ข้อที่ 6: Matrix Iterator
# - สร้าง MatrixIterator ที่ iterate เป็น row-by-row, col-by-col, spiral

# ข้อที่ 7: Pipeline class
# - Pipeline([func1, func2, func3])
# - __call__(x) รัน x ผ่านทุก function ตามลำดับ
# - __add__ เพิ่ม function ใหม่
# - __len__ จำนวน functions

# ข้อที่ 8: Graph class (ด้วย adjacency matrix)
# - __getitem__, __setitem__ สำหรับ edges
# - __contains__ ตรวจว่า node มีอยู่
# - __iter__ iterate nodes
# - __len__ จำนวน nodes

# ข้อที่ 9: TypedList - list ที่เก็บได้เฉพาะ type ที่กำหนด
# - TypedList(int) - เก็บเฉพาะ int
# - TypedList(str) - เก็บเฉพาะ str
# - __getitem__, __setitem__, __len__, __iter__, __contains__

# ข้อที่ 10: Event class สำหรับ observer pattern
# - Event()
# - __iadd__ (+= callback) subscribe
# - __isub__ (-= callback) unsubscribe
# - __call__(*args) fire event
```

#### เฉลยข้อที่ 2: Polynomial Class

```python
class Polynomial:
    """Polynomial class: a_n*x^n + ... + a_1*x + a_0"""
    
    def __init__(self, *coefficients):
        """
        coefficients: จากกำลังต่ำสุดไปสูงสุด
        Polynomial(1, 2, 3) = 1 + 2x + 3x^2
        """
        # ตัด zeros trailing
        coefs = list(coefficients)
        while len(coefs) > 1 and coefs[-1] == 0:
            coefs.pop()
        self.coefficients = tuple(coefs)
    
    @property
    def degree(self):
        return len(self.coefficients) - 1
    
    def __call__(self, x):
        """f(x) - Horner's method"""
        result = 0
        for coef in reversed(self.coefficients):
            result = result * x + coef
        return result
    
    def __add__(self, other):
        if isinstance(other, (int, float)):
            other = Polynomial(other)
        # Pad shorter one with zeros
        a = self.coefficients
        b = other.coefficients
        length = max(len(a), len(b))
        a = a + (0,) * (length - len(a))
        b = b + (0,) * (length - len(b))
        return Polynomial(*[x + y for x, y in zip(a, b)])
    
    def __sub__(self, other):
        if isinstance(other, (int, float)):
            other = Polynomial(other)
        return self.__add__(-other)
    
    def __mul__(self, other):
        if isinstance(other, (int, float)):
            return Polynomial(*[c * other for c in self.coefficients])
        result = [0] * (len(self.coefficients) + len(other.coefficients) - 1)
        for i, a in enumerate(self.coefficients):
            for j, b in enumerate(other.coefficients):
                result[i + j] += a * b
        return Polynomial(*result)
    
    def __neg__(self):
        return Polynomial(*[-c for c in self.coefficients])
    
    def __eq__(self, other):
        if isinstance(other, Polynomial):
            return self.coefficients == other.coefficients
        return NotImplemented
    
    def __str__(self):
        if all(c == 0 for c in self.coefficients):
            return "0"
        
        terms = []
        for power, coef in reversed(list(enumerate(self.coefficients))):
            if coef == 0:
                continue
            
            # Format coefficient
            if power == 0:
                term = str(coef)
            elif coef == 1:
                coef_str = ""
            elif coef == -1:
                coef_str = "-"
            else:
                coef_str = str(coef)
            
            # Format variable
            if power == 0:
                pass  # ไม่มี x
            elif power == 1:
                term = f"{coef_str}x"
            else:
                term = f"{coef_str}x^{power}"
            
            terms.append(term)
        
        if not terms:
            return "0"
        
        result = terms[0]
        for term in terms[1:]:
            if term.startswith('-'):
                result += f" - {term[1:]}"
            else:
                result += f" + {term}"
        
        return result
    
    def __repr__(self):
        return f"Polynomial{self.coefficients}"


# ทดสอบ
p1 = Polynomial(1, 2, 3)     # 1 + 2x + 3x^2
p2 = Polynomial(0, 1, 0, 1)  # x + x^3

print(f"p1 = {p1}")
print(f"p2 = {p2}")
print(f"p1(2) = {p1(2)}")    # 1 + 4 + 12 = 17
print(f"p1 + p2 = {p1 + p2}")
print(f"p1 - p2 = {p1 - p2}")
print(f"p1 * p2 = {p1 * p2}")
print(f"p1 * 2 = {p1 * 2}")
```

---

## สรุป Part 23

| Dunder Method | Operator/Function | ตัวอย่าง |
|--------------|-------------------|---------|
| `__add__` | `+` | `v1 + v2` |
| `__sub__` | `-` | `v1 - v2` |
| `__mul__` | `*` | `v1 * 3` |
| `__truediv__` | `/` | `v1 / 2` |
| `__eq__` | `==` | `a == b` |
| `__lt__` | `<` | `a < b` |
| `__len__` | `len()` | `len(obj)` |
| `__getitem__` | `[]` | `obj[0]` |
| `__setitem__` | `[] =` | `obj[0] = x` |
| `__contains__` | `in` | `x in obj` |
| `__iter__` | `for` | `for x in obj` |
| `__next__` | `next()` | `next(it)` |
| `__call__` | `()` | `obj(args)` |
| `__str__` | `str()` | `str(obj)` |
| `__repr__` | `repr()` | `repr(obj)` |

**ต่อไป**: Part 24 - OOP Encapsulation & Properties
