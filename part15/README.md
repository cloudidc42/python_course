# Part 15: File I/O - Reading & Writing Files

## บทนำ (Introduction)

**File I/O** (Input/Output) เป็นความสามารถพื้นฐานในการอ่านและเขียนข้อมูลลงในไฟล์
Python มีเครื่องมือครบครันสำหรับจัดการไฟล์ทั้ง text files, binary files, CSV, และ JSON

### ทำไมต้องเรียน File I/O
- เก็บข้อมูลถาวรระหว่าง program runs
- อ่านข้อมูลจาก configuration files
- ประมวลผล log files
- แลกเปลี่ยนข้อมูลด้วย CSV/JSON
- บันทึกผลลัพธ์การประมวลผล

---

## 1. การเปิดและปิดไฟล์ด้วย open()

### 1.1 open() Function

```python
# Syntax: open(file, mode='r', encoding=None, ...)
# คืน file object

# วิธีที่ 1: เปิดและปิดเอง (ต้องระวังลืมปิด!)
file = open("example.txt", "r")
content = file.read()
file.close()  # ต้องปิดเสมอ!

# วิธีที่ 2: ใช้ try-finally (ดีกว่าแต่ verbose)
file = None
try:
    file = open("example.txt", "r")
    content = file.read()
finally:
    if file:
        file.close()

# วิธีที่ 3: Context Manager - with statement (แนะนำสุด!)
with open("example.txt", "r") as file:
    content = file.read()
# ปิดอัตโนมัติเมื่อออกจาก with block (แม้เกิด exception)

print("File closed:", file.closed)  # True
```

### 1.2 File Modes

```python
# Mode ที่ใช้บ่อย:
# 'r'  - Read (default): เปิดอ่าน, error ถ้าไม่มีไฟล์
# 'w'  - Write: เปิดเขียน (ลบเนื้อหาเดิม หรือสร้างใหม่)
# 'a'  - Append: เพิ่มท้ายไฟล์ (หรือสร้างใหม่)
# 'r+' - Read+Write: อ่านและเขียน, error ถ้าไม่มีไฟล์
# 'w+' - Write+Read: เขียนและอ่าน, สร้างใหม่ถ้าไม่มี
# 'a+' - Append+Read: เพิ่มและอ่าน
# 'rb' - Read Binary
# 'wb' - Write Binary
# 'ab' - Append Binary
# 'x'  - Exclusive create: สร้างใหม่, error ถ้ามีอยู่แล้ว

# Text mode
with open("test.txt", "w") as f:
    f.write("Hello World\n")

with open("test.txt", "r") as f:
    print(f.read())

# Append mode
with open("test.txt", "a") as f:
    f.write("New line appended\n")

# Binary mode
with open("test.bin", "wb") as f:
    f.write(b"\x48\x65\x6c\x6c\x6f")  # "Hello" ใน binary

with open("test.bin", "rb") as f:
    data = f.read()
    print(data)          # b'Hello'
    print(data.decode()) # Hello
```

---

## 2. การอ่านไฟล์

### 2.1 read() - อ่านทั้งไฟล์

```python
# สร้างไฟล์ทดสอบก่อน
with open("sample.txt", "w", encoding="utf-8") as f:
    f.write("Line 1: Hello World\n")
    f.write("Line 2: Python Programming\n")
    f.write("Line 3: File I/O\n")
    f.write("Line 4: Reading and Writing\n")
    f.write("Line 5: The End\n")

# read() - อ่านทั้งหมดเป็น string เดียว
with open("sample.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(type(content))  # <class 'str'>
    print(content)

# read(n) - อ่าน n bytes/characters
with open("sample.txt", "r", encoding="utf-8") as f:
    first_10 = f.read(10)
    print(repr(first_10))  # 'Line 1: He'
    next_10 = f.read(10)
    print(repr(next_10))   # 'llo World\n'

# tell() และ seek()
with open("sample.txt", "r", encoding="utf-8") as f:
    print(f.tell())    # 0 (ตำแหน่งปัจจุบัน)
    f.read(5)
    print(f.tell())    # 5
    f.seek(0)          # กลับต้น
    print(f.tell())    # 0
    f.seek(0, 2)       # ไปท้ายไฟล์ (whence=2)
    print(f.tell())    # ขนาดไฟล์
```

### 2.2 readline() - อ่านทีละบรรทัด

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    # อ่านทีละบรรทัด
    line1 = f.readline()
    print(repr(line1))  # 'Line 1: Hello World\n'
    
    line2 = f.readline()
    print(repr(line2))  # 'Line 2: Python Programming\n'
    
    # อ่านจนหมด
    while True:
        line = f.readline()
        if not line:  # ถ้าอ่านจนหมด คืน empty string
            break
        print(line.strip())  # strip() ลบ \n
```

### 2.3 readlines() - อ่านทั้งหมดเป็น list

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(type(lines))   # <class 'list'>
    print(len(lines))    # 5
    print(lines[0])      # 'Line 1: Hello World\n'
    
    # ลบ newline
    clean_lines = [line.strip() for line in lines]
    print(clean_lines)

# วิธีที่ดีกว่า: iterate ตรงๆ (memory efficient)
with open("sample.txt", "r", encoding="utf-8") as f:
    for line in f:  # file object เป็น iterator
        print(line.strip())
```

### 2.4 อ่านไฟล์ขนาดใหญ่อย่างมีประสิทธิภาพ

