# Part 13: Dictionaries - Complete Guide

## บทนำ (Introduction)

**Dictionary** เป็นโครงสร้างข้อมูลแบบ **key-value pairs** ที่ใช้กันมากที่สุดใน Python
ตั้งแต่ Python 3.7+ dictionary รับประกัน insertion order (ordered)

### คุณสมบัติหลักของ Dictionary
| คุณสมบัติ | ความหมาย |
|-----------|-----------|
| Key-Value pairs | เก็บข้อมูลเป็นคู่ key:value |
| Mutable | แก้ไขได้หลังสร้าง |
| Keys ต้อง unique | ไม่มี key ซ้ำ |
| Keys ต้อง hashable | ใช้ int, str, tuple เป็น key ได้ |
| Ordered (3.7+) | เรียงตาม insertion order |
| Fast lookup | O(1) average case |

---

## 1. การสร้าง Dictionary

### 1.1 วิธีพื้นฐาน

```python
# วิธีที่ 1: Curly braces {}
person = {"name": "Alice", "age": 25, "city": "Bangkok"}
empty_dict = {}

# วิธีที่ 2: dict() constructor
person2 = dict(name="Bob", age=30, city="Chiang Mai")

# วิธีที่ 3: จาก list ของ tuples
items = [("apple", 10), ("banana", 5), ("cherry", 20)]
fruit_count = dict(items)

# วิธีที่ 4: dict.fromkeys()
keys = ["a", "b", "c", "d"]
defaults = dict.fromkeys(keys, 0)  # ทุก key มีค่าเริ่มต้นเป็น 0
print(defaults)  # {'a': 0, 'b': 0, 'c': 0, 'd': 0}

# วิธีที่ 5: zip()
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
score_dict = dict(zip(names, scores))
print(score_dict)  # {'Alice': 85, 'Bob': 92, 'Charlie': 78}
```

### 1.2 Mixed Key Types

```python
# Key ได้หลายประเภท (ต้อง hashable)
mixed = {
    1: "integer key",
    "name": "string key",
    (1, 2): "tuple key",
    True: "bool key"
}

print(mixed[1])       # integer key (True == 1 ทำให้ overwrite)
print(mixed["name"])  # string key
print(mixed[(1, 2)])  # tuple key

# ระวัง: True == 1 ใน Python
d = {1: "one", True: "true"}
print(d)  # {1: 'true'}  ← True overwrite 1!
```

---

## 2. Accessing, Adding, Updating, Deleting

### 2.1 Accessing

```python
person = {"name": "Alice", "age": 25, "city": "Bangkok"}

# วิธีที่ 1: [] - ถ้าไม่มี key จะ raise KeyError
print(person["name"])  # Alice

try:
    print(person["phone"])
except KeyError as e:
    print(f"Key not found: {e}")

# วิธีที่ 2: .get() - ถ้าไม่มีคืน None หรือค่า default
print(person.get("phone"))            # None
print(person.get("phone", "N/A"))     # N/A
print(person.get("name", "Unknown"))  # Alice
```

### 2.2 Adding และ Updating

```python
person = {"name": "Alice", "age": 25}

# เพิ่ม key ใหม่
person["city"] = "Bangkok"
person["email"] = "alice@example.com"
print(person)

# อัปเดต key ที่มีอยู่
person["age"] = 26
print(person["age"])  # 26

# update() - อัปเดตหลาย key พร้อมกัน
person.update({"age": 27, "phone": "080-123-4567"})
person.update(country="Thailand", job="Engineer")
print(person)
```

### 2.3 Deleting

```python
person = {"name": "Alice", "age": 25, "city": "Bangkok", "email": "alice@ex.com"}

# del - ลบ key (ถ้าไม่มีจะ raise KeyError)
del person["email"]
print(person)

# pop() - ลบและคืนค่า
age = person.pop("age")
print(f"Removed age: {age}")
print(person)

# pop() พร้อม default (ไม่ raise error ถ้าไม่มี key)
phone = person.pop("phone", None)
print(f"Phone: {phone}")  # None

# popitem() - ลบและคืน key-value สุดท้าย (Python 3.7+)
last_item = person.popitem()
print(f"Removed last: {last_item}")

# clear() - ล้างทั้งหมด
person.clear()
print(person)  # {}
```

