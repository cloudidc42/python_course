# Part 32: Database - SQLite & SQLAlchemy

## บทนำ

ฐานข้อมูลเป็นหัวใจของแอปพลิเคชันส่วนใหญ่ Python มีเครื่องมือที่ยอดเยี่ยมสำหรับทำงานกับฐานข้อมูล ตั้งแต่ SQLite ที่ built-in มาในภาษาจนถึง SQLAlchemy ORM ที่ทรงพลัง ในบทนี้เราจะเรียนรู้ทั้งสองแบบจากพื้นฐานจนถึงระดับ advanced

---

## 1. SQLite พื้นฐาน

### SQLite คืออะไร?

SQLite เป็น relational database ที่เบา ไม่ต้องติดตั้ง server ข้อมูลถูกเก็บในไฟล์ `.db` ไฟล์เดียว เหมาะสำหรับ:
- Development และ testing
- แอปพลิเคชัน desktop
- Embedded systems
- โปรเจกต์ขนาดเล็ก-กลาง

```python
import sqlite3

# ตัวอย่างที่ 1: เชื่อมต่อ SQLite database
# :memory: สร้าง database ใน RAM (ชั่วคราว)
conn = sqlite3.connect(':memory:')
print("เชื่อมต่อ in-memory database สำเร็จ!")
conn.close()

# เชื่อมต่อ (หรือสร้าง) file database
conn = sqlite3.connect('mydb.db')
print("เชื่อมต่อ file database สำเร็จ!")
conn.close()
```

---

## 2. sqlite3 Module - CRUD Operations

### Create Table

```python
import sqlite3

# ตัวอย่างที่ 2: สร้างตาราง
conn = sqlite3.connect('school.db')
cursor = conn.cursor()

cursor.execute('''
    CREATE TABLE IF NOT EXISTS students (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER,
        email TEXT UNIQUE,
        gpa REAL DEFAULT 0.0,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
''')

cursor.execute('''
    CREATE TABLE IF NOT EXISTS courses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        credits INTEGER,
        description TEXT
    )
''')

cursor.execute('''
    CREATE TABLE IF NOT EXISTS enrollments (
        student_id INTEGER,
        course_id INTEGER,
        grade REAL,
        enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (student_id, course_id),
        FOREIGN KEY (student_id) REFERENCES students(id),
        FOREIGN KEY (course_id) REFERENCES courses(id)
    )
''')

conn.commit()
print("สร้างตารางสำเร็จ!")
conn.close()
```

### Insert Data

```python
import sqlite3

# ตัวอย่างที่ 3: INSERT ข้อมูล
conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# INSERT แบบ single row
cursor.execute(
    "INSERT INTO students (name, age, email, gpa) VALUES (?, ?, ?, ?)",
    ("Alice Johnson", 20, "alice@university.edu", 3.8)
)

# INSERT หลาย rows พร้อมกัน
students = [
    ("Bob Smith", 22, "bob@university.edu", 3.5),
    ("Charlie Brown", 21, "charlie@university.edu", 3.6),
    ("Diana Prince", 20, "diana@university.edu", 3.9),
    ("Eve Wilson", 23, "eve@university.edu", 3.2)
]

cursor.executemany(
    "INSERT INTO students (name, age, email, gpa) VALUES (?, ?, ?, ?)",
    students
)

conn.commit()
print(f"เพิ่มนักเรียน {cursor.rowcount} คน")
print(f"Last inserted ID: {cursor.lastrowid}")
conn.close()
```

### Read Data

```python
import sqlite3

# ตัวอย่างที่ 4: SELECT ข้อมูลต่างๆ
conn = sqlite3.connect('school.db')
conn.row_factory = sqlite3.Row  # ทำให้เข้าถึงด้วย column name ได้
cursor = conn.cursor()

# fetchall() - ดึงข้อมูลทั้งหมด
cursor.execute("SELECT * FROM students ORDER BY gpa DESC")
students = cursor.fetchall()

print("รายชื่อนักเรียนทั้งหมด:")
for student in students:
    print(f"  {student['name']} (GPA: {student['gpa']})")

# fetchone() - ดึงแถวเดียว
cursor.execute("SELECT * FROM students WHERE gpa > ?", (3.7,))
top_student = cursor.fetchone()
if top_student:
    print(f"\nนักเรียนเกรดดี: {top_student['name']}")

# COUNT และ aggregations
cursor.execute("SELECT COUNT(*), AVG(gpa), MAX(gpa), MIN(gpa) FROM students")
stats = cursor.fetchone()
print(f"\nสถิติ: นักเรียน {stats[0]} คน, GPA เฉลี่ย {stats[1]:.2f}")

conn.close()
```

### Update Data

```python
import sqlite3

# ตัวอย่างที่ 5: UPDATE ข้อมูล
conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# อัพเดท row เดียว
cursor.execute(
    "UPDATE students SET gpa = ? WHERE email = ?",
    (4.0, "alice@university.edu")
)
print(f"อัพเดท {cursor.rowcount} แถว")

# อัพเดทหลาย rows ด้วย condition
cursor.execute(
    "UPDATE students SET age = age + 1 WHERE age < 21"
)
print(f"อัพเดทอายุ {cursor.rowcount} นักเรียน")

conn.commit()
conn.close()
```

### Delete Data

```python
import sqlite3

# ตัวอย่างที่ 6: DELETE ข้อมูล
conn = sqlite3.connect('school.db')
cursor = conn.cursor()

# ลบแถวที่ระบุ
cursor.execute("DELETE FROM students WHERE email = ?", ("eve@university.edu",))
print(f"ลบ {cursor.rowcount} แถว")

# ลบแบบมี condition
cursor.execute("DELETE FROM students WHERE gpa < ?", (3.0,))
print(f"ลบนักเรียน GPA ต่ำ {cursor.rowcount} คน")

conn.commit()
conn.close()
```

---

## 3. Parameterized Queries - ป้องกัน SQL Injection

```python
import sqlite3

# ตัวอย่างที่ 7: วิธีที่ผิด (เสี่ยง SQL Injection!)
def bad_search(name):
    conn = sqlite3.connect('school.db')
    cursor = conn.cursor()
    # อย่าทำแบบนี้!!! 
    # SQL Injection: ถ้า name = "' OR '1'='1" จะได้ข้อมูลทั้งหมด!
    query = f"SELECT * FROM students WHERE name = '{name}'"
    cursor.execute(query)
    return cursor.fetchall()

# วิธีที่ถูกต้อง - ใช้ parameterized queries
def safe_search(name):
    conn = sqlite3.connect('school.db')
    cursor = conn.cursor()
    # ? เป็น placeholder ที่ปลอดภัย
    cursor.execute("SELECT * FROM students WHERE name = ?", (name,))
    return cursor.fetchall()

# Named parameters
def search_by_criteria(min_gpa, max_age):
    conn = sqlite3.connect('school.db')
    cursor = conn.cursor()
    cursor.execute(
        "SELECT * FROM students WHERE gpa >= :min_gpa AND age <= :max_age",
        {'min_gpa': min_gpa, 'max_age': max_age}
    )
    return cursor.fetchall()

results = safe_search("Alice Johnson")
print("ค้นหาด้วย safe query:", results)
```

