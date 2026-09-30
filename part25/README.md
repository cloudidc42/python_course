# Part 25 - OOP: Abstract Classes & Interfaces

## สารบัญ
1. [Abstract Classes (abc module)](#abstract-classes-abc-module)
2. [ABC และ ABCMeta](#abc-และ-abcmeta)
3. [@abstractmethod Decorator](#abstractmethod-decorator)
4. [@abstractproperty](#abstractproperty)
5. [Abstract Class Patterns](#abstract-class-patterns)
6. [Interface Concept ใน Python](#interface-concept-ใน-python)
7. [Protocol Class (typing.Protocol)](#protocol-class-typingprotocol)
8. [Structural Subtyping](#structural-subtyping)
9. [Abstract Factory Pattern เบื้องต้น](#abstract-factory-pattern-เบื้องต้น)
10. [Template Method Pattern](#template-method-pattern)
11. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Abstract Classes (abc module)

**Abstract Class** คือ class ที่ออกแบบมาเพื่อเป็น "blueprint" ให้ subclasses ปฏิบัติตาม โดย:
- ไม่สามารถสร้าง instance ได้โดยตรง
- กำหนด interface ที่ subclasses ต้อง implement
- อาจมี concrete methods ที่ใช้ร่วมกันได้

### ทำไมต้องใช้ Abstract Classes?

```python
# ===== ปัญหาโดยไม่มี Abstract Class =====

class Shape:
    def area(self):
        raise NotImplementedError("Subclasses must implement area()")

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    # ลืม implement area()!

# ปัญหา: Python สร้าง Circle ได้โดยไม่ error!
c = Circle(5)
try:
    print(c.area())  # Error เกิดตอน runtime เท่านั้น
except NotImplementedError as e:
    print(f"Runtime Error: {e}")


# ===== แก้ไขด้วย Abstract Class =====

from abc import ABC, abstractmethod

class ProperShape(ABC):
    @abstractmethod
    def area(self):
        pass

class ProperCircle(ProperShape):
    def __init__(self, radius):
        self.radius = radius
    # ยังไม่ implement area()!

# ตอนนี้ Python ป้องกันตั้งแต่ instantiation!
try:
    c = ProperCircle(5)  # TypeError ทันที
except TypeError as e:
    print(f"Prevented at creation: {e}")
```

---

## ABC และ ABCMeta

Python มี 2 วิธีในการสร้าง abstract classes

### ตัวอย่างที่ 1: ABC vs ABCMeta

```python
from abc import ABC, ABCMeta, abstractmethod

# วิธีที่ 1: สืบทอดจาก ABC (แนะนำ - ง่ายกว่า)
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass


# วิธีที่ 2: ใช้ ABCMeta เป็น metaclass (ยืดหยุ่นกว่า)
class Vehicle(metaclass=ABCMeta):
    @abstractmethod
    def start(self):
        pass
    
    @abstractmethod
    def stop(self):
        pass


# วิธีที่ 3: เมื่อ class ต้อง inherit จาก class อื่นและ abstract พร้อมกัน
class ThreadSafeABC(ABCMeta):
    """Custom metaclass ที่รวม ABCMeta กับ functionality อื่น"""
    pass


# ทดสอบ
print(f"Shape เป็น abstract: {hasattr(Shape, '__abstractmethods__')}")
print(f"Abstract methods ของ Shape: {Shape.__abstractmethods__}")

class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    def area(self):
        return self.side ** 2
    
    def perimeter(self):
        return 4 * self.side

s = Square(5)
print(f"Square area: {s.area()}")
print(f"Square perimeter: {s.perimeter()}")
print(f"Square เป็น Shape: {isinstance(s, Shape)}")
```

### ตัวอย่างที่ 2: Abstract Class พร้อม Concrete Methods

```python
from abc import ABC, abstractmethod

class DataProcessor(ABC):
    """Abstract class สำหรับ data processing pipeline"""
    
    def __init__(self, name):
        self.name = name
        self._processed_count = 0
    
    @abstractmethod
    def validate(self, data) -> bool:
        """Validate ข้อมูล - subclasses ต้อง implement"""
        pass
    
    @abstractmethod
    def transform(self, data):
        """Transform ข้อมูล - subclasses ต้อง implement"""
        pass
    
    @abstractmethod
    def save(self, data) -> bool:
        """บันทึกข้อมูล - subclasses ต้อง implement"""
        pass
    
    # Concrete methods - ใช้ร่วมกันทุก subclass
    def process(self, data):
        """Template method - ลำดับการทำงานที่แน่นอน"""
        print(f"\n[{self.name}] เริ่มประมวลผล...")
        
        if not self.validate(data):
            print(f"  ข้อมูลไม่ถูกต้อง")
            return False
        
        transformed = self.transform(data)
        print(f"  Transform สำเร็จ")
        
        if not self.save(transformed):
            print(f"  บันทึกล้มเหลว")
            return False
        
        self._processed_count += 1
        print(f"  ประมวลผลสำเร็จ (รวม {self._processed_count} ครั้ง)")
        return True
    
    def get_stats(self):
        return {
            "processor": self.name,
            "processed_count": self._processed_count
        }
    
    def __str__(self):
        return f"{type(self).__name__}(name={self.name!r})"


class NumberProcessor(DataProcessor):
    """ประมวลผลตัวเลข"""
    
    def __init__(self):
        super().__init__("NumberProcessor")
        self._results = []
    
    def validate(self, data):
        return isinstance(data, (int, float)) and data > 0
    
    def transform(self, data):
        return data * 2 + 1
    
    def save(self, data):
        self._results.append(data)
        return True


class TextProcessor(DataProcessor):
    """ประมวลผล text"""
    
    def __init__(self, max_length=100):
        super().__init__("TextProcessor")
        self.max_length = max_length
        self._results = []
    
    def validate(self, data):
        return isinstance(data, str) and 0 < len(data) <= self.max_length
    
    def transform(self, data):
        # Normalize: lowercase, strip, ลบ multiple spaces
        import re
        return re.sub(r'\s+', ' ', data.lower().strip())
    
    def save(self, data):
        self._results.append(data)
        return True


# ทดสอบ
np = NumberProcessor()
tp = TextProcessor(50)

np.process(5)
np.process(-3)   # ไม่ผ่าน validate
np.process(10)

tp.process("  Hello   World  ")
tp.process("Python is AWESOME!")
tp.process("x" * 100)  # ยาวเกิน

print("\nสถิติ:")
print(np.get_stats())
print(tp.get_stats())
```

---

## @abstractmethod Decorator

### ตัวอย่างที่ 3: abstractmethod หลากหลายรูปแบบ

```python
from abc import ABC, abstractmethod

class FullAbstractExample(ABC):
    """แสดง abstract methods ทุกรูปแบบ"""
    
    # 1. Abstract instance method
    @abstractmethod
    def instance_method(self):
        pass
    
    # 2. Abstract class method
    @classmethod
    @abstractmethod
    def class_method(cls):
        pass
    
    # 3. Abstract static method
    @staticmethod
    @abstractmethod
    def static_method():
        pass
    
    # 4. Abstract property
    @property
    @abstractmethod
    def my_property(self):
        pass
    
    # 5. Abstract property with setter
    @property
    @abstractmethod
    def settable_property(self):
        pass
    
    @settable_property.setter
    @abstractmethod
    def settable_property(self, value):
        pass
    
    # Concrete method ที่ใช้ abstract methods
    def template(self):
        print(f"Instance: {self.instance_method()}")
        print(f"Class: {self.class_method()}")
        print(f"Static: {self.static_method()}")
        print(f"Property: {self.my_property}")


class ConcreteImplementation(FullAbstractExample):
    """ต้อง implement ทุก abstract methods"""
    
    def __init__(self, value):
        self._value = value
    
    def instance_method(self):
        return f"instance: {self._value}"
    
    @classmethod
    def class_method(cls):
        return f"class: {cls.__name__}"
    
    @staticmethod
    def static_method():
        return "static method"
    
    @property
    def my_property(self):
        return self._value * 2
    
    @property
    def settable_property(self):
        return self._value
    
    @settable_property.setter
    def settable_property(self, value):
        self._value = value


# ทดสอบ
ci = ConcreteImplementation(10)
ci.template()
ci.settable_property = 20
print(f"หลัง set: {ci.settable_property}")

# ตรวจสอบ abstract methods
print(f"\nAbstract methods ของ FullAbstractExample:")
print(FullAbstractExample.__abstractmethods__)
```

### ตัวอย่างที่ 4: Abstract Class กับ Default Implementation

```python
from abc import ABC, abstractmethod

class Serializable(ABC):
    """Abstract class ที่มี default implementation"""
    
    @abstractmethod
    def to_dict(self) -> dict:
        """ต้อง implement - แปลงเป็น dict"""
        pass
    
    def to_json(self) -> str:
        """Concrete: ใช้ to_dict() ที่ subclass implement"""
        import json
        return json.dumps(self.to_dict(), ensure_ascii=False, indent=2)
    
    def to_csv_row(self) -> str:
        """Concrete: แปลง dict values เป็น CSV row"""
        data = self.to_dict()
        return ','.join(str(v) for v in data.values())
    
    @classmethod
    def from_dict(cls, data: dict):
        """Concrete: สร้าง instance จาก dict
        - subclass ต้องมี __init__ ที่รับ **kwargs
        """
        return cls(**data)
    
    @abstractmethod
    def validate(self) -> bool:
        """ต้อง implement - ตรวจสอบความถูกต้อง"""
        pass
    
    def save(self, filepath: str) -> None:
        """Concrete: บันทึกเป็น JSON"""
        if not self.validate():
            raise ValueError("ข้อมูลไม่ถูกต้อง ไม่สามารถบันทึกได้")
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(self.to_json())
        print(f"บันทึกแล้ว: {filepath}")


class Product(Serializable):
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock
    
    def to_dict(self):
        return {
            "name": self.name,
            "price": self.price,
            "stock": self.stock
        }
    
    def validate(self):
        return (isinstance(self.name, str) and len(self.name) > 0 and
                isinstance(self.price, (int, float)) and self.price > 0 and
                isinstance(self.stock, int) and self.stock >= 0)
    
    def __repr__(self):
        return f"Product({self.name!r}, price={self.price}, stock={self.stock})"


class Customer(Serializable):
    def __init__(self, name, email, age):
        self.name = name
        self.email = email
        self.age = age
    
    def to_dict(self):
        return {"name": self.name, "email": self.email, "age": self.age}
    
    def validate(self):
        import re
        return (len(self.name) >= 2 and
                re.match(r'.+@.+\..+', self.email) and
                13 <= self.age <= 120)


# ทดสอบ
p = Product("Python Book", 599, 50)
print(p.to_json())
print(f"CSV: {p.to_csv_row()}")
print(f"Valid: {p.validate()}")

c = Customer("สมชาย ใจดี", "somchai@example.com", 30)
print(c.to_json())
```

---

## @abstractproperty

`@abstractproperty` เป็น deprecated แล้วใน Python 3.3+ ควรใช้ `@property @abstractmethod` แทน

### ตัวอย่างที่ 5: Abstract Properties ที่ถูกต้อง

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    """Abstract class พร้อม abstract properties"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """ชื่อสัตว์ - ต้อง implement"""
        pass
    
    @property
    @abstractmethod
    def sound(self) -> str:
        """เสียงที่ส่ง - ต้อง implement"""
        pass
    
    @property
    @abstractmethod
    def legs(self) -> int:
        """จำนวนขา - ต้อง implement"""
        pass
    
    # Concrete method ที่ใช้ abstract properties
    def describe(self):
        print(f"ฉันเป็น {self.name}")
        print(f"  ส่งเสียง: {self.sound}")
        print(f"  มี {self.legs} ขา")


class Dog(Animal):
    def __init__(self, given_name):
        self._given_name = given_name
    
    @property
    def name(self):
        return f"สุนัข ({self._given_name})"
    
    @property
    def sound(self):
        return "โฮ่ง!"
    
    @property
    def legs(self):
        return 4


class Spider(Animal):
    @property
    def name(self):
        return "แมงมุม"
    
    @property
    def sound(self):
        return "(เงียบ)"
    
    @property
    def legs(self):
        return 8


# ทดสอบ
animals = [Dog("Rex"), Spider()]
for animal in animals:
    animal.describe()
    print()
```

---

## Abstract Class Patterns

### ตัวอย่างที่ 6: Strategy Pattern ด้วย Abstract Class

```python
from abc import ABC, abstractmethod
from typing import List

class SortStrategy(ABC):
    """Abstract strategy สำหรับ sorting"""
    
    @abstractmethod
    def sort(self, data: List) -> List:
        pass
    
    @property
    @abstractmethod
    def name(self) -> str:
        pass


class BubbleSort(SortStrategy):
    @property
    def name(self):
        return "Bubble Sort"
    
    def sort(self, data):
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr


class QuickSort(SortStrategy):
    @property
    def name(self):
        return "Quick Sort"
    
    def sort(self, data):
        arr = data.copy()
        self._quicksort(arr, 0, len(arr) - 1)
        return arr
    
    def _quicksort(self, arr, low, high):
        if low < high:
            pi = self._partition(arr, low, high)
            self._quicksort(arr, low, pi - 1)
            self._quicksort(arr, pi + 1, high)
    
    def _partition(self, arr, low, high):
        pivot = arr[high]
        i = low - 1
        for j in range(low, high):
            if arr[j] <= pivot:
                i += 1
                arr[i], arr[j] = arr[j], arr[i]
        arr[i+1], arr[high] = arr[high], arr[i+1]
        return i + 1


class MergeSort(SortStrategy):
    @property
    def name(self):
        return "Merge Sort"
    
    def sort(self, data):
        if len(data) <= 1:
            return data.copy()
        mid = len(data) // 2
        left = self.sort(data[:mid])
        right = self.sort(data[mid:])
        return self._merge(left, right)
    
    def _merge(self, left, right):
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


class Sorter:
    """Context class ที่ใช้ strategy"""
    
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy
    
    @property
    def strategy(self):
        return self._strategy
    
    @strategy.setter
    def strategy(self, value):
        if not isinstance(value, SortStrategy):
            raise TypeError("strategy ต้องเป็น SortStrategy")
        self._strategy = value
    
    def sort(self, data):
        print(f"ใช้ {self._strategy.name}")
        result = self._strategy.sort(data)
        return result


# ทดสอบ
import random
import time

data = random.sample(range(1, 1000), 100)

sorter = Sorter(BubbleSort())
sorted1 = sorter.sort(data[:])

sorter.strategy = QuickSort()
sorted2 = sorter.sort(data[:])

sorter.strategy = MergeSort()
sorted3 = sorter.sort(data[:])

print(f"ทุก sort ให้ผลเดียวกัน: {sorted1 == sorted2 == sorted3}")
print(f"5 ตัวแรก: {sorted1[:5]}")
```

---

## Interface Concept ใน Python

Python ไม่มี `interface` keyword เหมือน Java แต่สามารถสร้าง interface-like behavior ได้หลายวิธี

### ตัวอย่างที่ 7: Interface ด้วย Pure Abstract Class

```python
from abc import ABC, abstractmethod

# "Interface" ใน Python = Pure Abstract Class
# (ทุก method เป็น abstract, ไม่มี instance variables)

class Drawable(ABC):
    """Interface: สิ่งที่วาดได้"""
    
    @abstractmethod
    def draw(self, canvas) -> None:
        pass
    
    @abstractmethod
    def get_bounds(self) -> tuple:
        """return (x, y, width, height)"""
        pass


class Resizable(ABC):
    """Interface: สิ่งที่ resize ได้"""
    
    @abstractmethod
    def resize(self, factor: float) -> None:
        pass
    
    @abstractmethod
    def set_size(self, width: float, height: float) -> None:
        pass


class Clickable(ABC):
    """Interface: สิ่งที่ click ได้"""
    
    @abstractmethod
    def on_click(self, x: float, y: float) -> None:
        pass
    
    @abstractmethod
    def contains_point(self, x: float, y: float) -> bool:
        pass


# Class ที่ implement หลาย interfaces
class Button(Drawable, Resizable, Clickable):
    """Button UI component"""
    
    def __init__(self, x, y, width, height, label):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.label = label
        self._click_handler = None
    
    # Drawable
    def draw(self, canvas):
        print(f"[{canvas}] วาด Button '{self.label}' ที่ ({self.x},{self.y}) "
              f"ขนาด {self.width}x{self.height}")
    
    def get_bounds(self):
        return (self.x, self.y, self.width, self.height)
    
    # Resizable
    def resize(self, factor):
        self.width *= factor
        self.height *= factor
    
    def set_size(self, width, height):
        self.width = width
        self.height = height
    
    # Clickable
    def on_click(self, x, y):
        if self.contains_point(x, y):
            print(f"Button '{self.label}' ถูกกด!")
            if self._click_handler:
                self._click_handler(self)
    
    def contains_point(self, x, y):
        return (self.x <= x <= self.x + self.width and
                self.y <= y <= self.y + self.height)
    
    def set_click_handler(self, handler):
        self._click_handler = handler


# ทดสอบ
def handle_click(button):
    print(f"  Handler เรียกสำหรับ '{button.label}'")

btn = Button(10, 20, 100, 40, "Submit")
btn.set_click_handler(handle_click)

btn.draw("MainCanvas")
btn.on_click(50, 35)   # ใน button
btn.on_click(200, 35)  # นอก button

btn.resize(1.5)
btn.draw("MainCanvas")

print(f"\nButton เป็น Drawable: {isinstance(btn, Drawable)}")
print(f"Button เป็น Resizable: {isinstance(btn, Resizable)}")
print(f"Button เป็น Clickable: {isinstance(btn, Clickable)}")
```

### ตัวอย่างที่ 8: Multiple Interface Implementation

```python
from abc import ABC, abstractmethod

class Readable(ABC):
    @abstractmethod
    def read(self) -> str:
        pass
    
    @abstractmethod
    def read_line(self) -> str:
        pass

class Writable(ABC):
    @abstractmethod
    def write(self, data: str) -> int:
        pass
    
    @abstractmethod
    def flush(self) -> None:
        pass

class Seekable(ABC):
    @abstractmethod
    def seek(self, position: int) -> None:
        pass
    
    @abstractmethod
    def tell(self) -> int:
        pass

class Closeable(ABC):
    @abstractmethod
    def close(self) -> None:
        pass
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()

class InMemoryBuffer(Readable, Writable, Seekable, Closeable):
    """In-memory buffer ที่ implement ทุก interfaces"""
    
    def __init__(self):
        self._data = []
        self._position = 0
        self._closed = False
    
    def _check_not_closed(self):
        if self._closed:
            raise IOError("Buffer ปิดแล้ว")
    
    # Readable
    def read(self):
        self._check_not_closed()
        result = ''.join(self._data[self._position:])
        self._position = len(self._data)
        return result
    
    def read_line(self):
        self._check_not_closed()
        text = ''.join(self._data[self._position:])
        idx = text.find('\n')
        if idx == -1:
            self._position = len(self._data)
            return text
        line = text[:idx+1]
        self._position += idx + 1
        return line
    
    # Writable
    def write(self, data):
        self._check_not_closed()
        self._data.extend(list(data))
        return len(data)
    
    def flush(self):
        pass  # In-memory ไม่ต้อง flush
    
    # Seekable
    def seek(self, position):
        self._check_not_closed()
        self._position = max(0, min(position, len(self._data)))
    
    def tell(self):
        self._check_not_closed()
        return self._position
    
    # Closeable
    def close(self):
        self._closed = True
        print("Buffer ปิดแล้ว")
    
    @property
    def value(self):
        return ''.join(self._data)


# ทดสอบ
with InMemoryBuffer() as buf:
    buf.write("Hello, World!\n")
    buf.write("Python Programming\n")
    buf.write("OOP is fun!")
    
    print("ค่าใน buffer:")
    print(buf.value)
    
    buf.seek(0)
    print("\nอ่านทีละบรรทัด:")
    for _ in range(3):
        line = buf.read_line()
        if line:
            print(f"  '{line.strip()}'")
    
    print(f"\nตำแหน่งปัจจุบัน: {buf.tell()}")
```

---

## Protocol Class (typing.Protocol)

Python 3.8+ มี `typing.Protocol` ที่ให้กำหนด interface แบบ structural (duck typing + type checking)

### ตัวอย่างที่ 9: typing.Protocol

```python
from typing import Protocol, runtime_checkable, Iterator, List

@runtime_checkable
class Comparable(Protocol):
    """Protocol: สิ่งที่เปรียบเทียบได้"""
    def __lt__(self, other) -> bool: ...
    def __eq__(self, other) -> bool: ...

@runtime_checkable
class Sized(Protocol):
    """Protocol: สิ่งที่มีขนาด"""
    def __len__(self) -> int: ...

@runtime_checkable
class Container(Protocol):
    """Protocol: container"""
    def __contains__(self, item) -> bool: ...

@runtime_checkable
class Collection(Sized, Container, Protocol):
    """รวม Protocol หลายตัว"""
    def __iter__(self) -> Iterator: ...


# ทดสอบ - classes ที่ implement protocol โดยไม่ต้อง inherit

class Scores:
    """Custom class ที่ implement Collection protocol"""
    
    def __init__(self, data):
        self._data = sorted(data, reverse=True)
    
    def __len__(self):
        return len(self._data)
    
    def __contains__(self, item):
        return item in self._data
    
    def __iter__(self):
        return iter(self._data)
    
    def __lt__(self, other):
        return min(self._data) < min(other._data)
    
    def __eq__(self, other):
        return sorted(self._data) == sorted(other._data)


scores = Scores([95, 87, 72, 65, 88, 91])

print(f"Scores ใช้ Sized protocol: {isinstance(scores, Sized)}")
print(f"Scores ใช้ Container protocol: {isinstance(scores, Container)}")
print(f"Scores ใช้ Collection protocol: {isinstance(scores, Collection)}")
print(f"Scores ใช้ Comparable protocol: {isinstance(scores, Comparable)}")

print(f"\nขนาด: {len(scores)}")
print(f"90 ใน scores: {90 in scores}")
print(f"91 ใน scores: {91 in scores}")

print("ทุกคะแนน:")
for score in scores:
    print(f"  {score}")
```

### ตัวอย่างที่ 10: Protocol สำหรับ Type Checking

```python
from typing import Protocol, runtime_checkable, Optional

@runtime_checkable
class Hashable(Protocol):
    def __hash__(self) -> int: ...

@runtime_checkable  
class SupportsRead(Protocol):
    def read(self, n: int = -1) -> str: ...

@runtime_checkable
class SupportsWrite(Protocol):
    def write(self, s: str) -> int: ...


class Logger(Protocol):
    """Protocol สำหรับ logging"""
    def log(self, level: str, message: str) -> None: ...
    def error(self, message: str) -> None: ...
    def info(self, message: str) -> None: ...


class SimpleLogger:
    """Implements Logger protocol โดยไม่ต้อง inherit"""
    
    def log(self, level, message):
        print(f"[{level.upper()}] {message}")
    
    def error(self, message):
        self.log("ERROR", message)
    
    def info(self, message):
        self.log("INFO", message)


class ColoredLogger:
    """อีก logger ที่ implement Logger protocol"""
    
    COLORS = {
        'ERROR': '\033[91m',   # Red
        'INFO': '\033[92m',    # Green
        'WARNING': '\033[93m', # Yellow
        'RESET': '\033[0m'
    }
    
    def log(self, level, message):
        color = self.COLORS.get(level.upper(), '')
        reset = self.COLORS['RESET']
        print(f"{color}[{level.upper()}]{reset} {message}")
    
    def error(self, message):
        self.log("ERROR", message)
    
    def info(self, message):
        self.log("INFO", message)


def run_application(logger: Logger):
    """Function ที่รับ Logger protocol ใดๆ"""
    logger.info("เริ่มต้นแอปพลิเคชัน")
    logger.info("กำลังโหลดการตั้งค่า")
    logger.error("พบข้อผิดพลาดในการเชื่อมต่อ")
    logger.info("ปิดแอปพลิเคชัน")


# ทำงานกับ logger ใดๆ ที่ implement Logger protocol
run_application(SimpleLogger())
print()
run_application(ColoredLogger())
```

---

## Structural Subtyping

Structural subtyping คือ type compatibility ที่อ้างอิงจาก structure (members) ไม่ใช่ explicit inheritance

### ตัวอย่างที่ 11: Structural vs Nominal Subtyping

```python
from typing import Protocol

# Nominal Subtyping (ปกติ) - ต้อง inherit
class Named:
    def get_name(self) -> str:
        return "Named"

class Animal(Named):  # ต้อง explicitly inherit
    def __init__(self, name):
        self._name = name
    
    def get_name(self):
        return self._name


# Structural Subtyping (Protocol) - ไม่ต้อง inherit
class HasName(Protocol):
    def get_name(self) -> str: ...


class Person:
    """ไม่ inherit จาก HasName แต่ compatible ด้วย structure"""
    
    def __init__(self, name):
        self._name = name
    
    def get_name(self):
        return self._name


class Robot:
    """อีก class ที่ compatible"""
    
    def __init__(self, model):
        self._model = model
    
    def get_name(self):
        return f"Robot-{self._model}"


def greet(entity: HasName):
    """รับ HasName protocol ใดๆ"""
    print(f"สวัสดี, {entity.get_name()}!")


# ทำงานได้กับทุก class ที่มี get_name()
greet(Animal("บักโกง"))
greet(Person("สมชาย"))
greet(Robot("R2D2"))


# ============================================================
# Protocols สำหรับ Data Classes
# ============================================================

class Serializable(Protocol):
    def to_dict(self) -> dict: ...
    
    @classmethod
    def from_dict(cls, data: dict): ...


def save_to_db(obj: Serializable, table: str):
    """บันทึก object ลง database (จำลอง)"""
    data = obj.to_dict()
    print(f"INSERT INTO {table}: {data}")


class UserRecord:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def to_dict(self):
        return {"name": self.name, "email": self.email}
    
    @classmethod
    def from_dict(cls, data):
        return cls(**data)


class ProductRecord:
    def __init__(self, sku, price):
        self.sku = sku
        self.price = price
    
    def to_dict(self):
        return {"sku": self.sku, "price": self.price}
    
    @classmethod
    def from_dict(cls, data):
        return cls(**data)


user = UserRecord("Alice", "alice@example.com")
product = ProductRecord("P001", 99.99)

save_to_db(user, "users")
save_to_db(product, "products")
```

---

## Abstract Factory Pattern เบื้องต้น

**Abstract Factory** คือ design pattern ที่ให้สร้าง family ของ related objects โดยไม่ระบุ concrete classes

### ตัวอย่างที่ 12: Abstract Factory Pattern

```python
from abc import ABC, abstractmethod

# ============================================================
# Abstract Products
# ============================================================

class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def on_click(self) -> str:
        pass

class TextInput(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def get_value(self) -> str:
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def is_checked(self) -> bool:
        pass


# ============================================================
# Concrete Products - Windows Style
# ============================================================

class WindowsButton(Button):
    def render(self):
        return "[Windows Button: ████████████]"
    
    def on_click(self):
        return "Windows button clicked (flat design)"

class WindowsTextInput(TextInput):
    def __init__(self):
        self._value = ""
    
    def render(self):
        return f"[Windows Input: |{self._value}         |]"
    
    def get_value(self):
        return self._value

class WindowsCheckbox(Checkbox):
    def __init__(self):
        self._checked = False
    
    def render(self):
        mark = "☑" if self._checked else "☐"
        return f"{mark} Windows Checkbox"
    
    def is_checked(self):
        return self._checked


# ============================================================
# Concrete Products - macOS Style
# ============================================================

class MacButton(Button):
    def render(self):
        return "( macOS Button )"
    
    def on_click(self):
        return "macOS button clicked (rounded design)"

class MacTextInput(TextInput):
    def __init__(self):
        self._value = ""
    
    def render(self):
        return f"╭──────────────╮\n│ {self._value:15} │\n╰──────────────╯"
    
    def get_value(self):
        return self._value

class MacCheckbox(Checkbox):
    def __init__(self):
        self._checked = False
    
    def render(self):
        mark = "✓" if self._checked else "○"
        return f"{mark} macOS Checkbox"
    
    def is_checked(self):
        return self._checked


# ============================================================
# Abstract Factory
# ============================================================

class UIFactory(ABC):
    """Abstract Factory สำหรับสร้าง UI components"""
    
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_text_input(self) -> TextInput:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass
    
    @property
    @abstractmethod
    def platform_name(self) -> str:
        pass


class WindowsUIFactory(UIFactory):
    @property
    def platform_name(self):
        return "Windows"
    
    def create_button(self):
        return WindowsButton()
    
    def create_text_input(self):
        return WindowsTextInput()
    
    def create_checkbox(self):
        return WindowsCheckbox()


class MacUIFactory(UIFactory):
    @property
    def platform_name(self):
        return "macOS"
    
    def create_button(self):
        return MacButton()
    
    def create_text_input(self):
        return MacTextInput()
    
    def create_checkbox(self):
        return MacCheckbox()


# ============================================================
# Client Code
# ============================================================

class LoginForm:
    """Form ที่ใช้ UI components แต่ไม่รู้ platform"""
    
    def __init__(self, factory: UIFactory):
        self.factory = factory
        self.username_input = factory.create_text_input()
        self.password_input = factory.create_text_input()
        self.remember_me = factory.create_checkbox()
        self.login_btn = factory.create_button()
    
    def render(self):
        print(f"\n=== Login Form ({self.factory.platform_name}) ===")
        print("Username:")
        print(f"  {self.username_input.render()}")
        print("Password:")
        print(f"  {self.password_input.render()}")
        print(f"{self.remember_me.render()}")
        print(f"{self.login_btn.render()}")
    
    def submit(self):
        result = self.login_btn.on_click()
        print(f"Submit: {result}")


def get_factory(platform: str) -> UIFactory:
    """Factory function ที่เลือก factory ตาม platform"""
    factories = {
        "windows": WindowsUIFactory,
        "mac": MacUIFactory,
        "macos": MacUIFactory,
    }
    factory_class = factories.get(platform.lower())
    if not factory_class:
        raise ValueError(f"ไม่รู้จัก platform: {platform}")
    return factory_class()


# ทดสอบ
for platform in ["windows", "mac"]:
    factory = get_factory(platform)
    form = LoginForm(factory)
    form.render()
    form.submit()
```

---

## Template Method Pattern

**Template Method** กำหนดโครงของ algorithm ใน abstract class แต่ปล่อยให้ subclasses implement ขั้นตอนบางขั้น

### ตัวอย่างที่ 13: Template Method Pattern

```python
from abc import ABC, abstractmethod
import time

class DataImporter(ABC):
    """Abstract class ที่กำหนด template สำหรับ data import"""
    
    def import_data(self, source: str) -> list:
        """Template Method - กำหนดขั้นตอนที่แน่นอน"""
        print(f"\n[{type(self).__name__}] เริ่ม import จาก {source}")
        
        # ขั้นตอน 1: เชื่อมต่อ
        self._connect(source)
        
        # ขั้นตอน 2: อ่านข้อมูล
        raw_data = self._read_data()
        print(f"  อ่านได้ {len(raw_data)} records")
        
        # ขั้นตอน 3: Validate (optional - default ผ่านทั้งหมด)
        valid_data = self._validate(raw_data)
        print(f"  ผ่าน validation {len(valid_data)} records")
        
        # ขั้นตอน 4: Transform
        transformed = self._transform(valid_data)
        
        # ขั้นตอน 5: ปิดการเชื่อมต่อ
        self._disconnect()
        
        print(f"  Import สำเร็จ")
        return transformed
    
    @abstractmethod
    def _connect(self, source: str) -> None:
        pass
    
    @abstractmethod
    def _read_data(self) -> list:
        pass
    
    @abstractmethod
    def _transform(self, data: list) -> list:
        pass
    
    def _validate(self, data: list) -> list:
        """Hook method - subclass อาจ override หรือไม่ก็ได้"""
        return data  # default: ผ่านทั้งหมด
    
    def _disconnect(self) -> None:
        """Hook method"""
        pass


class CSVImporter(DataImporter):
    """Import จาก CSV file"""
    
    def __init__(self):
        self._source = None
        self._raw_data = []
    
    def _connect(self, source):
        self._source = source
        print(f"  เปิดไฟล์ CSV: {source}")
    
    def _read_data(self):
        # จำลองการอ่าน CSV
        return [
            {"id": "1", "name": "Alice", "age": "30", "score": "95.5"},
            {"id": "2", "name": "Bob", "age": "-5", "score": "invalid"},
            {"id": "3", "name": "Charlie", "age": "25", "score": "87.3"},
            {"id": "4", "name": "", "age": "20", "score": "70"},
        ]
    
    def _validate(self, data):
        """Override validation: ตรวจสอบ CSV data"""
        valid = []
        for row in data:
            try:
                if not row.get('name'):
                    print(f"  SKIP: ชื่อว่างเปล่า - row {row['id']}")
                    continue
                age = int(row['age'])
                if age < 0 or age > 120:
                    print(f"  SKIP: อายุไม่สมเหตุสมผล - row {row['id']}")
                    continue
                float(row['score'])
                valid.append(row)
            except (ValueError, KeyError) as e:
                print(f"  SKIP: ข้อมูลผิดรูปแบบ - row {row.get('id', '?')}: {e}")
        return valid
    
    def _transform(self, data):
        """แปลง string เป็น proper types"""
        return [{
            "id": int(row["id"]),
            "name": row["name"].strip(),
            "age": int(row["age"]),
            "score": float(row["score"])
        } for row in data]
    
    def _disconnect(self):
        print(f"  ปิดไฟล์ CSV")


class APIImporter(DataImporter):
    """Import จาก REST API"""
    
    def __init__(self, api_key: str):
        self._api_key = api_key
        self._session = None
    
    def _connect(self, source):
        print(f"  เชื่อมต่อ API: {source}")
        print(f"  ใช้ API Key: {self._api_key[:8]}...")
        self._session = f"session-{id(self)}"
    
    def _read_data(self):
        # จำลองการเรียก API
        return [
            {"user_id": 101, "username": "alice", "email": "alice@example.com", "active": True},
            {"user_id": 102, "username": "bob", "email": "bob@example.com", "active": False},
            {"user_id": 103, "username": "charlie", "email": "charlie@example.com", "active": True},
        ]
    
    def _transform(self, data):
        """แปลงจาก API format เป็น internal format"""
        return [{
            "id": item["user_id"],
            "name": item["username"],
            "email": item["email"],
            "status": "active" if item["active"] else "inactive"
        } for item in data]
    
    def _disconnect(self):
        print(f"  ปิด API session: {self._session}")


# ทดสอบ
csv_importer = CSVImporter()
csv_data = csv_importer.import_data("users.csv")
print(f"\nข้อมูลจาก CSV ({len(csv_data)} records):")
for row in csv_data:
    print(f"  {row}")

api_importer = APIImporter("sk-secret-key-12345")
api_data = api_importer.import_data("https://api.example.com/users")
print(f"\nข้อมูลจาก API ({len(api_data)} records):")
for row in api_data:
    print(f"  {row}")
```

---

## ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Payment System

```python
from abc import ABC, abstractmethod
from datetime import datetime
from enum import Enum
import uuid

class PaymentStatus(Enum):
    PENDING = "รอดำเนินการ"
    PROCESSING = "กำลังดำเนินการ"
    COMPLETED = "สำเร็จ"
    FAILED = "ล้มเหลว"
    REFUNDED = "คืนเงินแล้ว"

class Transaction:
    """Transaction record"""
    
    def __init__(self, transaction_id, amount, payment_method, description=""):
        self.transaction_id = transaction_id
        self.amount = amount
        self.payment_method = payment_method
        self.description = description
        self.status = PaymentStatus.PENDING
        self.created_at = datetime.now()
        self.completed_at = None
        self.error_message = None
    
    def complete(self):
        self.status = PaymentStatus.COMPLETED
        self.completed_at = datetime.now()
    
    def fail(self, error):
        self.status = PaymentStatus.FAILED
        self.error_message = error
        self.completed_at = datetime.now()
    
    def __str__(self):
        return (f"Transaction({self.transaction_id[:8]}, "
                f"{self.amount:,.2f}, {self.status.value})")


class PaymentMethod(ABC):
    """Abstract base class สำหรับทุก payment method"""
    
    def __init__(self, name: str, fee_percent: float = 0):
        self.name = name
        self.fee_percent = fee_percent
        self._transaction_history = []
    
    @property
    @abstractmethod
    def payment_type(self) -> str:
        pass
    
    @abstractmethod
    def _validate_payment(self, amount: float) -> tuple:
        """
        Validate payment details
        Returns: (is_valid: bool, error_message: str)
        """
        pass
    
    @abstractmethod
    def _process_payment(self, amount: float, description: str) -> bool:
        """
        Process the actual payment
        Returns: True if successful, False otherwise
        """
        pass
    
    @abstractmethod
    def _process_refund(self, transaction_id: str, amount: float) -> bool:
        pass
    
    def calculate_fee(self, amount: float) -> float:
        return amount * self.fee_percent / 100
    
    def pay(self, amount: float, description: str = "") -> Transaction:
        """Template Method สำหรับ payment"""
        
        transaction = Transaction(
            transaction_id=str(uuid.uuid4()),
            amount=amount,
            payment_method=self.name,
            description=description
        )
        
        print(f"\n[{self.name}] เริ่มชำระเงิน {amount:,.2f} บาท")
        
        # Validate
        is_valid, error = self._validate_payment(amount)
        if not is_valid:
            transaction.fail(error)
            print(f"  ✗ Validation ล้มเหลว: {error}")
            self._transaction_history.append(transaction)
            return transaction
        
        transaction.status = PaymentStatus.PROCESSING
        
        # Add fee
        fee = self.calculate_fee(amount)
        total = amount + fee
        if fee > 0:
            print(f"  ค่าธรรมเนียม: {fee:.2f} บาท (ชำระรวม: {total:.2f} บาท)")
        
        # Process
        success = self._process_payment(total, description)
        
        if success:
            transaction.complete()
            print(f"  ✓ ชำระเงินสำเร็จ! Transaction: {transaction.transaction_id[:8]}")
        else:
            transaction.fail("การประมวลผลล้มเหลว")
            print(f"  ✗ การชำระเงินล้มเหลว")
        
        self._transaction_history.append(transaction)
        return transaction
    
    def refund(self, transaction_id: str) -> bool:
        """ค้นหา transaction และทำ refund"""
        txn = next((t for t in self._transaction_history 
                   if t.transaction_id == transaction_id), None)
        
        if not txn:
            print(f"ไม่พบ transaction: {transaction_id}")
            return False
        
        if txn.status != PaymentStatus.COMPLETED:
            print(f"ไม่สามารถคืนเงินได้ (status: {txn.status.value})")
            return False
        
        success = self._process_refund(transaction_id, txn.amount)
        if success:
            txn.status = PaymentStatus.REFUNDED
            print(f"คืนเงินสำเร็จ: {txn.amount:,.2f} บาท")
        return success
    
    def get_history(self):
        print(f"\nประวัติ {self.name}:")
        print(f"{'Transaction ID':10} {'จำนวนเงิน':>12} {'สถานะ':12} {'คำอธิบาย'}")
        print("-" * 60)
        for t in self._transaction_history:
            print(f"{t.transaction_id[:8]:10} {t.amount:>12,.2f} "
                  f"{t.status.value:12} {t.description[:20]}")
    
    def __str__(self):
        return f"{self.payment_type}: {self.name}"


class CreditCard(PaymentMethod):
    """Credit Card payment"""
    
    def __init__(self, card_number: str, cardholder: str, 
                 expiry: str, cvv: str, credit_limit: float):
        super().__init__(f"Credit Card ({card_number[-4:]})", fee_percent=1.5)
        self._card_number = card_number
        self._cardholder = cardholder
        self._expiry = expiry
        self._cvv = cvv
        self._credit_limit = credit_limit
        self._used_credit = 0
    
    @property
    def payment_type(self):
        return "บัตรเครดิต"
    
    @property
    def available_credit(self):
        return self._credit_limit - self._used_credit
    
    def _validate_payment(self, amount):
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        total = amount * (1 + self.fee_percent/100)
        if total > self.available_credit:
            return False, f"วงเงินไม่พอ (มี {self.available_credit:,.2f} บาท)"
        return True, ""
    
    def _process_payment(self, amount, description):
        # จำลองการส่งข้อมูลไป payment gateway
        print(f"  กำลังติดต่อ Bank...")
        self._used_credit += amount
        return True
    
    def _process_refund(self, transaction_id, amount):
        self._used_credit -= amount
        return True


class PromptPay(PaymentMethod):
    """PromptPay / QR Code payment"""
    
    def __init__(self, promptpay_id: str):
        super().__init__(f"PromptPay ({promptpay_id})", fee_percent=0)
        self._promptpay_id = promptpay_id
        self._max_per_transaction = 500000
    
    @property
    def payment_type(self):
        return "PromptPay"
    
    def _validate_payment(self, amount):
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        if amount > self._max_per_transaction:
            return False, f"เกินวงเงินต่อครั้ง ({self._max_per_transaction:,} บาท)"
        return True, ""
    
    def _process_payment(self, amount, description):
        print(f"  สร้าง QR Code สำหรับ {amount:,.2f} บาท...")
        print(f"  รอการสแกน QR Code...")
        return True
    
    def _process_refund(self, transaction_id, amount):
        print(f"  โอนเงินคืนผ่าน PromptPay: {amount:,.2f} บาท")
        return True


class Cryptocurrency(PaymentMethod):
    """Cryptocurrency payment"""
    
    def __init__(self, wallet_address: str, currency: str = "BTC"):
        super().__init__(f"{currency} Wallet", fee_percent=0.5)
        self._wallet = wallet_address
        self._currency = currency
        self._exchange_rate = 1500000 if currency == "BTC" else 75000  # THB
    
    @property
    def payment_type(self):
        return f"Cryptocurrency ({self._currency})"
    
    def _validate_payment(self, amount):
        crypto_amount = amount / self._exchange_rate
        if crypto_amount < 0.0001:
            return False, "จำนวนน้อยเกินไปสำหรับ crypto transaction"
        return True, ""
    
    def _process_payment(self, amount, description):
        crypto_amount = amount / self._exchange_rate
        print(f"  โอน {crypto_amount:.6f} {self._currency} ไป {self._wallet[:10]}...")
        return True
    
    def _process_refund(self, transaction_id, amount):
        print(f"  โอน {self._currency} คืน")
        return True


# =================== ทดสอบ Payment System ===================

print("=" * 60)
print("ระบบชำระเงิน")
print("=" * 60)

# สร้าง payment methods
cc = CreditCard("4532123456789012", "SOMCHAI JAIDEE", "12/26", "123", 100000)
pp = PromptPay("081-234-5678")
btc = Cryptocurrency("bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh", "BTC")

payments = [cc, pp, btc]

# ทดสอบชำระเงิน
print("\n=== ชำระเงิน ===")
t1 = cc.pay(1500, "ซื้อหนังสือ Python")
t2 = pp.pay(299, "ค่าสมัครสมาชิก")
t3 = btc.pay(5000, "NFT Purchase")

# ทดสอบ edge cases
print("\n=== ทดสอบ Edge Cases ===")
cc.pay(200000, "ซื้อของแพง - เกินวงเงิน")  # ควร fail
pp.pay(1000000, "เกินวงเงินต่อครั้ง")  # ควร fail

# Refund
print("\n=== Refund ===")
cc.refund(t1.transaction_id)

# History
print("\n=== ประวัติ ===")
cc.get_history()
pp.get_history()
```

### โปรแกรมที่ 2: Shape Renderer System

```python
from abc import ABC, abstractmethod
from typing import List

class Shape(ABC):
    """Abstract shape"""
    
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass
    
    @property
    @abstractmethod
    def name(self) -> str:
        pass


class Renderer(ABC):
    """Abstract renderer"""
    
    @abstractmethod
    def render_circle(self, shape) -> str:
        pass
    
    @abstractmethod
    def render_rectangle(self, shape) -> str:
        pass
    
    @abstractmethod
    def render_triangle(self, shape) -> str:
        pass
    
    def render(self, shape: Shape) -> str:
        """Dispatch ไปยัง method ที่เหมาะสม"""
        # Double dispatch pattern
        render_method = getattr(self, f"render_{shape.name.lower()}", None)
        if render_method:
            return render_method(shape)
        return f"[{type(self).__name__}] ไม่รองรับ {shape.name}"


import math

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    @property
    def name(self):
        return "circle"
    
    def area(self):
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    @property
    def name(self):
        return "rectangle"
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)


class Triangle(Shape):
    def __init__(self, a, b, c):
        self.a = a
        self.b = b
        self.c = c
    
    @property
    def name(self):
        return "triangle"
    
    def area(self):
        s = self.perimeter() / 2
        return math.sqrt(s * (s-self.a) * (s-self.b) * (s-self.c))
    
    def perimeter(self):
        return self.a + self.b + self.c


class TextRenderer(Renderer):
    """Render shapes เป็น text"""
    
    def render_circle(self, shape):
        return (f"Circle: radius={shape.radius:.2f}, "
                f"area={shape.area():.2f}, perimeter={shape.perimeter():.2f}")
    
    def render_rectangle(self, shape):
        return (f"Rectangle: {shape.width}x{shape.height}, "
                f"area={shape.area():.2f}, perimeter={shape.perimeter():.2f}")
    
    def render_triangle(self, shape):
        return (f"Triangle: sides=({shape.a},{shape.b},{shape.c}), "
                f"area={shape.area():.2f}, perimeter={shape.perimeter():.2f}")


class SVGRenderer(Renderer):
    """Render shapes เป็น SVG"""
    
    def __init__(self, width=500, height=500):
        self.width = width
        self.height = height
    
    def render_circle(self, shape):
        cx = self.width // 2
        cy = self.height // 2
        r = int(shape.radius * 10)
        return (f'<circle cx="{cx}" cy="{cy}" r="{r}" '
                f'fill="lightblue" stroke="navy" stroke-width="2"/>')
    
    def render_rectangle(self, shape):
        x = (self.width - shape.width * 10) // 2
        y = (self.height - shape.height * 10) // 2
        return (f'<rect x="{x}" y="{y}" '
                f'width="{int(shape.width * 10)}" height="{int(shape.height * 10)}" '
                f'fill="lightgreen" stroke="darkgreen" stroke-width="2"/>')
    
    def render_triangle(self, shape):
        cx = self.width // 2
        cy = self.height // 2
        return (f'<polygon points="{cx},{cy-50} {cx-40},{cy+30} {cx+40},{cy+30}" '
                f'fill="lightyellow" stroke="orange" stroke-width="2"/>')
    
    def render_all(self, shapes: List[Shape]) -> str:
        parts = [f'<svg width="{self.width}" height="{self.height}">']
        for shape in shapes:
            parts.append(f"  {self.render(shape)}")
        parts.append("</svg>")
        return "\n".join(parts)


class JSONRenderer(Renderer):
    """Render shapes เป็น JSON"""
    
    def render_circle(self, shape):
        import json
        return json.dumps({
            "type": "circle",
            "radius": shape.radius,
            "area": round(shape.area(), 4),
            "perimeter": round(shape.perimeter(), 4)
        })
    
    def render_rectangle(self, shape):
        import json
        return json.dumps({
            "type": "rectangle",
            "width": shape.width,
            "height": shape.height,
            "area": shape.area(),
            "perimeter": shape.perimeter()
        })
    
    def render_triangle(self, shape):
        import json
        return json.dumps({
            "type": "triangle",
            "sides": [shape.a, shape.b, shape.c],
            "area": round(shape.area(), 4),
            "perimeter": shape.perimeter()
        })


# =================== ทดสอบ Shape Renderer ===================

shapes = [
    Circle(5),
    Rectangle(4, 6),
    Triangle(3, 4, 5),
]

renderers = [
    ("Text", TextRenderer()),
    ("SVG", SVGRenderer()),
    ("JSON", JSONRenderer()),
]

for renderer_name, renderer in renderers:
    print(f"\n=== {renderer_name} Renderer ===")
    for shape in shapes:
        print(renderer.render(shape))

# SVG document
print("\n=== SVG Document ===")
svg_renderer = SVGRenderer(400, 300)
print(svg_renderer.render_all(shapes))
```

### โปรแกรมที่ 3: Plugin System

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Optional
import importlib

class Plugin(ABC):
    """Abstract base class สำหรับทุก plugin"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """ชื่อ plugin"""
        pass
    
    @property
    @abstractmethod
    def version(self) -> str:
        """เวอร์ชัน plugin"""
        pass
    
    @property
    @abstractmethod
    def description(self) -> str:
        """คำอธิบาย plugin"""
        pass
    
    @abstractmethod
    def initialize(self, config: dict) -> bool:
        """เริ่มต้น plugin ด้วย configuration"""
        pass
    
    @abstractmethod
    def execute(self, data: dict) -> dict:
        """ทำงาน - รับ data dict และ return result dict"""
        pass
    
    @abstractmethod
    def teardown(self) -> None:
        """ปิด plugin"""
        pass
    
    def __str__(self):
        return f"Plugin({self.name} v{self.version})"


class ProcessorPlugin(Plugin):
    """Plugin base สำหรับ data processing"""
    
    @abstractmethod
    def process(self, data) -> dict:
        pass
    
    def execute(self, data: dict) -> dict:
        """Wrapper ที่เรียก process()"""
        try:
            result = self.process(data.get("payload"))
            return {"status": "success", "result": result, "plugin": self.name}
        except Exception as e:
            return {"status": "error", "error": str(e), "plugin": self.name}


class TextCleanerPlugin(ProcessorPlugin):
    """Plugin สำหรับ clean text"""
    
    def __init__(self):
        self._config = {}
    
    @property
    def name(self):
        return "text-cleaner"
    
    @property
    def version(self):
        return "1.0.0"
    
    @property
    def description(self):
        return "Clean and normalize text data"
    
    def initialize(self, config):
        self._config = {
            "lowercase": config.get("lowercase", True),
            "remove_extra_spaces": config.get("remove_extra_spaces", True),
            "strip": config.get("strip", True),
        }
        print(f"  {self.name}: initialized with {self._config}")
        return True
    
    def process(self, data):
        if not isinstance(data, str):
            raise TypeError("data ต้องเป็น string")
        
        result = data
        if self._config.get("strip"):
            result = result.strip()
        if self._config.get("lowercase"):
            result = result.lower()
        if self._config.get("remove_extra_spaces"):
            import re
            result = re.sub(r'\s+', ' ', result)
        return result
    
    def teardown(self):
        print(f"  {self.name}: teardown")


class NumberFormatterPlugin(ProcessorPlugin):
    """Plugin สำหรับ format ตัวเลข"""
    
    def __init__(self):
        self._decimal_places = 2
        self._thousands_separator = True
    
    @property
    def name(self):
        return "number-formatter"
    
    @property
    def version(self):
        return "2.1.0"
    
    @property
    def description(self):
        return "Format numbers with proper separators and decimal places"
    
    def initialize(self, config):
        self._decimal_places = config.get("decimal_places", 2)
        self._thousands_separator = config.get("thousands_separator", True)
        print(f"  {self.name}: initialized")
        return True
    
    def process(self, data):
        num = float(data)
        if self._thousands_separator:
            return f"{num:,.{self._decimal_places}f}"
        return f"{num:.{self._decimal_places}f}"
    
    def teardown(self):
        print(f"  {self.name}: teardown")


class ValidationPlugin(Plugin):
    """Plugin สำหรับ validate data"""
    
    def __init__(self):
        self._rules = []
    
    @property
    def name(self):
        return "validator"
    
    @property
    def version(self):
        return "1.5.0"
    
    @property
    def description(self):
        return "Validate data against configurable rules"
    
    def initialize(self, config):
        self._rules = config.get("rules", [])
        print(f"  {self.name}: loaded {len(self._rules)} rules")
        return True
    
    def execute(self, data: dict) -> dict:
        payload = data.get("payload")
        errors = []
        
        for rule in self._rules:
            rule_type = rule.get("type")
            
            if rule_type == "required":
                if payload is None or payload == "":
                    errors.append("ข้อมูลจำเป็น")
            
            elif rule_type == "min_length":
                min_len = rule.get("value", 0)
                if isinstance(payload, str) and len(payload) < min_len:
                    errors.append(f"ความยาวต้องมีอย่างน้อย {min_len} ตัวอักษร")
            
            elif rule_type == "max_length":
                max_len = rule.get("value", 9999)
                if isinstance(payload, str) and len(payload) > max_len:
                    errors.append(f"ความยาวต้องไม่เกิน {max_len} ตัวอักษร")
            
            elif rule_type == "numeric":
                try:
                    float(payload)
                except (TypeError, ValueError):
                    errors.append("ต้องเป็นตัวเลข")
        
        if errors:
            return {"status": "invalid", "errors": errors, "plugin": self.name}
        return {"status": "valid", "plugin": self.name}
    
    def teardown(self):
        print(f"  {self.name}: teardown")


class PluginManager:
    """จัดการ plugins ทั้งหมด"""
    
    def __init__(self):
        self._plugins: Dict[str, Plugin] = {}
        self._initialized = set()
    
    def register(self, plugin: Plugin) -> None:
        if not isinstance(plugin, Plugin):
            raise TypeError("ต้องเป็น Plugin instance")
        self._plugins[plugin.name] = plugin
        print(f"ลงทะเบียน plugin: {plugin}")
    
    def initialize_all(self, configs: Dict[str, dict] = None) -> None:
        configs = configs or {}
        print("\nInitializing plugins:")
        for name, plugin in self._plugins.items():
            config = configs.get(name, {})
            success = plugin.initialize(config)
            if success:
                self._initialized.add(name)
    
    def execute(self, plugin_name: str, data: dict) -> dict:
        plugin = self._plugins.get(plugin_name)
        if not plugin:
            return {"status": "error", "error": f"ไม่พบ plugin: {plugin_name}"}
        if plugin_name not in self._initialized:
            return {"status": "error", "error": f"Plugin ยังไม่ถูก initialize: {plugin_name}"}
        return plugin.execute(data)
    
    def execute_pipeline(self, plugins: List[str], initial_data) -> dict:
        """รัน plugins เป็น pipeline"""
        current_data = initial_data
        
        for plugin_name in plugins:
            result = self.execute(plugin_name, {"payload": current_data})
            if result.get("status") not in ["success", "valid"]:
                return result
            if "result" in result:
                current_data = result["result"]
        
        return {"status": "success", "result": current_data}
    
    def teardown_all(self) -> None:
        print("\nTeardown plugins:")
        for plugin in self._plugins.values():
            plugin.teardown()
        self._initialized.clear()
    
    def list_plugins(self) -> None:
        print("\nPlugins ที่ลงทะเบียน:")
        for plugin in self._plugins.values():
            status = "✓" if plugin.name in self._initialized else "○"
            print(f"  {status} {plugin} - {plugin.description}")


# =================== ทดสอบ Plugin System ===================

# สร้าง manager
manager = PluginManager()

# ลงทะเบียน plugins
manager.register(TextCleanerPlugin())
manager.register(NumberFormatterPlugin())
manager.register(ValidationPlugin())

# Initialize พร้อม configurations
manager.initialize_all({
    "text-cleaner": {
        "lowercase": True,
        "remove_extra_spaces": True
    },
    "number-formatter": {
        "decimal_places": 2,
        "thousands_separator": True
    },
    "validator": {
        "rules": [
            {"type": "required"},
            {"type": "min_length", "value": 3},
            {"type": "max_length", "value": 100},
        ]
    }
})

manager.list_plugins()

# ทดสอบ individual plugins
print("\n=== ทดสอบ Individual Plugins ===")

r1 = manager.execute("text-cleaner", {"payload": "  HELLO   WORLD  "})
print(f"Text cleaned: {r1}")

r2 = manager.execute("number-formatter", {"payload": 1234567.89})
print(f"Number formatted: {r2}")

r3 = manager.execute("validator", {"payload": "Hello"})
print(f"Validation: {r3}")

r4 = manager.execute("validator", {"payload": "Hi"})  # สั้นเกิน
print(f"Validation (fail): {r4}")

# ทดสอบ pipeline
print("\n=== ทดสอบ Pipeline ===")
pipeline = ["text-cleaner"]
result = manager.execute_pipeline(pipeline, "  PYTHON Programming   ")
print(f"Pipeline result: {result['result']}")

# Teardown
manager.teardown_all()
```

---

## แบบฝึกหัด

### ข้อที่ 1: Abstract File Handler

```python
# สร้าง abstract file handler hierarchy:
# - FileHandler(ABC)
#   - TextFileHandler
#   - JSONFileHandler
#   - CSVFileHandler
# - methods: open(), read(), write(), close()
# - ใช้ context manager (__enter__, __exit__)

# เฉลย
from abc import ABC, abstractmethod
import json, csv
from io import StringIO

class FileHandler(ABC):
    """Abstract file handler"""
    
    def __init__(self, filepath: str):
        self.filepath = filepath
        self._file = None
        self._is_open = False
    
    @abstractmethod
    def open(self, mode: str = 'r') -> None:
        pass
    
    @abstractmethod
    def read(self):
        pass
    
    @abstractmethod
    def write(self, data) -> None:
        pass
    
    @abstractmethod
    def close(self) -> None:
        pass
    
    @property
    def is_open(self):
        return self._is_open
    
    def __enter__(self):
        self.open()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
        return False  # ไม่ suppress exceptions
    
    def _check_open(self):
        if not self._is_open:
            raise IOError("ต้องเปิดไฟล์ก่อน")


class TextFileHandler(FileHandler):
    def open(self, mode='r'):
        try:
            self._file = open(self.filepath, mode, encoding='utf-8')
            self._is_open = True
            print(f"เปิดไฟล์ text: {self.filepath}")
        except FileNotFoundError:
            if 'w' in mode:
                self._file = open(self.filepath, mode, encoding='utf-8')
                self._is_open = True
            else:
                raise
    
    def read(self) -> str:
        self._check_open()
        return self._file.read()
    
    def write(self, data: str) -> None:
        self._check_open()
        self._file.write(data)
    
    def close(self) -> None:
        if self._file:
            self._file.close()
        self._is_open = False
        print(f"ปิดไฟล์: {self.filepath}")


class JSONFileHandler(FileHandler):
    def open(self, mode='r'):
        self._mode = mode
        self._is_open = True
        self._data = None
        
        if 'r' in mode:
            try:
                with open(self.filepath, 'r', encoding='utf-8') as f:
                    self._data = json.load(f)
                print(f"โหลด JSON: {self.filepath}")
            except FileNotFoundError:
                self._data = {}
    
    def read(self):
        self._check_open()
        return self._data
    
    def write(self, data) -> None:
        self._check_open()
        self._data = data
    
    def close(self) -> None:
        if self._is_open and self._data is not None and 'w' in getattr(self, '_mode', ''):
            with open(self.filepath, 'w', encoding='utf-8') as f:
                json.dump(self._data, f, ensure_ascii=False, indent=2)
            print(f"บันทึก JSON: {self.filepath}")
        self._is_open = False


# ทดสอบ (ใช้ StringIO แทนไฟล์จริง)
print("=== ทดสอบ Context Manager ===")
import tempfile, os

# สร้างไฟล์ชั่วคราว
with tempfile.NamedTemporaryFile(mode='w', suffix='.txt', delete=False) as f:
    f.write("Hello, World!\nPython is great!")
    tmpfile = f.name

# ทดสอบอ่าน
with TextFileHandler(tmpfile) as handler:
    content = handler.read()
    print(f"เนื้อหา: {content}")

os.unlink(tmpfile)  # ลบไฟล์ชั่วคราว
```

### ข้อที่ 2 - 10: โจทย์ฝึกหัดเพิ่มเติม

```python
# ข้อที่ 2: Observer Pattern
# - Observer(ABC) -> EventLogger, EmailNotifier, SMSNotifier
# - Subject(ABC) -> EventSystem
# - Methods: attach(), detach(), notify()

# ข้อที่ 3: Command Pattern
# - Command(ABC) -> CreateCommand, UpdateCommand, DeleteCommand
# - execute(), undo()
# - CommandHistory class

# ข้อที่ 4: State Pattern
# - State(ABC) -> PendingState, ActiveState, SuspendedState, ClosedState
# - Account class ที่มี state

# ข้อที่ 5: Decorator Pattern
# - Component(ABC) -> ConcreteComponent
# - Decorator(ABC, Component) -> LoggingDecorator, CacheDecorator, RetryDecorator

# ข้อที่ 6: Iterator Pattern
# - Iterator(ABC) -> ForwardIterator, ReverseIterator, FilterIterator
# - Collection class ที่ return iterators ต่างๆ

# ข้อที่ 7: Repository Pattern
# - Repository(ABC) -> InMemoryRepository, FileRepository
# - CRUD methods: save, find_by_id, find_all, delete

# ข้อที่ 8: Builder Pattern
# - Builder(ABC) -> HouseBuilder, CarBuilder
# - Director class
# - reset(), build_foundation(), build_walls(), get_result()

# ข้อที่ 9: Chain of Responsibility
# - Handler(ABC) -> AuthHandler, ValidationHandler, ProcessingHandler
# - set_next(), handle()

# ข้อที่ 10: Composite Pattern
# - Component(ABC) -> Leaf, Composite
# - สร้าง file system tree
# - add(), remove(), display(), total_size()
```

#### เฉลยข้อที่ 2: Observer Pattern

```python
from abc import ABC, abstractmethod
from typing import List
from datetime import datetime

class Observer(ABC):
    """Abstract Observer"""
    
    @abstractmethod
    def update(self, event_type: str, data: dict) -> None:
        pass

class Subject(ABC):
    """Abstract Subject (Observable)"""
    
    @abstractmethod
    def attach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def detach(self, observer: Observer) -> None:
        pass
    
    @abstractmethod
    def notify(self, event_type: str, data: dict) -> None:
        pass


class EventLogger(Observer):
    def __init__(self, log_file="events.log"):
        self.log_file = log_file
        self.logs = []
    
    def update(self, event_type, data):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        log = f"[{timestamp}] {event_type}: {data}"
        self.logs.append(log)
        print(f"  LOG: {log}")


class EmailNotifier(Observer):
    def __init__(self, email: str):
        self.email = email
        self.sent_emails = []
    
    def update(self, event_type, data):
        important_events = {"user_login_failed", "payment_failed", "account_locked"}
        if event_type in important_events:
            email = {
                "to": self.email,
                "subject": f"Alert: {event_type}",
                "body": str(data)
            }
            self.sent_emails.append(email)
            print(f"  EMAIL sent to {self.email}: {event_type}")


class SecurityMonitor(Observer):
    def __init__(self):
        self.suspicious_activities = []
        self.blocked_ips = set()
    
    def update(self, event_type, data):
        if event_type == "login_failed":
            ip = data.get("ip", "unknown")
            
            # นับความพยายาม login ที่ล้มเหลว
            failed = sum(1 for a in self.suspicious_activities 
                        if a.get("ip") == ip and a.get("type") == "login_failed")
            
            self.suspicious_activities.append({
                "type": event_type, "ip": ip, 
                "time": datetime.now().isoformat()
            })
            
            if failed >= 3:
                self.blocked_ips.add(ip)
                print(f"  SECURITY: Block IP {ip} (ล้มเหลว {failed+1} ครั้ง)")


class UserAccountSystem(Subject):
    """Observable user account system"""
    
    def __init__(self):
        self._observers: List[Observer] = []
        self._users = {}
    
    def attach(self, observer):
        if observer not in self._observers:
            self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self, event_type, data):
        for observer in self._observers:
            observer.update(event_type, data)
    
    def register_user(self, username, password):
        self._users[username] = {"password": password, "locked": False}
        self.notify("user_registered", {"username": username})
        print(f"ลงทะเบียน {username} สำเร็จ")
    
    def login(self, username, password, ip="127.0.0.1"):
        if username not in self._users:
            self.notify("login_failed", {"username": username, "ip": ip, "reason": "not found"})
            return False
        
        user = self._users[username]
        if user.get("locked"):
            self.notify("login_blocked", {"username": username, "ip": ip})
            return False
        
        if user["password"] == password:
            self.notify("login_success", {"username": username, "ip": ip})
            return True
        else:
            self.notify("login_failed", {"username": username, "ip": ip, "reason": "wrong password"})
            return False


# ทดสอบ
system = UserAccountSystem()

# เพิ่ม observers
logger = EventLogger()
notifier = EmailNotifier("admin@example.com")
security = SecurityMonitor()

system.attach(logger)
system.attach(notifier)
system.attach(security)

print("=== ทดสอบ Observer Pattern ===")
system.register_user("alice", "password123")
system.register_user("bob", "securepass")

print("\n--- Login Tests ---")
system.login("alice", "password123", "192.168.1.1")  # สำเร็จ
system.login("alice", "wrong", "10.0.0.1")  # ล้มเหลว
system.login("alice", "wrong", "10.0.0.1")  # ล้มเหลว
system.login("alice", "wrong", "10.0.0.1")  # ล้มเหลว
system.login("alice", "wrong", "10.0.0.1")  # Block!

print(f"\nBlocked IPs: {security.blocked_ips}")
print(f"Emails ส่งแล้ว: {len(notifier.sent_emails)}")
print(f"\nLogs ทั้งหมด: {len(logger.logs)} รายการ")
```

---

## สรุป Part 25

| แนวคิด | Module/Syntax | วัตถุประสงค์ |
|--------|--------------|------------|
| Abstract Class | `class A(ABC)` | บังคับ interface สำหรับ subclasses |
| Abstract Method | `@abstractmethod` | ต้อง implement ใน subclass |
| Abstract Property | `@property @abstractmethod` | Abstract getter/setter |
| Pure Interface | ABC + เฉพาะ abstract methods | กำหนด contract |
| Protocol | `class P(Protocol)` | Structural subtyping |
| @runtime_checkable | decorator | ใช้ isinstance() กับ Protocol |
| Template Method | abstract + concrete methods | กำหนด algorithm skeleton |
| Abstract Factory | factory method pattern | สร้าง families of objects |

### หลักการเลือกใช้

```
ต้องการบังคับให้ implement?
├── ใช่ -> ใช้ ABC + @abstractmethod
└── ไม่ใช่ -> ใช้ Protocol (structural)

ต้องการ default implementation?
├── ใช่ -> ใช้ ABC พร้อม concrete methods
└── ไม่ใช่ -> Pure Abstract Class (interface-like)

ต้องการ isinstance() check?
├── ใช่ + มี hierarchy -> ใช้ ABC
└── ใช่ + ไม่มี hierarchy -> ใช้ @runtime_checkable Protocol
```

**ยินดีด้วย! คุณเรียนจบ Part 21-25 แล้ว!**

---

**ต่อไป**: Part 26 - Advanced OOP Patterns
