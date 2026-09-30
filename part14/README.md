# Part 14: Sets & Frozensets

## บทนำ (Introduction)

**Set** เป็นโครงสร้างข้อมูลแบบ **unordered collection ที่ไม่มีข้อมูลซ้ำ** ใน Python
เหมาะสำหรับงานที่ต้องการความเป็นเอกลักษณ์ของข้อมูล และ mathematical set operations

### คุณสมบัติหลักของ Set
| คุณสมบัติ | ความหมาย |
|-----------|-----------|
| Unordered | ไม่รับประกันลำดับ |
| Mutable | เพิ่ม/ลบ element ได้ |
| No Duplicates | ไม่เก็บข้อมูลซ้ำ |
| Elements ต้อง Hashable | เก็บได้เฉพาะ immutable types |
| Fast Lookup | O(1) average |

---

## 1. Sets พื้นฐาน

### 1.1 การสร้าง Set

```python
# วิธีที่ 1: Curly braces {}
fruits = {"apple", "banana", "cherry"}
numbers = {1, 2, 3, 4, 5}
mixed = {1, "hello", 3.14, True}

print(fruits)   # {'cherry', 'banana', 'apple'} (ลำดับอาจต่าง)
print(numbers)  # {1, 2, 3, 4, 5}

# ระวัง! {} เปล่าคือ dict ไม่ใช่ set
empty_dict = {}
empty_set = set()
print(type(empty_dict))  # <class 'dict'>
print(type(empty_set))   # <class 'set'>

# วิธีที่ 2: set() constructor
from_list = set([1, 2, 3, 2, 1])    # ลบ duplicates อัตโนมัติ
from_string = set("hello")           # แต่ละตัวอักษร
from_tuple = set((1, 2, 3, 2, 1))

print(from_list)    # {1, 2, 3}
print(from_string)  # {'h', 'e', 'l', 'o'} (l ซ้ำถูกลบ)
print(from_tuple)   # {1, 2, 3}
```

### 1.2 Set ลบ Duplicates อัตโนมัติ

```python
# ข้อมูลซ้ำถูกลบทันที
data = [1, 2, 3, 2, 1, 4, 3, 5, 4]
unique = set(data)
print(unique)  # {1, 2, 3, 4, 5}

# String characters
chars = set("mississippi")
print(chars)  # {'m', 'i', 's', 'p'}

# สร้าง set จาก list of strings
words = ["apple", "banana", "apple", "cherry", "banana"]
unique_words = set(words)
print(unique_words)  # {'apple', 'banana', 'cherry'}
```

### 1.3 Checking Membership

```python
fruits = {"apple", "banana", "cherry"}

# in operator (O(1) - เร็วกว่า list มาก!)
print("apple" in fruits)   # True
print("mango" in fruits)   # False
print("mango" not in fruits)  # True

# เปรียบเทียบ performance: set vs list
import time

# ข้อมูลขนาดใหญ่
big_list = list(range(1000000))
big_set = set(range(1000000))

# ค้นหาใน list
start = time.time()
999999 in big_list
list_time = time.time() - start

# ค้นหาใน set
start = time.time()
999999 in big_set
set_time = time.time() - start

print(f"List lookup: {list_time:.6f}s")
print(f"Set lookup:  {set_time:.6f}s")
# Set เร็วกว่ามาก!
```

---

## 2. Set Operations

### 2.1 Union (การรวม) - |

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# Union: ทุก element จาก A หรือ B
union1 = A | B
union2 = A.union(B)
print(union1)  # {1, 2, 3, 4, 5, 6, 7, 8}
print(union2)  # {1, 2, 3, 4, 5, 6, 7, 8}

# Union หลายชุด
C = {8, 9, 10}
multi_union = A | B | C
print(multi_union)  # {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

