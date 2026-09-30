# Part 37: Concurrency - Threading

## บทนำ

Threading เป็นเทคนิคการทำงานพร้อมกัน (concurrency) ที่ช่วยให้โปรแกรมสามารถทำหลายงานในเวลาเดียวกันได้ Python มี `threading` module ที่ทรงพลัง แต่มีข้อจำกัดจาก GIL (Global Interpreter Lock) ที่ต้องทำความเข้าใจก่อนใช้งาน

---

## 1. Concurrency vs Parallelism

### ความแตกต่างพื้นฐาน

| | Concurrency | Parallelism |
|--|-------------|-------------|
| ความหมาย | ทำหลายงานสลับกัน | ทำหลายงานพร้อมกันจริงๆ |
| CPU Cores | 1 core หรือมากกว่า | ต้องมีหลาย cores |
| เหมาะกับ | I/O-bound tasks | CPU-bound tasks |
| Python | Threading, asyncio | Multiprocessing |
| ตัวอย่าง | เปิดหลาย browser tabs | Render video หลายไฟล์พร้อมกัน |

```
Concurrency (1 CPU):
Time: ──────────────────────────────────→
Task A: ██████░░░░░░██████░░░░░░██████
Task B: ░░░░░░██████░░░░░░██████░░░░░░
        (สลับกันทำ)

Parallelism (Multi-CPU):
Time: ──────────────────────────────────→
CPU 1: ████████████████████
CPU 2: ████████████████████
       (ทำพร้อมกันจริงๆ)
```

### ตัวอย่างที่ 1: เปรียบเทียบ Sequential vs Concurrent

```python
import time
import threading

def download_file(file_id: int, duration: float) -> None:
    """จำลองการ download ไฟล์"""
    print(f"เริ่ม download file {file_id}")
    time.sleep(duration)  # I/O operation (network wait)
    print(f"Download file {file_id} เสร็จแล้ว ({duration}s)")

files = [(1, 2.0), (2, 1.5), (3, 3.0), (4, 0.5)]

# Sequential - ทำทีละงาน
print("=== Sequential ===")
start = time.time()
for file_id, duration in files:
    download_file(file_id, duration)
elapsed_sequential = time.time() - start
print(f"Sequential ใช้เวลา: {elapsed_sequential:.2f}s\n")

# Concurrent - ทำพร้อมกัน
print("=== Concurrent (Threading) ===")
start = time.time()
threads = []
for file_id, duration in files:
    t = threading.Thread(target=download_file, args=(file_id, duration))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

elapsed_concurrent = time.time() - start
print(f"Concurrent ใช้เวลา: {elapsed_concurrent:.2f}s")
print(f"เร็วขึ้น: {elapsed_sequential/elapsed_concurrent:.1f}x")
```

---

## 2. GIL (Global Interpreter Lock)

GIL คือ mutex ที่อนุญาตให้ Python bytecode รันได้ครั้งละ 1 thread เท่านั้น

### ทำไม Python มี GIL?

1. **Memory Safety** - ป้องกัน race conditions ใน CPython's memory management
2. **Reference Counting** - Python ใช้ reference counting สำหรับ garbage collection
3. **C Extensions** - ทำให้เขียน C extensions ได้ง่ายขึ้น

### ผลกระทบของ GIL

```
CPU-bound task (GIL ทำให้ thread ไม่ได้เร็วขึ้น):
Thread 1: ████████░░░░░░████████░░░░
Thread 2: ░░░░░░░░████████░░░░░░████
           ↑ GIL ทำให้สลับกันทำ ไม่ใช่พร้อมกัน

I/O-bound task (GIL ถูก release ระหว่าง I/O):
Thread 1: ████░░░░░░░░░░░░████░░░░░
Thread 2: ░░░░████░░░░░░████░░░░░░░
                ↑ ระหว่าง I/O GIL ถูก release
```

### ตัวอย่างที่ 2: GIL ผลกระทบต่อ CPU-bound tasks

```python
import threading
import time

def cpu_bound_task(n: int) -> int:
    """งาน CPU-intensive"""
    total = 0
    for i in range(n):
        total += i * i
    return total

N = 10_000_000

# Single thread
start = time.time()
cpu_bound_task(N)
single_time = time.time() - start
print(f"Single thread: {single_time:.3f}s")

# Two threads (คาดหวังว่าจะเร็วขึ้น 2x แต่จริงๆ ไม่ใช่!)
start = time.time()
t1 = threading.Thread(target=cpu_bound_task, args=(N//2,))
t2 = threading.Thread(target=cpu_bound_task, args=(N//2,))
t1.start()
t2.start()
t1.join()
t2.join()
double_time = time.time() - start
print(f"Double thread: {double_time:.3f}s")
print(f"Speedup: {single_time/double_time:.2f}x (ควรจะ 2x แต่ GIL ขัดขวาง)")

# ข้อสรุป: สำหรับ CPU-bound → ใช้ multiprocessing แทน!
```

### ตัวอย่างที่ 3: Threading เหมาะกับ I/O-bound tasks

```python
import threading
import time
import requests  # ต้องติดตั้ง: pip install requests
from urllib.request import urlopen

def fetch_url(url: str) -> int:
    """ดึงข้อมูลจาก URL"""
    try:
        with urlopen(url, timeout=5) as response:
            data = response.read()
            return len(data)
    except Exception as e:
        return 0

urls = [
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
]

# Sequential
start = time.time()
results = []
for url in urls:
    size = fetch_url(url)
    results.append(size)
sequential_time = time.time() - start
print(f"Sequential: {sequential_time:.2f}s")

# Threading
start = time.time()
results = [0] * len(urls)
def fetch_and_store(i: int, url: str) -> None:
    results[i] = fetch_url(url)

threads = [
    threading.Thread(target=fetch_and_store, args=(i, url))
    for i, url in enumerate(urls)
]
for t in threads: t.start()
for t in threads: t.join()

threaded_time = time.time() - start
print(f"Threaded: {threaded_time:.2f}s")
print(f"Speedup: {sequential_time/threaded_time:.1f}x")
```

---

## 3. Threading Module

### ตัวอย่างที่ 4: Thread Creation - วิธีพื้นฐาน

```python
import threading
import time

# วิธีที่ 1: สร้าง Thread ด้วย function
def worker(name: str, count: int) -> None:
    """Thread worker function"""
    for i in range(count):
        print(f"Thread {name}: กำลังทำงาน {i+1}/{count}")
        time.sleep(0.1)
    print(f"Thread {name}: เสร็จแล้ว!")

# สร้างและ start threads
t1 = threading.Thread(target=worker, args=("A", 3))
t2 = threading.Thread(target=worker, args=("B", 3))

t1.start()
t2.start()

# รอให้ทั้งคู่เสร็จ
t1.join()
t2.join()
print("ทุก thread เสร็จแล้ว")
```

### ตัวอย่างที่ 5: Thread Creation - subclass Thread

