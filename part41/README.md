# Part 41: Dataclasses & attrs

## บทนำ

ในการเขียนโปรแกรม Python เราบ่อยครั้งต้องสร้าง class ที่ทำหน้าที่เป็นแค่ "ภาชนะ" (container) สำหรับเก็บข้อมูล เช่น Point, User, Product เป็นต้น การเขียน class เหล่านี้ด้วยวิธีปกติต้องเขียน boilerplate code จำนวนมาก ได้แก่ `__init__`, `__repr__`, `__eq__` ฯลฯ

Python 3.7 แนะนำ **dataclasses** module ซึ่งช่วยลด boilerplate เหล่านี้อย่างมาก โดย decorator `@dataclass` จะ auto-generate methods เหล่านี้ให้อัตโนมัติ

---

## 1. @dataclass Decorator

### ปัญหาของ class ปกติ

```python
# วิธีเดิม - ต้องเขียน boilerplate มาก
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"
    
    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        return hash((self.x, self.y))

p1 = Point(1.0, 2.0)
p2 = Point(1.0, 2.0)
print(p1)          # Point(x=1.0, y=2.0)
print(p1 == p2)    # True
```

```python
# วิธีใหม่ด้วย @dataclass - สั้นกว่ามาก
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

p1 = Point(1.0, 2.0)
p2 = Point(1.0, 2.0)
print(p1)          # Point(x=1.0, y=2.0)
print(p1 == p2)    # True
```

ทั้งสอง class ทำงานเหมือนกันทุกประการ แต่ dataclass เขียนสั้นกว่ามาก

---

## 2. Auto-generation: __init__, __repr__, __eq__

### __init__ auto-generation

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    email: str

# @dataclass สร้าง __init__ ให้อัตโนมัติ เทียบเท่า:
# def __init__(self, name: str, age: int, email: str):
#     self.name = name
#     self.age = age
#     self.email = email

p = Person(name="Alice", age=30, email="alice@example.com")
print(p.name)   # Alice
print(p.age)    # 30
print(p.email)  # alice@example.com

# สามารถใช้ positional หรือ keyword arguments ได้
p2 = Person("Bob", 25, "bob@example.com")
print(p2)  # Person(name='Bob', age=25, email='bob@example.com')
```

### __repr__ auto-generation

```python
from dataclasses import dataclass

@dataclass
class Car:
    make: str
    model: str
    year: int
    price: float

car = Car("Toyota", "Camry", 2023, 25000.0)

# __repr__ ถูก auto-generate ให้มี format ที่อ่านง่าย
print(repr(car))
# Car(make='Toyota', model='Camry', year=2023, price=25000.0)

# ใน f-string หรือ print ก็ใช้ __repr__ เช่นกัน
print(f"My car: {car}")
# My car: Car(make='Toyota', model='Camry', year=2023, price=25000.0)
```

### __eq__ auto-generation

```python
from dataclasses import dataclass

@dataclass
class Color:
    red: int
    green: int
    blue: int

c1 = Color(255, 0, 0)
c2 = Color(255, 0, 0)
c3 = Color(0, 255, 0)

# __eq__ เปรียบเทียบ fields ทั้งหมด
print(c1 == c2)  # True
print(c1 == c3)  # False
print(c1 != c3)  # True

# แต่ไม่มี __hash__ โดยค่า default (เพราะ mutable)
# ดังนั้น hash ยังใช้ id ของ object
print(c1 is c2)  # False (คนละ object)
```

### ควบคุม auto-generation ด้วย parameters

```python
from dataclasses import dataclass

# ปิด/เปิดการ generate methods ต่างๆ
@dataclass(
    init=True,      # default: True - generate __init__
    repr=True,      # default: True - generate __repr__
    eq=True,        # default: True - generate __eq__
    order=False,    # default: False - generate __lt__, __le__, __gt__, __ge__
    unsafe_hash=False,  # default: False
    frozen=False,   # default: False - make immutable
)
class Product:
    name: str
    price: float
    quantity: int

# ตัวอย่างกับ order=True
@dataclass(order=True)
class Version:
    major: int
    minor: int
    patch: int

v1 = Version(1, 2, 3)
v2 = Version(1, 3, 0)
v3 = Version(2, 0, 0)

print(v1 < v2)   # True (1.2.3 < 1.3.0)
print(v2 < v3)   # True (1.3.0 < 2.0.0)

versions = [v3, v1, v2]
print(sorted(versions))
# [Version(major=1, minor=2, patch=3), Version(major=1, minor=3, patch=0), Version(major=2, minor=0, patch=0)]
```

---

## 3. field() Function

`field()` ใช้สำหรับกำหนด metadata และพฤติกรรมพิเศษของ field แต่ละตัว

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Student:
    name: str
    student_id: str
    # field ที่ไม่แสดงใน repr
    _password: str = field(repr=False)
    # field ที่ไม่ใช้ในการ compare
    login_count: int = field(default=0, compare=False)
    # field ที่ไม่มีใน __init__
    grade_point_average: float = field(default=0.0, init=False)
    
s = Student("Alice", "STU001", "secret123")
print(s)
# Student(name='Alice', student_id='STU001', login_count=0, grade_point_average=0.0)
# สังเกตว่า _password ไม่แสดงใน repr
```

```python
from dataclasses import dataclass, field

@dataclass
class Config:
    host: str
    port: int = field(default=8080)
    debug: bool = field(default=False)
    # metadata คือข้อมูลเพิ่มเติมที่ไม่ใช่ค่า field
    api_key: str = field(
        default="",
        repr=False,
        metadata={"description": "API authentication key", "required": True}
    )

c = Config("localhost")
print(c)
# Config(host='localhost', port=8080, debug=False)

# เข้าถึง metadata ผ่าน fields()
import dataclasses
for f in dataclasses.fields(Config):
    print(f"{f.name}: {f.metadata}")
```

---

## 4. Default Values และ default_factory

### Simple Default Values

