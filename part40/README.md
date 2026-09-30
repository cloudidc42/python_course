# Part 40: Type Hints & mypy

## บทนำ

Type hints (หรือ type annotations) เป็นฟีเจอร์ที่เพิ่มเข้ามาใน Python 3.5 ช่วยให้โค้ดอ่านง่ายขึ้น ลด bugs และทำให้ IDE ช่วยได้ดีขึ้น mypy เป็น static type checker ที่ตรวจสอบ type errors ก่อน runtime

---

## 1. Type Hints พื้นฐาน

### ทำไมต้องใช้ Type Hints?

| ประโยชน์ | รายละเอียด |
|----------|-----------|
| Documentation | code อธิบายตัวเองได้ |
| IDE Support | autocomplete และ error highlighting |
| Bug Prevention | พบ type errors ก่อน runtime |
| Refactoring | เปลี่ยน code ได้ปลอดภัยขึ้น |
| Team Communication | ชัดเจนว่า function รับ/return อะไร |

### ตัวอย่างที่ 1: การประกาศ Type Hints พื้นฐาน

```python
# ก่อนมี type hints
def greet(name):
    return "Hello, " + name

# หลังมี type hints - ชัดเจนขึ้น
def greet_typed(name: str) -> str:
    return "Hello, " + name

# Variable annotations
age: int = 25
name: str = "Alice"
height: float = 1.75
is_active: bool = True
data: bytes = b"hello"

# Type hints ไม่บังคับ runtime (แค่ hint)
# Python ยังคง dynamic typing
greet_typed(123)  # ไม่ error ตอน runtime แต่ mypy จะแจ้ง!

print(greet_typed("World"))
print(age, name, height, is_active)
```

### ตัวอย่างที่ 2: Function Annotations ละเอียด

```python
from typing import Optional, Union

# Parameters และ return type
def calculate_bmi(weight_kg: float, height_m: float) -> float:
    """คำนวณ BMI"""
    return weight_kg / (height_m ** 2)

# Optional parameter (อาจเป็น None)
def find_user(user_id: int, default: Optional[str] = None) -> Optional[dict]:
    users = {1: {"name": "Alice"}, 2: {"name": "Bob"}}
    return users.get(user_id)

# Multiple return types
def parse_number(value: str) -> Union[int, float, None]:
    try:
        return int(value)
    except ValueError:
        try:
            return float(value)
        except ValueError:
            return None

# *args และ **kwargs
def log_event(event: str, *args: str, **kwargs: int) -> None:
    print(f"Event: {event}, args: {args}, kwargs: {kwargs}")

# ทดสอบ
bmi = calculate_bmi(70, 1.75)
print(f"BMI: {bmi:.1f}")

user = find_user(1)
print(f"User: {user}")

num = parse_number("3.14")
print(f"Parsed: {num} (type: {type(num).__name__})")
```

---

## 2. typing Module

### ตัวอย่างที่ 3: Basic Type Constructs

```python
from typing import (
    Optional, Union, Any, Never,
    List, Dict, Tuple, Set, FrozenSet,
    Sequence, Mapping, Iterable, Iterator,
    Type, ClassVar
)

# Optional[X] เทียบเท่ากับ Union[X, None]
def get_name(user_id: int) -> Optional[str]:
    db = {1: "Alice", 2: "Bob"}
    return db.get(user_id)

# Union[X, Y] - หนึ่งในหลาย types
def process(data: Union[str, bytes, int]) -> str:
    if isinstance(data, int):
        return str(data)
    elif isinstance(data, bytes):
        return data.decode('utf-8')
    return data

# Any - ไม่ตรวจสอบ type (ใช้เมื่อจำเป็นจริงๆ)
def flexible_function(value: Any) -> Any:
    return value

# Type[X] - หมายถึง class ตัวเอง ไม่ใช่ instance
def create_instance(cls: Type[list]) -> list:
    return cls()

# ClassVar - class variable
class Config:
    MAX_RETRIES: ClassVar[int] = 3
    timeout: float  # instance variable

# ทดสอบ
print(get_name(1))
print(process(b"hello world"))
print(process(42))
config = Config()
config.timeout = 5.0
print(f"Max retries: {Config.MAX_RETRIES}")
```

---

## 3. Optional, Union, List, Dict, Tuple, Set

### ตัวอย่างที่ 4: Container Types

```python
from typing import List, Dict, Tuple, Set, FrozenSet, Optional

# List[T] - list ของ elements type T
def sort_numbers(numbers: List[int]) -> List[int]:
    return sorted(numbers)

# Dict[K, V] - dict ที่ key เป็น K, value เป็น V
def count_words(text: str) -> Dict[str, int]:
    words = text.lower().split()
    return {word: words.count(word) for word in set(words)}

# Tuple - fixed length และ types
def get_coordinates() -> Tuple[float, float]:
    return (13.7563, 100.5018)  # Bangkok

# Tuple ที่ยาวได้ไม่จำกัด
def process_ids(ids: Tuple[int, ...]) -> List[int]:
    return [i * 2 for i in ids]

# Set[T] - set ของ elements type T
def unique_items(items: List[str]) -> Set[str]:
    return set(items)

# Nested types
def get_users() -> List[Dict[str, Optional[str]]]:
    return [
        {"name": "Alice", "email": "alice@example.com"},
        {"name": "Bob", "email": None},
    ]

# Python 3.9+: ใช้ built-in types แทน typing
# def sort_numbers(numbers: list[int]) -> list[int]:
# def count_words(text: str) -> dict[str, int]:
# def unique_items(items: list[str]) -> set[str]:

# ทดสอบ
print(sort_numbers([3, 1, 4, 1, 5, 9, 2, 6]))
print(count_words("the quick brown fox jumps over the lazy dog"))
lat, lon = get_coordinates()
print(f"Bangkok: ({lat}, {lon})")
print(process_ids((1, 2, 3, 4, 5)))
```

### ตัวอย่างที่ 5: Python 3.10+ Union Syntax

```python
# Python 3.10+ ใช้ | แทน Union
def process_new(data: str | bytes | int) -> str:  # Python 3.10+
    if isinstance(data, int):
        return str(data)
    elif isinstance(data, bytes):
        return data.decode()
    return data

# Optional ด้วย X | None
def find_item(items: list[str], target: str) -> str | None:
    return target if target in items else None

# Complex union
def parse_config(value: str | int | float | bool | None) -> str:
    if value is None:
        return "null"
    return str(value)

print(process_new(b"bytes"))
print(process_new(42))
print(find_item(["a", "b", "c"], "b"))
print(parse_config(None))
```

---

## 4. TypeVar, Generic

### ตัวอย่างที่ 6: TypeVar - Type Variables

```python
from typing import TypeVar, List, Optional, Callable

# TypeVar สำหรับ generic functions
T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

# Generic function ที่ทำงานกับ type ใดก็ได้
def first(items: List[T]) -> Optional[T]:
    """ดึง element แรกของ list"""
    return items[0] if items else None

def last(items: List[T]) -> Optional[T]:
    """ดึง element สุดท้ายของ list"""
    return items[-1] if items else None

# TypeVar ที่จำกัด type (bounded)
Number = TypeVar('Number', int, float, complex)

def add(a: Number, b: Number) -> Number:
    """บวกตัวเลขสองตัว"""
    return a + b

# TypeVar กับ bound (ต้องเป็น subtype ของ X)
Comparable = TypeVar('Comparable', bound='SupportsLessThan')

def min_value(a: T, b: T) -> T:
    """หาค่าที่น้อยกว่า"""
    return a if a < b else b  # type: ignore

# ทดสอบ
numbers = [1, 2, 3, 4, 5]
strings = ["apple", "banana", "cherry"]

print(first(numbers))    # 1
print(first(strings))    # "apple"
print(last(numbers))     # 5

print(add(1, 2))         # 3
print(add(1.5, 2.5))     # 4.0
```