```python
import threading
import time
from typing import Optional

class DownloadThread(threading.Thread):
    """Custom Thread class สำหรับ download"""

    def __init__(
        self,
        file_url: str,
        file_name: str,
        speed_mbps: float = 1.0
    ):
        super().__init__(name=f"Download-{file_name}")
        self.file_url = file_url
        self.file_name = file_name
        self.speed_mbps = speed_mbps
        self.result: Optional[str] = None
        self.error: Optional[Exception] = None

    def run(self) -> None:
        """รันเมื่อ thread เริ่มทำงาน"""
        try:
            print(f"[{self.name}] เริ่ม download: {self.file_url}")
            # จำลองการ download
            file_size_mb = 5.0
            duration = file_size_mb / self.speed_mbps
            time.sleep(duration)
            self.result = f"{self.file_name} ({file_size_mb}MB)"
            print(f"[{self.name}] Download สำเร็จ: {self.result}")
        except Exception as e:
            self.error = e
            print(f"[{self.name}] Error: {e}")

# ใช้งาน
downloads = [
    DownloadThread("https://example.com/file1.zip", "file1.zip", 2.0),
    DownloadThread("https://example.com/file2.zip", "file2.zip", 1.5),
    DownloadThread("https://example.com/file3.zip", "file3.zip", 3.0),
]

for d in downloads: d.start()
for d in downloads: d.join()

print("\nDownload summary:")
for d in downloads:
    if d.error:
        print(f"  ✗ {d.file_name}: {d.error}")
    else:
        print(f"  ✓ {d.result}")
```

---

## 4. Thread Lifecycle

```
         start()
NEW ─────────────→ RUNNABLE ←─────────────┐
                      │                   │
                      ↓ (CPU available)   │ (lock acquired,
                   RUNNING               │  event set, etc.)
                      │
           ┌──────────┼──────────┐
           ↓          ↓          ↓
        BLOCKED    WAITING   TIMED_WAITING
     (I/O, lock)  (join,     (sleep, wait
                   wait)      with timeout)
                      ↓
                 TERMINATED
                 (run() returns
                  or exception)
```

### ตัวอย่างที่ 6: Thread States

```python
import threading
import time

def monitor_thread(thread: threading.Thread) -> None:
    """Monitor thread state"""
    while thread.is_alive():
        print(f"Thread alive: {thread.is_alive()}, "
              f"Name: {thread.name}, "
              f"Daemon: {thread.daemon}")
        time.sleep(0.5)

def long_task() -> None:
    print("Thread เริ่มทำงาน")
    time.sleep(2)
    print("Thread เสร็จแล้ว")

t = threading.Thread(target=long_task, name="LongTask")
t.daemon = False  # Non-daemon thread

print(f"Before start - alive: {t.is_alive()}")  # False
t.start()
print(f"After start - alive: {t.is_alive()}")   # True

t.join(timeout=3)  # รอสูงสุด 3 วินาที
print(f"After join - alive: {t.is_alive()}")    # False (ถ้าเสร็จก่อน timeout)

# ตรวจสอบ timeout
if t.is_alive():
    print("Thread ยังทำงานอยู่ (timeout)")
else:
    print("Thread เสร็จแล้ว")
```

### ตัวอย่างที่ 7: Thread Information

```python
import threading
import os

def show_thread_info() -> None:
    """แสดงข้อมูลของ thread ปัจจุบัน"""
    current = threading.current_thread()
    print(f"Thread name: {current.name}")
    print(f"Thread ID: {current.ident}")
    print(f"Native ID: {current.native_id}")
    print(f"Is daemon: {current.daemon}")
    print(f"Is alive: {current.is_alive()}")
    print(f"Process ID: {os.getpid()}")

# Main thread
print("=== Main Thread ===")
show_thread_info()

# Worker thread
print("\n=== Worker Thread ===")
t = threading.Thread(target=show_thread_info, name="MyWorker")
t.start()
t.join()

# ดูทุก thread ที่กำลังทำงาน
print(f"\n=== All Active Threads ===")
for thread in threading.enumerate():
    print(f"  {thread.name} (id={thread.ident}, daemon={thread.daemon})")
```

---

## 5. Thread Synchronization

### ตัวอย่างที่ 8: Race Condition (ปัญหา)

```python
import threading

# ตัวอย่างของ race condition
class Counter:
    def __init__(self):
        self.value = 0

    def increment(self):
        # Operation นี้ไม่ atomic ใน Python!
        # มันประกอบด้วย 3 operations:
        # 1. อ่านค่า self.value
        # 2. บวก 1
        # 3. เก็บค่ากลับ
        current = self.value    # อ่าน
        # GIL อาจสลับ thread ที่นี่!
        self.value = current + 1  # บวกและเก็บ

counter = Counter()

def increment_many(n: int) -> None:
    for _ in range(n):
        counter.increment()

# สร้าง threads หลายตัว
threads = [
    threading.Thread(target=increment_many, args=(10000,))
    for _ in range(10)
]

for t in threads: t.start()
for t in threads: t.join()

print(f"Expected: 100000")
print(f"Got: {counter.value}")
print(f"Race condition {'detected!' if counter.value != 100000 else 'not detected this time'}")
```

### ตัวอย่างที่ 9: Lock - แก้ Race Condition

```python
import threading

class ThreadSafeCounter:
    def __init__(self):
        self.value = 0
        self._lock = threading.Lock()

    def increment(self) -> None:
        with self._lock:  # context manager จะ acquire และ release lock อัตโนมัติ
            self.value += 1

    def get(self) -> int:
        with self._lock:
            return self.value

# ทดสอบ
safe_counter = ThreadSafeCounter()

def increment_safely(n: int) -> None:
    for _ in range(n):
        safe_counter.increment()

threads = [
    threading.Thread(target=increment_safely, args=(10000,))
    for _ in range(10)
]

for t in threads: t.start()
for t in threads: t.join()

print(f"Expected: 100000")
print(f"Got: {safe_counter.get()}")
print(f"Correct: {safe_counter.get() == 100000}")
```

### ตัวอย่างที่ 10: Lock ละเอียด

```python
import threading
import time

lock = threading.Lock()

def worker_with_lock(name: str) -> None:
    """Worker ที่ใช้ lock"""
    print(f"{name}: พยายามได้ lock...")
    
    # วิธีที่ 1: try-finally (รับประกัน release)
    lock.acquire()
    try:
        print(f"{name}: ได้ lock แล้ว กำลังทำงาน...")
        time.sleep(1)
        print(f"{name}: เสร็จแล้ว")
    finally:
        lock.release()
        print(f"{name}: release lock แล้ว")

def worker_with_context(name: str) -> None:
    """Worker ที่ใช้ context manager (แนะนำ)"""
    print(f"{name}: พยายามได้ lock...")
    with lock:
        print(f"{name}: ได้ lock แล้ว")
        time.sleep(1)
        print(f"{name}: เสร็จแล้ว")
    print(f"{name}: release lock แล้ว")

def try_lock(name: str) -> None:
    """ลองได้ lock แต่ไม่รอ"""
    if lock.acquire(blocking=False):
        try:
            print(f"{name}: ได้ lock!")
            time.sleep(0.5)
        finally:
            lock.release()
    else:
        print(f"{name}: ไม่ได้ lock (busy)")

# ทดสอบ
t1 = threading.Thread(target=worker_with_context, args=("Worker-1",))
t2 = threading.Thread(target=worker_with_context, args=("Worker-2",))
t1.start()
time.sleep(0.1)  # ให้ t1 ได้ lock ก่อน
t2.start()
t1.join()
t2.join()
```