```python
def read_large_file_in_chunks(filepath, chunk_size=1024):
    """อ่านไฟล์ขนาดใหญ่ทีละ chunk"""
    with open(filepath, "r", encoding="utf-8") as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

# ใช้งาน
for chunk in read_large_file_in_chunks("sample.txt", chunk_size=50):
    print(f"Chunk: {repr(chunk)}")

# Count lines อย่างมีประสิทธิภาพ
def count_lines(filepath):
    """นับบรรทัดโดยไม่โหลดทั้งไฟล์ลง memory"""
    count = 0
    with open(filepath, "r", encoding="utf-8") as f:
        for _ in f:
            count += 1
    return count

print(f"Lines: {count_lines('sample.txt')}")
```

---

## 3. การเขียนไฟล์

### 3.1 write() - เขียน string

```python
# เขียนใหม่ทั้งหมด (mode='w')
with open("output.txt", "w", encoding="utf-8") as f:
    chars_written = f.write("Hello, World!\n")
    print(f"Written: {chars_written} characters")
    
    f.write("Python File I/O\n")
    f.write("End of file\n")

# เพิ่มท้ายไฟล์ (mode='a')
with open("output.txt", "a", encoding="utf-8") as f:
    f.write("Appended line\n")

# ตรวจสอบผล
with open("output.txt", "r") as f:
    print(f.read())
```

### 3.2 writelines() - เขียนหลาย strings

```python
lines = [
    "First line\n",
    "Second line\n",
    "Third line\n",
    "Fourth line\n",
]

with open("multiline.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)  # เขียนทีเดียวหมด

# หรือจาก list of data
data = ["Alice", "Bob", "Charlie", "Diana"]
with open("names.txt", "w", encoding="utf-8") as f:
    f.writelines(f"{name}\n" for name in data)

# ตรวจสอบ
with open("names.txt", "r") as f:
    print(f.read())
```

### 3.3 print() เข้า File

```python
# print() สามารถเขียนลง file ได้ด้วย file parameter
with open("report.txt", "w", encoding="utf-8") as f:
    print("=" * 40, file=f)
    print("MONTHLY REPORT", file=f)
    print("=" * 40, file=f)
    print(f"{'Item':20} {'Amount':>10}", file=f)
    print("-" * 40, file=f)
    print(f"{'Sales':20} {'฿100,000':>10}", file=f)
    print(f"{'Expenses':20} {'฿40,000':>10}", file=f)
    print("-" * 40, file=f)
    print(f"{'Profit':20} {'฿60,000':>10}", file=f)

with open("report.txt", "r") as f:
    print(f.read())
```

---

## 4. Context Manager (with statement)

### 4.1 ทำไมต้องใช้ with

```python
# ปัญหาเมื่อไม่ใช้ with
f = open("test.txt", "w")
f.write("Hello")
# ถ้าเกิด exception ก่อนถึง close() → file ไม่ถูกปิด!
# f.close()  ← อาจไม่ถูกเรียก

# with ปิดไฟล์อัตโนมัติแม้เกิด exception
with open("test.txt", "w") as f:
    f.write("Hello")
    raise ValueError("Something went wrong")
# ไฟล์ถูกปิดแม้ว่าจะเกิด exception

# with เปิดหลายไฟล์พร้อมกัน
with open("input.txt", "r") as fin, open("output.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

### 4.2 สร้าง Context Manager เอง

```python
from contextlib import contextmanager

@contextmanager
def managed_file(filepath, mode="r", encoding="utf-8"):
    """Custom context manager สำหรับไฟล์"""
    file = None
    try:
        file = open(filepath, mode, encoding=encoding)
        print(f"Opened: {filepath}")
        yield file
    except IOError as e:
        print(f"Error: {e}")
        raise
    finally:
        if file:
            file.close()
            print(f"Closed: {filepath}")

with managed_file("test.txt", "w") as f:
    f.write("Custom context manager!\n")

with managed_file("test.txt", "r") as f:
    print(f.read())
```

---

## 5. Working with Paths

### 5.1 os.path Module

```python
import os

# Path operations
filepath = "/home/user/documents/report.txt"

print(os.path.dirname(filepath))   # /home/user/documents
print(os.path.basename(filepath))  # report.txt
print(os.path.splitext(filepath))  # ('/home/user/documents/report', '.txt')
print(os.path.split(filepath))     # ('/home/user/documents', 'report.txt')

# ตรวจสอบ
print(os.path.exists(filepath))     # True/False
print(os.path.isfile(filepath))     # True ถ้าเป็นไฟล์
print(os.path.isdir("/home/user"))  # True ถ้าเป็น directory
print(os.path.getsize(filepath))    # ขนาดไฟล์ (bytes)

# Join paths (platform-independent)
base = "/home/user"
subdir = "documents"
filename = "report.txt"
full_path = os.path.join(base, subdir, filename)
print(full_path)  # /home/user/documents/report.txt

# Absolute path
rel_path = "test.txt"
abs_path = os.path.abspath(rel_path)
print(abs_path)

# Current directory
print(os.getcwd())

# List directory
for item in os.listdir("."):
    item_path = os.path.join(".", item)
    if os.path.isfile(item_path):
        size = os.path.getsize(item_path)
        print(f"  FILE  {item:30} {size:,} bytes")
    else:
        print(f"  DIR   {item}")
```

### 5.2 pathlib.Path (Python 3.4+)

```python
from pathlib import Path

# สร้าง Path object
p = Path("/home/user/documents/report.txt")
home = Path.home()
cwd = Path.cwd()

print(p)              # /home/user/documents/report.txt
print(p.parent)       # /home/user/documents
print(p.name)         # report.txt
print(p.stem)         # report
print(p.suffix)       # .txt
print(p.parts)        # ('/', 'home', 'user', 'documents', 'report.txt')

