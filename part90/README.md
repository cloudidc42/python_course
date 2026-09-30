# Part 90: Message Queues - RabbitMQ & Apache Kafka

## บทนำ

Message Queue คือระบบ middleware ที่ช่วยให้ applications สื่อสารกันแบบ asynchronous โดยที่ producer ส่งข้อความไปยัง queue และ consumer รับข้อความออกมาประมวลผล ทำให้ระบบมี decoupling และ resilience สูง

---

## 1. Message Queue Concepts

### ทำไมต้องใช้ Message Queue?

| ปัญหา | แก้ด้วย Message Queue |
|-------|----------------------|
| Tight coupling | Services ไม่รู้จักกัน communicate ผ่าน queue |
| Synchronous bottlenecks | Process async แทน sync |
| Load spikes | Buffer messages ระหว่าง producer/consumer |
| Service downtime | Messages ไม่หายเมื่อ consumer down |
| Different speeds | Consumer ประมวลผลได้เร็วหรือช้าตามที่ต้องการ |

### Key Concepts

```python
# ตัวอย่าง 1: Basic Queue Concept (Python Simulation)
from queue import Queue
import threading
import time
import uuid
from dataclasses import dataclass, field
from typing import Optional, Callable, Any

@dataclass
class Message:
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    body: Any = None
    headers: dict = field(default_factory=dict)
    timestamp: float = field(default_factory=time.time)
    retry_count: int = 0

class SimpleQueue:
    """จำลอง Message Queue พื้นฐาน"""
    
    def __init__(self, name: str, max_size: int = 0):
        self.name = name
        self._queue = Queue(maxsize=max_size)
        self._processed = 0
    
    def publish(self, message: Message) -> bool:
        """Producer ส่ง message เข้า queue"""
        try:
            self._queue.put(message, block=False)
            print(f"[{self.name}] Published: {message.id}")
            return True
        except Exception:
            print(f"[{self.name}] Queue full!")
            return False
    
    def consume(self, timeout: float = 1.0) -> Optional[Message]:
        """Consumer รับ message จาก queue"""
        try:
            message = self._queue.get(timeout=timeout)
            self._processed += 1
            return message
        except Exception:
            return None
    
    def acknowledge(self, message: Message):
        """ยืนยันว่าประมวลผล message เสร็จแล้ว"""
        self._queue.task_done()
        print(f"[{self.name}] ACK: {message.id}")
    
    @property
    def size(self) -> int:
        return self._queue.qsize()
    
    @property
    def stats(self) -> dict:
        return {
            'queue': self.name,
            'pending': self.size,
            'processed': self._processed
        }

# ทดสอบ Basic Queue
queue = SimpleQueue("email-notifications")

# Producer thread
def producer():
    for i in range(5):
        msg = Message(
            body={'to': f'user{i}@example.com', 'subject': f'Notification {i}'},
            headers={'priority': 'high' if i % 2 == 0 else 'normal'}
        )
        queue.publish(msg)
        time.sleep(0.1)

# Consumer thread
results = []
def consumer():
    while True:
        msg = queue.consume(timeout=2.0)
        if msg is None:
            break
        print(f"  Processing: {msg.body['to']}")
        time.sleep(0.2)  # จำลองการประมวลผล
        queue.acknowledge(msg)
        results.append(msg)

p = threading.Thread(target=producer)
c = threading.Thread(target=consumer)

p.start()
c.start()
p.join()
c.join()

print(f"\nStats: {queue.stats}")
print(f"Processed: {len(results)} messages")
```

---

## 2. RabbitMQ Architecture

### 2.1 Core Components

```python
# ตัวอย่าง 2: RabbitMQ Architecture Simulation

# RabbitMQ Components:
# - Producer: ส่ง messages
# - Exchange: รับ messages และ route ไปยัง queues
# - Queue: เก็บ messages
# - Consumer: รับ messages จาก queue
# - Binding: เชื่อม Exchange กับ Queue

from abc import ABC, abstractmethod
from typing import Dict, List, Set
from dataclasses import dataclass, field
import fnmatch

@dataclass
class RMQMessage:
    """RabbitMQ Message"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    body: Any = None
    routing_key: str = ""
    exchange: str = ""
    headers: dict = field(default_factory=dict)
    persistent: bool = True
    timestamp: float = field(default_factory=time.time)
    
    def __repr__(self):
        return f"Message(id={self.id[:8]}, key={self.routing_key})"

class RMQQueue:
    """RabbitMQ Queue"""
    
    def __init__(self, name: str, durable: bool = True, auto_delete: bool = False):
        self.name = name
        self.durable = durable
        self.auto_delete = auto_delete
        self._messages: List[RMQMessage] = []
        self._unacked: Dict[str, RMQMessage] = {}
    
    def enqueue(self, message: RMQMessage):
        self._messages.append(message)
    
    def dequeue(self) -> Optional[RMQMessage]:
        if not self._messages:
            return None
        msg = self._messages.pop(0)
        self._unacked[msg.id] = msg
        return msg
    
    def ack(self, message_id: str) -> bool:
        return self._unacked.pop(message_id, None) is not None
    
    def nack(self, message_id: str, requeue: bool = True):
        msg = self._unacked.pop(message_id, None)
        if msg and requeue:
            self._messages.insert(0, msg)
    
    @property
    def message_count(self) -> int:
        return len(self._messages)
    
    @property
    def consumer_count(self) -> int:
        return 0  # simplified

class Exchange(ABC):
    def __init__(self, name: str):
        self.name = name
        self._bindings: List[tuple] = []  # [(queue, routing_key)]
    
    def bind(self, queue: RMQQueue, routing_key: str = ""):
        self._bindings.append((queue, routing_key))
    
    @abstractmethod
    def route(self, message: RMQMessage) -> List[RMQQueue]:
        pass
    
    def publish(self, message: RMQMessage):
        targets = self.route(message)
        for queue in targets:
            queue.enqueue(message)
        print(f"[Exchange:{self.name}] Routed to {len(targets)} queue(s)")
        return len(targets)

class DirectExchange(Exchange):
    """Direct Exchange: routing_key ต้องตรงกันทุกตัว"""
    
    def route(self, message: RMQMessage) -> List[RMQQueue]:
        return [
            queue for queue, key in self._bindings
            if key == message.routing_key
        ]

class FanoutExchange(Exchange):
    """Fanout Exchange: ส่งไปทุก queues"""
    
    def route(self, message: RMQMessage) -> List[RMQQueue]:
        return [queue for queue, _ in self._bindings]

class TopicExchange(Exchange):
    """Topic Exchange: routing_key ใช้ wildcard (* และ #)"""
    
    def route(self, message: RMQMessage) -> List[RMQQueue]:
        matched = []
        for queue, pattern in self._bindings:
            if self._matches(pattern, message.routing_key):
                matched.append(queue)
        return matched
    
    def _matches(self, pattern: str, routing_key: str) -> bool:
        """
        * matches exactly one word
        # matches zero or more words
        """
        # แปลง # เป็น * สำหรับ fnmatch
        regex_pattern = pattern.replace('#', '*').replace('*.*', '*')
        # ใช้ simple matching
        pattern_parts = pattern.split('.')
        key_parts = routing_key.split('.')
        
        return self._match_parts(pattern_parts, key_parts)
    
    def _match_parts(self, pattern_parts: list, key_parts: list) -> bool:
        if not pattern_parts and not key_parts:
            return True
        if not pattern_parts:
            return False
        if pattern_parts[0] == '#':
            # # matches any remaining
            return True
        if not key_parts:
            return pattern_parts[0] == '#'
        if pattern_parts[0] == '*' or pattern_parts[0] == key_parts[0]:
            return self._match_parts(pattern_parts[1:], key_parts[1:])
        return False

class HeadersExchange(Exchange):
    """Headers Exchange: match ตาม message headers"""
    
    def route(self, message: RMQMessage) -> List[RMQQueue]:
        matched = []
        for queue, binding_headers in self._bindings:
            if isinstance(binding_headers, dict):
                x_match = binding_headers.get('x-match', 'all')
                filter_headers = {k: v for k, v in binding_headers.items() if k != 'x-match'}
                
                if x_match == 'any':
                    matches = any(message.headers.get(k) == v for k, v in filter_headers.items())
                else:
                    matches = all(message.headers.get(k) == v for k, v in filter_headers.items())
                
                if matches:
                    matched.append(queue)
        return matched

# ทดสอบ Exchanges
print("=== RabbitMQ Exchange Types ===")

# 1. Direct Exchange
print("\n--- Direct Exchange ---")
direct_ex = DirectExchange("direct_logs")
error_queue = RMQQueue("errors")
warning_queue = RMQQueue("warnings")
info_queue = RMQQueue("info")

direct_ex.bind(error_queue, "error")
direct_ex.bind(warning_queue, "warning")
direct_ex.bind(info_queue, "info")

for level in ["error", "warning", "info", "debug"]:
    direct_ex.publish(RMQMessage(body=f"Log message", routing_key=level))

print(f"Errors: {error_queue.message_count}")
print(f"Warnings: {warning_queue.message_count}")
print(f"Info: {info_queue.message_count}")

# 2. Fanout Exchange
print("\n--- Fanout Exchange ---")
fanout_ex = FanoutExchange("notifications")
email_q = RMQQueue("email")
sms_q = RMQQueue("sms")
push_q = RMQQueue("push")

for q in [email_q, sms_q, push_q]:
    fanout_ex.bind(q)

fanout_ex.publish(RMQMessage(body="New order placed!", routing_key=""))
print(f"Email: {email_q.message_count}, SMS: {sms_q.message_count}, Push: {push_q.message_count}")

# 3. Topic Exchange
print("\n--- Topic Exchange ---")
topic_ex = TopicExchange("topic_logs")

all_errors_q = RMQQueue("all_errors")
web_logs_q = RMQQueue("web_logs")
critical_q = RMQQueue("critical")

topic_ex.bind(all_errors_q, "*.*.error")
topic_ex.bind(web_logs_q, "web.#")
topic_ex.bind(critical_q, "#.critical")

messages = [
    ("web.ui.error", "UI error"),
    ("web.api.info", "API info"),
    ("db.query.error", "DB error"),
    ("web.payment.critical", "Payment critical!"),
]

for routing_key, body in messages:
    topic_ex.publish(RMQMessage(body=body, routing_key=routing_key))

print(f"All Errors: {all_errors_q.message_count}")
print(f"Web Logs: {web_logs_q.message_count}")
print(f"Critical: {critical_q.message_count}")
```