```python
from dataclasses import dataclass, field

@dataclass
class Server:
    host: str
    port: int = 80        # default value แบบง่าย
    ssl: bool = False
    timeout: float = 30.0

s1 = Server("example.com")
print(s1)  # Server(host='example.com', port=80, ssl=False, timeout=30.0)

s2 = Server("example.com", 443, True)
print(s2)  # Server(host='example.com', port=443, ssl=True, timeout=30.0)
```

### default_factory สำหรับ Mutable Defaults

**ปัญหา**: ใน Python ปกติ เราไม่สามารถใช้ mutable objects เป็น default value ได้

```python
# ปัญหาของ mutable default ใน Python ทั่วไป
class BadExample:
    def __init__(self, items=[]):  # DANGER! shared between instances
        self.items = items

a = BadExample()
b = BadExample()
a.items.append(1)
print(b.items)  # [1] -- bug! b.items ถูกเปลี่ยนด้วย
```

```python
from dataclasses import dataclass, field
from typing import List, Dict, Set

@dataclass
class ShoppingCart:
    user_id: str
    # ใช้ default_factory สำหรับ mutable defaults
    items: List[str] = field(default_factory=list)
    metadata: Dict[str, str] = field(default_factory=dict)
    tags: Set[str] = field(default_factory=set)
    
cart1 = ShoppingCart("user1")
cart2 = ShoppingCart("user2")

cart1.items.append("apple")
print(cart1.items)  # ['apple']
print(cart2.items)  # []  -- ไม่ถูกกระทบ!
```

```python
from dataclasses import dataclass, field
import datetime

def get_current_time():
    """Factory function สำหรับ default value ที่ต้องคำนวณ"""
    return datetime.datetime.now()

@dataclass
class LogEntry:
    message: str
    level: str = "INFO"
    # timestamp จะถูก set ตอนสร้าง object แต่ละตัว
    timestamp: datetime.datetime = field(default_factory=get_current_time)
    
import time
log1 = LogEntry("First message")
time.sleep(0.01)
log2 = LogEntry("Second message")

print(log1.timestamp != log2.timestamp)  # True - แต่ละ instance มี timestamp ต่างกัน
```

---

## 5. Frozen Dataclasses (Immutable)

Frozen dataclass ทำให้ instances ไม่สามารถเปลี่ยนค่าได้หลังจากสร้าง เหมือน `namedtuple`

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ImmutablePoint:
    x: float
    y: float

p = ImmutablePoint(1.0, 2.0)
print(p)  # ImmutablePoint(x=1.0, y=2.0)

# พยายามเปลี่ยนค่า จะ raise FrozenInstanceError
try:
    p.x = 5.0
except Exception as e:
    print(f"Error: {e}")  # Error: cannot assign to field 'x'
```

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class RGB:
    red: int
    green: int
    blue: int
    
    def to_hex(self) -> str:
        return f"#{self.red:02X}{self.green:02X}{self.blue:02X}"
    
    def blend(self, other: 'RGB') -> 'RGB':
        """สร้าง RGB ใหม่จากการผสมสี - return ใหม่แทนการ modify"""
        return RGB(
            (self.red + other.red) // 2,
            (self.green + other.green) // 2,
            (self.blue + other.blue) // 2
        )

red = RGB(255, 0, 0)
blue = RGB(0, 0, 255)

print(red.to_hex())   # #FF0000
print(blue.to_hex())  # #0000FF

purple = red.blend(blue)
print(purple.to_hex())  # #7F007F

# Frozen dataclass สามารถใช้เป็น dict key หรือใน set ได้
color_set = {red, blue, purple}
color_dict = {red: "red", blue: "blue"}
print(len(color_set))  # 3
```

```python
from dataclasses import dataclass, field
from typing import Tuple

@dataclass(frozen=True)
class Vector3D:
    x: float
    y: float
    z: float
    
    def __add__(self, other: 'Vector3D') -> 'Vector3D':
        return Vector3D(self.x + other.x, self.y + other.y, self.z + other.z)
    
    def __mul__(self, scalar: float) -> 'Vector3D':
        return Vector3D(self.x * scalar, self.y * scalar, self.z * scalar)
    
    def magnitude(self) -> float:
        return (self.x**2 + self.y**2 + self.z**2) ** 0.5
    
    def normalize(self) -> 'Vector3D':
        mag = self.magnitude()
        return Vector3D(self.x / mag, self.y / mag, self.z / mag)

v1 = Vector3D(1, 2, 3)
v2 = Vector3D(4, 5, 6)

print(v1 + v2)       # Vector3D(x=5, y=7, z=9)
print(v1 * 2)        # Vector3D(x=2, y=4, z=6)
print(v1.magnitude()) # 3.7416...
```

---

## 6. Inheritance กับ Dataclasses

```python
from dataclasses import dataclass

@dataclass
class Animal:
    name: str
    species: str
    age: int

@dataclass
class Dog(Animal):
    breed: str
    is_trained: bool = False
    
    def bark(self) -> str:
        return f"{self.name} says: Woof!"

@dataclass  
class ServiceDog(Dog):
    certification: str = ""
    duties: list = None
    
    def __post_init__(self):
        if self.duties is None:
            self.duties = []

# สร้าง instances
dog = Dog(name="Rex", species="Canis lupus familiaris", age=3, breed="German Shepherd")
print(dog)
# Dog(name='Rex', species='Canis lupus familiaris', age=3, breed='German Shepherd', is_trained=False)

service_dog = ServiceDog(
    name="Guide",
    species="Canis lupus familiaris", 
    age=5,
    breed="Labrador",
    is_trained=True,
    certification="Guide Dog",
    duties=["navigation", "obstacle detection"]
)
print(service_dog.bark())  # Guide says: Woof!
```

### ข้อควรระวัง: fields ที่มี default ต้องมาหลัง fields ที่ไม่มี default