# Join paths ด้วย /
documents = home / "documents"
report = documents / "report.txt"
print(report)

# ตรวจสอบ
test_file = Path("sample.txt")
print(test_file.exists())    # True/False
print(test_file.is_file())   # True/False
print(test_file.is_dir())    # False

# สถิติไฟล์
if test_file.exists():
    stat = test_file.stat()
    print(f"Size: {stat.st_size} bytes")

# อ่านเขียนด้วย Path (สะดวกมาก!)
# เขียน
test_file.write_text("Hello from pathlib!\n", encoding="utf-8")

# อ่าน
content = test_file.read_text(encoding="utf-8")
print(content)

# Binary
bin_file = Path("test.bin")
bin_file.write_bytes(b"\x00\x01\x02\x03")
data = bin_file.read_bytes()
print(data)

# glob patterns
import os
# หาไฟล์ทั้งหมด
current = Path(".")
for txt_file in current.glob("*.txt"):
    print(txt_file)

for py_file in current.rglob("*.py"):  # recursive
    print(py_file)
```

### 5.3 File and Directory Operations

```python
import os
import shutil
from pathlib import Path

# สร้าง directory
os.makedirs("test_dir/subdir", exist_ok=True)
Path("test_dir2/nested").mkdir(parents=True, exist_ok=True)

# Copy file
shutil.copy("sample.txt", "test_dir/sample_copy.txt")
shutil.copy2("sample.txt", "test_dir/sample_copy2.txt")  # preserve metadata

# Move file
shutil.move("test_dir/sample_copy2.txt", "test_dir/subdir/moved.txt")

# Rename
os.rename("test_dir/sample_copy.txt", "test_dir/renamed.txt")
# หรือด้วย pathlib
Path("test_dir/renamed.txt").rename("test_dir/final.txt")

# Delete
os.remove("test_dir/final.txt")         # ลบไฟล์
os.rmdir("test_dir/subdir")             # ลบ directory ว่าง
shutil.rmtree("test_dir2")              # ลบ directory ทั้งหมด

# Walk directory tree
for dirpath, dirnames, filenames in os.walk("."):
    level = dirpath.replace(".", "").count(os.sep)
    indent = "  " * level
    print(f"{indent}{os.path.basename(dirpath)}/")
    for file in filenames:
        print(f"{indent}  {file}")
```

---

## 6. Binary Files

### 6.1 การอ่านเขียน Binary

```python
# เขียน binary data
with open("data.bin", "wb") as f:
    # เขียน bytes
    f.write(b"Hello, Binary World!")
    f.write(bytes([72, 101, 108, 108, 111]))  # "Hello"
    
    # struct สำหรับข้อมูลแบบ structured
    import struct
    # เขียน int (4 bytes), float (4 bytes)
    data = struct.pack("if", 42, 3.14)
    f.write(data)

# อ่าน binary data
with open("data.bin", "rb") as f:
    content = f.read()
    print(content[:20])      # แสดง bytes
    print(content[:5].decode("utf-8"))  # แปลงเป็น string
    
    # อ่านด้วย struct
    f.seek(-8, 2)  # ไปก่อนท้ายสุด 8 bytes
    raw = f.read(8)
    num, flt = struct.unpack("if", raw)
    print(f"int: {num}, float: {flt:.2f}")
```

### 6.2 Image File (อ่านเบื้องต้น)

```python
# อ่าน/เขียนไฟล์ image เป็น binary
def read_image_info(filepath):
    """อ่านข้อมูลพื้นฐานของ PNG/JPEG"""
    with open(filepath, "rb") as f:
        header = f.read(10)
        
        if header[:8] == b'\x89PNG\r\n\x1a\n':
            print(f"{filepath}: PNG image")
        elif header[:3] == b'\xff\xd8\xff':
            print(f"{filepath}: JPEG image")
        else:
            print(f"{filepath}: Unknown format")
        
        f.seek(0, 2)
        size = f.tell()
        print(f"File size: {size:,} bytes")

# Copy file (binary-safe)
def copy_file(src, dst):
    """Copy file ที่รองรับทั้ง text และ binary"""
    with open(src, "rb") as fin, open(dst, "wb") as fout:
        chunk_size = 64 * 1024  # 64KB chunks
        while True:
            chunk = fin.read(chunk_size)
            if not chunk:
                break
            fout.write(chunk)
    return True
```

---

## 7. CSV Files

### 7.1 csv Module พื้นฐาน

```python
import csv

# สร้างข้อมูลทดสอบ
students = [
    ["Name", "Age", "Grade", "City"],
    ["Alice", "20", "A", "Bangkok"],
    ["Bob", "22", "B", "Chiang Mai"],
    ["Charlie", "21", "A", "Phuket"],
    ["Diana", "23", "C", "Bangkok"],
]