# .union() รับ iterable (ไม่ต้องเป็น set)
result = A.union([6, 7, 8], (9, 10))
print(result)
```

### 2.2 Intersection (การตัดกัน) - &

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# Intersection: เฉพาะ element ที่มีทั้งใน A และ B
inter1 = A & B
inter2 = A.intersection(B)
print(inter1)  # {4, 5}
print(inter2)  # {4, 5}

# หลายชุด
C = {3, 4, 5, 6}
print(A & B & C)  # {4, 5}

# ตัวอย่าง: หา common friends
alice_friends = {"Bob", "Charlie", "Diana", "Eve"}
bob_friends = {"Alice", "Charlie", "Frank", "Diana"}
common = alice_friends & bob_friends
print(f"เพื่อนร่วม: {common}")  # {'Charlie', 'Diana'}
```

### 2.3 Difference (ผลต่าง) - -

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# A - B: element ใน A แต่ไม่ใน B
diff_AB = A - B
diff_AB2 = A.difference(B)
print(diff_AB)   # {1, 2, 3}
print(diff_AB2)  # {1, 2, 3}

# B - A: element ใน B แต่ไม่ใน A
diff_BA = B - A
print(diff_BA)   # {8, 6, 7}

# ตัวอย่าง: หา students ที่ลงวิชา math แต่ไม่ลง science
math_students = {"Alice", "Bob", "Charlie", "Diana"}
science_students = {"Bob", "Diana", "Eve", "Frank"}

only_math = math_students - science_students
print(f"เรียน math เท่านั้น: {only_math}")
```

### 2.4 Symmetric Difference (ผลต่างสมมาตร) - ^

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# A ^ B: element ที่อยู่ใน A หรือ B แต่ไม่ทั้งคู่
sym_diff1 = A ^ B
sym_diff2 = A.symmetric_difference(B)
print(sym_diff1)  # {1, 2, 3, 6, 7, 8}
print(sym_diff2)  # {1, 2, 3, 6, 7, 8}

# เท่ากับ (A | B) - (A & B) หรือ (A - B) | (B - A)
verify = (A | B) - (A & B)
print(verify == sym_diff1)  # True
```

---

## 3. Set Methods

### 3.1 add() และ update()

```python
fruits = {"apple", "banana"}

# add() - เพิ่ม 1 element
fruits.add("cherry")
fruits.add("apple")  # ซ้ำ - ไม่เพิ่ม
print(fruits)  # {'apple', 'banana', 'cherry'}

# update() - เพิ่มหลาย elements
fruits.update(["date", "elderberry"])
fruits.update({"fig", "grape"}, ("honeydew",))
print(fruits)
```

### 3.2 remove() และ discard()

```python
fruits = {"apple", "banana", "cherry", "date"}

# remove() - ลบ element (ถ้าไม่มีจะ raise KeyError)
fruits.remove("banana")
print(fruits)

try:
    fruits.remove("mango")
except KeyError as e:
    print(f"KeyError: {e}")

# discard() - ลบ element (ไม่ raise error ถ้าไม่มี)
fruits.discard("cherry")
fruits.discard("mango")  # ไม่ error!
print(fruits)

# pop() - ลบ element แบบสุ่ม (เพราะ set ไม่ ordered)
item = fruits.pop()
print(f"Popped: {item}")
print(fruits)

# clear() - ลบทั้งหมด
fruits.clear()
print(fruits)  # set()
```

### 3.3 In-place Operations

```python
A = {1, 2, 3, 4, 5}

# |= (update with union)
A |= {4, 5, 6, 7}
print(A)  # {1, 2, 3, 4, 5, 6, 7}

# &= (update with intersection)
B = {1, 2, 3, 4, 5}
B &= {3, 4, 5, 6, 7}
print(B)  # {3, 4, 5}

# -= (update with difference)
C = {1, 2, 3, 4, 5}
C -= {3, 4}
print(C)  # {1, 2, 5}

# ^= (update with symmetric difference)
D = {1, 2, 3, 4, 5}
D ^= {4, 5, 6, 7}
print(D)  # {1, 2, 3, 6, 7}

# intersection_update()
E = {1, 2, 3, 4, 5}
E.intersection_update([3, 4, 5, 6, 7])
print(E)  # {3, 4, 5}

# difference_update()
F = {1, 2, 3, 4, 5}
F.difference_update([3, 4])
print(F)  # {1, 2, 5}

# symmetric_difference_update()
G = {1, 2, 3, 4, 5}
G.symmetric_difference_update([4, 5, 6, 7])
print(G)  # {1, 2, 3, 6, 7}
```