---

## 3. Dictionary Methods ทั้งหมด

### 3.1 keys(), values(), items()

```python
student = {"name": "Bob", "math": 85, "science": 90, "english": 78}

# keys() - คืน view object ของ keys
keys = student.keys()
print(keys)        # dict_keys(['name', 'math', 'science', 'english'])
print(list(keys))  # ['name', 'math', 'science', 'english']

# values() - คืน view object ของ values
values = student.values()
print(values)        # dict_values(['Bob', 85, 90, 78])
print(list(values))

# items() - คืน view object ของ (key, value) tuples
items = student.items()
print(items)

# วน loop ด้วย items()
for key, value in student.items():
    print(f"{key}: {value}")

# View objects เป็น live view (เปลี่ยนตาม dict)
student["history"] = 88
print(keys)  # dict_keys(['name', 'math', 'science', 'english', 'history'])
```

### 3.2 get() และ setdefault()

```python
config = {"debug": True, "port": 8080}

# get() - อ่านอย่างปลอดภัย
debug = config.get("debug", False)
host = config.get("host", "localhost")
print(debug, host)  # True localhost

# setdefault() - ถ้าไม่มี key ให้เพิ่มด้วย default value
# ถ้ามี key แล้ว ไม่เปลี่ยนแปลง
result = config.setdefault("timeout", 30)
print(result)   # 30 (ค่าใหม่)
print(config)   # มี timeout=30 เพิ่มมา

result2 = config.setdefault("port", 9999)
print(result2)  # 8080 (ค่าเดิม ไม่เปลี่ยน)
print(config["port"])  # 8080
```

### 3.3 update()

```python
dict1 = {"a": 1, "b": 2, "c": 3}

# update จาก dict
dict1.update({"c": 30, "d": 4})
print(dict1)  # {'a': 1, 'b': 2, 'c': 30, 'd': 4}

# update จาก kwargs
dict1.update(e=5, f=6)
print(dict1)

# update จาก list of tuples
dict1.update([("g", 7), ("h", 8)])
print(dict1)
```

### 3.4 copy()

```python
original = {"a": 1, "b": [1, 2, 3]}

# Shallow copy
shallow = original.copy()
shallow["a"] = 99
print(original["a"])  # 1 (ไม่เปลี่ยน)

# แต่ nested object ยังเชื่อมกัน
shallow["b"].append(4)
print(original["b"])  # [1, 2, 3, 4] (เปลี่ยน!)

# Deep copy
import copy
deep = copy.deepcopy(original)
deep["b"].append(99)
print(original["b"])  # [1, 2, 3, 4] (ไม่เปลี่ยน)
```

---

## 4. Dictionary Comprehension

### 4.1 Basic Comprehension

```python
# {key: value for item in iterable}
squares = {x: x**2 for x in range(1, 11)}
print(squares)  # {1: 1, 2: 4, 3: 9, ...}

# แปลง list เป็น dict
fruits = ["apple", "banana", "cherry"]
fruit_lengths = {f: len(f) for f in fruits}
print(fruit_lengths)  # {'apple': 5, 'banana': 6, 'cherry': 6}

# จาก zip
keys = ["name", "age", "city"]
values = ["Alice", 25, "Bangkok"]
person = {k: v for k, v in zip(keys, values)}
print(person)
```

### 4.2 Conditional Comprehension

```python
# กรองด้วย if
scores = {"Alice": 85, "Bob": 45, "Charlie": 72, "Diana": 95, "Eve": 38}

# เฉพาะที่ผ่าน (>= 60)
passed = {name: score for name, score in scores.items() if score >= 60}
print(passed)

# แปลงค่าพร้อมกัน
grades = {
    name: "Pass" if score >= 60 else "Fail"
    for name, score in scores.items()
}
print(grades)

# กรองและแปลง
high_scores = {
    name: f"{score:.0f}%"
    for name, score in scores.items()
    if score >= 70
}
print(high_scores)
```

### 4.3 Nested และ Advanced

