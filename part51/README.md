# Part 51: Flask - Getting Started

## สารบัญ
1. [แนวคิด Web Development](#1-แนวคิด-web-development)
2. [HTTP Protocol](#2-http-protocol)
3. [REST Architecture](#3-rest-architecture)
4. [MVC Pattern](#4-mvc-pattern)
5. [Flask Framework คืออะไร](#5-flask-framework-คืออะไร)
6. [การติดตั้งและ Setup](#6-การติดตั้งและ-setup)
7. [โครงสร้างของ Flask Application](#7-โครงสร้างของ-flask-application)
8. [Flask App Factory Pattern](#8-flask-app-factory-pattern)
9. [Routes และ URL Rules](#9-routes-และ-url-rules)
10. [HTTP Methods](#10-http-methods)
11. [Request และ Response Objects](#11-request-และ-response-objects)
12. [Development Server](#12-development-server)
13. [Debug Mode](#13-debug-mode)
14. [Flask Configuration](#14-flask-configuration)
15. [Application Context](#15-application-context)
16. [Blueprint เบื้องต้น](#16-blueprint-เบื้องต้น)
17. [ตัวอย่างโปรแกรมจริง](#17-ตัวอย่างโปรแกรมจริง)
18. [แบบฝึกหัด](#18-แบบฝึกหัด)

---

## 1. แนวคิด Web Development

### Web Development คืออะไร

Web Development คือกระบวนการสร้างและพัฒนาเว็บไซต์หรือ web application ที่ทำงานบน internet หรือ intranet โดยแบ่งออกเป็น 2 ส่วนหลัก:

1. **Frontend (Client-side)**: ส่วนที่ผู้ใช้เห็นและโต้ตอบได้ - HTML, CSS, JavaScript
2. **Backend (Server-side)**: ส่วนที่ทำงานบน server - Python, Node.js, Ruby, PHP

```
Client (Browser)                    Server
     |                                  |
     |  --- HTTP Request ---------->    |
     |                                  | (Process request)
     |  <-- HTTP Response -----------   |
     |                                  |
     | (Render HTML/CSS/JS)             |
```

### Client-Server Architecture

```
[Client Browser]  <--HTTP-->  [Web Server]  <--SQL-->  [Database]
     |                             |
  User Interface            Business Logic
  (HTML/CSS/JS)            (Python/Flask)
```

สถาปัตยกรรมนี้ทำงานดังนี้:
- **Client**: ส่ง request ไปยัง server
- **Web Server**: รับ request, ประมวลผล, ดึงข้อมูลจาก database
- **Database**: เก็บข้อมูล
- **Server**: ส่ง response กลับมาเป็น HTML, JSON, หรือ data อื่นๆ

### Static vs Dynamic Websites

**Static Website**: เนื้อหาไม่เปลี่ยนแปลง
```
Browser --> Server --> ส่ง HTML file ที่เก็บอยู่ใน disk กลับไปตรงๆ
```

**Dynamic Website**: เนื้อหาสร้างขึ้นมาแบบ real-time
```
Browser --> Server --> ประมวลผล Python code --> ดึงข้อมูลจาก DB --> สร้าง HTML --> ส่งกลับ
```

---

## 2. HTTP Protocol

### HTTP คืออะไร

HTTP (HyperText Transfer Protocol) คือโปรโตคอลที่ใช้สื่อสารระหว่าง client และ server บน web

### HTTP Request Structure

```
GET /api/users HTTP/1.1
Host: example.com
Authorization: Bearer abc123
Content-Type: application/json
Accept: application/json

{request body - ถ้ามี}
```

ส่วนประกอบของ HTTP Request:
1. **Method**: GET, POST, PUT, DELETE, PATCH, etc.
2. **URL/Path**: เส้นทางที่ต้องการเข้าถึง
3. **HTTP Version**: HTTP/1.1 หรือ HTTP/2
4. **Headers**: ข้อมูลเพิ่มเติม เช่น authentication, content type
5. **Body**: ข้อมูลที่ส่งไป (เฉพาะ POST, PUT, PATCH)

### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 156
Date: Mon, 01 Jan 2024 12:00:00 GMT

{"users": [...]}
```

ส่วนประกอบของ HTTP Response:
1. **Status Code**: รหัสสถานะ
2. **Status Text**: คำอธิบายสถานะ
3. **Headers**: ข้อมูลเพิ่มเติม
4. **Body**: ข้อมูลที่ส่งกลับ

### HTTP Status Codes ที่สำคัญ

| Code | ความหมาย | ใช้เมื่อ |
|------|-----------|----------|
| 200 | OK | Request สำเร็จ |
| 201 | Created | สร้างข้อมูลใหม่สำเร็จ |
| 204 | No Content | สำเร็จแต่ไม่มีข้อมูลส่งกลับ |
| 301 | Moved Permanently | Redirect ถาวร |
| 302 | Found | Redirect ชั่วคราว |
| 400 | Bad Request | Request ไม่ถูกต้อง |
| 401 | Unauthorized | ยังไม่ได้ login |
| 403 | Forbidden | ไม่มีสิทธิ์ |
| 404 | Not Found | ไม่พบข้อมูล |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Server ไม่พร้อม |

### HTTP Methods

```python
# ตัวอย่างการใช้ HTTP methods ต่างๆ
# GET - ดึงข้อมูล
GET /api/users          # ดึงรายการ users ทั้งหมด
GET /api/users/1        # ดึง user ที่มี id = 1

# POST - สร้างข้อมูลใหม่
POST /api/users         # สร้าง user ใหม่

# PUT - อัพเดทข้อมูลทั้งหมด
PUT /api/users/1        # อัพเดท user ที่มี id = 1 (ทั้งหมด)

# PATCH - อัพเดทข้อมูลบางส่วน
PATCH /api/users/1      # อัพเดทบางฟิลด์ของ user ที่มี id = 1

# DELETE - ลบข้อมูล
DELETE /api/users/1     # ลบ user ที่มี id = 1
```

---

## 3. REST Architecture

### REST คืออะไร

REST (Representational State Transfer) คือรูปแบบสถาปัตยกรรมสำหรับการออกแบบ web services โดยมีหลักการสำคัญ:

1. **Stateless**: Server ไม่เก็บ state ของ client ระหว่าง requests
2. **Resource-based**: ทุกอย่างเป็น resource ที่มี URL ของตัวเอง
3. **Uniform Interface**: ใช้ HTTP methods มาตรฐาน
4. **Client-Server**: แยก client กับ server ออกจากกัน
5. **Cacheable**: Response สามารถ cache ได้

### RESTful API Design

```
Resource: Users
-----------------
GET    /users       -> ดึงรายการ users ทั้งหมด
POST   /users       -> สร้าง user ใหม่
GET    /users/{id}  -> ดึง user ตาม id
PUT    /users/{id}  -> อัพเดท user ทั้งหมด
PATCH  /users/{id}  -> อัพเดท user บางส่วน
DELETE /users/{id}  -> ลบ user

Resource: Posts (nested under users)
--------------------------------------
GET    /users/{id}/posts     -> ดึง posts ของ user
POST   /users/{id}/posts     -> สร้าง post ใหม่ของ user
```

### JSON Response Format

```json
{
    "status": "success",
    "data": {
        "id": 1,
        "name": "สมชาย ใจดี",
        "email": "somchai@example.com"
    },
    "message": "User found successfully"
}
```

---

## 4. MVC Pattern

### MVC คืออะไร

MVC (Model-View-Controller) คือ design pattern ที่แบ่งแอปพลิเคชันออกเป็น 3 ส่วน:

```
         User
          |
          v
     [Controller]  <-- รับ request, ประมวลผล logic
      /        \
[Model]      [View]
 |              |
 Database    Template/HTML
```

1. **Model**: จัดการข้อมูลและ business logic (เชื่อมกับ database)
2. **View**: แสดงผลข้อมูล (HTML templates)
3. **Controller**: รับ request, เรียกใช้ model, ส่งข้อมูลให้ view

### MVC ใน Flask

```python
# Model (models.py)
class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email
    
    @classmethod
    def get_by_id(cls, user_id):
        # ดึงข้อมูลจาก database
        pass

# Controller (routes.py / views.py)
@app.route('/users/<int:id>')
def get_user(id):
    user = User.get_by_id(id)  # เรียกใช้ Model
    return render_template('user.html', user=user)  # ส่งให้ View

# View (templates/user.html)
# <h1>{{ user.name }}</h1>
# <p>{{ user.email }}</p>
```

---

## 5. Flask Framework คืออะไร

### ประวัติและที่มา

Flask เป็น **micro web framework** สำหรับ Python สร้างโดย Armin Ronacher ในปี 2010 โดยเริ่มจากการเป็นแค่ joke สำหรับวัน April Fools แต่กลายมาเป็น framework ที่ได้รับความนิยมสูงมาก

Flask เรียกตัวเองว่า **"micro" framework** เพราะ:
- Core มีขนาดเล็ก
- ไม่บังคับให้ใช้ database หรือ form validation library ใดๆ
- ขยายได้ด้วย extensions

### Flask vs Django

| Feature | Flask | Django |
|---------|-------|--------|
| ขนาด | Micro framework | Full-stack framework |
| Database ORM | ไม่มี (ใช้ extension) | มี built-in |
| Admin Interface | ไม่มี (ใช้ extension) | มี built-in |
| Form validation | ไม่มี (ใช้ extension) | มี built-in |
| Flexibility | สูงมาก | ปานกลาง |
| Learning curve | ต่ำ | ปานกลาง |
| Project size | เล็ก-กลาง | กลาง-ใหญ่ |

### ข้อดีของ Flask

1. **เรียนรู้ง่าย**: เริ่มต้นได้ด้วยโค้ดไม่กี่บรรทัด
2. **Flexible**: เลือกใช้ library ที่ต้องการได้
3. **Extensible**: มี extensions มากมาย
4. **Pythonic**: ใช้ Python ได้อย่างเต็มที่
5. **ดีสำหรับ API**: เหมาะกับการสร้าง RESTful API

### Flask Ecosystem

```
Flask (Core)
├── Flask-SQLAlchemy  (Database ORM)
├── Flask-WTF         (Form validation)
├── Flask-Login       (Authentication)
├── Flask-Mail        (Email)
├── Flask-Migrate     (Database migrations)
├── Flask-JWT-Extended (JWT tokens)
├── Flask-CORS        (Cross-Origin Resource Sharing)
└── Flask-RESTful     (REST API)
```

---

## 6. การติดตั้งและ Setup

### ความต้องการของระบบ

```bash
# Python version ที่รองรับ
Python 3.8+

# ตรวจสอบ Python version
python --version
python3 --version
```

### การสร้าง Virtual Environment

```bash
# สร้าง virtual environment (แนะนำมาก)
python -m venv venv

# Activate virtual environment
# สำหรับ Linux/macOS:
source venv/bin/activate

# สำหรับ Windows:
venv\Scripts\activate

# ตรวจสอบว่า activate แล้ว
which python  # Linux/macOS
where python  # Windows
```

### การติดตั้ง Flask

```bash
# ติดตั้ง Flask
pip install flask

# ติดตั้ง Flask พร้อม dependencies ทั้งหมด
pip install flask[async]

# ตรวจสอบ version
python -c "import flask; print(flask.__version__)"
flask --version
```

### การสร้าง requirements.txt

```bash
# สร้าง requirements.txt
pip freeze > requirements.txt

# ติดตั้งจาก requirements.txt
pip install -r requirements.txt
```

ตัวอย่าง requirements.txt:
```
Flask==3.0.0
Werkzeug==3.0.0
Jinja2==3.1.2
click==8.1.7
itsdangerous==2.1.2
MarkupSafe==2.1.3
```

### โครงสร้าง Project เบื้องต้น

```bash
# สร้าง project structure
mkdir my_flask_app
cd my_flask_app
python -m venv venv
source venv/bin/activate
pip install flask
```

---

## 7. โครงสร้างของ Flask Application

### Simple Application (Single File)

```python
# app.py - Flask application อย่างง่าย
from flask import Flask

# สร้าง Flask instance
app = Flask(__name__)

# กำหนด route
@app.route('/')
def index():
    return 'Hello, World!'

# Run application
if __name__ == '__main__':
    app.run(debug=True)
```

```bash
# วิธีรัน
python app.py
# หรือ
flask run
```

### Project Structure แบบ Simple

```
my_flask_app/
├── app.py              # Main application file
├── requirements.txt    # Dependencies
└── venv/               # Virtual environment
```

### Project Structure แบบ Medium

```
my_flask_app/
├── app/
│   ├── __init__.py     # Application factory
│   ├── models.py       # Database models
│   ├── routes.py       # Routes/Controllers
│   ├── forms.py        # Form definitions
│   ├── templates/      # HTML templates
│   │   ├── base.html
│   │   ├── index.html
│   │   └── user/
│   │       ├── profile.html
│   │       └── settings.html
│   └── static/         # Static files
│       ├── css/
│       ├── js/
│       └── images/
├── config.py           # Configuration
├── run.py              # Entry point
└── requirements.txt
```

### Project Structure แบบ Large (Blueprint-based)

```
my_flask_app/
├── app/
│   ├── __init__.py
│   ├── extensions.py       # Flask extensions
│   ├── auth/               # Auth blueprint
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── blog/               # Blog blueprint
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── models.py
│   │   └── templates/
│   ├── api/                # API blueprint
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── templates/          # Shared templates
│   │   └── base.html
│   └── static/             # Static files
├── migrations/             # Database migrations
├── tests/                  # Tests
│   ├── __init__.py
│   ├── test_auth.py
│   └── test_blog.py
├── config.py
├── run.py
└── requirements.txt
```

---

## 8. Flask App Factory Pattern

### ทำไมต้องใช้ App Factory

App Factory Pattern คือการสร้าง Flask app ใน function แทนที่จะสร้างเป็น global variable ซึ่งมีข้อดีคือ:

1. **Testing**: สามารถสร้าง app instances หลายตัวพร้อมกัน configuration ต่างๆ ได้
2. **Multiple environments**: ง่ายต่อการ config สำหรับ development/testing/production
3. **Circular imports**: หลีกเลี่ยง circular import problems

### การสร้าง App Factory

```python
# app/__init__.py - App Factory Pattern

from flask import Flask
from config import config

def create_app(config_name='default'):
    """Application factory function"""
    app = Flask(__name__)
    
    # โหลด configuration
    app.config.from_object(config[config_name])
    
    # Initialize extensions (จะอธิบายใน Part 54-55)
    # db.init_app(app)
    # login_manager.init_app(app)
    
    # Register blueprints
    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)
    
    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint, url_prefix='/auth')
    
    return app
```

```python
# config.py - Configuration Classes

import os

class Config:
    """Base configuration"""
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-secret-key')
    DEBUG = False
    TESTING = False

class DevelopmentConfig(Config):
    """Development configuration"""
    DEBUG = True
    DATABASE_URI = 'sqlite:///dev.db'

class TestingConfig(Config):
    """Testing configuration"""
    TESTING = True
    DATABASE_URI = 'sqlite:///test.db'

class ProductionConfig(Config):
    """Production configuration"""
    DATABASE_URI = os.environ.get('DATABASE_URL', 'postgresql://...')

# Dictionary ของ configurations
config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

```python
# run.py - Entry point

import os
from app import create_app

# ดึง environment จาก environment variable
config_name = os.environ.get('FLASK_CONFIG', 'development')

# สร้าง app instance
app = create_app(config_name)

if __name__ == '__main__':
    app.run()
```

### ตัวอย่าง App Factory แบบเต็ม

```python
# app/__init__.py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager
from config import config

# สร้าง extension instances (ยังไม่ผูกกับ app)
db = SQLAlchemy()
login_manager = LoginManager()

def create_app(config_name='default'):
    app = Flask(__name__)
    
    # โหลด config
    app.config.from_object(config[config_name])
    
    # Initialize extensions กับ app
    db.init_app(app)
    login_manager.init_app(app)
    
    # Configure login manager
    login_manager.login_view = 'auth.login'
    login_manager.login_message = 'กรุณาเข้าสู่ระบบก่อน'
    
    # Register blueprints
    from .main import main as main_bp
    app.register_blueprint(main_bp)
    
    from .auth import auth as auth_bp
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    from .api import api as api_bp
    app.register_blueprint(api_bp, url_prefix='/api/v1')
    
    # Shell context สำหรับ flask shell
    @app.shell_context_processor
    def make_shell_context():
        return dict(db=db, app=app)
    
    return app
```

---

## 9. Routes และ URL Rules

### Route Basics

Route คือการ map URL path ไปยัง Python function

```python
from flask import Flask

app = Flask(__name__)

# Route เบื้องต้น
@app.route('/')
def index():
    return 'หน้าแรก'

@app.route('/about')
def about():
    return 'เกี่ยวกับเรา'

@app.route('/contact')
def contact():
    return 'ติดต่อเรา'
```

### URL Variables (Dynamic Routes)

```python
# ตัวแปรใน URL
@app.route('/user/<username>')
def user_profile(username):
    return f'โปรไฟล์ของ: {username}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Post #{post_id}'

@app.route('/price/<float:price>')
def show_price(price):
    return f'ราคา: {price} บาท'
```

### URL Converters

| Converter | ตัวอย่าง | ความหมาย |
|-----------|---------|-----------|
| `string` | `<string:name>` | รับข้อความ (default) |
| `int` | `<int:id>` | รับตัวเลขจำนวนเต็ม |
| `float` | `<float:price>` | รับตัวเลขทศนิยม |
| `path` | `<path:filepath>` | รับ path (รวม /) |
| `uuid` | `<uuid:token>` | รับ UUID |

```python
# ตัวอย่างการใช้ URL converters ทั้งหมด

@app.route('/user/<string:name>')
def user_by_name(name):
    return f'User: {name}'

@app.route('/post/<int:post_id>')
def post_detail(post_id):
    return f'Post ID: {post_id}'

@app.route('/price/<float:amount>')
def show_amount(amount):
    return f'Amount: {amount:.2f}'

@app.route('/files/<path:filepath>')
def serve_file(filepath):
    return f'File: {filepath}'

import uuid
@app.route('/token/<uuid:token_id>')
def verify_token(token_id):
    return f'Token: {token_id}'
```

### Multiple Routes สำหรับ Function เดียว

```python
# Function เดียวรองรับหลาย URL
@app.route('/')
@app.route('/home')
@app.route('/index')
def home():
    return 'หน้าแรก'
```

### URL Building ด้วย url_for()

```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route('/user/<username>')
def user_profile(username):
    return f'User: {username}'

# ใน application context
with app.test_request_context():
    # สร้าง URL สำหรับ function
    url = url_for('user_profile', username='somchai')
    print(url)  # /user/somchai
    
    # สร้าง URL พร้อม query string
    url2 = url_for('user_profile', username='somchai', page=2)
    print(url2)  # /user/somchai?page=2
    
    # สร้าง URL แบบ absolute
    url3 = url_for('user_profile', username='somchai', _external=True)
    print(url3)  # http://localhost:5000/user/somchai
```

### Trailing Slash Rules

```python
# URL ที่มี trailing slash (เหมือน directory)
@app.route('/projects/')
def projects():
    return 'รายการ projects'
# การเข้าถึง /projects (ไม่มี /) จะ redirect ไปที่ /projects/

# URL ที่ไม่มี trailing slash (เหมือน file)
@app.route('/about')
def about():
    return 'เกี่ยวกับเรา'
# การเข้าถึง /about/ จะได้ 404 error
```

---

## 10. HTTP Methods

### การระบุ HTTP Methods

```python
from flask import Flask, request

app = Flask(__name__)

# เฉพาะ GET (default)
@app.route('/users')
def get_users():
    return 'รายการ users'

# รองรับหลาย methods
@app.route('/users', methods=['GET', 'POST'])
def users():
    if request.method == 'GET':
        return 'ดึงรายการ users'
    elif request.method == 'POST':
        return 'สร้าง user ใหม่'

# เฉพาะ POST
@app.route('/login', methods=['POST'])
def login():
    return 'เข้าสู่ระบบ'
```

### ตัวอย่าง RESTful API Routes

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Mock database
users = [
    {'id': 1, 'name': 'สมชาย', 'email': 'somchai@example.com'},
    {'id': 2, 'name': 'สมหญิง', 'email': 'somying@example.com'},
]

@app.route('/api/users', methods=['GET'])
def get_users():
    """ดึงรายการ users ทั้งหมด"""
    return jsonify({'users': users})

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    """ดึง user ตาม id"""
    user = next((u for u in users if u['id'] == user_id), None)
    if user:
        return jsonify(user)
    return jsonify({'error': 'ไม่พบ user'}), 404

@app.route('/api/users', methods=['POST'])
def create_user():
    """สร้าง user ใหม่"""
    data = request.get_json()
    new_user = {
        'id': len(users) + 1,
        'name': data.get('name'),
        'email': data.get('email')
    }
    users.append(new_user)
    return jsonify(new_user), 201

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    """อัพเดท user"""
    user = next((u for u in users if u['id'] == user_id), None)
    if not user:
        return jsonify({'error': 'ไม่พบ user'}), 404
    
    data = request.get_json()
    user['name'] = data.get('name', user['name'])
    user['email'] = data.get('email', user['email'])
    return jsonify(user)

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    """ลบ user"""
    global users
    users = [u for u in users if u['id'] != user_id]
    return '', 204
```

### Method Decorators แบบใหม่ (Flask 2.0+)

```python
from flask import Flask

app = Flask(__name__)

# วิธีใหม่ใน Flask 2.0+
@app.get('/users')
def get_users():
    return 'GET users'

@app.post('/users')
def create_user():
    return 'POST user'

@app.put('/users/<int:id>')
def update_user(id):
    return f'PUT user {id}'

@app.delete('/users/<int:id>')
def delete_user(id):
    return f'DELETE user {id}'

@app.patch('/users/<int:id>')
def partial_update_user(id):
    return f'PATCH user {id}'
```

---

## 11. Request และ Response Objects

### Request Object

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/request-demo', methods=['GET', 'POST'])
def request_demo():
    # URL path
    print(request.path)          # /request-demo
    print(request.full_path)     # /request-demo?name=test
    print(request.url)           # http://localhost:5000/request-demo?name=test
    print(request.base_url)      # http://localhost:5000/request-demo
    
    # HTTP Method
    print(request.method)        # GET หรือ POST
    
    # Headers
    print(request.headers)       # Headers ทั้งหมด
    print(request.headers.get('Content-Type'))
    print(request.content_type)  # Content-Type header
    
    # Query string (GET parameters)
    name = request.args.get('name')      # ดึงค่าเดียว
    names = request.args.getlist('name') # ดึงหลายค่า
    all_args = request.args.to_dict()    # ดึงทั้งหมดเป็น dict
    
    return 'OK'
```

### Request Data

```python
@app.route('/data-demo', methods=['POST'])
def data_demo():
    # Form data (application/x-www-form-urlencoded)
    username = request.form.get('username')
    password = request.form.get('password')
    
    # JSON data (application/json)
    data = request.get_json()
    # หรือ
    data = request.json  # ถ้าไม่ต้องการ force=True
    
    # JSON พร้อม error handling
    data = request.get_json(force=True, silent=True)
    if data is None:
        return jsonify({'error': 'Invalid JSON'}), 400
    
    # File uploads
    file = request.files.get('photo')
    files = request.files.getlist('photos')
    
    # Cookies
    session_id = request.cookies.get('session_id')
    
    # Remote address
    ip_address = request.remote_addr
    
    return 'OK'
```

### Response Object

```python
from flask import Flask, jsonify, make_response, redirect, url_for

app = Flask(__name__)

# Response แบบ string (Flask แปลงให้เป็น Response object อัตโนมัติ)
@app.route('/simple')
def simple_response():
    return 'Hello World'  # status 200, text/html

# Response พร้อม status code
@app.route('/created')
def created_response():
    return 'Created', 201

# Response พร้อม headers
@app.route('/with-headers')
def response_with_headers():
    return 'Hello', 200, {'X-Custom-Header': 'value'}

# JSON Response
@app.route('/json')
def json_response():
    data = {'name': 'สมชาย', 'age': 25}
    return jsonify(data)  # แปลงเป็น JSON อัตโนมัติ

# make_response สำหรับ custom response
@app.route('/custom-response')
def custom_response():
    response = make_response('Custom Response')
    response.status_code = 200
    response.headers['Content-Type'] = 'text/plain'
    response.headers['X-Custom'] = 'my-value'
    response.set_cookie('user_id', '123', max_age=3600)
    return response

# Redirect
@app.route('/redirect-demo')
def redirect_demo():
    return redirect(url_for('simple_response'))

# Redirect ภายนอก
@app.route('/external-redirect')
def external_redirect():
    return redirect('https://www.google.com')
```

### Response Helpers

```python
from flask import abort, jsonify

@app.route('/protected')
def protected():
    # ถ้าไม่มีสิทธิ์ ส่ง 403
    if not is_admin():
        abort(403)
    return 'Protected content'

@app.route('/users/<int:id>')
def get_user(id):
    user = find_user(id)
    if not user:
        abort(404)  # ส่ง 404 Not Found
    return jsonify(user)
```

---

## 12. Development Server

### การรัน Development Server

```bash
# วิธีที่ 1: รันตรงๆ
python app.py

# วิธีที่ 2: ใช้ flask command
flask run

# วิธีที่ 3: ระบุ file
FLASK_APP=app.py flask run

# วิธีที่ 4: ระบุ host และ port
flask run --host=0.0.0.0 --port=8080

# วิธีที่ 5: ผ่าน environment variables
export FLASK_APP=app.py
export FLASK_ENV=development
flask run
```

### Flask CLI Commands

```bash
# ดูรายการ commands ทั้งหมด
flask --help

# รัน shell (interactive Python REPL)
flask shell

# แสดง URL rules ทั้งหมด
flask routes

# รัน server
flask run

# Custom command (จะอธิบายเพิ่มเติม)
flask db migrate  # Flask-Migrate
flask db upgrade  # Flask-Migrate
```

### การ Config ผ่าน Environment Variables

```bash
# .env file (ใช้ python-dotenv)
FLASK_APP=app.py
FLASK_ENV=development
FLASK_DEBUG=1
SECRET_KEY=your-secret-key-here
DATABASE_URL=sqlite:///app.db
```

```python
# ติดตั้ง python-dotenv
# pip install python-dotenv

from flask import Flask
from dotenv import load_dotenv
import os

# โหลด .env file
load_dotenv()

app = Flask(__name__)
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
app.config['DATABASE_URL'] = os.environ.get('DATABASE_URL')
```

---

## 13. Debug Mode

### การเปิด Debug Mode

```python
# วิธีที่ 1: ใน app.run()
app.run(debug=True)

# วิธีที่ 2: ผ่าน config
app.config['DEBUG'] = True

# วิธีที่ 3: ผ่าน environment variable
# FLASK_DEBUG=1 flask run
```

### สิ่งที่ Debug Mode ทำให้

1. **Auto-reload**: Server restart อัตโนมัติเมื่อ code เปลี่ยน
2. **Debugger**: Interactive debugger ใน browser เมื่อเกิด error
3. **Error pages**: แสดง traceback แบบละเอียด

```python
from flask import Flask

app = Flask(__name__)

@app.route('/error-demo')
def error_demo():
    # เมื่อ debug mode เปิดอยู่ จะเห็น interactive debugger ใน browser
    x = 1 / 0  # ZeroDivisionError
    return 'Never reached'

if __name__ == '__main__':
    # ห้ามเปิด debug=True ใน production!
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### คำเตือน Debug Mode

```
⚠️  WARNING: Do not use the development server in a production deployment.
Use a production WSGI server instead.
```

Debug mode ไม่ควรใช้ใน production เพราะ:
1. Werkzeug debugger มี security vulnerabilities
2. Performance ต่ำกว่า production server
3. Auto-reload ทำให้ server ช้า

---

## 14. Flask Configuration

### Configuration Methods

```python
from flask import Flask
import os

app = Flask(__name__)

# วิธีที่ 1: ตั้ง config โดยตรง
app.config['SECRET_KEY'] = 'hard-to-guess-string'
app.config['DEBUG'] = True

# วิธีที่ 2: จาก object
class Config:
    DEBUG = True
    SECRET_KEY = 'my-secret-key'
    DATABASE_URI = 'sqlite:///app.db'

app.config.from_object(Config)

# วิธีที่ 3: จาก dict
app.config.update(
    DEBUG=True,
    SECRET_KEY='my-secret-key'
)

# วิธีที่ 4: จาก environment variables
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')

# วิธีที่ 5: จาก .env file (python-dotenv)
from dotenv import load_dotenv
load_dotenv()
app.config.from_prefixed_env()  # โหลด env vars ที่ขึ้นต้นด้วย FLASK_
```

### Configuration Classes (Best Practice)

```python
# config.py
import os
from datetime import timedelta

class BaseConfig:
    """Base configuration"""
    # Security
    SECRET_KEY = os.environ.get('SECRET_KEY', 'default-secret-key')
    WTF_CSRF_ENABLED = True
    
    # Session
    PERMANENT_SESSION_LIFETIME = timedelta(days=7)
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    
    # Upload
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB
    UPLOAD_FOLDER = 'uploads'
    ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

class DevelopmentConfig(BaseConfig):
    """Development configuration"""
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'
    SQLALCHEMY_ECHO = True  # Log SQL queries
    SESSION_COOKIE_SECURE = False  # ปิดสำหรับ development

class TestingConfig(BaseConfig):
    """Testing configuration"""
    TESTING = True
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False  # ปิด CSRF สำหรับ testing

class ProductionConfig(BaseConfig):
    """Production configuration"""
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    
    # Security headers
    FORCE_HTTPS = True

config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

### Flask Built-in Configuration Keys

```python
# Configuration keys ที่สำคัญ
app.config['DEBUG'] = False                    # Debug mode
app.config['TESTING'] = False                  # Testing mode
app.config['SECRET_KEY'] = None                # Session key
app.config['SESSION_COOKIE_NAME'] = 'session' # Cookie name
app.config['SESSION_COOKIE_HTTPONLY'] = True   # Cookie httponly
app.config['SESSION_COOKIE_SECURE'] = False    # Cookie secure (HTTPS)
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(31)  # Session lifetime
app.config['MAX_CONTENT_LENGTH'] = None        # Max upload size
app.config['SEND_FILE_MAX_AGE_DEFAULT'] = None # Cache timeout
app.config['PREFERRED_URL_SCHEME'] = 'http'   # URL scheme
```

---

## 15. Application Context

### Context ใน Flask

Flask มี 2 ประเภทของ context:

1. **Application Context**: ข้อมูลเกี่ยวกับ application
2. **Request Context**: ข้อมูลเกี่ยวกับ HTTP request ปัจจุบัน

### Application Context

```python
from flask import Flask, g, current_app

app = Flask(__name__)

# current_app - proxy ไปยัง app instance ปัจจุบัน
# g - global object สำหรับเก็บข้อมูลระหว่าง request

@app.route('/')
def index():
    # current_app ใช้ได้ใน request context
    print(current_app.name)       # app
    print(current_app.debug)      # True/False
    print(current_app.config)     # Config dict
    
    # g - เก็บข้อมูลระหว่าง request lifecycle
    g.user_id = 1
    g.db_connection = get_db_connection()
    
    return 'OK'

# ใช้งาน Application Context แบบ explicit
with app.app_context():
    # ทำงานได้นอก request context
    print(current_app.name)
    
    # เหมาะสำหรับ database initialization, scripts
    # db.create_all()
```

### Request Context

```python
from flask import Flask, request, session

app = Flask(__name__)

@app.route('/context-demo')
def context_demo():
    # request - HTTP request ปัจจุบัน
    print(request.method)
    print(request.path)
    print(request.args)
    
    # session - ข้อมูล session ของ user
    session['user_id'] = 1
    print(session.get('user_id'))
    
    return 'OK'
```

### Context Hooks (Lifecycle)

```python
from flask import Flask, g, request
import sqlite3

app = Flask(__name__)

@app.before_request
def before_request():
    """ทำงานก่อนทุก request"""
    g.db = sqlite3.connect('database.db')
    print(f'Request: {request.method} {request.path}')

@app.after_request
def after_request(response):
    """ทำงานหลังทุก request (ก่อน teardown)"""
    print(f'Response: {response.status_code}')
    return response  # ต้อง return response เสมอ

@app.teardown_request
def teardown_request(exception):
    """ทำงานหลัง request จบ (แม้มี exception)"""
    db = g.pop('db', None)
    if db is not None:
        db.close()

@app.teardown_appcontext
def teardown_appcontext(exception):
    """ทำงานหลัง app context หมด"""
    pass
```

---

## 16. Blueprint เบื้องต้น

### Blueprint คืออะไร

Blueprint คือวิธีการจัดระเบียบ Flask application โดยแบ่งออกเป็นส่วนๆ (modules) แต่ละส่วนมี routes, templates, static files ของตัวเอง

ประโยชน์:
1. **Modularity**: แบ่ง code ออกเป็นส่วนๆ ง่ายต่อการบำรุงรักษา
2. **Reusability**: นำ blueprint ไปใช้ใน app อื่นได้
3. **Scalability**: รองรับ app ขนาดใหญ่

### การสร้าง Blueprint

```python
# auth/routes.py - Auth Blueprint

from flask import Blueprint, render_template, request, redirect, url_for

# สร้าง blueprint instance
auth = Blueprint('auth', __name__, 
                  url_prefix='/auth',
                  template_folder='templates')

@auth.route('/login')
def login():
    return render_template('auth/login.html')

@auth.route('/logout')
def logout():
    return redirect(url_for('main.index'))

@auth.route('/register')
def register():
    return render_template('auth/register.html')
```

```python
# main/routes.py - Main Blueprint

from flask import Blueprint, render_template

main = Blueprint('main', __name__)

@main.route('/')
def index():
    return render_template('main/index.html')

@main.route('/about')
def about():
    return render_template('main/about.html')
```

### การ Register Blueprint

```python
# app/__init__.py

from flask import Flask

def create_app():
    app = Flask(__name__)
    
    # Import and register blueprints
    from .auth.routes import auth
    app.register_blueprint(auth)  # url_prefix กำหนดใน Blueprint
    
    from .main.routes import main
    app.register_blueprint(main)
    
    # หรือกำหนด url_prefix ตอน register
    from .api.routes import api
    app.register_blueprint(api, url_prefix='/api/v1')
    
    return app
```

### URL Building กับ Blueprint

```python
from flask import url_for

# ชื่อ route ต้องใช้ pattern: blueprint_name.function_name
url_for('auth.login')      # /auth/login
url_for('auth.register')   # /auth/register
url_for('main.index')      # /
url_for('api.get_users')   # /api/v1/users
```

---

## 17. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 1: Hello World API

```python
# hello_world_api.py
# Simple REST API สำหรับ Hello World

from flask import Flask, jsonify, request
from datetime import datetime

app = Flask(__name__)

@app.route('/')
def index():
    """Root endpoint"""
    return jsonify({
        'message': 'Welcome to Hello World API',
        'version': '1.0.0',
        'endpoints': {
            'GET /': 'API information',
            'GET /hello': 'Get greeting',
            'POST /hello': 'Create personalized greeting',
            'GET /time': 'Get current time'
        }
    })

@app.route('/hello', methods=['GET'])
def get_hello():
    """GET /hello - ส่ง greeting กลับ"""
    name = request.args.get('name', 'World')
    language = request.args.get('lang', 'en')
    
    greetings = {
        'en': f'Hello, {name}!',
        'th': f'สวัสดี, {name}!',
        'ja': f'こんにちは, {name}!',
        'zh': f'你好, {name}!'
    }
    
    greeting = greetings.get(language, greetings['en'])
    
    return jsonify({
        'greeting': greeting,
        'name': name,
        'language': language
    })

@app.route('/hello', methods=['POST'])
def create_hello():
    """POST /hello - สร้าง custom greeting"""
    data = request.get_json()
    
    if not data:
        return jsonify({'error': 'ต้องส่ง JSON data'}), 400
    
    name = data.get('name')
    message = data.get('message', 'Hello')
    
    if not name:
        return jsonify({'error': 'ต้องระบุ name'}), 400
    
    return jsonify({
        'greeting': f'{message}, {name}!',
        'created_at': datetime.now().isoformat()
    }), 201

@app.route('/time')
def get_time():
    """GET /time - ส่งเวลาปัจจุบัน"""
    now = datetime.now()
    return jsonify({
        'datetime': now.isoformat(),
        'date': now.strftime('%Y-%m-%d'),
        'time': now.strftime('%H:%M:%S'),
        'timestamp': now.timestamp()
    })

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        'error': 'ไม่พบ endpoint ที่ต้องการ',
        'status': 404
    }), 404

@app.errorhandler(500)
def internal_error(error):
    return jsonify({
        'error': 'เกิดข้อผิดพลาดภายใน server',
        'status': 500
    }), 500

if __name__ == '__main__':
    print("Starting Hello World API...")
    print("Endpoints:")
    print("  GET  http://localhost:5000/")
    print("  GET  http://localhost:5000/hello?name=สมชาย&lang=th")
    print("  POST http://localhost:5000/hello")
    print("  GET  http://localhost:5000/time")
    app.run(debug=True)
```

```bash
# ทดสอบ API
curl http://localhost:5000/
curl "http://localhost:5000/hello?name=สมชาย&lang=th"
curl -X POST http://localhost:5000/hello \
     -H "Content-Type: application/json" \
     -d '{"name": "สมชาย", "message": "สวัสดี"}'
```

### ตัวอย่างที่ 2: Simple Web Application

```python
# simple_web_app.py
# Web application พร้อม HTML templates

from flask import Flask, render_template_string, request, redirect, url_for, jsonify

app = Flask(__name__)

# In-memory "database"
tasks = [
    {'id': 1, 'title': 'เรียน Flask', 'done': False},
    {'id': 2, 'title': 'สร้าง API', 'done': False},
    {'id': 3, 'title': 'Deploy app', 'done': False},
]

next_id = 4

# HTML Template (ปกติจะแยกเป็น file)
INDEX_TEMPLATE = """
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 600px; margin: 50px auto; padding: 20px; }
        h1 { color: #333; }
        .task { padding: 10px; border: 1px solid #ddd; margin: 5px 0; border-radius: 4px; display: flex; justify-content: space-between; align-items: center; }
        .done { text-decoration: line-through; color: #999; }
        input[type="text"] { width: 70%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        button { padding: 8px 16px; border: none; border-radius: 4px; cursor: pointer; }
        .btn-add { background: #007bff; color: white; }
        .btn-done { background: #28a745; color: white; font-size: 12px; }
        .btn-delete { background: #dc3545; color: white; font-size: 12px; }
        .stats { margin: 20px 0; padding: 10px; background: #f8f9fa; border-radius: 4px; }
    </style>
</head>
<body>
    <h1>Task Manager 📋</h1>
    
    <div class="stats">
        รวม: {{ tasks|length }} งาน | 
        เสร็จแล้ว: {{ tasks|selectattr('done')|list|length }} งาน |
        ยังไม่เสร็จ: {{ tasks|rejectattr('done')|list|length }} งาน
    </div>
    
    <form action="/tasks" method="post">
        <input type="text" name="title" placeholder="เพิ่ม task ใหม่..." required>
        <button type="submit" class="btn-add">เพิ่ม</button>
    </form>
    
    <div style="margin-top: 20px;">
        {% for task in tasks %}
        <div class="task">
            <span class="{{ 'done' if task.done else '' }}">{{ task.title }}</span>
            <div>
                <form action="/tasks/{{ task.id }}/toggle" method="post" style="display: inline;">
                    <button type="submit" class="btn-done">
                        {{ '↩ ยกเลิก' if task.done else '✓ เสร็จ' }}
                    </button>
                </form>
                <form action="/tasks/{{ task.id }}/delete" method="post" style="display: inline;">
                    <button type="submit" class="btn-delete">✕ ลบ</button>
                </form>
            </div>
        </div>
        {% else %}
        <p style="color: #999;">ไม่มี task ในตอนนี้</p>
        {% endfor %}
    </div>
    
    <div style="margin-top: 20px;">
        <a href="/api/tasks">ดู JSON API</a>
    </div>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(INDEX_TEMPLATE, tasks=tasks)

@app.route('/tasks', methods=['POST'])
def create_task():
    global next_id
    title = request.form.get('title', '').strip()
    
    if title:
        tasks.append({
            'id': next_id,
            'title': title,
            'done': False
        })
        next_id += 1
    
    return redirect(url_for('index'))

@app.route('/tasks/<int:task_id>/toggle', methods=['POST'])
def toggle_task(task_id):
    task = next((t for t in tasks if t['id'] == task_id), None)
    if task:
        task['done'] = not task['done']
    return redirect(url_for('index'))

@app.route('/tasks/<int:task_id>/delete', methods=['POST'])
def delete_task(task_id):
    global tasks
    tasks = [t for t in tasks if t['id'] != task_id]
    return redirect(url_for('index'))

@app.route('/api/tasks', methods=['GET'])
def api_get_tasks():
    """JSON API endpoint"""
    return jsonify({
        'tasks': tasks,
        'total': len(tasks),
        'done': sum(1 for t in tasks if t['done'])
    })

if __name__ == '__main__':
    app.run(debug=True)
```

### ตัวอย่างที่ 3: Complete Flask App with Factory Pattern

```python
# project/config.py
import os
from datetime import timedelta

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-key-change-in-prod')
    DEBUG = False
    TESTING = False

class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    SECRET_KEY = os.environ.get('SECRET_KEY')

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

```python
# project/app/__init__.py
from flask import Flask
from config import config

def create_app(config_name='default'):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    
    # Register blueprints
    from .main import main
    app.register_blueprint(main)
    
    from .api import api
    app.register_blueprint(api, url_prefix='/api')
    
    return app
```

```python
# project/app/main/__init__.py
from flask import Blueprint

main = Blueprint('main', __name__)

from . import routes
```

```python
# project/app/main/routes.py
from flask import render_template_string
from . import main

HOME_HTML = """
<!DOCTYPE html>
<html>
<head><title>My Flask App</title></head>
<body>
<h1>ยินดีต้อนรับสู่ Flask App</h1>
<p><a href="/api/status">ดู API Status</a></p>
</body>
</html>
"""

@main.route('/')
def index():
    return render_template_string(HOME_HTML)
```

```python
# project/app/api/__init__.py
from flask import Blueprint

api = Blueprint('api', __name__)

from . import routes
```

```python
# project/app/api/routes.py
from flask import jsonify
from . import api

@api.route('/status')
def status():
    return jsonify({
        'status': 'running',
        'version': '1.0.0'
    })

@api.route('/info')
def info():
    return jsonify({
        'app': 'My Flask App',
        'description': 'ตัวอย่าง Flask application ด้วย factory pattern'
    })
```

```python
# project/run.py
from app import create_app

app = create_app('development')

if __name__ == '__main__':
    app.run()
```

---

## 18. แบบฝึกหัด

### ข้อที่ 1: Simple Calculator API

สร้าง REST API สำหรับ calculator ที่รองรับ:
- `GET /calculate?a=5&b=3&op=add` (add, subtract, multiply, divide)
- Return JSON result

**เฉลย:**

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/calculate')
def calculate():
    try:
        a = float(request.args.get('a', 0))
        b = float(request.args.get('b', 0))
        op = request.args.get('op', 'add')
    except ValueError:
        return jsonify({'error': 'a และ b ต้องเป็นตัวเลข'}), 400
    
    operations = {
        'add': lambda x, y: x + y,
        'subtract': lambda x, y: x - y,
        'multiply': lambda x, y: x * y,
        'divide': lambda x, y: x / y if y != 0 else None
    }
    
    if op not in operations:
        return jsonify({'error': f'Operation {op} ไม่รองรับ'}), 400
    
    result = operations[op](a, b)
    
    if result is None:
        return jsonify({'error': 'ไม่สามารถหารด้วยศูนย์ได้'}), 400
    
    return jsonify({
        'a': a,
        'b': b,
        'operation': op,
        'result': result
    })

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 2: Student Grade API

สร้าง API สำหรับจัดการคะแนนนักเรียน:
- `GET /students` - ดูรายการทั้งหมด
- `POST /students` - เพิ่มนักเรียน
- `GET /students/<id>` - ดูข้อมูลนักเรียน
- `PUT /students/<id>` - แก้ไขข้อมูล
- `DELETE /students/<id>` - ลบนักเรียน

**เฉลย:**

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

students = {}
next_id = 1

def get_grade(score):
    if score >= 80: return 'A'
    elif score >= 70: return 'B'
    elif score >= 60: return 'C'
    elif score >= 50: return 'D'
    else: return 'F'

@app.route('/students', methods=['GET'])
def get_students():
    return jsonify({
        'students': list(students.values()),
        'total': len(students)
    })

@app.route('/students', methods=['POST'])
def create_student():
    global next_id
    data = request.get_json()
    
    if not data or 'name' not in data or 'score' not in data:
        return jsonify({'error': 'ต้องระบุ name และ score'}), 400
    
    try:
        score = float(data['score'])
        if not 0 <= score <= 100:
            raise ValueError()
    except (ValueError, TypeError):
        return jsonify({'error': 'score ต้องเป็นตัวเลข 0-100'}), 400
    
    student = {
        'id': next_id,
        'name': data['name'],
        'score': score,
        'grade': get_grade(score)
    }
    students[next_id] = student
    next_id += 1
    
    return jsonify(student), 201

@app.route('/students/<int:student_id>', methods=['GET'])
def get_student(student_id):
    student = students.get(student_id)
    if not student:
        return jsonify({'error': 'ไม่พบนักเรียน'}), 404
    return jsonify(student)

@app.route('/students/<int:student_id>', methods=['PUT'])
def update_student(student_id):
    student = students.get(student_id)
    if not student:
        return jsonify({'error': 'ไม่พบนักเรียน'}), 404
    
    data = request.get_json()
    if 'name' in data:
        student['name'] = data['name']
    if 'score' in data:
        score = float(data['score'])
        student['score'] = score
        student['grade'] = get_grade(score)
    
    return jsonify(student)

@app.route('/students/<int:student_id>', methods=['DELETE'])
def delete_student(student_id):
    if student_id not in students:
        return jsonify({'error': 'ไม่พบนักเรียน'}), 404
    del students[student_id]
    return '', 204

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 3: Configuration Management

สร้าง app ที่ใช้ configuration classes แยกสำหรับ development และ production

**เฉลย:**

```python
import os
from flask import Flask, jsonify

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-secret')
    APP_NAME = 'My App'

class DevelopmentConfig(Config):
    DEBUG = True
    ENV_NAME = 'development'
    LOG_LEVEL = 'DEBUG'

class ProductionConfig(Config):
    DEBUG = False
    ENV_NAME = 'production'
    LOG_LEVEL = 'WARNING'

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}

def create_app(config_name='default'):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    
    @app.route('/config')
    def show_config():
        return jsonify({
            'app_name': app.config['APP_NAME'],
            'env': app.config['ENV_NAME'],
            'debug': app.config['DEBUG'],
            'log_level': app.config['LOG_LEVEL']
        })
    
    return app

if __name__ == '__main__':
    env = os.environ.get('FLASK_ENV', 'development')
    app = create_app(env)
    app.run()
```

### ข้อที่ 4: Blueprint Application

สร้าง app ที่ใช้ 2 blueprints: `main` (หน้าเว็บ) และ `api` (REST API)

**เฉลย:**

```python
from flask import Flask, Blueprint, jsonify, render_template_string

# Blueprint 1: Main
main = Blueprint('main', __name__)

@main.route('/')
def index():
    return render_template_string('<h1>หน้าแรก</h1><a href="/about">เกี่ยวกับ</a>')

@main.route('/about')
def about():
    return render_template_string('<h1>เกี่ยวกับเรา</h1>')

# Blueprint 2: API
api = Blueprint('api', __name__, url_prefix='/api')

products = [
    {'id': 1, 'name': 'สินค้า A', 'price': 100},
    {'id': 2, 'name': 'สินค้า B', 'price': 200},
]

@api.route('/products')
def get_products():
    return jsonify({'products': products})

@api.route('/products/<int:product_id>')
def get_product(product_id):
    product = next((p for p in products if p['id'] == product_id), None)
    if not product:
        return jsonify({'error': 'ไม่พบสินค้า'}), 404
    return jsonify(product)

# App Factory
def create_app():
    app = Flask(__name__)
    app.register_blueprint(main)
    app.register_blueprint(api)
    return app

if __name__ == '__main__':
    app = create_app()
    app.run(debug=True)
```

### ข้อที่ 5: Request Info Inspector

สร้าง endpoint ที่แสดงข้อมูลของ HTTP request ทั้งหมด

**เฉลย:**

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/inspect', methods=['GET', 'POST', 'PUT', 'DELETE', 'PATCH'])
def inspect_request():
    """แสดงข้อมูล request ทั้งหมด"""
    info = {
        'method': request.method,
        'url': request.url,
        'path': request.path,
        'full_path': request.full_path,
        'host': request.host,
        'remote_addr': request.remote_addr,
        'headers': dict(request.headers),
        'args': dict(request.args),
        'form': dict(request.form),
        'json': request.get_json(silent=True),
        'cookies': dict(request.cookies),
        'content_type': request.content_type,
        'content_length': request.content_length,
        'is_json': request.is_json,
        'is_secure': request.is_secure,
    }
    return jsonify(info)

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 6: Error Handler

สร้าง app ที่มี custom error handlers ครบถ้วน

**เฉลย:**

```python
from flask import Flask, jsonify, request, abort

app = Flask(__name__)

# Custom Error Handlers
@app.errorhandler(400)
def bad_request(error):
    return jsonify({
        'error': 'Bad Request',
        'message': 'คำขอไม่ถูกต้อง',
        'status': 400
    }), 400

@app.errorhandler(401)
def unauthorized(error):
    return jsonify({
        'error': 'Unauthorized',
        'message': 'ต้องเข้าสู่ระบบก่อน',
        'status': 401
    }), 401

@app.errorhandler(403)
def forbidden(error):
    return jsonify({
        'error': 'Forbidden',
        'message': 'ไม่มีสิทธิ์เข้าถึง',
        'status': 403
    }), 403

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        'error': 'Not Found',
        'message': f'ไม่พบ endpoint: {request.path}',
        'status': 404
    }), 404

@app.errorhandler(500)
def internal_error(error):
    return jsonify({
        'error': 'Internal Server Error',
        'message': 'เกิดข้อผิดพลาดภายใน server',
        'status': 500
    }), 500

# Test endpoints
@app.route('/test-400')
def test_400():
    abort(400)

@app.route('/test-403')
def test_403():
    abort(403)

@app.route('/test-500')
def test_500():
    raise Exception('Test error')

@app.route('/api/data')
def get_data():
    api_key = request.headers.get('X-API-Key')
    if not api_key:
        abort(401)
    if api_key != 'secret-key':
        abort(403)
    return jsonify({'data': 'sensitive data'})

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 7: Todo API with Lifecycle Hooks

สร้าง Todo API พร้อม before_request และ after_request hooks

**เฉลย:**

```python
from flask import Flask, jsonify, request, g
from datetime import datetime
import time

app = Flask(__name__)

todos = {}
next_id = 1
request_log = []

@app.before_request
def before_request():
    """บันทึกเวลาเริ่มต้น request"""
    g.start_time = time.time()
    g.request_id = len(request_log) + 1

@app.after_request
def after_request(response):
    """บันทึก request log"""
    duration = time.time() - g.start_time
    log_entry = {
        'id': g.request_id,
        'method': request.method,
        'path': request.path,
        'status': response.status_code,
        'duration_ms': round(duration * 1000, 2),
        'timestamp': datetime.now().isoformat()
    }
    request_log.append(log_entry)
    
    # เพิ่ม response headers
    response.headers['X-Request-ID'] = str(g.request_id)
    response.headers['X-Response-Time'] = f"{log_entry['duration_ms']}ms"
    
    return response

@app.route('/todos', methods=['GET'])
def get_todos():
    return jsonify({'todos': list(todos.values())})

@app.route('/todos', methods=['POST'])
def create_todo():
    global next_id
    data = request.get_json()
    if not data or 'title' not in data:
        return jsonify({'error': 'ต้องระบุ title'}), 400
    
    todo = {
        'id': next_id,
        'title': data['title'],
        'done': False,
        'created_at': datetime.now().isoformat()
    }
    todos[next_id] = todo
    next_id += 1
    return jsonify(todo), 201

@app.route('/logs')
def get_logs():
    return jsonify({
        'logs': request_log[-20:],  # แสดง 20 รายการล่าสุด
        'total': len(request_log)
    })

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 8: Mini Library API

สร้าง API จัดการห้องสมุด: เพิ่ม/ค้นหา/ยืม/คืนหนังสือ

**เฉลย:**

```python
from flask import Flask, jsonify, request
from datetime import datetime, timedelta

app = Flask(__name__)

# Mock database
books = {
    1: {'id': 1, 'title': 'Python สำหรับผู้เริ่มต้น', 'author': 'สมชาย', 'available': True, 'borrower': None},
    2: {'id': 2, 'title': 'Flask Web Development', 'author': 'Smith', 'available': True, 'borrower': None},
    3: {'id': 3, 'title': 'Clean Code', 'author': 'Martin', 'available': True, 'borrower': None},
}
next_id = 4

@app.route('/books', methods=['GET'])
def get_books():
    available_only = request.args.get('available', '').lower() == 'true'
    search = request.args.get('q', '').lower()
    
    result = list(books.values())
    
    if available_only:
        result = [b for b in result if b['available']]
    
    if search:
        result = [b for b in result 
                  if search in b['title'].lower() or search in b['author'].lower()]
    
    return jsonify({
        'books': result,
        'total': len(result)
    })

@app.route('/books', methods=['POST'])
def add_book():
    global next_id
    data = request.get_json()
    
    if not data or not data.get('title') or not data.get('author'):
        return jsonify({'error': 'ต้องระบุ title และ author'}), 400
    
    book = {
        'id': next_id,
        'title': data['title'],
        'author': data['author'],
        'available': True,
        'borrower': None
    }
    books[next_id] = book
    next_id += 1
    return jsonify(book), 201

@app.route('/books/<int:book_id>/borrow', methods=['POST'])
def borrow_book(book_id):
    book = books.get(book_id)
    if not book:
        return jsonify({'error': 'ไม่พบหนังสือ'}), 404
    
    if not book['available']:
        return jsonify({'error': f'หนังสือถูกยืมโดย {book["borrower"]} อยู่แล้ว'}), 400
    
    data = request.get_json()
    borrower = data.get('name') if data else None
    if not borrower:
        return jsonify({'error': 'ต้องระบุชื่อผู้ยืม'}), 400
    
    book['available'] = False
    book['borrower'] = borrower
    book['due_date'] = (datetime.now() + timedelta(days=14)).strftime('%Y-%m-%d')
    
    return jsonify({
        'message': f'ยืมหนังสือ "{book["title"]}" สำเร็จ',
        'book': book,
        'due_date': book['due_date']
    })

@app.route('/books/<int:book_id>/return', methods=['POST'])
def return_book(book_id):
    book = books.get(book_id)
    if not book:
        return jsonify({'error': 'ไม่พบหนังสือ'}), 404
    
    if book['available']:
        return jsonify({'error': 'หนังสือยังไม่ได้ถูกยืม'}), 400
    
    borrower = book['borrower']
    book['available'] = True
    book['borrower'] = None
    book.pop('due_date', None)
    
    return jsonify({
        'message': f'คืนหนังสือ "{book["title"]}" โดย {borrower} สำเร็จ',
        'book': book
    })

if __name__ == '__main__':
    app.run(debug=True)
```

---

## สรุป

ใน Part 51 นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|---------------|
| Web Concepts | HTTP, REST, MVC architecture |
| Flask Basics | Installation, app creation, running |
| Routes | URL rules, variables, converters |
| HTTP Methods | GET, POST, PUT, DELETE, PATCH |
| Request/Response | Objects, data access, response creation |
| Configuration | Config classes, environments |
| App Context | g, current_app, lifecycle hooks |
| Blueprint | Modular app structure |
| App Factory | Pattern สำหรับ production apps |

### ขั้นตอนต่อไป

ใน Part 52 เราจะเรียนรู้เรื่อง:
- URL routing rules ขั้นสูง
- Jinja2 templating engine
- Template inheritance
- Static files management
- Error handlers
- Flash messages

---

*เขียนโดยหลักสูตร Python Advanced - Part 51*
