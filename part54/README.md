# Part 54: Flask - Database with SQLAlchemy

## สารบัญ
1. [Flask-SQLAlchemy Setup](#1-flask-sqlalchemy-setup)
2. [Database Models](#2-database-models)
3. [Relationships](#3-relationships)
4. [CRUD Operations](#4-crud-operations)
5. [Queries ขั้นสูง](#5-queries-ขั้นสูง)
6. [Flask-Migrate](#6-flask-migrate)
7. [Association Tables](#7-association-tables)
8. [Database Initialization](#8-database-initialization)
9. [Query Optimization](#9-query-optimization)
10. [Lazy vs Eager Loading](#10-lazy-vs-eager-loading)
11. [ตัวอย่างโปรแกรมจริง](#11-ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. Flask-SQLAlchemy Setup

### SQLAlchemy คืออะไร

SQLAlchemy คือ Python ORM (Object-Relational Mapper) ที่ช่วยให้เราทำงานกับ database ผ่าน Python objects แทนที่จะเขียน SQL โดยตรง

**Flask-SQLAlchemy** คือ Flask extension ที่ integrate SQLAlchemy เข้ากับ Flask app

```bash
# ติดตั้ง packages
pip install flask-sqlalchemy       # SQLAlchemy สำหรับ Flask
pip install flask-migrate          # Database migrations
pip install psycopg2-binary        # PostgreSQL driver
pip install pymysql                # MySQL driver
# SQLite ไม่ต้องติดตั้ง driver เพิ่มเติม
```

### Basic Setup

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Configuration
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # ปิดการแจ้งเตือนที่ไม่จำเป็น
app.config['SQLALCHEMY_ECHO'] = True  # แสดง SQL queries (dev only)

# สร้าง SQLAlchemy instance
db = SQLAlchemy(app)

# หรือใช้ App Factory Pattern
db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    
    db.init_app(app)
    
    return app
```

### Database URI Formats

```python
# SQLite (ไฟล์ใน project directory)
SQLALCHEMY_DATABASE_URI = 'sqlite:///app.db'

# SQLite (absolute path)
SQLALCHEMY_DATABASE_URI = 'sqlite:////home/user/myapp/app.db'

# SQLite (in memory - สำหรับ testing)
SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'

# PostgreSQL
SQLALCHEMY_DATABASE_URI = 'postgresql://username:password@localhost/dbname'
SQLALCHEMY_DATABASE_URI = 'postgresql+psycopg2://username:password@localhost:5432/dbname'

# MySQL
SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://username:password@localhost/dbname'

# ดึงจาก environment variable (best practice)
import os
SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL', 'sqlite:///dev.db')
```

### App Factory กับ SQLAlchemy

```python
# extensions.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

# app/__init__.py
from flask import Flask
from .extensions import db

def create_app(config_name='development'):
    app = Flask(__name__)
    
    # Load config
    if config_name == 'development':
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///dev.db'
        app.config['DEBUG'] = True
    elif config_name == 'testing':
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
        app.config['TESTING'] = True
    elif config_name == 'production':
        import os
        app.config['SQLALCHEMY_DATABASE_URI'] = os.environ.get('DATABASE_URL')
    
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    app.config['SECRET_KEY'] = 'dev-secret'
    
    # Initialize extensions
    db.init_app(app)
    
    # Register blueprints
    from .main import main
    app.register_blueprint(main)
    
    # Create tables
    with app.app_context():
        db.create_all()
    
    return app
```

---

## 2. Database Models

### การสร้าง Model

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    """User model"""
    
    # ชื่อ table (Flask-SQLAlchemy จะสร้างให้อัตโนมัติถ้าไม่ระบุ)
    __tablename__ = 'users'
    
    # Columns
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    full_name = db.Column(db.String(200))
    bio = db.Column(db.Text)
    avatar_url = db.Column(db.String(500))
    is_active = db.Column(db.Boolean, default=True, nullable=False)
    is_admin = db.Column(db.Boolean, default=False, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    def __repr__(self):
        return f'<User {self.username}>'
    
    def __str__(self):
        return self.username
    
    def to_dict(self):
        """แปลง model เป็น dictionary"""
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'full_name': self.full_name,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat() if self.created_at else None
        }
```

### Column Types ที่ใช้บ่อย

```python
class ColumnTypesDemo(db.Model):
    __tablename__ = 'column_types_demo'
    
    # Integer types
    id = db.Column(db.Integer, primary_key=True)          # auto-increment
    small_num = db.Column(db.SmallInteger)                 # -32768 to 32767
    big_num = db.Column(db.BigInteger)                     # large integers
    
    # String types
    name = db.Column(db.String(100))                       # VARCHAR(100)
    description = db.Column(db.Text)                       # TEXT (unlimited)
    code = db.Column(db.CHAR(10))                          # CHAR(10)
    
    # Numeric types
    price = db.Column(db.Numeric(10, 2))                   # DECIMAL(10,2)
    rating = db.Column(db.Float)                           # FLOAT
    
    # Boolean
    is_active = db.Column(db.Boolean, default=True)
    
    # Date/Time
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    date_only = db.Column(db.Date)                         # DATE
    time_only = db.Column(db.Time)                         # TIME
    timestamp = db.Column(db.TIMESTAMP)                    # TIMESTAMP
    
    # Binary
    data = db.Column(db.LargeBinary)                       # BLOB
    
    # Enum (PostgreSQL)
    from sqlalchemy import Enum
    status = db.Column(Enum('active', 'inactive', 'pending', name='status_enum'))
    
    # JSON (PostgreSQL, MySQL 5.7+, SQLite 3.38+)
    from sqlalchemy.dialects.postgresql import JSON
    metadata_json = db.Column(db.JSON)
    
    # Array (PostgreSQL only)
    from sqlalchemy.dialects.postgresql import ARRAY
    tags = db.Column(ARRAY(db.String))
```

### Column Options

```python
class ColumnOptions(db.Model):
    __tablename__ = 'column_options'
    
    # Primary Key
    id = db.Column(db.Integer, primary_key=True)
    
    # Not Null
    required_field = db.Column(db.String(100), nullable=False)
    
    # Default value
    status = db.Column(db.String(20), default='active')
    count = db.Column(db.Integer, default=0, server_default='0')
    
    # Unique
    email = db.Column(db.String(120), unique=True)
    
    # Index
    username = db.Column(db.String(80), index=True)
    
    # Comment
    notes = db.Column(db.Text, comment='บันทึกเพิ่มเติม')
    
    # Composite constraints
    __table_args__ = (
        # Unique constraint หลายคอลัมน์
        db.UniqueConstraint('username', 'email', name='unique_user_email'),
        # Index หลายคอลัมน์
        db.Index('idx_username_email', 'username', 'email'),
        # Check constraint
        db.CheckConstraint('price >= 0', name='check_price_positive'),
    )
```

### Model Methods และ Properties

```python
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    _password_hash = db.Column('password_hash', db.String(256), nullable=False)
    first_name = db.Column(db.String(50))
    last_name = db.Column(db.String(50))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Password property
    @property
    def password(self):
        raise AttributeError('password is not readable')
    
    @password.setter
    def password(self, password):
        self._password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self._password_hash, password)
    
    # Computed property
    @property
    def full_name(self):
        if self.first_name and self.last_name:
            return f'{self.first_name} {self.last_name}'
        return self.username
    
    # Class methods
    @classmethod
    def get_by_username(cls, username):
        return cls.query.filter_by(username=username).first()
    
    @classmethod
    def get_by_email(cls, email):
        return cls.query.filter_by(email=email.lower()).first()
    
    @classmethod
    def get_active_users(cls):
        return cls.query.filter_by(is_active=True).all()
    
    # Instance methods
    def activate(self):
        self.is_active = True
    
    def deactivate(self):
        self.is_active = False
    
    def to_dict(self, include_email=False):
        data = {
            'id': self.id,
            'username': self.username,
            'full_name': self.full_name,
            'created_at': self.created_at.isoformat()
        }
        if include_email:
            data['email'] = self.email
        return data
    
    def __repr__(self):
        return f'<User {self.username}>'
```

---

## 3. Relationships

### One-to-Many Relationship

ความสัมพันธ์ที่ record หนึ่งในตารางหนึ่ง สามารถมีได้หลาย records ในอีกตาราง

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    
    # Relationship: User มีหลาย Posts
    posts = db.relationship('Post', backref='author', lazy='dynamic')
    # backref='author' สร้าง property 'author' ใน Post model
    # lazy='dynamic' return query object แทน list
    
    def __repr__(self):
        return f'<User {self.username}>'


class Post(db.Model):
    __tablename__ = 'posts'
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    is_published = db.Column(db.Boolean, default=False)
    
    # Foreign Key: อ้างอิงไปยัง users.id
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    
    # Relationship: Post มีหลาย Comments
    comments = db.relationship('Comment', backref='post', cascade='all, delete-orphan')
    
    def __repr__(self):
        return f'<Post {self.title[:30]}>'


class Comment(db.Model):
    __tablename__ = 'comments'
    
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Foreign Keys
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    post_id = db.Column(db.Integer, db.ForeignKey('posts.id'), nullable=False)
    
    # Relationships
    user = db.relationship('User', backref='comments')


# การใช้งาน One-to-Many
# สร้าง user
user = User(username='somchai', email='somchai@example.com')
db.session.add(user)
db.session.commit()

# สร้าง posts ของ user
post1 = Post(title='Post 1', content='Content 1', user_id=user.id)
post2 = Post(title='Post 2', content='Content 2', user_id=user.id)
db.session.add_all([post1, post2])
db.session.commit()

# ดึง posts ของ user
user_posts = user.posts.all()  # lazy='dynamic' ต้องใช้ .all()
# หรือ
user_posts = Post.query.filter_by(user_id=user.id).all()

# ดึง author ของ post
author = post1.author  # backref property
```

### One-to-One Relationship

```python
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), nullable=False)
    
    # One-to-One: User มี Profile หนึ่งอัน
    profile = db.relationship('UserProfile', back_populates='user', uselist=False)

class UserProfile(db.Model):
    __tablename__ = 'user_profiles'
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), unique=True)  # unique ทำให้เป็น one-to-one
    avatar_url = db.Column(db.String(500))
    bio = db.Column(db.Text)
    website = db.Column(db.String(500))
    location = db.Column(db.String(200))
    
    user = db.relationship('User', back_populates='profile')

# การใช้งาน
user = User(username='somchai')
db.session.add(user)
db.session.flush()  # ให้ได้ user.id ก่อน

profile = UserProfile(
    user_id=user.id,
    bio='Python developer',
    location='Bangkok'
)
db.session.add(profile)
db.session.commit()

# ดึง profile ของ user
user_profile = user.profile

# ดึง user ของ profile
profile_user = profile.user
```

### Many-to-Many Relationship

```python
# Association table (junction table)
# สำหรับ many-to-many ที่ไม่มี extra columns
post_tags = db.Table('post_tags',
    db.Column('post_id', db.Integer, db.ForeignKey('posts.id'), primary_key=True),
    db.Column('tag_id', db.Integer, db.ForeignKey('tags.id'), primary_key=True)
)

class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text)
    
    # Many-to-Many relationship กับ Tag
    tags = db.relationship('Tag', secondary=post_tags, back_populates='posts')

class Tag(db.Model):
    __tablename__ = 'tags'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), unique=True, nullable=False)
    
    posts = db.relationship('Post', secondary=post_tags, back_populates='tags')

# การใช้งาน Many-to-Many
tag1 = Tag(name='python')
tag2 = Tag(name='flask')
tag3 = Tag(name='web')
db.session.add_all([tag1, tag2, tag3])

post = Post(title='Flask Tutorial', content='...')
post.tags.append(tag1)  # เพิ่ม tag ให้ post
post.tags.append(tag2)
db.session.add(post)
db.session.commit()

# ดึง tags ของ post
print(post.tags)  # [<Tag python>, <Tag flask>]

# ดึง posts ของ tag
print(tag1.posts)  # posts ที่มี tag python

# ลบ tag ออกจาก post
post.tags.remove(tag1)
db.session.commit()
```

### Relationship Options

```python
# lazy loading options:
# 'select'  (default) - โหลดเมื่อ access (1+N query problem)
# 'dynamic' - return query object (ต้องใช้ .all())
# 'joined'  - JOIN query (1 query)
# 'subquery' - subquery (1+1 query)
# 'noload'  - ไม่โหลดเลย
# True      - เหมือน 'select'
# False     - เหมือน 'joined'
# 'raise'   - raise error ถ้า access นอก context

# cascade options:
# 'all'         - ทุก operation
# 'save-update' - cascade เมื่อ add
# 'delete'      - cascade เมื่อ delete parent
# 'delete-orphan' - ลบ orphan records

class User(db.Model):
    # Lazy loading แบบต่างๆ
    posts = db.relationship('Post', lazy='select')      # default
    posts_dynamic = db.relationship('Post', lazy='dynamic')
    posts_joined = db.relationship('Post', lazy='joined')
    
    # Cascade
    posts_cascade = db.relationship('Post', cascade='all, delete-orphan')
    
    # Ordering
    posts_ordered = db.relationship('Post', 
                                    order_by='Post.created_at.desc()')
    
    # Foreign key specification (ถ้ามี ambiguous FK)
    owned_posts = db.relationship('Post', 
                                   foreign_keys='Post.owner_id')
```

---

## 4. CRUD Operations

### Create

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

# วิธีที่ 1: สร้างและ add แยก
user = User(username='somchai', email='somchai@example.com')
db.session.add(user)
db.session.commit()

# วิธีที่ 2: สร้างหลายอัน
users = [
    User(username='user1', email='user1@example.com'),
    User(username='user2', email='user2@example.com'),
    User(username='user3', email='user3@example.com'),
]
db.session.add_all(users)
db.session.commit()

# วิธีที่ 3: สร้างจาก dict
user_data = {'username': 'somying', 'email': 'somying@example.com'}
user = User(**user_data)
db.session.add(user)
db.session.commit()

# ตรวจสอบ id หลัง commit
print(user.id)  # ID ที่ได้จาก database

# db.session.flush() - sync กับ database แต่ยังไม่ commit
# ใช้เมื่อต้องการ id ก่อน commit
user = User(username='test', email='test@example.com')
db.session.add(user)
db.session.flush()
print(user.id)  # มี id แล้ว แต่ยังไม่ commit
db.session.commit()
```

### Read

```python
# ดึง record ทั้งหมด
users = User.query.all()

# ดึงตาม primary key
user = User.query.get(1)           # None ถ้าไม่พบ
user = db.session.get(User, 1)     # วิธีใหม่ (SQLAlchemy 2.0)

# ดึง record แรก
user = User.query.first()

# ดึงพร้อม raise 404
user = User.query.get_or_404(1)    # abort(404) ถ้าไม่พบ

# Filter
user = User.query.filter_by(username='somchai').first()
users = User.query.filter_by(is_active=True).all()

# Filter ด้วย expressions
from sqlalchemy import or_, and_, not_

# Equal
users = User.query.filter(User.username == 'somchai').all()

# Not equal
users = User.query.filter(User.username != 'admin').all()

# Like (case-sensitive)
users = User.query.filter(User.username.like('%chai%')).all()

# ilike (case-insensitive)
users = User.query.filter(User.username.ilike('%chai%')).all()

# In
users = User.query.filter(User.id.in_([1, 2, 3])).all()

# Not in
users = User.query.filter(~User.id.in_([1, 2, 3])).all()

# Is None / Is Not None
users = User.query.filter(User.bio.is_(None)).all()
users = User.query.filter(User.bio.isnot(None)).all()

# Range
users = User.query.filter(User.id.between(1, 100)).all()

# Comparison
users = User.query.filter(User.created_at >= datetime(2024, 1, 1)).all()

# AND (implicit)
users = User.query.filter(
    User.is_active == True,
    User.is_admin == False
).all()

# AND (explicit)
users = User.query.filter(
    and_(User.is_active == True, User.is_admin == False)
).all()

# OR
users = User.query.filter(
    or_(User.username == 'admin', User.is_admin == True)
).all()

# NOT
users = User.query.filter(
    not_(User.is_active == True)
).all()
```

### Update

```python
# วิธีที่ 1: ดึงแล้วแก้ไข
user = User.query.get(1)
if user:
    user.username = 'new_username'
    user.email = 'new@example.com'
    db.session.commit()

# วิธีที่ 2: update() สำหรับหลาย records
User.query.filter_by(is_active=False).update({'is_active': True})
db.session.commit()

# วิธีที่ 3: update ด้วย expressions
from sqlalchemy import func

Post.query.filter_by(user_id=1).update({
    'view_count': Post.view_count + 1  # Increment
})
db.session.commit()

# วิธีที่ 4: bulk update (ประหยัดการ query)
db.session.execute(
    db.update(User).where(User.is_active == False).values(is_active=True)
)
db.session.commit()
```

### Delete

```python
# วิธีที่ 1: ดึงแล้วลบ
user = User.query.get(1)
if user:
    db.session.delete(user)
    db.session.commit()

# วิธีที่ 2: delete() สำหรับหลาย records
User.query.filter_by(is_active=False).delete()
db.session.commit()

# วิธีที่ 3: bulk delete
db.session.execute(
    db.delete(User).where(User.is_active == False)
)
db.session.commit()

# Soft delete (ไม่ลบจริง แค่ mark ว่าลบแล้ว)
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
    deleted_at = db.Column(db.DateTime, nullable=True)
    
    def soft_delete(self):
        self.deleted_at = datetime.utcnow()
        db.session.commit()
    
    @classmethod
    def get_active(cls):
        return cls.query.filter(cls.deleted_at.is_(None))
```

### Error Handling

```python
from flask import Flask, jsonify
from sqlalchemy.exc import IntegrityError, SQLAlchemyError

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    
    try:
        user = User(
            username=data['username'],
            email=data['email']
        )
        db.session.add(user)
        db.session.commit()
        return jsonify(user.to_dict()), 201
    
    except IntegrityError as e:
        db.session.rollback()  # สำคัญ! rollback เสมอเมื่อ error
        return jsonify({'error': 'Username หรือ email มีอยู่แล้ว'}), 409
    
    except SQLAlchemyError as e:
        db.session.rollback()
        app.logger.error(f'Database error: {e}')
        return jsonify({'error': 'Database error'}), 500
```

---

## 5. Queries ขั้นสูง

### Filtering และ Ordering

```python
# Order by
users = User.query.order_by(User.username).all()          # ASC
users = User.query.order_by(User.username.asc()).all()    # ASC
users = User.query.order_by(User.created_at.desc()).all() # DESC

# หลายคอลัมน์
posts = Post.query.order_by(Post.is_published.desc(), Post.created_at.desc()).all()

# Limit และ Offset
users = User.query.limit(10).all()                  # 10 records
users = User.query.offset(20).limit(10).all()       # 10 records เริ่มจาก 20
users = User.query.paginate(page=2, per_page=10)    # Pagination

# Counting
total = User.query.count()
active_count = User.query.filter_by(is_active=True).count()

# Distinct
emails = db.session.query(User.email).distinct().all()

# Aggregation
from sqlalchemy import func

# COUNT
user_count = db.session.query(func.count(User.id)).scalar()

# SUM
total_price = db.session.query(func.sum(Order.amount)).scalar()

# AVG
avg_age = db.session.query(func.avg(User.age)).scalar()

# MIN, MAX
min_price = db.session.query(func.min(Product.price)).scalar()
max_price = db.session.query(func.max(Product.price)).scalar()

# GROUP BY
from sqlalchemy import func

results = db.session.query(
    Post.user_id,
    func.count(Post.id).label('post_count')
).group_by(Post.user_id).all()

for user_id, count in results:
    print(f'User {user_id}: {count} posts')
```

### Pagination

```python
from flask import Flask, jsonify, request

@app.route('/users')
def get_users():
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 10, type=int)
    
    # Paginate query
    pagination = User.query.filter_by(is_active=True)\
                           .order_by(User.created_at.desc())\
                           .paginate(page=page, per_page=per_page, error_out=False)
    
    return jsonify({
        'users': [u.to_dict() for u in pagination.items],
        'total': pagination.total,
        'pages': pagination.pages,
        'current_page': pagination.page,
        'per_page': pagination.per_page,
        'has_next': pagination.has_next,
        'has_prev': pagination.has_prev,
        'next_page': pagination.next_num if pagination.has_next else None,
        'prev_page': pagination.prev_num if pagination.has_prev else None,
    })
```

### JOIN Queries

```python
from sqlalchemy.orm import joinedload, subqueryload, contains_eager

# Explicit JOIN
results = db.session.query(User, Post)\
    .join(Post, User.id == Post.user_id)\
    .filter(Post.is_published == True)\
    .all()

for user, post in results:
    print(f'{user.username}: {post.title}')

# OUTER JOIN
results = db.session.query(User, Post)\
    .outerjoin(Post, User.id == Post.user_id)\
    .all()

# ดึง users พร้อม post count
from sqlalchemy import func

results = db.session.query(
    User,
    func.count(Post.id).label('post_count')
).outerjoin(Post, User.id == Post.user_id)\
 .group_by(User.id)\
 .all()

for user, count in results:
    print(f'{user.username}: {count} posts')

# Eager Loading ด้วย joinedload
users = User.query.options(joinedload(User.posts)).all()
for user in users:
    # ไม่มี N+1 problem
    print(f'{user.username}: {len(user.posts.all())} posts')
```

### Subqueries

```python
from sqlalchemy import exists

# Exists subquery
users_with_posts = User.query.filter(
    exists().where(Post.user_id == User.id)
).all()

# Scalar subquery
from sqlalchemy import select

post_count_sq = select(func.count(Post.id))\
    .where(Post.user_id == User.id)\
    .correlate(User)\
    .as_scalar()

users = db.session.query(User, post_count_sq.label('post_count')).all()
```

---

## 6. Flask-Migrate

### Flask-Migrate คืออะไร

Flask-Migrate ใช้ Alembic เพื่อจัดการ database schema changes (migrations) คล้ายกับ Django migrations

```bash
pip install flask-migrate
```

### Setup Flask-Migrate

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
migrate = Migrate(app, db)  # Initialize Flask-Migrate

# Models
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
```

### Migration Commands

```bash
# 1. Initialize migrations folder (ทำครั้งเดียว)
flask db init

# โครงสร้าง migrations/
# migrations/
# ├── alembic.ini
# ├── env.py
# ├── README
# └── versions/
#     └── (migration files จะอยู่ที่นี่)

# 2. สร้าง migration file จาก model changes
flask db migrate -m "Initial migration"
flask db migrate -m "Add user table"
flask db migrate -m "Add posts and comments tables"

# 3. Apply migrations ไปยัง database
flask db upgrade

# 4. Rollback migration
flask db downgrade

# 5. ดูสถานะ migration ปัจจุบัน
flask db current

# 6. ดูประวัติ migrations ทั้งหมด
flask db history

# 7. Upgrade ไปยัง version ที่ระบุ
flask db upgrade <revision>

# 8. Downgrade ไปยัง version ที่ระบุ
flask db downgrade <revision>
```

### Migration File ตัวอย่าง

```python
# migrations/versions/001_initial.py
"""Initial migration

Revision ID: abc123
Revises: 
Create Date: 2024-01-01 12:00:00

"""
from alembic import op
import sqlalchemy as sa

revision = 'abc123'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    """สร้าง tables"""
    op.create_table('users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('username', sa.String(length=80), nullable=False),
        sa.Column('email', sa.String(length=120), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('email'),
        sa.UniqueConstraint('username')
    )

def downgrade():
    """ลบ tables"""
    op.drop_table('users')
```

```python
# migrations/versions/002_add_posts.py
"""Add posts table

Revision ID: def456
Revises: abc123
"""
from alembic import op
import sqlalchemy as sa

revision = 'def456'
down_revision = 'abc123'  # อ้างอิง migration ก่อนหน้า

def upgrade():
    op.create_table('posts',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('title', sa.String(length=200), nullable=False),
        sa.Column('content', sa.Text(), nullable=False),
        sa.Column('user_id', sa.Integer(), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.ForeignKeyConstraint(['user_id'], ['users.id']),
        sa.PrimaryKeyConstraint('id')
    )
    
    # เพิ่ม index
    op.create_index('ix_posts_user_id', 'posts', ['user_id'])

def downgrade():
    op.drop_index('ix_posts_user_id', 'posts')
    op.drop_table('posts')
```

---

## 7. Association Tables

### Simple Association Table

```python
# Many-to-Many โดยใช้ db.Table (ไม่มี extra columns)
followers = db.Table('followers',
    db.Column('follower_id', db.Integer, db.ForeignKey('users.id'), primary_key=True),
    db.Column('followed_id', db.Integer, db.ForeignKey('users.id'), primary_key=True)
)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
    
    # Self-referential Many-to-Many (ผู้ติดตาม)
    following = db.relationship(
        'User',
        secondary=followers,
        primaryjoin=(followers.c.follower_id == id),
        secondaryjoin=(followers.c.followed_id == id),
        backref=db.backref('followers', lazy='dynamic'),
        lazy='dynamic'
    )
    
    def follow(self, user):
        if not self.is_following(user):
            self.following.append(user)
    
    def unfollow(self, user):
        if self.is_following(user):
            self.following.remove(user)
    
    def is_following(self, user):
        return self.following.filter(
            followers.c.followed_id == user.id
        ).count() > 0
```

### Association Object (มี Extra Columns)

```python
# สำหรับ many-to-many ที่มี extra columns
class Enrollment(db.Model):
    """นักเรียนลงทะเบียนวิชา (มีข้อมูลเพิ่มเติม)"""
    __tablename__ = 'enrollments'
    
    student_id = db.Column(db.Integer, db.ForeignKey('students.id'), primary_key=True)
    course_id = db.Column(db.Integer, db.ForeignKey('courses.id'), primary_key=True)
    
    # Extra columns
    enrolled_at = db.Column(db.DateTime, default=datetime.utcnow)
    grade = db.Column(db.Float, nullable=True)
    status = db.Column(db.String(20), default='enrolled')
    
    # Relationships
    student = db.relationship('Student', back_populates='enrollments')
    course = db.relationship('Course', back_populates='enrollments')

class Student(db.Model):
    __tablename__ = 'students'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    
    enrollments = db.relationship('Enrollment', back_populates='student')
    
    # Shortcut ไปยัง courses
    courses = db.relationship('Course', secondary='enrollments', 
                               viewonly=True,
                               overlaps='enrollments,student,course')

class Course(db.Model):
    __tablename__ = 'courses'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(200))
    
    enrollments = db.relationship('Enrollment', back_populates='course')
    students = db.relationship('Student', secondary='enrollments',
                                viewonly=True,
                                overlaps='enrollments,student,course')

# การใช้งาน
student = Student(name='สมชาย')
course = Course(name='Python 101')
db.session.add_all([student, course])
db.session.commit()

# ลงทะเบียน
enrollment = Enrollment(
    student_id=student.id,
    course_id=course.id
)
db.session.add(enrollment)
db.session.commit()

# อัพเดท grade
enrollment.grade = 85.5
db.session.commit()

# ดึงข้อมูล
for enroll in student.enrollments:
    print(f'{enroll.course.name}: {enroll.grade}')
```

---

## 8. Database Initialization

### สร้าง Tables ครั้งแรก

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))

# สร้าง tables ทั้งหมด
with app.app_context():
    db.create_all()

# ลบ tables ทั้งหมด (ระวัง!)
with app.app_context():
    db.drop_all()

# สร้างใหม่
with app.app_context():
    db.drop_all()
    db.create_all()
```

### Seed Data (Initial Data)

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
db = SQLAlchemy(app)

def seed_database():
    """เพิ่มข้อมูลเริ่มต้น"""
    
    # ตรวจสอบว่ามีข้อมูลอยู่แล้วหรือไม่
    if User.query.count() > 0:
        print('Database already seeded')
        return
    
    # สร้าง users
    admin = User(
        username='admin',
        email='admin@example.com',
        is_admin=True
    )
    admin.password = 'Admin1234!'
    
    users = [
        User(username='somchai', email='somchai@example.com'),
        User(username='somying', email='somying@example.com'),
    ]
    
    db.session.add(admin)
    db.session.add_all(users)
    db.session.commit()
    
    # สร้าง categories
    categories = [
        Category(name='Python', slug='python'),
        Category(name='Flask', slug='flask'),
        Category(name='Database', slug='database'),
    ]
    db.session.add_all(categories)
    db.session.commit()
    
    print('Database seeded successfully!')

# Flask CLI command
@app.cli.command('seed-db')
def seed_db_command():
    """เพิ่มข้อมูลเริ่มต้น"""
    with app.app_context():
        seed_database()

# รัน: flask seed-db
```

### Custom CLI Commands

```python
import click
from flask import Flask

app = Flask(__name__)

@app.cli.command('create-admin')
@click.argument('username')
@click.argument('email')
@click.password_option()
def create_admin(username, email, password):
    """สร้าง admin user"""
    admin = User(username=username, email=email, is_admin=True)
    admin.password = password
    db.session.add(admin)
    db.session.commit()
    click.echo(f'สร้าง admin {username} สำเร็จ!')

@app.cli.command('reset-db')
@click.confirmation_option(prompt='Are you sure you want to reset the database?')
def reset_db():
    """ลบและสร้าง database ใหม่"""
    db.drop_all()
    db.create_all()
    click.echo('Database reset successfully!')
```

---

## 9. Query Optimization

### N+1 Query Problem

```python
# BAD: N+1 queries
users = User.query.all()
for user in users:
    # แต่ละ user สร้าง 1 query
    print(f'{user.username}: {user.posts.count()} posts')
# รวม: 1 + N queries

# GOOD: Eager loading
from sqlalchemy.orm import joinedload, subqueryload

# joinedload: ใช้ JOIN (1 query)
users = User.query.options(joinedload(User.posts)).all()

# subqueryload: ใช้ subquery (2 queries)
users = User.query.options(subqueryload(User.posts)).all()

for user in users:
    print(f'{user.username}: {len(user.posts)} posts')
```

### Indexes

```python
class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), index=True)  # Single column index
    email = db.Column(db.String(120), unique=True)   # Unique index
    
    # Composite index
    __table_args__ = (
        db.Index('idx_username_active', 'username', 'is_active'),
    )
```

### Query Profiling

```python
from flask_sqlalchemy import SQLAlchemy
import logging

# เปิด SQL logging
logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)

app.config['SQLALCHEMY_ECHO'] = True  # แสดง SQL ทุก query

# ใน production ใช้ EXPLAIN
result = db.session.execute(
    db.text('EXPLAIN QUERY PLAN SELECT * FROM users WHERE username = :name'),
    {'name': 'somchai'}
)
```

---

## 10. Lazy vs Eager Loading

### Lazy Loading (Default)

```python
class User(db.Model):
    posts = db.relationship('Post', lazy='select')  # default

# ดึง users
users = User.query.all()  # 1 query

# เมื่อ access posts จะสร้าง query ใหม่สำหรับแต่ละ user (N+1 problem)
for user in users:
    posts = user.posts  # +1 query ต่อ user
```

### Eager Loading

```python
from sqlalchemy.orm import joinedload, subqueryload, selectinload

# Joined Loading (JOIN query - 1 query total)
users = User.query.options(joinedload(User.posts)).all()

# Subquery Loading (2 queries total)
users = User.query.options(subqueryload(User.posts)).all()

# Select In Loading (2 queries, efficient for many items)
users = User.query.options(selectinload(User.posts)).all()

# Nested eager loading
users = User.query.options(
    joinedload(User.posts).joinedload(Post.comments)
).all()

# เลือก specific columns
from sqlalchemy.orm import load_only
users = User.query.options(
    load_only(User.id, User.username),
    joinedload(User.posts).load_only(Post.id, Post.title)
).all()
```

### Dynamic Loading

```python
class User(db.Model):
    # lazy='dynamic' คืน query object
    posts = db.relationship('Post', lazy='dynamic')

user = User.query.get(1)

# posts เป็น query object สามารถ chain ได้
recent_posts = user.posts.filter(Post.created_at > some_date)\
                         .order_by(Post.created_at.desc())\
                         .limit(5)\
                         .all()

# นับจำนวน
post_count = user.posts.count()

# ตรวจสอบ
has_posts = user.posts.first() is not None
```

---

## 11. ตัวอย่างโปรแกรมจริง

### Blog with Posts and Comments

```python
# blog_app.py
# Complete blog application with SQLAlchemy

from flask import Flask, jsonify, request, abort
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///blog.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'blog-secret'

db = SQLAlchemy(app)

# Models
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    bio = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    posts = db.relationship('Post', backref='author', lazy='dynamic',
                             cascade='all, delete-orphan')
    comments = db.relationship('Comment', backref='author', lazy='dynamic')
    
    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'bio': self.bio,
            'post_count': self.posts.count(),
            'created_at': self.created_at.isoformat()
        }

# Tags association table
post_tags = db.Table('post_tags',
    db.Column('post_id', db.Integer, db.ForeignKey('posts.id'), primary_key=True),
    db.Column('tag_id', db.Integer, db.ForeignKey('tags.id'), primary_key=True)
)

class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    slug = db.Column(db.String(200), unique=True)
    content = db.Column(db.Text, nullable=False)
    excerpt = db.Column(db.String(500))
    is_published = db.Column(db.Boolean, default=False)
    view_count = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    
    comments = db.relationship('Comment', backref='post', lazy='dynamic',
                                cascade='all, delete-orphan')
    tags = db.relationship('Tag', secondary=post_tags, back_populates='posts')
    
    def to_dict(self, include_content=False):
        data = {
            'id': self.id,
            'title': self.title,
            'slug': self.slug,
            'excerpt': self.excerpt,
            'author': self.author.username if self.author else None,
            'tags': [t.name for t in self.tags],
            'comment_count': self.comments.count(),
            'view_count': self.view_count,
            'is_published': self.is_published,
            'created_at': self.created_at.isoformat()
        }
        if include_content:
            data['content'] = self.content
        return data

class Comment(db.Model):
    __tablename__ = 'comments'
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    post_id = db.Column(db.Integer, db.ForeignKey('posts.id'), nullable=False)
    
    def to_dict(self):
        return {
            'id': self.id,
            'content': self.content,
            'author': self.author.username,
            'created_at': self.created_at.isoformat()
        }

class Tag(db.Model):
    __tablename__ = 'tags'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), unique=True, nullable=False)
    
    posts = db.relationship('Post', secondary=post_tags, back_populates='tags')