# เขียน CSV
with open("students.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(students)  # เขียนทุก row

print("เขียน students.csv สำเร็จ")

# อ่าน CSV
print("\nอ่าน students.csv:")
with open("students.csv", "r", encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

### 7.2 DictWriter และ DictReader

```python
import csv

# ข้อมูลเป็น list of dicts
employees = [
    {"id": 1, "name": "Alice", "dept": "Engineering", "salary": 80000},
    {"id": 2, "name": "Bob", "dept": "Marketing", "salary": 60000},
    {"id": 3, "name": "Charlie", "dept": "HR", "salary": 55000},
    {"id": 4, "name": "Diana", "dept": "Engineering", "salary": 90000},
]

# เขียนด้วย DictWriter
with open("employees.csv", "w", newline="", encoding="utf-8") as f:
    fieldnames = ["id", "name", "dept", "salary"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    
    writer.writeheader()  # เขียน header
    writer.writerows(employees)  # เขียนทุก row

print("เขียน employees.csv สำเร็จ")

# อ่านด้วย DictReader
print("\nอ่าน employees.csv:")
with open("employees.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    print(f"Columns: {reader.fieldnames}")
    for row in reader:
        print(f"  {row['name']:10} | {row['dept']:12} | ฿{int(row['salary']):,}")
```

### 7.3 CSV Processor

```python
import csv
from collections import defaultdict

class CSVProcessor:
    """ประมวลผล CSV files"""
    
    def __init__(self, filepath):
        self.filepath = filepath
        self.data = []
        self.headers = []
        self._load()
    
    def _load(self):
        """โหลดข้อมูลจาก CSV"""
        with open(self.filepath, "r", encoding="utf-8") as f:
            reader = csv.DictReader(f)
            self.headers = reader.fieldnames
            self.data = list(reader)
    
    def filter_rows(self, condition):
        """กรอง rows ตาม condition function"""
        return [row for row in self.data if condition(row)]
    
    def aggregate(self, group_by, value_col, func=sum):
        """จัดกลุ่มและรวมค่า"""
        groups = defaultdict(list)
        for row in self.data:
            key = row[group_by]
            groups[key].append(float(row[value_col]))
        return {k: func(v) for k, v in groups.items()}
    
    def column_stats(self, col):
        """สถิติของ column"""
        values = [float(row[col]) for row in self.data
                  if row.get(col, "").replace(".", "").isdigit()]
        if not values:
            return None
        return {
            "count": len(values),
            "sum": sum(values),
            "avg": sum(values) / len(values),
            "min": min(values),
            "max": max(values)
        }
    
    def export(self, filepath, rows=None, columns=None):
        """Export ไปยังไฟล์ใหม่"""
        rows = rows or self.data
        cols = columns or self.headers
        with open(filepath, "w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=cols, extrasaction="ignore")
            writer.writeheader()
            writer.writerows(rows)

# ทดสอบ
# สร้างข้อมูลทดสอบก่อน
test_data = [
    {"name": "Alice", "dept": "Eng", "salary": "80000", "years": "5"},
    {"name": "Bob", "dept": "Mkt", "salary": "60000", "years": "3"},
    {"name": "Charlie", "dept": "Eng", "salary": "90000", "years": "8"},
    {"name": "Diana", "dept": "HR", "salary": "55000", "years": "2"},
    {"name": "Eve", "dept": "Mkt", "salary": "65000", "years": "4"},
]
with open("hr_data.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "dept", "salary", "years"])
    writer.writeheader()
    writer.writerows(test_data)

proc = CSVProcessor("hr_data.csv")

# กรอง Engineering
eng = proc.filter_rows(lambda r: r["dept"] == "Eng")
print("Engineering:", [r["name"] for r in eng])

# เฉลี่ยเงินเดือนต่อแผนก
dept_avg = proc.aggregate("dept", "salary", lambda v: sum(v)/len(v))
for dept, avg in sorted(dept_avg.items()):
    print(f"  {dept}: ฿{avg:,.0f}")

# Stats ของ salary
stats = proc.column_stats("salary")
print(f"\nSalary stats: {stats}")
```

---

## 8. JSON Files

### 8.1 json Module พื้นฐาน

```python
import json

# Python → JSON (Serialization)
person = {
    "name": "Alice",
    "age": 25,
    "is_student": False,
    "scores": [85, 90, 92],
    "address": {
        "city": "Bangkok",
        "country": "Thailand"
    },
    "phone": None
}

# dump() - เขียนลง file
with open("person.json", "w", encoding="utf-8") as f:
    json.dump(person, f, ensure_ascii=False, indent=2)

print("เขียน person.json สำเร็จ")

# dumps() - แปลงเป็น string
json_string = json.dumps(person, ensure_ascii=False, indent=2)
print(json_string)

# JSON → Python (Deserialization)
# load() - อ่านจาก file
with open("person.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)

print(f"\nLoaded: {loaded['name']}, age {loaded['age']}")
print(f"Scores: {loaded['scores']}")

# loads() - แปลงจาก string
json_str = '{"name": "Bob", "age": 30, "active": true}'
data = json.loads(json_str)
print(f"\nParsed: {data}")
```

### 8.2 JSON Type Mapping

```python
import json
from datetime import datetime

# Python → JSON Type mapping
data = {
    "str": "hello",           # str → string
    "int": 42,                # int → number
    "float": 3.14,            # float → number
    "bool_true": True,        # bool → true
    "bool_false": False,      # bool → false
    "none": None,             # None → null
    "list": [1, 2, 3],        # list → array
    "tuple": (4, 5, 6),       # tuple → array (!)
    "dict": {"key": "value"}  # dict → object
}

print(json.dumps(data, indent=2))

# Custom encoder สำหรับ types ที่ JSON ไม่รองรับ
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        if hasattr(obj, "__dict__"):
            return obj.__dict__
        return super().default(obj)

class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

data2 = {
    "user": User("Alice", 25),
    "created": datetime(2024, 1, 15, 10, 30),
    "tags": {"python", "programming"}
}

print(json.dumps(data2, cls=CustomEncoder, indent=2))
```

### 8.3 JSON Config System

```python
import json
from pathlib import Path
import os

class JSONConfig:
    """จัดการ config ด้วย JSON"""
    
    DEFAULT_CONFIG = {
        "app": {
            "name": "MyApp",
            "version": "1.0.0",
            "debug": False
        },
        "database": {
            "host": "localhost",
            "port": 5432,
            "name": "mydb"
        },
        "server": {
            "host": "0.0.0.0",
            "port": 8080,
            "timeout": 30
        }
    }
    
    def __init__(self, config_path="config.json"):
        self.config_path = Path(config_path)
        self.config = {}
        self._load()
    
    def _load(self):
        """โหลด config จากไฟล์"""
        import copy
        self.config = copy.deepcopy(self.DEFAULT_CONFIG)
        
        if self.config_path.exists():
            with open(self.config_path, "r", encoding="utf-8") as f:
                user_config = json.load(f)
            self._deep_merge(self.config, user_config)
        else:
            self._save()  # สร้างไฟล์ default
    
    def _deep_merge(self, base, override):
        for key, value in override.items():
            if key in base and isinstance(base[key], dict) and isinstance(value, dict):
                self._deep_merge(base[key], value)
            else:
                base[key] = value
    
    def _save(self):
        """บันทึก config"""
        with open(self.config_path, "w", encoding="utf-8") as f:
            json.dump(self.config, f, indent=2, ensure_ascii=False)
    
    def get(self, key_path, default=None):
        """ดึงค่าด้วย dot notation"""
        keys = key_path.split(".")
        current = self.config
        for key in keys:
            if isinstance(current, dict):
                current = current.get(key)
            else:
                return default
        return current if current is not None else default
    
    def set(self, key_path, value):
        """กำหนดค่าและบันทึก"""
        keys = key_path.split(".")
        current = self.config
        for key in keys[:-1]:
            current = current.setdefault(key, {})
        current[keys[-1]] = value
        self._save()
    
    def reload(self):
        """โหลดใหม่จากไฟล์"""
        self._load()

# ทดสอบ
cfg = JSONConfig("app_config.json")
print(f"App name: {cfg.get('app.name')}")
print(f"DB host: {cfg.get('database.host')}")
print(f"Debug: {cfg.get('app.debug')}")

cfg.set("app.debug", True)
cfg.set("database.host", "db.production.com")
cfg.set("features.dark_mode", True)  # สร้าง key ใหม่

print(f"\nAfter update:")
print(f"Debug: {cfg.get('app.debug')}")
print(f"DB host: {cfg.get('database.host')}")
print(f"Dark mode: {cfg.get('features.dark_mode')}")
```

---

## 9. Log File Reader

### 9.1 Log File System

```python
import json
import os
from datetime import datetime
from pathlib import Path

class Logger:
    """ระบบ logging ลงไฟล์"""
    
    LEVELS = {"DEBUG": 10, "INFO": 20, "WARNING": 30, "ERROR": 40, "CRITICAL": 50}
    
    def __init__(self, log_file="app.log", level="INFO"):
        self.log_file = Path(log_file)
        self.min_level = self.LEVELS.get(level, 20)
        self.log_file.parent.mkdir(parents=True, exist_ok=True)
    
    def _log(self, level, message, **extra):
        if self.LEVELS.get(level, 0) < self.min_level:
            return
        
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        log_entry = f"[{timestamp}] [{level:8}] {message}"
        if extra:
            log_entry += f" | {extra}"
        log_entry += "\n"
        
        with open(self.log_file, "a", encoding="utf-8") as f:
            f.write(log_entry)
        
        # แสดงใน console ด้วย
        print(log_entry.strip())
    
    def debug(self, msg, **kw): self._log("DEBUG", msg, **kw)
    def info(self, msg, **kw): self._log("INFO", msg, **kw)
    def warning(self, msg, **kw): self._log("WARNING", msg, **kw)
    def error(self, msg, **kw): self._log("ERROR", msg, **kw)
    def critical(self, msg, **kw): self._log("CRITICAL", msg, **kw)


class LogReader:
    """อ่านและวิเคราะห์ log files"""
    
    def __init__(self, log_file):
        self.log_file = Path(log_file)
    
    def read_all(self):
        """อ่าน log ทั้งหมด"""
        if not self.log_file.exists():
            return []
        with open(self.log_file, "r", encoding="utf-8") as f:
            return f.readlines()
    
    def filter_by_level(self, level):
        """กรองตาม level"""
        return [line for line in self.read_all() if f"[{level}" in line]
    
    def filter_by_date(self, date_str):
        """กรองตามวันที่ (YYYY-MM-DD)"""
        return [line for line in self.read_all() if line.startswith(f"[{date_str}")]
    
    def search(self, keyword):
        """ค้นหาใน log"""
        return [line for line in self.read_all() if keyword.lower() in line.lower()]
    
    def tail(self, n=20):
        """อ่าน n บรรทัดสุดท้าย (เหมือน tail command)"""
        lines = self.read_all()
        return lines[-n:] if lines else []
    
    def summary(self):
        """สรุปข้อมูล log"""
        from collections import Counter
        lines = self.read_all()
        levels = []
        for line in lines:
            for level in ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]:
                if f"[{level}" in line:
                    levels.append(level)
                    break
        
        return {
            "total_lines": len(lines),
            "level_counts": dict(Counter(levels)),
            "file_size": self.log_file.stat().st_size if self.log_file.exists() else 0
        }

# ทดสอบ
logger = Logger("test.log", level="DEBUG")
logger.info("Application started")
logger.debug("Debug message", user="alice")
logger.warning("Low disk space", disk_free="2GB")
logger.error("Database connection failed", host="localhost", port=5432)
logger.info("Processing complete", records=1000)
logger.critical("System out of memory")

reader = LogReader("test.log")
summary = reader.summary()
print(f"\nLog Summary:")
print(f"  Total lines: {summary['total_lines']}")
print(f"  Level counts: {summary['level_counts']}")

print("\nERROR logs:")
for line in reader.filter_by_level("ERROR"):
    print(f"  {line.strip()}")
```

---

## 10. โปรแกรมจริง (Real-world Examples)

### 10.1 Complete File Manager

```python
import os
import shutil
import json
from pathlib import Path
from datetime import datetime

class FileManager:
    """จัดการไฟล์และ directories"""
    
    def __init__(self, base_dir="."):
        self.base = Path(base_dir)
    
    def list_files(self, pattern="*", recursive=False):
        """แสดงรายการไฟล์"""
        if recursive:
            files = list(self.base.rglob(pattern))
        else:
            files = list(self.base.glob(pattern))
        
        result = []
        for f in sorted(files):
            if f.is_file():
                stat = f.stat()
                result.append({
                    "name": f.name,
                    "path": str(f),
                    "size": stat.st_size,
                    "modified": datetime.fromtimestamp(stat.st_mtime).strftime("%Y-%m-%d %H:%M"),
                    "extension": f.suffix
                })
        return result
    
    def organize_by_extension(self, source_dir, target_dir):
        """จัดระเบียบไฟล์ตาม extension"""
        source = Path(source_dir)
        target = Path(target_dir)
        moved = {}
        
        for file in source.iterdir():
            if file.is_file():
                ext = file.suffix.lstrip(".").upper() or "OTHERS"
                dest_dir = target / ext
                dest_dir.mkdir(parents=True, exist_ok=True)
                
                dest_file = dest_dir / file.name
                shutil.copy2(file, dest_file)
                moved.setdefault(ext, []).append(file.name)
        
        return moved
    
    def find_duplicates(self, directory):
        """หาไฟล์ที่ซ้ำกัน (ตามขนาด)"""
        from collections import defaultdict
        size_map = defaultdict(list)
        
        for file in Path(directory).rglob("*"):
            if file.is_file():
                size_map[file.stat().st_size].append(str(file))
        
        return {size: paths for size, paths in size_map.items()
                if len(paths) > 1}
    
    def backup(self, source, backup_dir, max_backups=5):
        """Backup ไฟล์ด้วย timestamp"""
        source = Path(source)
        backup = Path(backup_dir)
        backup.mkdir(parents=True, exist_ok=True)
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"{source.stem}_{timestamp}{source.suffix}"
        backup_path = backup / backup_name
        
        shutil.copy2(source, backup_path)
        
        # ลบ backup เก่า ถ้ามีเกิน max_backups
        existing = sorted(backup.glob(f"{source.stem}_*{source.suffix}"))
        while len(existing) > max_backups:
            existing.pop(0).unlink()
            existing = sorted(backup.glob(f"{source.stem}_*{source.suffix}"))
        
        return backup_path
    
    def search_in_files(self, directory, keyword, extension=".txt"):
        """ค้นหา keyword ในไฟล์"""
        results = []
        for file in Path(directory).rglob(f"*{extension}"):
            try:
                with open(file, "r", encoding="utf-8", errors="ignore") as f:
                    for line_num, line in enumerate(f, 1):
                        if keyword.lower() in line.lower():
                            results.append({
                                "file": str(file),
                                "line": line_num,
                                "content": line.strip()
                            })
            except Exception:
                pass
        return results

# ทดสอบ
fm = FileManager(".")
files = fm.list_files("*.txt")
print(f"Found {len(files)} .txt files:")
for f in files[:5]:
    print(f"  {f['name']:30} {f['size']:,} bytes  {f['modified']}")

dupes = fm.find_duplicates(".")
if dupes:
    print(f"\nDuplicate files by size:")
    for size, paths in list(dupes.items())[:3]:
        print(f"  {size:,} bytes: {paths}")
```

---

## แบบฝึกหัด (Exercises)

### ข้อ 1: Word Counter
```python
def count_words_in_file(filepath):
    """นับคำในไฟล์"""
    from collections import Counter
    with open(filepath, "r", encoding="utf-8") as f:
        text = f.read().lower()
    words = [w.strip(".,!?;:\"'") for w in text.split()]
    words = [w for w in words if w]
    return Counter(words)

# ทดสอบ
with open("test_words.txt", "w") as f:
    f.write("the quick brown fox jumps over the lazy dog the fox")

counts = count_words_in_file("test_words.txt")
print("Top 5 words:", counts.most_common(5))
```

### ข้อ 2: CSV to JSON Converter
```python
import csv
import json

def csv_to_json(csv_file, json_file):
    """แปลง CSV เป็น JSON"""
    data = []
    with open(csv_file, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            data.append(dict(row))
    
    with open(json_file, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    
    return len(data)

# ทดสอบ
with open("test.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["name", "age", "city"])
    writer.writerow(["Alice", "25", "Bangkok"])
    writer.writerow(["Bob", "30", "Chiang Mai"])

n = csv_to_json("test.csv", "test.json")
print(f"Converted {n} records")

with open("test.json") as f:
    print(json.load(f))
```

### ข้อ 3: Log Analyzer
```python
import re
from collections import Counter
from datetime import datetime

def analyze_log(log_file):
    """วิเคราะห์ log file"""
    pattern = r'\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] \[(\w+)\s*\] (.+)'
    
    results = {"by_level": Counter(), "errors": [], "warnings": []}
    
    with open(log_file, "r", encoding="utf-8") as f:
        for line in f:
            m = re.match(pattern, line)
            if m:
                timestamp, level, message = m.groups()
                results["by_level"][level.strip()] += 1
                if level.strip() == "ERROR":
                    results["errors"].append(message)
                elif level.strip() == "WARNING":
                    results["warnings"].append(message)
    
    return results

if __name__ == "__main__":
    # ต้องมีไฟล์ test.log อยู่ก่อน
    import os
    if os.path.exists("test.log"):
        result = analyze_log("test.log")
        print("Level counts:", dict(result["by_level"]))
        print(f"Errors ({len(result['errors'])}):", result["errors"][:3])
```

### ข้อ 4: File Line Sorter
```python
def sort_file_lines(input_file, output_file, key_func=None, reverse=False):
    """เรียงบรรทัดในไฟล์"""
    with open(input_file, "r", encoding="utf-8") as f:
        lines = f.readlines()
    
    lines.sort(key=key_func, reverse=reverse)
    
    with open(output_file, "w", encoding="utf-8") as f:
        f.writelines(lines)
    
    return len(lines)

# ทดสอบ
with open("unsorted.txt", "w") as f:
    f.write("banana\napple\ncherry\ndate\nfig\n")

n = sort_file_lines("unsorted.txt", "sorted.txt")
print(f"Sorted {n} lines")
with open("sorted.txt") as f:
    print(f.read())
```

### ข้อ 5: Config File Manager
```python
import json
from pathlib import Path

def load_config(filepath, defaults=None):
    """โหลด config โดยมี defaults"""
    config = defaults or {}
    path = Path(filepath)
    if path.exists():
        with open(path, "r") as f:
            user_config = json.load(f)
        config.update(user_config)
    else:
        save_config(filepath, config)
    return config

def save_config(filepath, config):
    with open(filepath, "w") as f:
        json.dump(config, f, indent=2)

defaults = {"theme": "light", "language": "en", "font_size": 14}
cfg = load_config("settings.json", defaults)
print("Config:", cfg)

cfg["theme"] = "dark"
save_config("settings.json", cfg)
print("Saved!")
```

### ข้อ 6: Multi-file Search
```python
from pathlib import Path

def search_files(directory, keyword, extensions=None):
    """ค้นหา keyword ในไฟล์หลายๆ ไฟล์"""
    extensions = extensions or [".txt", ".py", ".json", ".csv"]
    results = []
    
    for ext in extensions:
        for filepath in Path(directory).rglob(f"*{ext}"):
            try:
                with open(filepath, "r", encoding="utf-8", errors="ignore") as f:
                    for i, line in enumerate(f, 1):
                        if keyword in line:
                            results.append({
                                "file": str(filepath.name),
                                "line": i,
                                "text": line.strip()
                            })
            except Exception:
                pass
    return results

results = search_files(".", "Python", [".txt", ".md"])
print(f"Found {len(results)} matches:")
for r in results[:5]:
    print(f"  {r['file']}:{r['line']}: {r['text'][:60]}")
```

### ข้อ 7: CSV Pivot Table
```python
import csv
from collections import defaultdict

def pivot_table(csv_file, row_field, col_field, value_field, aggfunc=sum):
    """สร้าง pivot table จาก CSV"""
    data = defaultdict(lambda: defaultdict(list))
    
    with open(csv_file, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            r = row[row_field]
            c = row[col_field]
            v = float(row[value_field])
            data[r][c].append(v)
    
    pivot = {r: {c: aggfunc(vals) for c, vals in cols.items()}
             for r, cols in data.items()}
    return pivot

# สร้าง test data
with open("sales.csv", "w", newline="") as f:
    w = csv.writer(f)
    w.writerow(["region", "product", "sales"])
    w.writerows([
        ["North", "A", "1000"], ["North", "B", "2000"],
        ["South", "A", "1500"], ["South", "B", "1800"],
        ["North", "A", "900"], ["South", "B", "2200"],
    ])

pt = pivot_table("sales.csv", "region", "product", "sales")
print("Pivot table:")
for region, products in pt.items():
    for product, total in products.items():
        print(f"  {region} x {product}: {total:,.0f}")
```

### ข้อ 8: Batch File Renamer
```python
from pathlib import Path
import re

def batch_rename(directory, pattern, replacement, dry_run=True):
    """เปลี่ยนชื่อไฟล์แบบ batch"""
    renamed = []
    for filepath in Path(directory).iterdir():
        if filepath.is_file():
            new_name = re.sub(pattern, replacement, filepath.name)
            if new_name != filepath.name:
                renamed.append((filepath, filepath.parent / new_name))
    
    if dry_run:
        print("DRY RUN - จะเปลี่ยนชื่อ:")
        for old, new in renamed:
            print(f"  {old.name} → {new.name}")
    else:
        for old, new in renamed:
            old.rename(new)
        print(f"เปลี่ยนชื่อ {len(renamed)} ไฟล์")
    
    return renamed

# ทดสอบ (ดู dry run เท่านั้น)
# batch_rename(".", r"^\d+_", "", dry_run=True)
```

### ข้อ 9: JSON Database
```python
import json
from pathlib import Path
from datetime import datetime

class JSONDatabase:
    """Simple database ด้วย JSON"""
    
    def __init__(self, filepath):
        self.filepath = Path(filepath)
        self.data = self._load()
    
    def _load(self):
        if self.filepath.exists():
            with open(self.filepath, "r", encoding="utf-8") as f:
                return json.load(f)
        return {"records": [], "next_id": 1}
    
    def _save(self):
        with open(self.filepath, "w", encoding="utf-8") as f:
            json.dump(self.data, f, indent=2, ensure_ascii=False)
    
    def insert(self, record):
        record["_id"] = self.data["next_id"]
        record["_created"] = datetime.now().isoformat()
        self.data["records"].append(record)
        self.data["next_id"] += 1
        self._save()
        return record["_id"]
    
    def find(self, query=None):
        if query is None:
            return self.data["records"]
        return [r for r in self.data["records"]
                if all(r.get(k) == v for k, v in query.items())]
    
    def update(self, record_id, updates):
        for r in self.data["records"]:
            if r["_id"] == record_id:
                r.update(updates)
                r["_updated"] = datetime.now().isoformat()
                self._save()
                return True
        return False
    
    def delete(self, record_id):
        original_len = len(self.data["records"])
        self.data["records"] = [r for r in self.data["records"] if r["_id"] != record_id]
        if len(self.data["records"]) < original_len:
            self._save()
            return True
        return False

db = JSONDatabase("test_db.json")
id1 = db.insert({"name": "Alice", "age": 25, "dept": "Engineering"})
id2 = db.insert({"name": "Bob", "age": 30, "dept": "Marketing"})
id3 = db.insert({"name": "Charlie", "age": 28, "dept": "Engineering"})

print("All records:", len(db.find()))
print("Engineering:", [r["name"] for r in db.find({"dept": "Engineering"})])
db.update(id1, {"age": 26})
print("After update:", db.find({"name": "Alice"})[0]["age"])
db.delete(id2)
print("After delete:", len(db.find()))
```

### ข้อ 10: Report Generator
```python
import json
import csv
from datetime import datetime
from pathlib import Path

def generate_report(data_file, report_file):
    """สร้าง text report จาก JSON data"""
    with open(data_file, "r", encoding="utf-8") as f:
        data = json.load(f)
    
    with open(report_file, "w", encoding="utf-8") as f:
        # Header
        f.write("=" * 60 + "\n")
        f.write(f"SALES REPORT - {datetime.now().strftime('%Y-%m-%d %H:%M')}\n")
        f.write("=" * 60 + "\n\n")
        
        # Summary
        total = sum(item["amount"] for item in data)
        f.write(f"Total Sales: ฿{total:,.2f}\n")
        f.write(f"Total Orders: {len(data)}\n")
        f.write(f"Average Order: ฿{total/len(data):,.2f}\n\n")
        
        # Detail
        f.write("-" * 60 + "\n")
        f.write(f"{'Date':12} {'Product':15} {'Amount':>12}\n")
        f.write("-" * 60 + "\n")
        
        for item in sorted(data, key=lambda x: x["date"]):
            f.write(f"{item['date']:12} {item['product']:15} ฿{item['amount']:>10,.2f}\n")
        
        f.write("=" * 60 + "\n")

# ทดสอบ
test_data = [
    {"date": "2024-01-01", "product": "Widget A", "amount": 1500.00},
    {"date": "2024-01-02", "product": "Widget B", "amount": 2300.00},
    {"date": "2024-01-03", "product": "Widget A", "amount": 1800.00},
    {"date": "2024-01-04", "product": "Widget C", "amount": 900.00},
]

with open("sales_data.json", "w") as f:
    json.dump(test_data, f)

generate_report("sales_data.json", "sales_report.txt")
with open("sales_report.txt") as f:
    print(f.read())
```

---

## สรุป (Summary)

### File Modes

| Mode | Read | Write | Create | Truncate | Position |
|------|------|-------|--------|----------|----------|
| `r`  | ✓ | | | | ต้น |
| `w`  | | ✓ | ✓ | ✓ | ต้น |
| `a`  | | ✓ | ✓ | | ท้าย |
| `r+` | ✓ | ✓ | | | ต้น |
| `w+` | ✓ | ✓ | ✓ | ✓ | ต้น |
| `a+` | ✓ | ✓ | ✓ | | ท้าย |

### Best Practices

```python
# 1. ใช้ with statement เสมอ
with open("file.txt") as f:
    content = f.read()

# 2. ระบุ encoding ด้วยเสมอ (UTF-8)
with open("file.txt", encoding="utf-8") as f:
    content = f.read()

# 3. ใช้ pathlib สำหรับ path operations
from pathlib import Path
path = Path("dir") / "subdir" / "file.txt"

# 4. Handle exceptions
try:
    with open("file.txt") as f:
        content = f.read()
except FileNotFoundError:
    print("ไม่พบไฟล์")
except PermissionError:
    print("ไม่มีสิทธิ์เข้าถึง")
except IOError as e:
    print(f"Error: {e}")

# 5. ใช้ newline="" กับ csv
with open("file.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)

# 6. ใช้ ensure_ascii=False กับ json (สำหรับภาษาไทย)
with open("file.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
```

### เมื่อไหรใช้อะไร
| ประเภทข้อมูล | ใช้ |
|-------------|-----|
| ข้อความทั่วไป | `open()` ธรรมดา |
| ข้อมูลตาราง | `csv` module |
| Config/Data | `json` module |
| รูปภาพ/ไฟล์ไบนารี | Binary mode (`rb`/`wb`) |
| Path operations | `pathlib.Path` |
| Directory ops | `os` + `shutil` |

> **หมายเหตุ:** Part ต่อไปจะเรียน Functions ขั้นสูงและ Decorators
