# Part 29: Regular Expressions (Regex)

## สารบัญ
1. [Regex Syntax พื้นฐาน](#regex-syntax-พื้นฐาน)
2. [Character Classes](#character-classes)
3. [Quantifiers](#quantifiers)
4. [Anchors](#anchors)
5. [Groups และ Capturing](#groups-และ-capturing)
6. [Named Groups](#named-groups)
7. [Non-capturing Groups](#non-capturing-groups)
8. [Lookahead และ Lookbehind](#lookahead-และ-lookbehind)
9. [Backreferences](#backreferences)
10. [re Module](#re-module)
11. [Flags](#flags)
12. [Compiled Patterns](#compiled-patterns)
13. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
14. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Regex Syntax พื้นฐาน

**Regular Expression (Regex)** คือ pattern สำหรับค้นหาและจับคู่ข้อความ เป็นภาษากลางที่ใช้ได้ในหลายภาษาโปรแกรม

### Metacharacters ที่สำคัญ

| Symbol | ความหมาย | ตัวอย่าง |
|--------|----------|---------|
| `.` | ตัวอักษรใดก็ได้ (ยกเว้น newline) | `a.c` ตรงกับ "abc", "aXc" |
| `*` | 0 ครั้งหรือมากกว่า | `ab*` ตรงกับ "a", "ab", "abb" |
| `+` | 1 ครั้งหรือมากกว่า | `ab+` ตรงกับ "ab", "abb" แต่ไม่ใช่ "a" |
| `?` | 0 หรือ 1 ครั้ง | `ab?` ตรงกับ "a", "ab" |
| `^` | เริ่มต้น string | `^Hello` ตรงกับ "Hello World" |
| `$` | สิ้นสุด string | `World$` ตรงกับ "Hello World" |
| `\|` | หรือ (alternation) | `cat\|dog` ตรงกับ "cat" หรือ "dog" |
| `()` | Grouping | `(ab)+` ตรงกับ "ab", "abab" |
| `[]` | Character class | `[aeiou]` ตรงกับ vowel ใดก็ได้ |
| `\\` | Escape metacharacter | `\.` ตรงกับ "." จริงๆ |
| `{}` | Quantifier | `a{3}` ตรงกับ "aaa" |

### ตัวอย่าง 1: ทดสอบ pattern พื้นฐาน

```python
import re

# ฟังก์ชัน helper สำหรับแสดงผล
def test_pattern(pattern, text, flags=0):
    """ทดสอบ pattern และแสดงผลอย่างชัดเจน"""
    match = re.search(pattern, text, flags)
    if match:
        print(f"  Pattern '{pattern}' ✓ พบใน '{text}'")
        print(f"    Match: '{match.group()}'  Position: {match.span()}")
    else:
        print(f"  Pattern '{pattern}' ✗ ไม่พบใน '{text}'")

# . ตรงกับอักษรใดก็ได้
test_pattern(r"a.c", "abc")    # ตรง: 'abc'
test_pattern(r"a.c", "aXc")   # ตรง: 'aXc'
test_pattern(r"a.c", "ac")    # ไม่ตรง

# .* ตรงกับอะไรก็ได้
test_pattern(r"Hello.*World", "Hello Beautiful World")
test_pattern(r"Hello.*World", "HelloWorld")

# escape metacharacter
test_pattern(r"3\.14", "pi = 3.14")  # ตรง
test_pattern(r"3\.14", "3X14")       # ไม่ตรง (. ถูก escape)

# alternation
test_pattern(r"cat|dog|bird", "I have a dog")
test_pattern(r"cat|dog|bird", "I have a fish")
```

### ตัวอย่าง 2: Raw Strings ใน Python

```python
import re

# ทำไมต้องใช้ r"" (raw string)?
# '\n' ใน regular string = newline character
# r'\n' ใน raw string = สองอักขระ \ และ n

# ไม่ใช้ raw string - ต้อง escape สองชั้น
pattern1 = "\\d+"  # \d ใน regex = digit
# ใช้ raw string - ง่ายกว่ามาก
pattern2 = r"\d+"  # เหมือนกัน แต่อ่านง่ายกว่า

text = "สั่งซื้อสินค้า 42 ชิ้น ราคา 1500 บาท"
print(re.findall(pattern1, text))  # ['42', '1500']
print(re.findall(pattern2, text))  # ['42', '1500']

# ตัวอย่างที่เห็นความต่างชัด
# Pattern สำหรับ word boundary
bad  = "\\bword\\b"  # ต้อง escape สองชั้น
good = r"\bword\b"   # raw string - อ่านง่ายกว่า

print(bool(re.search(good, "find the word here")))  # True
```

---

## Character Classes

### ตัวอย่าง 3: Character Classes พื้นฐาน

```python
import re

text = "Hello World 123 Thai: สวัสดี"

# [abc] - ตัวใดตัวหนึ่งใน set
vowels = re.findall(r"[aeiouAEIOU]", "Hello World")
print(f"Vowels: {vowels}")  # ['e', 'o', 'o']

# [^abc] - ไม่ใช่ตัวใดตัวหนึ่งใน set (negation)
non_vowels = re.findall(r"[^aeiou\s]", "hello world")
print(f"Non-vowels: {non_vowels}")  # ['h', 'l', 'l', 'w', 'r', 'l', 'd']

# [a-z] - range
lowercase = re.findall(r"[a-z]+", "Hello World 123")
print(f"Lowercase: {lowercase}")  # ['ello', 'orld']

# [a-zA-Z] - uppercase และ lowercase
letters = re.findall(r"[a-zA-Z]+", "Hello World 123")
print(f"Letters: {letters}")  # ['Hello', 'World']

# [0-9] เหมือน \d
digits = re.findall(r"[0-9]+", "abc123def456")
print(f"Digits: {digits}")  # ['123', '456']

# รวม ranges
alphanumeric = re.findall(r"[a-zA-Z0-9]+", "hello_123 world-456")
print(f"Alphanumeric: {alphanumeric}")  # ['hello', '123', 'world', '456']
```

### ตัวอย่าง 4: Special Character Classes

```python
import re

text = "Phone: 081-234-5678, Email: test@example.com, Score: 95.5"

# \d - digit [0-9]
digits = re.findall(r"\d+", text)
print(f"\\d+ (digits): {digits}")
# ['081', '234', '5678', '95', '5']

# \D - non-digit [^0-9]
non_digits = re.findall(r"\D+", text)
print(f"\\D+ (non-digits): {non_digits[:3]}")

# \w - word character [a-zA-Z0-9_]
words = re.findall(r"\w+", text)
print(f"\\w+ (words): {words}")

# \W - non-word character [^a-zA-Z0-9_]
non_words = re.findall(r"\W+", text)
print(f"\\W+ (non-words): {non_words}")

# \s - whitespace [\t\n\r\f\v ]
parts = re.split(r"\s+", "Hello   World\tFoo\nBar")
print(f"\\s+ split: {parts}")  # ['Hello', 'World', 'Foo', 'Bar']

# \S - non-whitespace
non_ws = re.findall(r"\S+", "Hello   World\tFoo")
print(f"\\S+ (non-whitespace): {non_ws}")  # ['Hello', 'World', 'Foo']

# \b - word boundary
sentence = "cat concatenate category catch"
cat_words = re.findall(r"\bcat\b", sentence)
print(f"\\bcat\\b (exact 'cat'): {cat_words}")  # ['cat']

cat_prefix = re.findall(r"\bcat\w*", sentence)
print(f"\\bcat\\w* (starts with 'cat'): {cat_prefix}")
# ['cat', 'concatenate', 'category', 'catch']
```

---

## Quantifiers

### ตัวอย่าง 5: Greedy Quantifiers

```python
import re

html = "<h1>หัวข้อ</h1><p>เนื้อหา</p>"

# * - 0 ครั้งหรือมากกว่า (greedy)
print(re.findall(r"<.*>", html))
# ['<h1>หัวข้อ</h1><p>เนื้อหา</p>'] - greedy! จับทั้งหมด

# *? - non-greedy (lazy)
print(re.findall(r"<.*?>", html))
# ['<h1>', '</h1>', '<p>', '</p>'] - lazy! จับน้อยที่สุด

# + - 1 ครั้งหรือมากกว่า
print(re.findall(r"\d+", "a1 b22 c333"))   # ['1', '22', '333']
print(re.findall(r"\d+?", "a1 b22 c333"))  # ['1', '2', '2', '3', '3', '3']

# ? - 0 หรือ 1 ครั้ง
print(re.findall(r"colou?r", "color colour"))
# ['color', 'colour'] - u เป็น optional
```

### ตัวอย่าง 6: Quantifiers ที่กำหนดจำนวน

```python
import re

text = "a aa aaa aaaa aaaaa"

# {n} - ตรงตามจำนวน
print(re.findall(r"a{3}", text))    # ['aaa', 'aaa', 'aaa'] (ส่วนของ aaaa, aaaaa ด้วย)

# {n,} - n ครั้งหรือมากกว่า
print(re.findall(r"\ba{3,}\b", text))  # ['aaa', 'aaaa', 'aaaaa']

# {n,m} - n ถึง m ครั้ง (greedy)
print(re.findall(r"\ba{2,4}\b", text))  # ['aa', 'aaa', 'aaaa']

# ตัวอย่างจริง: validate phone number format
phones = ["0812345678", "081-234-5678", "02-123-4567", "123", "08123456789"]
pattern = r"^(\d{2,3}-?\d{3}-?\d{4})$"

for phone in phones:
    if re.match(pattern, phone):
        print(f"  ✓ '{phone}' เป็นเบอร์โทรที่ถูกต้อง")
    else:
        print(f"  ✗ '{phone}' ไม่ถูกต้อง")

# ตัวอย่าง: validate password strength
def check_password(pwd):
    checks = {
        "ความยาว >= 8": len(pwd) >= 8,
        "มีตัวพิมพ์ใหญ่": bool(re.search(r"[A-Z]", pwd)),
        "มีตัวพิมพ์เล็ก": bool(re.search(r"[a-z]", pwd)),
        "มีตัวเลข": bool(re.search(r"\d", pwd)),
        "มีสัญลักษณ์": bool(re.search(r"[!@#$%^&*(),.?\":{}|<>]", pwd))
    }
    score = sum(checks.values())
    return checks, score

password = "MyP@ss123"
checks, score = check_password(password)
print(f"\nPassword '{password}':")
for check, passed in checks.items():
    print(f"  {'✓' if passed else '✗'} {check}")
print(f"  Score: {score}/5")
```

---

## Anchors

### ตัวอย่าง 7: Start/End Anchors

```python
import re

# ^ - เริ่มต้น string
print(re.search(r"^Hello", "Hello World"))   # Match
print(re.search(r"^Hello", "Say Hello"))     # No match (ไม่ได้เริ่มต้น)

# $ - สิ้นสุด string
print(re.search(r"World$", "Hello World"))   # Match
print(re.search(r"World$", "World Peace"))   # No match (ไม่ได้สิ้นสุด)

# ^ และ $ ร่วมกัน - match ทั้ง string
lines = ["hello", "hello world", "HELLO", "  hello"]
for line in lines:
    if re.match(r"^hello$", line, re.IGNORECASE):
        print(f"  ✓ '{line}' ตรงกับ 'hello' (case-insensitive)")
    else:
        print(f"  ✗ '{line}' ไม่ตรง")

# \b - word boundary
text = "cat cats catfish concatenate"
print("\nผลลัพธ์:")
print(re.findall(r"\bcat\b", text))       # ['cat'] - exact word
print(re.findall(r"\bcat", text))         # ['cat', 'cat', 'cat', 'cat'] - starts with cat
print(re.findall(r"cat\b", text))         # ['cat', 'cat'] - ends with cat

# \B - non-word boundary
print(re.findall(r"\Bcat\B", "concatenate"))  # ['cat'] - ภายในคำ
```

---

## Groups และ Capturing

### ตัวอย่าง 8: Capturing Groups

```python
import re

# () สร้าง capturing group
match = re.search(r"(\d{4})-(\d{2})-(\d{2})", "วันที่: 2024-01-15")
if match:
    print(f"Match ทั้งหมด: {match.group(0)}")  # หรือ match.group()
    print(f"Group 1 (year): {match.group(1)}")
    print(f"Group 2 (month): {match.group(2)}")
    print(f"Group 3 (day): {match.group(3)}")
    print(f"Groups ทั้งหมด: {match.groups()}")

# findall กับ groups
dates = re.findall(r"(\d{4})-(\d{2})-(\d{2})", 
                   "วันนี้ 2024-01-15 วันพรุ่ง 2024-01-16")
print(f"\nDates: {dates}")
# [('2024', '01', '15'), ('2024', '01', '16')]

# groups ใช้ใน substitution
text = "name: john doe, age: 25"
result = re.sub(r"(\w+): (\w+)", r"\2 [\1]", text)
print(f"\nAfter sub: {result}")
# name: john doe, age: 25 → john [name] doe, 25 [age]
```

---

## Named Groups

Named groups ทำให้โค้ดอ่านง่ายขึ้น ใช้ `(?P<name>...)` syntax

### ตัวอย่าง 9: Named Groups

```python
import re

# (?P<name>...) สร้าง named group
date_pattern = r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})"
match = re.search(date_pattern, "วันเกิด: 1995-06-15")

if match:
    print(f"Year:  {match.group('year')}")   # 1995
    print(f"Month: {match.group('month')}")  # 06
    print(f"Day:   {match.group('day')}")    # 15
    print(f"Dict:  {match.groupdict()}")
    # {'year': '1995', 'month': '06', 'day': '15'}

# ตัวอย่างที่ซับซ้อน: parse URL
url_pattern = r"""(?x)    # verbose mode - allow whitespace and comments
    (?P<scheme>https?|ftp)   # protocol
    ://
    (?P<host>[^:/]+)         # hostname
    (?::(?P<port>\d+))?      # optional port
    (?P<path>/[^?#]*)?       # path
    (?:\?(?P<query>[^#]*))?  # optional query string
    (?:\#(?P<fragment>.*))?  # optional fragment
"""

urls = [
    "https://www.example.com/path/to/page?id=42&lang=th#section1",
    "http://api.example.com:8080/api/v1/users",
    "https://google.com"
]

for url in urls:
    m = re.match(url_pattern, url, re.VERBOSE)
    if m:
        d = m.groupdict()
        print(f"\nURL: {url}")
        for key, value in d.items():
            if value:
                print(f"  {key}: {value}")
```

### ตัวอย่าง 10: Named Groups ใน sub()

```python
import re

# ใช้ named groups ใน replacement
log_line = "2024-01-15 10:30:45 ERROR Failed to connect"
pattern = r"(?P<date>\d{4}-\d{2}-\d{2}) (?P<time>\d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.*)"

# ใช้ \g<name> ใน replacement
formatted = re.sub(pattern, r"[\g<level>] \g<date> \g<time>: \g<message>", log_line)
print(formatted)
# [ERROR] 2024-01-15 10:30:45: Failed to connect

# ใช้ฟังก์ชัน replacement
def format_log(match):
    d = match.groupdict()
    level_icons = {"ERROR": "❌", "WARNING": "⚠️", "INFO": "ℹ️", "DEBUG": "🔍"}
    icon = level_icons.get(d['level'], "•")
    return f"{icon} [{d['date']} {d['time']}] {d['message']}"

print(re.sub(pattern, format_log, log_line))
```

---

## Non-capturing Groups

### ตัวอย่าง 11: Non-capturing Groups (?:...)

```python
import re

# (?:...) จัดกลุ่มโดยไม่ capture
# ใช้เมื่อต้องการ grouping แต่ไม่ต้องการ capture ค่า

# ต้องการจับคู่วันในสัปดาห์
text = "Today is Monday and yesterday was Sunday"

# แบบ capturing group
matches1 = re.findall(r"(Mon|Tue|Wed|Thu|Fri|Sat|Sun)day", text)
print(f"With capture: {matches1}")  # ['Mon', 'Sun'] - แค่ส่วนที่ capture!

# แบบ non-capturing group
matches2 = re.findall(r"(?:Mon|Tue|Wed|Thu|Fri|Sat|Sun)day", text)
print(f"Non-capture: {matches2}")   # ['Monday', 'Sunday'] - ทั้งคำ!

# ตัวอย่างจริง: parse log อย่างมีประสิทธิภาพ
log = "2024-01-15 [ERROR] Server crashed: Connection timeout"
pattern = r"(\d{4}-\d{2}-\d{2}) \[(\w+)\] (?:\w+ (?:\w+ )?)?(.+)"

match = re.search(pattern, log)
if match:
    date, level, msg = match.groups()
    print(f"Date: {date}, Level: {level}, Message: {msg}")
```

---

## Lookahead และ Lookbehind

**Lookahead** และ **Lookbehind** ตรวจสอบ context แต่ไม่รวมใน match

| Syntax | ชื่อ | ความหมาย |
|--------|------|----------|
| `(?=...)` | Positive Lookahead | ตามด้วย pattern |
| `(?!...)` | Negative Lookahead | ไม่ตามด้วย pattern |
| `(?<=...)` | Positive Lookbehind | นำหน้าด้วย pattern |
| `(?<!...)` | Negative Lookbehind | ไม่ได้นำหน้าด้วย pattern |

### ตัวอย่าง 12: Lookahead

```python
import re

# Positive Lookahead (?=...)
# หา "100" ที่ตามด้วย " USD"
prices = "100 USD, 200 THB, 300 USD, 400 EUR"
usd_prices = re.findall(r"\d+ (?=USD)", prices)
print(f"USD prices: {usd_prices}")  # ['100 ', '300 '] (รวม space)

# แก้ด้วยการไม่รวม space
usd_prices2 = re.findall(r"\d+(?= USD)", prices)
print(f"USD prices: {usd_prices2}")  # ['100', '300']

# Negative Lookahead (?!...)
# หา "100" ที่ไม่ตามด้วย " USD"
non_usd = re.findall(r"\d+(?! USD)", prices)
print(f"Non-USD prices: {non_usd}")

# ตัวอย่างจริง: หา filename ที่ไม่ใช่ .txt
filenames = "file1.txt file2.py file3.txt file4.js"
non_txt = re.findall(r"\w+\.(?!txt)\w+", filenames)
print(f"Non-txt files: {non_txt}")  # ['file2.py', 'file4.js']
```

### ตัวอย่าง 13: Lookbehind

```python
import re

# Positive Lookbehind (?<=...)
# หา ตัวเลขที่นำหน้าด้วย "$"
text = "Price: $100, Cost: $200, Quantity: 50"
usd_amounts = re.findall(r"(?<=\$)\d+", text)
print(f"USD amounts: {usd_amounts}")  # ['100', '200']

# Negative Lookbehind (?<!...)
# หา ตัวเลขที่ไม่ได้นำหน้าด้วย "$"
non_usd_amounts = re.findall(r"(?<!\$)\d+", text)
print(f"Non-USD amounts: {non_usd_amounts}")

# รวม Lookahead + Lookbehind
# หาคำที่อยู่ระหว่าง < และ >
html = "<title>My Page</title><meta name='desc'>"
tags = re.findall(r"(?<=<)[^>]+(?=>)", html)
print(f"Tag contents: {tags}")  # ['title', '/title', "meta name='desc'"]

# ตัวอย่าง: เพิ่ม comma สำหรับตัวเลขใหญ่
def add_thousands_sep(number_str):
    """แปลง '1000000' เป็น '1,000,000'"""
    return re.sub(r"(?<=\d)(?=(\d{3})+$)", ",", number_str)

print(add_thousands_sep("1000000"))    # 1,000,000
print(add_thousands_sep("1234567890")) # 1,234,567,890
```

---

## Backreferences

**Backreference** อ้างถึง captured group ที่เจอแล้วในการ match

### ตัวอย่าง 14: Backreferences

```python
import re

# \1 อ้างถึง group 1 ที่ capture ไว้แล้ว
# หาคำที่ซ้ำกัน
text = "the the quick brown fox fox"
duplicates = re.findall(r"\b(\w+)\s+\1\b", text)
print(f"คำที่ซ้ำ: {duplicates}")  # ['the', 'fox']

# หาและลบคำที่ซ้ำ
cleaned = re.sub(r"\b(\w+)\s+\1\b", r"\1", text)
print(f"หลังลบซ้ำ: {cleaned}")  # "the quick brown fox"

# ตรวจสอบ HTML tags ที่สมบูรณ์ (opening/closing match)
html_snippets = [
    "<h1>หัวข้อ</h1>",
    "<p>เนื้อหา</p>",
    "<h1>หัวข้อ</h2>",   # ไม่สมบูรณ์
    "<div>เนื้อหา</span>" # ไม่สมบูรณ์
]

pattern = r"<([a-z][a-z0-9]*)[^>]*>.*?</\1>"
for html in html_snippets:
    if re.fullmatch(pattern, html, re.DOTALL):
        print(f"  ✓ '{html}' - tags สมบูรณ์")
    else:
        print(f"  ✗ '{html}' - tags ไม่สมบูรณ์")

# Named backreferences
pattern2 = r"(?P<tag>[a-z]+).*?(?P=tag)"  # (?P=name) อ้างถึง named group
match = re.search(pattern2, "div content div")
if match:
    print(f"\nMatched repeated: '{match.group()}'")
```

---

## re Module

### ตัวอย่าง 15: re.search() vs re.match()

```python
import re

text = "Hello World 123"

# re.search() - ค้นหาทุกที่ใน string
m = re.search(r"\d+", text)
print(f"search: {m.group()}")  # '123'

# re.match() - ค้นหาเฉพาะที่ตำแหน่งเริ่มต้น
m = re.match(r"\d+", text)
print(f"match: {m}")  # None - ไม่พบเพราะไม่ได้เริ่มต้น

m = re.match(r"Hello", text)
print(f"match Hello: {m.group()}")  # 'Hello'

# re.fullmatch() - ต้องตรงทั้ง string
m = re.fullmatch(r"\w+", "hello")
print(f"fullmatch word: {m.group()}")   # 'hello'

m = re.fullmatch(r"\w+", "hello world")
print(f"fullmatch words: {m}")   # None - มี space
```

### ตัวอย่าง 16: re.findall() และ re.finditer()

```python
import re

text = "2024-01-15 เวลา 10:30 และ 2024-01-16 เวลา 14:00"

# findall() - return list ของ matches
dates = re.findall(r"\d{4}-\d{2}-\d{2}", text)
print(f"Dates: {dates}")  # ['2024-01-15', '2024-01-16']

times = re.findall(r"\d{2}:\d{2}", text)
print(f"Times: {times}")  # ['10:30', '14:00']

# findall กับ groups - return list of tuples
pattern = r"(\d{4})-(\d{2})-(\d{2})"
parts = re.findall(pattern, text)
print(f"Date parts: {parts}")
# [('2024', '01', '15'), ('2024', '01', '16')]

# finditer() - return iterator ของ match objects
print("\nfinditer results:")
for match in re.finditer(r"\d{4}-\d{2}-\d{2}", text):
    print(f"  '{match.group()}' at position {match.start()}-{match.end()}")

# ข้อดีของ finditer: ได้ match object พร้อม position info
for match in re.finditer(r"(\d{4})-(\d{2})-(\d{2})", text):
    y, m, d = match.groups()
    print(f"  Year={y}, Month={m}, Day={d}, Span={match.span()}")
```

### ตัวอย่าง 17: re.sub() - การแทนที่

```python
import re

# sub(pattern, replacement, string)
text = "Hello     World   Python"

# ลดช่องว่างซ้ำเป็นช่องเดียว
cleaned = re.sub(r"\s+", " ", text)
print(f"Cleaned: '{cleaned}'")  # 'Hello World Python'

# ใช้ \n ใน replacement (backreference)
text2 = "John Smith, Jane Doe, Bob Jones"
# สลับชื่อ-นามสกุล
swapped = re.sub(r"(\w+) (\w+)", r"\2, \1", text2)
print(f"Swapped: {swapped}")
# 'Smith, John, Doe, Jane, Jones, Bob'

# ใช้ฟังก์ชันเป็น replacement
def censor_word(match):
    word = match.group()
    if len(word) <= 2:
        return "*" * len(word)
    return word[0] + "*" * (len(word)-2) + word[-1]

bad_text = "The damn system failed again"
censored = re.sub(r"\b\w{4,}\b", censor_word, bad_text)
print(f"Censored: {censored}")

# re.subn() - return tuple (new_string, count)
result, count = re.subn(r"\d+", "NUM", "abc123def456ghi789")
print(f"Result: {result}, Replacements: {count}")
# ('abcNUMdefNUMghiNUM', 3)
```

### ตัวอย่าง 18: re.split()

```python
import re

# split() แบ่ง string ด้วย pattern
text = "one1two2three3four"

# แบ่งด้วย digit
parts = re.split(r"\d", text)
print(parts)  # ['one', 'two', 'three', 'four']

# ใช้ capturing group - รวม delimiter ใน result
parts_with_sep = re.split(r"(\d)", text)
print(parts_with_sep)  # ['one', '1', 'two', '2', 'three', '3', 'four']

# split หลาย delimiters
csv_like = "apple,banana;cherry|date"
fruits = re.split(r"[,;|]", csv_like)
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# split พร้อม maxsplit
text = "a:b:c:d:e"
parts = re.split(r":", text, maxsplit=2)
print(parts)  # ['a', 'b', 'c:d:e']

# ตัวอย่างจริง: แบ่ง sentence เป็น tokens
sentence = "Hello, World! How are you?"
tokens = re.split(r"[\s,!?]+", sentence)
tokens = [t for t in tokens if t]  # ลบ empty strings
print(f"Tokens: {tokens}")
```

---

## Flags

### ตัวอย่าง 19: re.IGNORECASE, re.MULTILINE, re.DOTALL

```python
import re

# re.IGNORECASE (re.I) - ไม่สนใจ case
text = "Hello WORLD hello World"
matches = re.findall(r"hello", text, re.IGNORECASE)
print(f"IGNORECASE: {matches}")  # ['Hello', 'hello', 'Hello'] - แต่ case ต่างกัน... 
# จริงๆ: ['Hello', 'hello', 'Hello'] - ทุก variant ของ hello

# re.MULTILINE (re.M) - ^ และ $ ตรงกับ แต่ละบรรทัด
multiline_text = """First line
Second line
Third line"""

starts = re.findall(r"^\w+", multiline_text, re.MULTILINE)
print(f"MULTILINE starts: {starts}")  # ['First', 'Second', 'Third']

ends = re.findall(r"\w+$", multiline_text, re.MULTILINE)
print(f"MULTILINE ends: {ends}")  # ['line', 'line', 'line']

# re.DOTALL (re.S) - . ตรงกับ newline ด้วย
html = "<div>\n  Content\n</div>"

without_dotall = re.search(r"<div>(.*)</div>", html)
print(f"Without DOTALL: {without_dotall}")  # None!

with_dotall = re.search(r"<div>(.*)</div>", html, re.DOTALL)
print(f"With DOTALL: '{with_dotall.group(1)}'")
# '\n  Content\n'

# รวม flags
text3 = "Hello\nworld"
m = re.search(r"^world$", text3, re.MULTILINE | re.IGNORECASE)
print(f"Combined flags: {m.group()}")  # 'world'

# re.VERBOSE (re.X) - อนุญาตให้มี whitespace และ comment
email_pattern = re.compile(r"""
    ^                   # เริ่มต้น string
    [a-zA-Z0-9._%+-]+  # username
    @                   # @ symbol
    [a-zA-Z0-9.-]+     # domain name
    \.                  # dot
    [a-zA-Z]{2,}       # TLD
    $                   # สิ้นสุด string
""", re.VERBOSE)

emails = ["user@example.com", "invalid@", "test.user@domain.co.th"]
for email in emails:
    if email_pattern.match(email):
        print(f"  ✓ {email}")
    else:
        print(f"  ✗ {email}")
```

---

## Compiled Patterns

### ตัวอย่าง 20: การ Compile Pattern

```python
import re
import time

# Compile pattern เมื่อต้องใช้บ่อยๆ
email_re = re.compile(
    r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    re.IGNORECASE
)

# ใช้ compiled pattern
emails = [
    "user@example.com",
    "invalid-email",
    "test@domain.co.th",
    "@nodomain.com",
    "nodot@nodot"
]

for email in emails:
    if email_re.match(email):
        print(f"  ✓ Valid: {email}")
    else:
        print(f"  ✗ Invalid: {email}")

# เปรียบเทียบความเร็ว
text = "test 12345 more text 67890 end"
pattern_str = r"\d+"
pattern_compiled = re.compile(r"\d+")

# ทั้งสองวิธีได้ผลเหมือนกัน
print(re.findall(pattern_str, text))
print(pattern_compiled.findall(text))

# Compiled pattern มี methods เหมือน re module
m = pattern_compiled.search(text)
all_m = pattern_compiled.findall(text)
sub_m = pattern_compiled.sub("NUM", text)
print(f"Found: {all_m}")
print(f"Substituted: {sub_m}")
```

### ตัวอย่าง 21: Pattern สำหรับใช้บ่อยๆ

```python
import re

class Patterns:
    """รวบรวม compiled patterns ที่ใช้บ่อย"""
    
    # Email
    EMAIL = re.compile(
        r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    )
    
    # URL
    URL = re.compile(
        r'https?://(?:www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}'
        r'\.[a-zA-Z0-9()]{1,6}\b(?:[-a-zA-Z0-9()@:%_\+.~#?&/=]*)'
    )
    
    # Thai phone number
    THAI_PHONE = re.compile(r'^0[689]\d{8}$')
    
    # Thai ID card (13 digits)
    THAI_ID = re.compile(r'^\d{13}$')
    
    # IP address (IPv4)
    IPV4 = re.compile(
        r'^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}'
        r'(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$'
    )
    
    # Date formats
    DATE_ISO = re.compile(r'^\d{4}-\d{2}-\d{2}$')
    DATE_TH  = re.compile(r'^\d{2}/\d{2}/\d{4}$')
    
    @classmethod
    def validate_email(cls, email):
        return bool(cls.EMAIL.match(email))
    
    @classmethod
    def validate_phone(cls, phone):
        return bool(cls.THAI_PHONE.match(phone.replace("-", "").replace(" ", "")))
    
    @classmethod
    def validate_ip(cls, ip):
        return bool(cls.IPV4.match(ip))

# ทดสอบ
tests = {
    "email": ["user@example.com", "bad-email", "test@co.th"],
    "phone": ["0812345678", "02-123-4567", "12345"],
    "ip": ["192.168.1.1", "256.1.1.1", "10.0.0.1"],
}

for category, values in tests.items():
    print(f"\n{category}:")
    for v in values:
        method = getattr(Patterns, f"validate_{category}")
        print(f"  {'✓' if method(v) else '✗'} {v}")
```

---

## ตัวอย่างโปรแกรมจริง

### ตัวอย่าง 22: Email Validator สมบูรณ์

```python
import re
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class EmailValidation:
    email: str
    is_valid: bool
    errors: List[str]
    parts: Optional[dict] = None

class EmailValidator:
    """Email validator ที่ครบถ้วน"""
    
    # Pattern ตาม RFC 5322
    PATTERN = re.compile(r"""(?x)
        ^
        (?P<local>
            (?:[a-zA-Z0-9]        # เริ่มด้วย alphanumeric
            (?:[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]*
            [a-zA-Z0-9])?)        # จบด้วย alphanumeric
        )
        @
        (?P<domain>
            (?:[a-zA-Z0-9]        # เริ่มด้วย alphanumeric
            (?:[a-zA-Z0-9-]*      # อาจมี hyphen
            [a-zA-Z0-9])?         # จบด้วย alphanumeric
            \.)+                  # dot
            [a-zA-Z]{2,}          # TLD
        )
        $
    """)
    
    COMMON_DOMAINS = {"gmail.com", "yahoo.com", "hotmail.com", "outlook.com"}
    
    def validate(self, email: str) -> EmailValidation:
        errors = []
        
        # ตรวจสอบพื้นฐาน
        if not email:
            return EmailValidation(email, False, ["Email ว่างเปล่า"])
        
        if len(email) > 254:
            errors.append("Email ยาวเกินไป (max 254 chars)")
        
        if " " in email:
            errors.append("Email ไม่ควรมีช่องว่าง")
        
        # ตรวจสอบ pattern
        match = self.PATTERN.match(email.lower())
        if not match:
            errors.append("รูปแบบ email ไม่ถูกต้อง")
            return EmailValidation(email, False, errors)
        
        parts = match.groupdict()
        local = parts['local']
        domain = parts['domain']
        
        # ตรวจสอบ local part
        if len(local) > 64:
            errors.append("Local part ยาวเกินไป (max 64 chars)")
        
        if local.startswith('.') or local.endswith('.'):
            errors.append("Local part ไม่ควรเริ่มหรือจบด้วย '.'")
        
        if '..' in local:
            errors.append("Local part ไม่ควรมี '..' ต่อกัน")
        
        # ตรวจสอบ domain
        domain_parts = domain.rstrip('.').split('.')
        tld = domain_parts[-1]
        
        if len(tld) < 2:
            errors.append("TLD สั้นเกินไป")
        
        is_valid = len(errors) == 0
        return EmailValidation(
            email=email,
            is_valid=is_valid,
            errors=errors,
            parts=parts if is_valid else None
        )

# ทดสอบ
validator = EmailValidator()
test_emails = [
    "user@example.com",
    "user.name+tag@domain.co.th",
    "invalid@",
    "@nodomain.com",
    "user..double@example.com",
    "very.long." + "x" * 60 + "@example.com",
    "user@gmail.com",
    "test.user@subdomain.example.org"
]

print("Email Validation Results:")
print("=" * 60)
for email in test_emails:
    result = validator.validate(email)
    status = "✓ VALID" if result.is_valid else "✗ INVALID"
    print(f"{status}: {email}")
    if not result.is_valid:
        for err in result.errors:
            print(f"        - {err}")
```

### ตัวอย่าง 23: Phone Number Parser

```python
import re
from dataclasses import dataclass

@dataclass
class PhoneNumber:
    raw: str
    normalized: str
    country_code: str
    area_code: str
    number: str
    extension: str = ""

class ThaiPhoneParser:
    """Parser สำหรับเบอร์โทรไทย"""
    
    PATTERNS = {
        'mobile': re.compile(
            r'^(?:\+66|0066|0)?'  # country code optional
            r'(?P<prefix>[689]\d)'  # mobile prefix
            r'[-.\s]?'
            r'(?P<part1>\d{3})'
            r'[-.\s]?'
            r'(?P<part2>\d{4})$'
        ),
        'landline': re.compile(
            r'^(?:\+66|0066|0)?'
            r'(?P<area>[2-9])'
            r'[-.\s]?'
            r'(?P<part1>\d{3})'
            r'[-.\s]?'
            r'(?P<part2>\d{4})$'
        ),
    }
    
    def parse(self, phone_str: str) -> PhoneNumber:
        # ทำความสะอาดข้อมูล
        cleaned = re.sub(r'[\s\-\.\(\)]', '', phone_str)
        
        # ตรวจสอบ country code
        country_code = "66"  # Thailand
        if cleaned.startswith('+66'):
            cleaned = '0' + cleaned[3:]
        elif cleaned.startswith('0066'):
            cleaned = '0' + cleaned[4:]
        
        # แยก extension
        ext = ""
        ext_match = re.search(r'(?:ext|x|#)\.?\s*(\d+)$', cleaned, re.I)
        if ext_match:
            ext = ext_match.group(1)
            cleaned = cleaned[:ext_match.start()].strip()
        
        # ทดสอบ patterns
        for phone_type, pattern in self.PATTERNS.items():
            m = pattern.match(cleaned)
            if m:
                d = m.groupdict()
                if phone_type == 'mobile':
                    prefix = d['prefix']
                    normalized = f"0{prefix}-{d['part1']}-{d['part2']}"
                    return PhoneNumber(
                        raw=phone_str,
                        normalized=normalized,
                        country_code=country_code,
                        area_code=prefix,
                        number=f"{d['part1']}{d['part2']}",
                        extension=ext
                    )
                else:
                    area = d['area']
                    normalized = f"0{area}-{d['part1']}-{d['part2']}"
                    return PhoneNumber(
                        raw=phone_str,
                        normalized=normalized,
                        country_code=country_code,
                        area_code=area,
                        number=f"{d['part1']}{d['part2']}"
                    )
        
        raise ValueError(f"ไม่สามารถ parse เบอร์โทร: {phone_str}")
    
    def find_all_phones(self, text: str):
        """ค้นหาเบอร์โทรทั้งหมดใน text"""
        pattern = re.compile(
            r'\b(?:\+66|0066)?0?[689]\d{1}[-.\s]?\d{3}[-.\s]?\d{4}\b'
        )
        return [m.group() for m in pattern.finditer(text)]

parser = ThaiPhoneParser()

# Test parsing
phones = [
    "0812345678",
    "081-234-5678",
    "081 234 5678",
    "+66812345678",
    "02-123-4567",
    "02.123.4567",
]

print("Phone Parsing:")
for p in phones:
    try:
        result = parser.parse(p)
        print(f"  {p:20} → {result.normalized}")
    except ValueError as e:
        print(f"  {p:20} → Error: {e}")

# Test find_all
text = "ติดต่อได้ที่ 02-123-4567 หรือ 0812345678 ทุกวัน"
found = parser.find_all_phones(text)
print(f"\nพบเบอร์โทรใน text: {found}")
```

### ตัวอย่าง 24: HTML Tag Extractor

```python
import re
from typing import List, Dict, Optional
from dataclasses import dataclass, field

@dataclass
class HTMLTag:
    tag: str
    attributes: Dict[str, str]
    content: str
    raw: str
    is_self_closing: bool = False

class HTMLExtractor:
    """Simple HTML tag extractor ด้วย regex"""
    
    TAG_PATTERN = re.compile(
        r'<(?P<tag>[a-zA-Z][a-zA-Z0-9-]*)'  # tag name
        r'(?P<attrs>[^>]*)'                   # attributes
        r'(?P<self_close>/?)>',              # self-closing
        re.IGNORECASE
    )
    
    ATTR_PATTERN = re.compile(
        r'(?P<name>[a-zA-Z_:][a-zA-Z0-9_:.-]*)'  # attribute name
        r'(?:\s*=\s*'
        r'(?:'
        r'"(?P<dq>[^"]*)"'    # double quoted value
        r"|'(?P<sq>[^']*)'"   # single quoted value  
        r'|(?P<uq>\S+)'       # unquoted value
        r'))?',
        re.IGNORECASE
    )
    
    def extract_tags(self, html: str, tag_name: Optional[str] = None) -> List[HTMLTag]:
        """ดึง tags ทั้งหมด หรือ tag ที่กำหนด"""
        results = []
        
        for match in self.TAG_PATTERN.finditer(html):
            tag = match.group('tag').lower()
            
            if tag_name and tag.lower() != tag_name.lower():
                continue
            
            # Parse attributes
            attrs_str = match.group('attrs')
            attrs = {}
            for attr_match in self.ATTR_PATTERN.finditer(attrs_str):
                name = attr_match.group('name')
                # ดึงค่าจาก group ที่ match
                value = (attr_match.group('dq') or 
                        attr_match.group('sq') or 
                        attr_match.group('uq') or 
                        "")  # boolean attribute
                attrs[name.lower()] = value
            
            is_self_closing = bool(match.group('self_close'))
            
            # หา content (สำหรับ paired tags)
            content = ""
            if not is_self_closing:
                tag_end_pattern = re.compile(
                    rf'</\s*{re.escape(tag)}\s*>',
                    re.IGNORECASE
                )
                end_match = tag_end_pattern.search(html, match.end())
                if end_match:
                    content = html[match.end():end_match.start()]
            
            results.append(HTMLTag(
                tag=tag,
                attributes=attrs,
                content=content.strip(),
                raw=match.group(),
                is_self_closing=is_self_closing
            ))
        
        return results
    
    def extract_links(self, html: str) -> List[Dict]:
        """ดึง links ทั้งหมด"""
        links = []
        for tag in self.extract_tags(html, 'a'):
            links.append({
                'href': tag.attributes.get('href', ''),
                'text': re.sub(r'<[^>]+>', '', tag.content),  # strip inner tags
                'title': tag.attributes.get('title', ''),
                'target': tag.attributes.get('target', '_self')
            })
        return links
    
    def extract_images(self, html: str) -> List[Dict]:
        """ดึง images ทั้งหมด"""
        images = []
        for tag in self.extract_tags(html, 'img'):
            images.append({
                'src': tag.attributes.get('src', ''),
                'alt': tag.attributes.get('alt', ''),
                'width': tag.attributes.get('width', ''),
                'height': tag.attributes.get('height', '')
            })
        return images

# ทดสอบ
html = """
<html>
<head>
    <title>My Page</title>
    <meta charset="utf-8"/>
    <link rel="stylesheet" href="style.css"/>
</head>
<body>
    <h1 class="title" id="main-title">Welcome</h1>
    <p>Visit <a href="https://example.com" title="Example" target="_blank">Example.com</a></p>
    <img src="photo.jpg" alt="A photo" width="800" height="600"/>
    <a href="/about">About Us</a>
</body>
</html>
"""

extractor = HTMLExtractor()

print("=== Links ===")
for link in extractor.extract_links(html):
    print(f"  [{link['text']}] → {link['href']}")

print("\n=== Images ===")
for img in extractor.extract_images(html):
    print(f"  src={img['src']}, alt='{img['alt']}'")

print("\n=== All Tags ===")
for tag in extractor.extract_tags(html):
    attrs_str = ", ".join(f"{k}={v!r}" for k, v in tag.attributes.items())
    print(f"  <{tag.tag}> attrs=[{attrs_str}]")
```

### ตัวอย่าง 25: Log Parser สมบูรณ์

```python
import re
from datetime import datetime
from collections import Counter, defaultdict
from dataclasses import dataclass
from typing import List, Dict, Optional

@dataclass
class LogEntry:
    timestamp: datetime
    level: str
    module: str
    message: str
    duration_ms: Optional[float] = None
    status_code: Optional[int] = None

class LogParser:
    """Parser สำหรับ application logs"""
    
    # Log format: 2024-01-15 10:30:45.123 [ERROR] myapp.db: Message [duration=150ms] [status=500]
    MAIN_PATTERN = re.compile(r"""(?x)
        ^
        (?P<date>\d{4}-\d{2}-\d{2})        # date
        \s
        (?P<time>\d{2}:\d{2}:\d{2}         # time
        (?:\.\d{1,3})?)                    # optional milliseconds
        \s
        \[(?P<level>[A-Z]+)\]              # log level
        \s
        (?P<module>[\w.]+):               # module name
        \s
        (?P<message>.+?)                  # message
        (?:\s\[duration=(?P<duration>\d+(?:\.\d+)?)ms\])?    # optional duration
        (?:\s\[status=(?P<status>\d{3})\])?                  # optional status
        $
    """)
    
    def parse_line(self, line: str) -> Optional[LogEntry]:
        """Parse บรรทัด log เดียว"""
        match = self.MAIN_PATTERN.match(line.strip())
        if not match:
            return None
        
        d = match.groupdict()
        
        # Parse timestamp
        ts_str = f"{d['date']} {d['time']}"
        try:
            timestamp = datetime.strptime(ts_str, '%Y-%m-%d %H:%M:%S.%f')
        except ValueError:
            try:
                timestamp = datetime.strptime(ts_str, '%Y-%m-%d %H:%M:%S')
            except ValueError:
                return None
        
        return LogEntry(
            timestamp=timestamp,
            level=d['level'],
            module=d['module'],
            message=d['message'],
            duration_ms=float(d['duration']) if d['duration'] else None,
            status_code=int(d['status']) if d['status'] else None
        )
    
    def parse_file(self, lines: List[str]) -> List[LogEntry]:
        """Parse หลายบรรทัด"""
        entries = []
        for line in lines:
            entry = self.parse_line(line)
            if entry:
                entries.append(entry)
        return entries
    
    def analyze(self, entries: List[LogEntry]) -> Dict:
        """วิเคราะห์ log entries"""
        analysis = {
            'total': len(entries),
            'by_level': Counter(e.level for e in entries),
            'by_module': Counter(e.module for e in entries),
            'errors': [e for e in entries if e.level == 'ERROR'],
            'slow_requests': [
                e for e in entries 
                if e.duration_ms and e.duration_ms > 500
            ]
        }
        
        # คำนวณ average duration
        durations = [e.duration_ms for e in entries if e.duration_ms]
        if durations:
            analysis['avg_duration_ms'] = sum(durations) / len(durations)
            analysis['max_duration_ms'] = max(durations)
        
        return analysis

# ทดสอบ
sample_logs = [
    "2024-01-15 10:30:45.100 [INFO] myapp.server: Server started",
    "2024-01-15 10:30:46.200 [DEBUG] myapp.db: Connected to database",
    "2024-01-15 10:30:47.300 [INFO] myapp.api: GET /users [duration=45ms] [status=200]",
    "2024-01-15 10:30:48.400 [ERROR] myapp.db: Query failed: Connection timeout [duration=5000ms]",
    "2024-01-15 10:30:49.500 [WARNING] myapp.cache: Cache miss for key=user_42",
    "2024-01-15 10:30:50.600 [INFO] myapp.api: POST /orders [duration=120ms] [status=201]",
    "2024-01-15 10:30:51.700 [ERROR] myapp.api: Internal server error [duration=800ms] [status=500]",
    "2024-01-15 10:30:52.800 [INFO] myapp.api: GET /products [duration=35ms] [status=200]",
    "INVALID LINE - will be skipped",
]

parser = LogParser()
entries = parser.parse_file(sample_logs)
analysis = parser.analyze(entries)

print(f"Log Analysis:")
print(f"  Total entries: {analysis['total']}")
print(f"  By level: {dict(analysis['by_level'])}")
print(f"  Errors: {len(analysis['errors'])}")
for err in analysis['errors']:
    print(f"    - {err.module}: {err.message}")
if 'avg_duration_ms' in analysis:
    print(f"  Avg duration: {analysis['avg_duration_ms']:.1f}ms")
    print(f"  Max duration: {analysis['max_duration_ms']:.1f}ms")
print(f"  Slow requests (>500ms): {len(analysis['slow_requests'])}")
```

---

## แบบฝึกหัด

### ข้อ 1: Markdown Link Extractor

สร้าง regex ที่ดึง links จาก Markdown text

**คำตอบ:**

```python
import re

def extract_markdown_links(text):
    """ดึง links จาก Markdown [text](url) format"""
    pattern = re.compile(r'\[(?P<text>[^\]]+)\]\((?P<url>[^\)]+)\)')
    
    links = []
    for match in pattern.finditer(text):
        links.append({
            'text': match.group('text'),
            'url': match.group('url')
        })
    return links

markdown = """
# My Document

Visit [Google](https://google.com) or [GitHub](https://github.com).
See [documentation](docs/README.md) for more details.
"""

links = extract_markdown_links(markdown)
for link in links:
    print(f"  [{link['text']}] → {link['url']}")
```

### ข้อ 2: Password Validator

สร้าง regex สำหรับ validate password ตาม rules

**คำตอบ:**

```python
import re

def validate_password(password):
    """Validate password ตาม security rules"""
    rules = [
        (r'.{8,}',           "อย่างน้อย 8 ตัวอักษร"),
        (r'[A-Z]',           "มีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว"),
        (r'[a-z]',           "มีตัวพิมพ์เล็กอย่างน้อย 1 ตัว"),
        (r'\d',              "มีตัวเลขอย่างน้อย 1 ตัว"),
        (r'[!@#$%^&*(),.?]', "มีสัญลักษณ์พิเศษอย่างน้อย 1 ตัว"),
        (r'^(?!.*(.)\1{2,})','ไม่มีตัวอักษรซ้ำกันติดต่อกัน 3 ตัว'),
    ]
    
    results = []
    for pattern, description in rules:
        passed = bool(re.search(pattern, password))
        results.append((description, passed))
    
    return results

pwd = "MyP@ss123"
print(f"Password: {pwd}")
all_passed = True
for desc, passed in validate_password(pwd):
    icon = "✓" if passed else "✗"
    print(f"  {icon} {desc}")
    all_passed = all_passed and passed
print(f"Overall: {'VALID' if all_passed else 'INVALID'}")
```

### ข้อ 3: CSV Parser พร้อม Quote Handling

**คำตอบ:**

```python
import re

def parse_csv_line(line):
    """Parse CSV line ที่มี quoted fields"""
    pattern = re.compile(r'''(?:^|,)("(?:[^"]|"")*"|[^,]*)''')
    
    fields = []
    for match in pattern.finditer(line):
        field = match.group(1)
        if field.startswith('"') and field.endswith('"'):
            field = field[1:-1].replace('""', '"')
        fields.append(field.strip())
    return fields

lines = [
    'John,Doe,25,"New York, USA"',
    '"Smith, John","He said ""hello""",30,Bangkok',
    'Alice,Wonder,22,London'
]

for line in lines:
    fields = parse_csv_line(line)
    print(f"  {fields}")
```

### ข้อ 4-10: แบบฝึกหัดเพิ่มเติม

**ข้อ 4**: สร้าง regex สำหรับ parse Python f-string expressions

**ข้อ 5**: สร้าง Thai text tokenizer ที่แยกคำและ sentence

**ข้อ 6**: สร้าง SQL query parser ที่แยก SELECT columns, FROM table, WHERE conditions

**ข้อ 7**: สร้าง Version number parser (semantic versioning: `1.2.3-beta.1+build.123`)

**ข้อ 8**: สร้าง Credit card number validator ที่รองรับ Visa, Mastercard, Amex

**ข้อ 9**: สร้าง function ที่แปลง camelCase เป็น snake_case ด้วย regex

**ข้อ 10**: สร้าง template engine เล็กๆ ที่แทน `{{variable}}` ด้วยค่าจริง

---

## สรุป

| Feature | Syntax | ใช้เมื่อไหร่ |
|---------|--------|-------------|
| Character class | `[abc]` | ตัวใดตัวหนึ่ง |
| Negated class | `[^abc]` | ไม่ใช่ตัวที่กำหนด |
| Wildcard | `.` | อักษรใดก็ได้ |
| Quantifiers | `*`, `+`, `?`, `{n,m}` | ระบุจำนวน |
| Lazy | `*?`, `+?` | จับให้น้อยที่สุด |
| Anchors | `^`, `$`, `\b` | ตำแหน่งใน string |
| Capturing group | `(...)` | จับค่าและ reuse |
| Named group | `(?P<name>...)` | จับค่าพร้อมชื่อ |
| Non-capturing | `(?:...)` | grouping โดยไม่ capture |
| Lookahead | `(?=...)`, `(?!...)` | ตรวจ context ข้างหน้า |
| Lookbehind | `(?<=...)`, `(?<!...)` | ตรวจ context ข้างหลัง |
| Backreference | `\1`, `\g<name>` | อ้างถึง group ที่ match แล้ว |

**หลักการ:**
- ใช้ raw strings `r"..."` เสมอ
- Compile patterns ที่ใช้บ่อย
- เริ่มจาก simple แล้ว refine ให้ precise
- ใช้ `re.VERBOSE` สำหรับ pattern ซับซ้อน
- ทดสอบ edge cases เสมอ