```python
# Invert dict (swap key-value)
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}

# Nested comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat_dict = {f"({i},{j})": matrix[i][j]
             for i in range(len(matrix))
             for j in range(len(matrix[0]))}
print(flat_dict)

# Group by
words = ["apple", "ant", "banana", "bear", "cat", "cherry"]
by_first_letter = {
    letter: [w for w in words if w[0] == letter]
    for letter in set(w[0] for w in words)
}
print(by_first_letter)
```

---

## 5. Nested Dictionaries

### 5.1 สร้างและเข้าถึง

```python
# Nested dict
company = {
    "Engineering": {
        "Alice": {"salary": 80000, "level": "Senior"},
        "Bob": {"salary": 60000, "level": "Junior"},
    },
    "Marketing": {
        "Charlie": {"salary": 55000, "level": "Senior"},
    }
}

# เข้าถึง
print(company["Engineering"]["Alice"]["salary"])  # 80000
print(company["Marketing"]["Charlie"]["level"])   # Senior

# ใช้ get() เพื่อความปลอดภัย
emp_data = company.get("HR", {}).get("Dave", {})
print(emp_data)  # {} (ไม่พบแต่ไม่ error)

# แก้ไข
company["Engineering"]["Alice"]["salary"] = 85000
company["Engineering"]["Dave"] = {"salary": 70000, "level": "Mid"}
```

### 5.2 วน loop Nested Dict

```python
company = {
    "Engineering": {"Alice": 80000, "Bob": 60000},
    "Marketing": {"Charlie": 55000},
    "HR": {"Diana": 50000, "Eve": 48000}
}

# วน loop ทั้งหมด
total_salary = 0
for dept, employees in company.items():
    print(f"\n{dept}:")
    for name, salary in employees.items():
        print(f"  {name}: ฿{salary:,}")
        total_salary += salary

print(f"\nTotal salary: ฿{total_salary:,}")

# Flatten nested dict
def flatten_dict(nested, prefix=""):
    result = {}
    for key, value in nested.items():
        full_key = f"{prefix}.{key}" if prefix else key
        if isinstance(value, dict):
            result.update(flatten_dict(value, full_key))
        else:
            result[full_key] = value
    return result

config = {"db": {"host": "localhost", "port": 5432}, "app": {"debug": True}}
flat = flatten_dict(config)
print(flat)  # {'db.host': 'localhost', 'db.port': 5432, 'app.debug': True}
```

---

## 6. collections.defaultdict

### 6.1 DefaultDict พื้นฐาน

```python
from collections import defaultdict

# ปัญหากับ dict ปกติ
regular = {}
try:
    regular["count"] += 1
except KeyError:
    print("KeyError!")  # regular dict ไม่รู้จัก key ใหม่

# defaultdict แก้ปัญหา
counter = defaultdict(int)  # default value = 0
counter["apple"] += 1
counter["apple"] += 1
counter["banana"] += 1
print(dict(counter))  # {'apple': 2, 'banana': 1}

# defaultdict(list)
groups = defaultdict(list)
data = [("a", 1), ("b", 2), ("a", 3), ("b", 4), ("c", 5)]
for key, val in data:
    groups[key].append(val)
print(dict(groups))  # {'a': [1, 3], 'b': [2, 4], 'c': [5]}

# defaultdict(set)
friends = defaultdict(set)
relationships = [("Alice", "Bob"), ("Alice", "Charlie"), ("Bob", "Diana"), ("Bob", "Alice")]
for person, friend in relationships:
    friends[person].add(friend)
print(dict(friends))
```

### 6.2 ตัวอย่างจริง: Word Frequency

```python
from collections import defaultdict

def word_frequency(text):
    """นับความถี่ของคำ"""
    freq = defaultdict(int)
    words = text.lower().split()
    for word in words:
        # ลบ punctuation
        word = ''.join(c for c in word if c.isalpha())
        if word:
            freq[word] += 1
    return dict(freq)

text = """
Python is a programming language that lets you work quickly
and integrate systems more effectively. Python is easy to learn.
Python is powerful.
"""

freq = word_frequency(text)
# เรียงตามความถี่
sorted_freq = sorted(freq.items(), key=lambda x: x[1], reverse=True)
print("Top 10 words:")
for word, count in sorted_freq[:10]:
    print(f"  {word:15}: {count}")
```

