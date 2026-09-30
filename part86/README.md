# Part 86: Design Patterns in Python

## บทนำ

Design Patterns หรือรูปแบบการออกแบบซอฟต์แวร์ คือแนวทางการแก้ปัญหาที่ถูกนำมาใช้ซ้ำ (reusable solutions) สำหรับปัญหาที่พบบ่อยในการออกแบบซอฟต์แวร์ มาจากหนังสือ "Design Patterns: Elements of Reusable Object-Oriented Software" โดย Gang of Four (GoF) ประกอบด้วย Erich Gamma, Richard Helm, Ralph Johnson และ John Vlissides

---

## 1. Design Patterns Overview (GoF)

### ทำไมต้องใช้ Design Patterns?

1. **Reusability** - นำกลับมาใช้ซ้ำได้
2. **Communication** - ทำให้สื่อสารกันในทีมได้ง่ายขึ้น
3. **Best Practices** - เป็นวิธีที่ผ่านการพิสูจน์แล้วว่าใช้งานได้ดี
4. **Maintainability** - ง่ายต่อการดูแลรักษา

### 3 หมวดหมู่หลัก

| หมวด | คำอธิบาย | จำนวน Pattern |
|------|----------|---------------|
| **Creational** | เกี่ยวกับการสร้าง Object | 5 |
| **Structural** | เกี่ยวกับโครงสร้างของ Class และ Object | 7 |
| **Behavioral** | เกี่ยวกับพฤติกรรมและการสื่อสารระหว่าง Object | 11 |

```python
# ตัวอย่าง 1: ทำความเข้าใจ Pattern พื้นฐาน
# Pattern ไม่ใช่ code ที่ copy-paste ได้ แต่เป็นแนวคิด

# ไม่ดี - ไม่มี pattern
class UserService:
    def __init__(self):
        self.db = DatabaseConnection()  # hard dependency
        self.email = EmailService()     # hard dependency
    
    def register(self, username, email):
        self.db.save(username, email)
        self.email.send_welcome(email)

# ดีกว่า - ใช้ Dependency Injection (pattern)
class UserService:
    def __init__(self, db, email_service):
        self.db = db
        self.email_service = email_service
    
    def register(self, username, email):
        self.db.save(username, email)
        self.email_service.send_welcome(email)
```

---

## 2. Creational Patterns

### 2.1 Singleton Pattern

**ความหมาย**: รับประกันว่า Class มีเพียง instance เดียว และให้ global access point ต่อ instance นั้น

**ใช้เมื่อ**:
- ต้องการ shared resource (เช่น database connection, logger)
- ต้องการ global state ที่ควบคุมได้

```python
# ตัวอย่าง 2: Singleton แบบ Classic
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.initialized = True
            self.data = {}

# ทดสอบ
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True - เป็น instance เดียวกัน
s1.data['key'] = 'value'
print(s2.data)   # {'key': 'value'}
```

```python
# ตัวอย่าง 3: Singleton แบบ Thread-Safe
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# ทดสอบ Thread Safety
def create_instance():
    instance = ThreadSafeSingleton()
    print(f"Thread {threading.current_thread().name}: {id(instance)}")

threads = [threading.Thread(target=create_instance) for _ in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

```python
# ตัวอย่าง 4: Singleton แบบ Decorator
def singleton(cls):
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self):
        print("Creating database connection...")
        self.connection_string = "postgresql://localhost:5432/mydb"
    
    def query(self, sql):
        return f"Executing: {sql}"

# ทดสอบ
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2)  # True
```

```python
# ตัวอย่าง 5: Singleton แบบ Metaclass
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self):
        self.logs = []
    
    def log(self, message):
        self.logs.append(message)
        print(f"[LOG] {message}")
    
    def get_logs(self):
        return self.logs

# ทดสอบ
logger1 = Logger()
logger2 = Logger()
logger1.log("First message")
logger2.log("Second message")
print(logger1.get_logs())  # ['First message', 'Second message']
print(logger1 is logger2)  # True
```

```python
# ตัวอย่าง 6: Config Manager ด้วย Singleton
class ConfigManager:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._config = {}
        return cls._instance
    
    def set(self, key, value):
        self._config[key] = value
    
    def get(self, key, default=None):
        return self._config.get(key, default)
    
    def load_from_dict(self, config_dict):
        self._config.update(config_dict)

# การใช้งาน
config = ConfigManager()
config.load_from_dict({
    'database_url': 'postgresql://localhost/mydb',
    'debug': True,
    'max_connections': 10
})

# ในส่วนอื่นของโปรแกรม
config2 = ConfigManager()
print(config2.get('database_url'))  # postgresql://localhost/mydb
print(config2.get('debug'))         # True
```

---

### 2.2 Factory Method Pattern

**ความหมาย**: กำหนด interface สำหรับสร้าง object แต่ให้ subclass ตัดสินใจว่าจะสร้าง class ไหน

**ใช้เมื่อ**:
- ไม่รู้ล่วงหน้าว่าจะสร้าง object ชนิดไหน
- ต้องการให้ subclass ควบคุมการสร้าง object

```python
# ตัวอย่าง 7: Factory Method พื้นฐาน
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"
    
    def move(self):
        return "Running on 4 legs"

class Cat(Animal):
    def speak(self):
        return "Meow!"
    
    def move(self):
        return "Walking gracefully"

class Bird(Animal):
    def speak(self):
        return "Tweet!"
    
    def move(self):
        return "Flying with wings"

class AnimalFactory(ABC):
    @abstractmethod
    def create_animal(self) -> Animal:
        pass
    
    def get_animal_description(self):
        animal = self.create_animal()
        return f"Sound: {animal.speak()}, Movement: {animal.move()}"

class DogFactory(AnimalFactory):
    def create_animal(self) -> Animal:
        return Dog()

class CatFactory(AnimalFactory):
    def create_animal(self) -> Animal:
        return Cat()

# การใช้งาน
factories = [DogFactory(), CatFactory()]
for factory in factories:
    print(factory.get_animal_description())
```

```python
# ตัวอย่าง 8: Factory Method สำหรับ Payment System
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> str:
        pass
    
    @abstractmethod
    def refund(self, transaction_id: str) -> str:
        pass

class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> str:
        return f"Processing ${amount} via Credit Card"
    
    def refund(self, transaction_id: str) -> str:
        return f"Refunding transaction {transaction_id} via Credit Card"

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> str:
        return f"Processing ${amount} via PayPal"
    
    def refund(self, transaction_id: str) -> str:
        return f"Refunding transaction {transaction_id} via PayPal"

class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> str:
        return f"Processing ${amount} via Stripe"
    
    def refund(self, transaction_id: str) -> str:
        return f"Refunding transaction {transaction_id} via Stripe"

class PaymentFactory:
    @staticmethod
    def create_processor(payment_type: str) -> PaymentProcessor:
        processors = {
            'credit_card': CreditCardProcessor,
            'paypal': PayPalProcessor,
            'stripe': StripeProcessor,
        }
        
        if payment_type not in processors:
            raise ValueError(f"Unknown payment type: {payment_type}")
        
        return processors[payment_type]()

# การใช้งาน
payment_types = ['credit_card', 'paypal', 'stripe']
for pt in payment_types:
    processor = PaymentFactory.create_processor(pt)
    print(processor.process_payment(100.00))
```

---

### 2.3 Abstract Factory Pattern

**ความหมาย**: ให้ interface สำหรับสร้าง family ของ objects ที่เกี่ยวข้องกัน โดยไม่ต้องระบุ concrete class

```python
# ตัวอย่าง 9: Abstract Factory สำหรับ UI Components
from abc import ABC, abstractmethod

# Abstract Products
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
    def validate(self, value: str) -> bool:
        pass

# Concrete Products - Windows
class WindowsButton(Button):
    def render(self) -> str:
        return "<button class='windows-btn'>Click</button>"
    
    def on_click(self) -> str:
        return "Windows button clicked!"

class WindowsTextInput(TextInput):
    def render(self) -> str:
        return "<input class='windows-input' />"
    
    def validate(self, value: str) -> bool:
        return len(value) > 0

# Concrete Products - macOS
class MacButton(Button):
    def render(self) -> str:
        return "<button class='mac-btn'>Click</button>"
    
    def on_click(self) -> str:
        return "Mac button clicked!"

class MacTextInput(TextInput):
    def render(self) -> str:
        return "<input class='mac-input' />"
    
    def validate(self, value: str) -> bool:
        return len(value) > 0

# Abstract Factory
class UIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_text_input(self) -> TextInput:
        pass

