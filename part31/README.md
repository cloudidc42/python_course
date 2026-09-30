# Part 31: JSON, CSV & Data Formats

## บทนำ

ในการพัฒนาซอฟต์แวร์จริง เราต้องทำงานกับข้อมูลในรูปแบบต่างๆ อยู่เสมอ ไม่ว่าจะเป็น JSON จาก REST API, CSV จากไฟล์ Excel, XML จากระบบเก่า หรือ YAML จาก configuration files การเข้าใจวิธีอ่าน เขียน และแปลงข้อมูลเหล่านี้เป็นทักษะสำคัญสำหรับ Python developer ทุกคน

---

## 1. JSON Format และ Python

### JSON คืออะไร?

JSON (JavaScript Object Notation) คือรูปแบบข้อมูลที่ใช้กันแพร่หลายที่สุดในการแลกเปลี่ยนข้อมูลผ่าน web APIs มีลักษณะเป็น key-value pairs คล้ายกับ Python dictionary

```json
{
    "name": "สมชาย",
    "age": 25,
    "hobbies": ["coding", "reading", "gaming"],
    "address": {
        "city": "กรุงเทพ",
        "country": "Thailand"
    },
    "is_active": true,
    "score": null
}
```

### การ Map ระหว่าง JSON และ Python

| JSON Type | Python Type |
|-----------|-------------|
| object    | dict        |
| array     | list        |
| string    | str         |
| number (int) | int      |
| number (float) | float  |
| true      | True        |
| false     | False       |
| null      | None        |

---

## 2. json.loads() และ json.dumps()

### json.loads() - แปลง JSON string เป็น Python object

```python
import json

# ตัวอย่างที่ 1: แปลง JSON string เป็น dict
json_string = '{"name": "Alice", "age": 30, "city": "Bangkok"}'
data = json.loads(json_string)

print(type(data))       # <class 'dict'>
print(data['name'])     # Alice
print(data['age'])      # 30
```

```python
import json

# ตัวอย่างที่ 2: แปลง JSON array เป็น list
json_array = '[1, 2, 3, "hello", true, null]'
result = json.loads(json_array)

print(type(result))     # <class 'list'>
print(result)           # [1, 2, 3, 'hello', True, None]
```

```python
import json

# ตัวอย่างที่ 3: JSON ซ้อนกัน (Nested JSON)
nested_json = '''
{
    "company": "TechCorp",
    "employees": [
        {"name": "Bob", "role": "developer"},
        {"name": "Alice", "role": "designer"}
    ],
    "founded": 2020
}
'''
data = json.loads(nested_json)
print(data['company'])                    # TechCorp
print(data['employees'][0]['name'])       # Bob
print(len(data['employees']))             # 2
```

### json.dumps() - แปลง Python object เป็น JSON string

```python
import json

# ตัวอย่างที่ 4: แปลง dict เป็น JSON string
person = {
    "name": "สมชาย",
    "age": 25,
    "hobbies": ["coding", "music"]
}

json_str = json.dumps(person)
print(json_str)
# {"name": "สมชาย", "age": 25, "hobbies": ["coding", "music"]}
```

```python
import json

# ตัวอย่างที่ 5: json.dumps() พร้อม options
person = {
    "name": "สมชาย",
    "age": 25,
    "hobbies": ["coding", "music"]
}

# ensure_ascii=False เพื่อให้แสดงภาษาไทยได้
# indent=2 สำหรับ pretty printing
json_str = json.dumps(person, ensure_ascii=False, indent=2)
print(json_str)
```

ผลลัพธ์:
```json
{
  "name": "สมชาย",
  "age": 25,
  "hobbies": [
    "coding",
    "music"
  ]
}
```

```python
import json

# ตัวอย่างที่ 6: sort_keys และ separators
data = {"z": 1, "a": 2, "m": 3}

# sort_keys=True เรียง key ตาม alphabet
json_str = json.dumps(data, sort_keys=True, separators=(',', ':'))
print(json_str)  # {"a":2,"m":3,"z":1}
```

---

## 3. json.load() และ json.dump() - อ่านจาก/เขียนลงไฟล์

```python
import json

# ตัวอย่างที่ 7: เขียน JSON ลงไฟล์ด้วย json.dump()
config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "mydb"
    },
    "debug": True,
    "allowed_hosts": ["localhost", "127.0.0.1"]
}

with open('config.json', 'w', encoding='utf-8') as f:
    json.dump(config, f, ensure_ascii=False, indent=4)

print("บันทึกไฟล์สำเร็จ!")
```

```python
import json

# ตัวอย่างที่ 8: อ่าน JSON จากไฟล์ด้วย json.load()
with open('config.json', 'r', encoding='utf-8') as f:
    config = json.load(f)

print(config['database']['host'])     # localhost
print(config['database']['port'])     # 5432
print(config['allowed_hosts'])        # ['localhost', '127.0.0.1']
```

```python
import json
import os

# ตัวอย่างที่ 9: อ่านและอัพเดท JSON file
def update_json_file(filepath, updates):
    """อัพเดทข้อมูลใน JSON file"""
    # อ่านข้อมูลเดิม
    if os.path.exists(filepath):
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
    else:
        data = {}
    
    # อัพเดทข้อมูล
    data.update(updates)
    
    # เขียนกลับ
    with open(filepath, 'w', encoding='utf-8') as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    
    return data

# ใช้งาน
result = update_json_file('settings.json', {'theme': 'dark', 'language': 'th'})
print(result)
```

---

## 4. JSON Encoding/Decoding ของ Custom Objects

โดยปกติ json module ไม่รู้จัก custom class แต่เราสามารถสอนได้ด้วย 2 วิธี

### วิธีที่ 1: Custom JSONEncoder

```python
import json
from datetime import datetime, date

# ตัวอย่างที่ 10: Custom JSONEncoder สำหรับ datetime
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, date):
            return obj.isoformat()
        return super().default(obj)

# ใช้งาน
data = {
    "event": "Python Workshop",
    "start_date": date(2024, 1, 15),
    "created_at": datetime.now(),
    "participants": 50
}

json_str = json.dumps(data, cls=DateTimeEncoder, ensure_ascii=False, indent=2)
print(json_str)
```

