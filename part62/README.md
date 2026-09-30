# Part 62 - Django: Getting Started

## Django คืออะไร และทำไมต้องใช้ Django?

Django เป็น web framework ระดับสูง (high-level) สำหรับ Python ที่ถูกออกแบบมาเพื่อช่วยให้นักพัฒนาสร้าง web application ได้อย่างรวดเร็ว ปลอดภัย และมีประสิทธิภาพ Django ถูกสร้างขึ้นในปี 2003 โดยทีมพัฒนาหนังสือพิมพ์ Lawrence Journal-World และเปิดตัวสู่สาธารณะในปี 2005

Django ถูกใช้งานโดยองค์กรชั้นนำมากมาย เช่น Instagram, Pinterest, Mozilla, Disqus และ National Geographic ซึ่งแสดงให้เห็นว่า Django สามารถรองรับการใช้งานในระดับ production ที่มีผู้ใช้งานหลายล้านคนได้อย่างมีประสิทธิภาพ

---

## Django Philosophy (หลักการออกแบบของ Django)

### 1. Batteries Included (มาพร้อมทุกอย่าง)

Django ใช้หลักการ "Batteries Included" ซึ่งหมายความว่า framework มาพร้อมกับ component ทุกอย่างที่จำเป็นสำหรับการสร้าง web application โดยไม่ต้องพึ่งพา library ภายนอกมากนัก

ส่วนประกอบที่ Django มีให้พร้อมใช้งาน:
- **ORM (Object-Relational Mapper)**: สำหรับจัดการฐานข้อมูลผ่าน Python objects
- **Authentication System**: ระบบ login/logout, การจัดการ user และ permission
- **Admin Interface**: หน้า admin สำเร็จรูปสำหรับจัดการข้อมูล
- **Template Engine**: ระบบ template สำหรับสร้าง HTML
- **Form Handling**: การจัดการ HTML forms และ validation
- **Security Features**: การป้องกัน CSRF, XSS, SQL Injection
- **URL Routing**: การกำหนดเส้นทาง URL
- **Static Files Management**: การจัดการไฟล์ CSS, JavaScript, Image
- **Internationalization (i18n)**: รองรับหลายภาษา
- **Caching Framework**: ระบบ cache หลายรูปแบบ

```python
# ตัวอย่าง: Django ORM ใช้งานง่าย ไม่ต้องเขียน SQL
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title

# การ query ข้อมูล - ไม่ต้องเขียน SQL เลย
articles = Article.objects.filter(title__contains='Python')
latest = Article.objects.order_by('-published_date')[:5]
```

### 2. DRY Principle (Don't Repeat Yourself)

DRY เป็นหลักการสำคัญของ Django ที่ต้องการหลีกเลี่ยงการเขียนโค้ดซ้ำซ้อน Django ออกแบบมาให้นักพัฒนาต้องกำหนด logic ของแอปพลิเคชันเพียงครั้งเดียว แล้ว framework จะนำ logic นั้นไปใช้ในทุกที่ที่จำเป็นโดยอัตโนมัติ

```python
# ตัวอย่าง: กำหนด model เพียงครั้งเดียว
# Django จะสร้าง database schema, admin interface, form validation
# และ API documentation ให้โดยอัตโนมัติ

class Product(models.Model):
    name = models.CharField(max_length=200, verbose_name='ชื่อสินค้า')
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = 'สินค้า'
        verbose_name_plural = 'สินค้าทั้งหมด'
        ordering = ['-created_at']
    
    def __str__(self):
        return f'{self.name} - {self.price} บาท'
    
    def is_in_stock(self):
        return self.stock > 0
```

### 3. Explicit is Better Than Implicit

Django ชอบความชัดเจนมากกว่าการทำงานแบบ "magic" ที่เกิดขึ้นเบื้องหลัง แม้ว่าจะมีค่า default ที่ดีอยู่แล้ว แต่ทุกอย่างสามารถ customize ได้อย่างชัดเจน

### 4. Loose Coupling (การแยกส่วนอย่างหลวมๆ)

Django ออกแบบให้แต่ละชั้น (layer) ของแอปพลิเคชันทำงานได้โดยอิสระจากกัน ซึ่งทำให้ง่ายต่อการทดสอบและบำรุงรักษา

---

## Django vs Flask vs FastAPI: การเปรียบเทียบ

เพื่อให้เข้าใจว่าควรเลือกใช้ framework ไหน เราจะเปรียบเทียบ Django, Flask และ FastAPI ในด้านต่างๆ

### ตารางเปรียบเทียบ

| คุณสมบัติ | Django | Flask | FastAPI |
|---------|--------|-------|---------|
| ขนาด | Large (full-stack) | Micro | Modern async |
| ORM | Built-in | ไม่มี (ใช้ SQLAlchemy) | ไม่มี (ใช้ SQLAlchemy/Tortoise) |
| Admin Panel | Built-in | ไม่มี | ไม่มี |
| Authentication | Built-in | ต้องติดตั้งเพิ่ม | ต้องสร้างเอง |
| API Performance | ปานกลาง | ปานกลาง | สูงมาก (async) |
| Learning Curve | สูง | ต่ำ | ปานกลาง |
| Async Support | บางส่วน (3.1+) | ต้องใช้ Quart | Native |
| Auto Documentation | ไม่มี | ไม่มี | Built-in (Swagger/OpenAPI) |
| Community | ใหญ่มาก | ใหญ่ | กำลังเติบโต |
| เหมาะสำหรับ | Full web apps | API/Small apps | High-performance API |

### เมื่อไหรควรใช้ Django?

```python
# Django เหมาะสำหรับ:
# 1. เว็บไซต์ที่มีระบบ admin และการจัดการข้อมูล
# 2. แอปพลิเคชันที่ต้องการ authentication ที่ซับซ้อน
# 3. โปรเจกต์ขนาดกลางถึงใหญ่
# 4. ทีมที่ต้องการ convention over configuration

use_django_when = [
    "ต้องการ admin panel สำเร็จรูป",
    "ต้องการ ORM ที่ทรงพลัง",
    "ต้องการระบบ authentication ที่พร้อมใช้",
    "โปรเจกต์มีขนาดใหญ่และต้องการโครงสร้างที่ชัดเจน",
    "ทีมพัฒนามีสมาชิกหลายคน",
]
```

### เมื่อไหรควรใช้ Flask?

```python
# Flask เหมาะสำหรับ:
# 1. Microservices
# 2. Simple REST APIs
# 3. Prototype และ proof-of-concept
# 4. เมื่อต้องการ flexibility สูง

use_flask_when = [
    "ต้องการ simplicity และ flexibility",
    "สร้าง small to medium API",
    "ต้องการเลือก library เองทุกอย่าง",
    "Microservices architecture",
]
```

### เมื่อไหรควรใช้ FastAPI?

```python
# FastAPI เหมาะสำหรับ:
# 1. High-performance API
# 2. Real-time applications
# 3. เมื่อต้องการ automatic documentation
# 4. Machine Learning APIs

use_fastapi_when = [
    "ต้องการ performance สูง (async I/O)",
    "ต้องการ automatic API documentation",
    "สร้าง modern API ด้วย type hints",
    "Machine Learning model serving",
]
```

### ตัวอย่างโค้ดเปรียบเทียบ: Hello World

```python
# Django - views.py
from django.http import HttpResponse

def hello_world(request):
    return HttpResponse("Hello, Django World!")

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello_world, name='hello'),
]
```

```python
# Flask
from flask import Flask

app = Flask(__name__)

@app.route('/hello')
def hello_world():
    return 'Hello, Flask World!'

if __name__ == '__main__':
    app.run()
```

```python
# FastAPI
from fastapi import FastAPI

app = FastAPI()

@app.get('/hello')
async def hello_world():
    return {"message": "Hello, FastAPI World!"}
```

---

## การติดตั้ง Django และสร้าง Project

### ขั้นตอนที่ 1: สร้าง Virtual Environment

```bash
# สร้าง virtual environment
python -m venv django_env

# activate virtual environment (Linux/Mac)
source django_env/bin/activate

# activate virtual environment (Windows)
django_env\Scripts\activate

# ตรวจสอบว่า activate แล้ว
which python  # ควรแสดง path ภายใน django_env
```

### ขั้นตอนที่ 2: ติดตั้ง Django

```bash
# ติดตั้ง Django เวอร์ชันล่าสุด
pip install django

# ติดตั้ง Django เวอร์ชันที่ระบุ
pip install django==5.0

# ตรวจสอบเวอร์ชันที่ติดตั้ง
python -m django --version

# หรือ
django-admin --version
```

### ขั้นตอนที่ 3: สร้าง Django Project

```bash
# สร้าง project ใหม่ชื่อ 'myblog'
django-admin startproject myblog

# ดูโครงสร้าง directory ที่สร้างขึ้น
ls -la myblog/
```

### ตัวอย่าง: สร้าง Project แบบ Custom Directory

```bash
# สร้าง project ใน directory ปัจจุบัน (มี . ต่อท้าย)
mkdir my_project
cd my_project
django-admin startproject config .

# โครงสร้างจะเป็น:
# my_project/
#   manage.py
#   config/
#     __init__.py
#     settings.py
#     urls.py
#     wsgi.py
#     asgi.py
```

---

## โครงสร้าง Django Project (Project Structure)

หลังจากสร้าง project ด้วย `django-admin startproject myblog` จะได้โครงสร้างดังนี้:

```
myblog/
├── manage.py           # Command-line utility สำหรับจัดการ project
└── myblog/             # Package หลักของ project
    ├── __init__.py     # บอกว่า directory นี้เป็น Python package
    ├── settings.py     # การตั้งค่าทั้งหมดของ project
    ├── urls.py         # URL declarations (URL router หลัก)
    ├── wsgi.py         # WSGI entry point สำหรับ deployment
    └── asgi.py         # ASGI entry point สำหรับ async deployment
```

