# Part 89: Microservices with Python

## บทนำ

Microservices Architecture คือแนวทางการออกแบบซอฟต์แวร์ที่แบ่งแอปพลิเคชันออกเป็น services ขนาดเล็กที่ทำงานอิสระต่อกัน แต่ละ service มีหน้าที่เฉพาะ deploy ได้แยกกัน และสื่อสารกันผ่าน network

---

## 1. Microservices vs Monolith

### ข้อเปรียบเทียบ

| ด้าน | Monolith | Microservices |
|------|----------|---------------|
| **Deployment** | Deploy ทั้งหมดในครั้งเดียว | Deploy แต่ละ service แยกกัน |
| **Scaling** | Scale ทั้งหมด | Scale เฉพาะ service ที่ต้องการ |
| **Technology** | เทคโนโลยีเดียว | เลือกเทคโนโลยีต่าง service ได้ |
| **Team** | Team เดียว | แต่ละ team ดูแล service ของตัว |
| **Complexity** | Simple ตอนเริ่ม | Complex ตั้งแต่ต้น |
| **Testing** | ง่าย | ซับซ้อน (integration tests) |
| **Failure** | ล้มทั้งระบบ | Isolated failures |

```python
# ตัวอย่าง 1: Monolith vs Microservices Structure
# ตัวอย่างแสดงโครงสร้างเปรียบเทียบ

# MONOLITH: ทุกอย่างอยู่ใน codebase เดียว
class MonolithApp:
    def __init__(self):
        self.user_service = UserService(self.db)
        self.order_service = OrderService(self.db)
        self.payment_service = PaymentService(self.db)
        self.inventory_service = InventoryService(self.db)
        self.notification_service = NotificationService()
    
    # ปัญหา: coupling สูง, scale ไม่ได้เฉพาะส่วน

# MICROSERVICES: แต่ละส่วนเป็น service แยก
# user-service/main.py   → Port 8001
# order-service/main.py  → Port 8002
# payment-service/main.py → Port 8003
# inventory-service/main.py → Port 8004

# แต่ละ service มีของตัวเอง:
# - Database
# - Deployment
# - Technology stack
# - Team responsibility
```

```python
# ตัวอย่าง 2: Simple Microservice ด้วย Flask

# user_service/main.py
from flask import Flask, jsonify, request
from dataclasses import dataclass, asdict
from typing import Dict, Optional
import uuid

app = Flask(__name__)

# In-memory "database"
users_db: Dict[str, dict] = {}

@dataclass
class User:
    id: str
    username: str
    email: str
    created_at: str

@app.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint"""
    return jsonify({'status': 'healthy', 'service': 'user-service'})

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data or 'username' not in data or 'email' not in data:
        return jsonify({'error': 'username and email required'}), 400
    
    user_id = str(uuid.uuid4())
    from datetime import datetime
    user = {
        'id': user_id,
        'username': data['username'],
        'email': data['email'],
        'created_at': datetime.now().isoformat()
    }
    users_db[user_id] = user
    return jsonify(user), 201

@app.route('/users/<user_id>', methods=['GET'])
def get_user(user_id: str):
    user = users_db.get(user_id)
    if not user:
        return jsonify({'error': 'User not found'}), 404
    return jsonify(user)

@app.route('/users', methods=['GET'])
def list_users():
    return jsonify(list(users_db.values()))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8001, debug=True)

# หมายเหตุ: ตัวอย่างนี้แสดงโครงสร้าง
# รันจริงต้องติดตั้ง flask: pip install flask
print("User Service structure defined")
print("Routes: GET /health, POST /users, GET /users/<id>, GET /users")
```

---

## 2. Service Decomposition Strategies

```python
# ตัวอย่าง 3: Decomposition by Business Capability

# E-Commerce Microservices:
# 
# ┌─────────────────────────────────────────────────────┐
# │                    API Gateway                       │
# │                   :8080                              │
# └──────┬──────────────┬──────────────┬────────────────┘
#        │              │              │
# ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
# │    User     │ │   Product   │ │    Order    │
# │  Service   │ │   Service   │ │   Service   │
# │   :8001    │ │   :8002     │ │   :8003     │
# └─────────────┘ └─────────────┘ └─────────────┘
#        │              │              │
# ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
# │  Users DB   │ │ Products DB │ │  Orders DB  │
# └─────────────┘ └─────────────┘ └─────────────┘

# การสื่อสารระหว่าง services:
# 1. Synchronous: REST API, gRPC
# 2. Asynchronous: Message Queue (RabbitMQ, Kafka)

class ServiceRegistry:
    """Simple Service Registry"""
    def __init__(self):
        self._services = {}
    
    def register(self, name: str, host: str, port: int):
        self._services[name] = {
            'host': host,
            'port': port,
            'url': f"http://{host}:{port}",
            'healthy': True
        }
        print(f"Registered: {name} at {host}:{port}")
    
    def get_url(self, name: str) -> str:
        service = self._services.get(name)
        if not service:
            raise ValueError(f"Service '{name}' not found")
        if not service['healthy']:
            raise ConnectionError(f"Service '{name}' is unhealthy")
        return service['url']
    
    def mark_unhealthy(self, name: str):
        if name in self._services:
            self._services[name]['healthy'] = False
    
    def list_services(self):
        return {name: info['url'] for name, info in self._services.items()}

# ทดสอบ
registry = ServiceRegistry()
registry.register("user-service", "localhost", 8001)
registry.register("product-service", "localhost", 8002)
registry.register("order-service", "localhost", 8003)
registry.register("payment-service", "localhost", 8004)

print("\nRegistered services:")
for name, url in registry.list_services().items():
    print(f"  {name}: {url}")
```

---

## 3. Inter-Service Communication

### 3.1 REST API Communication