```python
import json
from dataclasses import dataclass, asdict
from typing import List

# ตัวอย่างที่ 11: Custom Encoder สำหรับ dataclass
@dataclass
class Student:
    name: str
    age: int
    grades: List[float]
    
    def average_grade(self):
        return sum(self.grades) / len(self.grades)

class StudentEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Student):
            return {
                "name": obj.name,
                "age": obj.age,
                "grades": obj.grades,
                "average": round(obj.average_grade(), 2)
            }
        return super().default(obj)

students = [
    Student("Alice", 20, [85.0, 90.0, 78.5]),
    Student("Bob", 22, [70.0, 65.5, 80.0])
]

json_str = json.dumps(students, cls=StudentEncoder, indent=2)
print(json_str)
```

### วิธีที่ 2: object_hook สำหรับ decode

```python
import json

# ตัวอย่างที่ 12: object_hook สำหรับ decode JSON เป็น object
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"

def point_decoder(dct):
    """แปลง dict เป็น Point object ถ้ามี key x และ y"""
    if 'x' in dct and 'y' in dct:
        return Point(dct['x'], dct['y'])
    return dct

json_str = '{"x": 10, "y": 20}'
point = json.loads(json_str, object_hook=point_decoder)
print(point)          # Point(x=10, y=20)
print(type(point))    # <class '__main__.Point'>
```

---

## 5. JSON with Datetime Handling

```python
import json
from datetime import datetime, date, timezone
import re

# ตัวอย่างที่ 13: Encoder และ Decoder สำหรับ datetime
class DateTimeHandler:
    
    @staticmethod
    def encode(obj):
        """สำหรับใช้กับ default parameter ใน json.dumps"""
        if isinstance(obj, datetime):
            return {"__type__": "datetime", "value": obj.isoformat()}
        if isinstance(obj, date):
            return {"__type__": "date", "value": obj.isoformat()}
        raise TypeError(f"Object of type {type(obj)} is not JSON serializable")
    
    @staticmethod
    def decode(dct):
        """สำหรับใช้กับ object_hook parameter ใน json.loads"""
        if "__type__" in dct:
            if dct["__type__"] == "datetime":
                return datetime.fromisoformat(dct["value"])
            if dct["__type__"] == "date":
                return date.fromisoformat(dct["value"])
        return dct

# ใช้งาน
data = {
    "name": "Meeting",
    "date": date(2024, 6, 15),
    "created_at": datetime(2024, 6, 1, 10, 30, 0)
}

# Encode
encoded = json.dumps(data, default=DateTimeHandler.encode, indent=2)
print("Encoded:", encoded)

# Decode
decoded = json.loads(encoded, object_hook=DateTimeHandler.decode)
print("Decoded date type:", type(decoded['date']))     # <class 'datetime.date'>
print("Decoded datetime type:", type(decoded['created_at']))  # <class 'datetime.datetime'>
```

```python
import json
from datetime import datetime

# ตัวอย่างที่ 14: Simple datetime handling แบบ string
def json_serial(obj):
    """JSON serializer สำหรับ objects ที่ไม่ใช่ serializable โดยปกติ"""
    if isinstance(obj, datetime):
        return obj.strftime('%Y-%m-%d %H:%M:%S')
    raise TypeError(f"Type {type(obj)} not serializable")

event_log = {
    "events": [
        {"action": "login", "timestamp": datetime(2024, 1, 15, 9, 0, 0)},
        {"action": "purchase", "timestamp": datetime(2024, 1, 15, 9, 15, 30)},
        {"action": "logout", "timestamp": datetime(2024, 1, 15, 10, 0, 0)}
    ]
}

json_str = json.dumps(event_log, default=json_serial, indent=2)
print(json_str)
```

---

## 6. CSV Format และ Python

### CSV คืออะไร?

CSV (Comma-Separated Values) คือรูปแบบข้อมูลที่เก็บข้อมูลแบบตาราง โดยแต่ละแถวคือ record และแต่ละ column คั่นด้วย comma (หรือ delimiter อื่น)

```
name,age,city,salary
Alice,30,Bangkok,50000
Bob,25,Chiang Mai,45000
Charlie,35,Phuket,60000
```

---

## 7. csv.reader และ csv.writer

```python
import csv

# ตัวอย่างที่ 15: เขียนข้อมูลลง CSV ด้วย csv.writer
employees = [
    ['Alice', 30, 'Bangkok', 50000],
    ['Bob', 25, 'Chiang Mai', 45000],
    ['Charlie', 35, 'Phuket', 60000]
]

with open('employees.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    
    # เขียน header
    writer.writerow(['Name', 'Age', 'City', 'Salary'])
    
    # เขียนข้อมูล
    writer.writerows(employees)

print("บันทึก CSV สำเร็จ!")
```

```python
import csv

# ตัวอย่างที่ 16: อ่าน CSV ด้วย csv.reader
with open('employees.csv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f)
    
    # อ่าน header
    headers = next(reader)
    print("Headers:", headers)
    
    # อ่านข้อมูล
    for row in reader:
        print(f"Name: {row[0]}, Age: {row[1]}, City: {row[2]}, Salary: {row[3]}")
```

```python
import csv

# ตัวอย่างที่ 17: csv.reader พร้อมประมวลผลข้อมูล
def calculate_average_salary(filename):
    """คำนวณเงินเดือนเฉลี่ยจากไฟล์ CSV"""
    salaries = []
    
    with open(filename, 'r', encoding='utf-8') as f:
        reader = csv.reader(f)
        next(reader)  # ข้าม header
        
        for row in reader:
            try:
                salary = float(row[3])
                salaries.append(salary)
            except (ValueError, IndexError):
                continue
    
    if salaries:
        return sum(salaries) / len(salaries)
    return 0

avg = calculate_average_salary('employees.csv')
print(f"เงินเดือนเฉลี่ย: {avg:,.2f} บาท")
```

---

## 8. csv.DictReader และ csv.DictWriter

DictReader และ DictWriter ทำให้ทำงานกับ CSV สะดวกขึ้น โดยใช้ column names แทน index