### manage.py - Command Line Tool

```python
# manage.py เป็นไฟล์ที่ถูกสร้างอัตโนมัติ
# ใช้สำหรับรัน management commands ต่างๆ

# ตัวอย่างคำสั่งที่ใช้บ่อย:
# python manage.py runserver          - รัน development server
# python manage.py makemigrations    - สร้าง migration files
# python manage.py migrate           - apply migrations ไปยัง database
# python manage.py createsuperuser   - สร้าง admin user
# python manage.py startapp <name>   - สร้าง app ใหม่
# python manage.py shell             - เปิด Python shell พร้อม Django context
# python manage.py test              - รัน tests
# python manage.py collectstatic     - รวบรวม static files

#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys


def main():
    """Run administrative tasks."""
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myblog.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
```

### __init__.py

```python
# myblog/__init__.py
# ไฟล์นี้ว่างเปล่า แต่มีความสำคัญ
# บอก Python ว่า directory นี้เป็น package
# ทำให้สามารถ import module จาก directory นี้ได้
```

### wsgi.py - Web Server Gateway Interface

```python
# myblog/wsgi.py
# ใช้สำหรับ deploy บน traditional web servers (Apache, Nginx)
# รองรับ synchronous requests

import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myblog.settings')

application = get_wsgi_application()
```

### asgi.py - Asynchronous Server Gateway Interface

```python
# myblog/asgi.py
# ใช้สำหรับ deploy บน async servers
# รองรับทั้ง HTTP, WebSockets, และ long-polling
# เหมาะสำหรับ real-time applications

import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myblog.settings')

application = get_asgi_application()
```

---

## Apps Concept ใน Django

Django ใช้แนวคิดของ "Apps" ในการแบ่ง project ออกเป็นส่วนย่อยๆ แต่ละ app ควรรับผิดชอบงานเพียงอย่างเดียว (Single Responsibility Principle)

### สร้าง App ใหม่

```bash
# สร้าง app ชื่อ 'blog'
python manage.py startapp blog

# โครงสร้าง app ที่สร้างขึ้น:
# blog/
#   __init__.py
#   admin.py      - การตั้งค่า Django admin
#   apps.py       - App configuration
#   models.py     - Database models
#   tests.py      - Test cases
#   views.py      - View functions/classes
#   migrations/   - Database migration files
#     __init__.py
```

### โครงสร้าง Project หลังเพิ่ม Apps

```
myblog/
├── manage.py
├── myblog/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── blog/                   # App สำหรับ blog
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── views.py
│   ├── urls.py             # (สร้างเอง)
│   ├── templates/          # (สร้างเอง)
│   │   └── blog/
│   │       ├── index.html
│   │       └── detail.html
│   └── migrations/
│       └── __init__.py
└── templates/              # Global templates (สร้างเอง)
    └── base.html
```

### apps.py - App Configuration

```python
# blog/apps.py
from django.apps import AppConfig


class BlogConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'blog'
    verbose_name = 'บล็อก'  # ชื่อที่แสดงใน admin
    
    def ready(self):
        # รันเมื่อ app พร้อมใช้งาน
        # มักใช้สำหรับ connect signals
        import blog.signals  # noqa
```

### การลงทะเบียน App ใน settings.py

```python
# myblog/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Apps ของเรา
    'blog.apps.BlogConfig',  # แนะนำให้ใช้แบบนี้ (explicit)
    # หรือใช้แบบสั้น:
    # 'blog',
]
```

---

## settings.py อย่างละเอียด

`settings.py` เป็นหัวใจสำคัญของ Django project ที่กำหนดการทำงานของระบบทั้งหมด

### ตัวอย่าง settings.py ฉบับสมบูรณ์

```python
# myblog/settings.py
"""
Django settings for myblog project.

Generated by 'django-admin startproject' using Django 5.0.
"""

from pathlib import Path
import os

# ============================================================
# BASE CONFIGURATION
# ============================================================

# Build paths inside the project like this: BASE_DIR / 'subdir'.
# BASE_DIR คือ directory หลักของ project
BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
# ควรเก็บ secret key ใน environment variable ในการ production
SECRET_KEY = 'django-insecure-your-secret-key-here'

# SECURITY WARNING: don't run with debug turned on in production!
# Development: True, Production: False
DEBUG = True

# Hosts ที่อนุญาตให้เข้าถึง
# Development: ['*'] หรือ ['localhost', '127.0.0.1']
# Production: ['yourdomain.com', 'www.yourdomain.com']
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '0.0.0.0']

# ============================================================
# INSTALLED APPS
# ============================================================

INSTALLED_APPS = [
    # Django built-in apps
    'django.contrib.admin',          # Admin interface
    'django.contrib.auth',           # Authentication framework
    'django.contrib.contenttypes',   # Content type framework
    'django.contrib.sessions',       # Session framework
    'django.contrib.messages',       # Messaging framework
    'django.contrib.staticfiles',    # Static files framework
    
    # Third-party apps (ติดตั้งเพิ่มเติม)
    # 'rest_framework',              # Django REST Framework
    # 'corsheaders',                 # CORS headers
    # 'debug_toolbar',               # Debug toolbar
    
    # Local apps
    'blog.apps.BlogConfig',
]

# ============================================================
# MIDDLEWARE
# ============================================================

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# ============================================================
# URL CONFIGURATION
# ============================================================

ROOT_URLCONF = 'myblog.urls'  # ไฟล์ urls.py หลัก

# ============================================================
# TEMPLATES
# ============================================================

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            BASE_DIR / 'templates',  # Global templates directory
        ],
        'APP_DIRS': True,  # ค้นหา templates ใน app/templates/ ด้วย
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# ============================================================
# WSGI/ASGI APPLICATION
# ============================================================

WSGI_APPLICATION = 'myblog.wsgi.application'
ASGI_APPLICATION = 'myblog.asgi.application'

# ============================================================
# DATABASE CONFIGURATION
# ============================================================

# Default: SQLite (เหมาะสำหรับ development)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# PostgreSQL configuration
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.postgresql',
#         'NAME': 'myblog_db',
#         'USER': 'postgres',
#         'PASSWORD': 'your_password',
#         'HOST': 'localhost',
#         'PORT': '5432',
#     }
# }

# MySQL configuration
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.mysql',
#         'NAME': 'myblog_db',
#         'USER': 'root',
#         'PASSWORD': 'your_password',
#         'HOST': 'localhost',
#         'PORT': '3306',
#     }
# }

# ============================================================
# PASSWORD VALIDATION
# ============================================================

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 8,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# ============================================================
# INTERNATIONALIZATION
# ============================================================

LANGUAGE_CODE = 'th'  # ภาษาไทย
TIME_ZONE = 'Asia/Bangkok'  # Timezone ประเทศไทย
USE_I18N = True    # เปิดใช้ internationalization
USE_TZ = True     # ใช้ timezone-aware datetimes

# ============================================================
# STATIC FILES (CSS, JavaScript, Images)
# ============================================================

# URL สำหรับเข้าถึง static files
STATIC_URL = '/static/'

# Directories ที่มี static files (สำหรับ development)
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]

# Directory ที่ collectstatic จะรวมไฟล์ (สำหรับ production)
STATIC_ROOT = BASE_DIR / 'staticfiles'

# ============================================================
# MEDIA FILES (User uploaded files)
# ============================================================

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# ============================================================
# DEFAULT PRIMARY KEY FIELD TYPE
# ============================================================

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# ============================================================
# EMAIL CONFIGURATION
# ============================================================

# Development: print emails to console
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Production SMTP configuration:
# EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
# EMAIL_HOST = 'smtp.gmail.com'
# EMAIL_PORT = 587
# EMAIL_USE_TLS = True
# EMAIL_HOST_USER = 'your-email@gmail.com'
# EMAIL_HOST_PASSWORD = 'your-app-password'

# ============================================================
# CACHE CONFIGURATION
# ============================================================

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}

# Redis cache (production):
# CACHES = {
#     'default': {
#         'BACKEND': 'django_redis.cache.RedisCache',
#         'LOCATION': 'redis://127.0.0.1:6379/1',
#         'OPTIONS': {
#             'CLIENT_CLASS': 'django_redis.client.DefaultClient',
#         }
#     }
# }

# ============================================================
# LOGGING CONFIGURATION
# ============================================================

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
        'file': {
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'debug.log',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': 'WARNING',
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}
```

### การใช้ Environment Variables ใน settings.py

```python
# settings.py - Production-ready configuration
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# ดึงค่าจาก environment variable
SECRET_KEY = os.environ.get('SECRET_KEY', 'fallback-secret-key-for-dev')
DEBUG = os.environ.get('DEBUG', 'True') == 'True'

# Parse database URL จาก environment variable
DATABASE_URL = os.environ.get('DATABASE_URL', '')

if DATABASE_URL:
    # ติดตั้ง: pip install dj-database-url
    import dj_database_url
    DATABASES = {
        'default': dj_database_url.parse(DATABASE_URL)
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
```

```bash
# .env file (อย่า commit ไปยัง git!)
SECRET_KEY=your-super-secret-key-here
DEBUG=False
DATABASE_URL=postgresql://user:password@localhost/dbname
```

---

## urls.py Patterns

URL configuration เป็นระบบ routing ของ Django ที่กำหนดว่า URL ไหนจะถูก handle โดย view ไหน

### urls.py หลัก (Project-level)