---

## 3. pika Library (RabbitMQ Python Client)

```python
# ตัวอย่าง 3: pika Basic Usage
# ต้องติดตั้ง: pip install pika

PIKA_PRODUCER_CODE = '''
import pika
import json

# Connection
connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        port=5672,
        virtual_host='/',
        credentials=pika.PlainCredentials('guest', 'guest')
    )
)
channel = connection.channel()

# Declare queue (idempotent - สร้างใหม่หรือใช้ที่มีอยู่)
channel.queue_declare(queue='task_queue', durable=True)

# Send message
message = json.dumps({
    'task': 'send_email',
    'to': 'alice@example.com',
    'subject': 'Hello',
    'body': 'Welcome!'
})

channel.basic_publish(
    exchange='',                    # default exchange
    routing_key='task_queue',
    body=message,
    properties=pika.BasicProperties(
        delivery_mode=pika.spec.PERSISTENT_DELIVERY_MODE,  # persistent message
        content_type='application/json',
        message_id=str(uuid.uuid4()),
        timestamp=int(time.time())
    )
)

print(f"Sent: {message}")
connection.close()
'''

PIKA_CONSUMER_CODE = '''
import pika
import json
import time

def callback(ch, method, properties, body):
    """Callback เมื่อได้รับ message"""
    message = json.loads(body)
    print(f"Received: {message}")
    
    # ประมวลผล
    time.sleep(1)  # จำลองงาน
    print(f"Task completed: {message['task']}")
    
    # ACK - ยืนยันว่าประมวลผลเสร็จ
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost')
)
channel = connection.channel()
channel.queue_declare(queue='task_queue', durable=True)

# รับครั้งละ 1 message ก่อน ACK
channel.basic_qos(prefetch_count=1)
channel.basic_consume(
    queue='task_queue',
    on_message_callback=callback
)

print("Waiting for messages...")
channel.start_consuming()
'''

print("pika Producer Code:")
print(PIKA_PRODUCER_CODE)
print("\npika Consumer Code:")
print(PIKA_CONSUMER_CODE)

# จำลอง pika โดยใช้ simulation
class PikaSimulation:
    """จำลองการทำงาน pika สำหรับ demo"""
    
    def __init__(self):
        self.queues: Dict[str, RMQQueue] = {}
        self.exchanges: Dict[str, Exchange] = {
            '': FanoutExchange('')  # default exchange
        }
    
    def queue_declare(self, queue: str, durable: bool = False):
        if queue not in self.queues:
            self.queues[queue] = RMQQueue(queue, durable)
        return queue
    
    def exchange_declare(self, exchange: str, exchange_type: str = 'direct'):
        if exchange not in self.exchanges:
            if exchange_type == 'fanout':
                self.exchanges[exchange] = FanoutExchange(exchange)
            elif exchange_type == 'topic':
                self.exchanges[exchange] = TopicExchange(exchange)
            else:
                self.exchanges[exchange] = DirectExchange(exchange)
    
    def queue_bind(self, queue: str, exchange: str, routing_key: str = ''):
        if queue in self.queues and exchange in self.exchanges:
            self.exchanges[exchange].bind(self.queues[queue], routing_key)
    
    def basic_publish(self, exchange: str, routing_key: str, body: Any, **kwargs):
        msg = RMQMessage(body=body, routing_key=routing_key, exchange=exchange)
        
        if exchange == '':
            # Default exchange - ส่งตรงไปยัง queue ที่ชื่อตรงกัน
            if routing_key in self.queues:
                self.queues[routing_key].enqueue(msg)
                print(f"Published to queue '{routing_key}': {body}")
        elif exchange in self.exchanges:
            self.exchanges[exchange].publish(msg)
    
    def basic_consume(self, queue: str, callback: Callable, auto_ack: bool = False):
        q = self.queues.get(queue)
        if not q:
            print(f"Queue '{queue}' not found")
            return
        
        print(f"Starting consumer for '{queue}'...")
        while q.message_count > 0:
            msg = q.dequeue()
            if msg:
                try:
                    callback(msg)
                    if auto_ack:
                        q.ack(msg.id)
                except Exception as e:
                    print(f"Error processing: {e}")
                    q.nack(msg.id, requeue=True)

# ทดสอบ simulation
channel = PikaSimulation()
channel.queue_declare('task_queue', durable=True)

# Publish messages
tasks = [
    {'task': 'send_email', 'to': 'alice@example.com'},
    {'task': 'process_image', 'image_id': 'IMG-001'},
    {'task': 'generate_report', 'report_type': 'monthly'},
]

import json
for task in tasks:
    channel.basic_publish('', 'task_queue', json.dumps(task))

# Consume
def process_task(msg: RMQMessage):
    task = json.loads(msg.body)
    print(f"Processing: {task['task']}")
    channel.queues['task_queue'].ack(msg.id)

channel.basic_consume('task_queue', process_task)
```

