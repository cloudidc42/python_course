# Part 43: Metaclasses & Class Creation

## บทนำ

Metaclass เป็นหนึ่งในคุณสมบัติที่ทรงพลังและน่าสับสนที่สุดของ Python เมื่อเราสร้าง class ในภาษา Python เรากำลังสร้าง object ชนิดหนึ่ง ซึ่ง class นั้นเองก็เป็น instance ของ metaclass

คำพูดที่มักถูกอ้างถึง: **"In Python, everything is an object, including classes"**

---

## 1. type() Function สำหรับสร้าง Class

`type()` มีสองรูปแบบการใช้งาน:
1. `type(obj)` - คืน type ของ object
2. `type(name, bases, dict)` - สร้าง class ใหม่

```python
# type() แบบ 1 argument - ดู type
x = 42
s = "hello"
lst = [1, 2, 3]

print(type(x))    # <class 'int'>
print(type(s))    # <class 'str'>
print(type(lst))  # <class 'list'>

class MyClass:
    pass

obj = MyClass()
print(type(obj))      # <class '__main__.MyClass'>
print(type(MyClass))  # <class 'type'>  <-- MyClass เป็น instance ของ type!
```

```python
# type() แบบ 3 arguments - สร้าง class
# type(name, bases, namespace)

# การสร้าง class แบบปกติ
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return f"{self.name} makes a sound"

# เทียบเท่ากับ type()
Animal2 = type(
    'Animal2',           # ชื่อ class
    (),                  # base classes (tuple ว่างหมายถึง inherit จาก object)
    {                    # class namespace (dict)
        '__init__': lambda self, name: setattr(self, 'name', name),
        'speak': lambda self: f"{self.name} makes a sound"
    }
)

a1 = Animal("Rex")
a2 = Animal2("Max")

print(a1.speak())  # Rex makes a sound
print(a2.speak())  # Max makes a sound
print(type(a1))    # <class '__main__.Animal'>
print(type(a2))    # <class '__main__.Animal2'>
```

```python
# สร้าง class ที่มี inheritance ด้วย type()
class Vehicle:
    def __init__(self, speed: int):
        self.speed = speed
    
    def describe(self) -> str:
        return f"Vehicle with speed {self.speed}"

# สร้าง Car ที่ inherit จาก Vehicle
Car = type(
    'Car',
    (Vehicle,),  # base classes
    {
        '__init__': lambda self, speed, brand: (
            Vehicle.__init__(self, speed),
            setattr(self, 'brand', brand)
        ),
        'describe': lambda self: f"{self.brand} car, speed {self.speed}",
        'max_passengers': 5,  # class attribute
    }
)

car = Car(100, "Toyota")
print(car.describe())  # Toyota car, speed 100
print(car.max_passengers)  # 5
print(isinstance(car, Vehicle))  # True
```

---

## 2. Metaclass Concept

Metaclass คือ "class ของ class" - มันควบคุมว่า class ถูกสร้างขึ้นอย่างไร

```
┌─────────────────────────────────────────────────┐
│                    type                          │
│              (metaclass of all)                  │
│  creates →  MyClass  →  instances of MyClass     │
└─────────────────────────────────────────────────┘

type       → metaclass  (สร้าง class จาก class definition)
MyClass    → class      (สร้าง instances)
instances  → objects    (ทำงานจริง)
```

```python
# ตรวจสอบ metaclass hierarchy
print(type(int))    # <class 'type'>
print(type(str))    # <class 'type'>
print(type(list))   # <class 'type'>
print(type(type))   # <class 'type'>  <- type เป็น metaclass ของตัวเอง!

# class ทุกตัวเป็น instance ของ type (โดยค่า default)
class MyClass:
    pass

print(isinstance(MyClass, type))  # True
print(isinstance(int, type))      # True

# ตรวจสอบ MRO (Method Resolution Order)
print(int.__mro__)
# (<class 'int'>, <class 'object'>)
print(type.__mro__)
# (<class 'type'>, <class 'object'>)
```

---

## 3. __new__ vs __init__

```python
# __new__: สร้าง instance (ก่อน initialization)
# __init__: initialize instance (หลังจากสร้างแล้ว)

class MyClass:
    def __new__(cls, *args, **kwargs):
        print(f"__new__ called with cls={cls}")
        # ต้อง return instance จาก super().__new__()
        instance = super().__new__(cls)
        print(f"Instance created: {instance}")
        return instance
    
    def __init__(self, value: int):
        print(f"__init__ called with self={self}, value={value}")
        self.value = value

obj = MyClass(42)
# Output:
# __new__ called with cls=<class '__main__.MyClass'>
# Instance created: <__main__.MyClass object at 0x...>
# __init__ called with self=<__main__.MyClass object at 0x...>, value=42
```

```python
# __new__ ใช้สำหรับ:
# 1. Immutable types (เพราะต้องกำหนดค่าก่อน return)
# 2. Singleton pattern
# 3. Factory methods

class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            print("Creating new instance")
            cls._instance = super().__new__(cls)
        else:
            print("Returning existing instance")
        return cls._instance
    
    def __init__(self):
        pass  # ถูกเรียกทุกครั้งที่ Singleton() ถึงแม้จะ return instance เดิม

s1 = Singleton()  # Creating new instance
s2 = Singleton()  # Returning existing instance
s3 = Singleton()  # Returning existing instance

print(s1 is s2)  # True
print(s2 is s3)  # True
```