```python
from dataclasses import dataclass

@dataclass
class Base:
    name: str
    value: int = 0  # field ที่มี default

# ปัญหา: child class ที่มี field ไม่มี default หลัง field ที่มี default
# @dataclass
# class Child(Base):
#     extra: str  # TypeError! non-default follows default

# วิธีแก้: ใส่ default ให้ child field ด้วย
@dataclass
class Child(Base):
    extra: str = ""  # ใส่ default ให้
    
c = Child("hello", 42, "world")
print(c)  # Child(name='hello', value=42, extra='world')
```

---

## 7. Post-init Processing (__post_init__)

`__post_init__` ถูกเรียกหลังจาก `__init__` auto-generated ทำงานเสร็จ ใช้สำหรับ validation หรือการคำนวณค่าเพิ่มเติม

```python
from dataclasses import dataclass, field
import math

@dataclass
class Circle:
    radius: float
    
    def __post_init__(self):
        # Validation
        if self.radius <= 0:
            raise ValueError(f"Radius must be positive, got {self.radius}")
    
    @property
    def area(self) -> float:
        return math.pi * self.radius ** 2
    
    @property
    def circumference(self) -> float:
        return 2 * math.pi * self.radius

try:
    c1 = Circle(5.0)
    print(f"Area: {c1.area:.2f}")         # Area: 78.54
    print(f"Circumference: {c1.circumference:.2f}")  # Circumference: 31.42

    c2 = Circle(-1.0)  # ValueError
except ValueError as e:
    print(f"Error: {e}")  # Error: Radius must be positive, got -1.0
```

```python
from dataclasses import dataclass, field

@dataclass
class FullName:
    first_name: str
    last_name: str
    # field ที่คำนวณจาก fields อื่น
    full_name: str = field(init=False)
    initials: str = field(init=False)
    
    def __post_init__(self):
        # คำนวณ derived fields
        self.full_name = f"{self.first_name} {self.last_name}"
        self.initials = f"{self.first_name[0]}.{self.last_name[0]}."
        
        # Normalize case
        self.first_name = self.first_name.title()
        self.last_name = self.last_name.title()
        self.full_name = f"{self.first_name} {self.last_name}"

name = FullName("john", "DOE")
print(name.full_name)  # John Doe
print(name.initials)   # J.D.
```

```python
from dataclasses import dataclass, field
from typing import List
import hashlib

@dataclass
class User:
    username: str
    email: str
    _raw_password: str = field(repr=False)
    # computed fields
    password_hash: str = field(init=False, repr=False)
    user_id: str = field(init=False)
    
    def __post_init__(self):
        # Hash password
        self.password_hash = hashlib.sha256(
            self._raw_password.encode()
        ).hexdigest()
        
        # Generate user_id from email
        self.user_id = hashlib.md5(
            self.email.lower().encode()
        ).hexdigest()[:8]
        
        # Validate email
        if "@" not in self.email:
            raise ValueError(f"Invalid email: {self.email}")
    
    def check_password(self, password: str) -> bool:
        return self.password_hash == hashlib.sha256(password.encode()).hexdigest()

user = User("alice", "alice@example.com", "secret123")
print(user)
# User(username='alice', email='alice@example.com', user_id='64e27b0c')
print(user.check_password("secret123"))  # True
print(user.check_password("wrong"))      # False
```

---

## 8. ClassVar ใน Dataclasses

`ClassVar` ใช้สำหรับกำหนด class-level attributes ที่ไม่ถูก include ใน `__init__` หรือ methods ที่ auto-generate

```python
from dataclasses import dataclass, field
from typing import ClassVar

@dataclass
class Employee:
    # ClassVar ไม่ถูก include ใน __init__ หรือ __repr__
    company_name: ClassVar[str] = "TechCorp Inc."
    employee_count: ClassVar[int] = 0
    
    # Instance fields ปกติ
    name: str
    department: str
    salary: float
    
    def __post_init__(self):
        # นับจำนวน employees
        Employee.employee_count += 1
    
    @classmethod
    def get_company_info(cls) -> str:
        return f"{cls.company_name}: {cls.employee_count} employees"

emp1 = Employee("Alice", "Engineering", 80000)
emp2 = Employee("Bob", "Marketing", 70000)
emp3 = Employee("Charlie", "Engineering", 85000)

print(Employee.get_company_info())
# TechCorp Inc.: 3 employees

# ClassVar ไม่อยู่ใน __repr__
print(emp1)
# Employee(name='Alice', department='Engineering', salary=80000)
```

```python
from dataclasses import dataclass
from typing import ClassVar, Dict

@dataclass
class Product:
    # Registry ของ products ทั้งหมด - Class-level
    _registry: ClassVar[Dict[str, 'Product']] = {}
    _valid_categories: ClassVar[set] = {"electronics", "clothing", "food", "books"}
    
    name: str
    price: float
    category: str
    sku: str = ""
    
    def __post_init__(self):
        if self.category not in Product._valid_categories:
            raise ValueError(f"Invalid category: {self.category}")
        
        if not self.sku:
            self.sku = f"{self.category[:3].upper()}-{self.name[:3].upper()}"
        
        Product._registry[self.sku] = self
    
    @classmethod
    def find_by_sku(cls, sku: str) -> 'Product':
        return cls._registry.get(sku)

p1 = Product("Laptop", 999.99, "electronics")
p2 = Product("T-Shirt", 29.99, "clothing")

print(p1.sku)  # ELE-LAP
print(Product.find_by_sku("ELE-LAP"))
# Product(name='Laptop', price=999.99, category='electronics', sku='ELE-LAP')
```

---

## 9. dataclasses.asdict() และ astuple()

### asdict()

