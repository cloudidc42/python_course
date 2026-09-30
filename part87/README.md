# Part 87: SOLID Principles & Clean Code

## บทนำ

SOLID คือกลุ่มของ 5 หลักการออกแบบซอฟต์แวร์แบบ Object-Oriented ที่ช่วยให้โค้ดมีความยืดหยุ่น บำรุงรักษาง่าย และขยายได้ นำเสนอโดย Robert C. Martin (Uncle Bob)

Clean Code คือแนวคิดการเขียนโค้ดที่อ่านง่าย เข้าใจง่าย และบำรุงรักษาง่าย จากหนังสือ "Clean Code" ของ Robert C. Martin

---

## 1. S — Single Responsibility Principle (SRP)

**"A class should have only one reason to change"**

คลาสหนึ่งควรมีหน้าที่รับผิดชอบเพียงอย่างเดียว และควรมีเหตุผลเดียวที่จะทำให้ต้องเปลี่ยนแปลง

```python
# ตัวอย่าง 1: SRP - ก่อน Refactoring (BAD)
class UserManager:
    """Class นี้ทำหลายอย่างเกินไป - ละเมิด SRP"""
    
    def __init__(self, db_connection):
        self.db = db_connection
    
    def create_user(self, username: str, email: str, password: str):
        # Validation
        if not username or len(username) < 3:
            raise ValueError("Username must be at least 3 characters")
        if '@' not in email:
            raise ValueError("Invalid email format")
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")
        
        # Password hashing
        import hashlib
        password_hash = hashlib.sha256(password.encode()).hexdigest()
        
        # Database save
        self.db.execute(
            "INSERT INTO users VALUES (?, ?, ?)",
            (username, email, password_hash)
        )
        
        # Send welcome email
        import smtplib
        with smtplib.SMTP('smtp.example.com') as server:
            server.sendmail(
                'noreply@example.com',
                email,
                f'Subject: Welcome!\n\nHello {username}!'
            )
        
        # Log
        with open('app.log', 'a') as f:
            f.write(f"User created: {username}\n")
```

```python
# ตัวอย่าง 2: SRP - หลัง Refactoring (GOOD)
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional

# 1. Validation - รับผิดชอบแค่ validation
class UserValidator:
    def validate(self, username: str, email: str, password: str):
        errors = []
        if not username or len(username) < 3:
            errors.append("Username must be at least 3 characters")
        if '@' not in email or '.' not in email.split('@')[-1]:
            errors.append("Invalid email format")
        if len(password) < 8:
            errors.append("Password must be at least 8 characters")
        if errors:
            raise ValueError("; ".join(errors))

# 2. Password handling - รับผิดชอบแค่ password
class PasswordHasher:
    def hash(self, password: str) -> str:
        import hashlib
        return hashlib.sha256(password.encode()).hexdigest()
    
    def verify(self, password: str, hashed: str) -> bool:
        return self.hash(password) == hashed

# 3. Repository - รับผิดชอบแค่ data access
@dataclass
class User:
    username: str
    email: str
    password_hash: str

class UserRepository:
    def __init__(self, db):
        self.db = db
    
    def save(self, user: User) -> User:
        self.db.execute(
            "INSERT INTO users VALUES (?, ?, ?)",
            (user.username, user.email, user.password_hash)
        )
        return user
    
    def find_by_username(self, username: str) -> Optional[User]:
        result = self.db.execute(
            "SELECT * FROM users WHERE username = ?", (username,)
        )
        return User(*result[0]) if result else None

# 4. Email service - รับผิดชอบแค่ email
class EmailService:
    def send_welcome(self, email: str, username: str):
        print(f"Sending welcome email to {email} for user {username}")

# 5. Logger - รับผิดชอบแค่ logging
class Logger:
    def info(self, message: str):
        print(f"[INFO] {message}")

# 6. UserService - orchestrates แค่ business flow
class UserService:
    def __init__(
        self,
        validator: UserValidator,
        hasher: PasswordHasher,
        repository: UserRepository,
        email_service: EmailService,
        logger: Logger
    ):
        self.validator = validator
        self.hasher = hasher
        self.repository = repository
        self.email_service = email_service
        self.logger = logger
    
    def create_user(self, username: str, email: str, password: str) -> User:
        self.validator.validate(username, email, password)
        user = User(
            username=username,
            email=email,
            password_hash=self.hasher.hash(password)
        )
        saved_user = self.repository.save(user)
        self.email_service.send_welcome(email, username)
        self.logger.info(f"User created: {username}")
        return saved_user
```

```python
# ตัวอย่าง 3: SRP ในระดับ Function
# BAD: Function ทำหลายอย่าง
def process_sales_report_bad(sales_data: list) -> str:
    # Filter, calculate, format - ทำ 3 อย่างในที่เดียว
    filtered = [s for s in sales_data if s['amount'] > 0]
    total = sum(s['amount'] for s in filtered)
    avg = total / len(filtered) if filtered else 0
    report = f"Total: ${total:.2f}\nAverage: ${avg:.2f}\nCount: {len(filtered)}"
    with open('report.txt', 'w') as f:
        f.write(report)
    return report

# GOOD: แยกหน้าที่ออกจากกัน
def filter_positive_sales(sales_data: list) -> list:
    return [s for s in sales_data if s['amount'] > 0]

def calculate_sales_stats(sales_data: list) -> dict:
    if not sales_data:
        return {'total': 0, 'average': 0, 'count': 0}
    total = sum(s['amount'] for s in sales_data)
    return {
        'total': total,
        'average': total / len(sales_data),
        'count': len(sales_data)
    }

def format_sales_report(stats: dict) -> str:
    return (
        f"Total: ${stats['total']:.2f}\n"
        f"Average: ${stats['average']:.2f}\n"
        f"Count: {stats['count']}"
    )

def save_report(content: str, filepath: str):
    print(f"Saving report to {filepath}")
    # with open(filepath, 'w') as f:
    #     f.write(content)

# Compose ใน service
def process_sales_report_good(sales_data: list, output_path: str) -> str:
    filtered = filter_positive_sales(sales_data)
    stats = calculate_sales_stats(filtered)
    report = format_sales_report(stats)
    save_report(report, output_path)
    return report
```

---

## 2. O — Open/Closed Principle (OCP)

**"Software entities should be open for extension, but closed for modification"**

โค้ดควรสามารถเพิ่มฟีเจอร์ใหม่ได้โดยไม่ต้องแก้ไขโค้ดเดิม

```python
# ตัวอย่าง 4: OCP - ละเมิดหลักการ (BAD)
class DiscountCalculator:
    """ทุกครั้งที่เพิ่ม discount type ต้อง modify class นี้"""
    
    def calculate(self, order_total: float, customer_type: str) -> float:
        if customer_type == 'regular':
            return order_total * 0.95  # 5% off
        elif customer_type == 'gold':
            return order_total * 0.90  # 10% off
        elif customer_type == 'platinum':
            return order_total * 0.80  # 20% off
        # ต้องมาแก้ตรงนี้ทุกครั้งที่เพิ่ม type ใหม่
        return order_total
```

