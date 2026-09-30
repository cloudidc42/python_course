# Part 11: Lists - Complete Guide

## บทนำ (Introduction)

**List** เป็นโครงสร้างข้อมูลที่ใช้บ่อยที่สุดใน Python เป็น **ordered, mutable, และ allow duplicate** collection ที่สามารถเก็บข้อมูลได้หลายประเภทในตัวแปรเดียว

### คุณสมบัติหลักของ List
| คุณสมบัติ | ความหมาย |
|-----------|-----------|
| Ordered | ข้อมูลเรียงตามลำดับที่ใส่เข้ามา |
| Mutable | แก้ไขข้อมูลได้หลังสร้าง |
| Allow Duplicates | เก็บข้อมูลซ้ำกันได้ |
| Dynamic | ขนาดเปลี่ยนแปลงได้ |

---

## 1. การสร้าง List (Creating Lists)

### 1.1 วิธีพื้นฐาน

```python
# สร้าง list เปล่า
empty_list = []
empty_list2 = list()

print(empty_list)    # []
print(empty_list2)   # []

# สร้าง list ด้วยข้อมูล
numbers = [1, 2, 3, 4, 5]
fruits = ["apple", "banana", "cherry"]
mixed = [1, "hello", 3.14, True, None]

print(numbers)  # [1, 2, 3, 4, 5]
print(fruits)   # ['apple', 'banana', 'cherry']
print(mixed)    # [1, 'hello', 3.14, True, None]
```

### 1.2 สร้าง List จาก iterable

```python
# จาก string
chars = list("Python")
print(chars)  # ['P', 'y', 't', 'h', 'o', 'n']

# จาก range
nums = list(range(1, 11))
print(nums)  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# จาก tuple
tup = (10, 20, 30)
from_tuple = list(tup)
print(from_tuple)  # [10, 20, 30]

# จาก set (ไม่รับประกันลำดับ)
s = {3, 1, 4, 1, 5, 9}
from_set = list(s)
print(sorted(from_set))  # [1, 3, 4, 5, 9]
```

### 1.3 List Repetition และ Concatenation

```python
# Repetition
zeros = [0] * 5
print(zeros)  # [0, 0, 0, 0, 0]

pattern = [1, 2] * 3
print(pattern)  # [1, 2, 1, 2, 1, 2]

# Concatenation
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined = list1 + list2
print(combined)  # [1, 2, 3, 4, 5, 6]
```

---

## 2. Indexing และ Slicing

### 2.1 Positive Indexing

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]
#          0         1         2         3         4

print(fruits[0])   # apple
print(fruits[2])   # cherry
print(fruits[4])   # elderberry
print(len(fruits)) # 5
```

### 2.2 Negative Indexing

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]
#           -5       -4        -3        -2        -1

print(fruits[-1])  # elderberry
print(fruits[-2])  # date
print(fruits[-5])  # apple
```

### 2.3 Slicing [start:stop:step]

```python
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# [start:stop] - ไม่รวม stop
print(numbers[2:5])    # [2, 3, 4]
print(numbers[:4])     # [0, 1, 2, 3]  (start=0)
print(numbers[6:])     # [6, 7, 8, 9]  (stop=end)
print(numbers[:])      # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]  (copy)

# [start:stop:step]
print(numbers[::2])    # [0, 2, 4, 6, 8]  (ทีละ 2)
print(numbers[1::2])   # [1, 3, 5, 7, 9]
print(numbers[::-1])   # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]  (reverse)
print(numbers[8:1:-2]) # [8, 6, 4, 2]
```

### 2.4 Modifying with Slicing

```python
nums = [1, 2, 3, 4, 5]

# แทนที่ด้วย slice
nums[1:3] = [20, 30]
print(nums)  # [1, 20, 30, 4, 5]

# ลบด้วย slice
nums[2:4] = []
print(nums)  # [1, 20, 5]

# แทรกด้วย slice
nums[1:1] = [100, 200]
print(nums)  # [1, 100, 200, 20, 5]
```

---

## 3. List Methods ทั้งหมด

### 3.1 append() - เพิ่มที่ท้าย

```python
fruits = ["apple", "banana"]
fruits.append("cherry")
fruits.append("date")
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# append list (ใส่เป็น nested list)
fruits.append(["elderberry", "fig"])
print(fruits)  # ['apple', 'banana', 'cherry', 'date', ['elderberry', 'fig']]
print(len(fruits))  # 5
```

### 3.2 extend() - เพิ่มหลายรายการ

```python
fruits = ["apple", "banana"]
more_fruits = ["cherry", "date"]
fruits.extend(more_fruits)
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# extend กับ string
letters = ["a", "b"]
letters.extend("cde")
print(letters)  # ['a', 'b', 'c', 'd', 'e']

# extend กับ range
nums = [1, 2, 3]
nums.extend(range(4, 7))
print(nums)  # [1, 2, 3, 4, 5, 6]
```