```python
from dataclasses import dataclass, asdict, field
from typing import List
import json

@dataclass
class Address:
    street: str
    city: str
    country: str
    postal_code: str

@dataclass
class Contact:
    name: str
    email: str
    phone: str
    address: Address
    tags: List[str] = field(default_factory=list)

contact = Contact(
    name="Alice Johnson",
    email="alice@example.com",
    phone="+1-555-0123",
    address=Address("123 Main St", "Springfield", "USA", "12345"),
    tags=["vip", "premium"]
)

# แปลงเป็น dict (recursive - nested dataclasses ก็ถูกแปลงด้วย)
contact_dict = asdict(contact)
print(contact_dict)
# {
#   'name': 'Alice Johnson',
#   'email': 'alice@example.com',
#   'phone': '+1-555-0123',
#   'address': {'street': '123 Main St', 'city': 'Springfield', 'country': 'USA', 'postal_code': '12345'},
#   'tags': ['vip', 'premium']
# }

# แปลงเป็น JSON ง่ายมาก
json_str = json.dumps(contact_dict, indent=2)
print(json_str)
```

### astuple()

```python
from dataclasses import dataclass, astuple

@dataclass
class Point3D:
    x: float
    y: float
    z: float

p = Point3D(1.0, 2.0, 3.0)
t = astuple(p)
print(t)     # (1.0, 2.0, 3.0)
print(type(t))  # <class 'tuple'>

# ใช้ unpacking
x, y, z = astuple(p)
print(f"x={x}, y={y}, z={z}")  # x=1.0, y=2.0, z=3.0

@dataclass
class Nested:
    point: Point3D
    label: str

n = Nested(Point3D(1, 2, 3), "origin")
print(astuple(n))  # ((1, 2, 3), 'origin') -- nested ก็แปลงด้วย
```

### dataclasses.replace()

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class Config:
    host: str = "localhost"
    port: int = 8080
    debug: bool = False
    max_connections: int = 100

# สร้าง config ใหม่โดย copy และเปลี่ยนบางค่า
prod_config = Config(host="prod.example.com", port=443)
debug_config = replace(prod_config, debug=True, max_connections=10)

print(prod_config)
# Config(host='prod.example.com', port=443, debug=False, max_connections=100)
print(debug_config)
# Config(host='prod.example.com', port=443, debug=True, max_connections=10)
```

---

## 10. dataclasses.fields() และ Introspection

```python
from dataclasses import dataclass, fields, field
from typing import ClassVar

@dataclass
class Schema:
    name: str
    age: int = field(default=0, metadata={"min": 0, "max": 150})
    email: str = field(default="", metadata={"pattern": r".*@.*"})
    _internal: ClassVar[str] = "hidden"

# Introspect fields
for f in fields(Schema):
    print(f"Field: {f.name}")
    print(f"  Type: {f.type}")
    print(f"  Default: {f.default}")
    print(f"  Metadata: {f.metadata}")
    print()
```

```python
from dataclasses import dataclass, fields, is_dataclass

@dataclass
class SimpleModel:
    id: int
    name: str
    value: float = 0.0

# ตรวจสอบว่าเป็น dataclass
print(is_dataclass(SimpleModel))       # True
print(is_dataclass(SimpleModel(1, "x")))  # True

# สร้าง instance จาก dict
def from_dict(cls, data: dict):
    """Generic function สร้าง dataclass จาก dict"""
    field_names = {f.name for f in fields(cls)}
    filtered = {k: v for k, v in data.items() if k in field_names}
    return cls(**filtered)

data = {"id": 1, "name": "test", "value": 3.14, "extra": "ignored"}
model = from_dict(SimpleModel, data)
print(model)  # SimpleModel(id=1, name='test', value=3.14)
```

---

## 11. attrs Library (Alternative)

`attrs` เป็น library ที่เก่ากว่า dataclasses และมีความสามารถมากกว่า

### ติดตั้ง

```bash
pip install attrs
```

### การใช้งานพื้นฐาน

```python
import attr

@attr.s
class Point:
    x = attr.ib(type=float)
    y = attr.ib(type=float)

p = Point(1.0, 2.0)
print(p)   # Point(x=1.0, y=2.0)
print(p == Point(1.0, 2.0))  # True
```

### attrs กับ modern syntax (attrs >= 20.1.0)

```python
from attrs import define, field, Factory

@define
class Person:
    name: str
    age: int
    hobbies: list = Factory(list)
    
    def greet(self) -> str:
        return f"Hi, I'm {self.name}!"

p = Person("Alice", 30)
p.hobbies.append("reading")
print(p)  # Person(name='Alice', age=30, hobbies=['reading'])
print(p.greet())  # Hi, I'm Alice!
```

### attrs validators

```python
from attrs import define, field, validators

@define
class Server:
    host: str = field(validator=validators.instance_of(str))
    port: int = field(
        validator=[
            validators.instance_of(int),
            validators.in_(range(1, 65536))
        ]
    )
    
try:
    s1 = Server("localhost", 8080)
    print(s1)  # Server(host='localhost', port=8080)
    
    s2 = Server("localhost", 99999)  # ValueError
except Exception as e:
    print(f"Error: {e}")
```

```python
from attrs import define, field, validators

def validate_email(instance, attribute, value):
    """Custom validator"""
    if "@" not in value:
        raise ValueError(f"Invalid email: {value}")

@define
class User:
    name: str
    email: str = field(validator=validate_email)
    age: int = field(validator=[
        validators.instance_of(int),
        validators.ge(0),  # >= 0
        validators.lt(150)  # < 150
    ])

try:
    u = User("Alice", "alice@example.com", 30)
    print(u)  # User(name='Alice', email='alice@example.com', age=30)
    
    u2 = User("Bob", "invalid-email", 25)  # ValueError
except ValueError as e:
    print(f"Validation error: {e}")
```

### attrs converters

```python
from attrs import define, field

@define
class Config:
    # converter แปลงค่า input อัตโนมัติ
    host: str = field(converter=str.lower)
    port: int = field(converter=int)
    tags: list = field(converter=list)

