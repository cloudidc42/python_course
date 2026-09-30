# Part 58: FastAPI - Routing, Validation & Pydantic

## สารบัญ

1. [Pydantic v2 Models](#1-pydantic-v2-models)
2. [Field Validation](#2-field-validation)
3. [Custom Validators](#3-custom-validators)
4. [Nested Models](#4-nested-models)
5. [Optional Fields](#5-optional-fields)
6. [Response Models](#6-response-models)
7. [APIRouter](#7-apirouter)
8. [Tags](#8-tags)
9. [Include Router](#9-include-router)
10. [CORS Middleware](#10-cors-middleware)
11. [Custom Middleware](#11-custom-middleware)
12. [BackgroundTasks](#12-backgroundtasks)
13. [Request/Response Lifecycle](#13-requestresponse-lifecycle)
14. [Webhook Handlers](#14-webhook-handlers)
15. [แบบฝึกหัด](#15-แบบฝึกหัด)

---

## 1. Pydantic v2 Models

Pydantic v2 มีการเปลี่ยนแปลงหลายอย่างจาก v1 โดยเฉพาะด้าน performance และ API

### การติดตั้ง

```bash
pip install pydantic>=2.0.0
```

### BaseModel พื้นฐาน

```python
from pydantic import BaseModel, Field
from typing import Optional, List, Dict, Any
from datetime import datetime
from enum import Enum

# ====== Basic Model ======
class UserModel(BaseModel):
    id: int
    username: str
    email: str
    is_active: bool = True
    created_at: datetime = None
    
    # Pydantic v2: ใช้ model_config แทน class Config
    model_config = {
        "str_strip_whitespace": True,  # Strip whitespace from strings
        "str_max_length": 200,
    }

# สร้าง instance
user = UserModel(
    id=1,
    username="alice",
    email="alice@example.com"
)
print(user.id)           # 1
print(user.username)     # "alice"
print(user.model_dump()) # {'id': 1, 'username': 'alice', 'email': 'alice@example.com', 'is_active': True, 'created_at': None}
print(user.model_dump_json())  # JSON string
```

### Model Methods ใน Pydantic v2

```python
from pydantic import BaseModel, model_validator
from typing import Optional

class Product(BaseModel):
    name: str
    price: float
    discount: Optional[float] = None
    final_price: Optional[float] = None
    
    # Pydantic v2: model_validator แทน root_validator
    @model_validator(mode='after')
    def calculate_final_price(self):
        if self.discount is not None:
            self.final_price = self.price * (1 - self.discount)
        else:
            self.final_price = self.price
        return self

product = Product(name="Laptop", price=1000.0, discount=0.1)
print(product.final_price)  # 900.0

# Model copy
product_copy = product.model_copy(update={"price": 1200.0})
print(product_copy.price)  # 1200.0

# JSON serialization
json_str = product.model_dump_json()
product_from_json = Product.model_validate_json(json_str)
```

### Pydantic v2: model_config

```python
from pydantic import BaseModel, ConfigDict

class StrictModel(BaseModel):
    model_config = ConfigDict(
        strict=True,                # ไม่ทำ coercion
        str_strip_whitespace=True,  # Strip whitespace
        str_min_length=1,           # Min string length
        use_enum_values=True,       # ใช้ enum values ไม่ใช่ enum objects
        validate_default=True,      # Validate default values
        populate_by_name=True,      # รับทั้ง alias และ field name
        from_attributes=True,       # สร้างจาก ORM objects
        arbitrary_types_allowed=True,  # อนุญาต custom types
    )
    
    name: str
    age: int

# Validate from dict
data = {"name": "  Alice  ", "age": 30}
model = StrictModel.model_validate(data)
print(model.name)  # "Alice" (stripped)
```

---

## 2. Field Validation

### Built-in Field Validators

```python
from pydantic import BaseModel, Field
from typing import Optional, List
import re

class UserRegistration(BaseModel):
    # String validation
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        pattern=r'^[a-zA-Z0-9_]+$',  # v2: ใช้ pattern แทน regex
        description="Username (alphanumeric and underscore only)",
        examples=["john_doe", "alice123"]
    )
    
    # Number validation
    age: int = Field(
        ...,
        ge=13,       # greater than or equal
        le=120,      # less than or equal
        description="User age"
    )
    
    score: float = Field(
        default=0.0,
        ge=0.0,
        le=100.0,
        description="User score (0-100)"
    )
    
    # String with constraints
    password: str = Field(
        ...,
        min_length=8,
        max_length=100,
        description="Password (min 8 chars)"
    )
    
    # Optional with default
    bio: Optional[str] = Field(
        default=None,
        max_length=500,
        description="User biography"
    )
    
    # List with constraints
    tags: List[str] = Field(
        default=[],
        max_length=10,  # Max 10 tags
        description="User tags"
    )

# ทดสอบ
valid_user = UserRegistration(
    username="john_doe",
    age=25,
    password="SecurePass123"
)

# Invalid - จะ raise ValidationError
try:
    invalid_user = UserRegistration(
        username="jo",  # Too short (< 3)
        age=10,         # Too young (< 13)
        password="weak" # Too short (< 8)
    )
except Exception as e:
    print(e)
```

### Number Field Validation

```python
from pydantic import BaseModel, Field
from decimal import Decimal

class PriceModel(BaseModel):
    # Integer constraints
    quantity: int = Field(..., ge=1, le=10000, multiple_of=1)
    
    # Float with precision
    price: float = Field(..., gt=0.0, description="Price in USD")
    
    # Decimal for financial data (more accurate)
    exact_price: Decimal = Field(..., gt=Decimal("0"), decimal_places=2)
    
    # Exclusive bounds
    percentage: float = Field(..., gt=0.0, lt=100.0)  # Exclusive

class InventoryItem(BaseModel):
    stock: int = Field(..., ge=0)           # Non-negative
    min_stock: int = Field(default=10, ge=0)
    max_stock: int = Field(default=1000, ge=1)
    reorder_point: int = Field(default=50, ge=0)
```

### String Field Validation

```python
from pydantic import BaseModel, Field, AnyUrl, EmailStr
from typing import Optional

# ต้องติดตั้ง: pip install pydantic[email]
class ContactModel(BaseModel):
    # Email validation
    email: EmailStr = Field(..., description="Valid email address")
    
    # URL validation
    website: Optional[AnyUrl] = Field(None, description="Website URL")
    
    # Phone number (with regex)
    phone: Optional[str] = Field(
        None,
        pattern=r'^\+?[1-9]\d{1,14}$',  # E.164 format
        description="Phone in E.164 format"
    )
    
    # Custom regex
    zip_code: Optional[str] = Field(
        None,
        pattern=r'^\d{5}(-\d{4})?$',  # US zip code
    )
    
    # IP address-like string
    server_ip: Optional[str] = Field(
        None,
        pattern=r'^(\d{1,3}\.){3}\d{1,3}$'
    )

# ตัวอย่าง valid
contact = ContactModel(
    email="alice@example.com",
    website="https://alice.com",
    phone="+66812345678"
)
```

---

## 3. Custom Validators

### field_validator (Pydantic v2)

```python
from pydantic import BaseModel, field_validator, model_validator
from typing import Optional
import re
from datetime import datetime

class UserProfile(BaseModel):
    username: str
    email: str
    password: str
    confirm_password: str
    birth_date: Optional[str] = None
    
    # field_validator: ตรวจสอบ field เดียว
    @field_validator('username')
    @classmethod
    def username_must_be_valid(cls, v: str) -> str:
        """Username ต้องเป็น alphanumeric และไม่เริ่มต้นด้วยตัวเลข"""
        v = v.strip().lower()
        
        if not re.match(r'^[a-zA-Z][a-zA-Z0-9_]{2,}$', v):
            raise ValueError(
                'Username must start with a letter and contain only '
                'alphanumeric characters and underscores (min 3 chars)'
            )
        
        # ตรวจสอบ reserved words
        reserved = ['admin', 'root', 'system', 'api']
        if v in reserved:
            raise ValueError(f'Username "{v}" is reserved')
        
        return v
    
    @field_validator('email')
    @classmethod
    def email_must_be_valid(cls, v: str) -> str:
        """Validate email format และแปลงเป็น lowercase"""
        v = v.strip().lower()
        
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, v):
            raise ValueError('Invalid email format')
        
        return v
    
    @field_validator('password')
    @classmethod
    def password_strength(cls, v: str) -> str:
        """ตรวจสอบความแข็งแกร่งของ password"""
        errors = []
        
        if len(v) < 8:
            errors.append("At least 8 characters")
        if not re.search(r'[A-Z]', v):
            errors.append("At least one uppercase letter")
        if not re.search(r'[a-z]', v):
            errors.append("At least one lowercase letter")
        if not re.search(r'\d', v):
            errors.append("At least one digit")
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', v):
            errors.append("At least one special character")
        
        if errors:
            raise ValueError(f"Password requirements not met: {', '.join(errors)}")
        
        return v
    
    @field_validator('birth_date')
    @classmethod
    def validate_birth_date(cls, v: Optional[str]) -> Optional[str]:
        """Validate birth date format และ age"""
        if v is None:
            return v
        
        try:
            birth = datetime.strptime(v, '%Y-%m-%d')
        except ValueError:
            raise ValueError('Birth date must be in YYYY-MM-DD format')
        
        today = datetime.now()
        age = (today - birth).days // 365
        
        if age < 13:
            raise ValueError('You must be at least 13 years old')
        if age > 120:
            raise ValueError('Invalid birth date')
        
        return v
    
    # model_validator: ตรวจสอบหลาย fields พร้อมกัน
    @model_validator(mode='after')
    def passwords_match(self):
        """ตรวจสอบ password และ confirm_password ตรงกัน"""
        if self.password != self.confirm_password:
            raise ValueError('Passwords do not match')
        return self

# ทดสอบ
try:
    user = UserProfile(
        username="alice_dev",
        email="alice@example.com",
        password="SecurePass123!",
        confirm_password="SecurePass123!",
        birth_date="1990-01-15"
    )
    print("Valid user:", user.username)
except Exception as e:
    print("Validation error:", e)
```

### model_validator Mode Before/After

```python
from pydantic import BaseModel, model_validator
from typing import Optional
from datetime import date

class DateRange(BaseModel):
    start_date: date
    end_date: date
    days: Optional[int] = None  # Calculated field
    
    @model_validator(mode='before')
    @classmethod
    def check_dates_before_creation(cls, values: dict) -> dict:
        """
        mode='before': รันก่อน field validation
        ใช้สำหรับ transform input data
        """
        # Convert string dates ถ้าเป็น string
        if isinstance(values.get('start_date'), str):
            values['start_date'] = date.fromisoformat(values['start_date'])
        if isinstance(values.get('end_date'), str):
            values['end_date'] = date.fromisoformat(values['end_date'])
        return values
    
    @model_validator(mode='after')
    def check_date_range(self):
        """
        mode='after': รันหลัง field validation
        ใช้สำหรับ cross-field validation
        """
        if self.start_date > self.end_date:
            raise ValueError('start_date must be before end_date')
        
        # Calculate days
        self.days = (self.end_date - self.start_date).days
        return self

# ใช้งาน
from fastapi import FastAPI
app = FastAPI()

@app.post("/date-range")
async def validate_date_range(date_range: DateRange):
    return {
        "start": str(date_range.start_date),
        "end": str(date_range.end_date),
        "days": date_range.days
    }
```

### BeforeValidator, AfterValidator (Pydantic v2)

```python
from pydantic import BaseModel
from pydantic.functional_validators import BeforeValidator, AfterValidator
from typing import Annotated
import re

# Custom type ด้วย Annotated
def normalize_phone(v: str) -> str:
    """ลบ spaces, dashes, brackets จาก phone number"""
    return re.sub(r'[\s\-\(\)]', '', v)

def validate_thai_phone(v: str) -> str:
    """ตรวจสอบ Thai phone number"""
    if not re.match(r'^0[0-9]{9}$', v):
        raise ValueError('Invalid Thai phone number (format: 0XXXXXXXXX)')
    return v

# Annotated type สำหรับ Thai phone
ThaiPhone = Annotated[
    str,
    BeforeValidator(normalize_phone),
    AfterValidator(validate_thai_phone)
]

class CustomerModel(BaseModel):
    name: str
    phone: ThaiPhone

# ทดสอบ
customer = CustomerModel(
    name="Alice",
    phone="081-234-5678"  # จะถูก normalize เป็น "0812345678"
)
print(customer.phone)  # "0812345678"
```

---

## 4. Nested Models

### Basic Nested Models

```python
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

class Address(BaseModel):
    street: str
    city: str
    state: Optional[str] = None
    country: str = "Thailand"
    postal_code: str

class ContactInfo(BaseModel):
    email: str
    phone: Optional[str] = None
    website: Optional[str] = None

class UserProfile(BaseModel):
    id: int
    username: str
    full_name: str
    address: Address  # Nested model
    contact: ContactInfo  # Nested model
    tags: List[str] = []
    created_at: datetime = Field(default_factory=datetime.utcnow)

# สร้าง nested instance
profile = UserProfile(
    id=1,
    username="alice",
    full_name="Alice Smith",
    address=Address(
        street="123 Main St",
        city="Bangkok",
        postal_code="10110"
    ),
    contact=ContactInfo(
        email="alice@example.com",
        phone="+66812345678"
    )
)

# Access nested fields
print(profile.address.city)     # "Bangkok"
print(profile.contact.email)    # "alice@example.com"

# Serialize
print(profile.model_dump())
print(profile.model_dump_json(indent=2))
```

### Recursive Models

```python
from pydantic import BaseModel
from typing import Optional, List

class Category(BaseModel):
    """Category model ที่มี subcategories (recursive)"""
    id: int
    name: str
    description: Optional[str] = None
    parent_id: Optional[int] = None
    subcategories: List['Category'] = []  # Self-referential
    
    class Config:
        # Pydantic v2
        pass

# Update forward references
Category.model_rebuild()

# สร้าง nested categories
electronics = Category(
    id=1,
    name="Electronics",
    subcategories=[
        Category(
            id=2,
            name="Laptops",
            parent_id=1,
            subcategories=[
                Category(id=3, name="Gaming Laptops", parent_id=2),
                Category(id=4, name="Business Laptops", parent_id=2),
            ]
        ),
        Category(id=5, name="Phones", parent_id=1)
    ]
)

print(electronics.subcategories[0].name)  # "Laptops"
```

### Union Types

```python
from pydantic import BaseModel
from typing import Union, Literal
from datetime import datetime

class TextContent(BaseModel):
    type: Literal["text"] = "text"
    content: str
    font_size: int = 16

class ImageContent(BaseModel):
    type: Literal["image"] = "image"
    url: str
    alt_text: str = ""
    width: Optional[int] = None
    height: Optional[int] = None

class VideoContent(BaseModel):
    type: Literal["video"] = "video"
    url: str
    duration_seconds: int
    thumbnail_url: Optional[str] = None

# Union ของหลาย content types
ContentBlock = Union[TextContent, ImageContent, VideoContent]

class BlogPost(BaseModel):
    id: int
    title: str
    content_blocks: List[ContentBlock]  # ใช้ Union

# สร้าง post ที่มี mixed content
post = BlogPost(
    id=1,
    title="My Blog Post",
    content_blocks=[
        TextContent(content="Hello, world!", font_size=18),
        ImageContent(url="https://example.com/image.jpg", alt_text="Sample image"),
        VideoContent(url="https://youtube.com/watch?v=xxx", duration_seconds=300),
        TextContent(content="Conclusion paragraph."),
    ]
)

for block in post.content_blocks:
    print(f"Type: {block.type}, Fields: {block.model_dump()}")
```

---

## 5. Optional Fields

### Optional vs Required

```python
from pydantic import BaseModel, Field
from typing import Optional, Union

class UpdateUserRequest(BaseModel):
    """
    สำหรับ PATCH requests - ทุก field เป็น Optional
    แต่ถ้า field ถูกส่งมา ต้องผ่าน validation
    """
    
    # Optional ด้วย None default
    full_name: Optional[str] = Field(default=None, min_length=2, max_length=200)
    
    # Optional field ที่ไม่มี default (explicit None)
    bio: Optional[str] = None
    
    # Optional ด้วย Union[type, None] (เหมือน Optional[type])
    age: Union[int, None] = Field(default=None, ge=0, le=150)
    
    # จะ omit ถ้า field ไม่ได้ส่งมา
    tags: Optional[list] = None

# exclude_unset - ไม่รวม fields ที่ไม่ได้ set
update = UpdateUserRequest(full_name="Alice Smith")
print(update.model_dump())              # {'full_name': 'Alice Smith', 'bio': None, 'age': None, 'tags': None}
print(update.model_dump(exclude_unset=True))  # {'full_name': 'Alice Smith'}
print(update.model_dump(exclude_none=True))   # {'full_name': 'Alice Smith'}
```

### Partial Updates Pattern

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# Database
users = {
    1: {"id": 1, "name": "Alice", "email": "alice@example.com", "bio": "Developer", "age": 30}
}

class UserFull(BaseModel):
    """Full user model - ใช้สำหรับ create (PUT)"""
    name: str
    email: str
    bio: Optional[str] = None
    age: Optional[int] = None

class UserPartial(BaseModel):
    """Partial model - ใช้สำหรับ update (PATCH)"""
    name: Optional[str] = None
    email: Optional[str] = None
    bio: Optional[str] = None
    age: Optional[int] = None

@app.put("/users/{user_id}")
async def replace_user(user_id: int, data: UserFull):
    """PUT: Replace user ทั้งหมด"""
    if user_id not in users:
        raise HTTPException(404, "User not found")
    
    users[user_id] = {"id": user_id, **data.model_dump()}
    return users[user_id]

@app.patch("/users/{user_id}")
async def update_user(user_id: int, data: UserPartial):
    """PATCH: Update บางส่วน"""
    if user_id not in users:
        raise HTTPException(404, "User not found")
    
    current = users[user_id].copy()
    
    # อัพเดทเฉพาะ fields ที่ส่งมา (exclude_unset=True)
    updates = data.model_dump(exclude_unset=True)
    current.update(updates)
    
    users[user_id] = current
    return current
```

---

## 6. Response Models

### Response Model Filtering

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: str
    password: str  # sensitive - ไม่ควรส่งกลับ

class UserDB(UserCreate):
    id: int
    password_hash: str  # sensitive
    created_at: datetime
    is_verified: bool = False

class UserPublic(BaseModel):
    """Public response - ไม่มี sensitive fields"""
    id: int
    username: str
    email: str
    created_at: datetime
    is_verified: bool

class UserAdmin(UserPublic):
    """Admin response - เพิ่ม internal info"""
    # เพิ่ม fields ที่ admin ควรเห็น
    pass

# Mock DB
users_store: dict = {}

@app.post("/users", response_model=UserPublic, status_code=201)
async def create_user(user_data: UserCreate):
    """สร้าง user - return UserPublic (ไม่มี password)"""
    import hashlib
    db_user = UserDB(
        id=len(users_store) + 1,
        username=user_data.username,
        email=user_data.email,
        password=user_data.password,
        password_hash=hashlib.sha256(user_data.password.encode()).hexdigest(),
        created_at=datetime.utcnow()
    )
    users_store[db_user.id] = db_user.model_dump()
    return db_user  # FastAPI filter เป็น UserPublic
```

### Generic Response Models

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional, List

T = TypeVar('T')

class PaginatedResponse(BaseModel, Generic[T]):
    """Generic paginated response"""
    items: List[T]
    total: int
    page: int
    per_page: int
    total_pages: int
    has_next: bool
    has_prev: bool

class APIResponse(BaseModel, Generic[T]):
    """Generic API response"""
    success: bool = True
    data: Optional[T] = None
    message: str = "Success"
    timestamp: datetime = Field(default_factory=datetime.utcnow)

# ใช้ใน endpoint
@app.get("/users", response_model=APIResponse[List[UserPublic]])
async def list_users():
    users = [{"id": 1, "username": "alice", "email": "a@example.com",
              "created_at": datetime.utcnow(), "is_verified": True}]
    return APIResponse(data=users)
```

---

## 7. APIRouter

### สร้าง Router แยก Module

```python
# routers/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

# สร้าง router ด้วย prefix และ tags
router = APIRouter(
    prefix="/users",
    tags=["users"],
    responses={
        401: {"description": "Unauthorized"},
        403: {"description": "Forbidden"},
    }
)

# Models
class UserCreate(BaseModel):
    username: str
    email: str

class UserResponse(BaseModel):
    id: int
    username: str
    email: str

# Mock DB
users_db = {}

@router.get("", response_model=List[UserResponse])
async def list_users():
    """GET /users - ดึง users ทั้งหมด"""
    return list(users_db.values())

@router.post("", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    """POST /users - สร้าง user ใหม่"""
    user_id = len(users_db) + 1
    new_user = {"id": user_id, **user.model_dump()}
    users_db[user_id] = new_user
    return new_user

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(user_id: int):
    """GET /users/{user_id}"""
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail=f"User {user_id} not found")
    return users_db[user_id]

@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: int):
    """DELETE /users/{user_id}"""
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail=f"User {user_id} not found")
    del users_db[user_id]
    return None
```

```python
# routers/products.py
from fastapi import APIRouter
from pydantic import BaseModel
from typing import Optional, List

router = APIRouter(
    prefix="/products",
    tags=["products"]
)

class ProductCreate(BaseModel):
    name: str
    price: float
    category: Optional[str] = None

class ProductResponse(ProductCreate):
    id: int

products_db = {}

@router.get("", response_model=List[ProductResponse])
async def list_products():
    return list(products_db.values())

@router.post("", response_model=ProductResponse, status_code=201)
async def create_product(product: ProductCreate):
    product_id = len(products_db) + 1
    new_product = {"id": product_id, **product.model_dump()}
    products_db[product_id] = new_product
    return new_product

@router.get("/{product_id}", response_model=ProductResponse)
async def get_product(product_id: int):
    from fastapi import HTTPException
    if product_id not in products_db:
        raise HTTPException(404, f"Product {product_id} not found")
    return products_db[product_id]
```

---

## 8. Tags

### การใช้ Tags สำหรับ Documentation

```python
from fastapi import FastAPI, APIRouter
from fastapi.openapi.utils import get_openapi

# กำหนด tag metadata สำหรับ documentation
tags_metadata = [
    {
        "name": "users",
        "description": "User management operations",
        "externalDocs": {
            "description": "User management docs",
            "url": "https://docs.example.com/users",
        },
    },
    {
        "name": "products",
        "description": "Product catalog operations",
    },
    {
        "name": "orders",
        "description": "Order management",
    },
    {
        "name": "auth",
        "description": "Authentication and authorization",
    },
]

app = FastAPI(
    title="E-Commerce API",
    openapi_tags=tags_metadata  # ใส่ tag metadata
)

# Router ด้วย tags
users_router = APIRouter(prefix="/users", tags=["users"])
products_router = APIRouter(prefix="/products", tags=["products"])

@users_router.get("")
async def list_users():
    return {"users": []}

@products_router.get("")
async def list_products():
    return {"products": []}

# endpoint ที่มีหลาย tags
@app.get("/search", tags=["users", "products"])
async def global_search(q: str):
    """ค้นหาทั้ง users และ products"""
    return {"results": []}

app.include_router(users_router)
app.include_router(products_router)
```

---

## 9. Include Router

### Application Factory Pattern

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

def create_app() -> FastAPI:
    """Application factory"""
    app = FastAPI(
        title="My API",
        version="1.0.0"
    )
    
    # Register middleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_methods=["*"],
        allow_headers=["*"],
    )
    
    # Register routers
    from .routers import users, products, orders, auth
    
    app.include_router(
        auth.router,
        prefix="/api/v1/auth",
        tags=["auth"]
    )
    
    app.include_router(
        users.router,
        prefix="/api/v1/users",
        tags=["users"],
        dependencies=[]  # Global dependencies สำหรับ router นี้
    )
    
    app.include_router(
        products.router,
        prefix="/api/v1/products",
        tags=["products"]
    )
    
    app.include_router(
        orders.router,
        prefix="/api/v1/orders",
        tags=["orders"]
    )
    
    return app

app = create_app()
```

### Nested Routers

```python
from fastapi import APIRouter

# Main router
api_router = APIRouter(prefix="/api")

# Version routers
v1_router = APIRouter(prefix="/v1")
v2_router = APIRouter(prefix="/v2")

# Feature routers
users_v1 = APIRouter(prefix="/users", tags=["users-v1"])
users_v2 = APIRouter(prefix="/users", tags=["users-v2"])

@users_v1.get("")
async def list_users_v1():
    """API v1 users"""
    return {"version": 1, "users": []}

@users_v2.get("")
async def list_users_v2():
    """API v2 users - with pagination"""
    return {"version": 2, "users": [], "pagination": {}}

# Assemble routers
v1_router.include_router(users_v1)
v2_router.include_router(users_v2)
api_router.include_router(v1_router)
api_router.include_router(v2_router)

from fastapi import FastAPI
app = FastAPI()
app.include_router(api_router)

# ผลลัพธ์:
# GET /api/v1/users -> list_users_v1
# GET /api/v2/users -> list_users_v2
```

### Router Dependencies

```python
from fastapi import APIRouter, Depends, HTTPException, Header

async def verify_token(x_token: str = Header(...)):
    """Common authentication dependency"""
    if x_token != "valid-token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return {"token": x_token}

# Protected router ที่ require authentication
protected_router = APIRouter(
    prefix="/protected",
    tags=["protected"],
    dependencies=[Depends(verify_token)]  # ทุก endpoint ใน router ต้อง authenticate
)

@protected_router.get("/data")
async def get_protected_data():
    """ต้อง authenticate เพื่อเข้าถึง"""
    return {"data": "secret information"}

@protected_router.get("/settings")
async def get_settings():
    """ต้อง authenticate เพื่อเข้าถึง"""
    return {"settings": {}}
```

---

## 10. CORS Middleware

### CORS Configuration

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Development: อนุญาตทุก origin
if False:  # ไม่แนะนำสำหรับ production
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

# Production: จำกัด origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://myapp.com",
        "https://app.myapp.com",
        "https://admin.myapp.com",
        "http://localhost:3000",      # Development
        "http://localhost:5173",      # Vite dev server
    ],
    allow_credentials=True,  # อนุญาต cookies
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
    allow_headers=[
        "Accept",
        "Accept-Language",
        "Content-Language",
        "Content-Type",
        "Authorization",
        "X-API-Key",
        "X-Request-ID",
    ],
    expose_headers=[
        "X-Total-Count",
        "X-Request-ID",
        "X-RateLimit-Limit",
        "X-RateLimit-Remaining",
        "X-RateLimit-Reset",
    ],
    max_age=600,  # Preflight cache 10 minutes
)

@app.get("/api/data")
async def get_data():
    return {"data": "This endpoint supports CORS"}
```

### Dynamic CORS

```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.responses import Response

app = FastAPI()

ALLOWED_ORIGINS = {
    "production": ["https://myapp.com", "https://app.myapp.com"],
    "staging": ["https://staging.myapp.com"],
    "development": ["http://localhost:3000", "http://localhost:5173"]
}

class DynamicCORSMiddleware(BaseHTTPMiddleware):
    """Dynamic CORS middleware ที่จัดการ origins ตาม environment"""
    
    async def dispatch(self, request: Request, call_next):
        origin = request.headers.get("origin", "")
        
        # ตรวจสอบว่า origin อนุญาตหรือไม่
        all_allowed = []
        for origins in ALLOWED_ORIGINS.values():
            all_allowed.extend(origins)
        
        response = await call_next(request)
        
        if origin in all_allowed:
            response.headers["Access-Control-Allow-Origin"] = origin
            response.headers["Access-Control-Allow-Credentials"] = "true"
            response.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE, PATCH, OPTIONS"
            response.headers["Access-Control-Allow-Headers"] = "Content-Type, Authorization"
            response.headers["Vary"] = "Origin"  # Important for caching
        
        return response

# app.add_middleware(DynamicCORSMiddleware)
```

---

## 11. Custom Middleware

### Logging Middleware

```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from starlette.responses import Response
import time
import uuid
import logging
import json

app = FastAPI()
logger = logging.getLogger("api")

class RequestLoggingMiddleware(BaseHTTPMiddleware):
    """Middleware สำหรับ log ทุก request/response"""
    
    async def dispatch(self, request: Request, call_next):
        # Generate request ID
        request_id = str(uuid.uuid4())[:8]
        
        # Start timing
        start_time = time.time()
        
        # Log incoming request
        logger.info(
            f"[{request_id}] {request.method} {request.url.path} "
            f"from {request.client.host if request.client else 'unknown'}"
        )
        
        # Process request
        try:
            response = await call_next(request)
        except Exception as e:
            logger.error(f"[{request_id}] Error: {str(e)}")
            raise
        
        # Calculate duration
        duration = (time.time() - start_time) * 1000  # ms
        
        # Log response
        logger.info(
            f"[{request_id}] {response.status_code} "
            f"({duration:.2f}ms)"
        )
        
        # เพิ่ม headers
        response.headers["X-Request-ID"] = request_id
        response.headers["X-Response-Time"] = f"{duration:.2f}ms"
        
        return response

app.add_middleware(RequestLoggingMiddleware)
```

### Rate Limiting Middleware

```python
import time
from collections import defaultdict
from fastapi.responses import JSONResponse

class RateLimitMiddleware(BaseHTTPMiddleware):
    """Simple in-memory rate limiting middleware"""
    
    def __init__(self, app, requests_per_minute: int = 60):
        super().__init__(app)
        self.requests_per_minute = requests_per_minute
        self.requests = defaultdict(list)
    
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host if request.client else "unknown"
        
        # Clean old requests
        now = time.time()
        minute_ago = now - 60
        self.requests[client_ip] = [
            req_time for req_time in self.requests[client_ip]
            if req_time > minute_ago
        ]
        
        # Check rate limit
        if len(self.requests[client_ip]) >= self.requests_per_minute:
            return JSONResponse(
                status_code=429,
                content={
                    "error": "RATE_LIMIT_EXCEEDED",
                    "message": "Too many requests",
                    "retry_after": 60
                },
                headers={"Retry-After": "60"}
            )
        
        # Record request
        self.requests[client_ip].append(now)
        
        response = await call_next(request)
        
        # Add rate limit headers
        remaining = self.requests_per_minute - len(self.requests[client_ip])
        response.headers["X-RateLimit-Limit"] = str(self.requests_per_minute)
        response.headers["X-RateLimit-Remaining"] = str(remaining)
        response.headers["X-RateLimit-Reset"] = str(int(now + 60))
        
        return response

app.add_middleware(RateLimitMiddleware, requests_per_minute=100)
```

### Security Headers Middleware

```python
class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    """เพิ่ม security headers ให้ทุก response"""
    
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        
        # Security headers
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = (
            "default-src 'self'; "
            "script-src 'self' 'unsafe-inline'; "
            "style-src 'self' 'unsafe-inline';"
        )
        
        return response

app.add_middleware(SecurityHeadersMiddleware)
```

---

## 12. BackgroundTasks

### Basic Background Tasks

```python
from fastapi import FastAPI, BackgroundTasks
import asyncio
import logging

app = FastAPI()
logger = logging.getLogger(__name__)

# Background task functions
async def send_welcome_email(email: str, username: str):
    """ส่ง welcome email (async)"""
    logger.info(f"Sending welcome email to {email}")
    await asyncio.sleep(2)  # Simulate email sending
    logger.info(f"Welcome email sent to {email}")

async def update_analytics(event: str, user_id: int):
    """อัพเดท analytics data"""
    logger.info(f"Updating analytics: {event} for user {user_id}")
    await asyncio.sleep(0.5)
    logger.info("Analytics updated")

def generate_thumbnail(image_path: str):
    """Generate thumbnail (sync)"""
    import time
    time.sleep(1)  # Simulate thumbnail generation
    logger.info(f"Thumbnail generated for {image_path}")

@app.post("/users/register")
async def register_user(
    user_data: dict,
    background_tasks: BackgroundTasks
):
    """
    ลงทะเบียน user และทำ background tasks:
    1. ส่ง welcome email
    2. อัพเดท analytics
    """
    # Create user (main operation)
    user = {
        "id": 1,
        "username": user_data.get("username"),
        "email": user_data.get("email")
    }
    
    # Schedule background tasks (ทำหลัง response ถูกส่ง)
    background_tasks.add_task(
        send_welcome_email,
        email=user["email"],
        username=user["username"]
    )
    
    background_tasks.add_task(
        update_analytics,
        event="user_registered",
        user_id=user["id"]
    )
    
    return {
        "user": user,
        "message": "Registration successful! Welcome email will be sent shortly."
    }

@app.post("/images/upload")
async def upload_image(
    image_data: dict,
    background_tasks: BackgroundTasks
):
    """Upload image และสร้าง thumbnail ใน background"""
    image_path = f"/uploads/{image_data.get('filename', 'image.jpg')}"
    
    # ทำ thumbnail generation ใน background
    background_tasks.add_task(generate_thumbnail, image_path)
    
    return {
        "image_path": image_path,
        "message": "Image uploaded. Thumbnail will be generated shortly."
    }
```

### Background Tasks กับ Dependency

```python
from fastapi import Depends

class EmailService:
    """Email service dependency"""
    
    async def send_email(self, to: str, subject: str, body: str):
        logger.info(f"Sending email to {to}: {subject}")
        await asyncio.sleep(1)
        logger.info(f"Email sent to {to}")
    
    async def send_verification_email(self, email: str, token: str):
        body = f"Click to verify: https://example.com/verify?token={token}"
        await self.send_email(email, "Verify your email", body)

def get_email_service():
    return EmailService()

@app.post("/users/verify-email")
async def request_email_verification(
    user_id: int,
    background_tasks: BackgroundTasks,
    email_service: EmailService = Depends(get_email_service)
):
    """ส่ง verification email ใน background"""
    import secrets
    token = secrets.token_urlsafe(32)
    
    background_tasks.add_task(
        email_service.send_verification_email,
        email=f"user{user_id}@example.com",
        token=token
    )
    
    return {"message": "Verification email will be sent shortly"}
```

---

## 13. Request/Response Lifecycle

### Lifespan Events (FastAPI v0.93+)

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
import asyncio

# Database connection pool (simulated)
db_pool = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Manage application lifespan"""
    # Startup
    print("Starting up application...")
    
    # Initialize resources
    global db_pool
    db_pool = {"status": "connected", "pool_size": 10}
    print("Database pool created")
    
    # Initialize cache
    print("Cache initialized")
    
    yield  # Application runs here
    
    # Shutdown
    print("Shutting down application...")
    db_pool = None
    print("Database pool closed")
    print("Cache cleared")

app = FastAPI(lifespan=lifespan)

@app.get("/db-status")
async def db_status():
    return {"db_pool": db_pool}
```

### Request/Response Hooks

```python
from fastapi import FastAPI, Request, Response
import time

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    """
    Middleware สำหรับ request/response processing
    รันสำหรับทุก request
    """
    # Before processing request
    start_time = time.time()
    
    # Add request ID
    request_id = f"req-{int(start_time * 1000)}"
    
    # Process request
    response = await call_next(request)
    
    # After processing response
    process_time = time.time() - start_time
    
    # Add headers to response
    response.headers["X-Process-Time"] = str(process_time)
    response.headers["X-Request-ID"] = request_id
    
    return response

# Startup and shutdown events (deprecated in newer FastAPI - ใช้ lifespan แทน)
@app.on_event("startup")
async def startup():
    print("Application started")

@app.on_event("shutdown")
async def shutdown():
    print("Application shutting down")
```

### Request Context

```python
from fastapi import FastAPI, Request
from typing import Optional
import contextvars

app = FastAPI()

# Context variable สำหรับเก็บ request context
request_id_var: contextvars.ContextVar[Optional[str]] = contextvars.ContextVar(
    'request_id', default=None
)

@app.middleware("http")
async def request_context_middleware(request: Request, call_next):
    """เก็บ request context ใน ContextVar"""
    import uuid
    request_id = str(uuid.uuid4())
    
    # Set context variable
    token = request_id_var.set(request_id)
    
    try:
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response
    finally:
        # Reset context variable
        request_id_var.reset(token)

def get_current_request_id() -> Optional[str]:
    """ดึง request ID จาก context"""
    return request_id_var.get()

@app.get("/test")
async def test_context():
    request_id = get_current_request_id()
    return {"request_id": request_id}
```

---

## 14. Webhook Handlers

### Basic Webhook Handler

```python
from fastapi import FastAPI, Request, Header, HTTPException
from pydantic import BaseModel
from typing import Optional, Any, Dict
import hmac
import hashlib
import json

app = FastAPI()

# === GitHub Webhook ===
GITHUB_SECRET = "your-webhook-secret"

class GitHubEvent(BaseModel):
    action: Optional[str] = None
    repository: Optional[Dict[str, Any]] = None
    sender: Optional[Dict[str, Any]] = None

def verify_github_signature(payload_body: bytes, signature_header: str, secret: str) -> bool:
    """ตรวจสอบ GitHub webhook signature"""
    if not signature_header:
        return False
    
    sha_name, signature = signature_header.split('=')
    if sha_name != 'sha256':
        return False
    
    mac = hmac.new(secret.encode(), msg=payload_body, digestmod=hashlib.sha256)
    return hmac.compare_digest(mac.hexdigest(), signature)

@app.post("/webhooks/github")
async def github_webhook(
    request: Request,
    x_github_event: str = Header(...),
    x_hub_signature_256: Optional[str] = Header(None)
):
    """รับ webhook events จาก GitHub"""
    
    # Get raw body สำหรับ signature verification
    body = await request.body()
    
    # Verify signature
    if x_hub_signature_256:
        if not verify_github_signature(body, x_hub_signature_256, GITHUB_SECRET):
            raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Parse payload
    payload = json.loads(body)
    
    # Handle different events
    if x_github_event == "push":
        repo = payload.get("repository", {})
        commits = payload.get("commits", [])
        print(f"Push to {repo.get('full_name')}: {len(commits)} commits")
        
    elif x_github_event == "pull_request":
        action = payload.get("action")
        pr = payload.get("pull_request", {})
        print(f"PR {action}: {pr.get('title')}")
        
    elif x_github_event == "issues":
        action = payload.get("action")
        issue = payload.get("issue", {})
        print(f"Issue {action}: {issue.get('title')}")
    
    return {"status": "processed", "event": x_github_event}
```

### Stripe Payment Webhook

```python
from fastapi import FastAPI, Request, Header, HTTPException
import stripe  # pip install stripe
import json

app = FastAPI()

STRIPE_WEBHOOK_SECRET = "whsec_your_secret_here"

@app.post("/webhooks/stripe")
async def stripe_webhook(
    request: Request,
    stripe_signature: str = Header(..., alias="stripe-signature")
):
    """รับ webhook events จาก Stripe"""
    payload = await request.body()
    
    # Verify Stripe signature
    try:
        event = stripe.Webhook.construct_event(
            payload, stripe_signature, STRIPE_WEBHOOK_SECRET
        )
    except ValueError:
        raise HTTPException(status_code=400, detail="Invalid payload")
    except stripe.error.SignatureVerificationError:
        raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Handle events
    event_type = event["type"]
    
    if event_type == "payment_intent.succeeded":
        payment_intent = event["data"]["object"]
        amount = payment_intent["amount"] / 100  # Convert cents to dollars
        print(f"Payment succeeded: ${amount:.2f}")
        # Update order status in DB
        
    elif event_type == "payment_intent.payment_failed":
        payment_intent = event["data"]["object"]
        print(f"Payment failed: {payment_intent['last_payment_error']}")
        # Notify customer
        
    elif event_type == "customer.subscription.created":
        subscription = event["data"]["object"]
        print(f"New subscription: {subscription['id']}")
        # Activate subscription
    
    return {"status": "success", "event_type": event_type}
```

### Generic Webhook Handler

```python
from fastapi import FastAPI, Request, Header
from pydantic import BaseModel
from typing import Any, Dict, Optional
from datetime import datetime
import asyncio

app = FastAPI()

# Webhook event log
webhook_events = []

class WebhookEvent(BaseModel):
    id: str
    source: str
    event_type: str
    data: Dict[str, Any]
    received_at: datetime

@app.post("/webhooks/{source}")
async def generic_webhook(
    source: str,
    request: Request,
    x_event_type: Optional[str] = Header(None)
):
    """Generic webhook handler ที่รับได้จากหลาย sources"""
    
    body = await request.body()
    
    try:
        import json
        data = json.loads(body)
    except json.JSONDecodeError:
        data = {"raw": body.decode()}
    
    event = {
        "id": f"evt_{int(datetime.utcnow().timestamp())}",
        "source": source,
        "event_type": x_event_type or data.get("type", "unknown"),
        "data": data,
        "received_at": datetime.utcnow().isoformat()
    }
    
    webhook_events.append(event)
    
    # Process asynchronously
    asyncio.create_task(process_webhook_event(event))
    
    return {"status": "received", "event_id": event["id"]}

async def process_webhook_event(event: dict):
    """Process webhook event asynchronously"""
    print(f"Processing webhook: {event['source']} / {event['event_type']}")
    await asyncio.sleep(0.5)  # Simulate processing
    print(f"Webhook processed: {event['id']}")

@app.get("/webhooks/events")
async def list_webhook_events():
    """ดู webhook events ที่ได้รับ"""
    return {"events": webhook_events[-50:]}  # Last 50 events
```

---

## 15. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Pydantic Models

**โจทย์**: สร้าง Pydantic model สำหรับ recipe management ที่มี validation ครบถ้วน

**เฉลย**:

```python
from pydantic import BaseModel, Field, field_validator, model_validator
from typing import Optional, List
from enum import Enum

class Difficulty(str, Enum):
    easy = "easy"
    medium = "medium"
    hard = "hard"

class Ingredient(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    amount: float = Field(..., gt=0)
    unit: str = Field(..., max_length=20)

class Recipe(BaseModel):
    title: str = Field(..., min_length=5, max_length=200)
    description: str = Field(..., min_length=10)
    ingredients: List[Ingredient] = Field(..., min_length=1)
    instructions: List[str] = Field(..., min_length=1)
    prep_time: int = Field(..., ge=1, description="Minutes")
    cook_time: int = Field(..., ge=0, description="Minutes")
    servings: int = Field(..., ge=1, le=100)
    difficulty: Difficulty = Difficulty.medium
    tags: List[str] = Field(default=[], max_length=10)
    
    total_time: Optional[int] = None
    
    @field_validator('title')
    @classmethod
    def title_must_be_capitalized(cls, v):
        return v.strip().title()
    
    @field_validator('tags')
    @classmethod
    def tags_lowercase(cls, v):
        return [tag.lower().strip() for tag in v]
    
    @model_validator(mode='after')
    def calculate_total_time(self):
        self.total_time = self.prep_time + self.cook_time
        return self

# Test
recipe = Recipe(
    title="spaghetti bolognese",
    description="Classic Italian pasta dish with meat sauce",
    ingredients=[
        Ingredient(name="Spaghetti", amount=200, unit="g"),
        Ingredient(name="Ground beef", amount=300, unit="g"),
    ],
    instructions=["Cook pasta", "Brown beef", "Mix together"],
    prep_time=15,
    cook_time=30,
    servings=4,
    tags=["Italian", "Pasta", "EASY"]
)
print(recipe.title)      # "Spaghetti Bolognese"
print(recipe.total_time) # 45
print(recipe.tags)       # ['italian', 'pasta', 'easy']
```

### แบบฝึกหัดที่ 2: Custom Validators

**โจทย์**: สร้าง validator สำหรับ Thai ID card number (เลขบัตรประชาชน)

**เฉลย**:

```python
from pydantic import BaseModel, field_validator
from typing import Optional

class ThaiIDValidation(BaseModel):
    id_number: str
    
    @field_validator('id_number')
    @classmethod
    def validate_thai_id(cls, v: str) -> str:
        """Validate Thai National ID number"""
        # ลบ dashes และ spaces
        v = v.replace('-', '').replace(' ', '')
        
        # ต้องมี 13 หลัก
        if len(v) != 13:
            raise ValueError('Thai ID must be 13 digits')
        
        # ต้องเป็นตัวเลขทั้งหมด
        if not v.isdigit():
            raise ValueError('Thai ID must contain only digits')
        
        # ไม่เริ่มต้นด้วย 0
        if v[0] == '0':
            raise ValueError('Thai ID cannot start with 0')
        
        # Checksum validation (MOD 11)
        total = sum(int(v[i]) * (13 - i) for i in range(12))
        check_digit = (11 - (total % 11)) % 10
        
        if check_digit != int(v[12]):
            raise ValueError('Invalid Thai ID checksum')
        
        return v

# Valid Thai ID (format: X-XXXX-XXXXX-XX-X)
# Note: For testing, use a real valid ID format
```

### แบบฝึกหัดที่ 3: Nested Models

**โจทย์**: สร้าง models สำหรับ E-commerce order system

**เฉลย**:

```python
from pydantic import BaseModel, Field, model_validator
from typing import List, Optional
from datetime import datetime
from enum import Enum

class OrderStatus(str, Enum):
    pending = "pending"
    confirmed = "confirmed"
    shipped = "shipped"
    delivered = "delivered"
    cancelled = "cancelled"

class Address(BaseModel):
    recipient_name: str
    street: str
    city: str
    state: str
    country: str = "Thailand"
    postal_code: str

class OrderItem(BaseModel):
    product_id: int
    product_name: str
    quantity: int = Field(..., ge=1)
    unit_price: float = Field(..., gt=0)
    discount_percent: float = Field(default=0, ge=0, le=100)
    
    @property
    def subtotal(self) -> float:
        return self.quantity * self.unit_price * (1 - self.discount_percent / 100)

class Order(BaseModel):
    id: Optional[int] = None
    customer_id: int
    items: List[OrderItem] = Field(..., min_length=1)
    shipping_address: Address
    billing_address: Optional[Address] = None
    status: OrderStatus = OrderStatus.pending
    notes: Optional[str] = None
    
    created_at: datetime = Field(default_factory=datetime.utcnow)
    total_amount: Optional[float] = None
    
    @model_validator(mode='after')
    def set_billing_address(self):
        if self.billing_address is None:
            self.billing_address = self.shipping_address
        return self
    
    @model_validator(mode='after')
    def calculate_total(self):
        self.total_amount = sum(item.subtotal for item in self.items)
        return self

# Create order
order = Order(
    customer_id=1,
    items=[
        OrderItem(
            product_id=1,
            product_name="Laptop",
            quantity=1,
            unit_price=29999.0,
            discount_percent=10
        ),
        OrderItem(
            product_id=2,
            product_name="Mouse",
            quantity=2,
            unit_price=999.0
        )
    ],
    shipping_address=Address(
        recipient_name="Alice",
        street="123 Main St",
        city="Bangkok",
        state="Bangkok",
        postal_code="10110"
    )
)

print(f"Total: {order.total_amount:.2f}")  # 26999.1 + 1998.0 = 28997.1
```

### แบบฝึกหัดที่ 4: APIRouter Organization

**โจทย์**: แบ่ง Blog API ออกเป็นหลาย routers

**เฉลย**:

```python
# routers/posts.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from typing import Optional, List

router = APIRouter(prefix="/posts", tags=["posts"])

class PostCreate(BaseModel):
    title: str
    content: str
    published: bool = False

posts_db = {}

@router.get("", response_model=List[dict])
async def list_posts():
    return list(posts_db.values())

@router.post("", status_code=201)
async def create_post(post: PostCreate):
    post_id = len(posts_db) + 1
    new_post = {"id": post_id, **post.model_dump()}
    posts_db[post_id] = new_post
    return new_post

@router.get("/{post_id}")
async def get_post(post_id: int):
    if post_id not in posts_db:
        raise HTTPException(404, "Post not found")
    return posts_db[post_id]

# routers/comments.py
from fastapi import APIRouter

comments_router = APIRouter(prefix="/posts/{post_id}/comments", tags=["comments"])
comments_db = {}

@comments_router.get("")
async def list_comments(post_id: int):
    return [c for c in comments_db.values() if c["post_id"] == post_id]

@comments_router.post("", status_code=201)
async def create_comment(post_id: int, text: str):
    comment_id = len(comments_db) + 1
    comment = {"id": comment_id, "post_id": post_id, "text": text}
    comments_db[comment_id] = comment
    return comment

# main.py
from fastapi import FastAPI

app = FastAPI(title="Blog API")

# Import and include routers
app.include_router(router)  # /posts
app.include_router(comments_router)  # /posts/{post_id}/comments
```

### แบบฝึกหัดที่ 5: Custom Middleware

**โจทย์**: สร้าง middleware ที่ตรวจสอบ API version ใน header

**เฉลย**:

```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.responses import JSONResponse

app = FastAPI()

SUPPORTED_API_VERSIONS = ["1.0", "2.0", "2.1"]
DEFAULT_VERSION = "2.0"

class APIVersionMiddleware(BaseHTTPMiddleware):
    """Middleware ตรวจสอบ API version จาก header"""
    
    async def dispatch(self, request: Request, call_next):
        # Skip version check สำหรับ docs endpoints
        if request.url.path in ["/docs", "/redoc", "/openapi.json"]:
            return await call_next(request)
        
        # Check Accept-Version header
        requested_version = request.headers.get("Accept-Version", DEFAULT_VERSION)
        
        if requested_version not in SUPPORTED_API_VERSIONS:
            return JSONResponse(
                status_code=400,
                content={
                    "error": "UNSUPPORTED_API_VERSION",
                    "message": f"Version {requested_version} is not supported",
                    "supported_versions": SUPPORTED_API_VERSIONS
                }
            )
        
        response = await call_next(request)
        response.headers["API-Version"] = requested_version
        return response

app.add_middleware(APIVersionMiddleware)

@app.get("/data")
async def get_data(request: Request):
    version = request.headers.get("Accept-Version", DEFAULT_VERSION)
    
    if version == "1.0":
        return {"data": "v1 format"}
    elif version in ["2.0", "2.1"]:
        return {"data": "v2 format", "version": version}
```

### แบบฝึกหัดที่ 6: BackgroundTasks

**โจทย์**: สร้าง order processing system ที่ส่ง notifications ใน background

**เฉลย**:

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from typing import List
import asyncio
import logging

app = FastAPI()
logger = logging.getLogger(__name__)

class OrderCreate(BaseModel):
    customer_email: str
    items: List[dict]
    total_amount: float

async def send_order_confirmation(email: str, order_id: int, total: float):
    """ส่ง order confirmation email"""
    await asyncio.sleep(1)  # Simulate sending
    logger.info(f"Order confirmation sent to {email} for order #{order_id}")

async def update_inventory(items: list):
    """อัพเดท inventory"""
    await asyncio.sleep(0.5)
    for item in items:
        logger.info(f"Inventory updated for item: {item}")

async def notify_warehouse(order_id: int, items: list):
    """แจ้ง warehouse ให้จัดสินค้า"""
    await asyncio.sleep(0.3)
    logger.info(f"Warehouse notified for order #{order_id}")

async def generate_invoice(order_id: int):
    """สร้าง invoice PDF"""
    await asyncio.sleep(2)
    logger.info(f"Invoice generated for order #{order_id}")

@app.post("/orders", status_code=201)
async def create_order(
    order_data: OrderCreate,
    background_tasks: BackgroundTasks
):
    """สร้าง order และทำ background tasks"""
    order_id = 1001
    
    # Main task: สร้าง order ใน DB
    order = {
        "id": order_id,
        "customer_email": order_data.customer_email,
        "items": order_data.items,
        "total": order_data.total_amount,
        "status": "confirmed"
    }
    
    # Background tasks
    background_tasks.add_task(
        send_order_confirmation,
        order_data.customer_email, order_id, order_data.total_amount
    )
    background_tasks.add_task(update_inventory, order_data.items)
    background_tasks.add_task(notify_warehouse, order_id, order_data.items)
    background_tasks.add_task(generate_invoice, order_id)
    
    return {
        "order": order,
        "message": "Order created! Confirmation email will be sent shortly."
    }
```

### แบบฝึกหัดที่ 7: Request Lifecycle

**โจทย์**: สร้าง middleware chain ที่ทำ: logging, auth, rate limiting

**เฉลย**:

```python
from fastapi import FastAPI, Request
from fastapi.middleware.base import BaseHTTPMiddleware
from fastapi.responses import JSONResponse
import time
import logging
from collections import defaultdict

app = FastAPI()
logger = logging.getLogger(__name__)
request_counts = defaultdict(list)

# 1. Logging Middleware (outermost - รันก่อน)
class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.time()
        logger.info(f"→ {request.method} {request.url.path}")
        response = await call_next(request)
        duration = (time.time() - start) * 1000
        logger.info(f"← {response.status_code} ({duration:.1f}ms)")
        response.headers["X-Duration"] = f"{duration:.1f}ms"
        return response

# 2. Rate Limiting Middleware
class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, limit=30):
        super().__init__(app)
        self.limit = limit
    
    async def dispatch(self, request: Request, call_next):
        ip = getattr(request.client, 'host', 'unknown')
        now = time.time()
        
        # Clean old entries
        request_counts[ip] = [t for t in request_counts[ip] if now - t < 60]
        
        if len(request_counts[ip]) >= self.limit:
            return JSONResponse(
                status_code=429,
                content={"error": "Rate limit exceeded", "retry_after": 60}
            )
        
        request_counts[ip].append(now)
        return await call_next(request)

# Add middleware (last added runs first)
app.add_middleware(RateLimitMiddleware, limit=30)
app.add_middleware(LoggingMiddleware)

# Order of execution:
# Request → LoggingMiddleware → RateLimitMiddleware → endpoint
# Response ← LoggingMiddleware ← RateLimitMiddleware ← endpoint
```

### แบบฝึกหัดที่ 8: Complete Validated API

**โจทย์**: สร้าง complete Product API ด้วย validation, router, และ middleware ครบถ้วน

**เฉลย**:

```python
from fastapi import FastAPI, APIRouter, HTTPException, BackgroundTasks, Query, Path
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.base import BaseHTTPMiddleware
from pydantic import BaseModel, Field, field_validator
from typing import Optional, List
from datetime import datetime, timezone
from enum import Enum
import uuid
import time

# ======= Models =======
class Category(str, Enum):
    electronics = "electronics"
    books = "books"
    clothing = "clothing"

class ProductCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=200)
    description: Optional[str] = Field(None, max_length=2000)
    price: float = Field(..., gt=0)
    category: Category
    stock: int = Field(default=0, ge=0)
    tags: List[str] = []
    
    @field_validator('name')
    @classmethod
    def name_title_case(cls, v):
        return v.strip().title()
    
    @field_validator('tags')
    @classmethod
    def tags_lowercase(cls, v):
        return [t.lower().strip() for t in v][:10]

class ProductResponse(ProductCreate):
    id: str
    created_at: datetime
    updated_at: datetime

# ======= Router =======
router = APIRouter(prefix="/products", tags=["products"])
products_db = {}

@router.get("", response_model=List[ProductResponse])
async def list_products(
    q: Optional[str] = Query(None),
    category: Optional[Category] = Query(None),
    min_price: Optional[float] = Query(None, gt=0),
    max_price: Optional[float] = Query(None, gt=0),
    page: int = Query(1, ge=1),
    per_page: int = Query(10, ge=1, le=50),
):
    items = list(products_db.values())
    if q:
        items = [p for p in items if q.lower() in p["name"].lower()]
    if category:
        items = [p for p in items if p["category"] == category]
    if min_price:
        items = [p for p in items if p["price"] >= min_price]
    if max_price:
        items = [p for p in items if p["price"] <= max_price]
    start = (page - 1) * per_page
    return items[start:start + per_page]

@router.post("", response_model=ProductResponse, status_code=201)
async def create_product(product: ProductCreate, background_tasks: BackgroundTasks):
    now = datetime.now(timezone.utc)
    new_product = {
        "id": str(uuid.uuid4()),
        **product.model_dump(),
        "created_at": now,
        "updated_at": now,
    }
    products_db[new_product["id"]] = new_product
    background_tasks.add_task(lambda: print(f"Product created: {new_product['name']}"))
    return new_product

@router.get("/{product_id}", response_model=ProductResponse)
async def get_product(product_id: str = Path(...)):
    product = products_db.get(product_id)
    if not product:
        raise HTTPException(404, "Product not found")
    return product

# ======= App =======
app = FastAPI(title="Product API v2")
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])
app.include_router(router, prefix="/api/v1")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, reload=True)
```

---

## สรุป

ใน Part 58 เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียนรู้ |
|--------|-------------------|
| Pydantic v2 | BaseModel, model_config, ConfigDict |
| Field Validation | Built-in validators, constraints |
| Custom Validators | field_validator, model_validator |
| Nested Models | Nested objects, recursive, Union types |
| Optional Fields | Optional fields, partial updates |
| Response Models | Filtering, Generic responses |
| APIRouter | Modular routing, prefix, tags |
| Tags | Swagger documentation grouping |
| Include Router | Application factory, nested routers |
| CORS | Configuration, dynamic CORS |
| Custom Middleware | Logging, rate limiting, security |
| BackgroundTasks | Async operations after response |
| Request Lifecycle | Middleware chain, context |
| Webhooks | GitHub, Stripe, generic handlers |

### ขั้นตอนต่อไป

- **Part 59**: FastAPI - Database & Async ORM (SQLAlchemy async, Alembic)
- **Part 60**: FastAPI - Authentication, JWT & OAuth2