```python
# __new__ กับ immutable types
class CustomInt(int):
    """int ที่มี label"""
    
    def __new__(cls, value: int, label: str = ""):
        # int เป็น immutable ต้องใช้ __new__ กำหนดค่า
        instance = super().__new__(cls, value)
        instance.label = label
        return instance
    
    def __repr__(self) -> str:
        return f"CustomInt({int(self)}, label={self.label!r})"

n = CustomInt(42, "answer")
print(n)         # CustomInt(42, label='answer')
print(n + 1)     # 43 (ยังทำงานเหมือน int)
print(n.label)   # answer
print(type(n))   # <class '__main__.CustomInt'>
```

---

## 4. Custom Metaclass

```python
# สร้าง metaclass โดย inherit จาก type
class LoggingMeta(type):
    """Metaclass ที่ log การสร้าง class"""
    
    def __new__(mcs, name, bases, namespace):
        print(f"Creating class: {name}")
        print(f"  Bases: {bases}")
        print(f"  Attributes: {list(namespace.keys())}")
        
        cls = super().__new__(mcs, name, bases, namespace)
        return cls
    
    def __init__(cls, name, bases, namespace):
        print(f"Initializing class: {name}")
        super().__init__(name, bases, namespace)

class MyClass(metaclass=LoggingMeta):
    class_var = "hello"
    
    def method(self):
        pass

# Output:
# Creating class: MyClass
#   Bases: ()
#   Attributes: ['__module__', '__qualname__', 'class_var', 'method']
# Initializing class: MyClass
```

```python
class ValidatingMeta(type):
    """Metaclass ที่ validate class definition"""
    
    def __new__(mcs, name, bases, namespace):
        # ตรวจสอบว่า class มี docstring
        if '__doc__' not in namespace or not namespace['__doc__']:
            raise TypeError(f"Class {name} must have a docstring")
        
        # ตรวจสอบว่า methods ทั้งหมดมี type hints
        for attr_name, attr_value in namespace.items():
            if callable(attr_value) and not attr_name.startswith('_'):
                if not hasattr(attr_value, '__annotations__'):
                    pass  # ข้าม
        
        return super().__new__(mcs, name, bases, namespace)

class GoodClass(metaclass=ValidatingMeta):
    """This is a well-documented class."""
    
    def compute(self, x: int) -> int:
        return x * 2

try:
    class BadClass(metaclass=ValidatingMeta):
        pass  # ไม่มี docstring
except TypeError as e:
    print(f"Error: {e}")  # Error: Class BadClass must have a docstring
```

---

## 5. __class__ Attribute

```python
# __class__ ใช้เข้าถึง class ของ instance
class Animal:
    def describe(self) -> str:
        return f"I am a {self.__class__.__name__}"
    
    @classmethod
    def create(cls) -> 'Animal':
        # cls คือ class ที่ถูกเรียก (ไม่ใช่เสมอ Animal)
        return cls()

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.__class__.__name__} says Woof!"

class Cat(Animal):
    def speak(self) -> str:
        return f"{self.__class__.__name__} says Meow!"

dog = Dog()
cat = Cat()

print(dog.describe())  # I am a Dog
print(cat.describe())  # I am a Cat
print(dog.speak())     # Dog says Woof!

# classmethod ใช้ cls ไม่ใช่ class ที่กำหนด
dog2 = Dog.create()
print(type(dog2))  # <class '__main__.Dog'>  -- ถูกต้อง!
```

```python
# __class__ ใน super()
class Base:
    def method(self):
        print(f"Base.method() - class is {self.__class__.__name__}")
        print(f"  __class__ in Base.method: {__class__.__name__}")

class Child(Base):
    def method(self):
        print(f"Child.method() - class is {self.__class__.__name__}")
        super().method()  # super() ใช้ __class__ implicitly

c = Child()
c.method()
# Child.method() - class is Child
# Base.method() - class is Child
#   __class__ in Base.method: Base
```

---

## 6. Class Decorators vs Metaclasses

### Class Decorators

```python
from typing import Type, TypeVar
from functools import wraps

T = TypeVar('T')

def singleton(cls: Type[T]) -> Type[T]:
    """Class decorator สำหรับ Singleton pattern"""
    instances = {}
    
    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Database:
    def __init__(self, url: str = "sqlite:///:memory:"):
        self.url = url
        print(f"Connecting to {url}")

db1 = Database("postgres://localhost/mydb")
db2 = Database()  # ไม่ connect ใหม่

print(db1 is db2)  # True
```

```python
def validate_types(cls):
    """Class decorator ที่เพิ่ม runtime type validation"""
    
    original_init = cls.__init__
    hints = cls.__init__.__annotations__ if hasattr(cls.__init__, '__annotations__') else {}
    
    def new_init(self, *args, **kwargs):
        # Validate argument types
        import inspect
        sig = inspect.signature(original_init)
        params = list(sig.parameters.items())[1:]  # skip self
        
        for (name, param), value in zip(params, args):
            if name in hints and not isinstance(value, hints[name]):
                raise TypeError(f"Argument '{name}' must be {hints[name].__name__}, got {type(value).__name__}")
        
        original_init(self, *args, **kwargs)
    
    cls.__init__ = new_init
    return cls

@validate_types
class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

try:
    r1 = Rectangle(5.0, 3.0)   # OK
    r2 = Rectangle("5", 3.0)   # TypeError
except TypeError as e:
    print(f"Error: {e}")
```