```python
import csv

# ตัวอย่างที่ 18: เขียน CSV ด้วย DictWriter
students = [
    {'name': 'Alice', 'score': 85, 'grade': 'A'},
    {'name': 'Bob', 'score': 72, 'grade': 'B'},
    {'name': 'Charlie', 'score': 65, 'grade': 'C'},
    {'name': 'Diana', 'score': 91, 'grade': 'A+'}
]

fieldnames = ['name', 'score', 'grade']

with open('students.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    
    writer.writeheader()      # เขียน header อัตโนมัติ
    writer.writerows(students) # เขียนข้อมูลทั้งหมด

print("บันทึกสำเร็จ!")
```

```python
import csv

# ตัวอย่างที่ 19: อ่าน CSV ด้วย DictReader
with open('students.csv', 'r', encoding='utf-8') as f:
    reader = csv.DictReader(f)
    
    for row in reader:
        # เข้าถึงข้อมูลด้วย column name
        print(f"{row['name']}: {row['score']} คะแนน (เกรด {row['grade']})")
```

```python
import csv

# ตัวอย่างที่ 20: อ่าน CSV เป็น list of dicts
def load_csv_as_dicts(filename):
    """โหลด CSV ทั้งหมดเป็น list of dictionaries"""
    with open(filename, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        return list(reader)

# ใช้งาน
students = load_csv_as_dicts('students.csv')
print(f"จำนวนนักเรียน: {len(students)}")

# หานักเรียนที่ได้คะแนนสูงสุด
top_student = max(students, key=lambda x: float(x['score']))
print(f"คะแนนสูงสุด: {top_student['name']} ({top_student['score']} คะแนน)")
```

---

## 9. CSV Handling: Headers, Encoding, Delimiters

```python
import csv

# ตัวอย่างที่ 21: CSV ด้วย delimiter อื่น (tab-separated)
data = [
    ['Product', 'Price', 'Stock'],
    ['iPhone 15', '35000', '50'],
    ['Samsung S24', '32000', '30'],
    ['MacBook Pro', '89000', '15']
]

# เขียน TSV (Tab-Separated Values)
with open('products.tsv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f, delimiter='\t')
    writer.writerows(data)

# อ่าน TSV
with open('products.tsv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f, delimiter='\t')
    for row in reader:
        print(row)
```

```python
import csv

# ตัวอย่างที่ 22: จัดการข้อมูลที่มีเครื่องหมาย comma ใน field
# quoting ช่วยให้จัดการข้อมูลที่มี comma ได้
products = [
    {'name': 'MacBook Pro, 16 inch', 'price': 89000, 'description': 'The "best" laptop'},
    {'name': 'iPhone 15', 'price': 35000, 'description': 'Latest Apple phone'}
]

with open('products.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(
        f,
        fieldnames=['name', 'price', 'description'],
        quoting=csv.QUOTE_ALL  # Quote ทุก field
    )
    writer.writeheader()
    writer.writerows(products)

# อ่านกลับ
with open('products.csv', 'r', encoding='utf-8') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], '-', row['description'])
```

```python
import csv
import codecs

# ตัวอย่างที่ 23: การจัดการ encoding ที่หลากหลาย
def read_csv_any_encoding(filename, encodings=('utf-8', 'utf-8-sig', 'tis-620', 'cp874')):
    """ลองอ่าน CSV ด้วย encoding ต่างๆ"""
    for encoding in encodings:
        try:
            with open(filename, 'r', encoding=encoding) as f:
                reader = csv.DictReader(f)
                data = list(reader)
            print(f"อ่านสำเร็จด้วย encoding: {encoding}")
            return data
        except UnicodeDecodeError:
            continue
    raise ValueError("ไม่สามารถอ่านไฟล์ได้ด้วย encoding ที่รองรับ")

# เขียนไฟล์ภาษาไทยด้วย utf-8-sig (สำหรับ Excel)
thai_data = [
    {'ชื่อ': 'สมชาย', 'อายุ': '25', 'เมือง': 'กรุงเทพ'},
    {'ชื่อ': 'สมหญิง', 'อายุ': '28', 'เมือง': 'เชียงใหม่'}
]

with open('thai_data.csv', 'w', newline='', encoding='utf-8-sig') as f:
    writer = csv.DictWriter(f, fieldnames=['ชื่อ', 'อายุ', 'เมือง'])
    writer.writeheader()
    writer.writerows(thai_data)

print("บันทึกข้อมูลภาษาไทยสำเร็จ!")
```

```python
import csv
from io import StringIO

# ตัวอย่างที่ 24: อ่าน CSV จาก string (ไม่จากไฟล์)
csv_string = """name,age,score
Alice,20,85
Bob,22,72
Charlie,21,91"""

reader = csv.DictReader(StringIO(csv_string))
for row in reader:
    print(f"{row['name']}: {row['score']}")
```

---

## 10. Excel Files (openpyxl เบื้องต้น)

openpyxl ช่วยให้เราอ่านและเขียนไฟล์ Excel (.xlsx) ได้โดยตรง

```python
# pip install openpyxl

from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.utils import get_column_letter

# ตัวอย่างที่ 25: สร้าง Excel file ด้วย openpyxl
wb = Workbook()
ws = wb.active
ws.title = "Sales Report"

# กำหนด headers
headers = ['Product', 'Q1', 'Q2', 'Q3', 'Q4', 'Total']
for col, header in enumerate(headers, 1):
    cell = ws.cell(row=1, column=col, value=header)
    cell.font = Font(bold=True, color="FFFFFF")
    cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    cell.alignment = Alignment(horizontal="center")

# ใส่ข้อมูล
products = [
    ['iPhone', 100, 120, 90, 150],
    ['iPad', 50, 60, 70, 80],
    ['MacBook', 30, 35, 40, 45],
]

for row_idx, product in enumerate(products, 2):
    for col_idx, value in enumerate(product, 1):
        ws.cell(row=row_idx, column=col_idx, value=value)
    # เพิ่ม formula สำหรับ Total
    ws.cell(row=row_idx, column=6, value=f"=SUM(B{row_idx}:E{row_idx})")

# ปรับ column width
for col in range(1, 7):
    ws.column_dimensions[get_column_letter(col)].width = 12

# บันทึกไฟล์
wb.save('sales_report.xlsx')
print("สร้าง Excel file สำเร็จ!")
```