---

## 4. Transactions

```python
import sqlite3

# ตัวอย่างที่ 8: Transaction management
def transfer_credits(from_student_id, to_student_id, course_id, conn):
    """โอนการลงทะเบียนจากนักเรียนหนึ่งไปอีกคน - ต้องทำแบบ atomic"""
    cursor = conn.cursor()
    
    try:
        # เริ่ม transaction (explicit)
        cursor.execute("BEGIN TRANSACTION")
        
        # ตรวจสอบว่า from_student มีการลงทะเบียน
        cursor.execute(
            "SELECT * FROM enrollments WHERE student_id=? AND course_id=?",
            (from_student_id, course_id)
        )
        enrollment = cursor.fetchone()
        
        if not enrollment:
            raise ValueError(f"Student {from_student_id} not enrolled in course {course_id}")
        
        # ลบการลงทะเบียนเดิม
        cursor.execute(
            "DELETE FROM enrollments WHERE student_id=? AND course_id=?",
            (from_student_id, course_id)
        )
        
        # เพิ่มการลงทะเบียนใหม่
        cursor.execute(
            "INSERT INTO enrollments (student_id, course_id) VALUES (?, ?)",
            (to_student_id, course_id)
        )
        
        # COMMIT ถ้าสำเร็จทุก step
        conn.commit()
        print("โอนสำเร็จ!")
        
    except Exception as e:
        # ROLLBACK ถ้าเกิด error
        conn.rollback()
        print(f"เกิดข้อผิดพลาด: {e}, ROLLBACK แล้ว")
        raise
```

---

## 5. Context Manager กับ Database

```python
import sqlite3
from contextlib import contextmanager

# ตัวอย่างที่ 9: ใช้ with statement กับ connection
with sqlite3.connect('school.db') as conn:
    # conn.commit() จะถูกเรียกอัตโนมัติถ้าไม่มี error
    # conn.rollback() ถ้ามี exception
    cursor = conn.cursor()
    cursor.execute("INSERT INTO courses (name, credits) VALUES (?, ?)", ("Python 101", 3))
    # ไม่ต้องเรียก commit() เอง!

print("เพิ่ม course สำเร็จ!")
```

```python
import sqlite3
from contextlib import contextmanager

# ตัวอย่างที่ 10: Custom Database Context Manager
@contextmanager
def get_db_connection(db_path):
    """Context manager สำหรับ database connection"""
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")  # เปิด foreign key support
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

# ใช้งาน
with get_db_connection('school.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM students")
    count = cursor.fetchone()[0]
    print(f"จำนวนนักเรียน: {count}")
```

```python
import sqlite3

# ตัวอย่างที่ 11: Database Manager class
class DatabaseManager:
    def __init__(self, db_path):
        self.db_path = db_path
        self._conn = None
    
    def __enter__(self):
        self._conn = sqlite3.connect(self.db_path)
        self._conn.row_factory = sqlite3.Row
        self._conn.execute("PRAGMA foreign_keys = ON")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            self._conn.commit()
        else:
            self._conn.rollback()
        self._conn.close()
        return False
    
    def execute(self, query, params=()):
        return self._conn.execute(query, params)
    
    def fetchall(self, query, params=()):
        return self._conn.execute(query, params).fetchall()
    
    def fetchone(self, query, params=()):
        return self._conn.execute(query, params).fetchone()

# ใช้งาน
with DatabaseManager('school.db') as db:
    students = db.fetchall("SELECT name, gpa FROM students WHERE gpa > ?", (3.5,))
    for s in students:
        print(f"{s['name']}: {s['gpa']}")
```

---

## 6. Advanced SQLite Queries

```python
import sqlite3

# ตัวอย่างที่ 12: JOIN queries
conn = sqlite3.connect('school.db')
conn.row_factory = sqlite3.Row

# ใส่ข้อมูลตัวอย่าง
with conn:
    conn.execute("INSERT OR IGNORE INTO courses VALUES (1, 'Python Programming', 3, 'Learn Python')")
    conn.execute("INSERT OR IGNORE INTO courses VALUES (2, 'Database Design', 3, 'Learn SQL')")
    conn.execute("INSERT OR IGNORE INTO enrollments (student_id, course_id, grade) VALUES (1, 1, 85.5)")
    conn.execute("INSERT OR IGNORE INTO enrollments (student_id, course_id, grade) VALUES (1, 2, 92.0)")
    conn.execute("INSERT OR IGNORE INTO enrollments (student_id, course_id, grade) VALUES (2, 1, 78.0)")

cursor = conn.cursor()

# INNER JOIN
cursor.execute('''
    SELECT s.name, c.name as course, e.grade
    FROM students s
    JOIN enrollments e ON s.id = e.student_id
    JOIN courses c ON e.course_id = c.id
    ORDER BY s.name, c.name
''')

print("รายการลงทะเบียน:")
for row in cursor.fetchall():
    print(f"  {row['name']} - {row['course']}: {row['grade']}")

# GROUP BY + aggregation
cursor.execute('''
    SELECT s.name, 
           COUNT(e.course_id) as num_courses,
           AVG(e.grade) as avg_grade
    FROM students s
    LEFT JOIN enrollments e ON s.id = e.student_id
    GROUP BY s.id, s.name
    HAVING num_courses > 0
    ORDER BY avg_grade DESC
''')

print("\nสรุปผลการเรียน:")
for row in cursor.fetchall():
    print(f"  {row['name']}: {row['num_courses']} วิชา, เฉลี่ย {row['avg_grade']:.1f}")

conn.close()
```

---

## 7. SQLAlchemy Core

SQLAlchemy Core ให้ใช้ SQL ในรูปแบบ Python objects แบบ type-safe

```python
# pip install sqlalchemy

from sqlalchemy import create_engine, text, MetaData, Table, Column, Integer, String, Float

# ตัวอย่างที่ 13: SQLAlchemy Core - สร้าง engine
engine = create_engine('sqlite:///core_demo.db', echo=True)

# สร้าง metadata
metadata = MetaData()

# Define tables
users_table = Table(
    'users', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(100), nullable=False),
    Column('email', String(200), unique=True),
    Column('age', Integer),
    Column('score', Float, default=0.0)
)

# สร้าง tables
metadata.create_all(engine)
print("สร้างตาราง users สำเร็จ!")
```