```python
# ตัวอย่าง 4: Service-to-Service REST Communication
import json
from typing import Optional, Any
from urllib.request import urlopen, Request
from urllib.error import URLError, HTTPError

class ServiceClient:
    """HTTP Client สำหรับ inter-service communication"""
    
    def __init__(self, base_url: str, timeout: int = 10):
        self.base_url = base_url.rstrip('/')
        self.timeout = timeout
    
    def get(self, path: str) -> Optional[dict]:
        url = f"{self.base_url}{path}"
        try:
            req = Request(url, headers={'Content-Type': 'application/json'})
            with urlopen(req, timeout=self.timeout) as response:
                return json.loads(response.read().decode())
        except HTTPError as e:
            print(f"HTTP Error {e.code}: {e.reason}")
            return None
        except URLError as e:
            print(f"Connection Error: {e.reason}")
            return None
    
    def post(self, path: str, data: dict) -> Optional[dict]:
        url = f"{self.base_url}{path}"
        try:
            body = json.dumps(data).encode()
            req = Request(
                url, 
                data=body,
                headers={'Content-Type': 'application/json'},
                method='POST'
            )
            with urlopen(req, timeout=self.timeout) as response:
                return json.loads(response.read().decode())
        except (HTTPError, URLError) as e:
            print(f"Error calling {url}: {e}")
            return None

class OrderServiceClient:
    """Typed client สำหรับ Order Service"""
    
    def __init__(self, base_url: str):
        self._client = ServiceClient(base_url)
    
    def get_order(self, order_id: str) -> Optional[dict]:
        return self._client.get(f"/orders/{order_id}")
    
    def create_order(self, customer_id: str, items: list) -> Optional[dict]:
        return self._client.post("/orders", {
            'customer_id': customer_id,
            'items': items
        })
    
    def get_customer_orders(self, customer_id: str) -> list:
        result = self._client.get(f"/orders?customer_id={customer_id}")
        return result or []

# ตัวอย่างการใช้งาน (ต้องมี service ทำงานอยู่จริง)
# order_client = OrderServiceClient("http://order-service:8003")
# order = order_client.create_order("CUST-001", [{"product_id": "P1", "qty": 2}])
print("OrderServiceClient defined - requires running service")
```

### 3.2 gRPC Communication

```python
# ตัวอย่าง 5: gRPC ด้วย Python
# ต้องติดตั้ง: pip install grpcio grpcio-tools

# user.proto
PROTO_DEFINITION = """
syntax = "proto3";

package userservice;

service UserService {
    rpc GetUser (GetUserRequest) returns (UserResponse);
    rpc CreateUser (CreateUserRequest) returns (UserResponse);
    rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
}

message GetUserRequest {
    string user_id = 1;
}

message CreateUserRequest {
    string username = 1;
    string email = 2;
}

message UserResponse {
    string id = 1;
    string username = 2;
    string email = 3;
    string created_at = 4;
    bool success = 5;
    string error = 6;
}

message ListUsersRequest {
    int32 page = 1;
    int32 page_size = 2;
}

message ListUsersResponse {
    repeated UserResponse users = 1;
    int32 total = 2;
}
"""

print("Proto definition for gRPC:")
print(PROTO_DEFINITION)

# สร้าง code จาก proto ด้วย:
# python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. user.proto

# user_service_grpc.py (Generated + Implementation)
GRPC_SERVER_EXAMPLE = '''
import grpc
from concurrent import futures
import uuid
from datetime import datetime
# import user_pb2
# import user_pb2_grpc

class UserServiceServicer:
    """gRPC Service Implementation"""
    
    def __init__(self):
        self._users = {}
    
    def GetUser(self, request, context):
        user = self._users.get(request.user_id)
        if not user:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details(f"User {request.user_id} not found")
            return UserResponse(success=False, error="User not found")
        
        return UserResponse(
            id=user["id"],
            username=user["username"],
            email=user["email"],
            created_at=user["created_at"],
            success=True
        )
    
    def CreateUser(self, request, context):
        if not request.username or not request.email:
            context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
            return UserResponse(success=False, error="Missing required fields")
        
        user_id = str(uuid.uuid4())
        user = {
            "id": user_id,
            "username": request.username,
            "email": request.email,
            "created_at": datetime.now().isoformat()
        }
        self._users[user_id] = user
        
        return UserResponse(
            id=user_id,
            username=request.username,
            email=request.email,
            created_at=user["created_at"],
            success=True
        )

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # user_pb2_grpc.add_UserServiceServicer_to_server(UserServiceServicer(), server)
    server.add_insecure_port("[::]:50051")
    print("gRPC User Service started on port 50051")
    server.start()
    server.wait_for_termination()

# gRPC Client
def create_grpc_client(host: str, port: int):
    channel = grpc.insecure_channel(f"{host}:{port}")
    # stub = user_pb2_grpc.UserServiceStub(channel)
    return channel

if __name__ == "__main__":
    serve()
'''

print("\ngRPC Server Implementation:")
print(GRPC_SERVER_EXAMPLE[:500])
```

---

## 4. API Gateway Pattern

