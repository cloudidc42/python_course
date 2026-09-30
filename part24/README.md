# Part 24 - OOP: Encapsulation & Properties

## สารบัญ
1. [Encapsulation Concept](#encapsulation-concept)
2. [Public, Protected, Private Attributes](#public-protected-private-attributes)
3. [Name Mangling](#name-mangling)
4. [@property Decorator](#property-decorator)
5. [@setter และ @deleter](#setter-และ-deleter)
6. [Validation ใน Setters](#validation-ใน-setters)
7. [Read-only Properties](#read-only-properties)
8. [Computed Properties](#computed-properties)
9. [__slots__ สำหรับ Memory Optimization](#__slots__-สำหรับ-memory-optimization)
10. [Data Hiding](#data-hiding)
11. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Encapsulation Concept

**Encapsulation (การห่อหุ้ม)** คือหนึ่งในหลักการสำคัญของ OOP ที่:

1. **รวมข้อมูลและพฤติกรรม** ไว้ด้วยกันใน class เดียว
2. **ซ่อนรายละเอียดภายใน** (implementation details) ไม่ให้ code ภายนอกเข้าถึงโดยตรง
3. **ควบคุม access** ต่อข้อมูลผ่าน interface ที่กำหนดไว้

### ทำไม Encapsulation ถึงสำคัญ?

```python
# ===== ไม่มี Encapsulation =====
# ข้อมูลเปิดเผยหมด ใครก็แก้ได้

class BankAccountBAD:
    def __init__(self, balance):
        self.balance = balance  # เข้าถึงได้ตรงๆ

account = BankAccountBAD(1000)
account.balance = -999999  # ปัญหา! ยอดเงินติดลบ
print(account.balance)  # -999999


# ===== มี Encapsulation =====
# ข้อมูลถูกป้องกัน ต้องผ่าน methods

class BankAccountGOOD:
    def __init__(self, balance):
        self._balance = balance  # Protected
    
    @property
    def balance(self):
        return self._balance
    
    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("จำนวนเงินต้องมากกว่า 0")
        self._balance += amount
    
    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("จำนวนเงินต้องมากกว่า 0")
        if amount > self._balance:
            raise ValueError("เงินไม่พอ")
        self._balance -= amount

account = BankAccountGOOD(1000)
try:
    account.balance = -999999  # AttributeError - ไม่มี setter
except AttributeError as e:
    print(f"ป้องกันได้: {e}")

account.deposit(500)
print(account.balance)  # 1500
```

---

## Public, Protected, Private Attributes

Python ใช้ naming convention แทน access modifiers (ไม่มี `public`, `private`, `protected` keyword)

### ตัวอย่างที่ 1: สามระดับ Access

```python
class AccessDemo:
    """Demo class แสดง 3 ระดับ access"""
    
    def __init__(self):
        # 1. PUBLIC - ไม่มี underscore
        # - ทุกคนเข้าถึงได้
        # - เป็น public API ของ class
        self.public_var = "ทุกคนเข้าถึงได้"
        
        # 2. PROTECTED - underscore นำหน้า (_)
        # - Convention บอกว่า "ควรเข้าถึงจากภายใน class/subclass"
        # - Python ไม่บังคับ แต่เป็น signal ให้นักพัฒนาระวัง
        self._protected_var = "ควรเข้าถึงจาก class/subclass เท่านั้น"
        
        # 3. PRIVATE - double underscore นำหน้า (__)
        # - Python ทำ Name Mangling: __var -> _ClassName__var
        # - ป้องกัน accidental access/override ใน subclasses
        self.__private_var = "เข้าถึงยาก มี name mangling"
    
    def show_all(self):
        """เข้าถึง private ได้จากภายใน class"""
        print(f"Public: {self.public_var}")
        print(f"Protected: {self._protected_var}")
        print(f"Private: {self.__private_var}")


class SubClass(AccessDemo):
    def access_parent_vars(self):
        print(f"Public: {self.public_var}")         # OK
        print(f"Protected: {self._protected_var}")  # OK (แต่ควรระวัง)
        
        try:
            print(f"Private: {self.__private_var}")  # AttributeError!
        except AttributeError as e:
            print(f"ไม่สามารถเข้าถึง private: {e}")
        
        # ต้องใช้ mangled name
        print(f"Private (mangled): {self._AccessDemo__private_var}")  # OK

# ทดสอบ
obj = AccessDemo()

# เข้าถึง public ได้ตรง
print(obj.public_var)       # OK

# เข้าถึง protected ได้ (แต่ convention บอกว่าไม่ควร)
print(obj._protected_var)   # OK แต่ IDE จะ warn

# เข้าถึง private ต้องใช้ mangled name
try:
    print(obj.__private_var)  # AttributeError
except AttributeError:
    print("ไม่สามารถเข้าถึง __private_var โดยตรง")

# ต้องใช้ mangled name (ไม่แนะนำ)
print(obj._AccessDemo__private_var)  # ได้ผล แต่ไม่ควรทำ

obj.show_all()

# ทดสอบ subclass
sub = SubClass()
sub.access_parent_vars()
```

### ตัวอย่างที่ 2: Convention vs Enforcement

```python
class Configuration:
    """Class สาธิต convention การตั้งชื่อ"""
    
    DEFAULT_TIMEOUT = 30  # Public class constant
    _DEFAULT_RETRY = 3    # Protected class constant
    
    def __init__(self):
        # Public - users ของ class ใช้ได้
        self.host = "localhost"
        self.port = 8080
        
        # Protected - เฉพาะ class และ subclasses
        self._connection = None
        self._retry_count = 0
        
        # Private - internal implementation detail
        self.__secret_key = "abc123"  # ถ้าโดน override ใน subclass จะเกิดปัญหา
        self.__internal_state = {}
    
    def connect(self):
        """Public method - ส่วน API ที่ user ใช้"""
        self._connection = f"tcp://{self.host}:{self.port}"
        self.__log(f"เชื่อมต่อ {self._connection}")
        return self._connection
    
    def _validate_config(self):
        """Protected method - ใช้ภายใน class และ subclasses"""
        if not self.host:
            raise ValueError("host ต้องไม่ว่างเปล่า")
        if not (1 <= self.port <= 65535):
            raise ValueError("port ต้องอยู่ระหว่าง 1-65535")
    
    def __log(self, message):
        """Private method - internal logging ไม่ให้ override"""
        print(f"[LOG] {message}")
    
    @property
    def secret_key(self):
        """อนุญาตอ่านได้แต่ไม่แก้ได้โดยตรง"""
        return "***" + self.__secret_key[-3:]  # แสดงแค่บางส่วน

# ทดสอบ
config = Configuration()
config.host = "192.168.1.1"  # Public - แก้ได้
config.port = 9090           # Public - แก้ได้

conn = config.connect()
print(f"Connection: {conn}")
print(f"Secret key (masked): {config.secret_key}")

# dir() แสดง attributes ทั้งหมด รวม mangled names
print("\nAttributes ที่เห็นได้:")
for attr in dir(config):
    if not attr.startswith('__') and not attr.endswith('__'):
        print(f"  {attr}")
```

---

## Name Mangling

Python ทำ **name mangling** กับ attributes ที่มี double underscore นำหน้า โดยเปลี่ยน `__name` เป็น `_ClassName__name`

### ตัวอย่างที่ 3: Name Mangling ในทางปฏิบัติ

```python
class Parent:
    def __init__(self):
        self.__value = "Parent's private"
    
    def get_value(self):
        return self.__value  # เข้าถึงผ่าน _Parent__value


class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__value = "Child's private"  # สร้าง _Child__value ใหม่ ไม่ override Parent's
    
    def get_child_value(self):
        return self.__value  # เข้าถึง _Child__value


child = Child()

# แต่ละ class มี __value ของตัวเอง
print(child.get_value())       # Parent's private (ใช้ _Parent__value)
print(child.get_child_value()) # Child's private (ใช้ _Child__value)

# ดู __dict__ เพื่อเห็น mangled names
print("\n__dict__ ของ child:")
for key, value in child.__dict__.items():
    print(f"  {key}: {value}")
# _Parent__value: Parent's private
# _Child__value: Child's private
```

### ตัวอย่างที่ 4: ทำไมถึงต้องการ Name Mangling

```python
class Counter:
    """Class ที่ต้องการป้องกัน subclass ทำให้ __count ผิด"""
    
    def __init__(self):
        self.__count = 0  # private - ป้องกันด้วย name mangling
    
    def increment(self):
        self.__count += 1  # เข้าถึง _Counter__count
    
    def get_count(self):
        return self.__count


class BuggyCounter(Counter):
    """Subclass ที่พยายาม override __count"""
    
    def __init__(self):
        super().__init__()
        self.__count = 100  # สร้าง _BuggyCounter__count ใหม่ ไม่ได้ override!
    
    def reset(self):
        self.__count = 0  # แก้ _BuggyCounter__count ไม่ได้แก้ _Counter__count


bc = BuggyCounter()
bc.increment()
bc.increment()
bc.increment()

print(f"Count: {bc.get_count()}")  # 3 - ไม่ใช่ 100!
bc.reset()
print(f"หลัง reset: {bc.get_count()}")  # 3 - reset ไม่ได้แก้ Counter's __count

print("\n__dict__:")
for key, value in bc.__dict__.items():
    print(f"  {key}: {value}")
```

---

## @property Decorator

`@property` แปลง method ให้เหมือน attribute - เรียกโดยไม่ต้องใส่ `()`

### ตัวอย่างที่ 5: @property พื้นฐาน

```python
class Rectangle:
    def __init__(self, width, height):
        self._width = width
        self._height = height
    
    # ===== Properties =====
    
    @property
    def width(self):
        """Getter สำหรับ width"""
        return self._width
    
    @property
    def height(self):
        """Getter สำหรับ height"""
        return self._height
    
    @property
    def area(self):
        """Computed property - คำนวณจาก width และ height"""
        return self._width * self._height
    
    @property
    def perimeter(self):
        """Computed property"""
        return 2 * (self._width + self._height)
    
    @property
    def diagonal(self):
        """Computed property"""
        return (self._width**2 + self._height**2)**0.5
    
    @property
    def is_square(self):
        """Boolean property"""
        return self._width == self._height
    
    def __str__(self):
        return f"Rectangle({self._width} × {self._height})"

# ทดสอบ
r = Rectangle(4, 3)

# เรียก properties เหมือน attributes
print(f"กว้าง: {r.width}")         # 4
print(f"สูง: {r.height}")          # 3
print(f"พื้นที่: {r.area}")        # 12
print(f"เส้นรอบรูป: {r.perimeter}")  # 14
print(f"เส้นทแยงมุม: {r.diagonal:.4f}")  # 5.0000
print(f"เป็นสี่เหลี่ยมจัตุรัส: {r.is_square}")  # False

# Properties ไม่สามารถ set ได้ (ไม่มี setter)
try:
    r.area = 100  # AttributeError
except AttributeError as e:
    print(f"\nError: ไม่สามารถ set computed property: {e}")
```

### ตัวอย่างที่ 6: property() ฟังก์ชัน (แบบเก่า)

```python
class OldStyle:
    """วิธีเก่าในการใช้ property() - เพื่อเข้าใจ"""
    
    def __init__(self, value):
        self._value = value
    
    def get_value(self):
        return self._value
    
    def set_value(self, v):
        self._value = v
    
    def del_value(self):
        del self._value
    
    # สร้าง property จาก getter, setter, deleter
    value = property(get_value, set_value, del_value, "docstring สำหรับ value")


class NewStyle:
    """วิธีใหม่ด้วย decorator - แนะนำ"""
    
    def __init__(self, value):
        self._value = value
    
    @property
    def value(self):
        """docstring สำหรับ value"""
        return self._value
    
    @value.setter
    def value(self, v):
        self._value = v
    
    @value.deleter
    def value(self):
        del self._value


# ทั้งสองแบบทำงานเหมือนกัน
old = OldStyle(42)
new = NewStyle(42)

print(old.value)  # 42
print(new.value)  # 42

old.value = 100
new.value = 100

print(old.value)  # 100
print(new.value)  # 100

# Help สำหรับ property
print(help(OldStyle.value))
```

---

## @setter และ @deleter

### ตัวอย่างที่ 7: Property พร้อม Setter และ Deleter

```python
class Person:
    def __init__(self, name, age):
        # เรียก setters ใน __init__ เพื่อ validate ตั้งแต่ต้น
        self.name = name
        self.age = age
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError(f"ชื่อต้องเป็น str ไม่ใช่ {type(value).__name__}")
        value = value.strip()
        if len(value) < 2:
            raise ValueError("ชื่อต้องมีอย่างน้อย 2 ตัวอักษร")
        if len(value) > 100:
            raise ValueError("ชื่อยาวเกินไป (สูงสุด 100 ตัวอักษร)")
        self._name = value
    
    @name.deleter
    def name(self):
        print(f"ลบชื่อ {self._name}")
        del self._name
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            raise TypeError("อายุต้องเป็น int")
        if value < 0 or value > 150:
            raise ValueError(f"อายุ {value} ไม่สมเหตุสมผล")
        self._age = value
    
    @property
    def birth_year(self):
        """Read-only computed property"""
        from datetime import date
        return date.today().year - self._age
    
    def __str__(self):
        return f"Person(name={self._name!r}, age={self._age})"


# ทดสอบ
p = Person("สมชาย", 25)
print(p)

# แก้ไขผ่าน setter
p.name = "  สมหญิง ใจดี  "  # strip whitespace อัตโนมัติ
print(p.name)  # สมหญิง ใจดี

p.age = 30
print(p.birth_year)  # คำนวณอัตโนมัติ

# ทดสอบ validation
try:
    p.name = "A"  # ชื่อสั้นเกิน
except ValueError as e:
    print(f"Error: {e}")

try:
    p.age = -5
except ValueError as e:
    print(f"Error: {e}")

try:
    p.age = "สามสิบ"
except TypeError as e:
    print(f"Error: {e}")

# ทดสอบ deleter
del p.name
try:
    print(p.name)
except AttributeError as e:
    print(f"Name deleted: {e}")
```

---

## Validation ใน Setters

### ตัวอย่างที่ 8: Validation Patterns

```python
import re
from datetime import date

class User:
    """User class พร้อม comprehensive validation"""
    
    # Class-level constants
    MIN_AGE = 13
    MAX_AGE = 120
    USERNAME_PATTERN = re.compile(r'^[a-zA-Z][a-zA-Z0-9_]{2,19}$')
    EMAIL_PATTERN = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    PASSWORD_MIN_LENGTH = 8
    
    def __init__(self, username, email, age, password):
        # ลำดับสำคัญ - บางอย่างต้องตั้งก่อน
        self.username = username
        self.email = email
        self.age = age
        self.password = password
        self._login_attempts = 0
        self._is_locked = False
    
    @property
    def username(self):
        return self._username
    
    @username.setter
    def username(self, value):
        if not isinstance(value, str):
            raise TypeError("username ต้องเป็น string")
        if not self.USERNAME_PATTERN.match(value):
            raise ValueError(
                "username ต้องขึ้นต้นด้วยตัวอักษร มีความยาว 3-20 ตัว "
                "และใช้ได้เฉพาะ a-z, 0-9, _"
            )
        self._username = value
    
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, value):
        if not isinstance(value, str):
            raise TypeError("email ต้องเป็น string")
        if not self.EMAIL_PATTERN.match(value.lower()):
            raise ValueError(f"'{value}' ไม่ใช่ email ที่ถูกต้อง")
        self._email = value.lower()
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            raise TypeError("อายุต้องเป็น int")
        if value < self.MIN_AGE:
            raise ValueError(f"ต้องมีอายุอย่างน้อย {self.MIN_AGE} ปี")
        if value > self.MAX_AGE:
            raise ValueError(f"อายุ {value} ไม่สมเหตุสมผล")
        self._age = value
    
    @property
    def password(self):
        """ไม่ return password จริง"""
        return "***"
    
    @password.setter
    def password(self, value):
        if not isinstance(value, str):
            raise TypeError("password ต้องเป็น string")
        if len(value) < self.PASSWORD_MIN_LENGTH:
            raise ValueError(f"password ต้องมีอย่างน้อย {self.PASSWORD_MIN_LENGTH} ตัวอักษร")
        if not any(c.isupper() for c in value):
            raise ValueError("password ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว")
        if not any(c.isdigit() for c in value):
            raise ValueError("password ต้องมีตัวเลขอย่างน้อย 1 ตัว")
        # เก็บ hashed password (จำลอง)
        import hashlib
        self._password_hash = hashlib.sha256(value.encode()).hexdigest()
    
    def verify_password(self, password):
        import hashlib
        return self._password_hash == hashlib.sha256(password.encode()).hexdigest()
    
    @property
    def is_locked(self):
        return self._is_locked
    
    def login(self, password):
        if self._is_locked:
            raise RuntimeError("บัญชีถูกล็อค")
        
        if self.verify_password(password):
            self._login_attempts = 0
            return True
        else:
            self._login_attempts += 1
            if self._login_attempts >= 5:
                self._is_locked = True
                raise RuntimeError("บัญชีถูกล็อค (ลองผิดเกิน 5 ครั้ง)")
            return False
    
    def __str__(self):
        status = "ล็อค" if self._is_locked else "ปกติ"
        return f"User(username={self._username!r}, email={self._email}, status={status})"
    
    def __repr__(self):
        return f"User(username={self._username!r}, email={self._email!r})"


# ทดสอบ
print("=== สร้าง User ===")
user = User("john_doe", "john@example.com", 25, "SecurePass1")
print(user)

print("\n=== ทดสอบ Validation ===")

# username ผิดรูปแบบ
test_cases = [
    ("username", "1startWithNumber", ValueError),
    ("username", "ab", ValueError),  # สั้นเกิน
    ("email", "not-an-email", ValueError),
    ("age", 10, ValueError),  # อายุน้อยเกิน
    ("age", "thirty", TypeError),
    ("password", "short", ValueError),  # สั้นเกิน
    ("password", "nouppercase1", ValueError),  # ไม่มีตัวพิมพ์ใหญ่
]

for attr, value, expected_error in test_cases:
    try:
        setattr(user, attr, value)
        print(f"ERROR: ควร raise {expected_error.__name__}")
    except expected_error as e:
        print(f"  OK: {attr}={value!r} -> {type(e).__name__}: {e}")

print("\n=== ทดสอบ Login ===")
print(f"Login สำเร็จ: {user.login('SecurePass1')}")  # True
print(f"Login ล้มเหลว: {user.login('wrongpass')}")    # False
```

---

## Read-only Properties

### ตัวอย่างที่ 9: Read-only Properties หลายแบบ

```python
from datetime import date

class ImmutablePoint:
    """Point ที่ไม่เปลี่ยนแปลงหลังสร้างแล้ว"""
    
    def __init__(self, x, y):
        self._x = x
        self._y = y
    
    @property
    def x(self):
        return self._x
    
    # ไม่มี @x.setter -> read-only
    
    @property
    def y(self):
        return self._y
    
    @property
    def distance_from_origin(self):
        return (self._x**2 + self._y**2)**0.5
    
    def __repr__(self):
        return f"ImmutablePoint({self._x}, {self._y})"


class Invoice:
    """ใบแจ้งหนี้ที่ lock หลังจาก finalize"""
    
    def __init__(self, items):
        self._items = list(items)
        self._finalized = False
        self._invoice_number = None
        self._created_date = date.today()
    
    @property
    def items(self):
        return tuple(self._items)  # return tuple = immutable copy
    
    @property
    def total(self):
        return sum(item['price'] * item['qty'] for item in self._items)
    
    @property
    def invoice_number(self):
        return self._invoice_number
    
    @property
    def is_finalized(self):
        return self._finalized
    
    @property
    def created_date(self):
        return self._created_date
    
    def add_item(self, name, price, qty):
        if self._finalized:
            raise RuntimeError("ไม่สามารถแก้ไขใบแจ้งหนี้ที่ finalize แล้ว")
        self._items.append({"name": name, "price": price, "qty": qty})
    
    def finalize(self):
        if not self._items:
            raise ValueError("ใบแจ้งหนี้ต้องมีสินค้าอย่างน้อย 1 รายการ")
        self._finalized = True
        self._invoice_number = f"INV-{date.today().strftime('%Y%m%d')}-{id(self) % 10000:04d}"
        print(f"Finalize ใบแจ้งหนี้ {self._invoice_number} แล้ว")
    
    def __str__(self):
        status = "FINAL" if self._finalized else "DRAFT"
        num = self._invoice_number or "DRAFT"
        return f"Invoice({num}, {len(self._items)} รายการ, {self.total:,.2f} บาท, {status})"


# ทดสอบ ImmutablePoint
p = ImmutablePoint(3, 4)
print(f"Point: {p}")
print(f"x: {p.x}, y: {p.y}")
print(f"ระยะทาง: {p.distance_from_origin}")

try:
    p.x = 10  # AttributeError
except AttributeError as e:
    print(f"Read-only: {e}")

# ทดสอบ Invoice
invoice = Invoice([])
invoice.add_item("Python Book", 599, 2)
invoice.add_item("Coffee Mug", 250, 3)

print(invoice)
print(f"รายการ: {invoice.items}")
print(f"ยอดรวม: {invoice.total:,.2f}")

invoice.finalize()
print(invoice)

# พยายามแก้ไขหลัง finalize
try:
    invoice.add_item("Keyboard", 1500, 1)
except RuntimeError as e:
    print(f"Error: {e}")
```

---

## Computed Properties

Properties ที่คำนวณจาก attributes อื่น โดยไม่ต้องเก็บค่าเอง

### ตัวอย่างที่ 10: Computed Properties หลากหลาย

```python
import math
from datetime import date, timedelta

class Mortgage:
    """คำนวณสินเชื่อที่อยู่อาศัย"""
    
    def __init__(self, principal, annual_rate, years):
        """
        Parameters:
            principal: เงินกู้ (บาท)
            annual_rate: อัตราดอกเบี้ยต่อปี (เปอร์เซ็นต์)
            years: จำนวนปี
        """
        self._principal = principal
        self._annual_rate = annual_rate
        self._years = years
        self._start_date = date.today()
    
    @property
    def principal(self):
        return self._principal
    
    @property
    def annual_rate(self):
        return self._annual_rate
    
    @property
    def monthly_rate(self):
        """อัตราดอกเบี้ยรายเดือน"""
        return self._annual_rate / 100 / 12
    
    @property
    def total_payments(self):
        """จำนวนงวดทั้งหมด"""
        return self._years * 12
    
    @property
    def monthly_payment(self):
        """ค่างวดรายเดือน - PMT formula"""
        r = self.monthly_rate
        n = self.total_payments
        if r == 0:  # ดอกเบี้ย 0%
            return self._principal / n
        return self._principal * r * (1 + r)**n / ((1 + r)**n - 1)
    
    @property
    def total_paid(self):
        """ยอดรวมที่จ่ายทั้งหมด"""
        return self.monthly_payment * self.total_payments
    
    @property
    def total_interest(self):
        """ดอกเบี้ยรวม"""
        return self.total_paid - self._principal
    
    @property
    def end_date(self):
        """วันที่ผ่อนหมด"""
        return self._start_date + timedelta(days=self._years * 365.25)
    
    def amortization_schedule(self, months=12):
        """ตารางการผ่อน"""
        balance = self._principal
        r = self.monthly_rate
        monthly = self.monthly_payment
        
        print(f"\nตารางการผ่อน ({months} เดือนแรก)")
        print(f"{'เดือน':>5} {'ยอดค้าง':>15} {'เงินต้น':>12} {'ดอกเบี้ย':>12} {'เงินต้นคงเหลือ':>15}")
        print("-" * 62)
        
        for month in range(1, months + 1):
            interest = balance * r
            principal_paid = monthly - interest
            balance -= principal_paid
            
            print(f"{month:>5} {monthly:>15,.2f} {principal_paid:>12,.2f} "
                  f"{interest:>12,.2f} {max(0, balance):>15,.2f}")
    
    def __str__(self):
        return (f"Mortgage(เงินกู้={self._principal:,.0f} บาท, "
                f"ดอกเบี้ย={self._annual_rate}%/ปี, "
                f"{self._years} ปี)")


# ทดสอบ
loan = Mortgage(2_000_000, 6.5, 20)

print(loan)
print(f"ค่างวดรายเดือน: {loan.monthly_payment:,.2f} บาท")
print(f"จำนวนงวด: {loan.total_payments} งวด")
print(f"ยอดรวมที่จ่าย: {loan.total_paid:,.2f} บาท")
print(f"ดอกเบี้ยรวม: {loan.total_interest:,.2f} บาท")
print(f"ผ่อนหมดวันที่: {loan.end_date}")

loan.amortization_schedule(months=6)
```

### ตัวอย่างที่ 11: Cached Property

```python
class CachedProperty:
    """Descriptor สำหรับ cached property"""
    
    def __init__(self, func):
        self.func = func
        self.attrname = None
        self.__doc__ = func.__doc__
    
    def __set_name__(self, owner, name):
        self.attrname = f"_cache_{name}"
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if not hasattr(obj, self.attrname):
            setattr(obj, self.attrname, self.func(obj))
        return getattr(obj, self.attrname)


class ExpensiveCalculation:
    def __init__(self, data):
        self.data = data
    
    @CachedProperty
    def statistics(self):
        """คำนวณสถิติ - ทำครั้งเดียวแล้ว cache"""
        print("  (กำลังคำนวณ...)")
        import time
        time.sleep(0.1)  # จำลองการคำนวณที่นาน
        n = len(self.data)
        mean = sum(self.data) / n
        variance = sum((x - mean)**2 for x in self.data) / n
        return {
            'n': n,
            'sum': sum(self.data),
            'mean': mean,
            'std': variance**0.5,
            'min': min(self.data),
            'max': max(self.data),
        }


# ทดสอบ cached property
import time

data = list(range(1, 101))  # 1 ถึง 100
ec = ExpensiveCalculation(data)

print("เรียกครั้งแรก:")
start = time.time()
stats = ec.statistics
print(f"  ใช้เวลา: {time.time() - start:.3f}s")
print(f"  Mean: {stats['mean']}")

print("\nเรียกครั้งที่สอง (จาก cache):")
start = time.time()
stats = ec.statistics
print(f"  ใช้เวลา: {time.time() - start:.5f}s")
print(f"  Mean: {stats['mean']}")


# Python 3.8+ มี functools.cached_property
from functools import cached_property

class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    @cached_property
    def area(self):
        print("  คำนวณ area...")
        import math
        return math.pi * self.radius ** 2
    
    @cached_property
    def circumference(self):
        print("  คำนวณ circumference...")
        import math
        return 2 * math.pi * self.radius


c = Circle(5)
print("\nCircle cached_property:")
print(f"Area (1st): {c.area:.4f}")
print(f"Area (2nd): {c.area:.4f}")  # จาก cache ไม่คำนวณใหม่
```

---

## __slots__ สำหรับ Memory Optimization

`__slots__` จำกัด attributes ที่ instance สามารถมีได้ ทำให้ประหยัด memory

### ตัวอย่างที่ 12: __slots__ พื้นฐาน

```python
class WithoutSlots:
    """ไม่มี __slots__ - ใช้ __dict__"""
    
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z


class WithSlots:
    """มี __slots__ - ประหยัด memory"""
    
    __slots__ = ('x', 'y', 'z')  # ระบุ attributes ที่อนุญาต
    
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z


# เปรียบเทียบ memory
import sys

obj_no_slots = WithoutSlots(1, 2, 3)
obj_slots = WithSlots(1, 2, 3)

print(f"ขนาดโดยไม่ใช้ __slots__: {sys.getsizeof(obj_no_slots)} bytes")
print(f"ขนาดโดยใช้ __slots__: {sys.getsizeof(obj_slots)} bytes")

# Without slots มี __dict__
print(f"\nWithout slots has __dict__: {hasattr(obj_no_slots, '__dict__')}")
print(f"  __dict__: {obj_no_slots.__dict__}")

# With slots ไม่มี __dict__
print(f"With slots has __dict__: {hasattr(obj_slots, '__dict__')}")

# With slots สามารถเพิ่ม attribute ได้ตาม __slots__
obj_slots.x = 10    # OK
try:
    obj_slots.w = 5  # AttributeError - w ไม่อยู่ใน __slots__
except AttributeError as e:
    print(f"\nError: {e}")

# Performance comparison
print("\nเปรียบเทียบ Performance:")
import timeit

setup_no_slots = "obj = WithoutSlots(1, 2, 3)"
setup_slots = "obj = WithSlots(1, 2, 3)"
code = "obj.x; obj.y; obj.z"

# เพิ่มคำ import ให้ timeit
globals_dict = {'WithoutSlots': WithoutSlots, 'WithSlots': WithSlots}

t_no_slots = timeit.timeit(
    stmt=setup_no_slots + "; " + code,
    globals=globals_dict,
    number=1000000
)
t_slots = timeit.timeit(
    stmt=setup_slots + "; " + code,
    globals=globals_dict,
    number=1000000
)

print(f"ไม่ใช้ __slots__: {t_no_slots:.3f}s")
print(f"ใช้ __slots__: {t_slots:.3f}s")
print(f"ประหยัดได้: {(1 - t_slots/t_no_slots)*100:.1f}%")
```

### ตัวอย่างที่ 13: __slots__ กับ Inheritance

```python
class Base:
    __slots__ = ('x', 'y')
    
    def __init__(self, x, y):
        self.x = x
        self.y = y


class Child(Base):
    __slots__ = ('z',)  # เพิ่ม z เข้ามา (x, y มาจาก Base)
    
    def __init__(self, x, y, z):
        super().__init__(x, y)
        self.z = z


class GrandChild(Child):
    # ไม่มี __slots__ -> มี __dict__ (กลับมาใช้ dict)
    pass


b = Base(1, 2)
c = Child(1, 2, 3)
gc = GrandChild(1, 2, 3)

print(f"Base has __dict__: {hasattr(b, '__dict__')}")   # False
print(f"Child has __dict__: {hasattr(c, '__dict__')}")  # False
print(f"GrandChild has __dict__: {hasattr(gc, '__dict__')}")  # True!

# GrandChild สามารถเพิ่ม attributes ได้ (มี __dict__)
gc.w = 4
print(f"gc.w = {gc.w}")  # 4


class Point3D:
    """Optimized 3D point ที่ใช้ใน data processing"""
    
    __slots__ = ('x', 'y', 'z')
    
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z
    
    def distance_to(self, other):
        return ((self.x - other.x)**2 + 
                (self.y - other.y)**2 + 
                (self.z - other.z)**2)**0.5
    
    def __repr__(self):
        return f"Point3D({self.x}, {self.y}, {self.z})"


# ทดสอบกับข้อมูลจำนวนมาก
points = [Point3D(i, i*2, i*3) for i in range(100000)]
print(f"\nสร้าง {len(points):,} points")
print(f"Point แรก: {points[0]}")
print(f"Point สุดท้าย: {points[-1]}")
print(f"ระยะทาง: {points[0].distance_to(points[1]):.4f}")
```

---

## Data Hiding

### ตัวอย่างที่ 14: Techniques สำหรับ Data Hiding

```python
class SecureStorage:
    """เก็บข้อมูลสำคัญอย่างปลอดภัย"""
    
    def __init__(self):
        self.__data = {}           # Private: เก็บข้อมูลจริง
        self.__access_log = []     # Private: บันทึกการเข้าถึง
        self.__master_key = None   # Private: key สำหรับ encrypt
    
    def set_master_key(self, key):
        """ตั้ง master key"""
        if not isinstance(key, str) or len(key) < 16:
            raise ValueError("Master key ต้องมีอย่างน้อย 16 ตัวอักษร")
        self.__master_key = key
        print("ตั้ง master key แล้ว")
    
    def store(self, key, value):
        """เก็บข้อมูล"""
        if self.__master_key is None:
            raise RuntimeError("ต้องตั้ง master key ก่อน")
        
        # Encrypt value (จำลอง)
        encrypted = self.__encrypt(str(value))
        self.__data[key] = encrypted
        self.__log(f"STORE: {key}")
    
    def retrieve(self, key):
        """ดึงข้อมูล"""
        if key not in self.__data:
            raise KeyError(f"ไม่พบ key: {key}")
        
        self.__log(f"RETRIEVE: {key}")
        encrypted = self.__data[key]
        return self.__decrypt(encrypted)
    
    def delete(self, key):
        """ลบข้อมูล"""
        if key not in self.__data:
            raise KeyError(f"ไม่พบ key: {key}")
        del self.__data[key]
        self.__log(f"DELETE: {key}")
    
    @property
    def keys(self):
        """แสดงรายการ keys ที่มี (แต่ไม่แสดง values)"""
        return list(self.__data.keys())
    
    @property
    def access_log(self):
        """แสดง access log (copy เท่านั้น ป้องกันการแก้ไข)"""
        return self.__access_log.copy()
    
    def __encrypt(self, text):
        """Private: จำลอง encryption ง่ายๆ"""
        if self.__master_key is None:
            return text
        shift = sum(ord(c) for c in self.__master_key) % 26
        result = []
        for char in text:
            if char.isalpha():
                base = ord('A') if char.isupper() else ord('a')
                result.append(chr((ord(char) - base + shift) % 26 + base))
            else:
                result.append(char)
        return ''.join(result)
    
    def __decrypt(self, text):
        """Private: จำลอง decryption"""
        if self.__master_key is None:
            return text
        shift = sum(ord(c) for c in self.__master_key) % 26
        result = []
        for char in text:
            if char.isalpha():
                base = ord('A') if char.isupper() else ord('a')
                result.append(chr((ord(char) - base - shift) % 26 + base))
            else:
                result.append(char)
        return ''.join(result)
    
    def __log(self, action):
        """Private: บันทึก log"""
        from datetime import datetime
        self.__access_log.append({
            "time": datetime.now().isoformat(),
            "action": action
        })


# ทดสอบ
storage = SecureStorage()
storage.set_master_key("MySecretKey12345")

storage.store("username", "admin")
storage.store("api_key", "sk-12345678")
storage.store("database_url", "postgresql://localhost/mydb")

print(f"Keys: {storage.keys}")

username = storage.retrieve("username")
print(f"Username: {username}")

storage.delete("api_key")

print(f"\nAccess Log:")
for entry in storage.access_log:
    print(f"  [{entry['time'][:19]}] {entry['action']}")

# พยายามเข้าถึง private data โดยตรง
print(f"\nพยายามเข้าถึง private:")
try:
    print(storage.__data)  # AttributeError
except AttributeError:
    print("  ไม่สามารถเข้าถึง __data โดยตรง")

# ต้องผ่าน method
print(f"  ต้องผ่าน retrieve(): {storage.retrieve('username')}")
```

---

## ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Temperature Class ครบถ้วน

```python
class Temperature:
    """ระบบจัดการอุณหภูมิที่ครบถ้วน"""
    
    ABSOLUTE_ZERO_CELSIUS = -273.15
    
    def __init__(self, celsius=None, fahrenheit=None, kelvin=None):
        """สร้าง Temperature จาก celsius, fahrenheit, หรือ kelvin"""
        
        # ต้องระบุอย่างน้อย 1 หน่วย
        if sum(v is not None for v in [celsius, fahrenheit, kelvin]) == 0:
            raise ValueError("ต้องระบุอุณหภูมิอย่างน้อย 1 หน่วย")
        if sum(v is not None for v in [celsius, fahrenheit, kelvin]) > 1:
            raise ValueError("ระบุได้แค่ 1 หน่วย")
        
        if celsius is not None:
            self.celsius = celsius
        elif fahrenheit is not None:
            self.fahrenheit = fahrenheit
        elif kelvin is not None:
            self.kelvin = kelvin
    
    # ===== Celsius =====
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("อุณหภูมิต้องเป็นตัวเลข")
        if value < self.ABSOLUTE_ZERO_CELSIUS:
            raise ValueError(
                f"อุณหภูมิต้องไม่ต่ำกว่า absolute zero ({self.ABSOLUTE_ZERO_CELSIUS}°C)"
            )
        self._celsius = float(value)
    
    # ===== Fahrenheit =====
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5/9
    
    # ===== Kelvin =====
    
    @property
    def kelvin(self):
        return self._celsius - self.ABSOLUTE_ZERO_CELSIUS
    
    @kelvin.setter
    def kelvin(self, value):
        if value < 0:
            raise ValueError("Kelvin ต้องไม่ติดลบ")
        self.celsius = value + self.ABSOLUTE_ZERO_CELSIUS
    
    # ===== Computed Properties =====
    
    @property
    def is_freezing(self):
        return self._celsius <= 0
    
    @property
    def is_boiling(self):
        return self._celsius >= 100
    
    @property
    def description(self):
        if self._celsius < -20:
            return "หนาวจัด"
        elif self._celsius < 0:
            return "หนาวมาก"
        elif self._celsius < 10:
            return "หนาว"
        elif self._celsius < 20:
            return "เย็น"
        elif self._celsius < 30:
            return "อบอุ่น"
        elif self._celsius < 35:
            return "อุ่น"
        elif self._celsius < 40:
            return "ร้อน"
        else:
            return "ร้อนจัด"
    
    # ===== Comparison =====
    
    def __eq__(self, other):
        if isinstance(other, Temperature):
            return abs(self._celsius - other._celsius) < 0.001
        return NotImplemented
    
    def __lt__(self, other):
        if isinstance(other, Temperature):
            return self._celsius < other._celsius
        return NotImplemented
    
    def __le__(self, other):
        return self < other or self == other
    
    def __gt__(self, other):
        if isinstance(other, Temperature):
            return self._celsius > other._celsius
        return NotImplemented
    
    def __ge__(self, other):
        return self > other or self == other
    
    def __hash__(self):
        return hash(round(self._celsius, 3))
    
    # ===== Arithmetic =====
    
    def __add__(self, other):
        if isinstance(other, (int, float)):
            return Temperature(celsius=self._celsius + other)
        if isinstance(other, Temperature):
            return Temperature(celsius=self._celsius + other._celsius)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, (int, float)):
            return Temperature(celsius=self._celsius - other)
        if isinstance(other, Temperature):
            return abs(self._celsius - other._celsius)  # ผลต่าง
        return NotImplemented
    
    # ===== String =====
    
    def __str__(self):
        return (f"{self._celsius:.1f}°C / {self.fahrenheit:.1f}°F / "
                f"{self.kelvin:.1f}K ({self.description})")
    
    def __repr__(self):
        return f"Temperature(celsius={self._celsius})"
    
    def __format__(self, spec):
        if spec == 'C':
            return f"{self._celsius:.1f}°C"
        elif spec == 'F':
            return f"{self.fahrenheit:.1f}°F"
        elif spec == 'K':
            return f"{self.kelvin:.1f}K"
        elif spec == 'short':
            return f"{self._celsius:.0f}°C"
        return str(self)
    
    # ===== Class Methods =====
    
    @classmethod
    def from_celsius(cls, value):
        return cls(celsius=value)
    
    @classmethod
    def from_fahrenheit(cls, value):
        return cls(fahrenheit=value)
    
    @classmethod
    def from_kelvin(cls, value):
        return cls(kelvin=value)
    
    @classmethod
    def body_temperature(cls):
        return cls(celsius=37)
    
    @classmethod
    def room_temperature(cls):
        return cls(celsius=25)
    
    @classmethod
    def absolute_zero(cls):
        return cls(celsius=cls.ABSOLUTE_ZERO_CELSIUS)


# =================== ทดสอบ Temperature ===================

print("=== สร้าง Temperature ===")
t1 = Temperature(celsius=100)
t2 = Temperature.from_fahrenheit(32)
t3 = Temperature.from_kelvin(300)
t4 = Temperature.body_temperature()

print(f"t1 (100°C): {t1}")
print(f"t2 (32°F): {t2}")
print(f"t3 (300K): {t3}")
print(f"t4 (body temp): {t4}")

print(f"\n=== Format ===")
print(f"t1 in Celsius: {t1:C}")
print(f"t1 in Fahrenheit: {t1:F}")
print(f"t1 in Kelvin: {t1:K}")

print(f"\n=== Properties ===")
print(f"t2 is freezing: {t2.is_freezing}")
print(f"t1 is boiling: {t1.is_boiling}")
print(f"Room temp description: {Temperature.room_temperature().description}")

print(f"\n=== Comparison ===")
print(f"t1 > t4: {t1 > t4}")
print(f"t2 < t4: {t2 < t4}")

temps = [Temperature(celsius=c) for c in [37, 100, -10, 25, 0]]
print(f"Sorted: {sorted(temps)}")

print(f"\n=== Arithmetic ===")
hot = Temperature(celsius=35)
result = hot + 5
print(f"35°C + 5 = {result}")

diff = abs(t1 - t4)  # ผลต่างเป็น float
print(f"|t1 - t4| = {diff:.1f}°C")
```

### โปรแกรมที่ 2: Circle Class

```python
import math

class Circle:
    """Circle class ที่ครบถ้วนพร้อม Encapsulation"""
    
    def __init__(self, radius, center=(0, 0)):
        """
        Parameters:
            radius: รัศมี (ต้องมากกว่า 0)
            center: จุดศูนย์กลาง (x, y)
        """
        self.radius = radius  # ใช้ setter
        self.center = center  # ใช้ setter
        self._color = "white"
        self._filled = True
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("รัศมีต้องเป็นตัวเลข")
        if value <= 0:
            raise ValueError("รัศมีต้องมากกว่า 0")
        self._radius = float(value)
        # Clear cached values ถ้ามี
        for cached in ['_cache_area', '_cache_circumference']:
            if hasattr(self, cached):
                delattr(self, cached)
    
    @property
    def center(self):
        return self._center
    
    @center.setter
    def center(self, value):
        if not isinstance(value, (tuple, list)) or len(value) != 2:
            raise ValueError("center ต้องเป็น tuple (x, y)")
        if not all(isinstance(v, (int, float)) for v in value):
            raise TypeError("coordinates ต้องเป็นตัวเลข")
        self._center = tuple(value)
    
    @property
    def x(self):
        return self._center[0]
    
    @property
    def y(self):
        return self._center[1]
    
    @property
    def diameter(self):
        return self._radius * 2
    
    @diameter.setter
    def diameter(self, value):
        self.radius = value / 2
    
    @property
    def area(self):
        return math.pi * self._radius ** 2
    
    @property
    def circumference(self):
        return 2 * math.pi * self._radius
    
    @property
    def color(self):
        return self._color
    
    @color.setter
    def color(self, value):
        valid_colors = ['red', 'green', 'blue', 'yellow', 'white', 'black',
                       'แดง', 'เขียว', 'น้ำเงิน', 'เหลือง', 'ขาว', 'ดำ']
        if isinstance(value, str) and (value.lower() in valid_colors or 
                                        value.startswith('#')):
            self._color = value
        else:
            raise ValueError(f"สี '{value}' ไม่ถูกต้อง")
    
    @property
    def filled(self):
        return self._filled
    
    @filled.setter
    def filled(self, value):
        if not isinstance(value, bool):
            raise TypeError("filled ต้องเป็น bool")
        self._filled = value
    
    # ===== Methods =====
    
    def contains_point(self, px, py):
        """ตรวจสอบว่า point (px, py) อยู่ภายใน circle หรือไม่"""
        dx = px - self._center[0]
        dy = py - self._center[1]
        return dx**2 + dy**2 <= self._radius**2
    
    def intersects(self, other):
        """ตรวจสอบว่า circles สองวงตัดกันหรือไม่"""
        dx = self._center[0] - other._center[0]
        dy = self._center[1] - other._center[1]
        distance = math.sqrt(dx**2 + dy**2)
        return distance <= self._radius + other._radius
    
    def distance_to(self, other):
        """ระยะทางระหว่างขอบวง (ลบหมายความว่าซ้อนทับกัน)"""
        dx = self._center[0] - other._center[0]
        dy = self._center[1] - other._center[1]
        center_dist = math.sqrt(dx**2 + dy**2)
        return center_dist - self._radius - other._radius
    
    def scale(self, factor):
        """ขยาย/ย่อ circle"""
        self.radius = self._radius * factor
        return self
    
    def move(self, dx, dy):
        """เลื่อนตำแหน่ง"""
        self.center = (self._center[0] + dx, self._center[1] + dy)
        return self
    
    def __eq__(self, other):
        if isinstance(other, Circle):
            return (abs(self._radius - other._radius) < 1e-9 and
                    self._center == other._center)
        return NotImplemented
    
    def __lt__(self, other):
        if isinstance(other, Circle):
            return self._radius < other._radius
        return NotImplemented
    
    def __le__(self, other):
        return self < other or self == other
    
    def __gt__(self, other):
        if isinstance(other, Circle):
            return self._radius > other._radius
        return NotImplemented
    
    def __ge__(self, other):
        return self > other or self == other
    
    def __hash__(self):
        return hash((round(self._radius, 6), self._center))
    
    def __contains__(self, point):
        """ใช้ 'point in circle' syntax"""
        return self.contains_point(*point)
    
    def __str__(self):
        return (f"Circle(r={self._radius:.2f}, "
                f"center={self._center}, "
                f"area={self.area:.2f})")
    
    def __repr__(self):
        return f"Circle(radius={self._radius}, center={self._center})"
    
    @classmethod
    def unit_circle(cls):
        return cls(1.0, (0, 0))
    
    @classmethod
    def from_diameter(cls, diameter, center=(0, 0)):
        return cls(diameter / 2, center)
    
    @staticmethod
    def concentric(circles):
        """ตรวจสอบว่า circles ทั้งหมด concentric (ศูนย์กลางเดียวกัน)"""
        if len(circles) < 2:
            return True
        center = circles[0].center
        return all(c.center == center for c in circles[1:])


# =================== ทดสอบ Circle ===================

c1 = Circle(5, (0, 0))
c2 = Circle(3, (7, 0))
c3 = Circle(2, (0, 0))

print("=== Circle Properties ===")
print(f"c1: {c1}")
print(f"radius: {c1.radius}")
print(f"diameter: {c1.diameter}")
print(f"area: {c1.area:.4f}")
print(f"circumference: {c1.circumference:.4f}")

print("\n=== ตรวจสอบ Containment ===")
print(f"(3, 4) ใน c1: {(3, 4) in c1}")   # sqrt(9+16)=5 = radius -> True
print(f"(4, 4) ใน c1: {(4, 4) in c1}")   # sqrt(16+16)>5 -> False
print(f"c1.contains_point(0, 0): {c1.contains_point(0, 0)}")  # True

print("\n=== ตรวจสอบ Intersection ===")
print(f"c1 intersects c2: {c1.intersects(c2)}")  # True (5+3 >= 7)
print(f"c1 intersects c3: {c1.intersects(c3)}")  # True (same center)
print(f"c2 intersects c3: {c2.intersects(c3)}")  # 7 <= 3+2? -> No

print("\n=== Comparison ===")
circles = [Circle(3), Circle(1), Circle(5), Circle(2)]
print(f"Sorted by radius: {sorted(circles)}")

print("\n=== Scale and Move ===")
c_test = Circle(2, (0, 0))
print(f"ก่อน: {c_test}")
c_test.scale(2).move(3, 4)
print(f"หลัง scale(2).move(3, 4): {c_test}")
```

### โปรแกรมที่ 3: Person with Validation

```python
import re
from datetime import date
from enum import Enum

class Gender(Enum):
    MALE = "ชาย"
    FEMALE = "หญิง"
    OTHER = "อื่นๆ"

class Address:
    """ที่อยู่"""
    
    def __init__(self, street, city, province, postal_code, country="Thailand"):
        self.street = street
        self.city = city
        self.province = province
        self.postal_code = postal_code
        self.country = country
    
    @property
    def postal_code(self):
        return self._postal_code
    
    @postal_code.setter
    def postal_code(self, value):
        value = str(value).strip()
        if not re.match(r'^\d{5}$', value):
            raise ValueError(f"รหัสไปรษณีย์ '{value}' ต้องเป็นตัวเลข 5 หลัก")
        self._postal_code = value
    
    def __str__(self):
        return f"{self.street}, {self.city}, {self.province} {self._postal_code}, {self.country}"


class Person:
    """Person class พร้อม comprehensive encapsulation"""
    
    __slots__ = (
        '_first_name', '_last_name', '_birth_date', 
        '_gender', '_email', '_phone', '_address',
        '_id_number', '_is_verified'
    )
    
    def __init__(self, first_name, last_name, birth_date, gender):
        self.first_name = first_name
        self.last_name = last_name
        self.birth_date = birth_date
        self.gender = gender
        self._email = None
        self._phone = None
        self._address = None
        self._id_number = None
        self._is_verified = False
    
    # ===== Name =====
    
    @property
    def first_name(self):
        return self._first_name
    
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str) or len(value.strip()) < 2:
            raise ValueError("ชื่อต้องมีอย่างน้อย 2 ตัวอักษร")
        self._first_name = value.strip().title()
    
    @property
    def last_name(self):
        return self._last_name
    
    @last_name.setter
    def last_name(self, value):
        if not isinstance(value, str) or len(value.strip()) < 2:
            raise ValueError("นามสกุลต้องมีอย่างน้อย 2 ตัวอักษร")
        self._last_name = value.strip().title()
    
    @property
    def full_name(self):
        return f"{self._first_name} {self._last_name}"
    
    # ===== Birth Date =====
    
    @property
    def birth_date(self):
        return self._birth_date
    
    @birth_date.setter
    def birth_date(self, value):
        if isinstance(value, str):
            try:
                value = date.fromisoformat(value)
            except ValueError:
                raise ValueError("วันเกิดต้องอยู่ในรูปแบบ YYYY-MM-DD")
        if not isinstance(value, date):
            raise TypeError("วันเกิดต้องเป็น date object หรือ string YYYY-MM-DD")
        if value > date.today():
            raise ValueError("วันเกิดต้องไม่เป็นอนาคต")
        if (date.today() - value).days > 150 * 365:
            raise ValueError("อายุไม่สมเหตุสมผล (เกิน 150 ปี)")
        self._birth_date = value
    
    @property
    def age(self):
        today = date.today()
        born = self._birth_date
        return today.year - born.year - ((today.month, today.day) < (born.month, born.day))
    
    # ===== Gender =====
    
    @property
    def gender(self):
        return self._gender
    
    @gender.setter
    def gender(self, value):
        if isinstance(value, str):
            try:
                value = Gender[value.upper()]
            except KeyError:
                try:
                    value = Gender(value)
                except ValueError:
                    raise ValueError(f"Gender ต้องเป็น {[g.name for g in Gender]}")
        if not isinstance(value, Gender):
            raise TypeError("gender ต้องเป็น Gender enum")
        self._gender = value
    
    # ===== Contact =====
    
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, value):
        if value is None:
            self._email = None
            return
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', value):
            raise ValueError(f"'{value}' ไม่ใช่ email ที่ถูกต้อง")
        self._email = value.lower()
    
    @property
    def phone(self):
        return self._phone
    
    @phone.setter
    def phone(self, value):
        if value is None:
            self._phone = None
            return
        cleaned = re.sub(r'[\s\-\(\)]', '', str(value))
        if not re.match(r'^(\+66|0)\d{8,9}$', cleaned):
            raise ValueError(f"'{value}' ไม่ใช่เบอร์โทรศัพท์ไทยที่ถูกต้อง")
        self._phone = cleaned
    
    @property
    def address(self):
        return self._address
    
    @address.setter
    def address(self, value):
        if value is not None and not isinstance(value, Address):
            raise TypeError("address ต้องเป็น Address object")
        self._address = value
    
    # ===== Verification =====
    
    @property
    def is_verified(self):
        return self._is_verified
    
    def verify(self, id_number):
        """ยืนยันตัวตน"""
        if not re.match(r'^\d{13}$', str(id_number)):
            raise ValueError("เลขบัตรประชาชนต้องมี 13 หลัก")
        self._id_number = str(id_number)
        self._is_verified = True
        print(f"{self.full_name} ยืนยันตัวตนแล้ว")
    
    # ===== String =====
    
    def __str__(self):
        parts = [f"{self.full_name} ({self.age} ปี)"]
        if self._email:
            parts.append(f"Email: {self._email}")
        if self._phone:
            parts.append(f"โทร: {self._phone}")
        if self._is_verified:
            parts.append("[ยืนยันแล้ว]")
        return " | ".join(parts)
    
    def __repr__(self):
        return (f"Person(first_name={self._first_name!r}, "
                f"last_name={self._last_name!r}, "
                f"birth_date={self._birth_date})")


# =================== ทดสอบ Person ===================

print("=== สร้าง Person ===")
p = Person("สมชาย", "ใจดี", "1990-05-15", Gender.MALE)

# เพิ่มข้อมูลติดต่อ
p.email = "somchai@example.com"
p.phone = "081-234-5678"

addr = Address("123 ถนนสุขุมวิท", "กรุงเทพมหานคร", "กรุงเทพมหานคร", "10110")
p.address = addr

print(p)
print(f"วันเกิด: {p.birth_date}")
print(f"อายุ: {p.age} ปี")
print(f"เพศ: {p.gender.value}")
print(f"ที่อยู่: {p.address}")

# ยืนยันตัวตน
p.verify("1234567890123")
print(f"ยืนยันแล้ว: {p.is_verified}")

print("\n=== ทดสอบ Validation ===")

# ชื่อผิด
try:
    p.first_name = "A"
except ValueError as e:
    print(f"  ชื่อ: {e}")

# Email ผิด
try:
    p.email = "not-an-email"
except ValueError as e:
    print(f"  Email: {e}")

# เบอร์โทรผิด
try:
    p.phone = "12345"
except ValueError as e:
    print(f"  Phone: {e}")

# วันเกิดในอนาคต
try:
    p.birth_date = "2030-01-01"
except ValueError as e:
    print(f"  วันเกิด: {e}")
```

---

## แบบฝึกหัด

### ข้อที่ 1: BankAccount with Encapsulation

```python
# สร้าง BankAccount class ที่มี:
# - account_number (read-only, สร้างอัตโนมัติ)
# - balance (read-only, แก้ได้ผ่าน deposit/withdraw)
# - interest_rate (property พร้อม validation 0-20%)
# - account_type (SAVINGS, CHECKING, FIXED)
# - limit_per_transaction (สูงสุดที่ถอนได้ครั้งเดียว)

# เฉลย
from enum import Enum

class AccountType(Enum):
    SAVINGS = "ออมทรัพย์"
    CHECKING = "กระแสรายวัน"
    FIXED = "ประจำ"

class BankAccount:
    _counter = 0
    
    def __init__(self, owner, account_type=AccountType.SAVINGS, 
                 initial_deposit=0, interest_rate=1.5):
        BankAccount._counter += 1
        self._account_number = f"TH{BankAccount._counter:010d}"
        self._owner = owner
        self._balance = 0.0
        self._account_type = account_type
        self.interest_rate = interest_rate  # ใช้ setter
        self._transaction_history = []
        
        if initial_deposit > 0:
            self.deposit(initial_deposit, "เงินฝากเปิดบัญชี")
    
    @property
    def account_number(self):
        return self._account_number
    
    @property
    def owner(self):
        return self._owner
    
    @property
    def balance(self):
        return self._balance
    
    @property
    def account_type(self):
        return self._account_type
    
    @property
    def interest_rate(self):
        return self._interest_rate
    
    @interest_rate.setter
    def interest_rate(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("อัตราดอกเบี้ยต้องเป็นตัวเลข")
        if not 0 <= value <= 20:
            raise ValueError("อัตราดอกเบี้ยต้องอยู่ระหว่าง 0-20%")
        self._interest_rate = float(value)
    
    @property
    def limit_per_transaction(self):
        """วงเงินถอนสูงสุดต่อครั้ง ขึ้นอยู่กับ account_type"""
        limits = {
            AccountType.SAVINGS: 50000,
            AccountType.CHECKING: 500000,
            AccountType.FIXED: 0,  # ถอนไม่ได้ก่อนกำหนด
        }
        return limits[self._account_type]
    
    def deposit(self, amount, description="ฝากเงิน"):
        if amount <= 0:
            raise ValueError("จำนวนเงินต้องมากกว่า 0")
        self._balance += amount
        self._record(f"ฝาก: {description}", amount)
    
    def withdraw(self, amount, description="ถอนเงิน"):
        if amount <= 0:
            raise ValueError("จำนวนเงินต้องมากกว่า 0")
        if amount > self._balance:
            raise ValueError("ยอดเงินไม่เพียงพอ")
        if self.limit_per_transaction > 0 and amount > self.limit_per_transaction:
            raise ValueError(f"เกินวงเงินต่อครั้ง ({self.limit_per_transaction:,} บาท)")
        if self._account_type == AccountType.FIXED:
            raise RuntimeError("บัญชีประจำไม่สามารถถอนได้ก่อนกำหนด")
        self._balance -= amount
        self._record(f"ถอน: {description}", -amount)
    
    def apply_interest(self):
        interest = self._balance * self._interest_rate / 100
        self._balance += interest
        self._record("ดอกเบี้ย", interest)
        return interest
    
    def _record(self, description, amount):
        self._transaction_history.append({
            "description": description,
            "amount": amount,
            "balance": self._balance
        })
    
    def statement(self):
        print(f"\n{'='*55}")
        print(f"บัญชี: {self._account_number} ({self._account_type.value})")
        print(f"เจ้าของ: {self._owner}")
        print(f"{'='*55}")
        for t in self._transaction_history:
            sign = "+" if t['amount'] >= 0 else ""
            print(f"{t['description']:30} {sign}{t['amount']:>10,.2f} | {t['balance']:>10,.2f}")
        print(f"{'='*55}")
        print(f"ยอดคงเหลือ: {self._balance:,.2f} บาท")
    
    def __str__(self):
        return (f"BankAccount({self._account_number}, "
                f"{self._owner}, "
                f"{self._account_type.value}, "
                f"{self._balance:,.2f} บาท)")


# ทดสอบ
acc = BankAccount("สมชาย ใจดี", AccountType.SAVINGS, 10000)
acc.deposit(5000, "รับเงินเดือน")
acc.withdraw(2000, "ค่าอาหาร")
acc.apply_interest()
acc.statement()

# ทดสอบ property validation
try:
    acc.interest_rate = 25  # เกิน 20%
except ValueError as e:
    print(f"\nError: {e}")
```

### ข้อที่ 2 - 10: โจทย์ฝึกหัดเพิ่มเติม

```python
# ข้อที่ 2: Product Inventory
# - Product(name, price, stock, min_stock)
# - price (ต้องมากกว่า 0, ลดได้ไม่เกิน 50%)
# - stock (ต้องไม่ติดลบ, แจ้งเตือนเมื่อต่ำกว่า min_stock)
# - sell(qty), restock(qty)

# ข้อที่ 3: StudentGrade with __slots__
# - __slots__ สำหรับ performance
# - ใช้ @property สำหรับทุก attribute
# - GPA คำนวณอัตโนมัติ

# ข้อที่ 4: Smart Home Device
# - Device(name, device_type)
# - is_on (bool property)
# - brightness (0-100 สำหรับ light)
# - temperature (15-30 สำหรับ AC)
# - schedule (dict ของ on/off times)

# ข้อที่ 5: File System Node
# - FileNode(name, path)
# - size (read-only)
# - permissions (rwx format)
# - owner, group
# - modified_date (read-only set ใน methods)

# ข้อที่ 6: Recipe with Ingredients
# - Ingredient(name, amount, unit)
# - amount ต้องมากกว่า 0
# - Recipe(name, servings)
# - servings ต้องมากกว่า 0
# - scale_to(new_servings) คำนวณ ingredient ใหม่

# ข้อที่ 7: Network Configuration
# - NetworkConfig(ip, subnet, gateway, dns)
# - ip ต้อง validate รูปแบบ IP
# - subnet_mask ต้อง validate
# - gateway ต้องอยู่ใน subnet

# ข้อที่ 8: Calendar Event
# - Event(title, start_datetime, end_datetime)
# - start ต้องไม่เป็นอดีต (เมื่อสร้าง)
# - end ต้องหลัง start
# - duration (computed, read-only)
# - is_all_day (computed)

# ข้อที่ 9: Secure Configuration
# - SecureConfig ที่ encrypt sensitive values
# - get/set ด้วย property
# - audit log ของการเข้าถึง

# ข้อที่ 10: Dimension class
# - Dimension(value, unit)
# - แปลงหน่วยอัตโนมัติ (cm, m, inch, foot)
# - arithmetic operations ที่จัดการหน่วยให้
```

---

## สรุป Part 24

| แนวคิด | Syntax | วัตถุประสงค์ |
|--------|--------|------------|
| Public | `self.name` | เข้าถึงได้ทุกที่ |
| Protected | `self._name` | Convention: เฉพาะ class/subclass |
| Private | `self.__name` | Name mangling ป้องกัน override |
| Property (getter) | `@property` | เข้าถึงเหมือน attribute |
| Property (setter) | `@prop.setter` | Validation ก่อน set |
| Property (deleter) | `@prop.deleter` | Logic ก่อน delete |
| Read-only | Getter ไม่มี Setter | ป้องกันการแก้ไข |
| Computed | Property ที่คำนวณ | ไม่เก็บค่า คำนวณทุกครั้ง |
| Cached Property | `@cached_property` | คำนวณครั้งเดียว cache ไว้ |
| __slots__ | `__slots__ = (...)` | ประหยัด memory |

**ต่อไป**: Part 25 - OOP Abstract Classes & Interfaces
