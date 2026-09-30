# Part 38: Concurrency - Multiprocessing

## บทนำ

Multiprocessing ใช้หลาย processes แทนที่จะเป็น threads ทำให้หลีกเลี่ยง GIL ได้และเหมาะกับงาน CPU-intensive อย่างแท้จริง แต่ละ process มี memory space ของตัวเอง ทำให้การแชร์ข้อมูลต้องใช้กลไกพิเศษ

---

## 1. Process vs Thread

| | Process | Thread |
|--|---------|--------|
| Memory | แยกกัน (isolated) | ใช้ร่วมกัน |
| GIL | ไม่มี (ต่าง process) | มี (ใน CPython) |
| Communication | IPC (Pipe, Queue, Shared Memory) | ตรงๆ ผ่าน shared memory |
| Overhead | สูง (startup, memory) | ต่ำ |
| Crash isolation | แยกกัน | ถ้าหนึ่ง crash ทั้งหมดพัง |
| เหมาะกับ | CPU-bound | I/O-bound |
| การสร้าง | ช้ากว่า | เร็วกว่า |

```
Threading (shared memory):
┌─────────────────────────┐
│         Process         │
│  ┌─────┐  ┌─────┐      │
│  │ T1  │  │ T2  │      │
│  └──┬──┘  └──┬──┘      │
│     └────┬───┘          │
│      Shared             │
│      Memory             │
└─────────────────────────┘

Multiprocessing (separate memory):
┌──────────┐   ┌──────────┐
│ Process 1 │   │ Process 2 │
│  Memory  │   │  Memory  │
│  ┌─────┐ │   │ ┌─────┐  │
│  │ GIL │ │   │ │ GIL │  │
│  └─────┘ │   │ └─────┘  │
└──────────┘   └──────────┘
    │ IPC (Queue/Pipe) ↑
    └──────────────────┘
```

### ตัวอย่างที่ 1: เปรียบเทียบ Thread vs Process สำหรับ CPU-bound

```python
import threading
import multiprocessing
import time
import math

def cpu_intensive(n: int) -> float:
    """งาน CPU-intensive: คำนวณ prime numbers"""
    count = 0
    for i in range(2, n):
        is_prime = all(i % j != 0 for j in range(2, int(math.sqrt(i)) + 1))
        if is_prime:
            count += 1
    return count

N = 100_000
NUM_WORKERS = 4
CHUNK_SIZE = N // NUM_WORKERS
chunks = [CHUNK_SIZE] * NUM_WORKERS

# Sequential
start = time.time()
results = [cpu_intensive(chunk) for chunk in chunks]
sequential_time = time.time() - start
print(f"Sequential: {sequential_time:.2f}s")

# Threading (ถูก GIL ขัดขวาง)
start = time.time()
threads = [threading.Thread(target=cpu_intensive, args=(chunk,)) for chunk in chunks]
for t in threads: t.start()
for t in threads: t.join()
thread_time = time.time() - start
print(f"Threading: {thread_time:.2f}s (GIL ขัดขวาง ไม่เร็วขึ้น)")

# Multiprocessing (แต่ละ process มี GIL ของตัวเอง)
start = time.time()
with multiprocessing.Pool(NUM_WORKERS) as pool:
    results = pool.map(cpu_intensive, chunks)
process_time = time.time() - start
print(f"Multiprocessing: {process_time:.2f}s")
print(f"Speedup vs Sequential: {sequential_time/process_time:.2f}x")
```

---

## 2. Multiprocessing Module

### ตัวอย่างที่ 2: สร้าง Process พื้นฐาน

```python
import multiprocessing
import os
import time

def worker_function(name: str, count: int) -> None:
    """Worker ที่รันใน process แยก"""
    pid = os.getpid()
    ppid = os.getppid()
    print(f"Process {name} (PID={pid}, Parent={ppid}): เริ่มทำงาน")
    
    for i in range(count):
        time.sleep(0.2)
        print(f"Process {name}: step {i+1}/{count}")
    
    print(f"Process {name}: เสร็จแล้ว")

if __name__ == '__main__':
    print(f"Main process PID: {os.getpid()}")
    
    # สร้าง processes
    p1 = multiprocessing.Process(
        target=worker_function,
        args=("Alpha", 3),
        name="Process-Alpha"
    )
    p2 = multiprocessing.Process(
        target=worker_function,
        args=("Beta", 3),
        name="Process-Beta"
    )
    
    p1.start()
    p2.start()
    
    print(f"P1 PID: {p1.pid}, alive: {p1.is_alive()}")
    print(f"P2 PID: {p2.pid}, alive: {p2.is_alive()}")
    
    p1.join()
    p2.join()
    
    print(f"P1 exitcode: {p1.exitcode}")
    print(f"P2 exitcode: {p2.exitcode}")
```

### ตัวอย่างที่ 3: Process Class แบบ Subclass

```python
import multiprocessing
import os
import time
from typing import Optional

class DataProcessor(multiprocessing.Process):
    """Custom Process สำหรับประมวลผลข้อมูล"""

    def __init__(
        self,
        data: list,
        result_queue: multiprocessing.Queue,
        processor_id: int
    ):
        super().__init__(name=f"DataProcessor-{processor_id}")
        self.data = data
        self.result_queue = result_queue
        self.processor_id = processor_id

    def run(self) -> None:
        """รันเมื่อ process เริ่มทำงาน"""
        print(f"[{self.name}] PID={os.getpid()} เริ่มทำงาน")
        
        results = []
        for item in self.data:
            # จำลองการประมวลผล
            time.sleep(0.01)
            result = {
                'original': item,
                'processed': item ** 2,
                'processor': self.processor_id
            }
            results.append(result)
        
        # ส่งผลลัพธ์กลับผ่าน queue
        self.result_queue.put({
            'processor_id': self.processor_id,
            'results': results,
            'count': len(results)
        })
        print(f"[{self.name}] เสร็จแล้ว: {len(results)} items")

if __name__ == '__main__':
    # แบ่งข้อมูลให้แต่ละ processor
    data = list(range(100))
    num_processors = 4
    chunk_size = len(data) // num_processors
    
    result_queue = multiprocessing.Queue()
    
    processors = [
        DataProcessor(
            data[i*chunk_size:(i+1)*chunk_size],
            result_queue,
            i
        )
        for i in range(num_processors)
    ]
    
    # เริ่ม processes
    for p in processors: p.start()
    
    # รวบรวมผลลัพธ์
    all_results = []
    for _ in range(num_processors):
        result = result_queue.get()
        all_results.extend(result['results'])
        print(f"Processor {result['processor_id']}: {result['count']} items")
    
    # รอให้เสร็จ
    for p in processors: p.join()
    
    print(f"\nรวม: {len(all_results)} items processed")
    print(f"ตัวอย่าง: {all_results[:3]}")
```