c = Config("LOCALHOST", "8080", ("api", "v1"))
print(c.host)   # localhost (แปลงเป็น lowercase)
print(c.port)   # 8080 (แปลงเป็น int)
print(c.tags)   # ['api', 'v1'] (แปลงเป็น list)
print(type(c.port))  # <class 'int'>
```

---

## 12. Pydantic Models (เบื้องต้น)

Pydantic เป็น library สำหรับ data validation โดยใช้ type hints Python

### ติดตั้ง

```bash
pip install pydantic
```

### Pydantic v2 (BaseModel)

```python
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
from datetime import datetime

class Address(BaseModel):
    street: str
    city: str
    country: str = "Thailand"
    postal_code: str

class UserProfile(BaseModel):
    id: int
    username: str
    email: str
    age: int = Field(ge=0, le=150)
    bio: Optional[str] = None
    address: Optional[Address] = None
    created_at: datetime = Field(default_factory=datetime.now)
    
    class Config:
        # อนุญาตให้ใช้ arbitrary types
        arbitrary_types_allowed = True

# Pydantic validate ข้อมูลอัตโนมัติ
user = UserProfile(
    id=1,
    username="alice",
    email="alice@example.com",
    age=30,
    address={"street": "123 Main St", "city": "Bangkok", "postal_code": "10100"}
)
print(user)
print(user.address.city)  # Bangkok

# Serialize เป็น dict
print(user.model_dump())

# Serialize เป็น JSON
print(user.model_dump_json())
```

```python
from pydantic import BaseModel, validator, Field
from typing import List

class Product(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)
    quantity: int = Field(ge=0)
    tags: List[str] = []
    
    @validator('name')
    def name_must_not_be_empty(cls, v):
        return v.strip()
    
    @validator('tags')
    def tags_must_be_lowercase(cls, v):
        return [tag.lower() for tag in v]
    
    @property
    def total_value(self) -> float:
        return self.price * self.quantity

try:
    p = Product(name="  Laptop  ", price=999.99, quantity=10, tags=["Electronics", "SALE"])
    print(p.name)        # Laptop (stripped)
    print(p.tags)        # ['electronics', 'sale'] (lowercased)
    print(p.total_value) # 9999.9
    
    invalid = Product(name="", price=-10, quantity=5)
except Exception as e:
    print(f"Validation error: {e}")
```

---

## 13. Comparison: dataclass vs TypedDict vs NamedTuple

### TypedDict

```python
from typing import TypedDict, Optional

class PersonDict(TypedDict):
    name: str
    age: int
    email: Optional[str]

# TypedDict ใช้สำหรับ type hint dict เท่านั้น
# ไม่สร้าง object จริง
person: PersonDict = {"name": "Alice", "age": 30, "email": None}
print(person["name"])  # Alice

# ไม่สามารถเพิ่ม methods ได้
# ไม่มี validation runtime
# เหมาะสำหรับ JSON-like data structures
```

### NamedTuple

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
    label: str = ""

p = Point(1.0, 2.0, "origin")
print(p)         # Point(x=1.0, y=2.0, label='origin')
print(p.x)       # 1.0
print(p[0])      # 1.0 (ใช้ index ได้)
print(len(p))    # 3

# Immutable
try:
    p.x = 5.0  # AttributeError
except AttributeError as e:
    print(f"Error: {e}")

# Unpack ได้เหมือน tuple
x, y, label = p
print(f"x={x}, y={y}")  # x=1.0, y=2.0
```

### ตาราง Comparison

```python
"""
Feature Comparison:

                    | dataclass | TypedDict | NamedTuple | attrs | pydantic
--------------------|-----------|-----------|------------|-------|----------
Type checking       |     ✓     |     ✓     |     ✓      |   ✓   |    ✓
Runtime validation  |     ✗     |     ✗     |     ✗      |   ✓   |    ✓
Mutable             |     ✓     |     ✓     |     ✗      |   ✓   |    ✗*
Immutable option    |     ✓     |     ✗     |     ✓      |   ✓   |    ✓
Dict-like access    |     ✗     |     ✓     |     ✗      |   ✗   |    ✓
Tuple-like access   |     ✗     |     ✗     |     ✓      |   ✗   |    ✗
Methods             |     ✓     |     ✗     |     ✓      |   ✓   |    ✓
Inheritance         |     ✓     |     ✓     |     ✓      |   ✓   |    ✓
Serialization       |    ~~     |     ✓     |    ~~      |  ~~   |    ✓
Performance         |     ✓     |     ✓     |     ✓✓    |   ✓   |   ~~
Dependencies        |     ✗     |     ✗     |     ✗      |   ✓   |    ✓

* pydantic v2 BaseModel เป็น immutable โดย default สามารถเปลี่ยนได้ด้วย model_config
"""

# เมื่อไหรควรใช้อะไร:
"""
dataclass:
  - เหมาะสำหรับ domain objects ทั่วไป
  - ต้องการ mutable objects ที่มี methods
  - ไม่ต้องการ dependency เพิ่ม
  - ต้องการ frozen (immutable) option

TypedDict:
  - เหมาะสำหรับ type-hint dict structures
  - ทำงานกับ JSON API responses
  - ต้องการความยืดหยุ่นเหมือน dict

NamedTuple:
  - ต้องการ immutable value objects
  - ต้องการ tuple interface (unpacking, indexing)
  - Performance-critical code

attrs:
  - ต้องการ validators ที่ซับซ้อน
  - ต้องการ converters
  - ต้องการ ordering และ hashing ที่ customize ได้

Pydantic:
  - ต้องการ runtime validation ที่ครอบคลุม
  - ทำงานกับ API inputs/outputs
  - FastAPI integration
  - ต้องการ JSON serialization ที่ดี
"""
```

---

## 14. Advanced Patterns

### Dataclass ที่ support JSON serialization