```python
# myblog/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # Admin URL
    path('admin/', admin.site.urls),
    
    # Include app URLs
    path('', include('blog.urls')),
    path('blog/', include('blog.urls', namespace='blog')),
    
    # API URLs
    path('api/', include('api.urls')),
]

# เพิ่ม media URL สำหรับ development เท่านั้น
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### urls.py ระดับ App

```python
# blog/urls.py
from django.urls import path
from . import views

# กำหนด namespace สำหรับ URL names
app_name = 'blog'

urlpatterns = [
    # Function-based views
    path('', views.index, name='index'),
    path('articles/', views.article_list, name='article-list'),
    path('articles/<int:pk>/', views.article_detail, name='article-detail'),
    path('articles/create/', views.article_create, name='article-create'),
    path('articles/<int:pk>/update/', views.article_update, name='article-update'),
    path('articles/<int:pk>/delete/', views.article_delete, name='article-delete'),
    
    # Class-based views
    path('posts/', views.PostListView.as_view(), name='post-list'),
    path('posts/<int:pk>/', views.PostDetailView.as_view(), name='post-detail'),
    
    # URL กับ parameter แบบ string
    path('category/<str:category_name>/', views.category_view, name='category'),
    
    # URL กับ slug
    path('articles/<slug:slug>/', views.article_by_slug, name='article-slug'),
]
```

### URL Patterns ที่ใช้บ่อย

```python
# การใช้ URL converters ต่างๆ
from django.urls import path, re_path
import views

urlpatterns = [
    # int: รับเฉพาะตัวเลขจำนวนเต็มบวก
    path('articles/<int:pk>/', views.detail, name='detail'),
    
    # str: รับ string ที่ไม่ใช่ slash (default)
    path('search/<str:query>/', views.search, name='search'),
    
    # slug: รับ string ที่ประกอบด้วย letters, numbers, hyphens, underscores
    path('posts/<slug:slug>/', views.post_detail, name='post-detail'),
    
    # uuid: รับ UUID format
    path('objects/<uuid:object_id>/', views.object_detail, name='object-detail'),
    
    # path: รับ string รวมถึง slash
    path('files/<path:file_path>/', views.file_view, name='file'),
    
    # Regular expression (ใช้ re_path)
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive, name='year-archive'),
]
```

### ตัวอย่างการใช้ URL ใน Templates และ Views

```python
# ใน views.py - redirect ไปยัง URL ด้วย name
from django.urls import reverse
from django.shortcuts import redirect

def create_article(request):
    if request.method == 'POST':
        # บันทึกข้อมูล...
        article_id = 1  # สมมติ
        return redirect(reverse('blog:article-detail', kwargs={'pk': article_id}))
    # ...

# หรือใช้ reverse_lazy สำหรับ class-based views
from django.urls import reverse_lazy

class ArticleCreateView(CreateView):
    success_url = reverse_lazy('blog:article-list')
```

```html
<!-- ใน templates - ใช้ URL tag -->
<a href="{% url 'blog:article-list' %}">รายการบทความ</a>
<a href="{% url 'blog:article-detail' pk=article.pk %}">{{ article.title }}</a>
<a href="{% url 'blog:article-slug' slug=article.slug %}">{{ article.title }}</a>
```

---

## Development Server

### การรัน Development Server

```bash
# รัน server ที่ default port 8000
python manage.py runserver

# รัน server ที่ port อื่น
python manage.py runserver 8080

# รัน server ที่ IP address และ port เฉพาะ
python manage.py runserver 0.0.0.0:8000

# รัน server พร้อม settings file เฉพาะ
python manage.py runserver --settings=myblog.settings_dev
```

### Output ที่เห็นเมื่อรัน Server

```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
September 30, 2026 - 10:00:00
Django version 5.0, using settings 'myblog.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

### ตัวอย่าง View แรก

```python
# blog/views.py
from django.shortcuts import render
from django.http import HttpResponse

def index(request):
    """View สำหรับหน้าแรก"""
    context = {
        'title': 'ยินดีต้อนรับสู่ Django Blog',
        'message': 'นี่คือหน้าแรกของ blog ของเรา',
    }
    return render(request, 'blog/index.html', context)

def hello(request):
    """View แบบง่ายที่สุด"""
    return HttpResponse('<h1>สวัสดี Django!</h1>')
```

---

## Django Admin Panel

Django มาพร้อมกับ admin interface ที่ทรงพลังและพร้อมใช้งาน เพียงแค่กำหนดค่าเล็กน้อย

### ขั้นตอนการตั้งค่า Admin

```bash
# ขั้นที่ 1: สร้าง database tables
python manage.py migrate

# ขั้นที่ 2: สร้าง superuser (admin account)
python manage.py createsuperuser
# จะถูกถามให้กรอก:
# Username: admin
# Email address: admin@example.com
# Password: ********
# Password (again): ********

# ขั้นที่ 3: รัน development server
python manage.py runserver

# เข้าถึง admin ที่: http://127.0.0.1:8000/admin/
```

### การลงทะเบียน Model ใน Admin

```python
# blog/admin.py
from django.contrib import admin
from .models import Article, Category, Tag

# วิธีที่ 1: การลงทะเบียนแบบพื้นฐาน
admin.site.register(Category)

# วิธีที่ 2: การลงทะเบียนพร้อม customization
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    # คอลัมน์ที่แสดงในรายการ
    list_display = ['title', 'author', 'category', 'published', 'created_at']
    
    # ฟิลด์ที่ค้นหาได้
    search_fields = ['title', 'content', 'author__username']
    
    # Filter ที่แถบด้านขวา
    list_filter = ['published', 'category', 'created_at']
    
    # ฟิลด์ที่คลิกแล้วเข้าไปแก้ไขได้
    list_display_links = ['title']
    
    # ฟิลด์ที่แก้ไขได้จากหน้ารายการเลย
    list_editable = ['published']
    
    # จำนวนรายการต่อหน้า
    list_per_page = 20
    
    # การจัดกลุ่ม fields ในหน้าแก้ไข
    fieldsets = (
        ('ข้อมูลหลัก', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('เนื้อหา', {
            'fields': ('content', 'summary', 'featured_image')
        }),
        ('การตั้งค่า', {
            'fields': ('published', 'tags'),
            'classes': ('collapse',),  # ซ่อนได้
        }),
        ('วันที่', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',),
        }),
    )
    
    # ฟิลด์ที่เป็น read-only
    readonly_fields = ['created_at', 'updated_at']
    
    # Inline models (many-to-many หรือ foreign key)
    # inlines = [CommentInline]
    
    # Auto-populate slug จาก title
    prepopulated_fields = {'slug': ('title',)}
    
    # Custom actions
    actions = ['publish_articles', 'unpublish_articles']
    
    def publish_articles(self, request, queryset):
        """เผยแพร่บทความที่เลือก"""
        updated = queryset.update(published=True)
        self.message_user(request, f'เผยแพร่ {updated} บทความแล้ว')
    publish_articles.short_description = 'เผยแพร่บทความที่เลือก'
    
    def unpublish_articles(self, request, queryset):
        """ยกเลิกการเผยแพร่บทความที่เลือก"""
        updated = queryset.update(published=False)
        self.message_user(request, f'ยกเลิกการเผยแพร่ {updated} บทความแล้ว')
    unpublish_articles.short_description = 'ยกเลิกการเผยแพร่บทความที่เลือก'
```

### การ Customize Admin Site

```python
# myblog/admin.py หรือ blog/admin.py
from django.contrib import admin

# เปลี่ยนชื่อและ header ของ admin site
admin.site.site_header = 'MyBlog Administration'
admin.site.site_title = 'MyBlog Admin Portal'
admin.site.index_title = 'Welcome to MyBlog Admin'
```

### Inline Admin - แสดง related objects ในหน้าเดียว

```python
# blog/admin.py
from django.contrib import admin
from .models import Article, Comment

class CommentInline(admin.TabularInline):  # หรือ StackedInline
    model = Comment
    extra = 1  # จำนวน empty form ที่แสดง
    fields = ['author', 'content', 'approved']
    readonly_fields = ['created_at']

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
    # ...
```

---

## Template System (ระบบ Template)

Django Template Language (DTL) เป็นภาษา template ที่ออกแบบมาเพื่อความง่ายและปลอดภัย

### การตั้งค่า Templates Directory

```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # global templates
        'APP_DIRS': True,  # ค้นหา templates ใน app/templates/ ด้วย
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### Base Template

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}MyBlog{% endblock %}</title>
    
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/main.css' %}">
    
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'blog:index' %}">หน้าแรก</a>
            <a href="{% url 'blog:article-list' %}">บทความ</a>
            {% if user.is_authenticated %}
                <a href="{% url 'blog:article-create' %}">เขียนบทความ</a>
                <a href="{% url 'admin:index' %}">Admin</a>
                <form method="post" action="{% url 'logout' %}">
                    {% csrf_token %}
                    <button type="submit">ออกจากระบบ</button>
                </form>
            {% else %}
                <a href="{% url 'login' %}">เข้าสู่ระบบ</a>
            {% endif %}
        </nav>
    </header>
    
    <main>
        {% if messages %}
            <div class="messages">
                {% for message in messages %}
                    <div class="alert alert-{{ message.tags }}">
                        {{ message }}
                    </div>
                {% endfor %}
            </div>
        {% endif %}
        
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        <p>&copy; 2026 MyBlog. สงวนลิขสิทธิ์</p>
    </footer>
    
    <script src="{% static 'js/main.js' %}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### Blog Index Template

```html
<!-- blog/templates/blog/index.html -->
{% extends 'base.html' %}
{% load static %}

{% block title %}หน้าแรก - MyBlog{% endblock %}

