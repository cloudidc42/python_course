# Part 21 - OOP: Classes & Objects

## สารบัญ
1. [OOP Paradigm คืออะไร](#oop-paradigm-คืออะไร)
2. [Class Definition และ Syntax](#class-definition-และ-syntax)
3. [__init__ Method (Constructor)](#__init__-method-constructor)
4. [Instance Variables vs Class Variables](#instance-variables-vs-class-variables)
5. [Instance Methods](#instance-methods)
6. [self Parameter](#self-parameter)
7. [Object Creation (Instantiation)](#object-creation-instantiation)
8. [__str__ และ __repr__](#__str__-และ-__repr__)
9. [__del__ Destructor](#__del__-destructor)
10. [Class Methods (@classmethod)](#class-methods-classmethod)
11. [Static Methods (@staticmethod)](#static-methods-staticmethod)
12. [Property Decorators (@property)](#property-decorators-property)
13. [__dict__ และ dir()](#__dict__-และ-dir)
14. [Object Comparison](#object-comparison)
15. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
16. [แบบฝึกหัด](#แบบฝึกหัด)

---

## OOP Paradigm คืออะไร

**Object-Oriented Programming (OOP)** หรือ การเขียนโปรแกรมเชิงวัตถุ คือกระบวนทัศน์ (paradigm) การเขียนโปรแกรมที่จัดระเบียบโค้ดโดยใช้แนวคิดของ "วัตถุ" (objects) ที่รวมข้อมูล (data) และพฤติกรรม (behavior) ไว้ด้วยกัน

### ทำไมต้องใช้ OOP?

ก่อนยุค OOP โปรแกรมเมอร์เขียนโค้ดในแบบ **Procedural Programming** ซึ่งเป็นการเขียนคำสั่งทีละขั้น แต่เมื่อโปรแกรมใหญ่ขึ้น ก็เกิดปัญหามากมาย:

1. **โค้ดยุ่งเหยิง** - ฟังก์ชันและตัวแปรกระจายอยู่ทั่วไป
2. **แก้ไขยาก** - เปลี่ยนจุดหนึ่งทำให้อีกจุดพัง
3. **นำกลับมาใช้ใหม่ยาก** - ไม่มีโครงสร้างที่ชัดเจน
4. **ทำงานร่วมกันยาก** - ทีมงานหลายคนเขียนโค้ดแล้วรวมกันลำบาก

OOP แก้ปัญหาเหล่านี้ด้วย 4 หลักการหลัก:

| หลักการ | ภาษาไทย | ความหมาย |
|---------|----------|----------|
| **Encapsulation** | การห่อหุ้ม | รวมข้อมูลและฟังก์ชันไว้ด้วยกัน ซ่อนรายละเอียดภายใน |
| **Inheritance** | การสืบทอด | คลาสใหม่รับคุณสมบัติจากคลาสเดิม |
| **Polymorphism** | พหุสัณฐาน | วัตถุต่างชนิดตอบสนองต่อคำสั่งเดียวกันได้ต่างกัน |
| **Abstraction** | การสรุปรูปแบบ | ซ่อนรายละเอียดที่ซับซ้อน แสดงเฉพาะสิ่งจำเป็น |

### เปรียบเทียบ Procedural vs OOP

```python
# ===== แบบ Procedural =====
# ข้อมูลและฟังก์ชันแยกกัน จัดการยาก

student_name = "สมชาย"
student_age = 20
student_gpa = 3.5

def get_student_info(name, age, gpa):
    return f"ชื่อ: {name}, อายุ: {age}, GPA: {gpa}"

def is_honor_student(gpa):
    return gpa >= 3.5

print(get_student_info(student_name, student_age, student_gpa))
print("เกียรตินิยม:", is_honor_student(student_gpa))


# ===== แบบ OOP =====
# ข้อมูลและฟังก์ชันอยู่ด้วยกัน จัดการง่าย

class Student:
    def __init__(self, name, age, gpa):
        self.name = name
        self.age = age
        self.gpa = gpa
    
    def get_info(self):
        return f"ชื่อ: {self.name}, อายุ: {self.age}, GPA: {self.gpa}"
    
    def is_honor(self):
        return self.gpa >= 3.5

student = Student("สมชาย", 20, 3.5)
print(student.get_info())
print("เกียรตินิยม:", student.is_honor())
```

---

## Class Definition และ Syntax

**Class** คือ "แบบพิมพ์" หรือ "เทมเพลต" สำหรับสร้าง objects ในชีวิตจริงเปรียบได้กับ "แบบบ้าน" ส่วน object คือ "บ้านจริงๆ" ที่สร้างจากแบบนั้น

### Syntax พื้นฐาน

```python
class ClassName:
    """Docstring อธิบาย class"""
    
    # Class body
    pass
```

### ตัวอย่างที่ 1: Class เปล่าที่สุด

```python
# การสร้าง class อย่างง่ายที่สุด
class Dog:
    pass

# สร้าง object จาก class
my_dog = Dog()
print(type(my_dog))       # <class '__main__.Dog'>
print(isinstance(my_dog, Dog))  # True
```

### ตัวอย่างที่ 2: Class พร้อม Attributes พื้นฐาน

```python
class Cat:
    """Class แทนแมว"""
    
    # Class variable - ใช้ร่วมกันทุก instance
    species = "Felis catus"
    
    def __init__(self, name, color):
        # Instance variables - แต่ละ instance มีของตัวเอง
        self.name = name
        self.color = color
    
    def speak(self):
        return f"{self.name} พูดว่า: เมี๊ยว!"

# สร้าง objects
cat1 = Cat("มะหมา", "ส้ม")
cat2 = Cat("ดำมัน", "ดำ")

print(cat1.speak())           # มะหมา พูดว่า: เมี๊ยว!
print(cat2.speak())           # ดำมัน พูดว่า: เมี๊ยว!
print(cat1.species)           # Felis catus
print(cat2.species)           # Felis catus
print(Cat.species)            # Felis catus (เข้าถึงผ่าน class โดยตรง)
```

### ตัวอย่างที่ 3: Naming Conventions

```python
# PEP 8 - แนวทางการตั้งชื่อ Python

class MyClass:              # PascalCase สำหรับ Class
    my_variable = 10        # snake_case สำหรับตัวแปร
    
    def my_method(self):    # snake_case สำหรับ method
        pass

class HTTPServer:           # Acronym เขียนตัวใหญ่ทั้งหมด
    pass

class XMLParser:            # Acronym เขียนตัวใหญ่ทั้งหมด
    pass
```

---

## __init__ Method (Constructor)

`__init__` คือ **special method** (หรือ "dunder method" - double underscore method) ที่ Python เรียกโดยอัตโนมัติเมื่อสร้าง object ใหม่ มันทำหน้าที่ initialize (กำหนดค่าเริ่มต้น) ให้กับ attributes ของ object

### ตัวอย่างที่ 4: __init__ พื้นฐาน

```python
class Person:
    def __init__(self, name, age):
        """
        Constructor - ถูกเรียกเมื่อสร้าง Person object
        
        Parameters:
            name (str): ชื่อบุคคล
            age (int): อายุบุคคล
        """
        print(f"กำลังสร้าง Person: {name}")
        self.name = name
        self.age = age

# เมื่อเรียก Person("สมชาย", 25) Python จะ:
# 1. สร้าง object ใหม่
# 2. เรียก Person.__init__(new_object, "สมชาย", 25) โดยอัตโนมัติ
p = Person("สมชาย", 25)
# Output: กำลังสร้าง Person: สมชาย

print(p.name)   # สมชาย
print(p.age)    # 25
```

### ตัวอย่างที่ 5: __init__ พร้อม Default Values

```python
class Book:
    def __init__(self, title, author, year=None, pages=0):
        """
        Parameters มีค่า default ทำให้ไม่จำเป็นต้องส่งทุก argument
        """
        self.title = title
        self.author = author
        self.year = year
        self.pages = pages
        self.is_read = False  # attribute ที่ไม่รับ parameter
    
    def mark_as_read(self):
        self.is_read = True
        print(f"คุณอ่าน '{self.title}' แล้ว!")

# สร้าง object ต่างแบบ
book1 = Book("Python เบื้องต้น", "สมชาย จริงใจ")
book2 = Book("Clean Code", "Robert Martin", 2008, 464)

print(book1.title, book1.year, book1.pages)
# Python เบื้องต้น None 0

print(book2.title, book2.year, book2.pages)
# Clean Code 2008 464

book1.mark_as_read()  # คุณอ่าน 'Python เบื้องต้น' แล้ว!
print(book1.is_read)  # True
```

### ตัวอย่างที่ 6: __init__ พร้อม Validation

```python
class Temperature:
    """Class สำหรับจัดการอุณหภูมิ พร้อม validation"""
    
    ABSOLUTE_ZERO_CELSIUS = -273.15
    
    def __init__(self, celsius):
        """
        สร้าง Temperature object พร้อม validate ค่า
        
        Parameters:
            celsius (float): อุณหภูมิในหน่วยเซลเซียส
            
        Raises:
            ValueError: ถ้าอุณหภูมิต่ำกว่า absolute zero
        """
        if celsius < self.ABSOLUTE_ZERO_CELSIUS:
            raise ValueError(
                f"อุณหภูมิต้องไม่ต่ำกว่า {self.ABSOLUTE_ZERO_CELSIUS}°C "
                f"(Absolute Zero)"
            )
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @property
    def fahrenheit(self):
        return (self._celsius * 9/5) + 32
    
    @property
    def kelvin(self):
        return self._celsius - self.ABSOLUTE_ZERO_CELSIUS

# ทดสอบ
t1 = Temperature(100)
print(f"Celsius: {t1.celsius}°C")     # 100°C
print(f"Fahrenheit: {t1.fahrenheit}°F")  # 212.0°F
print(f"Kelvin: {t1.kelvin:.2f}K")   # 373.15K

# ทดสอบ validation
try:
    t2 = Temperature(-300)
except ValueError as e:
    print(f"Error: {e}")
# Error: อุณหภูมิต้องไม่ต่ำกว่า -273.15°C (Absolute Zero)
```

---

## Instance Variables vs Class Variables

ความแตกต่างระหว่าง **Instance Variables** และ **Class Variables** เป็นเรื่องสำคัญมากใน OOP

### Instance Variables
- กำหนดใน `__init__` โดยใช้ `self.variable_name`
- **แต่ละ object มีสำเนาของตัวเอง** - เปลี่ยนของ object หนึ่งไม่กระทบอีก object

### Class Variables
- กำหนดนอก `__init__` ระดับ class
- **ทุก object ใช้ร่วมกัน** - เปลี่ยนผ่าน class กระทบทุก object

### ตัวอย่างที่ 7: ความแตกต่างระหว่าง Instance และ Class Variables

```python
class Counter:
    # Class variable - นับรวมทุก instance
    total_count = 0
    
    def __init__(self, name):
        # Instance variable - แต่ละ instance มีของตัวเอง
        self.name = name
        self.personal_count = 0
        
        # เพิ่ม class variable เมื่อสร้าง instance ใหม่
        Counter.total_count += 1
    
    def increment(self):
        self.personal_count += 1
    
    def info(self):
        return (f"{self.name}: personal={self.personal_count}, "
                f"total={Counter.total_count}")

# ทดสอบ
c1 = Counter("Alice")
c2 = Counter("Bob")
c3 = Counter("Charlie")

c1.increment()
c1.increment()
c1.increment()

c2.increment()

print(c1.info())  # Alice: personal=3, total=3
print(c2.info())  # Bob: personal=1, total=3
print(c3.info())  # Charlie: personal=0, total=3

print(f"จำนวน Counter ทั้งหมด: {Counter.total_count}")  # 3
```

### ตัวอย่างที่ 8: ระวัง! Shadowing Class Variables

```python
class Trap:
    value = 100  # Class variable
    
    def __init__(self, name):
        self.name = name

t1 = Trap("t1")
t2 = Trap("t2")

# อ่าน class variable ผ่าน instance - OK
print(t1.value)  # 100
print(t2.value)  # 100

# เปลี่ยนผ่าน CLASS - กระทบทุก instance
Trap.value = 200
print(t1.value)  # 200
print(t2.value)  # 200

# เปลี่ยนผ่าน INSTANCE - สร้าง instance variable ใหม่! (shadowing)
t1.value = 999  # สร้าง t1.value เป็น instance variable
print(t1.value)  # 999  <- instance variable ของ t1
print(t2.value)  # 200  <- ยังคง class variable
print(Trap.value)  # 200  <- class variable ไม่เปลี่ยน

# ลบ instance variable ของ t1
del t1.value
print(t1.value)  # 200  <- กลับไปใช้ class variable
```

### ตัวอย่างที่ 9: Class Variables ที่เป็น Mutable (ระวัง!)

```python
class Dangerous:
    items = []  # DANGER! shared mutable class variable
    
    def __init__(self, name):
        self.name = name
    
    def add_item(self, item):
        self.items.append(item)  # แก้ไข class variable โดยตรง!

class Safe:
    def __init__(self, name):
        self.name = name
        self.items = []  # แต่ละ instance มี list ของตัวเอง
    
    def add_item(self, item):
        self.items.append(item)

# ทดสอบ Dangerous
d1 = Dangerous("d1")
d2 = Dangerous("d2")

d1.add_item("apple")
d2.add_item("banana")

print(d1.items)  # ['apple', 'banana'] - ปนกัน!
print(d2.items)  # ['apple', 'banana'] - ปนกัน!

# ทดสอบ Safe
s1 = Safe("s1")
s2 = Safe("s2")

s1.add_item("apple")
s2.add_item("banana")

print(s1.items)  # ['apple'] - แยกกัน
print(s2.items)  # ['banana'] - แยกกัน
```

---

## Instance Methods

**Instance Methods** คือ functions ที่กำหนดใน class และทำงานกับข้อมูลของ instance (object) เฉพาะตัว

### ตัวอย่างที่ 10: Instance Methods หลายแบบ

```python
class Rectangle:
    """สี่เหลี่ยมผืนผ้า"""
    
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    # Instance method พื้นฐาน
    def area(self):
        """คำนวณพื้นที่"""
        return self.width * self.height
    
    def perimeter(self):
        """คำนวณเส้นรอบรูป"""
        return 2 * (self.width + self.height)
    
    def scale(self, factor):
        """ขยายขนาด - รับ argument เพิ่มเติม"""
        self.width *= factor
        self.height *= factor
    
    def is_square(self):
        """ตรวจสอบว่าเป็นสี่เหลี่ยมจัตุรัสหรือไม่"""
        return self.width == self.height
    
    def resize(self, new_width, new_height):
        """เปลี่ยนขนาด"""
        self.width = new_width
        self.height = new_height
        return self  # return self เพื่อทำ method chaining
    
    def info(self):
        print(f"Rectangle: {self.width} x {self.height}")
        print(f"  พื้นที่: {self.area()}")
        print(f"  เส้นรอบรูป: {self.perimeter()}")
        print(f"  เป็นสี่เหลี่ยมจัตุรัส: {self.is_square()}")

# ทดสอบ
rect = Rectangle(5, 3)
rect.info()
# Rectangle: 5 x 3
#   พื้นที่: 15
#   เส้นรอบรูป: 16
#   เป็นสี่เหลี่ยมจัตุรัส: False

rect.scale(2)
rect.info()
# Rectangle: 10 x 6

# Method chaining
rect.resize(4, 4).info()
# Rectangle: 4 x 4
#   พื้นที่: 16
#   เส้นรอบรูป: 16
#   เป็นสี่เหลี่ยมจัตุรัส: True
```

---

## self Parameter

`self` คือ reference ที่ชี้ไปยัง instance ของ object ปัจจุบัน เป็น convention ที่ Python ใช้ ต้องเป็น parameter แรกของ instance method เสมอ (แต่ตั้งชื่ออะไรก็ได้ - แนะนำให้ใช้ `self` ตาม convention)

### ตัวอย่างที่ 11: เข้าใจ self

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def distance_to_origin(self):
        # self.x และ self.y อ้างถึง attributes ของ instance นี้
        return (self.x**2 + self.y**2) ** 0.5
    
    def distance_to(self, other):
        # self = instance ปัจจุบัน
        # other = instance อีกตัวที่ส่งมา
        dx = self.x - other.x
        dy = self.y - other.y
        return (dx**2 + dy**2) ** 0.5
    
    # self ไม่จำเป็นต้องชื่อ self แต่นิยมใช้ตาม convention
    def move(this, dx, dy):  # ใช้ 'this' แทน 'self' ได้ แต่ไม่แนะนำ
        this.x += dx
        this.y += dy

p1 = Point(0, 0)
p2 = Point(3, 4)

print(p1.distance_to_origin())  # 0.0
print(p2.distance_to_origin())  # 5.0
print(p1.distance_to(p2))       # 5.0

# Python แปลง p1.distance_to(p2) เป็น Point.distance_to(p1, p2)
# self = p1, other = p2
print(Point.distance_to(p1, p2))  # 5.0 (เรียกตรงๆ)
```

### ตัวอย่างที่ 12: self ใน Method Calls

```python
class Chain:
    def __init__(self, value=0):
        self.value = value
    
    def add(self, n):
        self.value += n
        return self  # คืน self เพื่อ method chaining
    
    def multiply(self, n):
        self.value *= n
        return self
    
    def subtract(self, n):
        self.value -= n
        return self
    
    def result(self):
        return self.value

# Method chaining - เรียก method ต่อเนื่องกัน
answer = Chain(10).add(5).multiply(2).subtract(3).result()
print(answer)  # (10+5)*2-3 = 27
```

---

## Object Creation (Instantiation)

**Instantiation** คือกระบวนการสร้าง object จาก class ซึ่งใช้ class เหมือนฟังก์ชัน

### ตัวอย่างที่ 13: กระบวนการสร้าง Object

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        """
        __new__ ถูกเรียกก่อน __init__
        ทำหน้าที่ allocate memory และสร้าง instance
        ปกติไม่ต้อง override เว้นแต่มีเหตุผลพิเศษ
        """
        print(f"1. __new__ - กำลังสร้าง instance ใหม่")
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, name):
        print(f"2. __init__ - กำลัง initialize: {name}")
        self.name = name
    
    def greet(self):
        return f"สวัสดี, ฉันชื่อ {self.name}"

print("=== สร้าง object ===")
obj = MyClass("Python")
print(f"3. object ถูกสร้างแล้ว: {obj.greet()}")
```

### ตัวอย่างที่ 14: สร้าง Objects หลายตัว

```python
class Laptop:
    count = 0  # นับจำนวน laptop ที่สร้าง
    
    def __init__(self, brand, model, ram_gb, storage_gb):
        self.brand = brand
        self.model = model
        self.ram_gb = ram_gb
        self.storage_gb = storage_gb
        self.is_on = False
        Laptop.count += 1
        self.serial = f"LP{Laptop.count:04d}"  # Serial number
    
    def power_toggle(self):
        self.is_on = not self.is_on
        status = "เปิด" if self.is_on else "ปิด"
        print(f"{self.brand} {self.model} [{self.serial}]: {status}")

# สร้าง laptops หลายตัว
laptops = [
    Laptop("Apple", "MacBook Pro", 16, 512),
    Laptop("Dell", "XPS 15", 32, 1000),
    Laptop("Lenovo", "ThinkPad X1", 16, 256),
]

print(f"จำนวน Laptop ทั้งหมด: {Laptop.count}")  # 3

for laptop in laptops:
    print(f"{laptop.serial}: {laptop.brand} {laptop.model} "
          f"- RAM: {laptop.ram_gb}GB, Storage: {laptop.storage_gb}GB")

laptops[0].power_toggle()  # Apple MacBook Pro [LP0001]: เปิด
```

---

## __str__ และ __repr__

สอง special methods นี้ควบคุมการแสดงผล object ในรูปแบบ string

- `__str__`: ใช้สำหรับ "human-readable" string (ผู้ใช้ทั่วไปอ่าน)
- `__repr__`: ใช้สำหรับ "developer/debugging" string (ควรเป็น code ที่รันแล้วได้ object เดิม)

### ตัวอย่างที่ 15: __str__ และ __repr__

```python
class Color:
    def __init__(self, red, green, blue):
        if not all(0 <= v <= 255 for v in [red, green, blue]):
            raise ValueError("ค่า RGB ต้องอยู่ระหว่าง 0-255")
        self.red = red
        self.green = green
        self.blue = blue
    
    def __str__(self):
        """Human-readable: ใช้กับ print() และ str()"""
        return f"RGB({self.red}, {self.green}, {self.blue})"
    
    def __repr__(self):
        """Developer representation: ใช้กับ repr() และใน Python shell"""
        return f"Color(red={self.red}, green={self.green}, blue={self.blue})"
    
    def to_hex(self):
        return f"#{self.red:02X}{self.green:02X}{self.blue:02X}"

c = Color(255, 128, 0)
print(c)          # RGB(255, 128, 0)          <- ใช้ __str__
print(str(c))     # RGB(255, 128, 0)          <- ใช้ __str__
print(repr(c))    # Color(red=255, green=128, blue=0)  <- ใช้ __repr__
print(c.to_hex()) # #FF8000

# ใน list จะใช้ __repr__
colors = [Color(255, 0, 0), Color(0, 255, 0), Color(0, 0, 255)]
print(colors)
# [Color(red=255, green=0, blue=0), Color(red=0, green=255, blue=0), ...]
```

### ตัวอย่างที่ 16: เมื่อไม่มี __str__

```python
class WithoutStr:
    def __init__(self, x):
        self.x = x
    
    # ไม่มี __str__ และ __repr__

class WithReprOnly:
    def __init__(self, x):
        self.x = x
    
    def __repr__(self):
        return f"WithReprOnly({self.x})"
    
    # ถ้ามี __repr__ แต่ไม่มี __str__
    # Python จะใช้ __repr__ แทน __str__ โดยอัตโนมัติ

w1 = WithoutStr(42)
print(w1)    # <__main__.WithoutStr object at 0x...> (default)

w2 = WithReprOnly(42)
print(w2)    # WithReprOnly(42)  <- ใช้ __repr__ แทน __str__
print(str(w2))   # WithReprOnly(42)
print(repr(w2))  # WithReprOnly(42)
```

### ตัวอย่างที่ 17: __format__ สำหรับ f-string

```python
class Money:
    def __init__(self, amount, currency="THB"):
        self.amount = amount
        self.currency = currency
    
    def __str__(self):
        return f"{self.amount:,.2f} {self.currency}"
    
    def __repr__(self):
        return f"Money(amount={self.amount}, currency='{self.currency}')"
    
    def __format__(self, format_spec):
        """ควบคุมการแสดงผลใน f-string"""
        if format_spec == "short":
            return f"{self.amount:,.0f} {self.currency}"
        elif format_spec == "full":
            return f"{self.amount:,.2f} บาทไทย ({self.currency})"
        else:
            return str(self)

price = Money(1234567.89)
print(price)               # 1,234,567.89 THB
print(f"{price}")          # 1,234,567.89 THB
print(f"{price:short}")    # 1,234,568 THB
print(f"{price:full}")     # 1,234,567.89 บาทไทย (THB)
```

---

## __del__ Destructor

`__del__` ถูกเรียกเมื่อ object ถูกลบหรือถูก garbage collected แต่ **ไม่ควรพึ่งพา** เพราะไม่รับประกันว่าจะถูกเรียกเมื่อไหร่

### ตัวอย่างที่ 18: __del__ Destructor

```python
import time

class DatabaseConnection:
    """จำลองการเชื่อมต่อฐานข้อมูล"""
    
    active_connections = 0
    
    def __init__(self, host, database):
        self.host = host
        self.database = database
        self.connected = True
        DatabaseConnection.active_connections += 1
        print(f"เชื่อมต่อ {database}@{host} แล้ว "
              f"(connections: {DatabaseConnection.active_connections})")
    
    def query(self, sql):
        if not self.connected:
            raise RuntimeError("ไม่ได้เชื่อมต่อ")
        return f"Results of: {sql}"
    
    def close(self):
        """ปิด connection อย่างถูกต้อง - ใช้วิธีนี้แทน __del__"""
        if self.connected:
            self.connected = False
            DatabaseConnection.active_connections -= 1
            print(f"ปิด connection {self.database} แล้ว "
                  f"(connections: {DatabaseConnection.active_connections})")
    
    def __del__(self):
        """ถูกเรียกเมื่อ object ถูก garbage collected"""
        if self.connected:
            print(f"WARNING: connection {self.database} ไม่ได้ปิดอย่างถูกต้อง!")
            self.close()

# วิธีที่ดี - ใช้ context manager (with statement)
# จะเรียน __enter__ และ __exit__ ในภายหลัง

conn = DatabaseConnection("localhost", "mydb")
print(conn.query("SELECT * FROM users"))

conn.close()  # ปิด connection อย่างถูกต้อง

# เมื่อ object ถูก del
conn2 = DatabaseConnection("remote", "testdb")
del conn2  # เรียก __del__ โดยตรง
```

---

## Class Methods (@classmethod)

**Class Methods** คือ methods ที่รับ class (ไม่ใช่ instance) เป็น argument แรก ใช้ `cls` แทน `self` มักใช้เพื่อสร้าง **alternative constructors**

### ตัวอย่างที่ 19: @classmethod เป็น Alternative Constructor

```python
from datetime import date

class Person:
    def __init__(self, name, birth_year):
        self.name = name
        self.birth_year = birth_year
    
    @property
    def age(self):
        return date.today().year - self.birth_year
    
    # Alternative constructor จาก birth date string
    @classmethod
    def from_birth_date(cls, name, birth_date_str):
        """สร้าง Person จาก birth date string (YYYY-MM-DD)"""
        birth_year = int(birth_date_str.split("-")[0])
        return cls(name, birth_year)  # cls คือ Person (หรือ subclass)
    
    # Alternative constructor จาก dict
    @classmethod
    def from_dict(cls, data):
        """สร้าง Person จาก dictionary"""
        return cls(
            name=data["name"],
            birth_year=data["birth_year"]
        )
    
    def __str__(self):
        return f"{self.name} (อายุ {self.age} ปี)"

# ใช้ constructor หลัก
p1 = Person("สมชาย", 1990)
print(p1)  # สมชาย (อายุ 35 ปี) - ขึ้นอยู่กับปีปัจจุบัน

# ใช้ alternative constructor
p2 = Person.from_birth_date("สมหญิง", "1995-06-15")
print(p2)

p3 = Person.from_dict({"name": "สมศรี", "birth_year": 2000})
print(p3)
```

### ตัวอย่างที่ 20: @classmethod สำหรับ Factory Pattern

```python
class Animal:
    registry = {}  # เก็บชนิดสัตว์ทั้งหมด
    
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound
    
    @classmethod
    def register(cls, animal_type, animal_class):
        """ลงทะเบียน animal class"""
        cls.registry[animal_type] = animal_class
    
    @classmethod
    def create(cls, animal_type, name):
        """สร้าง animal ตาม type"""
        if animal_type not in cls.registry:
            raise ValueError(f"ไม่รู้จัก animal type: {animal_type}")
        return cls.registry[animal_type](name)
    
    def speak(self):
        return f"{self.name}: {self.sound}"

class Dog(Animal):
    def __init__(self, name):
        super().__init__(name, "โฮ่ง!")

class Cat(Animal):
    def __init__(self, name):
        super().__init__(name, "เมี๊ยว!")

# ลงทะเบียน
Animal.register("dog", Dog)
Animal.register("cat", Cat)

# สร้างสัตว์
dog = Animal.create("dog", "บักโกง")
cat = Animal.create("cat", "มะหมา")

print(dog.speak())  # บักโกง: โฮ่ง!
print(cat.speak())  # มะหมา: เมี๊ยว!
```

---

## Static Methods (@staticmethod)

**Static Methods** คือ methods ที่ไม่ต้องการ `self` หรือ `cls` - เป็นแค่ functions ที่อยู่ใน class namespace ใช้เมื่อ logic เกี่ยวข้องกับ class แต่ไม่ต้องการข้อมูลจาก instance หรือ class

### ตัวอย่างที่ 21: @staticmethod

```python
class MathHelper:
    """Helper class สำหรับ mathematical operations"""
    
    # Instance method - ต้องการ self
    def __init__(self):
        self.history = []
    
    # Static method - ไม่ต้องการ self หรือ cls
    @staticmethod
    def is_prime(n):
        """ตรวจสอบว่า n เป็นจำนวนเฉพาะหรือไม่"""
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True
    
    @staticmethod
    def factorial(n):
        """คำนวณ factorial"""
        if n < 0:
            raise ValueError("n ต้องไม่น้อยกว่า 0")
        if n <= 1:
            return 1
        return n * MathHelper.factorial(n - 1)
    
    @staticmethod
    def gcd(a, b):
        """Greatest Common Divisor โดย Euclidean algorithm"""
        while b:
            a, b = b, a % b
        return a

# เรียกผ่าน class (ไม่ต้องสร้าง instance)
print(MathHelper.is_prime(17))   # True
print(MathHelper.is_prime(20))   # False
print(MathHelper.factorial(5))   # 120
print(MathHelper.gcd(48, 18))    # 6

# เรียกผ่าน instance ก็ได้ แต่ไม่นิยม
helper = MathHelper()
print(helper.is_prime(23))  # True (แต่ควรเรียกผ่าน class)
```

### ตัวอย่างที่ 22: เปรียบเทียบ 3 ประเภท Method

```python
class MethodComparison:
    class_var = "ฉันคือ Class Variable"
    
    def __init__(self, value):
        self.instance_var = value
    
    def instance_method(self):
        """สามารถเข้าถึงได้ทั้ง instance และ class"""
        return f"Instance: {self.instance_var}, Class: {self.class_var}"
    
    @classmethod
    def class_method(cls):
        """เข้าถึงได้แค่ class variable ไม่สามารถเข้าถึง instance variable"""
        return f"Class: {cls.class_var}"
        # return cls.instance_var  # ERROR! ไม่มี instance_var บน class
    
    @staticmethod
    def static_method():
        """ไม่สามารถเข้าถึง instance หรือ class โดยตรง"""
        return "ฉันเป็น static method ไม่รู้จักทั้ง instance และ class"

obj = MethodComparison("ค่าของฉัน")

print(obj.instance_method())   # Instance: ค่าของฉัน, Class: ฉันคือ Class Variable
print(obj.class_method())      # Class: ฉันคือ Class Variable
print(obj.static_method())     # ฉันเป็น static method...

print(MethodComparison.class_method())   # เรียกผ่าน class
print(MethodComparison.static_method())  # เรียกผ่าน class
# print(MethodComparison.instance_method())  # ERROR! ต้องส่ง instance
```

---

## Property Decorators (@property)

`@property` ช่วยให้เราสร้าง **managed attributes** - attributes ที่มี getter, setter, deleter ทำให้ควบคุม access ได้

### ตัวอย่างที่ 23: @property พื้นฐาน

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius  # underscore = convention บอกว่า "protected"
    
    @property
    def radius(self):
        """Getter - เรียกเมื่ออ่าน circle.radius"""
        print("กำลังอ่าน radius")
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """Setter - เรียกเมื่อกำหนด circle.radius = value"""
        print(f"กำลังกำหนด radius = {value}")
        if value < 0:
            raise ValueError("รัศมีต้องไม่ติดลบ")
        self._radius = value
    
    @radius.deleter
    def radius(self):
        """Deleter - เรียกเมื่อ del circle.radius"""
        print("กำลังลบ radius")
        del self._radius
    
    @property
    def diameter(self):
        """Computed property - ไม่มี setter = read-only"""
        return self._radius * 2
    
    @property
    def area(self):
        """อีก computed property"""
        import math
        return math.pi * self._radius ** 2

c = Circle(5)
print(c.radius)    # กำลังอ่าน radius \n 5
c.radius = 10      # กำลังกำหนด radius = 10
print(c.diameter)  # 20 (computed, no setter)
print(f"พื้นที่: {c.area:.2f}")  # พื้นที่: 314.16

try:
    c.radius = -1
except ValueError as e:
    print(f"Error: {e}")  # Error: รัศมีต้องไม่ติดลบ

del c.radius  # กำลังลบ radius
```

### ตัวอย่างที่ 24: @property สำหรับ Data Validation

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name      # ใช้ setter ใน __init__
        self.salary = salary  # ใช้ setter ใน __init__
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError("ชื่อต้องเป็น string")
        if len(value.strip()) == 0:
            raise ValueError("ชื่อต้องไม่ว่างเปล่า")
        self._name = value.strip()
    
    @property
    def salary(self):
        return self._salary
    
    @salary.setter
    def salary(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("เงินเดือนต้องเป็นตัวเลข")
        if value < 0:
            raise ValueError("เงินเดือนต้องไม่ติดลบ")
        self._salary = float(value)
    
    @property
    def annual_salary(self):
        """Read-only computed property"""
        return self._salary * 12
    
    def give_raise(self, percent):
        self.salary *= (1 + percent / 100)
        print(f"ขึ้นเงินเดือน {percent}%: {self.salary:,.2f} บาท")

emp = Employee("สมชาย ใจดี", 30000)
print(f"{emp.name}: {emp.salary:,.2f} บาท/เดือน")
print(f"รายได้ต่อปี: {emp.annual_salary:,.2f} บาท")

emp.give_raise(10)

try:
    emp.salary = -5000
except ValueError as e:
    print(f"Error: {e}")
```

---

## __dict__ และ dir()

เครื่องมือสำหรับ **introspection** - การตรวจสอบ object ว่ามีอะไรอยู่ข้างใน

### ตัวอย่างที่ 25: __dict__ และ dir()

```python
class Sample:
    class_var = "class variable"
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self._private = "private-ish"
        self.__very_private = "very private"
    
    def method(self):
        pass
    
    @classmethod
    def class_method(cls):
        pass
    
    @staticmethod
    def static_method():
        pass

s = Sample(1, 2)

# __dict__ - แสดง attributes ของ instance (เฉพาะ instance variables)
print("=== s.__dict__ ===")
print(s.__dict__)
# {'x': 1, 'y': 2, '_private': 'private-ish', '_Sample__very_private': 'very private'}

print("\n=== Sample.__dict__ ===")
# class attributes และ methods
for key, value in Sample.__dict__.items():
    print(f"  {key}: {type(value).__name__}")

print("\n=== dir(s) - ทุกอย่างที่ object มี ===")
# รวม inherited attributes จาก object
public_attrs = [attr for attr in dir(s) if not attr.startswith('_')]
print(public_attrs)
```

### ตัวอย่างที่ 26: Introspection ขั้นสูง

```python
class IntrospectMe:
    """Class สำหรับทดสอบ introspection"""
    
    count = 0
    
    def __init__(self, name):
        self.name = name
        IntrospectMe.count += 1
    
    def greet(self):
        return f"Hello, I'm {self.name}"

obj = IntrospectMe("Test")

# ตรวจสอบ type
print(type(obj))              # <class '__main__.IntrospectMe'>
print(type(obj).__name__)     # IntrospectMe

# ตรวจสอบ class
print(obj.__class__)          # <class '__main__.IntrospectMe'>
print(obj.__class__.__name__) # IntrospectMe

# ตรวจสอบ module
print(obj.__module__)         # __main__

# ตรวจสอบว่า attribute มีอยู่หรือไม่
print(hasattr(obj, 'name'))     # True
print(hasattr(obj, 'greet'))    # True
print(hasattr(obj, 'nonexist')) # False

# ดึง attribute แบบ dynamic
attr_name = 'name'
print(getattr(obj, attr_name))  # Test
print(getattr(obj, 'nonexist', 'default'))  # default

# กำหนด attribute แบบ dynamic
setattr(obj, 'new_attr', 42)
print(obj.new_attr)  # 42

# ลบ attribute แบบ dynamic
delattr(obj, 'new_attr')
print(hasattr(obj, 'new_attr'))  # False
```

---

## Object Comparison

Python มีหลายวิธีในการเปรียบเทียบ objects

### ตัวอย่างที่ 27: Object Comparison พื้นฐาน

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = p1

# is vs == 
print(p1 is p2)   # False - คนละ object (คนละ memory address)
print(p1 is p3)   # True  - ชี้ไปยัง object เดียวกัน
print(p1 == p2)   # False - default ใช้ is (identity) ไม่ใช่ equality

print(id(p1))     # memory address ของ p1
print(id(p2))     # memory address ของ p2 (ต่างกัน)
print(id(p3))     # เหมือนกับ p1
```

### ตัวอย่างที่ 28: Custom Comparison Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def magnitude(self):
        return (self.x**2 + self.y**2)**0.5
    
    def __eq__(self, other):
        """== operator"""
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y
    
    def __ne__(self, other):
        """!= operator (optional, Python infers from __eq__)"""
        result = self.__eq__(other)
        if result is NotImplemented:
            return result
        return not result
    
    def __lt__(self, other):
        """< operator (เปรียบเทียบขนาด)"""
        if not isinstance(other, Vector):
            return NotImplemented
        return self.magnitude() < other.magnitude()
    
    def __le__(self, other):
        """<= operator"""
        return self < other or self == other
    
    def __gt__(self, other):
        """> operator"""
        if not isinstance(other, Vector):
            return NotImplemented
        return self.magnitude() > other.magnitude()
    
    def __ge__(self, other):
        """>= operator"""
        return self > other or self == other
    
    def __hash__(self):
        """ต้องกำหนดถ้ากำหนด __eq__ เพื่อใช้ใน set/dict"""
        return hash((self.x, self.y))
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(3, 4)  # magnitude = 5
v2 = Vector(3, 4)  # magnitude = 5
v3 = Vector(1, 1)  # magnitude ≈ 1.41

print(v1 == v2)  # True (same x, y)
print(v1 != v3)  # True
print(v1 > v3)   # True (5 > 1.41)
print(v3 < v1)   # True

# ใช้ใน set ได้เพราะมี __hash__
vector_set = {v1, v2, v3}
print(vector_set)  # {Vector(3, 4), Vector(1, 1)} - v1 และ v2 เหมือนกัน

# เรียงลำดับได้เพราะมี __lt__
vectors = [v1, v3, Vector(2, 2)]
print(sorted(vectors))
```

### ตัวอย่างที่ 29: functools.total_ordering

```python
from functools import total_ordering

@total_ordering  # สร้าง comparison methods อื่นๆ จาก __eq__ และ __lt__
class Student:
    def __init__(self, name, gpa):
        self.name = name
        self.gpa = gpa
    
    def __eq__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.gpa == other.gpa
    
    def __lt__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.gpa < other.gpa
    
    def __repr__(self):
        return f"Student({self.name!r}, gpa={self.gpa})"

students = [
    Student("สมชาย", 3.2),
    Student("สมหญิง", 3.8),
    Student("สมศรี", 3.5),
    Student("สมปอง", 2.9),
]

# เรียงลำดับ
ranked = sorted(students, reverse=True)
for i, s in enumerate(ranked, 1):
    print(f"อันดับ {i}: {s.name} - GPA {s.gpa}")

# ทดสอบ comparison operators ทั้งหมด
a = Student("A", 3.5)
b = Student("B", 3.8)
print(a < b)   # True
print(a <= b)  # True
print(a > b)   # False
print(a >= b)  # False
```

---

## ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: BankAccount Class

```python
from datetime import datetime
from enum import Enum

class TransactionType(Enum):
    DEPOSIT = "ฝากเงิน"
    WITHDRAWAL = "ถอนเงิน"
    TRANSFER_IN = "รับโอน"
    TRANSFER_OUT = "โอนออก"

class Transaction:
    def __init__(self, trans_type, amount, description=""):
        self.type = trans_type
        self.amount = amount
        self.description = description
        self.timestamp = datetime.now()
    
    def __str__(self):
        sign = "+" if self.type in [TransactionType.DEPOSIT, TransactionType.TRANSFER_IN] else "-"
        return (f"{self.timestamp.strftime('%Y-%m-%d %H:%M:%S')} | "
                f"{self.type.value:10} | {sign}{self.amount:>12,.2f} | "
                f"{self.description}")

class BankAccount:
    """ระบบบัญชีธนาคารอย่างง่าย"""
    
    _account_counter = 1000  # เริ่มที่ 1001
    interest_rate = 0.02     # ดอกเบี้ย 2% ต่อปี (class variable)
    
    def __init__(self, owner_name, initial_deposit=0):
        """
        สร้างบัญชีธนาคารใหม่
        
        Parameters:
            owner_name: ชื่อเจ้าของบัญชี
            initial_deposit: เงินฝากเริ่มต้น (default: 0)
        """
        if initial_deposit < 0:
            raise ValueError("เงินฝากเริ่มต้นต้องไม่ติดลบ")
        
        BankAccount._account_counter += 1
        self._account_number = f"TH{BankAccount._account_counter:08d}"
        self._owner = owner_name
        self._balance = 0.0
        self._transactions = []
        self._is_frozen = False
        self.created_at = datetime.now()
        
        if initial_deposit > 0:
            self._make_transaction(
                TransactionType.DEPOSIT,
                initial_deposit,
                "เงินฝากเปิดบัญชี"
            )
    
    # === Properties ===
    
    @property
    def account_number(self):
        return self._account_number
    
    @property
    def owner(self):
        return self._owner
    
    @property
    def balance(self):
        return self._balance
    
    @property
    def is_frozen(self):
        return self._is_frozen
    
    # === Private Methods ===
    
    def _check_not_frozen(self):
        if self._is_frozen:
            raise RuntimeError(f"บัญชี {self._account_number} ถูกระงับ")
    
    def _make_transaction(self, trans_type, amount, description=""):
        """บันทึก transaction และอัปเดต balance"""
        t = Transaction(trans_type, amount, description)
        self._transactions.append(t)
        
        if trans_type in [TransactionType.DEPOSIT, TransactionType.TRANSFER_IN]:
            self._balance += amount
        else:
            self._balance -= amount
        
        return t
    
    # === Public Methods ===
    
    def deposit(self, amount, description="ฝากเงิน"):
        """ฝากเงิน"""
        self._check_not_frozen()
        if amount <= 0:
            raise ValueError("จำนวนเงินที่ฝากต้องมากกว่า 0")
        
        t = self._make_transaction(TransactionType.DEPOSIT, amount, description)
        print(f"ฝากเงินสำเร็จ: {amount:,.2f} บาท | ยอดคงเหลือ: {self._balance:,.2f} บาท")
        return t
    
    def withdraw(self, amount, description="ถอนเงิน"):
        """ถอนเงิน"""
        self._check_not_frozen()
        if amount <= 0:
            raise ValueError("จำนวนเงินที่ถอนต้องมากกว่า 0")
        if amount > self._balance:
            raise ValueError(f"เงินไม่พอ (ยอดคงเหลือ: {self._balance:,.2f} บาท)")
        
        t = self._make_transaction(TransactionType.WITHDRAWAL, amount, description)
        print(f"ถอนเงินสำเร็จ: {amount:,.2f} บาท | ยอดคงเหลือ: {self._balance:,.2f} บาท")
        return t
    
    def transfer(self, target_account, amount, description="โอนเงิน"):
        """โอนเงินไปบัญชีอื่น"""
        self._check_not_frozen()
        target_account._check_not_frozen()
        
        if amount <= 0:
            raise ValueError("จำนวนเงินที่โอนต้องมากกว่า 0")
        if amount > self._balance:
            raise ValueError(f"เงินไม่พอโอน")
        
        # ทำ transaction ทั้งสองฝั่ง
        self._make_transaction(
            TransactionType.TRANSFER_OUT, amount,
            f"{description} -> {target_account.account_number}"
        )
        target_account._make_transaction(
            TransactionType.TRANSFER_IN, amount,
            f"{description} <- {self.account_number}"
        )
        
        print(f"โอนเงิน {amount:,.2f} บาท จาก {self.account_number} ไป {target_account.account_number} สำเร็จ")
    
    def apply_interest(self):
        """คิดดอกเบี้ยประจำปี"""
        interest = self._balance * self.interest_rate
        self._make_transaction(TransactionType.DEPOSIT, interest, "ดอกเบี้ยประจำปี")
        print(f"ดอกเบี้ย: {interest:,.2f} บาท | ยอดคงเหลือ: {self._balance:,.2f} บาท")
    
    def freeze(self):
        """ระงับบัญชี"""
        self._is_frozen = True
        print(f"ระงับบัญชี {self._account_number} แล้ว")
    
    def unfreeze(self):
        """ยกเลิกการระงับบัญชี"""
        self._is_frozen = False
        print(f"ยกเลิกการระงับบัญชี {self._account_number} แล้ว")
    
    def get_statement(self, last_n=None):
        """แสดง statement"""
        txns = self._transactions if last_n is None else self._transactions[-last_n:]
        print(f"\n{'='*70}")
        print(f"{'บัญชี: ' + self._account_number:^70}")
        print(f"{'เจ้าของ: ' + self._owner:^70}")
        print(f"{'='*70}")
        print(f"{'วันเวลา':22} | {'ประเภท':10} | {'จำนวนเงิน':>14} | คำอธิบาย")
        print(f"{'-'*70}")
        for t in txns:
            print(t)
        print(f"{'-'*70}")
        print(f"{'ยอดคงเหลือ':>50}: {self._balance:>12,.2f} บาท")
        print(f"{'='*70}\n")
    
    def __str__(self):
        status = "ถูกระงับ" if self._is_frozen else "ใช้งานปกติ"
        return (f"BankAccount(เลขที่: {self._account_number}, "
                f"เจ้าของ: {self._owner}, "
                f"ยอดคงเหลือ: {self._balance:,.2f}, "
                f"สถานะ: {status})")
    
    def __repr__(self):
        return f"BankAccount(account_number={self._account_number!r}, owner={self._owner!r})"
    
    @classmethod
    def open_joint_account(cls, owner1, owner2, initial_deposit=0):
        """เปิดบัญชีร่วม (class method)"""
        joint_name = f"{owner1} & {owner2}"
        account = cls(joint_name, initial_deposit)
        print(f"เปิดบัญชีร่วม {owner1} และ {owner2} สำเร็จ")
        return account
    
    @staticmethod
    def validate_account_number(acc_num):
        """ตรวจสอบรูปแบบเลขบัญชี (static method)"""
        return (isinstance(acc_num, str) and 
                acc_num.startswith("TH") and 
                len(acc_num) == 10 and
                acc_num[2:].isdigit())


# =================== ทดสอบ BankAccount ===================

print("=" * 50)
print("ทดสอบระบบธนาคาร")
print("=" * 50)

# สร้างบัญชี
acc1 = BankAccount("สมชาย ใจดี", 10000)
acc2 = BankAccount("สมหญิง มีทรัพย์", 5000)

print(acc1)
print(acc2)

# ฝากและถอนเงิน
print("\n--- ธุรกรรม ---")
acc1.deposit(5000, "รับเงินเดือน")
acc1.withdraw(1500, "ค่าอาหาร")
acc1.transfer(acc2, 3000, "ค่าเช่า")

# คิดดอกเบี้ย
acc1.apply_interest()

# แสดง statement
acc1.get_statement()
acc2.get_statement()

# ทดสอบ validation
print("\n--- ทดสอบ Error Handling ---")
try:
    acc1.withdraw(99999)
except ValueError as e:
    print(f"Error ถอนเงิน: {e}")

# ระงับบัญชี
acc1.freeze()
try:
    acc1.deposit(1000)
except RuntimeError as e:
    print(f"Error: {e}")

acc1.unfreeze()
acc1.deposit(1000, "ฝากหลังยกเลิกระงับ")

# บัญชีร่วม
joint = BankAccount.open_joint_account("พ่อ สุขใจ", "แม่ สุขใจ", 50000)
print(joint)

# Validate account number
print(BankAccount.validate_account_number(acc1.account_number))  # True
print(BankAccount.validate_account_number("INVALID"))             # False
```

### โปรแกรมที่ 2: Student Class

```python
from datetime import date

class Grade:
    """เกรดในแต่ละวิชา"""
    
    GRADE_SCALE = {
        (80, 100): ('A', 4.0),
        (75, 79):  ('B+', 3.5),
        (70, 74):  ('B', 3.0),
        (65, 69):  ('C+', 2.5),
        (60, 64):  ('C', 2.0),
        (55, 59):  ('D+', 1.5),
        (50, 54):  ('D', 1.0),
        (0,  49):  ('F', 0.0),
    }
    
    def __init__(self, subject, score, credit):
        self.subject = subject
        self.score = score
        self.credit = credit
        self.letter, self.grade_point = self._calculate_grade()
    
    def _calculate_grade(self):
        for (low, high), (letter, point) in self.GRADE_SCALE.items():
            if low <= self.score <= high:
                return letter, point
        return 'F', 0.0
    
    def __repr__(self):
        return f"Grade({self.subject!r}, score={self.score}, grade={self.letter})"

class Student:
    """ระบบจัดการข้อมูลนักศึกษา"""
    
    _student_count = 0
    
    def __init__(self, first_name, last_name, birth_date, faculty):
        Student._student_count += 1
        self.student_id = f"STD{Student._student_count:06d}"
        self.first_name = first_name
        self.last_name = last_name
        self.birth_date = birth_date
        self.faculty = faculty
        self._grades = []  # list ของ Grade objects
        self._year = 1
        self.email = f"{first_name.lower()}.{last_name.lower()}@university.ac.th"
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    @property
    def age(self):
        today = date.today()
        born = self.birth_date
        return today.year - born.year - ((today.month, today.day) < (born.month, born.day))
    
    @property
    def year(self):
        return self._year
    
    @year.setter
    def year(self, value):
        if value not in range(1, 5):
            raise ValueError("ชั้นปีต้องอยู่ระหว่าง 1-4")
        self._year = value
    
    @property
    def gpa(self):
        """คำนวณ GPA จากทุกวิชา"""
        if not self._grades:
            return 0.0
        total_points = sum(g.grade_point * g.credit for g in self._grades)
        total_credits = sum(g.credit for g in self._grades)
        return total_points / total_credits if total_credits > 0 else 0.0
    
    @property
    def total_credits(self):
        return sum(g.credit for g in self._grades)
    
    def add_grade(self, subject, score, credit=3):
        """เพิ่มผลการเรียน"""
        grade = Grade(subject, score, credit)
        self._grades.append(grade)
        print(f"บันทึกผล {subject}: {score} คะแนน ({grade.letter})")
        return grade
    
    def get_transcript(self):
        """แสดงใบแสดงผลการเรียน"""
        print(f"\n{'='*55}")
        print(f"{'TRANSCRIPT':^55}")
        print(f"{'='*55}")
        print(f"รหัสนักศึกษา: {self.student_id}")
        print(f"ชื่อ-นามสกุล: {self.full_name}")
        print(f"คณะ: {self.faculty} | ชั้นปี: {self._year}")
        print(f"อีเมล: {self.email}")
        print(f"{'-'*55}")
        print(f"{'วิชา':30} {'คะแนน':>7} {'เกรด':>5} {'หน่วยกิต':>8}")
        print(f"{'-'*55}")
        for g in self._grades:
            print(f"{g.subject:30} {g.score:>7} {g.letter:>5} {g.credit:>8}")
        print(f"{'-'*55}")
        print(f"{'หน่วยกิตรวม':>43}: {self.total_credits:>8}")
        print(f"{'เกรดเฉลี่ย (GPA)':>43}: {self.gpa:>8.2f}")
        
        # แสดงสถานะ
        if self.gpa >= 3.5:
            status = "เกียรตินิยมอันดับ 1"
        elif self.gpa >= 3.25:
            status = "เกียรตินิยมอันดับ 2"
        elif self.gpa >= 2.0:
            status = "ผ่านการศึกษา"
        else:
            status = "ต้องปรับปรุง"
        
        print(f"{'สถานะ':>43}: {status:>8}")
        print(f"{'='*55}\n")
    
    def __str__(self):
        return (f"Student({self.student_id}: {self.full_name}, "
                f"GPA: {self.gpa:.2f})")
    
    def __repr__(self):
        return (f"Student(first_name={self.first_name!r}, "
                f"last_name={self.last_name!r}, "
                f"student_id={self.student_id!r})")
    
    def __lt__(self, other):
        """เปรียบเทียบโดยใช้ GPA"""
        return self.gpa < other.gpa
    
    @classmethod
    def get_total_students(cls):
        return cls._student_count


# =================== ทดสอบ Student ===================

s1 = Student("สมชาย", "ใจดี", date(2002, 5, 15), "วิทยาการคอมพิวเตอร์")
s1.year = 3

# เพิ่มเกรด
s1.add_grade("Programming Fundamentals", 85, 3)
s1.add_grade("Data Structures", 78, 3)
s1.add_grade("Algorithms", 72, 3)
s1.add_grade("Database Systems", 88, 3)
s1.add_grade("Computer Networks", 65, 2)
s1.add_grade("Software Engineering", 92, 3)

s1.get_transcript()

s2 = Student("สมหญิง", "มีทรัพย์", date(2003, 8, 20), "วิทยาการคอมพิวเตอร์")
s2.year = 2
s2.add_grade("Programming Fundamentals", 95, 3)
s2.add_grade("Data Structures", 88, 3)

print(f"จำนวนนักศึกษาทั้งหมด: {Student.get_total_students()}")

# เรียงตาม GPA
students = [s1, s2]
top_students = sorted(students, reverse=True)
print("\nอันดับนักศึกษา:")
for i, s in enumerate(top_students, 1):
    print(f"  {i}. {s}")
```

### โปรแกรมที่ 3: Car Class

```python
from enum import Enum
from datetime import date

class FuelType(Enum):
    GASOLINE = "น้ำมันเบนซิน"
    DIESEL = "ดีเซล"
    ELECTRIC = "ไฟฟ้า"
    HYBRID = "ไฮบริด"

class CarCondition(Enum):
    EXCELLENT = "ดีเยี่ยม"
    GOOD = "ดี"
    FAIR = "พอใช้"
    POOR = "แย่"

class Car:
    """ระบบจัดการข้อมูลรถยนต์"""
    
    _car_registry = {}  # เก็บข้อมูลรถทุกคัน
    
    def __init__(self, make, model, year, fuel_type, mileage=0):
        self.make = make
        self.model = model
        self.year = year
        self.fuel_type = fuel_type
        self._mileage = mileage
        self._service_history = []
        self._is_running = False
        self._fuel_level = 100  # เปอร์เซ็นต์
        self.vin = self._generate_vin()
        
        # ลงทะเบียน
        Car._car_registry[self.vin] = self
    
    def _generate_vin(self):
        """สร้าง VIN number"""
        import random
        import string
        chars = string.ascii_uppercase + string.digits
        return ''.join(random.choices(chars, k=17))
    
    @property
    def mileage(self):
        return self._mileage
    
    @property
    def age(self):
        return date.today().year - self.year
    
    @property
    def condition(self):
        """ประเมินสภาพรถตาม mileage"""
        if self._mileage < 50000:
            return CarCondition.EXCELLENT
        elif self._mileage < 100000:
            return CarCondition.GOOD
        elif self._mileage < 150000:
            return CarCondition.FAIR
        else:
            return CarCondition.POOR
    
    @property
    def fuel_level(self):
        return self._fuel_level
    
    def start(self):
        if self._is_running:
            print(f"{self.make} {self.model} กำลังทำงานอยู่แล้ว")
            return
        if self._fuel_level <= 0:
            raise RuntimeError("น้ำมันหมด! เติมน้ำมันก่อน")
        self._is_running = True
        print(f"{self.make} {self.model}: สตาร์ทเครื่อง 🚗")
    
    def stop(self):
        if not self._is_running:
            print(f"{self.make} {self.model} ไม่ได้ทำงานอยู่")
            return
        self._is_running = False
        print(f"{self.make} {self.model}: ดับเครื่อง")
    
    def drive(self, distance_km):
        """ขับรถ"""
        if not self._is_running:
            raise RuntimeError("ต้องสตาร์ทเครื่องก่อน")
        
        fuel_consumption = distance_km * 0.1  # 10 ลิตร/100 กม. (สมมติ)
        
        if fuel_consumption > self._fuel_level:
            max_distance = self._fuel_level / 0.1
            raise RuntimeError(
                f"น้ำมันไม่พอ สามารถขับได้อีก {max_distance:.0f} กม."
            )
        
        self._mileage += distance_km
        self._fuel_level -= fuel_consumption
        print(f"ขับ {distance_km} กม. | ระยะทางรวม: {self._mileage:,} กม. | "
              f"น้ำมัน: {self._fuel_level:.1f}%")
    
    def refuel(self, liters=None):
        """เติมน้ำมัน"""
        if self.fuel_type == FuelType.ELECTRIC:
            self._fuel_level = 100
            print(f"ชาร์จแบตเตอรี่เต็มแล้ว")
        else:
            if liters is None:
                self._fuel_level = 100
                print(f"เติมน้ำมันเต็มถัง")
            else:
                self._fuel_level = min(100, self._fuel_level + liters)
                print(f"เติมน้ำมัน {liters} ลิตร | ระดับน้ำมัน: {self._fuel_level:.1f}%")
    
    def service(self, description, cost):
        """บันทึกประวัติการซ่อม"""
        record = {
            "date": date.today().isoformat(),
            "mileage": self._mileage,
            "description": description,
            "cost": cost
        }
        self._service_history.append(record)
        print(f"บันทึกการซ่อม: {description} | ค่าใช้จ่าย: {cost:,.2f} บาท")
    
    def get_service_history(self):
        if not self._service_history:
            print("ไม่มีประวัติการซ่อม")
            return
        print(f"\nประวัติการซ่อม: {self.make} {self.model}")
        total_cost = 0
        for record in self._service_history:
            print(f"  {record['date']} | {record['mileage']:,} กม. | "
                  f"{record['description']} | {record['cost']:,.2f} บาท")
            total_cost += record['cost']
        print(f"  ค่าใช้จ่ายรวม: {total_cost:,.2f} บาท")
    
    def __str__(self):
        return (f"{self.year} {self.make} {self.model} "
                f"({self.fuel_type.value}) | "
                f"ระยะทาง: {self._mileage:,} กม. | "
                f"สภาพ: {self.condition.value}")
    
    def __repr__(self):
        return (f"Car(make={self.make!r}, model={self.model!r}, "
                f"year={self.year}, vin={self.vin!r})")
    
    @classmethod
    def find_by_vin(cls, vin):
        """ค้นหารถด้วย VIN"""
        return cls._car_registry.get(vin)
    
    @classmethod
    def total_cars(cls):
        return len(cls._car_registry)
    
    @staticmethod
    def estimate_value(original_price, age_years, mileage):
        """ประเมินราคารถมือสอง (อย่างง่าย)"""
        depreciation = 0.15 * age_years  # ลด 15% ต่อปี
        mileage_factor = 1 - (mileage / 500000)  # ลดตาม mileage
        value = original_price * (1 - depreciation) * mileage_factor
        return max(value, original_price * 0.1)  # ไม่ต่ำกว่า 10% ของราคาเดิม


# =================== ทดสอบ Car ===================

print("=" * 60)
print("ระบบจัดการรถยนต์")
print("=" * 60)

# สร้างรถ
car1 = Car("Toyota", "Camry", 2022, FuelType.HYBRID, 15000)
car2 = Car("Tesla", "Model 3", 2023, FuelType.ELECTRIC, 8000)
car3 = Car("Honda", "Civic", 2019, FuelType.GASOLINE, 85000)

print(car1)
print(car2)
print(car3)

print(f"\nจำนวนรถทั้งหมด: {Car.total_cars()}")

# ทดสอบขับรถ
print("\n--- ทดสอบขับรถ ---")
car1.start()
car1.drive(50)
car1.drive(100)
car1.stop()

# ประวัติการซ่อม
print("\n--- ประวัติการซ่อม ---")
car1.service("เปลี่ยนน้ำมันเครื่อง", 1500)
car1.service("เปลี่ยนยาง 4 เส้น", 12000)
car1.service("ตรวจสภาพประจำปี", 3000)
car1.get_service_history()

# ประเมินราคา
original_price = 1_200_000
estimated = Car.estimate_value(original_price, car3.age, car3.mileage)
print(f"\nราคาประเมิน {car3.make} {car3.model}: {estimated:,.0f} บาท "
      f"(จากราคาเดิม {original_price:,} บาท)")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Library Book Class
สร้าง class `LibraryBook` ที่มี:
- Attributes: `title`, `author`, `isbn`, `year`, `available` (bool)
- Methods: `checkout()`, `return_book()`, `info()`
- `__str__` และ `__repr__`
- Validation ใน `__init__` (year ต้องสมเหตุสมผล, isbn ต้องมีความยาว 13 หลัก)

```python
# เฉลยข้อที่ 1
class LibraryBook:
    def __init__(self, title, author, isbn, year):
        if not isinstance(isbn, str) or len(isbn.replace('-', '')) != 13:
            raise ValueError("ISBN ต้องมี 13 หลัก")
        if year < 1450 or year > 2030:
            raise ValueError("ปีที่พิมพ์ไม่ถูกต้อง")
        
        self.title = title
        self.author = author
        self.isbn = isbn
        self.year = year
        self.available = True
        self.checkout_count = 0
    
    def checkout(self, borrower_name):
        if not self.available:
            raise RuntimeError(f"หนังสือ '{self.title}' ถูกยืมแล้ว")
        self.available = False
        self.checkout_count += 1
        self._current_borrower = borrower_name
        print(f"'{self.title}' ถูกยืมโดย {borrower_name} (ครั้งที่ {self.checkout_count})")
    
    def return_book(self):
        if self.available:
            raise RuntimeError(f"หนังสือ '{self.title}' ไม่ได้ถูกยืม")
        self.available = True
        borrower = getattr(self, '_current_borrower', 'ไม่ทราบ')
        del self._current_borrower
        print(f"'{self.title}' ถูกคืนโดย {borrower}")
    
    def info(self):
        status = "ว่าง" if self.available else "ถูกยืม"
        print(f"📚 {self.title} | โดย {self.author} | ปี {self.year}")
        print(f"   ISBN: {self.isbn} | สถานะ: {status} | ยืมไปแล้ว {self.checkout_count} ครั้ง")
    
    def __str__(self):
        return f"'{self.title}' by {self.author} ({self.year})"
    
    def __repr__(self):
        return f"LibraryBook(title={self.title!r}, author={self.author!r}, isbn={self.isbn!r})"

# ทดสอบ
book = LibraryBook("Python Programming", "สมชาย ใจดี", "978-0-13-468599-1", 2023)
book.info()
book.checkout("สมหญิง")
book.checkout("สมศรี")  # Error!
```

### ข้อที่ 2: ShoppingCart Class
```python
# เฉลยข้อที่ 2
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self._price = price
        self._stock = stock
    
    @property
    def price(self):
        return self._price
    
    @property
    def stock(self):
        return self._stock
    
    def reduce_stock(self, qty):
        if qty > self._stock:
            raise ValueError(f"สินค้า '{self.name}' มีไม่พอ (มี {self._stock} ชิ้น)")
        self._stock -= qty

class ShoppingCart:
    def __init__(self, customer_name):
        self.customer_name = customer_name
        self._items = {}  # {product: quantity}
        self.discount_percent = 0
    
    def add_item(self, product, quantity=1):
        if quantity <= 0:
            raise ValueError("จำนวนสินค้าต้องมากกว่า 0")
        if product not in self._items:
            self._items[product] = 0
        self._items[product] += quantity
        print(f"เพิ่ม {product.name} x{quantity}")
    
    def remove_item(self, product, quantity=None):
        if product not in self._items:
            raise ValueError(f"ไม่มี {product.name} ในตะกร้า")
        if quantity is None or quantity >= self._items[product]:
            del self._items[product]
        else:
            self._items[product] -= quantity
    
    @property
    def subtotal(self):
        return sum(p.price * qty for p, qty in self._items.items())
    
    @property
    def discount_amount(self):
        return self.subtotal * self.discount_percent / 100
    
    @property
    def total(self):
        return self.subtotal - self.discount_amount
    
    def apply_discount(self, percent):
        if not 0 <= percent <= 100:
            raise ValueError("ส่วนลดต้องอยู่ระหว่าง 0-100%")
        self.discount_percent = percent
    
    def checkout(self):
        if not self._items:
            raise RuntimeError("ตะกร้าว่างเปล่า")
        
        print(f"\n{'='*45}")
        print(f"{'ใบเสร็จ':^45}")
        print(f"{'ลูกค้า: ' + self.customer_name:^45}")
        print(f"{'='*45}")
        
        for product, qty in self._items.items():
            print(f"{product.name:25} x{qty:3} = {product.price * qty:>10,.2f}")
            product.reduce_stock(qty)
        
        print(f"{'-'*45}")
        print(f"{'ราคารวม':>35}: {self.subtotal:>10,.2f}")
        if self.discount_percent > 0:
            print(f"{'ส่วนลด ' + str(self.discount_percent) + '%':>35}: -{self.discount_amount:>9,.2f}")
        print(f"{'ยอดที่ต้องชำระ':>35}: {self.total:>10,.2f}")
        print(f"{'='*45}")
        
        self._items.clear()
        return self.total

# ทดสอบ
p1 = Product("น้ำแอปเปิ้ล", 25, 100)
p2 = Product("ขนมปัง", 35, 50)
p3 = Product("ไข่ไก่", 120, 30)

cart = ShoppingCart("สมชาย")
cart.add_item(p1, 3)
cart.add_item(p2, 2)
cart.add_item(p3, 1)
cart.apply_discount(10)
cart.checkout()
```

### ข้อที่ 3 - 10: โจทย์ฝึกหัดเพิ่มเติม

```python
# ข้อที่ 3: สร้าง Stack class (data structure)
# - push(item), pop(), peek(), is_empty(), size
# - __len__, __contains__, __str__

# ข้อที่ 4: สร้าง Queue class
# - enqueue(item), dequeue(), front(), is_empty(), size

# ข้อที่ 5: สร้าง Matrix class (2D)
# - __init__(rows, cols, fill=0)
# - get(row, col), set(row, col, value)
# - display(), transpose()

# ข้อที่ 6: สร้าง Employee class พร้อม:
# - Attributes: name, employee_id, department, salary, hire_date
# - Methods: give_raise(percent), years_of_service(), annual_bonus()
# - Class method: from_csv(csv_string)

# ข้อที่ 7: สร้าง Playlist class สำหรับเพลง
# - add_song(), remove_song(), play_all(), shuffle(), total_duration

# ข้อที่ 8: สร้าง Fraction class (เศษส่วน)
# - __add__, __sub__, __mul__, __truediv__
# - simplify(), to_float(), __str__

# ข้อที่ 9: สร้าง Contact class สำหรับสมุดโทรศัพท์
# - ContactBook class ที่เก็บ Contact หลายรายการ
# - add_contact(), search_by_name(), search_by_phone()

# ข้อที่ 10: สร้าง Logger class (Singleton pattern)
# - เฉพาะ 1 instance เท่านั้น (ใช้ __new__)
# - log_info(), log_warning(), log_error()
# - save_to_file(), get_logs()
```

#### เฉลยข้อที่ 8: Fraction Class

```python
from math import gcd

class Fraction:
    """คลาสเศษส่วน"""
    
    def __init__(self, numerator, denominator):
        if denominator == 0:
            raise ZeroDivisionError("ตัวส่วนต้องไม่เป็น 0")
        
        # จัดการเครื่องหมาย - ตัวส่วนต้องเป็นบวก
        if denominator < 0:
            numerator, denominator = -numerator, -denominator
        
        common = gcd(abs(numerator), denominator)
        self.numerator = numerator // common
        self.denominator = denominator // common
    
    def __add__(self, other):
        if isinstance(other, int):
            other = Fraction(other, 1)
        new_num = self.numerator * other.denominator + other.numerator * self.denominator
        new_den = self.denominator * other.denominator
        return Fraction(new_num, new_den)
    
    def __sub__(self, other):
        if isinstance(other, int):
            other = Fraction(other, 1)
        new_num = self.numerator * other.denominator - other.numerator * self.denominator
        new_den = self.denominator * other.denominator
        return Fraction(new_num, new_den)
    
    def __mul__(self, other):
        if isinstance(other, int):
            other = Fraction(other, 1)
        return Fraction(
            self.numerator * other.numerator,
            self.denominator * other.denominator
        )
    
    def __truediv__(self, other):
        if isinstance(other, int):
            other = Fraction(other, 1)
        return Fraction(
            self.numerator * other.denominator,
            self.denominator * other.numerator
        )
    
    def __eq__(self, other):
        if isinstance(other, int):
            other = Fraction(other, 1)
        return (self.numerator == other.numerator and 
                self.denominator == other.denominator)
    
    def __lt__(self, other):
        return self.to_float() < other.to_float()
    
    def to_float(self):
        return self.numerator / self.denominator
    
    def __str__(self):
        if self.denominator == 1:
            return str(self.numerator)
        return f"{self.numerator}/{self.denominator}"
    
    def __repr__(self):
        return f"Fraction({self.numerator}, {self.denominator})"

# ทดสอบ
f1 = Fraction(1, 2)  # 1/2
f2 = Fraction(1, 3)  # 1/3
f3 = Fraction(2, 4)  # จะ simplify เป็น 1/2

print(f1 + f2)  # 5/6
print(f1 - f2)  # 1/6
print(f1 * f2)  # 1/6
print(f1 / f2)  # 3/2
print(f1 == f3) # True (ทั้งคู่ = 1/2)
print(f1.to_float())  # 0.5
```

---

## สรุป Part 21

ในส่วนนี้เราได้เรียนรู้พื้นฐาน OOP ใน Python:

| แนวคิด | คีย์เวิร์ด | ใช้เมื่อ |
|--------|-----------|---------|
| Class definition | `class` | สร้าง blueprint ของ object |
| Constructor | `__init__` | กำหนดค่าเริ่มต้นของ object |
| Instance variable | `self.var` | ข้อมูลเฉพาะของแต่ละ object |
| Class variable | ระดับ class | ข้อมูลที่ทุก object ใช้ร่วมกัน |
| Instance method | `def method(self)` | ฟังก์ชันที่ทำงานกับ object |
| Class method | `@classmethod` | Factory methods, alternative constructors |
| Static method | `@staticmethod` | utility functions ที่เกี่ยวกับ class |
| Property | `@property` | controlled attribute access |
| String representation | `__str__`, `__repr__` | แสดงผล object เป็น string |

**ต่อไป**: Part 22 - OOP Inheritance & Method Resolution Order
