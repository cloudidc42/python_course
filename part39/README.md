# Part 39: Async/Await & asyncio

## บทนำ

Asynchronous programming ด้วย `asyncio` เป็นวิธีที่ Python แก้ปัญหา I/O-bound tasks ได้อย่างมีประสิทธิภาพโดยไม่ต้องใช้หลาย threads หรือ processes ด้วย single thread และ event loop เราสามารถจัดการ thousands of concurrent connections ได้

---

## 1. Asynchronous Programming Concepts

### Synchronous vs Asynchronous

```
Synchronous (blocking):
Thread: ████ wait ████ wait ████ wait ████
        work  I/O  work  I/O  work  I/O  work
        (CPU รอ I/O อยู่เปล่าๆ)

Asynchronous (non-blocking):
Thread: ████████████████████
        task1 task2 task3 task4
        (สลับทำงานระหว่างรอ I/O)
```

### ตัวอย่างที่ 1: เปรียบเทียบ Sync vs Async

```python
import asyncio
import time

# Synchronous version
def sync_fetch(url: str) -> str:
    time.sleep(1)  # จำลอง network request
    return f"Data from {url}"

def sync_main() -> None:
    start = time.time()
    results = [sync_fetch(f"url_{i}") for i in range(5)]
    print(f"Sync: {time.time()-start:.2f}s, {len(results)} results")

# Asynchronous version
async def async_fetch(url: str) -> str:
    await asyncio.sleep(1)  # Non-blocking wait
    return f"Data from {url}"

async def async_main() -> None:
    start = time.time()
    # ทำงานพร้อมกัน!
    results = await asyncio.gather(*[async_fetch(f"url_{i}") for i in range(5)])
    print(f"Async: {time.time()-start:.2f}s, {len(results)} results")

# รัน
sync_main()      # 5 seconds
asyncio.run(async_main())  # ~1 second
```

---

## 2. Event Loop

Event loop เป็นหัวใจของ asyncio ที่จัดการ coroutines และ I/O operations

### ตัวอย่างที่ 2: Event Loop พื้นฐาน

```python
import asyncio

async def greet(name: str, delay: float) -> str:
    print(f"สวัสดี {name}! (รอ {delay}s)")
    await asyncio.sleep(delay)
    print(f"ลาก่อน {name}!")
    return f"Done with {name}"

async def main() -> None:
    # asyncio.run() สร้าง event loop และรัน coroutine
    loop = asyncio.get_event_loop()
    print(f"Event loop: {loop}")
    print(f"Running: {loop.is_running()}")
    
    result = await greet("Alice", 1.0)
    print(f"Result: {result}")

asyncio.run(main())
```

### ตัวอย่างที่ 3: Event Loop จัดการหลาย Tasks

```python
import asyncio
import time

async def task_a() -> str:
    print("Task A: เริ่ม")
    await asyncio.sleep(2)  # ระหว่างรอ event loop ไปทำ task อื่น
    print("Task A: เสร็จ")
    return "Result A"

async def task_b() -> str:
    print("Task B: เริ่ม")
    await asyncio.sleep(1)
    print("Task B: เสร็จ")
    return "Result B"

async def task_c() -> str:
    print("Task C: เริ่ม")
    await asyncio.sleep(3)
    print("Task C: เสร็จ")
    return "Result C"

async def main() -> None:
    start = time.time()
    
    # ทำงานพร้อมกัน - ทั้งหมดใช้เวลาเท่ากับงานที่นานที่สุด (~3s)
    results = await asyncio.gather(task_a(), task_b(), task_c())
    
    elapsed = time.time() - start
    print(f"\nผลลัพธ์: {results}")
    print(f"ใช้เวลา: {elapsed:.2f}s (แทนที่จะเป็น 6s แบบ sequential)")

asyncio.run(main())
```

---

## 3. async/await Syntax

### ตัวอย่างที่ 4: Coroutine พื้นฐาน

```python
import asyncio
from typing import Optional

# async def สร้าง coroutine function
async def fetch_user(user_id: int) -> Optional[dict]:
    """ดึงข้อมูล user (จำลอง)"""
    await asyncio.sleep(0.5)  # จำลอง DB query
    
    if user_id <= 0:
        return None
    
    return {
        'id': user_id,
        'name': f'User_{user_id}',
        'email': f'user{user_id}@example.com'
    }

async def fetch_user_posts(user_id: int) -> list[dict]:
    """ดึง posts ของ user"""
    await asyncio.sleep(0.3)
    return [
        {'id': i, 'title': f'Post {i} by User {user_id}', 'user_id': user_id}
        for i in range(1, 4)
    ]

async def get_user_with_posts(user_id: int) -> dict:
    """ดึง user และ posts พร้อมกัน"""
    # await รอให้ coroutine เสร็จ
    user, posts = await asyncio.gather(
        fetch_user(user_id),
        fetch_user_posts(user_id)
    )
    
    if user is None:
        raise ValueError(f"User {user_id} not found")
    
    return {**user, 'posts': posts}

async def main() -> None:
    # ดึงหลาย users พร้อมกัน
    user_ids = [1, 2, 3, 4, 5]
    users = await asyncio.gather(*[get_user_with_posts(uid) for uid in user_ids])
    
    for user in users:
        print(f"User {user['name']}: {len(user['posts'])} posts")

asyncio.run(main())
```

### ตัวอย่างที่ 5: async/await กับ Exception Handling

```python
import asyncio

class APIError(Exception):
    def __init__(self, status_code: int, message: str):
        self.status_code = status_code
        super().__init__(f"API Error {status_code}: {message}")

async def api_call(endpoint: str, should_fail: bool = False) -> dict:
    """จำลอง API call ที่อาจ fail"""
    await asyncio.sleep(0.1)
    
    if should_fail:
        raise APIError(500, "Internal Server Error")
    
    return {'endpoint': endpoint, 'status': 'ok', 'data': [1, 2, 3]}

async def safe_api_call(endpoint: str) -> dict | None:
    """API call ที่มี error handling"""
    try:
        result = await api_call(endpoint, should_fail='/error' in endpoint)
        return result
    except APIError as e:
        print(f"Error calling {endpoint}: {e}")
        return None
    except asyncio.TimeoutError:
        print(f"Timeout calling {endpoint}")
        return None

async def main() -> None:
    endpoints = [
        "/api/users",
        "/api/products",
        "/api/error/server",     # จะ fail
        "/api/orders",
    ]
    
    results = await asyncio.gather(
        *[safe_api_call(ep) for ep in endpoints],
        return_exceptions=False  # ไม่ raise exceptions
    )
    
    successful = [r for r in results if r is not None]
    print(f"\nสำเร็จ: {len(successful)}/{len(endpoints)}")

asyncio.run(main())
```

---

## 4. Coroutines

### ตัวอย่างที่ 6: Coroutine Types

```python
import asyncio
import inspect

# 1. Coroutine function (async def)
async def simple_coroutine(x: int) -> int:
    await asyncio.sleep(0)
    return x * 2

# 2. ตรวจสอบว่าเป็น coroutine
coro = simple_coroutine(5)
print(f"Is coroutine: {asyncio.iscoroutine(coro)}")  # True
print(f"Type: {type(coro)}")  # <class 'coroutine'>

# 3. Coroutine function check
print(f"Is coroutine function: {asyncio.iscoroutinefunction(simple_coroutine)}")

# ต้อง await หรือ schedule ให้รัน
async def main() -> None:
    result = await coro  # await ทำให้รัน
    print(f"Result: {result}")

asyncio.run(main())
```

### ตัวอย่างที่ 7: Chaining Coroutines

```python
import asyncio
from typing import AsyncIterator

async def step1(data: str) -> str:
    """ขั้นตอนที่ 1: validate"""
    await asyncio.sleep(0.1)
    return data.strip().lower()

async def step2(data: str) -> str:
    """ขั้นตอนที่ 2: transform"""
    await asyncio.sleep(0.1)
    return data.replace(' ', '_')

async def step3(data: str) -> dict:
    """ขั้นตอนที่ 3: enrich"""
    await asyncio.sleep(0.1)
    return {
        'original': data,
        'slug': data,
        'length': len(data),
        'words': data.split('_')
    }

async def pipeline(input_data: str) -> dict:
    """Pipeline ที่เชื่อม coroutines"""
    validated = await step1(input_data)
    transformed = await step2(validated)
    enriched = await step3(transformed)
    return enriched

async def main() -> None:
    inputs = ["  Hello World  ", "  Python Programming  ", " Async Await "]
    
    # ประมวลผลพร้อมกัน
    results = await asyncio.gather(*[pipeline(inp) for inp in inputs])
    
    for result in results:
        print(f"Slug: {result['slug']}, Words: {result['words']}")

asyncio.run(main())
```