```python
# ตัวอย่าง 5: OCP - ถูกต้อง (GOOD)
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def apply(self, total: float) -> float:
        pass
    
    @abstractmethod
    def description(self) -> str:
        pass

class RegularDiscount(DiscountStrategy):
    def apply(self, total: float) -> float:
        return total * 0.95
    
    def description(self) -> str:
        return "Regular: 5% off"

class GoldDiscount(DiscountStrategy):
    def apply(self, total: float) -> float:
        return total * 0.90
    
    def description(self) -> str:
        return "Gold: 10% off"

class PlatinumDiscount(DiscountStrategy):
    def apply(self, total: float) -> float:
        return total * 0.80
    
    def description(self) -> str:
        return "Platinum: 20% off"

# เพิ่ม type ใหม่โดยไม่แก้ code เดิม
class VIPDiscount(DiscountStrategy):
    def apply(self, total: float) -> float:
        return total * 0.70
    
    def description(self) -> str:
        return "VIP: 30% off"

class SeasonalDiscount(DiscountStrategy):
    def __init__(self, percent: float):
        self._percent = percent
    
    def apply(self, total: float) -> float:
        return total * (1 - self._percent / 100)
    
    def description(self) -> str:
        return f"Seasonal: {self._percent}% off"

class DiscountCalculator:
    """ไม่ต้อง modify ถ้าเพิ่ม discount type ใหม่"""
    def calculate(self, total: float, discount: DiscountStrategy) -> float:
        result = discount.apply(total)
        print(f"{discount.description()}: ${total:.2f} -> ${result:.2f}")
        return result

# การใช้งาน
calc = DiscountCalculator()
total = 1000.0

for discount in [
    RegularDiscount(),
    GoldDiscount(),
    PlatinumDiscount(),
    VIPDiscount(),
    SeasonalDiscount(15)
]:
    calc.calculate(total, discount)
```

```python
# ตัวอย่าง 6: OCP กับ Report Generation
from abc import ABC, abstractmethod
from typing import List, Dict

class ReportGenerator(ABC):
    @abstractmethod
    def generate(self, data: List[Dict]) -> str:
        pass

class HTMLReport(ReportGenerator):
    def generate(self, data: List[Dict]) -> str:
        if not data:
            return "<p>No data</p>"
        headers = list(data[0].keys())
        rows = "\n".join(
            "<tr>" + "".join(f"<td>{row.get(h,'')}</td>" for h in headers) + "</tr>"
            for row in data
        )
        header_html = "".join(f"<th>{h}</th>" for h in headers)
        return f"<table><thead><tr>{header_html}</tr></thead><tbody>{rows}</tbody></table>"

class MarkdownReport(ReportGenerator):
    def generate(self, data: List[Dict]) -> str:
        if not data:
            return "No data"
        headers = list(data[0].keys())
        separator = " | ".join(["---"] * len(headers))
        header_row = " | ".join(headers)
        data_rows = "\n".join(
            " | ".join(str(row.get(h, '')) for h in headers)
            for row in data
        )
        return f"{header_row}\n{separator}\n{data_rows}"

# เพิ่มรูปแบบใหม่โดยไม่แก้ code เดิม
class JSONReport(ReportGenerator):
    def generate(self, data: List[Dict]) -> str:
        import json
        return json.dumps(data, indent=2, ensure_ascii=False)

class CSVReport(ReportGenerator):
    def generate(self, data: List[Dict]) -> str:
        if not data:
            return ""
        headers = list(data[0].keys())
        rows = [",".join(headers)]
        rows.extend(",".join(str(row.get(h, '')) for h in headers) for row in data)
        return "\n".join(rows)

class ReportService:
    def __init__(self, generator: ReportGenerator):
        self._generator = generator
    
    def set_generator(self, generator: ReportGenerator):
        self._generator = generator
    
    def create_report(self, data: List[Dict]) -> str:
        return self._generator.generate(data)

# ทดสอบ
data = [
    {'name': 'Alice', 'dept': 'Eng', 'salary': 90000},
    {'name': 'Bob', 'dept': 'HR', 'salary': 70000},
]

service = ReportService(MarkdownReport())
print("=== Markdown ===")
print(service.create_report(data))

service.set_generator(JSONReport())
print("\n=== JSON ===")
print(service.create_report(data))
```

---

## 3. L — Liskov Substitution Principle (LSP)

**"Subtypes must be substitutable for their base types"**

ถ้า S เป็น subtype ของ T แล้ว object ของ T สามารถแทนที่ด้วย object ของ S ได้โดยไม่เปลี่ยนความถูกต้องของโปรแกรม

```python
# ตัวอย่าง 7: LSP - ละเมิดหลักการ (BAD) - Classic Rectangle/Square problem
class Rectangle:
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height
    
    @property
    def width(self) -> float:
        return self._width
    
    @width.setter
    def width(self, value: float):
        self._width = value
    
    @property
    def height(self) -> float:
        return self._height
    
    @height.setter
    def height(self, value: float):
        self._height = value
    
    def area(self) -> float:
        return self._width * self._height

class SquareBad(Rectangle):
    """ละเมิด LSP - Square ไม่สามารถแทน Rectangle ได้ทุกกรณี"""
    @Rectangle.width.setter
    def width(self, value: float):
        self._width = value
        self._height = value  # ต้องเปลี่ยน height ด้วย
    
    @Rectangle.height.setter
    def height(self, value: float):
        self._height = value
        self._width = value  # ต้องเปลี่ยน width ด้วย

def test_rectangle(rect: Rectangle):
    """Function ที่คาดว่าจะทำงานกับ Rectangle หรือ subtype ของมัน"""
    rect.width = 4
    rect.height = 5
    expected = 4 * 5  # 20
    actual = rect.area()
    assert actual == expected, f"Expected {expected} but got {actual}"
    print(f"Test passed: area = {actual}")

# ทดสอบ
r = Rectangle(3, 3)
test_rectangle(r)  # ผ่าน

try:
    s = SquareBad(3, 3)
    test_rectangle(s)  # ล้มเหลว! area = 25 ไม่ใช่ 20
except AssertionError as e:
    print(f"LSP violated: {e}")
```

```python
# ตัวอย่าง 8: LSP - ถูกต้อง (GOOD)
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height
    
    @property
    def width(self) -> float:
        return self._width
    
    @property
    def height(self) -> float:
        return self._height
    
    def area(self) -> float:
        return self._width * self._height
    
    def perimeter(self) -> float:
        return 2 * (self._width + self._height)

class Square(Shape):
    def __init__(self, side: float):
        self._side = side
    
    @property
    def side(self) -> float:
        return self._side
    
    def area(self) -> float:
        return self._side ** 2
    
    def perimeter(self) -> float:
        return 4 * self._side

# ทั้ง Rectangle และ Square สามารถแทน Shape ได้
def print_shape_info(shape: Shape):
    print(f"{shape.__class__.__name__}: area={shape.area()}, perimeter={shape.perimeter()}")

shapes = [Rectangle(4, 5), Square(4), Rectangle(3, 3)]
for s in shapes:
    print_shape_info(s)
```

