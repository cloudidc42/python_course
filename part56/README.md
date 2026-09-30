# Part 56: Flask - REST API Design

## สารบัญ

1. [REST API คืออะไร](#1-rest-api-คืออะไร)
2. [Richardson Maturity Model](#2-richardson-maturity-model)
3. [Resource-based URLs](#3-resource-based-urls)
4. [HTTP Verbs Semantics](#4-http-verbs-semantics)
5. [HTTP Status Codes ที่ถูกต้อง](#5-http-status-codes-ที่ถูกต้อง)
6. [Request/Response Format (JSON)](#6-requestresponse-format-json)
7. [Flask-RESTful Extension](#7-flask-restful-extension)
8. [API Versioning](#8-api-versioning)
9. [Pagination](#9-pagination)
10. [Filtering และ Sorting](#10-filtering-และ-sorting)
11. [Rate Limiting (Flask-Limiter)](#11-rate-limiting-flask-limiter)
12. [API Documentation (Swagger/OpenAPI)](#12-api-documentation-swaggeropenapi)
13. [Error Responses Format](#13-error-responses-format)
14. [CORS (Flask-CORS)](#14-cors-flask-cors)
15. [Testing REST APIs](#15-testing-rest-apis)
16. [ตัวอย่างโปรแกรมจริง: Complete CRUD REST API](#16-ตัวอย่างโปรแกรมจริง-complete-crud-rest-api)
17. [แบบฝึกหัด](#17-แบบฝึกหัด)

---

## 1. REST API คืออะไร

**REST** (Representational State Transfer) เป็น architectural style สำหรับการออกแบบ web services ที่ถูกเสนอโดย Roy Fielding ในปี 2000

### หลักการพื้นฐานของ REST

REST ประกอบด้วย 6 หลักการหลัก:

1. **Client-Server**: แยก client ออกจาก server ชัดเจน
2. **Stateless**: server ไม่เก็บ state ของ client ระหว่าง request
3. **Cacheable**: response สามารถ cache ได้
4. **Uniform Interface**: interface ที่สม่ำเสมอระหว่าง client และ server
5. **Layered System**: ระบบแบบ layer ที่ client ไม่รู้ว่าคุยกับ server จริงหรือ intermediary
6. **Code on Demand** (optional): server สามารถส่ง executable code ให้ client

### ทำไมต้องใช้ REST API?

```python
# ตัวอย่างที่ไม่ดี: ไม่ใช่ REST - RPC style
# GET /getUser?id=1
# POST /createUser
# POST /deleteUser?id=1
# POST /updateUserName?id=1&name=John

# ตัวอย่างที่ดี: REST style
# GET    /users/1       -> ดึงข้อมูล user 1
# POST   /users         -> สร้าง user ใหม่
# DELETE /users/1       -> ลบ user 1
# PUT    /users/1       -> อัพเดท user 1 ทั้งหมด
# PATCH  /users/1       -> อัพเดทบางส่วนของ user 1
```

---

## 2. Richardson Maturity Model

**Richardson Maturity Model (RMM)** เป็นโมเดลที่ใช้วัดความ "สุก" ของ REST API โดย Leonard Richardson แบ่งออกเป็น 4 ระดับ:

### Level 0: The Swamp of POX (Plain Old XML)

ใช้ HTTP เป็นแค่ transport layer ไม่ได้ใช้ features ของ HTTP เลย

```python
# Level 0: ทุก operation ไปที่ endpoint เดียวกัน
from flask import Flask, request, jsonify

app = Flask(__name__)

# ❌ ไม่ดี: ทุก operation รวมอยู่ใน endpoint เดียว
@app.route('/api', methods=['POST'])
def api():
    data = request.get_json()
    action = data.get('action')
    
    if action == 'getUser':
        return jsonify({'user': {'id': 1, 'name': 'John'}})
    elif action == 'createUser':
        return jsonify({'status': 'created'})
    elif action == 'deleteUser':
        return jsonify({'status': 'deleted'})
    
    return jsonify({'error': 'Unknown action'})
```

### Level 1: Resources

เริ่มแยก resources ออกจากกัน แต่ยังไม่ใช้ HTTP methods อย่างถูกต้อง

```python
# Level 1: แยก resource แต่ยังใช้ POST ทั้งหมด
@app.route('/users', methods=['POST'])
def users():
    data = request.get_json()
    action = data.get('action')
    
    if action == 'get':
        return jsonify({'user': {'id': 1, 'name': 'John'}})
    elif action == 'create':
        return jsonify({'status': 'created'})

@app.route('/products', methods=['POST'])
def products():
    # Similar pattern...
    pass
```

### Level 2: HTTP Verbs

ใช้ HTTP methods อย่างถูกต้อง (GET, POST, PUT, DELETE, PATCH)

```python
# Level 2: ใช้ HTTP verbs อย่างถูกต้อง
users_db = [
    {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
    {'id': 2, 'name': 'Bob', 'email': 'bob@example.com'},
]

@app.route('/users', methods=['GET'])
def get_users():
    # GET ใช้สำหรับดึงข้อมูล ไม่ควรมี side effects
    return jsonify({'users': users_db})

@app.route('/users', methods=['POST'])
def create_user():
    # POST ใช้สำหรับสร้างข้อมูลใหม่
    data = request.get_json()
    new_user = {'id': len(users_db) + 1, **data}
    users_db.append(new_user)
    return jsonify(new_user), 201  # 201 Created

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = next((u for u in users_db if u['id'] == user_id), None)
    if not user:
        return jsonify({'error': 'User not found'}), 404  # 404 Not Found
    return jsonify(user)

@app.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    # PUT ใช้สำหรับอัพเดทข้อมูลทั้งหมด (replace)
    user = next((u for u in users_db if u['id'] == user_id), None)
    if not user:
        return jsonify({'error': 'User not found'}), 404
    
    data = request.get_json()
    user.update(data)
    return jsonify(user)

@app.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    # DELETE ใช้สำหรับลบข้อมูล
    global users_db
    user = next((u for u in users_db if u['id'] == user_id), None)
    if not user:
        return jsonify({'error': 'User not found'}), 404
    
    users_db = [u for u in users_db if u['id'] != user_id]
    return '', 204  # 204 No Content
```

### Level 3: Hypermedia Controls (HATEOAS)

Hypermedia as the Engine of Application State - response มี links บอก client ว่าทำอะไรได้ต่อ

```python
# Level 3: HATEOAS - response มี links
@app.route('/users/<int:user_id>', methods=['GET'])
def get_user_hateoas(user_id):
    user = {'id': user_id, 'name': 'Alice', 'email': 'alice@example.com'}
    
    # เพิ่ม hypermedia links
    response = {
        'data': user,
        '_links': {
            'self': {'href': f'/api/v1/users/{user_id}', 'method': 'GET'},
            'update': {'href': f'/api/v1/users/{user_id}', 'method': 'PUT'},
            'delete': {'href': f'/api/v1/users/{user_id}', 'method': 'DELETE'},
            'orders': {'href': f'/api/v1/users/{user_id}/orders', 'method': 'GET'},
            'collection': {'href': '/api/v1/users', 'method': 'GET'},
        }
    }
    return jsonify(response)
```

---

## 3. Resource-based URLs

การออกแบบ URL สำหรับ REST API ที่ดีควรยึดหลัก resource-based

### หลักการออกแบบ URL

```python
# ✅ URL ที่ดี - noun-based, ใช้ plural
/api/v1/users              # collection ของ users
/api/v1/users/123          # user เฉพาะคน
/api/v1/users/123/orders   # orders ของ user นั้น
/api/v1/products           # collection ของ products
/api/v1/categories/5/items # items ใน category 5

# ❌ URL ที่ไม่ดี - verb-based
/api/getUsers
/api/createUser
/api/user_delete/123
/api/v1/getUserOrders?user=123
```

### ตัวอย่างการออกแบบ URL สำหรับ E-commerce API

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# === Users Resources ===
# GET    /users              -> รายชื่อ users ทั้งหมด
# POST   /users              -> สร้าง user ใหม่
# GET    /users/{id}         -> ดึง user เฉพาะคน
# PUT    /users/{id}         -> อัพเดท user ทั้งหมด
# PATCH  /users/{id}         -> อัพเดทบางส่วนของ user
# DELETE /users/{id}         -> ลบ user

# === Products Resources ===
# GET    /products            -> รายชื่อ products
# POST   /products            -> สร้าง product ใหม่
# GET    /products/{id}       -> ดึง product เฉพาะ
# PUT    /products/{id}       -> อัพเดท product
# DELETE /products/{id}       -> ลบ product

# === Nested Resources ===
# GET    /users/{id}/orders          -> orders ของ user นั้น
# POST   /users/{id}/orders          -> สร้าง order สำหรับ user
# GET    /users/{id}/orders/{oid}    -> order เฉพาะของ user

# === Orders Resources ===
# GET    /orders                     -> รายชื่อ orders ทั้งหมด
# GET    /orders/{id}                -> order เฉพาะ
# GET    /orders/{id}/items          -> items ใน order นั้น
# PUT    /orders/{id}/status         -> เปลี่ยน status ของ order

@app.route('/api/v1/users/<int:user_id>/orders')
def get_user_orders(user_id):
    """ดึง orders ทั้งหมดของ user คนนั้น"""
    # Nested resource - user must exist first
    orders = [
        {'id': 1, 'user_id': user_id, 'total': 150.00, 'status': 'completed'},
        {'id': 2, 'user_id': user_id, 'total': 89.99, 'status': 'pending'},
    ]
    return jsonify({
        'user_id': user_id,
        'orders': orders,
        'total_count': len(orders)
    })
```

### URL Naming Conventions

```python
# ✅ ใช้ lowercase
/users
/product-categories    # kebab-case สำหรับ multi-word

# ❌ ไม่ใช้ uppercase หรือ camelCase
/Users
/productCategories
/Product_Categories

# ✅ ใช้ plural nouns
/users
/products
/orders

# ❌ ไม่ใช้ singular หรือ mixed
/user
/product
/Users

# ✅ resource hierarchy ที่สมเหตุสมผล
/departments/5/employees      # employees ใน department 5

# ❌ hierarchy ที่ลึกเกินไป (ไม่เกิน 3 ระดับ)
/departments/5/teams/3/projects/8/tasks/12/comments
```

---

## 4. HTTP Verbs Semantics

### GET - ดึงข้อมูล (Safe + Idempotent)

```python
from flask import Flask, jsonify, request
app = Flask(__name__)

# In-memory database สำหรับตัวอย่าง
books = [
    {'id': 1, 'title': 'Python Crash Course', 'author': 'Eric Matthes', 'price': 35.99},
    {'id': 2, 'title': 'Fluent Python', 'author': 'Luciano Ramalho', 'price': 49.99},
    {'id': 3, 'title': 'Clean Code', 'author': 'Robert Martin', 'price': 42.00},
]

@app.route('/books', methods=['GET'])
def get_books():
    """
    GET /books
    - Safe: ไม่เปลี่ยน state ของ server
    - Idempotent: เรียกกี่ครั้งก็ได้ผลเหมือนกัน
    - Cacheable: response สามารถ cache ได้
    """
    return jsonify({
        'books': books,
        'count': len(books)
    })

@app.route('/books/<int:book_id>', methods=['GET'])
def get_book(book_id):
    """GET /books/{id} - ดึง book เฉพาะเล่ม"""
    book = next((b for b in books if b['id'] == book_id), None)
    if not book:
        return jsonify({'error': 'Book not found', 'code': 'BOOK_NOT_FOUND'}), 404
    return jsonify(book)
```

### POST - สร้างข้อมูลใหม่ (Not Safe, Not Idempotent)

```python
@app.route('/books', methods=['POST'])
def create_book():
    """
    POST /books
    - Not Safe: เปลี่ยน state ของ server (สร้างข้อมูลใหม่)
    - Not Idempotent: เรียกซ้ำจะสร้างข้อมูลซ้ำ
    """
    data = request.get_json()
    
    # Validation
    required_fields = ['title', 'author', 'price']
    missing = [f for f in required_fields if f not in data]
    if missing:
        return jsonify({
            'error': 'Missing required fields',
            'missing_fields': missing
        }), 400  # 400 Bad Request
    
    # สร้าง book ใหม่
    new_book = {
        'id': max(b['id'] for b in books) + 1 if books else 1,
        'title': data['title'],
        'author': data['author'],
        'price': float(data['price'])
    }
    books.append(new_book)
    
    # 201 Created พร้อม Location header
    response = jsonify(new_book)
    response.status_code = 201
    response.headers['Location'] = f'/books/{new_book["id"]}'
    return response
```

### PUT - อัพเดทข้อมูลทั้งหมด (Idempotent)

```python
@app.route('/books/<int:book_id>', methods=['PUT'])
def replace_book(book_id):
    """
    PUT /books/{id}
    - Idempotent: เรียกซ้ำหลายครั้งได้ผลเหมือนกัน
    - Replace ข้อมูลทั้งหมด (ต้องส่งข้อมูลครบทุก field)
    """
    book = next((b for b in books if b['id'] == book_id), None)
    if not book:
        return jsonify({'error': 'Book not found'}), 404
    
    data = request.get_json()
    
    # PUT ต้องส่งข้อมูลครบทุก field
    required_fields = ['title', 'author', 'price']
    missing = [f for f in required_fields if f not in data]
    if missing:
        return jsonify({'error': 'Missing required fields', 'fields': missing}), 400
    
    # Replace ข้อมูลทั้งหมด
    book.clear()
    book.update({
        'id': book_id,
        'title': data['title'],
        'author': data['author'],
        'price': float(data['price'])
    })
    
    return jsonify(book)
```

### PATCH - อัพเดทบางส่วน

```python
@app.route('/books/<int:book_id>', methods=['PATCH'])
def update_book(book_id):
    """
    PATCH /books/{id}
    - อัพเดทเฉพาะ fields ที่ส่งมา
    - ไม่ต้องส่งข้อมูลครบทุก field
    """
    book = next((b for b in books if b['id'] == book_id), None)
    if not book:
        return jsonify({'error': 'Book not found'}), 404
    
    data = request.get_json()
    
    # อนุญาตให้อัพเดทเฉพาะ fields ที่ส่งมา
    allowed_fields = ['title', 'author', 'price']
    for field in allowed_fields:
        if field in data:
            book[field] = data[field]
    
    return jsonify(book)
```

### DELETE - ลบข้อมูล

```python
@app.route('/books/<int:book_id>', methods=['DELETE'])
def delete_book(book_id):
    """
    DELETE /books/{id}
    - Idempotent: เรียกซ้ำควรได้ผลเหมือนกัน
    - 204 No Content เมื่อลบสำเร็จ
    - บางครั้งอาจ return 200 พร้อม confirmation message
    """
    global books
    book = next((b for b in books if b['id'] == book_id), None)
    if not book:
        return jsonify({'error': 'Book not found'}), 404
    
    books = [b for b in books if b['id'] != book_id]
    return '', 204  # 204 No Content
```

---

## 5. HTTP Status Codes ที่ถูกต้อง

### 2xx - Success

```python
from flask import jsonify, make_response

def demo_success_codes():
    # 200 OK - Request สำเร็จ (GET, PUT, PATCH)
    response_200 = jsonify({'data': 'success'})
    response_200.status_code = 200
    
    # 201 Created - Resource ถูกสร้าง (POST)
    response_201 = jsonify({'id': 1, 'name': 'New Resource'})
    response_201.status_code = 201
    response_201.headers['Location'] = '/resources/1'
    
    # 202 Accepted - Request รับแล้ว แต่ยังไม่ complete (async operations)
    response_202 = jsonify({'message': 'Processing started', 'job_id': 'abc123'})
    response_202.status_code = 202
    
    # 204 No Content - สำเร็จแต่ไม่มี body (DELETE)
    response_204 = make_response('', 204)

# ตัวอย่าง endpoint ที่ใช้ status codes ถูกต้อง
@app.route('/books', methods=['POST'])
def create_book_v2():
    data = request.get_json()
    
    # สร้าง book
    new_book = {'id': 100, **data}
    
    # Return 201 Created พร้อม Location header
    return jsonify(new_book), 201, {
        'Location': f'/api/v1/books/{new_book["id"]}'
    }

@app.route('/books/<int:book_id>/publish', methods=['POST'])
def publish_book(book_id):
    """Long-running operation ที่ process แบบ async"""
    # ส่งงานไปทำ background
    job_id = 'job_abc123'
    
    return jsonify({
        'message': 'Book publishing started',
        'job_id': job_id,
        'status_url': f'/jobs/{job_id}'
    }), 202  # 202 Accepted
```

### 3xx - Redirection

```python
from flask import redirect

@app.route('/old-books', methods=['GET'])
def old_endpoint():
    """301 Moved Permanently - redirect ไปที่ URL ใหม่"""
    return redirect('/api/v1/books', code=301)

@app.route('/books/search', methods=['GET'])  
def search_redirect():
    """302 Found - temporary redirect"""
    query = request.args.get('q', '')
    return redirect(f'/api/v1/books?search={query}', code=302)
```

### 4xx - Client Errors

```python
@app.route('/books/<int:book_id>', methods=['GET'])
def get_book_with_codes(book_id):
    book = next((b for b in books if b['id'] == book_id), None)
    
    # 404 Not Found
    if not book:
        return jsonify({
            'error': 'Book not found',
            'code': 'RESOURCE_NOT_FOUND',
            'message': f'Book with id {book_id} does not exist'
        }), 404
    
    return jsonify(book), 200

@app.route('/admin/books', methods=['GET'])
def admin_books():
    # 401 Unauthorized - ไม่ได้ authentication
    if not request.headers.get('Authorization'):
        return jsonify({
            'error': 'Authentication required',
            'code': 'UNAUTHORIZED'
        }), 401
    
    # 403 Forbidden - authentication แล้ว แต่ไม่มีสิทธิ์
    token = request.headers.get('Authorization')
    if token != 'Bearer admin-token':
        return jsonify({
            'error': 'Insufficient permissions',
            'code': 'FORBIDDEN'
        }), 403
    
    return jsonify({'books': books})

# ตาราง Status Codes สำคัญ
STATUS_CODE_GUIDE = {
    # 2xx Success
    200: 'OK - Generic success',
    201: 'Created - Resource created successfully',
    202: 'Accepted - Async operation started',
    204: 'No Content - Success with no response body',
    
    # 3xx Redirection
    301: 'Moved Permanently',
    302: 'Found (Temporary Redirect)',
    304: 'Not Modified (Cached)',
    
    # 4xx Client Errors
    400: 'Bad Request - Invalid syntax/data',
    401: 'Unauthorized - Not authenticated',
    403: 'Forbidden - Authenticated but no permission',
    404: 'Not Found - Resource not found',
    405: 'Method Not Allowed',
    409: 'Conflict - Resource conflict (duplicate)',
    422: 'Unprocessable Entity - Validation error',
    429: 'Too Many Requests - Rate limit exceeded',
    
    # 5xx Server Errors
    500: 'Internal Server Error',
    502: 'Bad Gateway',
    503: 'Service Unavailable',
    504: 'Gateway Timeout',
}
```

---

## 6. Request/Response Format (JSON)

### Standard Response Format

```python
from flask import Flask, jsonify, request
from datetime import datetime
import traceback

app = Flask(__name__)

def success_response(data, message='Success', status_code=200, meta=None):
    """Helper function สำหรับ success response"""
    response = {
        'success': True,
        'message': message,
        'data': data,
        'timestamp': datetime.utcnow().isoformat() + 'Z'
    }
    if meta:
        response['meta'] = meta
    return jsonify(response), status_code

def error_response(code, message, details=None, status_code=400):
    """Helper function สำหรับ error response"""
    response = {
        'success': False,
        'error': {
            'code': code,
            'message': message,
        },
        'timestamp': datetime.utcnow().isoformat() + 'Z'
    }
    if details:
        response['error']['details'] = details
    return jsonify(response), status_code

# ตัวอย่างการใช้งาน
@app.route('/api/v1/users', methods=['GET'])
def list_users():
    users = [
        {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
        {'id': 2, 'name': 'Bob', 'email': 'bob@example.com'},
    ]
    
    return success_response(
        data={'users': users},
        meta={
            'total': len(users),
            'page': 1,
            'per_page': 10
        }
    )

@app.route('/api/v1/users/<int:user_id>', methods=['GET'])
def get_user_v2(user_id):
    # Simulate not found
    if user_id > 100:
        return error_response(
            code='USER_NOT_FOUND',
            message=f'User with id {user_id} does not exist',
            status_code=404
        )
    
    user = {'id': user_id, 'name': 'Alice', 'email': 'alice@example.com'}
    return success_response(data={'user': user})
```

### Request Validation

```python
from functools import wraps

def require_json(f):
    """Decorator ตรวจสอบว่า request มี Content-Type: application/json"""
    @wraps(f)
    def decorated(*args, **kwargs):
        if not request.is_json:
            return error_response(
                code='INVALID_CONTENT_TYPE',
                message='Content-Type must be application/json',
                status_code=415  # Unsupported Media Type
            )
        return f(*args, **kwargs)
    return decorated

def validate_schema(schema):
    """Decorator สำหรับ validate request body"""
    def decorator(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            data = request.get_json()
            errors = {}
            
            for field, rules in schema.items():
                if rules.get('required') and field not in data:
                    errors[field] = 'This field is required'
                elif field in data:
                    value = data[field]
                    if 'type' in rules and not isinstance(value, rules['type']):
                        errors[field] = f'Must be of type {rules["type"].__name__}'
                    if 'min_length' in rules and len(str(value)) < rules['min_length']:
                        errors[field] = f'Minimum length is {rules["min_length"]}'
                    if 'max_length' in rules and len(str(value)) > rules['max_length']:
                        errors[field] = f'Maximum length is {rules["max_length"]}'
            
            if errors:
                return error_response(
                    code='VALIDATION_ERROR',
                    message='Request validation failed',
                    details=errors,
                    status_code=422
                )
            
            return f(*args, **kwargs)
        return decorated
    return decorator

# Schema สำหรับ user creation
USER_SCHEMA = {
    'name': {'required': True, 'type': str, 'min_length': 2, 'max_length': 100},
    'email': {'required': True, 'type': str},
    'age': {'required': False, 'type': int},
}

@app.route('/api/v1/users', methods=['POST'])
@require_json
@validate_schema(USER_SCHEMA)
def create_user_validated():
    data = request.get_json()
    new_user = {'id': 100, **data}
    return success_response(data={'user': new_user}, status_code=201)
```

---

## 7. Flask-RESTful Extension

Flask-RESTful ทำให้การสร้าง REST API ง่ายขึ้นโดยใช้ class-based views

### การติดตั้ง

```bash
pip install flask flask-restful
```

### การใช้งานพื้นฐาน

```python
from flask import Flask
from flask_restful import Api, Resource, reqparse, fields, marshal_with

app = Flask(__name__)
api = Api(app)

# In-memory database
products = [
    {'id': 1, 'name': 'Laptop', 'price': 999.99, 'stock': 50},
    {'id': 2, 'name': 'Mouse', 'price': 29.99, 'stock': 200},
    {'id': 3, 'name': 'Keyboard', 'price': 49.99, 'stock': 150},
]

# Output fields - กำหนด format ของ response
product_fields = {
    'id': fields.Integer,
    'name': fields.String,
    'price': fields.Float,
    'stock': fields.Integer,
    'uri': fields.Url('product'),  # auto-generate URL
}

# Parser สำหรับ request arguments
product_parser = reqparse.RequestParser()
product_parser.add_argument('name', type=str, required=True, help='Product name is required')
product_parser.add_argument('price', type=float, required=True, help='Product price is required')
product_parser.add_argument('stock', type=int, default=0)

class ProductList(Resource):
    """Resource สำหรับ /products"""
    
    @marshal_with(product_fields)
    def get(self):
        """GET /products - ดึง products ทั้งหมด"""
        return products
    
    @marshal_with(product_fields)
    def post(self):
        """POST /products - สร้าง product ใหม่"""
        args = product_parser.parse_args()
        
        new_product = {
            'id': max(p['id'] for p in products) + 1 if products else 1,
            'name': args['name'],
            'price': args['price'],
            'stock': args['stock']
        }
        products.append(new_product)
        return new_product, 201

class ProductDetail(Resource):
    """Resource สำหรับ /products/{id}"""
    
    @marshal_with(product_fields)
    def get(self, product_id):
        """GET /products/{id}"""
        product = next((p for p in products if p['id'] == product_id), None)
        if not product:
            api.abort(404, message=f'Product {product_id} not found')
        return product
    
    @marshal_with(product_fields)
    def put(self, product_id):
        """PUT /products/{id}"""
        product = next((p for p in products if p['id'] == product_id), None)
        if not product:
            api.abort(404, message=f'Product {product_id} not found')
        
        args = product_parser.parse_args()
        product.update({
            'name': args['name'],
            'price': args['price'],
            'stock': args['stock']
        })
        return product
    
    def delete(self, product_id):
        """DELETE /products/{id}"""
        global products
        product = next((p for p in products if p['id'] == product_id), None)
        if not product:
            api.abort(404, message=f'Product {product_id} not found')
        
        products = [p for p in products if p['id'] != product_id]
        return '', 204

# Register routes
api.add_resource(ProductList, '/api/v1/products', endpoint='product_list')
api.add_resource(ProductDetail, '/api/v1/products/<int:product_id>', endpoint='product')

if __name__ == '__main__':
    app.run(debug=True)
```

### Flask-RESTful Nested Resources

```python
from flask_restful import Api, Resource

# Nested resource - orders ภายใต้ users
class UserOrderList(Resource):
    def get(self, user_id):
        """GET /users/{user_id}/orders"""
        orders = [
            {'id': 1, 'user_id': user_id, 'total': 99.99, 'status': 'pending'},
            {'id': 2, 'user_id': user_id, 'total': 149.99, 'status': 'completed'},
        ]
        return {'user_id': user_id, 'orders': orders}
    
    def post(self, user_id):
        """POST /users/{user_id}/orders"""
        data = reqparse.RequestParser()
        data.add_argument('items', type=list, location='json', required=True)
        args = data.parse_args()
        
        new_order = {
            'id': 100,
            'user_id': user_id,
            'items': args['items'],
            'status': 'pending'
        }
        return new_order, 201

class UserOrderDetail(Resource):
    def get(self, user_id, order_id):
        """GET /users/{user_id}/orders/{order_id}"""
        return {'order_id': order_id, 'user_id': user_id, 'status': 'pending'}

# Register nested routes
api.add_resource(UserOrderList, '/api/v1/users/<int:user_id>/orders')
api.add_resource(UserOrderDetail, '/api/v1/users/<int:user_id>/orders/<int:order_id>')
```

---

## 8. API Versioning

### Versioning Strategies

```python
from flask import Flask, Blueprint, jsonify

app = Flask(__name__)

# === Strategy 1: URL Path Versioning ===
# /api/v1/users, /api/v2/users
# ✅ ชัดเจน, ง่ายต่อการ cache
# ❌ URL เปลี่ยนเมื่อ version เปลี่ยน

v1 = Blueprint('v1', __name__, url_prefix='/api/v1')
v2 = Blueprint('v2', __name__, url_prefix='/api/v2')

@v1.route('/users')
def users_v1():
    """Version 1: ส่งกลับ basic user info"""
    return jsonify({
        'users': [
            {'id': 1, 'name': 'Alice'},
            {'id': 2, 'name': 'Bob'},
        ]
    })

@v2.route('/users')
def users_v2():
    """Version 2: ส่งกลับ detailed user info with pagination"""
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 10, type=int)
    
    users = [
        {'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'role': 'admin'},
        {'id': 2, 'name': 'Bob', 'email': 'bob@example.com', 'role': 'user'},
    ]
    
    return jsonify({
        'users': users,
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total': len(users),
            'pages': 1
        }
    })

app.register_blueprint(v1)
app.register_blueprint(v2)
```

### Header-based Versioning

```python
# === Strategy 2: Header Versioning ===
# Accept: application/vnd.myapi.v2+json
# ❌ URL ดูเหมือน REST compliant แต่ version ซ่อนอยู่ใน header

from flask import request, jsonify

@app.route('/api/users')
def users_versioned():
    """Handle multiple versions through Accept header"""
    accept = request.headers.get('Accept', '')
    
    if 'v2' in accept:
        # Version 2 response
        return jsonify({
            'version': '2.0',
            'users': [
                {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
            ],
            'meta': {'total': 1, 'page': 1}
        })
    else:
        # Default: Version 1 response
        return jsonify({
            'version': '1.0',
            'users': [
                {'id': 1, 'name': 'Alice'},
            ]
        })
```

### Query Parameter Versioning

```python
# === Strategy 3: Query Parameter Versioning ===
# /api/users?version=2
# ❌ query parameter ควรใช้สำหรับ filtering ไม่ใช่ versioning

@app.route('/api/users/versioned')
def users_query_versioned():
    version = request.args.get('version', '1')
    
    if version == '2':
        return jsonify({'version': 2, 'users': [], 'total': 0})
    
    return jsonify({'version': 1, 'users': []})
```

### Best Practice: URL Path Versioning with Blueprint

```python
# แนะนำ: ใช้ Blueprint สำหรับ versioning
from flask import Flask, Blueprint, jsonify, request

def create_app():
    app = Flask(__name__)
    
    # Register versioned blueprints
    from .api.v1 import create_v1_blueprint
    from .api.v2 import create_v2_blueprint
    
    app.register_blueprint(create_v1_blueprint(), url_prefix='/api/v1')
    app.register_blueprint(create_v2_blueprint(), url_prefix='/api/v2')
    
    return app

# api/v1/__init__.py
def create_v1_blueprint():
    v1 = Blueprint('api_v1', __name__)
    
    @v1.route('/users')
    def get_users():
        return jsonify({'users': [], 'version': '1'})
    
    @v1.route('/products')
    def get_products():
        return jsonify({'products': [], 'version': '1'})
    
    return v1
```

---

## 9. Pagination

### Cursor-based Pagination

```python
from flask import Flask, jsonify, request
import math

app = Flask(__name__)

# Mock large dataset
def generate_users(count=1000):
    return [
        {
            'id': i,
            'name': f'User {i}',
            'email': f'user{i}@example.com',
            'created_at': f'2024-01-{(i % 28) + 1:02d}'
        }
        for i in range(1, count + 1)
    ]

ALL_USERS = generate_users(100)

@app.route('/api/v1/users')
def paginated_users():
    """
    Pagination ผ่าน query parameters:
    - page: หน้าที่ต้องการ (default: 1)
    - per_page: จำนวน items ต่อหน้า (default: 10, max: 100)
    """
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)  # max 100
    
    # Validate parameters
    if page < 1:
        return jsonify({'error': 'Page must be >= 1'}), 400
    if per_page < 1:
        return jsonify({'error': 'per_page must be >= 1'}), 400
    
    # Calculate pagination
    total = len(ALL_USERS)
    total_pages = math.ceil(total / per_page)
    start_idx = (page - 1) * per_page
    end_idx = start_idx + per_page
    
    users_page = ALL_USERS[start_idx:end_idx]
    
    # Build pagination links (HATEOAS)
    base_url = '/api/v1/users'
    links = {
        'self': f'{base_url}?page={page}&per_page={per_page}',
        'first': f'{base_url}?page=1&per_page={per_page}',
        'last': f'{base_url}?page={total_pages}&per_page={per_page}',
    }
    
    if page > 1:
        links['prev'] = f'{base_url}?page={page-1}&per_page={per_page}'
    if page < total_pages:
        links['next'] = f'{base_url}?page={page+1}&per_page={per_page}'
    
    return jsonify({
        'data': users_page,
        'pagination': {
            'current_page': page,
            'per_page': per_page,
            'total_items': total,
            'total_pages': total_pages,
            'has_next': page < total_pages,
            'has_prev': page > 1
        },
        '_links': links
    })
```

### Cursor-based Pagination (สำหรับ real-time data)

```python
import base64
import json
from datetime import datetime

def encode_cursor(data):
    """Encode cursor เป็น base64 string"""
    return base64.b64encode(json.dumps(data).encode()).decode()

def decode_cursor(cursor_str):
    """Decode cursor จาก base64 string"""
    try:
        return json.loads(base64.b64decode(cursor_str.encode()).decode())
    except Exception:
        return None

@app.route('/api/v1/users/cursor')
def cursor_paginated_users():
    """
    Cursor-based pagination:
    - ไม่มีปัญหากับ real-time inserts
    - เหมาะกับ infinite scroll
    """
    limit = min(request.args.get('limit', 10, type=int), 100)
    cursor_str = request.args.get('cursor')
    
    # Parse cursor
    after_id = 0
    if cursor_str:
        cursor_data = decode_cursor(cursor_str)
        if cursor_data:
            after_id = cursor_data.get('id', 0)
    
    # Filter users after cursor
    filtered_users = [u for u in ALL_USERS if u['id'] > after_id]
    users_page = filtered_users[:limit]
    has_more = len(filtered_users) > limit
    
    # Build next cursor
    next_cursor = None
    if has_more and users_page:
        next_cursor = encode_cursor({'id': users_page[-1]['id']})
    
    return jsonify({
        'data': users_page,
        'meta': {
            'limit': limit,
            'has_more': has_more,
        },
        'cursors': {
            'next': next_cursor
        }
    })
```

---

## 10. Filtering และ Sorting

```python
from flask import Flask, jsonify, request
from functools import reduce
import operator

app = Flask(__name__)

# Mock dataset
PRODUCTS = [
    {'id': 1, 'name': 'Laptop Pro', 'category': 'electronics', 'price': 1299.99, 'rating': 4.5, 'stock': 50},
    {'id': 2, 'name': 'Gaming Mouse', 'category': 'electronics', 'price': 59.99, 'rating': 4.2, 'stock': 200},
    {'id': 3, 'name': 'Python Book', 'category': 'books', 'price': 39.99, 'rating': 4.8, 'stock': 100},
    {'id': 4, 'name': 'Desk Chair', 'category': 'furniture', 'price': 299.99, 'rating': 4.0, 'stock': 30},
    {'id': 5, 'name': 'USB Hub', 'category': 'electronics', 'price': 24.99, 'rating': 3.9, 'stock': 500},
    {'id': 6, 'name': 'Clean Code Book', 'category': 'books', 'price': 44.99, 'rating': 4.7, 'stock': 75},
]

@app.route('/api/v1/products')
def search_products():
    """
    Filtering และ Sorting ผ่าน query parameters:
    
    Filtering:
    - ?category=electronics        -> filter by category
    - ?min_price=10&max_price=100  -> filter by price range
    - ?min_rating=4.0              -> filter by minimum rating
    - ?search=laptop               -> search in name
    - ?in_stock=true               -> filter in-stock items
    
    Sorting:
    - ?sort=price                  -> sort by price ascending
    - ?sort=-price                 -> sort by price descending
    - ?sort=name,-price            -> sort by name asc, then price desc
    
    Pagination:
    - ?page=1&per_page=10
    """
    products = PRODUCTS.copy()
    
    # === Filtering ===
    
    # Filter by category
    category = request.args.get('category')
    if category:
        products = [p for p in products if p['category'].lower() == category.lower()]
    
    # Filter by price range
    min_price = request.args.get('min_price', type=float)
    max_price = request.args.get('max_price', type=float)
    if min_price is not None:
        products = [p for p in products if p['price'] >= min_price]
    if max_price is not None:
        products = [p for p in products if p['price'] <= max_price]
    
    # Filter by minimum rating
    min_rating = request.args.get('min_rating', type=float)
    if min_rating is not None:
        products = [p for p in products if p['rating'] >= min_rating]
    
    # Search in name (case-insensitive)
    search = request.args.get('search', '').strip()
    if search:
        products = [p for p in products if search.lower() in p['name'].lower()]
    
    # Filter in-stock
    in_stock = request.args.get('in_stock')
    if in_stock == 'true':
        products = [p for p in products if p['stock'] > 0]
    
    # === Sorting ===
    sort_param = request.args.get('sort', 'id')
    sort_fields = sort_param.split(',')
    
    for field in reversed(sort_fields):
        reverse = field.startswith('-')
        field_name = field.lstrip('-')
        
        if field_name in ['id', 'name', 'price', 'rating', 'stock']:
            try:
                products.sort(key=lambda x: x[field_name], reverse=reverse)
            except (KeyError, TypeError):
                pass  # ข้ามถ้า field ไม่ถูกต้อง
    
    # === Pagination ===
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 100)
    
    total = len(products)
    start = (page - 1) * per_page
    end = start + per_page
    page_products = products[start:end]
    
    # Build response with applied filters info
    applied_filters = {}
    if category:
        applied_filters['category'] = category
    if min_price:
        applied_filters['min_price'] = min_price
    if max_price:
        applied_filters['max_price'] = max_price
    if search:
        applied_filters['search'] = search
    
    return jsonify({
        'data': page_products,
        'meta': {
            'total': total,
            'page': page,
            'per_page': per_page,
            'total_pages': (total + per_page - 1) // per_page,
            'applied_filters': applied_filters,
            'sort': sort_param,
        }
    })
```

---

## 11. Rate Limiting (Flask-Limiter)

### การติดตั้ง

```bash
pip install flask-limiter
```

### การใช้งาน Flask-Limiter

```python
from flask import Flask, jsonify, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

# กำหนด rate limiter
limiter = Limiter(
    app=app,
    key_func=get_remote_address,  # จำกัดตาม IP address
    default_limits=['200 per day', '50 per hour'],
    storage_uri='memory://',  # ใช้ Redis ใน production: 'redis://localhost:6379'
)

@app.route('/api/v1/public/search')
@limiter.limit('10 per minute')  # จำกัด 10 requests ต่อนาที
def public_search():
    """Public endpoint ที่มี strict rate limit"""
    query = request.args.get('q', '')
    return jsonify({'results': [], 'query': query})

@app.route('/api/v1/auth/login', methods=['POST'])
@limiter.limit('5 per minute; 20 per hour')  # จำกัด login attempts
def login():
    """Login endpoint - จำกัดเพื่อป้องกัน brute force"""
    data = request.get_json()
    username = data.get('username')
    
    # Simulate authentication
    if username == 'admin' and data.get('password') == 'secret':
        return jsonify({'token': 'fake-jwt-token'})
    
    return jsonify({'error': 'Invalid credentials'}), 401

@app.route('/api/v1/data/export')
@limiter.limit('2 per hour')  # จำกัด export
def export_data():
    """Heavy endpoint ที่ควรจำกัด"""
    return jsonify({'data': 'Large dataset...'})

# Custom rate limit key (ตาม API key แทน IP)
def get_api_key():
    """ใช้ API key เป็น rate limit key"""
    return request.headers.get('X-API-Key', get_remote_address())

premium_limiter = Limiter(
    app=app,
    key_func=get_api_key,
    default_limits=['1000 per day', '100 per hour']
)

@app.route('/api/v1/premium/data')
@premium_limiter.limit('50 per minute')
def premium_data():
    """Premium endpoint สำหรับ authenticated users"""
    api_key = request.headers.get('X-API-Key')
    if not api_key:
        return jsonify({'error': 'API key required'}), 401
    return jsonify({'premium_data': '...'})

# Error handler สำหรับ rate limit exceeded
@app.errorhandler(429)
def ratelimit_handler(e):
    return jsonify({
        'error': 'Rate limit exceeded',
        'code': 'TOO_MANY_REQUESTS',
        'message': str(e.description),
        'retry_after': e.retry_after if hasattr(e, 'retry_after') else None
    }), 429
```

### Rate Limiting Response Headers

```python
@app.after_request
def add_rate_limit_headers(response):
    """เพิ่ม rate limit info ใน response headers"""
    # หมายเหตุ: Flask-Limiter จะเพิ่ม headers เหล่านี้โดยอัตโนมัติ
    # X-RateLimit-Limit: จำนวน requests ที่อนุญาต
    # X-RateLimit-Remaining: requests ที่เหลือ
    # X-RateLimit-Reset: เวลาที่ limit จะ reset
    return response
```

---

## 12. API Documentation (Swagger/OpenAPI)

### การใช้ flask-openapi3

```bash
pip install flask-openapi3
```

```python
from flask_openapi3 import OpenAPI, Info, Tag
from pydantic import BaseModel, Field
from typing import Optional, List

# สร้าง app ด้วย OpenAPI support
info = Info(
    title='Book Store API',
    version='1.0.0',
    description='REST API สำหรับ book store',
)

app = OpenAPI(__name__, info=info)

# Define tags
book_tag = Tag(name='books', description='Book operations')
user_tag = Tag(name='users', description='User operations')

# Pydantic models สำหรับ request/response
class BookCreate(BaseModel):
    title: str = Field(..., description='ชื่อหนังสือ', example='Python Programming')
    author: str = Field(..., description='ชื่อผู้แต่ง', example='John Doe')
    price: float = Field(..., gt=0, description='ราคา (บาท)', example=299.00)
    isbn: Optional[str] = Field(None, description='ISBN', example='978-3-16-148410-0')

class BookResponse(BaseModel):
    id: int
    title: str
    author: str
    price: float
    isbn: Optional[str]

class BookListResponse(BaseModel):
    books: List[BookResponse]
    total: int
    page: int
    per_page: int

# Path parameters
class BookPath(BaseModel):
    book_id: int = Field(..., description='Book ID')

# Query parameters
class BookQuery(BaseModel):
    page: Optional[int] = Field(1, ge=1, description='หน้าที่ต้องการ')
    per_page: Optional[int] = Field(10, ge=1, le=100, description='จำนวนต่อหน้า')
    search: Optional[str] = Field(None, description='ค้นหาชื่อหนังสือ')

@app.get('/api/v1/books', tags=[book_tag], responses={200: BookListResponse})
def list_books(query: BookQuery):
    """
    ดึงรายการหนังสือทั้งหมด
    
    สามารถ filter และ paginate ได้
    """
    books = [
        BookResponse(id=1, title='Python Programming', author='John', price=299.00, isbn=None)
    ]
    return BookListResponse(
        books=books,
        total=1,
        page=query.page,
        per_page=query.per_page
    ).dict(), 200

@app.get('/api/v1/books/<int:book_id>', tags=[book_tag], responses={200: BookResponse})
def get_book(path: BookPath):
    """ดึงข้อมูลหนังสือตาม ID"""
    book = BookResponse(
        id=path.book_id,
        title='Python Programming',
        author='John',
        price=299.00,
        isbn=None
    )
    return book.dict(), 200

@app.post('/api/v1/books', tags=[book_tag], responses={201: BookResponse})
def create_book(body: BookCreate):
    """สร้างหนังสือใหม่"""
    new_book = BookResponse(id=100, **body.dict())
    return new_book.dict(), 201

if __name__ == '__main__':
    # เข้าถึง Swagger UI ที่ /openapi/swagger
    # เข้าถึง ReDoc ที่ /openapi/redoc
    app.run(debug=True)
```

### Flasgger (Alternative)

```bash
pip install flasgger
```

```python
from flask import Flask, jsonify
from flasgger import Swagger, swag_from

app = Flask(__name__)

swagger_config = {
    'headers': [],
    'specs': [
        {
            'endpoint': 'apispec',
            'route': '/apispec.json',
            'rule_filter': lambda rule: True,
            'model_filter': lambda tag: True,
        }
    ],
    'static_url_path': '/flasgger_static',
    'swagger_ui': True,
    'specs_route': '/docs',
}

swagger_template = {
    'swagger': '2.0',
    'info': {
        'title': 'My API',
        'description': 'API Documentation',
        'version': '1.0'
    },
    'basePath': '/api/v1',
    'schemes': ['http', 'https'],
}

swagger = Swagger(app, config=swagger_config, template=swagger_template)

@app.route('/api/v1/users/<int:user_id>', methods=['GET'])
def get_user_documented(user_id):
    """
    ดึงข้อมูล user
    ---
    tags:
      - users
    parameters:
      - name: user_id
        in: path
        type: integer
        required: true
        description: User ID
    responses:
      200:
        description: User data
        schema:
          type: object
          properties:
            id:
              type: integer
              example: 1
            name:
              type: string
              example: Alice
            email:
              type: string
              example: alice@example.com
      404:
        description: User not found
    """
    user = {'id': user_id, 'name': 'Alice', 'email': 'alice@example.com'}
    return jsonify(user)
```

---

## 13. Error Responses Format

### Standard Error Format

```python
from flask import Flask, jsonify, request
from enum import Enum

app = Flask(__name__)

class ErrorCode(Enum):
    # General
    INTERNAL_ERROR = 'INTERNAL_ERROR'
    VALIDATION_ERROR = 'VALIDATION_ERROR'
    NOT_FOUND = 'NOT_FOUND'
    
    # Auth
    UNAUTHORIZED = 'UNAUTHORIZED'
    FORBIDDEN = 'FORBIDDEN'
    TOKEN_EXPIRED = 'TOKEN_EXPIRED'
    INVALID_TOKEN = 'INVALID_TOKEN'
    
    # Rate Limiting
    RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED'
    
    # Business Logic
    INSUFFICIENT_STOCK = 'INSUFFICIENT_STOCK'
    DUPLICATE_EMAIL = 'DUPLICATE_EMAIL'
    INVALID_ORDER_STATUS = 'INVALID_ORDER_STATUS'

class APIError(Exception):
    """Base class สำหรับ API errors"""
    
    def __init__(self, code: ErrorCode, message: str, status_code: int = 400, details=None):
        self.code = code
        self.message = message
        self.status_code = status_code
        self.details = details
        super().__init__(message)
    
    def to_dict(self):
        error_dict = {
            'success': False,
            'error': {
                'code': self.code.value,
                'message': self.message,
            }
        }
        if self.details:
            error_dict['error']['details'] = self.details
        return error_dict

class NotFoundError(APIError):
    def __init__(self, resource: str, resource_id=None):
        message = f'{resource} not found'
        if resource_id:
            message = f'{resource} with id {resource_id} not found'
        super().__init__(ErrorCode.NOT_FOUND, message, 404)

class ValidationError(APIError):
    def __init__(self, details: dict):
        super().__init__(
            ErrorCode.VALIDATION_ERROR,
            'Request validation failed',
            422,
            details
        )

class DuplicateError(APIError):
    def __init__(self, resource: str, field: str):
        super().__init__(
            ErrorCode.DUPLICATE_EMAIL,  # หรือ code ที่เหมาะสม
            f'{resource} with this {field} already exists',
            409
        )

# Register error handlers
@app.errorhandler(APIError)
def handle_api_error(error):
    return jsonify(error.to_dict()), error.status_code

@app.errorhandler(404)
def handle_404(error):
    return jsonify({
        'success': False,
        'error': {
            'code': 'NOT_FOUND',
            'message': 'The requested URL was not found'
        }
    }), 404

@app.errorhandler(405)
def handle_405(error):
    return jsonify({
        'success': False,
        'error': {
            'code': 'METHOD_NOT_ALLOWED',
            'message': f'Method not allowed. Allowed: {error.valid_methods}'
        }
    }), 405

@app.errorhandler(500)
def handle_500(error):
    return jsonify({
        'success': False,
        'error': {
            'code': 'INTERNAL_ERROR',
            'message': 'An internal server error occurred'
        }
    }), 500

# ตัวอย่างการใช้ custom errors
@app.route('/api/v1/users/<int:user_id>')
def get_user_safe(user_id):
    users = {1: {'id': 1, 'name': 'Alice'}, 2: {'id': 2, 'name': 'Bob'}}
    
    if user_id not in users:
        raise NotFoundError('User', user_id)
    
    return jsonify({'success': True, 'data': users[user_id]})

@app.route('/api/v1/users', methods=['POST'])
def create_user_safe():
    data = request.get_json()
    
    # Validation
    errors = {}
    if not data.get('name'):
        errors['name'] = 'Name is required'
    if not data.get('email'):
        errors['email'] = 'Email is required'
    elif '@' not in data['email']:
        errors['email'] = 'Invalid email format'
    
    if errors:
        raise ValidationError(errors)
    
    # Check duplicate
    if data['email'] == 'existing@example.com':
        raise DuplicateError('User', 'email')
    
    return jsonify({'success': True, 'data': {'id': 100, **data}}), 201
```

---

## 14. CORS (Flask-CORS)

### การติดตั้งและการใช้งาน

```bash
pip install flask-cors
```

```python
from flask import Flask, jsonify
from flask_cors import CORS

app = Flask(__name__)

# === Basic CORS - อนุญาตทุก origin ===
# CORS(app)  # ไม่ควรใช้ใน production

# === Production CORS - จำกัด origins ===
CORS(app, resources={
    r'/api/*': {
        'origins': ['https://yourdomain.com', 'https://app.yourdomain.com'],
        'methods': ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
        'allow_headers': ['Content-Type', 'Authorization', 'X-API-Key'],
        'expose_headers': ['X-Total-Count', 'X-RateLimit-Remaining'],
        'max_age': 600,  # Preflight cache 10 นาที
        'supports_credentials': True,  # Allow cookies/auth headers
    }
})

# === Per-route CORS ===
from flask_cors import cross_origin

@app.route('/api/v1/public/data')
@cross_origin(origins=['*'])  # Public endpoint
def public_data():
    return jsonify({'data': 'This is public'})

@app.route('/api/v1/private/data')
@cross_origin(
    origins=['https://trusted-domain.com'],
    allow_headers=['Authorization'],
    supports_credentials=True
)
def private_data():
    return jsonify({'data': 'This is private'})

# === Manual CORS headers (ถ้าต้องการควบคุมเอง) ===
@app.after_request
def add_cors_headers(response):
    # หมายเหตุ: Flask-CORS ทำให้อัตโนมัติ แต่ถ้าต้องการทำเอง:
    origin = request.headers.get('Origin', '')
    
    allowed_origins = ['https://app.example.com', 'https://admin.example.com']
    
    if origin in allowed_origins:
        response.headers['Access-Control-Allow-Origin'] = origin
        response.headers['Access-Control-Allow-Credentials'] = 'true'
    
    return response

# Handle preflight requests
@app.route('/api/v1/resource', methods=['OPTIONS'])
def handle_options():
    """Handle CORS preflight request"""
    response = jsonify({'status': 'ok'})
    response.headers['Access-Control-Allow-Origin'] = 'https://app.example.com'
    response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE'
    response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
    response.headers['Access-Control-Max-Age'] = '600'
    return response, 200
```

---

## 15. Testing REST APIs

### Unit Tests ด้วย pytest

```bash
pip install pytest flask-testing
```

```python
# test_api.py
import pytest
import json
from app import create_app  # สมมติว่า app ถูกสร้างใน app.py

@pytest.fixture
def app():
    """Create application instance สำหรับ testing"""
    app = create_app({
        'TESTING': True,
        'DATABASE': ':memory:',
    })
    return app

@pytest.fixture
def client(app):
    """Create test client"""
    return app.test_client()

@pytest.fixture
def runner(app):
    """Create test CLI runner"""
    return app.test_cli_runner()

class TestBookAPI:
    """Tests สำหรับ Book API"""
    
    def test_get_books_returns_200(self, client):
        """GET /books ควร return 200"""
        response = client.get('/api/v1/books')
        assert response.status_code == 200
    
    def test_get_books_returns_list(self, client):
        """GET /books ควร return list"""
        response = client.get('/api/v1/books')
        data = json.loads(response.data)
        assert 'books' in data
        assert isinstance(data['books'], list)
    
    def test_get_nonexistent_book_returns_404(self, client):
        """GET /books/9999 ควร return 404"""
        response = client.get('/api/v1/books/9999')
        assert response.status_code == 404
        
        data = json.loads(response.data)
        assert data['success'] == False
        assert 'error' in data
    
    def test_create_book_success(self, client):
        """POST /books ควร create book และ return 201"""
        payload = {
            'title': 'Test Book',
            'author': 'Test Author',
            'price': 29.99
        }
        response = client.post(
            '/api/v1/books',
            data=json.dumps(payload),
            content_type='application/json'
        )
        assert response.status_code == 201
        
        data = json.loads(response.data)
        assert data['title'] == payload['title']
        assert data['author'] == payload['author']
        assert 'id' in data
        
        # Check Location header
        assert 'Location' in response.headers
    
    def test_create_book_missing_field(self, client):
        """POST /books ขาด required field ควร return 400"""
        payload = {'title': 'Book Without Author'}  # ขาด author และ price
        
        response = client.post(
            '/api/v1/books',
            data=json.dumps(payload),
            content_type='application/json'
        )
        assert response.status_code == 400
        
        data = json.loads(response.data)
        assert data['success'] == False
    
    def test_update_book(self, client):
        """PUT /books/{id} ควร update book"""
        # First create a book
        create_response = client.post(
            '/api/v1/books',
            data=json.dumps({'title': 'Original', 'author': 'Author', 'price': 19.99}),
            content_type='application/json'
        )
        book_id = json.loads(create_response.data)['id']
        
        # Then update it
        update_payload = {
            'title': 'Updated Title',
            'author': 'New Author',
            'price': 25.99
        }
        response = client.put(
            f'/api/v1/books/{book_id}',
            data=json.dumps(update_payload),
            content_type='application/json'
        )
        assert response.status_code == 200
        
        data = json.loads(response.data)
        assert data['title'] == 'Updated Title'
    
    def test_delete_book(self, client):
        """DELETE /books/{id} ควร delete book และ return 204"""
        # Create a book first
        create_response = client.post(
            '/api/v1/books',
            data=json.dumps({'title': 'To Delete', 'author': 'Author', 'price': 9.99}),
            content_type='application/json'
        )
        book_id = json.loads(create_response.data)['id']
        
        # Delete it
        response = client.delete(f'/api/v1/books/{book_id}')
        assert response.status_code == 204
        
        # Verify it's deleted
        get_response = client.get(f'/api/v1/books/{book_id}')
        assert get_response.status_code == 404
    
    def test_pagination(self, client):
        """GET /books?page=1&per_page=2 ควร paginate correctly"""
        response = client.get('/api/v1/books?page=1&per_page=2')
        assert response.status_code == 200
        
        data = json.loads(response.data)
        assert 'pagination' in data or 'meta' in data

    def test_cors_headers(self, client):
        """ตรวจสอบ CORS headers"""
        response = client.get(
            '/api/v1/books',
            headers={'Origin': 'https://app.example.com'}
        )
        # CORS headers ควรมีอยู่
        # assert 'Access-Control-Allow-Origin' in response.headers
```

### Integration Tests ด้วย requests library

```python
# integration_test.py
import requests
import pytest

BASE_URL = 'http://localhost:5000/api/v1'

def test_full_crud_flow():
    """Test complete CRUD flow"""
    
    # 1. Create
    create_response = requests.post(
        f'{BASE_URL}/books',
        json={'title': 'Integration Test Book', 'author': 'Test', 'price': 19.99},
        headers={'Content-Type': 'application/json'}
    )
    assert create_response.status_code == 201
    book = create_response.json()
    book_id = book['id']
    
    # 2. Read
    get_response = requests.get(f'{BASE_URL}/books/{book_id}')
    assert get_response.status_code == 200
    assert get_response.json()['id'] == book_id
    
    # 3. Update
    update_response = requests.patch(
        f'{BASE_URL}/books/{book_id}',
        json={'price': 24.99}
    )
    assert update_response.status_code == 200
    assert update_response.json()['price'] == 24.99
    
    # 4. Delete
    delete_response = requests.delete(f'{BASE_URL}/books/{book_id}')
    assert delete_response.status_code == 204
    
    # 5. Verify deleted
    verify_response = requests.get(f'{BASE_URL}/books/{book_id}')
    assert verify_response.status_code == 404
    
    print("Full CRUD test passed!")
```

---

## 16. ตัวอย่างโปรแกรมจริง: Complete CRUD REST API

โปรแกรมนี้เป็น complete REST API สำหรับระบบ Task Management

```python
# task_api.py - Complete REST API สำหรับ Task Management
from flask import Flask, jsonify, request, make_response
from flask_cors import CORS
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from functools import wraps
from datetime import datetime, timezone
import uuid
import hashlib
import hmac

app = Flask(__name__)
CORS(app, resources={r'/api/*': {'origins': '*'}})

# Rate Limiter
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=['1000 per day', '100 per hour']
)

# ==================== In-Memory Database ====================
tasks_db = {}
users_db = {
    'user_001': {
        'id': 'user_001',
        'name': 'Alice',
        'email': 'alice@example.com',
        'api_key': 'alice-api-key-12345'
    },
    'user_002': {
        'id': 'user_002',
        'name': 'Bob',
        'email': 'bob@example.com',
        'api_key': 'bob-api-key-67890'
    }
}

# ==================== Authentication ====================
def require_api_key(f):
    """Decorator ตรวจสอบ API key"""
    @wraps(f)
    def decorated(*args, **kwargs):
        api_key = request.headers.get('X-API-Key')
        if not api_key:
            return jsonify({
                'success': False,
                'error': {
                    'code': 'MISSING_API_KEY',
                    'message': 'API key is required'
                }
            }), 401
        
        # Find user by API key
        user = next(
            (u for u in users_db.values() if u['api_key'] == api_key),
            None
        )
        
        if not user:
            return jsonify({
                'success': False,
                'error': {
                    'code': 'INVALID_API_KEY',
                    'message': 'Invalid API key'
                }
            }), 401
        
        # Attach user to request context
        request.current_user = user
        return f(*args, **kwargs)
    return decorated

# ==================== Helper Functions ====================
def create_task_dict(data, user_id):
    """สร้าง task dictionary ใหม่"""
    return {
        'id': str(uuid.uuid4()),
        'title': data['title'],
        'description': data.get('description', ''),
        'status': data.get('status', 'todo'),
        'priority': data.get('priority', 'medium'),
        'user_id': user_id,
        'tags': data.get('tags', []),
        'due_date': data.get('due_date'),
        'created_at': datetime.now(timezone.utc).isoformat(),
        'updated_at': datetime.now(timezone.utc).isoformat(),
    }

def validate_task_data(data, partial=False):
    """Validate task data, return errors dict"""
    errors = {}
    
    if not partial or 'title' in data:
        title = data.get('title', '')
        if not title:
            errors['title'] = 'Title is required'
        elif len(title) < 3:
            errors['title'] = 'Title must be at least 3 characters'
        elif len(title) > 200:
            errors['title'] = 'Title must be at most 200 characters'
    
    if 'status' in data:
        valid_statuses = ['todo', 'in_progress', 'review', 'done', 'cancelled']
        if data['status'] not in valid_statuses:
            errors['status'] = f'Status must be one of: {", ".join(valid_statuses)}'
    
    if 'priority' in data:
        valid_priorities = ['low', 'medium', 'high', 'urgent']
        if data['priority'] not in valid_priorities:
            errors['priority'] = f'Priority must be one of: {", ".join(valid_priorities)}'
    
    if 'due_date' in data and data['due_date']:
        try:
            datetime.fromisoformat(data['due_date'])
        except ValueError:
            errors['due_date'] = 'Invalid date format. Use ISO 8601 (YYYY-MM-DD)'
    
    return errors

# ==================== Task Routes ====================

@app.route('/api/v1/tasks', methods=['GET'])
@require_api_key
@limiter.limit('60 per minute')
def list_tasks():
    """
    GET /api/v1/tasks
    
    Query parameters:
    - status: filter by status
    - priority: filter by priority
    - search: search in title and description
    - sort: sort field (title, created_at, priority, due_date)
    - order: asc or desc
    - page: page number
    - per_page: items per page
    - tags: filter by tags (comma-separated)
    """
    user_id = request.current_user['id']
    
    # Get user's tasks
    user_tasks = [t for t in tasks_db.values() if t['user_id'] == user_id]
    
    # === Filtering ===
    status_filter = request.args.get('status')
    if status_filter:
        user_tasks = [t for t in user_tasks if t['status'] == status_filter]
    
    priority_filter = request.args.get('priority')
    if priority_filter:
        user_tasks = [t for t in user_tasks if t['priority'] == priority_filter]
    
    search = request.args.get('search', '').strip()
    if search:
        user_tasks = [
            t for t in user_tasks
            if search.lower() in t['title'].lower() or
               search.lower() in t['description'].lower()
        ]
    
    tags_filter = request.args.get('tags', '')
    if tags_filter:
        filter_tags = [tag.strip() for tag in tags_filter.split(',')]
        user_tasks = [
            t for t in user_tasks
            if any(tag in t['tags'] for tag in filter_tags)
        ]
    
    # === Sorting ===
    sort_field = request.args.get('sort', 'created_at')
    sort_order = request.args.get('order', 'desc')
    
    valid_sort_fields = ['title', 'created_at', 'updated_at', 'priority', 'status']
    if sort_field not in valid_sort_fields:
        sort_field = 'created_at'
    
    priority_order = {'urgent': 0, 'high': 1, 'medium': 2, 'low': 3}
    
    if sort_field == 'priority':
        user_tasks.sort(
            key=lambda t: priority_order.get(t['priority'], 99),
            reverse=(sort_order == 'asc')  # reversed เพราะ lower number = higher priority
        )
    else:
        user_tasks.sort(
            key=lambda t: t.get(sort_field, ''),
            reverse=(sort_order == 'desc')
        )
    
    # === Pagination ===
    page = max(request.args.get('page', 1, type=int), 1)
    per_page = min(max(request.args.get('per_page', 20, type=int), 1), 100)
    
    total = len(user_tasks)
    total_pages = (total + per_page - 1) // per_page if total > 0 else 1
    start = (page - 1) * per_page
    end = start + per_page
    page_tasks = user_tasks[start:end]
    
    return jsonify({
        'success': True,
        'data': {
            'tasks': page_tasks,
            'pagination': {
                'current_page': page,
                'per_page': per_page,
                'total_items': total,
                'total_pages': total_pages,
                'has_next': page < total_pages,
                'has_prev': page > 1
            },
            'filters': {
                'status': status_filter,
                'priority': priority_filter,
                'search': search if search else None,
            }
        }
    })

@app.route('/api/v1/tasks', methods=['POST'])
@require_api_key
@limiter.limit('30 per minute')
def create_task():
    """POST /api/v1/tasks - สร้าง task ใหม่"""
    if not request.is_json:
        return jsonify({
            'success': False,
            'error': {'code': 'INVALID_CONTENT_TYPE', 'message': 'Content-Type must be application/json'}
        }), 415
    
    data = request.get_json()
    
    # Validate
    errors = validate_task_data(data)
    if errors:
        return jsonify({
            'success': False,
            'error': {
                'code': 'VALIDATION_ERROR',
                'message': 'Request validation failed',
                'details': errors
            }
        }), 422
    
    # Create task
    user_id = request.current_user['id']
    task = create_task_dict(data, user_id)
    tasks_db[task['id']] = task
    
    return jsonify({
        'success': True,
        'data': {'task': task}
    }), 201, {'Location': f'/api/v1/tasks/{task["id"]}'}

@app.route('/api/v1/tasks/<task_id>', methods=['GET'])
@require_api_key
def get_task(task_id):
    """GET /api/v1/tasks/{id} - ดึง task เฉพาะ"""
    task = tasks_db.get(task_id)
    
    if not task:
        return jsonify({
            'success': False,
            'error': {'code': 'TASK_NOT_FOUND', 'message': f'Task {task_id} not found'}
        }), 404
    
    # Check ownership
    if task['user_id'] != request.current_user['id']:
        return jsonify({
            'success': False,
            'error': {'code': 'FORBIDDEN', 'message': 'You do not have access to this task'}
        }), 403
    
    return jsonify({
        'success': True,
        'data': {'task': task}
    })

@app.route('/api/v1/tasks/<task_id>', methods=['PUT'])
@require_api_key
def replace_task(task_id):
    """PUT /api/v1/tasks/{id} - Replace task ทั้งหมด"""
    task = tasks_db.get(task_id)
    
    if not task:
        return jsonify({
            'success': False,
            'error': {'code': 'TASK_NOT_FOUND', 'message': f'Task {task_id} not found'}
        }), 404
    
    if task['user_id'] != request.current_user['id']:
        return jsonify({
            'success': False,
            'error': {'code': 'FORBIDDEN', 'message': 'You do not have access to this task'}
        }), 403
    
    data = request.get_json()
    errors = validate_task_data(data)
    if errors:
        return jsonify({
            'success': False,
            'error': {'code': 'VALIDATION_ERROR', 'message': 'Validation failed', 'details': errors}
        }), 422
    
    # Replace (keep id, user_id, created_at)
    task.update({
        'title': data['title'],
        'description': data.get('description', ''),
        'status': data.get('status', 'todo'),
        'priority': data.get('priority', 'medium'),
        'tags': data.get('tags', []),
        'due_date': data.get('due_date'),
        'updated_at': datetime.now(timezone.utc).isoformat(),
    })
    
    return jsonify({'success': True, 'data': {'task': task}})

@app.route('/api/v1/tasks/<task_id>', methods=['PATCH'])
@require_api_key
def update_task(task_id):
    """PATCH /api/v1/tasks/{id} - อัพเดทบางส่วนของ task"""
    task = tasks_db.get(task_id)
    
    if not task:
        return jsonify({
            'success': False,
            'error': {'code': 'TASK_NOT_FOUND', 'message': f'Task {task_id} not found'}
        }), 404
    
    if task['user_id'] != request.current_user['id']:
        return jsonify({
            'success': False,
            'error': {'code': 'FORBIDDEN', 'message': 'Access denied'}
        }), 403
    
    data = request.get_json()
    errors = validate_task_data(data, partial=True)
    if errors:
        return jsonify({
            'success': False,
            'error': {'code': 'VALIDATION_ERROR', 'message': 'Validation failed', 'details': errors}
        }), 422
    
    # Update only provided fields
    updatable_fields = ['title', 'description', 'status', 'priority', 'tags', 'due_date']
    for field in updatable_fields:
        if field in data:
            task[field] = data[field]
    
    task['updated_at'] = datetime.now(timezone.utc).isoformat()
    
    return jsonify({'success': True, 'data': {'task': task}})

@app.route('/api/v1/tasks/<task_id>', methods=['DELETE'])
@require_api_key
def delete_task(task_id):
    """DELETE /api/v1/tasks/{id} - ลบ task"""
    task = tasks_db.get(task_id)
    
    if not task:
        return jsonify({
            'success': False,
            'error': {'code': 'TASK_NOT_FOUND', 'message': f'Task {task_id} not found'}
        }), 404
    
    if task['user_id'] != request.current_user['id']:
        return jsonify({
            'success': False,
            'error': {'code': 'FORBIDDEN', 'message': 'Access denied'}
        }), 403
    
    del tasks_db[task_id]
    return '', 204

# ==================== Statistics Route ====================
@app.route('/api/v1/tasks/stats', methods=['GET'])
@require_api_key
def task_stats():
    """GET /api/v1/tasks/stats - สถิติ tasks ของ user"""
    user_id = request.current_user['id']
    user_tasks = [t for t in tasks_db.values() if t['user_id'] == user_id]
    
    stats = {
        'total': len(user_tasks),
        'by_status': {},
        'by_priority': {},
        'completion_rate': 0
    }
    
    for task in user_tasks:
        status = task['status']
        priority = task['priority']
        stats['by_status'][status] = stats['by_status'].get(status, 0) + 1
        stats['by_priority'][priority] = stats['by_priority'].get(priority, 0) + 1
    
    if stats['total'] > 0:
        done_count = stats['by_status'].get('done', 0)
        stats['completion_rate'] = round(done_count / stats['total'] * 100, 2)
    
    return jsonify({'success': True, 'data': {'stats': stats}})

# ==================== Error Handlers ====================
@app.errorhandler(404)
def not_found(e):
    return jsonify({'success': False, 'error': {'code': 'NOT_FOUND', 'message': 'Route not found'}}), 404

@app.errorhandler(405)
def method_not_allowed(e):
    return jsonify({'success': False, 'error': {'code': 'METHOD_NOT_ALLOWED', 'message': 'Method not allowed'}}), 405

@app.errorhandler(500)
def internal_error(e):
    return jsonify({'success': False, 'error': {'code': 'INTERNAL_ERROR', 'message': 'Internal server error'}}), 500

# ==================== Main ====================
if __name__ == '__main__':
    # เพิ่ม sample tasks
    sample_user_id = 'user_001'
    for i in range(5):
        task = create_task_dict({
            'title': f'Sample Task {i+1}',
            'description': f'Description for task {i+1}',
            'status': ['todo', 'in_progress', 'done'][i % 3],
            'priority': ['low', 'medium', 'high'][i % 3],
            'tags': ['work', 'personal'][i % 2:i % 2 + 1]
        }, sample_user_id)
        tasks_db[task['id']] = task
    
    print("Task Management API started!")
    print("API Key for Alice: alice-api-key-12345")
    print("API Key for Bob: bob-api-key-67890")
    app.run(debug=True, port=5000)
```

---

## 17. แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Resource URL Structure

**โจทย์**: ออกแบบ URL structure สำหรับ Library Management System ที่มี:
- Books (หนังสือ)
- Authors (ผู้แต่ง)
- Members (สมาชิก)
- Borrowings (การยืม)

**เฉลย**:

```
# Books
GET    /api/v1/books                -> รายการหนังสือทั้งหมด
POST   /api/v1/books                -> เพิ่มหนังสือใหม่
GET    /api/v1/books/{id}           -> ดูหนังสือเฉพาะเล่ม
PUT    /api/v1/books/{id}           -> อัพเดทหนังสือทั้งหมด
PATCH  /api/v1/books/{id}           -> อัพเดทบางส่วน
DELETE /api/v1/books/{id}           -> ลบหนังสือ

# Authors
GET    /api/v1/authors              -> รายการผู้แต่งทั้งหมด
POST   /api/v1/authors              -> เพิ่มผู้แต่งใหม่
GET    /api/v1/authors/{id}         -> ดูผู้แต่งเฉพาะคน
GET    /api/v1/authors/{id}/books   -> หนังสือของผู้แต่งคนนั้น

# Members
GET    /api/v1/members              -> รายการสมาชิก
POST   /api/v1/members              -> สมัครสมาชิก
GET    /api/v1/members/{id}         -> ดูข้อมูลสมาชิก
GET    /api/v1/members/{id}/borrowings -> ประวัติการยืมของสมาชิก

# Borrowings
GET    /api/v1/borrowings           -> รายการการยืมทั้งหมด
POST   /api/v1/borrowings           -> ยืมหนังสือ
GET    /api/v1/borrowings/{id}      -> ดูการยืมเฉพาะ
PATCH  /api/v1/borrowings/{id}      -> อัพเดทสถานะ (return, extend)
```

### แบบฝึกหัดที่ 2: Implement CRUD API สำหรับ Todo List

**โจทย์**: สร้าง Flask API สำหรับ Todo List พร้อม:
- CRUD operations
- Validation
- Proper status codes
- Error handling

**เฉลย**:

```python
from flask import Flask, jsonify, request
from datetime import datetime, timezone
import uuid

app = Flask(__name__)

# In-memory store
todos = {}

def validate_todo(data, partial=False):
    errors = {}
    
    if not partial or 'title' in data:
        if not data.get('title'):
            errors['title'] = 'Title is required'
        elif len(data['title']) > 200:
            errors['title'] = 'Title too long (max 200 chars)'
    
    if 'completed' in data:
        if not isinstance(data['completed'], bool):
            errors['completed'] = 'Must be boolean'
    
    return errors

@app.route('/todos', methods=['GET'])
def list_todos():
    filter_completed = request.args.get('completed')
    result = list(todos.values())
    
    if filter_completed == 'true':
        result = [t for t in result if t['completed']]
    elif filter_completed == 'false':
        result = [t for t in result if not t['completed']]
    
    return jsonify({'todos': result, 'total': len(result)})

@app.route('/todos', methods=['POST'])
def create_todo():
    data = request.get_json() or {}
    errors = validate_todo(data)
    if errors:
        return jsonify({'error': 'Validation failed', 'details': errors}), 422
    
    todo = {
        'id': str(uuid.uuid4()),
        'title': data['title'],
        'description': data.get('description', ''),
        'completed': data.get('completed', False),
        'created_at': datetime.now(timezone.utc).isoformat(),
        'updated_at': datetime.now(timezone.utc).isoformat(),
    }
    todos[todo['id']] = todo
    return jsonify(todo), 201, {'Location': f'/todos/{todo["id"]}'}

@app.route('/todos/<todo_id>', methods=['GET'])
def get_todo(todo_id):
    todo = todos.get(todo_id)
    if not todo:
        return jsonify({'error': 'Todo not found'}), 404
    return jsonify(todo)

@app.route('/todos/<todo_id>', methods=['PATCH'])
def update_todo(todo_id):
    todo = todos.get(todo_id)
    if not todo:
        return jsonify({'error': 'Todo not found'}), 404
    
    data = request.get_json() or {}
    errors = validate_todo(data, partial=True)
    if errors:
        return jsonify({'error': 'Validation failed', 'details': errors}), 422
    
    for field in ['title', 'description', 'completed']:
        if field in data:
            todo[field] = data[field]
    todo['updated_at'] = datetime.now(timezone.utc).isoformat()
    
    return jsonify(todo)

@app.route('/todos/<todo_id>', methods=['DELETE'])
def delete_todo(todo_id):
    if todo_id not in todos:
        return jsonify({'error': 'Todo not found'}), 404
    del todos[todo_id]
    return '', 204
```

### แบบฝึกหัดที่ 3: Implement Pagination

**โจทย์**: เพิ่ม pagination ให้กับ API ที่มีอยู่พร้อม navigation links

**เฉลย**:

```python
def paginate(items, page, per_page, base_url):
    """Helper function สำหรับ pagination"""
    total = len(items)
    total_pages = max((total + per_page - 1) // per_page, 1)
    
    # Clamp page
    page = max(1, min(page, total_pages))
    
    start = (page - 1) * per_page
    end = start + per_page
    page_items = items[start:end]
    
    # Build links
    links = {
        'self': f'{base_url}?page={page}&per_page={per_page}',
        'first': f'{base_url}?page=1&per_page={per_page}',
        'last': f'{base_url}?page={total_pages}&per_page={per_page}',
    }
    if page > 1:
        links['prev'] = f'{base_url}?page={page-1}&per_page={per_page}'
    if page < total_pages:
        links['next'] = f'{base_url}?page={page+1}&per_page={per_page}'
    
    return {
        'items': page_items,
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total': total,
            'total_pages': total_pages,
            'has_next': page < total_pages,
            'has_prev': page > 1,
        },
        '_links': links
    }

@app.route('/api/v1/items')
def paginated_items():
    items = [{'id': i, 'name': f'Item {i}'} for i in range(1, 101)]
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 10, type=int), 50)
    
    result = paginate(items, page, per_page, '/api/v1/items')
    return jsonify(result)
```

### แบบฝึกหัดที่ 4: Error Handling Middleware

**โจทย์**: สร้าง comprehensive error handling middleware สำหรับ Flask API

**เฉลย**:

```python
from flask import Flask, jsonify, request, g
import traceback
import logging
import time

app = Flask(__name__)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.before_request
def before_request():
    """Log incoming request และบันทึก start time"""
    g.start_time = time.time()
    logger.info(f"REQUEST: {request.method} {request.path} from {request.remote_addr}")

@app.after_request
def after_request(response):
    """Log response และเพิ่ม headers"""
    elapsed = time.time() - g.start_time
    logger.info(f"RESPONSE: {response.status_code} ({elapsed:.3f}s)")
    
    # เพิ่ม timing header
    response.headers['X-Response-Time'] = f'{elapsed:.3f}s'
    response.headers['X-Request-ID'] = request.headers.get('X-Request-ID', 'unknown')
    return response

@app.errorhandler(Exception)
def handle_unexpected_error(error):
    """Handle unexpected errors"""
    logger.error(f"Unexpected error: {str(error)}\n{traceback.format_exc()}")
    
    return jsonify({
        'success': False,
        'error': {
            'code': 'INTERNAL_ERROR',
            'message': 'An unexpected error occurred',
            # ไม่แสดง stack trace ใน production
            'debug': str(error) if app.debug else None
        }
    }), 500
```

### แบบฝึกหัดที่ 5: API Versioning

**โจทย์**: สร้าง Flask API ที่รองรับ 2 versions ด้วย Blueprint

**เฉลย**:

```python
from flask import Flask, Blueprint, jsonify

def create_app():
    app = Flask(__name__)
    
    # V1 Blueprint
    v1 = Blueprint('v1', __name__, url_prefix='/api/v1')
    
    @v1.route('/users')
    def v1_users():
        return jsonify({
            'users': [{'id': 1, 'name': 'Alice'}],
            'version': '1.0'
        })
    
    @v1.route('/users/<int:user_id>')
    def v1_user(user_id):
        return jsonify({'id': user_id, 'name': 'Alice', 'version': '1.0'})
    
    # V2 Blueprint (เพิ่ม pagination และ detailed info)
    v2 = Blueprint('v2', __name__, url_prefix='/api/v2')
    
    @v2.route('/users')
    def v2_users():
        from flask import request
        page = request.args.get('page', 1, type=int)
        per_page = request.args.get('per_page', 10, type=int)
        
        return jsonify({
            'users': [
                {'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'role': 'admin'},
            ],
            'pagination': {'page': page, 'per_page': per_page, 'total': 1},
            'version': '2.0'
        })
    
    app.register_blueprint(v1)
    app.register_blueprint(v2)
    
    return app

app = create_app()
```

### แบบฝึกหัดที่ 6: Filtering และ Sorting

**โจทย์**: Implement advanced filtering และ sorting สำหรับ Product API

**เฉลย**:

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

PRODUCTS = [
    {'id': 1, 'name': 'Laptop', 'category': 'electronics', 'price': 999, 'rating': 4.5},
    {'id': 2, 'name': 'Phone', 'category': 'electronics', 'price': 599, 'rating': 4.2},
    {'id': 3, 'name': 'Desk', 'category': 'furniture', 'price': 299, 'rating': 4.0},
    {'id': 4, 'name': 'Chair', 'category': 'furniture', 'price': 199, 'rating': 3.8},
    {'id': 5, 'name': 'Book', 'category': 'education', 'price': 39, 'rating': 4.8},
]

@app.route('/api/v1/products')
def products():
    items = PRODUCTS.copy()
    
    # Filtering
    category = request.args.get('category')
    if category:
        items = [p for p in items if p['category'] == category]
    
    min_price = request.args.get('min_price', type=float)
    max_price = request.args.get('max_price', type=float)
    if min_price: items = [p for p in items if p['price'] >= min_price]
    if max_price: items = [p for p in items if p['price'] <= max_price]
    
    min_rating = request.args.get('min_rating', type=float)
    if min_rating: items = [p for p in items if p['rating'] >= min_rating]
    
    search = request.args.get('q', '').lower()
    if search: items = [p for p in items if search in p['name'].lower()]
    
    # Sorting: ?sort=-price,name (- = descending)
    sort = request.args.get('sort', 'id')
    for field in reversed(sort.split(',')):
        desc = field.startswith('-')
        fname = field.lstrip('-')
        if fname in ['id', 'name', 'price', 'rating']:
            items.sort(key=lambda x: x[fname], reverse=desc)
    
    return jsonify({'products': items, 'count': len(items)})
```

### แบบฝึกหัดที่ 7: Rate Limiting

**โจทย์**: เพิ่ม rate limiting ให้ API โดยแยก limit สำหรับ public และ authenticated users

**เฉลย**:

```python
from flask import Flask, jsonify, request, g
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

def get_limit_key():
    """Key function ที่ใช้ API key หากมี หรือ IP address"""
    api_key = request.headers.get('X-API-Key')
    if api_key:
        return f'api_key:{api_key}'
    return f'ip:{get_remote_address()}'

limiter = Limiter(
    app=app,
    key_func=get_limit_key,
    default_limits=['100 per hour']
)

@app.route('/api/public/search')
@limiter.limit('10 per minute')  # Strict สำหรับ public
def public_search():
    return jsonify({'results': []})

@app.route('/api/auth/search')
@limiter.limit('60 per minute')  # ผ่อนปรนสำหรับ authenticated
def auth_search():
    api_key = request.headers.get('X-API-Key')
    if not api_key:
        return jsonify({'error': 'API key required'}), 401
    return jsonify({'results': [], 'authenticated': True})

@app.errorhandler(429)
def rate_limit_error(e):
    return jsonify({
        'error': 'RATE_LIMIT_EXCEEDED',
        'message': 'Too many requests',
        'retry_after': '60 seconds'
    }), 429
```

### แบบฝึกหัดที่ 8: Complete API Testing

**โจทย์**: เขียน comprehensive test suite สำหรับ User API

**เฉลย**:

```python
import pytest
import json

# app.py
from flask import Flask, jsonify, request

def create_test_app():
    app = Flask(__name__)
    app.config['TESTING'] = True
    
    users = [
        {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
        {'id': 2, 'name': 'Bob', 'email': 'bob@example.com'},
    ]
    
    @app.route('/users', methods=['GET'])
    def get_users():
        return jsonify({'users': users})
    
    @app.route('/users/<int:user_id>', methods=['GET'])
    def get_user(user_id):
        user = next((u for u in users if u['id'] == user_id), None)
        if not user:
            return jsonify({'error': 'Not found'}), 404
        return jsonify(user)
    
    @app.route('/users', methods=['POST'])
    def create_user():
        data = request.get_json()
        if not data or not data.get('name') or not data.get('email'):
            return jsonify({'error': 'name and email required'}), 400
        new_user = {'id': len(users) + 1, **data}
        users.append(new_user)
        return jsonify(new_user), 201
    
    return app

# test_users.py
@pytest.fixture
def client():
    app = create_test_app()
    with app.test_client() as client:
        yield client

class TestUserAPI:
    def test_list_users(self, client):
        r = client.get('/users')
        assert r.status_code == 200
        data = r.get_json()
        assert 'users' in data
        assert len(data['users']) == 2
    
    def test_get_user(self, client):
        r = client.get('/users/1')
        assert r.status_code == 200
        assert r.get_json()['name'] == 'Alice'
    
    def test_get_nonexistent_user(self, client):
        r = client.get('/users/999')
        assert r.status_code == 404
    
    def test_create_user(self, client):
        r = client.post('/users', json={'name': 'Charlie', 'email': 'charlie@example.com'})
        assert r.status_code == 201
        data = r.get_json()
        assert data['name'] == 'Charlie'
        assert 'id' in data
    
    def test_create_user_missing_fields(self, client):
        r = client.post('/users', json={'name': 'NoEmail'})
        assert r.status_code == 400
    
    def test_create_user_no_body(self, client):
        r = client.post('/users', content_type='application/json', data='')
        assert r.status_code in [400, 422]
    
    def test_user_count_increases_after_create(self, client):
        initial = len(client.get('/users').get_json()['users'])
        client.post('/users', json={'name': 'New', 'email': 'new@example.com'})
        final = len(client.get('/users').get_json()['users'])
        assert final == initial + 1
    
    def test_response_content_type(self, client):
        r = client.get('/users')
        assert 'application/json' in r.content_type

if __name__ == '__main__':
    pytest.main([__file__, '-v'])
```

---

## สรุป

ใน Part 56 เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียนรู้ |
|--------|-------------------|
| REST Principles | 6 หลักการของ REST และ Richardson Maturity Model |
| URL Design | Resource-based URLs, naming conventions |
| HTTP Methods | GET, POST, PUT, PATCH, DELETE semantics |
| Status Codes | การใช้ 2xx, 3xx, 4xx, 5xx อย่างถูกต้อง |
| JSON Format | Standard request/response format |
| Flask-RESTful | Class-based views, marshalling, parsing |
| Versioning | Blueprint-based URL versioning |
| Pagination | Offset-based และ cursor-based pagination |
| Filtering/Sorting | Query parameters สำหรับ filtering และ sorting |
| Rate Limiting | Flask-Limiter สำหรับ protect API |
| Documentation | Swagger/OpenAPI documentation |
| Error Handling | Standard error format และ custom exceptions |
| CORS | Flask-CORS สำหรับ cross-origin requests |
| Testing | Unit tests และ integration tests |

### เครื่องมือที่ใช้

```bash
# ติดตั้ง packages ที่จำเป็น
pip install flask flask-restful flask-cors flask-limiter flask-openapi3 flasgger pytest
```

### ขั้นตอนต่อไป

- **Part 57**: FastAPI - Getting Started (เร็วกว่า Flask, automatic documentation)
- **Part 58**: FastAPI - Routing, Validation & Pydantic
- **Part 59**: FastAPI - Database & Async ORM
- **Part 60**: FastAPI - Authentication, JWT & OAuth2