---

## 4. Message Patterns

### 4.1 Work Queue Pattern

```python
# ตัวอย่าง 4: Work Queue (Task Distribution)
from threading import Thread
import time
import random

class WorkQueue:
    """Work Queue - กระจายงานให้ workers หลาย instance"""
    
    def __init__(self, name: str):
        self._queue = RMQQueue(name, durable=True)
        self._workers = []
    
    def submit_task(self, task: dict):
        msg = RMQMessage(body=task, routing_key=self._queue.name)
        self._queue.enqueue(msg)
        print(f"[Queue] Task submitted: {task.get('type', 'unknown')}")
    
    def add_worker(self, worker_id: str, process_fn: Callable):
        self._workers.append((worker_id, process_fn))
    
    def process_all(self):
        """จำลองการ distribute งานให้ workers"""
        round_robin = 0
        while self._queue.message_count > 0:
            if not self._workers:
                break
            worker_id, process_fn = self._workers[round_robin % len(self._workers)]
            msg = self._queue.dequeue()
            if msg:
                print(f"[{worker_id}] Processing: {msg.body}")
                try:
                    process_fn(msg.body)
                    self._queue.ack(msg.id)
                    print(f"[{worker_id}] Done: {msg.body.get('id', '')}")
                except Exception as e:
                    print(f"[{worker_id}] Failed: {e}")
                    self._queue.nack(msg.id, requeue=True)
            round_robin += 1

def email_worker(task: dict):
    time.sleep(0.1)  # จำลองการส่ง email
    print(f"  → Email sent to {task.get('recipient')}")

# ทดสอบ Work Queue
work_queue = WorkQueue("email_tasks")

for i in range(6):
    work_queue.submit_task({
        'id': f'TASK-{i:03d}',
        'type': 'send_email',
        'recipient': f'user{i}@example.com',
        'subject': f'Notification {i}'
    })

print(f"Queue size: {work_queue._queue.message_count}")

work_queue.add_worker("Worker-1", email_worker)
work_queue.add_worker("Worker-2", email_worker)
work_queue.add_worker("Worker-3", email_worker)

print("\nProcessing tasks:")
work_queue.process_all()
print(f"Queue size after: {work_queue._queue.message_count}")
```

### 4.2 Publish/Subscribe Pattern

```python
# ตัวอย่าง 5: Pub/Sub Pattern
from typing import Dict, List, Callable

class PubSubSystem:
    """Pub/Sub ด้วย Fanout Exchange"""
    
    def __init__(self):
        self._exchange = FanoutExchange("events")
        self._subscriber_queues: Dict[str, RMQQueue] = {}
        self._handlers: Dict[str, Callable] = {}
    
    def subscribe(self, subscriber_id: str, handler: Callable) -> str:
        """Subscribe รับ messages ทั้งหมด"""
        queue = RMQQueue(f"sub_{subscriber_id}", auto_delete=True)
        self._subscriber_queues[subscriber_id] = queue
        self._handlers[subscriber_id] = handler
        self._exchange.bind(queue)
        print(f"[PubSub] {subscriber_id} subscribed")
        return subscriber_id
    
    def unsubscribe(self, subscriber_id: str):
        if subscriber_id in self._subscriber_queues:
            del self._subscriber_queues[subscriber_id]
            del self._handlers[subscriber_id]
    
    def publish(self, event_type: str, data: dict):
        """Publish event ไปยัง subscribers ทั้งหมด"""
        msg = RMQMessage(
            body={'type': event_type, 'data': data},
            routing_key=event_type
        )
        count = self._exchange.publish(msg)
        print(f"[PubSub] Published '{event_type}' to {count} subscribers")
    
    def dispatch_all(self):
        """ส่ง messages ให้ handlers"""
        for sub_id, queue in self._subscriber_queues.items():
            handler = self._handlers[sub_id]
            while queue.message_count > 0:
                msg = queue.dequeue()
                if msg:
                    handler(msg.body)
                    queue.ack(msg.id)

# ทดสอบ Pub/Sub
pub_sub = PubSubSystem()

def email_handler(event: dict):
    print(f"  [EmailService] Got {event['type']}: {event['data']}")

def analytics_handler(event: dict):
    print(f"  [Analytics] Tracking {event['type']}: {event['data']}")

def audit_handler(event: dict):
    print(f"  [Audit] Logging {event['type']}: {event['data']}")

pub_sub.subscribe("email-service", email_handler)
pub_sub.subscribe("analytics", analytics_handler)
pub_sub.subscribe("audit-log", audit_handler)

print("\n=== Pub/Sub Demo ===")
pub_sub.publish("user.registered", {'user_id': 'U001', 'email': 'alice@example.com'})
pub_sub.publish("order.placed", {'order_id': 'O001', 'total': 599.0})

print("\nDispatching to handlers:")
pub_sub.dispatch_all()
```

### 4.3 Routing Pattern

```python
# ตัวอย่าง 6: Routing Pattern
class RoutingSystem:
    """Direct Exchange Routing"""
    
    def __init__(self, exchange_name: str):
        self._exchange = DirectExchange(exchange_name)
        self._queues: Dict[str, RMQQueue] = {}
    
    def add_queue(self, name: str, *routing_keys: str) -> RMQQueue:
        queue = RMQQueue(name, durable=True)
        self._queues[name] = queue
        for key in routing_keys:
            self._exchange.bind(queue, key)
            print(f"[Routing] Bound '{name}' to key '{key}'")
        return queue
    
    def route(self, routing_key: str, data: dict):
        msg = RMQMessage(body=data, routing_key=routing_key)
        count = self._exchange.publish(msg)
        if count == 0:
            print(f"[Routing] No handler for key '{routing_key}'")
    
    def consume_from(self, queue_name: str, handler: Callable):
        queue = self._queues.get(queue_name)
        if not queue:
            return
        while queue.message_count > 0:
            msg = queue.dequeue()
            if msg:
                handler(msg.body)
                queue.ack(msg.id)

# ทดสอบ Routing
print("\n=== Routing Pattern Demo ===")
router = RoutingSystem("direct_logs")

# Subscribe with specific routing keys
router.add_queue("critical_handler", "critical", "error")
router.add_queue("warning_handler", "warning")
router.add_queue("all_logs", "debug", "info", "warning", "error", "critical")

# Publish messages
log_messages = [
    ("info", {"message": "Server started", "timestamp": "2024-01-01 10:00:00"}),
    ("warning", {"message": "High memory usage: 85%", "component": "cache"}),
    ("error", {"message": "Database connection lost", "component": "db"}),
    ("critical", {"message": "Service unavailable", "impact": "all users"}),
    ("debug", {"message": "Request received", "path": "/api/users"}),
]

for level, data in log_messages:
    router.route(level, {**data, 'level': level})

print("\nConsuming critical messages:")
def critical_handler(msg):
    print(f"  🚨 CRITICAL: {msg.get('message')} [{msg.get('component', 'system')}]")

router.consume_from("critical_handler", critical_handler)

print("\nWarning count:", router._queues['warning_handler'].message_count)
print("All logs count:", router._queues['all_logs'].message_count)
```