### ตัวอย่างที่ 11: RLock (Reentrant Lock)

```python
import threading

# Lock ธรรมดาจะ deadlock ถ้า acquire ซ้ำใน thread เดียวกัน
# RLock อนุญาตให้ thread เดียวกัน acquire ซ้ำได้

class BankAccount:
    def __init__(self, owner: str, balance: float):
        self.owner = owner
        self.balance = balance
        self._lock = threading.RLock()  # ใช้ RLock แทน Lock

    def deposit(self, amount: float) -> None:
        with self._lock:
            self.balance += amount
            print(f"{self.owner}: deposit {amount}, balance = {self.balance}")

    def withdraw(self, amount: float) -> bool:
        with self._lock:
            if self.balance >= amount:
                self.balance -= amount
                print(f"{self.owner}: withdraw {amount}, balance = {self.balance}")
                return True
            print(f"{self.owner}: insufficient funds!")
            return False

    def transfer_to(self, other: 'BankAccount', amount: float) -> bool:
        with self._lock:          # RLock ถูก acquire ครั้งที่ 1
            if self.withdraw(amount):  # RLock ถูก acquire ครั้งที่ 2 (same thread!)
                other.deposit(amount)
                return True
        return False

# ทดสอบ
alice = BankAccount("Alice", 1000)
bob = BankAccount("Bob", 500)

alice.transfer_to(bob, 300)
print(f"Alice balance: {alice.balance}")
print(f"Bob balance: {bob.balance}")
```

---

## 6. Semaphore, Event, Condition

### ตัวอย่างที่ 12: Semaphore - จำกัดจำนวน concurrent access

```python
import threading
import time
import random

# Semaphore จำกัดให้มีได้แค่ N threads ทำงานพร้อมกัน
MAX_CONCURRENT = 3
semaphore = threading.Semaphore(MAX_CONCURRENT)

def access_resource(name: str) -> None:
    """เข้าถึง limited resource"""
    print(f"{name}: รอ semaphore...")
    with semaphore:
        print(f"{name}: กำลังใช้ resource!")
        time.sleep(random.uniform(0.5, 1.5))
        print(f"{name}: ปล่อย resource")

# 10 threads แต่ทำงานได้แค่ 3 พร้อมกัน
threads = [
    threading.Thread(target=access_resource, args=(f"Thread-{i}",))
    for i in range(10)
]

print(f"เริ่ม 10 threads แต่ semaphore จำกัดไว้ที่ {MAX_CONCURRENT}\n")
for t in threads: t.start()
for t in threads: t.join()
print("ทุก thread เสร็จแล้ว")
```

### ตัวอย่างที่ 13: Event - สัญญาณระหว่าง threads

```python
import threading
import time

# Event ใช้สำหรับส่งสัญญาณระหว่าง threads
# thread หนึ่ง "set" event ส่วนอีก thread "wait" event

start_event = threading.Event()
stop_event = threading.Event()

def producer(items: list) -> None:
    """รอสัญญาณ start แล้วผลิตข้อมูล"""
    print("Producer: รอ start signal...")
    start_event.wait()  # รอจนกว่า event จะถูก set
    
    for item in items:
        if stop_event.is_set():
            print("Producer: ได้รับ stop signal")
            break
        print(f"Producer: ผลิต {item}")
        time.sleep(0.3)
    
    print("Producer: เสร็จแล้ว")

def consumer() -> None:
    """รอสัญญาณ start แล้วกิน"""
    start_event.wait()
    
    count = 0
    while not stop_event.is_set():
        time.sleep(0.5)
        count += 1
        print(f"Consumer: กำลังทำงาน (cycle {count})")
        if count >= 5:
            print("Consumer: หยุดทำงาน")
            stop_event.set()

# เริ่ม threads
p = threading.Thread(target=producer, args=([f"item-{i}" for i in range(10)],))
c = threading.Thread(target=consumer)
p.start()
c.start()

time.sleep(1)
print("\nMain: ส่ง start signal!")
start_event.set()  # ส่งสัญญาณ

p.join()
c.join()
```

### ตัวอย่างที่ 14: Condition - Complex Synchronization

```python
import threading
import time
from collections import deque

class BoundedBuffer:
    """Buffer ที่มีขนาดจำกัด - Producer-Consumer pattern"""

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.buffer = deque()
        self.condition = threading.Condition()

    def put(self, item: any) -> None:
        """เพิ่มข้อมูล (รอถ้า buffer เต็ม)"""
        with self.condition:
            while len(self.buffer) >= self.capacity:
                print(f"Buffer เต็ม ({self.capacity}) - รอ...")
                self.condition.wait()  # รอและ release lock ชั่วคราว

            self.buffer.append(item)
            print(f"Put: {item} (buffer size: {len(self.buffer)})")
            self.condition.notify_all()  # แจ้ง threads ที่รออยู่

    def get(self) -> any:
        """ดึงข้อมูล (รอถ้า buffer ว่าง)"""
        with self.condition:
            while len(self.buffer) == 0:
                print("Buffer ว่าง - รอ...")
                self.condition.wait()

            item = self.buffer.popleft()
            print(f"Get: {item} (buffer size: {len(self.buffer)})")
            self.condition.notify_all()
            return item

# ทดสอบ
buffer = BoundedBuffer(capacity=3)

def producer(items: list) -> None:
    for item in items:
        buffer.put(item)
        time.sleep(0.2)

def consumer(count: int) -> None:
    for _ in range(count):
        item = buffer.get()
        time.sleep(0.5)  # consumer ช้ากว่า producer

p = threading.Thread(target=producer, args=([f"item-{i}" for i in range(8)],))
c = threading.Thread(target=consumer, args=(8,))
p.start()
c.start()
p.join()
c.join()
```

---

## 7. Timer Threads

### ตัวอย่างที่ 15: Timer Thread