```python
from sqlalchemy import create_engine, text

# ตัวอย่างที่ 14: SQLAlchemy Core - INSERT และ SELECT
engine = create_engine('sqlite:///core_demo.db')

# INSERT ด้วย text() 
with engine.connect() as conn:
    conn.execute(
        text("INSERT INTO users (name, email, age) VALUES (:name, :email, :age)"),
        {"name": "Alice", "email": "alice@test.com", "age": 25}
    )
    conn.commit()

# SELECT
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users"))
    for row in result:
        print(dict(row._mapping))
```

---

## 8. SQLAlchemy ORM

ORM (Object-Relational Mapping) ให้เราทำงานกับ database ผ่าน Python objects

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime, ForeignKey
from sqlalchemy.orm import DeclarativeBase, relationship, Session
from datetime import datetime

# ตัวอย่างที่ 15: Declaring Models
class Base(DeclarativeBase):
    pass

class Student(Base):
    __tablename__ = 'students_orm'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(200), unique=True, nullable=False)
    age = Column(Integer)
    gpa = Column(Float, default=0.0)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationship
    enrollments = relationship("Enrollment", back_populates="student")
    
    def __repr__(self):
        return f"<Student(name={self.name!r}, gpa={self.gpa})>"

class Course(Base):
    __tablename__ = 'courses_orm'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    credits = Column(Integer, default=3)
    description = Column(String(500))
    
    enrollments = relationship("Enrollment", back_populates="course")
    
    def __repr__(self):
        return f"<Course(name={self.name!r})>"

class Enrollment(Base):
    __tablename__ = 'enrollments_orm'
    
    id = Column(Integer, primary_key=True)
    student_id = Column(Integer, ForeignKey('students_orm.id'), nullable=False)
    course_id = Column(Integer, ForeignKey('courses_orm.id'), nullable=False)
    grade = Column(Float)
    enrolled_at = Column(DateTime, default=datetime.utcnow)
    
    student = relationship("Student", back_populates="enrollments")
    course = relationship("Course", back_populates="enrollments")

# สร้าง engine และ tables
engine = create_engine('sqlite:///orm_demo.db', echo=False)
Base.metadata.create_all(engine)
print("สร้าง ORM tables สำเร็จ!")
```

---

## 9. Session Management

```python
from sqlalchemy.orm import Session

# ตัวอย่างที่ 16: Session - เพิ่มข้อมูล
with Session(engine) as session:
    # สร้าง objects
    alice = Student(name="Alice Johnson", email="alice@univ.edu", age=20, gpa=3.8)
    bob = Student(name="Bob Smith", email="bob@univ.edu", age=22, gpa=3.5)
    charlie = Student(name="Charlie Brown", email="charlie@univ.edu", age=21, gpa=3.6)
    
    python_course = Course(name="Python Programming", credits=3)
    db_course = Course(name="Database Design", credits=3)
    
    # เพิ่มลง session
    session.add_all([alice, bob, charlie, python_course, db_course])
    session.commit()
    
    print(f"Alice ID: {alice.id}")
    print(f"Python Course ID: {python_course.id}")
```

```python
from sqlalchemy.orm import Session
from sqlalchemy import select

# ตัวอย่างที่ 17: Session - Query ข้อมูล
with Session(engine) as session:
    # Query แบบต่างๆ
    
    # ดึงทั้งหมด
    stmt = select(Student)
    students = session.scalars(stmt).all()
    print(f"นักเรียนทั้งหมด: {len(students)} คน")
    
    # Filter
    stmt = select(Student).where(Student.gpa > 3.6)
    top_students = session.scalars(stmt).all()
    print("\nนักเรียนเกรดดี:")
    for s in top_students:
        print(f"  {s.name}: {s.gpa}")
    
    # Get by primary key
    student = session.get(Student, 1)
    if student:
        print(f"\nนักเรียน ID 1: {student.name}")
    
    # Filter หลาย conditions
    stmt = select(Student).where(
        Student.age >= 20,
        Student.gpa >= 3.5
    ).order_by(Student.gpa.desc())
    
    for s in session.scalars(stmt):
        print(f"  {s.name} (อายุ {s.age}, GPA {s.gpa})")
```

```python
from sqlalchemy.orm import Session
from sqlalchemy import select, func

# ตัวอย่างที่ 18: Session - Update และ Delete
with Session(engine) as session:
    # Update - ดึง object มาแก้ไข
    student = session.get(Student, 1)
    if student:
        student.gpa = 4.0
        session.commit()
        print(f"อัพเดท GPA ของ {student.name} เป็น {student.gpa}")
    
    # Update หลาย rows
    stmt = select(Student).where(Student.age < 21)
    young_students = session.scalars(stmt).all()
    for s in young_students:
        s.age += 1
    session.commit()
    
    # Delete
    student_to_delete = session.get(Student, 1)
    if student_to_delete:
        session.delete(student_to_delete)
        session.commit()
        print("ลบนักเรียนสำเร็จ")
    
    # Aggregate functions
    stmt = select(
        func.count(Student.id).label('count'),
        func.avg(Student.gpa).label('avg_gpa'),
        func.max(Student.gpa).label('max_gpa')
    )
    result = session.execute(stmt).first()
    print(f"\nสถิติ: {result.count} คน, เฉลี่ย {result.avg_gpa:.2f}")