---

## 3. ProcessPoolExecutor

### ตัวอย่างที่ 4: ProcessPoolExecutor พื้นฐาน

```python
from concurrent.futures import ProcessPoolExecutor, as_completed
import multiprocessing
import time
import math

def prime_check(n: int) -> tuple[int, bool]:
    """ตรวจสอบว่า n เป็นจำนวนเฉพาะหรือไม่"""
    if n < 2:
        return n, False
    if n == 2:
        return n, True
    if n % 2 == 0:
        return n, False
    
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        if n % i == 0:
            return n, False
    return n, True

def find_primes_range(start: int, end: int) -> list[int]:
    """หาจำนวนเฉพาะในช่วง [start, end]"""
    return [n for n in range(start, end + 1) if prime_check(n)[1]]

if __name__ == '__main__':
    # แบ่งงานเป็น chunks
    ranges = [(i*10000, (i+1)*10000-1) for i in range(8)]
    
    print(f"ใช้ {multiprocessing.cpu_count()} CPU cores")
    
    # Sequential
    start = time.time()
    all_primes = []
    for r in ranges:
        all_primes.extend(find_primes_range(*r))
    seq_time = time.time() - start
    print(f"Sequential: {seq_time:.2f}s, found {len(all_primes)} primes")
    
    # ProcessPoolExecutor
    start = time.time()
    all_primes_parallel = []
    
    with ProcessPoolExecutor(max_workers=4) as executor:
        futures = {
            executor.submit(find_primes_range, start_n, end_n): (start_n, end_n)
            for start_n, end_n in ranges
        }
        
        for future in as_completed(futures):
            range_info = futures[future]
            try:
                primes = future.result()
                all_primes_parallel.extend(primes)
                print(f"Range {range_info}: {len(primes)} primes")
            except Exception as e:
                print(f"Error: {e}")
    
    parallel_time = time.time() - start
    print(f"\nParallel: {parallel_time:.2f}s, found {len(all_primes_parallel)} primes")
    print(f"Speedup: {seq_time/parallel_time:.2f}x")
    print(f"Results match: {sorted(all_primes) == sorted(all_primes_parallel)}")
```

---

## 4. Shared Memory: Value, Array

### ตัวอย่างที่ 5: multiprocessing.Value

```python
import multiprocessing
import time

def increment_counter(counter: multiprocessing.Value, lock, n: int) -> None:
    """เพิ่มค่า counter ที่แชร์กัน"""
    for _ in range(n):
        with lock:
            counter.value += 1

if __name__ == '__main__':
    # Value ใช้สำหรับ single value ที่แชร์ระหว่าง processes
    # typecodes: 'i' = int, 'd' = double, 'c' = char
    counter = multiprocessing.Value('i', 0)
    lock = multiprocessing.Lock()
    
    N = 10000
    processes = [
        multiprocessing.Process(
            target=increment_counter,
            args=(counter, lock, N)
        )
        for _ in range(4)
    ]
    
    for p in processes: p.start()
    for p in processes: p.join()
    
    print(f"Expected: {4 * N}")
    print(f"Got: {counter.value}")
    print(f"Correct: {counter.value == 4 * N}")
```

### ตัวอย่างที่ 6: multiprocessing.Array

```python
import multiprocessing
import numpy as np
from typing import Tuple

def worker_fill_array(
    shared_array: multiprocessing.Array,
    start_idx: int,
    end_idx: int,
    multiplier: int
) -> None:
    """เติมค่าใน shared array"""
    for i in range(start_idx, end_idx):
        shared_array[i] = i * multiplier

if __name__ == '__main__':
    size = 1000
    
    # สร้าง shared array ของ doubles
    shared_array = multiprocessing.Array('d', size)
    
    # แบ่งงานให้ 4 processes
    chunk = size // 4
    processes = [
        multiprocessing.Process(
            target=worker_fill_array,
            args=(shared_array, i*chunk, (i+1)*chunk, i+1)
        )
        for i in range(4)
    ]
    
    for p in processes: p.start()
    for p in processes: p.join()
    
    # ดูผลลัพธ์
    result = list(shared_array)
    print(f"First 10 values: {result[:10]}")
    print(f"Middle 10 values (idx 500-509): {result[500:510]}")
    print(f"Array size: {len(result)}")
```

### ตัวอย่างที่ 7: Shared Memory กับ NumPy (Python 3.8+)

