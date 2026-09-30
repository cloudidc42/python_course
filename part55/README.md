# Part 55: Flask - Authentication & Sessions

## สารบัญ
1. [Session Management](#1-session-management)
2. [Flask-Login Extension](#2-flask-login-extension)
3. [User Model กับ UserMixin](#3-user-model-กับ-usermixin)
4. [Login/Logout Flow](#4-loginlogout-flow)
5. [@login_required Decorator](#5-login_required-decorator)
6. [Password Hashing](#6-password-hashing)
7. [Remember Me Functionality](#7-remember-me-functionality)
8. [Role-Based Access Control](#8-role-based-access-control)
9. [JWT Tokens](#9-jwt-tokens)
10. [OAuth2 เบื้องต้น](#10-oauth2-เบื้องต้น)
11. [Security Best Practices](#11-security-best-practices)
12. [ตัวอย่างโปรแกรมจริง](#12-ตัวอย่างโปรแกรมจริง)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. Session Management

### Session คืออะไร

HTTP เป็น stateless protocol หมายความว่า server ไม่จำ client ระหว่าง requests ดังนั้นเราต้องใช้ **sessions** เพื่อเก็บข้อมูล state ของ user

```
Request 1: GET /login
  -> Server ส่ง login form

Request 2: POST /login (username, password)
  -> Server verify, สร้าง session, ส่ง cookie กลับ

Request 3: GET /dashboard (cookie มาด้วย)
  -> Server ตรวจสอบ cookie/session -> รู้ว่าใคร
  -> ส่ง dashboard กลับ
```

### Flask Session

```python
from flask import Flask, session, request, redirect, url_for, render_template_string

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key-here'  # จำเป็นสำหรับ session

@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')
    
    # ตรวจสอบ credentials (ตัวอย่าง)
    if username == 'admin' and password == 'secret':
        # เก็บข้อมูลใน session
        session['user_id'] = 1
        session['username'] = username
        session['is_admin'] = True
        return redirect(url_for('dashboard'))
    
    return 'Login failed', 401

@app.route('/dashboard')
def dashboard():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    return f"Welcome {session['username']}!"

@app.route('/logout')
def logout():
    session.clear()  # ลบทุกอย่างใน session
    # หรือ
    session.pop('user_id', None)  # ลบเฉพาะ key
    return redirect(url_for('login'))
```

### Session Security

```python
from flask import Flask, session
from datetime import timedelta

app = Flask(__name__)
app.config['SECRET_KEY'] = 'very-hard-to-guess-secret-key'

# Session lifetime
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(days=7)

# Cookie security settings
app.config['SESSION_COOKIE_SECURE'] = True      # HTTPS only
app.config['SESSION_COOKIE_HTTPONLY'] = True    # ไม่ให้ JavaScript อ่าน
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # CSRF protection

@app.route('/login', methods=['POST'])
def login():
    # ทำ login...
    session.permanent = True  # ใช้ PERMANENT_SESSION_LIFETIME
    session['user_id'] = user.id
    return redirect(url_for('index'))
```

### Server-side Sessions ด้วย Flask-Session

```bash
pip install flask-session
pip install redis  # สำหรับ Redis backend
```

```python
from flask import Flask, session
from flask_session import Session

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'

# ใช้ filesystem (development)
app.config['SESSION_TYPE'] = 'filesystem'
app.config['SESSION_FILE_DIR'] = '/tmp/flask-sessions'

# ใช้ Redis (production)
import redis
app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://localhost:6379')

Session(app)  # Initialize Flask-Session

@app.route('/login', methods=['POST'])
def login():
    session['user_id'] = 123  # เก็บบน server แทน cookie
    return 'Logged in'
```

---

## 2. Flask-Login Extension

### Flask-Login คืออะไร

Flask-Login เป็น extension ที่จัดการ user authentication ใน Flask โดยมีคุณสมบัติ:
1. จัดการ user session อัตโนมัติ
2. `current_user` proxy ใน templates
3. `@login_required` decorator
4. "Remember me" functionality
5. ป้องกัน views ที่ต้อง login

```bash
pip install flask-login
```

### Setup Flask-Login

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

db = SQLAlchemy(app)
login_manager = LoginManager(app)

# กำหนด route สำหรับ login (redirect ถ้าไม่ได้ login)
login_manager.login_view = 'auth.login'

# ข้อความที่แสดงเมื่อ redirect ไปหน้า login
login_manager.login_message = 'กรุณาเข้าสู่ระบบก่อนเข้าถึงหน้านี้'
login_manager.login_message_category = 'warning'

# User loader - บอก Flask-Login วิธีโหลด user จาก session
@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

---

## 3. User Model กับ UserMixin

### UserMixin

`UserMixin` เป็น class ที่ให้ methods ที่ Flask-Login ต้องการ:

| Method/Property | ความหมาย |
|----------------|----------|
| `is_authenticated` | True ถ้า user login แล้ว |
| `is_active` | True ถ้า account ยังใช้งานได้ |
| `is_anonymous` | True ถ้าเป็น anonymous user |
| `get_id()` | คืน unique identifier ของ user |

```python
from flask_login import UserMixin
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime

db = SQLAlchemy()

class User(UserMixin, db.Model):
    """User model กับ Flask-Login integration"""
    
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    _password_hash = db.Column('password_hash', db.String(256), nullable=False)
    full_name = db.Column(db.String(200))
    bio = db.Column(db.Text)
    avatar_url = db.Column(db.String(500))
    role = db.Column(db.String(20), default='user')  # 'user', 'moderator', 'admin'
    is_active = db.Column(db.Boolean, default=True, nullable=False)
    is_verified = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    # Password property
    @property
    def password(self):
        raise AttributeError('password is not readable directly')
    
    @password.setter
    def password(self, password):
        """Hash และเก็บ password"""
        self._password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        """ตรวจสอบ password"""
        return check_password_hash(self._password_hash, password)
    
    # UserMixin methods (override ถ้าต้องการ)
    def get_id(self):
        return str(self.id)
    
    @property
    def is_authenticated(self):
        return True
    
    @property
    def is_active(self):
        return self._is_active
    
    @is_active.setter
    def is_active(self, value):
        self._is_active = value
    
    # Custom properties
    @property
    def is_admin(self):
        return self.role == 'admin'
    
    @property
    def is_moderator(self):
        return self.role in ['admin', 'moderator']
    
    def has_role(self, role):
        """ตรวจสอบว่า user มี role ที่กำหนด"""
        role_hierarchy = {'user': 0, 'moderator': 1, 'admin': 2}
        user_level = role_hierarchy.get(self.role, 0)
        required_level = role_hierarchy.get(role, 0)
        return user_level >= required_level
    
    def update_last_login(self):
        """อัพเดทเวลา login ล่าสุด"""
        self.last_login = datetime.utcnow()
        db.session.commit()
    
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'full_name': self.full_name,
            'role': self.role,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat()
        }
    
    def __repr__(self):
        return f'<User {self.username}>'
```

### Anonymous User

```python
from flask_login import AnonymousUserMixin

class CustomAnonymousUser(AnonymousUserMixin):
    """Custom anonymous user"""
    
    def has_role(self, role):
        return False
    
    @property
    def is_admin(self):
        return False

# กำหนด anonymous user class
login_manager.anonymous_user = CustomAnonymousUser
```

---

## 4. Login/Logout Flow

### Complete Login Flow

```python
from flask import Flask, render_template_string, redirect, url_for, flash, request
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, login_user, logout_user, current_user, login_required
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

app = Flask(__name__)
app.config['SECRET_KEY'] = 'auth-secret-key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///auth.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
login_manager = LoginManager(app)
login_manager.login_view = 'login'
login_manager.login_message = 'กรุณาเข้าสู่ระบบ'
login_manager.login_message_category = 'warning'

class LoginForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[DataRequired()])
    password = PasswordField('รหัสผ่าน', validators=[DataRequired()])
    remember = BooleanField('จดจำฉัน')
    submit = SubmitField('เข้าสู่ระบบ')

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

@app.route('/login', methods=['GET', 'POST'])
def login():
    # ถ้า login แล้ว redirect ไปหน้า dashboard
    if current_user.is_authenticated:
        return redirect(url_for('dashboard'))
    
    form = LoginForm()
    
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        
        # ตรวจสอบ user และ password
        if user is None or not user.check_password(form.password.data):
            flash('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง', 'danger')
            return redirect(url_for('login'))
        
        # ตรวจสอบ account status
        if not user.is_active:
            flash('บัญชีนี้ถูกระงับการใช้งาน', 'danger')
            return redirect(url_for('login'))
        
        # Login user
        login_user(user, remember=form.remember.data)
        
        # อัพเดทเวลา login
        user.update_last_login()
        
        flash(f'ยินดีต้อนรับ {user.full_name or user.username}!', 'success')
        
        # Redirect ไปยัง next page (ถ้ามี)
        next_page = request.args.get('next')
        if not next_page or not next_page.startswith('/'):
            next_page = url_for('dashboard')
        
        return redirect(next_page)
    
    return render_template_string(LOGIN_HTML, form=form)

@app.route('/logout')
@login_required
def logout():
    username = current_user.username
    logout_user()
    flash(f'ออกจากระบบแล้ว ({username})', 'info')
    return redirect(url_for('login'))

@app.route('/dashboard')
@login_required
def dashboard():
    return render_template_string('''
    <h1>Dashboard</h1>
    <p>ยินดีต้อนรับ {{ current_user.username }}!</p>
    <p>Role: {{ current_user.role }}</p>
    <a href="{{ url_for('logout') }}">ออกจากระบบ</a>
    ''')

LOGIN_HTML = """
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>Login</title>
<style>
body{font-family:Arial;display:flex;justify-content:center;align-items:center;min-height:100vh;margin:0;background:#f5f5f5}
.login-box{background:white;padding:30px;border-radius:10px;box-shadow:0 4px 20px rgba(0,0,0,0.1);width:350px}
h2{text-align:center;color:#2c3e50;margin-bottom:25px}
.form-group{margin-bottom:15px}
label{display:block;font-weight:bold;margin-bottom:5px;color:#555;font-size:0.9em}
input[type=text],input[type=password]{width:100%;padding:10px;border:1px solid #ddd;border-radius:6px;box-sizing:border-box}
.check-group{display:flex;align-items:center;gap:8px;margin:10px 0}
.btn{width:100%;padding:12px;background:#3498db;color:white;border:none;border-radius:6px;font-size:1em;cursor:pointer}
.btn:hover{background:#2980b9}
.alert{padding:10px;border-radius:6px;margin-bottom:15px;font-size:0.9em}
.alert-success{background:#d4edda;color:#155724}
.alert-danger{background:#f8d7da;color:#721c24}
.alert-warning{background:#fff3cd;color:#856404}
.error-msg{color:red;font-size:0.82em;margin-top:3px}
</style>
</head>
<body>
<div class="login-box">
    <h2>🔐 เข้าสู่ระบบ</h2>
    
    {% with messages = get_flashed_messages(with_categories=true) %}
    {% for category, message in messages %}
    <div class="alert alert-{{ category }}">{{ message }}</div>
    {% endfor %}
    {% endwith %}
    
    <form method="POST">
        {{ form.hidden_tag() }}
        <div class="form-group">
            {{ form.username.label }}
            {{ form.username(placeholder="ชื่อผู้ใช้") }}
            {% for err in form.username.errors %}<div class="error-msg">{{ err }}</div>{% endfor %}
        </div>
        <div class="form-group">
            {{ form.password.label }}
            {{ form.password(placeholder="รหัสผ่าน") }}
            {% for err in form.password.errors %}<div class="error-msg">{{ err }}</div>{% endfor %}
        </div>
        <div class="check-group">
            {{ form.remember() }}
            {{ form.remember.label }}
        </div>
        {{ form.submit(class="btn") }}
    </form>
    <p style="text-align:center;margin-top:15px;font-size:0.9em">
        ยังไม่มีบัญชี? <a href="/register">สมัครสมาชิก</a>
    </p>
</div>
</body>
</html>
"""
```

---

## 5. @login_required Decorator

### พื้นฐาน @login_required

```python
from flask_login import login_required, current_user

@app.route('/profile')
@login_required
def profile():
    """ต้อง login ก่อนเข้า"""
    return f'Profile: {current_user.username}'

@app.route('/settings')
@login_required
def settings():
    """ถ้าไม่ได้ login จะ redirect ไปยัง login_manager.login_view"""
    return 'Settings page'
```

### Custom Decorators

```python
from functools import wraps
from flask import flash, redirect, url_for, abort
from flask_login import current_user

def admin_required(f):
    """Decorator ที่ต้องการ admin role"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_authenticated:
            return redirect(url_for('login'))
        if not current_user.is_admin:
            abort(403)
        return f(*args, **kwargs)
    return decorated_function

def role_required(role):
    """Decorator factory ที่รับ role parameter"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.is_authenticated:
                flash('กรุณาเข้าสู่ระบบ', 'warning')
                return redirect(url_for('login', next=request.url))
            if not current_user.has_role(role):
                flash('คุณไม่มีสิทธิ์เข้าถึงหน้านี้', 'danger')
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator

def verified_required(f):
    """ต้อง verify email แล้ว"""
    @wraps(f)
    @login_required
    def decorated_function(*args, **kwargs):
        if not current_user.is_verified:
            flash('กรุณายืนยัน email ก่อน', 'warning')
            return redirect(url_for('verify_email'))
        return f(*args, **kwargs)
    return decorated_function

# การใช้งาน
@app.route('/admin')
@admin_required
def admin_panel():
    return 'Admin panel'

@app.route('/moderate')
@role_required('moderator')
def moderate():
    return 'Moderation panel'

@app.route('/premium')
@verified_required
def premium_content():
    return 'Premium content'
```

### current_user ใน Templates

```html
<!-- templates/base.html -->
{% if current_user.is_authenticated %}
    <p>สวัสดี, {{ current_user.username }}</p>
    
    {% if current_user.is_admin %}
    <a href="{{ url_for('admin.index') }}">Admin Panel</a>
    {% endif %}
    
    <a href="{{ url_for('logout') }}">ออกจากระบบ</a>
{% else %}
    <a href="{{ url_for('login') }}">เข้าสู่ระบบ</a>
    <a href="{{ url_for('register') }}">สมัครสมาชิก</a>
{% endif %}
```

---

## 6. Password Hashing

### werkzeug.security

```python
from werkzeug.security import generate_password_hash, check_password_hash

# สร้าง password hash
password = 'MySecurePassword123!'
hashed = generate_password_hash(password)
print(hashed)
# scrypt:32768:8:1$...$...

# ตรวจสอบ password
is_correct = check_password_hash(hashed, password)
print(is_correct)  # True

is_wrong = check_password_hash(hashed, 'WrongPassword')
print(is_wrong)    # False

# กำหนด method
hashed_pbkdf2 = generate_password_hash(password, method='pbkdf2:sha256', salt_length=16)
hashed_scrypt = generate_password_hash(password, method='scrypt')
```

### bcrypt

```bash
pip install flask-bcrypt
```

```python
from flask import Flask
from flask_bcrypt import Bcrypt

app = Flask(__name__)
bcrypt = Bcrypt(app)

# Hash password
password = 'MyPassword123!'
hashed = bcrypt.generate_password_hash(password).decode('utf-8')
print(hashed)  # $2b$12$...

# ตรวจสอบ
is_correct = bcrypt.check_password_hash(hashed, password)
print(is_correct)  # True

# กำหนด rounds (default 12)
hashed_strong = bcrypt.generate_password_hash(password, rounds=14).decode('utf-8')

# ใน User model
class User(db.Model):
    _password_hash = db.Column('password_hash', db.String(256))
    
    @property
    def password(self):
        raise AttributeError('Not readable')
    
    @password.setter
    def password(self, raw_password):
        self._password_hash = bcrypt.generate_password_hash(raw_password).decode('utf-8')
    
    def check_password(self, raw_password):
        return bcrypt.check_password_hash(self._password_hash, raw_password)
```

### Password Validation

```python
import re

class PasswordValidator:
    """Validate password strength"""
    
    def __init__(self, min_length=8, require_upper=True, 
                 require_lower=True, require_digit=True,
                 require_special=False):
        self.min_length = min_length
        self.require_upper = require_upper
        self.require_lower = require_lower
        self.require_digit = require_digit
        self.require_special = require_special
    
    def validate(self, password):
        errors = []
        
        if len(password) < self.min_length:
            errors.append(f'ต้องมีอย่างน้อย {self.min_length} ตัวอักษร')
        
        if self.require_upper and not re.search(r'[A-Z]', password):
            errors.append('ต้องมีตัวพิมพ์ใหญ่ (A-Z)')
        
        if self.require_lower and not re.search(r'[a-z]', password):
            errors.append('ต้องมีตัวพิมพ์เล็ก (a-z)')
        
        if self.require_digit and not re.search(r'[0-9]', password):
            errors.append('ต้องมีตัวเลข (0-9)')
        
        if self.require_special and not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            errors.append('ต้องมีอักขระพิเศษ')
        
        return errors
    
    def is_valid(self, password):
        return len(self.validate(password)) == 0

# ใช้งาน
validator = PasswordValidator(min_length=10, require_special=True)
errors = validator.validate('weak')
print(errors)  # ['ต้องมีอย่างน้อย 10 ตัวอักษร', ...]
```

### Password Reset Flow

```python
from itsdangerous import URLSafeTimedSerializer, SignatureExpired, BadSignature

def generate_reset_token(email, secret_key, salt='password-reset-salt'):
    """สร้าง token สำหรับ reset password"""
    serializer = URLSafeTimedSerializer(secret_key)
    return serializer.dumps(email, salt=salt)

def verify_reset_token(token, secret_key, salt='password-reset-salt', max_age=3600):
    """ตรวจสอบ token (1 ชั่วโมง)"""
    serializer = URLSafeTimedSerializer(secret_key)
    try:
        email = serializer.loads(token, salt=salt, max_age=max_age)
        return email
    except (SignatureExpired, BadSignature):
        return None

# Usage in routes
@app.route('/forgot-password', methods=['GET', 'POST'])
def forgot_password():
    if request.method == 'POST':
        email = request.form.get('email')
        user = User.query.filter_by(email=email).first()
        
        if user:
            token = generate_reset_token(email, app.config['SECRET_KEY'])
            reset_url = url_for('reset_password', token=token, _external=True)
            
            # ส่ง email (ต้องใช้ Flask-Mail)
            # send_reset_email(user.email, reset_url)
            print(f'Reset URL: {reset_url}')  # สำหรับทดสอบ
        
        flash('ถ้าอีเมลนี้มีในระบบ เราจะส่งลิงก์ reset password ให้', 'info')
    
    return render_template_string('<form method="POST"><input name="email" type="email" placeholder="อีเมล" required><button>ส่ง</button>{{ csrf_token }}</form>')

@app.route('/reset-password/<token>', methods=['GET', 'POST'])
def reset_password(token):
    email = verify_reset_token(token, app.config['SECRET_KEY'])
    
    if not email:
        flash('ลิงก์หมดอายุหรือไม่ถูกต้อง', 'danger')
        return redirect(url_for('forgot_password'))
    
    if request.method == 'POST':
        new_password = request.form.get('password')
        user = User.query.filter_by(email=email).first()
        
        if user:
            user.password = new_password
            db.session.commit()
            flash('เปลี่ยนรหัสผ่านสำเร็จ', 'success')
            return redirect(url_for('login'))
    
    return render_template_string('<form method="POST"><input name="password" type="password" placeholder="รหัสผ่านใหม่" required><button>เปลี่ยนรหัสผ่าน</button></form>')
```

---

## 7. Remember Me Functionality

### การทำงาน Remember Me

```python
from flask_login import login_user
from datetime import timedelta

@app.route('/login', methods=['POST'])
def login():
    user = User.query.filter_by(username=request.form.get('username')).first()
    
    if user and user.check_password(request.form.get('password')):
        remember = request.form.get('remember', 'false') == 'true'
        
        # login_user กับ remember=True จะสร้าง long-lived cookie
        login_user(user, remember=remember)
        
        return redirect(url_for('dashboard'))
    
    return 'Login failed', 401

# กำหนดอายุของ "remember me" cookie
app.config['REMEMBER_COOKIE_DURATION'] = timedelta(days=30)
app.config['REMEMBER_COOKIE_SECURE'] = True
app.config['REMEMBER_COOKIE_HTTPONLY'] = True
app.config['REMEMBER_COOKIE_SAMESITE'] = 'Lax'
```

### Fresh Login

```python
from flask_login import login_fresh, fresh_login_required

@app.route('/change-password', methods=['GET', 'POST'])
@fresh_login_required  # ต้อง login ใหม่ล่าสุด (ไม่ใช้ remember me)
def change_password():
    """เฉพาะ fresh login เท่านั้น (ไม่ใช่จาก remember me cookie)"""
    if request.method == 'POST':
        if not current_user.check_password(request.form.get('current_password')):
            flash('รหัสผ่านปัจจุบันไม่ถูกต้อง', 'danger')
            return redirect(url_for('change_password'))
        
        current_user.password = request.form.get('new_password')
        db.session.commit()
        flash('เปลี่ยนรหัสผ่านสำเร็จ', 'success')
        return redirect(url_for('profile'))
    
    return render_template_string('Change Password form...')
```

---

## 8. Role-Based Access Control

### RBAC พื้นฐาน

```python
from flask import Flask, abort, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, login_required, current_user
from functools import wraps

app = Flask(__name__)
db = SQLAlchemy(app)
login_manager = LoginManager(app)

# Role hierarchy
ROLES = {
    'guest': 0,
    'user': 1,
    'moderator': 2,
    'admin': 3,
    'superadmin': 4
}

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), nullable=False)
    role = db.Column(db.String(20), default='user')
    
    def has_role(self, required_role):
        return ROLES.get(self.role, 0) >= ROLES.get(required_role, 0)
    
    @property
    def is_admin(self):
        return self.role in ('admin', 'superadmin')

def require_role(role):
    """Decorator สำหรับ role-based access"""
    def decorator(func):
        @wraps(func)
        @login_required
        def wrapper(*args, **kwargs):
            if not current_user.has_role(role):
                if request.is_json:
                    return jsonify({'error': 'Insufficient permissions'}), 403
                abort(403)
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Routes
@app.route('/user-only')
@require_role('user')
def user_only():
    return 'User content'

@app.route('/moderator-panel')
@require_role('moderator')
def moderator_panel():
    return 'Moderator panel'

@app.route('/admin-panel')
@require_role('admin')
def admin_panel():
    return 'Admin panel'
```

### Permission-based RBAC

```python
# การจัดการ permissions แบบละเอียด

class Permission:
    """Permission constants"""
    READ = 'read'
    WRITE = 'write'
    DELETE = 'delete'
    ADMIN = 'admin'
    MODERATE = 'moderate'

# Role permissions mapping
ROLE_PERMISSIONS = {
    'user': {Permission.READ, Permission.WRITE},
    'moderator': {Permission.READ, Permission.WRITE, Permission.MODERATE},
    'admin': {Permission.READ, Permission.WRITE, Permission.DELETE, 
              Permission.MODERATE, Permission.ADMIN},
}

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
    role = db.Column(db.String(20), default='user')
    
    def has_permission(self, permission):
        role_perms = ROLE_PERMISSIONS.get(self.role, set())
        return permission in role_perms
    
    def can(self, permission):
        return self.has_permission(permission)

def permission_required(permission):
    """Decorator สำหรับ permission-based access"""
    def decorator(func):
        @wraps(func)
        @login_required
        def wrapper(*args, **kwargs):
            if not current_user.can(permission):
                abort(403)
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@app.route('/posts/<int:id>', methods=['DELETE'])
@permission_required(Permission.DELETE)
def delete_post(id):
    # ลบ post
    pass

@app.route('/moderate')
@permission_required(Permission.MODERATE)
def moderate():
    # จัดการ content
    pass
```

---

## 9. JWT Tokens

### Flask-JWT-Extended

```bash
pip install flask-jwt-extended
```

### JWT Setup

```python
from flask import Flask, jsonify, request
from flask_jwt_extended import (
    JWTManager, create_access_token, create_refresh_token,
    jwt_required, get_jwt_identity, get_jwt,
    set_access_cookies, set_refresh_cookies,
    unset_jwt_cookies
)
from datetime import timedelta

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'jwt-secret-key-change-in-production'
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)
app.config['JWT_REFRESH_TOKEN_EXPIRES'] = timedelta(days=30)

jwt = JWTManager(app)

# Custom response เมื่อ token ไม่ถูกต้อง
@jwt.expired_token_loader
def expired_token_callback(jwt_header, jwt_payload):
    return jsonify({'error': 'Token หมดอายุ', 'code': 'token_expired'}), 401

@jwt.invalid_token_loader
def invalid_token_callback(error):
    return jsonify({'error': 'Token ไม่ถูกต้อง', 'code': 'invalid_token'}), 401

@jwt.unauthorized_loader
def missing_token_callback(error):
    return jsonify({'error': 'ต้องการ token', 'code': 'missing_token'}), 401
```

### JWT Login/Logout

```python
@app.route('/api/auth/login', methods=['POST'])
def jwt_login():
    """Login และรับ JWT tokens"""
    data = request.get_json()
    
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'error': 'ต้องระบุ username และ password'}), 400
    
    user = User.query.filter_by(username=data['username']).first()
    
    if not user or not user.check_password(data['password']):
        return jsonify({'error': 'Credentials ไม่ถูกต้อง'}), 401
    
    if not user.is_active:
        return jsonify({'error': 'บัญชีถูกระงับ'}), 401
    
    # สร้าง tokens
    additional_claims = {
        'user_id': user.id,
        'username': user.username,
        'role': user.role,
        'email': user.email
    }
    
    access_token = create_access_token(
        identity=user.id,
        additional_claims=additional_claims
    )
    refresh_token = create_refresh_token(identity=user.id)
    
    return jsonify({
        'access_token': access_token,
        'refresh_token': refresh_token,
        'user': user.to_dict()
    })

@app.route('/api/auth/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh_token():
    """Refresh access token ด้วย refresh token"""
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    
    if not user or not user.is_active:
        return jsonify({'error': 'User ไม่ถูกต้อง'}), 401
    
    new_access_token = create_access_token(identity=user_id)
    return jsonify({'access_token': new_access_token})

@app.route('/api/auth/logout', methods=['DELETE'])
@jwt_required()
def jwt_logout():
    """ออกจากระบบ (JWT blacklist)"""
    # ต้องใช้ JWT blacklist (ดูด้านล่าง)
    jti = get_jwt()['jti']
    # blacklist_token(jti)
    return jsonify({'message': 'ออกจากระบบสำเร็จ'})

@app.route('/api/profile')
@jwt_required()
def get_profile():
    """Protected endpoint"""
    user_id = get_jwt_identity()
    claims = get_jwt()
    
    user = User.query.get(user_id)
    if not user:
        return jsonify({'error': 'User ไม่พบ'}), 404
    
    return jsonify({
        'user': user.to_dict(),
        'claims': {
            'user_id': claims.get('user_id'),
            'role': claims.get('role')
        }
    })
```

### JWT Token Blacklist

```python
from flask_sqlalchemy import SQLAlchemy
from flask_jwt_extended import get_jti

db = SQLAlchemy()

class TokenBlocklist(db.Model):
    """Table เก็บ revoked JWT tokens"""
    __tablename__ = 'token_blocklist'
    
    id = db.Column(db.Integer, primary_key=True)
    jti = db.Column(db.String(36), nullable=False, unique=True, index=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

# Check token blacklist
@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_payload):
    jti = jwt_payload['jti']
    token = TokenBlocklist.query.filter_by(jti=jti).first()
    return token is not None

@app.route('/api/auth/logout', methods=['DELETE'])
@jwt_required()
def jwt_logout():
    jti = get_jwt()['jti']
    
    # เพิ่มใน blacklist
    blocked_token = TokenBlocklist(jti=jti)
    db.session.add(blocked_token)
    db.session.commit()
    
    return jsonify({'message': 'ออกจากระบบสำเร็จ'})
```

### ใช้ JWT กับ Role Permissions

```python
from flask_jwt_extended import jwt_required, get_jwt
from functools import wraps

def jwt_role_required(role):
    """JWT decorator สำหรับ role-based access"""
    def decorator(func):
        @wraps(func)
        @jwt_required()
        def wrapper(*args, **kwargs):
            claims = get_jwt()
            user_role = claims.get('role', 'user')
            
            role_levels = {'user': 1, 'moderator': 2, 'admin': 3}
            if role_levels.get(user_role, 0) < role_levels.get(role, 0):
                return jsonify({'error': f'ต้องการ role: {role}'}), 403
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@app.route('/api/admin/users')
@jwt_role_required('admin')
def admin_list_users():
    users = User.query.all()
    return jsonify({'users': [u.to_dict() for u in users]})
```

---

## 10. OAuth2 เบื้องต้น

### OAuth2 Flow

```
User -> App -> Authorization Server (Google/GitHub)
                    |
                    v (User grants permission)
App <- Authorization Server (Authorization Code)
    |
    v (Exchange code for token)
Authorization Server -> App (Access Token)
    |
    v
App -> Resource Server (Google API/GitHub API)
    -> User Profile, Email, etc.
```

### Flask-OAuthlib / Authlib

```bash
pip install authlib flask-dance
```

### OAuth2 กับ Google (Flask-Dance)

```python
from flask import Flask, redirect, url_for, flash
from flask_dance.contrib.google import make_google_blueprint, google
from flask_login import login_user, LoginManager

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'
app.config['GOOGLE_OAUTH_CLIENT_ID'] = 'your-client-id'
app.config['GOOGLE_OAUTH_CLIENT_SECRET'] = 'your-client-secret'

# สร้าง blueprint สำหรับ Google OAuth
google_bp = make_google_blueprint(
    scope=['openid', 'email', 'profile'],
    redirect_to='google_login'
)
app.register_blueprint(google_bp, url_prefix='/google_login')

@app.route('/google/login')
def google_login():
    if not google.authorized:
        return redirect(url_for('google.login'))
    
    # ดึงข้อมูล user จาก Google
    resp = google.get('/oauth2/v2/userinfo')
    if not resp.ok:
        flash('ไม่สามารถดึงข้อมูลจาก Google', 'error')
        return redirect(url_for('login'))
    
    user_info = resp.json()
    email = user_info.get('email')
    google_id = user_info.get('id')
    name = user_info.get('name')
    
    # ตรวจสอบว่ามี user ใน database แล้วหรือยัง
    user = User.query.filter_by(email=email).first()
    
    if not user:
        # สร้าง user ใหม่
        user = User(
            username=email.split('@')[0],
            email=email,
            full_name=name,
            oauth_provider='google',
            oauth_id=google_id,
            is_verified=True
        )
        import secrets
        user._password_hash = secrets.token_hex(32)  # Random password สำหรับ OAuth users
        db.session.add(user)
        db.session.commit()
    
    login_user(user)
    flash(f'เข้าสู่ระบบสำเร็จ! ยินดีต้อนรับ {name}', 'success')
    return redirect(url_for('dashboard'))
```

### OAuth2 กับ GitHub (Authlib)

```python
from authlib.integrations.flask_client import OAuth
from flask import Flask, redirect, url_for, session, jsonify

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'

oauth = OAuth(app)

# Register GitHub OAuth
github = oauth.register(
    name='github',
    client_id='your-github-client-id',
    client_secret='your-github-client-secret',
    access_token_url='https://github.com/login/oauth/access_token',
    access_token_params=None,
    authorize_url='https://github.com/login/oauth/authorize',
    authorize_params=None,
    api_base_url='https://api.github.com/',
    client_kwargs={'scope': 'user:email'},
)

@app.route('/login/github')
def github_login():
    redirect_uri = url_for('github_callback', _external=True)
    return github.authorize_redirect(redirect_uri)

@app.route('/login/github/callback')
def github_callback():
    token = github.authorize_access_token()
    resp = github.get('user', token=token)
    user_info = resp.json()
    
    github_id = str(user_info['id'])
    email = user_info.get('email', f'{user_info["login"]}@github.com')
    name = user_info.get('name', user_info['login'])
    
    # Find or create user
    user = User.query.filter_by(oauth_provider='github', oauth_id=github_id).first()
    
    if not user:
        user = User(
            username=user_info['login'],
            email=email,
            full_name=name,
            oauth_provider='github',
            oauth_id=github_id,
            is_verified=True
        )
        import secrets
        user._password_hash = secrets.token_hex(32)
        db.session.add(user)
        db.session.commit()
    
    login_user(user)
    return redirect(url_for('dashboard'))
```

---

## 11. Security Best Practices

### Input Validation

```python
from flask import request
import re
import html

def sanitize_input(value, max_length=None):
    """Sanitize user input"""
    if value is None:
        return None
    
    # Strip whitespace
    value = str(value).strip()
    
    # Escape HTML (ป้องกัน XSS)
    # value = html.escape(value)  # ถ้าต้องการ escape HTML
    
    # จำกัดความยาว
    if max_length and len(value) > max_length:
        value = value[:max_length]
    
    return value

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def validate_username(username):
    # ตัวอักษร ตัวเลข และ _ เท่านั้น, 3-20 ตัวอักษร
    pattern = r'^[a-zA-Z0-9_]{3,20}$'
    return bool(re.match(pattern, username))
```

### Rate Limiting

```bash
pip install flask-limiter
```

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=['200 per day', '50 per hour']
)

# จำกัดการ login ป้องกัน brute force
@app.route('/login', methods=['POST'])
@limiter.limit('5 per minute')
def login():
    pass

# จำกัด API requests
@app.route('/api/data')
@limiter.limit('30 per minute')
def api_data():
    pass
```

### Secure Headers

```python
# Security headers
@app.after_request
def add_security_headers(response):
    """เพิ่ม security headers ทุก response"""
    # ป้องกัน clickjacking
    response.headers['X-Frame-Options'] = 'DENY'
    
    # ป้องกัน XSS
    response.headers['X-XSS-Protection'] = '1; mode=block'
    
    # ป้องกัน MIME type sniffing
    response.headers['X-Content-Type-Options'] = 'nosniff'
    
    # HTTPS only
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    
    # Content Security Policy
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self' https://cdnjs.cloudflare.com; "
        "style-src 'self' https://fonts.googleapis.com; "
        "img-src 'self' data:; "
    )
    
    # Referrer policy
    response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    
    return response
```

### Logging Security Events

```python
import logging
from datetime import datetime

# Security logger
security_logger = logging.getLogger('security')
security_handler = logging.FileHandler('security.log')
security_handler.setFormatter(logging.Formatter(
    '%(asctime)s - %(levelname)s - %(message)s'
))
security_logger.addHandler(security_handler)
security_logger.setLevel(logging.INFO)

def log_security_event(event_type, user_id=None, ip=None, details=None):
    """บันทึก security events"""
    security_logger.info(
        f'Event: {event_type} | User: {user_id} | IP: {ip} | Details: {details}'
    )

# ใช้งาน
@app.route('/login', methods=['POST'])
def login():
    ip = request.remote_addr
    username = request.form.get('username')
    
    user = User.query.filter_by(username=username).first()
    
    if not user or not user.check_password(request.form.get('password')):
        log_security_event('LOGIN_FAILED', ip=ip, details=f'username: {username}')
        # Increment failed attempts
        return jsonify({'error': 'Credentials ไม่ถูกต้อง'}), 401
    
    log_security_event('LOGIN_SUCCESS', user_id=user.id, ip=ip)
    login_user(user)
    return jsonify({'message': 'Login successful'})
```

---

## 12. ตัวอย่างโปรแกรมจริง

### Complete Authentication System

```python
# auth_system.py
# ระบบ authentication ครบถ้วน

from flask import Flask, jsonify, request, redirect, url_for, flash, render_template_string
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user
from flask_bcrypt import Bcrypt
from flask_wtf import FlaskForm
from flask_wtf.csrf import CSRFProtect
from wtforms import StringField, PasswordField, BooleanField, SubmitField, EmailField
from wtforms.validators import DataRequired, Email, Length, EqualTo, ValidationError
from datetime import datetime, timedelta
from functools import wraps
import re

app = Flask(__name__)
app.config['SECRET_KEY'] = 'auth-system-secret-key-2024'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///auth_system.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(days=7)
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['WTF_CSRF_ENABLED'] = True

db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
csrf = CSRFProtect(app)
login_manager = LoginManager(app)
login_manager.login_view = 'login'
login_manager.login_message = 'กรุณาเข้าสู่ระบบก่อน'
login_manager.login_message_category = 'warning'

# User Model
class User(UserMixin, db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    _password_hash = db.Column('password_hash', db.String(256), nullable=False)
    full_name = db.Column(db.String(200))
    role = db.Column(db.String(20), default='user')
    is_active = db.Column(db.Boolean, default=True)
    failed_login_count = db.Column(db.Integer, default=0)
    locked_until = db.Column(db.DateTime)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    @property
    def password(self):
        raise AttributeError('ไม่สามารถอ่าน password โดยตรง')
    
    @password.setter
    def password(self, raw_password):
        self._password_hash = bcrypt.generate_password_hash(raw_password).decode('utf-8')
    
    def check_password(self, raw_password):
        return bcrypt.check_password_hash(self._password_hash, raw_password)
    
    @property
    def is_locked(self):
        if self.locked_until and self.locked_until > datetime.utcnow():
            return True
        return False
    
    def lock_account(self, minutes=30):
        self.locked_until = datetime.utcnow() + timedelta(minutes=minutes)
    
    def reset_failed_attempts(self):
        self.failed_login_count = 0
        self.locked_until = None
    
    @property
    def is_admin(self):
        return self.role == 'admin'
    
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'full_name': self.full_name,
            'role': self.role,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat(),
            'last_login': self.last_login.isoformat() if self.last_login else None
        }
    
    def __repr__(self):
        return f'<User {self.username}>'

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

# Forms
class RegisterForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[DataRequired(), Length(3, 20)])
    email = EmailField('อีเมล', validators=[DataRequired(), Email()])
    full_name = StringField('ชื่อ-นามสกุล', validators=[DataRequired()])
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(), Length(8, 128, message='ต้องมีอย่างน้อย 8 ตัวอักษร')
    ])
    confirm = PasswordField('ยืนยันรหัสผ่าน', validators=[
        DataRequired(), EqualTo('password', message='รหัสผ่านไม่ตรงกัน')
    ])
    submit = SubmitField('สมัครสมาชิก')
    
    def validate_username(self, field):
        if not re.match(r'^[a-zA-Z0-9_]+$', field.data):
            raise ValidationError('ใช้ได้เฉพาะตัวอักษร ตัวเลข และ _')
        if User.query.filter_by(username=field.data).first():
            raise ValidationError('ชื่อผู้ใช้นี้มีแล้ว')
    
    def validate_email(self, field):
        if User.query.filter_by(email=field.data.lower()).first():
            raise ValidationError('อีเมลนี้มีแล้ว')
    
    def validate_password(self, field):
        pw = field.data
        errors = []
        if not re.search(r'[A-Z]', pw): errors.append('ต้องมีตัวพิมพ์ใหญ่')
        if not re.search(r'[0-9]', pw): errors.append('ต้องมีตัวเลข')
        if errors:
            raise ValidationError('; '.join(errors))

class LoginForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[DataRequired()])
    password = PasswordField('รหัสผ่าน', validators=[DataRequired()])
    remember = BooleanField('จดจำฉัน')
    submit = SubmitField('เข้าสู่ระบบ')

# Decorators
def admin_required(f):
    @wraps(f)
    @login_required
    def decorated(*args, **kwargs):
        if not current_user.is_admin:
            flash('ต้องการสิทธิ์ admin', 'danger')
            return redirect(url_for('dashboard'))
        return f(*args, **kwargs)
    return decorated

# HTML Templates
STYLES = """
<style>
* { box-sizing: border-box; }
body { font-family: Arial, sans-serif; background: #f5f5f5; margin: 0; }
.container { max-width: 900px; margin: 0 auto; padding: 20px; }
nav { background: #2c3e50; padding: 15px 0; margin-bottom: 30px; }
nav .container { display: flex; justify-content: space-between; align-items: center; max-width: 900px; margin: 0 auto; padding: 0 20px; }
nav a { color: white; text-decoration: none; margin: 0 10px; }
.card { background: white; border-radius: 10px; padding: 30px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; }
.form-group { margin-bottom: 18px; }
label { display: block; font-weight: bold; margin-bottom: 5px; color: #555; }
input[type=text], input[type=email], input[type=password] {
    width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; font-size: 0.95em;
}
input:focus { outline: none; border-color: #3498db; box-shadow: 0 0 0 3px rgba(52,152,219,0.1); }
.is-invalid { border-color: #e74c3c !important; }
.error-msg { color: #e74c3c; font-size: 0.82em; margin-top: 3px; }
.btn { padding: 10px 20px; border: none; border-radius: 6px; cursor: pointer; font-size: 0.95em; }
.btn-primary { background: #3498db; color: white; }
.btn-danger { background: #e74c3c; color: white; }
.btn-success { background: #27ae60; color: white; }
.btn-block { width: 100%; }
.alert { padding: 12px; border-radius: 6px; margin-bottom: 15px; }
.alert-success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.alert-danger { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
.alert-warning { background: #fff3cd; color: #856404; border: 1px solid #ffc107; }
.alert-info { background: #d1ecf1; color: #0c5460; border: 1px solid #bee5eb; }
.check-group { display: flex; align-items: center; gap: 10px; }
.check-group input { width: auto; }
.text-center { text-align: center; }
.mt-3 { margin-top: 15px; }
table { width: 100%; border-collapse: collapse; }
th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
th { background: #f8f9fa; font-weight: bold; }
tr:hover { background: #f8f9fa; }
.badge { padding: 3px 8px; border-radius: 12px; font-size: 0.8em; }
.badge-admin { background: #dc3545; color: white; }
.badge-user { background: #6c757d; color: white; }
.badge-active { background: #28a745; color: white; }
.badge-inactive { background: #ffc107; color: #333; }
</style>
"""

BASE_HTML = STYLES + """
<nav>
<div class="container">
    <strong style="color:white">🔐 Auth System</strong>
    <div>
    {% if current_user.is_authenticated %}
        <a href="/dashboard">Dashboard</a>
        {% if current_user.is_admin %}<a href="/admin">Admin</a>{% endif %}
        <a href="/profile">{{ current_user.username }}</a>
        <a href="/logout">ออกจากระบบ</a>
    {% else %}
        <a href="/login">เข้าสู่ระบบ</a>
        <a href="/register">สมัครสมาชิก</a>
    {% endif %}
    </div>
</div>
</nav>
<div class="container">
{% with messages = get_flashed_messages(with_categories=true) %}
{% if messages %}{% for category, message in messages %}
<div class="alert alert-{{ category }}">{{ message }}</div>
{% endfor %}{% endif %}
{% endwith %}
{% block content %}{% endblock %}
</div>
"""

# Routes
@app.route('/')
def index():
    html = BASE_HTML.replace("{% block content %}{% endblock %}", """
    <div class="card text-center">
        <h1>🔐 Authentication System Demo</h1>
        <p>ตัวอย่างระบบ Authentication ด้วย Flask-Login</p>
        {% if current_user.is_authenticated %}
        <p><a href="/dashboard" class="btn btn-primary">ไปยัง Dashboard</a></p>
        {% else %}
        <p>
            <a href="/register" class="btn btn-primary" style="margin-right:10px">สมัครสมาชิก</a>
            <a href="/login" class="btn btn-success">เข้าสู่ระบบ</a>
        </p>
        {% endif %}
    </div>
    """)
    return render_template_string(html)

@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('dashboard'))
    
    form = RegisterForm()
    
    if form.validate_on_submit():
        user = User(
            username=form.username.data,
            email=form.email.data.lower(),
            full_name=form.full_name.data
        )
        user.password = form.password.data
        db.session.add(user)
        db.session.commit()
        
        flash(f'สมัครสมาชิกสำเร็จ! ยินดีต้อนรับ {user.full_name}', 'success')
        return redirect(url_for('login'))
    
    html = BASE_HTML.replace("{% block content %}{% endblock %}", """
    <div class="card" style="max-width:500px;margin:0 auto">
        <h2 class="text-center">สมัครสมาชิก</h2>
        <form method="POST">
            {{ form.hidden_tag() }}
            {% for field in [form.username, form.email, form.full_name, form.password, form.confirm] %}
            <div class="form-group">
                {{ field.label }}
                {{ field(class="form-control" + (" is-invalid" if field.errors else ""), style="width:100%;padding:10px;border:1px solid;border-radius:6px;border-color:" + ("#e74c3c" if field.errors else "#ddd")) }}
                {% for err in field.errors %}<div class="error-msg">{{ err }}</div>{% endfor %}
            </div>
            {% endfor %}
            {{ form.submit(class="btn btn-primary btn-block", style="margin-top:10px") }}
        </form>
        <div class="mt-3 text-center">
            <small>มีบัญชีแล้ว? <a href="/login">เข้าสู่ระบบ</a></small>
        </div>
    </div>
    """)
    return render_template_string(html, form=form)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('dashboard'))
    
    form = LoginForm()
    
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        
        if not user:
            flash('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง', 'danger')
        elif user.is_locked:
            flash(f'บัญชีถูกล็อคชั่วคราว กรุณาลองใหม่ใน {(user.locked_until - datetime.utcnow()).seconds // 60} นาที', 'danger')
        elif not user.is_active:
            flash('บัญชีถูกระงับการใช้งาน', 'danger')
        elif not user.check_password(form.password.data):
            user.failed_login_count += 1
            if user.failed_login_count >= 5:
                user.lock_account(30)
                flash('พยายาม login ผิดหลายครั้ง บัญชีถูกล็อค 30 นาที', 'danger')
            else:
                remaining = 5 - user.failed_login_count
                flash(f'รหัสผ่านไม่ถูกต้อง (เหลือ {remaining} ครั้ง)', 'danger')
            db.session.commit()
        else:
            user.reset_failed_attempts()
            user.last_login = datetime.utcnow()
            db.session.commit()
            
            login_user(user, remember=form.remember.data)
            flash(f'ยินดีต้อนรับ {user.full_name or user.username}!', 'success')
            
            next_page = request.args.get('next')
            if not next_page or not next_page.startswith('/'):
                next_page = url_for('dashboard')
            return redirect(next_page)
    
    html = BASE_HTML.replace("{% block content %}{% endblock %}", """
    <div class="card" style="max-width:400px;margin:0 auto">
        <h2 class="text-center">เข้าสู่ระบบ</h2>
        <form method="POST">
            {{ form.hidden_tag() }}
            {% for field in [form.username, form.password] %}
            <div class="form-group">
                {{ field.label }}
                {{ field(style="width:100%;padding:10px;border:1px solid #ddd;border-radius:6px") }}
                {% for err in field.errors %}<div class="error-msg">{{ err }}</div>{% endfor %}
            </div>
            {% endfor %}
            <div class="check-group">
                {{ form.remember() }}
                {{ form.remember.label }}
            </div>
            {{ form.submit(class="btn btn-primary btn-block", style="margin-top:15px") }}
        </form>
        <div class="mt-3 text-center">
            <small>ยังไม่มีบัญชี? <a href="/register">สมัครสมาชิก</a></small>
        </div>
    </div>
    """)
    return render_template_string(html, form=form)

@app.route('/logout')
@login_required
def logout():
    username = current_user.username
    logout_user()
    flash(f'ออกจากระบบแล้ว ({username})', 'info')
    return redirect(url_for('login'))

@app.route('/dashboard')
@login_required
def dashboard():
    html = BASE_HTML.replace("{% block content %}{% endblock %}", """
    <h2>Dashboard</h2>
    <div class="card">
        <h3>ยินดีต้อนรับ {{ current_user.full_name or current_user.username }}!</h3>
        <table>
            <tr><td><strong>Username:</strong></td><td>{{ current_user.username }}</td></tr>
            <tr><td><strong>Email:</strong></td><td>{{ current_user.email }}</td></tr>
            <tr><td><strong>Role:</strong></td><td>
                <span class="badge badge-{{ current_user.role }}">{{ current_user.role }}</span>
            </td></tr>
            <tr><td><strong>สถานะ:</strong></td><td>
                <span class="badge badge-{{ 'active' if current_user.is_active else 'inactive' }}">
                    {{ 'ใช้งาน' if current_user.is_active else 'ระงับ' }}
                </span>
            </td></tr>
            <tr><td><strong>Login ล่าสุด:</strong></td><td>
                {{ current_user.last_login.strftime('%d/%m/%Y %H:%M') if current_user.last_login else 'ไม่มีข้อมูล' }}
            </td></tr>
        </table>
    </div>
    """)
    return render_template_string(html)

@app.route('/admin')
@admin_required
def admin_panel():
    users = User.query.order_by(User.created_at.desc()).all()
    
    html = BASE_HTML.replace("{% block content %}{% endblock %}", """
    <h2>Admin Panel</h2>
    <div class="card">
        <h3>ผู้ใช้ทั้งหมด ({{ users|length }} คน)</h3>
        <table>
            <thead>
                <tr><th>ID</th><th>Username</th><th>Email</th><th>Role</th><th>สถานะ</th><th>สร้างเมื่อ</th></tr>
            </thead>
            <tbody>
            {% for user in users %}
            <tr>
                <td>{{ user.id }}</td>
                <td>{{ user.username }}</td>
                <td>{{ user.email }}</td>
                <td><span class="badge badge-{{ user.role }}">{{ user.role }}</span></td>
                <td><span class="badge badge-{{ 'active' if user.is_active else 'inactive' }}">{{ 'ใช้งาน' if user.is_active else 'ระงับ' }}</span></td>
                <td>{{ user.created_at.strftime('%d/%m/%Y') }}</td>
            </tr>
            {% endfor %}
            </tbody>
        </table>
    </div>
    """)
    return render_template_string(html, users=users)

# Initialize database
with app.app_context():
    db.create_all()
    
    if User.query.count() == 0:
        admin = User(username='admin', email='admin@example.com', full_name='Admin User', role='admin')
        admin.password = 'Admin1234!'
        
        user1 = User(username='somchai', email='somchai@example.com', full_name='สมชาย ใจดี')
        user1.password = 'User1234!'
        
        db.session.add_all([admin, user1])
        db.session.commit()
        print('สร้าง demo users แล้ว: admin/Admin1234!, somchai/User1234!')

if __name__ == '__main__':
    print('\n=== Auth System Demo ===')
    print('URL: http://localhost:5000')
    print('Admin: admin / Admin1234!')
    print('User: somchai / User1234!\n')
    app.run(debug=True)
```

---

## 13. แบบฝึกหัด

### ข้อที่ 1: JWT Authentication API

**เฉลย:**

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_jwt_extended import (JWTManager, create_access_token, create_refresh_token,
                                  jwt_required, get_jwt_identity, get_jwt)
from datetime import timedelta, datetime

app = Flask(__name__)
app.config['SECRET_KEY'] = 'jwt-api-secret'
app.config['JWT_SECRET_KEY'] = 'jwt-token-secret'
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)
app.config['JWT_REFRESH_TOKEN_EXPIRES'] = timedelta(days=30)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///jwt_api.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
jwt = JWTManager(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    role = db.Column(db.String(20), default='user')
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    def set_password(self, password):
        self.password_hash = bcrypt.generate_password_hash(password).decode('utf-8')
    
    def check_password(self, password):
        return bcrypt.check_password_hash(self.password_hash, password)
    
    def to_dict(self):
        return {'id': self.id, 'username': self.username,
                'email': self.email, 'role': self.role}

with app.app_context():
    db.create_all()
    if User.query.count() == 0:
        admin = User(username='admin', email='admin@example.com', role='admin')
        admin.set_password('Admin1234!')
        db.session.add(admin)
        db.session.commit()

@app.route('/api/auth/register', methods=['POST'])
def register():
    data = request.get_json() or {}
    if not all(k in data for k in ['username', 'email', 'password']):
        return jsonify({'error': 'ต้องระบุ username, email, password'}), 400
    
    if User.query.filter_by(username=data['username']).first():
        return jsonify({'error': 'Username มีแล้ว'}), 409
    
    user = User(username=data['username'], email=data['email'])
    user.set_password(data['password'])
    db.session.add(user)
    db.session.commit()
    
    return jsonify({'message': 'สมัครสำเร็จ', 'user': user.to_dict()}), 201

@app.route('/api/auth/login', methods=['POST'])
def login():
    data = request.get_json() or {}
    user = User.query.filter_by(username=data.get('username')).first()
    
    if not user or not user.check_password(data.get('password', '')):
        return jsonify({'error': 'Credentials ไม่ถูกต้อง'}), 401
    
    access_token = create_access_token(
        identity=user.id,
        additional_claims={'username': user.username, 'role': user.role}
    )
    refresh_token = create_refresh_token(identity=user.id)
    
    return jsonify({
        'access_token': access_token,
        'refresh_token': refresh_token,
        'user': user.to_dict()
    })

@app.route('/api/auth/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh():
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    if not user:
        return jsonify({'error': 'User ไม่พบ'}), 404
    
    access_token = create_access_token(
        identity=user_id,
        additional_claims={'username': user.username, 'role': user.role}
    )
    return jsonify({'access_token': access_token})

@app.route('/api/me')
@jwt_required()
def get_me():
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    return jsonify(user.to_dict())

@app.route('/api/admin/users')
@jwt_required()
def admin_users():
    claims = get_jwt()
    if claims.get('role') != 'admin':
        return jsonify({'error': 'Admin only'}), 403
    
    users = User.query.all()
    return jsonify({'users': [u.to_dict() for u in users]})

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 2-8 (สรุป)

```python
# ข้อ 2: Role-based system พร้อม 3 roles (user, moderator, admin)
# ข้อ 3: Password Reset ด้วย itsdangerous tokens
# ข้อ 4: Remember Me + session expiry
# ข้อ 5: Account lockout หลัง login ผิดหลายครั้ง
# ข้อ 6: Email verification flow
# ข้อ 7: OAuth login ด้วย Google
# ข้อ 8: Complete auth API พร้อม JWT blacklist

# ตัวอย่าง Account Lockout
class User(db.Model):
    MAX_FAILED_ATTEMPTS = 5
    LOCKOUT_DURATION = 30  # minutes
    
    failed_login_count = db.Column(db.Integer, default=0)
    locked_until = db.Column(db.DateTime)
    
    @property
    def is_locked(self):
        return self.locked_until and self.locked_until > datetime.utcnow()
    
    def record_failed_login(self):
        self.failed_login_count += 1
        if self.failed_login_count >= self.MAX_FAILED_ATTEMPTS:
            self.locked_until = datetime.utcnow() + timedelta(minutes=self.LOCKOUT_DURATION)
        db.session.commit()
    
    def record_successful_login(self):
        self.failed_login_count = 0
        self.locked_until = None
        self.last_login = datetime.utcnow()
        db.session.commit()
```

---

## สรุป

ใน Part 55 นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|---------------|
| Session | Flask session, server-side sessions |
| Flask-Login | Setup, user_loader, current_user |
| UserMixin | is_authenticated, is_active, get_id() |
| Login/Logout | login_user(), logout_user(), flow |
| @login_required | Decorator, custom decorators |
| Password Hashing | werkzeug, bcrypt, password validation |
| Remember Me | Cookie lifetime, fresh_login_required |
| RBAC | Roles, permissions, decorators |
| JWT | Flask-JWT-Extended, tokens, refresh, blacklist |
| OAuth2 | Google, GitHub OAuth flow |
| Security | Rate limiting, headers, logging |

### ขั้นตอนต่อไป

ใน Part 56 เราจะเรียนรู้เรื่อง:
- Flask REST API ขั้นสูง
- Flask-RESTful
- API versioning
- API documentation (Swagger)
- Testing Flask applications

---

*เขียนโดยหลักสูตร Python Advanced - Part 55*