```

---

## 10. Relationships - One-to-Many

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey, Text
from sqlalchemy.orm import DeclarativeBase, relationship, Session, joinedload
from sqlalchemy import select

# ตัวอย่างที่ 19: Blog system - One-to-Many
class BlogBase(DeclarativeBase):
    pass

class Author(BlogBase):
    __tablename__ = 'authors'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(200), unique=True)
    bio = Column(Text)
    
    # One author -> Many posts
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<Author({self.name!r})>"

class Post(BlogBase):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text)
    author_id = Column(Integer, ForeignKey('authors.id'), nullable=False)
    is_published = Column(Integer, default=0)  # 0=draft, 1=published
    
    # Many posts -> One author
    author = relationship("Author", back_populates="posts")
    comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")

class Comment(BlogBase):
    __tablename__ = 'comments'
    
    id = Column(Integer, primary_key=True)
    content = Column(Text, nullable=False)
    author_name = Column(String(100))
    post_id = Column(Integer, ForeignKey('posts.id'), nullable=False)
    
    post = relationship("Post", back_populates="comments")

blog_engine = create_engine('sqlite:///blog.db')
BlogBase.metadata.create_all(blog_engine)

# ใส่ข้อมูล
with Session(blog_engine) as session:
    # สร้าง author พร้อม posts
    alice = Author(
        name="Alice Writer",
        email="alice@blog.com",
        bio="Python enthusiast and tech blogger"
    )
    
    post1 = Post(
        title="Getting Started with Python",
        content="Python is an amazing language...",
        author=alice,
        is_published=1
    )
    
    post2 = Post(
        title="SQLAlchemy Tutorial",
        content="SQLAlchemy makes database operations easy...",
        author=alice,
        is_published=1
    )
    
    comment1 = Comment(content="Great article!", author_name="Bob", post=post1)
    comment2 = Comment(content="Very helpful!", author_name="Charlie", post=post1)
    
    session.add(alice)
    session.commit()
    
    print(f"Author: {alice.name} ({len(alice.posts)} posts)")

# Query with eager loading
with Session(blog_engine) as session:
    stmt = select(Author).options(joinedload(Author.posts))
    authors = session.scalars(stmt).unique().all()
    
    for author in authors:
        print(f"\n{author.name}:")
        for post in author.posts:
            print(f"  - {post.title}")
```

---

## 11. Relationships - Many-to-Many

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey, Table
from sqlalchemy.orm import DeclarativeBase, relationship, Session
from sqlalchemy import select

# ตัวอย่างที่ 20: Many-to-Many ด้วย association table
class SchoolBase(DeclarativeBase):
    pass

# Association table สำหรับ Student-Course (Many-to-Many)
student_course = Table(
    'student_course',
    SchoolBase.metadata,
    Column('student_id', Integer, ForeignKey('students_m2m.id'), primary_key=True),
    Column('course_id', Integer, ForeignKey('courses_m2m.id'), primary_key=True),
    Column('grade', Float)
)

class StudentM2M(SchoolBase):
    __tablename__ = 'students_m2m'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    
    # Many-to-Many: นักเรียนลงทะเบียนหลายวิชา
    courses = relationship("CourseM2M", secondary=student_course, back_populates="students")
    
    def __repr__(self):
        return f"<Student({self.name!r})>"

class CourseM2M(SchoolBase):
    __tablename__ = 'courses_m2m'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    
    # Many-to-Many: วิชามีนักเรียนหลายคน
    students = relationship("StudentM2M", secondary=student_course, back_populates="courses")
    
    def __repr__(self):
        return f"<Course({self.name!r})>"

school_engine = create_engine('sqlite:///school_m2m.db')
SchoolBase.metadata.create_all(school_engine)

# ใส่ข้อมูล
with Session(school_engine) as session:
    alice = StudentM2M(name="Alice")
    bob = StudentM2M(name="Bob")
    charlie = StudentM2M(name="Charlie")
    
    python_course = CourseM2M(name="Python")
    db_course = CourseM2M(name="Database")
    web_course = CourseM2M(name="Web Development")
    
    # Assign ผ่าน relationship
    alice.courses = [python_course, db_course]
    bob.courses = [python_course, web_course]
    charlie.courses = [db_course, web_course]
    
    session.add_all([alice, bob, charlie])
    session.commit()

# Query
with Session(school_engine) as session:
    students = session.scalars(select(StudentM2M)).all()
    for student in students:
        course_names = [c.name for c in student.courses]
        print(f"{student.name}: {', '.join(course_names)}")
```

---

## 12. Advanced ORM Queries

```python
from sqlalchemy.orm import Session
from sqlalchemy import select, and_, or_, func, desc, asc
from sqlalchemy import case

# ตัวอย่างที่ 21: Complex queries ด้วย SQLAlchemy
with Session(engine) as session:
    # AND condition
    stmt = select(Student).where(
        and_(Student.age >= 20, Student.gpa >= 3.5)
    )
    
    # OR condition
    stmt = select(Student).where(
        or_(Student.gpa > 3.8, Student.age < 21)
    )
    
    # LIKE (contains)
    stmt = select(Student).where(Student.name.like('%Alice%'))
    
    # IN condition
    stmt = select(Student).where(Student.age.in_([20, 21, 22]))
    
    # NOT IN
    stmt = select(Student).where(Student.age.not_in([25, 30]))
    
    # BETWEEN
    stmt = select(Student).where(Student.gpa.between(3.5, 4.0))
    
    # NULL check
    stmt = select(Student).where(Student.email.is_(None))
    stmt = select(Student).where(Student.email.is_not(None))
    
    # ORDER BY multiple columns
    stmt = select(Student).order_by(desc(Student.gpa), asc(Student.name))
    
    # LIMIT and OFFSET (pagination)
    stmt = select(Student).order_by(Student.id).limit(10).offset(20)
    
    # CASE expression
    stmt = select(
        Student.name,
        Student.gpa,
        case(
            (Student.gpa >= 3.7, "เกียรตินิยมอันดับ 1"),
            (Student.gpa >= 3.5, "เกียรตินิยมอันดับ 2"),
            else_="ผ่าน"
        ).label("honor")
    )
    
    results = session.execute(stmt).all()
    for row in results:
        print(f"{row.name}: GPA {row.gpa} - {row.honor}")
```

---

## 13. Repository Pattern กับ SQLAlchemy

```python
from sqlalchemy.orm import Session
from sqlalchemy import select
from typing import Optional, List, Type, TypeVar
from sqlalchemy.orm import DeclarativeBase

T = TypeVar('T')

# ตัวอย่างที่ 22: Generic Repository Pattern
class BaseRepository:
    def __init__(self, model_class, session: Session):
        self.model = model_class
        self.session = session
    
    def get_by_id(self, id: int):
        return self.session.get(self.model, id)
    
    def get_all(self):
        return self.session.scalars(select(self.model)).all()
    
    def add(self, obj):
        self.session.add(obj)
        self.session.flush()  # ได้ ID โดยไม่ต้อง commit
        return obj
    
    def delete(self, obj):
        self.session.delete(obj)
    
    def save(self):
        self.session.commit()

class StudentRepository(BaseRepository):
    def __init__(self, session: Session):
        super().__init__(Student, session)
    
    def find_by_email(self, email: str) -> Optional[Student]:
        stmt = select(Student).where(Student.email == email)
        return self.session.scalars(stmt).first()
    
    def find_top_students(self, min_gpa: float) -> List[Student]:
        stmt = select(Student).where(Student.gpa >= min_gpa).order_by(Student.gpa.desc())
        return self.session.scalars(stmt).all()
    
    def get_statistics(self) -> dict:
        stmt = select(
            func.count(Student.id).label('total'),
            func.avg(Student.gpa).label('avg_gpa'),
            func.max(Student.gpa).label('max_gpa')
        )
        result = self.session.execute(stmt).first()
        return {
            'total': result.total,
            'avg_gpa': round(float(result.avg_gpa or 0), 2),
            'max_gpa': result.max_gpa
        }