```python
# ตัวอย่าง 9: LSP กับ Collection Classes
from abc import ABC, abstractmethod
from typing import Any, Iterator, List

class ReadOnlyCollection(ABC):
    @abstractmethod
    def get(self, index: int) -> Any:
        pass
    
    @abstractmethod
    def size(self) -> int:
        pass
    
    @abstractmethod
    def __iter__(self) -> Iterator:
        pass

class MutableCollection(ReadOnlyCollection):
    @abstractmethod
    def add(self, item: Any):
        pass
    
    @abstractmethod
    def remove(self, index: int):
        pass

class ImmutableList(ReadOnlyCollection):
    def __init__(self, data: List):
        self._data = list(data)
    
    def get(self, index: int) -> Any:
        return self._data[index]
    
    def size(self) -> int:
        return len(self._data)
    
    def __iter__(self) -> Iterator:
        return iter(self._data)

class DynamicList(MutableCollection):
    def __init__(self):
        self._data = []
    
    def get(self, index: int) -> Any:
        return self._data[index]
    
    def size(self) -> int:
        return len(self._data)
    
    def __iter__(self) -> Iterator:
        return iter(self._data)
    
    def add(self, item: Any):
        self._data.append(item)
    
    def remove(self, index: int):
        self._data.pop(index)

# Function ที่ทำงานกับ ReadOnlyCollection
def print_all(collection: ReadOnlyCollection):
    for item in collection:
        print(item)
    print(f"Size: {collection.size()}")

# ทั้งสองสามารถ pass ให้ print_all ได้
immutable = ImmutableList([1, 2, 3, 4, 5])
dynamic = DynamicList()
dynamic.add(10)
dynamic.add(20)
dynamic.add(30)

print("Immutable:")
print_all(immutable)

print("\nDynamic:")
print_all(dynamic)
```

```python
# ตัวอย่าง 10: LSP กับ Exception Handling
class DatabaseError(Exception):
    pass

class ConnectionError(DatabaseError):
    pass

class QueryError(DatabaseError):
    pass

class BaseDatabase(ABC):
    @abstractmethod
    def connect(self, url: str) -> bool:
        """ต้อง raise ConnectionError ถ้า connection ล้มเหลว"""
        pass
    
    @abstractmethod
    def query(self, sql: str) -> list:
        """ต้อง raise QueryError ถ้า query ไม่ valid"""
        pass

class PostgreSQLDB(BaseDatabase):
    def connect(self, url: str) -> bool:
        print(f"Connecting to PostgreSQL: {url}")
        # raise ConnectionError("Cannot connect") ถ้าล้มเหลว
        return True
    
    def query(self, sql: str) -> list:
        if not sql.strip():
            raise QueryError("Empty SQL query")
        print(f"Executing: {sql}")
        return [{"id": 1}]

class MySQLDB(BaseDatabase):
    def connect(self, url: str) -> bool:
        print(f"Connecting to MySQL: {url}")
        return True
    
    def query(self, sql: str) -> list:
        if not sql.strip():
            raise QueryError("Empty SQL query")
        print(f"MySQL executing: {sql}")
        return [{"id": 1}]

# Code ที่ทำงานกับ BaseDatabase
def run_query(db: BaseDatabase, url: str, sql: str) -> list:
    try:
        db.connect(url)
        return db.query(sql)
    except ConnectionError as e:
        print(f"Connection failed: {e}")
        return []
    except QueryError as e:
        print(f"Query failed: {e}")
        return []

# ทั้งสอง DB สามารถแทนกันได้
for db in [PostgreSQLDB(), MySQLDB()]:
    results = run_query(db, "localhost:5432/mydb", "SELECT * FROM users")
    print(f"Results: {results}\n")
```

---

## 4. I — Interface Segregation Principle (ISP)

**"Clients should not be forced to depend on interfaces they do not use"**

Interface ขนาดใหญ่ควรแยกเป็น Interface ย่อยๆ

```python
# ตัวอย่าง 11: ISP - ละเมิดหลักการ (BAD)
from abc import ABC, abstractmethod

class WorkerInterface(ABC):
    """Fat Interface - บังคับให้ implement ทุก method"""
    
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass
    
    @abstractmethod
    def sleep(self):
        pass
    
    @abstractmethod
    def recharge_battery(self):
        pass

class Human(WorkerInterface):
    def work(self): print("Human working")
    def eat(self): print("Human eating")
    def sleep(self): print("Human sleeping")
    def recharge_battery(self):
        raise NotImplementedError("Humans don't have batteries!")

class Robot(WorkerInterface):
    def work(self): print("Robot working")
    def eat(self):
        raise NotImplementedError("Robots don't eat!")
    def sleep(self):
        raise NotImplementedError("Robots don't sleep!")
    def recharge_battery(self): print("Robot recharging")
```

```python
# ตัวอย่าง 12: ISP - ถูกต้อง (GOOD)
from abc import ABC, abstractmethod

class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Feedable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Restable(ABC):
    @abstractmethod
    def sleep(self):
        pass

class Rechargeable(ABC):
    @abstractmethod
    def recharge(self):
        pass

class Human(Workable, Feedable, Restable):
    def work(self): print("Human working")
    def eat(self): print("Human eating")
    def sleep(self): print("Human sleeping")

class Robot(Workable, Rechargeable):
    def work(self): print("Robot working")
    def recharge(self): print("Robot recharging battery")

# Functions ที่ depend เฉพาะ interface ที่ต้องการ
def make_work(worker: Workable):
    worker.work()

def feed_entity(entity: Feedable):
    entity.eat()

# ทำงานกับ type ที่เหมาะสม
workers = [Human(), Robot()]
for w in workers:
    make_work(w)

# เฉพาะ Feedable entities
feedable = [Human()]
for f in feedable:
    feed_entity(f)
```

```python
# ตัวอย่าง 13: ISP กับ Repository Pattern
from abc import ABC, abstractmethod
from typing import Optional, List

# แยก interfaces ตาม use case
class Readable(ABC):
    @abstractmethod
    def find_by_id(self, id: int):
        pass
    
    @abstractmethod
    def find_all(self) -> list:
        pass

class Writable(ABC):
    @abstractmethod
    def save(self, entity) -> None:
        pass
    
    @abstractmethod
    def delete(self, id: int) -> None:
        pass

class Searchable(ABC):
    @abstractmethod
    def search(self, query: str) -> list:
        pass

class Pageable(ABC):
    @abstractmethod
    def find_page(self, page: int, size: int) -> list:
        pass

# Full repository implements all
class FullRepository(Readable, Writable, Searchable, Pageable):
    def __init__(self):
        self._data = {}
        self._counter = 0
    
    def find_by_id(self, id: int):
        return self._data.get(id)
    
    def find_all(self) -> list:
        return list(self._data.values())
    
    def save(self, entity) -> None:
        self._counter += 1
        entity['id'] = self._counter
        self._data[self._counter] = entity
    
    def delete(self, id: int) -> None:
        self._data.pop(id, None)
    
    def search(self, query: str) -> list:
        return [
            item for item in self._data.values()
            if query.lower() in str(item).lower()
        ]
    
    def find_page(self, page: int, size: int) -> list:
        all_items = list(self._data.values())
        start = (page - 1) * size
        return all_items[start:start + size]

# Read-only repository
class ReadOnlyRepository(Readable, Searchable):
    def __init__(self, data: list):
        self._data = {i+1: item for i, item in enumerate(data)}
    
    def find_by_id(self, id: int):
        return self._data.get(id)
    
    def find_all(self) -> list:
        return list(self._data.values())
    
    def search(self, query: str) -> list:
        return [
            item for item in self._data.values()
            if query.lower() in str(item).lower()
        ]

# Service ที่ต้องการแค่ read
def display_users(repo: Readable):
    users = repo.find_all()
    for user in users:
        print(f"  - {user}")

# ทดสอบ
full_repo = FullRepository()
full_repo.save({'name': 'Alice', 'role': 'admin'})
full_repo.save({'name': 'Bob', 'role': 'user'})
full_repo.save({'name': 'Charlie', 'role': 'user'})

print("All users:")
display_users(full_repo)

print("\nSearch 'admin':")
results = full_repo.search('admin')
print(results)
```

---

## 5. D — Dependency Inversion Principle (DIP)

**"Depend on abstractions, not on concretions"**

