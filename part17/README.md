# Part 17: Modules, Packages & pip

## สารบัญ
1. [การ import Modules แบบต่างๆ](#1-การ-import-modules-แบบต่างๆ)
2. [Creating Your Own Modules](#2-creating-your-own-modules)
3. [\_\_name\_\_ == "\_\_main\_\_"](#3-__name__--__main__)
4. [Package Structure](#4-package-structure)
5. [Relative Imports](#5-relative-imports)
6. [Python Standard Library](#6-python-standard-library)
7. [pip Package Manager](#7-pip-package-manager)
8. [Virtual Environments](#8-virtual-environments)
9. [requirements.txt](#9-requirementstxt)
10. [Popular Third-party Packages](#10-popular-third-party-packages)
11. [ตัวอย่างโปรแกรมจริง](#11-ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. การ import Modules แบบต่างๆ

Module คือไฟล์ Python ที่มีโค้ด (functions, classes, variables) ที่เราสามารถนำไปใช้ในโปรแกรมอื่นได้

### 1.1 import แบบพื้นฐาน

```python
# import ทั้ง module
import math

# ใช้งานต้องระบุชื่อ module
print(math.pi)           # 3.141592653589793
print(math.sqrt(16))     # 4.0
print(math.ceil(4.2))    # 5
print(math.floor(4.8))   # 4
print(math.factorial(5)) # 120
```

```python
# import หลาย modules
import os
import sys
import json

# ดู Python version
print(f"Python version: {sys.version}")

# ดู current directory
print(f"Current dir: {os.getcwd()}")

# แปลง dict เป็น JSON string
data = {"name": "Alice", "age": 25}
json_string = json.dumps(data, ensure_ascii=False)
print(f"JSON: {json_string}")
```

### 1.2 from ... import

```python
# import เฉพาะสิ่งที่ต้องการ
from math import pi, sqrt, factorial

# ใช้งานโดยตรงโดยไม่ต้องระบุ module
print(pi)           # 3.141592653589793
print(sqrt(25))     # 5.0
print(factorial(6)) # 720
```

```python
# from ... import *  (ไม่แนะนำ!)
from math import *  # import ทุกอย่างจาก math

print(sin(pi/2))    # 1.0
print(cos(0))       # 1.0
print(tan(pi/4))    # ~1.0

# ปัญหา: อาจเกิด name conflicts
# ไม่รู้ว่า sin มาจากที่ไหน
```

```python
# from ... import ที่แนะนำ - ระบุชัดเจน
from datetime import datetime, date, timedelta

now = datetime.now()
today = date.today()
tomorrow = today + timedelta(days=1)

print(f"ตอนนี้: {now.strftime('%Y-%m-%d %H:%M:%S')}")
print(f"วันนี้: {today}")
print(f"พรุ่งนี้: {tomorrow}")
```

### 1.3 import ... as (alias)

```python
# ตั้งชื่อใหม่ให้ module
import numpy as np          # convention
import pandas as pd         # convention
import matplotlib.pyplot as plt  # convention

# ตัวอย่างที่ไม่ต้องติดตั้ง
import datetime as dt
import collections as col
import functools as ft

now = dt.datetime.now()
counter = col.Counter([1, 2, 2, 3, 3, 3])
print(f"Time: {now}")
print(f"Counter: {counter}")
```

```python
# import function พร้อม alias
from os.path import join as path_join
from os.path import exists as path_exists
from os.path import dirname as path_dirname

file_path = path_join("/home", "user", "documents", "file.txt")
print(f"Path: {file_path}")
print(f"Exists: {path_exists(file_path)}")
print(f"Directory: {path_dirname(file_path)}")
```

```python
# ใช้ alias เมื่อชื่อ module ยาวหรือซ้ำกัน
from xml.etree import ElementTree as ET
import email.mime.text as MIMEText
import urllib.parse as urlparse

# ใช้งาน
url = "https://example.com/search?q=python&lang=th"
parsed = urlparse.urlparse(url)
print(f"Scheme: {parsed.scheme}")
print(f"Host: {parsed.netloc}")
print(f"Path: {parsed.path}")
print(f"Query: {parsed.query}")
```

### 1.4 Lazy Import (import ภายใน function)

```python
# import ภายใน function - ใช้เมื่อต้องการเท่านั้น
def process_csv(filename):
    """ประมวลผล CSV file"""
    import csv  # lazy import
    
    with open(filename, 'r', newline='', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        return list(reader)

def send_email(subject, body, to_address):
    """ส่ง email"""
    import smtplib  # lazy import
    from email.mime.text import MIMEText
    
    msg = MIMEText(body)
    msg['Subject'] = subject
    # ... rest of code
    pass
```

---

## 2. Creating Your Own Modules

### 2.1 สร้าง Module อย่างง่าย

```python
# ไฟล์: math_utils.py
"""
Module สำหรับการคำนวณทางคณิตศาสตร์
"""

PI = 3.14159265358979

def circle_area(radius):
    """คำนวณพื้นที่วงกลม"""
    if radius < 0:
        raise ValueError("radius ต้องเป็นค่าบวก")
    return PI * radius ** 2

def circle_circumference(radius):
    """คำนวณเส้นรอบวงกลม"""
    if radius < 0:
        raise ValueError("radius ต้องเป็นค่าบวก")
    return 2 * PI * radius

def rectangle_area(width, height):
    """คำนวณพื้นที่สี่เหลี่ยมผืนผ้า"""
    return width * height

def triangle_area(base, height):
    """คำนวณพื้นที่สามเหลี่ยม"""
    return 0.5 * base * height

def is_prime(n):
    """ตรวจสอบว่าเป็นจำนวนเฉพาะหรือไม่"""
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

def fibonacci(n):
    """สร้าง Fibonacci sequence"""
    if n <= 0:
        return []
    elif n == 1:
        return [0]
    
    sequence = [0, 1]
    while len(sequence) < n:
        sequence.append(sequence[-1] + sequence[-2])
    return sequence
```

```python
# ใช้งาน math_utils.py
# (สมมติว่าไฟล์อยู่ในโฟลเดอร์เดียวกัน)

# import ทั้ง module
# import math_utils
# print(math_utils.circle_area(5))

# หรือ import เฉพาะส่วน
# from math_utils import circle_area, fibonacci, is_prime

# จำลองการใช้งาน (ไม่ต้อง import จริง)
PI = 3.14159265358979

def circle_area(radius):
    return PI * radius ** 2

def fibonacci(n):
    if n <= 0:
        return []
    elif n == 1:
        return [0]
    sequence = [0, 1]
    while len(sequence) < n:
        sequence.append(sequence[-1] + sequence[-2])
    return sequence

def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

# ใช้งาน
print(f"พื้นที่วงกลม radius=5: {circle_area(5):.2f}")
print(f"Fibonacci 10 ตัว: {fibonacci(10)}")
print(f"จำนวนเฉพาะ 1-20: {[n for n in range(1, 21) if is_prime(n)]}")
```

### 2.2 Module Attributes

```python
# ทุก module มี attributes พิเศษ
import os

print(f"Module name: {os.__name__}")
print(f"Module file: {os.__file__}")
print(f"Module doc: {os.__doc__[:50] if os.__doc__ else 'None'}...")

# ดูสิ่งทั้งหมดใน module
public_attrs = [attr for attr in dir(os) if not attr.startswith('_')]
print(f"จำนวน public attributes: {len(public_attrs)}")
print(f"บางส่วน: {public_attrs[:10]}")
```

```python
# สร้าง module พร้อม metadata
"""
Module: string_utils.py
Version: 1.0.0
Author: Your Name
Description: Utility functions สำหรับ string manipulation
"""

__version__ = "1.0.0"
__author__ = "Your Name"
__all__ = ['clean_text', 'word_count', 'truncate']  # ควบคุม import *

def clean_text(text):
    """ทำความสะอาด text"""
    import re
    # ลบ whitespace ซ้ำ
    text = re.sub(r'\s+', ' ', text)
    # ลบ special characters ยกเว้น Thai, English, numbers
    text = re.sub(r'[^\w\sก-๙]', '', text)
    return text.strip()

def word_count(text):
    """นับจำนวนคำ"""
    if not text:
        return 0
    return len(text.split())

def truncate(text, max_length=100, suffix="..."):
    """ตัด text ให้สั้นลง"""
    if len(text) <= max_length:
        return text
    return text[:max_length - len(suffix)] + suffix

def _private_helper():
    """ฟังก์ชัน private (ไม่ถูก export)"""
    pass

# ทดสอบ
sample_text = "  สวัสดี   Python   Programming!  "
print(f"Clean: '{clean_text(sample_text)}'")
print(f"Words: {word_count(sample_text)}")
print(f"Truncate: '{truncate('Python is a great programming language', 20)}'")
```

---

## 3. \_\_name\_\_ == "\_\_main\_\_"

```python
# ทำความเข้าใจ __name__
# เมื่อรัน script โดยตรง: __name__ == "__main__"
# เมื่อ import เป็น module: __name__ == "ชื่อไฟล์"

print(f"__name__ ปัจจุบัน: {__name__}")

# pattern ที่ใช้บ่อย
if __name__ == "__main__":
    print("รันโดยตรง ไม่ใช่ import")
    # ใส่โค้ดที่ต้องการรันเมื่อเป็น script โดยตรง
```

```python
# ตัวอย่าง: calculator.py
"""Calculator module พร้อม test"""

def add(a, b):
    """บวกสองจำนวน"""
    return a + b

def subtract(a, b):
    """ลบสองจำนวน"""
    return a - b

def multiply(a, b):
    """คูณสองจำนวน"""
    return a * b

def divide(a, b):
    """หารสองจำนวน"""
    if b == 0:
        raise ZeroDivisionError("ไม่สามารถหารด้วยศูนย์")
    return a / b

def run_tests():
    """รัน tests"""
    test_cases = [
        (add(2, 3), 5, "add(2, 3)"),
        (subtract(10, 4), 6, "subtract(10, 4)"),
        (multiply(3, 4), 12, "multiply(3, 4)"),
        (divide(15, 3), 5.0, "divide(15, 3)"),
    ]
    
    passed = 0
    for result, expected, description in test_cases:
        if result == expected:
            print(f"✓ {description} = {result}")
            passed += 1
        else:
            print(f"✗ {description}: expected {expected}, got {result}")
    
    print(f"\n{passed}/{len(test_cases)} tests passed")

# รันเฉพาะเมื่อรัน script โดยตรง
if __name__ == "__main__":
    print("=== Calculator Tests ===")
    run_tests()
    
    print("\n=== Interactive Calculator ===")
    # อาจมี interactive loop ที่นี่
```

```python
# pattern สำหรับ module ที่สมบูรณ์
"""
server.py - Simple HTTP server module
"""

import os
import sys

DEFAULT_HOST = "localhost"
DEFAULT_PORT = 8080

class Server:
    def __init__(self, host=DEFAULT_HOST, port=DEFAULT_PORT):
        self.host = host
        self.port = port
        self.running = False
    
    def start(self):
        print(f"เริ่ม server ที่ {self.host}:{self.port}")
        self.running = True
    
    def stop(self):
        print("หยุด server")
        self.running = False
    
    def handle_request(self, path):
        print(f"รับ request: {path}")
        return {"status": 200, "body": f"Hello from {path}"}

def main():
    """Main function สำหรับรัน server"""
    import argparse
    
    parser = argparse.ArgumentParser(description="Simple HTTP Server")
    parser.add_argument("--host", default=DEFAULT_HOST)
    parser.add_argument("--port", type=int, default=DEFAULT_PORT)
    
    # ใช้ namespace โดยตรงแทน parse_args() เพื่อไม่ต้องรับ command line args จริง
    args = argparse.Namespace(host=DEFAULT_HOST, port=DEFAULT_PORT)
    
    server = Server(args.host, args.port)
    server.start()
    
    # จำลอง requests
    for path in ["/", "/users", "/api/data"]:
        response = server.handle_request(path)
        print(f"Response: {response}")
    
    server.stop()

if __name__ == "__main__":
    main()
```

---

## 4. Package Structure

Package คือ directory ที่มีไฟล์ `__init__.py` ที่บรรจุ modules หลายไฟล์

### 4.1 โครงสร้าง Package

```
mypackage/
├── __init__.py          # ทำให้เป็น package
├── utils.py             # utility functions
├── models.py            # data models
├── config.py            # configuration
└── subpackage/
    ├── __init__.py      # subpackage
    ├── api.py
    └── database.py
```

### 4.2 \_\_init\_\_.py

```python
# mypackage/__init__.py

"""
MyPackage - Example Python Package
"""

__version__ = "1.0.0"
__author__ = "Your Name"

# import สิ่งสำคัญมาให้ใช้งานได้ง่าย
# from .utils import helper_function
# from .models import User, Product

# กำหนดสิ่งที่ export เมื่อใช้ from package import *
__all__ = ['utils', 'models']

# ทำ initialization ที่จำเป็น
print(f"MyPackage v{__version__} initialized")
```

```python
# ตัวอย่าง: สร้าง package structure จำลอง

# จำลอง myapp/config.py
class Config:
    DEBUG = False
    DATABASE_URL = "sqlite:///app.db"
    SECRET_KEY = "dev-secret-key"
    MAX_CONNECTIONS = 10

class DevelopmentConfig(Config):
    DEBUG = True
    DATABASE_URL = "sqlite:///dev.db"

class ProductionConfig(Config):
    DEBUG = False
    DATABASE_URL = "postgresql://user:pass@localhost/prod"
    MAX_CONNECTIONS = 100

# จำลอง myapp/models.py
class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email
    
    def __repr__(self):
        return f"User(id={self.id}, name='{self.name}')"
    
    def to_dict(self):
        return {"id": self.id, "name": self.name, "email": self.email}

class Product:
    def __init__(self, id, name, price):
        self.id = id
        self.name = name
        self.price = price
    
    def __repr__(self):
        return f"Product(id={self.id}, name='{self.name}', price={self.price})"

# จำลอง myapp/utils.py
def format_currency(amount, currency="THB"):
    """จัดรูปแบบเงิน"""
    if currency == "THB":
        return f"฿{amount:,.2f}"
    elif currency == "USD":
        return f"${amount:,.2f}"
    return f"{amount:,.2f} {currency}"

def paginate(items, page=1, per_page=10):
    """แบ่งหน้าข้อมูล"""
    start = (page - 1) * per_page
    end = start + per_page
    return {
        "items": items[start:end],
        "total": len(items),
        "page": page,
        "per_page": per_page,
        "total_pages": (len(items) + per_page - 1) // per_page
    }

# ทดสอบ
users = [User(i, f"User{i}", f"user{i}@example.com") for i in range(1, 25)]
page_data = paginate(users, page=2, per_page=5)
print(f"หน้า {page_data['page']}/{page_data['total_pages']}")
print(f"รายการ: {page_data['items']}")
print(f"ราคา: {format_currency(1999.99)}")
```

### 4.3 Package ที่ซับซ้อน

```python
# โครงสร้าง: webapp/
# webapp/__init__.py
# webapp/routes.py
# webapp/templates/
# webapp/static/
# webapp/database/
#   __init__.py
#   models.py
#   migrations.py

# จำลอง webapp/__init__.py
class WebApp:
    """Simple web application"""
    
    def __init__(self, name, config=None):
        self.name = name
        self.config = config or {}
        self.routes = {}
        self.middleware = []
    
    def route(self, path):
        """Decorator สำหรับ register route"""
        def decorator(func):
            self.routes[path] = func
            return func
        return decorator
    
    def use(self, middleware_func):
        """เพิ่ม middleware"""
        self.middleware.append(middleware_func)
    
    def handle(self, path, request=None):
        """จัดการ request"""
        if path not in self.routes:
            return {"status": 404, "body": "Not Found"}
        
        handler = self.routes[path]
        return handler(request or {})

# สร้าง app
app = WebApp("MyApp")

@app.route("/")
def home(request):
    return {"status": 200, "body": "Welcome to MyApp!"}

@app.route("/users")
def users(request):
    return {"status": 200, "body": ["Alice", "Bob", "Charlie"]}

@app.route("/api/status")
def status(request):
    return {"status": 200, "body": {"app": app.name, "version": "1.0"}}

# ทดสอบ
for path in ["/", "/users", "/api/status", "/notfound"]:
    response = app.handle(path)
    print(f"GET {path}: {response}")
```

---

## 5. Relative Imports

Relative imports ใช้ภายใน package เพื่อ import จาก module ที่อยู่ใน package เดียวกัน

```python
# โครงสร้าง:
# mypackage/
# ├── __init__.py
# ├── module_a.py
# ├── module_b.py
# └── subpackage/
#     ├── __init__.py
#     └── module_c.py

# ใน module_b.py:
# from . import module_a          # import sibling module
# from .module_a import function  # import function จาก sibling

# ใน subpackage/module_c.py:
# from .. import module_a         # import จาก parent package
# from ..module_b import Class    # import จาก parent's module

# ตัวอย่าง relative imports
"""
# mypackage/utils.py
def helper():
    return "helper function"

# mypackage/models.py  
from .utils import helper  # . = current package (mypackage)

class User:
    def get_help(self):
        return helper()

# mypackage/subpackage/api.py
from ..models import User      # .. = parent package (mypackage)
from ..utils import helper     # import จาก parent package

def create_user():
    user = User()
    return user
"""

# จำลองการทำงาน
print("Relative imports ใช้งานภายใน package เท่านั้น")
print("ตัวอย่าง:")
print("  from . import module     # import sibling module")
print("  from .module import func # import function จาก sibling")
print("  from .. import module    # import จาก parent")
print("  from ..module import func # import จาก parent's module")
```

---

## 6. Python Standard Library

Python มาพร้อม Standard Library ขนาดใหญ่มาก

### 6.1 os และ os.path

```python
import os
import os.path

# ข้อมูลระบบ
print(f"OS: {os.name}")
print(f"Current dir: {os.getcwd()}")
print(f"Home dir: {os.path.expanduser('~')}")

# จัดการ paths
path = os.path.join("/home", "user", "documents", "file.txt")
print(f"Path: {path}")
print(f"Exists: {os.path.exists(path)}")
print(f"Dir: {os.path.dirname(path)}")
print(f"Filename: {os.path.basename(path)}")
name, ext = os.path.splitext("document.pdf")
print(f"Name: {name}, Extension: {ext}")

# Environment variables
home = os.environ.get("HOME", "/home/user")
path_var = os.environ.get("PATH", "")
print(f"HOME: {home}")
print(f"PATH (first 50 chars): {path_var[:50]}")

# Directory operations
# os.makedirs("new/nested/directory", exist_ok=True)
# os.listdir(".")  # list directory contents

# Walk directory tree
for root, dirs, files in os.walk("/tmp"):
    for filename in files[:3]:  # แสดงแค่ 3 ไฟล์แรก
        filepath = os.path.join(root, filename)
        print(f"File: {filepath}")
    break  # แค่ level แรก
```

### 6.2 pathlib (Modern Path Handling)

```python
from pathlib import Path

# สร้าง path
home = Path.home()
docs = home / "Documents"
file_path = docs / "report.txt"

print(f"Home: {home}")
print(f"Docs: {docs}")
print(f"File: {file_path}")
print(f"Name: {file_path.name}")
print(f"Stem: {file_path.stem}")      # ชื่อไม่มี extension
print(f"Suffix: {file_path.suffix}")  # extension
print(f"Parent: {file_path.parent}")

# ตรวจสอบ
print(f"Exists: {file_path.exists()}")
print(f"Is file: {file_path.is_file()}")
print(f"Is dir: {file_path.is_dir()}")

# glob patterns
tmp_path = Path("/tmp")
py_files = list(tmp_path.glob("*.py"))
print(f"Python files in /tmp: {len(py_files)}")

all_files = list(tmp_path.rglob("*"))
print(f"All files recursive: {len(all_files)}")
```

### 6.3 datetime

```python
from datetime import datetime, date, time, timedelta
import calendar

# datetime พื้นฐาน
now = datetime.now()
today = date.today()

print(f"Now: {now}")
print(f"Today: {today}")
print(f"Year: {now.year}, Month: {now.month}, Day: {now.day}")
print(f"Hour: {now.hour}, Minute: {now.minute}, Second: {now.second}")

# formatting
formatted = now.strftime("%d/%m/%Y %H:%M:%S")
print(f"Formatted: {formatted}")

thai_format = now.strftime("วันที่ %d เดือน %m ปี %Y เวลา %H:%M น.")
print(f"Thai: {thai_format}")

# parsing
date_str = "2024-01-15 14:30:00"
parsed = datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S")
print(f"Parsed: {parsed}")

# arithmetic
tomorrow = today + timedelta(days=1)
next_week = today + timedelta(weeks=1)
last_month = today - timedelta(days=30)

print(f"Tomorrow: {tomorrow}")
print(f"Next week: {next_week}")
print(f"30 days ago: {last_month}")

# คำนวณความต่าง
birthday = date(1990, 5, 15)
age_days = (today - birthday).days
age_years = age_days // 365
print(f"อายุประมาณ: {age_years} ปี ({age_days} วัน)")
```

### 6.4 collections

```python
from collections import defaultdict, Counter, OrderedDict, namedtuple, deque

# 1. Counter - นับความถี่
words = "the quick brown fox jumps over the lazy dog the".split()
word_count = Counter(words)
print(f"Word counts: {word_count}")
print(f"Most common 3: {word_count.most_common(3)}")

letters = Counter("programming")
print(f"Letter count: {letters}")

# 2. defaultdict - dict พร้อม default value
scores = defaultdict(list)  # ถ้า key ไม่มี จะสร้าง [] อัตโนมัติ
scores["Alice"].append(90)
scores["Alice"].append(85)
scores["Bob"].append(78)

for name, grades in scores.items():
    avg = sum(grades) / len(grades)
    print(f"{name}: avg {avg:.1f}")

# 3. namedtuple - tuple ที่มีชื่อ field
Point = namedtuple('Point', ['x', 'y'])
Color = namedtuple('Color', ['red', 'green', 'blue'])

p = Point(3, 4)
c = Color(255, 128, 0)

print(f"Point: x={p.x}, y={p.y}")
print(f"Color: R={c.red}, G={c.green}, B={c.blue}")
print(f"Distance: {(p.x**2 + p.y**2)**0.5:.2f}")

# 4. deque - double-ended queue
dq = deque([1, 2, 3, 4, 5])
dq.appendleft(0)  # เพิ่มด้านซ้าย
dq.append(6)      # เพิ่มด้านขวา
print(f"Deque: {dq}")

dq.popleft()  # ลบด้านซ้าย
dq.pop()      # ลบด้านขวา
print(f"After pop: {dq}")

# เหมาะสำหรับ sliding window
data = [1, 2, 3, 4, 5, 6, 7, 8]
window = deque(data[:3], maxlen=3)  # size 3
for item in data[3:]:
    window.append(item)  # ถ้าเต็ม จะ pop left อัตโนมัติ
    print(f"Window: {list(window)}")
```

### 6.5 itertools

```python
import itertools

# 1. chain - รวม iterables
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]
combined = list(itertools.chain(list1, list2, list3))
print(f"Chain: {combined}")

# 2. combinations and permutations
items = ['A', 'B', 'C']
combs = list(itertools.combinations(items, 2))
perms = list(itertools.permutations(items, 2))
print(f"Combinations of 2: {combs}")
print(f"Permutations of 2: {perms}")

# 3. product - Cartesian product
colors = ['red', 'blue']
sizes = ['S', 'M', 'L']
products = list(itertools.product(colors, sizes))
print(f"Products: {products}")

# 4. groupby - จัดกลุ่ม
data = [
    {"name": "Alice", "dept": "Engineering"},
    {"name": "Bob", "dept": "Marketing"},
    {"name": "Charlie", "dept": "Engineering"},
    {"name": "Diana", "dept": "Marketing"},
    {"name": "Eve", "dept": "Engineering"},
]

# ต้อง sort ก่อน groupby
data.sort(key=lambda x: x["dept"])
for dept, members in itertools.groupby(data, key=lambda x: x["dept"]):
    member_names = [m["name"] for m in members]
    print(f"{dept}: {member_names}")

# 5. accumulate
import operator
numbers = [1, 2, 3, 4, 5]
cumsum = list(itertools.accumulate(numbers))
cumprod = list(itertools.accumulate(numbers, operator.mul))
print(f"Cumulative sum: {cumsum}")
print(f"Cumulative product: {cumprod}")
```

### 6.6 json

```python
import json
from datetime import datetime

# Encoding (Python -> JSON)
data = {
    "name": "Alice",
    "age": 25,
    "scores": [90, 85, 92],
    "address": {
        "city": "Bangkok",
        "country": "Thailand"
    },
    "active": True,
    "notes": None
}

# แปลงเป็น JSON string
json_str = json.dumps(data, ensure_ascii=False, indent=2)
print("JSON string:")
print(json_str)

# Decoding (JSON -> Python)
loaded = json.loads(json_str)
print(f"\nLoaded type: {type(loaded)}")
print(f"Name: {loaded['name']}")

# Custom encoder
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

data_with_date = {
    "user": "Alice",
    "created_at": datetime.now(),
    "data": [1, 2, 3]
}

json_with_date = json.dumps(data_with_date, cls=DateTimeEncoder, indent=2)
print("\nJSON with datetime:")
print(json_with_date)

# อ่าน/เขียน JSON file
import tempfile
import os

with tempfile.NamedTemporaryFile(mode='w', suffix='.json', delete=False, encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
    temp_file = f.name

with open(temp_file, 'r', encoding='utf-8') as f:
    loaded_from_file = json.load(f)

os.unlink(temp_file)
print(f"\nLoaded from file: {loaded_from_file['name']}")
```

### 6.7 re (Regular Expressions)

```python
import re

# พื้นฐาน
text = "Python 3.12 released on 2023-10-02"

# search - หาตำแหน่งแรก
match = re.search(r'\d+\.\d+', text)
if match:
    print(f"Version: {match.group()}")

# findall - หาทั้งหมด
dates = re.findall(r'\d{4}-\d{2}-\d{2}', text)
print(f"Dates: {dates}")

numbers = re.findall(r'\d+', text)
print(f"All numbers: {numbers}")

# match - ตรวจสอบจากต้น string
emails = [
    "user@example.com",
    "invalid-email",
    "another.user@domain.co.th"
]

email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
for email in emails:
    if re.match(email_pattern, email):
        print(f"✓ Valid: {email}")
    else:
        print(f"✗ Invalid: {email}")

# sub - แทนที่
phone = "โทร: 081-234-5678 หรือ 02 123 4567"
normalized = re.sub(r'[\s\-]', '', phone)
print(f"Normalized: {normalized}")

# split
data = "apple,banana;cherry|grape"
fruits = re.split(r'[,;|]', data)
print(f"Fruits: {fruits}")

# groups
pattern = r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})'
date_str = "2024-03-15"
match = re.match(pattern, date_str)
if match:
    print(f"Year: {match.group('year')}")
    print(f"Month: {match.group('month')}")
    print(f"Day: {match.group('day')}")
```

---

## 7. pip Package Manager

pip คือ package installer สำหรับ Python

### 7.1 คำสั่ง pip พื้นฐาน

```bash
# ตรวจสอบ version
pip --version
pip3 --version

# ติดตั้ง package
pip install requests
pip install "requests>=2.28.0"      # ระบุ version ขั้นต่ำ
pip install "requests==2.28.1"      # ระบุ version เฉพาะ
pip install "requests>=2.25,<3.0"   # ระบุ range

# ติดตั้งหลาย packages พร้อมกัน
pip install requests flask sqlalchemy

# ถอนการติดตั้ง
pip uninstall requests
pip uninstall requests flask -y  # -y = ยืนยันอัตโนมัติ

# อัพเดท package
pip install --upgrade requests
pip install -U requests  # shorthand

# ดู packages ที่ติดตั้ง
pip list
pip list --outdated  # packages ที่มีเวอร์ชันใหม่

# ดูข้อมูล package
pip show requests
pip show --files requests

# export packages
pip freeze > requirements.txt
pip freeze --local > requirements.txt  # เฉพาะ local

# ติดตั้งจาก requirements.txt
pip install -r requirements.txt

# ค้นหา package
pip search requests  # (บางครั้งไม่ work ใน pip เวอร์ชันใหม่)

# ดาวน์โหลดโดยไม่ติดตั้ง
pip download requests -d ./downloads
```

### 7.2 pip.conf และ options

```bash
# ใช้ mirror (ไทย/เร็วกว่า)
pip install requests -i https://pypi.org/simple/

# ติดตั้งพร้อม dependencies ทั้งหมด
pip install requests[security]  # extras

# user install (ไม่ต้อง sudo)
pip install --user requests

# ดู cache
pip cache list
pip cache purge

# ตรวจสอบ dependencies
pip check
```

---

## 8. Virtual Environments

Virtual environment ช่วยแยก dependencies ของแต่ละ project ออกจากกัน

### 8.1 venv (built-in)

```bash
# สร้าง virtual environment
python -m venv myenv
python3 -m venv myenv

# activate
# Windows:
myenv\Scripts\activate

# Mac/Linux:
source myenv/bin/activate

# เมื่อ activate แล้ว prompt จะมี (myenv)
(myenv) $ pip install requests

# deactivate
deactivate

# ลบ virtual environment
rm -rf myenv  # Mac/Linux
rmdir /s myenv  # Windows
```

### 8.2 venv structure

```
myenv/
├── bin/               # executables (Mac/Linux)
│   ├── python
│   ├── python3
│   ├── pip
│   └── activate
├── Scripts/           # executables (Windows)
│   ├── python.exe
│   ├── pip.exe
│   └── activate.bat
├── lib/               # installed packages
│   └── python3.x/
│       └── site-packages/
│           └── requests/
└── pyvenv.cfg
```

### 8.3 virtualenv (third-party)

```bash
# ติดตั้ง virtualenv
pip install virtualenv

# สร้าง environment
virtualenv myenv
virtualenv -p python3.11 myenv  # ระบุ Python version

# ใช้งานเหมือน venv
source myenv/bin/activate
```

### 8.4 conda

```bash
# สร้าง environment
conda create -n myproject python=3.11

# activate
conda activate myproject

# ติดตั้ง package
conda install numpy pandas

# ดู environments
conda env list

# deactivate
conda deactivate

# ลบ environment
conda env remove -n myproject
```

### 8.5 Python code สำหรับ virtual environment

```python
import sys
import os

def check_virtual_env():
    """ตรวจสอบว่าอยู่ใน virtual environment หรือไม่"""
    
    # วิธีที่ 1: ตรวจสอบ sys.prefix
    in_venv = (sys.prefix != sys.base_prefix)
    
    # วิธีที่ 2: ตรวจสอบ VIRTUAL_ENV environment variable
    venv_path = os.environ.get('VIRTUAL_ENV')
    
    if in_venv:
        print(f"อยู่ใน virtual environment: {sys.prefix}")
    else:
        print("ไม่ได้อยู่ใน virtual environment")
    
    if venv_path:
        print(f"VIRTUAL_ENV: {venv_path}")
    
    print(f"Python executable: {sys.executable}")
    print(f"Python version: {sys.version}")
    
    return in_venv

check_virtual_env()
```

---

## 9. requirements.txt

### 9.1 รูปแบบ requirements.txt

```text
# requirements.txt - ตัวอย่าง

# Web Framework
Flask==2.3.3
Django>=4.2,<5.0

# Database
SQLAlchemy==2.0.21
psycopg2-binary>=2.9.7

# HTTP requests
requests>=2.31.0
httpx[http2]>=0.24.0

# Data processing
numpy>=1.25.0
pandas>=2.0.0

# Testing
pytest>=7.4.0
pytest-cov>=4.1.0

# Development only (อาจแยกไว้ใน requirements-dev.txt)
black>=23.7.0
flake8>=6.0.0
mypy>=1.5.0

# Security
cryptography>=41.0.0

# Utilities
python-dotenv>=1.0.0
pydantic>=2.0.0
```

### 9.2 แยก requirements files

```bash
# requirements/
# ├── base.txt        - production dependencies
# ├── development.txt - dev/test tools
# └── production.txt  - production-specific

# requirements/base.txt
# Flask>=2.3.0
# SQLAlchemy>=2.0.0
# requests>=2.31.0

# requirements/development.txt
# -r base.txt
# pytest>=7.4.0
# black>=23.7.0

# requirements/production.txt  
# -r base.txt
# gunicorn>=21.2.0
# sentry-sdk>=1.30.0
```

### 9.3 จัดการ requirements ด้วย Python

```python
import subprocess
import sys
import pkg_resources

def get_installed_packages():
    """ดู packages ที่ติดตั้งแล้ว"""
    installed = {pkg.key: pkg.version for pkg in pkg_resources.working_set}
    return installed

def check_requirements(requirements_file):
    """ตรวจสอบว่า requirements ครบหรือไม่"""
    missing = []
    outdated = []
    
    installed = get_installed_packages()
    
    try:
        with open(requirements_file) as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith('#'):
                    continue
                
                # แยก package name และ version
                if '>=' in line:
                    name, version = line.split('>=')
                    pkg_name = name.strip().lower()
                    min_version = version.strip()
                    
                    if pkg_name not in installed:
                        missing.append(pkg_name)
                    # สามารถตรวจสอบ version ได้ด้วย
    except FileNotFoundError:
        print(f"ไม่พบไฟล์: {requirements_file}")
        return
    
    if missing:
        print(f"Missing packages: {missing}")
    else:
        print("ทุก package ครบถ้วน!")

def install_requirements(requirements_file):
    """ติดตั้ง requirements"""
    result = subprocess.run(
        [sys.executable, "-m", "pip", "install", "-r", requirements_file],
        capture_output=True,
        text=True
    )
    
    if result.returncode == 0:
        print("ติดตั้ง requirements สำเร็จ")
    else:
        print(f"เกิดข้อผิดพลาด: {result.stderr}")

# ดู installed packages
packages = get_installed_packages()
print(f"Installed packages: {len(packages)}")
for name, version in list(packages.items())[:5]:
    print(f"  {name}: {version}")
```

---

## 10. Popular Third-party Packages

### 10.1 requests - HTTP Library

```python
# pip install requests
import json

# จำลอง requests (ไม่ต้อง install จริง)
class MockResponse:
    def __init__(self, status_code, data):
        self.status_code = status_code
        self._data = data
    
    def json(self):
        return self._data
    
    @property
    def text(self):
        return json.dumps(self._data)
    
    def raise_for_status(self):
        if self.status_code >= 400:
            raise Exception(f"HTTP Error: {self.status_code}")

# Real usage (requires pip install requests):
# import requests
#
# response = requests.get("https://api.github.com/users/python")
# response.raise_for_status()
# data = response.json()
# print(f"Name: {data['name']}")
#
# # POST request
# payload = {"username": "alice", "password": "secret"}
# response = requests.post("https://api.example.com/login", json=payload)
#
# # Session (reuse connections)
# session = requests.Session()
# session.headers.update({"Authorization": "Bearer token123"})
# response = session.get("https://api.example.com/data")

print("requests ใช้สำหรับ HTTP requests")
print("pip install requests")
```

### 10.2 pandas - Data Analysis

```python
# pip install pandas
# จำลองการใช้งาน pandas

# Real usage:
# import pandas as pd
#
# # สร้าง DataFrame
# df = pd.DataFrame({
#     "name": ["Alice", "Bob", "Charlie"],
#     "age": [25, 30, 35],
#     "salary": [50000, 60000, 70000]
# })
#
# # กรองข้อมูล
# young = df[df["age"] < 32]
#
# # สถิติ
# print(df.describe())
# print(df["salary"].mean())
#
# # อ่านไฟล์
# df_csv = pd.read_csv("data.csv")
# df_excel = pd.read_excel("data.xlsx")
#
# # เขียนไฟล์
# df.to_csv("output.csv", index=False)

print("pandas ใช้สำหรับ data analysis และ manipulation")
print("pip install pandas")
```

### 10.3 flask - Web Framework

```python
# pip install flask
# จำลองการใช้งาน Flask

# Real usage:
# from flask import Flask, jsonify, request
#
# app = Flask(__name__)
#
# @app.route("/")
# def home():
#     return "Hello, World!"
#
# @app.route("/api/users", methods=["GET"])
# def get_users():
#     users = [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
#     return jsonify(users)
#
# @app.route("/api/users", methods=["POST"])
# def create_user():
#     data = request.json
#     return jsonify({"id": 3, **data}), 201
#
# if __name__ == "__main__":
#     app.run(debug=True, port=5000)

print("flask ใช้สำหรับสร้าง web applications และ APIs")
print("pip install flask")
```

### 10.4 pytest - Testing

```python
# pip install pytest
# จำลองการทดสอบด้วย pytest

# Real usage (ใน test_calculator.py):
# import pytest
# from calculator import add, subtract, divide
#
# def test_add():
#     assert add(2, 3) == 5
#     assert add(-1, 1) == 0
#     assert add(0, 0) == 0
#
# def test_divide():
#     assert divide(10, 2) == 5.0
#     assert divide(0, 5) == 0.0
#
# def test_divide_by_zero():
#     with pytest.raises(ZeroDivisionError):
#         divide(10, 0)
#
# @pytest.mark.parametrize("a,b,expected", [
#     (2, 3, 5),
#     (0, 0, 0),
#     (-1, 1, 0),
# ])
# def test_add_parametrize(a, b, expected):
#     assert add(a, b) == expected

# รัน pytest:
# pytest test_calculator.py
# pytest -v  # verbose
# pytest --cov=.  # พร้อม coverage

print("pytest ใช้สำหรับ unit testing")
print("pip install pytest pytest-cov")
```

### 10.5 python-dotenv

```python
# pip install python-dotenv
# จำลองการใช้งาน

# .env file:
# DATABASE_URL=postgresql://user:pass@localhost/mydb
# SECRET_KEY=mysecretkey123
# DEBUG=True
# API_KEY=your-api-key-here

# Real usage:
# from dotenv import load_dotenv
# import os
#
# load_dotenv()  # โหลดจากไฟล์ .env
#
# db_url = os.getenv("DATABASE_URL")
# secret_key = os.getenv("SECRET_KEY")
# debug = os.getenv("DEBUG", "False").lower() == "true"
# api_key = os.getenv("API_KEY")

# จำลอง
import os

# จำลองตั้งค่า environment variables
os.environ["APP_NAME"] = "MyApp"
os.environ["APP_DEBUG"] = "False"
os.environ["APP_PORT"] = "8080"

app_name = os.getenv("APP_NAME", "DefaultApp")
debug = os.getenv("APP_DEBUG", "False").lower() == "true"
port = int(os.getenv("APP_PORT", "5000"))

print(f"App: {app_name}")
print(f"Debug: {debug}")
print(f"Port: {port}")
```

---

## 11. ตัวอย่างโปรแกรมจริง

### 11.1 Module System สำหรับ Data Processing

```python
# จำลอง data_processor package

# === data_processor/readers.py ===
import csv
import json

def read_csv_data(content_lines):
    """อ่านข้อมูล CSV จาก list of lines"""
    rows = []
    reader = csv.DictReader(content_lines)
    for row in reader:
        rows.append(dict(row))
    return rows

def read_json_data(json_string):
    """อ่านข้อมูล JSON"""
    return json.loads(json_string)

# === data_processor/transformers.py ===
def clean_numeric(value, default=0):
    """แปลงค่าเป็น float"""
    try:
        return float(value)
    except (ValueError, TypeError):
        return default

def normalize_text(text):
    """ทำความสะอาด text"""
    if not text:
        return ""
    return str(text).strip().lower()

def transform_records(records, transformations):
    """แปลงข้อมูล records ตาม transformations"""
    result = []
    for record in records:
        new_record = {}
        for key, transform_func in transformations.items():
            if key in record:
                new_record[key] = transform_func(record[key])
            else:
                new_record[key] = None
        result.append(new_record)
    return result

# === data_processor/analyzers.py ===
def calculate_stats(numbers):
    """คำนวณสถิติพื้นฐาน"""
    if not numbers:
        return {}
    
    sorted_nums = sorted(numbers)
    n = len(sorted_nums)
    
    stats = {
        "count": n,
        "min": sorted_nums[0],
        "max": sorted_nums[-1],
        "sum": sum(sorted_nums),
        "mean": sum(sorted_nums) / n,
        "median": sorted_nums[n // 2] if n % 2 == 1 
                  else (sorted_nums[n//2-1] + sorted_nums[n//2]) / 2
    }
    
    # Standard deviation
    mean = stats["mean"]
    variance = sum((x - mean) ** 2 for x in sorted_nums) / n
    stats["std"] = variance ** 0.5
    
    return stats

def group_by(records, key):
    """จัดกลุ่มข้อมูล"""
    groups = {}
    for record in records:
        group_key = record.get(key)
        if group_key not in groups:
            groups[group_key] = []
        groups[group_key].append(record)
    return groups

# === data_processor/__init__.py ===
# from .readers import read_csv_data, read_json_data
# from .transformers import transform_records, clean_numeric
# from .analyzers import calculate_stats, group_by

# === main.py ===
# ข้อมูลตัวอย่าง (จำลอง CSV)
sample_data = [
    {"name": "Alice", "dept": "Engineering", "salary": "75000", "age": "28"},
    {"name": "Bob", "dept": "Marketing", "salary": "65000", "age": "32"},
    {"name": "Charlie", "dept": "Engineering", "salary": "80000", "age": "35"},
    {"name": "Diana", "dept": "HR", "salary": "60000", "age": "29"},
    {"name": "Eve", "dept": "Engineering", "salary": "72000", "age": "31"},
    {"name": "Frank", "dept": "Marketing", "salary": "68000", "age": "27"},
]

# Transform data
transformations = {
    "name": normalize_text,
    "dept": normalize_text,
    "salary": clean_numeric,
    "age": lambda x: clean_numeric(x, 0),
}

processed = transform_records(sample_data, transformations)

# Analyze
salaries = [r["salary"] for r in processed]
salary_stats = calculate_stats(salaries)

print("=== สถิติเงินเดือน ===")
for key, value in salary_stats.items():
    if isinstance(value, float):
        print(f"  {key}: {value:,.2f}")
    else:
        print(f"  {key}: {value}")

# Group by department
by_dept = group_by(processed, "dept")
print("\n=== แยกตามแผนก ===")
for dept, members in by_dept.items():
    dept_salaries = [m["salary"] for m in members]
    avg_salary = sum(dept_salaries) / len(dept_salaries)
    print(f"  {dept}: {len(members)} คน, เฉลี่ย {avg_salary:,.0f} บาท")
```

---

## 12. แบบฝึกหัด

### ข้อที่ 1: สร้าง Module สำหรับ String Utils

```python
# เฉลย: string_utils module

def reverse_string(s):
    """กลับสตริง"""
    return s[::-1]

def count_vowels(s):
    """นับจำนวน vowel"""
    vowels = set('aeiouAEIOU')
    return sum(1 for c in s if c in vowels)

def is_palindrome(s):
    """ตรวจสอบว่าเป็น palindrome"""
    cleaned = ''.join(c.lower() for c in s if c.isalnum())
    return cleaned == cleaned[::-1]

def camel_to_snake(name):
    """แปลง camelCase เป็น snake_case"""
    import re
    s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
    return re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()

def snake_to_camel(name):
    """แปลง snake_case เป็น camelCase"""
    components = name.split('_')
    return components[0] + ''.join(x.title() for x in components[1:])

# ทดสอบ
test_strings = ["Hello World", "racecar", "A man a plan a canal Panama"]
for s in test_strings:
    print(f"'{s}' -> reversed: '{reverse_string(s)}'")
    print(f"  vowels: {count_vowels(s)}")
    print(f"  palindrome: {is_palindrome(s)}")

camel_names = ["camelCaseVariable", "myFunctionName", "XMLParser"]
for name in camel_names:
    snake = camel_to_snake(name)
    back = snake_to_camel(snake)
    print(f"\n{name} -> {snake} -> {back}")
```

### ข้อที่ 2: Package สำหรับ Currency Converter

```python
# เฉลย: currency_converter package

class CurrencyConverter:
    """แปลงสกุลเงิน"""
    
    # อัตราแลกเปลี่ยนสมมติ (USD เป็นฐาน)
    RATES = {
        "USD": 1.0,
        "EUR": 0.92,
        "GBP": 0.79,
        "JPY": 149.50,
        "THB": 35.20,
        "SGD": 1.35,
        "CNY": 7.28,
    }
    
    def __init__(self, base_currency="USD"):
        if base_currency not in self.RATES:
            raise ValueError(f"ไม่รองรับสกุลเงิน: {base_currency}")
        self.base_currency = base_currency
    
    def convert(self, amount, from_currency, to_currency):
        """แปลงเงิน"""
        if from_currency not in self.RATES:
            raise ValueError(f"ไม่รองรับ: {from_currency}")
        if to_currency not in self.RATES:
            raise ValueError(f"ไม่รองรับ: {to_currency}")
        
        # แปลงเป็น USD ก่อน แล้วแปลงไป target
        usd_amount = amount / self.RATES[from_currency]
        result = usd_amount * self.RATES[to_currency]
        return round(result, 2)
    
    def get_rate(self, from_currency, to_currency):
        """ดูอัตราแลกเปลี่ยน"""
        return self.RATES[to_currency] / self.RATES[from_currency]
    
    def format_amount(self, amount, currency):
        """จัดรูปแบบตัวเลข"""
        symbols = {
            "USD": "$", "EUR": "€", "GBP": "£",
            "JPY": "¥", "THB": "฿", "SGD": "S$", "CNY": "¥"
        }
        symbol = symbols.get(currency, "")
        return f"{symbol}{amount:,.2f} {currency}"

# ทดสอบ
converter = CurrencyConverter()

# แปลงเงิน
conversions = [
    (1000, "USD", "THB"),
    (5000, "THB", "USD"),
    (100, "EUR", "JPY"),
    (50000, "JPY", "GBP"),
]

print("=== Currency Converter ===")
for amount, from_curr, to_curr in conversions:
    result = converter.convert(amount, from_curr, to_curr)
    rate = converter.get_rate(from_curr, to_curr)
    print(f"{converter.format_amount(amount, from_curr)} = "
          f"{converter.format_amount(result, to_curr)} "
          f"(rate: {rate:.4f})")
```

### ข้อที่ 3-10: แบบฝึกหัดเพิ่มเติม

```python
# ข้อที่ 3: import math module และคำนวณสถิติ
import math
import statistics

data = [4, 8, 15, 16, 23, 42]

print("=== Math Statistics ===")
print(f"Mean: {statistics.mean(data):.2f}")
print(f"Median: {statistics.median(data):.2f}")
print(f"Stdev: {statistics.stdev(data):.2f}")
print(f"Variance: {statistics.variance(data):.2f}")
print(f"Sum: {sum(data)}")
print(f"Max: {max(data)}, Min: {min(data)}")

# ข้อที่ 4: ใช้ itertools สร้าง combinations
from itertools import combinations, permutations, product

items = [1, 2, 3, 4]
print("\n=== Combinations ===")
print(f"C(4,2): {list(combinations(items, 2))}")
print(f"P(4,2): {list(permutations(items, 2))}")

# ข้อที่ 5: ใช้ collections.Counter วิเคราะห์ข้อความ
from collections import Counter

text = """Python is a versatile programming language 
Python is used for web development data science 
machine learning automation and more Python is popular"""

words = text.lower().split()
word_freq = Counter(words)
print("\n=== Word Frequency ===")
for word, count in word_freq.most_common(5):
    print(f"  '{word}': {count} ครั้ง")

# ข้อที่ 6: ใช้ datetime สำหรับ scheduler จำลอง
from datetime import datetime, timedelta

class TaskScheduler:
    def __init__(self):
        self.tasks = []
    
    def schedule(self, task_name, delay_minutes):
        run_time = datetime.now() + timedelta(minutes=delay_minutes)
        self.tasks.append({
            "name": task_name,
            "scheduled_at": run_time,
            "delay_minutes": delay_minutes
        })
    
    def get_pending_tasks(self):
        now = datetime.now()
        return [t for t in self.tasks if t["scheduled_at"] > now]
    
    def show_schedule(self):
        for task in sorted(self.tasks, key=lambda x: x["scheduled_at"]):
            print(f"  {task['name']}: {task['scheduled_at'].strftime('%H:%M:%S')}")

scheduler = TaskScheduler()
scheduler.schedule("Backup database", 5)
scheduler.schedule("Send report", 30)
scheduler.schedule("Cleanup temp files", 60)

print("\n=== Task Schedule ===")
scheduler.show_schedule()
print(f"Pending tasks: {len(scheduler.get_pending_tasks())}")

# ข้อที่ 7: สร้าง plugin system อย่างง่าย
class PluginManager:
    def __init__(self):
        self.plugins = {}
    
    def register(self, name):
        """Decorator สำหรับ register plugin"""
        def decorator(cls):
            self.plugins[name] = cls
            print(f"Registered plugin: {name}")
            return cls
        return decorator
    
    def get_plugin(self, name):
        if name not in self.plugins:
            raise KeyError(f"ไม่พบ plugin: {name}")
        return self.plugins[name]
    
    def list_plugins(self):
        return list(self.plugins.keys())

plugin_manager = PluginManager()

@plugin_manager.register("csv_processor")
class CSVProcessor:
    def process(self, data):
        return f"Processing CSV: {len(data)} records"

@plugin_manager.register("json_processor")  
class JSONProcessor:
    def process(self, data):
        return f"Processing JSON: {len(data)} items"

print("\n=== Plugin System ===")
print(f"Registered plugins: {plugin_manager.list_plugins()}")

csv_plugin = plugin_manager.get_plugin("csv_processor")
result = csv_plugin().process([1, 2, 3])
print(f"Result: {result}")

# ข้อที่ 8: Config Manager ด้วย __init__.py pattern
import os
import json

class ConfigManager:
    """จัดการ configuration จากหลายแหล่ง"""
    
    def __init__(self):
        self._config = {}
    
    def load_defaults(self, defaults):
        """โหลด default values"""
        self._config.update(defaults)
        return self
    
    def load_from_env(self, prefix="APP_"):
        """โหลดจาก environment variables"""
        for key, value in os.environ.items():
            if key.startswith(prefix):
                config_key = key[len(prefix):].lower()
                self._config[config_key] = value
        return self
    
    def get(self, key, default=None):
        return self._config.get(key, default)
    
    def set(self, key, value):
        self._config[key] = value
    
    def __repr__(self):
        return f"ConfigManager({self._config})"

config = ConfigManager()
config.load_defaults({
    "host": "localhost",
    "port": 8080,
    "debug": False,
    "database": "sqlite:///app.db"
})
config.load_from_env("APP_")

# ตั้งค่า test env vars
os.environ["APP_HOST"] = "production.server.com"
os.environ["APP_PORT"] = "443"
config.load_from_env("APP_")

print("\n=== Config Manager ===")
print(f"Host: {config.get('host')}")
print(f"Port: {config.get('port')}")
print(f"Debug: {config.get('debug')}")
print(f"DB: {config.get('database')}")
print(f"Non-existent: {config.get('redis_url', 'not configured')}")
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|--------------|
| import | import, from...import, import...as |
| สร้าง module | การสร้างและจัดระเบียบโค้ดใน module |
| \_\_name\_\_ | ใช้แยก script vs import |
| Package | สร้าง package ด้วย \_\_init\_\_.py |
| Relative imports | การ import ภายใน package |
| Standard Library | os, pathlib, datetime, collections, itertools, json, re |
| pip | การจัดการ packages |
| Virtual Environments | การแยก environments |
| requirements.txt | การกำหนด dependencies |
| Third-party | requests, pandas, flask, pytest |

### ขั้นต่อไป

ในบทถัดไป (Part 18) เราจะเรียนรู้เกี่ยวกับ **Comprehensions** ซึ่งเป็นวิธีเขียนโค้ดแบบ Pythonic ที่กระชับและมีประสิทธิภาพ