### 3.3 insert() - แทรกตำแหน่งที่ต้องการ

```python
fruits = ["apple", "banana", "cherry"]
fruits.insert(1, "mango")  # แทรกที่ index 1
print(fruits)  # ['apple', 'mango', 'banana', 'cherry']

fruits.insert(0, "avocado")  # แทรกหน้าสุด
print(fruits)  # ['avocado', 'apple', 'mango', 'banana', 'cherry']

fruits.insert(100, "zucchini")  # index เกิน → ท้ายสุด
print(fruits)  # ['avocado', 'apple', 'mango', 'banana', 'cherry', 'zucchini']

fruits.insert(-1, "papaya")  # ก่อนตัวสุดท้าย
print(fruits)  # ['avocado', 'apple', 'mango', 'banana', 'cherry', 'papaya', 'zucchini']
```

### 3.4 remove() - ลบด้วยค่า (ตัวแรก)

```python
fruits = ["apple", "banana", "apple", "cherry"]
fruits.remove("apple")  # ลบ "apple" ตัวแรก
print(fruits)  # ['banana', 'apple', 'cherry']

# ถ้าไม่พบ → ValueError
try:
    fruits.remove("mango")
except ValueError as e:
    print(f"Error: {e}")  # Error: list.remove(x): x not in list
```

### 3.5 pop() - ลบและคืนค่า

```python
fruits = ["apple", "banana", "cherry", "date"]

# pop ท้ายสุด (default)
last = fruits.pop()
print(last)    # date
print(fruits)  # ['apple', 'banana', 'cherry']

# pop ตาม index
second = fruits.pop(1)
print(second)  # banana
print(fruits)  # ['apple', 'cherry']

# pop index ลบ
item = fruits.pop(-1)
print(item)    # cherry
```

### 3.6 clear() - ลบทั้งหมด

```python
numbers = [1, 2, 3, 4, 5]
numbers.clear()
print(numbers)   # []
print(len(numbers))  # 0
```

### 3.7 index() - หาตำแหน่ง

```python
fruits = ["apple", "banana", "cherry", "banana", "date"]

print(fruits.index("banana"))        # 1 (ตัวแรก)
print(fruits.index("banana", 2))     # 3 (เริ่มหาจาก index 2)
print(fruits.index("banana", 2, 5))  # 3 (ช่วง index 2-4)

try:
    print(fruits.index("mango"))
except ValueError:
    print("ไม่พบ mango ใน list")
```

### 3.8 count() - นับจำนวน

```python
numbers = [1, 2, 3, 2, 1, 2, 4, 1]
print(numbers.count(1))  # 3
print(numbers.count(2))  # 3
print(numbers.count(5))  # 0

words = ["hello", "world", "hello", "python"]
print(words.count("hello"))  # 2
```

### 3.9 sort() - เรียงลำดับ (in-place)

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
numbers.sort()
print(numbers)  # [1, 1, 2, 3, 4, 5, 6, 9]

# เรียงจากมากไปน้อย
numbers.sort(reverse=True)
print(numbers)  # [9, 6, 5, 4, 3, 2, 1, 1]

# เรียง string
words = ["banana", "apple", "cherry", "date"]
words.sort()
print(words)  # ['apple', 'banana', 'cherry', 'date']

# เรียงด้วย key
words.sort(key=len)  # เรียงตามความยาว
print(words)  # ['date', 'apple', 'banana', 'cherry']

words.sort(key=str.upper)  # เรียงโดยไม่สนตัวพิมพ์ใหญ่-เล็ก
print(words)  # ['apple', 'banana', 'cherry', 'date']
```

### 3.10 reverse() - กลับด้าน (in-place)

```python
numbers = [1, 2, 3, 4, 5]
numbers.reverse()
print(numbers)  # [5, 4, 3, 2, 1]

fruits = ["apple", "banana", "cherry"]
fruits.reverse()
print(fruits)  # ['cherry', 'banana', 'apple']
```

### 3.11 copy() - คัดลอก List

```python
original = [1, 2, 3, 4, 5]
copied = original.copy()
copied.append(6)

print(original)  # [1, 2, 3, 4, 5]  ไม่เปลี่ยน
print(copied)    # [1, 2, 3, 4, 5, 6]
```

---

## 4. List Comprehension

### 4.1 Basic List Comprehension

```python
# แบบ for loop ธรรมดา
squares_loop = []
for x in range(1, 11):
    squares_loop.append(x ** 2)

# แบบ List Comprehension (กระชับกว่า)
squares = [x ** 2 for x in range(1, 11)]
print(squares)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# คูณ 2
doubled = [x * 2 for x in [1, 2, 3, 4, 5]]
print(doubled)  # [2, 4, 6, 8, 10]