```python
from dataclasses import dataclass, asdict, field, fields
from typing import List, Dict, Any, Type, TypeVar
import json

T = TypeVar('T')

@dataclass
class JsonMixin:
    """Mixin ที่เพิ่ม JSON support ให้ dataclass"""
    
    def to_json(self) -> str:
        return json.dumps(asdict(self), default=str)
    
    @classmethod
    def from_json(cls: Type[T], json_str: str) -> T:
        data = json.loads(json_str)
        return cls(**data)
    
    def to_dict(self) -> Dict[str, Any]:
        return asdict(self)

@dataclass
class Config(JsonMixin):
    host: str
    port: int
    debug: bool = False

c = Config("localhost", 8080, True)
json_str = c.to_json()
print(json_str)  # {"host": "localhost", "port": 8080, "debug": true}

c2 = Config.from_json(json_str)
print(c2)  # Config(host='localhost', port=8080, debug=True)
print(c == c2)  # True
```

### Dataclass Factory Pattern

```python
from dataclasses import dataclass, field
from typing import Dict, Any, Type, TypeVar
import copy

T = TypeVar('T')

@dataclass
class Template:
    """Template dataclass ที่ใช้สร้าง instances จาก defaults"""
    _defaults: Dict[str, Any] = field(default_factory=dict, repr=False, init=False)
    
    def register_default(self, name: str, value: Any):
        self._defaults[name] = value
    
    def create_from_template(self: T, **overrides) -> T:
        data = copy.deepcopy(self._defaults)
        data.update(overrides)
        return type(self)(**{
            k: v for k, v in data.items() 
            if k != '_defaults'
        })

@dataclass
class EmailConfig:
    smtp_host: str = "smtp.gmail.com"
    smtp_port: int = 587
    use_tls: bool = True
    timeout: int = 30

# สร้าง config สำหรับ environments ต่างๆ
default_config = EmailConfig()
dev_config = EmailConfig(smtp_host="localhost", smtp_port=1025, use_tls=False)
prod_config = EmailConfig(smtp_host="smtp.company.com", timeout=60)

print(dev_config)
# EmailConfig(smtp_host='localhost', smtp_port=1025, use_tls=False, timeout=30)
```

---

## 15. Practical Example: Database ORM-like Pattern

```python
from dataclasses import dataclass, field, fields, asdict
from typing import ClassVar, Dict, List, Optional, Any
import sqlite3

@dataclass
class Model:
    """Base class สำหรับ database models"""
    TABLE_NAME: ClassVar[str] = ""
    id: Optional[int] = field(default=None, init=False)
    
    @classmethod
    def create_table(cls, conn: sqlite3.Connection):
        field_defs = []
        for f in fields(cls):
            if f.name == 'id':
                field_defs.append("id INTEGER PRIMARY KEY AUTOINCREMENT")
            elif f.name.startswith('_') or f.name == 'TABLE_NAME':
                continue
            elif f.type == 'str' or f.type == str:
                field_defs.append(f"{f.name} TEXT")
            elif f.type == 'int' or f.type == int:
                field_defs.append(f"{f.name} INTEGER")
            elif f.type == 'float' or f.type == float:
                field_defs.append(f"{f.name} REAL")
        
        sql = f"CREATE TABLE IF NOT EXISTS {cls.TABLE_NAME} ({', '.join(field_defs)})"
        conn.execute(sql)
        conn.commit()
    
    def save(self, conn: sqlite3.Connection):
        data = asdict(self)
        data.pop('id', None)
        
        if self.id is None:
            placeholders = ', '.join(['?'] * len(data))
            columns = ', '.join(data.keys())
            sql = f"INSERT INTO {self.TABLE_NAME} ({columns}) VALUES ({placeholders})"
            cursor = conn.execute(sql, list(data.values()))
            self.id = cursor.lastrowid
        else:
            sets = ', '.join([f"{k}=?" for k in data.keys()])
            sql = f"UPDATE {self.TABLE_NAME} SET {sets} WHERE id=?"
            conn.execute(sql, list(data.values()) + [self.id])
        conn.commit()

@dataclass
class Article(Model):
    TABLE_NAME: ClassVar[str] = "articles"
    title: str = ""
    content: str = ""
    author: str = ""

# ใช้งาน
conn = sqlite3.connect(":memory:")
Article.create_table(conn)

article = Article(title="Python Dataclasses", content="...", author="Alice")
article.save(conn)
print(f"Saved article with id: {article.id}")  # Saved article with id: 1
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง BankAccount dataclass

สร้าง `BankAccount` dataclass ที่มี:
- `account_number: str`
- `owner: str`
- `balance: float = 0.0`
- `transactions: List[float]` (default empty list)
- `__post_init__` ตรวจสอบ balance ไม่เป็นลบ
- methods: `deposit()`, `withdraw()`, `get_statement()`

```python
# เฉลย
from dataclasses import dataclass, field
from typing import List