{% block content %}
<div class="container">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
    
    <h2>บทความล่าสุด</h2>
    
    {% if articles %}
        <div class="article-grid">
            {% for article in articles %}
            <div class="article-card">
                {% if article.featured_image %}
                    <img src="{{ article.featured_image.url }}" alt="{{ article.title }}">
                {% endif %}
                
                <h3>
                    <a href="{% url 'blog:article-detail' pk=article.pk %}">
                        {{ article.title }}
                    </a>
                </h3>
                
                <p class="meta">
                    โดย {{ article.author.get_full_name|default:article.author.username }}
                    เมื่อ {{ article.created_at|date:"d M Y" }}
                    ใน {{ article.category.name }}
                </p>
                
                <p>{{ article.summary|truncatewords:30 }}</p>
                
                <div class="tags">
                    {% for tag in article.tags.all %}
                        <span class="tag">{{ tag.name }}</span>
                    {% endfor %}
                </div>
            </div>
            {% endfor %}
        </div>
        
        <!-- Pagination -->
        {% if is_paginated %}
        <div class="pagination">
            {% if page_obj.has_previous %}
                <a href="?page={{ page_obj.previous_page_number }}">ก่อนหน้า</a>
            {% endif %}
            
            <span>หน้า {{ page_obj.number }} จาก {{ page_obj.paginator.num_pages }}</span>
            
            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">ถัดไป</a>
            {% endif %}
        </div>
        {% endif %}
        
    {% else %}
        <p>ยังไม่มีบทความ</p>
    {% endif %}
</div>
{% endblock %}
```

### Template Tags และ Filters ที่ใช้บ่อย

```html
<!-- ตัวอย่าง Template Tags และ Filters -->

<!-- Variables -->
{{ variable }}
{{ object.attribute }}
{{ dictionary.key }}

<!-- Filters -->
{{ name|upper }}                    <!-- แปลงเป็นตัวพิมพ์ใหญ่ -->
{{ name|lower }}                    <!-- แปลงเป็นตัวพิมพ์เล็ก -->
{{ text|truncatewords:50 }}         <!-- ตัดข้อความให้เหลือ 50 คำ -->
{{ text|truncatechars:200 }}        <!-- ตัดข้อความให้เหลือ 200 ตัวอักษร -->
{{ date|date:"d/m/Y" }}             <!-- format วันที่ -->
{{ date|timesince }}                <!-- "2 hours ago" -->
{{ number|floatformat:2 }}          <!-- 1234.57 -->
{{ list|join:", " }}                <!-- join list ด้วย comma -->
{{ html|safe }}                     <!-- แสดง HTML โดยไม่ escape -->
{{ value|default:"ไม่มีข้อมูล" }}  <!-- ค่า default -->
{{ value|linebreaks }}              <!-- แปลง newlines เป็น <br> -->
{{ items|length }}                  <!-- นับจำนวนรายการ -->

<!-- Control flow -->
{% if condition %}
    ...
{% elif other_condition %}
    ...
{% else %}
    ...
{% endif %}

<!-- Loops -->
{% for item in items %}
    {{ forloop.counter }}   <!-- นับ 1, 2, 3, ... -->
    {{ forloop.counter0 }}  <!-- นับ 0, 1, 2, ... -->
    {{ forloop.first }}     <!-- True ในรอบแรก -->
    {{ forloop.last }}      <!-- True ในรอบสุดท้าย -->
    {{ item }}
{% empty %}
    <p>ไม่มีรายการ</p>
{% endfor %}

<!-- URL -->
{% url 'blog:article-detail' pk=article.pk %}

<!-- CSRF Token (ต้องมีใน forms ทุกอัน) -->
{% csrf_token %}

<!-- Static files -->
{% load static %}
{% static 'css/style.css' %}

<!-- Template inheritance -->
{% extends 'base.html' %}
{% block content %}...{% endblock %}
{% include 'partials/header.html' %}
{% include 'partials/sidebar.html' with category=category %}
```

---

## Static Files (ไฟล์ CSS, JS, Images)

### การตั้งค่า Static Files

```python
# settings.py
# URL path สำหรับ static files
STATIC_URL = '/static/'

# Directories เพิ่มเติมที่มี static files
STATICFILES_DIRS = [
    BASE_DIR / 'static',        # Global static directory
]

# Directory สำหรับ collectstatic (production)
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Static file finders (ค้นหา static files จากที่ไหนบ้าง)
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',   # จาก STATICFILES_DIRS
    'django.contrib.staticfiles.finders.AppDirectoriesFinder', # จาก app/static/
]
```

### โครงสร้าง Static Files

```
myblog/
├── static/                     # Global static files
│   ├── css/
│   │   ├── main.css
│   │   └── bootstrap.min.css
│   ├── js/
│   │   ├── main.js
│   │   └── jquery.min.js
│   └── images/
│       └── logo.png
├── blog/
│   └── static/                 # App-specific static files
│       └── blog/
│           ├── css/
│           │   └── blog.css
│           └── js/
│               └── blog.js
└── staticfiles/                # Collected static files (gitignore this)
```

### ตัวอย่างไฟล์ CSS

```css
/* static/css/main.css */
/* Base styles */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: 'Sarabun', 'Noto Sans Thai', sans-serif;
    font-size: 16px;
    line-height: 1.6;
    color: #333;
    background-color: #f5f5f5;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

header {
    background-color: #2c3e50;
    color: white;
    padding: 1rem;
}

header nav a {
    color: white;
    text-decoration: none;
    margin: 0 10px;
}

.article-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 20px;
    margin-top: 20px;
}