```python
import threading
import time

def reminder(message: str) -> None:
    """ฟังก์ชันที่รันหลังจาก delay"""
    print(f"⏰ Reminder: {message}")

# สร้าง timer (รันหลังจาก 2 วินาที)
timer = threading.Timer(
    interval=2.0,
    function=reminder,
    args=("ดื่มน้ำด้วยนะ!",)
)

print("ตั้ง timer 2 วินาที...")
timer.start()

# ยังทำงานอื่นได้ระหว่างรอ
for i in range(5):
    print(f"ทำงานอื่นอยู่... {i+1}")
    time.sleep(0.5)

timer.join()
print("Timer เสร็จแล้ว")

# ยกเลิก timer
cancel_timer = threading.Timer(5.0, reminder, args=("Timer นี้จะไม่รัน",))
cancel_timer.start()
cancel_timer.cancel()
print("Timer ถูกยกเลิก")
```

### ตัวอย่างที่ 16: Repeating Timer

```python
import threading
import time

class RepeatingTimer:
    """Timer ที่รันซ้ำทุก interval"""

    def __init__(self, interval: float, function, *args, **kwargs):
        self.interval = interval
        self.function = function
        self.args = args
        self.kwargs = kwargs
        self._timer: threading.Timer | None = None
        self._stopped = threading.Event()

    def _run(self) -> None:
        if not self._stopped.is_set():
            self.function(*self.args, **self.kwargs)
            self._timer = threading.Timer(self.interval, self._run)
            self._timer.daemon = True
            self._timer.start()

    def start(self) -> None:
        self._timer = threading.Timer(self.interval, self._run)
        self._timer.daemon = True
        self._timer.start()

    def stop(self) -> None:
        self._stopped.set()
        if self._timer:
            self._timer.cancel()

# ใช้งาน
counter = 0

def tick() -> None:
    global counter
    counter += 1
    print(f"Tick #{counter} at {time.strftime('%H:%M:%S')}")

timer = RepeatingTimer(1.0, tick)
timer.start()

print("Timer เริ่มแล้ว รอ 5 วินาที...")
time.sleep(5)
timer.stop()
print(f"Timer หยุดแล้ว (รัน {counter} ครั้ง)")
```

---

## 8. Thread Pools: ThreadPoolExecutor

### ตัวอย่างที่ 17: ThreadPoolExecutor พื้นฐาน

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def process_item(item: int) -> dict:
    """ประมวลผล item หนึ่งรายการ"""
    time.sleep(0.1)  # จำลอง I/O
    return {'item': item, 'result': item ** 2, 'thread': __import__('threading').current_thread().name}

items = list(range(20))

# วิธีที่ 1: map - ง่ายที่สุด
print("=== map() ===")
with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(process_item, items))
print(f"ประมวลผล {len(results)} items")

# วิธีที่ 2: submit - ยืดหยุ่นกว่า
print("\n=== submit() ===")
with ThreadPoolExecutor(max_workers=5) as executor:
    futures = {executor.submit(process_item, item): item for item in items}
    
    for future in as_completed(futures):
        original_item = futures[future]
        try:
            result = future.result()
            print(f"Item {original_item}: {result['result']}")
        except Exception as e:
            print(f"Item {original_item} error: {e}")
```

### ตัวอย่างที่ 18: ThreadPoolExecutor กับ Timeout

```python
from concurrent.futures import ThreadPoolExecutor, TimeoutError, as_completed
import time
import random

def slow_task(task_id: int) -> str:
    """งานที่ใช้เวลานาน"""
    duration = random.uniform(0.5, 3.0)
    time.sleep(duration)
    return f"Task {task_id} complete ({duration:.1f}s)"

with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(slow_task, i) for i in range(10)]
    
    for i, future in enumerate(futures):
        try:
            result = future.result(timeout=1.5)  # timeout 1.5 วินาที
            print(f"✓ {result}")
        except TimeoutError:
            print(f"✗ Task {i}: Timeout!")
            future.cancel()  # พยายามยกเลิก (อาจไม่สำเร็จถ้าเริ่มแล้ว)
        except Exception as e:
            print(f"✗ Task {i}: Error - {e}")
```

---

## 9. concurrent.futures

### ตัวอย่างที่ 19: เปรียบเทียบ Thread vs Process Pool

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time
import math

def cpu_task(n: int) -> float:
    """CPU-intensive: คำนวณ sqrt ซ้ำๆ"""
    result = 0.0
    for i in range(n):
        result += math.sqrt(i)
    return result

def io_task(duration: float) -> str:
    """I/O-bound: รอ (จำลอง network/disk)"""
    time.sleep(duration)
    return f"Done after {duration}s"

N = 100_000
tasks = [N] * 8

# Threading กับ CPU task
start = time.time()
with ThreadPoolExecutor(max_workers=4) as ex:
    list(ex.map(cpu_task, tasks))
thread_cpu_time = time.time() - start
print(f"Thread + CPU: {thread_cpu_time:.2f}s")

# Processing กับ CPU task
start = time.time()
with ProcessPoolExecutor(max_workers=4) as ex:
    list(ex.map(cpu_task, tasks))
process_cpu_time = time.time() - start
print(f"Process + CPU: {process_cpu_time:.2f}s")
print(f"CPU speedup: {thread_cpu_time/process_cpu_time:.2f}x\n")

# I/O tasks
io_tasks = [0.5] * 8

start = time.time()
with ThreadPoolExecutor(max_workers=8) as ex:
    list(ex.map(io_task, io_tasks))
thread_io_time = time.time() - start
print(f"Thread + I/O: {thread_io_time:.2f}s (expected ~0.5s)")
```

### ตัวอย่างที่ 20: Future callbacks

```python
from concurrent.futures import ThreadPoolExecutor
import time

executor = ThreadPoolExecutor(max_workers=3)
results = []

def task(n: int) -> int:
    time.sleep(0.5)
    if n == 3:
        raise ValueError(f"Task {n} failed!")
    return n * n

def on_done(future) -> None:
    """Callback ที่รันเมื่อ future เสร็จ"""
    try:
        result = future.result()
        print(f"✓ Result: {result}")
        results.append(result)
    except Exception as e:
        print(f"✗ Error: {e}")

# ส่ง tasks และ attach callbacks
futures = []
for i in range(6):
    f = executor.submit(task, i)
    f.add_done_callback(on_done)  # callback เรียกอัตโนมัติเมื่อเสร็จ
    futures.append(f)

executor.shutdown(wait=True)
print(f"\nSuccessful results: {sorted(results)}")
```

---

## 10. Thread-safe Data Structures

### ตัวอย่างที่ 21: queue.Queue - Thread-safe Queue

