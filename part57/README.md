# Part 57: FastAPI - Getting Started

## สารบัญ

1. [FastAPI vs Flask vs Django](#1-fastapi-vs-flask-vs-django)
2. [Installation และ Setup](#2-installation-และ-setup)
3. [First FastAPI App](#3-first-fastapi-app)
4. [Path Operations](#4-path-operations)
5. [Automatic Documentation](#5-automatic-documentation)
6. [Path Parameters](#6-path-parameters)
7. [Query Parameters](#7-query-parameters)
8. [Request Body](#8-request-body)
9. [Response Models](#9-response-models)
10. [HTTP Status Codes](#10-http-status-codes)
11. [Async Handlers](#11-async-handlers)
12. [Dependency Injection เบื้องต้น](#12-dependency-injection-เบื้องต้น)
13. [ตัวอย่างโปรแกรมจริง: Items API](#13-ตัวอย่างโปรแกรมจริง-items-api)
14. [ตัวอย่างโปรแกรมจริง: User Management API](#14-ตัวอย่างโปรแกรมจริง-user-management-api)
15. [แบบฝึกหัด](#15-แบบฝึกหัด)

---

## 1. FastAPI vs Flask vs Django

### ตารางเปรียบเทียบ

| Feature | FastAPI | Flask | Django |
|---------|---------|-------|--------|
| **Performance** | ⭐⭐⭐⭐⭐ (ASGI) | ⭐⭐⭐ (WSGI) | ⭐⭐⭐ (WSGI) |
| **Async Support** | Native async/await | Limited | Limited |
| **Auto Documentation** | Swagger + ReDoc | Manual/Extension | Manual/Extension |
| **Type Hints** | First-class | Optional | Optional |
| **Validation** | Pydantic (built-in) | Manual/WTForms | Django Forms |
| **Learning Curve** | Medium | Low | High |
| **Batteries Included** | Partial | Minimal | Full |
| **ORM** | 3rd party | 3rd party | Built-in (Django ORM) |
| **Auth** | 3rd party | 3rd party | Built-in |
| **Admin Panel** | 3rd party | 3rd party | Built-in |
| **Best For** | APIs, Microservices | Small APIs, Prototypes | Full-stack web apps |

### เมื่อไหรควรใช้อะไร

```
FastAPI  → REST APIs, Microservices, High-performance APIs, Real-time apps
Flask    → Simple APIs, Prototyping, Learning, Small projects
Django   → Full-stack web applications, Content management systems
```

### Performance Benchmark

```python
# FastAPI ใช้ Starlette framework ด้านล่าง
# รองรับ ASGI (Asynchronous Server Gateway Interface)
# สามารถ handle concurrent requests ได้ดีกว่า WSGI

# Flask: synchronous, 1 request = 1 thread
# FastAPI: asynchronous, 1 event loop handles many requests

# ตัวอย่าง: 1000 concurrent requests
# Flask: ต้องการ 1000 threads (memory intensive)
# FastAPI: ใช้ event loop ที่มีประสิทธิภาพมากกว่า

import asyncio
import time

# FastAPI style - async handler
async def fetch_from_db():
    await asyncio.sleep(0.1)  # Simulate I/O
    return {'data': '...'}

async def handle_many_requests():
    # สามารถ handle หลาย requests พร้อมกัน
    tasks = [fetch_from_db() for _ in range(100)]
    results = await asyncio.gather(*tasks)
    return results

# ผลลัพธ์: ~0.1 วินาที แทนที่จะเป็น 10 วินาที (sequential)
```

---

## 2. Installation และ Setup

### การติดตั้ง

```bash
# ติดตั้ง FastAPI และ Uvicorn (ASGI server)
pip install fastapi uvicorn[standard]

# ติดตั้งเพิ่มเติมสำหรับ development
pip install fastapi uvicorn[standard] python-multipart

# สร้าง virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows

# ติดตั้ง requirements
pip install -r requirements.txt
```

### Project Structure แนะนำ

```
my_fastapi_app/
├── main.py              # Entry point
├── app/
│   ├── __init__.py
│   ├── main.py          # FastAPI app instance
│   ├── api/
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── endpoints/
│   │   │   │   ├── users.py
│   │   │   │   ├── items.py
│   │   │   │   └── auth.py
│   │   │   └── router.py
│   ├── core/
│   │   ├── config.py    # Settings
│   │   └── security.py  # Auth utilities
│   ├── models/
│   │   ├── user.py
│   │   └── item.py
│   ├── schemas/
│   │   ├── user.py      # Pydantic schemas
│   │   └── item.py
│   └── db/
│       ├── database.py
│       └── crud.py
└── tests/
    ├── test_users.py
    └── test_items.py
```

### Configuration

```python
# app/core/config.py
from pydantic_settings import BaseSettings
from typing import Optional, List

class Settings(BaseSettings):
    """Application settings จาก environment variables"""
    
    # App
    APP_NAME: str = "My FastAPI App"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    
    # API
    API_V1_PREFIX: str = "/api/v1"
    
    # Database
    DATABASE_URL: str = "sqlite:///./app.db"
    
    # JWT
    SECRET_KEY: str = "your-secret-key"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    # CORS
    ALLOWED_ORIGINS: List[str] = ["http://localhost:3000"]
    
    class Config:
        env_file = ".env"

settings = Settings()

# ใช้งาน:
# from app.core.config import settings
# print(settings.APP_NAME)
```

---

## 3. First FastAPI App

### Hello World

```python
# main.py
from fastapi import FastAPI

# สร้าง FastAPI instance
app = FastAPI(
    title="My First FastAPI App",
    description="ตัวอย่าง FastAPI แรกของเรา",
    version="1.0.0"
)

@app.get("/")
async def root():
    """
    Root endpoint - ส่ง greeting message กลับมา
    
    FastAPI จะ:
    1. รับ GET request ที่ /
    2. เรียก function นี้
    3. แปลง dict เป็น JSON อัตโนมัติ
    """
    return {"message": "Hello, FastAPI!"}

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {"status": "healthy", "version": "1.0.0"}

# รัน: uvicorn main:app --reload
# หรือ: python -m uvicorn main:app --reload
```

### รัน application

```bash
# Development mode (auto-reload เมื่อ code เปลี่ยน)
uvicorn main:app --reload

# หรือกำหนด host และ port
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Production mode
uvicorn main:app --workers 4 --host 0.0.0.0 --port 8000
```

### หลังจากรัน

```
# เข้าถึงได้ที่:
http://localhost:8000          -> API endpoints
http://localhost:8000/docs     -> Swagger UI (interactive)
http://localhost:8000/redoc    -> ReDoc (readable docs)
http://localhost:8000/openapi.json -> OpenAPI schema JSON
```

---

## 4. Path Operations

### HTTP Methods

```python
from fastapi import FastAPI, HTTPException
from typing import Optional

app = FastAPI()

# In-memory database
items_db = {
    1: {"id": 1, "name": "Laptop", "price": 999.99, "in_stock": True},
    2: {"id": 2, "name": "Mouse", "price": 29.99, "in_stock": True},
    3: {"id": 3, "name": "Keyboard", "price": 49.99, "in_stock": False},
}
next_id = 4

# GET - ดึงข้อมูล
@app.get("/items")
async def read_items():
    """GET /items - ดึง items ทั้งหมด"""
    return {"items": list(items_db.values())}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    """GET /items/{item_id} - ดึง item เฉพาะ"""
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    return items_db[item_id]

# POST - สร้างข้อมูลใหม่
@app.post("/items", status_code=201)
async def create_item(item: dict):
    """POST /items - สร้าง item ใหม่"""
    global next_id
    new_item = {"id": next_id, **item}
    items_db[next_id] = new_item
    next_id += 1
    return new_item

# PUT - อัพเดทข้อมูลทั้งหมด
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: dict):
    """PUT /items/{item_id} - Replace item"""
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    updated = {"id": item_id, **item}
    items_db[item_id] = updated
    return updated

# PATCH - อัพเดทบางส่วน
@app.patch("/items/{item_id}")
async def partial_update_item(item_id: int, item: dict):
    """PATCH /items/{item_id} - Partial update"""
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    
    current = items_db[item_id]
    current.update(item)
    return current

# DELETE - ลบข้อมูล
@app.delete("/items/{item_id}", status_code=204)
async def delete_item(item_id: int):
    """DELETE /items/{item_id} - ลบ item"""
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    del items_db[item_id]
    return None  # 204 No Content
```

### Decorator Parameters

```python
from fastapi import FastAPI
from typing import Set, Optional

app = FastAPI()

@app.get(
    "/products",
    tags=["products"],                     # Swagger UI grouping
    summary="Get all products",            # Short description
    description="Retrieve all products with optional filtering",  # Long description
    response_description="List of products",  # Response description
    deprecated=False,                      # Mark as deprecated
)
async def get_products():
    """
    Retrieve all products.
    
    **Features:**
    - Pagination support
    - Filter by category
    - Sort by price or name
    """
    return {"products": []}

@app.get(
    "/products/{product_id}",
    tags=["products"],
    responses={
        200: {"description": "Product found"},
        404: {"description": "Product not found"},
    }
)
async def get_product(product_id: int):
    return {"id": product_id, "name": "Sample Product"}
```

---

## 5. Automatic Documentation

### Swagger UI (Interactive)

FastAPI สร้าง documentation อัตโนมัติจาก:
- Type hints
- Pydantic models
- Function docstrings
- Decorator parameters

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional

app = FastAPI(
    title="Product API",
    description="""
## Product Management API

API สำหรับจัดการ products ใน online store

### Features
- **CRUD Operations**: Create, Read, Update, Delete products
- **Search & Filter**: Advanced filtering and search
- **Authentication**: JWT-based authentication
    """,
    version="2.0.0",
    terms_of_service="https://example.com/terms",
    contact={
        "name": "API Support",
        "url": "https://example.com/contact",
        "email": "support@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
)

class ProductCreate(BaseModel):
    name: str = Field(
        ...,
        title="Product Name",
        description="ชื่อสินค้า",
        example="Gaming Laptop",
        min_length=1,
        max_length=200
    )
    price: float = Field(
        ...,
        title="Price",
        description="ราคา (บาท)",
        example=29999.00,
        gt=0
    )
    category: Optional[str] = Field(
        None,
        title="Category",
        description="หมวดหมู่สินค้า",
        example="electronics"
    )
    
    class Config:
        json_schema_extra = {
            "example": {
                "name": "Gaming Laptop",
                "price": 29999.00,
                "category": "electronics"
            }
        }

@app.post(
    "/products",
    response_model=ProductCreate,
    status_code=201,
    tags=["products"],
    summary="Create a new product",
    description="สร้างสินค้าใหม่ในระบบ",
)
async def create_product(product: ProductCreate):
    """
    สร้างสินค้าใหม่
    
    - **name**: ชื่อสินค้า (จำเป็น)
    - **price**: ราคาสินค้า (ต้องมากกว่า 0)
    - **category**: หมวดหมู่ (ไม่จำเป็น)
    """
    return product
```

### Custom OpenAPI Schema

```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI()

def custom_openapi():
    """Customize OpenAPI schema"""
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title="Custom API",
        version="1.0.0",
        description="API with custom documentation",
        routes=app.routes,
    )
    
    # เพิ่ม custom security scheme
    openapi_schema["components"]["securitySchemes"] = {
        "APIKeyHeader": {
            "type": "apiKey",
            "in": "header",
            "name": "X-API-Key"
        },
        "BearerAuth": {
            "type": "http",
            "scheme": "bearer",
            "bearerFormat": "JWT"
        }
    }
    
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

---

## 6. Path Parameters

### Basic Path Parameters

```python
from fastapi import FastAPI, Path
from enum import Enum

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """
    FastAPI จะ:
    1. แปลง {user_id} จาก string เป็น int อัตโนมัติ
    2. Validate ว่าเป็น integer
    3. Return 422 ถ้าไม่ใช่ integer
    """
    return {"user_id": user_id, "type": type(user_id).__name__}

@app.get("/files/{file_path:path}")
async def get_file(file_path: str):
    """
    :path ทำให้ path parameter รับ / ได้
    เช่น /files/data/2024/report.pdf
    """
    return {"file_path": file_path}
```

### Path Parameters with Validation

```python
from fastapi import Path, HTTPException

@app.get("/users/{user_id}/profile")
async def get_user_profile(
    user_id: int = Path(
        ...,           # Required
        title="User ID",
        description="ID ของ user ที่ต้องการ",
        ge=1,          # Greater than or equal to 1
        le=9999999,    # Less than or equal to
        example=42
    )
):
    """ดึง profile ของ user พร้อม validation"""
    if user_id > 1000:
        raise HTTPException(status_code=404, detail="User not found")
    
    return {
        "user_id": user_id,
        "name": f"User {user_id}",
        "email": f"user{user_id}@example.com"
    }

@app.get("/products/{product_id}")
async def get_product_validated(
    product_id: int = Path(
        ...,
        ge=1,
        description="Product ID (must be positive integer)"
    )
):
    return {"product_id": product_id}
```

### Enum Path Parameters

```python
class Category(str, Enum):
    """Categories ที่รองรับ"""
    electronics = "electronics"
    books = "books"
    clothing = "clothing"
    food = "food"

@app.get("/products/category/{category}")
async def get_by_category(category: Category):
    """
    FastAPI จะ:
    1. Validate ว่า category เป็น value ที่ถูกต้อง
    2. แสดง options ใน Swagger UI
    3. Return 422 ถ้าไม่ถูกต้อง
    """
    return {
        "category": category,
        "category_value": category.value,
        "products": [f"Product in {category.value}"]
    }
```

### Multiple Path Parameters

```python
@app.get("/users/{user_id}/orders/{order_id}/items/{item_id}")
async def get_order_item(
    user_id: int = Path(..., ge=1, description="User ID"),
    order_id: int = Path(..., ge=1, description="Order ID"),
    item_id: int = Path(..., ge=1, description="Item ID"),
):
    """Nested resources ด้วย multiple path parameters"""
    return {
        "user_id": user_id,
        "order_id": order_id,
        "item_id": item_id,
    }
```

---

## 7. Query Parameters

### Basic Query Parameters

```python
from fastapi import FastAPI, Query
from typing import Optional, List

app = FastAPI()

@app.get("/items")
async def read_items(
    skip: int = 0,       # Optional ด้วย default value
    limit: int = 10,     # Optional ด้วย default value
    active: bool = True  # Boolean query param
):
    """
    GET /items?skip=0&limit=10&active=true
    
    FastAPI แปลง types อัตโนมัติ:
    - ?active=true  -> True
    - ?active=false -> False
    - ?active=1     -> True
    - ?active=0     -> False
    """
    items = [{"id": i, "active": active} for i in range(skip, skip + limit)]
    return {"items": items, "skip": skip, "limit": limit}

@app.get("/products")
async def search_products(
    q: Optional[str] = None,   # Optional query param
    category: Optional[str] = None,
    min_price: Optional[float] = None,
    max_price: Optional[float] = None,
):
    """ค้นหา products ด้วย query parameters"""
    return {
        "query": q,
        "category": category,
        "price_range": [min_price, max_price]
    }
```

### Query Parameters with Validation

```python
@app.get("/search")
async def search(
    q: str = Query(
        ...,              # Required
        title="Search Query",
        description="คำค้นหา",
        min_length=2,     # Minimum length
        max_length=100,   # Maximum length
        example="python book"
    ),
    page: int = Query(
        default=1,
        ge=1,             # Must be >= 1
        description="Page number"
    ),
    per_page: int = Query(
        default=10,
        ge=1,             # Must be >= 1
        le=100,           # Must be <= 100
        description="Items per page"
    ),
    sort: str = Query(
        default="created_at",
        regex="^(created_at|name|price|rating)$",  # Regex validation
        description="Sort field"
    ),
    order: str = Query(
        default="desc",
        regex="^(asc|desc)$",
        description="Sort order"
    ),
):
    """Advanced search ด้วย validated query parameters"""
    return {
        "query": q,
        "pagination": {"page": page, "per_page": per_page},
        "sorting": {"sort": sort, "order": order}
    }
```

### List Query Parameters

```python
@app.get("/items/by-tags")
async def get_items_by_tags(
    tags: List[str] = Query(
        default=[],
        description="Filter by tags (ส่งหลายครั้งได้)"
    )
):
    """
    รับ multiple values สำหรับ query param เดียว:
    GET /items/by-tags?tags=python&tags=fastapi&tags=api
    """
    return {
        "tags": tags,
        "example_url": "/items/by-tags?tags=python&tags=fastapi"
    }

@app.get("/users/batch")
async def get_users_batch(
    ids: List[int] = Query(
        ...,
        description="User IDs ที่ต้องการ (หลาย IDs)"
    )
):
    """GET /users/batch?ids=1&ids=2&ids=3"""
    return {"user_ids": ids}
```

### Path + Query Parameters

```python
@app.get("/users/{user_id}/posts")
async def get_user_posts(
    user_id: int = Path(..., ge=1, description="User ID"),
    page: int = Query(default=1, ge=1),
    per_page: int = Query(default=10, ge=1, le=50),
    published: Optional[bool] = Query(default=None),
):
    """รวม path parameter กับ query parameters"""
    return {
        "user_id": user_id,
        "filters": {
            "published": published
        },
        "pagination": {
            "page": page,
            "per_page": per_page
        }
    }
```

---

## 8. Request Body

### Basic Request Body

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

app = FastAPI()

class Item(BaseModel):
    """Pydantic model สำหรับ item"""
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

@app.post("/items")
async def create_item(item: Item):
    """
    FastAPI จะ:
    1. อ่าน request body เป็น JSON
    2. Validate ด้วย Pydantic
    3. แปลงเป็น Item instance
    4. Return 422 ถ้า validation ล้มเหลว
    """
    item_dict = item.dict()
    
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    
    return item_dict
```

### Complex Request Body

```python
class Address(BaseModel):
    street: str
    city: str
    country: str
    postal_code: str

class UserCreate(BaseModel):
    """Complex model ด้วย nested model"""
    username: str = Field(..., min_length=3, max_length=50)
    email: str = Field(..., description="Email address")
    password: str = Field(..., min_length=8)
    full_name: Optional[str] = Field(None, max_length=200)
    age: Optional[int] = Field(None, ge=0, le=150)
    address: Optional[Address] = None
    tags: list[str] = []
    
    class Config:
        json_schema_extra = {
            "example": {
                "username": "john_doe",
                "email": "john@example.com",
                "password": "securepassword123",
                "full_name": "John Doe",
                "age": 30,
                "address": {
                    "street": "123 Main St",
                    "city": "Bangkok",
                    "country": "Thailand",
                    "postal_code": "10110"
                },
                "tags": ["admin", "premium"]
            }
        }

@app.post("/users", status_code=201)
async def create_user(user: UserCreate):
    """สร้าง user ใหม่ด้วย complex nested data"""
    # In real app: hash password, save to DB
    user_dict = user.dict()
    user_dict['id'] = 100
    user_dict.pop('password')  # ไม่ส่ง password กลับ
    return user_dict
```

### Path + Query + Body

```python
@app.put("/users/{user_id}")
async def update_user(
    user_id: int,                          # Path parameter
    update_data: UserCreate,               # Request body
    notify: bool = Query(default=False),   # Query parameter
):
    """รวม path, query, และ body parameters"""
    return {
        "user_id": user_id,
        "updated_data": update_data.dict(),
        "notify_user": notify
    }
```

### Multiple Bodies

```python
class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str
    email: str

from fastapi import Body

@app.post("/order")
async def create_order(
    item: Item,
    user: User,
    discount: float = Body(default=0.0, ge=0.0, le=1.0),
):
    """
    รับ multiple body objects:
    {
        "item": {"name": "Laptop", "price": 999.99},
        "user": {"username": "john", "email": "john@example.com"},
        "discount": 0.1
    }
    """
    total = item.price * (1 - discount)
    return {
        "item": item.dict(),
        "user": user.dict(),
        "discount": discount,
        "total": total
    }
```

---

## 9. Response Models

### กำหนด Response Model

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: str
    password: str  # ต้องการเป็น input
    full_name: Optional[str] = None

class UserResponse(BaseModel):
    """Response model - ไม่มี password"""
    id: int
    username: str
    email: str
    full_name: Optional[str] = None
    is_active: bool = True

@app.post(
    "/users",
    response_model=UserResponse,  # FastAPI จะ filter ตาม model นี้
    status_code=201,
)
async def create_user(user: UserCreate):
    """
    FastAPI จะ:
    1. รับ UserCreate (มี password)
    2. Process
    3. Filter response ด้วย UserResponse (ไม่มี password)
    """
    # In real app: hash password, save to DB
    db_user = {
        "id": 1,
        "username": user.username,
        "email": user.email,
        "full_name": user.full_name,
        "password": "hashed_password",  # จะถูก filter ออก
        "is_active": True
    }
    return db_user  # FastAPI จะ filter เป็น UserResponse อัตโนมัติ
```

### Response Model Options

```python
@app.get(
    "/users",
    response_model=List[UserResponse],           # List of responses
    response_model_exclude_unset=True,           # ไม่รวม fields ที่ไม่ได้ set
    response_model_include={"id", "username"},   # รวมเฉพาะ fields เหล่านี้
    # หรือ:
    # response_model_exclude={"password", "email"},  # ไม่รวม fields เหล่านี้
)
async def get_users():
    return [
        {"id": 1, "username": "alice", "email": "alice@example.com", "is_active": True},
        {"id": 2, "username": "bob", "email": "bob@example.com", "is_active": False},
    ]
```

### Multiple Response Types

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse, HTMLResponse, PlainTextResponse
from pydantic import BaseModel
from typing import Union

app = FastAPI()

class ItemResponse(BaseModel):
    id: int
    name: str

class ErrorResponse(BaseModel):
    error: str
    detail: str

@app.get(
    "/items/{item_id}",
    responses={
        200: {
            "model": ItemResponse,
            "description": "Item found"
        },
        404: {
            "model": ErrorResponse,
            "description": "Item not found"
        }
    }
)
async def get_item(item_id: int):
    if item_id > 100:
        return JSONResponse(
            status_code=404,
            content={"error": "NOT_FOUND", "detail": f"Item {item_id} not found"}
        )
    return ItemResponse(id=item_id, name=f"Item {item_id}")
```

---

## 10. HTTP Status Codes

### ใช้ Status Codes ใน FastAPI

```python
from fastapi import FastAPI, status
from fastapi.responses import Response

app = FastAPI()

@app.post("/items", status_code=status.HTTP_201_CREATED)
async def create_item(item: dict):
    """201 Created"""
    return {"id": 1, **item}

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_item(item_id: int):
    """204 No Content - ไม่มี response body"""
    return None  # หรือ return Response(status_code=204)

@app.get("/redirect-me")
async def redirect():
    """301 Redirect"""
    from fastapi.responses import RedirectResponse
    return RedirectResponse(url="/new-location", status_code=301)

# ใช้ HTTPException
from fastapi import HTTPException

@app.get("/protected")
async def protected_route():
    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Authentication required",
        headers={"WWW-Authenticate": "Bearer"},
    )
```

### Custom Exception Handler

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request: Request, exc: StarletteHTTPException):
    """Custom handler สำหรับ HTTP exceptions"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error": {
                "code": f"HTTP_{exc.status_code}",
                "message": exc.detail,
                "status_code": exc.status_code
            }
        }
    )

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Custom handler สำหรับ validation errors"""
    errors = []
    for error in exc.errors():
        errors.append({
            "field": " -> ".join(str(loc) for loc in error["loc"]),
            "message": error["msg"],
            "type": error["type"]
        })
    
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error": {
                "code": "VALIDATION_ERROR",
                "message": "Request validation failed",
                "details": errors
            }
        }
    )

# Custom exception class
class AppError(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"success": False, "error": {"code": exc.code, "message": exc.message}}
    )

@app.get("/test-error")
async def test_error():
    raise AppError("BUSINESS_ERROR", "Something went wrong", 400)
```

---

## 11. Async Handlers

### เมื่อไหรใช้ async?

```python
import asyncio
import aiohttp
from fastapi import FastAPI

app = FastAPI()

# ✅ ใช้ async เมื่อ:
# 1. Database operations (SQLAlchemy async)
# 2. HTTP requests ไปยัง external services
# 3. File I/O operations
# 4. Waiting for multiple operations พร้อมกัน

@app.get("/async-example")
async def async_example():
    """
    Async handler - ดีที่สุดสำหรับ I/O bound operations
    """
    # Simulate async database query
    await asyncio.sleep(0.1)  # ไม่ block event loop
    
    # Simulate async HTTP request
    # async with aiohttp.ClientSession() as session:
    #     async with session.get('https://api.example.com') as resp:
    #         data = await resp.json()
    
    return {"result": "async operation completed"}

@app.get("/sync-example")
def sync_example():
    """
    Sync handler - FastAPI จะรันใน thread pool
    ใช้สำหรับ CPU-bound operations หรือ blocking I/O
    """
    import time
    time.sleep(0.1)  # Blocking operation - จะรันใน separate thread
    return {"result": "sync operation completed"}

# Concurrent Operations
@app.get("/concurrent")
async def concurrent_example():
    """รัน multiple async operations พร้อมกัน"""
    
    async def fetch_user():
        await asyncio.sleep(0.1)
        return {"id": 1, "name": "Alice"}
    
    async def fetch_orders():
        await asyncio.sleep(0.15)
        return [{"id": 1, "total": 99.99}]
    
    async def fetch_products():
        await asyncio.sleep(0.2)
        return [{"id": 1, "name": "Laptop"}]
    
    # รัน concurrent - ใช้เวลา ~0.2 วินาที (ไม่ใช่ 0.45 วินาที)
    user, orders, products = await asyncio.gather(
        fetch_user(),
        fetch_orders(),
        fetch_products()
    )
    
    return {
        "user": user,
        "orders": orders,
        "products": products
    }
```

### Background Tasks

```python
from fastapi import FastAPI, BackgroundTasks
import asyncio

app = FastAPI()

async def send_notification(email: str, message: str):
    """Background task - ส่ง notification"""
    print(f"Sending notification to {email}: {message}")
    await asyncio.sleep(2)  # Simulate sending email
    print(f"Notification sent to {email}")

async def process_data(data: dict):
    """Background task - process data"""
    await asyncio.sleep(5)  # Long processing
    print(f"Data processed: {data}")

@app.post("/users/{user_id}/activate")
async def activate_user(user_id: int, background_tasks: BackgroundTasks):
    """
    Activate user และส่ง notification ใน background
    Response กลับทันที ไม่ต้องรอ background task
    """
    # Do main work
    user = {"id": user_id, "status": "active"}
    
    # Schedule background tasks
    background_tasks.add_task(
        send_notification,
        email=f"user{user_id}@example.com",
        message="Your account has been activated!"
    )
    
    background_tasks.add_task(
        process_data,
        data={"user_id": user_id, "action": "activation"}
    )
    
    return {"user": user, "message": "User activated, notification will be sent shortly"}
```

---

## 12. Dependency Injection เบื้องต้น

### Dependencies คืออะไร

Dependency Injection (DI) ช่วยให้ reuse code ได้ง่าย เช่น authentication, database sessions, logging

```python
from fastapi import FastAPI, Depends, HTTPException
from typing import Optional

app = FastAPI()

# Simple dependency
def get_db():
    """Dependency สำหรับ database connection"""
    # In real app: create DB session
    db = {"connection": "fake_db_connection"}
    try:
        yield db  # yield เพื่อ cleanup หลัง request
    finally:
        # Cleanup: close DB connection
        pass

# Authentication dependency
def get_current_user(
    api_key: str = Depends(lambda: "dummy"),  # ตัวอย่าง
):
    """Dependency สำหรับ authentication"""
    pass

# Query parameters dependency
class CommonQueryParams:
    """Reusable query parameters"""
    def __init__(
        self,
        skip: int = 0,
        limit: int = 10,
        search: Optional[str] = None
    ):
        self.skip = skip
        self.limit = limit
        self.search = search

@app.get("/items")
async def get_items(
    commons: CommonQueryParams = Depends(CommonQueryParams),
    db = Depends(get_db)
):
    """ใช้ dependencies"""
    items = list(range(commons.skip, commons.skip + commons.limit))
    return {
        "items": items,
        "skip": commons.skip,
        "limit": commons.limit,
        "search": commons.search,
        "db_status": db
    }
```

### API Key Authentication Dependency

```python
from fastapi import Header

API_KEYS = {"key-1": "user1", "key-2": "user2"}

async def verify_api_key(x_api_key: str = Header(...)):
    """Dependency: ตรวจสอบ API key"""
    if x_api_key not in API_KEYS:
        raise HTTPException(
            status_code=401,
            detail="Invalid API key"
        )
    return {"username": API_KEYS[x_api_key], "api_key": x_api_key}

@app.get("/protected/items")
async def get_protected_items(
    current_user: dict = Depends(verify_api_key)
):
    """Protected endpoint ที่ต้องการ API key"""
    return {
        "items": ["item1", "item2"],
        "requested_by": current_user["username"]
    }
```

### Dependency Chains

```python
async def get_db():
    return {"db": "connected"}

async def get_user_service(db=Depends(get_db)):
    """Service ที่ depend on database"""
    return {"service": "user_service", "db": db}

async def get_current_user(
    api_key: str = Header(...),
    user_service=Depends(get_user_service)
):
    """Auth dependency ที่ depend on user_service"""
    if api_key not in API_KEYS:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return {"username": API_KEYS[api_key]}

@app.get("/me")
async def get_my_profile(user: dict = Depends(get_current_user)):
    """Endpoint ที่ depend on chain of dependencies"""
    return {"profile": user}
```

---

## 13. ตัวอย่างโปรแกรมจริง: Items API

```python
# items_api.py - Complete Items Management API
from fastapi import FastAPI, HTTPException, Query, Path, Depends, status
from fastapi.responses import JSONResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime, timezone
import uuid

app = FastAPI(
    title="Items Management API",
    description="API สำหรับจัดการ items",
    version="1.0.0"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ==================== Schemas ====================
class ItemBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=200, description="ชื่อ item")
    description: Optional[str] = Field(None, max_length=1000)
    price: float = Field(..., gt=0, description="ราคา")
    category: Optional[str] = Field(None, max_length=50)
    tags: List[str] = Field(default=[], description="Tags สำหรับ item")
    in_stock: bool = Field(default=True)

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    """Partial update - ทุก field เป็น Optional"""
    name: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = None
    price: Optional[float] = Field(None, gt=0)
    category: Optional[str] = None
    tags: Optional[List[str]] = None
    in_stock: Optional[bool] = None

class ItemResponse(ItemBase):
    id: str
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True

class PaginatedItemsResponse(BaseModel):
    items: List[ItemResponse]
    total: int
    page: int
    per_page: int
    total_pages: int
    has_next: bool
    has_prev: bool

# ==================== Database (In-Memory) ====================
items_db: dict = {}

def get_all_items() -> list:
    return list(items_db.values())

def get_item_by_id(item_id: str) -> Optional[dict]:
    return items_db.get(item_id)

# ==================== Common Parameters ====================
class PaginationParams:
    def __init__(
        self,
        page: int = Query(default=1, ge=1, description="Page number"),
        per_page: int = Query(default=10, ge=1, le=100, description="Items per page"),
    ):
        self.page = page
        self.per_page = per_page

class SearchParams:
    def __init__(
        self,
        q: Optional[str] = Query(default=None, description="Search query"),
        category: Optional[str] = Query(default=None, description="Filter by category"),
        min_price: Optional[float] = Query(default=None, ge=0),
        max_price: Optional[float] = Query(default=None, ge=0),
        in_stock: Optional[bool] = Query(default=None),
        sort: str = Query(default="created_at", regex="^(name|price|created_at|category)$"),
        order: str = Query(default="desc", regex="^(asc|desc)$"),
    ):
        self.q = q
        self.category = category
        self.min_price = min_price
        self.max_price = max_price
        self.in_stock = in_stock
        self.sort = sort
        self.order = order

# ==================== Endpoints ====================
@app.get("/", tags=["root"])
async def root():
    return {
        "message": "Items API",
        "version": "1.0.0",
        "docs": "/docs"
    }

@app.get(
    "/items",
    response_model=PaginatedItemsResponse,
    tags=["items"],
    summary="Get all items"
)
async def list_items(
    pagination: PaginationParams = Depends(PaginationParams),
    search: SearchParams = Depends(SearchParams),
):
    """
    ดึง items ทั้งหมดพร้อม pagination, filtering, และ sorting
    """
    items = get_all_items()
    
    # Filtering
    if search.q:
        q_lower = search.q.lower()
        items = [
            i for i in items
            if q_lower in i["name"].lower() or
               (i.get("description") and q_lower in i["description"].lower())
        ]
    
    if search.category:
        items = [i for i in items if i.get("category") == search.category]
    
    if search.min_price is not None:
        items = [i for i in items if i["price"] >= search.min_price]
    
    if search.max_price is not None:
        items = [i for i in items if i["price"] <= search.max_price]
    
    if search.in_stock is not None:
        items = [i for i in items if i["in_stock"] == search.in_stock]
    
    # Sorting
    reverse = search.order == "desc"
    items.sort(key=lambda x: x.get(search.sort, ""), reverse=reverse)
    
    # Pagination
    total = len(items)
    total_pages = max((total + pagination.per_page - 1) // pagination.per_page, 1)
    start = (pagination.page - 1) * pagination.per_page
    end = start + pagination.per_page
    page_items = items[start:end]
    
    return PaginatedItemsResponse(
        items=page_items,
        total=total,
        page=pagination.page,
        per_page=pagination.per_page,
        total_pages=total_pages,
        has_next=pagination.page < total_pages,
        has_prev=pagination.page > 1
    )

@app.post(
    "/items",
    response_model=ItemResponse,
    status_code=status.HTTP_201_CREATED,
    tags=["items"],
    summary="Create a new item"
)
async def create_item(item: ItemCreate):
    """สร้าง item ใหม่"""
    now = datetime.now(timezone.utc)
    new_item = {
        "id": str(uuid.uuid4()),
        **item.dict(),
        "created_at": now,
        "updated_at": now,
    }
    items_db[new_item["id"]] = new_item
    return new_item

@app.get(
    "/items/{item_id}",
    response_model=ItemResponse,
    tags=["items"],
    responses={
        404: {"description": "Item not found"}
    }
)
async def get_item(
    item_id: str = Path(..., description="Item ID (UUID)")
):
    """ดึง item เฉพาะตาม ID"""
    item = get_item_by_id(item_id)
    if not item:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found"
        )
    return item

@app.put(
    "/items/{item_id}",
    response_model=ItemResponse,
    tags=["items"]
)
async def replace_item(
    item_id: str,
    item: ItemCreate
):
    """Replace item ทั้งหมด (PUT)"""
    existing = get_item_by_id(item_id)
    if not existing:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    
    updated = {
        **existing,
        **item.dict(),
        "updated_at": datetime.now(timezone.utc)
    }
    items_db[item_id] = updated
    return updated

@app.patch(
    "/items/{item_id}",
    response_model=ItemResponse,
    tags=["items"]
)
async def update_item(
    item_id: str,
    item: ItemUpdate
):
    """อัพเดทบางส่วนของ item (PATCH)"""
    existing = get_item_by_id(item_id)
    if not existing:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    
    # อัพเดทเฉพาะ fields ที่ส่งมา (exclude_unset)
    update_data = item.dict(exclude_unset=True)
    updated = {**existing, **update_data, "updated_at": datetime.now(timezone.utc)}
    items_db[item_id] = updated
    return updated

@app.delete(
    "/items/{item_id}",
    status_code=status.HTTP_204_NO_CONTENT,
    tags=["items"]
)
async def delete_item(item_id: str):
    """ลบ item"""
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    del items_db[item_id]
    return None

# ==================== Startup Data ====================
@app.on_event("startup")
async def startup_event():
    """เพิ่ม sample data เมื่อ startup"""
    sample_items = [
        {
            "name": "Gaming Laptop",
            "description": "High-performance gaming laptop",
            "price": 29999.00,
            "category": "electronics",
            "tags": ["gaming", "laptop"],
            "in_stock": True
        },
        {
            "name": "Python Programming Book",
            "description": "Complete guide to Python",
            "price": 499.00,
            "category": "books",
            "tags": ["python", "programming"],
            "in_stock": True
        },
    ]
    
    for data in sample_items:
        now = datetime.now(timezone.utc)
        item = {
            "id": str(uuid.uuid4()),
            **data,
            "created_at": now,
            "updated_at": now,
        }
        items_db[item["id"]] = item
    
    print(f"Loaded {len(items_db)} sample items")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 14. ตัวอย่างโปรแกรมจริง: User Management API

```python
# user_management_api.py
from fastapi import FastAPI, HTTPException, Depends, status, Header
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional, List
from datetime import datetime, timezone
import uuid
import hashlib
import secrets
import re

app = FastAPI(
    title="User Management API",
    description="Complete User Management System",
    version="1.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# ==================== Schemas ====================
class UserBase(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, pattern=r'^[a-zA-Z0-9_]+$')
    email: str = Field(..., description="Valid email address")
    full_name: Optional[str] = Field(None, max_length=200)
    bio: Optional[str] = Field(None, max_length=500)
    
    @validator('email')
    def validate_email(cls, v):
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, v):
            raise ValueError('Invalid email format')
        return v.lower()

class UserCreate(UserBase):
    password: str = Field(..., min_length=8, max_length=100)
    
    @validator('password')
    def validate_password(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not re.search(r'[0-9]', v):
            raise ValueError('Password must contain at least one digit')
        return v

class UserUpdate(BaseModel):
    full_name: Optional[str] = Field(None, max_length=200)
    bio: Optional[str] = Field(None, max_length=500)
    email: Optional[str] = None
    
    @validator('email', pre=True, always=False)
    def validate_email(cls, v):
        if v is None:
            return v
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, v):
            raise ValueError('Invalid email format')
        return v.lower()

class UserResponse(BaseModel):
    id: str
    username: str
    email: str
    full_name: Optional[str]
    bio: Optional[str]
    is_active: bool
    created_at: datetime
    updated_at: datetime

class LoginRequest(BaseModel):
    username: str
    password: str

class LoginResponse(BaseModel):
    access_token: str
    token_type: str = "Bearer"
    user: UserResponse

# ==================== Database ====================
users_db: dict = {}
api_tokens: dict = {}  # token -> user_id mapping

def hash_password(password: str) -> str:
    """Hash password ด้วย SHA256 (ใช้ bcrypt ใน production)"""
    return hashlib.sha256(password.encode()).hexdigest()

def verify_password(plain: str, hashed: str) -> bool:
    return hash_password(plain) == hashed

def create_user_dict(data: UserCreate) -> dict:
    now = datetime.now(timezone.utc)
    return {
        "id": str(uuid.uuid4()),
        "username": data.username.lower(),
        "email": data.email,
        "full_name": data.full_name,
        "bio": data.bio,
        "password_hash": hash_password(data.password),
        "is_active": True,
        "created_at": now,
        "updated_at": now,
    }

# ==================== Dependencies ====================
def get_current_user(x_api_token: str = Header(...)):
    """Dependency ตรวจสอบ API token"""
    if x_api_token not in api_tokens:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    user_id = api_tokens[x_api_token]
    user = users_db.get(user_id)
    
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    
    if not user["is_active"]:
        raise HTTPException(status_code=400, detail="Account deactivated")
    
    return user

# ==================== Auth Endpoints ====================
@app.post("/auth/register", response_model=UserResponse, status_code=201, tags=["auth"])
async def register(user_data: UserCreate):
    """ลงทะเบียน user ใหม่"""
    # Check username unique
    if any(u["username"] == user_data.username.lower() for u in users_db.values()):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Username already taken"
        )
    
    # Check email unique
    if any(u["email"] == user_data.email for u in users_db.values()):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Email already registered"
        )
    
    user = create_user_dict(user_data)
    users_db[user["id"]] = user
    
    # Return user without password_hash
    return {k: v for k, v in user.items() if k != "password_hash"}

@app.post("/auth/login", response_model=LoginResponse, tags=["auth"])
async def login(credentials: LoginRequest):
    """Login และรับ API token"""
    # Find user
    user = next(
        (u for u in users_db.values() if u["username"] == credentials.username.lower()),
        None
    )
    
    if not user or not verify_password(credentials.password, user["password_hash"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid username or password"
        )
    
    if not user["is_active"]:
        raise HTTPException(status_code=400, detail="Account is deactivated")
    
    # Generate token
    token = secrets.token_urlsafe(32)
    api_tokens[token] = user["id"]
    
    user_response = {k: v for k, v in user.items() if k != "password_hash"}
    
    return LoginResponse(
        access_token=token,
        user=UserResponse(**user_response)
    )

@app.post("/auth/logout", tags=["auth"])
async def logout(current_user: dict = Depends(get_current_user), x_api_token: str = Header(...)):
    """Logout - invalidate token"""
    if x_api_token in api_tokens:
        del api_tokens[x_api_token]
    return {"message": "Logged out successfully"}

# ==================== User Endpoints ====================
@app.get("/users/me", response_model=UserResponse, tags=["users"])
async def get_my_profile(current_user: dict = Depends(get_current_user)):
    """ดึง profile ของตัวเอง"""
    return {k: v for k, v in current_user.items() if k != "password_hash"}

@app.patch("/users/me", response_model=UserResponse, tags=["users"])
async def update_my_profile(
    update_data: UserUpdate,
    current_user: dict = Depends(get_current_user)
):
    """อัพเดท profile ของตัวเอง"""
    # Check email uniqueness if changing
    if update_data.email and update_data.email != current_user["email"]:
        if any(u["email"] == update_data.email for u in users_db.values()):
            raise HTTPException(status_code=409, detail="Email already in use")
    
    # Update fields
    updates = update_data.dict(exclude_unset=True)
    current_user.update(updates)
    current_user["updated_at"] = datetime.now(timezone.utc)
    
    return {k: v for k, v in current_user.items() if k != "password_hash"}

@app.get("/users/{username}", response_model=UserResponse, tags=["users"])
async def get_user_by_username(username: str):
    """ดึง public profile ของ user"""
    user = next(
        (u for u in users_db.values() if u["username"] == username.lower()),
        None
    )
    if not user:
        raise HTTPException(status_code=404, detail=f"User {username} not found")
    
    return {k: v for k, v in user.items() if k != "password_hash"}

@app.get("/users", response_model=List[UserResponse], tags=["users"])
async def list_users(
    page: int = 1,
    per_page: int = 10,
    current_user: dict = Depends(get_current_user)
):
    """ดึงรายชื่อ users ทั้งหมด (ต้อง auth)"""
    all_users = [
        {k: v for k, v in u.items() if k != "password_hash"}
        for u in users_db.values()
        if u["is_active"]
    ]
    
    start = (page - 1) * per_page
    return all_users[start:start + per_page]

# ==================== Startup ====================
@app.on_event("startup")
async def startup_event():
    """สร้าง demo users"""
    demo_users = [
        UserCreate(
            username="alice",
            email="alice@example.com",
            password="Password123",
            full_name="Alice Smith",
            bio="Software developer"
        ),
        UserCreate(
            username="bob",
            email="bob@example.com",
            password="Password456",
            full_name="Bob Johnson",
            bio="Data scientist"
        )
    ]
    
    for user_data in demo_users:
        user = create_user_dict(user_data)
        users_db[user["id"]] = user
    
    print(f"Created {len(users_db)} demo users")
    print("Demo credentials: alice / Password123")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8001, reload=True)
```

---

## 15. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Hello World FastAPI

**โจทย์**: สร้าง FastAPI application ที่มี endpoints ต่อไปนี้:
- `GET /` -> ส่ง greeting message
- `GET /about` -> ส่ง info เกี่ยวกับ API
- `GET /health` -> Health check

**เฉลย**:

```python
from fastapi import FastAPI
from datetime import datetime

app = FastAPI(title="My First FastAPI")

@app.get("/")
async def root():
    return {
        "message": "Welcome to FastAPI!",
        "docs": "/docs"
    }

@app.get("/about")
async def about():
    return {
        "name": "My API",
        "version": "1.0.0",
        "framework": "FastAPI",
        "author": "Your Name"
    }

@app.get("/health")
async def health():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat() + "Z"
    }
```

### แบบฝึกหัดที่ 2: Path Parameters

**โจทย์**: สร้าง endpoints สำหรับ Blog API:
- `GET /posts/{post_id}` - ดึง post
- `GET /users/{username}/posts` - posts ของ user
- `GET /categories/{category}/posts/{post_id}` - post ใน category

**เฉลย**:

```python
from fastapi import FastAPI, Path, HTTPException
from enum import Enum

app = FastAPI()

class PostCategory(str, Enum):
    tech = "tech"
    science = "science"
    arts = "arts"

@app.get("/posts/{post_id}")
async def get_post(post_id: int = Path(..., ge=1, description="Post ID")):
    if post_id > 1000:
        raise HTTPException(status_code=404, detail="Post not found")
    return {"id": post_id, "title": f"Post {post_id}", "content": "..."}

@app.get("/users/{username}/posts")
async def get_user_posts(
    username: str = Path(..., min_length=3, max_length=50)
):
    return {
        "username": username,
        "posts": [
            {"id": 1, "title": f"Post by {username}"},
        ]
    }

@app.get("/categories/{category}/posts/{post_id}")
async def get_category_post(
    category: PostCategory,
    post_id: int = Path(..., ge=1)
):
    return {
        "category": category.value,
        "post_id": post_id,
        "title": f"Post {post_id} in {category.value}"
    }
```

### แบบฝึกหัดที่ 3: Query Parameters

**โจทย์**: สร้าง search endpoint สำหรับ Movie database ที่รองรับ:
- ค้นหาด้วย title
- Filter ด้วย genre, year, min_rating
- Sort และ paginate

**เฉลย**:

```python
from fastapi import FastAPI, Query
from typing import Optional

app = FastAPI()

MOVIES = [
    {"id": 1, "title": "The Matrix", "genre": "sci-fi", "year": 1999, "rating": 8.7},
    {"id": 2, "title": "Inception", "genre": "sci-fi", "year": 2010, "rating": 8.8},
    {"id": 3, "title": "Interstellar", "genre": "sci-fi", "year": 2014, "rating": 8.6},
    {"id": 4, "title": "The Godfather", "genre": "drama", "year": 1972, "rating": 9.2},
    {"id": 5, "title": "Pulp Fiction", "genre": "crime", "year": 1994, "rating": 8.9},
]

@app.get("/movies")
async def search_movies(
    q: Optional[str] = Query(None, description="Search in title"),
    genre: Optional[str] = Query(None),
    year: Optional[int] = Query(None, ge=1900, le=2030),
    min_rating: Optional[float] = Query(None, ge=0.0, le=10.0),
    sort: str = Query("rating", regex="^(title|year|rating)$"),
    order: str = Query("desc", regex="^(asc|desc)$"),
    page: int = Query(1, ge=1),
    per_page: int = Query(5, ge=1, le=50),
):
    movies = MOVIES.copy()
    
    if q:
        movies = [m for m in movies if q.lower() in m["title"].lower()]
    if genre:
        movies = [m for m in movies if m["genre"] == genre]
    if year:
        movies = [m for m in movies if m["year"] == year]
    if min_rating:
        movies = [m for m in movies if m["rating"] >= min_rating]
    
    movies.sort(key=lambda x: x[sort], reverse=(order == "desc"))
    
    total = len(movies)
    start = (page - 1) * per_page
    return {
        "movies": movies[start:start + per_page],
        "total": total,
        "page": page,
        "per_page": per_page
    }
```

### แบบฝึกหัดที่ 4: Pydantic Models

**โจทย์**: สร้าง Pydantic models สำหรับ Order system พร้อม validation

**เฉลย**:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, validator
from typing import List, Optional
from enum import Enum

app = FastAPI()

class OrderStatus(str, Enum):
    pending = "pending"
    confirmed = "confirmed"
    shipped = "shipped"
    delivered = "delivered"
    cancelled = "cancelled"

class OrderItem(BaseModel):
    product_id: int = Field(..., ge=1)
    quantity: int = Field(..., ge=1, le=100)
    unit_price: float = Field(..., gt=0)
    
    @property
    def subtotal(self) -> float:
        return self.quantity * self.unit_price

class OrderCreate(BaseModel):
    customer_name: str = Field(..., min_length=2, max_length=100)
    customer_email: str = Field(...)
    items: List[OrderItem] = Field(..., min_items=1)
    shipping_address: str = Field(..., min_length=10)
    
    @validator('customer_email')
    def validate_email(cls, v):
        if '@' not in v or '.' not in v.split('@')[-1]:
            raise ValueError('Invalid email')
        return v.lower()
    
    @validator('items')
    def validate_items(cls, v):
        if len(v) == 0:
            raise ValueError('Order must have at least one item')
        return v
    
    @property
    def total(self) -> float:
        return sum(item.quantity * item.unit_price for item in self.items)

class OrderResponse(OrderCreate):
    id: int
    status: OrderStatus = OrderStatus.pending
    total_amount: float

@app.post("/orders", response_model=OrderResponse, status_code=201)
async def create_order(order: OrderCreate):
    return OrderResponse(
        id=1001,
        total_amount=order.total,
        **order.dict()
    )
```

### แบบฝึกหัดที่ 5: Response Models

**โจทย์**: สร้าง API ที่แยก Input model ออกจาก Output model อย่างชัดเจน

**เฉลย**:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

app = FastAPI()

class ProductCreate(BaseModel):
    """Input model - ข้อมูลที่ client ส่งมา"""
    name: str
    price: float = Field(..., gt=0)
    cost: float = Field(..., gt=0)  # Internal cost - ไม่ควรส่งกลับ
    description: Optional[str] = None
    is_featured: bool = False

class ProductPublic(BaseModel):
    """Public response - ไม่มี cost"""
    id: int
    name: str
    price: float
    description: Optional[str]
    is_featured: bool
    created_at: datetime

class ProductAdmin(ProductPublic):
    """Admin response - มี cost"""
    cost: float
    margin: float  # Calculated field

@app.post("/products", response_model=ProductPublic, status_code=201)
async def create_product(product: ProductCreate):
    now = datetime.utcnow()
    new_product = {
        "id": 1,
        **product.dict(),
        "created_at": now
    }
    # FastAPI จะ filter เป็น ProductPublic (ไม่มี cost)
    return new_product

@app.get("/admin/products/{product_id}", response_model=ProductAdmin)
async def get_product_admin(product_id: int):
    """Admin endpoint ที่เห็น cost"""
    product = {
        "id": product_id,
        "name": "Test Product",
        "price": 999.00,
        "cost": 450.00,
        "description": "Test",
        "is_featured": True,
        "created_at": datetime.utcnow(),
        "margin": 549.00,  # price - cost
    }
    return product
```

### แบบฝึกหัดที่ 6: Async Operations

**โจทย์**: สร้าง endpoint ที่ดึงข้อมูลจาก multiple sources พร้อมกันแบบ async

**เฉลย**:

```python
import asyncio
from fastapi import FastAPI
from typing import Dict, Any

app = FastAPI()

async def fetch_user_data(user_id: int) -> Dict:
    """Simulate async database query"""
    await asyncio.sleep(0.1)
    return {"id": user_id, "name": f"User {user_id}", "email": f"user{user_id}@example.com"}

async def fetch_user_orders(user_id: int) -> list:
    """Simulate async database query"""
    await asyncio.sleep(0.15)
    return [
        {"id": 1, "total": 99.99, "status": "completed"},
        {"id": 2, "total": 149.99, "status": "pending"},
    ]

async def fetch_user_stats(user_id: int) -> Dict:
    """Simulate async calculation"""
    await asyncio.sleep(0.05)
    return {
        "total_orders": 15,
        "total_spent": 2499.99,
        "loyalty_points": 250
    }

@app.get("/users/{user_id}/dashboard")
async def get_user_dashboard(user_id: int):
    """
    ดึงข้อมูลจาก 3 sources พร้อมกัน
    เวลา: ~0.15 วินาที (ไม่ใช่ 0.3 วินาที)
    """
    user, orders, stats = await asyncio.gather(
        fetch_user_data(user_id),
        fetch_user_orders(user_id),
        fetch_user_stats(user_id)
    )
    
    return {
        "user": user,
        "recent_orders": orders[:3],
        "stats": stats
    }
```

### แบบฝึกหัดที่ 7: Dependency Injection

**โจทย์**: สร้าง reusable dependencies สำหรับ pagination และ authentication

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException, Header, Query
from typing import Optional

app = FastAPI()

# Mock database
VALID_TOKENS = {"token-alice": "alice", "token-bob": "bob"}
USERS_DB = {
    "alice": {"id": 1, "username": "alice", "role": "admin"},
    "bob": {"id": 2, "username": "bob", "role": "user"},
}

# Pagination dependency
class Pagination:
    def __init__(
        self,
        page: int = Query(default=1, ge=1),
        per_page: int = Query(default=10, ge=1, le=100),
    ):
        self.page = page
        self.per_page = per_page
        self.offset = (page - 1) * per_page

# Auth dependency
async def get_current_user(authorization: str = Header(...)):
    """Extract user from Authorization header"""
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Invalid authorization format")
    
    token = authorization.replace("Bearer ", "")
    username = VALID_TOKENS.get(token)
    
    if not username:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return USERS_DB[username]

# Admin dependency (chains from get_current_user)
async def require_admin(user: dict = Depends(get_current_user)):
    if user["role"] != "admin":
        raise HTTPException(status_code=403, detail="Admin access required")
    return user

ITEMS = [{"id": i, "name": f"Item {i}"} for i in range(1, 51)]

@app.get("/items")
async def list_items(
    pagination: Pagination = Depends(Pagination),
    user: dict = Depends(get_current_user)
):
    start = pagination.offset
    end = start + pagination.per_page
    return {
        "items": ITEMS[start:end],
        "total": len(ITEMS),
        "page": pagination.page,
        "requested_by": user["username"]
    }

@app.delete("/items/{item_id}")
async def delete_item(
    item_id: int,
    admin: dict = Depends(require_admin)  # Only admins
):
    return {"message": f"Item {item_id} deleted by {admin['username']}"}
```

### แบบฝึกหัดที่ 8: Complete Mini API

**โจทย์**: สร้าง complete Note-taking API ด้วย FastAPI พร้อม CRUD ครบถ้วน

**เฉลย**:

```python
from fastapi import FastAPI, HTTPException, Depends, Query, Path, status
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime, timezone
import uuid

app = FastAPI(title="Notes API", version="1.0.0")

# Schemas
class NoteCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    content: str = Field(..., min_length=1)
    tags: List[str] = []
    is_pinned: bool = False

class NoteUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    content: Optional[str] = None
    tags: Optional[List[str]] = None
    is_pinned: Optional[bool] = None

class NoteResponse(BaseModel):
    id: str
    title: str
    content: str
    tags: List[str]
    is_pinned: bool
    created_at: datetime
    updated_at: datetime

# Database
notes_db: dict = {}

@app.get("/notes", response_model=List[NoteResponse])
async def list_notes(
    q: Optional[str] = Query(None),
    tag: Optional[str] = Query(None),
    pinned: Optional[bool] = Query(None),
    page: int = Query(1, ge=1),
    per_page: int = Query(10, ge=1, le=50),
):
    notes = list(notes_db.values())
    
    if q:
        notes = [n for n in notes if q.lower() in n["title"].lower() or q.lower() in n["content"].lower()]
    if tag:
        notes = [n for n in notes if tag in n["tags"]]
    if pinned is not None:
        notes = [n for n in notes if n["is_pinned"] == pinned]
    
    # Pinned notes first
    notes.sort(key=lambda x: (not x["is_pinned"], x["created_at"]), reverse=False)
    
    start = (page - 1) * per_page
    return notes[start:start + per_page]

@app.post("/notes", response_model=NoteResponse, status_code=201)
async def create_note(note: NoteCreate):
    now = datetime.now(timezone.utc)
    new_note = {
        "id": str(uuid.uuid4()),
        **note.dict(),
        "created_at": now,
        "updated_at": now,
    }
    notes_db[new_note["id"]] = new_note
    return new_note

@app.get("/notes/{note_id}", response_model=NoteResponse)
async def get_note(note_id: str = Path(...)):
    note = notes_db.get(note_id)
    if not note:
        raise HTTPException(status_code=404, detail="Note not found")
    return note

@app.patch("/notes/{note_id}", response_model=NoteResponse)
async def update_note(note_id: str, update_data: NoteUpdate):
    note = notes_db.get(note_id)
    if not note:
        raise HTTPException(status_code=404, detail="Note not found")
    
    updates = update_data.dict(exclude_unset=True)
    note.update(updates)
    note["updated_at"] = datetime.now(timezone.utc)
    return note

@app.delete("/notes/{note_id}", status_code=204)
async def delete_note(note_id: str):
    if note_id not in notes_db:
        raise HTTPException(status_code=404, detail="Note not found")
    del notes_db[note_id]

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, reload=True)
```

---

## สรุป

ใน Part 57 เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียนรู้ |
|--------|-------------------|
| FastAPI vs Others | เปรียบเทียบ performance, features, use cases |
| Setup | Installation, project structure, configuration |
| First App | สร้าง app แรก, รันด้วย uvicorn |
| Path Operations | HTTP methods, decorator parameters |
| Auto Documentation | Swagger UI, ReDoc, custom schema |
| Path Parameters | Type hints, validation, Enum |
| Query Parameters | Optional, required, List, validation |
| Request Body | Pydantic models, nested models |
| Response Models | Input/Output separation, filtering |
| Status Codes | Standard codes, custom exceptions |
| Async Handlers | async/await, concurrent operations |
| Dependency Injection | Reusable dependencies, chains |

### Commands

```bash
# รัน development server
uvicorn main:app --reload

# รัน production
uvicorn main:app --workers 4 --host 0.0.0.0 --port 8000

# ดู documentation
open http://localhost:8000/docs
open http://localhost:8000/redoc
```

### ขั้นตอนต่อไป

- **Part 58**: FastAPI - Routing, Validation & Pydantic (advanced)
- **Part 59**: FastAPI - Database & Async ORM
- **Part 60**: FastAPI - Authentication, JWT & OAuth2