.article-card {
    background: white;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
```

### ใช้ Static Files ใน Templates

```html
<!-- templates/base.html -->
{% load static %}

<!-- CSS -->
<link rel="stylesheet" href="{% static 'css/main.css' %}">
<link rel="stylesheet" href="{% static 'blog/css/blog.css' %}">

<!-- JavaScript -->
<script src="{% static 'js/main.js' %}"></script>

<!-- Image -->
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

### Collect Static Files (สำหรับ Production)

```bash
# รวบรวม static files ทั้งหมดไปยัง STATIC_ROOT
python manage.py collectstatic

# Force overwrite โดยไม่ถาม
python manage.py collectstatic --noinput
```

---

## Management Commands ที่ใช้บ่อย

### คำสั่ง Database

```bash
# สร้าง migration files จากการเปลี่ยนแปลง models
python manage.py makemigrations

# สร้าง migration เฉพาะ app
python manage.py makemigrations blog

# แสดง SQL ที่จะ execute
python manage.py sqlmigrate blog 0001

# Apply migrations ไปยัง database
python manage.py migrate

# Apply migrations เฉพาะ app
python manage.py migrate blog

# Rollback migration
python manage.py migrate blog 0001  # กลับไปที่ migration 0001

# ดูสถานะ migrations
python manage.py showmigrations

# สร้าง empty migration
python manage.py makemigrations --empty blog
```

### คำสั่ง User Management

```bash
# สร้าง superuser
python manage.py createsuperuser

# เปลี่ยน password
python manage.py changepassword username
```

### คำสั่ง Shell

```bash
# เปิด Python shell พร้อม Django setup
python manage.py shell

# เปิด shell ด้วย IPython (ติดตั้งเพิ่ม)
python manage.py shell -i ipython

# รัน Python script ใน Django context
python manage.py shell < script.py
```

### คำสั่ง Testing

```bash
# รัน tests ทั้งหมด
python manage.py test

# รัน tests เฉพาะ app
python manage.py test blog

# รัน tests เฉพาะ class
python manage.py test blog.tests.ArticleTestCase

# รัน tests พร้อม verbose output
python manage.py test --verbosity=2

# สร้าง test database และเก็บไว้
python manage.py test --keepdb
```

### คำสั่ง Static Files

```bash
# รวบรวม static files
python manage.py collectstatic

# ตรวจสอบ static files
python manage.py findstatic css/main.css
```

### คำสั่ง อื่นๆ

```bash
# ตรวจสอบ project สำหรับ common issues
python manage.py check

# ตรวจสอบสำหรับ deployment
python manage.py check --deploy

# ดู URLs ทั้งหมด
python manage.py show_urls  # ต้องติดตั้ง django-extensions

# Flush database (ลบข้อมูลทั้งหมด)
python manage.py flush

# Load data จาก fixtures
python manage.py loaddata fixtures/initial_data.json

# Dump data ไปยัง fixtures
python manage.py dumpdata blog --indent=2 > fixtures/blog_data.json

# Clear cache
python manage.py clear_cache  # ต้องมี cache backend configured
```

### สร้าง Custom Management Command

```python
# blog/management/__init__.py
# blog/management/commands/__init__.py
# blog/management/commands/publish_articles.py

from django.core.management.base import BaseCommand, CommandError
from blog.models import Article
from django.utils import timezone


class Command(BaseCommand):
    help = 'เผยแพร่บทความที่กำหนด scheduled date'
    
    def add_arguments(self, parser):
        # Optional argument
        parser.add_argument(
            '--dry-run',
            action='store_true',
            help='แสดงผลโดยไม่ save',
        )
        # Positional argument
        parser.add_argument('article_ids', nargs='*', type=int)
    
    def handle(self, *args, **options):
        dry_run = options['dry_run']
        article_ids = options['article_ids']
        
        now = timezone.now()
        
        if article_ids:
            articles = Article.objects.filter(
                id__in=article_ids,
                published=False
            )
        else:
            articles = Article.objects.filter(
                published=False,
                scheduled_at__lte=now
            )
        
        count = 0
        for article in articles:
            if not dry_run:
                article.published = True
                article.save()
            
            self.stdout.write(
                self.style.SUCCESS(f'เผยแพร่: {article.title}')
            )
            count += 1
        
        if dry_run:
            self.stdout.write(f'Dry run: จะเผยแพร่ {count} บทความ')
        else:
            self.stdout.write(
                self.style.SUCCESS(f'เผยแพร่ {count} บทความสำเร็จ')
            )
```

```bash
# รัน custom command
python manage.py publish_articles
python manage.py publish_articles --dry-run
python manage.py publish_articles 1 2 3
```

---

## ตัวอย่างโปรแกรมจริง: Simple Blog Setup

### ขั้นตอนที่ 1: สร้าง Models

```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify
from django.urls import reverse


class Category(models.Model):
    """หมวดหมู่บทความ"""
    name = models.CharField(max_length=100, unique=True, verbose_name='ชื่อหมวดหมู่')
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True, verbose_name='คำอธิบาย')
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = 'หมวดหมู่'
        verbose_name_plural = 'หมวดหมู่ทั้งหมด'
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        return reverse('blog:category', kwargs={'slug': self.slug})


class Tag(models.Model):
    """แท็กสำหรับบทความ"""
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    
    class Meta:
        verbose_name = 'แท็ก'
        verbose_name_plural = 'แท็กทั้งหมด'
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)


class Article(models.Model):
    """บทความ"""
    STATUS_DRAFT = 'draft'
    STATUS_PUBLISHED = 'published'
    STATUS_CHOICES = [
        (STATUS_DRAFT, 'แบบร่าง'),
        (STATUS_PUBLISHED, 'เผยแพร่แล้ว'),
    ]
    
    title = models.CharField(max_length=200, verbose_name='หัวข้อ')
    slug = models.SlugField(unique=True, blank=True)
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='articles',
        verbose_name='ผู้เขียน'
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='articles',
        verbose_name='หมวดหมู่'
    )
    tags = models.ManyToManyField(Tag, blank=True, verbose_name='แท็ก')
    summary = models.TextField(max_length=500, blank=True, verbose_name='สรุป')
    content = models.TextField(verbose_name='เนื้อหา')
    featured_image = models.ImageField(
        upload_to='articles/',
        blank=True,
        null=True,
        verbose_name='รูปภาพหลัก'
    )
    status = models.CharField(
        max_length=10,
        choices=STATUS_CHOICES,
        default=STATUS_DRAFT,
        verbose_name='สถานะ'
    )
    views_count = models.PositiveIntegerField(default=0, verbose_name='จำนวนการดู')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='วันที่สร้าง')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='วันที่แก้ไขล่าสุด')
    published_at = models.DateTimeField(null=True, blank=True, verbose_name='วันที่เผยแพร่')
    
    class Meta:
        verbose_name = 'บทความ'
        verbose_name_plural = 'บทความทั้งหมด'
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        
        # ตั้ง published_at เมื่อเปลี่ยนสถานะเป็น published
        from django.utils import timezone
        if self.status == self.STATUS_PUBLISHED and not self.published_at:
            self.published_at = timezone.now()
        
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        return reverse('blog:article-detail', kwargs={'slug': self.slug})
    
    def is_published(self):
        return self.status == self.STATUS_PUBLISHED
    
    def increment_views(self):
        """เพิ่มจำนวนการดู"""
        Article.objects.filter(pk=self.pk).update(views_count=models.F('views_count') + 1)


class Comment(models.Model):
    """ความคิดเห็น"""
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name='บทความ'
    )
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name='ผู้แสดงความคิดเห็น'
    )
    content = models.TextField(verbose_name='ความคิดเห็น')
    approved = models.BooleanField(default=False, verbose_name='อนุมัติแล้ว')
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = 'ความคิดเห็น'
        verbose_name_plural = 'ความคิดเห็นทั้งหมด'
        ordering = ['created_at']
    
    def __str__(self):
        return f'ความคิดเห็นโดย {self.author} ใน {self.article}'
```

### ขั้นตอนที่ 2: สร้าง Views

```python
# blog/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from django.core.paginator import Paginator
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from django.db.models import Q

from .models import Article, Category, Tag, Comment
from .forms import ArticleForm, CommentForm


# ============================================================
# Function-Based Views
# ============================================================

def index(request):
    """หน้าแรก - แสดงบทความล่าสุด"""
    articles = Article.objects.filter(
        status=Article.STATUS_PUBLISHED
    ).select_related('author', 'category').prefetch_related('tags')
    
    # Pagination
    paginator = Paginator(articles, 6)  # 6 บทความต่อหน้า
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    # Categories สำหรับ sidebar
    categories = Category.objects.all()
    
    context = {
        'page_obj': page_obj,
        'categories': categories,
        'title': 'บทความทั้งหมด',
    }
    return render(request, 'blog/index.html', context)


def article_detail(request, slug):
    """แสดงรายละเอียดบทความ"""
    article = get_object_or_404(
        Article,
        slug=slug,
        status=Article.STATUS_PUBLISHED
    )
    
    # เพิ่มจำนวนการดู
    article.increment_views()
    
    # Comments ที่อนุมัติแล้ว
    comments = article.comments.filter(approved=True)
    
    # Form สำหรับแสดงความคิดเห็น
    comment_form = CommentForm()
    
    # Related articles
    related_articles = Article.objects.filter(
        status=Article.STATUS_PUBLISHED,
        category=article.category
    ).exclude(pk=article.pk)[:3]
    
    context = {
        'article': article,
        'comments': comments,
        'comment_form': comment_form,
        'related_articles': related_articles,
    }
    return render(request, 'blog/article_detail.html', context)


@login_required
def add_comment(request, article_slug):
    """เพิ่มความคิดเห็น"""
    article = get_object_or_404(Article, slug=article_slug)
    
    if request.method == 'POST':
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.article = article
            comment.author = request.user
            comment.save()
            messages.success(request, 'ความคิดเห็นของคุณรอการอนุมัติ')
        else:
            messages.error(request, 'กรุณาตรวจสอบข้อมูล')
    
    return redirect('blog:article-detail', slug=article_slug)


def search_articles(request):
    """ค้นหาบทความ"""
    query = request.GET.get('q', '')
    articles = []
    
    if query:
        articles = Article.objects.filter(
            Q(title__icontains=query) |
            Q(content__icontains=query) |
            Q(summary__icontains=query),
            status=Article.STATUS_PUBLISHED
        ).distinct()
    
    context = {
        'articles': articles,
        'query': query,
    }
    return render(request, 'blog/search.html', context)


# ============================================================
# Class-Based Views
# ============================================================

class ArticleListView(ListView):
    """รายการบทความแบบ Class-Based View"""
    model = Article
    template_name = 'blog/article_list.html'
    context_object_name = 'articles'
    paginate_by = 10
    
    def get_queryset(self):
        return Article.objects.filter(
            status=Article.STATUS_PUBLISHED
        ).select_related('author', 'category')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        return context


class ArticleDetailView(DetailView):
    """รายละเอียดบทความแบบ Class-Based View"""
    model = Article
    template_name = 'blog/article_detail.html'
    context_object_name = 'article'
    
    def get_object(self):
        obj = super().get_object()
        obj.increment_views()
        return obj


class ArticleCreateView(LoginRequiredMixin, CreateView):
    """สร้างบทความใหม่"""
    model = Article
    form_class = ArticleForm
    template_name = 'blog/article_form.html'
    success_url = reverse_lazy('blog:article-list')
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        messages.success(self.request, 'สร้างบทความสำเร็จ')
        return super().form_valid(form)


class ArticleUpdateView(LoginRequiredMixin, UpdateView):
    """แก้ไขบทความ"""
    model = Article
    form_class = ArticleForm
    template_name = 'blog/article_form.html'
    
    def get_queryset(self):
        # ให้แก้ไขได้เฉพาะบทความของตัวเอง
        return Article.objects.filter(author=self.request.user)
    
    def form_valid(self, form):
        messages.success(self.request, 'แก้ไขบทความสำเร็จ')
        return super().form_valid(form)


class ArticleDeleteView(LoginRequiredMixin, DeleteView):
    """ลบบทความ"""
    model = Article
    template_name = 'blog/article_confirm_delete.html'
    success_url = reverse_lazy('blog:article-list')
    
    def get_queryset(self):
        return Article.objects.filter(author=self.request.user)
    
    def delete(self, request, *args, **kwargs):
        messages.success(request, 'ลบบทความสำเร็จ')
        return super().delete(request, *args, **kwargs)
```

### ขั้นตอนที่ 3: สร้าง Forms

```python
# blog/forms.py
from django import forms
from .models import Article, Comment


class ArticleForm(forms.ModelForm):
    """Form สำหรับสร้าง/แก้ไขบทความ"""
    
    class Meta:
        model = Article
        fields = ['title', 'category', 'tags', 'summary', 'content', 
                  'featured_image', 'status']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'หัวข้อบทความ'
            }),
            'summary': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 3,
                'placeholder': 'สรุปบทความ (ไม่เกิน 500 ตัวอักษร)'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 20,
                'placeholder': 'เนื้อหาบทความ'
            }),
            'status': forms.Select(attrs={'class': 'form-control'}),
        }
        labels = {
            'title': 'หัวข้อ',
            'category': 'หมวดหมู่',
            'tags': 'แท็ก',
            'summary': 'สรุป',
            'content': 'เนื้อหา',
            'featured_image': 'รูปภาพหลัก',
            'status': 'สถานะ',
        }
    
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 5:
            raise forms.ValidationError('หัวข้อต้องมีอย่างน้อย 5 ตัวอักษร')
        return title
    
    def clean_content(self):
        content = self.cleaned_data.get('content')
        if len(content) < 100:
            raise forms.ValidationError('เนื้อหาต้องมีอย่างน้อย 100 ตัวอักษร')
        return content


class CommentForm(forms.ModelForm):
    """Form สำหรับแสดงความคิดเห็น"""
    
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
                'placeholder': 'แสดงความคิดเห็น...'
            }),
        }
        labels = {
            'content': 'ความคิดเห็น',
        }
```

### ขั้นตอนที่ 4: ตั้งค่า URLs

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.ArticleListView.as_view(), name='index'),
    path('articles/', views.ArticleListView.as_view(), name='article-list'),
    path('articles/<slug:slug>/', views.article_detail, name='article-detail'),
    path('articles/create/', views.ArticleCreateView.as_view(), name='article-create'),
    path('articles/<slug:slug>/update/', views.ArticleUpdateView.as_view(), name='article-update'),
    path('articles/<slug:slug>/delete/', views.ArticleDeleteView.as_view(), name='article-delete'),
    path('articles/<slug:article_slug>/comment/', views.add_comment, name='add-comment'),
    path('search/', views.search_articles, name='search'),
]
```

### ขั้นตอนที่ 5: สร้าง Migrations และรัน

```bash
# สร้าง migration
python manage.py makemigrations blog