```python
import threading
import queue
import time
import random

def producer(q: queue.Queue, items: list) -> None:
    """ผลิตข้อมูลและใส่ใน queue"""
    for item in items:
        time.sleep(random.uniform(0.1, 0.3))
        q.put(item)
        print(f"Producer: put {item} (queue size: {q.qsize()})")
    
    # ส่งสัญญาณว่าหมดแล้ว
    q.put(None)
    print("Producer: เสร็จแล้ว")

def consumer(q: queue.Queue, name: str) -> None:
    """ดึงข้อมูลจาก queue และประมวลผล"""
    while True:
        try:
            item = q.get(timeout=2.0)  # รอสูงสุด 2 วินาที
            if item is None:
                q.put(None)  # ส่งต่อ poison pill ให้ consumer ตัวอื่น
                break
            
            time.sleep(random.uniform(0.2, 0.5))  # จำลองการประมวลผล
            print(f"{name}: processed {item}")
            q.task_done()  # บอกว่าประมวลผล item นี้เสร็จแล้ว
        except queue.Empty:
            print(f"{name}: timeout, ออกจากการทำงาน")
            break

# ทดสอบ
q = queue.Queue(maxsize=5)  # buffer ขนาด 5

items = [f"task-{i}" for i in range(10)]

p = threading.Thread(target=producer, args=(q, items))
c1 = threading.Thread(target=consumer, args=(q, "Consumer-1"))
c2 = threading.Thread(target=consumer, args=(q, "Consumer-2"))

p.start()
c1.start()
c2.start()

p.join()
c1.join()
c2.join()

print("ระบบ Producer-Consumer เสร็จแล้ว")
```

### ตัวอย่างที่ 22: Queue Types

```python
import queue

# Queue ประเภทต่างๆ
fifo_q = queue.Queue()         # FIFO - First In First Out
lifo_q = queue.LifoQueue()     # LIFO - Last In First Out (Stack)
prio_q = queue.PriorityQueue() # Priority Queue

# FIFO Queue
for i in [3, 1, 4, 1, 5]:
    fifo_q.put(i)
print("FIFO:", [fifo_q.get() for _ in range(5)])  # [3, 1, 4, 1, 5]

# LIFO Queue
for i in [3, 1, 4, 1, 5]:
    lifo_q.put(i)
print("LIFO:", [lifo_q.get() for _ in range(5)])  # [5, 1, 4, 1, 3]

# Priority Queue (เรียงจากน้อยไปมาก)
for item in [(3, "low"), (1, "high"), (2, "medium")]:
    prio_q.put(item)
print("Priority:", [prio_q.get() for _ in range(3)])
# [(1, 'high'), (2, 'medium'), (3, 'low')]
```

---

## 11. Daemon Threads

### ตัวอย่างที่ 23: Daemon vs Non-daemon Threads

```python
import threading
import time

def background_monitor() -> None:
    """Background monitoring - daemon thread"""
    while True:
        print(f"Monitor: ระบบทำงานปกติ (เวลา: {time.strftime('%H:%M:%S')})")
        time.sleep(1)

def foreground_task() -> None:
    """Foreground task - non-daemon"""
    for i in range(3):
        print(f"Task: กำลังทำงาน step {i+1}")
        time.sleep(1.5)
    print("Task: เสร็จแล้ว")

# Daemon thread - จะถูกหยุดเมื่อ main thread เสร็จ
monitor = threading.Thread(target=background_monitor, name="Monitor")
monitor.daemon = True  # ตั้งก่อน start()!

# Non-daemon thread - โปรแกรมรอจนกว่าจะเสร็จ
task = threading.Thread(target=foreground_task, name="Task")
task.daemon = False

monitor.start()
task.start()

task.join()  # รอ task เสร็จ
print("Main: task เสร็จแล้ว โปรแกรมจะจบ")
print("Main: monitor daemon จะถูกหยุดโดยอัตโนมัติ")
# monitor จะถูกหยุดเมื่อ main thread จบ
```

---

## 12. Thread Communication

### ตัวอย่างที่ 24: Thread-local Storage

```python
import threading
import time

# thread_local เก็บข้อมูลแยกสำหรับแต่ละ thread
thread_local = threading.local()

def set_user_context(user_id: int, user_name: str) -> None:
    """ตั้ง user context สำหรับ thread นี้"""
    thread_local.user_id = user_id
    thread_local.user_name = user_name

def process_request(request_data: str) -> str:
    """ประมวลผล request โดยใช้ user context ของ thread นี้"""
    user_id = getattr(thread_local, 'user_id', 'unknown')
    user_name = getattr(thread_local, 'user_name', 'anonymous')
    return f"User {user_name} ({user_id}) processed: {request_data}"

def handle_user(user_id: int, user_name: str, requests: list) -> None:
    """จำลอง user session"""
    set_user_context(user_id, user_name)
    
    for req in requests:
        result = process_request(req)
        print(result)
        time.sleep(0.1)

# Thread แต่ละตัวมี context ของตัวเอง
threads = [
    threading.Thread(
        target=handle_user,
        args=(1, "Alice", ["GET /profile", "PUT /settings"])
    ),
    threading.Thread(
        target=handle_user,
        args=(2, "Bob", ["GET /orders", "POST /checkout"])
    ),
]

for t in threads: t.start()
for t in threads: t.join()
```

---

## 13. Race Conditions และวิธีป้องกัน

### ตัวอย่างที่ 25: Double-checked Locking Pattern

```python
import threading
import time

class Singleton:
    """Singleton ที่ thread-safe"""
    _instance = None
    _lock = threading.Lock()

    @classmethod
    def get_instance(cls) -> 'Singleton':
        # Double-checked locking - เพิ่ม performance
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:  # ตรวจสอบอีกครั้งหลัง acquire lock
                    print("Creating singleton instance")
                    time.sleep(0.1)  # จำลองการ initialize
                    cls._instance = cls()
        return cls._instance

def get_singleton() -> None:
    instance = Singleton.get_instance()
    print(f"Got instance: {id(instance)}")

threads = [threading.Thread(target=get_singleton) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```

### ตัวอย่างที่ 26: Atomic Operations

```python
import threading
from queue import Queue

class AtomicCounter:
    """Counter ที่ thread-safe ด้วย Lock"""

    def __init__(self, initial: int = 0):
        self._value = initial
        self._lock = threading.Lock()

    def increment(self, amount: int = 1) -> int:
        with self._lock:
            self._value += amount
            return self._value

    def decrement(self, amount: int = 1) -> int:
        with self._lock:
            self._value -= amount
            return self._value

    def compare_and_swap(self, expected: int, new_value: int) -> bool:
        """Atomic compare-and-swap operation"""
        with self._lock:
            if self._value == expected:
                self._value = new_value
                return True
            return False

    @property
    def value(self) -> int:
        with self._lock:
            return self._value

# ทดสอบ
counter = AtomicCounter(0)
results = []

def worker(n: int) -> None:
    for _ in range(n):
        val = counter.increment()
    results.append(counter.value)

threads = [threading.Thread(target=worker, args=(1000,)) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Expected: 10000")
print(f"Got: {counter.value}")
print(f"Correct: {counter.value == 10000}")
```

---

## 14. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 27: Web Scraper

