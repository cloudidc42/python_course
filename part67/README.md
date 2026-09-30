# Part 67: Django Authentication & Permissions

## บทนำ

Django มีระบบ Authentication ในตัวที่ทรงพลัง รองรับ User model, login/logout, password management และ permissions system ในบทนี้เราจะเรียนรู้การ customize ระบบ auth ของ Django, สร้าง Custom User model, การจัดการ permissions และ group-based access control รวมถึง third-party packages อย่าง django-allauth และ django-guardian

## สารบัญ

1. [Django Auth System Overview](#1-django-auth-system-overview)
2. [Custom User Model](#2-custom-user-model)
3. [User Registration Flow](#3-user-registration-flow)
4. [Login/Logout](#4-loginlogout)
5. [Password Reset Flow](#5-password-reset-flow)
6. [Email Verification](#6-email-verification)
7. [django-allauth](#7-django-allauth)
8. [Social Authentication](#8-social-authentication)
9. [Permissions System](#9-permissions-system)
10. [Group-based Permissions](#10-group-based-permissions)
11. [Object-level Permissions](#11-object-level-permissions)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. Django Auth System Overview

### โครงสร้างหลักของ Django Auth

```
django.contrib.auth
├── models
│   ├── User                 # Default user model
│   ├── Permission           # Permission object
│   ├── Group               # User group
│   └── AbstractBaseUser    # Base class สำหรับ custom user
├── backends
│   └── ModelBackend        # Default authentication backend
├── views                   # Login, logout, password reset views
├── forms                   # AuthenticationForm, PasswordResetForm
└── middleware
    └── AuthenticationMiddleware
```

### Components หลัก

```python
# ตัวอย่างที่ 1: User model attributes
from django.contrib.auth.models import User

user = User.objects.get(username='john')

# Attributes
print(user.id)              # Primary key
print(user.username)        # Username
print(user.email)           # Email
print(user.first_name)      # First name
print(user.last_name)       # Last name
print(user.is_active)       # Active status
print(user.is_staff)        # Can access admin
print(user.is_superuser)    # Has all permissions
print(user.date_joined)     # Registration date
print(user.last_login)      # Last login time

# Methods
user.set_password('newpassword')
user.check_password('password')
user.get_full_name()
user.get_short_name()

# Permissions
user.has_perm('app.permission_codename')
user.has_perms(['app.perm1', 'app.perm2'])
user.has_module_perms('app')
user.get_all_permissions()
user.get_group_permissions()
user.get_user_permissions()
```

### Authentication Backend

```python
# ตัวอย่างที่ 2: Authentication Backend
from django.contrib.auth.backends import ModelBackend
from django.contrib.auth import get_user_model

User = get_user_model()

class EmailBackend(ModelBackend):
    """Authentication ด้วย email แทน username"""
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        # ลอง authenticate ด้วย email
        try:
            user = User.objects.get(email=username)
        except User.DoesNotExist:
            return None
        
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        return None
    
    def get_user(self, user_id):
        try:
            user = User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
        return user if self.user_can_authenticate(user) else None

class UsernameOrEmailBackend(ModelBackend):
    """Authentication ด้วย username หรือ email"""
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        from django.db.models import Q
        
        try:
            user = User.objects.get(
                Q(username=username) | Q(email=username)
            )
        except User.DoesNotExist:
            return None
        except User.MultipleObjectsReturned:
            # ถ้ามี email ซ้ำ ใช้ username แทน
            try:
                user = User.objects.get(username=username)
            except User.DoesNotExist:
                return None
        
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        return None

# settings.py
AUTHENTICATION_BACKENDS = [
    'accounts.backends.UsernameOrEmailBackend',
    'django.contrib.auth.backends.ModelBackend',
]
```

---

## 2. Custom User Model

### สิ่งสำคัญ: ต้องสร้าง Custom User model ก่อน migrate ครั้งแรก

### AbstractUser (แนะนำ)

```python
# ตัวอย่างที่ 3: AbstractUser - extend User model ที่มีอยู่
# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    """Custom User model ที่ extend AbstractUser"""
    
    # เพิ่ม fields ใหม่
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    phone = models.CharField(max_length=20, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    website = models.URLField(blank=True)
    
    # Email เป็น required และ unique
    email = models.EmailField(unique=True)
    
    # เพิ่ม choices
    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
        ('N', 'Prefer not to say'),
    ]
    gender = models.CharField(
        max_length=1,
        choices=GENDER_CHOICES,
        blank=True
    )
    
    # Tracking fields
    last_active = models.DateTimeField(null=True, blank=True)
    email_verified = models.BooleanField(default=False)
    
    USERNAME_FIELD = 'email'  # ใช้ email เป็น login field
    REQUIRED_FIELDS = ['username']  # Fields สำหรับ createsuperuser
    
    class Meta:
        verbose_name = 'User'
        verbose_name_plural = 'Users'
    
    def __str__(self):
        return self.email
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}".strip() or self.username
    
    def get_avatar_url(self):
        if self.avatar:
            return self.avatar.url
        return '/static/default_avatar.png'
    
    def update_last_active(self):
        from django.utils import timezone
        self.last_active = timezone.now()
        self.save(update_fields=['last_active'])

# settings.py
AUTH_USER_MODEL = 'accounts.User'
```

### AbstractBaseUser (Full Control)

```python
# ตัวอย่างที่ 4: AbstractBaseUser - สร้าง User model ตั้งแต่ต้น
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models

class CustomUserManager(BaseUserManager):
    """Manager สำหรับ CustomUser"""
    
    def create_user(self, email, password=None, **extra_fields):
        """สร้าง regular user"""
        if not email:
            raise ValueError('Email is required')
        
        email = self.normalize_email(email)
        extra_fields.setdefault('is_active', True)
        extra_fields.setdefault('is_staff', False)
        extra_fields.setdefault('is_superuser', False)
        
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user
    
    def create_superuser(self, email, password=None, **extra_fields):
        """สร้าง superuser"""
        extra_fields.setdefault('is_active', True)
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        
        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True')
        
        return self.create_user(email, password, **extra_fields)
    
    def active(self):
        return self.filter(is_active=True)

class CustomUser(AbstractBaseUser, PermissionsMixin):
    """
    Custom User model ที่ใช้ email แทน username
    และมี fields พิเศษเพิ่มเติม
    """
    
    # Core fields
    email = models.EmailField(unique=True, db_index=True)
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    
    # Profile fields
    phone = models.CharField(max_length=20, blank=True)
    avatar = models.ImageField(upload_to='avatars/%Y/%m/', blank=True, null=True)
    bio = models.TextField(blank=True, max_length=500)
    
    # Status fields
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    email_verified = models.BooleanField(default=False)
    
    # Subscription
    TIER_FREE = 'free'
    TIER_PREMIUM = 'premium'
    TIER_ENTERPRISE = 'enterprise'
    TIER_CHOICES = [
        (TIER_FREE, 'Free'),
        (TIER_PREMIUM, 'Premium'),
        (TIER_ENTERPRISE, 'Enterprise'),
    ]
    tier = models.CharField(max_length=20, choices=TIER_CHOICES, default=TIER_FREE)
    
    # Timestamps
    date_joined = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    last_login = models.DateTimeField(null=True, blank=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['first_name', 'last_name']
    
    objects = CustomUserManager()
    
    class Meta:
        db_table = 'users'
        verbose_name = 'User'
        verbose_name_plural = 'Users'
        ordering = ['-date_joined']
    
    def __str__(self):
        return self.email
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    def get_short_name(self):
        return self.first_name
    
    def has_premium_access(self):
        return self.tier in [self.TIER_PREMIUM, self.TIER_ENTERPRISE]
    
    def is_email_verified(self):
        return self.email_verified

# settings.py
AUTH_USER_MODEL = 'accounts.CustomUser'
```

### Admin Registration

```python
# ตัวอย่างที่ 5: Register Custom User ใน Admin
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

class CustomUserAdmin(UserAdmin):
    """Admin configuration สำหรับ Custom User"""
    
    list_display = ['email', 'username', 'first_name', 'last_name', 'is_active', 'email_verified', 'date_joined']
    list_filter = ['is_active', 'is_staff', 'email_verified', 'date_joined']
    search_fields = ['email', 'username', 'first_name', 'last_name']
    ordering = ['-date_joined']
    
    fieldsets = (
        (None, {'fields': ('email', 'username', 'password')}),
        ('Personal info', {'fields': ('first_name', 'last_name', 'bio', 'phone', 'birth_date', 'avatar')}),
        ('Permissions', {
            'fields': ('is_active', 'is_staff', 'is_superuser', 'email_verified', 'groups', 'user_permissions'),
        }),
        ('Important dates', {'fields': ('last_login', 'date_joined')}),
    )
    
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'username', 'password1', 'password2', 'first_name', 'last_name'),
        }),
    )

admin.site.register(User, CustomUserAdmin)
```

---

## 3. User Registration Flow

### Registration Serializer

```python
# ตัวอย่างที่ 6: Registration Serializer
from rest_framework import serializers
from django.contrib.auth import get_user_model
from django.contrib.auth.password_validation import validate_password

User = get_user_model()

class UserRegistrationSerializer(serializers.ModelSerializer):
    password = serializers.CharField(
        min_length=8,
        write_only=True,
        required=True,
        style={'input_type': 'password'}
    )
    password_confirm = serializers.CharField(
        write_only=True,
        required=True,
        style={'input_type': 'password'}
    )
    
    class Meta:
        model = User
        fields = [
            'email', 'username', 'password', 'password_confirm',
            'first_name', 'last_name', 'phone'
        ]
        extra_kwargs = {
            'first_name': {'required': True},
            'last_name': {'required': True},
        }
    
    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email already registered")
        return value.lower()
    
    def validate_username(self, value):
        if User.objects.filter(username=value).exists():
            raise serializers.ValidationError("Username already taken")
        
        # ตรวจสอบ reserved usernames
        reserved = ['admin', 'root', 'api', 'www', 'mail', 'support']
        if value.lower() in reserved:
            raise serializers.ValidationError("This username is reserved")
        
        return value
    
    def validate_password(self, value):
        # ใช้ Django's password validators
        try:
            validate_password(value)
        except Exception as e:
            raise serializers.ValidationError(list(e.messages))
        return value
    
    def validate(self, data):
        if data['password'] != data['password_confirm']:
            raise serializers.ValidationError({
                'password_confirm': "Passwords don't match"
            })
        return data
    
    def create(self, validated_data):
        validated_data.pop('password_confirm')
        password = validated_data.pop('password')
        
        user = User(**validated_data)
        user.set_password(password)
        user.is_active = False  # ต้อง verify email ก่อน
        user.save()
        
        return user
```

### Registration View

```python
# ตัวอย่างที่ 7: Registration View
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import AllowAny
from django.core.mail import send_mail
from django.template.loader import render_to_string
from django.utils.http import urlsafe_base64_encode
from django.utils.encoding import force_bytes
from django.contrib.auth.tokens import default_token_generator

class RegisterView(APIView):
    permission_classes = [AllowAny]
    serializer_class = UserRegistrationSerializer
    
    def post(self, request):
        serializer = UserRegistrationSerializer(data=request.data)
        
        if not serializer.is_valid():
            return Response(
                serializer.errors,
                status=status.HTTP_400_BAD_REQUEST
            )
        
        user = serializer.save()
        
        # ส่ง verification email
        self.send_verification_email(user, request)
        
        return Response({
            'message': 'Registration successful. Please check your email to verify your account.',
            'email': user.email
        }, status=status.HTTP_201_CREATED)
    
    def send_verification_email(self, user, request):
        """ส่ง email สำหรับ verify account"""
        token = default_token_generator.make_token(user)
        uid = urlsafe_base64_encode(force_bytes(user.pk))
        
        verify_url = f"{request.scheme}://{request.get_host()}/api/auth/verify-email/{uid}/{token}/"
        
        context = {
            'user': user,
            'verify_url': verify_url,
        }
        
        html_message = render_to_string('emails/verify_email.html', context)
        
        send_mail(
            subject='Verify your email address',
            message=f'Click the link to verify: {verify_url}',
            from_email='noreply@example.com',
            recipient_list=[user.email],
            html_message=html_message,
        )
```

---

## 4. Login/Logout

### Login View

```python
# ตัวอย่างที่ 8: Custom Login View
from django.contrib.auth import authenticate, login, logout
from rest_framework_simplejwt.tokens import RefreshToken

class LoginSerializer(serializers.Serializer):
    email = serializers.EmailField()
    password = serializers.CharField(write_only=True)
    remember_me = serializers.BooleanField(default=False)
    
    def validate(self, data):
        email = data.get('email')
        password = data.get('password')
        
        if email and password:
            user = authenticate(
                request=self.context.get('request'),
                username=email,  # EmailBackend ใช้ email
                password=password
            )
            
            if not user:
                raise serializers.ValidationError('Invalid email or password')
            
            if not user.is_active:
                raise serializers.ValidationError('Account is disabled')
            
            if not user.email_verified:
                raise serializers.ValidationError('Please verify your email first')
        else:
            raise serializers.ValidationError('Must provide email and password')
        
        data['user'] = user
        return data

class LoginView(APIView):
    permission_classes = [AllowAny]
    
    def post(self, request):
        serializer = LoginSerializer(
            data=request.data,
            context={'request': request}
        )
        
        if not serializer.is_valid():
            return Response(
                serializer.errors,
                status=status.HTTP_400_BAD_REQUEST
            )
        
        user = serializer.validated_data['user']
        remember_me = serializer.validated_data.get('remember_me', False)
        
        # สร้าง JWT tokens
        refresh = RefreshToken.for_user(user)
        access = refresh.access_token
        
        # ถ้า remember_me ให้ extend token lifetime
        if remember_me:
            from datetime import timedelta
            refresh.set_exp(lifetime=timedelta(days=30))
        
        # Update last login
        user.update_last_active()
        
        # Session login (optional)
        login(request, user)
        
        return Response({
            'access': str(access),
            'refresh': str(refresh),
            'user': {
                'id': user.id,
                'email': user.email,
                'username': user.username,
                'full_name': user.full_name,
                'is_staff': user.is_staff,
            }
        })

class LogoutView(APIView):
    """Blacklist refresh token เมื่อ logout"""
    
    def post(self, request):
        try:
            refresh_token = request.data.get('refresh')
            if refresh_token:
                token = RefreshToken(refresh_token)
                token.blacklist()
            
            logout(request)
            
            return Response({'message': 'Logged out successfully'})
        except Exception:
            return Response(
                {'error': 'Invalid token'},
                status=status.HTTP_400_BAD_REQUEST
            )
```

### Login Rate Limiting

```python
# ตัวอย่างที่ 9: Rate limiting สำหรับ Login
from rest_framework.throttling import AnonRateThrottle
from django.core.cache import cache
import hashlib

class LoginRateThrottle(AnonRateThrottle):
    """Rate limit การ login attempt"""
    rate = '5/15min'
    scope = 'login'
    
    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            return None
        
        # Throttle ตาม IP และ email
        email = request.data.get('email', '')
        ip = self.get_ident(request)
        
        key_email = hashlib.md5(f"login_email_{email}".encode()).hexdigest()
        key_ip = hashlib.md5(f"login_ip_{ip}".encode()).hexdigest()
        
        # Check ทั้ง email และ IP
        return f"throttle_login_{key_ip}"

class LoginView(APIView):
    permission_classes = [AllowAny]
    throttle_classes = [LoginRateThrottle]
    
    def post(self, request):
        email = request.data.get('email', '')
        
        # Check failed attempts
        cache_key = f"login_failed_{email}"
        failed_attempts = cache.get(cache_key, 0)
        
        if failed_attempts >= 5:
            return Response(
                {'error': 'Too many failed attempts. Please wait 15 minutes.'},
                status=status.HTTP_429_TOO_MANY_REQUESTS
            )
        
        serializer = LoginSerializer(data=request.data, context={'request': request})
        
        if not serializer.is_valid():
            # เพิ่ม failed attempts
            cache.set(cache_key, failed_attempts + 1, 900)  # 15 minutes
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
        # Login สำเร็จ - reset failed attempts
        cache.delete(cache_key)
        
        user = serializer.validated_data['user']
        refresh = RefreshToken.for_user(user)
        
        return Response({
            'access': str(refresh.access_token),
            'refresh': str(refresh),
        })
```

---

## 5. Password Reset Flow

### Password Reset Serializers

```python
# ตัวอย่างที่ 10: Password Reset Flow
class PasswordResetRequestSerializer(serializers.Serializer):
    email = serializers.EmailField()
    
    def validate_email(self, value):
        # ไม่บอก user ว่า email มีอยู่หรือไม่ (security best practice)
        return value.lower()

class PasswordResetConfirmSerializer(serializers.Serializer):
    uid = serializers.CharField()
    token = serializers.CharField()
    new_password = serializers.CharField(min_length=8, write_only=True)
    new_password_confirm = serializers.CharField(min_length=8, write_only=True)
    
    def validate(self, data):
        if data['new_password'] != data['new_password_confirm']:
            raise serializers.ValidationError({
                'new_password_confirm': "Passwords don't match"
            })
        
        # Validate token
        try:
            from django.utils.http import urlsafe_base64_decode
            from django.utils.encoding import force_str
            
            uid = force_str(urlsafe_base64_decode(data['uid']))
            user = User.objects.get(pk=uid)
        except (TypeError, ValueError, OverflowError, User.DoesNotExist):
            raise serializers.ValidationError({'uid': 'Invalid user ID'})
        
        if not default_token_generator.check_token(user, data['token']):
            raise serializers.ValidationError({'token': 'Invalid or expired token'})
        
        # Validate password
        try:
            validate_password(data['new_password'], user)
        except Exception as e:
            raise serializers.ValidationError({'new_password': list(e.messages)})
        
        data['user'] = user
        return data

class PasswordChangeSerializer(serializers.Serializer):
    old_password = serializers.CharField(write_only=True)
    new_password = serializers.CharField(min_length=8, write_only=True)
    new_password_confirm = serializers.CharField(min_length=8, write_only=True)
    
    def validate_old_password(self, value):
        user = self.context['request'].user
        if not user.check_password(value):
            raise serializers.ValidationError("Incorrect current password")
        return value
    
    def validate(self, data):
        if data['new_password'] != data['new_password_confirm']:
            raise serializers.ValidationError({
                'new_password_confirm': "Passwords don't match"
            })
        
        try:
            validate_password(data['new_password'], self.context['request'].user)
        except Exception as e:
            raise serializers.ValidationError({'new_password': list(e.messages)})
        
        return data
```

### Password Reset Views

```python
# ตัวอย่างที่ 11: Password Reset Views
class PasswordResetRequestView(APIView):
    """ขอ reset password - ส่ง email"""
    permission_classes = [AllowAny]
    
    def post(self, request):
        serializer = PasswordResetRequestSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        email = serializer.validated_data['email']
        
        try:
            user = User.objects.get(email=email, is_active=True)
            self.send_reset_email(user, request)
        except User.DoesNotExist:
            pass  # ไม่บอก user ว่า email ไม่มี
        
        # ตอบ success เสมอ (ป้องกัน email enumeration)
        return Response({
            'message': 'If an account exists with this email, you will receive a password reset link.'
        })
    
    def send_reset_email(self, user, request):
        token = default_token_generator.make_token(user)
        uid = urlsafe_base64_encode(force_bytes(user.pk))
        
        reset_url = f"http://frontend.com/reset-password/{uid}/{token}/"
        
        send_mail(
            subject='Password Reset Request',
            message=f'Click the link to reset your password: {reset_url}',
            from_email='noreply@example.com',
            recipient_list=[user.email],
        )

class PasswordResetConfirmView(APIView):
    """ยืนยัน reset password ด้วย token"""
    permission_classes = [AllowAny]
    
    def post(self, request):
        serializer = PasswordResetConfirmSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        user = serializer.validated_data['user']
        user.set_password(serializer.validated_data['new_password'])
        user.save()
        
        # Invalidate all tokens (optional)
        # from rest_framework_simplejwt.token_blacklist.models import OutstandingToken
        # OutstandingToken.objects.filter(user=user).delete()
        
        return Response({'message': 'Password reset successfully'})

class PasswordChangeView(APIView):
    """เปลี่ยน password ขณะ login"""
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        serializer = PasswordChangeSerializer(
            data=request.data,
            context={'request': request}
        )
        serializer.is_valid(raise_exception=True)
        
        user = request.user
        user.set_password(serializer.validated_data['new_password'])
        user.save()
        
        # Generate new tokens
        refresh = RefreshToken.for_user(user)
        
        return Response({
            'message': 'Password changed successfully',
            'access': str(refresh.access_token),
            'refresh': str(refresh),
        })
```

---

## 6. Email Verification

```python
# ตัวอย่างที่ 12: Email Verification
from django.utils.http import urlsafe_base64_decode
from django.utils.encoding import force_str
from django.contrib.auth.tokens import default_token_generator

class EmailVerificationView(APIView):
    """ยืนยัน email ด้วย token"""
    permission_classes = [AllowAny]
    
    def get(self, request, uid, token):
        try:
            user_id = force_str(urlsafe_base64_decode(uid))
            user = User.objects.get(pk=user_id)
        except (TypeError, ValueError, OverflowError, User.DoesNotExist):
            return Response(
                {'error': 'Invalid verification link'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        if user.email_verified:
            return Response({'message': 'Email already verified'})
        
        if not default_token_generator.check_token(user, token):
            return Response(
                {'error': 'Verification link has expired'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        user.email_verified = True
        user.is_active = True
        user.save()
        
        return Response({'message': 'Email verified successfully! You can now login.'})

class ResendVerificationEmailView(APIView):
    """ส่ง verification email ใหม่"""
    permission_classes = [AllowAny]
    
    def post(self, request):
        email = request.data.get('email')
        
        if not email:
            return Response(
                {'error': 'Email is required'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        try:
            user = User.objects.get(email=email)
            
            if user.email_verified:
                return Response({'message': 'Email already verified'})
            
            if not user.is_active:
                token = default_token_generator.make_token(user)
                uid = urlsafe_base64_encode(force_bytes(user.pk))
                verify_url = f"http://frontend.com/verify-email/{uid}/{token}/"
                
                send_mail(
                    subject='Verify Your Email',
                    message=f'Click to verify: {verify_url}',
                    from_email='noreply@example.com',
                    recipient_list=[email],
                )
        except User.DoesNotExist:
            pass  # ไม่บอกว่า email ไม่มี
        
        return Response({
            'message': 'If your email is registered, you will receive a verification link.'
        })
```

### Custom Token for Email Verification

```python
# ตัวอย่างที่ 13: Custom Token Generator
from django.contrib.auth.tokens import PasswordResetTokenGenerator
import hashlib

class EmailVerificationTokenGenerator(PasswordResetTokenGenerator):
    """Token generator สำหรับ email verification"""
    
    def _make_hash_value(self, user, timestamp):
        # รวม user data ที่ unique
        return (
            str(user.pk) +
            str(timestamp) +
            str(user.email_verified) +  # Token invalid หลัง verify
            str(user.email)             # Token invalid ถ้า email เปลี่ยน
        )

email_verification_token = EmailVerificationTokenGenerator()
```

---

## 7. django-allauth

```bash
pip install django-allauth
```

### Setup

```python
# ตัวอย่างที่ 14: django-allauth Setup
# settings.py
INSTALLED_APPS = [
    'django.contrib.sites',
    
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.github',
    'allauth.socialaccount.providers.facebook',
]

SITE_ID = 1

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]

# allauth settings
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_UNIQUE_EMAIL = True
ACCOUNT_USERNAME_REQUIRED = True
ACCOUNT_EMAIL_VERIFICATION = 'mandatory'  # 'optional', 'none'
ACCOUNT_AUTHENTICATION_METHOD = 'email'  # 'username', 'email', 'username_email'
ACCOUNT_LOGIN_ATTEMPTS_LIMIT = 5
ACCOUNT_LOGIN_ATTEMPTS_TIMEOUT = 300  # 5 minutes
ACCOUNT_SESSION_REMEMBER = True
ACCOUNT_PASSWORD_MIN_LENGTH = 8

SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'SCOPE': ['profile', 'email'],
        'AUTH_PARAMS': {'access_type': 'online'},
        'OAUTH_PKCE_ENABLED': True,
    },
    'github': {
        'SCOPE': ['user:email'],
    },
}

# URLs
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/'
```

```python
# urls.py
urlpatterns = [
    path('accounts/', include('allauth.urls')),
    # หรือสำหรับ API only
    path('auth/', include('dj_rest_auth.urls')),
    path('auth/registration/', include('dj_rest_auth.registration.urls')),
]
```

### Custom Adapter

```python
# ตัวอย่างที่ 15: Custom allauth Adapter
from allauth.account.adapter import DefaultAccountAdapter
from allauth.socialaccount.adapter import DefaultSocialAccountAdapter

class CustomAccountAdapter(DefaultAccountAdapter):
    """Customize registration process"""
    
    def is_open_for_signup(self, request):
        """ควบคุมว่า open registration หรือไม่"""
        return True  # หรือดึงจาก settings
    
    def save_user(self, request, user, form, commit=True):
        """Custom user saving"""
        user = super().save_user(request, user, form, commit=False)
        
        # เพิ่ม logic เพิ่มเติม
        data = form.cleaned_data
        user.phone = data.get('phone', '')
        
        if commit:
            user.save()
        
        return user
    
    def send_confirmation_mail(self, request, emailconfirmation, signup):
        """Custom email confirmation"""
        context = {
            'user': emailconfirmation.email_address.user,
            'activate_url': self.get_email_confirmation_url(request, emailconfirmation),
        }
        
        # ส่ง email ด้วย template ของเราเอง
        from django.core.mail import send_mail
        from django.template.loader import render_to_string
        
        html = render_to_string('emails/confirm_email.html', context)
        send_mail(
            subject='Confirm your email',
            message='',
            from_email='noreply@example.com',
            recipient_list=[emailconfirmation.email_address.email],
            html_message=html,
        )

class CustomSocialAccountAdapter(DefaultSocialAccountAdapter):
    """Customize social authentication"""
    
    def populate_user(self, request, sociallogin, data):
        """Populate user data จาก social account"""
        user = super().populate_user(request, sociallogin, data)
        
        # เพิ่ม custom data
        if sociallogin.account.provider == 'google':
            user.email_verified = True
        
        return user
    
    def is_auto_signup_allowed(self, request, sociallogin):
        """อนุญาต auto signup สำหรับ social login"""
        return True

# settings.py
ACCOUNT_ADAPTER = 'accounts.adapters.CustomAccountAdapter'
SOCIALACCOUNT_ADAPTER = 'accounts.adapters.CustomSocialAccountAdapter'
```

---

## 8. Social Authentication

```python
# ตัวอย่างที่ 16: Social Authentication ด้วย JWT
# pip install dj-rest-auth

# settings.py
INSTALLED_APPS += [
    'rest_framework.authtoken',
    'dj_rest_auth',
    'dj_rest_auth.registration',
]

REST_AUTH = {
    'USE_JWT': True,
    'JWT_AUTH_COOKIE': 'auth-token',
    'JWT_AUTH_REFRESH_COOKIE': 'refresh-token',
    'JWT_AUTH_RETURN_EXPIRATION': True,
}
```

```python
# ตัวอย่างที่ 17: Custom Social Login View
from allauth.socialaccount.providers.google.views import GoogleOAuth2Adapter
from allauth.socialaccount.providers.github.views import GitHubOAuth2Adapter
from allauth.socialaccount.providers.oauth2.client import OAuth2Client
from dj_rest_auth.registration.views import SocialLoginView

class GoogleLoginView(SocialLoginView):
    """Login ด้วย Google"""
    adapter_class = GoogleOAuth2Adapter
    callback_url = 'http://localhost:3000/auth/google/callback'
    client_class = OAuth2Client
    
    def get_response(self):
        response = super().get_response()
        
        # เพิ่ม custom data ใน response
        if response.status_code == 200:
            user = self.user
            response.data['user_info'] = {
                'id': user.id,
                'email': user.email,
                'full_name': user.get_full_name(),
                'is_new_user': getattr(user, '_is_new', False),
            }
        
        return response

class GitHubLoginView(SocialLoginView):
    """Login ด้วย GitHub"""
    adapter_class = GitHubOAuth2Adapter
    callback_url = 'http://localhost:3000/auth/github/callback'
    client_class = OAuth2Client

# urls.py
urlpatterns = [
    path('auth/google/', GoogleLoginView.as_view(), name='google_login'),
    path('auth/github/', GitHubLoginView.as_view(), name='github_login'),
]
```

---

## 9. Permissions System

### Django Model Permissions

```python
# ตัวอย่างที่ 18: Model Permissions
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    
    class Meta:
        # Django จะสร้าง permissions อัตโนมัติ:
        # - view_article
        # - add_article
        # - change_article
        # - delete_article
        
        # สร้าง custom permissions เพิ่มเติม
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature articles'),
            ('moderate_article', 'Can moderate articles'),
        ]

# การตรวจสอบ permissions
user = request.user

# Model permissions
user.has_perm('api.add_article')      # django.contrib.auth จัดการ
user.has_perm('api.change_article')
user.has_perm('api.delete_article')
user.has_perm('api.view_article')

# Custom permissions
user.has_perm('api.publish_article')
user.has_perm('api.feature_article')

# Multiple permissions
user.has_perms(['api.add_article', 'api.change_article'])

# Module permissions
user.has_module_perms('api')
```

### Custom Permission Checks

```python
# ตัวอย่างที่ 19: Custom Permission Checks
from rest_framework.permissions import BasePermission, SAFE_METHODS

class ArticlePermission(BasePermission):
    """
    - GET: ทุกคน
    - POST: ต้อง login และมี add_article permission
    - PUT/PATCH: เจ้าของหรือ editor
    - DELETE: เจ้าของหรือ admin
    """
    
    def has_permission(self, request, view):
        # Safe methods (GET, HEAD, OPTIONS) - ทุกคน
        if request.method in SAFE_METHODS:
            return True
        
        # ต้อง authenticated
        if not request.user.is_authenticated:
            return False
        
        # ตรวจสอบ permission ตาม method
        if request.method == 'POST':
            return request.user.has_perm('api.add_article')
        
        return True  # Object-level permission จะ handle ที่ has_object_permission
    
    def has_object_permission(self, request, view, obj):
        # Safe methods - ทุกคน
        if request.method in SAFE_METHODS:
            return True
        
        # Admin access ทุกอย่าง
        if request.user.is_superuser:
            return True
        
        # เจ้าของ article
        if obj.author == request.user:
            return True
        
        # Editor สามารถแก้ไขได้
        if request.method in ['PUT', 'PATCH']:
            return request.user.has_perm('api.change_article')
        
        # Admin เท่านั้นที่ลบได้
        if request.method == 'DELETE':
            return request.user.is_staff
        
        return False

class IsEmailVerified(BasePermission):
    """ต้อง verify email"""
    message = {'error': 'Please verify your email address'}
    
    def has_permission(self, request, view):
        return (
            request.user.is_authenticated and
            request.user.email_verified
        )

class HasSubscription(BasePermission):
    """ต้องมี active subscription"""
    message = {'error': 'This feature requires a premium subscription'}
    
    def has_permission(self, request, view):
        if not request.user.is_authenticated:
            return False
        
        return (
            request.user.is_staff or  # Admin bypass
            hasattr(request.user, 'subscription') and 
            request.user.subscription.is_active()
        )
```

---

## 10. Group-based Permissions

```python
# ตัวอย่างที่ 20: Group-based Permissions
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType

# สร้าง groups และ permissions
def create_groups():
    """สร้าง groups สำหรับ application"""
    
    # Content type
    article_ct = ContentType.objects.get_for_model(Article)
    comment_ct = ContentType.objects.get_for_model(Comment)
    
    # Permissions
    perms = {
        'add_article': Permission.objects.get(codename='add_article', content_type=article_ct),
        'change_article': Permission.objects.get(codename='change_article', content_type=article_ct),
        'delete_article': Permission.objects.get(codename='delete_article', content_type=article_ct),
        'publish_article': Permission.objects.get(codename='publish_article', content_type=article_ct),
    }
    
    # Writer group - เขียน draft ได้
    writers, _ = Group.objects.get_or_create(name='Writers')
    writers.permissions.set([perms['add_article'], perms['change_article']])
    
    # Editor group - แก้ไขและ publish ได้
    editors, _ = Group.objects.get_or_create(name='Editors')
    editors.permissions.set([
        perms['add_article'],
        perms['change_article'],
        perms['publish_article'],
    ])
    
    # Moderator group - ดูแล comments
    moderators, _ = Group.objects.get_or_create(name='Moderators')
    # ... add comment permissions
    
    return writers, editors, moderators

# Management command
# python manage.py create_groups
```

```python
# ตัวอย่างที่ 21: Group-based Permission Mixin
class GroupRequiredMixin:
    """Mixin ที่ require group membership"""
    group_required = []  # list of group names
    
    def has_permission(self, request, view):
        if not request.user.is_authenticated:
            return False
        
        if request.user.is_superuser:
            return True
        
        user_groups = request.user.groups.values_list('name', flat=True)
        
        return any(group in user_groups for group in self.group_required)

class WriterPermission(GroupRequiredMixin, BasePermission):
    group_required = ['Writers', 'Editors', 'Admins']
    message = "You must be a Writer to perform this action"

class EditorPermission(GroupRequiredMixin, BasePermission):
    group_required = ['Editors', 'Admins']
    message = "You must be an Editor to perform this action"

# ใช้งาน
class ArticleViewSet(viewsets.ModelViewSet):
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]
        elif self.action == 'create':
            return [WriterPermission()]
        elif self.action in ['update', 'partial_update']:
            return [EditorPermission()]
        elif self.action == 'destroy':
            return [IsAdminUser()]
        elif self.action == 'publish':
            return [EditorPermission()]
        return [IsAuthenticated()]
```

### Assigning Groups

```python
# ตัวอย่างที่ 22: Assigning Groups to Users
from django.contrib.auth.models import Group

# เพิ่ม user เข้า group
user = User.objects.get(username='john')
editor_group = Group.objects.get(name='Editors')
user.groups.add(editor_group)

# ลบ user ออกจาก group
user.groups.remove(editor_group)

# กำหนด groups ใหม่
user.groups.set([editor_group])

# ตรวจสอบ group membership
user.groups.filter(name='Editors').exists()

# ตรวจสอบ permission ผ่าน group
user.has_perm('api.publish_article')  # True ถ้า group มี permission นี้

# API endpoint สำหรับจัดการ groups
class UserGroupView(APIView):
    permission_classes = [IsAdminUser]
    
    def post(self, request, user_id):
        """เพิ่ม user เข้า group"""
        user = get_object_or_404(User, pk=user_id)
        group_name = request.data.get('group')
        
        try:
            group = Group.objects.get(name=group_name)
            user.groups.add(group)
            return Response({'message': f'User added to {group_name}'})
        except Group.DoesNotExist:
            return Response(
                {'error': f'Group {group_name} not found'},
                status=status.HTTP_404_NOT_FOUND
            )
    
    def delete(self, request, user_id):
        """ลบ user ออกจาก group"""
        user = get_object_or_404(User, pk=user_id)
        group_name = request.data.get('group')
        
        try:
            group = Group.objects.get(name=group_name)
            user.groups.remove(group)
            return Response({'message': f'User removed from {group_name}'})
        except Group.DoesNotExist:
            return Response(
                {'error': f'Group {group_name} not found'},
                status=status.HTTP_404_NOT_FOUND
            )
```

---

## 11. Object-level Permissions

### django-guardian

```bash
pip install django-guardian
```

```python
# ตัวอย่างที่ 23: Object-level Permissions ด้วย django-guardian
# settings.py
INSTALLED_APPS += ['guardian']

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'guardian.backends.ObjectPermissionBackend',
]

ANONYMOUS_USER_NAME = None  # Optional

# ใช้งาน
from guardian.shortcuts import (
    assign_perm,
    remove_perm,
    get_perms,
    get_objects_for_user,
    get_users_with_perms,
)

# กำหนด permission สำหรับ specific object
article = Article.objects.get(id=1)
user = User.objects.get(username='john')

# Assign object-level permission
assign_perm('change_article', user, article)  # user สามารถ change article นี้ได้
assign_perm('delete_article', user, article)

# กำหนด permission สำหรับ group
from django.contrib.auth.models import Group
editors = Group.objects.get(name='Editors')
assign_perm('change_article', editors, article)

# ตรวจสอบ object-level permission
user.has_perm('api.change_article', article)  # True

# ดู permissions ทั้งหมดของ user สำหรับ object
get_perms(user, article)  # ['change_article', 'delete_article']

# ดู users ที่มี permission สำหรับ object
get_users_with_perms(article)

# ดู objects ที่ user มี permission
articles_can_change = get_objects_for_user(user, 'api.change_article', Article)
```

```python
# ตัวอย่างที่ 24: DRF Permission Class ด้วย guardian
from guardian.shortcuts import get_perms
from rest_framework.permissions import BasePermission

class ObjectPermissionChecker(BasePermission):
    """Check object-level permissions ด้วย guardian"""
    
    perms_map = {
        'GET': [],
        'OPTIONS': [],
        'HEAD': [],
        'POST': ['%(app_label)s.add_%(model_name)s'],
        'PUT': ['%(app_label)s.change_%(model_name)s'],
        'PATCH': ['%(app_label)s.change_%(model_name)s'],
        'DELETE': ['%(app_label)s.delete_%(model_name)s'],
    }
    
    def get_required_object_perms(self, method, obj):
        """คำนวณ required permissions ตาม method และ model"""
        kwargs = {
            'app_label': obj._meta.app_label,
            'model_name': obj._meta.model_name,
        }
        
        if method not in self.perms_map:
            raise Exception(f'Unexpected HTTP method: {method}')
        
        return [perm % kwargs for perm in self.perms_map[method]]
    
    def has_object_permission(self, request, view, obj):
        required_perms = self.get_required_object_perms(request.method, obj)
        
        if not required_perms:
            return True
        
        user = request.user
        
        # Superuser bypass
        if user.is_superuser:
            return True
        
        # Check model-level permissions first
        if not user.has_perms(required_perms):
            # Check object-level permissions
            obj_perms = get_perms(user, obj)
            return all(
                perm.split('.')[1] in obj_perms 
                for perm in required_perms
            )
        
        return True

# ใช้ใน view
class ArticleViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, ObjectPermissionChecker]
    
    def perform_create(self, serializer):
        article = serializer.save(author=self.request.user)
        
        # ให้ author มี full permissions สำหรับ article ที่สร้าง
        assign_perm('change_article', self.request.user, article)
        assign_perm('delete_article', self.request.user, article)
        
        return article
```

### Signals สำหรับ Auto-assign Permissions

```python
# ตัวอย่างที่ 25: Signals for Auto-assign Permissions
from django.db.models.signals import post_save
from django.dispatch import receiver
from guardian.shortcuts import assign_perm

@receiver(post_save, sender=Article)
def assign_article_permissions(sender, instance, created, **kwargs):
    """Auto-assign permissions เมื่อสร้าง article"""
    if created:
        # Author ได้ full permissions
        assign_perm('api.view_article', instance.author, instance)
        assign_perm('api.change_article', instance.author, instance)
        assign_perm('api.delete_article', instance.author, instance)
        assign_perm('api.publish_article', instance.author, instance)

@receiver(post_save, sender=User)
def setup_user_permissions(sender, instance, created, **kwargs):
    """Setup default permissions สำหรับ new user"""
    if created:
        # เพิ่ม user เข้า default group
        default_group, _ = Group.objects.get_or_create(name='Users')
        instance.groups.add(default_group)
```

### Permission Utilities

```python
# ตัวอย่างที่ 26: Permission Utilities
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

class PermissionManager:
    """Helper class สำหรับจัดการ permissions"""
    
    @staticmethod
    def get_model_permissions(model_class):
        """ดู permissions ทั้งหมดของ model"""
        ct = ContentType.objects.get_for_model(model_class)
        return Permission.objects.filter(content_type=ct)
    
    @staticmethod
    def grant_permission(user_or_group, permission_codename, app_label):
        """ให้ permission แก่ user หรือ group"""
        try:
            permission = Permission.objects.get(
                codename=permission_codename,
                content_type__app_label=app_label
            )
            user_or_group.user_permissions.add(permission)
        except Permission.DoesNotExist:
            raise ValueError(f"Permission {app_label}.{permission_codename} not found")
    
    @staticmethod
    def revoke_permission(user_or_group, permission_codename, app_label):
        """ถอน permission จาก user หรือ group"""
        try:
            permission = Permission.objects.get(
                codename=permission_codename,
                content_type__app_label=app_label
            )
            user_or_group.user_permissions.remove(permission)
        except Permission.DoesNotExist:
            pass
    
    @staticmethod
    def list_user_permissions(user):
        """List permissions ทั้งหมดของ user"""
        return {
            'user_permissions': list(
                user.user_permissions.values_list('codename', flat=True)
            ),
            'group_permissions': list(
                Permission.objects.filter(
                    group__user=user
                ).values_list('codename', flat=True).distinct()
            ),
            'all_permissions': list(user.get_all_permissions()),
        }
```

### Complete Auth System Example

```python
# ตัวอย่างที่ 27: Complete Authentication Flow
# accounts/views.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework_simplejwt.tokens import RefreshToken

class AuthViewSet(viewsets.ViewSet):
    """ViewSet ที่รวม authentication endpoints"""
    
    @action(detail=False, methods=['post'], permission_classes=[AllowAny])
    def register(self, request):
        """POST /auth/register/"""
        serializer = UserRegistrationSerializer(data=request.data)
        if serializer.is_valid():
            user = serializer.save()
            send_verification_email(user, request)
            return Response({
                'message': 'Registration successful. Check your email.',
                'email': user.email
            }, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['post'], permission_classes=[AllowAny])
    def login(self, request):
        """POST /auth/login/"""
        serializer = LoginSerializer(
            data=request.data, context={'request': request}
        )
        if serializer.is_valid():
            user = serializer.validated_data['user']
            refresh = RefreshToken.for_user(user)
            return Response({
                'access': str(refresh.access_token),
                'refresh': str(refresh),
                'user': UserSerializer(user).data
            })
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['post'], permission_classes=[IsAuthenticated])
    def logout(self, request):
        """POST /auth/logout/"""
        try:
            refresh = RefreshToken(request.data.get('refresh'))
            refresh.blacklist()
        except Exception:
            pass
        return Response({'message': 'Logged out'})
    
    @action(detail=False, methods=['post'], permission_classes=[AllowAny])
    def request_password_reset(self, request):
        """POST /auth/request-password-reset/"""
        serializer = PasswordResetRequestSerializer(data=request.data)
        if serializer.is_valid():
            send_password_reset_email(serializer.validated_data['email'])
        return Response({'message': 'If email exists, reset link sent.'})
    
    @action(detail=False, methods=['post'], permission_classes=[AllowAny])
    def reset_password(self, request):
        """POST /auth/reset-password/"""
        serializer = PasswordResetConfirmSerializer(data=request.data)
        if serializer.is_valid():
            user = serializer.validated_data['user']
            user.set_password(serializer.validated_data['new_password'])
            user.save()
            return Response({'message': 'Password reset successful'})
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def profile(self, request):
        """GET /auth/profile/"""
        serializer = UserSerializer(request.user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['put', 'patch'], permission_classes=[IsAuthenticated])
    def update_profile(self, request):
        """PUT/PATCH /auth/update-profile/"""
        serializer = UserProfileUpdateSerializer(
            request.user,
            data=request.data,
            partial=request.method == 'PATCH',
            context={'request': request}
        )
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['post'], permission_classes=[IsAuthenticated])
    def change_password(self, request):
        """POST /auth/change-password/"""
        serializer = PasswordChangeSerializer(
            data=request.data, context={'request': request}
        )
        if serializer.is_valid():
            user = request.user
            user.set_password(serializer.validated_data['new_password'])
            user.save()
            return Response({'message': 'Password changed successfully'})
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['post'], permission_classes=[IsAuthenticated])
    def deactivate(self, request):
        """POST /auth/deactivate/ - Soft delete account"""
        user = request.user
        user.is_active = False
        user.save()
        return Response({'message': 'Account deactivated'})
```

### Middleware สำหรับ Track User Activity

```python
# ตัวอย่างที่ 28: Activity Tracking Middleware
from django.utils import timezone
from django.utils.deprecation import MiddlewareMixin
from django.core.cache import cache

class UserActivityMiddleware(MiddlewareMixin):
    """Track user activity และ update last_active"""
    
    def process_request(self, request):
        if request.user.is_authenticated:
            user_id = request.user.id
            cache_key = f"user_active_{user_id}"
            
            # Update เฉพาะทุก 5 นาที
            if not cache.get(cache_key):
                # Async update เพื่อไม่ block request
                from django.db import connection
                connection.ensure_connection()
                
                request.user.__class__.objects.filter(
                    pk=user_id
                ).update(last_active=timezone.now())
                
                cache.set(cache_key, True, 300)  # 5 minutes
```

### JWT Middleware

```python
# ตัวอย่างที่ 29: JWT Token Auto-refresh Middleware
from rest_framework_simplejwt.tokens import AccessToken
from rest_framework_simplejwt.exceptions import TokenError
from django.conf import settings
from datetime import timedelta
import jwt

class JWTAutoRefreshMiddleware:
    """Auto-refresh JWT token ถ้าใกล้หมดอายุ"""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        # ตรวจสอบ token ที่กำลังจะหมดอายุ
        auth_header = request.META.get('HTTP_AUTHORIZATION', '')
        
        if auth_header.startswith('Bearer '):
            token_str = auth_header.split(' ')[1]
            
            try:
                token = AccessToken(token_str)
                exp = token.payload.get('exp')
                
                if exp:
                    from datetime import datetime
                    exp_datetime = datetime.fromtimestamp(exp)
                    now = datetime.now()
                    
                    # ถ้าเหลือ น้อยกว่า 5 นาที ให้ refresh
                    if (exp_datetime - now).total_seconds() < 300:
                        response['X-Token-Expiring'] = 'true'
            
            except TokenError:
                pass
        
        return response
```

---

## 12. แบบฝึกหัด

### ข้อ 1: Custom User Model with Profile

สร้าง Custom User model ที่มี:
- ใช้ email เป็น login
- มี profile ด้วย bio, avatar, social links
- Email verification

**เฉลย:**

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models
from django.utils.translation import gettext_lazy as _

class User(AbstractUser):
    email = models.EmailField(_('email address'), unique=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
    
    class Meta:
        swappable = 'AUTH_USER_MODEL'

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField(blank=True, max_length=500)
    avatar = models.ImageField(upload_to='avatars/%Y/%m/', blank=True, null=True)
    website = models.URLField(blank=True)
    twitter = models.CharField(max_length=100, blank=True)
    github = models.CharField(max_length=100, blank=True)
    linkedin = models.URLField(blank=True)
    location = models.CharField(max_length=100, blank=True)
    email_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Profile of {self.user.email}"
    
    @classmethod
    def get_or_create_for_user(cls, user):
        profile, _ = cls.objects.get_or_create(user=user)
        return profile

# Signals
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)
```

### ข้อ 2: Two-Factor Authentication

สร้าง 2FA ด้วย TOTP

**เฉลย:**

```python
# pip install pyotp qrcode
import pyotp
import qrcode
import io
import base64
from django.db import models

class TwoFactorAuth(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='two_factor')
    secret_key = models.CharField(max_length=32)
    is_enabled = models.BooleanField(default=False)
    backup_codes = models.JSONField(default=list)
    
    def generate_secret(self):
        self.secret_key = pyotp.random_base32()
        self.save()
        return self.secret_key
    
    def get_totp_uri(self):
        return pyotp.totp.TOTP(self.secret_key).provisioning_uri(
            name=self.user.email,
            issuer_name='MyApp'
        )
    
    def get_qr_code(self):
        uri = self.get_totp_uri()
        qr = qrcode.QRCode(version=1, box_size=10, border=5)
        qr.add_data(uri)
        qr.make(fit=True)
        
        img = qr.make_image(fill='black', back_color='white')
        buffer = io.BytesIO()
        img.save(buffer, format='PNG')
        
        return base64.b64encode(buffer.getvalue()).decode()
    
    def verify_token(self, token):
        totp = pyotp.TOTP(self.secret_key)
        return totp.verify(token, valid_window=1)
    
    def generate_backup_codes(self):
        import secrets
        codes = [secrets.token_hex(8) for _ in range(10)]
        self.backup_codes = [
            {'code': code, 'used': False} for code in codes
        ]
        self.save()
        return codes
    
    def use_backup_code(self, code):
        for bc in self.backup_codes:
            if bc['code'] == code and not bc['used']:
                bc['used'] = True
                self.save()
                return True
        return False

class TwoFactorSetupView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        """GET /2fa/setup/ - ดึง QR code"""
        two_factor, _ = TwoFactorAuth.objects.get_or_create(user=request.user)
        
        if not two_factor.secret_key:
            two_factor.generate_secret()
        
        return Response({
            'qr_code': two_factor.get_qr_code(),
            'secret_key': two_factor.secret_key,
            'is_enabled': two_factor.is_enabled,
        })
    
    def post(self, request):
        """POST /2fa/setup/ - enable 2FA"""
        token = request.data.get('token')
        
        try:
            two_factor = request.user.two_factor
        except TwoFactorAuth.DoesNotExist:
            return Response({'error': 'Setup 2FA first'}, status=400)
        
        if not two_factor.verify_token(token):
            return Response({'error': 'Invalid token'}, status=400)
        
        two_factor.is_enabled = True
        two_factor.save()
        
        backup_codes = two_factor.generate_backup_codes()
        
        return Response({
            'message': '2FA enabled successfully',
            'backup_codes': backup_codes
        })
```

### ข้อ 3: Role-based Access Control (RBAC)

สร้างระบบ RBAC ที่ flexible

**เฉลย:**

```python
from django.db import models

class Role(models.Model):
    name = models.CharField(max_length=100, unique=True)
    description = models.TextField(blank=True)
    permissions = models.ManyToManyField('auth.Permission', blank=True)
    parent = models.ForeignKey('self', on_delete=models.SET_NULL, null=True, blank=True)
    
    def __str__(self):
        return self.name
    
    def get_all_permissions(self):
        """ดึง permissions รวม parent roles"""
        perms = set(self.permissions.all())
        
        if self.parent:
            perms |= self.parent.get_all_permissions()
        
        return perms

class UserRole(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='user_roles')
    role = models.ForeignKey(Role, on_delete=models.CASCADE)
    resource = models.CharField(max_length=200, blank=True)  # Optional: scope to resource
    expires_at = models.DateTimeField(null=True, blank=True)
    assigned_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='assigned_roles')
    assigned_at = models.DateTimeField(auto_now_add=True)
    
    def is_active(self):
        if self.expires_at:
            from django.utils import timezone
            return timezone.now() < self.expires_at
        return True
    
    class Meta:
        unique_together = ['user', 'role', 'resource']

class RBACPermission(BasePermission):
    """Permission checker ที่ใช้ Role-based system"""
    
    def has_permission(self, request, view):
        if not request.user.is_authenticated:
            return False
        
        if request.user.is_superuser:
            return True
        
        # Check permissions จาก user roles
        required_perm = self.get_required_permission(request, view)
        if not required_perm:
            return True
        
        return self.user_has_permission(request.user, required_perm)
    
    def get_required_permission(self, request, view):
        """ดึง required permission จาก view"""
        return getattr(view, 'required_permission', None)
    
    def user_has_permission(self, user, permission):
        active_roles = UserRole.objects.filter(
            user=user
        ).select_related('role').filter(
            models.Q(expires_at__isnull=True) | models.Q(expires_at__gt=timezone.now())
        )
        
        for user_role in active_roles:
            role_perms = user_role.role.get_all_permissions()
            for perm in role_perms:
                if f"{perm.content_type.app_label}.{perm.codename}" == permission:
                    return True
        
        return False
```

### ข้อ 4: Session Management

สร้างระบบจัดการ sessions

**เฉลย:**

```python
from django.db import models
import uuid

class UserSession(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sessions')
    session_key = models.CharField(max_length=100, unique=True, default=uuid.uuid4)
    ip_address = models.GenericIPAddressField()
    user_agent = models.CharField(max_length=500)
    device_type = models.CharField(max_length=50, blank=True)
    location = models.CharField(max_length=200, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    last_active = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    revoked_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        ordering = ['-last_active']

class SessionManagementView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        """GET /sessions/ - ดู sessions ทั้งหมด"""
        sessions = UserSession.objects.filter(
            user=request.user,
            is_active=True
        )
        
        sessions_data = []
        for session in sessions:
            sessions_data.append({
                'id': str(session.session_key),
                'ip': session.ip_address,
                'device': session.user_agent[:50],
                'created': session.created_at,
                'last_active': session.last_active,
                'is_current': session.session_key == request.session.session_key,
            })
        
        return Response(sessions_data)
    
    def delete(self, request, session_key=None):
        """DELETE /sessions/{key}/ หรือ /sessions/all/"""
        if session_key == 'all':
            # Revoke ทุก sessions ยกเว้น current
            UserSession.objects.filter(
                user=request.user,
                is_active=True
            ).exclude(
                session_key=request.session.session_key
            ).update(is_active=False, revoked_at=timezone.now())
            return Response({'message': 'All other sessions revoked'})
        
        try:
            session = UserSession.objects.get(
                session_key=session_key,
                user=request.user
            )
            session.is_active = False
            session.revoked_at = timezone.now()
            session.save()
            return Response({'message': 'Session revoked'})
        except UserSession.DoesNotExist:
            return Response({'error': 'Session not found'}, status=404)
```

### ข้อ 5: API Key Management

สร้าง API Key system สำหรับ programmatic access

**เฉลย:**

```python
import secrets
import hashlib
from django.db import models

class APIKey(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='api_keys')
    name = models.CharField(max_length=100)
    key_hash = models.CharField(max_length=128, db_index=True)
    prefix = models.CharField(max_length=8, db_index=True)
    
    # Permissions
    scopes = models.JSONField(default=list)
    
    # Metadata
    created_at = models.DateTimeField(auto_now_add=True)
    last_used_at = models.DateTimeField(null=True, blank=True)
    expires_at = models.DateTimeField(null=True, blank=True)
    is_active = models.BooleanField(default=True)
    
    @classmethod
    def create_key(cls, user, name, scopes=None, expires_at=None):
        """สร้าง API key ใหม่"""
        # Generate key
        raw_key = f"ak_{secrets.token_urlsafe(32)}"
        prefix = raw_key[:8]
        
        # Hash สำหรับ store
        key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
        
        key_obj = cls.objects.create(
            user=user,
            name=name,
            key_hash=key_hash,
            prefix=prefix,
            scopes=scopes or [],
            expires_at=expires_at,
        )
        
        # Return raw key เฉพาะตอนสร้าง (ไม่เก็บ raw key)
        key_obj._raw_key = raw_key
        return key_obj
    
    @classmethod
    def verify_key(cls, raw_key):
        """Verify และ return key object"""
        key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
        
        try:
            key = cls.objects.select_related('user').get(
                key_hash=key_hash,
                is_active=True
            )
        except cls.DoesNotExist:
            return None
        
        if key.expires_at and key.expires_at < timezone.now():
            return None
        
        # Update last used
        key.last_used_at = timezone.now()
        key.save(update_fields=['last_used_at'])
        
        return key
    
    def has_scope(self, scope):
        return scope in self.scopes or '*' in self.scopes

class APIKeyView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        """List API keys"""
        keys = APIKey.objects.filter(user=request.user)
        data = [{
            'id': k.id,
            'name': k.name,
            'prefix': k.prefix,
            'scopes': k.scopes,
            'created_at': k.created_at,
            'last_used_at': k.last_used_at,
            'expires_at': k.expires_at,
            'is_active': k.is_active,
        } for k in keys]
        return Response(data)
    
    def post(self, request):
        """Create API key"""
        name = request.data.get('name', 'API Key')
        scopes = request.data.get('scopes', ['read'])
        
        key_obj = APIKey.create_key(request.user, name, scopes)
        
        return Response({
            'id': key_obj.id,
            'name': key_obj.name,
            'key': key_obj._raw_key,  # แสดงครั้งเดียว
            'scopes': key_obj.scopes,
            'message': 'Store this key securely. It will not be shown again.'
        }, status=status.HTTP_201_CREATED)
    
    def delete(self, request, key_id):
        """Revoke API key"""
        try:
            key = APIKey.objects.get(id=key_id, user=request.user)
            key.is_active = False
            key.save()
            return Response({'message': 'API key revoked'})
        except APIKey.DoesNotExist:
            return Response({'error': 'Key not found'}, status=404)
```

### ข้อ 6: Permission Decorator

สร้าง custom decorators สำหรับ permission checking

**เฉลย:**

```python
# accounts/decorators.py
from functools import wraps
from rest_framework.response import Response
from rest_framework import status

def require_permission(permission):
    """Decorator ที่ require specific permission"""
    def decorator(func):
        @wraps(func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return Response(
                    {'error': 'Authentication required'},
                    status=status.HTTP_401_UNAUTHORIZED
                )
            
            if not request.user.has_perm(permission):
                return Response(
                    {'error': f'Permission denied: {permission}'},
                    status=status.HTTP_403_FORBIDDEN
                )
            
            return func(request, *args, **kwargs)
        return wrapper
    return decorator

def require_group(*groups):
    """Decorator ที่ require group membership"""
    def decorator(func):
        @wraps(func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return Response({'error': 'Authentication required'}, status=401)
            
            if request.user.is_superuser:
                return func(request, *args, **kwargs)
            
            user_groups = request.user.groups.values_list('name', flat=True)
            if not any(g in user_groups for g in groups):
                return Response(
                    {'error': f'Must be member of: {", ".join(groups)}'},
                    status=403
                )
            
            return func(request, *args, **kwargs)
        return wrapper
    return decorator

def require_verified_email(func):
    """Decorator ที่ require verified email"""
    @wraps(func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            return Response({'error': 'Authentication required'}, status=401)
        
        if not request.user.email_verified:
            return Response(
                {'error': 'Email verification required'},
                status=status.HTTP_403_FORBIDDEN
            )
        
        return func(request, *args, **kwargs)
    return wrapper

# ใช้งาน
from rest_framework.decorators import api_view
from .decorators import require_permission, require_group, require_verified_email

@api_view(['POST'])
@require_verified_email
@require_permission('api.add_article')
def create_article(request):
    serializer = ArticleSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save(author=request.user)
        return Response(serializer.data, status=201)
    return Response(serializer.errors, status=400)
```

### ข้อ 7: Audit Log

สร้างระบบ Audit logging

**เฉลย:**

```python
from django.db import models
from django.contrib.contenttypes.models import ContentType
from django.contrib.contenttypes.fields import GenericForeignKey

class AuditLog(models.Model):
    ACTION_CREATE = 'create'
    ACTION_UPDATE = 'update'
    ACTION_DELETE = 'delete'
    ACTION_LOGIN = 'login'
    ACTION_LOGOUT = 'logout'
    ACTION_PERMISSION = 'permission'
    
    ACTION_CHOICES = [
        (ACTION_CREATE, 'Create'),
        (ACTION_UPDATE, 'Update'),
        (ACTION_DELETE, 'Delete'),
        (ACTION_LOGIN, 'Login'),
        (ACTION_LOGOUT, 'Logout'),
        (ACTION_PERMISSION, 'Permission Change'),
    ]
    
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    action = models.CharField(max_length=50, choices=ACTION_CHOICES)
    
    # Generic relation to any model
    content_type = models.ForeignKey(ContentType, on_delete=models.SET_NULL, null=True, blank=True)
    object_id = models.PositiveIntegerField(null=True, blank=True)
    content_object = GenericForeignKey('content_type', 'object_id')
    
    # Details
    changes = models.JSONField(default=dict)
    ip_address = models.GenericIPAddressField(null=True)
    user_agent = models.CharField(max_length=500, blank=True)
    
    timestamp = models.DateTimeField(auto_now_add=True, db_index=True)
    
    class Meta:
        ordering = ['-timestamp']
        indexes = [
            models.Index(fields=['user', 'timestamp']),
            models.Index(fields=['content_type', 'object_id']),
        ]
    
    @classmethod
    def log(cls, user, action, obj=None, changes=None, request=None):
        log_data = {
            'user': user,
            'action': action,
            'changes': changes or {},
        }
        
        if obj:
            log_data['content_type'] = ContentType.objects.get_for_model(obj)
            log_data['object_id'] = obj.pk
        
        if request:
            log_data['ip_address'] = request.META.get('REMOTE_ADDR')
            log_data['user_agent'] = request.META.get('HTTP_USER_AGENT', '')[:500]
        
        return cls.objects.create(**log_data)

class AuditMiddleware:
    """Middleware สำหรับ auto audit logging"""
    
    TRACKED_METHODS = ['POST', 'PUT', 'PATCH', 'DELETE']
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        if (request.user.is_authenticated and 
            request.method in self.TRACKED_METHODS and
            response.status_code < 400):
            
            self.log_action(request, response)
        
        return response
    
    def log_action(self, request, response):
        action_map = {
            'POST': AuditLog.ACTION_CREATE,
            'PUT': AuditLog.ACTION_UPDATE,
            'PATCH': AuditLog.ACTION_UPDATE,
            'DELETE': AuditLog.ACTION_DELETE,
        }
        
        AuditLog.log(
            user=request.user,
            action=action_map.get(request.method, 'unknown'),
            request=request,
            changes={'path': request.path, 'method': request.method}
        )
```

### ข้อ 8: Complete Auth Tests

สร้าง comprehensive tests สำหรับ authentication system

**เฉลย:**

```python
from rest_framework.test import APITestCase
from django.contrib.auth import get_user_model

User = get_user_model()

class AuthenticationTests(APITestCase):
    
    def setUp(self):
        self.user = User.objects.create_user(
            email='test@example.com',
            username='testuser',
            password='testpass123',
            email_verified=True
        )
    
    def test_register_success(self):
        data = {
            'email': 'new@example.com',
            'username': 'newuser',
            'password': 'newpass123',
            'password_confirm': 'newpass123',
            'first_name': 'New',
            'last_name': 'User'
        }
        response = self.client.post('/api/auth/register/', data)
        self.assertEqual(response.status_code, 201)
    
    def test_register_duplicate_email(self):
        data = {
            'email': 'test@example.com',  # already exists
            'username': 'another',
            'password': 'pass123',
            'password_confirm': 'pass123',
        }
        response = self.client.post('/api/auth/register/', data)
        self.assertEqual(response.status_code, 400)
    
    def test_login_success(self):
        data = {'email': 'test@example.com', 'password': 'testpass123'}
        response = self.client.post('/api/auth/login/', data)
        self.assertEqual(response.status_code, 200)
        self.assertIn('access', response.data)
        self.assertIn('refresh', response.data)
    
    def test_login_wrong_password(self):
        data = {'email': 'test@example.com', 'password': 'wrongpass'}
        response = self.client.post('/api/auth/login/', data)
        self.assertEqual(response.status_code, 400)
    
    def test_login_unverified_email(self):
        unverified = User.objects.create_user(
            email='unverified@example.com',
            username='unverified',
            password='pass123',
            email_verified=False
        )
        data = {'email': 'unverified@example.com', 'password': 'pass123'}
        response = self.client.post('/api/auth/login/', data)
        self.assertEqual(response.status_code, 400)
    
    def test_protected_endpoint_requires_auth(self):
        response = self.client.get('/api/auth/profile/')
        self.assertEqual(response.status_code, 401)
    
    def test_protected_endpoint_with_token(self):
        # Login
        login_response = self.client.post('/api/auth/login/', {
            'email': 'test@example.com',
            'password': 'testpass123'
        })
        token = login_response.data['access']
        
        # Use token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {token}')
        response = self.client.get('/api/auth/profile/')
        self.assertEqual(response.status_code, 200)
    
    def test_token_refresh(self):
        login_response = self.client.post('/api/auth/login/', {
            'email': 'test@example.com',
            'password': 'testpass123'
        })
        refresh_token = login_response.data['refresh']
        
        response = self.client.post('/api/auth/token/refresh/', {
            'refresh': refresh_token
        })
        self.assertEqual(response.status_code, 200)
        self.assertIn('access', response.data)
    
    def test_logout_blacklists_token(self):
        login_response = self.client.post('/api/auth/login/', {
            'email': 'test@example.com',
            'password': 'testpass123'
        })
        refresh_token = login_response.data['refresh']
        
        self.client.credentials(
            HTTP_AUTHORIZATION=f'Bearer {login_response.data["access"]}'
        )
        
        # Logout
        response = self.client.post('/api/auth/logout/', {
            'refresh': refresh_token
        })
        self.assertEqual(response.status_code, 200)
        
        # Try refresh after logout
        self.client.credentials()
        response = self.client.post('/api/auth/token/refresh/', {
            'refresh': refresh_token
        })
        self.assertEqual(response.status_code, 401)
```

---

## สรุป

Django Authentication & Permissions system เป็น foundation สำคัญของ web application security:

1. **Custom User Model** - ควรสร้างตั้งแต่ต้น ก่อน migrate ครั้งแรก
2. **Authentication Backends** - customize วิธี authenticate (email, social)
3. **Registration Flow** - validation, email verification, security
4. **Password Management** - reset, change, strong validation
5. **JWT Authentication** - stateless, suitable for APIs
6. **Permissions** - model-level, object-level, group-based
7. **django-allauth** - social auth, email verification built-in
8. **django-guardian** - object-level permissions
9. **Security** - rate limiting, brute force protection, audit logs

---

*หัวข้อถัดไป: Part 68 - GraphQL with Strawberry & Graphene*