# Apply migration
python manage.py migrate

# สร้าง superuser
python manage.py createsuperuser

# รัน server
python manage.py runserver
```

---

## ตัวอย่างโปรแกรมจริง: Content Management System (CMS)

### CMS Models

```python
# cms/models.py
from django.db import models
from django.contrib.auth.models import User


class Page(models.Model):
    """หน้าเว็บสำหรับ CMS"""
    
    TEMPLATE_CHOICES = [
        ('default', 'Default'),
        ('landing', 'Landing Page'),
        ('about', 'About Page'),
        ('contact', 'Contact Page'),
    ]
    
    title = models.CharField(max_length=200, verbose_name='หัวข้อหน้า')
    slug = models.SlugField(unique=True, verbose_name='URL Slug')
    template = models.CharField(
        max_length=20,
        choices=TEMPLATE_CHOICES,
        default='default'
    )
    content = models.TextField(verbose_name='เนื้อหา')
    meta_title = models.CharField(max_length=60, blank=True, verbose_name='Meta Title')
    meta_description = models.CharField(max_length=160, blank=True, verbose_name='Meta Description')
    is_published = models.BooleanField(default=False, verbose_name='เผยแพร่')
    order = models.IntegerField(default=0, verbose_name='ลำดับ')
    parent = models.ForeignKey(
        'self',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='children'
    )
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = 'หน้า'
        verbose_name_plural = 'หน้าทั้งหมด'
        ordering = ['order', 'title']
    
    def __str__(self):
        return self.title
    
    def get_breadcrumbs(self):
        """ดึง breadcrumbs จาก parent pages"""
        breadcrumbs = [self]
        parent = self.parent
        while parent:
            breadcrumbs.insert(0, parent)
            parent = parent.parent
        return breadcrumbs


class MenuItem(models.Model):
    """รายการเมนูในการนำทาง"""
    MENU_CHOICES = [
        ('main', 'เมนูหลัก'),
        ('footer', 'Footer Menu'),
        ('sidebar', 'Sidebar Menu'),
    ]
    
    menu = models.CharField(max_length=20, choices=MENU_CHOICES)
    title = models.CharField(max_length=100)
    url = models.CharField(max_length=200, blank=True)
    page = models.ForeignKey(Page, on_delete=models.SET_NULL, null=True, blank=True)
    parent = models.ForeignKey(
        'self',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='children'
    )
    order = models.IntegerField(default=0)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        ordering = ['order']
    
    def __str__(self):
        return f'{self.menu}: {self.title}'
    
    def get_url(self):
        if self.page:
            return f'/{self.page.slug}/'
        return self.url
```

### CMS Admin Configuration

```python
# cms/admin.py
from django.contrib import admin
from .models import Page, MenuItem


class PageChildInline(admin.TabularInline):
    model = Page
    fk_name = 'parent'
    fields = ['title', 'slug', 'is_published', 'order']
    extra = 0


@admin.register(Page)
class PageAdmin(admin.ModelAdmin):
    list_display = ['title', 'slug', 'template', 'is_published', 'parent', 'order']
    list_filter = ['is_published', 'template']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    list_editable = ['is_published', 'order']
    inlines = [PageChildInline]
    
    fieldsets = (
        ('เนื้อหาหลัก', {
            'fields': ('title', 'slug', 'template', 'content', 'parent', 'order')
        }),
        ('SEO', {
            'fields': ('meta_title', 'meta_description'),
            'classes': ('collapse',),
        }),
        ('การตั้งค่า', {
            'fields': ('is_published',),
        }),
    )
    
    def save_model(self, request, obj, form, change):
        if not change:
            obj.created_by = request.user
        super().save_model(request, obj, form, change)


@admin.register(MenuItem)
class MenuItemAdmin(admin.ModelAdmin):
    list_display = ['title', 'menu', 'url', 'page', 'parent', 'order', 'is_active']
    list_filter = ['menu', 'is_active']
    list_editable = ['order', 'is_active']
```

### CMS Views

```python
# cms/views.py
from django.shortcuts import render, get_object_or_404
from .models import Page, MenuItem


def page_view(request, slug=None):
    """แสดงหน้าจาก CMS"""
    if slug:
        page = get_object_or_404(Page, slug=slug, is_published=True)
    else:
        # หน้าแรก
        page = get_object_or_404(Page, slug='home', is_published=True)
    
    # ดึงเมนูหลัก
    main_menu = MenuItem.objects.filter(
        menu='main',
        parent=None,
        is_active=True
    ).prefetch_related('children')
    
    # กำหนด template ตามที่ตั้งค่าไว้
    template = f'cms/pages/{page.template}.html'
    
    context = {
        'page': page,
        'main_menu': main_menu,
        'breadcrumbs': page.get_breadcrumbs(),
    }
    return render(request, template, context)
```

---

## Django Shell - การใช้งาน Interactive Shell

### การใช้ Django Shell สำหรับทดสอบ

```python
# รัน: python manage.py shell

# Import models
from blog.models import Article, Category, Tag
from django.contrib.auth.models import User

# สร้าง Category
cat = Category.objects.create(name='Python', slug='python')
cat2 = Category.objects.create(name='Django', slug='django')

# สร้าง User
user = User.objects.create_superuser('admin', 'admin@example.com', 'password123')

# สร้าง Article
article = Article.objects.create(
    title='บทความแรกของเรา',
    author=user,
    category=cat,
    content='นี่คือเนื้อหาของบทความแรก...',
    status='published'
)

# Query ข้อมูล
articles = Article.objects.all()
published = Article.objects.filter(status='published')
recent = Article.objects.order_by('-created_at')[:5]

# Complex queries
from django.db.models import Q, Count
articles_with_comments = Article.objects.annotate(
    comment_count=Count('comments')
).filter(comment_count__gt=0)

# F expressions
from django.db.models import F
Article.objects.filter(views_count__lt=100).update(
    views_count=F('views_count') + 10
)

print(f"มีบทความทั้งหมด: {Article.objects.count()} บทความ")
```

---

## การตั้งค่า Production-Ready

### settings สำหรับ Production

```python
# settings_production.py
import os
from .settings import *  # import base settings

# Override สำหรับ production
DEBUG = False
ALLOWED_HOSTS = [os.environ.get('ALLOWED_HOST', 'yourdomain.com')]

# Security headers
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
X_FRAME_OPTIONS = 'DENY'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Static files
STATIC_ROOT = '/var/www/myblog/static/'
MEDIA_ROOT = '/var/www/myblog/media/'
```

---

## สรุป Django Project Structure สมบูรณ์

```
myblog_project/
├── .env                        # Environment variables (gitignore)
├── .gitignore
├── requirements.txt            # Python dependencies
├── manage.py                   # Django management utility
│
├── config/                     # Project configuration
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py            # Base settings
│   │   ├── development.py     # Development settings
│   │   └── production.py      # Production settings
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── blog/                       # Blog app
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── signals.py
│   ├── tests.py
│   ├── urls.py
│   ├── views.py
│   ├── management/
│   │   └── commands/
│   │       └── publish_articles.py
│   ├── migrations/
│   │   ├── __init__.py
│   │   └── 0001_initial.py
│   ├── static/
│   │   └── blog/
│   │       ├── css/
│   │       └── js/
│   └── templates/
│       └── blog/
│           ├── index.html
│           ├── article_list.html
│           ├── article_detail.html
│           ├── article_form.html
│           └── search.html
│
├── templates/                  # Global templates
│   ├── base.html
│   ├── 404.html
│   └── 500.html
│
├── static/                     # Global static files
│   ├── css/
│   ├── js/
│   └── images/
│
├── media/                      # User uploaded files (gitignore)
│
├── staticfiles/                # Collected static files (gitignore)
│
└── fixtures/                   # Test/initial data
    └── initial_data.json
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Django Project พื้นฐาน

**โจทย์**: สร้าง Django project ชื่อ `myshop` พร้อม app ชื่อ `products`

```
สร้าง:
1. Django project ชื่อ myshop
2. App ชื่อ products
3. ลงทะเบียน app ใน settings.py
4. รัน development server
5. เข้าหน้า admin ได้สำเร็จ
```

**เฉลย**:

```bash
# สร้าง virtual environment
python -m venv myshop_env
source myshop_env/bin/activate

# ติดตั้ง Django
pip install django

# สร้าง project
django-admin startproject myshop
cd myshop

# สร้าง app
python manage.py startapp products

# สร้าง database
python manage.py migrate

# สร้าง superuser
python manage.py createsuperuser

# รัน server
python manage.py runserver
```

```python
# myshop/settings.py - เพิ่ม products ใน INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # เพิ่ม app ของเรา
    'products.apps.ProductsConfig',
]
```