---

## 5. asyncio.run()

### ตัวอย่างที่ 8: asyncio.run() และ Event Loop Management

```python
import asyncio

async def simple_task(n: int) -> int:
    await asyncio.sleep(0.1)
    return n ** 2

# asyncio.run() - วิธีที่แนะนำใน Python 3.7+
# สร้าง event loop ใหม่, รัน coroutine, แล้วปิด loop
result = asyncio.run(simple_task(5))
print(f"Result: {result}")

# asyncio.run() ไม่สามารถเรียกซ้อนกันได้
# ถ้าอยู่ใน Jupyter หรือ async context ต้องใช้ await แทน

async def nested_example() -> None:
    # ถ้าอยู่ใน async context แล้ว ใช้ await โดยตรง
    result = await simple_task(10)
    print(f"Nested result: {result}")
    
    # หรือใช้ asyncio.get_event_loop()
    loop = asyncio.get_running_loop()
    print(f"Current loop: {loop}")

asyncio.run(nested_example())
```

---

## 6. asyncio.create_task()

### ตัวอย่างที่ 9: Tasks vs Coroutines

```python
import asyncio
import time

async def background_worker(name: str, delay: float) -> str:
    """Background task"""
    print(f"{name}: เริ่ม")
    await asyncio.sleep(delay)
    print(f"{name}: เสร็จ")
    return f"{name} completed"

async def main() -> None:
    print("=== Coroutine (ทำทีละงาน) ===")
    start = time.time()
    await background_worker("Task A", 1.0)
    await background_worker("Task B", 1.0)
    print(f"Sequential: {time.time()-start:.2f}s")
    
    print("\n=== create_task (ทำพร้อมกัน) ===")
    start = time.time()
    
    # create_task() schedule coroutine ให้รันโดยเร็วที่สุด
    task_a = asyncio.create_task(background_worker("Task A", 1.0))
    task_b = asyncio.create_task(background_worker("Task B", 1.0))
    
    # รอผลลัพธ์
    result_a = await task_a
    result_b = await task_b
    
    print(f"Concurrent: {time.time()-start:.2f}s")
    print(f"Results: {result_a}, {result_b}")

asyncio.run(main())
```

### ตัวอย่างที่ 10: Task Cancellation

```python
import asyncio

async def long_operation(name: str) -> str:
    try:
        print(f"{name}: เริ่มทำงาน")
        for i in range(10):
            await asyncio.sleep(0.5)
            print(f"{name}: progress {i+1}/10")
        return f"{name}: เสร็จสมบูรณ์"
    except asyncio.CancelledError:
        print(f"{name}: ถูก cancel!")
        # Cleanup code here
        raise  # ต้อง re-raise CancelledError

async def main() -> None:
    task = asyncio.create_task(long_operation("MyTask"))
    
    # รอ 1.5 วินาที แล้ว cancel
    await asyncio.sleep(1.5)
    
    print("กำลัง cancel task...")
    task.cancel()
    
    try:
        await task
    except asyncio.CancelledError:
        print("Task ถูก cancel สำเร็จ")
    
    print(f"Task cancelled: {task.cancelled()}")
    print(f"Task done: {task.done()}")

asyncio.run(main())
```

### ตัวอย่างที่ 11: Task Groups (Python 3.11+)

```python
import asyncio
import sys

async def fetch_data(url: str) -> dict:
    await asyncio.sleep(0.5)
    return {'url': url, 'data': 'content'}

async def main() -> None:
    # TaskGroup - ใหม่ใน Python 3.11
    if sys.version_info >= (3, 11):
        async with asyncio.TaskGroup() as tg:
            task1 = tg.create_task(fetch_data("url1"))
            task2 = tg.create_task(fetch_data("url2"))
            task3 = tg.create_task(fetch_data("url3"))
        # ทุก tasks เสร็จแล้วเมื่อออกจาก context
        print(f"Results: {[task1.result(), task2.result(), task3.result()]}")
    else:
        # Python < 3.11
        results = await asyncio.gather(
            fetch_data("url1"),
            fetch_data("url2"),
            fetch_data("url3")
        )
        print(f"Results: {results}")

asyncio.run(main())
```

---

## 7. asyncio.gather()

### ตัวอย่างที่ 12: gather() - รันหลาย coroutines พร้อมกัน

```python
import asyncio
import time

async def fetch_weather(city: str) -> dict:
    """ดึงข้อมูลอากาศ (จำลอง)"""
    import random
    await asyncio.sleep(random.uniform(0.5, 1.5))
    return {
        'city': city,
        'temp': random.randint(20, 35),
        'humidity': random.randint(40, 90),
        'condition': random.choice(['sunny', 'cloudy', 'rainy'])
    }

async def main() -> None:
    cities = ['Bangkok', 'Tokyo', 'London', 'New York', 'Sydney']
    
    start = time.time()
    
    # gather รัน coroutines พร้อมกัน และรอทั้งหมดเสร็จ
    weather_data = await asyncio.gather(
        *[fetch_weather(city) for city in cities]
    )
    
    elapsed = time.time() - start
    
    print(f"ดึงข้อมูล {len(cities)} เมืองใน {elapsed:.2f}s")
    for data in weather_data:
        print(f"  {data['city']}: {data['temp']}°C, {data['condition']}")

asyncio.run(main())
```

### ตัวอย่างที่ 13: gather() กับ return_exceptions

```python
import asyncio

async def risky_operation(n: int) -> int:
    await asyncio.sleep(0.1)
    if n % 3 == 0:
        raise ValueError(f"ไม่ชอบเลข {n}!")
    return n * 2

async def main() -> None:
    operations = list(range(10))
    
    # return_exceptions=True - ไม่ raise ทันที เก็บ exception ไว้ใน results
    results = await asyncio.gather(
        *[risky_operation(n) for n in operations],
        return_exceptions=True
    )
    
    successes = []
    failures = []
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            failures.append((operations[i], result))
        else:
            successes.append(result)
    
    print(f"สำเร็จ: {successes}")
    print(f"ล้มเหลว: {[(n, str(e)) for n, e in failures]}")
    
    print("\n--- Without return_exceptions (จะ raise ทันที) ---")
    try:
        results = await asyncio.gather(
            *[risky_operation(n) for n in operations[:5]],
            return_exceptions=False  # default
        )
    except ValueError as e:
        print(f"Error: {e} (tasks อื่นๆ ยังคงรันอยู่แต่ results หายไป)")

asyncio.run(main())
```

---

## 8. asyncio.wait()

### ตัวอย่างที่ 14: asyncio.wait() - ยืดหยุ่นกว่า gather

```python
import asyncio
import time
import random

async def variable_task(task_id: int) -> str:
    duration = random.uniform(0.5, 3.0)
    await asyncio.sleep(duration)
    return f"Task {task_id} done ({duration:.1f}s)"

async def main() -> None:
    tasks = [
        asyncio.create_task(variable_task(i))
        for i in range(6)
    ]
    
    # FIRST_COMPLETED - รอแค่ task แรกที่เสร็จ
    print("=== FIRST_COMPLETED ===")
    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED
    )
    print(f"เสร็จแล้ว: {len(done)}, ยังรอ: {len(pending)}")
    for task in done:
        print(f"  {task.result()}")
    
    # ยกเลิก tasks ที่เหลือ
    for task in pending:
        task.cancel()
    
    # รอให้ cancel เสร็จ
    await asyncio.gather(*pending, return_exceptions=True)
    print("Cancel tasks ที่เหลือแล้ว")
    
    print("\n=== ALL_COMPLETED with timeout ===")
    tasks2 = [
        asyncio.create_task(variable_task(i))
        for i in range(4)
    ]
    
    done2, pending2 = await asyncio.wait(tasks2, timeout=2.0)
    print(f"เสร็จใน 2s: {len(done2)}, ยังค้างอยู่: {len(pending2)}")
    
    for task in pending2:
        task.cancel()
    await asyncio.gather(*pending2, return_exceptions=True)

asyncio.run(main())
```

---

## 9. asyncio.Queue

### ตัวอย่างที่ 15: Async Queue - Producer-Consumer