```python
# ตัวอย่าง 6: API Gateway
from typing import Dict, Callable, Optional, Any
from dataclasses import dataclass
import time

@dataclass
class Route:
    method: str
    path: str
    service_name: str
    service_path: str
    require_auth: bool = True

class AuthMiddleware:
    """JWT Authentication Middleware"""
    
    def __init__(self, secret: str):
        self._secret = secret
        self._valid_tokens = {"valid-token-123": "user-1"}
    
    def authenticate(self, token: str) -> Optional[str]:
        """Return user_id if valid, None if invalid"""
        return self._valid_tokens.get(token)

class RateLimiter:
    """Rate Limiter Middleware"""
    
    def __init__(self, requests_per_minute: int = 60):
        self._limit = requests_per_minute
        self._requests: Dict[str, list] = {}
    
    def is_allowed(self, client_id: str) -> bool:
        now = time.time()
        if client_id not in self._requests:
            self._requests[client_id] = []
        
        # ลบ requests เก่ากว่า 60 วินาที
        self._requests[client_id] = [
            t for t in self._requests[client_id]
            if now - t < 60
        ]
        
        if len(self._requests[client_id]) >= self._limit:
            return False
        
        self._requests[client_id].append(now)
        return True

class RequestLogger:
    """Request Logging Middleware"""
    
    def log_request(self, method: str, path: str, user_id: Optional[str]):
        timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
        user = user_id or 'anonymous'
        print(f"[{timestamp}] {method} {path} user={user}")

class APIGateway:
    """API Gateway - Entry point สำหรับทุก requests"""
    
    def __init__(self):
        self._routes: list = []
        self._services: Dict[str, str] = {}
        self._auth = AuthMiddleware("secret-key")
        self._rate_limiter = RateLimiter(requests_per_minute=100)
        self._logger = RequestLogger()
    
    def register_service(self, name: str, base_url: str):
        self._services[name] = base_url
        print(f"Gateway: Registered service '{name}' at {base_url}")
    
    def add_route(self, route: Route):
        self._routes.append(route)
    
    def handle_request(
        self, 
        method: str, 
        path: str, 
        headers: dict, 
        body: Optional[dict] = None,
        client_ip: str = "127.0.0.1"
    ) -> dict:
        """Process incoming request"""
        
        # 1. Rate limiting
        if not self._rate_limiter.is_allowed(client_ip):
            return {'status': 429, 'error': 'Too Many Requests'}
        
        # 2. Find matching route
        route = self._find_route(method, path)
        if not route:
            return {'status': 404, 'error': 'Route not found'}
        
        # 3. Authentication
        user_id = None
        if route.require_auth:
            token = headers.get('Authorization', '').replace('Bearer ', '')
            user_id = self._auth.authenticate(token)
            if not user_id:
                return {'status': 401, 'error': 'Unauthorized'}
        
        # 4. Log request
        self._logger.log_request(method, path, user_id)
        
        # 5. Forward to service
        service_url = self._services.get(route.service_name)
        if not service_url:
            return {'status': 503, 'error': f"Service '{route.service_name}' unavailable"}
        
        # จำลอง forwarding
        return self._forward_request(
            service_url + route.service_path,
            method,
            headers,
            body,
            user_id
        )
    
    def _find_route(self, method: str, path: str) -> Optional[Route]:
        for route in self._routes:
            if route.method == method and self._path_matches(route.path, path):
                return route
        return None
    
    def _path_matches(self, pattern: str, path: str) -> bool:
        """Simple path matching"""
        if '{' not in pattern:
            return pattern == path
        
        pattern_parts = pattern.split('/')
        path_parts = path.split('/')
        
        if len(pattern_parts) != len(path_parts):
            return False
        
        for p, a in zip(pattern_parts, path_parts):
            if p.startswith('{') and p.endswith('}'):
                continue  # path parameter
            if p != a:
                return False
        return True
    
    def _forward_request(
        self, 
        url: str, 
        method: str, 
        headers: dict, 
        body: Optional[dict],
        user_id: Optional[str]
    ) -> dict:
        """จำลองการ forward request ไปยัง service"""
        print(f"  → Forwarding {method} to {url}")
        if user_id:
            print(f"  → User: {user_id}")
        # ในระบบจริงจะใช้ requests library
        return {'status': 200, 'forwarded_to': url, 'user_id': user_id}

# ทดสอบ API Gateway
gateway = APIGateway()
gateway.register_service("users", "http://user-service:8001")
gateway.register_service("products", "http://product-service:8002")
gateway.register_service("orders", "http://order-service:8003")

# Register routes
gateway.add_route(Route("GET", "/api/users", "users", "/users"))
gateway.add_route(Route("POST", "/api/users", "users", "/users", require_auth=False))
gateway.add_route(Route("GET", "/api/users/{id}", "users", "/users/{id}"))
gateway.add_route(Route("GET", "/api/products", "products", "/products", require_auth=False))
gateway.add_route(Route("POST", "/api/orders", "orders", "/orders"))

print("\n=== Gateway Requests ===")

# ไม่มี auth token
result = gateway.handle_request(
    "GET", "/api/users",
    headers={},
    client_ip="192.168.1.1"
)
print(f"No auth: {result}")

# มี auth token
result = gateway.handle_request(
    "GET", "/api/users",
    headers={"Authorization": "Bearer valid-token-123"},
    client_ip="192.168.1.1"
)
print(f"With auth: {result}")

# Public route
result = gateway.handle_request(
    "GET", "/api/products",
    headers={},
    client_ip="192.168.1.2"
)
print(f"Public route: {result}")
```

---

## 5. Circuit Breaker Pattern