### ตัวอย่างที่ 7: Generic Classes

```python
from typing import TypeVar, Generic, Optional, Iterator
from dataclasses import dataclass

T = TypeVar('T')

class Stack(Generic[T]):
    """Generic stack implementation"""
    
    def __init__(self) -> None:
        self._items: list[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        if not self._items:
            raise IndexError("Stack is empty")
        return self._items.pop()
    
    def peek(self) -> T:
        if not self._items:
            raise IndexError("Stack is empty")
        return self._items[-1]
    
    def is_empty(self) -> bool:
        return len(self._items) == 0
    
    def __len__(self) -> int:
        return len(self._items)
    
    def __iter__(self) -> Iterator[T]:
        return iter(reversed(self._items))

# Generic class กับ 2 type parameters
K = TypeVar('K')
V = TypeVar('V')

class Pair(Generic[K, V]):
    """Pair ของสอง types"""
    
    def __init__(self, first: K, second: V) -> None:
        self.first = first
        self.second = second
    
    def swap(self) -> 'Pair[V, K]':
        return Pair(self.second, self.first)
    
    def __repr__(self) -> str:
        return f"Pair({self.first!r}, {self.second!r})"

# ทดสอบ
int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
int_stack.push(3)
print(f"Stack: {list(int_stack)}")
print(f"Pop: {int_stack.pop()}")

str_stack: Stack[str] = Stack()
str_stack.push("hello")
str_stack.push("world")
print(f"String stack peek: {str_stack.peek()}")

pair = Pair("hello", 42)
print(f"Pair: {pair}")
swapped = pair.swap()
print(f"Swapped: {swapped}")
```

---

## 5. Protocol

### ตัวอย่างที่ 8: Protocol - Structural Subtyping

```python
from typing import Protocol, runtime_checkable

# Protocol กำหนด interface โดยไม่ต้อง inherit
@runtime_checkable
class Drawable(Protocol):
    """Protocol สำหรับ objects ที่ draw ได้"""
    
    def draw(self) -> str: ...
    
    def get_color(self) -> str: ...

@runtime_checkable
class Serializable(Protocol):
    """Protocol สำหรับ objects ที่ serialize ได้"""
    
    def to_dict(self) -> dict: ...
    
    @classmethod
    def from_dict(cls, data: dict) -> 'Serializable': ...

# Classes ที่ implement protocol โดยไม่ต้อง inherit!
class Circle:
    def __init__(self, radius: float, color: str = "red"):
        self.radius = radius
        self.color = color
    
    def draw(self) -> str:
        return f"Drawing circle r={self.radius}"
    
    def get_color(self) -> str:
        return self.color
    
    def to_dict(self) -> dict:
        return {"type": "circle", "radius": self.radius, "color": self.color}
    
    @classmethod
    def from_dict(cls, data: dict) -> 'Circle':
        return cls(data["radius"], data.get("color", "red"))

class Rectangle:
    def __init__(self, w: float, h: float):
        self.w, self.h = w, h
    
    def draw(self) -> str:
        return f"Drawing rect {self.w}x{self.h}"
    
    def get_color(self) -> str:
        return "blue"

def render(shape: Drawable) -> None:
    """Render shape ใดก็ได้ที่ implement Drawable"""
    print(f"  Color: {shape.get_color()}")
    print(f"  {shape.draw()}")

# ทดสอบ - ทำงานได้โดยไม่ต้อง inherit
shapes: list[Drawable] = [Circle(5.0), Rectangle(3.0, 4.0)]
for shape in shapes:
    render(shape)

# Runtime check
circle = Circle(5.0)
print(f"\nCircle is Drawable: {isinstance(circle, Drawable)}")
print(f"Circle is Serializable: {isinstance(circle, Serializable)}")
```

### ตัวอย่างที่ 9: Protocol กับ Generic

```python
from typing import Protocol, TypeVar, runtime_checkable

T_co = TypeVar('T_co', covariant=True)  # covariant
T_contra = TypeVar('T_contra', contravariant=True)  # contravariant

class Container(Protocol[T_co]):
    """Protocol สำหรับ container objects"""
    
    def __contains__(self, item: object) -> bool: ...
    def __len__(self) -> int: ...
    def __iter__(self): ...

class SupportsWrite(Protocol[T_contra]):
    """Protocol สำหรับ writable objects"""
    
    def write(self, data: T_contra) -> int: ...

# ใช้ Protocol สำหรับ type checking
def has_items(container: Container) -> bool:
    return len(container) > 0

# list, set, dict, tuple ล้วน implement Container protocol
print(has_items([1, 2, 3]))       # True
print(has_items(set()))           # False
print(has_items({"a": 1}))        # True
print(has_items(()))              # False
```

---

## 6. Literal Types

### ตัวอย่างที่ 10: Literal - จำกัด values

```python
from typing import Literal, Union, overload

# Literal จำกัด values ที่รับได้
Direction = Literal["north", "south", "east", "west"]
Status = Literal["active", "inactive", "pending"]
Priority = Literal[1, 2, 3]
TrueType = Literal[True]

def move(direction: Direction, steps: int) -> str:
    """เคลื่อนที่ตามทิศทาง"""
    return f"Moving {direction} by {steps} steps"

def set_status(item_id: int, status: Status) -> None:
    print(f"Item {item_id}: {status}")

def set_priority(task_id: int, priority: Priority) -> None:
    print(f"Task {task_id}: priority {priority}")

# ทดสอบ
print(move("north", 5))
print(move("east", 3))
set_status(42, "active")
set_priority(1, 2)

# Literal Union
LogLevel = Literal["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]

def log(level: LogLevel, message: str) -> None:
    print(f"[{level}] {message}")

log("INFO", "Application started")
log("ERROR", "Something went wrong")
```

### ตัวอย่างที่ 11: Literal กับ overload

```python
from typing import Literal, overload

@overload
def process(mode: Literal["text"], data: str) -> str: ...
@overload
def process(mode: Literal["bytes"], data: bytes) -> bytes: ...
@overload
def process(mode: Literal["number"], data: int) -> int: ...

def process(mode, data):
    """ประมวลผลตาม mode"""
    if mode == "text":
        return data.upper()
    elif mode == "bytes":
        return data.decode().upper().encode()
    elif mode == "number":
        return data * 2
    raise ValueError(f"Unknown mode: {mode}")

# Type checker รู้ว่า return type คืออะไร
text_result: str = process("text", "hello")
bytes_result: bytes = process("bytes", b"world")
num_result: int = process("number", 42)

print(text_result, bytes_result, num_result)
```

---

## 7. TypedDict

### ตัวอย่างที่ 12: TypedDict สำหรับ Dictionary Types