```python
import asyncio
import random
import time

async def async_producer(
    queue: asyncio.Queue,
    producer_id: int,
    n_items: int
) -> None:
    """Async producer"""
    for i in range(n_items):
        item = {
            'producer': producer_id,
            'item_id': f"P{producer_id}-{i:03d}",
            'data': random.random()
        }
        await queue.put(item)
        print(f"Producer {producer_id}: put {item['item_id']}")
        await asyncio.sleep(random.uniform(0.1, 0.3))
    
    print(f"Producer {producer_id}: เสร็จแล้ว")

async def async_consumer(
    queue: asyncio.Queue,
    consumer_id: int
) -> int:
    """Async consumer"""
    processed = 0
    
    while True:
        try:
            item = await asyncio.wait_for(queue.get(), timeout=2.0)
            
            await asyncio.sleep(random.uniform(0.2, 0.5))
            print(f"Consumer {consumer_id}: processed {item['item_id']}")
            processed += 1
            queue.task_done()
            
        except asyncio.TimeoutError:
            break
    
    print(f"Consumer {consumer_id}: เสร็จ ({processed} items)")
    return processed

async def main() -> None:
    q = asyncio.Queue(maxsize=10)
    
    # สร้าง producers และ consumers
    producers = [
        asyncio.create_task(async_producer(q, i, 5))
        for i in range(2)
    ]
    consumers = [
        asyncio.create_task(async_consumer(q, i))
        for i in range(3)
    ]
    
    # รอ producers เสร็จ
    await asyncio.gather(*producers)
    
    # รอให้ queue ว่าง
    await q.join()
    
    # หยุด consumers
    for c in consumers:
        c.cancel()
    
    results = await asyncio.gather(*consumers, return_exceptions=True)
    print(f"\nProcessed: {[r for r in results if isinstance(r, int)]}")

asyncio.run(main())
```

---

## 10. asyncio Streams

### ตัวอย่างที่ 16: Echo Server ด้วย asyncio streams

```python
import asyncio

async def handle_client(
    reader: asyncio.StreamReader,
    writer: asyncio.StreamWriter
) -> None:
    """จัดการ client connection"""
    addr = writer.get_extra_info('peername')
    print(f"New connection from {addr}")
    
    try:
        while True:
            data = await reader.read(1024)
            if not data:
                break
            
            message = data.decode('utf-8').strip()
            print(f"Received from {addr}: {message}")
            
            response = f"Echo: {message}\n"
            writer.write(response.encode('utf-8'))
            await writer.drain()  # flush buffer
            
    except asyncio.IncompleteReadError:
        pass
    except ConnectionResetError:
        pass
    finally:
        print(f"Connection closed: {addr}")
        writer.close()
        await writer.wait_closed()

async def run_server(host: str = '127.0.0.1', port: int = 8888) -> None:
    """รัน echo server"""
    server = await asyncio.start_server(
        handle_client,
        host,
        port
    )
    
    async with server:
        print(f"Server รันอยู่ที่ {host}:{port}")
        await server.serve_forever()

async def run_client(host: str = '127.0.0.1', port: int = 8888) -> None:
    """Client ที่เชื่อมต่อ server"""
    reader, writer = await asyncio.open_connection(host, port)
    
    messages = ["Hello!", "How are you?", "Goodbye!"]
    
    for msg in messages:
        writer.write(f"{msg}\n".encode())
        await writer.drain()
        
        response = await asyncio.wait_for(reader.readline(), timeout=5.0)
        print(f"Server says: {response.decode().strip()}")
        
        await asyncio.sleep(0.1)
    
    writer.close()
    await writer.wait_closed()

async def demo_server() -> None:
    """Demo server กับ client"""
    # รัน server ใน background
    server_task = asyncio.create_task(run_server())
    
    # รอให้ server เริ่มก่อน
    await asyncio.sleep(0.1)
    
    # รัน client
    await run_client()
    
    # หยุด server
    server_task.cancel()
    try:
        await server_task
    except asyncio.CancelledError:
        pass

# asyncio.run(demo_server())  # uncomment เพื่อทดสอบ
print("ดูตัวอย่าง streams ด้านบน")
```

---

## 11. aiohttp สำหรับ Async HTTP

### ตัวอย่างที่ 17: aiohttp Client

```python
# ติดตั้ง: pip install aiohttp
import asyncio
import time

async def fetch_with_aiohttp() -> None:
    """ตัวอย่างการใช้ aiohttp"""
    try:
        import aiohttp
    except ImportError:
        print("aiohttp ไม่ได้ติดตั้ง - แสดง mock version")
        await demo_mock_http()
        return
    
    async with aiohttp.ClientSession() as session:
        urls = [
            "https://httpbin.org/get",
            "https://httpbin.org/uuid",
            "https://httpbin.org/headers",
        ]
        
        async def fetch_one(url: str) -> dict:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as response:
                return {
                    'url': url,
                    'status': response.status,
                    'size': len(await response.text())
                }
        
        start = time.time()
        results = await asyncio.gather(*[fetch_one(url) for url in urls])
        elapsed = time.time() - start
        
        for r in results:
            print(f"{r['url']}: {r['status']} ({r['size']} bytes)")
        print(f"ทั้งหมด: {elapsed:.2f}s")

async def demo_mock_http() -> None:
    """Mock HTTP demo โดยไม่ต้องใช้ aiohttp"""
    async def mock_get(url: str) -> dict:
        await asyncio.sleep(0.3)
        return {'url': url, 'status': 200, 'data': 'mock response'}
    
    start = time.time()
    urls = [f"https://api.example.com/data/{i}" for i in range(5)]
    results = await asyncio.gather(*[mock_get(url) for url in urls])
    elapsed = time.time() - start
    
    print(f"Mock HTTP: {len(results)} requests in {elapsed:.2f}s")
    for r in results:
        print(f"  {r['url']}: {r['status']}")

asyncio.run(fetch_with_aiohttp())
```

### ตัวอย่างที่ 18: aiohttp กับ Rate Limiting

```python
import asyncio
import time
from typing import Optional

class RateLimiter:
    """Rate limiter สำหรับ async requests"""
    
    def __init__(self, max_rate: float, period: float = 1.0):
        self.max_rate = max_rate
        self.period = period
        self._semaphore = asyncio.Semaphore(int(max_rate))
        self._reset_task: Optional[asyncio.Task] = None
    
    async def __aenter__(self):
        await self._semaphore.acquire()
        return self
    
    async def __aexit__(self, *args):
        # Release หลังจาก period วินาที
        asyncio.create_task(self._release_after(self.period))
    
    async def _release_after(self, delay: float) -> None:
        await asyncio.sleep(delay)
        self._semaphore.release()

async def rate_limited_fetch(
    url: str,
    rate_limiter: RateLimiter,
    session_id: int
) -> dict:
    """ดึงข้อมูลพร้อม rate limiting"""
    async with rate_limiter:
        start = time.time()
        await asyncio.sleep(0.1)  # จำลอง network
        elapsed = time.time() - start
        return {'url': url, 'session': session_id, 'elapsed': elapsed}

async def main() -> None:
    # จำกัด 3 requests ต่อวินาที
    limiter = RateLimiter(max_rate=3, period=1.0)
    
    urls = [f"https://api.example.com/item/{i}" for i in range(10)]
    
    start = time.time()
    results = await asyncio.gather(*[
        rate_limited_fetch(url, limiter, i)
        for i, url in enumerate(urls)
    ])
    elapsed = time.time() - start
    
    print(f"ส่ง {len(results)} requests ด้วย rate limiting")
    print(f"ใช้เวลา: {elapsed:.2f}s")
    for r in results[:3]:
        print(f"  {r['url']}: {r['elapsed']:.3f}s")

asyncio.run(main())
```

---

## 12. aiofiles สำหรับ Async File I/O

### ตัวอย่างที่ 19: aiofiles

```python
# ติดตั้ง: pip install aiofiles
import asyncio
import os

async def async_file_demo() -> None:
    """Demo การใช้ aiofiles"""
    try:
        import aiofiles
        
        # เขียนไฟล์แบบ async
        filename = "/tmp/test_async.txt"
        
        async with aiofiles.open(filename, 'w', encoding='utf-8') as f:
            await f.write("Hello, Async World!\n")
            await f.write("Python asyncio is powerful!\n")
        
        # อ่านไฟล์แบบ async
        async with aiofiles.open(filename, 'r', encoding='utf-8') as f:
            content = await f.read()
        
        print(f"File content:\n{content}")
        
        # อ่านทีละบรรทัด
        async with aiofiles.open(filename, 'r') as f:
            async for line in f:
                print(f"Line: {line.rstrip()}")
        
        os.remove(filename)
        
    except ImportError:
        print("aiofiles ไม่ได้ติดตั้ง - ใช้ asyncio.to_thread แทน")
        await mock_async_file()

async def mock_async_file() -> None:
    """จำลอง async file I/O ด้วย asyncio.to_thread"""
    
    def read_file(filename: str) -> str:
        with open(filename, 'r') as f:
            return f.read()
    
    def write_file(filename: str, content: str) -> None:
        with open(filename, 'w') as f:
            f.write(content)
    
    filename = "/tmp/test_thread.txt"
    
    # รัน blocking I/O ใน thread pool
    await asyncio.to_thread(write_file, filename, "Hello from thread!\n" * 3)
    content = await asyncio.to_thread(read_file, filename)
    
    print(f"File content (via thread):\n{content}")
    os.remove(filename)

asyncio.run(async_file_demo())
```