```python
import threading
import time
import queue
import urllib.request
import urllib.error
from html.parser import HTMLParser
from typing import Optional

class TitleParser(HTMLParser):
    """HTML Parser สำหรับดึง title"""
    def __init__(self):
        super().__init__()
        self.title = ""
        self._in_title = False

    def handle_starttag(self, tag, attrs):
        if tag == "title":
            self._in_title = True

    def handle_endtag(self, tag):
        if tag == "title":
            self._in_title = False

    def handle_data(self, data):
        if self._in_title:
            self.title += data

class WebScraper:
    """Thread-based web scraper"""

    def __init__(self, num_workers: int = 5):
        self.num_workers = num_workers
        self.url_queue = queue.Queue()
        self.results = {}
        self.results_lock = threading.Lock()
        self.errors = []
        self.errors_lock = threading.Lock()

    def _fetch_url(self, url: str) -> Optional[str]:
        """ดึงเนื้อหาจาก URL"""
        try:
            req = urllib.request.Request(
                url,
                headers={'User-Agent': 'Mozilla/5.0 Python Web Scraper'}
            )
            with urllib.request.urlopen(req, timeout=10) as response:
                html = response.read().decode('utf-8', errors='ignore')
                parser = TitleParser()
                parser.feed(html)
                return parser.title.strip() or "No title found"
        except urllib.error.URLError as e:
            return None
        except Exception as e:
            return None

    def _worker(self) -> None:
        """Worker thread"""
        while True:
            try:
                url = self.url_queue.get(timeout=1)
                if url is None:  # Poison pill
                    self.url_queue.task_done()
                    break

                print(f"Fetching: {url}")
                title = self._fetch_url(url)

                if title:
                    with self.results_lock:
                        self.results[url] = title
                else:
                    with self.errors_lock:
                        self.errors.append(url)

                self.url_queue.task_done()
            except queue.Empty:
                break

    def scrape(self, urls: list[str]) -> dict:
        """Scrape หลาย URLs พร้อมกัน"""
        # ใส่ URLs ใน queue
        for url in urls:
            self.url_queue.put(url)

        # เพิ่ม poison pills สำหรับ workers
        for _ in range(self.num_workers):
            self.url_queue.put(None)

        # สร้างและ start workers
        workers = [
            threading.Thread(target=self._worker, name=f"Scraper-{i}")
            for i in range(self.num_workers)
        ]
        
        start = time.time()
        for w in workers: w.start()
        for w in workers: w.join()
        elapsed = time.time() - start

        print(f"\nScrape เสร็จใน {elapsed:.2f}s")
        print(f"สำเร็จ: {len(self.results)}, ล้มเหลว: {len(self.errors)}")

        return self.results

# ทดสอบ (จะใช้เวลาจริง)
urls = [
    "https://httpbin.org/html",
    "https://example.com",
    "https://httpbin.org/html",
]

scraper = WebScraper(num_workers=3)
results = scraper.scrape(urls)

for url, title in results.items():
    print(f"  {url}: {title[:50]}")
```

### ตัวอย่างที่ 28: File Downloader

```python
import threading
import os
import time
from dataclasses import dataclass
from enum import Enum
from typing import Callable, Optional

class DownloadStatus(Enum):
    PENDING = "pending"
    DOWNLOADING = "downloading"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"

@dataclass
class DownloadTask:
    url: str
    filename: str
    status: DownloadStatus = DownloadStatus.PENDING
    progress: float = 0.0
    error: Optional[str] = None
    file_size: int = 0

class FileDownloader:
    """Multi-threaded file downloader"""

    def __init__(
        self,
        max_concurrent: int = 3,
        on_progress: Optional[Callable] = None
    ):
        self.max_concurrent = max_concurrent
        self.on_progress = on_progress
        self._semaphore = threading.Semaphore(max_concurrent)
        self._lock = threading.Lock()
        self.tasks: list[DownloadTask] = []

    def _download(self, task: DownloadTask) -> None:
        """ดาวน์โหลดไฟล์"""
        with self._semaphore:
            task.status = DownloadStatus.DOWNLOADING
            
            try:
                # จำลองการ download
                total_size = 1000  # KB
                task.file_size = total_size

                for chunk in range(0, total_size, 100):
                    time.sleep(0.1)  # จำลอง network latency
                    task.progress = min((chunk + 100) / total_size * 100, 100)
                    
                    if self.on_progress:
                        self.on_progress(task)

                task.status = DownloadStatus.COMPLETED
                task.progress = 100.0
                
            except Exception as e:
                task.status = DownloadStatus.FAILED
                task.error = str(e)

    def download_all(self, urls_and_filenames: list[tuple]) -> list[DownloadTask]:
        """ดาวน์โหลดหลายไฟล์พร้อมกัน"""
        self.tasks = [
            DownloadTask(url=url, filename=filename)
            for url, filename in urls_and_filenames
        ]

        threads = [
            threading.Thread(
                target=self._download,
                args=(task,),
                name=f"Download-{task.filename}"
            )
            for task in self.tasks
        ]

        print(f"เริ่มดาวน์โหลด {len(self.tasks)} ไฟล์ (max {self.max_concurrent} พร้อมกัน)")
        for t in threads: t.start()
        for t in threads: t.join()

        return self.tasks

def progress_callback(task: DownloadTask) -> None:
    bar_length = 20
    filled = int(bar_length * task.progress / 100)
    bar = "█" * filled + "░" * (bar_length - filled)
    print(f"\r{task.filename}: [{bar}] {task.progress:.0f}%", end="", flush=True)

# ทดสอบ
downloader = FileDownloader(max_concurrent=2)
files = [
    ("https://example.com/file1.zip", "file1.zip"),
    ("https://example.com/file2.zip", "file2.zip"),
    ("https://example.com/file3.zip", "file3.zip"),
    ("https://example.com/file4.zip", "file4.zip"),
]

results = downloader.download_all(files)
print("\n")

for task in results:
    status_emoji = "✓" if task.status == DownloadStatus.COMPLETED else "✗"
    print(f"{status_emoji} {task.filename}: {task.status.value}")
```

### ตัวอย่างที่ 29: Producer-Consumer Pattern