```python
# ตัวอย่าง 7: Circuit Breaker
from enum import Enum
from datetime import datetime
import time

class CircuitState(Enum):
    CLOSED = "closed"       # ปกติ - requests ผ่านได้
    OPEN = "open"           # เปิด - requests ถูก block
    HALF_OPEN = "half_open" # กำลังทดสอบ

class CircuitBreaker:
    """Circuit Breaker Pattern"""
    
    def __init__(
        self,
        failure_threshold: int = 5,
        reset_timeout: int = 60,
        success_threshold: int = 2
    ):
        self._failure_threshold = failure_threshold
        self._reset_timeout = reset_timeout
        self._success_threshold = success_threshold
        self._state = CircuitState.CLOSED
        self._failure_count = 0
        self._success_count = 0
        self._last_failure_time = None
    
    def call(self, func: Callable, *args, **kwargs):
        if self._state == CircuitState.OPEN:
            if self._should_try_reset():
                self._state = CircuitState.HALF_OPEN
                print(f"Circuit: HALF_OPEN - testing...")
            else:
                raise CircuitBreakerOpenError(
                    f"Circuit is OPEN. Retry after {self._reset_timeout}s"
                )
        
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise
    
    def _on_success(self):
        if self._state == CircuitState.HALF_OPEN:
            self._success_count += 1
            if self._success_count >= self._success_threshold:
                self._reset()
                print("Circuit: CLOSED - service recovered!")
        elif self._state == CircuitState.CLOSED:
            self._failure_count = 0
    
    def _on_failure(self):
        self._failure_count += 1
        self._last_failure_time = time.time()
        
        if self._state == CircuitState.HALF_OPEN:
            self._state = CircuitState.OPEN
            self._success_count = 0
            print(f"Circuit: OPEN (half-open failed)")
        elif self._failure_count >= self._failure_threshold:
            self._state = CircuitState.OPEN
            print(f"Circuit: OPEN (too many failures: {self._failure_count})")
    
    def _should_try_reset(self) -> bool:
        return (self._last_failure_time is not None and
                time.time() - self._last_failure_time >= self._reset_timeout)
    
    def _reset(self):
        self._state = CircuitState.CLOSED
        self._failure_count = 0
        self._success_count = 0
    
    @property
    def state(self) -> str:
        return self._state.value

class CircuitBreakerOpenError(Exception):
    pass

# ทดสอบ Circuit Breaker
import random

call_count = [0]
fail_mode = [True]

def unreliable_service():
    """Service ที่ไม่เสถียร"""
    call_count[0] += 1
    if fail_mode[0]:
        raise ConnectionError("Service unavailable")
    return {"data": "success", "call": call_count[0]}

cb = CircuitBreaker(
    failure_threshold=3,
    reset_timeout=2,   # 2 วินาทีสำหรับทดสอบ
    success_threshold=2
)

print("=== Circuit Breaker Demo ===")
print("Testing with failing service:")
for i in range(7):
    try:
        result = cb.call(unreliable_service)
        print(f"  Call {i+1}: Success - {result}")
    except CircuitBreakerOpenError as e:
        print(f"  Call {i+1}: Circuit OPEN - {e}")
    except ConnectionError as e:
        print(f"  Call {i+1}: Failed ({cb.state}) - {e}")

print(f"\nWaiting for reset timeout...")
time.sleep(2.5)

print("Service recovered, testing again:")
fail_mode[0] = False  # Service recovered
for i in range(5):
    try:
        result = cb.call(unreliable_service)
        print(f"  Call {i+1}: Success - state={cb.state}")
    except Exception as e:
        print(f"  Call {i+1}: Error - {e}")
```

---

## 6. Saga Pattern

```python
# ตัวอย่าง 8: Saga Pattern - Distributed Transactions

class SagaStep:
    """Step ใน Saga"""
    def __init__(self, name: str, action: Callable, compensate: Callable):
        self.name = name
        self.action = action
        self.compensate = compensate

class Saga:
    """Orchestration-based Saga"""
    
    def __init__(self, name: str):
        self._name = name
        self._steps: list = []
        self._completed_steps: list = []
    
    def add_step(self, step: SagaStep) -> 'Saga':
        self._steps.append(step)
        return self
    
    def execute(self, context: dict) -> bool:
        print(f"\n=== Saga: {self._name} ===")
        self._completed_steps = []
        
        for step in self._steps:
            print(f"  Executing: {step.name}")
            try:
                result = step.action(context)
                context[f"{step.name}_result"] = result
                self._completed_steps.append(step)
                print(f"  ✅ {step.name} completed")
            except Exception as e:
                print(f"  ❌ {step.name} failed: {e}")
                self._compensate(context)
                return False
        
        print(f"=== Saga completed successfully ===")
        return True
    
    def _compensate(self, context: dict):
        print(f"\n  --- Compensating ---")
        for step in reversed(self._completed_steps):
            try:
                step.compensate(context)
                print(f"  ↩️  Compensated: {step.name}")
            except Exception as e:
                print(f"  ⚠️ Compensation failed for {step.name}: {e}")

# Simulated Services
class OrderService:
    def create_order(self, context: dict) -> str:
        order_id = "ORD-001"
        print(f"    OrderService: Created {order_id}")
        context['order_id'] = order_id
        return order_id
    
    def cancel_order(self, context: dict):
        print(f"    OrderService: Cancelled {context.get('order_id')}")

class InventoryService:
    def __init__(self, should_fail: bool = False):
        self.should_fail = should_fail
    
    def reserve(self, context: dict) -> str:
        if self.should_fail:
            raise ValueError("Insufficient stock!")
        reservation_id = "RES-001"
        print(f"    InventoryService: Reserved {reservation_id}")
        context['reservation_id'] = reservation_id
        return reservation_id
    
    def release(self, context: dict):
        print(f"    InventoryService: Released {context.get('reservation_id')}")

class PaymentService:
    def charge(self, context: dict) -> str:
        transaction_id = "TXN-001"
        print(f"    PaymentService: Charged ${context.get('amount')} -> {transaction_id}")
        context['transaction_id'] = transaction_id
        return transaction_id
    
    def refund(self, context: dict):
        print(f"    PaymentService: Refunded {context.get('transaction_id')}")

class ShippingService:
    def create_shipment(self, context: dict) -> str:
        tracking = "TRACK-001"
        print(f"    ShippingService: Created shipment {tracking}")
        context['tracking_number'] = tracking
        return tracking
    
    def cancel_shipment(self, context: dict):
        print(f"    ShippingService: Cancelled {context.get('tracking_number')}")

# ทดสอบ Successful Saga
order_svc = OrderService()
inventory_svc = InventoryService(should_fail=False)
payment_svc = PaymentService()
shipping_svc = ShippingService()

saga = Saga("CreateOrder")
saga.add_step(SagaStep(
    "CreateOrder",
    order_svc.create_order,
    order_svc.cancel_order
))
saga.add_step(SagaStep(
    "ReserveInventory",
    inventory_svc.reserve,
    inventory_svc.release
))
saga.add_step(SagaStep(
    "ProcessPayment",
    payment_svc.charge,
    payment_svc.refund
))
saga.add_step(SagaStep(
    "CreateShipment",
    shipping_svc.create_shipment,
    shipping_svc.cancel_shipment
))

context = {'amount': 599.0, 'customer_id': 'CUST-001'}
success = saga.execute(context)
print(f"Result: {'Success' if success else 'Failed'}")

# ทดสอบ Failed Saga (inventory out of stock)
print("\n" + "="*50)
failing_inventory = InventoryService(should_fail=True)

saga2 = Saga("CreateOrder-WithFailure")
saga2.add_step(SagaStep("CreateOrder", order_svc.create_order, order_svc.cancel_order))
saga2.add_step(SagaStep("ReserveInventory", failing_inventory.reserve, failing_inventory.release))
saga2.add_step(SagaStep("ProcessPayment", payment_svc.charge, payment_svc.refund))

context2 = {'amount': 599.0, 'customer_id': 'CUST-002'}
success2 = saga2.execute(context2)
print(f"Result: {'Success' if success2 else 'Failed - compensated'}")
```

