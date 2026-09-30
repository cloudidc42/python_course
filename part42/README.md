# Part 42: Protocol, ABC & Structural Subtyping

## บทนำ

ใน Python มีสองวิธีหลักในการกำหนด interfaces และความสัมพันธ์ระหว่าง types:

1. **Nominal Subtyping** (ABC) - class ต้อง explicitly declare ว่า implement interface
2. **Structural Subtyping** (Protocol) - class ถือว่า implement interface ถ้ามี methods/attributes ที่ถูกต้อง (Duck Typing แบบ type-safe)

Python 3.8 แนะนำ `typing.Protocol` ซึ่งนำ structural subtyping มาสู่ type system ของ Python

---

## 1. Protocol Class (typing.Protocol)

### Duck Typing ปกติ

```python
# Python Duck Typing แบบดั้งเดิม - ไม่มี type safety
def make_sound(animal):
    return animal.sound()  # ไม่มี type hint, type checker ไม่รู้ว่าต้องการอะไร

class Dog:
    def sound(self):
        return "Woof!"

class Cat:
    def sound(self):
        return "Meow!"

print(make_sound(Dog()))  # Woof!
print(make_sound(Cat()))  # Meow!
```

### Protocol สำหรับ type-safe duck typing

```python
from typing import Protocol

class Soundable(Protocol):
    def sound(self) -> str:
        ...  # Protocol method ไม่ต้องมี implementation (ใช้ ... หรือ pass)

def make_sound(animal: Soundable) -> str:
    return animal.sound()

class Dog:
    def sound(self) -> str:
        return "Woof!"

class Cat:
    def sound(self) -> str:
        return "Meow!"

class Car:
    def honk(self) -> str:
        return "Beep!"

# Dog และ Cat ผ่านการ type check แม้ไม่ได้ inherit Protocol
dog = Dog()
cat = Cat()
car = Car()

print(make_sound(dog))  # Woof! - OK
print(make_sound(cat))  # Meow! - OK
# make_sound(car)  # mypy จะ report error: Car ไม่มี sound()
```

### Protocol ที่มีหลาย methods

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...
    
    def get_position(self) -> tuple[float, float]:
        ...
    
    def get_size(self) -> tuple[float, float]:
        ...

class Circle:
    def __init__(self, x: float, y: float, radius: float):
        self._x = x
        self._y = y
        self._radius = radius
    
    def draw(self) -> None:
        print(f"Drawing circle at ({self._x}, {self._y}) with radius {self._radius}")
    
    def get_position(self) -> tuple[float, float]:
        return (self._x, self._y)
    
    def get_size(self) -> tuple[float, float]:
        return (self._radius * 2, self._radius * 2)

class Rectangle:
    def __init__(self, x: float, y: float, width: float, height: float):
        self._x = x
        self._y = y
        self._width = width
        self._height = height
    
    def draw(self) -> None:
        print(f"Drawing rectangle at ({self._x}, {self._y}) {self._width}x{self._height}")
    
    def get_position(self) -> tuple[float, float]:
        return (self._x, self._y)
    
    def get_size(self) -> tuple[float, float]:
        return (self._width, self._height)

def render_scene(shapes: list[Drawable]) -> None:
    for shape in shapes:
        shape.draw()
        pos = shape.get_position()
        size = shape.get_size()
        print(f"  Position: {pos}, Size: {size}")

shapes: list[Drawable] = [
    Circle(0, 0, 50),
    Rectangle(10, 20, 100, 50),
    Circle(200, 150, 30)
]

render_scene(shapes)
```

---

## 2. Structural Subtyping vs Nominal Subtyping

### Nominal Subtyping (ABC approach)

```python
from abc import ABC, abstractmethod

# Nominal - class ต้อง explicitly inherit
class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass
    
    @abstractmethod
    def move(self) -> str:
        pass

class Dog(Animal):  # ต้อง inherit Animal
    def speak(self) -> str:
        return "Woof!"
    
    def move(self) -> str:
        return "Running on 4 legs"

class Bird(Animal):  # ต้อง inherit Animal
    def speak(self) -> str:
        return "Tweet!"
    
    def move(self) -> str:
        return "Flying with wings"

# class ที่มี methods แต่ไม่ inherit Animal ไม่ผ่าน type check
class Robot:  # ไม่ inherit Animal
    def speak(self) -> str:
        return "Beep boop"
    
    def move(self) -> str:
        return "Rolling on wheels"

def interact(animal: Animal) -> None:
    print(f"Sound: {animal.speak()}")
    print(f"Move: {animal.move()}")

interact(Dog())   # OK
interact(Bird())  # OK
# interact(Robot())  # type error: Robot ไม่ใช่ Animal
```

### Structural Subtyping (Protocol approach)

```python
from typing import Protocol

# Structural - class ไม่ต้อง explicitly inherit
class AnimalLike(Protocol):
    def speak(self) -> str:
        ...
    
    def move(self) -> str:
        ...

class Dog:  # ไม่ inherit Protocol
    def speak(self) -> str:
        return "Woof!"
    
    def move(self) -> str:
        return "Running on 4 legs"

class Robot:  # ไม่ inherit Protocol
    def speak(self) -> str:
        return "Beep boop"
    
    def move(self) -> str:
        return "Rolling on wheels"