### ตัวอย่างที่ 20: Async File Processing

```python
import asyncio
import os
import tempfile

async def process_file_async(filename: str) -> dict:
    """ประมวลผลไฟล์แบบ async"""
    # อ่านไฟล์ใน thread pool (ไม่บล็อก event loop)
    content = await asyncio.to_thread(
        lambda: open(filename).read()
    )
    
    # ประมวลผลใน coroutine
    words = content.lower().split()
    word_count = len(words)
    unique_words = len(set(words))
    
    return {
        'filename': os.path.basename(filename),
        'size': len(content),
        'words': word_count,
        'unique_words': unique_words
    }

async def main() -> None:
    # สร้างไฟล์ทดสอบ
    tmp_dir = tempfile.mkdtemp()
    filenames = []
    
    texts = [
        "Python is a programming language that is easy to learn",
        "asyncio enables concurrent code using coroutines",
        "The event loop runs tasks and callbacks",
    ]
    
    for i, text in enumerate(texts):
        filename = os.path.join(tmp_dir, f"file_{i}.txt")
        with open(filename, 'w') as f:
            f.write(text * 100)
        filenames.append(filename)
    
    # ประมวลผลพร้อมกัน
    results = await asyncio.gather(
        *[process_file_async(f) for f in filenames]
    )
    
    for r in results:
        print(f"{r['filename']}: {r['words']} words, {r['unique_words']} unique")
    
    # ทำความสะอาด
    import shutil
    shutil.rmtree(tmp_dir)

asyncio.run(main())
```

---

## 13. Async Context Managers

### ตัวอย่างที่ 21: Custom Async Context Manager

```python
import asyncio
from contextlib import asynccontextmanager
from typing import AsyncGenerator

class AsyncDatabaseConnection:
    """จำลอง async database connection"""
    
    def __init__(self, dsn: str):
        self.dsn = dsn
        self._connected = False
    
    async def connect(self) -> None:
        await asyncio.sleep(0.1)  # จำลองการเชื่อมต่อ
        self._connected = True
        print(f"Connected to: {self.dsn}")
    
    async def disconnect(self) -> None:
        await asyncio.sleep(0.05)
        self._connected = False
        print(f"Disconnected from: {self.dsn}")
    
    async def execute(self, query: str) -> list:
        if not self._connected:
            raise RuntimeError("Not connected!")
        await asyncio.sleep(0.1)
        return [{'query': query, 'rows': 10}]
    
    # async context manager protocol
    async def __aenter__(self) -> 'AsyncDatabaseConnection':
        await self.connect()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb) -> bool:
        await self.disconnect()
        return False  # ไม่ suppress exceptions

@asynccontextmanager
async def managed_connection(dsn: str) -> AsyncGenerator:
    """Factory สำหรับ async connection"""
    conn = AsyncDatabaseConnection(dsn)
    try:
        await conn.connect()
        yield conn
    except Exception as e:
        print(f"Connection error: {e}")
        raise
    finally:
        await conn.disconnect()

async def main() -> None:
    # วิธีที่ 1: class-based
    async with AsyncDatabaseConnection("postgresql://localhost/mydb") as db:
        results = await db.execute("SELECT * FROM users")
        print(f"Query results: {results}")
    
    print()
    
    # วิธีที่ 2: generator-based
    async with managed_connection("postgresql://localhost/orders") as db:
        results = await db.execute("SELECT * FROM orders")
        print(f"Orders: {results}")

asyncio.run(main())
```

---

## 14. Async Generators

### ตัวอย่างที่ 22: Async Generator

```python
import asyncio
from typing import AsyncGenerator, AsyncIterator

async def async_range(start: int, stop: int, step: int = 1) -> AsyncGenerator[int, None]:
    """Async version ของ range()"""
    current = start
    while current < stop:
        await asyncio.sleep(0)  # yield control to event loop
        yield current
        current += step

async def fetch_pages(base_url: str, max_pages: int = 5) -> AsyncGenerator[dict, None]:
    """Generator สำหรับ paginated API"""
    page = 1
    
    while page <= max_pages:
        await asyncio.sleep(0.1)  # จำลอง API call
        
        data = {
            'page': page,
            'total_pages': max_pages,
            'items': [f"item_{(page-1)*10 + i}" for i in range(10)],
            'has_next': page < max_pages
        }
        
        yield data
        
        if not data['has_next']:
            break
        
        page += 1

async def main() -> None:
    # ใช้ async generator
    print("=== async_range ===")
    async for i in async_range(0, 10, 2):
        print(f"  {i}", end=" ")
    print()
    
    print("\n=== fetch_pages ===")
    all_items = []
    
    async for page_data in fetch_pages("https://api.example.com/items", 3):
        print(f"Page {page_data['page']}/{page_data['total_pages']}: "
              f"{len(page_data['items'])} items")
        all_items.extend(page_data['items'])
    
    print(f"\nTotal items: {len(all_items)}")
    
    # collect ทั้งหมดด้วย list comprehension
    # [item async for item in async_range(0, 5)]  # ทำงานได้ใน Python 3.6+
    items = [i async for i in async_range(0, 5)]
    print(f"\nAsync list comprehension: {items}")

asyncio.run(main())
```

### ตัวอย่างที่ 23: Async Iterator Protocol

```python
import asyncio
from typing import AsyncIterator

class AsyncCounter:
    """Async iterator ที่นับจาก 0 ถึง max"""
    
    def __init__(self, max_value: int, delay: float = 0.1):
        self.max_value = max_value
        self.delay = delay
        self._current = 0
    
    def __aiter__(self) -> 'AsyncCounter':
        return self
    
    async def __anext__(self) -> int:
        if self._current >= self.max_value:
            raise StopAsyncIteration
        
        await asyncio.sleep(self.delay)
        value = self._current
        self._current += 1
        return value

async def main() -> None:
    counter = AsyncCounter(5, delay=0.2)
    
    async for value in counter:
        print(f"Count: {value}")
    
    # async comprehension
    squares = [value ** 2 async for value in AsyncCounter(5, delay=0.1)]
    print(f"\nSquares: {squares}")

asyncio.run(main())
```

---

## 15. asyncio.timeout() (Python 3.11+)

### ตัวอย่างที่ 24: asyncio.timeout() และ wait_for()

```python
import asyncio
import sys

async def slow_operation(duration: float) -> str:
    """Operation ที่ช้า"""
    await asyncio.sleep(duration)
    return f"Completed after {duration}s"

async def main() -> None:
    # วิธีที่ 1: asyncio.wait_for() - ทำงานได้ทุก version
    print("=== wait_for() ===")
    try:
        result = await asyncio.wait_for(
            slow_operation(2.0),
            timeout=1.0
        )
        print(f"Result: {result}")
    except asyncio.TimeoutError:
        print("Timeout after 1s!")
    
    # วิธีที่ 2: asyncio.timeout() - Python 3.11+
    if sys.version_info >= (3, 11):
        print("\n=== asyncio.timeout() ===")
        try:
            async with asyncio.timeout(1.0):
                result = await slow_operation(2.0)
                print(f"Result: {result}")
        except asyncio.TimeoutError:
            print("Timeout after 1s!")
        
        # timeout ที่ปรับได้
        print("\n=== adjustable timeout ===")
        async with asyncio.timeout(5.0) as cm:
            print("เริ่มทำงาน...")
            cm.reschedule(asyncio.get_event_loop().time() + 0.5)  # เปลี่ยน timeout
            try:
                result = await slow_operation(2.0)
                print(f"Result: {result}")
            except asyncio.TimeoutError:
                print("Timeout หลังจาก reschedule!")
    
    # Pattern: retry กับ timeout
    print("\n=== retry with timeout ===")
    for attempt in range(3):
        try:
            result = await asyncio.wait_for(
                slow_operation(0.5 if attempt == 2 else 2.0),
                timeout=1.0
            )
            print(f"Success on attempt {attempt + 1}: {result}")
            break
        except asyncio.TimeoutError:
            print(f"Attempt {attempt + 1} timed out, retrying...")
    else:
        print("All attempts failed!")

asyncio.run(main())
```