---

## 7. Service Discovery

```python
# ตัวอย่าง 9: Service Discovery

from typing import List
import random

@dataclass
class ServiceInstance:
    service_id: str
    host: str
    port: int
    metadata: dict = None
    
    @property
    def url(self) -> str:
        return f"http://{self.host}:{self.port}"
    
    def __repr__(self):
        return f"ServiceInstance({self.service_id}@{self.url})"

class ServiceDiscovery:
    """Client-side Service Discovery"""
    
    def __init__(self):
        self._registry: Dict[str, List[ServiceInstance]] = {}
    
    def register(self, service_name: str, instance: ServiceInstance):
        if service_name not in self._registry:
            self._registry[service_name] = []
        self._registry[service_name].append(instance)
        print(f"Registered: {service_name} -> {instance}")
    
    def deregister(self, service_name: str, service_id: str):
        if service_name in self._registry:
            self._registry[service_name] = [
                i for i in self._registry[service_name]
                if i.service_id != service_id
            ]
    
    def get_instances(self, service_name: str) -> List[ServiceInstance]:
        return self._registry.get(service_name, [])
    
    def get_instance(self, service_name: str, strategy: str = 'round_robin') -> Optional[ServiceInstance]:
        instances = self.get_instances(service_name)
        if not instances:
            return None
        
        if strategy == 'random':
            return random.choice(instances)
        elif strategy == 'round_robin':
            if not hasattr(self, '_counters'):
                self._counters = {}
            idx = self._counters.get(service_name, 0)
            instance = instances[idx % len(instances)]
            self._counters[service_name] = idx + 1
            return instance
        
        return instances[0]

# ทดสอบ Service Discovery
discovery = ServiceDiscovery()

# Register multiple instances (horizontal scaling)
for i in range(3):
    discovery.register("product-service", ServiceInstance(
        service_id=f"product-{i+1}",
        host="localhost",
        port=8100 + i
    ))

discovery.register("user-service", ServiceInstance(
    service_id="user-1",
    host="localhost",
    port=8001
))

print("\nRound-robin load balancing:")
for i in range(6):
    instance = discovery.get_instance("product-service")
    print(f"  Request {i+1} → {instance}")
```

---

## 8. Distributed Tracing (OpenTelemetry)

```python
# ตัวอย่าง 10: Distributed Tracing

import time
import uuid
from typing import Optional, Dict

class Span:
    """Represents a single operation in a trace"""
    
    def __init__(
        self,
        name: str,
        trace_id: str,
        parent_id: Optional[str] = None
    ):
        self.span_id = str(uuid.uuid4())[:8]
        self.trace_id = trace_id
        self.parent_id = parent_id
        self.name = name
        self.start_time = time.time()
        self.end_time = None
        self.tags: Dict[str, str] = {}
        self.events: list = []
        self.status = "OK"
    
    def set_tag(self, key: str, value: str) -> 'Span':
        self.tags[key] = str(value)
        return self
    
    def log(self, message: str):
        self.events.append({
            'time': time.time(),
            'message': message
        })
    
    def set_error(self, error: str):
        self.status = "ERROR"
        self.tags['error'] = error
    
    def finish(self):
        self.end_time = time.time()
    
    @property
    def duration_ms(self) -> float:
        if self.end_time:
            return (self.end_time - self.start_time) * 1000
        return (time.time() - self.start_time) * 1000
    
    def __repr__(self):
        return (f"Span(name={self.name}, id={self.span_id}, "
                f"trace={self.trace_id}, duration={self.duration_ms:.1f}ms)")

class Tracer:
    """Simple Distributed Tracer"""
    
    def __init__(self, service_name: str):
        self.service_name = service_name
        self._spans: List[Span] = []
    
    def start_trace(self, operation: str) -> Span:
        trace_id = str(uuid.uuid4())[:16]
        span = Span(operation, trace_id)
        span.set_tag("service", self.service_name)
        self._spans.append(span)
        return span
    
    def start_child_span(self, parent: Span, operation: str) -> Span:
        span = Span(operation, parent.trace_id, parent.span_id)
        span.set_tag("service", self.service_name)
        self._spans.append(span)
        return span
    
    def get_trace(self, trace_id: str) -> List[Span]:
        return [s for s in self._spans if s.trace_id == trace_id]

# Propagation headers
class TraceContext:
    """Propagate trace context between services"""
    TRACE_ID_HEADER = 'X-Trace-Id'
    SPAN_ID_HEADER = 'X-Span-Id'
    
    @staticmethod
    def inject(span: Span) -> Dict[str, str]:
        return {
            TraceContext.TRACE_ID_HEADER: span.trace_id,
            TraceContext.SPAN_ID_HEADER: span.span_id
        }
    
    @staticmethod
    def extract(headers: Dict[str, str]) -> tuple:
        return (
            headers.get(TraceContext.TRACE_ID_HEADER),
            headers.get(TraceContext.SPAN_ID_HEADER)
        )

# ทดสอบ Distributed Tracing
order_tracer = Tracer("order-service")
payment_tracer = Tracer("payment-service")
inventory_tracer = Tracer("inventory-service")

# Simulate request flow
root_span = order_tracer.start_trace("POST /orders")
root_span.set_tag("http.method", "POST")
root_span.set_tag("http.url", "/orders")
root_span.set_tag("customer_id", "CUST-001")

# Child span - validate order
validate_span = order_tracer.start_child_span(root_span, "validate_order")
time.sleep(0.01)  # จำลอง latency
validate_span.finish()

# Child span - call inventory service
inventory_span = order_tracer.start_child_span(root_span, "call_inventory_service")
inventory_span.set_tag("http.url", "http://inventory-service/reserve")

# Inventory service receives and creates its own spans
headers = TraceContext.inject(inventory_span)
trace_id, parent_span_id = TraceContext.extract(headers)

# Inventory service span
inv_reserve_span = Span("reserve_items", trace_id, parent_span_id)
inv_reserve_span.set_tag("service", "inventory-service")
time.sleep(0.02)
inv_reserve_span.log("Reserved 2 items of PROD-001")
inv_reserve_span.finish()

inventory_span.finish()

# Child span - call payment service
payment_span = order_tracer.start_child_span(root_span, "call_payment_service")
time.sleep(0.03)
payment_span.finish()

# Child span - save order
save_span = order_tracer.start_child_span(root_span, "save_order")
time.sleep(0.005)
save_span.finish()

root_span.set_tag("http.status_code", "201")
root_span.finish()

# Print trace
trace = order_tracer.get_trace(root_span.trace_id)
print(f"\n=== Distributed Trace: {root_span.trace_id} ===")
for span in trace:
    indent = "  " if span.parent_id else ""
    print(f"{indent}[{span.span_id}] {span.name}: {span.duration_ms:.1f}ms - {span.status}")
    for key, value in span.tags.items():
        print(f"{indent}  {key}={value}")
```

