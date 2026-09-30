# Part 03: Strings & String Methods

## สารบัญ (Table of Contents)

1. [String คืออะไร?](#string-คืออะไร)
2. [การสร้าง String แบบต่างๆ](#การสร้าง-string-แบบต่างๆ)
3. [String Indexing](#string-indexing)
4. [String Slicing](#string-slicing)
5. [String Methods ทั้งหมด](#string-methods-ทั้งหมด)
6. [String Formatting](#string-formatting)
7. [Raw Strings และ Escape Characters](#raw-strings-และ-escape-characters)
8. [String Concatenation และ Repetition](#string-concatenation-และ-repetition)
9. [Multiline Strings](#multiline-strings)
10. [String Comparison](#string-comparison)
11. [Regular String Operations](#regular-string-operations)
12. [ตัวอย่างโค้ด 30+ ตัวอย่าง](#ตัวอย่างโค้ด)
13. [แบบฝึกหัด](#แบบฝึกหัด)
14. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## String คืออะไร?

String (str) คือชนิดข้อมูลที่ใช้เก็บข้อความ ใน Python strings เป็น **immutable** (ไม่สามารถแก้ไขตัวอักษรแต่ละตัวได้) และเป็น **sequence** ของ Unicode characters

```python
# String เป็น sequence ของ characters
s = "Python"
print(len(s))    # 6 (จำนวนตัวอักษร)
print(s[0])      # P (ตัวแรก)
print(s[-1])     # n (ตัวสุดท้าย)

# String เป็น immutable
try:
    s[0] = "J"    # TypeError: 'str' object does not support item assignment
except TypeError as e:
    print(f"Error: {e}")

# แต่สร้างใหม่ได้
s = "J" + s[1:]  # สร้าง string ใหม่
print(s)          # Jython
```

### String ใน Memory

```
s = "Hello"

Index:    0   1   2   3   4
Char:    'H' 'e' 'l' 'l' 'o'
Neg:     -5  -4  -3  -2  -1
```

---

## การสร้าง String แบบต่างๆ

### 1. Single Quotes

```python
s1 = 'Hello World'
s2 = 'Python\'s great'    # ใช้ escape สำหรับ apostrophe
s3 = "Python's great"     # หรือใช้ double quotes แทน
```

### 2. Double Quotes

```python
s1 = "Hello World"
s2 = "She said \"Hello\""    # escape double quote
s3 = 'She said "Hello"'      # หรือใช้ single quotes แทน
```

### 3. Triple Quotes (Multiline)

```python
# Triple single quotes
s1 = '''This is
a multiline
string'''

# Triple double quotes
s2 = """This is also
a multiline
string"""

# นิยมใช้กับ docstrings
def my_function():
    """
    นี่คือ docstring ของ function
    อธิบายการทำงานของ function
    
    Args:
        ไม่มี parameters
    
    Returns:
        ไม่มีค่า return
    """
    pass
```

### 4. String Constructors

```python
# str() constructor
s1 = str()           # "" (empty string)
s2 = str(42)         # "42"
s3 = str(3.14)       # "3.14"
s4 = str(True)       # "True"
s5 = str(None)       # "None"
s6 = str([1, 2, 3])  # "[1, 2, 3]"
```

### 5. String ที่ไม่ธรรมดา

```python
# Byte string (สำหรับ binary data)
b = b"Hello"         # bytes object
print(type(b))       # <class 'bytes'>

# Raw string (ไม่แปล escape sequences)
r = r"C:\Users\name\Desktop"
print(r)              # C:\Users\name\Desktop

# F-string (formatted string)
name = "Alice"
f = f"Hello {name}!"
print(f)              # Hello Alice!

# Unicode string (Python 3 ทุก string เป็น unicode)
thai = "สวัสดีโลก"
emoji = "Python 🐍"
arabic = "مرحبا"
print(thai, emoji, arabic)
```

---

## String Indexing

```python
s = "Python Programming"
#    0123456789...

# Positive indexing (เริ่มจาก 0)
print(s[0])    # P (ตัวแรก)
print(s[1])    # y
print(s[6])    # ' ' (ช่องว่าง)
print(s[7])    # P

# Negative indexing (เริ่มจาก -1 = ตัวสุดท้าย)
print(s[-1])   # g (ตัวสุดท้าย)
print(s[-2])   # n
print(s[-11])  # P (ตัว 'P' ใน 'Programming')

# ดู index ทั้งหมด
for i, char in enumerate(s):
    print(f"[{i:2}] [{i-len(s):3}] = {repr(char)}")

# IndexError เมื่อ index เกิน
try:
    print(s[100])
except IndexError as e:
    print(f"Error: {e}")
```

---

## String Slicing

```python
s = "Hello, World!"
#    0123456789012

# รูปแบบ: s[start:stop:step]
# start: เริ่มที่ index (รวม)
# stop:  หยุดที่ index (ไม่รวม)
# step:  ก้าวกระโดด (default 1)

# Basic slicing
print(s[0:5])    # Hello  (index 0-4)
print(s[7:12])   # World  (index 7-11)
print(s[7:])     # World! (จาก 7 ถึงจบ)
print(s[:5])     # Hello  (จากต้นถึง 4)
print(s[:])      # Hello, World! (ทั้งหมด)

# Negative slicing
print(s[-6:])    # World! (6 ตัวสุดท้าย)
print(s[-6:-1])  # World  (ไม่รวมตัวสุดท้าย)
print(s[:-1])    # Hello, World (ไม่รวมตัวสุดท้าย)

# Step
print(s[::2])    # Hlo ol!   (ทุกๆ 2 ตัว)
print(s[::3])    # Hl r!     (ทุกๆ 3 ตัว)
print(s[::-1])   # !dlroW ,olleH (กลับ string)
print(s[0:10:2]) # Hlo o     (ตั้งแต่ต้นถึง 9 ทุกๆ 2 ตัว)

# ตัวอย่าง use cases
email = "user@example.com"
domain = email[email.index('@')+1:]
print(f"Domain: {domain}")   # example.com

# กลับคำ
word = "Python"
reversed_word = word[::-1]
print(reversed_word)   # nohtyP

# ตรวจ palindrome
def is_palindrome(s):
    return s == s[::-1]

print(is_palindrome("racecar"))  # True
print(is_palindrome("Python"))   # False
```

---

## String Methods ทั้งหมด

### Case Methods (เปลี่ยนตัวพิมพ์)

```python
s = "hello world PYTHON"

print(s.upper())       # HELLO WORLD PYTHON (ตัวใหญ่ทั้งหมด)
print(s.lower())       # hello world python (ตัวเล็กทั้งหมด)
print(s.capitalize())  # Hello world python (ตัวแรกใหญ่)
print(s.title())       # Hello World Python (ทุกคำขึ้นต้นใหญ่)
print(s.swapcase())    # HELLO WORLD python (สลับ case)

# casefold() - เข้มกว่า lower() สำหรับ Unicode
german = "Straße"
print(german.lower())      # straße
print(german.casefold())   # strasse (เปลี่ยน ß เป็น ss)

# เปรียบเทียบโดยไม่สน case
s1 = "Python"
s2 = "PYTHON"
print(s1.casefold() == s2.casefold())  # True
```

### Strip Methods (ตัดช่องว่าง)

```python
s = "   Hello, World!   "

print(s.strip())       # "Hello, World!"   (ตัดทั้งสองข้าง)
print(s.lstrip())      # "Hello, World!   " (ตัดซ้าย)
print(s.rstrip())      # "   Hello, World!" (ตัดขวา)

# ตัด characters เฉพาะ
s2 = "***Hello***"
print(s2.strip('*'))   # Hello
print(s2.lstrip('*'))  # Hello***
print(s2.rstrip('*'))  # ***Hello

# ตัดหลายตัว
s3 = "...---Hello---..."
print(s3.strip('.-'))  # Hello

# ใช้งานจริง - ทำความสะอาด input
user_input = "  username@email.com  \n\t"
clean = user_input.strip()
print(repr(clean))   # 'username@email.com'
```

### Search Methods (ค้นหา)

```python
s = "Python is great. Python is easy."

# find() - คืน index, -1 ถ้าไม่พบ
print(s.find("Python"))         # 0
print(s.find("Python", 1))      # 17 (หาจาก index 1)
print(s.find("Python", 1, 10))  # -1 (หาในช่วง 1-9)
print(s.find("Java"))           # -1 (ไม่พบ)

# rfind() - หาจากขวา
print(s.rfind("Python"))        # 17 (ตัวสุดท้าย)

# index() - เหมือน find() แต่ throw exception ถ้าไม่พบ
print(s.index("Python"))        # 0
try:
    print(s.index("Java"))
except ValueError as e:
    print(f"Error: {e}")   # substring not found

# rindex() - หาจากขวา throw exception
print(s.rindex("Python"))       # 17

# count() - นับจำนวนครั้ง
print(s.count("Python"))        # 2
print(s.count("is"))            # 2
print(s.count("a"))             # 2
print(s.count("xyz"))           # 0

# count ช่วง
print(s.count("Python", 0, 10)) # 1 (นับเฉพาะช่วง 0-9)
```

### Check Methods (ตรวจสอบ)

```python
# startswith() และ endswith()
s = "Hello, World!"

print(s.startswith("Hello"))      # True
print(s.startswith("World"))      # False
print(s.startswith(("Hello", "Hi")))  # True (ตรวจหลาย prefix)

print(s.endswith("!"))            # True
print(s.endswith("World!"))       # True
print(s.endswith(("!", ".")))     # True

# ตรวจประเภท characters
print("123".isdigit())      # True (ตัวเลขทั้งหมด)
print("12.3".isdigit())     # False
print("abc".isalpha())      # True (ตัวอักษรทั้งหมด)
print("abc123".isalpha())   # False
print("abc123".isalnum())   # True (ตัวอักษรหรือตัวเลข)
print("  ".isspace())       # True (whitespace ทั้งหมด)
print("HELLO".isupper())    # True (ตัวใหญ่ทั้งหมด)
print("hello".islower())    # True (ตัวเล็กทั้งหมด)
print("Hello World".istitle())  # True (title case)

# ตรวจ ASCII
print("hello".isascii())     # True
print("สวัสดี".isascii())    # False

# ตรวจ identifier (ชื่อตัวแปรที่ถูกต้อง)
print("my_var".isidentifier())    # True
print("1var".isidentifier())      # False
print("class".isidentifier())     # True (keyword แต่ valid identifier)
import keyword
print(keyword.iskeyword("class")) # True (แต่เป็น keyword ใช้ไม่ได้)
```

### Replace Methods (แทนที่)

```python
s = "I love Python. Python is great!"

# replace() - แทนที่ทั้งหมด
print(s.replace("Python", "Java"))
# I love Java. Java is great!

# replace กี่ครั้ง
print(s.replace("Python", "Java", 1))
# I love Java. Python is great!

# แทนที่ช่องว่าง
s2 = "Hello   World"
print(s2.replace(" ", ""))    # HelloWorld
print(s2.replace("   ", " ")) # Hello World

# ลบ substring
print(s.replace("Python", ""))  # I love .  is great!
print(s.replace("Python. ", "").replace("Python", ""))  # I love  is great!
```

### Split และ Join Methods

```python
# split() - แบ่ง string
s = "apple,banana,cherry,date"

# split ด้วย delimiter
fruits = s.split(",")
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# split ด้วย whitespace (default)
sentence = "Hello World Python"
words = sentence.split()
print(words)  # ['Hello', 'World', 'Python']

# split จำกัดจำนวนครั้ง
result = s.split(",", 2)  # แบ่งแค่ 2 ครั้ง
print(result)  # ['apple', 'banana', 'cherry,date']

# rsplit() - แบ่งจากขวา
result = s.rsplit(",", 2)
print(result)  # ['apple,banana', 'cherry', 'date']

# splitlines() - แบ่งตาม newline
text = "line1\nline2\nline3"
lines = text.splitlines()
print(lines)  # ['line1', 'line2', 'line3']

# ตัวอย่างแยก CSV
csv_line = "สมชาย,25,กรุงเทพ,engineer"
name, age, city, job = csv_line.split(",")
print(f"ชื่อ: {name}, อายุ: {age}")

# join() - รวม string
fruits_list = ["apple", "banana", "cherry"]

# รวมด้วย separator
print(", ".join(fruits_list))    # apple, banana, cherry
print(" - ".join(fruits_list))   # apple - banana - cherry
print("".join(fruits_list))      # applebananacherry
print("|".join(fruits_list))     # apple|banana|cherry

# join กับ numbers (ต้องแปลงเป็น str ก่อน)
numbers = [1, 2, 3, 4, 5]
print(", ".join(str(n) for n in numbers))  # 1, 2, 3, 4, 5

# ตัวอย่างจริง: สร้าง SQL query
columns = ["name", "age", "city"]
values = ["สมชาย", "25", "กรุงเทพ"]

sql = f"INSERT INTO users ({', '.join(columns)}) VALUES ({', '.join(repr(v) for v in values)})"
print(sql)
```

### Padding Methods (เติม/จัดตำแหน่ง)

```python
s = "Hello"

# center() - จัดกลาง
print(s.center(20))         # "       Hello        "
print(s.center(20, '-'))    # "-------Hello--------"

# ljust() - ชิดซ้าย
print(s.ljust(20))          # "Hello               "
print(s.ljust(20, '.'))     # "Hello..............."

# rjust() - ชิดขวา
print(s.rjust(20))          # "               Hello"
print(s.rjust(20, '.'))     # "...............Hello"

# zfill() - เติม 0 ด้านหน้า (สำหรับตัวเลข)
print("42".zfill(5))        # "00042"
print("-42".zfill(5))       # "-0042"
print("3.14".zfill(8))      # "00003.14"

# ใช้งานจริง: แสดงตาราง
items = [("Apple", 1.50), ("Banana", 0.75), ("Cherry", 3.00)]
print(f"{'Item':<10} {'Price':>8}")
print("-" * 20)
for item, price in items:
    print(f"{item:<10} ${price:>7.2f}")
```

### Encode/Decode Methods

```python
# encode() - แปลง str เป็น bytes
s = "Hello สวัสดี"
encoded_utf8 = s.encode('utf-8')
encoded_ascii = "Hello".encode('ascii')

print(encoded_utf8)   # b'Hello \xe0\xb8\xaa\xe0\xb8\xa7\xe0\xb8\xb1\xe0\xb8\xaa\xe0\xb8\x94\xe0\xb8\xb5'
print(type(encoded_utf8))  # <class 'bytes'>

# decode() - แปลง bytes กลับเป็น str
decoded = encoded_utf8.decode('utf-8')
print(decoded)   # Hello สวัสดี

# ใช้กับ errors
try:
    "สวัสดี".encode('ascii')  # ไม่รองรับ Thai
except UnicodeEncodeError:
    encoded = "สวัสดี".encode('ascii', errors='ignore')  # ตัดทิ้ง
    print(encoded)   # b''
    
    encoded = "สวัสดี".encode('ascii', errors='replace')  # แทน ?
    print(encoded)   # b'??????'
```

### Translate Method

```python
# maketrans() + translate() - แปลง characters
translation = str.maketrans('aeiou', '12345')
s = "hello world"
print(s.translate(translation))  # h2ll4 w4rld

# ลบ characters
delete_table = str.maketrans('', '', 'aeiou')  # ลบสระ
print("Hello World".translate(delete_table))    # Hll Wrld

# ตัวอย่าง: แปลง ROT13
rot13 = str.maketrans(
    'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ',
    'nopqrstuvwxyzabcdefghijklmNOPQRSTUVWXYZABCDEFGHIJKLM'
)

message = "Hello World"
encoded = message.translate(rot13)
decoded = encoded.translate(rot13)
print(f"Original: {message}")
print(f"Encoded:  {encoded}")
print(f"Decoded:  {decoded}")
```

### Partition Methods

```python
# partition() - แบ่งเป็น 3 ส่วน (ก่อน, separator, หลัง)
s = "user@example.com"
before, sep, after = s.partition('@')
print(before)   # user
print(sep)      # @
print(after)    # example.com

# ถ้าไม่พบ separator
result = "hello".partition('@')
print(result)   # ('hello', '', '')

# rpartition() - แบ่งจากขวา
path = "/home/user/file.txt"
before, sep, after = path.rpartition('/')
print(before)   # /home/user
print(after)    # file.txt
```

---

## String Formatting

### วิธีที่ 1: % Formatting (เก่า)

```python
name = "Alice"
age = 25
score = 98.5

# %s สำหรับ string, %d สำหรับ int, %f สำหรับ float
print("ชื่อ: %s" % name)
print("อายุ: %d ปี" % age)
print("คะแนน: %.2f" % score)

# หลายค่า
print("ชื่อ: %s, อายุ: %d" % (name, age))

# ระบุความกว้าง
print("%10s" % name)    # "     Alice" (ชิดขวา)
print("%-10s" % name)   # "Alice     " (ชิดซ้าย)
print("%010d" % age)    # "0000000025" (เติม 0)
```

### วิธีที่ 2: .format() Method

```python
name = "Bob"
age = 30
price = 1234.567

# ตำแหน่ง
print("Hello {}!".format(name))
print("{} is {} years old".format(name, age))

# ระบุ index
print("{0} and {1}".format("first", "second"))
print("{1} and {0}".format("first", "second"))  # สลับ

# ระบุ name
print("{name} is {age} years old".format(name="Charlie", age=35))

# ระบุ format
print("{:.2f}".format(price))     # 1234.57
print("{:>10}".format(name))      # "       Bob" (ชิดขวา)
print("{:<10}".format(name))      # "Bob       " (ชิดซ้าย)
print("{:^10}".format(name))      # "   Bob    " (กลาง)
print("{:0>5}".format(42))        # "00042" (เติม 0 ชิดขวา)
print("{:+d}".format(42))         # "+42" (แสดง sign)

# แสดง binary, octal, hex
n = 255
print("{:b}".format(n))    # 11111111
print("{:o}".format(n))    # 377
print("{:x}".format(n))    # ff
print("{:X}".format(n))    # FF
print("{:#x}".format(n))   # 0xff (เติม prefix)

# format กับ dictionary
person = {"name": "David", "age": 28}
print("{name} is {age}".format(**person))
```

### วิธีที่ 3: f-strings (แนะนำที่สุด - Python 3.6+)

```python
name = "Eve"
age = 22
price = 9876.54321

# Basic f-string
print(f"Hello {name}!")
print(f"{name} is {age} years old")

# Expressions ใน f-string
print(f"Next year: {age + 1}")
print(f"Uppercase: {name.upper()}")
print(f"Pi: {22/7:.4f}")

# Format specifiers
print(f"{price:.2f}")          # 9876.54
print(f"{price:,.2f}")         # 9,876.54 (comma separator)
print(f"{age:04d}")            # 0022 (zero pad)
print(f"{name:>10}")           # "       Eve"
print(f"{name:<10}")           # "Eve       "
print(f"{name:^10}")           # "   Eve    "
print(f"{name:*^10}")          # "***Eve****"

# Multiline f-string
info = (
    f"Name: {name}\n"
    f"Age: {age}\n"
    f"Price: ${price:,.2f}"
)
print(info)

# Nested f-string
width = 10
print(f"{'hello':^{width}}")   # "  hello   "

# Dictionary ใน f-string
person = {"name": "Frank", "job": "Engineer"}
print(f"{person['name']} is an {person['job']}")

# Conditional expression
score = 85
grade = f"{'Pass' if score >= 50 else 'Fail'}"
print(f"Score: {score} - {grade}")

# f-string debug (Python 3.8+)
x = 42
print(f"{x = }")           # x = 42
print(f"{x*2 = }")         # x*2 = 84

# Multiline f-string
report = f"""
=== รายงาน ===
ชื่อ:    {name}
อายุ:    {age}
ราคา:   {price:,.2f}
=============
"""
print(report)
```

### Format Specification Mini-Language

```python
# รูปแบบ: {[field_name][!conversion][:format_spec]}
# format_spec: [[fill]align][sign][#][0][width][grouping_option][.precision][type]

# Type specifiers
n = 1234567.8910

print(f"{n:d}")      # ??? (ต้องเป็น int)
print(f"{int(n):d}") # 1234567
print(f"{n:f}")      # 1234567.891000
print(f"{n:e}")      # 1.234568e+06
print(f"{n:g}")      # 1.23457e+06 (ใช้ e ถ้าจำเป็น)
print(f"{n:n}")      # 1234567.891 (locale-aware)
print(f"{0.5:%}")    # 50.000000%
print(f"{0.5:.1%}")  # 50.0%

# Fill and align
print(f"{'hello':>10}")     # "     hello"
print(f"{'hello':<10}")     # "hello     "
print(f"{'hello':^10}")     # "  hello   "
print(f"{'hello':*^10}")    # "**hello***"
print(f"{'hello':0^10}")    # "00hello000"

# Width and precision
print(f"{3.14159:.2f}")     # 3.14
print(f"{3.14159:8.2f}")    # "    3.14"
print(f"{'hi':5}")          # "hi   "

# Sign
print(f"{42:+d}")    # +42
print(f"{-42:+d}")   # -42
print(f"{42: d}")    # " 42" (space for positive)
print(f"{-42: d}")   # "-42"
```

---

## Raw Strings และ Escape Characters

### Escape Characters

```python
# Escape sequences
print("Hello\nWorld")    # \n = newline
print("Hello\tWorld")    # \t = tab
print("Hello\\World")    # \\ = backslash
print("Hello\"World")    # \" = double quote
print("Hello\'World")    # \' = single quote
print("\a")               # \a = bell (alert)
print("\b")               # \b = backspace
print("\r")               # \r = carriage return
print("\f")               # \f = form feed
print("\v")               # \v = vertical tab

# Unicode escape
print("\u0041")           # A (Unicode codepoint)
print("\U0001F600")       # 😀 (4-byte Unicode)
print("\N{SNOWMAN}")      # ☃ (Unicode name)

# Hex escape
print("\x41")             # A (hex value)
print("\x48\x65\x6c\x6c\x6f")  # Hello

# Octal escape
print("\101")             # A (octal value)

# ตัวอย่าง: multiline text
address = "123 Main St\nAnytown, ST 12345\nUSA"
print(address)

# ตัวอย่าง: path Windows
path_escaped = "C:\\Users\\John\\Desktop\\file.txt"
print(path_escaped)
```

### Raw Strings

```python
# Raw string ไม่แปล escape sequences
path = r"C:\Users\John\Desktop\file.txt"
print(path)    # C:\Users\John\Desktop\file.txt

# ไม่มี \n \t
text = r"Hello\nWorld"
print(text)    # Hello\nWorld  (ไม่ขึ้นบรรทัดใหม่)

# Raw string มักใช้กับ Regular Expressions
import re
pattern = r"\d+\.\d+"   # หาตัวเลขทศนิยม
text = "Pi is approximately 3.14159"
match = re.search(pattern, text)
print(match.group())   # 3.14159

# ระวัง: raw string ไม่สามารถลงท้ายด้วย \
# r"path\"   # SyntaxError!
# ใช้ r"path\\" แทน (ลงท้ายด้วย double backslash)
```

---

## String Concatenation และ Repetition

```python
# Concatenation (+)
first = "Hello"
second = " World"
result = first + second
print(result)   # Hello World

# Repetition (*)
repeated = "Ha" * 3
print(repeated)  # HaHaHa

line = "-" * 40
print(line)      # ----------------------------------------

# Join หลาย strings
parts = ["Python", " is", " awesome"]
sentence = "".join(parts)
print(sentence)  # Python is awesome

# += (augmented concatenation)
s = "Hello"
s += " World"
print(s)   # Hello World

# การ Concatenate กับ non-string ต้องแปลงก่อน
age = 25
# print("อายุ: " + age)  # TypeError!
print("อายุ: " + str(age))  # ถูกต้อง
print(f"อายุ: {age}")       # ดีกว่า (f-string)

# Concatenation ใน loop (ไม่มีประสิทธิภาพ)
bad_way = ""
for i in range(1000):
    bad_way += str(i)  # สร้าง string ใหม่ทุก iteration!

# วิธีที่ดีกว่า
good_way = "".join(str(i) for i in range(1000))

# หรือ
parts = [str(i) for i in range(1000)]
good_way2 = "".join(parts)
```

---

## Multiline Strings

```python
# Triple quotes สำหรับ multiline
poem = """
กาลครั้งหนึ่งนานมาแล้ว
มีโปรแกรมเมอร์คนหนึ่ง
เขาเรียน Python ทุกวัน
จนเป็นเซียน
"""
print(poem)

# Implicit string concatenation
long_string = (
    "This is the first part. "
    "This is the second part. "
    "This is the third part."
)
print(long_string)

# Explicit line continuation
long_string2 = "This is the first part. " \
               "This is the second part. " \
               "This is the third part."
print(long_string2)

# Multiline ใน HTML template
html = """
<!DOCTYPE html>
<html>
  <head>
    <title>{title}</title>
  </head>
  <body>
    <h1>{heading}</h1>
    <p>{content}</p>
  </body>
</html>
""".format(
    title="My Page",
    heading="Welcome",
    content="This is my webpage."
)
print(html)

# ลบ leading whitespace ด้วย textwrap.dedent
import textwrap

def show_help():
    return textwrap.dedent("""
        Usage: program [options] <file>
        
        Options:
          -h, --help    Show this help message
          -v, --verbose Verbose output
          -o <file>     Output file
    """).strip()

print(show_help())
```

---

## String Comparison

```python
# เปรียบเทียบด้วย == และ !=
s1 = "Hello"
s2 = "Hello"
s3 = "hello"
s4 = "World"

print(s1 == s2)   # True (ค่าเหมือนกัน)
print(s1 == s3)   # False (case ต่างกัน)
print(s1 != s4)   # True

# เปรียบเทียบลำดับ (lexicographic)
print("apple" < "banana")   # True  ('a' < 'b')
print("apple" > "Apple")    # True  ('a' > 'A' เพราะ ASCII)
print("Z" < "a")            # True  ('Z'=90 < 'a'=97)

# ค่า ASCII ของอักขระ
print(ord('A'))   # 65
print(ord('a'))   # 97
print(ord('Z'))   # 90
print(ord('z'))   # 122
print(chr(65))    # A
print(chr(97))    # a

# Case-insensitive comparison
s1 = "Python"
s2 = "PYTHON"
print(s1.lower() == s2.lower())     # True
print(s1.casefold() == s2.casefold())  # True (ดีกว่าสำหรับ Unicode)

# เรียงลำดับ
words = ["banana", "Apple", "cherry", "Date"]
print(sorted(words))                          # ['Apple', 'Date', 'banana', 'cherry']
print(sorted(words, key=str.lower))           # ['Apple', 'banana', 'cherry', 'Date']
print(sorted(words, key=str.casefold))        # ['Apple', 'banana', 'cherry', 'Date']

# String interning
a = "hello"
b = "hello"
c = "hel" + "lo"
d = input("พิมพ์ 'hello': ")  # "hello"

print(a is b)  # True (Python intern short strings)
print(a is c)  # True หรือ False (ขึ้นกับ implementation)
print(a == d)  # True (ค่าเหมือนกัน)
print(a is d)  # False (ไม่ intern runtime strings)
```

---

## Regular String Operations

### ตรวจสอบ substring

```python
# 'in' operator
s = "Python is awesome"

print("Python" in s)     # True
print("Java" in s)       # False
print("Python" not in s) # False
print("is" in s)         # True

# ใช้กับ conditional
if "Python" in s:
    print("พบ Python!")
```

### นับ และค้นหา

```python
s = "abcabcabc"

# นับ occurrence
print(s.count("abc"))    # 3
print(s.count("ab"))     # 3
print(s.count("a"))      # 3

# หา all occurrences (ไม่มี built-in method, ทำเอง)
def find_all(s, sub):
    positions = []
    start = 0
    while True:
        pos = s.find(sub, start)
        if pos == -1:
            break
        positions.append(pos)
        start = pos + 1
    return positions

print(find_all("abcabcabc", "abc"))  # [0, 3, 6]

# หาด้วย regex
import re
positions = [m.start() for m in re.finditer("abc", "abcabcabc")]
print(positions)   # [0, 3, 6]
```

### String Operations เบ็ดเตล็ด

```python
# len() - ความยาว
print(len("Hello"))           # 5
print(len("สวัสดี"))          # 6
print(len(""))                # 0

# min() max() - character ที่เล็ก/ใหญ่สุด
print(min("Python"))   # P (ค่า ASCII ต่ำสุด)
print(max("Python"))   # y (ค่า ASCII สูงสุด)

# sorted() - เรียง characters
print(sorted("Python"))         # ['P', 'h', 'n', 'o', 't', 'y']
print("".join(sorted("Python"))) # Phnoty

# reversed() - กลับ string
print("".join(reversed("Python")))   # nohtyP
print("Python"[::-1])                # nohtyP (เร็วกว่า)

# กลับ words ใน sentence
sentence = "Hello World Python"
words = sentence.split()
reversed_sentence = " ".join(reversed(words))
print(reversed_sentence)  # Python World Hello

# String multiplication
print("=" * 30)         # ==============================
print("Ha" * 3)         # HaHaHa
print("-" * 10 + "+" + "-" * 10)   # ----------+----------
```

---

## ตัวอย่างโค้ด

### ตัวอย่างที่ 1: String Inspector

```python
# ตัวอย่างที่ 1: ตรวจสอบ String properties
def inspect_string(s):
    """แสดงข้อมูลทั้งหมดของ string"""
    print(f"\nString: {repr(s)}")
    print(f"  Length:    {len(s)}")
    print(f"  Upper:     {s.upper()}")
    print(f"  Lower:     {s.lower()}")
    print(f"  Title:     {s.title()}")
    print(f"  Reversed:  {s[::-1]}")
    print(f"  isalpha:   {s.isalpha()}")
    print(f"  isdigit:   {s.isdigit()}")
    print(f"  isalnum:   {s.isalnum()}")
    print(f"  isspace:   {s.isspace()}")
    print(f"  istitle:   {s.istitle()}")
    
    # Unicode info
    print(f"  Characters:")
    for i, char in enumerate(s[:5]):  # แสดงแค่ 5 ตัวแรก
        print(f"    [{i}] {repr(char)} = U+{ord(char):04X}")

inspect_string("Hello World")
inspect_string("Python123")
inspect_string("สวัสดี")
```

### ตัวอย่างที่ 2: Text Processor

```python
# ตัวอย่างที่ 2: Text Processor
def clean_text(text):
    """ทำความสะอาด text"""
    # ลบช่องว่างหัวท้าย
    text = text.strip()
    # แปลง whitespace หลายตัวเป็นตัวเดียว
    import re
    text = re.sub(r'\s+', ' ', text)
    return text

def word_frequency(text):
    """นับความถี่ของคำ"""
    words = text.lower().split()
    # ลบ punctuation
    import string
    words = [w.strip(string.punctuation) for w in words]
    
    freq = {}
    for word in words:
        if word:
            freq[word] = freq.get(word, 0) + 1
    return dict(sorted(freq.items(), key=lambda x: -x[1]))

text = """Python is a programming language. Python is easy to learn.
          Python is widely used in data science and web development."""

cleaned = clean_text(text)
print("Cleaned:", cleaned[:60], "...")

freq = word_frequency(cleaned)
print("\nTop 5 words:")
for word, count in list(freq.items())[:5]:
    print(f"  {word}: {count}")
```

### ตัวอย่างที่ 3: Template Engine อย่างง่าย

```python
# ตัวอย่างที่ 3: Template Engine
def render_template(template, **variables):
    """แทนที่ {{variable}} ด้วยค่าจริง"""
    result = template
    for key, value in variables.items():
        placeholder = "{{" + key + "}}"
        result = result.replace(placeholder, str(value))
    return result

email_template = """
สวัสดีคุณ {{name}},

ขอขอบคุณที่สั่งซื้อสินค้า #{{order_id}}
ยอดรวม: {{total}} บาท

สินค้าจะถึงมือคุณภายใน {{days}} วันทำการ

ขอบคุณค่ะ
ทีมงาน MyShop
"""

email = render_template(
    email_template,
    name="สมชาย",
    order_id="ORD-2024-001",
    total="1,250",
    days="3-5"
)
print(email)
```

### ตัวอย่างที่ 4: String Validator

```python
# ตัวอย่างที่ 4: String Validator
import re

class StringValidator:
    """Validator สำหรับ string ต่างๆ"""
    
    @staticmethod
    def is_valid_email(email):
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, email))
    
    @staticmethod
    def is_valid_phone(phone):
        # เบอร์โทรไทย: 08x, 09x, 02x
        pattern = r'^0[0-9]{8,9}$'
        clean = phone.replace('-', '').replace(' ', '')
        return bool(re.match(pattern, clean))
    
    @staticmethod
    def is_strong_password(password):
        """ตรวจสอบ password ที่แข็งแรง"""
        checks = {
            "ยาวอย่างน้อย 8 ตัว": len(password) >= 8,
            "มีตัวพิมพ์ใหญ่": any(c.isupper() for c in password),
            "มีตัวพิมพ์เล็ก": any(c.islower() for c in password),
            "มีตัวเลข": any(c.isdigit() for c in password),
            "มีอักขระพิเศษ": any(c in "!@#$%^&*" for c in password),
        }
        return checks

# ทดสอบ
validator = StringValidator()

emails = ["user@example.com", "invalid.email", "test@.com"]
for email in emails:
    valid = validator.is_valid_email(email)
    print(f"Email {email}: {'✅' if valid else '❌'}")

phones = ["0812345678", "0987654321", "123456789"]
for phone in phones:
    valid = validator.is_valid_phone(phone)
    print(f"Phone {phone}: {'✅' if valid else '❌'}")

passwords = ["weak", "StrongPass1!", "NoSpecial1"]
for pwd in passwords:
    checks = validator.is_strong_password(pwd)
    all_pass = all(checks.values())
    print(f"\nPassword '{pwd}': {'✅ Strong' if all_pass else '❌ Weak'}")
    for check, passed in checks.items():
        print(f"  {'✅' if passed else '❌'} {check}")
```

### ตัวอย่างที่ 5: Caesar Cipher

```python
# ตัวอย่างที่ 5: Caesar Cipher
def caesar_encrypt(text, shift):
    """เข้ารหัส Caesar Cipher"""
    result = []
    for char in text:
        if char.isalpha():
            # หา base (A=65, a=97)
            base = ord('A') if char.isupper() else ord('a')
            # เลื่อน shift ตัว (mod 26)
            encrypted = chr((ord(char) - base + shift) % 26 + base)
            result.append(encrypted)
        else:
            result.append(char)
    return "".join(result)

def caesar_decrypt(text, shift):
    """ถอดรหัส Caesar Cipher"""
    return caesar_encrypt(text, -shift)

# ทดสอบ
message = "Hello World! Python is Great."
shift = 13  # ROT13

encrypted = caesar_encrypt(message, shift)
decrypted = caesar_decrypt(encrypted, shift)

print(f"Original:  {message}")
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")
```

### ตัวอย่างที่ 6: Text Alignment และ Tables

```python
# ตัวอย่างที่ 6: สร้างตารางสวยงาม
def print_table(headers, rows, col_widths=None):
    """พิมพ์ตารางสวยงาม"""
    if col_widths is None:
        col_widths = [max(len(str(row[i])) for row in [headers] + rows) 
                      for i in range(len(headers))]
    
    # สร้าง border
    border = "+" + "+".join("-" * (w + 2) for w in col_widths) + "+"
    
    def format_row(row):
        cells = [f" {str(cell):<{col_widths[i]}} " 
                 for i, cell in enumerate(row)]
        return "|" + "|".join(cells) + "|"
    
    print(border)
    print(format_row(headers))
    print(border)
    for row in rows:
        print(format_row(row))
    print(border)

# ทดสอบ
headers = ["ชื่อ", "อายุ", "เมือง", "งาน"]
rows = [
    ["สมชาย", 25, "กรุงเทพ", "Developer"],
    ["มานี", 30, "เชียงใหม่", "Designer"],
    ["วิชัย", 28, "ภูเก็ต", "Manager"],
]

print_table(headers, rows)
```

### ตัวอย่างที่ 7: URL Parser

```python
# ตัวอย่างที่ 7: Parse URL
def parse_url(url):
    """แยกส่วนประกอบของ URL"""
    result = {}
    
    # แยก protocol
    if "://" in url:
        protocol, rest = url.split("://", 1)
        result["protocol"] = protocol
    else:
        rest = url
        result["protocol"] = "http"
    
    # แยก fragment (#)
    if "#" in rest:
        rest, result["fragment"] = rest.split("#", 1)
    
    # แยก query string (?)
    if "?" in rest:
        rest, query = rest.split("?", 1)
        result["query"] = dict(
            param.split("=", 1) if "=" in param else (param, "")
            for param in query.split("&")
        )
    
    # แยก path
    if "/" in rest:
        host, path = rest.split("/", 1)
        result["path"] = "/" + path
    else:
        host = rest
        result["path"] = "/"
    
    # แยก port
    if ":" in host:
        host, port = host.rsplit(":", 1)
        result["port"] = int(port)
    
    result["host"] = host
    return result

# ทดสอบ
urls = [
    "https://www.example.com/path/to/page?q=python&page=1#section",
    "http://localhost:8080/api/users",
    "ftp://files.example.com/data.csv",
]

for url in urls:
    parsed = parse_url(url)
    print(f"\nURL: {url}")
    for key, value in parsed.items():
        print(f"  {key}: {value}")
```

### ตัวอย่างที่ 8: String Metrics

```python
# ตัวอย่างที่ 8: String Similarity Metrics

def levenshtein_distance(s1, s2):
    """คำนวณ Edit Distance ระหว่างสอง string"""
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    
    return dp[m][n]

def similarity_ratio(s1, s2):
    """คำนวณ similarity 0-1"""
    distance = levenshtein_distance(s1, s2)
    max_len = max(len(s1), len(s2))
    if max_len == 0:
        return 1.0
    return 1 - distance / max_len

# ทดสอบ
pairs = [
    ("Python", "Python"),
    ("Python", "python"),
    ("kitten", "sitting"),
    ("Sunday", "Saturday"),
    ("programming", "programing"),
]

print(f"{'String 1':<15} {'String 2':<15} {'Distance':>8} {'Similarity':>10}")
print("-" * 55)
for s1, s2 in pairs:
    dist = levenshtein_distance(s1, s2)
    sim = similarity_ratio(s1, s2)
    print(f"{s1:<15} {s2:<15} {dist:>8} {sim:>10.1%}")
```

### ตัวอย่างที่ 9: Text Statistics

```python
# ตัวอย่างที่ 9: Text Statistics
def text_statistics(text):
    """คำนวณสถิติของ text"""
    import re
    
    stats = {}
    
    # นับตัวอักษร
    stats['total_chars'] = len(text)
    stats['chars_no_space'] = len(text.replace(' ', ''))
    stats['spaces'] = text.count(' ')
    
    # นับคำ
    words = text.split()
    stats['word_count'] = len(words)
    stats['unique_words'] = len(set(w.lower() for w in words))
    
    # นับประโยค
    sentences = re.split(r'[.!?]+', text)
    stats['sentence_count'] = len([s for s in sentences if s.strip()])
    
    # คำนวณ averages
    if stats['sentence_count'] > 0:
        stats['avg_words_per_sentence'] = stats['word_count'] / stats['sentence_count']
    
    if stats['word_count'] > 0:
        stats['avg_word_length'] = sum(len(w) for w in words) / stats['word_count']
    
    # ตัวอักษร
    stats['uppercase'] = sum(1 for c in text if c.isupper())
    stats['lowercase'] = sum(1 for c in text if c.islower())
    stats['digits'] = sum(1 for c in text if c.isdigit())
    stats['punctuation'] = sum(1 for c in text if c in '.,!?;:')
    
    # คำที่ยาวที่สุด
    if words:
        stats['longest_word'] = max(words, key=len)
    
    return stats

sample_text = """
Python is a high-level, general-purpose programming language. 
Its design philosophy emphasizes code readability with the use of significant indentation. 
Python is dynamically typed and garbage-collected. 
It supports multiple programming paradigms, including structured, object-oriented and functional programming.
"""

stats = text_statistics(sample_text.strip())
print("=== Text Statistics ===")
for key, value in stats.items():
    if isinstance(value, float):
        print(f"  {key:30}: {value:.2f}")
    else:
        print(f"  {key:30}: {value}")
```

### ตัวอย่างที่ 10: String Pattern Matching

```python
# ตัวอย่างที่ 10: Pattern Matching
import re

def extract_data(text):
    """ดึงข้อมูลจาก text"""
    
    # หา email addresses
    emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', text)
    
    # หา phone numbers
    phones = re.findall(r'0[0-9]{8,9}', text)
    
    # หา URLs
    urls = re.findall(r'https?://[^\s]+', text)
    
    # หา dates
    dates = re.findall(r'\d{1,2}/\d{1,2}/\d{2,4}', text)
    
    # หา numbers
    numbers = re.findall(r'\b\d+\.?\d*\b', text)
    
    return {
        "emails": emails,
        "phones": phones,
        "urls": urls,
        "dates": dates,
        "numbers": numbers,
    }

sample = """
ติดต่อเราได้ที่ info@company.com หรือ support@example.org
โทร: 0812345678 หรือ 0987654321
เว็บไซต์: https://www.company.com และ http://example.org
วันที่สั่งซื้อ: 15/3/2024 หรือ 1/12/2024
ราคา 1500 บาท ส่วนลด 10.5%
"""

data = extract_data(sample)
for category, items in data.items():
    if items:
        print(f"\n{category}:")
        for item in items:
            print(f"  - {item}")
```

### ตัวอย่างที่ 11: String Compression

```python
# ตัวอย่างที่ 11: Run-Length Encoding
def rle_encode(s):
    """เข้ารหัสด้วย Run-Length Encoding"""
    if not s:
        return ""
    
    result = []
    count = 1
    
    for i in range(1, len(s)):
        if s[i] == s[i-1]:
            count += 1
        else:
            result.append(f"{s[i-1]}{count}" if count > 1 else s[i-1])
            count = 1
    
    result.append(f"{s[-1]}{count}" if count > 1 else s[-1])
    return "".join(result)

def rle_decode(s):
    """ถอดรหัส Run-Length Encoding"""
    result = []
    i = 0
    
    while i < len(s):
        char = s[i]
        i += 1
        count = ""
        
        while i < len(s) and s[i].isdigit():
            count += s[i]
            i += 1
        
        result.append(char * (int(count) if count else 1))
    
    return "".join(result)

# ทดสอบ
test_strings = [
    "AAABBBCCCC",
    "AABBC",
    "ABCDE",
    "AAAAAAAAAA",
]

print(f"{'Original':<20} {'Encoded':<20} {'Ratio':>8}")
print("-" * 50)
for s in test_strings:
    encoded = rle_encode(s)
    ratio = len(encoded) / len(s)
    print(f"{s:<20} {encoded:<20} {ratio:>7.1%}")
    assert rle_decode(encoded) == s, "Decode ผิด!"
```

### ตัวอย่างที่ 12: Multiline String Templates

```python
# ตัวอย่างที่ 12: เอกสาร Template
def create_invoice(customer, items, tax_rate=0.07):
    """สร้างใบแจ้งหนี้"""
    subtotal = sum(qty * price for _, qty, price in items)
    tax = subtotal * tax_rate
    total = subtotal + tax
    
    # สร้าง items rows
    rows = []
    for name, qty, price in items:
        subtotal_item = qty * price
        rows.append(f"  {name:<20} {qty:>3} x {price:>8,.2f} = {subtotal_item:>10,.2f}")
    
    invoice = f"""
╔══════════════════════════════════════════════════╗
║                   ใบแจ้งหนี้                      ║
╚══════════════════════════════════════════════════╝

ลูกค้า: {customer}
วันที่:  {__import__('datetime').date.today()}

รายการ:
{'─'*55}
{"รายการ":<22} {"จำนวน":>5} {"ราคา/หน่วย":>12} {"รวม":>12}
{'─'*55}
{chr(10).join(rows)}
{'─'*55}
{"ราคาสินค้า":>49} {subtotal:>10,.2f}
{f"ภาษี {tax_rate:.0%}":>49} {tax:>10,.2f}
{'─'*55}
{"ยอดรวมทั้งสิ้น":>49} {total:>10,.2f}
{'═'*55}

ขอบคุณที่ใช้บริการ
    """
    return invoice

# ทดสอบ
customer = "บริษัท ABC จำกัด"
items = [
    ("MacBook Pro 14\"", 2, 79900.00),
    ("Magic Mouse", 2, 2900.00),
    ("USB-C Hub", 3, 1500.00),
]

print(create_invoice(customer, items))
```

### ตัวอย่างที่ 13: String Chunking

```python
# ตัวอย่างที่ 13: แบ่ง String เป็น Chunks
def chunk_string(s, size):
    """แบ่ง string เป็นส่วนๆ ขนาด size"""
    return [s[i:i+size] for i in range(0, len(s), size)]

def wrap_text(text, width=60):
    """ตัดบรรทัดอัตโนมัติ"""
    words = text.split()
    lines = []
    current_line = []
    current_length = 0
    
    for word in words:
        if current_length + len(word) + (1 if current_line else 0) <= width:
            current_line.append(word)
            current_length += len(word) + (1 if len(current_line) > 1 else 0)
        else:
            if current_line:
                lines.append(" ".join(current_line))
            current_line = [word]
            current_length = len(word)
    
    if current_line:
        lines.append(" ".join(current_line))
    
    return "\n".join(lines)

# ทดสอบ chunk
text = "Python Programming"
chunks = chunk_string(text, 3)
print(f"Chunks of 3: {chunks}")

# ทดสอบ wrap
long_text = """Python is a high-level, general-purpose programming language that emphasizes code readability and simplicity. It was created by Guido van Rossum and first released in 1991."""

print("\nWrapped at 60 chars:")
print(wrap_text(long_text, 60))
```

---

## แบบฝึกหัด

### ข้อที่ 1: String Reversal และ Palindrome

เขียนฟังก์ชัน:
1. `reverse_words(sentence)` - กลับลำดับคำ เช่น "Hello World" → "World Hello"
2. `is_palindrome_sentence(sentence)` - ตรวจว่าประโยคเป็น palindrome (ไม่สน space, case, punctuation)

### ข้อที่ 2: String Statistics Dashboard

เขียน function `text_dashboard(text)` ที่แสดงสถิติ:
- จำนวนคำ, ตัวอักษร, ประโยค
- คำที่ปรากฏบ่อย 3 อันดับ
- ตัวอักษรที่ปรากฏบ่อย 5 อันดับ
- ประโยคที่ยาวที่สุด

### ข้อที่ 3: Password Strength Checker

เขียน function `check_password_strength(password)` ที่คืน dict:
- `score`: 0-100
- `strength`: "Weak" / "Fair" / "Strong" / "Very Strong"
- `suggestions`: list ของคำแนะนำ

### ข้อที่ 4: CSV Parser

เขียน function `parse_csv(text)` ที่รับ CSV text แล้วคืน list of dicts:
```
name,age,city
Alice,25,Bangkok
Bob,30,Chiang Mai
```
→ `[{"name": "Alice", "age": "25", "city": "Bangkok"}, ...]`

### ข้อที่ 5: Template Engine

สร้าง simple template engine ที่รองรับ:
- `{{variable}}` - แทนค่า variable
- `{{variable|upper}}` - แปลง filter (upper, lower, title)
- `{{variable|default:ค่าเริ่มต้น}}` - ค่า default

### ข้อที่ 6: Word Wrap

เขียน function `word_wrap(text, width, align='left')` ที่:
- ตัดบรรทัดที่ความกว้างที่กำหนด
- รองรับ alignment: 'left', 'right', 'center', 'justify'

### ข้อที่ 7: String Formatter

เขียน function `format_number(n, style)` ที่รองรับ:
- `style='currency'`: "1,234.56 บาท"
- `style='percentage'`: "85.50%"
- `style='scientific'`: "1.23e+06"
- `style='ordinal'`: "1st", "2nd", "3rd", "4th"

### ข้อที่ 8: Slug Generator

เขียน function `slugify(text)` ที่แปลง title เป็น URL-friendly slug:
- "Hello World" → "hello-world"
- "Python 3.12 Released!" → "python-312-released"
- "สวัสดีโลก Hello" → "hello" (ตัด non-ASCII ออก)

### ข้อที่ 9: Multi-language Greeting

เขียนโปรแกรมที่รับชื่อและรหัสภาษา แล้วทักทายในภาษานั้น:
- `en`: "Hello, {name}!"
- `th`: "สวัสดี, {name}!"
- `ja`: "こんにちは、{name}！"
- `fr`: "Bonjour, {name}!"
- ถ้าไม่รู้จักภาษา ใช้ English

### ข้อที่ 10: Log Parser

เขียน function `parse_log(log_text)` ที่แยก log entries:
```
[2024-01-15 10:30:45] ERROR: Connection refused
[2024-01-15 10:31:00] INFO: Server started
[2024-01-15 10:31:05] WARNING: High memory usage
```
คืน list of dicts ที่มี datetime, level, message

---

## เฉลยแบบฝึกหัด

### เฉลยข้อที่ 1: String Reversal

```python
# เฉลยข้อที่ 1
import re

def reverse_words(sentence):
    """กลับลำดับคำ"""
    words = sentence.split()
    return " ".join(reversed(words))

def is_palindrome_sentence(sentence):
    """ตรวจ palindrome ไม่สน space, case, punctuation"""
    # ลบ non-alphanumeric
    clean = re.sub(r'[^a-zA-Z0-9]', '', sentence).lower()
    return clean == clean[::-1]

# ทดสอบ
print(reverse_words("Hello World Python"))
# Python World Hello

test_sentences = [
    "A man a plan a canal Panama",
    "race a car",
    "Was it a car or a cat I saw",
    "Never odd or even",
]

for s in test_sentences:
    result = is_palindrome_sentence(s)
    print(f"'{s[:30]}': {'✅ Palindrome' if result else '❌ Not palindrome'}")
```

### เฉลยข้อที่ 2: Text Statistics Dashboard

```python
# เฉลยข้อที่ 2
import re
from collections import Counter

def text_dashboard(text):
    """แสดง text statistics dashboard"""
    
    # นับคำ
    words = re.findall(r'\b[a-zA-Z]+\b', text.lower())
    
    # นับตัวอักษร
    chars = [c for c in text if c.isalpha()]
    
    # นับประโยค
    sentences = [s.strip() for s in re.split(r'[.!?]+', text) if s.strip()]
    
    # Top words และ chars
    word_freq = Counter(words).most_common(3)
    char_freq = Counter(chars).most_common(5)
    
    # ประโยคที่ยาวสุด
    longest = max(sentences, key=len) if sentences else ""
    
    print("=" * 60)
    print("TEXT STATISTICS DASHBOARD")
    print("=" * 60)
    print(f"คำทั้งหมด:        {len(words)}")
    print(f"ตัวอักษร:          {len(chars)}")
    print(f"ประโยค:           {len(sentences)}")
    print(f"\nTop 3 คำ:")
    for word, count in word_freq:
        print(f"  '{word}': {count} ครั้ง")
    print(f"\nTop 5 ตัวอักษร:")
    for char, count in char_freq:
        print(f"  '{char}': {count} ครั้ง")
    print(f"\nประโยคที่ยาวสุด:")
    print(f"  {longest[:80]}...")

text_dashboard("""
Python is a versatile programming language. Python is used for web development,
data science, and automation. Many developers love Python because Python is easy to learn.
""")
```

### เฉลยข้อที่ 4: CSV Parser

```python
# เฉลยข้อที่ 4
def parse_csv(text):
    """แยก CSV text เป็น list of dicts"""
    lines = text.strip().splitlines()
    if not lines:
        return []
    
    headers = [h.strip() for h in lines[0].split(',')]
    result = []
    
    for line in lines[1:]:
        if line.strip():
            values = [v.strip() for v in line.split(',')]
            row = dict(zip(headers, values))
            result.append(row)
    
    return result

# ทดสอบ
csv_text = """name,age,city,job
Alice,25,Bangkok,Developer
Bob,30,Chiang Mai,Designer
Charlie,35,Phuket,Manager"""

data = parse_csv(csv_text)
for person in data:
    print(person)
```

### เฉลยข้อที่ 8: Slug Generator

```python
# เฉลยข้อที่ 8
import re
import unicodedata

def slugify(text):
    """แปลง text เป็น URL-friendly slug"""
    # Normalize unicode (เช่น é → e)
    text = unicodedata.normalize('NFD', text)
    text = ''.join(c for c in text if unicodedata.category(c) != 'Mn')
    
    # แปลงเป็น lowercase
    text = text.lower()
    
    # เก็บเฉพาะ alphanumeric และ whitespace
    text = re.sub(r'[^a-z0-9\s-]', '', text)
    
    # แทน whitespace ด้วย -
    text = re.sub(r'[\s-]+', '-', text).strip('-')
    
    return text

# ทดสอบ
tests = [
    "Hello World",
    "Python 3.12 Released!",
    "  Spaces   Around  ",
    "Café au lait",
    "สวัสดีโลก Hello World",
]

for text in tests:
    print(f"'{text}' → '{slugify(text)}'")
```

### เฉลยข้อที่ 10: Log Parser

```python
# เฉลยข้อที่ 10
import re
from datetime import datetime

def parse_log(log_text):
    """แยก log entries"""
    pattern = r'\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] (\w+): (.+)'
    entries = []
    
    for line in log_text.strip().splitlines():
        match = re.match(pattern, line.strip())
        if match:
            dt_str, level, message = match.groups()
            entries.append({
                "datetime": datetime.strptime(dt_str, "%Y-%m-%d %H:%M:%S"),
                "level": level,
                "message": message.strip()
            })
    
    return entries

# ทดสอบ
log = """
[2024-01-15 10:30:45] ERROR: Connection refused
[2024-01-15 10:31:00] INFO: Server started on port 8080
[2024-01-15 10:31:05] WARNING: High memory usage: 85%
[2024-01-15 10:31:10] INFO: User alice logged in
[2024-01-15 10:31:15] ERROR: Database connection timeout
"""

entries = parse_log(log)
for entry in entries:
    print(f"[{entry['datetime']}] {entry['level']:7} | {entry['message']}")

# สรุปตาม level
from collections import Counter
level_counts = Counter(e['level'] for e in entries)
print("\nSummary:")
for level, count in sorted(level_counts.items()):
    print(f"  {level}: {count}")
```

---

## สรุป

ใน Part 03 นี้เราได้เรียนรู้:

1. **การสร้าง String** - single/double/triple quotes, raw strings, f-strings
2. **Indexing** - positive/negative indexing
3. **Slicing** - `[start:stop:step]`, negative slicing
4. **String Methods** - case, strip, search, check, replace, split/join, padding
5. **String Formatting** - % format, .format(), f-strings (แนะนำ)
6. **Escape Characters** - `\n`, `\t`, `\\`, `\u`, raw strings
7. **Concatenation** - `+`, `*`, join()
8. **Comparison** - `==`, `<`, `>`, case-insensitive comparison
9. **Regular Operations** - `in`, `len()`, `min()`, `max()`, `sorted()`

## ขั้นตอนต่อไป

- **Part 04**: Numbers, Math & Type Conversion
- ฝึกใช้ f-strings แทน % format และ .format()
- ลอง string methods ทุกตัว
- ทำแบบฝึกหัดเกี่ยวกับ text processing

---

*หมายเหตุ: ทุก code block ทดสอบแล้วบน Python 3.12*