### 4.4 Message Durability

```python
# ตัวอย่าง 7: Message Durability and Dead Letter Queue
from typing import Optional
from datetime import datetime, timedelta

@dataclass
class DurableMessage(RMQMessage):
    max_retries: int = 3
    ttl_seconds: Optional[float] = None
    
    def is_expired(self) -> bool:
        if self.ttl_seconds is None:
            return False
        return time.time() - self.timestamp > self.ttl_seconds

class DeadLetterQueue:
    """Dead Letter Queue สำหรับ messages ที่ process ไม่ได้"""
    
    def __init__(self):
        self._messages: List[dict] = []
    
    def add(self, message: RMQMessage, reason: str):
        self._messages.append({
            'message': message,
            'reason': reason,
            'dead_at': datetime.now().isoformat()
        })
        print(f"[DLQ] Message {message.id[:8]} sent to DLQ: {reason}")
    
    def count(self) -> int:
        return len(self._messages)
    
    def get_all(self) -> List[dict]:
        return self._messages.copy()

class DurableQueue:
    """Queue พร้อม retry logic และ DLQ"""
    
    def __init__(self, name: str, max_retries: int = 3):
        self.name = name
        self.max_retries = max_retries
        self._messages: List[DurableMessage] = []
        self._dlq = DeadLetterQueue()
    
    def publish(self, body: Any, ttl_seconds: Optional[float] = None):
        msg = DurableMessage(
            body=body,
            max_retries=self.max_retries,
            ttl_seconds=ttl_seconds
        )
        self._messages.append(msg)
    
    def consume(self, handler: Callable) -> int:
        """Returns number of successfully processed messages"""
        processed = 0
        messages = self._messages.copy()
        self._messages.clear()
        
        for msg in messages:
            # Check TTL
            if msg.is_expired():
                self._dlq.add(msg, f"TTL expired after {msg.ttl_seconds}s")
                continue
            
            # Try to process
            success = False
            for attempt in range(msg.max_retries + 1):
                try:
                    handler(msg.body)
                    processed += 1
                    success = True
                    break
                except Exception as e:
                    if attempt == msg.max_retries:
                        self._dlq.add(msg, f"Max retries ({msg.max_retries}) exceeded: {e}")
                    else:
                        print(f"  Retry {attempt+1}: {e}")
                        time.sleep(0.01 * (2 ** attempt))
        
        return processed
    
    @property
    def dlq_count(self) -> int:
        return self._dlq.count()
    
    def get_dlq_messages(self) -> list:
        return self._dlq.get_all()

# ทดสอบ
print("\n=== Durable Queue with DLQ ===")
queue = DurableQueue("payments", max_retries=3)

# Publish messages
queue.publish({'amount': 99.99, 'currency': 'THB', 'order_id': 'O001'})
queue.publish({'amount': 0, 'currency': 'THB', 'order_id': 'O002'})  # Will fail
queue.publish({'amount': 150.0, 'currency': 'THB', 'order_id': 'O003'}, ttl_seconds=0.001)  # Immediate TTL

time.sleep(0.01)  # Wait for TTL to expire

fail_count = [0]
def payment_processor(payment: dict):
    if payment['amount'] <= 0:
        fail_count[0] += 1
        raise ValueError(f"Invalid amount: {payment['amount']}")
    print(f"  ✅ Payment processed: ${payment['amount']} for {payment['order_id']}")

processed = queue.consume(payment_processor)
print(f"\nProcessed: {processed}, DLQ: {queue.dlq_count}")
for item in queue.get_dlq_messages():
    print(f"  DLQ: Order {item['message'].body.get('order_id')} - {item['reason']}")
```

---

## 5. Apache Kafka Concepts

### 5.1 Kafka Architecture

```python
# ตัวอย่าง 8: Kafka Core Concepts

# Kafka Components:
# - Producer: ส่ง messages
# - Topic: Category/feed name (เหมือน channel)
# - Partition: แบ่ง topic ออกเป็นส่วนๆ (parallel processing)
# - Broker: Kafka server
# - Consumer: อ่าน messages
# - Consumer Group: กลุ่ม consumers ที่ share กัน
# - Offset: ตำแหน่งใน partition

from dataclasses import dataclass, field
from typing import Dict, List, Optional, Tuple
import time
import json
import uuid

@dataclass
class KafkaRecord:
    """Kafka Message Record"""
    key: Optional[str]
    value: Any
    topic: str
    partition: int = 0
    offset: int = 0
    timestamp: float = field(default_factory=time.time)
    headers: Dict[str, str] = field(default_factory=dict)
    
    def __repr__(self):
        return f"Record(topic={self.topic}, partition={self.partition}, offset={self.offset})"

class KafkaPartition:
    """Partition ใน Kafka Topic"""
    
    def __init__(self, topic: str, partition_id: int):
        self.topic = topic
        self.partition_id = partition_id
        self._records: List[KafkaRecord] = []
        self._current_offset = 0
    
    def append(self, record: KafkaRecord) -> int:
        record.partition = self.partition_id
        record.offset = self._current_offset
        self._records.append(record)
        self._current_offset += 1
        return record.offset
    
    def read(self, from_offset: int, max_records: int = 100) -> List[KafkaRecord]:
        return self._records[from_offset:from_offset + max_records]
    
    @property
    def latest_offset(self) -> int:
        return self._current_offset
    
    @property
    def size(self) -> int:
        return len(self._records)

class KafkaTopic:
    """Kafka Topic"""
    
    def __init__(self, name: str, num_partitions: int = 3, replication_factor: int = 1):
        self.name = name
        self.num_partitions = num_partitions
        self._partitions = [
            KafkaPartition(name, i) for i in range(num_partitions)
        ]
    
    def get_partition(self, key: Optional[str] = None, partition: Optional[int] = None) -> KafkaPartition:
        if partition is not None:
            return self._partitions[partition % self.num_partitions]
        if key is not None:
            # Hash partitioning
            p = hash(key) % self.num_partitions
            return self._partitions[p]
        # Round robin (simplified)
        return self._partitions[0]
    
    def all_partitions(self) -> List[KafkaPartition]:
        return self._partitions
    
    def total_messages(self) -> int:
        return sum(p.size for p in self._partitions)

class KafkaBroker:
    """Simple Kafka Broker Simulation"""
    
    def __init__(self, broker_id: str = "broker-1"):
        self.broker_id = broker_id
        self._topics: Dict[str, KafkaTopic] = {}
        self._consumer_offsets: Dict[str, Dict[str, int]] = {}
    
    def create_topic(self, name: str, partitions: int = 3) -> KafkaTopic:
        if name not in self._topics:
            self._topics[name] = KafkaTopic(name, partitions)
            print(f"[Kafka] Created topic '{name}' with {partitions} partitions")
        return self._topics[name]
    
    def get_topic(self, name: str) -> Optional[KafkaTopic]:
        return self._topics.get(name)
    
    def produce(self, topic_name: str, key: Optional[str], value: Any, 
                headers: Dict[str, str] = None) -> KafkaRecord:
        topic = self._topics.get(topic_name)
        if not topic:
            topic = self.create_topic(topic_name)
        
        partition = topic.get_partition(key)
        record = KafkaRecord(
            key=key,
            value=value,
            topic=topic_name,
            headers=headers or {}
        )
        offset = partition.append(record)
        return record
    
    def consume(
        self, 
        topic_name: str, 
        group_id: str, 
        partition_id: int = 0,
        max_records: int = 10
    ) -> List[KafkaRecord]:
        topic = self._topics.get(topic_name)
        if not topic:
            return []
        
        # Get consumer group offset
        group_key = f"{group_id}:{topic_name}:{partition_id}"
        from_offset = self._consumer_offsets.get(group_key, 0)
        
        partition = topic._partitions[partition_id]
        records = partition.read(from_offset, max_records)
        
        if records:
            # Update offset
            self._consumer_offsets[group_key] = from_offset + len(records)
        
        return records
    
    def commit_offset(self, group_id: str, topic_name: str, partition_id: int, offset: int):
        group_key = f"{group_id}:{topic_name}:{partition_id}"
        self._consumer_offsets[group_key] = offset + 1
    
    def get_offset(self, group_id: str, topic_name: str, partition_id: int) -> int:
        group_key = f"{group_id}:{topic_name}:{partition_id}"
        return self._consumer_offsets.get(group_key, 0)

# ทดสอบ Kafka
print("=== Apache Kafka Demo ===")
broker = KafkaBroker()

# Create topics
order_topic = broker.create_topic("orders", partitions=3)
payment_topic = broker.create_topic("payments", partitions=2)

# Produce messages
print("\nProducing messages:")
orders = [
    ("ORDER-001", {'customer_id': 'C001', 'total': 599.0, 'items': 2}),
    ("ORDER-002", {'customer_id': 'C002', 'total': 1299.0, 'items': 5}),
    ("ORDER-003", {'customer_id': 'C001', 'total': 299.0, 'items': 1}),
    ("ORDER-004", {'customer_id': 'C003', 'total': 899.0, 'items': 3}),
]

for key, value in orders:
    record = broker.produce("orders", key=key, value=value)
    print(f"  Produced: {record}")

print(f"\nTopic stats:")
for p in order_topic.all_partitions():
    print(f"  Partition {p.partition_id}: {p.size} records")

# Consume messages
print("\nConsuming (group-1, partition-0):")
records = broker.consume("orders", "order-processor", partition_id=0)
for record in records:
    print(f"  {record}: {record.value}")

print("\nConsuming again (same group, same partition):")
records2 = broker.consume("orders", "order-processor", partition_id=0)
print(f"  Got {len(records2)} records (should be 0 if already committed)")

print("\nConsuming (different group):")
records3 = broker.consume("orders", "analytics-service", partition_id=0)
print(f"  Got {len(records3)} records")
```