---

## 7. collections.OrderedDict

```python
from collections import OrderedDict

# Python 3.7+ dict ปกติก็ ordered แล้ว
# แต่ OrderedDict มีฟีเจอร์พิเศษ

od = OrderedDict()
od["first"] = 1
od["second"] = 2
od["third"] = 3

print(od)

# move_to_end()
od.move_to_end("first")  # ย้ายไปท้าย
print(od)

od.move_to_end("third", last=False)  # ย้ายไปหน้า
print(od)

# popitem() - ลบ first หรือ last
last = od.popitem(last=True)    # ลบท้าย
first = od.popitem(last=False)  # ลบหน้า
print(last, first)

# OrderedDict เปรียบเทียบตามลำดับ
d1 = OrderedDict([("a", 1), ("b", 2)])
d2 = OrderedDict([("b", 2), ("a", 1)])
print(d1 == d2)  # False (ลำดับต่างกัน)

# แต่ dict ปกติ
print({"a": 1, "b": 2} == {"b": 2, "a": 1})  # True
```

---

## 8. collections.Counter

### 8.1 Counter พื้นฐาน

```python
from collections import Counter

# สร้าง Counter
c1 = Counter("aabbbccccd")
c2 = Counter(["apple", "banana", "apple", "cherry", "banana", "apple"])
c3 = Counter({"red": 4, "blue": 2})
c4 = Counter(a=3, b=2, c=1)

print(c1)  # Counter({'c': 4, 'b': 3, 'a': 2, 'd': 1})
print(c2)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})

# เข้าถึง count
print(c1["b"])   # 3
print(c1["z"])   # 0 (ไม่ raise KeyError!)

# most_common()
print(c1.most_common(3))    # [('c', 4), ('b', 3), ('a', 2)]
print(c2.most_common())     # ทั้งหมด เรียงจากมากไปน้อย
```

### 8.2 Counter Operations

```python
from collections import Counter

inventory1 = Counter(apple=10, banana=5, cherry=3)
inventory2 = Counter(apple=3, banana=8, mango=4)

# บวก
total = inventory1 + inventory2
print(total)  # Counter({'banana': 13, 'apple': 13, 'mango': 4, 'cherry': 3})

# ลบ (ไม่เก็บ <= 0)
diff = inventory1 - inventory2
print(diff)   # Counter({'apple': 7, 'cherry': 3})

# intersection (min)
common = inventory1 & inventory2
print(common) # Counter({'apple': 3, 'banana': 5})

# union (max)
union = inventory1 | inventory2
print(union)  # Counter({'banana': 8, 'apple': 10, 'mango': 4, 'cherry': 3})

# update() - เพิ่ม count
inventory1.update(apple=5)
print(inventory1["apple"])  # 15

# subtract() - ลด count (อาจเป็น negative)
inventory1.subtract(apple=20)
print(inventory1["apple"])  # -5
```

### 8.3 ตัวอย่างจริง: Text Analysis

```python
from collections import Counter
import re

def analyze_text(text):
    """วิเคราะห์ text"""
    # แยกคำ
    words = re.findall(r'\b[a-zA-Z]+\b', text.lower())
    
    # นับคำ
    word_count = Counter(words)
    
    # นับตัวอักษร
    letters = Counter(c for c in text.lower() if c.isalpha())
    
    return {
        "total_words": len(words),
        "unique_words": len(word_count),
        "most_common_words": word_count.most_common(5),
        "most_common_letters": letters.most_common(5),
        "word_frequencies": dict(word_count)
    }

text = """
To be or not to be that is the question
Whether tis nobler in the mind to suffer
The slings and arrows of outrageous fortune
"""

result = analyze_text(text)
print(f"Total words: {result['total_words']}")
print(f"Unique words: {result['unique_words']}")
print(f"Most common words: {result['most_common_words']}")
print(f"Most common letters: {result['most_common_letters']}")
```

---

## 9. ChainMap