---

## 16. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 25: Async Web Scraper

```python
import asyncio
import time
import re
from urllib.request import urlopen
from urllib.error import URLError
from typing import Optional
import urllib.request

class AsyncWebScraper:
    """Async web scraper ด้วย urllib"""
    
    def __init__(
        self,
        max_concurrent: int = 10,
        timeout: float = 10.0,
        delay: float = 0.1
    ):
        self.max_concurrent = max_concurrent
        self.timeout = timeout
        self.delay = delay
        self._semaphore = asyncio.Semaphore(max_concurrent)
    
    async def fetch_url(self, url: str) -> Optional[dict]:
        """ดึงเนื้อหาจาก URL"""
        async with self._semaphore:
            try:
                # รัน blocking I/O ใน thread pool
                result = await asyncio.wait_for(
                    asyncio.to_thread(self._sync_fetch, url),
                    timeout=self.timeout
                )
                
                if self.delay > 0:
                    await asyncio.sleep(self.delay)
                
                return result
                
            except asyncio.TimeoutError:
                return {'url': url, 'status': 'timeout', 'content': None}
            except Exception as e:
                return {'url': url, 'status': 'error', 'error': str(e)}
    
    def _sync_fetch(self, url: str) -> dict:
        """Synchronous fetch (รันใน thread)"""
        try:
            req = urllib.request.Request(
                url,
                headers={'User-Agent': 'AsyncBot/1.0'}
            )
            with urllib.request.urlopen(req, timeout=5) as response:
                content = response.read().decode('utf-8', errors='ignore')
                
                # ดึง title
                title_match = re.search(r'<title[^>]*>(.*?)</title>', content, re.IGNORECASE | re.DOTALL)
                title = title_match.group(1).strip() if title_match else "No title"
                
                return {
                    'url': url,
                    'status': 'ok',
                    'status_code': response.status,
                    'title': title[:100],
                    'size': len(content)
                }
        except URLError as e:
            return {'url': url, 'status': 'error', 'error': str(e)}
    
    async def scrape(self, urls: list[str]) -> list[dict]:
        """Scrape หลาย URLs"""
        print(f"Scraping {len(urls)} URLs...")
        start = time.time()
        
        results = await asyncio.gather(
            *[self.fetch_url(url) for url in urls],
            return_exceptions=False
        )
        
        elapsed = time.time() - start
        successful = sum(1 for r in results if r and r.get('status') == 'ok')
        
        print(f"เสร็จใน {elapsed:.2f}s: {successful}/{len(urls)} สำเร็จ")
        return results

async def main() -> None:
    scraper = AsyncWebScraper(max_concurrent=5, timeout=5.0)
    
    urls = [
        "https://example.com",
        "https://httpbin.org/html",
        "https://httpbin.org/html",
    ]
    
    results = await scraper.scrape(urls)
    
    for result in results:
        if result and result.get('status') == 'ok':
            print(f"  ✓ {result['url'][:50]}")
            print(f"    Title: {result.get('title', 'N/A')}")
            print(f"    Size: {result.get('size', 0):,} bytes")
        else:
            print(f"  ✗ {result.get('url', 'unknown')}: {result.get('status', 'error')}")

asyncio.run(main())
```

### ตัวอย่างที่ 26: Async API Client

```python
import asyncio
import json
import time
from dataclasses import dataclass
from typing import Any, Optional

@dataclass
class APIResponse:
    status: int
    data: Any
    elapsed: float
    cached: bool = False

class AsyncAPIClient:
    """Async HTTP client พร้อม caching และ retry"""
    
    def __init__(
        self,
        base_url: str,
        max_retries: int = 3,
        retry_delay: float = 1.0,
        cache_ttl: float = 60.0
    ):
        self.base_url = base_url
        self.max_retries = max_retries
        self.retry_delay = retry_delay
        self._cache: dict = {}
        self._cache_times: dict = {}
        self.cache_ttl = cache_ttl
        self._request_count = 0
    
    async def _fetch(self, endpoint: str) -> dict:
        """จำลอง HTTP request"""
        await asyncio.sleep(0.2)  # จำลอง network latency
        
        # จำลอง error บางครั้ง
        import random
        if random.random() < 0.1:  # 10% error rate
            raise ConnectionError(f"Network error for {endpoint}")
        
        return {
            'endpoint': endpoint,
            'data': f"Response from {endpoint}",
            'timestamp': time.time()
        }
    
    def _is_cached(self, key: str) -> bool:
        """ตรวจสอบว่า cache ยังไม่หมดอายุ"""
        if key not in self._cache:
            return False
        age = time.time() - self._cache_times[key]
        return age < self.cache_ttl
    
    async def get(self, endpoint: str, use_cache: bool = True) -> APIResponse:
        """GET request พร้อม caching"""
        cache_key = f"GET:{endpoint}"
        
        # ตรวจสอบ cache
        if use_cache and self._is_cached(cache_key):
            return APIResponse(
                status=200,
                data=self._cache[cache_key],
                elapsed=0,
                cached=True
            )
        
        # ส่ง request พร้อม retry
        start = time.time()
        last_error = None
        
        for attempt in range(self.max_retries):
            try:
                self._request_count += 1
                data = await self._fetch(f"{self.base_url}{endpoint}")
                elapsed = time.time() - start
                
                # เก็บใน cache
                if use_cache:
                    self._cache[cache_key] = data
                    self._cache_times[cache_key] = time.time()
                
                return APIResponse(status=200, data=data, elapsed=elapsed)
                
            except ConnectionError as e:
                last_error = e
                if attempt < self.max_retries - 1:
                    wait_time = self.retry_delay * (2 ** attempt)
                    print(f"Attempt {attempt+1} failed, retrying in {wait_time}s...")
                    await asyncio.sleep(wait_time)
        
        return APIResponse(
            status=500,
            data={'error': str(last_error)},
            elapsed=time.time() - start
        )
    
    def get_stats(self) -> dict:
        return {
            'total_requests': self._request_count,
            'cached_items': len(self._cache)
        }

async def main() -> None:
    client = AsyncAPIClient(
        base_url="https://api.example.com",
        max_retries=3,
        cache_ttl=5.0
    )
    
    endpoints = [
        "/users/1",
        "/users/2",
        "/products/1",
        "/users/1",  # cache hit
        "/orders/recent",
    ]
    
    print("ส่ง requests...")
    start = time.time()
    
    results = await asyncio.gather(*[
        client.get(ep)
        for ep in endpoints
    ])
    
    elapsed = time.time() - start
    
    for ep, result in zip(endpoints, results):
        cached = "CACHED" if result.cached else f"{result.elapsed:.3f}s"
        status = "✓" if result.status == 200 else "✗"
        print(f"  {status} {ep}: {cached}")
    
    print(f"\nTotal time: {elapsed:.2f}s")
    print(f"Stats: {client.get_stats()}")

asyncio.run(main())
```

### ตัวอย่างที่ 27: Chat Server (WebSocket จำลอง)

