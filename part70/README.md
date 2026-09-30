# Part 70: Project - Full-Stack Blog API with FastAPI

## บทนำ

ในบทนี้เราจะสร้าง Full-Stack Blog API ที่สมบูรณ์ด้วย FastAPI โดยครอบคลุม features ทั้งหมดที่ production-ready application ต้องการ ตั้งแต่ Authentication, CRUD operations, Search, Caching ไปจนถึง Docker deployment

## สารบัญ

1. [Project Overview](#1-project-overview)
2. [Project Structure](#2-project-structure)
3. [Database Models](#3-database-models)
4. [Authentication System](#4-authentication-system)
5. [Blog Posts CRUD](#5-blog-posts-crud)
6. [Comments System](#6-comments-system)
7. [Tags and Categories](#7-tags-and-categories)
8. [Image Upload](#8-image-upload)
9. [Search Functionality](#9-search-functionality)
10. [Caching with Redis](#10-caching-with-redis)
11. [Email Notifications](#11-email-notifications)
12. [Admin Endpoints](#12-admin-endpoints)
13. [Testing](#13-testing)
14. [Docker Setup](#14-docker-setup)

---

## 1. Project Overview

### Features

```
Full-Stack Blog API:
├── Authentication
│   ├── User registration with email verification
│   ├── JWT login/logout
│   ├── Password reset via email
│   └── Profile management
├── Blog Posts
│   ├── CRUD operations
│   ├── Rich text content
│   ├── Featured images
│   ├── Draft/Published states
│   └── View count tracking
├── Comments
│   ├── Nested comments
│   ├── Comment moderation
│   └── Like system
├── Tags & Categories
│   ├── Tag management
│   └── Category hierarchy
├── Search
│   ├── Full-text search
│   └── Filtered search
├── Performance
│   ├── Redis caching
│   ├── Pagination
│   └── Rate limiting
├── Admin
│   ├── User management
│   ├── Content moderation
│   └── Analytics
└── Infrastructure
    ├── PostgreSQL database
    ├── Redis cache
    ├── MinIO file storage
    └── Docker Compose
```

### Dependencies

```bash
# requirements.txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
alembic==1.12.1
asyncpg==0.29.0
pydantic==2.5.2
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
Pillow==10.1.0
redis==5.0.1
celery==5.3.6
fastapi-mail==1.4.1
sqlalchemy-searchable==1.4.1
python-slugify==8.0.1
aiofiles==23.2.1
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.2
faker==20.1.0
```

---

## 2. Project Structure

```
blog_api/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app entry point
│   ├── config.py                # Settings
│   ├── database.py              # Database connection
│   ├── dependencies.py          # Common dependencies
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   ├── comment.py
│   │   ├── tag.py
│   │   └── category.py
│   │
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   ├── comment.py
│   │   ├── tag.py
│   │   └── common.py
│   │
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── users.py
│   │   ├── posts.py
│   │   ├── comments.py
│   │   ├── tags.py
│   │   ├── categories.py
│   │   ├── upload.py
│   │   ├── search.py
│   │   └── admin.py
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── post.py
│   │   ├── cache.py
│   │   ├── email.py
│   │   ├── storage.py
│   │   └── search.py
│   │
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── rate_limit.py
│   │   └── logging.py
│   │
│   └── utils/
│       ├── __init__.py
│       ├── security.py
│       └── pagination.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_auth.py
│   ├── test_posts.py
│   ├── test_comments.py
│   └── test_search.py
│
├── alembic/
│   ├── env.py
│   └── versions/
│
├── docker-compose.yml
├── Dockerfile
├── .env.example
└── requirements.txt
```

---

## 3. Database Models

### SQLAlchemy Models

```python
# app/models/user.py
from sqlalchemy import (
    Column, Integer, String, Boolean, DateTime,
    Text, ForeignKey, Table
)
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    username = Column(String(100), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    
    # Profile
    first_name = Column(String(100), default="")
    last_name = Column(String(100), default="")
    bio = Column(Text, default="")
    avatar_url = Column(String(500), nullable=True)
    website = Column(String(255), nullable=True)
    
    # Status
    is_active = Column(Boolean, default=True)
    is_verified = Column(Boolean, default=False)
    is_admin = Column(Boolean, default=False)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    last_login = Column(DateTime(timezone=True), nullable=True)
    
    # Relationships
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    comments = relationship("Comment", back_populates="author", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User {self.username}>"
```

```python
# app/models/post.py
from sqlalchemy import (
    Column, Integer, String, Boolean, DateTime,
    Text, ForeignKey, Enum, Table
)
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
import enum
from app.database import Base

class PostStatus(str, enum.Enum):
    DRAFT = "draft"
    PUBLISHED = "published"
    ARCHIVED = "archived"

# Many-to-many table สำหรับ Post และ Tag
post_tags = Table(
    'post_tags',
    Base.metadata,
    Column('post_id', Integer, ForeignKey('posts.id'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id'), primary_key=True)
)

class Post(Base):
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(300), nullable=False)
    slug = Column(String(350), unique=True, index=True, nullable=False)
    content = Column(Text, nullable=False)
    excerpt = Column(Text, default="")
    featured_image_url = Column(String(500), nullable=True)
    
    # Status & visibility
    status = Column(
        Enum(PostStatus),
        default=PostStatus.DRAFT,
        nullable=False
    )
    
    # Metrics
    view_count = Column(Integer, default=0)
    
    # Foreign keys
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    published_at = Column(DateTime(timezone=True), nullable=True)
    
    # Relationships
    author = relationship("User", back_populates="posts")
    category = relationship("Category", back_populates="posts")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")
    comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Post {self.title[:50]}>"
```

```python
# app/models/comment.py
from sqlalchemy import Column, Integer, Boolean, DateTime, Text, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class Comment(Base):
    __tablename__ = "comments"
    
    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    is_approved = Column(Boolean, default=True)
    is_deleted = Column(Boolean, default=False)
    like_count = Column(Integer, default=0)
    
    # Foreign keys
    post_id = Column(Integer, ForeignKey("posts.id"), nullable=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    parent_id = Column(Integer, ForeignKey("comments.id"), nullable=True)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # Relationships
    post = relationship("Post", back_populates="comments")
    author = relationship("User", back_populates="comments")
    replies = relationship("Comment", back_populates="parent")
    parent = relationship("Comment", back_populates="replies", remote_side=[id])
    
    def __repr__(self):
        return f"<Comment {self.id}>"

# app/models/tag.py
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base
from .post import post_tags

class Tag(Base):
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False)
    slug = Column(String(120), unique=True, index=True, nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    posts = relationship("Post", secondary=post_tags, back_populates="tags")

# app/models/category.py
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class Category(Base):
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    slug = Column(String(120), unique=True, index=True, nullable=False)
    description = Column(Text, default="")
    parent_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    parent = relationship("Category", back_populates="children", remote_side=[id])
    children = relationship("Category", back_populates="parent")
    posts = relationship("Post", back_populates="category")
```

### Database Setup

```python
# app/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from app.config import settings

DATABASE_URL = settings.database_url.replace(
    "postgresql://", "postgresql+asyncpg://"
)

engine = create_async_engine(
    DATABASE_URL,
    echo=settings.debug,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False,
)

Base = declarative_base()

async def get_db():
    """Dependency สำหรับ database session"""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

async def init_db():
    """สร้าง tables ทั้งหมด"""
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
```

---

## 4. Authentication System

### Config และ Security

```python
# app/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    # App
    app_name: str = "Blog API"
    debug: bool = False
    api_v1_prefix: str = "/api/v1"
    
    # Database
    database_url: str = "postgresql://user:password@localhost/blogdb"
    
    # JWT
    secret_key: str = "your-secret-key-change-in-production"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 60
    refresh_token_expire_days: int = 30
    
    # Redis
    redis_url: str = "redis://localhost:6379"
    
    # Email
    mail_username: str = ""
    mail_password: str = ""
    mail_from: str = "noreply@example.com"
    mail_server: str = "smtp.gmail.com"
    mail_port: int = 587
    
    # Storage
    storage_type: str = "local"  # 'local' or 's3'
    upload_dir: str = "uploads"
    
    # Rate limiting
    rate_limit_requests: int = 100
    rate_limit_window: int = 60
    
    class Config:
        env_file = ".env"

@lru_cache()
def get_settings():
    return Settings()

settings = get_settings()

# app/utils/security.py
from datetime import datetime, timedelta
from typing import Optional, Union
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (
        expires_delta or timedelta(minutes=settings.access_token_expire_minutes)
    )
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)

def create_refresh_token(user_id: int) -> str:
    expire = datetime.utcnow() + timedelta(days=settings.refresh_token_expire_days)
    data = {"sub": str(user_id), "exp": expire, "type": "refresh"}
    return jwt.encode(data, settings.secret_key, algorithm=settings.algorithm)

def decode_token(token: str) -> Optional[dict]:
    try:
        payload = jwt.decode(
            token,
            settings.secret_key,
            algorithms=[settings.algorithm]
        )
        return payload
    except JWTError:
        return None
```

### Auth Schemas

```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional
from datetime import datetime

class UserCreate(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    password: str = Field(..., min_length=8)
    first_name: str = Field(default="", max_length=100)
    last_name: str = Field(default="", max_length=100)
    
    @validator('username')
    def username_alphanumeric(cls, v):
        if not v.replace('_', '').replace('-', '').isalnum():
            raise ValueError('Username must be alphanumeric (- and _ allowed)')
        return v.lower()
    
    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        return v

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: int
    email: str
    username: str
    first_name: str
    last_name: str
    bio: str
    avatar_url: Optional[str]
    is_active: bool
    is_verified: bool
    is_admin: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

class UserUpdate(BaseModel):
    first_name: Optional[str] = Field(None, max_length=100)
    last_name: Optional[str] = Field(None, max_length=100)
    bio: Optional[str] = Field(None, max_length=500)
    website: Optional[str] = None

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    user: UserResponse

class PasswordReset(BaseModel):
    email: EmailStr

class PasswordResetConfirm(BaseModel):
    token: str
    new_password: str = Field(..., min_length=8)
    new_password_confirm: str
    
    @validator('new_password_confirm')
    def passwords_match(cls, v, values):
        if 'new_password' in values and v != values['new_password']:
            raise ValueError('Passwords do not match')
        return v
```

### Auth Router

```python
# app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status, BackgroundTasks
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from typing import Annotated

from app.database import get_db
from app.models.user import User
from app.schemas.user import (
    UserCreate, UserLogin, UserResponse, TokenResponse,
    PasswordReset, PasswordResetConfirm, UserUpdate
)
from app.utils.security import (
    verify_password, get_password_hash,
    create_access_token, create_refresh_token, decode_token
)
from app.services.email import send_verification_email, send_password_reset_email
from app.services.cache import cache_service
from app.dependencies import get_current_user

router = APIRouter(prefix="/auth", tags=["Authentication"])
security = HTTPBearer()

@router.post("/register", response_model=dict, status_code=status.HTTP_201_CREATED)
async def register(
    user_data: UserCreate,
    background_tasks: BackgroundTasks,
    db: AsyncSession = Depends(get_db)
):
    """Register new user"""
    # Check existing email
    result = await db.execute(
        select(User).where(User.email == user_data.email)
    )
    if result.scalar_one_or_none():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Check existing username
    result = await db.execute(
        select(User).where(User.username == user_data.username)
    )
    if result.scalar_one_or_none():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Username already taken"
        )
    
    # Create user
    user = User(
        email=user_data.email,
        username=user_data.username,
        hashed_password=get_password_hash(user_data.password),
        first_name=user_data.first_name,
        last_name=user_data.last_name,
        is_verified=False,
    )
    
    db.add(user)
    await db.commit()
    await db.refresh(user)
    
    # Send verification email in background
    background_tasks.add_task(
        send_verification_email,
        user.email,
        user.id,
        user.username
    )
    
    return {
        "message": "Registration successful. Check your email.",
        "user_id": user.id
    }

@router.post("/login", response_model=TokenResponse)
async def login(
    credentials: UserLogin,
    db: AsyncSession = Depends(get_db)
):
    """Login with email and password"""
    result = await db.execute(
        select(User).where(User.email == credentials.email)
    )
    user = result.scalar_one_or_none()
    
    if not user or not verify_password(credentials.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Account disabled"
        )
    
    if not user.is_verified:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Please verify your email first"
        )
    
    # Update last login
    from sqlalchemy import update
    from datetime import datetime
    await db.execute(
        update(User)
        .where(User.id == user.id)
        .values(last_login=datetime.utcnow())
    )
    await db.commit()
    
    # Create tokens
    access_token = create_access_token({"sub": str(user.id)})
    refresh_token = create_refresh_token(user.id)
    
    # Store refresh token in Redis
    await cache_service.set(
        f"refresh_token:{user.id}",
        refresh_token,
        expire=60 * 60 * 24 * 30  # 30 days
    )
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token,
        user=UserResponse.model_validate(user)
    )

@router.post("/logout")
async def logout(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Logout - blacklist token"""
    token = credentials.credentials
    payload = decode_token(token)
    
    if payload:
        # Blacklist token
        exp = payload.get("exp", 0)
        from datetime import datetime
        ttl = max(0, int(exp - datetime.utcnow().timestamp()))
        await cache_service.set(
            f"blacklist:{token}",
            "1",
            expire=ttl
        )
    
    # Remove refresh token
    await cache_service.delete(f"refresh_token:{current_user.id}")
    
    return {"message": "Logged out successfully"}

@router.post("/refresh", response_model=dict)
async def refresh_token(
    refresh_token_str: str,
    db: AsyncSession = Depends(get_db)
):
    """Refresh access token"""
    payload = decode_token(refresh_token_str)
    
    if not payload or payload.get("type") != "refresh":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid refresh token"
        )
    
    user_id = int(payload.get("sub", 0))
    
    # Verify stored refresh token
    stored_token = await cache_service.get(f"refresh_token:{user_id}")
    if not stored_token or stored_token != refresh_token_str:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Refresh token invalid or expired"
        )
    
    # Create new access token
    access_token = create_access_token({"sub": str(user_id)})
    
    return {"access_token": access_token, "token_type": "bearer"}

@router.get("/verify-email/{token}")
async def verify_email(
    token: str,
    db: AsyncSession = Depends(get_db)
):
    """Verify email address"""
    user_id_str = await cache_service.get(f"email_verify:{token}")
    
    if not user_id_str:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid or expired verification link"
        )
    
    result = await db.execute(
        select(User).where(User.id == int(user_id_str))
    )
    user = result.scalar_one_or_none()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    user.is_verified = True
    await db.commit()
    
    await cache_service.delete(f"email_verify:{token}")
    
    return {"message": "Email verified successfully"}

@router.post("/forgot-password")
async def forgot_password(
    data: PasswordReset,
    background_tasks: BackgroundTasks,
    db: AsyncSession = Depends(get_db)
):
    """Request password reset"""
    result = await db.execute(
        select(User).where(User.email == data.email)
    )
    user = result.scalar_one_or_none()
    
    if user:
        background_tasks.add_task(
            send_password_reset_email,
            user.email,
            user.id
        )
    
    # ตอบ success เสมอ
    return {"message": "If email exists, reset link sent"}

@router.post("/reset-password")
async def reset_password(
    data: PasswordResetConfirm,
    db: AsyncSession = Depends(get_db)
):
    """Reset password with token"""
    user_id_str = await cache_service.get(f"password_reset:{data.token}")
    
    if not user_id_str:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid or expired token"
        )
    
    result = await db.execute(
        select(User).where(User.id == int(user_id_str))
    )
    user = result.scalar_one_or_none()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    user.hashed_password = get_password_hash(data.new_password)
    await db.commit()
    
    await cache_service.delete(f"password_reset:{data.token}")
    
    return {"message": "Password reset successfully"}
```

### Dependencies

```python
# app/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

from app.database import get_db
from app.models.user import User
from app.utils.security import decode_token
from app.services.cache import cache_service

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db)
) -> User:
    """Extract current user จาก JWT token"""
    token = credentials.credentials
    
    # Check blacklist
    is_blacklisted = await cache_service.get(f"blacklist:{token}")
    if is_blacklisted:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token has been revoked"
        )
    
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token"
        )
    
    user_id = int(payload.get("sub", 0))
    
    result = await db.execute(
        select(User).where(User.id == user_id)
    )
    user = result.scalar_one_or_none()
    
    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found or inactive"
        )
    
    return user

async def get_current_admin(
    current_user: User = Depends(get_current_user)
) -> User:
    """Require admin user"""
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Admin access required"
        )
    return current_user

async def get_optional_user(
    credentials: HTTPAuthorizationCredentials = Depends(
        HTTPBearer(auto_error=False)
    ),
    db: AsyncSession = Depends(get_db)
) -> User | None:
    """Optional authentication"""
    if not credentials:
        return None
    
    try:
        return await get_current_user(credentials, db)
    except HTTPException:
        return None
```

---

## 5. Blog Posts CRUD

### Post Schemas

```python
# app/schemas/post.py
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from datetime import datetime
from app.models.post import PostStatus
from app.schemas.user import UserResponse

class PostCreate(BaseModel):
    title: str = Field(..., min_length=5, max_length=300)
    content: str = Field(..., min_length=50)
    excerpt: str = Field(default="", max_length=500)
    category_id: Optional[int] = None
    tag_ids: Optional[List[int]] = []
    status: PostStatus = PostStatus.DRAFT
    
    @validator('title')
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty')
        return v.strip()

class PostUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=5, max_length=300)
    content: Optional[str] = Field(None, min_length=50)
    excerpt: Optional[str] = Field(None, max_length=500)
    category_id: Optional[int] = None
    tag_ids: Optional[List[int]] = None
    status: Optional[PostStatus] = None

class PostResponse(BaseModel):
    id: int
    title: str
    slug: str
    content: str
    excerpt: str
    featured_image_url: Optional[str]
    status: PostStatus
    view_count: int
    author: UserResponse
    category_id: Optional[int]
    tags: List[dict] = []
    comment_count: int = 0
    created_at: datetime
    updated_at: Optional[datetime]
    published_at: Optional[datetime]
    
    class Config:
        from_attributes = True

class PostListResponse(BaseModel):
    id: int
    title: str
    slug: str
    excerpt: str
    featured_image_url: Optional[str]
    status: PostStatus
    view_count: int
    author: UserResponse
    comment_count: int = 0
    created_at: datetime
    
    class Config:
        from_attributes = True

class PaginatedPosts(BaseModel):
    items: List[PostListResponse]
    total: int
    page: int
    page_size: int
    total_pages: int
    has_next: bool
    has_previous: bool
```

### Posts Router

```python
# app/routers/posts.py
from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func, update, delete
from sqlalchemy.orm import selectinload, joinedload
from typing import Optional, List
from datetime import datetime
from slugify import slugify

from app.database import get_db
from app.models.post import Post, PostStatus, post_tags
from app.models.tag import Tag
from app.models.user import User
from app.schemas.post import (
    PostCreate, PostUpdate, PostResponse,
    PostListResponse, PaginatedPosts
)
from app.dependencies import get_current_user, get_optional_user
from app.services.cache import cache_service

router = APIRouter(prefix="/posts", tags=["Posts"])

async def generate_unique_slug(title: str, db: AsyncSession, exclude_id: int = None) -> str:
    """สร้าง unique slug"""
    base_slug = slugify(title)
    slug = base_slug
    counter = 1
    
    while True:
        query = select(Post).where(Post.slug == slug)
        if exclude_id:
            query = query.where(Post.id != exclude_id)
        
        result = await db.execute(query)
        if not result.scalar_one_or_none():
            break
        
        slug = f"{base_slug}-{counter}"
        counter += 1
    
    return slug

@router.get("/", response_model=PaginatedPosts)
async def list_posts(
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=100),
    status: Optional[PostStatus] = None,
    category_id: Optional[int] = None,
    author_id: Optional[int] = None,
    tag: Optional[str] = None,
    order_by: str = Query("created_at", regex="^(created_at|view_count|title)$"),
    order_dir: str = Query("desc", regex="^(asc|desc)$"),
    db: AsyncSession = Depends(get_db),
    current_user: Optional[User] = Depends(get_optional_user)
):
    """List posts ด้วย pagination และ filtering"""
    
    # Cache key
    cache_key = f"posts:list:{page}:{page_size}:{status}:{category_id}:{author_id}:{tag}:{order_by}:{order_dir}"
    cached = await cache_service.get(cache_key)
    if cached:
        return cached
    
    query = select(Post).options(
        selectinload(Post.author),
        selectinload(Post.tags),
    )
    
    # Filters
    if status:
        query = query.where(Post.status == status)
    elif not (current_user and current_user.is_admin):
        query = query.where(Post.status == PostStatus.PUBLISHED)
    
    if category_id:
        query = query.where(Post.category_id == category_id)
    
    if author_id:
        query = query.where(Post.author_id == author_id)
    
    if tag:
        query = query.join(post_tags).join(Tag).where(Tag.slug == tag)
    
    # Count
    count_query = select(func.count()).select_from(query.subquery())
    total = await db.scalar(count_query)
    
    # Ordering
    order_col = getattr(Post, order_by)
    if order_dir == "desc":
        order_col = order_col.desc()
    
    # Pagination
    offset = (page - 1) * page_size
    query = query.order_by(order_col).offset(offset).limit(page_size)
    
    result = await db.execute(query)
    posts = result.scalars().all()
    
    total_pages = (total + page_size - 1) // page_size
    
    response = PaginatedPosts(
        items=[PostListResponse.model_validate(p) for p in posts],
        total=total,
        page=page,
        page_size=page_size,
        total_pages=total_pages,
        has_next=page < total_pages,
        has_previous=page > 1
    )
    
    await cache_service.set(cache_key, response.model_dump(), expire=300)
    
    return response

@router.post("/", response_model=PostResponse, status_code=status.HTTP_201_CREATED)
async def create_post(
    post_data: PostCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Create new post"""
    
    slug = await generate_unique_slug(post_data.title, db)
    
    post = Post(
        title=post_data.title,
        slug=slug,
        content=post_data.content,
        excerpt=post_data.excerpt or post_data.content[:200],
        author_id=current_user.id,
        category_id=post_data.category_id,
        status=post_data.status,
    )
    
    if post_data.status == PostStatus.PUBLISHED:
        post.published_at = datetime.utcnow()
    
    db.add(post)
    await db.flush()
    
    # Set tags
    if post_data.tag_ids:
        result = await db.execute(
            select(Tag).where(Tag.id.in_(post_data.tag_ids))
        )
        tags = result.scalars().all()
        post.tags = tags
    
    await db.commit()
    await db.refresh(post)
    
    # Load relationships
    result = await db.execute(
        select(Post)
        .options(selectinload(Post.author), selectinload(Post.tags))
        .where(Post.id == post.id)
    )
    post = result.scalar_one()
    
    # Invalidate cache
    await cache_service.delete_pattern("posts:list:*")
    
    return PostResponse.model_validate(post)

@router.get("/{slug}", response_model=PostResponse)
async def get_post(
    slug: str,
    db: AsyncSession = Depends(get_db),
    current_user: Optional[User] = Depends(get_optional_user)
):
    """Get post by slug"""
    
    # Try cache
    cache_key = f"post:{slug}"
    cached = await cache_service.get(cache_key)
    if cached:
        return cached
    
    result = await db.execute(
        select(Post)
        .options(
            selectinload(Post.author),
            selectinload(Post.tags),
            selectinload(Post.category)
        )
        .where(Post.slug == slug)
    )
    post = result.scalar_one_or_none()
    
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    
    if post.status != PostStatus.PUBLISHED:
        if not current_user or (
            current_user.id != post.author_id and not current_user.is_admin
        ):
            raise HTTPException(status_code=404, detail="Post not found")
    
    # Increment view count
    await db.execute(
        update(Post)
        .where(Post.id == post.id)
        .values(view_count=Post.view_count + 1)
    )
    await db.commit()
    
    response = PostResponse.model_validate(post)
    await cache_service.set(cache_key, response.model_dump(), expire=60)
    
    return response

@router.put("/{post_id}", response_model=PostResponse)
async def update_post(
    post_id: int,
    post_data: PostUpdate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Update post"""
    
    result = await db.execute(
        select(Post).where(Post.id == post_id)
    )
    post = result.scalar_one_or_none()
    
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    
    if post.author_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Permission denied")
    
    # Update fields
    if post_data.title is not None:
        post.title = post_data.title
        post.slug = await generate_unique_slug(post_data.title, db, exclude_id=post_id)
    
    if post_data.content is not None:
        post.content = post_data.content
    
    if post_data.excerpt is not None:
        post.excerpt = post_data.excerpt
    
    if post_data.category_id is not None:
        post.category_id = post_data.category_id
    
    if post_data.status is not None:
        if post_data.status == PostStatus.PUBLISHED and post.status != PostStatus.PUBLISHED:
            post.published_at = datetime.utcnow()
        post.status = post_data.status
    
    if post_data.tag_ids is not None:
        result = await db.execute(
            select(Tag).where(Tag.id.in_(post_data.tag_ids))
        )
        post.tags = result.scalars().all()
    
    await db.commit()
    
    # Invalidate caches
    await cache_service.delete(f"post:{post.slug}")
    await cache_service.delete_pattern("posts:list:*")
    
    await db.refresh(post)
    return PostResponse.model_validate(post)

@router.delete("/{post_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_post(
    post_id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Delete post"""
    
    result = await db.execute(
        select(Post).where(Post.id == post_id)
    )
    post = result.scalar_one_or_none()
    
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    
    if post.author_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Permission denied")
    
    await db.delete(post)
    await db.commit()
    
    await cache_service.delete(f"post:{post.slug}")
    await cache_service.delete_pattern("posts:list:*")
```

---

## 6. Comments System

```python
# app/routers/comments.py
from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from sqlalchemy.orm import selectinload
from typing import Optional, List

from app.database import get_db
from app.models.comment import Comment
from app.models.post import Post
from app.models.user import User
from app.dependencies import get_current_user, get_optional_user
from pydantic import BaseModel, Field

router = APIRouter(prefix="/comments", tags=["Comments"])

class CommentCreate(BaseModel):
    content: str = Field(..., min_length=1, max_length=2000)
    parent_id: Optional[int] = None

class CommentResponse(BaseModel):
    id: int
    content: str
    post_id: int
    author_id: int
    author_username: str
    parent_id: Optional[int]
    like_count: int
    created_at: str
    replies: List['CommentResponse'] = []
    
    class Config:
        from_attributes = True

@router.post("/posts/{post_id}/comments", response_model=CommentResponse, status_code=201)
async def create_comment(
    post_id: int,
    comment_data: CommentCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Create comment on post"""
    
    result = await db.execute(
        select(Post).where(Post.id == post_id)
    )
    post = result.scalar_one_or_none()
    
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    
    parent = None
    if comment_data.parent_id:
        result = await db.execute(
            select(Comment).where(
                Comment.id == comment_data.parent_id,
                Comment.post_id == post_id
            )
        )
        parent = result.scalar_one_or_none()
        if not parent:
            raise HTTPException(status_code=404, detail="Parent comment not found")
        
        if parent.parent_id:
            raise HTTPException(
                status_code=400,
                detail="Cannot reply to a reply"
            )
    
    comment = Comment(
        content=comment_data.content,
        post_id=post_id,
        author_id=current_user.id,
        parent_id=comment_data.parent_id,
    )
    
    db.add(comment)
    await db.commit()
    await db.refresh(comment)
    
    return {
        'id': comment.id,
        'content': comment.content,
        'post_id': comment.post_id,
        'author_id': comment.author_id,
        'author_username': current_user.username,
        'parent_id': comment.parent_id,
        'like_count': 0,
        'created_at': comment.created_at.isoformat(),
        'replies': [],
    }

@router.get("/posts/{post_id}/comments")
async def get_post_comments(
    post_id: int,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    db: AsyncSession = Depends(get_db)
):
    """Get comments for a post"""
    
    result = await db.execute(
        select(Comment)
        .options(
            selectinload(Comment.author),
            selectinload(Comment.replies).selectinload(Comment.author)
        )
        .where(
            Comment.post_id == post_id,
            Comment.parent_id == None,
            Comment.is_approved == True,
            Comment.is_deleted == False
        )
        .order_by(Comment.created_at.desc())
        .offset((page - 1) * page_size)
        .limit(page_size)
    )
    comments = result.scalars().all()
    
    return [{
        'id': c.id,
        'content': c.content,
        'author_username': c.author.username,
        'like_count': c.like_count,
        'created_at': c.created_at.isoformat(),
        'replies': [{
            'id': r.id,
            'content': r.content,
            'author_username': r.author.username,
            'created_at': r.created_at.isoformat(),
        } for r in c.replies if not r.is_deleted]
    } for c in comments]

@router.delete("/{comment_id}", status_code=204)
async def delete_comment(
    comment_id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """Delete comment"""
    
    result = await db.execute(
        select(Comment).where(Comment.id == comment_id)
    )
    comment = result.scalar_one_or_none()
    
    if not comment:
        raise HTTPException(status_code=404, detail="Comment not found")
    
    if comment.author_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Permission denied")
    
    comment.is_deleted = True
    comment.content = "[deleted]"
    await db.commit()
```

---

## 7. Tags and Categories

```python
# app/routers/tags.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from slugify import slugify
from pydantic import BaseModel, Field

from app.database import get_db
from app.models.tag import Tag
from app.dependencies import get_current_admin

router = APIRouter(prefix="/tags", tags=["Tags"])

class TagCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)

class TagResponse(BaseModel):
    id: int
    name: str
    slug: str
    post_count: int = 0
    
    class Config:
        from_attributes = True

@router.get("/", response_model=list)
async def list_tags(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Tag).order_by(Tag.name))
    tags = result.scalars().all()
    return [{"id": t.id, "name": t.name, "slug": t.slug} for t in tags]

@router.post("/", response_model=TagResponse, status_code=201)
async def create_tag(
    tag_data: TagCreate,
    current_admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    slug = slugify(tag_data.name)
    
    result = await db.execute(select(Tag).where(Tag.slug == slug))
    if result.scalar_one_or_none():
        raise HTTPException(status_code=400, detail="Tag already exists")
    
    tag = Tag(name=tag_data.name, slug=slug)
    db.add(tag)
    await db.commit()
    await db.refresh(tag)
    
    return TagResponse(id=tag.id, name=tag.name, slug=tag.slug)

@router.delete("/{tag_id}", status_code=204)
async def delete_tag(
    tag_id: int,
    current_admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(select(Tag).where(Tag.id == tag_id))
    tag = result.scalar_one_or_none()
    
    if not tag:
        raise HTTPException(status_code=404, detail="Tag not found")
    
    await db.delete(tag)
    await db.commit()
```

---

## 8. Image Upload

```python
# app/routers/upload.py
import os
import uuid
import aiofiles
from fastapi import APIRouter, Depends, HTTPException, UploadFile, File
from PIL import Image
import io

from app.dependencies import get_current_user
from app.models.user import User
from app.config import settings

router = APIRouter(prefix="/upload", tags=["Upload"])

ALLOWED_CONTENT_TYPES = ["image/jpeg", "image/png", "image/gif", "image/webp"]
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

@router.post("/image")
async def upload_image(
    file: UploadFile = File(...),
    current_user: User = Depends(get_current_user)
):
    """Upload image file"""
    
    # Validate content type
    if file.content_type not in ALLOWED_CONTENT_TYPES:
        raise HTTPException(
            status_code=400,
            detail=f"File type {file.content_type} not allowed"
        )
    
    # Read file
    contents = await file.read()
    
    # Validate size
    if len(contents) > MAX_FILE_SIZE:
        raise HTTPException(
            status_code=400,
            detail=f"File too large. Max {MAX_FILE_SIZE // 1024 // 1024}MB"
        )
    
    # Validate it's actually an image
    try:
        img = Image.open(io.BytesIO(contents))
        img.verify()
        
        # Reset and open again for processing
        img = Image.open(io.BytesIO(contents))
        
        # Auto-rotate based on EXIF
        try:
            from PIL import ImageOps
            img = ImageOps.exif_transpose(img)
        except Exception:
            pass
        
        # Convert to RGB if needed
        if img.mode in ('RGBA', 'LA', 'P'):
            background = Image.new('RGB', img.size, (255, 255, 255))
            if img.mode == 'P':
                img = img.convert('RGBA')
            background.paste(img, mask=img.split()[-1] if img.mode == 'RGBA' else None)
            img = background
        
        # Resize if too large
        max_dimension = 2000
        if max(img.size) > max_dimension:
            img.thumbnail((max_dimension, max_dimension), Image.Resampling.LANCZOS)
    
    except Exception:
        raise HTTPException(status_code=400, detail="Invalid image file")
    
    # Generate filename
    ext = os.path.splitext(file.filename)[1].lower() or '.jpg'
    filename = f"{uuid.uuid4()}{ext}"
    
    # Save file
    upload_dir = os.path.join(settings.upload_dir, "images")
    os.makedirs(upload_dir, exist_ok=True)
    
    file_path = os.path.join(upload_dir, filename)
    
    output = io.BytesIO()
    if ext in ['.jpg', '.jpeg']:
        img.save(output, format='JPEG', quality=85, optimize=True)
    elif ext == '.png':
        img.save(output, format='PNG', optimize=True)
    else:
        img.save(output, format='GIF')
    
    async with aiofiles.open(file_path, 'wb') as f:
        await f.write(output.getvalue())
    
    url = f"/media/images/{filename}"
    
    return {
        "url": url,
        "filename": filename,
        "original_filename": file.filename,
        "size": len(output.getvalue()),
        "content_type": file.content_type,
    }
```

---

## 9. Search Functionality

```python
# app/routers/search.py
from fastapi import APIRouter, Depends, Query
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, or_
from sqlalchemy.orm import selectinload
from typing import Optional

from app.database import get_db
from app.models.post import Post, PostStatus
from app.models.user import User
from app.models.tag import Tag

router = APIRouter(prefix="/search", tags=["Search"])

@router.get("/")
async def search(
    q: str = Query(..., min_length=2, max_length=100),
    type: Optional[str] = Query(None, regex="^(posts|users|tags)$"),
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=50),
    db: AsyncSession = Depends(get_db)
):
    """Full-text search across posts, users, and tags"""
    
    results = {}
    
    if not type or type == "posts":
        post_query = select(Post).options(
            selectinload(Post.author)
        ).where(
            Post.status == PostStatus.PUBLISHED,
            or_(
                Post.title.ilike(f"%{q}%"),
                Post.content.ilike(f"%{q}%"),
                Post.excerpt.ilike(f"%{q}%"),
            )
        ).order_by(Post.created_at.desc())
        
        from sqlalchemy import func
        count_result = await db.scalar(
            select(func.count()).select_from(post_query.subquery())
        )
        
        post_result = await db.execute(
            post_query.offset((page - 1) * page_size).limit(page_size)
        )
        posts = post_result.scalars().all()
        
        results["posts"] = {
            "items": [{
                "id": p.id,
                "title": p.title,
                "slug": p.slug,
                "excerpt": p.excerpt[:200],
                "author": p.author.username,
                "created_at": p.created_at.isoformat(),
            } for p in posts],
            "total": count_result,
        }
    
    if not type or type == "users":
        user_result = await db.execute(
            select(User).where(
                User.is_active == True,
                or_(
                    User.username.ilike(f"%{q}%"),
                    User.first_name.ilike(f"%{q}%"),
                    User.last_name.ilike(f"%{q}%"),
                )
            ).limit(10)
        )
        users = user_result.scalars().all()
        
        results["users"] = [{
            "id": u.id,
            "username": u.username,
            "full_name": f"{u.first_name} {u.last_name}".strip(),
        } for u in users]
    
    if not type or type == "tags":
        tag_result = await db.execute(
            select(Tag).where(
                Tag.name.ilike(f"%{q}%")
            ).limit(10)
        )
        tags = tag_result.scalars().all()
        
        results["tags"] = [{
            "id": t.id,
            "name": t.name,
            "slug": t.slug,
        } for t in tags]
    
    return {
        "query": q,
        "results": results
    }
```

---

## 10. Caching with Redis

```python
# app/services/cache.py
import json
import redis.asyncio as aioredis
from typing import Any, Optional
from app.config import settings

class CacheService:
    """Redis cache service"""
    
    def __init__(self):
        self._redis = None
    
    async def get_redis(self):
        if not self._redis:
            self._redis = await aioredis.from_url(
                settings.redis_url,
                encoding="utf-8",
                decode_responses=True,
            )
        return self._redis
    
    async def get(self, key: str) -> Optional[Any]:
        r = await self.get_redis()
        value = await r.get(key)
        if value:
            return json.loads(value)
        return None
    
    async def set(self, key: str, value: Any, expire: int = 300):
        r = await self.get_redis()
        await r.setex(key, expire, json.dumps(value, default=str))
    
    async def delete(self, key: str):
        r = await self.get_redis()
        await r.delete(key)
    
    async def delete_pattern(self, pattern: str):
        r = await self.get_redis()
        keys = await r.keys(pattern)
        if keys:
            await r.delete(*keys)
    
    async def increment(self, key: str, expire: int = 60) -> int:
        r = await self.get_redis()
        pipe = r.pipeline()
        await pipe.incr(key)
        await pipe.expire(key, expire)
        results = await pipe.execute()
        return results[0]
    
    async def get_or_set(self, key: str, func, expire: int = 300):
        """Cache-aside pattern"""
        cached = await self.get(key)
        if cached is not None:
            return cached
        
        value = await func()
        await self.set(key, value, expire)
        return value
    
    async def close(self):
        if self._redis:
            await self._redis.close()

cache_service = CacheService()

# Rate limiter ด้วย Redis
class RateLimiter:
    def __init__(self, requests: int, window: int):
        self.requests = requests
        self.window = window
    
    async def is_allowed(self, identifier: str) -> tuple[bool, int]:
        key = f"rate_limit:{identifier}"
        count = await cache_service.increment(key, expire=self.window)
        
        if count > self.requests:
            return False, 0
        
        remaining = self.requests - count
        return True, remaining
```

---

## 11. Email Notifications

```python
# app/services/email.py
import secrets
from fastapi_mail import FastMail, MessageSchema, ConnectionConfig
from app.config import settings
from app.services.cache import cache_service

mail_config = ConnectionConfig(
    MAIL_USERNAME=settings.mail_username,
    MAIL_PASSWORD=settings.mail_password,
    MAIL_FROM=settings.mail_from,
    MAIL_PORT=settings.mail_port,
    MAIL_SERVER=settings.mail_server,
    MAIL_STARTTLS=True,
    MAIL_SSL_TLS=False,
    USE_CREDENTIALS=True,
)

fastmail = FastMail(mail_config)

async def send_verification_email(email: str, user_id: int, username: str):
    """ส่ง email verification"""
    token = secrets.token_urlsafe(32)
    
    # Store token in Redis (expire 24 hours)
    await cache_service.set(
        f"email_verify:{token}",
        str(user_id),
        expire=86400
    )
    
    verify_url = f"http://localhost:8000/api/v1/auth/verify-email/{token}"
    
    message = MessageSchema(
        subject="Verify Your Email",
        recipients=[email],
        body=f"""
        <html>
        <body>
            <h2>Hello {username}!</h2>
            <p>Please verify your email by clicking the link below:</p>
            <a href="{verify_url}">Verify Email</a>
            <p>This link expires in 24 hours.</p>
        </body>
        </html>
        """,
        subtype="html"
    )
    
    try:
        await fastmail.send_message(message)
    except Exception as e:
        print(f"Email send error: {e}")

async def send_password_reset_email(email: str, user_id: int):
    """ส่ง password reset email"""
    token = secrets.token_urlsafe(32)
    
    await cache_service.set(
        f"password_reset:{token}",
        str(user_id),
        expire=3600  # 1 hour
    )
    
    reset_url = f"http://localhost:3000/reset-password?token={token}"
    
    message = MessageSchema(
        subject="Password Reset Request",
        recipients=[email],
        body=f"""
        <html>
        <body>
            <h2>Password Reset</h2>
            <p>Click the link below to reset your password:</p>
            <a href="{reset_url}">Reset Password</a>
            <p>This link expires in 1 hour. If you didn't request this, ignore this email.</p>
        </body>
        </html>
        """,
        subtype="html"
    )
    
    try:
        await fastmail.send_message(message)
    except Exception as e:
        print(f"Email send error: {e}")

async def send_new_comment_notification(
    post_author_email: str,
    commenter_username: str,
    post_title: str,
    post_slug: str
):
    """แจ้ง author เมื่อมี comment ใหม่"""
    post_url = f"http://localhost:3000/posts/{post_slug}"
    
    message = MessageSchema(
        subject=f"New comment on '{post_title}'",
        recipients=[post_author_email],
        body=f"""
        <html>
        <body>
            <p>{commenter_username} commented on your post '<a href="{post_url}">{post_title}</a>'</p>
        </body>
        </html>
        """,
        subtype="html"
    )
    
    try:
        await fastmail.send_message(message)
    except Exception as e:
        print(f"Email notification error: {e}")
```

---

## 12. Admin Endpoints

```python
# app/routers/admin.py
from fastapi import APIRouter, Depends, Query
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func, update
from datetime import datetime, timedelta

from app.database import get_db
from app.models.user import User
from app.models.post import Post, PostStatus
from app.models.comment import Comment
from app.dependencies import get_current_admin

router = APIRouter(prefix="/admin", tags=["Admin"])

@router.get("/stats")
async def get_stats(
    admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    """Dashboard statistics"""
    now = datetime.utcnow()
    last_30_days = now - timedelta(days=30)
    
    total_users = await db.scalar(select(func.count(User.id)))
    new_users = await db.scalar(
        select(func.count(User.id)).where(User.created_at >= last_30_days)
    )
    total_posts = await db.scalar(select(func.count(Post.id)))
    published_posts = await db.scalar(
        select(func.count(Post.id)).where(Post.status == PostStatus.PUBLISHED)
    )
    total_comments = await db.scalar(select(func.count(Comment.id)))
    
    return {
        "users": {
            "total": total_users,
            "new_last_30_days": new_users,
        },
        "posts": {
            "total": total_posts,
            "published": published_posts,
            "draft": total_posts - published_posts,
        },
        "comments": {
            "total": total_comments,
        }
    }

@router.get("/users")
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    search: str = Query(None),
    admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    """List all users"""
    query = select(User)
    
    if search:
        query = query.where(
            User.username.ilike(f"%{search}%") |
            User.email.ilike(f"%{search}%")
        )
    
    total = await db.scalar(select(func.count()).select_from(query.subquery()))
    
    result = await db.execute(
        query.offset((page - 1) * page_size).limit(page_size)
    )
    users = result.scalars().all()
    
    return {
        "total": total,
        "users": [{
            "id": u.id,
            "email": u.email,
            "username": u.username,
            "is_active": u.is_active,
            "is_verified": u.is_verified,
            "is_admin": u.is_admin,
            "created_at": u.created_at.isoformat(),
        } for u in users]
    }

@router.patch("/users/{user_id}/toggle-active")
async def toggle_user_active(
    user_id: int,
    admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    """Toggle user active status"""
    from sqlalchemy import select as sa_select
    result = await db.execute(sa_select(User).where(User.id == user_id))
    user = result.scalar_one_or_none()
    
    if not user:
        from fastapi import HTTPException
        raise HTTPException(status_code=404, detail="User not found")
    
    user.is_active = not user.is_active
    await db.commit()
    
    return {"user_id": user_id, "is_active": user.is_active}

@router.get("/comments/pending")
async def get_pending_comments(
    admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    """Get unapproved comments"""
    from sqlalchemy.orm import selectinload
    result = await db.execute(
        select(Comment)
        .options(selectinload(Comment.author), selectinload(Comment.post))
        .where(Comment.is_approved == False, Comment.is_deleted == False)
        .order_by(Comment.created_at.desc())
        .limit(50)
    )
    comments = result.scalars().all()
    
    return [{
        "id": c.id,
        "content": c.content,
        "author": c.author.username,
        "post_title": c.post.title,
        "created_at": c.created_at.isoformat(),
    } for c in comments]

@router.post("/comments/{comment_id}/approve")
async def approve_comment(
    comment_id: int,
    admin=Depends(get_current_admin),
    db: AsyncSession = Depends(get_db)
):
    """Approve comment"""
    await db.execute(
        update(Comment)
        .where(Comment.id == comment_id)
        .values(is_approved=True)
    )
    await db.commit()
    return {"message": "Comment approved"}
```

---

## 13. Testing

```python
# tests/conftest.py
import pytest
import pytest_asyncio
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

from app.main import app
from app.database import get_db, Base
from app.utils.security import get_password_hash

TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"

engine_test = create_async_engine(TEST_DATABASE_URL, echo=False)
TestingSessionLocal = async_sessionmaker(
    engine_test, class_=AsyncSession, expire_on_commit=False
)

@pytest_asyncio.fixture(scope="session", autouse=True)
async def setup_database():
    async with engine_test.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with engine_test.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest_asyncio.fixture
async def db_session():
    async with TestingSessionLocal() as session:
        yield session
        await session.rollback()

@pytest_asyncio.fixture
async def client(db_session):
    async def override_get_db():
        yield db_session
    
    app.dependency_overrides[get_db] = override_get_db
    
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as ac:
        yield ac
    
    app.dependency_overrides.clear()

@pytest_asyncio.fixture
async def test_user(db_session):
    from app.models.user import User
    user = User(
        email="test@example.com",
        username="testuser",
        hashed_password=get_password_hash("TestPass123"),
        is_active=True,
        is_verified=True,
    )
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    return user

@pytest_asyncio.fixture
async def auth_token(client, test_user):
    response = await client.post("/api/v1/auth/login", json={
        "email": "test@example.com",
        "password": "TestPass123"
    })
    return response.json()["access_token"]

@pytest_asyncio.fixture
async def auth_headers(auth_token):
    return {"Authorization": f"Bearer {auth_token}"}
```

```python
# tests/test_auth.py
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_register_success(client: AsyncClient):
    response = await client.post("/api/v1/auth/register", json={
        "email": "newuser@example.com",
        "username": "newuser123",
        "password": "NewPass123",
        "first_name": "New",
        "last_name": "User"
    })
    assert response.status_code == 201
    data = response.json()
    assert "message" in data
    assert "user_id" in data

@pytest.mark.asyncio
async def test_register_duplicate_email(client: AsyncClient, test_user):
    response = await client.post("/api/v1/auth/register", json={
        "email": "test@example.com",  # Duplicate
        "username": "anotheruser",
        "password": "AnotherPass123",
    })
    assert response.status_code == 400
    assert "already registered" in response.json()["detail"]

@pytest.mark.asyncio
async def test_login_success(client: AsyncClient, test_user):
    response = await client.post("/api/v1/auth/login", json={
        "email": "test@example.com",
        "password": "TestPass123"
    })
    assert response.status_code == 200
    data = response.json()
    assert "access_token" in data
    assert "refresh_token" in data
    assert "user" in data

@pytest.mark.asyncio
async def test_login_wrong_password(client: AsyncClient, test_user):
    response = await client.post("/api/v1/auth/login", json={
        "email": "test@example.com",
        "password": "WrongPassword"
    })
    assert response.status_code == 401

@pytest.mark.asyncio
async def test_get_protected_route_without_auth(client: AsyncClient):
    response = await client.get("/api/v1/users/me")
    assert response.status_code == 403  # No auth header

@pytest.mark.asyncio
async def test_get_protected_route_with_auth(client: AsyncClient, auth_headers):
    response = await client.get("/api/v1/users/me", headers=auth_headers)
    assert response.status_code == 200

# tests/test_posts.py
@pytest.mark.asyncio
async def test_list_posts_public(client: AsyncClient):
    response = await client.get("/api/v1/posts/")
    assert response.status_code == 200
    data = response.json()
    assert "items" in data
    assert "total" in data

@pytest.mark.asyncio
async def test_create_post(client: AsyncClient, auth_headers):
    response = await client.post(
        "/api/v1/posts/",
        headers=auth_headers,
        json={
            "title": "Test Post Title",
            "content": "This is test content " * 10,
            "excerpt": "Test excerpt",
            "status": "published"
        }
    )
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Test Post Title"
    assert "slug" in data

@pytest.mark.asyncio
async def test_create_post_requires_auth(client: AsyncClient):
    response = await client.post("/api/v1/posts/", json={
        "title": "Test",
        "content": "Content"
    })
    assert response.status_code == 403

@pytest.mark.asyncio
async def test_get_post_by_slug(client: AsyncClient, auth_headers):
    # Create post first
    create_response = await client.post(
        "/api/v1/posts/",
        headers=auth_headers,
        json={
            "title": "Slug Test Post",
            "content": "Content " * 10,
            "status": "published"
        }
    )
    slug = create_response.json()["slug"]
    
    # Get by slug
    response = await client.get(f"/api/v1/posts/{slug}")
    assert response.status_code == 200
    assert response.json()["slug"] == slug
```

---

## 14. Docker Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://bloguser:blogpass@db:5432/blogdb
      - REDIS_URL=redis://redis:6379
      - SECRET_KEY=your-secret-key-change-in-production
      - DEBUG=false
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./uploads:/app/uploads
    restart: unless-stopped
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: bloguser
      POSTGRES_PASSWORD: blogpass
      POSTGRES_DB: blogdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bloguser -d blogdb"]
      interval: 5s
      timeout: 5s
      retries: 5
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3
    command: redis-server --appendonly yes
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./uploads:/uploads
    depends_on:
      - app

volumes:
  postgres_data:
  redis_data:
```

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app
COPY . .

# Create upload directory
RUN mkdir -p uploads/images

# Run migrations and start server
CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000"]
```

### Main App

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from contextlib import asynccontextmanager

from app.config import settings
from app.database import init_db
from app.services.cache import cache_service

from app.routers import (
    auth, users, posts, comments,
    tags, search, upload, admin
)

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await init_db()
    print("Database initialized")
    yield
    # Shutdown
    await cache_service.close()
    print("Cache closed")

app = FastAPI(
    title=settings.app_name,
    description="Full-Stack Blog API",
    version="1.0.0",
    lifespan=lifespan,
    docs_url="/api/docs",
    redoc_url="/api/redoc",
    openapi_url="/api/openapi.json",
)

# Middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Static files
app.mount("/media", StaticFiles(directory="uploads"), name="media")

# Routers
prefix = settings.api_v1_prefix
app.include_router(auth.router, prefix=prefix)
app.include_router(users.router, prefix=prefix)
app.include_router(posts.router, prefix=prefix)
app.include_router(comments.router, prefix=prefix)
app.include_router(tags.router, prefix=prefix)
app.include_router(search.router, prefix=prefix)
app.include_router(upload.router, prefix=prefix)
app.include_router(admin.router, prefix=prefix)

@app.get("/health")
async def health_check():
    return {"status": "healthy", "app": settings.app_name}

@app.get("/")
async def root():
    return {
        "message": "Blog API",
        "docs": "/api/docs",
        "version": "1.0.0"
    }
```

### Alembic Setup

```python
# alembic/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from sqlalchemy.ext.asyncio import AsyncEngine
from alembic import context
import asyncio

from app.database import Base
from app.config import settings

# Import all models
from app.models.user import User
from app.models.post import Post
from app.models.comment import Comment
from app.models.tag import Tag
from app.models.category import Category

config = context.config
config.set_main_option("sqlalchemy.url", settings.database_url)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )
        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### Setup Instructions

```bash
# 1. Clone หรือสร้าง project
mkdir blog_api && cd blog_api

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Setup environment variables
cp .env.example .env
# แก้ไข .env ตามต้องการ

# 5. Run with Docker
docker-compose up --build

# หรือ Run locally:
# Start PostgreSQL และ Redis ก่อน

# 6. Initialize database
alembic init alembic
alembic revision --autogenerate -m "Initial migration"
alembic upgrade head

# 7. Start development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 8. Run tests
pytest tests/ -v

# 9. View API docs
# http://localhost:8000/api/docs
```

### .env.example

```bash
# .env.example
APP_NAME="Blog API"
DEBUG=true

# Database
DATABASE_URL=postgresql://bloguser:blogpass@localhost:5432/blogdb

# JWT
SECRET_KEY=your-very-secret-key-change-this-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=30

# Redis
REDIS_URL=redis://localhost:6379

# Email (Gmail example)
MAIL_USERNAME=your@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587

# Storage
STORAGE_TYPE=local
UPLOAD_DIR=uploads

# Rate Limiting
RATE_LIMIT_REQUESTS=100
RATE_LIMIT_WINDOW=60
```

### Testing Guide

```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_auth.py -v

# Run with coverage
pytest tests/ --cov=app --cov-report=html

# Run only async tests
pytest tests/ -v -m asyncio

# Test with specific marks
pytest tests/ -v -k "test_login"
```

---

## สรุปโปรเจกต์

โปรเจกต์ Full-Stack Blog API นี้ครอบคลุม:

**Authentication:**
- JWT token-based auth
- Email verification
- Password reset flow
- Refresh token rotation

**Content Management:**
- Posts CRUD with slug generation
- Comments with nested replies
- Tags and categories
- Image upload with validation

**Performance:**
- Redis caching (cache-aside pattern)
- Async SQLAlchemy
- Database connection pooling
- Pagination

**Security:**
- Password hashing (bcrypt)
- Token blacklisting
- Rate limiting
- Input validation (Pydantic)

**DevOps:**
- Docker Compose setup
- Database migrations (Alembic)
- Health check endpoint
- Environment configuration

**Testing:**
- Async test client
- Database fixtures
- Auth helpers
- Integration tests

---

*สิ้นสุด Parts 66-70 - Django REST Framework, Authentication, GraphQL, WebSockets, และ FastAPI Project*