```python
from collections import ChainMap

# ChainMap รวม dict หลายตัวเป็นหนึ่ง (ไม่ copy ข้อมูล)
defaults = {"color": "blue", "font": "Arial", "size": 12}
user_prefs = {"color": "red", "size": 16}
local_prefs = {"font": "Comic Sans"}

# ค้นหาตามลำดับ: local → user → defaults
config = ChainMap(local_prefs, user_prefs, defaults)

print(config["color"])  # red (จาก user_prefs)
print(config["font"])   # Comic Sans (จาก local_prefs)
print(config["size"])   # 16 (จาก user_prefs)

# แก้ไขใน ChainMap → แก้ map แรก (local_prefs)
config["color"] = "green"
print(local_prefs)  # {'font': 'Comic Sans', 'color': 'green'}
print(user_prefs)   # ไม่เปลี่ยน

# new_child() - สร้าง child scope ใหม่
child_config = config.new_child({"size": 20})
print(child_config["size"])   # 20
print(config["size"])         # 16 (parent ไม่เปลี่ยน)
```

---

## 10. Merging Dictionaries

### 10.1 วิธีต่างๆ ในการ merge

```python
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
dict3 = {"b": 20, "e": 5}  # มี key ซ้ำกับ dict1

# วิธีที่ 1: update() (Python 3.x)
merged = dict1.copy()
merged.update(dict2)
print(merged)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# วิธีที่ 2: ** unpacking (Python 3.5+)
merged2 = {**dict1, **dict2, **dict3}
print(merged2)  # {'a': 1, 'b': 20, 'c': 3, 'd': 4, 'e': 5}

# วิธีที่ 3: | operator (Python 3.9+)
merged3 = dict1 | dict2
print(merged3)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# วิธีที่ 4: |= (in-place, Python 3.9+)
dict4 = {"x": 10, "y": 20}
dict4 |= {"y": 99, "z": 30}
print(dict4)  # {'x': 10, 'y': 99, 'z': 30}
```

---

## 11. โปรแกรมจริง (Real-world Examples)

### 11.1 Phonebook

```python
class Phonebook:
    """สมุดโทรศัพท์"""
    
    def __init__(self):
        self.contacts = {}
    
    def add_contact(self, name, phone, email=None, address=None):
        """เพิ่มผู้ติดต่อ"""
        self.contacts[name] = {
            "phone": phone,
            "email": email,
            "address": address
        }
        print(f"เพิ่ม {name} สำเร็จ")
    
    def update_contact(self, name, **kwargs):
        """อัปเดตข้อมูล"""
        if name not in self.contacts:
            print(f"ไม่พบ {name}")
            return
        self.contacts[name].update(kwargs)
        print(f"อัปเดต {name} สำเร็จ")
    
    def delete_contact(self, name):
        """ลบผู้ติดต่อ"""
        if name in self.contacts:
            del self.contacts[name]
            print(f"ลบ {name} สำเร็จ")
        else:
            print(f"ไม่พบ {name}")
    
    def search(self, query):
        """ค้นหาด้วยชื่อหรือเบอร์โทร"""
        results = {}
        query = query.lower()
        for name, info in self.contacts.items():
            if (query in name.lower() or
                query in (info.get("phone") or "") or
                query in (info.get("email") or "")):
                results[name] = info
        return results
    
    def display(self):
        """แสดงทั้งหมด"""
        if not self.contacts:
            print("สมุดโทรศัพท์ว่าง")
            return
        print(f"\n{'ชื่อ':20} {'โทรศัพท์':15} {'อีเมล'}")
        print("-" * 60)
        for name in sorted(self.contacts.keys()):
            info = self.contacts[name]
            print(f"{name:20} {info['phone']:15} {info.get('email', 'N/A')}")
    
    def export(self):
        """Export เป็น list ของ tuples"""
        return [(name, info) for name, info in sorted(self.contacts.items())]

# ทดสอบ
pb = Phonebook()
pb.add_contact("Alice Smith", "080-123-4567", "alice@email.com", "Bangkok")
pb.add_contact("Bob Jones", "090-987-6543", "bob@email.com")
pb.add_contact("Charlie Brown", "062-555-1234", "charlie@email.com", "Chiang Mai")
pb.add_contact("Diana Prince", "091-777-8888")

pb.display()

print("\nค้นหา 'alice':")
results = pb.search("alice")
for name, info in results.items():
    print(f"  {name}: {info}")

pb.update_contact("Bob Jones", email="bob.new@email.com", phone="090-000-1111")
pb.delete_contact("Diana Prince")
pb.display()
```