---

## 6. kafka-python Library

```python
# ตัวอย่าง 9: kafka-python Usage
# ต้องติดตั้ง: pip install kafka-python

KAFKA_PRODUCER_CODE = '''
from kafka import KafkaProducer
import json

# สร้าง Producer
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda k: k.encode('utf-8') if k else None,
    acks='all',                    # รอ acks จากทุก replicas
    retries=3,                     # retry 3 ครั้งถ้าล้มเหลว
    max_in_flight_requests_per_connection=1  # รักษาลำดับ
)

# Produce message
def send_order_event(order: dict):
    order_id = order['id']
    
    # ส่ง message
    future = producer.send(
        topic='orders',
        key=order_id,
        value=order,
        headers=[
            ('source', b'order-service'),
            ('version', b'1.0')
        ]
    )
    
    # รอผล (synchronous)
    record_metadata = future.get(timeout=10)
    print(f"Sent to: {record_metadata.topic}[{record_metadata.partition}]@{record_metadata.offset}")
    return record_metadata

# Batch produce
orders = [
    {'id': 'ORD-001', 'customer': 'Alice', 'total': 599.0},
    {'id': 'ORD-002', 'customer': 'Bob', 'total': 1299.0},
]

for order in orders:
    send_order_event(order)

producer.flush()  # ส่ง messages ที่ยังค้างอยู่
producer.close()
'''

KAFKA_CONSUMER_CODE = '''
from kafka import KafkaConsumer
from kafka.errors import KafkaError
import json

# สร้าง Consumer Group
consumer = KafkaConsumer(
    'orders',
    bootstrap_servers=['localhost:9092'],
    group_id='order-processor',
    auto_offset_reset='earliest',   # เริ่มจาก oldest message
    enable_auto_commit=False,        # manual commit
    value_deserializer=lambda v: json.loads(v.decode('utf-8')),
    key_deserializer=lambda k: k.decode('utf-8') if k else None,
    max_poll_records=10,
    session_timeout_ms=30000
)

try:
    while True:
        # Poll for messages (timeout 1s)
        records = consumer.poll(timeout_ms=1000)
        
        for topic_partition, messages in records.items():
            for msg in messages:
                print(f"Received [{topic_partition.partition}@{msg.offset}]: {msg.value}")
                
                try:
                    # Process message
                    process_order(msg.value)
                    
                    # Manual commit after successful processing
                    consumer.commit()
                    
                except Exception as e:
                    print(f"Error processing: {e}")
                    # ไม่ commit เพื่อให้ retry ใน next poll

except KeyboardInterrupt:
    print("Stopping consumer...")
finally:
    consumer.close()
'''

print("Kafka Producer:")
print(KAFKA_PRODUCER_CODE[:400])
print("\nKafka Consumer:")
print(KAFKA_CONSUMER_CODE[:400])
```

---

## 7. Consumer Groups

```python
# ตัวอย่าง 10: Consumer Groups
class ConsumerGroup:
    """Consumer Group - แบ่ง partitions ให้ consumers"""
    
    def __init__(self, group_id: str, broker: KafkaBroker):
        self.group_id = group_id
        self._broker = broker
        self._consumers: Dict[str, dict] = {}
    
    def join(self, consumer_id: str) -> dict:
        self._consumers[consumer_id] = {'id': consumer_id, 'active': True}
        assignment = self._rebalance()
        print(f"[Group:{self.group_id}] {consumer_id} joined. Assignment: {assignment}")
        return assignment
    
    def leave(self, consumer_id: str):
        if consumer_id in self._consumers:
            del self._consumers[consumer_id]
            self._rebalance()
            print(f"[Group:{self.group_id}] {consumer_id} left")
    
    def _rebalance(self) -> Dict[str, List[int]]:
        """จัด assignment ใหม่ (simplified round-robin)"""
        if not self._consumers:
            return {}
        
        consumer_ids = list(self._consumers.keys())
        assignment = {cid: [] for cid in consumer_ids}
        
        # Simple: assign partitions round-robin
        # In real Kafka: more sophisticated rebalancing strategies
        partitions = [0, 1, 2]  # assume 3 partitions
        for i, partition in enumerate(partitions):
            consumer = consumer_ids[i % len(consumer_ids)]
            assignment[consumer].append(partition)
        
        return assignment
    
    def consume(self, consumer_id: str, topic: str, partitions: List[int],
                max_records: int = 10) -> List[KafkaRecord]:
        """Consumer รับ messages ตาม partition assignment"""
        all_records = []
        for partition_id in partitions:
            records = self._broker.consume(topic, self.group_id, partition_id, max_records)
            all_records.extend(records)
        return all_records
    
    def commit(self, consumer_id: str, topic: str, records: List[KafkaRecord]):
        """Commit offsets"""
        for record in records:
            self._broker.commit_offset(
                self.group_id, topic, record.partition, record.offset
            )

# ทดสอบ Consumer Groups
print("\n=== Consumer Groups Demo ===")
broker2 = KafkaBroker()
broker2.create_topic("events", partitions=3)

# Produce events
event_types = ['pageview', 'click', 'purchase', 'signup', 'logout']
for i in range(15):
    event_type = event_types[i % len(event_types)]
    broker2.produce("events",
        key=f"session-{i % 5}",
        value={'event': event_type, 'user': f'user-{i}', 'timestamp': time.time()}
    )

# Consumer Group 1: Analytics
group1 = ConsumerGroup("analytics", broker2)
assignment1 = group1.join("analytics-1")
assignment2 = group1.join("analytics-2")

print(f"\nGroup 'analytics':")
print(f"  analytics-1 assignment: {assignment1.get('analytics-1', [])}")
print(f"  analytics-2 assignment: {assignment2.get('analytics-2', [])}")

# Consume from assigned partitions
for consumer_id, partitions in assignment2.items():
    records = group1.consume(consumer_id, "events", partitions)
    print(f"\n  {consumer_id} consumed {len(records)} records:")
    for r in records[:3]:  # show first 3
        print(f"    [{r.partition}@{r.offset}] {r.value}")
    group1.commit(consumer_id, "events", records)
```