# ใช้งาน
with Session(engine) as session:
    repo = StudentRepository(session)
    
    # เพิ่มนักเรียนใหม่
    new_student = Student(name="Frank", email="frank@test.edu", gpa=3.7)
    repo.add(new_student)
    repo.save()
    
    # ค้นหา
    stats = repo.get_statistics()
    print(f"สถิติ: {stats}")
```

---

## 14. Alembic Migrations เบื้องต้น

Alembic ช่วยจัดการการเปลี่ยนแปลง database schema ตามเวลา

```bash
# ติดตั้ง alembic
# pip install alembic

# สร้าง alembic project
# alembic init migrations

# สร้าง migration script
# alembic revision --autogenerate -m "add_phone_to_students"

# apply migration
# alembic upgrade head

# rollback migration
# alembic downgrade -1
```

```python
# ตัวอย่างที่ 23: Migration script ตัวอย่าง (env.py)
# ในไฟล์ migrations/env.py
from alembic import context
from sqlalchemy import engine_from_config, pool
from models import Base  # import models ของเรา

def run_migrations_online():
    connectable = engine_from_config(
        context.config.get_section(context.config.config_ini_section),
        prefix='sqlalchemy.',
        poolclass=pool.NullPool,
    )
    
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=Base.metadata
        )
        with context.begin_transaction():
            context.run_migrations()
```

```python
# ตัวอย่างที่ 24: Migration script ตัวอย่าง
# ในไฟล์ migrations/versions/001_add_phone_to_students.py

from alembic import op
import sqlalchemy as sa

def upgrade():
    """เพิ่ม column phone ในตาราง students"""
    op.add_column('students_orm', sa.Column('phone', sa.String(20), nullable=True))
    op.create_index('ix_students_phone', 'students_orm', ['phone'])

def downgrade():
    """ลบ column phone"""
    op.drop_index('ix_students_phone')
    op.drop_column('students_orm', 'phone')
```

---

## 15. Database Connection Pooling

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool, StaticPool, NullPool

# ตัวอย่างที่ 25: Connection Pool configurations
# QueuePool (default สำหรับ non-SQLite)
engine = create_engine(
    'sqlite:///pool_demo.db',
    poolclass=StaticPool,  # สำหรับ SQLite ที่ใช้งาน concurrent
    connect_args={'check_same_thread': False}
)

# สำหรับ PostgreSQL/MySQL
# engine = create_engine(
#     'postgresql://user:pass@localhost/dbname',
#     pool_size=5,           # จำนวน connections ปกติ
#     max_overflow=10,       # connections พิเศษเมื่อ pool เต็ม
#     pool_timeout=30,       # timeout รอ connection (วินาที)
#     pool_recycle=3600,     # recycle connections ทุก 1 ชั่วโมง
#     pool_pre_ping=True,    # ทดสอบ connection ก่อนใช้
# )

print(f"Pool class: {type(engine.pool).__name__}")
print(f"Pool size: {engine.pool.size()}")
```

---

## 16. ตัวอย่างโปรแกรมจริง - Student Database

```python
import sqlite3
from contextlib import contextmanager
from typing import Optional, List
import json

# ตัวอย่างที่ 26: Complete Student Database System
class StudentDB:
    def __init__(self, db_path='students_complete.db'):
        self.db_path = db_path
        self._init_db()
    
    @contextmanager
    def get_conn(self):
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        conn.execute("PRAGMA foreign_keys = ON")
        try:
            yield conn
            conn.commit()
        except Exception:
            conn.rollback()
            raise
        finally:
            conn.close()
    
    def _init_db(self):
        with self.get_conn() as conn:
            conn.executescript('''
                CREATE TABLE IF NOT EXISTS departments (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL UNIQUE,
                    code TEXT NOT NULL UNIQUE
                );
                
                CREATE TABLE IF NOT EXISTS students (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    student_id TEXT NOT NULL UNIQUE,
                    name TEXT NOT NULL,
                    email TEXT UNIQUE,
                    dept_id INTEGER,
                    gpa REAL DEFAULT 0.0,
                    year INTEGER DEFAULT 1,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (dept_id) REFERENCES departments(id)
                );
                
                CREATE INDEX IF NOT EXISTS idx_students_dept ON students(dept_id);
                CREATE INDEX IF NOT EXISTS idx_students_gpa ON students(gpa);
            ''')
    
    def add_department(self, name: str, code: str) -> int:
        with self.get_conn() as conn:
            cursor = conn.execute(
                "INSERT INTO departments (name, code) VALUES (?, ?)",
                (name, code)
            )
            return cursor.lastrowid
    
    def add_student(self, student_id: str, name: str, email: str,
                    dept_code: str, gpa: float = 0.0, year: int = 1) -> int:
        with self.get_conn() as conn:
            # หา department id
            dept = conn.execute(
                "SELECT id FROM departments WHERE code = ?", (dept_code,)
            ).fetchone()
            
            if not dept:
                raise ValueError(f"Department '{dept_code}' not found")
            
            cursor = conn.execute(
                """INSERT INTO students (student_id, name, email, dept_id, gpa, year)
                   VALUES (?, ?, ?, ?, ?, ?)""",
                (student_id, name, email, dept['id'], gpa, year)
            )
            return cursor.lastrowid
    
    def get_department_stats(self) -> List[dict]:
        with self.get_conn() as conn:
            rows = conn.execute('''
                SELECT d.name, d.code,
                       COUNT(s.id) as student_count,
                       ROUND(AVG(s.gpa), 2) as avg_gpa,
                       MAX(s.gpa) as top_gpa
                FROM departments d
                LEFT JOIN students s ON d.id = s.dept_id
                GROUP BY d.id, d.name, d.code
                ORDER BY avg_gpa DESC
            ''').fetchall()
            return [dict(row) for row in rows]
    
    def search_students(self, query: str = None, min_gpa: float = 0.0,
                        dept_code: str = None) -> List[dict]:
        conditions = ["s.gpa >= ?"]
        params = [min_gpa]
        
        if query:
            conditions.append("(s.name LIKE ? OR s.student_id LIKE ?)")
            params.extend([f'%{query}%', f'%{query}%'])
        
        if dept_code:
            conditions.append("d.code = ?")
            params.append(dept_code)
        
        where_clause = " AND ".join(conditions)
        
        with self.get_conn() as conn:
            rows = conn.execute(f'''
                SELECT s.student_id, s.name, s.email, s.gpa, s.year,
                       d.name as department, d.code as dept_code
                FROM students s
                LEFT JOIN departments d ON s.dept_id = d.id
                WHERE {where_clause}
                ORDER BY s.gpa DESC
            ''', params).fetchall()
            return [dict(row) for row in rows]

# ใช้งาน
db = StudentDB()

# เพิ่ม departments
cs_id = db.add_department("Computer Science", "CS")
ee_id = db.add_department("Electrical Engineering", "EE")
me_id = db.add_department("Mechanical Engineering", "ME")

# เพิ่ม students
db.add_student("CS001", "Alice Johnson", "alice@univ.edu", "CS", gpa=3.9, year=2)
db.add_student("CS002", "Bob Smith", "bob@univ.edu", "CS", gpa=3.5, year=3)
db.add_student("EE001", "Charlie Brown", "charlie@univ.edu", "EE", gpa=3.7, year=2)
db.add_student("ME001", "Diana Prince", "diana@univ.edu", "ME", gpa=3.8, year=1)

# Query
stats = db.get_department_stats()
print("สถิติแต่ละภาควิชา:")
for dept in stats:
    print(f"  {dept['name']} ({dept['code']}): {dept['student_count']} คน, GPA เฉลี่ย {dept['avg_gpa']}")

results = db.search_students(min_gpa=3.7, dept_code="CS")
print(f"\nนักเรียน CS ที่มี GPA >= 3.7: {len(results)} คน")
for s in results:
    print(f"  {s['student_id']}: {s['name']} (GPA {s['gpa']})")
```