```python
from typing import TypedDict, Required, NotRequired

# TypedDict กำหนด structure ของ dict
class UserDict(TypedDict):
    id: int
    name: str
    email: str

class UserDictPartial(TypedDict, total=False):
    """total=False ทำให้ทุก key เป็น optional"""
    id: int
    name: str
    email: str

# Python 3.11+ ใช้ Required/NotRequired
class ProductDict(TypedDict):
    id: int
    name: str
    price: float
    description: NotRequired[str]  # optional
    category: Required[str]        # required (default)

def create_user(data: UserDict) -> str:
    return f"User: {data['name']} ({data['email']})"

def display_product(product: ProductDict) -> str:
    desc = product.get('description', 'No description')
    return f"{product['name']}: ${product['price']} - {desc}"

# ทดสอบ
user: UserDict = {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
}
print(create_user(user))

product: ProductDict = {
    "id": 1,
    "name": "Python Book",
    "price": 29.99,
    "category": "Programming",
    "description": "Learn Python fast!"
}
print(display_product(product))
```

### ตัวอย่างที่ 13: Inheritance กับ TypedDict

```python
from typing import TypedDict

class BaseRecord(TypedDict):
    id: int
    created_at: str

class UserRecord(BaseRecord):
    name: str
    email: str
    role: str

class AdminRecord(UserRecord):
    permissions: list[str]
    department: str

def process_user(user: UserRecord) -> str:
    return f"{user['name']} ({user['role']})"

def process_admin(admin: AdminRecord) -> str:
    return f"{admin['name']}: {', '.join(admin['permissions'])}"

# ทดสอบ
admin: AdminRecord = {
    "id": 1,
    "created_at": "2024-01-01",
    "name": "Admin Alice",
    "email": "admin@example.com",
    "role": "admin",
    "permissions": ["read", "write", "delete"],
    "department": "IT"
}

print(process_user(admin))
print(process_admin(admin))
```

---

## 8. Callable Types

### ตัวอย่างที่ 14: Callable Type Hints

```python
from typing import Callable, TypeVar
from functools import wraps

# Callable[[ArgTypes], ReturnType]
T = TypeVar('T')
R = TypeVar('R')

# Function ที่รับ function เป็น argument
def apply_twice(func: Callable[[int], int], value: int) -> int:
    """Apply function สองครั้ง"""
    return func(func(value))

def map_list(
    items: list[T],
    transform: Callable[[T], R]
) -> list[R]:
    """Map function ไปยัง list"""
    return [transform(item) for item in items]

def filter_list(
    items: list[T],
    predicate: Callable[[T], bool]
) -> list[T]:
    """Filter list ด้วย predicate"""
    return [item for item in items if predicate(item)]

# Higher-order function
def compose(
    f: Callable[[R], T],
    g: Callable[[T], R]
) -> Callable[[T], T]:
    """Compose สอง functions"""
    def composed(x: T) -> T:
        return f(g(x))
    return composed

# Decorator type
F = TypeVar('F', bound=Callable[..., object])

def log_calls(func: F) -> F:
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper  # type: ignore

# ทดสอบ
print(apply_twice(lambda x: x * 2, 3))      # 12
print(map_list([1, 2, 3, 4], lambda x: x**2))  # [1, 4, 9, 16]
print(filter_list([1, 2, 3, 4, 5, 6], lambda x: x % 2 == 0))

double = lambda x: x * 2
add_one = lambda x: x + 1
double_then_add = compose(add_one, double)
print(double_then_add(5))  # 11
```

### ตัวอย่างที่ 15: Callable กับ Protocol

```python
from typing import Protocol

class Processor(Protocol):
    """Protocol สำหรับ callable objects"""
    
    def __call__(self, data: str) -> str:
        """Process string data"""
        ...

class UpperCaseProcessor:
    """Processor ที่แปลงเป็นตัวพิมพ์ใหญ่"""
    
    def __call__(self, data: str) -> str:
        return data.upper()

class PrefixProcessor:
    """Processor ที่เพิ่ม prefix"""
    
    def __init__(self, prefix: str):
        self.prefix = prefix
    
    def __call__(self, data: str) -> str:
        return f"{self.prefix}{data}"

def process_pipeline(
    data: str,
    processors: list[Processor]
) -> str:
    """ประมวลผลข้อมูลผ่าน pipeline"""
    result = data
    for processor in processors:
        result = processor(result)
    return result

# ทดสอบ
pipeline: list[Processor] = [
    UpperCaseProcessor(),
    PrefixProcessor("[PROCESSED] "),
    lambda s: s + "!"  # lambda ก็ implement Processor protocol!
]

result = process_pipeline("hello world", pipeline)
print(result)  # [PROCESSED] HELLO WORLD!
```

---

## 9. Type Aliases

### ตัวอย่างที่ 16: Type Aliases

```python
from typing import TypeAlias  # Python 3.10+

# Simple type alias
Name: TypeAlias = str
Age: TypeAlias = int
Score: TypeAlias = float

# Complex type alias
UserId: TypeAlias = int
UserData: TypeAlias = dict[str, str | int | None]
UserList: TypeAlias = list[UserData]
UserMap: TypeAlias = dict[UserId, UserData]
Callback: TypeAlias = Callable[[str, int], bool]

# Python 3.12+ type statement (new syntax)
# type Vector = list[float]
# type Matrix = list[Vector]

# ใช้ type alias ในฟังก์ชัน
def create_user_map(users: UserList) -> UserMap:
    """สร้าง map จาก user id ไปยัง user data"""
    return {
        int(user.get('id', 0)): user  # type: ignore
        for user in users
    }

# Vector operations
Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[Vector]

def dot_product(v1: Vector, v2: Vector) -> float:
    """คำนวณ dot product"""
    assert len(v1) == len(v2), "Vectors must have same length"
    return sum(a * b for a, b in zip(v1, v2))

def matrix_vector_multiply(m: Matrix, v: Vector) -> Vector:
    """คูณ matrix กับ vector"""
    return [dot_product(row, v) for row in m]

# ทดสอบ
v1: Vector = [1.0, 2.0, 3.0]
v2: Vector = [4.0, 5.0, 6.0]
print(f"Dot product: {dot_product(v1, v2)}")

m: Matrix = [[1.0, 0.0], [0.0, 1.0]]  # Identity matrix
v: Vector = [3.0, 4.0]
result = matrix_vector_multiply(m, v)
print(f"Matrix * Vector: {result}")
```

---

## 10. Final

### ตัวอย่างที่ 17: Final - ค่าคงที่

```python
from typing import Final, ClassVar

# Final variable - ไม่สามารถ reassign ได้
MAX_SIZE: Final = 100
API_URL: Final[str] = "https://api.example.com"
VERSION: Final = (1, 0, 0)

# Final ใน class
class Config:
    # Class constant
    MAX_CONNECTIONS: Final[int] = 10
    DEFAULT_TIMEOUT: Final[float] = 30.0
    
    # ClassVar + Final
    APP_NAME: ClassVar[Final[str]] = "MyApp"
    
    def __init__(self, debug: bool = False):
        # Final instance variable (ตั้งได้แค่ใน __init__)
        self.debug: Final[bool] = debug
        self.created_at: Final[float] = __import__('time').time()

class ReadOnlyPoint:
    """Point ที่ immutable"""
    
    def __init__(self, x: float, y: float):
        self._x: Final[float] = x
        self._y: Final[float] = y
    
    @property
    def x(self) -> float:
        return self._x
    
    @property
    def y(self) -> float:
        return self._y
    
    def distance_to(self, other: 'ReadOnlyPoint') -> float:
        import math
        return math.sqrt((self.x - other.x)**2 + (self.y - other.y)**2)

# ทดสอบ
config = Config(debug=True)
print(f"Config: debug={config.debug}, app={Config.APP_NAME}")

p1 = ReadOnlyPoint(0.0, 0.0)
p2 = ReadOnlyPoint(3.0, 4.0)
print(f"Distance: {p1.distance_to(p2)}")
```