---

## 8. Celery

### 8.1 Celery with RabbitMQ

```python
# ตัวอย่าง 11: Celery Basics
# ต้องติดตั้ง: pip install celery

CELERY_SETUP = '''
# celery_app.py
from celery import Celery
from celery.utils.log import get_task_logger

# Setup Celery app
app = Celery(
    'myapp',
    broker='amqp://guest:guest@localhost:5672//',
    backend='redis://localhost:6379/0'  # สำหรับเก็บผล
)

# Configuration
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='Asia/Bangkok',
    enable_utc=True,
    
    # Retry settings
    task_acks_late=True,
    task_reject_on_worker_lost=True,
    
    # Rate limiting
    task_annotations={
        'tasks.send_email': {'rate_limit': '10/m'},
        'tasks.process_payment': {'rate_limit': '100/s'}
    },
    
    # Queue routing
    task_routes={
        'tasks.send_email': {'queue': 'email'},
        'tasks.send_sms': {'queue': 'sms'},
        'tasks.process_payment': {'queue': 'payments'},
    }
)

logger = get_task_logger(__name__)

# Define tasks
@app.task(
    name='tasks.send_email',
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    autoretry_for=(ConnectionError,),
    retry_backoff=True
)
def send_email(self, to: str, subject: str, body: str):
    """ส่ง email (async task)"""
    try:
        logger.info(f"Sending email to {to}")
        # Actual email sending logic
        import smtplib
        with smtplib.SMTP('smtp.example.com') as server:
            server.sendmail('from@example.com', to, f"Subject: {subject}\\n\\n{body}")
        logger.info(f"Email sent to {to}")
        return {'status': 'sent', 'to': to}
    except Exception as exc:
        logger.error(f"Failed to send email: {exc}")
        raise self.retry(exc=exc, countdown=60)

@app.task(
    name='tasks.process_payment',
    bind=True,
    max_retries=5
)
def process_payment(self, order_id: str, amount: float, currency: str):
    """Process payment (async task)"""
    try:
        logger.info(f"Processing payment for order {order_id}: {amount} {currency}")
        # Payment processing logic
        result = charge_payment_gateway(order_id, amount, currency)
        return {'status': 'success', 'transaction_id': result['txn_id']}
    except PaymentError as exc:
        logger.error(f"Payment failed: {exc}")
        raise self.retry(exc=exc, countdown=30 * (2 ** self.request.retries))

# Canvas: Chaining tasks
from celery import chain, group, chord

# Chain: ทำ task ต่อเนื่อง
def process_order(order_id: str):
    workflow = chain(
        validate_order.s(order_id),
        reserve_inventory.s(),
        process_payment.s(amount=599.0, currency='THB'),
        send_confirmation_email.s()
    )
    return workflow.apply_async()

# Group: ทำ tasks พร้อมกัน
def notify_all_users(message: str, user_emails: list):
    tasks = group(
        send_email.s(email, 'Notification', message)
        for email in user_emails
    )
    return tasks.apply_async()

# Chord: Group + callback
def process_batch(items: list):
    chord(
        group(process_item.s(item) for item in items),
        aggregate_results.s()
    ).apply_async()
'''

print("Celery Setup:")
print(CELERY_SETUP[:600])
```

```python
# ตัวอย่าง 12: Celery จำลองการทำงาน
from typing import Callable
import functools
import time
from queue import Queue as ThreadQueue
from threading import Thread
from concurrent.futures import ThreadPoolExecutor

class FakeCeleryTask:
    """จำลอง Celery Task สำหรับ demo"""
    
    def __init__(self, func: Callable, name: str = None):
        self.func = func
        self.name = name or func.__name__
        self._results = {}
    
    def delay(self, *args, **kwargs):
        """Async execution (simulate)"""
        task_id = str(uuid.uuid4())[:8]
        print(f"[Celery] Task queued: {self.name}[{task_id}]")
        
        # Execute in background (simulation)
        def run():
            try:
                result = self.func(*args, **kwargs)
                self._results[task_id] = {'status': 'SUCCESS', 'result': result}
            except Exception as e:
                self._results[task_id] = {'status': 'FAILURE', 'error': str(e)}
        
        t = Thread(target=run, daemon=True)
        t.start()
        return FakeAsyncResult(task_id, self._results, t)
    
    def apply_async(self, args=None, kwargs=None, countdown=0, eta=None):
        """More control over execution"""
        if countdown:
            time.sleep(countdown)
        return self.delay(*(args or []), **(kwargs or {}))
    
    def __call__(self, *args, **kwargs):
        """Direct synchronous call"""
        return self.func(*args, **kwargs)

class FakeAsyncResult:
    def __init__(self, task_id: str, results_store: dict, thread: Thread):
        self.id = task_id
        self._store = results_store
        self._thread = thread
    
    def get(self, timeout: float = None) -> any:
        self._thread.join(timeout)
        result = self._store.get(self.id, {})
        if result.get('status') == 'FAILURE':
            raise Exception(result.get('error'))
        return result.get('result')
    
    @property
    def status(self) -> str:
        result = self._store.get(self.id, {})
        if not result:
            return 'PENDING'
        return result.get('status', 'PENDING')

def celery_task(name: str = None):
    """Decorator จำลอง @app.task"""
    def decorator(func):
        task = FakeCeleryTask(func, name or func.__name__)
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        wrapper.delay = task.delay
        wrapper.apply_async = task.apply_async
        wrapper.s = lambda *a, **kw: (func, a, kw)  # signature
        return wrapper
    return decorator

# Task definitions
@celery_task(name='tasks.send_email')
def send_email(to: str, subject: str, body: str) -> dict:
    print(f"  [Worker] Sending email to {to}...")
    time.sleep(0.1)
    return {'status': 'sent', 'to': to}

@celery_task(name='tasks.generate_report')
def generate_report(report_type: str, date_range: dict) -> dict:
    print(f"  [Worker] Generating {report_type} report...")
    time.sleep(0.2)
    return {
        'report_type': report_type,
        'rows': 1250,
        'generated_at': time.time()
    }

@celery_task(name='tasks.process_image')
def process_image(image_id: str, operations: list) -> dict:
    print(f"  [Worker] Processing image {image_id}: {operations}")
    time.sleep(0.15)
    return {'image_id': image_id, 'processed': True}

# ทดสอบ Celery Tasks
print("=== Celery Tasks Demo ===")

# Async tasks
print("\nQueuing tasks...")
result1 = send_email.delay("alice@example.com", "Welcome!", "Welcome to our platform!")
result2 = generate_report.delay("monthly_sales", {"start": "2024-01-01", "end": "2024-01-31"})
result3 = process_image.delay("IMG-001", ["resize", "compress"])

print("\nWaiting for results...")
try:
    email_result = result1.get(timeout=5)
    print(f"Email result: {email_result}")
    
    report_result = result2.get(timeout=5)
    print(f"Report result: rows={report_result['rows']}")
    
    image_result = result3.get(timeout=5)
    print(f"Image result: {image_result}")
except Exception as e:
    print(f"Error: {e}")
```