```python
import asyncio
import json
import time
from typing import Set

class ChatServer:
    """Async chat server"""
    
    def __init__(self):
        self.clients: Set[asyncio.Queue] = set()
        self.messages: list = []
        self._lock = asyncio.Lock()
    
    async def join(self, client_id: str) -> asyncio.Queue:
        """Client เข้าร่วม chat"""
        queue = asyncio.Queue()
        async with self._lock:
            self.clients.add(queue)
        
        await self.broadcast({
            'type': 'system',
            'message': f"{client_id} เข้าร่วม chat",
            'timestamp': time.time()
        }, exclude=queue)
        
        return queue
    
    async def leave(self, client_id: str, queue: asyncio.Queue) -> None:
        """Client ออกจาก chat"""
        async with self._lock:
            self.clients.discard(queue)
        
        await self.broadcast({
            'type': 'system',
            'message': f"{client_id} ออกจาก chat",
            'timestamp': time.time()
        })
    
    async def broadcast(
        self,
        message: dict,
        exclude: asyncio.Queue | None = None
    ) -> None:
        """ส่ง message ไปยัง clients ทั้งหมด"""
        async with self._lock:
            clients = self.clients.copy()
        
        for client_queue in clients:
            if client_queue is not exclude:
                await client_queue.put(message)
    
    async def send_message(self, sender: str, text: str) -> None:
        """ส่ง chat message"""
        message = {
            'type': 'message',
            'sender': sender,
            'text': text,
            'timestamp': time.time()
        }
        self.messages.append(message)
        await self.broadcast(message)

async def chat_client(
    server: ChatServer,
    name: str,
    messages_to_send: list[str]
) -> None:
    """Client จำลอง"""
    queue = await server.join(name)
    
    # รับและแสดง messages ใน background
    async def receive_messages() -> None:
        while True:
            msg = await queue.get()
            if msg.get('type') == 'system':
                print(f"  [System] {msg['message']}")
            else:
                print(f"  [{msg['sender']}]: {msg['text']}")
    
    receiver = asyncio.create_task(receive_messages())
    
    # ส่ง messages
    for text in messages_to_send:
        await asyncio.sleep(0.3)
        await server.send_message(name, text)
    
    # ออกจาก chat
    await asyncio.sleep(0.5)
    await server.leave(name, queue)
    receiver.cancel()

async def main() -> None:
    server = ChatServer()
    
    print("=== Async Chat Demo ===")
    
    await asyncio.gather(
        chat_client(server, "Alice", ["สวัสดี!", "ทุกคนสบายดีมั้ย?", "ลาก่อน!"]),
        chat_client(server, "Bob", ["Hi Alice!", "ดีครับ", "bye!"])
    )
    
    print(f"\nMessages history: {len(server.messages)} messages")

asyncio.run(main())
```

### ตัวอย่างที่ 28: Async Task Scheduler

```python
import asyncio
import time
from dataclasses import dataclass, field
from typing import Callable, Coroutine, Any
from heapq import heappush, heappop

@dataclass(order=True)
class ScheduledTask:
    run_at: float
    task_id: int
    name: str = field(compare=False)
    coro_factory: Callable = field(compare=False)

class AsyncTaskScheduler:
    """Async task scheduler ที่รัน tasks ตามเวลาที่กำหนด"""
    
    def __init__(self):
        self._tasks: list = []
        self._task_id = 0
        self._running = True
    
    def schedule(
        self,
        coro_factory: Callable,
        delay: float = 0,
        name: str = ""
    ) -> int:
        """Schedule task ให้รันหลังจาก delay วินาที"""
        task_id = self._task_id
        self._task_id += 1
        
        run_at = time.monotonic() + delay
        task = ScheduledTask(run_at, task_id, name or f"task_{task_id}", coro_factory)
        heappush(self._tasks, task)
        
        return task_id
    
    async def run(self) -> None:
        """รัน scheduler loop"""
        while self._running or self._tasks:
            if not self._tasks:
                await asyncio.sleep(0.1)
                continue
            
            now = time.monotonic()
            next_run = self._tasks[0].run_at
            
            if next_run > now:
                await asyncio.sleep(min(next_run - now, 0.1))
                continue
            
            task = heappop(self._tasks)
            asyncio.create_task(
                task.coro_factory(),
                name=task.name
            )
    
    def stop(self) -> None:
        self._running = False

async def main() -> None:
    scheduler = AsyncTaskScheduler()
    results = []
    
    async def timed_task(task_name: str, delay: float) -> None:
        start = time.monotonic()
        await asyncio.sleep(0.1)
        actual = time.monotonic() - start
        results.append(f"{task_name} (expected: {delay}s, actual: {actual:.1f}s)")
        print(f"Ran: {task_name}")
    
    # Schedule tasks
    scheduler.schedule(lambda: timed_task("immediate", 0), delay=0)
    scheduler.schedule(lambda: timed_task("0.3s delay", 0.3), delay=0.3)
    scheduler.schedule(lambda: timed_task("0.6s delay", 0.6), delay=0.6)
    scheduler.schedule(lambda: timed_task("1.0s delay", 1.0), delay=1.0)
    
    print("Starting scheduler...")
    
    # รัน scheduler
    scheduler_task = asyncio.create_task(scheduler.run())
    
    # รอ 1.5 วินาทีให้ tasks ทำงาน
    await asyncio.sleep(1.5)
    
    scheduler.stop()
    scheduler_task.cancel()
    
    print(f"\nCompleted {len(results)} tasks")
    for r in results:
        print(f"  {r}")

asyncio.run(main())
```

### ตัวอย่างที่ 29: Async Connection Pool

```python
import asyncio
from dataclasses import dataclass
from typing import Optional
import time

@dataclass
class Connection:
    id: int
    created_at: float
    _in_use: bool = False
    
    @property
    def in_use(self) -> bool:
        return self._in_use
    
    async def execute(self, query: str) -> dict:
        """จำลองการ execute query"""
        await asyncio.sleep(0.05)  # จำลอง DB latency
        return {'query': query, 'rows': 10, 'connection_id': self.id}

class AsyncConnectionPool:
    """Async connection pool"""
    
    def __init__(self, pool_size: int = 5, max_wait: float = 10.0):
        self.pool_size = pool_size
        self.max_wait = max_wait
        self._connections: list[Connection] = []
        self._available: asyncio.Queue = asyncio.Queue()
        self._lock = asyncio.Lock()
        self._initialized = False
    
    async def initialize(self) -> None:
        """สร้าง connections"""
        async with self._lock:
            if self._initialized:
                return
            
            for i in range(self.pool_size):
                await asyncio.sleep(0.01)  # จำลองการสร้าง connection
                conn = Connection(id=i, created_at=time.time())
                self._connections.append(conn)
                await self._available.put(conn)
            
            self._initialized = True
            print(f"Pool initialized with {self.pool_size} connections")
    
    async def acquire(self) -> Connection:
        """ขอ connection จาก pool"""
        try:
            conn = await asyncio.wait_for(
                self._available.get(),
                timeout=self.max_wait
            )
            conn._in_use = True
            return conn
        except asyncio.TimeoutError:
            raise TimeoutError(f"ไม่มี connection ว่างใน {self.max_wait}s")
    
    async def release(self, conn: Connection) -> None:
        """คืน connection ให้ pool"""
        conn._in_use = False
        await self._available.put(conn)
    
    async def execute(self, query: str) -> dict:
        """Execute query โดยจัดการ connection อัตโนมัติ"""
        conn = await self.acquire()
        try:
            return await conn.execute(query)
        finally:
            await self.release(conn)
    
    def stats(self) -> dict:
        in_use = sum(1 for c in self._connections if c.in_use)
        return {
            'total': len(self._connections),
            'in_use': in_use,
            'available': len(self._connections) - in_use
        }

async def main() -> None:
    pool = AsyncConnectionPool(pool_size=3)
    await pool.initialize()
    
    # ส่ง queries มากกว่า pool size
    queries = [f"SELECT * FROM table_{i}" for i in range(10)]
    
    async def run_query(query: str) -> dict:
        result = await pool.execute(query)
        print(f"Query '{query[:30]}' completed on conn {result['connection_id']}")
        return result
    
    start = time.time()
    results = await asyncio.gather(*[run_query(q) for q in queries])
    elapsed = time.time() - start
    
    print(f"\nExecuted {len(results)} queries in {elapsed:.2f}s")
    print(f"Pool stats: {pool.stats()}")

asyncio.run(main())
```

### ตัวอย่างที่ 30: Async Retry Decorator

```python
import asyncio
import functools
import random
from typing import TypeVar, Callable, Coroutine, Any

T = TypeVar('T')

def async_retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    backoff: float = 2.0,
    exceptions: tuple = (Exception,)
):
    """Decorator สำหรับ retry async function"""
    def decorator(func: Callable[..., Coroutine]) -> Callable:
        @functools.wraps(func)
        async def wrapper(*args, **kwargs) -> Any:
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    
                    if attempt < max_attempts - 1:
                        wait_time = delay * (backoff ** attempt)
                        print(f"Attempt {attempt + 1} failed: {e}")
                        print(f"Retrying in {wait_time:.1f}s...")
                        await asyncio.sleep(wait_time)
                    else:
                        print(f"All {max_attempts} attempts failed")
            
            raise last_exception
        
        return wrapper
    return decorator

@async_retry(max_attempts=3, delay=0.5, backoff=2.0, exceptions=(ConnectionError,))
async def unreliable_api_call(endpoint: str) -> dict:
    """API call ที่ไม่น่าเชื่อถือ"""
    if random.random() < 0.6:  # 60% fail rate
        raise ConnectionError(f"Connection failed for {endpoint}")
    
    await asyncio.sleep(0.1)
    return {'endpoint': endpoint, 'status': 'success'}

async def main() -> None:
    results = []
    
    for i in range(5):
        try:
            result = await unreliable_api_call(f"/api/item/{i}")
            results.append(result)
            print(f"✓ Success: {result}")
        except ConnectionError as e:
            print(f"✗ Failed after retries: {e}")
    
    print(f"\nSuccessful: {len(results)}/5")

asyncio.run(main())
```