### เปรียบเทียบ

```python
"""
Class Decorators vs Metaclasses:

Class Decorators:
+ เข้าใจง่ายกว่า
+ apply ที่ specific classes
+ สามารถ stack หลาย decorators ได้
+ ไม่ส่งผลต่อ subclasses (โดยค่า default)
- ถูกเรียกหลัง class สร้างเสร็จแล้ว
- ไม่สามารถเปลี่ยนแปลง class creation process ได้

Metaclasses:
+ ควบคุม class creation ได้อย่างสมบูรณ์
+ ส่งผลต่อ subclasses ทั้งหมดอัตโนมัติ
+ สามารถเปลี่ยนแปลง namespace ก่อนสร้าง class
+ เหมาะกับ framework development
- ซับซ้อนกว่า
- อาจเกิด metaclass conflicts
"""
```

---

## 7. __init_subclass__

`__init_subclass__` ถูกเรียกเมื่อมี subclass สร้างขึ้น โดยไม่ต้องใช้ metaclass

```python
class Plugin:
    """Base class ที่ track subclasses อัตโนมัติ"""
    
    _registry = {}
    
    def __init_subclass__(cls, plugin_name: str = None, **kwargs):
        super().__init_subclass__(**kwargs)
        
        if plugin_name is None:
            plugin_name = cls.__name__.lower()
        
        cls.plugin_name = plugin_name
        Plugin._registry[plugin_name] = cls
        print(f"Registered plugin: {plugin_name} -> {cls.__name__}")
    
    @classmethod
    def get_plugin(cls, name: str):
        return cls._registry.get(name)

class AudioPlugin(Plugin, plugin_name="audio"):
    def process(self, data):
        return f"Audio processed: {data}"

class VideoPlugin(Plugin, plugin_name="video"):
    def process(self, data):
        return f"Video processed: {data}"

class ImagePlugin(Plugin):  # ใช้ default name "imageplugin"
    def process(self, data):
        return f"Image processed: {data}"

# ใช้งาน
audio = Plugin.get_plugin("audio")()
print(audio.process("music.mp3"))

video = Plugin.get_plugin("video")()
print(video.process("movie.mp4"))

print("Registered plugins:", list(Plugin._registry.keys()))
```

```python
class Validated:
    """Base class ที่ validate subclass definitions"""
    
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        
        # ตรวจสอบว่า subclass มี required methods
        required_methods = getattr(cls, '_required_methods', [])
        for method_name in required_methods:
            if not hasattr(cls, method_name):
                raise TypeError(f"{cls.__name__} must implement {method_name}()")

class Model(Validated):
    _required_methods = ['validate', 'serialize']

try:
    class GoodModel(Model):
        def validate(self):
            return True
        
        def serialize(self):
            return {}

    class BadModel(Model):
        def validate(self):
            return True
        # ลืม implement serialize()

except TypeError as e:
    print(f"Error: {e}")
```

---

## 8. __set_name__

`__set_name__` ถูกเรียกเมื่อ descriptor ถูก assign ให้ class attribute

```python
class TypedDescriptor:
    """Descriptor ที่ enforce type checking"""
    
    def __set_name__(self, owner, name):
        # ถูกเรียกเมื่อ descriptor ถูก assign ใน class
        print(f"TypedDescriptor '{name}' assigned to {owner.__name__}")
        self.name = name
        self.private_name = f"_{name}"
    
    def __init__(self, expected_type):
        self.expected_type = expected_type
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)
    
    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"Attribute '{self.name}' must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(obj, self.private_name, value)

class Person:
    name = TypedDescriptor(str)    # __set_name__ ถูกเรียกที่นี่
    age = TypedDescriptor(int)     # __set_name__ ถูกเรียกที่นี่
    height = TypedDescriptor(float)  # __set_name__ ถูกเรียกที่นี่
    
    def __init__(self, name: str, age: int, height: float):
        self.name = name       # ผ่าน descriptor
        self.age = age
        self.height = height

p = Person("Alice", 30, 1.65)
print(f"Name: {p.name}, Age: {p.age}, Height: {p.height}")

try:
    p.age = "thirty"  # TypeError
except TypeError as e:
    print(f"Error: {e}")
```

---

## 9. Singleton Pattern ด้วย Metaclass

```python
class SingletonMeta(type):
    """Metaclass สำหรับ Singleton pattern"""
    
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self, url: str):
        self.url = url
        self._connected = False
    
    def connect(self):
        if not self._connected:
            print(f"Connecting to {self.url}")
            self._connected = True
        else:
            print("Already connected")
    
    def query(self, sql: str):
        if not self._connected:
            raise RuntimeError("Not connected!")
        return f"Results for: {sql}"

class Cache(metaclass=SingletonMeta):
    def __init__(self):
        self._data = {}
    
    def get(self, key):
        return self._data.get(key)
    
    def set(self, key, value):
        self._data[key] = value

# Database Singleton
db1 = Database("postgres://localhost/mydb")
db2 = Database("mysql://localhost/other")  # ไม่ได้สร้างใหม่

print(db1 is db2)    # True
print(db1.url)       # postgres://localhost/mydb (url แรก)

# Cache Singleton
cache1 = Cache()
cache2 = Cache()
cache1.set("key1", "value1")
print(cache2.get("key1"))  # value1 - same instance!
```

