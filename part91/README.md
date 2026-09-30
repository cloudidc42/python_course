# Part 91 - Redis: Caching & Data Structures

## บทนำ

Redis (Remote Dictionary Server) คือ in-memory data structure store ที่มีประสิทธิภาพสูง ใช้เป็น database, cache, message broker และ queue ได้ Redis รองรับ data structures หลากหลาย เช่น strings, hashes, lists, sets, sorted sets พร้อม range queries, bitmaps, hyperloglogs, geospatial indexes และ streams

### ทำไมต้องใช้ Redis?

- **ความเร็วสูงมาก**: ข้อมูลอยู่ใน RAM ทำให้ read/write latency อยู่ในระดับ microsecond
- **Data structures ที่หลากหลาย**: รองรับ data types มากกว่า plain key-value
- **Persistence**: สามารถ save ข้อมูลลง disk ได้ (RDB/AOF)
- **Replication**: รองรับ master-slave replication
- **Cluster**: รองรับ horizontal scaling ด้วย Redis Cluster
- **Pub/Sub**: built-in messaging system
- **Atomic operations**: operations ทุกอย่างเป็น atomic

---

## 1. การติดตั้งและ Setup

### ติดตั้ง Redis Server

```bash
# Ubuntu/Debian
sudo apt-get install redis-server

# macOS
brew install redis

# Docker
docker run -d -p 6379:6379 redis:latest

# เริ่มต้น Redis
redis-server

# ทดสอบด้วย CLI
redis-cli ping
# ผลลัพธ์: PONG
```

### ติดตั้ง redis-py

```bash
pip install redis
pip install redis[hiredis]  # เพื่อประสิทธิภาพที่ดีขึ้น
```

---

## 2. การเชื่อมต่อ Redis ด้วย Python

### ตัวอย่างที่ 1: การเชื่อมต่อพื้นฐาน

```python
import redis

# วิธีที่ 1: สร้าง connection ตรง ๆ ด้วย Redis class
r = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    decode_responses=True  # auto decode bytes to str
)

# ทดสอบการเชื่อมต่อ
try:
    r.ping()
    print("เชื่อมต่อ Redis สำเร็จ!")
except redis.ConnectionError as e:
    print(f"เชื่อมต่อไม่ได้: {e}")

# วิธีที่ 2: ใช้ StrictRedis (deprecated แล้ว แต่ยังใช้งานได้)
r_strict = redis.StrictRedis(
    host='localhost',
    port=6379,
    db=0,
    decode_responses=True
)

# วิธีที่ 3: ใช้ URL
r_url = redis.from_url(
    "redis://localhost:6379/0",
    decode_responses=True
)

# Redis ที่ต้องใช้ password
r_auth = redis.Redis(
    host='localhost',
    port=6379,
    password='secret_password',
    decode_responses=True
)

print("ทดสอบ SET/GET:")
r.set("hello", "world")
value = r.get("hello")
print(f"ค่าที่ได้: {value}")
```

### ตัวอย่างที่ 2: ConnectionPool

```python
import redis

# ConnectionPool ช่วยจัดการ connections อย่างมีประสิทธิภาพ
# แทนที่จะสร้าง connection ใหม่ทุกครั้ง จะ reuse connections เดิม

pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    db=0,
    max_connections=20,      # จำนวน connections สูงสุด
    decode_responses=True
)

# สร้าง Redis instance จาก pool
r = redis.Redis(connection_pool=pool)

# ทุก Redis operation จะใช้ connection จาก pool
r.set("key1", "value1")
print(r.get("key1"))

# ตรวจสอบสถานะ pool
print(f"Connections in pool: {pool._created_connections}")

# สำหรับ production ควรสร้าง pool เดียวและ share ทั่วทั้ง application
class RedisClient:
    _pool = None
    
    @classmethod
    def get_pool(cls):
        if cls._pool is None:
            cls._pool = redis.ConnectionPool(
                host='localhost',
                port=6379,
                db=0,
                max_connections=50,
                decode_responses=True,
                socket_timeout=5,
                socket_connect_timeout=5,
                retry_on_timeout=True
            )
        return cls._pool
    
    @classmethod
    def get_client(cls):
        return redis.Redis(connection_pool=cls.get_pool())

# ใช้งาน
client = RedisClient.get_client()
client.set("app:version", "1.0.0")
print(client.get("app:version"))
```

---

## 3. Redis Data Types

### 3.1 String - ประเภทข้อมูลพื้นฐาน

### ตัวอย่างที่ 3: String Operations

```python
import redis
import json
from datetime import timedelta

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# SET และ GET พื้นฐาน
r.set("name", "Alice")
print(r.get("name"))         # Alice

# SET พร้อม TTL (Time To Live)
r.set("session:123", "user_data", ex=3600)      # หมดอายุใน 1 ชั่วโมง
r.set("cache:key", "cached_value", px=5000)     # หมดอายุใน 5000 ms
r.set("temp:key", "value", exat=1700000000)     # หมดอายุ ณ Unix timestamp

# SETNX - Set if Not eXists (ใช้ทำ distributed lock อย่างง่าย)
result = r.setnx("lock:resource", "locked")
print(f"Lock acquired: {result}")  # True ถ้าสำเร็จ

# GETSET - ได้ค่าเก่า แล้วตั้งค่าใหม่ (atomic)
old_value = r.getset("counter", "0")
print(f"Old value: {old_value}")

# MSET/MGET - Set/Get หลายค่าพร้อมกัน
r.mset({
    "user:1:name": "Alice",
    "user:1:email": "alice@example.com",
    "user:1:age": "25"
})

values = r.mget("user:1:name", "user:1:email", "user:1:age")
print(values)  # ['Alice', 'alice@example.com', '25']

# Increment/Decrement
r.set("page:views", 0)
r.incr("page:views")       # เพิ่มทีละ 1
r.incrby("page:views", 5)  # เพิ่มทีละ 5
r.decr("page:views")       # ลดทีละ 1
r.decrby("page:views", 2)  # ลดทีละ 2
print(r.get("page:views"))  # '3'

# Float increment
r.set("price", 10.5)
r.incrbyfloat("price", 2.3)
print(r.get("price"))  # '12.8'

# เก็บ JSON เป็น string
user_data = {"id": 1, "name": "Bob", "role": "admin"}
r.set("user:1", json.dumps(user_data))
stored_user = json.loads(r.get("user:1"))
print(stored_user)

# APPEND
r.set("log", "2024-01-01: started")
r.append("log", "\n2024-01-02: running")
print(r.get("log"))

# String length
r.set("text", "Hello World")
print(r.strlen("text"))  # 11

# GETRANGE - ดึงส่วนของ string
print(r.getrange("text", 0, 4))   # Hello
print(r.getrange("text", 6, -1))  # World
```

### 3.2 Hash - เก็บ field-value pairs

### ตัวอย่างที่ 4: Hash Operations

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# HSET - เซ็ตหลาย field พร้อมกัน
r.hset("user:1000", mapping={
    "name": "Alice",
    "email": "alice@example.com",
    "age": "30",
    "city": "Bangkok"
})

# HGET - ดึง field เดียว
name = r.hget("user:1000", "name")
print(f"Name: {name}")  # Alice

# HMGET - ดึงหลาย fields
fields = r.hmget("user:1000", "name", "email", "age")
print(fields)  # ['Alice', 'alice@example.com', '30']

# HGETALL - ดึงทุก field
all_data = r.hgetall("user:1000")
print(all_data)  # dict ทั้งหมด

# HKEYS, HVALS, HLEN
print(r.hkeys("user:1000"))   # ['name', 'email', 'age', 'city']
print(r.hvals("user:1000"))   # ['Alice', 'alice@example.com', '30', 'Bangkok']
print(r.hlen("user:1000"))    # 4

# HEXISTS - ตรวจสอบว่า field มีอยู่หรือไม่
print(r.hexists("user:1000", "name"))   # True
print(r.hexists("user:1000", "phone"))  # False

# HDEL - ลบ field
r.hdel("user:1000", "city")

# HINCRBY - เพิ่มค่า numeric field
r.hset("user:1000", "login_count", 0)
r.hincrby("user:1000", "login_count", 1)
r.hincrbyfloat("user:1000", "balance", 100.50)

# HSETNX - Set field ถ้ายังไม่มี
r.hsetnx("user:1000", "created_at", "2024-01-01")

# ใช้ Hash เก็บข้อมูล Session
def store_session(session_id: str, user_data: dict, ttl: int = 3600):
    key = f"session:{session_id}"
    r.hset(key, mapping=user_data)
    r.expire(key, ttl)

def get_session(session_id: str) -> dict:
    key = f"session:{session_id}"
    return r.hgetall(key)