---

## 11. ClassVar

### ตัวอย่างที่ 18: ClassVar - Class Variables

```python
from typing import ClassVar
from dataclasses import dataclass, field

class Counter:
    """Counter ที่ใช้ ClassVar"""
    
    # ClassVar - เป็น class variable ไม่ใช่ instance variable
    count: ClassVar[int] = 0
    instances: ClassVar[list['Counter']] = []
    
    def __init__(self, name: str):
        self.name = name  # instance variable
        Counter.count += 1
        Counter.instances.append(self)
    
    @classmethod
    def get_count(cls) -> int:
        return cls.count
    
    @classmethod
    def get_all(cls) -> list['Counter']:
        return cls.instances.copy()
    
    def __repr__(self) -> str:
        return f"Counter(name={self.name!r})"

@dataclass
class DatabaseConfig:
    """Config class ที่ใช้ ClassVar กับ dataclass"""
    
    # ClassVar ไม่ถูก include ใน __init__
    DEFAULT_PORT: ClassVar[int] = 5432
    SUPPORTED_DRIVRES: ClassVar[list[str]] = ['postgresql', 'mysql', 'sqlite']
    
    # Instance variables
    host: str = "localhost"
    port: int = field(default_factory=lambda: DatabaseConfig.DEFAULT_PORT)
    database: str = "mydb"
    driver: str = "postgresql"

# ทดสอบ
c1 = Counter("first")
c2 = Counter("second")
c3 = Counter("third")

print(f"Total counters: {Counter.get_count()}")
print(f"All counters: {Counter.get_all()}")

db = DatabaseConfig(host="db.example.com", database="production")
print(f"DB Config: {db}")
print(f"Default port: {DatabaseConfig.DEFAULT_PORT}")
print(f"Supported drivers: {DatabaseConfig.SUPPORTED_DRIVRES}")
```

---

## 12. Annotated

### ตัวอย่างที่ 19: Annotated - Metadata สำหรับ Types

```python
from typing import Annotated, get_type_hints
import dataclasses

# Annotated[T, metadata] เพิ่ม metadata ให้กับ type
# metadata ใช้สำหรับ validation, documentation, etc.

# Custom validators
class Gt:
    """Greater than validator"""
    def __init__(self, value): self.value = value
    def __repr__(self): return f"Gt({self.value})"

class Lt:
    """Less than validator"""
    def __init__(self, value): self.value = value

class MaxLen:
    """Maximum length validator"""
    def __init__(self, max_len): self.max_len = max_len

class MinLen:
    """Minimum length validator"""
    def __init__(self, min_len): self.min_len = min_len

# Type aliases with constraints
PositiveInt = Annotated[int, Gt(0)]
Age = Annotated[int, Gt(0), Lt(150)]
Name = Annotated[str, MinLen(1), MaxLen(100)]
Email = Annotated[str, MaxLen(254)]

# ใช้ใน function
def create_user(
    name: Name,
    age: Age,
    email: Email,
    score: Annotated[float, Gt(0.0), Lt(100.0)]
) -> dict:
    return {'name': name, 'age': age, 'email': email, 'score': score}

# Pydantic จะใช้ Annotated metadata สำหรับ validation
# FastAPI ใช้ Annotated สำหรับ query parameters

print(create_user("Alice", 30, "alice@example.com", 95.5))

# Runtime access ไปยัง metadata
hints = get_type_hints(create_user, include_extras=True)
print(f"\nType hints for create_user:")
for param, hint in hints.items():
    print(f"  {param}: {hint}")
```

---

## 13. mypy Installation และ Configuration

### ตัวอย่างที่ 20: การติดตั้งและใช้ mypy

```bash
# ติดตั้ง mypy
pip install mypy

# ตรวจสอบ file เดียว
mypy script.py

# ตรวจสอบทั้ง directory
mypy src/

# Options ที่ใช้บ่อย
mypy --strict script.py           # เข้มงวดที่สุด
mypy --ignore-missing-imports .   # ignore ถ้าไม่มี stubs
mypy --check-untyped-defs .       # ตรวจ functions ที่ไม่มี annotations
mypy --show-error-codes .         # แสดง error codes
mypy --pretty .                   # แสดง output สวยงาม
```

### ตัวอย่างที่ 21: mypy.ini Configuration

```ini
# mypy.ini หรือ setup.cfg หรือ pyproject.toml

[mypy]
python_version = 3.12
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
disallow_incomplete_defs = True
check_untyped_defs = True
disallow_untyped_decorators = True
no_implicit_optional = True
warn_redundant_casts = True
warn_unused_ignores = True
warn_no_return = True
warn_unreachable = True
strict_equality = True
show_error_codes = True

# Per-module settings
[mypy-requests.*]
ignore_missing_imports = True

[mypy-numpy.*]
ignore_missing_imports = True

[mypy-tests.*]
ignore_errors = True
```

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
strict = true

[[tool.mypy.overrides]]
module = "tests.*"
ignore_errors = true
```

### ตัวอย่างที่ 22: Common mypy Errors

```python
# ตัวอย่าง code ที่จะแสดง mypy errors

# Error: Incompatible types
def add_numbers(a: int, b: int) -> int:
    return a + b

result = add_numbers("1", "2")  # mypy error: Argument 1 has incompatible type "str"

# Error: Missing return type
def get_name():  # mypy warning: Function is missing a return type annotation
    return "Alice"

# Error: Possibly None
def find_user(user_id: int) -> dict | None:
    if user_id == 1:
        return {"name": "Alice"}
    return None

user = find_user(1)
print(user["name"])  # mypy error: Item "None" of "dict | None" has no attribute "__getitem__"

# Fix: ตรวจสอบ None ก่อน
if user is not None:
    print(user["name"])  # OK!

# หรือใช้ assert
assert user is not None, "User not found"
print(user["name"])  # OK!
```

---

## 14. mypy เบื้องต้น

### ตัวอย่างที่ 23: mypy Type Narrowing

```python
from typing import Union

def process_value(value: Union[int, str, None]) -> str:
    # mypy จะ narrow type หลัง isinstance checks
    if value is None:
        return "null"
    
    if isinstance(value, int):
        # ที่นี่ mypy รู้ว่า value เป็น int
        return str(value * 2)
    
    # ที่นี่ mypy รู้ว่า value เป็น str
    return value.upper()

# Type narrowing กับ assert
def get_user_name(user: dict | None) -> str:
    assert user is not None  # บอก mypy ว่าไม่ใช่ None
    return user["name"]  # OK!

# Type narrowing กับ TypeGuard (Python 3.10+)
from typing import TypeGuard

def is_string_list(items: list) -> TypeGuard[list[str]]:
    return all(isinstance(item, str) for item in items)

def process_strings(items: list) -> list[str]:
    if is_string_list(items):
        # mypy รู้ว่า items เป็น list[str]
        return [s.upper() for s in items]
    return [str(item) for item in items]

print(process_value(42))
print(process_value("hello"))
print(process_value(None))
print(process_strings(["a", "b", "c"]))
```

### ตัวอย่างที่ 24: mypy Comments และ type: ignore

```python
# บางครั้งต้อง override mypy

# type: ignore - ignore error ในบรรทัดนั้น
result: str = 42  # type: ignore[assignment]