### Thread-safe Singleton

```python
import threading

class ThreadSafeSingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()
    
    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
        return cls._instances[cls]

class AppConfig(metaclass=ThreadSafeSingletonMeta):
    def __init__(self):
        self.settings = {}
    
    def set(self, key: str, value) -> None:
        self.settings[key] = value
    
    def get(self, key: str, default=None):
        return self.settings.get(key, default)

config = AppConfig()
config.set("debug", True)
config.set("version", "1.0.0")

same_config = AppConfig()
print(same_config.get("debug"))    # True
print(same_config.get("version"))  # 1.0.0
```

---

## 10. Registry Pattern

```python
class RegistryMeta(type):
    """Metaclass ที่ auto-register subclasses"""
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        
        # Initialize registry สำหรับ base class
        if not bases:  # base class ไม่มี bases
            cls._registry = {}
        else:
            # Register subclass ใน parent's registry
            for base in bases:
                if hasattr(base, '_registry'):
                    key = namespace.get('__registry_key__', name.lower())
                    base._registry[key] = cls
        
        return cls

class Handler(metaclass=RegistryMeta):
    """Base handler class"""
    
    def handle(self, request: dict) -> dict:
        raise NotImplementedError
    
    @classmethod
    def get_handler(cls, name: str) -> 'Handler':
        handler_cls = cls._registry.get(name)
        if handler_cls is None:
            raise KeyError(f"No handler registered for: {name}")
        return handler_cls()

class JSONHandler(Handler):
    __registry_key__ = "json"
    
    def handle(self, request: dict) -> dict:
        import json
        return {"response": json.dumps(request), "format": "json"}

class XMLHandler(Handler):
    __registry_key__ = "xml"
    
    def handle(self, request: dict) -> dict:
        # Simplified XML generation
        items = "\n".join(f"  <{k}>{v}</{k}>" for k, v in request.items())
        return {"response": f"<root>\n{items}\n</root>", "format": "xml"}

class CSVHandler(Handler):
    __registry_key__ = "csv"
    
    def handle(self, request: dict) -> dict:
        header = ",".join(request.keys())
        row = ",".join(str(v) for v in request.values())
        return {"response": f"{header}\n{row}", "format": "csv"}

# ใช้งาน
print("Registered handlers:", list(Handler._registry.keys()))
# ['json', 'xml', 'csv']

request_data = {"name": "Alice", "age": 30}

for format_name in ["json", "xml", "csv"]:
    handler = Handler.get_handler(format_name)
    result = handler.handle(request_data)
    print(f"\n{result['format'].upper()} format:")
    print(result['response'])
```

---

## 11. ORM-like Metaclass

```python
class FieldDescriptor:
    """Descriptor สำหรับ model fields"""
    
    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"
    
    def __init__(self, field_type, required=True, default=None):
        self.field_type = field_type
        self.required = required
        self.default = default
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, self.default)
    
    def __set__(self, obj, value):
        if value is None and self.required:
            raise ValueError(f"Field '{self.name}' is required")
        if value is not None and not isinstance(value, self.field_type):
            try:
                value = self.field_type(value)
            except (ValueError, TypeError):
                raise TypeError(f"Field '{self.name}' must be {self.field_type.__name__}")
        setattr(obj, self.private_name, value)

class ModelMeta(type):
    """Metaclass สำหรับ ORM-like model"""
    
    def __new__(mcs, name, bases, namespace):
        fields = {}
        
        # Collect fields จาก namespace
        for key, value in namespace.items():
            if isinstance(value, FieldDescriptor):
                fields[key] = value
        
        namespace['_fields'] = fields
        namespace['_table_name'] = name.lower() + 's'
        
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

class Model(metaclass=ModelMeta):
    """Base model class"""
    
    def __init__(self, **kwargs):
        for name, field in self._fields.items():
            value = kwargs.get(name, field.default)
            setattr(self, name, value)
    
    def to_dict(self) -> dict:
        return {
            name: getattr(self, name)
            for name in self._fields
        }
    
    @classmethod
    def get_schema(cls) -> str:
        """Generate SQL CREATE TABLE statement"""
        type_map = {str: "TEXT", int: "INTEGER", float: "REAL", bool: "INTEGER"}
        
        columns = ["id INTEGER PRIMARY KEY AUTOINCREMENT"]
        for name, field in cls._fields.items():
            sql_type = type_map.get(field.field_type, "TEXT")
            null = "" if field.required else " NOT NULL"
            columns.append(f"{name} {sql_type}{null}")
        
        return f"CREATE TABLE {cls._table_name} (\n  " + ",\n  ".join(columns) + "\n);"
    
    def __repr__(self) -> str:
        fields_str = ", ".join(
            f"{name}={getattr(self, name)!r}"
            for name in self._fields
        )
        return f"{self.__class__.__name__}({fields_str})"

class User(Model):
    name = FieldDescriptor(str)
    email = FieldDescriptor(str)
    age = FieldDescriptor(int, default=0)
    is_active = FieldDescriptor(bool, required=False, default=True)

class Product(Model):
    title = FieldDescriptor(str)
    price = FieldDescriptor(float)
    stock = FieldDescriptor(int, default=0)

# ใช้งาน
user = User(name="Alice", email="alice@example.com", age=30)
print(user)
print(user.to_dict())
print()
print("User table schema:")
print(User.get_schema())

product = Product(title="Laptop", price=999.99, stock=10)
print(product)
print()
print("Product table schema:")
print(Product.get_schema())
```

