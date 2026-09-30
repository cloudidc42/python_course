# Part 64: Django - Views, Templates & URLs

## สารบัญ

1. [Function-Based Views (FBV) พื้นฐาน](#function-based-views)
2. [Class-Based Views (CBV)](#class-based-views)
3. [Generic Views](#generic-views)
4. [URL Patterns](#url-patterns)
5. [Named URLs และ URL Namespaces](#named-urls)
6. [Django Templates](#django-templates)
7. [Template Tags](#template-tags)
8. [Template Filters](#template-filters)
9. [Template Inheritance](#template-inheritance)
10. [Context Processors](#context-processors)
11. [Middleware Concepts](#middleware-concepts)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Function-Based Views (FBV)

### แนวคิดพื้นฐาน

**Function-Based Views** คือฟังก์ชัน Python ธรรมดาที่รับ `HttpRequest` object และส่งคืน `HttpResponse` object กลับมา เป็นวิธีที่เข้าใจง่ายที่สุดในการสร้าง view ใน Django

```python
# views.py
from django.http import HttpResponse, HttpRequest

def hello_world(request: HttpRequest) -> HttpResponse:
    return HttpResponse("Hello, World!")
```

### ตัวอย่าง FBV พื้นฐาน

**ตัวอย่างที่ 1: View ที่ส่งคืน HTML ธรรมดา**

```python
# myapp/views.py
from django.http import HttpResponse

def home(request):
    html = """
    <!DOCTYPE html>
    <html>
    <head><title>หน้าแรก</title></head>
    <body>
        <h1>ยินดีต้อนรับสู่เว็บไซต์ของเรา</h1>
        <p>นี่คือหน้าแรก</p>
    </body>
    </html>
    """
    return HttpResponse(html)
```

**ตัวอย่างที่ 2: View ที่ใช้ Template**

```python
# myapp/views.py
from django.shortcuts import render

def home(request):
    context = {
        'title': 'หน้าแรก',
        'message': 'ยินดีต้อนรับสู่เว็บไซต์ของเรา',
    }
    return render(request, 'myapp/home.html', context)
```

**ตัวอย่างที่ 3: View ที่รับ URL Parameters**

```python
# myapp/views.py
from django.shortcuts import render, get_object_or_404
from .models import Article

def article_detail(request, article_id):
    # get_object_or_404 จะ raise Http404 ถ้าไม่พบ object
    article = get_object_or_404(Article, id=article_id)
    context = {
        'article': article,
    }
    return render(request, 'myapp/article_detail.html', context)
```

**ตัวอย่างที่ 4: View ที่จัดการ HTTP Methods**

```python
# myapp/views.py
from django.http import HttpResponse, HttpResponseNotAllowed
from django.shortcuts import render

def contact(request):
    if request.method == 'GET':
        # แสดงฟอร์มติดต่อ
        return render(request, 'myapp/contact.html')
    elif request.method == 'POST':
        # จัดการข้อมูลที่ส่งมา
        name = request.POST.get('name', '')
        email = request.POST.get('email', '')
        message = request.POST.get('message', '')
        
        # บันทึกข้อมูลหรือส่งอีเมล
        # ...
        
        return HttpResponse(f"ขอบคุณ {name} ที่ติดต่อมา")
    else:
        return HttpResponseNotAllowed(['GET', 'POST'])
```

**ตัวอย่างที่ 5: View ที่ส่งคืน JSON Response**

```python
# myapp/views.py
import json
from django.http import JsonResponse

def api_users(request):
    users = [
        {'id': 1, 'name': 'สมชาย', 'email': 'somchai@example.com'},
        {'id': 2, 'name': 'สมหญิง', 'email': 'somying@example.com'},
    ]
    return JsonResponse({'users': users, 'total': len(users)})
```

**ตัวอย่างที่ 6: View ที่ใช้ Decorator**

```python
# myapp/views.py
from django.contrib.auth.decorators import login_required
from django.http import HttpResponse
from django.shortcuts import render

@login_required
def profile(request):
    context = {
        'user': request.user,
    }
    return render(request, 'myapp/profile.html', context)

# หรือใช้ require_http_methods decorator
from django.views.decorators.http import require_http_methods

@require_http_methods(["GET", "POST"])
def my_view(request):
    if request.method == "GET":
        return render(request, 'myapp/form.html')
    # POST handling...
    return HttpResponse("บันทึกสำเร็จ")
```

**ตัวอย่างที่ 7: View ที่ใช้ Redirect**

```python
# myapp/views.py
from django.shortcuts import redirect, render
from django.urls import reverse

def old_page(request):
    # Redirect ไปยัง URL ใหม่
    return redirect('myapp:new_page')

def login_redirect(request):
    if request.user.is_authenticated:
        return redirect(reverse('myapp:dashboard'))
    return render(request, 'myapp/login.html')
```

**ตัวอย่างที่ 8: View ที่แสดง List ของ Objects**

```python
# myapp/views.py
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def post_list(request):
    posts_queryset = Post.objects.all().order_by('-created_at')
    
    # Pagination - แสดง 10 posts ต่อหน้า
    paginator = Paginator(posts_queryset, 10)
    page_number = request.GET.get('page', 1)
    posts = paginator.get_page(page_number)
    
    context = {
        'posts': posts,
        'total_posts': posts_queryset.count(),
    }
    return render(request, 'myapp/post_list.html', context)
```

---

## Class-Based Views (CBV)

### แนวคิดพื้นฐาน

**Class-Based Views** คือ views ที่เขียนในรูปแบบ class แทนที่จะเป็น function ช่วยให้สามารถใช้ inheritance และ mixins เพื่อ reuse โค้ดได้ดีกว่า

```python
# myapp/views.py
from django.views import View
from django.http import HttpResponse

class HelloWorldView(View):
    def get(self, request):
        return HttpResponse("Hello from CBV!")
    
    def post(self, request):
        return HttpResponse("POST received!")
```

### การลงทะเบียน CBV ใน urls.py

```python
# myapp/urls.py
from django.urls import path
from .views import HelloWorldView

urlpatterns = [
    path('hello/', HelloWorldView.as_view(), name='hello'),
]
```

**ตัวอย่างที่ 9: CBV พื้นฐานที่ใช้ Template**

```python
# myapp/views.py
from django.views import View
from django.shortcuts import render, get_object_or_404
from .models import Article

class ArticleListView(View):
    template_name = 'myapp/article_list.html'
    
    def get(self, request):
        articles = Article.objects.all().order_by('-created_at')
        return render(request, self.template_name, {'articles': articles})

class ArticleDetailView(View):
    template_name = 'myapp/article_detail.html'
    
    def get(self, request, pk):
        article = get_object_or_404(Article, pk=pk)
        return render(request, self.template_name, {'article': article})
```

**ตัวอย่างที่ 10: TemplateView - View ที่แสดง Template อย่างเดียว**

```python
# myapp/views.py
from django.views.generic import TemplateView

class AboutView(TemplateView):
    template_name = 'myapp/about.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'เกี่ยวกับเรา'
        context['team_members'] = ['สมชาย', 'สมหญิง', 'สมศรี']
        return context
```

**ตัวอย่างที่ 11: RedirectView**

```python
# myapp/views.py
from django.views.generic import RedirectView

class OldPageRedirectView(RedirectView):
    # กำหนด URL ที่จะ redirect ไป
    url = '/new-page/'
    permanent = False  # True = 301 redirect, False = 302 redirect
    
# หรือ redirect ไปที่ named URL
class HomeRedirectView(RedirectView):
    pattern_name = 'myapp:home'
```

---

## Generic Views

### ListView

**ตัวอย่างที่ 12: ListView พื้นฐาน**

```python
# myapp/views.py
from django.views.generic import ListView
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'myapp/post_list.html'
    context_object_name = 'posts'      # ชื่อตัวแปรใน template (default: object_list)
    paginate_by = 10                   # จำนวน items ต่อหน้า
    ordering = ['-created_at']         # เรียงตาม field
    
    def get_queryset(self):
        # Override เพื่อ filter ข้อมูล
        return Post.objects.filter(is_published=True).order_by('-created_at')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['total_posts'] = Post.objects.count()
        return context
```

**Template สำหรับ ListView (post_list.html)**

```html
{% extends 'base.html' %}

{% block content %}
<h1>บทความทั้งหมด ({{ total_posts }} บทความ)</h1>

<div class="post-list">
    {% for post in posts %}
    <article class="post-card">
        <h2><a href="{% url 'myapp:post_detail' post.pk %}">{{ post.title }}</a></h2>
        <p class="date">{{ post.created_at|date:"d M Y" }}</p>
        <p>{{ post.content|truncatewords:50 }}</p>
    </article>
    {% empty %}
    <p>ยังไม่มีบทความ</p>
    {% endfor %}
</div>

<!-- Pagination -->
{% if is_paginated %}
<nav class="pagination">
    {% if page_obj.has_previous %}
    <a href="?page={{ page_obj.previous_page_number }}">ก่อนหน้า</a>
    {% endif %}
    
    <span>หน้า {{ page_obj.number }} จาก {{ page_obj.paginator.num_pages }}</span>
    
    {% if page_obj.has_next %}
    <a href="?page={{ page_obj.next_page_number }}">ถัดไป</a>
    {% endif %}
</nav>
{% endif %}
{% endblock %}
```

### DetailView

**ตัวอย่างที่ 13: DetailView พื้นฐาน**

```python
# myapp/views.py
from django.views.generic import DetailView
from .models import Post

class PostDetailView(DetailView):
    model = Post
    template_name = 'myapp/post_detail.html'
    context_object_name = 'post'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        # เพิ่ม related posts
        context['related_posts'] = Post.objects.filter(
            category=self.object.category
        ).exclude(pk=self.object.pk)[:3]
        return context
```

### CreateView

**ตัวอย่างที่ 14: CreateView พื้นฐาน**

```python
# myapp/views.py
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    template_name = 'myapp/post_form.html'
    fields = ['title', 'content', 'category']
    success_url = reverse_lazy('myapp:post_list')
    
    def form_valid(self, form):
        # กำหนด author เป็น user ที่ login อยู่
        form.instance.author = self.request.user
        return super().form_valid(form)
```

**Template สำหรับ CreateView (post_form.html)**

```html
{% extends 'base.html' %}

{% block content %}
<h1>สร้างบทความใหม่</h1>

<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">บันทึก</button>
    <a href="{% url 'myapp:post_list' %}">ยกเลิก</a>
</form>
{% endblock %}
```

### UpdateView

**ตัวอย่างที่ 15: UpdateView พื้นฐาน**

```python
# myapp/views.py
from django.views.generic.edit import UpdateView
from django.urls import reverse_lazy
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post

class PostUpdateView(LoginRequiredMixin, UpdateView):
    model = Post
    template_name = 'myapp/post_form.html'
    fields = ['title', 'content', 'category']
    
    def get_success_url(self):
        # Redirect ไปยัง detail view ของ post ที่เพิ่งแก้ไข
        return reverse_lazy('myapp:post_detail', kwargs={'pk': self.object.pk})
    
    def get_queryset(self):
        # อนุญาตให้แก้ไขได้เฉพาะ posts ของตัวเอง
        return Post.objects.filter(author=self.request.user)
```

### DeleteView

**ตัวอย่างที่ 16: DeleteView พื้นฐาน**

```python
# myapp/views.py
from django.views.generic.edit import DeleteView
from django.urls import reverse_lazy
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post

class PostDeleteView(LoginRequiredMixin, DeleteView):
    model = Post
    template_name = 'myapp/post_confirm_delete.html'
    success_url = reverse_lazy('myapp:post_list')
    
    def get_queryset(self):
        # อนุญาตให้ลบได้เฉพาะ posts ของตัวเอง
        return Post.objects.filter(author=self.request.user)
```

**Template สำหรับ DeleteView (post_confirm_delete.html)**

```html
{% extends 'base.html' %}

{% block content %}
<h1>ยืนยันการลบ</h1>
<p>คุณต้องการลบบทความ "<strong>{{ post.title }}</strong>" ใช่หรือไม่?</p>

<form method="post">
    {% csrf_token %}
    <button type="submit" class="btn-danger">ลบ</button>
    <a href="{% url 'myapp:post_detail' post.pk %}">ยกเลิก</a>
</form>
{% endblock %}
```

---

## URL Patterns

### การกำหนด URL Patterns พื้นฐาน

```python
# myapp/urls.py
from django.urls import path, re_path
from . import views

urlpatterns = [
    # path() พื้นฐาน
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    
    # path() ที่รับ integer parameter
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    
    # path() ที่รับ string parameter
    path('category/<str:slug>/', views.category_detail, name='category_detail'),
    
    # path() ที่รับ slug parameter
    path('article/<slug:slug>/', views.article_detail, name='article_detail'),
    
    # path() ที่รับ uuid parameter
    path('item/<uuid:item_id>/', views.item_detail, name='item_detail'),
    
    # path() ที่รับ path parameter (รวม slash ด้วย)
    path('files/<path:file_path>/', views.file_detail, name='file_detail'),
]
```

**ตัวอย่างที่ 17: URL Converters ประเภทต่างๆ**

```python
# myapp/urls.py
from django.urls import path, re_path
from . import views

urlpatterns = [
    # int: รับตัวเลขจำนวนเต็มบวก เช่น 1, 42, 1000
    path('post/<int:pk>/', views.post_detail),
    
    # str: รับ string ที่ไม่มี / เช่น "hello", "world-post"
    path('page/<str:name>/', views.page_detail),
    
    # slug: รับ slug string เช่น "my-first-post"
    path('article/<slug:slug>/', views.article_detail),
    
    # uuid: รับ UUID เช่น "123e4567-e89b-12d3-a456-426614174000"
    path('item/<uuid:item_uuid>/', views.item_detail),
    
    # path: รับ path ที่มี / ด้วย เช่น "images/2024/photo.jpg"
    path('media/<path:file_path>/', views.media_detail),
]
```

### re_path - Regular Expression URLs

**ตัวอย่างที่ 18: re_path สำหรับ URLs ที่ซับซ้อน**

```python
# myapp/urls.py
from django.urls import path, re_path
from . import views

urlpatterns = [
    # re_path() ใช้ regular expression
    # รับปีที่มี 4 หลัก
    re_path(r'^archive/(?P<year>[0-9]{4})/$', views.archive_year),
    
    # รับปีและเดือน
    re_path(r'^archive/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.archive_month),
    
    # รับไฟล์ที่นามสกุล .pdf
    re_path(r'^download/(?P<filename>[\w\-]+\.pdf)$', views.download_pdf),
    
    # รับ username ที่มีตัวอักษรและตัวเลขเท่านั้น
    re_path(r'^user/(?P<username>[a-zA-Z0-9]+)/$', views.user_profile),
]
```

### include() - การแยก URL Configuration

**ตัวอย่างที่ 19: การใช้ include()**

```python
# myproject/urls.py (main urls.py)
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    
    # include URLs จาก myapp
    path('blog/', include('myapp.urls')),
    
    # include URLs จาก shop app
    path('shop/', include('shop.urls')),
    
    # include URLs พร้อม namespace
    path('api/', include(('api.urls', 'api'), namespace='api')),
    
    # include โดยตรงจาก list
    path('extra/', include([
        path('help/', views.help_view, name='help'),
        path('contact/', views.contact_view, name='contact'),
    ])),
]
```

---

## Named URLs และ URL Namespaces

### Named URLs

**ตัวอย่างที่ 20: Named URLs**

```python
# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.PostListView.as_view(), name='post_list'),
    path('<int:pk>/', views.PostDetailView.as_view(), name='post_detail'),
    path('create/', views.PostCreateView.as_view(), name='post_create'),
    path('<int:pk>/update/', views.PostUpdateView.as_view(), name='post_update'),
    path('<int:pk>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
]
```

**การใช้ Named URLs ใน Template**

```html
<!-- การใช้ {% url %} tag ใน template -->
<a href="{% url 'post_list' %}">รายการบทความ</a>
<a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a>
<a href="{% url 'post_create' %}">สร้างบทความใหม่</a>
<a href="{% url 'post_update' pk=post.pk %}">แก้ไข</a>
<a href="{% url 'post_delete' pk=post.pk %}">ลบ</a>
```

**การใช้ Named URLs ใน Python Code**

```python
# views.py
from django.urls import reverse
from django.shortcuts import redirect

def some_view(request):
    # ใช้ reverse() เพื่อสร้าง URL จาก name
    url = reverse('post_detail', kwargs={'pk': 1})
    return redirect(url)
    
    # หรือใช้ reverse_lazy() สำหรับ class attributes
    # (ใช้เมื่อ URL configuration ยังไม่โหลด)
```

### URL Namespaces

**ตัวอย่างที่ 21: URL Namespaces**

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'  # กำหนด namespace

urlpatterns = [
    path('', views.PostListView.as_view(), name='post_list'),
    path('<int:pk>/', views.PostDetailView.as_view(), name='post_detail'),
    path('create/', views.PostCreateView.as_view(), name='post_create'),
]
```

```python
# myproject/urls.py
from django.urls import path, include

urlpatterns = [
    path('blog/', include('blog.urls', namespace='blog')),
    # หรือ (app_name กำหนดใน blog/urls.py แล้ว)
    path('blog/', include('blog.urls')),
]
```

**การใช้ Named URLs กับ Namespace**

```html
<!-- ใน template ใช้ namespace:name -->
<a href="{% url 'blog:post_list' %}">บทความทั้งหมด</a>
<a href="{% url 'blog:post_detail' pk=post.pk %}">{{ post.title }}</a>
```

```python
# ใน Python code
from django.urls import reverse

url = reverse('blog:post_detail', kwargs={'pk': 1})
```

**ตัวอย่างที่ 22: Nested Namespaces**

```python
# api/v1/urls.py
from django.urls import path
from . import views

app_name = 'v1'

urlpatterns = [
    path('users/', views.user_list, name='user_list'),
    path('posts/', views.post_list, name='post_list'),
]

# api/urls.py
from django.urls import path, include

app_name = 'api'

urlpatterns = [
    path('v1/', include('api.v1.urls', namespace='v1')),
]

# myproject/urls.py
urlpatterns = [
    path('api/', include('api.urls', namespace='api')),
]

# การใช้งาน: api:v1:user_list
url = reverse('api:v1:user_list')
```

---

## Django Templates

### โครงสร้าง Template Directory

```
myproject/
├── myapp/
│   ├── templates/
│   │   └── myapp/
│   │       ├── base.html
│   │       ├── home.html
│   │       ├── post_list.html
│   │       └── post_detail.html
│   └── views.py
└── templates/          # Global templates
    ├── base.html
    └── 404.html
```

### การตั้งค่า Templates ใน settings.py

```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # Global template directory
        'APP_DIRS': True,  # ค้นหา templates ใน app ด้วย
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

### Template Variables

**ตัวอย่างที่ 23: การใช้ Template Variables**

```python
# views.py
from django.shortcuts import render

def profile(request):
    context = {
        'user': request.user,
        'name': 'สมชาย ใจดี',
        'age': 25,
        'is_admin': True,
        'hobbies': ['อ่านหนังสือ', 'เล่นดนตรี', 'ดูหนัง'],
        'address': {
            'city': 'กรุงเทพฯ',
            'country': 'ไทย',
        },
    }
    return render(request, 'myapp/profile.html', context)
```

```html
<!-- myapp/profile.html -->
<h1>โปรไฟล์ของ {{ name }}</h1>
<p>อายุ: {{ age }} ปี</p>

<!-- การเข้าถึง attribute ของ object -->
<p>Email: {{ user.email }}</p>

<!-- การเข้าถึง key ของ dictionary -->
<p>เมือง: {{ address.city }}, {{ address.country }}</p>

<!-- การเข้าถึง index ของ list -->
<p>งานอดิเรกแรก: {{ hobbies.0 }}</p>

<!-- Boolean value -->
{% if is_admin %}
<p>คุณเป็น Admin</p>
{% endif %}
```

---

## Template Tags

### if Tag

**ตัวอย่างที่ 24: if/elif/else Tags**

```html
{% if user.is_authenticated %}
    <p>ยินดีต้อนรับ, {{ user.username }}!</p>
    <a href="{% url 'logout' %}">ออกจากระบบ</a>
{% elif user.is_anonymous %}
    <p>คุณยังไม่ได้เข้าสู่ระบบ</p>
    <a href="{% url 'login' %}">เข้าสู่ระบบ</a>
{% else %}
    <p>สถานะไม่ทราบ</p>
{% endif %}

<!-- การใช้ Comparison operators -->
{% if age >= 18 %}
    <p>ผู้ใหญ่</p>
{% endif %}

<!-- การใช้ and, or, not, in, not in, is, is not -->
{% if user.is_authenticated and user.is_staff %}
    <p>Admin Panel</p>
{% endif %}

{% if name not in banned_users %}
    <p>ผู้ใช้ปกติ</p>
{% endif %}
```

### for Tag

**ตัวอย่างที่ 25: for/empty Tags**

```html
<!-- for loop พื้นฐาน -->
<ul>
{% for post in posts %}
    <li>{{ post.title }}</li>
{% empty %}
    <li>ยังไม่มีบทความ</li>
{% endfor %}
</ul>

<!-- Loop variables ที่ Django ให้มา -->
{% for item in items %}
    <p>
        ลำดับที่: {{ forloop.counter }}      {# 1, 2, 3, ... #}
        ลำดับ (0-based): {{ forloop.counter0 }}  {# 0, 1, 2, ... #}
        ย้อนกลับ: {{ forloop.revcounter }}   {# n, n-1, ..., 1 #}
        แรก: {{ forloop.first }}             {# True ถ้าเป็นตัวแรก #}
        สุดท้าย: {{ forloop.last }}          {# True ถ้าเป็นตัวสุดท้าย #}
    </p>
{% endfor %}

<!-- Nested loops -->
{% for category in categories %}
    <h2>{{ category.name }}</h2>
    <ul>
    {% for post in category.posts.all %}
        <li>{{ post.title }}</li>
    {% endfor %}
    </ul>
{% endfor %}

<!-- Loop กับ unpack -->
{% for key, value in dictionary.items %}
    <p>{{ key }}: {{ value }}</p>
{% endfor %}
```

### block และ extends Tags

**ตัวอย่างที่ 26: block/extends Tags**

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}เว็บไซต์ของเรา{% endblock %}</title>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        <nav>{% block nav %}{% include 'partials/nav.html' %}{% endblock %}</nav>
    </header>
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        {% block footer %}
        <p>&copy; 2024 เว็บไซต์ของเรา</p>
        {% endblock %}
    </footer>
    
    {% block extra_js %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/myapp/home.html -->
{% extends 'base.html' %}

{% block title %}หน้าแรก - เว็บไซต์ของเรา{% endblock %}

{% block content %}
<h1>ยินดีต้อนรับ!</h1>
<p>นี่คือหน้าแรกของเว็บไซต์</p>

<!-- ใช้ block.super เพื่อเก็บเนื้อหาเดิมของ block -->
{% block footer %}
{{ block.super }}
<p>ติดต่อเรา: contact@example.com</p>
{% endblock %}
{% endblock %}
```

### include Tag

**ตัวอย่างที่ 27: include Tag**

```html
<!-- templates/partials/nav.html -->
<ul class="nav">
    <li><a href="{% url 'home' %}">หน้าแรก</a></li>
    <li><a href="{% url 'about' %}">เกี่ยวกับเรา</a></li>
    {% if user.is_authenticated %}
    <li><a href="{% url 'profile' %}">โปรไฟล์</a></li>
    <li><a href="{% url 'logout' %}">ออกจากระบบ</a></li>
    {% else %}
    <li><a href="{% url 'login' %}">เข้าสู่ระบบ</a></li>
    {% endif %}
</ul>
```

```html
<!-- การ include template พร้อมส่ง context เพิ่มเติม -->
{% include 'partials/nav.html' %}
{% include 'partials/post_card.html' with post=featured_post %}
{% include 'partials/user_card.html' with user=author only %}
```

### url Tag

**ตัวอย่างที่ 28: url Tag**

```html
<!-- URL พื้นฐาน -->
<a href="{% url 'home' %}">หน้าแรก</a>

<!-- URL ที่มี positional argument -->
<a href="{% url 'post_detail' post.pk %}">{{ post.title }}</a>

<!-- URL ที่มี keyword argument -->
<a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a>

<!-- URL ที่มี namespace -->
<a href="{% url 'blog:post_detail' pk=post.pk %}">{{ post.title }}</a>

<!-- เก็บ URL ในตัวแปร -->
{% url 'post_detail' pk=post.pk as post_url %}
<a href="{{ post_url }}">{{ post.title }}</a>
```

### static Tag

**ตัวอย่างที่ 29: static Tag**

```python
# settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
```

```html
<!-- ต้อง load static ก่อนใช้งาน -->
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <!-- CSS -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    <link rel="stylesheet" href="{% static 'vendor/bootstrap/css/bootstrap.min.css' %}">
</head>
<body>
    <!-- Images -->
    <img src="{% static 'images/logo.png' %}" alt="Logo">
    
    <!-- JavaScript -->
    <script src="{% static 'js/main.js' %}"></script>
    <script src="{% static 'vendor/jquery/jquery.min.js' %}"></script>
</body>
</html>
```

---

## Template Filters

### Built-in Filters

**ตัวอย่างที่ 30: Template Filters ที่ใช้บ่อย**

```html
<!-- date filter: จัดรูปแบบวันที่ -->
{{ post.created_at|date:"d/m/Y" }}           {# 25/12/2024 #}
{{ post.created_at|date:"d M Y H:i" }}       {# 25 Dec 2024 14:30 #}
{{ post.created_at|date:"l, j F Y" }}        {# Wednesday, 25 December 2024 #}
{{ post.created_at|date:"SHORT_DATE_FORMAT" }} {# รูปแบบย่อตาม locale #}

<!-- length filter: นับความยาว -->
{{ post.title|length }}        {# จำนวนตัวอักษร #}
{{ posts|length }}             {# จำนวน items ใน list #}

<!-- upper/lower filter: ตัวอักษรใหญ่/เล็ก -->
{{ post.title|upper }}
{{ post.title|lower }}
{{ post.title|title }}         {# Title Case #}
{{ post.title|capfirst }}      {# ตัวแรกใหญ่ #}

<!-- truncatewords filter: ตัดคำ -->
{{ post.content|truncatewords:50 }}          {# ตัดที่ 50 คำ #}
{{ post.content|truncatechars:200 }}         {# ตัดที่ 200 ตัวอักษร #}
{{ post.content|truncatewords_html:50 }}     {# ตัดคำแต่รักษา HTML tags #}

<!-- linebreaks filter: แปลง newlines เป็น HTML -->
{{ post.content|linebreaks }}     {# แปลง \n เป็น <p> #}
{{ post.content|linebreaksbr }}   {# แปลง \n เป็น <br> #}

<!-- safe filter: บอกว่า HTML content ปลอดภัย -->
{{ post.content|safe }}           {# ไม่ escape HTML #}

<!-- escape filter: escape HTML characters -->
{{ user_input|escape }}

<!-- default filter: ค่า default ถ้าเป็น None/empty -->
{{ user.bio|default:"ยังไม่มีข้อมูล" }}
{{ value|default_if_none:"ไม่มีค่า" }}

<!-- add filter: บวกตัวเลข -->
{{ value|add:10 }}
{{ value|add:"-5" }}

<!-- divisibleby filter: หารลงตัว -->
{% if forloop.counter|divisibleby:2 %}
<p>บรรทัดคู่</p>
{% endif %}

<!-- floatformat filter: จัดรูปแบบ float -->
{{ price|floatformat:2 }}        {# 99.90 #}
{{ percentage|floatformat:"2g" }} {# 99.9 (ไม่มี trailing zero) #}

<!-- slugify filter: แปลงเป็น slug -->
{{ title|slugify }}   {# "Hello World" -> "hello-world" #}

<!-- striptags filter: ลบ HTML tags #}
{{ content|striptags }}

<!-- wordcount filter: นับคำ -->
{{ content|wordcount }}

<!-- yesno filter: แสดง yes/no/maybe -->
{{ is_published|yesno:"เผยแพร่,ฉบับร่าง,รอดำเนินการ" }}

<!-- join filter: รวม list เป็น string -->
{{ tags|join:", " }}

<!-- first/last filter -->
{{ items|first }}
{{ items|last }}

<!-- urlencode filter: encode URL -->
{{ query|urlencode }}

<!-- filesizeformat filter: แสดงขนาดไฟล์ -->
{{ file_size|filesizeformat }}  {# 25.4 KB #}

<!-- timesince/timeuntil filter -->
{{ post.created_at|timesince }}   {# "3 days, 2 hours" #}
{{ event.start_date|timeuntil }}  {# "2 weeks, 3 days" #}
```

---

## Template Inheritance

### Base Template Pattern

**ตัวอย่างที่ 31: โครงสร้าง Template Inheritance ที่สมบูรณ์**

```html
<!-- templates/base.html - Template หลัก -->
{% load static %}
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="{% block meta_description %}เว็บไซต์ดีๆ{% endblock %}">
    
    <title>{% block title %}เว็บไซต์ของเรา{% endblock %} | My Site</title>
    
    <!-- Base CSS -->
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    <link rel="stylesheet" href="{% static 'css/base.css' %}">
    
    <!-- Extra CSS สำหรับแต่ละหน้า -->
    {% block extra_css %}{% endblock %}
</head>
<body class="{% block body_class %}{% endblock %}">
    
    <!-- Navigation -->
    <nav class="navbar">
        {% include 'partials/navbar.html' %}
    </nav>
    
    <!-- Messages -->
    {% if messages %}
    <div class="messages">
        {% for message in messages %}
        <div class="alert alert-{{ message.tags }}">
            {{ message }}
        </div>
        {% endfor %}
    </div>
    {% endif %}
    
    <!-- Breadcrumbs -->
    {% block breadcrumbs %}{% endblock %}
    
    <!-- Main Content -->
    <main class="container">
        {% block content %}{% endblock %}
    </main>
    
    <!-- Sidebar (optional) -->
    {% block sidebar %}{% endblock %}
    
    <!-- Footer -->
    <footer>
        {% include 'partials/footer.html' %}
    </footer>
    
    <!-- Base JavaScript -->
    <script src="{% static 'js/jquery.min.js' %}"></script>
    <script src="{% static 'js/bootstrap.min.js' %}"></script>
    <script src="{% static 'js/base.js' %}"></script>
    
    <!-- Extra JavaScript -->
    {% block extra_js %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/blog/base.html - Blog Base Template -->
{% extends 'base.html' %}

{% block body_class %}blog-layout{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-8">
        {% block blog_content %}{% endblock %}
    </div>
    <div class="col-md-4">
        {% block sidebar %}
        {% include 'blog/partials/sidebar.html' %}
        {% endblock %}
    </div>
</div>
{% endblock %}
```

```html
<!-- templates/blog/post_detail.html - Specific Page -->
{% extends 'blog/base.html' %}

{% block title %}{{ post.title }}{% endblock %}
{% block meta_description %}{{ post.excerpt }}{% endblock %}

{% block breadcrumbs %}
<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li><a href="{% url 'home' %}">หน้าแรก</a></li>
        <li><a href="{% url 'blog:post_list' %}">บทความ</a></li>
        <li class="active">{{ post.title }}</li>
    </ol>
</nav>
{% endblock %}

{% block blog_content %}
<article>
    <h1>{{ post.title }}</h1>
    <p class="meta">
        โดย {{ post.author.get_full_name }} 
        เมื่อ {{ post.created_at|date:"d M Y" }}
    </p>
    <div class="content">
        {{ post.content|safe }}
    </div>
</article>
{% endblock %}

{% block extra_js %}
{{ block.super }}
<script src="{% static 'js/post-detail.js' %}"></script>
{% endblock %}
```

---

## Context Processors

### แนวคิดของ Context Processors

**Context Processors** คือ function ที่รับ `request` และส่งคืน dictionary ซึ่ง Django จะรวมเข้ากับ context ของทุก template โดยอัตโนมัติ

**ตัวอย่างที่ 32: Custom Context Processors**

```python
# myapp/context_processors.py
from .models import Category, Setting

def site_settings(request):
    """เพิ่มการตั้งค่าเว็บไซต์เข้าไปในทุก template"""
    try:
        settings = Setting.objects.get(is_active=True)
    except Setting.DoesNotExist:
        settings = None
    
    return {
        'site_settings': settings,
        'site_name': 'My Awesome Website',
        'contact_email': 'contact@example.com',
    }

def categories(request):
    """เพิ่มรายการ categories เข้าไปในทุก template"""
    return {
        'categories': Category.objects.all()[:10],
    }

def cart_count(request):
    """เพิ่มจำนวนสินค้าในตะกร้าเข้าไปในทุก template"""
    if request.user.is_authenticated:
        count = request.user.cart.items.count()
    else:
        count = 0
    return {
        'cart_count': count,
    }
```

**การลงทะเบียน Context Processors**

```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [...],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                # Built-in context processors
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                
                # Custom context processors
                'myapp.context_processors.site_settings',
                'myapp.context_processors.categories',
                'myapp.context_processors.cart_count',
            ],
        },
    },
]
```

---

## Middleware Concepts

### แนวคิดของ Middleware

**Middleware** คือ component ที่อยู่ระหว่าง request และ response เปรียบเหมือน "layer" ที่ทุก request/response ต้องผ่าน ใช้สำหรับ:
- Authentication/Authorization
- Logging
- Caching
- Security
- Session management

**ตัวอย่างที่ 33: Custom Middleware**

```python
# myapp/middleware.py

class SimpleLoggingMiddleware:
    """Middleware สำหรับ log requests"""
    
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.
    
    def __call__(self, request):
        # Code ที่รันก่อน view (before request)
        import time
        start_time = time.time()
        
        print(f"Request: {request.method} {request.path}")
        
        # ส่ง request ไปยัง view
        response = self.get_response(request)
        
        # Code ที่รันหลัง view (after response)
        duration = time.time() - start_time
        print(f"Response: {response.status_code} ({duration:.2f}s)")
        
        return response
    
    def process_exception(self, request, exception):
        """จัดการ exception"""
        print(f"Exception: {exception}")
        return None  # None = ใช้ default exception handling


class MaintenanceModeMiddleware:
    """Middleware สำหรับ maintenance mode"""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        from django.conf import settings
        from django.http import HttpResponse
        
        if getattr(settings, 'MAINTENANCE_MODE', False):
            # ยกเว้น admin และ static files
            if not request.path.startswith('/admin/') and \
               not request.path.startswith('/static/'):
                return HttpResponse(
                    "เว็บไซต์กำลังปิดปรับปรุงชั่วคราว กรุณากลับมาใหม่ในภายหลัง",
                    status=503
                )
        
        return self.get_response(request)
```

**การลงทะเบียน Middleware**

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    
    # Custom Middleware
    'myapp.middleware.SimpleLoggingMiddleware',
    'myapp.middleware.MaintenanceModeMiddleware',
]
```

**ตัวอย่างที่ 34: Middleware ที่ใช้ process_view**

```python
# myapp/middleware.py
from django.http import HttpResponseForbidden

class IPWhitelistMiddleware:
    """Middleware สำหรับกำหนด IP Whitelist"""
    
    ALLOWED_IPS = ['127.0.0.1', '192.168.1.0']
    RESTRICTED_PATHS = ['/admin/', '/api/internal/']
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        return self.get_response(request)
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        """รันก่อน view function"""
        client_ip = self.get_client_ip(request)
        
        for restricted_path in self.RESTRICTED_PATHS:
            if request.path.startswith(restricted_path):
                if client_ip not in self.ALLOWED_IPS:
                    return HttpResponseForbidden(
                        f"การเข้าถึงถูกปฏิเสธสำหรับ IP: {client_ip}"
                    )
        
        return None  # None = ดำเนินการต่อ
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            return x_forwarded_for.split(',')[0]
        return request.META.get('REMOTE_ADDR')
```

**ตัวอย่างที่ 35: ตัวอย่างโปรเจกต์สมบูรณ์ - Blog Application**

```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify

class Category(models.Model):
    name = models.CharField(max_length=100, verbose_name="ชื่อหมวดหมู่")
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name = "หมวดหมู่"
        verbose_name_plural = "หมวดหมู่"
    
    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'ฉบับร่าง'),
        ('published', 'เผยแพร่'),
    ]
    
    title = models.CharField(max_length=200, verbose_name="หัวข้อ")
    slug = models.SlugField(unique=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="ผู้เขียน")
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    content = models.TextField(verbose_name="เนื้อหา")
    excerpt = models.TextField(max_length=500, blank=True, verbose_name="ข้อความย่อ")
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    views_count = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['-created_at']
        verbose_name = "บทความ"
        verbose_name_plural = "บทความ"
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)
    
    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('blog:post_detail', kwargs={'slug': self.slug})
```

```python
# blog/views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from django.shortcuts import get_object_or_404
from django.db.models import Q
from .models import Post, Category

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        queryset = Post.objects.filter(status='published')
        
        # Search functionality
        search_query = self.request.GET.get('q')
        if search_query:
            queryset = queryset.filter(
                Q(title__icontains=search_query) |
                Q(content__icontains=search_query)
            )
        
        # Filter by category
        category_slug = self.kwargs.get('category_slug')
        if category_slug:
            queryset = queryset.filter(category__slug=category_slug)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['search_query'] = self.request.GET.get('q', '')
        context['categories'] = Category.objects.all()
        
        category_slug = self.kwargs.get('category_slug')
        if category_slug:
            context['current_category'] = get_object_or_404(Category, slug=category_slug)
        
        return context

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    slug_field = 'slug'
    slug_url_kwarg = 'slug'
    
    def get_object(self):
        obj = super().get_object()
        # เพิ่ม view count
        Post.objects.filter(pk=obj.pk).update(views_count=obj.views_count + 1)
        return obj
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['related_posts'] = Post.objects.filter(
            category=self.object.category,
            status='published'
        ).exclude(pk=self.object.pk)[:3]
        return context
```

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.PostListView.as_view(), name='post_list'),
    path('category/<slug:category_slug>/', views.PostListView.as_view(), name='category_posts'),
    path('<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    path('create/', views.PostCreateView.as_view(), name='post_create'),
    path('<slug:slug>/edit/', views.PostUpdateView.as_view(), name='post_update'),
    path('<slug:slug>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
]
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Function-Based View พื้นฐาน

**โจทย์:** สร้าง view ที่แสดงข้อมูลผู้ใช้จาก GET parameters และ validate ข้อมูล

```python
# โจทย์: สร้าง view ที่รับ name และ age จาก GET parameters
# ถ้า age < 0 หรือ > 150 ให้แสดง error
# ถ้าไม่มี name ให้ใช้ "ผู้เยี่ยมชม"

# URL: /user-info/?name=สมชาย&age=25
```

**เฉลย:**

```python
# myapp/views.py
from django.http import HttpResponse, HttpResponseBadRequest
from django.shortcuts import render

def user_info(request):
    name = request.GET.get('name', 'ผู้เยี่ยมชม')
    age_str = request.GET.get('age', '')
    
    context = {'name': name}
    
    if age_str:
        try:
            age = int(age_str)
            if age < 0 or age > 150:
                return HttpResponseBadRequest("อายุไม่ถูกต้อง (ต้องอยู่ระหว่าง 0-150)")
            context['age'] = age
        except ValueError:
            return HttpResponseBadRequest("กรุณาระบุอายุเป็นตัวเลข")
    
    return render(request, 'myapp/user_info.html', context)

# myapp/urls.py
urlpatterns = [
    path('user-info/', views.user_info, name='user_info'),
]
```

```html
<!-- templates/myapp/user_info.html -->
{% extends 'base.html' %}
{% block content %}
<h1>ข้อมูลผู้ใช้</h1>
<p>ชื่อ: {{ name }}</p>
{% if age %}
<p>อายุ: {{ age }} ปี</p>
{% else %}
<p>ไม่ได้ระบุอายุ</p>
{% endif %}
{% endblock %}
```

---

### แบบฝึกหัดที่ 2: Class-Based View

**โจทย์:** สร้าง CBV สำหรับแสดง dashboard ที่แสดง statistics ต่างๆ

```python
# โจทย์: สร้าง DashboardView (LoginRequired) ที่แสดง:
# - จำนวน posts ทั้งหมด
# - จำนวน posts ของ user ที่ login
# - posts ที่สร้างล่าสุด 5 รายการ
```

**เฉลย:**

```python
# myapp/views.py
from django.views.generic import TemplateView
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post

class DashboardView(LoginRequiredMixin, TemplateView):
    template_name = 'myapp/dashboard.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        user = self.request.user
        
        context['total_posts'] = Post.objects.count()
        context['my_posts_count'] = Post.objects.filter(author=user).count()
        context['recent_posts'] = Post.objects.filter(
            author=user
        ).order_by('-created_at')[:5]
        
        return context
```

```html
<!-- templates/myapp/dashboard.html -->
{% extends 'base.html' %}
{% block content %}
<h1>Dashboard ของ {{ request.user.username }}</h1>

<div class="stats">
    <div class="stat-card">
        <h2>{{ total_posts }}</h2>
        <p>บทความทั้งหมดในระบบ</p>
    </div>
    <div class="stat-card">
        <h2>{{ my_posts_count }}</h2>
        <p>บทความของฉัน</p>
    </div>
</div>

<h2>บทความล่าสุดของฉัน</h2>
<ul>
{% for post in recent_posts %}
    <li>
        <a href="{% url 'blog:post_detail' post.pk %}">{{ post.title }}</a>
        <span>{{ post.created_at|date:"d M Y" }}</span>
    </li>
{% empty %}
    <li>ยังไม่มีบทความ</li>
{% endfor %}
</ul>
{% endblock %}
```

---

### แบบฝึกหัดที่ 3: URL Patterns

**โจทย์:** ออกแบบ URL patterns สำหรับ e-commerce website

```
/                           - หน้าแรก
/products/                  - รายการสินค้าทั้งหมด
/products/category/<slug>/  - สินค้าตามหมวดหมู่
/products/<int:pk>/         - รายละเอียดสินค้า
/cart/                      - ตะกร้าสินค้า
/checkout/                  - ชำระเงิน
/orders/                    - ประวัติการสั่งซื้อ
/orders/<int:pk>/           - รายละเอียดคำสั่งซื้อ
/api/products/              - API สำหรับสินค้า
/api/cart/                  - API สำหรับตะกร้า
```

**เฉลย:**

```python
# shop/urls.py
from django.urls import path, include
from . import views

app_name = 'shop'

product_patterns = [
    path('', views.ProductListView.as_view(), name='product_list'),
    path('category/<slug:category_slug>/', views.ProductListView.as_view(), name='category_products'),
    path('<int:pk>/', views.ProductDetailView.as_view(), name='product_detail'),
]

order_patterns = [
    path('', views.OrderListView.as_view(), name='order_list'),
    path('<int:pk>/', views.OrderDetailView.as_view(), name='order_detail'),
]

api_patterns = [
    path('products/', views.ProductAPIView.as_view(), name='api_products'),
    path('cart/', views.CartAPIView.as_view(), name='api_cart'),
]

urlpatterns = [
    path('', views.HomeView.as_view(), name='home'),
    path('products/', include(product_patterns)),
    path('cart/', views.CartView.as_view(), name='cart'),
    path('checkout/', views.CheckoutView.as_view(), name='checkout'),
    path('orders/', include(order_patterns)),
    path('api/', include((api_patterns, 'api'))),
]
```

---

### แบบฝึกหัดที่ 4: ListView กับ Search และ Filter

**โจทย์:** สร้าง ProductListView ที่รองรับการค้นหาและ filter ตามราคา

**เฉลย:**

```python
# shop/views.py
from django.views.generic import ListView
from django.db.models import Q
from .models import Product

class ProductListView(ListView):
    model = Product
    template_name = 'shop/product_list.html'
    context_object_name = 'products'
    paginate_by = 12
    
    def get_queryset(self):
        queryset = Product.objects.filter(is_active=True)
        
        # Search
        search = self.request.GET.get('q')
        if search:
            queryset = queryset.filter(
                Q(name__icontains=search) |
                Q(description__icontains=search)
            )
        
        # Price filter
        min_price = self.request.GET.get('min_price')
        max_price = self.request.GET.get('max_price')
        
        if min_price:
            queryset = queryset.filter(price__gte=min_price)
        if max_price:
            queryset = queryset.filter(price__lte=max_price)
        
        # Sorting
        sort_by = self.request.GET.get('sort', '-created_at')
        valid_sorts = ['price', '-price', 'name', '-name', '-created_at']
        if sort_by in valid_sorts:
            queryset = queryset.order_by(sort_by)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['search_query'] = self.request.GET.get('q', '')
        context['min_price'] = self.request.GET.get('min_price', '')
        context['max_price'] = self.request.GET.get('max_price', '')
        context['sort_by'] = self.request.GET.get('sort', '-created_at')
        return context
```

---

### แบบฝึกหัดที่ 5: Template Inheritance

**โจทย์:** สร้าง template structure สำหรับเว็บไซต์ที่มี 3 section: blog, shop, account

**เฉลย:**

```html
<!-- templates/base.html - Root template -->
{% load static %}
<!DOCTYPE html>
<html lang="th">
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/base.css' %}">
    {% block extra_head %}{% endblock %}
</head>
<body>
    {% include 'partials/header.html' %}
    <main>{% block content %}{% endblock %}</main>
    {% include 'partials/footer.html' %}
    <script src="{% static 'js/base.js' %}"></script>
    {% block extra_scripts %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/blog/base.html - Blog section base -->
{% extends 'base.html' %}
{% block content %}
<div class="blog-layout">
    <section class="blog-main">{% block blog_main %}{% endblock %}</section>
    <aside class="blog-sidebar">{% include 'blog/partials/sidebar.html' %}</aside>
</div>
{% endblock %}
```

```html
<!-- templates/shop/base.html - Shop section base -->
{% extends 'base.html' %}
{% block extra_head %}
<link rel="stylesheet" href="{% static 'css/shop.css' %}">
{% endblock %}
{% block content %}
<div class="shop-layout">
    <aside class="shop-filters">{% include 'shop/partials/filters.html' %}</aside>
    <section class="shop-products">{% block shop_content %}{% endblock %}</section>
</div>
{% endblock %}
```

```html
<!-- templates/account/base.html - Account section base -->
{% extends 'base.html' %}
{% block content %}
<div class="account-layout">
    <nav class="account-nav">{% include 'account/partials/nav.html' %}</nav>
    <section class="account-main">{% block account_content %}{% endblock %}</section>
</div>
{% endblock %}
```

---

### แบบฝึกหัดที่ 6: Context Processors

**โจทย์:** สร้าง context processor สำหรับแสดงประกาศเว็บไซต์ (site notices)

**เฉลย:**

```python
# myapp/models.py
from django.db import models

class SiteNotice(models.Model):
    NOTICE_TYPES = [
        ('info', 'ข้อมูล'),
        ('warning', 'คำเตือน'),
        ('success', 'สำเร็จ'),
        ('error', 'ข้อผิดพลาด'),
    ]
    
    message = models.TextField(verbose_name="ข้อความ")
    notice_type = models.CharField(max_length=10, choices=NOTICE_TYPES, default='info')
    is_active = models.BooleanField(default=True)
    start_date = models.DateTimeField()
    end_date = models.DateTimeField(null=True, blank=True)
    
    def __str__(self):
        return self.message[:50]
```

```python
# myapp/context_processors.py
from django.utils import timezone
from .models import SiteNotice

def site_notices(request):
    """เพิ่มประกาศเว็บไซต์ที่ active เข้าไปในทุก template"""
    now = timezone.now()
    notices = SiteNotice.objects.filter(
        is_active=True,
        start_date__lte=now
    ).filter(
        models.Q(end_date__isnull=True) | models.Q(end_date__gte=now)
    )
    return {'site_notices': notices}
```

```html
<!-- templates/partials/notices.html -->
{% for notice in site_notices %}
<div class="notice notice-{{ notice.notice_type }}">
    {{ notice.message }}
</div>
{% endfor %}
```

---

### แบบฝึกหัดที่ 7: Custom Middleware

**โจทย์:** สร้าง middleware สำหรับบันทึก performance metrics

**เฉลย:**

```python
# myapp/middleware.py
import time
import logging

logger = logging.getLogger('performance')

class PerformanceMiddleware:
    """บันทึกเวลาที่ใช้ในการประมวลผลแต่ละ request"""
    
    SLOW_REQUEST_THRESHOLD = 1.0  # วินาที
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        start_time = time.time()
        
        response = self.get_response(request)
        
        duration = time.time() - start_time
        
        # เพิ่ม header แสดงเวลาที่ใช้
        response['X-Response-Time'] = f"{duration:.3f}s"
        
        # Log slow requests
        if duration > self.SLOW_REQUEST_THRESHOLD:
            logger.warning(
                f"Slow request: {request.method} {request.path} "
                f"took {duration:.3f}s"
            )
        else:
            logger.debug(
                f"{request.method} {request.path} "
                f"completed in {duration:.3f}s"
            )
        
        return response
```

---

### แบบฝึกหัดที่ 8: Complete Mini-Project

**โจทย์:** สร้าง mini-project สำหรับระบบ FAQ (Frequently Asked Questions)

**เฉลย:**

```python
# faq/models.py
from django.db import models

class FAQCategory(models.Model):
    name = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    order = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['order', 'name']
    
    def __str__(self):
        return self.name

class FAQ(models.Model):
    category = models.ForeignKey(FAQCategory, on_delete=models.CASCADE, related_name='faqs')
    question = models.CharField(max_length=500)
    answer = models.TextField()
    is_published = models.BooleanField(default=True)
    order = models.IntegerField(default=0)
    views_count = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['order', 'question']
    
    def __str__(self):
        return self.question

# faq/views.py
from django.views.generic import ListView, DetailView
from django.db.models import Q
from .models import FAQ, FAQCategory

class FAQListView(ListView):
    model = FAQCategory
    template_name = 'faq/list.html'
    context_object_name = 'categories'
    
    def get_queryset(self):
        return FAQCategory.objects.prefetch_related(
            'faqs'
        ).filter(faqs__is_published=True).distinct()
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        search = self.request.GET.get('q')
        
        if search:
            context['search_results'] = FAQ.objects.filter(
                is_published=True
            ).filter(
                Q(question__icontains=search) |
                Q(answer__icontains=search)
            )
            context['search_query'] = search
        
        return context

# faq/urls.py
from django.urls import path
from . import views

app_name = 'faq'

urlpatterns = [
    path('', views.FAQListView.as_view(), name='list'),
]
```

```html
<!-- templates/faq/list.html -->
{% extends 'base.html' %}

{% block title %}คำถามที่พบบ่อย{% endblock %}

{% block content %}
<h1>คำถามที่พบบ่อย (FAQ)</h1>

<!-- Search Form -->
<form method="get" class="search-form">
    <input type="text" name="q" value="{{ search_query|default:'' }}" 
           placeholder="ค้นหาคำถาม...">
    <button type="submit">ค้นหา</button>
</form>

<!-- Search Results -->
{% if search_query %}
    <h2>ผลการค้นหา "{{ search_query }}"</h2>
    {% if search_results %}
        {% for faq in search_results %}
        <div class="faq-item">
            <h3>{{ faq.question }}</h3>
            <p>{{ faq.answer|linebreaks }}</p>
        </div>
        {% endfor %}
    {% else %}
        <p>ไม่พบคำถามที่ตรงกับ "{{ search_query }}"</p>
    {% endif %}
{% else %}
    <!-- FAQ by Category -->
    {% for category in categories %}
    <section class="faq-category">
        <h2>{{ category.name }}</h2>
        <div class="accordion">
            {% for faq in category.faqs.all %}
            {% if faq.is_published %}
            <div class="faq-item">
                <h3 class="faq-question">{{ faq.question }}</h3>
                <div class="faq-answer">
                    {{ faq.answer|linebreaks }}
                </div>
            </div>
            {% endif %}
            {% endfor %}
        </div>
    </section>
    {% empty %}
    <p>ยังไม่มีคำถาม</p>
    {% endfor %}
{% endif %}
{% endblock %}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Function-Based Views (FBV)** - วิธีสร้าง view แบบง่ายที่สุด เหมาะกับ logic ที่ไม่ซับซ้อน
2. **Class-Based Views (CBV)** - วิธีสร้าง view แบบ OOP ที่ reuse โค้ดได้ดีกว่า
3. **Generic Views** - ListView, DetailView, CreateView, UpdateView, DeleteView ที่ Django มีให้พร้อมใช้
4. **URL Patterns** - path(), re_path(), include() สำหรับกำหนดเส้นทาง URL
5. **Named URLs และ Namespaces** - การตั้งชื่อ URL เพื่อ reverse ได้ง่าย
6. **Django Templates** - ระบบ template ที่มี variables, tags, filters ครบครัน
7. **Template Inheritance** - การ inherit template เพื่อ DRY (Don't Repeat Yourself)
8. **Context Processors** - การเพิ่ม data เข้าไปใน context ของทุก template
9. **Middleware** - การสร้าง layer สำหรับจัดการ request/response

---

## แหล่งเรียนรู้เพิ่มเติม

- [Django Documentation - Views](https://docs.djangoproject.com/en/stable/topics/http/views/)
- [Django Documentation - URL dispatcher](https://docs.djangoproject.com/en/stable/topics/http/urls/)
- [Django Documentation - Templates](https://docs.djangoproject.com/en/stable/topics/templates/)
- [Django Documentation - Class-based views](https://docs.djangoproject.com/en/stable/topics/class-based-views/)
- [Classy Class-Based Views](https://ccbv.co.uk/) - แหล่งอ้างอิง CBV
- [Django Documentation - Middleware](https://docs.djangoproject.com/en/stable/topics/http/middleware/)