# ดึง string
fruits = ["apple", "banana", "cherry"]
upper_fruits = [f.upper() for f in fruits]
print(upper_fruits)  # ['APPLE', 'BANANA', 'CHERRY']

# สร้าง list จาก string
words = "the quick brown fox"
word_list = [word for word in words.split()]
print(word_list)  # ['the', 'quick', 'brown', 'fox']
```

### 4.2 Conditional List Comprehension

```python
# กรองเฉพาะเลขคู่
numbers = range(1, 21)
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

# กรองสตริงที่ยาวกว่า 5 ตัวอักษร
fruits = ["apple", "fig", "banana", "kiwi", "watermelon"]
long_fruits = [f for f in fruits if len(f) > 5]
print(long_fruits)  # ['banana', 'watermelon']

# if-else ใน comprehension
numbers = range(1, 11)
even_odd = ["even" if n % 2 == 0 else "odd" for n in numbers]
print(even_odd)  # ['odd', 'even', 'odd', 'even', ...]
```

### 4.3 Nested List Comprehension

```python
# สร้าง matrix 3x3
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]
for row in matrix:
    print(row)
# [1, 2, 3]
# [2, 4, 6]
# [3, 6, 9]

# Flatten nested list
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in nested for num in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Cartesian product
colors = ["red", "blue"]
sizes = ["S", "M", "L"]
products = [(c, s) for c in colors for s in sizes]
print(products)
# [('red', 'S'), ('red', 'M'), ('red', 'L'), ('blue', 'S'), ...]
```

### 4.4 Advanced Comprehension

```python
# หลาย condition
nums = range(1, 51)
fizzbuzz = [
    "FizzBuzz" if n % 15 == 0
    else "Fizz" if n % 3 == 0
    else "Buzz" if n % 5 == 0
    else str(n)
    for n in nums
]
print(fizzbuzz[:15])

# กรองและแปลงพร้อมกัน
data = ["10", "abc", "20", "xyz", "30"]
valid_nums = [int(x) for x in data if x.isdigit()]
print(valid_nums)  # [10, 20, 30]

# สร้าง dict จาก list comprehension (ดู part 13)
squares_dict = {x: x**2 for x in range(1, 6)}
print(squares_dict)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

---

## 5. Nested Lists (Lists ซ้อนกัน)

### 5.1 การสร้างและเข้าถึง Nested List

```python
# Matrix 2D
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[0])     # [1, 2, 3]
print(matrix[1][2])  # 6
print(matrix[-1][-1])  # 9

# แก้ไขค่า
matrix[1][1] = 99
print(matrix[1])  # [4, 99, 6]
```

### 5.2 วนซ้ำ Nested List

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# วิธีที่ 1: nested for loop
for row in matrix:
    for element in row:
        print(element, end=" ")
    print()

# วิธีที่ 2: enumerate
for i, row in enumerate(matrix):
    for j, val in enumerate(row):
        print(f"matrix[{i}][{j}] = {val}")
```

### 5.3 Transpose Matrix

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# วิธีที่ 1: ด้วย zip
transposed = [list(row) for row in zip(*matrix)]
print(transposed)
# [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# วิธีที่ 2: manual
rows = len(matrix)
cols = len(matrix[0])
transposed2 = [[matrix[r][c] for r in range(rows)] for c in range(cols)]
print(transposed2)
```

---

## 6. List as Stack and Queue

### 6.1 Stack (LIFO - Last In First Out)

```python
# Stack ด้วย List
stack = []

# Push
stack.append("first")
stack.append("second")
stack.append("third")
print(stack)  # ['first', 'second', 'third']

# Pop (จากบน)
top = stack.pop()
print(top)    # third
print(stack)  # ['first', 'second']

# Peek (ดูค่าบนสุด)
peek = stack[-1]
print(peek)   # second

# ตัวอย่าง: ตรวจสอบวงเล็บ
def check_brackets(expression):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    
    for char in expression:
        if char in '([{':
            stack.append(char)
        elif char in ')]}':
            if not stack or stack[-1] != pairs[char]:
                return False
            stack.pop()
    
    return len(stack) == 0

print(check_brackets("(1 + 2) * [3 + 4]"))  # True
print(check_brackets("(1 + 2] * [3 + 4)"))  # False
```

### 6.2 Queue (FIFO - First In First Out)

```python
from collections import deque

# Queue ด้วย deque (efficient กว่า list)
queue = deque()

# Enqueue
queue.append("first")
queue.append("second")
queue.append("third")
print(queue)  # deque(['first', 'second', 'third'])

# Dequeue (จากหน้า)
front = queue.popleft()
print(front)   # first
print(queue)   # deque(['second', 'third'])

# Queue ด้วย list (ไม่แนะนำสำหรับข้อมูลมาก - pop(0) ช้า)
queue_list = ["first", "second", "third"]
queue_list.append("fourth")
dequeued = queue_list.pop(0)
print(dequeued)  # first
```