store_session("abc123", {"user_id": "1", "username": "alice", "role": "admin"})
print(get_session("abc123"))
```

### 3.3 List - Linked List

### ตัวอย่างที่ 5: List Operations

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# LPUSH/RPUSH - เพิ่มที่ต้น/ปลาย list
r.lpush("tasks", "task3", "task2", "task1")
r.rpush("queue", "item1", "item2", "item3")

# LRANGE - ดึงช่วงของ list
print(r.lrange("tasks", 0, -1))   # ดึงทั้งหมด
print(r.lrange("tasks", 0, 1))    # ดึง 2 ตัวแรก

# LPOP/RPOP - ดึงและลบที่ต้น/ปลาย
first = r.lpop("tasks")
last = r.rpop("tasks")
print(f"First: {first}, Last: {last}")

# LLEN - ความยาว list
print(r.llen("queue"))

# LINDEX - ดึงโดย index
print(r.lindex("queue", 0))   # item1 (ตัวแรก)
print(r.lindex("queue", -1))  # item3 (ตัวสุดท้าย)

# LSET - เซ็ตค่าโดย index
r.lset("queue", 0, "modified_item1")

# LINSERT - แทรก element ก่อน/หลัง pivot
r.linsert("queue", "BEFORE", "item2", "new_item")
r.linsert("queue", "AFTER", "item2", "after_item2")

# LREM - ลบ elements ที่ตรงกัน
r.lrem("queue", 1, "modified_item1")  # ลบ 1 ตัวที่พบจากซ้าย

# LTRIM - ตัด list ให้เหลือแค่ช่วงที่กำหนด
r.ltrim("queue", 0, 9)  # เก็บแค่ 10 ตัวแรก

# BLPOP/BRPOP - Blocking pop (รอจนกว่าจะมีข้อมูล)
# ใช้สำหรับทำ task queue
print("รอ task...")
result = r.blpop("job_queue", timeout=2)  # รอ 2 วินาที
if result:
    queue_name, task = result
    print(f"Got task from {queue_name}: {task}")
else:
    print("ไม่มี task ใน 2 วินาที")

# ใช้ List ทำ Simple Queue
class SimpleQueue:
    def __init__(self, redis_client, queue_name):
        self.r = redis_client
        self.name = queue_name
    
    def enqueue(self, item):
        self.r.rpush(self.name, item)
    
    def dequeue(self, timeout=0):
        result = self.r.blpop(self.name, timeout=timeout)
        if result:
            return result[1]
        return None
    
    def peek(self):
        return self.r.lindex(self.name, 0)
    
    def size(self):
        return self.r.llen(self.name)

queue = SimpleQueue(r, "email_queue")
queue.enqueue("send_welcome_email:user@example.com")
queue.enqueue("send_invoice:order123")
print(f"Queue size: {queue.size()}")
print(f"Next item: {queue.peek()}")
task = queue.dequeue(timeout=1)
print(f"Processing: {task}")
```

### 3.4 Set - Unordered Collection

### ตัวอย่างที่ 6: Set Operations

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# SADD - เพิ่ม members
r.sadd("tags:article1", "python", "redis", "database", "backend")
r.sadd("tags:article2", "python", "web", "fastapi", "backend")

# SMEMBERS - ดึงสมาชิกทั้งหมด
print(r.smembers("tags:article1"))

# SCARD - จำนวนสมาชิก
print(r.scard("tags:article1"))

# SISMEMBER - ตรวจสอบว่าเป็นสมาชิกหรือไม่
print(r.sismember("tags:article1", "python"))   # True
print(r.sismember("tags:article1", "java"))     # False

# SREM - ลบสมาชิก
r.srem("tags:article1", "database")

# Set Operations
# SUNION - รวม sets
common_tags = r.sunion("tags:article1", "tags:article2")
print(f"All tags: {common_tags}")

# SINTER - ตัดกัน (intersection)
shared_tags = r.sinter("tags:article1", "tags:article2")
print(f"Shared tags: {shared_tags}")

# SDIFF - ความแตกต่าง
unique_to_article1 = r.sdiff("tags:article1", "tags:article2")
print(f"Unique to article1: {unique_to_article1}")

# SUNIONSTORE/SINTERSTORE/SDIFFSTORE - เก็บผลลัพธ์ใน key ใหม่
r.sunionstore("tags:combined", "tags:article1", "tags:article2")
r.sinterstore("tags:common", "tags:article1", "tags:article2")

# SPOP - ดึงสมาชิกแบบ random และลบออก
random_tag = r.spop("tags:combined")
print(f"Random tag: {random_tag}")

# SRANDMEMBER - ดึงสมาชิกแบบ random โดยไม่ลบ
random_tags = r.srandmember("tags:article1", count=2)
print(f"Random 2 tags: {random_tags}")

# SMOVE - ย้าย member ระหว่าง sets
r.smove("tags:article1", "tags:article2", "redis")

# ใช้ Set สำหรับ Online Users tracking
class OnlineUserTracker:
    def __init__(self, redis_client):
        self.r = redis_client
        self.key = "online_users"
    
    def user_online(self, user_id: str):
        self.r.sadd(self.key, user_id)
        self.r.expire(self.key, 300)  # reset TTL ทุกครั้ง
    
    def user_offline(self, user_id: str):
        self.r.srem(self.key, user_id)
    
    def is_online(self, user_id: str) -> bool:
        return self.r.sismember(self.key, user_id)
    
    def get_online_count(self) -> int:
        return self.r.scard(self.key)
    
    def get_all_online(self) -> set:
        return self.r.smembers(self.key)

tracker = OnlineUserTracker(r)
tracker.user_online("user1")
tracker.user_online("user2")
tracker.user_online("user3")
print(f"Online users: {tracker.get_online_count()}")
print(f"User1 online: {tracker.is_online('user1')}")
tracker.user_offline("user2")
print(f"Still online: {tracker.get_all_online()}")
```

### 3.5 Sorted Set - Set พร้อม Score

### ตัวอย่างที่ 7: Sorted Set Operations

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# ZADD - เพิ่ม members พร้อม score
r.zadd("leaderboard", {
    "Alice": 1500,
    "Bob": 1200,
    "Charlie": 1800,
    "Diana": 1600,
    "Eve": 1100
})

# ZRANGE - ดึงตาม rank (ascending)
print("Bottom players:", r.zrange("leaderboard", 0, -1, withscores=True))

# ZREVRANGE - ดึงตาม rank (descending)
print("Top players:", r.zrevrange("leaderboard", 0, 2, withscores=True))

# ZRANK/ZREVRANK - ดู rank ของ member
print(f"Charlie rank (asc): {r.zrank('leaderboard', 'Charlie')}")   # 0 = ต่ำสุด
print(f"Charlie rank (desc): {r.zrevrank('leaderboard', 'Charlie')}")  # 0 = สูงสุด

# ZSCORE - ดู score
print(f"Alice score: {r.zscore('leaderboard', 'Alice')}")

# ZINCRBY - เพิ่ม score
r.zincrby("leaderboard", 200, "Bob")  # Bob +200
print(f"Bob new score: {r.zscore('leaderboard', 'Bob')}")

# ZRANGEBYSCORE - ดึงตาม score range
players_1200_1700 = r.zrangebyscore("leaderboard", 1200, 1700, withscores=True)
print(f"Players 1200-1700: {players_1200_1700}")

# ZCOUNT - นับ members ใน score range
count = r.zcount("leaderboard", 1200, 1700)
print(f"Count 1200-1700: {count}")

# ZREM - ลบ member
r.zrem("leaderboard", "Eve")

# ZPOPMIN/ZPOPMAX - ดึงและลบ lowest/highest score
lowest = r.zpopmin("leaderboard", count=1)
print(f"Removed lowest: {lowest}")

# ZCARD - จำนวน members ทั้งหมด
print(f"Total players: {r.zcard('leaderboard')}")

# Sorted Set สำหรับ Leaderboard
class GameLeaderboard:
    def __init__(self, redis_client, game_name: str):
        self.r = redis_client
        self.key = f"leaderboard:{game_name}"
    
    def submit_score(self, player: str, score: float):
        # เก็บ score สูงสุด
        current_score = self.r.zscore(self.key, player)
        if current_score is None or score > current_score:
            self.r.zadd(self.key, {player: score})
    
    def get_top_players(self, count: int = 10):
        return self.r.zrevrange(self.key, 0, count - 1, withscores=True)
    
    def get_player_rank(self, player: str):
        rank = self.r.zrevrank(self.key, player)
        if rank is not None:
            return rank + 1  # 1-indexed
        return None
    
    def get_player_score(self, player: str):
        return self.r.zscore(self.key, player)
    
    def get_players_around(self, player: str, count: int = 5):
        rank = self.r.zrevrank(self.key, player)
        if rank is None:
            return []
        start = max(0, rank - count // 2)
        end = start + count - 1
        return self.r.zrevrange(self.key, start, end, withscores=True)

lb = GameLeaderboard(r, "puzzle_game")
lb.submit_score("player1", 5000)
lb.submit_score("player2", 7500)
lb.submit_score("player3", 6000)
lb.submit_score("player1", 8000)  # update ถ้าสูงกว่า

print("Top 3:", lb.get_top_players(3))
print("player2 rank:", lb.get_player_rank("player2"))
print("Players near player2:", lb.get_players_around("player2"))
```

---

## 4. Caching Patterns

### 4.1 Cache-Aside Pattern (Lazy Loading)

### ตัวอย่างที่ 8: Cache-Aside

```python
import redis
import json
import time
from typing import Optional, Any, Callable

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# จำลอง database
fake_db = {
    1: {"id": 1, "name": "Alice", "email": "alice@example.com", "age": 30},
    2: {"id": 2, "name": "Bob", "email": "bob@example.com", "age": 25},
    3: {"id": 3, "name": "Charlie", "email": "charlie@example.com", "age": 35},
}

def get_user_from_db(user_id: int) -> Optional[dict]:
    """จำลองการดึงข้อมูลจาก database (ช้า)"""
    time.sleep(0.1)  # จำลอง latency
    return fake_db.get(user_id)

# Cache-Aside Pattern
def get_user(user_id: int, ttl: int = 300) -> Optional[dict]:
    """
    Cache-Aside: 
    1. ตรวจสอบ cache ก่อน
    2. ถ้าไม่มี ดึงจาก DB แล้ว cache ไว้
    """
    cache_key = f"user:{user_id}"
    
    # Step 1: ตรวจสอบ cache
    cached = r.get(cache_key)
    if cached:
        print(f"Cache HIT for user:{user_id}")
        return json.loads(cached)
    
    # Step 2: Cache miss - ดึงจาก database
    print(f"Cache MISS for user:{user_id} - querying DB...")
    user = get_user_from_db(user_id)
    
    if user:
        # Step 3: เก็บใน cache
        r.setex(cache_key, ttl, json.dumps(user))
        print(f"Cached user:{user_id} for {ttl}s")
    
    return user

# ทดสอบ
start = time.time()
user = get_user(1)
print(f"First call: {time.time()-start:.3f}s -> {user['name']}")

start = time.time()
user = get_user(1)  # ควรได้จาก cache
print(f"Second call: {time.time()-start:.3f}s -> {user['name']}")

# Invalidation หลัง update
def update_user(user_id: int, updates: dict):
    fake_db[user_id].update(updates)
    # ลบ cache key เพื่อให้ดึงข้อมูลใหม่ในครั้งถัดไป
    r.delete(f"user:{user_id}")
    print(f"Updated user:{user_id} and invalidated cache")

update_user(1, {"age": 31})
user = get_user(1)  # จะดึงจาก DB ใหม่
print(f"After update: age = {user['age']}")
```