# Create tables
with app.app_context():
    db.create_all()
    
    # Seed data ถ้ายังไม่มี
    if User.query.count() == 0:
        u1 = User(username='somchai', email='somchai@example.com', bio='Python developer')
        u2 = User(username='somying', email='somying@example.com', bio='Flask enthusiast')
        db.session.add_all([u1, u2])
        db.session.flush()
        
        t_py = Tag(name='python')
        t_fl = Tag(name='flask')
        t_db = Tag(name='database')
        db.session.add_all([t_py, t_fl, t_db])
        
        p1 = Post(title='Getting Started with Flask', slug='getting-started-flask',
                  content='Flask is a micro web framework...', excerpt='Learn Flask basics',
                  is_published=True, user_id=u1.id)
        p1.tags = [t_py, t_fl]
        
        p2 = Post(title='SQLAlchemy Tutorial', slug='sqlalchemy-tutorial',
                  content='SQLAlchemy is a Python ORM...', excerpt='Learn SQLAlchemy',
                  is_published=True, user_id=u1.id)
        p2.tags = [t_py, t_db]
        
        db.session.add_all([p1, p2])
        db.session.commit()

# API Routes
@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify({'users': [u.to_dict() for u in users]})

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data or not data.get('username') or not data.get('email'):
        return jsonify({'error': 'ต้องระบุ username และ email'}), 400
    
    if User.query.filter_by(username=data['username']).first():
        return jsonify({'error': 'Username มีอยู่แล้ว'}), 409
    
    user = User(username=data['username'], email=data['email'],
                bio=data.get('bio'))
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict()), 201