---

## 7. Copying Lists (Shallow vs Deep Copy)

### 7.1 Assignment (ไม่ใช่การ copy!)

```python
original = [1, 2, 3, [4, 5]]
reference = original  # เป็น reference เดียวกัน

reference.append(6)
print(original)   # [1, 2, 3, [4, 5], 6]  เปลี่ยนด้วย!
print(reference)  # [1, 2, 3, [4, 5], 6]
print(original is reference)  # True
```

### 7.2 Shallow Copy

```python
import copy

original = [1, 2, 3, [4, 5]]

# วิธีที่ 1: .copy()
shallow1 = original.copy()

# วิธีที่ 2: [:]
shallow2 = original[:]

# วิธีที่ 3: list()
shallow3 = list(original)

# วิธีที่ 4: copy.copy()
shallow4 = copy.copy(original)

# ทดสอบ shallow copy
shallow1.append(6)
print(original)  # [1, 2, 3, [4, 5]]  ไม่เปลี่ยน
print(shallow1)  # [1, 2, 3, [4, 5], 6]

# แต่! nested list ยังเชื่อมกันอยู่
shallow1[3].append(99)
print(original)  # [1, 2, 3, [4, 5, 99]]  เปลี่ยน!
print(shallow1)  # [1, 2, 3, [4, 5, 99], 6]
```

### 7.3 Deep Copy

```python
import copy

original = [1, 2, 3, [4, 5, [6, 7]]]
deep = copy.deepcopy(original)

# แก้ไข nested list ใน deep copy
deep[3].append(99)
deep[3][2].append(100)

print(original)  # [1, 2, 3, [4, 5, [6, 7]]]  ไม่เปลี่ยน!
print(deep)      # [1, 2, 3, [4, 5, [6, 7, 100], 99]]
```

---

## 8. Sorting

### 8.1 sort() vs sorted()

```python
# sort() - in-place, แก้ไข list เดิม, คืน None
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
result = numbers.sort()
print(result)   # None
print(numbers)  # [1, 1, 2, 3, 4, 5, 6, 9]

# sorted() - คืน list ใหม่, list เดิมไม่เปลี่ยน
numbers2 = [3, 1, 4, 1, 5, 9, 2, 6]
sorted_nums = sorted(numbers2)
print(sorted_nums)  # [1, 1, 2, 3, 4, 5, 6, 9]
print(numbers2)     # [3, 1, 4, 1, 5, 9, 2, 6]  ไม่เปลี่ยน!
```

### 8.2 Key Parameter

```python
# เรียงตามความยาว string
words = ["banana", "fig", "apple", "cherry", "date"]
sorted_by_len = sorted(words, key=len)
print(sorted_by_len)  # ['fig', 'date', 'apple', 'banana', 'cherry']

# เรียงตามตัวอักษรสุดท้าย
sorted_by_last = sorted(words, key=lambda w: w[-1])
print(sorted_by_last)

# เรียง list ของ tuple
students = [("Alice", 85), ("Bob", 92), ("Charlie", 78), ("Diana", 95)]

# เรียงตามคะแนน
by_score = sorted(students, key=lambda s: s[1])
print(by_score)

# เรียงตามชื่อ
by_name = sorted(students, key=lambda s: s[0])
print(by_name)

# เรียงหลายเกณฑ์
data = [("Bob", 85), ("Alice", 85), ("Charlie", 90)]
multi_sorted = sorted(data, key=lambda x: (-x[1], x[0]))  # คะแนนมาก→น้อย, ชื่อ A→Z
print(multi_sorted)
```

### 8.3 Custom Comparison ด้วย functools.cmp_to_key

```python
from functools import cmp_to_key

def compare(a, b):
    if len(a) < len(b):
        return -1
    elif len(a) > len(b):
        return 1
    else:
        return (a > b) - (a < b)  # เรียง A-Z ถ้ายาวเท่ากัน

words = ["banana", "fig", "apple", "cherry"]
sorted_words = sorted(words, key=cmp_to_key(compare))
print(sorted_words)  # ['fig', 'apple', 'banana', 'cherry']
```

---

## 9. Filter และ Map กับ Lists

### 9.1 filter()

```python
# filter(function, iterable) → คืน filter object
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# กรองเฉพาะเลขคู่
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# กรอง None values
mixed = [1, None, 2, None, 3, 0, "", "hello"]
cleaned = list(filter(None, mixed))  # กรองค่า falsy
print(cleaned)  # [1, 2, 3, 'hello']

# กรองด้วย named function
def is_positive(n):
    return n > 0

nums = [-3, -1, 0, 2, 5, -2, 8]
positives = list(filter(is_positive, nums))
print(positives)  # [2, 5, 8]
```