```python
from multiprocessing import shared_memory
import numpy as np
import multiprocessing
import time

def process_chunk(
    shm_name: str,
    shape: tuple,
    dtype: str,
    start_row: int,
    end_row: int
) -> None:
    """ประมวลผลส่วนหนึ่งของ array ใน shared memory"""
    # เชื่อมต่อกับ shared memory ที่มีอยู่
    existing_shm = shared_memory.SharedMemory(name=shm_name)
    array = np.ndarray(shape, dtype=dtype, buffer=existing_shm.buf)
    
    # ประมวลผล (เพิ่มค่าแต่ละ element)
    array[start_row:end_row] *= 2
    
    existing_shm.close()  # ไม่ต้อง unlink เพราะ parent จะทำ

if __name__ == '__main__':
    # สร้าง NumPy array ขนาดใหญ่
    original_data = np.random.rand(1000, 100)
    
    # สร้าง shared memory
    shm = shared_memory.SharedMemory(create=True, size=original_data.nbytes)
    shared_array = np.ndarray(original_data.shape, dtype=original_data.dtype, buffer=shm.buf)
    
    # คัดลอกข้อมูลเข้า shared memory
    np.copyto(shared_array, original_data)
    
    # แบ่งงาน
    num_processes = 4
    rows_per_process = len(shared_array) // num_processes
    
    processes = [
        multiprocessing.Process(
            target=process_chunk,
            args=(
                shm.name,
                original_data.shape,
                str(original_data.dtype),
                i * rows_per_process,
                (i+1) * rows_per_process
            )
        )
        for i in range(num_processes)
    ]
    
    start = time.time()
    for p in processes: p.start()
    for p in processes: p.join()
    elapsed = time.time() - start
    
    # ตรวจสอบผลลัพธ์
    expected = original_data * 2
    print(f"Results match: {np.allclose(shared_array, expected)}")
    print(f"Time: {elapsed:.3f}s")
    
    # ทำความสะอาด
    shm.close()
    shm.unlink()
```

---

## 5. Manager Objects

### ตัวอย่างที่ 8: Manager - Shared Python Objects

```python
import multiprocessing
from multiprocessing import Manager
import time

def worker_update_dict(
    shared_dict: dict,
    shared_list: list,
    worker_id: int
) -> None:
    """Worker ที่แก้ไข shared data structures"""
    key = f"worker_{worker_id}"
    
    # อัปเดต dict
    shared_dict[key] = {
        'status': 'running',
        'pid': multiprocessing.current_process().pid
    }
    
    time.sleep(0.5)
    
    # เพิ่มใน list
    shared_list.append(f"Task from worker {worker_id}")
    
    # อัปเดต dict อีกครั้ง
    shared_dict[key]['status'] = 'done'

if __name__ == '__main__':
    with Manager() as manager:
        # สร้าง shared data structures
        shared_dict = manager.dict()
        shared_list = manager.list()
        
        # สร้าง processes
        processes = [
            multiprocessing.Process(
                target=worker_update_dict,
                args=(shared_dict, shared_list, i)
            )
            for i in range(5)
        ]
        
        for p in processes: p.start()
        for p in processes: p.join()
        
        print("Shared dict:")
        for k, v in shared_dict.items():
            print(f"  {k}: {v}")
        
        print("\nShared list:")
        for item in shared_list:
            print(f"  {item}")
```

### ตัวอย่างที่ 9: Manager Namespace

```python
import multiprocessing
from multiprocessing import Manager

def monitor_worker(namespace, worker_id: int) -> None:
    """Worker ที่อัปเดต global state"""
    import time
    namespace.active_workers += 1
    namespace.tasks_completed = getattr(namespace, 'tasks_completed', 0)
    
    for i in range(3):
        time.sleep(0.2)
        namespace.tasks_completed += 1
    
    namespace.active_workers -= 1

if __name__ == '__main__':
    with Manager() as manager:
        ns = manager.Namespace()
        ns.active_workers = 0
        ns.tasks_completed = 0
        ns.start_time = __import__('time').time()
        
        processes = [
            multiprocessing.Process(
                target=monitor_worker,
                args=(ns, i)
            )
            for i in range(4)
        ]
        
        for p in processes: p.start()
        
        # Monitor ขณะที่ processes ทำงาน
        import time
        for _ in range(5):
            time.sleep(0.3)
            elapsed = time.time() - ns.start_time
            print(f"[{elapsed:.1f}s] Active: {ns.active_workers}, "
                  f"Completed: {ns.tasks_completed}")
        
        for p in processes: p.join()
        print(f"\nFinal: {ns.tasks_completed} tasks completed")
```

---

## 6. Queue และ Pipe สำหรับ IPC

### ตัวอย่างที่ 10: multiprocessing.Queue

```python
import multiprocessing
import time
import random
from typing import Any

def producer_process(queue: multiprocessing.Queue, num_items: int) -> None:
    """Producer: สร้างและส่งข้อมูล"""
    for i in range(num_items):
        item = {
            'id': i,
            'data': f"item_{i}",
            'timestamp': time.time()
        }
        queue.put(item)
        print(f"Producer: sent item {i}")
        time.sleep(random.uniform(0.05, 0.15))
    
    # ส่ง sentinel value เพื่อบอกว่าหมดแล้ว
    queue.put(None)
    print("Producer: done")

def consumer_process(queue: multiprocessing.Queue, consumer_id: int) -> None:
    """Consumer: รับและประมวลผลข้อมูล"""
    processed = 0
    
    while True:
        item = queue.get()
        
        if item is None:
            # ส่ง sentinel ต่อให้ consumer อื่น
            queue.put(None)
            break
        
        time.sleep(random.uniform(0.1, 0.3))  # จำลองการประมวลผล
        processed += 1
        print(f"Consumer {consumer_id}: processed item {item['id']}")
    
    print(f"Consumer {consumer_id}: done, processed {processed} items")

if __name__ == '__main__':
    q = multiprocessing.Queue(maxsize=10)
    
    p = multiprocessing.Process(target=producer_process, args=(q, 10))
    c1 = multiprocessing.Process(target=consumer_process, args=(q, 1))
    c2 = multiprocessing.Process(target=consumer_process, args=(q, 2))
    
    p.start()
    c1.start()
    c2.start()
    
    p.join()
    c1.join()
    c2.join()
    
    print("ระบบ Producer-Consumer เสร็จแล้ว")
```

### ตัวอย่างที่ 11: Pipe - สื่อสาร 2 directions

