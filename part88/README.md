# Part 88: Clean Architecture & Domain-Driven Design

## บทนำ

Clean Architecture (โดย Robert C. Martin) และ Domain-Driven Design (DDD โดย Eric Evans) คือ 2 แนวคิดสำคัญที่ช่วยให้เราสร้างซอฟต์แวร์ที่มีโครงสร้างชัดเจน บำรุงรักษาง่าย และสอดคล้องกับ business domain

---

## 1. Clean Architecture

### 1.1 หลักการสำคัญ

Clean Architecture จัดระเบียบโค้ดเป็น layers แบบ concentric circles โดย:
- **Dependencies ชี้เข้าด้านใน** (Dependency Rule)
- Inner layers ไม่รู้จัก outer layers
- Business logic อยู่ตรงกลาง ไม่ขึ้นกับ frameworks หรือ databases

```
┌─────────────────────────────────────────────┐
│         Frameworks & Drivers                 │
│   ┌─────────────────────────────────────┐   │
│   │      Interface Adapters              │   │
│   │  ┌───────────────────────────────┐  │   │
│   │  │       Use Cases               │  │   │
│   │  │  ┌─────────────────────────┐  │  │   │
│   │  │  │       Entities          │  │  │   │
│   │  │  │   (Business Rules)      │  │  │   │
│   │  │  └─────────────────────────┘  │  │   │
│   │  └───────────────────────────────┘  │   │
│   └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### 1.2 Layers

| Layer | ชื่อ | หน้าที่ |
|-------|------|--------|
| 1 (กลาง) | **Entities** | Enterprise business rules |
| 2 | **Use Cases** | Application business rules |
| 3 | **Interface Adapters** | Controllers, Presenters, Gateways |
| 4 (นอกสุด) | **Frameworks & Drivers** | Web, DB, UI |

---

## 2. ตัวอย่าง Clean Architecture: User Management System

### 2.1 Layer 1: Entities (Enterprise Business Rules)

```python
# entities/user.py
# ตัวอย่าง 1: Entity
from dataclasses import dataclass, field
from typing import Optional
from datetime import datetime
import re

class UserValidationError(Exception):
    pass

@dataclass
class Email:
    """Value Object สำหรับ Email"""
    value: str
    
    def __post_init__(self):
        self._validate()
    
    def _validate(self):
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, self.value):
            raise UserValidationError(f"Invalid email: {self.value}")
    
    def __str__(self) -> str:
        return self.value
    
    def __eq__(self, other) -> bool:
        if isinstance(other, Email):
            return self.value.lower() == other.value.lower()
        return False

@dataclass
class UserName:
    """Value Object สำหรับ Username"""
    value: str
    
    def __post_init__(self):
        self._validate()
    
    def _validate(self):
        if len(self.value) < 3:
            raise UserValidationError("Username must be at least 3 characters")
        if len(self.value) > 50:
            raise UserValidationError("Username must not exceed 50 characters")
        if not re.match(r'^[a-zA-Z0-9_-]+$', self.value):
            raise UserValidationError("Username can only contain letters, numbers, _ and -")
    
    def __str__(self) -> str:
        return self.value

@dataclass
class UserId:
    """Value Object สำหรับ User ID"""
    value: str
    
    @classmethod
    def generate(cls) -> 'UserId':
        import uuid
        return cls(value=str(uuid.uuid4()))
    
    def __str__(self) -> str:
        return self.value

class User:
    """Entity - มี Identity"""
    
    def __init__(
        self,
        id: UserId,
        username: UserName,
        email: Email,
        created_at: datetime = None,
        is_active: bool = True
    ):
        self._id = id
        self._username = username
        self._email = email
        self._created_at = created_at or datetime.now()
        self._is_active = is_active
        self._domain_events = []
    
    @classmethod
    def create(cls, username: str, email: str) -> 'User':
        """Factory method"""
        user = cls(
            id=UserId.generate(),
            username=UserName(username),
            email=Email(email)
        )
        user._domain_events.append(UserCreatedEvent(user._id, user._email))
        return user
    
    @property
    def id(self) -> UserId:
        return self._id
    
    @property
    def username(self) -> UserName:
        return self._username
    
    @property
    def email(self) -> Email:
        return self._email
    
    @property
    def is_active(self) -> bool:
        return self._is_active
    
    @property
    def created_at(self) -> datetime:
        return self._created_at
    
    def change_email(self, new_email: str) -> None:
        old_email = self._email
        self._email = Email(new_email)
        self._domain_events.append(EmailChangedEvent(self._id, old_email, self._email))
    
    def deactivate(self) -> None:
        if not self._is_active:
            raise UserValidationError("User is already inactive")
        self._is_active = False
        self._domain_events.append(UserDeactivatedEvent(self._id))
    
    def activate(self) -> None:
        if self._is_active:
            raise UserValidationError("User is already active")
        self._is_active = True
    
    def pop_events(self) -> list:
        events = self._domain_events.copy()
        self._domain_events.clear()
        return events
    
    def __eq__(self, other) -> bool:
        if isinstance(other, User):
            return self._id == other._id
        return False
    
    def __repr__(self) -> str:
        return f"User(id={self._id}, username={self._username}, active={self._is_active})"

# Domain Events
@dataclass
class UserCreatedEvent:
    user_id: UserId
    email: Email
    occurred_at: datetime = field(default_factory=datetime.now)

@dataclass
class EmailChangedEvent:
    user_id: UserId
    old_email: Email
    new_email: Email
    occurred_at: datetime = field(default_factory=datetime.now)

@dataclass
class UserDeactivatedEvent:
    user_id: UserId
    occurred_at: datetime = field(default_factory=datetime.now)

# ทดสอบ Entities
try:
    user = User.create("alice_smith", "alice@example.com")
    print(f"Created: {user}")
    
    user.change_email("alice.new@example.com")
    print(f"New email: {user.email}")
    
    events = user.pop_events()
    print(f"Events: {[type(e).__name__ for e in events]}")
    
    # Validation
    bad_user = User.create("ab", "not-an-email")
except UserValidationError as e:
    print(f"Validation error: {e}")
```

### 2.2 Layer 2: Use Cases (Application Business Rules)

```python
# use_cases/create_user.py
# ตัวอย่าง 2: Use Cases
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional

# Input/Output Data Transfer Objects
@dataclass
class CreateUserInput:
    username: str
    email: str
    password: str

@dataclass
class CreateUserOutput:
    user_id: str
    username: str
    email: str
    created_at: str

@dataclass
class GetUserInput:
    user_id: str

@dataclass
class GetUserOutput:
    user_id: str
    username: str
    email: str
    is_active: bool
    created_at: str

# Repository Interface (Port)
class UserRepository(ABC):
    @abstractmethod
    def save(self, user: User) -> None:
        pass
    
    @abstractmethod
    def find_by_id(self, user_id: UserId) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_by_email(self, email: Email) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_by_username(self, username: UserName) -> Optional[User]:
        pass

# Password Hasher Interface (Port)
class PasswordHasher(ABC):
    @abstractmethod
    def hash(self, password: str) -> str:
        pass
    
    @abstractmethod
    def verify(self, password: str, hashed: str) -> bool:
        pass

# Event Publisher Interface (Port)
class EventPublisher(ABC):
    @abstractmethod
    def publish(self, event) -> None:
        pass