class Statue:  # ไม่มี methods ที่ถูกต้อง
    def display(self) -> str:
        return "Standing still"

def interact(creature: AnimalLike) -> None:
    print(f"Sound: {creature.speak()}")
    print(f"Move: {creature.move()}")

interact(Dog())    # OK - มี speak() และ move()
interact(Robot())  # OK - มี speak() และ move() (structural match!)
# interact(Statue())  # type error: ไม่มี speak() และ move()
```

### เปรียบเทียบ Use Cases

```python
"""
เมื่อไหรควรใช้ ABC (Nominal):
- ต้องการ enforce hierarchy ที่ชัดเจน
- ต้องการ runtime isinstance() checks ที่แน่นอน
- class ทั้งหมดอยู่ใน codebase เดียวกัน
- ต้องการ shared implementation ใน base class
- API ที่ third-party ต้อง implement

เมื่อไหรควรใช้ Protocol (Structural):
- ทำงานกับ third-party libraries ที่ไม่รู้ล่วงหน้า
- ต้องการ "duck typing with type safety"
- ต้องการความยืดหยุ่นมากกว่า
- หลีกเลี่ยง tight coupling
- Retrofit type annotations ให้ existing code
"""
```

---

## 3. @runtime_checkable

ปกติ Protocol ใช้ได้แค่กับ static type checker เท่านั้น แต่ `@runtime_checkable` ทำให้ใช้ `isinstance()` ได้

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None:
        ...

class File:
    def close(self) -> None:
        print("Closing file")

class DatabaseConnection:
    def close(self) -> None:
        print("Closing DB connection")

class Timer:
    def start(self) -> None:
        print("Starting timer")

# ตอนนี้ isinstance() ใช้ได้
file = File()
db = DatabaseConnection()
timer = Timer()

print(isinstance(file, Closeable))   # True
print(isinstance(db, Closeable))     # True
print(isinstance(timer, Closeable))  # False - ไม่มี close()

# ใช้ใน context manager
def safely_close_all(resources: list) -> None:
    for resource in resources:
        if isinstance(resource, Closeable):
            resource.close()
        else:
            print(f"Warning: {type(resource).__name__} is not closeable")

safely_close_all([file, db, timer])
```

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Serializable(Protocol):
    def to_dict(self) -> dict:
        ...
    
    def to_json(self) -> str:
        ...

# ข้อสำคัญ: runtime_checkable ตรวจแค่ว่า method มีอยู่ ไม่ตรวจ signature
class FakeSerializable:
    def to_dict(self):  # ไม่ได้ return dict จริง
        return "not a dict"
    
    def to_json(self):
        return 42  # ไม่ได้ return str จริง

fake = FakeSerializable()
print(isinstance(fake, Serializable))  # True! - เพราะตรวจแค่ชื่อ method
# นี่เป็น limitation ของ runtime_checkable
```

---

## 4. Protocol Inheritance

Protocol สามารถ inherit จาก Protocol อื่นได้ เพื่อสร้าง composed interfaces

```python
from typing import Protocol

class Readable(Protocol):
    def read(self, n: int = -1) -> bytes:
        ...

class Writable(Protocol):
    def write(self, data: bytes) -> int:
        ...

class Seekable(Protocol):
    def seek(self, pos: int) -> int:
        ...
    
    def tell(self) -> int:
        ...

# Composed protocol
class ReadWriteable(Readable, Writable, Protocol):
    ...

class FullFileAccess(Readable, Writable, Seekable, Protocol):
    ...

class InMemoryBuffer:
    def __init__(self):
        self._data = bytearray()
        self._pos = 0
    
    def read(self, n: int = -1) -> bytes:
        if n == -1:
            result = bytes(self._data[self._pos:])
            self._pos = len(self._data)
        else:
            result = bytes(self._data[self._pos:self._pos + n])
            self._pos += n
        return result
    
    def write(self, data: bytes) -> int:
        self._data[self._pos:self._pos + len(data)] = data
        self._pos += len(data)
        return len(data)
    
    def seek(self, pos: int) -> int:
        self._pos = pos
        return self._pos
    
    def tell(self) -> int:
        return self._pos

# InMemoryBuffer สามารถใช้แทน FullFileAccess ได้
def copy_stream(src: Readable, dst: Writable, chunk_size: int = 1024) -> int:
    total = 0
    while True:
        data = src.read(chunk_size)
        if not data:
            break
        dst.write(data)
        total += len(data)
    return total

buf1 = InMemoryBuffer()
buf1.write(b"Hello, World!")
buf1.seek(0)

buf2 = InMemoryBuffer()
bytes_copied = copy_stream(buf1, buf2)
buf2.seek(0)
print(f"Copied {bytes_copied} bytes: {buf2.read()}")
```

---

## 5. ABC vs Protocol

### ABC (Abstract Base Class)

```python
from abc import ABC, abstractmethod
from typing import List

class Repository(ABC):
    """Abstract repository - ต้อง explicitly inherit"""
    
    @abstractmethod
    def find_by_id(self, id: int):
        pass
    
    @abstractmethod
    def find_all(self) -> List:
        pass
    
    @abstractmethod
    def save(self, entity) -> None:
        pass
    
    @abstractmethod
    def delete(self, id: int) -> None:
        pass
    
    # Concrete method ที่ subclass ได้รับ
    def find_or_raise(self, id: int):
        result = self.find_by_id(id)
        if result is None:
            raise ValueError(f"Entity with id {id} not found")
        return result