---

## 12. Abstract Metaclass

```python
from abc import ABCMeta, abstractmethod

# ABCMeta เป็น metaclass ของ ABC
print(type(ABCMeta))  # <class 'type'>

class Shape(metaclass=ABCMeta):
    """Abstract shape ด้วย ABCMeta โดยตรง"""
    
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass
    
    def describe(self) -> str:
        return f"{self.__class__.__name__}: area={self.area():.2f}, perimeter={self.perimeter():.2f}"

# เทียบเท่ากับ
from abc import ABC

class Shape2(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

# Combined metaclass
class StrictABCMeta(ABCMeta):
    """ABCMeta ที่เพิ่ม validation"""
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        
        # ตรวจสอบว่า concrete class มี docstrings
        if bases and not cls.__abstractmethods__:
            for method_name in dir(cls):
                method = getattr(cls, method_name)
                if callable(method) and not method_name.startswith('_'):
                    if not method.__doc__:
                        import warnings
                        warnings.warn(
                            f"{cls.__name__}.{method_name}() has no docstring",
                            UserWarning
                        )
        
        return cls

class StrictBase(metaclass=StrictABCMeta):
    @abstractmethod
    def compute(self) -> float:
        """Compute the result."""
        pass
```

---

## 13. Advanced Pattern: Auto-implement __repr__ และ __eq__

```python
class AutoMethodsMeta(type):
    """Metaclass ที่ auto-generate __repr__ และ __eq__"""
    
    def __new__(mcs, name, bases, namespace):
        # หา __init__ parameters
        init = namespace.get('__init__')
        
        if init and '__repr__' not in namespace:
            import inspect
            try:
                sig = inspect.signature(init)
                params = [p for p in sig.parameters if p != 'self']
                
                if params:
                    def make_repr(param_names):
                        def __repr__(self):
                            parts = [f"{p}={getattr(self, p)!r}" for p in param_names]
                            return f"{self.__class__.__name__}({', '.join(parts)})"
                        return __repr__
                    
                    namespace['__repr__'] = make_repr(params)
            except (ValueError, TypeError):
                pass
        
        if '__eq__' not in namespace:
            def make_eq(param_names_from_init):
                def __eq__(self, other):
                    if type(self) != type(other):
                        return NotImplemented
                    return all(
                        getattr(self, p) == getattr(other, p)
                        for p in param_names_from_init
                        if hasattr(self, p) and hasattr(other, p)
                    )
                return __eq__
            
            if init:
                import inspect
                try:
                    sig = inspect.signature(init)
                    params = [p for p in sig.parameters if p != 'self']
                    namespace['__eq__'] = make_eq(params)
                except (ValueError, TypeError):
                    pass
        
        return super().__new__(mcs, name, bases, namespace)

class Vector(metaclass=AutoMethodsMeta):
    def __init__(self, x: float, y: float, z: float = 0.0):
        self.x = x
        self.y = y
        self.z = z

v1 = Vector(1, 2, 3)
v2 = Vector(1, 2, 3)
v3 = Vector(4, 5, 6)

print(v1)          # Vector(x=1, y=2, z=3)
print(v1 == v2)    # True
print(v1 == v3)    # False
```

---

## 14. Metaclass สำหรับ Event System

```python
from typing import Callable, Dict, List, Any

class EventMeta(type):
    """Metaclass สำหรับ event-driven classes"""
    
    def __new__(mcs, name, bases, namespace):
        # หา methods ที่ขึ้นต้นด้วย 'on_'
        event_handlers = {}
        for attr_name, attr_value in namespace.items():
            if attr_name.startswith('on_') and callable(attr_value):
                event_name = attr_name[3:]  # ตัด 'on_' ออก
                event_handlers[event_name] = attr_value
        
        namespace['_event_handlers'] = event_handlers
        
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

class EventEmitter(metaclass=EventMeta):
    """Base class สำหรับ event-driven objects"""
    
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = {}
    
    def on(self, event: str, handler: Callable) -> None:
        if event not in self._listeners:
            self._listeners[event] = []
        self._listeners[event].append(handler)
    
    def emit(self, event: str, *args, **kwargs) -> None:
        # เรียก built-in handler ถ้ามี
        built_in = self._event_handlers.get(event)
        if built_in:
            built_in(self, *args, **kwargs)
        
        # เรียก registered listeners
        for handler in self._listeners.get(event, []):
            handler(*args, **kwargs)

class Button(EventEmitter):
    def __init__(self, label: str):
        super().__init__()
        self.label = label
    
    def click(self) -> None:
        self.emit("click", button=self)
    
    def on_click(self, button: 'Button') -> None:
        """Built-in click handler"""
        print(f"Button '{button.label}' was clicked!")

btn = Button("Submit")

# Register external listener
btn.on("click", lambda button=None: print(f"External handler: {button.label} clicked!"))

btn.click()
# Button 'Submit' was clicked!
# External handler: Submit clicked!
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Singleton ด้วย metaclass

```python
# สร้าง thread-safe Singleton metaclass และใช้กับ Logger class