### ตัวอย่างที่ 31: asyncio.to_thread สำหรับ Blocking Code

```python
import asyncio
import time
import hashlib
import os

def cpu_intensive_hash(data: bytes, iterations: int = 100000) -> str:
    """CPU-intensive hashing (blocking)"""
    result = data
    for _ in range(iterations):
        result = hashlib.sha256(result).digest()
    return result.hex()

def read_large_file(filename: str) -> bytes:
    """อ่านไฟล์ขนาดใหญ่ (blocking I/O)"""
    with open(filename, 'rb') as f:
        return f.read()

async def process_data_async(data: bytes, name: str) -> dict:
    """ประมวลผลข้อมูลแบบ async โดยใช้ to_thread สำหรับ blocking ops"""
    print(f"{name}: เริ่มประมวลผล")
    start = time.time()
    
    # รัน CPU-intensive task ใน thread pool
    hash_result = await asyncio.to_thread(cpu_intensive_hash, data, 50000)
    
    elapsed = time.time() - start
    print(f"{name}: เสร็จใน {elapsed:.2f}s")
    
    return {
        'name': name,
        'data_size': len(data),
        'hash': hash_result[:16] + "...",
        'elapsed': elapsed
    }

async def main() -> None:
    # สร้างข้อมูลทดสอบ
    datasets = [
        (b"Dataset A: " + b"x" * 1000, "Dataset A"),
        (b"Dataset B: " + b"y" * 1000, "Dataset B"),
        (b"Dataset C: " + b"z" * 1000, "Dataset C"),
    ]
    
    print("=== Sequential (blocking) ===")
    start = time.time()
    for data, name in datasets:
        hash_val = cpu_intensive_hash(data, 50000)
        print(f"{name}: {hash_val[:16]}...")
    seq_time = time.time() - start
    print(f"Sequential: {seq_time:.2f}s\n")
    
    print("=== Async (to_thread) ===")
    start = time.time()
    results = await asyncio.gather(*[
        process_data_async(data, name)
        for data, name in datasets
    ])
    async_time = time.time() - start
    print(f"\nAsync: {async_time:.2f}s")
    print(f"Speedup: {seq_time/async_time:.2f}x")

asyncio.run(main())
```

### ตัวอย่างที่ 32: Async Event System

```python
import asyncio
from typing import Callable, Coroutine, Any

class AsyncEventEmitter:
    """Async event emitter"""
    
    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}
        self._once_listeners: dict[str, list[Callable]] = {}
    
    def on(self, event: str, listener: Callable) -> None:
        """ลงทะเบียน listener"""
        self._listeners.setdefault(event, []).append(listener)
    
    def once(self, event: str, listener: Callable) -> None:
        """Listener ที่รันแค่ครั้งเดียว"""
        self._once_listeners.setdefault(event, []).append(listener)
    
    def off(self, event: str, listener: Callable) -> None:
        """ลบ listener"""
        if event in self._listeners:
            self._listeners[event] = [
                l for l in self._listeners[event] if l != listener
            ]
    
    async def emit(self, event: str, *args, **kwargs) -> None:
        """Emit event ไปยัง listeners"""
        listeners = self._listeners.get(event, []).copy()
        once_listeners = self._once_listeners.pop(event, [])
        
        all_listeners = listeners + once_listeners
        
        if not all_listeners:
            return
        
        # รัน listeners พร้อมกัน
        tasks = []
        for listener in all_listeners:
            if asyncio.iscoroutinefunction(listener):
                tasks.append(asyncio.create_task(listener(*args, **kwargs)))
            else:
                listener(*args, **kwargs)
        
        if tasks:
            await asyncio.gather(*tasks)

async def main() -> None:
    emitter = AsyncEventEmitter()
    
    # ลงทะเบียน listeners
    async def on_user_login(user_id: int, ip: str) -> None:
        await asyncio.sleep(0.1)
        print(f"  [Auth] User {user_id} logged in from {ip}")
    
    async def on_user_login_analytics(user_id: int, ip: str) -> None:
        await asyncio.sleep(0.05)
        print(f"  [Analytics] Track login: user={user_id}")
    
    def on_user_login_sync(user_id: int, ip: str) -> None:
        print(f"  [Cache] Warm cache for user {user_id}")
    
    emitter.on('user.login', on_user_login)
    emitter.on('user.login', on_user_login_analytics)
    emitter.on('user.login', on_user_login_sync)
    
    # once listener
    emitter.once('user.login', lambda uid, ip: print(f"  [Welcome] First login event!"))
    
    print("Emitting user.login events:")
    await emitter.emit('user.login', 42, '192.168.1.1')
    print("Second login (once listener should not fire):")
    await emitter.emit('user.login', 43, '10.0.0.1')

asyncio.run(main())
```

### ตัวอย่างที่ 33: Async Pipeline

```python
import asyncio
from typing import AsyncGenerator

async def generate_numbers(n: int) -> AsyncGenerator[int, None]:
    """สร้างตัวเลข"""
    for i in range(n):
        await asyncio.sleep(0.05)
        yield i

async def filter_even(
    source: AsyncGenerator[int, None]
) -> AsyncGenerator[int, None]:
    """กรองเฉพาะเลขคู่"""
    async for num in source:
        if num % 2 == 0:
            yield num

async def square(
    source: AsyncGenerator[int, None]
) -> AsyncGenerator[int, None]:
    """ยกกำลัง 2"""
    async for num in source:
        await asyncio.sleep(0.01)
        yield num ** 2

async def batch(
    source: AsyncGenerator[int, None],
    size: int
) -> AsyncGenerator[list[int], None]:
    """รวมเป็น batches"""
    current_batch = []
    async for item in source:
        current_batch.append(item)
        if len(current_batch) >= size:
            yield current_batch
            current_batch = []
    if current_batch:
        yield current_batch

async def main() -> None:
    # สร้าง pipeline
    numbers = generate_numbers(20)
    evens = filter_even(numbers)
    squares = square(evens)
    batches = batch(squares, size=3)
    
    print("Async Pipeline: numbers → filter_even → square → batch")
    
    async for batch_data in batches:
        print(f"  Batch: {batch_data}")

asyncio.run(main())
```

### ตัวอย่างที่ 34: Async Semaphore กับ HTTP Client Pool

```python
import asyncio
import time
import random
from dataclasses import dataclass
from typing import Optional

@dataclass
class MockResponse:
    status: int
    url: str
    latency: float
    data: dict

class AsyncHTTPPool:
    """จำลอง async HTTP connection pool"""
    
    def __init__(self, max_connections: int = 10):
        self._semaphore = asyncio.Semaphore(max_connections)
        self._request_count = 0
        self._errors = 0
    
    async def get(
        self,
        url: str,
        timeout: float = 5.0
    ) -> Optional[MockResponse]:
        """HTTP GET request"""
        async with self._semaphore:
            self._request_count += 1
            
            try:
                # จำลอง request
                latency = random.uniform(0.05, 0.5)
                await asyncio.wait_for(
                    asyncio.sleep(latency),
                    timeout=timeout
                )
                
                # จำลอง occasional errors
                if random.random() < 0.1:
                    raise ConnectionError(f"Failed to connect to {url}")
                
                return MockResponse(
                    status=200,
                    url=url,
                    latency=latency,
                    data={'url': url, 'content_length': random.randint(100, 10000)}
                )
                
            except asyncio.TimeoutError:
                self._errors += 1
                return MockResponse(status=408, url=url, latency=timeout, data={})
            except ConnectionError:
                self._errors += 1
                return MockResponse(status=503, url=url, latency=0, data={})
    
    def stats(self) -> dict:
        return {
            'total_requests': self._request_count,
            'errors': self._errors,
            'success_rate': f"{(self._request_count - self._errors) / max(1, self._request_count) * 100:.1f}%"
        }

async def main() -> None:
    pool = AsyncHTTPPool(max_connections=5)
    
    # สร้าง URLs
    urls = [f"https://api.example.com/resource/{i}" for i in range(30)]
    
    start = time.time()
    
    responses = await asyncio.gather(*[
        pool.get(url)
        for url in urls
    ])
    
    elapsed = time.time() - start
    
    successful = [r for r in responses if r and r.status == 200]
    
    print(f"ส่ง {len(urls)} requests ใน {elapsed:.2f}s")
    print(f"สำเร็จ: {len(successful)}/{len(urls)}")
    print(f"Stats: {pool.stats()}")
    
    if successful:
        avg_latency = sum(r.latency for r in successful) / len(successful)
        print(f"Average latency: {avg_latency:.3f}s")

asyncio.run(main())
```

