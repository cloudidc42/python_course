# Part 08 - Loops: while Loop

## สารบัญ

1. [while Loop พื้นฐาน](#1-while-loop-พื้นฐาน)
2. [while กับ break](#2-while-กับ-break)
3. [while กับ continue](#3-while-กับ-continue)
4. [while/else](#4-whileelse)
5. [Infinite Loops และวิธีหลีกเลี่ยง](#5-infinite-loops-และวิธีหลีกเลี่ยง)
6. [do-while Pattern ใน Python](#6-do-while-pattern-ใน-python)
7. [while กับ User Input](#7-while-กับ-user-input)
8. [Counter Patterns](#8-counter-patterns)
9. [Flag Variables](#9-flag-variables)
10. [ตัวอย่างโปรแกรมจริง](#10-ตัวอย่างโปรแกรมจริง)
11. [แบบฝึกหัด](#11-แบบฝึกหัด)

---

## 1. while Loop พื้นฐาน

### ความหมายและการทำงาน

`while` loop ทำงานซ้ำๆ ตราบที่เงื่อนไขยังเป็น `True` ต่างจาก `for` loop ที่วนซ้ำผ่าน iterable, `while` loop ใช้เมื่อไม่รู้ว่าจะวนซ้ำกี่ครั้ง

### Syntax

```python
while condition:
    # โค้ดที่จะทำงานซ้ำ
    statement
    # อย่าลืมอัปเดตเงื่อนไข มิฉะนั้นจะวนไม่จบ
```

### ตัวอย่างที่ 1: while loop พื้นฐาน

```python
# นับเลข 1 ถึง 5
count = 1

while count <= 5:
    print(f"รอบที่ {count}")
    count += 1  # สำคัญ! ต้องอัปเดตตัวแปร

print("จบการวนซ้ำ")
```

**Output:**
```
รอบที่ 1
รอบที่ 2
รอบที่ 3
รอบที่ 4
รอบที่ 5
จบการวนซ้ำ
```

### ตัวอย่างที่ 2: เปรียบเทียบ while กับ for

```python
# while loop
i = 0
while i < 5:
    print(f"while: {i}")
    i += 1

print()

# for loop (เทียบเท่า)
for i in range(5):
    print(f"for: {i}")
```

### ตัวอย่างที่ 3: while กับเงื่อนไขที่ซับซ้อน

```python
# ลดค่าของ list จนกว่าจะว่าง
stack = [1, 2, 3, 4, 5]

print(f"Stack เริ่มต้น: {stack}")
print("Pop ออกทีละตัว:")

while stack:  # ทำงานจนกว่า list จะว่าง (Falsy)
    item = stack.pop()
    print(f"  Pop: {item}, stack เหลือ: {stack}")

print("Stack ว่างแล้ว")
```

### ตัวอย่างที่ 4: while กับ string processing

```python
# ตัดช่องว่างออกทีละตัว
text = "   Hello World   "
print(f"ก่อน: '{text}'")

left_idx = 0
while left_idx < len(text) and text[left_idx] == ' ':
    left_idx += 1

right_idx = len(text) - 1
while right_idx >= 0 and text[right_idx] == ' ':
    right_idx -= 1

trimmed = text[left_idx:right_idx + 1]
print(f"หลัง trim: '{trimmed}'")
print(f"(เหมือน str.strip(): '{text.strip()}')")
```

---

## 2. while กับ break

### ตัวอย่างที่ 5: break ใน while loop

```python
# ค้นหาตัวเลขที่หารด้วย 7 และ 11 ลงตัวตัวแรก
print("หาตัวเลขที่หารด้วย 7 และ 11 ลงตัวตัวแรก:")
n = 1
while True:  # loop ไม่มีที่สิ้นสุด
    if n % 7 == 0 and n % 11 == 0:
        print(f"พบ: {n}")
        break  # หยุด loop
    n += 1

print(f"(ยืนยัน: {n} % 7 = {n%7}, {n} % 11 = {n%11})")
```

### ตัวอย่างที่ 6: break กับการ validate input

```python
# Simulate การรับ input จากผู้ใช้
valid_inputs = ["yes", "no"]
test_inputs = ["maybe", "yep", "YES", "yes"]  # จำลอง user input

input_idx = 0
while True:
    user_input = test_inputs[input_idx]
    input_idx += 1
    
    print(f"User input: '{user_input}'")
    
    if user_input.lower() in valid_inputs:
        print(f"Input ถูกต้อง: {user_input.lower()}")
        break
    else:
        print("  กรุณาตอบ yes หรือ no เท่านั้น")

print("โปรแกรมดำเนินต่อ...")
```

---

## 3. while กับ continue

### ตัวอย่างที่ 7: continue ใน while loop

```python
# ข้ามตัวเลขที่หารด้วย 3 ลงตัว
i = 0
print("ตัวเลข 1-20 ที่ไม่หารด้วย 3 ลงตัว:")
while i < 20:
    i += 1
    if i % 3 == 0:
        continue  # ข้ามรอบนี้
    print(i, end=" ")
print()
```

### ตัวอย่างที่ 8: continue กับ data processing

```python
# ประมวลผลข้อมูล ข้ามค่าที่ไม่ถูกต้อง
data = [5, -1, 3, None, 8, 0, -4, 7, None, 2]
valid_sum = 0
valid_count = 0
skipped = 0
idx = 0

while idx < len(data):
    value = data[idx]
    idx += 1
    
    # ข้ามค่าที่ไม่ถูกต้อง
    if value is None:
        print(f"  ข้าม None")
        skipped += 1
        continue
    
    if value <= 0:
        print(f"  ข้าม {value} (ไม่เป็นบวก)")
        skipped += 1
        continue
    
    # ประมวลผลค่าที่ถูกต้อง
    valid_sum += value
    valid_count += 1
    print(f"  นับ {value}, ผลรวมสะสม: {valid_sum}")

print(f"\nสรุป: นับ {valid_count} ค่า, ข้าม {skipped} ค่า")
print(f"ผลรวม: {valid_sum}")
if valid_count > 0:
    print(f"เฉลี่ย: {valid_sum/valid_count:.2f}")
```

---

## 4. while/else

### ความหมาย

`else` ใน while loop จะทำงานเมื่อ condition กลายเป็น False ตามปกติ (ไม่ใช่จาก break)

### ตัวอย่างที่ 9: while/else

```python
# ค้นหาตัวเลขในรายการ
def search_in_list(lst, target):
    idx = 0
    while idx < len(lst):
        if lst[idx] == target:
            print(f"พบ {target} ที่ index {idx}")
            break
        idx += 1
    else:
        print(f"ไม่พบ {target} ในรายการ")

numbers = [3, 7, 2, 8, 5, 1]
search_in_list(numbers, 8)
search_in_list(numbers, 10)
```

### ตัวอย่างที่ 10: while/else กับ retry pattern

```python
import random
random.seed(42)  # กำหนด seed เพื่อผลลัพธ์ที่แน่นอน

def connect_to_server(max_retries=3):
    """จำลองการเชื่อมต่อ server พร้อม retry"""
    attempt = 0
    
    while attempt < max_retries:
        attempt += 1
        print(f"พยายามเชื่อมต่อ... รอบที่ {attempt}")
        
        # จำลอง: 70% โอกาสล้มเหลว (เพื่อ demo)
        success = random.random() > 0.7
        
        if success:
            print(f"เชื่อมต่อสำเร็จในรอบที่ {attempt}!")
            break
        else:
            print(f"  เชื่อมต่อล้มเหลว")
            if attempt < max_retries:
                print(f"  รอ 1 วินาทีแล้วลองใหม่...")
    else:
        print(f"ไม่สามารถเชื่อมต่อได้หลังจาก {max_retries} ครั้ง")
        return False
    
    return True

result = connect_to_server()
print(f"\nผลลัพธ์: {'สำเร็จ' if result else 'ล้มเหลว'}")
```

---

## 5. Infinite Loops และวิธีหลีกเลี่ยง

### ความหมาย

Infinite loop คือ loop ที่ไม่มีวันจบ เกิดเมื่อเงื่อนไขไม่เคยเป็น False

### ตัวอย่างที่ 11: Infinite Loop และ safeguard

```python
# ❌ Infinite loop (อย่าทำ!)
# while True:
#     print("วนไม่จบ!")

# ✅ Infinite loop ที่มี exit condition
print("Infinite loop ที่ปลอดภัย:")
max_iterations = 5  # safeguard
iteration = 0

while True:
    iteration += 1
    print(f"  รอบที่ {iteration}")
    
    if iteration >= max_iterations:
        print("  ถึงจำนวนสูงสุดแล้ว หยุด")
        break

# ✅ แบบที่ดีกว่า: ใช้ safeguard counter
def safe_while_loop(condition_func, max_iter=1000):
    """while loop ที่มี safeguard"""
    count = 0
    while condition_func() and count < max_iter:
        # ทำงาน
        count += 1
    
    if count >= max_iter:
        print(f"คำเตือน: ถึงจำนวน iteration สูงสุด ({max_iter})")
    
    return count
```

### สาเหตุของ Infinite Loop และวิธีแก้

```python
# สาเหตุที่ 1: ลืมอัปเดตตัวแปร
# ❌
# i = 0
# while i < 5:
#     print(i)
#     # ลืม i += 1 !!!

# ✅
i = 0
while i < 5:
    print(i, end=" ")
    i += 1  # ต้องอัปเดต
print()

# สาเหตุที่ 2: condition ไม่เคยเป็น False
# ❌
# x = 1
# while x > 0:
#     x += 1  # x จะไม่มีวันเป็น 0 หรือลบ

# ✅ ต้องเขียน condition ที่สุดท้ายจะเป็น False
x = 10
while x > 0:
    print(x, end=" ")
    x -= 2  # ลดค่า
print()

# สาเหตุที่ 3: เงื่อนไขที่ขึ้นกับ side effect
# ✅ ใช้ counter เพื่อป้องกัน
items = []
counter = 0
max_count = 5

while len(items) < 10 and counter < max_count:
    items.append(counter)
    counter += 1
    print(f"items: {items}")
```

---

## 6. do-while Pattern ใน Python

### ความหมาย

Python ไม่มี `do-while` loop โดยตรง (ต่างจาก C/Java) แต่เราสามารถจำลองได้

### ตัวอย่างที่ 12: do-while pattern

```python
# ใน C/Java:
# do {
#     statement
# } while (condition);

# Python Pattern 1: ใช้ while True + break
print("Pattern 1: while True + break")
count = 0
while True:
    count += 1
    print(f"  ทำงานรอบที่ {count}")
    if count >= 3:  # เงื่อนไขหยุด
        break

print()

# Python Pattern 2: ทำงานก่อน loop ครั้งแรก
print("Pattern 2: ทำงานก่อน แล้ว while")
result = ""
# ทำงานครั้งแรก
result = "first run"
print(f"  First run: {result}")

runs = 1
while result != "stop" and runs < 5:
    runs += 1
    result = f"run {runs}"
    print(f"  {result}")

print()

# Pattern จริงๆ: Menu loop
print("Pattern 3: Menu loop (do-while style)")
choices_log = ["1", "2", "3", "4"]  # จำลอง user input
choice_idx = 0

while True:
    print("\nเมนู:")
    print("  1. ตัวเลือก 1")
    print("  2. ตัวเลือก 2")
    print("  3. ตัวเลือก 3")
    print("  4. ออก")
    
    choice = choices_log[choice_idx]
    choice_idx += 1
    print(f"เลือก: {choice}")
    
    if choice == "1":
        print("คุณเลือก 1")
    elif choice == "2":
        print("คุณเลือก 2")
    elif choice == "3":
        print("คุณเลือก 3")
    elif choice == "4":
        print("ออกจากโปรแกรม")
        break
    else:
        print("ตัวเลือกไม่ถูกต้อง")
    
    if choice_idx >= len(choices_log):
        break
```

---

## 7. while กับ User Input

### ตัวอย่างที่ 13: รับและ validate input

```python
# จำลองการรับ input
def simulate_input(prompts_responses):
    """Simulate user input สำหรับ demo"""
    idx = [0]
    def get_input(prompt=""):
        print(f"{prompt}{prompts_responses[idx[0]]}")
        response = prompts_responses[idx[0]]
        idx[0] += 1
        return response
    return get_input

# จำลอง: ผู้ใช้ใส่ข้อมูลผิดก่อน แล้วถูกในที่สุด
def get_valid_age():
    """รับอายุที่ถูกต้อง"""
    user_inputs = ["abc", "-5", "150", "25"]  # จำลอง
    fake_input = simulate_input(user_inputs)
    
    while True:
        raw_input = fake_input("กรุณาใส่อายุ (1-120): ")
        
        # ตรวจสอบว่าเป็นตัวเลข
        if not raw_input.isdigit():
            print("  ❌ กรุณาใส่ตัวเลขเท่านั้น")
            continue
        
        age = int(raw_input)
        
        # ตรวจสอบช่วง
        if age < 1:
            print("  ❌ อายุต้องมากกว่า 0")
            continue
        
        if age > 120:
            print("  ❌ อายุต้องไม่เกิน 120 ปี")
            continue
        
        # ถูกต้อง
        print(f"  ✓ อายุ {age} ปี ถูกต้อง!")
        return age

age = get_valid_age()
print(f"\nอายุที่ได้รับ: {age}")
```

### ตัวอย่างที่ 14: การ validate หลายเงื่อนไข

```python
def validate_username():
    """ตรวจสอบ username ที่ถูกต้อง"""
    rules = [
        (lambda s: len(s) >= 3, "ต้องมีความยาวอย่างน้อย 3 ตัวอักษร"),
        (lambda s: len(s) <= 20, "ต้องมีความยาวไม่เกิน 20 ตัวอักษร"),
        (lambda s: s[0].isalpha(), "ต้องเริ่มต้นด้วยตัวอักษร"),
        (lambda s: all(c.isalnum() or c == '_' for c in s), "ใช้ได้แค่ a-z, 0-9, _"),
    ]
    
    # จำลอง user input
    test_inputs = ["ab", "123abc", "valid_user", "user name", "valid123"]
    
    for test in test_inputs:
        print(f"\nทดสอบ username: '{test}'")
        is_valid = True
        for check, message in rules:
            if not check(test):
                print(f"  ❌ {message}")
                is_valid = False
                break
        
        if is_valid:
            print(f"  ✓ username ถูกต้อง!")

validate_username()
```

---

## 8. Counter Patterns

### ตัวอย่างที่ 15: Counter patterns ต่างๆ

```python
# Pattern 1: Simple counter
print("=== Simple Counter ===")
count = 0
items = [True, False, True, True, False, True]
for item in items:
    if item:
        count += 1
print(f"จำนวน True: {count}")

# Pattern 2: Down counter
print("\n=== Down Counter ===")
countdown = 5
while countdown > 0:
    print(f"  {countdown}...", end="")
    countdown -= 1
print("เปิดตัว!")

# Pattern 3: Step counter
print("\n=== Step Counter ===")
total_distance = 1000
step_size = 100
current = 0

while current < total_distance:
    current += step_size
    progress = current / total_distance * 100
    bar = "█" * int(progress // 10)
    print(f"  {current:4}m [{bar:<10}] {progress:.0f}%")

print("ถึงเป้าหมายแล้ว!")

# Pattern 4: Attempt counter
print("\n=== Attempt Counter ===")
MAX_ATTEMPTS = 3
correct_answer = 42
# จำลอง: ผู้ใช้ตอบผิด 2 ครั้งแล้วถูก
user_answers = [10, 25, 42]
attempt = 0

while attempt < MAX_ATTEMPTS:
    guess = user_answers[attempt]
    attempt += 1
    print(f"  ครั้งที่ {attempt}: ตอบ {guess}", end="")
    
    if guess == correct_answer:
        print(" ✓ ถูกต้อง!")
        break
    else:
        remaining = MAX_ATTEMPTS - attempt
        if remaining > 0:
            print(f" ✗ ผิด (เหลือ {remaining} ครั้ง)")
        else:
            print(f" ✗ ผิด")
else:
    print(f"  หมดจำนวนครั้งแล้ว คำตอบที่ถูกต้องคือ {correct_answer}")
```

### ตัวอย่างที่ 16: Accumulator pattern

```python
# สะสมค่าในขณะ loop
print("=== Accumulator Pattern ===")

# Running total
sales = [150, 230, 180, 420, 310, 290, 380]
running_total = 0
running_avg = 0

print(f"{'วัน':>4} {'ยอดขาย':>8} {'สะสม':>10} {'เฉลี่ย':>8}")
print("-" * 35)

for day, sale in enumerate(sales, 1):
    running_total += sale
    running_avg = running_total / day
    print(f"{day:>4} {sale:>8,} {running_total:>10,} {running_avg:>8.1f}")

print("-" * 35)
print(f"{'รวม':>4} {'':>8} {running_total:>10,} {running_avg:>8.1f}")
```

---

## 9. Flag Variables

### ความหมาย

Flag variable คือตัวแปร boolean ที่ใช้ควบคุมการทำงานของ loop หรือ program state

### ตัวอย่างที่ 17: Flag variable พื้นฐาน

```python
# Flag variable ควบคุม loop
print("=== Flag Variable ===")

found = False
search_value = 7
data = [3, 5, 8, 7, 2, 9, 1]
index = 0

while not found and index < len(data):
    if data[index] == search_value:
        found = True
    else:
        index += 1

if found:
    print(f"พบ {search_value} ที่ index {index}")
else:
    print(f"ไม่พบ {search_value}")
```

### ตัวอย่างที่ 18: หลาย Flag variables

```python
# จำลองสถานะเครื่องจักร
print("=== Machine State Flags ===")

# สถานะ
is_running = False
has_error = False
is_paused = False

# Commands
commands = ["start", "run", "pause", "resume", "stop"]

for cmd in commands:
    print(f"\nคำสั่ง: {cmd}")
    
    if cmd == "start":
        if not is_running:
            is_running = True
            has_error = False
            print("  เครื่องเริ่มทำงาน")
        else:
            print("  เครื่องทำงานอยู่แล้ว")
    
    elif cmd == "stop":
        if is_running:
            is_running = False
            is_paused = False
            print("  เครื่องหยุดทำงาน")
        else:
            print("  เครื่องไม่ได้ทำงาน")
    
    elif cmd == "pause":
        if is_running and not is_paused:
            is_paused = True
            print("  เครื่องหยุดชั่วคราว")
        elif is_paused:
            print("  เครื่องหยุดชั่วคราวอยู่แล้ว")
        else:
            print("  เครื่องไม่ได้ทำงาน")
    
    elif cmd == "resume":
        if is_paused:
            is_paused = False
            print("  เครื่องทำงานต่อ")
        else:
            print("  เครื่องไม่ได้หยุดชั่วคราว")
    
    elif cmd == "run":
        if is_running and not is_paused:
            print("  เครื่องกำลังทำงาน...")
        elif is_paused:
            print("  เครื่องหยุดชั่วคราว ไม่สามารถทำงานได้")
        else:
            print("  เครื่องไม่ได้เปิด")
    
    # แสดงสถานะ
    status = []
    if is_running: status.append("Running")
    if is_paused: status.append("Paused")
    if has_error: status.append("Error")
    print(f"  สถานะ: {', '.join(status) if status else 'Stopped'}")
```

---

## 10. ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Guess the Number Game

```python
import random

def guess_number_game():
    """เกมทายตัวเลข"""
    print("=" * 40)
    print("เกมทายตัวเลข")
    print("=" * 40)
    
    secret_number = random.randint(1, 100)
    max_attempts = 7
    attempts = 0
    guessed = False
    
    print(f"คิดตัวเลขระหว่าง 1-100 ไว้แล้ว")
    print(f"คุณมี {max_attempts} ครั้ง")
    
    # จำลอง player guesses
    # ในโปรแกรมจริงใช้ input()
    player_guesses = [50, 75, 62, 56, secret_number]
    guess_idx = 0
    
    while attempts < max_attempts and not guessed:
        attempts += 1
        guess = player_guesses[guess_idx]
        guess_idx = min(guess_idx + 1, len(player_guesses) - 1)
        
        print(f"\nครั้งที่ {attempts}/{max_attempts}: ทาย {guess}")
        
        if guess < secret_number:
            diff = secret_number - guess
            if diff > 20:
                hint = "น้อยกว่ามาก"
            elif diff > 10:
                hint = "น้อยกว่าพอสมควร"
            else:
                hint = "น้อยกว่านิดหน่อย"
            print(f"  ❌ น้อยเกินไป ({hint})")
        
        elif guess > secret_number:
            diff = guess - secret_number
            if diff > 20:
                hint = "มากกว่ามาก"
            elif diff > 10:
                hint = "มากกว่าพอสมควร"
            else:
                hint = "มากกว่านิดหน่อย"
            print(f"  ❌ มากเกินไป ({hint})")
        
        else:
            guessed = True
            print(f"  🎉 ถูกต้อง! ตัวเลขคือ {secret_number}")
    
    if guessed:
        if attempts <= 3:
            rating = "ยอดเยี่ยม! 🌟"
        elif attempts <= 5:
            rating = "ดีมาก! 👍"
        else:
            rating = "ผ่านไป 😊"
        print(f"\nคุณทายถูกใน {attempts} ครั้ง - {rating}")
    else:
        print(f"\nหมดจำนวนครั้งแล้ว! ตัวเลขคือ {secret_number}")

guess_number_game()
```

### โปรแกรมที่ 2: Menu-Driven Program

```python
def library_system():
    """ระบบจัดการห้องสมุดแบบ Menu-Driven"""
    
    books = [
        {"id": 1, "title": "Python Basics", "author": "John Smith", "available": True},
        {"id": 2, "title": "Data Science", "author": "Jane Doe", "available": True},
        {"id": 3, "title": "Web Development", "author": "Bob Johnson", "available": False},
        {"id": 4, "title": "Machine Learning", "author": "Alice Brown", "available": True},
    ]
    
    borrowed_by = {}
    
    def show_all_books():
        print("\n=== รายการหนังสือทั้งหมด ===")
        print(f"{'ID':>4} {'ชื่อหนังสือ':<25} {'ผู้แต่ง':<20} {'สถานะ'}")
        print("-" * 65)
        for book in books:
            status = "✓ ว่าง" if book["available"] else "✗ ถูกยืม"
            print(f"{book['id']:>4} {book['title']:<25} {book['author']:<20} {status}")
    
    def borrow_book(book_id, member_name):
        for book in books:
            if book["id"] == book_id:
                if book["available"]:
                    book["available"] = False
                    borrowed_by[book_id] = member_name
                    print(f"✓ ยืม '{book['title']}' สำเร็จ")
                    return True
                else:
                    print(f"✗ หนังสือ '{book['title']}' ถูกยืมไปแล้ว")
                    return False
        print(f"✗ ไม่พบหนังสือ ID: {book_id}")
        return False
    
    def return_book(book_id):
        for book in books:
            if book["id"] == book_id:
                if not book["available"]:
                    book["available"] = True
                    borrower = borrowed_by.pop(book_id, "ไม่ทราบชื่อ")
                    print(f"✓ คืน '{book['title']}' สำเร็จ (ยืมโดย: {borrower})")
                    return True
                else:
                    print(f"✗ หนังสือ '{book['title']}' ไม่ได้ถูกยืม")
                    return False
        print(f"✗ ไม่พบหนังสือ ID: {book_id}")
        return False
    
    # จำลองการใช้งาน
    print("=" * 60)
    print("ระบบจัดการห้องสมุด")
    print("=" * 60)
    
    # Simulate actions
    actions = [
        ("show", None, None),
        ("borrow", 1, "Alice"),
        ("borrow", 3, "Bob"),
        ("borrow", 1, "Charlie"),  # ยืมไปแล้ว
        ("show", None, None),
        ("return", 1, None),
        ("show", None, None),
    ]
    
    for action, book_id, member in actions:
        print(f"\n>>> Action: {action}" + (f" book_id={book_id}" if book_id else "") + (f" member={member}" if member else ""))
        
        if action == "show":
            show_all_books()
        elif action == "borrow":
            borrow_book(book_id, member)
        elif action == "return":
            return_book(book_id)

library_system()
```

### โปรแกรมที่ 3: ATM Simulation

```python
def atm_simulation():
    """จำลองระบบ ATM"""
    
    # ข้อมูล accounts
    accounts = {
        "1234567890": {
            "pin": "1234",
            "name": "สมชาย ใจดี",
            "balance": 15000.00,
            "pin_attempts": 0
        },
        "0987654321": {
            "pin": "5678",
            "name": "สมหญิง รักดี",
            "balance": 28500.00,
            "pin_attempts": 0
        }
    }
    
    MAX_PIN_ATTEMPTS = 3
    
    def get_account(card_number):
        return accounts.get(card_number)
    
    def verify_pin(account, pin):
        if account["pin"] == pin:
            account["pin_attempts"] = 0
            return True
        account["pin_attempts"] += 1
        return False
    
    def is_locked(account):
        return account["pin_attempts"] >= MAX_PIN_ATTEMPTS
    
    def show_balance(account):
        print(f"\n  ยอดเงินคงเหลือ: {account['balance']:,.2f} บาท")
    
    def withdraw(account, amount):
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        if amount % 100 != 0:
            return False, "จำนวนเงินต้องเป็นจำนวนเต็มร้อย"
        if amount > account['balance']:
            return False, f"ยอดเงินไม่เพียงพอ (มี {account['balance']:,.2f} บาท)"
        if amount > 20000:
            return False, "ถอนได้ไม่เกิน 20,000 บาทต่อครั้ง"
        
        account['balance'] -= amount
        return True, f"ถอนสำเร็จ {amount:,.2f} บาท"
    
    def deposit(account, amount):
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        account['balance'] += amount
        return True, f"ฝากสำเร็จ {amount:,.2f} บาท"
    
    # จำลองการใช้งาน
    print("=" * 50)
    print("ระบบ ATM (จำลอง)")
    print("=" * 50)
    
    # Session 1: Login ผิด 2 ครั้งแล้วถูก
    print("\n--- Session 1: สมชาย ---")
    card_num = "1234567890"
    account = get_account(card_num)
    
    if account:
        print(f"สวัสดีคุณ {account['name']}")
        
        pin_tries = ["9999", "0000", "1234"]
        for pin in pin_tries:
            if is_locked(account):
                print("บัตรถูกอายัด! กรุณาติดต่อธนาคาร")
                break
            
            print(f"ใส่ PIN: {pin}")
            if verify_pin(account, pin):
                print("PIN ถูกต้อง!")
                
                # Menu loop
                show_balance(account)
                
                operations = [
                    ("withdraw", 5000),
                    ("deposit", 2000),
                    ("balance", 0),
                    ("withdraw", 25000),  # เกิน limit
                    ("exit", 0),
                ]
                
                for op, amount in operations:
                    print(f"\n  ดำเนินการ: {op}" + (f" {amount:,}" if amount else ""))
                    
                    if op == "balance":
                        show_balance(account)
                    elif op == "withdraw":
                        success, message = withdraw(account, amount)
                        print(f"  {'✓' if success else '✗'} {message}")
                        if success:
                            show_balance(account)
                    elif op == "deposit":
                        success, message = deposit(account, amount)
                        print(f"  {'✓' if success else '✗'} {message}")
                        if success:
                            show_balance(account)
                    elif op == "exit":
                        print("  ขอบคุณที่ใช้บริการ")
                        break
                break
            else:
                remaining = MAX_PIN_ATTEMPTS - account["pin_attempts"]
                if remaining > 0:
                    print(f"  PIN ผิด! เหลือ {remaining} ครั้ง")
                else:
                    print("  PIN ผิดครบ 3 ครั้ง!")

atm_simulation()
```

---

## 11. แบบฝึกหัด

### แบบฝึกหัดข้อที่ 1: Fibonacci Sequence

```
จงเขียนโปรแกรมสร้าง Fibonacci sequence ด้วย while loop จนถึงจำนวน n
```

**เฉลย:**

```python
def fibonacci(n):
    """สร้าง Fibonacci sequence n ตัว"""
    if n <= 0:
        return []
    if n == 1:
        return [0]
    
    sequence = [0, 1]
    while len(sequence) < n:
        next_val = sequence[-1] + sequence[-2]
        sequence.append(next_val)
    
    return sequence[:n]

# ทดสอบ
for n in [1, 5, 10, 15]:
    fib = fibonacci(n)
    print(f"Fibonacci({n}): {fib}")
    print(f"  ผลรวม: {sum(fib):,}")
    print()

# แสดงแบบ visual
fib_10 = fibonacci(10)
max_val = max(fib_10)
print("Fibonacci 10 แบบ bar chart:")
for i, val in enumerate(fib_10):
    bar = "█" * int(val / max_val * 20) if max_val > 0 else ""
    print(f"  F({i:2}): {val:5} {bar}")
```

---

### แบบฝึกหัดข้อที่ 2: Collatz Conjecture

```
Collatz Conjecture: เริ่มจากตัวเลขใดๆ
- ถ้าเป็นเลขคู่ หารด้วย 2
- ถ้าเป็นเลขคี่ คูณด้วย 3 แล้วบวก 1
- ทำซ้ำจนกว่าจะเท่ากับ 1
จงหาว่าแต่ละตัวเลขต้องใช้กี่ขั้นตอน
```

**เฉลย:**

```python
def collatz(n):
    """Collatz Conjecture"""
    if n <= 0:
        return None, None
    
    sequence = [n]
    steps = 0
    
    while n != 1:
        if n % 2 == 0:
            n = n // 2
        else:
            n = 3 * n + 1
        sequence.append(n)
        steps += 1
    
    return steps, sequence

# ทดสอบ
print("Collatz Conjecture:")
print(f"{'n':>4} {'ขั้นตอน':>8} {'ค่าสูงสุด':>12}")
print("-" * 30)

for start in [1, 6, 11, 27, 100]:
    steps, seq = collatz(start)
    max_val = max(seq)
    print(f"{start:>4} {steps:>8} {max_val:>12,}")

# หา n ที่มีขั้นตอนมากที่สุดระหว่าง 1-100
print("\nหา n ที่มีขั้นตอนมากที่สุดระหว่าง 1-100:")
max_steps = 0
max_n = 0

n = 1
while n <= 100:
    steps, _ = collatz(n)
    if steps > max_steps:
        max_steps = steps
        max_n = n
    n += 1

print(f"n = {max_n} ใช้ {max_steps} ขั้นตอน")
```

---

### แบบฝึกหัดข้อที่ 3: Bank Account Simulation

```
จงเขียนระบบบัญชีธนาคารอย่างง่าย:
- ฝาก/ถอนเงิน
- แสดง statement
- ตรวจสอบยอดคงเหลือ
ใช้ while loop สำหรับ menu
```

**เฉลย:**

```python
def bank_account():
    """ระบบบัญชีธนาคารอย่างง่าย"""
    balance = 10000.0
    transactions = []
    
    def deposit_money(amount):
        nonlocal balance
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        balance += amount
        transactions.append({"type": "ฝาก", "amount": amount, "balance": balance})
        return True, f"ฝาก {amount:,.2f} บาท สำเร็จ"
    
    def withdraw_money(amount):
        nonlocal balance
        if amount <= 0:
            return False, "จำนวนเงินต้องมากกว่า 0"
        if amount > balance:
            return False, "ยอดเงินไม่เพียงพอ"
        balance -= amount
        transactions.append({"type": "ถอน", "amount": amount, "balance": balance})
        return True, f"ถอน {amount:,.2f} บาท สำเร็จ"
    
    def show_statement():
        print("\n=== Statement ===")
        print(f"{'ประเภท':<8} {'จำนวน':>12} {'คงเหลือ':>12}")
        print("-" * 35)
        for t in transactions:
            sign = "+" if t["type"] == "ฝาก" else "-"
            print(f"{t['type']:<8} {sign}{t['amount']:>11,.2f} {t['balance']:>12,.2f}")
        print("-" * 35)
        print(f"{'ยอดคงเหลือ':>22} {balance:>12,.2f}")
    
    # Simulate operations
    operations = [
        ("deposit", 5000),
        ("withdraw", 2000),
        ("withdraw", 500),
        ("deposit", 1000),
        ("withdraw", 20000),  # เกินยอด
        ("statement", 0),
    ]
    
    print(f"ยอดเริ่มต้น: {balance:,.2f} บาท")
    
    for op, amount in operations:
        if op == "deposit":
            success, msg = deposit_money(amount)
            print(f"\n{'✓' if success else '✗'} {msg}")
            print(f"  ยอดคงเหลือ: {balance:,.2f} บาท")
        elif op == "withdraw":
            success, msg = withdraw_money(amount)
            print(f"\n{'✓' if success else '✗'} {msg}")
            print(f"  ยอดคงเหลือ: {balance:,.2f} บาท")
        elif op == "statement":
            show_statement()

bank_account()
```

---

### แบบฝึกหัดข้อที่ 4: Number Base Converter

```
จงเขียนโปรแกรมแปลงเลขฐาน 10 เป็นฐาน 2, 8, 16
โดยใช้ while loop ในการแปลง
```

**เฉลย:**

```python
def convert_base(n, base):
    """แปลงเลขฐาน 10 เป็นฐาน base"""
    if n == 0:
        return "0"
    
    digits = "0123456789ABCDEF"
    result = []
    negative = n < 0
    n = abs(n)
    
    while n > 0:
        remainder = n % base
        result.append(digits[remainder])
        n //= base
    
    if negative:
        result.append("-")
    
    return "".join(reversed(result))

# ทดสอบ
numbers = [0, 1, 8, 10, 16, 42, 100, 255, 1024]

print(f"{'ฐาน 10':>8} {'ฐาน 2':>18} {'ฐาน 8':>8} {'ฐาน 16':>8}")
print("-" * 50)

for num in numbers:
    bin_str = convert_base(num, 2)
    oct_str = convert_base(num, 8)
    hex_str = convert_base(num, 16)
    
    # ยืนยันด้วย Python built-in
    assert bin_str == bin(num)[2:].upper() or num == 0
    
    print(f"{num:>8} {bin_str:>18} {oct_str:>8} {hex_str:>8}")
```

---

### แบบฝึกหัดข้อที่ 5: Digital Root

```
Digital Root คือการบวกหลักของตัวเลขซ้ำๆ จนกว่าจะเหลือหลักเดียว
เช่น 9875 -> 9+8+7+5=29 -> 2+9=11 -> 1+1=2
จงเขียนโปรแกรมหา Digital Root
```

**เฉลย:**

```python
def digital_root(n):
    """คำนวณ Digital Root"""
    n = abs(int(n))
    steps = []
    
    while n >= 10:
        digits_sum = sum(int(d) for d in str(n))
        steps.append(f"{n} -> {'+'.join(str(d) for d in str(n))} = {digits_sum}")
        n = digits_sum
    
    return n, steps

# ทดสอบ
test_numbers = [0, 5, 9875, 12345, 999, 100, 9999999]

for num in test_numbers:
    root, steps = digital_root(num)
    print(f"\nDigital Root ของ {num}:")
    for step in steps:
        print(f"  {step}")
    print(f"  ผลลัพธ์: {root}")
```

---

### แบบฝึกหัดข้อที่ 6: Password Generator

```
จงเขียนโปรแกรม generate รหัสผ่านที่มีความแข็งแกร่ง
- กำหนดความยาว
- ต้องมีตัวพิมพ์ใหญ่ ตัวพิมพ์เล็ก ตัวเลข สัญลักษณ์
- ใช้ while loop จนได้รหัสผ่านที่ตรงตามเงื่อนไข
```

**เฉลย:**

```python
import random
import string

def generate_password(length=12):
    """Generate รหัสผ่านที่แข็งแกร่ง"""
    
    if length < 8:
        length = 8
    
    # ตัวอักษรที่ใช้ได้
    uppercase = string.ascii_uppercase
    lowercase = string.ascii_lowercase
    digits = string.digits
    special = "!@#$%^&*"
    all_chars = uppercase + lowercase + digits + special
    
    # Generate จนได้รหัสผ่านที่ตรงตามเงื่อนไข
    attempts = 0
    
    while True:
        attempts += 1
        password = "".join(random.choices(all_chars, k=length))
        
        has_upper = any(c in uppercase for c in password)
        has_lower = any(c in lowercase for c in password)
        has_digit = any(c in digits for c in password)
        has_special = any(c in special for c in password)
        
        if has_upper and has_lower and has_digit and has_special:
            break
    
    return password, attempts

random.seed(42)
print("Generated Passwords:")
for length in [8, 12, 16, 20]:
    pwd, attempts = generate_password(length)
    print(f"  ความยาว {length:2d}: {pwd} (ลอง {attempts} ครั้ง)")
```

---

### แบบฝึกหัดข้อที่ 7: Roman Numerals

```
จงเขียนโปรแกรมแปลงตัวเลขอารบิคเป็นเลขโรมัน
โดยใช้ while loop
```

**เฉลย:**

```python
def to_roman(number):
    """แปลงตัวเลขอารบิคเป็นเลขโรมัน"""
    if not 1 <= number <= 3999:
        return "ไม่รองรับ (ต้องอยู่ระหว่าง 1-3999)"
    
    roman_values = [
        (1000, "M"), (900, "CM"), (500, "D"), (400, "CD"),
        (100, "C"),  (90, "XC"),  (50, "L"),  (40, "XL"),
        (10, "X"),   (9, "IX"),   (5, "V"),   (4, "IV"),
        (1, "I")
    ]
    
    result = ""
    remaining = number
    
    while remaining > 0:
        for value, symbol in roman_values:
            while remaining >= value:
                result += symbol
                remaining -= value
    
    return result

# ทดสอบ
test_numbers = [1, 4, 9, 14, 40, 49, 90, 399, 1000, 1994, 2024, 3999]
print(f"{'อารบิค':>8} {'โรมัน'}")
print("-" * 25)
for num in test_numbers:
    print(f"{num:>8} {to_roman(num)}")
```

---

### แบบฝึกหัดข้อที่ 8: Text Statistics

```
จงเขียนโปรแกรมวิเคราะห์ข้อความ:
- จำนวนคำ
- จำนวนประโยค
- จำนวนตัวอักษร (ไม่รวมช่องว่าง)
- คำยาวที่สุด
- ประโยคยาวที่สุด
```

**เฉลย:**

```python
def text_statistics(text):
    """วิเคราะห์สถิติข้อความ"""
    
    # จำนวนตัวอักษร
    char_count = len(text)
    char_no_space = len(text.replace(" ", ""))
    
    # จำนวนคำ
    words = text.split()
    word_count = len(words)
    
    # จำนวนประโยค (นับจาก . ! ?)
    sentences = []
    current = []
    
    i = 0
    while i < len(text):
        char = text[i]
        current.append(char)
        if char in ".!?":
            sentence = "".join(current).strip()
            if sentence:
                sentences.append(sentence)
            current = []
        i += 1
    
    if current:  # ส่วนที่เหลือ
        remainder = "".join(current).strip()
        if remainder:
            sentences.append(remainder)
    
    # คำยาวที่สุด
    if words:
        longest_word = max(words, key=len)
        # กำจัด punctuation
        longest_word_clean = "".join(c for c in longest_word if c.isalpha())
    else:
        longest_word_clean = ""
    
    # ประโยคยาวที่สุด
    if sentences:
        longest_sentence = max(sentences, key=len)
    else:
        longest_sentence = ""
    
    print("=== Text Statistics ===")
    print(f"ข้อความ: {text[:50]}...")
    print(f"\nจำนวนตัวอักษรทั้งหมด: {char_count:,}")
    print(f"จำนวนตัวอักษร (ไม่รวมช่องว่าง): {char_no_space:,}")
    print(f"จำนวนคำ: {word_count:,}")
    print(f"จำนวนประโยค: {len(sentences):,}")
    print(f"\nคำยาวที่สุด: '{longest_word_clean}' ({len(longest_word_clean)} ตัวอักษร)")
    if longest_sentence:
        print(f"ประโยคยาวที่สุด ({len(longest_sentence)} ตัวอักษร):")
        print(f"  '{longest_sentence[:60]}...'")

sample = """Python is a high-level, general-purpose programming language. 
Its design philosophy emphasizes code readability. 
Python was created by Guido van Rossum and was first released in 1991. 
Python consistently ranks as one of the most popular programming languages!"""

text_statistics(sample)
```

---

### แบบฝึกหัดข้อที่ 9: Simple Calculator with History

```
จงเขียนเครื่องคิดเลขที่เก็บประวัติการคำนวณ
ใช้ while loop สำหรับรับ input ต่อเนื่อง
```

**เฉลย:**

```python
def calculator_with_history():
    """เครื่องคิดเลขพร้อมประวัติ"""
    
    history = []
    current_result = 0
    
    def calculate(a, op, b):
        if op == "+": return a + b
        elif op == "-": return a - b
        elif op == "*": return a * b
        elif op == "/":
            if b == 0: raise ValueError("หารด้วยศูนย์ไม่ได้")
            return a / b
        elif op == "**": return a ** b
        elif op == "%":
            if b == 0: raise ValueError("หารด้วยศูนย์ไม่ได้")
            return a % b
        raise ValueError(f"ไม่รู้จักตัวดำเนินการ: {op}")
    
    # จำลอง expressions
    expressions = [
        (10, "+", 5),
        (15, "-", 3),
        (4, "*", 7),
        (20, "/", 4),
        (2, "**", 8),
        (17, "%", 5),
        (10, "/", 0),  # Error case
    ]
    
    print("=== เครื่องคิดเลข ===")
    
    for a, op, b in expressions:
        print(f"\n{a} {op} {b} = ", end="")
        
        try:
            result = calculate(a, op, b)
            current_result = result
            history.append(f"{a} {op} {b} = {result}")
            print(f"{result}")
        except ValueError as e:
            print(f"Error: {e}")
            history.append(f"{a} {op} {b} = Error: {e}")
    
    print("\n=== ประวัติการคำนวณ ===")
    for i, record in enumerate(history, 1):
        print(f"  {i:2}. {record}")

calculator_with_history()
```

---

### แบบฝึกหัดข้อที่ 10: Inventory Management

```
จงเขียนระบบจัดการสินค้าคงคลังอย่างง่าย:
- เพิ่ม/ลดสินค้า
- ตรวจสอบสินค้าที่ต่ำกว่า threshold
- แสดงรายงาน
```

**เฉลย:**

```python
def inventory_system():
    """ระบบจัดการสินค้าคงคลัง"""
    
    inventory = {
        "Apple": {"qty": 100, "min_qty": 20, "price": 25},
        "Banana": {"qty": 50, "min_qty": 15, "price": 15},
        "Cherry": {"qty": 30, "min_qty": 25, "price": 50},
        "Date": {"qty": 15, "min_qty": 10, "price": 40},
        "Elderberry": {"qty": 8, "min_qty": 10, "price": 80},  # ต่ำกว่า min
    }
    
    def add_stock(item, qty):
        if item not in inventory:
            inventory[item] = {"qty": 0, "min_qty": 10, "price": 0}
        inventory[item]["qty"] += qty
        return f"เพิ่ม {item} อีก {qty} ชิ้น (รวม: {inventory[item]['qty']})"
    
    def sell_stock(item, qty):
        if item not in inventory:
            return False, f"ไม่พบสินค้า: {item}"
        if inventory[item]["qty"] < qty:
            return False, f"สินค้าไม่เพียงพอ (มี: {inventory[item]['qty']})"
        inventory[item]["qty"] -= qty
        return True, f"ขาย {item} {qty} ชิ้น (คงเหลือ: {inventory[item]['qty']})"
    
    def check_low_stock():
        low_items = []
        for item, data in inventory.items():
            if data["qty"] < data["min_qty"]:
                shortage = data["min_qty"] - data["qty"]
                low_items.append((item, data["qty"], data["min_qty"], shortage))
        return low_items
    
    def show_report():
        print("\n=== รายงานสินค้าคงคลัง ===")
        print(f"{'สินค้า':<15} {'จำนวน':>8} {'ขั้นต่ำ':>8} {'มูลค่า':>10} {'สถานะ'}")
        print("-" * 55)
        
        total_value = 0
        for item, data in sorted(inventory.items()):
            value = data["qty"] * data["price"]
            total_value += value
            status = "⚠️ ต่ำ" if data["qty"] < data["min_qty"] else "✓ ปกติ"
            print(f"{item:<15} {data['qty']:>8} {data['min_qty']:>8} {value:>10,} {status}")
        
        print("-" * 55)
        print(f"{'มูลค่าสินค้ารวม':>35} {total_value:>10,} บาท")
    
    # ดำเนินการ
    print("=== ระบบจัดการสินค้าคงคลัง ===")
    
    show_report()
    
    # ตรวจสอบสินค้าต่ำ
    low = check_low_stock()
    if low:
        print("\n⚠️ สินค้าที่ต้องสั่งซื้อเพิ่ม:")
        for item, qty, min_qty, shortage in low:
            print(f"  - {item}: มี {qty}, ขั้นต่ำ {min_qty} (ขาด {shortage})")
    
    # ดำเนินการขายและรับสินค้า
    operations = [
        ("sell", "Apple", 30),
        ("sell", "Cherry", 10),
        ("sell", "Date", 20),   # เกินที่มี
        ("add", "Elderberry", 15),
        ("add", "Cherry", 20),
    ]
    
    print("\n=== การดำเนินการ ===")
    for op, item, qty in operations:
        if op == "sell":
            success, msg = sell_stock(item, qty)
            print(f"{'✓' if success else '✗'} {msg}")
        elif op == "add":
            msg = add_stock(item, qty)
            print(f"✓ {msg}")
    
    show_report()
    
    low = check_low_stock()
    if low:
        print("\n⚠️ สินค้าที่ต้องสั่งซื้อเพิ่ม:")
        for item, qty, min_qty, shortage in low:
            print(f"  - {item}: มี {qty}, ขั้นต่ำ {min_qty} (ขาด {shortage})")
    else:
        print("\n✓ สินค้าทุกรายการอยู่ในระดับปกติ")

inventory_system()
```

---

## สรุป Part 08

| หัวข้อ | สิ่งที่ได้เรียน |
|--------|----------------|
| while loop พื้นฐาน | วนซ้ำจนเงื่อนไขเป็น False |
| break | หยุด loop ทันที |
| continue | ข้ามรอบนั้น วนต่อ |
| while/else | else ทำงานเมื่อไม่มี break |
| Infinite loop | while True + break |
| do-while pattern | ทำงานก่อนอย่างน้อย 1 ครั้ง |
| User input | validate ด้วย while loop |
| Counter patterns | นับจำนวนการทำงาน |
| Flag variables | ควบคุมสถานะด้วย boolean |

### เปรียบเทียบ for vs while

| for loop | while loop |
|----------|------------|
| รู้จำนวนรอบล่วงหน้า | ไม่รู้จำนวนรอบ |
| วนผ่าน iterable | ทำงานจนเงื่อนไขเป็น False |
| Pythonic มากกว่า | ยืดหยุ่นกว่า |
| เหมาะกับ collection | เหมาะกับ event-driven |

### Key Takeaways:
1. ต้องอัปเดตเงื่อนไขใน while loop เสมอ มิฉะนั้นจะเป็น infinite loop
2. ใช้ **break** เพื่อออกจาก loop เมื่อพบสิ่งที่ต้องการ
3. ใช้ **continue** เพื่อข้ามรอบที่ไม่ต้องการ
4. **while/else** มีประโยชน์สำหรับ search patterns
5. ใช้ **flag variables** เพื่อควบคุม state ที่ซับซ้อน
6. เพิ่ม **safeguard counter** เพื่อป้องกัน infinite loop

---

*Part 08 จบแล้ว ไปต่อที่ [Part 09 - Functions: Basics](../part09/README.md)*