```python
import multiprocessing
import time

def child_process(conn: multiprocessing.connection.Connection) -> None:
    """Child process ที่สื่อสารกับ parent"""
    print(f"Child: รอข้อมูลจาก parent...")
    
    while True:
        data = conn.recv()  # รับข้อมูล
        
        if data == "QUIT":
            print("Child: ได้รับ QUIT signal")
            conn.send("BYE")
            break
        
        print(f"Child: ได้รับ '{data}' กำลังประมวลผล...")
        time.sleep(0.2)
        
        # ส่งผลลัพธ์กลับ
        result = data.upper() + "!"
        conn.send(result)
        print(f"Child: ส่ง '{result}' กลับ")
    
    conn.close()

if __name__ == '__main__':
    # สร้าง Pipe - ได้ connection 2 ปลาย
    parent_conn, child_conn = multiprocessing.Pipe(duplex=True)
    
    # สร้าง child process
    p = multiprocessing.Process(target=child_process, args=(child_conn,))
    p.start()
    child_conn.close()  # Parent ไม่ใช้ child_conn
    
    # ส่งข้อมูลไปยัง child
    messages = ["hello", "world", "python", "QUIT"]
    
    for msg in messages:
        print(f"Parent: ส่ง '{msg}'")
        parent_conn.send(msg)
        
        response = parent_conn.recv()
        print(f"Parent: ได้รับ '{response}'")
        
        if msg == "QUIT":
            break
    
    parent_conn.close()
    p.join()
    print(f"Child exitcode: {p.exitcode}")
```

### ตัวอย่างที่ 12: Pipe สำหรับ One-way Communication

```python
import multiprocessing
import time

def data_generator(pipe_out: multiprocessing.connection.Connection) -> None:
    """สร้างข้อมูลและส่งผ่าน pipe"""
    for i in range(5):
        data = {'index': i, 'value': i * i, 'timestamp': time.time()}
        pipe_out.send(data)
        print(f"Generator: ส่ง data[{i}]")
        time.sleep(0.2)
    pipe_out.send(None)  # Sentinel
    pipe_out.close()

def data_consumer(pipe_in: multiprocessing.connection.Connection) -> None:
    """รับข้อมูลจาก pipe และประมวลผล"""
    total = 0
    count = 0
    
    while True:
        data = pipe_in.recv()
        if data is None:
            break
        
        total += data['value']
        count += 1
        print(f"Consumer: รับ data[{data['index']}] = {data['value']}")
    
    pipe_in.close()
    print(f"Consumer: รับทั้งหมด {count} items, sum = {total}")

if __name__ == '__main__':
    reader, writer = multiprocessing.Pipe(duplex=False)
    
    gen = multiprocessing.Process(target=data_generator, args=(writer,))
    con = multiprocessing.Process(target=data_consumer, args=(reader,))
    
    gen.start()
    con.start()
    
    writer.close()  # Parent ไม่ใช้ writer
    reader.close()  # Parent ไม่ใช้ reader
    
    gen.join()
    con.join()
```

---

## 7. Pool.map(), Pool.apply_async()

### ตัวอย่างที่ 13: Pool.map()

```python
import multiprocessing
import time
import math

def complex_calculation(n: int) -> dict:
    """การคำนวณที่ซับซ้อน"""
    result = sum(math.sin(i) * math.cos(i) for i in range(n))
    return {
        'input': n,
        'result': result,
        'pid': multiprocessing.current_process().pid
    }

if __name__ == '__main__':
    inputs = [10000 * i for i in range(1, 9)]
    
    # Sequential
    start = time.time()
    sequential_results = [complex_calculation(n) for n in inputs]
    seq_time = time.time() - start
    print(f"Sequential: {seq_time:.2f}s")
    
    # Pool.map() - parallel
    start = time.time()
    with multiprocessing.Pool(processes=4) as pool:
        parallel_results = pool.map(complex_calculation, inputs)
    parallel_time = time.time() - start
    print(f"Parallel: {parallel_time:.2f}s")
    print(f"Speedup: {seq_time/parallel_time:.2f}x")
    
    # แสดงว่าใช้ PIDs ต่างกัน
    pids = set(r['pid'] for r in parallel_results)
    print(f"Worker PIDs: {pids}")
```

### ตัวอย่างที่ 14: Pool.apply_async()

```python
import multiprocessing
import time
import random

def process_file(filename: str, operation: str) -> dict:
    """จำลองการประมวลผลไฟล์"""
    time.sleep(random.uniform(0.5, 2.0))
    return {
        'filename': filename,
        'operation': operation,
        'status': 'success',
        'output_file': f"processed_{filename}"
    }

if __name__ == '__main__':
    files = [f"document_{i:03d}.pdf" for i in range(10)]
    
    with multiprocessing.Pool(processes=4) as pool:
        # apply_async ส่งงานทีละชิ้นและ return AsyncResult
        async_results = []
        
        for f in files:
            result = pool.apply_async(
                process_file,
                args=(f, "convert_to_text"),
                callback=lambda r: print(f"✓ Completed: {r['filename']}")
            )
            async_results.append(result)
        
        print("ส่งงานทั้งหมดแล้ว รอผลลัพธ์...")
        
        # รวบรวมผลลัพธ์
        results = []
        for ar in async_results:
            try:
                result = ar.get(timeout=10)
                results.append(result)
            except multiprocessing.TimeoutError:
                print("Timeout!")
    
    print(f"\nประมวลผลสำเร็จ: {len(results)}/{len(files)} ไฟล์")
```

### ตัวอย่างที่ 15: Pool.starmap() และ imap()