### 4.2 Write-Through Pattern

### ตัวอย่างที่ 9: Write-Through Cache

```python
import redis
import json
from typing import Optional

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

fake_db = {}  # จำลอง database

def write_through_set(key: str, data: dict, ttl: int = 3600):
    """
    Write-Through: เขียน cache และ database พร้อมกัน
    ข้อดี: cache มีข้อมูลล่าสุดเสมอ
    ข้อเสีย: write latency สูงขึ้น
    """
    # 1. เขียน database ก่อน
    fake_db[key] = data.copy()
    print(f"Written to DB: {key}")
    
    # 2. เขียน cache ทันที
    r.setex(key, ttl, json.dumps(data))
    print(f"Cached: {key}")

def write_through_get(key: str) -> Optional[dict]:
    """ดึงข้อมูลจาก cache หรือ DB"""
    cached = r.get(key)
    if cached:
        print(f"Cache HIT: {key}")
        return json.loads(cached)
    
    # Cache miss (อาจเกิดเมื่อ cache หมดอายุ)
    if key in fake_db:
        print(f"Cache MISS: {key} - loading from DB")
        data = fake_db[key]
        r.setex(key, 3600, json.dumps(data))
        return data
    
    return None

# ทดสอบ Write-Through
write_through_set("product:1", {"id": 1, "name": "Widget", "price": 29.99})
write_through_set("product:2", {"id": 2, "name": "Gadget", "price": 49.99})

product = write_through_get("product:1")
print(f"Product: {product}")
```

### 4.3 Write-Behind (Write-Back) Pattern

### ตัวอย่างที่ 10: Write-Behind Cache

```python
import redis
import json
import threading
import time
from collections import defaultdict
from typing import Optional

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Write-Behind: เขียน cache ก่อน แล้วค่อย flush ไป DB ทีหลัง
# ใช้ List ใน Redis เก็บ pending writes

class WriteBehindCache:
    def __init__(self, redis_client, flush_interval: int = 5):
        self.r = redis_client
        self.flush_interval = flush_interval
        self.dirty_queue = "cache:dirty_queue"
        self.cache_prefix = "cache:data:"
        self.fake_db = {}  # จำลอง DB
        
        # เริ่ม background thread สำหรับ flush
        self.running = True
        self.flush_thread = threading.Thread(target=self._flush_loop, daemon=True)
        self.flush_thread.start()
    
    def set(self, key: str, data: dict, ttl: int = 3600):
        """เขียน cache ก่อน แล้วเพิ่มใน dirty queue"""
        full_key = f"{self.cache_prefix}{key}"
        
        # เขียน cache
        self.r.setex(full_key, ttl, json.dumps(data))
        
        # เพิ่มใน dirty queue
        self.r.rpush(self.dirty_queue, json.dumps({"key": key, "data": data}))
        print(f"Cached {key} and queued for DB write")
    
    def get(self, key: str) -> Optional[dict]:
        """ดึงจาก cache ก่อน"""
        full_key = f"{self.cache_prefix}{key}"
        cached = self.r.get(full_key)
        if cached:
            return json.loads(cached)
        return self.fake_db.get(key)
    
    def _flush_to_db(self, key: str, data: dict):
        """จำลองการ flush ไป DB"""
        self.fake_db[key] = data
        print(f"Flushed {key} to DB")
    
    def _flush_loop(self):
        """Background thread สำหรับ flush dirty data"""
        while self.running:
            time.sleep(self.flush_interval)
            self._flush_dirty()
    
    def _flush_dirty(self):
        """Flush ข้อมูลใน dirty queue ไป DB"""
        count = self.r.llen(self.dirty_queue)
        if count > 0:
            print(f"Flushing {count} dirty items to DB...")
            for _ in range(count):
                item = self.r.lpop(self.dirty_queue)
                if item:
                    data = json.loads(item)
                    self._flush_to_db(data["key"], data["data"])
    
    def stop(self):
        self.running = False
        self._flush_dirty()  # Final flush

# ทดสอบ
cache = WriteBehindCache(r, flush_interval=3)

cache.set("order:1", {"id": 1, "total": 150.00, "status": "processing"})
cache.set("order:2", {"id": 2, "total": 75.50, "status": "shipped"})

print(cache.get("order:1"))  # จาก cache
time.sleep(4)  # รอให้ flush
print(f"DB after flush: {cache.fake_db}")
cache.stop()
```

---

## 5. Cache Invalidation & TTL Management

### ตัวอย่างที่ 11: TTL Management

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# ตั้ง TTL ด้วยวิธีต่าง ๆ
r.set("key1", "value1", ex=60)       # หมดอายุใน 60 วินาที
r.set("key2", "value2", px=5000)     # หมดอายุใน 5000 milliseconds

# ตรวจสอบ TTL
print(f"TTL key1: {r.ttl('key1')} วินาที")
print(f"TTL key2: {r.pttl('key2')} milliseconds")

# -1 = ไม่มี expiry, -2 = key ไม่มีอยู่
r.set("permanent_key", "forever")
print(f"TTL permanent: {r.ttl('permanent_key')}")  # -1

# ตั้ง expiry หลัง set
r.set("data", "important")
r.expire("data", 300)        # หมดอายุใน 5 นาที
r.pexpire("data", 300000)    # หมดอายุใน 5 นาที (milliseconds)

# ตั้ง expiry แบบ Unix timestamp
import time
future_time = int(time.time()) + 3600
r.expireat("data", future_time)

# ยกเลิก expiry (ทำให้ permanent)
r.persist("data")
print(f"After persist TTL: {r.ttl('data')}")  # -1

# Key Space Notifications (ต้องเปิดใน Redis config ก่อน)
# redis.conf: notify-keyspace-events "Ex"
# ใช้สำหรับ react เมื่อ key หมดอายุ

class CacheMonitor:
    """ตรวจสอบ cache statistics"""
    
    def __init__(self, redis_client):
        self.r = redis_client
    
    def get_cache_info(self) -> dict:
        info = self.r.info()
        return {
            "used_memory": info["used_memory_human"],
            "hits": info["keyspace_hits"],
            "misses": info["keyspace_misses"],
            "hit_rate": self._calc_hit_rate(info),
            "total_keys": info.get("db0", {}).get("keys", 0) if "db0" in info else 0
        }
    
    def _calc_hit_rate(self, info: dict) -> float:
        hits = info.get("keyspace_hits", 0)
        misses = info.get("keyspace_misses", 0)
        total = hits + misses
        return round(hits / total * 100, 2) if total > 0 else 0.0
    
    def scan_keys_by_pattern(self, pattern: str):
        """Scan keys โดยไม่ block (ดีกว่า KEYS command)"""
        cursor = 0
        keys = []
        while True:
            cursor, batch = self.r.scan(cursor, match=pattern, count=100)
            keys.extend(batch)
            if cursor == 0:
                break
        return keys
    
    def delete_by_pattern(self, pattern: str) -> int:
        """ลบ keys ที่ตรงกับ pattern"""
        keys = self.scan_keys_by_pattern(pattern)
        if keys:
            return self.r.delete(*keys)
        return 0

monitor = CacheMonitor(r)

# สร้าง test keys
for i in range(10):
    r.set(f"test:key:{i}", f"value{i}", ex=60)
    r.set(f"prod:key:{i}", f"value{i}")

# Scan และลบ test keys
test_keys = monitor.scan_keys_by_pattern("test:*")
print(f"Found {len(test_keys)} test keys: {test_keys[:3]}...")
deleted = monitor.delete_by_pattern("test:*")
print(f"Deleted {deleted} test keys")

cache_info = monitor.get_cache_info()
print(f"Cache stats: {cache_info}")
```

---

## 6. Redis Pub/Sub

### ตัวอย่างที่ 12: Publisher/Subscriber

```python
import redis
import threading
import json
import time

# Publisher
pub_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Subscriber ต้องใช้ connection แยก
sub_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

def publisher():
    """ส่ง messages ไปยัง channel"""
    time.sleep(1)  # รอให้ subscriber พร้อม
    
    events = [
        {"type": "user.signup", "user_id": 1, "email": "user@example.com"},
        {"type": "order.created", "order_id": 100, "total": 299.99},
        {"type": "product.updated", "product_id": 5, "price": 49.99},
    ]
    
    for event in events:
        channel = f"events:{event['type'].split('.')[0]}"
        message = json.dumps(event)
        subscribers = pub_client.publish(channel, message)
        print(f"Published to {channel}: {event['type']} ({subscribers} subscribers)")
        time.sleep(0.5)

def subscriber():
    """รับ messages จาก channels"""
    pubsub = sub_client.pubsub()
    
    # Subscribe หลาย channels
    pubsub.subscribe("events:user", "events:order", "events:product")
    print("Subscribed to channels: events:user, events:order, events:product")
    
    # รับ messages
    for message in pubsub.listen():
        if message['type'] == 'message':
            channel = message['channel']
            data = json.loads(message['data'])
            print(f"Received on {channel}: {data}")
        elif message['type'] == 'subscribe':
            print(f"Subscribed to: {message['channel']}")