---

## 4. Subset และ Superset

### 4.1 issubset() และ issuperset()

```python
A = {1, 2, 3}
B = {1, 2, 3, 4, 5}
C = {1, 2, 3}

# issubset() - A ⊆ B (ทุก element ใน A มีอยู่ใน B)
print(A.issubset(B))    # True (A ⊆ B)
print(B.issubset(A))    # False
print(A.issubset(C))    # True (A ⊆ C เพราะเท่ากัน)

# ใช้ operator <= และ <
print(A <= B)  # True (subset)
print(A < B)   # True (proper subset - A ≠ B)
print(A < C)   # False (A == C ดังนั้นไม่ใช่ proper subset)
print(A <= C)  # True (subset, รวมเท่ากัน)

# issuperset() - B ⊇ A (ทุก element ใน A มีอยู่ใน B)
print(B.issuperset(A))  # True
print(A.issuperset(B))  # False

# ใช้ operator >= และ >
print(B >= A)  # True (superset)
print(B > A)   # True (proper superset)
print(C >= A)  # True (superset, เท่ากัน)
print(C > A)   # False (not proper superset)

# isdisjoint() - ไม่มี element ร่วมกัน
D = {6, 7, 8}
print(A.isdisjoint(D))  # True (ไม่มีส่วนร่วม)
print(A.isdisjoint(B))  # False (มีส่วนร่วม)
```

---

## 5. Set Comprehension

### 5.1 Basic Set Comprehension

```python
# {expression for item in iterable}
squares = {x**2 for x in range(1, 11)}
print(squares)  # {1, 4, 9, 16, 25, 36, 49, 64, 81, 100}

# สังเกต: เหมือน list comprehension แต่ใช้ {}
evens = {x for x in range(1, 21) if x % 2 == 0}
print(evens)  # {2, 4, 6, 8, 10, 12, 14, 16, 18, 20}

# แปลง string เป็น set ของคำ
sentence = "the quick brown fox jumps over the lazy dog"
unique_words = {word for word in sentence.split()}
print(f"Unique words: {len(unique_words)}")

# ดึงตัวอักษรพิเศษ
chars = {c for c in "Hello, World!" if not c.isalnum() and c != " "}
print(chars)  # {',', '!'}
```

### 5.2 Advanced Set Comprehension

```python
# หา factor ทั้งหมด
def get_factors(n):
    return {i for i in range(1, n+1) if n % i == 0}

print(get_factors(24))  # {1, 2, 3, 4, 6, 8, 12, 24}
print(get_factors(13))  # {1, 13}  (prime!)

# หา vowels
sentence = "Python programming is fun and easy"
vowels = {c.lower() for c in sentence if c.lower() in "aeiou"}
print(vowels)  # {'a', 'e', 'i', 'o', 'u'}

# Common elements จาก หลาย list
lists = [[1, 2, 3, 4], [2, 3, 5, 6], [3, 4, 5, 7], [1, 3, 6, 7]]
common = set.intersection(*[set(lst) for lst in lists])
print(common)  # {3}
```

---

## 6. Frozensets

### 6.1 Frozenset พื้นฐาน

```python
# frozenset เหมือน set แต่ immutable (ไม่สามารถแก้ไขได้)
fs = frozenset([1, 2, 3, 4, 5])
print(fs)       # frozenset({1, 2, 3, 4, 5})
print(type(fs)) # <class 'frozenset'>

# สร้างจาก iterable ต่างๆ
fs1 = frozenset("Python")
fs2 = frozenset({1, 2, 3})
fs3 = frozenset(range(5))
print(fs1)  # frozenset({'P', 'y', 't', 'h', 'o', 'n'})

# frozenset ไม่มี method ที่แก้ไขได้
try:
    fs.add(6)
except AttributeError as e:
    print(f"Error: {e}")
# AttributeError: 'frozenset' object has no attribute 'add'
```

### 6.2 Frozenset เป็น Dict Key

