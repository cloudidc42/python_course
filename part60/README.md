# Part 60: FastAPI - Authentication, JWT & OAuth2

## สารบัญ

1. [HTTP Basic Auth](#1-http-basic-auth)
2. [API Key Authentication](#2-api-key-authentication)
3. [JWT Tokens (python-jose)](#3-jwt-tokens-python-jose)
4. [OAuth2 with Password Bearer](#4-oauth2-with-password-bearer)
5. [OAuth2 Scopes](#5-oauth2-scopes)
6. [Refresh Tokens](#6-refresh-tokens)
7. [Token Blacklisting](#7-token-blacklisting)
8. [Role-Based Access Control RBAC](#8-role-based-access-control-rbac)
9. [OAuth2 Social Login](#9-oauth2-social-login)
10. [Security Dependencies](#10-security-dependencies)
11. [HTTPS in FastAPI](#11-https-in-fastapi)
12. [ตัวอย่างโปรแกรมจริง: Complete Auth System with JWT](#12-ตัวอย่างโปรแกรมจริง-complete-auth-system-with-jwt)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. HTTP Basic Auth

HTTP Basic Auth เป็น authentication แบบง่ายที่สุด ส่ง username:password ใน Authorization header

### Basic Auth ใน FastAPI

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials
import secrets

app = FastAPI()
security = HTTPBasic()

# Mock user database
USERS = {
    "alice": "password123",
    "bob": "securepass456",
}

def verify_basic_auth(credentials: HTTPBasicCredentials = Depends(security)):
    """ตรวจสอบ HTTP Basic Auth credentials"""
    
    stored_password = USERS.get(credentials.username)
    
    if not stored_password:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid credentials",
            headers={"WWW-Authenticate": "Basic"},
        )
    
    # ใช้ secrets.compare_digest เพื่อป้องกัน timing attacks
    is_correct_password = secrets.compare_digest(
        credentials.password.encode('utf-8'),
        stored_password.encode('utf-8')
    )
    
    if not is_correct_password:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid credentials",
            headers={"WWW-Authenticate": "Basic"},
        )
    
    return credentials.username

@app.get("/basic-protected")
async def protected_route(username: str = Depends(verify_basic_auth)):
    return {"message": f"Hello, {username}! You are authenticated."}

# ทดสอบ:
# curl -u alice:password123 http://localhost:8000/basic-protected
```

### Basic Auth กับ Hash Password

```python
from passlib.context import CryptContext
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBasic, HTTPBasicCredentials

# pip install passlib[bcrypt]
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Hash passwords
USERS_HASHED = {
    "alice": pwd_context.hash("password123"),
    "bob": pwd_context.hash("securepass456"),
}

app = FastAPI()
security = HTTPBasic()

def authenticate_user(credentials: HTTPBasicCredentials = Depends(security)):
    username = credentials.username
    password = credentials.password
    
    if username not in USERS_HASHED:
        raise HTTPException(
            status_code=401,
            detail="Invalid credentials",
            headers={"WWW-Authenticate": "Basic"}
        )
    
    if not pwd_context.verify(password, USERS_HASHED[username]):
        raise HTTPException(
            status_code=401,
            detail="Invalid credentials",
            headers={"WWW-Authenticate": "Basic"}
        )
    
    return username

@app.get("/secure")
async def secure_endpoint(user: str = Depends(authenticate_user)):
    return {"user": user, "message": "Authenticated!"}
```

---

## 2. API Key Authentication

API Key เป็น authentication ที่นิยมสำหรับ machine-to-machine communication

### API Key ใน Header

```python
from fastapi import FastAPI, Depends, HTTPException, Security, status
from fastapi.security import APIKeyHeader, APIKeyQuery, APIKeyCookie
import secrets
import hashlib
from typing import Optional

app = FastAPI()

# Define security schemes
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)
api_key_query = APIKeyQuery(name="api_key", auto_error=False)

# Mock API key database
API_KEYS = {
    "user-key-1": {"user_id": 1, "name": "Alice", "tier": "premium"},
    "user-key-2": {"user_id": 2, "name": "Bob", "tier": "basic"},
    "admin-key-1": {"user_id": 0, "name": "Admin", "tier": "admin"},
}

def get_api_key(
    header_key: Optional[str] = Security(api_key_header),
    query_key: Optional[str] = Security(api_key_query),
) -> dict:
    """
    ดึง API key จาก header หรือ query parameter
    Header ควรใช้ใน production
    Query สะดวกสำหรับ testing
    """
    api_key = header_key or query_key
    
    if not api_key:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="API key is required",
            headers={"WWW-Authenticate": "APIKey"},
        )
    
    # Lookup API key
    key_data = API_KEYS.get(api_key)
    
    if not key_data:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Invalid API key"
        )
    
    return key_data

@app.get("/api/data")
async def get_data(user: dict = Depends(get_api_key)):
    return {
        "data": "Protected data",
        "requested_by": user["name"],
        "tier": user["tier"]
    }
```

### Tier-based API Keys

```python
from enum import Enum
from functools import wraps

class APITier(str, Enum):
    basic = "basic"
    premium = "premium"
    admin = "admin"

TIER_LIMITS = {
    APITier.basic: {"requests_per_hour": 100, "max_results": 10},
    APITier.premium: {"requests_per_hour": 10000, "max_results": 100},
    APITier.admin: {"requests_per_hour": float('inf'), "max_results": 1000},
}

def require_tier(minimum_tier: APITier):
    """Dependency factory สำหรับกำหนด minimum tier"""
    tier_hierarchy = {
        APITier.basic: 1,
        APITier.premium: 2,
        APITier.admin: 3,
    }
    
    def check_tier(user: dict = Depends(get_api_key)):
        user_tier = APITier(user["tier"])
        
        if tier_hierarchy[user_tier] < tier_hierarchy[minimum_tier]:
            raise HTTPException(
                status_code=403,
                detail=f"This endpoint requires {minimum_tier} tier or higher"
            )
        return user
    
    return check_tier

@app.get("/api/basic")
async def basic_endpoint(user: dict = Depends(get_api_key)):
    """ทุก tier เข้าถึงได้"""
    return {"data": "Basic data", "user": user["name"]}

@app.get("/api/premium")
async def premium_endpoint(user: dict = Depends(require_tier(APITier.premium))):
    """ต้อง premium tier ขึ้นไป"""
    return {"data": "Premium data", "user": user["name"]}

@app.get("/api/admin")
async def admin_endpoint(user: dict = Depends(require_tier(APITier.admin))):
    """ต้อง admin tier"""
    return {"data": "Admin data", "user": user["name"]}
```

### API Key Generation

```python
import secrets
import hashlib
import hmac
import base64
from datetime import datetime

def generate_api_key(user_id: int, secret_key: str = "app-secret") -> str:
    """
    สร้าง API key ที่ปลอดภัย
    Format: usr_{user_id}_{random_part}_{checksum}
    """
    random_part = secrets.token_urlsafe(24)
    
    # Create HMAC checksum
    message = f"{user_id}:{random_part}"
    checksum = hmac.new(
        secret_key.encode(),
        message.encode(),
        hashlib.sha256
    ).hexdigest()[:8]
    
    return f"usr_{user_id}_{random_part}_{checksum}"

def verify_api_key_format(api_key: str, secret_key: str = "app-secret") -> Optional[int]:
    """ตรวจสอบ format ของ API key และดึง user_id"""
    try:
        parts = api_key.split('_')
        if len(parts) != 4 or parts[0] != 'usr':
            return None
        
        user_id = int(parts[1])
        random_part = parts[2]
        checksum = parts[3]
        
        # Verify checksum
        message = f"{user_id}:{random_part}"
        expected_checksum = hmac.new(
            secret_key.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()[:8]
        
        if not hmac.compare_digest(checksum, expected_checksum):
            return None
        
        return user_id
    except Exception:
        return None

# ตัวอย่าง
key = generate_api_key(123)
print(f"Generated key: {key}")
user_id = verify_api_key_format(key)
print(f"User ID: {user_id}")
```

---

## 3. JWT Tokens (python-jose)

### การติดตั้ง

```bash
pip install python-jose[cryptography] passlib[bcrypt]
```

### JWT Token สร้างและ verify

```python
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta, timezone
from typing import Optional, Union
import os

# ====== Configuration ======
SECRET_KEY = os.getenv("SECRET_KEY", "your-super-secret-key-please-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REFRESH_TOKEN_EXPIRE_DAYS = 7

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# ====== Password Utilities ======
def hash_password(password: str) -> str:
    """Hash password ด้วย bcrypt"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """ตรวจสอบ password"""
    return pwd_context.verify(plain_password, hashed_password)

# ====== JWT Token Utilities ======
def create_access_token(
    data: dict,
    expires_delta: Optional[timedelta] = None
) -> str:
    """สร้าง JWT access token"""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({
        "exp": expire,
        "iat": datetime.now(timezone.utc),
        "type": "access"
    })
    
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def create_refresh_token(data: dict) -> str:
    """สร้าง JWT refresh token (อายุยาวกว่า)"""
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    
    to_encode.update({
        "exp": expire,
        "iat": datetime.now(timezone.utc),
        "type": "refresh"
    })
    
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    """Decode และ verify JWT token"""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError as e:
        raise ValueError(f"Invalid token: {str(e)}")

def verify_access_token(token: str) -> dict:
    """Verify access token และ return payload"""
    try:
        payload = decode_token(token)
        
        if payload.get("type") != "access":
            raise ValueError("Not an access token")
        
        username = payload.get("sub")
        if not username:
            raise ValueError("Token missing subject (sub)")
        
        return payload
    except ValueError:
        raise
    except Exception as e:
        raise ValueError(f"Token verification failed: {str(e)}")

# ====== ตัวอย่างการใช้ ======
def demo_jwt():
    # สร้าง token
    token = create_access_token(
        data={"sub": "alice", "role": "admin", "user_id": 1}
    )
    print(f"Token: {token[:50]}...")
    
    # Decode token
    payload = decode_token(token)
    print(f"Payload: {payload}")
    print(f"Subject: {payload['sub']}")
    print(f"Role: {payload['role']}")
    
    # Expired token
    expired_token = create_access_token(
        data={"sub": "alice"},
        expires_delta=timedelta(seconds=-1)  # Already expired
    )
    
    try:
        decode_token(expired_token)
    except ValueError as e:
        print(f"Expected error: {e}")
```

### JWT Payload Structure

```python
from typing import Optional, List

class TokenPayload:
    """JWT Token payload structure"""
    
    def __init__(
        self,
        sub: str,            # Subject (user ID or username)
        type: str = "access",
        role: str = "user",
        permissions: List[str] = None,
        exp: Optional[int] = None,
        iat: Optional[int] = None,
        jti: Optional[str] = None,  # JWT ID (unique identifier)
    ):
        self.sub = sub
        self.type = type
        self.role = role
        self.permissions = permissions or []
        self.exp = exp
        self.iat = iat
        self.jti = jti
    
    def to_dict(self) -> dict:
        return {
            "sub": self.sub,
            "type": self.type,
            "role": self.role,
            "permissions": self.permissions,
        }

# สร้าง token พร้อม permissions
def create_token_with_permissions(user_id: int, role: str, permissions: List[str]) -> str:
    import uuid
    
    data = {
        "sub": str(user_id),
        "role": role,
        "permissions": permissions,
        "jti": str(uuid.uuid4()),  # Unique token ID (for blacklisting)
    }
    
    return create_access_token(data)
```

---

## 4. OAuth2 with Password Bearer

### OAuth2PasswordBearer Setup

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# กำหนด OAuth2 scheme
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="auth/token",  # URL สำหรับรับ token
    scopes={
        "read": "Read access",
        "write": "Write access",
        "admin": "Admin access",
    }
)

# Mock user database
USERS_DB = {
    "alice": {
        "id": 1,
        "username": "alice",
        "email": "alice@example.com",
        "hashed_password": hash_password("password123"),
        "role": "admin",
        "disabled": False,
    },
    "bob": {
        "id": 2,
        "username": "bob",
        "email": "bob@example.com",
        "hashed_password": hash_password("password456"),
        "role": "user",
        "disabled": False,
    }
}

# ====== Pydantic Models ======
class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int

class TokenData(BaseModel):
    username: Optional[str] = None
    role: Optional[str] = None

class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    role: str

# ====== Authentication ======
def authenticate_user(username: str, password: str) -> Optional[dict]:
    """ตรวจสอบ username และ password"""
    user = USERS_DB.get(username)
    if not user:
        return None
    if not verify_password(password, user["hashed_password"]):
        return None
    if user["disabled"]:
        return None
    return user

async def get_current_user(token: str = Depends(oauth2_scheme)) -> dict:
    """Dependency ดึง current user จาก JWT token"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = verify_access_token(token)
        username: str = payload.get("sub")
        if not username:
            raise credentials_exception
    except ValueError:
        raise credentials_exception
    
    user = USERS_DB.get(username)
    if not user:
        raise credentials_exception
    
    return user

async def get_current_active_user(
    current_user: dict = Depends(get_current_user)
) -> dict:
    """ตรวจสอบว่า user ไม่ถูก disable"""
    if current_user["disabled"]:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# ====== Endpoints ======
@app.post("/auth/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """
    Login endpoint ที่ใช้ OAuth2 form data
    Content-Type: application/x-www-form-urlencoded
    Body: username=alice&password=password123
    """
    user = authenticate_user(form_data.username, form_data.password)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    access_token = create_access_token(
        data={"sub": user["username"], "role": user["role"], "user_id": user["id"]}
    )
    
    return Token(
        access_token=access_token,
        token_type="bearer",
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )

@app.get("/auth/me", response_model=UserResponse)
async def get_me(current_user: dict = Depends(get_current_active_user)):
    """ดู profile ของตัวเอง"""
    return UserResponse(
        id=current_user["id"],
        username=current_user["username"],
        email=current_user["email"],
        role=current_user["role"]
    )

@app.get("/users")
async def list_users(current_user: dict = Depends(get_current_active_user)):
    """ดูรายชื่อ users"""
    return {"users": list(USERS_DB.keys()), "requested_by": current_user["username"]}
```

---

## 5. OAuth2 Scopes

OAuth2 Scopes ช่วยให้ควบคุม permissions แบบละเอียดกว่า roles

```python
from fastapi import FastAPI, Depends, HTTPException, Security, status
from fastapi.security import OAuth2PasswordBearer, SecurityScopes
from jose import jwt, JWTError
from typing import List, Optional

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="auth/token",
    scopes={
        "users:read": "Read user data",
        "users:write": "Create/update users",
        "posts:read": "Read posts",
        "posts:write": "Create/update posts",
        "admin": "Admin operations",
    }
)

# User ที่มี scopes ต่างกัน
USERS_DB = {
    "alice": {
        "id": 1,
        "username": "alice",
        "password": hash_password("password123"),
        "scopes": ["users:read", "users:write", "posts:read", "posts:write", "admin"],
    },
    "bob": {
        "id": 2,
        "username": "bob",
        "password": hash_password("password456"),
        "scopes": ["users:read", "posts:read"],
    }
}

def create_scoped_token(username: str, scopes: List[str]) -> str:
    """สร้าง token พร้อม scopes"""
    data = {
        "sub": username,
        "scopes": scopes,
    }
    return create_access_token(data)

async def get_current_user_with_scopes(
    security_scopes: SecurityScopes,
    token: str = Depends(oauth2_scheme)
) -> dict:
    """
    Dependency ที่ตรวจสอบ scopes ด้วย
    
    security_scopes.scopes: list ของ scopes ที่ endpoint ต้องการ
    """
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
    
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": authenticate_value},
    )
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if not username:
            raise credentials_exception
        
        token_scopes = payload.get("scopes", [])
        token_data = {"username": username, "scopes": token_scopes}
    except JWTError:
        raise credentials_exception
    
    user = USERS_DB.get(username)
    if not user:
        raise credentials_exception
    
    # ตรวจสอบ scopes
    for scope in security_scopes.scopes:
        if scope not in token_data["scopes"]:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Not enough permissions. Required scope: {scope}",
                headers={"WWW-Authenticate": authenticate_value},
            )
    
    return user

# Endpoints ที่ใช้ scopes
@app.get("/users/me")
async def read_users_me(
    current_user: dict = Security(get_current_user_with_scopes, scopes=["users:read"])
):
    """ต้องการ users:read scope"""
    return {"username": current_user["username"]}

@app.post("/users")
async def create_user_endpoint(
    user_data: dict,
    current_user: dict = Security(get_current_user_with_scopes, scopes=["users:write"])
):
    """ต้องการ users:write scope"""
    return {"message": "User created", "by": current_user["username"]}

@app.get("/posts")
async def read_posts(
    current_user: dict = Security(get_current_user_with_scopes, scopes=["posts:read"])
):
    """ต้องการ posts:read scope"""
    return {"posts": [], "reader": current_user["username"]}

@app.delete("/admin/users/{user_id}")
async def admin_delete_user(
    user_id: int,
    current_user: dict = Security(get_current_user_with_scopes, scopes=["admin"])
):
    """ต้องการ admin scope"""
    return {"deleted": user_id, "by": current_user["username"]}

# Login ที่รองรับ scopes
@app.post("/auth/token")
async def login_with_scopes(form_data: OAuth2PasswordRequestForm = Depends()):
    """Login พร้อม requested scopes"""
    user = USERS_DB.get(form_data.username)
    
    if not user or not verify_password(form_data.password, user["password"]):
        raise HTTPException(401, "Incorrect username or password")
    
    # ตรวจสอบว่า requested scopes อยู่ใน user's scopes
    requested_scopes = form_data.scopes
    available_scopes = user["scopes"]
    
    # Grant only scopes that user has
    granted_scopes = [s for s in requested_scopes if s in available_scopes]
    if not granted_scopes:
        granted_scopes = available_scopes  # Default: grant all user's scopes
    
    token = create_scoped_token(user["username"], granted_scopes)
    
    return {
        "access_token": token,
        "token_type": "bearer",
        "scopes": granted_scopes
    }
```

---

## 6. Refresh Tokens

Refresh tokens ช่วยให้ user ไม่ต้อง login บ่อยๆ โดยใช้ refresh token เพื่อรับ access token ใหม่

```python
from fastapi import FastAPI, Depends, HTTPException, status
from pydantic import BaseModel
from typing import Optional, Dict, Set
import secrets
from datetime import datetime, timedelta, timezone

app = FastAPI()

# Storage สำหรับ refresh tokens (ใช้ Redis ใน production)
refresh_token_store: Dict[str, dict] = {}  # token -> {user_id, expires_at, device_id}

class TokenPair(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    expires_in: int

class RefreshRequest(BaseModel):
    refresh_token: str

def generate_refresh_token() -> str:
    """สร้าง random refresh token"""
    return secrets.token_urlsafe(48)

def store_refresh_token(
    refresh_token: str,
    user_id: int,
    username: str,
    device_id: Optional[str] = None
):
    """เก็บ refresh token"""
    expires_at = datetime.now(timezone.utc) + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    refresh_token_store[refresh_token] = {
        "user_id": user_id,
        "username": username,
        "expires_at": expires_at,
        "device_id": device_id,
        "created_at": datetime.now(timezone.utc),
    }

def validate_refresh_token(refresh_token: str) -> Optional[dict]:
    """ตรวจสอบ refresh token"""
    token_data = refresh_token_store.get(refresh_token)
    
    if not token_data:
        return None
    
    if datetime.now(timezone.utc) > token_data["expires_at"]:
        # Token expired - ลบออก
        del refresh_token_store[refresh_token]
        return None
    
    return token_data

def revoke_refresh_token(refresh_token: str) -> bool:
    """ยกเลิก refresh token"""
    if refresh_token in refresh_token_store:
        del refresh_token_store[refresh_token]
        return True
    return False

# ====== Endpoints ======
@app.post("/auth/login", response_model=TokenPair)
async def login_with_refresh(
    form_data: OAuth2PasswordRequestForm = Depends(),
    device_id: Optional[str] = None,
):
    """Login และรับ token pair"""
    user = USERS_DB.get(form_data.username)
    
    if not user or not verify_password(form_data.password, user["password"]):
        raise HTTPException(401, "Invalid credentials")
    
    # สร้าง access token
    access_token = create_access_token(
        data={"sub": user["username"], "user_id": user["id"], "role": user.get("role", "user")}
    )
    
    # สร้าง refresh token
    refresh_token = generate_refresh_token()
    store_refresh_token(
        refresh_token=refresh_token,
        user_id=user["id"],
        username=user["username"],
        device_id=device_id
    )
    
    return TokenPair(
        access_token=access_token,
        refresh_token=refresh_token,
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )

@app.post("/auth/refresh", response_model=TokenPair)
async def refresh_token_endpoint(request: RefreshRequest):
    """ใช้ refresh token เพื่อรับ access token ใหม่"""
    
    token_data = validate_refresh_token(request.refresh_token)
    
    if not token_data:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired refresh token"
        )
    
    # ยกเลิก refresh token เก่า (Token Rotation)
    revoke_refresh_token(request.refresh_token)
    
    # สร้าง token ใหม่
    new_access_token = create_access_token(
        data={
            "sub": token_data["username"],
            "user_id": token_data["user_id"]
        }
    )
    
    new_refresh_token = generate_refresh_token()
    store_refresh_token(
        refresh_token=new_refresh_token,
        user_id=token_data["user_id"],
        username=token_data["username"],
        device_id=token_data.get("device_id")
    )
    
    return TokenPair(
        access_token=new_access_token,
        refresh_token=new_refresh_token,
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )

@app.post("/auth/logout")
async def logout(
    request: RefreshRequest,
    current_user: dict = Depends(get_current_active_user)
):
    """Logout - ยกเลิก refresh token"""
    revoke_refresh_token(request.refresh_token)
    return {"message": "Logged out successfully"}

@app.post("/auth/logout-all")
async def logout_all_devices(current_user: dict = Depends(get_current_active_user)):
    """Logout จากทุก devices"""
    user_id = current_user["id"]
    
    # ลบ refresh tokens ทั้งหมดของ user
    tokens_to_remove = [
        token for token, data in refresh_token_store.items()
        if data["user_id"] == user_id
    ]
    
    for token in tokens_to_remove:
        del refresh_token_store[token]
    
    return {
        "message": f"Logged out from {len(tokens_to_remove)} devices",
        "devices_removed": len(tokens_to_remove)
    }
```

---

## 7. Token Blacklisting

Token blacklisting ช่วยให้ invalidate token ก่อน หมดอายุ

```python
from fastapi import FastAPI, Depends, HTTPException
from typing import Set, Dict, Optional
from datetime import datetime, timezone
import asyncio

app = FastAPI()

# In-memory blacklist (ใช้ Redis ใน production)
token_blacklist: Set[str] = set()
blacklist_with_expiry: Dict[str, datetime] = {}

def blacklist_token(jti: str, expires_at: Optional[datetime] = None):
    """เพิ่ม JWT ID เข้า blacklist"""
    token_blacklist.add(jti)
    if expires_at:
        blacklist_with_expiry[jti] = expires_at

def is_token_blacklisted(jti: str) -> bool:
    """ตรวจสอบว่า token อยู่ใน blacklist"""
    return jti in token_blacklist

async def cleanup_expired_blacklist():
    """ลบ entries ที่ expired ออกจาก blacklist (ควรรันเป็น background task)"""
    now = datetime.now(timezone.utc)
    expired = [
        jti for jti, expires_at in blacklist_with_expiry.items()
        if now > expires_at
    ]
    for jti in expired:
        token_blacklist.discard(jti)
        del blacklist_with_expiry[jti]
    
    if expired:
        print(f"Cleaned up {len(expired)} expired blacklisted tokens")

# Token ที่มี JTI (JWT ID)
import uuid

def create_token_with_jti(data: dict) -> tuple[str, str]:
    """สร้าง token พร้อม JTI และ return (token, jti)"""
    jti = str(uuid.uuid4())
    data_with_jti = {**data, "jti": jti}
    token = create_access_token(data_with_jti)
    return token, jti

async def get_current_user_with_blacklist_check(
    token: str = Depends(oauth2_scheme)
) -> dict:
    """Dependency ที่ตรวจสอบ blacklist ด้วย"""
    try:
        payload = verify_access_token(token)
    except ValueError:
        raise HTTPException(401, "Invalid token")
    
    # Check blacklist
    jti = payload.get("jti")
    if jti and is_token_blacklisted(jti):
        raise HTTPException(
            status_code=401,
            detail="Token has been revoked",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    username = payload.get("sub")
    user = USERS_DB.get(username)
    if not user:
        raise HTTPException(401, "User not found")
    
    return user

@app.post("/auth/logout-jwt")
async def logout_with_blacklist(
    token: str = Depends(oauth2_scheme),
    current_user: dict = Depends(get_current_user_with_blacklist_check)
):
    """Logout โดย blacklist token"""
    try:
        payload = verify_access_token(token)
        jti = payload.get("jti")
        
        if jti:
            # ดึงเวลา expires
            exp = payload.get("exp")
            if exp:
                expires_at = datetime.fromtimestamp(exp, tz=timezone.utc)
                blacklist_token(jti, expires_at)
            else:
                blacklist_token(jti)
    except Exception:
        pass
    
    return {"message": "Logged out successfully"}

@app.get("/protected")
async def protected(current_user: dict = Depends(get_current_user_with_blacklist_check)):
    return {"user": current_user["username"], "message": "Access granted"}
```

---

## 8. Role-Based Access Control RBAC

```python
from fastapi import FastAPI, Depends, HTTPException, status
from enum import Enum
from typing import List, Optional, Set

app = FastAPI()

class Role(str, Enum):
    guest = "guest"
    user = "user"
    moderator = "moderator"
    admin = "admin"
    superadmin = "superadmin"

# Role hierarchy
ROLE_HIERARCHY = {
    Role.guest: 0,
    Role.user: 1,
    Role.moderator: 2,
    Role.admin: 3,
    Role.superadmin: 4,
}

# Permissions สำหรับแต่ละ role
ROLE_PERMISSIONS: dict[Role, Set[str]] = {
    Role.guest: {"read:public"},
    Role.user: {"read:public", "read:own", "write:own", "delete:own"},
    Role.moderator: {"read:public", "read:own", "write:own", "delete:own",
                     "read:all", "moderate:content"},
    Role.admin: {"read:public", "read:own", "write:own", "delete:own",
                 "read:all", "write:all", "delete:all", "moderate:content",
                 "manage:users"},
    Role.superadmin: {"*"},  # All permissions
}

# ====== Helper Functions ======
def has_role(user_role: str, required_role: Role) -> bool:
    """ตรวจสอบว่า user มี role ที่ต้องการหรือสูงกว่า"""
    try:
        user_role_enum = Role(user_role)
        return ROLE_HIERARCHY.get(user_role_enum, 0) >= ROLE_HIERARCHY.get(required_role, 0)
    except ValueError:
        return False

def has_permission(user_role: str, permission: str) -> bool:
    """ตรวจสอบว่า user มี permission ที่ต้องการ"""
    try:
        role = Role(user_role)
        permissions = ROLE_PERMISSIONS.get(role, set())
        return "*" in permissions or permission in permissions
    except ValueError:
        return False

# ====== Role Dependencies ======
def require_role(minimum_role: Role):
    """Dependency factory สำหรับ require minimum role"""
    async def check_role(current_user: dict = Depends(get_current_active_user)):
        user_role = current_user.get("role", "user")
        
        if not has_role(user_role, minimum_role):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Requires role: {minimum_role.value} or higher. "
                       f"Current role: {user_role}"
            )
        return current_user
    return check_role

def require_permission(permission: str):
    """Dependency factory สำหรับ require specific permission"""
    async def check_permission(current_user: dict = Depends(get_current_active_user)):
        user_role = current_user.get("role", "user")
        
        if not has_permission(user_role, permission):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Missing permission: {permission}"
            )
        return current_user
    return check_permission

def require_any_permission(*permissions: str):
    """ต้องมี permission อย่างน้อย 1 อย่าง"""
    async def check_permissions(current_user: dict = Depends(get_current_active_user)):
        user_role = current_user.get("role", "user")
        
        if not any(has_permission(user_role, perm) for perm in permissions):
            raise HTTPException(
                status_code=403,
                detail=f"Missing one of permissions: {', '.join(permissions)}"
            )
        return current_user
    return check_permissions

# ====== Endpoints ======
@app.get("/public")
async def public_data():
    """ทุกคนเข้าถึงได้"""
    return {"data": "Public information"}

@app.get("/user-profile")
async def get_user_profile(
    user: dict = Depends(require_role(Role.user))
):
    """ต้องเป็น user ขึ้นไป"""
    return {"profile": user}

@app.get("/moderate/reports")
async def view_reports(
    user: dict = Depends(require_role(Role.moderator))
):
    """ต้องเป็น moderator ขึ้นไป"""
    return {"reports": []}

@app.get("/admin/users")
async def list_all_users(
    user: dict = Depends(require_role(Role.admin))
):
    """ต้องเป็น admin ขึ้นไป"""
    return {"users": list(USERS_DB.keys()), "requested_by": user["username"]}

@app.delete("/admin/users/{user_id}")
async def delete_user(
    user_id: int,
    user: dict = Depends(require_permission("manage:users"))
):
    """ต้องมี manage:users permission"""
    return {"deleted": user_id, "by": user["username"]}

@app.post("/content/moderate")
async def moderate_content(
    content_id: int,
    action: str,
    user: dict = Depends(require_any_permission("moderate:content", "manage:users"))
):
    """ต้องมี moderate:content หรือ manage:users permission"""
    return {"moderated": content_id, "action": action, "by": user["username"]}

# ====== Resource Ownership ======
async def get_resource_owner_or_admin(
    resource_id: int,
    current_user: dict = Depends(get_current_active_user)
) -> dict:
    """ตรวจสอบว่าเป็นเจ้าของ resource หรือ admin"""
    
    # Simulate fetching resource
    resource_owner_id = 1  # In real app: fetch from DB
    
    user_id = current_user.get("id")
    is_admin = has_role(current_user.get("role", "user"), Role.admin)
    
    if user_id != resource_owner_id and not is_admin:
        raise HTTPException(
            status_code=403,
            detail="You don't have access to this resource"
        )
    
    return current_user

@app.delete("/posts/{post_id}")
async def delete_post(
    post_id: int,
    user: dict = Depends(get_resource_owner_or_admin)
):
    """ลบ post (เจ้าของหรือ admin เท่านั้น)"""
    return {"deleted": post_id, "by": user["username"]}
```

---

## 9. OAuth2 Social Login

### Google OAuth2

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import RedirectResponse
import httpx
import os
import secrets

app = FastAPI()

GOOGLE_CLIENT_ID = os.getenv("GOOGLE_CLIENT_ID")
GOOGLE_CLIENT_SECRET = os.getenv("GOOGLE_CLIENT_SECRET")
GOOGLE_REDIRECT_URI = "http://localhost:8000/auth/google/callback"

GOOGLE_AUTH_URL = "https://accounts.google.com/o/oauth2/v2/auth"
GOOGLE_TOKEN_URL = "https://oauth2.googleapis.com/token"
GOOGLE_USERINFO_URL = "https://www.googleapis.com/oauth2/v3/userinfo"

# State storage (ใช้ Redis ใน production)
oauth_states: dict = {}

@app.get("/auth/google")
async def google_login():
    """เริ่มต้น Google OAuth2 flow"""
    state = secrets.token_urlsafe(32)
    oauth_states[state] = {"created_at": "now"}
    
    params = {
        "client_id": GOOGLE_CLIENT_ID,
        "redirect_uri": GOOGLE_REDIRECT_URI,
        "response_type": "code",
        "scope": "openid email profile",
        "state": state,
        "access_type": "offline",  # รับ refresh token
        "prompt": "select_account",
    }
    
    query_string = "&".join(f"{k}={v}" for k, v in params.items())
    auth_url = f"{GOOGLE_AUTH_URL}?{query_string}"
    
    return RedirectResponse(url=auth_url)

@app.get("/auth/google/callback")
async def google_callback(code: str, state: str, request: Request):
    """รับ callback จาก Google"""
    
    # Verify state (CSRF protection)
    if state not in oauth_states:
        raise HTTPException(400, "Invalid state parameter")
    del oauth_states[state]
    
    # Exchange code สำหรับ token
    async with httpx.AsyncClient() as client:
        token_response = await client.post(
            GOOGLE_TOKEN_URL,
            data={
                "client_id": GOOGLE_CLIENT_ID,
                "client_secret": GOOGLE_CLIENT_SECRET,
                "code": code,
                "grant_type": "authorization_code",
                "redirect_uri": GOOGLE_REDIRECT_URI,
            }
        )
        
        if token_response.status_code != 200:
            raise HTTPException(400, "Failed to exchange code for token")
        
        token_data = token_response.json()
        google_access_token = token_data["access_token"]
        
        # ดึงข้อมูล user จาก Google
        user_response = await client.get(
            GOOGLE_USERINFO_URL,
            headers={"Authorization": f"Bearer {google_access_token}"}
        )
        
        if user_response.status_code != 200:
            raise HTTPException(400, "Failed to get user info")
        
        google_user = user_response.json()
    
    # สร้าง/อัพเดท user ใน database
    user = await get_or_create_user_from_google(google_user)
    
    # สร้าง JWT token สำหรับ app ของเรา
    access_token = create_access_token(
        data={"sub": str(user["id"]), "email": user["email"]}
    )
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "user": {
            "id": user["id"],
            "email": user["email"],
            "name": user["name"],
        }
    }

async def get_or_create_user_from_google(google_user: dict) -> dict:
    """สร้างหรือดึง user จาก Google profile"""
    email = google_user.get("email")
    
    # In real app: ค้นหา user ใน database ด้วย email
    # ถ้าไม่มี ให้สร้างใหม่
    
    user = {
        "id": 100,
        "email": email,
        "name": google_user.get("name"),
        "picture": google_user.get("picture"),
        "provider": "google",
        "provider_id": google_user.get("sub"),
    }
    
    return user
```

### GitHub OAuth2

```python
GITHUB_CLIENT_ID = os.getenv("GITHUB_CLIENT_ID")
GITHUB_CLIENT_SECRET = os.getenv("GITHUB_CLIENT_SECRET")
GITHUB_REDIRECT_URI = "http://localhost:8000/auth/github/callback"

@app.get("/auth/github")
async def github_login():
    """เริ่มต้น GitHub OAuth2 flow"""
    state = secrets.token_urlsafe(32)
    oauth_states[state] = {"created_at": "now"}
    
    params = {
        "client_id": GITHUB_CLIENT_ID,
        "redirect_uri": GITHUB_REDIRECT_URI,
        "scope": "read:user user:email",
        "state": state,
    }
    
    query_string = "&".join(f"{k}={v}" for k, v in params.items())
    auth_url = f"https://github.com/login/oauth/authorize?{query_string}"
    
    return RedirectResponse(url=auth_url)

@app.get("/auth/github/callback")
async def github_callback(code: str, state: str):
    """รับ callback จาก GitHub"""
    
    if state not in oauth_states:
        raise HTTPException(400, "Invalid state")
    del oauth_states[state]
    
    async with httpx.AsyncClient() as client:
        # Exchange code สำหรับ access token
        token_response = await client.post(
            "https://github.com/login/oauth/access_token",
            data={
                "client_id": GITHUB_CLIENT_ID,
                "client_secret": GITHUB_CLIENT_SECRET,
                "code": code,
                "redirect_uri": GITHUB_REDIRECT_URI,
            },
            headers={"Accept": "application/json"}
        )
        
        token_data = token_response.json()
        github_token = token_data.get("access_token")
        
        if not github_token:
            raise HTTPException(400, "Failed to get GitHub token")
        
        # ดึงข้อมูล user
        user_response = await client.get(
            "https://api.github.com/user",
            headers={
                "Authorization": f"Bearer {github_token}",
                "Accept": "application/json"
            }
        )
        
        github_user = user_response.json()
        
        # ดึง email (GitHub อาจไม่ส่ง email ใน /user)
        if not github_user.get("email"):
            emails_response = await client.get(
                "https://api.github.com/user/emails",
                headers={"Authorization": f"Bearer {github_token}"}
            )
            emails = emails_response.json()
            primary_email = next(
                (e["email"] for e in emails if e["primary"]), None
            )
            github_user["email"] = primary_email
    
    user = {
        "id": github_user["id"],
        "username": github_user["login"],
        "email": github_user.get("email"),
        "name": github_user.get("name"),
        "avatar_url": github_user.get("avatar_url"),
        "provider": "github",
    }
    
    access_token = create_access_token(
        data={"sub": str(user["id"]), "username": user["username"]}
    )
    
    return {"access_token": access_token, "user": user}
```

---

## 10. Security Dependencies

### Composable Security Dependencies

```python
from fastapi import FastAPI, Depends, HTTPException, Security, Header
from fastapi.security import OAuth2PasswordBearer, APIKeyHeader
from typing import Optional

app = FastAPI()

# ====== Multiple Auth Methods ======
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token", auto_error=False)
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

async def authenticate(
    bearer_token: Optional[str] = Depends(oauth2_scheme),
    api_key: Optional[str] = Security(api_key_header),
) -> dict:
    """
    Support multiple authentication methods:
    1. JWT Bearer token
    2. API Key
    """
    # ลอง JWT Bearer ก่อน
    if bearer_token:
        try:
            payload = verify_access_token(bearer_token)
            user = USERS_DB.get(payload.get("sub"))
            if user:
                return {**user, "auth_method": "jwt"}
        except ValueError:
            pass
    
    # ลอง API Key
    if api_key:
        key_data = API_KEYS.get(api_key)
        if key_data:
            user_id = key_data["user_id"]
            # Find user
            for user in USERS_DB.values():
                if user["id"] == user_id:
                    return {**user, "auth_method": "api_key"}
    
    raise HTTPException(
        status_code=401,
        detail="Authentication required",
        headers={"WWW-Authenticate": "Bearer"}
    )

@app.get("/data")
async def get_data(user: dict = Depends(authenticate)):
    return {
        "data": "Protected",
        "user": user["username"],
        "auth_method": user.get("auth_method")
    }
```

### Request Context Security

```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityMiddleware(BaseHTTPMiddleware):
    """
    Security middleware ที่ทำ:
    1. ตรวจสอบ request headers
    2. Block suspicious requests
    3. Add security headers
    """
    
    BLOCKED_USER_AGENTS = ["sqlmap", "nikto", "nmap"]
    MAX_CONTENT_LENGTH = 10 * 1024 * 1024  # 10MB
    
    async def dispatch(self, request: Request, call_next):
        # Check Content-Length
        content_length = request.headers.get("content-length", 0)
        if int(content_length) > self.MAX_CONTENT_LENGTH:
            from fastapi.responses import JSONResponse
            return JSONResponse(
                status_code=413,
                content={"error": "Request too large"}
            )
        
        # Check User-Agent
        user_agent = request.headers.get("user-agent", "").lower()
        for blocked_agent in self.BLOCKED_USER_AGENTS:
            if blocked_agent in user_agent:
                from fastapi.responses import JSONResponse
                return JSONResponse(
                    status_code=403,
                    content={"error": "Blocked"}
                )
        
        response = await call_next(request)
        
        # Add security headers
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        
        return response

app.add_middleware(SecurityMiddleware)
```

---

## 11. HTTPS in FastAPI

### Development HTTPS

```bash
# สร้าง self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# รัน uvicorn ด้วย HTTPS
uvicorn main:app --ssl-keyfile=key.pem --ssl-certfile=cert.pem
```

### Production HTTPS

```python
# main.py - Production setup
import uvicorn
import ssl

if __name__ == "__main__":
    # Production: ใช้ HTTPS
    ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
    ssl_context.load_cert_chain('/path/to/cert.pem', '/path/to/key.pem')
    
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=443,
        ssl_keyfile="/path/to/key.pem",
        ssl_certfile="/path/to/cert.pem",
        workers=4,
    )
```

### HTTP to HTTPS Redirect

```python
from fastapi import FastAPI, Request
from fastapi.responses import RedirectResponse
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

# Redirect HTTP ไป HTTPS อัตโนมัติ (ใช้ใน production)
# app.add_middleware(HTTPSRedirectMiddleware)

# หรือทำเอง
@app.middleware("http")
async def redirect_http_to_https(request: Request, call_next):
    if request.url.scheme == "http":
        url = request.url.replace(scheme="https")
        return RedirectResponse(url=str(url), status_code=301)
    return await call_next(request)
```

---

## 12. ตัวอย่างโปรแกรมจริง: Complete Auth System with JWT

```python
# auth_system.py - Complete Authentication System
from fastapi import FastAPI, Depends, HTTPException, status, Security, BackgroundTasks
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm, APIKeyHeader
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field, field_validator
from jose import jwt, JWTError
from passlib.context import CryptContext
from typing import Optional, List, Dict, Set
from datetime import datetime, timedelta, timezone
from enum import Enum
import secrets
import uuid
import re
import os

# ==================== Configuration ====================
SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key-change-in-prod-12345")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
REFRESH_TOKEN_EXPIRE_DAYS = 7

# ==================== Password Hashing ====================
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

# ==================== JWT Utilities ====================
def create_token(data: dict, expires_delta: timedelta, token_type: str = "access") -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + expires_delta
    jti = str(uuid.uuid4())
    to_encode.update({"exp": expire, "iat": datetime.now(timezone.utc), "jti": jti, "type": token_type})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM), jti

def decode_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError as e:
        raise ValueError(str(e))

# ==================== Storage ====================
users_db: Dict[str, dict] = {}  # username -> user dict
api_keys_db: Dict[str, dict] = {}  # api_key -> key data
blacklisted_tokens: Set[str] = set()  # Set of blacklisted JTIs
refresh_tokens_db: Dict[str, dict] = {}  # refresh_token -> data

# ==================== Enums ====================
class UserRole(str, Enum):
    user = "user"
    moderator = "moderator"
    admin = "admin"

# ==================== Pydantic Models ====================
class UserRegister(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, pattern=r'^[a-zA-Z0-9_]+$')
    email: str = Field(...)
    password: str = Field(..., min_length=8)
    full_name: Optional[str] = None
    
    @field_validator('email')
    @classmethod
    def validate_email(cls, v):
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', v):
            raise ValueError('Invalid email')
        return v.lower()
    
    @field_validator('password')
    @classmethod
    def validate_password(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Must have uppercase')
        if not re.search(r'[0-9]', v):
            raise ValueError('Must have number')
        return v

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    expires_in: int

class UserResponse(BaseModel):
    id: str
    username: str
    email: str
    full_name: Optional[str]
    role: str
    is_active: bool
    created_at: datetime

class ChangePassword(BaseModel):
    current_password: str
    new_password: str = Field(..., min_length=8)

class APIKeyCreate(BaseModel):
    name: str = Field(..., min_length=3, max_length=100)
    scopes: List[str] = ["read"]

class APIKeyResponse(BaseModel):
    id: str
    name: str
    key: str  # Only shown once!
    scopes: List[str]
    created_at: datetime

# ==================== Security Schemes ====================
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login", auto_error=False)
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

# ==================== Dependencies ====================
async def get_current_user(
    bearer_token: Optional[str] = Depends(oauth2_scheme),
    api_key: Optional[str] = Security(api_key_header),
) -> dict:
    """Get user from JWT bearer token or API key"""
    
    # Try JWT first
    if bearer_token:
        try:
            payload = decode_token(bearer_token)
            
            if payload.get("type") != "access":
                raise ValueError("Not an access token")
            
            jti = payload.get("jti")
            if jti and jti in blacklisted_tokens:
                raise ValueError("Token has been revoked")
            
            user_id = payload.get("sub")
            user = next((u for u in users_db.values() if u["id"] == user_id), None)
            
            if user and user["is_active"]:
                return {**user, "auth_method": "jwt"}
        except ValueError:
            pass
    
    # Try API key
    if api_key:
        key_data = api_keys_db.get(api_key)
        if key_data and key_data.get("is_active"):
            user = next((u for u in users_db.values() if u["id"] == key_data["user_id"]), None)
            if user and user["is_active"]:
                return {**user, "auth_method": "api_key", "api_key_scopes": key_data["scopes"]}
    
    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Authentication required",
        headers={"WWW-Authenticate": "Bearer"},
    )

def require_role(minimum_role: UserRole):
    role_levels = {UserRole.user: 1, UserRole.moderator: 2, UserRole.admin: 3}
    
    async def check(user: dict = Depends(get_current_user)):
        user_level = role_levels.get(UserRole(user.get("role", "user")), 0)
        required_level = role_levels.get(minimum_role, 0)
        
        if user_level < required_level:
            raise HTTPException(403, f"Requires {minimum_role.value} role")
        return user
    return check

# ==================== FastAPI App ====================
app = FastAPI(
    title="Auth System API",
    description="Complete Authentication System with JWT, Refresh Tokens, and API Keys",
    version="1.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# ==================== Auth Endpoints ====================
@app.post("/auth/register", response_model=UserResponse, status_code=201, tags=["auth"])
async def register(user_data: UserRegister):
    """ลงทะเบียน user ใหม่"""
    if user_data.username.lower() in users_db:
        raise HTTPException(409, "Username already taken")
    
    if any(u["email"] == user_data.email for u in users_db.values()):
        raise HTTPException(409, "Email already registered")
    
    user = {
        "id": str(uuid.uuid4()),
        "username": user_data.username.lower(),
        "email": user_data.email,
        "password_hash": hash_password(user_data.password),
        "full_name": user_data.full_name,
        "role": UserRole.user.value,
        "is_active": True,
        "created_at": datetime.now(timezone.utc),
    }
    users_db[user["username"]] = user
    
    return UserResponse(**{k: v for k, v in user.items() if k != "password_hash"})

@app.post("/auth/login", response_model=TokenResponse, tags=["auth"])
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """Login ด้วย username/password"""
    user = users_db.get(form_data.username.lower())
    
    if not user or not verify_password(form_data.password, user["password_hash"]):
        raise HTTPException(
            status_code=401,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"}
        )
    
    if not user["is_active"]:
        raise HTTPException(400, "Account is disabled")
    
    # Create token pair
    access_token, access_jti = create_token(
        {"sub": user["id"], "role": user["role"]},
        timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
        "access"
    )
    
    refresh_token = secrets.token_urlsafe(48)
    refresh_tokens_db[refresh_token] = {
        "user_id": user["id"],
        "username": user["username"],
        "expires_at": datetime.now(timezone.utc) + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS),
        "created_at": datetime.now(timezone.utc),
    }
    
    return TokenResponse(
        access_token=access_token,
        refresh_token=refresh_token,
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )

@app.post("/auth/refresh", response_model=TokenResponse, tags=["auth"])
async def refresh_token(refresh_token: str):
    """ต่ออายุ token ด้วย refresh token"""
    token_data = refresh_tokens_db.get(refresh_token)
    
    if not token_data:
        raise HTTPException(401, "Invalid refresh token")
    
    if datetime.now(timezone.utc) > token_data["expires_at"]:
        del refresh_tokens_db[refresh_token]
        raise HTTPException(401, "Refresh token expired")
    
    # Token rotation - invalidate old refresh token
    del refresh_tokens_db[refresh_token]
    
    user = next((u for u in users_db.values() if u["id"] == token_data["user_id"]), None)
    if not user or not user["is_active"]:
        raise HTTPException(401, "User not found or inactive")
    
    # Create new tokens
    new_access, _ = create_token(
        {"sub": user["id"], "role": user["role"]},
        timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
        "access"
    )
    
    new_refresh = secrets.token_urlsafe(48)
    refresh_tokens_db[new_refresh] = {
        "user_id": user["id"],
        "username": user["username"],
        "expires_at": datetime.now(timezone.utc) + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS),
        "created_at": datetime.now(timezone.utc),
    }
    
    return TokenResponse(
        access_token=new_access,
        refresh_token=new_refresh,
        expires_in=ACCESS_TOKEN_EXPIRE_MINUTES * 60
    )

@app.post("/auth/logout", tags=["auth"])
async def logout(
    refresh_token: Optional[str] = None,
    current_user: dict = Depends(get_current_user),
    bearer_token: Optional[str] = Depends(oauth2_scheme)
):
    """Logout - blacklist current token"""
    # Blacklist access token
    if bearer_token:
        try:
            payload = decode_token(bearer_token)
            jti = payload.get("jti")
            if jti:
                blacklisted_tokens.add(jti)
        except Exception:
            pass
    
    # Revoke refresh token
    if refresh_token and refresh_token in refresh_tokens_db:
        del refresh_tokens_db[refresh_token]
    
    return {"message": "Logged out successfully"}

# ==================== User Endpoints ====================
@app.get("/users/me", response_model=UserResponse, tags=["users"])
async def get_me(current_user: dict = Depends(get_current_user)):
    """ดู profile ของตัวเอง"""
    return UserResponse(**{k: v for k, v in current_user.items() 
                          if k not in ["password_hash", "auth_method", "api_key_scopes"]})

@app.put("/users/me/password", tags=["users"])
async def change_password(
    data: ChangePassword,
    current_user: dict = Depends(get_current_user)
):
    """เปลี่ยน password"""
    if not verify_password(data.current_password, current_user["password_hash"]):
        raise HTTPException(400, "Current password is incorrect")
    
    users_db[current_user["username"]]["password_hash"] = hash_password(data.new_password)
    
    # Revoke all refresh tokens
    user_id = current_user["id"]
    to_remove = [t for t, d in refresh_tokens_db.items() if d["user_id"] == user_id]
    for t in to_remove:
        del refresh_tokens_db[t]
    
    return {"message": "Password changed successfully. Please login again."}

# ==================== API Keys ====================
@app.post("/api-keys", response_model=APIKeyResponse, status_code=201, tags=["api-keys"])
async def create_api_key(
    key_data: APIKeyCreate,
    current_user: dict = Depends(get_current_user)
):
    """สร้าง API key ใหม่"""
    key = secrets.token_urlsafe(32)
    key_id = str(uuid.uuid4())
    
    api_key_record = {
        "id": key_id,
        "key": key,
        "name": key_data.name,
        "user_id": current_user["id"],
        "scopes": key_data.scopes,
        "is_active": True,
        "created_at": datetime.now(timezone.utc),
    }
    
    api_keys_db[key] = api_key_record
    
    return APIKeyResponse(**api_key_record)

@app.get("/api-keys", tags=["api-keys"])
async def list_api_keys(current_user: dict = Depends(get_current_user)):
    """ดู API keys ทั้งหมดของตัวเอง (ไม่แสดง key value)"""
    user_keys = [
        {k: v for k, v in data.items() if k != "key"}  # Hide actual key
        for data in api_keys_db.values()
        if data["user_id"] == current_user["id"]
    ]
    return {"api_keys": user_keys}

@app.delete("/api-keys/{key_id}", tags=["api-keys"])
async def revoke_api_key(
    key_id: str,
    current_user: dict = Depends(get_current_user)
):
    """ยกเลิก API key"""
    key = next((k for k, d in api_keys_db.items() if d["id"] == key_id), None)
    
    if not key:
        raise HTTPException(404, "API key not found")
    
    if api_keys_db[key]["user_id"] != current_user["id"]:
        raise HTTPException(403, "Not your API key")
    
    del api_keys_db[key]
    return {"message": "API key revoked"}

# ==================== Admin Endpoints ====================
@app.get("/admin/users", tags=["admin"])
async def admin_list_users(
    admin: dict = Depends(require_role(UserRole.admin))
):
    """Admin: ดูรายชื่อ users ทั้งหมด"""
    return {
        "users": [
            {k: v for k, v in u.items() if k != "password_hash"}
            for u in users_db.values()
        ],
        "total": len(users_db)
    }

@app.put("/admin/users/{username}/role", tags=["admin"])
async def admin_change_role(
    username: str,
    new_role: UserRole,
    admin: dict = Depends(require_role(UserRole.admin))
):
    """Admin: เปลี่ยน role ของ user"""
    user = users_db.get(username)
    if not user:
        raise HTTPException(404, f"User {username} not found")
    
    users_db[username]["role"] = new_role.value
    return {"message": f"Role updated to {new_role.value}", "username": username}

@app.put("/admin/users/{username}/deactivate", tags=["admin"])
async def admin_deactivate_user(
    username: str,
    admin: dict = Depends(require_role(UserRole.admin))
):
    """Admin: ปิดการใช้งาน user"""
    user = users_db.get(username)
    if not user:
        raise HTTPException(404, f"User {username} not found")
    
    if username == admin["username"]:
        raise HTTPException(400, "Cannot deactivate yourself")
    
    users_db[username]["is_active"] = False
    return {"message": f"User {username} deactivated"}

# ==================== Startup ======
@app.on_event("startup")
async def startup():
    """สร้าง demo users"""
    demo_data = [
        ("alice", "alice@example.com", "Password123!", UserRole.admin),
        ("bob", "bob@example.com", "Password456!", UserRole.user),
        ("charlie", "charlie@example.com", "Password789!", UserRole.moderator),
    ]
    
    for username, email, password, role in demo_data:
        user = {
            "id": str(uuid.uuid4()),
            "username": username,
            "email": email,
            "password_hash": hash_password(password),
            "full_name": username.capitalize(),
            "role": role.value,
            "is_active": True,
            "created_at": datetime.now(timezone.utc),
        }
        users_db[username] = user
    
    print("Auth System started!")
    print("Credentials:")
    print("  Admin: alice / Password123!")
    print("  User:  bob / Password456!")
    print("  Mod:   charlie / Password789!")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)
```

---

## 13. แบบฝึกหัด

### แบบฝึกหัดที่ 1: HTTP Basic Auth

**โจทย์**: สร้าง admin panel endpoint ที่ใช้ HTTP Basic Auth พร้อม timing-attack protection

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBasic, HTTPBasicCredentials
import secrets, hashlib

app = FastAPI()
security = HTTPBasic()

ADMIN_USERS = {
    "admin": hashlib.sha256("admin123".encode()).hexdigest(),
    "operator": hashlib.sha256("operator456".encode()).hexdigest(),
}

def require_basic_auth(creds: HTTPBasicCredentials = Depends(security)):
    stored_hash = ADMIN_USERS.get(creds.username, "")
    input_hash = hashlib.sha256(creds.password.encode()).hexdigest()
    
    # Timing-safe comparison
    if not stored_hash or not secrets.compare_digest(stored_hash, input_hash):
        raise HTTPException(
            status_code=401,
            detail="Invalid credentials",
            headers={"WWW-Authenticate": "Basic realm='Admin Panel'"}
        )
    return creds.username

@app.get("/admin/dashboard")
async def dashboard(user: str = Depends(require_basic_auth)):
    return {"admin": user, "dashboard": "Admin Panel"}
```

### แบบฝึกหัดที่ 2: API Key System

**โจทย์**: สร้าง API key management system ที่มี tier-based access

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException, Security
from fastapi.security import APIKeyHeader
from typing import Optional
import secrets

app = FastAPI()
api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

KEYS_DB = {
    "free-key-001": {"tier": "free", "user_id": 1, "limit": 100},
    "pro-key-001": {"tier": "pro", "user_id": 2, "limit": 10000},
    "enterprise-key-001": {"tier": "enterprise", "user_id": 3, "limit": float('inf')},
}

TIER_RANKS = {"free": 1, "pro": 2, "enterprise": 3}

def get_api_key(key: Optional[str] = Security(api_key_header)):
    if not key:
        raise HTTPException(401, "API key required")
    key_data = KEYS_DB.get(key)
    if not key_data:
        raise HTTPException(403, "Invalid API key")
    return key_data

def require_tier(min_tier: str):
    def check(key_data: dict = Depends(get_api_key)):
        if TIER_RANKS.get(key_data["tier"], 0) < TIER_RANKS.get(min_tier, 0):
            raise HTTPException(403, f"Requires {min_tier} tier")
        return key_data
    return check

@app.get("/api/basic")
async def basic_api(key: dict = Depends(get_api_key)):
    return {"data": "Free data", "tier": key["tier"]}

@app.get("/api/advanced")
async def advanced_api(key: dict = Depends(require_tier("pro"))):
    return {"data": "Pro data", "tier": key["tier"]}
```

### แบบฝึกหัดที่ 3: JWT Authentication

**โจทย์**: สร้าง complete JWT auth system ด้วย passlib และ python-jose

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import jwt, JWTError
from passlib.context import CryptContext
from pydantic import BaseModel
from datetime import datetime, timedelta, timezone
from typing import Optional

SECRET = "jwt-secret-key-2024"
ALGORITHM = "HS256"
EXPIRE_MINUTES = 60

pwd = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2 = OAuth2PasswordBearer(tokenUrl="/token")

USERS = {
    "alice": {"id": 1, "password": pwd.hash("pass123"), "role": "admin"},
    "bob": {"id": 2, "password": pwd.hash("pass456"), "role": "user"},
}

app = FastAPI()

class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"

def make_token(username: str, role: str) -> str:
    exp = datetime.now(timezone.utc) + timedelta(minutes=EXPIRE_MINUTES)
    return jwt.encode({"sub": username, "role": role, "exp": exp}, SECRET, ALGORITHM)

async def current_user(token: str = Depends(oauth2)):
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGORITHM])
        username = payload["sub"]
        user = USERS.get(username)
        if not user:
            raise ValueError()
        return {"username": username, "role": payload["role"], **user}
    except (JWTError, ValueError):
        raise HTTPException(401, "Invalid token", headers={"WWW-Authenticate": "Bearer"})

@app.post("/token", response_model=Token)
async def login(form: OAuth2PasswordRequestForm = Depends()):
    user = USERS.get(form.username)
    if not user or not pwd.verify(form.password, user["password"]):
        raise HTTPException(401, "Wrong credentials")
    return Token(access_token=make_token(form.username, user["role"]))

@app.get("/me")
async def me(user: dict = Depends(current_user)):
    return {"username": user["username"], "role": user["role"]}
```

### แบบฝึกหัดที่ 4: Refresh Token

**โจทย์**: เพิ่ม refresh token mechanism ให้กับ JWT auth

**เฉลย**:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import secrets
from datetime import datetime, timedelta, timezone

app = FastAPI()
refresh_store = {}

class TokenPair(BaseModel):
    access_token: str
    refresh_token: str

@app.post("/auth/login", response_model=TokenPair)
async def login(username: str, password: str):
    # Validate (simplified)
    if not (username == "alice" and password == "pass123"):
        raise HTTPException(401, "Invalid credentials")
    
    access = make_token(username, "user")
    refresh = secrets.token_urlsafe(32)
    refresh_store[refresh] = {
        "username": username,
        "exp": datetime.now(timezone.utc) + timedelta(days=7)
    }
    return TokenPair(access_token=access, refresh_token=refresh)

@app.post("/auth/refresh", response_model=TokenPair)
async def refresh(refresh_token: str):
    data = refresh_store.get(refresh_token)
    if not data or datetime.now(timezone.utc) > data["exp"]:
        raise HTTPException(401, "Invalid or expired refresh token")
    
    # Rotate: delete old, create new
    del refresh_store[refresh_token]
    
    new_access = make_token(data["username"], "user")
    new_refresh = secrets.token_urlsafe(32)
    refresh_store[new_refresh] = {
        "username": data["username"],
        "exp": datetime.now(timezone.utc) + timedelta(days=7)
    }
    return TokenPair(access_token=new_access, refresh_token=new_refresh)
```

### แบบฝึกหัดที่ 5: RBAC

**โจทย์**: สร้าง RBAC system ที่มี role hierarchy

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException
from enum import IntEnum

app = FastAPI()

class Role(IntEnum):
    guest = 0
    user = 1
    mod = 2
    admin = 3

USERS = {
    "alice": {"role": Role.admin, "id": 1},
    "bob": {"role": Role.user, "id": 2},
    "charlie": {"role": Role.mod, "id": 3},
}

def get_user():
    # Simplified - in real app: verify JWT token
    return USERS["alice"]

def require_role(min_role: Role):
    def check(user: dict = Depends(get_user)):
        if user["role"] < min_role:
            raise HTTPException(403, f"Requires {min_role.name} role")
        return user
    return check

@app.get("/public")
async def public():
    return {"data": "Public"}

@app.get("/user-area")
async def user_area(user: dict = Depends(require_role(Role.user))):
    return {"data": "User area", "role": user["role"].name}

@app.get("/mod-area")
async def mod_area(user: dict = Depends(require_role(Role.mod))):
    return {"data": "Mod area"}

@app.get("/admin-area")
async def admin_area(user: dict = Depends(require_role(Role.admin))):
    return {"data": "Admin area"}
```

### แบบฝึกหัดที่ 6: Token Blacklist

**โจทย์**: Implement token blacklist สำหรับ logout

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer
from typing import Set
import uuid
from datetime import datetime, timedelta, timezone

app = FastAPI()
oauth2 = OAuth2PasswordBearer(tokenUrl="/login")
blacklist: Set[str] = set()

def create_token_with_jti(user_id: str) -> tuple[str, str]:
    jti = str(uuid.uuid4())
    exp = datetime.now(timezone.utc) + timedelta(hours=1)
    token = jwt.encode({"sub": user_id, "jti": jti, "exp": exp}, SECRET, ALGORITHM)
    return token, jti

async def verified_user(token: str = Depends(oauth2)):
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGORITHM])
        jti = payload.get("jti")
        if jti in blacklist:
            raise HTTPException(401, "Token has been revoked")
        return payload
    except JWTError:
        raise HTTPException(401, "Invalid token")

@app.post("/logout")
async def logout(
    token: str = Depends(oauth2),
    user: dict = Depends(verified_user)
):
    jti = user.get("jti")
    if jti:
        blacklist.add(jti)
    return {"message": "Logged out"}

@app.get("/me")
async def me(user: dict = Depends(verified_user)):
    return {"user_id": user["sub"]}
```

### แบบฝึกหัดที่ 7: OAuth2 Scopes

**โจทย์**: สร้าง endpoint ที่ใช้ scopes ต่างกัน

**เฉลย**:

```python
from fastapi import FastAPI, Security
from fastapi.security import OAuth2PasswordBearer, SecurityScopes
from typing import List

app = FastAPI()
oauth2 = OAuth2PasswordBearer(
    tokenUrl="/token",
    scopes={"read": "Read access", "write": "Write", "admin": "Admin"}
)

async def check_scopes(security_scopes: SecurityScopes, token: str = Depends(oauth2)):
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGORITHM])
        token_scopes: List[str] = payload.get("scopes", [])
        
        for scope in security_scopes.scopes:
            if scope not in token_scopes:
                raise HTTPException(
                    403,
                    f"Missing scope: {scope}",
                    headers={"WWW-Authenticate": f'Bearer scope="{security_scopes.scope_str}"'}
                )
        return payload
    except JWTError:
        raise HTTPException(401, "Invalid token")

@app.get("/items")
async def read_items(user: dict = Security(check_scopes, scopes=["read"])):
    return {"items": [], "reader": user["sub"]}

@app.post("/items")
async def create_item(user: dict = Security(check_scopes, scopes=["read", "write"])):
    return {"created": True}

@app.delete("/items/{id}")
async def delete_item(id: int, user: dict = Security(check_scopes, scopes=["admin"])):
    return {"deleted": id}
```

### แบบฝึกหัดที่ 8: Complete Auth Mini Project

**โจทย์**: สร้าง complete mini auth project ที่รวมทุกอย่าง

**เฉลย**:

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import jwt, JWTError
from passlib.context import CryptContext
from pydantic import BaseModel
from typing import Optional, Dict, Set
from datetime import datetime, timedelta, timezone
import secrets, uuid

# Config
SECRET = "mini-auth-secret-2024"
ALGORITHM = "HS256"
ACCESS_EXPIRE = 30  # minutes
REFRESH_EXPIRE = 7  # days

# Setup
pwd = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2 = OAuth2PasswordBearer(tokenUrl="/auth/token")

# Storage
USERS: Dict[str, dict] = {
    "alice": {
        "id": "u1", "username": "alice",
        "hashed_pw": pwd.hash("Alice123!"),
        "role": "admin", "active": True
    },
    "bob": {
        "id": "u2", "username": "bob",
        "hashed_pw": pwd.hash("Bob456!"),
        "role": "user", "active": True
    }
}
REFRESH_TOKENS: Dict[str, dict] = {}
BLACKLIST: Set[str] = set()

# Models
class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

class UserOut(BaseModel):
    id: str
    username: str
    role: str

# Utils
def make_access(user_id: str, role: str) -> tuple:
    jti = str(uuid.uuid4())
    exp = datetime.now(timezone.utc) + timedelta(minutes=ACCESS_EXPIRE)
    token = jwt.encode({"sub": user_id, "role": role, "jti": jti, "exp": exp, "type": "access"}, SECRET, ALGORITHM)
    return token, jti

def get_current_user(token: str = Depends(oauth2)) -> dict:
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGORITHM])
        if payload.get("type") != "access":
            raise ValueError()
        if payload.get("jti") in BLACKLIST:
            raise ValueError("Revoked")
        user = next((u for u in USERS.values() if u["id"] == payload["sub"]), None)
        if not user or not user["active"]:
            raise ValueError()
        return user
    except (JWTError, ValueError):
        raise HTTPException(401, "Invalid token", headers={"WWW-Authenticate": "Bearer"})

# App
app = FastAPI(title="Mini Auth", version="1.0")

@app.post("/auth/token", response_model=Token)
async def login(form: OAuth2PasswordRequestForm = Depends()):
    user = USERS.get(form.username)
    if not user or not pwd.verify(form.password, user["hashed_pw"]):
        raise HTTPException(401, "Wrong credentials")
    
    access, jti = make_access(user["id"], user["role"])
    refresh = secrets.token_urlsafe(32)
    REFRESH_TOKENS[refresh] = {
        "user_id": user["id"],
        "exp": datetime.now(timezone.utc) + timedelta(days=REFRESH_EXPIRE)
    }
    return Token(access_token=access, refresh_token=refresh)

@app.post("/auth/refresh", response_model=Token)
async def refresh(token: str):
    data = REFRESH_TOKENS.get(token)
    if not data or datetime.now(timezone.utc) > data["exp"]:
        raise HTTPException(401, "Invalid or expired refresh token")
    
    del REFRESH_TOKENS[token]
    user = next((u for u in USERS.values() if u["id"] == data["user_id"]), None)
    if not user:
        raise HTTPException(401, "User not found")
    
    new_access, _ = make_access(user["id"], user["role"])
    new_refresh = secrets.token_urlsafe(32)
    REFRESH_TOKENS[new_refresh] = {
        "user_id": user["id"],
        "exp": datetime.now(timezone.utc) + timedelta(days=REFRESH_EXPIRE)
    }
    return Token(access_token=new_access, refresh_token=new_refresh)

@app.post("/auth/logout")
async def logout(
    current_user: dict = Depends(get_current_user),
    token: str = Depends(oauth2)
):
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGORITHM])
        BLACKLIST.add(payload["jti"])
    except JWTError:
        pass
    return {"message": "Logged out"}

@app.get("/me", response_model=UserOut)
async def me(user: dict = Depends(get_current_user)):
    return UserOut(id=user["id"], username=user["username"], role=user["role"])

@app.get("/admin/dashboard")
async def admin_dashboard(user: dict = Depends(get_current_user)):
    if user["role"] != "admin":
        raise HTTPException(403, "Admin only")
    return {"users": len(USERS), "tokens": len(REFRESH_TOKENS)}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, reload=True)
    # Login: alice / Alice123!  or  bob / Bob456!
```

---

## สรุป

ใน Part 60 เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียนรู้ |
|--------|-------------------|
| HTTP Basic Auth | HTTPBasic, timing-attack protection |
| API Key Auth | Header/Query API keys, tier-based |
| JWT Tokens | python-jose, create/decode/verify |
| OAuth2 Bearer | OAuth2PasswordBearer, form login |
| OAuth2 Scopes | SecurityScopes, granular permissions |
| Refresh Tokens | Token pairs, rotation, revocation |
| Token Blacklist | JTI-based blacklisting, cleanup |
| RBAC | Role hierarchy, permission checking |
| Social Login | Google OAuth2, GitHub OAuth2 |
| Security Middleware | SecurityHeaders, request filtering |
| HTTPS | SSL setup, HTTP redirect |

### Security Best Practices

```python
# 1. ใช้ strong SECRET_KEY
SECRET_KEY = secrets.token_urlsafe(64)  # ใน production

# 2. ใช้ bcrypt สำหรับ password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# 3. ตั้งค่า CORS อย่างเข้มงวด
ALLOWED_ORIGINS = ["https://yourdomain.com"]

# 4. ใช้ HTTPS เสมอ (ใน production)
# uvicorn main:app --ssl-keyfile=key.pem --ssl-certfile=cert.pem

# 5. Rate limit login endpoint
# 6. Log authentication events
# 7. ใช้ refresh token rotation
# 8. Set short expiry สำหรับ access tokens (15-60 นาที)
# 9. ตรวจสอบ token blacklist ทุก request
# 10. ใช้ Redis สำหรับ blacklist และ refresh tokens ใน production
```

### ขั้นตอนต่อไป

- **Part 61**: FastAPI - Testing & Deployment
- **Part 62**: Docker สำหรับ Python Applications
- **Part 63**: CI/CD Pipelines