---

### แบบฝึกหัดที่ 2: สร้าง Model สำหรับร้านค้า

**โจทย์**: สร้าง models สำหรับระบบร้านค้าออนไลน์

```
สร้าง models:
1. Category - หมวดหมู่สินค้า (name, slug, description)
2. Product - สินค้า (name, slug, category, price, stock, image, description, is_active)
3. สร้าง migrations และ apply
4. ลงทะเบียนใน admin พร้อม customization
```

**เฉลย**:

```python
# products/models.py
from django.db import models
from django.utils.text import slugify


class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True, blank=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = 'Categories'
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)


class Product(models.Model):
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, blank=True)
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='products'
    )
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    image = models.ImageField(upload_to='products/', blank=True, null=True)
    description = models.TextField(blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
    
    def is_in_stock(self):
        return self.stock > 0
    
    @property
    def price_display(self):
        return f'฿{self.price:,.2f}'
```

```python
# products/admin.py
from django.contrib import admin
from .models import Category, Product


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'product_count']
    prepopulated_fields = {'slug': ('name',)}
    
    def product_count(self, obj):
        return obj.products.count()
    product_count.short_description = 'จำนวนสินค้า'


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ['name', 'category', 'price', 'stock', 'is_active', 'is_in_stock']
    list_filter = ['is_active', 'category']
    search_fields = ['name', 'description']
    prepopulated_fields = {'slug': ('name',)}
    list_editable = ['is_active', 'stock']
    
    def is_in_stock(self, obj):
        return obj.is_in_stock()
    is_in_stock.boolean = True
    is_in_stock.short_description = 'มีสินค้า'
```

```bash
# สร้าง migrations
python manage.py makemigrations products
python manage.py migrate
```

---

### แบบฝึกหัดที่ 3: สร้าง Views และ URLs

**โจทย์**: สร้าง views สำหรับแสดงรายการสินค้าและรายละเอียด

```
สร้าง:
1. View สำหรับรายการสินค้าทั้งหมด
2. View สำหรับรายละเอียดสินค้า
3. URL patterns ที่เหมาะสม
4. Template พื้นฐาน
```

**เฉลย**:

```python
# products/views.py
from django.shortcuts import render, get_object_or_404
from .models import Product, Category


def product_list(request):
    """รายการสินค้าทั้งหมด"""
    products = Product.objects.filter(is_active=True).select_related('category')
    categories = Category.objects.all()
    
    # Filter ตาม category
    category_slug = request.GET.get('category')
    if category_slug:
        category = get_object_or_404(Category, slug=category_slug)
        products = products.filter(category=category)
    else:
        category = None
    
    context = {
        'products': products,
        'categories': categories,
        'current_category': category,
    }
    return render(request, 'products/list.html', context)


def product_detail(request, slug):
    """รายละเอียดสินค้า"""
    product = get_object_or_404(Product, slug=slug, is_active=True)
    
    # Related products
    related = Product.objects.filter(
        category=product.category,
        is_active=True
    ).exclude(pk=product.pk)[:4]
    
    context = {
        'product': product,
        'related_products': related,
    }
    return render(request, 'products/detail.html', context)
```

```python
# products/urls.py
from django.urls import path
from . import views

app_name = 'products'

urlpatterns = [
    path('', views.product_list, name='list'),
    path('<slug:slug>/', views.product_detail, name='detail'),
]
```

```python
# myshop/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('products/', include('products.urls')),
]
```

```html
<!-- products/templates/products/list.html -->
{% extends 'base.html' %}

{% block content %}
<h1>สินค้าทั้งหมด</h1>

<!-- Category Filter -->
<div class="categories">
    <a href="{% url 'products:list' %}" 
       class="{% if not current_category %}active{% endif %}">
        ทั้งหมด
    </a>
    {% for cat in categories %}
    <a href="?category={{ cat.slug }}"
       class="{% if current_category == cat %}active{% endif %}">
        {{ cat.name }}
    </a>
    {% endfor %}
</div>

<!-- Product Grid -->
<div class="product-grid">
    {% for product in products %}
    <div class="product-card">
        {% if product.image %}
            <img src="{{ product.image.url }}" alt="{{ product.name }}">
        {% endif %}
        <h3><a href="{% url 'products:detail' slug=product.slug %}">
            {{ product.name }}
        </a></h3>
        <p class="price">{{ product.price_display }}</p>
        {% if product.is_in_stock %}
            <span class="in-stock">มีสินค้า ({{ product.stock }})</span>
        {% else %}
            <span class="out-of-stock">สินค้าหมด</span>
        {% endif %}
    </div>
    {% empty %}
        <p>ไม่พบสินค้า</p>
    {% endfor %}
</div>
{% endblock %}
```

---

### แบบฝึกหัดที่ 4: การใช้ Django Shell

**โจทย์**: ใช้ Django shell เพิ่มข้อมูลตัวอย่างและ query ข้อมูล

**เฉลย**:

```python
# รัน: python manage.py shell

from products.models import Category, Product

# สร้าง Categories
electronics = Category.objects.create(
    name='อิเล็กทรอนิกส์',
    description='สินค้าอิเล็กทรอนิกส์ทุกชนิด'
)
clothing = Category.objects.create(
    name='เสื้อผ้า',
    description='เสื้อผ้าแฟชั่น'
)

# สร้าง Products
Product.objects.create(
    name='iPhone 15 Pro',
    category=electronics,
    price=39900,
    stock=50,
    description='สมาร์ทโฟน Apple รุ่นล่าสุด'
)

Product.objects.create(
    name='Samsung Galaxy S24',
    category=electronics,
    price=29900,
    stock=30
)

Product.objects.create(
    name='เสื้อยืด Basic',
    category=clothing,
    price=299,
    stock=100
)

# Query ข้อมูล
# ดูสินค้าทั้งหมด
all_products = Product.objects.all()
print(f"สินค้าทั้งหมด: {all_products.count()} รายการ")

# Filter สินค้าราคาต่ำกว่า 1000 บาท
cheap = Product.objects.filter(price__lt=1000)
print(f"สินค้าราคาต่ำกว่า 1000 บาท: {cheap.count()} รายการ")

# Filter สินค้าที่มีสต็อก
in_stock = Product.objects.filter(stock__gt=0)
for product in in_stock:
    print(f"  - {product.name}: {product.stock} ชิ้น")

# Aggregate
from django.db.models import Avg, Max, Min, Sum
stats = Product.objects.aggregate(
    avg_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price'),
    total_value=Sum('price')
)
print(f"ราคาเฉลี่ย: {stats['avg_price']:.2f}")
print(f"ราคาสูงสุด: {stats['max_price']}")
print(f"ราคาต่ำสุด: {stats['min_price']}")
```

---

### แบบฝึกหัดที่ 5: การตั้งค่า Static Files

**โจทย์**: ตั้งค่า static files ให้ถูกต้องและใช้งาน CSS ใน template

**เฉลย**:

```python
# settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
STATIC_ROOT = BASE_DIR / 'staticfiles'
```

```bash
# สร้าง directories
mkdir -p static/css static/js static/images
```

```css
/* static/css/style.css */
:root {
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --dark-color: #2c3e50;
    --light-color: #ecf0f1;
}

body {
    font-family: 'Sarabun', sans-serif;
    margin: 0;
    padding: 0;
    background-color: var(--light-color);
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
}

.product-card {
    background: white;
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    transition: transform 0.2s;
}

.product-card:hover {
    transform: translateY(-5px);
}

.price {
    font-size: 1.2em;
    font-weight: bold;
    color: var(--primary-color);
}

.in-stock { color: var(--secondary-color); }
.out-of-stock { color: #e74c3c; }
```

```html
<!-- templates/base.html -->
{% load static %}
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}MyShop{% endblock %}</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        <div class="container">
            <h1><a href="/">MyShop</a></h1>
            <nav>
                <a href="{% url 'products:list' %}">สินค้า</a>
            </nav>
        </div>
    </header>
    
    <main class="container">
        {% block content %}{% endblock %}
    </main>
    
    <script src="{% static 'js/main.js' %}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

---

### แบบฝึกหัดที่ 6: Custom Management Command

**โจทย์**: สร้าง management command `create_sample_data` สำหรับสร้างข้อมูลตัวอย่าง

**เฉลย**:

```python
# products/management/__init__.py  (empty file)
# products/management/commands/__init__.py  (empty file)
# products/management/commands/create_sample_data.py

from django.core.management.base import BaseCommand
from products.models import Category, Product
import random


class Command(BaseCommand):
    help = 'สร้างข้อมูลตัวอย่างสำหรับร้านค้า'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--products',
            type=int,
            default=20,
            help='จำนวนสินค้าที่ต้องการสร้าง (default: 20)'
        )
        parser.add_argument(
            '--clear',
            action='store_true',
            help='ลบข้อมูลเดิมก่อนสร้างใหม่'
        )
    
    def handle(self, *args, **options):
        if options['clear']:
            Product.objects.all().delete()
            Category.objects.all().delete()
            self.stdout.write(self.style.WARNING('ลบข้อมูลเดิมแล้ว'))
        
        # สร้าง categories
        categories_data = [
            ('อิเล็กทรอนิกส์', 'electronic'),
            ('เสื้อผ้า', 'clothing'),
            ('อาหาร', 'food'),
            ('หนังสือ', 'book'),
        ]
        
        categories = []
        for name, slug in categories_data:
            cat, created = Category.objects.get_or_create(
                slug=slug,
                defaults={'name': name, 'description': f'สินค้าหมวด{name}'}
            )
            categories.append(cat)
            if created:
                self.stdout.write(f'สร้าง category: {name}')
        
        # สร้าง products
        product_count = options['products']
        created_count = 0
        
        for i in range(1, product_count + 1):
            category = random.choice(categories)
            price = random.choice([99, 199, 299, 499, 999, 1499, 1999, 2999])
            stock = random.randint(0, 100)
            
            product, created = Product.objects.get_or_create(
                name=f'สินค้า {category.name} #{i:03d}',
                defaults={
                    'category': category,
                    'price': price,
                    'stock': stock,
                    'description': f'สินค้าตัวอย่างในหมวด{category.name}',
                    'is_active': random.choice([True, True, True, False]),
                }
            )
            
            if created:
                created_count += 1
        
        self.stdout.write(
            self.style.SUCCESS(
                f'สร้างข้อมูลสำเร็จ: {len(categories)} หมวดหมู่, {created_count} สินค้า'
            )
        )