```python
# frozenset เป็น hashable → ใช้เป็น dict key ได้!
# set ทำแบบนี้ไม่ได้

# ใช้ frozenset เป็น key ใน dict
graph = {}
edge1 = frozenset({"A", "B"})
edge2 = frozenset({"B", "C"})
edge3 = frozenset({"A", "C"})

graph[edge1] = 5  # ระยะห่าง A-B
graph[edge2] = 3  # ระยะห่าง B-C
graph[edge3] = 7  # ระยะห่าง A-C

print(graph)
print(graph[frozenset({"A", "B"})])  # 5
print(graph[frozenset({"B", "A"})])  # 5 (ลำดับไม่สำคัญ!)

# frozenset ใน set
valid_combos = {
    frozenset({"rock", "scissors"}),  # rock beats scissors
    frozenset({"scissors", "paper"}), # scissors beats paper
    frozenset({"paper", "rock"}),     # paper beats rock
}

print(frozenset({"rock", "scissors"}) in valid_combos)  # True
```

### 6.3 frozenset Operations

```python
# frozenset รองรับ set operations ทั้งหมด แต่คืน frozenset
fs1 = frozenset({1, 2, 3, 4})
fs2 = frozenset({3, 4, 5, 6})

print(fs1 | fs2)   # frozenset({1, 2, 3, 4, 5, 6})
print(fs1 & fs2)   # frozenset({3, 4})
print(fs1 - fs2)   # frozenset({1, 2})
print(fs1 ^ fs2)   # frozenset({1, 2, 5, 6})

# เปรียบเทียบกับ set
s = {1, 2, 3, 4}
print(fs1 == s)      # True (เนื้อหาเหมือนกัน)
print(fs1 <= s)      # True
```

---

## 7. Set vs List vs Tuple

### 7.1 เปรียบเทียบ

```python
import sys
import timeit

# สร้างข้อมูล
data = list(range(10000))
my_list = list(data)
my_set = set(data)
my_tuple = tuple(data)

# Memory
print(f"List size:  {sys.getsizeof(my_list):,} bytes")
print(f"Set size:   {sys.getsizeof(my_set):,} bytes")
print(f"Tuple size: {sys.getsizeof(my_tuple):,} bytes")

# Lookup speed
lookup_val = 9999

list_time = timeit.timeit(lambda: 9999 in my_list, number=10000)
set_time = timeit.timeit(lambda: 9999 in my_set, number=10000)
tuple_time = timeit.timeit(lambda: 9999 in my_tuple, number=10000)

print(f"\nLookup 9999:")
print(f"List:  {list_time:.4f}s")
print(f"Set:   {set_time:.4f}s")   # เร็วที่สุด!
print(f"Tuple: {tuple_time:.4f}s")
```

### 7.2 เมื่อไหรควรใช้อะไร

```python
# ใช้ List เมื่อ:
shopping_cart = ["apple", "banana", "apple"]  # ต้องการ duplicates + ordered
playlist = ["song1", "song2", "song3"]        # ต้องการ index/slice
stack = []                                     # ต้องการ append/pop

# ใช้ Tuple เมื่อ:
coordinates = (10.5, 20.3)       # ข้อมูลคงที่
rgb = (255, 128, 0)               # ใช้เป็น dict key
db_record = ("Alice", 25, "BKK") # return multiple values

# ใช้ Set เมื่อ:
unique_visitors = set()                    # เก็บเฉพาะ unique
tags = {"python", "programming", "code"}  # ค้นหาเร็ว, ไม่ซ้ำ
math_set | science_set                    # set operations

# ตัวอย่างการเลือกใช้
def process_data(items):
    # List: เก็บลำดับ, อนุญาต duplicates
    ordered_results = []
    
    # Set: เก็บ unique ตรวจสอบ
    seen = set()
    
    for item in items:
        if item not in seen:  # O(1) lookup!
            seen.add(item)
            ordered_results.append(item)
    
    return ordered_results  # unique แต่รักษาลำดับ

data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
print(process_data(data))  # [3, 1, 4, 5, 9, 2, 6]
```

---

## 8. โปรแกรมจริง (Real-world Examples)

### 8.1 Unique Elements และ Duplicate Finder

