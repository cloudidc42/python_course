# Part 59: FastAPI - Database & Async ORM

## สารบัญ

1. [SQLAlchemy Async](#1-sqlalchemy-async)
2. [Database Session Management](#2-database-session-management)
3. [Dependency Injection สำหรับ DB](#3-dependency-injection-สำหรับ-db)
4. [Async CRUD Operations](#4-async-crud-operations)
5. [SQLModel (FastAPI Creator's Library)](#5-sqlmodel-fastapi-creators-library)
6. [Relationships in Async Context](#6-relationships-in-async-context)
7. [Database Migrations with Alembic](#7-database-migrations-with-alembic)
8. [Connection Pooling](#8-connection-pooling)
9. [Transaction Management](#9-transaction-management)
10. [Multiple Databases](#10-multiple-databases)
11. [ตัวอย่างโปรแกรมจริง: Complete Async API with Database](#11-ตัวอย่างโปรแกรมจริง-complete-async-api-with-database)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. SQLAlchemy Async

### การติดตั้ง

```bash
# ติดตั้ง SQLAlchemy async พร้อม async driver
pip install sqlalchemy[asyncio] asyncpg aiomysql aiosqlite

# สำหรับ PostgreSQL
pip install asyncpg

# สำหรับ SQLite (development)
pip install aiosqlite

# สำหรับ MySQL
pip install aiomysql

# รวมทั้งหมด
pip install fastapi uvicorn sqlalchemy[asyncio] aiosqlite alembic
```

### Async Engine และ Session

```python
# database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, Text
from sqlalchemy.sql import func

# ====== Database URLs ======
# SQLite (development)
SQLITE_URL = "sqlite+aiosqlite:///./app.db"

# PostgreSQL (production)
POSTGRES_URL = "postgresql+asyncpg://user:password@localhost:5432/mydb"

# MySQL
MYSQL_URL = "mysql+aiomysql://user:password@localhost:3306/mydb"

# ====== Create Engine ======
engine = create_async_engine(
    SQLITE_URL,
    echo=True,  # Log SQL statements (ปิดใน production)
    future=True,
)

# ====== Session Factory ======
AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,  # ไม่ expire objects หลัง commit
    autocommit=False,
    autoflush=False,
)

# ====== Base Model ======
class Base(DeclarativeBase):
    pass
```

### การสร้าง Table Models

```python
# models.py
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, Text, ForeignKey, Table
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from database import Base

class TimestampMixin:
    """Mixin สำหรับ created_at, updated_at fields"""
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    updated_at = Column(DateTime(timezone=True), onupdate=func.now(), server_default=func.now())

class User(Base, TimestampMixin):
    """User table"""
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(200), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    full_name = Column(String(200))
    bio = Column(Text)
    is_active = Column(Boolean, default=True, nullable=False)
    
    # Relationships
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    orders = relationship("Order", back_populates="user")

class Category(Base):
    """Category table"""
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False)
    description = Column(Text)
    
    products = relationship("Product", back_populates="category")

class Product(Base, TimestampMixin):
    """Product table"""
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False, index=True)
    description = Column(Text)
    price = Column(Float, nullable=False)
    stock = Column(Integer, default=0, nullable=False)
    is_active = Column(Boolean, default=True)
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    
    category = relationship("Category", back_populates="products")
    order_items = relationship("OrderItem", back_populates="product")

class Post(Base, TimestampMixin):
    """Blog post table"""
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    published = Column(Boolean, default=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    
    author = relationship("User", back_populates="posts")
```

### สร้าง Tables

```python
# create_tables.py
import asyncio
from database import engine, Base
from models import User, Category, Product, Post  # Import ทุก models

async def create_tables():
    """สร้าง database tables"""
    async with engine.begin() as conn:
        # สร้างทุก tables ที่ inherit จาก Base
        await conn.run_sync(Base.metadata.create_all)
    print("Tables created successfully!")

async def drop_tables():
    """ลบ database tables ทั้งหมด (ระวัง!)"""
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    print("Tables dropped!")

if __name__ == "__main__":
    asyncio.run(create_tables())
```

---

## 2. Database Session Management

### Session Lifecycle

```python
# database.py
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from contextlib import asynccontextmanager

engine = create_async_engine("sqlite+aiosqlite:///./app.db", echo=True)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)

@asynccontextmanager
async def get_db_session():
    """Context manager สำหรับ database session"""
    session = AsyncSessionLocal()
    try:
        yield session
        await session.commit()
    except Exception:
        await session.rollback()
        raise
    finally:
        await session.close()

# ใช้งาน:
async def example_usage():
    async with get_db_session() as db:
        # ใช้ db session ที่นี่
        result = await db.execute(...)
        await db.commit()
```

### Session ใน FastAPI

```python
# dependencies.py
from sqlalchemy.ext.asyncio import AsyncSession
from typing import AsyncGenerator
from database import AsyncSessionLocal

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """
    FastAPI dependency สำหรับ database session
    
    - สร้าง session ก่อน request
    - yield session ให้ endpoint ใช้
    - commit ถ้าไม่มี error
    - rollback ถ้ามี error
    - ปิด session หลัง request
    """
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

# ใช้ใน endpoint:
from fastapi import FastAPI, Depends
from sqlalchemy.ext.asyncio import AsyncSession

app = FastAPI()

@app.get("/users")
async def list_users(db: AsyncSession = Depends(get_db)):
    """ใช้ session ผ่าน dependency injection"""
    from sqlalchemy import select
    from models import User
    
    result = await db.execute(select(User).where(User.is_active == True))
    users = result.scalars().all()
    return users
```

---

## 3. Dependency Injection สำหรับ DB

### DB Dependency Pattern

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update, delete
from typing import Optional

app = FastAPI()

# ====== Dependency ======
async def get_db():
    from database import AsyncSessionLocal
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise

# ====== Repository Pattern ======
class UserRepository:
    """Repository สำหรับ User operations"""
    
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def get_by_id(self, user_id: int):
        from models import User
        result = await self.db.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalar_one_or_none()
    
    async def get_by_email(self, email: str):
        from models import User
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
    
    async def get_all(self, skip: int = 0, limit: int = 10):
        from models import User
        result = await self.db.execute(
            select(User).offset(skip).limit(limit)
        )
        return result.scalars().all()
    
    async def create(self, user_data: dict):
        from models import User
        user = User(**user_data)
        self.db.add(user)
        await self.db.flush()  # Get ID without commit
        await self.db.refresh(user)
        return user
    
    async def update(self, user_id: int, update_data: dict):
        from models import User
        result = await self.db.execute(
            update(User)
            .where(User.id == user_id)
            .values(**update_data)
            .returning(User)
        )
        return result.scalar_one_or_none()
    
    async def delete(self, user_id: int) -> bool:
        from models import User
        result = await self.db.execute(
            delete(User).where(User.id == user_id)
        )
        return result.rowcount > 0

def get_user_repository(db: AsyncSession = Depends(get_db)) -> UserRepository:
    return UserRepository(db)

# ====== Endpoints ======
from pydantic import BaseModel

class UserCreate(BaseModel):
    username: str
    email: str
    password: str

@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    user_repo: UserRepository = Depends(get_user_repository)
):
    user = await user_repo.get_by_id(user_id)
    if not user:
        raise HTTPException(404, "User not found")
    return user

@app.post("/users", status_code=201)
async def create_user(
    user_data: UserCreate,
    user_repo: UserRepository = Depends(get_user_repository)
):
    # Check email uniqueness
    existing = await user_repo.get_by_email(user_data.email)
    if existing:
        raise HTTPException(409, "Email already registered")
    
    import hashlib
    user = await user_repo.create({
        "username": user_data.username,
        "email": user_data.email,
        "password_hash": hashlib.sha256(user_data.password.encode()).hexdigest()
    })
    return user
```

---

## 4. Async CRUD Operations

### Select Queries

```python
from sqlalchemy import select, func, and_, or_, desc, asc, text
from sqlalchemy.orm import selectinload, joinedload
from sqlalchemy.ext.asyncio import AsyncSession
from models import User, Post, Product

async def crud_examples(db: AsyncSession):
    
    # ====== SELECT ======
    
    # ดึงทุก records
    result = await db.execute(select(User))
    all_users = result.scalars().all()
    
    # ดึง record เดียว
    result = await db.execute(select(User).where(User.id == 1))
    user = result.scalar_one_or_none()  # None ถ้าไม่พบ
    
    # Filter หลายเงื่อนไข
    result = await db.execute(
        select(User).where(
            and_(
                User.is_active == True,
                User.email.like('%@example.com')
            )
        )
    )
    
    # ORDER BY
    result = await db.execute(
        select(User)
        .order_by(desc(User.created_at))
        .limit(10)
        .offset(0)
    )
    
    # COUNT
    result = await db.execute(
        select(func.count(User.id)).where(User.is_active == True)
    )
    count = result.scalar()
    
    # DISTINCT
    result = await db.execute(
        select(Product.category_id).distinct()
    )
    categories = result.scalars().all()
    
    # IN operator
    user_ids = [1, 2, 3, 4, 5]
    result = await db.execute(
        select(User).where(User.id.in_(user_ids))
    )
    
    # OR conditions
    result = await db.execute(
        select(Product).where(
            or_(
                Product.price < 100,
                Product.price > 10000
            )
        )
    )
    
    # LIKE search
    result = await db.execute(
        select(Product).where(
            Product.name.ilike(f'%laptop%')  # case-insensitive
        )
    )
    
    return all_users
```

### Insert Operations

```python
from sqlalchemy import insert

async def insert_examples(db: AsyncSession):
    
    # ====== INSERT ======
    
    # Insert single record
    from models import User
    new_user = User(
        username="alice",
        email="alice@example.com",
        password_hash="hashed_password",
        full_name="Alice Smith"
    )
    db.add(new_user)
    await db.flush()  # Flush to get generated ID
    await db.refresh(new_user)  # Refresh to get all DB-generated values
    print(f"Created user with ID: {new_user.id}")
    
    # Insert multiple records
    from models import Product
    products = [
        Product(name="Laptop", price=999.99, stock=50),
        Product(name="Mouse", price=29.99, stock=200),
        Product(name="Keyboard", price=49.99, stock=150),
    ]
    db.add_all(products)
    await db.flush()
    
    # Bulk insert ด้วย insert()
    await db.execute(
        insert(Product).values([
            {"name": "Monitor", "price": 299.99, "stock": 30},
            {"name": "Webcam", "price": 79.99, "stock": 100},
        ])
    )
    
    await db.commit()
```

### Update Operations

```python
from sqlalchemy import update as sa_update
from datetime import datetime, timezone

async def update_examples(db: AsyncSession):
    
    from models import User, Product
    
    # ====== UPDATE ======
    
    # Update record โดย ORM
    result = await db.execute(select(User).where(User.id == 1))
    user = result.scalar_one_or_none()
    if user:
        user.full_name = "Alice Johnson"
        user.updated_at = datetime.now(timezone.utc)
        await db.commit()
    
    # Bulk update ด้วย update()
    await db.execute(
        sa_update(Product)
        .where(Product.stock == 0)
        .values(is_active=False)
    )
    await db.commit()
    
    # Update หลาย fields
    await db.execute(
        sa_update(User)
        .where(User.id == 1)
        .values(
            full_name="Updated Name",
            bio="Updated bio",
        )
    )
    await db.commit()
    
    # Conditional update
    await db.execute(
        sa_update(Product)
        .where(
            and_(
                Product.price > 1000,
                Product.is_active == True
            )
        )
        .values(price=Product.price * 0.9)  # 10% discount
    )
    await db.commit()
```

### Delete Operations

```python
from sqlalchemy import delete as sa_delete

async def delete_examples(db: AsyncSession):
    
    from models import User, Product
    
    # ====== DELETE ======
    
    # Delete by ORM object
    result = await db.execute(select(User).where(User.id == 99))
    user = result.scalar_one_or_none()
    if user:
        await db.delete(user)
        await db.commit()
    
    # Bulk delete
    await db.execute(
        sa_delete(Product).where(Product.is_active == False)
    )
    await db.commit()
    
    # Soft delete (ไม่ลบจริง แค่ set flag)
    await db.execute(
        sa_update(User)
        .where(User.id == 1)
        .values(is_active=False)
    )
    await db.commit()
```

---

## 5. SQLModel (FastAPI Creator's Library)

SQLModel รวม Pydantic + SQLAlchemy ไว้ด้วยกัน ทำให้ define models ได้ครั้งเดียว

### การติดตั้ง

```bash
pip install sqlmodel
```

### SQLModel Models

```python
from sqlmodel import SQLModel, Field, Relationship, Session, create_engine, select
from typing import Optional, List
from datetime import datetime

# ====== SQLModel Definitions ======

class UserBase(SQLModel):
    """Shared fields ระหว่าง input/output"""
    username: str = Field(index=True, min_length=3, max_length=50)
    email: str = Field(index=True)
    full_name: Optional[str] = None
    bio: Optional[str] = None
    is_active: bool = True

class User(UserBase, table=True):
    """Database table model (table=True)"""
    __tablename__ = "users"
    
    id: Optional[int] = Field(default=None, primary_key=True)
    password_hash: str
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    # Relationships
    posts: List["Post"] = Relationship(back_populates="author")

class UserCreate(UserBase):
    """Request model สำหรับ create"""
    password: str = Field(min_length=8)

class UserRead(UserBase):
    """Response model"""
    id: int
    created_at: datetime

class UserUpdate(SQLModel):
    """Partial update model"""
    full_name: Optional[str] = None
    bio: Optional[str] = None
    email: Optional[str] = None

class PostBase(SQLModel):
    title: str = Field(min_length=5, max_length=200)
    content: str
    published: bool = False

class Post(PostBase, table=True):
    __tablename__ = "posts"
    
    id: Optional[int] = Field(default=None, primary_key=True)
    author_id: Optional[int] = Field(default=None, foreign_key="users.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    
    author: Optional[User] = Relationship(back_populates="posts")

class PostCreate(PostBase):
    pass

class PostRead(PostBase):
    id: int
    author_id: int
    created_at: datetime
```

### FastAPI + SQLModel

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlmodel import Session, SQLModel, create_engine, select
import asyncio

# ====== Async SQLModel (sqlmodel ใช้ async engine ด้วย) ======
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite+aiosqlite:///./sqlmodel_app.db"

async_engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(
    bind=async_engine, class_=AsyncSession, expire_on_commit=False
)

app = FastAPI(title="SQLModel API")

async def get_session():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise

@app.on_event("startup")
async def create_db_and_tables():
    async with async_engine.begin() as conn:
        await conn.run_sync(SQLModel.metadata.create_all)

@app.get("/users", response_model=List[UserRead])
async def list_users(
    session: AsyncSession = Depends(get_session),
    skip: int = 0,
    limit: int = 10
):
    result = await session.execute(select(User).offset(skip).limit(limit))
    users = result.scalars().all()
    return users

@app.post("/users", response_model=UserRead, status_code=201)
async def create_user(
    user_data: UserCreate,
    session: AsyncSession = Depends(get_session)
):
    import hashlib
    
    # Check email uniqueness
    result = await session.execute(
        select(User).where(User.email == user_data.email)
    )
    existing = result.scalar_one_or_none()
    if existing:
        raise HTTPException(409, "Email already registered")
    
    user = User(
        username=user_data.username,
        email=user_data.email,
        full_name=user_data.full_name,
        password_hash=hashlib.sha256(user_data.password.encode()).hexdigest()
    )
    session.add(user)
    await session.commit()
    await session.refresh(user)
    return user
```

---

## 6. Relationships in Async Context

### Loading Relationships Async

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload, joinedload, noload
from sqlalchemy.ext.asyncio import AsyncSession
from models import User, Post, Product, Category

async def load_relationships(db: AsyncSession):
    
    # ====== Eager Loading ======
    
    # selectinload: ออก 2 queries (ดีกว่าสำหรับ many-to-many)
    result = await db.execute(
        select(User)
        .where(User.id == 1)
        .options(selectinload(User.posts))
    )
    user = result.scalar_one_or_none()
    if user:
        # ไม่ต้องรอ lazy load เพราะ selectinload โหลดมาแล้ว
        for post in user.posts:
            print(f"Post: {post.title}")
    
    # joinedload: ออก 1 query ด้วย JOIN (ดีสำหรับ many-to-one / one-to-one)
    result = await db.execute(
        select(Post)
        .where(Post.id == 1)
        .options(joinedload(Post.author))
    )
    post = result.unique().scalar_one_or_none()
    if post:
        print(f"Author: {post.author.username}")
    
    # โหลด nested relationships
    result = await db.execute(
        select(User)
        .options(
            selectinload(User.posts).selectinload(Post.author)
        )
    )
    users = result.unique().scalars().all()
    
    # noload: ไม่โหลด relationship เลย
    result = await db.execute(
        select(User).options(noload(User.posts))
    )
```

### Lazy Loading ใน Async (ต้องระวัง)

```python
from sqlalchemy.ext.asyncio import AsyncSession

async def lazy_load_warning(db: AsyncSession):
    """
    ⚠️ Lazy loading ไม่ work ใน async context โดย default
    ต้องใช้ eager loading หรือ lazy='select' ไม่ได้ใช้ async
    """
    from models import User
    from sqlalchemy import select
    
    result = await db.execute(select(User).where(User.id == 1))
    user = result.scalar_one_or_none()
    
    if user:
        # ❌ นี้จะ raise MissingGreenlet error ถ้าใช้ lazy loading
        # posts = user.posts  # Error!
        
        # ✅ วิธีที่ถูกต้อง: ใช้ explicit query
        from models import Post
        result = await db.execute(
            select(Post).where(Post.author_id == user.id)
        )
        posts = result.scalars().all()
        return user, posts
```

### Relationship Queries

```python
async def relationship_queries(db: AsyncSession):
    from models import User, Post, Product, Category
    from sqlalchemy import select, func
    
    # ====== JOIN Queries ======
    
    # Inner Join
    result = await db.execute(
        select(User, Post)
        .join(Post, Post.author_id == User.id)
        .where(Post.published == True)
    )
    user_posts = result.all()
    
    # Left outer join
    result = await db.execute(
        select(User, func.count(Post.id).label("post_count"))
        .outerjoin(Post, Post.author_id == User.id)
        .group_by(User.id)
        .order_by(func.count(Post.id).desc())
    )
    user_stats = result.all()
    
    # Subquery
    from sqlalchemy import subquery
    
    active_user_ids = (
        select(User.id).where(User.is_active == True).scalar_subquery()
    )
    
    result = await db.execute(
        select(Post)
        .where(Post.author_id.in_(active_user_ids))
        .where(Post.published == True)
    )
    active_posts = result.scalars().all()
```

---

## 7. Database Migrations with Alembic

### การติดตั้งและ Setup

```bash
pip install alembic

# Initialize alembic
alembic init alembic
```

### alembic.ini Configuration

```ini
# alembic.ini
[alembic]
script_location = alembic
sqlalchemy.url = sqlite:///./app.db

# สำหรับ PostgreSQL
# sqlalchemy.url = postgresql://user:password@localhost/mydb
```

### env.py Configuration

```python
# alembic/env.py
import asyncio
from logging.config import fileConfig
from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config
from alembic import context

# Import Base จาก models ของเรา
import sys
sys.path.append('.')
from database import Base
from models import User, Product, Post, Category  # Import ทุก models

config = context.config
fileConfig(config.config_file_name)
target_metadata = Base.metadata

def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode"""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()

def do_run_migrations(connection: Connection) -> None:
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

async def run_async_migrations() -> None:
    """Run migrations in 'online' mode async"""
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    
    await connectable.dispose()

def run_migrations_online() -> None:
    asyncio.run(run_async_migrations())

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### Migration Commands

```bash
# สร้าง migration แรก (auto-detect จาก models)
alembic revision --autogenerate -m "Create initial tables"

# สร้าง migration ด้วยตนเอง
alembic revision -m "Add user bio column"

# รัน migrations ทั้งหมด
alembic upgrade head

# Rollback 1 migration
alembic downgrade -1

# Rollback ทั้งหมด
alembic downgrade base

# ดู current version
alembic current

# ดู history
alembic history

# ดูว่ามี pending migrations
alembic check
```

### Migration File Example

```python
# alembic/versions/xxxx_create_initial_tables.py
"""Create initial tables

Revision ID: abc123
Revises: 
Create Date: 2024-01-01 10:00:00

"""
from typing import Sequence, Union
from alembic import op
import sqlalchemy as sa

revision: str = 'abc123'
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None

def upgrade() -> None:
    """สร้าง tables"""
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('username', sa.String(50), nullable=False),
        sa.Column('email', sa.String(200), nullable=False),
        sa.Column('password_hash', sa.String(255), nullable=False),
        sa.Column('full_name', sa.String(200)),
        sa.Column('bio', sa.Text()),
        sa.Column('is_active', sa.Boolean(), default=True),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.text('now()')),
        sa.Column('updated_at', sa.DateTime(timezone=True)),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('username'),
        sa.UniqueConstraint('email'),
    )
    op.create_index(op.f('ix_users_username'), 'users', ['username'])
    op.create_index(op.f('ix_users_email'), 'users', ['email'])

def downgrade() -> None:
    """ลบ tables"""
    op.drop_index(op.f('ix_users_email'), table_name='users')
    op.drop_index(op.f('ix_users_username'), table_name='users')
    op.drop_table('users')
```

---

## 8. Connection Pooling

### Pool Configuration

```python
from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy.pool import NullPool, StaticPool, QueuePool

# SQLite (development) - ใช้ StaticPool
dev_engine = create_async_engine(
    "sqlite+aiosqlite:///./dev.db",
    echo=True,
    poolclass=StaticPool,  # ใช้ connection เดียว (สำหรับ SQLite)
    connect_args={"check_same_thread": False}
)

# PostgreSQL (production)
prod_engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/mydb",
    pool_size=20,          # จำนวน connections ใน pool
    max_overflow=40,       # Extra connections ที่อนุญาต
    pool_pre_ping=True,    # Test connection ก่อนใช้
    pool_recycle=3600,     # Recycle connections ทุก 1 ชั่วโมง
    pool_timeout=30,       # Timeout รอ connection (seconds)
    echo=False,            # ปิด SQL logging ใน production
    echo_pool=False,       # ปิด pool logging
)

# Testing - ไม่ใช้ pool
test_engine = create_async_engine(
    "sqlite+aiosqlite:///:memory:",
    poolclass=NullPool,  # ไม่ใช้ pool (สร้าง/ทำลาย connection ทุกครั้ง)
)
```

### Pool Events

```python
from sqlalchemy import event
from sqlalchemy.ext.asyncio import create_async_engine

engine = create_async_engine("sqlite+aiosqlite:///./app.db")

@event.listens_for(engine.sync_engine, "connect")
def on_connect(dbapi_connection, connection_record):
    """Event เมื่อสร้าง database connection ใหม่"""
    print("New database connection created")

@event.listens_for(engine.sync_engine, "checkout")
def on_checkout(dbapi_connection, connection_record, connection_proxy):
    """Event เมื่อ checkout connection จาก pool"""
    print("Connection checked out from pool")

@event.listens_for(engine.sync_engine, "checkin")
def on_checkin(dbapi_connection, connection_record):
    """Event เมื่อ return connection กลับ pool"""
    print("Connection returned to pool")
```

---

## 9. Transaction Management

### Manual Transaction

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update

async def transfer_funds(
    db: AsyncSession,
    from_user_id: int,
    to_user_id: int,
    amount: float
):
    """โอนเงินด้วย transaction (ต้อง atomic)"""
    
    async with db.begin():  # Begin transaction (auto-rollback ถ้า exception)
        # ดึง users
        from_result = await db.execute(
            select(Account).where(Account.user_id == from_user_id)
            .with_for_update()  # Lock ป้องกัน race condition
        )
        from_account = from_result.scalar_one_or_none()
        
        if not from_account:
            raise ValueError(f"Account for user {from_user_id} not found")
        
        if from_account.balance < amount:
            raise ValueError("Insufficient funds")
        
        to_result = await db.execute(
            select(Account).where(Account.user_id == to_user_id)
            .with_for_update()
        )
        to_account = to_result.scalar_one_or_none()
        
        if not to_account:
            raise ValueError(f"Account for user {to_user_id} not found")
        
        # Deduct from sender
        from_account.balance -= amount
        
        # Add to receiver
        to_account.balance += amount
        
        # ถ้าทุกอย่าง OK จะ commit อัตโนมัติ
        # ถ้ามี exception จะ rollback อัตโนมัติ
    
    return {
        "status": "success",
        "from_balance": from_account.balance,
        "to_balance": to_account.balance
    }
```

### Savepoints

```python
async def complex_transaction(db: AsyncSession):
    """Transaction ที่ใช้ savepoints"""
    
    async with db.begin():
        # Main transaction
        user = User(username="alice", email="alice@example.com")
        db.add(user)
        await db.flush()
        
        try:
            # Savepoint - ถ้า fail แค่ rollback ถึง savepoint นี้
            async with db.begin_nested():
                profile = UserProfile(user_id=user.id, bio="...")
                db.add(profile)
                await db.flush()
                
                # อาจ fail
                if not profile.bio:
                    raise ValueError("Bio required")
                
        except ValueError:
            # Rollback เฉพาะ nested transaction
            # User ยังถูกสร้าง
            print("Profile creation failed, but user was created")
        
        await db.commit()  # Commit main transaction
```

---

## 10. Multiple Databases

### Multiple Database Setup

```python
# databases.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

# Primary database (main)
primary_engine = create_async_engine(
    "postgresql+asyncpg://user:pass@primary-host/primary_db",
    pool_size=20,
    max_overflow=40,
)

# Read replica (for queries)
replica_engine = create_async_engine(
    "postgresql+asyncpg://user:pass@replica-host/primary_db",
    pool_size=30,
    max_overflow=60,
)

# Analytics database (separate)
analytics_engine = create_async_engine(
    "postgresql+asyncpg://user:pass@analytics-host/analytics_db",
    pool_size=10,
)

# Session factories
PrimarySession = async_sessionmaker(bind=primary_engine, expire_on_commit=False)
ReplicaSession = async_sessionmaker(bind=replica_engine, expire_on_commit=False)
AnalyticsSession = async_sessionmaker(bind=analytics_engine, expire_on_commit=False)

# Dependencies
async def get_primary_db():
    async with PrimarySession() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise

async def get_replica_db():
    """Read-only session สำหรับ queries"""
    async with ReplicaSession() as session:
        yield session

async def get_analytics_db():
    async with AnalyticsSession() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise
```

---

## 11. ตัวอย่างโปรแกรมจริง: Complete Async API with Database

```python
# complete_api.py - Blog API ด้วย SQLAlchemy Async
from fastapi import FastAPI, Depends, HTTPException, Query, Path, status, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase, selectinload
from sqlalchemy import Column, Integer, String, Text, Boolean, DateTime, ForeignKey, func, select, update as sa_update, delete as sa_delete
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime, timezone
import hashlib
import asyncio

# ====== Database Setup ======
DATABASE_URL = "sqlite+aiosqlite:///./blog.db"

engine = create_async_engine(DATABASE_URL, echo=False)
AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)

class Base(DeclarativeBase):
    pass

# ====== Models ======
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(200), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    full_name = Column(String(200))
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))

class Post(Base):
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    summary = Column(String(500))
    published = Column(Boolean, default=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))
    updated_at = Column(DateTime(timezone=True), onupdate=lambda: datetime.now(timezone.utc))

class Comment(Base):
    __tablename__ = "comments"
    
    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    post_id = Column(Integer, ForeignKey("posts.id"), nullable=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))

# ====== Pydantic Schemas ======
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: str
    password: str = Field(..., min_length=8)
    full_name: Optional[str] = None

class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    full_name: Optional[str]
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

class PostCreate(BaseModel):
    title: str = Field(..., min_length=5, max_length=200)
    content: str = Field(..., min_length=10)
    summary: Optional[str] = Field(None, max_length=500)
    published: bool = False

class PostUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=5, max_length=200)
    content: Optional[str] = None
    summary: Optional[str] = None
    published: Optional[bool] = None

class PostResponse(BaseModel):
    id: int
    title: str
    content: str
    summary: Optional[str]
    published: bool
    author_id: int
    created_at: datetime
    updated_at: Optional[datetime]
    
    class Config:
        from_attributes = True

class PostWithAuthor(PostResponse):
    author: Optional[UserResponse] = None

class CommentCreate(BaseModel):
    content: str = Field(..., min_length=1)

class CommentResponse(BaseModel):
    id: int
    content: str
    post_id: int
    author_id: int
    created_at: datetime
    
    class Config:
        from_attributes = True

# ====== Database Dependency ======
async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise

# ====== Repositories ======
class UserRepository:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def get_by_id(self, user_id: int) -> Optional[User]:
        result = await self.db.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalar_one_or_none()
    
    async def get_by_username(self, username: str) -> Optional[User]:
        result = await self.db.execute(
            select(User).where(User.username == username)
        )
        return result.scalar_one_or_none()
    
    async def get_by_email(self, email: str) -> Optional[User]:
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()
    
    async def list_users(self, skip: int = 0, limit: int = 10) -> List[User]:
        result = await self.db.execute(
            select(User)
            .where(User.is_active == True)
            .offset(skip).limit(limit)
            .order_by(User.created_at.desc())
        )
        return result.scalars().all()
    
    async def create(self, data: dict) -> User:
        user = User(**data)
        self.db.add(user)
        await self.db.flush()
        await self.db.refresh(user)
        return user

class PostRepository:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def get_by_id(self, post_id: int) -> Optional[Post]:
        result = await self.db.execute(
            select(Post).where(Post.id == post_id)
        )
        return result.scalar_one_or_none()
    
    async def list_posts(
        self,
        author_id: Optional[int] = None,
        published_only: bool = True,
        skip: int = 0,
        limit: int = 10
    ) -> List[Post]:
        query = select(Post)
        
        if author_id:
            query = query.where(Post.author_id == author_id)
        if published_only:
            query = query.where(Post.published == True)
        
        query = query.offset(skip).limit(limit).order_by(Post.created_at.desc())
        result = await self.db.execute(query)
        return result.scalars().all()
    
    async def count_posts(self, author_id: Optional[int] = None, published_only: bool = True) -> int:
        query = select(func.count(Post.id))
        if author_id:
            query = query.where(Post.author_id == author_id)
        if published_only:
            query = query.where(Post.published == True)
        result = await self.db.execute(query)
        return result.scalar()
    
    async def create(self, data: dict) -> Post:
        post = Post(**data)
        self.db.add(post)
        await self.db.flush()
        await self.db.refresh(post)
        return post
    
    async def update(self, post_id: int, data: dict) -> Optional[Post]:
        post = await self.get_by_id(post_id)
        if not post:
            return None
        for key, value in data.items():
            setattr(post, key, value)
        post.updated_at = datetime.now(timezone.utc)
        await self.db.flush()
        await self.db.refresh(post)
        return post
    
    async def delete(self, post_id: int) -> bool:
        result = await self.db.execute(
            sa_delete(Post).where(Post.id == post_id)
        )
        return result.rowcount > 0

# ====== FastAPI Application ======
app = FastAPI(title="Blog API", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

def get_user_repo(db: AsyncSession = Depends(get_db)) -> UserRepository:
    return UserRepository(db)

def get_post_repo(db: AsyncSession = Depends(get_db)) -> PostRepository:
    return PostRepository(db)

# ====== User Endpoints ======
@app.post("/api/v1/users", response_model=UserResponse, status_code=201, tags=["users"])
async def register_user(
    user_data: UserCreate,
    user_repo: UserRepository = Depends(get_user_repo)
):
    # Check uniqueness
    if await user_repo.get_by_email(user_data.email):
        raise HTTPException(409, "Email already registered")
    if await user_repo.get_by_username(user_data.username):
        raise HTTPException(409, "Username already taken")
    
    user = await user_repo.create({
        "username": user_data.username.lower(),
        "email": user_data.email.lower(),
        "password_hash": hashlib.sha256(user_data.password.encode()).hexdigest(),
        "full_name": user_data.full_name,
    })
    return user

@app.get("/api/v1/users", response_model=List[UserResponse], tags=["users"])
async def list_users(
    page: int = Query(1, ge=1),
    per_page: int = Query(10, ge=1, le=50),
    user_repo: UserRepository = Depends(get_user_repo)
):
    skip = (page - 1) * per_page
    users = await user_repo.list_users(skip=skip, limit=per_page)
    return users

@app.get("/api/v1/users/{user_id}", response_model=UserResponse, tags=["users"])
async def get_user(
    user_id: int = Path(..., ge=1),
    user_repo: UserRepository = Depends(get_user_repo)
):
    user = await user_repo.get_by_id(user_id)
    if not user:
        raise HTTPException(404, f"User {user_id} not found")
    return user

# ====== Post Endpoints ======
@app.post("/api/v1/posts", response_model=PostResponse, status_code=201, tags=["posts"])
async def create_post(
    post_data: PostCreate,
    author_id: int = Query(..., description="Author user ID"),
    post_repo: PostRepository = Depends(get_post_repo),
    user_repo: UserRepository = Depends(get_user_repo)
):
    # Verify author exists
    author = await user_repo.get_by_id(author_id)
    if not author:
        raise HTTPException(404, f"User {author_id} not found")
    
    post = await post_repo.create({
        "title": post_data.title,
        "content": post_data.content,
        "summary": post_data.summary,
        "published": post_data.published,
        "author_id": author_id,
    })
    return post

@app.get("/api/v1/posts", response_model=dict, tags=["posts"])
async def list_posts(
    author_id: Optional[int] = Query(None),
    published_only: bool = Query(True),
    page: int = Query(1, ge=1),
    per_page: int = Query(10, ge=1, le=50),
    post_repo: PostRepository = Depends(get_post_repo)
):
    skip = (page - 1) * per_page
    
    posts, total = await asyncio.gather(
        post_repo.list_posts(author_id=author_id, published_only=published_only, skip=skip, limit=per_page),
        post_repo.count_posts(author_id=author_id, published_only=published_only)
    )
    
    total_pages = (total + per_page - 1) // per_page if total > 0 else 1
    
    return {
        "posts": posts,
        "pagination": {
            "total": total,
            "page": page,
            "per_page": per_page,
            "total_pages": total_pages,
        }
    }

@app.get("/api/v1/posts/{post_id}", response_model=PostResponse, tags=["posts"])
async def get_post(
    post_id: int = Path(..., ge=1),
    post_repo: PostRepository = Depends(get_post_repo)
):
    post = await post_repo.get_by_id(post_id)
    if not post:
        raise HTTPException(404, f"Post {post_id} not found")
    return post

@app.patch("/api/v1/posts/{post_id}", response_model=PostResponse, tags=["posts"])
async def update_post(
    post_id: int,
    update_data: PostUpdate,
    author_id: int = Query(...),
    post_repo: PostRepository = Depends(get_post_repo)
):
    post = await post_repo.get_by_id(post_id)
    if not post:
        raise HTTPException(404, f"Post {post_id} not found")
    if post.author_id != author_id:
        raise HTTPException(403, "Not authorized to update this post")
    
    updates = update_data.model_dump(exclude_unset=True)
    updated_post = await post_repo.update(post_id, updates)
    return updated_post

@app.delete("/api/v1/posts/{post_id}", status_code=204, tags=["posts"])
async def delete_post(
    post_id: int,
    author_id: int = Query(...),
    post_repo: PostRepository = Depends(get_post_repo)
):
    post = await post_repo.get_by_id(post_id)
    if not post:
        raise HTTPException(404, f"Post {post_id} not found")
    if post.author_id != author_id:
        raise HTTPException(403, "Not authorized")
    
    await post_repo.delete(post_id)
    return None

# ====== Startup ======
@app.on_event("startup")
async def startup():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    print("Database tables created!")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)
```

---

## 12. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Database Setup

**โจทย์**: สร้าง async database engine และ session สำหรับ TODO app

**เฉลย**:

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Text
from datetime import datetime, timezone

DATABASE_URL = "sqlite+aiosqlite:///./todo.db"

engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = async_sessionmaker(
    bind=engine, expire_on_commit=False
)

class Base(DeclarativeBase):
    pass

class TodoItem(Base):
    __tablename__ = "todos"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    description = Column(Text)
    completed = Column(Boolean, default=False)
    priority = Column(Integer, default=3)  # 1-5
    due_date = Column(DateTime(timezone=True))
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise

import asyncio
async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    print("TODO database initialized!")

asyncio.run(init_db())
```

### แบบฝึกหัดที่ 2: CRUD Repository

**โจทย์**: สร้าง repository pattern สำหรับ Product management

**เฉลย**:

```python
from sqlalchemy import select, update, delete, func, and_
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Optional, List

class ProductRepository:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def get(self, product_id: int):
        from models import Product
        result = await self.db.execute(
            select(Product).where(Product.id == product_id)
        )
        return result.scalar_one_or_none()
    
    async def list(
        self,
        category: Optional[str] = None,
        min_price: Optional[float] = None,
        max_price: Optional[float] = None,
        in_stock: Optional[bool] = None,
        skip: int = 0,
        limit: int = 10
    ):
        from models import Product
        query = select(Product)
        
        conditions = []
        if category:
            conditions.append(Product.category == category)
        if min_price is not None:
            conditions.append(Product.price >= min_price)
        if max_price is not None:
            conditions.append(Product.price <= max_price)
        if in_stock is not None:
            if in_stock:
                conditions.append(Product.stock > 0)
            else:
                conditions.append(Product.stock == 0)
        
        if conditions:
            query = query.where(and_(*conditions))
        
        query = query.offset(skip).limit(limit)
        result = await self.db.execute(query)
        return result.scalars().all()
    
    async def create(self, data: dict):
        from models import Product
        product = Product(**data)
        self.db.add(product)
        await self.db.flush()
        await self.db.refresh(product)
        return product
    
    async def update(self, product_id: int, data: dict):
        product = await self.get(product_id)
        if not product:
            return None
        for key, value in data.items():
            setattr(product, key, value)
        await self.db.flush()
        await self.db.refresh(product)
        return product
    
    async def delete(self, product_id: int) -> bool:
        from models import Product
        result = await self.db.execute(
            delete(Product).where(Product.id == product_id)
        )
        return result.rowcount > 0
    
    async def update_stock(self, product_id: int, quantity_delta: int):
        from models import Product
        product = await self.get(product_id)
        if not product:
            return None
        new_stock = max(0, product.stock + quantity_delta)
        product.stock = new_stock
        await self.db.flush()
        return product
```

### แบบฝึกหัดที่ 3: Relationships

**โจทย์**: สร้าง query ที่ load User พร้อม posts และ comment counts

**เฉลย**:

```python
from sqlalchemy import select, func, and_
from sqlalchemy.orm import selectinload
from sqlalchemy.ext.asyncio import AsyncSession

async def get_user_with_stats(db: AsyncSession, user_id: int):
    """ดึง user พร้อม posts และ stats"""
    from models import User, Post, Comment
    
    # User with posts (selectinload)
    result = await db.execute(
        select(User)
        .where(User.id == user_id)
        .options(selectinload(User.posts))
    )
    user = result.scalar_one_or_none()
    
    if not user:
        return None
    
    # Count published posts
    post_count_result = await db.execute(
        select(func.count(Post.id))
        .where(and_(Post.author_id == user_id, Post.published == True))
    )
    published_count = post_count_result.scalar()
    
    return {
        "user": {
            "id": user.id,
            "username": user.username,
            "email": user.email,
        },
        "stats": {
            "total_posts": len(user.posts),
            "published_posts": published_count,
        },
        "recent_posts": [
            {"id": p.id, "title": p.title, "published": p.published}
            for p in user.posts[:5]
        ]
    }
```

### แบบฝึกหัดที่ 4: Alembic Migration

**โจทย์**: สร้าง migration สำหรับเพิ่ม `profile_picture` column ใน users table

**เฉลย**:

```python
# alembic/versions/add_profile_picture.py
"""Add profile_picture to users

Revision ID: add_pic_001
Revises: initial_tables
Create Date: 2024-01-15 10:00:00
"""
from alembic import op
import sqlalchemy as sa

revision = 'add_pic_001'
down_revision = 'initial_tables'
branch_labels = None
depends_on = None

def upgrade() -> None:
    """เพิ่ม profile_picture column"""
    op.add_column(
        'users',
        sa.Column('profile_picture', sa.String(500), nullable=True)
    )
    
    # เพิ่ม index ถ้าต้องการ
    # op.create_index('ix_users_profile', 'users', ['profile_picture'])

def downgrade() -> None:
    """ลบ profile_picture column"""
    # op.drop_index('ix_users_profile', table_name='users')
    op.drop_column('users', 'profile_picture')

# Commands:
# alembic revision --autogenerate -m "Add profile_picture"  
# alembic upgrade head
# alembic downgrade -1
```

### แบบฝึกหัดที่ 5: Transaction

**โจทย์**: สร้าง order placement ที่ใช้ transaction เพื่อ:
1. ลด stock สินค้า
2. สร้าง order
3. สร้าง order items

**เฉลย**:

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update
from fastapi import HTTPException

async def place_order(
    db: AsyncSession,
    user_id: int,
    items: list  # [{"product_id": 1, "quantity": 2}, ...]
):
    """
    Place order ด้วย transaction:
    1. ตรวจสอบ stock
    2. สร้าง order
    3. สร้าง order items
    4. ลด stock
    """
    from models import Product, Order, OrderItem
    
    async with db.begin():  # Transaction
        # 1. Check và lock stock
        products = {}
        for item in items:
            result = await db.execute(
                select(Product)
                .where(Product.id == item["product_id"])
                .with_for_update()  # Lock row
            )
            product = result.scalar_one_or_none()
            
            if not product:
                raise HTTPException(404, f"Product {item['product_id']} not found")
            
            if product.stock < item["quantity"]:
                raise HTTPException(
                    400,
                    f"Insufficient stock for {product.name}. "
                    f"Available: {product.stock}, Requested: {item['quantity']}"
                )
            
            products[item["product_id"]] = product
        
        # 2. สร้าง order
        total = sum(
            products[i["product_id"]].price * i["quantity"]
            for i in items
        )
        
        order = Order(user_id=user_id, total_amount=total, status="pending")
        db.add(order)
        await db.flush()
        
        # 3. สร้าง order items และลด stock
        for item in items:
            product = products[item["product_id"]]
            
            order_item = OrderItem(
                order_id=order.id,
                product_id=item["product_id"],
                quantity=item["quantity"],
                unit_price=product.price
            )
            db.add(order_item)
            
            # ลด stock
            product.stock -= item["quantity"]
        
        await db.flush()
        await db.refresh(order)
        
        return order  # Auto-commit เมื่อออก from async with db.begin()
```

### แบบฝึกหัดที่ 6: Pagination Query

**โจทย์**: สร้าง paginated query สำหรับ posts ที่มี filtering และ sorting

**เฉลย**:

```python
from sqlalchemy import select, func, and_, desc, asc
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Optional
import math

async def paginated_posts(
    db: AsyncSession,
    author_id: Optional[int] = None,
    search: Optional[str] = None,
    published_only: bool = True,
    sort_by: str = "created_at",
    sort_order: str = "desc",
    page: int = 1,
    per_page: int = 10
):
    from models import Post
    
    # Build filter conditions
    conditions = []
    if author_id:
        conditions.append(Post.author_id == author_id)
    if search:
        conditions.append(
            Post.title.ilike(f'%{search}%') | Post.content.ilike(f'%{search}%')
        )
    if published_only:
        conditions.append(Post.published == True)
    
    where_clause = and_(*conditions) if conditions else None
    
    # Count total
    count_query = select(func.count(Post.id))
    if where_clause is not None:
        count_query = count_query.where(where_clause)
    
    total_result = await db.execute(count_query)
    total = total_result.scalar()
    
    # Sorting
    sort_column = getattr(Post, sort_by, Post.created_at)
    sort_func = desc if sort_order == "desc" else asc
    
    # Data query
    data_query = select(Post)
    if where_clause is not None:
        data_query = data_query.where(where_clause)
    data_query = (
        data_query
        .order_by(sort_func(sort_column))
        .offset((page - 1) * per_page)
        .limit(per_page)
    )
    
    result = await db.execute(data_query)
    posts = result.scalars().all()
    
    return {
        "posts": posts,
        "pagination": {
            "total": total,
            "page": page,
            "per_page": per_page,
            "total_pages": math.ceil(total / per_page) if total > 0 else 1,
            "has_next": page * per_page < total,
            "has_prev": page > 1,
        }
    }
```

### แบบฝึกหัดที่ 7: Connection Pool

**โจทย์**: Configure production-ready connection pool สำหรับ PostgreSQL

**เฉลย**:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.pool import NullPool
import os

def create_db_engine(environment: str = "production"):
    """สร้าง engine ตาม environment"""
    
    if environment == "testing":
        # Testing: ไม่ใช้ pool
        return create_async_engine(
            "sqlite+aiosqlite:///:memory:",
            poolclass=NullPool,
            echo=True
        )
    
    elif environment == "development":
        # Development: SQLite ด้วย pool เล็ก
        return create_async_engine(
            "sqlite+aiosqlite:///./dev.db",
            pool_size=5,
            max_overflow=10,
            pool_pre_ping=True,
            echo=True
        )
    
    else:  # Production
        # Production: PostgreSQL ด้วย pool ใหญ่
        db_url = os.getenv("DATABASE_URL", "postgresql+asyncpg://user:pass@localhost/mydb")
        
        return create_async_engine(
            db_url,
            pool_size=20,
            max_overflow=40,
            pool_timeout=30,
            pool_recycle=3600,
            pool_pre_ping=True,
            connect_args={
                "command_timeout": 30,
                "timeout": 10,
            },
            echo=False
        )

env = os.getenv("ENVIRONMENT", "production")
engine = create_db_engine(env)
SessionFactory = async_sessionmaker(bind=engine, expire_on_commit=False)

async def get_db():
    async with SessionFactory() as session:
        try:
            yield session
        except Exception:
            await session.rollback()
            raise
```

### แบบฝึกหัดที่ 8: Complete Task Manager API

**โจทย์**: สร้าง complete Task Manager API ด้วย SQLAlchemy Async ที่มี:
- User management
- Task CRUD
- Categories
- Pagination

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException, Query
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Text, ForeignKey, select, func, delete
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime, timezone
import asyncio

# Database
DATABASE_URL = "sqlite+aiosqlite:///./tasks.db"
engine = create_async_engine(DATABASE_URL, echo=False)
SessionLocal = async_sessionmaker(bind=engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

# Models
class Category(Base):
    __tablename__ = "categories"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False)
    color = Column(String(7), default="#000000")  # hex color

class Task(Base):
    __tablename__ = "tasks"
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    description = Column(Text)
    completed = Column(Boolean, default=False)
    priority = Column(Integer, default=3)  # 1-5
    category_id = Column(Integer, ForeignKey("categories.id"))
    created_at = Column(DateTime(timezone=True), default=lambda: datetime.now(timezone.utc))
    due_date = Column(DateTime(timezone=True))

# Schemas
class TaskCreate(BaseModel):
    title: str = Field(..., min_length=2, max_length=200)
    description: Optional[str] = None
    priority: int = Field(default=3, ge=1, le=5)
    category_id: Optional[int] = None
    due_date: Optional[datetime] = None

class TaskResponse(BaseModel):
    id: int
    title: str
    description: Optional[str]
    completed: bool
    priority: int
    category_id: Optional[int]
    created_at: datetime
    due_date: Optional[datetime]
    class Config:
        from_attributes = True

# Dependencies
async def get_db():
    async with SessionLocal() as session:
        try:
            yield session
        except:
            await session.rollback()
            raise

# App
app = FastAPI(title="Task Manager API")

@app.get("/tasks", response_model=dict)
async def list_tasks(
    completed: Optional[bool] = None,
    priority: Optional[int] = Query(None, ge=1, le=5),
    category_id: Optional[int] = None,
    page: int = Query(1, ge=1),
    per_page: int = Query(10, ge=1, le=50),
    db: AsyncSession = Depends(get_db)
):
    query = select(Task)
    if completed is not None:
        query = query.where(Task.completed == completed)
    if priority:
        query = query.where(Task.priority == priority)
    if category_id:
        query = query.where(Task.category_id == category_id)
    
    count_result = await db.execute(select(func.count()).select_from(query.subquery()))
    total = count_result.scalar()
    
    tasks_result = await db.execute(
        query.offset((page-1)*per_page).limit(per_page).order_by(Task.created_at.desc())
    )
    tasks = tasks_result.scalars().all()
    
    return {
        "tasks": tasks,
        "total": total,
        "page": page,
        "per_page": per_page
    }

@app.post("/tasks", response_model=TaskResponse, status_code=201)
async def create_task(task_data: TaskCreate, db: AsyncSession = Depends(get_db)):
    task = Task(**task_data.model_dump())
    db.add(task)
    await db.commit()
    await db.refresh(task)
    return task

@app.patch("/tasks/{task_id}/complete", response_model=TaskResponse)
async def complete_task(task_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Task).where(Task.id == task_id))
    task = result.scalar_one_or_none()
    if not task:
        raise HTTPException(404, "Task not found")
    task.completed = True
    await db.commit()
    await db.refresh(task)
    return task

@app.delete("/tasks/{task_id}", status_code=204)
async def delete_task(task_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Task).where(Task.id == task_id))
    if not result.scalar_one_or_none():
        raise HTTPException(404, "Task not found")
    await db.execute(delete(Task).where(Task.id == task_id))
    await db.commit()

@app.on_event("startup")
async def startup():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    print("Task Manager API ready!")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, reload=True)
```

---

## สรุป

ใน Part 59 เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียนรู้ |
|--------|-------------------|
| SQLAlchemy Async | Async engine, session factory |
| Session Management | Session lifecycle, dependency |
| Repository Pattern | Encapsulate DB operations |
| CRUD Operations | Select, Insert, Update, Delete async |
| SQLModel | Combined Pydantic + SQLAlchemy |
| Relationships | selectinload, joinedload, lazy loading |
| Alembic | Database migrations, versioning |
| Connection Pooling | Pool configuration สำหรับ production |
| Transactions | Atomic operations, savepoints |
| Multiple Databases | Primary, replica, analytics |

### Stack ที่แนะนำ

```bash
# Development
pip install fastapi uvicorn sqlalchemy[asyncio] aiosqlite alembic pydantic

# Production
pip install fastapi uvicorn[standard] sqlalchemy[asyncio] asyncpg alembic pydantic redis
```

### ขั้นตอนต่อไป

- **Part 60**: FastAPI - Authentication, JWT & OAuth2
