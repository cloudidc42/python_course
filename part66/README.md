# Part 66: Django REST Framework (DRF)

## บทนำ

Django REST Framework (DRF) คือ toolkit ที่ทรงพลังสำหรับการสร้าง Web APIs บน Django โดยมีคุณสมบัติครบครัน ตั้งแต่ serialization, authentication, permissions ไปจนถึง browsable API interface ที่ใช้งานได้ทันที DRF เป็นมาตรฐานอุตสาหกรรมสำหรับ REST API development ใน Python ecosystem

## สารบัญ

1. [DRF Installation และ Setup](#1-drf-installation-และ-setup)
2. [Serializers](#2-serializers)
3. [ModelSerializer](#3-modelserializer)
4. [APIView](#4-apiview)
5. [ViewSets](#5-viewsets)
6. [Routers](#6-routers)
7. [Authentication](#7-authentication)
8. [Permissions](#8-permissions)
9. [Pagination](#9-pagination)
10. [Filtering](#10-filtering)
11. [Throttling](#11-throttling)
12. [API Documentation](#12-api-documentation)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. DRF Installation และ Setup

### การติดตั้ง

```bash
# สร้าง virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# ติดตั้ง Django และ DRF
pip install django djangorestframework

# ติดตั้ง packages เพิ่มเติม
pip install djangorestframework-simplejwt  # JWT authentication
pip install django-filter                   # Advanced filtering
pip install drf-spectacular                 # API documentation
pip install Pillow                          # Image handling
pip install django-cors-headers             # CORS support
```

### การสร้างโปรเจกต์

```bash
# สร้าง Django project
django-admin startproject myapi .

# สร้าง app
python manage.py startapp api
```

### การตั้งค่า settings.py

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party apps
    'rest_framework',
    'rest_framework_simplejwt',
    'django_filters',
    'drf_spectacular',
    'corsheaders',
    
    # Local apps
    'api',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',  # ต้องอยู่ก่อน CommonMiddleware
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# DRF Global Configuration
REST_FRAMEWORK = {
    # Authentication classes
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    
    # Permission classes
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    
    # Pagination
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    
    # Filtering
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    
    # Throttling
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
    },
    
    # Schema
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
    
    # Renderer
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
    
    # Parser
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser',
    ],
    
    # Exception handling
    'EXCEPTION_HANDLER': 'api.exceptions.custom_exception_handler',
}

# CORS settings
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]

CORS_ALLOW_CREDENTIALS = True

# JWT settings
from datetime import timedelta
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'AUTH_HEADER_TYPES': ('Bearer',),
}

# drf-spectacular settings
SPECTACULAR_SETTINGS = {
    'TITLE': 'My API',
    'DESCRIPTION': 'API Documentation',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
}
```

### โครงสร้างโปรเจกต์

```
myapi/
├── myapi/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── api/
│   ├── __init__.py
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── urls.py
│   ├── permissions.py
│   ├── filters.py
│   ├── pagination.py
│   └── exceptions.py
└── manage.py
```

---

## 2. Serializers

Serializer ทำหน้าที่แปลง complex data types (เช่น Django model instances) ให้เป็น Python data types ที่สามารถ render เป็น JSON, XML หรือ formats อื่นๆ และทำงานในทิศทางตรงข้าม (deserialization + validation)

### Basic Serializer

```python
# api/serializers.py
from rest_framework import serializers

# ตัวอย่างที่ 1: Serializer พื้นฐาน
class ArticleSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=200)
    content = serializers.CharField()
    author = serializers.CharField(max_length=100)
    published_at = serializers.DateTimeField(read_only=True)
    is_published = serializers.BooleanField(default=False)
    
    def create(self, validated_data):
        """สร้าง instance ใหม่จาก validated data"""
        return Article.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        """อัพเดต instance ที่มีอยู่แล้ว"""
        instance.title = validated_data.get('title', instance.title)
        instance.content = validated_data.get('content', instance.content)
        instance.author = validated_data.get('author', instance.author)
        instance.is_published = validated_data.get('is_published', instance.is_published)
        instance.save()
        return instance
```

### การใช้งาน Serializer

```python
# ตัวอย่างที่ 2: การใช้งาน Serializer ใน shell
# python manage.py shell

from api.serializers import ArticleSerializer

# Serialization: Python object -> dict -> JSON
article = Article.objects.get(id=1)
serializer = ArticleSerializer(article)
print(serializer.data)
# {'id': 1, 'title': 'Hello World', 'content': '...', ...}

import json
json_str = json.dumps(serializer.data)

# Many objects
articles = Article.objects.all()
serializer = ArticleSerializer(articles, many=True)
print(serializer.data)

# Deserialization: JSON -> validated data -> object
data = {
    'title': 'New Article',
    'content': 'This is content',
    'author': 'John'
}
serializer = ArticleSerializer(data=data)
if serializer.is_valid():
    article = serializer.save()  # เรียก create() method
    print(f"Created: {article.id}")
else:
    print(serializer.errors)

# Update existing
article = Article.objects.get(id=1)
serializer = ArticleSerializer(article, data={'title': 'Updated'}, partial=True)
if serializer.is_valid():
    serializer.save()  # เรียก update() method
```

### Field Types

```python
# ตัวอย่างที่ 3: Field types ต่างๆ
class FieldDemoSerializer(serializers.Serializer):
    # Basic fields
    char_field = serializers.CharField(
        max_length=100,
        min_length=5,
        allow_blank=False,
        trim_whitespace=True
    )
    integer_field = serializers.IntegerField(
        min_value=0,
        max_value=1000
    )
    float_field = serializers.FloatField()
    decimal_field = serializers.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    boolean_field = serializers.BooleanField()
    email_field = serializers.EmailField()
    url_field = serializers.URLField()
    uuid_field = serializers.UUIDField()
    
    # Date/Time fields
    date_field = serializers.DateField(format='%Y-%m-%d')
    datetime_field = serializers.DateTimeField(
        format='%Y-%m-%d %H:%M:%S',
        input_formats=['%Y-%m-%d %H:%M:%S', 'iso-8601']
    )
    time_field = serializers.TimeField()
    duration_field = serializers.DurationField()
    
    # Choice fields
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    status = serializers.ChoiceField(choices=STATUS_CHOICES)
    
    # List fields
    tags = serializers.ListField(
        child=serializers.CharField(max_length=50),
        min_length=1,
        max_length=10
    )
    
    # Dict fields
    metadata = serializers.DictField(
        child=serializers.CharField()
    )
    
    # JSON field
    json_data = serializers.JSONField()
    
    # File fields
    image = serializers.ImageField(
        max_length=None,
        use_url=True,
        allow_empty_file=False
    )
    document = serializers.FileField()
    
    # Read-only and write-only
    id = serializers.IntegerField(read_only=True)
    password = serializers.CharField(write_only=True)
    
    # Hidden field (ไม่ show ใน response, ไม่รับจาก input)
    created_by = serializers.HiddenField(
        default=serializers.CurrentUserDefault()
    )
```

### Nested Serializers

```python
# ตัวอย่างที่ 4: Nested Serializers
class AuthorSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    username = serializers.CharField()
    email = serializers.EmailField()

class CommentSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    text = serializers.CharField()
    author = AuthorSerializer()
    created_at = serializers.DateTimeField(read_only=True)

class ArticleDetailSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField()
    content = serializers.CharField()
    author = AuthorSerializer()  # Nested serializer
    comments = CommentSerializer(many=True, read_only=True)  # Nested list
    tags = serializers.ListField(child=serializers.CharField())
```

### Custom Validation

```python
# ตัวอย่างที่ 5: Custom Validation
class UserRegistrationSerializer(serializers.Serializer):
    username = serializers.CharField(max_length=150)
    email = serializers.EmailField()
    password = serializers.CharField(min_length=8, write_only=True)
    password_confirm = serializers.CharField(min_length=8, write_only=True)
    age = serializers.IntegerField()
    
    def validate_username(self, value):
        """Field-level validation"""
        if User.objects.filter(username=value).exists():
            raise serializers.ValidationError("Username already taken")
        if value.lower() in ['admin', 'root', 'system']:
            raise serializers.ValidationError("Username not allowed")
        return value
    
    def validate_email(self, value):
        """Validate email format and uniqueness"""
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email already registered")
        return value.lower()
    
    def validate_age(self, value):
        """Validate age range"""
        if value < 13:
            raise serializers.ValidationError("Must be at least 13 years old")
        if value > 120:
            raise serializers.ValidationError("Invalid age")
        return value
    
    def validate(self, data):
        """Object-level validation"""
        if data['password'] != data['password_confirm']:
            raise serializers.ValidationError({
                'password_confirm': "Passwords don't match"
            })
        return data
    
    def create(self, validated_data):
        validated_data.pop('password_confirm')
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            password=validated_data['password']
        )
        return user
```

### SerializerMethodField

```python
# ตัวอย่างที่ 6: SerializerMethodField
class ArticleSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField()
    content = serializers.CharField()
    created_at = serializers.DateTimeField(read_only=True)
    
    # Computed fields
    word_count = serializers.SerializerMethodField()
    time_since_published = serializers.SerializerMethodField()
    is_recent = serializers.SerializerMethodField()
    
    def get_word_count(self, obj):
        """คำนวณจำนวนคำในเนื้อหา"""
        return len(obj.content.split())
    
    def get_time_since_published(self, obj):
        """คำนวณเวลาที่ผ่านมาตั้งแต่ publish"""
        from django.utils import timezone
        delta = timezone.now() - obj.created_at
        if delta.days > 0:
            return f"{delta.days} days ago"
        hours = delta.seconds // 3600
        if hours > 0:
            return f"{hours} hours ago"
        minutes = delta.seconds // 60
        return f"{minutes} minutes ago"
    
    def get_is_recent(self, obj):
        """ตรวจสอบว่า publish ภายใน 7 วันที่ผ่านมาหรือไม่"""
        from django.utils import timezone
        from datetime import timedelta
        return obj.created_at > timezone.now() - timedelta(days=7)
```

---

## 3. ModelSerializer

ModelSerializer เป็น shortcut สำหรับสร้าง Serializer จาก Django Model โดยอัตโนมัติ

### ตัวอย่าง Models

```python
# api/models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = 'categories'
    
    def __str__(self):
        return self.name

class Tag(models.Model):
    name = models.CharField(max_length=50)
    slug = models.SlugField(unique=True)
    
    def __str__(self):
        return self.name

class Article(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    excerpt = models.TextField(blank=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='articles')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, related_name='articles')
    tags = models.ManyToManyField(Tag, blank=True, related_name='articles')
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')
    featured_image = models.ImageField(upload_to='articles/', blank=True, null=True)
    view_count = models.PositiveIntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title

class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='replies')
    is_approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Comment by {self.author} on {self.article}"
```

### Basic ModelSerializer

```python
# ตัวอย่างที่ 7: Basic ModelSerializer
from rest_framework import serializers
from .models import Article, Category, Tag, Comment
from django.contrib.auth.models import User

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = '__all__'  # หรือระบุ fields: ['id', 'name', 'slug']
        # exclude = ['description']  # ยกเว้น fields บางอัน
        read_only_fields = ['id']

class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ['id', 'name', 'slug']
```

### ModelSerializer with Nested

```python
# ตัวอย่างที่ 8: ModelSerializer with Nested Serializers
class UserMinimalSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'first_name', 'last_name']

class ArticleListSerializer(serializers.ModelSerializer):
    """Serializer สำหรับ list view (ข้อมูลย่อ)"""
    author = UserMinimalSerializer(read_only=True)
    category_name = serializers.CharField(source='category.name', read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    comment_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Article
        fields = [
            'id', 'title', 'slug', 'excerpt', 'author',
            'category_name', 'tags', 'status', 'view_count',
            'comment_count', 'created_at'
        ]
    
    def get_comment_count(self, obj):
        return obj.comments.filter(is_approved=True).count()

class ArticleDetailSerializer(serializers.ModelSerializer):
    """Serializer สำหรับ detail view (ข้อมูลเต็ม)"""
    author = UserMinimalSerializer(read_only=True)
    category = CategorySerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    
    # Write-only fields สำหรับการสร้าง/แก้ไข
    author_id = serializers.PrimaryKeyRelatedField(
        queryset=User.objects.all(),
        source='author',
        write_only=True
    )
    category_id = serializers.PrimaryKeyRelatedField(
        queryset=Category.objects.all(),
        source='category',
        write_only=True,
        allow_null=True
    )
    tag_ids = serializers.PrimaryKeyRelatedField(
        queryset=Tag.objects.all(),
        source='tags',
        many=True,
        write_only=True
    )
    
    class Meta:
        model = Article
        fields = [
            'id', 'title', 'slug', 'content', 'excerpt',
            'author', 'author_id',
            'category', 'category_id',
            'tags', 'tag_ids',
            'status', 'featured_image',
            'view_count', 'created_at', 'updated_at'
        ]
        read_only_fields = ['id', 'view_count', 'created_at', 'updated_at']
    
    def create(self, validated_data):
        tags = validated_data.pop('tags', [])
        article = Article.objects.create(**validated_data)
        article.tags.set(tags)
        return article
    
    def update(self, instance, validated_data):
        tags = validated_data.pop('tags', None)
        
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        
        if tags is not None:
            instance.tags.set(tags)
        
        return instance
```

### Extra kwargs และ Validators

```python
# ตัวอย่างที่ 9: Extra kwargs และ Validators
from rest_framework.validators import UniqueValidator, UniqueTogetherValidator

class ArticleCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['title', 'slug', 'content', 'excerpt', 'category', 'tags', 'status']
        extra_kwargs = {
            'slug': {
                'validators': [
                    UniqueValidator(
                        queryset=Article.objects.all(),
                        message="Slug must be unique"
                    )
                ]
            },
            'content': {
                'min_length': 100,
                'error_messages': {
                    'min_length': 'Content must be at least 100 characters'
                }
            },
            'excerpt': {'required': False, 'allow_blank': True},
        }
        validators = [
            UniqueTogetherValidator(
                queryset=Article.objects.all(),
                fields=['title', 'author']
            )
        ]
    
    def validate_slug(self, value):
        """Auto-generate slug จาก title ถ้าไม่ได้ระบุ"""
        if not value:
            from django.utils.text import slugify
            value = slugify(self.initial_data.get('title', ''))
        return value
```

---

## 4. APIView

APIView เป็น class-based view ที่ DRF ให้มา ซึ่ง handle HTTP methods ต่างๆ และมี built-in authentication, permission checking

### Basic APIView

```python
# ตัวอย่างที่ 10: Basic APIView
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404
from .models import Article
from .serializers import ArticleListSerializer, ArticleDetailSerializer

class ArticleListView(APIView):
    """
    GET  /api/articles/    - list all articles
    POST /api/articles/    - create new article
    """
    
    def get(self, request):
        """List articles"""
        articles = Article.objects.filter(status='published')
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        """Create new article"""
        serializer = ArticleDetailSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)
            return Response(
                serializer.data,
                status=status.HTTP_201_CREATED
            )
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )

class ArticleDetailView(APIView):
    """
    GET    /api/articles/{id}/   - get article detail
    PUT    /api/articles/{id}/   - update article (full)
    PATCH  /api/articles/{id}/   - update article (partial)
    DELETE /api/articles/{id}/   - delete article
    """
    
    def get_object(self, pk):
        return get_object_or_404(Article, pk=pk)
    
    def get(self, request, pk):
        article = self.get_object(pk)
        # เพิ่ม view count
        article.view_count += 1
        article.save(update_fields=['view_count'])
        
        serializer = ArticleDetailSerializer(article)
        return Response(serializer.data)
    
    def put(self, request, pk):
        article = self.get_object(pk)
        serializer = ArticleDetailSerializer(article, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def patch(self, request, pk):
        article = self.get_object(pk)
        serializer = ArticleDetailSerializer(
            article, data=request.data, partial=True
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk):
        article = self.get_object(pk)
        article.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### APIView with Permission Checking

```python
# ตัวอย่างที่ 11: APIView with Permission
from rest_framework.permissions import IsAuthenticated, IsAdminUser

class ArticleListView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        # request.user มีให้ใช้
        user_articles = Article.objects.filter(author=request.user)
        serializer = ArticleListSerializer(user_articles, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        serializer = ArticleDetailSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class AdminArticleView(APIView):
    permission_classes = [IsAdminUser]
    
    def get(self, request):
        """Admin เท่านั้นที่ดูได้ทุก articles"""
        articles = Article.objects.all()
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
```

### Function-based Views (FBV)

```python
# ตัวอย่างที่ 12: Function-based Views ด้วย @api_view decorator
from rest_framework.decorators import api_view, permission_classes, authentication_classes
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def article_list(request):
    """
    ฟังก์ชันสำหรับ list และ create articles
    """
    if request.method == 'GET':
        articles = Article.objects.filter(status='published')
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = ArticleDetailSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
@permission_classes([IsAuthenticated])
def article_detail(request, pk):
    """
    ฟังก์ชันสำหรับ retrieve, update, delete article
    """
    article = get_object_or_404(Article, pk=pk)
    
    if request.method == 'GET':
        serializer = ArticleDetailSerializer(article)
        return Response(serializer.data)
    
    elif request.method in ['PUT', 'PATCH']:
        partial = request.method == 'PATCH'
        serializer = ArticleDetailSerializer(
            article, data=request.data, partial=partial
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        article.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Generic Views

```python
# ตัวอย่างที่ 13: Generic Views - ลดการเขียนโค้ดซ้ำ
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated

class ArticleListCreateView(generics.ListCreateAPIView):
    """
    GET  /api/articles/  - list
    POST /api/articles/  - create
    """
    queryset = Article.objects.filter(status='published')
    serializer_class = ArticleListSerializer
    permission_classes = [IsAuthenticated]
    
    def get_serializer_class(self):
        """ใช้ serializer ต่างกันตาม method"""
        if self.request.method == 'POST':
            return ArticleDetailSerializer
        return ArticleListSerializer
    
    def perform_create(self, serializer):
        """Override เพื่อ inject author"""
        serializer.save(author=self.request.user)

class ArticleRetrieveUpdateDestroyView(generics.RetrieveUpdateDestroyAPIView):
    """
    GET    /api/articles/{id}/  - retrieve
    PUT    /api/articles/{id}/  - full update
    PATCH  /api/articles/{id}/  - partial update
    DELETE /api/articles/{id}/  - destroy
    """
    queryset = Article.objects.all()
    serializer_class = ArticleDetailSerializer
    permission_classes = [IsAuthenticated]
    
    def get_object(self):
        obj = super().get_object()
        # ตรวจสอบ permission เพิ่มเติม
        if obj.author != self.request.user and not self.request.user.is_staff:
            from rest_framework.exceptions import PermissionDenied
            raise PermissionDenied("You don't have permission to edit this article")
        return obj
```

---

## 5. ViewSets

ViewSet รวม logic สำหรับ set ของ related views เข้าไว้ด้วยกัน ช่วยลดการซ้ำซ้อนของโค้ด

### ModelViewSet

```python
# ตัวอย่างที่ 14: ModelViewSet - Full CRUD ด้วยโค้ดน้อยที่สุด
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAdminUser

class ArticleViewSet(viewsets.ModelViewSet):
    """
    ViewSet สำหรับ Article - รองรับ CRUD operations ทั้งหมด
    
    list:   GET  /articles/
    create: POST /articles/
    retrieve: GET /articles/{id}/
    update: PUT /articles/{id}/
    partial_update: PATCH /articles/{id}/
    destroy: DELETE /articles/{id}/
    """
    queryset = Article.objects.all()
    serializer_class = ArticleDetailSerializer
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        """Override queryset ตาม request"""
        queryset = Article.objects.all()
        
        # Filter ตาม status
        status_filter = self.request.query_params.get('status')
        if status_filter:
            queryset = queryset.filter(status=status_filter)
        
        # Filter ตาม author
        author_id = self.request.query_params.get('author')
        if author_id:
            queryset = queryset.filter(author_id=author_id)
        
        return queryset.select_related('author', 'category').prefetch_related('tags')
    
    def get_serializer_class(self):
        """ใช้ serializer ต่างกันตาม action"""
        if self.action == 'list':
            return ArticleListSerializer
        return ArticleDetailSerializer
    
    def get_permissions(self):
        """ตั้งค่า permission ตาม action"""
        if self.action in ['list', 'retrieve']:
            permission_classes = []  # Public access
        elif self.action in ['create']:
            permission_classes = [IsAuthenticated]
        else:
            permission_classes = [IsAuthenticated]
        return [permission() for permission in permission_classes]
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
    
    def perform_update(self, serializer):
        serializer.save()
    
    # Custom actions
    @action(detail=True, methods=['post'], permission_classes=[IsAuthenticated])
    def publish(self, request, pk=None):
        """POST /articles/{id}/publish/ - เปลี่ยนสถานะเป็น published"""
        article = self.get_object()
        if article.author != request.user:
            return Response(
                {'error': 'Only author can publish'},
                status=status.HTTP_403_FORBIDDEN
            )
        article.status = 'published'
        article.save()
        serializer = self.get_serializer(article)
        return Response(serializer.data)
    
    @action(detail=True, methods=['get'])
    def comments(self, request, pk=None):
        """GET /articles/{id}/comments/ - ดึง comments ของ article"""
        article = self.get_object()
        comments = article.comments.filter(is_approved=True, parent=None)
        serializer = CommentSerializer(comments, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def my_articles(self, request):
        """GET /articles/my_articles/ - ดึง articles ของ user ปัจจุบัน"""
        articles = Article.objects.filter(author=request.user)
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def trending(self, request):
        """GET /articles/trending/ - top 10 articles ที่มี view count สูงสุด"""
        articles = Article.objects.filter(
            status='published'
        ).order_by('-view_count')[:10]
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
```

### ReadOnlyModelViewSet

```python
# ตัวอย่างที่ 15: ReadOnlyModelViewSet - สำหรับ read-only endpoints
class CategoryViewSet(viewsets.ReadOnlyModelViewSet):
    """
    ViewSet แบบ read-only
    list:     GET /categories/
    retrieve: GET /categories/{id}/
    """
    queryset = Category.objects.all()
    serializer_class = CategorySerializer
    
    @action(detail=True, methods=['get'])
    def articles(self, request, pk=None):
        """GET /categories/{id}/articles/ - articles ของ category นี้"""
        category = self.get_object()
        articles = category.articles.filter(status='published')
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
```

### Custom ViewSet

```python
# ตัวอย่างที่ 16: Custom ViewSet
class UserViewSet(viewsets.ViewSet):
    """Custom ViewSet ที่ implement เฉพาะ actions ที่ต้องการ"""
    
    def list(self, request):
        """GET /users/"""
        users = User.objects.all()
        serializer = UserSerializer(users, many=True)
        return Response(serializer.data)
    
    def retrieve(self, request, pk=None):
        """GET /users/{id}/"""
        user = get_object_or_404(User, pk=pk)
        serializer = UserSerializer(user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def me(self, request):
        """GET /users/me/ - profile ของ user ปัจจุบัน"""
        serializer = UserSerializer(request.user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['patch'], permission_classes=[IsAuthenticated])
    def update_profile(self, request):
        """PATCH /users/update_profile/ - update profile"""
        serializer = UserProfileSerializer(
            request.user,
            data=request.data,
            partial=True
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

---

## 6. Routers

Router ทำหน้าที่ map ViewSets ไปยัง URL patterns โดยอัตโนมัติ

### BasicRouter และ DefaultRouter

```python
# ตัวอย่างที่ 17: Routers
# api/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter, SimpleRouter
from . import views

# DefaultRouter - มี API root view
router = DefaultRouter()
router.register(r'articles', views.ArticleViewSet, basename='article')
router.register(r'categories', views.CategoryViewSet, basename='category')
router.register(r'users', views.UserViewSet, basename='user')
router.register(r'comments', views.CommentViewSet, basename='comment')

# URL patterns ที่ router สร้างให้อัตโนมัติ:
# GET  /articles/             -> article-list
# POST /articles/             -> article-list
# GET  /articles/{id}/        -> article-detail
# PUT  /articles/{id}/        -> article-detail
# PATCH /articles/{id}/       -> article-detail
# DELETE /articles/{id}/      -> article-detail
# POST /articles/{id}/publish/  -> article-publish (custom action)
# GET  /articles/{id}/comments/ -> article-comments (custom action)
# GET  /articles/my_articles/   -> article-my-articles (custom action)
# GET  /articles/trending/      -> article-trending (custom action)

urlpatterns = [
    path('', include(router.urls)),
    
    # เพิ่ม non-router URLs
    path('auth/login/', views.LoginView.as_view(), name='login'),
    path('auth/logout/', views.LogoutView.as_view(), name='logout'),
    path('auth/register/', views.RegisterView.as_view(), name='register'),
]
```

```python
# myapi/urls.py
from django.contrib import admin
from django.urls import path, include
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView, SpectacularRedocView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('api.urls')),
    
    # API Documentation
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    path('api/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
    
    # DRF browsable API auth
    path('api-auth/', include('rest_framework.urls')),
]
```

---

## 7. Authentication

### Token Authentication

```python
# ตัวอย่างที่ 18: Token Authentication
# settings.py เพิ่ม
INSTALLED_APPS += ['rest_framework.authtoken']

# หลัง migrate
# python manage.py migrate

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

```python
# api/views.py
from rest_framework.authtoken.models import Token
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.response import Response
from rest_framework import status

class LoginView(ObtainAuthToken):
    """Custom login view ที่ return ข้อมูลเพิ่มเติม"""
    
    def post(self, request, *args, **kwargs):
        serializer = self.serializer_class(
            data=request.data,
            context={'request': request}
        )
        serializer.is_valid(raise_exception=True)
        user = serializer.validated_data['user']
        
        # สร้างหรือดึง token
        token, created = Token.objects.get_or_create(user=user)
        
        return Response({
            'token': token.key,
            'user_id': user.pk,
            'username': user.username,
            'email': user.email,
        })

class LogoutView(APIView):
    """ลบ token เมื่อ logout"""
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        try:
            request.user.auth_token.delete()
        except Token.DoesNotExist:
            pass
        return Response(
            {'message': 'Logged out successfully'},
            status=status.HTTP_200_OK
        )
```

### JWT Authentication

```python
# ตัวอย่างที่ 19: JWT Authentication ด้วย SimpleJWT
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
    TokenBlacklistView,
)

urlpatterns = [
    path('auth/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('auth/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('auth/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
    path('auth/token/blacklist/', TokenBlacklistView.as_view(), name='token_blacklist'),
]
```

```python
# ตัวอย่างที่ 20: Custom JWT claims
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView

class CustomTokenObtainPairSerializer(TokenObtainPairSerializer):
    @classmethod
    def get_token(cls, user):
        token = super().get_token(user)
        
        # เพิ่ม custom claims
        token['username'] = user.username
        token['email'] = user.email
        token['is_staff'] = user.is_staff
        token['is_superuser'] = user.is_superuser
        
        return token
    
    def validate(self, attrs):
        data = super().validate(attrs)
        
        # เพิ่ม user info ใน response
        data['user'] = {
            'id': self.user.id,
            'username': self.user.username,
            'email': self.user.email,
        }
        
        return data

class CustomTokenObtainPairView(TokenObtainPairView):
    serializer_class = CustomTokenObtainPairSerializer
```

### Session Authentication

```python
# ตัวอย่างที่ 21: Session Authentication
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
}
```

### Custom Authentication

```python
# ตัวอย่างที่ 22: Custom Authentication
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
import hmac
import hashlib
import time

class APIKeyAuthentication(BaseAuthentication):
    """Authentication ด้วย API Key"""
    
    def authenticate(self, request):
        api_key = request.META.get('HTTP_X_API_KEY')
        
        if not api_key:
            return None  # ไม่ใช่ authentication method นี้
        
        try:
            # ดึง user จาก API key
            from .models import APIKey
            key_obj = APIKey.objects.select_related('user').get(key=api_key, is_active=True)
        except APIKey.DoesNotExist:
            raise AuthenticationFailed('Invalid API Key')
        
        # ตรวจสอบ expiry
        if key_obj.expires_at and key_obj.expires_at < timezone.now():
            raise AuthenticationFailed('API Key expired')
        
        # Update last used
        key_obj.last_used_at = timezone.now()
        key_obj.save(update_fields=['last_used_at'])
        
        return (key_obj.user, key_obj)
    
    def authenticate_header(self, request):
        return 'APIKey'
```

---

## 8. Permissions

### Built-in Permissions

```python
# ตัวอย่างที่ 23: Built-in Permissions
from rest_framework.permissions import (
    AllowAny,           # ทุกคนเข้าถึงได้
    IsAuthenticated,    # ต้อง login
    IsAdminUser,        # ต้องเป็น admin (is_staff=True)
    IsAuthenticatedOrReadOnly,  # Read: ทุกคน, Write: ต้อง login
)

class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    
    def get_permissions(self):
        """Dynamic permission ตาม action"""
        if self.action in ['list', 'retrieve']:
            # Public - ทุกคนดูได้
            permission_classes = [AllowAny]
        elif self.action == 'create':
            # ต้อง login เพื่อสร้าง
            permission_classes = [IsAuthenticated]
        else:
            # แก้ไข/ลบต้องเป็น staff หรือเจ้าของ
            permission_classes = [IsAuthenticated]
        
        return [permission() for permission in permission_classes]
```

### Custom Permissions

```python
# ตัวอย่างที่ 24: Custom Permissions
from rest_framework.permissions import BasePermission

class IsOwner(BasePermission):
    """อนุญาตเฉพาะเจ้าของ object"""
    
    message = "You must be the owner of this object"
    
    def has_object_permission(self, request, view, obj):
        return obj.author == request.user

class IsOwnerOrReadOnly(BasePermission):
    """Read: ทุกคน, Write: เฉพาะเจ้าของ"""
    
    def has_permission(self, request, view):
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        return request.user.is_authenticated
    
    def has_object_permission(self, request, view, obj):
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        return obj.author == request.user

class IsVerifiedUser(BasePermission):
    """ต้องเป็น user ที่ verify email แล้ว"""
    
    message = "Email verification required"
    
    def has_permission(self, request, view):
        return (
            request.user.is_authenticated and
            hasattr(request.user, 'profile') and
            request.user.profile.email_verified
        )

class CanPublishArticle(BasePermission):
    """ตรวจสอบสิทธิ์การ publish"""
    
    def has_permission(self, request, view):
        if not request.user.is_authenticated:
            return False
        return (
            request.user.is_staff or
            request.user.has_perm('api.can_publish_article')
        )
```

```python
# ตัวอย่างที่ 25: ใช้ Custom Permissions
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleDetailSerializer
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]
        elif self.action == 'create':
            return [IsAuthenticated(), IsVerifiedUser()]
        elif self.action == 'publish':
            return [IsAuthenticated(), CanPublishArticle()]
        else:
            return [IsAuthenticated(), IsOwnerOrReadOnly()]
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        article = self.get_object()
        article.status = 'published'
        article.save()
        return Response({'status': 'published'})
```

---

## 9. Pagination

```python
# ตัวอย่างที่ 26: Custom Pagination Classes
from rest_framework.pagination import (
    PageNumberPagination,
    LimitOffsetPagination,
    CursorPagination
)
from rest_framework.response import Response

class StandardPagination(PageNumberPagination):
    """Pagination แบบ page number"""
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100
    
    def get_paginated_response(self, data):
        return Response({
            'pagination': {
                'count': self.page.paginator.count,
                'next': self.get_next_link(),
                'previous': self.get_previous_link(),
                'current_page': self.page.number,
                'total_pages': self.page.paginator.num_pages,
            },
            'results': data
        })
    
    def get_paginated_response_schema(self, schema):
        return {
            'type': 'object',
            'properties': {
                'pagination': {
                    'type': 'object',
                    'properties': {
                        'count': {'type': 'integer'},
                        'next': {'type': 'string', 'nullable': True},
                        'previous': {'type': 'string', 'nullable': True},
                        'current_page': {'type': 'integer'},
                        'total_pages': {'type': 'integer'},
                    }
                },
                'results': schema,
            }
        }

class SmallPagination(PageNumberPagination):
    page_size = 5
    page_size_query_param = 'page_size'
    max_page_size = 20

class LargeResultsPagination(LimitOffsetPagination):
    """Pagination แบบ limit/offset"""
    default_limit = 10
    max_limit = 100
    
    def get_paginated_response(self, data):
        return Response({
            'count': self.count,
            'next': self.get_next_link(),
            'previous': self.get_previous_link(),
            'limit': self.limit,
            'offset': self.offset,
            'results': data
        })

class ArticleCursorPagination(CursorPagination):
    """Pagination แบบ cursor (สำหรับ real-time data)"""
    page_size = 10
    ordering = '-created_at'
    cursor_query_param = 'cursor'
```

```python
# ตัวอย่างที่ 27: ใช้ Pagination ใน ViewSet
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.filter(status='published')
    serializer_class = ArticleListSerializer
    pagination_class = StandardPagination  # หรือ LargeResultsPagination
    
    def list(self, request):
        queryset = self.filter_queryset(self.get_queryset())
        
        # Paginate
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```

---

## 10. Filtering

### Django Filter Backend

```python
# ตัวอย่างที่ 28: Django Filter Backend
# pip install django-filter

import django_filters
from .models import Article

class ArticleFilter(django_filters.FilterSet):
    # ตัวกรองพื้นฐาน
    title = django_filters.CharFilter(lookup_expr='icontains')
    author_username = django_filters.CharFilter(
        field_name='author__username',
        lookup_expr='icontains'
    )
    category = django_filters.ModelChoiceFilter(queryset=Category.objects.all())
    status = django_filters.ChoiceFilter(choices=Article.STATUS_CHOICES)
    
    # Range filters
    created_after = django_filters.DateTimeFilter(
        field_name='created_at',
        lookup_expr='gte'
    )
    created_before = django_filters.DateTimeFilter(
        field_name='created_at',
        lookup_expr='lte'
    )
    min_views = django_filters.NumberFilter(
        field_name='view_count',
        lookup_expr='gte'
    )
    
    # Multiple choice
    tags = django_filters.ModelMultipleChoiceFilter(
        queryset=Tag.objects.all()
    )
    
    class Meta:
        model = Article
        fields = {
            'status': ['exact'],
            'created_at': ['gte', 'lte', 'gt', 'lt'],
            'view_count': ['gte', 'lte'],
        }
```

```python
# ตัวอย่างที่ 29: ใช้ FilterSet ใน ViewSet
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleListSerializer
    filterset_class = ArticleFilter
    
    # Search fields (ใช้ SearchFilter)
    search_fields = [
        'title',
        'content',
        '^author__username',  # ^ = starts with
        '@excerpt',           # @ = full text search
    ]
    
    # Ordering fields (ใช้ OrderingFilter)
    ordering_fields = ['created_at', 'view_count', 'title']
    ordering = ['-created_at']  # Default ordering
```

---

## 11. Throttling

```python
# ตัวอย่างที่ 30: Custom Throttling
from rest_framework.throttling import (
    AnonRateThrottle,
    UserRateThrottle,
    ScopedRateThrottle,
    BaseThrottle
)

class BurstRateThrottle(UserRateThrottle):
    """Rate limit สำหรับ burst requests"""
    scope = 'burst'

class SustainedRateThrottle(UserRateThrottle):
    """Rate limit สำหรับ sustained requests"""
    scope = 'sustained'

class PublicAPIThrottle(AnonRateThrottle):
    """Rate limit สำหรับ public API"""
    rate = '50/hour'

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'api.throttling.BurstRateThrottle',
        'api.throttling.SustainedRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'burst': '10/min',
        'sustained': '200/day',
        'anon': '100/day',
    }
}
```

```python
# ตัวอย่างที่ 31: Per-view Throttling
class ArticleViewSet(viewsets.ModelViewSet):
    throttle_classes = [BurstRateThrottle, SustainedRateThrottle]
    
    @action(detail=False, methods=['post'])
    def bulk_create(self, request):
        """Endpoint ที่ต้องการ throttling พิเศษ"""
        # ...
        pass

class PublicArticleView(APIView):
    throttle_classes = [PublicAPIThrottle]
    permission_classes = [AllowAny]
    
    def get(self, request):
        articles = Article.objects.filter(status='published')
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)
```

---

## 12. API Documentation

### drf-spectacular

```python
# ตัวอย่างที่ 32: API Documentation ด้วย drf-spectacular
# pip install drf-spectacular

from drf_spectacular.utils import (
    extend_schema,
    extend_schema_view,
    OpenApiParameter,
    OpenApiTypes,
    OpenApiResponse,
    inline_serializer
)

@extend_schema_view(
    list=extend_schema(
        summary='List all articles',
        description='Returns a paginated list of published articles',
        parameters=[
            OpenApiParameter(
                name='status',
                type=OpenApiTypes.STR,
                location=OpenApiParameter.QUERY,
                description='Filter by status',
                enum=['draft', 'published', 'archived']
            ),
            OpenApiParameter(
                name='search',
                type=OpenApiTypes.STR,
                location=OpenApiParameter.QUERY,
                description='Search in title and content'
            ),
        ],
        responses={
            200: ArticleListSerializer(many=True),
            401: OpenApiResponse(description='Unauthorized'),
        },
        tags=['Articles']
    ),
    create=extend_schema(
        summary='Create article',
        description='Create a new article',
        request=ArticleDetailSerializer,
        responses={
            201: ArticleDetailSerializer,
            400: OpenApiResponse(description='Bad Request'),
        },
        tags=['Articles']
    ),
)
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleDetailSerializer
    
    @extend_schema(
        summary='Publish article',
        description='Change article status to published',
        responses={
            200: ArticleDetailSerializer,
            403: OpenApiResponse(description='Forbidden'),
        },
        tags=['Articles']
    )
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        article = self.get_object()
        article.status = 'published'
        article.save()
        serializer = self.get_serializer(article)
        return Response(serializer.data)
```

### Custom Exception Handler

```python
# ตัวอย่างที่ 33: Custom Exception Handler
from rest_framework.views import exception_handler
from rest_framework.exceptions import ValidationError, NotFound
import logging

logger = logging.getLogger(__name__)

def custom_exception_handler(exc, context):
    """Custom exception handler ที่ return response format เดียวกัน"""
    response = exception_handler(exc, context)
    
    if response is not None:
        # ปรับ response format
        error_data = {
            'success': False,
            'error': {
                'status_code': response.status_code,
                'message': '',
                'details': response.data
            }
        }
        
        if isinstance(exc, ValidationError):
            error_data['error']['message'] = 'Validation error'
        elif isinstance(exc, NotFound):
            error_data['error']['message'] = 'Resource not found'
        else:
            error_data['error']['message'] = str(exc)
        
        response.data = error_data
        
        # Log errors
        if response.status_code >= 500:
            logger.error(f"Server error: {exc}", exc_info=True)
    
    return response
```

### Complete URLs Configuration

```python
# ตัวอย่างที่ 34: Complete URLs configuration
# api/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)
from . import views

router = DefaultRouter()
router.register(r'articles', views.ArticleViewSet)
router.register(r'categories', views.CategoryViewSet)
router.register(r'comments', views.CommentViewSet)
router.register(r'users', views.UserViewSet)

urlpatterns = [
    # Auth endpoints
    path('auth/register/', views.RegisterView.as_view(), name='register'),
    path('auth/login/', views.LoginView.as_view(), name='login'),
    path('auth/logout/', views.LogoutView.as_view(), name='logout'),
    path('auth/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('auth/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('auth/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
    
    # API endpoints
    path('', include(router.urls)),
]
```

### Complete View Example

```python
# ตัวอย่างที่ 35: Complete Blog API ViewSet
class CommentViewSet(viewsets.ModelViewSet):
    """ViewSet สำหรับ Comments"""
    serializer_class = CommentSerializer
    permission_classes = [IsAuthenticated]
    pagination_class = StandardPagination
    
    def get_queryset(self):
        return Comment.objects.filter(
            is_approved=True,
            parent=None
        ).select_related('author', 'article').prefetch_related('replies')
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]
        return [IsAuthenticated()]
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
    
    @action(detail=True, methods=['post'], permission_classes=[IsAuthenticated])
    def reply(self, request, pk=None):
        """POST /comments/{id}/reply/ - ตอบกลับ comment"""
        parent_comment = self.get_object()
        
        if parent_comment.parent is not None:
            return Response(
                {'error': 'Cannot reply to a reply'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        serializer = CommentSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(
                author=request.user,
                article=parent_comment.article,
                parent=parent_comment
            )
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=True, methods=['post'], permission_classes=[IsAdminUser])
    def approve(self, request, pk=None):
        """POST /comments/{id}/approve/ - Admin approve comment"""
        comment = self.get_object()
        comment.is_approved = True
        comment.save()
        return Response({'status': 'approved'})
```

### Testing DRF APIs

```python
# ตัวอย่างที่ 36: Testing DRF APIs
from rest_framework.test import APITestCase, APIClient
from rest_framework import status
from django.contrib.auth.models import User
from django.urls import reverse
from .models import Article, Category

class ArticleAPITests(APITestCase):
    
    def setUp(self):
        """Setup ข้อมูลสำหรับ testing"""
        self.client = APIClient()
        
        # สร้าง users
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123',
            email='test@example.com'
        )
        self.admin = User.objects.create_user(
            username='admin',
            password='adminpass123',
            is_staff=True
        )
        
        # สร้าง category
        self.category = Category.objects.create(
            name='Technology',
            slug='technology'
        )
        
        # สร้าง articles
        self.article = Article.objects.create(
            title='Test Article',
            slug='test-article',
            content='This is test content with more than 100 characters for validation purposes in our test suite.',
            author=self.user,
            category=self.category,
            status='published'
        )
    
    def test_list_articles_anonymous(self):
        """Anonymous user สามารถดู list articles ได้"""
        url = reverse('article-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
    
    def test_create_article_authenticated(self):
        """Authenticated user สร้าง article ได้"""
        self.client.force_authenticate(user=self.user)
        
        url = reverse('article-list')
        data = {
            'title': 'New Article',
            'slug': 'new-article',
            'content': 'Content must be at least 100 characters long to pass validation in the test scenario.',
            'category': self.category.id,
            'status': 'draft'
        }
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Article.objects.count(), 2)
    
    def test_create_article_anonymous_forbidden(self):
        """Anonymous user สร้าง article ไม่ได้"""
        url = reverse('article-list')
        data = {'title': 'Test', 'content': 'Test'}
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_update_article_owner(self):
        """เจ้าของ article แก้ไขได้"""
        self.client.force_authenticate(user=self.user)
        url = reverse('article-detail', kwargs={'pk': self.article.pk})
        data = {'title': 'Updated Title'}
        response = self.client.patch(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.article.refresh_from_db()
        self.assertEqual(self.article.title, 'Updated Title')
    
    def test_delete_article_non_owner(self):
        """User อื่นลบ article ไม่ได้"""
        other_user = User.objects.create_user(
            username='other', password='pass123'
        )
        self.client.force_authenticate(user=other_user)
        url = reverse('article-detail', kwargs={'pk': self.article.pk})
        response = self.client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_jwt_authentication(self):
        """ทดสอบ JWT authentication"""
        # Login
        url = reverse('token_obtain_pair')
        data = {'username': 'testuser', 'password': 'testpass123'}
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        token = response.data['access']
        
        # ใช้ token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {token}')
        url = reverse('article-list')
        response = self.client.post(url, {'title': 'Test'}, format='json')
        # ต้อง authenticated แล้ว (แม้ validation จะ fail)
        self.assertNotEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_pagination(self):
        """ทดสอบ pagination"""
        # สร้าง articles มากกว่า page size
        for i in range(15):
            Article.objects.create(
                title=f'Article {i}',
                slug=f'article-{i}',
                content='Content' * 20,
                author=self.user,
                status='published'
            )
        
        url = reverse('article-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('pagination', response.data)
        self.assertEqual(len(response.data['results']), 10)
    
    def test_filter_by_status(self):
        """ทดสอบ filtering"""
        Article.objects.create(
            title='Draft Article',
            slug='draft-article',
            content='Content' * 20,
            author=self.user,
            status='draft'
        )
        
        url = reverse('article-list')
        response = self.client.get(url, {'status': 'published'})
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        for article in response.data['results']:
            self.assertEqual(article['status'], 'published')
    
    def test_search(self):
        """ทดสอบ search functionality"""
        url = reverse('article-list')
        response = self.client.get(url, {'search': 'Test Article'})
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(len(response.data['results']) > 0)
```

### Response Helpers

```python
# ตัวอย่างที่ 37: Custom Response Helpers
from rest_framework.response import Response
from rest_framework import status

class APIResponse:
    """Helper class สำหรับสร้าง consistent API responses"""
    
    @staticmethod
    def success(data=None, message='Success', status_code=status.HTTP_200_OK):
        return Response({
            'success': True,
            'message': message,
            'data': data
        }, status=status_code)
    
    @staticmethod
    def created(data=None, message='Created successfully'):
        return Response({
            'success': True,
            'message': message,
            'data': data
        }, status=status.HTTP_201_CREATED)
    
    @staticmethod
    def error(message='Error occurred', errors=None, status_code=status.HTTP_400_BAD_REQUEST):
        return Response({
            'success': False,
            'message': message,
            'errors': errors
        }, status=status_code)
    
    @staticmethod
    def not_found(message='Resource not found'):
        return Response({
            'success': False,
            'message': message
        }, status=status.HTTP_404_NOT_FOUND)
    
    @staticmethod
    def forbidden(message='Access forbidden'):
        return Response({
            'success': False,
            'message': message
        }, status=status.HTTP_403_FORBIDDEN)

# การใช้งาน
class ArticleView(APIView):
    def get(self, request, pk):
        try:
            article = Article.objects.get(pk=pk)
            serializer = ArticleDetailSerializer(article)
            return APIResponse.success(serializer.data)
        except Article.DoesNotExist:
            return APIResponse.not_found('Article not found')
    
    def post(self, request):
        serializer = ArticleDetailSerializer(data=request.data)
        if serializer.is_valid():
            article = serializer.save(author=request.user)
            return APIResponse.created(serializer.data, 'Article created')
        return APIResponse.error('Validation failed', serializer.errors)
```

### Mixins

```python
# ตัวอย่างที่ 38: Custom Mixins
from rest_framework import mixins

class SoftDeleteMixin:
    """Mixin สำหรับ soft delete (ไม่ลบจริง)"""
    
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.is_deleted = True
        instance.deleted_at = timezone.now()
        instance.save()
        return Response(status=status.HTTP_204_NO_CONTENT)
    
    def get_queryset(self):
        return super().get_queryset().filter(is_deleted=False)

class AuditMixin:
    """Mixin สำหรับ audit trail"""
    
    def perform_create(self, serializer):
        serializer.save(
            created_by=self.request.user,
            modified_by=self.request.user
        )
    
    def perform_update(self, serializer):
        serializer.save(modified_by=self.request.user)

class ArticleViewSet(AuditMixin, SoftDeleteMixin, viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```

### Advanced Serializer Patterns

```python
# ตัวอย่างที่ 39: Dynamic Fields Serializer
class DynamicFieldsSerializer(serializers.ModelSerializer):
    """Serializer ที่รับ fields parameter จาก query string"""
    
    def __init__(self, *args, **kwargs):
        fields = kwargs.pop('fields', None)
        super().__init__(*args, **kwargs)
        
        if fields is not None:
            allowed = set(fields)
            existing = set(self.fields)
            for field_name in existing - allowed:
                self.fields.pop(field_name)

class ArticleSerializer(DynamicFieldsSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'created_at']

# การใช้งาน: /api/articles/?fields=id,title,author
class ArticleViewSet(viewsets.ModelViewSet):
    def get_serializer(self, *args, **kwargs):
        fields = self.request.query_params.get('fields')
        if fields:
            kwargs['fields'] = fields.split(',')
        return super().get_serializer(*args, **kwargs)
```

```python
# ตัวอย่างที่ 40: Writable Nested Serializer
class CommentSerializer(serializers.ModelSerializer):
    replies = serializers.SerializerMethodField()
    
    class Meta:
        model = Comment
        fields = ['id', 'content', 'author', 'created_at', 'replies']
        read_only_fields = ['id', 'author', 'created_at']
    
    def get_replies(self, obj):
        if obj.replies.exists():
            return CommentSerializer(obj.replies.all(), many=True).data
        return []

class ArticleWithCommentsSerializer(serializers.ModelSerializer):
    comments = CommentSerializer(many=True, required=False)
    
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'comments']
    
    def create(self, validated_data):
        comments_data = validated_data.pop('comments', [])
        article = Article.objects.create(**validated_data)
        
        for comment_data in comments_data:
            Comment.objects.create(article=article, **comment_data)
        
        return article
    
    def update(self, instance, validated_data):
        comments_data = validated_data.pop('comments', None)
        
        # Update article
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        
        # Update comments ถ้ามีส่งมา
        if comments_data is not None:
            instance.comments.all().delete()
            for comment_data in comments_data:
                Comment.objects.create(article=instance, **comment_data)
        
        return instance
```

---

## 13. แบบฝึกหัด

### ข้อ 1: User Profile API

สร้าง API สำหรับ User Profile management

**โจทย์:**
สร้าง endpoint สำหรับ:
- GET /api/profile/ - ดู profile ของตัวเอง
- PUT /api/profile/ - แก้ไข profile
- POST /api/profile/avatar/ - upload avatar image
- GET /api/profile/{username}/ - ดู profile ของ user อื่น

**เฉลย:**

```python
# models.py
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    website = models.URLField(blank=True)
    location = models.CharField(max_length=100, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    is_private = models.BooleanField(default=False)
    followers = models.ManyToManyField('self', symmetrical=False, related_name='following', blank=True)
    
    def __str__(self):
        return f"Profile of {self.user.username}"

# serializers.py
class UserProfileSerializer(serializers.ModelSerializer):
    username = serializers.CharField(source='user.username', read_only=True)
    email = serializers.EmailField(source='user.email', read_only=True)
    first_name = serializers.CharField(source='user.first_name')
    last_name = serializers.CharField(source='user.last_name')
    follower_count = serializers.SerializerMethodField()
    following_count = serializers.SerializerMethodField()
    
    class Meta:
        model = UserProfile
        fields = [
            'username', 'email', 'first_name', 'last_name',
            'bio', 'avatar', 'website', 'location', 'birth_date',
            'follower_count', 'following_count'
        ]
    
    def get_follower_count(self, obj):
        return obj.followers.count()
    
    def get_following_count(self, obj):
        return obj.following.count()
    
    def update(self, instance, validated_data):
        # Handle nested user data
        user_data = {}
        if 'user' in validated_data:
            user_data = validated_data.pop('user')
        
        # Update profile
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        
        # Update user
        if user_data:
            for attr, value in user_data.items():
                setattr(instance.user, attr, value)
            instance.user.save()
        
        return instance

# views.py
class ProfileView(generics.RetrieveUpdateAPIView):
    serializer_class = UserProfileSerializer
    permission_classes = [IsAuthenticated]
    
    def get_object(self):
        profile, _ = UserProfile.objects.get_or_create(user=self.request.user)
        return profile

class ProfileAvatarView(APIView):
    permission_classes = [IsAuthenticated]
    parser_classes = [MultiPartParser, FormParser]
    
    def post(self, request):
        if 'avatar' not in request.FILES:
            return Response(
                {'error': 'No avatar file provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        profile, _ = UserProfile.objects.get_or_create(user=request.user)
        
        # ลบ avatar เก่า
        if profile.avatar:
            import os
            if os.path.isfile(profile.avatar.path):
                os.remove(profile.avatar.path)
        
        profile.avatar = request.FILES['avatar']
        profile.save()
        
        serializer = UserProfileSerializer(profile)
        return Response(serializer.data)

class PublicProfileView(generics.RetrieveAPIView):
    serializer_class = UserProfileSerializer
    permission_classes = [AllowAny]
    lookup_field = 'user__username'
    lookup_url_kwarg = 'username'
    queryset = UserProfile.objects.filter(is_private=False)
```

### ข้อ 2: Advanced Filtering

สร้าง FilterSet ที่รองรับ:
- ค้นหาด้วย title, content, author
- Filter ตาม date range
- Filter ตาม tag (multiple)
- Filter ตาม minimum views

**เฉลย:**

```python
import django_filters
from .models import Article

class AdvancedArticleFilter(django_filters.FilterSet):
    # Text search
    title = django_filters.CharFilter(lookup_expr='icontains', label='Title contains')
    content = django_filters.CharFilter(lookup_expr='icontains', label='Content contains')
    author_name = django_filters.CharFilter(
        method='filter_author_name',
        label='Author name'
    )
    
    # Date range
    created_after = django_filters.DateFilter(
        field_name='created_at',
        lookup_expr='date__gte',
        label='Created after (YYYY-MM-DD)'
    )
    created_before = django_filters.DateFilter(
        field_name='created_at',
        lookup_expr='date__lte',
        label='Created before (YYYY-MM-DD)'
    )
    
    # Tags (multiple values)
    tags = django_filters.ModelMultipleChoiceFilter(
        queryset=Tag.objects.all(),
        conjoined=False,  # OR logic (any of these tags)
        label='Tags'
    )
    
    # Has all tags (AND logic)
    required_tags = django_filters.ModelMultipleChoiceFilter(
        queryset=Tag.objects.all(),
        conjoined=True,  # AND logic (all of these tags)
        field_name='tags',
        label='Required tags (all)'
    )
    
    # Minimum views
    min_views = django_filters.NumberFilter(
        field_name='view_count',
        lookup_expr='gte',
        label='Minimum views'
    )
    
    # Has featured image
    has_image = django_filters.BooleanFilter(
        method='filter_has_image',
        label='Has featured image'
    )
    
    class Meta:
        model = Article
        fields = ['status', 'category']
    
    def filter_author_name(self, queryset, name, value):
        from django.db.models import Q
        return queryset.filter(
            Q(author__username__icontains=value) |
            Q(author__first_name__icontains=value) |
            Q(author__last_name__icontains=value)
        )
    
    def filter_has_image(self, queryset, name, value):
        if value:
            return queryset.exclude(featured_image='').exclude(featured_image__isnull=True)
        else:
            return queryset.filter(
                models.Q(featured_image='') | models.Q(featured_image__isnull=True)
            )
```

### ข้อ 3: Custom Pagination

สร้าง cursor-based pagination สำหรับ real-time feed

**เฉลย:**

```python
from rest_framework.pagination import CursorPagination
from rest_framework.response import Response

class FeedCursorPagination(CursorPagination):
    """Cursor pagination สำหรับ news feed"""
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 50
    ordering = '-created_at'
    cursor_query_param = 'cursor'
    
    def get_paginated_response(self, data):
        return Response({
            'next_cursor': self.get_next_link(),
            'prev_cursor': self.get_previous_link(),
            'count': len(data),
            'results': data
        })
    
    def get_paginated_response_schema(self, schema):
        return {
            'type': 'object',
            'properties': {
                'next_cursor': {'type': 'string', 'nullable': True},
                'prev_cursor': {'type': 'string', 'nullable': True},
                'count': {'type': 'integer'},
                'results': schema,
            }
        }

class FeedViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Article.objects.filter(status='published')
    serializer_class = ArticleListSerializer
    pagination_class = FeedCursorPagination
    permission_classes = [AllowAny]
```

### ข้อ 4: Rate Limiting per User Tier

สร้าง throttling ที่แตกต่างตาม user tier

**เฉลย:**

```python
from rest_framework.throttling import UserRateThrottle

class FreeTierThrottle(UserRateThrottle):
    scope = 'free'
    
    def get_cache_key(self, request, view):
        if not request.user.is_authenticated:
            return None
        if hasattr(request.user, 'subscription') and request.user.subscription.tier != 'free':
            return None  # ไม่ apply throttle กับ user tier อื่น
        return f'throttle_free_{request.user.pk}'

class PremiumTierThrottle(UserRateThrottle):
    scope = 'premium'
    
    def get_cache_key(self, request, view):
        if not request.user.is_authenticated:
            return None
        if not (hasattr(request.user, 'subscription') and 
                request.user.subscription.tier == 'premium'):
            return None
        return f'throttle_premium_{request.user.pk}'

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'api.throttling.FreeTierThrottle',
        'api.throttling.PremiumTierThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'free': '100/day',
        'premium': '10000/day',
        'anon': '50/day',
    }
}
```

### ข้อ 5: Bulk Operations

สร้าง endpoint สำหรับ bulk create/update/delete

**เฉลย:**

```python
class BulkArticleSerializer(serializers.ListSerializer):
    def create(self, validated_data):
        articles = [Article(**item) for item in validated_data]
        return Article.objects.bulk_create(articles)
    
    def update(self, instance, validated_data):
        article_mapping = {article.id: article for article in instance}
        data_mapping = {item['id']: item for item in validated_data}
        
        result = []
        for article_id, data in data_mapping.items():
            article = article_mapping.get(article_id)
            if article:
                result.append(self.child.update(article, data))
        
        return result

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'status']
        list_serializer_class = BulkArticleSerializer

class BulkArticleView(APIView):
    permission_classes = [IsAuthenticated, IsAdminUser]
    
    def post(self, request):
        """Bulk create"""
        serializer = ArticleSerializer(data=request.data, many=True)
        if serializer.is_valid():
            articles = serializer.save(author=request.user)
            return Response(
                ArticleSerializer(articles, many=True).data,
                status=status.HTTP_201_CREATED
            )
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def put(self, request):
        """Bulk update"""
        ids = [item.get('id') for item in request.data]
        articles = Article.objects.filter(id__in=ids, author=request.user)
        
        serializer = ArticleSerializer(
            articles, data=request.data, many=True
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request):
        """Bulk delete"""
        ids = request.data.get('ids', [])
        if not ids:
            return Response(
                {'error': 'No IDs provided'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        deleted_count, _ = Article.objects.filter(
            id__in=ids,
            author=request.user
        ).delete()
        
        return Response({
            'message': f'Deleted {deleted_count} articles'
        })
```

### ข้อ 6: File Upload API

สร้าง API สำหรับ upload files

**เฉลย:**

```python
from rest_framework.parsers import MultiPartParser, FormParser
import os
from PIL import Image

class FileUploadSerializer(serializers.Serializer):
    file = serializers.FileField()
    title = serializers.CharField(max_length=200, required=False)
    
    def validate_file(self, value):
        # ตรวจสอบขนาดไฟล์ (max 10MB)
        if value.size > 10 * 1024 * 1024:
            raise serializers.ValidationError("File size must be less than 10MB")
        
        # ตรวจสอบ extension
        ext = os.path.splitext(value.name)[1].lower()
        allowed_exts = ['.jpg', '.jpeg', '.png', '.gif', '.pdf', '.doc', '.docx']
        if ext not in allowed_exts:
            raise serializers.ValidationError(f"File type {ext} not allowed")
        
        return value

class ImageUploadView(APIView):
    parser_classes = [MultiPartParser, FormParser]
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        serializer = FileUploadSerializer(data=request.data)
        
        if not serializer.is_valid():
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
        file = serializer.validated_data['file']
        
        # ตรวจสอบว่าเป็น image จริงๆ
        try:
            img = Image.open(file)
            img.verify()
        except Exception:
            return Response(
                {'error': 'Invalid image file'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Reset file pointer
        file.seek(0)
        
        # บันทึก
        from .models import UploadedFile
        upload = UploadedFile.objects.create(
            file=file,
            title=serializer.validated_data.get('title', file.name),
            uploaded_by=request.user
        )
        
        return Response({
            'id': upload.id,
            'url': request.build_absolute_uri(upload.file.url),
            'title': upload.title,
        }, status=status.HTTP_201_CREATED)
```

### ข้อ 7: Versioned API

สร้าง API ที่รองรับ versioning

**เฉลย:**

```python
# urls.py - URL-based versioning
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('api.v1.urls')),
    path('api/v2/', include('api.v2.urls')),
]

# settings.py - Header-based versioning
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version',
}

# views.py - Version-aware view
class ArticleViewSet(viewsets.ModelViewSet):
    
    def get_serializer_class(self):
        version = self.request.version
        
        if version == 'v2':
            return ArticleSerializerV2
        return ArticleSerializer
    
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())
        
        if request.version == 'v2':
            # v2 มี metadata เพิ่มเติม
            data = {
                'version': 'v2',
                'timestamp': timezone.now().isoformat(),
                'articles': self.get_serializer(queryset, many=True).data
            }
            return Response(data)
        
        # v1 format
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```

### ข้อ 8: Complete CRUD API with Tests

สร้าง complete CRUD API พร้อม test coverage 100%

**เฉลย:**

```python
# views.py
class TagViewSet(viewsets.ModelViewSet):
    queryset = Tag.objects.all()
    serializer_class = TagSerializer
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]
        return [IsAuthenticated(), IsAdminUser()]

# tests.py
class TagAPITests(APITestCase):
    def setUp(self):
        self.admin = User.objects.create_user(
            username='admin', password='adminpass', is_staff=True
        )
        self.user = User.objects.create_user(
            username='user', password='userpass'
        )
        self.tag = Tag.objects.create(name='Python', slug='python')
    
    def test_list_tags_public(self):
        url = reverse('tag-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data), 1)
    
    def test_create_tag_admin(self):
        self.client.force_authenticate(self.admin)
        url = reverse('tag-list')
        response = self.client.post(url, {'name': 'Django', 'slug': 'django'})
        self.assertEqual(response.status_code, 201)
        self.assertEqual(Tag.objects.count(), 2)
    
    def test_create_tag_non_admin_forbidden(self):
        self.client.force_authenticate(self.user)
        url = reverse('tag-list')
        response = self.client.post(url, {'name': 'Test', 'slug': 'test'})
        self.assertEqual(response.status_code, 403)
    
    def test_update_tag(self):
        self.client.force_authenticate(self.admin)
        url = reverse('tag-detail', kwargs={'pk': self.tag.pk})
        response = self.client.patch(url, {'name': 'Python 3'})
        self.assertEqual(response.status_code, 200)
        self.tag.refresh_from_db()
        self.assertEqual(self.tag.name, 'Python 3')
    
    def test_delete_tag(self):
        self.client.force_authenticate(self.admin)
        url = reverse('tag-detail', kwargs={'pk': self.tag.pk})
        response = self.client.delete(url)
        self.assertEqual(response.status_code, 204)
        self.assertEqual(Tag.objects.count(), 0)
    
    def test_retrieve_tag(self):
        url = reverse('tag-detail', kwargs={'pk': self.tag.pk})
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data['name'], 'Python')
    
    def test_tag_not_found(self):
        url = reverse('tag-detail', kwargs={'pk': 9999})
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)
    
    def test_create_duplicate_slug_fails(self):
        self.client.force_authenticate(self.admin)
        url = reverse('tag-list')
        response = self.client.post(url, {'name': 'Python2', 'slug': 'python'})
        self.assertEqual(response.status_code, 400)
```

---

## สรุป

Django REST Framework เป็น framework ที่ครบครันสำหรับการสร้าง REST API โดย features สำคัญที่ควรเข้าใจ:

1. **Serializers** - แปลงข้อมูลระหว่าง Python objects และ JSON รวมถึง validation
2. **ModelSerializer** - shortcut สำหรับสร้าง serializer จาก model
3. **APIView/GenericView** - class-based views สำหรับ request handling
4. **ViewSets** - รวม CRUD operations ไว้ใน class เดียว
5. **Routers** - auto-generate URL patterns จาก ViewSets
6. **Authentication** - Token, JWT, Session authentication
7. **Permissions** - ควบคุมการเข้าถึง resources
8. **Pagination** - จัดการ large datasets
9. **Filtering** - ค้นหาและกรองข้อมูล
10. **Throttling** - rate limiting

การสร้าง API ที่ดีควรใช้ consistent response format, ตั้งค่า authentication และ permissions ให้เหมาะสม, มี pagination สำหรับ list endpoints, และมี documentation ที่ชัดเจน

---

*หัวข้อถัดไป: Part 67 - Django Authentication & Permissions*