1. High-level modules ไม่ควร depend on low-level modules ทั้งคู่ควร depend on abstractions
2. Abstractions ไม่ควร depend on details ส่วน details ควร depend on abstractions

```python
# ตัวอย่าง 14: DIP - ละเมิดหลักการ (BAD)
class MySQLDatabase:
    def query(self, sql: str) -> list:
        print(f"MySQL: {sql}")
        return [{'id': 1}]

class UserServiceBad:
    """High-level module depend on concrete MySQL"""
    def __init__(self):
        self.db = MySQLDatabase()  # Hard dependency!
    
    def get_users(self) -> list:
        return self.db.query("SELECT * FROM users")
```

```python
# ตัวอย่าง 15: DIP - ถูกต้อง (GOOD)
from abc import ABC, abstractmethod
from typing import List, Dict

class Database(ABC):
    """Abstraction"""
    @abstractmethod
    def query(self, sql: str) -> List[Dict]:
        pass
    
    @abstractmethod
    def execute(self, sql: str, params: tuple = ()) -> bool:
        pass

class MySQLDatabase(Database):
    """Detail depends on abstraction"""
    def query(self, sql: str) -> List[Dict]:
        print(f"MySQL query: {sql}")
        return [{'id': 1, 'name': 'Alice'}]
    
    def execute(self, sql: str, params: tuple = ()) -> bool:
        print(f"MySQL execute: {sql} with {params}")
        return True

class PostgreSQLDatabase(Database):
    def query(self, sql: str) -> List[Dict]:
        print(f"PostgreSQL query: {sql}")
        return [{'id': 1, 'name': 'Alice'}]
    
    def execute(self, sql: str, params: tuple = ()) -> bool:
        print(f"PostgreSQL execute: {sql} with {params}")
        return True

class InMemoryDatabase(Database):
    """สำหรับ testing"""
    def __init__(self):
        self._data = {}
        self._queries = []
    
    def query(self, sql: str) -> List[Dict]:
        self._queries.append(sql)
        return list(self._data.values())
    
    def execute(self, sql: str, params: tuple = ()) -> bool:
        self._queries.append(sql)
        return True

class UserService:
    """High-level module depends on abstraction"""
    def __init__(self, db: Database):  # Injected dependency
        self._db = db
    
    def get_users(self) -> List[Dict]:
        return self._db.query("SELECT * FROM users")
    
    def create_user(self, username: str, email: str) -> bool:
        return self._db.execute(
            "INSERT INTO users (username, email) VALUES (?, ?)",
            (username, email)
        )

# การใช้งาน
service_mysql = UserService(MySQLDatabase())
service_postgres = UserService(PostgreSQLDatabase())
service_test = UserService(InMemoryDatabase())

for name, service in [
    ("MySQL", service_mysql),
    ("PostgreSQL", service_postgres),
    ("InMemory", service_test),
]:
    print(f"\n=== {name} ===")
    users = service.get_users()
    service.create_user("alice", "alice@example.com")
```

```python
# ตัวอย่าง 16: DIP กับ Notification System
from abc import ABC, abstractmethod

class NotificationSender(ABC):
    @abstractmethod
    def send(self, recipient: str, subject: str, body: str) -> bool:
        pass

class EmailSender(NotificationSender):
    def send(self, recipient: str, subject: str, body: str) -> bool:
        print(f"📧 Email to {recipient}: [{subject}] {body}")
        return True

class SMSSender(NotificationSender):
    def send(self, recipient: str, subject: str, body: str) -> bool:
        print(f"📱 SMS to {recipient}: {body[:160]}")
        return True

class SlackSender(NotificationSender):
    def __init__(self, channel: str):
        self.channel = channel
    
    def send(self, recipient: str, subject: str, body: str) -> bool:
        print(f"💬 Slack to #{self.channel}: {subject} - {body}")
        return True

class NotificationService:
    """Depends on abstraction, not concrete sender"""
    def __init__(self, senders: list):
        self._senders = senders
    
    def notify(self, recipient: str, subject: str, body: str):
        for sender in self._senders:
            try:
                sender.send(recipient, subject, body)
            except Exception as e:
                print(f"Failed to send via {type(sender).__name__}: {e}")

# การใช้งาน
service = NotificationService([
    EmailSender(),
    SMSSender(),
    SlackSender("alerts")
])

service.notify("alice@example.com", "Order Confirmed", "Your order #123 has been confirmed!")
```

---

## 6. Clean Code Principles

### 6.1 Meaningful Names

```python
# ตัวอย่าง 17: Meaningful Names
# BAD
def calc(d, r):
    return d * r / 100

x = [{'n': 'Alice', 'a': 30}, {'n': 'Bob', 'a': 25}]
d = []
for i in x:
    if i['a'] >= 18:
        d.append(i['n'])

# GOOD
def calculate_discount_amount(price: float, discount_rate_percent: float) -> float:
    """คำนวณจำนวนเงินส่วนลด"""
    return price * discount_rate_percent / 100

employees = [
    {'name': 'Alice', 'age': 30},
    {'name': 'Bob', 'age': 25}
]

def get_adult_employee_names(employees: list) -> list:
    """ดึงชื่อพนักงานที่บรรลุนิติภาวะ"""
    MINIMUM_ADULT_AGE = 18
    return [
        emp['name'] 
        for emp in employees 
        if emp['age'] >= MINIMUM_ADULT_AGE
    ]

adult_names = get_adult_employee_names(employees)
print(adult_names)
```

```python
# ตัวอย่าง 18: Naming Conventions
# BAD: ชื่อที่ทำให้เข้าใจผิด
def get_active_account(accounts: list) -> list:
    """ชื่อบอกว่า return account เดียว แต่ return list"""
    return [a for a in accounts if a['active']]

# GOOD: ชื่อตรงกับ behavior
def get_active_accounts(accounts: list) -> list:
    return [a for a in accounts if a['active']]

def get_first_active_account(accounts: list):
    active = [a for a in accounts if a['active']]
    return active[0] if active else None

# BAD: ชื่อคลุมเครือ
class Manager:
    pass

class Data:
    pass

# GOOD: ชื่อที่ชัดเจน
class UserAccountManager:
    pass

class UserProfileData:
    pass

# BAD: ย่อเกินไป
class UsrMgr:
    def proc(self, u): pass
    def del_u(self, uid): pass

# GOOD: ชื่อเต็มที่อ่านเข้าใจ
class UserManager:
    def process_new_user(self, user: dict): pass
    def delete_user(self, user_id: int): pass

# Variable naming
# BAD
temp = get_user(id)
if temp:
    process(temp)

# GOOD
user = get_user(user_id)
if user:
    process_user(user)

# Boolean naming
# BAD
flag = True
status = check_user()

# GOOD
is_user_authenticated = True
has_valid_subscription = check_subscription()
can_edit_document = user.has_permission('edit')
```

---

### 6.2 Functions - Small and Single Purpose