class UserRepository(Repository):
    def __init__(self):
        self._storage = {}
        self._next_id = 1
    
    def find_by_id(self, id: int):
        return self._storage.get(id)
    
    def find_all(self) -> List:
        return list(self._storage.values())
    
    def save(self, entity) -> None:
        if not hasattr(entity, 'id') or entity.id is None:
            entity.id = self._next_id
            self._next_id += 1
        self._storage[entity.id] = entity
    
    def delete(self, id: int) -> None:
        self._storage.pop(id, None)

# ABC ป้องกันการ instantiate class ที่ไม่ complete
try:
    class IncompleteRepo(Repository):
        def find_by_id(self, id: int):
            return None
        # ลืม implement find_all, save, delete
    
    repo = IncompleteRepo()  # TypeError!
except TypeError as e:
    print(f"Error: {e}")
```

### Protocol

```python
from typing import Protocol, List, TypeVar

T = TypeVar('T')

class RepositoryProtocol(Protocol[T]):
    """Protocol repository - ไม่ต้อง explicitly inherit"""
    
    def find_by_id(self, id: int) -> T | None:
        ...
    
    def find_all(self) -> List[T]:
        ...
    
    def save(self, entity: T) -> None:
        ...
    
    def delete(self, id: int) -> None:
        ...

# สามารถใช้กับ library ที่มีอยู่แล้ว
# ถึงแม้ InMemoryStore ไม่รู้จัก RepositoryProtocol
class InMemoryStore:
    """Third-party class ที่ไม่ inherit Protocol"""
    
    def __init__(self):
        self._data = {}
        self._counter = 1
    
    def find_by_id(self, id: int):
        return self._data.get(id)
    
    def find_all(self):
        return list(self._data.values())
    
    def save(self, entity):
        if not hasattr(entity, 'id') or entity.id is None:
            entity.id = self._counter
            self._counter += 1
        self._data[entity.id] = entity
    
    def delete(self, id: int):
        self._data.pop(id, None)

# InMemoryStore ใช้เป็น RepositoryProtocol ได้
def process_entities(repo: RepositoryProtocol, entity) -> None:
    repo.save(entity)
    print(f"Saved: {repo.find_by_id(entity.id)}")

from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    id: int = None

store = InMemoryStore()
process_entities(store, Product("Laptop", 999.99))
```

---

## 6. Generic Protocols

```python
from typing import Protocol, TypeVar, Generic, Iterator, Iterable

T = TypeVar('T')
T_co = TypeVar('T_co', covariant=True)

class Container(Protocol[T]):
    def get(self) -> T:
        ...
    
    def set(self, value: T) -> None:
        ...
    
    def is_empty(self) -> bool:
        ...

class Stack(Container[T]):
    def __init__(self):
        self._items: list[T] = []
    
    def get(self) -> T:
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self._items[-1]
    
    def set(self, value: T) -> None:
        self._items.append(value)
    
    def is_empty(self) -> bool:
        return len(self._items) == 0
    
    def pop(self) -> T:
        return self._items.pop()

# Generic function ที่ทำงานกับ Container ของ type ใดก็ได้
def transfer(src: Container[T], dst: Container[T]) -> None:
    value = src.get()
    dst.set(value)

int_src: Stack[int] = Stack()
int_src.set(42)
int_dst: Stack[int] = Stack()

transfer(int_src, int_dst)
print(int_dst.get())  # 42
```

### Iterable Protocol

```python
from typing import Protocol, TypeVar, Iterator

T_co = TypeVar('T_co', covariant=True)

class Iterable(Protocol[T_co]):
    def __iter__(self) -> Iterator[T_co]:
        ...

class IterableRange:
    """Custom iterable ที่ implement Protocol โดยไม่รู้จัก Protocol"""
    
    def __init__(self, start: int, stop: int, step: int = 1):
        self.start = start
        self.stop = stop
        self.step = step
    
    def __iter__(self):
        current = self.start
        while current < self.stop:
            yield current
            current += self.step

def sum_iterable(items: Iterable[int]) -> int:
    return sum(items)

r = IterableRange(1, 10, 2)  # 1, 3, 5, 7, 9
print(sum_iterable(r))  # 25
print(sum_iterable([1, 2, 3, 4, 5]))  # 15 - list ก็ใช้ได้
```

---

## 7. Covariance และ Contravariance

### Covariance (T_co)

```python
from typing import TypeVar, Generic

T_co = TypeVar('T_co', covariant=True)  # covariant: Producer[Cat] ใช้แทน Producer[Animal] ได้

class Animal:
    def breathe(self):
        return "breathing"

class Dog(Animal):
    def bark(self):
        return "Woof!"

class Cat(Animal):
    def meow(self):
        return "Meow!"

class Producer(Generic[T_co]):
    """Covariant: ผลิต T เท่านั้น ไม่รับ T เข้า"""
    
    def __init__(self, item: T_co):
        self._item = item
    
    def produce(self) -> T_co:
        return self._item