```python
import threading
import queue
import time
import random
from dataclasses import dataclass
from typing import Optional

@dataclass
class WorkItem:
    id: int
    data: str
    priority: int = 1

class WorkerPool:
    """Thread pool สำหรับ producer-consumer"""

    def __init__(self, num_workers: int, queue_size: int = 100):
        self.num_workers = num_workers
        self._queue: queue.PriorityQueue = queue.PriorityQueue(maxsize=queue_size)
        self._workers: list[threading.Thread] = []
        self._results: list = []
        self._results_lock = threading.Lock()
        self._running = threading.Event()
        self._running.set()
        self._stats = {'processed': 0, 'errors': 0}
        self._stats_lock = threading.Lock()

    def _worker(self, worker_id: int) -> None:
        """Worker ที่รับงานจาก queue"""
        print(f"Worker {worker_id}: พร้อมทำงาน")
        
        while self._running.is_set():
            try:
                priority, item = self._queue.get(timeout=1.0)
                
                try:
                    # ประมวลผล
                    result = self._process(item)
                    
                    with self._results_lock:
                        self._results.append(result)
                    
                    with self._stats_lock:
                        self._stats['processed'] += 1
                        
                except Exception as e:
                    print(f"Worker {worker_id}: Error processing item {item.id}: {e}")
                    with self._stats_lock:
                        self._stats['errors'] += 1
                finally:
                    self._queue.task_done()
                    
            except queue.Empty:
                continue

        print(f"Worker {worker_id}: หยุดทำงาน")

    def _process(self, item: WorkItem) -> dict:
        """ประมวลผล work item"""
        time.sleep(random.uniform(0.1, 0.5))
        return {
            'id': item.id,
            'result': item.data.upper(),
            'priority': item.priority
        }

    def start(self) -> None:
        """เริ่ม workers"""
        for i in range(self.num_workers):
            w = threading.Thread(
                target=self._worker,
                args=(i,),
                name=f"Worker-{i}",
                daemon=True
            )
            self._workers.append(w)
            w.start()

    def submit(self, item: WorkItem) -> None:
        """ส่งงานเข้า queue"""
        # ใส่ priority เพื่อให้ PriorityQueue เรียงลำดับ
        self._queue.put((item.priority, item))

    def wait_and_stop(self) -> list:
        """รอให้งานเสร็จทั้งหมด แล้วหยุด"""
        self._queue.join()  # รอจนกว่า queue จะว่าง
        self._running.clear()  # บอก workers ให้หยุด
        return self._results

    def get_stats(self) -> dict:
        with self._stats_lock:
            return self._stats.copy()

# ทดสอบ
pool = WorkerPool(num_workers=4, queue_size=50)
pool.start()

# ส่งงาน
print("ส่งงาน 20 รายการ...")
for i in range(20):
    priority = random.choice([1, 2, 3])  # 1 = สูงสุด, 3 = ต่ำสุด
    item = WorkItem(id=i, data=f"task_{i}", priority=priority)
    pool.submit(item)
    time.sleep(0.05)  # จำลอง producer ที่ไม่เร็วเกินไป

# รอและดูผลลัพธ์
results = pool.wait_and_stop()
stats = pool.get_stats()

print(f"\nผลลัพธ์: {len(results)} items")
print(f"Stats: {stats}")
```

### ตัวอย่างที่ 30: Thread-safe Cache

```python
import threading
import time
import hashlib
from typing import Any, Optional
from dataclasses import dataclass, field
from datetime import datetime, timedelta

@dataclass
class CacheEntry:
    value: Any
    expires_at: datetime
    hits: int = 0

class ThreadSafeCache:
    """Cache ที่ thread-safe มี TTL และ LRU eviction"""

    def __init__(self, max_size: int = 100, ttl_seconds: int = 300):
        self.max_size = max_size
        self.ttl = timedelta(seconds=ttl_seconds)
        self._cache: dict[str, CacheEntry] = {}
        self._lock = threading.RLock()
        self._stats = {'hits': 0, 'misses': 0, 'evictions': 0}
        
        # Background cleanup thread
        self._cleanup_thread = threading.Thread(
            target=self._cleanup_expired,
            daemon=True
        )
        self._cleanup_thread.start()

    def get(self, key: str) -> Optional[Any]:
        with self._lock:
            if key in self._cache:
                entry = self._cache[key]
                if datetime.now() < entry.expires_at:
                    entry.hits += 1
                    self._stats['hits'] += 1
                    return entry.value
                else:
                    del self._cache[key]
            
            self._stats['misses'] += 1
            return None

    def set(self, key: str, value: Any) -> None:
        with self._lock:
            # Evict ถ้า cache เต็ม
            if len(self._cache) >= self.max_size and key not in self._cache:
                self._evict_lru()
            
            self._cache[key] = CacheEntry(
                value=value,
                expires_at=datetime.now() + self.ttl
            )

    def _evict_lru(self) -> None:
        """ลบ entry ที่ถูกใช้น้อยที่สุด"""
        if not self._cache:
            return
        
        lru_key = min(self._cache.keys(), key=lambda k: self._cache[k].hits)
        del self._cache[lru_key]
        self._stats['evictions'] += 1

    def _cleanup_expired(self) -> None:
        """Background thread ลบ expired entries"""
        while True:
            time.sleep(60)
            with self._lock:
                now = datetime.now()
                expired = [k for k, v in self._cache.items() if v.expires_at < now]
                for key in expired:
                    del self._cache[key]

    def get_stats(self) -> dict:
        with self._lock:
            return {
                **self._stats,
                'size': len(self._cache),
                'hit_rate': (
                    self._stats['hits'] / 
                    (self._stats['hits'] + self._stats['misses']) * 100
                    if (self._stats['hits'] + self._stats['misses']) > 0
                    else 0
                )
            }

# ทดสอบ
cache = ThreadSafeCache(max_size=10, ttl_seconds=5)

def worker_cache(worker_id: int, cache: ThreadSafeCache) -> None:
    for i in range(20):
        key = f"key_{i % 5}"  # ใช้ 5 keys หมุนเวียน
        
        value = cache.get(key)
        if value is None:
            value = f"computed_{key}_{worker_id}"
            time.sleep(0.01)  # จำลอง expensive computation
            cache.set(key, value)
        
        time.sleep(0.05)

threads = [
    threading.Thread(target=worker_cache, args=(i, cache))
    for i in range(5)
]
for t in threads: t.start()
for t in threads: t.join()

stats = cache.get_stats()
print(f"Cache stats: {stats}")
print(f"Hit rate: {stats['hit_rate']:.1f}%")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Thread-safe Stack

**โจทย์:** สร้าง thread-safe Stack ที่รองรับ push, pop, peek และ size

**เฉลย:**

```python
import threading
from typing import TypeVar, Generic, Optional

T = TypeVar('T')

class ThreadSafeStack(Generic[T]):
    """Thread-safe Stack implementation"""

    def __init__(self, max_size: int = None):
        self._stack: list[T] = []
        self._lock = threading.Lock()
        self._not_empty = threading.Condition(self._lock)
        self._not_full = threading.Condition(self._lock)
        self.max_size = max_size

    def push(self, item: T, timeout: float = None) -> bool:
        with self._not_full:
            if self.max_size is not None:
                # รอถ้า stack เต็ม
                deadline = time.time() + (timeout or float('inf'))
                while len(self._stack) >= self.max_size:
                    remaining = deadline - time.time()
                    if remaining <= 0:
                        return False
                    self._not_full.wait(remaining)
            
            self._stack.append(item)
            self._not_empty.notify()
            return True

    def pop(self, timeout: float = None) -> Optional[T]:
        import time
        with self._not_empty:
            if timeout is not None:
                deadline = time.time() + timeout
                while not self._stack:
                    remaining = deadline - time.time()
                    if remaining <= 0:
                        return None
                    self._not_empty.wait(remaining)
            else:
                while not self._stack:
                    self._not_empty.wait()
            
            item = self._stack.pop()
            if self.max_size is not None:
                self._not_full.notify()
            return item

    def peek(self) -> Optional[T]:
        with self._lock:
            return self._stack[-1] if self._stack else None

    def size(self) -> int:
        with self._lock:
            return len(self._stack)

    def is_empty(self) -> bool:
        with self._lock:
            return len(self._stack) == 0