# type: ignore[error-code] - ignore specific error
data = get_some_data()  # type: ignore[name-defined]

# cast - บอก mypy ว่า type คืออะไร
from typing import cast

value: object = "hello"
# mypy ไม่รู้ว่า value เป็น str
# length = len(value)  # error!

str_value = cast(str, value)  # บอก mypy ว่าเป็น str
length = len(str_value)  # OK!

# TYPE_CHECKING - import แค่สำหรับ type checking
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from collections.abc import Generator  # ไม่ถูก import ตอน runtime

# reveal_type - debug tool สำหรับ mypy
x = [1, 2, 3]
reveal_type(x)  # mypy: Revealed type is "builtins.list[builtins.int]"
```

---

## 15. Type Checking Best Practices

### ตัวอย่างที่ 25: Best Practices

```python
from typing import Optional, Union, Any
from dataclasses import dataclass

# ✓ ดี: ใช้ type hints ทุกที่ที่สำคัญ
def calculate_tax(
    amount: float,
    tax_rate: float,
    discount: float = 0.0
) -> float:
    return (amount - discount) * (1 + tax_rate)

# ✓ ดี: ใช้ Optional แทน Union[X, None]
def find_item(name: str) -> Optional[dict]:
    ...

# ✓ ดี: ใช้ Python 3.10+ union syntax
def process(data: str | bytes) -> str:
    if isinstance(data, bytes):
        return data.decode()
    return data

# ✓ ดี: ใช้ TypedDict สำหรับ dict ที่มี structure
@dataclass
class User:
    id: int
    name: str
    email: str
    age: Optional[int] = None

# ✓ ดี: ใช้ Protocol แทน ABC เมื่อเป็นไปได้
from typing import Protocol

class Closeable(Protocol):
    def close(self) -> None: ...

# ✗ หลีกเลี่ยง: Any ที่ไม่จำเป็น
def bad_process(data: Any) -> Any:  # ไม่ดี
    ...

# ✓ ดีกว่า: ระบุ type ที่ชัดเจน
def good_process(data: str | bytes | int) -> str:
    return str(data)

# ✓ ดี: ใช้ TypeVar สำหรับ generic functions
from typing import TypeVar
T = TypeVar('T')

def identity(value: T) -> T:
    return value
```

### ตัวอย่างที่ 26: Gradual Typing

```python
"""
Gradual typing: ค่อยๆ เพิ่ม type hints
"""

# ขั้นที่ 1: ไม่มี annotations (legacy code)
def process_data_v0(data):
    return [item * 2 for item in data]

# ขั้นที่ 2: เพิ่ม return type
def process_data_v1(data) -> list:
    return [item * 2 for item in data]

# ขั้นที่ 3: เพิ่ม parameter types
def process_data_v2(data: list) -> list:
    return [item * 2 for item in data]

# ขั้นที่ 4: ใช้ Generic types
from typing import TypeVar
T = TypeVar('T', int, float)

def process_data_v3(data: list[T]) -> list[T]:
    return [item * 2 for item in data]

# ขั้นที่ 5: Full type safety
def process_data_final(data: list[int | float]) -> list[int | float]:
    return [item * 2 for item in data]

# ทดสอบ
print(process_data_v3([1, 2, 3]))        # list[int]
print(process_data_v3([1.0, 2.0, 3.0])) # list[float]
```

---

## 16. Python 3.10+ Union Type

### ตัวอย่างที่ 27: New Union Syntax (X | Y)

```python
# Python 3.10+: ใช้ | แทน Union
# Python 3.9 และก่อนหน้า ต้องใช้ from __future__ import annotations

from __future__ import annotations  # ใช้ใน Python 3.7+ เพื่อ enable 3.10+ syntax

# ใหม่ใน Python 3.10+
def process(value: int | str | None) -> str:
    if value is None:
        return "none"
    return str(value)

# match statement (Python 3.10+) กับ type narrowing
def describe(value: int | str | list[int]) -> str:
    match value:
        case int():
            return f"Integer: {value}"
        case str():
            return f"String: {value}"
        case list():
            return f"List of {len(value)} integers"
        case _:
            return "Unknown"

print(process(42))
print(process("hello"))
print(process(None))
print(describe(42))
print(describe("hello"))
print(describe([1, 2, 3]))

# isinstance กับ union type (Python 3.10+)
def check_type(value: object) -> None:
    if isinstance(value, int | str):  # Python 3.10+
        print(f"int or str: {value}")
    elif isinstance(value, (list, tuple)):  # Python 3.9 compatible
        print(f"sequence: {value}")
```

---

## 17. Python 3.12 Type Parameter Syntax

### ตัวอย่างที่ 28: Type Parameter Syntax (PEP 695)

```python
# Python 3.12+ ใช้ syntax ใหม่สำหรับ generics

# Python 3.11 และก่อนหน้า (verbose)
from typing import TypeVar, Generic

T_old = TypeVar('T_old')

class Stack_old(Generic[T_old]):
    def push(self, item: T_old) -> None: ...
    def pop(self) -> T_old: ...

# Python 3.12+ (concise) - ยังไม่ได้ในทุก environment
# type Point[T] = tuple[T, T]  # type alias
# class Stack[T]:               # generic class
#     def push(self, item: T) -> None: ...
#     def pop(self) -> T: ...
# def first[T](items: list[T]) -> T | None:  # generic function
#     return items[0] if items else None

# ตัวอย่างที่รันได้ใน Python 3.11 ด้วย typing
from typing import TypeVar, Generic, overload

KT = TypeVar('KT')
VT = TypeVar('VT')

class BiDict(Generic[KT, VT]):
    """Bidirectional dictionary"""
    
    def __init__(self) -> None:
        self._forward: dict[KT, VT] = {}
        self._backward: dict[VT, KT] = {}
    
    def put(self, key: KT, value: VT) -> None:
        self._forward[key] = value
        self._backward[value] = key
    
    def get_by_key(self, key: KT) -> VT | None:
        return self._forward.get(key)
    
    def get_by_value(self, value: VT) -> KT | None:
        return self._backward.get(value)

bidict: BiDict[str, int] = BiDict()
bidict.put("one", 1)
bidict.put("two", 2)
bidict.put("three", 3)

print(bidict.get_by_key("two"))      # 2
print(bidict.get_by_value(3))        # "three"
```

---

## 18. ตัวอย่างโปรแกรมจริงที่ fully typed

### ตัวอย่างที่ 29: Typed Repository Pattern

```python
from __future__ import annotations
from typing import Generic, TypeVar, Optional, Protocol, runtime_checkable
from dataclasses import dataclass, field
from datetime import datetime
import uuid

T = TypeVar('T')
ID = TypeVar('ID')