### 11.2 Word Frequency Counter (Complete)

```python
import re
from collections import Counter, defaultdict

class TextAnalyzer:
    """วิเคราะห์ข้อความ"""
    
    def __init__(self, text):
        self.text = text
        self.words = self._tokenize()
    
    def _tokenize(self):
        """แยกคำ"""
        return re.findall(r'\b[a-zA-Z]+\b', self.text.lower())
    
    def word_frequency(self):
        """ความถี่ของแต่ละคำ"""
        return Counter(self.words)
    
    def top_words(self, n=10):
        """คำที่ใช้บ่อยที่สุด"""
        return self.word_frequency().most_common(n)
    
    def sentence_count(self):
        """จำนวนประโยค"""
        return len(re.split(r'[.!?]+', self.text))
    
    def avg_word_length(self):
        """ความยาวเฉลี่ยของคำ"""
        if not self.words:
            return 0
        return sum(len(w) for w in self.words) / len(self.words)
    
    def word_positions(self):
        """ตำแหน่งของแต่ละคำ"""
        positions = defaultdict(list)
        for i, word in enumerate(self.words):
            positions[word].append(i)
        return dict(positions)
    
    def vocabulary_richness(self):
        """ความหลากหลายของคำศัพท์ (type-token ratio)"""
        if not self.words:
            return 0
        return len(set(self.words)) / len(self.words)
    
    def summary(self):
        """สรุปข้อมูล"""
        wf = self.word_frequency()
        return {
            "total_words": len(self.words),
            "unique_words": len(set(self.words)),
            "sentences": self.sentence_count(),
            "avg_word_length": round(self.avg_word_length(), 2),
            "vocabulary_richness": round(self.vocabulary_richness(), 3),
            "top_5_words": self.top_words(5)
        }

# ทดสอบ
text = """
Python is an interpreted high-level general-purpose programming language. 
Python's design philosophy emphasizes code readability. 
Python is dynamically typed and garbage-collected. 
It supports multiple programming paradigms.
Python was created by Guido van Rossum and first released in 1991.
"""

analyzer = TextAnalyzer(text)
summary = analyzer.summary()

print("=" * 40)
print("TEXT ANALYSIS SUMMARY")
print("=" * 40)
for key, value in summary.items():
    print(f"{key:25}: {value}")
```

### 11.3 Config Management

```python
import json
from copy import deepcopy

class ConfigManager:
    """จัดการ configuration แบบ hierarchical"""
    
    DEFAULT_CONFIG = {
        "database": {
            "host": "localhost",
            "port": 5432,
            "name": "mydb",
            "user": "postgres",
            "pool_size": 5
        },
        "server": {
            "host": "0.0.0.0",
            "port": 8080,
            "debug": False,
            "workers": 4
        },
        "cache": {
            "backend": "memory",
            "ttl": 300,
            "max_size": 1000
        },
        "logging": {
            "level": "INFO",
            "file": "app.log",
            "format": "%(asctime)s - %(levelname)s - %(message)s"
        }
    }
    
    def __init__(self):
        self.config = deepcopy(self.DEFAULT_CONFIG)
    
    def load(self, overrides):
        """โหลดค่า override"""
        self._deep_update(self.config, overrides)
    
    def _deep_update(self, base, updates):
        """อัปเดต nested dict"""
        for key, value in updates.items():
            if key in base and isinstance(base[key], dict) and isinstance(value, dict):
                self._deep_update(base[key], value)
            else:
                base[key] = value
    
    def get(self, path, default=None):
        """ดึงค่าด้วย dot notation (e.g., 'database.host')"""
        keys = path.split(".")
        current = self.config
        for key in keys:
            if isinstance(current, dict):
                current = current.get(key)
            else:
                return default
        return current if current is not None else default
    
    def set(self, path, value):
        """กำหนดค่าด้วย dot notation"""
        keys = path.split(".")
        current = self.config
        for key in keys[:-1]:
            current = current.setdefault(key, {})
        current[keys[-1]] = value
    
    def display(self):
        """แสดง config"""
        def print_dict(d, indent=0):
            for k, v in d.items():
                if isinstance(v, dict):
                    print(f"{'  ' * indent}{k}:")
                    print_dict(v, indent + 1)
                else:
                    print(f"{'  ' * indent}{k}: {v}")
        print_dict(self.config)

# ทดสอบ
cfg = ConfigManager()

# โหลด environment-specific config
prod_config = {
    "database": {"host": "db.prod.com", "pool_size": 20},
    "server": {"debug": False, "workers": 8},
    "logging": {"level": "WARNING"}
}

cfg.load(prod_config)

print("Configuration:")
cfg.display()
print(f"\nDB Host: {cfg.get('database.host')}")
print(f"Server Debug: {cfg.get('server.debug')}")

cfg.set("cache.backend", "redis")
cfg.set("cache.host", "redis.prod.com")
print(f"\nCache: {cfg.get('cache')}")
```