# Concrete Factories
class WindowsUIFactory(UIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_text_input(self) -> TextInput:
        return WindowsTextInput()

class MacUIFactory(UIFactory):
    def create_button(self) -> Button:
        return MacButton()
    
    def create_text_input(self) -> TextInput:
        return MacTextInput()

# Client code
def create_login_form(factory: UIFactory):
    button = factory.create_button()
    text_input = factory.create_text_input()
    
    print(f"Button: {button.render()}")
    print(f"Input: {text_input.render()}")
    print(f"Button click: {button.on_click()}")

# การใช้งาน
import platform
if platform.system() == 'Windows':
    factory = WindowsUIFactory()
else:
    factory = MacUIFactory()

create_login_form(factory)
```

```python
# ตัวอย่าง 10: Abstract Factory สำหรับ Database
from abc import ABC, abstractmethod
from typing import List, Dict

class DatabaseConnection(ABC):
    @abstractmethod
    def connect(self, url: str) -> bool:
        pass
    
    @abstractmethod
    def execute(self, query: str) -> List[Dict]:
        pass

class QueryBuilder(ABC):
    @abstractmethod
    def select(self, table: str, columns: List[str]) -> str:
        pass
    
    @abstractmethod
    def insert(self, table: str, data: Dict) -> str:
        pass

# PostgreSQL Implementation
class PostgreSQLConnection(DatabaseConnection):
    def connect(self, url: str) -> bool:
        print(f"Connecting to PostgreSQL: {url}")
        return True
    
    def execute(self, query: str) -> List[Dict]:
        print(f"PostgreSQL executing: {query}")
        return [{"id": 1, "name": "test"}]

class PostgreSQLQueryBuilder(QueryBuilder):
    def select(self, table: str, columns: List[str]) -> str:
        cols = ", ".join(columns)
        return f'SELECT {cols} FROM "{table}"'
    
    def insert(self, table: str, data: Dict) -> str:
        cols = ", ".join(data.keys())
        vals = ", ".join([f"'{v}'" for v in data.values()])
        return f'INSERT INTO "{table}" ({cols}) VALUES ({vals})'

# MySQL Implementation  
class MySQLConnection(DatabaseConnection):
    def connect(self, url: str) -> bool:
        print(f"Connecting to MySQL: {url}")
        return True
    
    def execute(self, query: str) -> List[Dict]:
        print(f"MySQL executing: {query}")
        return [{"id": 1, "name": "test"}]

class MySQLQueryBuilder(QueryBuilder):
    def select(self, table: str, columns: List[str]) -> str:
        cols = ", ".join(columns)
        return f"SELECT {cols} FROM `{table}`"
    
    def insert(self, table: str, data: Dict) -> str:
        cols = ", ".join(data.keys())
        vals = ", ".join([f"'{v}'" for v in data.values()])
        return f"INSERT INTO `{table}` ({cols}) VALUES ({vals})"

# Abstract Factory
class DatabaseFactory(ABC):
    @abstractmethod
    def create_connection(self) -> DatabaseConnection:
        pass
    
    @abstractmethod
    def create_query_builder(self) -> QueryBuilder:
        pass

class PostgreSQLFactory(DatabaseFactory):
    def create_connection(self) -> DatabaseConnection:
        return PostgreSQLConnection()
    
    def create_query_builder(self) -> QueryBuilder:
        return PostgreSQLQueryBuilder()

class MySQLFactory(DatabaseFactory):
    def create_connection(self) -> DatabaseConnection:
        return MySQLConnection()
    
    def create_query_builder(self) -> QueryBuilder:
        return MySQLQueryBuilder()

# การใช้งาน
def setup_database(factory: DatabaseFactory, db_url: str):
    conn = factory.create_connection()
    qb = factory.create_query_builder()
    
    conn.connect(db_url)
    
    query = qb.select("users", ["id", "name", "email"])
    results = conn.execute(query)
    print(f"Query: {query}")
    print(f"Results: {results}")
    
    insert_query = qb.insert("users", {"name": "Alice", "email": "alice@example.com"})
    print(f"Insert: {insert_query}")

# ทดสอบ
print("=== PostgreSQL ===")
setup_database(PostgreSQLFactory(), "postgresql://localhost:5432/mydb")

print("\n=== MySQL ===")
setup_database(MySQLFactory(), "mysql://localhost:3306/mydb")
```

---

### 2.4 Builder Pattern

**ความหมาย**: แยกกระบวนการสร้าง object ออกจาก representation ของมัน เพื่อให้กระบวนการสร้างเดียวกันสร้าง representations ที่ต่างกันได้

```python
# ตัวอย่าง 11: Builder สำหรับ Query Builder
class SQLQuery:
    def __init__(self):
        self.table = None
        self.columns = []
        self.conditions = []
        self.order_by = None
        self.limit = None
        self.offset = None
        self.joins = []

class SQLQueryBuilder:
    def __init__(self):
        self._query = SQLQuery()
    
    def from_table(self, table: str) -> 'SQLQueryBuilder':
        self._query.table = table
        return self
    
    def select(self, *columns) -> 'SQLQueryBuilder':
        self._query.columns.extend(columns)
        return self
    
    def where(self, condition: str) -> 'SQLQueryBuilder':
        self._query.conditions.append(condition)
        return self
    
    def order_by(self, column: str, direction: str = 'ASC') -> 'SQLQueryBuilder':
        self._query.order_by = f"{column} {direction}"
        return self
    
    def limit(self, n: int) -> 'SQLQueryBuilder':
        self._query.limit = n
        return self
    
    def offset(self, n: int) -> 'SQLQueryBuilder':
        self._query.offset = n
        return self
    
    def join(self, table: str, on: str, join_type: str = 'INNER') -> 'SQLQueryBuilder':
        self._query.joins.append(f"{join_type} JOIN {table} ON {on}")
        return self
    
    def build(self) -> str:
        if not self._query.table:
            raise ValueError("Table is required")
        
        cols = ", ".join(self._query.columns) if self._query.columns else "*"
        query = f"SELECT {cols} FROM {self._query.table}"
        
        for join in self._query.joins:
            query += f" {join}"
        
        if self._query.conditions:
            conditions = " AND ".join(self._query.conditions)
            query += f" WHERE {conditions}"
        
        if self._query.order_by:
            query += f" ORDER BY {self._query.order_by}"
        
        if self._query.limit is not None:
            query += f" LIMIT {self._query.limit}"
        
        if self._query.offset is not None:
            query += f" OFFSET {self._query.offset}"
        
        return query

# การใช้งาน
query = (SQLQueryBuilder()
    .from_table("users")
    .select("id", "name", "email")
    .join("orders", "users.id = orders.user_id")
    .where("users.active = true")
    .where("orders.total > 100")
    .order_by("users.name")
    .limit(10)
    .offset(20)
    .build()
)

print(query)
```

```python
# ตัวอย่าง 12: Builder สำหรับ HTTP Request
class HTTPRequest:
    def __init__(self):
        self.method = 'GET'
        self.url = ''
        self.headers = {}
        self.params = {}
        self.body = None
        self.timeout = 30
        self.auth = None

class HTTPRequestBuilder:
    def __init__(self):
        self._request = HTTPRequest()
    
    def method(self, method: str) -> 'HTTPRequestBuilder':
        self._request.method = method.upper()
        return self
    
    def url(self, url: str) -> 'HTTPRequestBuilder':
        self._request.url = url
        return self
    
    def header(self, key: str, value: str) -> 'HTTPRequestBuilder':
        self._request.headers[key] = value
        return self
    
    def param(self, key: str, value: str) -> 'HTTPRequestBuilder':
        self._request.params[key] = value
        return self
    
    def body(self, data) -> 'HTTPRequestBuilder':
        self._request.body = data
        return self
    
    def timeout(self, seconds: int) -> 'HTTPRequestBuilder':
        self._request.timeout = seconds
        return self
    
    def bearer_auth(self, token: str) -> 'HTTPRequestBuilder':
        self._request.auth = f"Bearer {token}"
        return self
    
    def json_content(self) -> 'HTTPRequestBuilder':
        self._request.headers['Content-Type'] = 'application/json'
        return self
    
    def build(self) -> HTTPRequest:
        if not self._request.url:
            raise ValueError("URL is required")
        return self._request

# การใช้งาน
request = (HTTPRequestBuilder()
    .method('POST')
    .url('https://api.example.com/users')
    .header('Accept', 'application/json')
    .json_content()
    .bearer_auth('my-secret-token')
    .body({'name': 'Alice', 'email': 'alice@example.com'})
    .timeout(60)
    .build()
)

print(f"Method: {request.method}")
print(f"URL: {request.url}")
print(f"Headers: {request.headers}")
print(f"Body: {request.body}")
```

```python
# ตัวอย่าง 13: Builder สำหรับ Email
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class Email:
    to: List[str]
    subject: str
    body: str
    from_address: str = "noreply@example.com"
    cc: List[str] = field(default_factory=list)
    bcc: List[str] = field(default_factory=list)
    attachments: List[str] = field(default_factory=list)
    html: bool = False
    priority: str = "normal"

class EmailBuilder:
    def __init__(self):
        self._to = []
        self._subject = ""
        self._body = ""
        self._from = "noreply@example.com"
        self._cc = []
        self._bcc = []
        self._attachments = []
        self._html = False
        self._priority = "normal"
    
    def to(self, *addresses) -> 'EmailBuilder':
        self._to.extend(addresses)
        return self
    
    def subject(self, subject: str) -> 'EmailBuilder':
        self._subject = subject
        return self
    
    def body(self, body: str) -> 'EmailBuilder':
        self._body = body
        return self
    
    def from_address(self, address: str) -> 'EmailBuilder':
        self._from = address
        return self
    
    def cc(self, *addresses) -> 'EmailBuilder':
        self._cc.extend(addresses)
        return self
    
    def bcc(self, *addresses) -> 'EmailBuilder':
        self._bcc.extend(addresses)
        return self
    
    def attach(self, *files) -> 'EmailBuilder':
        self._attachments.extend(files)
        return self
    
    def as_html(self) -> 'EmailBuilder':
        self._html = True
        return self
    
    def high_priority(self) -> 'EmailBuilder':
        self._priority = "high"
        return self
    
    def build(self) -> Email:
        if not self._to:
            raise ValueError("At least one recipient is required")
        if not self._subject:
            raise ValueError("Subject is required")
        if not self._body:
            raise ValueError("Body is required")
        
        return Email(
            to=self._to,
            subject=self._subject,
            body=self._body,
            from_address=self._from,
            cc=self._cc,
            bcc=self._bcc,
            attachments=self._attachments,
            html=self._html,
            priority=self._priority
        )

# การใช้งาน
email = (EmailBuilder()
    .to("user@example.com", "admin@example.com")
    .subject("Welcome to our platform!")
    .from_address("welcome@myapp.com")
    .body("<h1>Welcome!</h1><p>Thanks for joining.</p>")
    .as_html()
    .cc("support@myapp.com")
    .attach("welcome_guide.pdf")
    .high_priority()
    .build()
)

print(f"To: {email.to}")
print(f"Subject: {email.subject}")
print(f"Priority: {email.priority}")
print(f"HTML: {email.html}")
```

---

### 2.5 Prototype Pattern

**ความหมาย**: ระบุชนิดของ object ที่จะสร้างโดยใช้ instance ต้นแบบ และสร้าง object ใหม่โดยการ copy ต้นแบบนั้น

```python
# ตัวอย่าง 14: Prototype Pattern
import copy
from typing import Dict, Any

class Prototype:
    def clone(self):
        return copy.deepcopy(self)

class Document(Prototype):
    def __init__(self, title: str, content: str, metadata: Dict[str, Any]):
        self.title = title
        self.content = content
        self.metadata = metadata
        self.version = 1
    
    def update_version(self):
        self.version += 1
    
    def __repr__(self):
        return f"Document(title='{self.title}', v{self.version})"

# การใช้งาน
original = Document(
    title="Project Report",
    content="This is the original content...",
    metadata={"author": "Alice", "tags": ["report", "2024"]}
)

# Clone และแก้ไข
draft = original.clone()
draft.title = "Project Report - Draft"
draft.content = "This is the draft content..."
draft.update_version()

print(f"Original: {original}")
print(f"Draft: {draft}")
print(f"Original metadata: {original.metadata}")
print(f"Draft metadata: {draft.metadata}")

# แก้ไข draft ไม่กระทบ original
draft.metadata['tags'].append('draft')
print(f"\nAfter modifying draft:")
print(f"Original tags: {original.metadata['tags']}")  # ไม่เปลี่ยน
print(f"Draft tags: {draft.metadata['tags']}")          # เปลี่ยน
```

```python
# ตัวอย่าง 15: Prototype Registry
import copy

class PrototypeRegistry:
    def __init__(self):
        self._prototypes = {}
    
    def register(self, name: str, prototype):
        self._prototypes[name] = prototype
    
    def clone(self, name: str):
        if name not in self._prototypes:
            raise ValueError(f"Prototype '{name}' not found")
        return copy.deepcopy(self._prototypes[name])

class Shape:
    def __init__(self, color: str, x: int = 0, y: int = 0):
        self.color = color
        self.x = x
        self.y = y
    
    def draw(self):
        return f"{self.__class__.__name__}(color={self.color}, pos=({self.x},{self.y}))"

class Circle(Shape):
    def __init__(self, color: str, radius: int, x: int = 0, y: int = 0):
        super().__init__(color, x, y)
        self.radius = radius
    
    def draw(self):
        return f"Circle(color={self.color}, r={self.radius}, pos=({self.x},{self.y}))"

class Rectangle(Shape):
    def __init__(self, color: str, width: int, height: int, x: int = 0, y: int = 0):
        super().__init__(color, x, y)
        self.width = width
        self.height = height
    
    def draw(self):
        return f"Rect(color={self.color}, {self.width}x{self.height}, pos=({self.x},{self.y}))"

# สร้าง Registry
registry = PrototypeRegistry()
registry.register("red_circle", Circle("red", 50))
registry.register("blue_rect", Rectangle("blue", 100, 50))

# Clone และแก้ไขตำแหน่ง
c1 = registry.clone("red_circle")
c1.x, c1.y = 10, 20

c2 = registry.clone("red_circle")
c2.x, c2.y = 100, 200

r1 = registry.clone("blue_rect")
r1.x, r1.y = 50, 50

print(c1.draw())
print(c2.draw())
print(r1.draw())
```

---

## 3. Structural Patterns

### 3.1 Adapter Pattern

**ความหมาย**: แปลง interface ของ class ให้เป็น interface อื่นที่ client คาดหวัง ทำให้ class ที่ incompatible ทำงานร่วมกันได้

```python
# ตัวอย่าง 16: Adapter Pattern
class LegacyPaymentSystem:
    """ระบบเดิมที่ไม่สามารถเปลี่ยนได้"""
    def make_payment(self, amount_cents: int, currency_code: str) -> dict:
        return {
            'status': 'SUCCESS',
            'amount': amount_cents,
            'currency': currency_code,
            'transaction_id': f"TXN{amount_cents}"
        }

class NewPaymentInterface:
    """Interface ใหม่ที่ระบบใหม่คาดหวัง"""
    def pay(self, amount: float, currency: str) -> bool:
        raise NotImplementedError

class PaymentAdapter(NewPaymentInterface):
    """Adapter เชื่อมต่อระบบเดิมกับ interface ใหม่"""
    def __init__(self, legacy_system: LegacyPaymentSystem):
        self.legacy_system = legacy_system
    
    def pay(self, amount: float, currency: str) -> bool:
        # แปลงจาก float เป็น cents
        amount_cents = int(amount * 100)
        result = self.legacy_system.make_payment(amount_cents, currency.upper())
        return result['status'] == 'SUCCESS'

# การใช้งาน
legacy = LegacyPaymentSystem()
adapter = PaymentAdapter(legacy)

# ใช้งานผ่าน interface ใหม่
success = adapter.pay(29.99, "usd")
print(f"Payment successful: {success}")
```

```python
# ตัวอย่าง 17: Adapter สำหรับ Third-party API
import json
from typing import List

# Third-party weather API (ไม่สามารถเปลี่ยนได้)
class OpenWeatherAPI:
    def get_weather_data(self, city: str) -> dict:
        # จำลองการตอบกลับจาก API
        return {
            'name': city,
            'main': {
                'temp': 298.15,  # Kelvin
                'humidity': 65
            },
            'weather': [{'description': 'clear sky'}],
            'wind': {'speed': 5.2}  # m/s
        }

# Interface ที่ application ของเราต้องการ
class WeatherService:
    def get_temperature(self, city: str) -> float:
        raise NotImplementedError
    
    def get_description(self, city: str) -> str:
        raise NotImplementedError

class OpenWeatherAdapter(WeatherService):
    def __init__(self):
        self._api = OpenWeatherAPI()
        self._cache = {}
    
    def _get_data(self, city: str) -> dict:
        if city not in self._cache:
            self._cache[city] = self._api.get_weather_data(city)
        return self._cache[city]
    
    def get_temperature(self, city: str) -> float:
        data = self._get_data(city)
        kelvin = data['main']['temp']
        return round(kelvin - 273.15, 2)  # แปลงเป็น Celsius
    
    def get_description(self, city: str) -> str:
        data = self._get_data(city)
        return data['weather'][0]['description'].title()
    
    def get_humidity(self, city: str) -> int:
        data = self._get_data(city)
        return data['main']['humidity']

# การใช้งาน
weather = OpenWeatherAdapter()
city = "Bangkok"
print(f"City: {city}")
print(f"Temperature: {weather.get_temperature(city)}°C")
print(f"Description: {weather.get_description(city)}")
print(f"Humidity: {weather.get_humidity(city)}%")
```

---

### 3.2 Decorator Pattern

**ความหมาย**: เพิ่ม responsibilities ให้ object แบบ dynamic โดยไม่เปลี่ยน class

```python
# ตัวอย่าง 18: Decorator Pattern สำหรับ Coffee Shop
from abc import ABC, abstractmethod

class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float:
        pass
    
    @abstractmethod
    def description(self) -> str:
        pass

class SimpleCoffee(Coffee):
    def cost(self) -> float:
        return 1.0
    
    def description(self) -> str:
        return "Simple coffee"

class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    def cost(self) -> float:
        return self._coffee.cost()
    
    def description(self) -> str:
        return self._coffee.description()

class Milk(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.5
    
    def description(self) -> str:
        return self._coffee.description() + ", milk"

class Sugar(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.25
    
    def description(self) -> str:
        return self._coffee.description() + ", sugar"

class Vanilla(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 1.0
    
    def description(self) -> str:
        return self._coffee.description() + ", vanilla"

class WhippedCream(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 1.5
    
    def description(self) -> str:
        return self._coffee.description() + ", whipped cream"

# การใช้งาน
# สั่ง coffee ธรรมดา
coffee = SimpleCoffee()
print(f"{coffee.description()}: ${coffee.cost()}")

# เพิ่ม milk
coffee_with_milk = Milk(coffee)
print(f"{coffee_with_milk.description()}: ${coffee_with_milk.cost()}")

# เพิ่ม milk + sugar + vanilla
fancy_coffee = Vanilla(Sugar(Milk(SimpleCoffee())))
print(f"{fancy_coffee.description()}: ${fancy_coffee.cost()}")

# Latte with everything
latte = WhippedCream(Vanilla(Sugar(Milk(Sugar(SimpleCoffee())))))
print(f"{latte.description()}: ${latte.cost()}")
```

```python
# ตัวอย่าง 19: Decorator ด้วย Python Decorator Syntax
import time
import functools
import logging

# Function Decorators ใน Python
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

def retry(max_attempts=3, delay=1.0):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}. Retrying...")
                    time.sleep(delay)
        return wrapper
    return decorator

def cache(func):
    cached_results = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cached_results:
            cached_results[args] = func(*args)
        return cached_results[args]
    return wrapper

def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

# การใช้งาน
@timer
@cache
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

@retry(max_attempts=3, delay=0.1)
def fetch_data(url: str) -> dict:
    import random
    if random.random() < 0.7:  # 70% chance of failure
        raise ConnectionError("Network error")
    return {"data": "success"}

@log_calls
def add(a: int, b: int) -> int:
    return a + b

# ทดสอบ
print(fibonacci(10))
add(3, 4)
```

---

### 3.3 Facade Pattern

**ความหมาย**: ให้ simplified interface ต่อ subsystem ที่ซับซ้อน

```python
# ตัวอย่าง 20: Facade สำหรับระบบ Home Theater
class Amplifier:
    def on(self): print("Amp on")
    def off(self): print("Amp off")
    def set_volume(self, volume): print(f"Setting volume to {volume}")
    def set_input(self, source): print(f"Setting input to {source}")

class DVDPlayer:
    def on(self): print("DVD on")
    def off(self): print("DVD off")
    def play(self, movie): print(f"Playing: {movie}")
    def stop(self): print("DVD stopped")
    def eject(self): print("DVD ejected")

class Projector:
    def on(self): print("Projector on")
    def off(self): print("Projector off")
    def wide_screen_mode(self): print("Wide screen mode")
    def tv_mode(self): print("TV mode")

class TheaterLights:
    def on(self): print("Lights on")
    def off(self): print("Lights off")
    def dim(self, level): print(f"Dimming lights to {level}%")

class Screen:
    def down(self): print("Screen down")
    def up(self): print("Screen up")

# Facade
class HomeTheaterFacade:
    def __init__(self):
        self.amp = Amplifier()
        self.dvd = DVDPlayer()
        self.projector = Projector()
        self.lights = TheaterLights()
        self.screen = Screen()
    
    def watch_movie(self, movie: str):
        print(f"\n=== Preparing to watch '{movie}' ===")
        self.lights.dim(10)
        self.screen.down()
        self.projector.on()
        self.projector.wide_screen_mode()
        self.amp.on()
        self.amp.set_input("DVD")
        self.amp.set_volume(5)
        self.dvd.on()
        self.dvd.play(movie)
        print("=== Enjoy your movie! ===\n")
    
    def end_movie(self):
        print("\n=== Shutting down home theater ===")
        self.dvd.stop()
        self.dvd.eject()
        self.dvd.off()
        self.amp.off()
        self.projector.off()
        self.screen.up()
        self.lights.on()
        print("=== Goodbye! ===\n")

# การใช้งาน - ง่ายมากผ่าน Facade
theater = HomeTheaterFacade()
theater.watch_movie("Inception")
theater.end_movie()
```

```python
# ตัวอย่าง 21: Facade สำหรับ Order Processing
class InventoryService:
    def check_stock(self, product_id: str, quantity: int) -> bool:
        print(f"Checking stock for {product_id}: {quantity} units")
        return True
    
    def reserve_items(self, product_id: str, quantity: int):
        print(f"Reserving {quantity} units of {product_id}")
    
    def release_items(self, product_id: str, quantity: int):
        print(f"Releasing {quantity} units of {product_id}")

class PaymentService:
    def charge(self, user_id: str, amount: float) -> str:
        print(f"Charging ${amount} to user {user_id}")
        return f"TXN_{user_id}_{int(amount)}"
    
    def refund(self, transaction_id: str):
        print(f"Refunding transaction {transaction_id}")

class ShippingService:
    def calculate_shipping(self, address: str) -> float:
        print(f"Calculating shipping to {address}")
        return 5.99
    
    def create_shipment(self, order_id: str, address: str) -> str:
        print(f"Creating shipment for order {order_id} to {address}")
        return f"SHIP_{order_id}"
    
    def track_shipment(self, tracking_id: str) -> str:
        return f"In transit: {tracking_id}"

class NotificationService:
    def send_email(self, email: str, message: str):
        print(f"Sending email to {email}: {message}")
    
    def send_sms(self, phone: str, message: str):
        print(f"Sending SMS to {phone}: {message}")

class OrderFacade:
    def __init__(self):
        self.inventory = InventoryService()
        self.payment = PaymentService()
        self.shipping = ShippingService()
        self.notification = NotificationService()
    
    def place_order(self, user_id: str, product_id: str, 
                    quantity: int, amount: float, 
                    address: str, email: str) -> dict:
        print(f"\n=== Processing Order ===")
        
        # 1. Check stock
        if not self.inventory.check_stock(product_id, quantity):
            return {"success": False, "error": "Out of stock"}
        
        # 2. Reserve items
        self.inventory.reserve_items(product_id, quantity)
        
        # 3. Process payment
        shipping_cost = self.shipping.calculate_shipping(address)
        total = amount + shipping_cost
        transaction_id = self.payment.charge(user_id, total)
        
        # 4. Create shipment
        order_id = f"ORD_{user_id}_{product_id}"
        tracking_id = self.shipping.create_shipment(order_id, address)
        
        # 5. Notify customer
        self.notification.send_email(
            email, 
            f"Your order {order_id} has been placed! Tracking: {tracking_id}"
        )
        
        print("=== Order Complete ===\n")
        return {
            "success": True,
            "order_id": order_id,
            "transaction_id": transaction_id,
            "tracking_id": tracking_id
        }

# การใช้งาน
order_system = OrderFacade()
result = order_system.place_order(
    user_id="USER001",
    product_id="PROD123",
    quantity=2,
    amount=99.99,
    address="123 Main St, Bangkok",
    email="customer@example.com"
)
print(f"Order result: {result}")
```

---

### 3.4 Proxy Pattern

**ความหมาย**: ให้ surrogate หรือ placeholder สำหรับ object อื่น เพื่อควบคุม access ไปยัง object นั้น

```python
# ตัวอย่าง 22: Proxy Pattern - Virtual Proxy (Lazy Loading)
class RealImage:
    def __init__(self, filename: str):
        self.filename = filename
        self._load()
    
    def _load(self):
        print(f"Loading image from disk: {self.filename}")
        # จำลองการโหลดรูปภาพ (ใช้เวลา)
        self.data = f"<image data of {self.filename}>"
    
    def display(self):
        print(f"Displaying: {self.filename}")
        return self.data

class ProxyImage:
    """Lazy loading - โหลด image เฉพาะเมื่อจำเป็น"""
    def __init__(self, filename: str):
        self.filename = filename
        self._real_image = None
    
    def display(self):
        if self._real_image is None:
            print(f"Creating real image for: {self.filename}")
            self._real_image = RealImage(self.filename)
        return self._real_image.display()

# การใช้งาน
print("Creating proxy images (no loading yet)...")
images = [
    ProxyImage("photo1.jpg"),
    ProxyImage("photo2.jpg"),
    ProxyImage("photo3.jpg")
]

print("\nDisplaying first image (loads now):")
images[0].display()

print("\nDisplaying first image again (cached):")
images[0].display()

print("\nDisplaying second image (loads now):")
images[1].display()
```

```python
# ตัวอย่าง 23: Protection Proxy - Access Control
class SensitiveDocument:
    def __init__(self, content: str):
        self._content = content
    
    def read(self) -> str:
        return self._content
    
    def write(self, content: str):
        self._content = content
    
    def delete(self):
        self._content = ""

class DocumentProxy:
    """Protection Proxy - ควบคุม permission"""
    def __init__(self, document: SensitiveDocument, user_role: str):
        self._document = document
        self._user_role = user_role
    
    def read(self) -> str:
        if self._user_role in ['admin', 'editor', 'viewer']:
            return self._document.read()
        raise PermissionError(f"Role '{self._user_role}' cannot read")
    
    def write(self, content: str):
        if self._user_role in ['admin', 'editor']:
            self._document.write(content)
        else:
            raise PermissionError(f"Role '{self._user_role}' cannot write")
    
    def delete(self):
        if self._user_role == 'admin':
            self._document.delete()
        else:
            raise PermissionError(f"Role '{self._user_role}' cannot delete")

# การใช้งาน
doc = SensitiveDocument("Top Secret Content")

admin = DocumentProxy(doc, 'admin')
editor = DocumentProxy(doc, 'editor')
viewer = DocumentProxy(doc, 'viewer')

print(f"Admin reads: {admin.read()}")
admin.write("Updated content")
print(f"Editor reads: {editor.read()}")
editor.write("Editor's update")

try:
    viewer.write("Viewer trying to write...")
except PermissionError as e:
    print(f"Access denied: {e}")

try:
    editor.delete()
except PermissionError as e:
    print(f"Access denied: {e}")
```

---

### 3.5 Composite Pattern

**ความหมาย**: จัด objects ให้เป็น tree structures เพื่อแทน part-whole hierarchies

```python
# ตัวอย่าง 24: Composite Pattern - File System
from abc import ABC, abstractmethod
from typing import List, Optional

class FileSystemComponent(ABC):
    def __init__(self, name: str):
        self.name = name
    
    @abstractmethod
    def size(self) -> int:
        pass
    
    @abstractmethod
    def display(self, indent: int = 0):
        pass

class File(FileSystemComponent):
    def __init__(self, name: str, size: int):
        super().__init__(name)
        self._size = size
    
    def size(self) -> int:
        return self._size
    
    def display(self, indent: int = 0):
        print(f"{'  ' * indent}📄 {self.name} ({self._size} KB)")

class Directory(FileSystemComponent):
    def __init__(self, name: str):
        super().__init__(name)
        self._children: List[FileSystemComponent] = []
    
    def add(self, component: FileSystemComponent):
        self._children.append(component)
    
    def remove(self, component: FileSystemComponent):
        self._children.remove(component)
    
    def size(self) -> int:
        return sum(child.size() for child in self._children)
    
    def display(self, indent: int = 0):
        print(f"{'  ' * indent}📁 {self.name}/ ({self.size()} KB)")
        for child in self._children:
            child.display(indent + 1)

# การใช้งาน
root = Directory("root")

src = Directory("src")
src.add(File("main.py", 5))
src.add(File("utils.py", 3))
src.add(File("models.py", 8))

tests = Directory("tests")
tests.add(File("test_main.py", 4))
tests.add(File("test_utils.py", 2))

docs = Directory("docs")
docs.add(File("README.md", 10))
docs.add(File("API.md", 7))

root.add(src)
root.add(tests)
root.add(docs)
root.add(File("requirements.txt", 1))

root.display()
print(f"\nTotal size: {root.size()} KB")
```

---

## 4. Behavioral Patterns

### 4.1 Observer Pattern

**ความหมาย**: กำหนด one-to-many dependency ระหว่าง objects เพื่อให้เมื่อ object หนึ่งเปลี่ยนแปลง ทุก object ที่ depend อยู่จะถูก notify และ update โดยอัตโนมัติ

```python
# ตัวอย่าง 25: Observer Pattern - Event System
from typing import Callable, Dict, List, Any
from dataclasses import dataclass

@dataclass
class Event:
    name: str
    data: Any = None

class EventBus:
    """Simple Event Bus implementation"""
    def __init__(self):
        self._subscribers: Dict[str, List[Callable]] = {}
    
    def subscribe(self, event_name: str, callback: Callable):
        if event_name not in self._subscribers:
            self._subscribers[event_name] = []
        self._subscribers[event_name].append(callback)
    
    def unsubscribe(self, event_name: str, callback: Callable):
        if event_name in self._subscribers:
            self._subscribers[event_name].remove(callback)
    
    def publish(self, event: Event):
        if event.name in self._subscribers:
            for callback in self._subscribers[event.name]:
                callback(event)

# Observers
def send_welcome_email(event: Event):
    print(f"📧 Sending welcome email to: {event.data['email']}")

def update_analytics(event: Event):
    print(f"📊 Analytics: New user registered - {event.data['username']}")

def log_event(event: Event):
    print(f"📋 Log: {event.name} - {event.data}")

def grant_initial_permissions(event: Event):
    print(f"🔐 Granting initial permissions to: {event.data['username']}")

# การใช้งาน
bus = EventBus()

# Subscribe
bus.subscribe('user.registered', send_welcome_email)
bus.subscribe('user.registered', update_analytics)
bus.subscribe('user.registered', grant_initial_permissions)
bus.subscribe('user.registered', log_event)

# Trigger event
user_data = {'username': 'alice', 'email': 'alice@example.com'}
bus.publish(Event('user.registered', user_data))
```

```python
# ตัวอย่าง 26: Observer Pattern - Stock Market
from abc import ABC, abstractmethod
from typing import List
from dataclasses import dataclass

@dataclass
class StockPrice:
    symbol: str
    price: float
    change: float

class StockObserver(ABC):
    @abstractmethod
    def update(self, stock: StockPrice):
        pass

class StockMarket:
    def __init__(self):
        self._observers: List[StockObserver] = []
        self._stocks = {}
    
    def subscribe(self, observer: StockObserver):
        self._observers.append(observer)
    
    def unsubscribe(self, observer: StockObserver):
        self._observers.remove(observer)
    
    def _notify(self, stock: StockPrice):
        for observer in self._observers:
            observer.update(stock)
    
    def update_price(self, symbol: str, price: float):
        old_price = self._stocks.get(symbol, price)
        change = ((price - old_price) / old_price) * 100 if old_price else 0
        self._stocks[symbol] = price
        
        stock = StockPrice(symbol, price, round(change, 2))
        self._notify(stock)

class PhoneAlert(StockObserver):
    def __init__(self, watchlist: List[str]):
        self.watchlist = watchlist
    
    def update(self, stock: StockPrice):
        if stock.symbol in self.watchlist:
            direction = "📈" if stock.change >= 0 else "📉"
            print(f"📱 Alert: {stock.symbol} {direction} ${stock.price} ({stock.change:+.2f}%)")

class PortfolioTracker(StockObserver):
    def __init__(self, holdings: dict):
        self.holdings = holdings  # {symbol: shares}
    
    def update(self, stock: StockPrice):
        if stock.symbol in self.holdings:
            shares = self.holdings[stock.symbol]
            value = shares * stock.price
            print(f"💰 Portfolio: {stock.symbol} x{shares} = ${value:.2f}")

class TradingBot(StockObserver):
    def __init__(self, buy_threshold: float, sell_threshold: float):
        self.buy_threshold = buy_threshold
        self.sell_threshold = sell_threshold
    
    def update(self, stock: StockPrice):
        if stock.change <= self.buy_threshold:
            print(f"🤖 Bot: BUY signal for {stock.symbol} (down {stock.change:.2f}%)")
        elif stock.change >= self.sell_threshold:
            print(f"🤖 Bot: SELL signal for {stock.symbol} (up {stock.change:.2f}%)")

# การใช้งาน
market = StockMarket()

phone = PhoneAlert(['AAPL', 'GOOGL'])
portfolio = PortfolioTracker({'AAPL': 10, 'MSFT': 5})
bot = TradingBot(buy_threshold=-3.0, sell_threshold=5.0)

market.subscribe(phone)
market.subscribe(portfolio)
market.subscribe(bot)

# จำลองการเปลี่ยนแปลงราคา
print("=== Market Update ===")
market.update_price('AAPL', 150.00)
market.update_price('AAPL', 145.00)   # -3.33% → bot should buy
market.update_price('GOOGL', 2800.00)
market.update_price('MSFT', 380.00)
market.update_price('MSFT', 400.00)   # +5.26% → bot should sell
```

---

### 4.2 Strategy Pattern

**ความหมาย**: กำหนด family ของ algorithms, encapsulate แต่ละอัน และทำให้สับเปลี่ยนกันได้

```python
# ตัวอย่าง 27: Strategy Pattern - Sorting
from abc import ABC, abstractmethod
from typing import List

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: List[int]) -> List[int]:
        pass

class BubbleSort(SortStrategy):
    def sort(self, data: List[int]) -> List[int]:
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr

class QuickSort(SortStrategy):
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        mid = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + mid + self.sort(right)

class MergeSort(SortStrategy):
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
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
        return result + left[i:] + right[j:]

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def sort(self, data: List[int]) -> List[int]:
        return self._strategy.sort(data)

# การใช้งาน
data = [64, 34, 25, 12, 22, 11, 90]
sorter = Sorter(BubbleSort())

print(f"Original: {data}")
print(f"Bubble sort: {sorter.sort(data)}")

sorter.set_strategy(QuickSort())
print(f"Quick sort: {sorter.sort(data)}")

sorter.set_strategy(MergeSort())
print(f"Merge sort: {sorter.sort(data)}")
```

```python
# ตัวอย่าง 28: Strategy Pattern - Discount System
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List

@dataclass
class Product:
    name: str
    price: float

@dataclass
class CartItem:
    product: Product
    quantity: int
    
    @property
    def subtotal(self) -> float:
        return self.product.price * self.quantity

class DiscountStrategy(ABC):
    @abstractmethod
    def apply(self, cart_items: List[CartItem], total: float) -> float:
        pass
    
    @abstractmethod
    def description(self) -> str:
        pass

class NoDiscount(DiscountStrategy):
    def apply(self, cart_items: List[CartItem], total: float) -> float:
        return total
    
    def description(self) -> str:
        return "No discount"

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percent: float):
        self.percent = percent
    
    def apply(self, cart_items: List[CartItem], total: float) -> float:
        return total * (1 - self.percent / 100)
    
    def description(self) -> str:
        return f"{self.percent}% off"

class FixedAmountDiscount(DiscountStrategy):
    def __init__(self, amount: float):
        self.amount = amount
    
    def apply(self, cart_items: List[CartItem], total: float) -> float:
        return max(0, total - self.amount)
    
    def description(self) -> str:
        return f"${self.amount} off"

class BuyXGetYFree(DiscountStrategy):
    def __init__(self, buy_x: int, get_y: int, product_name: str):
        self.buy_x = buy_x
        self.get_y = get_y
        self.product_name = product_name
    
    def apply(self, cart_items: List[CartItem], total: float) -> float:
        for item in cart_items:
            if item.product.name == self.product_name:
                free_items = (item.quantity // (self.buy_x + self.get_y)) * self.get_y
                total -= free_items * item.product.price
        return total
    
    def description(self) -> str:
        return f"Buy {self.buy_x} get {self.get_y} free on {self.product_name}"

class ShoppingCart:
    def __init__(self):
        self._items: List[CartItem] = []
        self._discount: DiscountStrategy = NoDiscount()
    
    def add_item(self, product: Product, quantity: int):
        self._items.append(CartItem(product, quantity))
    
    def set_discount(self, strategy: DiscountStrategy):
        self._discount = strategy
    
    def total(self) -> float:
        raw_total = sum(item.subtotal for item in self._items)
        return self._discount.apply(self._items, raw_total)
    
    def summary(self):
        print("\n=== Cart Summary ===")
        raw_total = 0
        for item in self._items:
            print(f"  {item.product.name} x{item.quantity}: ${item.subtotal:.2f}")
            raw_total += item.subtotal
        print(f"Subtotal: ${raw_total:.2f}")
        print(f"Discount: {self._discount.description()}")
        print(f"Total: ${self.total():.2f}")

# การใช้งาน
cart = ShoppingCart()
cart.add_item(Product("Coffee", 15.00), 3)
cart.add_item(Product("Muffin", 5.00), 4)

# ไม่มีส่วนลด
cart.summary()

# ส่วนลด 20%
cart.set_discount(PercentageDiscount(20))
cart.summary()

# ลดราคา $10
cart.set_discount(FixedAmountDiscount(10))
cart.summary()

# Buy 3 get 1 free Coffee
cart.set_discount(BuyXGetYFree(3, 1, "Coffee"))
cart.summary()
```

---

### 4.3 Command Pattern

**ความหมาย**: Encapsulate a request as an object ทำให้สามารถ queue, log, หรือ undo requests ได้

```python
# ตัวอย่าง 29: Command Pattern - Text Editor with Undo/Redo
from abc import ABC, abstractmethod
from typing import List

class Command(ABC):
    @abstractmethod
    def execute(self) -> str:
        pass
    
    @abstractmethod
    def undo(self) -> str:
        pass

class TextEditor:
    def __init__(self):
        self._text = ""
    
    def get_text(self) -> str:
        return self._text
    
    def set_text(self, text: str):
        self._text = text

class TypeCommand(Command):
    def __init__(self, editor: TextEditor, text: str):
        self._editor = editor
        self._text = text
        self._previous = ""
    
    def execute(self) -> str:
        self._previous = self._editor.get_text()
        self._editor.set_text(self._previous + self._text)
        return f"Typed: '{self._text}'"
    
    def undo(self) -> str:
        self._editor.set_text(self._previous)
        return f"Undone typing: '{self._text}'"

class DeleteCommand(Command):
    def __init__(self, editor: TextEditor, n_chars: int):
        self._editor = editor
        self._n = n_chars
        self._deleted = ""
    
    def execute(self) -> str:
        text = self._editor.get_text()
        self._deleted = text[-self._n:]
        self._editor.set_text(text[:-self._n])
        return f"Deleted: '{self._deleted}'"
    
    def undo(self) -> str:
        current = self._editor.get_text()
        self._editor.set_text(current + self._deleted)
        return f"Restored: '{self._deleted}'"

class CommandHistory:
    def __init__(self):
        self._history: List[Command] = []
        self._redo_stack: List[Command] = []
    
    def execute(self, command: Command):
        result = command.execute()
        self._history.append(command)
        self._redo_stack.clear()
        return result
    
    def undo(self):
        if not self._history:
            return "Nothing to undo"
        command = self._history.pop()
        result = command.undo()
        self._redo_stack.append(command)
        return result
    
    def redo(self):
        if not self._redo_stack:
            return "Nothing to redo"
        command = self._redo_stack.pop()
        result = command.execute()
        self._history.append(command)
        return result

# การใช้งาน
editor = TextEditor()
history = CommandHistory()

def show_state():
    print(f"  Text: '{editor.get_text()}'")

print("=== Text Editor with Undo/Redo ===")
history.execute(TypeCommand(editor, "Hello"))
show_state()

history.execute(TypeCommand(editor, " World"))
show_state()

history.execute(TypeCommand(editor, "!"))
show_state()

history.execute(DeleteCommand(editor, 1))
show_state()

print("\nUndo:")
history.undo()
show_state()

history.undo()
show_state()

print("\nRedo:")
history.redo()
show_state()
```

---

### 4.4 Iterator Pattern

```python
# ตัวอย่าง 30: Iterator Pattern
from typing import Iterator, Any

class TreeNode:
    def __init__(self, value: int):
        self.value = value
        self.left = None
        self.right = None

class BinaryTree:
    def __init__(self):
        self.root = None
    
    def insert(self, value: int):
        self.root = self._insert(self.root, value)
    
    def _insert(self, node, value):
        if node is None:
            return TreeNode(value)
        if value < node.value:
            node.left = self._insert(node.left, value)
        else:
            node.right = self._insert(node.right, value)
        return node
    
    def __iter__(self) -> Iterator[int]:
        return InOrderIterator(self.root)
    
    def breadth_first(self):
        return BreadthFirstIterator(self.root)

class InOrderIterator:
    def __init__(self, root):
        self._stack = []
        self._push_left(root)
    
    def _push_left(self, node):
        while node:
            self._stack.append(node)
            node = node.left
    
    def __iter__(self):
        return self
    
    def __next__(self) -> int:
        if not self._stack:
            raise StopIteration
        node = self._stack.pop()
        value = node.value
        self._push_left(node.right)
        return value

class BreadthFirstIterator:
    def __init__(self, root):
        from collections import deque
        self._queue = deque([root] if root else [])
    
    def __iter__(self):
        return self
    
    def __next__(self) -> int:
        if not self._queue:
            raise StopIteration
        node = self._queue.popleft()
        if node.left:
            self._queue.append(node.left)
        if node.right:
            self._queue.append(node.right)
        return node.value

# การใช้งาน
tree = BinaryTree()
for val in [5, 3, 7, 1, 4, 6, 8]:
    tree.insert(val)

print("In-order traversal:", list(tree))
print("Breadth-first traversal:", list(tree.breadth_first()))
```

---

### 4.5 Template Method Pattern

```python
# ตัวอย่าง 31: Template Method Pattern
from abc import ABC, abstractmethod

class DataMiner(ABC):
    """Template Method กำหนด algorithm skeleton"""
    
    def mine(self, path: str):
        """Template method"""
        raw_data = self.extract_data(path)
        parsed_data = self.parse_data(raw_data)
        analysis = self.analyze_data(parsed_data)
        self.send_report(analysis)
    
    @abstractmethod
    def extract_data(self, path: str) -> str:
        pass
    
    @abstractmethod
    def parse_data(self, raw_data: str) -> list:
        pass
    
    def analyze_data(self, data: list) -> dict:
        """Default implementation - can be overridden"""
        return {
            'count': len(data),
            'sample': data[:5] if data else []
        }
    
    def send_report(self, analysis: dict):
        """Default implementation"""
        print(f"Report: {analysis}")

class CSVDataMiner(DataMiner):
    def extract_data(self, path: str) -> str:
        print(f"Reading CSV from: {path}")
        return "name,age,city\nAlice,30,BKK\nBob,25,NYC\nCharlie,35,LON"
    
    def parse_data(self, raw_data: str) -> list:
        lines = raw_data.strip().split('\n')
        headers = lines[0].split(',')
        return [
            dict(zip(headers, line.split(',')))
            for line in lines[1:]
        ]

class JSONDataMiner(DataMiner):
    def extract_data(self, path: str) -> str:
        print(f"Reading JSON from: {path}")
        return '[{"name":"Alice","age":30},{"name":"Bob","age":25}]'
    
    def parse_data(self, raw_data: str) -> list:
        import json
        return json.loads(raw_data)
    
    def analyze_data(self, data: list) -> dict:
        """Override analysis for JSON"""
        ages = [item.get('age', 0) for item in data]
        return {
            'count': len(data),
            'avg_age': sum(ages) / len(ages) if ages else 0,
            'names': [item.get('name') for item in data]
        }

# การใช้งาน
print("=== CSV Mining ===")
csv_miner = CSVDataMiner()
csv_miner.mine("data.csv")

print("\n=== JSON Mining ===")
json_miner = JSONDataMiner()
json_miner.mine("data.json")
```

---

### 4.6 State Pattern

```python
# ตัวอย่าง 32: State Pattern - Order System
from abc import ABC, abstractmethod

class OrderState(ABC):
    @abstractmethod
    def next(self, order: 'Order'):
        pass
    
    @abstractmethod
    def cancel(self, order: 'Order'):
        pass
    
    def __str__(self):
        return self.__class__.__name__

class PendingState(OrderState):
    def next(self, order: 'Order'):
        print("Payment received. Processing order...")
        order.state = ProcessingState()
    
    def cancel(self, order: 'Order'):
        print("Order cancelled before payment.")
        order.state = CancelledState()

class ProcessingState(OrderState):
    def next(self, order: 'Order'):
        print("Order packed and shipped!")
        order.state = ShippedState()
    
    def cancel(self, order: 'Order'):
        print("Cancelling and refunding payment...")
        order.state = CancelledState()

class ShippedState(OrderState):
    def next(self, order: 'Order'):
        print("Package delivered!")
        order.state = DeliveredState()
    
    def cancel(self, order: 'Order'):
        print("Cannot cancel - already shipped. Please return.")

class DeliveredState(OrderState):
    def next(self, order: 'Order'):
        print("Order already delivered.")
    
    def cancel(self, order: 'Order'):
        print("Order delivered. Initiate return process.")

class CancelledState(OrderState):
    def next(self, order: 'Order'):
        print("Order is cancelled. Cannot proceed.")
    
    def cancel(self, order: 'Order'):
        print("Order already cancelled.")

class Order:
    def __init__(self, order_id: str):
        self.order_id = order_id
        self.state = PendingState()
    
    def next(self):
        print(f"[{self.order_id}] State: {self.state}")
        self.state.next(self)
    
    def cancel(self):
        print(f"[{self.order_id}] Cancelling from state: {self.state}")
        self.state.cancel(self)
    
    def status(self):
        print(f"[{self.order_id}] Current state: {self.state}")

# การใช้งาน
print("=== Normal Order Flow ===")
order = Order("ORD-001")
order.status()
order.next()   # Pending -> Processing
order.next()   # Processing -> Shipped
order.next()   # Shipped -> Delivered
order.next()   # Already delivered

print("\n=== Cancelled Order ===")
order2 = Order("ORD-002")
order2.next()      # Pending -> Processing
order2.cancel()    # Cancel while processing
order2.next()      # Cannot proceed - cancelled
```

---

## 5. Python-Specific Patterns

### 5.1 Context Manager Pattern

```python
# ตัวอย่าง 33: Context Manager
from contextlib import contextmanager

class DatabaseTransaction:
    def __init__(self, db_name: str):
        self.db_name = db_name
        self.queries = []
    
    def __enter__(self):
        print(f"BEGIN TRANSACTION on {self.db_name}")
        return self
    
    def execute(self, query: str):
        self.queries.append(query)
        print(f"  Queued: {query}")
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            print(f"COMMIT: {len(self.queries)} queries")
            return True
        else:
            print(f"ROLLBACK due to {exc_type.__name__}: {exc_val}")
            return False

# การใช้งาน
print("=== Successful Transaction ===")
with DatabaseTransaction("mydb") as db:
    db.execute("INSERT INTO users VALUES ...")
    db.execute("UPDATE accounts SET balance = ...")

print("\n=== Failed Transaction ===")
try:
    with DatabaseTransaction("mydb") as db:
        db.execute("INSERT INTO orders VALUES ...")
        raise ValueError("Payment failed!")
        db.execute("UPDATE inventory SET ...")
except ValueError as e:
    print(f"Transaction failed: {e}")

@contextmanager
def timer_context(name: str):
    import time
    print(f"Starting: {name}")
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"Finished: {name} ({elapsed:.4f}s)")

with timer_context("data processing"):
    # จำลองงาน
    total = sum(range(1000000))
```

### 5.2 Descriptor Pattern

```python
# ตัวอย่าง 34: Descriptor Pattern
class Validated:
    """Generic validator descriptor"""
    def __set_name__(self, owner, name):
        self.name = name
        self.storage_name = f"_{name}"
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.storage_name, None)
    
    def __set__(self, instance, value):
        self.validate(value)
        setattr(instance, self.storage_name, value)
    
    def validate(self, value):
        pass

class PositiveNumber(Validated):
    def validate(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} must be a number")
        if value <= 0:
            raise ValueError(f"{self.name} must be positive, got {value}")

class NonEmptyString(Validated):
    def validate(self, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.name} must be a string")
        if not value.strip():
            raise ValueError(f"{self.name} cannot be empty")

class Product:
    name = NonEmptyString()
    price = PositiveNumber()
    quantity = PositiveNumber()
    
    def __init__(self, name: str, price: float, quantity: int):
        self.name = name
        self.price = price
        self.quantity = quantity
    
    def total_value(self) -> float:
        return self.price * self.quantity

# การใช้งาน
p = Product("Laptop", 999.99, 10)
print(f"{p.name}: ${p.price} x {p.quantity} = ${p.total_value()}")

try:
    p2 = Product("", 100, 5)
except ValueError as e:
    print(f"Error: {e}")

try:
    p3 = Product("Phone", -50, 3)
except ValueError as e:
    print(f"Error: {e}")
```

---

## 6. Anti-Patterns to Avoid

```python
# ตัวอย่าง 35: Anti-Pattern - God Object
# BAD: God Object - รู้และทำทุกอย่าง
class GodObject:
    """Anti-pattern: Class ที่รู้และทำทุกอย่าง"""
    def __init__(self):
        self.users = []
        self.products = []
        self.orders = []
        self.db_connection = None
        self.email_service = None
    
    def add_user(self, username, email, password):
        # hash password, validate email, save to db...
        pass
    
    def process_payment(self, user_id, amount):
        # validate user, check balance, process...
        pass
    
    def send_invoice(self, order_id, email):
        # generate PDF, send email...
        pass
    
    def generate_report(self, start_date, end_date):
        # query db, calculate statistics...
        pass

# GOOD: แยก responsibilities ออก
class UserRepository:
    def create(self, username: str, email: str) -> 'User':
        pass

class PaymentService:
    def process(self, user_id: str, amount: float) -> str:
        pass

class InvoiceService:
    def generate_and_send(self, order_id: str) -> bool:
        pass

class ReportService:
    def generate_sales_report(self, start_date, end_date) -> dict:
        pass
```

```python
# ตัวอย่าง 36: Anti-Pattern - Spaghetti Code vs Clean Code
# BAD: Spaghetti code
def process_order_bad(data):
    if data.get('user_id'):
        u = get_user(data['user_id'])
        if u and u.get('active'):
            if data.get('items'):
                t = 0
                for i in data['items']:
                    p = get_product(i['id'])
                    if p and p['stock'] >= i['qty']:
                        t += p['price'] * i['qty']
                    else:
                        return {'error': 'out_of_stock'}
                if t > 0:
                    if charge_card(u['card'], t):
                        for i in data['items']:
                            update_stock(i['id'], -i['qty'])
                        send_email(u['email'], f'Order confirmed: ${t}')
                        return {'success': True, 'total': t}
    return {'error': 'invalid'}

# GOOD: Clean code with proper structure
def validate_user(user_id: str) -> dict:
    user = get_user(user_id)
    if not user or not user.get('active'):
        raise ValueError(f"Invalid or inactive user: {user_id}")
    return user

def calculate_order_total(items: list) -> float:
    total = 0
    for item in items:
        product = get_product(item['id'])
        if not product or product['stock'] < item['qty']:
            raise ValueError(f"Product {item['id']} out of stock")
        total += product['price'] * item['qty']
    return total

def process_payment(user: dict, amount: float) -> bool:
    if not charge_card(user['card'], amount):
        raise PaymentError("Payment failed")
    return True

def update_inventory(items: list):
    for item in items:
        update_stock(item['id'], -item['qty'])

def process_order_good(data: dict) -> dict:
    try:
        user = validate_user(data.get('user_id', ''))
        total = calculate_order_total(data.get('items', []))
        process_payment(user, total)
        update_inventory(data['items'])
        send_email(user['email'], f'Order confirmed: ${total}')
        return {'success': True, 'total': total}
    except (ValueError, PaymentError) as e:
        return {'error': str(e)}
```

```python
# ตัวอย่าง 37: Anti-Pattern - Magic Numbers
# BAD: Magic Numbers
def calculate_shipping_bad(weight: float) -> float:
    if weight <= 5:
        return 50
    elif weight <= 20:
        return 50 + (weight - 5) * 3
    else:
        return 50 + 15 * 3 + (weight - 20) * 2

# GOOD: Named constants
BASE_SHIPPING_COST = 50.0
LIGHT_WEIGHT_LIMIT = 5.0     # kg
MEDIUM_WEIGHT_LIMIT = 20.0   # kg
LIGHT_RATE_PER_KG = 3.0      # THB per kg
HEAVY_RATE_PER_KG = 2.0      # THB per kg

def calculate_shipping_good(weight: float) -> float:
    if weight <= LIGHT_WEIGHT_LIMIT:
        return BASE_SHIPPING_COST
    elif weight <= MEDIUM_WEIGHT_LIMIT:
        extra_weight = weight - LIGHT_WEIGHT_LIMIT
        return BASE_SHIPPING_COST + extra_weight * LIGHT_RATE_PER_KG
    else:
        medium_portion = MEDIUM_WEIGHT_LIMIT - LIGHT_WEIGHT_LIMIT
        heavy_portion = weight - MEDIUM_WEIGHT_LIMIT
        return (BASE_SHIPPING_COST + 
                medium_portion * LIGHT_RATE_PER_KG + 
                heavy_portion * HEAVY_RATE_PER_KG)
```

```python
# ตัวอย่าง 38: Anti-Pattern - Deep Nesting
# BAD: Arrow code (deep nesting)
def process_data_bad(data):
    if data:
        if isinstance(data, dict):
            if 'users' in data:
                for user in data['users']:
                    if user.get('active'):
                        if user.get('email'):
                            if '@' in user['email']:
                                # actual logic here
                                print(f"Processing: {user['email']}")

# GOOD: Early return (guard clauses)
def process_data_good(data):
    if not data:
        return
    if not isinstance(data, dict):
        return
    if 'users' not in data:
        return
    
    for user in data['users']:
        if not user.get('active'):
            continue
        if not user.get('email'):
            continue
        if '@' not in user['email']:
            continue
        
        # actual logic here
        print(f"Processing: {user['email']}")
```

```python
# ตัวอย่าง 39: Anti-Pattern - Mutable Default Arguments
# BAD: Mutable default argument
def add_item_bad(item, items=[]):
    items.append(item)
    return items

print(add_item_bad("apple"))   # ['apple']
print(add_item_bad("banana"))  # ['apple', 'banana'] - BUG!
print(add_item_bad("cherry"))  # ['apple', 'banana', 'cherry'] - BUG!

# GOOD: Use None as default
def add_item_good(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item_good("apple"))   # ['apple']
print(add_item_good("banana"))  # ['banana']
print(add_item_good("cherry"))  # ['cherry']
```

```python
# ตัวอย่าง 40: Anti-Pattern - Exception Swallowing
# BAD: Catching all exceptions and doing nothing
def get_user_bad(user_id: str):
    try:
        return database.find_user(user_id)
    except:
        pass  # Silent failure - terrible!

# GOOD: Handle specific exceptions appropriately
def get_user_good(user_id: str):
    try:
        return database.find_user(user_id)
    except DatabaseConnectionError as e:
        logging.error(f"Database unavailable: {e}")
        raise ServiceUnavailableError("Cannot connect to database") from e
    except UserNotFoundError:
        return None  # Expected case - user might not exist
    except Exception as e:
        logging.critical(f"Unexpected error fetching user {user_id}: {e}")
        raise
```

```python
# ตัวอย่าง 41: Anti-Pattern - Premature Optimization
# BAD: Over-engineered for no reason
class PrematurelyOptimized:
    def __init__(self):
        self._cache = {}
        self._lock = threading.Lock()
        self._pool = ThreadPoolExecutor(max_workers=10)
        # ... complex setup for a simple list
    
    def get_names(self):
        with self._lock:
            # Complex thread-safe operation for a simple list
            pass

# GOOD: Simple first, optimize when needed
class Simple:
    def __init__(self):
        self._names = []
    
    def get_names(self):
        return self._names.copy()

# "Make it work, make it right, make it fast" - Kent Beck
```

```python
# ตัวอย่าง 42: Python's Borg Pattern (variant of Singleton)
class Borg:
    """Borg pattern - shared state instead of shared identity"""
    _shared_state = {}
    
    def __init__(self):
        self.__dict__ = self._shared_state

class BorgLogger(Borg):
    def __init__(self):
        super().__init__()
        if not hasattr(self, 'messages'):
            self.messages = []
    
    def log(self, message: str):
        self.messages.append(message)

b1 = BorgLogger()
b2 = BorgLogger()

b1.log("First message")
b2.log("Second message")

print(f"b1 messages: {b1.messages}")
print(f"b2 messages: {b2.messages}")
print(f"Same object? {b1 is b2}")        # False - different objects
print(f"Same state? {b1.__dict__ is b2.__dict__}")  # True - shared state
```

---

## 7. รูปแบบเพิ่มเติมที่ใช้บ่อยใน Python

```python
# ตัวอย่าง 43: Null Object Pattern
class NullUser:
    """แทนที่การ return None"""
    @property
    def name(self): return "Guest"
    
    @property
    def email(self): return ""
    
    @property
    def is_authenticated(self): return False
    
    def can_edit(self, resource): return False
    
    def __bool__(self): return False

class UserRepository:
    def find(self, user_id: str):
        # Simulate database lookup
        if user_id == "admin":
            return type('User', (), {
                'name': 'Admin',
                'email': 'admin@example.com',
                'is_authenticated': True,
                'can_edit': lambda r: True
            })()
        return NullUser()

# การใช้งาน - ไม่ต้อง check None ทุกที่
repo = UserRepository()

user = repo.find("admin")
print(f"User: {user.name}, Auth: {user.is_authenticated}")

guest = repo.find("unknown")
print(f"User: {guest.name}, Auth: {guest.is_authenticated}")

# ใช้ใน conditional
if guest:
    print("Authenticated user")
else:
    print("Unauthenticated - showing guest content")
```

```python
# ตัวอย่าง 44: Monostate Pattern
class Settings:
    """Monostate - instances share all state"""
    _state = {
        'debug': False,
        'log_level': 'INFO',
        'max_retries': 3,
    }
    
    def __getattr__(self, key):
        if key.startswith('_'):
            raise AttributeError(key)
        return self._state.get(key)
    
    def __setattr__(self, key, value):
        if key.startswith('_'):
            super().__setattr__(key, value)
        else:
            self._state[key] = value

s1 = Settings()
s2 = Settings()

s1.debug = True
s1.log_level = 'DEBUG'

print(f"s1.debug: {s1.debug}")
print(f"s2.debug: {s2.debug}")    # True - shared state
print(f"Same object? {s1 is s2}") # False
```

```python
# ตัวอย่าง 45: Registry Pattern
class PluginRegistry:
    """Registry สำหรับ plugins"""
    _registry = {}
    
    @classmethod
    def register(cls, name: str):
        def decorator(plugin_class):
            cls._registry[name] = plugin_class
            return plugin_class
        return decorator
    
    @classmethod
    def get(cls, name: str):
        if name not in cls._registry:
            raise KeyError(f"Plugin '{name}' not found")
        return cls._registry[name]
    
    @classmethod
    def list_plugins(cls):
        return list(cls._registry.keys())

@PluginRegistry.register('markdown')
class MarkdownPlugin:
    def render(self, text: str) -> str:
        return f"<p>{text}</p>"

@PluginRegistry.register('rst')
class RSTPlugin:
    def render(self, text: str) -> str:
        return f".. note:: {text}"

@PluginRegistry.register('html')
class HTMLPlugin:
    def render(self, text: str) -> str:
        return text

# การใช้งาน
print("Available plugins:", PluginRegistry.list_plugins())

for plugin_name in PluginRegistry.list_plugins():
    Plugin = PluginRegistry.get(plugin_name)
    plugin = Plugin()
    print(f"{plugin_name}: {plugin.render('Hello World')}")
```

---

## 8. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Singleton Logger
สร้าง `Logger` class แบบ Singleton ที่:
- มี method `log(level, message)` โดย level คือ DEBUG, INFO, WARNING, ERROR
- เก็บ logs ใน list
- มี method `get_logs(level=None)` ที่ filter ตาม level ได้
- Thread-safe

```python
# เฉลยแบบฝึกหัดที่ 1
import threading
from enum import Enum
from datetime import datetime
from typing import List, Optional

class LogLevel(Enum):
    DEBUG = 0
    INFO = 1
    WARNING = 2
    ERROR = 3

class Logger:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._logs = []
                    cls._instance._logs_lock = threading.Lock()
        return cls._instance
    
    def log(self, level: LogLevel, message: str):
        timestamp = datetime.now().isoformat()
        entry = {
            'timestamp': timestamp,
            'level': level,
            'message': message
        }
        with self._logs_lock:
            self._logs.append(entry)
        print(f"[{timestamp}] {level.name}: {message}")
    
    def get_logs(self, level: Optional[LogLevel] = None) -> List[dict]:
        with self._logs_lock:
            if level is None:
                return self._logs.copy()
            return [log for log in self._logs if log['level'] == level]
    
    def debug(self, message: str): self.log(LogLevel.DEBUG, message)
    def info(self, message: str): self.log(LogLevel.INFO, message)
    def warning(self, message: str): self.log(LogLevel.WARNING, message)
    def error(self, message: str): self.log(LogLevel.ERROR, message)

# ทดสอบ
logger = Logger()
logger.info("Application started")
logger.debug("Loading configuration")
logger.warning("Memory usage high")
logger.error("Connection failed")

errors = logger.get_logs(LogLevel.ERROR)
print(f"\nErrors: {len(errors)}")

# Singleton check
logger2 = Logger()
print(f"Same instance: {logger is logger2}")
```

### แบบฝึกหัดที่ 2: Plugin System
สร้าง plugin system ที่:
- มี abstract `Transformer` class ที่มี method `transform(text: str) -> str`
- สร้าง plugins: `UpperCase`, `LowerCase`, `Reverse`, `TitleCase`, `SnakeCase`
- มี `TransformPipeline` ที่ apply หลาย transformers ตามลำดับ

```python
# เฉลยแบบฝึกหัดที่ 2
from abc import ABC, abstractmethod
from typing import List
import re

class Transformer(ABC):
    @abstractmethod
    def transform(self, text: str) -> str:
        pass
    
    def __repr__(self):
        return self.__class__.__name__

class UpperCase(Transformer):
    def transform(self, text: str) -> str:
        return text.upper()

class LowerCase(Transformer):
    def transform(self, text: str) -> str:
        return text.lower()

class Reverse(Transformer):
    def transform(self, text: str) -> str:
        return text[::-1]

class TitleCase(Transformer):
    def transform(self, text: str) -> str:
        return text.title()

class SnakeCase(Transformer):
    def transform(self, text: str) -> str:
        s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', text)
        result = re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()
        return result.replace(' ', '_')

class TrimSpaces(Transformer):
    def transform(self, text: str) -> str:
        return ' '.join(text.split())

class TransformPipeline:
    def __init__(self):
        self._transformers: List[Transformer] = []
    
    def add(self, transformer: Transformer) -> 'TransformPipeline':
        self._transformers.append(transformer)
        return self
    
    def transform(self, text: str) -> str:
        result = text
        for transformer in self._transformers:
            result = transformer.transform(result)
        return result
    
    def __repr__(self):
        steps = " -> ".join(str(t) for t in self._transformers)
        return f"Pipeline({steps})"

# ทดสอบ
text = "  Hello World from Python  "
pipeline = (TransformPipeline()
    .add(TrimSpaces())
    .add(SnakeCase())
)

print(f"Original: '{text}'")
print(f"Pipeline: {pipeline}")
print(f"Result: '{pipeline.transform(text)}'")

# Pipeline 2
pipeline2 = (TransformPipeline()
    .add(TrimSpaces())
    .add(TitleCase())
    .add(Reverse())
)
print(f"\nPipeline2: {pipeline2}")
print(f"Result: '{pipeline2.transform(text)}'")
```

### แบบฝึกหัดที่ 3: Observer สำหรับ Chat System
```python
# เฉลยแบบฝึกหัดที่ 3
from abc import ABC, abstractmethod
from typing import List, Dict
from datetime import datetime
from dataclasses import dataclass

@dataclass
class Message:
    sender: str
    content: str
    timestamp: str = None
    
    def __post_init__(self):
        if self.timestamp is None:
            self.timestamp = datetime.now().strftime("%H:%M:%S")

class MessageObserver(ABC):
    @abstractmethod
    def on_message(self, room: str, message: Message):
        pass

class ChatRoom:
    def __init__(self, name: str):
        self.name = name
        self._observers: List[MessageObserver] = []
        self._history: List[Message] = []
    
    def join(self, observer: MessageObserver):
        self._observers.append(observer)
    
    def leave(self, observer: MessageObserver):
        if observer in self._observers:
            self._observers.remove(observer)
    
    def send(self, sender: str, content: str):
        message = Message(sender, content)
        self._history.append(message)
        for observer in self._observers:
            observer.on_message(self.name, message)
    
    def get_history(self) -> List[Message]:
        return self._history.copy()

class ChatClient(MessageObserver):
    def __init__(self, username: str):
        self.username = username
        self._messages: List[tuple] = []
    
    def on_message(self, room: str, message: Message):
        if message.sender != self.username:
            entry = (room, message)
            self._messages.append(entry)
            print(f"[{message.timestamp}] [{room}] {message.sender}: {message.content}")
    
    def get_unread(self) -> List[tuple]:
        return self._messages.copy()

class NotificationBot(MessageObserver):
    def __init__(self, keywords: List[str]):
        self.keywords = [kw.lower() for kw in keywords]
    
    def on_message(self, room: str, message: Message):
        content_lower = message.content.lower()
        if any(kw in content_lower for kw in self.keywords):
            print(f"🔔 ALERT: Keyword detected in {room}: '{message.content}'")

# ทดสอบ
room = ChatRoom("general")

alice = ChatClient("alice")
bob = ChatClient("bob")
charlie = ChatClient("charlie")
bot = NotificationBot(["urgent", "help", "error"])

room.join(alice)
room.join(bob)
room.join(charlie)
room.join(bot)

print("=== Chat Session ===")
room.send("alice", "Hello everyone!")
room.send("bob", "Hi Alice!")
room.send("charlie", "urgent: server is down!")
room.send("alice", "I need help with this error")

print(f"\nAlice's unread: {len(alice.get_unread())} messages")
print(f"Bob's unread: {len(bob.get_unread())} messages")
```

### แบบฝึกหัดที่ 4: Builder สำหรับ Report Generation
```python
# เฉลยแบบฝึกหัดที่ 4
from dataclasses import dataclass, field
from typing import List, Dict, Optional
from datetime import datetime

@dataclass
class ReportSection:
    title: str
    content: str
    level: int = 1

@dataclass
class Report:
    title: str
    author: str
    sections: List[ReportSection]
    created_at: str
    tags: List[str] = field(default_factory=list)
    footer: Optional[str] = None
    page_numbers: bool = True
    
    def to_markdown(self) -> str:
        lines = []
        lines.append(f"# {self.title}")
        lines.append(f"**Author**: {self.author}")
        lines.append(f"**Date**: {self.created_at}")
        if self.tags:
            lines.append(f"**Tags**: {', '.join(self.tags)}")
        lines.append("")
        
        for section in self.sections:
            prefix = "#" * (section.level + 1)
            lines.append(f"{prefix} {section.title}")
            lines.append(section.content)
            lines.append("")
        
        if self.footer:
            lines.append("---")
            lines.append(self.footer)
        
        return "\n".join(lines)

class ReportBuilder:
    def __init__(self, title: str, author: str):
        self._title = title
        self._author = author
        self._sections = []
        self._tags = []
        self._footer = None
        self._page_numbers = True
        self._created_at = datetime.now().strftime("%Y-%m-%d")
    
    def add_section(self, title: str, content: str, level: int = 1) -> 'ReportBuilder':
        self._sections.append(ReportSection(title, content, level))
        return self
    
    def tag(self, *tags) -> 'ReportBuilder':
        self._tags.extend(tags)
        return self
    
    def footer(self, text: str) -> 'ReportBuilder':
        self._footer = text
        return self
    
    def no_page_numbers(self) -> 'ReportBuilder':
        self._page_numbers = False
        return self
    
    def build(self) -> Report:
        if not self._sections:
            raise ValueError("Report must have at least one section")
        return Report(
            title=self._title,
            author=self._author,
            sections=self._sections,
            created_at=self._created_at,
            tags=self._tags,
            footer=self._footer,
            page_numbers=self._page_numbers
        )

# ทดสอบ
report = (ReportBuilder("Q4 Sales Report", "Alice Smith")
    .tag("sales", "quarterly", "2024")
    .add_section(
        "Executive Summary",
        "Q4 showed 15% growth compared to Q3, driven by strong performance in the enterprise segment.",
        level=1
    )
    .add_section(
        "Revenue",
        "Total revenue: $2.5M\n- Enterprise: $1.5M (60%)\n- SMB: $0.7M (28%)\n- Consumer: $0.3M (12%)",
        level=2
    )
    .add_section(
        "Key Metrics",
        "- New customers: 45\n- Churn rate: 3.2%\n- NPS: 72",
        level=2
    )
    .footer("Confidential - For Internal Use Only")
    .build()
)

print(report.to_markdown())
```

### แบบฝึกหัดที่ 5: Strategy สำหรับ Export System
```python
# เฉลยแบบฝึกหัดที่ 5
from abc import ABC, abstractmethod
from typing import List, Dict
import json

class ExportStrategy(ABC):
    @abstractmethod
    def export(self, data: List[Dict], filename: str) -> str:
        pass
    
    @property
    @abstractmethod
    def extension(self) -> str:
        pass

class CSVExport(ExportStrategy):
    @property
    def extension(self) -> str:
        return 'csv'
    
    def export(self, data: List[Dict], filename: str) -> str:
        if not data:
            return ""
        headers = list(data[0].keys())
        lines = [','.join(headers)]
        for row in data:
            values = [str(row.get(h, '')) for h in headers]
            lines.append(','.join(values))
        content = '\n'.join(lines)
        full_name = f"{filename}.{self.extension}"
        print(f"Exported to {full_name}:\n{content[:200]}...")
        return full_name

class JSONExport(ExportStrategy):
    @property
    def extension(self) -> str:
        return 'json'
    
    def export(self, data: List[Dict], filename: str) -> str:
        content = json.dumps(data, indent=2, ensure_ascii=False)
        full_name = f"{filename}.{self.extension}"
        print(f"Exported to {full_name}:\n{content[:200]}...")
        return full_name

class HTMLExport(ExportStrategy):
    @property
    def extension(self) -> str:
        return 'html'
    
    def export(self, data: List[Dict], filename: str) -> str:
        if not data:
            return ""
        headers = list(data[0].keys())
        rows = ""
        for row in data:
            cells = "".join(f"<td>{row.get(h, '')}</td>" for h in headers)
            rows += f"<tr>{cells}</tr>\n"
        header_row = "".join(f"<th>{h}</th>" for h in headers)
        content = f"""<table>
  <thead><tr>{header_row}</tr></thead>
  <tbody>{rows}</tbody>
</table>"""
        full_name = f"{filename}.{self.extension}"
        print(f"Exported to {full_name}:\n{content[:200]}...")
        return full_name

class DataExporter:
    def __init__(self, strategy: ExportStrategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy: ExportStrategy):
        self._strategy = strategy
    
    def export(self, data: List[Dict], filename: str) -> str:
        return self._strategy.export(data, filename)

# ทดสอบ
data = [
    {'id': 1, 'name': 'Alice', 'dept': 'Engineering', 'salary': 90000},
    {'id': 2, 'name': 'Bob', 'dept': 'Marketing', 'salary': 75000},
    {'id': 3, 'name': 'Charlie', 'dept': 'Engineering', 'salary': 95000},
]

exporter = DataExporter(CSVExport())
exporter.export(data, "employees")

exporter.set_strategy(JSONExport())
exporter.export(data, "employees")

exporter.set_strategy(HTMLExport())
exporter.export(data, "employees")
```

### แบบฝึกหัดที่ 6 - 10 (Exercises)

**แบบฝึกหัดที่ 6**: สร้าง Command Pattern สำหรับ Smart Home (เปิด/ปิด lights, thermostat, TV) พร้อม undo/redo

**แบบฝึกหัดที่ 7**: สร้าง Composite Pattern สำหรับ Menu System (menu, submenu, menu items) คำนวณ total items และ display tree

**แบบฝึกหัดที่ 8**: สร้าง Proxy Pattern สำหรับ Caching Database (cache query results, expire after timeout)

**แบบฝึกหัดที่ 9**: สร้าง Template Method สำหรับ Data Validation Pipeline (validate type → validate range → validate format → save)

**แบบฝึกหัดที่ 10**: สร้าง Abstract Factory สำหรับ Notification System (Email, SMS, Push Notification) ที่ support ทั้ง Production และ Testing environments

---

## สรุป

| Pattern | หมวด | ใช้เมื่อ |
|---------|------|---------|
| Singleton | Creational | ต้องการ single instance |
| Factory Method | Creational | ให้ subclass ตัดสินใจสร้าง object |
| Abstract Factory | Creational | สร้าง family of objects |
| Builder | Creational | สร้าง complex objects ทีละขั้น |
| Prototype | Creational | Clone objects |
| Adapter | Structural | เชื่อมต่อ incompatible interfaces |
| Decorator | Structural | เพิ่ม behavior แบบ dynamic |
| Facade | Structural | ลด complexity ของ subsystem |
| Proxy | Structural | ควบคุม access ต่อ object |
| Composite | Structural | Tree structures |
| Observer | Behavioral | Event notification |
| Strategy | Behavioral | Interchangeable algorithms |
| Command | Behavioral | Encapsulate requests |
| Iterator | Behavioral | Sequential access |
| Template Method | Behavioral | Algorithm skeleton |
| State | Behavioral | Behavior changes with state |

---

*Part 86 เสร็จสมบูรณ์ | ต่อไป: Part 87 - SOLID Principles & Clean Code*