import threading
from datetime import datetime

class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()
    
    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self, name: str = "app"):
        self.name = name
        self._logs = []
    
    def log(self, level: str, message: str) -> None:
        entry = {
            "timestamp": datetime.now().isoformat(),
            "level": level,
            "logger": self.name,
            "message": message
        }
        self._logs.append(entry)
        print(f"[{entry['timestamp']}] [{level}] {self.name}: {message}")
    
    def info(self, msg): self.log("INFO", msg)
    def error(self, msg): self.log("ERROR", msg)
    def debug(self, msg): self.log("DEBUG", msg)
    
    def get_logs(self): return self._logs.copy()

# Test
log1 = Logger("auth")
log2 = Logger("payment")

print(log1 is log2)  # True - same instance
log1.info("User logged in")
log2.error("Payment failed")
print(f"Total logs: {len(log1.get_logs())}")  # 2
```

### แบบฝึกหัดที่ 2: Registry Pattern

```python
# สร้าง plugin registry system

class PluginRegistry(type):
    _registry = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        
        if bases:  # ไม่ register base class
            plugin_type = namespace.get('plugin_type', name.lower())
            mcs._registry[plugin_type] = cls
        
        return cls
    
    @classmethod
    def get(mcs, plugin_type: str):
        return mcs._registry.get(plugin_type)
    
    @classmethod
    def list_plugins(mcs):
        return list(mcs._registry.keys())

class BasePlugin(metaclass=PluginRegistry):
    plugin_type = "base"
    
    def run(self, data): 
        raise NotImplementedError

class CSVPlugin(BasePlugin):
    plugin_type = "csv"
    
    def run(self, data):
        return ",".join(str(x) for x in data)

class JSONPlugin(BasePlugin):
    plugin_type = "json"
    
    def run(self, data):
        import json
        return json.dumps(data)

class XMLPlugin(BasePlugin):
    plugin_type = "xml"
    
    def run(self, data):
        items = "\n".join(f"  <item>{x}</item>" for x in data)
        return f"<root>\n{items}\n</root>"

print("Available plugins:", PluginRegistry.list_plugins())

data = [1, 2, 3, "hello", True]
for plugin_type in ["csv", "json", "xml"]:
    plugin_cls = PluginRegistry.get(plugin_type)
    if plugin_cls:
        print(f"\n{plugin_type.upper()}:")
        print(plugin_cls().run(data))
```

### แบบฝึกหัดที่ 3: Auto-generate properties

```python
# สร้าง metaclass ที่ auto-generate getter/setter properties

class AutoPropertyMeta(type):
    """Metaclass ที่ auto-generate properties จาก _fields dict"""
    
    def __new__(mcs, name, bases, namespace):
        fields = namespace.get('_auto_fields', {})
        
        for field_name, field_config in fields.items():
            field_type = field_config.get('type', object)
            private_name = f"_{field_name}"
            default = field_config.get('default', None)
            
            def make_property(fn, pt, pn, dv):
                def getter(self):
                    return getattr(self, pn, dv)
                
                def setter(self, value):
                    if value is not None and not isinstance(value, pt):
                        raise TypeError(f"{fn} must be {pt.__name__}")
                    setattr(self, pn, value)
                
                return property(getter, setter)
            
            namespace[field_name] = make_property(
                field_name, field_type, private_name, default
            )
        
        return super().__new__(mcs, name, bases, namespace)

class Config(metaclass=AutoPropertyMeta):
    _auto_fields = {
        'host': {'type': str, 'default': 'localhost'},
        'port': {'type': int, 'default': 8080},
        'debug': {'type': bool, 'default': False},
    }
    
    def __init__(self, host=None, port=None, debug=None):
        if host: self.host = host
        if port: self.port = port
        if debug is not None: self.debug = debug

c = Config("example.com", 443)
print(c.host)   # example.com
print(c.port)   # 443
print(c.debug)  # False

try:
    c.port = "not_a_number"
except TypeError as e:
    print(f"Error: {e}")
```

### แบบฝึกหัดที่ 4: __init_subclass__ สำหรับ validation

```python
from abc import ABC, abstractmethod

class ValidatedModel(ABC):
    """Base class ที่ validate subclass ด้วย __init_subclass__"""
    
    def __init_subclass__(cls, required_fields=None, **kwargs):
        super().__init_subclass__(**kwargs)
        
        if required_fields:
            cls._required_fields = required_fields
            
            original_init = cls.__init__ if '__init__' in cls.__dict__ else None
            
            def new_init(self, **kwargs):
                for field in required_fields:
                    if field not in kwargs:
                        raise ValueError(f"Required field missing: {field}")
                if original_init:
                    original_init(self, **kwargs)
                else:
                    for k, v in kwargs.items():
                        setattr(self, k, v)
            
            cls.__init__ = new_init