# Use Case
class CreateUserUseCase:
    """Application business rules สำหรับการสร้าง user"""
    
    def __init__(
        self,
        user_repository: UserRepository,
        password_hasher: PasswordHasher,
        event_publisher: EventPublisher
    ):
        self._repo = user_repository
        self._hasher = password_hasher
        self._publisher = event_publisher
    
    def execute(self, input_data: CreateUserInput) -> CreateUserOutput:
        # Check duplicate
        existing = self._repo.find_by_email(Email(input_data.email))
        if existing:
            raise ValueError(f"Email {input_data.email} is already registered")
        
        existing_username = self._repo.find_by_username(UserName(input_data.username))
        if existing_username:
            raise ValueError(f"Username {input_data.username} is already taken")
        
        # Create user (entity)
        user = User.create(input_data.username, input_data.email)
        
        # Hash password (application concern)
        password_hash = self._hasher.hash(input_data.password)
        
        # Save
        self._repo.save(user)
        
        # Publish domain events
        for event in user.pop_events():
            self._publisher.publish(event)
        
        return CreateUserOutput(
            user_id=str(user.id),
            username=str(user.username),
            email=str(user.email),
            created_at=user.created_at.isoformat()
        )

class GetUserUseCase:
    def __init__(self, user_repository: UserRepository):
        self._repo = user_repository
    
    def execute(self, input_data: GetUserInput) -> Optional[GetUserOutput]:
        user = self._repo.find_by_id(UserId(input_data.user_id))
        if not user:
            return None
        return GetUserOutput(
            user_id=str(user.id),
            username=str(user.username),
            email=str(user.email),
            is_active=user.is_active,
            created_at=user.created_at.isoformat()
        )

class DeactivateUserUseCase:
    def __init__(self, user_repository: UserRepository, event_publisher: EventPublisher):
        self._repo = user_repository
        self._publisher = event_publisher
    
    def execute(self, user_id: str) -> bool:
        user = self._repo.find_by_id(UserId(user_id))
        if not user:
            raise ValueError(f"User {user_id} not found")
        
        user.deactivate()
        self._repo.save(user)
        
        for event in user.pop_events():
            self._publisher.publish(event)
        
        return True
```

### 2.3 Layer 3: Interface Adapters

```python
# ตัวอย่าง 3: Interface Adapters - Implementations

# Infrastructure implementations
import hashlib
from typing import Dict, Optional

class InMemoryUserRepository(UserRepository):
    """Adapter สำหรับ in-memory storage (testing)"""
    
    def __init__(self):
        self._users: Dict[str, User] = {}
        self._email_index: Dict[str, str] = {}
        self._username_index: Dict[str, str] = {}
    
    def save(self, user: User) -> None:
        user_id = str(user.id)
        self._users[user_id] = user
        self._email_index[str(user.email).lower()] = user_id
        self._username_index[str(user.username).lower()] = user_id
    
    def find_by_id(self, user_id: UserId) -> Optional[User]:
        return self._users.get(str(user_id))
    
    def find_by_email(self, email: Email) -> Optional[User]:
        user_id = self._email_index.get(str(email).lower())
        return self._users.get(user_id) if user_id else None
    
    def find_by_username(self, username: UserName) -> Optional[User]:
        user_id = self._username_index.get(str(username).lower())
        return self._users.get(user_id) if user_id else None
    
    def count(self) -> int:
        return len(self._users)

class SHA256PasswordHasher(PasswordHasher):
    def hash(self, password: str) -> str:
        return hashlib.sha256(password.encode()).hexdigest()
    
    def verify(self, password: str, hashed: str) -> bool:
        return self.hash(password) == hashed

class ConsoleEventPublisher(EventPublisher):
    def publish(self, event) -> None:
        print(f"[EVENT] {type(event).__name__}: {event}")

# Controller (Interface Adapter)
class UserController:
    """แปลง HTTP request เป็น use case input"""
    
    def __init__(
        self,
        create_user_use_case: CreateUserUseCase,
        get_user_use_case: GetUserUseCase,
        deactivate_user_use_case: DeactivateUserUseCase
    ):
        self._create = create_user_use_case
        self._get = get_user_use_case
        self._deactivate = deactivate_user_use_case
    
    def create_user(self, request: dict) -> dict:
        try:
            input_data = CreateUserInput(
                username=request.get('username', ''),
                email=request.get('email', ''),
                password=request.get('password', '')
            )
            output = self._create.execute(input_data)
            return {
                'status': 201,
                'data': {
                    'user_id': output.user_id,
                    'username': output.username,
                    'email': output.email,
                    'created_at': output.created_at
                }
            }
        except (UserValidationError, ValueError) as e:
            return {'status': 400, 'error': str(e)}
    
    def get_user(self, user_id: str) -> dict:
        output = self._get.execute(GetUserInput(user_id))
        if not output:
            return {'status': 404, 'error': 'User not found'}
        return {'status': 200, 'data': vars(output)}
    
    def deactivate_user(self, user_id: str) -> dict:
        try:
            self._deactivate.execute(user_id)
            return {'status': 200, 'message': 'User deactivated'}
        except ValueError as e:
            return {'status': 404, 'error': str(e)}

# สร้างและรัน
repo = InMemoryUserRepository()
hasher = SHA256PasswordHasher()
publisher = ConsoleEventPublisher()

create_use_case = CreateUserUseCase(repo, hasher, publisher)
get_use_case = GetUserUseCase(repo)
deactivate_use_case = DeactivateUserUseCase(repo, publisher)

controller = UserController(create_use_case, get_use_case, deactivate_use_case)

# ทดสอบ
print("=== Create User ===")
response = controller.create_user({
    'username': 'alice_smith',
    'email': 'alice@example.com',
    'password': 'SecurePass123!'
})
print(f"Response: {response}")

user_id = response['data']['user_id']

print("\n=== Get User ===")
user_response = controller.get_user(user_id)
print(f"Response: {user_response}")

print("\n=== Duplicate Email ===")
dup_response = controller.create_user({
    'username': 'alice2',
    'email': 'alice@example.com',  # duplicate
    'password': 'Pass123!'
})
print(f"Response: {dup_response}")
```

---

## 3. Domain-Driven Design (DDD)

### 3.1 Core Concepts

```python
# ตัวอย่าง 4: Value Objects
from dataclasses import dataclass
from typing import Optional
import re