---

## 17. ตัวอย่างโปรแกรมจริง - Blog Database ด้วย SQLAlchemy ORM

```python
from sqlalchemy import create_engine, Column, Integer, String, Text, DateTime, Boolean, ForeignKey, func
from sqlalchemy.orm import DeclarativeBase, relationship, Session, selectinload
from sqlalchemy import select, desc
from datetime import datetime
from typing import Optional

# ตัวอย่างที่ 27: Blog Database
class BlogBase2(DeclarativeBase):
    pass

class Tag(BlogBase2):
    __tablename__ = 'tags'
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    posts = relationship("BlogPost", secondary="post_tags", back_populates="tags")

from sqlalchemy import Table
post_tags = Table('post_tags', BlogBase2.metadata,
    Column('post_id', Integer, ForeignKey('blog_posts.id'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id'), primary_key=True)
)

class BlogAuthor(BlogBase2):
    __tablename__ = 'blog_authors'
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(200), unique=True, nullable=False)
    full_name = Column(String(100))
    posts = relationship("BlogPost", back_populates="author")

class BlogPost(BlogBase2):
    __tablename__ = 'blog_posts'
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    slug = Column(String(200), unique=True, nullable=False)
    content = Column(Text)
    summary = Column(String(500))
    is_published = Column(Boolean, default=False)
    view_count = Column(Integer, default=0)
    author_id = Column(Integer, ForeignKey('blog_authors.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    published_at = Column(DateTime)
    
    author = relationship("BlogAuthor", back_populates="posts")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")
    comments = relationship("BlogComment", back_populates="post", cascade="all, delete-orphan")

class BlogComment(BlogBase2):
    __tablename__ = 'blog_comments'
    id = Column(Integer, primary_key=True)
    content = Column(Text, nullable=False)
    author_name = Column(String(100))
    post_id = Column(Integer, ForeignKey('blog_posts.id'), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    post = relationship("BlogPost", back_populates="comments")

# Blog Service
class BlogService:
    def __init__(self, engine):
        self.engine = engine
    
    def create_post(self, title: str, content: str, author_id: int,
                    tag_names: list = None, publish: bool = False) -> BlogPost:
        slug = title.lower().replace(' ', '-').replace('/', '-')
        
        with Session(self.engine) as session:
            post = BlogPost(
                title=title,
                slug=slug,
                content=content,
                author_id=author_id,
                is_published=publish,
                published_at=datetime.utcnow() if publish else None
            )
            
            if tag_names:
                for tag_name in tag_names:
                    tag = session.scalars(
                        select(Tag).where(Tag.name == tag_name)
                    ).first()
                    if not tag:
                        tag = Tag(name=tag_name)
                        session.add(tag)
                    post.tags.append(tag)
            
            session.add(post)
            session.commit()
            session.refresh(post)
            return post
    
    def get_published_posts(self, page: int = 1, per_page: int = 10) -> list:
        with Session(self.engine) as session:
            stmt = (
                select(BlogPost)
                .where(BlogPost.is_published == True)
                .options(selectinload(BlogPost.author), selectinload(BlogPost.tags))
                .order_by(desc(BlogPost.published_at))
                .limit(per_page)
                .offset((page - 1) * per_page)
            )
            return session.scalars(stmt).all()
    
    def get_post_stats(self) -> dict:
        with Session(self.engine) as session:
            result = session.execute(
                select(
                    func.count(BlogPost.id).label('total'),
                    func.sum(case((BlogPost.is_published == True, 1), else_=0)).label('published'),
                    func.sum(BlogPost.view_count).label('total_views')
                )
            ).first()
            return dict(result._mapping)

blog_engine = create_engine('sqlite:///blog2.db')
BlogBase2.metadata.create_all(blog_engine)

# ใช้งาน
with Session(blog_engine) as session:
    author = BlogAuthor(username="alice", email="alice@blog.com", full_name="Alice Writer")
    session.add(author)
    session.commit()
    author_id = author.id

blog_service = BlogService(blog_engine)
post = blog_service.create_post(
    "Python Tips for Beginners",
    "Here are some great Python tips...",
    author_id=author_id,
    tag_names=["python", "tutorial", "beginner"],
    publish=True
)
print(f"สร้างโพสต์: {post.title} (ID: {post.id})")
```

---

## 18. ตัวอย่างโปรแกรมจริง - Inventory System