@dataclass
class BankAccount:
    account_number: str
    owner: str
    balance: float = 0.0
    transactions: List[float] = field(default_factory=list)
    
    def __post_init__(self):
        if self.balance < 0:
            raise ValueError("Initial balance cannot be negative")
    
    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
        self.transactions.append(amount)
    
    def withdraw(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        self.transactions.append(-amount)
    
    def get_statement(self) -> str:
        lines = [f"Account: {self.account_number}", f"Owner: {self.owner}"]
        for i, t in enumerate(self.transactions, 1):
            prefix = "+" if t > 0 else ""
            lines.append(f"  {i}. {prefix}{t:.2f}")
        lines.append(f"Balance: {self.balance:.2f}")
        return "\n".join(lines)

acc = BankAccount("ACC001", "Alice", 1000.0)
acc.deposit(500.0)
acc.withdraw(200.0)
print(acc.get_statement())
```

### แบบฝึกหัดที่ 2: Frozen dataclass สำหรับ coordinate system

สร้าง frozen dataclass `GeoCoordinate` ที่:
- มี `latitude: float` และ `longitude: float`
- validate ว่า latitude อยู่ระหว่าง -90 ถึง 90
- validate ว่า longitude อยู่ระหว่าง -180 ถึง 180
- มี method `distance_to()` คำนวณระยะห่างระหว่าง 2 จุด (Haversine formula)

```python
# เฉลย
from dataclasses import dataclass
import math

@dataclass(frozen=True)
class GeoCoordinate:
    latitude: float
    longitude: float
    
    def __post_init__(self):
        if not -90 <= self.latitude <= 90:
            raise ValueError(f"Latitude must be between -90 and 90, got {self.latitude}")
        if not -180 <= self.longitude <= 180:
            raise ValueError(f"Longitude must be between -180 and 180, got {self.longitude}")
    
    def distance_to(self, other: 'GeoCoordinate') -> float:
        """คำนวณระยะห่างเป็นกิโลเมตรด้วย Haversine formula"""
        R = 6371  # Earth radius in km
        
        lat1, lon1 = math.radians(self.latitude), math.radians(self.longitude)
        lat2, lon2 = math.radians(other.latitude), math.radians(other.longitude)
        
        dlat = lat2 - lat1
        dlon = lon2 - lon1
        
        a = math.sin(dlat/2)**2 + math.cos(lat1) * math.cos(lat2) * math.sin(dlon/2)**2
        c = 2 * math.asin(math.sqrt(a))
        
        return R * c

bangkok = GeoCoordinate(13.7563, 100.5018)
tokyo = GeoCoordinate(35.6762, 139.6503)
print(f"Distance: {bangkok.distance_to(tokyo):.0f} km")
# Distance: 4609 km
```

### แบบฝึกหัดที่ 3: Dataclass inheritance สำหรับ shapes

```python
# สร้าง hierarchy ของ shapes:
# Shape (base) -> Circle, Rectangle, Triangle
# แต่ละ shape มี area() และ perimeter()

from dataclasses import dataclass
import math

@dataclass
class Shape:
    color: str = "black"
    
    def area(self) -> float:
        raise NotImplementedError
    
    def perimeter(self) -> float:
        raise NotImplementedError
    
    def describe(self) -> str:
        return (f"{type(self).__name__}(color={self.color}, "
                f"area={self.area():.2f}, perimeter={self.perimeter():.2f})")

@dataclass
class Circle(Shape):
    radius: float = 0.0
    
    def area(self) -> float:
        return math.pi * self.radius ** 2
    
    def perimeter(self) -> float:
        return 2 * math.pi * self.radius

@dataclass
class Rectangle(Shape):
    width: float = 0.0
    height: float = 0.0
    
    def area(self) -> float:
        return self.width * self.height
    
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)

@dataclass
class Triangle(Shape):
    a: float = 0.0
    b: float = 0.0
    c: float = 0.0
    
    def __post_init__(self):
        if not (self.a + self.b > self.c and 
                self.b + self.c > self.a and 
                self.a + self.c > self.b):
            raise ValueError("Invalid triangle sides")
    
    def area(self) -> float:
        s = self.perimeter() / 2
        return math.sqrt(s * (s-self.a) * (s-self.b) * (s-self.c))
    
    def perimeter(self) -> float:
        return self.a + self.b + self.c

shapes = [
    Circle("red", 5.0),
    Rectangle("blue", 4.0, 6.0),
    Triangle("green", 3.0, 4.0, 5.0)
]

for shape in shapes:
    print(shape.describe())
```

### แบบฝึกหัดที่ 4: Config system ด้วย dataclasses

```python
# สร้าง config system ที่มี:
# - DatabaseConfig, CacheConfig, AppConfig
# - สามารถ load จาก dict ได้
# - สามารถ validate ได้
# - มี default values ที่สมเหตุสมผล

from dataclasses import dataclass, field, asdict
from typing import Optional, Dict, Any

@dataclass
class DatabaseConfig:
    host: str = "localhost"
    port: int = 5432
    name: str = "app_db"
    user: str = "postgres"
    password: str = field(default="", repr=False)
    pool_size: int = 10
    
    def __post_init__(self):
        if not 1 <= self.port <= 65535:
            raise ValueError(f"Invalid port: {self.port}")

@dataclass
class CacheConfig:
    backend: str = "redis"
    host: str = "localhost"
    port: int = 6379
    ttl: int = 3600
    max_connections: int = 50

@dataclass
class AppConfig:
    name: str = "MyApp"
    version: str = "1.0.0"
    debug: bool = False
    database: DatabaseConfig = field(default_factory=DatabaseConfig)
    cache: CacheConfig = field(default_factory=CacheConfig)
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'AppConfig':
        db_data = data.pop('database', {})
        cache_data = data.pop('cache', {})
        
        config = cls(**data)
        if db_data:
            config.database = DatabaseConfig(**db_data)
        if cache_data:
            config.cache = CacheConfig(**cache_data)
        return config

# ตัวอย่างการใช้งาน
config_data = {
    "name": "ProductionApp",
    "debug": False,
    "database": {
        "host": "db.example.com",
        "name": "prod_db",
        "password": "secretpassword"
    }
}

config = AppConfig.from_dict(config_data)
print(config.name)              # ProductionApp
print(config.database.host)    # db.example.com
print(config.cache.backend)    # redis (default)
```

### แบบฝึกหัดที่ 5: Event system ด้วย frozen dataclasses

```python
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any, Dict

@dataclass(frozen=True)
class Event:
    """Immutable event object"""
    event_type: str
    data: Dict[str, Any] = field(default_factory=dict)
    timestamp: datetime = field(default_factory=datetime.now)
    source: str = "system"
    
    def to_log_format(self) -> str:
        return f"[{self.timestamp.isoformat()}] {self.event_type} from {self.source}: {self.data}"

# ตัวอย่าง events
user_registered = Event(
    event_type="user.registered",
    data={"user_id": 123, "email": "alice@example.com"},
    source="auth_service"
)

order_placed = Event(
    event_type="order.placed", 
    data={"order_id": 456, "amount": 99.99},
    source="order_service"
)

print(user_registered.to_log_format())
print(order_placed.to_log_format())