class UserModel(ValidatedModel, required_fields=['name', 'email']):
    def __init__(self, **kwargs):
        self.name = kwargs.get('name')
        self.email = kwargs.get('email')
        self.role = kwargs.get('role', 'user')
    
    def __repr__(self):
        return f"User(name={self.name!r}, email={self.email!r}, role={self.role!r})"

# Test
try:
    u1 = UserModel(name="Alice", email="alice@example.com")
    print(u1)
    
    u2 = UserModel(name="Bob")  # ขาด email
except ValueError as e:
    print(f"Error: {e}")
```

### แบบฝึกหัดที่ 5: Descriptor กับ __set_name__

```python
from typing import Any, Type, Optional
import re

class ValidatedField:
    """Generic descriptor ที่ validate values"""
    
    def __set_name__(self, owner: Type, name: str):
        self.name = name
        self.private_name = f"_{name}"
    
    def __init__(
        self,
        validator=None,
        required: bool = True,
        default: Any = None,
        doc: str = ""
    ):
        self.validator = validator
        self.required = required
        self.default = default
        self.__doc__ = doc
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, self.default)
    
    def __set__(self, obj, value):
        if value is None and self.required:
            raise ValueError(f"'{self.name}' is required")
        if value is not None and self.validator:
            if not self.validator(value):
                raise ValueError(f"'{self.name}' failed validation: {value!r}")
        setattr(obj, self.private_name, value)

class FormData:
    username = ValidatedField(
        validator=lambda v: isinstance(v, str) and 3 <= len(v) <= 20,
        doc="Username (3-20 chars)"
    )
    email = ValidatedField(
        validator=lambda v: isinstance(v, str) and '@' in v and '.' in v.split('@')[-1],
        doc="Valid email address"
    )
    age = ValidatedField(
        validator=lambda v: isinstance(v, int) and 13 <= v <= 120,
        required=False,
        default=None,
        doc="Age (13-120)"
    )
    
    def __init__(self, username: str, email: str, age: Optional[int] = None):
        self.username = username
        self.email = email
        self.age = age
    
    def __repr__(self):
        return f"FormData(username={self.username!r}, email={self.email!r}, age={self.age!r})"

# Test
try:
    f1 = FormData("alice", "alice@example.com", 25)
    print(f1)
    
    f2 = FormData("ab", "invalid")  # username too short, invalid email
except ValueError as e:
    print(f"Validation error: {e}")
```

### แบบฝึกหัดที่ 6: ORM metaclass ที่สมบูรณ์

```python
import sqlite3
from typing import Any, Dict, List, Optional, Type, TypeVar

T = TypeVar('T', bound='BaseModel')

class Column:
    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"
    
    def __init__(self, col_type: str, primary_key=False, nullable=True, default=None):
        self.col_type = col_type
        self.primary_key = primary_key
        self.nullable = nullable
        self.default = default
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, self.default)
    
    def __set__(self, obj, value):
        setattr(obj, self.private_name, value)

class ModelMeta(type):
    def __new__(mcs, name, bases, namespace):
        columns = {}
        primary_key_col = None
        
        for key, value in namespace.items():
            if isinstance(value, Column):
                columns[key] = value
                if value.primary_key:
                    primary_key_col = key
        
        namespace['_columns'] = columns
        namespace['_primary_key'] = primary_key_col or 'id'
        namespace['_table_name'] = name.lower() + 's'
        
        return super().__new__(mcs, name, bases, namespace)

class BaseModel(metaclass=ModelMeta):
    id = Column('INTEGER', primary_key=True)
    
    def __init__(self, **kwargs):
        for name, col in self._columns.items():
            setattr(self, name, kwargs.get(name, col.default))
    
    def to_dict(self) -> Dict[str, Any]:
        return {name: getattr(self, name) for name in self._columns}
    
    @classmethod
    def create_table_sql(cls) -> str:
        col_defs = []
        for name, col in cls._columns.items():
            nullable = "" if col.nullable else " NOT NULL"
            pk = " PRIMARY KEY AUTOINCREMENT" if col.primary_key else ""
            col_defs.append(f"{name} {col.col_type}{nullable}{pk}")
        return f"CREATE TABLE IF NOT EXISTS {cls._table_name} ({', '.join(col_defs)})"
    
    def __repr__(self):
        fields = ", ".join(f"{k}={getattr(self, k)!r}" for k in self._columns if k != 'id')
        return f"{self.__class__.__name__}(id={self.id}, {fields})"

class Article(BaseModel):
    title = Column('TEXT', nullable=False)
    content = Column('TEXT')
    author = Column('TEXT')
    views = Column('INTEGER', default=0)

class Comment(BaseModel):
    article_id = Column('INTEGER')
    text = Column('TEXT', nullable=False)
    author = Column('TEXT')

print("Articles table:", Article.create_table_sql())
print("Comments table:", Comment.create_table_sql())

a = Article(title="Python Metaclasses", content="...", author="Alice")
print(a)
print(a.to_dict())
```

### แบบฝึกหัดที่ 7: Decorator vs Metaclass

```python
# เขียนฟีเจอร์เดียวกันด้วยทั้ง decorator และ metaclass แล้วเปรียบเทียบ