### 9.2 map()

```python
# map(function, iterable) → คืน map object
numbers = [1, 2, 3, 4, 5]

# ยกกำลังสอง
squares = list(map(lambda x: x**2, numbers))
print(squares)  # [1, 4, 9, 16, 25]

# แปลง string เป็น int
str_nums = ["10", "20", "30", "40"]
int_nums = list(map(int, str_nums))
print(int_nums)  # [10, 20, 30, 40]

# map หลาย iterable
a = [1, 2, 3]
b = [10, 20, 30]
summed = list(map(lambda x, y: x + y, a, b))
print(summed)  # [11, 22, 33]
```

---

## 10. List Unpacking

### 10.1 Basic Unpacking

```python
# Unpacking ปกติ (จำนวนต้องตรงกัน)
a, b, c = [1, 2, 3]
print(a, b, c)  # 1 2 3

# Swap variables
x, y = 10, 20
x, y = y, x
print(x, y)  # 20 10

# Unpack ใน loop
pairs = [(1, "one"), (2, "two"), (3, "three")]
for num, word in pairs:
    print(f"{num} = {word}")
```

### 10.2 Star Unpacking (*)

```python
# เก็บส่วนที่เหลือ
first, *rest = [1, 2, 3, 4, 5]
print(first)  # 1
print(rest)   # [2, 3, 4, 5]

*start, last = [1, 2, 3, 4, 5]
print(start)  # [1, 2, 3, 4]
print(last)   # 5

first, *middle, last = [1, 2, 3, 4, 5]
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5

# Star ใน function call
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
result = add(*nums)
print(result)  # 6
```

---

## 11. โปรแกรมจริง (Real-world Examples)

### 11.1 Student Grade System

```python
def student_grade_system():
    """ระบบจัดการเกรดนักเรียน"""
    students = []
    
    def add_student(name, grades):
        """เพิ่มนักเรียน"""
        avg = sum(grades) / len(grades)
        grade_letter = get_grade_letter(avg)
        students.append({
            "name": name,
            "grades": grades,
            "average": avg,
            "grade": grade_letter
        })
    
    def get_grade_letter(avg):
        """แปลงคะแนนเป็นเกรด"""
        if avg >= 90: return "A"
        elif avg >= 80: return "B"
        elif avg >= 70: return "C"
        elif avg >= 60: return "D"
        else: return "F"
    
    def get_top_students(n=3):
        """ดึง n นักเรียนที่มีคะแนนสูงสุด"""
        sorted_students = sorted(students, key=lambda s: s["average"], reverse=True)
        return sorted_students[:n]
    
    def get_class_average():
        """คำนวณเฉลี่ยทั้งห้อง"""
        if not students:
            return 0
        return sum(s["average"] for s in students) / len(students)
    
    def get_grade_distribution():
        """สรุปการกระจายเกรด"""
        distribution = {}
        for s in students:
            grade = s["grade"]
            distribution[grade] = distribution.get(grade, 0) + 1
        return distribution
    
    def print_report():
        """แสดงรายงาน"""
        print("=" * 50)
        print("STUDENT GRADE REPORT")
        print("=" * 50)
        for s in sorted(students, key=lambda x: x["name"]):
            grades_str = ", ".join(map(str, s["grades"]))
            print(f"{s['name']:15} | Avg: {s['average']:5.1f} | Grade: {s['grade']} | Scores: [{grades_str}]")
        print("-" * 50)
        print(f"Class Average: {get_class_average():.1f}")
        print(f"\nTop 3 Students:")
        for i, s in enumerate(get_top_students(3), 1):
            print(f"  {i}. {s['name']} ({s['average']:.1f})")
        print(f"\nGrade Distribution: {get_grade_distribution()}")
    
    # เพิ่มข้อมูลนักเรียน
    add_student("Alice", [95, 88, 92, 97, 85])
    add_student("Bob", [72, 68, 75, 70, 78])
    add_student("Charlie", [85, 90, 88, 92, 87])
    add_student("Diana", [60, 65, 58, 62, 70])
    add_student("Eve", [98, 95, 99, 97, 96])
    
    print_report()

student_grade_system()
```

### 11.2 Shopping Cart