```python
# ตัวอย่าง 19: Small Functions
# BAD: Function ใหญ่เกินไป
def process_user_registration(data: dict) -> dict:
    # Validate
    if not data.get('username'):
        return {'error': 'Username required'}
    if len(data['username']) < 3:
        return {'error': 'Username too short'}
    if not data.get('email'):
        return {'error': 'Email required'}
    if '@' not in data['email']:
        return {'error': 'Invalid email'}
    if not data.get('password'):
        return {'error': 'Password required'}
    if len(data['password']) < 8:
        return {'error': 'Password too short'}
    
    # Check duplicate
    import sqlite3
    conn = sqlite3.connect('users.db')
    existing = conn.execute(
        "SELECT id FROM users WHERE username=? OR email=?",
        (data['username'], data['email'])
    ).fetchone()
    if existing:
        return {'error': 'Username or email already exists'}
    
    # Hash password
    import hashlib
    password_hash = hashlib.sha256(data['password'].encode()).hexdigest()
    
    # Save
    conn.execute(
        "INSERT INTO users (username, email, password) VALUES (?, ?, ?)",
        (data['username'], data['email'], password_hash)
    )
    conn.commit()
    
    # Send email
    print(f"Sending welcome email to {data['email']}")
    
    return {'success': True, 'username': data['username']}
```

```python
# GOOD: แยก functions ขนาดเล็กๆ
from dataclasses import dataclass
from typing import Optional, List

@dataclass
class RegistrationData:
    username: str
    email: str
    password: str

@dataclass
class ValidationError:
    field: str
    message: str

def validate_username(username: str) -> Optional[ValidationError]:
    if not username:
        return ValidationError('username', 'Username is required')
    if len(username) < 3:
        return ValidationError('username', 'Username must be at least 3 characters')
    if not username.isalnum():
        return ValidationError('username', 'Username must be alphanumeric')
    return None

def validate_email(email: str) -> Optional[ValidationError]:
    if not email:
        return ValidationError('email', 'Email is required')
    if '@' not in email or '.' not in email.split('@')[-1]:
        return ValidationError('email', 'Invalid email format')
    return None

def validate_password(password: str) -> Optional[ValidationError]:
    if not password:
        return ValidationError('password', 'Password is required')
    if len(password) < 8:
        return ValidationError('password', 'Password must be at least 8 characters')
    if not any(c.isdigit() for c in password):
        return ValidationError('password', 'Password must contain at least one digit')
    return None

def validate_registration(data: RegistrationData) -> List[ValidationError]:
    errors = []
    for validator in [validate_username, validate_email, validate_password]:
        field_name = validator.__name__.replace('validate_', '')
        value = getattr(data, field_name, '')
        error = validator(value)
        if error:
            errors.append(error)
    return errors

def hash_password(password: str) -> str:
    import hashlib
    return hashlib.sha256(password.encode()).hexdigest()

def is_username_available(username: str, db=None) -> bool:
    # Simulated check
    existing_usernames = ['admin', 'root', 'test']
    return username not in existing_usernames

def save_user(data: RegistrationData, password_hash: str, db=None):
    print(f"Saving user: {data.username}")

def send_welcome_notification(email: str, username: str):
    print(f"Sending welcome email to {email}")

def register_user(data: RegistrationData) -> dict:
    """Orchestrates the registration process"""
    errors = validate_registration(data)
    if errors:
        return {'success': False, 'errors': [{'field': e.field, 'message': e.message} for e in errors]}
    
    if not is_username_available(data.username):
        return {'success': False, 'errors': [{'field': 'username', 'message': 'Username already taken'}]}
    
    password_hash = hash_password(data.password)
    save_user(data, password_hash)
    send_welcome_notification(data.email, data.username)
    
    return {'success': True, 'username': data.username}

# ทดสอบ
data = RegistrationData("alice123", "alice@example.com", "SecurePass1")
result = register_user(data)
print(f"Registration result: {result}")

invalid_data = RegistrationData("al", "not-an-email", "short")
result2 = register_user(invalid_data)
print(f"Invalid registration: {result2}")
```

---

### 6.3 Comments

```python
# ตัวอย่าง 20: การใช้ Comments อย่างถูกต้อง

# BAD: Comments ที่ไม่จำเป็น (อธิบายสิ่งที่ code ชัดอยู่แล้ว)
i = i + 1  # increment i by 1
name = user.get_name()  # get the name

# BAD: Comment ที่ outdated
# เช็คว่า user มีสิทธิ์หรือไม่ (แต่ code จริงทำอะไรอื่น)
def check_access(user, resource):
    return resource.owner_id == user.id

# GOOD: Comment อธิบาย WHY ไม่ใช่ WHAT
def calculate_optimal_batch_size(memory_mb: int, item_size_kb: float) -> int:
    # ใช้แค่ 80% ของ memory ที่มี เพื่อเว้นที่สำหรับ overhead
    # และหลีกเลี่ยง OOM errors ใน production
    usable_memory_kb = (memory_mb * 1024) * 0.80
    return max(1, int(usable_memory_kb / item_size_kb))

# GOOD: Comment อธิบาย algorithm ที่ไม่ชัดเจน
def find_next_prime(n: int) -> int:
    """หาจำนวนเฉพาะถัดไปที่มากกว่า n"""
    candidate = n + 1
    while True:
        # ใช้ trial division ถึงแค่ sqrt เพราะ
        # ถ้า n ไม่ใช่จำนวนเฉพาะ ต้องมี factor ที่ <= sqrt(n)
        is_prime = True
        for i in range(2, int(candidate**0.5) + 1):
            if candidate % i == 0:
                is_prime = False
                break
        if is_prime:
            return candidate
        candidate += 1

# GOOD: TODO comments กับ context
def send_email(to: str, subject: str, body: str):
    # TODO(alice): Add retry logic with exponential backoff
    # See: https://github.com/example/issue/123
    print(f"Sending: {subject} to {to}")

# GOOD: Warning comment
def delete_all_users(confirm: bool = False):
    # WARNING: This function is DESTRUCTIVE and cannot be undone.
    # Only call this during testing or initial setup.
    # Never call in production without explicit confirmation.
    if not confirm:
        raise ValueError("Must explicitly confirm deletion with confirm=True")
    print("Deleting all users...")
```

---

### 6.4 Error Handling

```python
# ตัวอย่าง 21: Clean Error Handling
from typing import Optional, Union
from dataclasses import dataclass

# BAD: Return code error handling
def divide_bad(a: float, b: float) -> float:
    if b == 0:
        return -1  # Magic error code!
    return a / b

result = divide_bad(10, 0)
if result == -1:  # ทำยังไงรู้ว่า -1 หมายถึง error?
    print("Error occurred")

# GOOD: Exception-based error handling
class MathError(Exception):
    pass

class DivisionByZeroError(MathError):
    def __init__(self, dividend: float):
        super().__init__(f"Cannot divide {dividend} by zero")
        self.dividend = dividend

def divide_good(a: float, b: float) -> float:
    if b == 0:
        raise DivisionByZeroError(a)
    return a / b

try:
    result = divide_good(10, 0)
except DivisionByZeroError as e:
    print(f"Math error: {e}")
    print(f"Dividend was: {e.dividend}")
```

```python
# ตัวอย่าง 22: Result Pattern สำหรับ Expected Errors
from dataclasses import dataclass
from typing import TypeVar, Generic, Optional

T = TypeVar('T')

@dataclass
class Success(Generic[T]):
    value: T
    
    @property
    def is_success(self) -> bool:
        return True
    
    def unwrap(self) -> T:
        return self.value

@dataclass
class Failure(Generic[T]):
    error: str
    
    @property
    def is_success(self) -> bool:
        return False
    
    def unwrap(self) -> T:
        raise RuntimeError(f"Cannot unwrap Failure: {self.error}")

Result = Union[Success[T], Failure[T]]

def parse_age(value: str) -> Result:
    try:
        age = int(value)
        if age < 0 or age > 150:
            return Failure(f"Age {age} is not in valid range [0, 150]")
        return Success(age)
    except ValueError:
        return Failure(f"'{value}' is not a valid integer")

def create_user_profile(name: str, age_str: str) -> Result:
    age_result = parse_age(age_str)
    if not age_result.is_success:
        return Failure(f"Invalid age: {age_result.error}")
    
    return Success({
        'name': name,
        'age': age_result.unwrap()
    })

# การใช้งาน
test_cases = [
    ("Alice", "30"),
    ("Bob", "abc"),
    ("Charlie", "-5"),
    ("Diana", "25"),
]

for name, age_str in test_cases:
    result = create_user_profile(name, age_str)
    if result.is_success:
        print(f"✅ Created: {result.unwrap()}")
    else:
        print(f"❌ Error: {result.error}")
```