@app.route('/api/posts', methods=['GET'])
def get_posts():
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 10, type=int)
    tag = request.args.get('tag')
    author = request.args.get('author')
    
    query = Post.query.filter_by(is_published=True)\
                     .order_by(Post.created_at.desc())
    
    if tag:
        query = query.join(Post.tags).filter(Tag.name == tag)
    
    if author:
        query = query.join(Post.author).filter(User.username == author)
    
    pagination = query.paginate(page=page, per_page=per_page, error_out=False)
    
    return jsonify({
        'posts': [p.to_dict() for p in pagination.items],
        'total': pagination.total,
        'pages': pagination.pages,
        'current_page': page
    })

@app.route('/api/posts/<slug>', methods=['GET'])
def get_post(slug):
    post = Post.query.filter_by(slug=slug, is_published=True).first_or_404()
    post.view_count += 1
    db.session.commit()
    
    post_data = post.to_dict(include_content=True)
    post_data['comments'] = [c.to_dict() for c in post.comments
                              .order_by(Comment.created_at.desc()).all()]
    
    return jsonify(post_data)

@app.route('/api/posts', methods=['POST'])
def create_post():
    data = request.get_json()
    
    required_fields = ['title', 'content', 'user_id']
    for field in required_fields:
        if not data.get(field):
            return jsonify({'error': f'ต้องระบุ {field}'}), 400
    
    user = User.query.get(data['user_id'])
    if not user:
        return jsonify({'error': 'ไม่พบ user'}), 404
    
    # สร้าง slug จาก title
    import re
    slug = re.sub(r'[^a-z0-9-]', '', data['title'].lower().replace(' ', '-'))
    slug = re.sub(r'-+', '-', slug)
    
    # ตรวจสอบ slug ซ้ำ
    if Post.query.filter_by(slug=slug).first():
        import time
        slug = f'{slug}-{int(time.time())}'
    
    post = Post(
        title=data['title'],
        slug=slug,
        content=data['content'],
        excerpt=data.get('excerpt', data['content'][:200]),
        user_id=data['user_id'],
        is_published=data.get('is_published', False)
    )
    
    # เพิ่ม tags
    for tag_name in data.get('tags', []):
        tag = Tag.query.filter_by(name=tag_name).first()
        if not tag:
            tag = Tag(name=tag_name)
            db.session.add(tag)
        post.tags.append(tag)
    
    db.session.add(post)
    db.session.commit()
    return jsonify(post.to_dict(include_content=True)), 201