```python
import multiprocessing
import time

def multiply(x: float, y: float) -> float:
    """คูณตัวเลข 2 ตัว"""
    return x * y

def slow_square(n: int) -> int:
    """square ที่ช้า"""
    time.sleep(0.1)
    return n ** 2

if __name__ == '__main__':
    pairs = [(i, i+1) for i in range(10)]
    
    with multiprocessing.Pool(4) as pool:
        # starmap - ส่ง argument เป็น tuple
        results = pool.starmap(multiply, pairs)
        print(f"starmap results: {results}")
        
        # imap - lazy evaluation, เหมาะกับข้อมูลขนาดใหญ่
        # imap_unordered - เร็วกว่า แต่ผลลัพธ์ไม่เรียงลำดับ
        numbers = range(20)
        
        print("\nimap_unordered (ผลลัพธ์ไม่เรียงลำดับ):")
        for result in pool.imap_unordered(slow_square, numbers, chunksize=4):
            print(f"  {result}", end="", flush=True)
        print()
```

---

## 8. Synchronization Primitives

### ตัวอย่างที่ 16: multiprocessing.Lock

```python
import multiprocessing
import os
import time

def write_to_file(
    filename: str,
    lock: multiprocessing.Lock,
    worker_id: int,
    n_writes: int
) -> None:
    """เขียนไฟล์ด้วย lock"""
    for i in range(n_writes):
        with lock:
            with open(filename, 'a') as f:
                f.write(f"Worker {worker_id}, line {i}, PID {os.getpid()}\n")
        time.sleep(0.01)

if __name__ == '__main__':
    filename = "test_multiprocess.txt"
    lock = multiprocessing.Lock()
    
    # ล้างไฟล์
    with open(filename, 'w') as f:
        f.write("")
    
    processes = [
        multiprocessing.Process(
            target=write_to_file,
            args=(filename, lock, i, 5)
        )
        for i in range(4)
    ]
    
    for p in processes: p.start()
    for p in processes: p.join()
    
    # ตรวจสอบผลลัพธ์
    with open(filename) as f:
        lines = f.readlines()
    print(f"จำนวนบรรทัดในไฟล์: {len(lines)} (ควรเป็น 20)")
    
    # ทำความสะอาด
    import os
    if os.path.exists(filename):
        os.remove(filename)
```

### ตัวอย่างที่ 17: multiprocessing.Event และ Barrier

```python
import multiprocessing
import time
import random

def phase_worker(
    worker_id: int,
    barrier: multiprocessing.Barrier,
    results: multiprocessing.Queue
) -> None:
    """Worker ที่ทำงาน 2 phases โดยรอทุกคนในแต่ละ phase"""
    
    # Phase 1
    duration = random.uniform(0.5, 2.0)
    print(f"Worker {worker_id}: Phase 1 ใช้เวลา {duration:.1f}s")
    time.sleep(duration)
    
    print(f"Worker {worker_id}: รอที่ barrier...")
    barrier.wait()  # รอจนกว่า workers ทุกคน phase 1 เสร็จ
    
    print(f"Worker {worker_id}: เริ่ม Phase 2!")
    
    # Phase 2
    duration = random.uniform(0.3, 1.0)
    time.sleep(duration)
    results.put(f"Worker {worker_id} done both phases")
    print(f"Worker {worker_id}: เสร็จทั้ง 2 phases")

if __name__ == '__main__':
    num_workers = 4
    barrier = multiprocessing.Barrier(num_workers)
    results_q = multiprocessing.Queue()
    
    processes = [
        multiprocessing.Process(
            target=phase_worker,
            args=(i, barrier, results_q)
        )
        for i in range(num_workers)
    ]
    
    print(f"เริ่ม {num_workers} workers พร้อมกัน...")
    start = time.time()
    
    for p in processes: p.start()
    for p in processes: p.join()
    
    elapsed = time.time() - start
    print(f"\nทั้งหมดเสร็จใน {elapsed:.2f}s")
    
    while not results_q.empty():
        print(f"  {results_q.get()}")
```

---

## 9. CPU-bound vs I/O-bound Tasks

### ตัวอย่างที่ 18: Benchmark CPU vs I/O

```python
import multiprocessing
import threading
import time
import math
import urllib.request

def cpu_task(n: int) -> float:
    """CPU-bound: คำนวณ pi ด้วย Leibniz formula"""
    pi = 0.0
    for i in range(n):
        pi += ((-1) ** i) / (2 * i + 1)
    return pi * 4

def io_task(url: str) -> int:
    """I/O-bound: HTTP request"""
    try:
        with urllib.request.urlopen(url, timeout=5) as resp:
            return len(resp.read())
    except:
        return 0

if __name__ == '__main__':
    N_CPU = 4
    N_ITER = 1_000_000
    cpu_tasks = [N_ITER] * N_CPU
    
    print("=== CPU-bound Tasks ===")
    
    # Sequential
    start = time.time()
    [cpu_task(n) for n in cpu_tasks]
    seq = time.time() - start
    print(f"Sequential: {seq:.2f}s")
    
    # Threads
    start = time.time()
    threads = [threading.Thread(target=cpu_task, args=(n,)) for n in cpu_tasks]
    for t in threads: t.start()
    for t in threads: t.join()
    thr = time.time() - start
    print(f"Threads: {thr:.2f}s (GIL overhead: {thr/seq:.1f}x)")
    
    # Processes
    start = time.time()
    with multiprocessing.Pool(N_CPU) as pool:
        pool.map(cpu_task, cpu_tasks)
    proc = time.time() - start
    print(f"Processes: {proc:.2f}s (Speedup: {seq/proc:.1f}x)")
    
    print("\n=== Decision Guide ===")
    print("CPU-bound → multiprocessing")
    print("I/O-bound → threading หรือ asyncio")
    print("Mixed → concurrent.futures ช่วย abstract ได้")
```

---

## 10. Memory Isolation

### ตัวอย่างที่ 19: Memory Isolation ระหว่าง Processes