---

### 6.5 DRY, KISS, YAGNI

```python
# ตัวอย่าง 23: DRY - Don't Repeat Yourself
# BAD: Code ซ้ำ
def validate_user_email_bad(user_data: dict) -> bool:
    email = user_data.get('email', '')
    if not email:
        return False
    if '@' not in email:
        return False
    parts = email.split('@')
    if len(parts) != 2:
        return False
    if '.' not in parts[1]:
        return False
    return True

def validate_admin_email_bad(admin_data: dict) -> bool:
    email = admin_data.get('email', '')
    if not email:
        return False
    if '@' not in email:
        return False
    parts = email.split('@')
    if len(parts) != 2:
        return False
    if '.' not in parts[1]:
        return False
    return True

# GOOD: DRY
import re

def is_valid_email(email: str) -> bool:
    """Validate email format"""
    if not email:
        return False
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def validate_entity_email(entity_data: dict) -> bool:
    return is_valid_email(entity_data.get('email', ''))

# ใช้งาน
validate_user_email = validate_entity_email
validate_admin_email = validate_entity_email

print(is_valid_email("alice@example.com"))  # True
print(is_valid_email("not-an-email"))       # False
```

```python
# ตัวอย่าง 24: KISS - Keep It Simple, Stupid
# BAD: Over-engineered
class AbstractUserFactoryInterface:
    pass

class UserFactoryImplementation(AbstractUserFactoryInterface):
    def create_user_with_default_settings(self, username, email, password):
        return {
            'username': username,
            'email': email,
            'password': password,
            'settings': {}
        }

factory = UserFactoryImplementation()
user = factory.create_user_with_default_settings("alice", "alice@example.com", "pass")

# GOOD: Simple and direct
def create_user(username: str, email: str, password: str) -> dict:
    return {
        'username': username,
        'email': email,
        'password': password,
        'settings': {}
    }

user = create_user("alice", "alice@example.com", "pass")
```

```python
# ตัวอย่าง 25: YAGNI - You Aren't Gonna Need It
# BAD: Build features "just in case"
class UserProfile:
    def __init__(self, name: str):
        self.name = name
        # "might need these later"
        self.avatar_url = None
        self.bio = None
        self.social_links = {}
        self.preferences = {}
        self.notification_settings = {}
        self.payment_methods = []
        self.shipping_addresses = []
        self.loyalty_points = 0
        self.subscription_tier = 'free'
    
    # ยังไม่มี requirement แต่สร้างไว้ก่อน
    def calculate_loyalty_discount(self): pass
    def upgrade_subscription(self): pass
    def sync_social_accounts(self): pass

# GOOD: Build what you need now
class UserProfile:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email
        self.created_at = datetime.now()
    
    # เพิ่มตาม requirement จริง
    def display_name(self) -> str:
        return self.name or self.email.split('@')[0]

from datetime import datetime
```

---

## 7. Refactoring Techniques

```python
# ตัวอย่าง 26: Extract Method
# BEFORE
def print_invoice(order: dict):
    print("=" * 40)
    print(f"Invoice #{order['id']}")
    print("=" * 40)
    total = 0
    for item in order['items']:
        subtotal = item['price'] * item['quantity']
        total += subtotal
        print(f"{item['name']:<20} ${subtotal:.2f}")
    print("-" * 40)
    tax = total * 0.07
    print(f"{'Subtotal:':<20} ${total:.2f}")
    print(f"{'Tax (7%):':<20} ${tax:.2f}")
    print(f"{'Total:':<20} ${total + tax:.2f}")

# AFTER: Extract Method
def print_header(invoice_id: int):
    print("=" * 40)
    print(f"Invoice #{invoice_id}")
    print("=" * 40)

def print_line_items(items: list) -> float:
    total = 0
    for item in items:
        subtotal = item['price'] * item['quantity']
        total += subtotal
        print(f"{item['name']:<20} ${subtotal:.2f}")
    return total

def calculate_tax(amount: float, rate: float = 0.07) -> float:
    return amount * rate

def print_totals(subtotal: float, tax: float):
    print("-" * 40)
    print(f"{'Subtotal:':<20} ${subtotal:.2f}")
    print(f"{'Tax (7%):':<20} ${tax:.2f}")
    print(f"{'Total:':<20} ${subtotal + tax:.2f}")

def print_invoice_clean(order: dict):
    print_header(order['id'])
    subtotal = print_line_items(order['items'])
    tax = calculate_tax(subtotal)
    print_totals(subtotal, tax)

# ทดสอบ
order = {
    'id': 1001,
    'items': [
        {'name': 'Python Book', 'price': 599, 'quantity': 1},
        {'name': 'USB Hub', 'price': 299, 'quantity': 2},
    ]
}
print_invoice_clean(order)
```

```python
# ตัวอย่าง 27: Replace Conditional with Polymorphism
# BEFORE: Long if-elif chain
def calculate_area_bad(shape_type: str, *dims) -> float:
    if shape_type == 'circle':
        import math
        return math.pi * dims[0] ** 2
    elif shape_type == 'rectangle':
        return dims[0] * dims[1]
    elif shape_type == 'triangle':
        return 0.5 * dims[0] * dims[1]
    elif shape_type == 'square':
        return dims[0] ** 2
    else:
        raise ValueError(f"Unknown shape: {shape_type}")

# AFTER: Polymorphism
import math
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
    
    def area(self) -> float:
        return math.pi * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height
    
    def area(self) -> float:
        return self.width * self.height

class Triangle(Shape):
    def __init__(self, base: float, height: float):
        self.base = base
        self.height = height
    
    def area(self) -> float:
        return 0.5 * self.base * self.height

# ไม่มี if-elif แล้ว
def calculate_total_area(shapes: list) -> float:
    return sum(shape.area() for shape in shapes)

shapes = [Circle(5), Rectangle(4, 6), Triangle(3, 8)]
for shape in shapes:
    print(f"{shape.__class__.__name__}: {shape.area():.2f}")
print(f"Total: {calculate_total_area(shapes):.2f}")
```

```python
# ตัวอย่าง 28: Replace Magic Numbers with Constants
# BEFORE
def is_eligible_for_loan(credit_score: int, annual_income: float, existing_debt: float) -> bool:
    if credit_score < 650:
        return False
    if annual_income < 30000:
        return False
    debt_ratio = existing_debt / annual_income
    if debt_ratio > 0.43:
        return False
    return True

# AFTER
MINIMUM_CREDIT_SCORE = 650
MINIMUM_ANNUAL_INCOME = 30_000.0
MAXIMUM_DEBT_TO_INCOME_RATIO = 0.43

def is_eligible_for_loan(
    credit_score: int, 
    annual_income: float, 
    existing_debt: float
) -> bool:
    has_good_credit = credit_score >= MINIMUM_CREDIT_SCORE
    has_sufficient_income = annual_income >= MINIMUM_ANNUAL_INCOME
    
    debt_ratio = existing_debt / annual_income if annual_income > 0 else float('inf')
    has_acceptable_debt = debt_ratio <= MAXIMUM_DEBT_TO_INCOME_RATIO
    
    return has_good_credit and has_sufficient_income and has_acceptable_debt

# ทดสอบ
test_cases = [
    (720, 50000, 15000, True),
    (600, 50000, 15000, False),  # Low credit
    (720, 25000, 5000, False),   # Low income
    (720, 50000, 25000, False),  # High debt ratio
]

for score, income, debt, expected in test_cases:
    result = is_eligible_for_loan(score, income, debt)
    status = "✅" if result == expected else "❌"
    print(f"{status} Score:{score}, Income:{income}, Debt:{debt} -> {result}")
```