# Covariance: Producer[Dog] เป็น subtype ของ Producer[Animal]
def feed_animal(producer: Producer[Animal]) -> None:
    animal = producer.produce()
    print(f"Feeding: {type(animal).__name__}")

dog_producer: Producer[Dog] = Producer(Dog())
feed_animal(dog_producer)  # OK! เพราะ Producer เป็น covariant

# หากไม่มี covariance จะ type error
```

### Contravariance (T_contra)

```python
from typing import TypeVar, Generic

T_contra = TypeVar('T_contra', contravariant=True)  # Consumer[Animal] ใช้แทน Consumer[Dog] ได้

class Consumer(Generic[T_contra]):
    """Contravariant: รับ T เท่านั้น ไม่ produce T"""
    
    def consume(self, item: T_contra) -> None:
        print(f"Consuming: {type(item).__name__}")

def give_dog_to(consumer: Consumer[Dog]) -> None:
    dog = Dog()
    consumer.consume(dog)

# Contravariance: Consumer[Animal] เป็น subtype ของ Consumer[Dog]
animal_consumer: Consumer[Animal] = Consumer()
give_dog_to(animal_consumer)  # OK! เพราะ Consumer เป็น contravariant

# Consumer[Animal] รับ Dog ได้เพราะ Dog เป็น Animal
```

### Invariance (ปกติ)

```python
from typing import TypeVar, Generic, List

T = TypeVar('T')  # invariant

class Box(Generic[T]):
    """Invariant: ทั้ง produce และ consume T"""
    
    def __init__(self):
        self._items: List[T] = []
    
    def put(self, item: T) -> None:
        self._items.append(item)
    
    def get(self) -> T:
        return self._items[-1]

# Box[Dog] ไม่ใช่ subtype ของ Box[Animal] และ ไม่ใช่ supertype
# เพราะถ้า Box[Dog] เป็น subtype ของ Box[Animal]
# เราอาจ put Cat เข้า Box[Dog] ซึ่งผิด!
dog_box: Box[Dog] = Box[Dog]()
# animal_box: Box[Animal] = dog_box  # type error
```

---

## 8. TypeVar Bounds

```python
from typing import TypeVar, Protocol
from abc import ABC, abstractmethod

# TypeVar กับ bound - T ต้องเป็น subtype ของ bound
class Comparable(Protocol):
    def __lt__(self, other: 'Comparable') -> bool:
        ...
    def __le__(self, other: 'Comparable') -> bool:
        ...

T = TypeVar('T', bound=Comparable)

def minimum(a: T, b: T) -> T:
    """ทำงานกับ type ใดก็ได้ที่ comparable"""
    return a if a < b else b

print(minimum(3, 5))       # 3
print(minimum(3.14, 2.71)) # 2.71
print(minimum("apple", "banana"))  # apple

# TypeVar กับ constraints - T ต้องเป็นหนึ่งใน types ที่กำหนด
NumberT = TypeVar('NumberT', int, float, complex)

def add(a: NumberT, b: NumberT) -> NumberT:
    return a + b

print(add(1, 2))       # 3 (int)
print(add(1.5, 2.5))  # 4.0 (float)
# add(1, 2.5)  # type error: int และ float คนละ type
```

```python
from typing import TypeVar

# TypeVar หลายตัว
K = TypeVar('K')
V = TypeVar('V')

def swap_dict(d: dict[K, V]) -> dict[V, K]:
    """Swap keys และ values"""
    return {v: k for k, v in d.items()}

original = {"a": 1, "b": 2, "c": 3}
swapped = swap_dict(original)
print(swapped)  # {1: 'a', 2: 'b', 3: 'c'}
```

---

## 9. ParamSpec

`ParamSpec` (Python 3.10+) ใช้สำหรับ capture parameter specifications ของ functions

```python
from typing import TypeVar, Callable
from typing import ParamSpec
import functools
import time

P = ParamSpec('P')
T = TypeVar('T')

def timer(func: Callable[P, T]) -> Callable[P, T]:
    """Decorator ที่ preserve type signature"""
    
    @functools.wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    
    return wrapper

@timer
def greet(name: str, greeting: str = "Hello") -> str:
    time.sleep(0.01)
    return f"{greeting}, {name}!"

# Type checker รู้ว่า greet() มี signature เดิม
result = greet("Alice")          # "Hello, Alice!"
result = greet("Bob", "Hi")      # "Hi, Bob!"
# greet(123)  # type error: name ต้องเป็น str
```

```python
from typing import TypeVar, Callable, ParamSpec
import logging
import functools

P = ParamSpec('P')
T = TypeVar('T')