# Pattern Subscribe
def pattern_subscriber():
    """Subscribe ด้วย pattern"""
    pubsub = sub_client.pubsub()
    pubsub.psubscribe("events:*")  # Subscribe ทุก channel ที่ขึ้นต้นด้วย events:
    
    count = 0
    for message in pubsub.listen():
        if message['type'] == 'pmessage':
            print(f"Pattern message on {message['channel']}: {message['data']}")
            count += 1
            if count >= 3:
                break
    
    pubsub.close()

# รัน subscriber ใน thread แยก
sub_thread = threading.Thread(target=subscriber, daemon=True)
sub_thread.start()

# รัน publisher
publisher_thread = threading.Thread(target=publisher, daemon=True)
publisher_thread.start()

time.sleep(5)  # รอดูผลลัพธ์

# Pub/Sub สำหรับ Real-time notifications
class NotificationSystem:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def send_notification(self, user_id: int, notification: dict):
        channel = f"notifications:{user_id}"
        message = json.dumps(notification)
        return self.r.publish(channel, message)
    
    def listen_for_user(self, user_id: int, callback):
        pubsub = self.r.pubsub()
        channel = f"notifications:{user_id}"
        pubsub.subscribe(**{channel: callback})
        pubsub.run_in_thread(sleep_time=0.01, daemon=True)
        return pubsub

def handle_notification(message):
    if message['type'] == 'message':
        data = json.loads(message['data'])
        print(f"Notification received: {data}")