```python
def find_duplicates(lst):
    """หา duplicates ใน list"""
    seen = set()
    duplicates = set()
    for item in lst:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)
    return duplicates

def remove_duplicates_ordered(lst):
    """ลบ duplicates โดยรักษาลำดับ"""
    seen = set()
    return [x for x in lst if x not in seen and not seen.add(x)]

data = [1, 2, 3, 2, 4, 3, 5, 1, 6]
print(f"Duplicates: {find_duplicates(data)}")
print(f"Unique (ordered): {remove_duplicates_ordered(data)}")

# ตัวอย่างจริง: ตรวจสอบ email ซ้ำ
emails = [
    "alice@example.com",
    "bob@example.com",
    "alice@example.com",  # ซ้ำ
    "charlie@example.com",
    "bob@example.com"     # ซ้ำ
]

unique_emails = set(emails)
duplicate_emails = {e for e in emails if emails.count(e) > 1}

print(f"\nTotal emails: {len(emails)}")
print(f"Unique emails: {len(unique_emails)}")
print(f"Duplicate emails: {duplicate_emails}")
```

### 8.2 Common Interests / Social Network

```python
class SocialNetwork:
    """Social Network ด้วย Set"""
    
    def __init__(self):
        self.users = {}       # user → set of friends
        self.interests = {}   # user → set of interests
    
    def add_user(self, name, interests=None):
        self.users[name] = set()
        self.interests[name] = set(interests or [])
    
    def add_friendship(self, user1, user2):
        self.users[user1].add(user2)
        self.users[user2].add(user1)
    
    def common_friends(self, user1, user2):
        return self.users[user1] & self.users[user2]
    
    def friend_suggestions(self, user):
        """แนะนำเพื่อน: เพื่อนของเพื่อนที่ยังไม่เป็นเพื่อนกัน"""
        friends_of_friends = set()
        for friend in self.users[user]:
            friends_of_friends |= self.users[friend]
        # ลบตัวเอง และเพื่อนที่มีอยู่แล้ว
        return friends_of_friends - self.users[user] - {user}
    
    def common_interests(self, user1, user2):
        return self.interests[user1] & self.interests[user2]
    
    def users_with_interest(self, interest):
        return {u for u, interests in self.interests.items()
                if interest in interests}
    
    def recommend_by_interests(self, user, min_common=2):
        """แนะนำตามความสนใจร่วม"""
        recommendations = {}
        user_interests = self.interests[user]
        for other_user in self.users:
            if other_user != user and other_user not in self.users[user]:
                common = user_interests & self.interests[other_user]
                if len(common) >= min_common:
                    recommendations[other_user] = common
        return recommendations

# ทดสอบ
sn = SocialNetwork()

sn.add_user("Alice", ["python", "music", "hiking", "photography"])
sn.add_user("Bob", ["java", "music", "gaming", "hiking"])
sn.add_user("Charlie", ["python", "gaming", "cooking", "music"])
sn.add_user("Diana", ["python", "photography", "cooking", "hiking"])
sn.add_user("Eve", ["music", "cooking", "hiking", "yoga"])

sn.add_friendship("Alice", "Bob")
sn.add_friendship("Bob", "Charlie")
sn.add_friendship("Charlie", "Diana")
sn.add_friendship("Diana", "Eve")

print("Common friends (Alice & Charlie):", sn.common_friends("Alice", "Charlie"))
print("Friend suggestions for Alice:", sn.friend_suggestions("Alice"))
print("Common interests (Alice & Bob):", sn.common_interests("Alice", "Bob"))
print("Python users:", sn.users_with_interest("python"))
print("\nAlice's recommendations by interests:")
for person, interests in sn.recommend_by_interests("Alice").items():
    print(f"  {person}: {interests}")
```

### 8.3 Removing Duplicates จาก Data Pipeline

