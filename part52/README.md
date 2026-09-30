# Part 52: Flask - Routing, Templates & Static Files

## สารบัญ
1. [URL Routing Rules ครบถ้วน](#1-url-routing-rules-ครบถ้วน)
2. [URL Converters](#2-url-converters)
3. [Variable Rules ขั้นสูง](#3-variable-rules-ขั้นสูง)
4. [Jinja2 Templating Engine](#4-jinja2-templating-engine)
5. [Template Inheritance](#5-template-inheritance)
6. [Template Filters และ Tests](#6-template-filters-และ-tests)
7. [Custom Template Filters](#7-custom-template-filters)
8. [Static Files Management](#8-static-files-management)
9. [url_for() Function](#9-url_for-function)
10. [Redirects](#10-redirects)
11. [Error Handlers](#11-error-handlers)
12. [Flash Messages](#12-flash-messages)
13. [Response Objects](#13-response-objects)
14. [ตัวอย่างโปรแกรมจริง](#14-ตัวอย่างโปรแกรมจริง)
15. [แบบฝึกหัด](#15-แบบฝึกหัด)

---

## 1. URL Routing Rules ครบถ้วน

### พื้นฐาน URL Routing

URL routing คือกระบวนการ map HTTP requests ไปยัง Python functions โดย Flask ใช้ decorator `@app.route()` เพื่อกำหนด routing rules

```python
from flask import Flask

app = Flask(__name__)

# Route พื้นฐาน
@app.route('/')
def index():
    return 'หน้าแรก'

# Route ที่มีหลาย URL
@app.route('/home')
@app.route('/index')
@app.route('/')
def home():
    return 'หน้าแรก'

# Route ที่รองรับหลาย methods
@app.route('/contact', methods=['GET', 'POST'])
def contact():
    return 'ติดต่อเรา'
```

### Routing ด้วย add_url_rule()

```python
from flask import Flask

app = Flask(__name__)

def show_user(username):
    return f'User: {username}'

# เพิ่ม URL rule โดยตรง
app.add_url_rule(
    '/user/<username>',   # URL pattern
    'show_user',          # endpoint name
    show_user             # view function
)

# เทียบเท่ากับ
# @app.route('/user/<username>')
# def show_user(username):
#     return f'User: {username}'
```

### Route Endpoint Names

```python
from flask import Flask, url_for

app = Flask(__name__)

# ชื่อ endpoint คือชื่อ function โดย default
@app.route('/products')
def products():
    return 'Products'

# กำหนดชื่อ endpoint เอง
@app.route('/items', endpoint='item_list')
def show_items():
    return 'Items'

# ใช้ endpoint name ใน url_for()
with app.test_request_context():
    print(url_for('products'))   # /products
    print(url_for('item_list'))  # /items
```

### Strict Slashes

```python
from flask import Flask

app = Flask(__name__)

# strict_slashes=True (default)
# /about/ -> redirect ไป /about
@app.route('/about', strict_slashes=False)
def about():
    return 'About page'

# กับ trailing slash
@app.route('/docs/')  # /docs redirect ไป /docs/
def docs():
    return 'Documentation'
```

### Route Priority และ Order

```python
from flask import Flask

app = Flask(__name__)

# Route ที่ specific กว่าจะถูก match ก่อน
@app.route('/users/new')       # จะถูก match ก่อน
def new_user():
    return 'Create new user'

@app.route('/users/<username>')  # จะถูก match หลัง
def user_profile(username):
    return f'Profile: {username}'
```

---

## 2. URL Converters

### Built-in Converters

Flask มี URL converters ที่ built-in มาให้:

```python
from flask import Flask
import uuid as uuid_lib

app = Flask(__name__)

# string: รับข้อความ (ไม่รวม /)
@app.route('/user/<string:username>')
def user_string(username):
    return f'Username (string): {username!r}'
# /user/john -> username = 'john'
# /user/john/doe -> 404 (string ไม่รับ /)

# int: รับจำนวนเต็ม
@app.route('/post/<int:post_id>')
def post_int(post_id):
    return f'Post ID (int): {post_id}'
# /post/42 -> post_id = 42
# /post/abc -> 404

# float: รับทศนิยม
@app.route('/price/<float:price>')
def price_float(price):
    return f'Price (float): {price}'
# /price/9.99 -> price = 9.99

# path: รับ path (รวม /)
@app.route('/files/<path:filepath>')
def serve_file(filepath):
    return f'File path: {filepath}'
# /files/images/2024/photo.jpg -> filepath = 'images/2024/photo.jpg'

# uuid: รับ UUID
@app.route('/token/<uuid:token>')
def verify_token(token):
    return f'Token: {token}'
# /token/550e8400-e29b-41d4-a716-446655440000 -> token = UUID object
```

### Custom URL Converters

```python
from flask import Flask
from werkzeug.routing import BaseConverter

app = Flask(__name__)

# Custom converter สำหรับ list
class ListConverter(BaseConverter):
    """Converter ที่รับ comma-separated values"""
    
    def to_python(self, value):
        """แปลง URL string เป็น Python object"""
        return value.split(',')
    
    def to_url(self, values):
        """แปลง Python object เป็น URL string"""
        return ','.join(str(v) for v in values)

# Register converter
app.url_map.converters['list'] = ListConverter

@app.route('/tags/<list:tags>')
def show_tags(tags):
    return f'Tags: {tags}'
# /tags/python,flask,web -> tags = ['python', 'flask', 'web']

# Custom converter สำหรับ regex
import re
from werkzeug.routing import BaseConverter

class RegexConverter(BaseConverter):
    """Converter ที่ใช้ regex"""
    
    def __init__(self, url_map, *items):
        super().__init__(url_map)
        self.regex = items[0]

app.url_map.converters['regex'] = RegexConverter

@app.route('/code/<regex("[A-Z]{2}[0-9]{4}"):product_code>')
def product_by_code(product_code):
    return f'Product code: {product_code}'
# /code/TH1234 -> product_code = 'TH1234'
```

---

## 3. Variable Rules ขั้นสูง

### Multiple Variables

```python
from flask import Flask

app = Flask(__name__)

# หลาย variables ใน URL
@app.route('/blog/<int:year>/<int:month>/<slug>')
def blog_post(year, month, slug):
    return f'Blog post: {year}/{month}/{slug}'

# ตัวอย่าง: /blog/2024/01/my-first-post

@app.route('/user/<username>/post/<int:post_id>')
def user_post(username, post_id):
    return f'User {username}, Post #{post_id}'

@app.route('/category/<category>/tag/<tag>')
def category_tag(category, tag):
    return f'Category: {category}, Tag: {tag}'
```

### Query String Parameters

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/search')
def search():
    # ดึง query parameters
    query = request.args.get('q', '')
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 10, type=int)
    sort = request.args.get('sort', 'name')
    order = request.args.get('order', 'asc')
    
    return f'Search: {query}, Page: {page}, Sort: {sort} {order}'

# URL: /search?q=python&page=2&sort=date&order=desc

@app.route('/filter')
def filter_items():
    # รับหลายค่าสำหรับ key เดียว
    categories = request.args.getlist('cat')
    # URL: /filter?cat=python&cat=flask&cat=web
    return f'Categories: {categories}'
```

---

## 4. Jinja2 Templating Engine

### Jinja2 คืออะไร

Jinja2 คือ templating engine สำหรับ Python ที่ใช้กับ Flask โดย default มีความสามารถ:
- แสดงตัวแปร Python ใน HTML
- ใช้ control flow (if/for)
- Template inheritance
- Filters และ functions

### การสร้าง Template

```
project/
├── app.py
└── templates/
    ├── base.html
    ├── index.html
    └── user/
        └── profile.html
```

```python
# app.py
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    # ส่ง variables ไปให้ template
    return render_template('index.html', 
                           title='หน้าแรก',
                           username='สมชาย',
                           items=[1, 2, 3])
```

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <h1>สวัสดี {{ username }}!</h1>
    
    {% if items %}
    <ul>
        {% for item in items %}
        <li>{{ item }}</li>
        {% endfor %}
    </ul>
    {% else %}
    <p>ไม่มีรายการ</p>
    {% endif %}
</body>
</html>
```

### Jinja2 Syntax

```html
<!-- 1. แสดงตัวแปร: {{ }} -->
<p>{{ username }}</p>
<p>{{ user.name }}</p>
<p>{{ users[0] }}</p>
<p>{{ data['key'] }}</p>

<!-- 2. Statements (if, for, etc.): {% %} -->
{% if user.is_admin %}
    <p>Admin user</p>
{% elif user.is_moderator %}
    <p>Moderator</p>
{% else %}
    <p>Regular user</p>
{% endif %}

<!-- 3. Comments: {# #} -->
{# This is a Jinja2 comment - not shown in HTML #}

<!-- 4. Raw text (ไม่ประมวลผล Jinja2): -->
{% raw %}
    {{ this is not processed }}
{% endraw %}
```

### Jinja2 Control Flow

```html
<!-- for loop -->
{% for user in users %}
    <p>{{ loop.index }}. {{ user.name }} ({{ user.email }})</p>
{% else %}
    <p>ไม่มีผู้ใช้</p>
{% endfor %}

<!-- Loop variables พิเศษ -->
{% for item in items %}
    loop.index     - 1-based index
    loop.index0    - 0-based index
    loop.revindex  - reverse index (นับถอยหลัง)
    loop.first     - True ถ้าเป็น item แรก
    loop.last      - True ถ้าเป็น item สุดท้าย
    loop.length    - จำนวน items ทั้งหมด
    loop.depth     - ความลึกของ loop (สำหรับ nested loops)
    loop.depth0    - ความลึก (0-based)
    loop.cycle()   - วนซ้ำผ่านค่าที่กำหนด
{% endfor %}

<!-- ตัวอย่าง loop variables -->
{% for product in products %}
<tr class="{{ loop.cycle('odd', 'even') }}">
    <td>{{ loop.index }}</td>
    <td>{{ product.name }}</td>
    {% if loop.first %}
    <td>⭐ Best seller</td>
    {% endif %}
</tr>
{% endfor %}

<!-- nested loops -->
{% for category in categories %}
    <h2>{{ category.name }}</h2>
    {% for product in category.products %}
        <p>{{ loop.depth }}: {{ product.name }}</p>
    {% endfor %}
{% endfor %}
```

### Jinja2 Expressions

```html
<!-- Arithmetic -->
{{ 2 + 2 }}     → 4
{{ 10 - 5 }}    → 5
{{ 3 * 4 }}     → 12
{{ 9 / 3 }}     → 3.0
{{ 9 // 3 }}    → 3 (integer division)
{{ 9 % 2 }}     → 1 (modulo)
{{ 2 ** 10 }}   → 1024 (power)

<!-- Comparison -->
{{ 1 == 1 }}    → True
{{ 1 != 2 }}    → True
{{ 5 > 3 }}     → True
{{ 5 >= 5 }}    → True

<!-- Logical -->
{{ true and false }}   → False
{{ true or false }}    → True
{{ not true }}         → False

<!-- String concatenation -->
{{ 'Hello, ' ~ name ~ '!' }}

<!-- Conditional expression (ternary) -->
{{ 'Admin' if user.is_admin else 'User' }}

<!-- In/not in -->
{{ 'python' in tags }}
{{ 'admin' not in roles }}
```

---

## 5. Template Inheritance

### ทำไมต้องใช้ Template Inheritance

Template inheritance ช่วยให้:
1. ไม่ต้องเขียน HTML ซ้ำซ้อน (DRY principle)
2. มี consistent layout ทั้ง site
3. แก้ไขได้ที่เดียว

### การสร้าง Base Template

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    {# Title block - child templates สามารถ override ได้ #}
    <title>{% block title %}My Website{% endblock %}</title>
    
    {# CSS block #}
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
    {% block extra_css %}{% endblock %}
</head>
<body>
    {# Navigation #}
    <nav>
        <a href="{{ url_for('main.index') }}">หน้าแรก</a>
        <a href="{{ url_for('main.about') }}">เกี่ยวกับ</a>
        <a href="{{ url_for('main.contact') }}">ติดต่อ</a>
        {% if current_user.is_authenticated %}
            <a href="{{ url_for('auth.logout') }}">ออกจากระบบ</a>
        {% else %}
            <a href="{{ url_for('auth.login') }}">เข้าสู่ระบบ</a>
        {% endif %}
    </nav>
    
    {# Flash messages #}
    {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
            {% for category, message in messages %}
                <div class="alert alert-{{ category }}">
                    {{ message }}
                </div>
            {% endfor %}
        {% endif %}
    {% endwith %}
    
    {# Main content block #}
    <main class="container">
        {% block content %}
        {# Child templates จะใส่ content ที่นี่ #}
        {% endblock %}
    </main>
    
    {# Footer #}
    <footer>
        <p>&copy; 2024 My Website</p>
    </footer>
    
    {# JavaScript #}
    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### Child Templates

```html
<!-- templates/index.html -->
{% extends 'base.html' %}

{% block title %}หน้าแรก - My Website{% endblock %}

{% block content %}
<h1>ยินดีต้อนรับ</h1>
<p>นี่คือหน้าแรกของเว็บไซต์</p>
{% endblock %}
```

```html
<!-- templates/user/profile.html -->
{% extends 'base.html' %}

{% block title %}โปรไฟล์ {{ user.name }} - My Website{% endblock %}

{% block extra_css %}
<link rel="stylesheet" href="{{ url_for('static', filename='css/profile.css') }}">
{% endblock %}

{% block content %}
<div class="profile-container">
    <img src="{{ user.avatar_url }}" alt="{{ user.name }}">
    <h1>{{ user.name }}</h1>
    <p>{{ user.bio }}</p>
    
    <h2>Posts ({{ user.posts|length }})</h2>
    {% for post in user.posts %}
    <div class="post-card">
        <h3>{{ post.title }}</h3>
        <p>{{ post.excerpt }}</p>
        <a href="{{ url_for('blog.post', slug=post.slug) }}">อ่านต่อ</a>
    </div>
    {% endfor %}
</div>
{% endblock %}

{% block extra_js %}
<script src="{{ url_for('static', filename='js/profile.js') }}"></script>
{% endblock %}
```

### Template Blocks และ super()

```html
<!-- templates/dashboard.html -->
{% extends 'base.html' %}

{% block title %}Dashboard - My Website{% endblock %}

{% block extra_css %}
{# เพิ่ม CSS ต่อจาก parent #}
{{ super() }}
<link rel="stylesheet" href="{{ url_for('static', filename='css/dashboard.css') }}">
{% endblock %}

{% block content %}
<div class="dashboard">
    <aside class="sidebar">
        {% block sidebar %}
        <ul>
            <li><a href="#">Overview</a></li>
            <li><a href="#">Analytics</a></li>
            <li><a href="#">Settings</a></li>
        </ul>
        {% endblock %}
    </aside>
    
    <main class="dashboard-content">
        {% block dashboard_content %}{% endblock %}
    </main>
</div>
{% endblock %}
```

```html
<!-- templates/dashboard/analytics.html -->
{% extends 'dashboard.html' %}

{% block dashboard_content %}
<h1>Analytics</h1>
<p>Charts และ graphs ที่นี่</p>
{% endblock %}
```

### Include Templates

```html
<!-- templates/components/navbar.html -->
<nav class="navbar">
    <a href="/">หน้าแรก</a>
    <a href="/about">เกี่ยวกับ</a>
</nav>
```

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<body>
    {# Include sub-template #}
    {% include 'components/navbar.html' %}
    
    {% block content %}{% endblock %}
    
    {# Include พร้อมจัดการ error #}
    {% include 'components/analytics.html' ignore missing %}
</body>
</html>
```

### Macros (Template Functions)

```html
<!-- templates/macros/forms.html -->
{% macro render_field(field, label_class="", field_class="") %}
<div class="form-group">
    <label class="{{ label_class }}" for="{{ field.id }}">
        {{ field.label.text }}
        {% if field.flags.required %}
        <span class="required">*</span>
        {% endif %}
    </label>
    {{ field(class=field_class) }}
    {% for error in field.errors %}
    <span class="error">{{ error }}</span>
    {% endfor %}
</div>
{% endmacro %}

{% macro render_button(text, type="submit", class="btn-primary") %}
<button type="{{ type }}" class="btn {{ class }}">
    {{ text }}
</button>
{% endmacro %}
```

```html
<!-- templates/register.html -->
{% extends 'base.html' %}
{% from 'macros/forms.html' import render_field, render_button %}

{% block content %}
<form method="post">
    {{ form.hidden_tag() }}
    {{ render_field(form.username, field_class="form-control") }}
    {{ render_field(form.email, field_class="form-control") }}
    {{ render_field(form.password, field_class="form-control") }}
    {{ render_button('สมัครสมาชิก') }}
</form>
{% endblock %}
```

---

## 6. Template Filters และ Tests

### Built-in Filters

```html
<!-- String filters -->
{{ name | upper }}           → SOMCHAI
{{ name | lower }}           → somchai
{{ name | title }}           → Somchai
{{ name | capitalize }}      → Somchai
{{ name | trim }}            → ตัด whitespace หัวท้าย
{{ name | strip }}           → เหมือน trim
{{ text | truncate(50) }}    → ตัดข้อความให้สั้นลง
{{ text | wordcount }}       → นับจำนวน words
{{ html | striptags }}       → ลบ HTML tags
{{ text | replace('a', 'b') }} → แทนที่ข้อความ

<!-- Number filters -->
{{ price | float }}          → แปลงเป็น float
{{ count | int }}            → แปลงเป็น int
{{ price | round(2) }}       → ปัดทศนิยม 2 ตำแหน่ง
{{ price | abs }}            → ค่าสัมบูรณ์

<!-- List/Dict filters -->
{{ items | length }}         → จำนวน items
{{ items | count }}          → เหมือน length
{{ items | first }}          → item แรก
{{ items | last }}           → item สุดท้าย
{{ items | sort }}           → เรียงลำดับ
{{ items | reverse }}        → กลับลำดับ
{{ items | list }}           → แปลงเป็น list
{{ items | join(', ') }}     → join ด้วย delimiter

{{ dict | items }}           → dict items
{{ dict | keys }}            → dict keys
{{ dict | values }}          → dict values

<!-- Type/Format filters -->
{{ value | default('N/A') }} → ค่า default ถ้า value เป็น None
{{ value | default('N/A', boolean=True) }}  → ถ้า value เป็น falsy
{{ html | safe }}            → render HTML ไม่ escape
{{ text | escape }}          → escape HTML characters
{{ text | e }}               → เหมือน escape

<!-- Date filters (ต้องการ custom filter หรือ library) -->
{{ date | strftime('%d/%m/%Y') }}

<!-- Formatting -->
{{ 1000000 | format_number }}  → 1,000,000
```

### Filter Chaining

```html
<!-- ใช้หลาย filters ต่อกัน -->
{{ title | trim | title | truncate(50) }}
{{ users | sort(attribute='name') | list }}
{{ text | striptags | truncate(100) | capitalize }}
```

### Jinja2 Tests

```html
<!-- Tests ใช้กับ 'is' keyword -->
{% if value is defined %}
{% if value is undefined %}
{% if value is none %}
{% if value is string %}
{% if value is number %}
{% if value is integer %}
{% if value is float %}
{% if value is sequence %}
{% if value is iterable %}
{% if value is mapping %}
{% if value is callable %}
{% if value is sameas other_value %}

<!-- ตัวอย่างการใช้งาน -->
{% if user is defined %}
    <p>สวัสดี, {{ user.name }}!</p>
{% endif %}

{% if age is number and age >= 18 %}
    <p>คุณบรรลุนิติภาวะแล้ว</p>
{% endif %}

{% for item in items %}
    {% if loop.index is odd %}
    <tr class="odd-row">
    {% else %}
    <tr class="even-row">
    {% endif %}
        <td>{{ item }}</td>
    </tr>
{% endfor %}
```

---

## 7. Custom Template Filters

### การสร้าง Custom Filter

```python
from flask import Flask
from datetime import datetime
import re

app = Flask(__name__)

# Decorator วิธีที่ 1
@app.template_filter('currency')
def currency_filter(value, currency='฿'):
    """แสดงตัวเลขเป็นรูปแบบสกุลเงิน"""
    return f"{currency}{value:,.2f}"

# Decorator วิธีที่ 2
@app.template_filter('timeago')
def timeago_filter(value):
    """แสดงเวลาในรูปแบบ relative (เช่น '2 ชั่วโมงที่แล้ว')"""
    now = datetime.utcnow()
    if isinstance(value, str):
        value = datetime.fromisoformat(value)
    
    diff = now - value
    seconds = diff.total_seconds()
    
    if seconds < 60:
        return 'เมื่อกี้'
    elif seconds < 3600:
        minutes = int(seconds / 60)
        return f'{minutes} นาทีที่แล้ว'
    elif seconds < 86400:
        hours = int(seconds / 3600)
        return f'{hours} ชั่วโมงที่แล้ว'
    elif seconds < 604800:
        days = int(seconds / 86400)
        return f'{days} วันที่แล้ว'
    else:
        return value.strftime('%d/%m/%Y')

# วิธีที่ 3: ใช้ app.jinja_env.filters
def highlight(text, query):
    """Highlight search term ใน text"""
    if not query:
        return text
    pattern = re.compile(re.escape(query), re.IGNORECASE)
    return pattern.sub(f'<mark>\\g<0></mark>', text)

app.jinja_env.filters['highlight'] = highlight

# ลงทะเบียนหลาย filters พร้อมกัน
def register_filters(app):
    @app.template_filter('phone')
    def phone_filter(value):
        """Format phone number: 0812345678 -> 081-234-5678"""
        if len(value) == 10:
            return f'{value[:3]}-{value[3:6]}-{value[6:]}'
        return value
    
    @app.template_filter('truncate_words')
    def truncate_words_filter(text, num_words=20):
        """ตัดข้อความตามจำนวน words"""
        words = text.split()
        if len(words) <= num_words:
            return text
        return ' '.join(words[:num_words]) + '...'
    
    @app.template_filter('nl2br')
    def nl2br_filter(text):
        """แปลง newline เป็น <br>"""
        return text.replace('\n', '<br>\n')
```

### Custom Template Global Functions

```python
from flask import Flask
from datetime import datetime

app = Flask(__name__)

# Global functions ใน templates
@app.template_global('now')
def get_current_time():
    return datetime.now()

# หรือ
app.jinja_env.globals['now'] = datetime.now

# Template context processor (ส่งตัวแปรให้ทุก template)
@app.context_processor
def inject_common_variables():
    return {
        'year': datetime.now().year,
        'site_name': 'My Website',
        'nav_items': [
            {'url': '/', 'label': 'หน้าแรก'},
            {'url': '/about', 'label': 'เกี่ยวกับ'},
            {'url': '/contact', 'label': 'ติดต่อ'},
        ]
    }
```

### การใช้ Custom Filters ใน Template

```html
<!-- ใช้ custom filters -->
<p>ราคา: {{ product.price | currency }}</p>
<p>ราคา USD: {{ product.price | currency('$') }}</p>
<p>เขียนเมื่อ: {{ post.created_at | timeago }}</p>
<p>เบอร์โทร: {{ user.phone | phone }}</p>
<p>{{ post.content | truncate_words(50) | nl2br | safe }}</p>
<p>{{ post.title | highlight(search_query) | safe }}</p>

<!-- ใช้ global function -->
<footer>© {{ now().year }} My Website</footer>

<!-- ใช้ context processor variables -->
<title>{{ site_name }}</title>
```

---

## 8. Static Files Management

### โครงสร้าง Static Files

```
project/
├── app/
│   ├── static/
│   │   ├── css/
│   │   │   ├── main.css
│   │   │   ├── bootstrap.min.css
│   │   │   └── custom.css
│   │   ├── js/
│   │   │   ├── main.js
│   │   │   ├── jquery.min.js
│   │   │   └── utils.js
│   │   ├── images/
│   │   │   ├── logo.png
│   │   │   └── background.jpg
│   │   ├── fonts/
│   │   │   └── sarabun.woff2
│   │   └── favicon.ico
│   └── templates/
└── run.py
```

### การใช้ Static Files ใน Templates

```html
<!-- ใช้ url_for() เพื่อสร้าง URL ของ static files -->

<!-- CSS -->
<link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">

<!-- JavaScript -->
<script src="{{ url_for('static', filename='js/main.js') }}"></script>

<!-- Images -->
<img src="{{ url_for('static', filename='images/logo.png') }}" alt="Logo">

<!-- Favicon -->
<link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}">

<!-- Dynamic path -->
<img src="{{ url_for('static', filename='images/' + user.avatar) }}" alt="Avatar">

<!-- URL จะเป็น: /static/css/main.css -->
```

### Static Files ใน Python Code

```python
from flask import Flask, url_for, send_from_directory
import os

app = Flask(__name__)

# URL ของ static files
with app.test_request_context():
    css_url = url_for('static', filename='css/main.css')
    # /static/css/main.css

# Serve static files จาก custom directory
@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(
        app.config['UPLOAD_FOLDER'],
        filename
    )

# Custom static folder
app = Flask(__name__, 
            static_folder='public',    # แทน 'static'
            static_url_path='/assets') # แทน '/static'
```

### Cache Busting สำหรับ Static Files

```python
from flask import Flask, url_for
import os
import hashlib

app = Flask(__name__)

def get_file_hash(filepath):
    """คำนวณ MD5 hash ของไฟล์"""
    with open(filepath, 'rb') as f:
        return hashlib.md5(f.read()).hexdigest()[:8]

@app.context_processor
def inject_static_version():
    def versioned_url(filename):
        """สร้าง URL พร้อม version hash"""
        static_path = os.path.join(app.static_folder, filename)
        if os.path.exists(static_path):
            file_hash = get_file_hash(static_path)
            return url_for('static', filename=filename) + f'?v={file_hash}'
        return url_for('static', filename=filename)
    
    return dict(versioned_url=versioned_url)

# ใน template
# <link rel="stylesheet" href="{{ versioned_url('css/main.css') }}">
# Output: /static/css/main.css?v=a1b2c3d4
```

---

## 9. url_for() Function

### พื้นฐาน url_for()

```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route('/')
def index():
    pass

@app.route('/user/<username>')
def user_profile(username):
    pass

@app.route('/post/<int:post_id>/edit')
def edit_post(post_id):
    pass

# ใน Python code (ต้องอยู่ใน request context หรือ test_request_context)
with app.test_request_context():
    # Route พื้นฐาน
    print(url_for('index'))               # /
    
    # Route ที่มี variable
    print(url_for('user_profile', username='somchai'))  # /user/somchai
    
    # Route ที่มีหลาย variables
    print(url_for('edit_post', post_id=42))  # /post/42/edit
    
    # เพิ่ม query string
    print(url_for('user_profile', 
                  username='somchai', 
                  page=2, 
                  tab='posts'))
    # /user/somchai?page=2&tab=posts
    
    # Absolute URL
    print(url_for('index', _external=True))
    # http://localhost:5000/
    
    # HTTPS
    print(url_for('index', _external=True, _scheme='https'))
    # https://localhost:5000/
    
    # Anchor
    print(url_for('index', _anchor='section1'))
    # /#section1
    
    # Static files
    print(url_for('static', filename='css/main.css'))
    # /static/css/main.css
```

### url_for() กับ Blueprints

```python
from flask import Flask, Blueprint, url_for

auth = Blueprint('auth', __name__, url_prefix='/auth')
main = Blueprint('main', __name__)

@auth.route('/login')
def login():
    pass

@main.route('/')
def index():
    pass

app = Flask(__name__)
app.register_blueprint(auth)
app.register_blueprint(main)

with app.test_request_context():
    # Blueprint routes ต้องใส่ blueprint_name.function_name
    print(url_for('auth.login'))    # /auth/login
    print(url_for('main.index'))    # /
    
    # ใน auth blueprint สามารถใช้ shortcut
    # url_for('.login')  # หมายถึง auth.login ถ้าอยู่ใน auth blueprint
```

### url_for() ใน Templates

```html
<!-- ใน Jinja2 templates -->
<a href="{{ url_for('index') }}">หน้าแรก</a>
<a href="{{ url_for('user_profile', username=user.username) }}">โปรไฟล์</a>
<a href="{{ url_for('auth.login') }}">เข้าสู่ระบบ</a>
<a href="{{ url_for('static', filename='css/style.css') }}">CSS</a>

<!-- กับ query parameters -->
<a href="{{ url_for('search', q='python', page=2) }}">หน้า 2</a>
```

---

## 10. Redirects

### HTTP Redirects

```python
from flask import Flask, redirect, url_for, request

app = Flask(__name__)

# Redirect พื้นฐาน (302 Temporary)
@app.route('/old-page')
def old_page():
    return redirect('/new-page')

# Redirect ด้วย url_for
@app.route('/dashboard')
def dashboard():
    if not is_authenticated():
        return redirect(url_for('auth.login'))
    return 'Dashboard'

# Redirect พร้อม status code
@app.route('/moved')
def moved():
    return redirect('/new-location', 301)  # Permanent redirect

# Redirect กลับไปยัง previous page
@app.route('/login')
def login():
    next_url = request.args.get('next', '/')
    # Login logic
    return redirect(next_url)

# Redirect ไปยัง external URL
@app.route('/github')
def github():
    return redirect('https://github.com')
```

### After Form Submission (PRG Pattern)

```python
from flask import Flask, redirect, url_for, request, flash

app = Flask(__name__)
app.secret_key = 'secret'

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form.get('username')
        email = request.form.get('email')
        
        # Process registration
        # ...
        
        flash('สมัครสมาชิกสำเร็จ!', 'success')
        
        # PRG (Post/Redirect/Get) pattern
        # Redirect หลัง POST เพื่อป้องกัน form resubmission
        return redirect(url_for('login'))
    
    return render_template('register.html')
```

---

## 11. Error Handlers

### Built-in Error Handlers

```python
from flask import Flask, jsonify, render_template, request

app = Flask(__name__)

# 400 Bad Request
@app.errorhandler(400)
def bad_request(error):
    if request.is_json or request.path.startswith('/api/'):
        return jsonify({'error': 'Bad Request', 'message': str(error)}), 400
    return render_template('errors/400.html', error=error), 400

# 401 Unauthorized
@app.errorhandler(401)
def unauthorized(error):
    if request.is_json:
        return jsonify({'error': 'Unauthorized', 'message': 'กรุณาเข้าสู่ระบบ'}), 401
    return redirect(url_for('auth.login', next=request.url))

# 403 Forbidden
@app.errorhandler(403)
def forbidden(error):
    return render_template('errors/403.html', error=error), 403

# 404 Not Found
@app.errorhandler(404)
def not_found(error):
    if request.path.startswith('/api/'):
        return jsonify({'error': 'Not Found', 'path': request.path}), 404
    return render_template('errors/404.html', error=error), 404

# 500 Internal Server Error
@app.errorhandler(500)
def internal_error(error):
    # Log error
    app.logger.error(f'Server Error: {error}')
    
    if request.is_json:
        return jsonify({'error': 'Internal Server Error'}), 500
    return render_template('errors/500.html', error=error), 500

# Handle Exception class
@app.errorhandler(Exception)
def handle_exception(error):
    """Catch-all error handler"""
    app.logger.exception('Unhandled exception')
    return render_template('errors/500.html'), 500
```

### Error Templates

```html
<!-- templates/errors/404.html -->
{% extends 'base.html' %}

{% block title %}404 - ไม่พบหน้าที่ต้องการ{% endblock %}

{% block content %}
<div class="error-page">
    <h1>404</h1>
    <h2>ไม่พบหน้าที่คุณต้องการ</h2>
    <p>ขออภัย หน้าที่คุณกำลังมองหาไม่มีอยู่หรือถูกย้ายไปแล้ว</p>
    <a href="{{ url_for('main.index') }}" class="btn btn-primary">
        กลับหน้าแรก
    </a>
</div>
{% endblock %}
```

### Custom Exception Classes

```python
from flask import Flask, jsonify
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

# Custom Exception
class APIException(Exception):
    def __init__(self, message, status_code=400, payload=None):
        super().__init__()
        self.message = message
        self.status_code = status_code
        self.payload = payload

class ValidationError(APIException):
    def __init__(self, message, field=None):
        super().__init__(message, 422)
        self.field = field

class NotFoundError(APIException):
    def __init__(self, resource='Resource'):
        super().__init__(f'{resource} not found', 404)

# Register handlers
@app.errorhandler(APIException)
def handle_api_exception(error):
    response = {
        'error': error.message,
        'status': error.status_code
    }
    if error.payload:
        response['details'] = error.payload
    return jsonify(response), error.status_code

@app.errorhandler(ValidationError)
def handle_validation_error(error):
    return jsonify({
        'error': error.message,
        'field': error.field,
        'status': 422
    }), 422

# ใช้งาน
@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user = User.get(user_id)
    if not user:
        raise NotFoundError('User')
    return jsonify(user)

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data.get('email'):
        raise ValidationError('Email is required', 'email')
    # ...
```

---

## 12. Flash Messages

### Flash Messages คืออะไร

Flash messages คือ one-time messages ที่ส่งจาก server ไปยัง user หลัง redirect มักใช้สำหรับ:
- แสดงผลการทำงาน (success, error)
- แจ้งเตือน
- ข้อความสำคัญ

### การใช้ Flash Messages

```python
from flask import Flask, flash, redirect, url_for, render_template, request, get_flashed_messages

app = Flask(__name__)
app.secret_key = 'your-secret-key'  # จำเป็นสำหรับ session

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        
        # ตรวจสอบ credentials
        if username == 'admin' and password == 'secret':
            flash('เข้าสู่ระบบสำเร็จ!', 'success')
            return redirect(url_for('dashboard'))
        else:
            flash('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง', 'error')
            flash('ลองอีกครั้ง', 'warning')
    
    return render_template('login.html')

@app.route('/profile/update', methods=['POST'])
def update_profile():
    try:
        # อัพเดทข้อมูล
        flash('อัพเดทข้อมูลสำเร็จ', 'success')
    except Exception as e:
        flash(f'เกิดข้อผิดพลาด: {str(e)}', 'danger')
    
    return redirect(url_for('profile'))

# ดึง flash messages โดยตรง
@app.route('/messages')
def show_messages():
    messages = get_flashed_messages(with_categories=True)
    return render_template('messages.html', messages=messages)
```

### Flash Messages ใน Templates

```html
<!-- templates/base.html -->
{% with messages = get_flashed_messages(with_categories=true) %}
    {% if messages %}
    <div class="flash-messages">
        {% for category, message in messages %}
        <div class="alert alert-{{ category }} alert-dismissible">
            {{ message }}
            <button type="button" class="close" data-dismiss="alert">&times;</button>
        </div>
        {% endfor %}
    </div>
    {% endif %}
{% endwith %}

<!-- หรือแสดงเฉพาะ category -->
{% with errors = get_flashed_messages(category_filter=['error', 'danger']) %}
    {% for error in errors %}
    <div class="alert alert-danger">{{ error }}</div>
    {% endfor %}
{% endwith %}
```

---

## 13. Response Objects

### สร้าง Response Objects

```python
from flask import Flask, Response, make_response, jsonify

app = Flask(__name__)

# วิธีที่ 1: String (Flask แปลงให้อัตโนมัติ)
@app.route('/1')
def response1():
    return 'Hello World'  # 200 OK, text/html

# วิธีที่ 2: Tuple
@app.route('/2')
def response2():
    return 'Created', 201, {'Content-Type': 'text/plain'}

# วิธีที่ 3: make_response()
@app.route('/3')
def response3():
    response = make_response('Custom Response')
    response.status_code = 202
    response.headers['X-Custom'] = 'value'
    response.set_cookie('key', 'value')
    return response

# วิธีที่ 4: Response class
@app.route('/4')
def response4():
    return Response(
        '{"key": "value"}',
        status=200,
        mimetype='application/json',
        headers={'X-Custom': 'value'}
    )

# วิธีที่ 5: jsonify()
@app.route('/5')
def response5():
    return jsonify({'key': 'value'})

# วิธีที่ 6: Generator (streaming response)
@app.route('/stream')
def stream():
    def generate():
        for i in range(100):
            yield f'data: {i}\n\n'
    
    return Response(generate(), mimetype='text/event-stream')
```

### Response Headers และ Cookies

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/with-cookie')
def with_cookie():
    response = make_response('Cookie set!')
    
    # Set cookie
    response.set_cookie(
        'user_id',            # cookie name
        '123',                # cookie value
        max_age=3600,         # expire in seconds
        expires=None,         # datetime object
        path='/',             # cookie path
        domain=None,          # cookie domain
        secure=False,         # HTTPS only
        httponly=True,        # ไม่ให้ JS อ่าน
        samesite='Lax'        # SameSite policy
    )
    
    # Delete cookie
    response.delete_cookie('old_cookie')
    
    # Set headers
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    
    return response

# Custom Response types
@app.route('/download')
def download_file():
    """ส่ง file download"""
    from flask import send_file
    return send_file(
        'files/document.pdf',
        as_attachment=True,
        download_name='my-document.pdf'
    )

@app.route('/csv')
def download_csv():
    """ส่ง CSV response"""
    import io
    import csv
    
    output = io.StringIO()
    writer = csv.writer(output)
    writer.writerow(['ชื่อ', 'อีเมล', 'อายุ'])
    writer.writerow(['สมชาย', 'somchai@example.com', '25'])
    
    response = make_response(output.getvalue())
    response.headers['Content-Type'] = 'text/csv; charset=utf-8-sig'
    response.headers['Content-Disposition'] = 'attachment; filename=users.csv'
    return response
```

---

## 14. ตัวอย่างโปรแกรมจริง

### ตัวอย่างที่ 1: Personal Website

```python
# personal_website/app.py

from flask import Flask, render_template_string, url_for
from datetime import datetime

app = Flask(__name__)

# Data
PROFILE = {
    'name': 'สมชาย ใจดี',
    'title': 'Full Stack Developer',
    'bio': 'นักพัฒนา Web Application ที่มีประสบการณ์ 5 ปี',
    'email': 'somchai@example.com',
    'github': 'https://github.com/somchai',
    'linkedin': 'https://linkedin.com/in/somchai',
    'skills': ['Python', 'Flask', 'Django', 'JavaScript', 'React', 'PostgreSQL'],
    'avatar': 'https://via.placeholder.com/150'
}

PROJECTS = [
    {
        'id': 1,
        'title': 'E-commerce Platform',
        'description': 'ระบบ e-commerce ครบวงจรสร้างด้วย Django และ React',
        'tech': ['Python', 'Django', 'React', 'PostgreSQL'],
        'url': 'https://github.com/somchai/ecommerce',
        'demo': 'https://demo.example.com'
    },
    {
        'id': 2,
        'title': 'Task Management API',
        'description': 'REST API สำหรับจัดการ tasks สร้างด้วย Flask',
        'tech': ['Python', 'Flask', 'SQLAlchemy'],
        'url': 'https://github.com/somchai/task-api',
        'demo': None
    },
    {
        'id': 3,
        'title': 'Data Analytics Dashboard',
        'description': 'Dashboard แสดงผลข้อมูลแบบ real-time',
        'tech': ['Python', 'Pandas', 'Plotly', 'Flask'],
        'url': 'https://github.com/somchai/dashboard',
        'demo': 'https://dashboard.example.com'
    }
]

BLOG_POSTS = [
    {
        'id': 1,
        'title': 'เริ่มต้น Flask Web Development',
        'excerpt': 'บทความแนะนำการสร้าง web application ด้วย Flask framework',
        'date': datetime(2024, 1, 15),
        'tags': ['Python', 'Flask', 'Tutorial'],
        'read_time': 8
    },
    {
        'id': 2,
        'title': 'PostgreSQL Performance Tuning',
        'excerpt': 'เทคนิคการปรับประสิทธิภาพ PostgreSQL database',
        'date': datetime(2024, 2, 20),
        'tags': ['Database', 'PostgreSQL', 'Performance'],
        'read_time': 12
    }
]

BASE_HTML = """
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}{{ profile.name }}{% endblock %}</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Sarabun', Arial, sans-serif; color: #333; background: #f5f5f5; }
        .container { max-width: 900px; margin: 0 auto; padding: 20px; }
        nav { background: #2c3e50; padding: 15px 0; }
        nav .container { display: flex; justify-content: space-between; align-items: center; }
        nav a { color: white; text-decoration: none; margin: 0 15px; }
        nav a:hover { color: #3498db; }
        .hero { background: linear-gradient(135deg, #2c3e50, #3498db); color: white; padding: 80px 0; text-align: center; }
        .hero img { border-radius: 50%; border: 4px solid white; margin-bottom: 20px; }
        .hero h1 { font-size: 2.5em; margin-bottom: 10px; }
        .hero p { font-size: 1.2em; opacity: 0.9; }
        section { padding: 60px 0; }
        section:nth-child(even) { background: white; }
        h2 { font-size: 2em; margin-bottom: 30px; color: #2c3e50; border-bottom: 3px solid #3498db; display: inline-block; padding-bottom: 5px; }
        .skills { display: flex; flex-wrap: wrap; gap: 10px; }
        .skill-tag { background: #3498db; color: white; padding: 8px 16px; border-radius: 20px; font-size: 0.9em; }
        .project-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
        .project-card { background: white; border-radius: 8px; padding: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); border-top: 3px solid #3498db; }
        .project-card h3 { margin-bottom: 10px; }
        .project-card p { color: #666; margin-bottom: 15px; font-size: 0.9em; }
        .tech-stack { display: flex; flex-wrap: wrap; gap: 5px; margin-bottom: 15px; }
        .tech-tag { background: #ecf0f1; padding: 4px 10px; border-radius: 12px; font-size: 0.8em; }
        .btn { display: inline-block; padding: 8px 16px; border-radius: 4px; text-decoration: none; font-size: 0.9em; margin-right: 8px; }
        .btn-primary { background: #3498db; color: white; }
        .btn-secondary { background: #ecf0f1; color: #333; }
        .blog-list { space-y: 20px; }
        .blog-item { background: white; padding: 20px; border-radius: 8px; margin-bottom: 15px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); display: flex; justify-content: space-between; align-items: center; }
        .blog-tags span { background: #e8f4f8; color: #3498db; padding: 3px 8px; border-radius: 12px; font-size: 0.8em; margin-right: 5px; }
        footer { background: #2c3e50; color: white; padding: 30px 0; text-align: center; }
        .social-links a { color: #3498db; text-decoration: none; margin: 0 10px; }
    </style>
</head>
<body>
    <nav>
        <div class="container">
            <strong style="color:white">{{ profile.name }}</strong>
            <div>
                <a href="/">หน้าแรก</a>
                <a href="/projects">โปรเจค</a>
                <a href="/blog">บทความ</a>
                <a href="/contact">ติดต่อ</a>
            </div>
        </div>
    </nav>
    
    {% block content %}{% endblock %}
    
    <footer>
        <div class="container">
            <div class="social-links">
                <a href="{{ profile.github }}">GitHub</a>
                <a href="{{ profile.linkedin }}">LinkedIn</a>
                <a href="mailto:{{ profile.email }}">Email</a>
            </div>
            <p style="margin-top: 15px; opacity: 0.7">© {{ 2024 }} {{ profile.name }}</p>
        </div>
    </footer>
</body>
</html>
"""

HOME_HTML = BASE_HTML.replace(
    "{% block content %}{% endblock %}",
    """
    <div class="hero">
        <div class="container">
            <img src="{{ profile.avatar }}" alt="{{ profile.name }}" width="150">
            <h1>{{ profile.name }}</h1>
            <p>{{ profile.title }}</p>
            <p style="margin-top:10px; opacity:0.8">{{ profile.bio }}</p>
        </div>
    </div>
    
    <section>
        <div class="container">
            <h2>Skills</h2>
            <div class="skills">
                {% for skill in profile.skills %}
                <span class="skill-tag">{{ skill }}</span>
                {% endfor %}
            </div>
        </div>
    </section>
    
    <section>
        <div class="container">
            <h2>โปรเจคล่าสุด</h2>
            <div class="project-grid">
                {% for project in projects[:2] %}
                <div class="project-card">
                    <h3>{{ project.title }}</h3>
                    <p>{{ project.description }}</p>
                    <div class="tech-stack">
                        {% for tech in project.tech %}
                        <span class="tech-tag">{{ tech }}</span>
                        {% endfor %}
                    </div>
                    <a href="{{ project.url }}" class="btn btn-primary">GitHub</a>
                    {% if project.demo %}
                    <a href="{{ project.demo }}" class="btn btn-secondary">Demo</a>
                    {% endif %}
                </div>
                {% endfor %}
            </div>
            <div style="margin-top:20px">
                <a href="/projects" class="btn btn-primary">ดูทั้งหมด</a>
            </div>
        </div>
    </section>
    """
)

@app.route('/')
def index():
    return render_template_string(HOME_HTML, 
                                  profile=PROFILE, 
                                  projects=PROJECTS)

@app.route('/projects')
def projects():
    html = BASE_HTML.replace(
        "{% block content %}{% endblock %}",
        """
        <section>
            <div class="container">
                <h2>โปรเจคทั้งหมด</h2>
                <div class="project-grid">
                    {% for project in projects %}
                    <div class="project-card">
                        <h3>{{ project.title }}</h3>
                        <p>{{ project.description }}</p>
                        <div class="tech-stack">
                            {% for tech in project.tech %}
                            <span class="tech-tag">{{ tech }}</span>
                            {% endfor %}
                        </div>
                        <a href="{{ project.url }}" class="btn btn-primary">GitHub</a>
                        {% if project.demo %}
                        <a href="{{ project.demo }}" class="btn btn-secondary">Demo</a>
                        {% endif %}
                    </div>
                    {% endfor %}
                </div>
            </div>
        </section>
        """
    )
    return render_template_string(html, profile=PROFILE, projects=PROJECTS)

@app.route('/blog')
def blog():
    html = BASE_HTML.replace(
        "{% block content %}{% endblock %}",
        """
        <section>
            <div class="container">
                <h2>บทความ</h2>
                {% for post in posts %}
                <div class="blog-item">
                    <div>
                        <h3><a href="/blog/{{ post.id }}" style="text-decoration:none;color:#2c3e50">{{ post.title }}</a></h3>
                        <p style="color:#666;margin:8px 0">{{ post.excerpt }}</p>
                        <div class="blog-tags">
                            {% for tag in post.tags %}
                            <span>{{ tag }}</span>
                            {% endfor %}
                        </div>
                    </div>
                    <div style="text-align:right;min-width:120px">
                        <small>{{ post.date.strftime('%d/%m/%Y') }}</small><br>
                        <small>{{ post.read_time }} นาที</small>
                    </div>
                </div>
                {% endfor %}
            </div>
        </section>
        """
    )
    return render_template_string(html, profile=PROFILE, posts=BLOG_POSTS)

@app.route('/contact')
def contact():
    html = BASE_HTML.replace(
        "{% block content %}{% endblock %}",
        """
        <section>
            <div class="container" style="max-width:600px">
                <h2>ติดต่อเรา</h2>
                <p>Email: <a href="mailto:{{ profile.email }}">{{ profile.email }}</a></p>
                <p>GitHub: <a href="{{ profile.github }}">{{ profile.github }}</a></p>
                <p>LinkedIn: <a href="{{ profile.linkedin }}">{{ profile.linkedin }}</a></p>
            </div>
        </section>
        """
    )
    return render_template_string(html, profile=PROFILE)

@app.errorhandler(404)
def not_found(e):
    html = BASE_HTML.replace(
        "{% block content %}{% endblock %}",
        """
        <section style="text-align:center;padding:100px 0">
            <div class="container">
                <h1 style="font-size:6em;color:#3498db">404</h1>
                <h2>ไม่พบหน้าที่ต้องการ</h2>
                <a href="/" class="btn btn-primary" style="margin-top:20px">กลับหน้าแรก</a>
            </div>
        </section>
        """
    )
    return render_template_string(html, profile=PROFILE), 404

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 15. แบบฝึกหัด

### ข้อที่ 1: URL Converter ที่กำหนดเอง

สร้าง URL converter ชื่อ `date` ที่รับ format YYYY-MM-DD และแปลงเป็น datetime object

**เฉลย:**

```python
from flask import Flask
from werkzeug.routing import BaseConverter
from datetime import datetime

app = Flask(__name__)

class DateConverter(BaseConverter):
    regex = r'\d{4}-\d{2}-\d{2}'
    
    def to_python(self, value):
        try:
            return datetime.strptime(value, '%Y-%m-%d').date()
        except ValueError:
            raise ValueError(f'Invalid date: {value}')
    
    def to_url(self, value):
        if isinstance(value, str):
            return value
        return value.strftime('%Y-%m-%d')

app.url_map.converters['date'] = DateConverter

@app.route('/events/<date:event_date>')
def events_on_date(event_date):
    return f'Events on {event_date.strftime("%d %B %Y")}'

with app.test_request_context():
    from flask import url_for
    from datetime import date
    print(url_for('events_on_date', event_date=date(2024, 1, 15)))
    # /events/2024-01-15

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 2: Custom Template Filters

สร้าง custom filters สำหรับ:
1. `thai_date` - แสดงวันที่เป็นภาษาไทย
2. `mask_email` - ซ่อนบางส่วนของ email

**เฉลย:**

```python
from flask import Flask, render_template_string
from datetime import datetime

app = Flask(__name__)

THAI_MONTHS = {
    1: 'มกราคม', 2: 'กุมภาพันธ์', 3: 'มีนาคม',
    4: 'เมษายน', 5: 'พฤษภาคม', 6: 'มิถุนายน',
    7: 'กรกฎาคม', 8: 'สิงหาคม', 9: 'กันยายน',
    10: 'ตุลาคม', 11: 'พฤศจิกายน', 12: 'ธันวาคม'
}

@app.template_filter('thai_date')
def thai_date_filter(value):
    if isinstance(value, str):
        value = datetime.fromisoformat(value)
    thai_year = value.year + 543
    return f'{value.day} {THAI_MONTHS[value.month]} {thai_year}'

@app.template_filter('mask_email')
def mask_email_filter(email):
    if '@' not in email:
        return email
    name, domain = email.split('@', 1)
    if len(name) <= 2:
        masked_name = name[0] + '*'
    else:
        masked_name = name[0] + '*' * (len(name) - 2) + name[-1]
    return f'{masked_name}@{domain}'

@app.route('/test-filters')
def test_filters():
    template = """
    <p>วันที่: {{ date | thai_date }}</p>
    <p>Email: {{ email | mask_email }}</p>
    """
    return render_template_string(template,
                                  date=datetime(2024, 6, 15),
                                  email='somchai@example.com')

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 3: Template Inheritance

สร้าง blog layout ด้วย template inheritance:
- `base.html`: layout หลัก
- `blog/index.html`: หน้ารายการบทความ
- `blog/post.html`: หน้าแสดงบทความ

**เฉลย:**

```python
from flask import Flask, render_template_string

app = Flask(__name__)

# สร้าง templates ในรูปแบบ strings สำหรับตัวอย่าง
# ในโปรเจคจริงควรแยกเป็น files ในโฟลเดอร์ templates/

POSTS_DB = [
    {'id': 1, 'title': 'เรียน Python', 'content': 'Python เป็นภาษาโปรแกรมที่ยอดเยี่ยม...', 
     'author': 'สมชาย', 'date': '2024-01-15', 'tags': ['Python', 'Tutorial']},
    {'id': 2, 'title': 'Flask สำหรับมือใหม่', 'content': 'Flask คือ micro web framework...', 
     'author': 'สมหญิง', 'date': '2024-02-20', 'tags': ['Flask', 'Web']},
]

@app.route('/blog')
def blog_index():
    html = """
    <!DOCTYPE html>
    <html lang="th">
    <head>
        <meta charset="UTF-8">
        <title>Blog - My Site</title>
        <style>
            body { font-family: Arial; max-width: 800px; margin: 40px auto; padding: 20px; }
            .post-card { border: 1px solid #ddd; padding: 20px; margin: 15px 0; border-radius: 8px; }
            .tag { background: #007bff; color: white; padding: 3px 8px; border-radius: 12px; font-size: 0.8em; }
            a { color: #007bff; text-decoration: none; }
        </style>
    </head>
    <body>
        <h1>บทความทั้งหมด ({{ posts|length }} บทความ)</h1>
        {% for post in posts %}
        <div class="post-card">
            <h2><a href="/blog/{{ post.id }}">{{ post.title }}</a></h2>
            <p>โดย {{ post.author }} | {{ post.date }}</p>
            <p>{{ post.content[:100] }}...</p>
            <div>
                {% for tag in post.tags %}
                <span class="tag">{{ tag }}</span>
                {% endfor %}
            </div>
        </div>
        {% endfor %}
    </body>
    </html>
    """
    return render_template_string(html, posts=POSTS_DB)

@app.route('/blog/<int:post_id>')
def blog_post(post_id):
    post = next((p for p in POSTS_DB if p['id'] == post_id), None)
    if not post:
        return 'ไม่พบบทความ', 404
    
    html = """
    <!DOCTYPE html>
    <html lang="th">
    <head>
        <meta charset="UTF-8">
        <title>{{ post.title }} - Blog</title>
    </head>
    <body style="max-width:800px;margin:40px auto;padding:20px;font-family:Arial">
        <a href="/blog">← กลับ</a>
        <h1>{{ post.title }}</h1>
        <p>โดย <strong>{{ post.author }}</strong> | {{ post.date }}</p>
        <hr>
        <p>{{ post.content }}</p>
        <div>
            {% for tag in post.tags %}
            <span style="background:#007bff;color:white;padding:3px 8px;border-radius:12px;font-size:0.8em;margin-right:5px">{{ tag }}</span>
            {% endfor %}
        </div>
    </body>
    </html>
    """
    return render_template_string(html, post=post)

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 4: Flash Messages

สร้าง todo app ที่ใช้ flash messages แจ้งผลลัพธ์ทุกการกระทำ

**เฉลย:**

```python
from flask import Flask, flash, redirect, url_for, request, render_template_string

app = Flask(__name__)
app.secret_key = 'flash-demo-secret'

todos = []
next_id = 1

HTML = """
<!DOCTYPE html>
<html lang="th">
<head><meta charset="UTF-8"><title>Todo</title>
<style>
body{font-family:Arial;max-width:600px;margin:40px auto;padding:20px}
.alert{padding:10px;margin:10px 0;border-radius:4px}
.alert-success{background:#d4edda;color:#155724;border:1px solid #c3e6cb}
.alert-error{background:#f8d7da;color:#721c24;border:1px solid #f5c6cb}
.todo{display:flex;justify-content:space-between;align-items:center;padding:10px;border:1px solid #ddd;margin:5px 0;border-radius:4px}
.done{text-decoration:line-through;color:#999}
input{padding:8px;width:70%;border:1px solid #ddd;border-radius:4px}
button{padding:8px 16px;border:none;border-radius:4px;cursor:pointer}
.btn-add{background:#007bff;color:white}
.btn-done{background:#28a745;color:white;font-size:0.8em}
.btn-del{background:#dc3545;color:white;font-size:0.8em}
</style></head>
<body>
<h1>Todo List</h1>

{% with messages = get_flashed_messages(with_categories=true) %}
{% if messages %}
{% for category, message in messages %}
<div class="alert alert-{{ category }}">{{ message }}</div>
{% endfor %}
{% endif %}
{% endwith %}

<form method="post" action="/add">
<input name="title" placeholder="เพิ่ม todo..." required>
<button type="submit" class="btn-add">เพิ่ม</button>
</form>

<div style="margin-top:20px">
{% for todo in todos %}
<div class="todo">
<span class="{{ 'done' if todo.done else '' }}">{{ todo.title }}</span>
<div>
<form action="/toggle/{{ todo.id }}" method="post" style="display:inline">
<button class="btn-done">{{ '↩' if todo.done else '✓' }}</button>
</form>
<form action="/delete/{{ todo.id }}" method="post" style="display:inline">
<button class="btn-del">✕</button>
</form>
</div>
</div>
{% else %}
<p style="color:#999">ไม่มี todo</p>
{% endfor %}
</div>
</body></html>
"""

@app.route('/')
def index():
    return render_template_string(HTML, todos=todos)

@app.route('/add', methods=['POST'])
def add():
    global next_id
    title = request.form.get('title', '').strip()
    if title:
        todos.append({'id': next_id, 'title': title, 'done': False})
        next_id += 1
        flash(f'เพิ่ม "{title}" สำเร็จ!', 'success')
    else:
        flash('กรุณาระบุชื่อ todo', 'error')
    return redirect(url_for('index'))

@app.route('/toggle/<int:todo_id>', methods=['POST'])
def toggle(todo_id):
    todo = next((t for t in todos if t['id'] == todo_id), None)
    if todo:
        todo['done'] = not todo['done']
        status = 'เสร็จแล้ว' if todo['done'] else 'ยังไม่เสร็จ'
        flash(f'เปลี่ยนสถานะ "{todo["title"]}" เป็น {status}', 'success')
    return redirect(url_for('index'))

@app.route('/delete/<int:todo_id>', methods=['POST'])
def delete(todo_id):
    global todos
    todo = next((t for t in todos if t['id'] == todo_id), None)
    if todo:
        todos = [t for t in todos if t['id'] != todo_id]
        flash(f'ลบ "{todo["title"]}" สำเร็จ', 'success')
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 5: Error Handler ครบถ้วน

สร้าง app ที่มี custom error handlers สวยงามสำหรับ 404 และ 500

**เฉลย:**

```python
from flask import Flask, render_template_string, jsonify, request, abort

app = Flask(__name__)

ERROR_BASE = """
<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>Error {{ code }}</title>
<style>
body{font-family:Arial;display:flex;justify-content:center;align-items:center;min-height:100vh;margin:0;background:#f5f5f5}
.error-box{text-align:center;padding:60px;background:white;border-radius:12px;box-shadow:0 4px 20px rgba(0,0,0,0.1);max-width:500px}
.error-code{font-size:6em;font-weight:bold;color:{{ color }};margin:0}
h2{color:#333;margin:10px 0}
p{color:#666;margin:15px 0}
a{background:{{ color }};color:white;padding:10px 24px;border-radius:6px;text-decoration:none;display:inline-block;margin-top:10px}
</style>
</head>
<body>
<div class="error-box">
    <p class="error-code">{{ code }}</p>
    <h2>{{ title }}</h2>
    <p>{{ message }}</p>
    <a href="/">กลับหน้าแรก</a>
</div>
</body>
</html>
"""

@app.errorhandler(404)
def not_found(e):
    if request.path.startswith('/api/'):
        return jsonify({'error': 'Not Found', 'path': request.path}), 404
    return render_template_string(ERROR_BASE,
        code=404,
        title='ไม่พบหน้าที่ต้องการ',
        message=f'ไม่พบ: {request.path}',
        color='#3498db'
    ), 404

@app.errorhandler(403)
def forbidden(e):
    return render_template_string(ERROR_BASE,
        code=403,
        title='ไม่มีสิทธิ์เข้าถึง',
        message='คุณไม่มีสิทธิ์เข้าถึงหน้านี้',
        color='#e74c3c'
    ), 403

@app.errorhandler(500)
def server_error(e):
    app.logger.error(f'Server error: {e}')
    return render_template_string(ERROR_BASE,
        code=500,
        title='เกิดข้อผิดพลาด',
        message='เซิร์ฟเวอร์เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง',
        color='#e74c3c'
    ), 500

@app.route('/')
def index():
    return 'Home Page - ลอง /forbidden หรือ /error เพื่อทดสอบ'

@app.route('/forbidden')
def test_forbidden():
    abort(403)

@app.route('/error')
def test_error():
    raise Exception('Test error')

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 6: Static Files Server

สร้าง mini static file server ที่ serve images และไฟล์ต่างๆ

**เฉลย:**

```python
import os
from flask import Flask, send_from_directory, jsonify, abort, url_for

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'uploads'

# สร้าง folder ถ้ายังไม่มี
os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)

@app.route('/files/<filename>')
def serve_file(filename):
    """Serve file จาก uploads folder"""
    try:
        return send_from_directory(
            app.config['UPLOAD_FOLDER'],
            filename,
            as_attachment=False
        )
    except FileNotFoundError:
        abort(404)

@app.route('/download/<filename>')
def download_file(filename):
    """Force download"""
    return send_from_directory(
        app.config['UPLOAD_FOLDER'],
        filename,
        as_attachment=True
    )

@app.route('/files')
def list_files():
    """รายการไฟล์ทั้งหมด"""
    try:
        files = os.listdir(app.config['UPLOAD_FOLDER'])
        file_info = []
        for filename in files:
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
            file_info.append({
                'name': filename,
                'size': os.path.getsize(filepath),
                'url': url_for('serve_file', filename=filename, _external=True),
                'download_url': url_for('download_file', filename=filename, _external=True)
            })
        return jsonify({'files': file_info, 'count': len(file_info)})
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 7: Response ประเภทต่างๆ

สร้าง API ที่ส่ง response ประเภทต่างๆ: JSON, XML, CSV, HTML

**เฉลย:**

```python
from flask import Flask, Response, jsonify, make_response
import json
import csv
import io

app = Flask(__name__)

DATA = [
    {'id': 1, 'name': 'สินค้า A', 'price': 99.99},
    {'id': 2, 'name': 'สินค้า B', 'price': 149.50},
    {'id': 3, 'name': 'สินค้า C', 'price': 299.00},
]

@app.route('/data/json')
def data_json():
    return jsonify({'products': DATA})

@app.route('/data/xml')
def data_xml():
    xml = '<?xml version="1.0" encoding="UTF-8"?>\n<products>\n'
    for item in DATA:
        xml += f'  <product>\n'
        xml += f'    <id>{item["id"]}</id>\n'
        xml += f'    <name>{item["name"]}</name>\n'
        xml += f'    <price>{item["price"]}</price>\n'
        xml += f'  </product>\n'
    xml += '</products>'
    return Response(xml, mimetype='application/xml')

@app.route('/data/csv')
def data_csv():
    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=['id', 'name', 'price'])
    writer.writeheader()
    writer.writerows(DATA)
    
    response = make_response(output.getvalue())
    response.headers['Content-Type'] = 'text/csv; charset=utf-8'
    response.headers['Content-Disposition'] = 'attachment; filename=products.csv'
    return response

@app.route('/data/html')
def data_html():
    rows = ''.join(f'<tr><td>{d["id"]}</td><td>{d["name"]}</td><td>{d["price"]:.2f}</td></tr>'
                   for d in DATA)
    html = f"""
    <table border="1">
        <thead><tr><th>ID</th><th>ชื่อ</th><th>ราคา</th></tr></thead>
        <tbody>{rows}</tbody>
    </table>
    """
    return Response(html, mimetype='text/html')

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 8: Content Website

สร้าง content website ที่มี categories และ articles พร้อม search functionality

**เฉลย:**

```python
from flask import Flask, render_template_string, request, redirect, url_for

app = Flask(__name__)

ARTICLES = [
    {'id': 1, 'title': 'Python Basics', 'category': 'python', 
     'content': 'Python เป็นภาษา high-level ที่ใช้งานง่าย', 'views': 100},
    {'id': 2, 'title': 'Flask Tutorial', 'category': 'flask',
     'content': 'Flask คือ micro web framework สำหรับ Python', 'views': 200},
    {'id': 3, 'title': 'SQLAlchemy Guide', 'category': 'database',
     'content': 'SQLAlchemy เป็น ORM ที่ทรงพลังสำหรับ Python', 'views': 150},
    {'id': 4, 'title': 'Advanced Python', 'category': 'python',
     'content': 'เรียนรู้ decorators, generators, และ metaclasses', 'views': 80},
]

TEMPLATE = """
<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8"><title>Content Site</title>
<style>
body{font-family:Arial;max-width:900px;margin:0 auto;padding:20px;background:#f5f5f5}
.header{background:#2c3e50;color:white;padding:20px;border-radius:8px;margin-bottom:20px;display:flex;justify-content:space-between;align-items:center}
.header a{color:white;text-decoration:none}
.search{display:flex;gap:10px}
.search input{padding:8px;border:none;border-radius:4px;flex:1}
.search button{padding:8px 16px;background:#3498db;color:white;border:none;border-radius:4px;cursor:pointer}
.categories{display:flex;gap:10px;margin-bottom:20px;flex-wrap:wrap}
.cat-btn{padding:6px 14px;border-radius:20px;text-decoration:none;background:white;border:1px solid #ddd;color:#333}
.cat-btn.active,.cat-btn:hover{background:#3498db;color:white;border-color:#3498db}
.articles{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:15px}
.article{background:white;border-radius:8px;padding:20px;box-shadow:0 2px 4px rgba(0,0,0,0.1)}
.article h3{margin:0 0 10px}
.article h3 a{color:#2c3e50;text-decoration:none}
.article p{color:#666;font-size:0.9em}
.cat-tag{background:#e8f4f8;color:#3498db;padding:3px 8px;border-radius:12px;font-size:0.8em}
.views{color:#999;font-size:0.8em}
.no-results{text-align:center;padding:40px;color:#666}
</style>
</head>
<body>
<div class="header">
    <a href="/"><h2 style="margin:0">📚 Content Site</h2></a>
    <form class="search" action="/" method="get">
        <input name="q" value="{{ q }}" placeholder="ค้นหา...">
        <button type="submit">ค้นหา</button>
    </form>
</div>

<div class="categories">
    <a href="/" class="cat-btn {{ 'active' if not selected_cat else '' }}">ทั้งหมด ({{ total_count }})</a>
    {% for cat, count in categories.items() %}
    <a href="/?cat={{ cat }}" class="cat-btn {{ 'active' if selected_cat == cat else '' }}">{{ cat }} ({{ count }})</a>
    {% endfor %}
</div>

{% if q %}
<p>ผลการค้นหา "{{ q }}": {{ articles|length }} บทความ</p>
{% endif %}

{% if articles %}
<div class="articles">
{% for article in articles %}
<div class="article">
    <h3><a href="/article/{{ article.id }}">{{ article.title }}</a></h3>
    <p>{{ article.content[:80] }}...</p>
    <div style="display:flex;justify-content:space-between;align-items:center;margin-top:10px">
        <span class="cat-tag">{{ article.category }}</span>
        <span class="views">👁 {{ article.views }}</span>
    </div>
</div>
{% endfor %}
</div>
{% else %}
<div class="no-results">
    <p>ไม่พบบทความที่ตรงกับการค้นหา</p>
    <a href="/">ดูทั้งหมด</a>
</div>
{% endif %}
</body>
</html>
"""

@app.route('/')
def index():
    q = request.args.get('q', '')
    selected_cat = request.args.get('cat', '')
    
    filtered = ARTICLES
    if q:
        filtered = [a for a in filtered 
                   if q.lower() in a['title'].lower() or q.lower() in a['content'].lower()]
    if selected_cat:
        filtered = [a for a in filtered if a['category'] == selected_cat]
    
    cats = {}
    for a in ARTICLES:
        cats[a['category']] = cats.get(a['category'], 0) + 1
    
    return render_template_string(TEMPLATE,
                                  articles=filtered,
                                  categories=cats,
                                  selected_cat=selected_cat,
                                  q=q,
                                  total_count=len(ARTICLES))

@app.route('/article/<int:article_id>')
def article_detail(article_id):
    article = next((a for a in ARTICLES if a['id'] == article_id), None)
    if not article:
        return 'ไม่พบบทความ', 404
    article['views'] += 1
    return f"""
    <div style="max-width:700px;margin:40px auto;font-family:Arial;padding:20px">
        <a href="/">← กลับ</a>
        <h1>{article['title']}</h1>
        <span style="background:#e8f4f8;color:#3498db;padding:3px 8px;border-radius:12px">
            {article['category']}
        </span>
        <p style="margin-top:20px">{article['content']}</p>
        <p style="color:#999">เข้าชม: {article['views']} ครั้ง</p>
    </div>
    """

if __name__ == '__main__':
    app.run(debug=True)
```

---

## สรุป

ใน Part 52 นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|---------------|
| URL Routing | Rules ครบถ้วน, endpoint names, strict slashes |
| URL Converters | Built-in + custom converters |
| Jinja2 | Syntax, control flow, expressions |
| Template Inheritance | extends, blocks, super(), include, macros |
| Template Filters | Built-in + custom filters |
| Static Files | โครงสร้าง, url_for(), cache busting |
| url_for() | Routes, blueprints, external URL |
| Redirects | PRG pattern, status codes |
| Error Handlers | Custom error pages, API errors |
| Flash Messages | One-time messages, categories |
| Response Objects | Types, headers, cookies |

### ขั้นตอนต่อไป

ใน Part 53 เราจะเรียนรู้:
- HTML Forms และการจัดการ
- WTForms library
- Flask-WTF extension
- Validation และ CSRF protection
- File uploads

---

*เขียนโดยหลักสูตร Python Advanced - Part 52*