```python
from openpyxl import load_workbook

# ตัวอย่างที่ 26: อ่าน Excel file
wb = load_workbook('sales_report.xlsx', data_only=True)
ws = wb.active

print(f"Sheet name: {ws.title}")
print(f"Max row: {ws.max_row}")
print(f"Max column: {ws.max_column}")
print()

# อ่านข้อมูลทั้งหมด
for row in ws.iter_rows(values_only=True):
    print(row)
```

---

## 11. YAML Format (PyYAML)

YAML ใช้กันมากสำหรับ configuration files เช่น Docker Compose, Kubernetes, GitHub Actions

```yaml
# ตัวอย่าง YAML
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret123

services:
  - name: web
    port: 8000
    debug: true
  - name: worker
    port: null
    debug: false
```

```python
# pip install pyyaml
import yaml

# ตัวอย่างที่ 27: อ่าน YAML
yaml_config = """
database:
  host: localhost
  port: 5432
  name: myapp_db

cache:
  backend: redis
  timeout: 300
  
features:
  - authentication
  - authorization  
  - logging
"""

config = yaml.safe_load(yaml_config)

print(config['database']['host'])    # localhost
print(config['database']['port'])    # 5432
print(config['features'])            # ['authentication', 'authorization', 'logging']
print(type(config['database']['port']))  # <class 'int'>
```

```python
import yaml

# ตัวอย่างที่ 28: เขียน Python object เป็น YAML
data = {
    'app': {
        'name': 'MyWebApp',
        'version': '1.0.0',
        'debug': False,
        'allowed_hosts': ['localhost', '127.0.0.1', 'myapp.com']
    },
    'database': {
        'engine': 'postgresql',
        'host': 'db.example.com',
        'port': 5432
    }
}

yaml_str = yaml.dump(data, default_flow_style=False, allow_unicode=True, indent=2)
print(yaml_str)

# บันทึกลงไฟล์
with open('config.yaml', 'w', encoding='utf-8') as f:
    yaml.dump(data, f, default_flow_style=False, allow_unicode=True, indent=2)
```

```python
import yaml

# ตัวอย่างที่ 29: อ่าน YAML file และ validate
def load_config(filepath):
    """โหลด YAML config พร้อม error handling"""
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            config = yaml.safe_load(f)
        
        # ตรวจสอบ required keys
        required = ['database', 'app']
        missing = [key for key in required if key not in config]
        
        if missing:
            raise ValueError(f"Missing required config keys: {missing}")
        
        return config
        
    except yaml.YAMLError as e:
        raise ValueError(f"Invalid YAML format: {e}")
    except FileNotFoundError:
        raise FileNotFoundError(f"Config file not found: {filepath}")

# อ่าน YAML ที่มีหลาย documents
multi_doc_yaml = """
---
name: Development
debug: true
---
name: Production
debug: false
"""

docs = list(yaml.safe_load_all(multi_doc_yaml))
for doc in docs:
    print(doc)
```

---

## 12. TOML Format

TOML ถูกออกแบบมาให้อ่านง่ายและเข้าใจง่าย ใช้ใน Python projects (pyproject.toml, Cargo.toml)

```python
# Python 3.11+ มี tomllib ใน standard library
# สำหรับเวอร์ชันเก่าใช้: pip install tomli
import tomllib  # หรือ import tomli as tomllib

# ตัวอย่างที่ 30: อ่าน TOML
toml_content = b"""
[project]
name = "my-package"
version = "1.0.0"
requires-python = ">=3.9"

[project.dependencies]
requests = ">=2.28.0"
pydantic = ">=2.0.0"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]

[database]
host = "localhost"
port = 5432
name = "mydb"
"""

config = tomllib.loads(toml_content.decode())
print(config['project']['name'])              # my-package
print(config['project']['version'])           # 1.0.0
print(config['database']['host'])             # localhost
```

```python
# สำหรับการเขียน TOML ใช้: pip install tomli-w
import tomli_w

# ตัวอย่างที่ 31: เขียน TOML
config = {
    "project": {
        "name": "awesome-app",
        "version": "2.0.0",
        "authors": ["Alice <alice@example.com>"]
    },
    "build-system": {
        "requires": ["setuptools>=61.0"],
        "build-backend": "setuptools.build_meta"
    }
}

toml_str = tomli_w.dumps(config)
print(toml_str)

with open('pyproject.toml', 'wb') as f:
    tomli_w.dump(config, f)
```

---

## 13. XML Parsing (xml.etree.ElementTree)

XML ใช้กันมากในระบบเก่าและ enterprise systems

```python
import xml.etree.ElementTree as ET

# ตัวอย่างที่ 32: parse XML string
xml_content = """
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
    <book category="fiction">
        <title>The Python Bible</title>
        <author>John Doe</author>
        <price currency="THB">350.00</price>
        <year>2024</year>
    </book>
    <book category="technical">
        <title>Clean Code</title>
        <author>Robert C. Martin</author>
        <price currency="THB">450.00</price>
        <year>2008</year>
    </book>
</bookstore>
"""

root = ET.fromstring(xml_content)

print(f"Root tag: {root.tag}")
print(f"Number of books: {len(root)}")

# วนลูปอ่านข้อมูล
for book in root.findall('book'):
    title = book.find('title').text
    author = book.find('author').text
    price = book.find('price').text
    category = book.get('category')
    
    print(f"\n{title}")
    print(f"  Author: {author}")
    print(f"  Category: {category}")
    print(f"  Price: {price} บาท")
```