### ตัวอย่างที่ 35: Complete Async Application

```python
import asyncio
import time
import json
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Task:
    id: str
    type: str
    data: dict
    priority: int = 1
    retries: int = 0
    max_retries: int = 3

@dataclass
class TaskResult:
    task_id: str
    success: bool
    result: Optional[dict] = None
    error: Optional[str] = None
    duration: float = 0.0

class AsyncTaskProcessor:
    """Complete async task processing system"""
    
    def __init__(self, workers: int = 5):
        self.workers = workers
        self._queue: asyncio.PriorityQueue = asyncio.PriorityQueue()
        self._results: list[TaskResult] = []
        self._stats = {'processed': 0, 'failed': 0, 'retried': 0}
        self._running = True
    
    async def submit(self, task: Task) -> None:
        """ส่ง task เข้าระบบ"""
        await self._queue.put((task.priority, time.monotonic(), task))
    
    async def _process_task(self, task: Task) -> TaskResult:
        """ประมวลผล task"""
        start = time.time()
        
        try:
            # จำลองการประมวลผลตาม type
            if task.type == 'email':
                await asyncio.sleep(0.2)
                result = {'sent': True, 'recipients': task.data.get('to', [])}
            elif task.type == 'report':
                await asyncio.sleep(0.5)
                result = {'pages': 10, 'format': 'PDF'}
            elif task.type == 'sync':
                await asyncio.sleep(0.3)
                result = {'records_synced': 100}
            else:
                result = {'processed': True}
            
            return TaskResult(
                task_id=task.id,
                success=True,
                result=result,
                duration=time.time() - start
            )
        except Exception as e:
            return TaskResult(
                task_id=task.id,
                success=False,
                error=str(e),
                duration=time.time() - start
            )
    
    async def _worker(self, worker_id: int) -> None:
        """Worker loop"""
        while self._running:
            try:
                _, _, task = await asyncio.wait_for(
                    self._queue.get(),
                    timeout=0.5
                )
                
                result = await self._process_task(task)
                
                if not result.success and task.retries < task.max_retries:
                    task.retries += 1
                    self._stats['retried'] += 1
                    print(f"Worker {worker_id}: Retrying {task.id} ({task.retries})")
                    await self._queue.put((task.priority - 1, time.monotonic(), task))
                else:
                    self._results.append(result)
                    if result.success:
                        self._stats['processed'] += 1
                    else:
                        self._stats['failed'] += 1
                    
                    status = "✓" if result.success else "✗"
                    print(f"Worker {worker_id}: {status} {task.id} ({result.duration:.2f}s)")
                
                self._queue.task_done()
                
            except asyncio.TimeoutError:
                continue
    
    async def run(self, tasks: list[Task]) -> list[TaskResult]:
        """รัน processor"""
        # ส่ง tasks
        for task in tasks:
            await self.submit(task)
        
        # สร้าง workers
        worker_tasks = [
            asyncio.create_task(self._worker(i))
            for i in range(self.workers)
        ]
        
        # รอจนกว่า queue จะว่าง
        await self._queue.join()
        
        # หยุด workers
        self._running = False
        for wt in worker_tasks:
            wt.cancel()
        await asyncio.gather(*worker_tasks, return_exceptions=True)
        
        return self._results
    
    def summary(self) -> dict:
        return {
            **self._stats,
            'total': len(self._results),
            'success_rate': f"{self._stats['processed'] / max(1, len(self._results)) * 100:.1f}%"
        }

async def main() -> None:
    processor = AsyncTaskProcessor(workers=3)
    
    tasks = [
        Task(f"T{i:03d}", 
             ['email', 'report', 'sync'][i % 3],
             {'data': f'item_{i}'},
             priority=i % 3 + 1)
        for i in range(12)
    ]
    
    print(f"ประมวลผล {len(tasks)} tasks ด้วย {processor.workers} workers\n")
    
    start = time.time()
    results = await processor.run(tasks)
    elapsed = time.time() - start
    
    print(f"\n=== Summary ===")
    print(f"Time: {elapsed:.2f}s")
    print(f"Stats: {processor.summary()}")

asyncio.run(main())
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Async Rate Limiter

**โจทย์:** สร้าง rate limiter ที่จำกัด N requests ต่อ second

**เฉลย:**

```python
import asyncio
import time
from collections import deque

class AsyncRateLimiter:
    """Token bucket rate limiter"""
    
    def __init__(self, rate: float, burst: int = 1):
        self.rate = rate
        self.burst = burst
        self._tokens = burst
        self._last_update = time.monotonic()
        self._lock = asyncio.Lock()
    
    async def acquire(self, tokens: int = 1) -> float:
        """รอจนได้ token แล้ว return wait time"""
        async with self._lock:
            now = time.monotonic()
            
            # เพิ่ม tokens ตามเวลาที่ผ่านไป
            elapsed = now - self._last_update
            self._tokens = min(self.burst, self._tokens + elapsed * self.rate)
            self._last_update = now
            
            if self._tokens >= tokens:
                self._tokens -= tokens
                return 0
            
            # รอให้ได้ tokens
            wait_time = (tokens - self._tokens) / self.rate
            self._tokens = 0
            self._last_update += wait_time
            
            await asyncio.sleep(wait_time)
            return wait_time

async def make_request(
    url: str,
    limiter: AsyncRateLimiter,
    request_id: int
) -> dict:
    wait = await limiter.acquire()
    print(f"Request {request_id}: {url} (waited {wait:.2f}s)")
    await asyncio.sleep(0.05)  # จำลอง request
    return {'id': request_id, 'url': url, 'wait': wait}

async def main() -> None:
    limiter = AsyncRateLimiter(rate=3.0, burst=3)  # 3 req/s, burst 3
    
    start = time.time()
    results = await asyncio.gather(*[
        make_request(f"https://api.example.com/{i}", limiter, i)
        for i in range(10)
    ])
    elapsed = time.time() - start
    
    print(f"\nทั้งหมด {len(results)} requests ใน {elapsed:.2f}s")
    print(f"Rate: {len(results)/elapsed:.1f} req/s (จำกัดที่ 3 req/s)")

asyncio.run(main())
```

---

### แบบฝึกหัดที่ 2-8 (สรุปย่อ)

แบบฝึกหัดที่ 2: สร้าง async cache ด้วย TTL และ LRU eviction

แบบฝึกหัดที่ 3: สร้าง async circuit breaker ที่หยุด requests เมื่อ error rate สูงเกินไป

แบบฝึกหัดที่ 4: สร้าง async batch processor ที่รวม requests เป็น batches

แบบฝึกหัดที่ 5: สร้าง async health checker ที่ monitor endpoints หลายตัวพร้อมกัน

แบบฝึกหัดที่ 6: สร้าง async pub/sub system

แบบฝึกหัดที่ 7: สร้าง async data pipeline ที่มี backpressure

แบบฝึกหัดที่ 8: สร้าง complete async web crawler ที่มี depth limit และ URL deduplication

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Asynchronous Concepts** - event loop, coroutines, non-blocking I/O
2. **async/await Syntax** - การเขียน coroutine functions
3. **asyncio.run()** - entry point สำหรับ async programs
4. **create_task()** - schedule coroutines ให้รันพร้อมกัน
5. **gather() และ wait()** - รันและรอ multiple coroutines
6. **asyncio.Queue** - async-safe queue สำหรับ producer-consumer
7. **Streams** - async networking
8. **aiohttp/aiofiles** - third-party async libraries
9. **Async Context Managers** - `async with`
10. **Async Generators** - `async for` และ `yield`
11. **timeout** - จัดการ timeout

### เมื่อไรควรใช้ asyncio?

- **I/O-bound tasks**: HTTP requests, database queries, file I/O
- **High concurrency**: thousands of concurrent connections
- **Real-time systems**: chat servers, notifications, streaming

### asyncio vs Threading

- asyncio ดีกว่าสำหรับ I/O-bound ที่ต้องการ concurrent connections จำนวนมาก
- Threading ง่ายกว่าสำหรับโค้ด blocking ที่มีอยู่แล้ว
- asyncio มี overhead น้อยกว่า (coroutines เบากว่า threads)

---

*ถัดไป: Part 40 - Type Hints & mypy*