---

## แบบฝึกหัด (Exercises)

### ข้อ 1: Grade Book
```python
def create_gradebook():
    gradebook = {}
    
    def add_grade(student, subject, score):
        if student not in gradebook:
            gradebook[student] = {}
        gradebook[student][subject] = score
    
    def get_average(student):
        scores = gradebook.get(student, {}).values()
        return sum(scores) / len(scores) if scores else 0
    
    def get_top_student():
        return max(gradebook, key=lambda s: get_average(s))
    
    add_grade("Alice", "Math", 90)
    add_grade("Alice", "Science", 85)
    add_grade("Bob", "Math", 75)
    add_grade("Bob", "Science", 80)
    
    for student in gradebook:
        print(f"{student}: {get_average(student):.1f}")
    print(f"Top student: {get_top_student()}")

create_gradebook()
```

### ข้อ 2: Inventory Management
```python
from collections import defaultdict

inventory = defaultdict(lambda: {"count": 0, "price": 0})

def restock(item, count, price):
    inventory[item]["count"] += count
    inventory[item]["price"] = price

def sell(item, count):
    if inventory[item]["count"] >= count:
        inventory[item]["count"] -= count
        return True
    return False

def total_value():
    return sum(v["count"] * v["price"] for v in inventory.values())

restock("Apple", 100, 10)
restock("Banana", 50, 5)
sell("Apple", 30)

for item, data in inventory.items():
    print(f"{item}: {data['count']} units @ ฿{data['price']}")
print(f"Total value: ฿{total_value():,}")
```

### ข้อ 3: Caesar Cipher
```python
def caesar_cipher(text, shift):
    cipher_map = {}
    for i in range(26):
        cipher_map[chr(65 + i)] = chr(65 + (i + shift) % 26)
        cipher_map[chr(97 + i)] = chr(97 + (i + shift) % 26)
    return "".join(cipher_map.get(c, c) for c in text)

print(caesar_cipher("Hello World", 3))   # Khoor Zruog
print(caesar_cipher("Khoor Zruog", -3))  # Hello World
```

### ข้อ 4: Graph Adjacency List
```python
from collections import defaultdict, deque

graph = defaultdict(list)

def add_edge(u, v, directed=False):
    graph[u].append(v)
    if not directed:
        graph[v].append(u)

def bfs(start):
    visited = set()
    queue = deque([start])
    visited.add(start)
    result = []
    while queue:
        node = queue.popleft()
        result.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return result

add_edge("A", "B")
add_edge("A", "C")
add_edge("B", "D")
add_edge("C", "D")
add_edge("D", "E")

print("BFS from A:", bfs("A"))
```

### ข้อ 5: Memoization
```python
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

for i in range(15):
    print(f"fib({i}) = {fibonacci(i)}")
```

### ข้อ 6: Most Frequent Element
```python
from collections import Counter

def most_frequent(lst):
    if not lst:
        return None
    c = Counter(lst)
    return c.most_common(1)[0][0]

def top_n(lst, n):
    return [item for item, _ in Counter(lst).most_common(n)]

data = [1, 3, 2, 1, 4, 1, 3, 5, 3, 2, 1]
print(f"Most frequent: {most_frequent(data)}")  # 1
print(f"Top 3: {top_n(data, 3)}")               # [1, 3, 2]
```