@app.route('/api/posts/<int:post_id>/comments', methods=['POST'])
def add_comment(post_id):
    post = Post.query.get_or_404(post_id)
    data = request.get_json()
    
    if not data or not data.get('content') or not data.get('user_id'):
        return jsonify({'error': 'ต้องระบุ content และ user_id'}), 400
    
    user = User.query.get(data['user_id'])
    if not user:
        return jsonify({'error': 'ไม่พบ user'}), 404
    
    comment = Comment(
        content=data['content'],
        user_id=data['user_id'],
        post_id=post_id
    )
    db.session.add(comment)
    db.session.commit()
    return jsonify(comment.to_dict()), 201

@app.route('/api/stats', methods=['GET'])
def get_stats():
    from sqlalchemy import func
    
    total_users = User.query.count()
    total_posts = Post.query.filter_by(is_published=True).count()
    total_comments = Comment.query.count()
    
    # Top authors
    top_authors = db.session.query(
        User.username,
        func.count(Post.id).label('post_count')
    ).join(Post, User.id == Post.user_id)\
     .filter(Post.is_published == True)\
     .group_by(User.id)\
     .order_by(func.count(Post.id).desc())\
     .limit(5).all()
    
    return jsonify({
        'total_users': total_users,
        'total_posts': total_posts,
        'total_comments': total_comments,
        'top_authors': [{'username': u, 'posts': c} for u, c in top_authors]
    })

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 12. แบบฝึกหัด