---

## 9. Health Checks

```python
# ตัวอย่าง 11: Health Check System
from dataclasses import dataclass, field
from typing import Callable, Dict, List
from enum import Enum
from datetime import datetime
import time

class HealthStatus(Enum):
    HEALTHY = "healthy"
    DEGRADED = "degraded"
    UNHEALTHY = "unhealthy"

@dataclass
class CheckResult:
    name: str
    status: HealthStatus
    message: str
    duration_ms: float
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())

@dataclass
class HealthReport:
    service: str
    status: HealthStatus
    checks: List[CheckResult]
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())
    
    def to_dict(self) -> dict:
        return {
            'service': self.service,
            'status': self.status.value,
            'timestamp': self.timestamp,
            'checks': [
                {
                    'name': c.name,
                    'status': c.status.value,
                    'message': c.message,
                    'duration_ms': c.duration_ms
                }
                for c in self.checks
            ]
        }

class HealthChecker:
    def __init__(self, service_name: str):
        self.service_name = service_name
        self._checks: Dict[str, Callable] = {}
    
    def register_check(self, name: str, check_fn: Callable):
        self._checks[name] = check_fn
    
    def run_all(self) -> HealthReport:
        results = []
        overall_status = HealthStatus.HEALTHY
        
        for name, check_fn in self._checks.items():
            start = time.time()
            try:
                status, message = check_fn()
                duration = (time.time() - start) * 1000
                results.append(CheckResult(name, status, message, duration))
                
                if status == HealthStatus.UNHEALTHY:
                    overall_status = HealthStatus.UNHEALTHY
                elif status == HealthStatus.DEGRADED and overall_status == HealthStatus.HEALTHY:
                    overall_status = HealthStatus.DEGRADED
            except Exception as e:
                duration = (time.time() - start) * 1000
                results.append(CheckResult(
                    name, HealthStatus.UNHEALTHY,
                    f"Check failed: {str(e)}", duration
                ))
                overall_status = HealthStatus.UNHEALTHY
        
        return HealthReport(self.service_name, overall_status, results)

# Checks implementations
def check_database() -> tuple:
    # จำลอง DB check
    try:
        time.sleep(0.001)  # จำลอง latency
        return HealthStatus.HEALTHY, "Database connected: 1ms latency"
    except Exception as e:
        return HealthStatus.UNHEALTHY, str(e)

def check_redis() -> tuple:
    # จำลอง Redis check
    return HealthStatus.HEALTHY, "Redis connected: 0.5ms latency"

def check_external_api() -> tuple:
    # จำลอง external API check
    return HealthStatus.DEGRADED, "External API: high latency (500ms)"

def check_disk_space() -> tuple:
    import shutil
    total, used, free = shutil.disk_usage("/")
    percent_used = used / total * 100
    if percent_used > 90:
        return HealthStatus.UNHEALTHY, f"Disk usage critical: {percent_used:.1f}%"
    elif percent_used > 80:
        return HealthStatus.DEGRADED, f"Disk usage high: {percent_used:.1f}%"
    return HealthStatus.HEALTHY, f"Disk usage normal: {percent_used:.1f}%"

# ทดสอบ Health Check
checker = HealthChecker("order-service")
checker.register_check("database", check_database)
checker.register_check("redis", check_redis)
checker.register_check("external_api", check_external_api)
checker.register_check("disk", check_disk_space)

report = checker.run_all()
import json
print(json.dumps(report.to_dict(), indent=2))
```

---

## 10. Container Orchestration (Kubernetes Concepts)

