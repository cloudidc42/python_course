# Part 46: Environment Variables & Configuration Management

## สารบัญ

1. [Environment Variables คืออะไร?](#1-environment-variables-คืออะไร)
2. [การใช้ os.environ](#2-การใช้-osenviron)
3. [python-dotenv และ .env files](#3-python-dotenv-และ-env-files)
4. [Configuration Management Patterns](#4-configuration-management-patterns)
5. [12-Factor App Methodology](#5-12-factor-app-methodology)
6. [Settings Class Pattern](#6-settings-class-pattern)
7. [Pydantic Settings](#7-pydantic-settings)
8. [Configuration Validation](#8-configuration-validation)
9. [Environment-Specific Configs](#9-environment-specific-configs)
10. [Secrets Management Best Practices](#10-secrets-management-best-practices)
11. [แบบฝึกหัด](#11-แบบฝึกหัด)

---

## 1. Environment Variables คืออะไร?

**Environment Variables** หรือ **ตัวแปรสภาพแวดล้อม** คือค่าตัวแปรที่ถูกกำหนดไว้ในระดับ Operating System หรือ Shell Environment ซึ่งโปรแกรมสามารถอ่านค่าเหล่านี้ได้ขณะ runtime

### ทำไมต้องใช้ Environment Variables?

1. **Security (ความปลอดภัย)**: ไม่ต้อง hardcode secrets เช่น passwords, API keys ลงใน source code
2. **Portability (พกพาได้)**: โปรแกรมเดียวกันรันได้หลาย environment (dev, staging, production)
3. **Flexibility (ยืดหยุ่น)**: เปลี่ยน configuration ได้โดยไม่ต้องแก้ code
4. **Collaboration (ทำงานร่วมกัน)**: แต่ละนักพัฒนาใช้ค่าของตัวเองได้
5. **DevOps Best Practice**: เป็น standard ที่ยอมรับกันทั่วโลก

### ตัวอย่างปัญหาเมื่อไม่ใช้ Environment Variables

```python
# ❌ แบบที่ผิด - อย่าทำแบบนี้!
DATABASE_URL = "postgresql://admin:SuperSecret123@production-db.company.com/myapp"
API_KEY = "sk-1234567890abcdef"
SECRET_KEY = "my-super-secret-key-dont-share"

def connect_to_database():
    # ค่า password ถูก commit ขึ้น GitHub แล้ว!
    return psycopg2.connect(DATABASE_URL)
```

```python
# ✅ แบบที่ถูก - ใช้ Environment Variables
import os

DATABASE_URL = os.environ.get("DATABASE_URL")
API_KEY = os.environ.get("API_KEY")
SECRET_KEY = os.environ.get("SECRET_KEY")

def connect_to_database():
    # ปลอดภัย! ค่าจริงไม่อยู่ใน code
    return psycopg2.connect(DATABASE_URL)
```

---

## 2. การใช้ os.environ

`os.environ` คือ dictionary-like object ที่เก็บ environment variables ทั้งหมดของระบบ

### ตัวอย่างที่ 1: การอ่าน Environment Variables พื้นฐาน

```python
import os

# วิธีที่ 1: อ่านโดยตรง (จะ raise KeyError ถ้าไม่มี)
path = os.environ["PATH"]
print(f"PATH: {path}")

# วิธีที่ 2: ใช้ .get() (คืนค่า None ถ้าไม่มี)
home = os.environ.get("HOME")
print(f"HOME: {home}")

# วิธีที่ 3: ใช้ .get() พร้อม default value
port = os.environ.get("PORT", "8080")
print(f"PORT: {port}")

# วิธีที่ 4: ตรวจสอบว่ามีตัวแปรหรือไม่
if "DATABASE_URL" in os.environ:
    print("DATABASE_URL มีอยู่แล้ว")
else:
    print("DATABASE_URL ไม่มี")
```

### ตัวอย่างที่ 2: การแสดง Environment Variables ทั้งหมด

```python
import os

def show_all_env_vars():
    """แสดง environment variables ทั้งหมด"""
    print("=== All Environment Variables ===")
    for key, value in sorted(os.environ.items()):
        # ซ่อนค่า sensitive variables
        if any(sensitive in key.upper() for sensitive in ['SECRET', 'PASSWORD', 'KEY', 'TOKEN']):
            print(f"{key} = ***HIDDEN***")
        else:
            # truncate ค่าที่ยาวเกินไป
            display_value = value[:50] + "..." if len(value) > 50 else value
            print(f"{key} = {display_value}")

show_all_env_vars()
```

### ตัวอย่างที่ 3: การตั้งค่า Environment Variables ใน Python

```python
import os

# ตั้งค่า environment variable (จะมีผลแค่ใน process นี้)
os.environ["MY_APP_DEBUG"] = "true"
os.environ["MY_APP_PORT"] = "9000"

print(os.environ.get("MY_APP_DEBUG"))  # "true"
print(os.environ.get("MY_APP_PORT"))   # "9000"

# ลบ environment variable
del os.environ["MY_APP_DEBUG"]
# หรือ
os.environ.pop("MY_APP_DEBUG", None)  # ไม่ raise error ถ้าไม่มี
```

### ตัวอย่างที่ 4: Type Conversion จาก Environment Variables

```python
import os

def get_env_bool(key: str, default: bool = False) -> bool:
    """อ่าน boolean จาก environment variable"""
    value = os.environ.get(key, "").lower()
    if value in ("true", "1", "yes", "on"):
        return True
    elif value in ("false", "0", "no", "off"):
        return False
    return default

def get_env_int(key: str, default: int = 0) -> int:
    """อ่าน integer จาก environment variable"""
    value = os.environ.get(key)
    if value is None:
        return default
    try:
        return int(value)
    except ValueError:
        print(f"Warning: {key}='{value}' is not a valid integer, using default {default}")
        return default

def get_env_list(key: str, separator: str = ",", default: list = None) -> list:
    """อ่าน list จาก environment variable (comma-separated)"""
    value = os.environ.get(key)
    if value is None:
        return default or []
    return [item.strip() for item in value.split(separator) if item.strip()]

# ตัวอย่างการใช้งาน
os.environ["DEBUG"] = "true"
os.environ["PORT"] = "8080"
os.environ["ALLOWED_HOSTS"] = "localhost, 127.0.0.1, example.com"

debug = get_env_bool("DEBUG")          # True
port = get_env_int("PORT")             # 8080
hosts = get_env_list("ALLOWED_HOSTS") # ['localhost', '127.0.0.1', 'example.com']

print(f"Debug: {debug} ({type(debug).__name__})")
print(f"Port: {port} ({type(port).__name__})")
print(f"Hosts: {hosts} ({type(hosts).__name__})")
```

### ตัวอย่างที่ 5: การใช้ os.getenv

```python
import os

# os.getenv() เหมือน os.environ.get() แต่เป็น convenience function
database_host = os.getenv("DB_HOST", "localhost")
database_port = os.getenv("DB_PORT", "5432")
database_name = os.getenv("DB_NAME", "myapp")

print(f"Connecting to: {database_host}:{database_port}/{database_name}")

# ตรวจสอบ required environment variables
required_vars = ["SECRET_KEY", "DATABASE_URL"]
missing_vars = [var for var in required_vars if not os.getenv(var)]

if missing_vars:
    raise EnvironmentError(
        f"Missing required environment variables: {', '.join(missing_vars)}\n"
        f"Please set these in your .env file or environment"
    )
```

---

## 3. python-dotenv และ .env files

**python-dotenv** คือ library ที่ช่วยโหลด environment variables จากไฟล์ `.env` ซึ่งเป็น text file ที่เก็บ key=value pairs

### การติดตั้ง

```bash
pip install python-dotenv
```

### โครงสร้างไฟล์ .env

```bash
# .env file example
# Comment lines start with #

# Application Settings
APP_NAME=MyAwesomeApp
APP_ENV=development
DEBUG=true
SECRET_KEY=your-secret-key-here-change-in-production

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydb
DB_USER=user
DB_PASSWORD=password

# External Services
REDIS_URL=redis://localhost:6379/0
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASSWORD=your-app-password

# API Keys
OPENAI_API_KEY=sk-...
STRIPE_SECRET_KEY=sk_test_...
SENDGRID_API_KEY=SG....

# Feature Flags
FEATURE_NEW_UI=true
FEATURE_ANALYTICS=false

# Server Settings
HOST=0.0.0.0
PORT=8000
WORKERS=4
```

### ตัวอย่างที่ 6: การใช้ python-dotenv พื้นฐาน

```python
from dotenv import load_dotenv
import os

# โหลด .env file จาก directory ปัจจุบัน
load_dotenv()

# ตอนนี้สามารถอ่านค่าจาก os.environ ได้เลย
app_name = os.getenv("APP_NAME")
debug = os.getenv("DEBUG", "false").lower() == "true"
database_url = os.getenv("DATABASE_URL")

print(f"App: {app_name}")
print(f"Debug: {debug}")
print(f"Database: {database_url}")
```

### ตัวอย่างที่ 7: dotenv_values และ find_dotenv

```python
from dotenv import load_dotenv, dotenv_values, find_dotenv
import os

# find_dotenv() หาไฟล์ .env โดยอัตโนมัติ (ค้นหาจาก directory ปัจจุบันขึ้นไป)
dotenv_path = find_dotenv()
print(f"Found .env at: {dotenv_path}")

# โหลดไฟล์ .env จาก path ที่กำหนด
load_dotenv(dotenv_path)

# dotenv_values() อ่านค่าเป็น dict โดยไม่ต้อง set environment variables
config = dotenv_values(".env")
print(f"Config: {config}")

# โหลด .env file หลายไฟล์ (ค่าหลังทับค่าก่อน)
load_dotenv(".env")           # base config
load_dotenv(".env.local", override=True)  # local overrides

# override=True ทับค่าที่มีอยู่แล้ว
# override=False (default) ไม่ทับค่าที่มีอยู่แล้ว
```

### ตัวอย่างที่ 8: โครงสร้างไฟล์ .env สำหรับหลาย Environment

```python
"""
โครงสร้าง project:
myproject/
├── .env                # ค่า default (commit ได้ ถ้าไม่มี secrets)
├── .env.development    # สำหรับ dev
├── .env.staging        # สำหรับ staging
├── .env.production     # สำหรับ production (ไม่ควร commit!)
├── .env.test          # สำหรับ testing
├── .env.example       # template ที่ commit ได้ (ไม่มีค่าจริง)
└── .gitignore         # ต้อง add .env* (ยกเว้น .env.example)
"""

from dotenv import load_dotenv
import os

def load_env_for_environment():
    """โหลด .env file ตาม environment ปัจจุบัน"""
    env = os.getenv("APP_ENV", "development")
    
    # โหลดตามลำดับ (หลังสุด override ก่อน)
    load_dotenv(".env")                    # base defaults
    load_dotenv(f".env.{env}")             # environment-specific
    load_dotenv(".env.local", override=True)  # local overrides (ไม่ commit)
    
    print(f"Loaded config for environment: {env}")

load_env_for_environment()
```

### ตัวอย่างที่ 9: .env.example Template

```bash
# .env.example - โปรแกรมใหม่ให้ copy ไฟล์นี้เป็น .env แล้วใส่ค่าจริง
# คำสั่ง: cp .env.example .env

# Application
APP_NAME=MyApp
APP_ENV=development
DEBUG=true
SECRET_KEY=change-this-to-a-random-string

# Database (required)
DATABASE_URL=postgresql://USER:PASSWORD@HOST:PORT/DB_NAME

# Redis (optional)
REDIS_URL=redis://localhost:6379/0

# Email (required for production)
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_USER=your@email.com
EMAIL_PASSWORD=your-password

# API Keys (get from respective services)
# OPENAI_API_KEY=sk-...
# STRIPE_SECRET_KEY=sk_test_...
```

---

## 4. Configuration Management Patterns

### Pattern 1: Simple Dictionary Config

```python
import os
from dotenv import load_dotenv

load_dotenv()

# Simple config dictionary
config = {
    "debug": os.getenv("DEBUG", "false").lower() == "true",
    "database_url": os.getenv("DATABASE_URL", "sqlite:///./app.db"),
    "secret_key": os.getenv("SECRET_KEY", "dev-secret-key"),
    "host": os.getenv("HOST", "0.0.0.0"),
    "port": int(os.getenv("PORT", "8000")),
}

print(config)
```

### Pattern 2: Config Class

```python
import os
from dataclasses import dataclass, field
from typing import List, Optional
from dotenv import load_dotenv

load_dotenv()

@dataclass
class DatabaseConfig:
    url: str = field(default_factory=lambda: os.getenv("DATABASE_URL", "sqlite:///./app.db"))
    pool_size: int = field(default_factory=lambda: int(os.getenv("DB_POOL_SIZE", "5")))
    max_overflow: int = field(default_factory=lambda: int(os.getenv("DB_MAX_OVERFLOW", "10")))
    echo: bool = field(default_factory=lambda: os.getenv("DB_ECHO", "false").lower() == "true")

@dataclass
class RedisConfig:
    url: str = field(default_factory=lambda: os.getenv("REDIS_URL", "redis://localhost:6379/0"))
    max_connections: int = field(default_factory=lambda: int(os.getenv("REDIS_MAX_CONN", "10")))

@dataclass
class AppConfig:
    name: str = field(default_factory=lambda: os.getenv("APP_NAME", "MyApp"))
    env: str = field(default_factory=lambda: os.getenv("APP_ENV", "development"))
    debug: bool = field(default_factory=lambda: os.getenv("DEBUG", "false").lower() == "true")
    secret_key: str = field(default_factory=lambda: os.getenv("SECRET_KEY", "dev-secret"))
    allowed_hosts: List[str] = field(
        default_factory=lambda: [
            h.strip() for h in os.getenv("ALLOWED_HOSTS", "localhost").split(",")
        ]
    )
    database: DatabaseConfig = field(default_factory=DatabaseConfig)
    redis: RedisConfig = field(default_factory=RedisConfig)

    @property
    def is_production(self) -> bool:
        return self.env == "production"

    @property
    def is_development(self) -> bool:
        return self.env == "development"

# Global config instance
app_config = AppConfig()

print(f"App: {app_config.name}")
print(f"Env: {app_config.env}")
print(f"Is Production: {app_config.is_production}")
print(f"Database URL: {app_config.database.url}")
```

### Pattern 3: Singleton Config

```python
import os
from typing import Optional
from dotenv import load_dotenv

class Config:
    """Singleton pattern สำหรับ configuration"""
    _instance: Optional["Config"] = None
    _initialized: bool = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if not self._initialized:
            load_dotenv()
            self._load_config()
            Config._initialized = True

    def _load_config(self):
        self.debug = self._get_bool("DEBUG", False)
        self.testing = self._get_bool("TESTING", False)
        self.secret_key = os.environ["SECRET_KEY"]  # Required!
        self.database_url = os.getenv("DATABASE_URL", "sqlite:///./app.db")
        self.allowed_hosts = self._get_list("ALLOWED_HOSTS", ["localhost"])
        self.port = self._get_int("PORT", 8000)

    def _get_bool(self, key: str, default: bool = False) -> bool:
        return os.getenv(key, str(default)).lower() in ("true", "1", "yes")

    def _get_int(self, key: str, default: int = 0) -> int:
        try:
            return int(os.getenv(key, str(default)))
        except ValueError:
            return default

    def _get_list(self, key: str, default: list = None) -> list:
        value = os.getenv(key)
        if not value:
            return default or []
        return [item.strip() for item in value.split(",")]

    def __repr__(self):
        return f"Config(env={os.getenv('APP_ENV', 'development')}, debug={self.debug})"

# ใช้งาน
config1 = Config()
config2 = Config()
print(config1 is config2)  # True - same instance!
```

---

## 5. 12-Factor App Methodology

**12-Factor App** คือ methodology สำหรับสร้าง software-as-a-service ที่ scalable, maintainable และ portable

Factor III ของ 12-Factor กล่าวว่า: **"Store config in the environment"**

### หลักการสำคัญ

```python
"""
12-Factor App Principles ที่เกี่ยวกับ Config:

Factor III - Config:
- Config คือทุกอย่างที่แตกต่างกันระหว่าง deployments (dev, staging, prod)
- ต้องไม่มี config ใดอยู่ใน code
- ใช้ environment variables สำหรับ config
- Test: โค้ดสามารถ open source ได้โดยไม่เปิดเผย credentials

สิ่งที่ควรเป็น Config (ไม่ใช่ hardcode):
- Database URLs
- Credentials สำหรับ external services
- Per-deploy values (canonical hostname, etc.)
- Feature flags
- Third-party service API keys
"""

# ตัวอย่าง 12-Factor Config
import os

class TwelveFactorConfig:
    """Configuration ตามหลัก 12-Factor App"""
    
    # Factor III: Config in environment
    DATABASE_URL = os.environ.get("DATABASE_URL")
    REDIS_URL = os.environ.get("REDIS_URL", "redis://localhost:6379")
    SECRET_KEY = os.environ.get("SECRET_KEY")
    
    # Factor X: Dev/prod parity - ใช้ service เดียวกัน
    LOG_LEVEL = os.environ.get("LOG_LEVEL", "INFO")
    
    # Factor XI: Logs - treat as event streams
    LOG_FORMAT = os.environ.get("LOG_FORMAT", "json")  # json หรือ text
    
    @classmethod
    def validate(cls):
        """ตรวจสอบว่า required config ครบ"""
        required = ["DATABASE_URL", "SECRET_KEY"]
        missing = [key for key in required if not getattr(cls, key)]
        if missing:
            raise ValueError(
                f"Missing required environment variables: {', '.join(missing)}\n"
                "See .env.example for reference"
            )
        return True

# ตรวจสอบตอน startup
# TwelveFactorConfig.validate()
```

---

## 6. Settings Class Pattern

### Pattern ที่ดีที่สุดสำหรับ Production

```python
import os
from typing import Optional, List
from dotenv import load_dotenv

# โหลด environment ก่อน import settings
load_dotenv()


class BaseSettings:
    """Base settings class พร้อม validation helpers"""
    
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
    
    @staticmethod
    def _require(key: str) -> str:
        """ดึงค่า required environment variable"""
        value = os.environ.get(key)
        if not value:
            raise ValueError(
                f"Required environment variable '{key}' is not set.\n"
                f"Please add it to your .env file or environment."
            )
        return value
    
    @staticmethod
    def _optional(key: str, default: str = None) -> Optional[str]:
        """ดึงค่า optional environment variable"""
        return os.environ.get(key, default)
    
    @staticmethod
    def _bool(key: str, default: bool = False) -> bool:
        """ดึงค่า boolean environment variable"""
        value = os.environ.get(key)
        if value is None:
            return default
        return value.lower() in ("true", "1", "yes", "on")
    
    @staticmethod
    def _int(key: str, default: int = 0) -> int:
        """ดึงค่า integer environment variable"""
        value = os.environ.get(key)
        if value is None:
            return default
        try:
            return int(value)
        except ValueError:
            raise ValueError(f"Environment variable '{key}' must be an integer, got: '{value}'")
    
    @staticmethod
    def _list(key: str, default: List[str] = None, separator: str = ",") -> List[str]:
        """ดึงค่า list environment variable"""
        value = os.environ.get(key)
        if value is None:
            return default or []
        return [item.strip() for item in value.split(separator) if item.strip()]


class DatabaseSettings(BaseSettings):
    def __init__(self):
        self.url: str = self._require("DATABASE_URL")
        self.pool_size: int = self._int("DB_POOL_SIZE", 5)
        self.max_overflow: int = self._int("DB_MAX_OVERFLOW", 10)
        self.pool_timeout: int = self._int("DB_POOL_TIMEOUT", 30)
        self.echo_sql: bool = self._bool("DB_ECHO_SQL", False)


class CacheSettings(BaseSettings):
    def __init__(self):
        self.url: str = self._optional("REDIS_URL", "redis://localhost:6379/0")
        self.max_connections: int = self._int("CACHE_MAX_CONNECTIONS", 20)
        self.default_ttl: int = self._int("CACHE_DEFAULT_TTL", 300)  # seconds


class EmailSettings(BaseSettings):
    def __init__(self):
        self.host: str = self._optional("EMAIL_HOST", "localhost")
        self.port: int = self._int("EMAIL_PORT", 587)
        self.username: Optional[str] = self._optional("EMAIL_USER")
        self.password: Optional[str] = self._optional("EMAIL_PASSWORD")
        self.use_tls: bool = self._bool("EMAIL_USE_TLS", True)
        self.from_email: str = self._optional("DEFAULT_FROM_EMAIL", "noreply@example.com")


class Settings(BaseSettings):
    """Main application settings"""
    
    def __init__(self):
        # Core
        self.app_name: str = self._optional("APP_NAME", "MyApp")
        self.environment: str = self._optional("APP_ENV", "development")
        self.debug: bool = self._bool("DEBUG", False)
        self.secret_key: str = self._require("SECRET_KEY")
        
        # Server
        self.host: str = self._optional("HOST", "0.0.0.0")
        self.port: int = self._int("PORT", 8000)
        self.workers: int = self._int("WORKERS", 1)
        self.allowed_hosts: List[str] = self._list("ALLOWED_HOSTS", ["*"])
        self.cors_origins: List[str] = self._list("CORS_ORIGINS", [])
        
        # Sub-settings
        self.database = DatabaseSettings()
        self.cache = CacheSettings()
        self.email = EmailSettings()
    
    @property
    def is_development(self) -> bool:
        return self.environment == "development"
    
    @property
    def is_production(self) -> bool:
        return self.environment == "production"
    
    @property
    def is_testing(self) -> bool:
        return self.environment == "test"
    
    def __str__(self):
        return (
            f"Settings("
            f"app={self.app_name}, "
            f"env={self.environment}, "
            f"debug={self.debug}, "
            f"host={self.host}:{self.port}"
            f")"
        )


# สร้าง global settings instance
# settings = Settings()
# print(settings)
```

---

## 7. Pydantic Settings

**Pydantic Settings** คือ extension ของ Pydantic ที่ช่วยจัดการ configuration อย่าง type-safe พร้อม validation

### การติดตั้ง

```bash
pip install pydantic-settings
# หรือ
pip install pydantic[dotenv]  # สำหรับ pydantic v1
```

### ตัวอย่างที่ 10: Pydantic Settings พื้นฐาน

```python
from pydantic_settings import BaseSettings
from pydantic import Field, validator
from typing import Optional, List

class AppSettings(BaseSettings):
    """Application settings โดยใช้ Pydantic"""
    
    # ค่าจะถูกอ่านจาก environment variables อัตโนมัติ
    app_name: str = "MyApp"
    debug: bool = False
    secret_key: str  # Required! ไม่มี default = ต้องมีใน env
    
    # Database
    database_url: str = "sqlite:///./app.db"
    db_pool_size: int = Field(5, ge=1, le=100)  # min=1, max=100
    
    # Server
    host: str = "0.0.0.0"
    port: int = Field(8000, ge=1, le=65535)
    
    # Optional
    redis_url: Optional[str] = None
    allowed_hosts: List[str] = ["localhost", "127.0.0.1"]
    
    class Config:
        env_file = ".env"           # อ่านจาก .env file
        env_file_encoding = "utf-8"
        case_sensitive = False      # DATABASE_URL = database_url
        
# ใช้งาน
# settings = AppSettings()
# print(settings.app_name)
# print(settings.port)
```

### ตัวอย่างที่ 11: Pydantic Settings พร้อม Validators

```python
from pydantic_settings import BaseSettings
from pydantic import validator, root_validator, SecretStr
from typing import Optional, List, Set
import secrets

class SecureSettings(BaseSettings):
    """Settings พร้อม validation ครบถ้วน"""
    
    # ใช้ SecretStr สำหรับ sensitive values
    secret_key: SecretStr
    database_password: SecretStr
    api_key: Optional[SecretStr] = None
    
    # Database URL
    db_host: str = "localhost"
    db_port: int = 5432
    db_name: str = "myapp"
    db_user: str = "postgres"
    
    # Computed property
    database_url: Optional[str] = None
    
    # Application
    environment: str = "development"
    debug: bool = False
    allowed_hosts: List[str] = []
    
    @validator("environment")
    def validate_environment(cls, v):
        allowed = {"development", "staging", "production", "test"}
        if v not in allowed:
            raise ValueError(f"environment must be one of {allowed}")
        return v
    
    @validator("debug", always=True)
    def validate_debug_in_production(cls, v, values):
        if values.get("environment") == "production" and v:
            raise ValueError("debug must be False in production!")
        return v
    
    @validator("secret_key")
    def validate_secret_key_strength(cls, v):
        key = v.get_secret_value()
        if len(key) < 32:
            raise ValueError("secret_key must be at least 32 characters")
        return v
    
    @root_validator
    def set_database_url(cls, values):
        """สร้าง database_url จาก individual settings"""
        if not values.get("database_url"):
            host = values.get("db_host", "localhost")
            port = values.get("db_port", 5432)
            name = values.get("db_name", "myapp")
            user = values.get("db_user", "postgres")
            password = values.get("database_password")
            if password:
                pwd = password.get_secret_value()
                values["database_url"] = f"postgresql://{user}:{pwd}@{host}:{port}/{name}"
        return values
    
    def get_secret_key(self) -> str:
        """ดึงค่า secret key จริง"""
        return self.secret_key.get_secret_value()
    
    class Config:
        env_file = ".env"
        env_prefix = "APP_"  # ทุก env var ต้องขึ้นต้นด้วย APP_


# ตัวอย่าง:
# export APP_SECRET_KEY="my-super-secret-key-minimum-32-chars!!"
# export APP_DATABASE_PASSWORD="db-password"
# settings = SecureSettings()
```

### ตัวอย่างที่ 12: Pydantic Settings v2 (ใหม่กว่า)

```python
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, field_validator, model_validator, SecretStr
from typing import Optional, List, Annotated

class ModernSettings(BaseSettings):
    """Pydantic Settings v2 syntax"""
    
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
        env_prefix="",
        env_nested_delimiter="__",  # ใช้ DB__HOST สำหรับ nested fields
        extra="ignore",             # ไม่ error ถ้ามี env var เกิน
    )
    
    # Fields
    app_name: str = "MyApp"
    debug: bool = False
    secret_key: SecretStr = Field(..., min_length=32)
    port: Annotated[int, Field(ge=1, le=65535)] = 8000
    allowed_hosts: List[str] = ["localhost"]
    
    @field_validator("allowed_hosts", mode="before")
    @classmethod
    def parse_hosts(cls, v):
        """รับได้ทั้ง string และ list"""
        if isinstance(v, str):
            return [h.strip() for h in v.split(",")]
        return v
    
    @model_validator(mode="after")
    def check_production_settings(self) -> "ModernSettings":
        if not self.debug and self.secret_key.get_secret_value() == "development":
            raise ValueError("Please set a real SECRET_KEY for non-debug environments")
        return self


# ตัวอย่าง nested settings ด้วย env_nested_delimiter
class NestedSettings(BaseSettings):
    model_config = SettingsConfigDict(
        env_nested_delimiter="__"
    )
    
    class Database:
        host: str = "localhost"
        port: int = 5432
    
    # ตั้งค่าด้วย: DATABASE__HOST=myhost DATABASE__PORT=5432
    database: Database = Database()
```

---

## 8. Configuration Validation

### ตัวอย่างที่ 13: Manual Validation

```python
import os
import re
from typing import List, Tuple

class ConfigValidator:
    """ตรวจสอบความถูกต้องของ configuration"""
    
    def __init__(self):
        self.errors: List[str] = []
        self.warnings: List[str] = []
    
    def require(self, key: str) -> "ConfigValidator":
        """ตรวจสอบว่า required env var มีค่า"""
        if not os.environ.get(key):
            self.errors.append(f"Required: {key} is not set")
        return self
    
    def require_url(self, key: str) -> "ConfigValidator":
        """ตรวจสอบว่าเป็น URL ที่ valid"""
        value = os.environ.get(key)
        if not value:
            self.errors.append(f"Required: {key} is not set")
        elif not re.match(r"^https?://|^postgresql://|^redis://|^sqlite://", value):
            self.errors.append(f"Invalid URL format for {key}: {value}")
        return self
    
    def require_min_length(self, key: str, min_len: int) -> "ConfigValidator":
        """ตรวจสอบความยาวขั้นต่ำ"""
        value = os.environ.get(key, "")
        if len(value) < min_len:
            self.errors.append(
                f"{key} must be at least {min_len} characters "
                f"(got {len(value)})"
            )
        return self
    
    def warn_default(self, key: str, default_value: str) -> "ConfigValidator":
        """แจ้งเตือนถ้ายังใช้ค่า default"""
        value = os.environ.get(key)
        if value == default_value or not value:
            self.warnings.append(
                f"Warning: {key} is using default value '{default_value}'. "
                f"Change this in production!"
            )
        return self
    
    def validate(self) -> Tuple[bool, List[str], List[str]]:
        """คืนผลการ validate"""
        return len(self.errors) == 0, self.errors, self.warnings
    
    def raise_if_invalid(self):
        """Raise exception ถ้ามี errors"""
        is_valid, errors, warnings = self.validate()
        
        for warning in warnings:
            print(f"⚠️  {warning}")
        
        if not is_valid:
            error_msg = "\n".join(f"  - {e}" for e in errors)
            raise ValueError(
                f"Configuration validation failed:\n{error_msg}\n\n"
                f"Please check your .env file or environment variables."
            )
        
        print("✅ Configuration validation passed!")


# ใช้งาน
def validate_app_config():
    validator = ConfigValidator()
    
    (validator
        .require("SECRET_KEY")
        .require_min_length("SECRET_KEY", 32)
        .warn_default("SECRET_KEY", "dev-secret-key")
        .require_url("DATABASE_URL")
        .require("APP_NAME")
    )
    
    validator.raise_if_invalid()

# validate_app_config()
```

### ตัวอย่างที่ 14: Startup Configuration Check

```python
import os
import sys
from dotenv import load_dotenv

def check_required_config():
    """เช็ค configuration ตอน startup"""
    load_dotenv()
    
    required_vars = {
        "SECRET_KEY": "Application secret key (min 32 chars)",
        "DATABASE_URL": "Database connection string",
    }
    
    optional_but_recommended = {
        "REDIS_URL": "Redis for caching/sessions",
        "EMAIL_HOST": "Email server for notifications",
        "SENTRY_DSN": "Error tracking with Sentry",
    }
    
    missing_required = []
    missing_recommended = []
    
    # ตรวจสอบ required
    for var, description in required_vars.items():
        if not os.environ.get(var):
            missing_required.append(f"  {var}: {description}")
    
    # ตรวจสอบ recommended
    for var, description in optional_but_recommended.items():
        if not os.environ.get(var):
            missing_recommended.append(f"  {var}: {description}")
    
    # แจ้งเตือน
    if missing_recommended:
        print("ℹ️  Optional (recommended) variables not set:")
        for var in missing_recommended:
            print(var)
    
    # Error ถ้าขาด required
    if missing_required:
        print("\n❌ Required environment variables are missing:")
        for var in missing_required:
            print(var)
        print("\nPlease check your .env file. See .env.example for reference.")
        sys.exit(1)
    
    print("✅ All required configuration is present.")


# เรียกใช้ตอน app startup
# check_required_config()
```

---

## 9. Environment-Specific Configs

### ตัวอย่างที่ 15: Environment-Based Config Selection

```python
import os
from dotenv import load_dotenv

class DevelopmentConfig:
    """Config สำหรับ Development"""
    DEBUG = True
    TESTING = False
    DATABASE_URL = "sqlite:///./dev.db"
    CACHE_TYPE = "simple"  # in-memory cache
    LOG_LEVEL = "DEBUG"
    SEND_EMAILS = False     # ไม่ส่ง email จริง
    EMAIL_BACKEND = "console"  # แสดงใน console แทน


class StagingConfig:
    """Config สำหรับ Staging"""
    DEBUG = False
    TESTING = False
    DATABASE_URL = os.getenv("DATABASE_URL")
    CACHE_TYPE = "redis"
    LOG_LEVEL = "INFO"
    SEND_EMAILS = True
    EMAIL_BACKEND = "smtp"


class ProductionConfig:
    """Config สำหรับ Production"""
    DEBUG = False
    TESTING = False
    DATABASE_URL = os.getenv("DATABASE_URL")
    CACHE_TYPE = "redis"
    LOG_LEVEL = "WARNING"
    SEND_EMAILS = True
    EMAIL_BACKEND = "smtp"
    
    # Production-only security settings
    SECURE_COOKIES = True
    FORCE_HTTPS = True
    RATE_LIMITING = True


class TestingConfig:
    """Config สำหรับ Testing"""
    DEBUG = True
    TESTING = True
    DATABASE_URL = "sqlite:///:memory:"  # in-memory database
    CACHE_TYPE = "simple"
    LOG_LEVEL = "WARNING"
    SEND_EMAILS = False
    WTF_CSRF_ENABLED = False  # ปิด CSRF ตอน test


# Config registry
config_map = {
    "development": DevelopmentConfig,
    "staging": StagingConfig,
    "production": ProductionConfig,
    "test": TestingConfig,
}

def get_config():
    """ดึง config ตาม environment"""
    env = os.getenv("APP_ENV", "development").lower()
    config_class = config_map.get(env)
    
    if config_class is None:
        raise ValueError(
            f"Unknown environment: '{env}'. "
            f"Valid options: {list(config_map.keys())}"
        )
    
    return config_class()

config = get_config()
print(f"Using config: {type(config).__name__}")
```

### ตัวอย่างที่ 16: Layered Configuration

```python
import os
from typing import Any, Dict
from dotenv import load_dotenv, dotenv_values


class LayeredConfig:
    """
    Configuration ที่มีหลาย layers (ลำดับความสำคัญจากน้อยไปมาก):
    1. Defaults (lowest priority)
    2. .env file
    3. Environment-specific .env file
    4. System environment variables
    5. Runtime overrides (highest priority)
    """
    
    DEFAULTS: Dict[str, Any] = {
        "APP_NAME": "MyApp",
        "DEBUG": "false",
        "PORT": "8000",
        "HOST": "0.0.0.0",
        "LOG_LEVEL": "INFO",
        "DATABASE_URL": "sqlite:///./app.db",
    }
    
    def __init__(self):
        self._config: Dict[str, str] = {}
        self._load_all_layers()
    
    def _load_all_layers(self):
        # Layer 1: Defaults
        self._config.update(self.DEFAULTS)
        
        # Layer 2: Base .env file
        base_env = dotenv_values(".env")
        self._config.update(base_env)
        
        # Layer 3: Environment-specific .env
        app_env = os.getenv("APP_ENV", "development")
        env_specific = dotenv_values(f".env.{app_env}")
        self._config.update(env_specific)
        
        # Layer 4: Local overrides (never commit!)
        local_env = dotenv_values(".env.local")
        self._config.update(local_env)
        
        # Layer 5: System environment variables (highest priority)
        # ค่าจาก system env จะ override ทุกอย่าง
        for key in self._config:
            if key in os.environ:
                self._config[key] = os.environ[key]
    
    def get(self, key: str, default: Any = None) -> Any:
        return self._config.get(key, default)
    
    def __getitem__(self, key: str) -> str:
        if key not in self._config:
            raise KeyError(f"Configuration key '{key}' not found")
        return self._config[key]
    
    def __contains__(self, key: str) -> bool:
        return key in self._config


# config = LayeredConfig()
# print(config.get("PORT"))
```

---

## 10. Secrets Management Best Practices

### ตัวอย่างที่ 17: Secret Validation

```python
import os
import hashlib
import secrets
from typing import Optional


def generate_secret_key(length: int = 50) -> str:
    """สร้าง secret key ที่ random และ secure"""
    return secrets.token_hex(length)


def check_secret_strength(secret: str) -> dict:
    """ตรวจสอบความแข็งแกร่งของ secret"""
    return {
        "length": len(secret),
        "is_long_enough": len(secret) >= 32,
        "has_uppercase": any(c.isupper() for c in secret),
        "has_lowercase": any(c.islower() for c in secret),
        "has_digits": any(c.isdigit() for c in secret),
        "has_special": any(not c.isalnum() for c in secret),
        "is_not_common": secret not in [
            "secret", "password", "admin", "changeme",
            "dev-secret-key", "development"
        ]
    }


def mask_secret(value: str, visible_chars: int = 4) -> str:
    """ซ่อน secret ส่วนใหญ่ แสดงแค่ส่วนท้าย"""
    if len(value) <= visible_chars:
        return "*" * len(value)
    return "*" * (len(value) - visible_chars) + value[-visible_chars:]


# ตัวอย่าง
secret_key = generate_secret_key()
print(f"Generated: {secret_key}")
print(f"Masked: {mask_secret(secret_key)}")

strength = check_secret_strength(secret_key)
print(f"Strength check: {strength}")
```

### ตัวอย่างที่ 18: Secrets Manager Pattern

```python
import os
import json
import base64
from typing import Optional, Dict


class SecretsManager:
    """
    Abstract secrets manager - รองรับหลาย backend:
    - Environment variables
    - AWS Secrets Manager
    - HashiCorp Vault
    - Azure Key Vault
    """
    
    def get_secret(self, name: str) -> Optional[str]:
        raise NotImplementedError
    
    def get_all_secrets(self) -> Dict[str, str]:
        raise NotImplementedError


class EnvSecretsManager(SecretsManager):
    """ดึง secrets จาก environment variables (สำหรับ development)"""
    
    def get_secret(self, name: str) -> Optional[str]:
        return os.environ.get(name)
    
    def get_all_secrets(self) -> Dict[str, str]:
        return dict(os.environ)


class FileSecretsManager(SecretsManager):
    """ดึง secrets จาก encrypted file (สำหรับ simple cases)"""
    
    def __init__(self, secrets_file: str = "secrets.json"):
        self.secrets_file = secrets_file
        self._cache: Optional[Dict] = None
    
    def _load(self) -> Dict:
        if self._cache is None:
            with open(self.secrets_file) as f:
                self._cache = json.load(f)
        return self._cache
    
    def get_secret(self, name: str) -> Optional[str]:
        secrets = self._load()
        return secrets.get(name)
    
    def get_all_secrets(self) -> Dict[str, str]:
        return self._load().copy()


def get_secrets_manager() -> SecretsManager:
    """สร้าง secrets manager ตาม environment"""
    backend = os.getenv("SECRETS_BACKEND", "env")
    
    if backend == "env":
        return EnvSecretsManager()
    elif backend == "file":
        return FileSecretsManager()
    else:
        raise ValueError(f"Unknown secrets backend: {backend}")

# secrets = get_secrets_manager()
# api_key = secrets.get_secret("API_KEY")
```

### ตัวอย่างที่ 19: .gitignore สำหรับ Config Files

```bash
# .gitignore สำหรับ Python project

# Environment files - NEVER commit these!
.env
.env.local
.env.development.local
.env.test.local
.env.staging.local
.env.production
.env.*.local

# Secrets
secrets.json
secrets.yaml
*.key
*.pem
*.pfx
credentials.json

# Python
__pycache__/
*.py[cod]
*.pyo
dist/
build/
*.egg-info/
.venv/
venv/
env/

# ✅ DO commit these:
# .env.example       <- template ไม่มีค่าจริง
# .env.test          <- ถ้าไม่มี secrets จริง
```

### ตัวอย่างที่ 20: Complete Config Module

```python
"""
config.py - Complete configuration module
"""
import os
import logging
from functools import lru_cache
from typing import Optional, List
from dotenv import load_dotenv

# โหลด .env ก่อนทุกอย่าง
load_dotenv()

logger = logging.getLogger(__name__)


class Settings:
    """
    Application settings
    
    อ่านค่าจาก environment variables
    Validation ทำตอน init
    """
    
    def __init__(self):
        self._validate_and_load()
    
    def _validate_and_load(self):
        # === Required settings ===
        self.secret_key: str = self._require("SECRET_KEY", min_length=32)
        
        # === Application ===
        self.app_name: str = os.getenv("APP_NAME", "MyApp")
        self.app_version: str = os.getenv("APP_VERSION", "1.0.0")
        self.environment: str = self._validate_choice(
            "APP_ENV", 
            ["development", "staging", "production", "test"],
            "development"
        )
        self.debug: bool = self._get_bool("DEBUG", default=self.environment == "development")
        
        # === Server ===
        self.host: str = os.getenv("HOST", "0.0.0.0")
        self.port: int = self._get_int("PORT", 8000, min_val=1, max_val=65535)
        self.workers: int = self._get_int("WORKERS", 1, min_val=1)
        self.allowed_hosts: List[str] = self._get_list("ALLOWED_HOSTS", ["*"])
        
        # === Database ===
        self.database_url: str = os.getenv("DATABASE_URL", "sqlite:///./app.db")
        self.db_pool_size: int = self._get_int("DB_POOL_SIZE", 5, min_val=1)
        
        # === Cache ===
        self.redis_url: Optional[str] = os.getenv("REDIS_URL")
        
        # === Email ===
        self.email_host: str = os.getenv("EMAIL_HOST", "localhost")
        self.email_port: int = self._get_int("EMAIL_PORT", 587)
        self.email_user: Optional[str] = os.getenv("EMAIL_USER")
        self.email_password: Optional[str] = os.getenv("EMAIL_PASSWORD")
        
        # === Logging ===
        self.log_level: str = self._validate_choice(
            "LOG_LEVEL",
            ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"],
            "INFO"
        )
        
        # Production safety checks
        if self.is_production:
            self._production_checks()
    
    def _require(self, key: str, min_length: int = 0) -> str:
        value = os.environ.get(key)
        if not value:
            raise ValueError(f"Required environment variable '{key}' is not set")
        if len(value) < min_length:
            raise ValueError(f"'{key}' must be at least {min_length} characters")
        return value
    
    def _get_bool(self, key: str, default: bool = False) -> bool:
        value = os.environ.get(key)
        if value is None:
            return default
        return value.lower() in ("true", "1", "yes", "on")
    
    def _get_int(self, key: str, default: int = 0, 
                 min_val: int = None, max_val: int = None) -> int:
        value = os.environ.get(key)
        if value is None:
            return default
        try:
            int_val = int(value)
        except ValueError:
            raise ValueError(f"'{key}' must be an integer, got: '{value}'")
        if min_val is not None and int_val < min_val:
            raise ValueError(f"'{key}' must be >= {min_val}")
        if max_val is not None and int_val > max_val:
            raise ValueError(f"'{key}' must be <= {max_val}")
        return int_val
    
    def _get_list(self, key: str, default: List[str] = None) -> List[str]:
        value = os.environ.get(key)
        if not value:
            return default or []
        return [item.strip() for item in value.split(",") if item.strip()]
    
    def _validate_choice(self, key: str, choices: List[str], default: str) -> str:
        value = os.environ.get(key, default).lower()
        if value not in choices:
            raise ValueError(f"'{key}' must be one of {choices}, got: '{value}'")
        return value
    
    def _production_checks(self):
        """ตรวจสอบ production-specific requirements"""
        checks = [
            (self.debug == False, "DEBUG must be False in production"),
            (self.secret_key != "dev-secret-key", "Change SECRET_KEY in production"),
            ("*" not in self.allowed_hosts, "Do not use '*' for ALLOWED_HOSTS in production"),
        ]
        
        failures = [msg for check, msg in checks if not check]
        if failures:
            raise ValueError(
                "Production configuration errors:\n" + 
                "\n".join(f"  - {msg}" for msg in failures)
            )
    
    @property
    def is_development(self) -> bool:
        return self.environment == "development"
    
    @property
    def is_production(self) -> bool:
        return self.environment == "production"
    
    @property
    def is_testing(self) -> bool:
        return self.environment == "test"
    
    def __repr__(self) -> str:
        return (
            f"Settings("
            f"app={self.app_name!r}, "
            f"env={self.environment!r}, "
            f"debug={self.debug}, "
            f"port={self.port}"
            f")"
        )


@lru_cache(maxsize=1)
def get_settings() -> Settings:
    """
    Get cached settings instance
    
    ใช้ lru_cache เพื่อ return instance เดิมทุกครั้ง
    (dependency injection pattern สำหรับ FastAPI)
    """
    return Settings()


# สร้าง module-level instance สำหรับ convenience
# settings = get_settings()
```

### ตัวอย่างที่ 21: Configuration สำหรับ FastAPI

```python
"""
การใช้ Pydantic Settings กับ FastAPI
"""
from fastapi import FastAPI, Depends
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    app_name: str = "My API"
    debug: bool = False
    database_url: str = "sqlite:///./sql_app.db"
    secret_key: str = "change-this-in-production"
    
    class Config:
        env_file = ".env"


# ใช้ lru_cache เพื่อ cache settings
from functools import lru_cache

@lru_cache()
def get_settings():
    return Settings()


app = FastAPI()

@app.get("/info")
async def app_info(settings: Settings = Depends(get_settings)):
    return {
        "app_name": settings.app_name,
        "debug": settings.debug,
    }

# ตอน test สามารถ override ได้:
# app.dependency_overrides[get_settings] = lambda: Settings(debug=True)
```

### ตัวอย่างที่ 22: Secrets Rotation

```python
"""
Pattern สำหรับ rotating secrets โดยไม่ต้อง downtime
"""
import os
from datetime import datetime


class RotatingSecretManager:
    """
    จัดการ secrets ที่มีการ rotate
    รองรับ old secret ชั่วคราวระหว่าง rotation
    """
    
    def __init__(self):
        self.current_key = os.getenv("SECRET_KEY")
        # old key ยังใช้งานได้ระหว่าง rotation
        self.previous_key = os.getenv("SECRET_KEY_PREVIOUS")
        self.rotation_date = os.getenv("SECRET_KEY_ROTATION_DATE")
    
    def get_active_keys(self) -> list:
        """คืน list ของ keys ที่ยังใช้งานได้"""
        keys = [self.current_key]
        if self.previous_key:
            keys.append(self.previous_key)
        return [k for k in keys if k]
    
    def is_rotation_needed(self, max_age_days: int = 90) -> bool:
        """ตรวจสอบว่าถึงเวลา rotate หรือยัง"""
        if not self.rotation_date:
            return True
        
        rotation_datetime = datetime.fromisoformat(self.rotation_date)
        age = (datetime.now() - rotation_datetime).days
        return age >= max_age_days
    
    def validate_token(self, token: str, key: str) -> bool:
        """Validate token กับ keys ที่มีทั้งหมด"""
        for active_key in self.get_active_keys():
            # ลอง validate กับแต่ละ key
            try:
                # ในกรณีจริงใช้ JWT หรือ HMAC validation
                if self._validate_with_key(token, active_key):
                    return True
            except Exception:
                continue
        return False
    
    def _validate_with_key(self, token: str, key: str) -> bool:
        """placeholder สำหรับ real validation logic"""
        import hashlib
        expected = hashlib.sha256(f"{token}{key}".encode()).hexdigest()
        return True  # simplified

# manager = RotatingSecretManager()
# if manager.is_rotation_needed():
#     print("⚠️  Time to rotate your secrets!")
```

### ตัวอย่างที่ 23: Config Logging

```python
"""
Log configuration summary เมื่อ startup (โดยไม่เปิดเผย secrets)
"""
import os
import logging

logger = logging.getLogger(__name__)


def log_config_summary(settings) -> None:
    """Log configuration summary ตอน startup"""
    
    sensitive_keys = frozenset([
        "secret_key", "password", "api_key", "token",
        "private_key", "auth", "credential"
    ])
    
    def is_sensitive(key: str) -> bool:
        key_lower = key.lower()
        return any(sensitive in key_lower for sensitive in sensitive_keys)
    
    def format_value(key: str, value) -> str:
        if is_sensitive(key):
            return "***" if value else "(not set)"
        if value is None:
            return "(not set)"
        if isinstance(value, bool):
            return str(value)
        if isinstance(value, list):
            return f"[{', '.join(str(v) for v in value)}]"
        # Truncate long values
        str_val = str(value)
        return str_val[:50] + "..." if len(str_val) > 50 else str_val
    
    logger.info("=" * 50)
    logger.info("Application Configuration Summary")
    logger.info("=" * 50)
    
    for key, value in vars(settings).items():
        if not key.startswith("_"):
            formatted = format_value(key, value)
            logger.info(f"  {key}: {formatted}")
    
    logger.info("=" * 50)


# ตัวอย่าง output:
# Application Configuration Summary
# ==================================================
#   app_name: MyApp
#   environment: development
#   debug: True
#   secret_key: ***
#   database_url: sqlite:///./app.db
#   port: 8000
# ==================================================
```

### ตัวอย่างที่ 24: Environment Variable Documentation Generator

```python
"""
Auto-generate documentation สำหรับ environment variables
"""
import os
from dataclasses import dataclass, field
from typing import Any, Optional, List


@dataclass
class EnvVar:
    """คำอธิบาย environment variable"""
    name: str
    description: str
    required: bool = False
    default: Optional[str] = None
    example: Optional[str] = None
    valid_values: Optional[List[str]] = None
    sensitive: bool = False


# Define environment variables
ENV_VARS = [
    EnvVar(
        name="APP_NAME",
        description="Name of the application",
        default="MyApp",
        example="MyAwesomeApp"
    ),
    EnvVar(
        name="APP_ENV",
        description="Application environment",
        default="development",
        valid_values=["development", "staging", "production", "test"]
    ),
    EnvVar(
        name="SECRET_KEY",
        description="Secret key for cryptographic operations",
        required=True,
        sensitive=True,
        example="run: python -c \"import secrets; print(secrets.token_hex(50))\""
    ),
    EnvVar(
        name="DATABASE_URL",
        description="Database connection URL",
        required=True,
        example="postgresql://user:password@localhost:5432/mydb"
    ),
    EnvVar(
        name="DEBUG",
        description="Enable debug mode",
        default="false",
        valid_values=["true", "false"]
    ),
    EnvVar(
        name="PORT",
        description="Port to listen on",
        default="8000",
        example="8080"
    ),
]


def generate_env_example() -> str:
    """สร้าง .env.example จาก definitions"""
    lines = [
        "# .env.example",
        "# Copy this file to .env and fill in the values",
        "# Command: cp .env.example .env",
        "",
    ]
    
    for var in ENV_VARS:
        # Comment
        lines.append(f"# {var.description}")
        if var.required:
            lines.append("# REQUIRED")
        if var.valid_values:
            lines.append(f"# Valid values: {', '.join(var.valid_values)}")
        if var.example:
            lines.append(f"# Example: {var.example}")
        if var.sensitive:
            lines.append("# ⚠️  SENSITIVE - Never commit real values!")
        
        # Value
        if var.default:
            lines.append(f"{var.name}={var.default}")
        else:
            lines.append(f"{var.name}=")
        lines.append("")
    
    return "\n".join(lines)


def generate_docs_table() -> str:
    """สร้าง Markdown table สำหรับ docs"""
    lines = [
        "| Variable | Required | Default | Description |",
        "|----------|----------|---------|-------------|",
    ]
    
    for var in ENV_VARS:
        required = "✅ Yes" if var.required else "No"
        default = f"`{var.default}`" if var.default else "-"
        lines.append(f"| `{var.name}` | {required} | {default} | {var.description} |")
    
    return "\n".join(lines)


# print(generate_env_example())
# print(generate_docs_table())
```

### ตัวอย่างที่ 25: Config Testing

```python
"""
ทดสอบ configuration ด้วย pytest
"""
import os
import pytest
from unittest.mock import patch


# สมมติว่ามี Settings class
class Settings:
    def __init__(self):
        from dotenv import load_dotenv
        load_dotenv()
        self.debug = os.getenv("DEBUG", "false").lower() == "true"
        self.port = int(os.getenv("PORT", "8000"))
        self.secret_key = os.environ["SECRET_KEY"]
        self.database_url = os.getenv("DATABASE_URL", "sqlite:///./app.db")


class TestSettings:
    """Tests สำหรับ Settings"""
    
    def test_default_values(self):
        """ทดสอบ default values"""
        with patch.dict(os.environ, {
            "SECRET_KEY": "test-secret-key-minimum-32-chars-long"
        }, clear=True):
            settings = Settings()
            assert settings.debug == False
            assert settings.port == 8000
    
    def test_override_with_env_var(self):
        """ทดสอบการ override ด้วย env var"""
        with patch.dict(os.environ, {
            "SECRET_KEY": "test-secret-key-minimum-32-chars-long",
            "DEBUG": "true",
            "PORT": "9000",
        }):
            settings = Settings()
            assert settings.debug == True
            assert settings.port == 9000
    
    def test_missing_required_raises_error(self):
        """ทดสอบว่า raise error เมื่อขาด required var"""
        with patch.dict(os.environ, {}, clear=True):
            with pytest.raises(KeyError):
                settings = Settings()
    
    def test_invalid_port_raises_error(self):
        """ทดสอบ port ที่ไม่ valid"""
        with patch.dict(os.environ, {
            "SECRET_KEY": "test-secret-key-minimum-32-chars-long",
            "PORT": "not-a-number",
        }):
            with pytest.raises(ValueError):
                settings = Settings()


# =====================
# Fixtures สำหรับ pytest
# =====================

@pytest.fixture
def test_settings():
    """สร้าง test settings"""
    env = {
        "SECRET_KEY": "test-secret-key-minimum-32-chars-long!!",
        "DATABASE_URL": "sqlite:///:memory:",
        "APP_ENV": "test",
        "DEBUG": "false",
    }
    with patch.dict(os.environ, env):
        yield Settings()


def test_with_fixture(test_settings):
    """ใช้ fixture"""
    assert test_settings.database_url == "sqlite:///:memory:"
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Basic Environment Variables Reader
สร้างฟังก์ชัน `read_config()` ที่อ่าน environment variables ต่อไปนี้และ return เป็น dict พร้อม type conversion:
- `APP_NAME` (string, default: "MyApp")
- `DEBUG` (boolean, default: False)
- `PORT` (int, default: 8000)
- `MAX_CONNECTIONS` (int, default: 100)
- `ALLOWED_HOSTS` (list of strings, default: ["localhost"])

```python
# เฉลย
import os

def read_config() -> dict:
    """อ่าน configuration จาก environment variables"""
    
    def get_bool(key: str, default: bool = False) -> bool:
        value = os.environ.get(key, str(default)).lower()
        return value in ("true", "1", "yes")
    
    def get_int(key: str, default: int) -> int:
        try:
            return int(os.environ.get(key, str(default)))
        except ValueError:
            return default
    
    def get_list(key: str, default: list) -> list:
        value = os.environ.get(key)
        if not value:
            return default
        return [item.strip() for item in value.split(",") if item.strip()]
    
    return {
        "APP_NAME": os.environ.get("APP_NAME", "MyApp"),
        "DEBUG": get_bool("DEBUG", False),
        "PORT": get_int("PORT", 8000),
        "MAX_CONNECTIONS": get_int("MAX_CONNECTIONS", 100),
        "ALLOWED_HOSTS": get_list("ALLOWED_HOSTS", ["localhost"]),
    }

# ทดสอบ
import os
os.environ["DEBUG"] = "true"
os.environ["PORT"] = "9000"
os.environ["ALLOWED_HOSTS"] = "localhost,example.com,api.example.com"

config = read_config()
print(config)
# {'APP_NAME': 'MyApp', 'DEBUG': True, 'PORT': 9000, 
#  'MAX_CONNECTIONS': 100, 'ALLOWED_HOSTS': ['localhost', 'example.com', 'api.example.com']}
```

### แบบฝึกหัดที่ 2: .env File Generator
สร้างฟังก์ชันที่รับ dict ของ config และสร้างเนื้อหา .env file

```python
# เฉลย
from typing import Dict, Any, Optional

def generate_env_file(
    config: Dict[str, Any],
    comments: Optional[Dict[str, str]] = None
) -> str:
    """สร้างเนื้อหา .env file จาก config dict"""
    lines = ["# Generated configuration file", ""]
    comments = comments or {}
    
    for key, value in config.items():
        # เพิ่ม comment ถ้ามี
        if key in comments:
            lines.append(f"# {comments[key]}")
        
        # Convert value เป็น string
        if isinstance(value, bool):
            str_value = "true" if value else "false"
        elif isinstance(value, list):
            str_value = ",".join(str(v) for v in value)
        else:
            str_value = str(value)
        
        # Quote ถ้ามี spaces
        if " " in str_value:
            str_value = f'"{str_value}"'
        
        lines.append(f"{key}={str_value}")
    
    return "\n".join(lines)


# ทดสอบ
config = {
    "APP_NAME": "My Awesome App",
    "DEBUG": True,
    "PORT": 8000,
    "ALLOWED_HOSTS": ["localhost", "example.com"],
    "SECRET_KEY": "my-secret-key",
}

comments = {
    "APP_NAME": "Name of the application",
    "SECRET_KEY": "SENSITIVE - Change in production!",
}

print(generate_env_file(config, comments))
```

### แบบฝึกหัดที่ 3: Settings Validator
สร้าง class `ConfigValidator` ที่:
- รับ dict ของ validation rules
- ตรวจสอบ environment variables ตาม rules
- Return report ของ errors และ warnings

```python
# เฉลย
import os
import re
from dataclasses import dataclass, field
from typing import List, Dict, Any, Callable, Optional

@dataclass
class ValidationRule:
    key: str
    required: bool = False
    type_: str = "string"  # string, int, bool, url, email
    min_value: Optional[float] = None
    max_value: Optional[float] = None
    min_length: Optional[int] = None
    choices: Optional[List[str]] = None
    default: Optional[str] = None


@dataclass
class ValidationResult:
    is_valid: bool
    errors: List[str] = field(default_factory=list)
    warnings: List[str] = field(default_factory=list)
    
    def __str__(self):
        lines = []
        if self.errors:
            lines.append("Errors:")
            lines.extend(f"  ❌ {e}" for e in self.errors)
        if self.warnings:
            lines.append("Warnings:")
            lines.extend(f"  ⚠️  {w}" for w in self.warnings)
        if self.is_valid and not self.warnings:
            lines.append("✅ All configuration is valid!")
        return "\n".join(lines)


class ConfigValidator:
    def __init__(self, rules: List[ValidationRule]):
        self.rules = rules
    
    def validate(self) -> ValidationResult:
        errors = []
        warnings = []
        
        for rule in self.rules:
            value = os.environ.get(rule.key)
            
            # Check required
            if rule.required and not value:
                errors.append(f"Required variable '{rule.key}' is not set")
                continue
            
            # Use default if not set
            if not value:
                if rule.default:
                    warnings.append(f"'{rule.key}' using default value: {rule.default}")
                continue
            
            # Type validation
            if rule.type_ == "int":
                try:
                    int_val = int(value)
                    if rule.min_value is not None and int_val < rule.min_value:
                        errors.append(f"'{rule.key}' must be >= {rule.min_value}")
                    if rule.max_value is not None and int_val > rule.max_value:
                        errors.append(f"'{rule.key}' must be <= {rule.max_value}")
                except ValueError:
                    errors.append(f"'{rule.key}' must be an integer, got: '{value}'")
            
            elif rule.type_ == "url":
                if not re.match(r"^https?://|^postgresql://|^redis://", value):
                    errors.append(f"'{rule.key}' is not a valid URL: '{value}'")
            
            # Length validation
            if rule.min_length and len(value) < rule.min_length:
                errors.append(f"'{rule.key}' must be at least {rule.min_length} chars")
            
            # Choices validation
            if rule.choices and value.lower() not in rule.choices:
                errors.append(f"'{rule.key}' must be one of {rule.choices}, got: '{value}'")
        
        return ValidationResult(
            is_valid=len(errors) == 0,
            errors=errors,
            warnings=warnings
        )


# ทดสอบ
rules = [
    ValidationRule("APP_NAME", default="MyApp"),
    ValidationRule("SECRET_KEY", required=True, min_length=32),
    ValidationRule("PORT", type_="int", default="8000", min_value=1, max_value=65535),
    ValidationRule("APP_ENV", choices=["development", "staging", "production", "test"]),
    ValidationRule("DATABASE_URL", required=True, type_="url"),
]

os.environ["SECRET_KEY"] = "a" * 32
os.environ["DATABASE_URL"] = "postgresql://localhost/mydb"
os.environ["APP_ENV"] = "development"

validator = ConfigValidator(rules)
result = validator.validate()
print(result)
```

### แบบฝึกหัดที่ 4: Multi-Environment Config Loader
สร้างระบบที่โหลด config จากหลายไฟล์ตาม priority

```python
# เฉลย
import os
from pathlib import Path
from typing import Dict, Optional

class MultiEnvLoader:
    """
    โหลด configuration จากหลาย sources ตาม priority:
    1. Defaults
    2. .env
    3. .env.{environment}
    4. .env.local
    5. System environment (highest priority)
    """
    
    def __init__(self, base_dir: str = "."):
        self.base_dir = Path(base_dir)
        self._config: Dict[str, str] = {}
    
    def _load_dotenv_file(self, filepath: Path) -> Dict[str, str]:
        """อ่านไฟล์ .env และ return dict"""
        result = {}
        if not filepath.exists():
            return result
        
        with open(filepath) as f:
            for line in f:
                line = line.strip()
                # Skip comments and empty lines
                if not line or line.startswith("#"):
                    continue
                # Parse KEY=VALUE
                if "=" in line:
                    key, _, value = line.partition("=")
                    key = key.strip()
                    value = value.strip()
                    # Remove quotes
                    if len(value) >= 2 and value[0] == value[-1] in ('"', "'"):
                        value = value[1:-1]
                    result[key] = value
        
        return result
    
    def load(self, environment: Optional[str] = None) -> Dict[str, str]:
        """โหลด configuration ทั้งหมด"""
        env = environment or os.getenv("APP_ENV", "development")
        config: Dict[str, str] = {}
        
        # Load in order of priority (lowest to highest)
        files_to_load = [
            self.base_dir / ".env",
            self.base_dir / f".env.{env}",
            self.base_dir / ".env.local",
        ]
        
        for filepath in files_to_load:
            file_config = self._load_dotenv_file(filepath)
            if file_config:
                print(f"  Loaded: {filepath}")
                config.update(file_config)
        
        # System environment overrides everything
        for key in config:
            if key in os.environ:
                config[key] = os.environ[key]
        
        self._config = config
        return config
    
    def get(self, key: str, default: str = None) -> Optional[str]:
        return self._config.get(key, default)
    
    def __getitem__(self, key: str) -> str:
        return self._config[key]


# loader = MultiEnvLoader(".")
# config = loader.load("development")
# print(config)
```

### แบบฝึกหัดที่ 5: Pydantic Settings with Nested Models

```python
# เฉลย
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, validator, SecretStr
from typing import Optional, List

class DatabaseSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="DB_")
    
    host: str = "localhost"
    port: int = Field(5432, ge=1, le=65535)
    name: str = "myapp"
    user: str = "postgres"
    password: SecretStr = SecretStr("")
    pool_size: int = Field(5, ge=1, le=100)
    
    @property
    def url(self) -> str:
        pwd = self.password.get_secret_value()
        return f"postgresql://{self.user}:{pwd}@{self.host}:{self.port}/{self.name}"


class RedisSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="REDIS_")
    
    host: str = "localhost"
    port: int = 6379
    db: int = 0
    password: Optional[SecretStr] = None
    
    @property
    def url(self) -> str:
        if self.password:
            pwd = self.password.get_secret_value()
            return f"redis://:{pwd}@{self.host}:{self.port}/{self.db}"
        return f"redis://{self.host}:{self.port}/{self.db}"


class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        case_sensitive=False,
    )
    
    name: str = Field("MyApp", env="APP_NAME")
    env: str = Field("development", env="APP_ENV")
    debug: bool = Field(False, env="DEBUG")
    secret_key: SecretStr = Field(..., env="SECRET_KEY")
    port: int = Field(8000, env="PORT")
    
    # Nested settings
    database: DatabaseSettings = DatabaseSettings()
    redis: RedisSettings = RedisSettings()
    
    @property
    def is_production(self) -> bool:
        return self.env == "production"


# settings = AppSettings()
# print(settings.database.url)
# print(settings.redis.url)
```

### แบบฝึกหัดที่ 6: Config Encryption

```python
# เฉลย - การเข้ารหัส config อย่างง่าย
import os
import base64
import hashlib
from cryptography.fernet import Fernet


def generate_encryption_key() -> bytes:
    """สร้าง encryption key"""
    return Fernet.generate_key()


def encrypt_config_value(value: str, key: bytes) -> str:
    """เข้ารหัส config value"""
    f = Fernet(key)
    encrypted = f.encrypt(value.encode())
    return base64.urlsafe_b64encode(encrypted).decode()


def decrypt_config_value(encrypted_value: str, key: bytes) -> str:
    """ถอดรหัส config value"""
    f = Fernet(key)
    decoded = base64.urlsafe_b64decode(encrypted_value.encode())
    return f.decrypt(decoded).decode()


class EncryptedConfig:
    """Config manager ที่รองรับ encrypted values"""
    
    ENCRYPTED_PREFIX = "enc:"
    
    def __init__(self, encryption_key: bytes = None):
        self.encryption_key = encryption_key or self._get_key()
        self._fernet = Fernet(self.encryption_key)
    
    def _get_key(self) -> bytes:
        """ดึง key จาก environment"""
        key_env = os.environ.get("CONFIG_ENCRYPTION_KEY")
        if not key_env:
            raise ValueError("CONFIG_ENCRYPTION_KEY is not set")
        return key_env.encode()
    
    def get(self, key: str, default: str = None) -> str:
        """อ่าน config value และ decrypt ถ้าจำเป็น"""
        value = os.environ.get(key, default)
        if value and value.startswith(self.ENCRYPTED_PREFIX):
            encrypted_part = value[len(self.ENCRYPTED_PREFIX):]
            return self._fernet.decrypt(
                base64.urlsafe_b64decode(encrypted_part)
            ).decode()
        return value
    
    def encrypt_value(self, plaintext: str) -> str:
        """เข้ารหัส value สำหรับใส่ใน .env"""
        encrypted = self._fernet.encrypt(plaintext.encode())
        encoded = base64.urlsafe_b64encode(encrypted).decode()
        return f"{self.ENCRYPTED_PREFIX}{encoded}"


# Demo (ต้องติดตั้ง: pip install cryptography)
# key = generate_encryption_key()
# print(f"Key: {key.decode()}")

# encrypted = encrypt_config_value("my-secret-password", key)
# print(f"Encrypted: {encrypted}")

# decrypted = decrypt_config_value(encrypted, key)
# print(f"Decrypted: {decrypted}")
```

### แบบฝึกหัดที่ 7: Config Hot Reload

```python
# เฉลย - Config ที่ reload ได้โดยไม่ต้อง restart
import os
import time
import threading
from pathlib import Path
from typing import Callable, Dict, Any
from dotenv import dotenv_values


class HotReloadConfig:
    """Config ที่ detect การเปลี่ยนแปลงและ reload อัตโนมัติ"""
    
    def __init__(self, env_file: str = ".env", check_interval: float = 5.0):
        self.env_file = Path(env_file)
        self.check_interval = check_interval
        self._config: Dict[str, str] = {}
        self._last_modified: float = 0
        self._callbacks: list[Callable] = []
        self._lock = threading.Lock()
        self._running = False
        self._thread: threading.Thread = None
        
        # Initial load
        self._load()
    
    def _load(self):
        """โหลด config จากไฟล์"""
        if not self.env_file.exists():
            return
        
        with self._lock:
            self._config = dict(dotenv_values(str(self.env_file)))
            self._last_modified = self.env_file.stat().st_mtime
    
    def _check_and_reload(self):
        """ตรวจสอบว่าไฟล์เปลี่ยนหรือไม่"""
        if not self.env_file.exists():
            return
        
        current_mtime = self.env_file.stat().st_mtime
        if current_mtime > self._last_modified:
            print(f"Config file changed, reloading...")
            old_config = self._config.copy()
            self._load()
            
            # Notify callbacks
            for callback in self._callbacks:
                callback(old_config, self._config)
    
    def _watch_loop(self):
        """Loop ที่รันใน background thread"""
        while self._running:
            try:
                self._check_and_reload()
            except Exception as e:
                print(f"Error watching config: {e}")
            time.sleep(self.check_interval)
    
    def start_watching(self):
        """เริ่ม background thread ที่ watch ไฟล์"""
        if not self._running:
            self._running = True
            self._thread = threading.Thread(target=self._watch_loop, daemon=True)
            self._thread.start()
            print(f"Watching {self.env_file} for changes...")
    
    def stop_watching(self):
        """หยุด watching"""
        self._running = False
        if self._thread:
            self._thread.join(timeout=10)
    
    def on_change(self, callback: Callable[[Dict, Dict], None]):
        """Register callback เมื่อ config เปลี่ยน"""
        self._callbacks.append(callback)
        return callback
    
    def get(self, key: str, default: str = None) -> str:
        with self._lock:
            return self._config.get(key, default)
    
    def __getitem__(self, key: str) -> str:
        with self._lock:
            return self._config[key]


# ใช้งาน
# config = HotReloadConfig(".env", check_interval=2.0)

# @config.on_change
# def handle_config_change(old: dict, new: dict):
#     changed_keys = [k for k in new if new[k] != old.get(k)]
#     print(f"Config changed: {changed_keys}")

# config.start_watching()
```

### แบบฝึกหัดที่ 8: Complete Application Configuration System

```python
# เฉลย - ระบบ config สมบูรณ์พร้อมใช้งานจริง

"""
สร้างระบบ configuration สมบูรณ์ที่:
1. อ่านจาก .env file
2. Validate ทุก field
3. Support environment-specific config
4. Log config summary ตอน startup
5. มี type-safe access

รัน: python solution.py
"""

import os
import sys
import logging
from typing import Optional, List, Dict, Any
from pathlib import Path

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class ConfigurationError(Exception):
    """Raised when configuration is invalid"""
    pass


def _env(key: str, default: str = None, required: bool = False) -> Optional[str]:
    """Helper function อ่าน env var"""
    value = os.environ.get(key, default)
    if required and not value:
        raise ConfigurationError(
            f"Required environment variable '{key}' is not set.\n"
            f"Please add '{key}=<value>' to your .env file."
        )
    return value


def _bool(key: str, default: bool = False) -> bool:
    value = os.environ.get(key)
    if value is None:
        return default
    return value.lower() in ("true", "1", "yes", "on")


def _int(key: str, default: int, min_val: int = None, max_val: int = None) -> int:
    value = os.environ.get(key, str(default))
    try:
        result = int(value)
    except ValueError:
        raise ConfigurationError(f"'{key}' must be an integer, got: '{value}'")
    
    if min_val is not None and result < min_val:
        raise ConfigurationError(f"'{key}' must be >= {min_val}, got: {result}")
    if max_val is not None and result > max_val:
        raise ConfigurationError(f"'{key}' must be <= {max_val}, got: {result}")
    return result


def _list(key: str, default: List[str] = None) -> List[str]:
    value = os.environ.get(key)
    if not value:
        return default or []
    return [item.strip() for item in value.split(",") if item.strip()]


class AppConfig:
    """Complete application configuration"""
    
    def __init__(self):
        self._load()
    
    def _load(self):
        try:
            # Core
            self.app_name = _env("APP_NAME", "MyApp")
            self.version = _env("APP_VERSION", "1.0.0")
            self.environment = _env("APP_ENV", "development")
            self.debug = _bool("DEBUG", default=self.environment == "development")
            self.secret_key = _env("SECRET_KEY", required=True)
            
            # Validate secret key
            if self.secret_key and len(self.secret_key) < 16:
                raise ConfigurationError("SECRET_KEY must be at least 16 characters")
            
            # Server
            self.host = _env("HOST", "0.0.0.0")
            self.port = _int("PORT", 8000, min_val=1, max_val=65535)
            self.allowed_hosts = _list("ALLOWED_HOSTS", ["localhost"])
            
            # Database
            self.database_url = _env("DATABASE_URL", "sqlite:///./app.db")
            self.db_pool_size = _int("DB_POOL_SIZE", 5, min_val=1, max_val=100)
            
            # Cache
            self.redis_url = _env("REDIS_URL")
            
            # Logging
            self.log_level = _env("LOG_LEVEL", "INFO")
            
            # Validate environment
            valid_envs = {"development", "staging", "production", "test"}
            if self.environment not in valid_envs:
                raise ConfigurationError(
                    f"APP_ENV must be one of {valid_envs}, got: '{self.environment}'"
                )
            
            # Production checks
            if self.is_production:
                if self.debug:
                    raise ConfigurationError("DEBUG must be False in production")
                if self.secret_key in ("secret", "change-me", "dev"):
                    raise ConfigurationError("Use a real SECRET_KEY in production")
        
        except ConfigurationError:
            raise
        except Exception as e:
            raise ConfigurationError(f"Failed to load configuration: {e}") from e
    
    @property
    def is_development(self) -> bool:
        return self.environment == "development"
    
    @property
    def is_production(self) -> bool:
        return self.environment == "production"
    
    @property
    def is_testing(self) -> bool:
        return self.environment == "test"
    
    def summary(self) -> str:
        """สรุป config สำหรับ logging"""
        lines = [
            f"{'='*40}",
            f"App: {self.app_name} v{self.version}",
            f"Environment: {self.environment}",
            f"Debug: {self.debug}",
            f"Server: {self.host}:{self.port}",
            f"Database: {self.database_url[:30]}...",
            f"Redis: {'Yes' if self.redis_url else 'No'}",
            f"{'='*40}",
        ]
        return "\n".join(lines)


# Demo
if __name__ == "__main__":
    # Set minimal required env
    os.environ.setdefault("SECRET_KEY", "test-secret-key-for-demo-minimum-16")
    
    try:
        config = AppConfig()
        print(config.summary())
        print(f"\n✅ Configuration loaded successfully!")
    except ConfigurationError as e:
        print(f"❌ Configuration Error:\n{e}", file=sys.stderr)
        sys.exit(1)
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| `os.environ` | อ่าน/เขียน environment variables |
| `python-dotenv` | โหลด .env file |
| Config Patterns | dict, class, singleton |
| 12-Factor App | Store config in environment |
| Pydantic Settings | Type-safe config validation |
| Multi-environment | dev/staging/prod configs |
| Secrets Management | Best practices, rotation |

### Best Practices สรุป

1. **อย่า hardcode** secrets ใน code
2. **ใช้ .env.example** เป็น template สำหรับนักพัฒนาใหม่
3. **Add .env ใน .gitignore** เสมอ
4. **Validate config ตอน startup** ก่อนให้ application รัน
5. **ใช้ Pydantic Settings** สำหรับ type safety
6. **แยก config ตาม environment** dev/staging/prod
7. **Log config summary** ตอน startup (แต่ซ่อน secrets)
8. **Test config** ด้วย pytest และ `patch.dict(os.environ, ...)`

### การติดตั้ง Libraries ที่ใช้

```bash
pip install python-dotenv
pip install pydantic-settings
pip install cryptography  # สำหรับ encrypted config
```