### ข้อที่ 1: User CRUD API

สร้าง REST API สำหรับจัดการ Users ครบ CRUD พร้อม pagination

**เฉลย:**

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    full_name = db.Column(db.String(200))
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    def to_dict(self):
        return {
            'id': self.id, 'username': self.username,
            'email': self.email, 'full_name': self.full_name,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat()
        }

with app.app_context():
    db.create_all()

@app.route('/api/users', methods=['GET'])
def get_users():
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 10, type=int)
    active_only = request.args.get('active', 'false').lower() == 'true'
    
    q = User.query
    if active_only:
        q = q.filter_by(is_active=True)
    
    pag = q.order_by(User.created_at.desc()).paginate(page=page, per_page=per_page, error_out=False)
    return jsonify({
        'users': [u.to_dict() for u in pag.items],
        'total': pag.total, 'pages': pag.pages, 'page': page
    })

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json() or {}
    if not data.get('username') or not data.get('email'):
        return jsonify({'error': 'ต้องระบุ username และ email'}), 400
    
    if User.query.filter_by(username=data['username']).first():
        return jsonify({'error': 'Username นี้มีแล้ว'}), 409
    
    user = User(username=data['username'], email=data['email'],
                full_name=data.get('full_name'))
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict()), 201

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    user = User.query.get_or_404(user_id)
    data = request.get_json() or {}
    
    for field in ['full_name', 'is_active']:
        if field in data:
            setattr(user, field, data[field])
    
    if 'email' in data:
        existing = User.query.filter_by(email=data['email']).first()
        if existing and existing.id != user_id:
            return jsonify({'error': 'Email นี้มีแล้ว'}), 409
        user.email = data['email']
    
    db.session.commit()
    return jsonify(user.to_dict())

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = User.query.get_or_404(user_id)
    db.session.delete(user)
    db.session.commit()
    return '', 204

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 2: Product Inventory