---

## 8. Code Smells

```python
# ตัวอย่าง 29: Code Smells และวิธีแก้

# Smell 1: Long Parameter List
def create_user_bad(
    first_name: str, last_name: str, email: str, 
    phone: str, address: str, city: str, country: str,
    age: int, gender: str, occupation: str
) -> dict:
    return locals()

# FIX: Parameter Object
from dataclasses import dataclass

@dataclass
class PersonalInfo:
    first_name: str
    last_name: str
    age: int
    gender: str
    occupation: str

@dataclass
class ContactInfo:
    email: str
    phone: str
    address: str
    city: str
    country: str

def create_user_good(personal: PersonalInfo, contact: ContactInfo) -> dict:
    return {**vars(personal), **vars(contact)}
```

```python
# ตัวอย่าง 30: Smell - Data Clumps
# BAD: ข้อมูลที่ต้องใช้ด้วยกันเสมอ
def send_money(sender_account: str, sender_bank: str, 
               receiver_account: str, receiver_bank: str,
               amount: float):
    pass

def validate_transaction(account: str, bank: str, amount: float):
    pass

# FIX: Introduce Data Class
@dataclass
class BankAccount:
    account_number: str
    bank_code: str
    
    def __str__(self):
        return f"{self.bank_code}:{self.account_number}"

def send_money_clean(sender: BankAccount, receiver: BankAccount, amount: float):
    print(f"Transfer ${amount} from {sender} to {receiver}")

def validate_transaction_clean(account: BankAccount, amount: float):
    print(f"Validating {account} for ${amount}")

sender = BankAccount("1234567890", "KBTH")
receiver = BankAccount("0987654321", "SCB")
send_money_clean(sender, receiver, 1000.0)
```

```python
# ตัวอย่าง 31: Smell - Feature Envy
# BAD: Method ใช้ data ของ class อื่นมากกว่า class ตัวเอง
class Order:
    def __init__(self, items: list, customer: 'Customer'):
        self.items = items
        self.customer = customer
    
    def get_discount_amount(self) -> float:
        # "Feature Envy" - method นี้อยากเป็น method ของ Customer
        if self.customer.membership_years >= 5:
            return sum(i['price'] for i in self.items) * 0.15
        elif self.customer.membership_years >= 2:
            return sum(i['price'] for i in self.items) * 0.10
        elif self.customer.is_student:
            return sum(i['price'] for i in self.items) * 0.05
        return 0

# FIX: Move method to the class it envies
class Customer:
    def __init__(self, membership_years: int, is_student: bool):
        self.membership_years = membership_years
        self.is_student = is_student
    
    def get_discount_rate(self) -> float:
        if self.membership_years >= 5:
            return 0.15
        elif self.membership_years >= 2:
            return 0.10
        elif self.is_student:
            return 0.05
        return 0.0

class OrderClean:
    def __init__(self, items: list, customer: Customer):
        self.items = items
        self.customer = customer
    
    @property
    def subtotal(self) -> float:
        return sum(i['price'] for i in self.items)
    
    def get_discount_amount(self) -> float:
        return self.subtotal * self.customer.get_discount_rate()
    
    def total(self) -> float:
        return self.subtotal - self.get_discount_amount()

# ทดสอบ
customer = Customer(membership_years=3, is_student=False)
order = OrderClean(
    items=[{'price': 500}, {'price': 300}],
    customer=customer
)
print(f"Subtotal: ${order.subtotal}")
print(f"Discount: ${order.get_discount_amount()}")
print(f"Total: ${order.total()}")
```

---

## 9. Unit Testing ใน Clean Code

```python
# ตัวอย่าง 32: Clean Unit Tests
import unittest
from typing import Optional

# Code to test
class BankAccount:
    def __init__(self, initial_balance: float = 0):
        if initial_balance < 0:
            raise ValueError("Initial balance cannot be negative")
        self._balance = initial_balance
        self._transactions = []
    
    def deposit(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        self._transactions.append(('deposit', amount))
        return self._balance
    
    def withdraw(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self._balance:
            raise ValueError(f"Insufficient funds: balance ${self._balance}, requested ${amount}")
        self._balance -= amount
        self._transactions.append(('withdrawal', amount))
        return self._balance
    
    @property
    def balance(self) -> float:
        return self._balance
    
    def transaction_count(self) -> int:
        return len(self._transactions)

class TestBankAccount(unittest.TestCase):
    
    def setUp(self):
        """Called before each test"""
        self.account = BankAccount(initial_balance=1000.0)
    
    # Test naming: test_<method>_<scenario>_<expected_result>
    def test_deposit_positive_amount_increases_balance(self):
        new_balance = self.account.deposit(500.0)
        self.assertEqual(new_balance, 1500.0)
        self.assertEqual(self.account.balance, 1500.0)
    
    def test_withdraw_valid_amount_decreases_balance(self):
        new_balance = self.account.withdraw(300.0)
        self.assertEqual(new_balance, 700.0)
    
    def test_withdraw_exact_balance_leaves_zero(self):
        new_balance = self.account.withdraw(1000.0)
        self.assertEqual(new_balance, 0.0)
    
    def test_withdraw_more_than_balance_raises_error(self):
        with self.assertRaises(ValueError) as context:
            self.account.withdraw(1500.0)
        self.assertIn("Insufficient funds", str(context.exception))
    
    def test_deposit_negative_amount_raises_error(self):
        with self.assertRaises(ValueError):
            self.account.deposit(-100.0)
    
    def test_deposit_zero_raises_error(self):
        with self.assertRaises(ValueError):
            self.account.deposit(0)
    
    def test_initial_negative_balance_raises_error(self):
        with self.assertRaises(ValueError):
            BankAccount(-500)
    
    def test_transaction_count_tracks_operations(self):
        self.account.deposit(200)
        self.account.withdraw(100)
        self.account.deposit(50)
        self.assertEqual(self.account.transaction_count(), 3)
    
    def test_multiple_deposits_accumulate(self):
        for amount in [100, 200, 300]:
            self.account.deposit(amount)
        self.assertEqual(self.account.balance, 1600.0)

# รัน tests
if __name__ == '__main__':
    unittest.main(verbosity=2)
else:
    # รันใน script
    loader = unittest.TestLoader()
    suite = loader.loadTestsFromTestCase(TestBankAccount)
    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)
```

---

## 10. Boundaries & Modules