```python
class ShoppingCart:
    """ตะกร้าสินค้าออนไลน์"""
    
    def __init__(self):
        self.items = []
    
    def add_item(self, name, price, quantity=1):
        """เพิ่มสินค้า"""
        for item in self.items:
            if item["name"] == name:
                item["quantity"] += quantity
                print(f"อัปเดต {name}: {item['quantity']} ชิ้น")
                return
        self.items.append({"name": name, "price": price, "quantity": quantity})
        print(f"เพิ่ม {name} x{quantity} (฿{price:.2f})")
    
    def remove_item(self, name):
        """ลบสินค้า"""
        self.items = [item for item in self.items if item["name"] != name]
        print(f"ลบ {name} ออกจากตะกร้า")
    
    def update_quantity(self, name, quantity):
        """อัปเดตจำนวน"""
        for item in self.items:
            if item["name"] == name:
                if quantity <= 0:
                    self.remove_item(name)
                else:
                    item["quantity"] = quantity
                return
        print(f"ไม่พบ {name} ในตะกร้า")
    
    def get_total(self):
        """คำนวณยอดรวม"""
        return sum(item["price"] * item["quantity"] for item in self.items)
    
    def get_item_count(self):
        """จำนวนสินค้าทั้งหมด"""
        return sum(item["quantity"] for item in self.items)
    
    def apply_discount(self, percent):
        """ลดราคา"""
        discount = self.get_total() * (percent / 100)
        return self.get_total() - discount
    
    def sort_by_price(self, reverse=False):
        """เรียงตามราคา"""
        self.items.sort(key=lambda x: x["price"], reverse=reverse)
    
    def print_cart(self):
        """แสดงตะกร้า"""
        print("\n" + "=" * 55)
        print(f"{'สินค้า':20} {'ราคา':10} {'จำนวน':8} {'รวม':10}")
        print("-" * 55)
        for item in self.items:
            subtotal = item["price"] * item["quantity"]
            print(f"{item['name']:20} ฿{item['price']:8.2f} {item['quantity']:8} ฿{subtotal:8.2f}")
        print("-" * 55)
        total = self.get_total()
        print(f"{'ยอดรวม':38} ฿{total:8.2f}")
        print(f"{'หลังลด 10%':38} ฿{self.apply_discount(10):8.2f}")
        print(f"จำนวนสินค้าทั้งหมด: {self.get_item_count()} ชิ้น")
        print("=" * 55)

# ทดสอบ
cart = ShoppingCart()
cart.add_item("แอปเปิ้ล", 25.00, 3)
cart.add_item("กล้วย", 15.00, 6)
cart.add_item("ส้ม", 30.00, 2)
cart.add_item("แอปเปิ้ล", 25.00, 2)  # เพิ่มของที่มีอยู่แล้ว
cart.update_quantity("กล้วย", 4)
cart.print_cart()
```

### 11.3 Data Manipulation

```python
def data_manipulation_demo():
    """ตัวอย่างการจัดการข้อมูล"""
    
    # ข้อมูลยอดขายรายวัน
    sales_data = [
        {"date": "2024-01-01", "product": "A", "amount": 1500},
        {"date": "2024-01-01", "product": "B", "amount": 2300},
        {"date": "2024-01-02", "product": "A", "amount": 1800},
        {"date": "2024-01-02", "product": "C", "amount": 900},
        {"date": "2024-01-03", "product": "B", "amount": 2100},
        {"date": "2024-01-03", "product": "A", "amount": 1200},
        {"date": "2024-01-04", "product": "C", "amount": 1400},
        {"date": "2024-01-04", "product": "B", "amount": 2800},
    ]
    
    # 1. ยอดขายรวมทั้งหมด
    total = sum(s["amount"] for s in sales_data)
    print(f"ยอดขายรวม: ฿{total:,}")
    
    # 2. ยอดขายเฉลี่ยต่อวัน
    dates = list(set(s["date"] for s in sales_data))
    daily_totals = [
        sum(s["amount"] for s in sales_data if s["date"] == d)
        for d in sorted(dates)
    ]
    print(f"ยอดขายเฉลี่ยต่อวัน: ฿{sum(daily_totals)/len(daily_totals):,.0f}")
    
    # 3. สินค้าขายดีสุด
    products = list(set(s["product"] for s in sales_data))
    product_totals = [
        (p, sum(s["amount"] for s in sales_data if s["product"] == p))
        for p in products
    ]
    best_product = max(product_totals, key=lambda x: x[1])
    print(f"สินค้าขายดีสุด: {best_product[0]} (฿{best_product[1]:,})")
    
    # 4. เรียงตามยอดขาย
    sorted_data = sorted(sales_data, key=lambda x: x["amount"], reverse=True)
    print("\nTop 3 ยอดขายสูงสุด:")
    for i, s in enumerate(sorted_data[:3], 1):
        print(f"  {i}. {s['date']} - Product {s['product']}: ฿{s['amount']:,}")
    
    # 5. Filter เฉพาะที่ยอดขายสูงกว่า 1500
    high_sales = [s for s in sales_data if s["amount"] > 1500]
    print(f"\nรายการที่ยอดขาย > 1500: {len(high_sales)} รายการ")

data_manipulation_demo()
```