notif_system = NotificationSystem(pub_client)
pubsub = notif_system.listen_for_user(42, handle_notification)
time.sleep(0.2)
notif_system.send_notification(42, {"title": "New message", "body": "Hello!"})
notif_system.send_notification(42, {"title": "Order shipped", "body": "Your order is on the way!"})
time.sleep(0.5)
pubsub.unsubscribe()
```

---

## 7. Distributed Locking

### ตัวอย่างที่ 13: Simple Distributed Lock

```python
import redis
import time
import uuid
import threading

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class DistributedLock:
    """
    Distributed Lock ด้วย Redis
    ใช้ SET NX EX ซึ่งเป็น atomic operation
    """
    
    def __init__(self, redis_client, lock_name: str, expire: int = 10):
        self.r = redis_client
        self.lock_name = f"lock:{lock_name}"
        self.expire = expire
        self.lock_value = str(uuid.uuid4())  # unique value สำหรับ lock นี้
        self._acquired = False
    
    def acquire(self, timeout: float = 10.0) -> bool:
        """พยายาม acquire lock"""
        start = time.time()
        
        while time.time() - start < timeout:
            # SET NX EX - atomic: set ถ้ายังไม่มี key
            acquired = self.r.set(
                self.lock_name,
                self.lock_value,
                nx=True,  # Only set if Not eXists
                ex=self.expire
            )
            
            if acquired:
                self._acquired = True
                print(f"Lock {self.lock_name} acquired by {self.lock_value[:8]}...")
                return True
            
            # รอแล้วลองใหม่
            time.sleep(0.1)
        
        print(f"Failed to acquire lock {self.lock_name}")
        return False
    
    def release(self) -> bool:
        """Release lock (ต้องเป็นเจ้าของเท่านั้น)"""
        # ใช้ Lua script เพื่อให้ check-and-delete เป็น atomic
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        result = self.r.eval(lua_script, 1, self.lock_name, self.lock_value)
        if result:
            self._acquired = False
            print(f"Lock {self.lock_name} released")
            return True
        else:
            print(f"Could not release lock (not owner or already expired)")
            return False
    
    def extend(self, additional_seconds: int) -> bool:
        """ต่ออายุ lock"""
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("expire", KEYS[1], ARGV[2])
        else
            return 0
        end
        """
        result = self.r.eval(lua_script, 1, self.lock_name, self.lock_value, additional_seconds)
        return bool(result)
    
    def __enter__(self):
        self.acquire()
        return self
    
    def __exit__(self, *args):
        if self._acquired:
            self.release()

# ทดสอบ concurrent access
results = []

def critical_section(worker_id: int):
    """งานที่ต้องทำแบบ exclusive"""
    lock = DistributedLock(r, "critical_resource", expire=5)
    
    if lock.acquire(timeout=10):
        try:
            print(f"Worker {worker_id}: เริ่มทำงาน critical section")
            time.sleep(0.5)  # จำลองงาน
            results.append(worker_id)
            print(f"Worker {worker_id}: เสร็จสิ้น")
        finally:
            lock.release()
    else:
        print(f"Worker {worker_id}: ไม่สามารถ acquire lock ได้")

# รัน concurrent workers
threads = [threading.Thread(target=critical_section, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Execution order: {results}")
print("หมายเหตุ: แต่ละ worker รันแบบ serial ไม่ใช่ parallel")
```

### ตัวอย่างที่ 14: Redlock Algorithm

```python
import redis
import time
import uuid
from typing import List, Optional

# Redlock - ใช้ Redis หลาย instances เพื่อ safety สูงขึ้น
class Redlock:
    """
    Redlock Algorithm - Distributed lock บน Redis cluster
    ต้องการ Redis instances จำนวนคี่ (เช่น 3, 5)
    Lock สำเร็จเมื่อ acquire ได้จาก majority (N/2 + 1)
    """
    
    CLOCK_DRIFT_FACTOR = 0.01
    
    def __init__(self, redis_nodes: List[redis.Redis]):
        self.nodes = redis_nodes
        self.quorum = len(nodes) // 2 + 1
    
    def _acquire_instance(self, node: redis.Redis, resource: str, 
                           value: str, ttl: int) -> bool:
        try:
            return bool(node.set(f"lock:{resource}", value, nx=True, px=ttl))
        except redis.RedisError:
            return False
    
    def _release_instance(self, node: redis.Redis, resource: str, value: str):
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        try:
            node.eval(lua_script, 1, f"lock:{resource}", value)
        except redis.RedisError:
            pass
    
    def acquire(self, resource: str, ttl: int = 10000) -> Optional[dict]:
        """
        Acquire lock บน Redis cluster
        Returns lock token หรือ None ถ้าไม่สำเร็จ
        """
        value = str(uuid.uuid4())
        start = int(time.time() * 1000)
        
        acquired_count = 0
        for node in self.nodes:
            if self._acquire_instance(node, resource, value, ttl):
                acquired_count += 1
        
        elapsed = int(time.time() * 1000) - start
        validity = ttl - elapsed - int(ttl * self.CLOCK_DRIFT_FACTOR)
        
        if acquired_count >= self.quorum and validity > 0:
            return {"value": value, "validity": validity, "resource": resource}
        else:
            # ล้มเหลว - release ทุก node
            for node in self.nodes:
                self._release_instance(node, resource, value)
            return None
    
    def release(self, lock_token: dict):
        """Release lock บนทุก nodes"""
        resource = lock_token["resource"]
        value = lock_token["value"]
        for node in self.nodes:
            self._release_instance(node, resource, value)

# จำลอง Redis cluster (ในการใช้งานจริงควรเป็น instances แยก)
# nodes = [
#     redis.Redis(host='redis1', port=6379),
#     redis.Redis(host='redis2', port=6379),
#     redis.Redis(host='redis3', port=6379),
# ]
# 
# redlock = Redlock(nodes)
# lock = redlock.acquire("payment_processor", ttl=5000)
# if lock:
#     try:
#         # critical section
#         pass
#     finally:
#         redlock.release(lock)
print("Redlock algorithm สำหรับ distributed systems ที่ต้องการ high availability")
```

---

## 8. Session Storage Pattern

### ตัวอย่างที่ 15: Session Management

```python
import redis
import json
import uuid
import hashlib
import time
from typing import Optional, Dict, Any

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class SessionStore:
    """
    Session Storage ด้วย Redis
    เหมาะกับ stateless applications เช่น REST APIs
    """
    
    SESSION_PREFIX = "session:"
    DEFAULT_TTL = 86400  # 24 ชั่วโมง
    
    def __init__(self, redis_client, ttl: int = DEFAULT_TTL):
        self.r = redis_client
        self.ttl = ttl
    
    def create_session(self, user_data: Dict[str, Any]) -> str:
        """สร้าง session ใหม่ และคืน session_id"""
        session_id = str(uuid.uuid4())
        key = f"{self.SESSION_PREFIX}{session_id}"
        
        session_data = {
            **user_data,
            "created_at": int(time.time()),
            "last_active": int(time.time())
        }
        
        # เก็บใน Hash เพื่อ update ทีละ field ได้
        self.r.hset(key, mapping={
            k: json.dumps(v) if isinstance(v, (dict, list)) else str(v)
            for k, v in session_data.items()
        })
        self.r.expire(key, self.ttl)
        
        return session_id
    
    def get_session(self, session_id: str) -> Optional[Dict[str, Any]]:
        """ดึงข้อมูล session"""
        key = f"{self.SESSION_PREFIX}{session_id}"
        data = self.r.hgetall(key)
        
        if not data:
            return None
        
        # Refresh TTL เมื่อมีการใช้งาน (sliding expiration)
        self.r.expire(key, self.ttl)
        
        # Update last_active
        self.r.hset(key, "last_active", str(int(time.time())))
        
        return data
    
    def update_session(self, session_id: str, updates: Dict[str, Any]) -> bool:
        """อัปเดต session data"""
        key = f"{self.SESSION_PREFIX}{session_id}"
        
        if not self.r.exists(key):
            return False
        
        self.r.hset(key, mapping={
            k: json.dumps(v) if isinstance(v, (dict, list)) else str(v)
            for k, v in updates.items()
        })
        self.r.expire(key, self.ttl)
        return True
    
    def delete_session(self, session_id: str) -> bool:
        """ลบ session (logout)"""
        key = f"{self.SESSION_PREFIX}{session_id}"
        return bool(self.r.delete(key))
    
    def extend_session(self, session_id: str, additional_seconds: int) -> bool:
        """ต่ออายุ session"""
        key = f"{self.SESSION_PREFIX}{session_id}"
        ttl = self.r.ttl(key)
        if ttl > 0:
            self.r.expire(key, ttl + additional_seconds)
            return True
        return False
    
    def get_active_sessions_count(self) -> int:
        """นับ session ที่ active อยู่"""
        cursor = 0
        count = 0
        while True:
            cursor, keys = self.r.scan(cursor, match=f"{self.SESSION_PREFIX}*", count=100)
            count += len(keys)
            if cursor == 0:
                break
        return count

# ทดสอบ
store = SessionStore(r, ttl=3600)

# Login
session_id = store.create_session({
    "user_id": "123",
    "username": "alice",
    "email": "alice@example.com",
    "role": "admin",
    "permissions": ["read", "write", "delete"]
})
print(f"Session created: {session_id}")

# ใช้ session
session = store.get_session(session_id)
print(f"Session data: {dict(list(session.items())[:3])}...")

# Update session
store.update_session(session_id, {"last_page": "/dashboard", "theme": "dark"})

# Logout
store.delete_session(session_id)
print(f"Session deleted, active sessions: {store.get_active_sessions_count()}")
```

---

## 9. Rate Limiting

### ตัวอย่างที่ 16: Fixed Window Rate Limiter

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class FixedWindowRateLimiter:
    """
    Fixed Window Rate Limiter
    อนุญาต N requests ต่อ window (เช่น 100 requests/minute)
    """
    
    def __init__(self, redis_client, max_requests: int, window_seconds: int):
        self.r = redis_client
        self.max_requests = max_requests
        self.window = window_seconds
    
    def is_allowed(self, identifier: str) -> tuple[bool, dict]:
        """
        ตรวจสอบว่าอนุญาตให้ทำ request ได้หรือไม่
        Returns: (allowed, info_dict)
        """
        # สร้าง key ที่รวม window timestamp
        window_start = int(time.time() // self.window) * self.window
        key = f"ratelimit:{identifier}:{window_start}"
        
        # Atomic increment
        current = self.r.incr(key)
        
        # ตั้ง TTL สำหรับ key ใหม่
        if current == 1:
            self.r.expire(key, self.window * 2)  # เผื่อเวลา cleanup
        
        remaining = max(0, self.max_requests - current)
        reset_at = window_start + self.window
        
        return current <= self.max_requests, {
            "limit": self.max_requests,
            "remaining": remaining,
            "reset": reset_at,
            "current": current
        }

limiter = FixedWindowRateLimiter(r, max_requests=5, window_seconds=10)

for i in range(8):
    allowed, info = limiter.is_allowed("user:123")
    status = "OK" if allowed else "BLOCKED"
    print(f"Request {i+1}: {status} | Remaining: {info['remaining']}")
```

### ตัวอย่างที่ 17: Sliding Window Rate Limiter

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class SlidingWindowRateLimiter:
    """
    Sliding Window Rate Limiter ด้วย Sorted Set
    แม่นยำกว่า Fixed Window แต่ใช้ memory มากกว่า
    """
    
    def __init__(self, redis_client, max_requests: int, window_seconds: int):
        self.r = redis_client
        self.max_requests = max_requests
        self.window = window_seconds
    
    def is_allowed(self, identifier: str) -> tuple[bool, dict]:
        key = f"sliding_ratelimit:{identifier}"
        now = time.time()
        window_start = now - self.window
        
        # ใช้ Lua script เพื่อ atomic operations
        lua_script = """
        local key = KEYS[1]
        local now = tonumber(ARGV[1])
        local window_start = tonumber(ARGV[2])
        local max_requests = tonumber(ARGV[3])
        local window = tonumber(ARGV[4])
        
        -- ลบ entries เก่าออกจาก sorted set
        redis.call('zremrangebyscore', key, 0, window_start)
        
        -- นับ requests ใน window ปัจจุบัน
        local count = redis.call('zcard', key)
        
        if count < max_requests then
            -- เพิ่ม request ใหม่
            redis.call('zadd', key, now, now)
            redis.call('expire', key, window * 2)
            return {1, count + 1}
        else
            return {0, count}
        end
        """
        
        result = self.r.eval(
            lua_script, 1, key,
            now, window_start, self.max_requests, self.window
        )
        
        allowed = bool(result[0])
        current_count = int(result[1])
        
        return allowed, {
            "limit": self.max_requests,
            "remaining": max(0, self.max_requests - current_count),
            "current": current_count
        }

sliding_limiter = SlidingWindowRateLimiter(r, max_requests=5, window_seconds=10)

print("Sliding Window Rate Limiter:")
for i in range(8):
    allowed, info = sliding_limiter.is_allowed("api_user:456")
    status = "OK" if allowed else "BLOCKED"
    print(f"Request {i+1}: {status} | Remaining: {info['remaining']}/{info['limit']}")
    time.sleep(0.2)
```

### ตัวอย่างที่ 18: Token Bucket Rate Limiter

```python
import redis
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class TokenBucketRateLimiter:
    """
    Token Bucket Algorithm
    - bucket มี capacity สูงสุด N tokens
    - tokens เพิ่มขึ้นตามเวลา (refill_rate tokens/second)
    - request ใช้ 1 token ต่อครั้ง
    - รองรับ burst requests ได้
    """
    
    def __init__(self, redis_client, capacity: int, refill_rate: float):
        self.r = redis_client
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
    
    def consume(self, identifier: str, tokens: int = 1) -> tuple[bool, dict]:
        key = f"token_bucket:{identifier}"
        
        lua_script = """
        local key = KEYS[1]
        local capacity = tonumber(ARGV[1])
        local refill_rate = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])
        local tokens_requested = tonumber(ARGV[4])
        
        -- ดึงข้อมูล bucket ปัจจุบัน
        local bucket = redis.call('hmget', key, 'tokens', 'last_refill')
        local current_tokens = tonumber(bucket[1]) or capacity
        local last_refill = tonumber(bucket[2]) or now
        
        -- คำนวณ tokens ที่เพิ่มขึ้นตามเวลา
        local elapsed = now - last_refill
        local new_tokens = math.min(capacity, current_tokens + elapsed * refill_rate)
        
        if new_tokens >= tokens_requested then
            -- มี tokens พอ
            local remaining = new_tokens - tokens_requested
            redis.call('hset', key, 'tokens', remaining, 'last_refill', now)
            redis.call('expire', key, 3600)
            return {1, math.floor(remaining)}
        else
            -- ไม่มี tokens พอ
            redis.call('hset', key, 'tokens', new_tokens, 'last_refill', now)
            redis.call('expire', key, 3600)
            return {0, math.floor(new_tokens)}
        end
        """
        
        result = self.r.eval(
            lua_script, 1, key,
            self.capacity, self.refill_rate,
            time.time(), tokens
        )
        
        allowed = bool(result[0])
        remaining_tokens = int(result[1])
        
        return allowed, {
            "capacity": self.capacity,
            "remaining": remaining_tokens,
            "refill_rate": self.refill_rate
        }

# ทดสอบ Token Bucket (capacity=10, refill 2 tokens/second)
bucket_limiter = TokenBucketRateLimiter(r, capacity=10, refill_rate=2.0)

print("Token Bucket Rate Limiter:")
# Burst requests
for i in range(12):
    allowed, info = bucket_limiter.consume("client:789")
    status = "OK" if allowed else "BLOCKED"
    print(f"Request {i+1}: {status} | Tokens remaining: {info['remaining']}/{info['capacity']}")

# รอ 3 วินาที แล้วลองใหม่ (ควรมี 6 tokens)
print("\nรอ 3 วินาที...")
time.sleep(3)
allowed, info = bucket_limiter.consume("client:789")
print(f"After refill: {allowed} | Tokens: {info['remaining']}/{info['capacity']}")
```

---

## 10. Caching ใน FastAPI

### ตัวอย่างที่ 19: FastAPI + Redis Cache Dependency

```python
from fastapi import FastAPI, Depends, HTTPException
from functools import wraps
import redis
import json
import hashlib
import time
from typing import Optional, Any, Callable

app = FastAPI()

# Redis client
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    decode_responses=True
)

# Dependency สำหรับ Redis client
def get_redis():
    return redis_client

# Cache decorator สำหรับ FastAPI endpoints
def cache_response(ttl: int = 300, key_prefix: str = ""):
    """Decorator สำหรับ cache API responses"""
    def decorator(func: Callable):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # สร้าง cache key จาก function args
            cache_parts = [key_prefix or func.__name__]
            for k, v in sorted(kwargs.items()):
                cache_parts.append(f"{k}:{v}")
            cache_key = ":".join(cache_parts)
            
            r = redis_client
            
            # ตรวจสอบ cache
            cached = r.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # ไม่มีใน cache - รัน function จริง
            result = await func(*args, **kwargs)
            
            # Cache ผลลัพธ์
            r.setex(cache_key, ttl, json.dumps(result))
            return result
        
        return wrapper
    return decorator

# Product database (จำลอง)
products_db = {
    1: {"id": 1, "name": "Laptop", "price": 999.99, "stock": 50},
    2: {"id": 2, "name": "Phone", "price": 699.99, "stock": 100},
    3: {"id": 3, "name": "Tablet", "price": 499.99, "stock": 75},
}

@app.get("/products/{product_id}")
@cache_response(ttl=60, key_prefix="product")
async def get_product(product_id: int):
    """ดึงข้อมูล product พร้อม caching"""
    if product_id not in products_db:
        raise HTTPException(status_code=404, detail="Product not found")
    
    # จำลอง DB query
    time.sleep(0.1)
    return products_db[product_id]

@app.put("/products/{product_id}")
async def update_product(
    product_id: int,
    price: float,
    r: redis.Redis = Depends(get_redis)
):
    """อัปเดต product และ invalidate cache"""
    if product_id not in products_db:
        raise HTTPException(status_code=404, detail="Product not found")
    
    products_db[product_id]["price"] = price
    
    # Invalidate cache
    cache_key = f"product:product_id:{product_id}"
    r.delete(cache_key)
    
    return {"message": "Updated", "product": products_db[product_id]}

# Rate limiting middleware
class RateLimitMiddleware:
    def __init__(self, app, redis_client, max_requests: int = 100, window: int = 60):
        self.app = app
        self.r = redis_client
        self.max_requests = max_requests
        self.window = window
    
    async def __call__(self, scope, receive, send):
        if scope["type"] == "http":
            # ดึง client IP
            client_ip = scope.get("client", ["unknown"])[0]
            key = f"ratelimit:global:{client_ip}"
            
            current = self.r.incr(key)
            if current == 1:
                self.r.expire(key, self.window)
            
            if current > self.max_requests:
                from starlette.responses import JSONResponse
                response = JSONResponse(
                    {"error": "Rate limit exceeded"},
                    status_code=429,
                    headers={
                        "X-RateLimit-Limit": str(self.max_requests),
                        "X-RateLimit-Remaining": "0",
                        "Retry-After": str(self.r.ttl(key))
                    }
                )
                await response(scope, receive, send)
                return
        
        await self.app(scope, receive, send)

# เพิ่ม middleware
# app.add_middleware(RateLimitMiddleware, redis_client=redis_client)

print("FastAPI + Redis Cache ready!")
print("รัน: uvicorn filename:app --reload")
```

---

## 11. ตัวอย่างโปรแกรมจริง: Caching Layer

### ตัวอย่างที่ 20: Full Caching Layer

```python
import redis
import json
import time
import hashlib
from typing import Optional, Any, Dict, List
from functools import wraps

class CacheLayer:
    """
    Complete Caching Layer สำหรับ Application
    รองรับ:
    - Function result caching
    - Cache invalidation by tags
    - Cache statistics
    - Multi-level caching
    """
    
    def __init__(self, redis_client, default_ttl: int = 300):
        self.r = redis_client
        self.default_ttl = default_ttl
        self.stats_key = "cache:stats"
    
    def _make_key(self, namespace: str, *args, **kwargs) -> str:
        """สร้าง cache key"""
        parts = [namespace]
        parts.extend(str(a) for a in args)
        parts.extend(f"{k}={v}" for k, v in sorted(kwargs.items()))
        key_str = ":".join(parts)
        # Hash ถ้า key ยาวเกินไป
        if len(key_str) > 100:
            return f"{namespace}:{hashlib.md5(key_str.encode()).hexdigest()}"
        return key_str
    
    def get(self, key: str) -> Optional[Any]:
        """ดึงข้อมูลจาก cache"""
        value = self.r.get(key)
        if value is not None:
            self.r.hincrby(self.stats_key, "hits", 1)
            return json.loads(value)
        self.r.hincrby(self.stats_key, "misses", 1)
        return None
    
    def set(self, key: str, value: Any, ttl: Optional[int] = None, tags: List[str] = None):
        """เก็บข้อมูลใน cache พร้อม optional tags"""
        ttl = ttl or self.default_ttl
        self.r.setex(key, ttl, json.dumps(value))
        
        # เพิ่ม key ไปยัง tag sets
        if tags:
            for tag in tags:
                tag_key = f"cache:tag:{tag}"
                self.r.sadd(tag_key, key)
                self.r.expire(tag_key, ttl * 2)
        
        self.r.hincrby(self.stats_key, "sets", 1)
    
    def invalidate(self, *keys: str):
        """ลบ cache keys"""
        if keys:
            deleted = self.r.delete(*keys)
            self.r.hincrby(self.stats_key, "invalidations", deleted)
            return deleted
        return 0
    
    def invalidate_by_tag(self, tag: str) -> int:
        """ลบ cache ทุก key ที่มี tag นี้"""
        tag_key = f"cache:tag:{tag}"
        keys = self.r.smembers(tag_key)
        
        count = 0
        if keys:
            count = self.r.delete(*keys)
            self.r.delete(tag_key)
        
        return count
    
    def get_stats(self) -> Dict:
        stats = self.r.hgetall(self.stats_key)
        hits = int(stats.get("hits", 0))
        misses = int(stats.get("misses", 0))
        total = hits + misses
        
        return {
            "hits": hits,
            "misses": misses,
            "sets": int(stats.get("sets", 0)),
            "invalidations": int(stats.get("invalidations", 0)),
            "hit_rate": f"{hits/total*100:.1f}%" if total > 0 else "N/A"
        }
    
    def cached(self, namespace: str, ttl: Optional[int] = None, tags: List[str] = None):
        """Decorator สำหรับ cache function results"""
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                key = self._make_key(namespace, *args, **kwargs)
                
                # ตรวจสอบ cache
                cached_value = self.get(key)
                if cached_value is not None:
                    return cached_value
                
                # รัน function จริง
                result = func(*args, **kwargs)
                
                # Cache ผลลัพธ์
                self.set(key, result, ttl=ttl, tags=tags)
                return result
            
            # เพิ่ม method สำหรับ invalidate cache ของ function นี้
            wrapper.invalidate = lambda *args, **kwargs: \
                self.invalidate(self._make_key(namespace, *args, **kwargs))
            
            return wrapper
        return decorator

# ทดสอบ CacheLayer
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)
cache = CacheLayer(r, default_ttl=60)

# จำลอง database
products = {
    1: {"id": 1, "name": "Widget", "category": "tools", "price": 9.99},
    2: {"id": 2, "name": "Gadget", "category": "electronics", "price": 29.99},
    3: {"id": 3, "name": "Gizmo", "category": "electronics", "price": 49.99},
}

@cache.cached("product", ttl=60, tags=["products"])
def get_product(product_id: int):
    print(f"  [DB] Loading product {product_id}...")
    time.sleep(0.1)
    return products.get(product_id)

@cache.cached("products_by_category", ttl=120, tags=["products"])
def get_products_by_category(category: str):
    print(f"  [DB] Loading products in category '{category}'...")
    time.sleep(0.2)
    return [p for p in products.values() if p["category"] == category]

# ทดสอบ
print("=== Testing Cache Layer ===")
p1 = get_product(1)
print(f"First call: {p1['name']}")
p1_cached = get_product(1)
print(f"Cached call: {p1_cached['name']}")

electronics = get_products_by_category("electronics")
print(f"Electronics: {[p['name'] for p in electronics]}")

# Invalidate ทั้ง products category
print("\nInvalidating all 'products' tagged caches...")
invalidated = cache.invalidate_by_tag("products")
print(f"Invalidated {invalidated} cache entries")

# ต้องดึงจาก DB ใหม่
p1_fresh = get_product(1)
print(f"After invalidation: {p1_fresh['name']}")

print("\nCache Statistics:")
print(cache.get_stats())
```

---

## 12. Leaderboard System

### ตัวอย่างที่ 21: Real-time Leaderboard

```python
import redis
import time
import random
from typing import List, Dict, Optional, Tuple

class LeaderboardSystem:
    """
    Real-time Leaderboard System ด้วย Redis Sorted Sets
    Features:
    - Global leaderboard
    - Weekly/Monthly leaderboards  
    - Player profiles
    - Rank history
    """
    
    def __init__(self, redis_client, game_id: str):
        self.r = redis_client
        self.game_id = game_id
        self.global_key = f"lb:{game_id}:global"
        self.player_prefix = f"player:{game_id}:"
    
    def _get_period_key(self, period: str) -> str:
        """สร้าง key สำหรับ period (weekly/monthly)"""
        now = time.time()
        if period == "weekly":
            week = int(now // (7 * 24 * 3600))
            return f"lb:{self.game_id}:week:{week}"
        elif period == "monthly":
            import datetime
            dt = datetime.datetime.now()
            return f"lb:{self.game_id}:month:{dt.year}-{dt.month:02d}"
        return self.global_key
    
    def submit_score(self, player_id: str, score: float, player_name: str = None):
        """ส่ง score ใหม่"""
        pipe = self.r.pipeline()
        
        # Update global leaderboard (เก็บ score สูงสุด)
        current_score = self.r.zscore(self.global_key, player_id)
        if current_score is None or score > current_score:
            pipe.zadd(self.global_key, {player_id: score})
        
        # Update period leaderboards
        for period in ["weekly", "monthly"]:
            period_key = self._get_period_key(period)
            period_score = self.r.zscore(period_key, player_id)
            if period_score is None or score > period_score:
                pipe.zadd(period_key, {player_id: score})
                pipe.expire(period_key, 7 * 24 * 3600 if period == "weekly" else 35 * 24 * 3600)
        
        # เก็บ player profile
        if player_name:
            player_key = f"{self.player_prefix}{player_id}"
            pipe.hset(player_key, mapping={
                "name": player_name,
                "player_id": player_id,
                "last_score": str(score),
                "last_played": str(int(time.time()))
            })
        
        pipe.execute()
    
    def get_top(self, count: int = 10, period: str = "global") -> List[Dict]:
        """ดึง top players"""
        key = self._get_period_key(period) if period != "global" else self.global_key
        entries = self.r.zrevrange(key, 0, count - 1, withscores=True)
        
        result = []
        for rank, (player_id, score) in enumerate(entries, 1):
            player_key = f"{self.player_prefix}{player_id}"
            player_info = self.r.hgetall(player_key)
            
            result.append({
                "rank": rank,
                "player_id": player_id,
                "name": player_info.get("name", player_id),
                "score": int(score)
            })
        
        return result
    
    def get_player_rank(self, player_id: str, period: str = "global") -> Optional[Dict]:
        """ดึง rank ของ player คนใดคนหนึ่ง"""
        key = self._get_period_key(period) if period != "global" else self.global_key
        
        rank = self.r.zrevrank(key, player_id)
        score = self.r.zscore(key, player_id)
        
        if rank is None:
            return None
        
        total_players = self.r.zcard(key)
        player_key = f"{self.player_prefix}{player_id}"
        player_info = self.r.hgetall(player_key)
        
        return {
            "rank": rank + 1,
            "player_id": player_id,
            "name": player_info.get("name", player_id),
            "score": int(score) if score else 0,
            "total_players": total_players,
            "percentile": round((total_players - rank) / total_players * 100, 1)
        }
    
    def get_nearby_players(self, player_id: str, count: int = 5) -> List[Dict]:
        """ดึง players รอบ ๆ player คนนี้"""
        rank = self.r.zrevrank(self.global_key, player_id)
        if rank is None:
            return []
        
        total = self.r.zcard(self.global_key)
        start = max(0, rank - count // 2)
        end = min(total - 1, start + count - 1)
        
        entries = self.r.zrevrange(self.global_key, start, end, withscores=True)
        result = []
        
        for i, (pid, score) in enumerate(entries, start + 1):
            player_key = f"{self.player_prefix}{pid}"
            player_info = self.r.hgetall(player_key)
            result.append({
                "rank": i,
                "player_id": pid,
                "name": player_info.get("name", pid),
                "score": int(score),
                "is_current": pid == player_id
            })
        
        return result

# ทดสอบ Leaderboard
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)
lb = LeaderboardSystem(r, "puzzle_master")

# จำลองผู้เล่น
players = [
    ("p1", "Alice", 8500),
    ("p2", "Bob", 7200),
    ("p3", "Charlie", 9100),
    ("p4", "Diana", 6800),
    ("p5", "Eve", 8900),
    ("p6", "Frank", 7600),
    ("p7", "Grace", 9500),
    ("p8", "Henry", 5500),
]

print("=== Leaderboard Demo ===")
for pid, name, score in players:
    lb.submit_score(pid, score, name)
    # จำลอง score update
    better_score = score + random.randint(0, 500)
    lb.submit_score(pid, better_score, name)

print("\nTop 5 Players:")
for player in lb.get_top(5):
    print(f"  #{player['rank']} {player['name']}: {player['score']:,}")

print("\nAlice's rank:")
alice_rank = lb.get_player_rank("p1")
print(f"  Rank {alice_rank['rank']}/{alice_rank['total_players']} ({alice_rank['percentile']}th percentile)")
print(f"  Score: {alice_rank['score']:,}")

print("\nPlayers near Alice:")
for p in lb.get_nearby_players("p1", count=5):
    marker = " <-- YOU" if p["is_current"] else ""
    print(f"  #{p['rank']} {p['name']}: {p['score']:,}{marker}")
```

---

## 13. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Shopping Cart ด้วย Redis Hash

**โจทย์**: สร้าง ShoppingCart class ที่:
- เก็บ cart items ใน Redis Hash
- รองรับ add, remove, update quantity
- คำนวณ total price
- Cart หมดอายุใน 24 ชั่วโมง

```python
# เฉลย
import redis
import json

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class ShoppingCart:
    CART_PREFIX = "cart:"
    CART_TTL = 86400  # 24 ชั่วโมง
    
    def __init__(self, redis_client, cart_id: str):
        self.r = redis_client
        self.cart_id = cart_id
        self.key = f"{self.CART_PREFIX}{cart_id}"
    
    def add_item(self, product_id: str, name: str, price: float, quantity: int = 1):
        item_key = f"item:{product_id}"
        existing = self.r.hget(self.key, item_key)
        
        if existing:
            item = json.loads(existing)
            item["quantity"] += quantity
        else:
            item = {"product_id": product_id, "name": name, "price": price, "quantity": quantity}
        
        self.r.hset(self.key, item_key, json.dumps(item))
        self.r.expire(self.key, self.CART_TTL)
        return item
    
    def remove_item(self, product_id: str):
        self.r.hdel(self.key, f"item:{product_id}")
    
    def update_quantity(self, product_id: str, quantity: int):
        if quantity <= 0:
            self.remove_item(product_id)
            return
        
        item_key = f"item:{product_id}"
        existing = self.r.hget(self.key, item_key)
        if existing:
            item = json.loads(existing)
            item["quantity"] = quantity
            self.r.hset(self.key, item_key, json.dumps(item))
    
    def get_items(self) -> list:
        all_items = self.r.hgetall(self.key)
        return [json.loads(v) for k, v in all_items.items() if k.startswith("item:")]
    
    def get_total(self) -> float:
        items = self.get_items()
        return sum(item["price"] * item["quantity"] for item in items)
    
    def clear(self):
        self.r.delete(self.key)
    
    def checkout_summary(self) -> dict:
        items = self.get_items()
        return {
            "items": items,
            "item_count": sum(item["quantity"] for item in items),
            "total": round(self.get_total(), 2)
        }

# ทดสอบ
cart = ShoppingCart(r, "user123_cart")
cart.add_item("p001", "Laptop", 999.99, 1)
cart.add_item("p002", "Mouse", 29.99, 2)
cart.add_item("p003", "Keyboard", 79.99, 1)
cart.update_quantity("p002", 3)

summary = cart.checkout_summary()
print(f"Items: {summary['item_count']}")
print(f"Total: ${summary['total']:.2f}")
for item in summary['items']:
    print(f"  - {item['name']} x{item['quantity']}: ${item['price'] * item['quantity']:.2f}")
```

### แบบฝึกหัดที่ 2: Job Queue ด้วย Redis List

**โจทย์**: สร้าง Job Queue ที่:
- รองรับ priority levels (high, normal, low)
- Worker ดึง job จาก priority สูงสุดก่อน
- Track job status (pending, processing, done, failed)
- Retry failed jobs ได้

```python
# เฉลย
import redis
import json
import uuid
import time
from enum import Enum

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class Priority(Enum):
    HIGH = "high"
    NORMAL = "normal"
    LOW = "low"

class JobQueue:
    STATUS_PREFIX = "job:status:"
    
    def __init__(self, redis_client, queue_name: str):
        self.r = redis_client
        self.name = queue_name
        self.queues = {
            Priority.HIGH: f"{queue_name}:high",
            Priority.NORMAL: f"{queue_name}:normal",
            Priority.LOW: f"{queue_name}:low"
        }
    
    def enqueue(self, job_type: str, payload: dict, 
                priority: Priority = Priority.NORMAL) -> str:
        job_id = str(uuid.uuid4())[:8]
        job = {
            "id": job_id,
            "type": job_type,
            "payload": payload,
            "created_at": int(time.time()),
            "retry_count": 0
        }
        
        queue_key = self.queues[priority]
        self.r.rpush(queue_key, json.dumps(job))
        
        # Track status
        status_key = f"{self.STATUS_PREFIX}{job_id}"
        self.r.hset(status_key, mapping={
            "status": "pending",
            "priority": priority.value,
            "created_at": str(int(time.time()))
        })
        self.r.expire(status_key, 3600)
        
        return job_id
    
    def dequeue(self, timeout: int = 1) -> dict | None:
        """ดึง job จาก priority สูงสุดก่อน"""
        for priority in [Priority.HIGH, Priority.NORMAL, Priority.LOW]:
            queue_key = self.queues[priority]
            item = self.r.lpop(queue_key)
            if item:
                job = json.loads(item)
                # Mark as processing
                status_key = f"{self.STATUS_PREFIX}{job['id']}"
                self.r.hset(status_key, mapping={
                    "status": "processing",
                    "started_at": str(int(time.time()))
                })
                return job
        return None
    
    def complete_job(self, job_id: str):
        status_key = f"{self.STATUS_PREFIX}{job_id}"
        self.r.hset(status_key, mapping={
            "status": "done",
            "completed_at": str(int(time.time()))
        })
    
    def fail_job(self, job_id: str, error: str, job_data: dict, max_retries: int = 3):
        status_key = f"{self.STATUS_PREFIX}{job_id}"
        retry_count = job_data.get("retry_count", 0) + 1
        
        if retry_count <= max_retries:
            job_data["retry_count"] = retry_count
            self.r.rpush(self.queues[Priority.LOW], json.dumps(job_data))
            self.r.hset(status_key, mapping={
                "status": "retrying",
                "retry_count": str(retry_count),
                "error": error
            })
        else:
            self.r.hset(status_key, mapping={
                "status": "failed",
                "error": error,
                "failed_at": str(int(time.time()))
            })
    
    def get_job_status(self, job_id: str) -> dict:
        return self.r.hgetall(f"{self.STATUS_PREFIX}{job_id}")
    
    def get_queue_sizes(self) -> dict:
        return {p.value: self.r.llen(k) for p, k in self.queues.items()}

# ทดสอบ
jq = JobQueue(r, "app_jobs")

# เพิ่ม jobs
id1 = jq.enqueue("send_email", {"to": "user@example.com", "subject": "Welcome"}, Priority.NORMAL)
id2 = jq.enqueue("process_payment", {"order_id": 123, "amount": 99.99}, Priority.HIGH)
id3 = jq.enqueue("generate_report", {"type": "monthly"}, Priority.LOW)
id4 = jq.enqueue("send_sms", {"phone": "+66812345678", "msg": "Order shipped"}, Priority.HIGH)

print("Queue sizes:", jq.get_queue_sizes())

# Worker loop
print("\nProcessing jobs:")
for _ in range(4):
    job = jq.dequeue()
    if job:
        print(f"Processing {job['type']} (ID: {job['id']})")
        # จำลองการทำงาน
        if job['type'] == 'process_payment':
            jq.complete_job(job['id'])
        else:
            jq.complete_job(job['id'])

print("\nJob statuses:")
for job_id in [id1, id2, id3, id4]:
    status = jq.get_job_status(job_id)
    print(f"  Job {job_id}: {status.get('status', 'unknown')}")
```

### แบบฝึกหัดที่ 3: Autocomplete ด้วย Sorted Set

**โจทย์**: สร้าง Autocomplete system ที่ค้นหาคำ prefix ได้

```python
# เฉลย
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class AutocompleteSystem:
    """
    Autocomplete ด้วย Redis Sorted Set
    เทคนิค: เก็บทุก prefix ของแต่ละคำใน Sorted Set
    """
    
    def __init__(self, redis_client, index_name: str):
        self.r = redis_client
        self.key = f"autocomplete:{index_name}"
    
    def add_term(self, term: str, score: float = 0):
        """เพิ่มคำใหม่ พร้อม score (ใช้สำหรับ popularity ranking)"""
        term_lower = term.lower()
        
        # เพิ่ม prefix ทุกความยาวที่เป็นไปได้
        for i in range(1, len(term_lower) + 1):
            prefix = term_lower[:i]
            self.r.zadd(self.key, {prefix: 0})
        
        # เพิ่มคำเต็มพร้อม marker (ใช้ ASCII 255 เป็นตัวคั่น)
        self.r.zadd(self.key, {term_lower + "\xff": score})
    
    def search(self, prefix: str, max_results: int = 10) -> list:
        """ค้นหาคำที่ขึ้นต้นด้วย prefix"""
        prefix_lower = prefix.lower()
        
        # หา rank ของ prefix
        rank = self.r.zrank(self.key, prefix_lower)
        if rank is None:
            return []
        
        results = []
        range_start = rank
        
        while len(results) < max_results:
            entries = self.r.zrange(self.key, range_start, range_start + 20)
            if not entries:
                break
            
            for entry in entries:
                if not entry.startswith(prefix_lower):
                    return results
                
                # ถ้าเป็นคำเต็ม (ลงท้ายด้วย \xff)
                if entry.endswith("\xff"):
                    word = entry[:-1]  # ลบ marker ออก
                    results.append(word)
                    if len(results) >= max_results:
                        break
            
            range_start += len(entries)
        
        return results[:max_results]
    
    def increment_popularity(self, term: str):
        """เพิ่ม popularity ของคำ"""
        self.r.zincrby(self.key, 1, term.lower() + "\xff")

# ทดสอบ
ac = AutocompleteSystem(r, "products")

# เพิ่มคำ
terms = ["python", "pytorch", "pandas", "pydantic", "postgresql",
         "redis", "react", "ruby", "rust", "rails",
         "javascript", "java", "julia", "jenkins"]

for term in terms:
    ac.add_term(term)

# ค้นหา
print("Autocomplete 'py':", ac.search("py"))
print("Autocomplete 're':", ac.search("re"))
print("Autocomplete 'ja':", ac.search("ja"))
```

### แบบฝึกหัดที่ 4: Event Sourcing ด้วย Redis Streams

```python
# เฉลย - Redis Streams สำหรับ Event Sourcing
import redis
import json
import time

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class EventStore:
    """Event Store ด้วย Redis Streams"""
    
    def __init__(self, redis_client, stream_name: str):
        self.r = redis_client
        self.stream = stream_name
    
    def append_event(self, event_type: str, data: dict) -> str:
        """เพิ่ม event ใหม่"""
        event = {
            "type": event_type,
            "data": json.dumps(data),
            "timestamp": str(int(time.time()))
        }
        # XADD - เพิ่มใน stream, Redis สร้าง ID อัตโนมัติ
        event_id = self.r.xadd(self.stream, event)
        return event_id
    
    def get_events(self, from_id: str = "0", count: int = 100) -> list:
        """ดึง events ตั้งแต่ ID ที่กำหนด"""
        entries = self.r.xrange(self.stream, min=from_id, count=count)
        return [
            {
                "id": entry_id,
                "type": data["type"],
                "data": json.loads(data["data"]),
                "timestamp": int(data["timestamp"])
            }
            for entry_id, data in entries
        ]
    
    def get_latest_events(self, count: int = 10) -> list:
        """ดึง events ล่าสุด"""
        entries = self.r.xrevrange(self.stream, count=count)
        return [
            {
                "id": entry_id,
                "type": data["type"],
                "data": json.loads(data["data"])
            }
            for entry_id, data in entries
        ]
    
    def get_event_count(self) -> int:
        return self.r.xlen(self.stream)

# ทดสอบ Event Store
event_store = EventStore(r, "order_events")

# สร้าง order events
e1 = event_store.append_event("order.created", {"order_id": 1, "customer": "Alice", "total": 150.0})
e2 = event_store.append_event("order.payment_received", {"order_id": 1, "amount": 150.0})
e3 = event_store.append_event("order.processing", {"order_id": 1, "warehouse": "Bangkok"})
e4 = event_store.append_event("order.shipped", {"order_id": 1, "tracking": "TH123456"})
e5 = event_store.append_event("order.delivered", {"order_id": 1, "delivered_at": "2024-01-15"})

print(f"Total events: {event_store.get_event_count()}")
print("\nAll events:")
for event in event_store.get_events():
    print(f"  [{event['type']}] {event['data']}")
```

### แบบฝึกหัดที่ 5-8: (สรุปหัวข้อและ hints)

**แบบฝึกหัดที่ 5: Bloom Filter** - ใช้ Redis Bitmap สร้าง Bloom Filter สำหรับตรวจสอบ email ซ้ำ

**แบบฝึกหัดที่ 6: Circuit Breaker** - สร้าง Circuit Breaker pattern ด้วย Redis สำหรับ external API calls

**แบบฝึกหัดที่ 7: Geospatial** - ใช้ Redis GEO commands หา restaurants ใกล้เคียง

**แบบฝึกหัดที่ 8: Real-time Analytics** - HyperLogLog สำหรับนับ unique visitors โดยประมาณ

```python
# แบบฝึกหัดที่ 8: HyperLogLog สำหรับ Unique Visitors
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class UniqueVisitorCounter:
    """นับ unique visitors ด้วย HyperLogLog"""
    
    def __init__(self, redis_client):
        self.r = redis_client
    
    def track_visit(self, page: str, user_id: str):
        key = f"unique_visitors:{page}"
        self.r.pfadd(key, user_id)
    
    def get_unique_count(self, page: str) -> int:
        return self.r.pfcount(f"unique_visitors:{page}")
    
    def get_combined_count(self, *pages: str) -> int:
        keys = [f"unique_visitors:{page}" for page in pages]
        return self.r.pfcount(*keys)

counter = UniqueVisitorCounter(r)

# จำลองการเข้าชม
import random
users = [f"user_{i}" for i in range(1000)]

for _ in range(5000):
    page = random.choice(["/home", "/products", "/about"])
    user = random.choice(users)
    counter.track_visit(page, user)

print(f"Unique visitors /home: ~{counter.get_unique_count('/home')}")
print(f"Unique visitors /products: ~{counter.get_unique_count('/products')}")
print(f"Total unique across all pages: ~{counter.get_combined_count('/home', '/products', '/about')}")
print("(HyperLogLog มี error rate ~0.81%)")
```

---

## สรุป

Redis เป็นเครื่องมือที่ทรงพลังสำหรับการพัฒนา application ที่ต้องการความเร็วสูง key concepts ที่ควรจำ:

| Pattern | ใช้เมื่อ | ข้อควรระวัง |
|---------|----------|-------------|
| Cache-Aside | Read-heavy workloads | Cache stampede |
| Write-Through | Consistency สำคัญ | Write latency เพิ่ม |
| Write-Behind | Write-heavy, eventual consistency OK | Data loss risk |
| Pub/Sub | Real-time notifications | Message persistence |
| Distributed Lock | Exclusive access needed | Lock expiration |
| Rate Limiting | API protection | Window boundary issues |

### Best Practices

1. **ตั้ง TTL เสมอ** สำหรับ cached data เพื่อป้องกัน memory leak
2. **ใช้ ConnectionPool** แทนการสร้าง connection ใหม่ทุกครั้ง
3. **ใช้ Pipeline** เมื่อต้องทำ operations หลายอย่างพร้อมกัน
4. **ใช้ SCAN แทน KEYS** ในการ production เพราะ KEYS จะ block Redis
5. **ใช้ Lua scripts** สำหรับ atomic operations ที่ซับซ้อน
6. **Monitor memory usage** และตั้ง `maxmemory-policy`

```python
# Pipeline example - ทำหลาย operations พร้อมกันอย่างมีประสิทธิภาพ
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# แทนที่จะทำทีละ operation
with r.pipeline() as pipe:
    for i in range(100):
        pipe.set(f"key:{i}", f"value:{i}", ex=3600)
    results = pipe.execute()

print(f"Pipeline executed {len(results)} commands")

# Transaction
with r.pipeline() as pipe:
    while True:
        try:
            pipe.watch("account:balance")  # Watch key สำหรับ optimistic locking
            balance = float(pipe.get("account:balance") or "0")
            
            pipe.multi()  # เริ่ม transaction
            pipe.set("account:balance", str(balance - 100))
            pipe.execute()  # Execute atomically
            break
        except redis.WatchError:
            # ค่าเปลี่ยนระหว่างรอ - retry
            continue

print("Transaction completed")
```

---

## แหล่งเรียนรู้เพิ่มเติม

- [Redis Documentation](https://redis.io/docs/)
- [redis-py Documentation](https://redis-py.readthedocs.io/)
- [Redis University](https://university.redis.com/)
- [Redis Design Patterns](https://redis.com/redis-best-practices/)