import threading
import time

stack = ThreadSafeStack[int](max_size=10)

def pusher(n: int) -> None:
    for i in range(n):
        stack.push(i)
        time.sleep(0.01)

def popper(n: int) -> None:
    results = []
    for _ in range(n):
        val = stack.pop(timeout=1.0)
        if val is not None:
            results.append(val)
        time.sleep(0.02)
    print(f"Popped {len(results)} items")

p = threading.Thread(target=pusher, args=(20,))
c = threading.Thread(target=popper, args=(20,))
p.start()
c.start()
p.join()
c.join()
print(f"Stack size after: {stack.size()}")
```

---

### แบบฝึกหัดที่ 2: Parallel Image Processor

**โจทย์:** สร้าง image processor ที่ใช้ threads ประมวลผลภาพหลายรูปพร้อมกัน (จำลอง)

**เฉลย:**

```python
import threading
import time
import random
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass
from typing import Optional

@dataclass
class ImageTask:
    image_id: str
    width: int
    height: int
    operation: str

@dataclass
class ProcessedImage:
    image_id: str
    original_size: tuple
    final_size: tuple
    duration: float
    operations_applied: list[str]

class ImageProcessor:
    """จำลอง parallel image processor"""

    OPERATIONS = {
        'resize': lambda img: (img[0] // 2, img[1] // 2),
        'blur': lambda img: img,
        'sharpen': lambda img: img,
        'compress': lambda img: (int(img[0] * 0.9), int(img[1] * 0.9)),
        'watermark': lambda img: img,
    }

    def __init__(self, workers: int = 4):
        self.workers = workers
        self._processed = 0
        self._lock = threading.Lock()

    def _apply_operation(self, op: str, size: tuple) -> tuple:
        """ใช้ operation กับรูปภาพ"""
        time.sleep(random.uniform(0.1, 0.5))  # จำลองการประมวลผล
        
        if op in self.OPERATIONS:
            return self.OPERATIONS[op](size)
        return size

    def process_image(self, task: ImageTask) -> ProcessedImage:
        """ประมวลผลภาพหนึ่งรูป"""
        start = time.time()
        current_size = (task.width, task.height)
        original_size = current_size
        operations_applied = []

        # ใช้ operations ทีละขั้น
        for op in task.operation.split(','):
            op = op.strip()
            current_size = self._apply_operation(op, current_size)
            operations_applied.append(op)

        duration = time.time() - start

        with self._lock:
            self._processed += 1
            print(f"ประมวลผล {task.image_id}: "
                  f"{original_size} → {current_size} "
                  f"({duration:.2f}s)")

        return ProcessedImage(
            image_id=task.image_id,
            original_size=original_size,
            final_size=current_size,
            duration=duration,
            operations_applied=operations_applied
        )

    def process_batch(self, tasks: list[ImageTask]) -> list[ProcessedImage]:
        """ประมวลผลหลายรูปพร้อมกัน"""
        print(f"เริ่มประมวลผล {len(tasks)} รูป ด้วย {self.workers} workers")
        start = time.time()

        with ThreadPoolExecutor(max_workers=self.workers) as executor:
            futures = {
                executor.submit(self.process_image, task): task
                for task in tasks
            }
            
            results = []
            for future in as_completed(futures):
                try:
                    result = future.result()
                    results.append(result)
                except Exception as e:
                    task = futures[future]
                    print(f"Error processing {task.image_id}: {e}")

        elapsed = time.time() - start
        print(f"\nประมวลผลเสร็จใน {elapsed:.2f}s "
              f"({len(results)}/{len(tasks)} สำเร็จ)")
        return results

# ทดสอบ
processor = ImageProcessor(workers=4)

tasks = [
    ImageTask(f"img_{i:03d}", 1920, 1080, "resize,blur,compress")
    for i in range(10)
]

results = processor.process_batch(tasks)

# สรุปผล
total_time = sum(r.duration for r in results)
avg_time = total_time / len(results)
print(f"\nสรุป:")
print(f"  ประมวลผลทั้งหมด: {len(results)} รูป")
print(f"  เวลาเฉลี่ยต่อรูป: {avg_time:.2f}s")
print(f"  เวลารวม (sequential): {total_time:.2f}s")
```

---

### แบบฝึกหัดที่ 3-8 (สรุปย่อ)

แบบฝึกหัดที่ 3: สร้าง `ReadWriteLock` ที่อนุญาตให้ readers หลายคนอ่านพร้อมกัน แต่ writers ต้อง exclusive access

แบบฝึกหัดที่ 4: สร้าง `ThreadPoolExecutor` wrapper ที่มี rate limiting (จำกัด tasks per second)

แบบฝึกหัดที่ 5: สร้าง multi-threaded log aggregator ที่รวม logs จาก threads ต่างๆ

แบบฝึกหัดที่ 6: สร้าง parallel hash calculator สำหรับไฟล์หลายไฟล์

แบบฝึกหัดที่ 7: สร้าง thread-safe event bus สำหรับ publish-subscribe pattern

แบบฝึกหัดที่ 8: สร้าง crawler ที่รวม threading กับ rate limiting และ retry logic

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Concurrency vs Parallelism** - ความแตกต่างและเมื่อไรควรใช้อะไร
2. **GIL** - ข้อจำกัดของ Python threading และผลต่อ CPU-bound tasks
3. **Threading Module** - การสร้างและจัดการ threads
4. **Thread Lifecycle** - สถานะต่างๆ ของ thread
5. **Synchronization** - Lock, RLock, Semaphore, Event, Condition
6. **Thread Safety** - การป้องกัน race conditions
7. **ThreadPoolExecutor** - การจัดการ pool ของ threads
8. **Queue** - thread-safe data structures
9. **Daemon Threads** - background threads

### เมื่อไรควรใช้ Threading?

- **I/O-bound tasks**: network requests, file I/O, database queries
- **เมื่อต้องการ responsiveness**: UI, server handlers
- **เมื่อ tasks รอกัน**: producer-consumer patterns

### เมื่อไรไม่ควรใช้ Threading?

- **CPU-bound tasks**: ใช้ multiprocessing แทน
- **เมื่อ code ซับซ้อนมาก**: พิจารณา asyncio
- **เมื่อ memory สำคัญ**: threads ใช้ memory มากกว่า coroutines

---

*ถัดไป: Part 38 - Concurrency: Multiprocessing*