สร้างระบบจัดการสินค้า (Product) กับ Category (One-to-Many)

**เฉลย:**

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///inventory.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class Category(db.Model):
    __tablename__ = 'categories'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True, nullable=False)
    products = db.relationship('Product', backref='category', lazy='dynamic')
    
    def to_dict(self):
        return {'id': self.id, 'name': self.name, 'product_count': self.products.count()}

class Product(db.Model):
    __tablename__ = 'products'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(200), nullable=False)
    price = db.Column(db.Numeric(10, 2), nullable=False)
    stock = db.Column(db.Integer, default=0)
    category_id = db.Column(db.Integer, db.ForeignKey('categories.id'))
    
    def to_dict(self):
        return {
            'id': self.id, 'name': self.name,
            'price': float(self.price), 'stock': self.stock,
            'category': self.category.name if self.category else None
        }

with app.app_context():
    db.create_all()
    if Category.query.count() == 0:
        cats = [Category(name='Electronics'), Category(name='Books'), Category(name='Clothing')]
        db.session.add_all(cats)
        db.session.commit()
        
        products = [
            Product(name='Laptop', price=25000, stock=10, category_id=1),
            Product(name='Python Book', price=500, stock=50, category_id=2),
            Product(name='T-Shirt', price=200, stock=100, category_id=3),
        ]
        db.session.add_all(products)
        db.session.commit()