def logged(logger: logging.Logger) -> Callable[[Callable[P, T]], Callable[P, T]]:
    """Decorator factory ที่มี parameter"""
    
    def decorator(func: Callable[P, T]) -> Callable[P, T]:
        @functools.wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
            logger.info(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
            try:
                result = func(*args, **kwargs)
                logger.info(f"{func.__name__} returned {result}")
                return result
            except Exception as e:
                logger.error(f"{func.__name__} raised {type(e).__name__}: {e}")
                raise
        return wrapper
    return decorator

app_logger = logging.getLogger("app")
logging.basicConfig(level=logging.INFO)

@logged(app_logger)
def calculate(x: int, y: int, operation: str = "add") -> int:
    if operation == "add":
        return x + y
    elif operation == "multiply":
        return x * y
    raise ValueError(f"Unknown operation: {operation}")

calculate(3, 4)
calculate(3, 4, "multiply")
```

---

## 10. Concatenate

`Concatenate` ใช้กับ `ParamSpec` สำหรับ decorators ที่เพิ่ม/ลด parameters

```python
from typing import TypeVar, Callable, Concatenate
from typing import ParamSpec

P = ParamSpec('P')
T = TypeVar('T')

# Decorator ที่เพิ่ม parameter
def with_user(
    func: Callable[Concatenate[str, P], T]
) -> Callable[P, T]:
    """Inject user parameter อัตโนมัติ"""
    
    @functools.wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        # inject "current_user" เป็น first argument
        return func("current_user", *args, **kwargs)
    
    return wrapper

import functools

@with_user
def get_profile(user: str, include_details: bool = False) -> dict:
    return {
        "user": user,
        "details": "..." if include_details else None
    }

# ตอนเรียก ไม่ต้องระบุ user
profile = get_profile()
print(profile)  # {'user': 'current_user', 'details': None}

profile_with_details = get_profile(include_details=True)
print(profile_with_details)
```

---

## 11. TypeGuard

`TypeGuard` (Python 3.10+) ใช้สำหรับ type narrowing ใน custom type guard functions

```python
from typing import TypeGuard, Union, List, Any

def is_string_list(val: List[Any]) -> TypeGuard[List[str]]:
    """Type guard: ตรวจว่า list ทุกตัวเป็น str"""
    return all(isinstance(x, str) for x in val)

def process_strings(items: List[Any]) -> None:
    if is_string_list(items):
        # ใน block นี้ type checker รู้ว่า items เป็น List[str]
        joined = ", ".join(items)  # OK - ไม่มี error
        print(f"Strings: {joined}")
    else:
        print("Not all strings")

process_strings(["hello", "world"])  # Strings: hello, world
process_strings([1, 2, 3])          # Not all strings
process_strings(["a", 1, "b"])      # Not all strings
```

```python
from typing import TypeGuard, Union
from dataclasses import dataclass

@dataclass
class Success:
    value: str
    
@dataclass
class Error:
    message: str
    code: int

Result = Union[Success, Error]

def is_success(result: Result) -> TypeGuard[Success]:
    return isinstance(result, Success)

def handle_result(result: Result) -> str:
    if is_success(result):
        # type narrowed to Success
        return f"Success: {result.value}"
    else:
        # type narrowed to Error
        return f"Error {result.code}: {result.message}"

results: list[Result] = [
    Success("Data loaded"),
    Error("Not found", 404),
    Success("Updated"),
    Error("Server error", 500)
]

for r in results:
    print(handle_result(r))
```

---

## 12. Protocol Best Practices

### Best Practice 1: Protocol ควรมีขนาดเล็ก (Interface Segregation)

```python
from typing import Protocol

# ไม่ดี: Protocol ใหญ่เกินไป
class BigProtocol(Protocol):
    def method1(self): ...
    def method2(self): ...
    def method3(self): ...
    def method4(self): ...
    def method5(self): ...

# ดีกว่า: แยกเป็น Protocol เล็กๆ
class Closeable(Protocol):
    def close(self) -> None: ...

class Flushable(Protocol):
    def flush(self) -> None: ...

class Readable(Protocol):
    def read(self, n: int) -> bytes: ...

# Compose เมื่อต้องการ
class CloseableAndFlushable(Closeable, Flushable, Protocol):
    pass
```

### Best Practice 2: Use Protocol for Callbacks

```python
from typing import Protocol

class EventHandler(Protocol):
    def __call__(self, event_type: str, data: dict) -> None:
        ...

def on_click(event_type: str, data: dict) -> None:
    print(f"Click event: {data}")

def on_hover(event_type: str, data: dict) -> None:
    print(f"Hover event: {data}")

class EventSystem:
    def __init__(self):
        self._handlers: dict[str, list[EventHandler]] = {}
    
    def on(self, event_type: str, handler: EventHandler) -> None:
        if event_type not in self._handlers:
            self._handlers[event_type] = []
        self._handlers[event_type].append(handler)
    
    def emit(self, event_type: str, data: dict) -> None:
        for handler in self._handlers.get(event_type, []):
            handler(event_type, data)

events = EventSystem()
events.on("click", on_click)
events.on("hover", on_hover)
events.emit("click", {"x": 100, "y": 200})
events.emit("hover", {"element": "button"})
```

### Best Practice 3: Protocol Members ควร document expected behavior

```python
from typing import Protocol

class Comparable(Protocol):
    """Protocol สำหรับ objects ที่สามารถเปรียบเทียบลำดับได้
    
    จะถือว่า implement protocol นี้ถ้ามี __lt__ method
    ที่ return bool และทำงานตาม total ordering
    """
    
    def __lt__(self, other: 'Comparable') -> bool:
        """Return True ถ้า self น้อยกว่า other
        
        Must satisfy:
        - Irreflexivity: not (a < a)
        - Asymmetry: if a < b then not (b < a)
        - Transitivity: if a < b and b < c then a < c
        """
        ...

class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius
    
    def __lt__(self, other: 'Temperature') -> bool:
        return self.celsius < other.celsius
    
    def __repr__(self):
        return f"{self.celsius}°C"

temps = [Temperature(30), Temperature(15), Temperature(25), Temperature(10)]
print(sorted(temps))  # [10°C, 15°C, 25°C, 30°C]
```

---

## 13. Protocol กับ ABC ร่วมกัน

```python
from typing import Protocol, runtime_checkable
from abc import ABC, abstractmethod

@runtime_checkable
class Saveable(Protocol):
    """Protocol: structural checking"""
    def save(self) -> None: ...
    def load(self) -> None: ...

class PersistentMixin(ABC):
    """ABC: nominal checking + shared implementation"""
    
    @abstractmethod
    def get_filename(self) -> str:
        pass
    
    def save(self) -> None:
        filename = self.get_filename()
        print(f"Saving to {filename}")
    
    def load(self) -> None:
        filename = self.get_filename()
        print(f"Loading from {filename}")

class Document(PersistentMixin):
    def __init__(self, name: str, content: str):
        self.name = name
        self.content = content
    
    def get_filename(self) -> str:
        return f"{self.name}.txt"

doc = Document("report", "Hello World")
doc.save()   # Saving to report.txt

# Document implement Saveable Protocol เพราะมี save() และ load()
print(isinstance(doc, Saveable))  # True
```

---

## 14. Advanced Protocol Examples

### File-like Protocol

```python
from typing import Protocol, Optional, AnyStr

class FileLike(Protocol):
    """Protocol สำหรับ file-like objects"""
    
    def read(self, size: int = -1) -> AnyStr:
        ...
    
    def write(self, data: AnyStr) -> int:
        ...
    
    def close(self) -> None:
        ...
    
    def __enter__(self) -> 'FileLike':
        ...
    
    def __exit__(self, *args) -> Optional[bool]:
        ...

import io
import gzip

def process_file(f: FileLike) -> str:
    """ทำงานกับ file-like objects ทุกชนิด"""
    return f.read()

# ทำงานกับ regular file
with open("/dev/null", "w") as f:
    content = f.write("test")

# ทำงานกับ StringIO
string_buffer = io.StringIO("Hello from StringIO!")
print(process_file(string_buffer))

# ทำงานกับ BytesIO
bytes_buffer = io.BytesIO(b"Hello from BytesIO!")
print(process_file(bytes_buffer))
```

### Plugin System ด้วย Protocol

```python
from typing import Protocol, List, Dict, Any

class Plugin(Protocol):
    """Protocol สำหรับ plugin system"""
    
    @property
    def name(self) -> str:
        ...
    
    @property
    def version(self) -> str:
        ...
    
    def initialize(self, config: Dict[str, Any]) -> None:
        ...
    
    def execute(self, data: Any) -> Any:
        ...
    
    def cleanup(self) -> None:
        ...

class LoggingPlugin:
    """Plugin สำหรับ logging - ไม่รู้จัก Plugin Protocol"""
    
    @property
    def name(self) -> str:
        return "logging"
    
    @property
    def version(self) -> str:
        return "1.0.0"
    
    def initialize(self, config: Dict[str, Any]) -> None:
        import logging
        level = config.get("level", "INFO")
        logging.basicConfig(level=getattr(logging, level))
        self._logger = logging.getLogger("plugin.logging")
    
    def execute(self, data: Any) -> Any:
        self._logger.info(f"Processing: {data}")
        return data
    
    def cleanup(self) -> None:
        pass  # Nothing to clean up

class TransformPlugin:
    """Plugin สำหรับ data transformation"""
    
    @property
    def name(self) -> str:
        return "transform"
    
    @property
    def version(self) -> str:
        return "2.1.0"
    
    def initialize(self, config: Dict[str, Any]) -> None:
        self._upper = config.get("uppercase", False)
    
    def execute(self, data: Any) -> Any:
        if isinstance(data, str) and self._upper:
            return data.upper()
        return data
    
    def cleanup(self) -> None:
        pass

class PluginManager:
    def __init__(self):
        self._plugins: List[Plugin] = []
    
    def register(self, plugin: Plugin, config: Dict[str, Any] = None) -> None:
        plugin.initialize(config or {})
        self._plugins.append(plugin)
        print(f"Registered plugin: {plugin.name} v{plugin.version}")
    
    def process(self, data: Any) -> Any:
        result = data
        for plugin in self._plugins:
            result = plugin.execute(result)
        return result
    
    def shutdown(self) -> None:
        for plugin in self._plugins:
            plugin.cleanup()

manager = PluginManager()
manager.register(LoggingPlugin(), {"level": "INFO"})
manager.register(TransformPlugin(), {"uppercase": True})

result = manager.process("hello world")
print(f"Final result: {result}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Sorter Protocol

สร้าง Protocol `Sortable` ที่มี method `__lt__` แล้วเขียน generic function `bubble_sort()`

```python
from typing import Protocol, TypeVar, List

class Sortable(Protocol):
    def __lt__(self, other: 'Sortable') -> bool:
        ...

T = TypeVar('T', bound=Sortable)

def bubble_sort(items: List[T]) -> List[T]:
    items = items.copy()
    n = len(items)
    for i in range(n):
        for j in range(n - i - 1):
            if items[j + 1] < items[j]:
                items[j], items[j + 1] = items[j + 1], items[j]
    return items

# Test
print(bubble_sort([3, 1, 4, 1, 5, 9, 2, 6]))
print(bubble_sort(["banana", "apple", "cherry"]))
print(bubble_sort([3.14, 1.41, 2.71]))
```

### แบบฝึกหัดที่ 2: Cache Protocol

```python
from typing import Protocol, TypeVar, Optional, Any

K = TypeVar('K')
V = TypeVar('V')

class CacheProtocol(Protocol[K, V]):
    def get(self, key: K) -> Optional[V]: ...
    def set(self, key: K, value: V) -> None: ...
    def delete(self, key: K) -> None: ...
    def clear(self) -> None: ...
    def __len__(self) -> int: ...

class InMemoryCache:
    def __init__(self, max_size: int = 100):
        self._data: dict = {}
        self._max_size = max_size
    
    def get(self, key) -> Optional[Any]:
        return self._data.get(key)
    
    def set(self, key, value) -> None:
        if len(self._data) >= self._max_size:
            oldest = next(iter(self._data))
            del self._data[oldest]
        self._data[key] = value
    
    def delete(self, key) -> None:
        self._data.pop(key, None)
    
    def clear(self) -> None:
        self._data.clear()
    
    def __len__(self) -> int:
        return len(self._data)

def memoize(cache: CacheProtocol, key: str, compute):
    result = cache.get(key)
    if result is None:
        result = compute()
        cache.set(key, result)
    return result

cache = InMemoryCache()
result = memoize(cache, "expensive_compute", lambda: sum(range(1000000)))
print(f"Result: {result}, Cache size: {len(cache)}")
```

### แบบฝึกหัดที่ 3: Observer Pattern ด้วย Protocol

```python
from typing import Protocol, TypeVar, Generic, List

T = TypeVar('T')

class Observer(Protocol[T]):
    def update(self, value: T) -> None: ...

class Observable(Generic[T]):
    def __init__(self):
        self._observers: List[Observer[T]] = []
        self._value: T = None
    
    def subscribe(self, observer: Observer[T]) -> None:
        self._observers.append(observer)
    
    def unsubscribe(self, observer: Observer[T]) -> None:
        self._observers.remove(observer)
    
    def notify(self, value: T) -> None:
        self._value = value
        for obs in self._observers:
            obs.update(value)

class PrintObserver:
    def update(self, value) -> None:
        print(f"[Print] Received: {value}")

class LogObserver:
    def __init__(self):
        self.history = []
    
    def update(self, value) -> None:
        self.history.append(value)
        print(f"[Log] Logged value #{len(self.history)}: {value}")

temperature = Observable[float]()
printer = PrintObserver()
logger = LogObserver()

temperature.subscribe(printer)
temperature.subscribe(logger)

temperature.notify(23.5)
temperature.notify(25.1)
temperature.notify(22.8)

print(f"Logged temperatures: {logger.history}")
```

### แบบฝึกหัดที่ 4: Strategy Pattern ด้วย Protocol

```python
from typing import Protocol, List

class SortStrategy(Protocol):
    def sort(self, data: List[int]) -> List[int]: ...

class BubbleSortStrategy:
    def sort(self, data: List[int]) -> List[int]:
        data = data.copy()
        n = len(data)
        for i in range(n):
            for j in range(n - i - 1):
                if data[j] > data[j + 1]:
                    data[j], data[j + 1] = data[j + 1], data[j]
        return data

class QuickSortStrategy:
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def sort(self, data: List[int]) -> List[int]:
        return self._strategy.sort(data)
    
    def change_strategy(self, strategy: SortStrategy) -> None:
        self._strategy = strategy

data = [64, 34, 25, 12, 22, 11, 90]

sorter = Sorter(BubbleSortStrategy())
print("Bubble sort:", sorter.sort(data))

sorter.change_strategy(QuickSortStrategy())
print("Quick sort:", sorter.sort(data))
```

### แบบฝึกหัดที่ 5: TypeGuard Custom Validator

```python
from typing import TypeGuard, Union, Any
from dataclasses import dataclass

@dataclass
class ValidatedUser:
    name: str
    email: str
    age: int

def is_valid_user_data(data: Any) -> TypeGuard[dict]:
    """TypeGuard สำหรับ validate user data"""
    if not isinstance(data, dict):
        return False
    if not all(key in data for key in ['name', 'email', 'age']):
        return False
    if not isinstance(data['name'], str) or not data['name']:
        return False
    if not isinstance(data['email'], str) or '@' not in data['email']:
        return False
    if not isinstance(data['age'], int) or not (0 <= data['age'] <= 150):
        return False
    return True

def create_user(data: Any) -> ValidatedUser:
    if is_valid_user_data(data):
        # data is narrowed to dict ที่มี fields ครบ
        return ValidatedUser(
            name=data['name'],
            email=data['email'],
            age=data['age']
        )
    raise ValueError(f"Invalid user data: {data}")

test_data = [
    {"name": "Alice", "email": "alice@example.com", "age": 30},
    {"name": "", "email": "invalid", "age": -1},
    {"name": "Bob", "email": "bob@example.com"},
]

for d in test_data:
    try:
        user = create_user(d)
        print(f"Created: {user}")
    except ValueError as e:
        print(f"Error: {e}")
```

### แบบฝึกหัดที่ 6: ABC กับ Template Method Pattern

```python
from abc import ABC, abstractmethod
from typing import List

class DataProcessor(ABC):
    """ABC ด้วย Template Method Pattern"""
    
    def process(self, data: List) -> List:
        """Template method - กำหนด algorithm skeleton"""
        validated = self.validate(data)
        transformed = self.transform(validated)
        result = self.format(transformed)
        return result
    
    @abstractmethod
    def validate(self, data: List) -> List:
        """Subclass ต้อง implement"""
        pass
    
    @abstractmethod
    def transform(self, data: List) -> List:
        """Subclass ต้อง implement"""
        pass
    
    def format(self, data: List) -> List:
        """Optional override"""
        return data

class NumberProcessor(DataProcessor):
    def validate(self, data: List) -> List:
        return [x for x in data if isinstance(x, (int, float))]
    
    def transform(self, data: List) -> List:
        return [x * 2 for x in data]

class StringProcessor(DataProcessor):
    def validate(self, data: List) -> List:
        return [x for x in data if isinstance(x, str)]
    
    def transform(self, data: List) -> List:
        return [x.upper() for x in data]
    
    def format(self, data: List) -> List:
        return [f"[{x}]" for x in data]

mixed_data = [1, "hello", 2.5, "world", None, 3]

num_processor = NumberProcessor()
str_processor = StringProcessor()

print("Numbers:", num_processor.process(mixed_data))
print("Strings:", str_processor.process(mixed_data))
```

### แบบฝึกหัดที่ 7: Generic Protocol with Covariance

```python
from typing import Protocol, TypeVar, List, Iterator

T_co = TypeVar('T_co', covariant=True)

class Collection(Protocol[T_co]):
    def __iter__(self) -> Iterator[T_co]: ...
    def __len__(self) -> int: ...

class TypedList:
    def __init__(self, items: List):
        self._items = items
    
    def __iter__(self):
        return iter(self._items)
    
    def __len__(self) -> int:
        return len(self._items)

def print_all(items: Collection) -> None:
    print(f"Collection of {len(items)} items:")
    for item in items:
        print(f"  - {item}")

print_all(TypedList([1, 2, 3]))
print_all(TypedList(["a", "b", "c"]))
print_all([10, 20, 30])  # list ก็ implement Collection Protocol
```

### แบบฝึกหัดที่ 8: Full Protocol System

สร้าง mini ORM ที่ใช้ Protocol

```python
from typing import Protocol, List, Optional, TypeVar, Generic

T = TypeVar('T')

class Model(Protocol):
    id: Optional[int]
    
    def to_dict(self) -> dict: ...

class Repository(Protocol[T]):
    def save(self, model: T) -> T: ...
    def find_by_id(self, id: int) -> Optional[T]: ...
    def find_all(self) -> List[T]: ...
    def delete(self, id: int) -> bool: ...

class GenericRepository(Generic[T]):
    def __init__(self):
        self._store: dict[int, T] = {}
        self._next_id = 1
    
    def save(self, model: T) -> T:
        if not hasattr(model, 'id') or model.id is None:
            model.id = self._next_id
            self._next_id += 1
        self._store[model.id] = model
        return model
    
    def find_by_id(self, id: int) -> Optional[T]:
        return self._store.get(id)
    
    def find_all(self) -> List[T]:
        return list(self._store.values())
    
    def delete(self, id: int) -> bool:
        if id in self._store:
            del self._store[id]
            return True
        return False

from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    email: str
    id: Optional[int] = None
    
    def to_dict(self) -> dict:
        return {"id": self.id, "name": self.name, "email": self.email}

user_repo: Repository[User] = GenericRepository()

u1 = user_repo.save(User("Alice", "alice@example.com"))
u2 = user_repo.save(User("Bob", "bob@example.com"))

print("All users:", user_repo.find_all())
print("User 1:", user_repo.find_by_id(1))
user_repo.delete(1)
print("After delete:", user_repo.find_all())
```

---

## สรุป

| หัวข้อ | Key Points |
|--------|-----------|
| `Protocol` | Structural subtyping - ไม่ต้อง explicitly inherit |
| `ABC` | Nominal subtyping - ต้อง explicitly inherit |
| `@runtime_checkable` | ทำให้ Protocol ใช้กับ `isinstance()` ได้ |
| Protocol Inheritance | Compose protocols ขนาดเล็ก |
| `TypeVar` bound | จำกัด TypeVar ให้ต้องเป็น subtype |
| `ParamSpec` | Capture function parameter spec สำหรับ decorators |
| `TypeGuard` | Custom type narrowing functions |
| Covariance | Producer - T_co สำหรับ read-only |
| Contravariance | Consumer - T_contra สำหรับ write-only |
| Invariance | Both read and write - T ธรรมดา |

**เมื่อไหรควรใช้อะไร:**
- **Protocol**: third-party integration, duck typing แบบ type-safe, ลด coupling
- **ABC**: shared implementation, enforce hierarchy, runtime isinstance checks