### 8.2 Periodic Tasks (Celery Beat)

```python
# ตัวอย่าง 13: Periodic Tasks

CELERY_BEAT_CODE = '''
# celery_beat.py
from celery import Celery
from celery.schedules import crontab

app = Celery('myapp', broker='amqp://localhost//')

app.conf.beat_schedule = {
    # ทุกวันตอน 08:00
    'send-daily-summary': {
        'task': 'tasks.send_daily_summary',
        'schedule': crontab(hour=8, minute=0),
    },
    
    # ทุก 5 นาที
    'check-inventory': {
        'task': 'tasks.check_inventory_levels',
        'schedule': crontab(minute='*/5'),
    },
    
    # ทุกวันจันทร์ตอน 09:00
    'weekly-report': {
        'task': 'tasks.generate_weekly_report',
        'schedule': crontab(hour=9, minute=0, day_of_week=1),
        'args': ('last_week',),
    },
    
    # ทุก 30 วินาที (สำหรับ real-time monitoring)
    'health-check': {
        'task': 'tasks.health_check',
        'schedule': 30.0,  # every 30 seconds
    },
}

@app.task
def send_daily_summary():
    users = get_all_active_users()
    for user in users:
        send_email.delay(
            user['email'],
            'Daily Summary',
            generate_summary(user['id'])
        )

@app.task
def check_inventory_levels():
    low_stock_items = get_low_stock_items(threshold=10)
    if low_stock_items:
        notify_warehouse.delay(low_stock_items)

@app.task
def generate_weekly_report(period: str):
    report = compile_sales_data(period)
    send_to_management.delay(report)

# รัน worker + beat
# celery -A celery_beat worker -l info
# celery -A celery_beat beat -l info
'''

# จำลอง Periodic Task Scheduler
import time
from datetime import datetime

class TaskScheduler:
    """Simple periodic task scheduler"""
    
    def __init__(self):
        self._schedules = []
        self._running = False
    
    def every(self, seconds: float, task: Callable, *args, **kwargs):
        self._schedules.append({
            'interval': seconds,
            'task': task,
            'args': args,
            'kwargs': kwargs,
            'last_run': 0,
            'run_count': 0
        })
    
    def run_once(self):
        """ทดสอบรัน 1 รอบ"""
        now = time.time()
        for schedule in self._schedules:
            if now - schedule['last_run'] >= schedule['interval']:
                task = schedule['task']
                print(f"[Scheduler] Running: {task.__name__}")
                try:
                    task(*schedule['args'], **schedule['kwargs'])
                    schedule['last_run'] = now
                    schedule['run_count'] += 1
                except Exception as e:
                    print(f"[Scheduler] Error: {e}")

# ทดสอบ Scheduler
def cleanup_expired_sessions():
    print("  → Cleaning expired sessions...")

def sync_analytics_data():
    print("  → Syncing analytics...")

def send_pending_notifications():
    print("  → Sending pending notifications...")

scheduler = TaskScheduler()
scheduler.every(30, cleanup_expired_sessions)
scheduler.every(60, sync_analytics_data)
scheduler.every(10, send_pending_notifications)

print("=== Task Scheduler Demo ===")
print("Running scheduled tasks...")
scheduler.run_once()
```

---

## 9. Order Processing System

```python
# ตัวอย่าง 14: Complete Order Processing System

class OrderProcessor:
    """ระบบ order processing ด้วย message queue"""
    
    def __init__(self, broker: KafkaBroker):
        self._broker = broker
        self._setup_topics()
    
    def _setup_topics(self):
        self._broker.create_topic("order.created", partitions=3)
        self._broker.create_topic("order.payment.processed", partitions=2)
        self._broker.create_topic("order.inventory.reserved", partitions=2)
        self._broker.create_topic("order.shipped", partitions=2)
        self._broker.create_topic("order.failed", partitions=1)
    
    def submit_order(self, order: dict) -> str:
        order_id = f"ORD-{str(uuid.uuid4())[:8].upper()}"
        order['id'] = order_id
        order['status'] = 'pending'
        order['created_at'] = datetime.now().isoformat()
        
        self._broker.produce(
            "order.created",
            key=order_id,
            value=order
        )
        print(f"[OrderSystem] Order submitted: {order_id}")
        return order_id
    
    def process_payment(self, order: dict) -> bool:
        """Simulate payment processing"""
        # จำลองการตรวจสอบ payment
        if order.get('total', 0) > 10000:
            print(f"  [Payment] Flagged for review: {order['id']}")
            return False
        
        print(f"  [Payment] Processing ${order.get('total', 0):.2f} for {order['id']}")
        time.sleep(0.01)
        
        self._broker.produce(
            "order.payment.processed",
            key=order['id'],
            value={
                'order_id': order['id'],
                'transaction_id': f"TXN-{str(uuid.uuid4())[:8]}",
                'amount': order.get('total', 0),
                'status': 'success'
            }
        )
        return True
    
    def reserve_inventory(self, order: dict) -> bool:
        """Simulate inventory reservation"""
        for item in order.get('items', []):
            print(f"  [Inventory] Reserving {item['qty']}x {item['name']}")
        
        self._broker.produce(
            "order.inventory.reserved",
            key=order['id'],
            value={
                'order_id': order['id'],
                'items': order.get('items', []),
                'status': 'reserved'
            }
        )
        return True
    
    def process_queue(self, group_id: str):
        """Process all pending orders"""
        # Process new orders
        records = self._broker.consume("order.created", group_id, partition_id=0, max_records=5)
        
        for record in records:
            order = record.value
            print(f"\nProcessing order: {order['id']}")
            
            payment_ok = self.process_payment(order)
            if not payment_ok:
                self._broker.produce("order.failed", key=order['id'],
                    value={'order_id': order['id'], 'reason': 'payment_failed'})
                continue
            
            inventory_ok = self.reserve_inventory(order)
            if not inventory_ok:
                self._broker.produce("order.failed", key=order['id'],
                    value={'order_id': order['id'], 'reason': 'out_of_stock'})
                continue
            
            self._broker.produce("order.shipped", key=order['id'],
                value={
                    'order_id': order['id'],
                    'tracking_number': f"TRACK-{str(uuid.uuid4())[:8].upper()}",
                    'estimated_delivery': '2-3 days'
                })
            print(f"  ✅ Order {order['id']} completed!")

# ทดสอบ Order Processing
print("=== Order Processing System ===")
from datetime import datetime

kafka = KafkaBroker()
processor = OrderProcessor(kafka)

# Submit orders
orders = [
    {
        'customer_id': 'CUST-001',
        'items': [
            {'name': 'Python Book', 'qty': 2, 'price': 599.0},
            {'name': 'USB Hub', 'qty': 1, 'price': 299.0}
        ],
        'total': 1497.0
    },
    {
        'customer_id': 'CUST-002',
        'items': [{'name': 'Laptop', 'qty': 1, 'price': 35000.0}],
        'total': 35000.0  # จะ flag ว่า too high
    },
    {
        'customer_id': 'CUST-003',
        'items': [{'name': 'Mouse', 'qty': 1, 'price': 499.0}],
        'total': 499.0
    }
]

for order in orders:
    processor.submit_order(order)

print("\nProcessing orders:")
processor.process_queue("order-workers")
```

