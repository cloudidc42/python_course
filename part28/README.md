# Part 28: Context Managers & with Statement

## สารบัญ
1. [Context Manager Protocol](#context-manager-protocol)
2. [with Statement](#with-statement)
3. [Custom Context Managers (Class-based)](#custom-context-managers-class-based)
4. [contextlib.contextmanager Decorator](#contextlibcontextmanager-decorator)
5. [Multiple Context Managers](#multiple-context-managers)
6. [contextlib.ExitStack](#contextlibexitstack)
7. [contextlib.suppress](#contextlibsuppress)
8. [contextlib.redirect_stdout](#contextlibredirect_stdout)
9. [Reusable Context Managers](#reusable-context-managers)
10. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
11. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Context Manager Protocol

**Context Manager** คือ object ที่ implement protocol สองเมธอด:
- `__enter__(self)`: ทำงานเมื่อเข้า `with` block, return value ให้ `as` clause
- `__exit__(self, exc_type, exc_val, exc_tb)`: ทำงานเสมอเมื่อออกจาก `with` block ไม่ว่าจะมี exception หรือไม่

**วัตถุประสงค์**: รับรองว่า resources จะถูก cleanup อย่างถูกต้องเสมอ (เหมือน try/finally แต่ elegant กว่า)

```
__enter__() → with block → __exit__()
```

**พารามิเตอร์ของ __exit__:**
- `exc_type`: ประเภทของ exception (None ถ้าไม่มี)
- `exc_val`: ค่าของ exception (None ถ้าไม่มี)
- `exc_tb`: traceback object (None ถ้าไม่มี)
- **Return True**: suppress exception (กลืน error)
- **Return False/None**: propagate exception (ให้ error ผ่านไป)

### ตัวอย่าง 1: ทำความเข้าใจ Protocol

```python
class SimpleContext:
    """Context manager อย่างง่าย เพื่อเข้าใจ flow"""
    
    def __init__(self, name):
        self.name = name
        print(f"[{self.name}] __init__: สร้าง object")
    
    def __enter__(self):
        print(f"[{self.name}] __enter__: เข้า with block")
        return self  # ค่าที่ return มาจาก 'as'
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"[{self.name}] __exit__: ออกจาก with block")
        print(f"  exc_type={exc_type}, exc_val={exc_val}")
        return False  # ไม่กลืน exception

print("=== ไม่มี exception ===")
with SimpleContext("Demo") as ctx:
    print(f"  ภายใน with block, ctx={ctx.name}")

print("\n=== มี exception ===")
try:
    with SimpleContext("WithError") as ctx:
        print("  ทำงานปกติ")
        raise ValueError("เกิด error!")
        print("  บรรทัดนี้ไม่ทำงาน")
except ValueError as e:
    print(f"Exception ถูก propagate: {e}")
```

---

## with Statement

`with` statement เป็น syntactic sugar สำหรับ try/finally pattern ที่พบบ่อย

### ตัวอย่าง 2: เปรียบเทียบ with vs try/finally

```python
# วิธีแบบเก่า - ใช้ try/finally
file = open("test.txt", "w")
try:
    file.write("Hello, World!")
finally:
    file.close()  # ต้อง close เสมอ

# วิธีแบบใหม่ - ใช้ with (ง่ายกว่า, ปลอดภัยกว่า)
with open("test.txt", "w") as file:
    file.write("Hello, World!")
# file.close() ถูกเรียกอัตโนมัติ

# อ่านไฟล์ด้วย with
with open("test.txt", "r") as file:
    content = file.read()
print(content)

# ลบไฟล์ทดสอบ
import os
os.remove("test.txt")
```

### ตัวอย่าง 3: with Statement ทำงานอย่างไร

```python
# Python แปลง:
# with EXPR as VAR:
#     BLOCK
#
# เป็น:
# mgr = EXPR
# VAR = mgr.__enter__()
# try:
#     BLOCK
# except:
#     if not mgr.__exit__(*sys.exc_info()):
#         raise
# else:
#     mgr.__exit__(None, None, None)

# ตัวอย่าง: เมื่อ __exit__ return True จะกลืน exception
class SuppressValueError:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ValueError:
            print(f"กลืน ValueError: {exc_val}")
            return True  # suppress!
        return False  # propagate อื่นๆ

with SuppressValueError():
    print("ก่อน error")
    raise ValueError("error นี้จะถูกกลืน")
    print("บรรทัดนี้ไม่ทำงาน")

print("หลัง with block - โค้ดยังทำงานต่อได้")
```

---

## Custom Context Managers (Class-based)

### ตัวอย่าง 4: File Manager

```python
class ManagedFile:
    """Context manager สำหรับจัดการไฟล์ด้วย logging"""
    
    def __init__(self, filepath, mode='r', encoding='utf-8'):
        self.filepath = filepath
        self.mode = mode
        self.encoding = encoding
        self.file = None
    
    def __enter__(self):
        print(f"เปิดไฟล์: {self.filepath} (mode={self.mode})")
        self.file = open(self.filepath, self.mode, encoding=self.encoding)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            print(f"เกิด error ขณะใช้ไฟล์: {exc_type.__name__}: {exc_val}")
        
        if self.file and not self.file.closed:
            self.file.close()
            print(f"ปิดไฟล์: {self.filepath}")
        
        return False  # propagate exceptions

# ใช้งาน
with ManagedFile("demo.txt", "w") as f:
    f.write("บรรทัดที่ 1\n")
    f.write("บรรทัดที่ 2\n")

with ManagedFile("demo.txt", "r") as f:
    content = f.read()
    print(f"เนื้อหา:\n{content}")

import os
os.remove("demo.txt")
```

### ตัวอย่าง 5: Timer Context Manager

```python
import time

class Timer:
    """Context manager สำหรับวัดเวลา"""
    
    def __init__(self, name="", decimals=4):
        self.name = name
        self.decimals = decimals
        self.elapsed = None
        self.start_time = None
    
    def __enter__(self):
        self.start_time = time.perf_counter()
        return self  # return self เพื่อเข้าถึง elapsed ได้
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start_time
        label = f"[{self.name}] " if self.name else ""
        print(f"{label}เวลาที่ใช้: {self.elapsed:.{self.decimals}f}s")
        return False

# ใช้งาน
with Timer("การเรียงลำดับ") as t:
    import random
    data = [random.random() for _ in range(100000)]
    sorted_data = sorted(data)

print(f"elapsed attribute: {t.elapsed:.4f}s")

# nested timers
with Timer("ทั้งหมด"):
    with Timer("ส่วนที่ 1"):
        sum(range(1000000))
    with Timer("ส่วนที่ 2"):
        [x**2 for x in range(100000)]
```

### ตัวอย่าง 6: Database Transaction Context Manager

```python
import sqlite3
import os

class DatabaseTransaction:
    """Context manager สำหรับ database transaction"""
    
    def __init__(self, db_path):
        self.db_path = db_path
        self.conn = None
        self.cursor = None
    
    def __enter__(self):
        self.conn = sqlite3.connect(self.db_path)
        self.conn.row_factory = sqlite3.Row
        self.cursor = self.conn.cursor()
        self.conn.execute("BEGIN")
        print(f"เริ่ม transaction")
        return self.cursor
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            self.conn.commit()
            print("COMMIT transaction สำเร็จ")
        else:
            self.conn.rollback()
            print(f"ROLLBACK transaction เนื่องจาก: {exc_type.__name__}: {exc_val}")
        
        self.conn.close()
        return False  # propagate exceptions

# ตัวอย่างการใช้งาน
db_path = "/tmp/demo_transactions.db"

# เตรียม database
with DatabaseTransaction(db_path) as cursor:
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS accounts (
            id INTEGER PRIMARY KEY,
            name TEXT,
            balance REAL
        )
    """)
    cursor.execute("DELETE FROM accounts")
    cursor.executemany(
        "INSERT INTO accounts VALUES (?, ?, ?)",
        [(1, "สมชาย", 1000.0), (2, "สมหญิง", 500.0)]
    )

# Successful transaction
print("\n=== Transaction ที่สำเร็จ ===")
with DatabaseTransaction(db_path) as cursor:
    cursor.execute("UPDATE accounts SET balance = balance - 200 WHERE id = 1")
    cursor.execute("UPDATE accounts SET balance = balance + 200 WHERE id = 2")

# Failed transaction (will rollback)
print("\n=== Transaction ที่ล้มเหลว ===")
try:
    with DatabaseTransaction(db_path) as cursor:
        cursor.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
        raise ValueError("เกิดข้อผิดพลาด!")
        cursor.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
except ValueError:
    pass

# ตรวจสอบผลลัพธ์
with DatabaseTransaction(db_path) as cursor:
    cursor.execute("SELECT * FROM accounts")
    rows = cursor.fetchall()
    print("\nยอดเงินหลัง transactions:")
    for row in rows:
        print(f"  {row['name']}: {row['balance']:.2f}")

os.remove(db_path)
```

### ตัวอย่าง 7: Lock Context Manager

```python
import threading
import time

class RWLock:
    """Read-Write Lock context manager"""
    
    def __init__(self):
        self._read_lock = threading.Lock()
        self._write_lock = threading.Lock()
        self._readers = 0
    
    class ReadContext:
        def __init__(self, rw_lock):
            self.rw_lock = rw_lock
        
        def __enter__(self):
            with self.rw_lock._read_lock:
                self.rw_lock._readers += 1
                if self.rw_lock._readers == 1:
                    self.rw_lock._write_lock.acquire()
            return self
        
        def __exit__(self, *args):
            with self.rw_lock._read_lock:
                self.rw_lock._readers -= 1
                if self.rw_lock._readers == 0:
                    self.rw_lock._write_lock.release()
    
    class WriteContext:
        def __init__(self, rw_lock):
            self.rw_lock = rw_lock
        
        def __enter__(self):
            self.rw_lock._write_lock.acquire()
            return self
        
        def __exit__(self, *args):
            self.rw_lock._write_lock.release()
    
    def reader(self):
        return self.ReadContext(self)
    
    def writer(self):
        return self.WriteContext(self)

# ใช้งาน
data = {"value": 0}
lock = RWLock()

with lock.writer():
    data["value"] = 100
    print(f"เขียนค่า: {data['value']}")

with lock.reader():
    print(f"อ่านค่า: {data['value']}")
```

---

## contextlib.contextmanager Decorator

วิธีที่ง่ายกว่าการสร้าง class คือใช้ `@contextlib.contextmanager` กับ generator function

โค้ดก่อน `yield` เหมือน `__enter__`, โค้ดหลัง `yield` เหมือน `__exit__`

### ตัวอย่าง 8: contextmanager พื้นฐาน

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(name=""):
    """Timer context manager ด้วย generator"""
    start = time.perf_counter()
    print(f"{'['+name+'] ' if name else ''}เริ่มจับเวลา")
    
    try:
        yield  # ส่งการควบคุมกลับไปที่ with block
    finally:
        # finally block ทำงานเสมอ (เหมือน __exit__)
        elapsed = time.perf_counter() - start
        print(f"{'['+name+'] ' if name else ''}ใช้เวลา: {elapsed:.4f}s")

with timer("การคำนวณ"):
    result = sum(range(1000000))
    print(f"ผลลัพธ์: {result}")
```

### ตัวอย่าง 9: contextmanager พร้อม yield value

```python
from contextlib import contextmanager
import os
import tempfile

@contextmanager
def temp_directory():
    """สร้าง temporary directory และลบเมื่อใช้เสร็จ"""
    tmpdir = tempfile.mkdtemp()
    print(f"สร้าง temp dir: {tmpdir}")
    
    try:
        yield tmpdir  # ส่ง path กลับไปให้ as clause
    finally:
        import shutil
        shutil.rmtree(tmpdir, ignore_errors=True)
        print(f"ลบ temp dir: {tmpdir}")

@contextmanager
def temp_file(suffix=".txt", mode="w"):
    """สร้าง temporary file"""
    fd, path = tempfile.mkstemp(suffix=suffix)
    
    try:
        with os.fdopen(fd, mode) as f:
            yield f, path  # yield tuple
    finally:
        try:
            os.unlink(path)
            print(f"ลบ temp file: {path}")
        except FileNotFoundError:
            pass

# ใช้งาน
with temp_directory() as tmpdir:
    test_file = os.path.join(tmpdir, "test.txt")
    with open(test_file, "w") as f:
        f.write("ข้อมูลชั่วคราว")
    print(f"ไฟล์ในโฟลเดอร์: {os.listdir(tmpdir)}")

print("หลัง with block - folder ถูกลบแล้ว")
```

### ตัวอย่าง 10: contextmanager สำหรับ Error Handling

```python
from contextlib import contextmanager

@contextmanager
def handled_errors(*exception_types, log=True, reraise=True):
    """Context manager สำหรับ handle errors"""
    try:
        yield
    except exception_types as e:
        if log:
            print(f"[Error] {type(e).__name__}: {e}")
        if reraise:
            raise
    except Exception as e:
        print(f"[Unexpected Error] {type(e).__name__}: {e}")
        raise

@contextmanager
def suppress_and_log(*exception_types):
    """กลืน exception แต่ log ไว้"""
    try:
        yield
    except exception_types as e:
        print(f"[Suppressed] {type(e).__name__}: {e}")

# ใช้งาน
with suppress_and_log(ValueError, TypeError):
    x = int("not a number")  # ValueError - ถูกกลืน
print("โค้ดยังทำงานต่อได้")

with suppress_and_log(FileNotFoundError):
    with open("nonexistent.txt") as f:
        content = f.read()
print("ไม่มี crash แม้ไฟล์ไม่มี")
```

---

## Multiple Context Managers

### ตัวอย่าง 11: Multiple Context Managers ใน with เดียว

```python
from contextlib import contextmanager

@contextmanager
def logger(name):
    print(f"[{name}] เข้า")
    try:
        yield name
    finally:
        print(f"[{name}] ออก")

# ใช้ comma ใน with statement (Python 2.7+)
with logger("A") as a, logger("B") as b, logger("C") as c:
    print(f"  ทำงานใน {a}, {b}, {c}")

# เทียบเท่ากับ nested with
print("---")
with logger("A") as a:
    with logger("B") as b:
        with logger("C") as c:
            print(f"  ทำงานใน {a}, {b}, {c}")
```

### ตัวอย่าง 12: เปิดหลายไฟล์พร้อมกัน

```python
import os

# สร้างไฟล์ทดสอบ
for i in range(3):
    with open(f"/tmp/file_{i}.txt", "w") as f:
        f.write(f"ข้อมูลในไฟล์ {i}\n" * 3)

# เปิดหลายไฟล์พร้อมกัน
with (open("/tmp/file_0.txt") as f1,
      open("/tmp/file_1.txt") as f2,
      open("/tmp/file_2.txt") as f3):
    
    combined = ""
    for line in f1:
        combined += line
    for line in f2:
        combined += line
    for line in f3:
        combined += line
    
    print(f"อ่านได้ {len(combined.splitlines())} บรรทัด")

# ลบไฟล์
for i in range(3):
    os.remove(f"/tmp/file_{i}.txt")
```

---

## contextlib.ExitStack

`ExitStack` ทำให้จัดการ context managers แบบ dynamic ได้ (จำนวน context managers ไม่คงที่)

### ตัวอย่าง 13: ExitStack พื้นฐาน

```python
from contextlib import ExitStack, contextmanager

@contextmanager
def managed_resource(name):
    print(f"เปิด {name}")
    try:
        yield name
    finally:
        print(f"ปิด {name}")

# เปิดไฟล์จำนวน dynamic
filenames = [f"/tmp/test_{i}.txt" for i in range(5)]

# สร้างไฟล์
for fn in filenames:
    with open(fn, "w") as f:
        f.write(f"Content of {fn}")

# ExitStack จัดการหลาย context managers
with ExitStack() as stack:
    files = [
        stack.enter_context(open(fn))
        for fn in filenames
    ]
    
    for i, f in enumerate(files):
        print(f"ไฟล์ {i}: {f.read()}")
# ทุกไฟล์ถูกปิดอัตโนมัติเมื่อออกจาก with

import os
for fn in filenames:
    os.remove(fn)
```

### ตัวอย่าง 14: ExitStack กับ Dynamic Resources

```python
from contextlib import ExitStack, contextmanager
import sqlite3
import os

@contextmanager
def db_connection(path):
    """Context manager สำหรับ database connection"""
    conn = sqlite3.connect(path)
    print(f"เชื่อมต่อ DB: {path}")
    try:
        yield conn
    finally:
        conn.close()
        print(f"ปิด DB: {path}")

def process_databases(db_paths):
    """ประมวลผลหลาย databases"""
    with ExitStack() as stack:
        # เปิดทุก connection
        connections = {
            path: stack.enter_context(db_connection(path))
            for path in db_paths
        }
        
        # ทำงานกับ connections
        results = {}
        for path, conn in connections.items():
            cursor = conn.cursor()
            cursor.execute("SELECT sqlite_version()")
            results[path] = cursor.fetchone()[0]
        
        return results
    # ทุก connections ถูกปิดอัตโนมัติ

# สร้าง databases ทดสอบ
db_paths = [f"/tmp/db_{i}.db" for i in range(3)]
for path in db_paths:
    conn = sqlite3.connect(path)
    conn.close()

results = process_databases(db_paths)
for path, version in results.items():
    print(f"  {os.path.basename(path)}: SQLite {version}")

for path in db_paths:
    os.remove(path)
```

### ตัวอย่าง 15: ExitStack สำหรับ Cleanup Callbacks

```python
from contextlib import ExitStack
import os

def cleanup_callback(message):
    print(f"Cleanup: {message}")

with ExitStack() as stack:
    # register callbacks ที่จะถูกเรียกเมื่อออกจาก with
    stack.callback(cleanup_callback, "ขั้นตอนที่ 1")
    stack.callback(cleanup_callback, "ขั้นตอนที่ 2")
    stack.callback(cleanup_callback, "ขั้นตอนที่ 3")
    
    print("ทำงานภายใน with block")
    # Callbacks ถูกเรียกตามลำดับ LIFO (ล่างขึ้นบน)
# Output:
# Cleanup: ขั้นตอนที่ 3
# Cleanup: ขั้นตอนที่ 2
# Cleanup: ขั้นตอนที่ 1
```

---

## contextlib.suppress

`suppress` กลืน exceptions ที่กำหนดโดยไม่ต้องเขียน try/except

### ตัวอย่าง 16: suppress

```python
from contextlib import suppress
import os

# วิธีเก่า
try:
    os.remove("nonexistent.txt")
except FileNotFoundError:
    pass  # ไม่สนใจ error นี้

# วิธีใหม่ด้วย suppress
with suppress(FileNotFoundError):
    os.remove("nonexistent.txt")

# suppress หลาย exception types
with suppress(FileNotFoundError, PermissionError):
    os.remove("some_file.txt")

# ตัวอย่างจริง: dict key ที่อาจไม่มี
data = {"a": 1, "b": 2}

with suppress(KeyError):
    del data["c"]  # ไม่ crash ถ้า key ไม่มี
    del data["d"]  # บรรทัดนี้ไม่ทำงานถ้า KeyError เกิดจากบรรทัดก่อน

print(data)  # {"a": 1, "b": 2}

# suppress ใช้ใน loop เพื่อข้ามรายการที่มีปัญหา
values = ["1", "2", "abc", "4", "xyz", "6"]
result = []
for v in values:
    with suppress(ValueError):
        result.append(int(v))
print(result)  # [1, 2, 4, 6]
```

---

## contextlib.redirect_stdout

### ตัวอย่าง 17: redirect_stdout และ redirect_stderr

```python
from contextlib import redirect_stdout, redirect_stderr
import io

# Capture stdout
captured = io.StringIO()
with redirect_stdout(captured):
    print("Hello!")
    print("This goes to captured")
    for i in range(3):
        print(f"Line {i}")

output = captured.getvalue()
print(f"Captured output ({len(output)} chars):")
print(repr(output))

# Redirect stdout ไปยังไฟล์
with open("/tmp/output.txt", "w") as f:
    with redirect_stdout(f):
        print("This goes to file")
        help(len)  # redirect help() output

with open("/tmp/output.txt") as f:
    lines = f.readlines()
print(f"บันทึก {len(lines)} บรรทัดลงไฟล์")

import os
os.remove("/tmp/output.txt")
```

### ตัวอย่าง 18: nullcontext (Python 3.7+)

```python
from contextlib import nullcontext

def process_data(data, lock=None):
    """ฟังก์ชันที่ใช้ lock ถ้ามี"""
    cm = lock if lock is not None else nullcontext()
    
    with cm:
        return [x * 2 for x in data]

import threading

# ใช้กับ lock
lock = threading.Lock()
result1 = process_data([1, 2, 3], lock)

# ใช้โดยไม่มี lock (nullcontext เป็น no-op)
result2 = process_data([4, 5, 6])

print(result1)  # [2, 4, 6]
print(result2)  # [8, 10, 12]
```

---

## Reusable Context Managers

Context managers บางตัวสามารถใช้ซ้ำได้ (reentrant) บางตัวไม่ได้

### ตัวอย่าง 19: Reentrant Context Manager

```python
from contextlib import contextmanager
import threading

class ReentrantLock:
    """Context manager ที่ใช้ซ้ำได้จาก thread เดิม"""
    
    def __init__(self):
        self._lock = threading.RLock()
        self._depth = 0
    
    def __enter__(self):
        self._lock.acquire()
        self._depth += 1
        print(f"Lock acquired (depth={self._depth})")
        return self
    
    def __exit__(self, *args):
        self._depth -= 1
        print(f"Lock released (depth={self._depth})")
        self._lock.release()
        return False

lock = ReentrantLock()

with lock:
    print("ระดับ 1")
    with lock:  # ใช้ซ้ำได้ (reentrant)
        print("ระดับ 2")
        with lock:
            print("ระดับ 3")
```

### ตัวอย่าง 20: Context Manager พร้อม State

```python
from contextlib import contextmanager
from typing import Optional

class ConnectionPool:
    """Connection pool ที่ใช้ context manager"""
    
    def __init__(self, max_connections=5):
        self.max_connections = max_connections
        self._available = list(range(1, max_connections + 1))
        self._in_use = []
        self._lock = __import__('threading').Lock()
    
    @contextmanager
    def acquire(self, timeout=5.0):
        """ขอ connection จาก pool"""
        import time
        
        deadline = time.time() + timeout
        conn_id = None
        
        # รอ connection ว่าง
        while time.time() < deadline:
            with self._lock:
                if self._available:
                    conn_id = self._available.pop(0)
                    self._in_use.append(conn_id)
                    break
            time.sleep(0.01)
        
        if conn_id is None:
            raise TimeoutError("ไม่มี connection ว่างใน timeout")
        
        print(f"ได้รับ connection #{conn_id} "
              f"(ว่าง={len(self._available)}, ใช้อยู่={len(self._in_use)})")
        
        try:
            yield conn_id
        finally:
            with self._lock:
                self._in_use.remove(conn_id)
                self._available.append(conn_id)
            print(f"คืน connection #{conn_id} "
                  f"(ว่าง={len(self._available)}, ใช้อยู่={len(self._in_use)})")
    
    def stats(self):
        return {
            "available": len(self._available),
            "in_use": len(self._in_use),
            "total": self.max_connections
        }

pool = ConnectionPool(max_connections=3)

with pool.acquire() as conn1:
    print(f"ใช้ conn #{conn1}")
    with pool.acquire() as conn2:
        print(f"ใช้ conn #{conn2}")

print(f"สถิติ: {pool.stats()}")
```

---

## ตัวอย่างโปรแกรมจริง

### ตัวอย่าง 21: Database Connection Manager สมบูรณ์

```python
import sqlite3
import os
from contextlib import contextmanager
from typing import Optional

class Database:
    """Database class พร้อม context managers สำหรับทุกการดำเนินการ"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self._connection = None
    
    @contextmanager
    def connection(self):
        """จัดการ connection lifecycle"""
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        
        try:
            yield conn
        finally:
            conn.close()
    
    @contextmanager
    def transaction(self):
        """จัดการ transaction พร้อม auto commit/rollback"""
        with self.connection() as conn:
            try:
                yield conn
                conn.commit()
                print("Transaction committed")
            except Exception as e:
                conn.rollback()
                print(f"Transaction rolled back: {e}")
                raise
    
    @contextmanager
    def cursor(self, commit=False):
        """จัดการ cursor lifecycle"""
        with self.connection() as conn:
            cursor = conn.cursor()
            try:
                yield cursor
                if commit:
                    conn.commit()
            except Exception:
                conn.rollback()
                raise
    
    def init_schema(self):
        with self.transaction() as conn:
            conn.executescript("""
                CREATE TABLE IF NOT EXISTS products (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL,
                    price REAL NOT NULL,
                    stock INTEGER DEFAULT 0
                );
                
                CREATE TABLE IF NOT EXISTS orders (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    product_id INTEGER,
                    quantity INTEGER,
                    total REAL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (product_id) REFERENCES products(id)
                );
            """)

db_path = "/tmp/shop.db"
db = Database(db_path)
db.init_schema()

# เพิ่มสินค้า
with db.transaction() as conn:
    conn.executemany(
        "INSERT OR REPLACE INTO products (id, name, price, stock) VALUES (?, ?, ?, ?)",
        [(1, "กล้วย", 10.0, 100), (2, "แอปเปิ้ล", 25.0, 50)]
    )

# สร้าง order (transaction)
with db.transaction() as conn:
    conn.execute("UPDATE products SET stock = stock - 5 WHERE id = 1")
    conn.execute(
        "INSERT INTO orders (product_id, quantity, total) VALUES (?, ?, ?)",
        (1, 5, 50.0)
    )

# อ่านข้อมูล
with db.cursor() as cursor:
    cursor.execute("SELECT * FROM products")
    products = cursor.fetchall()
    for p in products:
        print(f"  {p['name']}: {p['price']}฿ (stock: {p['stock']})")

os.remove(db_path)
```

### ตัวอย่าง 22: File Management Context Manager

```python
import os
import shutil
import tempfile
from contextlib import contextmanager

@contextmanager
def safe_write(filepath, backup=True):
    """เขียนไฟล์อย่างปลอดภัย - backup เดิม, ใช้ temp file"""
    dirpath = os.path.dirname(os.path.abspath(filepath))
    backup_path = filepath + ".bak"
    
    # backup ไฟล์เดิม
    if backup and os.path.exists(filepath):
        shutil.copy2(filepath, backup_path)
        print(f"Backup: {backup_path}")
    
    # เขียนลง temp file ก่อน
    fd, temp_path = tempfile.mkstemp(dir=dirpath)
    
    try:
        with os.fdopen(fd, 'w', encoding='utf-8') as temp_file:
            yield temp_file
        
        # ถ้าสำเร็จ: เปลี่ยนชื่อ temp file เป็นชื่อจริง
        if os.path.exists(filepath):
            os.remove(filepath)
        os.rename(temp_path, filepath)
        
        # ลบ backup ถ้าสำเร็จ
        if backup and os.path.exists(backup_path):
            os.remove(backup_path)
        
        print(f"เขียนไฟล์สำเร็จ: {filepath}")
        
    except Exception as e:
        # ถ้าล้มเหลว: ลบ temp file
        if os.path.exists(temp_path):
            os.remove(temp_path)
        
        # restore จาก backup
        if backup and os.path.exists(backup_path):
            shutil.copy2(backup_path, filepath)
            os.remove(backup_path)
            print(f"Restored from backup")
        
        raise

# ใช้งาน
test_path = "/tmp/important_file.txt"

# เขียนครั้งแรก
with safe_write(test_path, backup=False) as f:
    f.write("เนื้อหาเดิม\n")

# แก้ไขอย่างปลอดภัย
with safe_write(test_path) as f:
    f.write("เนื้อหาใหม่\n")
    f.write("บรรทัดที่สอง\n")

with open(test_path) as f:
    print(f.read())

os.remove(test_path)
```

### ตัวอย่าง 23: Timing Context Manager สมบูรณ์

```python
import time
import statistics
from contextlib import contextmanager
from dataclasses import dataclass, field
from typing import List

@dataclass
class TimingStats:
    name: str
    elapsed_times: List[float] = field(default_factory=list)
    
    def record(self, elapsed: float):
        self.elapsed_times.append(elapsed)
    
    @property
    def count(self):
        return len(self.elapsed_times)
    
    @property
    def total(self):
        return sum(self.elapsed_times)
    
    @property
    def mean(self):
        return statistics.mean(self.elapsed_times) if self.elapsed_times else 0
    
    @property
    def stdev(self):
        if len(self.elapsed_times) < 2:
            return 0
        return statistics.stdev(self.elapsed_times)
    
    @property
    def min_time(self):
        return min(self.elapsed_times) if self.elapsed_times else 0
    
    @property
    def max_time(self):
        return max(self.elapsed_times) if self.elapsed_times else 0
    
    def report(self):
        print(f"=== Timing Report: {self.name} ===")
        print(f"  จำนวนครั้ง: {self.count}")
        print(f"  รวม: {self.total:.4f}s")
        print(f"  เฉลี่ย: {self.mean*1000:.3f}ms")
        print(f"  Std Dev: {self.stdev*1000:.3f}ms")
        print(f"  Min: {self.min_time*1000:.3f}ms")
        print(f"  Max: {self.max_time*1000:.3f}ms")

class BenchmarkContext:
    """Context manager สำหรับ benchmark"""
    
    _stats: dict = {}
    
    def __init__(self, name: str, repeat: int = 1):
        self.name = name
        self.repeat = repeat
        if name not in self._stats:
            self._stats[name] = TimingStats(name)
    
    def __enter__(self):
        self._start = time.perf_counter()
        return self
    
    def __exit__(self, *args):
        elapsed = time.perf_counter() - self._start
        self._stats[self.name].record(elapsed)
        return False
    
    @classmethod
    def report_all(cls):
        for stats in cls._stats.values():
            stats.report()

# ใช้งาน
for _ in range(5):
    with BenchmarkContext("sort operation"):
        import random
        data = [random.random() for _ in range(10000)]
        sorted(data)

with BenchmarkContext("sum operation"):
    sum(range(1000000))

BenchmarkContext.report_all()
```

### ตัวอย่าง 24: Transaction Context Manager สำหรับ In-Memory State

```python
from contextlib import contextmanager
import copy

class TransactionalDict:
    """Dictionary ที่รองรับ transactions"""
    
    def __init__(self, data=None):
        self._data = dict(data or {})
        self._snapshots = []
    
    @contextmanager
    def transaction(self):
        """ทำ transaction - rollback ถ้า error"""
        # เก็บ snapshot ก่อน
        snapshot = copy.deepcopy(self._data)
        self._snapshots.append(snapshot)
        
        try:
            yield self
            self._snapshots.pop()  # สำเร็จ - ลบ snapshot
            print("Transaction committed")
        except Exception as e:
            # Rollback
            self._data = self._snapshots.pop()
            print(f"Transaction rolled back: {e}")
            raise
    
    def __setitem__(self, key, value):
        self._data[key] = value
    
    def __getitem__(self, key):
        return self._data[key]
    
    def __repr__(self):
        return f"TransactionalDict({self._data})"

state = TransactionalDict({"balance": 1000, "name": "สมชาย"})
print(f"เริ่มต้น: {state}")

# Transaction ที่สำเร็จ
with state.transaction():
    state["balance"] -= 200
    state["last_transaction"] = "debit_200"

print(f"หลัง transaction: {state}")

# Transaction ที่ล้มเหลว
try:
    with state.transaction():
        state["balance"] -= 5000
        if state["balance"] < 0:
            raise ValueError("ยอดเงินไม่พอ!")
except ValueError:
    pass

print(f"หลัง rollback: {state}")
```

### ตัวอย่าง 25: Configuration Context Manager

```python
from contextlib import contextmanager
from typing import Any, Dict

class Config:
    """Configuration manager พร้อม context manager สำหรับ temporary overrides"""
    
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._config = {}
            cls._instance._stack = []
        return cls._instance
    
    def set(self, key: str, value: Any):
        self._config[key] = value
    
    def get(self, key: str, default=None):
        return self._config.get(key, default)
    
    @contextmanager
    def override(self, **overrides):
        """Temporarily override config values"""
        # เก็บค่าเดิม
        old_values = {k: self._config.get(k) for k in overrides}
        existed = {k: k in self._config for k in overrides}
        
        # ใส่ค่าใหม่
        self._config.update(overrides)
        
        try:
            yield self
        finally:
            # restore ค่าเดิม
            for key in overrides:
                if existed[key]:
                    self._config[key] = old_values[key]
                elif key in self._config:
                    del self._config[key]

config = Config()
config.set("env", "production")
config.set("debug", False)
config.set("db_url", "postgres://prod-server/db")

print("Config ปกติ:")
print(f"  env: {config.get('env')}")
print(f"  debug: {config.get('debug')}")

# Override ชั่วคราวสำหรับ testing
with config.override(env="testing", debug=True, db_url="sqlite:///test.db"):
    print("\nConfig ใน test context:")
    print(f"  env: {config.get('env')}")
    print(f"  debug: {config.get('debug')}")
    print(f"  db_url: {config.get('db_url')}")

print("\nConfig หลัง override:")
print(f"  env: {config.get('env')}")
print(f"  debug: {config.get('debug')}")
print(f"  db_url: {config.get('db_url')}")
```

---

## ตัวอย่างโปรแกรมจริง (เพิ่มเติม)

### ตัวอย่าง 26: HTTP Session Context Manager

```python
from contextlib import contextmanager
import time

class MockHTTPSession:
    """จำลอง HTTP session"""
    
    def __init__(self, base_url, timeout=30, retries=3):
        self.base_url = base_url
        self.timeout = timeout
        self.retries = retries
        self._is_open = False
        self._request_count = 0
    
    def open(self):
        print(f"เปิด session ไปยัง {self.base_url}")
        self._is_open = True
    
    def close(self):
        print(f"ปิด session (ส่ง {self._request_count} requests)")
        self._is_open = False
    
    def get(self, path, **kwargs):
        if not self._is_open:
            raise RuntimeError("Session ไม่ได้เปิดอยู่")
        self._request_count += 1
        return {"status": 200, "data": f"Response from {self.base_url}{path}"}
    
    def __enter__(self):
        self.open()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
        return False

@contextmanager  
def http_session(base_url, **kwargs):
    """Factory function สำหรับ HTTP session"""
    session = MockHTTPSession(base_url, **kwargs)
    session.open()
    try:
        yield session
    finally:
        session.close()

# ใช้ class-based
with MockHTTPSession("https://api.example.com") as session:
    response = session.get("/users")
    print(f"Response: {response}")

# ใช้ contextmanager
with http_session("https://api2.example.com") as session:
    r1 = session.get("/products")
    r2 = session.get("/orders")
    print(f"Products: {r1['status']}")
    print(f"Orders: {r2['status']}")
```

### ตัวอย่าง 27: Profiling Context Manager

```python
import cProfile
import pstats
import io
from contextlib import contextmanager

@contextmanager
def profile(sort_by='cumulative', lines=10):
    """Context manager สำหรับ profiling"""
    pr = cProfile.Profile()
    pr.enable()
    
    try:
        yield pr
    finally:
        pr.disable()
        
        s = io.StringIO()
        ps = pstats.Stats(pr, stream=s).sort_stats(sort_by)
        ps.print_stats(lines)
        print(s.getvalue())

@contextmanager
def memory_profile():
    """Profile memory usage"""
    import tracemalloc
    tracemalloc.start()
    
    try:
        yield
    finally:
        current, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        print(f"Memory - Current: {current/1024:.1f}KB, Peak: {peak/1024:.1f}KB")

# ใช้งาน
def intensive_task():
    return [i**2 for i in range(10000)]

with memory_profile():
    result = intensive_task()
    more_data = list(range(100000))

print(f"สร้างข้อมูล {len(result)} + {len(more_data)} items")
```

### ตัวอย่าง 28: Environment Context Manager

```python
import os
from contextlib import contextmanager

@contextmanager
def env_vars(**kwargs):
    """Temporarily set environment variables"""
    old_values = {}
    existed = {}
    
    for key, value in kwargs.items():
        old_values[key] = os.environ.get(key)
        existed[key] = key in os.environ
        os.environ[key] = str(value)
    
    try:
        yield
    finally:
        for key in kwargs:
            if existed[key]:
                os.environ[key] = old_values[key]
            elif key in os.environ:
                del os.environ[key]

# ใช้งาน
print(f"ก่อน: DATABASE_URL = {os.environ.get('DATABASE_URL', 'ไม่มี')}")

with env_vars(DATABASE_URL="sqlite:///test.db", DEBUG="true"):
    print(f"ใน context: DATABASE_URL = {os.environ.get('DATABASE_URL')}")
    print(f"ใน context: DEBUG = {os.environ.get('DEBUG')}")

print(f"หลัง: DATABASE_URL = {os.environ.get('DATABASE_URL', 'ไม่มี')}")
print(f"หลัง: DEBUG = {os.environ.get('DEBUG', 'ไม่มี')}")
```

### ตัวอย่าง 29: Atomic Operations Context Manager

```python
import os
import shutil
import tempfile
from contextlib import contextmanager

@contextmanager
def atomic_directory_update(target_dir):
    """
    อัพเดท directory อย่าง atomic:
    1. สร้าง temp dir ใหม่
    2. ทำงานใน temp dir
    3. ถ้าสำเร็จ: swap temp กับ target
    4. ถ้าล้มเหลว: ลบ temp
    """
    parent = os.path.dirname(target_dir)
    tmp_dir = tempfile.mkdtemp(dir=parent)
    backup_dir = target_dir + ".backup"
    
    try:
        yield tmp_dir  # ทำงานใน temp dir
        
        # Atomic swap
        if os.path.exists(target_dir):
            os.rename(target_dir, backup_dir)
        os.rename(tmp_dir, target_dir)
        
        # ลบ backup
        if os.path.exists(backup_dir):
            shutil.rmtree(backup_dir)
        
        print(f"อัพเดท directory สำเร็จ: {target_dir}")
        
    except Exception as e:
        # ล้มเหลว: cleanup temp
        if os.path.exists(tmp_dir):
            shutil.rmtree(tmp_dir)
        # restore จาก backup
        if os.path.exists(backup_dir):
            os.rename(backup_dir, target_dir)
        print(f"ล้มเหลว - restored: {e}")
        raise

# ตัวอย่างการใช้งาน
target = "/tmp/website_files"
os.makedirs(target, exist_ok=True)

# สร้างไฟล์เดิม
with open(os.path.join(target, "index.html"), "w") as f:
    f.write("<html>Old content</html>")

print(f"ไฟล์เดิม: {os.listdir(target)}")

with atomic_directory_update(target) as tmp:
    # สร้างไฟล์ใหม่ใน tmp dir
    with open(os.path.join(tmp, "index.html"), "w") as f:
        f.write("<html>New content</html>")
    with open(os.path.join(tmp, "style.css"), "w") as f:
        f.write("body { color: red; }")

print(f"ไฟล์ใหม่: {os.listdir(target)}")

shutil.rmtree(target)
```

### ตัวอย่าง 30: Logging Context Manager

```python
import logging
import sys
from contextlib import contextmanager

@contextmanager
def log_context(name, level=logging.DEBUG, capture=False):
    """
    Context manager สำหรับ logging:
    - เพิ่ม context info ใน log messages
    - optionally capture log output
    """
    logger = logging.getLogger(name)
    old_level = logger.level
    logger.setLevel(level)
    
    # สร้าง handler สำหรับ capture
    if capture:
        handler = logging.StreamHandler(sys.stdout)
        handler.setLevel(level)
        formatter = logging.Formatter('%(asctime)s [%(levelname)s] %(name)s: %(message)s')
        handler.setFormatter(formatter)
        logger.addHandler(handler)
    
    extra_filter = logging.Filter()
    
    try:
        yield logger
    finally:
        logger.setLevel(old_level)
        if capture:
            logger.removeHandler(handler)

# ใช้งาน
logging.basicConfig(level=logging.WARNING)

with log_context("myapp.database", level=logging.DEBUG, capture=True) as logger:
    logger.debug("เชื่อมต่อ database")
    logger.info("Query สำเร็จ")
    logger.warning("Connection pool ใกล้เต็ม")

print("\n--- นอก context ---")
# logger นอก context ไม่แสดง DEBUG/INFO
outer_logger = logging.getLogger("myapp.database")
outer_logger.debug("Message นี้ไม่แสดง")
print("ทำงานต่อปกติ")
```

---

## แบบฝึกหัด

### ข้อ 1: สร้าง Mutex Context Manager

สร้าง context manager สำหรับ mutual exclusion ที่ timeout ได้

**คำตอบ:**

```python
import threading
import time
from contextlib import contextmanager

@contextmanager
def mutex(lock, timeout=None):
    """Context manager สำหรับ mutex lock พร้อม timeout"""
    acquired = lock.acquire(timeout=timeout if timeout else -1)
    if not acquired:
        raise TimeoutError(f"ไม่สามารถ acquire lock ภายใน {timeout}s")
    try:
        yield
    finally:
        lock.release()

my_lock = threading.Lock()

with mutex(my_lock, timeout=5.0):
    print("ได้ lock แล้ว - ทำงานใน critical section")
    time.sleep(0.01)  # จำลองงาน

print("ปล่อย lock แล้ว")
```

### ข้อ 2: สร้าง Stopwatch Context Manager

สร้าง context manager ที่วัดเวลา lap ได้

**คำตอบ:**

```python
import time
from contextlib import contextmanager

class Stopwatch:
    def __init__(self):
        self.laps = []
        self._start = None
        self._lap_start = None
    
    def __enter__(self):
        self._start = time.perf_counter()
        self._lap_start = self._start
        return self
    
    def lap(self, name=""):
        now = time.perf_counter()
        elapsed = now - self._lap_start
        self.laps.append((name or f"Lap {len(self.laps)+1}", elapsed))
        self._lap_start = now
        return elapsed
    
    def __exit__(self, *args):
        self.total = time.perf_counter() - self._start
        print(f"\nStopwatch Results:")
        for name, t in self.laps:
            print(f"  {name}: {t*1000:.2f}ms")
        print(f"  Total: {self.total*1000:.2f}ms")

with Stopwatch() as sw:
    time.sleep(0.01)
    sw.lap("Phase 1")
    time.sleep(0.02)
    sw.lap("Phase 2")
    time.sleep(0.015)
    sw.lap("Phase 3")
```

### ข้อ 3-10: แบบฝึกหัดเพิ่มเติม

**ข้อ 3**: สร้าง `@retry_on_exception` context manager

**ข้อ 4**: สร้าง `working_directory` context manager ที่ cd และ cd กลับ

**ข้อ 5**: สร้าง `json_config` context manager ที่อ่าน JSON, yield, และ save กลับ

**ข้อ 6**: สร้าง `MockPatch` context manager สำหรับ monkey-patching

**ข้อ 7**: สร้าง `record_sql` context manager ที่ log SQL queries

**ข้อ 8**: สร้าง `stdin_redirect` context manager ที่ inject input

**ข้อ 9**: สร้าง `rate_limiter` context manager ที่ throttle code execution

**ข้อ 10**: สร้าง hierarchical context manager ที่ nest ได้และ share state

---

## สรุป

| เครื่องมือ | ใช้เมื่อไหร่ |
|-----------|-------------|
| Class-based CM | ต้องการ state, methods เพิ่มเติม |
| `@contextmanager` | simple CM แบบ generator ง่ายๆ |
| `ExitStack` | จำนวน CM ไม่คงที่ (dynamic) |
| `suppress` | กลืน exceptions บางประเภท |
| `redirect_stdout` | capture หรือ redirect output |
| `nullcontext` | optional CM (no-op when not needed) |

**Use Cases หลัก:**
- **Resource Management**: files, connections, locks
- **Transaction**: commit/rollback atomically
- **State Management**: save/restore state
- **Timing/Profiling**: measure performance
- **Testing**: mock, patch, capture output