@dataclass
class BaseEntity:
    """Base class สำหรับ entities"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

@dataclass
class User(BaseEntity):
    name: str = ""
    email: str = ""
    age: int = 0
    is_active: bool = True

@dataclass
class Product(BaseEntity):
    name: str = ""
    price: float = 0.0
    stock: int = 0
    category: str = ""

class Repository(Protocol[T]):
    """Generic repository protocol"""
    
    def save(self, entity: T) -> T: ...
    def find_by_id(self, entity_id: str) -> Optional[T]: ...
    def find_all(self) -> list[T]: ...
    def delete(self, entity_id: str) -> bool: ...
    def count(self) -> int: ...

class InMemoryRepository(Generic[T]):
    """In-memory repository implementation"""
    
    def __init__(self) -> None:
        self._storage: dict[str, T] = {}
    
    def save(self, entity: T) -> T:
        entity_id = getattr(entity, 'id', str(uuid.uuid4()))
        self._storage[entity_id] = entity
        return entity
    
    def find_by_id(self, entity_id: str) -> Optional[T]:
        return self._storage.get(entity_id)
    
    def find_all(self) -> list[T]:
        return list(self._storage.values())
    
    def delete(self, entity_id: str) -> bool:
        if entity_id in self._storage:
            del self._storage[entity_id]
            return True
        return False
    
    def count(self) -> int:
        return len(self._storage)

class UserRepository(InMemoryRepository[User]):
    """Specific repository สำหรับ User"""
    
    def find_by_email(self, email: str) -> Optional[User]:
        return next(
            (user for user in self._storage.values() if user.email == email),
            None
        )
    
    def find_active_users(self) -> list[User]:
        return [u for u in self._storage.values() if u.is_active]

class ProductRepository(InMemoryRepository[Product]):
    """Specific repository สำหรับ Product"""
    
    def find_by_category(self, category: str) -> list[Product]:
        return [p for p in self._storage.values() if p.category == category]
    
    def find_in_stock(self) -> list[Product]:
        return [p for p in self._storage.values() if p.stock > 0]
    
    def find_by_price_range(self, min_price: float, max_price: float) -> list[Product]:
        return [
            p for p in self._storage.values()
            if min_price <= p.price <= max_price
        ]

# ทดสอบ
user_repo = UserRepository()
product_repo = ProductRepository()

# สร้าง users
alice = User(name="Alice", email="alice@example.com", age=30)
bob = User(name="Bob", email="bob@example.com", age=25)
charlie = User(name="Charlie", email="charlie@example.com", age=35, is_active=False)

user_repo.save(alice)
user_repo.save(bob)
user_repo.save(charlie)

print(f"Total users: {user_repo.count()}")
print(f"Active users: {len(user_repo.find_active_users())}")
print(f"Find Alice: {user_repo.find_by_email('alice@example.com')}")

# สร้าง products
products_data = [
    ("Python Book", 29.99, 50, "Books"),
    ("Keyboard", 99.99, 10, "Electronics"),
    ("Mouse", 49.99, 0, "Electronics"),
    ("Java Book", 34.99, 30, "Books"),
]

for name, price, stock, category in products_data:
    product_repo.save(Product(name=name, price=price, stock=stock, category=category))

print(f"\nTotal products: {product_repo.count()}")
print(f"In stock: {len(product_repo.find_in_stock())}")
books = product_repo.find_by_category("Books")
print(f"Books: {[b.name for b in books]}")
affordable = product_repo.find_by_price_range(0, 50)
print(f"Under $50: {[p.name for p in affordable]}")
```

### ตัวอย่างที่ 30: Typed Event System

```python
from __future__ import annotations
from typing import TypeVar, Generic, Callable, Protocol, Any
from dataclasses import dataclass, field
from datetime import datetime
import functools

# TypeVar สำหรับ event data
E = TypeVar('E')

@dataclass
class Event(Generic[E]):
    """Generic event"""
    name: str
    data: E
    timestamp: datetime = field(default_factory=datetime.now)
    source: str = ""

# Callable types
EventHandler = Callable[[Event[E]], None]
AsyncEventHandler = Callable[[Event[E]], Any]

class TypedEventEmitter(Generic[E]):
    """Typed event emitter"""
    
    def __init__(self) -> None:
        self._handlers: dict[str, list[EventHandler]] = {}
    
    def on(self, event_name: str, handler: EventHandler) -> None:
        self._handlers.setdefault(event_name, []).append(handler)
    
    def off(self, event_name: str, handler: EventHandler) -> None:
        if event_name in self._handlers:
            self._handlers[event_name] = [
                h for h in self._handlers[event_name] if h != handler
            ]
    
    def emit(self, event: Event[E]) -> None:
        handlers = self._handlers.get(event.name, [])
        for handler in handlers:
            handler(event)

# Specific event types
@dataclass
class UserEventData:
    user_id: int
    username: str
    action: str

@dataclass
class PaymentEventData:
    payment_id: str
    amount: float
    currency: str
    status: str

# Type aliases
UserEvent = Event[UserEventData]
PaymentEvent = Event[PaymentEventData]

UserEmitter = TypedEventEmitter[UserEventData]
PaymentEmitter = TypedEventEmitter[PaymentEventData]

# Event handlers
def on_user_login(event: UserEvent) -> None:
    print(f"[Auth] User {event.data.username} logged in at {event.timestamp}")

def on_payment_completed(event: PaymentEvent) -> None:
    data = event.data
    print(f"[Payment] {data.payment_id}: ${data.amount} {data.currency} - {data.status}")

# ทดสอบ
user_emitter: UserEmitter = TypedEventEmitter()
payment_emitter: PaymentEmitter = TypedEventEmitter()

user_emitter.on("login", on_user_login)
payment_emitter.on("completed", on_payment_completed)

user_emitter.emit(UserEvent(
    name="login",
    data=UserEventData(user_id=42, username="alice", action="login")
))

payment_emitter.emit(PaymentEvent(
    name="completed",
    data=PaymentEventData(
        payment_id="PAY-001",
        amount=150.00,
        currency="USD",
        status="success"
    )
))
```

### ตัวอย่างที่ 31: Typed Configuration System

```python
from __future__ import annotations
from typing import TypedDict, Required, NotRequired, Literal, Final
from dataclasses import dataclass, field
import os

Environment = Literal["development", "staging", "production"]
LogLevel = Literal["DEBUG", "INFO", "WARNING", "ERROR"]

class DatabaseConfig(TypedDict):
    host: str
    port: int
    name: str
    username: str
    password: str
    pool_size: NotRequired[int]
    ssl: NotRequired[bool]

class RedisConfig(TypedDict):
    host: str
    port: int
    db: int
    password: NotRequired[str]
    max_connections: NotRequired[int]

class AppConfig(TypedDict):
    environment: Environment
    debug: bool
    secret_key: str
    allowed_hosts: list[str]
    database: DatabaseConfig
    redis: NotRequired[RedisConfig]
    log_level: NotRequired[LogLevel]

@dataclass
class ConfigLoader:
    """โหลด config จาก environment variables"""
    
    environment: Environment = field(
        default_factory=lambda: os.getenv("APP_ENV", "development")  # type: ignore
    )
    
    def load(self) -> AppConfig:
        """โหลด config ตาม environment"""
        base_config: AppConfig = {
            "environment": self.environment,
            "debug": self.environment == "development",
            "secret_key": os.getenv("SECRET_KEY", "dev-secret-key"),
            "allowed_hosts": ["*"] if self.environment == "development" else [],
            "database": self._load_db_config(),
            "log_level": "DEBUG" if self.environment == "development" else "INFO",
        }
        
        redis_url = os.getenv("REDIS_URL")
        if redis_url:
            base_config["redis"] = self._load_redis_config()
        
        return base_config
    
    def _load_db_config(self) -> DatabaseConfig:
        return {
            "host": os.getenv("DB_HOST", "localhost"),
            "port": int(os.getenv("DB_PORT", "5432")),
            "name": os.getenv("DB_NAME", "myapp"),
            "username": os.getenv("DB_USER", "postgres"),
            "password": os.getenv("DB_PASSWORD", ""),
            "pool_size": int(os.getenv("DB_POOL_SIZE", "10")),
            "ssl": os.getenv("DB_SSL", "false").lower() == "true",
        }
    
    def _load_redis_config(self) -> RedisConfig:
        return {
            "host": os.getenv("REDIS_HOST", "localhost"),
            "port": int(os.getenv("REDIS_PORT", "6379")),
            "db": int(os.getenv("REDIS_DB", "0")),
            "max_connections": int(os.getenv("REDIS_MAX_CONN", "50")),
        }

# ทดสอบ
loader = ConfigLoader(environment="development")
config = loader.load()

print(f"Environment: {config['environment']}")
print(f"Debug: {config['debug']}")
print(f"Log level: {config.get('log_level', 'INFO')}")
print(f"DB Host: {config['database']['host']}")
print(f"DB Pool: {config['database'].get('pool_size', 5)}")
```

### ตัวอย่างที่ 32: Runtime Type Validation

```python
from __future__ import annotations
from typing import get_type_hints, get_origin, get_args, Union
from dataclasses import dataclass
import inspect

def validate_types(instance: object) -> list[str]:
    """ตรวจสอบ types ของ dataclass instance"""
    errors = []
    hints = get_type_hints(type(instance))
    
    for field_name, expected_type in hints.items():
        if not hasattr(instance, field_name):
            continue
        
        value = getattr(instance, field_name)
        
        # จัดการ Optional (Union[X, None])
        origin = get_origin(expected_type)
        if origin is Union:
            args = get_args(expected_type)
            if not any(isinstance(value, arg) for arg in args if arg is not type(None)):
                if value is not None:
                    errors.append(
                        f"{field_name}: expected {expected_type}, got {type(value)}"
                    )
        elif not isinstance(value, expected_type):
            errors.append(
                f"{field_name}: expected {expected_type}, got {type(value)}"
            )
    
    return errors

@dataclass
class UserProfile:
    name: str
    age: int
    email: str
    score: float
    active: bool

# ทดสอบ
valid_user = UserProfile(
    name="Alice",
    age=30,
    email="alice@example.com",
    score=9.5,
    active=True
)

errors = validate_types(valid_user)
print(f"Valid user errors: {errors}")  # []

# สร้าง instance ที่ type ผิด (ทำใน Python runtime)
invalid_user = UserProfile.__new__(UserProfile)
invalid_user.name = 123  # ผิด! ควรเป็น str
invalid_user.age = "30"  # ผิด! ควรเป็น int
invalid_user.email = "alice@example.com"
invalid_user.score = 9.5
invalid_user.active = True

errors = validate_types(invalid_user)
print(f"Invalid user errors: {errors}")
```

### ตัวอย่างที่ 33: Fully Typed State Machine

```python
from __future__ import annotations
from typing import TypeVar, Generic, Callable, FrozenSet, Optional
from enum import Enum, auto
from dataclasses import dataclass

class OrderStatus(Enum):
    PENDING = auto()
    CONFIRMED = auto()
    PROCESSING = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()
    REFUNDED = auto()

S = TypeVar('S', bound=Enum)
E = TypeVar('E')

@dataclass(frozen=True)
class Transition(Generic[S]):
    from_state: S
    to_state: S
    action: str
    guard: Optional[Callable[[], bool]] = None

class StateMachine(Generic[S]):
    """Type-safe state machine"""
    
    def __init__(
        self,
        initial_state: S,
        transitions: list[Transition[S]]
    ) -> None:
        self._state = initial_state
        self._transitions: dict[tuple[S, str], Transition[S]] = {}
        self._history: list[tuple[S, str, S]] = []
        
        for t in transitions:
            self._transitions[(t.from_state, t.action)] = t
    
    @property
    def current_state(self) -> S:
        return self._state
    
    def can_trigger(self, action: str) -> bool:
        key = (self._state, action)
        if key not in self._transitions:
            return False
        t = self._transitions[key]
        return t.guard() if t.guard else True
    
    def trigger(self, action: str) -> bool:
        if not self.can_trigger(action):
            return False
        
        transition = self._transitions[(self._state, action)]
        old_state = self._state
        self._state = transition.to_state
        self._history.append((old_state, action, self._state))
        return True
    
    def available_actions(self) -> list[str]:
        return [
            action
            for (state, action) in self._transitions
            if state == self._state and self.can_trigger(action)
        ]
    
    def get_history(self) -> list[tuple[S, str, S]]:
        return self._history.copy()

# สร้าง order state machine
order_transitions: list[Transition[OrderStatus]] = [
    Transition(OrderStatus.PENDING, OrderStatus.CONFIRMED, "confirm"),
    Transition(OrderStatus.CONFIRMED, OrderStatus.PROCESSING, "start_processing"),
    Transition(OrderStatus.PROCESSING, OrderStatus.SHIPPED, "ship"),
    Transition(OrderStatus.SHIPPED, OrderStatus.DELIVERED, "deliver"),
    Transition(OrderStatus.PENDING, OrderStatus.CANCELLED, "cancel"),
    Transition(OrderStatus.CONFIRMED, OrderStatus.CANCELLED, "cancel"),
    Transition(OrderStatus.DELIVERED, OrderStatus.REFUNDED, "refund"),
]

machine: StateMachine[OrderStatus] = StateMachine(
    OrderStatus.PENDING,
    order_transitions
)

print(f"Initial state: {machine.current_state.name}")
print(f"Available: {machine.available_actions()}")

# ดำเนินการ
for action in ["confirm", "start_processing", "ship", "deliver"]:
    success = machine.trigger(action)
    print(f"Trigger '{action}': {'✓' if success else '✗'} → {machine.current_state.name}")

print(f"\nHistory:")
for from_s, action, to_s in machine.get_history():
    print(f"  {from_s.name} --{action}--> {to_s.name}")
```

### ตัวอย่างที่ 34: Typed Decorator Pattern

```python
from __future__ import annotations
from typing import TypeVar, Callable, ParamSpec, Concatenate
from functools import wraps
import time
import logging

P = ParamSpec('P')  # Python 3.10+
R = TypeVar('R')
T = TypeVar('T')

# ใช้ ParamSpec สำหรับ decorator ที่ preserve signature
def timer(func: Callable[P, R]) -> Callable[P, R]:
    """Decorator วัดเวลา"""
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__}: {elapsed:.3f}s")
        return result
    return wrapper

def retry(
    max_attempts: int = 3,
    exceptions: tuple[type[Exception], ...] = (Exception,)
) -> Callable[[Callable[P, R]], Callable[P, R]]:
    """Retry decorator"""
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt < max_attempts - 1:
                        time.sleep(0.1 * (2 ** attempt))
                    else:
                        raise
            raise RuntimeError("Unreachable")  # satisfies return type
        return wrapper
    return decorator

def validate_args(**validators: Callable[[object], bool]) -> Callable:
    """Decorator ที่ validate arguments"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            sig = __import__('inspect').signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            for param_name, validator in validators.items():
                if param_name in bound.arguments:
                    value = bound.arguments[param_name]
                    if not validator(value):
                        raise ValueError(
                            f"Invalid value for '{param_name}': {value!r}"
                        )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@timer
