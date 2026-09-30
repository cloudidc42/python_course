# Part 27: Generators & Iterators

## สารบัญ
1. [Iterator Protocol](#iterator-protocol)
2. [Custom Iterators](#custom-iterators)
3. [Generator Functions (yield)](#generator-functions-yield)
4. [Generator Expressions](#generator-expressions)
5. [yield from](#yield-from)
6. [Generator Pipelines](#generator-pipelines)
7. [Infinite Generators](#infinite-generators)
8. [send() Method](#send-method)
9. [close() และ throw()](#close-และ-throw)
10. [itertools Module](#itertools-module)
11. [Memory Efficiency](#memory-efficiency)
12. [Real-World Use Cases](#real-world-use-cases)
13. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Iterator Protocol

**Iterator** คือ object ที่ implement protocol สองเมธอด:
- `__iter__()`: return ตัวเอง
- `__next__()`: return ค่าถัดไป หรือ raise `StopIteration` เมื่อหมด

**Iterable** คือ object ที่ implement `__iter__()` ที่ return iterator (เช่น list, tuple, str)

```
Iterable → iter() → Iterator → next() → ค่าถัดไป
```

### ตัวอย่าง 1: ดู Iterator Protocol ใน Action

```python
# list เป็น iterable แต่ไม่ใช่ iterator
my_list = [1, 2, 3]
print(hasattr(my_list, '__iter__'))  # True
print(hasattr(my_list, '__next__'))  # False - ไม่ใช่ iterator!

# ต้องใช้ iter() เพื่อได้ iterator
my_iter = iter(my_list)
print(hasattr(my_iter, '__iter__'))  # True
print(hasattr(my_iter, '__next__'))  # True - เป็น iterator!

# เรียก next() เพื่อดูค่า
print(next(my_iter))  # 1
print(next(my_iter))  # 2
print(next(my_iter))  # 3

try:
    print(next(my_iter))  # StopIteration!
except StopIteration:
    print("หมดค่าแล้ว")

# for loop ทำงานแบบนี้ภายใน
numbers = [10, 20, 30]
_iter = iter(numbers)
while True:
    try:
        value = next(_iter)
        print(value)
    except StopIteration:
        break
```

### ตัวอย่าง 2: ทดสอบ iterables ต่างๆ

```python
# สิ่งที่ iterable ใน Python
print("=== Iterables ===")

# strings
for char in "Hello":
    print(char, end=" ")
print()

# dict iterates over keys
data = {"a": 1, "b": 2, "c": 3}
for key in data:
    print(key, end=" ")
print()

# ไฟล์ก็ iterable
# for line in open("file.txt"):
#     print(line)

# range เป็น iterable (ไม่ใช่ list!)
r = range(5)
print(type(r))    # <class 'range'>
print(r[2])       # 2 - รองรับ indexing
print(3 in r)     # True - รองรับ membership test
print(list(r))    # [0, 1, 2, 3, 4]

# range ไม่เก็บค่าทั้งหมดในหน่วยความจำ
huge_range = range(10**18)
print(f"range(10^18): ใช้หน่วยความจำน้อยมาก")
print(f"ค่าสุดท้าย: {huge_range[-1]}")  # ได้ทันที
```

---

## Custom Iterators

### ตัวอย่าง 3: สร้าง Iterator อย่างง่าย

```python
class CountUp:
    """Iterator ที่นับขึ้นจาก start ถึง end"""
    
    def __init__(self, start, end, step=1):
        self.current = start
        self.end = end
        self.step = step
    
    def __iter__(self):
        """Return ตัวเอง - ทำให้เป็น iterator"""
        return self
    
    def __next__(self):
        """Return ค่าถัดไป หรือ raise StopIteration"""
        if self.current > self.end:
            raise StopIteration
        
        value = self.current
        self.current += self.step
        return value

# ใช้ใน for loop
for num in CountUp(1, 10, 2):
    print(num, end=" ")  # 1 3 5 7 9
print()

# ใช้ใน list comprehension
squared = [x**2 for x in CountUp(1, 5)]
print(squared)  # [1, 4, 9, 16, 25]

# ใช้ next() โดยตรง
counter = CountUp(10, 15)
print(next(counter))  # 10
print(next(counter))  # 11
```

### ตัวอย่าง 4: Iterator พร้อม State

```python
class Fibonacci:
    """Iterator ที่สร้าง Fibonacci sequence"""
    
    def __init__(self, max_count=None, max_value=None):
        self.a, self.b = 0, 1
        self.count = 0
        self.max_count = max_count
        self.max_value = max_value
    
    def __iter__(self):
        return self
    
    def __next__(self):
        # ตรวจสอบ stopping conditions
        if self.max_count is not None and self.count >= self.max_count:
            raise StopIteration
        if self.max_value is not None and self.a > self.max_value:
            raise StopIteration
        
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return value

# สร้าง 10 ตัวแรก
print(list(Fibonacci(max_count=10)))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# ค่าที่น้อยกว่า 100
print(list(Fibonacci(max_value=100)))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```

### ตัวอย่าง 5: Iterable Class (แยก Iterable กับ Iterator)

```python
class NumberRange:
    """Iterable ที่สร้าง iterator ใหม่ทุกครั้ง"""
    
    def __init__(self, start, stop, step=1):
        self.start = start
        self.stop = stop
        self.step = step
    
    def __iter__(self):
        """Return iterator ใหม่ทุกครั้ง"""
        return NumberRangeIterator(self.start, self.stop, self.step)
    
    def __len__(self):
        if self.step > 0:
            return max(0, (self.stop - self.start + self.step - 1) // self.step)
        return 0
    
    def __contains__(self, item):
        """รองรับ 'in' operator"""
        if self.step > 0:
            return self.start <= item < self.stop and (item - self.start) % self.step == 0
        return False

class NumberRangeIterator:
    """Iterator สำหรับ NumberRange"""
    
    def __init__(self, start, stop, step):
        self.current = start
        self.stop = stop
        self.step = step
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += self.step
        return value

r = NumberRange(0, 20, 3)
print(list(r))        # [0, 3, 6, 9, 12, 15, 18]
print(len(r))         # 7
print(9 in r)         # True
print(10 in r)        # False

# สามารถ iterate หลายครั้ง (เพราะเป็น Iterable ไม่ใช่ Iterator)
for x in r:
    print(x, end=" ")
print()
for x in r:
    print(x, end=" ")
print()
```

---

## Generator Functions (yield)

**Generator function** คือฟังก์ชันที่มี `yield` statement แทน `return` มันสร้าง **generator object** ซึ่งเป็น lazy iterator

ข้อดีของ generator:
- **Memory efficient**: สร้างค่าทีละตัวแทนที่จะสร้างทั้งหมดก่อน
- **Lazy evaluation**: คำนวณเมื่อต้องการเท่านั้น
- **Code simplicity**: ง่ายกว่า custom iterator class

### ตัวอย่าง 6: Generator Function พื้นฐาน

```python
def count_generator(start, end):
    """Generator function - ใช้ yield แทน return"""
    current = start
    while current <= end:
        print(f"  [กำลังสร้างค่า {current}]")
        yield current  # หยุดชั่วคราว และส่งค่า
        current += 1
    # เมื่อฟังก์ชันจบ StopIteration ถูก raise อัตโนมัติ

# สร้าง generator object (ยังไม่ทำงาน!)
gen = count_generator(1, 5)
print(f"Type: {type(gen)}")  # <class 'generator'>
print("เริ่มดึงค่า:")

print(next(gen))  # สร้างค่า 1, แสดง 1
print(next(gen))  # สร้างค่า 2, แสดง 2
print("--- for loop ---")

# for loop ดึงค่าต่อจากที่ค้างไว้
for value in gen:
    print(value)
```

### ตัวอย่าง 7: Fibonacci Generator

```python
def fibonacci():
    """Generator ที่สร้าง Fibonacci ได้ไม่จำกัด"""
    a, b = 0, 1
    while True:  # infinite loop - ไม่เป็นไรสำหรับ generator
        yield a
        a, b = b, a + b

# ดึง 10 ตัวแรก
fib = fibonacci()
first_10 = [next(fib) for _ in range(10)]
print(first_10)
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# ใช้ itertools.islice เพื่อจำกัดจำนวน
import itertools
first_20 = list(itertools.islice(fibonacci(), 20))
print(first_20)

# หา fibonacci ที่น้อยกว่า 1000
def fibonacci_below(limit):
    for fib in fibonacci():
        if fib >= limit:
            break
        yield fib

print(list(fibonacci_below(1000)))
```

### ตัวอย่าง 8: Generator พร้อม Multiple Yields

```python
def file_reader(filepath, chunk_size=1024):
    """อ่านไฟล์เป็น chunks"""
    # ตัวอย่างนี้จำลองการทำงาน
    data = "A" * 5000  # จำลองข้อมูลในไฟล์
    
    start = 0
    while start < len(data):
        chunk = data[start:start + chunk_size]
        yield chunk
        start += chunk_size

# ประมวลผลทีละ chunk โดยไม่โหลดทั้งหมด
total_chars = 0
for chunk in file_reader("fake_file.txt"):
    total_chars += len(chunk)

print(f"อ่านทั้งหมด {total_chars} characters")

# Generator ที่ yield หลายประเภท
def data_pipeline(items):
    """ประมวลผลข้อมูลแบบ step-by-step"""
    for item in items:
        # Step 1: Filter
        if item < 0:
            continue
        
        # Step 2: Transform
        processed = item * 2
        
        # Step 3: Yield result
        yield processed

data = [1, -2, 3, -4, 5, 6, -7, 8]
result = list(data_pipeline(data))
print(result)  # [2, 6, 10, 12, 16]
```

### ตัวอย่าง 9: Generator กับ Exception Handling

```python
def safe_generator(items):
    """Generator ที่ handle exceptions ภายใน"""
    for item in items:
        try:
            result = 100 / item  # อาจ ZeroDivisionError
            yield result
        except ZeroDivisionError:
            print(f"  ข้าม {item} (หารด้วยศูนย์)")
            yield None
        except Exception as e:
            print(f"  Error สำหรับ {item}: {e}")

data = [10, 5, 0, 2, 0, 1]
for value in safe_generator(data):
    if value is not None:
        print(f"  → {value:.2f}")
```

---

## Generator Expressions

คล้าย list comprehension แต่ใช้วงเล็บธรรมดา `()` แทน `[]` - สร้าง generator ที่ lazy

### ตัวอย่าง 10: List Comprehension vs Generator Expression

```python
import sys

# List comprehension - สร้างทั้งหมดทันที
list_comp = [x**2 for x in range(1000000)]
print(f"List: {sys.getsizeof(list_comp):,} bytes")

# Generator expression - lazy
gen_exp = (x**2 for x in range(1000000))
print(f"Generator: {sys.getsizeof(gen_exp):,} bytes")

# ผลลัพธ์เหมือนกัน แต่ memory ต่างกันมาก!
print(sum(x**2 for x in range(1000)))  # 332833500
```

### ตัวอย่าง 11: Generator Expression ซ้อนกัน

```python
# nested generator expressions
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# flatten matrix
flat = (elem for row in matrix for elem in row)
print(list(flat))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# filter และ transform
result = (x**2 for x in range(20) if x % 2 == 0)
print(list(result))  # [0, 4, 16, 36, 64, 100, 144, 196, 256, 324]

# ส่งเป็น argument โดยตรง (ไม่ต้องใส่วงเล็บเพิ่ม)
total = sum(x**2 for x in range(10))
print(total)  # 285

max_val = max(len(word) for word in ["hello", "world", "python", "generators"])
print(max_val)  # 10 (generators)
```

---

## yield from

`yield from` ใช้สำหรับ delegate ไปยัง generator อื่น (หรือ iterable ใดๆ)

### ตัวอย่าง 12: yield from พื้นฐาน

```python
def generator_a():
    yield 1
    yield 2
    yield 3

def generator_b():
    yield 4
    yield 5

# วิธีปกติ (ไม่ใช้ yield from)
def combined_old():
    for value in generator_a():
        yield value
    for value in generator_b():
        yield value

# ใช้ yield from (สั้นกว่ามาก!)
def combined_new():
    yield from generator_a()
    yield from generator_b()
    yield from range(6, 9)  # yield from ใช้กับ iterable ใดก็ได้

print(list(combined_old()))  # [1, 2, 3, 4, 5]
print(list(combined_new()))  # [1, 2, 3, 4, 5, 6, 7, 8]
```

### ตัวอย่าง 13: yield from กับ Recursive Generators

```python
def flatten(nested):
    """Flatten nested structure ด้วย recursive generator"""
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)  # recursive!
        else:
            yield item

# ทดสอบกับ structure ซ้อนกันหลายชั้น
data = [1, [2, 3, [4, 5]], 6, [7, [8, [9]]]]
print(list(flatten(data)))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# เดิน tree structure
def tree_walk(tree):
    """DFS traversal ของ tree"""
    if isinstance(tree, dict):
        yield tree.get('value')
        for child in tree.get('children', []):
            yield from tree_walk(child)
    else:
        yield tree

tree = {
    'value': 1,
    'children': [
        {'value': 2, 'children': [
            {'value': 4, 'children': []},
            {'value': 5, 'children': []}
        ]},
        {'value': 3, 'children': [
            {'value': 6, 'children': []}
        ]}
    ]
}

print(list(tree_walk(tree)))  # [1, 2, 4, 5, 3, 6]
```

---

## Generator Pipelines

### ตัวอย่าง 14: Data Processing Pipeline

```python
def read_data(items):
    """Stage 1: ส่งข้อมูลดิบ"""
    for item in items:
        yield item

def filter_positive(gen):
    """Stage 2: กรองเฉพาะค่าบวก"""
    for item in gen:
        if item > 0:
            yield item

def double_values(gen):
    """Stage 3: คูณสองทุกค่า"""
    for item in gen:
        yield item * 2

def add_prefix(gen, prefix="ITEM"):
    """Stage 4: เพิ่ม prefix"""
    for item in gen:
        yield f"{prefix}_{item}"

# สร้าง pipeline
raw_data = [-5, 3, -1, 8, 0, 2, -4, 7]

pipeline = add_prefix(
    double_values(
        filter_positive(
            read_data(raw_data)
        )
    )
)

# ประมวลผลแบบ lazy
for result in pipeline:
    print(result)
# ITEM_6, ITEM_16, ITEM_4, ITEM_14

# ใช้ฟังก์ชัน pipe() ให้อ่านง่ายขึ้น
def pipe(data, *functions):
    """Apply functions แบบ pipeline"""
    result = data
    for func in functions:
        result = func(result)
    return result

result = pipe(
    raw_data,
    read_data,
    filter_positive,
    double_values,
    lambda g: add_prefix(g, "VALUE")
)
print(list(result))
```

### ตัวอย่าง 15: ETL Pipeline

```python
import csv
from io import StringIO

def read_csv_data(csv_string):
    """Extract: อ่านข้อมูล CSV"""
    reader = csv.DictReader(StringIO(csv_string))
    for row in reader:
        yield row

def transform_types(records):
    """Transform: แปลงประเภทข้อมูล"""
    for record in records:
        yield {
            'name': record['name'].strip(),
            'age': int(record['age']),
            'salary': float(record['salary'])
        }

def filter_adults(records, min_age=18):
    """Filter: กรองเฉพาะผู้ใหญ่"""
    for record in records:
        if record['age'] >= min_age:
            yield record

def calculate_tax(records, tax_rate=0.2):
    """Transform: คำนวณภาษี"""
    for record in records:
        record = dict(record)
        record['tax'] = record['salary'] * tax_rate
        record['net_salary'] = record['salary'] - record['tax']
        yield record

# ข้อมูลตัวอย่าง
csv_data = """name,age,salary
สมชาย,25,50000
สมหญิง,17,30000
วิชัย,30,75000
นงนุช,16,25000
อนันต์,28,60000"""

# สร้าง ETL pipeline
pipeline = calculate_tax(
    filter_adults(
        transform_types(
            read_csv_data(csv_data)
        )
    )
)

print(f"{'ชื่อ':<15} {'เงินเดือน':>12} {'ภาษี':>10} {'สุทธิ':>12}")
print("-" * 52)
for record in pipeline:
    print(f"{record['name']:<15} {record['salary']:>12,.0f} "
          f"{record['tax']:>10,.0f} {record['net_salary']:>12,.0f}")
```

---

## Infinite Generators

### ตัวอย่าง 16: Infinite Sequence Generators

```python
import itertools

def natural_numbers(start=1):
    """ตัวเลขธรรมชาติ 1, 2, 3, ..."""
    n = start
    while True:
        yield n
        n += 1

def powers_of_two():
    """2^0, 2^1, 2^2, ..."""
    n = 0
    while True:
        yield 2 ** n
        n += 1

def cycle_values(*values):
    """วนซ้ำค่าที่กำหนด"""
    while True:
        for value in values:
            yield value

# ใช้ itertools.islice เพื่อดึงจำนวนจำกัด
print("ตัวเลขธรรมชาติ 10 ตัวแรก:")
print(list(itertools.islice(natural_numbers(), 10)))

print("กำลังของ 2 แรก 8 ตัว:")
print(list(itertools.islice(powers_of_two(), 8)))

print("วนซ้ำ 12 รอบ:")
seasons = cycle_values("ฤดูใบไม้ผลิ", "ฤดูร้อน", "ฤดูใบไม้ร่วง", "ฤดูหนาว")
print(list(itertools.islice(seasons, 8)))
```

### ตัวอย่าง 17: Sieve of Eratosthenes (จำนวนเฉพาะ)

```python
def integers_from(n):
    """ตัวเลขจำนวนเต็มตั้งแต่ n"""
    while True:
        yield n
        n += 1

def sieve(numbers):
    """กรองจำนวนที่หารด้วย prime ตัวแรกลงตัว"""
    prime = next(numbers)
    yield prime
    yield from sieve(n for n in numbers if n % prime != 0)

def first_n_primes(n):
    """หา n จำนวนเฉพาะแรก"""
    import itertools
    return list(itertools.islice(sieve(integers_from(2)), n))

# ระวัง: recursion depth สำหรับจำนวนใหญ่
print(first_n_primes(20))
# [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71]

# วิธีที่ดีกว่าสำหรับจำนวนมาก
def prime_generator():
    """สร้างจำนวนเฉพาะไม่จำกัด (วิธีที่ efficient กว่า)"""
    yield 2
    primes = [2]
    candidate = 3
    while True:
        is_prime = True
        for p in primes:
            if p * p > candidate:
                break
            if candidate % p == 0:
                is_prime = False
                break
        if is_prime:
            primes.append(candidate)
            yield candidate
        candidate += 2

primes = prime_generator()
print("100 จำนวนเฉพาะแรก:")
first_100 = list(itertools.islice(primes, 100))
print(first_100)
```

---

## send() Method

`send()` ทำให้ generator รับค่าจากภายนอกขณะทำงาน - เปลี่ยน generator เป็น **coroutine**

```
caller → send(value) → generator (ได้ value ผ่าน yield expression)
caller ← yield expression ← generator
```

### ตัวอย่าง 18: send() พื้นฐาน

```python
def accumulator():
    """Generator ที่สะสมค่าที่ถูก send เข้ามา"""
    total = 0
    while True:
        # yield ส่งค่าออก และรับค่าที่ถูก send เข้ามา
        value = yield total
        if value is None:
            break
        total += value
    yield total  # ผลรวมสุดท้าย

# ต้อง next() หรือ send(None) ก่อนเพื่อ advance ไปยัง yield แรก
gen = accumulator()
current = next(gen)  # หรือ gen.send(None)
print(f"เริ่มต้น: {current}")  # 0

current = gen.send(10)
print(f"หลัง send(10): {current}")  # 10

current = gen.send(20)
print(f"หลัง send(20): {current}")  # 30

current = gen.send(5)
print(f"หลัง send(5): {current}")  # 35
```

### ตัวอย่าง 19: Coroutine Pattern

```python
def running_average():
    """Coroutine ที่คำนวณ running average"""
    total = 0
    count = 0
    average = None
    
    while True:
        value = yield average
        if value is None:
            return
        total += value
        count += 1
        average = total / count

def coroutine(func):
    """Decorator เพื่อ advance coroutine ไปยัง yield แรก"""
    import functools
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)  # advance ไปยัง yield แรก
        return gen
    return wrapper

@coroutine
def smart_average():
    total = 0
    count = 0
    average = None
    
    while True:
        value = yield average
        if value is None:
            return
        total += value
        count += 1
        average = total / count

avg = smart_average()  # ไม่ต้อง next() เพราะ @coroutine handle แล้ว

values = [10, 20, 30, 40, 50]
for v in values:
    current_avg = avg.send(v)
    print(f"เพิ่ม {v}: เฉลี่ย = {current_avg:.2f}")
```

### ตัวอย่าง 20: Data Sink (Coroutine สำหรับรับข้อมูล)

```python
def coroutine(func):
    import functools
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)
        return gen
    return wrapper

@coroutine
def writer(filename):
    """จำลองการเขียนลงไฟล์"""
    items = []
    while True:
        item = yield
        if item is None:
            print(f"บันทึกลง {filename}: {items}")
            return
        items.append(item)

@coroutine
def broadcaster(*targets):
    """ส่งข้อมูลไปหลาย targets"""
    while True:
        item = yield
        for target in targets:
            target.send(item)

# สร้าง sinks
log_writer = writer("log.txt")
db_writer = writer("db.txt")

# สร้าง broadcaster
broadcast = broadcaster(log_writer, db_writer)

# ส่งข้อมูล
for data in ["record1", "record2", "record3"]:
    broadcast.send(data)

broadcast.send(None)   # signal to finish
log_writer.send(None)  # close
db_writer.send(None)   # close
```

---

## close() และ throw()

### ตัวอย่าง 21: close() - ปิด Generator

```python
def resource_generator():
    """Generator ที่ต้องการ cleanup"""
    print("เปิด resource")
    try:
        count = 0
        while True:
            yield count
            count += 1
    except GeneratorExit:
        print("Generator ถูกปิด - cleanup!")
    finally:
        print("finally block ทำงาน")

gen = resource_generator()
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2
gen.close()       # ปิด generator - GeneratorExit ถูก raise
```

### ตัวอย่าง 22: throw() - ส่ง Exception เข้า Generator

```python
def resilient_generator():
    """Generator ที่ handle exceptions ได้"""
    i = 0
    while True:
        try:
            yield i
            i += 1
        except ValueError as e:
            print(f"  ได้รับ ValueError: {e}")
            yield f"Error: {e}"
        except RuntimeError:
            print("  ได้รับ RuntimeError - หยุด")
            return

gen = resilient_generator()
print(next(gen))                        # 0
print(next(gen))                        # 1
print(gen.throw(ValueError, "bad value"))  # ส่ง exception เข้า
print(next(gen))                        # 2

try:
    gen.throw(RuntimeError)             # จะ raise StopIteration
except StopIteration:
    print("Generator หยุดแล้ว")
```

---

## itertools Module

`itertools` มี building blocks สำหรับ functional programming กับ iterators

### ตัวอย่าง 23: Infinite Iterators

```python
import itertools

# count(start, step) - นับขึ้น
print("count:")
counter = itertools.count(10, 5)
print(list(itertools.islice(counter, 6)))  # [10, 15, 20, 25, 30, 35]

# cycle(iterable) - วนซ้ำ
print("cycle:")
colors = itertools.cycle(["แดง", "เขียว", "น้ำเงิน"])
print(list(itertools.islice(colors, 9)))

# repeat(value, times) - ซ้ำค่าเดิม
print("repeat:")
print(list(itertools.repeat("Hello", 4)))  # ['Hello', 'Hello', 'Hello', 'Hello']
print(list(itertools.repeat(0, 5)))        # [0, 0, 0, 0, 0]
```

### ตัวอย่าง 24: Finite Iterators

```python
import itertools

# chain() - ต่อ iterables
print("chain:")
result = list(itertools.chain([1, 2], [3, 4], [5, 6]))
print(result)  # [1, 2, 3, 4, 5, 6]

result = list(itertools.chain.from_iterable([[1, 2], [3, 4], [5, 6]]))
print(result)  # [1, 2, 3, 4, 5, 6]

# compress() - กรองด้วย boolean mask
print("compress:")
data = ['a', 'b', 'c', 'd', 'e']
mask = [True, False, True, False, True]
print(list(itertools.compress(data, mask)))  # ['a', 'c', 'e']

# dropwhile() - ข้ามค่าจนกว่า condition จะเป็น False
print("dropwhile:")
print(list(itertools.dropwhile(lambda x: x < 5, [1, 2, 4, 6, 7, 2, 1])))
# [6, 7, 2, 1] - เมื่อ condition เป็น False แล้วจะไม่ filter อีก

# takewhile() - ดึงค่าจนกว่า condition จะเป็น False
print("takewhile:")
print(list(itertools.takewhile(lambda x: x < 5, [1, 2, 4, 6, 7, 2, 1])))
# [1, 2, 4]

# islice() - ดึงบางส่วน
print("islice:")
print(list(itertools.islice(range(100), 5, 20, 3)))  # [5, 8, 11, 14, 17]

# starmap() - map กับ tuple arguments
print("starmap:")
pairs = [(2, 3), (4, 2), (3, 5)]
print(list(itertools.starmap(pow, pairs)))  # [8, 16, 243]
```

### ตัวอย่าง 25: Combinatoric Iterators

```python
import itertools

# product() - Cartesian product
print("product:")
print(list(itertools.product([1, 2], [3, 4])))
# [(1, 3), (1, 4), (2, 3), (2, 4)]

print(list(itertools.product("AB", repeat=3)))
# ['AAA', 'AAB', 'ABA', ...] ทั้งหมด 8 ชุด

# permutations() - เรียงลำดับ
print("permutations:")
print(list(itertools.permutations([1, 2, 3])))
# [(1,2,3), (1,3,2), (2,1,3), (2,3,1), (3,1,2), (3,2,1)]

print(list(itertools.permutations([1, 2, 3, 4], 2)))  # เลือก 2 จาก 4
# 12 คู่

# combinations() - เลือกโดยไม่สนลำดับ
print("combinations:")
print(list(itertools.combinations([1, 2, 3, 4], 2)))
# [(1,2), (1,3), (1,4), (2,3), (2,4), (3,4)]

# combinations_with_replacement() - เลือกซ้ำได้
print("combinations_with_replacement:")
print(list(itertools.combinations_with_replacement([1, 2, 3], 2)))
# [(1,1), (1,2), (1,3), (2,2), (2,3), (3,3)]
```

### ตัวอย่าง 26: Grouping Iterators

```python
import itertools

# groupby() - จัดกลุ่มค่าที่ติดกันและเท่ากัน
print("groupby:")
data = [1, 1, 2, 2, 2, 3, 1, 1]
for key, group in itertools.groupby(data):
    print(f"  {key}: {list(group)}")

# จัดกลุ่มตาม key function
words = ["apple", "ant", "banana", "bear", "cat", "cherry"]
words.sort(key=lambda w: w[0])  # ต้อง sort ก่อน!

print("จัดกลุ่มตามตัวอักษรแรก:")
for letter, group in itertools.groupby(words, key=lambda w: w[0]):
    print(f"  {letter}: {list(group)}")

# zip_longest() - zip แต่เติม fillvalue เมื่อหมด
print("zip_longest:")
a = [1, 2, 3, 4, 5]
b = ["a", "b", "c"]
print(list(itertools.zip_longest(a, b, fillvalue="N/A")))
# [(1,'a'), (2,'b'), (3,'c'), (4,'N/A'), (5,'N/A')]

# accumulate() - คำนวณสะสม
print("accumulate:")
import operator
print(list(itertools.accumulate([1, 2, 3, 4, 5])))              # [1, 3, 6, 10, 15]
print(list(itertools.accumulate([1, 2, 3, 4, 5], operator.mul))) # [1, 2, 6, 24, 120]

# pairwise() (Python 3.10+)
# print(list(itertools.pairwise([1, 2, 3, 4, 5])))
# [(1,2), (2,3), (3,4), (4,5)]
```

---

## Memory Efficiency

### ตัวอย่าง 27: เปรียบเทียบ Memory Usage

```python
import sys
import tracemalloc

def measure_memory(func, *args, **kwargs):
    tracemalloc.start()
    result = func(*args, **kwargs)
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    return result, peak

def with_list(n):
    return sum([x**2 for x in range(n)])

def with_generator(n):
    return sum(x**2 for x in range(n))

n = 1000000

_, list_peak = measure_memory(with_list, n)
_, gen_peak = measure_memory(with_generator, n)

print(f"List: peak memory = {list_peak / 1024 / 1024:.1f} MB")
print(f"Generator: peak memory = {gen_peak / 1024 / 1024:.1f} MB")
print(f"Generator ประหยัด: {list_peak / gen_peak:.0f}x")

# sizeof comparison
small_list = list(range(1000))
small_gen = (x for x in range(1000))
print(f"\nList size: {sys.getsizeof(small_list):,} bytes")
print(f"Gen size:  {sys.getsizeof(small_gen):,} bytes")
```

### ตัวอย่าง 28: Reading Large Files with Generators

```python
import os
from io import StringIO

def read_large_file_by_lines(fileobj):
    """อ่านไฟล์ใหญ่ทีละบรรทัด"""
    while True:
        line = fileobj.readline()
        if not line:
            break
        yield line.strip()

def read_in_chunks(fileobj, chunk_size=8192):
    """อ่านไฟล์เป็น chunks"""
    while True:
        chunk = fileobj.read(chunk_size)
        if not chunk:
            break
        yield chunk

def grep_generator(lines, pattern):
    """กรองบรรทัดที่มี pattern"""
    import re
    regex = re.compile(pattern)
    for line in lines:
        if regex.search(line):
            yield line

# จำลองไฟล์ขนาดใหญ่
fake_log = StringIO("""
2024-01-15 10:00:00 INFO Server started
2024-01-15 10:01:00 ERROR Connection failed
2024-01-15 10:02:00 INFO Request processed
2024-01-15 10:03:00 WARNING High memory usage
2024-01-15 10:04:00 ERROR Timeout occurred
2024-01-15 10:05:00 INFO Request processed
""".strip())

# Pipeline สำหรับ grep
error_lines = list(grep_generator(
    read_large_file_by_lines(fake_log),
    r"ERROR|WARNING"
))

print("บรรทัดที่มี ERROR หรือ WARNING:")
for line in error_lines:
    print(f"  {line}")
```

---

## Real-World Use Cases

### ตัวอย่าง 29: Database Query Generator

```python
import sqlite3
import os

def db_query_generator(db_path, query, params=(), batch_size=100):
    """
    Query database และ return ผลลัพธ์ทีละ batch
    แทนที่จะโหลดทั้งหมดในครั้งเดียว
    """
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row  # ทำให้ access ด้วย column name ได้
    cursor = conn.cursor()
    
    try:
        cursor.execute(query, params)
        
        while True:
            rows = cursor.fetchmany(batch_size)
            if not rows:
                break
            for row in rows:
                yield dict(row)
    finally:
        conn.close()

# ใช้งาน
def demo_db_generator():
    db_path = "/tmp/demo.db"
    
    # สร้าง database ตัวอย่าง
    conn = sqlite3.connect(db_path)
    c = conn.cursor()
    c.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT, age INTEGER)")
    c.execute("DELETE FROM users")  # ล้างข้อมูลเก่า
    
    data = [(i, f"User {i}", 20 + i % 40) for i in range(1, 101)]
    c.executemany("INSERT INTO users VALUES (?, ?, ?)", data)
    conn.commit()
    conn.close()
    
    # Query ด้วย generator
    adults = (
        user for user in db_query_generator(
            db_path,
            "SELECT * FROM users WHERE age >= ?",
            (18,),
            batch_size=10
        )
        if user['age'] >= 25
    )
    
    total = 0
    for user in adults:
        total += 1
    print(f"พบผู้ใช้ที่มีอายุ >= 25: {total} คน")

demo_db_generator()
```

### ตัวอย่าง 30: API Pagination Generator

```python
import json
from unittest.mock import patch

def paginated_api_generator(base_url, params=None, page_size=10):
    """
    Generator สำหรับดึงข้อมูลจาก paginated API
    ดึงหน้าถัดไปอัตโนมัติ
    """
    params = params or {}
    page = 1
    
    # จำลอง API call
    def mock_api_call(url, page, page_size):
        total_items = 45
        start = (page - 1) * page_size
        end = min(start + page_size, total_items)
        
        items = [{"id": i, "value": f"item_{i}"} 
                for i in range(start + 1, end + 1)]
        
        return {
            "items": items,
            "page": page,
            "total": total_items,
            "has_next": end < total_items
        }
    
    while True:
        response = mock_api_call(base_url, page, page_size)
        
        for item in response['items']:
            yield item
        
        if not response.get('has_next', False):
            break
        
        page += 1

# ดึงข้อมูลทั้งหมดโดยไม่ต้องรู้จำนวนหน้า
all_items = list(paginated_api_generator(
    "https://api.example.com/items",
    page_size=10
))

print(f"ดึงข้อมูลทั้งหมด: {len(all_items)} items")
print(f"5 items แรก: {all_items[:5]}")
```

### ตัวอย่าง 31: Log Parser Generator

```python
import re
from datetime import datetime

def parse_log_entries(log_lines):
    """Parse log entries จาก raw lines"""
    pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) (.+)'
    
    for line in log_lines:
        match = re.match(pattern, line)
        if match:
            timestamp_str, level, message = match.groups()
            yield {
                'timestamp': datetime.strptime(timestamp_str, '%Y-%m-%d %H:%M:%S'),
                'level': level,
                'message': message
            }

def filter_by_level(entries, *levels):
    """กรองตาม log level"""
    levels_set = set(levels)
    for entry in entries:
        if entry['level'] in levels_set:
            yield entry

def within_timerange(entries, start, end):
    """กรองตามช่วงเวลา"""
    for entry in entries:
        if start <= entry['timestamp'] <= end:
            yield entry

# ตัวอย่างข้อมูล
log_data = """2024-01-15 10:00:00 INFO Server started
2024-01-15 10:01:00 ERROR Connection failed: timeout
2024-01-15 10:02:00 INFO Request /api/users processed
2024-01-15 10:03:00 WARNING Memory usage at 85%
2024-01-15 10:04:00 ERROR Database query failed
2024-01-15 10:05:00 DEBUG Cache hit for key=user_1
2024-01-15 10:06:00 INFO Request /api/products processed""".splitlines()

# สร้าง pipeline
start_time = datetime(2024, 1, 15, 10, 0, 0)
end_time = datetime(2024, 1, 15, 10, 5, 0)

critical_entries = within_timerange(
    filter_by_level(
        parse_log_entries(log_data),
        "ERROR", "WARNING"
    ),
    start_time, end_time
)

print("Log entries ที่สำคัญ:")
for entry in critical_entries:
    print(f"  [{entry['level']}] {entry['timestamp']} - {entry['message']}")
```

---

## แบบฝึกหัด

### ข้อ 1: สร้าง `range_float` generator

สร้าง generator ที่ทำงานเหมือน range() แต่รองรับ float step

**คำตอบ:**

```python
def range_float(start, stop=None, step=1.0):
    """Range ที่รองรับ float"""
    if stop is None:
        start, stop = 0.0, start
    
    current = float(start)
    step = float(step)
    
    if step > 0:
        while current < stop:
            yield round(current, 10)  # แก้ floating point error
            current += step
    elif step < 0:
        while current > stop:
            yield round(current, 10)
            current += step
    else:
        raise ValueError("step ต้องไม่เป็น 0")

print(list(range_float(0, 1, 0.1)))
# [0.0, 0.1, 0.2, ..., 0.9]

print(list(range_float(1.0, 0, -0.25)))
# [1.0, 0.75, 0.5, 0.25]
```

### ข้อ 2: สร้าง `window` generator

สร้าง sliding window generator

**คำตอบ:**

```python
from collections import deque

def window(iterable, size):
    """Sliding window generator"""
    it = iter(iterable)
    win = deque(maxlen=size)
    
    # เติม window แรก
    for _ in range(size):
        try:
            win.append(next(it))
        except StopIteration:
            return
    
    yield tuple(win)
    
    # เลื่อน window
    for item in it:
        win.append(item)
        yield tuple(win)

data = [1, 2, 3, 4, 5, 6, 7]
print(list(window(data, 3)))
# [(1,2,3), (2,3,4), (3,4,5), (4,5,6), (5,6,7)]
```

### ข้อ 3: สร้าง `chunked` generator

แบ่ง iterable เป็น chunks ขนาดที่กำหนด

**คำตอบ:**

```python
def chunked(iterable, size):
    """แบ่ง iterable เป็น chunks"""
    it = iter(iterable)
    while True:
        chunk = []
        try:
            for _ in range(size):
                chunk.append(next(it))
            yield chunk
        except StopIteration:
            if chunk:
                yield chunk
            break

data = range(17)
print(list(chunked(data, 5)))
# [[0,1,2,3,4], [5,6,7,8,9], [10,11,12,13,14], [15,16]]
```

### ข้อ 4: สร้าง `unique_everseen` generator

กรองเฉพาะค่าที่ไม่เคยเห็นมาก่อน (รักษาลำดับ)

**คำตอบ:**

```python
def unique_everseen(iterable, key=None):
    """Return unique elements ตามลำดับที่เจอครั้งแรก"""
    seen = set()
    
    for element in iterable:
        k = key(element) if key else element
        if k not in seen:
            seen.add(k)
            yield element

data = [1, 2, 2, 3, 1, 4, 3, 5]
print(list(unique_everseen(data)))  # [1, 2, 3, 4, 5]

words = ["hello", "HELLO", "world", "World"]
print(list(unique_everseen(words, key=str.lower)))
# ['hello', 'world']
```

### ข้อ 5-10: แบบฝึกหัดเพิ่มเติม

**ข้อ 5**: สร้าง `roundrobin` generator ที่ดึงค่าจากหลาย iterables สลับกัน

**ข้อ 6**: สร้าง `flatten_deep` generator สำหรับ flatten nested structure ลึกไม่จำกัด

**ข้อ 7**: สร้าง coroutine `grep` ที่รับ pattern และ yield บรรทัดที่ตรงกัน

**ข้อ 8**: สร้าง generator สำหรับสร้าง permutations ของ string โดยไม่ใช้ itertools

**ข้อ 9**: สร้าง `batch_processor` generator ที่ประมวลผล items เป็น batches พร้อม progress tracking

**ข้อ 10**: สร้าง pipeline สำหรับ process CSV data: อ่าน → validate → transform → aggregate

---

## สรุป

| เครื่องมือ | ใช้เมื่อไหร่ |
|-----------|-------------|
| Iterator class | ต้องการ state หรือ custom iteration logic ซับซ้อน |
| Generator function | สร้าง sequence แบบ lazy ง่ายๆ |
| Generator expression | filter/transform แบบ one-liner |
| yield from | delegate ไปยัง sub-generators |
| send() | coroutine - รับ input ขณะทำงาน |
| itertools | combinatorial, infinite iterators |

**หลักการสำคัญ:**
- **Lazy evaluation**: คำนวณเมื่อต้องการเท่านั้น
- **Memory efficiency**: ไม่สร้างข้อมูลทั้งหมดในครั้งเดียว
- **Pipeline composition**: รวม generators เป็น pipeline ที่ elegant
- **Infinite sequences**: สร้าง sequence ที่ไม่มีที่สิ้นสุดได้