```python
def data_pipeline_demo():
    """ตัวอย่าง data pipeline ที่ใช้ set"""
    
    # ข้อมูลดิบที่อาจมี duplicates
    raw_data = [
        {"id": 1, "name": "Alice", "email": "alice@ex.com"},
        {"id": 2, "name": "Bob", "email": "bob@ex.com"},
        {"id": 1, "name": "Alice", "email": "alice@ex.com"},  # duplicate
        {"id": 3, "name": "Charlie", "email": "charlie@ex.com"},
        {"id": 2, "name": "Bob", "email": "bob_new@ex.com"},  # same id, diff email
    ]
    
    # Remove exact duplicates (ใช้ frozenset ของ items)
    seen_records = set()
    unique_records = []
    for record in raw_data:
        record_key = frozenset(record.items())
        if record_key not in seen_records:
            seen_records.add(record_key)
            unique_records.append(record)
    
    print("หลังลบ exact duplicates:")
    for r in unique_records:
        print(f"  {r}")
    
    # ตรวจสอบ IDs ซ้ำ
    ids = [r["id"] for r in unique_records]
    duplicate_ids = {id_ for id_ in ids if ids.count(id_) > 1}
    
    if duplicate_ids:
        print(f"\nพบ ID ซ้ำ: {duplicate_ids}")
    
    # Validate emails
    valid_domains = {"ex.com", "gmail.com", "yahoo.com"}
    for record in unique_records:
        domain = record["email"].split("@")[-1]
        if domain not in valid_domains:
            print(f"Warning: {record['name']} มี email domain ที่ไม่รู้จัก: {domain}")
    
    # หา unique fields
    all_emails = {r["email"] for r in unique_records}
    all_ids = {r["id"] for r in unique_records}
    print(f"\nUnique emails: {len(all_emails)}")
    print(f"Unique IDs: {len(all_ids)}")

data_pipeline_demo()
```

---

## 9. Advanced Set Operations

### 9.1 Power Set

```python
def power_set(s):
    """สร้าง power set (ทุก subset ที่เป็นไปได้)"""
    result = {frozenset()}
    for elem in s:
        result = result | {subset | {elem} for subset in result}
    return result

ps = power_set({1, 2, 3})
print(f"Power set of {{1,2,3}}:")
for subset in sorted(ps, key=len):
    print(f"  {set(subset)}")
print(f"Total subsets: {len(ps)} = 2^3")
```

### 9.2 Set Coverage Problem

```python
def greedy_cover(universe, subsets):
    """Greedy Set Cover Algorithm"""
    covered = set()
    selected = []
    
    while covered != universe:
        # เลือก subset ที่ cover ได้มากที่สุด
        best = max(subsets, key=lambda s: len(s - covered))
        selected.append(best)
        covered |= best
    
    return selected

universe = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
subsets = [
    frozenset({1, 2, 3, 4, 5}),
    frozenset({4, 5, 6, 7}),
    frozenset({6, 7, 8, 9}),
    frozenset({1, 3, 5, 7, 9}),
    frozenset({8, 9, 10}),
    frozenset({2, 4, 6, 8, 10}),
]

selected = greedy_cover(universe, subsets)
print(f"Selected {len(selected)} subsets to cover all elements:")
for s in selected:
    print(f"  {sorted(s)}")
```

### 9.3 Anagram Detection

```python
def are_anagrams(s1, s2):
    """ตรวจสอบว่า 2 strings เป็น anagram"""
    # วิธีที่ 1: เรียงและเปรียบเทียบ
    return sorted(s1.lower()) == sorted(s2.lower())

def are_anagrams_v2(s1, s2):
    """ด้วย Counter"""
    from collections import Counter
    return Counter(s1.lower()) == Counter(s2.lower())

def group_anagrams(words):
    """จัดกลุ่ม anagrams"""
    groups = {}
    for word in words:
        key = frozenset(word.lower())  # frozenset ของตัวอักษร
        groups.setdefault(key, []).append(word)
    return list(groups.values())

words = ["eat", "tea", "tan", "ate", "nat", "bat", "tab"]
groups = group_anagrams(words)
print("Anagram groups:")
for g in groups:
    print(f"  {g}")
```

---

## แบบฝึกหัด (Exercises)

### ข้อ 1: Unique Characters
```python
def unique_chars(s):
    """ตรวจสอบว่า string มีตัวอักษรซ้ำไหม"""
    return len(set(s)) == len(s)

print(unique_chars("abcdef"))  # True
print(unique_chars("aabcdef")) # False

def first_unique(s):
    """หาตัวอักษรที่ไม่ซ้ำตัวแรก"""
    from collections import Counter
    counts = Counter(s)
    for char in s:
        if counts[char] == 1:
            return char
    return None

print(first_unique("aabbcde"))  # c
```