```python
# ตัวอย่าง 12: Kubernetes Configuration Files (Python สร้าง K8s YAML)

import yaml
from dataclasses import dataclass, field, asdict
from typing import List, Dict, Optional

def create_deployment_yaml(
    name: str,
    image: str,
    port: int,
    replicas: int = 2,
    env_vars: Dict[str, str] = None,
    memory_limit: str = "256Mi",
    cpu_limit: str = "500m"
) -> str:
    deployment = {
        'apiVersion': 'apps/v1',
        'kind': 'Deployment',
        'metadata': {
            'name': name,
            'labels': {'app': name}
        },
        'spec': {
            'replicas': replicas,
            'selector': {
                'matchLabels': {'app': name}
            },
            'template': {
                'metadata': {
                    'labels': {'app': name}
                },
                'spec': {
                    'containers': [{
                        'name': name,
                        'image': image,
                        'ports': [{'containerPort': port}],
                        'env': [
                            {'name': k, 'value': v}
                            for k, v in (env_vars or {}).items()
                        ],
                        'resources': {
                            'limits': {
                                'memory': memory_limit,
                                'cpu': cpu_limit
                            },
                            'requests': {
                                'memory': '128Mi',
                                'cpu': '100m'
                            }
                        },
                        'livenessProbe': {
                            'httpGet': {
                                'path': '/health',
                                'port': port
                            },
                            'initialDelaySeconds': 10,
                            'periodSeconds': 30
                        },
                        'readinessProbe': {
                            'httpGet': {
                                'path': '/health',
                                'port': port
                            },
                            'initialDelaySeconds': 5,
                            'periodSeconds': 10
                        }
                    }]
                }
            }
        }
    }
    return yaml.dump(deployment, default_flow_style=False)

def create_service_yaml(name: str, port: int, service_type: str = "ClusterIP") -> str:
    service = {
        'apiVersion': 'v1',
        'kind': 'Service',
        'metadata': {'name': name},
        'spec': {
            'selector': {'app': name},
            'ports': [{
                'protocol': 'TCP',
                'port': port,
                'targetPort': port
            }],
            'type': service_type
        }
    }
    return yaml.dump(service, default_flow_style=False)

# สร้าง K8s config สำหรับ user-service
print("=== Kubernetes Deployment YAML ===")
deployment_yaml = create_deployment_yaml(
    name="user-service",
    image="myapp/user-service:v1.0.0",
    port=8001,
    replicas=3,
    env_vars={
        "DATABASE_URL": "postgresql://db-service:5432/users",
        "REDIS_URL": "redis://redis-service:6379",
        "LOG_LEVEL": "INFO"
    }
)
print(deployment_yaml)

print("=== Kubernetes Service YAML ===")
service_yaml = create_service_yaml("user-service", 8001)
print(service_yaml)
```

---

## 11. Microservices E-Commerce Example

```python
# ตัวอย่าง 13: Microservices ขนาดย่อม
from typing import Optional, List
from dataclasses import dataclass, field
from datetime import datetime
import uuid

# === Shared Kernel ===
@dataclass
class ServiceEvent:
    event_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    event_type: str = ""
    source_service: str = ""
    payload: dict = field(default_factory=dict)
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())

class MessageBroker:
    """Simulated Message Broker"""
    def __init__(self):
        self._subscribers = {}
        self._messages = []
    
    def publish(self, topic: str, event: ServiceEvent):
        self._messages.append((topic, event))
        handlers = self._subscribers.get(topic, [])
        for handler in handlers:
            try:
                handler(event)
            except Exception as e:
                print(f"Handler error: {e}")
    
    def subscribe(self, topic: str, handler):
        if topic not in self._subscribers:
            self._subscribers[topic] = []
        self._subscribers[topic].append(handler)

# Global broker
broker = MessageBroker()

# === User Service ===
class UserMicroservice:
    def __init__(self):
        self._users = {}
    
    def create_user(self, username: str, email: str) -> dict:
        user_id = str(uuid.uuid4())
        user = {'id': user_id, 'username': username, 'email': email, 'active': True}
        self._users[user_id] = user
        
        broker.publish('user.created', ServiceEvent(
            event_type='UserCreated',
            source_service='user-service',
            payload={'user_id': user_id, 'email': email}
        ))
        return user
    
    def get_user(self, user_id: str) -> Optional[dict]:
        return self._users.get(user_id)

# === Product Service ===
class ProductMicroservice:
    def __init__(self):
        self._products = {}
    
    def add_product(self, name: str, price: float, stock: int) -> dict:
        product_id = str(uuid.uuid4())
        product = {'id': product_id, 'name': name, 'price': price, 'stock': stock}
        self._products[product_id] = product
        return product
    
    def reserve_stock(self, product_id: str, quantity: int) -> bool:
        product = self._products.get(product_id)
        if not product or product['stock'] < quantity:
            return False
        product['stock'] -= quantity
        broker.publish('inventory.reserved', ServiceEvent(
            event_type='StockReserved',
            source_service='product-service',
            payload={'product_id': product_id, 'quantity': quantity}
        ))
        return True
    
    def get_product(self, product_id: str) -> Optional[dict]:
        return self._products.get(product_id)

# === Order Service ===
class OrderMicroservice:
    def __init__(self, product_service: 'ProductMicroservice'):
        self._orders = {}
        self._product_service = product_service
        
        # Subscribe to events
        broker.subscribe('user.created', self._on_user_created)
    
    def _on_user_created(self, event: ServiceEvent):
        print(f"[OrderService] New user registered: {event.payload.get('email')}")
    
    def create_order(self, user_id: str, items: List[dict]) -> Optional[dict]:
        # Validate and reserve stock
        for item in items:
            if not self._product_service.reserve_stock(
                item['product_id'], item['quantity']
            ):
                return None
        
        order_id = str(uuid.uuid4())
        total = 0
        order_items = []
        for item in items:
            product = self._product_service.get_product(item['product_id'])
            subtotal = product['price'] * item['quantity']
            total += subtotal
            order_items.append({
                **item,
                'product_name': product['name'],
                'unit_price': product['price'],
                'subtotal': subtotal
            })
        
        order = {
            'id': order_id,
            'user_id': user_id,
            'items': order_items,
            'total': total,
            'status': 'pending',
            'created_at': datetime.now().isoformat()
        }
        self._orders[order_id] = order
        
        broker.publish('order.created', ServiceEvent(
            event_type='OrderCreated',
            source_service='order-service',
            payload={'order_id': order_id, 'user_id': user_id, 'total': total}
        ))
        return order

# === Notification Service ===
class NotificationMicroservice:
    def __init__(self):
        self._user_service = None
        
        broker.subscribe('user.created', self._send_welcome)
        broker.subscribe('order.created', self._send_order_confirmation)
        broker.subscribe('inventory.reserved', self._log_reservation)
    
    def set_user_service(self, user_service: UserMicroservice):
        self._user_service = user_service
    
    def _send_welcome(self, event: ServiceEvent):
        email = event.payload.get('email')
        print(f"[NotificationService] 📧 Welcome email sent to {email}")
    
    def _send_order_confirmation(self, event: ServiceEvent):
        order_id = event.payload.get('order_id')
        total = event.payload.get('total')
        print(f"[NotificationService] 📧 Order confirmation for {order_id} - Total: ${total:.2f}")
    
    def _log_reservation(self, event: ServiceEvent):
        product_id = event.payload.get('product_id')
        qty = event.payload.get('quantity')
        print(f"[NotificationService] 📦 Stock reserved: {qty} units of {product_id}")

# ทดสอบ Microservices
print("=== Microservices Demo ===")

# สร้าง services
user_svc = UserMicroservice()
product_svc = ProductMicroservice()
order_svc = OrderMicroservice(product_svc)
notification_svc = NotificationMicroservice()
notification_svc.set_user_service(user_svc)

# Register user
print("\n--- Creating User ---")
user = user_svc.create_user("alice", "alice@example.com")
print(f"Created user: {user['username']}")

# Add products
product1 = product_svc.add_product("Python Book", 599.0, 10)
product2 = product_svc.add_product("VS Code T-Shirt", 299.0, 5)

# Create order
print("\n--- Creating Order ---")
order = order_svc.create_order(
    user['id'],
    [
        {'product_id': product1['id'], 'quantity': 2},
        {'product_id': product2['id'], 'quantity': 1}
    ]
)

if order:
    print(f"Order created: {order['id']}")
    print(f"Total: ${order['total']:.2f}")
    print(f"Items: {len(order['items'])}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Service Mesh Simulation
```python
# เฉลย
from typing import Dict, Optional, Callable, Tuple
from dataclasses import dataclass, field
from datetime import datetime
import time
import random

