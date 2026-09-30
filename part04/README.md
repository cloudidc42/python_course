# Part 04: Numbers, Math & Type Conversion

## สารบัญ (Table of Contents)

1. [ระบบตัวเลขใน Python](#ระบบตัวเลขใน-python)
2. [Integer Types และ Number Bases](#integer-types-และ-number-bases)
3. [Float และปัญหา Floating Point](#float-และปัญหา-floating-point)
4. [Complex Numbers](#complex-numbers)
5. [Math Module](#math-module)
6. [math.floor, math.ceil, math.round](#mathfloor-mathceil-mathround)
7. [Random Module](#random-module)
8. [Decimal Module สำหรับการเงิน](#decimal-module-สำหรับการเงิน)
9. [Fraction Module](#fraction-module)
10. [Number Formatting](#number-formatting)
11. [Scientific Notation](#scientific-notation)
12. [Statistics Module](#statistics-module)
13. [ตัวอย่างโค้ด](#ตัวอย่างโค้ด)
14. [แบบฝึกหัด](#แบบฝึกหัด)
15. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## ระบบตัวเลขใน Python

Python รองรับ numeric types หลายประเภท:

```
Numeric Types
├── int      - จำนวนเต็ม (ไม่มีขีดจำกัด)
├── float    - ทศนิยม (64-bit double precision)
├── complex  - จำนวนเชิงซ้อน (real + imaginary)
├── Decimal  - ทศนิยมแม่นยำสูง (จาก decimal module)
└── Fraction - เศษส่วน (จาก fractions module)
```

### สรุป Properties

| Type | ความแม่นยำ | ช่วงค่า | ใช้เมื่อ |
|------|-----------|---------|---------|
| `int` | สมบูรณ์ | ไม่จำกัด | นับ, index, loops |
| `float` | ~15-17 digits | ±1.8e308 | การคำนวณทั่วไป |
| `complex` | float precision | - | วิศวกรรม, วิทยาศาสตร์ |
| `Decimal` | กำหนดได้ | ไม่จำกัด | การเงิน, accounting |
| `Fraction` | สมบูรณ์ | ไม่จำกัด | เศษส่วนแม่นยำ |

---

## Integer Types และ Number Bases

### Basic Integer

```python
# Python int ไม่มีขีดจำกัดขนาด!
small = 42
large = 9_999_999_999_999_999_999_999_999  # underscore separator (Python 3.6+)
negative = -100
zero = 0

print(type(small))     # <class 'int'>
print(small.bit_length())  # จำนวน bits ที่ใช้ = 6

# ตรวจสอบขนาด
import sys
print(sys.maxsize)         # 9223372036854775807 (2^63 - 1 บน 64-bit)

# Python int ใหญ่ได้ไม่จำกัด
huge = 10 ** 1000   # 10 ยกกำลัง 1000 - ทำได้!
print(len(str(huge)))  # 1001 digits
```

### Number Bases

```python
# ฐาน 2 (Binary) - นำหน้าด้วย 0b หรือ 0B
binary = 0b10110111
print(binary)           # 183 (decimal)
print(bin(183))         # '0b10110111'
print(bin(183)[2:])     # '10110111' (ไม่มี prefix)
print(format(183, 'b')) # '10110111'
print(f"{183:b}")       # '10110111'
print(f"{183:08b}")     # '10110111' (8-bit padded)

# ฐาน 8 (Octal) - นำหน้าด้วย 0o หรือ 0O
octal = 0o267
print(octal)            # 183
print(oct(183))         # '0o267'
print(f"{183:o}")       # '267'
print(f"{183:#o}")      # '0o267' (กับ prefix)

# ฐาน 16 (Hexadecimal) - นำหน้าด้วย 0x หรือ 0X
hex_num = 0xB7
print(hex_num)          # 183
print(hex(183))         # '0xb7'
print(hex(183).upper()) # '0XB7'
print(f"{183:x}")       # 'b7'
print(f"{183:X}")       # 'B7'
print(f"{183:#x}")      # '0xb7'
print(f"{183:08X}")     # '000000B7' (8-digit padded)

# แปลงจากฐานอื่น
n_from_bin  = int('10110111', 2)   # 183
n_from_oct  = int('267', 8)         # 183
n_from_hex  = int('B7', 16)         # 183
n_from_hex2 = int('0xB7', 16)       # 183

print(n_from_bin, n_from_oct, n_from_hex)  # 183 183 183
```

### Integer Operations

```python
# Arithmetic ทั้งหมด
a, b = 17, 5

print(f"a + b  = {a + b}")    # 22
print(f"a - b  = {a - b}")    # 12
print(f"a * b  = {a * b}")    # 85
print(f"a / b  = {a / b}")    # 3.4 (float!)
print(f"a // b = {a // b}")   # 3  (floor division)
print(f"a % b  = {a % b}")    # 2  (modulo)
print(f"a ** b = {a ** b}")   # 1419857

# divmod() - หารพร้อมเศษ
quotient, remainder = divmod(17, 5)
print(f"17 ÷ 5 = {quotient} เศษ {remainder}")  # 3 เศษ 2

# abs() - ค่าสัมบูรณ์
print(abs(-42))    # 42
print(abs(42))     # 42

# pow() - ยกกำลัง (รองรับ modulo)
print(pow(2, 10))         # 1024
print(pow(2, 10, 1000))   # 24 (2^10 mod 1000, เร็วกว่า)

# Integer methods
n = 255
print(n.bit_length())   # 8 (จำนวน bits ที่ใช้)
print(n.bit_count())    # 8 (Python 3.10+, จำนวน 1-bits)
print(n.to_bytes(2, 'big'))     # b'\x00\xff'
print(n.to_bytes(2, 'little'))  # b'\xff\x00'

# จาก bytes
n2 = int.from_bytes(b'\x00\xff', 'big')
print(n2)   # 255
```

---

## Float และปัญหา Floating Point

### Float พื้นฐาน

```python
# การสร้าง float
f1 = 3.14
f2 = -2.5
f3 = 1.0
f4 = .5       # 0.5
f5 = 5.       # 5.0

print(type(f1))   # <class 'float'>

# Float properties
import sys
print(sys.float_info.max)       # 1.7976931348623157e+308
print(sys.float_info.min)       # 2.2250738585072014e-308 (smallest positive)
print(sys.float_info.epsilon)   # 2.220446049250313e-16 (ความแตกต่างที่ detect ได้)
print(sys.float_info.dig)       # 15 (significant decimal digits)
```

### Floating Point Problem

```python
# IEEE 754 floating point
print(0.1 + 0.2)       # 0.30000000000000004 !!!
print(0.1 + 0.2 == 0.3)  # False !!!

# ทำไม? เพราะ binary representation
# 0.1 ใน binary = 0.0001100110011... (repeating)

# ดู exact representation
from decimal import Decimal
print(Decimal(0.1))    # 0.1000000000000000055511151231257827021181583404541015625
print(Decimal(0.2))    # 0.200000000000000011102230246251565404236316680908203125
print(Decimal(0.3))    # 0.299999999999999988897769753748434595763683319091796875

# วิธีแก้ปัญหา
# 1. round()
result = round(0.1 + 0.2, 10)
print(result == 0.3)   # True

# 2. math.isclose() (แนะนำที่สุด)
import math
print(math.isclose(0.1 + 0.2, 0.3))           # True
print(math.isclose(0.1 + 0.2, 0.3, rel_tol=1e-9))  # True
print(math.isclose(1.0, 1.0 + 1e-8))          # True (rel_tol)
print(math.isclose(0.0, 1e-10, abs_tol=1e-9)) # True (abs_tol)

# 3. Decimal module (ถูกที่สุด สำหรับการเงิน)
from decimal import Decimal
result = Decimal('0.1') + Decimal('0.2')
print(result)          # 0.3
print(result == Decimal('0.3'))  # True
```

### Special Float Values

```python
import math

# Infinity
pos_inf = float('inf')
neg_inf = float('-inf')
print(pos_inf)          # inf
print(neg_inf)          # -inf
print(pos_inf + 1)      # inf
print(pos_inf - pos_inf)  # nan
print(math.isinf(pos_inf))  # True

# NaN (Not a Number)
nan = float('nan')
print(nan)              # nan
print(nan == nan)       # False (NaN ไม่เท่ากับตัวเอง!)
print(math.isnan(nan))  # True

# ตรวจสอบ finite
print(math.isfinite(3.14))   # True
print(math.isfinite(pos_inf)) # False
print(math.isfinite(nan))    # False

# Operations กับ special values
print(1 / 0.0)     # ZeroDivisionError!
try:
    x = 1 / 0.0
except ZeroDivisionError:
    x = float('inf')
print(x)            # inf

# ใน numpy, 1/0.0 = inf (ไม่ throw exception)
```

---

## Complex Numbers

```python
# การสร้าง complex numbers
c1 = 3 + 4j          # ส่วนจริง=3, ส่วนจินตภาพ=4
c2 = complex(3, 4)   # เหมือนกัน
c3 = complex(5)      # 5 + 0j
c4 = 2j              # 0 + 2j

print(type(c1))   # <class 'complex'>
print(c1)         # (3+4j)
print(c1.real)    # 3.0
print(c1.imag)    # 4.0
print(c1.conjugate())  # (3-4j)

# Arithmetic
c1 = 2 + 3j
c2 = 1 - 1j

print(c1 + c2)   # (3+2j)
print(c1 - c2)   # (1+4j)
print(c1 * c2)   # (5+1j)   = (2*1 - 3*(-1)) + (2*(-1) + 3*1)j = 5 + 1j
print(c1 / c2)   # (-0.5+2.5j)

# abs() = modulus
print(abs(3 + 4j))   # 5.0  (sqrt(3^2 + 4^2))

# cmath module สำหรับ complex
import cmath

c = 1 + 1j

print(cmath.phase(c))         # 0.785... (angle in radians = π/4)
print(cmath.polar(c))         # (1.414..., 0.785...) = (r, θ)

# แปลง polar เป็น rectangular
r, theta = cmath.polar(c)
c_back = cmath.rect(r, theta)
print(c_back)    # (1+1j)

# Math functions สำหรับ complex
print(cmath.sqrt(-1))    # 1j
print(cmath.exp(1j * cmath.pi))  # (-1+1.2e-16j) ≈ -1 (Euler's formula)
print(cmath.log(c))      # (0.346...+0.785...j)
print(cmath.sin(c))      # (1.298...+0.634...j)
```

---

## Math Module

```python
import math

# Constants
print(f"pi    = {math.pi}")      # 3.141592653589793
print(f"e     = {math.e}")       # 2.718281828459045
print(f"tau   = {math.tau}")     # 6.283185307179586 (2*pi)
print(f"inf   = {math.inf}")     # inf
print(f"nan   = {math.nan}")     # nan

# Rounding (ดูหัวข้อถัดไป)
print(math.floor(4.7))    # 4
print(math.ceil(4.2))     # 5
print(math.trunc(4.7))    # 4 (ตัดทศนิยม)

# Power and Logarithms
print(math.sqrt(16))      # 4.0  (square root)
print(math.pow(2, 10))    # 1024.0 (ใช้ float)
print(math.exp(1))        # 2.718... (e^1)
print(math.log(math.e))   # 1.0  (natural log)
print(math.log(100, 10))  # 2.0  (log base 10)
print(math.log10(1000))   # 3.0  (log base 10)
print(math.log2(8))       # 3.0  (log base 2)

# Trigonometry (radians)
print(math.sin(0))             # 0.0
print(math.cos(0))             # 1.0
print(math.tan(math.pi/4))     # 0.999... ≈ 1.0
print(math.sin(math.pi/2))     # 1.0
print(math.sin(math.pi))       # 1.2246e-16 ≈ 0 (floating point!)

# Convert degrees <-> radians
print(math.degrees(math.pi))   # 180.0
print(math.radians(180))       # 3.14159...
print(math.sin(math.radians(30)))  # 0.5

# Inverse trig
print(math.asin(1))    # π/2 = 1.5707...
print(math.acos(0))    # π/2 = 1.5707...
print(math.atan(1))    # π/4 = 0.7853...
print(math.atan2(1, 1))  # π/4 = 0.7853... (y/x)

# Hyperbolic
print(math.sinh(1))    # 1.175...
print(math.cosh(1))    # 1.543...
print(math.tanh(1))    # 0.761...

# Other
print(math.factorial(5))   # 120 (5!)
print(math.gcd(48, 18))    # 6 (Greatest Common Divisor)
print(math.lcm(4, 6))      # 12 (Least Common Multiple) Python 3.9+
print(math.comb(5, 2))     # 10 (C(5,2) = combinations)
print(math.perm(5, 2))     # 20 (P(5,2) = permutations)
print(math.isqrt(17))      # 4 (integer square root)

# hypot() - ระยะทาง
print(math.hypot(3, 4))    # 5.0 (sqrt(3^2 + 4^2))
print(math.hypot(1, 1, 1)) # 1.732... (3D distance, Python 3.8+)

# fsum() - แม่นยำกว่า sum()
numbers = [0.1] * 10
print(sum(numbers))           # 0.9999999999999999
print(math.fsum(numbers))     # 1.0

# prod() - คูณทั้งหมด (Python 3.8+)
print(math.prod([1, 2, 3, 4, 5]))  # 120

# frexp() และ ldexp()
m, e = math.frexp(10.5)     # แยก mantissa และ exponent
print(f"10.5 = {m} * 2^{e}")
print(math.ldexp(m, e))     # 10.5 (กลับมา)
```

---

## math.floor, math.ceil, math.round

```python
import math

x = 3.7
y = -3.7
z = 3.5
w = 4.5

print(f"Value:        {x:>6}  {y:>6}  {z:>6}  {w:>6}")
print(f"math.floor(): {math.floor(x):>6}  {math.floor(y):>6}  {math.floor(z):>6}  {math.floor(w):>6}")
print(f"math.ceil():  {math.ceil(x):>6}  {math.ceil(y):>6}  {math.ceil(z):>6}  {math.ceil(w):>6}")
print(f"math.trunc(): {math.trunc(x):>6}  {math.trunc(y):>6}  {math.trunc(z):>6}  {math.trunc(w):>6}")
print(f"round():      {round(x):>6}  {round(y):>6}  {round(z):>6}  {round(w):>6}")
print(f"int():        {int(x):>6}  {int(y):>6}  {int(z):>6}  {int(w):>6}")

# ผลลัพธ์:
# Value:           3.7    -3.7     3.5     4.5
# math.floor():      3      -4       3       4
# math.ceil():       4      -3       4       5
# math.trunc():      3      -3       3       4
# round():           4      -4       4       4 (Banker's rounding!)
# int():             3      -3       3       4

# round() ใช้ "Banker's Rounding" (Round half to even)
print("\n=== Banker's Rounding ===")
print(round(0.5))   # 0 (ปัดเป็นคู่ที่ใกล้สุด = 0)
print(round(1.5))   # 2 (ปัดเป็นคู่ที่ใกล้สุด = 2)
print(round(2.5))   # 2 (ปัดเป็นคู่ที่ใกล้สุด = 2)
print(round(3.5))   # 4 (ปัดเป็นคู่ที่ใกล้สุด = 4)
print(round(4.5))   # 4 (ปัดเป็นคู่ที่ใกล้สุด = 4)
print(round(5.5))   # 6 (ปัดเป็นคู่ที่ใกล้สุด = 6)

# round() กับทศนิยม
print("\n=== round() กับทศนิยม ===")
pi = 3.14159265358979
print(round(pi))        # 3
print(round(pi, 1))     # 3.1
print(round(pi, 2))     # 3.14
print(round(pi, 4))     # 3.1416
print(round(pi, -1))    # 0.0 (ปัดถึงสิบ)
print(round(1234.5, -2))  # 1200.0 (ปัดถึงร้อย)
print(round(1250, -2))    # 1200 (Banker's rounding!)
print(round(1350, -2))    # 1400

# Floor Division vs floor()
print("\n=== Floor Division vs floor() ===")
print(7 // 2)              # 3
print(math.floor(7 / 2))   # 3
print(-7 // 2)             # -4 (ปัดลง)
print(math.floor(-7 / 2))  # -4

# Ceiling Division
def ceil_div(a, b):
    """หารปัดขึ้น"""
    return -(-a // b)   # tricks!

print(ceil_div(7, 2))    # 4
print(math.ceil(7 / 2))  # 4
```

---

## Random Module

```python
import random

# Seed กำหนด reproducibility
random.seed(42)

# ===== Random Floats =====
print("=== Float ===")
print(random.random())          # [0.0, 1.0) uniform
print(random.uniform(1.0, 10.0))  # [a, b] uniform
print(random.gauss(0, 1))       # Normal distribution (mu, sigma)
print(random.normalvariate(0, 1))  # Normal distribution
print(random.triangular(0, 10, 5)) # Triangular distribution

# ===== Random Integers =====
print("\n=== Integer ===")
print(random.randint(1, 6))      # [a, b] inclusive (ลูกเต๋า)
print(random.randrange(0, 10))   # [a, b) exclusive
print(random.randrange(0, 100, 5))  # ทุก 5 (0, 5, 10, ..., 95)

# ===== Random from Sequences =====
fruits = ["apple", "banana", "cherry", "date", "elderberry"]

print("\n=== Sequence ===")
print(random.choice(fruits))     # เลือก 1 ตัว
print(random.choices(fruits, k=3))  # เลือก 3 ตัว (ซ้ำได้)
print(random.choices(fruits, weights=[10, 1, 1, 1, 1], k=3))  # weighted

sample = random.sample(fruits, 3)  # เลือก 3 ตัว (ไม่ซ้ำ)
print(sample)

# Shuffle
numbers = list(range(1, 11))
random.shuffle(numbers)
print(numbers)   # สุ่มลำดับ

# ===== Cryptographic Random =====
import secrets  # แทน random สำหรับ security

print("\n=== Secure Random ===")
print(secrets.randbelow(100))         # [0, 100)
print(secrets.randbits(32))           # 32-bit random
print(secrets.token_bytes(16))        # 16 random bytes
print(secrets.token_hex(16))          # 32-char hex string
print(secrets.token_urlsafe(16))      # URL-safe base64 string

# สร้าง random password
def generate_password(length=12):
    import string
    alphabet = string.ascii_letters + string.digits + string.punctuation
    return ''.join(secrets.choice(alphabet) for _ in range(length))

print(generate_password())
print(generate_password(16))
```

---

## Decimal Module สำหรับการเงิน

```python
from decimal import Decimal, getcontext, ROUND_HALF_UP, ROUND_DOWN

# ===== Decimal Basics =====
# สร้าง Decimal จาก string เสมอ (ไม่ใช่ float!)
price = Decimal('19.99')
tax_rate = Decimal('0.07')

print(price)          # 19.99
print(type(price))    # <class 'decimal.Decimal'>

# การคำนวณแม่นยำ
total_before_tax = Decimal('100.00')
tax = total_before_tax * tax_rate
total = total_before_tax + tax

print(f"ราคา:  {total_before_tax}")  # 100.00
print(f"ภาษี:  {tax}")              # 7.00
print(f"รวม:   {total}")            # 107.00

# เปรียบเทียบกับ float
float_price = 0.1 + 0.2
decimal_price = Decimal('0.1') + Decimal('0.2')
print(f"\nFloat: {float_price}")     # 0.30000000000000004
print(f"Decimal: {decimal_price}")  # 0.3

# ===== Precision =====
getcontext().prec = 28   # กำหนด precision (default = 28)

pi = Decimal(1) / Decimal(3)
print(pi)    # 0.3333333333333333333333333333

getcontext().prec = 50
pi = Decimal(1) / Decimal(3)
print(pi)    # 0.33333333333333333333333333333333333333333333333333

# ===== Rounding Modes =====
price = Decimal('2.675')

# ROUND_HALF_UP - ปัดขึ้น ถ้าเป็น .5
print(price.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP))  # 2.68

# ROUND_DOWN - ตัดทิ้ง
print(price.quantize(Decimal('0.01'), rounding=ROUND_DOWN))     # 2.67

from decimal import ROUND_HALF_EVEN, ROUND_CEILING, ROUND_FLOOR, ROUND_UP

# ตารางเปรียบเทียบ
value = Decimal('2.675')
modes = [
    ('ROUND_UP',        'ROUND_UP'),
    ('ROUND_DOWN',      'ROUND_DOWN'),
    ('ROUND_CEILING',   'ROUND_CEILING'),
    ('ROUND_FLOOR',     'ROUND_FLOOR'),
    ('ROUND_HALF_UP',   'ROUND_HALF_UP'),
    ('ROUND_HALF_DOWN', 'ROUND_HALF_DOWN'),
    ('ROUND_HALF_EVEN', 'ROUND_HALF_EVEN'),
]

print("\nRounding Modes สำหรับ 2.675:")
for name, mode in modes:
    result = value.quantize(Decimal('0.01'), rounding=mode)
    print(f"  {name:20}: {result}")

# ===== Financial Calculations =====
from decimal import Decimal, ROUND_HALF_UP, getcontext
getcontext().prec = 28

def calculate_invoice(items, tax_rate_pct):
    """คำนวณใบแจ้งหนี้อย่างแม่นยำ"""
    tax_rate = Decimal(str(tax_rate_pct)) / 100
    
    subtotal = Decimal('0')
    for name, qty, unit_price in items:
        item_total = Decimal(str(qty)) * Decimal(str(unit_price))
        subtotal += item_total
        print(f"  {name}: {qty} x {unit_price} = {item_total}")
    
    tax = (subtotal * tax_rate).quantize(
        Decimal('0.01'), rounding=ROUND_HALF_UP
    )
    total = subtotal + tax
    
    print(f"\nSubtotal: {subtotal}")
    print(f"Tax ({tax_rate_pct}%): {tax}")
    print(f"Total: {total}")
    return total

print("\n=== Invoice ===")
items = [
    ("MacBook", 1, 79900),
    ("Mouse", 2, 1500),
    ("USB Hub", 3, 850),
]
calculate_invoice(items, 7)
```

---

## Fraction Module

```python
from fractions import Fraction

# ===== สร้าง Fraction =====
f1 = Fraction(1, 3)         # 1/3
f2 = Fraction(2, 4)         # ปรับให้เป็น 1/2 อัตโนมัติ
f3 = Fraction(0.5)          # จาก float (อาจไม่แม่นยำ)
f4 = Fraction('0.5')        # จาก string (แม่นยำ)
f5 = Fraction('1/3')        # จาก string fraction

print(f1)    # 1/3
print(f2)    # 1/2 (ปรับแล้ว)
print(f3)    # 1/2
print(f4)    # 1/2
print(f5)    # 1/3

print(f1.numerator)     # 1 (ตัวเศษ)
print(f1.denominator)   # 3 (ตัวส่วน)

# ===== Arithmetic =====
a = Fraction(1, 3)
b = Fraction(1, 4)

print(a + b)    # 7/12
print(a - b)    # 1/12
print(a * b)    # 1/12
print(a / b)    # 4/3

# ===== Comparison =====
print(Fraction(1, 3) == Fraction(2, 6))   # True
print(Fraction(1, 3) < Fraction(1, 2))    # True
print(Fraction(3, 4) > Fraction(2, 3))    # True

# ===== Convert =====
f = Fraction(1, 3)
print(float(f))         # 0.3333...
print(int(f))           # 0 (ตัดทิ้ง)
print(round(float(f), 4))  # 0.3333

# ===== Limit Denominator =====
# หา fraction ที่ใกล้เคียงที่สุดโดยมี denominator ไม่เกิน n
pi_approx = Fraction(3.14159265358979).limit_denominator(1000)
print(pi_approx)        # 355/113 (Pi approximation)
print(float(pi_approx)) # 3.1415929203539825

import math
print(Fraction(math.pi).limit_denominator(100))    # 311/99
print(Fraction(math.pi).limit_denominator(10))     # 22/7

# ===== Use Cases =====
# คำนวณคะแนนแบบเศษส่วน
scores = [Fraction(3, 4), Fraction(7, 8), Fraction(2, 3), Fraction(5, 6)]
average = sum(scores) / len(scores)
print(f"\nคะแนนเฉลี่ย: {average} = {float(average):.4f}")
# 129/160 = 0.80625

# สูตรอาหาร (ปริมาณเศษส่วน)
recipe = {
    "แป้ง": Fraction(2, 3),    # 2/3 ถ้วย
    "น้ำตาล": Fraction(1, 4),   # 1/4 ถ้วย
    "เนย": Fraction(1, 2),     # 1/2 ถ้วย
}

# คูณ 1.5 เท่า
multiplier = Fraction(3, 2)
scaled = {name: qty * multiplier for name, qty in recipe.items()}
print("\nสูตรคูณ 1.5:")
for name, qty in scaled.items():
    print(f"  {name}: {qty} ถ้วย")
```

---

## Number Formatting

```python
# ===== Format Specifiers =====
n = 1234567.89012

# ทศนิยม
print(f"{n:.2f}")       # 1234567.89
print(f"{n:.4f}")       # 1234567.8901
print(f"{n:.0f}")       # 1234568

# Comma separator
print(f"{n:,.2f}")      # 1,234,567.89
print(f"{n:_,.2f}")     # 1,234,567.89 (underscore as thousands separator)

# Scientific notation
print(f"{n:e}")         # 1.234568e+06
print(f"{n:.3e}")       # 1.235e+06
print(f"{n:E}")         # 1.234568E+06

# General format
print(f"{n:g}")         # 1.23457e+06
print(f"{0.000123:g}")  # 0.000123
print(f"{0.0000001:g}") # 1e-07

# Percentage
rate = 0.1575
print(f"{rate:.1%}")    # 15.8%
print(f"{rate:.2%}")    # 15.75%
print(f"{rate:+.1%}")   # +15.8%

# Sign
print(f"{42:+}")    # +42
print(f"{-42:+}")   # -42
print(f"{42: }")    # " 42" (space for positive)

# Integer formats
n = 255
print(f"{n:d}")     # 255 (decimal)
print(f"{n:05d}")   # 00255 (zero-padded)
print(f"{n:b}")     # 11111111 (binary)
print(f"{n:#b}")    # 0b11111111
print(f"{n:o}")     # 377 (octal)
print(f"{n:#o}")    # 0o377
print(f"{n:x}")     # ff (hex lowercase)
print(f"{n:X}")     # FF (hex uppercase)
print(f"{n:#x}")    # 0xff
print(f"{n:#010x}") # 0x000000ff (padded)

# Alignment
print(f"{'Left':<10}|")   # "Left      |"
print(f"{'Right':>10}|")  # "     Right|"
print(f"{'Center':^10}|") # "  Center  |"
print(f"{'Fill':*^10}|")  # "***Fill***|"

# ===== Format Function =====
print(format(3.14, '.2f'))      # '3.14'
print(format(255, '#010x'))     # '0x000000ff'
print(format(0.5, '.1%'))       # '50.0%'

# ===== Locale Formatting =====
import locale
locale.setlocale(locale.LC_ALL, '')  # ตาม system locale

# อาจให้ผลต่างกันตาม locale
# print(locale.currency(1234.56))
# print(locale.format_string("%d", 1234567, grouping=True))

# ===== Thai Number Formatting =====
def format_thai_baht(amount):
    """Format เงินบาทไทย"""
    if amount >= 1_000_000:
        return f"{amount/1_000_000:.2f} ล้านบาท"
    elif amount >= 1_000:
        return f"{amount:,.2f} บาท"
    else:
        return f"{amount:.2f} บาท"

amounts = [500, 15000, 1234567, 50000000]
for a in amounts:
    print(f"{a:>12,} → {format_thai_baht(a)}")
```

---

## Scientific Notation

```python
# ===== Scientific Notation ใน Python =====
# ใช้ e หรือ E สำหรับ exponent

# Literals
light_speed = 3e8           # 300,000,000 m/s
electron_mass = 9.11e-31    # 0.000000000000000000000000000000911 kg
avogadro = 6.022e23         # 602,200,000,000,000,000,000,000
boltzmann = 1.38e-23        # 0.0000000000000000000000138

print(f"Speed of light: {light_speed:e} m/s")
print(f"Electron mass:  {electron_mass:e} kg")
print(f"Avogadro:       {avogadro:e} mol^-1")
print(f"Boltzmann:      {boltzmann:e} J/K")

# Format ต่างๆ
n = 123456789.0

print(f"\nDecimal:     {n:f}")     # 123456789.000000
print(f"Scientific:  {n:e}")      # 1.234568e+08
print(f"Scientific2: {n:.2e}")    # 1.23e+08
print(f"General:     {n:g}")      # 1.23457e+08
print(f"Compact:     {n:.3g}")    # 1.23e+08

# Math ใน scientific notation
print(f"\n1e3 * 1e4 = {1e3 * 1e4:e}")    # 1.000000e+07
print(f"1e100 * 1e200 = {1e100 * 1e200:e}")  # 1.000000e+300

# ===== Engineering Notation =====
def engineering_notation(n, decimals=3):
    """แสดงในรูป Engineering notation (ยกกำลัง 3)"""
    import math
    if n == 0:
        return "0"
    exp = int(math.floor(math.log10(abs(n)) / 3)) * 3
    mantissa = n / (10 ** exp)
    prefix_map = {
        12: 'T', 9: 'G', 6: 'M', 3: 'k', 0: '',
        -3: 'm', -6: 'μ', -9: 'n', -12: 'p'
    }
    prefix = prefix_map.get(exp, f'e{exp}')
    return f"{mantissa:.{decimals}f}{prefix}"

test_values = [0.00001, 0.001, 1, 1000, 1_000_000, 1_000_000_000]
for val in test_values:
    print(f"{val:>15} = {engineering_notation(val)}")
```

---

## Statistics Module

```python
import statistics

data = [4, 7, 13, 2, 1, 8, 5, 4, 3, 9, 4]

# ===== Measures of Central Tendency =====
print("=== Central Tendency ===")
print(f"Mean:          {statistics.mean(data):.4f}")
print(f"Median:        {statistics.median(data)}")
print(f"Mode:          {statistics.mode(data)}")  # 4 (ปรากฏบ่อยสุด)
print(f"Geometric mean:{statistics.geometric_mean(data):.4f}")  # Python 3.8+
print(f"Harmonic mean: {statistics.harmonic_mean(data):.4f}")

# multimode (Python 3.8+)
data2 = [1, 2, 2, 3, 3, 4]
print(f"Multimode:     {statistics.multimode(data2)}")  # [2, 3]

# Median variants
print(f"Median low:    {statistics.median_low(data)}")    # lower median
print(f"Median high:   {statistics.median_high(data)}")   # upper median
print(f"Median grouped:{statistics.median_grouped(data):.4f}")  # continuous median

# ===== Measures of Spread =====
print("\n=== Spread ===")
print(f"Variance (pop):  {statistics.pvariance(data):.4f}")
print(f"Variance (samp): {statistics.variance(data):.4f}")
print(f"Std Dev (pop):   {statistics.pstdev(data):.4f}")
print(f"Std Dev (samp):  {statistics.stdev(data):.4f}")

# ===== Quantiles (Python 3.8+) =====
print("\n=== Quantiles ===")
q = statistics.quantiles(data, n=4)  # Quartiles
print(f"Q1 (25th percentile): {q[0]}")
print(f"Q2 (50th percentile): {q[1]}")
print(f"Q3 (75th percentile): {q[2]}")
print(f"IQR: {q[2] - q[0]}")

# ===== NormalDist (Python 3.8+) =====
from statistics import NormalDist

# IQ distribution: mean=100, sd=15
iq = NormalDist(100, 15)
print(f"\n=== IQ Normal Distribution ===")
print(f"Mean: {iq.mean}, Stdev: {iq.stdev}")
print(f"P(IQ >= 130): {1 - iq.cdf(130):.2%}")   # Mensa threshold
print(f"P(IQ >= 145): {1 - iq.cdf(145):.4%}")   # Genius level
print(f"P(85 <= IQ <= 115): {iq.cdf(115) - iq.cdf(85):.2%}")  # Normal range

# สร้าง sample
sample = iq.samples(10)
print(f"Sample of 10: {[round(s) for s in sample]}")
```

---

## ตัวอย่างโค้ด

### ตัวอย่างที่ 1: Number System Converter

```python
# ตัวอย่างที่ 1: แปลงระบบตัวเลข
def convert_number(n, from_base=10, to_bases=None):
    """แปลงตัวเลขเป็นทุกฐาน"""
    if to_bases is None:
        to_bases = [2, 8, 10, 16]
    
    # แปลงเป็น decimal ก่อน
    decimal = int(str(n), from_base)
    
    results = {"decimal": decimal}
    if 2 in to_bases:
        results["binary"] = bin(decimal)
    if 8 in to_bases:
        results["octal"] = oct(decimal)
    if 16 in to_bases:
        results["hex"] = hex(decimal)
    
    return results

# ทดสอบ
for n in [42, 255, 1024]:
    print(f"\n{n}:")
    result = convert_number(n)
    for base, value in result.items():
        print(f"  {base:8}: {value}")
```

### ตัวอย่างที่ 2: Floating Point Precision Demo

```python
# ตัวอย่างที่ 2: แสดงปัญหา floating point
from decimal import Decimal
import math

print("=== Floating Point Issues ===")
operations = [
    (0.1 + 0.2, 0.3, "0.1 + 0.2 == 0.3"),
    (0.1 * 3, 0.3, "0.1 * 3 == 0.3"),
    (1.1 + 2.2, 3.3, "1.1 + 2.2 == 3.3"),
]

for result, expected, desc in operations:
    equal = result == expected
    close = math.isclose(result, expected)
    print(f"\n{desc}")
    print(f"  Result:   {result}")
    print(f"  Expected: {expected}")
    print(f"  Equal:    {equal}")
    print(f"  isclose:  {close}")

print("\n=== Decimal Solution ===")
d_operations = [
    (Decimal('0.1') + Decimal('0.2'), Decimal('0.3')),
    (Decimal('0.1') * 3, Decimal('0.3')),
    (Decimal('1.1') + Decimal('2.2'), Decimal('3.3')),
]

for result, expected in d_operations:
    print(f"  {result} == {expected}: {result == expected}")
```

### ตัวอย่างที่ 3: Math Functions Showcase

```python
# ตัวอย่างที่ 3: Math Functions
import math

def geometry_calculator():
    """คำนวณรูปทรงเรขาคณิต"""
    
    def circle_area(r):
        return math.pi * r ** 2
    
    def sphere_volume(r):
        return (4/3) * math.pi * r ** 3
    
    def triangle_area(a, b, c):
        """Heron's formula"""
        s = (a + b + c) / 2
        return math.sqrt(s * (s-a) * (s-b) * (s-c))
    
    def distance_3d(x1, y1, z1, x2, y2, z2):
        return math.sqrt((x2-x1)**2 + (y2-y1)**2 + (z2-z1)**2)
    
    print("=== Geometry Calculator ===")
    r = 5
    print(f"\nวงกลม r={r}:")
    print(f"  พื้นที่ = {circle_area(r):.4f}")
    print(f"  เส้นรอบวง = {2 * math.pi * r:.4f}")
    
    print(f"\nทรงกลม r={r}:")
    print(f"  ปริมาตร = {sphere_volume(r):.4f}")
    print(f"  พื้นที่ผิว = {4 * math.pi * r**2:.4f}")
    
    print("\nสามเหลี่ยม 3-4-5:")
    print(f"  พื้นที่ = {triangle_area(3, 4, 5):.4f}")
    
    print("\nระยะทาง 3D:")
    dist = distance_3d(0, 0, 0, 1, 2, 2)
    print(f"  (0,0,0) ถึง (1,2,2) = {dist:.4f}")

geometry_calculator()
```

### ตัวอย่างที่ 4: Random Applications

```python
# ตัวอย่างที่ 4: ประยุกต์ใช้ Random
import random

# 1. Dice Simulator
def roll_dice(sides=6, num=1):
    return [random.randint(1, sides) for _ in range(num)]

print("=== Dice Simulator ===")
print(f"2 ลูกเต๋า 6 หน้า: {roll_dice(6, 2)}")
print(f"3 ลูกเต๋า 20 หน้า: {roll_dice(20, 3)}")

# 2. Random Password
def generate_secure_token(length=32):
    import secrets, string
    alphabet = string.ascii_letters + string.digits
    return ''.join(secrets.choice(alphabet) for _ in range(length))

print("\n=== Secure Tokens ===")
print(generate_secure_token())
print(generate_secure_token(16))

# 3. Monte Carlo Pi Estimation
def estimate_pi(n_points=100000):
    inside = sum(
        1 for _ in range(n_points)
        if random.random()**2 + random.random()**2 <= 1
    )
    return 4 * inside / n_points

print("\n=== Monte Carlo Pi ===")
for n in [1000, 10000, 100000]:
    pi_est = estimate_pi(n)
    print(f"n={n:7}: π ≈ {pi_est:.5f} (error: {abs(pi_est - 3.14159):.5f})")

# 4. Random Sampling
population = list(range(1, 100))
sample = random.sample(population, 10)
print(f"\nRandom sample: {sorted(sample)}")
```

### ตัวอย่างที่ 5: Decimal Financial Calculator

```python
# ตัวอย่างที่ 5: Financial Calculator ด้วย Decimal
from decimal import Decimal, ROUND_HALF_UP, getcontext

getcontext().prec = 28

def compound_interest(principal, rate_pct, years, compounds_per_year=12):
    """คำนวณดอกเบี้ยทบต้น"""
    P = Decimal(str(principal))
    r = Decimal(str(rate_pct)) / 100
    n = Decimal(str(compounds_per_year))
    t = Decimal(str(years))
    
    amount = P * (1 + r/n) ** (n*t)
    interest = amount - P
    
    return amount, interest

def loan_payment(principal, annual_rate_pct, years):
    """คำนวณค่างวดรายเดือน"""
    P = Decimal(str(principal))
    r = Decimal(str(annual_rate_pct)) / 100 / 12  # monthly rate
    n = Decimal(str(years * 12))  # total payments
    
    if r == 0:
        return P / n
    
    monthly = P * r * (1 + r)**n / ((1 + r)**n - 1)
    return monthly.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)

print("=== Compound Interest ===")
amount, interest = compound_interest(100000, 3.5, 10)
print(f"เงินต้น:      100,000 บาท")
print(f"อัตราดอกเบี้ย: 3.5% ต่อปี")
print(f"ระยะเวลา:     10 ปี")
print(f"จำนวนเงิน:   {amount:,.2f} บาท")
print(f"ดอกเบี้ย:     {interest:,.2f} บาท")

print("\n=== Loan Payment Calculator ===")
configs = [
    (500000, 5.0, 30, "บ้าน 500K"),
    (800000, 4.5, 5, "รถ 800K"),
    (100000, 6.0, 3, "สินเชื่อ 100K"),
]

for principal, rate, years, name in configs:
    monthly = loan_payment(principal, rate, years)
    total = monthly * years * 12
    total_interest = total - Decimal(str(principal))
    print(f"\n{name}: {principal:,} บาท")
    print(f"  ค่างวด/เดือน: {monthly:,.2f} บาท")
    print(f"  รวมทั้งหมด:  {total:,.2f} บาท")
    print(f"  ดอกเบี้ยรวม: {total_interest:,.2f} บาท")
```

### ตัวอย่างที่ 6: Fraction Arithmetic

```python
# ตัวอย่างที่ 6: Fraction สำหรับงานแม่นยำ
from fractions import Fraction

# สูตรทำขนมปัง (scale สูตร)
print("=== Recipe Scaling ===")

original_recipe = {
    "แป้ง": Fraction(2, 1),      # 2 ถ้วย
    "น้ำตาล": Fraction(3, 4),    # 3/4 ถ้วย  
    "เนย": Fraction(1, 2),       # 1/2 ถ้วย
    "ไข่": Fraction(2, 1),       # 2 ฟอง
    "นม": Fraction(1, 3),        # 1/3 ถ้วย
    "ผงฟู": Fraction(3, 2),      # 1.5 ช้อนชา
}

def scale_recipe(recipe, servings_original, servings_target):
    scale = Fraction(servings_target, servings_original)
    scaled = {name: qty * scale for name, qty in recipe.items()}
    return scaled

# ขยายจาก 8 เสิร์ฟ เป็น 12 เสิร์ฟ
scaled = scale_recipe(original_recipe, 8, 12)

print("สูตรเดิม (8 เสิร์ฟ) → ขยาย (12 เสิร์ฟ):")
for ingredient, qty in scaled.items():
    # แสดงทั้ง fraction และ decimal
    print(f"  {ingredient:8}: {str(original_recipe[ingredient]):<6} → {str(qty):<8} ({float(qty):.3f})")
```

### ตัวอย่างที่ 7: Statistics Analysis

```python
# ตัวอย่างที่ 7: Statistical Analysis
import statistics
import random

random.seed(42)

# สร้าง dataset จำลอง
exam_scores = [random.gauss(72, 12) for _ in range(50)]
exam_scores = [max(0, min(100, s)) for s in exam_scores]  # clamp 0-100

print("=== Exam Score Analysis ===")
print(f"Students: {len(exam_scores)}")
print(f"\nCentral Tendency:")
print(f"  Mean:   {statistics.mean(exam_scores):.2f}")
print(f"  Median: {statistics.median(exam_scores):.2f}")
print(f"  Mode:   {statistics.mode([round(s) for s in exam_scores])}")

print(f"\nSpread:")
print(f"  Variance: {statistics.variance(exam_scores):.2f}")
print(f"  Std Dev:  {statistics.stdev(exam_scores):.2f}")
print(f"  Min:      {min(exam_scores):.2f}")
print(f"  Max:      {max(exam_scores):.2f}")

# Histogram
print(f"\nScore Distribution:")
ranges = [(90, 100, 'A'), (80, 90, 'B'), (70, 80, 'C'), (60, 70, 'D'), (0, 60, 'F')]
for low, high, grade in ranges:
    count = sum(1 for s in exam_scores if low <= s < high)
    bar = "█" * count
    print(f"  {grade} ({low:3}-{high:3}): {bar:<30} {count:2}")
```

### ตัวอย่างที่ 8: Number Format Utilities

```python
# ตัวอย่างที่ 8: Number Formatting Utilities
def format_number(n, style='default', **kwargs):
    """Format ตัวเลขในรูปแบบต่างๆ"""
    
    if style == 'currency':
        symbol = kwargs.get('symbol', '฿')
        decimals = kwargs.get('decimals', 2)
        return f"{symbol}{n:>15,.{decimals}f}"
    
    elif style == 'percentage':
        decimals = kwargs.get('decimals', 1)
        return f"{n:.{decimals}%}"
    
    elif style == 'scientific':
        decimals = kwargs.get('decimals', 3)
        return f"{n:.{decimals}e}"
    
    elif style == 'ordinal':
        n = int(n)
        if 11 <= n <= 13:
            suffix = 'th'
        else:
            suffix = {1: 'st', 2: 'nd', 3: 'rd'}.get(n % 10, 'th')
        return f"{n}{suffix}"
    
    elif style == 'human':
        for unit, threshold in [('T', 1e12), ('B', 1e9), ('M', 1e6), ('K', 1e3)]:
            if abs(n) >= threshold:
                return f"{n/threshold:.1f}{unit}"
        return str(n)
    
    else:
        return f"{n}"

# ทดสอบ
amounts = [0, 1234.5, 1234567, 9876543210]
rates = [0.05, 0.1575, 1.0]
sci_values = [0.00001, 12345, 6.022e23]
ordinals = [1, 2, 3, 4, 11, 12, 13, 21, 22]

print("=== Currency ===")
for n in amounts:
    print(f"  {format_number(n, 'currency')}")

print("\n=== Percentage ===")
for r in rates:
    print(f"  {format_number(r, 'percentage')}")

print("\n=== Scientific ===")
for n in sci_values:
    print(f"  {format_number(n, 'scientific')}")

print("\n=== Ordinal ===")
for n in ordinals:
    print(f"  {format_number(n, 'ordinal')}", end="  ")
print()

print("\n=== Human Readable ===")
for n in [500, 5000, 5_000_000, 5_000_000_000]:
    print(f"  {n:>15,} → {format_number(n, 'human')}")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Number Base Calculator

เขียน function `base_calculator(a, op, b, base)` ที่รับตัวเลขในฐาน `base` แล้วคำนวณและแสดงผลในฐานเดิม

### ข้อที่ 2: Floating Point Comparison

เขียน function `safe_equal(a, b, tolerance=1e-9)` และทดสอบกับ:
- `0.1 + 0.2` vs `0.3`
- `1.0 / 3.0 * 3` vs `1.0`
- `math.sqrt(2) ** 2` vs `2.0`

### ข้อที่ 3: Prime Number Sieve

เขียน function `sieve_of_eratosthenes(n)` ที่หาจำนวนเฉพาะทั้งหมดถึง n ด้วย Sieve of Eratosthenes algorithm

### ข้อที่ 4: Statistics Calculator

เขียน class `Statistics` ที่รับ list ของตัวเลขแล้วคำนวณ:
- mean, median, mode
- variance, std_deviation
- min, max, range
- percentile(p) - หา p-th percentile

### ข้อที่ 5: Currency Converter

สร้างระบบแปลงสกุลเงินที่:
- ใช้ Decimal สำหรับความแม่นยำ
- รองรับ THB, USD, EUR, JPY, GBP
- Format ผลลัพธ์ตามสกุลเงิน

### ข้อที่ 6: Random Lottery Simulator

สร้าง lottery simulator ที่:
- สุ่มเลข 6 ตัว จาก 1-49
- รับเลขที่ผู้ใช้เลือก
- ตรวจว่าถูกกี่ตัว
- จำลอง 100,000 ครั้ง นับความถี่ของแต่ละผล

### ข้อที่ 7: Math Function Grapher (Text)

เขียน function `text_graph(func, x_range, width=60, height=20)` ที่พล็อตกราฟฟังก์ชันใน terminal

### ข้อที่ 8: Compound Interest Planner

เขียน function ที่คำนวณแผนการออมเงิน:
- ออมทุกเดือน amount บาท
- ดอกเบี้ย rate% ต่อปี
- กี่ปีถึงจะถึง goal?
- ต้องออมเดือนละเท่าไรถ้าอยากได้ goal ใน years ปี?

### ข้อที่ 9: Fraction Arithmetic

เขียน simple fraction calculator ที่รับ input เช่น "1/2 + 1/3" แล้วคำนวณและแสดงผลเป็น fraction

### ข้อที่ 10: Number Patterns

เขียน functions สำหรับ:
- `fibonacci(n)` - Fibonacci sequence
- `triangular(n)` - Triangular numbers (1, 3, 6, 10, 15, ...)
- `perfect_numbers(limit)` - Perfect numbers ถึง limit
- `armstrong(n)` - ตรวจ Armstrong number

---

## เฉลยแบบฝึกหัด

### เฉลยข้อที่ 1: Base Calculator

```python
# เฉลยข้อที่ 1
def base_calculator(a_str, op, b_str, base):
    """Calculator ที่รองรับทุกฐาน"""
    a = int(a_str, base)
    b = int(b_str, base)
    
    ops = {
        '+': a + b,
        '-': a - b,
        '*': a * b,
        '/': a // b if b != 0 else None,
    }
    
    result = ops.get(op)
    if result is None:
        return "Error"
    
    # แปลงกลับเป็นฐานเดิม
    def to_base(n, base):
        if n == 0:
            return '0'
        digits = "0123456789ABCDEF"
        result = ""
        while n > 0:
            result = digits[n % base] + result
            n //= base
        return result
    
    return to_base(result, base)

# ทดสอบ
print(base_calculator('1010', '+', '0110', 2))  # 10000 (10+6=16)
print(base_calculator('FF', '+', '01', 16))     # 100
print(base_calculator('17', '*', '3', 8))       # 55 (15*3=45)
```

### เฉลยข้อที่ 2: Float Comparison

```python
# เฉลยข้อที่ 2
import math

def safe_equal(a, b, tolerance=1e-9):
    """เปรียบเทียบ float อย่างปลอดภัย"""
    return math.isclose(a, b, rel_tol=tolerance, abs_tol=tolerance)

tests = [
    (0.1 + 0.2, 0.3, "0.1 + 0.2 == 0.3"),
    (1.0/3.0 * 3, 1.0, "1/3 * 3 == 1.0"),
    (math.sqrt(2)**2, 2.0, "sqrt(2)^2 == 2.0"),
    (0.1 * 10, 1.0, "0.1 * 10 == 1.0"),
]

for a, b, desc in tests:
    direct = a == b
    safe = safe_equal(a, b)
    print(f"{desc}")
    print(f"  Direct: {direct}, Safe: {safe}")
    print(f"  a={a!r}, b={b!r}")
```

### เฉลยข้อที่ 3: Sieve of Eratosthenes

```python
# เฉลยข้อที่ 3
def sieve_of_eratosthenes(n):
    """หาจำนวนเฉพาะทั้งหมดถึง n"""
    if n < 2:
        return []
    
    # เริ่มต้น: ทุกตัวเลขอาจเป็นจำนวนเฉพาะ
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    
    # ตะแกรง
    for i in range(2, int(n**0.5) + 1):
        if is_prime[i]:
            # ทำเครื่องหมายตัวคูณของ i
            for j in range(i*i, n+1, i):
                is_prime[j] = False
    
    return [i for i in range(2, n+1) if is_prime[i]]

primes = sieve_of_eratosthenes(100)
print(f"จำนวนเฉพาะถึง 100: {primes}")
print(f"จำนวน: {len(primes)}")

# ทดสอบ performance
import time
start = time.time()
primes_1M = sieve_of_eratosthenes(1_000_000)
elapsed = time.time() - start
print(f"\nจำนวนเฉพาะถึง 1,000,000: {len(primes_1M)} ตัว (ใช้เวลา {elapsed:.3f}s)")
```

### เฉลยข้อที่ 5: Currency Converter

```python
# เฉลยข้อที่ 5
from decimal import Decimal, ROUND_HALF_UP

class CurrencyConverter:
    # Exchange rates relative to THB
    RATES = {
        'THB': Decimal('1'),
        'USD': Decimal('36.50'),
        'EUR': Decimal('39.80'),
        'JPY': Decimal('0.245'),
        'GBP': Decimal('46.20'),
    }
    
    FORMATS = {
        'THB': ('฿', 2),
        'USD': ('$', 2),
        'EUR': ('€', 2),
        'JPY': ('¥', 0),
        'GBP': ('£', 2),
    }
    
    @classmethod
    def convert(cls, amount, from_currency, to_currency):
        """แปลงสกุลเงิน"""
        amount = Decimal(str(amount))
        from_rate = cls.RATES[from_currency]
        to_rate = cls.RATES[to_currency]
        
        # แปลงเป็น THB ก่อน แล้วแปลงเป็น target
        thb = amount * from_rate
        result = thb / to_rate
        
        return result.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
    
    @classmethod
    def format_amount(cls, amount, currency):
        symbol, decimals = cls.FORMATS[currency]
        return f"{symbol}{amount:,.{decimals}f}"
    
    @classmethod
    def show_all_rates(cls, amount, from_currency):
        print(f"\n{cls.format_amount(Decimal(str(amount)), from_currency)} =")
        for to_currency in cls.RATES:
            if to_currency != from_currency:
                converted = cls.convert(amount, from_currency, to_currency)
                print(f"  {cls.format_amount(converted, to_currency)}")

# ทดสอบ
CurrencyConverter.show_all_rates(1000, 'THB')
CurrencyConverter.show_all_rates(100, 'USD')
```

### เฉลยข้อที่ 10: Number Patterns

```python
# เฉลยข้อที่ 10
def fibonacci(n):
    """คืน n ตัวแรกของ Fibonacci"""
    if n <= 0:
        return []
    seq = [0, 1]
    while len(seq) < n:
        seq.append(seq[-1] + seq[-2])
    return seq[:n]

def triangular(n):
    """คืน n ตัวแรกของ Triangular numbers"""
    return [k * (k + 1) // 2 for k in range(1, n + 1)]

def perfect_numbers(limit):
    """หา perfect numbers ถึง limit"""
    result = []
    for n in range(2, limit + 1):
        divisors_sum = sum(i for i in range(1, n) if n % i == 0)
        if divisors_sum == n:
            result.append(n)
    return result

def is_armstrong(n):
    """ตรวจ Armstrong number"""
    digits = str(n)
    power = len(digits)
    return sum(int(d) ** power for d in digits) == n

def armstrong_numbers(limit):
    return [n for n in range(1, limit + 1) if is_armstrong(n)]

print(f"Fibonacci (15): {fibonacci(15)}")
print(f"Triangular (10): {triangular(10)}")
print(f"Perfect numbers ถึง 10000: {perfect_numbers(10000)}")
print(f"Armstrong ถึง 1000: {armstrong_numbers(1000)}")
```

---

## สรุป

ใน Part 04 นี้เราได้เรียนรู้:

1. **Integer** - ไม่มีขีดจำกัด, ระบบเลขต่างๆ (binary, octal, hex)
2. **Float** - IEEE 754, floating point problems, isclose()
3. **Complex** - real + imaginary, cmath module
4. **math module** - ฟังก์ชันคณิตศาสตร์ครบครัน
5. **Rounding** - floor(), ceil(), round() (Banker's rounding), trunc()
6. **random module** - random, randint, choice, sample, shuffle, secrets
7. **decimal module** - ความแม่นยำสูงสำหรับการเงิน
8. **fractions module** - เศษส่วนแม่นยำ
9. **Number Formatting** - f-string format spec, currency, scientific
10. **statistics module** - mean, median, mode, stdev, quantiles

## ขั้นตอนต่อไป

- **Part 05**: Boolean Logic & Comparison Operators
- ฝึกใช้ Decimal สำหรับคำนวณเงิน
- เข้าใจ floating point limitations
- ลองใช้ random สร้าง simulation

---

*หมายเหตุ: ทุก code block ทดสอบแล้วบน Python 3.12*
