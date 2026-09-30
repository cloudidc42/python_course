# Part 53: Flask - Forms & Input Validation

## สารบัญ
1. [HTML Forms Basics](#1-html-forms-basics)
2. [Handling Form Data ใน Flask](#2-handling-form-data-ใน-flask)
3. [WTForms Library](#3-wtforms-library)
4. [Flask-WTF Extension](#4-flask-wtf-extension)
5. [Form Field Types](#5-form-field-types)
6. [Built-in Validators](#6-built-in-validators)
7. [Custom Validators](#7-custom-validators)
8. [CSRF Protection](#8-csrf-protection)
9. [File Uploads](#9-file-uploads)
10. [Form Rendering ใน Jinja2](#10-form-rendering-ใน-jinja2)
11. [Form Validation Messages](#11-form-validation-messages)
12. [ตัวอย่างโปรแกรมจริง](#12-ตัวอย่างโปรแกรมจริง)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. HTML Forms Basics

### HTML Form คืออะไร

HTML Form คือส่วนของ HTML ที่ใช้รับข้อมูลจาก user แล้วส่งไปยัง server

```html
<!-- Form พื้นฐาน -->
<form action="/submit" method="POST">
    <!-- Text input -->
    <label for="name">ชื่อ:</label>
    <input type="text" id="name" name="name" required>
    
    <!-- Email input -->
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <!-- Password input -->
    <label for="password">รหัสผ่าน:</label>
    <input type="password" id="password" name="password">
    
    <!-- Number input -->
    <label for="age">อายุ:</label>
    <input type="number" id="age" name="age" min="0" max="150">
    
    <!-- Textarea -->
    <label for="message">ข้อความ:</label>
    <textarea id="message" name="message" rows="4" cols="50"></textarea>
    
    <!-- Select -->
    <label for="city">เมือง:</label>
    <select id="city" name="city">
        <option value="">เลือกเมือง</option>
        <option value="bkk">กรุงเทพฯ</option>
        <option value="cm">เชียงใหม่</option>
        <option value="kkn">ขอนแก่น</option>
    </select>
    
    <!-- Radio buttons -->
    <p>เพศ:</p>
    <input type="radio" id="male" name="gender" value="male">
    <label for="male">ชาย</label>
    <input type="radio" id="female" name="gender" value="female">
    <label for="female">หญิง</label>
    
    <!-- Checkboxes -->
    <p>ความสนใจ:</p>
    <input type="checkbox" id="python" name="interests" value="python">
    <label for="python">Python</label>
    <input type="checkbox" id="flask" name="interests" value="flask">
    <label for="flask">Flask</label>
    
    <!-- File upload -->
    <label for="photo">รูปภาพ:</label>
    <input type="file" id="photo" name="photo" accept="image/*">
    
    <!-- Hidden field -->
    <input type="hidden" name="csrf_token" value="abc123">
    
    <!-- Submit button -->
    <button type="submit">ส่งข้อมูล</button>
    <button type="reset">ล้างข้อมูล</button>
</form>
```

### Form Encoding Types

```html
<!-- application/x-www-form-urlencoded (default) -->
<!-- ใช้สำหรับข้อมูลทั่วไป -->
<form method="POST" enctype="application/x-www-form-urlencoded">
    ...
</form>

<!-- multipart/form-data -->
<!-- ต้องใช้เมื่อมีการ upload files -->
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="photo">
    ...
</form>

<!-- text/plain -->
<!-- ไม่ค่อยใช้ -->
<form method="POST" enctype="text/plain">
    ...
</form>
```

### Form Validation ฝั่ง HTML5

```html
<!-- HTML5 Built-in Validation -->
<form>
    <!-- required - ต้องกรอก -->
    <input type="text" name="name" required>
    
    <!-- minlength/maxlength - ความยาวขั้นต่ำ/สูงสุด -->
    <input type="text" name="username" minlength="3" maxlength="20">
    
    <!-- pattern - regex pattern -->
    <input type="text" name="phone" pattern="[0-9]{10}" 
           title="กรอกเบอร์โทร 10 หลัก">
    
    <!-- min/max - สำหรับ number -->
    <input type="number" name="age" min="18" max="100">
    
    <!-- type validation -->
    <input type="email" name="email">    <!-- validate email format -->
    <input type="url" name="website">    <!-- validate URL format -->
    <input type="tel" name="phone">      <!-- telephone field -->
</form>
```

---

## 2. Handling Form Data ใน Flask

### ดึงข้อมูลจาก Form

```python
from flask import Flask, request, redirect, url_for, render_template_string

app = Flask(__name__)

@app.route('/form', methods=['GET', 'POST'])
def handle_form():
    if request.method == 'POST':
        # ดึงข้อมูลจาก form fields
        name = request.form.get('name')          # None ถ้าไม่มี
        name = request.form['name']               # KeyError ถ้าไม่มี
        
        # ดึงพร้อม default value
        age = request.form.get('age', 0, type=int)
        
        # ดึงหลายค่า (checkbox)
        interests = request.form.getlist('interests')
        
        # ดึงทั้งหมดเป็น dict
        all_data = request.form.to_dict()
        
        # ดึงทั้งหมดพร้อมหลายค่า
        all_data_multi = request.form.to_dict(flat=False)
        
        print(f'Name: {name}')
        print(f'Age: {age}')
        print(f'Interests: {interests}')
        
        return redirect(url_for('success'))
    
    return render_template_string('''
        <form method="POST">
            <input name="name" type="text" placeholder="ชื่อ" required><br>
            <input name="age" type="number" placeholder="อายุ"><br>
            <input name="interests" type="checkbox" value="python"> Python
            <input name="interests" type="checkbox" value="flask"> Flask<br>
            <button type="submit">ส่ง</button>
        </form>
    ''')

@app.route('/success')
def success():
    return 'ส่งข้อมูลสำเร็จ!'
```

### Form Validation แบบ Manual

```python
from flask import Flask, request, render_template_string, flash, redirect, url_for
import re

app = Flask(__name__)
app.secret_key = 'secret'

def validate_registration(data):
    """Validate registration form data"""
    errors = {}
    
    # Validate username
    username = data.get('username', '').strip()
    if not username:
        errors['username'] = 'ต้องระบุชื่อผู้ใช้'
    elif len(username) < 3:
        errors['username'] = 'ชื่อผู้ใช้ต้องมีอย่างน้อย 3 ตัวอักษร'
    elif len(username) > 20:
        errors['username'] = 'ชื่อผู้ใช้ต้องไม่เกิน 20 ตัวอักษร'
    elif not re.match(r'^[a-zA-Z0-9_]+$', username):
        errors['username'] = 'ชื่อผู้ใช้ต้องเป็นตัวอักษร ตัวเลข หรือ _ เท่านั้น'
    
    # Validate email
    email = data.get('email', '').strip()
    if not email:
        errors['email'] = 'ต้องระบุ email'
    elif not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
        errors['email'] = 'รูปแบบ email ไม่ถูกต้อง'
    
    # Validate password
    password = data.get('password', '')
    if not password:
        errors['password'] = 'ต้องระบุรหัสผ่าน'
    elif len(password) < 8:
        errors['password'] = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร'
    elif not re.search(r'[A-Z]', password):
        errors['password'] = 'รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว'
    elif not re.search(r'[0-9]', password):
        errors['password'] = 'รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว'
    
    # Validate confirm password
    confirm = data.get('confirm_password', '')
    if password and password != confirm:
        errors['confirm_password'] = 'รหัสผ่านไม่ตรงกัน'
    
    return errors

@app.route('/register', methods=['GET', 'POST'])
def register():
    errors = {}
    form_data = {}
    
    if request.method == 'POST':
        form_data = request.form.to_dict()
        errors = validate_registration(form_data)
        
        if not errors:
            # บันทึกข้อมูล
            flash('สมัครสมาชิกสำเร็จ!', 'success')
            return redirect(url_for('register'))
    
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head><meta charset="UTF-8"><title>Register</title>
    <style>
        body { font-family: Arial; max-width: 500px; margin: 40px auto; padding: 20px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
        .error { color: red; font-size: 0.85em; margin-top: 3px; }
        .field-error { border-color: red !important; }
        .alert { padding: 10px; background: #d4edda; color: #155724; border-radius: 4px; margin-bottom: 15px; }
        button { background: #007bff; color: white; padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; }
    </style>
    </head>
    <body>
        <h1>สมัครสมาชิก</h1>
        
        {% with messages = get_flashed_messages(with_categories=true) %}
        {% for category, message in messages %}
        <div class="alert">{{ message }}</div>
        {% endfor %}
        {% endwith %}
        
        <form method="POST">
            <div class="form-group">
                <label>ชื่อผู้ใช้:</label>
                <input type="text" name="username" 
                       value="{{ form_data.get('username', '') }}"
                       class="{{ 'field-error' if errors.get('username') else '' }}">
                {% if errors.get('username') %}
                <div class="error">{{ errors.username }}</div>
                {% endif %}
            </div>
            
            <div class="form-group">
                <label>Email:</label>
                <input type="email" name="email"
                       value="{{ form_data.get('email', '') }}"
                       class="{{ 'field-error' if errors.get('email') else '' }}">
                {% if errors.get('email') %}
                <div class="error">{{ errors.email }}</div>
                {% endif %}
            </div>
            
            <div class="form-group">
                <label>รหัสผ่าน:</label>
                <input type="password" name="password"
                       class="{{ 'field-error' if errors.get('password') else '' }}">
                {% if errors.get('password') %}
                <div class="error">{{ errors.password }}</div>
                {% endif %}
            </div>
            
            <div class="form-group">
                <label>ยืนยันรหัสผ่าน:</label>
                <input type="password" name="confirm_password"
                       class="{{ 'field-error' if errors.get('confirm_password') else '' }}">
                {% if errors.get('confirm_password') %}
                <div class="error">{{ errors.confirm_password }}</div>
                {% endif %}
            </div>
            
            <button type="submit">สมัครสมาชิก</button>
        </form>
    </body>
    </html>
    ''', errors=errors, form_data=form_data)
```

---

## 3. WTForms Library

### WTForms คืออะไร

WTForms เป็น Python library สำหรับการสร้างและ validate forms มีข้อดี:
1. **Type-safe**: กำหนด type ของ field ชัดเจน
2. **Reusable**: สร้าง form class ใช้ซ้ำได้
3. **Validation**: มี validators พร้อมใช้
4. **Rendering**: render HTML ได้อัตโนมัติ

```bash
# ติดตั้ง WTForms
pip install WTForms
pip install Flask-WTF  # Flask integration
```

### การสร้าง Form Class ด้วย WTForms

```python
from wtforms import Form, StringField, PasswordField, EmailField, IntegerField
from wtforms import TextAreaField, BooleanField, SelectField, RadioField
from wtforms import MultipleFileField, FileField
from wtforms.validators import DataRequired, Email, Length, EqualTo, NumberRange

class RegistrationForm(Form):
    """Form สำหรับสมัครสมาชิก"""
    
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(message='ต้องระบุชื่อผู้ใช้'),
        Length(min=3, max=20, message='ต้องมี 3-20 ตัวอักษร')
    ])
    
    email = EmailField('Email', validators=[
        DataRequired(message='ต้องระบุ email'),
        Email(message='รูปแบบ email ไม่ถูกต้อง')
    ])
    
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(message='ต้องระบุรหัสผ่าน'),
        Length(min=8, message='รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    ])
    
    confirm_password = PasswordField('ยืนยันรหัสผ่าน', validators=[
        DataRequired(),
        EqualTo('password', message='รหัสผ่านไม่ตรงกัน')
    ])
    
    age = IntegerField('อายุ', validators=[
        NumberRange(min=18, max=100, message='ต้องมีอายุ 18-100 ปี')
    ])
    
    bio = TextAreaField('ประวัติย่อ', validators=[
        Length(max=500, message='ประวัติไม่เกิน 500 ตัวอักษร')
    ])
    
    gender = SelectField('เพศ', choices=[
        ('', 'เลือกเพศ'),
        ('male', 'ชาย'),
        ('female', 'หญิง'),
        ('other', 'อื่นๆ')
    ])
    
    agree_terms = BooleanField('ยอมรับเงื่อนไข', validators=[
        DataRequired(message='ต้องยอมรับเงื่อนไข')
    ])
```

### การใช้งาน WTForms Form

```python
from flask import Flask, render_template_string, request
from wtforms import Form, StringField, validators

app = Flask(__name__)

class LoginForm(Form):
    username = StringField('ชื่อผู้ใช้', [validators.DataRequired()])
    password = StringField('รหัสผ่าน', [validators.DataRequired()])

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm(request.form)
    
    if request.method == 'POST' and form.validate():
        # Form data ผ่าน validation แล้ว
        username = form.username.data
        password = form.password.data
        
        return f'Login: {username}'
    
    return render_template_string('''
    <form method="POST">
        <p>
            <label>{{ form.username.label }}</label>
            {{ form.username() }}
            {% for error in form.username.errors %}
            <span style="color:red">{{ error }}</span>
            {% endfor %}
        </p>
        <p>
            <label>{{ form.password.label }}</label>
            {{ form.password(type="password") }}
            {% for error in form.password.errors %}
            <span style="color:red">{{ error }}</span>
            {% endfor %}
        </p>
        <button type="submit">เข้าสู่ระบบ</button>
    </form>
    ''', form=form)
```

---

## 4. Flask-WTF Extension

### Flask-WTF คืออะไร

Flask-WTF เป็น extension ที่รวม WTForms เข้ากับ Flask มีคุณสมบัติเพิ่มเติม:
1. **CSRF Protection**: ป้องกัน Cross-Site Request Forgery อัตโนมัติ
2. **File Uploads**: จัดการ file uploads ง่ายขึ้น
3. **reCAPTCHA**: รองรับ Google reCAPTCHA
4. **Bootstrap**: รองรับ Bootstrap forms

```bash
pip install Flask-WTF
```

### การ Setup Flask-WTF

```python
from flask import Flask
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'  # จำเป็นสำหรับ CSRF
app.config['WTF_CSRF_ENABLED'] = True

csrf = CSRFProtect(app)
```

### FlaskForm (Flask-WTF Form Class)

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, EmailField, SubmitField
from wtforms import SelectField, TextAreaField, BooleanField, IntegerField
from wtforms.validators import DataRequired, Email, Length, EqualTo, Optional

class RegisterForm(FlaskForm):
    """Form สมัครสมาชิกด้วย Flask-WTF"""
    
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(message='กรุณาระบุชื่อผู้ใช้'),
        Length(min=3, max=20)
    ], render_kw={'placeholder': 'กรอกชื่อผู้ใช้...'})
    
    email = EmailField('อีเมล', validators=[
        DataRequired(),
        Email(message='รูปแบบอีเมลไม่ถูกต้อง')
    ])
    
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(),
        Length(min=8, message='รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    ])
    
    confirm = PasswordField('ยืนยันรหัสผ่าน', validators=[
        DataRequired(),
        EqualTo('password', message='รหัสผ่านไม่ตรงกัน')
    ])
    
    role = SelectField('บทบาท', choices=[
        ('user', 'ผู้ใช้ทั่วไป'),
        ('moderator', 'ผู้ดูแล'),
        ('admin', 'ผู้ดูแลระบบ')
    ])
    
    bio = TextAreaField('ประวัติ', validators=[Optional(), Length(max=500)])
    
    agree = BooleanField('ยอมรับข้อกำหนด', validators=[DataRequired()])
    
    submit = SubmitField('สมัครสมาชิก')
```

### ใช้งาน FlaskForm

```python
from flask import Flask, render_template_string, redirect, url_for, flash
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired, Length

app = Flask(__name__)
app.config['SECRET_KEY'] = 'dev-secret-key'

class LoginForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[DataRequired()])
    password = PasswordField('รหัสผ่าน', validators=[DataRequired()])
    submit = SubmitField('เข้าสู่ระบบ')

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()  # ไม่ต้องส่ง request.form เพราะ Flask-WTF ทำให้อัตโนมัติ
    
    if form.validate_on_submit():  # ตรวจสอบ POST + validation + CSRF
        username = form.username.data
        password = form.password.data
        
        # ตรวจสอบ credentials (ตัวอย่าง)
        if username == 'admin' and password == 'secret':
            flash(f'ยินดีต้อนรับ {username}!', 'success')
            return redirect(url_for('dashboard'))
        else:
            flash('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง', 'error')
    
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head><meta charset="UTF-8"><title>Login</title>
    <style>
        body { font-family: Arial; max-width: 400px; margin: 60px auto; padding: 20px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; font-weight: bold; margin-bottom: 5px; }
        input { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
        .error { color: red; font-size: 0.85em; }
        .alert { padding: 10px; border-radius: 4px; margin-bottom: 15px; }
        .alert-success { background: #d4edda; color: #155724; }
        .alert-error { background: #f8d7da; color: #721c24; }
        button { width: 100%; padding: 10px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
    </head>
    <body>
        <h2>เข้าสู่ระบบ</h2>
        
        {% with messages = get_flashed_messages(with_categories=true) %}
        {% for category, message in messages %}
        <div class="alert alert-{{ category }}">{{ message }}</div>
        {% endfor %}
        {% endwith %}
        
        <form method="POST">
            {{ form.hidden_tag() }}  {# CSRF token #}
            
            <div class="form-group">
                {{ form.username.label }}
                {{ form.username(class="form-control") }}
                {% for error in form.username.errors %}
                <div class="error">{{ error }}</div>
                {% endfor %}
            </div>
            
            <div class="form-group">
                {{ form.password.label }}
                {{ form.password(class="form-control") }}
                {% for error in form.password.errors %}
                <div class="error">{{ error }}</div>
                {% endfor %}
            </div>
            
            {{ form.submit(class="btn-submit") }}
        </form>
    </body>
    </html>
    ''', form=form)
```

---

## 5. Form Field Types

### String Fields

```python
from flask_wtf import FlaskForm
from wtforms import (
    StringField,      # text input
    PasswordField,    # password input
    EmailField,       # email input
    TextAreaField,    # textarea
    SearchField,      # search input
    TelField,         # tel input
    URLField,         # url input
    ColorField,       # color picker
)
from wtforms.validators import DataRequired, Email, URL

class StringFieldsDemo(FlaskForm):
    # Basic text input
    name = StringField('ชื่อ')
    
    # Password input (value ไม่แสดง)
    password = PasswordField('รหัสผ่าน')
    
    # Email input (มี built-in HTML5 validation)
    email = EmailField('อีเมล', validators=[Email()])
    
    # Textarea
    bio = TextAreaField('ประวัติ', render_kw={'rows': 5, 'cols': 40})
    
    # Search input
    query = SearchField('ค้นหา')
    
    # URL input
    website = URLField('เว็บไซต์', validators=[URL()])
    
    # Phone input
    phone = TelField('เบอร์โทร')
```

### Numeric Fields

```python
from wtforms import IntegerField, DecimalField, FloatField, IntegerRangeField, DecimalRangeField
from wtforms.validators import NumberRange

class NumericFieldsDemo(FlaskForm):
    # Integer
    age = IntegerField('อายุ', validators=[NumberRange(min=0, max=150)])
    
    # Decimal (ใช้ Decimal type)
    price = DecimalField('ราคา', places=2)  # 2 decimal places
    
    # Float
    weight = FloatField('น้ำหนัก')
    
    # Range slider (int)
    rating = IntegerRangeField('คะแนน', render_kw={'min': 1, 'max': 10})
    
    # Range slider (decimal)
    percentage = DecimalRangeField('เปอร์เซ็นต์', render_kw={'min': 0, 'max': 100, 'step': 0.5})
```

### Date/Time Fields

```python
from wtforms import DateField, TimeField, DateTimeField, DateTimeLocalField, MonthField, WeekField

class DateTimeFieldsDemo(FlaskForm):
    # Date: YYYY-MM-DD
    birth_date = DateField('วันเกิด')
    
    # Time: HH:MM
    meeting_time = TimeField('เวลาประชุม')
    
    # DateTime: YYYY-MM-DD HH:MM:SS
    event_datetime = DateTimeField('วันเวลาอีเวนท์', format='%Y-%m-%d %H:%M')
    
    # DateTime Local (HTML5)
    local_datetime = DateTimeLocalField('วันเวลาท้องถิ่น')
    
    # Month: YYYY-MM
    birth_month = MonthField('เดือนเกิด')
```

### Choice Fields

```python
from wtforms import SelectField, SelectMultipleField, RadioField
from wtforms.widgets import ListWidget, CheckboxInput

class ChoiceFieldsDemo(FlaskForm):
    # Single select
    city = SelectField('เมือง', choices=[
        ('', 'เลือกเมือง'),
        ('bkk', 'กรุงเทพฯ'),
        ('cm', 'เชียงใหม่'),
        ('kkn', 'ขอนแก่น'),
    ])
    
    # Select with coerce (แปลง type)
    category_id = SelectField('หมวดหมู่', 
                               choices=[(1, 'Python'), (2, 'Flask'), (3, 'Database')],
                               coerce=int)  # แปลง value เป็น int
    
    # Multiple select
    skills = SelectMultipleField('ทักษะ', choices=[
        ('python', 'Python'),
        ('javascript', 'JavaScript'),
        ('sql', 'SQL'),
    ])
    
    # Radio buttons
    gender = RadioField('เพศ', choices=[
        ('male', 'ชาย'),
        ('female', 'หญิง'),
        ('other', 'อื่นๆ')
    ])
    
    # Checkboxes (SelectMultipleField + CheckboxInput)
    interests = SelectMultipleField('ความสนใจ',
        choices=[
            ('coding', 'เขียนโปรแกรม'),
            ('reading', 'อ่านหนังสือ'),
            ('gaming', 'เล่นเกม'),
        ],
        widget=ListWidget(prefix_label=False),
        option_widget=CheckboxInput()
    )
```

### File Fields

```python
from flask_wtf.file import FileField, FileRequired, FileAllowed, MultipleFileField

class FileFieldsDemo(FlaskForm):
    # Single file
    photo = FileField('รูปภาพ', validators=[
        FileRequired(message='กรุณาเลือกไฟล์'),
        FileAllowed(['jpg', 'jpeg', 'png', 'gif'], 'เฉพาะไฟล์รูปภาพเท่านั้น!')
    ])
    
    # Multiple files
    documents = MultipleFileField('เอกสาร', validators=[
        FileAllowed(['pdf', 'doc', 'docx'], 'เฉพาะ PDF และ Word เท่านั้น!')
    ])
```

### Boolean และ Hidden Fields

```python
from wtforms import BooleanField, HiddenField, SubmitField
from wtforms.validators import DataRequired

class MiscFieldsDemo(FlaskForm):
    # Checkbox
    agree_terms = BooleanField('ยอมรับข้อกำหนด', validators=[
        DataRequired(message='ต้องยอมรับข้อกำหนด')
    ])
    
    newsletter = BooleanField('รับข่าวสาร', default=True)
    
    # Hidden field
    user_id = HiddenField('User ID')
    next_url = HiddenField('Next URL')
    
    # Submit button
    submit = SubmitField('บันทึก')
    cancel = SubmitField('ยกเลิก')  # multiple submit buttons
```

---

## 6. Built-in Validators

### Validators ที่ใช้บ่อย

```python
from wtforms.validators import (
    DataRequired,     # ต้องกรอก (ไม่ยอมรับ whitespace เพียงอย่างเดียว)
    InputRequired,    # ต้องกรอก (ยอมรับ whitespace)
    Optional,         # ไม่บังคับกรอก
    Length,           # ความยาวขั้นต่ำ/สูงสุด
    NumberRange,      # ช่วงตัวเลข
    Email,            # รูปแบบ email
    URL,              # รูปแบบ URL
    IPAddress,        # IP address
    MacAddress,       # MAC address
    UUID,             # UUID format
    EqualTo,          # เปรียบเทียบกับ field อื่น
    NoneOf,           # ต้องไม่เป็นค่าใดๆ ใน list
    AnyOf,            # ต้องเป็นค่าใดค่าหนึ่งใน list
    Regexp,           # Regular expression
    ValidationError,  # raise error
)
```

### ตัวอย่างการใช้ Validators

```python
from flask_wtf import FlaskForm
from wtforms import StringField, IntegerField, EmailField, URLField
from wtforms.validators import (
    DataRequired, Optional, Length, NumberRange,
    Email, URL, EqualTo, Regexp, NoneOf, AnyOf
)

class UserProfileForm(FlaskForm):
    
    # DataRequired - ต้องกรอก ไม่ยอมรับ whitespace
    first_name = StringField('ชื่อ', validators=[
        DataRequired(message='กรุณากรอกชื่อ')
    ])
    
    # Optional - ไม่บังคับ
    middle_name = StringField('ชื่อกลาง', validators=[Optional()])
    
    # Length - จำกัดความยาว
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(),
        Length(min=3, max=20, message='ต้องมี 3-20 ตัวอักษร')
    ])
    
    # Email
    email = EmailField('อีเมล', validators=[
        DataRequired(),
        Email(message='อีเมลไม่ถูกต้อง')
    ])
    
    # NumberRange
    age = IntegerField('อายุ', validators=[
        DataRequired(),
        NumberRange(min=0, max=150, message='อายุต้องอยู่ระหว่าง 0-150 ปี')
    ])
    
    # URL
    website = URLField('เว็บไซต์', validators=[
        Optional(),
        URL(require_tld=True, message='URL ไม่ถูกต้อง')
    ])
    
    # EqualTo - เปรียบเทียบสองฟิลด์
    password = StringField('รหัสผ่าน', validators=[DataRequired()])
    confirm_password = StringField('ยืนยันรหัสผ่าน', validators=[
        DataRequired(),
        EqualTo('password', message='รหัสผ่านไม่ตรงกัน')
    ])
    
    # Regexp - pattern matching
    phone = StringField('เบอร์โทร', validators=[
        Optional(),
        Regexp(r'^0[0-9]{9}$', message='เบอร์โทรต้องเป็น 10 หลัก ขึ้นต้นด้วย 0')
    ])
    
    # NoneOf - ต้องไม่เป็นค่าใดๆ ใน list
    username2 = StringField('Username', validators=[
        NoneOf(['admin', 'root', 'system'], message='ชื่อนี้ไม่สามารถใช้ได้')
    ])
    
    # AnyOf - ต้องเป็นค่าใดค่าหนึ่ง
    status = StringField('สถานะ', validators=[
        AnyOf(['active', 'inactive', 'pending'], message='สถานะไม่ถูกต้อง')
    ])
```

---

## 7. Custom Validators

### Function Validators

```python
from flask_wtf import FlaskForm
from wtforms import StringField, IntegerField, EmailField
from wtforms.validators import DataRequired, ValidationError

# Validator แบบ function
def validate_thai_id(form, field):
    """ตรวจสอบเลขบัตรประชาชนไทย"""
    id_num = field.data
    
    if not id_num:
        return
    
    if not id_num.isdigit() or len(id_num) != 13:
        raise ValidationError('เลขบัตรประชาชนต้องเป็นตัวเลข 13 หลัก')
    
    # ตรวจสอบ checksum
    total = sum(int(id_num[i]) * (13 - i) for i in range(12))
    check_digit = (11 - (total % 11)) % 10
    
    if check_digit != int(id_num[12]):
        raise ValidationError('เลขบัตรประชาชนไม่ถูกต้อง')


def validate_username_available(form, field):
    """ตรวจสอบว่า username ยังไม่มีใครใช้"""
    # จำลอง database check
    taken_usernames = ['admin', 'user1', 'test']
    if field.data.lower() in taken_usernames:
        raise ValidationError(f'ชื่อผู้ใช้ "{field.data}" มีผู้ใช้แล้ว')


class UserForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(),
        validate_username_available  # custom validator function
    ])
    
    thai_id = StringField('เลขบัตรประชาชน', validators=[
        validate_thai_id  # custom validator
    ])
```

### Class-based Validators

```python
from wtforms.validators import ValidationError
import re

class PasswordStrength:
    """Validator ตรวจสอบความแข็งแกร่งของรหัสผ่าน"""
    
    def __init__(self, min_length=8, require_uppercase=True,
                 require_lowercase=True, require_digit=True,
                 require_special=False, message=None):
        self.min_length = min_length
        self.require_uppercase = require_uppercase
        self.require_lowercase = require_lowercase
        self.require_digit = require_digit
        self.require_special = require_special
        self.message = message
    
    def __call__(self, form, field):
        password = field.data
        errors = []
        
        if len(password) < self.min_length:
            errors.append(f'รหัสผ่านต้องมีอย่างน้อย {self.min_length} ตัวอักษร')
        
        if self.require_uppercase and not re.search(r'[A-Z]', password):
            errors.append('ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว')
        
        if self.require_lowercase and not re.search(r'[a-z]', password):
            errors.append('ต้องมีตัวพิมพ์เล็กอย่างน้อย 1 ตัว')
        
        if self.require_digit and not re.search(r'[0-9]', password):
            errors.append('ต้องมีตัวเลขอย่างน้อย 1 ตัว')
        
        if self.require_special and not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            errors.append('ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว')
        
        if errors:
            if self.message:
                raise ValidationError(self.message)
            raise ValidationError('; '.join(errors))


class Blacklist:
    """Validator ตรวจสอบ blacklist"""
    
    def __init__(self, blacklist, message='ค่านี้ไม่สามารถใช้ได้'):
        self.blacklist = [item.lower() for item in blacklist]
        self.message = message
    
    def __call__(self, form, field):
        if field.data and field.data.lower() in self.blacklist:
            raise ValidationError(self.message)


# ใช้งาน
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField
from wtforms.validators import DataRequired

class RegistrationForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(),
        Blacklist(['admin', 'root', 'system', 'administrator'],
                  'ชื่อนี้สงวนไว้')
    ])
    
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(),
        PasswordStrength(
            min_length=10,
            require_uppercase=True,
            require_digit=True,
            require_special=True
        )
    ])
```

### Validators ที่ใช้ข้อมูลจาก Database

```python
from flask_wtf import FlaskForm
from wtforms import StringField, EmailField
from wtforms.validators import DataRequired, ValidationError, Email

class UniqueEmailForm(FlaskForm):
    email = EmailField('อีเมล', validators=[
        DataRequired(),
        Email()
    ])
    
    def validate_email(self, field):
        """Custom validator สำหรับ field เฉพาะ (ตั้งชื่อ validate_<fieldname>)"""
        # จำลอง database check
        existing_emails = ['taken@example.com', 'admin@example.com']
        
        if field.data.lower() in existing_emails:
            raise ValidationError('อีเมลนี้มีผู้ใช้แล้ว')

class UpdateProfileForm(FlaskForm):
    email = EmailField('อีเมล', validators=[DataRequired(), Email()])
    
    def __init__(self, current_user_id=None, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.current_user_id = current_user_id
    
    def validate_email(self, field):
        """ตรวจสอบว่าอีเมลซ้ำกับคนอื่น (ไม่ใช่ตัวเอง)"""
        # User อาจจะเก็บอีเมลเดิมไว้ก็ได้
        users_db = {
            1: {'email': 'user1@example.com'},
            2: {'email': 'user2@example.com'},
        }
        
        for user_id, user_data in users_db.items():
            if user_data['email'] == field.data and user_id != self.current_user_id:
                raise ValidationError('อีเมลนี้มีผู้ใช้แล้ว')
```

---

## 8. CSRF Protection

### CSRF คืออะไร

CSRF (Cross-Site Request Forgery) คือการโจมตีที่ผู้ไม่หวังดีหลอกให้ browser ของ user ส่ง request ไปยัง site ที่ user login อยู่โดยที่ user ไม่รู้ตัว

### วิธีป้องกัน CSRF ด้วย Flask-WTF

```python
from flask import Flask
from flask_wtf.csrf import CSRFProtect, generate_csrf

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'
app.config['WTF_CSRF_ENABLED'] = True       # เปิดใช้งาน (default)
app.config['WTF_CSRF_TIME_LIMIT'] = 3600    # token หมดอายุใน 1 ชั่วโมง

csrf = CSRFProtect(app)
```

### CSRF Token ใน Forms

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

class ContactForm(FlaskForm):
    name = StringField('ชื่อ')
    message = StringField('ข้อความ')
    submit = SubmitField('ส่ง')
```

```html
<!-- Template: CSRF token จะ include อัตโนมัติ -->
<form method="POST">
    {{ form.hidden_tag() }}  {# รวม CSRF token และ hidden fields ทั้งหมด #}
    {# หรือ #}
    {{ form.csrf_token }}    {# เฉพาะ CSRF token #}
    
    {{ form.name.label }}
    {{ form.name() }}
    
    {{ form.submit() }}
</form>
```

### CSRF สำหรับ AJAX Requests

```python
# Flask side
from flask import Flask, jsonify, request
from flask_wtf.csrf import CSRFProtect, generate_csrf

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'
csrf = CSRFProtect(app)

# API endpoint ที่ต้อง CSRF
@app.route('/api/update', methods=['POST'])
def api_update():
    data = request.get_json()
    return jsonify({'status': 'ok'})

# Exempt API endpoints จาก CSRF (สำหรับ external API)
@csrf.exempt
@app.route('/api/webhook', methods=['POST'])
def webhook():
    # Webhook ที่มาจาก external service
    return jsonify({'status': 'received'})

# ส่ง CSRF token ไปให้ JavaScript
@app.route('/get-csrf-token')
def get_csrf_token():
    return jsonify({'csrf_token': generate_csrf()})
```

```javascript
// JavaScript side
// วิธีที่ 1: ดึง CSRF token จาก meta tag
const csrfToken = document.querySelector('meta[name="csrf-token"]').content;

// วิธีที่ 2: ดึงจาก API
fetch('/get-csrf-token')
    .then(r => r.json())
    .then(data => {
        // ใช้ token สำหรับ requests ต่อไป
        const token = data.csrf_token;
        
        fetch('/api/update', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': token  // ส่ง token ใน header
            },
            body: JSON.stringify({data: 'value'})
        });
    });
```

---

## 9. File Uploads

### การรับไฟล์ upload

```python
import os
from flask import Flask, request, redirect, url_for, flash
from werkzeug.utils import secure_filename
from flask_wtf import FlaskForm
from flask_wtf.file import FileField, FileRequired, FileAllowed
from wtforms import SubmitField

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret'
app.config['UPLOAD_FOLDER'] = 'uploads'
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16MB max

# สร้าง folder ถ้ายังไม่มี
os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)

ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'pdf', 'doc', 'docx'}

def allowed_file(filename):
    """ตรวจสอบนามสกุลไฟล์"""
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

class UploadForm(FlaskForm):
    photo = FileField('รูปภาพ', validators=[
        FileRequired(message='กรุณาเลือกไฟล์'),
        FileAllowed(['jpg', 'jpeg', 'png', 'gif'], 'เฉพาะรูปภาพเท่านั้น!')
    ])
    submit = SubmitField('อัพโหลด')

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    form = UploadForm()
    
    if form.validate_on_submit():
        file = form.photo.data
        
        # Secure filename (ป้องกัน path traversal attack)
        filename = secure_filename(file.filename)
        
        # เพิ่ม timestamp เพื่อป้องกัน filename conflicts
        import time
        name, ext = os.path.splitext(filename)
        filename = f"{name}_{int(time.time())}{ext}"
        
        # บันทึกไฟล์
        file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(file_path)
        
        flash(f'อัพโหลดสำเร็จ: {filename}', 'success')
        return redirect(url_for('upload_file'))
    
    return render_template_string('''
    <form method="POST" enctype="multipart/form-data">
        {{ form.hidden_tag() }}
        {{ form.photo.label }}
        {{ form.photo() }}
        {% for error in form.photo.errors %}
        <p style="color:red">{{ error }}</p>
        {% endfor %}
        {{ form.submit() }}
    </form>
    ''', form=form)
```

### Advanced File Upload

```python
import os
import uuid
from PIL import Image  # pip install Pillow
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'uploads'
app.config['MAX_CONTENT_LENGTH'] = 10 * 1024 * 1024  # 10MB

def generate_unique_filename(original_filename):
    """สร้าง unique filename"""
    ext = os.path.splitext(original_filename)[1].lower()
    return str(uuid.uuid4()) + ext

def resize_image(filepath, max_size=(800, 800)):
    """ย่อขนาดรูปภาพ"""
    with Image.open(filepath) as img:
        img.thumbnail(max_size, Image.LANCZOS)
        img.save(filepath, optimize=True, quality=85)

@app.route('/upload-image', methods=['POST'])
def upload_image():
    if 'image' not in request.files:
        return jsonify({'error': 'ไม่พบไฟล์'}), 400
    
    file = request.files['image']
    
    if file.filename == '':
        return jsonify({'error': 'ไม่ได้เลือกไฟล์'}), 400
    
    # ตรวจสอบ MIME type
    allowed_mimes = {'image/jpeg', 'image/png', 'image/gif', 'image/webp'}
    if file.content_type not in allowed_mimes:
        return jsonify({'error': 'ประเภทไฟล์ไม่รองรับ'}), 400
    
    # สร้าง filename ที่ unique
    filename = generate_unique_filename(file.filename)
    filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
    
    # บันทึกไฟล์
    file.save(filepath)
    
    # ย่อขนาด (ถ้าเป็นรูปภาพ)
    try:
        resize_image(filepath)
    except Exception:
        pass  # ถ้า resize ไม่ได้ ก็ไม่เป็นไร
    
    file_url = f'/uploads/{filename}'
    
    return jsonify({
        'success': True,
        'filename': filename,
        'url': file_url,
        'size': os.path.getsize(filepath)
    })

@app.errorhandler(413)
def too_large(e):
    return jsonify({'error': 'ไฟล์ใหญ่เกินไป (สูงสุด 10MB)'}), 413
```

---

## 10. Form Rendering ใน Jinja2

### Manual Form Rendering

```html
<!-- templates/forms/manual.html -->
<form method="POST" action="{{ action_url }}">
    {{ form.hidden_tag() }}
    
    <!-- Text field พร้อม error -->
    <div class="mb-3">
        {{ form.username.label(class="form-label") }}
        {{ form.username(class="form-control" + (" is-invalid" if form.username.errors else "")) }}
        {% if form.username.errors %}
        <div class="invalid-feedback">
            {% for error in form.username.errors %}
            {{ error }}{% if not loop.last %}, {% endif %}
            {% endfor %}
        </div>
        {% endif %}
    </div>
    
    <!-- Password field -->
    <div class="mb-3">
        {{ form.password.label(class="form-label") }}
        {{ form.password(class="form-control") }}
        {% if form.password.errors %}
        <div class="text-danger small">
            {% for error in form.password.errors %}{{ error }}{% endfor %}
        </div>
        {% endif %}
    </div>
    
    <!-- Select field -->
    <div class="mb-3">
        {{ form.city.label(class="form-label") }}
        {{ form.city(class="form-select") }}
    </div>
    
    <!-- Checkbox -->
    <div class="mb-3 form-check">
        {{ form.agree(class="form-check-input") }}
        {{ form.agree.label(class="form-check-label") }}
        {% if form.agree.errors %}
        <div class="text-danger small">{{ form.agree.errors[0] }}</div>
        {% endif %}
    </div>
    
    {{ form.submit(class="btn btn-primary") }}
</form>
```

### Macro สำหรับ Form Fields

```html
<!-- templates/macros/forms.html -->
{% macro render_field(field, extra_classes="") %}
<div class="mb-3">
    {{ field.label(class="form-label fw-semibold") }}
    
    {# กำหนด CSS class ตาม field type #}
    {% if field.type in ['SelectField', 'SelectMultipleField'] %}
        {% set field_class = "form-select" %}
    {% elif field.type == 'BooleanField' %}
        {% set field_class = "form-check-input" %}
    {% elif field.type == 'TextAreaField' %}
        {% set field_class = "form-control" %}
    {% else %}
        {% set field_class = "form-control" %}
    {% endif %}
    
    {# เพิ่ม is-invalid ถ้ามี error #}
    {% if field.errors %}
        {% set field_class = field_class + " is-invalid" %}
    {% endif %}
    
    {{ field(class=field_class + " " + extra_classes) }}
    
    {% if field.errors %}
    <div class="invalid-feedback">
        {% for error in field.errors %}{{ error }}{% endfor %}
    </div>
    {% elif field.description %}
    <div class="form-text text-muted">{{ field.description }}</div>
    {% endif %}
</div>
{% endmacro %}

{% macro render_checkbox(field) %}
<div class="mb-3 form-check">
    {{ field(class="form-check-input" + (" is-invalid" if field.errors else "")) }}
    {{ field.label(class="form-check-label") }}
    {% if field.errors %}
    <div class="invalid-feedback">{{ field.errors[0] }}</div>
    {% endif %}
</div>
{% endmacro %}

{% macro render_submit(field, extra_classes="btn-primary") %}
{{ field(class="btn " + extra_classes) }}
{% endmacro %}
```

### ใช้ Macro ใน Templates

```html
<!-- templates/register.html -->
{% extends 'base.html' %}
{% from 'macros/forms.html' import render_field, render_checkbox, render_submit %}

{% block content %}
<div class="container" style="max-width: 500px">
    <h2>สมัครสมาชิก</h2>
    
    <form method="POST" enctype="multipart/form-data">
        {{ form.hidden_tag() }}
        
        {{ render_field(form.username) }}
        {{ render_field(form.email) }}
        {{ render_field(form.password) }}
        {{ render_field(form.confirm_password) }}
        {{ render_field(form.city) }}
        {{ render_checkbox(form.agree) }}
        {{ render_submit(form.submit) }}
    </form>
</div>
{% endblock %}
```

---

## 11. Form Validation Messages

### Custom Error Messages

```python
from flask_wtf import FlaskForm
from wtforms import StringField, IntegerField
from wtforms.validators import DataRequired, Length, NumberRange

class ProductForm(FlaskForm):
    # Custom messages
    name = StringField('ชื่อสินค้า', validators=[
        DataRequired(message='กรุณากรอกชื่อสินค้า'),
        Length(
            min=2, max=100,
            message='ชื่อสินค้าต้องมีความยาว %(min)d-%(max)d ตัวอักษร'
        )
    ])
    
    price = IntegerField('ราคา', validators=[
        DataRequired(message='กรุณากรอกราคา'),
        NumberRange(
            min=1,
            message='ราคาต้องมากกว่า %(min)d บาท'
        )
    ])
```

### แสดง Errors ใน Template

```html
<!-- แสดง errors หลายรูปแบบ -->

<!-- รูปแบบที่ 1: แสดงทีละ field -->
{% if form.username.errors %}
<ul class="errors">
    {% for error in form.username.errors %}
    <li>{{ error }}</li>
    {% endfor %}
</ul>
{% endif %}

<!-- รูปแบบที่ 2: แสดง error แรกอย่างเดียว -->
{% if form.username.errors %}
<span class="error">{{ form.username.errors[0] }}</span>
{% endif %}

<!-- รูปแบบที่ 3: แสดง errors ทั้งหมดรวมกัน -->
{% if form.errors %}
<div class="alert alert-danger">
    <ul>
    {% for field_name, errors in form.errors.items() %}
        {% for error in errors %}
        <li>{{ form[field_name].label.text }}: {{ error }}</li>
        {% endfor %}
    {% endfor %}
    </ul>
</div>
{% endif %}
```

### การส่ง Errors ผ่าน Flash Messages

```python
from flask import flash, redirect, url_for
from flask_wtf import FlaskForm

@app.route('/submit', methods=['POST'])
def submit():
    form = MyForm()
    
    if form.validate_on_submit():
        # Success
        flash('บันทึกสำเร็จ!', 'success')
        return redirect(url_for('index'))
    else:
        # ส่ง validation errors เป็น flash messages
        for field_name, errors in form.errors.items():
            field_label = getattr(form, field_name).label.text
            for error in errors:
                flash(f'{field_label}: {error}', 'danger')
    
    return render_template('form.html', form=form)
```

---

## 12. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 1: Registration Form

```python
# registration_app.py
# Complete registration system

from flask import Flask, render_template_string, redirect, url_for, flash
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, EmailField, SelectField
from wtforms import TextAreaField, BooleanField, IntegerField, SubmitField
from wtforms.validators import DataRequired, Email, Length, EqualTo, NumberRange
from wtforms.validators import Optional, ValidationError
import re

app = Flask(__name__)
app.config['SECRET_KEY'] = 'registration-secret-key'

# Mock users database
users_db = {}

class RegistrationForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(message='กรุณากรอกชื่อผู้ใช้'),
        Length(min=3, max=20, message='ต้องมี 3-20 ตัวอักษร')
    ], description='ตัวอักษร ตัวเลข และ _ เท่านั้น')
    
    email = EmailField('อีเมล', validators=[
        DataRequired(message='กรุณากรอกอีเมล'),
        Email(message='รูปแบบอีเมลไม่ถูกต้อง')
    ])
    
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(message='กรุณากรอกรหัสผ่าน'),
        Length(min=8, message='รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    ])
    
    confirm_password = PasswordField('ยืนยันรหัสผ่าน', validators=[
        DataRequired(message='กรุณายืนยันรหัสผ่าน'),
        EqualTo('password', message='รหัสผ่านไม่ตรงกัน')
    ])
    
    full_name = StringField('ชื่อ-นามสกุล', validators=[
        DataRequired(message='กรุณากรอกชื่อ-นามสกุล')
    ])
    
    age = IntegerField('อายุ', validators=[
        DataRequired(message='กรุณากรอกอายุ'),
        NumberRange(min=13, max=120, message='อายุต้องอยู่ระหว่าง 13-120 ปี')
    ])
    
    gender = SelectField('เพศ', choices=[
        ('', 'เลือกเพศ'),
        ('male', 'ชาย'),
        ('female', 'หญิง'),
        ('other', 'ไม่ระบุ')
    ], validators=[DataRequired(message='กรุณาเลือกเพศ')])
    
    phone = StringField('เบอร์โทร', validators=[Optional()])
    
    bio = TextAreaField('แนะนำตัว', validators=[
        Optional(),
        Length(max=500, message='ไม่เกิน 500 ตัวอักษร')
    ])
    
    agree_terms = BooleanField('ฉันยอมรับข้อกำหนดและเงื่อนไข', validators=[
        DataRequired(message='คุณต้องยอมรับข้อกำหนด')
    ])
    
    submit = SubmitField('สมัครสมาชิก')
    
    def validate_username(self, field):
        """ตรวจสอบ username เพิ่มเติม"""
        if not re.match(r'^[a-zA-Z0-9_]+$', field.data):
            raise ValidationError('ชื่อผู้ใช้ต้องเป็นตัวอักษร ตัวเลข หรือ _ เท่านั้น')
        if field.data.lower() in users_db:
            raise ValidationError('ชื่อผู้ใช้นี้มีผู้ใช้แล้ว')
    
    def validate_email(self, field):
        """ตรวจสอบ email ซ้ำ"""
        for user in users_db.values():
            if user['email'].lower() == field.data.lower():
                raise ValidationError('อีเมลนี้มีผู้ใช้แล้ว')
    
    def validate_phone(self, field):
        """ตรวจสอบเบอร์โทร"""
        if field.data:
            clean_phone = re.sub(r'[\s\-\(\)]', '', field.data)
            if not re.match(r'^(0[0-9]{9}|[+]66[0-9]{9})$', clean_phone):
                raise ValidationError('รูปแบบเบอร์โทรไม่ถูกต้อง')
    
    def validate_password(self, field):
        """ตรวจสอบความแข็งแกร่งรหัสผ่าน"""
        password = field.data
        if not re.search(r'[A-Z]', password):
            raise ValidationError('รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว')
        if not re.search(r'[0-9]', password):
            raise ValidationError('รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว')

REGISTER_HTML = """
<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>สมัครสมาชิก</title>
<style>
* { box-sizing: border-box; }
body { font-family: 'Sarabun', Arial, sans-serif; background: #f5f5f5; margin: 0; padding: 20px; }
.container { max-width: 600px; margin: 0 auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 15px rgba(0,0,0,0.1); }
h2 { color: #2c3e50; margin-bottom: 25px; text-align: center; }
.form-group { margin-bottom: 18px; }
label { display: block; font-weight: 600; color: #555; margin-bottom: 5px; font-size: 0.95em; }
input, select, textarea { width: 100%; padding: 10px 12px; border: 1px solid #ddd; border-radius: 6px; font-size: 0.95em; transition: border-color 0.2s; }
input:focus, select:focus, textarea:focus { border-color: #3498db; outline: none; box-shadow: 0 0 0 3px rgba(52,152,219,0.1); }
.is-invalid { border-color: #e74c3c !important; }
.error-msg { color: #e74c3c; font-size: 0.82em; margin-top: 4px; }
.hint { color: #999; font-size: 0.8em; margin-top: 3px; }
.form-check { display: flex; align-items: center; gap: 10px; }
.form-check input { width: auto; }
.btn { width: 100%; padding: 12px; background: #3498db; color: white; border: none; border-radius: 6px; font-size: 1em; cursor: pointer; margin-top: 10px; }
.btn:hover { background: #2980b9; }
.alert { padding: 12px; border-radius: 6px; margin-bottom: 20px; }
.alert-success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.alert-danger { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
.required { color: #e74c3c; }
.row { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
</style>
</head>
<body>
<div class="container">
    <h2>📝 สมัครสมาชิก</h2>
    
    {% with messages = get_flashed_messages(with_categories=true) %}
    {% for category, message in messages %}
    <div class="alert alert-{{ category }}">{{ message }}</div>
    {% endfor %}
    {% endwith %}
    
    <form method="POST">
        {{ form.hidden_tag() }}
        
        <div class="form-group">
            {{ form.username.label }}
            {{ form.username(placeholder="กรอกชื่อผู้ใช้...", class=("is-invalid" if form.username.errors else "")) }}
            {% for error in form.username.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            {% if form.username.description %}<div class="hint">{{ form.username.description }}</div>{% endif %}
        </div>
        
        <div class="form-group">
            {{ form.email.label }}
            {{ form.email(placeholder="example@email.com", class=("is-invalid" if form.email.errors else "")) }}
            {% for error in form.email.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
        </div>
        
        <div class="row">
            <div class="form-group">
                {{ form.password.label }}
                {{ form.password(class=("is-invalid" if form.password.errors else "")) }}
                {% for error in form.password.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            </div>
            <div class="form-group">
                {{ form.confirm_password.label }}
                {{ form.confirm_password(class=("is-invalid" if form.confirm_password.errors else "")) }}
                {% for error in form.confirm_password.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            </div>
        </div>
        
        <div class="row">
            <div class="form-group">
                {{ form.full_name.label }}
                {{ form.full_name(placeholder="ชื่อ นามสกุล", class=("is-invalid" if form.full_name.errors else "")) }}
                {% for error in form.full_name.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            </div>
            <div class="form-group">
                {{ form.age.label }}
                {{ form.age(placeholder="อายุ", class=("is-invalid" if form.age.errors else "")) }}
                {% for error in form.age.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            </div>
        </div>
        
        <div class="row">
            <div class="form-group">
                {{ form.gender.label }}
                {{ form.gender(class=("is-invalid" if form.gender.errors else "")) }}
                {% for error in form.gender.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
            </div>
            <div class="form-group">
                {{ form.phone.label }}
                {{ form.phone(placeholder="08XXXXXXXX") }}
            </div>
        </div>
        
        <div class="form-group">
            {{ form.bio.label }}
            {{ form.bio(rows=3, placeholder="แนะนำตัวสั้นๆ...") }}
            {% for error in form.bio.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
        </div>
        
        <div class="form-group form-check">
            {{ form.agree_terms() }}
            {{ form.agree_terms.label }}
        </div>
        {% for error in form.agree_terms.errors %}<div class="error-msg">{{ error }}</div>{% endfor %}
        
        {{ form.submit(class="btn") }}
    </form>
</div>
</body>
</html>
"""

@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    
    if form.validate_on_submit():
        user_data = {
            'username': form.username.data,
            'email': form.email.data,
            'full_name': form.full_name.data,
            'age': form.age.data,
            'gender': form.gender.data,
            'phone': form.phone.data,
            'bio': form.bio.data,
        }
        users_db[form.username.data.lower()] = user_data
        
        flash(f'สมัครสมาชิกสำเร็จ! ยินดีต้อนรับ {form.full_name.data}', 'success')
        return redirect(url_for('register'))
    
    return render_template_string(REGISTER_HTML, form=form)

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 13. แบบฝึกหัด

### ข้อที่ 1: Login Form พร้อม Validation

สร้าง login form ที่มี username/password พร้อม remember me checkbox

**เฉลย:**

```python
from flask import Flask, render_template_string, redirect, url_for, flash, session
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired, Length

app = Flask(__name__)
app.config['SECRET_KEY'] = 'login-secret'

FAKE_USERS = {'admin': 'Password1', 'user': 'Password2'}

class LoginForm(FlaskForm):
    username = StringField('ชื่อผู้ใช้', validators=[
        DataRequired(message='กรุณากรอกชื่อผู้ใช้'),
        Length(min=3, max=20)
    ])
    password = PasswordField('รหัสผ่าน', validators=[
        DataRequired(message='กรุณากรอกรหัสผ่าน')
    ])
    remember_me = BooleanField('จดจำฉัน')
    submit = SubmitField('เข้าสู่ระบบ')

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        username = form.username.data
        password = form.password.data
        if username in FAKE_USERS and FAKE_USERS[username] == password:
            session['username'] = username
            if form.remember_me.data:
                session.permanent = True
            flash(f'ยินดีต้อนรับ {username}!', 'success')
            return redirect(url_for('dashboard'))
        flash('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง', 'danger')
    
    return render_template_string("""
    <form method="POST" style="max-width:400px;margin:40px auto;padding:20px;background:#fff;border-radius:8px;box-shadow:0 2px 10px rgba(0,0,0,0.1)">
        <h2>เข้าสู่ระบบ</h2>
        {{ form.hidden_tag() }}
        {% with messages = get_flashed_messages(with_categories=true) %}
        {% for cat, msg in messages %}<div style="padding:8px;background:#{{ 'd4edda' if cat == 'success' else 'f8d7da' }};border-radius:4px;margin-bottom:10px">{{ msg }}</div>{% endfor %}
        {% endwith %}
        {% for field in [form.username, form.password] %}
        <div style="margin-bottom:12px">
            {{ field.label }}<br>
            {{ field(style="width:100%;padding:8px;border:1px solid #ddd;border-radius:4px") }}
            {% for err in field.errors %}<p style="color:red;font-size:0.85em">{{ err }}</p>{% endfor %}
        </div>
        {% endfor %}
        <div style="margin-bottom:12px">{{ form.remember_me() }} {{ form.remember_me.label }}</div>
        {{ form.submit(style="width:100%;padding:10px;background:#007bff;color:white;border:none;border-radius:4px;cursor:pointer") }}
    </form>
    """, form=form)

@app.route('/dashboard')
def dashboard():
    username = session.get('username')
    if not username:
        return redirect(url_for('login'))
    return f'<h1>Dashboard</h1><p>ยินดีต้อนรับ {username}!</p><a href="/logout">ออกจากระบบ</a>'

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('login'))

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 2: Contact Form

สร้าง contact form ที่มีชื่อ อีเมล หัวข้อ และข้อความ

**เฉลย:**

```python
from flask import Flask, render_template_string, redirect, url_for, flash
from flask_wtf import FlaskForm
from wtforms import StringField, EmailField, SelectField, TextAreaField, SubmitField
from wtforms.validators import DataRequired, Email, Length

app = Flask(__name__)
app.config['SECRET_KEY'] = 'contact-secret'

class ContactForm(FlaskForm):
    name = StringField('ชื่อ-นามสกุล', validators=[DataRequired()])
    email = EmailField('อีเมล', validators=[DataRequired(), Email()])
    subject = SelectField('หัวข้อ', choices=[
        ('general', 'สอบถามทั่วไป'),
        ('support', 'ขอความช่วยเหลือ'),
        ('feedback', 'แสดงความคิดเห็น'),
        ('bug', 'รายงานปัญหา'),
    ])
    message = TextAreaField('ข้อความ', validators=[
        DataRequired(message='กรุณากรอกข้อความ'),
        Length(min=10, max=1000, message='ข้อความต้องมี 10-1000 ตัวอักษร')
    ])
    submit = SubmitField('ส่งข้อความ')

messages_store = []

@app.route('/contact', methods=['GET', 'POST'])
def contact():
    form = ContactForm()
    if form.validate_on_submit():
        messages_store.append({
            'name': form.name.data,
            'email': form.email.data,
            'subject': form.subject.data,
            'message': form.message.data
        })
        flash('ส่งข้อความสำเร็จ! เราจะติดต่อกลับภายใน 24 ชั่วโมง', 'success')
        return redirect(url_for('contact'))
    
    return render_template_string("""
    <div style="max-width:600px;margin:40px auto;padding:20px;font-family:Arial">
    <h2>ติดต่อเรา</h2>
    {% with messages = get_flashed_messages(with_categories=true) %}
    {% for cat, msg in messages %}<div style="padding:10px;background:#d4edda;color:#155724;border-radius:4px;margin-bottom:15px">{{ msg }}</div>{% endfor %}
    {% endwith %}
    <form method="POST">
        {{ form.hidden_tag() }}
        {% for field in [form.name, form.email, form.subject, form.message] %}
        <div style="margin-bottom:15px">
            {{ field.label(style="display:block;font-weight:bold;margin-bottom:5px") }}
            {{ field(style="width:100%;padding:8px;border:1px solid #ddd;border-radius:4px;box-sizing:border-box") }}
            {% for err in field.errors %}<p style="color:red;font-size:0.85em;margin:3px 0">{{ err }}</p>{% endfor %}
        </div>
        {% endfor %}
        {{ form.submit(style="background:#007bff;color:white;border:none;padding:10px 24px;border-radius:4px;cursor:pointer") }}
    </form>
    <div style="margin-top:20px"><a href="/messages">ดูข้อความทั้งหมด</a></div>
    </div>
    """, form=form)

@app.route('/messages')
def view_messages():
    return render_template_string("""
    <div style="max-width:600px;margin:40px auto;padding:20px;font-family:Arial">
    <h2>ข้อความทั้งหมด ({{ messages|length }})</h2>
    {% for msg in messages %}
    <div style="border:1px solid #ddd;padding:15px;margin:10px 0;border-radius:6px">
        <strong>{{ msg.name }}</strong> ({{ msg.email }})<br>
        <em>{{ msg.subject }}</em><br>
        <p>{{ msg.message }}</p>
    </div>
    {% endfor %}
    <a href="/contact">← กลับ</a>
    </div>
    """, messages=messages_store)

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 3: Profile Update Form

สร้าง form แก้ไขโปรไฟล์ผู้ใช้ที่มี pre-filled data

**เฉลย:**

```python
from flask import Flask, render_template_string, redirect, url_for, flash
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField, SelectField, SubmitField
from wtforms.validators import DataRequired, Length, Optional, URL

app = Flask(__name__)
app.config['SECRET_KEY'] = 'profile-secret'

CURRENT_USER = {
    'id': 1, 'username': 'somchai', 'full_name': 'สมชาย ใจดี',
    'bio': 'นักพัฒนา Python', 'website': 'https://somchai.dev',
    'city': 'bkk', 'gender': 'male'
}

class ProfileForm(FlaskForm):
    full_name = StringField('ชื่อ-นามสกุล', validators=[DataRequired()])
    bio = TextAreaField('แนะนำตัว', validators=[Optional(), Length(max=500)])
    website = StringField('เว็บไซต์', validators=[Optional(), URL()])
    city = SelectField('เมือง', choices=[
        ('bkk', 'กรุงเทพฯ'), ('cm', 'เชียงใหม่'),
        ('kkn', 'ขอนแก่น'), ('other', 'อื่นๆ')
    ])
    gender = SelectField('เพศ', choices=[
        ('male', 'ชาย'), ('female', 'หญิง'), ('other', 'ไม่ระบุ')
    ])
    submit = SubmitField('บันทึก')

@app.route('/profile/edit', methods=['GET', 'POST'])
def edit_profile():
    form = ProfileForm(data=CURRENT_USER)  # Pre-fill with current data
    if form.validate_on_submit():
        CURRENT_USER.update({
            'full_name': form.full_name.data,
            'bio': form.bio.data,
            'website': form.website.data,
            'city': form.city.data,
            'gender': form.gender.data
        })
        flash('บันทึกโปรไฟล์สำเร็จ!', 'success')
        return redirect(url_for('edit_profile'))
    
    return render_template_string("""
    <div style="max-width:500px;margin:40px auto;padding:20px;font-family:Arial;background:#fff;border-radius:8px;box-shadow:0 2px 10px rgba(0,0,0,0.1)">
    <h2>แก้ไขโปรไฟล์</h2>
    {% with messages = get_flashed_messages(with_categories=true) %}
    {% for cat, msg in messages %}<div style="padding:10px;background:#d4edda;color:#155724;border-radius:4px;margin-bottom:15px">{{ msg }}</div>{% endfor %}
    {% endwith %}
    <form method="POST">
        {{ form.hidden_tag() }}
        {% for field in [form.full_name, form.bio, form.website, form.city, form.gender] %}
        <div style="margin-bottom:15px">
            {{ field.label(style="display:block;font-weight:bold;margin-bottom:5px") }}
            {{ field(style="width:100%;padding:8px;border:1px solid #ddd;border-radius:4px;box-sizing:border-box") }}
            {% for err in field.errors %}<p style="color:red;font-size:0.85em">{{ err }}</p>{% endfor %}
        </div>
        {% endfor %}
        {{ form.submit(style="background:#28a745;color:white;border:none;padding:10px 20px;border-radius:4px;cursor:pointer") }}
    </form>
    </div>
    """, form=form)

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 4-8 (ย่อ)

```python
# ข้อที่ 4: File Upload Form
from flask import Flask, render_template_string, redirect, url_for, flash
from flask_wtf import FlaskForm
from flask_wtf.file import FileField, FileRequired, FileAllowed
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)
app.config['SECRET_KEY'] = 'upload-secret'
app.config['UPLOAD_FOLDER'] = 'uploads'
os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)

class DocumentUploadForm(FlaskForm):
    title = StringField('ชื่อเอกสาร', validators=[DataRequired()])
    file = FileField('ไฟล์', validators=[
        FileRequired(),
        FileAllowed(['pdf', 'doc', 'docx', 'txt'], 'เฉพาะ PDF, Word, Text เท่านั้น')
    ])
    submit = SubmitField('อัพโหลด')

documents = []

@app.route('/upload-doc', methods=['GET', 'POST'])
def upload_doc():
    form = DocumentUploadForm()
    if form.validate_on_submit():
        file = form.file.data
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        documents.append({'title': form.title.data, 'filename': filename})
        flash('อัพโหลดสำเร็จ!', 'success')
        return redirect(url_for('upload_doc'))
    
    return render_template_string("""
    <div style="max-width:400px;margin:40px auto;font-family:Arial">
    <h2>อัพโหลดเอกสาร</h2>
    {% with m = get_flashed_messages(with_categories=true) %}{% for c,msg in m %}<div style="padding:8px;background:#d4edda;border-radius:4px;margin-bottom:10px">{{ msg }}</div>{% endfor %}{% endwith %}
    <form method="POST" enctype="multipart/form-data">
        {{ form.hidden_tag() }}
        <div><label>{{ form.title.label }}</label><br>{{ form.title(style="width:100%;padding:8px;border:1px solid #ddd;border-radius:4px") }}{% for e in form.title.errors %}<p style="color:red">{{ e }}</p>{% endfor %}</div><br>
        <div><label>{{ form.file.label }}</label><br>{{ form.file() }}{% for e in form.file.errors %}<p style="color:red">{{ e }}</p>{% endfor %}</div><br>
        {{ form.submit(style="background:#007bff;color:white;border:none;padding:8px 16px;border-radius:4px;cursor:pointer") }}
    </form>
    <h3>เอกสาร:</h3>
    {% for doc in documents %}<p>{{ doc.title }} ({{ doc.filename }})</p>{% endfor %}
    </div>
    """, form=form, documents=documents)

if __name__ == '__main__':
    app.run(debug=True)
```

---

## สรุป

ใน Part 53 นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|---------------|
| HTML Forms | Basics, encoding types, HTML5 validation |
| Form Handling | request.form, manual validation |
| WTForms | Form classes, field types |
| Flask-WTF | FlaskForm, validate_on_submit() |
| Field Types | String, Numeric, Date, Choice, File, Boolean |
| Built-in Validators | DataRequired, Length, Email, EqualTo, Regexp |
| Custom Validators | Function validators, class validators |
| CSRF Protection | Token generation, AJAX |
| File Uploads | secure_filename, allowed extensions, size limits |
| Form Rendering | Manual, macros, error display |

### ขั้นตอนต่อไป

ใน Part 54 เราจะเรียนรู้:
- Flask-SQLAlchemy database setup
- Database models
- Relationships
- CRUD operations
- Database migrations

---

*เขียนโดยหลักสูตร Python Advanced - Part 53*
