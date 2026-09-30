# Part 16: Error Handling & Exceptions

## สารบัญ
1. [ประเภทของ Errors](#1-ประเภทของ-errors)
2. [Built-in Exceptions ทั้งหมด](#2-built-in-exceptions-ทั้งหมด)
3. [try/except Block](#3-tryexcept-block)
4. [Multiple except Clauses](#4-multiple-except-clauses)
5. [except as e](#5-except-as-e)
6. [else Clause ใน try/except](#6-else-clause-ใน-tryexcept)
7. [finally Clause](#7-finally-clause)
8. [Raising Exceptions](#8-raising-exceptions)
9. [Custom Exceptions](#9-custom-exceptions)
10. [Exception Chaining](#10-exception-chaining)
11. [Context Managers กับ Exceptions](#11-context-managers-กับ-exceptions)
12. [Best Practices](#12-best-practices)
13. [ตัวอย่างโปรแกรมจริง](#13-ตัวอย่างโปรแกรมจริง)
14. [แบบฝึกหัด](#14-แบบฝึกหัด)

---

## 1. ประเภทของ Errors

ใน Python errors แบ่งออกเป็น 3 ประเภทหลัก:

### 1.1 Syntax Error (ข้อผิดพลาดทางไวยากรณ์)

Syntax Error เกิดขึ้นเมื่อโค้ดที่เขียนไม่ถูกต้องตามกฎไวยากรณ์ของ Python Python จะตรวจพบก่อนที่โปรแกรมจะรัน

```python
# ตัวอย่าง Syntax Error
# if x = 5:   # ผิด! ต้องใช้ == ไม่ใช่ =
# print("Hello"  # ผิด! ขาด )

# ตัวอย่างที่ถูกต้อง
x = 5
if x == 5:
    print("x เท่ากับ 5")
```

```python
# Syntax Error จาก indentation ผิด
def my_function():
    x = 10
    y = 20
    return x + y  # ต้อง indent ให้ถูกต้อง

result = my_function()
print(f"ผลลัพธ์: {result}")
```

### 1.2 Runtime Error (ข้อผิดพลาดขณะรัน)

Runtime Error เกิดขึ้นขณะที่โปรแกรมกำลังทำงาน Python จะหยุดทำงานและแสดง error message

```python
# ตัวอย่าง Runtime Error - ZeroDivisionError
def divide_numbers(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        print("ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้!")
        return None

print(divide_numbers(10, 2))   # Output: 5.0
print(divide_numbers(10, 0))   # Output: ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้!
```

```python
# ตัวอย่าง Runtime Error - IndexError
my_list = [1, 2, 3]
try:
    print(my_list[10])  # Index เกินขอบเขต
except IndexError as e:
    print(f"IndexError: {e}")

# Output: IndexError: list index out of range
```

```python
# ตัวอย่าง Runtime Error - KeyError
my_dict = {"name": "Alice", "age": 25}
try:
    print(my_dict["email"])  # Key ไม่มีอยู่
except KeyError as e:
    print(f"KeyError: {e}")

# Output: KeyError: 'email'
```

### 1.3 Logic Error (ข้อผิดพลาดทางตรรกะ)

Logic Error เป็นข้อผิดพลาดที่ซับซ้อนที่สุด โปรแกรมรันได้โดยไม่มี error แต่ผลลัพธ์ไม่ถูกต้อง

```python
# ตัวอย่าง Logic Error - การคำนวณผิด
def calculate_average(numbers):
    # Bug: หารด้วย len ผิด
    total = sum(numbers)
    # ผิด: return total / len(numbers) + 1  # บวก 1 เกินมา
    return total / len(numbers)  # ถูกต้อง

numbers = [10, 20, 30, 40, 50]
avg = calculate_average(numbers)
print(f"ค่าเฉลี่ย: {avg}")  # Output: 30.0
```

```python
# Logic Error - เงื่อนไขผิด
def find_positive_numbers(numbers):
    result = []
    for num in numbers:
        if num >= 0:  # ผิด: รวม 0 ด้วย ควรใช้ > 0
            result.append(num)
    return result

# ตัวอย่างที่ถูกต้อง
def find_truly_positive_numbers(numbers):
    result = []
    for num in numbers:
        if num > 0:  # ถูกต้อง: ไม่รวม 0
            result.append(num)
    return result

nums = [-3, -1, 0, 2, 5]
print(find_positive_numbers(nums))         # [0, 2, 5] - มี 0 ด้วย
print(find_truly_positive_numbers(nums))   # [2, 5] - ถูกต้อง
```

---

## 2. Built-in Exceptions ทั้งหมด

Python มี built-in exceptions หลายประเภท ดูลำดับชั้น (hierarchy) ได้ดังนี้:

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── AttributeError
    ├── BufferError
    ├── EOFError
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── MemoryError
    ├── NameError
    │   └── UnboundLocalError
    ├── OSError
    │   ├── FileExistsError
    │   ├── FileNotFoundError
    │   ├── IsADirectoryError
    │   ├── NotADirectoryError
    │   ├── PermissionError
    │   ├── TimeoutError
    │   └── ...
    ├── ReferenceError
    ├── RuntimeError
    │   ├── NotImplementedError
    │   └── RecursionError
    ├── StopIteration
    ├── StopAsyncIteration
    ├── SyntaxError
    │   └── IndentationError
    │       └── TabError
    ├── SystemError
    ├── TypeError
    ├── ValueError
    │   └── UnicodeError
    └── Warning
        ├── DeprecationWarning
        ├── RuntimeWarning
        ├── SyntaxWarning
        ├── UserWarning
        └── ...
```

### ตัวอย่าง Built-in Exceptions ที่พบบ่อย

```python
# 1. ZeroDivisionError - หารด้วยศูนย์
try:
    result = 10 / 0
except ZeroDivisionError:
    print("ZeroDivisionError: ไม่สามารถหารด้วยศูนย์")

# 2. ValueError - ค่าผิดประเภทหรือไม่ถูกต้อง
try:
    num = int("abc")
except ValueError:
    print("ValueError: ไม่สามารถแปลง 'abc' เป็น int")

# 3. TypeError - ประเภทข้อมูลผิด
try:
    result = "5" + 5
except TypeError:
    print("TypeError: ไม่สามารถบวก string กับ int")

# 4. IndexError - index เกินขอบเขต
try:
    my_list = [1, 2, 3]
    print(my_list[5])
except IndexError:
    print("IndexError: index เกินขอบเขต")

# 5. KeyError - key ไม่มีใน dictionary
try:
    my_dict = {"a": 1}
    print(my_dict["b"])
except KeyError:
    print("KeyError: key ไม่มีใน dictionary")

# 6. AttributeError - attribute ไม่มี
try:
    x = 5
    x.append(10)
except AttributeError:
    print("AttributeError: int ไม่มี method append")

# 7. FileNotFoundError - ไฟล์ไม่พบ
try:
    with open("nonexistent.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("FileNotFoundError: ไม่พบไฟล์")

# 8. ImportError - import module ไม่ได้
try:
    import nonexistent_module
except ImportError:
    print("ImportError: ไม่สามารถ import module")

# 9. NameError - ตัวแปรยังไม่ถูกกำหนด
try:
    print(undefined_variable)
except NameError:
    print("NameError: ตัวแปรยังไม่ถูกกำหนด")

# 10. MemoryError - หน่วยความจำไม่พอ
try:
    big_list = [0] * (10**9)  # อาจทำให้เกิด MemoryError
except MemoryError:
    print("MemoryError: หน่วยความจำไม่พอ")

# 11. RecursionError - recursive มากเกินไป
try:
    def infinite_recursion():
        return infinite_recursion()
    infinite_recursion()
except RecursionError:
    print("RecursionError: recursive มากเกินไป")

# 12. OverflowError - ตัวเลขใหญ่เกินไป
try:
    import math
    result = math.exp(1000)
except OverflowError:
    print("OverflowError: ตัวเลขใหญ่เกินไป")
```

---

## 3. try/except Block

`try/except` คือโครงสร้างหลักสำหรับการจัดการ exceptions ใน Python

### รูปแบบพื้นฐาน

```python
try:
    # โค้ดที่อาจเกิด exception
    risky_code()
except ExceptionType:
    # จัดการ exception ที่เกิดขึ้น
    handle_error()
```

### ตัวอย่างการใช้งาน

```python
# ตัวอย่างที่ 1: Basic try/except
def safe_divide(a, b):
    try:
        result = a / b
        print(f"{a} / {b} = {result}")
        return result
    except ZeroDivisionError:
        print("ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้")
        return None

safe_divide(10, 2)   # 10 / 2 = 5.0
safe_divide(10, 0)   # ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้
```

```python
# ตัวอย่างที่ 2: รับ input จากผู้ใช้อย่างปลอดภัย
def get_integer_input(prompt):
    try:
        value = int(input(prompt))
        return value
    except ValueError:
        print("กรุณาป้อนตัวเลขจำนวนเต็ม!")
        return None

# จำลองการรับ input
def simulate_input(value):
    try:
        result = int(value)
        print(f"ได้รับค่า: {result}")
        return result
    except ValueError:
        print(f"'{value}' ไม่ใช่ตัวเลขจำนวนเต็ม!")
        return None

simulate_input("42")    # ได้รับค่า: 42
simulate_input("abc")   # 'abc' ไม่ใช่ตัวเลขจำนวนเต็ม!
simulate_input("3.14")  # '3.14' ไม่ใช่ตัวเลขจำนวนเต็ม!
```

```python
# ตัวอย่างที่ 3: อ่านไฟล์อย่างปลอดภัย
def read_file_safe(filename):
    try:
        with open(filename, 'r', encoding='utf-8') as f:
            content = f.read()
        print(f"อ่านไฟล์ '{filename}' สำเร็จ ({len(content)} ตัวอักษร)")
        return content
    except FileNotFoundError:
        print(f"ข้อผิดพลาด: ไม่พบไฟล์ '{filename}'")
        return None

# ทดสอบ
content = read_file_safe("existing_file.txt")
content = read_file_safe("missing_file.txt")
```

```python
# ตัวอย่างที่ 4: แปลงข้อมูล JSON อย่างปลอดภัย
import json

def parse_json_safe(json_string):
    try:
        data = json.loads(json_string)
        print(f"แปลง JSON สำเร็จ: {type(data).__name__}")
        return data
    except json.JSONDecodeError as e:
        print(f"JSON ไม่ถูกต้อง: {e}")
        return None

# ทดสอบ
valid_json = '{"name": "Alice", "age": 25}'
invalid_json = '{name: Alice}'

parse_json_safe(valid_json)    # แปลง JSON สำเร็จ: dict
parse_json_safe(invalid_json)  # JSON ไม่ถูกต้อง: ...
```

---

## 4. Multiple except Clauses

เราสามารถระบุ except หลายชนิดในหนึ่ง try block ได้

```python
# ตัวอย่างที่ 1: Multiple except clauses
def process_data(data, index):
    try:
        value = data[index]
        result = 100 / value
        return result
    except IndexError:
        print(f"IndexError: index {index} เกินขอบเขต (ขนาด: {len(data)})")
        return None
    except ZeroDivisionError:
        print(f"ZeroDivisionError: ค่าที่ index {index} เป็น 0")
        return None
    except TypeError:
        print(f"TypeError: ค่าที่ index {index} ไม่ใช่ตัวเลข")
        return None

data = [10, 0, "abc", 5]
print(process_data(data, 0))   # 10.0
print(process_data(data, 1))   # ZeroDivisionError
print(process_data(data, 2))   # TypeError
print(process_data(data, 10))  # IndexError
```

```python
# ตัวอย่างที่ 2: จับหลาย exceptions ในบรรทัดเดียว (tuple)
def convert_and_calculate(value, divisor):
    try:
        num = float(value)
        result = num / divisor
        return result
    except (ValueError, TypeError):
        print(f"ข้อผิดพลาดประเภทข้อมูล: ไม่สามารถแปลง '{value}' เป็นตัวเลข")
        return None
    except ZeroDivisionError:
        print("ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์")
        return None

print(convert_and_calculate("10", 2))    # 5.0
print(convert_and_calculate("abc", 2))  # ข้อผิดพลาดประเภทข้อมูล
print(convert_and_calculate("10", 0))   # ข้อผิดพลาด: หารด้วยศูนย์
```

```python
# ตัวอย่างที่ 3: จับ exception ทั่วไปด้วย Exception
def risky_operation(x):
    try:
        if x < 0:
            raise ValueError("ค่าต้องเป็นบวก")
        if x == 0:
            raise ZeroDivisionError("ค่าต้องไม่ใช่ศูนย์")
        return 100 / x
    except ValueError as e:
        print(f"ValueError: {e}")
        return None
    except ZeroDivisionError as e:
        print(f"ZeroDivisionError: {e}")
        return None
    except Exception as e:
        print(f"Unexpected error: {type(e).__name__}: {e}")
        return None

print(risky_operation(10))   # 10.0
print(risky_operation(0))    # ZeroDivisionError
print(risky_operation(-1))   # ValueError
```

---

## 5. except as e

การใช้ `as e` ทำให้เราเข้าถึงข้อมูลของ exception ได้

```python
# ตัวอย่างที่ 1: ดู exception message
try:
    result = int("not_a_number")
except ValueError as e:
    print(f"Error type: {type(e).__name__}")
    print(f"Error message: {e}")
    print(f"Error args: {e.args}")

# Output:
# Error type: ValueError
# Error message: invalid literal for int() with base 10: 'not_a_number'
# Error args: ("invalid literal for int() with base 10: 'not_a_number'",)
```

```python
# ตัวอย่างที่ 2: logging exception details
import traceback

def process_file(filename):
    try:
        with open(filename, 'r') as f:
            data = f.read()
            numbers = [int(x) for x in data.split()]
            return sum(numbers)
    except FileNotFoundError as e:
        print(f"[ERROR] ไม่พบไฟล์: {e.filename}")
        return None
    except ValueError as e:
        print(f"[ERROR] ข้อมูลในไฟล์ไม่ถูกต้อง: {e}")
        return None
    except Exception as e:
        print(f"[ERROR] เกิดข้อผิดพลาดที่ไม่คาดคิด: {type(e).__name__}: {e}")
        traceback.print_exc()
        return None

process_file("numbers.txt")
```

```python
# ตัวอย่างที่ 3: เก็บ exception ไว้ใช้งาน
def safe_api_call(url):
    errors = []
    
    try:
        # จำลอง API call
        if "invalid" in url:
            raise ValueError(f"URL ไม่ถูกต้อง: {url}")
        if "timeout" in url:
            raise TimeoutError("การเชื่อมต่อหมดเวลา")
        return {"status": "success", "data": "some_data"}
    except ValueError as e:
        errors.append({"type": "ValueError", "message": str(e)})
    except TimeoutError as e:
        errors.append({"type": "TimeoutError", "message": str(e)})
    except Exception as e:
        errors.append({"type": type(e).__name__, "message": str(e)})
    
    return {"status": "error", "errors": errors}

result1 = safe_api_call("https://api.example.com/data")
result2 = safe_api_call("https://invalid-url.com")
result3 = safe_api_call("https://timeout.example.com")

print(result1)
print(result2)
print(result3)
```

---

## 6. else Clause ใน try/except

`else` clause จะทำงานก็ต่อเมื่อ try block ไม่มี exception เกิดขึ้น

```python
# รูปแบบ
try:
    # โค้ดที่อาจเกิด exception
    pass
except ExceptionType:
    # จัดการ exception
    pass
else:
    # ทำงานเมื่อ try สำเร็จ (ไม่มี exception)
    pass
```

```python
# ตัวอย่างที่ 1: else clause พื้นฐาน
def read_and_process(filename):
    try:
        with open(filename, 'r') as f:
            content = f.read()
    except FileNotFoundError:
        print(f"ไม่พบไฟล์: {filename}")
    else:
        # ทำงานเฉพาะเมื่ออ่านไฟล์สำเร็จ
        word_count = len(content.split())
        print(f"ไฟล์ '{filename}' มี {word_count} คำ")
        return word_count
    return None

# ทดสอบ
# read_and_process("test.txt")   # ถ้าไฟล์มีอยู่
# read_and_process("nofile.txt") # ถ้าไฟล์ไม่มี
```

```python
# ตัวอย่างที่ 2: else สำหรับการแปลงข้อมูล
def parse_and_validate(data):
    try:
        number = float(data)
    except ValueError:
        print(f"'{data}' ไม่สามารถแปลงเป็น float ได้")
        return None
    else:
        # ทำงานเฉพาะเมื่อแปลงสำเร็จ
        if number < 0:
            print(f"คำเตือน: {number} เป็นค่าลบ")
        elif number > 1000:
            print(f"คำเตือน: {number} มีค่ามากผิดปกติ")
        else:
            print(f"ค่า {number} อยู่ในช่วงปกติ")
        return number

parse_and_validate("42.5")
parse_and_validate("abc")
parse_and_validate("-10")
parse_and_validate("5000")
```

```python
# ตัวอย่างที่ 3: ใช้ else เพื่อแยก success logic
import json

def load_config(config_file):
    try:
        with open(config_file, 'r') as f:
            config = json.load(f)
    except FileNotFoundError:
        print(f"ไม่พบไฟล์ config: {config_file}")
        return {}
    except json.JSONDecodeError as e:
        print(f"รูปแบบ JSON ผิด: {e}")
        return {}
    else:
        # ทำงานเฉพาะเมื่อโหลดสำเร็จ
        print(f"โหลด config สำเร็จ: {len(config)} settings")
        
        # Validate required keys
        required_keys = ['host', 'port', 'database']
        missing_keys = [key for key in required_keys if key not in config]
        
        if missing_keys:
            print(f"คำเตือน: ขาด keys: {missing_keys}")
        
        return config

# config = load_config("config.json")
```

---

## 7. finally Clause

`finally` clause จะทำงานเสมอ ไม่ว่าจะเกิด exception หรือไม่ก็ตาม ใช้สำหรับ cleanup

```python
# รูปแบบ
try:
    # โค้ดที่อาจเกิด exception
    pass
except ExceptionType:
    # จัดการ exception
    pass
else:
    # ทำงานเมื่อสำเร็จ
    pass
finally:
    # ทำงานเสมอ (cleanup)
    pass
```

```python
# ตัวอย่างที่ 1: finally สำหรับ cleanup
def process_file_with_cleanup(filename):
    file_handle = None
    try:
        file_handle = open(filename, 'r')
        content = file_handle.read()
        return len(content)
    except FileNotFoundError:
        print(f"ไม่พบไฟล์: {filename}")
        return -1
    finally:
        # ปิดไฟล์เสมอ ไม่ว่าจะสำเร็จหรือไม่
        if file_handle:
            file_handle.close()
            print("ปิดไฟล์แล้ว")

result = process_file_with_cleanup("test.txt")
```

```python
# ตัวอย่างที่ 2: finally กับ database connection
class MockDB:
    def __init__(self):
        self.connected = False
    
    def connect(self):
        self.connected = True
        print("เชื่อมต่อฐานข้อมูลแล้ว")
    
    def execute(self, query):
        if "DROP" in query:
            raise PermissionError("ไม่มีสิทธิ์ทำ DROP operation")
        print(f"รัน query: {query}")
        return [{"id": 1, "name": "Alice"}]
    
    def close(self):
        self.connected = False
        print("ปิดการเชื่อมต่อฐานข้อมูลแล้ว")

def query_database(query):
    db = MockDB()
    result = None
    
    try:
        db.connect()
        result = db.execute(query)
        print(f"ได้ผลลัพธ์: {len(result)} แถว")
    except PermissionError as e:
        print(f"Permission error: {e}")
    except Exception as e:
        print(f"Database error: {e}")
    finally:
        db.close()  # ปิด connection เสมอ
    
    return result

query_database("SELECT * FROM users")
query_database("DROP TABLE users")
```

```python
# ตัวอย่างที่ 3: finally กับ return
def function_with_finally():
    try:
        print("ใน try")
        return "จาก try"
    except Exception:
        print("ใน except")
        return "จาก except"
    finally:
        print("ใน finally")  # ทำงานก่อน return เสมอ
        # ถ้า return ใน finally จะ override return ก่อนหน้า
        # return "จาก finally"  # ถ้า uncomment จะได้ "จาก finally"

result = function_with_finally()
print(f"ผลลัพธ์: {result}")

# Output:
# ใน try
# ใน finally
# ผลลัพธ์: จาก try
```

---

## 8. Raising Exceptions

เราสามารถ raise exception เองได้ด้วย `raise`

```python
# ตัวอย่างที่ 1: raise exception พื้นฐาน
def validate_age(age):
    if not isinstance(age, int):
        raise TypeError(f"age ต้องเป็น int ไม่ใช่ {type(age).__name__}")
    if age < 0:
        raise ValueError(f"age ต้องเป็นบวก ไม่ใช่ {age}")
    if age > 150:
        raise ValueError(f"age {age} ไม่สมเหตุสมผล (มากกว่า 150)")
    return True

# ทดสอบ
try:
    validate_age(25)
    print("อายุ 25 ปี: ถูกต้อง")
except (TypeError, ValueError) as e:
    print(f"Error: {e}")

try:
    validate_age(-5)
except (TypeError, ValueError) as e:
    print(f"Error: {e}")

try:
    validate_age("twenty")
except (TypeError, ValueError) as e:
    print(f"Error: {e}")
```

```python
# ตัวอย่างที่ 2: re-raise exception
def process_payment(amount):
    try:
        if amount <= 0:
            raise ValueError(f"จำนวนเงินต้องมากกว่า 0")
        print(f"ประมวลผลการชำระเงิน: {amount} บาท")
        return True
    except ValueError:
        print("บันทึก log: การชำระเงินล้มเหลว")
        raise  # re-raise exception เดิม

try:
    process_payment(-100)
except ValueError as e:
    print(f"การชำระเงินล้มเหลว: {e}")
```

```python
# ตัวอย่างที่ 3: raise ใน except block
def safe_open(filename):
    try:
        f = open(filename, 'r')
        return f
    except PermissionError:
        raise  # re-raise โดยไม่แก้ไข
    except FileNotFoundError:
        # สร้างไฟล์ใหม่แทน
        print(f"ไม่พบ '{filename}' กำลังสร้างใหม่...")
        f = open(filename, 'w')
        f.close()
        return open(filename, 'r')
```

```python
# ตัวอย่างที่ 4: raise from None (suppress chaining)
def get_user(user_id):
    database = {"1": "Alice", "2": "Bob"}
    try:
        return database[str(user_id)]
    except KeyError:
        raise ValueError(f"ไม่พบผู้ใช้ ID: {user_id}") from None

try:
    user = get_user(999)
except ValueError as e:
    print(f"Error: {e}")
```

---

## 9. Custom Exceptions

เราสามารถสร้าง exception ของตัวเองได้โดย inherit จาก Exception

```python
# ตัวอย่างที่ 1: Custom exception พื้นฐาน
class ValidationError(Exception):
    """Exception สำหรับการ validate ข้อมูล"""
    pass

class AgeValidationError(ValidationError):
    """Exception สำหรับการ validate อายุ"""
    
    def __init__(self, age, message="อายุไม่ถูกต้อง"):
        self.age = age
        self.message = message
        super().__init__(f"{message}: {age}")

def validate_user_age(age):
    if age < 0:
        raise AgeValidationError(age, "อายุต้องเป็นบวก")
    if age > 120:
        raise AgeValidationError(age, "อายุมากเกินไป")
    if not isinstance(age, int):
        raise AgeValidationError(age, "อายุต้องเป็นจำนวนเต็ม")
    return True

# ทดสอบ
try:
    validate_user_age(-5)
except AgeValidationError as e:
    print(f"AgeValidationError: {e}")
    print(f"ค่าที่ส่ง: {e.age}")
    print(f"ข้อความ: {e.message}")
```

```python
# ตัวอย่างที่ 2: Custom exception hierarchy
class AppError(Exception):
    """Base exception สำหรับ application"""
    
    def __init__(self, message, code=None):
        self.message = message
        self.code = code
        super().__init__(message)

class DatabaseError(AppError):
    """Exception สำหรับ database operations"""
    
    def __init__(self, message, query=None):
        self.query = query
        super().__init__(message, code="DB_ERROR")

class AuthenticationError(AppError):
    """Exception สำหรับ authentication"""
    
    def __init__(self, message, username=None):
        self.username = username
        super().__init__(message, code="AUTH_ERROR")

class AuthorizationError(AppError):
    """Exception สำหรับ authorization"""
    
    def __init__(self, message, resource=None, required_role=None):
        self.resource = resource
        self.required_role = required_role
        super().__init__(message, code="AUTHZ_ERROR")

# ตัวอย่างการใช้งาน
def get_user_data(username, password, resource):
    # Simulate authentication
    users = {"admin": "password123", "user": "secret"}
    
    if username not in users:
        raise AuthenticationError(
            f"ไม่พบผู้ใช้ '{username}'",
            username=username
        )
    
    if users[username] != password:
        raise AuthenticationError(
            "รหัสผ่านไม่ถูกต้อง",
            username=username
        )
    
    # Simulate authorization
    admin_resources = ["/admin", "/settings"]
    if resource in admin_resources and username != "admin":
        raise AuthorizationError(
            f"ไม่มีสิทธิ์เข้าถึง '{resource}'",
            resource=resource,
            required_role="admin"
        )
    
    return {"username": username, "data": "some_user_data"}

# ทดสอบ
test_cases = [
    ("admin", "password123", "/admin"),
    ("user", "wrong_pass", "/profile"),
    ("unknown", "pass", "/home"),
    ("user", "secret", "/admin"),
]

for username, password, resource in test_cases:
    try:
        data = get_user_data(username, password, resource)
        print(f"สำเร็จ: {data}")
    except AuthenticationError as e:
        print(f"Auth Error [{e.code}]: {e.message} (user: {e.username})")
    except AuthorizationError as e:
        print(f"Authz Error [{e.code}]: {e.message} (resource: {e.resource})")
    except AppError as e:
        print(f"App Error [{e.code}]: {e.message}")
```

```python
# ตัวอย่างที่ 3: Custom exception พร้อม extra info
class NetworkError(Exception):
    """Exception สำหรับ network operations"""
    
    def __init__(self, url, status_code=None, message="Network error"):
        self.url = url
        self.status_code = status_code
        self.message = message
        
        full_message = f"{message}"
        if status_code:
            full_message += f" (HTTP {status_code})"
        full_message += f" - URL: {url}"
        
        super().__init__(full_message)
    
    def is_client_error(self):
        """ตรวจสอบว่าเป็น 4xx error หรือไม่"""
        return self.status_code and 400 <= self.status_code < 500
    
    def is_server_error(self):
        """ตรวจสอบว่าเป็น 5xx error หรือไม่"""
        return self.status_code and 500 <= self.status_code < 600

def fetch_url(url):
    # จำลอง HTTP request
    if "not-found" in url:
        raise NetworkError(url, 404, "Not Found")
    if "server-error" in url:
        raise NetworkError(url, 500, "Internal Server Error")
    if "forbidden" in url:
        raise NetworkError(url, 403, "Forbidden")
    return {"status": "ok", "data": "content"}

urls = [
    "https://api.example.com/data",
    "https://api.example.com/not-found",
    "https://api.example.com/server-error",
    "https://api.example.com/forbidden",
]

for url in urls:
    try:
        result = fetch_url(url)
        print(f"สำเร็จ: {url}")
    except NetworkError as e:
        if e.is_client_error():
            print(f"Client Error: {e}")
        elif e.is_server_error():
            print(f"Server Error (retry later): {e}")
        else:
            print(f"Network Error: {e}")
```

---

## 10. Exception Chaining

Exception chaining ช่วยให้เราเห็นว่า exception หนึ่งเกิดจาก exception อื่น

```python
# ตัวอย่างที่ 1: raise ... from ... (explicit chaining)
def read_config(filename):
    try:
        with open(filename) as f:
            import json
            return json.load(f)
    except FileNotFoundError as e:
        raise RuntimeError(f"ไม่สามารถโหลด config จาก '{filename}'") from e

try:
    config = read_config("nonexistent_config.json")
except RuntimeError as e:
    print(f"RuntimeError: {e}")
    print(f"Caused by: {e.__cause__}")
```

```python
# ตัวอย่างที่ 2: raise ... from None (suppress chaining)
def lookup_user(user_id):
    users = {1: "Alice", 2: "Bob"}
    try:
        return users[user_id]
    except KeyError:
        # ซ่อน implementation detail
        raise ValueError(f"ผู้ใช้ ID {user_id} ไม่มีในระบบ") from None

try:
    user = lookup_user(999)
except ValueError as e:
    print(f"ValueError: {e}")
    print(f"Cause: {e.__cause__}")  # None เพราะ suppress

# ตัวอย่างที่ 3: Implicit chaining (ไม่ได้ตั้งใจ)
try:
    try:
        x = int("abc")
    except ValueError:
        y = 1 / 0  # เกิด ZeroDivisionError ขณะจัดการ ValueError
except ZeroDivisionError as e:
    print(f"ZeroDivisionError: {e}")
    print(f"Context: {e.__context__}")  # ValueError เดิม
```

```python
# ตัวอย่างที่ 4: การใช้ exception chaining ใน application จริง
class DataProcessingError(Exception):
    pass

class DatabaseQueryError(Exception):
    pass

def fetch_from_db(query):
    # จำลอง database error
    if "invalid" in query:
        raise DatabaseQueryError(f"Query ผิดพลาด: {query}")
    return [{"id": 1, "value": 42}]

def process_query_results(data):
    results = []
    for row in data:
        try:
            processed = row["value"] * 2
            results.append(processed)
        except KeyError as e:
            raise DataProcessingError(f"ขาด column: {e}") from e
    return results

def run_report(query):
    try:
        raw_data = fetch_from_db(query)
        processed_data = process_query_results(raw_data)
        return processed_data
    except DatabaseQueryError as e:
        raise DataProcessingError("Report ล้มเหลวเพราะ DB error") from e

# ทดสอบ
try:
    report = run_report("SELECT invalid query")
except DataProcessingError as e:
    print(f"Error: {e}")
    if e.__cause__:
        print(f"Caused by: {e.__cause__}")
```

---

## 11. Context Managers กับ Exceptions

Context managers ใช้ `with` statement และช่วยจัดการ cleanup อัตโนมัติ

```python
# ตัวอย่างที่ 1: with statement พื้นฐาน
# แทน:
# f = open("file.txt", "r")
# try:
#     content = f.read()
# finally:
#     f.close()

# ใช้:
try:
    with open("file.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("ไม่พบไฟล์")
```

```python
# ตัวอย่างที่ 2: สร้าง Context Manager ด้วย class
class ManagedResource:
    def __init__(self, name):
        self.name = name
    
    def __enter__(self):
        print(f"เปิดทรัพยากร: {self.name}")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"ปิดทรัพยากร: {self.name}")
        if exc_type is not None:
            print(f"เกิด exception: {exc_type.__name__}: {exc_val}")
            return False  # False = ไม่ suppress exception
        return True
    
    def do_work(self):
        print(f"ทำงานกับ: {self.name}")
        return "result"

# ทดสอบ - ไม่มี exception
print("=== ไม่มี exception ===")
with ManagedResource("Database") as resource:
    result = resource.do_work()
    print(f"ผลลัพธ์: {result}")

print()

# ทดสอบ - มี exception
print("=== มี exception ===")
try:
    with ManagedResource("File") as resource:
        resource.do_work()
        raise ValueError("เกิดข้อผิดพลาด!")
except ValueError:
    print("Exception ถูก propagate ขึ้นมา")
```

```python
# ตัวอย่างที่ 3: Context Manager ด้วย contextlib
from contextlib import contextmanager

@contextmanager
def database_transaction():
    print("เริ่ม transaction")
    try:
        yield {"connection": "mock_db"}
        print("Commit transaction")
    except Exception as e:
        print(f"Rollback transaction เพราะ: {e}")
        raise
    finally:
        print("ปิด connection")

# ทดสอบ - สำเร็จ
print("=== Transaction สำเร็จ ===")
try:
    with database_transaction() as db:
        print(f"ทำงานกับ DB: {db}")
        # ทำงานปกติ
except Exception as e:
    print(f"Error: {e}")

print()

# ทดสอบ - ล้มเหลว
print("=== Transaction ล้มเหลว ===")
try:
    with database_transaction() as db:
        print(f"ทำงานกับ DB: {db}")
        raise RuntimeError("ข้อมูลไม่ถูกต้อง")
except Exception as e:
    print(f"Error ถูก handle: {e}")
```

```python
# ตัวอย่างที่ 4: suppress exception ด้วย contextlib.suppress
from contextlib import suppress

# แทน:
# try:
#     os.remove("file.txt")
# except FileNotFoundError:
#     pass

import os
with suppress(FileNotFoundError):
    os.remove("nonexistent_file.txt")
    print("ลบไฟล์แล้ว")

print("โปรแกรมทำงานต่อได้")
```

---

## 12. Best Practices

### 12.1 หลักการพื้นฐาน

```python
# ✅ ดี: ระบุ exception ที่เฉพาะเจาะจง
def good_practice():
    try:
        value = int("123")
    except ValueError:
        print("ข้อมูลไม่ใช่ตัวเลข")

# ❌ ไม่ดี: จับ exception แบบกว้างเกินไป
def bad_practice():
    try:
        value = int("123")
    except:  # จับทุก exception รวม SystemExit, KeyboardInterrupt
        print("มีข้อผิดพลาด")
```

```python
# ✅ ดี: ใช้ finally สำหรับ cleanup
def read_file_good(filename):
    f = None
    try:
        f = open(filename)
        return f.read()
    except FileNotFoundError:
        return None
    finally:
        if f:
            f.close()

# ✅ ดีกว่า: ใช้ with statement
def read_file_better(filename):
    try:
        with open(filename) as f:
            return f.read()
    except FileNotFoundError:
        return None
```

```python
# ✅ ดี: ให้ข้อมูลใน exception message
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(
            f"ยอดเงินไม่พอ: มี {balance} บาท แต่ต้องการ {amount} บาท "
            f"(ขาด {amount - balance} บาท)"
        )

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount

try:
    new_balance = withdraw(100, 150)
except InsufficientFundsError as e:
    print(f"Error: {e}")
    print(f"ยอดเงินปัจจุบัน: {e.balance}")
    print(f"จำนวนที่ต้องการ: {e.amount}")
```

```python
# ✅ ดี: Log exceptions อย่างเหมาะสม
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def process_user_data(user_data):
    try:
        name = user_data["name"]
        age = int(user_data["age"])
        
        if age < 0:
            raise ValueError(f"อายุต้องเป็นบวก: {age}")
        
        logger.info(f"ประมวลผล user '{name}' อายุ {age} สำเร็จ")
        return {"name": name, "age": age}
        
    except KeyError as e:
        logger.error(f"ข้อมูลไม่ครบ: ขาด key {e}")
        raise
    except ValueError as e:
        logger.warning(f"ข้อมูลไม่ถูกต้อง: {e}")
        raise
    except Exception as e:
        logger.exception(f"เกิดข้อผิดพลาดที่ไม่คาดคิด: {e}")
        raise

# ทดสอบ
data1 = {"name": "Alice", "age": "25"}
data2 = {"name": "Bob"}
data3 = {"name": "Charlie", "age": "-5"}

for data in [data1, data2, data3]:
    try:
        result = process_user_data(data)
        print(f"สำเร็จ: {result}")
    except Exception as e:
        print(f"Error: {type(e).__name__}: {e}")
```

### 12.2 สรุป Best Practices

| หลักการ | คำอธิบาย |
|---------|----------|
| ระบุ exception ที่เฉพาะ | อย่าใช้ `except:` หรือ `except Exception:` โดยไม่จำเป็น |
| ใช้ `with` statement | แทน try/finally สำหรับ resource management |
| ให้ข้อมูลเพียงพอ | ใส่ context ใน exception message |
| สร้าง custom exceptions | เมื่อ built-in exceptions ไม่เพียงพอ |
| Log exceptions | ใช้ logging module แทน print |
| Re-raise เมื่อจำเป็น | ถ้าจัดการไม่ได้ ให้ propagate ขึ้นไป |
| อย่า swallow exceptions | ถ้า catch แล้ว ต้องทำอะไรสักอย่าง |

---

## 13. ตัวอย่างโปรแกรมจริง

### 13.1 File Handler ที่สมบูรณ์

```python
import os
import json
import logging
from pathlib import Path

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class FileHandlerError(Exception):
    """Base exception สำหรับ FileHandler"""
    pass


class FileReadError(FileHandlerError):
    """Exception เมื่ออ่านไฟล์ไม่ได้"""
    def __init__(self, filepath, reason):
        self.filepath = filepath
        super().__init__(f"ไม่สามารถอ่านไฟล์ '{filepath}': {reason}")


class FileWriteError(FileHandlerError):
    """Exception เมื่อเขียนไฟล์ไม่ได้"""
    def __init__(self, filepath, reason):
        self.filepath = filepath
        super().__init__(f"ไม่สามารถเขียนไฟล์ '{filepath}': {reason}")


class FileHandler:
    """จัดการการอ่าน/เขียนไฟล์อย่างปลอดภัย"""
    
    def __init__(self, base_dir="."):
        self.base_dir = Path(base_dir)
    
    def read_text(self, filename, encoding="utf-8"):
        """อ่านไฟล์ text"""
        filepath = self.base_dir / filename
        
        try:
            with open(filepath, 'r', encoding=encoding) as f:
                content = f.read()
            logger.info(f"อ่านไฟล์สำเร็จ: {filepath}")
            return content
        except FileNotFoundError:
            raise FileReadError(filepath, "ไฟล์ไม่มีอยู่")
        except PermissionError:
            raise FileReadError(filepath, "ไม่มีสิทธิ์อ่านไฟล์")
        except UnicodeDecodeError:
            raise FileReadError(filepath, f"encoding '{encoding}' ไม่ถูกต้อง")
        except OSError as e:
            raise FileReadError(filepath, str(e))
    
    def write_text(self, filename, content, encoding="utf-8", mode="w"):
        """เขียนไฟล์ text"""
        filepath = self.base_dir / filename
        
        try:
            # สร้าง directory ถ้าไม่มี
            filepath.parent.mkdir(parents=True, exist_ok=True)
            
            with open(filepath, mode, encoding=encoding) as f:
                f.write(content)
            
            logger.info(f"เขียนไฟล์สำเร็จ: {filepath}")
            return True
        except PermissionError:
            raise FileWriteError(filepath, "ไม่มีสิทธิ์เขียนไฟล์")
        except OSError as e:
            raise FileWriteError(filepath, str(e))
    
    def read_json(self, filename):
        """อ่านไฟล์ JSON"""
        try:
            content = self.read_text(filename)
            data = json.loads(content)
            return data
        except FileReadError:
            raise
        except json.JSONDecodeError as e:
            filepath = self.base_dir / filename
            raise FileReadError(filepath, f"JSON ไม่ถูกต้อง: {e}")
    
    def write_json(self, filename, data, indent=2):
        """เขียนไฟล์ JSON"""
        try:
            content = json.dumps(data, ensure_ascii=False, indent=indent)
            self.write_text(filename, content)
            return True
        except (TypeError, ValueError) as e:
            filepath = self.base_dir / filename
            raise FileWriteError(filepath, f"ข้อมูลไม่สามารถแปลงเป็น JSON: {e}")
    
    def file_exists(self, filename):
        """ตรวจสอบว่าไฟล์มีอยู่หรือไม่"""
        return (self.base_dir / filename).exists()
    
    def safe_delete(self, filename):
        """ลบไฟล์อย่างปลอดภัย"""
        filepath = self.base_dir / filename
        try:
            os.remove(filepath)
            logger.info(f"ลบไฟล์สำเร็จ: {filepath}")
            return True
        except FileNotFoundError:
            logger.warning(f"ไม่พบไฟล์ที่จะลบ: {filepath}")
            return False
        except PermissionError:
            logger.error(f"ไม่มีสิทธิ์ลบไฟล์: {filepath}")
            return False


# ทดสอบ FileHandler
handler = FileHandler("/tmp")

# เขียนไฟล์
try:
    handler.write_text("test.txt", "สวัสดี Python!\nThis is a test.")
    print("เขียนไฟล์สำเร็จ")
except FileWriteError as e:
    print(f"Write Error: {e}")

# อ่านไฟล์
try:
    content = handler.read_text("test.txt")
    print(f"เนื้อหา: {content[:50]}...")
except FileReadError as e:
    print(f"Read Error: {e}")

# เขียน/อ่าน JSON
data = {"users": [{"name": "Alice", "age": 25}, {"name": "Bob", "age": 30}]}
try:
    handler.write_json("users.json", data)
    loaded = handler.read_json("users.json")
    print(f"JSON users: {len(loaded['users'])} คน")
except (FileReadError, FileWriteError) as e:
    print(f"JSON Error: {e}")

# ลบไฟล์
handler.safe_delete("test.txt")
handler.safe_delete("users.json")
```

### 13.2 API Caller ที่สมบูรณ์

```python
import json
import time
from typing import Dict, Any, Optional


class APIError(Exception):
    """Base exception สำหรับ API"""
    def __init__(self, message, status_code=None, response_body=None):
        self.status_code = status_code
        self.response_body = response_body
        super().__init__(message)


class APIConnectionError(APIError):
    """Exception เมื่อเชื่อมต่อ API ไม่ได้"""
    pass


class APITimeoutError(APIError):
    """Exception เมื่อ API timeout"""
    pass


class APIAuthError(APIError):
    """Exception เมื่อ authentication ล้มเหลว"""
    pass


class APIRateLimitError(APIError):
    """Exception เมื่อถึง rate limit"""
    def __init__(self, retry_after=None):
        self.retry_after = retry_after
        super().__init__(
            f"Rate limit exceeded. Retry after {retry_after} seconds"
        )


class MockAPIClient:
    """จำลอง API Client พร้อม error handling"""
    
    def __init__(self, base_url: str, api_key: str, timeout: int = 30):
        self.base_url = base_url
        self.api_key = api_key
        self.timeout = timeout
        self._request_count = 0
        self._max_requests_per_minute = 60
        self._last_minute_requests = []
    
    def _check_rate_limit(self):
        """ตรวจสอบ rate limit"""
        current_time = time.time()
        # ลบ request ที่เก่ากว่า 60 วินาที
        self._last_minute_requests = [
            t for t in self._last_minute_requests 
            if current_time - t < 60
        ]
        
        if len(self._last_minute_requests) >= self._max_requests_per_minute:
            retry_after = 60 - (current_time - self._last_minute_requests[0])
            raise APIRateLimitError(retry_after=int(retry_after))
        
        self._last_minute_requests.append(current_time)
    
    def _simulate_request(self, endpoint: str, method: str, data=None) -> Dict:
        """จำลองการส่ง HTTP request"""
        # จำลอง responses ต่างๆ
        if not self.api_key or self.api_key == "invalid":
            return {"status": 401, "body": {"error": "Unauthorized"}}
        
        if "timeout" in endpoint:
            raise TimeoutError("Request timed out")
        
        if "not-found" in endpoint:
            return {"status": 404, "body": {"error": "Not Found"}}
        
        if "server-error" in endpoint:
            return {"status": 500, "body": {"error": "Internal Server Error"}}
        
        # Success response
        return {
            "status": 200,
            "body": {
                "data": {"id": 1, "name": "Test"},
                "message": "success"
            }
        }
    
    def request(
        self, 
        method: str, 
        endpoint: str, 
        data: Optional[Dict] = None,
        retry_count: int = 3
    ) -> Dict[str, Any]:
        """ส่ง API request พร้อม retry logic"""
        
        for attempt in range(retry_count):
            try:
                # Check rate limit
                self._check_rate_limit()
                
                # ส่ง request
                response = self._simulate_request(endpoint, method, data)
                
                # ตรวจสอบ response
                status = response.get("status")
                body = response.get("body", {})
                
                if status == 200:
                    return body
                elif status == 401:
                    raise APIAuthError(
                        "Authentication ล้มเหลว: ตรวจสอบ API key",
                        status_code=status
                    )
                elif status == 404:
                    raise APIError(
                        f"ไม่พบ resource: {endpoint}",
                        status_code=status,
                        response_body=body
                    )
                elif status == 500:
                    if attempt < retry_count - 1:
                        wait_time = 2 ** attempt  # Exponential backoff
                        print(f"Server error, retry {attempt + 1}/{retry_count} ใน {wait_time}s...")
                        time.sleep(wait_time)
                        continue
                    raise APIError(
                        "Server error",
                        status_code=status,
                        response_body=body
                    )
                else:
                    raise APIError(
                        f"Unknown status code: {status}",
                        status_code=status
                    )
                    
            except APIRateLimitError as e:
                if attempt < retry_count - 1:
                    print(f"Rate limit, รอ {e.retry_after}s...")
                    # ไม่รอจริงในการทดสอบ
                    continue
                raise
            except TimeoutError:
                if attempt < retry_count - 1:
                    print(f"Timeout, retry {attempt + 1}/{retry_count}...")
                    continue
                raise APITimeoutError(
                    f"Request timeout หลัง {retry_count} ครั้ง",
                    status_code=None
                )
            except (APIAuthError, APIError):
                raise  # ไม่ retry สำหรับ auth errors
            except Exception as e:
                raise APIConnectionError(
                    f"Connection error: {e}"
                ) from e
        
        raise APIError("Maximum retry attempts exceeded")
    
    def get(self, endpoint: str) -> Dict:
        return self.request("GET", endpoint)
    
    def post(self, endpoint: str, data: Dict) -> Dict:
        return self.request("POST", endpoint, data)


# ทดสอบ API Client
client = MockAPIClient(
    base_url="https://api.example.com",
    api_key="valid_key_123"
)

test_endpoints = [
    "/users",
    "/not-found",
    "/server-error",
    "/timeout",
]

for endpoint in test_endpoints:
    print(f"\n--- Testing: {endpoint} ---")
    try:
        result = client.get(endpoint)
        print(f"สำเร็จ: {result}")
    except APIAuthError as e:
        print(f"Auth Error: {e}")
    except APITimeoutError as e:
        print(f"Timeout: {e}")
    except APIRateLimitError as e:
        print(f"Rate Limit: retry after {e.retry_after}s")
    except APIConnectionError as e:
        print(f"Connection Error: {e}")
    except APIError as e:
        print(f"API Error (HTTP {e.status_code}): {e}")
```

### 13.3 Input Validator ที่สมบูรณ์

```python
import re
from datetime import datetime
from typing import Any, Dict, List, Optional


class ValidationError(Exception):
    """Exception สำหรับ validation errors"""
    
    def __init__(self, field: str, value: Any, message: str):
        self.field = field
        self.value = value
        self.message = message
        super().__init__(f"Field '{field}': {message} (ได้รับ: {repr(value)})")


class MultiValidationError(Exception):
    """Exception สำหรับหลาย validation errors"""
    
    def __init__(self, errors: List[ValidationError]):
        self.errors = errors
        messages = [str(e) for e in errors]
        super().__init__(f"พบ {len(errors)} ข้อผิดพลาด:\n" + "\n".join(messages))


class FormValidator:
    """Validator สำหรับ form data"""
    
    def validate_string(
        self, 
        field: str, 
        value: Any,
        min_length: int = 0,
        max_length: int = 1000,
        pattern: Optional[str] = None,
        required: bool = True
    ) -> str:
        """Validate string field"""
        
        if value is None or value == "":
            if required:
                raise ValidationError(field, value, "จำเป็นต้องกรอก")
            return value or ""
        
        if not isinstance(value, str):
            try:
                value = str(value)
            except Exception:
                raise ValidationError(field, value, "ต้องเป็น string")
        
        value = value.strip()
        
        if len(value) < min_length:
            raise ValidationError(
                field, value,
                f"ต้องมีอย่างน้อย {min_length} ตัวอักษร"
            )
        
        if len(value) > max_length:
            raise ValidationError(
                field, value,
                f"ต้องไม่เกิน {max_length} ตัวอักษร"
            )
        
        if pattern and not re.match(pattern, value):
            raise ValidationError(
                field, value,
                f"รูปแบบไม่ถูกต้อง"
            )
        
        return value
    
    def validate_email(self, field: str, value: Any) -> str:
        """Validate email"""
        email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        value = self.validate_string(field, value)
        
        if not re.match(email_pattern, value):
            raise ValidationError(field, value, "รูปแบบ email ไม่ถูกต้อง")
        
        return value.lower()
    
    def validate_number(
        self, 
        field: str,
        value: Any,
        min_val: Optional[float] = None,
        max_val: Optional[float] = None,
        integer_only: bool = False
    ) -> float:
        """Validate number"""
        
        if value is None:
            raise ValidationError(field, value, "จำเป็นต้องกรอก")
        
        try:
            if integer_only:
                num = int(value)
            else:
                num = float(value)
        except (ValueError, TypeError):
            type_name = "จำนวนเต็ม" if integer_only else "ตัวเลข"
            raise ValidationError(field, value, f"ต้องเป็น{type_name}")
        
        if min_val is not None and num < min_val:
            raise ValidationError(
                field, value,
                f"ต้องมีค่าอย่างน้อย {min_val}"
            )
        
        if max_val is not None and num > max_val:
            raise ValidationError(
                field, value,
                f"ต้องมีค่าไม่เกิน {max_val}"
            )
        
        return num
    
    def validate_date(
        self,
        field: str,
        value: Any,
        format_str: str = "%Y-%m-%d"
    ) -> datetime:
        """Validate date"""
        
        if not value:
            raise ValidationError(field, value, "จำเป็นต้องกรอก")
        
        try:
            date = datetime.strptime(str(value), format_str)
            return date
        except ValueError:
            raise ValidationError(
                field, value,
                f"รูปแบบวันที่ต้องเป็น {format_str}"
            )
    
    def validate_form(self, form_data: Dict) -> Dict:
        """Validate ทั้ง form พร้อมรวบรวม errors"""
        errors = []
        validated = {}
        
        # ตรวจสอบทุก field
        validators = {
            "name": lambda: self.validate_string(
                "name", form_data.get("name"),
                min_length=2, max_length=50
            ),
            "email": lambda: self.validate_email(
                "email", form_data.get("email")
            ),
            "age": lambda: self.validate_number(
                "age", form_data.get("age"),
                min_val=1, max_val=120, integer_only=True
            ),
            "phone": lambda: self.validate_string(
                "phone", form_data.get("phone"),
                pattern=r'^\d{10}$',
                required=False
            ),
        }
        
        for field, validator in validators.items():
            try:
                validated[field] = validator()
            except ValidationError as e:
                errors.append(e)
        
        if errors:
            raise MultiValidationError(errors)
        
        return validated


# ทดสอบ FormValidator
validator = FormValidator()

test_data = [
    # ข้อมูลที่ถูกต้อง
    {
        "name": "Alice",
        "email": "alice@example.com",
        "age": "25",
        "phone": "0812345678"
    },
    # ข้อมูลที่มีข้อผิดพลาดหลายอย่าง
    {
        "name": "A",
        "email": "not-an-email",
        "age": "200",
        "phone": "123"
    },
    # ข้อมูลไม่ครบ
    {
        "name": "",
        "email": "",
        "age": None,
    },
]

for i, data in enumerate(test_data, 1):
    print(f"\n=== ทดสอบชุดที่ {i} ===")
    try:
        result = validator.validate_form(data)
        print(f"ผ่าน validation: {result}")
    except MultiValidationError as e:
        print(f"ไม่ผ่าน validation: {len(e.errors)} ข้อผิดพลาด")
        for error in e.errors:
            print(f"  - {error}")
    except ValidationError as e:
        print(f"ข้อผิดพลาด: {e}")
```

---

## 14. แบบฝึกหัด

### ข้อที่ 1: Safe Calculator

เขียนฟังก์ชัน `safe_calculator(a, operator, b)` ที่จัดการ exceptions ทั้งหมด

```python
# TODO: เขียนโค้ดของคุณที่นี่

# เฉลย
def safe_calculator(a, operator, b):
    """Calculator ที่ปลอดภัย"""
    try:
        a = float(a)
        b = float(b)
    except (ValueError, TypeError) as e:
        raise ValueError(f"ค่าที่ใส่ไม่ใช่ตัวเลข: {e}") from e
    
    operations = {
        '+': lambda x, y: x + y,
        '-': lambda x, y: x - y,
        '*': lambda x, y: x * y,
        '/': lambda x, y: x / y,
        '**': lambda x, y: x ** y,
        '%': lambda x, y: x % y,
    }
    
    if operator not in operations:
        raise ValueError(f"operator ไม่รองรับ: '{operator}'. รองรับ: {list(operations.keys())}")
    
    try:
        result = operations[operator](a, b)
        return result
    except ZeroDivisionError:
        raise ZeroDivisionError(f"ไม่สามารถหาร {a} ด้วย 0")
    except OverflowError:
        raise OverflowError(f"ผลลัพธ์ใหญ่เกินไป: {a} {operator} {b}")

# ทดสอบ
test_cases = [
    (10, '+', 5),
    (10, '-', 3),
    (4, '*', 3),
    (10, '/', 2),
    (10, '/', 0),
    ('abc', '+', 5),
    (2, '^', 3),
    (2, '**', 10),
]

for a, op, b in test_cases:
    try:
        result = safe_calculator(a, op, b)
        print(f"{a} {op} {b} = {result}")
    except Exception as e:
        print(f"Error ({type(e).__name__}): {e}")
```

### ข้อที่ 2: Custom Exception Hierarchy

สร้าง exception hierarchy สำหรับระบบ e-commerce

```python
# เฉลย
class ECommerceError(Exception):
    """Base exception สำหรับ e-commerce"""
    pass

class ProductError(ECommerceError):
    def __init__(self, product_id, message):
        self.product_id = product_id
        super().__init__(f"Product {product_id}: {message}")

class OutOfStockError(ProductError):
    def __init__(self, product_id, requested, available):
        self.requested = requested
        self.available = available
        super().__init__(
            product_id,
            f"สินค้าไม่พอ (ต้องการ: {requested}, มี: {available})"
        )

class PaymentError(ECommerceError):
    def __init__(self, amount, reason):
        self.amount = amount
        super().__init__(f"ชำระเงิน {amount} บาทล้มเหลว: {reason}")

class InsufficientFundsError(PaymentError):
    def __init__(self, amount, balance):
        self.balance = balance
        super().__init__(amount, f"ยอดเงินไม่พอ (มี: {balance} บาท)")

# Inventory
inventory = {"P001": 5, "P002": 0, "P003": 10}

def purchase(product_id, quantity, payment_amount, balance):
    # ตรวจสอบสินค้า
    if product_id not in inventory:
        raise ProductError(product_id, "ไม่พบสินค้า")
    
    available = inventory[product_id]
    if available < quantity:
        raise OutOfStockError(product_id, quantity, available)
    
    # คำนวณราคา
    prices = {"P001": 100, "P002": 200, "P003": 50}
    total = prices[product_id] * quantity
    
    # ตรวจสอบการชำระเงิน
    if payment_amount < total:
        raise InsufficientFundsError(total, payment_amount)
    
    if balance < total:
        raise InsufficientFundsError(total, balance)
    
    # สำเร็จ
    inventory[product_id] -= quantity
    return {"product": product_id, "quantity": quantity, "total": total}

# ทดสอบ
orders = [
    ("P001", 3, 500, 500),   # สำเร็จ
    ("P002", 1, 200, 200),   # Out of stock
    ("P003", 2, 50, 200),    # ยอดเงินไม่พอ
    ("P999", 1, 100, 100),   # สินค้าไม่มี
]

for product_id, qty, payment, balance in orders:
    try:
        result = purchase(product_id, qty, payment, balance)
        print(f"สั่งซื้อสำเร็จ: {result}")
    except OutOfStockError as e:
        print(f"สินค้าไม่พอ: {e}")
    except InsufficientFundsError as e:
        print(f"ยอดเงินไม่พอ: {e}")
    except ProductError as e:
        print(f"Product Error: {e}")
    except ECommerceError as e:
        print(f"E-Commerce Error: {e}")
```

### ข้อที่ 3: Retry Decorator

สร้าง decorator ที่ retry function เมื่อเกิด exception

```python
# เฉลย
import time
import functools
from typing import Type, Tuple

def retry(
    max_attempts: int = 3,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    delay: float = 1.0,
    backoff: float = 2.0
):
    """Decorator สำหรับ retry function เมื่อเกิด exception"""
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt < max_attempts:
                        wait_time = delay * (backoff ** (attempt - 1))
                        print(f"Attempt {attempt}/{max_attempts} failed: {e}")
                        print(f"Retrying in {wait_time:.1f}s...")
                        # time.sleep(wait_time)  # comment out เพื่อทดสอบเร็ว
                    else:
                        print(f"All {max_attempts} attempts failed")
            
            raise last_exception
        
        return wrapper
    return decorator

# จำลอง flaky function
call_count = [0]

@retry(max_attempts=3, exceptions=(ConnectionError, TimeoutError), delay=0.1)
def flaky_api_call(url):
    call_count[0] += 1
    if call_count[0] < 3:
        raise ConnectionError(f"Connection failed (attempt {call_count[0]})")
    return {"status": "ok", "data": "result"}

# ทดสอบ
try:
    result = flaky_api_call("https://api.example.com")
    print(f"สำเร็จ: {result}")
except ConnectionError as e:
    print(f"ล้มเหลวทั้งหมด: {e}")
```

### ข้อที่ 4-10: แบบฝึกหัดเพิ่มเติม

```python
# ข้อที่ 4: Context Manager สำหรับ Timer
# สร้าง context manager ที่วัดเวลาและ raise exception ถ้าเกิน timeout

import time
from contextlib import contextmanager

@contextmanager
def timeout_context(seconds, operation_name="operation"):
    start_time = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start_time
        if elapsed > seconds:
            print(f"คำเตือน: {operation_name} ใช้เวลา {elapsed:.2f}s (เกิน {seconds}s)")
        else:
            print(f"{operation_name} เสร็จใน {elapsed:.2f}s")

# ทดสอบ
with timeout_context(1.0, "การดึงข้อมูล"):
    time.sleep(0.5)  # จำลองการทำงาน

with timeout_context(0.3, "การประมวลผล"):
    time.sleep(0.5)  # จำลองการทำงานที่นานเกินไป
```

```python
# ข้อที่ 5: Exception logging decorator
import logging
import functools

def log_exceptions(logger=None, reraise=True):
    """Decorator ที่ log exceptions อัตโนมัติ"""
    if logger is None:
        logger = logging.getLogger(__name__)
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                logger.error(
                    f"Exception in {func.__name__}: "
                    f"{type(e).__name__}: {e}",
                    exc_info=True
                )
                if reraise:
                    raise
                return None
        return wrapper
    return decorator

logging.basicConfig(level=logging.ERROR)

@log_exceptions()
def divide(a, b):
    return a / b

# ทดสอบ
result = divide(10, 2)
print(f"10 / 2 = {result}")

try:
    result = divide(10, 0)
except ZeroDivisionError:
    print("Exception ถูก log และ re-raise")
```

```python
# ข้อที่ 6: Graceful shutdown
import signal
import sys

class GracefulShutdown:
    """จัดการ shutdown อย่างสวยงาม"""
    
    def __init__(self):
        self.shutdown_requested = False
        signal.signal(signal.SIGINT, self._handle_signal)
        signal.signal(signal.SIGTERM, self._handle_signal)
    
    def _handle_signal(self, signum, frame):
        print(f"\nได้รับ signal {signum}, กำลัง shutdown...")
        self.shutdown_requested = True
    
    def should_continue(self):
        return not self.shutdown_requested

# จำลองการทำงาน
shutdown = GracefulShutdown()
items_to_process = list(range(10))

for i, item in enumerate(items_to_process):
    if not shutdown.should_continue():
        print(f"หยุดทำงานที่ item {i}")
        break
    
    try:
        # จำลองการประมวลผล
        result = item * 2
        print(f"ประมวลผล item {item}: {result}")
    except Exception as e:
        print(f"Error ที่ item {item}: {e}")
        continue

print("โปรแกรมปิดอย่างปลอดภัย")
```

```python
# ข้อที่ 7-10: เฉลยย่อ

# ข้อที่ 7: จัดการ exception chain อย่างถูกต้อง
class DatabaseError(Exception): pass
class ConnectionError(Exception): pass

def connect_db(host):
    try:
        if host == "bad_host":
            raise OSError(f"Cannot connect to {host}")
        return f"connection_to_{host}"
    except OSError as e:
        raise ConnectionError(f"ไม่สามารถเชื่อมต่อ {host}") from e

def query_db(connection, query):
    try:
        if not connection:
            raise ValueError("ไม่มี connection")
        return [{"result": "data"}]
    except Exception as e:
        raise DatabaseError("Query ล้มเหลว") from e

try:
    conn = connect_db("bad_host")
    data = query_db(conn, "SELECT *")
except ConnectionError as e:
    print(f"Connection Error: {e}")
    print(f"Caused by: {e.__cause__}")
```

```python
# ข้อที่ 8: ตรวจสอบ type ด้วย exception
def ensure_type(value, expected_type, field_name="value"):
    if not isinstance(value, expected_type):
        raise TypeError(
            f"'{field_name}' ต้องเป็น {expected_type.__name__} "
            f"ไม่ใช่ {type(value).__name__}"
        )
    return value

# ทดสอบ
try:
    x = ensure_type(42, int, "age")
    print(f"age: {x}")
    y = ensure_type("hello", int, "age")
except TypeError as e:
    print(f"TypeError: {e}")
```

```python
# ข้อที่ 9: Validation Pipeline
from typing import Callable, List, Any

class ValidationPipeline:
    def __init__(self):
        self.validators: List[Callable] = []
    
    def add_validator(self, validator: Callable):
        self.validators.append(validator)
        return self  # method chaining
    
    def validate(self, value: Any) -> Any:
        for validator in self.validators:
            value = validator(value)  # ผ่านแต่ละ validator
        return value

# สร้าง validators
def strip_whitespace(value):
    if isinstance(value, str):
        return value.strip()
    return value

def to_lowercase(value):
    if isinstance(value, str):
        return value.lower()
    return value

def validate_not_empty(value):
    if not value:
        raise ValueError("ค่าต้องไม่ว่าง")
    return value

def validate_min_length(min_len):
    def validator(value):
        if len(str(value)) < min_len:
            raise ValueError(f"ต้องมีอย่างน้อย {min_len} ตัวอักษร")
        return value
    return validator

# ใช้งาน Pipeline
username_validator = (
    ValidationPipeline()
    .add_validator(strip_whitespace)
    .add_validator(to_lowercase)
    .add_validator(validate_not_empty)
    .add_validator(validate_min_length(3))
)

test_usernames = ["  Alice  ", "ab", "", "  Bob Smith  "]
for username in test_usernames:
    try:
        result = username_validator.validate(username)
        print(f"'{username}' -> '{result}' ✓")
    except ValueError as e:
        print(f"'{username}' -> Error: {e}")
```

```python
# ข้อที่ 10: Exception Report Generator
from datetime import datetime
import traceback

class ExceptionReport:
    """สร้าง report จาก exceptions"""
    
    def __init__(self):
        self.exceptions = []
    
    def capture(self, func, *args, **kwargs):
        """เรียก function และ capture exception"""
        try:
            result = func(*args, **kwargs)
            self.exceptions.append({
                "function": func.__name__,
                "status": "success",
                "result": result,
                "timestamp": datetime.now().isoformat()
            })
            return result
        except Exception as e:
            self.exceptions.append({
                "function": func.__name__,
                "status": "error",
                "exception_type": type(e).__name__,
                "exception_message": str(e),
                "traceback": traceback.format_exc(),
                "timestamp": datetime.now().isoformat()
            })
            return None
    
    def get_report(self):
        """สร้าง summary report"""
        total = len(self.exceptions)
        successes = sum(1 for e in self.exceptions if e["status"] == "success")
        failures = total - successes
        
        report = f"""
=== Exception Report ===
วันที่: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
ทั้งหมด: {total} operations
สำเร็จ: {successes}
ล้มเหลว: {failures}

รายละเอียด:
"""
        for i, exc in enumerate(self.exceptions, 1):
            if exc["status"] == "success":
                report += f"  {i}. ✓ {exc['function']}: สำเร็จ\n"
            else:
                report += (
                    f"  {i}. ✗ {exc['function']}: "
                    f"{exc['exception_type']}: {exc['exception_message']}\n"
                )
        
        return report

# ทดสอบ
reporter = ExceptionReport()

def good_function(x):
    return x * 2

def bad_function(x):
    return x / 0

def type_error_function(x):
    return x + "string"

reporter.capture(good_function, 5)
reporter.capture(bad_function, 10)
reporter.capture(type_error_function, 5)
reporter.capture(good_function, 20)

print(reporter.get_report())
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|--------------|
| ประเภท Errors | Syntax, Runtime, Logic errors |
| Built-in Exceptions | ZeroDivisionError, ValueError, TypeError และอื่นๆ |
| try/except | โครงสร้างพื้นฐานการจัดการ exception |
| Multiple except | จับ exception หลายประเภท |
| except as e | เข้าถึงข้อมูล exception |
| else clause | โค้ดที่รันเมื่อสำเร็จ |
| finally clause | Cleanup code ที่รันเสมอ |
| raise | ส่ง exception เอง |
| Custom exceptions | สร้าง exception ของตัวเอง |
| Exception chaining | เชื่อม exceptions เข้าด้วยกัน |
| Context managers | with statement สำหรับ resource management |
| Best practices | หลักการเขียน error handling ที่ดี |

### ขั้นต่อไป

ในบทถัดไป (Part 17) เราจะเรียนรู้เกี่ยวกับ **Modules, Packages & pip** ซึ่งช่วยให้เราจัดการโค้ดขนาดใหญ่ได้อย่างมีประสิทธิภาพ