### ข้อ 2: Set Difference Applications
```python
def permissions_check(user_perms, required_perms):
    """ตรวจสอบว่า user มี permissions ที่ต้องการครบไหม"""
    missing = required_perms - user_perms
    return len(missing) == 0, missing

user_perms = {"read", "write", "execute"}
required = {"read", "write", "admin"}
ok, missing = permissions_check(user_perms, required)
print(f"Access granted: {ok}")
print(f"Missing permissions: {missing}")
```

### ข้อ 3: Intersection ของหลาย Sets
```python
def common_elements(*lists):
    """หา elements ที่มีอยู่ในทุก list"""
    if not lists:
        return set()
    sets = [set(lst) for lst in lists]
    return set.intersection(*sets)

lists = [[1,2,3,4,5], [2,3,4,6,7], [3,4,5,8,9], [1,3,4,7,9]]
print(common_elements(*lists))  # {3, 4}
```

### ข้อ 4: Tag System
```python
class TagSystem:
    def __init__(self):
        self.items = {}  # item → set of tags
    
    def add_item(self, item, tags):
        self.items[item] = set(tags)
    
    def find_by_tags(self, required_tags, mode="all"):
        required = set(required_tags)
        if mode == "all":
            return {item for item, tags in self.items.items()
                    if required.issubset(tags)}
        else:  # any
            return {item for item, tags in self.items.items()
                    if required & tags}
    
    def similar_items(self, item, min_common=2):
        if item not in self.items:
            return {}
        item_tags = self.items[item]
        return {other: len(item_tags & tags)
                for other, tags in self.items.items()
                if other != item and len(item_tags & tags) >= min_common}

ts = TagSystem()
ts.add_item("article1", {"python", "programming", "tutorial"})
ts.add_item("article2", {"python", "data-science", "tutorial"})
ts.add_item("article3", {"java", "programming", "oop"})
ts.add_item("video1", {"python", "programming", "advanced"})

print("Articles with 'python' AND 'tutorial':", ts.find_by_tags(["python", "tutorial"]))
print("Articles with 'python' OR 'java':", ts.find_by_tags(["python", "java"], "any"))
print("Similar to article1:", ts.similar_items("article1"))
```

### ข้อ 5: Dice Game Probability
```python
import itertools

def dice_outcomes(num_dice, sides=6):
    """สร้าง set ของ outcomes ทั้งหมด"""
    return set(itertools.product(range(1, sides+1), repeat=num_dice))

def probability_of_sum(total, num_dice=2, sides=6):
    """ความน่าจะเป็นของผลรวมที่ต้องการ"""
    all_outcomes = dice_outcomes(num_dice, sides)
    favorable = {o for o in all_outcomes if sum(o) == total}
    return len(favorable) / len(all_outcomes)

for s in range(2, 13):
    prob = probability_of_sum(s)
    bar = "█" * int(prob * 100)
    print(f"Sum {s:2d}: {prob:.4f} {bar}")
```

### ข้อ 6: Graph Reachability
```python
def reachable_nodes(graph, start):
    """หา nodes ทั้งหมดที่เข้าถึงได้จาก start"""
    visited = set()
    stack = [start]
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            stack.extend(graph.get(node, set()) - visited)
    return visited

graph = {
    "A": {"B", "C"},
    "B": {"D", "E"},
    "C": {"F"},
    "D": set(),
    "E": {"F"},
    "F": set(),
    "G": {"H"},  # disconnected
    "H": set()
}

print("Reachable from A:", reachable_nodes(graph, "A"))
print("Reachable from G:", reachable_nodes(graph, "G"))
```

### ข้อ 7: Spell Checker
```python
def spell_checker(text, dictionary):
    words = set(text.lower().split())
    correct = words & set(dictionary)
    misspelled = words - set(dictionary)
    return correct, misspelled

dictionary = ["the", "quick", "brown", "fox", "jumps", "over", "lazy", "dog"]
text = "the quik brwon fox jumps over the laazy dog"
correct, wrong = spell_checker(text, dictionary)
print(f"Correct: {correct}")
print(f"Misspelled: {wrong}")
```

