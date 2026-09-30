# Part 95 — Security Best Practices & OWASP

**ระดับ:** Expert  
**หัวข้อ:** การรักษาความปลอดภัยสำหรับ Python Web Applications ตามแนวทาง OWASP  
**ภาษา:** Thai (คำอธิบาย) + English (code/keywords)

---

## สารบัญ

1. [OWASP Top 10 Overview](#1-owasp-top-10-overview)
2. [SQL Injection Prevention](#2-sql-injection-prevention)
3. [XSS Prevention](#3-xss-prevention)
4. [Authentication Security](#4-authentication-security)
5. [HTTPS/TLS Configuration](#5-httpstls-configuration)
6. [Input Validation & Sanitization](#6-input-validation--sanitization)
7. [Secret Management](#7-secret-management)
8. [CORS Configuration](#8-cors-configuration)
9. [Rate Limiting](#9-rate-limiting)
10. [Dependency Scanning](#10-dependency-scanning)
11. [แบบฝึกหัด (Exercises)](#11-แบบฝึกหัด)

---

## 1. OWASP Top 10 Overview

**OWASP (Open Web Application Security Project)** คือองค์กรไม่แสวงหาผลกำไรที่เผยแพร่รายการช่องโหว่ความปลอดภัยที่พบบ่อยที่สุดในเว็บแอปพลิเคชัน รู้จักกันในชื่อ **OWASP Top 10** ซึ่งอัปเดตล่าสุดในปี 2021

### OWASP Top 10 (2021) สำหรับ Python Web Apps

| ลำดับ | หมวดหมู่ | ความเสี่ยง |
|-------|----------|------------|
| A01 | Broken Access Control | ผู้ใช้เข้าถึงข้อมูล/ฟังก์ชันที่ไม่ได้รับอนุญาต |
| A02 | Cryptographic Failures | การเข้ารหัสที่อ่อนแอหรือไม่มีการเข้ารหัส |
| A03 | Injection | SQL, NoSQL, OS Command Injection |
| A04 | Insecure Design | ออกแบบระบบโดยไม่คำนึงถึงความปลอดภัย |
| A05 | Security Misconfiguration | ตั้งค่าระบบผิดพลาด เช่น debug mode เปิดอยู่ |
| A06 | Vulnerable Components | ใช้ library/framework เวอร์ชันที่มีช่องโหว่ |
| A07 | Auth & Session Failures | ระบบ authentication/session ที่ไม่ปลอดภัย |
| A08 | Software Integrity Failures | ไม่ตรวจสอบความสมบูรณ์ของ software |
| A09 | Logging & Monitoring Failures | ไม่มีระบบ log และ monitor ที่เพียงพอ |
| A10 | SSRF | Server-Side Request Forgery |

```python
# ตัวอย่างที่ 1: Security Headers Middleware สำหรับ Flask
# ครอบคลุม OWASP หลายข้อในการตั้งค่า HTTP Headers

from flask import Flask, Response
from functools import wraps

app = Flask(__name__)

def add_security_headers(f):
    """Decorator ที่เพิ่ม Security Headers ทุก response"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        response = f(*args, **kwargs)
        if isinstance(response, str):
            response = Response(response)
        
        # ป้องกัน XSS และ injection ผ่าน browser
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'DENY'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        
        # บังคับ HTTPS (HSTS)
        response.headers['Strict-Transport-Security'] = (
            'max-age=31536000; includeSubDomains; preload'
        )
        
        # Content Security Policy
        response.headers['Content-Security-Policy'] = (
            "default-src 'self'; "
            "script-src 'self' 'nonce-{nonce}'; "
            "style-src 'self' 'unsafe-inline'; "
            "img-src 'self' data: https:; "
            "frame-ancestors 'none';"
        )
        
        # ซ่อน server information
        response.headers['Server'] = 'WebServer'
        response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
        
        return response
    return decorated_function


@app.route('/')
@add_security_headers
def index():
    return "Hello, Secure World!"
```

---

## 2. SQL Injection Prevention

**SQL Injection** คือการโจมตีที่ผู้ไม่ประสงค์ดีแทรก SQL code เข้าไปใน input เพื่อเข้าถึงหรือแก้ไขข้อมูลในฐานข้อมูล เป็นหนึ่งในช่องโหว่ที่อันตรายและพบบ่อยที่สุด

### 2.1 ตัวอย่างที่เสี่ยง (Vulnerable) vs ปลอดภัย (Safe)

```python
# ตัวอย่างที่ 2: SQL Injection - ช่องโหว่และการป้องกัน

import sqlite3
import sqlalchemy
from sqlalchemy import create_engine, text
from sqlalchemy.orm import Session

# ========== VULNERABLE - อย่าทำแบบนี้! ==========

def get_user_vulnerable(username: str) -> dict:
    """
    ช่องโหว่: นำ input มาต่อ string โดยตรง
    ผู้โจมตีสามารถใส่: username = "admin' OR '1'='1"
    ทำให้ได้ข้อมูลทุก user
    """
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    
    # อันตราย! String concatenation โดยตรง
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)  # SQL Injection ได้ที่นี่
    return cursor.fetchone()


# ========== SAFE - Parameterized Queries ==========

def get_user_safe_sqlite(username: str) -> dict:
    """
    ปลอดภัย: ใช้ parameterized query
    SQLite จะ escape ค่าทั้งหมดให้อัตโนมัติ
    """
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    
    # ปลอดภัย: ใช้ ? เป็น placeholder
    query = "SELECT * FROM users WHERE username = ?"
    cursor.execute(query, (username,))
    return cursor.fetchone()


def get_user_safe_sqlalchemy(username: str, session: Session) -> dict:
    """
    ปลอดภัย: ใช้ SQLAlchemy parameterized query
    """
    # ปลอดภัย: ใช้ :username เป็น named parameter
    result = session.execute(
        text("SELECT * FROM users WHERE username = :username"),
        {"username": username}
    )
    return result.fetchone()
```

### 2.2 SQLAlchemy ORM Pattern (แนะนำ)

```python
# ตัวอย่างที่ 3: SQLAlchemy ORM Safe Patterns

from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.orm import DeclarativeBase, Session, sessionmaker
from sqlalchemy import select, and_, or_

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    role = Column(String(20), default='user')

# สร้าง engine และ session factory
engine = create_engine(
    "postgresql+psycopg2://user:password@localhost/dbname",
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20,
)
SessionLocal = sessionmaker(bind=engine)


def get_user_by_username(username: str) -> User | None:
    """
    ปลอดภัย 100%: ORM จัดการ parameterization ให้ทั้งหมด
    ไม่มีทางเกิด SQL Injection จาก ORM query
    """
    with SessionLocal() as session:
        stmt = select(User).where(User.username == username)
        return session.execute(stmt).scalar_one_or_none()


def search_users_safe(search_term: str, role: str) -> list[User]:
    """
    ปลอดภัย: ค้นหาด้วยหลายเงื่อนไข
    ORM จัดการ escaping ให้ทุก parameter
    """
    with SessionLocal() as session:
        stmt = (
            select(User)
            .where(
                and_(
                    User.role == role,
                    or_(
                        User.username.ilike(f"%{search_term}%"),
                        User.email.ilike(f"%{search_term}%"),
                    )
                )
            )
            .limit(100)  # จำกัดผลลัพธ์เสมอ
        )
        return session.execute(stmt).scalars().all()
```

---

## 3. XSS Prevention

**Cross-Site Scripting (XSS)** คือการโจมตีที่แทรก JavaScript code ที่เป็นอันตรายลงในหน้าเว็บ ทำให้ browser ของผู้ใช้รันโค้ดนั้น

### 3.1 Output Escaping

```python
# ตัวอย่างที่ 4: XSS Prevention ด้วย Output Escaping

import html
import markupsafe
from markupsafe import Markup, escape

# ========== VULNERABLE ==========
def render_comment_vulnerable(user_comment: str) -> str:
    # อันตราย: แสดงผล user input โดยตรง
    return f"<div class='comment'>{user_comment}</div>"

# ผู้โจมตีส่ง: <script>document.cookie</script>
# จะถูก render เป็น JavaScript!


# ========== SAFE: ใช้ html.escape ==========
def render_comment_safe(user_comment: str) -> str:
    """
    ปลอดภัย: escape HTML characters ทั้งหมด
    < → &lt;  > → &gt;  " → &quot;  & → &amp;  ' → &#x27;
    """
    safe_comment = html.escape(user_comment, quote=True)
    return f"<div class='comment'>{safe_comment}</div>"


# ========== SAFE: ใช้ MarkupSafe (Jinja2) ==========
def render_with_markupsafe(user_comment: str) -> Markup:
    """
    MarkupSafe ใช้ใน Jinja2 templates โดยอัตโนมัติ
    Markup() บอกว่า string นี้ปลอดภัยแล้ว
    escape() จะ escape string ที่ไม่ปลอดภัย
    """
    safe_comment = escape(user_comment)  # escape user input
    # Markup() สำหรับ HTML ที่เราสร้างเองและปลอดภัยแล้ว
    return Markup(f"<div class='comment'>{safe_comment}</div>")


# ========== SAFE: ใช้ bleach สำหรับ Rich Text ==========
import bleach

ALLOWED_TAGS = ['b', 'i', 'em', 'strong', 'a', 'p', 'br']
ALLOWED_ATTRIBUTES = {'a': ['href', 'title']}
ALLOWED_PROTOCOLS = ['http', 'https', 'mailto']

def sanitize_rich_text(user_html: str) -> str:
    """
    ปลอดภัย: อนุญาต HTML บางส่วนแต่ลบ tag อันตราย
    เหมาะสำหรับ comment system ที่ต้องการ formatting
    """
    return bleach.clean(
        user_html,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
        protocols=ALLOWED_PROTOCOLS,
        strip=True,         # ลบ tag ที่ไม่อนุญาตออก
        strip_comments=True # ลบ HTML comments ออก
    )


# ทดสอบ
malicious_input = '<script>alert("XSS")</script><b>Bold text</b>'
print(sanitize_rich_text(malicious_input))
# Output: <b>Bold text</b>
```

### 3.2 Content Security Policy (CSP)

```python
# ตัวอย่างที่ 5: Content Security Policy Headers ใน FastAPI

from fastapi import FastAPI, Request, Response
from fastapi.middleware.base import BaseHTTPMiddleware
import secrets

app = FastAPI()


class CSPMiddleware(BaseHTTPMiddleware):
    """
    Middleware ที่เพิ่ม Content Security Policy headers
    ป้องกัน XSS โดยบอก browser ว่าโหลด resource จากที่ไหนได้บ้าง
    """
    
    async def dispatch(self, request: Request, call_next):
        # สร้าง nonce สำหรับ inline scripts (ถ้าจำเป็น)
        nonce = secrets.token_urlsafe(16)
        request.state.csp_nonce = nonce
        
        response = await call_next(request)
        
        # กำหนด CSP policy
        csp_policy = "; ".join([
            "default-src 'self'",
            f"script-src 'self' 'nonce-{nonce}'",
            "style-src 'self' 'unsafe-inline'",
            "img-src 'self' data: https:",
            "font-src 'self' https://fonts.gstatic.com",
            "connect-src 'self'",
            "frame-ancestors 'none'",
            "base-uri 'self'",
            "form-action 'self'",
            "upgrade-insecure-requests",
        ])
        
        response.headers["Content-Security-Policy"] = csp_policy
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        
        return response


app.add_middleware(CSPMiddleware)
```

---

## 4. Authentication Security

การรักษาความปลอดภัยของระบบ authentication เป็นสิ่งสำคัญที่สุดอย่างหนึ่ง ต้องเก็บรหัสผ่านอย่างปลอดภัยและจัดการ session/token อย่างถูกต้อง

### 4.1 Password Hashing ด้วย bcrypt และ argon2

```python
# ตัวอย่างที่ 6: Secure Password Hashing

import bcrypt
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError, InvalidHashError

# ========== bcrypt ==========

def hash_password_bcrypt(password: str) -> bytes:
    """
    Hash รหัสผ่านด้วย bcrypt
    - cost factor (work_factor=12) ควบคุมความช้าในการ hash
    - bcrypt จัดการ salt ให้อัตโนมัติ
    - ยิ่ง cost สูง ยิ่งปลอดภัยแต่ช้ากว่า
    """
    password_bytes = password.encode('utf-8')
    salt = bcrypt.gensalt(rounds=12)  # work factor 12 (แนะนำ minimum)
    hashed = bcrypt.hashpw(password_bytes, salt)
    return hashed


def verify_password_bcrypt(password: str, hashed: bytes) -> bool:
    """bcrypt.checkpw() ใช้ constant-time comparison ป้องกัน timing attack"""
    try:
        return bcrypt.checkpw(password.encode('utf-8'), hashed)
    except Exception:
        return False


# ========== Argon2id (แนะนำ - winner Password Hashing Competition 2015) ==========
# ปลอดภัยกว่า bcrypt ต่อ GPU/ASIC attacks เพราะใช้ memory สูง

ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=2)


def hash_password_argon2(password: str) -> str:
    return ph.hash(password)


def verify_password_argon2(password: str, hashed: str) -> bool:
    try:
        return ph.verify(hashed, password)
    except (VerifyMismatchError, InvalidHashError, Exception):
        return False


def check_and_rehash(password: str, hashed: str, user_id: int, db) -> bool:
    """ตรวจสอบและ rehash อัตโนมัติเมื่อ parameters ล้าสมัย"""
    if not verify_password_argon2(password, hashed):
        return False
    if ph.check_needs_rehash(hashed):
        db.update_password_hash(user_id, ph.hash(password))
    return True
```

### 4.2 JWT Best Practices

```python
# ตัวอย่างที่ 7: JWT Security Best Practices

import jwt
import secrets
from datetime import datetime, timedelta, timezone
from typing import Optional
from dataclasses import dataclass

# ห้ามใช้ 'none' algorithm!
# ห้ามใช้ HS256 สำหรับ public API - ควรใช้ RS256 หรือ ES256

@dataclass
class TokenConfig:
    secret_key: str        # ต้องยาวอย่างน้อย 256 bits (32 bytes)
    algorithm: str = "HS256"
    access_token_expire: int = 15    # นาที (สั้น!)
    refresh_token_expire: int = 7    # วัน


def create_access_token(
    user_id: int, email: str, roles: list[str], config: TokenConfig,
) -> str:
    """สร้าง JWT token: อายุสั้น, มี jti ป้องกัน replay, ระบุ iss/aud"""
    now = datetime.now(timezone.utc)
    payload = {
        "sub": str(user_id), "email": email, "roles": roles,
        "iat": now,
        "exp": now + timedelta(minutes=config.access_token_expire),
        "nbf": now,
        "iss": "myapp.com", "aud": "myapp-api",
        "jti": secrets.token_urlsafe(16),  # ป้องกัน replay attack
    }
    return jwt.encode(payload, config.secret_key, algorithm=config.algorithm)


def verify_access_token(
    token: str, config: TokenConfig, revoked_jtis: set[str],
) -> Optional[dict]:
    """ตรวจสอบ JWT: signature, expiry, iss/aud, และ revocation list"""
    try:
        payload = jwt.decode(
            token, config.secret_key,
            algorithms=[config.algorithm],  # ระบุ algorithms เสมอ ห้าม any!
            audience="myapp-api", issuer="myapp.com",
            options={"require": ["exp", "iat", "sub", "jti"]},
        )
        if payload.get("jti") in revoked_jtis:
            return None  # Token ถูก revoke แล้ว
        return payload
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None
```

---

## 5. HTTPS/TLS Configuration

**TLS (Transport Layer Security)** เป็นการเข้ารหัสข้อมูลระหว่าง client และ server ป้องกัน man-in-the-middle attacks

### 5.1 ssl module

```python
# ตัวอย่างที่ 8: HTTPS/TLS Configuration ใน Python

import ssl
import socket
import requests
import httpx
from pathlib import Path

# ========== สร้าง SSL Context ที่ปลอดภัย ==========

def create_secure_ssl_context(
    certfile: str,
    keyfile: str,
    ca_certfile: Optional[str] = None,
) -> ssl.SSLContext:
    """สร้าง SSL Context บังคับ TLS 1.2+ พร้อม forward-secrecy ciphers"""
    context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
    context.load_cert_chain(certfile=certfile, keyfile=keyfile)
    if ca_certfile:
        context.load_verify_locations(cafile=ca_certfile)
        context.verify_mode = ssl.CERT_REQUIRED  # mutual TLS
    context.minimum_version = ssl.TLSVersion.TLSv1_2
    context.set_ciphers(
        "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:"
        "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305"
    )
    context.options |= ssl.OP_CIPHER_SERVER_PREFERENCE | ssl.OP_NO_RENEGOTIATION
    return context


def make_secure_request(url: str, ca_bundle: Optional[str] = None) -> dict:
    """ทำ HTTPS request - ห้ามใช้ verify=False เด็ดขาด!"""
    session = requests.Session()
    try:
        response = session.get(
            url,
            verify=ca_bundle or True,  # ห้ามเป็น False!
            timeout=(5, 30),
        )
        response.raise_for_status()
        return response.json()
    except requests.exceptions.SSLError as e:
        raise ValueError(f"SSL verification failed: {e}")
    except requests.exceptions.Timeout:
        raise TimeoutError("Request timed out")


# ========== ตรวจสอบ Certificate ของ server ==========

def check_certificate_expiry(hostname: str, port: int = 443) -> dict:
    """ตรวจสอบวันหมดอายุของ SSL certificate สำหรับ monitoring"""
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port), timeout=10) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            cert = ssock.getpeercert()
    return {
        "subject": dict(x[0] for x in cert.get("subject", [])),
        "not_after": cert.get("notAfter"),
        "san": cert.get("subjectAltName", []),
    }
```

---

## 6. Input Validation & Sanitization

การตรวจสอบ input ทุกอย่างที่มาจากผู้ใช้เป็นหลักการพื้นฐาน: **"Never trust user input"**

### 6.1 Pydantic Validators

```python
# ตัวอย่างที่ 9: Input Validation ด้วย Pydantic v2

from pydantic import (
    BaseModel, EmailStr, field_validator, model_validator,
    Field, SecretStr, HttpUrl
)
from pydantic import validator
import re
from typing import Annotated

# Custom types ด้วย Annotated
PositiveInt = Annotated[int, Field(gt=0)]
SafeString = Annotated[str, Field(min_length=1, max_length=500)]


class UserRegistration(BaseModel):
    """Schema สำหรับการลงทะเบียน user พร้อม validation เข้มงวด"""
    
    username: Annotated[str, Field(min_length=3, max_length=30)]
    email: EmailStr
    password: SecretStr = Field(min_length=8)
    age: Annotated[int, Field(ge=13, le=120)]
    website: Optional[HttpUrl] = None
    
    @field_validator("username")
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        """ตรวจสอบว่า username มีเฉพาะตัวอักษร ตัวเลข และ underscore"""
        if not re.match(r'^[a-zA-Z0-9_]+$', v):
            raise ValueError(
                "Username must contain only letters, numbers, and underscores"
            )
        # ป้องกัน reserved words
        reserved = {'admin', 'root', 'system', 'null', 'undefined'}
        if v.lower() in reserved:
            raise ValueError("This username is reserved")
        return v.lower()  # normalize เป็น lowercase
    
    @field_validator("password")
    @classmethod
    def password_strength(cls, v: SecretStr) -> SecretStr:
        """ตรวจสอบความแข็งแรงของรหัสผ่าน"""
        password = v.get_secret_value()
        
        checks = {
            "uppercase": bool(re.search(r'[A-Z]', password)),
            "lowercase": bool(re.search(r'[a-z]', password)),
            "digit": bool(re.search(r'\d', password)),
            "special": bool(re.search(r'[!@#$%^&*(),.?":{}|<>]', password)),
        }
        
        failed = [name for name, passed in checks.items() if not passed]
        if failed:
            raise ValueError(
                f"Password must contain: {', '.join(failed)}"
            )
        return v
    
    @model_validator(mode='after')
    def check_age_and_content(self) -> 'UserRegistration':
        """ตรวจสอบเงื่อนไขที่ต้องใช้หลายฟิลด์พร้อมกัน"""
        # ตัวอย่าง: ผู้ใช้อายุต่ำกว่า 18 มีข้อจำกัด
        if self.age < 18 and self.website:
            raise ValueError("Users under 18 cannot set a website")
        return self


class SearchQuery(BaseModel):
    """Schema สำหรับ search query ป้องกัน injection"""
    
    query: Annotated[str, Field(min_length=1, max_length=200)]
    page: Annotated[int, Field(ge=1, le=1000)] = 1
    per_page: Annotated[int, Field(ge=1, le=100)] = 20
    sort_by: str = Field(default="created_at")
    
    @field_validator("query")
    @classmethod
    def sanitize_query(cls, v: str) -> str:
        """ลบ characters ที่อาจใช้สำหรับ injection"""
        # ลบ SQL special characters
        dangerous_chars = [';', '--', '/*', '*/', 'xp_', 'EXEC', 'EXECUTE']
        v_upper = v.upper()
        for char in dangerous_chars:
            if char.upper() in v_upper:
                raise ValueError(f"Invalid characters in search query")
        return v.strip()
    
    @field_validator("sort_by")
    @classmethod
    def validate_sort_field(cls, v: str) -> str:
        """Whitelist approach: อนุญาตเฉพาะ fields ที่กำหนดไว้"""
        allowed_fields = {"created_at", "updated_at", "username", "email"}
        if v not in allowed_fields:
            raise ValueError(f"sort_by must be one of: {allowed_fields}")
        return v
```

### 6.2 bleach สำหรับ HTML Sanitization

```python
# ตัวอย่างที่ 10: Advanced HTML Sanitization ด้วย bleach

import bleach
from bleach.linkifier import LinkifyFilter

# Configuration สำหรับ blog comment system
BLOG_ALLOWED_TAGS = [
    'p', 'br', 'b', 'i', 'em', 'strong', 'u', 's',
    'h1', 'h2', 'h3', 'h4',
    'ul', 'ol', 'li',
    'a', 'blockquote', 'code', 'pre',
]

BLOG_ALLOWED_ATTRS = {
    'a': ['href', 'title', 'rel'],
    'img': ['src', 'alt', 'width', 'height'],
    '*': ['class'],
}

def sanitize_blog_comment(html_content: str) -> str:
    """
    Sanitize HTML สำหรับ blog comment
    อนุญาต formatting พื้นฐานแต่ลบ script และ event handlers
    """
    # ทำความสะอาด HTML
    clean = bleach.clean(
        html_content,
        tags=BLOG_ALLOWED_TAGS,
        attributes=BLOG_ALLOWED_ATTRS,
        strip=True,
        strip_comments=True,
    )
    
    # Auto-linkify URLs ที่เป็น plain text
    clean = bleach.linkify(
        clean,
        callbacks=[bleach.callbacks.nofollow],  # เพิ่ม rel="nofollow"
        skip_tags=['code', 'pre'],              # ไม่ linkify ใน code block
    )
    
    return clean


# ทดสอบ
malicious = """
<p>Hello!</p>
<script>alert('XSS')</script>
<img src="x" onerror="alert('XSS')">
<a href="javascript:alert('XSS')">Click me</a>
<p onclick="evil()">Paragraph</p>
"""

safe_output = sanitize_blog_comment(malicious)
print(safe_output)
# Output: <p>Hello!</p> &lt;img src="x"&gt; Click me <p>Paragraph</p>
```

---

## 7. Secret Management

**การจัดการ secrets อย่างปลอดภัย** เป็นสิ่งสำคัญ ห้ามเก็บ API keys, passwords หรือ credentials ใน source code เด็ดขาด

### 7.1 python-dotenv

```python
# ตัวอย่างที่ 11: Secret Management ด้วย python-dotenv

# ไฟล์ .env (อย่า commit ไฟล์นี้ลง git!)
# DATABASE_URL=postgresql://user:secret@localhost/mydb
# SECRET_KEY=your-super-secret-key-here
# AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
# REDIS_URL=redis://localhost:6379/0

import os
from pathlib import Path
from dotenv import load_dotenv
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict

# โหลด .env file (environment variables จะ override ค่าใน .env)
env_path = Path(__file__).parent / '.env'
load_dotenv(env_path, override=False)  # override=False: env vars จริงสำคัญกว่า


# ========== Pydantic Settings (แนะนำ) ==========

class AppSettings(BaseSettings):
    """
    Settings ที่อ่านจาก environment variables อัตโนมัติ
    ตรวจสอบ types และ required fields ให้ด้วย
    """
    model_config = SettingsConfigDict(
        env_file='.env',
        env_file_encoding='utf-8',
        case_sensitive=False,
        extra='ignore',  # ไม่ error ถ้ามี env var ที่ไม่รู้จัก
    )
    
    # Database
    database_url: str
    database_pool_size: int = 10
    
    # Security
    secret_key: str
    jwt_algorithm: str = "HS256"
    access_token_expire_minutes: int = 15
    
    # Redis
    redis_url: str = "redis://localhost:6379/0"
    
    # AWS (Optional)
    aws_region: str = "ap-southeast-1"
    aws_access_key_id: Optional[str] = None
    aws_secret_access_key: Optional[str] = None
    
    # App
    debug: bool = False
    allowed_hosts: list[str] = ["localhost", "127.0.0.1"]
    
    def validate_secret_key_strength(self) -> None:
        """ตรวจสอบว่า secret key แข็งแรงพอ"""
        if len(self.secret_key) < 32:
            raise ValueError("SECRET_KEY must be at least 32 characters")
        if self.secret_key in ("secret", "password", "changeme"):
            raise ValueError("SECRET_KEY is too weak - use a random value")


@lru_cache(maxsize=1)
def get_settings() -> AppSettings:
    """
    lru_cache ทำให้อ่าน settings ครั้งเดียวตลอด application lifecycle
    """
    settings = AppSettings()
    settings.validate_secret_key_strength()
    return settings
```

### 7.2 AWS Secrets Manager Pattern

```python
# ตัวอย่างที่ 12: AWS Secrets Manager Integration

import boto3
import json
from botocore.exceptions import ClientError
from functools import lru_cache
import logging

logger = logging.getLogger(__name__)


class SecretsManager:
    """Wrapper สำหรับ AWS Secrets Manager พร้อม in-memory cache"""
    
    def __init__(self, region_name: str = "ap-southeast-1"):
        self.client = boto3.client("secretsmanager", region_name=region_name)
        self._cache: dict = {}
    
    def get_secret(self, secret_name: str) -> dict:
        """ดึง secret พร้อม cache เพื่อลด API calls และ latency"""
        if secret_name in self._cache:
            return self._cache[secret_name]
        try:
            response = self.client.get_secret_value(SecretId=secret_name)
            secret = json.loads(response["SecretString"])
            self._cache[secret_name] = secret
            return secret
        except ClientError as e:
            code = e.response["Error"]["Code"]
            if code == "ResourceNotFoundException":
                raise ValueError(f"Secret '{secret_name}' not found")
            raise
    
    def get_database_credentials(self) -> dict:
        """ดึง database credentials จาก Secrets Manager"""
        return self.get_secret("myapp/production/database")


def create_db_engine_from_secrets():
    """สร้าง database engine โดยใช้ secrets จาก AWS"""
    sm = SecretsManager()
    creds = sm.get_database_credentials()
    db_url = (
        f"postgresql+psycopg2://{creds['username']}:{creds['password']}"
        f"@{creds['host']}:{creds['port']}/{creds['dbname']}"
    )
    return create_engine(db_url)
```

---

## 8. CORS Configuration

**CORS (Cross-Origin Resource Sharing)** ต้องตั้งค่าให้ถูกต้อง การตั้งค่า `allow_origins=["*"]` นั้นอันตรายสำหรับ API ที่มีการ authentication

### 8.1 FastAPI CORS

```python
# ตัวอย่างที่ 13: CORS Configuration ใน FastAPI

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# ========== VULNERABLE: ห้ามใช้แบบนี้สำหรับ authenticated API! ==========
# app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_credentials=True)
# ERROR: ห้ามใช้ allow_origins=["*"] พร้อม allow_credentials=True

# ========== SAFE: กำหนด origins ที่อนุญาตอย่างชัดเจน ==========

ALLOWED_ORIGINS = [
    "https://myapp.com",
    "https://www.myapp.com",
    "https://admin.myapp.com",
]

# เพิ่ม localhost สำหรับ development เท่านั้น
if os.getenv("ENVIRONMENT") == "development":
    ALLOWED_ORIGINS.extend([
        "http://localhost:3000",
        "http://localhost:8080",
        "http://127.0.0.1:3000",
    ])

app.add_middleware(
    CORSMiddleware,
    allow_origins=ALLOWED_ORIGINS,      # กำหนด origins ชัดเจน
    allow_credentials=True,             # อนุญาต cookies/auth headers
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=[
        "Authorization",
        "Content-Type",
        "X-Request-ID",
        "X-CSRF-Token",
    ],
    expose_headers=["X-Request-ID"],    # Headers ที่ browser อ่านได้
    max_age=3600,                       # Cache preflight 1 ชั่วโมง
)
```

### 8.2 Flask CORS

```python
# ตัวอย่างที่ 14: CORS Configuration ใน Flask

from flask import Flask
from flask_cors import CORS

app = Flask(__name__)

# ========== SAFE: Per-route CORS configuration ==========

# กำหนด CORS สำหรับ public API endpoints
CORS(
    app,
    resources={
        r"/api/public/*": {
            "origins": "*",              # Public endpoints อนุญาตทุก origin
            "methods": ["GET"],          # อนุญาตเฉพาะ GET
            "allow_headers": ["Content-Type"],
        },
        r"/api/v1/*": {
            "origins": ALLOWED_ORIGINS,  # Private endpoints จำกัด origins
            "methods": ["GET", "POST", "PUT", "DELETE"],
            "allow_headers": ["Authorization", "Content-Type"],
            "supports_credentials": True,
            "max_age": 3600,
        },
    }
)


# ========== Custom CORS Middleware สำหรับ advanced use case ==========

from flask import request, make_response

@app.before_request
def handle_preflight():
    """จัดการ CORS preflight requests"""
    if request.method == "OPTIONS":
        origin = request.headers.get("Origin", "")
        
        if origin not in ALLOWED_ORIGINS:
            return make_response("", 403)
        
        response = make_response("", 204)
        response.headers["Access-Control-Allow-Origin"] = origin
        response.headers["Access-Control-Allow-Methods"] = (
            "GET, POST, PUT, DELETE, OPTIONS"
        )
        response.headers["Access-Control-Allow-Headers"] = (
            "Authorization, Content-Type"
        )
        response.headers["Access-Control-Max-Age"] = "3600"
        response.headers["Vary"] = "Origin"
        return response
```

---

## 9. Rate Limiting

**Rate Limiting** ป้องกัน brute force attacks, DDoS และการใช้ API อย่างไม่เหมาะสม

### 9.1 SlowAPI (FastAPI)

```python
# ตัวอย่างที่ 15: Rate Limiting ด้วย SlowAPI สำหรับ FastAPI

from fastapi import FastAPI, Request, Depends
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from slowapi.middleware import SlowAPIMiddleware

# สร้าง limiter โดยใช้ IP address เป็น key
limiter = Limiter(
    key_func=get_remote_address,
    default_limits=["1000/hour"],  # default limit สำหรับทุก endpoint
    storage_uri="redis://localhost:6379",  # ใช้ Redis สำหรับ distributed apps
)

app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)
app.add_middleware(SlowAPIMiddleware)


def get_user_id(request: Request) -> str:
    """ใช้ user ID เป็น rate limit key สำหรับ authenticated users"""
    user = getattr(request.state, "user", None)
    if user:
        return f"user:{user.id}"
    return get_remote_address(request)  # fallback เป็น IP


@app.post("/api/auth/login")
@limiter.limit("5/minute")           # เข้มงวดสำหรับ login
async def login(request: Request, credentials: dict):
    pass  # ตรวจสอบ credentials ด้วย argon2


@app.post("/api/auth/forgot-password")
@limiter.limit("3/hour")             # เข้มงวดมากสำหรับ password reset
async def forgot_password(request: Request, email: str):
    pass  # ส่ง reset email


@app.get("/api/data")
@limiter.limit("100/minute", key_func=get_user_id)
async def get_data(request: Request):
    return {"data": "..."}
```

### 9.2 Flask-Limiter

```python
# ตัวอย่างที่ 16: Rate Limiting ด้วย Flask-Limiter

from flask import Flask, jsonify, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200/day", "50/hour"],
    storage_uri="redis://localhost:6379",
    strategy="fixed-window-elastic-expiry",  # ป้องกัน burst attacks
    on_breach=lambda limit: (
        jsonify({
            "error": "rate_limit_exceeded",
            "message": f"Too many requests. Retry after {limit.reset_at}",
            "retry_after": limit.reset_at.isoformat(),
        }),
        429
    )
)


@app.route("/api/login", methods=["POST"])
@limiter.limit("5 per minute; 20 per hour")
def login():
    pass  # 5 ครั้ง/นาที และ 20 ครั้ง/ชั่วโมง ป้องกัน brute force


@app.route("/api/register", methods=["POST"])
@limiter.limit("3 per hour", key_func=lambda: request.json.get("email", ""))
def register():
    pass  # rate limit ต่อ email ป้องกัน spam registration


@app.route("/api/search")
@limiter.limit("30/minute")
def search():
    pass


# Progressive backoff: เพิ่มความเข้มงวดเมื่อ fail ซ้ำๆ
import redis as redis_lib

redis_client = redis_lib.Redis(host='localhost', port=6379, decode_responses=True)

def record_auth_failure(identifier: str) -> None:
    """บันทึก failure พร้อม progressive TTL: 1m → 15m → 1h"""
    key = f"auth_failures:{identifier}"
    failures = redis_client.incr(key)
    ttl_map = {5: 60, 10: 900}
    ttl = next((v for k, v in ttl_map.items() if failures <= k), 3600)
    redis_client.expire(key, ttl)
```

---

## 10. Dependency Scanning

การตรวจสอบ dependencies เป็นประจำช่วยค้นหาช่องโหว่ที่รู้จักใน libraries ที่ใช้

### 10.1 Safety และ Bandit

```python
# ตัวอย่างที่ 17: Security Scanning ใน CI/CD Pipeline

# ============================================================
# การใช้ safety - ตรวจสอบ dependencies
# ============================================================
# pip install safety
# 
# ตรวจสอบ dependencies ปัจจุบัน:
# safety check
#
# ตรวจสอบจาก requirements.txt:
# safety check -r requirements.txt
#
# ตรวจสอบและ export รายงาน:
# safety check --json > safety-report.json
#
# ในรูปแบบ CI-friendly:
# safety check --exit-code  # exit 1 ถ้าพบช่องโหว่

# ============================================================
# การใช้ Bandit - Static Analysis Security Testing (SAST)
# ============================================================
# pip install bandit
#
# สแกน directory:
# bandit -r ./src
#
# สแกนและกำหนด severity level:
# bandit -r ./src -l  # เฉพาะ LOW และสูงกว่า
# bandit -r ./src -ll # เฉพาะ MEDIUM และสูงกว่า
#
# export รายงาน:
# bandit -r ./src -f json -o bandit-report.json
#
# ข้ามไฟล์ test:
# bandit -r ./src --exclude ./tests

# ============================================================
# pyproject.toml configuration สำหรับ Bandit
# ============================================================
# [tool.bandit]
# exclude_dirs = ["tests", "venv"]
# skips = ["B101"]  # ข้าม assert_used ใน test files
# tests = ["B201", "B301"]

# ============================================================
# ตัวอย่าง Makefile สำหรับ security checks
# ============================================================
# security-check:
#     @echo "Running safety check..."
#     safety check -r requirements.txt
#     @echo "Running bandit..."
#     bandit -r src/ -f json -o reports/bandit.json
#     @echo "Security checks complete!"


# ============================================================
# ตัวอย่างสิ่งที่ Bandit ตรวจจับ
# ============================================================

import subprocess
import hashlib
import random
import pickle

# B602: subprocess_popen_with_shell_equals_true
def vulnerable_command(user_input: str):
    # Bandit จะ flag นี้: B602 High severity
    subprocess.call(f"ls {user_input}", shell=True)  # DANGEROUS!

def safe_command(directory: str):
    # ปลอดภัย: ไม่ใช้ shell=True และ validate input
    allowed_dirs = {"/tmp", "/var/log"}
    if directory not in allowed_dirs:
        raise ValueError("Invalid directory")
    subprocess.run(["ls", directory], shell=False, check=True)  # SAFE

# B303: use_of_md5
def hash_data_vulnerable(data: bytes) -> str:
    # Bandit จะ flag: B303 Medium - MD5 ไม่ปลอดภัยสำหรับ crypto
    return hashlib.md5(data).hexdigest()  # DANGEROUS for passwords!

def hash_data_safe(data: bytes) -> str:
    # ปลอดภัย: ใช้ SHA-256 หรือ SHA-3
    return hashlib.sha256(data).hexdigest()

# B311: random - ไม่ cryptographically secure
def generate_token_vulnerable() -> str:
    # Bandit จะ flag: B311 Low - random ไม่ secure
    return str(random.randint(100000, 999999))  # INSECURE!

import secrets
def generate_token_safe() -> str:
    # ปลอดภัย: ใช้ secrets module
    return secrets.token_urlsafe(32)  # Cryptographically secure

# B301: pickle - อันตรายถ้ารับจาก untrusted sources
def load_data_vulnerable(data: bytes):
    # Bandit จะ flag: B301 Medium
    return pickle.loads(data)  # DANGEROUS with untrusted data!

import json
def load_data_safe(data: str) -> dict:
    # ปลอดภัย: ใช้ JSON แทน pickle
    return json.loads(data)
```

### 10.2 GitHub Actions CI/CD Pipeline

```yaml
# ตัวอย่างที่ 18: GitHub Actions Security Scanning Workflow
# บันทึกเป็น .github/workflows/security.yml

# name: Security Scan
# 
# on:
#   push:
#     branches: [main, develop]
#   pull_request:
#     branches: [main]
#   schedule:
#     - cron: '0 6 * * 1'  # ทุกวันจันทร์ตอนเช้า
# 
# jobs:
#   security:
#     runs-on: ubuntu-latest
#     steps:
#       - uses: actions/checkout@v4
#       
#       - name: Set up Python
#         uses: actions/setup-python@v5
#         with:
#           python-version: '3.12'
#       
#       - name: Install dependencies
#         run: |
#           pip install safety bandit pip-audit
#       
#       - name: Run Safety check
#         run: safety check -r requirements.txt --json > safety-report.json
#         continue-on-error: true
#       
#       - name: Run Bandit
#         run: bandit -r src/ -f json -o bandit-report.json -ll
#         continue-on-error: true
#       
#       - name: Run pip-audit
#         run: pip-audit --format=json > pip-audit-report.json
#       
#       - name: Upload reports
#         uses: actions/upload-artifact@v4
#         if: always()
#         with:
#           name: security-reports
#           path: '*-report.json'
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดที่ 1: SQL Injection Prevention

**โจทย์:** ฟังก์ชันต่อไปนี้มีช่องโหว่ SQL Injection แก้ไขให้ปลอดภัย

```python
# โค้ดที่มีช่องโหว่ (ต้องแก้ไข)
import sqlite3

def find_product(name: str, category: str) -> list:
    conn = sqlite3.connect('shop.db')
    cursor = conn.cursor()
    query = "SELECT * FROM products WHERE name LIKE '%" + name + "%' AND category = '" + category + "'"
    cursor.execute(query)
    return cursor.fetchall()
```

**เฉลย:**

```python
# แก้ไขแล้ว: ใช้ parameterized queries
import sqlite3

def find_product_safe(name: str, category: str) -> list:
    """
    ปลอดภัย: ใช้ ? placeholders
    SQLite จะ escape ค่าทั้งหมดโดยอัตโนมัติ
    """
    conn = sqlite3.connect('shop.db')
    cursor = conn.cursor()
    
    # ใช้ ? สำหรับ parameters ทุกตัว
    query = "SELECT * FROM products WHERE name LIKE ? AND category = ?"
    # เพิ่ม wildcards ก่อนส่งเป็น parameter
    cursor.execute(query, (f"%{name}%", category))
    return cursor.fetchall()
```

---

### แบบฝึกหัดที่ 2: Password Security

**โจทย์:** สร้างระบบ registration และ login ที่ปลอดภัยด้วย argon2

```python
# TODO: สร้างฟังก์ชัน register_user() และ login_user()
# ที่ใช้ argon2 สำหรับ hash รหัสผ่าน
# และตรวจสอบ password strength ด้วย
```

**เฉลย:**

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError
import re

ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=2)
users_db = {}  # ในชีวิตจริงใช้ database จริง

def validate_password(password: str) -> tuple[bool, str]:
    """ตรวจสอบความแข็งแรงของรหัสผ่าน"""
    if len(password) < 8:
        return False, "รหัสผ่านต้องยาวอย่างน้อย 8 ตัวอักษร"
    if not re.search(r'[A-Z]', password):
        return False, "ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว"
    if not re.search(r'[0-9]', password):
        return False, "ต้องมีตัวเลขอย่างน้อย 1 ตัว"
    if not re.search(r'[!@#$%^&*]', password):
        return False, "ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว"
    return True, "OK"


def register_user(username: str, password: str) -> dict:
    """ลงทะเบียน user ใหม่ด้วยรหัสผ่านที่ hash แล้ว"""
    if username in users_db:
        return {"success": False, "message": "Username already exists"}
    
    is_valid, msg = validate_password(password)
    if not is_valid:
        return {"success": False, "message": msg}
    
    hashed = ph.hash(password)
    users_db[username] = {"username": username, "password_hash": hashed}
    return {"success": True, "message": "Registration successful"}


def login_user(username: str, password: str) -> dict:
    """ตรวจสอบ credentials และ login"""
    user = users_db.get(username)
    if not user:
        # ใช้เวลาเท่ากันแม้ไม่มี user (ป้องกัน timing attack)
        ph.hash("dummy_password_to_prevent_timing_attack")
        return {"success": False, "message": "Invalid credentials"}
    
    try:
        ph.verify(user["password_hash"], password)
        
        # Rehash ถ้า parameters ล้าสมัย
        if ph.check_needs_rehash(user["password_hash"]):
            users_db[username]["password_hash"] = ph.hash(password)
        
        return {"success": True, "message": "Login successful"}
    except VerifyMismatchError:
        return {"success": False, "message": "Invalid credentials"}
```

---

### แบบฝึกหัดที่ 3: XSS Prevention

**โจทย์:** สร้างระบบ comment ที่อนุญาต HTML formatting พื้นฐาน (bold, italic, links) แต่ป้องกัน XSS

```python
# TODO: สร้างฟังก์ชัน process_comment(html: str) -> str
# ที่อนุญาต <b>, <i>, <a> แต่ลบ <script> และ event handlers ทั้งหมด
```

**เฉลย:**

```python
import bleach
import html as html_module

SAFE_TAGS = ['b', 'i', 'em', 'strong', 'a', 'p', 'br']
SAFE_ATTRS = {'a': ['href', 'title']}

def process_comment(user_html: str) -> str:
    """
    Process comment HTML:
    1. ตัด HTML ที่ยาวเกินไป
    2. Sanitize ด้วย bleach
    3. เพิ่ม rel="nofollow" ให้ links
    """
    # จำกัดความยาว
    if len(user_html) > 5000:
        user_html = user_html[:5000]
    
    # Sanitize
    clean = bleach.clean(
        user_html,
        tags=SAFE_TAGS,
        attributes=SAFE_ATTRS,
        strip=True,
        strip_comments=True,
    )
    
    # Auto-linkify และเพิ่ม nofollow
    clean = bleach.linkify(clean, callbacks=[bleach.callbacks.nofollow])
    
    return clean


# ทดสอบ
test_cases = [
    '<b>Bold</b> and <i>italic</i>',           # ผ่าน
    '<script>alert("XSS")</script>',             # ถูก strip
    '<a href="http://safe.com">Link</a>',        # ผ่าน
    '<a href="javascript:evil()">Click</a>',     # ถูก strip href
    '<p onclick="evil()">Text</p>',              # onclick ถูก strip
]

for test in test_cases:
    result = process_comment(test)
    print(f"Input:  {test}")
    print(f"Output: {result}\n")
```

---

### แบบฝึกหัดที่ 4: Secret Management

**โจทย์:** แก้ไขโค้ดต่อไปนี้ที่เก็บ credentials ใน code โดยตรง

```python
# โค้ดที่ไม่ปลอดภัย (ต้องแก้ไข)
import psycopg2

DATABASE_PASSWORD = "super_secret_password123"
API_KEY = "sk-prod-abc123def456"

def get_db_connection():
    return psycopg2.connect(
        host="db.production.com",
        database="myapp",
        user="admin",
        password=DATABASE_PASSWORD
    )
```

**เฉลย:**

```python
# แก้ไขแล้ว: ใช้ environment variables
import os
import psycopg2
from dotenv import load_dotenv
from pydantic_settings import BaseSettings

# โหลด .env file สำหรับ development
load_dotenv()


class DatabaseSettings(BaseSettings):
    """อ่าน database credentials จาก environment variables"""
    db_host: str = "localhost"
    db_name: str = "myapp"
    db_user: str = "postgres"
    db_password: str
    db_port: int = 5432
    api_key: str
    
    class Config:
        env_file = ".env"
        # ห้าม print หรือ log ค่า sensitive
        case_sensitive = False


def get_db_connection():
    """
    ปลอดภัย: อ่าน credentials จาก environment variables
    ไม่มี hardcoded credentials ใน code เลย
    """
    settings = DatabaseSettings()
    
    return psycopg2.connect(
        host=settings.db_host,
        database=settings.db_name,
        user=settings.db_user,
        password=settings.db_password,  # มาจาก env var
        port=settings.db_port,
        sslmode="require",              # บังคับ SSL
        connect_timeout=10,
    )

# ไฟล์ .env (ไม่ commit ลง git):
# DB_HOST=db.production.com
# DB_NAME=myapp
# DB_USER=admin
# DB_PASSWORD=super_secret_password123
# API_KEY=sk-prod-abc123def456

# ไฟล์ .gitignore ต้องมี:
# .env
# .env.local
# .env.production
```

---

### แบบฝึกหัดที่ 5: Rate Limiting

**โจทย์:** เพิ่ม rate limiting ให้ FastAPI endpoint `/api/auth/login` ป้องกัน brute force attack (สูงสุด 5 ครั้ง/นาที ต่อ IP)

```python
# TODO: เพิ่ม rate limiting ให้ endpoint นี้
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class LoginRequest(BaseModel):
    username: str
    password: str

@app.post("/api/auth/login")
async def login(credentials: LoginRequest):
    # TODO: เพิ่ม rate limiting
    result = authenticate_user(credentials.username, credentials.password)
    return result
```

**เฉลย:**

```python
from fastapi import FastAPI, Request, HTTPException
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from slowapi.middleware import SlowAPIMiddleware
from pydantic import BaseModel
import redis, time

limiter = Limiter(
    key_func=get_remote_address,
    storage_uri="redis://localhost:6379",
)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)
app.add_middleware(SlowAPIMiddleware)
redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)


class LoginRequest(BaseModel):
    username: str
    password: str


@app.post("/api/auth/login")
@limiter.limit("5/minute")
async def login(request: Request, credentials: LoginRequest):
    """Rate limited login: 5 ครั้ง/นาที/IP พร้อม progressive block"""
    client_ip = get_remote_address(request)
    block_key = f"blocked:{client_ip}"
    
    if redis_client.exists(block_key):
        ttl = redis_client.ttl(block_key)
        raise HTTPException(429, f"Blocked. Retry in {ttl}s")
    
    start = time.monotonic()
    # authenticate ด้วย argon2 จาก database (ดูตัวอย่างที่ 6)
    success = False  # แทนที่ด้วย real authentication
    
    if not success:
        failures = redis_client.incr(f"failures:{client_ip}")
        redis_client.expire(f"failures:{client_ip}", 3600)
        if failures >= 10:
            redis_client.setex(block_key, 3600, "1")
        elif failures >= 5:
            redis_client.setex(block_key, 300, "1")
        # Constant-time response ป้องกัน timing attack
        elapsed = time.monotonic() - start
        if elapsed < 0.1:
            time.sleep(0.1 - elapsed)
        raise HTTPException(401, "Invalid credentials")
    
    redis_client.delete(f"failures:{client_ip}")
    return {"message": "Login successful"}
```

---

## สรุป

ความปลอดภัยเป็นเรื่องที่ต้องคิดตั้งแต่การออกแบบ ไม่ใช่เพิ่มทีหลัง หลักการสำคัญ:

| หลักการ | รายละเอียด |
|---------|-----------|
| **Never Trust User Input** | Validate และ sanitize ทุก input |
| **Least Privilege** | ให้สิทธิ์น้อยที่สุดที่จำเป็น |
| **Defense in Depth** | ป้องกันหลายชั้น อย่าพึ่ง layer เดียว |
| **Fail Securely** | เมื่อ error ให้ secure โดย default |
| **Keep It Simple** | ระบบที่ซับซ้อนมีช่องโหว่มากกว่า |
| **Security by Default** | ค่า default ต้องปลอดภัยที่สุด |

### Checklist ก่อน Deploy

- [ ] ปิด debug mode ใน production
- [ ] ใช้ environment variables สำหรับ secrets ทั้งหมด
- [ ] Hash รหัสผ่านด้วย argon2 หรือ bcrypt
- [ ] ใช้ parameterized queries เสมอ
- [ ] เพิ่ม security headers ทุก response
- [ ] ตั้งค่า CORS ให้ถูกต้อง
- [ ] เปิด rate limiting สำหรับ auth endpoints
- [ ] สแกน dependencies ด้วย safety และ bandit
- [ ] ตั้งค่า HTTPS และ HSTS
- [ ] Log security events สำหรับ monitoring

### เครื่องมือที่แนะนำ

```
pip install argon2-cffi bcrypt PyJWT pydantic pydantic-settings
pip install python-dotenv bleach markupsafe
pip install slowapi flask-limiter
pip install safety bandit pip-audit
pip install fastapi flask sqlalchemy
```

---

*จบ Part 95 — Security Best Practices & OWASP*