---

## 10. Event Streaming & Notification System

```python
# ตัวอย่าง 15: Event Streaming with Kafka

class EventStreamingSystem:
    """Real-time event streaming"""
    
    def __init__(self):
        self._broker = KafkaBroker()
        self._setup()
    
    def _setup(self):
        self._broker.create_topic("user.events", partitions=4)
        self._broker.create_topic("product.events", partitions=3)
        self._broker.create_topic("system.metrics", partitions=2)
    
    def track_user_event(self, user_id: str, event_type: str, data: dict):
        self._broker.produce(
            "user.events",
            key=user_id,
            value={
                'user_id': user_id,
                'event': event_type,
                'data': data,
                'timestamp': time.time()
            },
            headers={'source': 'web-app', 'version': '2.0'}
        )
    
    def track_product_event(self, product_id: str, event_type: str, data: dict):
        self._broker.produce(
            "product.events",
            key=product_id,
            value={'product_id': product_id, 'event': event_type, **data}
        )
    
    def publish_metric(self, service: str, metric: str, value: float):
        self._broker.produce(
            "system.metrics",
            key=service,
            value={'service': service, 'metric': metric, 'value': value, 'ts': time.time()}
        )
    
    def get_user_journey(self, user_id: str, group_id: str) -> list:
        """ดู user events ทั้งหมด"""
        events = []
        for partition_id in range(4):
            records = self._broker.consume("user.events", group_id, partition_id, 50)
            events.extend([r.value for r in records if r.key == user_id])
        return sorted(events, key=lambda e: e['timestamp'])

# ทดสอบ Event Streaming
print("=== Event Streaming System ===")
streaming = EventStreamingSystem()

# Track user journey
user_id = "USER-001"
streaming.track_user_event(user_id, "page_view", {"page": "/home"})
streaming.track_user_event(user_id, "search", {"query": "python book"})
streaming.track_user_event(user_id, "product_view", {"product_id": "PROD-001"})
streaming.track_user_event(user_id, "add_to_cart", {"product_id": "PROD-001", "qty": 1})
streaming.track_user_event(user_id, "checkout_start", {"cart_total": 599.0})
streaming.track_user_event(user_id, "purchase", {"order_id": "ORD-001", "total": 599.0})

# Track metrics
streaming.publish_metric("user-service", "active_users", 1250)
streaming.publish_metric("order-service", "orders_per_minute", 45)
streaming.publish_metric("api-gateway", "response_time_ms", 125.5)

# Retrieve journey
journey = streaming.get_user_journey(user_id, "analytics")
print(f"\nUser {user_id} journey ({len(journey)} events):")
for event in journey:
    print(f"  {event['event']}: {event['data']}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Priority Queue System
```python
# เฉลย
import heapq
from dataclasses import dataclass, field
from typing import Optional
from enum import IntEnum
import time
import uuid

class Priority(IntEnum):
    CRITICAL = 0    # สูงสุด
    HIGH = 1
    MEDIUM = 2
    LOW = 3
    BULK = 4        # ต่ำสุด

@dataclass(order=True)
class PriorityMessage:
    priority: int
    timestamp: float = field(compare=False)
    message_id: str = field(compare=False)
    body: dict = field(compare=False)
    
    @classmethod
    def create(cls, body: dict, priority: Priority) -> 'PriorityMessage':
        return cls(
            priority=int(priority),
            timestamp=time.time(),
            message_id=str(uuid.uuid4()),
            body=body
        )

class PriorityQueue:
    """Priority Queue ที่ process messages ตาม priority"""
    
    def __init__(self, name: str):
        self.name = name
        self._heap = []
        self._processed = 0
    
    def publish(self, body: dict, priority: Priority = Priority.MEDIUM):
        msg = PriorityMessage.create(body, priority)
        heapq.heappush(self._heap, msg)
        print(f"[PQ:{self.name}] Queued [{priority.name}]: {body.get('type', 'unknown')}")
        return msg.message_id
    
    def consume(self) -> Optional[PriorityMessage]:
        if not self._heap:
            return None
        msg = heapq.heappop(self._heap)
        self._processed += 1
        return msg
    
    def peek(self) -> Optional[PriorityMessage]:
        return self._heap[0] if self._heap else None
    
    @property
    def size(self) -> int:
        return len(self._heap)
    
    def process_all(self, handler: Callable):
        print(f"\n[PQ:{self.name}] Processing {self.size} messages by priority:")
        while self._heap:
            msg = self.consume()
            priority_name = Priority(msg.priority).name
            print(f"  [{priority_name}] Processing: {msg.body}")
            handler(msg.body)
        print(f"Processed: {self._processed} messages")

# ทดสอบ
pq = PriorityQueue("notifications")

# Publish ลำดับสลับกัน
pq.publish({'type': 'newsletter', 'to': 'user1@example.com'}, Priority.BULK)
pq.publish({'type': 'otp', 'to': 'user2@example.com', 'code': '123456'}, Priority.CRITICAL)
pq.publish({'type': 'order_confirm', 'order_id': 'ORD-001'}, Priority.HIGH)
pq.publish({'type': 'weekly_digest', 'to': 'user3@example.com'}, Priority.LOW)
pq.publish({'type': 'password_reset', 'token': 'abc123'}, Priority.CRITICAL)
pq.publish({'type': 'payment_alert', 'amount': 999}, Priority.HIGH)
pq.publish({'type': 'welcome_email', 'user': 'alice'}, Priority.MEDIUM)

def send_notification(msg: dict):
    pass  # จำลองการส่ง

pq.process_all(send_notification)
```

### แบบฝึกหัดที่ 2 - 8

**แบบฝึกหัดที่ 2**: สร้าง Request-Reply Pattern ด้วย Message Queue

**แบบฝึกหัดที่ 3**: สร้าง Message Deduplication System (ป้องกัน process message ซ้ำ)

**แบบฝึกหัดที่ 4**: สร้าง Batch Processing System ที่ group messages เป็น batch ก่อน process

**แบบฝึกหัดที่ 5**: สร้าง Event Store ด้วย Kafka Simulation

**แบบฝึกหัดที่ 6**: สร้าง Task Chain ด้วย Celery-like Framework

**แบบฝึกหัดที่ 7**: สร้าง Message Routing System ที่รองรับ Filter, Transform, Route

**แบบฝึกหัดที่ 8**: สร้าง Complete Notification System ที่รองรับ Email, SMS, Push พร้อม retry, priority, และ rate limiting

---

## สรุป

| ระบบ | Use Case | Strength |
|------|----------|---------|
| **RabbitMQ** | Task queues, RPC, Routing | Flexible routing, AMQP standard |
| **Apache Kafka** | Event streaming, Log aggregation | High throughput, Replay-able |
| **Redis Queue** | Simple tasks, Rate limiting | Low latency, In-memory |
| **Celery** | Python async tasks, Scheduling | Python-native, Rich features |

### เมื่อไหร่ใช้อะไร

- **RabbitMQ**: ต้องการ complex routing, RPC pattern, ข้อมูล traditional queue
- **Kafka**: Event streaming, audit log, real-time analytics, high volume
- **Celery + Redis/RabbitMQ**: Python async tasks, periodic jobs, background processing

> "The key to scalable systems is to decouple components through async communication." - Martin Fowler

---

*Part 90 เสร็จสมบูรณ์ | Python Course Parts 86-90 Complete!*