### ข้อ 8: Matrix to Set Operations
```python
def matrix_to_set_of_rows(matrix):
    return {frozenset(row) for row in matrix}

def unique_rows(matrix):
    seen = set()
    result = []
    for row in matrix:
        key = tuple(row)
        if key not in seen:
            seen.add(key)
            result.append(row)
    return result

matrix = [[1,2,3], [4,5,6], [1,2,3], [7,8,9], [4,5,6]]
print("Unique rows:", unique_rows(matrix))
```

### ข้อ 9: Version Compatibility Checker
```python
def check_compatibility(required_features, available_features):
    """ตรวจสอบว่า features ที่ต้องการมีครบไหม"""
    required = frozenset(required_features)
    available = frozenset(available_features)
    
    missing = required - available
    extra = available - required
    
    return {
        "compatible": len(missing) == 0,
        "missing": missing,
        "extra": extra,
        "coverage": len(required & available) / len(required) if required else 1.0
    }

required = ["auth", "database", "cache", "logging", "email"]
available = ["auth", "database", "cache", "logging", "storage", "api"]

result = check_compatibility(required, available)
print(f"Compatible: {result['compatible']}")
print(f"Missing: {result['missing']}")
print(f"Extra: {result['extra']}")
print(f"Coverage: {result['coverage']:.0%}")
```

### ข้อ 10: Student Enrollment System
```python
class EnrollmentSystem:
    def __init__(self):
        self.courses = {}  # course → set of students
        self.students = {}  # student → set of courses
    
    def enroll(self, student, course):
        self.courses.setdefault(course, set()).add(student)
        self.students.setdefault(student, set()).add(course)
    
    def drop(self, student, course):
        self.courses.get(course, set()).discard(student)
        self.students.get(student, set()).discard(course)
    
    def common_students(self, *courses):
        """นักเรียนที่ลงทะเบียนทุก course"""
        sets = [self.courses.get(c, set()) for c in courses]
        return set.intersection(*sets) if sets else set()
    
    def recommended_courses(self, student):
        """แนะนำ courses จากเพื่อนร่วม course"""
        my_courses = self.students.get(student, set())
        my_classmates = set()
        for course in my_courses:
            my_classmates |= self.courses[course]
        my_classmates.discard(student)
        
        recommended = set()
        for classmate in my_classmates:
            recommended |= self.students.get(classmate, set())
        return recommended - my_courses

es = EnrollmentSystem()
for student in ["Alice", "Bob", "Charlie", "Diana"]:
    es.enroll(student, "Python")
for student in ["Bob", "Charlie", "Eve"]:
    es.enroll(student, "Data Science")
for student in ["Alice", "Charlie", "Frank"]:
    es.enroll(student, "Web Dev")

print("Students in both Python & Data Science:", es.common_students("Python", "Data Science"))
print("Recommended for Alice:", es.recommended_courses("Alice"))
```

---

## สรุป (Summary)

### Set Operations Summary

| Operation | Operator | Method |
|-----------|----------|--------|
| Union | `\|` | `.union()` |
| Intersection | `&` | `.intersection()` |
| Difference | `-` | `.difference()` |
| Symmetric Diff | `^` | `.symmetric_difference()` |
| Subset | `<=` | `.issubset()` |
| Proper Subset | `<` | |
| Superset | `>=` | `.issuperset()` |
| Proper Superset | `>` | |
| Disjoint | | `.isdisjoint()` |

### Set vs Frozenset

| คุณสมบัติ | Set | Frozenset |
|-----------|-----|-----------|
| Mutable | ✓ | |
| Hashable | | ✓ |
| Dict Key | | ✓ |
| ใน Set | | ✓ |
| Operations | ทุกอย่าง | อ่านอย่างเดียว |

### เมื่อไหรควรใช้ Set
- ต้องการ unique elements
- ค้นหาข้อมูล (lookup) บ่อยมาก
- ต้องการทำ set operations (union, intersection, difference)
- ตรวจสอบ membership (in/not in)
- ลบ duplicates จาก collection

> **หมายเหตุ:** Part ต่อไปจะเรียน File I/O ซึ่งเป็นการอ่านและเขียนไฟล์ใน Python