```python
import multiprocessing
import os

# Global variable
GLOBAL_DATA = {'value': 100, 'items': [1, 2, 3]}

def child_modify_global() -> None:
    """ลองแก้ไข global data ใน child process"""
    print(f"Child PID={os.getpid()}: GLOBAL_DATA = {GLOBAL_DATA}")
    
    # การแก้ไขนี้จะไม่กระทบ parent
    GLOBAL_DATA['value'] = 999
    GLOBAL_DATA['items'].append(999)
    
    print(f"Child: หลังแก้ไข GLOBAL_DATA = {GLOBAL_DATA}")

if __name__ == '__main__':
    print(f"Parent PID={os.getpid()}: GLOBAL_DATA = {GLOBAL_DATA}")
    
    p = multiprocessing.Process(target=child_modify_global)
    p.start()
    p.join()
    
    # Parent ยังคง global data เดิม!
    print(f"Parent: หลัง child เสร็จ GLOBAL_DATA = {GLOBAL_DATA}")
    print(f"ข้อมูลไม่เปลี่ยนแปลง เพราะ processes มี memory แยกกัน!")
```

---

## 11. Spawning vs Forking

### ตัวอย่างที่ 20: Start Methods

```python
import multiprocessing
import os

def show_process_info(method: str) -> None:
    """แสดงข้อมูล process"""
    print(f"Method: {method}")
    print(f"  PID: {os.getpid()}")
    print(f"  Parent PID: {os.getppid()}")

# Python มี 3 start methods:
# 'spawn'  - สร้าง Python interpreter ใหม่ทั้งหมด (ช้า, ปลอดภัย)
#            default บน Windows และ macOS
# 'fork'   - clone process ด้วย fork() (เร็ว, อาจมีปัญหากับ threads)
#            default บน Unix/Linux
# 'forkserver' - ใช้ server สำหรับ fork (ปลอดภัยกว่า fork)

if __name__ == '__main__':
    # ดู default start method
    print(f"Default start method: {multiprocessing.get_start_method()}")
    
    # เปลี่ยน start method
    # multiprocessing.set_start_method('spawn')  # เปลี่ยนได้แค่ครั้งเดียว
    
    # ใช้ context เพื่อ test methods ต่างๆ
    for method in ['spawn', 'fork', 'forkserver']:
        try:
            ctx = multiprocessing.get_context(method)
            p = ctx.Process(
                target=show_process_info,
                args=(method,)
            )
            p.start()
            p.join()
        except ValueError:
            print(f"Method '{method}' ไม่รองรับบน OS นี้")
```

---

## 12. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 21: Image Processing

```python
import multiprocessing
import time
import os
import random
from dataclasses import dataclass
from concurrent.futures import ProcessPoolExecutor, as_completed

@dataclass
class ImageInfo:
    filename: str
    width: int
    height: int
    format: str

def simulate_image_filter(image: ImageInfo, filter_name: str) -> dict:
    """จำลองการใช้ image filter"""
    # จำลองเวลาที่ใช้ตาม image size
    pixels = image.width * image.height
    processing_time = pixels / 1_000_000 * random.uniform(0.5, 1.5)
    time.sleep(processing_time)
    
    return {
        'filename': image.filename,
        'filter': filter_name,
        'input_size': (image.width, image.height),
        'output_size': (image.width, image.height),
        'pid': os.getpid(),
        'duration': processing_time
    }

def process_image_batch(args: tuple) -> list:
    """ประมวลผล batch ของ images"""
    images, filters = args
    results = []
    
    for image in images:
        for f in filters:
            result = simulate_image_filter(image, f)
            results.append(result)
    
    return results

if __name__ == '__main__':
    # สร้าง test images
    images = [
        ImageInfo(f"photo_{i:04d}.jpg", 
                  random.choice([1920, 3840]),
                  random.choice([1080, 2160]),
                  "JPEG")
        for i in range(20)
    ]
    
    filters = ['grayscale', 'blur', 'sharpen']
    
    print(f"ประมวลผล {len(images)} images x {len(filters)} filters = "
          f"{len(images) * len(filters)} operations")
    
    # แบ่ง images เป็น batches
    num_cpus = multiprocessing.cpu_count()
    batch_size = len(images) // num_cpus
    batches = [
        (images[i:i+batch_size], filters)
        for i in range(0, len(images), batch_size)
    ]
    
    # Sequential
    start = time.time()
    seq_results = []
    for image in images[:5]:  # แค่ 5 รูปสำหรับ demo
        for f in filters:
            result = simulate_image_filter(image, f)
            seq_results.append(result)
    seq_time = time.time() - start
    print(f"\nSequential (5 images): {seq_time:.2f}s")
    
    # Parallel
    start = time.time()
    with ProcessPoolExecutor(max_workers=num_cpus) as executor:
        futures = [executor.submit(process_image_batch, batch) for batch in batches]
        
        all_results = []
        for future in as_completed(futures):
            batch_results = future.result()
            all_results.extend(batch_results)
    
    parallel_time = time.time() - start
    print(f"Parallel ({len(images)} images): {parallel_time:.2f}s")
    
    # สรุปผล
    pids_used = set(r['pid'] for r in all_results)
    total_duration = sum(r['duration'] for r in all_results)
    print(f"\nสรุป:")
    print(f"  Operations: {len(all_results)}")
    print(f"  Worker PIDs: {pids_used}")
    print(f"  Total CPU time: {total_duration:.2f}s")
    print(f"  Wall clock time: {parallel_time:.2f}s")
    print(f"  Parallelism efficiency: {total_duration/parallel_time/len(pids_used)*100:.0f}%")
```

### ตัวอย่างที่ 22: Data Analysis Pipeline