```

```bash
# รัน command
python manage.py create_sample_data
python manage.py create_sample_data --products=50
python manage.py create_sample_data --clear --products=30
```

---

### แบบฝึกหัดที่ 7: การใช้ Django Admin อย่างละเอียด

**โจทย์**: ปรับแต่ง Admin Interface ให้สมบูรณ์

**เฉลย**:

```python
# products/admin.py
from django.contrib import admin
from django.utils.html import format_html
from django.db.models import Count, Sum
from .models import Category, Product


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'product_count', 'active_product_count']
    prepopulated_fields = {'slug': ('name',)}
    search_fields = ['name']
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.annotate(
            _product_count=Count('products'),
            _active_count=Count('products', filter=Count('products__is_active'))
        )
    
    def product_count(self, obj):
        return obj.products.count()
    product_count.short_description = 'สินค้าทั้งหมด'
    product_count.admin_order_field = '_product_count'
    
    def active_product_count(self, obj):
        return obj.products.filter(is_active=True).count()
    active_product_count.short_description = 'สินค้าที่ active'


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = [
        'name', 'category', 'price_display', 'stock_display',
        'is_active', 'created_at'
    ]
    list_filter = ['is_active', 'category', 'created_at']
    search_fields = ['name', 'description']
    prepopulated_fields = {'slug': ('name',)}
    list_editable = ['is_active']
    list_per_page = 25
    date_hierarchy = 'created_at'
    
    fieldsets = (
        ('ข้อมูลพื้นฐาน', {
            'fields': ('name', 'slug', 'category', 'description')
        }),
        ('ราคาและสต็อก', {
            'fields': ('price', 'stock')
        }),
        ('รูปภาพ', {
            'fields': ('image',),
            'classes': ('collapse',),
        }),
        ('การตั้งค่า', {
            'fields': ('is_active',)
        }),
    )
    
    readonly_fields = ['created_at', 'updated_at']
    
    actions = ['make_active', 'make_inactive', 'restock']
    
    def price_display(self, obj):
        return format_html(
            '<strong style="color: #3498db;">฿{:,.2f}</strong>',
            obj.price
        )
    price_display.short_description = 'ราคา'
    price_display.admin_order_field = 'price'
    
    def stock_display(self, obj):
        if obj.stock == 0:
            return format_html(
                '<span style="color: red;">หมด</span>'
            )
        elif obj.stock < 10:
            return format_html(
                '<span style="color: orange;">เหลือ {} ชิ้น</span>',
                obj.stock
            )
        return format_html(
            '<span style="color: green;">{} ชิ้น</span>',
            obj.stock
        )
    stock_display.short_description = 'สต็อก'
    stock_display.admin_order_field = 'stock'
    
    def make_active(self, request, queryset):
        updated = queryset.update(is_active=True)
        self.message_user(request, f'เปิดใช้งาน {updated} สินค้าแล้ว')
    make_active.short_description = 'เปิดใช้งานสินค้าที่เลือก'
    
    def make_inactive(self, request, queryset):
        updated = queryset.update(is_active=False)
        self.message_user(request, f'ปิดใช้งาน {updated} สินค้าแล้ว')
    make_inactive.short_description = 'ปิดใช้งานสินค้าที่เลือก'
    
    def restock(self, request, queryset):
        updated = queryset.filter(stock=0).update(stock=10)
        self.message_user(request, f'เติมสต็อก {updated} สินค้า (10 ชิ้น/สินค้า)')
    restock.short_description = 'เติมสต็อกสินค้าที่หมด'
```

---

### แบบฝึกหัดที่ 8: สร้าง Complete Mini Blog

**โจทย์**: สร้าง mini blog ที่สมบูรณ์ตั้งแต่ต้นจนจบ

**เฉลย**:

```bash
# ขั้นตอนที่ 1: สร้าง project และ app
django-admin startproject miniblog
cd miniblog
python manage.py startapp posts

# ขั้นตอนที่ 2: แก้ไข settings.py
# เพิ่ม 'posts.apps.PostsConfig' ใน INSTALLED_APPS
```

```python
# posts/models.py
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify


class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, blank=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    body = models.TextField()
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    active = models.BooleanField(default=True)
    
    class Meta:
        ordering = ['-created']
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)
```

```python
# posts/views.py
from django.shortcuts import render, get_object_or_404
from .models import Post


def post_list(request):
    posts = Post.objects.filter(active=True)
    return render(request, 'posts/list.html', {'posts': posts})


def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, active=True)
    return render(request, 'posts/detail.html', {'post': post})
```

```python
# posts/urls.py
from django.urls import path
from . import views

app_name = 'posts'
urlpatterns = [
    path('', views.post_list, name='list'),
    path('<slug:slug>/', views.post_detail, name='detail'),
]
```

```python
# miniblog/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('posts/', include('posts.urls')),
    path('', include('posts.urls')),
]
```

```python
# posts/admin.py
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'active', 'created']
    list_filter = ['active', 'created']
    search_fields = ['title', 'body']
    prepopulated_fields = {'slug': ('title',)}
    list_editable = ['active']
```

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}MiniBlog{% endblock %}</title>
    <style>
        body { font-family: sans-serif; max-width: 800px; margin: 40px auto; padding: 0 20px; }
        .post-card { border: 1px solid #ddd; border-radius: 8px; padding: 20px; margin: 20px 0; }
        .post-card h2 a { text-decoration: none; color: #2c3e50; }
        .meta { color: #888; font-size: 0.9em; }
    </style>
</head>
<body>
    <header>
        <h1><a href="/">MiniBlog</a></h1>
        <hr>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

```html
<!-- posts/templates/posts/list.html -->
{% extends 'base.html' %}

{% block title %}บทความทั้งหมด - MiniBlog{% endblock %}

{% block content %}
<h2>บทความทั้งหมด</h2>
{% for post in posts %}
    <div class="post-card">
        <h2><a href="{% url 'posts:detail' slug=post.slug %}">{{ post.title }}</a></h2>
        <p class="meta">
            โดย {{ post.author.get_full_name|default:post.author.username }} |
            {{ post.created|date:"d M Y" }}
        </p>
        <p>{{ post.body|truncatewords:30 }}</p>
        <a href="{% url 'posts:detail' slug=post.slug %}">อ่านต่อ →</a>
    </div>
{% empty %}
    <p>ยังไม่มีบทความ</p>
{% endfor %}
{% endblock %}
```

```html
<!-- posts/templates/posts/detail.html -->
{% extends 'base.html' %}

{% block title %}{{ post.title }} - MiniBlog{% endblock %}

{% block content %}
<article>
    <h1>{{ post.title }}</h1>
    <p class="meta">
        โดย {{ post.author.get_full_name|default:post.author.username }} |
        {{ post.created|date:"d M Y H:i" }}
    </p>
    <hr>
    <div class="content">
        {{ post.body|linebreaks }}
    </div>
</article>
<a href="{% url 'posts:list' %}">← กลับไปยังรายการบทความ</a>
{% endblock %}
```

```bash
# รัน project
python manage.py makemigrations posts
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver

# เข้า admin และเพิ่มบทความที่: http://127.0.0.1:8000/admin/
# ดูบทความที่: http://127.0.0.1:8000/posts/
```

---

## สรุปสิ่งที่เรียนรู้

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Django Getting Started ครอบคลุม:

1. **Django Philosophy** - หลักการ Batteries Included, DRY, Explicit, Loose Coupling
2. **การเปรียบเทียบ** Django vs Flask vs FastAPI
3. **การติดตั้งและสร้าง Project** ด้วย django-admin startproject
4. **Project Structure** ทุกไฟล์ที่สำคัญ (manage.py, settings.py, urls.py, wsgi.py, asgi.py)
5. **Apps Concept** การแบ่ง project เป็น apps
6. **settings.py** อย่างละเอียดทุก configuration
7. **urls.py Patterns** การกำหนด URL routing
8. **Development Server** การรัน runserver
9. **Django Admin** การตั้งค่าและ customize admin interface
10. **Template System** Django Template Language (DTL)
11. **Static Files** การจัดการ CSS, JS, Images
12. **Management Commands** คำสั่งที่ใช้บ่อย + Custom commands

### ขั้นตอนต่อไป (Part 63)

ใน Part ถัดไปเราจะเรียนรู้เรื่อง **Django Models & ORM** ซึ่งครอบคลุม:
- Django ORM อย่างละเอียด
- Field types ทั้งหมด
- Relationships (ForeignKey, ManyToMany, OneToOne)
- QuerySet API
- Database migrations
- Model Managers
- Signals

---

## แหล่งข้อมูลเพิ่มเติม

- [Django Official Documentation](https://docs.djangoproject.com/)
- [Django Tutorial (Official)](https://docs.djangoproject.com/en/5.0/intro/tutorial01/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Girls Tutorial](https://tutorial.djangogirls.org/)
- [Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)
- [Django Packages](https://djangopackages.org/)