# Feature: Auto-log method calls

import functools
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# วิธีที่ 1: Class Decorator
def auto_log_decorator(cls):
    for name in dir(cls):
        if not name.startswith('_'):
            method = getattr(cls, name)
            if callable(method):
                @functools.wraps(method)
                def logged_method(self, *args, method_name=name, original_method=method, **kwargs):
                    logger.info(f"Calling {cls.__name__}.{method_name}({args}, {kwargs})")
                    result = original_method(self, *args, **kwargs)
                    logger.info(f"{cls.__name__}.{method_name} returned {result}")
                    return result
                setattr(cls, name, logged_method)
    return cls

# วิธีที่ 2: Metaclass
class AutoLogMeta(type):
    def __new__(mcs, name, bases, namespace):
        for attr_name, attr_value in list(namespace.items()):
            if callable(attr_value) and not attr_name.startswith('_'):
                namespace[attr_name] = mcs._wrap_method(attr_value, name)
        return super().__new__(mcs, name, bases, namespace)
    
    @staticmethod
    def _wrap_method(method, class_name):
        @functools.wraps(method)
        def wrapper(self, *args, **kwargs):
            logger.info(f"Calling {class_name}.{method.__name__}({args}, {kwargs})")
            result = method(self, *args, **kwargs)
            logger.info(f"{class_name}.{method.__name__} returned {result}")
            return result
        return wrapper

@auto_log_decorator
class CalculatorDecorator:
    def add(self, a, b): return a + b
    def multiply(self, a, b): return a * b

class CalculatorMeta(metaclass=AutoLogMeta):
    def add(self, a, b): return a + b
    def multiply(self, a, b): return a * b

c1 = CalculatorDecorator()
c2 = CalculatorMeta()

print("Decorator:", c1.add(3, 4))
print("Metaclass:", c2.multiply(3, 4))
```

### แบบฝึกหัดที่ 8: DSL ด้วย Metaclass

```python
# สร้าง mini DSL สำหรับ state machine

class StateMeta(type):
    """Metaclass สำหรับ State Machine DSL"""
    
    def __new__(mcs, name, bases, namespace):
        states = {}
        transitions = {}
        initial_state = None
        
        for key, value in namespace.items():
            if isinstance(value, dict) and '__state__' in value:
                state_name = value['__state__']
                states[state_name] = value.get('handler')
                if value.get('initial', False):
                    initial_state = state_name
            elif isinstance(value, dict) and '__transition__' in value:
                from_state = value['from']
                trigger = value['__transition__']
                to_state = value['to']
                if from_state not in transitions:
                    transitions[from_state] = {}
                transitions[from_state][trigger] = to_state
        
        namespace['_states'] = states
        namespace['_transitions'] = transitions
        namespace['_initial_state'] = initial_state
        
        return super().__new__(mcs, name, bases, namespace)

class StateMachine(metaclass=StateMeta):
    def __init__(self):
        self._current_state = self._initial_state
    
    @property
    def state(self):
        return self._current_state
    
    def trigger(self, event: str) -> bool:
        transitions = self._transitions.get(self._current_state, {})
        if event in transitions:
            new_state = transitions[event]
            print(f"Transition: {self._current_state} --{event}--> {new_state}")
            self._current_state = new_state
            return True
        print(f"No transition for event '{event}' in state '{self._current_state}'")
        return False

# ใช้งาน StateMachine
class TrafficLight(StateMachine):
    # กำหนด states
    red_state = {'__state__': 'red', 'initial': True}
    green_state = {'__state__': 'green'}
    yellow_state = {'__state__': 'yellow'}
    
    # กำหนด transitions
    red_to_green = {'__transition__': 'go', 'from': 'red', 'to': 'green'}
    green_to_yellow = {'__transition__': 'slow', 'from': 'green', 'to': 'yellow'}
    yellow_to_red = {'__transition__': 'stop', 'from': 'yellow', 'to': 'red'}

light = TrafficLight()
print(f"Initial state: {light.state}")  # red

light.trigger("go")    # red -> green
light.trigger("slow")  # green -> yellow
light.trigger("stop")  # yellow -> red
light.trigger("stop")  # No transition (already red)
```

---

## สรุป

| หัวข้อ | Key Points |
|--------|-----------|
| `type()` | สร้าง class dynamically ด้วย `type(name, bases, dict)` |
| Metaclass | Class ของ class ควบคุม class creation |
| `__new__` | สร้าง instance ก่อน init |
| `__init__` | Initialize instance หลังสร้าง |
| Custom metaclass | Inherit จาก `type`, override `__new__`/`__init__` |
| `__init_subclass__` | React เมื่อมี subclass สร้างขึ้น (Python 3.6+) |
| `__set_name__` | Descriptor ได้รับชื่อ attribute |
| Singleton | Metaclass คุม `__call__` เพื่อ return same instance |
| Registry | Track subclasses อัตโนมัติ |

**Pythonic guideline:**
- ใช้ class decorators ก่อน (เข้าใจง่ายกว่า)
- ใช้ `__init_subclass__` สำหรับ subclass hooks
- ใช้ metaclass เมื่อจำเป็นจริงๆ เช่น framework development, ORM
- "If you're not sure if you need a metaclass, you probably don't" - Tim Peters