### ข้อ 7: Nested Dict Merge
```python
def deep_merge(dict1, dict2):
    result = dict1.copy()
    for key, value in dict2.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

d1 = {"a": {"x": 1, "y": 2}, "b": 3}
d2 = {"a": {"y": 20, "z": 30}, "c": 4}
merged = deep_merge(d1, d2)
print(merged)  # {'a': {'x': 1, 'y': 20, 'z': 30}, 'b': 3, 'c': 4}
```

### ข้อ 8: Two Sum
```python
def two_sum(nums, target):
    """หา index 2 ตัวที่รวมกันได้ target"""
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return (seen[complement], i)
        seen[num] = i
    return None

print(two_sum([2, 7, 11, 15], 9))   # (0, 1)
print(two_sum([3, 2, 4], 6))        # (1, 2)
```

### ข้อ 9: Dict Frequency Sort
```python
def sort_by_frequency(lst):
    from collections import Counter
    c = Counter(lst)
    return sorted(lst, key=lambda x: (-c[x], x))

print(sort_by_frequency([4, 5, 6, 5, 4, 3, 4]))
# [4, 4, 4, 5, 5, 3, 6]  เรียงตามความถี่ (มาก→น้อย), ถ้าเท่ากันเรียงค่า
```

### ข้อ 10: Student Database
```python
class StudentDB:
    def __init__(self):
        self.students = {}
        self.next_id = 1
    
    def add(self, name, subjects):
        sid = self.next_id
        self.students[sid] = {
            "name": name,
            "subjects": subjects,
            "gpa": sum(subjects.values()) / len(subjects) if subjects else 0
        }
        self.next_id += 1
        return sid
    
    def search_by_name(self, name):
        return {sid: s for sid, s in self.students.items()
                if name.lower() in s["name"].lower()}
    
    def top_students(self, n=3):
        return sorted(self.students.items(),
                      key=lambda x: x[1]["gpa"], reverse=True)[:n]
    
    def subject_average(self, subject):
        scores = [s["subjects"].get(subject) for s in self.students.values()
                  if subject in s["subjects"]]
        return sum(scores) / len(scores) if scores else 0

db = StudentDB()
db.add("Alice", {"Math": 90, "Science": 85, "English": 88})
db.add("Bob", {"Math": 70, "Science": 75, "English": 80})
db.add("Charlie", {"Math": 95, "Science": 92, "English": 91})

print("Top students:")
for sid, s in db.top_students():
    print(f"  {s['name']}: GPA {s['gpa']:.1f}")

print(f"Math average: {db.subject_average('Math'):.1f}")
```

---

## สรุป (Summary)

### Dictionary Methods

| Method | การใช้งาน | Return |
|--------|-----------|--------|
| `get(key, default)` | อ่านอย่างปลอดภัย | value หรือ default |
| `keys()` | ดู keys ทั้งหมด | dict_keys view |
| `values()` | ดู values ทั้งหมด | dict_values view |
| `items()` | ดู key-value pairs | dict_items view |
| `update(other)` | อัปเดตจาก dict อื่น | None |
| `pop(key)` | ลบและคืนค่า | value |
| `popitem()` | ลบ item สุดท้าย | (key, value) |
| `clear()` | ล้างทั้งหมด | None |
| `copy()` | shallow copy | dict |
| `setdefault(key, default)` | ตั้งค่า default ถ้าไม่มี key | value |

### เมื่อไหรควรใช้อะไร
| สถานการณ์ | ใช้ |
|-----------|-----|
| นับจำนวน | `Counter` |
| จัดกลุ่ม list | `defaultdict(list)` |
| รวม settings | `ChainMap` |
| ต้องการ ordered history | `OrderedDict` |
| ทั่วไป | `dict` |

> **หมายเหตุ:** Part ต่อไปจะเรียน Sets ซึ่งเป็นโครงสร้างข้อมูลที่เก็บข้อมูลไม่ซ้ำกัน