@app.route('/api/products')
def get_products():
    cat_id = request.args.get('category', type=int)
    min_price = request.args.get('min_price', type=float)
    max_price = request.args.get('max_price', type=float)
    in_stock = request.args.get('in_stock', 'false').lower() == 'true'
    
    q = Product.query
    if cat_id: q = q.filter_by(category_id=cat_id)
    if min_price: q = q.filter(Product.price >= min_price)
    if max_price: q = q.filter(Product.price <= max_price)
    if in_stock: q = q.filter(Product.stock > 0)
    
    return jsonify({'products': [p.to_dict() for p in q.all()]})

@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.get_json() or {}
    if not data.get('name') or data.get('price') is None:
        return jsonify({'error': 'ต้องระบุ name และ price'}), 400
    
    product = Product(
        name=data['name'], price=data['price'],
        stock=data.get('stock', 0),
        category_id=data.get('category_id')
    )
    db.session.add(product)
    db.session.commit()
    return jsonify(product.to_dict()), 201

@app.route('/api/products/<int:pid>/stock', methods=['PATCH'])
def update_stock(pid):
    product = Product.query.get_or_404(pid)
    data = request.get_json() or {}
    action = data.get('action', 'set')
    amount = data.get('amount', 0)
    
    if action == 'add': product.stock += amount
    elif action == 'subtract':
        if product.stock < amount:
            return jsonify({'error': 'stock ไม่พอ'}), 400
        product.stock -= amount
    else: product.stock = amount
    
    db.session.commit()
    return jsonify(product.to_dict())