```python
import xml.etree.ElementTree as ET

# ตัวอย่างที่ 33: สร้าง XML และบันทึกไฟล์
root = ET.Element('students')

# เพิ่ม students
student_data = [
    ('Alice', '20', 'CS', '3.8'),
    ('Bob', '22', 'EE', '3.5'),
    ('Charlie', '21', 'ME', '3.6')
]

for name, age, dept, gpa in student_data:
    student = ET.SubElement(root, 'student')
    student.set('id', name.lower())
    
    name_elem = ET.SubElement(student, 'name')
    name_elem.text = name
    
    age_elem = ET.SubElement(student, 'age')
    age_elem.text = age
    
    dept_elem = ET.SubElement(student, 'department')
    dept_elem.text = dept
    
    gpa_elem = ET.SubElement(student, 'gpa')
    gpa_elem.text = gpa

# สร้าง tree และ indent (Python 3.9+)
tree = ET.ElementTree(root)
ET.indent(tree, space='  ')

# บันทึกไฟล์
tree.write('students.xml', encoding='unicode', xml_declaration=True)

# แสดงผล
print(ET.tostring(root, encoding='unicode'))
```

```python
import xml.etree.ElementTree as ET

# ตัวอย่างที่ 34: XPath queries ใน ElementTree
xml_content = """
<catalog>
    <product id="1" category="electronics">
        <name>Laptop</name>
        <price>45000</price>
        <stock>10</stock>
    </product>
    <product id="2" category="electronics">
        <name>Phone</name>
        <price>25000</price>
        <stock>50</stock>
    </product>
    <product id="3" category="books">
        <name>Python Book</name>
        <price>500</price>
        <stock>100</stock>
    </product>
</catalog>
"""

root = ET.fromstring(xml_content)

# หา electronics ทั้งหมด
electronics = root.findall(".//product[@category='electronics']")
print("Electronics:")
for item in electronics:
    print(f"  {item.find('name').text}: {item.find('price').text} บาท")

# หาสินค้าที่มีราคาแพง (ใช้ findall + filter)
all_products = root.findall('.//product')
expensive = [p for p in all_products if float(p.find('price').text) > 10000]

print("\nสินค้าราคาเกิน 10,000 บาท:")
for p in expensive:
    print(f"  {p.find('name').text}")
```

---

## 14. Data Serialization Patterns

```python
import json
import pickle
import struct

# ตัวอย่างที่ 35: เปรียบเทียบ serialization formats
import time

data = {
    "users": [{"id": i, "name": f"User{i}", "score": i * 1.5} for i in range(1000)]
}

# JSON serialization
start = time.time()
json_bytes = json.dumps(data).encode('utf-8')
json_time = time.time() - start
json_size = len(json_bytes)

# Pickle serialization (ไม่ใช้กับ untrusted data!)
start = time.time()
pickle_bytes = pickle.dumps(data)
pickle_time = time.time() - start
pickle_size = len(pickle_bytes)

print(f"JSON:   {json_size:,} bytes, {json_time*1000:.2f}ms")
print(f"Pickle: {pickle_size:,} bytes, {pickle_time*1000:.2f}ms")
```

```python
import json

# Pattern: Repository pattern กับ JSON storage
class JSONRepository:
    """Simple JSON-based data storage"""
    
    def __init__(self, filepath):
        self.filepath = filepath
        self._data = self._load()
    
    def _load(self):
        try:
            with open(self.filepath, 'r', encoding='utf-8') as f:
                return json.load(f)
        except FileNotFoundError:
            return {}
    
    def _save(self):
        with open(self.filepath, 'w', encoding='utf-8') as f:
            json.dump(self._data, f, ensure_ascii=False, indent=2)
    
    def get(self, key, default=None):
        return self._data.get(key, default)
    
    def set(self, key, value):
        self._data[key] = value
        self._save()
    
    def delete(self, key):
        if key in self._data:
            del self._data[key]
            self._save()
    
    def all(self):
        return dict(self._data)

# ใช้งาน
repo = JSONRepository('data_store.json')
repo.set('config', {'theme': 'dark', 'lang': 'th'})
repo.set('user', {'name': 'Alice', 'role': 'admin'})
print(repo.get('config'))     # {'theme': 'dark', 'lang': 'th'}
print(repo.all())
```

---

## แบบฝึกหัด

### ข้อที่ 1: JSON API Response Handler
สร้างโปรแกรมจัดการ API responses ที่:
- แปลง JSON response เป็น Python objects
- จัดการ errors และ null values
- Cache responses ลง local file

**เฉลย:**
```python
import json
import os
from datetime import datetime, timedelta

class APIResponseCache:
    def __init__(self, cache_dir='api_cache', ttl_minutes=60):
        self.cache_dir = cache_dir
        self.ttl = timedelta(minutes=ttl_minutes)
        os.makedirs(cache_dir, exist_ok=True)
    
    def _cache_key(self, url):
        import hashlib
        return hashlib.md5(url.encode()).hexdigest() + '.json'
    
    def _cache_path(self, url):
        return os.path.join(self.cache_dir, self._cache_key(url))
    
    def get(self, url):
        path = self._cache_path(url)
        if not os.path.exists(path):
            return None
        
        with open(path, 'r') as f:
            cached = json.load(f)
        
        cached_time = datetime.fromisoformat(cached['timestamp'])
        if datetime.now() - cached_time > self.ttl:
            os.remove(path)
            return None
        
        return cached['data']
    
    def set(self, url, data):
        path = self._cache_path(url)
        cache_entry = {
            'url': url,
            'timestamp': datetime.now().isoformat(),
            'data': data
        }
        with open(path, 'w') as f:
            json.dump(cache_entry, f, ensure_ascii=False, indent=2)
    
    def parse_response(self, response_text):
        """Parse JSON response พร้อม error handling"""
        try:
            data = json.loads(response_text)
            return {'success': True, 'data': data, 'error': None}
        except json.JSONDecodeError as e:
            return {'success': False, 'data': None, 'error': str(e)}

# ใช้งาน
cache = APIResponseCache(ttl_minutes=30)
response = '{"users": [{"id": 1, "name": "Alice"}], "total": 1}'
result = cache.parse_response(response)
if result['success']:
    cache.set('https://api.example.com/users', result['data'])
    print("Cached:", cache.get('https://api.example.com/users'))
```

---

### ข้อที่ 2: CSV Data Processor
สร้างโปรแกรมที่:
- อ่านไฟล์ CSV ข้อมูลยอดขาย
- คำนวณสถิติต่างๆ (sum, average, max, min)
- export ผลลัพธ์เป็น JSON