@retry(max_attempts=3, exceptions=(ValueError,))
@validate_args(
    price=lambda x: isinstance(x, (int, float)) and x > 0,
    quantity=lambda x: isinstance(x, int) and x > 0
)
def create_order(
    product_id: str,
    price: float,
    quantity: int,
    discount: float = 0.0
) -> dict:
    """สร้าง order"""
    total = price * quantity * (1 - discount)
    return {
        'product_id': product_id,
        'price': price,
        'quantity': quantity,
        'total': total
    }

# ทดสอบ
order = create_order("PROD-001", 29.99, 3, discount=0.1)
print(f"Order total: ${order['total']:.2f}")

# ทดสอบ validation
try:
    bad_order = create_order("PROD-002", -10.0, 2)
except ValueError as e:
    print(f"Validation error: {e}")
```

### ตัวอย่างที่ 35: Complete mypy Example

```python
"""
สคริปต์นี้ควรผ่าน mypy --strict
รัน: mypy --strict this_file.py
"""
from __future__ import annotations

from typing import TypeVar, Generic, Final, ClassVar
from typing import Callable, Iterator, Generator
from typing import Protocol, runtime_checkable
from dataclasses import dataclass, field
from enum import Enum
import math

# Constants
PI: Final[float] = math.pi
MAX_SHAPES: Final[int] = 1000