if __name__ == '__main__':
    app.run(debug=True)
```

### ข้อที่ 3-8 (สรุป)

```python
# ข้อที่ 3: Many-to-Many (Students & Courses)
# สร้าง Student, Course models พร้อม enrollment
# ข้อที่ 4: Search & Filter
# API ที่ค้นหาได้หลายเงื่อนไข, sort ได้
# ข้อที่ 5: Soft Delete
# User model ที่ soft delete ด้วย deleted_at
# ข้อที่ 6: Pagination + Sorting
# API ที่ paginate และ sort ได้หลาย columns
# ข้อที่ 7: Statistics/Analytics
# endpoint ที่ดึงสถิติด้วย aggregation queries
# ข้อที่ 8: Database Seeder + CLI Commands
# seed data พร้อม flask CLI command

# ตัวอย่าง Statistics API
from sqlalchemy import func

@app.route('/api/stats')
def stats():
    return jsonify({
        'total_users': User.query.count(),
        'active_users': User.query.filter_by(is_active=True).count(),
        'total_posts': Post.query.count(),
        'published_posts': Post.query.filter_by(is_published=True).count(),
        'avg_posts_per_user': db.session.query(
            func.avg(
                db.session.query(func.count(Post.id))
                .filter(Post.user_id == User.id)
                .correlate(User)
                .as_scalar()
            )
        ).scalar()
    })
```

---

## สรุป

ใน Part 54 นี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|---------------|
| Setup | Flask-SQLAlchemy, DB URIs |
| Models | Column types, options, methods |
| Relationships | One-to-Many, One-to-One, Many-to-Many |
| CRUD | Create, Read, Update, Delete |
| Queries | filter, order_by, limit, offset, pagination |
| Flask-Migrate | init, migrate, upgrade, downgrade |
| Association Tables | Simple table, Association Object |
| Initialization | create_all, seed data, CLI commands |
| Optimization | N+1 problem, indexes, eager loading |
| Loading | lazy, eager, dynamic |

### ขั้นตอนต่อไป

ใน Part 55 เราจะเรียนรู้:
- Session management
- Flask-Login extension
- Authentication flow
- Password hashing
- JWT tokens
- OAuth2 basics

---

*เขียนโดยหลักสูตร Python Advanced - Part 54*