**เฉลย:**
```python
import csv
import json
from collections import defaultdict

# สร้างข้อมูลตัวอย่าง
sample_data = """product,category,quantity,price,date
iPhone 15,Electronics,5,35000,2024-01-15
iPad Pro,Electronics,3,32000,2024-01-15
Python Book,Books,10,500,2024-01-16
Clean Code,Books,8,450,2024-01-16
MacBook Pro,Electronics,2,89000,2024-01-17
Data Science Book,Books,6,600,2024-01-17
"""

with open('sales.csv', 'w', encoding='utf-8') as f:
    f.write(sample_data)

def analyze_sales(filename):
    sales_by_category = defaultdict(list)
    
    with open(filename, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            category = row['category']
            amount = int(row['quantity']) * float(row['price'])
            sales_by_category[category].append({
                'product': row['product'],
                'amount': amount,
                'date': row['date']
            })
    
    results = {}
    for category, items in sales_by_category.items():
        amounts = [item['amount'] for item in items]
        results[category] = {
            'total_sales': sum(amounts),
            'average_sale': sum(amounts) / len(amounts),
            'max_sale': max(amounts),
            'min_sale': min(amounts),
            'transaction_count': len(items),
            'items': items
        }
    
    return results

analysis = analyze_sales('sales.csv')
print(json.dumps(analysis, ensure_ascii=False, indent=2))
```

---

### ข้อที่ 3: YAML Config Manager
สร้าง Config Manager ที่:
- อ่าน YAML config หลายระดับ (development, staging, production)
- merge configs
- validate required fields

**เฉลย:**
```python
import yaml
from typing import Dict, Any

def deep_merge(base: dict, override: dict) -> dict:
    """Merge two dicts recursively"""
    result = dict(base)
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

class ConfigManager:
    BASE_CONFIG = """
database:
  host: localhost
  port: 5432
  pool_size: 5
app:
  debug: false
  log_level: INFO
  secret_key: CHANGE_ME
"""
    
    ENV_CONFIGS = {
        'development': """
database:
  name: myapp_dev
app:
  debug: true
  log_level: DEBUG
""",
        'production': """
database:
  host: prod-db.example.com
  name: myapp_prod
  pool_size: 20
app:
  log_level: WARNING
"""
    }
    
    REQUIRED_FIELDS = ['database.host', 'database.name', 'app.secret_key']
    
    def __init__(self, environment='development'):
        self.environment = environment
        self.config = self._build_config()
    
    def _build_config(self):
        base = yaml.safe_load(self.BASE_CONFIG)
        env_override = yaml.safe_load(
            self.ENV_CONFIGS.get(self.environment, '{}')
        )
        return deep_merge(base, env_override)
    
    def get(self, path: str, default=None):
        """Get config value ด้วย dot notation"""
        keys = path.split('.')
        value = self.config
        for key in keys:
            if isinstance(value, dict) and key in value:
                value = value[key]
            else:
                return default
        return value
    
    def validate(self):
        missing = []
        for field in self.REQUIRED_FIELDS:
            if self.get(field) is None:
                missing.append(field)
        if missing:
            raise ValueError(f"Missing required config: {missing}")

# ใช้งาน
config = ConfigManager('development')
config.config['database']['name'] = 'myapp_dev'
config.config['app']['secret_key'] = 'dev-secret-key'
config.validate()
print(f"DB Host: {config.get('database.host')}")
print(f"Debug: {config.get('app.debug')}")
```

---

### ข้อที่ 4: XML to JSON Converter
สร้างโปรแกรมแปลง XML เป็น JSON

**เฉลย:**
```python
import xml.etree.ElementTree as ET
import json

def xml_to_dict(element):
    """แปลง XML element เป็น dict"""
    result = {}
    
    # เพิ่ม attributes
    if element.attrib:
        result['@attributes'] = element.attrib
    
    # เพิ่ม text content
    if element.text and element.text.strip():
        if element.attrib or list(element):
            result['#text'] = element.text.strip()
        else:
            return element.text.strip()
    
    # ประมวลผล children
    children = list(element)
    if children:
        child_dict = {}
        for child in children:
            child_data = xml_to_dict(child)
            if child.tag in child_dict:
                if not isinstance(child_dict[child.tag], list):
                    child_dict[child.tag] = [child_dict[child.tag]]
                child_dict[child.tag].append(child_data)
            else:
                child_dict[child.tag] = child_data
        result.update(child_dict)
    
    return result if result else None

xml_str = """
<catalog>
    <book id="1" language="Thai">
        <title>Python สำหรับมือใหม่</title>
        <author>สมชาย</author>
        <price>350</price>
    </book>
    <book id="2" language="English">
        <title>Clean Code</title>
        <author>Robert Martin</author>
        <price>550</price>
    </book>
</catalog>
"""

root = ET.fromstring(xml_str)
result = {root.tag: xml_to_dict(root)}
print(json.dumps(result, ensure_ascii=False, indent=2))
```

---

### ข้อที่ 5: Multi-Format Data Pipeline
สร้าง data pipeline ที่:
- รับข้อมูลจาก CSV
- แปลงและ validate ข้อมูล  
- export เป็น JSON และ XML

**เฉลย:**
```python
import csv
import json
import xml.etree.ElementTree as ET
from dataclasses import dataclass, asdict
from typing import List, Optional
import io

@dataclass
class Product:
    id: int
    name: str
    price: float
    category: str
    stock: int
    
    def to_xml_element(self):
        elem = ET.Element('product', id=str(self.id))
        for field in ['name', 'price', 'category', 'stock']:
            child = ET.SubElement(elem, field)
            child.text = str(getattr(self, field))
        return elem

def load_from_csv(csv_content: str) -> List[Product]:
    products = []
    reader = csv.DictReader(io.StringIO(csv_content))
    for i, row in enumerate(reader, 1):
        try:
            product = Product(
                id=i,
                name=row['name'].strip(),
                price=float(row['price']),
                category=row['category'].strip(),
                stock=int(row['stock'])
            )
            products.append(product)
        except (KeyError, ValueError) as e:
            print(f"Warning: Skipping row {i}: {e}")
    return products

def export_to_json(products: List[Product]) -> str:
    return json.dumps([asdict(p) for p in products], ensure_ascii=False, indent=2)

def export_to_xml(products: List[Product]) -> str:
    root = ET.Element('catalog')
    for product in products:
        root.append(product.to_xml_element())
    ET.indent(root, space='  ')
    return ET.tostring(root, encoding='unicode')

# ทดสอบ
csv_data = """name,price,category,stock
iPhone 15,35000,Electronics,20
Python Book,500,Books,100
MacBook Pro,89000,Electronics,5
"""

products = load_from_csv(csv_data)
print(f"โหลดสินค้า {len(products)} รายการ")
print("\n=== JSON Output ===")
print(export_to_json(products))
print("\n=== XML Output ===")
print(export_to_xml(products))
```

