# Part 65: Django - Forms & Class-Based Views

## สารบัญ

1. [Django Forms พื้นฐาน](#django-forms-พื้นฐาน)
2. [ModelForm](#modelform)
3. [Form Validation](#form-validation)
4. [Custom Validators](#custom-validators)
5. [Form Widgets](#form-widgets)
6. [Formsets](#formsets)
7. [Inline Formsets](#inline-formsets)
8. [File Uploads กับ ImageField](#file-uploads)
9. [Class-Based Views Deep Dive](#class-based-views-deep-dive)
10. [View Decorators](#view-decorators)
11. [AJAX กับ Django](#ajax-กับ-django)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Django Forms พื้นฐาน

### แนวคิดของ Django Forms

**Django Forms** คือระบบจัดการ HTML forms ที่ Django มีให้ ช่วยในการ:
- สร้าง HTML form fields อัตโนมัติ
- Validate ข้อมูลที่ผู้ใช้ส่งมา
- แสดง error messages
- แปลงข้อมูลให้อยู่ในรูปแบบที่ Python ใช้ได้

### การสร้าง Form พื้นฐาน

**ตัวอย่างที่ 1: Form พื้นฐาน**

```python
# myapp/forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        label='ชื่อ',
        widget=forms.TextInput(attrs={'placeholder': 'กรอกชื่อของคุณ'})
    )
    email = forms.EmailField(
        label='อีเมล',
        widget=forms.EmailInput(attrs={'placeholder': 'your@email.com'})
    )
    subject = forms.CharField(
        max_length=200,
        label='หัวข้อ'
    )
    message = forms.CharField(
        label='ข้อความ',
        widget=forms.Textarea(attrs={'rows': 5})
    )
    phone = forms.CharField(
        max_length=20,
        label='เบอร์โทรศัพท์',
        required=False  # ไม่บังคับกรอก
    )
```

**ตัวอย่างที่ 2: การใช้ Form ใน View**

```python
# myapp/views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import ContactForm

def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # ดึงข้อมูลที่ validated แล้ว
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            subject = form.cleaned_data['subject']
            message = form.cleaned_data['message']
            
            # ประมวลผลข้อมูล (เช่น ส่งอีเมล)
            # send_email(name, email, subject, message)
            
            messages.success(request, f'ขอบคุณ {name} ที่ติดต่อมา เราจะตอบกลับเร็วๆ นี้')
            return redirect('myapp:contact')
    else:
        form = ContactForm()
    
    return render(request, 'myapp/contact.html', {'form': form})
```

**ตัวอย่างที่ 3: Template สำหรับ Form**

```html
<!-- templates/myapp/contact.html -->
{% extends 'base.html' %}
{% block content %}
<h1>ติดต่อเรา</h1>

{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <li class="{{ message.tags }}">{{ message }}</li>
    {% endfor %}
</ul>
{% endif %}

<form method="post" novalidate>
    {% csrf_token %}
    
    <!-- แสดง form ทั้งหมดเป็น paragraphs -->
    {{ form.as_p }}
    
    <!-- หรือแสดงทีละ field -->
    <div class="form-group">
        <label for="{{ form.name.id_for_label }}">{{ form.name.label }}</label>
        {{ form.name }}
        {% if form.name.errors %}
        <ul class="errors">
            {% for error in form.name.errors %}
            <li>{{ error }}</li>
            {% endfor %}
        </ul>
        {% endif %}
    </div>
    
    <!-- แสดง non-field errors -->
    {% if form.non_field_errors %}
    <ul class="non-field-errors">
        {% for error in form.non_field_errors %}
        <li>{{ error }}</li>
        {% endfor %}
    </ul>
    {% endif %}
    
    <button type="submit">ส่งข้อความ</button>
</form>
{% endblock %}
```

### Form Field Types

**ตัวอย่างที่ 4: Field Types ต่างๆ**

```python
# myapp/forms.py
from django import forms
import datetime

class RegistrationForm(forms.Form):
    # Text fields
    username = forms.CharField(max_length=50)
    password = forms.CharField(widget=forms.PasswordInput)
    email = forms.EmailField()
    website = forms.URLField(required=False)
    
    # Number fields
    age = forms.IntegerField(min_value=0, max_value=150)
    price = forms.DecimalField(max_digits=10, decimal_places=2)
    
    # Date/Time fields
    birth_date = forms.DateField(
        widget=forms.DateInput(attrs={'type': 'date'})
    )
    meeting_time = forms.DateTimeField(
        widget=forms.DateTimeInput(attrs={'type': 'datetime-local'})
    )
    
    # Boolean fields
    agree_terms = forms.BooleanField(
        label='ฉันยอมรับเงื่อนไขการใช้งาน',
        required=True
    )
    newsletter = forms.BooleanField(
        label='สมัครรับจดหมายข่าว',
        required=False
    )
    
    # Choice fields
    GENDER_CHOICES = [
        ('', 'เลือกเพศ'),
        ('M', 'ชาย'),
        ('F', 'หญิง'),
        ('O', 'ไม่ระบุ'),
    ]
    gender = forms.ChoiceField(choices=GENDER_CHOICES)
    
    INTEREST_CHOICES = [
        ('python', 'Python'),
        ('django', 'Django'),
        ('javascript', 'JavaScript'),
        ('react', 'React'),
    ]
    interests = forms.MultipleChoiceField(
        choices=INTEREST_CHOICES,
        widget=forms.CheckboxSelectMultiple,
        required=False
    )
    
    # File fields
    avatar = forms.ImageField(required=False)
    
    # Hidden field
    referral_code = forms.CharField(
        widget=forms.HiddenInput,
        required=False
    )
```

---

## ModelForm

### แนวคิดของ ModelForm

**ModelForm** คือ Form ที่สร้างจาก Django Model โดยอัตโนมัติ ช่วยลดการเขียนโค้ดซ้ำ เพราะ Django จะสร้าง form fields จาก model fields ให้เอง

**ตัวอย่างที่ 5: ModelForm พื้นฐาน**

```python
# myapp/models.py
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=200, verbose_name='หัวข้อ')
    content = models.TextField(verbose_name='เนื้อหา')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.CharField(max_length=50, choices=[
        ('tech', 'เทคโนโลยี'),
        ('lifestyle', 'ไลฟ์สไตล์'),
        ('travel', 'ท่องเที่ยว'),
    ])
    is_published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = 'บทความ'

# myapp/forms.py
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'is_published']
        # หรือใช้ exclude เพื่อยกเว้น fields
        # exclude = ['author', 'created_at', 'updated_at']
        
        labels = {
            'title': 'หัวข้อบทความ',
            'content': 'เนื้อหา',
            'category': 'หมวดหมู่',
            'is_published': 'เผยแพร่',
        }
        
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10, 'class': 'editor'}),
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'หัวข้อบทความ'}),
        }
        
        help_texts = {
            'title': 'กรอกหัวข้อบทความที่ชัดเจนและน่าสนใจ',
            'content': 'รองรับ HTML formatting',
        }
```

**ตัวอย่างที่ 6: การใช้ ModelForm ใน View**

```python
# myapp/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .forms import PostForm
from .models import Post

@login_required
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)  # ยังไม่บันทึกลง DB
            post.author = request.user       # กำหนด author
            post.save()                      # บันทึกลง DB
            
            messages.success(request, 'สร้างบทความสำเร็จ!')
            return redirect('myapp:post_detail', pk=post.pk)
    else:
        form = PostForm()
    
    return render(request, 'myapp/post_form.html', {'form': form})

@login_required
def update_post(request, pk):
    post = get_object_or_404(Post, pk=pk, author=request.user)
    
    if request.method == 'POST':
        form = PostForm(request.POST, instance=post)  # ผูก instance กับ form
        if form.is_valid():
            form.save()
            messages.success(request, 'แก้ไขบทความสำเร็จ!')
            return redirect('myapp:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)  # Pre-fill form ด้วยข้อมูลเดิม
    
    return render(request, 'myapp/post_form.html', {'form': form, 'post': post})
```

**ตัวอย่างที่ 7: ModelForm กับ Many-to-Many Fields**

```python
# myapp/models.py
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    
    def __str__(self):
        return self.name

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    tags = models.ManyToManyField(Tag, blank=True)

# myapp/forms.py
from django import forms
from .models import Article, Tag

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'tags']
        widgets = {
            'tags': forms.CheckboxSelectMultiple,
        }
    
    # หรือใช้ ModelMultipleChoiceField
    tags = forms.ModelMultipleChoiceField(
        queryset=Tag.objects.all(),
        widget=forms.CheckboxSelectMultiple,
        required=False,
        label='แท็ก'
    )
```

---

## Form Validation

### clean() Method

**ตัวอย่างที่ 8: clean_fieldname() Method**

```python
# myapp/forms.py
from django import forms
from django.contrib.auth.models import User

class UserRegistrationForm(forms.Form):
    username = forms.CharField(max_length=50)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    password_confirm = forms.CharField(
        widget=forms.PasswordInput,
        label='ยืนยันรหัสผ่าน'
    )
    age = forms.IntegerField()
    
    def clean_username(self):
        """Validate เฉพาะ username field"""
        username = self.cleaned_data.get('username')
        
        # ตรวจสอบว่า username มีอยู่แล้วหรือไม่
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError(
                f'Username "{username}" ถูกใช้งานแล้ว'
            )
        
        # ตรวจสอบตัวอักษรที่อนุญาต
        import re
        if not re.match(r'^[a-zA-Z0-9_]+$', username):
            raise forms.ValidationError(
                'Username ต้องมีเฉพาะตัวอักษรภาษาอังกฤษ ตัวเลข หรือ _'
            )
        
        return username.lower()  # แปลงเป็นตัวพิมพ์เล็กก่อน return
    
    def clean_email(self):
        """Validate เฉพาะ email field"""
        email = self.cleaned_data.get('email')
        
        # ตรวจสอบ domain ที่ไม่อนุญาต
        blocked_domains = ['tempmail.com', 'throwaway.email']
        domain = email.split('@')[1].lower()
        
        if domain in blocked_domains:
            raise forms.ValidationError(
                f'ไม่สามารถใช้ email จาก {domain} ได้'
            )
        
        # ตรวจสอบว่า email มีอยู่แล้วหรือไม่
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('Email นี้ถูกลงทะเบียนแล้ว')
        
        return email.lower()
    
    def clean_age(self):
        """Validate เฉพาะ age field"""
        age = self.cleaned_data.get('age')
        
        if age < 13:
            raise forms.ValidationError('ต้องมีอายุอย่างน้อย 13 ปี')
        
        return age
    
    def clean(self):
        """Validate หลาย fields พร้อมกัน (cross-field validation)"""
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        password_confirm = cleaned_data.get('password_confirm')
        
        if password and password_confirm:
            if password != password_confirm:
                raise forms.ValidationError({
                    'password_confirm': 'รหัสผ่านไม่ตรงกัน'
                })
            
            # ตรวจสอบความซับซ้อนของรหัสผ่าน
            if len(password) < 8:
                raise forms.ValidationError({
                    'password': 'รหัสผ่านต้องมีความยาวอย่างน้อย 8 ตัวอักษร'
                })
        
        return cleaned_data
```

**ตัวอย่างที่ 9: Form Validation ใน ModelForm**

```python
# myapp/forms.py
from django import forms
from django.utils import timezone
from .models import Event

class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ['title', 'start_date', 'end_date', 'max_participants', 'location']
    
    def clean_start_date(self):
        start_date = self.cleaned_data.get('start_date')
        
        # วันที่เริ่มต้องไม่เป็นอดีต
        if start_date < timezone.now().date():
            raise forms.ValidationError('วันที่เริ่มต้องไม่เป็นวันในอดีต')
        
        return start_date
    
    def clean(self):
        cleaned_data = super().clean()
        start_date = cleaned_data.get('start_date')
        end_date = cleaned_data.get('end_date')
        
        if start_date and end_date:
            if end_date < start_date:
                raise forms.ValidationError(
                    'วันสิ้นสุดต้องมากกว่าหรือเท่ากับวันที่เริ่มต้น'
                )
            
            # Duration ต้องไม่เกิน 30 วัน
            duration = (end_date - start_date).days
            if duration > 30:
                raise forms.ValidationError(
                    'ระยะเวลาของ event ต้องไม่เกิน 30 วัน'
                )
        
        return cleaned_data
```

---

## Custom Validators

### การสร้าง Custom Validators

**ตัวอย่างที่ 10: Custom Validator Functions**

```python
# myapp/validators.py
from django.core.exceptions import ValidationError
from django.utils.translation import gettext_lazy as _
import re

def validate_thai_phone(value):
    """Validate เบอร์โทรศัพท์ไทย"""
    # รูปแบบ: 0xxxxxxxxx หรือ +66xxxxxxxxx
    pattern = r'^(\+66|0)[0-9]{9}$'
    if not re.match(pattern, value):
        raise ValidationError(
            _('กรุณากรอกเบอร์โทรศัพท์ที่ถูกต้อง (เช่น 0812345678 หรือ +66812345678)'),
            code='invalid_phone'
        )

def validate_no_profanity(value):
    """ตรวจสอบว่าไม่มีคำหยาบ"""
    profanity_words = ['คำหยาบ1', 'คำหยาบ2']  # ใส่คำหยาบจริงที่ต้องการตรวจสอบ
    value_lower = value.lower()
    
    for word in profanity_words:
        if word in value_lower:
            raise ValidationError(
                _('ข้อความมีคำที่ไม่เหมาะสม'),
                code='profanity'
            )

def validate_file_size(value):
    """Validate ขนาดไฟล์ (ไม่เกิน 5MB)"""
    filesize = value.size
    max_size = 5 * 1024 * 1024  # 5 MB
    
    if filesize > max_size:
        raise ValidationError(
            _(f'ขนาดไฟล์ต้องไม่เกิน 5 MB (ไฟล์ของคุณมีขนาด {filesize / 1024 / 1024:.2f} MB)')
        )

def validate_image_dimensions(value):
    """Validate ขนาดภาพ"""
    from PIL import Image
    
    img = Image.open(value)
    width, height = img.size
    
    if width < 200 or height < 200:
        raise ValidationError(
            _(f'ภาพต้องมีขนาดอย่างน้อย 200x200 pixels (ขนาดปัจจุบัน: {width}x{height})')
        )
    
    if width > 4000 or height > 4000:
        raise ValidationError(
            _(f'ภาพต้องมีขนาดไม่เกิน 4000x4000 pixels')
        )


# Class-based Validators
from django.core.validators import BaseValidator

class MaxFileSizeValidator(BaseValidator):
    """Validator class สำหรับขนาดไฟล์"""
    message = 'ขนาดไฟล์ต้องไม่เกิน %(limit_value)s MB'
    code = 'max_file_size'
    
    def compare(self, a, b):
        return a > b * 1024 * 1024
    
    def clean(self, x):
        return x.size / 1024 / 1024


# การใช้งานใน Model
class UserProfile(models.Model):
    phone = models.CharField(
        max_length=20,
        validators=[validate_thai_phone]
    )
    avatar = models.ImageField(
        upload_to='avatars/',
        validators=[validate_file_size, validate_image_dimensions]
    )
```

**ตัวอย่างที่ 11: Custom Validators ใน Form**

```python
# myapp/forms.py
from django import forms
from .validators import validate_thai_phone, validate_no_profanity

class ProfileForm(forms.Form):
    phone = forms.CharField(
        max_length=20,
        validators=[validate_thai_phone],
        label='เบอร์โทรศัพท์'
    )
    bio = forms.CharField(
        widget=forms.Textarea,
        validators=[validate_no_profanity],
        label='แนะนำตัว'
    )
    
    # Validator แบบ lambda (simple cases)
    username = forms.CharField(
        validators=[
            lambda v: None if len(v) >= 3 else (_ for _ in ()).throw(
                forms.ValidationError('Username ต้องมีความยาวอย่างน้อย 3 ตัวอักษร')
            )
        ]
    )
```

---

## Form Widgets

### Widget ต่างๆ ใน Django

**ตัวอย่างที่ 12: Widgets พื้นฐาน**

```python
# myapp/forms.py
from django import forms

class StyledForm(forms.Form):
    # TextInput - input type="text"
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'กรอกชื่อ',
            'id': 'name-input',
            'autocomplete': 'off',
        })
    )
    
    # Textarea
    description = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'cols': 60,
        })
    )
    
    # PasswordInput
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
    
    # EmailInput
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    
    # NumberInput
    quantity = forms.IntegerField(
        widget=forms.NumberInput(attrs={
            'class': 'form-control',
            'min': 1,
            'max': 100,
        })
    )
    
    # DateInput
    birth_date = forms.DateField(
        widget=forms.DateInput(attrs={
            'class': 'form-control',
            'type': 'date',
        })
    )
    
    # Select (Dropdown)
    COUNTRY_CHOICES = [
        ('TH', 'ไทย'),
        ('JP', 'ญี่ปุ่น'),
        ('US', 'สหรัฐอเมริกา'),
    ]
    country = forms.ChoiceField(
        choices=COUNTRY_CHOICES,
        widget=forms.Select(attrs={'class': 'form-control'})
    )
    
    # SelectMultiple
    colors = forms.MultipleChoiceField(
        choices=[('red', 'แดง'), ('green', 'เขียว'), ('blue', 'น้ำเงิน')],
        widget=forms.SelectMultiple(attrs={'class': 'form-control'})
    )
    
    # RadioSelect
    gender = forms.ChoiceField(
        choices=[('M', 'ชาย'), ('F', 'หญิง')],
        widget=forms.RadioSelect
    )
    
    # CheckboxInput
    agree = forms.BooleanField(
        widget=forms.CheckboxInput(attrs={'class': 'form-check-input'})
    )
    
    # CheckboxSelectMultiple
    notifications = forms.MultipleChoiceField(
        choices=[
            ('email', 'อีเมล'),
            ('sms', 'SMS'),
            ('push', 'Push Notification'),
        ],
        widget=forms.CheckboxSelectMultiple,
        required=False
    )
    
    # HiddenInput
    token = forms.CharField(
        widget=forms.HiddenInput,
        required=False
    )
    
    # FileInput
    document = forms.FileField(
        widget=forms.FileInput(attrs={'accept': '.pdf,.doc,.docx'}),
        required=False
    )
    
    # ClearableFileInput (แสดงปุ่มล้างไฟล์ด้วย)
    image = forms.ImageField(
        widget=forms.ClearableFileInput(attrs={'accept': 'image/*'}),
        required=False
    )
    
    # SplitDateTimeWidget - แยก date และ time
    appointment = forms.SplitDateTimeField(
        widget=forms.SplitDateTimeWidget(
            date_attrs={'type': 'date', 'class': 'form-control'},
            time_attrs={'type': 'time', 'class': 'form-control'},
        ),
        required=False
    )
```

**ตัวอย่างที่ 13: Custom Widget**

```python
# myapp/widgets.py
from django import forms
from django.utils.html import format_html

class RatingWidget(forms.Widget):
    """Widget สำหรับ star rating"""
    
    def render(self, name, value, attrs=None, renderer=None):
        html = '<div class="star-rating">'
        for i in range(1, 6):
            checked = 'checked' if value and int(value) >= i else ''
            html += f'''
            <input type="radio" 
                   id="{name}_{i}" 
                   name="{name}" 
                   value="{i}"
                   {checked}>
            <label for="{name}_{i}" title="{i} ดาว">★</label>
            '''
        html += '</div>'
        return format_html(html)
    
    def value_from_datadict(self, data, files, name):
        return data.get(name)


# การใช้งาน
class ReviewForm(forms.Form):
    rating = forms.IntegerField(
        widget=RatingWidget,
        min_value=1,
        max_value=5,
        label='คะแนน'
    )
    comment = forms.CharField(
        widget=forms.Textarea,
        label='ความคิดเห็น'
    )
```

---

## Formsets

### การสร้างและใช้ Formsets

**ตัวอย่างที่ 14: Formset พื้นฐาน**

```python
# myapp/forms.py
from django import forms
from django.forms import formset_factory

class IngredientForm(forms.Form):
    name = forms.CharField(max_length=100, label='ชื่อวัตถุดิบ')
    quantity = forms.DecimalField(label='จำนวน')
    unit = forms.CharField(max_length=20, label='หน่วย')

# สร้าง Formset จาก Form
IngredientFormSet = formset_factory(
    IngredientForm,
    extra=3,        # แสดง form เพิ่มเติม 3 form
    max_num=10,     # จำนวน form สูงสุด
    min_num=1,      # จำนวน form ขั้นต่ำ
    validate_max=True,
    validate_min=True,
    can_delete=True  # อนุญาตให้ลบ form ได้
)
```

**ตัวอย่างที่ 15: การใช้ Formset ใน View**

```python
# myapp/views.py
from django.shortcuts import render, redirect
from .forms import IngredientFormSet, RecipeForm

def create_recipe(request):
    if request.method == 'POST':
        recipe_form = RecipeForm(request.POST)
        ingredient_formset = IngredientFormSet(request.POST, prefix='ingredients')
        
        if recipe_form.is_valid() and ingredient_formset.is_valid():
            recipe = recipe_form.save()
            
            for form in ingredient_formset:
                if form.cleaned_data and not form.cleaned_data.get('DELETE'):
                    ingredient = form.save(commit=False)
                    ingredient.recipe = recipe
                    ingredient.save()
            
            return redirect('myapp:recipe_detail', pk=recipe.pk)
    else:
        recipe_form = RecipeForm()
        ingredient_formset = IngredientFormSet(prefix='ingredients')
    
    return render(request, 'myapp/create_recipe.html', {
        'recipe_form': recipe_form,
        'ingredient_formset': ingredient_formset,
    })
```

**ตัวอย่างที่ 16: Template สำหรับ Formset**

```html
<!-- templates/myapp/create_recipe.html -->
{% extends 'base.html' %}
{% block content %}
<h1>สร้างสูตรอาหาร</h1>

<form method="post">
    {% csrf_token %}
    
    <h2>ข้อมูลสูตรอาหาร</h2>
    {{ recipe_form.as_p }}
    
    <h2>วัตถุดิบ</h2>
    
    <!-- Management form - จำเป็นสำหรับ formset -->
    {{ ingredient_formset.management_form }}
    
    <div id="formset-container">
        {% for form in ingredient_formset %}
        <div class="ingredient-form">
            {{ form.as_p }}
            {% if form.DELETE %}
            {{ form.DELETE }} <label for="{{ form.DELETE.id_for_label }}">ลบ</label>
            {% endif %}
        </div>
        {% endfor %}
    </div>
    
    <button type="button" id="add-ingredient">เพิ่มวัตถุดิบ</button>
    <button type="submit">บันทึกสูตรอาหาร</button>
</form>

<script>
// JavaScript สำหรับเพิ่ม form ใหม่ใน formset
document.getElementById('add-ingredient').addEventListener('click', function() {
    const container = document.getElementById('formset-container');
    const totalForms = document.getElementById('id_ingredients-TOTAL_FORMS');
    const formCount = parseInt(totalForms.value);
    
    // Clone form แรก
    const newForm = container.children[0].cloneNode(true);
    
    // อัพเดต id และ name
    newForm.innerHTML = newForm.innerHTML.replace(
        /ingredients-0/g, 
        `ingredients-${formCount}`
    );
    
    // ล้างค่าใน form ใหม่
    newForm.querySelectorAll('input, select, textarea').forEach(input => {
        if (input.type !== 'hidden') {
            input.value = '';
        }
    });
    
    container.appendChild(newForm);
    totalForms.value = formCount + 1;
});
</script>
{% endblock %}
```

---

## Inline Formsets

### การสร้าง Inline Formsets

**ตัวอย่างที่ 17: Inline Formset**

```python
# myapp/models.py
from django.db import models

class Recipe(models.Model):
    name = models.CharField(max_length=200)
    description = models.TextField()

class Ingredient(models.Model):
    recipe = models.ForeignKey(Recipe, on_delete=models.CASCADE, related_name='ingredients')
    name = models.CharField(max_length=100)
    quantity = models.DecimalField(max_digits=8, decimal_places=2)
    unit = models.CharField(max_length=20)

# myapp/forms.py
from django.forms import inlineformset_factory
from .models import Recipe, Ingredient
from django import forms

class RecipeForm(forms.ModelForm):
    class Meta:
        model = Recipe
        fields = ['name', 'description']

class IngredientForm(forms.ModelForm):
    class Meta:
        model = Ingredient
        fields = ['name', 'quantity', 'unit']
        widgets = {
            'name': forms.TextInput(attrs={'class': 'form-control'}),
            'quantity': forms.NumberInput(attrs={'class': 'form-control', 'step': '0.01'}),
            'unit': forms.TextInput(attrs={'class': 'form-control'}),
        }

# สร้าง inline formset
IngredientInlineFormSet = inlineformset_factory(
    Recipe,         # Parent model
    Ingredient,     # Child model
    form=IngredientForm,
    extra=3,
    can_delete=True,
    min_num=1,
    validate_min=True,
)
```

**ตัวอย่างที่ 18: การใช้งาน Inline Formset ใน View**

```python
# myapp/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .forms import RecipeForm, IngredientInlineFormSet
from .models import Recipe

@login_required
def create_recipe(request):
    if request.method == 'POST':
        form = RecipeForm(request.POST)
        if form.is_valid():
            recipe = form.save(commit=False)
            recipe.author = request.user
            recipe.save()
            
            formset = IngredientInlineFormSet(request.POST, instance=recipe)
            if formset.is_valid():
                formset.save()
                return redirect('myapp:recipe_detail', pk=recipe.pk)
            else:
                recipe.delete()  # ลบ recipe ถ้า formset ไม่ valid
    else:
        form = RecipeForm()
        formset = IngredientInlineFormSet()
    
    return render(request, 'myapp/recipe_form.html', {
        'form': form,
        'formset': formset,
    })

@login_required
def update_recipe(request, pk):
    recipe = get_object_or_404(Recipe, pk=pk, author=request.user)
    
    if request.method == 'POST':
        form = RecipeForm(request.POST, instance=recipe)
        formset = IngredientInlineFormSet(request.POST, instance=recipe)
        
        if form.is_valid() and formset.is_valid():
            form.save()
            formset.save()
            return redirect('myapp:recipe_detail', pk=recipe.pk)
    else:
        form = RecipeForm(instance=recipe)
        formset = IngredientInlineFormSet(instance=recipe)
    
    return render(request, 'myapp/recipe_form.html', {
        'form': form,
        'formset': formset,
        'recipe': recipe,
    })
```

---

## File Uploads

### การจัดการ File Uploads

**ตัวอย่างที่ 19: การตั้งค่าสำหรับ File Upload**

```python
# settings.py
import os

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# myproject/urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... your url patterns
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

**ตัวอย่างที่ 20: Model กับ ImageField**

```python
# myapp/models.py
from django.db import models
import os
import uuid

def upload_to_user(instance, filename):
    """กำหนด path สำหรับ upload"""
    ext = filename.split('.')[-1]
    filename = f"{uuid.uuid4()}.{ext}"
    return os.path.join('users', str(instance.user.id), filename)

class UserProfile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    bio = models.TextField(blank=True)
    
    # ImageField ต้องการ Pillow library
    avatar = models.ImageField(
        upload_to=upload_to_user,
        null=True,
        blank=True,
        verbose_name='รูปโปรไฟล์'
    )
    
    # FileField สำหรับไฟล์ทั่วไป
    resume = models.FileField(
        upload_to='resumes/',
        null=True,
        blank=True,
        verbose_name='เรซูเม่'
    )
    
    def delete(self, *args, **kwargs):
        # ลบไฟล์เมื่อ delete record
        if self.avatar:
            if os.path.isfile(self.avatar.path):
                os.remove(self.avatar.path)
        if self.resume:
            if os.path.isfile(self.resume.path):
                os.remove(self.resume.path)
        super().delete(*args, **kwargs)
```

**ตัวอย่างที่ 21: Form กับ File Upload**

```python
# myapp/forms.py
from django import forms
from .models import UserProfile
from .validators import validate_file_size, validate_image_dimensions

class UserProfileForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['bio', 'avatar', 'resume']
        widgets = {
            'bio': forms.Textarea(attrs={'rows': 4}),
            'avatar': forms.ClearableFileInput(attrs={'accept': 'image/jpeg,image/png,image/gif'}),
            'resume': forms.ClearableFileInput(attrs={'accept': '.pdf,.doc,.docx'}),
        }
    
    def clean_avatar(self):
        avatar = self.cleaned_data.get('avatar')
        
        if avatar:
            # ตรวจสอบขนาดไฟล์
            if avatar.size > 2 * 1024 * 1024:  # 2 MB
                raise forms.ValidationError('รูปภาพต้องมีขนาดไม่เกิน 2 MB')
            
            # ตรวจสอบประเภทไฟล์
            allowed_types = ['image/jpeg', 'image/png', 'image/gif']
            if hasattr(avatar, 'content_type') and avatar.content_type not in allowed_types:
                raise forms.ValidationError('รองรับเฉพาะไฟล์ JPEG, PNG, GIF เท่านั้น')
        
        return avatar
    
    def clean_resume(self):
        resume = self.cleaned_data.get('resume')
        
        if resume:
            if resume.size > 5 * 1024 * 1024:  # 5 MB
                raise forms.ValidationError('เรซูเม่ต้องมีขนาดไม่เกิน 5 MB')
            
            # ตรวจสอบนามสกุลไฟล์
            import os
            ext = os.path.splitext(resume.name)[1].lower()
            if ext not in ['.pdf', '.doc', '.docx']:
                raise forms.ValidationError('รองรับเฉพาะไฟล์ PDF, DOC, DOCX เท่านั้น')
        
        return resume
```

**ตัวอย่างที่ 22: View สำหรับ File Upload**

```python
# myapp/views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from .forms import UserProfileForm
from .models import UserProfile

@login_required
def update_profile(request):
    profile, created = UserProfile.objects.get_or_create(user=request.user)
    
    if request.method == 'POST':
        # ต้องส่ง request.FILES ด้วย!
        form = UserProfileForm(request.POST, request.FILES, instance=profile)
        
        if form.is_valid():
            form.save()
            return redirect('myapp:profile')
    else:
        form = UserProfileForm(instance=profile)
    
    return render(request, 'myapp/update_profile.html', {'form': form})
```

**Template สำหรับ File Upload**

```html
<!-- templates/myapp/update_profile.html -->
{% extends 'base.html' %}
{% block content %}
<h1>แก้ไขโปรไฟล์</h1>

<!-- ต้องระบุ enctype="multipart/form-data" -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    <div class="form-group">
        <label>รูปโปรไฟล์ปัจจุบัน:</label>
        {% if form.instance.avatar %}
        <img src="{{ form.instance.avatar.url }}" alt="Avatar" width="100">
        {% else %}
        <p>ยังไม่มีรูปโปรไฟล์</p>
        {% endif %}
    </div>
    
    {{ form.as_p }}
    
    <button type="submit">บันทึก</button>
</form>
{% endblock %}
```

---

## Class-Based Views Deep Dive

### Mixins สำคัญ

**ตัวอย่างที่ 23: LoginRequiredMixin**

```python
# myapp/views.py
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Post

class PostListView(LoginRequiredMixin, ListView):
    """แสดงรายการ posts เฉพาะ user ที่ login"""
    model = Post
    template_name = 'myapp/post_list.html'
    login_url = '/login/'          # URL สำหรับ redirect ถ้ายังไม่ login
    redirect_field_name = 'next'   # ชื่อ query parameter สำหรับ redirect back
    
    def get_queryset(self):
        # แสดงเฉพาะ posts ของ user ที่ login
        return Post.objects.filter(author=self.request.user)


class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content', 'category']
    success_url = reverse_lazy('myapp:post_list')
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

**ตัวอย่างที่ 24: PermissionRequiredMixin**

```python
# myapp/views.py
from django.contrib.auth.mixins import PermissionRequiredMixin, LoginRequiredMixin
from django.views.generic import UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Post

class PostUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = Post
    fields = ['title', 'content', 'category', 'is_published']
    permission_required = 'myapp.change_post'  # permission name
    # หรือหลาย permissions:
    # permission_required = ['myapp.change_post', 'myapp.view_post']
    
    raise_exception = True  # raise PermissionDenied แทน redirect ไป login
    
    def get_success_url(self):
        return reverse_lazy('myapp:post_detail', kwargs={'pk': self.object.pk})
    
    def has_permission(self):
        """Override เพื่อเพิ่มเงื่อนไขเอง"""
        has_base_permission = super().has_permission()
        # อนุญาตถ้ามี permission หรือเป็นเจ้าของ post
        is_owner = self.get_object().author == self.request.user
        return has_base_permission or is_owner


class PostDeleteView(LoginRequiredMixin, PermissionRequiredMixin, DeleteView):
    model = Post
    permission_required = 'myapp.delete_post'
    success_url = reverse_lazy('myapp:post_list')
    
    def handle_no_permission(self):
        """Custom behavior เมื่อไม่มี permission"""
        from django.contrib import messages
        messages.error(self.request, 'คุณไม่มีสิทธิ์ลบบทความนี้')
        return super().handle_no_permission()
```

**ตัวอย่างที่ 25: UserPassesTestMixin**

```python
# myapp/views.py
from django.contrib.auth.mixins import UserPassesTestMixin, LoginRequiredMixin
from django.views.generic import UpdateView, DeleteView
from django.http import HttpResponseForbidden
from .models import Post

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    
    def test_func(self):
        """กำหนดเงื่อนไขที่ต้องผ่าน"""
        post = self.get_object()
        # อนุญาตเฉพาะ author หรือ staff
        return self.request.user == post.author or self.request.user.is_staff
    
    def handle_no_permission(self):
        """จัดการกรณีไม่ผ่านเงื่อนไข"""
        return HttpResponseForbidden('คุณไม่มีสิทธิ์แก้ไขบทความนี้')
```

### Custom Mixins

**ตัวอย่างที่ 26: สร้าง Custom Mixin**

```python
# myapp/mixins.py
from django.http import HttpResponseForbidden
from django.shortcuts import get_object_or_404

class AuthorRequiredMixin:
    """Mixin ที่อนุญาตเฉพาะ author ของ object"""
    author_field = 'author'
    
    def dispatch(self, request, *args, **kwargs):
        obj = self.get_object()
        author = getattr(obj, self.author_field)
        
        if author != request.user and not request.user.is_staff:
            return HttpResponseForbidden(
                'คุณไม่มีสิทธิ์ดำเนินการนี้'
            )
        
        return super().dispatch(request, *args, **kwargs)


class SuccessMessageMixin:
    """Mixin สำหรับแสดง success message"""
    success_message = ''
    
    def form_valid(self, form):
        response = super().form_valid(form)
        if self.success_message:
            from django.contrib import messages
            messages.success(
                self.request,
                self.success_message.format(**form.cleaned_data)
            )
        return response


# การใช้งาน
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import UpdateView
from .mixins import AuthorRequiredMixin, SuccessMessageMixin
from .models import Post

class PostUpdateView(LoginRequiredMixin, AuthorRequiredMixin, SuccessMessageMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    success_message = 'แก้ไขบทความ "{title}" สำเร็จ!'
```

---

## View Decorators

### @login_required Decorator

**ตัวอย่างที่ 27: @login_required**

```python
# myapp/views.py
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect

# การใช้งานพื้นฐาน
@login_required
def dashboard(request):
    return render(request, 'myapp/dashboard.html')

# กำหนด login URL
@login_required(login_url='/accounts/login/')
def profile(request):
    return render(request, 'myapp/profile.html')

# กำหนด redirect field name
@login_required(redirect_field_name='redirect_to')
def settings(request):
    return render(request, 'myapp/settings.html')
```

### @permission_required Decorator

**ตัวอย่างที่ 28: @permission_required**

```python
# myapp/views.py
from django.contrib.auth.decorators import permission_required, login_required
from django.shortcuts import render, redirect
from django.http import HttpResponseForbidden

# ต้องมี permission 'myapp.add_post'
@login_required
@permission_required('myapp.add_post', raise_exception=True)
def create_post(request):
    return render(request, 'myapp/create_post.html')

# หลาย permissions (ต้องมีทั้งหมด)
@login_required
@permission_required(['myapp.add_post', 'myapp.change_post'])
def manage_posts(request):
    return render(request, 'myapp/manage_posts.html')

# Custom permission check
from functools import wraps

def staff_required(view_func):
    """Decorator ที่ต้องการ staff permission"""
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            return redirect('login')
        if not request.user.is_staff:
            return HttpResponseForbidden('เฉพาะเจ้าหน้าที่เท่านั้น')
        return view_func(request, *args, **kwargs)
    return wrapper

@staff_required
def admin_only_view(request):
    return render(request, 'myapp/admin.html')
```

**ตัวอย่างที่ 29: Decorators อื่นๆ**

```python
# myapp/views.py
from django.views.decorators.http import (
    require_http_methods, require_GET, require_POST, require_safe
)
from django.views.decorators.cache import cache_page, never_cache
from django.views.decorators.vary import vary_on_headers, vary_on_cookie

# อนุญาตเฉพาะ GET method
@require_GET
def api_get_data(request):
    pass

# อนุญาตเฉพาะ POST method
@require_POST
def api_submit_data(request):
    pass

# อนุญาต GET และ HEAD methods เท่านั้น
@require_safe
def safe_view(request):
    pass

# Cache view 15 นาที
@cache_page(60 * 15)
def expensive_view(request):
    pass

# ห้าม cache
@never_cache
def sensitive_view(request):
    pass

# Custom decorator ที่ตรวจสอบ request rate
from functools import wraps
from django.http import HttpResponse
import time

def rate_limit(max_requests=10, window_seconds=60):
    """Rate limiting decorator"""
    request_counts = {}
    
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            client_ip = request.META.get('REMOTE_ADDR')
            now = time.time()
            
            if client_ip not in request_counts:
                request_counts[client_ip] = []
            
            # ลบ requests ที่เกิน window
            request_counts[client_ip] = [
                t for t in request_counts[client_ip]
                if now - t < window_seconds
            ]
            
            if len(request_counts[client_ip]) >= max_requests:
                return HttpResponse('Too Many Requests', status=429)
            
            request_counts[client_ip].append(now)
            return view_func(request, *args, **kwargs)
        
        return wrapper
    return decorator

@rate_limit(max_requests=5, window_seconds=60)
def api_endpoint(request):
    pass
```

---

## AJAX กับ Django

### JsonResponse

**ตัวอย่างที่ 30: JsonResponse พื้นฐาน**

```python
# myapp/views.py
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from django.contrib.auth.decorators import login_required
import json

@require_http_methods(["GET"])
def api_get_posts(request):
    """API endpoint สำหรับดึงรายการ posts"""
    from .models import Post
    
    posts = Post.objects.filter(
        is_published=True
    ).values('id', 'title', 'created_at', 'author__username')[:10]
    
    return JsonResponse({
        'status': 'success',
        'data': list(posts),
        'count': len(list(posts)),
    })

@login_required
@require_http_methods(["POST"])
def api_like_post(request, post_id):
    """API endpoint สำหรับ like post"""
    try:
        data = json.loads(request.body)
        action = data.get('action', 'like')
        
        from .models import Post, Like
        post = Post.objects.get(id=post_id)
        
        if action == 'like':
            like, created = Like.objects.get_or_create(
                user=request.user,
                post=post
            )
            message = 'ถูกใจแล้ว' if created else 'ถูกใจอยู่แล้ว'
        elif action == 'unlike':
            Like.objects.filter(user=request.user, post=post).delete()
            message = 'ยกเลิกถูกใจแล้ว'
        else:
            return JsonResponse({'status': 'error', 'message': 'action ไม่ถูกต้อง'}, status=400)
        
        likes_count = post.likes.count()
        
        return JsonResponse({
            'status': 'success',
            'message': message,
            'likes_count': likes_count,
        })
    
    except Post.DoesNotExist:
        return JsonResponse({'status': 'error', 'message': 'ไม่พบบทความ'}, status=404)
    except json.JSONDecodeError:
        return JsonResponse({'status': 'error', 'message': 'JSON ไม่ถูกต้อง'}, status=400)
    except Exception as e:
        return JsonResponse({'status': 'error', 'message': str(e)}, status=500)
```

**ตัวอย่างที่ 31: AJAX Form Submit**

```python
# myapp/views.py
from django.http import JsonResponse
from django.views.decorators.csrf import ensure_csrf_cookie
from django.contrib.auth.decorators import login_required
from .forms import CommentForm

@login_required
@require_http_methods(["POST"])
def ajax_add_comment(request, post_id):
    """AJAX endpoint สำหรับเพิ่ม comment"""
    form = CommentForm(request.POST)
    
    if form.is_valid():
        comment = form.save(commit=False)
        comment.author = request.user
        comment.post_id = post_id
        comment.save()
        
        return JsonResponse({
            'status': 'success',
            'comment': {
                'id': comment.id,
                'content': comment.content,
                'author': comment.author.username,
                'created_at': comment.created_at.strftime('%d/%m/%Y %H:%M'),
            }
        })
    
    return JsonResponse({
        'status': 'error',
        'errors': form.errors,
    }, status=400)
```

**ตัวอย่างที่ 32: JavaScript สำหรับ AJAX กับ Django**

```html
<!-- templates/myapp/post_detail.html -->
{% extends 'base.html' %}
{% block content %}
<article>
    <h1>{{ post.title }}</h1>
    <p>{{ post.content }}</p>
    
    <!-- Like Button -->
    <div id="like-section">
        <button id="like-btn" data-post-id="{{ post.id }}">
            ❤️ ถูกใจ (<span id="likes-count">{{ post.likes.count }}</span>)
        </button>
    </div>
    
    <!-- Comment Form -->
    <div id="comment-section">
        <h2>ความคิดเห็น</h2>
        
        <form id="comment-form">
            {% csrf_token %}
            {{ comment_form.as_p }}
            <button type="submit">ส่งความคิดเห็น</button>
        </form>
        
        <div id="comments-list">
            {% for comment in post.comments.all %}
            <div class="comment">
                <strong>{{ comment.author.username }}</strong>
                <p>{{ comment.content }}</p>
                <small>{{ comment.created_at|date:"d M Y H:i" }}</small>
            </div>
            {% endfor %}
        </div>
    </div>
</article>

<script>
// ดึง CSRF token
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

// Like button handler
document.getElementById('like-btn').addEventListener('click', async function() {
    const postId = this.dataset.postId;
    const csrfToken = getCookie('csrftoken');
    
    try {
        const response = await fetch(`/api/posts/${postId}/like/`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': csrfToken,
            },
            body: JSON.stringify({ action: 'like' }),
        });
        
        const data = await response.json();
        
        if (data.status === 'success') {
            document.getElementById('likes-count').textContent = data.likes_count;
        } else {
            alert('เกิดข้อผิดพลาด: ' + data.message);
        }
    } catch (error) {
        console.error('Error:', error);
    }
});

// Comment form handler
document.getElementById('comment-form').addEventListener('submit', async function(e) {
    e.preventDefault();
    
    const formData = new FormData(this);
    const csrfToken = getCookie('csrftoken');
    const postId = '{{ post.id }}';
    
    try {
        const response = await fetch(`/api/posts/${postId}/comments/`, {
            method: 'POST',
            headers: {
                'X-CSRFToken': csrfToken,
            },
            body: formData,
        });
        
        const data = await response.json();
        
        if (data.status === 'success') {
            // เพิ่ม comment ใหม่ใน list
            const commentsList = document.getElementById('comments-list');
            const newComment = document.createElement('div');
            newComment.className = 'comment';
            newComment.innerHTML = `
                <strong>${data.comment.author}</strong>
                <p>${data.comment.content}</p>
                <small>${data.comment.created_at}</small>
            `;
            commentsList.prepend(newComment);
            
            // ล้างฟอร์ม
            this.reset();
        } else {
            // แสดง errors
            const errors = data.errors;
            let errorMessage = '';
            for (const [field, messages] of Object.entries(errors)) {
                errorMessage += `${field}: ${messages.join(', ')}\n`;
            }
            alert('ข้อผิดพลาด:\n' + errorMessage);
        }
    } catch (error) {
        console.error('Error:', error);
    }
});
</script>
{% endblock %}
```

**ตัวอย่างที่ 33: CBV AJAX View**

```python
# myapp/views.py
from django.views import View
from django.http import JsonResponse
from django.contrib.auth.mixins import LoginRequiredMixin
import json

class LikePostView(LoginRequiredMixin, View):
    """CBV สำหรับจัดการ like/unlike post"""
    
    def post(self, request, post_id):
        try:
            data = json.loads(request.body)
            action = data.get('action', 'like')
            
            from .models import Post, Like
            
            try:
                post = Post.objects.get(id=post_id, is_published=True)
            except Post.DoesNotExist:
                return JsonResponse({'error': 'ไม่พบบทความ'}, status=404)
            
            if action == 'like':
                Like.objects.get_or_create(user=request.user, post=post)
            elif action == 'unlike':
                Like.objects.filter(user=request.user, post=post).delete()
            else:
                return JsonResponse({'error': 'action ไม่ถูกต้อง'}, status=400)
            
            return JsonResponse({
                'likes_count': post.likes.count(),
                'user_liked': post.likes.filter(user=request.user).exists(),
            })
        
        except json.JSONDecodeError:
            return JsonResponse({'error': 'Invalid JSON'}, status=400)
    
    def get(self, request, post_id):
        from .models import Post
        
        try:
            post = Post.objects.get(id=post_id)
            return JsonResponse({
                'likes_count': post.likes.count(),
                'user_liked': post.likes.filter(user=request.user).exists() 
                    if request.user.is_authenticated else False,
            })
        except Post.DoesNotExist:
            return JsonResponse({'error': 'ไม่พบบทความ'}, status=404)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Registration Form

**โจทย์:** สร้าง Registration Form ที่สมบูรณ์พร้อม validation

**เฉลย:**

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.models import User
import re

class RegistrationForm(forms.Form):
    username = forms.CharField(
        max_length=50,
        label='ชื่อผู้ใช้',
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    first_name = forms.CharField(
        max_length=50,
        label='ชื่อจริง',
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    last_name = forms.CharField(
        max_length=50,
        label='นามสกุล',
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    email = forms.EmailField(
        label='อีเมล',
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    password = forms.CharField(
        label='รหัสผ่าน',
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
    password_confirm = forms.CharField(
        label='ยืนยันรหัสผ่าน',
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
    birth_date = forms.DateField(
        label='วันเกิด',
        widget=forms.DateInput(attrs={'class': 'form-control', 'type': 'date'}),
        required=False
    )
    agree_terms = forms.BooleanField(
        label='ฉันยอมรับเงื่อนไขการใช้งาน',
        required=True
    )
    
    def clean_username(self):
        username = self.cleaned_data.get('username')
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError('Username นี้ถูกใช้งานแล้ว')
        if not re.match(r'^[a-zA-Z0-9_]+$', username):
            raise forms.ValidationError('Username ต้องมีเฉพาะตัวอักษร ตัวเลข หรือ _')
        return username
    
    def clean_email(self):
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('Email นี้ถูกลงทะเบียนแล้ว')
        return email
    
    def clean_password(self):
        password = self.cleaned_data.get('password')
        if len(password) < 8:
            raise forms.ValidationError('รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
        if not re.search(r'[A-Z]', password):
            raise forms.ValidationError('รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว')
        if not re.search(r'[0-9]', password):
            raise forms.ValidationError('รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว')
        return password
    
    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        password_confirm = cleaned_data.get('password_confirm')
        
        if password and password_confirm and password != password_confirm:
            raise forms.ValidationError({
                'password_confirm': 'รหัสผ่านไม่ตรงกัน'
            })
        
        return cleaned_data
    
    def save(self):
        """บันทึก user ใหม่"""
        data = self.cleaned_data
        user = User.objects.create_user(
            username=data['username'],
            email=data['email'],
            password=data['password'],
            first_name=data['first_name'],
            last_name=data['last_name'],
        )
        return user
```

---

### แบบฝึกหัดที่ 2: ModelForm กับ Custom Widgets

**โจทย์:** สร้าง ProductForm ที่มี custom widget และ validation

**เฉลย:**

```python
# shop/models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    category = models.ForeignKey('Category', on_delete=models.SET_NULL, null=True)
    image = models.ImageField(upload_to='products/', null=True, blank=True)
    is_active = models.BooleanField(default=True)
    discount_percent = models.DecimalField(
        max_digits=5, decimal_places=2, default=0
    )

# shop/forms.py
from django import forms
from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ['name', 'description', 'price', 'stock', 'category', 
                  'image', 'is_active', 'discount_percent']
        widgets = {
            'name': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'ชื่อสินค้า',
            }),
            'description': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
            }),
            'price': forms.NumberInput(attrs={
                'class': 'form-control',
                'step': '0.01',
                'min': '0',
            }),
            'stock': forms.NumberInput(attrs={
                'class': 'form-control',
                'min': '0',
            }),
            'discount_percent': forms.NumberInput(attrs={
                'class': 'form-control',
                'step': '0.01',
                'min': '0',
                'max': '100',
            }),
        }
    
    def clean_price(self):
        price = self.cleaned_data.get('price')
        if price < 0:
            raise forms.ValidationError('ราคาต้องไม่น้อยกว่า 0')
        return price
    
    def clean_discount_percent(self):
        discount = self.cleaned_data.get('discount_percent')
        if discount < 0 or discount > 100:
            raise forms.ValidationError('ส่วนลดต้องอยู่ระหว่าง 0-100%')
        return discount
    
    def clean_image(self):
        image = self.cleaned_data.get('image')
        if image:
            if image.size > 5 * 1024 * 1024:
                raise forms.ValidationError('ขนาดรูปต้องไม่เกิน 5 MB')
        return image
```

---

### แบบฝึกหัดที่ 3: Inline Formset สำหรับ Order

**โจทย์:** สร้าง Order form พร้อม inline formset สำหรับ order items

**เฉลย:**

```python
# shop/models.py
from django.db import models
from django.contrib.auth.models import User

class Order(models.Model):
    STATUS_CHOICES = [
        ('pending', 'รอดำเนินการ'),
        ('processing', 'กำลังดำเนินการ'),
        ('shipped', 'จัดส่งแล้ว'),
        ('delivered', 'ได้รับแล้ว'),
        ('cancelled', 'ยกเลิก'),
    ]
    
    customer = models.ForeignKey(User, on_delete=models.CASCADE)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    shipping_address = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    @property
    def total_price(self):
        return sum(item.total_price for item in self.items.all())

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='items')
    product = models.ForeignKey('Product', on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
    
    @property
    def total_price(self):
        return self.quantity * self.unit_price

# shop/forms.py
from django import forms
from django.forms import inlineformset_factory
from .models import Order, OrderItem

class OrderForm(forms.ModelForm):
    class Meta:
        model = Order
        fields = ['shipping_address']
        widgets = {
            'shipping_address': forms.Textarea(attrs={
                'rows': 3,
                'class': 'form-control',
                'placeholder': 'ที่อยู่สำหรับจัดส่ง'
            })
        }

class OrderItemForm(forms.ModelForm):
    class Meta:
        model = OrderItem
        fields = ['product', 'quantity', 'unit_price']
        widgets = {
            'product': forms.Select(attrs={'class': 'form-control'}),
            'quantity': forms.NumberInput(attrs={'class': 'form-control', 'min': 1}),
            'unit_price': forms.NumberInput(attrs={'class': 'form-control', 'step': '0.01'}),
        }
    
    def clean_quantity(self):
        quantity = self.cleaned_data.get('quantity')
        if quantity <= 0:
            raise forms.ValidationError('จำนวนต้องมากกว่า 0')
        return quantity

OrderItemInlineFormSet = inlineformset_factory(
    Order, OrderItem,
    form=OrderItemForm,
    extra=1,
    min_num=1,
    validate_min=True,
    can_delete=True
)
```

---

### แบบฝึกหัดที่ 4: File Upload พร้อม Image Resize

**โจทย์:** สร้าง system upload รูปภาพที่ resize อัตโนมัติและ save หลายขนาด

**เฉลย:**

```python
# myapp/utils.py
from PIL import Image
import os
from io import BytesIO
from django.core.files.base import ContentFile

def resize_image(image_field, sizes=None):
    """Resize รูปภาพเป็นหลายขนาด"""
    if sizes is None:
        sizes = {
            'thumbnail': (150, 150),
            'medium': (800, 600),
            'large': (1920, 1080),
        }
    
    results = {}
    
    for size_name, (width, height) in sizes.items():
        img = Image.open(image_field)
        img.thumbnail((width, height), Image.Resampling.LANCZOS)
        
        # Convert to RGB ถ้าเป็น RGBA
        if img.mode in ('RGBA', 'P'):
            img = img.convert('RGB')
        
        # บันทึกในหน่วยความจำก่อน
        img_io = BytesIO()
        img.save(img_io, format='JPEG', quality=85, optimize=True)
        img_io.seek(0)
        
        results[size_name] = ContentFile(img_io.read())
    
    return results

# myapp/models.py
from django.db import models
from .utils import resize_image

class Gallery(models.Model):
    title = models.CharField(max_length=200)
    original = models.ImageField(upload_to='gallery/original/')
    thumbnail = models.ImageField(upload_to='gallery/thumbnails/', blank=True)
    medium = models.ImageField(upload_to='gallery/medium/', blank=True)
    
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        
        if self.original:
            resized = resize_image(self.original, {
                'thumbnail': (150, 150),
                'medium': (800, 600),
            })
            
            base_name = os.path.splitext(os.path.basename(self.original.name))[0]
            
            if 'thumbnail' in resized:
                self.thumbnail.save(
                    f'{base_name}_thumb.jpg',
                    resized['thumbnail'],
                    save=False
                )
            if 'medium' in resized:
                self.medium.save(
                    f'{base_name}_medium.jpg',
                    resized['medium'],
                    save=False
                )
            
            # Save อีกครั้งเพื่ออัพเดต thumbnail และ medium fields
            super().save(update_fields=['thumbnail', 'medium'])
```

---

### แบบฝึกหัดที่ 5: AJAX Form Validation

**โจทย์:** สร้าง AJAX endpoint สำหรับตรวจสอบ username แบบ real-time

**เฉลย:**

```python
# accounts/views.py
from django.http import JsonResponse
from django.contrib.auth.models import User
from django.views.decorators.http import require_GET
import re

@require_GET
def check_username_availability(request):
    """AJAX endpoint ตรวจสอบ username"""
    username = request.GET.get('username', '').strip()
    
    if not username:
        return JsonResponse({'available': False, 'message': 'กรุณากรอก username'})
    
    if len(username) < 3:
        return JsonResponse({'available': False, 'message': 'Username ต้องมีอย่างน้อย 3 ตัวอักษร'})
    
    if len(username) > 50:
        return JsonResponse({'available': False, 'message': 'Username ต้องไม่เกิน 50 ตัวอักษร'})
    
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        return JsonResponse({'available': False, 'message': 'Username ต้องมีเฉพาะตัวอักษร ตัวเลข หรือ _'})
    
    reserved_usernames = ['admin', 'root', 'api', 'www', 'mail', 'support']
    if username.lower() in reserved_usernames:
        return JsonResponse({'available': False, 'message': 'Username นี้ไม่สามารถใช้ได้'})
    
    is_available = not User.objects.filter(username__iexact=username).exists()
    
    return JsonResponse({
        'available': is_available,
        'message': 'Username นี้ว่างอยู่' if is_available else 'Username นี้ถูกใช้งานแล้ว',
    })

# accounts/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('check-username/', views.check_username_availability, name='check_username'),
]
```

```html
<!-- templates/accounts/register.html -->
<div class="form-group">
    <label for="id_username">ชื่อผู้ใช้</label>
    <input type="text" id="id_username" name="username" class="form-control">
    <div id="username-feedback" class="feedback-message"></div>
</div>

<script>
let usernameCheckTimeout;

document.getElementById('id_username').addEventListener('input', function() {
    clearTimeout(usernameCheckTimeout);
    
    const username = this.value.trim();
    const feedback = document.getElementById('username-feedback');
    
    if (username.length === 0) {
        feedback.textContent = '';
        feedback.className = 'feedback-message';
        return;
    }
    
    feedback.textContent = 'กำลังตรวจสอบ...';
    feedback.className = 'feedback-message checking';
    
    usernameCheckTimeout = setTimeout(async () => {
        try {
            const response = await fetch(
                `/accounts/check-username/?username=${encodeURIComponent(username)}`
            );
            const data = await response.json();
            
            feedback.textContent = data.message;
            feedback.className = `feedback-message ${data.available ? 'available' : 'unavailable'}`;
        } catch (error) {
            feedback.textContent = 'เกิดข้อผิดพลาดในการตรวจสอบ';
            feedback.className = 'feedback-message error';
        }
    }, 500);
});
</script>
```

---

### แบบฝึกหัดที่ 6: CBV กับ Mixin

**โจทย์:** สร้าง Blog CRUD Views ด้วย CBV พร้อม Custom Mixins

**เฉลย:**

```python
# blog/mixins.py
from django.contrib.auth.mixins import LoginRequiredMixin
from django.http import HttpResponseForbidden
from django.contrib import messages

class AuthorOrStaffMixin:
    """อนุญาตเฉพาะ author หรือ staff"""
    
    def get_object(self, queryset=None):
        obj = super().get_object(queryset)
        if obj.author != self.request.user and not self.request.user.is_staff:
            raise PermissionError("คุณไม่มีสิทธิ์ดำเนินการนี้")
        return obj
    
    def dispatch(self, request, *args, **kwargs):
        try:
            return super().dispatch(request, *args, **kwargs)
        except PermissionError as e:
            messages.error(request, str(e))
            from django.shortcuts import redirect
            return redirect('blog:post_list')


class LogActionMixin:
    """บันทึก log การกระทำ"""
    action_log_message = ''
    
    def form_valid(self, form):
        response = super().form_valid(form)
        if self.action_log_message:
            print(f"[LOG] User {self.request.user}: {self.action_log_message}")
        return response


# blog/views.py
from django.views.generic import CreateView, UpdateView, DeleteView, ListView, DetailView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from .mixins import AuthorOrStaffMixin, LogActionMixin
from .models import Post
from .forms import PostForm

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        return Post.objects.filter(status='published').order_by('-created_at')

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'

class PostCreateView(LoginRequiredMixin, LogActionMixin, CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_url = reverse_lazy('blog:post_list')
    action_log_message = 'สร้างบทความใหม่'
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        messages.success(self.request, 'สร้างบทความสำเร็จ!')
        return super().form_valid(form)

class PostUpdateView(LoginRequiredMixin, AuthorOrStaffMixin, LogActionMixin, UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    action_log_message = 'แก้ไขบทความ'
    
    def get_success_url(self):
        messages.success(self.request, 'แก้ไขบทความสำเร็จ!')
        return reverse_lazy('blog:post_detail', kwargs={'pk': self.object.pk})

class PostDeleteView(LoginRequiredMixin, AuthorOrStaffMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('blog:post_list')
    
    def delete(self, request, *args, **kwargs):
        messages.success(request, 'ลบบทความสำเร็จ!')
        return super().delete(request, *args, **kwargs)
```

---

### แบบฝึกหัดที่ 7: Custom Form Validation สำหรับระบบจอง

**โจทย์:** สร้างระบบจองห้องพักพร้อม validation

**เฉลย:**

```python
# booking/forms.py
from django import forms
from django.utils import timezone
from .models import Booking, Room
import datetime

class BookingForm(forms.ModelForm):
    class Meta:
        model = Booking
        fields = ['room', 'check_in_date', 'check_out_date', 'num_guests', 'special_requests']
        widgets = {
            'check_in_date': forms.DateInput(attrs={'type': 'date', 'class': 'form-control'}),
            'check_out_date': forms.DateInput(attrs={'type': 'date', 'class': 'form-control'}),
            'num_guests': forms.NumberInput(attrs={'class': 'form-control', 'min': 1}),
            'special_requests': forms.Textarea(attrs={'rows': 3, 'class': 'form-control'}),
        }
    
    def clean_check_in_date(self):
        check_in = self.cleaned_data.get('check_in_date')
        today = timezone.now().date()
        
        if check_in < today:
            raise forms.ValidationError('วันเช็คอินต้องไม่เป็นวันในอดีต')
        
        # จองล่วงหน้าได้สูงสุด 1 ปี
        max_advance = today + datetime.timedelta(days=365)
        if check_in > max_advance:
            raise forms.ValidationError('ไม่สามารถจองล่วงหน้าเกิน 1 ปีได้')
        
        return check_in
    
    def clean(self):
        cleaned_data = super().clean()
        check_in = cleaned_data.get('check_in_date')
        check_out = cleaned_data.get('check_out_date')
        room = cleaned_data.get('room')
        num_guests = cleaned_data.get('num_guests')
        
        if check_in and check_out:
            # Check out ต้องหลัง check in
            if check_out <= check_in:
                raise forms.ValidationError('วันเช็คเอาต์ต้องหลังวันเช็คอิน')
            
            # พักได้สูงสุด 30 คืน
            duration = (check_out - check_in).days
            if duration > 30:
                raise forms.ValidationError('ไม่สามารถจองพักเกิน 30 คืนได้')
        
        if room and num_guests:
            # ตรวจสอบความจุของห้อง
            if num_guests > room.max_occupancy:
                raise forms.ValidationError(
                    f'ห้องนี้รองรับได้สูงสุด {room.max_occupancy} คน'
                )
        
        if check_in and check_out and room:
            # ตรวจสอบว่าห้องว่างในช่วงเวลาที่จอง
            conflicting = Booking.objects.filter(
                room=room,
                status__in=['pending', 'confirmed'],
                check_in_date__lt=check_out,
                check_out_date__gt=check_in,
            )
            
            # ยกเว้น booking ปัจจุบัน (กรณี update)
            if self.instance and self.instance.pk:
                conflicting = conflicting.exclude(pk=self.instance.pk)
            
            if conflicting.exists():
                raise forms.ValidationError(
                    'ห้องนี้ถูกจองในช่วงเวลาดังกล่าวแล้ว'
                )
        
        return cleaned_data
```

---

### แบบฝึกหัดที่ 8: Complete Mini-Project - Event Registration System

**โจทย์:** สร้างระบบลงทะเบียนงาน event ที่สมบูรณ์

**เฉลย:**

```python
# events/models.py
from django.db import models
from django.contrib.auth.models import User

class Event(models.Model):
    title = models.CharField(max_length=200, verbose_name='ชื่องาน')
    description = models.TextField(verbose_name='รายละเอียด')
    location = models.CharField(max_length=300, verbose_name='สถานที่')
    start_datetime = models.DateTimeField(verbose_name='วันเวลาเริ่มต้น')
    end_datetime = models.DateTimeField(verbose_name='วันเวลาสิ้นสุด')
    max_participants = models.IntegerField(default=50)
    organizer = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    @property
    def registered_count(self):
        return self.registrations.filter(status='confirmed').count()
    
    @property
    def is_full(self):
        return self.registered_count >= self.max_participants
    
    @property
    def available_spots(self):
        return max(0, self.max_participants - self.registered_count)

class Registration(models.Model):
    STATUS_CHOICES = [
        ('pending', 'รอยืนยัน'),
        ('confirmed', 'ยืนยันแล้ว'),
        ('cancelled', 'ยกเลิก'),
    ]
    
    event = models.ForeignKey(Event, on_delete=models.CASCADE, related_name='registrations')
    participant = models.ForeignKey(User, on_delete=models.CASCADE)
    full_name = models.CharField(max_length=200)
    phone = models.CharField(max_length=20)
    email = models.EmailField()
    dietary_requirements = models.TextField(blank=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    registered_at = models.DateTimeField(auto_now_add=True)

# events/forms.py
from django import forms
from .models import Event, Registration
from .validators import validate_thai_phone

class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ['title', 'description', 'location', 'start_datetime', 
                  'end_datetime', 'max_participants']
        widgets = {
            'start_datetime': forms.DateTimeInput(
                attrs={'type': 'datetime-local', 'class': 'form-control'}
            ),
            'end_datetime': forms.DateTimeInput(
                attrs={'type': 'datetime-local', 'class': 'form-control'}
            ),
        }
    
    def clean(self):
        cleaned_data = super().clean()
        start = cleaned_data.get('start_datetime')
        end = cleaned_data.get('end_datetime')
        
        if start and end:
            from django.utils import timezone
            if start < timezone.now():
                raise forms.ValidationError({'start_datetime': 'วันเวลาเริ่มต้องไม่เป็นอดีต'})
            if end <= start:
                raise forms.ValidationError({'end_datetime': 'วันเวลาสิ้นสุดต้องหลังวันเวลาเริ่มต้น'})
        
        return cleaned_data

class RegistrationForm(forms.ModelForm):
    class Meta:
        model = Registration
        fields = ['full_name', 'phone', 'email', 'dietary_requirements']
        widgets = {
            'dietary_requirements': forms.Textarea(attrs={'rows': 2}),
        }
    
    def __init__(self, *args, event=None, **kwargs):
        super().__init__(*args, **kwargs)
        self.event = event
    
    def clean_phone(self):
        phone = self.cleaned_data.get('phone')
        validate_thai_phone(phone)
        return phone
    
    def clean(self):
        cleaned_data = super().clean()
        
        if self.event and self.event.is_full:
            raise forms.ValidationError('ขออภัย งานนี้มีผู้ลงทะเบียนเต็มแล้ว')
        
        return cleaned_data

# events/views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.shortcuts import get_object_or_404, redirect
from django.contrib import messages
from django.urls import reverse_lazy
from .models import Event, Registration
from .forms import EventForm, RegistrationForm

class EventListView(ListView):
    model = Event
    template_name = 'events/list.html'
    context_object_name = 'events'
    paginate_by = 10
    
    def get_queryset(self):
        from django.utils import timezone
        return Event.objects.filter(
            end_datetime__gte=timezone.now()
        ).order_by('start_datetime')

class EventDetailView(DetailView):
    model = Event
    template_name = 'events/detail.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        if self.request.user.is_authenticated:
            context['user_registration'] = Registration.objects.filter(
                event=self.object,
                participant=self.request.user,
                status__in=['pending', 'confirmed']
            ).first()
        context['registration_form'] = RegistrationForm(event=self.object)
        return context

class EventRegisterView(LoginRequiredMixin, CreateView):
    model = Registration
    form_class = RegistrationForm
    template_name = 'events/register.html'
    
    def get_event(self):
        return get_object_or_404(Event, pk=self.kwargs['event_pk'])
    
    def get_form_kwargs(self):
        kwargs = super().get_form_kwargs()
        kwargs['event'] = self.get_event()
        return kwargs
    
    def form_valid(self, form):
        event = self.get_event()
        
        # ตรวจสอบว่า user ลงทะเบียนแล้วหรือยัง
        if Registration.objects.filter(
            event=event,
            participant=self.request.user,
            status__in=['pending', 'confirmed']
        ).exists():
            messages.error(self.request, 'คุณได้ลงทะเบียนงานนี้แล้ว')
            return redirect('events:detail', pk=event.pk)
        
        form.instance.event = event
        form.instance.participant = self.request.user
        form.instance.status = 'confirmed'
        
        response = super().form_valid(form)
        messages.success(self.request, f'ลงทะเบียนงาน "{event.title}" สำเร็จ!')
        
        return response
    
    def get_success_url(self):
        return reverse_lazy('events:detail', kwargs={'pk': self.kwargs['event_pk']})

# events/urls.py
from django.urls import path
from . import views

app_name = 'events'

urlpatterns = [
    path('', views.EventListView.as_view(), name='list'),
    path('<int:pk>/', views.EventDetailView.as_view(), name='detail'),
    path('create/', views.EventCreateView.as_view(), name='create'),
    path('<int:event_pk>/register/', views.EventRegisterView.as_view(), name='register'),
]
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Django Forms** - การสร้างและใช้งาน form ใน Django
2. **ModelForm** - การสร้าง form จาก model เพื่อลดการเขียนโค้ดซ้ำ
3. **Form Validation** - clean(), clean_fieldname() สำหรับ validate ข้อมูล
4. **Custom Validators** - การสร้าง validator function และ class เอง
5. **Form Widgets** - การ customize การแสดงผล form fields
6. **Formsets** - การจัดการหลาย forms พร้อมกัน
7. **Inline Formsets** - การจัดการ related objects ใน form เดียว
8. **File Uploads** - การจัดการ file และ image uploads
9. **CBV Mixins** - LoginRequiredMixin, PermissionRequiredMixin, UserPassesTestMixin
10. **View Decorators** - @login_required, @permission_required, custom decorators
11. **AJAX** - JsonResponse, AJAX form submission, real-time validation

---

## แหล่งเรียนรู้เพิ่มเติม

- [Django Documentation - Forms](https://docs.djangoproject.com/en/stable/topics/forms/)
- [Django Documentation - ModelForm](https://docs.djangoproject.com/en/stable/topics/forms/modelforms/)
- [Django Documentation - Formsets](https://docs.djangoproject.com/en/stable/topics/forms/formsets/)
- [Django Documentation - File Uploads](https://docs.djangoproject.com/en/stable/topics/http/file-uploads/)
- [Django Documentation - Class-based views](https://docs.djangoproject.com/en/stable/topics/class-based-views/)
- [Django Documentation - Mixins](https://docs.djangoproject.com/en/stable/topics/class-based-views/mixins/)
- [Classy Class-Based Views](https://ccbv.co.uk/) - อ้างอิง CBV ที่สมบูรณ์