---

## 12. เทคนิคขั้นสูง (Advanced Techniques)

### 12.1 zip() กับ Lists

```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
grades = ["B", "A", "C"]

# รวม 3 lists
combined = list(zip(names, scores, grades))
print(combined)
# [('Alice', 85, 'B'), ('Bob', 92, 'A'), ('Charlie', 78, 'C')]

# Unzip
n, s, g = zip(*combined)
print(list(n))  # ['Alice', 'Bob', 'Charlie']

# สร้าง dict จาก 2 lists
keys = ["name", "age", "city"]
values = ["Alice", 25, "Bangkok"]
d = dict(zip(keys, values))
print(d)  # {'name': 'Alice', 'age': 25, 'city': 'Bangkok'}
```

### 12.2 enumerate() กับ Lists

```python
fruits = ["apple", "banana", "cherry"]

# พื้นฐาน
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# กำหนด start
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}. {fruit}")

# สร้าง list ของ tuples (index, value)
indexed = list(enumerate(fruits))
print(indexed)  # [(0, 'apple'), (1, 'banana'), (2, 'cherry')]
```

### 12.3 any() และ all() กับ Lists

```python
numbers = [2, 4, 6, 8, 10]

# any() - คืน True ถ้าอย่างน้อย 1 ตัวเป็น True
print(any(n > 5 for n in numbers))   # True
print(any(n > 20 for n in numbers))  # False

# all() - คืน True ถ้าทุกตัวเป็น True
print(all(n % 2 == 0 for n in numbers))  # True (ทุกตัวเป็นเลขคู่)
print(all(n > 5 for n in numbers))       # False (2 และ 4 ไม่ > 5)

# ตัวอย่างใช้งานจริง
passwords = ["pass1", "pass123", "secure_pass_456"]
all_valid = all(len(p) >= 8 for p in passwords)
print(f"ทุก password ยาวอย่างน้อย 8: {all_valid}")
```

### 12.4 List เป็น Iterable ใน Functions

```python
def process_numbers(numbers):
    """ประมวลผล list of numbers"""
    if not numbers:
        return {"sum": 0, "avg": 0, "min": None, "max": None}
    
    return {
        "sum": sum(numbers),
        "avg": sum(numbers) / len(numbers),
        "min": min(numbers),
        "max": max(numbers),
        "sorted": sorted(numbers),
        "unique": list(set(numbers)),
        "count": len(numbers)
    }

data = [5, 2, 8, 1, 9, 3, 7, 4, 6, 5, 2, 8]
result = process_numbers(data)
for key, value in result.items():
    print(f"{key}: {value}")
```

---

## แบบฝึกหัด (Exercises)

### ข้อ 1: Reverse Words
เขียนฟังก์ชัน `reverse_words(sentence)` ที่รับ string แล้วคืน string ที่กลับลำดับคำ

```python
def reverse_words(sentence):
    words = sentence.split()
    words.reverse()
    return " ".join(words)

# ทดสอบ
print(reverse_words("Hello World Python"))  # Python World Hello
print(reverse_words("I love coding"))       # coding love I
```

### ข้อ 2: Remove Duplicates
เขียนฟังก์ชัน `remove_duplicates(lst)` ที่ลบ element ซ้ำโดยรักษาลำดับเดิม

```python
def remove_duplicates(lst):
    seen = []
    return [x for x in lst if x not in seen and not seen.append(x)]

# หรือวิธีที่ดีกว่า (ใช้ dict เพื่อ performance)
def remove_duplicates_v2(lst):
    seen = {}
    return [seen.setdefault(x, x) for x in lst if x not in seen]

print(remove_duplicates([1, 2, 3, 2, 1, 4, 3, 5]))  # [1, 2, 3, 4, 5]
```

### ข้อ 3: Flatten Nested List
เขียนฟังก์ชัน `flatten(nested)` ที่แปลง nested list ให้เป็น flat list

```python
def flatten(nested):
    result = []
    for item in nested:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

print(flatten([1, [2, 3], [4, [5, 6]], 7]))  # [1, 2, 3, 4, 5, 6, 7]
```

### ข้อ 4: Chunk List
เขียนฟังก์ชัน `chunk(lst, size)` แบ่ง list เป็น chunks ขนาดที่กำหนด

```python
def chunk(lst, size):
    return [lst[i:i+size] for i in range(0, len(lst), size)]

print(chunk([1,2,3,4,5,6,7,8,9], 3))
# [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### ข้อ 5: Rotate List
เขียนฟังก์ชัน `rotate(lst, n)` ที่หมุน list ไปทางขวา n ตำแหน่ง

```python
def rotate(lst, n):
    if not lst:
        return lst
    n = n % len(lst)
    return lst[-n:] + lst[:-n]