---

### ข้อที่ 6: JSON Schema Validator (แบบง่าย)
สร้างโปรแกรมตรวจสอบ JSON structure

**เฉลย:**
```python
def validate_json(data: dict, schema: dict, path: str = '') -> list:
    """Validate dict ตาม schema ที่กำหนด"""
    errors = []
    
    for field, rules in schema.items():
        field_path = f"{path}.{field}" if path else field
        
        if rules.get('required', False) and field not in data:
            errors.append(f"Required field missing: {field_path}")
            continue
        
        if field not in data:
            continue
        
        value = data[field]
        expected_type = rules.get('type')
        
        if expected_type and not isinstance(value, expected_type):
            errors.append(f"Type error at {field_path}: expected {expected_type.__name__}, got {type(value).__name__}")
        
        if 'min_length' in rules and isinstance(value, str):
            if len(value) < rules['min_length']:
                errors.append(f"Value too short at {field_path}: min {rules['min_length']}, got {len(value)}")
        
        if 'nested' in rules and isinstance(value, dict):
            errors.extend(validate_json(value, rules['nested'], field_path))
    
    return errors

# ใช้งาน
user_schema = {
    'name': {'type': str, 'required': True, 'min_length': 2},
    'age': {'type': int, 'required': True},
    'email': {'type': str, 'required': True},
    'address': {
        'type': dict,
        'required': False,
        'nested': {
            'city': {'type': str},
            'country': {'type': str}
        }
    }
}

valid_user = {'name': 'Alice', 'age': 25, 'email': 'alice@example.com'}
invalid_user = {'name': 'A', 'age': 'twenty-five'}  # ผิด type และ min_length

print("Valid user errors:", validate_json(valid_user, user_schema))
print("Invalid user errors:", validate_json(invalid_user, user_schema))
```

---

### ข้อที่ 7-10: โปรเจกต์ขนาดใหญ่

### ข้อที่ 7: Address Book ด้วย JSON

**เฉลย:**
```python
import json
import os
from datetime import datetime

class AddressBook:
    def __init__(self, filename='contacts.json'):
        self.filename = filename
        self.contacts = self._load()
    
    def _load(self):
        if os.path.exists(self.filename):
            with open(self.filename, 'r', encoding='utf-8') as f:
                return json.load(f)
        return {}
    
    def _save(self):
        with open(self.filename, 'w', encoding='utf-8') as f:
            json.dump(self.contacts, f, ensure_ascii=False, indent=2)
    
    def add(self, name, phone, email='', address='', tags=None):
        contact_id = str(len(self.contacts) + 1)
        self.contacts[contact_id] = {
            'name': name,
            'phone': phone,
            'email': email,
            'address': address,
            'tags': tags or [],
            'created_at': datetime.now().isoformat(),
            'updated_at': datetime.now().isoformat()
        }
        self._save()
        return contact_id
    
    def search(self, query):
        query = query.lower()
        results = []
        for cid, contact in self.contacts.items():
            if (query in contact['name'].lower() or
                query in contact['phone'] or
                query in contact.get('email', '').lower()):
                results.append({**contact, 'id': cid})
        return results
    
    def export_csv(self, filename):
        import csv
        with open(filename, 'w', newline='', encoding='utf-8-sig') as f:
            fieldnames = ['id', 'name', 'phone', 'email', 'address']
            writer = csv.DictWriter(f, fieldnames=fieldnames)
            writer.writeheader()
            for cid, contact in self.contacts.items():
                writer.writerow({
                    'id': cid,
                    'name': contact['name'],
                    'phone': contact['phone'],
                    'email': contact.get('email', ''),
                    'address': contact.get('address', '')
                })

# ใช้งาน
book = AddressBook()
book.add('สมชาย', '081-234-5678', 'somchai@gmail.com', 'กรุงเทพ')
book.add('สมหญิง', '082-345-6789', 'somying@gmail.com', 'เชียงใหม่')

results = book.search('สม')
print(f"พบ {len(results)} รายการ")
for r in results:
    print(f"  {r['name']}: {r['phone']}")

book.export_csv('contacts_export.csv')
print("Export สำเร็จ!")
```

---

### ข้อที่ 8: CSV Report Generator