@dataclass
class ServiceConfig:
    name: str
    host: str
    port: int
    timeout_ms: int = 5000
    max_retries: int = 3

class RetryPolicy:
    def __init__(self, max_retries: int = 3, backoff_ms: int = 100):
        self.max_retries = max_retries
        self.backoff_ms = backoff_ms
    
    def execute(self, func: Callable) -> any:
        last_error = None
        for attempt in range(self.max_retries + 1):
            try:
                return func()
            except Exception as e:
                last_error = e
                if attempt < self.max_retries:
                    wait = self.backoff_ms * (2 ** attempt) / 1000
                    print(f"  Retry {attempt+1} after {wait:.2f}s")
                    time.sleep(wait)
        raise last_error

class Sidecar:
    """Envoy-like Sidecar Proxy"""
    
    def __init__(self, service_name: str):
        self.service_name = service_name
        self._circuit_breaker = CircuitBreaker(failure_threshold=3, reset_timeout=5)
        self._retry_policy = RetryPolicy(max_retries=2, backoff_ms=50)
        self._requests_total = 0
        self._requests_failed = 0
    
    def call(self, target_service: str, func: Callable) -> any:
        self._requests_total += 1
        
        def wrapped():
            return self._circuit_breaker.call(func)
        
        try:
            result = self._retry_policy.execute(wrapped)
            return result
        except Exception as e:
            self._requests_failed += 1
            raise
    
    def metrics(self) -> dict:
        success_rate = (
            (self._requests_total - self._requests_failed) / self._requests_total * 100
            if self._requests_total > 0 else 0
        )
        return {
            'service': self.service_name,
            'requests_total': self._requests_total,
            'requests_failed': self._requests_failed,
            'success_rate': f"{success_rate:.1f}%",
            'circuit_state': self._circuit_breaker.state
        }

# ทดสอบ Service Mesh
order_sidecar = Sidecar("order-service")
payment_sidecar = Sidecar("payment-service")

call_count = [0]
def unstable_payment():
    call_count[0] += 1
    if call_count[0] % 3 != 0:  # fail 2 out of 3 times
        raise ConnectionError("Payment service temporarily unavailable")
    return {"status": "success", "transaction_id": f"TXN-{call_count[0]}"}

print("=== Service Mesh Demo ===")
for i in range(5):
    try:
        result = order_sidecar.call("payment-service", unstable_payment)
        print(f"Request {i+1}: Success - {result}")
    except Exception as e:
        print(f"Request {i+1}: Failed - {e}")

print(f"\nMetrics: {order_sidecar.metrics()}")
```

### แบบฝึกหัดที่ 2 - 8

**แบบฝึกหัดที่ 2**: สร้าง Rate Limiter แบบ Token Bucket

**แบบฝึกหัดที่ 3**: สร้าง Service Registry ที่รองรับ health check อัตโนมัติ

**แบบฝึกหัดที่ 4**: สร้าง API Versioning Middleware สำหรับ API Gateway

**แบบฝึกหัดที่ 5**: Implement Choreography-based Saga Pattern

**แบบฝึกหัดที่ 6**: สร้าง Bulkhead Pattern เพื่อ isolate failures

**แบบฝึกหัดที่ 7**: สร้าง Distributed Lock ด้วย Redis Simulation

**แบบฝึกหัดที่ 8**: สร้าง Service Mesh ที่รองรับ mTLS authentication simulation

---

## สรุป

| Pattern | วัตถุประสงค์ |
|---------|------------|
| API Gateway | Entry point เดียวสำหรับ clients |
| Service Discovery | ค้นหา services แบบ dynamic |
| Circuit Breaker | ป้องกัน cascade failures |
| Saga | Distributed transactions |
| CQRS | แยก read/write operations |
| Event Sourcing | เก็บประวัติทุก state change |
| Sidecar/Service Mesh | Cross-cutting concerns |
| Health Check | ตรวจสอบสถานะ services |

Microservices ไม่ใช่ silver bullet - ใช้เมื่อระบบซับซ้อนพอและทีมใหญ่พอ

> "Don't start with microservices. Start with a monolith, and when it becomes painful to change, extract services." - Sam Newman

---

*Part 89 เสร็จสมบูรณ์ | ต่อไป: Part 90 - Message Queues (RabbitMQ & Apache Kafka)*