# Type variables
T = TypeVar('T')
N = TypeVar('N', int, float)

# Enums
class ShapeType(Enum):
    CIRCLE = "circle"
    RECTANGLE = "rectangle"
    TRIANGLE = "triangle"

# Protocols
@runtime_checkable
class Shape(Protocol):
    @property
    def area(self) -> float: ...
    
    @property
    def perimeter(self) -> float: ...
    
    @property
    def shape_type(self) -> ShapeType: ...

# Dataclasses
@dataclass(frozen=True)
class Circle:
    radius: float
    
    @property
    def area(self) -> float:
        return PI * self.radius ** 2
    
    @property
    def perimeter(self) -> float:
        return 2 * PI * self.radius
    
    @property
    def shape_type(self) -> ShapeType:
        return ShapeType.CIRCLE

@dataclass(frozen=True)
class Rectangle:
    width: float
    height: float
    
    @property
    def area(self) -> float:
        return self.width * self.height
    
    @property
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)
    
    @property
    def shape_type(self) -> ShapeType:
        return ShapeType.RECTANGLE

# Generic container
class ShapeCollection(Generic[T]):
    _count: ClassVar[int] = 0
    
    def __init__(self) -> None:
        self._shapes: list[T] = []
        ShapeCollection._count += 1
    
    def add(self, shape: T) -> None:
        self._shapes.append(shape)
    
    def __iter__(self) -> Iterator[T]:
        return iter(self._shapes)
    
    def __len__(self) -> int:
        return len(self._shapes)
    
    @classmethod
    def get_instance_count(cls) -> int:
        return cls._count

def total_area(shapes: list[Shape]) -> float:
    """คำนวณพื้นที่รวม"""
    return sum(shape.area for shape in shapes)

def filter_by_type(
    shapes: list[Shape],
    shape_type: ShapeType
) -> list[Shape]:
    """กรอง shapes ตาม type"""
    return [s for s in shapes if s.shape_type == shape_type]

def create_shapes(n: int) -> Generator[Shape, None, None]:
    """Generator สำหรับสร้าง shapes"""
    for i in range(n):
        if i % 2 == 0:
            yield Circle(float(i + 1))
        else:
            yield Rectangle(float(i + 1), float(i + 2))

# ทดสอบ
collection: ShapeCollection[Shape] = ShapeCollection()
for shape in create_shapes(10):
    collection.add(shape)

shapes = list(collection)
print(f"Total shapes: {len(shapes)}")
print(f"Total area: {total_area(shapes):.2f}")

circles = filter_by_type(shapes, ShapeType.CIRCLE)
rectangles = filter_by_type(shapes, ShapeType.RECTANGLE)
print(f"Circles: {len(circles)}, Rectangles: {len(rectangles)}")

# ตรวจสอบ Protocol
for shape in shapes[:3]:
    print(f"  {shape.shape_type.value}: area={shape.area:.2f}, "
          f"perimeter={shape.perimeter:.2f}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Type-annotated Calculator

**เฉลย:**

```python
from typing import TypeVar, overload
from decimal import Decimal

Number = TypeVar('Number', int, float, Decimal)

class TypedCalculator:
    """Calculator ที่มี type hints ครบถ้วน"""
    
    def add(self, a: Number, b: Number) -> Number:
        return a + b
    
    def subtract(self, a: Number, b: Number) -> Number:
        return a - b
    
    def multiply(self, a: Number, b: Number) -> Number:
        return a * b
    
    @overload
    def divide(self, a: int, b: int) -> float: ...
    @overload
    def divide(self, a: float, b: float) -> float: ...
    @overload
    def divide(self, a: Decimal, b: Decimal) -> Decimal: ...
    
    def divide(self, a, b):
        if b == 0:
            raise ZeroDivisionError("Cannot divide by zero")
        return a / b
    
    def power(self, base: Number, exponent: int) -> Number:
        return base ** exponent

calc = TypedCalculator()
print(calc.add(1, 2))
print(calc.add(1.5, 2.5))
print(calc.divide(10, 3))
print(calc.power(2, 10))
```

---

### แบบฝึกหัดที่ 2-8 (สรุปย่อ)

แบบฝึกหัดที่ 2: สร้าง generic `Result[T, E]` type สำหรับ error handling

แบบฝึกหัดที่ 3: สร้าง typed dependency injection container

แบบฝึกหัดที่ 4: สร้าง Protocol-based plugin system

แบบฝึกหัดที่ 5: สร้าง TypedDict ครบถ้วนสำหรับ JSON API responses

แบบฝึกหัดที่ 6: สร้าง typed decorator สำหรับ caching

แบบฝึกหัดที่ 7: สร้าง typed event bus ด้วย Generic

แบบฝึกหัดที่ 8: ตั้งค่า mypy ให้ strict และแก้ errors ทั้งหมด

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Type Hints พื้นฐาน** - function annotations, variable annotations
2. **typing module** - Union, Optional, Any, Type, ClassVar
3. **Container Types** - List, Dict, Tuple, Set และ built-in generics (3.9+)
4. **TypeVar & Generic** - การสร้าง generic functions และ classes
5. **Protocol** - structural subtyping สำหรับ duck typing ที่ type-safe
6. **Literal** - จำกัด values ที่รับได้
7. **TypedDict** - type hints สำหรับ dictionaries
8. **Callable** - type hints สำหรับ functions
9. **Type Aliases** - สร้าง aliases สำหรับ complex types
10. **Final** - ค่าคงที่ที่ไม่สามารถ reassign
11. **ClassVar** - class variables
12. **Annotated** - เพิ่ม metadata ให้ types
13. **mypy** - static type checker
14. **Python 3.10+ syntax** - X | Y แทน Union[X, Y]
15. **Python 3.12 syntax** - type parameter syntax (PEP 695)

### Best Practices

- เริ่มด้วย return types ก่อน แล้วค่อยเพิ่ม parameter types
- ใช้ `Optional[X]` หรือ `X | None` แทน `Union[X, None]`
- ใช้ Python 3.10+ syntax (`X | Y`) เมื่อเป็นไปได้
- หลีกเลี่ยง `Any` ให้มากที่สุด
- ใช้ `Protocol` แทน `ABC` เมื่อต้องการ structural typing
- ตั้งค่า mypy ให้ strict และรันใน CI/CD
- ใช้ `TYPE_CHECKING` import เพื่อหลีกเลี่ยง circular imports

---

*จบ Part 40 - Type Hints & mypy*

*หลักสูตร Python ขั้นสูง Parts 36-40 เสร็จสมบูรณ์*