**เฉลย:**
```python
import csv
import json
from collections import defaultdict
from datetime import datetime

def generate_monthly_report(sales_csv, output_json):
    """สร้าง monthly report จากข้อมูล CSV"""
    monthly_data = defaultdict(lambda: {
        'total_revenue': 0,
        'total_orders': 0,
        'products_sold': defaultdict(int),
        'daily_revenue': defaultdict(float)
    })
    
    with open(sales_csv, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            date = datetime.strptime(row['date'], '%Y-%m-%d')
            month_key = date.strftime('%Y-%m')
            amount = int(row['quantity']) * float(row['price'])
            
            monthly_data[month_key]['total_revenue'] += amount
            monthly_data[month_key]['total_orders'] += 1
            monthly_data[month_key]['products_sold'][row['product']] += int(row['quantity'])
            monthly_data[month_key]['daily_revenue'][row['date']] += amount
    
    report = {}
    for month, data in monthly_data.items():
        top_products = sorted(
            data['products_sold'].items(),
            key=lambda x: x[1],
            reverse=True
        )[:3]
        
        report[month] = {
            'total_revenue': round(data['total_revenue'], 2),
            'total_orders': data['total_orders'],
            'average_order_value': round(data['total_revenue'] / data['total_orders'], 2),
            'top_products': [{'name': p, 'units': u} for p, u in top_products],
            'daily_revenue': dict(data['daily_revenue'])
        }
    
    with open(output_json, 'w', encoding='utf-8') as f:
        json.dump(report, f, ensure_ascii=False, indent=2)
    
    return report

# สร้างข้อมูลตัวอย่าง
sample_sales = """product,quantity,price,date
iPhone 15,5,35000,2024-01-15
iPad Pro,3,32000,2024-01-15
Python Book,10,500,2024-01-16
MacBook,2,89000,2024-01-17
iPhone 15,3,35000,2024-02-10
"""

with open('sales.csv', 'w') as f:
    f.write(sample_sales)

report = generate_monthly_report('sales.csv', 'monthly_report.json')
for month, data in report.items():
    print(f"\n{month}:")
    print(f"  รายรับ: {data['total_revenue']:,.2f} บาท")
    print(f"  ออเดอร์: {data['total_orders']} รายการ")
```

---

### ข้อที่ 9: Multi-format Config System

**เฉลย:**
```python
import json
import yaml
import tomllib
import os
from pathlib import Path

class UniversalConfigLoader:
    """โหลด config จากหลาย format"""
    
    LOADERS = {
        '.json': '_load_json',
        '.yaml': '_load_yaml',
        '.yml': '_load_yaml',
        '.toml': '_load_toml',
    }
    
    def load(self, filepath: str) -> dict:
        path = Path(filepath)
        suffix = path.suffix.lower()
        
        loader = self.LOADERS.get(suffix)
        if not loader:
            raise ValueError(f"Unsupported format: {suffix}")
        
        return getattr(self, loader)(path)
    
    def _load_json(self, path):
        with open(path, 'r', encoding='utf-8') as f:
            return json.load(f)
    
    def _load_yaml(self, path):
        with open(path, 'r', encoding='utf-8') as f:
            return yaml.safe_load(f)
    
    def _load_toml(self, path):
        with open(path, 'rb') as f:
            return tomllib.load(f)
    
    def save(self, data: dict, filepath: str, **kwargs):
        path = Path(filepath)
        suffix = path.suffix.lower()
        
        if suffix == '.json':
            with open(path, 'w', encoding='utf-8') as f:
                json.dump(data, f, ensure_ascii=False, indent=2, **kwargs)
        elif suffix in ('.yaml', '.yml'):
            with open(path, 'w', encoding='utf-8') as f:
                yaml.dump(data, f, default_flow_style=False, allow_unicode=True)
        else:
            raise ValueError(f"Cannot save to format: {suffix}")

# ใช้งาน
loader = UniversalConfigLoader()

# สร้างไฟล์ JSON ก่อน
config = {'app': {'name': 'MyApp', 'version': '1.0'}, 'debug': True}
loader.save(config, 'config.json')

# โหลด
loaded = loader.load('config.json')
print(loaded)
```

---

### ข้อที่ 10: Data Backup System

**เฉลย:**
```python
import json
import csv
import zipfile
import os
from datetime import datetime
from pathlib import Path

class DataBackupSystem:
    """ระบบ backup ข้อมูลหลาย format"""
    
    def __init__(self, backup_dir='backups'):
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
    
    def create_backup(self, data: dict, name: str) -> str:
        """สร้าง backup ในรูปแบบ zip file"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_name = f"{name}_{timestamp}"
        backup_path = self.backup_dir / f"{backup_name}.zip"
        
        with zipfile.ZipFile(backup_path, 'w', zipfile.ZIP_DEFLATED) as zf:
            # บันทึก JSON
            json_str = json.dumps(data, ensure_ascii=False, indent=2)
            zf.writestr(f"{backup_name}/data.json", json_str)
            
            # บันทึก metadata
            metadata = {
                'backup_name': backup_name,
                'created_at': timestamp,
                'data_keys': list(data.keys()),
                'total_records': sum(
                    len(v) if isinstance(v, (list, dict)) else 1
                    for v in data.values()
                )
            }
            zf.writestr(
                f"{backup_name}/metadata.json",
                json.dumps(metadata, indent=2)
            )
        
        return str(backup_path)
    
    def restore(self, backup_path: str) -> dict:
        """กู้คืนข้อมูลจาก backup"""
        with zipfile.ZipFile(backup_path, 'r') as zf:
            names = zf.namelist()
            data_file = next(n for n in names if n.endswith('data.json'))
            with zf.open(data_file) as f:
                return json.loads(f.read().decode('utf-8'))
    
    def list_backups(self) -> list:
        """แสดงรายการ backups"""
        return sorted(self.backup_dir.glob('*.zip'), reverse=True)

# ใช้งาน
backup = DataBackupSystem()

data = {
    'users': [{'id': 1, 'name': 'Alice'}, {'id': 2, 'name': 'Bob'}],
    'products': [{'id': 1, 'name': 'iPhone', 'price': 35000}],
    'settings': {'version': '1.0', 'backup_enabled': True}
}

path = backup.create_backup(data, 'myapp')
print(f"Backup สร้างที่: {path}")

restored = backup.restore(path)
print(f"กู้คืน users: {len(restored['users'])} คน")
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

1. **JSON**: รูปแบบข้อมูลยอดนิยมสำหรับ web APIs พร้อมการจัดการ custom objects และ datetime
2. **CSV**: รูปแบบข้อมูลแบบตาราง พร้อม DictReader/DictWriter สำหรับการทำงานที่สะดวก
3. **Excel**: การสร้างและอ่านไฟล์ Excel ด้วย openpyxl
4. **YAML**: รูปแบบ config ที่อ่านง่าย
5. **TOML**: รูปแบบ config สมัยใหม่ที่ Python ใช้ใน pyproject.toml
6. **XML**: การ parse XML ด้วย ElementTree
7. **Patterns**: Best practices สำหรับ data serialization

ทักษะเหล่านี้จะใช้ได้ทุกวันในการพัฒนาซอฟต์แวร์จริง!
