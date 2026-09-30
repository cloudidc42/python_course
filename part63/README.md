# Part 63: Django - Models, ORM & Migrations

## สารบัญ (Table of Contents)

1. [บทนำ Django Models](#บทนำ-django-models)
2. [การติดตั้งและตั้งค่า Django](#การติดตั้งและตั้งค่า-django)
3. [Django Models พื้นฐาน](#django-models-พื้นฐาน)
4. [Field Types ทั้งหมด](#field-types-ทั้งหมด)
5. [Field Options](#field-options)
6. [Model Meta Class](#model-meta-class)
7. [Relationships ระหว่าง Models](#relationships-ระหว่าง-models)
8. [Django ORM Queryset API](#django-orm-queryset-api)
9. [Aggregate Functions](#aggregate-functions)
10. [annotate() และ aggregate()](#annotate-และ-aggregate)
11. [Q Objects สำหรับ Complex Queries](#q-objects-สำหรับ-complex-queries)
12. [F Objects สำหรับ Field References](#f-objects-สำหรับ-field-references)
13. [select_related() และ prefetch_related()](#select_related-และ-prefetch_related)
14. [Migrations](#migrations)
15. [Data Migrations](#data-migrations)
16. [Advanced ORM Techniques](#advanced-orm-techniques)
17. [แบบฝึกหัด](#แบบฝึกหัด)
18. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## บทนำ Django Models

Django Models คือหัวใจสำคัญของ Django Framework ที่ทำหน้าที่เป็นชั้น **Data Access Layer (DAL)** หรือที่เราเรียกว่า **ORM (Object-Relational Mapping)** ซึ่งช่วยให้เราสามารถทำงานกับฐานข้อมูลผ่านทาง Python objects โดยไม่ต้องเขียน SQL โดยตรง

### แนวคิดหลักของ Django ORM

- **Model** = Python class ที่ map กับตารางในฐานข้อมูล
- **Field** = attribute ของ class ที่ map กับ column ในตาราง
- **Instance** = object ที่ map กับแถว (row) ในตาราง
- **QuerySet** = ชุดของ objects ที่ได้จากการ query ฐานข้อมูล

### ข้อดีของ Django ORM

1. **Database Abstraction** - รองรับ MySQL, PostgreSQL, SQLite, Oracle ได้โดยไม่ต้องเปลี่ยน code
2. **Security** - ป้องกัน SQL Injection อัตโนมัติ
3. **Productivity** - เขียน Python แทน SQL ทำให้พัฒนาได้เร็วขึ้น
4. **Migrations** - ติดตามการเปลี่ยนแปลง schema อัตโนมัติ

---

## การติดตั้งและตั้งค่า Django

### ติดตั้ง Django

```bash
# สร้าง virtual environment
python -m venv django_env
source django_env/bin/activate  # Linux/Mac
# django_env\Scripts\activate  # Windows

# ติดตั้ง Django
pip install django

# ตรวจสอบ version
python -m django --version
```

### สร้าง Django Project

```bash
# สร้าง project ใหม่
django-admin startproject myproject
cd myproject

# สร้าง app ใหม่
python manage.py startapp myapp

# โครงสร้างไฟล์ที่ได้
# myproject/
# ├── manage.py
# ├── myproject/
# │   ├── __init__.py
# │   ├── settings.py
# │   ├── urls.py
# │   └── wsgi.py
# └── myapp/
#     ├── __init__.py
#     ├── admin.py
#     ├── apps.py
#     ├── migrations/
#     │   └── __init__.py
#     ├── models.py
#     ├── tests.py
#     └── views.py
```

### ตั้งค่า settings.py

```python
# myproject/settings.py

# เพิ่ม app ที่สร้างขึ้นใน INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # เพิ่ม app ของเรา
]

# ตั้งค่า Database (SQLite สำหรับ development)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# ตั้งค่า PostgreSQL (สำหรับ production)
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.postgresql',
#         'NAME': 'mydb',
#         'USER': 'myuser',
#         'PASSWORD': 'mypassword',
#         'HOST': 'localhost',
#         'PORT': '5432',
#     }
# }
```

---

## Django Models พื้นฐาน

### ตัวอย่างที่ 1: Model พื้นฐาน

```python
# myapp/models.py
from django.db import models


class Article(models.Model):
    """Model สำหรับบทความ"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=False)
    views = models.IntegerField(default=0)

    def __str__(self):
        return self.title

    class Meta:
        ordering = ['-published_date']
        verbose_name = 'บทความ'
        verbose_name_plural = 'บทความทั้งหมด'
```

### ตัวอย่างที่ 2: Model กับ Custom Methods

```python
# myapp/models.py
from django.db import models
from django.utils import timezone


class Product(models.Model):
    """Model สำหรับสินค้า"""
    name = models.CharField(max_length=100, verbose_name='ชื่อสินค้า')
    description = models.TextField(blank=True, verbose_name='รายละเอียด')
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.name} - ฿{self.price}"

    def is_in_stock(self):
        """ตรวจสอบว่ามีสินค้าในคลังหรือไม่"""
        return self.stock > 0

    def discount_price(self, percent):
        """คำนวณราคาหลังหักส่วนลด"""
        discount = self.price * (percent / 100)
        return self.price - discount

    @property
    def stock_status(self):
        """Property สำหรับแสดงสถานะสินค้า"""
        if self.stock == 0:
            return 'หมด'
        elif self.stock < 10:
            return 'ใกล้หมด'
        else:
            return 'มีสินค้า'
```

### ตัวอย่างที่ 3: Abstract Model (Base Model)

```python
# myapp/models.py
from django.db import models


class TimestampedModel(models.Model):
    """Abstract model สำหรับ timestamp fields"""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True  # ไม่สร้างตารางในฐานข้อมูล


class Post(TimestampedModel):
    """Model สำหรับโพสต์ - สืบทอดจาก TimestampedModel"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.title


class Comment(TimestampedModel):
    """Model สำหรับความคิดเห็น - สืบทอดจาก TimestampedModel"""
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    text = models.TextField()
    author_name = models.CharField(max_length=100)

    def __str__(self):
        return f"ความคิดเห็นจาก {self.author_name}"
```

---

## Field Types ทั้งหมด

Django มี Field Types หลากหลายเพื่อรองรับข้อมูลประเภทต่างๆ

### ตัวอย่างที่ 4: Text Fields

```python
from django.db import models


class TextFieldsDemo(models.Model):
    """ตัวอย่าง Text Field ประเภทต่างๆ"""

    # CharField - สำหรับข้อความสั้น (ต้องระบุ max_length)
    name = models.CharField(max_length=100)

    # TextField - สำหรับข้อความยาว
    description = models.TextField()

    # SlugField - สำหรับ URL-friendly text
    slug = models.SlugField(max_length=100, unique=True)

    # EmailField - สำหรับ email address
    email = models.EmailField(unique=True)

    # URLField - สำหรับ URL
    website = models.URLField(blank=True)

    # IPAddressField - สำหรับ IP address
    ip_address = models.GenericIPAddressField(null=True, blank=True)

    # UUIDField - สำหรับ UUID
    import uuid
    unique_id = models.UUIDField(default=uuid.uuid4, editable=False)

    def __str__(self):
        return self.name
```

### ตัวอย่างที่ 5: Numeric Fields

```python
from django.db import models


class NumericFieldsDemo(models.Model):
    """ตัวอย่าง Numeric Field ประเภทต่างๆ"""

    # IntegerField - จำนวนเต็ม (-2147483648 ถึง 2147483647)
    age = models.IntegerField()

    # BigIntegerField - จำนวนเต็มขนาดใหญ่
    big_number = models.BigIntegerField()

    # SmallIntegerField - จำนวนเต็มขนาดเล็ก (-32768 ถึง 32767)
    small_count = models.SmallIntegerField()

    # PositiveIntegerField - จำนวนเต็มบวก
    views = models.PositiveIntegerField(default=0)

    # FloatField - ทศนิยม (อาจมี precision issues)
    rating = models.FloatField()

    # DecimalField - ทศนิยมแบบแม่นยำ (ใช้สำหรับเงิน)
    price = models.DecimalField(max_digits=10, decimal_places=2)

    # AutoField - primary key อัตโนมัติ (Django สร้างให้อัตโนมัติ)
    # id = models.AutoField(primary_key=True)  # Django สร้างให้แล้ว

    # BigAutoField - primary key แบบ big integer
    # id = models.BigAutoField(primary_key=True)

    def __str__(self):
        return f"Numeric Demo {self.id}"
```

### ตัวอย่างที่ 6: Date and Time Fields

```python
from django.db import models
from django.utils import timezone


class DateTimeFieldsDemo(models.Model):
    """ตัวอย่าง Date/Time Field ประเภทต่างๆ"""

    # DateField - เก็บวันที่ (YYYY-MM-DD)
    birth_date = models.DateField()

    # TimeField - เก็บเวลา (HH:MM:SS)
    start_time = models.TimeField()

    # DateTimeField - เก็บวันที่และเวลา
    event_datetime = models.DateTimeField()

    # DateTimeField กับ auto_now_add - บันทึกเวลาสร้างครั้งแรกเท่านั้น
    created_at = models.DateTimeField(auto_now_add=True)

    # DateTimeField กับ auto_now - อัปเดตทุกครั้งที่ save
    updated_at = models.DateTimeField(auto_now=True)

    # DurationField - เก็บระยะเวลา (timedelta)
    duration = models.DurationField(null=True, blank=True)

    def __str__(self):
        return f"DateTime Demo {self.id}"
```

### ตัวอย่างที่ 7: Boolean and File Fields

```python
from django.db import models


class BooleanFileFieldsDemo(models.Model):
    """ตัวอย่าง Boolean และ File Field"""

    # BooleanField - True/False
    is_active = models.BooleanField(default=True)

    # NullBooleanField - True/False/None (deprecated ใน Django 4.0+)
    # ใช้ BooleanField(null=True) แทน
    is_verified = models.BooleanField(null=True, blank=True)

    # FileField - เก็บ path ของไฟล์
    document = models.FileField(upload_to='documents/', null=True, blank=True)

    # ImageField - เก็บ path ของรูปภาพ (ต้องติดตั้ง Pillow)
    profile_picture = models.ImageField(
        upload_to='profiles/',
        null=True,
        blank=True
    )

    # FilePathField - เก็บ path ที่กำหนดเอง
    log_file = models.FilePathField(path='/var/log/', null=True, blank=True)

    def __str__(self):
        return f"BooleanFile Demo {self.id}"
```

### ตัวอย่างที่ 8: JSON and Binary Fields

```python
from django.db import models


class AdvancedFieldsDemo(models.Model):
    """ตัวอย่าง Advanced Field ประเภทต่างๆ"""

    # JSONField - เก็บข้อมูล JSON (Django 3.1+)
    metadata = models.JSONField(default=dict)
    settings_data = models.JSONField(null=True, blank=True)

    # BinaryField - เก็บข้อมูล binary
    binary_data = models.BinaryField(null=True, blank=True)

    # ตัวอย่างการใช้ JSONField
    # product.metadata = {"color": "red", "size": "XL", "tags": ["sale", "new"]}

    def get_setting(self, key, default=None):
        """ดึงค่า setting จาก JSONField"""
        if self.settings_data is None:
            return default
        return self.settings_data.get(key, default)

    def __str__(self):
        return f"Advanced Fields Demo {self.id}"
```

---

## Field Options

Field Options คือ arguments ที่ส่งให้กับ Field เพื่อกำหนดพฤติกรรมและ validation

### ตัวอย่างที่ 9: Field Options พื้นฐาน

```python
from django.db import models


class FieldOptionsDemo(models.Model):
    """ตัวอย่าง Field Options ต่างๆ"""

    # null=True - อนุญาตให้เก็บ NULL ในฐานข้อมูล
    middle_name = models.CharField(max_length=100, null=True)

    # blank=True - อนุญาตให้ form validation ผ่านเมื่อไม่มีค่า
    bio = models.TextField(blank=True)

    # null=True, blank=True - ใช้ร่วมกันสำหรับ optional fields
    website = models.URLField(null=True, blank=True)

    # default - ค่าเริ่มต้น
    score = models.IntegerField(default=0)
    is_active = models.BooleanField(default=True)
    status = models.CharField(max_length=20, default='pending')

    # unique=True - ต้องไม่ซ้ำกันในทั้งตาราง
    username = models.CharField(max_length=50, unique=True)
    email = models.EmailField(unique=True)

    # verbose_name - ชื่อที่แสดงใน admin และ forms
    first_name = models.CharField(max_length=50, verbose_name='ชื่อ')
    last_name = models.CharField(max_length=50, verbose_name='นามสกุล')

    # db_column - กำหนดชื่อ column ในฐานข้อมูล
    phone = models.CharField(max_length=20, db_column='phone_number')

    # db_index=True - สร้าง index สำหรับ column นี้
    category = models.CharField(max_length=50, db_index=True)

    # editable=False - ไม่แสดงใน forms และ admin
    internal_code = models.CharField(max_length=10, editable=False)

    # primary_key=True - กำหนดเป็น primary key
    # (ถ้าไม่ระบุ Django จะสร้าง id field อัตโนมัติ)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"
```

### ตัวอย่างที่ 10: Choices Field Option

```python
from django.db import models


class OrderStatus(models.TextChoices):
    """Choices สำหรับสถานะออเดอร์ (Django 3.0+)"""
    PENDING = 'pending', 'รอดำเนินการ'
    PROCESSING = 'processing', 'กำลังดำเนินการ'
    SHIPPED = 'shipped', 'จัดส่งแล้ว'
    DELIVERED = 'delivered', 'ส่งถึงแล้ว'
    CANCELLED = 'cancelled', 'ยกเลิก'


class PaymentMethod(models.IntegerChoices):
    """Choices แบบ Integer"""
    CASH = 1, 'เงินสด'
    CREDIT_CARD = 2, 'บัตรเครดิต'
    BANK_TRANSFER = 3, 'โอนเงิน'
    QR_CODE = 4, 'QR Code'


class Order(models.Model):
    """Model สำหรับออเดอร์"""

    # ใช้ TextChoices
    status = models.CharField(
        max_length=20,
        choices=OrderStatus.choices,
        default=OrderStatus.PENDING,
        verbose_name='สถานะ'
    )

    # ใช้ IntegerChoices
    payment_method = models.IntegerField(
        choices=PaymentMethod.choices,
        default=PaymentMethod.CASH,
        verbose_name='วิธีชำระเงิน'
    )

    # Choices แบบ tuple (วิธีเก่า)
    PRIORITY_CHOICES = [
        ('low', 'ต่ำ'),
        ('medium', 'ปานกลาง'),
        ('high', 'สูง'),
        ('urgent', 'เร่งด่วน'),
    ]
    priority = models.CharField(
        max_length=10,
        choices=PRIORITY_CHOICES,
        default='medium'
    )

    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"Order #{self.id} - {self.get_status_display()}"

    def is_completed(self):
        return self.status == OrderStatus.DELIVERED
```

### ตัวอย่างที่ 11: Validators

```python
from django.db import models
from django.core.validators import (
    MinValueValidator,
    MaxValueValidator,
    MinLengthValidator,
    RegexValidator
)


class ValidatedModel(models.Model):
    """ตัวอย่างการใช้ Validators"""

    # MinValueValidator และ MaxValueValidator
    age = models.IntegerField(
        validators=[
            MinValueValidator(0, message='อายุต้องไม่น้อยกว่า 0'),
            MaxValueValidator(150, message='อายุต้องไม่เกิน 150')
        ]
    )

    # MinLengthValidator
    username = models.CharField(
        max_length=50,
        validators=[MinLengthValidator(3, message='username ต้องมีอย่างน้อย 3 ตัวอักษร')]
    )

    # RegexValidator
    phone = models.CharField(
        max_length=15,
        validators=[
            RegexValidator(
                regex=r'^\+?1?\d{9,15}$',
                message='กรุณากรอกเบอร์โทรศัพท์ที่ถูกต้อง'
            )
        ]
    )

    # DecimalField กับ range validation
    score = models.DecimalField(
        max_digits=5,
        decimal_places=2,
        validators=[
            MinValueValidator(0.00),
            MaxValueValidator(100.00)
        ]
    )

    def __str__(self):
        return self.username
```

---

## Model Meta Class

`Meta` class ใน Django Model ใช้สำหรับกำหนด metadata และพฤติกรรมของ Model

### ตัวอย่างที่ 12: Model Meta Options

```python
from django.db import models


class Employee(models.Model):
    """Model สำหรับพนักงาน พร้อม Meta options"""
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    department = models.CharField(max_length=100)
    salary = models.DecimalField(max_digits=10, decimal_places=2)
    hire_date = models.DateField()
    is_active = models.BooleanField(default=True)

    class Meta:
        # กำหนดชื่อตารางในฐานข้อมูล (ค่าเริ่มต้น: appname_modelname)
        db_table = 'employees'

        # กำหนดการเรียงลำดับเริ่มต้น
        # - ขึ้นต้นด้วย '-' หมายถึง descending
        ordering = ['last_name', 'first_name']

        # กำหนดชื่อที่แสดงใน Admin (เอกพจน์)
        verbose_name = 'พนักงาน'

        # กำหนดชื่อที่แสดงใน Admin (พหูพจน์)
        verbose_name_plural = 'พนักงานทั้งหมด'

        # unique_together - กำหนด unique constraint สำหรับหลาย fields
        unique_together = [['first_name', 'last_name', 'department']]

        # indexes - สร้าง database indexes
        indexes = [
            models.Index(fields=['last_name', 'first_name'], name='name_idx'),
            models.Index(fields=['department'], name='dept_idx'),
        ]

        # constraints - สร้าง database constraints (Django 2.2+)
        constraints = [
            models.CheckConstraint(
                check=models.Q(salary__gte=0),
                name='salary_non_negative'
            )
        ]

        # permissions - กำหนด custom permissions
        permissions = [
            ('can_view_salary', 'สามารถดูเงินเดือนได้'),
            ('can_approve_leave', 'สามารถอนุมัติวันลาได้'),
        ]

    def __str__(self):
        return f"{self.first_name} {self.last_name}"

    def full_name(self):
        return f"{self.first_name} {self.last_name}"
```

### ตัวอย่างที่ 13: Proxy Models

```python
from django.db import models


class Person(models.Model):
    """Base Model"""
    name = models.CharField(max_length=100)
    email = models.EmailField()
    birth_date = models.DateField()
    is_staff = models.BooleanField(default=False)
    is_customer = models.BooleanField(default=True)

    def __str__(self):
        return self.name


class StaffMember(Person):
    """Proxy Model สำหรับพนักงาน - ใช้ตารางเดียวกันกับ Person"""

    class Meta:
        proxy = True
        ordering = ['name']
        verbose_name = 'พนักงาน'

    def send_staff_email(self):
        """Method เฉพาะสำหรับ StaffMember"""
        print(f"ส่ง email ถึงพนักงาน {self.email}")


class Customer(Person):
    """Proxy Model สำหรับลูกค้า - ใช้ตารางเดียวกันกับ Person"""

    class Meta:
        proxy = True
        ordering = ['-birth_date']
        verbose_name = 'ลูกค้า'

    def send_promotional_email(self):
        """Method เฉพาะสำหรับ Customer"""
        print(f"ส่งโปรโมชั่นให้ {self.email}")
```

---

## Relationships ระหว่าง Models

### ตัวอย่างที่ 14: ForeignKey (Many-to-One)

```python
from django.db import models


class Category(models.Model):
    """หมวดหมู่สินค้า"""
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.name


class Article(models.Model):
    """บทความ - มี ForeignKey ไปยัง Category"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,       # ลบบทความเมื่อลบหมวดหมู่
        related_name='articles',        # ชื่อสำหรับ reverse relation
        verbose_name='หมวดหมู่'
    )
    author_name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title


# on_delete options:
# CASCADE      - ลบ related objects ด้วย
# PROTECT      - ป้องกันการลบถ้ายังมี related objects
# SET_NULL     - ตั้งค่าเป็น NULL (ต้องมี null=True)
# SET_DEFAULT  - ตั้งค่าเป็น default value
# SET()        - ตั้งค่าเป็นค่าที่กำหนด
# DO_NOTHING   - ไม่ทำอะไร (อาจ raise IntegrityError)


# การใช้งาน:
# category = Category.objects.get(id=1)
# articles = category.articles.all()  # ใช้ related_name
# article.category  # ForeignKey object
# article.category_id  # ID ของ ForeignKey
```

### ตัวอย่างที่ 15: ManyToManyField

```python
from django.db import models


class Tag(models.Model):
    """แท็ก"""
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.name


class Post(models.Model):
    """โพสต์ - มี ManyToMany กับ Tag"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    tags = models.ManyToManyField(
        Tag,
        blank=True,
        related_name='posts',
        verbose_name='แท็ก'
    )
    published = models.BooleanField(default=False)

    def __str__(self):
        return self.title


# การใช้งาน ManyToMany:
# post = Post.objects.get(id=1)
# tag1 = Tag.objects.get(name='python')
# tag2 = Tag.objects.get(name='django')

# เพิ่ม tags
# post.tags.add(tag1, tag2)

# ลบ tag
# post.tags.remove(tag1)

# ตั้งค่า tags ใหม่ทั้งหมด
# post.tags.set([tag1, tag2])

# ลบทุก tags
# post.tags.clear()

# ดู posts ทั้งหมดของ tag
# tag1.posts.all()


class ManyToManyWithThrough(models.Model):
    """ตัวอย่าง ManyToMany กับ Through model"""
    pass


class Student(models.Model):
    """นักเรียน"""
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(
        'Course',
        through='Enrollment',
        related_name='students'
    )

    def __str__(self):
        return self.name


class Course(models.Model):
    """วิชาเรียน"""
    name = models.CharField(max_length=200)
    code = models.CharField(max_length=10, unique=True)

    def __str__(self):
        return f"{self.code} - {self.name}"


class Enrollment(models.Model):
    """ตาราง Through สำหรับการลงทะเบียนเรียน"""
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_date = models.DateField(auto_now_add=True)
    grade = models.CharField(max_length=2, blank=True)
    is_active = models.BooleanField(default=True)

    class Meta:
        unique_together = ['student', 'course']

    def __str__(self):
        return f"{self.student} - {self.course}"
```

### ตัวอย่างที่ 16: OneToOneField

```python
from django.db import models
from django.contrib.auth.models import User


class UserProfile(models.Model):
    """Profile เพิ่มเติมของ User - OneToOne กับ User model"""
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
        verbose_name='ผู้ใช้'
    )
    bio = models.TextField(blank=True, verbose_name='ประวัติย่อ')
    avatar = models.ImageField(
        upload_to='avatars/',
        null=True,
        blank=True
    )
    phone = models.CharField(max_length=20, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    website = models.URLField(blank=True)

    def __str__(self):
        return f"Profile ของ {self.user.username}"

    @property
    def full_name(self):
        return f"{self.user.first_name} {self.user.last_name}"


# การใช้งาน:
# user = User.objects.get(username='john')
# profile = user.profile  # ใช้ related_name
# user.profile.bio  # เข้าถึง profile fields

# สร้าง UserProfile อัตโนมัติเมื่อสร้าง User ใหม่
from django.db.models.signals import post_save
from django.dispatch import receiver


@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)


@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### ตัวอย่างที่ 17: Self-Referential Relationships

```python
from django.db import models


class Category(models.Model):
    """หมวดหมู่แบบ tree structure"""
    name = models.CharField(max_length=100)
    parent = models.ForeignKey(
        'self',                         # อ้างอิงตัวเอง
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',        # ชื่อสำหรับ child categories
        verbose_name='หมวดหมู่หลัก'
    )

    def __str__(self):
        return self.name

    def get_ancestors(self):
        """ดึง ancestors ทั้งหมด"""
        ancestors = []
        parent = self.parent
        while parent:
            ancestors.append(parent)
            parent = parent.parent
        return ancestors[::-1]  # เรียงจาก root

    def get_children(self):
        """ดึง children ทั้งหมด"""
        return self.children.all()

    class Meta:
        verbose_name = 'หมวดหมู่'
        verbose_name_plural = 'หมวดหมู่'


class Employee(models.Model):
    """พนักงานแบบมีผู้จัดการ"""
    name = models.CharField(max_length=100)
    manager = models.ForeignKey(
        'self',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='subordinates'
    )

    def __str__(self):
        return self.name
```

---

## Django ORM Queryset API

QuerySet คือ collection ของ objects จาก database ที่สามารถ filter, order, และ slice ได้

### ตัวอย่างที่ 18: Basic QuerySet Methods

```python
from myapp.models import Article, Category

# all() - ดึงทุก records
all_articles = Article.objects.all()
print(f"จำนวนบทความทั้งหมด: {all_articles.count()}")

# get() - ดึง record เดียว (raise exception ถ้าไม่พบหรือพบมากกว่า 1)
try:
    article = Article.objects.get(id=1)
    print(f"บทความ: {article.title}")
except Article.DoesNotExist:
    print("ไม่พบบทความ")
except Article.MultipleObjectsReturned:
    print("พบมากกว่า 1 บทความ")

# filter() - กรองตาม conditions
published_articles = Article.objects.filter(is_published=True)
recent_articles = Article.objects.filter(
    is_published=True,
    category__name='Python'
)

# exclude() - ยกเว้นที่ตรงกับ conditions
unpublished = Article.objects.exclude(is_published=True)
non_python = Article.objects.exclude(category__name='Python')

# first() และ last() - ดึง record แรก/สุดท้าย
first_article = Article.objects.first()
last_article = Article.objects.order_by('-created_at').last()

# exists() - ตรวจสอบว่ามี records หรือไม่
has_articles = Article.objects.filter(is_published=True).exists()
print(f"มีบทความที่ publish แล้ว: {has_articles}")

# count() - นับจำนวน records
article_count = Article.objects.filter(is_published=True).count()
print(f"จำนวนบทความที่ publish: {article_count}")
```

### ตัวอย่างที่ 19: Lookup Expressions (Field Lookups)

```python
from myapp.models import Article, Product
from datetime import date

# Exact match (ค่าเริ่มต้น)
Article.objects.filter(title='Django Tutorial')
Article.objects.filter(title__exact='Django Tutorial')

# Case-insensitive exact match
Article.objects.filter(title__iexact='django tutorial')

# Contains
Article.objects.filter(title__contains='Django')
Article.objects.filter(title__icontains='django')  # case-insensitive

# Starts with / Ends with
Article.objects.filter(title__startswith='Django')
Article.objects.filter(title__endswith='Tutorial')
Article.objects.filter(title__istartswith='django')
Article.objects.filter(title__iendswith='tutorial')

# In - อยู่ใน list
Article.objects.filter(id__in=[1, 2, 3, 4, 5])
Article.objects.filter(status__in=['published', 'featured'])

# Greater than / Less than
Product.objects.filter(price__gt=100)   # greater than
Product.objects.filter(price__gte=100)  # greater than or equal
Product.objects.filter(price__lt=500)   # less than
Product.objects.filter(price__lte=500)  # less than or equal

# Between (range)
Product.objects.filter(price__range=(100, 500))

# Date lookups
Article.objects.filter(created_at__date=date(2024, 1, 1))
Article.objects.filter(created_at__year=2024)
Article.objects.filter(created_at__month=1)
Article.objects.filter(created_at__day=15)
Article.objects.filter(created_at__week=1)
Article.objects.filter(created_at__week_day=2)  # 1=Sunday, 2=Monday, ...

# Null checks
Article.objects.filter(published_date__isnull=True)
Article.objects.filter(published_date__isnull=False)

# Regex
Article.objects.filter(title__regex=r'^Django')
Article.objects.filter(title__iregex=r'^django')

# Related model lookups (spanning relationships)
# ใช้ __ (double underscore) สำหรับ traverse relationships
Article.objects.filter(category__name='Python')
Article.objects.filter(category__parent__name='Technology')
Article.objects.filter(author__profile__bio__contains='developer')
```

### ตัวอย่างที่ 20: Ordering และ Slicing

```python
from myapp.models import Article, Product

# order_by() - เรียงลำดับ
articles = Article.objects.order_by('title')          # ascending
articles = Article.objects.order_by('-created_at')    # descending
articles = Article.objects.order_by('category', '-created_at')  # หลาย fields

# Random ordering
import random
articles = Article.objects.order_by('?')  # random (ช้า สำหรับ large datasets)

# reverse() - กลับลำดับ
articles = Article.objects.order_by('created_at').reverse()

# Slicing (แปลงเป็น SQL LIMIT/OFFSET)
first_five = Article.objects.all()[:5]          # LIMIT 5
articles_6_to_10 = Article.objects.all()[5:10]  # OFFSET 5 LIMIT 5
third_article = Article.objects.all()[2]        # OFFSET 2 LIMIT 1

# หมายเหตุ: ไม่รองรับ negative indexing
# Article.objects.all()[-1]  # ไม่ได้! ใช้ .last() แทน

# distinct() - ลบ duplicates
categories = Article.objects.values_list('category', flat=True).distinct()

# Chaining queries
result = (
    Article.objects
    .filter(is_published=True)
    .exclude(category__name='Draft')
    .order_by('-created_at')
    [:10]
)
```

### ตัวอย่างที่ 21: values() และ values_list()

```python
from myapp.models import Article

# values() - คืน QuerySet ของ dictionaries
articles_dict = Article.objects.values('id', 'title', 'created_at')
for article in articles_dict:
    print(article)  # {'id': 1, 'title': '...', 'created_at': ...}

# values_list() - คืน QuerySet ของ tuples
articles_tuples = Article.objects.values_list('id', 'title')
for article in articles_tuples:
    print(article)  # (1, '...')

# flat=True - คืน QuerySet ของ single values
titles = Article.objects.values_list('title', flat=True)
for title in titles:
    print(title)  # 'Django Tutorial'

# ดึง IDs ทั้งหมด
article_ids = Article.objects.values_list('id', flat=True)
print(list(article_ids))  # [1, 2, 3, 4, 5]

# named=True - คืน QuerySet ของ named tuples (Django 3.0+)
articles_named = Article.objects.values_list('id', 'title', named=True)
for article in articles_named:
    print(article.id, article.title)

# ใช้กับ Related Fields
articles_with_cat = Article.objects.values('title', 'category__name')
for article in articles_with_cat:
    print(f"{article['title']} - {article['category__name']}")
```

### ตัวอย่างที่ 22: Creating, Updating, Deleting

```python
from myapp.models import Article, Category

# CREATE - สร้าง record ใหม่

# วิธีที่ 1: สร้างและ save แยกกัน
article = Article(title='บทความใหม่', content='เนื้อหา...')
article.save()  # บันทึกลง database

# วิธีที่ 2: create() - สร้างและ save พร้อมกัน
article = Article.objects.create(
    title='บทความใหม่',
    content='เนื้อหา...',
    is_published=True
)

# วิธีที่ 3: get_or_create() - ดึงหรือสร้างใหม่
category, created = Category.objects.get_or_create(
    name='Python',
    defaults={'slug': 'python'}
)
if created:
    print("สร้าง category ใหม่")
else:
    print("ดึง category เดิม")

# วิธีที่ 4: update_or_create() - อัปเดตหรือสร้างใหม่
category, created = Category.objects.update_or_create(
    name='Python',
    defaults={'slug': 'python', 'description': 'Python programming'}
)

# UPDATE - อัปเดต records

# อัปเดต single object
article = Article.objects.get(id=1)
article.title = 'ชื่อใหม่'
article.save()

# อัปเดตเฉพาะ specific fields
article.save(update_fields=['title', 'updated_at'])

# อัปเดตหลาย records พร้อมกัน (efficient!)
Article.objects.filter(is_published=False).update(is_published=True)

# DELETE - ลบ records

# ลบ single object
article = Article.objects.get(id=1)
article.delete()

# ลบหลาย records พร้อมกัน
deleted_count, _ = Article.objects.filter(is_published=False).delete()
print(f"ลบ {deleted_count} บทความ")

# bulk_create() - สร้างหลาย records พร้อมกัน (efficient!)
articles = [
    Article(title='บทความ 1', content='เนื้อหา 1'),
    Article(title='บทความ 2', content='เนื้อหา 2'),
    Article(title='บทความ 3', content='เนื้อหา 3'),
]
Article.objects.bulk_create(articles)

# bulk_update() - อัปเดตหลาย records พร้อมกัน (Django 2.2+)
articles = Article.objects.filter(is_published=True)
for article in articles:
    article.views += 1
Article.objects.bulk_update(articles, ['views'])
```

---

## Aggregate Functions

Aggregate functions ใช้สำหรับคำนวณค่าจาก multiple records

### ตัวอย่างที่ 23: Basic Aggregations

```python
from django.db.models import Count, Sum, Avg, Max, Min, StdDev, Variance
from myapp.models import Product, Order


# Count - นับจำนวน
total_products = Product.objects.aggregate(Count('id'))
print(total_products)  # {'id__count': 100}

# ใช้ custom key
total_products = Product.objects.aggregate(
    total=Count('id')
)
print(total_products)  # {'total': 100}

# Sum - รวม
total_stock = Product.objects.aggregate(
    total_stock=Sum('stock')
)
print(f"สต็อกรวม: {total_stock['total_stock']}")

# Avg - ค่าเฉลี่ย
avg_price = Product.objects.aggregate(
    avg_price=Avg('price')
)
print(f"ราคาเฉลี่ย: {avg_price['avg_price']:.2f}")

# Max / Min - ค่าสูงสุด/ต่ำสุด
price_range = Product.objects.aggregate(
    max_price=Max('price'),
    min_price=Min('price')
)
print(f"ราคาสูงสุด: {price_range['max_price']}")
print(f"ราคาต่ำสุด: {price_range['min_price']}")

# รวมหลาย aggregations
stats = Product.objects.aggregate(
    total=Count('id'),
    avg_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price'),
    total_stock=Sum('stock'),
)
print(stats)

# Count กับ filter (conditional aggregation)
from django.db.models import Q
in_stock = Product.objects.aggregate(
    in_stock=Count('id', filter=Q(stock__gt=0)),
    out_of_stock=Count('id', filter=Q(stock=0))
)
print(in_stock)
```

---

## annotate() และ aggregate()

### ตัวอย่างที่ 24: annotate() กับ aggregate

```python
from django.db.models import Count, Sum, Avg, Max, Min, F
from myapp.models import Category, Article, Product, Order


# annotate() - เพิ่ม calculated field ให้แต่ละ object
# ต่างจาก aggregate() ที่คืนค่าเดียว

# นับจำนวนบทความในแต่ละ category
categories = Category.objects.annotate(
    article_count=Count('articles')
)
for cat in categories:
    print(f"{cat.name}: {cat.article_count} บทความ")

# filter หลัง annotate
popular_categories = Category.objects.annotate(
    article_count=Count('articles')
).filter(article_count__gte=5)

# order_by กับ annotated field
categories = Category.objects.annotate(
    article_count=Count('articles')
).order_by('-article_count')

# annotate กับ Sum
from myapp.models import Order, OrderItem
orders = Order.objects.annotate(
    total_items=Sum('items__quantity'),
    total_amount=Sum('items__price')
)

# annotate กับ Avg
from myapp.models import Review
products = Product.objects.annotate(
    avg_rating=Avg('reviews__rating'),
    review_count=Count('reviews')
)

# annotate กับ expression
from django.db.models import ExpressionWrapper, DecimalField
products = Product.objects.annotate(
    discounted_price=ExpressionWrapper(
        F('price') * 0.9,
        output_field=DecimalField(max_digits=10, decimal_places=2)
    )
)

# ตัวอย่างการนับ unique values
from django.db.models import Count
stats = Article.objects.aggregate(
    total=Count('id'),
    unique_categories=Count('category', distinct=True)
)
print(stats)
```

### ตัวอย่างที่ 25: Complex Annotations

```python
from django.db.models import Count, Sum, Avg, Case, When, IntegerField
from myapp.models import Product, Order


# Case/When - Conditional annotations
products = Product.objects.annotate(
    stock_level=Case(
        When(stock=0, then=0),
        When(stock__lt=10, then=1),
        When(stock__lt=100, then=2),
        default=3,
        output_field=IntegerField()
    )
)

# Subquery annotation
from django.db.models import OuterRef, Subquery
from myapp.models import OrderItem

latest_order = Order.objects.filter(
    customer=OuterRef('pk')
).order_by('-created_at').values('created_at')[:1]

from myapp.models import Customer
customers = Customer.objects.annotate(
    last_order_date=Subquery(latest_order)
)

# ใช้ annotate กับ GROUP BY ใน raw SQL
# SELECT category_id, COUNT(*) as count, AVG(price) as avg_price
# FROM products GROUP BY category_id
from myapp.models import Product
result = Product.objects.values('category').annotate(
    count=Count('id'),
    avg_price=Avg('price')
).order_by('category')
```

---

## Q Objects สำหรับ Complex Queries

Q objects ใช้สำหรับสร้าง complex queries ที่ต้องการ OR, AND, NOT conditions

### ตัวอย่างที่ 26: Q Objects พื้นฐาน

```python
from django.db.models import Q
from myapp.models import Article, Product


# AND (เหมือน filter() ปกติ)
articles = Article.objects.filter(
    Q(is_published=True) & Q(category__name='Python')
)

# OR - ใช้ | operator
articles = Article.objects.filter(
    Q(category__name='Python') | Q(category__name='Django')
)

# NOT - ใช้ ~ operator
articles = Article.objects.filter(
    ~Q(is_published=False)
)

# ผสมกัน
articles = Article.objects.filter(
    (Q(category__name='Python') | Q(category__name='Django')) &
    Q(is_published=True)
)

# ค้นหาแบบ full-text search
search_term = 'python'
articles = Article.objects.filter(
    Q(title__icontains=search_term) |
    Q(content__icontains=search_term) |
    Q(category__name__icontains=search_term)
)

# Dynamic Q objects
def search_articles(query=None, category=None, is_published=None):
    """ค้นหาบทความแบบ dynamic"""
    filters = Q()

    if query:
        filters &= (
            Q(title__icontains=query) |
            Q(content__icontains=query)
        )

    if category:
        filters &= Q(category__name=category)

    if is_published is not None:
        filters &= Q(is_published=is_published)

    return Article.objects.filter(filters)


# ตัวอย่างการใช้
results = search_articles(
    query='Django',
    category='Python',
    is_published=True
)
```

### ตัวอย่างที่ 27: Q Objects กับ exclude()

```python
from django.db.models import Q
from myapp.models import Product


# exclude() กับ Q objects
# สินค้าที่ไม่ใช่ Python และ ไม่ใช่ Django
articles = Article.objects.exclude(
    Q(category__name='Python') | Q(category__name='Django')
)

# สินค้าที่ราคาไม่อยู่ระหว่าง 100-500 หรือสต็อกหมด
products = Product.objects.filter(
    ~(Q(price__range=(100, 500)) & Q(stock__gt=0))
)

# ตัวอย่าง complex Q สำหรับ e-commerce
def get_available_products(
    min_price=None,
    max_price=None,
    categories=None,
    in_stock_only=True,
    search=None
):
    q = Q()

    if in_stock_only:
        q &= Q(stock__gt=0)

    if min_price is not None:
        q &= Q(price__gte=min_price)

    if max_price is not None:
        q &= Q(price__lte=max_price)

    if categories:
        q &= Q(category__name__in=categories)

    if search:
        q &= (Q(name__icontains=search) | Q(description__icontains=search))

    return Product.objects.filter(q).distinct()


# ใช้งาน
products = get_available_products(
    min_price=100,
    max_price=500,
    categories=['Electronics', 'Gadgets'],
    search='phone'
)
```

---

## F Objects สำหรับ Field References

F objects ใช้สำหรับอ้างอิง field values โดยตรงใน database โดยไม่ต้องดึงข้อมูลมาใน Python

### ตัวอย่างที่ 28: F Objects พื้นฐาน

```python
from django.db.models import F
from myapp.models import Product, Article


# เพิ่ม views ทีละ 1 โดยไม่ต้อง read ก่อน
Article.objects.filter(id=1).update(views=F('views') + 1)

# เปรียบเทียบ fields กัน
# สินค้าที่มีราคาขายน้อยกว่าหรือเท่ากับราคาต้นทุน
products = Product.objects.filter(
    sale_price__lte=F('cost_price')
)

# เพิ่มราคาทุก products 10%
Product.objects.all().update(
    price=F('price') * 1.1
)

# ใช้ F กับ arithmetic
from django.db.models import ExpressionWrapper, FloatField
products = Product.objects.annotate(
    profit_margin=ExpressionWrapper(
        (F('price') - F('cost_price')) / F('cost_price') * 100,
        output_field=FloatField()
    )
)

# เรียงลำดับตาม F expression
# สินค้าที่ profit margin น้อยที่สุดอยู่บน
products = products.order_by('profit_margin')

# F กับ duration
from datetime import timedelta
from django.utils import timezone
# บทความที่ publish มาแล้วอย่างน้อย 7 วัน
old_articles = Article.objects.filter(
    published_date__lte=timezone.now() - timedelta(days=7)
)

# F กับ related fields
# filter โดยใช้ related field
articles = Article.objects.filter(
    views__gte=F('category__average_views')
)
```

### ตัวอย่างที่ 29: F Objects กับ Annotations

```python
from django.db.models import F, Sum, ExpressionWrapper, DecimalField
from myapp.models import OrderItem


# คำนวณ total price สำหรับแต่ละ order item
order_items = OrderItem.objects.annotate(
    line_total=ExpressionWrapper(
        F('quantity') * F('unit_price'),
        output_field=DecimalField(max_digits=10, decimal_places=2)
    )
)

for item in order_items:
    print(f"{item.product.name}: {item.quantity} x {item.unit_price} = {item.line_total}")

# ใช้ F กับ update atomically (thread-safe)
# ถ้าใช้ Python: article.views = article.views + 1 (race condition!)
# ใช้ F แทน: (atomic operation ใน database)
Article.objects.filter(id=1).update(views=F('views') + 1)

# F กับ datetime
from django.utils import timezone
from datetime import timedelta

# ขยาย deadline ทุก task ออกไป 7 วัน
Task.objects.filter(status='pending').update(
    deadline=F('deadline') + timedelta(days=7)
)
```

---

## select_related() และ prefetch_related()

### ตัวอย่างที่ 30: N+1 Query Problem

```python
from myapp.models import Article


# ปัญหา N+1 Query - ไม่ดี!
articles = Article.objects.all()  # 1 query

for article in articles:
    # แต่ละ iteration เรียก query เพิ่มเพื่อดึง category
    print(article.category.name)  # N queries!

# รวมทั้งหมด = 1 + N queries (N = จำนวนบทความ)


# แก้ปัญหาด้วย select_related() - สำหรับ ForeignKey และ OneToOne
articles = Article.objects.select_related('category').all()

for article in articles:
    # ไม่มี query เพิ่ม! ข้อมูล category ถูกดึงมาพร้อมกันแล้ว
    print(article.category.name)

# รวมทั้งหมด = 1 query (JOIN)


# prefetch_related() - สำหรับ ManyToMany และ reverse ForeignKey
from myapp.models import Post

posts = Post.objects.prefetch_related('tags').all()

for post in posts:
    # tags ถูก prefetch มาแล้ว
    for tag in post.tags.all():
        print(tag.name)

# รวมทั้งหมด = 2 queries (1 สำหรับ posts, 1 สำหรับ tags)
```

### ตัวอย่างที่ 31: select_related() และ prefetch_related() Advanced

```python
from django.db.models import Prefetch
from myapp.models import Category, Article, Comment


# select_related() หลาย levels
articles = Article.objects.select_related(
    'category',          # category
    'category__parent',  # parent category
    'author',            # author
    'author__profile'    # author's profile
)

# select_related() ทั้งหมด (ระวัง performance!)
articles = Article.objects.select_related()

# prefetch_related() หลาย relationships
categories = Category.objects.prefetch_related(
    'articles',              # all articles
    'articles__comments',    # all comments ของ articles
    'articles__tags'         # all tags ของ articles
)

# Prefetch object - กำหนด queryset สำหรับ prefetch
from myapp.models import Comment

published_articles = Prefetch(
    'articles',
    queryset=Article.objects.filter(is_published=True).order_by('-created_at'),
    to_attr='published_articles'  # เก็บใน attribute ชื่อ published_articles
)

categories = Category.objects.prefetch_related(published_articles)
for cat in categories:
    # ใช้ to_attr แทน .all()
    for article in cat.published_articles:
        print(article.title)

# รวมกัน select_related() และ prefetch_related()
articles = Article.objects.select_related(
    'category',
    'author'
).prefetch_related(
    'tags',
    Prefetch(
        'comments',
        queryset=Comment.objects.filter(is_approved=True),
        to_attr='approved_comments'
    )
)
```

---

## Migrations

Migrations คือระบบที่ Django ใช้ติดตามการเปลี่ยนแปลง Database Schema

### คำสั่ง Migrations

```bash
# สร้าง migration files จาก models ที่เปลี่ยนแปลง
python manage.py makemigrations

# สร้าง migration สำหรับ app เฉพาะ
python manage.py makemigrations myapp

# สร้าง migration พร้อมระบุชื่อ
python manage.py makemigrations myapp --name add_category_field

# ดูรายการ migrations ทั้งหมด
python manage.py showmigrations

# ดู migrations ของ app เฉพาะ
python manage.py showmigrations myapp

# รัน migrations ทั้งหมด
python manage.py migrate

# รัน migrations สำหรับ app เฉพาะ
python manage.py migrate myapp

# Rollback ไป migration ก่อนหน้า
python manage.py migrate myapp 0001

# Rollback migrations ทั้งหมดของ app
python manage.py migrate myapp zero

# ดู SQL ที่ migration จะรัน
python manage.py sqlmigrate myapp 0001_initial

# ตรวจสอบว่ามี migrations ที่ยังไม่ได้รันหรือไม่
python manage.py migrate --check

# Squash migrations (รวมหลาย migrations เป็น 1)
python manage.py squashmigrations myapp 0001 0010
```

### ตัวอย่างที่ 32: Migration File

```python
# myapp/migrations/0001_initial.py
# ไฟล์ migration ที่ Django สร้างให้อัตโนมัติ

from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):

    initial = True  # migration แรกของ app

    dependencies = [
        # dependencies ไปยัง migrations ของ app อื่น
    ]

    operations = [
        migrations.CreateModel(
            name='Category',
            fields=[
                ('id', models.BigAutoField(
                    auto_created=True,
                    primary_key=True,
                    serialize=False,
                    verbose_name='ID'
                )),
                ('name', models.CharField(max_length=100)),
                ('slug', models.SlugField(unique=True)),
            ],
        ),
        migrations.CreateModel(
            name='Article',
            fields=[
                ('id', models.BigAutoField(
                    auto_created=True,
                    primary_key=True,
                    serialize=False,
                    verbose_name='ID'
                )),
                ('title', models.CharField(max_length=200)),
                ('content', models.TextField()),
                ('is_published', models.BooleanField(default=False)),
                ('created_at', models.DateTimeField(auto_now_add=True)),
                ('category', models.ForeignKey(
                    on_delete=django.db.models.deletion.CASCADE,
                    related_name='articles',
                    to='myapp.category'
                )),
            ],
        ),
    ]
```

### ตัวอย่างที่ 33: Migration กับการเพิ่ม Field

```python
# เมื่อเพิ่ม field ใหม่ใน model
class Article(models.Model):
    # fields เดิม...
    author = models.CharField(max_length=100)  # field ใหม่

# รัน: python manage.py makemigrations
# Django จะถามว่าจะใช้ค่า default อะไรสำหรับ records เดิม

# myapp/migrations/0002_article_author.py
from django.db import migrations, models


class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0001_initial'),
    ]

    operations = [
        migrations.AddField(
            model_name='article',
            name='author',
            field=models.CharField(default='Unknown', max_length=100),
            preserve_default=False,  # ไม่เก็บ default หลัง migration
        ),
    ]
```

### ตัวอย่างที่ 34: Migration Operations

```python
# myapp/migrations/0003_complex_migration.py
from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0002_article_author'),
    ]

    operations = [
        # เพิ่ม field ใหม่
        migrations.AddField(
            model_name='article',
            name='views',
            field=models.IntegerField(default=0),
        ),

        # ลบ field
        migrations.RemoveField(
            model_name='article',
            name='old_field',
        ),

        # เปลี่ยนชื่อ field
        migrations.RenameField(
            model_name='article',
            old_name='pub_date',
            new_name='published_date',
        ),

        # เปลี่ยน field options
        migrations.AlterField(
            model_name='article',
            name='title',
            field=models.CharField(max_length=300),  # เพิ่ม max_length
        ),

        # เพิ่ม index
        migrations.AddIndex(
            model_name='article',
            index=models.Index(fields=['title'], name='article_title_idx'),
        ),

        # สร้าง model ใหม่
        migrations.CreateModel(
            name='Tag',
            fields=[
                ('id', models.AutoField(primary_key=True)),
                ('name', models.CharField(max_length=50, unique=True)),
            ],
        ),

        # ลบ model
        migrations.DeleteModel(
            name='OldModel',
        ),

        # เพิ่ม constraint
        migrations.AddConstraint(
            model_name='article',
            constraint=models.UniqueConstraint(
                fields=['title', 'author'],
                name='unique_article_per_author'
            ),
        ),
    ]
```

---

## Data Migrations

Data Migrations ใช้สำหรับย้ายหรือแปลงข้อมูลใน database ระหว่างการ migration

### ตัวอย่างที่ 35: Data Migration พื้นฐาน

```python
# สร้าง data migration: python manage.py makemigrations --empty myapp
# myapp/migrations/0004_populate_slug.py

from django.db import migrations
from django.utils.text import slugify


def populate_slug(apps, schema_editor):
    """สร้าง slug จาก title สำหรับ records เดิม"""
    # ใช้ apps.get_model() แทนการ import โดยตรง
    # เพื่อให้ได้ model ณ เวลาที่ migration รัน
    Article = apps.get_model('myapp', 'Article')

    for article in Article.objects.all():
        base_slug = slugify(article.title)
        slug = base_slug
        counter = 1

        # ตรวจสอบ unique
        while Article.objects.filter(slug=slug).exists():
            slug = f"{base_slug}-{counter}"
            counter += 1

        article.slug = slug
        article.save()


def reverse_slug(apps, schema_editor):
    """Reverse migration - ลบ slug ทั้งหมด"""
    Article = apps.get_model('myapp', 'Article')
    Article.objects.all().update(slug='')


class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0003_add_slug_field'),
    ]

    operations = [
        migrations.RunPython(
            populate_slug,
            reverse_slug  # optional reverse function
        ),
    ]
```

### ตัวอย่างที่ 36: Complex Data Migration

```python
# myapp/migrations/0005_split_name_field.py
# แยก name field เป็น first_name และ last_name

from django.db import migrations


def split_name(apps, schema_editor):
    """แยก full_name เป็น first_name และ last_name"""
    Person = apps.get_model('myapp', 'Person')

    for person in Person.objects.all():
        if person.full_name:
            parts = person.full_name.strip().split(' ', 1)
            person.first_name = parts[0]
            person.last_name = parts[1] if len(parts) > 1 else ''
            person.save(update_fields=['first_name', 'last_name'])


def merge_name(apps, schema_editor):
    """รวม first_name และ last_name กลับเป็น full_name"""
    Person = apps.get_model('myapp', 'Person')

    for person in Person.objects.all():
        person.full_name = f"{person.first_name} {person.last_name}".strip()
        person.save(update_fields=['full_name'])


class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0004_add_name_fields'),
    ]

    operations = [
        migrations.RunPython(split_name, merge_name),
    ]


# ตัวอย่างการใช้ RunSQL ใน migration
class MigrationWithSQL(migrations.Migration):

    operations = [
        migrations.RunSQL(
            # Forward SQL
            sql="""
                UPDATE myapp_article
                SET views = 0
                WHERE views IS NULL;
            """,
            # Reverse SQL
            reverse_sql=migrations.RunSQL.noop  # ไม่มี reverse
        ),
    ]
```

---

## Advanced ORM Techniques

### ตัวอย่างที่ 37: Custom Manager

```python
from django.db import models
from django.utils import timezone


class PublishedManager(models.Manager):
    """Custom Manager สำหรับ published articles เท่านั้น"""

    def get_queryset(self):
        """Override get_queryset() เพื่อ filter เฉพาะ published"""
        return super().get_queryset().filter(
            is_published=True,
            published_date__lte=timezone.now()
        )

    def by_category(self, category_name):
        """Custom method สำหรับ filter ตาม category"""
        return self.get_queryset().filter(
            category__name=category_name
        )

    def recent(self, days=7):
        """บทความล่าสุดใน N วัน"""
        cutoff = timezone.now() - timezone.timedelta(days=days)
        return self.get_queryset().filter(
            published_date__gte=cutoff
        )


class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    is_published = models.BooleanField(default=False)
    published_date = models.DateTimeField(null=True, blank=True)
    category = models.ForeignKey('Category', on_delete=models.CASCADE)
    views = models.IntegerField(default=0)

    # Default manager ยังคงเป็น objects
    objects = models.Manager()

    # Custom manager
    published = PublishedManager()

    def __str__(self):
        return self.title


# การใช้งาน Custom Manager
# Article.objects.all()              # ทุก articles (รวม unpublished)
# Article.published.all()            # เฉพาะ published articles
# Article.published.by_category('Python')  # published Python articles
# Article.published.recent(30)       # published ใน 30 วันล่าสุด
```

### ตัวอย่างที่ 38: Raw SQL Queries

```python
from django.db import connection
from myapp.models import Article


# Raw SQL กับ Manager
articles = Article.objects.raw(
    'SELECT * FROM myapp_article WHERE is_published = %s',
    [True]
)
for article in articles:
    print(article.title)

# Raw SQL กับ parameters
articles = Article.objects.raw(
    'SELECT * FROM myapp_article WHERE category_id = %s AND views > %s',
    [1, 100]
)

# Raw SQL ที่ซับซ้อน
articles = Article.objects.raw('''
    SELECT a.*, c.name as category_name
    FROM myapp_article a
    JOIN myapp_category c ON a.category_id = c.id
    WHERE a.is_published = 1
    ORDER BY a.published_date DESC
    LIMIT %s
''', [10])

# Direct database queries
with connection.cursor() as cursor:
    cursor.execute(
        'UPDATE myapp_article SET views = views + 1 WHERE id = %s',
        [1]
    )

with connection.cursor() as cursor:
    cursor.execute('SELECT COUNT(*) FROM myapp_article')
    count = cursor.fetchone()[0]
    print(f"จำนวนบทความ: {count}")
```

### ตัวอย่างที่ 39: Transactions

```python
from django.db import transaction
from myapp.models import Account


# atomic() - ทำให้ทุก operations เป็น transaction เดียว
@transaction.atomic
def transfer_money(from_account_id, to_account_id, amount):
    """โอนเงินระหว่าง accounts"""
    from_account = Account.objects.select_for_update().get(id=from_account_id)
    to_account = Account.objects.select_for_update().get(id=to_account_id)

    if from_account.balance < amount:
        raise ValueError('ยอดเงินไม่เพียงพอ')

    from_account.balance -= amount
    from_account.save()

    to_account.balance += amount
    to_account.save()

    return True


# ใช้ context manager
def complex_operation():
    with transaction.atomic():
        # operations ทั้งหมดใน block นี้เป็น transaction เดียว
        article = Article.objects.create(title='New Article', content='...')
        category = Category.objects.get(name='Python')
        article.category = category
        article.save()

        # ถ้ามี exception จะ rollback อัตโนมัติ


# Savepoints
def operation_with_savepoint():
    with transaction.atomic():
        article = Article.objects.create(title='Article 1', content='...')

        try:
            with transaction.atomic():
                # Savepoint
                Article.objects.create(title='Article 2', content='...')
                raise Exception("เกิดข้อผิดพลาด!")
                # Rollback ถึง savepoint (Article 2 ถูกลบ)
        except Exception:
            pass  # ไม่ rollback ทั้งหมด

        # Article 1 ยังอยู่
```

### ตัวอย่างที่ 40: select_for_update()

```python
from django.db import transaction
from myapp.models import Product


@transaction.atomic
def purchase_product(product_id, quantity):
    """ซื้อสินค้า - ใช้ select_for_update เพื่อ lock row"""

    # Lock row เพื่อป้องกัน race condition
    product = Product.objects.select_for_update().get(id=product_id)

    if product.stock < quantity:
        raise ValueError(f'สต็อกไม่เพียงพอ (มี {product.stock}, ต้องการ {quantity})')

    product.stock -= quantity
    product.save()

    return product


# nowait=True - ถ้า lock ไม่ได้ให้ raise exception ทันที
try:
    with transaction.atomic():
        product = Product.objects.select_for_update(nowait=True).get(id=1)
        # ทำงานกับ product
except Exception:
    # ไม่สามารถ lock ได้ มี transaction อื่นกำลังใช้อยู่
    pass

# skip_locked=True - ข้ามไป rows ที่ locked อยู่
available_products = Product.objects.select_for_update(
    skip_locked=True
).filter(stock__gt=0)
```

---

## ตัวอย่างโปรเจคจริง: Blog System

### ตัวอย่างที่ 41: Blog Models ครบชุด

```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User
from django.utils import timezone
from django.utils.text import slugify
from django.urls import reverse


class Category(models.Model):
    """หมวดหมู่บล็อก"""
    name = models.CharField(max_length=100, unique=True, verbose_name='ชื่อ')
    slug = models.SlugField(max_length=100, unique=True)
    description = models.TextField(blank=True, verbose_name='คำอธิบาย')
    parent = models.ForeignKey(
        'self',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='subcategories'
    )
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name = 'หมวดหมู่'
        verbose_name_plural = 'หมวดหมู่'
        ordering = ['name']

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)


class Tag(models.Model):
    """แท็ก"""
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(max_length=50, unique=True)

    class Meta:
        ordering = ['name']

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)


class PostManager(models.Manager):
    """Custom Manager สำหรับ Post"""

    def published(self):
        return self.filter(
            status='published',
            published_at__lte=timezone.now()
        )

    def drafts(self):
        return self.filter(status='draft')

    def by_author(self, author):
        return self.filter(author=author)


class Post(models.Model):
    """บล็อกโพสต์"""

    STATUS_CHOICES = [
        ('draft', 'ฉบับร่าง'),
        ('published', 'เผยแพร่แล้ว'),
        ('archived', 'เก็บเข้าคลัง'),
    ]

    title = models.CharField(max_length=250, verbose_name='หัวข้อ')
    slug = models.SlugField(max_length=250, unique_for_date='published_at')
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name='ผู้เขียน'
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts'
    )
    tags = models.ManyToManyField(Tag, blank=True, related_name='posts')
    body = models.TextField(verbose_name='เนื้อหา')
    excerpt = models.TextField(blank=True, verbose_name='สรุปย่อ')
    cover_image = models.ImageField(
        upload_to='posts/%Y/%m/%d/',
        null=True,
        blank=True
    )
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='draft'
    )
    featured = models.BooleanField(default=False)
    views = models.PositiveIntegerField(default=0)
    likes = models.PositiveIntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)

    objects = PostManager()

    class Meta:
        ordering = ['-published_at']
        verbose_name = 'โพสต์'
        verbose_name_plural = 'โพสต์ทั้งหมด'
        indexes = [
            models.Index(fields=['slug', 'status']),
            models.Index(fields=['-published_at', 'status']),
        ]

    def __str__(self):
        return self.title

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        if self.status == 'published' and not self.published_at:
            self.published_at = timezone.now()
        super().save(*args, **kwargs)

    def get_absolute_url(self):
        return reverse('blog:post_detail', kwargs={
            'year': self.published_at.year,
            'month': self.published_at.month,
            'day': self.published_at.day,
            'slug': self.slug
        })

    @property
    def reading_time(self):
        """ประมาณเวลาอ่าน (นาที)"""
        word_count = len(self.body.split())
        return max(1, word_count // 200)  # 200 words per minute


class Comment(models.Model):
    """ความคิดเห็น"""
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments'
    )
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        null=True,
        blank=True
    )
    author_name = models.CharField(max_length=100, verbose_name='ชื่อ')
    author_email = models.EmailField(verbose_name='อีเมล')
    body = models.TextField(verbose_name='ความคิดเห็น')
    is_approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies'
    )

    class Meta:
        ordering = ['created_at']
        verbose_name = 'ความคิดเห็น'
        verbose_name_plural = 'ความคิดเห็น'

    def __str__(self):
        return f"ความคิดเห็นจาก {self.author_name} ใน {self.post.title}"
```

### ตัวอย่างที่ 42: Blog Views กับ ORM

```python
# blog/views.py
from django.db.models import Count, Q, Avg
from django.utils import timezone
from .models import Post, Category, Tag, Comment


def get_blog_stats():
    """ดึงสถิติของบล็อก"""
    stats = Post.objects.published().aggregate(
        total_posts=Count('id'),
        total_views=models.Sum('views'),
        avg_likes=Avg('likes')
    )
    return stats


def get_popular_posts(limit=5):
    """ดึง popular posts"""
    return Post.objects.published().order_by('-views')[:limit]


def get_recent_posts(limit=5):
    """ดึง recent posts"""
    return (
        Post.objects.published()
        .select_related('author', 'category')
        .prefetch_related('tags')
        .order_by('-published_at')[:limit]
    )


def search_posts(query):
    """ค้นหาโพสต์"""
    return Post.objects.published().filter(
        Q(title__icontains=query) |
        Q(body__icontains=query) |
        Q(excerpt__icontains=query) |
        Q(tags__name__icontains=query) |
        Q(category__name__icontains=query)
    ).distinct().order_by('-published_at')


def get_posts_by_category(category_slug):
    """ดึงโพสต์ตาม category"""
    return (
        Post.objects.published()
        .filter(
            Q(category__slug=category_slug) |
            Q(category__parent__slug=category_slug)
        )
        .select_related('author', 'category')
        .prefetch_related('tags')
        .order_by('-published_at')
    )


def get_category_stats():
    """สถิติของแต่ละ category"""
    return (
        Category.objects.annotate(
            post_count=Count('posts', filter=Q(posts__status='published')),
            total_views=models.Sum(
                'posts__views',
                filter=Q(posts__status='published')
            )
        )
        .filter(post_count__gt=0)
        .order_by('-post_count')
    )


def get_related_posts(post, limit=4):
    """ดึงโพสต์ที่เกี่ยวข้อง"""
    return (
        Post.objects.published()
        .filter(
            Q(category=post.category) |
            Q(tags__in=post.tags.all())
        )
        .exclude(id=post.id)
        .distinct()
        .order_by('-published_at')
        [:limit]
    )
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Library Management System

สร้าง Django models สำหรับระบบห้องสมุดที่มี:
- `Book` (หนังสือ) - title, isbn, publication_year, price
- `Author` (ผู้แต่ง) - name, bio, birth_date
- `Genre` (ประเภท) - name, description
- `Member` (สมาชิก) - name, email, membership_date, is_active
- `Loan` (การยืม) - book, member, loan_date, due_date, return_date

ต้องมี:
- หนังสือ 1 เล่มมีผู้แต่งได้หลายคน (ManyToMany)
- หนังสือ 1 เล่มมีประเภทได้หลายประเภท (ManyToMany)
- สมาชิก 1 คนยืมหนังสือได้หลายเล่ม แต่หนังสือ 1 เล่มมีได้ 1 loan ที่ active

---

### แบบฝึกหัดที่ 2: ORM Queries

จากโมเดลใน Exercise 1 ให้เขียน queries เพื่อ:
1. ดึงหนังสือทั้งหมดที่ตีพิมพ์หลังปี 2010
2. ดึงสมาชิกที่ active และยืมหนังสืออยู่
3. นับจำนวนหนังสือในแต่ละประเภท
4. หาค่าเฉลี่ยราคาหนังสือในแต่ละประเภท
5. ค้นหาหนังสือที่ title หรือ author name มีคำว่า 'python'

---

### แบบฝึกหัดที่ 3: Complex Queries กับ Q Objects

เขียน query function `search_books(query=None, genre=None, min_price=None, max_price=None, available_only=False)` ที่:
- ค้นหาจาก title หรือ author name
- กรองตาม genre
- กรองตาม price range
- กรองเฉพาะที่ไม่ได้ถูกยืมอยู่

---

### แบบฝึกหัดที่ 4: Annotations และ Aggregations

เขียน queries เพื่อ:
1. หา author ที่มีหนังสือมากที่สุด พร้อม count
2. หา member ที่ยืมหนังสือมากที่สุดตลอดกาล
3. คำนวณ overdue loans (เลยกำหนดคืนแล้ว)
4. หาหนังสือที่ถูกยืมบ่อยที่สุดในรอบ 30 วัน

---

### แบบฝึกหัดที่ 5: select_related และ prefetch_related

เขียน view function ที่ดึง loans ทั้งหมดพร้อม:
- ข้อมูล book และ authors ของ book
- ข้อมูล member
- genres ของ book
โดยใช้ select_related() และ prefetch_related() อย่างเหมาะสม

---

### แบบฝึกหัดที่ 6: Custom Manager

สร้าง Custom Manager สำหรับ `Loan` model ที่มี methods:
- `active()` - loans ที่ยังไม่คืน
- `overdue()` - loans ที่เลยกำหนด
- `returned_this_month()` - loans ที่คืนในเดือนนี้
- `by_member(member)` - loans ของ member คนนั้น

---

### แบบฝึกหัดที่ 7: Migration

1. สร้าง initial migration สำหรับ Library models
2. เพิ่ม field `late_fee` (DecimalField) ให้ `Loan`
3. สร้าง data migration ที่คำนวณ late_fee จาก return_date - due_date (50 บาทต่อวัน)
4. เพิ่ม index ให้กับ `Book.isbn` และ `Member.email`

---

### แบบฝึกหัดที่ 8: ระบบ E-commerce

สร้าง models สำหรับระบบ E-commerce:
- `Product`, `Category`, `Customer`, `Order`, `OrderItem`
- เขียน queries สำหรับ:
  1. สินค้าขายดีสุด 10 อันดับ
  2. รายได้รวมแยกตามเดือน
  3. ลูกค้าที่ซื้อมากที่สุด
  4. สินค้าที่ใกล้หมดสต็อก (stock < 10)

---

## เฉลยแบบฝึกหัด

### เฉลยแบบฝึกหัดที่ 1

```python
# library/models.py
from django.db import models


class Author(models.Model):
    """ผู้แต่ง"""
    name = models.CharField(max_length=200, verbose_name='ชื่อ')
    bio = models.TextField(blank=True, verbose_name='ประวัติ')
    birth_date = models.DateField(null=True, blank=True, verbose_name='วันเกิด')
    email = models.EmailField(blank=True, unique=True)

    class Meta:
        ordering = ['name']
        verbose_name = 'ผู้แต่ง'

    def __str__(self):
        return self.name


class Genre(models.Model):
    """ประเภทหนังสือ"""
    name = models.CharField(max_length=100, unique=True, verbose_name='ชื่อประเภท')
    description = models.TextField(blank=True, verbose_name='คำอธิบาย')

    class Meta:
        ordering = ['name']
        verbose_name = 'ประเภท'

    def __str__(self):
        return self.name


class Book(models.Model):
    """หนังสือ"""
    title = models.CharField(max_length=300, verbose_name='ชื่อหนังสือ')
    isbn = models.CharField(
        max_length=13,
        unique=True,
        verbose_name='ISBN',
        db_index=True
    )
    publication_year = models.IntegerField(verbose_name='ปีที่พิมพ์')
    price = models.DecimalField(
        max_digits=8,
        decimal_places=2,
        verbose_name='ราคา'
    )
    authors = models.ManyToManyField(
        Author,
        related_name='books',
        verbose_name='ผู้แต่ง'
    )
    genres = models.ManyToManyField(
        Genre,
        related_name='books',
        verbose_name='ประเภท'
    )
    total_copies = models.PositiveIntegerField(default=1, verbose_name='จำนวนเล่ม')
    available_copies = models.PositiveIntegerField(default=1, verbose_name='เล่มที่ว่าง')
    added_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['title']
        verbose_name = 'หนังสือ'
        indexes = [
            models.Index(fields=['title'], name='book_title_idx'),
            models.Index(fields=['isbn'], name='book_isbn_idx'),
        ]

    def __str__(self):
        return f"{self.title} ({self.publication_year})"

    def is_available(self):
        return self.available_copies > 0


class Member(models.Model):
    """สมาชิกห้องสมุด"""
    name = models.CharField(max_length=200, verbose_name='ชื่อ')
    email = models.EmailField(unique=True, verbose_name='อีเมล', db_index=True)
    phone = models.CharField(max_length=20, blank=True, verbose_name='โทรศัพท์')
    membership_date = models.DateField(auto_now_add=True, verbose_name='วันสมัครสมาชิก')
    is_active = models.BooleanField(default=True, verbose_name='สถานะ')

    class Meta:
        ordering = ['name']
        verbose_name = 'สมาชิก'

    def __str__(self):
        return f"{self.name} ({self.email})"


class LoanManager(models.Manager):
    """Custom Manager สำหรับ Loan"""

    def active(self):
        return self.filter(return_date__isnull=True)

    def overdue(self):
        from django.utils import timezone
        return self.filter(
            return_date__isnull=True,
            due_date__lt=timezone.now().date()
        )

    def returned_this_month(self):
        from django.utils import timezone
        now = timezone.now()
        return self.filter(
            return_date__year=now.year,
            return_date__month=now.month
        )

    def by_member(self, member):
        return self.filter(member=member)


class Loan(models.Model):
    """การยืมหนังสือ"""
    book = models.ForeignKey(
        Book,
        on_delete=models.PROTECT,
        related_name='loans',
        verbose_name='หนังสือ'
    )
    member = models.ForeignKey(
        Member,
        on_delete=models.CASCADE,
        related_name='loans',
        verbose_name='สมาชิก'
    )
    loan_date = models.DateField(auto_now_add=True, verbose_name='วันที่ยืม')
    due_date = models.DateField(verbose_name='กำหนดคืน')
    return_date = models.DateField(
        null=True,
        blank=True,
        verbose_name='วันที่คืน'
    )
    late_fee = models.DecimalField(
        max_digits=8,
        decimal_places=2,
        default=0,
        verbose_name='ค่าปรับล่าช้า'
    )

    objects = LoanManager()

    class Meta:
        ordering = ['-loan_date']
        verbose_name = 'การยืม'

    def __str__(self):
        return f"{self.member.name} ยืม {self.book.title}"

    def is_overdue(self):
        from django.utils import timezone
        if self.return_date:
            return False
        return self.due_date < timezone.now().date()

    def calculate_late_fee(self, fee_per_day=50):
        """คำนวณค่าปรับ"""
        from django.utils import timezone
        if not self.is_overdue():
            return 0
        days_overdue = (timezone.now().date() - self.due_date).days
        return days_overdue * fee_per_day
```

### เฉลยแบบฝึกหัดที่ 2

```python
from library.models import Book, Member, Genre, Loan
from django.db.models import Count, Avg, Q


# 1. ดึงหนังสือทั้งหมดที่ตีพิมพ์หลังปี 2010
books_after_2010 = Book.objects.filter(
    publication_year__gt=2010
).order_by('-publication_year')
print(f"หนังสือหลังปี 2010: {books_after_2010.count()} เล่ม")


# 2. ดึงสมาชิกที่ active และยืมหนังสืออยู่
active_borrowers = Member.objects.filter(
    is_active=True,
    loans__return_date__isnull=True
).distinct()
print(f"สมาชิกที่กำลังยืมหนังสือ: {active_borrowers.count()} คน")


# 3. นับจำนวนหนังสือในแต่ละประเภท
genre_stats = Genre.objects.annotate(
    book_count=Count('books')
).order_by('-book_count')
for genre in genre_stats:
    print(f"{genre.name}: {genre.book_count} เล่ม")


# 4. หาค่าเฉลี่ยราคาหนังสือในแต่ละประเภท
genre_avg_price = Genre.objects.annotate(
    avg_price=Avg('books__price'),
    book_count=Count('books')
).filter(book_count__gt=0).order_by('-avg_price')
for genre in genre_avg_price:
    print(f"{genre.name}: ราคาเฉลี่ย {genre.avg_price:.2f} บาท")


# 5. ค้นหาหนังสือที่ title หรือ author name มีคำว่า 'python'
python_books = Book.objects.filter(
    Q(title__icontains='python') |
    Q(authors__name__icontains='python')
).distinct()
print(f"หนังสือเกี่ยวกับ Python: {python_books.count()} เล่ม")
```

### เฉลยแบบฝึกหัดที่ 3

```python
from django.db.models import Q
from library.models import Book


def search_books(
    query=None,
    genre=None,
    min_price=None,
    max_price=None,
    available_only=False
):
    """ค้นหาหนังสือแบบ dynamic"""
    filters = Q()

    # ค้นหาจาก title หรือ author name
    if query:
        filters &= (
            Q(title__icontains=query) |
            Q(authors__name__icontains=query)
        )

    # กรองตาม genre
    if genre:
        filters &= Q(genres__name__iexact=genre)

    # กรองตาม price range
    if min_price is not None:
        filters &= Q(price__gte=min_price)

    if max_price is not None:
        filters &= Q(price__lte=max_price)

    # กรองเฉพาะที่ว่างให้ยืม
    if available_only:
        filters &= Q(available_copies__gt=0)

    return (
        Book.objects.filter(filters)
        .prefetch_related('authors', 'genres')
        .distinct()
        .order_by('title')
    )


# ตัวอย่างการใช้
results = search_books(
    query='python',
    genre='Programming',
    min_price=100,
    max_price=800,
    available_only=True
)
for book in results:
    print(f"{book.title} - {book.price} บาท")
```

### เฉลยแบบฝึกหัดที่ 4

```python
from django.db.models import Count, Sum, Q, F, ExpressionWrapper, IntegerField
from django.utils import timezone
from datetime import timedelta
from library.models import Author, Member, Loan, Book


# 1. Author ที่มีหนังสือมากที่สุด
top_authors = Author.objects.annotate(
    book_count=Count('books')
).order_by('-book_count')[:10]
for author in top_authors:
    print(f"{author.name}: {author.book_count} เล่ม")


# 2. Member ที่ยืมหนังสือมากที่สุดตลอดกาล
top_borrowers = Member.objects.annotate(
    loan_count=Count('loans')
).order_by('-loan_count')[:10]
for member in top_borrowers:
    print(f"{member.name}: {member.loan_count} ครั้ง")


# 3. คำนวณ overdue loans
today = timezone.now().date()
overdue_loans = Loan.objects.filter(
    return_date__isnull=True,
    due_date__lt=today
).annotate(
    days_overdue=ExpressionWrapper(
        today - F('due_date'),
        output_field=IntegerField()
    )
).order_by('-days_overdue')
for loan in overdue_loans:
    print(f"{loan.member.name} - {loan.book.title}: เกินกำหนด {loan.days_overdue} วัน")


# 4. หนังสือที่ถูกยืมบ่อยที่สุดในรอบ 30 วัน
thirty_days_ago = timezone.now().date() - timedelta(days=30)
popular_books = Book.objects.filter(
    loans__loan_date__gte=thirty_days_ago
).annotate(
    loan_count=Count('loans', filter=Q(loans__loan_date__gte=thirty_days_ago))
).order_by('-loan_count')[:10]
for book in popular_books:
    print(f"{book.title}: {book.loan_count} ครั้งใน 30 วัน")
```

### เฉลยแบบฝึกหัดที่ 5

```python
from django.db.models import Prefetch
from library.models import Loan, Book, Author


def get_all_loans_optimized():
    """ดึง loans ทั้งหมดพร้อม related data แบบ optimized"""

    # Prefetch authors ของแต่ละ book
    book_with_authors = Prefetch(
        'book',
        queryset=Book.objects.prefetch_related(
            Prefetch(
                'authors',
                queryset=Author.objects.only('name')
            ),
            'genres'
        )
    )

    loans = (
        Loan.objects
        .select_related(
            'member',        # OneToOne / ForeignKey
            'book',          # ForeignKey
        )
        .prefetch_related(
            'book__authors', # ManyToMany ผ่าน book
            'book__genres'   # ManyToMany ผ่าน book
        )
        .order_by('-loan_date')
    )

    return loans


# ใช้งาน
loans = get_all_loans_optimized()
for loan in loans:
    authors = ', '.join(a.name for a in loan.book.authors.all())
    genres = ', '.join(g.name for g in loan.book.genres.all())
    print(f"""
    สมาชิก: {loan.member.name}
    หนังสือ: {loan.book.title}
    ผู้แต่ง: {authors}
    ประเภท: {genres}
    วันยืม: {loan.loan_date}
    """)
```

### เฉลยแบบฝึกหัดที่ 6

(Custom Manager ถูกเขียนไว้แล้วใน Model `LoanManager` ในเฉลยข้อ 1)

```python
# การใช้งาน Custom Manager
from library.models import Loan, Member


# loans ที่ยังไม่คืน
active_loans = Loan.objects.active()
print(f"การยืมที่ active: {active_loans.count()} รายการ")

# loans ที่เกินกำหนด
overdue_loans = Loan.objects.overdue()
print(f"การยืมที่เกินกำหนด: {overdue_loans.count()} รายการ")

# loans ที่คืนในเดือนนี้
this_month_returns = Loan.objects.returned_this_month()
print(f"คืนหนังสือเดือนนี้: {this_month_returns.count()} รายการ")

# loans ของ member คนหนึ่ง
member = Member.objects.get(id=1)
member_loans = Loan.objects.by_member(member)
print(f"การยืมของ {member.name}: {member_loans.count()} รายการ")
```

### เฉลยแบบฝึกหัดที่ 7

```bash
# สร้าง initial migration
python manage.py makemigrations library --name initial

# เพิ่ม late_fee field ใน model แล้วรัน
python manage.py makemigrations library --name add_late_fee

# สร้าง data migration
python manage.py makemigrations --empty library --name calculate_late_fees
```

```python
# library/migrations/0003_calculate_late_fees.py
from django.db import migrations


def calculate_late_fees(apps, schema_editor):
    """คำนวณ late_fee สำหรับ loans เดิม"""
    Loan = apps.get_model('library', 'Loan')
    FEE_PER_DAY = 50  # 50 บาทต่อวัน

    for loan in Loan.objects.filter(return_date__isnull=False):
        if loan.return_date > loan.due_date:
            days_late = (loan.return_date - loan.due_date).days
            loan.late_fee = days_late * FEE_PER_DAY
            loan.save(update_fields=['late_fee'])


def reverse_late_fees(apps, schema_editor):
    """Reverse: ตั้ง late_fee กลับเป็น 0"""
    Loan = apps.get_model('library', 'Loan')
    Loan.objects.all().update(late_fee=0)


class Migration(migrations.Migration):

    dependencies = [
        ('library', '0002_add_late_fee'),
    ]

    operations = [
        migrations.RunPython(calculate_late_fees, reverse_late_fees),
    ]
```

```python
# library/migrations/0004_add_indexes.py
from django.db import migrations, models


class Migration(migrations.Migration):

    dependencies = [
        ('library', '0003_calculate_late_fees'),
    ]

    operations = [
        migrations.AddIndex(
            model_name='book',
            index=models.Index(fields=['isbn'], name='book_isbn_idx'),
        ),
        migrations.AddIndex(
            model_name='member',
            index=models.Index(fields=['email'], name='member_email_idx'),
        ),
    ]
```

### เฉลยแบบฝึกหัดที่ 8

```python
# shop/models.py
from django.db import models
from django.contrib.auth.models import User


class Category(models.Model):
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)

    def __str__(self):
        return self.name


class Product(models.Model):
    name = models.CharField(max_length=200)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name


class Customer(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    phone = models.CharField(max_length=20, blank=True)
    address = models.TextField(blank=True)

    def __str__(self):
        return self.user.get_full_name() or self.user.username


class Order(models.Model):
    STATUS_CHOICES = [
        ('pending', 'รอดำเนินการ'),
        ('processing', 'กำลังดำเนินการ'),
        ('shipped', 'จัดส่งแล้ว'),
        ('delivered', 'ส่งถึงแล้ว'),
        ('cancelled', 'ยกเลิก'),
    ]
    customer = models.ForeignKey(Customer, on_delete=models.CASCADE, related_name='orders')
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    created_at = models.DateTimeField(auto_now_add=True)
    total_amount = models.DecimalField(max_digits=10, decimal_places=2, default=0)

    def __str__(self):
        return f"Order #{self.id} - {self.customer}"


class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='items')
    product = models.ForeignKey(Product, on_delete=models.PROTECT)
    quantity = models.PositiveIntegerField(default=1)
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)

    @property
    def subtotal(self):
        return self.quantity * self.unit_price

    def __str__(self):
        return f"{self.product.name} x {self.quantity}"


# shop/queries.py
from django.db.models import Count, Sum, F, Q
from django.utils import timezone
from datetime import timedelta
from .models import Product, Order, OrderItem, Customer


def get_best_selling_products(limit=10):
    """สินค้าขายดีสุด 10 อันดับ"""
    return (
        Product.objects
        .filter(orderitem__order__status='delivered')
        .annotate(
            total_sold=Sum('orderitem__quantity')
        )
        .order_by('-total_sold')
        [:limit]
    )


def get_monthly_revenue():
    """รายได้รวมแยกตามเดือน"""
    from django.db.models.functions import TruncMonth
    return (
        Order.objects
        .filter(status='delivered')
        .annotate(month=TruncMonth('created_at'))
        .values('month')
        .annotate(revenue=Sum('total_amount'))
        .order_by('month')
    )


def get_top_customers(limit=10):
    """ลูกค้าที่ซื้อมากที่สุด"""
    return (
        Customer.objects
        .annotate(
            total_orders=Count('orders', filter=Q(orders__status='delivered')),
            total_spent=Sum(
                'orders__total_amount',
                filter=Q(orders__status='delivered')
            )
        )
        .filter(total_orders__gt=0)
        .order_by('-total_spent')
        [:limit]
    )


def get_low_stock_products(threshold=10):
    """สินค้าที่ใกล้หมดสต็อก"""
    return (
        Product.objects
        .filter(
            is_active=True,
            stock__lt=threshold,
            stock__gt=0
        )
        .order_by('stock')
    )


# ตัวอย่างการใช้
print("สินค้าขายดีสุด 10 อันดับ:")
for product in get_best_selling_products():
    print(f"  {product.name}: {product.total_sold} ชิ้น")

print("\nรายได้รายเดือน:")
for item in get_monthly_revenue():
    print(f"  {item['month'].strftime('%Y-%m')}: {item['revenue']:.2f} บาท")

print("\nลูกค้าอันดับต้น:")
for customer in get_top_customers():
    print(f"  {customer}: {customer.total_orders} orders, {customer.total_spent:.2f} บาท")

print("\nสินค้าใกล้หมด:")
for product in get_low_stock_products():
    print(f"  {product.name}: เหลือ {product.stock} ชิ้น")
```

---

## สรุป

ในบทนี้เราได้เรียนรู้เกี่ยวกับ Django Models, ORM และ Migrations อย่างครบถ้วน:

### สิ่งที่ได้เรียนรู้

| หัวข้อ | สาระสำคัญ |
|--------|-----------|
| **Django Models** | Python class ที่ map กับ database table |
| **Field Types** | CharField, IntegerField, DateField, ForeignKey, ManyToManyField, etc. |
| **Field Options** | null, blank, default, unique, choices, verbose_name |
| **Meta Class** | ordering, db_table, indexes, constraints, permissions |
| **ORM Queries** | filter(), exclude(), get(), all(), order_by(), values() |
| **Aggregations** | Count, Sum, Avg, Max, Min |
| **Annotations** | annotate() สำหรับ calculated fields |
| **Q Objects** | OR, AND, NOT สำหรับ complex queries |
| **F Objects** | อ้างอิง field values โดยตรงใน database |
| **Performance** | select_related(), prefetch_related() |
| **Migrations** | makemigrations, migrate, data migrations |

### Best Practices

1. **ใช้ select_related() กับ ForeignKey** เสมอเมื่อต้องการ related data
2. **ใช้ prefetch_related() กับ ManyToMany** เพื่อหลีกเลี่ยง N+1 queries
3. **ใช้ F objects** สำหรับ atomic updates เพื่อหลีกเลี่ยง race conditions
4. **สร้าง indexes** สำหรับ fields ที่ใช้ filter หรือ order บ่อย
5. **ใช้ bulk_create/bulk_update** เมื่อต้องการสร้างหรืออัปเดตหลาย records
6. **เขียน Data Migrations** เมื่อต้องการแปลงข้อมูลพร้อมกับ schema change
7. **ใช้ Custom Managers** เพื่อ encapsulate business logic ของ queries

### Part ถัดไป

- **Part 64**: Django Views, URLs และ Templates
- **Part 65**: Django Forms และ Model Forms
- **Part 66**: Django REST Framework (DRF)

---

*เนื้อหาส่วนนี้เป็นส่วนหนึ่งของหลักสูตร Python ฉบับสมบูรณ์*
*สร้างโดย: Python Course Team*
*อัปเดตล่าสุด: 2024*