print(rotate([1,2,3,4,5], 2))  # [4, 5, 1, 2, 3]
print(rotate([1,2,3,4,5], 7))  # [4, 5, 1, 2, 3]
```

### ข้อ 6: Matrix Multiplication
เขียนฟังก์ชัน `matrix_multiply(A, B)` คูณ matrix 2 ตัว

```python
def matrix_multiply(A, B):
    rows_A, cols_A = len(A), len(A[0])
    cols_B = len(B[0])
    result = [[0] * cols_B for _ in range(rows_A)]
    for i in range(rows_A):
        for j in range(cols_B):
            for k in range(cols_A):
                result[i][j] += A[i][k] * B[k][j]
    return result

A = [[1, 2], [3, 4]]
B = [[5, 6], [7, 8]]
print(matrix_multiply(A, B))  # [[19, 22], [43, 50]]
```

### ข้อ 7: Find Pairs
หาคู่ตัวเลขทั้งหมดใน list ที่รวมกันได้ target

```python
def find_pairs(nums, target):
    pairs = []
    seen = set()
    for num in nums:
        complement = target - num
        if complement in seen:
            pairs.append((complement, num))
        seen.add(num)
    return pairs

print(find_pairs([1, 2, 3, 4, 5, 6, 7, 8, 9], 10))
# [(1, 9), (2, 8), (3, 7), (4, 6)]
```

### ข้อ 8: Moving Average
คำนวณ moving average ของ list ตัวเลข

```python
def moving_average(nums, window):
    return [
        round(sum(nums[i:i+window]) / window, 2)
        for i in range(len(nums) - window + 1)
    ]

prices = [10, 12, 11, 14, 13, 15, 16, 14, 17, 18]
print(moving_average(prices, 3))  # [11.0, 12.33, 12.67, 14.0, 14.67, 15.0, 15.67, 16.33]
```

### ข้อ 9: Anagram Groups
จัดกลุ่ม words ที่เป็น anagram ของกันและกัน

```python
def group_anagrams(words):
    groups = {}
    for word in words:
        key = "".join(sorted(word))
        groups.setdefault(key, []).append(word)
    return list(groups.values())

words = ["eat", "tea", "tan", "ate", "nat", "bat"]
print(group_anagrams(words))
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

### ข้อ 10: Inventory System
สร้างระบบ inventory อย่างง่าย

```python
class Inventory:
    def __init__(self):
        self.items = []
    
    def add_item(self, name, quantity, price):
        self.items.append({"name": name, "qty": quantity, "price": price})
    
    def get_low_stock(self, threshold=5):
        return [i for i in self.items if i["qty"] <= threshold]
    
    def get_total_value(self):
        return sum(i["qty"] * i["price"] for i in self.items)
    
    def search(self, keyword):
        return [i for i in self.items if keyword.lower() in i["name"].lower()]
    
    def sort_by(self, field, reverse=False):
        return sorted(self.items, key=lambda x: x[field], reverse=reverse)

inv = Inventory()
inv.add_item("Laptop", 3, 25000)
inv.add_item("Mouse", 15, 500)
inv.add_item("Keyboard", 4, 800)
inv.add_item("Monitor", 2, 8000)

print("สินค้าใกล้หมด:", inv.get_low_stock())
print(f"มูลค่าสินค้าทั้งหมด: ฿{inv.get_total_value():,}")
print("ค้นหา 'key':", inv.search("key"))
```

---

## สรุป (Summary)

| Method | การใช้งาน | Return |
|--------|-----------|--------|
| `append(x)` | เพิ่มท้าย | None |
| `extend(iterable)` | เพิ่มหลายตัว | None |
| `insert(i, x)` | แทรกที่ตำแหน่ง | None |
| `remove(x)` | ลบตัวแรกที่เจอ | None |
| `pop([i])` | ลบและคืนค่า | element |
| `clear()` | ล้างทั้งหมด | None |
| `index(x)` | หาตำแหน่ง | int |
| `count(x)` | นับจำนวน | int |
| `sort()` | เรียง in-place | None |
| `reverse()` | กลับลำดับ | None |
| `copy()` | shallow copy | list |

**เมื่อไหรควรใช้ List:**
- เมื่อต้องการ ordered collection ที่แก้ไขได้
- เมื่อต้องการข้อมูลซ้ำกันได้
- เมื่อต้องการ stack หรือ queue (แต่ถ้า queue ใหญ่ใช้ deque)
- เมื่อต้องการ iterate และ modify ข้อมูล

> **หมายเหตุ:** Part ต่อไปจะเรียนเรื่อง Tuples ซึ่งคล้าย List แต่ immutable (ไม่สามารถแก้ไขได้)