@dataclass(frozen=True)  # Immutable
class Money:
    """Value Object - ไม่มี identity แต่มี value"""
    amount: float
    currency: str
    
    def __post_init__(self):
        if self.amount < 0:
            raise ValueError(f"Money amount cannot be negative: {self.amount}")
        if not self.currency or len(self.currency) != 3:
            raise ValueError(f"Invalid currency code: {self.currency}")
        # ทำ immutable post-init
        object.__setattr__(self, 'currency', self.currency.upper())
    
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError(f"Cannot add {self.currency} and {other.currency}")
        return Money(self.amount + other.amount, self.currency)
    
    def subtract(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError(f"Cannot subtract different currencies")
        result = self.amount - other.amount
        if result < 0:
            raise ValueError("Insufficient funds")
        return Money(result, self.currency)
    
    def multiply(self, factor: float) -> 'Money':
        return Money(self.amount * factor, self.currency)
    
    def __lt__(self, other: 'Money') -> bool:
        if self.currency != other.currency:
            raise ValueError("Cannot compare different currencies")
        return self.amount < other.amount
    
    def __str__(self) -> str:
        return f"{self.currency} {self.amount:.2f}"

@dataclass(frozen=True)
class Address:
    street: str
    city: str
    postal_code: str
    country: str
    
    def __post_init__(self):
        if not self.postal_code:
            raise ValueError("Postal code is required")
        if not self.country or len(self.country) < 2:
            raise ValueError("Invalid country")
    
    def formatted(self) -> str:
        return f"{self.street}, {self.city} {self.postal_code}, {self.country}"

@dataclass(frozen=True)
class PhoneNumber:
    value: str
    
    def __post_init__(self):
        # ลบ spaces และ dashes
        cleaned = re.sub(r'[\s\-\(\)]', '', self.value)
        object.__setattr__(self, 'value', cleaned)
        
        if not re.match(r'^\+?[0-9]{7,15}$', cleaned):
            raise ValueError(f"Invalid phone number: {self.value}")

# ทดสอบ Value Objects
price = Money(99.99, "THB")
tax = price.multiply(0.07)
total = price.add(tax)
print(f"Price: {price}")
print(f"Tax: {tax}")
print(f"Total: {total}")

address = Address("123 Main St", "Bangkok", "10100", "TH")
print(f"Address: {address.formatted()}")

phone = PhoneNumber("+66 81-234-5678")
print(f"Phone: {phone.value}")
```

### 3.2 Aggregates

```python
# ตัวอย่าง 5: Aggregate Root
from dataclasses import dataclass, field
from typing import List, Optional
from enum import Enum
from datetime import datetime

class OrderStatus(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

@dataclass(frozen=True)
class OrderId:
    value: str
    
    @classmethod
    def generate(cls) -> 'OrderId':
        import uuid
        return cls(value=f"ORD-{str(uuid.uuid4())[:8].upper()}")

@dataclass(frozen=True)
class ProductId:
    value: str

@dataclass(frozen=True)
class CustomerId:
    value: str

@dataclass
class OrderLine:
    """Value Object ภายใน Aggregate"""
    product_id: ProductId
    product_name: str
    unit_price: Money
    quantity: int
    
    def __post_init__(self):
        if self.quantity <= 0:
            raise ValueError("Quantity must be positive")
        if self.unit_price.amount <= 0:
            raise ValueError("Unit price must be positive")
    
    @property
    def subtotal(self) -> Money:
        return self.unit_price.multiply(self.quantity)

class Order:
    """Aggregate Root"""
    
    def __init__(
        self,
        id: OrderId,
        customer_id: CustomerId,
        shipping_address: Address
    ):
        self._id = id
        self._customer_id = customer_id
        self._shipping_address = shipping_address
        self._lines: List[OrderLine] = []
        self._status = OrderStatus.PENDING
        self._created_at = datetime.now()
        self._domain_events = []
    
    @classmethod
    def create(cls, customer_id: str, shipping_address: Address) -> 'Order':
        order = cls(
            id=OrderId.generate(),
            customer_id=CustomerId(customer_id),
            shipping_address=shipping_address
        )
        return order
    
    @property
    def id(self) -> OrderId:
        return self._id
    
    @property
    def status(self) -> OrderStatus:
        return self._status
    
    @property
    def total(self) -> Optional[Money]:
        if not self._lines:
            return None
        result = self._lines[0].subtotal
        for line in self._lines[1:]:
            result = result.add(line.subtotal)
        return result
    
    @property
    def line_count(self) -> int:
        return len(self._lines)
    
    def add_item(self, product_id: str, product_name: str, 
                 unit_price: float, currency: str, quantity: int):
        """Business rule: ไม่สามารถเพิ่ม item หลัง confirm"""
        if self._status != OrderStatus.PENDING:
            raise ValueError(f"Cannot add items to order in status: {self._status.value}")
        
        # Check ว่า product มีอยู่แล้วหรือไม่
        existing = self._find_line(ProductId(product_id))
        if existing:
            self._lines.remove(existing)
            quantity += existing.quantity
        
        line = OrderLine(
            product_id=ProductId(product_id),
            product_name=product_name,
            unit_price=Money(unit_price, currency),
            quantity=quantity
        )
        self._lines.append(line)
    
    def remove_item(self, product_id: str):
        line = self._find_line(ProductId(product_id))
        if not line:
            raise ValueError(f"Product {product_id} not in order")
        self._lines.remove(line)
    
    def confirm(self):
        if self._status != OrderStatus.PENDING:
            raise ValueError(f"Cannot confirm order in status: {self._status.value}")
        if not self._lines:
            raise ValueError("Cannot confirm empty order")
        
        self._status = OrderStatus.CONFIRMED
        self._domain_events.append({
            'type': 'OrderConfirmed',
            'order_id': self._id.value,
            'total': str(self.total),
            'timestamp': datetime.now().isoformat()
        })
    
    def ship(self, tracking_number: str):
        if self._status != OrderStatus.CONFIRMED:
            raise ValueError(f"Cannot ship order in status: {self._status.value}")
        self._status = OrderStatus.SHIPPED
        self._tracking_number = tracking_number
    
    def cancel(self, reason: str):
        if self._status in [OrderStatus.SHIPPED, OrderStatus.DELIVERED]:
            raise ValueError(f"Cannot cancel order in status: {self._status.value}")
        self._status = OrderStatus.CANCELLED
        self._cancellation_reason = reason
        self._domain_events.append({
            'type': 'OrderCancelled',
            'order_id': self._id.value,
            'reason': reason
        })
    
    def _find_line(self, product_id: ProductId) -> Optional[OrderLine]:
        for line in self._lines:
            if line.product_id == product_id:
                return line
        return None
    
    def get_lines(self) -> List[OrderLine]:
        return self._lines.copy()
    
    def pop_events(self) -> list:
        events = self._domain_events.copy()
        self._domain_events.clear()
        return events
    
    def __repr__(self) -> str:
        return f"Order(id={self._id.value}, status={self._status.value}, items={len(self._lines)})"

# ทดสอบ Order Aggregate
address = Address("456 Shopping Ave", "Bangkok", "10110", "TH")
order = Order.create("CUST-001", address)

order.add_item("PROD-001", "Python Book", 599.0, "THB", 2)
order.add_item("PROD-002", "USB Hub", 299.0, "THB", 1)

print(f"Order: {order}")
print(f"Total: {order.total}")
print(f"Lines: {order.line_count}")

order.confirm()
print(f"\nAfter confirm: {order}")
events = order.pop_events()
print(f"Events: {events}")

order.ship("TH-TRACK-12345")
print(f"After ship: {order}")
```

### 3.3 Repositories in DDD

```python
# ตัวอย่าง 6: Repository Pattern ใน DDD
from abc import ABC, abstractmethod
from typing import Optional, List

class OrderRepository(ABC):
    """Port - Interface สำหรับ Order Repository"""
    
    @abstractmethod
    def save(self, order: Order) -> None:
        pass
    
    @abstractmethod
    def find_by_id(self, order_id: OrderId) -> Optional[Order]:
        pass
    
    @abstractmethod
    def find_by_customer(self, customer_id: CustomerId) -> List[Order]:
        pass
    
    @abstractmethod
    def find_pending_orders(self) -> List[Order]:
        pass

class InMemoryOrderRepository(OrderRepository):
    """Adapter - Implementation สำหรับ testing"""
    
    def __init__(self):
        self._orders: dict = {}
    
    def save(self, order: Order) -> None:
        self._orders[order.id.value] = order
    
    def find_by_id(self, order_id: OrderId) -> Optional[Order]:
        return self._orders.get(order_id.value)
    
    def find_by_customer(self, customer_id: CustomerId) -> List[Order]:
        return [
            order for order in self._orders.values()
            if order._customer_id == customer_id
        ]
    
    def find_pending_orders(self) -> List[Order]:
        return [
            order for order in self._orders.values()
            if order.status == OrderStatus.PENDING
        ]
    
    def count(self) -> int:
        return len(self._orders)

# ทดสอบ Repository
repo = InMemoryOrderRepository()

# สร้างหลาย orders
for i in range(3):
    addr = Address(f"{i+1} Test St", "Bangkok", "10100", "TH")
    order = Order.create("CUST-001", addr)
    order.add_item("PROD-001", "Book", 100.0, "THB", 1)
    repo.save(order)

    if i == 0:
        order.confirm()  # ยืนยัน order แรก
        repo.save(order)

pending = repo.find_pending_orders()
print(f"Pending orders: {len(pending)}")

customer_orders = repo.find_by_customer(CustomerId("CUST-001"))
print(f"Customer orders: {len(customer_orders)}")
```

---

## 4. Domain Services

```python
# ตัวอย่าง 7: Domain Service
class PricingService:
    """Domain Service - business logic ที่ไม่เป็นของ Entity ใด"""
    
    def __init__(self, discount_repository):
        self._discount_repo = discount_repository
    
    def calculate_order_price(
        self, 
        order: Order, 
        customer_tier: str
    ) -> Money:
        """คำนวณราคา order โดยพิจารณา discounts"""
        base_total = order.total
        if not base_total:
            return Money(0, "THB")
        
        # Apply tier discount
        tier_discounts = {
            'bronze': 0.0,
            'silver': 0.05,
            'gold': 0.10,
            'platinum': 0.15
        }
        discount_rate = tier_discounts.get(customer_tier.lower(), 0)
        
        discount_amount = base_total.multiply(discount_rate)
        return base_total.subtract(discount_amount)
    
    def is_free_shipping_eligible(self, order: Order) -> bool:
        """Business rule: Free shipping ถ้า total > 1000 THB"""
        FREE_SHIPPING_THRESHOLD = Money(1000.0, "THB")
        total = order.total
        return total is not None and total > FREE_SHIPPING_THRESHOLD

class InventoryService:
    """Domain Service สำหรับ inventory"""
    
    def __init__(self, inventory_repository):
        self._inventory_repo = inventory_repository
    
    def check_availability(self, product_id: str, quantity: int) -> bool:
        """ตรวจสอบ stock"""
        stock = self._inventory_repo.get_stock(product_id)
        return stock >= quantity
    
    def reserve_items(self, order: Order) -> bool:
        """จอง items สำหรับ order"""
        for line in order.get_lines():
            if not self.check_availability(line.product_id.value, line.quantity):
                return False
        
        for line in order.get_lines():
            self._inventory_repo.reserve(line.product_id.value, line.quantity)
        
        return True
```

---

## 5. CQRS Pattern (Command Query Responsibility Segregation)

```python
# ตัวอย่าง 8: CQRS
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any, Optional, List

# Commands (write)
@dataclass
class Command(ABC):
    pass

@dataclass
class CreateOrderCommand(Command):
    customer_id: str
    shipping_address_street: str
    shipping_address_city: str
    shipping_address_postal_code: str
    shipping_address_country: str

@dataclass
class AddItemToOrderCommand(Command):
    order_id: str
    product_id: str
    product_name: str
    unit_price: float
    currency: str
    quantity: int

@dataclass
class ConfirmOrderCommand(Command):
    order_id: str

# Queries (read)
@dataclass
class Query(ABC):
    pass

@dataclass
class GetOrderByIdQuery(Query):
    order_id: str

@dataclass
class GetOrdersByCustomerQuery(Query):
    customer_id: str
    status: Optional[str] = None

# Command Handlers
class CommandHandler(ABC):
    @abstractmethod
    def handle(self, command: Command) -> Any:
        pass

class CreateOrderCommandHandler(CommandHandler):
    def __init__(self, order_repo: OrderRepository):
        self._repo = order_repo
    
    def handle(self, command: CreateOrderCommand) -> str:
        address = Address(
            street=command.shipping_address_street,
            city=command.shipping_address_city,
            postal_code=command.shipping_address_postal_code,
            country=command.shipping_address_country
        )
        order = Order.create(command.customer_id, address)
        self._repo.save(order)
        return order.id.value

class AddItemCommandHandler(CommandHandler):
    def __init__(self, order_repo: OrderRepository):
        self._repo = order_repo
    
    def handle(self, command: AddItemToOrderCommand) -> bool:
        order = self._repo.find_by_id(OrderId(command.order_id))
        if not order:
            raise ValueError(f"Order {command.order_id} not found")
        
        order.add_item(
            command.product_id,
            command.product_name,
            command.unit_price,
            command.currency,
            command.quantity
        )
        self._repo.save(order)
        return True

class ConfirmOrderCommandHandler(CommandHandler):
    def __init__(self, order_repo: OrderRepository):
        self._repo = order_repo
    
    def handle(self, command: ConfirmOrderCommand) -> bool:
        order = self._repo.find_by_id(OrderId(command.order_id))
        if not order:
            raise ValueError(f"Order {command.order_id} not found")
        order.confirm()
        self._repo.save(order)
        return True

# Query Handlers (Read models - can be separate/optimized)
@dataclass
class OrderSummaryDTO:
    order_id: str
    customer_id: str
    status: str
    total: Optional[str]
    item_count: int

class QueryHandler(ABC):
    @abstractmethod
    def handle(self, query: Query) -> Any:
        pass

class GetOrderByIdQueryHandler(QueryHandler):
    def __init__(self, order_repo: OrderRepository):
        self._repo = order_repo
    
    def handle(self, query: GetOrderByIdQuery) -> Optional[OrderSummaryDTO]:
        order = self._repo.find_by_id(OrderId(query.order_id))
        if not order:
            return None
        return OrderSummaryDTO(
            order_id=order.id.value,
            customer_id=order._customer_id.value,
            status=order.status.value,
            total=str(order.total) if order.total else None,
            item_count=order.line_count
        )

# Command Bus / Query Bus
class CommandBus:
    def __init__(self):
        self._handlers = {}
    
    def register(self, command_type: type, handler: CommandHandler):
        self._handlers[command_type] = handler
    
    def dispatch(self, command: Command) -> Any:
        handler = self._handlers.get(type(command))
        if not handler:
            raise ValueError(f"No handler for {type(command).__name__}")
        return handler.handle(command)

class QueryBus:
    def __init__(self):
        self._handlers = {}
    
    def register(self, query_type: type, handler: QueryHandler):
        self._handlers[query_type] = handler
    
    def ask(self, query: Query) -> Any:
        handler = self._handlers.get(type(query))
        if not handler:
            raise ValueError(f"No handler for {type(query).__name__}")
        return handler.handle(query)

# Setup
repo = InMemoryOrderRepository()

command_bus = CommandBus()
command_bus.register(CreateOrderCommand, CreateOrderCommandHandler(repo))
command_bus.register(AddItemToOrderCommand, AddItemCommandHandler(repo))
command_bus.register(ConfirmOrderCommand, ConfirmOrderCommandHandler(repo))

query_bus = QueryBus()
query_bus.register(GetOrderByIdQuery, GetOrderByIdQueryHandler(repo))

# การใช้งาน
print("=== CQRS Demo ===")

# Commands
order_id = command_bus.dispatch(CreateOrderCommand(
    customer_id="CUST-001",
    shipping_address_street="123 Main St",
    shipping_address_city="Bangkok",
    shipping_address_postal_code="10100",
    shipping_address_country="TH"
))
print(f"Created order: {order_id}")

command_bus.dispatch(AddItemToOrderCommand(
    order_id=order_id,
    product_id="PROD-001",
    product_name="Python Book",
    unit_price=599.0,
    currency="THB",
    quantity=2
))

command_bus.dispatch(ConfirmOrderCommand(order_id=order_id))

# Query
result = query_bus.ask(GetOrderByIdQuery(order_id=order_id))
print(f"Order summary: {result}")
```

---

## 6. Event Sourcing

```python
# ตัวอย่าง 9: Event Sourcing
from dataclasses import dataclass, field
from typing import List, Any
from datetime import datetime
import json

@dataclass
class DomainEvent:
    event_type: str
    aggregate_id: str
    data: dict
    version: int
    occurred_at: str = field(default_factory=lambda: datetime.now().isoformat())

class EventStore:
    """Store events แทน state"""
    
    def __init__(self):
        self._events: List[DomainEvent] = []
    
    def append(self, event: DomainEvent):
        self._events.append(event)
        print(f"[EventStore] Stored: {event.event_type} for {event.aggregate_id}")
    
    def get_events_for(self, aggregate_id: str) -> List[DomainEvent]:
        return [e for e in self._events if e.aggregate_id == aggregate_id]
    
    def get_all(self) -> List[DomainEvent]:
        return self._events.copy()

class BankAccountES:
    """Bank Account ที่ใช้ Event Sourcing"""
    
    def __init__(self, account_id: str):
        self._id = account_id
        self._balance = 0.0
        self._owner = None
        self._is_active = False
        self._version = 0
        self._pending_events: List[DomainEvent] = []
    
    @classmethod
    def create(cls, account_id: str, owner: str, initial_deposit: float) -> 'BankAccountES':
        account = cls(account_id)
        account._apply(DomainEvent(
            event_type='AccountOpened',
            aggregate_id=account_id,
            data={'owner': owner, 'initial_balance': initial_deposit},
            version=1
        ))
        return account
    
    @classmethod
    def rebuild_from_events(cls, account_id: str, events: List[DomainEvent]) -> 'BankAccountES':
        account = cls(account_id)
        for event in events:
            account._apply(event, recording=False)
        return account
    
    def deposit(self, amount: float):
        if not self._is_active:
            raise ValueError("Account is not active")
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._apply(DomainEvent(
            event_type='MoneyDeposited',
            aggregate_id=self._id,
            data={'amount': amount},
            version=self._version + 1
        ))
    
    def withdraw(self, amount: float):
        if not self._is_active:
            raise ValueError("Account is not active")
        if amount <= 0:
            raise ValueError("Withdrawal must be positive")
        if amount > self._balance:
            raise ValueError("Insufficient funds")
        self._apply(DomainEvent(
            event_type='MoneyWithdrawn',
            aggregate_id=self._id,
            data={'amount': amount},
            version=self._version + 1
        ))
    
    def close(self):
        if not self._is_active:
            raise ValueError("Account is already closed")
        self._apply(DomainEvent(
            event_type='AccountClosed',
            aggregate_id=self._id,
            data={'final_balance': self._balance},
            version=self._version + 1
        ))
    
    def _apply(self, event: DomainEvent, recording: bool = True):
        """Apply event to state"""
        if event.event_type == 'AccountOpened':
            self._owner = event.data['owner']
            self._balance = event.data['initial_balance']
            self._is_active = True
        elif event.event_type == 'MoneyDeposited':
            self._balance += event.data['amount']
        elif event.event_type == 'MoneyWithdrawn':
            self._balance -= event.data['amount']
        elif event.event_type == 'AccountClosed':
            self._is_active = False
        
        self._version = event.version
        if recording:
            self._pending_events.append(event)
    
    def pop_events(self) -> List[DomainEvent]:
        events = self._pending_events.copy()
        self._pending_events.clear()
        return events
    
    @property
    def balance(self) -> float:
        return self._balance
    
    @property
    def is_active(self) -> bool:
        return self._is_active

# ทดสอบ Event Sourcing
event_store = EventStore()

# สร้าง account
account = BankAccountES.create("ACC-001", "Alice", 1000.0)
for event in account.pop_events():
    event_store.append(event)

# ทำ transactions
account.deposit(500.0)
account.withdraw(200.0)
account.deposit(1000.0)
account.withdraw(300.0)

for event in account.pop_events():
    event_store.append(event)

print(f"\nCurrent balance: {account.balance}")
print(f"Active: {account.is_active}")

# Rebuild จาก events
print("\n=== Rebuilding from events ===")
events = event_store.get_events_for("ACC-001")
rebuilt = BankAccountES.rebuild_from_events("ACC-001", events)
print(f"Rebuilt balance: {rebuilt.balance}")
print(f"Rebuilt active: {rebuilt.is_active}")
print(f"Same state: {account.balance == rebuilt.balance}")
```

---

## 7. Hexagonal Architecture (Ports and Adapters)

```python
# ตัวอย่าง 10: Hexagonal Architecture
from abc import ABC, abstractmethod
from typing import Optional, List
from dataclasses import dataclass

# === DOMAIN (Core) ===
@dataclass
class Product:
    id: str
    name: str
    price: float
    stock: int
    
    def is_available(self, quantity: int) -> bool:
        return self.stock >= quantity
    
    def reduce_stock(self, quantity: int):
        if not self.is_available(quantity):
            raise ValueError(f"Insufficient stock for {self.name}")
        self.stock -= quantity

# === PORTS (Interfaces) ===
class ProductRepositoryPort(ABC):
    """Primary Port - เข้าถึง domain"""
    @abstractmethod
    def find_by_id(self, product_id: str) -> Optional[Product]:
        pass
    
    @abstractmethod
    def save(self, product: Product) -> None:
        pass
    
    @abstractmethod
    def find_all(self) -> List[Product]:
        pass

class NotificationPort(ABC):
    """Secondary Port - ออกจาก domain"""
    @abstractmethod
    def notify_low_stock(self, product: Product) -> None:
        pass
    
    @abstractmethod
    def notify_out_of_stock(self, product: Product) -> None:
        pass

# === DOMAIN SERVICE (Application Core) ===
LOW_STOCK_THRESHOLD = 5

class InventoryService:
    """Core business logic"""
    
    def __init__(
        self,
        product_repo: ProductRepositoryPort,
        notification: NotificationPort
    ):
        self._repo = product_repo
        self._notification = notification
    
    def purchase(self, product_id: str, quantity: int) -> bool:
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ValueError(f"Product {product_id} not found")
        
        product.reduce_stock(quantity)
        self._repo.save(product)
        
        if product.stock == 0:
            self._notification.notify_out_of_stock(product)
        elif product.stock <= LOW_STOCK_THRESHOLD:
            self._notification.notify_low_stock(product)
        
        return True
    
    def get_catalog(self) -> List[Product]:
        return self._repo.find_all()
    
    def add_stock(self, product_id: str, quantity: int) -> int:
        product = self._repo.find_by_id(product_id)
        if not product:
            raise ValueError(f"Product {product_id} not found")
        product.stock += quantity
        self._repo.save(product)
        return product.stock

# === ADAPTERS (Implementations) ===

# Primary Adapter - HTTP/REST
class HTTPAdapter:
    def __init__(self, inventory_service: InventoryService):
        self._service = inventory_service
    
    def handle_purchase_request(self, request: dict) -> dict:
        try:
            success = self._service.purchase(
                product_id=request['product_id'],
                quantity=int(request['quantity'])
            )
            return {'status': 200, 'success': success}
        except ValueError as e:
            return {'status': 400, 'error': str(e)}
    
    def handle_catalog_request(self) -> dict:
        products = self._service.get_catalog()
        return {
            'status': 200,
            'products': [
                {
                    'id': p.id,
                    'name': p.name,
                    'price': p.price,
                    'in_stock': p.stock > 0
                }
                for p in products
            ]
        }

# Secondary Adapters
class InMemoryProductRepository(ProductRepositoryPort):
    def __init__(self):
        self._products = {}
    
    def find_by_id(self, product_id: str) -> Optional[Product]:
        return self._products.get(product_id)
    
    def save(self, product: Product) -> None:
        self._products[product.id] = product
    
    def find_all(self) -> List[Product]:
        return list(self._products.values())

class ConsoleNotification(NotificationPort):
    def notify_low_stock(self, product: Product) -> None:
        print(f"⚠️ LOW STOCK: {product.name} - only {product.stock} left!")
    
    def notify_out_of_stock(self, product: Product) -> None:
        print(f"🚫 OUT OF STOCK: {product.name}!")

class EmailNotification(NotificationPort):
    def notify_low_stock(self, product: Product) -> None:
        print(f"📧 Email: Low stock alert for {product.name} ({product.stock} remaining)")
    
    def notify_out_of_stock(self, product: Product) -> None:
        print(f"📧 Email: {product.name} is now out of stock!")

# === Composition Root ===
def create_inventory_system(use_email: bool = False):
    repo = InMemoryProductRepository()
    notification = EmailNotification() if use_email else ConsoleNotification()
    service = InventoryService(repo, notification)
    adapter = HTTPAdapter(service)
    return repo, service, adapter

# ทดสอบ
repo, service, http = create_inventory_system()

# Seed data
for product_data in [
    Product("PROD-001", "Python Book", 599.0, 8),
    Product("PROD-002", "USB Hub", 299.0, 3),
    Product("PROD-003", "Webcam", 899.0, 1),
]:
    repo.save(product_data)

print("=== Hexagonal Architecture Demo ===")
catalog = http.handle_catalog_request()
print(f"Catalog: {len(catalog['products'])} products")

print("\nPurchasing Python Books...")
for i in range(4):
    result = http.handle_purchase_request({'product_id': 'PROD-001', 'quantity': 1})
    print(f"  Purchase {i+1}: {result}")

print("\nPurchasing USB Hub (out of stock)...")
result = http.handle_purchase_request({'product_id': 'PROD-002', 'quantity': 5})
print(f"Result: {result}")
```

---

## 8. E-Commerce Domain Example

```python
# ตัวอย่าง 11: Complete E-Commerce Domain

# === Value Objects ===
from dataclasses import dataclass, field
from typing import Optional, List
from enum import Enum
from datetime import datetime

@dataclass(frozen=True)
class ProductSku:
    value: str
    
    def __post_init__(self):
        if not self.value or not self.value.strip():
            raise ValueError("SKU cannot be empty")

@dataclass(frozen=True)
class Rating:
    value: float
    
    def __post_init__(self):
        if not (0 <= self.value <= 5):
            raise ValueError("Rating must be between 0 and 5")

# === Entities ===
class Category:
    def __init__(self, id: str, name: str, description: str = ""):
        self._id = id
        self._name = name
        self._description = description
    
    @property
    def id(self): return self._id
    
    @property
    def name(self): return self._name

class ProductItem:
    """Entity"""
    def __init__(self, sku: ProductSku, name: str, price: Money, 
                 category: Category, description: str = ""):
        self._sku = sku
        self._name = name
        self._price = price
        self._category = category
        self._description = description
        self._reviews: List['Review'] = []
        self._stock = 0
    
    @property
    def sku(self): return self._sku
    
    @property
    def name(self): return self._name
    
    @property
    def price(self): return self._price
    
    @property
    def category(self): return self._category
    
    @property
    def stock(self): return self._stock
    
    def restock(self, quantity: int):
        if quantity <= 0:
            raise ValueError("Restock quantity must be positive")
        self._stock += quantity
    
    def reserve(self, quantity: int) -> bool:
        if self._stock < quantity:
            return False
        self._stock -= quantity
        return True
    
    def add_review(self, rating: float, comment: str, reviewer: str):
        self._reviews.append(Review(Rating(rating), comment, reviewer))
    
    @property
    def average_rating(self) -> Optional[float]:
        if not self._reviews:
            return None
        return sum(r.rating.value for r in self._reviews) / len(self._reviews)
    
    def __repr__(self):
        return f"Product(sku={self._sku.value}, name={self._name}, price={self._price})"

@dataclass
class Review:
    rating: Rating
    comment: str
    reviewer: str
    created_at: datetime = field(default_factory=datetime.now)

# === Shopping Cart Aggregate ===
class CartItem:
    def __init__(self, product: ProductItem, quantity: int):
        self._product = product
        self._quantity = quantity
    
    @property
    def product(self): return self._product
    
    @property
    def quantity(self): return self._quantity
    
    @property
    def subtotal(self) -> Money:
        return self._product.price.multiply(self._quantity)
    
    def update_quantity(self, new_quantity: int):
        if new_quantity <= 0:
            raise ValueError("Quantity must be positive")
        self._quantity = new_quantity

class ShoppingCart:
    """Aggregate Root"""
    
    def __init__(self, cart_id: str, customer_id: str):
        self._id = cart_id
        self._customer_id = customer_id
        self._items: List[CartItem] = []
        self._created_at = datetime.now()
    
    def add_product(self, product: ProductItem, quantity: int):
        existing = self._find_item(product.sku)
        if existing:
            existing.update_quantity(existing.quantity + quantity)
        else:
            self._items.append(CartItem(product, quantity))
    
    def remove_product(self, sku: ProductSku):
        item = self._find_item(sku)
        if item:
            self._items.remove(item)
    
    def update_quantity(self, sku: ProductSku, quantity: int):
        item = self._find_item(sku)
        if not item:
            raise ValueError(f"Product {sku.value} not in cart")
        if quantity == 0:
            self._items.remove(item)
        else:
            item.update_quantity(quantity)
    
    def clear(self):
        self._items.clear()
    
    @property
    def total(self) -> Money:
        if not self._items:
            return Money(0, "THB")
        result = self._items[0].subtotal
        for item in self._items[1:]:
            result = result.add(item.subtotal)
        return result
    
    @property
    def item_count(self) -> int:
        return sum(item.quantity for item in self._items)
    
    def _find_item(self, sku: ProductSku) -> Optional[CartItem]:
        for item in self._items:
            if item.product.sku == sku:
                return item
        return None
    
    def get_items(self) -> List[CartItem]:
        return self._items.copy()

# ทดสอบ E-Commerce Domain
tech_category = Category("CAT-001", "Technology")

book = ProductItem(
    ProductSku("BOOK-PYTHON-2024"),
    "Python Mastery",
    Money(599.0, "THB"),
    tech_category,
    "Complete Python programming guide"
)
book.restock(100)
book.add_review(5.0, "Excellent book!", "alice")
book.add_review(4.5, "Very helpful", "bob")

laptop_stand = ProductItem(
    ProductSku("ACC-STAND-001"),
    "Laptop Stand",
    Money(399.0, "THB"),
    tech_category
)
laptop_stand.restock(50)

# สร้าง cart
cart = ShoppingCart("CART-001", "CUST-001")
cart.add_product(book, 2)
cart.add_product(laptop_stand, 1)

print("=== Shopping Cart ===")
for item in cart.get_items():
    print(f"  {item.product.name} x{item.quantity} = {item.subtotal}")
print(f"Total: {cart.total}")
print(f"Item count: {cart.item_count}")
print(f"Book rating: {book.average_rating:.1f}/5.0")
```

---

## 9. Domain Events และ Integration Events

```python
# ตัวอย่าง 12: Domain Events
from dataclasses import dataclass, field
from datetime import datetime
from typing import Callable, Dict, List, Type
import uuid

@dataclass
class DomainEventBase:
    event_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    occurred_at: str = field(default_factory=lambda: datetime.now().isoformat())

@dataclass
class OrderPlacedEvent(DomainEventBase):
    order_id: str = ""
    customer_id: str = ""
    total_amount: float = 0.0
    items_count: int = 0

@dataclass
class PaymentProcessedEvent(DomainEventBase):
    order_id: str = ""
    amount: float = 0.0
    payment_method: str = ""
    transaction_id: str = ""

@dataclass
class StockReservedEvent(DomainEventBase):
    order_id: str = ""
    product_id: str = ""
    quantity: int = 0

# Event Bus
EventHandler = Callable[[DomainEventBase], None]

class DomainEventBus:
    def __init__(self):
        self._handlers: Dict[Type, List[EventHandler]] = {}
    
    def subscribe(self, event_type: Type, handler: EventHandler):
        if event_type not in self._handlers:
            self._handlers[event_type] = []
        self._handlers[event_type].append(handler)
    
    def publish(self, event: DomainEventBase):
        handlers = self._handlers.get(type(event), [])
        for handler in handlers:
            try:
                handler(event)
            except Exception as e:
                print(f"Handler error for {type(event).__name__}: {e}")

# Event Handlers (Subscribers)
def handle_order_placed(event: OrderPlacedEvent):
    print(f"📦 InventoryService: Reserving items for order {event.order_id}")

def send_order_confirmation(event: OrderPlacedEvent):
    print(f"📧 EmailService: Sending confirmation for order {event.order_id}")

def update_analytics(event: OrderPlacedEvent):
    print(f"📊 Analytics: Recording order {event.order_id} - ${event.total_amount}")

def handle_payment_processed(event: PaymentProcessedEvent):
    print(f"✅ OrderService: Payment confirmed for order {event.order_id}")

def send_receipt(event: PaymentProcessedEvent):
    print(f"🧾 EmailService: Sending receipt for ${event.amount} to customer")

# Setup
event_bus = DomainEventBus()
event_bus.subscribe(OrderPlacedEvent, handle_order_placed)
event_bus.subscribe(OrderPlacedEvent, send_order_confirmation)
event_bus.subscribe(OrderPlacedEvent, update_analytics)
event_bus.subscribe(PaymentProcessedEvent, handle_payment_processed)
event_bus.subscribe(PaymentProcessedEvent, send_receipt)

# Simulate order flow
print("=== Domain Events Flow ===")
event_bus.publish(OrderPlacedEvent(
    order_id="ORD-001",
    customer_id="CUST-001",
    total_amount=1198.0,
    items_count=3
))

print()
event_bus.publish(PaymentProcessedEvent(
    order_id="ORD-001",
    amount=1198.0,
    payment_method="credit_card",
    transaction_id="TXN-12345"
))
```

---

## 10. Complete DDD Application Example

```python
# ตัวอย่าง 13: Complete Application ด้วย DDD

class ProductCatalogService:
    """Application Service - Orchestrates domain objects"""
    
    def __init__(self, product_repo, event_bus: DomainEventBus):
        self._repo = product_repo
        self._event_bus = event_bus
    
    def create_product(
        self, sku: str, name: str, price: float, 
        currency: str, category_id: str, description: str = ""
    ) -> ProductItem:
        # Create domain objects
        product = ProductItem(
            ProductSku(sku),
            name,
            Money(price, currency),
            Category(category_id, "General"),
            description
        )
        self._repo.save(product)
        return product
    
    def add_stock(self, sku: str, quantity: int) -> int:
        product = self._repo.find_by_sku(ProductSku(sku))
        if not product:
            raise ValueError(f"Product {sku} not found")
        product.restock(quantity)
        self._repo.save(product)
        return product.stock
    
    def review_product(self, sku: str, rating: float, comment: str, reviewer: str):
        product = self._repo.find_by_sku(ProductSku(sku))
        if not product:
            raise ValueError(f"Product {sku} not found")
        product.add_review(rating, comment, reviewer)
        self._repo.save(product)

class CartService:
    """Application Service สำหรับ Shopping Cart"""
    
    def __init__(self, cart_repo, product_repo, event_bus: DomainEventBus):
        self._cart_repo = cart_repo
        self._product_repo = product_repo
        self._event_bus = event_bus
    
    def get_or_create_cart(self, customer_id: str) -> ShoppingCart:
        cart = self._cart_repo.find_by_customer(customer_id)
        if not cart:
            cart = ShoppingCart(
                f"CART-{customer_id}",
                customer_id
            )
            self._cart_repo.save(cart)
        return cart
    
    def add_item(self, customer_id: str, sku: str, quantity: int):
        product = self._product_repo.find_by_sku(ProductSku(sku))
        if not product:
            raise ValueError(f"Product {sku} not found")
        if product.stock < quantity:
            raise ValueError(f"Insufficient stock for {product.name}")
        
        cart = self.get_or_create_cart(customer_id)
        cart.add_product(product, quantity)
        self._cart_repo.save(cart)
    
    def get_cart_summary(self, customer_id: str) -> dict:
        cart = self.get_or_create_cart(customer_id)
        return {
            'customer_id': customer_id,
            'items': [
                {
                    'sku': item.product.sku.value,
                    'name': item.product.name,
                    'quantity': item.quantity,
                    'unit_price': str(item.product.price),
                    'subtotal': str(item.subtotal)
                }
                for item in cart.get_items()
            ],
            'total': str(cart.total),
            'item_count': cart.item_count
        }

# Repositories
class InMemoryProductRepo:
    def __init__(self):
        self._products = {}
    
    def save(self, product: ProductItem):
        self._products[product.sku.value] = product
    
    def find_by_sku(self, sku: ProductSku) -> Optional[ProductItem]:
        return self._products.get(sku.value)

class InMemoryCartRepo:
    def __init__(self):
        self._carts = {}
    
    def save(self, cart: ShoppingCart):
        self._carts[cart._customer_id] = cart
    
    def find_by_customer(self, customer_id: str) -> Optional[ShoppingCart]:
        return self._carts.get(customer_id)

# Wiring
event_bus = DomainEventBus()
product_repo = InMemoryProductRepo()
cart_repo = InMemoryCartRepo()

catalog_service = ProductCatalogService(product_repo, event_bus)
cart_service = CartService(cart_repo, product_repo, event_bus)

# Demo
print("=== E-Commerce DDD Demo ===")

# สร้าง products
p1 = catalog_service.create_product("BOOK-001", "Clean Code", 750.0, "THB", "CAT-001")
p2 = catalog_service.create_product("BOOK-002", "DDD Book", 890.0, "THB", "CAT-001")

catalog_service.add_stock("BOOK-001", 50)
catalog_service.add_stock("BOOK-002", 30)

catalog_service.review_product("BOOK-001", 5.0, "Must read!", "alice")
catalog_service.review_product("BOOK-001", 4.5, "Great book", "bob")

# Shopping
cart_service.add_item("CUST-001", "BOOK-001", 2)
cart_service.add_item("CUST-001", "BOOK-002", 1)

summary = cart_service.get_cart_summary("CUST-001")
print(f"\nCart Summary:")
for item in summary['items']:
    print(f"  {item['name']} x{item['quantity']} = {item['subtotal']}")
print(f"Total: {summary['total']}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Library Management Domain

```python
# เฉลย - Library Domain
from dataclasses import dataclass, field
from typing import Optional, List
from datetime import datetime, timedelta
from enum import Enum

class LoanStatus(Enum):
    ACTIVE = "active"
    RETURNED = "returned"
    OVERDUE = "overdue"

@dataclass(frozen=True)
class ISBN:
    value: str
    
    def __post_init__(self):
        clean = self.value.replace('-', '').replace(' ', '')
        if not (len(clean) in [10, 13] and clean.isdigit()):
            raise ValueError(f"Invalid ISBN: {self.value}")

@dataclass(frozen=True)
class MemberId:
    value: str

class Book:
    """Entity"""
    def __init__(self, isbn: ISBN, title: str, author: str, total_copies: int):
        self._isbn = isbn
        self._title = title
        self._author = author
        self._total_copies = total_copies
        self._available_copies = total_copies
    
    @property
    def isbn(self): return self._isbn
    
    @property
    def title(self): return self._title
    
    @property
    def author(self): return self._author
    
    @property
    def available_copies(self): return self._available_copies
    
    def is_available(self) -> bool:
        return self._available_copies > 0
    
    def checkout(self):
        if not self.is_available():
            raise ValueError(f"'{self._title}' is not available")
        self._available_copies -= 1
    
    def return_copy(self):
        if self._available_copies >= self._total_copies:
            raise ValueError("All copies already returned")
        self._available_copies += 1
    
    def __repr__(self):
        return f"Book(isbn={self._isbn.value}, title={self._title}, available={self._available_copies}/{self._total_copies})"

@dataclass
class Loan:
    """Entity"""
    loan_id: str
    book_isbn: ISBN
    member_id: MemberId
    checkout_date: datetime
    due_date: datetime
    return_date: Optional[datetime] = None
    
    @property
    def status(self) -> LoanStatus:
        if self.return_date:
            return LoanStatus.RETURNED
        if datetime.now() > self.due_date:
            return LoanStatus.OVERDUE
        return LoanStatus.ACTIVE
    
    @property
    def days_overdue(self) -> int:
        if self.status != LoanStatus.OVERDUE:
            return 0
        return (datetime.now() - self.due_date).days
    
    @property
    def overdue_fine(self) -> float:
        FINE_PER_DAY = 5.0
        return self.days_overdue * FINE_PER_DAY

class LibraryService:
    """Domain Service"""
    LOAN_PERIOD_DAYS = 14
    
    def __init__(self, book_repo, loan_repo):
        self._books = book_repo
        self._loans = loan_repo
    
    def checkout_book(self, isbn: str, member_id: str) -> Loan:
        book = self._books.find_by_isbn(ISBN(isbn))
        if not book:
            raise ValueError(f"Book {isbn} not found")
        
        active_loans = self._loans.find_active_by_member(MemberId(member_id))
        if len(active_loans) >= 3:
            raise ValueError("Member has reached maximum loan limit (3)")
        
        existing = self._loans.find_active_by_book_and_member(ISBN(isbn), MemberId(member_id))
        if existing:
            raise ValueError("Member already has this book")
        
        book.checkout()
        self._books.save(book)
        
        import uuid
        loan = Loan(
            loan_id=str(uuid.uuid4()),
            book_isbn=ISBN(isbn),
            member_id=MemberId(member_id),
            checkout_date=datetime.now(),
            due_date=datetime.now() + timedelta(days=self.LOAN_PERIOD_DAYS)
        )
        self._loans.save(loan)
        return loan
    
    def return_book(self, loan_id: str) -> dict:
        loan = self._loans.find_by_id(loan_id)
        if not loan:
            raise ValueError(f"Loan {loan_id} not found")
        if loan.status == LoanStatus.RETURNED:
            raise ValueError("Book already returned")
        
        book = self._books.find_by_isbn(loan.book_isbn)
        book.return_copy()
        self._books.save(book)
        
        fine = loan.overdue_fine
        loan.return_date = datetime.now()
        self._loans.save(loan)
        
        return {
            'loan_id': loan_id,
            'returned': True,
            'overdue_fine': fine
        }

# Simple repositories
class BookRepo:
    def __init__(self):
        self._books = {}
    
    def save(self, book: Book):
        self._books[book.isbn.value] = book
    
    def find_by_isbn(self, isbn: ISBN) -> Optional[Book]:
        return self._books.get(isbn.value)

class LoanRepo:
    def __init__(self):
        self._loans = {}
    
    def save(self, loan: Loan):
        self._loans[loan.loan_id] = loan
    
    def find_by_id(self, loan_id: str) -> Optional[Loan]:
        return self._loans.get(loan_id)
    
    def find_active_by_member(self, member_id: MemberId) -> List[Loan]:
        return [
            l for l in self._loans.values()
            if l.member_id == member_id and l.status != LoanStatus.RETURNED
        ]
    
    def find_active_by_book_and_member(self, isbn: ISBN, member_id: MemberId) -> Optional[Loan]:
        for loan in self._loans.values():
            if (loan.book_isbn == isbn and loan.member_id == member_id 
                    and loan.status != LoanStatus.RETURNED):
                return loan
        return None

# ทดสอบ
book_repo = BookRepo()
loan_repo = LoanRepo()
library = LibraryService(book_repo, loan_repo)

# เพิ่มหนังสือ
for isbn, title, author, copies in [
    ("9780132350884", "Clean Code", "Robert C. Martin", 3),
    ("9780321125217", "Domain-Driven Design", "Eric Evans", 2),
]:
    book = Book(ISBN(isbn), title, author, copies)
    book_repo.save(book)

# Checkout
loan1 = library.checkout_book("9780132350884", "MEMBER-001")
print(f"Checked out: {loan1.loan_id}")
print(f"Due: {loan1.due_date.strftime('%Y-%m-%d')}")

# Return
result = library.return_book(loan1.loan_id)
print(f"Returned: {result}")
```

---

## สรุป

Clean Architecture และ DDD ช่วยให้:

1. **Clean Architecture**:
   - แยก Business Logic ออกจาก Infrastructure
   - ทดสอบได้ง่าย เพราะไม่ขึ้นกับ Framework
   - เปลี่ยน Database/Framework ได้โดยไม่กระทบ core logic

2. **Domain-Driven Design**:
   - โค้ดสะท้อน Business domain
   - Ubiquitous Language ทำให้ทีมสื่อสารกันได้ง่าย
   - Bounded Contexts แบ่งโปรแกรมเป็นส่วนย่อย

| Concept | คำอธิบาย |
|---------|---------|
| Entity | มี Identity, สามารถเปลี่ยนแปลงได้ |
| Value Object | ไม่มี Identity, Immutable, เปรียบเทียบด้วย Value |
| Aggregate | กลุ่ม Entities/VOs ที่มี Root |
| Repository | Interface สำหรับ Data Access |
| Domain Service | Business logic ที่ไม่เป็นของ Entity ใด |
| Domain Event | สิ่งที่เกิดขึ้นใน Domain |
| CQRS | แยก Read/Write operations |
| Event Sourcing | เก็บ Events แทน State |

---

*Part 88 เสร็จสมบูรณ์ | ต่อไป: Part 89 - Microservices with Python*