```python
import sqlite3
from contextlib import contextmanager
from datetime import datetime
from typing import List, Optional

# ตัวอย่างที่ 28: Inventory Management System
class InventorySystem:
    def __init__(self, db_path='inventory.db'):
        self.db_path = db_path
        self._init_db()
    
    @contextmanager
    def db(self):
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        try:
            yield conn
            conn.commit()
        except Exception:
            conn.rollback()
            raise
        finally:
            conn.close()
    
    def _init_db(self):
        with self.db() as conn:
            conn.executescript('''
                CREATE TABLE IF NOT EXISTS products (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    sku TEXT UNIQUE NOT NULL,
                    name TEXT NOT NULL,
                    category TEXT,
                    price REAL NOT NULL,
                    cost REAL NOT NULL,
                    stock INTEGER DEFAULT 0,
                    reorder_point INTEGER DEFAULT 10,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                );
                
                CREATE TABLE IF NOT EXISTS stock_movements (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    product_id INTEGER NOT NULL,
                    movement_type TEXT NOT NULL,  -- 'IN', 'OUT', 'ADJUSTMENT'
                    quantity INTEGER NOT NULL,
                    reference TEXT,  -- เลขที่เอกสาร
                    notes TEXT,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (product_id) REFERENCES products(id)
                );
            ''')
    
    def add_product(self, sku, name, price, cost, category=None, 
                    initial_stock=0, reorder_point=10) -> int:
        with self.db() as conn:
            cursor = conn.execute('''
                INSERT INTO products (sku, name, category, price, cost, stock, reorder_point)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            ''', (sku, name, category, price, cost, initial_stock, reorder_point))
            
            if initial_stock > 0:
                conn.execute('''
                    INSERT INTO stock_movements (product_id, movement_type, quantity, notes)
                    VALUES (?, 'IN', ?, 'Initial stock')
                ''', (cursor.lastrowid, initial_stock))
            
            return cursor.lastrowid
    
    def receive_stock(self, sku: str, quantity: int, reference: str = None) -> dict:
        """รับสินค้าเข้าคลัง"""
        with self.db() as conn:
            product = conn.execute(
                "SELECT * FROM products WHERE sku = ?", (sku,)
            ).fetchone()
            
            if not product:
                raise ValueError(f"Product '{sku}' not found")
            
            conn.execute(
                "UPDATE products SET stock = stock + ? WHERE sku = ?",
                (quantity, sku)
            )
            conn.execute('''
                INSERT INTO stock_movements (product_id, movement_type, quantity, reference)
                VALUES (?, 'IN', ?, ?)
            ''', (product['id'], quantity, reference))
            
            new_stock = product['stock'] + quantity
            return {'sku': sku, 'added': quantity, 'new_stock': new_stock}
    
    def issue_stock(self, sku: str, quantity: int, reference: str = None) -> dict:
        """จ่ายสินค้าออกจากคลัง"""
        with self.db() as conn:
            product = conn.execute(
                "SELECT * FROM products WHERE sku = ?", (sku,)
            ).fetchone()
            
            if not product:
                raise ValueError(f"Product '{sku}' not found")
            
            if product['stock'] < quantity:
                raise ValueError(
                    f"Insufficient stock: available {product['stock']}, requested {quantity}"
                )
            
            conn.execute(
                "UPDATE products SET stock = stock - ? WHERE sku = ?",
                (quantity, sku)
            )
            conn.execute('''
                INSERT INTO stock_movements (product_id, movement_type, quantity, reference)
                VALUES (?, 'OUT', ?, ?)
            ''', (product['id'], quantity, reference))
            
            new_stock = product['stock'] - quantity
            return {'sku': sku, 'issued': quantity, 'new_stock': new_stock}
    
    def get_low_stock_alert(self) -> List[dict]:
        """รายการสินค้าที่ stock ต่ำกว่า reorder point"""
        with self.db() as conn:
            rows = conn.execute('''
                SELECT sku, name, stock, reorder_point,
                       (reorder_point - stock) as shortage
                FROM products
                WHERE stock <= reorder_point
                ORDER BY shortage DESC
            ''').fetchall()
            return [dict(row) for row in rows]
    
    def get_inventory_value(self) -> dict:
        """คำนวณมูลค่าสินค้าคงคลัง"""
        with self.db() as conn:
            result = conn.execute('''
                SELECT 
                    COUNT(*) as product_count,
                    SUM(stock * cost) as total_cost,
                    SUM(stock * price) as total_retail_value,
                    SUM(stock * (price - cost)) as potential_profit
                FROM products
            ''').fetchone()
            return dict(result)

# ใช้งาน
inv = InventorySystem()

# เพิ่มสินค้า
inv.add_product('IPHONE15', 'iPhone 15', 35000, 28000, 'Electronics', initial_stock=20)
inv.add_product('IPAD', 'iPad Pro', 32000, 25000, 'Electronics', initial_stock=15)
inv.add_product('MACBOOK', 'MacBook Pro', 89000, 70000, 'Electronics', initial_stock=5, reorder_point=3)
inv.add_product('PYTHON_BOOK', 'Python Programming', 500, 200, 'Books', initial_stock=100)

# รับสินค้า
result = inv.receive_stock('IPHONE15', 50, 'PO-2024-001')
print(f"รับสินค้า: stock ใหม่ = {result['new_stock']}")

# จ่ายสินค้า
result = inv.issue_stock('MACBOOK', 4, 'SO-2024-001')
print(f"จ่ายสินค้า: stock คงเหลือ = {result['new_stock']}")

# Low stock alert
low_stock = inv.get_low_stock_alert()
print(f"\nสินค้า stock ต่ำ: {len(low_stock)} รายการ")
for item in low_stock:
    print(f"  {item['sku']}: {item['stock']} ชิ้น (reorder point: {item['reorder_point']})")

# มูลค่าสินค้า
value = inv.get_inventory_value()
print(f"\nมูลค่าสินค้าคงคลัง:")
print(f"  ราคาทุนรวม: {value['total_cost']:,.2f} บาท")
print(f"  ราคาขายรวม: {value['total_retail_value']:,.2f} บาท")
print(f"  กำไรที่คาด: {value['potential_profit']:,.2f} บาท")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Library Management System

**เฉลย:**
```python
import sqlite3
from contextlib import contextmanager
from datetime import datetime, timedelta