```python
import multiprocessing
from multiprocessing import Pool, Manager
import time
import random
import statistics
from typing import NamedTuple

class SalesRecord(NamedTuple):
    date: str
    product_id: str
    quantity: int
    price: float
    region: str

def generate_sales_data(n: int) -> list[SalesRecord]:
    """สร้าง sales data จำลอง"""
    regions = ['North', 'South', 'East', 'West']
    products = [f'P{i:04d}' for i in range(100)]
    
    data = []
    for _ in range(n):
        data.append(SalesRecord(
            date=f"2024-{random.randint(1,12):02d}-{random.randint(1,28):02d}",
            product_id=random.choice(products),
            quantity=random.randint(1, 100),
            price=round(random.uniform(10, 1000), 2),
            region=random.choice(regions)
        ))
    return data

def analyze_region(records: list[SalesRecord]) -> dict:
    """วิเคราะห์ข้อมูลของ region หนึ่ง"""
    if not records:
        return {}
    
    time.sleep(0.1)  # จำลองการวิเคราะห์
    
    revenues = [r.quantity * r.price for r in records]
    
    return {
        'region': records[0].region,
        'total_records': len(records),
        'total_revenue': round(sum(revenues), 2),
        'avg_revenue': round(statistics.mean(revenues), 2),
        'max_revenue': round(max(revenues), 2),
        'min_revenue': round(min(revenues), 2),
        'std_dev': round(statistics.stdev(revenues) if len(revenues) > 1 else 0, 2),
        'unique_products': len(set(r.product_id for r in records)),
        'top_product': max(
            set(r.product_id for r in records),
            key=lambda p: sum(r.quantity * r.price for r in records if r.product_id == p)
        )
    }

if __name__ == '__main__':
    print("สร้าง data...")
    all_records = generate_sales_data(10000)
    
    # แบ่งข้อมูลตาม region
    by_region = {}
    for record in all_records:
        by_region.setdefault(record.region, []).append(record)
    
    print(f"ข้อมูลทั้งหมด: {len(all_records)} records")
    print(f"Regions: {list(by_region.keys())}")
    
    # Sequential analysis
    start = time.time()
    seq_results = [analyze_region(records) for records in by_region.values()]
    seq_time = time.time() - start
    print(f"\nSequential analysis: {seq_time:.3f}s")
    
    # Parallel analysis
    start = time.time()
    with Pool(processes=len(by_region)) as pool:
        par_results = pool.map(analyze_region, list(by_region.values()))
    par_time = time.time() - start
    print(f"Parallel analysis: {par_time:.3f}s")
    print(f"Speedup: {seq_time/par_time:.2f}x")
    
    # แสดงผลลัพธ์
    print("\nRegional Analysis:")
    for result in sorted(par_results, key=lambda x: x['total_revenue'], reverse=True):
        print(f"\n  Region: {result['region']}")
        print(f"    Records: {result['total_records']:,}")
        print(f"    Revenue: ${result['total_revenue']:,.2f}")
        print(f"    Avg per sale: ${result['avg_revenue']:.2f}")
        print(f"    Top product: {result['top_product']}")
```

### ตัวอย่างที่ 23: CPU-intensive Calculation

```python
import multiprocessing
from concurrent.futures import ProcessPoolExecutor
import time
import hashlib
import os

def crack_hash_chunk(args: tuple) -> tuple[str | None, int]:
    """พยายาม crack hash จาก wordlist chunk"""
    target_hash, words = args
    
    for word in words:
        if hashlib.md5(word.encode()).hexdigest() == target_hash:
            return word, len(words)
    
    return None, len(words)

def parallel_hash_cracker(target_hash: str, wordlist: list[str]) -> str | None:
    """Crack MD5 hash โดยใช้ multiple processes"""
    num_cpus = multiprocessing.cpu_count()
    chunk_size = max(1, len(wordlist) // (num_cpus * 4))
    
    # แบ่ง wordlist เป็น chunks
    chunks = [
        (target_hash, wordlist[i:i+chunk_size])
        for i in range(0, len(wordlist), chunk_size)
    ]
    
    print(f"Cracking hash: {target_hash}")
    print(f"Wordlist size: {len(wordlist)}")
    print(f"Chunks: {len(chunks)}, Workers: {num_cpus}")
    
    start = time.time()
    found = None
    attempts = 0
    
    with ProcessPoolExecutor(max_workers=num_cpus) as executor:
        futures = {executor.submit(crack_hash_chunk, chunk): i 
                   for i, chunk in enumerate(chunks)}
        
        for future in futures:
            result, count = future.result()
            attempts += count
            
            if result is not None:
                found = result
                # ยกเลิก futures ที่เหลือ
                for f in futures:
                    f.cancel()
                break
    
    elapsed = time.time() - start
    
    if found:
        print(f"พบ! '{found}' ใน {elapsed:.3f}s ({attempts:,} attempts)")
    else:
        print(f"ไม่พบ ใน {elapsed:.3f}s ({attempts:,} attempts)")
    
    return found

if __name__ == '__main__':
    # สร้าง wordlist จำลอง
    import random, string
    wordlist = [''.join(random.choices(string.ascii_lowercase, k=6)) 
                for _ in range(50000)]
    
    # เลือก word แบบสุ่มเป็น target
    target_word = random.choice(wordlist)
    target_hash = hashlib.md5(target_word.encode()).hexdigest()
    
    print(f"Target word (hidden): {target_word}")
    result = parallel_hash_cracker(target_hash, wordlist)
    print(f"Cracked: {result}")
    print(f"Correct: {result == target_word}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Parallel Word Counter

**โจทย์:** นับคำใน text files หลายไฟล์พร้อมกันโดยใช้ multiprocessing

**เฉลย:**

```python
import multiprocessing
from collections import Counter
import re
import time
import tempfile
import os

def generate_test_file(filename: str, num_words: int) -> None:
    """สร้างไฟล์ทดสอบ"""
    words = ['python', 'programming', 'language', 'data', 'science',
             'machine', 'learning', 'artificial', 'intelligence', 'code']
    
    import random
    content = ' '.join(random.choices(words, k=num_words))
    with open(filename, 'w') as f:
        f.write(content)

