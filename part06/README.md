# Part 06 - Control Flow: if/elif/else

## สารบัญ

1. [if Statement พื้นฐาน](#1-if-statement-พื้นฐาน)
2. [if/else Statement](#2-ifelse-statement)
3. [if/elif/else Chains](#3-ifelifelse-chains)
4. [Nested if Statements](#4-nested-if-statements)
5. [Ternary Operator (Conditional Expression)](#5-ternary-operator-conditional-expression)
6. [match/case Statement (Python 3.10+)](#6-matchcase-statement-python-310)
7. [Guard Clauses และ Early Returns](#7-guard-clauses-และ-early-returns)
8. [Complex Conditions](#8-complex-conditions)
9. [Best Practices สำหรับ Conditionals](#9-best-practices-สำหรับ-conditionals)
10. [ตัวอย่างโปรแกรมจริง](#10-ตัวอย่างโปรแกรมจริง)
11. [แบบฝึกหัด](#11-แบบฝึกหัด)

---

## 1. if Statement พื้นฐาน

### ความหมายและการทำงาน

`if` statement คือโครงสร้างการควบคุมการทำงาน (control flow) พื้นฐานที่สุดใน Python ใช้เพื่อตรวจสอบเงื่อนไข และดำเนินการเฉพาะเมื่อเงื่อนไขเป็น `True`

### Syntax พื้นฐาน

```python
if condition:
    # โค้ดที่จะทำงานเมื่อ condition เป็น True
    statement
```

**หมายเหตุสำคัญ:**
- Python ใช้ **indentation (การเยื้องบรรทัด)** เพื่อกำหนดขอบเขตของ block
- มาตรฐานคือ **4 spaces** ต่อ 1 level
- เครื่องหมาย `:` ต้องต่อท้าย condition เสมอ

### ตัวอย่างที่ 1: if พื้นฐาน

```python
# ตัวอย่างที่ 1: ตรวจสอบตัวเลขบวก
number = 10

if number > 0:
    print("ตัวเลขนี้เป็นบวก")
    print(f"ค่าของตัวเลขคือ: {number}")

print("จบโปรแกรม")  # บรรทัดนี้ทำงานเสมอ
```

**Output:**
```
ตัวเลขนี้เป็นบวก
ค่าของตัวเลขคือ: 10
จบโปรแกรม
```

### ตัวอย่างที่ 2: ค่า Truthy และ Falsy

Python มีแนวคิดเรื่อง **Truthy** และ **Falsy** ซึ่งหมายถึงค่าที่ถูกแปลงเป็น `True` หรือ `False` โดยอัตโนมัติ

```python
# ค่าที่เป็น Falsy (ถูกแปลงเป็น False)
falsy_values = [0, 0.0, "", [], {}, (), None, False]

print("=== ค่า Falsy ===")
for value in falsy_values:
    if value:
        print(f"{repr(value)} เป็น Truthy")
    else:
        print(f"{repr(value)} เป็น Falsy")

print("\n=== ค่า Truthy ===")
truthy_values = [1, -1, "hello", [1, 2], {"a": 1}, True]
for value in truthy_values:
    if value:
        print(f"{repr(value)} เป็น Truthy")
```

**Output:**
```
=== ค่า Falsy ===
0 เป็น Falsy
0.0 เป็น Falsy
'' เป็น Falsy
[] เป็น Falsy
{} เป็น Falsy
() เป็น Falsy
None เป็น Falsy
False เป็น Falsy

=== ค่า Truthy ===
1 เป็น Truthy
-1 เป็น Truthy
'hello' เป็น Truthy
[1, 2] เป็น Truthy
{'a': 1} เป็น Truthy
True เป็น Truthy
```

### ตัวอย่างที่ 3: การใช้ Comparison Operators

```python
# Comparison Operators ทั้งหมด
x = 15
y = 10

print(f"x = {x}, y = {y}")
print(f"x > y: {x > y}")      # Greater than
print(f"x < y: {x < y}")      # Less than
print(f"x >= y: {x >= y}")    # Greater than or equal
print(f"x <= y: {x <= y}")    # Less than or equal
print(f"x == y: {x == y}")    # Equal
print(f"x != y: {x != y}")    # Not equal

# ตัวอย่างการใช้งาน
age = 18
if age >= 18:
    print("\nคุณบรรลุนิติภาวะแล้ว")

name = "Alice"
if name == "Alice":
    print("สวัสดี Alice!")

password = "secret123"
if password != "":
    print("มีการใส่รหัสผ่าน")
```

---

## 2. if/else Statement

### การทำงาน

`if/else` ใช้เมื่อต้องการกำหนดการทำงาน 2 ทาง: ทำงานอะไรเมื่อเงื่อนไขเป็น True และทำงานอะไรเมื่อเงื่อนไขเป็น False

```python
if condition:
    # ทำงานเมื่อ condition เป็น True
    true_block
else:
    # ทำงานเมื่อ condition เป็น False
    false_block
```

### ตัวอย่างที่ 4: if/else พื้นฐาน

```python
# ตรวจสอบเลขคู่-คี่
number = 7

if number % 2 == 0:
    print(f"{number} เป็นเลขคู่")
else:
    print(f"{number} เป็นเลขคี่")
```

### ตัวอย่างที่ 5: ตรวจสอบอายุ

```python
# ระบบตรวจสอบอายุ
def check_age(age):
    if age >= 18:
        print(f"อายุ {age} ปี: ผ่านการตรวจสอบ สามารถเข้าได้")
        return True
    else:
        years_left = 18 - age
        print(f"อายุ {age} ปี: อายุไม่ถึง ต้องรออีก {years_left} ปี")
        return False

check_age(20)
check_age(15)
check_age(18)
```

**Output:**
```
อายุ 20 ปี: ผ่านการตรวจสอบ สามารถเข้าได้
อายุ 15 ปี: อายุไม่ถึง ต้องรออีก 3 ปี
อายุ 18 ปี: ผ่านการตรวจสอบ สามารถเข้าได้
```

### ตัวอย่างที่ 6: ตรวจสอบรหัสผ่าน

```python
# ระบบตรวจสอบรหัสผ่าน
stored_password = "python2024"
input_password = "python2024"

if input_password == stored_password:
    print("รหัสผ่านถูกต้อง! เข้าสู่ระบบสำเร็จ")
    print("ยินดีต้อนรับสู่ระบบ")
else:
    print("รหัสผ่านไม่ถูกต้อง!")
    print("กรุณาลองใหม่อีกครั้ง")
```

### ตัวอย่างที่ 7: การใช้ in operator

```python
# ตรวจสอบว่าสิ่งใดอยู่ใน collection
fruits = ["apple", "banana", "cherry", "date"]
search_item = "banana"

if search_item in fruits:
    print(f"พบ '{search_item}' ในรายการผลไม้")
    index = fruits.index(search_item)
    print(f"อยู่ที่ตำแหน่งที่ {index + 1}")
else:
    print(f"ไม่พบ '{search_item}' ในรายการผลไม้")

# ตรวจสอบ substring
email = "user@example.com"
if "@" in email:
    print(f"\n'{email}' เป็น email ที่ถูกต้อง")
else:
    print(f"\n'{email}' ไม่ใช่ email ที่ถูกต้อง")
```

---

## 3. if/elif/else Chains

### ความหมาย

`elif` (ย่อจาก "else if") ใช้เมื่อต้องการตรวจสอบหลายเงื่อนไขแบบต่อเนื่อง Python จะตรวจสอบแต่ละเงื่อนไขตามลำดับ และหยุดที่เงื่อนไขแรกที่เป็น True

```python
if condition1:
    block1
elif condition2:
    block2
elif condition3:
    block3
else:
    default_block
```

### ตัวอย่างที่ 8: Grade System

```python
# ระบบให้เกรด
def get_grade(score):
    if score >= 90:
        return "A", "ดีเยี่ยม"
    elif score >= 80:
        return "B", "ดี"
    elif score >= 70:
        return "C", "พอใช้"
    elif score >= 60:
        return "D", "ผ่าน"
    else:
        return "F", "ไม่ผ่าน"

# ทดสอบ
scores = [95, 85, 75, 65, 55, 100, 0]
print("คะแนน  เกรด  ผล")
print("-" * 25)
for score in scores:
    grade, result = get_grade(score)
    print(f"  {score:3d}    {grade}   {result}")
```

**Output:**
```
คะแนน  เกรด  ผล
-------------------------
   95    A   ดีเยี่ยม
   85    B   ดี
   75    C   พอใช้
   65    D   ผ่าน
   55    F   ไม่ผ่าน
  100    A   ดีเยี่ยม
    0    F   ไม่ผ่าน
```

### ตัวอย่างที่ 9: ฤดูกาล

```python
# ระบุฤดูกาล
def get_season(month):
    if month in [12, 1, 2]:
        return "ฤดูหนาว (Winter)"
    elif month in [3, 4, 5]:
        return "ฤดูใบไม้ผลิ (Spring)"
    elif month in [6, 7, 8]:
        return "ฤดูร้อน (Summer)"
    elif month in [9, 10, 11]:
        return "ฤดูใบไม้ร่วง (Autumn/Fall)"
    else:
        return "เดือนไม่ถูกต้อง"

for month in range(1, 13):
    print(f"เดือน {month:2d}: {get_season(month)}")
```

### ตัวอย่างที่ 10: BMI Calculator

```python
# คำนวณและวิเคราะห์ค่า BMI
def calculate_bmi(weight_kg, height_m):
    bmi = weight_kg / (height_m ** 2)
    
    if bmi < 18.5:
        category = "น้ำหนักน้อยเกินไป (Underweight)"
        advice = "ควรรับประทานอาหารให้มากขึ้น"
    elif bmi < 25:
        category = "น้ำหนักปกติ (Normal weight)"
        advice = "รักษาน้ำหนักนี้ต่อไป"
    elif bmi < 30:
        category = "น้ำหนักเกิน (Overweight)"
        advice = "ควรออกกำลังกายและควบคุมอาหาร"
    else:
        category = "โรคอ้วน (Obese)"
        advice = "ควรปรึกษาแพทย์"
    
    return bmi, category, advice

# ทดสอบ
test_cases = [
    (50, 1.70),   # Underweight
    (65, 1.70),   # Normal
    (80, 1.70),   # Overweight
    (100, 1.70),  # Obese
]

for weight, height in test_cases:
    bmi, category, advice = calculate_bmi(weight, height)
    print(f"\nน้ำหนัก: {weight}kg, ส่วนสูง: {height}m")
    print(f"BMI: {bmi:.2f}")
    print(f"หมวดหมู่: {category}")
    print(f"คำแนะนำ: {advice}")
```

---

## 4. Nested if Statements

### ความหมาย

Nested if คือ if ที่อยู่ภายใน if อีกอัน ใช้เมื่อต้องการตรวจสอบเงื่อนไขหลายระดับ

### ตัวอย่างที่ 11: การตรวจสอบสิทธิ์การเข้าถึง

```python
# ระบบตรวจสอบสิทธิ์ผู้ใช้
def check_access(username, password, is_admin):
    if username == "admin" or username == "user":
        if password == "correct_password":
            if is_admin:
                print(f"ยินดีต้อนรับ {username}! คุณมีสิทธิ์ Admin")
                print("สามารถเข้าถึงทุก features ได้")
            else:
                print(f"ยินดีต้อนรับ {username}!")
                print("คุณมีสิทธิ์ผู้ใช้ทั่วไป")
        else:
            print("รหัสผ่านไม่ถูกต้อง!")
    else:
        print(f"ไม่พบผู้ใช้ '{username}' ในระบบ")

# ทดสอบ
check_access("admin", "correct_password", True)
print()
check_access("user", "correct_password", False)
print()
check_access("admin", "wrong_password", True)
print()
check_access("unknown", "correct_password", False)
```

### ตัวอย่างที่ 12: ตรวจสอบตัวเลข 3 ระดับ

```python
# การจำแนกตัวเลข
def classify_number(num):
    print(f"\nวิเคราะห์ตัวเลข: {num}")
    
    if num == 0:
        print("เป็นศูนย์ (Zero)")
    else:
        if num > 0:
            print("เป็นจำนวนบวก (Positive)")
            if num % 2 == 0:
                print("และเป็นเลขคู่ (Even)")
                if num > 100:
                    print("และมีค่ามากกว่า 100")
                else:
                    print("และมีค่าไม่เกิน 100")
            else:
                print("และเป็นเลขคี่ (Odd)")
        else:
            print("เป็นจำนวนลบ (Negative)")
            if num < -100:
                print("และมีค่าน้อยกว่า -100")

classify_number(0)
classify_number(4)
classify_number(150)
classify_number(7)
classify_number(-50)
classify_number(-200)
```

### คำเตือน: หลีกเลี่ยง Nested if ที่ซับซ้อนเกินไป

```python
# ❌ ไม่แนะนำ: nested ลึกเกินไป
def bad_example(a, b, c, d):
    if a > 0:
        if b > 0:
            if c > 0:
                if d > 0:
                    return "ทั้งหมดบวก"
                else:
                    return "d ไม่บวก"
            else:
                return "c ไม่บวก"
        else:
            return "b ไม่บวก"
    else:
        return "a ไม่บวก"

# ✅ แนะนำ: ใช้ and แทน
def good_example(a, b, c, d):
    if a > 0 and b > 0 and c > 0 and d > 0:
        return "ทั้งหมดบวก"
    elif a <= 0:
        return "a ไม่บวก"
    elif b <= 0:
        return "b ไม่บวก"
    elif c <= 0:
        return "c ไม่บวก"
    else:
        return "d ไม่บวก"

print(good_example(1, 2, 3, 4))
print(good_example(-1, 2, 3, 4))
```

---

## 5. Ternary Operator (Conditional Expression)

### ความหมาย

Ternary operator หรือ conditional expression คือการเขียน if/else ในบรรทัดเดียว ทำให้โค้ดกระชับขึ้น

### Syntax

```python
value_if_true if condition else value_if_false
```

### ตัวอย่างที่ 13: Ternary พื้นฐาน

```python
# แบบปกติ
age = 20
if age >= 18:
    status = "ผู้ใหญ่"
else:
    status = "เยาวชน"
print(f"สถานะ: {status}")

# แบบ Ternary
status = "ผู้ใหญ่" if age >= 18 else "เยาวชน"
print(f"สถานะ (ternary): {status}")

# ตัวอย่างอื่น
x = 10
y = 20
max_val = x if x > y else y
print(f"\nค่ามากกว่าระหว่าง {x} กับ {y}: {max_val}")

# ใช้ใน f-string
score = 75
result = f"คะแนน {score}: {'ผ่าน' if score >= 60 else 'ไม่ผ่าน'}"
print(result)
```

### ตัวอย่างที่ 14: Ternary ใน List

```python
# สร้างรายการพร้อม condition
numbers = [1, -2, 3, -4, 5, -6, 7, -8, 9, -10]

# แปลงตัวเลขลบให้เป็น 0
normalized = [n if n > 0 else 0 for n in numbers]
print(f"Original: {numbers}")
print(f"Normalized: {normalized}")

# หาค่า absolute value
abs_values = [n if n >= 0 else -n for n in numbers]
print(f"Absolute: {abs_values}")

# จัดประเภท
categories = ["บวก" if n > 0 else "ลบ" if n < 0 else "ศูนย์" for n in numbers]
print(f"Categories: {categories}")
```

### ตัวอย่างที่ 15: Nested Ternary (ใช้อย่างระมัดระวัง)

```python
# Nested ternary - ใช้ได้แต่ควรระวัง
score = 85

# แบบ nested ternary
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "D" if score >= 60 else "F"
print(f"คะแนน {score}: เกรด {grade}")

# แบบที่อ่านง่ายกว่า (แนะนำ)
def get_grade_readable(score):
    if score >= 90: return "A"
    if score >= 80: return "B"
    if score >= 70: return "C"
    if score >= 60: return "D"
    return "F"

print(f"คะแนน {score}: เกรด {get_grade_readable(score)}")
```

---

## 6. match/case Statement (Python 3.10+)

### ความหมาย

`match/case` เป็น feature ใหม่ที่เพิ่มใน Python 3.10 คล้ายกับ switch/case ในภาษาอื่น แต่มีความสามารถมากกว่า เรียกว่า **Structural Pattern Matching**

### Syntax พื้นฐาน

```python
match subject:
    case pattern1:
        block1
    case pattern2:
        block2
    case _:  # default case (wildcard)
        default_block
```

### ตัวอย่างที่ 16: match/case พื้นฐาน

```python
# ตรวจสอบคำสั่ง
def process_command(command):
    match command:
        case "start":
            return "เริ่มต้นโปรแกรม..."
        case "stop":
            return "หยุดโปรแกรม..."
        case "pause":
            return "พักโปรแกรม..."
        case "help":
            return "แสดงวิธีใช้งาน..."
        case _:
            return f"ไม่รู้จักคำสั่ง: '{command}'"

# ทดสอบ
commands = ["start", "stop", "pause", "help", "quit", "exit"]
for cmd in commands:
    print(f"'{cmd}': {process_command(cmd)}")
```

### ตัวอย่างที่ 17: match กับหลาย patterns (OR pattern)

```python
# ใช้ | สำหรับหลาย patterns
def classify_day(day):
    match day.lower():
        case "monday" | "tuesday" | "wednesday" | "thursday" | "friday":
            return "วันทำงาน (Weekday)"
        case "saturday" | "sunday":
            return "วันหยุดสุดสัปดาห์ (Weekend)"
        case _:
            return "ไม่ใช่วันที่ถูกต้อง"

days = ["Monday", "Saturday", "Wednesday", "Sunday", "Holiday"]
for day in days:
    print(f"{day}: {classify_day(day)}")
```

### ตัวอย่างที่ 18: match กับ Data Structures

```python
# match กับ tuple
def describe_point(point):
    match point:
        case (0, 0):
            return "จุดกำเนิด (Origin)"
        case (x, 0):
            return f"อยู่บนแกน X ที่ {x}"
        case (0, y):
            return f"อยู่บนแกน Y ที่ {y}"
        case (x, y):
            return f"จุด ({x}, {y})"

points = [(0, 0), (3, 0), (0, 4), (3, 4)]
for point in points:
    print(f"{point}: {describe_point(point)}")

print()

# match กับ list/sequence
def process_list(items):
    match items:
        case []:
            return "รายการว่าง"
        case [single]:
            return f"มีสมาชิกหนึ่งคน: {single}"
        case [first, second]:
            return f"มีสมาชิกสองคน: {first}, {second}"
        case [first, *rest]:
            return f"สมาชิกคนแรก: {first}, ที่เหลืออีก {len(rest)} คน"

lists = [[], [1], [1, 2], [1, 2, 3, 4, 5]]
for lst in lists:
    print(f"{lst}: {process_list(lst)}")
```

### ตัวอย่างที่ 19: match กับ Guard (if condition)

```python
# match พร้อม guard clause
def categorize_number(n):
    match n:
        case n if n < 0:
            return f"{n} เป็นจำนวนลบ"
        case 0:
            return "ศูนย์"
        case n if n % 2 == 0:
            return f"{n} เป็นจำนวนบวกคู่"
        case n if n % 2 != 0:
            return f"{n} เป็นจำนวนบวกคี่"

for num in [-5, 0, 4, 7, 100, 13]:
    print(categorize_number(num))
```

### ตัวอย่างที่ 20: match กับ Dict/Class Pattern

```python
# match กับ dictionary
def process_event(event):
    match event:
        case {"type": "click", "button": button}:
            return f"คลิกปุ่ม: {button}"
        case {"type": "keypress", "key": key}:
            return f"กดคีย์: {key}"
        case {"type": "scroll", "direction": direction, "amount": amount}:
            return f"เลื่อน {direction} {amount} หน่วย"
        case _:
            return "เหตุการณ์ไม่รู้จัก"

events = [
    {"type": "click", "button": "left"},
    {"type": "keypress", "key": "Enter"},
    {"type": "scroll", "direction": "down", "amount": 100},
    {"type": "hover"},
]

for event in events:
    print(f"{event} -> {process_event(event)}")
```

---

## 7. Guard Clauses และ Early Returns

### ความหมาย

Guard clauses คือเทคนิคการตรวจสอบเงื่อนไขที่ต้องการ "หลีกเลี่ยง" ก่อน แล้วคืนค่า (return) ออกจาก function ทันที ทำให้โค้ดที่เหลืออ่านง่ายขึ้น

### ตัวอย่างที่ 21: เปรียบเทียบ Nested vs Guard Clauses

```python
# ❌ แบบ Nested (อ่านยาก)
def process_user_bad(user):
    if user is not None:
        if user.get("active"):
            if user.get("age", 0) >= 18:
                if user.get("verified"):
                    return f"ประมวลผลผู้ใช้: {user['name']}"
                else:
                    return "ผู้ใช้ยังไม่ได้ยืนยัน"
            else:
                return "ผู้ใช้อายุน้อยเกินไป"
        else:
            return "ผู้ใช้ถูกระงับการใช้งาน"
    else:
        return "ไม่พบข้อมูลผู้ใช้"

# ✅ แบบ Guard Clauses (อ่านง่าย)
def process_user_good(user):
    # Guard: ตรวจสอบเงื่อนไขที่ไม่ต้องการก่อน
    if user is None:
        return "ไม่พบข้อมูลผู้ใช้"
    
    if not user.get("active"):
        return "ผู้ใช้ถูกระงับการใช้งาน"
    
    if user.get("age", 0) < 18:
        return "ผู้ใช้อายุน้อยเกินไป"
    
    if not user.get("verified"):
        return "ผู้ใช้ยังไม่ได้ยืนยัน"
    
    # โค้ดหลัก (Happy path)
    return f"ประมวลผลผู้ใช้: {user['name']}"

# ทดสอบ
test_users = [
    None,
    {"name": "Alice", "active": False, "age": 20, "verified": True},
    {"name": "Bob", "active": True, "age": 15, "verified": True},
    {"name": "Charlie", "active": True, "age": 25, "verified": False},
    {"name": "Diana", "active": True, "age": 25, "verified": True},
]

print("=== Guard Clauses ===")
for user in test_users:
    result = process_user_good(user)
    name = user["name"] if user else "None"
    print(f"{name}: {result}")
```

### ตัวอย่างที่ 22: Guard Clauses สำหรับ Input Validation

```python
# Validation ด้วย Guard Clauses
def calculate_discount(price, discount_percent, customer_type):
    # Guards
    if price <= 0:
        raise ValueError("ราคาต้องมากกว่า 0")
    
    if not 0 <= discount_percent <= 100:
        raise ValueError("ส่วนลดต้องอยู่ระหว่าง 0-100%")
    
    if customer_type not in ["normal", "vip", "premium"]:
        raise ValueError(f"ประเภทลูกค้าไม่ถูกต้อง: {customer_type}")
    
    # Logic หลัก
    base_discount = price * (discount_percent / 100)
    
    # เพิ่มส่วนลดพิเศษตามประเภทลูกค้า
    if customer_type == "vip":
        extra_discount = price * 0.05  # เพิ่ม 5%
    elif customer_type == "premium":
        extra_discount = price * 0.10  # เพิ่ม 10%
    else:
        extra_discount = 0
    
    total_discount = base_discount + extra_discount
    final_price = price - total_discount
    
    return {
        "original_price": price,
        "discount": total_discount,
        "final_price": final_price
    }

# ทดสอบ
try:
    result = calculate_discount(1000, 10, "vip")
    print(f"ราคาเดิม: {result['original_price']:.2f}")
    print(f"ส่วนลดรวม: {result['discount']:.2f}")
    print(f"ราคาสุดท้าย: {result['final_price']:.2f}")
except ValueError as e:
    print(f"Error: {e}")
```

---

## 8. Complex Conditions

### Logical Operators

| Operator | ความหมาย | ตัวอย่าง |
|----------|-----------|----------|
| `and` | ทั้งสองต้องเป็น True | `x > 0 and y > 0` |
| `or` | อย่างน้อยหนึ่งต้องเป็น True | `x > 0 or y > 0` |
| `not` | ตรงกันข้าม | `not (x > 0)` |

### ตัวอย่างที่ 23: Logical Operators

```python
# and, or, not
x, y, z = 5, 10, 15

print("=== and ===")
print(f"x > 0 and y > 0: {x > 0 and y > 0}")
print(f"x > 0 and y > 20: {x > 0 and y > 20}")

print("\n=== or ===")
print(f"x > 10 or y > 5: {x > 10 or y > 5}")
print(f"x > 10 or y > 20: {x > 10 or y > 20}")

print("\n=== not ===")
print(f"not (x > 10): {not (x > 10)}")
print(f"not (x > 0): {not (x > 0)}")

print("\n=== การรวมกัน ===")
# ตรวจสอบช่วงอายุ
age = 25
is_student = True

if 18 <= age <= 30 and is_student:
    print("เป็นนักศึกษาในช่วงอายุ 18-30 ปี")

# Short-circuit evaluation
def expensive_check():
    print("  ทำการตรวจสอบ...")
    return True

print("\nShort-circuit with and:")
result = False and expensive_check()  # expensive_check จะไม่ถูกเรียก
print(f"False and expensive_check(): ไม่เรียก expensive_check")

print("\nShort-circuit with or:")
result = True or expensive_check()  # expensive_check จะไม่ถูกเรียก
print(f"True or expensive_check(): ไม่เรียก expensive_check")
```

### ตัวอย่างที่ 24: Chained Comparisons

```python
# Python รองรับ chained comparisons
x = 5

# แทนที่จะเขียน: x > 0 and x < 10
if 0 < x < 10:
    print(f"{x} อยู่ระหว่าง 0 ถึง 10")

# ตรวจสอบช่วงคะแนน
score = 75
if 60 <= score < 70:
    print("เกรด D")
elif 70 <= score < 80:
    print("เกรด C")
elif 80 <= score < 90:
    print("เกรด B")
elif 90 <= score <= 100:
    print("เกรด A")

# ตรวจสอบลำดับ
a, b, c = 1, 5, 10
if a < b < c:
    print(f"{a} < {b} < {c} เป็นจริง")
```

### ตัวอย่างที่ 25: is และ is not

```python
# is ตรวจสอบ identity (ชี้ไปที่ object เดียวกัน)
# == ตรวจสอบ equality (มีค่าเท่ากัน)

x = None
if x is None:
    print("x เป็น None")

if x is not None:
    print("x ไม่เป็น None")
else:
    print("x เป็น None (ยืนยัน)")

# ตัวอย่างที่ควรระวัง
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(f"\na == b: {a == b}")    # True (ค่าเท่ากัน)
print(f"a is b: {a is b}")    # False (คนละ object)
print(f"a is c: {a is c}")    # True (object เดียวกัน)

# ใช้ is กับ None, True, False เท่านั้น
value = None
if value is None:  # ✅ ถูกต้อง
    print("value is None")

# if value == None:  # ⚠️ ทำงานได้ แต่ไม่แนะนำ
```

---

## 9. Best Practices สำหรับ Conditionals

### ตัวอย่างที่ 26: หลักการ "Pythonic" Conditions

```python
# ✅ Pythonic style

# 1. ใช้ Truthy/Falsy แทนการเปรียบเทียบกับ True/False
items = [1, 2, 3]
name = "Alice"

# ❌ ไม่แนะนำ
if len(items) > 0:
    print("มีสมาชิก")
if name == "":
    print("ชื่อว่าง")
if name != "":
    print("มีชื่อ")

# ✅ แนะนำ
if items:
    print("มีสมาชิก")
if not name:
    print("ชื่อว่าง")
if name:
    print("มีชื่อ")

# 2. ใช้ in สำหรับการตรวจสอบหลายค่า
color = "red"

# ❌ ไม่แนะนำ
if color == "red" or color == "green" or color == "blue":
    print("เป็นสีหลัก RGB")

# ✅ แนะนำ
if color in ("red", "green", "blue"):
    print("เป็นสีหลัก RGB")

# 3. ใช้ any() และ all()
scores = [85, 92, 78, 95, 88]

# ตรวจสอบว่าทุกคะแนนผ่าน
if all(score >= 60 for score in scores):
    print("ทุกคนผ่าน!")

# ตรวจสอบว่ามีคะแนนเต็มอย่างน้อยหนึ่งคน
if any(score == 100 for score in scores):
    print("มีคนได้คะแนนเต็ม!")
```

### ตัวอย่างที่ 27: การจัดลำดับ Conditions

```python
# จัดลำดับเงื่อนไขจากที่พบบ่อยที่สุดไปหาน้อยที่สุด
# เพื่อประสิทธิภาพที่ดีขึ้น

import random

def categorize_http_status(status_code):
    # จัดลำดับตามความถี่ที่พบบ่อย
    if status_code == 200:          # พบมากที่สุด
        return "OK"
    elif status_code == 404:        # พบบ่อยครั้ง
        return "Not Found"
    elif status_code == 400:        # พบปานกลาง
        return "Bad Request"
    elif status_code == 401:
        return "Unauthorized"
    elif status_code == 403:
        return "Forbidden"
    elif status_code == 500:        # พบน้อย
        return "Internal Server Error"
    elif 200 <= status_code < 300:
        return "Success (2xx)"
    elif 300 <= status_code < 400:
        return "Redirect (3xx)"
    elif 400 <= status_code < 500:
        return "Client Error (4xx)"
    elif 500 <= status_code < 600:
        return "Server Error (5xx)"
    else:
        return "Unknown"

# ทดสอบ
status_codes = [200, 404, 500, 302, 201, 403]
for code in status_codes:
    print(f"HTTP {code}: {categorize_http_status(code)}")
```

### ตัวอย่างที่ 28: Dictionary Dispatch Pattern

```python
# แทนที่ if/elif ยาวๆ ด้วย dictionary
def handle_operation_bad(operation, x, y):
    if operation == "add":
        return x + y
    elif operation == "subtract":
        return x - y
    elif operation == "multiply":
        return x * y
    elif operation == "divide":
        if y == 0:
            raise ValueError("หารด้วยศูนย์ไม่ได้")
        return x / y
    else:
        raise ValueError(f"ไม่รู้จักการดำเนินการ: {operation}")

# ✅ แบบ Dictionary Dispatch
def handle_operation_good(operation, x, y):
    operations = {
        "add": lambda a, b: a + b,
        "subtract": lambda a, b: a - b,
        "multiply": lambda a, b: a * b,
        "divide": lambda a, b: a / b if b != 0 else (_ for _ in ()).throw(ValueError("หารด้วยศูนย์ไม่ได้")),
    }
    
    if operation not in operations:
        raise ValueError(f"ไม่รู้จักการดำเนินการ: {operation}")
    
    return operations[operation](x, y)

# ทดสอบ
ops = ["add", "subtract", "multiply"]
for op in ops:
    result = handle_operation_bad(op, 10, 3)
    print(f"10 {op} 3 = {result}")
```

---

## 10. ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Grade Calculator

```python
def grade_calculator():
    """ระบบคำนวณเกรดนักศึกษา"""
    print("=" * 50)
    print("ระบบคำนวณเกรดนักศึกษา")
    print("=" * 50)
    
    # ข้อมูลนักศึกษา
    students = [
        {"name": "สมชาย ใจดี", "scores": [85, 92, 78, 88, 95]},
        {"name": "สมหญิง รักเรียน", "scores": [70, 65, 72, 68, 75]},
        {"name": "วิชัย เก่งมาก", "scores": [98, 95, 97, 92, 99]},
        {"name": "นารี อ่อนหวาน", "scores": [55, 60, 58, 62, 50]},
        {"name": "ประยุทธ ขยัน", "scores": [75, 78, 80, 73, 77]},
    ]
    
    def calculate_average(scores):
        return sum(scores) / len(scores)
    
    def get_grade(average):
        if average >= 80:
            return "A", "ดีเยี่ยม", True
        elif average >= 70:
            return "B", "ดี", True
        elif average >= 60:
            return "C", "พอใช้", True
        elif average >= 50:
            return "D", "อ่อน", True
        else:
            return "F", "ไม่ผ่าน", False
    
    print(f"\n{'ชื่อ':<20} {'เฉลี่ย':>8} {'เกรด':>6} {'ผล':>10}")
    print("-" * 50)
    
    pass_count = 0
    fail_count = 0
    
    for student in students:
        avg = calculate_average(student["scores"])
        grade, description, passed = get_grade(avg)
        
        if passed:
            pass_count += 1
            status = "✓ ผ่าน"
        else:
            fail_count += 1
            status = "✗ ไม่ผ่าน"
        
        print(f"{student['name']:<20} {avg:>8.1f} {grade:>6} {status:>10}")
    
    print("-" * 50)
    print(f"\nสรุป: ผ่าน {pass_count} คน | ไม่ผ่าน {fail_count} คน")
    print(f"อัตราผ่าน: {pass_count/len(students)*100:.1f}%")

grade_calculator()
```

### โปรแกรมที่ 2: Tax Calculator

```python
def tax_calculator(income):
    """คำนวณภาษีเงินได้บุคคลธรรมดา (อัตราสมมุติ)"""
    
    # อัตราภาษีแบบขั้นบันได
    tax_brackets = [
        (150000, 0),      # 0 - 150,000: ยกเว้น
        (300000, 0.05),   # 150,001 - 300,000: 5%
        (500000, 0.10),   # 300,001 - 500,000: 10%
        (750000, 0.15),   # 500,001 - 750,000: 15%
        (1000000, 0.20),  # 750,001 - 1,000,000: 20%
        (2000000, 0.25),  # 1,000,001 - 2,000,000: 25%
        (float('inf'), 0.35),  # มากกว่า 2,000,000: 35%
    ]
    
    if income <= 0:
        return 0, 0
    
    total_tax = 0
    remaining = income
    prev_bracket = 0
    
    print(f"\nรายได้: {income:,.2f} บาท")
    print(f"{'ช่วงรายได้':<30} {'ภาษี':>10}")
    print("-" * 45)
    
    for bracket, rate in tax_brackets:
        if remaining <= 0:
            break
            
        taxable = min(remaining, bracket - prev_bracket)
        tax = taxable * rate
        total_tax += tax
        
        if taxable > 0:
            range_str = f"{prev_bracket+1:,.0f} - {min(bracket, income):,.0f}"
            print(f"  {range_str:<28} {tax:>10,.2f}")
        
        remaining -= taxable
        prev_bracket = bracket
    
    effective_rate = (total_tax / income) * 100 if income > 0 else 0
    
    print("-" * 45)
    print(f"{'ภาษีรวม':<30} {total_tax:>10,.2f} บาท")
    print(f"{'อัตราภาษีที่แท้จริง':<30} {effective_rate:>10.2f}%")
    print(f"{'รายได้สุทธิหลังหักภาษี':<30} {income-total_tax:>10,.2f} บาท")
    
    return total_tax, effective_rate

# ทดสอบ
incomes = [100000, 500000, 1000000, 3000000]
for income in incomes:
    tax_calculator(income)
    print()
```

### โปรแกรมที่ 3: Login System

```python
def login_system():
    """ระบบ Login อย่างง่าย"""
    
    # ฐานข้อมูลผู้ใช้ (ในโปรแกรมจริงควรเก็บใน database)
    users = {
        "admin": {
            "password": "admin123",
            "role": "admin",
            "name": "ผู้ดูแลระบบ",
            "active": True
        },
        "alice": {
            "password": "alice456",
            "role": "user",
            "name": "Alice Smith",
            "active": True
        },
        "bob": {
            "password": "bob789",
            "role": "user",
            "name": "Bob Jones",
            "active": False  # บัญชีถูกระงับ
        }
    }
    
    MAX_ATTEMPTS = 3
    
    def authenticate(username, password):
        """ตรวจสอบ credentials"""
        # ตรวจสอบว่ามี username หรือไม่
        if username not in users:
            return False, "ไม่พบชื่อผู้ใช้ในระบบ"
        
        user = users[username]
        
        # ตรวจสอบว่าบัญชียังใช้งานได้
        if not user["active"]:
            return False, "บัญชีของคุณถูกระงับการใช้งาน กรุณาติดต่อผู้ดูแลระบบ"
        
        # ตรวจสอบรหัสผ่าน
        if user["password"] != password:
            return False, "รหัสผ่านไม่ถูกต้อง"
        
        return True, "เข้าสู่ระบบสำเร็จ"
    
    def show_menu(role):
        """แสดงเมนูตาม role"""
        print("\n=== เมนูหลัก ===")
        print("1. ดูโปรไฟล์")
        print("2. เปลี่ยนรหัสผ่าน")
        
        if role == "admin":
            print("3. จัดการผู้ใช้ [Admin only]")
            print("4. ดูรายงาน [Admin only]")
        
        print("0. ออกจากระบบ")
    
    # ทดสอบการ login
    test_credentials = [
        ("unknown", "password"),
        ("bob", "bob789"),        # บัญชีถูกระงับ
        ("alice", "wrong"),       # รหัสผ่านผิด
        ("alice", "alice456"),    # ถูกต้อง
        ("admin", "admin123"),    # ถูกต้อง + admin
    ]
    
    print("=" * 50)
    print("ทดสอบระบบ Login")
    print("=" * 50)
    
    for username, password in test_credentials:
        print(f"\nทดสอบ: username='{username}', password='{password}'")
        success, message = authenticate(username, password)
        
        if success:
            user = users[username]
            print(f"✓ {message}")
            print(f"  ยินดีต้อนรับ {user['name']} (Role: {user['role']})")
            show_menu(user['role'])
        else:
            print(f"✗ {message}")

login_system()
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดข้อที่ 1: ตรวจสอบปีอธิกสุรทิน

```
จงเขียนโปรแกรมตรวจสอบว่าปีที่รับเข้ามาเป็นปีอธิกสุรทิน (Leap Year) หรือไม่
กฎ: ปีอธิกสุรทินต้องหารด้วย 4 ลงตัว แต่ถ้าหารด้วย 100 ลงตัว ต้องหารด้วย 400 ลงตัวด้วย
```

**เฉลย:**

```python
def is_leap_year(year):
    """ตรวจสอบปีอธิกสุรทิน"""
    if year % 400 == 0:
        return True
    elif year % 100 == 0:
        return False
    elif year % 4 == 0:
        return True
    else:
        return False

# ทดสอบ
test_years = [1900, 2000, 2024, 2023, 1600, 1700]
for year in test_years:
    result = "อธิกสุรทิน" if is_leap_year(year) else "ปกติ"
    print(f"ปี {year}: {result}")
```

---

### แบบฝึกหัดข้อที่ 2: เครื่องคิดเลขอย่างง่าย

```
จงเขียน function calculator(a, operator, b) ที่รับตัวเลข 2 ตัวและตัวดำเนินการ
(+, -, *, /, //, %, **) แล้วคืนผลลัพธ์
จัดการกรณี division by zero ด้วย
```

**เฉลย:**

```python
def calculator(a, operator, b):
    """เครื่องคิดเลขอย่างง่าย"""
    if operator == "+":
        return a + b
    elif operator == "-":
        return a - b
    elif operator == "*":
        return a * b
    elif operator == "/":
        if b == 0:
            return "Error: หารด้วยศูนย์ไม่ได้"
        return a / b
    elif operator == "//":
        if b == 0:
            return "Error: หารด้วยศูนย์ไม่ได้"
        return a // b
    elif operator == "%":
        if b == 0:
            return "Error: หารด้วยศูนย์ไม่ได้"
        return a % b
    elif operator == "**":
        return a ** b
    else:
        return f"Error: ไม่รู้จักตัวดำเนินการ '{operator}'"

# ทดสอบ
tests = [
    (10, "+", 5),
    (10, "-", 3),
    (4, "*", 7),
    (15, "/", 4),
    (15, "//", 4),
    (15, "%", 4),
    (2, "**", 10),
    (10, "/", 0),
    (5, "^", 2),
]

for a, op, b in tests:
    result = calculator(a, op, b)
    print(f"{a} {op} {b} = {result}")
```

---

### แบบฝึกหัดข้อที่ 3: ระบบจัดประเภทอุณหภูมิ

```
จงเขียนโปรแกรมที่รับอุณหภูมิ (Celsius) และจัดประเภท:
- น้อยกว่า 0°C: "น้ำแข็ง"
- 0-10°C: "หนาวมาก"
- 11-20°C: "หนาว"
- 21-30°C: "เย็นสบาย"
- 31-35°C: "อบอ้าว"
- มากกว่า 35°C: "ร้อนมาก"
```

**เฉลย:**

```python
def classify_temperature(celsius):
    """จัดประเภทอุณหภูมิ"""
    if celsius < 0:
        category = "น้ำแข็ง ❄️"
        advice = "ระวังน้ำแข็งเกาะ"
    elif celsius <= 10:
        category = "หนาวมาก 🥶"
        advice = "สวมเสื้อหนาหลายชั้น"
    elif celsius <= 20:
        category = "หนาว 🧥"
        advice = "ใส่เสื้อกันหนาว"
    elif celsius <= 30:
        category = "เย็นสบาย 😊"
        advice = "อากาศดี เหมาะออกกำลังกาย"
    elif celsius <= 35:
        category = "อบอ้าว 😅"
        advice = "ดื่มน้ำมากๆ"
    else:
        category = "ร้อนมาก 🌡️"
        advice = "หลีกเลี่ยงแสงแดด"
    
    fahrenheit = (celsius * 9/5) + 32
    print(f"{celsius}°C ({fahrenheit:.1f}°F): {category}")
    print(f"  คำแนะนำ: {advice}")

# ทดสอบ
temps = [-5, 5, 15, 25, 32, 40]
for temp in temps:
    classify_temperature(temp)
    print()
```

---

### แบบฝึกหัดข้อที่ 4: ระบบตรวจสอบรหัสผ่าน

```
จงเขียนโปรแกรมตรวจสอบความแข็งแกร่งของรหัสผ่าน:
- ความยาวน้อยกว่า 6 ตัว: "อ่อนแอมาก"
- ความยาว 6-8 ตัว แต่ไม่มีตัวพิมพ์ใหญ่/ตัวเลข/สัญลักษณ์: "อ่อนแอ"
- ผสมตัวพิมพ์ใหญ่และตัวเลข: "ปานกลาง"
- ผสมตัวพิมพ์ใหญ่ ตัวเลข และสัญลักษณ์: "แข็งแกร่ง"
```

**เฉลย:**

```python
def check_password_strength(password):
    """ตรวจสอบความแข็งแกร่งรหัสผ่าน"""
    length = len(password)
    has_upper = any(c.isupper() for c in password)
    has_lower = any(c.islower() for c in password)
    has_digit = any(c.isdigit() for c in password)
    has_special = any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?" for c in password)
    
    print(f"\nรหัสผ่าน: '{password}'")
    print(f"  ความยาว: {length} ตัว")
    print(f"  ตัวพิมพ์ใหญ่: {'✓' if has_upper else '✗'}")
    print(f"  ตัวพิมพ์เล็ก: {'✓' if has_lower else '✗'}")
    print(f"  ตัวเลข: {'✓' if has_digit else '✗'}")
    print(f"  สัญลักษณ์: {'✓' if has_special else '✗'}")
    
    if length < 6:
        strength = "อ่อนแอมาก ❌"
    elif length < 8 and not (has_upper and has_digit):
        strength = "อ่อนแอ ⚠️"
    elif has_upper and has_lower and has_digit and has_special and length >= 12:
        strength = "แข็งแกร่งมาก ✅✅"
    elif has_upper and has_lower and has_digit and has_special:
        strength = "แข็งแกร่ง ✅"
    elif has_upper and has_digit:
        strength = "ปานกลาง 🔶"
    else:
        strength = "อ่อนแอ ⚠️"
    
    print(f"  ความแข็งแกร่ง: {strength}")

# ทดสอบ
passwords = ["abc", "password", "Pass123", "P@ssw0rd!", "MyV3ryStr0ng!Pass#2024"]
for pwd in passwords:
    check_password_strength(pwd)
```

---

### แบบฝึกหัดข้อที่ 5: แปลงเกรดระหว่างระบบ

```
จงเขียนโปรแกรมแปลงเกรดจากระบบ 4.0 เป็นระบบ A-F และระดับความหมาย
```

**เฉลย:**

```python
def convert_grade(gpa):
    """แปลงเกรด GPA เป็นตัวอักษรและคำอธิบาย"""
    if not 0 <= gpa <= 4.0:
        return "ไม่ถูกต้อง", "GPA ต้องอยู่ระหว่าง 0.0 - 4.0"
    
    if gpa == 4.0:
        letter, description = "A", "ดีเยี่ยม (Excellent)"
    elif gpa >= 3.5:
        letter, description = "A-", "ดีมาก (Very Good)"
    elif gpa >= 3.0:
        letter, description = "B+", "ดี (Good)"
    elif gpa >= 2.5:
        letter, description = "B", "ค่อนข้างดี (Above Average)"
    elif gpa >= 2.0:
        letter, description = "C+", "ปานกลาง (Average)"
    elif gpa >= 1.5:
        letter, description = "C", "พอใช้ (Below Average)"
    elif gpa >= 1.0:
        letter, description = "D", "อ่อน (Poor)"
    else:
        letter, description = "F", "ไม่ผ่าน (Fail)"
    
    return letter, description

# ทดสอบ
gpas = [4.0, 3.7, 3.2, 2.8, 2.3, 1.8, 1.2, 0.5, 0.0]
print(f"{'GPA':>6} {'เกรด':>6} {'คำอธิบาย'}")
print("-" * 40)
for gpa in gpas:
    letter, desc = convert_grade(gpa)
    print(f"{gpa:>6.1f} {letter:>6} {desc}")
```

---

### แบบฝึกหัดข้อที่ 6: ระบบจัดส่งสินค้า

```
จงเขียนโปรแกรมคำนวณค่าจัดส่งสินค้า:
- น้ำหนัก 0-1 kg: 50 บาท
- น้ำหนัก 1-5 kg: 100 บาท
- น้ำหนัก 5-10 kg: 200 บาท
- น้ำหนักมากกว่า 10 kg: 200 บาท + 20 บาท/kg ที่เกิน 10 kg
- ถ้าซื้อสินค้ามากกว่า 1000 บาท จัดส่งฟรี (ยกเว้นน้ำหนักเกิน 10 kg)
```

**เฉลย:**

```python
def calculate_shipping(weight_kg, order_value):
    """คำนวณค่าจัดส่ง"""
    
    if weight_kg <= 0:
        return 0, "น้ำหนักไม่ถูกต้อง"
    
    # คำนวณค่าจัดส่งตามน้ำหนัก
    if weight_kg <= 1:
        base_fee = 50
    elif weight_kg <= 5:
        base_fee = 100
    elif weight_kg <= 10:
        base_fee = 200
    else:
        extra_kg = weight_kg - 10
        base_fee = 200 + (extra_kg * 20)
    
    # ตรวจสอบ Free Shipping
    if order_value >= 1000 and weight_kg <= 10:
        shipping_fee = 0
        note = "จัดส่งฟรี!"
    else:
        shipping_fee = base_fee
        if order_value >= 1000:
            note = "น้ำหนักเกิน 10kg ไม่ได้รับส่วนลดค่าจัดส่ง"
        else:
            note = f"ซื้อเพิ่มอีก {1000-order_value:.2f} บาท เพื่อรับจัดส่งฟรี"
    
    return shipping_fee, note

# ทดสอบ
print(f"{'น้ำหนัก':>8} {'ยอดซื้อ':>10} {'ค่าส่ง':>8} หมายเหตุ")
print("-" * 60)
test_cases = [
    (0.5, 500), (0.5, 1200), (3, 800), (3, 1500),
    (8, 200), (12, 500), (12, 2000),
]
for weight, value in test_cases:
    fee, note = calculate_shipping(weight, value)
    print(f"{weight:>8.1f}kg {value:>10.2f}฿ {fee:>8.2f}฿ {note}")
```

---

### แบบฝึกหัดข้อที่ 7: ตรวจสอบปีราศีจีน

```
จงเขียนโปรแกรมที่รับปีแล้วบอกว่าเป็นปีราศีอะไร
(12 ปีราศีจีน: หนู วัว เสือ กระต่าย มังกร งู ม้า แพะ ลิง ไก่ หมา หมู)
```

**เฉลย:**

```python
def get_chinese_zodiac(year):
    """หาปีราศีจีน"""
    zodiac_animals = [
        "หนู", "วัว", "เสือ", "กระต่าย",
        "มังกร", "งู", "ม้า", "แพะ",
        "ลิง", "ไก่", "หมา", "หมู"
    ]
    
    # ปี 2020 เป็นปีหนู
    base_year = 2020
    index = (year - base_year) % 12
    
    animal = zodiac_animals[index]
    
    # คำทำนาย (สมมุติ)
    fortunes = {
        "หนู": "ปีแห่งความฉลาดและความมั่งมี",
        "วัว": "ปีแห่งความขยันและความมั่นคง",
        "เสือ": "ปีแห่งความกล้าหาญและการผจญภัย",
        "กระต่าย": "ปีแห่งความสงบและความเจริญรุ่งเรือง",
        "มังกร": "ปีแห่งโชคลาภและความสำเร็จ",
        "งู": "ปีแห่งปัญญาและความลึกซึ้ง",
        "ม้า": "ปีแห่งพลังและอิสรภาพ",
        "แพะ": "ปีแห่งความงามและความคิดสร้างสรรค์",
        "ลิง": "ปีแห่งความชาญฉลาดและความร่าเริง",
        "ไก่": "ปีแห่งความตรงไปตรงมาและความมุ่งมั่น",
        "หมา": "ปีแห่งความซื่อสัตย์และความภักดี",
        "หมู": "ปีแห่งความมีน้ำใจและความสุข",
    }
    
    return animal, fortunes.get(animal, "ไม่พบข้อมูล")

# ทดสอบ
test_years = [1990, 2000, 2010, 2020, 2024, 2025, 2026]
for year in test_years:
    animal, fortune = get_chinese_zodiac(year)
    print(f"ปี {year}: ปี{animal} - {fortune}")
```

---

### แบบฝึกหัดข้อที่ 8: ระบบคะแนน RPG

```
จงเขียนโปรแกรมกำหนด level และ class ของตัวละคร RPG จากคะแนนประสบการณ์ (XP):
- Level 1: 0-99 XP
- Level 2: 100-299 XP
- Level 3: 300-599 XP
- Level 4: 600-999 XP
- Level 5+: 1000+ XP (คำนวณ level เพิ่มเติม)
Class ขึ้นอยู่กับ level: 1-2=Novice, 3-4=Warrior, 5+=Hero
```

**เฉลย:**

```python
def get_rpg_level(xp):
    """คำนวณ Level และ Class จาก XP"""
    if xp < 0:
        return None, None, "XP ไม่ถูกต้อง"
    
    if xp < 100:
        level = 1
    elif xp < 300:
        level = 2
    elif xp < 600:
        level = 3
    elif xp < 1000:
        level = 4
    else:
        # Level 5 ขึ้นไป: ทุก 500 XP เพิ่ม 1 level
        level = 5 + (xp - 1000) // 500
    
    # กำหนด Class
    if level <= 2:
        char_class = "Novice"
        class_emoji = "👶"
    elif level <= 4:
        char_class = "Warrior"
        class_emoji = "⚔️"
    elif level <= 7:
        char_class = "Hero"
        class_emoji = "🦸"
    else:
        char_class = "Legend"
        class_emoji = "👑"
    
    # คำนวณ XP ถึง next level
    level_thresholds = [0, 100, 300, 600, 1000]
    if level < 5:
        next_xp = level_thresholds[level]
        xp_needed = next_xp - xp
    else:
        next_level_xp = 1000 + ((level - 4) * 500)
        xp_needed = next_level_xp - xp
    
    return level, char_class, f"{class_emoji} {char_class} (XP ถึง Level ถัดไป: {xp_needed})"

# ทดสอบ
xp_values = [0, 50, 150, 400, 800, 1200, 2500]
print(f"{'XP':>6} {'Level':>7} {'Class'}")
print("-" * 45)
for xp in xp_values:
    level, char_class, info = get_rpg_level(xp)
    print(f"{xp:>6} Level {level:>2} {info}")
```

---

### แบบฝึกหัดข้อที่ 9: ระบบจองตั๋ว

```
จงเขียนโปรแกรมคำนวณราคาตั๋วภาพยนตร์:
- ราคาปกติ: 180 บาท
- เด็ก (อายุน้อยกว่า 12): ลด 50%
- ผู้สูงอายุ (อายุ 60+): ลด 30%
- วันอังคาร: ลด 20% (ไม่รวมกับส่วนลดอื่น)
- สมาชิก: ลด 10% เพิ่มเติม (รวมกับส่วนลดได้)
```

**เฉลย:**

```python
def calculate_ticket_price(age, day_of_week, is_member):
    """คำนวณราคาตั๋วภาพยนตร์"""
    BASE_PRICE = 180
    
    # กำหนดส่วนลดตามอายุหรือวัน
    if day_of_week.lower() == "tuesday":
        discount_rate = 0.20
        discount_reason = "วันอังคารพิเศษ"
    elif age < 12:
        discount_rate = 0.50
        discount_reason = "เด็ก"
    elif age >= 60:
        discount_rate = 0.30
        discount_reason = "ผู้สูงอายุ"
    else:
        discount_rate = 0
        discount_reason = "ปกติ"
    
    # คำนวณราคาหลักส่วนลด
    price_after_main_discount = BASE_PRICE * (1 - discount_rate)
    
    # ส่วนลดสมาชิก
    if is_member:
        member_discount = price_after_main_discount * 0.10
        final_price = price_after_main_discount - member_discount
        member_note = "(-10% สมาชิก)"
    else:
        member_discount = 0
        final_price = price_after_main_discount
        member_note = ""
    
    print(f"ราคาปกติ: {BASE_PRICE} บาท")
    print(f"ประเภท: {discount_reason}")
    if discount_rate > 0:
        print(f"ส่วนลด: {discount_rate*100:.0f}% = {BASE_PRICE*discount_rate:.0f} บาท")
    if is_member:
        print(f"ส่วนลดสมาชิก: {member_discount:.0f} บาท")
    print(f"ราคาสุดท้าย: {final_price:.0f} บาท {member_note}")
    
    return final_price

# ทดสอบ
print("=== กรณีที่ 1: เด็กอายุ 8 ปี สมาชิก ===")
calculate_ticket_price(8, "Friday", True)

print("\n=== กรณีที่ 2: ผู้ใหญ่อายุ 25 ปี วันอังคาร ===")
calculate_ticket_price(25, "Tuesday", False)

print("\n=== กรณีที่ 3: ผู้สูงอายุ 65 ปี สมาชิก ===")
calculate_ticket_price(65, "Saturday", True)

print("\n=== กรณีที่ 4: ผู้ใหญ่ปกติ ===")
calculate_ticket_price(30, "Wednesday", False)
```

---

### แบบฝึกหัดข้อที่ 10: ระบบแนะนำอาหาร

```
จงเขียนโปรแกรมแนะนำอาหารตามเงื่อนไข:
- รับ: งบประมาณ (บาท), ประเภทอาหาร (Thai/Western/Japanese), 
        จำนวนคน, ต้องการ vegetarian หรือไม่
- แนะนำเมนูที่เหมาะสม
```

**เฉลย:**

```python
def recommend_food(budget, cuisine_type, num_people, vegetarian=False):
    """แนะนำอาหารตามเงื่อนไข"""
    
    budget_per_person = budget / num_people
    
    print(f"\nงบประมาณรวม: {budget} บาท")
    print(f"จำนวนคน: {num_people} คน")
    print(f"งบต่อคน: {budget_per_person:.0f} บาท")
    print(f"ประเภทอาหาร: {cuisine_type}")
    print(f"Vegetarian: {'ใช่' if vegetarian else 'ไม่'}")
    print("\n=== เมนูแนะนำ ===")
    
    if cuisine_type.lower() == "thai":
        if vegetarian:
            if budget_per_person < 50:
                print("- ข้าวผัดผักรวม (35 บาท)")
                print("- ต้มยำเห็ด (40 บาท)")
            elif budget_per_person < 150:
                print("- แกงเขียวหวานเต้าหู้ (80 บาท)")
                print("- ผัดไทยไม่ใส่กุ้ง (70 บาท)")
                print("- ต้มข่าเห็ด (90 บาท)")
            else:
                print("- แกงมัสมั่นผัก (150 บาท)")
                print("- ยำวุ้นเส้นเจ (120 บาท)")
                print("- ข้าวซอยเต้าหู้ (130 บาท)")
        else:
            if budget_per_person < 50:
                print("- ข้าวมันไก่ (45 บาท)")
                print("- ข้าวราดแกง (40 บาท)")
            elif budget_per_person < 150:
                print("- ผัดกะเพราหมูสับ (80 บาท)")
                print("- ต้มยำกุ้ง (120 บาท)")
                print("- ผัดไทยกุ้งสด (100 บาท)")
            else:
                print("- ปูผัดผงกะหรี่ (350 บาท)")
                print("- ต้มยำทะเลหม้อไฟ (280 บาท)")
                print("- ยำทะเลสด (200 บาท)")
    
    elif cuisine_type.lower() == "japanese":
        if vegetarian:
            if budget_per_person < 100:
                print("- ซูชิผัก (80 บาท)")
                print("- มิโซะซุป (50 บาท)")
            else:
                print("- เซตข้าวหน้าผัก (180 บาท)")
                print("- ซูชิผักออร์แกนิค (250 บาท)")
        else:
            if budget_per_person < 200:
                print("- ราเม็งหมู (150 บาท)")
                print("- ข้าวหน้าไก่คาราเกะ (180 บาท)")
            else:
                print("- ซูชิแซลมอน (350 บาท)")
                print("- ชาบูหมูวากิว (450 บาท)")
    
    elif cuisine_type.lower() == "western":
        if vegetarian:
            if budget_per_person < 150:
                print("- พิซซ่ามาร์เกอรีต้า (120 บาท)")
                print("- แซนวิชผัก (100 บาท)")
            else:
                print("- พาสต้า Aglio e Olio (200 บาท)")
                print("- ซาลัด Caesar Vegetarian (180 บาท)")
        else:
            if budget_per_person < 200:
                print("- เบอร์เกอร์ไก่ (150 บาท)")
                print("- พาสต้าโบโลเนส (180 บาท)")
            else:
                print("- สเต็กเนื้อ Ribeye (600 บาท)")
                print("- แร็คแลมบ์ (500 บาท)")
    else:
        print(f"ไม่รองรับประเภทอาหาร: {cuisine_type}")
        print("ประเภทที่รองรับ: Thai, Japanese, Western")

# ทดสอบ
recommend_food(500, "Thai", 2, False)
recommend_food(300, "Japanese", 1, True)
recommend_food(2000, "Western", 3, False)
```

---

## สรุป Part 06

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| if statement | การตรวจสอบเงื่อนไขพื้นฐาน |
| if/else | การกำหนดทางเลือก 2 ทาง |
| if/elif/else | การกำหนดทางเลือกหลายทาง |
| Nested if | การตรวจสอบเงื่อนไขหลายระดับ |
| Ternary operator | การเขียน if/else ในบรรทัดเดียว |
| match/case | Pattern matching ใน Python 3.10+ |
| Guard clauses | เทคนิคลด nesting ทำโค้ดอ่านง่าย |
| Complex conditions | and, or, not, in, is |
| Best practices | วิธีเขียน conditional ที่ดี |

### Key Takeaways:
1. Python ใช้ **indentation** แทนวงเล็บปีกกา
2. ค่า **Falsy** ได้แก่ `0`, `""`, `[]`, `{}`, `None`, `False`
3. ใช้ **Guard clauses** เพื่อลด nested if
4. **match/case** เหมาะสำหรับ pattern matching ที่ซับซ้อน
5. ใช้ **in** แทน `or` เมื่อตรวจสอบหลายค่า

---

*Part 06 จบแล้ว ไปต่อที่ [Part 07 - Loops: for Loop](../part07/README.md)*