# สามารถใช้เป็น dict key เพราะ frozen
event_handlers = {
    "user.registered": lambda e: print(f"Welcome! {e.data}"),
    "order.placed": lambda e: print(f"Processing order: {e.data}")
}

for event in [user_registered, order_placed]:
    handler = event_handlers.get(event.event_type)
    if handler:
        handler(event)
```

### แบบฝึกหัดที่ 6: Comparison ของ data containers

```python
# เขียน class เดียวกันด้วย 4 วิธีแล้วเปรียบเทียบ:
# dataclass, TypedDict, NamedTuple, dict ปกติ

from dataclasses import dataclass
from typing import TypedDict, NamedTuple, Optional

# Scenario: Product catalog item

# 1. dataclass
@dataclass
class ProductDC:
    id: int
    name: str
    price: float
    category: str
    in_stock: bool = True
    
    def apply_discount(self, percent: float) -> 'ProductDC':
        from dataclasses import replace
        return replace(self, price=self.price * (1 - percent/100))

# 2. TypedDict
class ProductTD(TypedDict):
    id: int
    name: str
    price: float
    category: str
    in_stock: bool

# 3. NamedTuple
class ProductNT(NamedTuple):
    id: int
    name: str
    price: float
    category: str
    in_stock: bool = True

# เปรียบเทียบการใช้งาน
dc = ProductDC(1, "Laptop", 999.99, "electronics")
td: ProductTD = {"id": 1, "name": "Laptop", "price": 999.99, "category": "electronics", "in_stock": True}
nt = ProductNT(1, "Laptop", 999.99, "electronics")

print("dataclass:")
print(f"  Access: {dc.name}, Method: {dc.apply_discount(10).price:.2f}")

print("TypedDict:")
print(f"  Access: {td['name']}")

print("NamedTuple:")
print(f"  Access: {nt.name}, Index: {nt[1]}")
print(f"  Unpack: ", end="")
id_, name, *rest = nt
print(f"id={id_}, name={name}")
```

### แบบฝึกหัดที่ 7: Dataclass with slots (Python 3.10+)

```python
# Python 3.10+ รองรับ slots=True ใน @dataclass
# ซึ่งทำให้ memory efficient ยิ่งขึ้น

from dataclasses import dataclass
import sys

@dataclass
class PointNormal:
    x: float
    y: float
    z: float

@dataclass(slots=True)  # Python 3.10+
class PointSlotted:
    x: float
    y: float
    z: float

# เปรียบเทียบ memory usage
normal = PointNormal(1.0, 2.0, 3.0)
# slotted = PointSlotted(1.0, 2.0, 3.0)

print(f"Normal point size: {sys.getsizeof(normal.__dict__)} bytes for __dict__")
# Slotted version ไม่มี __dict__ จึงใช้ memory น้อยกว่า

# ตัวอย่างกับ Python 3.9 (ไม่รองรับ slots=True)
# ใช้ __slots__ แทน
class EfficientPoint:
    __slots__ = ['x', 'y', 'z']
    
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z
    
    def __repr__(self):
        return f"EfficientPoint(x={self.x}, y={self.y}, z={self.z})"

ep = EfficientPoint(1.0, 2.0, 3.0)
print(ep)
```

### แบบฝึกหัดที่ 8: Pydantic model สำหรับ API

```python
from pydantic import BaseModel, Field, validator
from typing import List, Optional
from datetime import datetime
import uuid

class CreateUserRequest(BaseModel):
    username: str = Field(min_length=3, max_length=50)
    email: str
    password: str = Field(min_length=8)
    age: Optional[int] = Field(None, ge=13, le=120)
    
    @validator('email')
    def validate_email(cls, v):
        if '@' not in v or '.' not in v.split('@')[1]:
            raise ValueError('Invalid email format')
        return v.lower()
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v

class UserResponse(BaseModel):
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    username: str
    email: str
    age: Optional[int] = None
    created_at: datetime = Field(default_factory=datetime.now)
    is_active: bool = True
    
    class Config:
        json_encoders = {
            datetime: lambda v: v.isoformat()
        }

# Test API flow
try:
    # Valid request
    request = CreateUserRequest(
        username="alice_dev",
        email="Alice@EXAMPLE.COM",
        password="SecurePass1",
        age=25
    )
    print("Valid request:", request)
    
    # Create response
    response = UserResponse(
        username=request.username,
        email=request.email,
        age=request.age
    )
    print("Response JSON:", response.model_dump_json(indent=2))
    
except Exception as e:
    print(f"Validation error: {e}")

# Invalid request
try:
    invalid = CreateUserRequest(
        username="a",  # too short
        email="notanemail",
        password="weak"
    )
except Exception as e:
    print(f"\nInvalid request errors: {e}")
```

---

## สรุป

| หัวข้อ | Key Points |
|--------|-----------|
| `@dataclass` | Auto-generate `__init__`, `__repr__`, `__eq__` |
| `field()` | กำหนด metadata, default, repr, compare |
| `default_factory` | สำหรับ mutable defaults (list, dict) |
| `frozen=True` | สร้าง immutable dataclass |
| `__post_init__` | ทำงานหลัง `__init__` สำหรับ validation |
| `ClassVar` | Class-level attributes ที่ไม่ใช่ instance fields |
| `asdict()/astuple()` | แปลง dataclass เป็น dict/tuple |
| `attrs` | Library ทางเลือกที่มี validators/converters |
| `Pydantic` | Runtime validation + serialization |

**เมื่อไหรควรใช้อะไร:**
- **dataclass**: โปรแกรมทั่วไป ไม่ต้องการ dependencies เพิ่ม
- **attrs**: ต้องการ validators/converters ที่ยืดหยุ่น
- **Pydantic**: API development, FastAPI, ต้องการ validation เข้มงวด
- **TypedDict**: type-hint สำหรับ dict structures
- **NamedTuple**: immutable value objects ที่ต้องการ tuple interface