def count_words_in_file(filename: str) -> Counter:
    """นับคำในไฟล์"""
    try:
        with open(filename, 'r', encoding='utf-8') as f:
            text = f.read().lower()
        
        words = re.findall(r'\b[a-z]+\b', text)
        return Counter(words)
    except Exception as e:
        print(f"Error reading {filename}: {e}")
        return Counter()

def merge_counters(counters: list[Counter]) -> Counter:
    """รวม counters ทั้งหมด"""
    total = Counter()
    for counter in counters:
        total.update(counter)
    return total

if __name__ == '__main__':
    # สร้างไฟล์ทดสอบ
    tmp_dir = tempfile.mkdtemp()
    test_files = []
    
    for i in range(8):
        filename = os.path.join(tmp_dir, f"text_{i:02d}.txt")
        generate_test_file(filename, 10000)
        test_files.append(filename)
    
    print(f"สร้าง {len(test_files)} ไฟล์")
    
    # Sequential
    start = time.time()
    seq_counters = [count_words_in_file(f) for f in test_files]
    seq_total = merge_counters(seq_counters)
    seq_time = time.time() - start
    print(f"Sequential: {seq_time:.3f}s")
    
    # Parallel
    start = time.time()
    with multiprocessing.Pool(4) as pool:
        par_counters = pool.map(count_words_in_file, test_files)
    par_total = merge_counters(par_counters)
    par_time = time.time() - start
    print(f"Parallel: {par_time:.3f}s")
    print(f"Speedup: {seq_time/par_time:.2f}x")
    
    # ตรวจสอบ
    print(f"\nTop 5 words:")
    for word, count in par_total.most_common(5):
        print(f"  '{word}': {count:,}")
    
    print(f"\nResults match: {seq_total == par_total}")
    
    # ทำความสะอาด
    import shutil
    shutil.rmtree(tmp_dir)
```

---

### แบบฝึกหัดที่ 2: Parallel Matrix Multiplication

**เฉลย:**

```python
import multiprocessing
import time
import random

def multiply_row(args: tuple) -> list[float]:
    """คูณ row หนึ่งกับ matrix B"""
    row, matrix_b = args
    n = len(matrix_b[0])
    result_row = []
    for j in range(n):
        val = sum(row[k] * matrix_b[k][j] for k in range(len(row)))
        result_row.append(val)
    return result_row

def matrix_multiply_parallel(A: list, B: list, num_workers: int = 4) -> list:
    """คูณ matrices แบบ parallel"""
    args = [(row, B) for row in A]
    
    with multiprocessing.Pool(num_workers) as pool:
        result = pool.map(multiply_row, args)
    
    return result

if __name__ == '__main__':
    # สร้าง matrices ขนาด 200x200
    N = 200
    A = [[random.random() for _ in range(N)] for _ in range(N)]
    B = [[random.random() for _ in range(N)] for _ in range(N)]
    
    print(f"Matrix multiplication: {N}x{N}")
    
    # Sequential
    start = time.time()
    result_seq = []
    for row in A:
        result_seq.append(multiply_row((row, B)))
    seq_time = time.time() - start
    print(f"Sequential: {seq_time:.2f}s")
    
    # Parallel
    start = time.time()
    result_par = matrix_multiply_parallel(A, B)
    par_time = time.time() - start
    print(f"Parallel: {par_time:.2f}s")
    print(f"Speedup: {seq_time/par_time:.2f}x")
    
    # ตรวจสอบความถูกต้อง (row แรก)
    diff = max(abs(result_seq[0][j] - result_par[0][j]) for j in range(N))
    print(f"Max difference (row 0): {diff:.2e}")
    print(f"Results match: {diff < 1e-10}")
```

---

### แบบฝึกหัดที่ 3-8 (สรุปย่อ)

แบบฝึกหัดที่ 3: สร้าง parallel text processor ที่ใช้ Manager dict เก็บ word frequencies

แบบฝึกหัดที่ 4: สร้าง distributed task queue ด้วย multiprocessing.Queue และหลาย workers

แบบฝึกหัดที่ 5: สร้าง parallel file checksum calculator ด้วย MD5/SHA256

แบบฝึกหัดที่ 6: สร้าง parallel web scraper ที่ใช้ processes แทน threads

แบบฝึกหัดที่ 7: สร้าง parallel sorting algorithm (parallel merge sort)

แบบฝึกหัดที่ 8: สร้าง Map-Reduce framework จำลองด้วย multiprocessing

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Process vs Thread** - ความแตกต่างและเมื่อไรควรใช้อะไร
2. **multiprocessing Module** - การสร้างและจัดการ processes
3. **ProcessPoolExecutor** - การใช้ process pool อย่างมีประสิทธิภาพ
4. **Shared Memory** - Value, Array, shared_memory สำหรับแชร์ข้อมูล
5. **Manager Objects** - แชร์ Python data structures ระหว่าง processes
6. **IPC** - Queue และ Pipe สำหรับสื่อสารระหว่าง processes
7. **Pool methods** - map, apply_async, starmap, imap
8. **Synchronization** - Lock, Event, Barrier
9. **Start Methods** - spawn, fork, forkserver

### เมื่อไรควรใช้ Multiprocessing?

- **CPU-bound tasks**: การคำนวณหนัก, image processing, data analysis
- **เมื่อต้องการหลีกเลี่ยง GIL**: การประมวลผลที่ต้องการ true parallelism
- **Memory isolation**: เมื่อต้องการป้องกันไม่ให้ process หนึ่ง crash กระทบอื่น

### ข้อควรระวัง

- Process startup overhead สูงกว่า thread มาก
- การแชร์ข้อมูลต้องใช้ IPC ซึ่งมี overhead
- ไม่ควรส่ง objects ขนาดใหญ่ผ่าน Queue/Pipe บ่อยๆ
- ต้องใช้ `if __name__ == '__main__':` บน Windows

---

*ถัดไป: Part 39 - Async/Await & asyncio*