class LibraryDB:
    def __init__(self):
        self.db_path = 'library.db'
        self._init()
    
    @contextmanager
    def db(self):
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        conn.execute("PRAGMA foreign_keys = ON")
        try:
            yield conn
            conn.commit()
        except:
            conn.rollback()
            raise
        finally:
            conn.close()
    
    def _init(self):
        with self.db() as conn:
            conn.executescript('''
                CREATE TABLE IF NOT EXISTS books (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    isbn TEXT UNIQUE NOT NULL,
                    title TEXT NOT NULL,
                    author TEXT NOT NULL,
                    total_copies INTEGER DEFAULT 1,
                    available_copies INTEGER DEFAULT 1
                );
                
                CREATE TABLE IF NOT EXISTS members (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    card_number TEXT UNIQUE NOT NULL,
                    name TEXT NOT NULL,
                    email TEXT
                );
                
                CREATE TABLE IF NOT EXISTS borrows (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    book_id INTEGER NOT NULL,
                    member_id INTEGER NOT NULL,
                    borrow_date DATE NOT NULL,
                    due_date DATE NOT NULL,
                    return_date DATE,
                    FOREIGN KEY (book_id) REFERENCES books(id),
                    FOREIGN KEY (member_id) REFERENCES members(id)
                );
            ''')
    
    def borrow_book(self, isbn: str, card_number: str) -> dict:
        with self.db() as conn:
            book = conn.execute("SELECT * FROM books WHERE isbn = ?", (isbn,)).fetchone()
            member = conn.execute("SELECT * FROM members WHERE card_number = ?", (card_number,)).fetchone()
            
            if not book or not member:
                raise ValueError("Book or member not found")
            if book['available_copies'] <= 0:
                raise ValueError("No copies available")
            
            borrow_date = datetime.now().date()
            due_date = borrow_date + timedelta(days=14)
            
            conn.execute(
                "INSERT INTO borrows (book_id, member_id, borrow_date, due_date) VALUES (?,?,?,?)",
                (book['id'], member['id'], borrow_date, due_date)
            )
            conn.execute("UPDATE books SET available_copies = available_copies - 1 WHERE isbn = ?", (isbn,))
            
            return {'title': book['title'], 'due_date': str(due_date)}

library = LibraryDB()
with library.db() as conn:
    conn.execute("INSERT OR IGNORE INTO books (isbn, title, author, total_copies, available_copies) VALUES ('978-0-13-110362-7', 'The C Programming Language', 'K&R', 3, 3)")
    conn.execute("INSERT OR IGNORE INTO members (card_number, name) VALUES ('LIB001', 'Alice')")
result = library.borrow_book('978-0-13-110362-7', 'LIB001')
print(f"ยืมหนังสือ: {result['title']}, คืนวันที่ {result['due_date']}")
```

---

### ข้อที่ 2-10: แบบฝึกหัดเพิ่มเติม

### ข้อที่ 2: Employee Management

**เฉลย:**
```python
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey, DateTime
from sqlalchemy.orm import DeclarativeBase, relationship, Session
from sqlalchemy import select, func
from datetime import datetime

class EmpBase(DeclarativeBase):
    pass

class Department(EmpBase):
    __tablename__ = 'departments_emp'
    id = Column(Integer, primary_key=True)
    name = Column(String(100), unique=True)
    employees = relationship("Employee", back_populates="department")

class Employee(EmpBase):
    __tablename__ = 'employees'
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(200), unique=True)
    salary = Column(Float)
    hire_date = Column(DateTime, default=datetime.utcnow)
    dept_id = Column(Integer, ForeignKey('departments_emp.id'))
    manager_id = Column(Integer, ForeignKey('employees.id'), nullable=True)
    department = relationship("Department", back_populates="employees")
    subordinates = relationship("Employee", backref='manager', remote_side=[id])

emp_engine = create_engine('sqlite:///employees.db')
EmpBase.metadata.create_all(emp_engine)

with Session(emp_engine) as session:
    it_dept = Department(name="IT")
    hr_dept = Department(name="HR")
    session.add_all([it_dept, hr_dept])
    session.flush()
    
    ceo = Employee(name="CEO Alice", email="ceo@company.com", salary=200000, department=it_dept)
    session.add(ceo)
    session.flush()
    
    dev1 = Employee(name="Dev Bob", email="bob@company.com", salary=80000, department=it_dept, manager_id=ceo.id)
    dev2 = Employee(name="Dev Charlie", email="charlie@company.com", salary=75000, department=it_dept, manager_id=ceo.id)
    session.add_all([dev1, dev2])
    session.commit()

with Session(emp_engine) as session:
    result = session.execute(
        select(Department.name, func.count(Employee.id).label('count'), func.avg(Employee.salary).label('avg_sal'))
        .join(Employee)
        .group_by(Department.id)
    ).all()
    for row in result:
        print(f"{row.name}: {row.count} คน, เงินเดือนเฉลี่ย {row.avg_sal:,.0f}")
```

---

### ข้อที่ 3: Task/Todo Database

**เฉลย:**
```python
import sqlite3
from contextlib import contextmanager
from datetime import datetime

class TodoDB:
    STATUSES = ('pending', 'in_progress', 'completed', 'cancelled')
    PRIORITIES = ('low', 'medium', 'high', 'urgent')
    
    def __init__(self):
        self.db_path = 'todo.db'
        with self.get_conn() as conn:
            conn.execute('''CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL, description TEXT, status TEXT DEFAULT 'pending',
                priority TEXT DEFAULT 'medium', due_date DATE, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                completed_at TIMESTAMP
            )''')
    
    @contextmanager
    def get_conn(self):
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        try:
            yield conn
            conn.commit()
        except:
            conn.rollback()
            raise
        finally:
            conn.close()
    
    def add(self, title, desc='', priority='medium', due_date=None):
        with self.get_conn() as conn:
            cursor = conn.execute(
                "INSERT INTO tasks (title, description, priority, due_date) VALUES (?,?,?,?)",
                (title, desc, priority, due_date)
            )
            return cursor.lastrowid
    
    def complete(self, task_id):
        with self.get_conn() as conn:
            conn.execute(
                "UPDATE tasks SET status='completed', completed_at=? WHERE id=?",
                (datetime.now(), task_id)
            )
    
    def get_by_priority(self, priority):
        with self.get_conn() as conn:
            return [dict(r) for r in conn.execute(
                "SELECT * FROM tasks WHERE priority=? AND status!='completed'", (priority,)
            ).fetchall()]

todo = TodoDB()
t1 = todo.add("Study SQLAlchemy", priority="high", due_date="2024-02-01")
t2 = todo.add("Write tests", priority="medium")
t3 = todo.add("Deploy to production", priority="urgent")
todo.complete(t1)
urgent_tasks = todo.get_by_priority("urgent")
print(f"Urgent tasks: {[t['title'] for t in urgent_tasks]}")
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

1. **SQLite**: การใช้ sqlite3 module สำหรับ CRUD operations, parameterized queries, และ transactions
2. **SQLAlchemy Core**: SQL ใน Python แบบ type-safe
3. **SQLAlchemy ORM**: Declarative models, relationships, และ session management
4. **Relationships**: One-to-Many และ Many-to-Many patterns
5. **Alembic**: Database migration management
6. **Connection Pooling**: การจัดการ connections อย่างมีประสิทธิภาพ
7. **Design Patterns**: Repository pattern สำหรับ database access

ทักษะเหล่านี้จะช่วยให้คุณสร้างแอปพลิเคชันที่ทำงานกับฐานข้อมูลได้อย่างมืออาชีพ!
