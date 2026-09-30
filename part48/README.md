# Part 48: Config Files - YAML, TOML & INI

## สารบัญ

1. [Configuration File Formats Overview](#1-configuration-file-formats-overview)
2. [INI Files - configparser](#2-ini-files---configparser)
3. [YAML Format](#3-yaml-format)
4. [TOML Format](#4-toml-format)
5. [JSON Config Files](#5-json-config-files)
6. [pyproject.toml](#6-pyprojecttoml)
7. [Configuration Hierarchies](#7-configuration-hierarchies)
8. [Config Validation with Pydantic](#8-config-validation-with-pydantic)
9. [Dynamic Config Loading](#9-dynamic-config-loading)
10. [Config Hot Reload](#10-config-hot-reload)
11. [แบบฝึกหัด](#11-แบบฝึกหัด)

---

## 1. Configuration File Formats Overview

### เปรียบเทียบ Format ต่างๆ

| Format | Extension | Human Readable | Comments | Complex Data | Python Support |
|--------|-----------|----------------|----------|--------------|----------------|
| INI | `.ini`, `.cfg` | ✅ Easy | ✅ Yes | ❌ Limited | Built-in |
| YAML | `.yaml`, `.yml` | ✅ Very Easy | ✅ Yes | ✅ Full | PyYAML |
| TOML | `.toml` | ✅ Easy | ✅ Yes | ✅ Good | Built-in (3.11+) |
| JSON | `.json` | ✅ Medium | ❌ No | ✅ Full | Built-in |
| XML | `.xml` | ❌ Verbose | ✅ Yes | ✅ Full | Built-in |

### เมื่อไรควรใช้อะไร

```
INI   → Simple settings, ไม่มี nested structure, backward compatibility
YAML  → Complex configs, Docker Compose, Kubernetes, Ansible
TOML  → Python project config (pyproject.toml), Cargo (Rust)
JSON  → API configs, data เป็นหลัก, ไม่ต้อง comments
```

### ตัวอย่างเนื้อหาเดียวกันใน 4 formats

**INI:**
```ini
[database]
host = localhost
port = 5432
name = myapp

[server]
host = 0.0.0.0
port = 8000
debug = true
```

**YAML:**
```yaml
database:
  host: localhost
  port: 5432
  name: myapp

server:
  host: "0.0.0.0"
  port: 8000
  debug: true
```

**TOML:**
```toml
[database]
host = "localhost"
port = 5432
name = "myapp"

[server]
host = "0.0.0.0"
port = 8000
debug = true
```

**JSON:**
```json
{
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "myapp"
  },
  "server": {
    "host": "0.0.0.0",
    "port": 8000,
    "debug": true
  }
}
```

---

## 2. INI Files - configparser

**configparser** เป็น standard library module สำหรับอ่าน/เขียน INI format

### โครงสร้าง INI File

```ini
# config.ini
# Comments start with # or ;
; This is also a comment

[DEFAULT]
# ค่าใน DEFAULT section จะถูก inherit ไปทุก section
debug = false
log_level = INFO

[database]
host = localhost
port = 5432
name = myapp
user = postgres
password = secret
# Override DEFAULT values
log_level = DEBUG

[server]
host = 0.0.0.0
port = 8000
workers = 4

[cache]
backend = redis
url = redis://localhost:6379/0
ttl = 300

[email]
smtp_host = smtp.gmail.com
smtp_port = 587
use_tls = true
```

### ตัวอย่างที่ 1: configparser พื้นฐาน

```python
import configparser

# สร้าง parser
config = configparser.ConfigParser()

# อ่านไฟล์
config.read("config.ini")

# อ่าน sections
print(config.sections())  # ['database', 'server', 'cache', 'email']

# อ่านค่า
db_host = config["database"]["host"]         # 'localhost'
db_port = config["database"].getint("port")  # 5432 (int)
debug = config["server"].getboolean("debug", fallback=False)

# อ่านค่าพร้อม fallback
timeout = config.get("server", "timeout", fallback="30")

# วนลูปดูทุก section
for section in config.sections():
    print(f"\n[{section}]")
    for key, value in config[section].items():
        print(f"  {key} = {value}")
```

### ตัวอย่างที่ 2: configparser อ่านค่าตาม type

```python
import configparser

config = configparser.ConfigParser()

# สร้าง config ใน memory
config["database"] = {
    "host": "localhost",
    "port": "5432",
    "pool_size": "10",
    "use_ssl": "true",
    "tables": "users,posts,comments",
}

# Type conversion methods
section = config["database"]

host = section.get("host")                    # str: 'localhost'
port = section.getint("port")                 # int: 5432
pool_size = section.getint("pool_size")       # int: 10
use_ssl = section.getboolean("use_ssl")       # bool: True
tables_str = section.get("tables")            # str: 'users,posts,comments'
tables = tables_str.split(",")               # list: ['users', 'posts', 'comments']

print(f"Host: {host} ({type(host).__name__})")
print(f"Port: {port} ({type(port).__name__})")
print(f"SSL: {use_ssl} ({type(use_ssl).__name__})")
print(f"Tables: {tables}")
```

### ตัวอย่างที่ 3: เขียน INI File

```python
import configparser

config = configparser.ConfigParser()

# เพิ่ม DEFAULT
config["DEFAULT"] = {
    "debug": "false",
    "log_level": "INFO",
}

# เพิ่ม sections
config["database"] = {
    "host": "localhost",
    "port": "5432",
    "name": "myapp",
    "user": "postgres",
    "password": "secret",
}

config["server"] = {
    "host": "0.0.0.0",
    "port": "8000",
    "workers": "4",
}

# เขียนลงไฟล์
with open("config.ini", "w") as configfile:
    config.write(configfile)

print("Config file created!")

# อ่านกลับมา
config2 = configparser.ConfigParser()
config2.read("config.ini")
print(config2["database"]["host"])  # localhost
```

### ตัวอย่างที่ 4: ConfigParser พร้อม Interpolation

```python
import configparser

# configparser รองรับ string interpolation
config_str = """
[DEFAULT]
base_dir = /app
log_dir = %(base_dir)s/logs

[database]
data_dir = %(base_dir)s/data
backup_dir = %(data_dir)s/backups

[server]
static_dir = %(base_dir)s/static
template_dir = %(base_dir)s/templates
"""

config = configparser.ConfigParser()
config.read_string(config_str)

print(config["database"]["data_dir"])    # /app/data
print(config["database"]["backup_dir"]) # /app/data/backups
print(config["server"]["static_dir"])   # /app/static

# ปิด interpolation ถ้าไม่ต้องการ
config_raw = configparser.RawConfigParser()
config_raw.read_string(config_str)
print(config_raw["database"]["data_dir"])  # %(base_dir)s/data (raw)
```

### ตัวอย่างที่ 5: Config Class บน INI

```python
import configparser
from pathlib import Path
from typing import List, Optional


class INIConfig:
    """Wrapper class สำหรับ INI config file"""
    
    def __init__(self, config_file: str = "config.ini"):
        self._config = configparser.ConfigParser()
        self._file = Path(config_file)
        
        if self._file.exists():
            self._config.read(self._file)
        else:
            self._create_default()
    
    def _create_default(self):
        """สร้าง default config"""
        self._config["DEFAULT"] = {
            "debug": "false",
        }
        self._config["app"] = {
            "name": "MyApp",
            "version": "1.0.0",
        }
        self._save()
    
    def _save(self):
        """บันทึกลงไฟล์"""
        with open(self._file, "w") as f:
            self._config.write(f)
    
    def get(self, section: str, key: str, fallback=None):
        return self._config.get(section, key, fallback=fallback)
    
    def get_int(self, section: str, key: str, fallback: int = 0) -> int:
        return self._config.getint(section, key, fallback=fallback)
    
    def get_bool(self, section: str, key: str, fallback: bool = False) -> bool:
        return self._config.getboolean(section, key, fallback=fallback)
    
    def set(self, section: str, key: str, value: str):
        if not self._config.has_section(section):
            self._config.add_section(section)
        self._config.set(section, key, str(value))
        self._save()
    
    def sections(self) -> List[str]:
        return self._config.sections()
    
    def to_dict(self) -> dict:
        result = {}
        for section in self._config.sections():
            result[section] = dict(self._config[section])
        return result


# ใช้งาน
# config = INIConfig("app.ini")
# config.set("database", "host", "localhost")
# print(config.get("database", "host"))
```

---

## 3. YAML Format

**YAML** (YAML Ain't Markup Language) เป็น format ที่อ่านง่ายที่สุด รองรับ data types หลากหลาย

### การติดตั้ง

```bash
pip install pyyaml
# หรือ
pip install ruamel.yaml  # version ที่ preserve comments
```

### โครงสร้าง YAML

```yaml
# config.yaml

# String values
app_name: "My Application"
version: "1.2.3"
description: >  # Folded scalar (newlines → spaces)
  This is a long description
  that spans multiple lines.

# Numbers
port: 8000
timeout: 30.5
max_connections: null  # None ใน Python

# Boolean
debug: true
ssl_enabled: false

# Lists
allowed_hosts:
  - localhost
  - 127.0.0.1
  - example.com

# หรือ inline list
tags: [python, backend, api]

# Nested objects
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: "${DB_PASSWORD}"  # จะ interpolate ถ้าใช้ library ที่รองรับ

# List of objects
services:
  - name: web
    image: python:3.11
    port: 8000
  - name: worker
    image: python:3.11
    command: celery worker

# Anchors & Aliases (YAML specific)
defaults: &defaults
  timeout: 30
  retries: 3

production:
  <<: *defaults  # merge
  timeout: 60    # override

# Multi-line strings
script: |
  #!/bin/bash
  echo "Hello"
  python app.py
```

### ตัวอย่างที่ 6: PyYAML พื้นฐาน

```python
import yaml

# อ่าน YAML file
with open("config.yaml") as f:
    config = yaml.safe_load(f)

print(config["app_name"])              # str
print(config["port"])                  # int
print(config["debug"])                 # bool
print(config["allowed_hosts"])         # list
print(config["database"]["host"])      # nested dict

# อ่าน YAML string
yaml_str = """
name: Alice
age: 30
hobbies:
  - reading
  - coding
address:
  city: Bangkok
  country: Thailand
"""

data = yaml.safe_load(yaml_str)
print(data["name"])              # Alice
print(data["hobbies"])           # ['reading', 'coding']
print(data["address"]["city"])   # Bangkok

# เขียน YAML
config_data = {
    "database": {
        "host": "localhost",
        "port": 5432,
    },
    "features": ["api", "admin", "worker"],
    "debug": True,
}

# เขียนลงไฟล์
with open("output.yaml", "w") as f:
    yaml.dump(config_data, f, default_flow_style=False, allow_unicode=True)

# เขียนเป็น string
yaml_output = yaml.dump(config_data, default_flow_style=False)
print(yaml_output)
```

### ตัวอย่างที่ 7: YAML Data Types

```python
import yaml

yaml_str = """
# Strings
name: "Hello World"
quoted: 'Single quotes'
unquoted: plain string
multiline: |
  Line 1
  Line 2
  Line 3
folded: >
  This is a
  folded string

# Numbers
integer: 42
negative: -10
float: 3.14
scientific: 1.5e10
hex: 0xFF
octal: 0o77
binary: 0b1010

# Booleans (YAML 1.1 vs 1.2)
true_val: true
false_val: false
yes_val: yes  # YAML 1.1 treats as True (safe_load ignores)
no_val: no    # YAML 1.1 treats as False

# Null
null_val: null
tilde_val: ~
empty_val:

# Dates (parsed as datetime.date)
date: 2024-01-15
datetime: 2024-01-15T10:30:00

# Lists
list1:
  - item1
  - item2
  - item3
list2: [a, b, c]

# Nested
nested:
  level1:
    level2:
      value: deep
"""

data = yaml.safe_load(yaml_str)

print(f"integer: {data['integer']} ({type(data['integer']).__name__})")
print(f"float: {data['float']} ({type(data['float']).__name__})")
print(f"true_val: {data['true_val']} ({type(data['true_val']).__name__})")
print(f"null_val: {data['null_val']} ({type(data['null_val']).__name__})")
print(f"date: {data['date']} ({type(data['date']).__name__})")
print(f"nested: {data['nested']['level1']['level2']['value']}")
```

### ตัวอย่างที่ 8: YAML Anchors และ Merge Keys

```python
import yaml

# YAML anchors ช่วยลด duplication
yaml_str = """
# Define anchors with &
default_config: &default
  timeout: 30
  retries: 3
  log_level: INFO

# Use alias with *
development:
  <<: *default  # merge all keys from anchor
  debug: true
  log_level: DEBUG  # override

staging:
  <<: *default
  workers: 2

production:
  <<: *default
  workers: 8
  timeout: 60   # override
  ssl: true
"""

config = yaml.safe_load(yaml_str)

print("Development:", config["development"])
# {'timeout': 30, 'retries': 3, 'log_level': 'DEBUG', 'debug': True}

print("Production:", config["production"])
# {'timeout': 60, 'retries': 3, 'log_level': 'INFO', 'workers': 8, 'ssl': True}
```

### ตัวอย่างที่ 9: ruamel.yaml (Preserve Comments)

```python
# ruamel.yaml preserves comments เมื่อ read/write
# pip install ruamel.yaml

from ruamel.yaml import YAML

yaml = YAML()
yaml.preserve_quotes = True

# อ่านไฟล์ที่มี comments
yaml_str = """
# Application Configuration
app:
  name: MyApp  # Application name
  version: "1.0.0"  # Semantic versioning

# Database settings
database:
  host: localhost  # Change in production
  port: 5432
"""

data = yaml.load(yaml_str)

# แก้ไขค่า
data["app"]["version"] = "1.1.0"
data["database"]["host"] = "db.example.com"

# เขียนกลับ - comments ยังอยู่!
import io
stream = io.StringIO()
yaml.dump(data, stream)
result = stream.getvalue()
print(result)
# ยังมี comments อยู่ครบ!
```

### ตัวอย่างที่ 10: YAML Config Loader ที่สมบูรณ์

```python
import yaml
import os
from pathlib import Path
from typing import Any, Optional


class YAMLConfigLoader:
    """โหลด YAML config พร้อม environment variable interpolation"""
    
    def __init__(self, config_file: str):
        self.config_file = Path(config_file)
        self._config = {}
        self._load()
    
    def _load(self):
        """โหลด config และ interpolate env vars"""
        if not self.config_file.exists():
            raise FileNotFoundError(f"Config file not found: {self.config_file}")
        
        with open(self.config_file) as f:
            content = f.read()
        
        # Interpolate environment variables: ${VAR_NAME} หรือ ${VAR_NAME:default}
        content = self._interpolate_env(content)
        
        self._config = yaml.safe_load(content) or {}
    
    def _interpolate_env(self, content: str) -> str:
        """แทนที่ ${VAR} และ ${VAR:default} ด้วยค่าจาก environment"""
        import re
        
        def replace_var(match):
            var_expr = match.group(1)
            if ":" in var_expr:
                var_name, default = var_expr.split(":", 1)
            else:
                var_name = var_expr
                default = None
            
            value = os.environ.get(var_name.strip(), default)
            if value is None:
                raise ValueError(
                    f"Required environment variable '{var_name}' not set in config"
                )
            return value
        
        return re.sub(r"\$\{([^}]+)\}", replace_var, content)
    
    def get(self, *keys: str, default: Any = None) -> Any:
        """อ่านค่าด้วย dot notation: get('database', 'host')"""
        current = self._config
        for key in keys:
            if not isinstance(current, dict):
                return default
            current = current.get(key)
            if current is None:
                return default
        return current
    
    def __getitem__(self, key: str) -> Any:
        return self._config[key]
    
    def to_dict(self) -> dict:
        return self._config.copy()


# ตัวอย่าง config.yaml:
# database:
#   host: ${DB_HOST:localhost}
#   password: ${DB_PASSWORD}  # required!

# loader = YAMLConfigLoader("config.yaml")
# host = loader.get("database", "host")
```

---

## 4. TOML Format

**TOML** (Tom's Obvious Minimal Language) เป็น format ที่ชัดเจน ไม่ ambiguous และเป็น standard สำหรับ Python projects

### Python 3.11+ built-in tomllib

```bash
# Python 3.11+ มี tomllib built-in (read-only)
# สำหรับ write และ Python < 3.11:
pip install tomli      # read-only, fast
pip install tomli-w    # write support
# หรือ
pip install toml       # read/write (older)
```

### โครงสร้าง TOML

```toml
# config.toml

# Strings
app_name = "My Application"
version = "1.2.3"
description = """
Multi-line
string here
"""

# Numbers
port = 8000
timeout = 30.5

# Booleans
debug = true
ssl_enabled = false

# Null ไม่มีใน TOML ใช้ workaround

# Arrays
allowed_hosts = ["localhost", "127.0.0.1", "example.com"]
tags = ["python", "backend", "api"]

# Tables (nested)
[database]
host = "localhost"
port = 5432
name = "myapp"

[database.credentials]
username = "admin"
password = "secret"

# Array of Tables
[[services]]
name = "web"
port = 8000

[[services]]
name = "worker"
command = "celery worker"

# Inline tables
server = {host = "0.0.0.0", port = 8080}

# Dates
created = 2024-01-15
updated = 2024-01-15T10:30:00Z
```

### ตัวอย่างที่ 11: TOML พื้นฐาน (Python 3.11+)

```python
# Python 3.11+ - built-in tomllib
import tomllib

# อ่านไฟล์ (binary mode!)
with open("config.toml", "rb") as f:  # ต้อง binary mode
    config = tomllib.load(f)

print(config["app_name"])
print(config["database"]["host"])
print(config["services"])  # list of dicts

# อ่าน string
toml_str = """
[app]
name = "MyApp"
version = "1.0.0"
debug = false

[database]
host = "localhost"
port = 5432
"""

config = tomllib.loads(toml_str)
print(config["app"]["name"])
print(config["database"]["port"])
```

### ตัวอย่างที่ 12: TOML Read/Write ด้วย tomli-w

```python
# pip install tomli tomli-w  (สำหรับ Python < 3.11)
# Python 3.11+: import tomllib แทน tomli

try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib  # fallback

import tomli_w  # สำหรับ write

# อ่าน TOML
with open("config.toml", "rb") as f:
    config = tomllib.load(f)

# แก้ไขค่า
config["app"]["version"] = "2.0.0"
config["database"]["host"] = "db.example.com"

# เพิ่ม key ใหม่
config["cache"] = {
    "backend": "redis",
    "url": "redis://localhost:6379/0",
}

# เขียนกลับ
with open("config.toml", "wb") as f:  # binary write mode
    tomli_w.dump(config, f)

# เขียนเป็น string
toml_str = tomli_w.dumps(config)
print(toml_str)
```

### ตัวอย่างที่ 13: TOML Type Handling

```python
try:
    import tomllib
except ImportError:
    import tomli as tomllib

from datetime import date, datetime

toml_str = """
# TOML handles types natively!
integer = 42
negative = -10
float_val = 3.14
large = 1_000_000  # underscores allowed

# Booleans (lowercase only!)
enabled = true
disabled = false

# Strings
plain = "hello"
multiline = """
line 1
line 2
"""
literal = 'no \\n escaping here'

# Dates/Times
date_only = 2024-01-15
time_only = 10:30:00
datetime_z = 2024-01-15T10:30:00Z
datetime_local = 2024-01-15T10:30:00

# Arrays (typed)
integers = [1, 2, 3]
strings = ["a", "b", "c"]
mixed = ["string", 42, true]  # ❌ Error! TOML arrays must be same type

# Inline tables
point = {x = 1, y = 2}
"""

config = tomllib.loads(toml_str)

print(f"integer: {config['integer']} ({type(config['integer']).__name__})")
print(f"float: {config['float_val']} ({type(config['float_val']).__name__})")
print(f"enabled: {config['enabled']} ({type(config['enabled']).__name__})")
print(f"date: {config['date_only']} ({type(config['date_only']).__name__})")
print(f"datetime: {config['datetime_z']} ({type(config['datetime_z']).__name__})")
```

### ตัวอย่างที่ 14: TOML Config Class

```python
try:
    import tomllib
except ImportError:
    import tomli as tomllib

from pathlib import Path
from typing import Any, Optional


class TOMLConfig:
    """TOML configuration manager"""
    
    def __init__(self, config_file: str = "config.toml"):
        self._file = Path(config_file)
        self._data = {}
        self.load()
    
    def load(self):
        """โหลด config จากไฟล์"""
        if self._file.exists():
            with open(self._file, "rb") as f:
                self._data = tomllib.load(f)
        else:
            self._data = {}
    
    def get(self, *path: str, default: Any = None) -> Any:
        """อ่านค่าด้วย path: get('database', 'host')"""
        current = self._data
        for key in path:
            if not isinstance(current, dict):
                return default
            current = current.get(key)
            if current is None:
                return default
        return current
    
    def get_section(self, section: str) -> dict:
        """อ่าน section ทั้งหมด"""
        return self._data.get(section, {})
    
    def __getitem__(self, key: str) -> Any:
        return self._data[key]
    
    def __contains__(self, key: str) -> bool:
        return key in self._data


# ใช้งาน
# config = TOMLConfig("app.toml")
# host = config.get("database", "host", default="localhost")
# db_config = config.get_section("database")
```

---

## 5. JSON Config Files

### ตัวอย่างที่ 15: JSON Config พื้นฐาน

```python
import json
from pathlib import Path
from typing import Any


def load_json_config(filepath: str) -> dict:
    """โหลด JSON config"""
    path = Path(filepath)
    if not path.exists():
        raise FileNotFoundError(f"Config not found: {filepath}")
    
    with open(path) as f:
        return json.load(f)


def save_json_config(config: dict, filepath: str, indent: int = 2):
    """บันทึก JSON config"""
    Path(filepath).parent.mkdir(parents=True, exist_ok=True)
    with open(filepath, "w") as f:
        json.dump(config, f, indent=indent, ensure_ascii=False)


# JSON กับ Comments (JSON5 หรือ JSONC)
# JSON มาตรฐานไม่รองรับ comments
# แต่ใช้ strip comments ก่อน parse ได้

def load_json_with_comments(filepath: str) -> dict:
    """โหลด JSON ที่มี comments (JSONC format)"""
    import re
    
    with open(filepath) as f:
        content = f.read()
    
    # Remove single-line comments
    content = re.sub(r"//.*?\n", "\n", content)
    # Remove multi-line comments
    content = re.sub(r"/\*.*?\*/", "", content, flags=re.DOTALL)
    
    return json.load(content)


# ตัวอย่าง .vscode/settings.json (JSONC format)
jsonc_example = '''
{
    // Editor settings
    "editor.fontSize": 14,
    "editor.tabSize": 4,
    
    /* Python settings */
    "python.defaultInterpreterPath": "./venv/bin/python",
    "python.linting.enabled": true
}
'''
```

### ตัวอย่างที่ 16: JSON Config พร้อม Schema Validation

```python
import json
import jsonschema  # pip install jsonschema

# กำหนด schema สำหรับ config
CONFIG_SCHEMA = {
    "type": "object",
    "required": ["app", "database"],
    "properties": {
        "app": {
            "type": "object",
            "required": ["name", "port"],
            "properties": {
                "name": {"type": "string"},
                "port": {"type": "integer", "minimum": 1, "maximum": 65535},
                "debug": {"type": "boolean"},
            }
        },
        "database": {
            "type": "object",
            "required": ["host", "name"],
            "properties": {
                "host": {"type": "string"},
                "port": {"type": "integer"},
                "name": {"type": "string"},
            }
        }
    }
}


def load_validated_config(filepath: str) -> dict:
    """โหลดและ validate JSON config"""
    with open(filepath) as f:
        config = json.load(f)
    
    try:
        jsonschema.validate(config, CONFIG_SCHEMA)
        print("✅ Config validation passed!")
    except jsonschema.ValidationError as e:
        raise ValueError(f"Config validation failed: {e.message}")
    
    return config


# ตัวอย่าง config.json ที่ valid
valid_config = {
    "app": {
        "name": "MyApp",
        "port": 8000,
        "debug": False,
    },
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "mydb",
    }
}
```

---

## 6. pyproject.toml

**pyproject.toml** คือ standard configuration file สำหรับ Python projects ตาม PEP 518, 621

### ตัวอย่างที่ 17: pyproject.toml สมบูรณ์

```toml
# pyproject.toml

[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "myapp"
version = "1.2.3"
description = "My awesome Python application"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "Jane Doe", email = "jane@example.com"},
]
maintainers = [
    {name = "John Doe", email = "john@example.com"},
]
keywords = ["python", "cli", "tool"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Environment :: Console",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]
requires-python = ">=3.9"

# Runtime dependencies
dependencies = [
    "click>=8.0",
    "rich>=13.0",
    "pydantic>=2.0",
    "pydantic-settings>=2.0",
    "python-dotenv>=1.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "black>=23.0",
    "ruff>=0.1.0",
    "mypy>=1.0",
]
docs = [
    "mkdocs>=1.5",
    "mkdocs-material>=9.0",
]

[project.urls]
Homepage = "https://github.com/example/myapp"
Documentation = "https://myapp.readthedocs.io"
Repository = "https://github.com/example/myapp.git"
Issues = "https://github.com/example/myapp/issues"
Changelog = "https://github.com/example/myapp/CHANGELOG.md"

[project.scripts]
myapp = "myapp.cli.main:cli"

[project.entry-points."myapp.plugins"]
example = "myapp.plugins.example:Plugin"


# === Tool configurations ===

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
addopts = [
    "--strict-markers",
    "--strict-config",
    "-ra",
]
markers = [
    "slow: marks tests as slow",
    "integration: marks integration tests",
]

[tool.coverage.run]
source = ["myapp"]
omit = ["tests/*", "**/__init__.py"]

[tool.coverage.report]
show_missing = true
fail_under = 80

[tool.black]
line-length = 88
target-version = ["py39", "py310", "py311", "py312"]
include = '\.pyi?$'
extend-exclude = '''
/(
    migrations
    | .venv
)/
'''

[tool.ruff]
line-length = 88
target-version = "py39"
select = ["E", "F", "W", "I", "N", "UP"]
ignore = ["E501"]

[tool.ruff.isort]
known-first-party = ["myapp"]

[tool.mypy]
python_version = "3.11"
strict = true
ignore_missing_imports = true
exclude = ["tests/", "docs/"]

[tool.setuptools.packages.find]
where = ["src"]  # หาก project ใช้ src layout
```

### ตัวอย่างที่ 18: อ่าน pyproject.toml ใน Python

```python
"""อ่านข้อมูลจาก pyproject.toml"""

try:
    import tomllib  # Python 3.11+
except ImportError:
    import tomli as tomllib

from pathlib import Path


def get_project_info() -> dict:
    """อ่าน project info จาก pyproject.toml"""
    pyproject_path = Path("pyproject.toml")
    
    if not pyproject_path.exists():
        return {}
    
    with open(pyproject_path, "rb") as f:
        data = tomllib.load(f)
    
    return data.get("project", {})


def get_version() -> str:
    """อ่าน version จาก pyproject.toml"""
    info = get_project_info()
    return info.get("version", "unknown")


def get_dependencies() -> list:
    """อ่าน dependencies จาก pyproject.toml"""
    info = get_project_info()
    return info.get("dependencies", [])


# ใช้งาน
# print(f"Version: {get_version()}")
# print(f"Dependencies: {get_dependencies()}")
```

---

## 7. Configuration Hierarchies

### ตัวอย่างที่ 19: Multi-level Config Hierarchy

```python
"""
Config loading hierarchy:
1. Built-in defaults (hardcoded)
2. System config (/etc/myapp/config.yaml)
3. User config (~/.config/myapp/config.yaml)
4. Project config (./config.yaml)
5. Environment variables
6. Command-line arguments (highest priority)
"""

import os
import yaml
from pathlib import Path
from typing import Any, Dict


def deep_merge(base: dict, override: dict) -> dict:
    """Merge two dicts recursively"""
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result


class HierarchicalConfig:
    """Config ที่ merge จากหลาย sources"""
    
    # Default values
    DEFAULTS = {
        "app": {
            "name": "MyApp",
            "debug": False,
            "log_level": "INFO",
        },
        "server": {
            "host": "0.0.0.0",
            "port": 8000,
            "workers": 1,
        },
        "database": {
            "host": "localhost",
            "port": 5432,
            "pool_size": 5,
        },
    }
    
    def __init__(self, app_name: str = "myapp"):
        self.app_name = app_name
        self._config = {}
        self._load_all()
    
    def _load_yaml(self, path: Path) -> dict:
        """โหลด YAML file"""
        if path.exists():
            with open(path) as f:
                return yaml.safe_load(f) or {}
        return {}
    
    def _load_all(self):
        """โหลด config ทุก layer"""
        config = self.DEFAULTS.copy()
        
        # System-wide config
        system_config = self._load_yaml(
            Path(f"/etc/{self.app_name}/config.yaml")
        )
        config = deep_merge(config, system_config)
        
        # User config
        user_config = self._load_yaml(
            Path.home() / ".config" / self.app_name / "config.yaml"
        )
        config = deep_merge(config, user_config)
        
        # Project config (search upward)
        project_config = self._find_and_load_project_config()
        config = deep_merge(config, project_config)
        
        # Environment variables override
        config = self._apply_env_vars(config)
        
        self._config = config
    
    def _find_and_load_project_config(self) -> dict:
        """หาและโหลด project config จาก current dir ขึ้นไป"""
        current = Path.cwd()
        for path in [current, *current.parents]:
            config_file = path / f"{self.app_name}.yaml"
            if config_file.exists():
                return self._load_yaml(config_file)
        return {}
    
    def _apply_env_vars(self, config: dict) -> dict:
        """Override config ด้วย environment variables"""
        prefix = self.app_name.upper() + "_"
        
        for key, value in os.environ.items():
            if key.startswith(prefix):
                # Convert MYAPP_DATABASE_HOST → config['database']['host']
                parts = key[len(prefix):].lower().split("_", 1)
                if len(parts) == 2:
                    section, option = parts
                    if section in config and isinstance(config[section], dict):
                        config[section][option] = value
                elif len(parts) == 1:
                    config[parts[0]] = value
        
        return config
    
    def get(self, *path: str, default: Any = None) -> Any:
        current = self._config
        for key in path:
            if not isinstance(current, dict):
                return default
            current = current.get(key, default)
            if current is default:
                return default
        return current
    
    def __getitem__(self, key: str) -> Any:
        return self._config[key]


# config = HierarchicalConfig("myapp")
# print(config.get("server", "port"))
```

---

## 8. Config Validation with Pydantic

### ตัวอย่างที่ 20: Pydantic + YAML/TOML Validation

```python
import yaml
from pydantic import BaseModel, Field, validator, root_validator
from typing import Optional, List
from pathlib import Path


class DatabaseConfig(BaseModel):
    host: str = "localhost"
    port: int = Field(5432, ge=1, le=65535)
    name: str
    user: str = "postgres"
    password: str = ""
    pool_size: int = Field(5, ge=1, le=100)
    ssl: bool = False
    
    @property
    def url(self) -> str:
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}/{self.name}"


class RedisConfig(BaseModel):
    host: str = "localhost"
    port: int = 6379
    db: int = 0
    password: Optional[str] = None
    
    @property
    def url(self) -> str:
        if self.password:
            return f"redis://:{self.password}@{self.host}:{self.port}/{self.db}"
        return f"redis://{self.host}:{self.port}/{self.db}"


class LoggingConfig(BaseModel):
    level: str = "INFO"
    file: Optional[str] = None
    format: str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    
    @validator("level")
    def validate_level(cls, v):
        allowed = {"DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"}
        if v.upper() not in allowed:
            raise ValueError(f"log level must be one of {allowed}")
        return v.upper()


class AppConfig(BaseModel):
    """Main application config - validated by Pydantic"""
    name: str
    version: str = "1.0.0"
    debug: bool = False
    host: str = "0.0.0.0"
    port: int = Field(8000, ge=1, le=65535)
    workers: int = Field(1, ge=1)
    allowed_hosts: List[str] = ["*"]
    secret_key: str
    
    database: DatabaseConfig
    redis: Optional[RedisConfig] = None
    logging: LoggingConfig = LoggingConfig()
    
    @validator("secret_key")
    def validate_secret_key(cls, v, values):
        if not values.get("debug", True) and len(v) < 32:
            raise ValueError("secret_key must be at least 32 chars in production")
        return v
    
    class Config:
        # อนุญาตให้มี extra fields ใน yaml
        extra = "ignore"


def load_config(config_file: str) -> AppConfig:
    """โหลด YAML config และ validate ด้วย Pydantic"""
    with open(config_file) as f:
        raw_config = yaml.safe_load(f)
    
    try:
        return AppConfig(**raw_config)
    except Exception as e:
        raise ValueError(f"Invalid configuration: {e}")


# config.yaml:
# name: MyApp
# debug: false
# secret_key: "my-secret-key-that-is-at-least-32-chars-long"
# database:
#   name: myapp
#   user: postgres
#   password: secret
#
# config = load_config("config.yaml")
# print(config.database.url)
```

---

## 9. Dynamic Config Loading

### ตัวอย่างที่ 21: Auto-detect Config Format

```python
import json
import os
from pathlib import Path
from typing import Any, Optional, Union

try:
    import tomllib
except ImportError:
    try:
        import tomli as tomllib
    except ImportError:
        tomllib = None

try:
    import yaml
except ImportError:
    yaml = None

import configparser


class UniversalConfigLoader:
    """โหลด config จากหลาย format โดย auto-detect"""
    
    SUPPORTED_FORMATS = {
        ".yaml": "yaml",
        ".yml": "yaml",
        ".toml": "toml",
        ".json": "json",
        ".ini": "ini",
        ".cfg": "ini",
    }
    
    def load(self, filepath: Union[str, Path]) -> dict:
        """โหลด config จาก file ใดก็ได้"""
        path = Path(filepath)
        
        if not path.exists():
            raise FileNotFoundError(f"Config file not found: {filepath}")
        
        format_ = self.SUPPORTED_FORMATS.get(path.suffix.lower())
        if not format_:
            raise ValueError(
                f"Unsupported format: {path.suffix}. "
                f"Supported: {list(self.SUPPORTED_FORMATS.keys())}"
            )
        
        loader = getattr(self, f"_load_{format_}")
        return loader(path)
    
    def _load_yaml(self, path: Path) -> dict:
        if yaml is None:
            raise ImportError("PyYAML not installed: pip install pyyaml")
        with open(path) as f:
            return yaml.safe_load(f) or {}
    
    def _load_toml(self, path: Path) -> dict:
        if tomllib is None:
            raise ImportError("tomllib/tomli not available: pip install tomli")
        with open(path, "rb") as f:
            return tomllib.load(f)
    
    def _load_json(self, path: Path) -> dict:
        with open(path) as f:
            return json.load(f)
    
    def _load_ini(self, path: Path) -> dict:
        config = configparser.ConfigParser()
        config.read(path)
        result = {}
        for section in config.sections():
            result[section] = dict(config[section])
        return result
    
    def find_and_load(self, name: str, search_dirs: list = None) -> Optional[dict]:
        """หา config file จาก หลาย directories และ formats"""
        search_dirs = search_dirs or [Path.cwd(), Path.home() / ".config"]
        
        for directory in search_dirs:
            for ext in self.SUPPORTED_FORMATS:
                filepath = Path(directory) / f"{name}{ext}"
                if filepath.exists():
                    print(f"Loading config from: {filepath}")
                    return self.load(filepath)
        
        return None


# ใช้งาน
loader = UniversalConfigLoader()

# โหลดไฟล์เฉพาะ
# config = loader.load("config.yaml")
# config = loader.load("pyproject.toml")

# ค้นหาอัตโนมัติ
# config = loader.find_and_load("myapp")  # จะหา myapp.yaml, myapp.toml, etc.
```

### ตัวอย่างที่ 22: Config with Environment Overrides

```python
import os
import yaml
from typing import Any

def load_config_with_env_override(config_file: str, env_prefix: str = "APP") -> dict:
    """
    โหลด YAML config และ override ด้วย environment variables
    
    Pattern: APP_DATABASE_HOST → config['database']['host']
    """
    
    def parse_env_vars(prefix: str) -> dict:
        """แปลง env vars เป็น nested dict"""
        result = {}
        prefix_upper = prefix.upper() + "_"
        
        for key, value in os.environ.items():
            if not key.startswith(prefix_upper):
                continue
            
            # Remove prefix และแปลงเป็น lowercase
            config_key = key[len(prefix_upper):].lower()
            
            # แปลง double underscore เป็น nested: DATABASE__HOST → database.host
            parts = config_key.split("__")
            
            # สร้าง nested dict
            current = result
            for part in parts[:-1]:
                if part not in current:
                    current[part] = {}
                current = current[part]
            
            # Type inference
            current[parts[-1]] = _infer_type(value)
        
        return result
    
    def _infer_type(value: str) -> Any:
        """Infer Python type จาก string"""
        if value.lower() in ("true", "yes"):
            return True
        if value.lower() in ("false", "no"):
            return False
        try:
            return int(value)
        except ValueError:
            pass
        try:
            return float(value)
        except ValueError:
            pass
        return value
    
    def deep_merge(base: dict, override: dict) -> dict:
        result = base.copy()
        for key, value in override.items():
            if key in result and isinstance(result[key], dict) and isinstance(value, dict):
                result[key] = deep_merge(result[key], value)
            else:
                result[key] = value
        return result
    
    # Load YAML
    with open(config_file) as f:
        base_config = yaml.safe_load(f) or {}
    
    # Parse env var overrides
    env_overrides = parse_env_vars(env_prefix)
    
    # Merge
    return deep_merge(base_config, env_overrides)


# ตัวอย่าง:
# config.yaml มี database.host = localhost
# APP_DATABASE_HOST = production.db.example.com
# ผลลัพธ์: database.host = production.db.example.com

# APP_DATABASE__HOST = production.db  (double underscore)
# ผลลัพธ์: database.host = production.db
```

---

## 10. Config Hot Reload

### ตัวอย่างที่ 23: Watchdog-based Config Reload

```python
"""
pip install watchdog
"""
import yaml
import time
import threading
from pathlib import Path
from typing import Callable


class ConfigWatcher:
    """Watch config file และ reload เมื่อเปลี่ยนแปลง"""
    
    def __init__(self, config_file: str):
        self._file = Path(config_file)
        self._config = {}
        self._callbacks = []
        self._last_mtime = 0
        self._running = False
        self._thread = None
        self._load()
    
    def _load(self):
        """โหลด config"""
        if self._file.exists():
            with open(self._file) as f:
                self._config = yaml.safe_load(f) or {}
            self._last_mtime = self._file.stat().st_mtime
    
    def _check(self):
        """ตรวจสอบว่าไฟล์เปลี่ยนหรือไม่"""
        if not self._file.exists():
            return
        
        current_mtime = self._file.stat().st_mtime
        if current_mtime > self._last_mtime:
            old_config = self._config.copy()
            self._load()
            
            # Notify all callbacks
            for callback in self._callbacks:
                try:
                    callback(old_config, self._config.copy())
                except Exception as e:
                    print(f"Config callback error: {e}")
    
    def watch(self, interval: float = 1.0):
        """เริ่ม watching ใน background thread"""
        if self._running:
            return
        
        self._running = True
        
        def loop():
            while self._running:
                self._check()
                time.sleep(interval)
        
        self._thread = threading.Thread(target=loop, daemon=True)
        self._thread.start()
        print(f"Watching {self._file} for changes...")
    
    def stop(self):
        """หยุด watching"""
        self._running = False
    
    def on_change(self, callback: Callable[[dict, dict], None]):
        """Register callback เมื่อ config เปลี่ยน"""
        self._callbacks.append(callback)
        return callback
    
    def get(self, *keys, default=None):
        current = self._config
        for key in keys:
            if not isinstance(current, dict):
                return default
            current = current.get(key, default)
        return current


# ใช้งาน
# watcher = ConfigWatcher("config.yaml")
# 
# @watcher.on_change
# def on_config_changed(old_config, new_config):
#     print(f"Config changed!")
#     # Reload services, connections, etc.
# 
# watcher.watch(interval=2.0)
```

### ตัวอย่างที่ 24: Complete Config System

```python
"""
complete_config.py - ระบบ config สมบูรณ์ที่รองรับหลาย formats
"""

import os
import json
import yaml
import configparser
from pathlib import Path
from typing import Any, Optional, Union, List, Dict
from functools import lru_cache

try:
    import tomllib
except ImportError:
    try:
        import tomli as tomllib
    except ImportError:
        tomllib = None


class ConfigError(Exception):
    pass


class Config:
    """
    Config manager ที่:
    1. รองรับหลาย formats (YAML, TOML, JSON, INI)
    2. รองรับ environment variable overrides
    3. รองรับ default values
    4. มี type-safe access methods
    """
    
    def __init__(
        self,
        config_file: Optional[str] = None,
        defaults: Dict[str, Any] = None,
        env_prefix: str = "",
    ):
        self._data: Dict[str, Any] = {}
        self._defaults = defaults or {}
        self._env_prefix = env_prefix
        
        # Load
        if config_file:
            self._load_file(config_file)
        
        self._apply_env_overrides()
    
    def _load_file(self, filepath: str):
        path = Path(filepath)
        
        if not path.exists():
            return
        
        ext = path.suffix.lower()
        
        if ext in (".yaml", ".yml"):
            with open(path) as f:
                self._data = yaml.safe_load(f) or {}
        
        elif ext == ".toml":
            if tomllib is None:
                raise ConfigError("TOML support requires: pip install tomli")
            with open(path, "rb") as f:
                self._data = tomllib.load(f)
        
        elif ext == ".json":
            with open(path) as f:
                self._data = json.load(f)
        
        elif ext in (".ini", ".cfg"):
            parser = configparser.ConfigParser()
            parser.read(path)
            for section in parser.sections():
                self._data[section] = dict(parser[section])
        
        else:
            raise ConfigError(f"Unsupported format: {ext}")
    
    def _apply_env_overrides(self):
        """Override config ด้วย env vars"""
        if not self._env_prefix:
            return
        
        prefix = self._env_prefix.upper() + "_"
        for key, value in os.environ.items():
            if not key.startswith(prefix):
                continue
            
            config_path = key[len(prefix):].lower().split("__")
            
            # Navigate/create nested dict
            current = self._data
            for part in config_path[:-1]:
                if part not in current:
                    current[part] = {}
                current = current[part]
            
            current[config_path[-1]] = self._parse_value(value)
    
    def _parse_value(self, value: str) -> Any:
        """Parse string value to appropriate type"""
        lower = value.lower()
        if lower in ("true", "yes", "1", "on"):
            return True
        if lower in ("false", "no", "0", "off"):
            return False
        try:
            return int(value)
        except ValueError:
            pass
        try:
            return float(value)
        except ValueError:
            pass
        return value
    
    def get(self, *path: str, default: Any = None) -> Any:
        """อ่านค่าด้วย dot path"""
        # ลอง defaults ก่อน
        current = self._data
        default_current = self._defaults
        
        for key in path:
            if isinstance(current, dict):
                current = current.get(key)
            else:
                current = None
            
            if isinstance(default_current, dict):
                default_current = default_current.get(key, default)
        
        return current if current is not None else (default_current or default)
    
    def require(self, *path: str) -> Any:
        """อ่านค่า required - raise ถ้าไม่มี"""
        value = self.get(*path)
        if value is None:
            raise ConfigError(f"Required config missing: {'.'.join(path)}")
        return value
    
    def get_int(self, *path: str, default: int = 0) -> int:
        value = self.get(*path, default=default)
        try:
            return int(value)
        except (TypeError, ValueError):
            return default
    
    def get_bool(self, *path: str, default: bool = False) -> bool:
        value = self.get(*path)
        if value is None:
            return default
        if isinstance(value, bool):
            return value
        return str(value).lower() in ("true", "1", "yes", "on")
    
    def get_list(self, *path: str, default: List = None) -> List:
        value = self.get(*path)
        if value is None:
            return default or []
        if isinstance(value, list):
            return value
        return [item.strip() for item in str(value).split(",")]
    
    def get_section(self, *path: str) -> "Config":
        """อ่าน section เป็น Config object"""
        section_data = self.get(*path) or {}
        child = Config.__new__(Config)
        child._data = section_data if isinstance(section_data, dict) else {}
        child._defaults = {}
        child._env_prefix = ""
        return child
    
    def __getitem__(self, key: str) -> Any:
        if key in self._data:
            return self._data[key]
        if key in self._defaults:
            return self._defaults[key]
        raise KeyError(key)
    
    def __contains__(self, key: str) -> bool:
        return key in self._data or key in self._defaults
    
    def to_dict(self) -> dict:
        return self._data.copy()


# ตัวอย่าง config.yaml:
# app:
#   name: MyApp
#   debug: false
#   secret_key: my-secret
# database:
#   host: localhost
#   port: 5432

# config = Config("config.yaml", env_prefix="APP")
# app_name = config.get("app", "name", default="MyApp")
# db_port = config.get_int("database", "port", default=5432)
# debug = config.get_bool("app", "debug")
```

### ตัวอย่างที่ 25: Config Schema Documentation Generator

```python
"""สร้าง documentation จาก config schema"""

import yaml
from typing import Any, Dict

def generate_config_docs(schema: Dict, indent: int = 0) -> str:
    """สร้าง Markdown docs จาก config schema"""
    lines = []
    prefix = "  " * indent
    
    for key, info in schema.items():
        if isinstance(info, dict) and "type" in info:
            # Leaf node
            type_str = info.get("type", "string")
            required = "**required**" if info.get("required") else ""
            default = f"(default: `{info['default']}`)" if "default" in info else ""
            desc = info.get("description", "")
            
            lines.append(f"{prefix}- `{key}` ({type_str}) {required} {default}")
            if desc:
                lines.append(f"{prefix}  {desc}")
        elif isinstance(info, dict):
            # Nested section
            lines.append(f"{prefix}- **{key}**:")
            lines.extend(generate_config_docs(info, indent + 1).split("\n"))
    
    return "\n".join(lines)


# Define schema
APP_CONFIG_SCHEMA = {
    "app": {
        "name": {"type": "string", "default": "MyApp", "description": "Application name"},
        "debug": {"type": "bool", "default": False, "description": "Enable debug mode"},
        "secret_key": {"type": "string", "required": True, "description": "Secret key (min 32 chars)"},
    },
    "database": {
        "host": {"type": "string", "default": "localhost", "description": "DB host"},
        "port": {"type": "int", "default": 5432, "description": "DB port"},
        "name": {"type": "string", "required": True, "description": "Database name"},
    },
    "server": {
        "host": {"type": "string", "default": "0.0.0.0"},
        "port": {"type": "int", "default": 8000},
        "workers": {"type": "int", "default": 1},
    },
}

# print(generate_config_docs(APP_CONFIG_SCHEMA))
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดที่ 1: INI Config Manager

```python
# เฉลย - INI config manager สำหรับ web app
import configparser
from pathlib import Path


class WebAppConfig:
    """Config manager สำหรับ web application"""
    
    CONFIG_FILE = "webapp.ini"
    
    DEFAULTS = {
        "debug": "false",
        "log_level": "INFO",
    }
    
    def __init__(self, config_file: str = None):
        self._config = configparser.ConfigParser(defaults=self.DEFAULTS)
        self._file = Path(config_file or self.CONFIG_FILE)
        
        if self._file.exists():
            self._config.read(self._file)
        else:
            self._create_default_config()
    
    def _create_default_config(self):
        """สร้าง default config"""
        self._config["app"] = {
            "name": "WebApp",
            "version": "1.0.0",
            "secret_key": "change-this-in-production",
        }
        self._config["server"] = {
            "host": "0.0.0.0",
            "port": "8000",
            "workers": "1",
        }
        self._config["database"] = {
            "url": "sqlite:///./app.db",
            "pool_size": "5",
        }
        self.save()
    
    def save(self):
        with open(self._file, "w") as f:
            self._config.write(f)
    
    @property
    def debug(self) -> bool:
        return self._config.getboolean("app", "debug", fallback=False)
    
    @property
    def app_name(self) -> str:
        return self._config.get("app", "name", fallback="WebApp")
    
    @property
    def server_port(self) -> int:
        return self._config.getint("server", "port", fallback=8000)
    
    @property
    def database_url(self) -> str:
        return self._config.get("database", "url", fallback="sqlite:///./app.db")
    
    def update(self, section: str, key: str, value: str):
        """อัพเดต config"""
        if not self._config.has_section(section):
            self._config.add_section(section)
        self._config.set(section, key, str(value))
        self.save()
    
    def to_dict(self) -> dict:
        result = {}
        for section in self._config.sections():
            result[section] = {
                k: v for k, v in self._config.items(section)
                if k not in self.DEFAULTS  # exclude DEFAULT keys
            }
        return result


# ทดสอบ
import os, tempfile

with tempfile.NamedTemporaryFile(suffix=".ini", delete=False) as tmp:
    config_path = tmp.name

config = WebAppConfig(config_path)
print(f"App: {config.app_name}")
print(f"Port: {config.server_port}")
print(f"Debug: {config.debug}")

config.update("server", "port", "9000")
print(f"New port: {config.server_port}")
os.unlink(config_path)
```

### แบบฝึกหัดที่ 2: YAML Config Merger

```python
# เฉลย
import yaml
from pathlib import Path
from typing import Any, Dict


def deep_merge(base: dict, override: dict) -> dict:
    """Merge two dicts recursively"""
    result = base.copy()
    for key, value in override.items():
        if (key in result and 
            isinstance(result[key], dict) and 
            isinstance(value, dict)):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result


def load_yaml_with_includes(filepath: str) -> dict:
    """โหลด YAML พร้อม include support"""
    
    class IncludeLoader(yaml.SafeLoader):
        pass
    
    base_path = Path(filepath).parent
    
    def construct_include(loader, node):
        """Handle !include directive"""
        include_file = base_path / loader.construct_scalar(node)
        return load_yaml_with_includes(str(include_file))
    
    IncludeLoader.add_constructor("!include", construct_include)
    
    with open(filepath) as f:
        return yaml.load(f, Loader=IncludeLoader) or {}


class YAMLConfigMerger:
    """Merge YAML configs จากหลายไฟล์"""
    
    def __init__(self, base_config: str):
        self._config = {}
        self._load_base(base_config)
    
    def _load_base(self, filepath: str):
        with open(filepath) as f:
            self._config = yaml.safe_load(f) or {}
    
    def merge_from_file(self, filepath: str, override: bool = True):
        """Merge config จากไฟล์"""
        with open(filepath) as f:
            extra = yaml.safe_load(f) or {}
        
        if override:
            self._config = deep_merge(self._config, extra)
        else:
            self._config = deep_merge(extra, self._config)
        return self
    
    def merge_from_dict(self, data: dict):
        """Merge config จาก dict"""
        self._config = deep_merge(self._config, data)
        return self
    
    def to_dict(self) -> dict:
        return self._config.copy()
    
    def save(self, filepath: str):
        with open(filepath, "w") as f:
            yaml.dump(self._config, f, default_flow_style=False)
    
    def get(self, *path, default=None):
        current = self._config
        for key in path:
            if not isinstance(current, dict):
                return default
            current = current.get(key, default)
        return current


# ทดสอบ
import tempfile, os

base_yaml = """
app:
  name: MyApp
  debug: false
database:
  host: localhost
  port: 5432
"""

override_yaml = """
app:
  debug: true
database:
  host: production-db.example.com
  ssl: true
"""

with tempfile.NamedTemporaryFile(mode='w', suffix='.yaml', delete=False) as f:
    f.write(base_yaml)
    base_file = f.name

with tempfile.NamedTemporaryFile(mode='w', suffix='.yaml', delete=False) as f:
    f.write(override_yaml)
    override_file = f.name

merger = YAMLConfigMerger(base_file)
merger.merge_from_file(override_file)

result = merger.to_dict()
print(f"Debug: {result['app']['debug']}")          # True
print(f"DB Host: {result['database']['host']}")    # production-db.example.com
print(f"DB Port: {result['database']['port']}")    # 5432 (from base)

os.unlink(base_file)
os.unlink(override_file)
```

### แบบฝึกหัดที่ 3-8: Additional Practice

```python
# แบบฝึกหัดที่ 3: TOML Config Generator
"""
สร้างฟังก์ชัน generate_pyproject_toml() ที่:
- รับข้อมูล project จาก user
- สร้าง pyproject.toml ที่สมบูรณ์
- รวม tool configs สำหรับ pytest, black, ruff
"""

# แบบฝึกหัดที่ 4: Multi-format Config Converter
"""
สร้าง class ConfigConverter ที่แปลง config ระหว่าง formats:
- INI → YAML
- YAML → TOML
- JSON → YAML
- ฯลฯ
"""

# แบบฝึกหัดที่ 5: Config Diff Tool
"""
สร้างฟังก์ชัน diff_configs(config1_file, config2_file) ที่:
- โหลด config สองไฟล์
- แสดงความแตกต่าง
- Format เป็น colored output
"""

# แบบฝึกหัดที่ 6: Config Template Engine
"""
สร้าง ConfigTemplateEngine ที่:
- อ่าน YAML template
- แทนที่ {{ variable }} ด้วยค่าจริง
- รองรับ conditional sections
"""

# แบบฝึกหัดที่ 7: Config Validation CLI
"""
สร้าง CLI tool ที่:
- รับ config file ทุก format
- Validate ด้วย JSON schema
- แสดง errors อย่างชัดเจน
"""

# แบบฝึกหัดที่ 8: App Config สมบูรณ์
"""
สร้างระบบ config สมบูรณ์สำหรับ REST API:
- อ่านจาก config.yaml + environment variables
- Validate ด้วย Pydantic
- Support hot reload
- Generate .env.example อัตโนมัติ
"""

# เฉลย แบบฝึกหัดที่ 4: Multi-format Config Converter
import json
import yaml
from pathlib import Path

try:
    import tomllib
except ImportError:
    import tomli as tomllib

try:
    import tomli_w
    HAS_TOML_WRITE = True
except ImportError:
    HAS_TOML_WRITE = False


class ConfigConverter:
    """แปลง config ระหว่าง formats"""
    
    def convert(self, source_file: str, dest_file: str) -> bool:
        """แปลงไฟล์"""
        src_path = Path(source_file)
        dst_path = Path(dest_file)
        
        # Load
        data = self._load(src_path)
        
        # Save
        self._save(data, dst_path)
        
        print(f"Converted {src_path.suffix} → {dst_path.suffix}: {dst_path}")
        return True
    
    def _load(self, path: Path) -> dict:
        ext = path.suffix.lower()
        
        if ext in (".yaml", ".yml"):
            with open(path) as f:
                return yaml.safe_load(f) or {}
        elif ext == ".toml":
            with open(path, "rb") as f:
                return tomllib.load(f)
        elif ext == ".json":
            with open(path) as f:
                return json.load(f)
        else:
            raise ValueError(f"Unsupported source format: {ext}")
    
    def _save(self, data: dict, path: Path):
        path.parent.mkdir(parents=True, exist_ok=True)
        ext = path.suffix.lower()
        
        if ext in (".yaml", ".yml"):
            with open(path, "w") as f:
                yaml.dump(data, f, default_flow_style=False, allow_unicode=True)
        elif ext == ".toml":
            if not HAS_TOML_WRITE:
                raise ImportError("pip install tomli-w")
            with open(path, "wb") as f:
                tomli_w.dump(data, f)
        elif ext == ".json":
            with open(path, "w") as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
        else:
            raise ValueError(f"Unsupported destination format: {ext}")


# ทดสอบ
import tempfile, os

yaml_content = """
app:
  name: TestApp
  version: "1.0.0"
  debug: false
database:
  host: localhost
  port: 5432
"""

with tempfile.NamedTemporaryFile(mode='w', suffix='.yaml', delete=False) as f:
    f.write(yaml_content)
    yaml_file = f.name

converter = ConfigConverter()

# YAML → JSON
json_file = yaml_file.replace('.yaml', '.json')
converter.convert(yaml_file, json_file)

# แสดงผล JSON
with open(json_file) as f:
    print(json.dumps(json.load(f), indent=2))

os.unlink(yaml_file)
os.unlink(json_file)
```

---

## สรุป

### เปรียบเทียบ Format

```
INI:
  ✅ Simple, human-readable
  ✅ Python built-in (configparser)
  ❌ No nested structures (beyond sections)
  ❌ No native types (all strings)
  ❌ Limited comment support

YAML:
  ✅ Very readable
  ✅ Full data types support
  ✅ Comments supported
  ✅ Anchors & aliases (DRY)
  ❌ Whitespace sensitive (indent errors)
  ❌ Requires PyYAML
  ❌ Security issues with yaml.load() (use safe_load!)

TOML:
  ✅ Clear, unambiguous syntax
  ✅ Strong data types (int, float, bool, date)
  ✅ Comments supported
  ✅ Python 3.11+ built-in (tomllib)
  ❌ No multi-line string support in keys
  ❌ Limited for deeply nested configs

JSON:
  ✅ Universal support
  ✅ Language-agnostic
  ✅ Strong tooling
  ❌ No comments
  ❌ Verbose (quotes everywhere)
  ❌ No trailing commas
```

### Best Practices

1. **ใช้ YAML สำหรับ** Docker Compose, Kubernetes, complex configs
2. **ใช้ TOML สำหรับ** pyproject.toml, simple Python project configs
3. **ใช้ INI สำหรับ** legacy configs, simple key-value settings
4. **ใช้ JSON สำหรับ** API configs, language-agnostic data
5. **Validate เสมอ** ด้วย Pydantic หรือ schema validation
6. **ใช้ yaml.safe_load()** ไม่ใช่ yaml.load() (security!)
7. **Support environment overrides** สำหรับ 12-Factor compliance

### Installation Summary

```bash
pip install pyyaml          # YAML support
pip install ruamel.yaml     # YAML with comment preservation
pip install tomli           # TOML read (Python < 3.11)
pip install tomli-w         # TOML write
pip install jsonschema      # JSON schema validation
pip install pydantic        # Config validation
```