```python
# ตัวอย่าง 33: Clean Boundaries
from abc import ABC, abstractmethod
from typing import Any

# Boundary Interface - กั้นระหว่าง domain และ external service
class StorageInterface(ABC):
    @abstractmethod
    def save(self, key: str, data: Any) -> bool:
        pass
    
    @abstractmethod
    def load(self, key: str) -> Any:
        pass
    
    @abstractmethod
    def delete(self, key: str) -> bool:
        pass

# Concrete implementation (อยู่นอก domain)
class RedisStorage(StorageInterface):
    def save(self, key: str, data: Any) -> bool:
        print(f"Redis SAVE {key}: {data}")
        return True
    
    def load(self, key: str) -> Any:
        print(f"Redis LOAD {key}")
        return None
    
    def delete(self, key: str) -> bool:
        print(f"Redis DELETE {key}")
        return True

class FileStorage(StorageInterface):
    def save(self, key: str, data: Any) -> bool:
        print(f"File SAVE {key}: {data}")
        return True
    
    def load(self, key: str) -> Any:
        print(f"File LOAD {key}")
        return None
    
    def delete(self, key: str) -> bool:
        print(f"File DELETE {key}")
        return True

# Domain code - ใช้แค่ interface
class SessionManager:
    def __init__(self, storage: StorageInterface):
        self._storage = storage
    
    def create_session(self, user_id: str) -> str:
        import uuid
        session_id = str(uuid.uuid4())
        self._storage.save(f"session:{session_id}", {'user_id': user_id})
        return session_id
    
    def validate_session(self, session_id: str) -> bool:
        data = self._storage.load(f"session:{session_id}")
        return data is not None
    
    def destroy_session(self, session_id: str):
        self._storage.delete(f"session:{session_id}")

# การใช้งาน
session_mgr = SessionManager(RedisStorage())
session_id = session_mgr.create_session("user123")
print(f"Session: {session_id}")
print(f"Valid: {session_mgr.validate_session(session_id)}")
session_mgr.destroy_session(session_id)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: SRP - Refactor Invoice System
แยก `InvoiceProcessor` ออกเป็น multiple classes ตาม SRP

```python
# เฉลย
from dataclasses import dataclass, field
from typing import List, Optional
from datetime import datetime

@dataclass
class InvoiceItem:
    description: str
    quantity: float
    unit_price: float
    
    @property
    def subtotal(self) -> float:
        return self.quantity * self.unit_price

@dataclass
class Invoice:
    invoice_number: str
    customer_name: str
    customer_email: str
    items: List[InvoiceItem]
    tax_rate: float = 0.07
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
    
    @property
    def subtotal(self) -> float:
        return sum(item.subtotal for item in self.items)
    
    @property
    def tax_amount(self) -> float:
        return self.subtotal * self.tax_rate
    
    @property
    def total(self) -> float:
        return self.subtotal + self.tax_amount

class InvoiceValidator:
    def validate(self, invoice: Invoice) -> List[str]:
        errors = []
        if not invoice.invoice_number:
            errors.append("Invoice number is required")
        if not invoice.customer_name:
            errors.append("Customer name is required")
        if not invoice.items:
            errors.append("Invoice must have at least one item")
        for item in invoice.items:
            if item.quantity <= 0:
                errors.append(f"Invalid quantity for '{item.description}'")
            if item.unit_price < 0:
                errors.append(f"Invalid price for '{item.description}'")
        return errors

class InvoiceFormatter:
    def format_text(self, invoice: Invoice) -> str:
        lines = [
            f"Invoice #{invoice.invoice_number}",
            f"Date: {invoice.created_at[:10]}",
            f"Customer: {invoice.customer_name}",
            "=" * 50,
        ]
        for item in invoice.items:
            lines.append(f"{item.description:<30} {item.quantity:>5} x ${item.unit_price:.2f} = ${item.subtotal:.2f}")
        lines.extend([
            "=" * 50,
            f"{'Subtotal:':<40} ${invoice.subtotal:.2f}",
            f"{'Tax ({:.0f}%):'.format(invoice.tax_rate*100):<40} ${invoice.tax_amount:.2f}",
            f"{'Total:':<40} ${invoice.total:.2f}",
        ])
        return "\n".join(lines)

class InvoiceEmailSender:
    def send(self, invoice: Invoice, formatted: str) -> bool:
        print(f"Sending invoice to {invoice.customer_email}")
        print(f"Content preview: {formatted[:100]}...")
        return True

class InvoiceRepository:
    def __init__(self):
        self._storage = {}
    
    def save(self, invoice: Invoice) -> bool:
        self._storage[invoice.invoice_number] = invoice
        return True
    
    def find(self, invoice_number: str) -> Optional[Invoice]:
        return self._storage.get(invoice_number)

class InvoiceProcessor:
    def __init__(
        self,
        validator: InvoiceValidator,
        formatter: InvoiceFormatter,
        sender: InvoiceEmailSender,
        repository: InvoiceRepository
    ):
        self.validator = validator
        self.formatter = formatter
        self.sender = sender
        self.repository = repository
    
    def process(self, invoice: Invoice) -> dict:
        errors = self.validator.validate(invoice)
        if errors:
            return {'success': False, 'errors': errors}
        
        formatted = self.formatter.format_text(invoice)
        self.repository.save(invoice)
        self.sender.send(invoice, formatted)
        
        return {'success': True, 'invoice_number': invoice.invoice_number}

# ทดสอบ
invoice = Invoice(
    invoice_number="INV-2024-001",
    customer_name="Alice Smith",
    customer_email="alice@example.com",
    items=[
        InvoiceItem("Python Course", 1, 999.00),
        InvoiceItem("Docker Workshop", 2, 299.00),
    ]
)

processor = InvoiceProcessor(
    InvoiceValidator(),
    InvoiceFormatter(),
    InvoiceEmailSender(),
    InvoiceRepository()
)

result = processor.process(invoice)
print(f"Result: {result}")
print("\n" + InvoiceFormatter().format_text(invoice))
```

### แบบฝึกหัดที่ 2 - 8

**แบบฝึกหัดที่ 2**: OCP - สร้าง Notification system ที่ extensible โดยไม่ต้อง modify เมื่อเพิ่ม channel ใหม่

**แบบฝึกหัดที่ 3**: LSP - สร้าง Shape hierarchy ที่ถูกต้อง (Polygon, ConvexPolygon, RegularPolygon)

**แบบฝึกหัดที่ 4**: ISP - Refactor UserRepository ขนาดใหญ่เป็น interfaces ย่อยๆ

**แบบฝึกหัดที่ 5**: DIP - Refactor ReportGenerator ให้ depend on abstractions

**แบบฝึกหัดที่ 6**: Clean Code - Refactor code ที่ให้มาให้ใช้ meaningful names, extract methods

**แบบฝึกหัดที่ 7**: เขียน Unit Tests สำหรับ Calculator class ที่ครอบคลุม edge cases

**แบบฝึกหัดที่ 8**: Identify code smells ใน legacy code ที่ให้มา และ refactor

---

## สรุป

| หลักการ | คำย่อ | ใจความ |
|---------|-------|--------|
| Single Responsibility | SRP | Class ทำหน้าที่เดียว |
| Open/Closed | OCP | Extend ได้, Modify ไม่ได้ |
| Liskov Substitution | LSP | Subtype ทำงานแทน Base type ได้ |
| Interface Segregation | ISP | Interface เล็กและเฉพาะเจาะจง |
| Dependency Inversion | DIP | Depend on Abstractions |

Clean Code ไม่ใช่แค่ทำให้โค้ดทำงานได้ แต่ทำให้โค้ด **อ่านง่าย**, **เข้าใจง่าย**, **แก้ไขง่าย**, และ **ทดสอบง่าย**

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

---

*Part 87 เสร็จสมบูรณ์ | ต่อไป: Part 88 - Clean Architecture & Domain-Driven Design*
