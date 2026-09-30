# Part 30: datetime, time & timezone Handling

## สารบัญ
1. [datetime Module ครบถ้วน](#datetime-module-ครบถ้วน)
2. [date, time, datetime, timedelta Objects](#date-time-datetime-timedelta-objects)
3. [Formatting: strftime และ strptime](#formatting-strftime-และ-strptime)
4. [Timezone-aware datetime](#timezone-aware-datetime)
5. [pytz และ zoneinfo](#pytz-และ-zoneinfo)
6. [UTC และ Timezone Conversions](#utc-และ-timezone-conversions)
7. [dateutil Library](#dateutil-library)
8. [time Module](#time-module)
9. [calendar Module](#calendar-module)
10. [Practical Patterns](#practical-patterns)
11. [ISO 8601 Format](#iso-8601-format)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## datetime Module ครบถ้วน

module `datetime` มี classes หลัก:
- `date`: วันที่ (year, month, day)
- `time`: เวลา (hour, minute, second, microsecond)
- `datetime`: วันที่และเวลา (รวม date + time)
- `timedelta`: ช่วงเวลา (duration)
- `timezone`: timezone ชนิดง่าย (offset-based)

### ตัวอย่าง 1: Import และ Overview

```python
import datetime

# ดู classes ทั้งหมดใน module
print(dir(datetime))

# Import แบบต่างๆ
from datetime import date, time, datetime, timedelta, timezone

# ค่าสูงสุด/ต่ำสุด
print(f"date.min: {date.min}")           # 0001-01-01
print(f"date.max: {date.max}")           # 9999-12-31
print(f"datetime.min: {datetime.min}")   # 0001-01-01 00:00:00
print(f"datetime.max: {datetime.max}")   # 9999-12-31 23:59:59.999999
print(f"time.min: {time.min}")           # 00:00:00
print(f"time.max: {time.max}")           # 23:59:59.999999
```

---

## date, time, datetime, timedelta Objects

### ตัวอย่าง 2: date Object

```python
from datetime import date

# สร้าง date
d1 = date(2024, 1, 15)
print(d1)               # 2024-01-15
print(type(d1))         # <class 'datetime.date'>

# Attributes
print(f"Year:  {d1.year}")   # 2024
print(f"Month: {d1.month}")  # 1
print(f"Day:   {d1.day}")    # 15

# วันนี้
today = date.today()
print(f"Today: {today}")

# สร้างจาก ordinal (จำนวนวันตั้งแต่ 1 Jan 0001)
d_ordinal = date.fromordinal(738000)
print(f"From ordinal: {d_ordinal}")

# สร้างจาก POSIX timestamp
import time
d_timestamp = date.fromtimestamp(time.time())
print(f"From timestamp: {d_timestamp}")

# วันในสัปดาห์ (0=Monday, 6=Sunday)
days = ["จันทร์", "อังคาร", "พุธ", "พฤหัสบดี", "ศุกร์", "เสาร์", "อาทิตย์"]
print(f"วันนี้เป็นวัน: {days[today.weekday()]}")
print(f"isoweekday (1=Mon): {today.isoweekday()}")

# ISO format: (year, week, weekday)
print(f"ISO calendar: {today.isocalendar()}")
```

### ตัวอย่าง 3: time Object

```python
from datetime import time

# สร้าง time
t1 = time(10, 30, 45)
print(t1)  # 10:30:45

t2 = time(14, 30, 45, 123456)  # ใส่ microsecond ด้วย
print(t2)  # 14:30:45.123456

t_midnight = time(0, 0, 0)
t_noon = time(12, 0, 0)

# Attributes
print(f"Hour:        {t2.hour}")        # 14
print(f"Minute:      {t2.minute}")      # 30
print(f"Second:      {t2.second}")      # 45
print(f"Microsecond: {t2.microsecond}") # 123456

# Comparison
print(f"t_noon > t_midnight: {t_noon > t_midnight}")  # True
print(f"t1 == t_noon: {t1 == t_noon}")              # False

# time กับ timezone
import datetime
utc = datetime.timezone.utc
t_utc = time(10, 30, tzinfo=utc)
print(f"UTC time: {t_utc}")  # 10:30:00+00:00
```

### ตัวอย่าง 4: datetime Object

```python
from datetime import datetime, date, time, timezone

# สร้าง datetime
dt1 = datetime(2024, 1, 15, 10, 30, 45)
print(dt1)  # 2024-01-15 10:30:45

# ปัจจุบัน
now = datetime.now()
print(f"Local now: {now}")

utc_now = datetime.now(timezone.utc)
print(f"UTC now: {utc_now}")

# now() กับ utcnow() (deprecated)
# ใช้ datetime.now(timezone.utc) แทน utcnow()

# สร้างจาก date และ time
d = date(2024, 6, 15)
t = time(14, 30, 0)
dt_combined = datetime.combine(d, t)
print(f"Combined: {dt_combined}")

# Attributes
print(f"Year:   {dt1.year}")
print(f"Month:  {dt1.month}")
print(f"Day:    {dt1.day}")
print(f"Hour:   {dt1.hour}")
print(f"Minute: {dt1.minute}")
print(f"Second: {dt1.second}")

# แปลงเป็น date หรือ time
print(f"Date part: {dt1.date()}")  # 2024-01-15
print(f"Time part: {dt1.time()}")  # 10:30:45

# สร้างจาก timestamp
import time as time_module
dt_from_ts = datetime.fromtimestamp(time_module.time())
print(f"From timestamp: {dt_from_ts}")

# replace() - สร้าง datetime ใหม่โดยแก้ไขบางส่วน
dt_modified = dt1.replace(year=2025, hour=12)
print(f"Modified: {dt_modified}")  # 2025-01-15 12:30:45
```

### ตัวอย่าง 5: timedelta Object

```python
from datetime import datetime, date, timedelta

# สร้าง timedelta
delta1 = timedelta(days=7)
delta2 = timedelta(hours=12, minutes=30)
delta3 = timedelta(weeks=2, days=3, hours=4, minutes=30, seconds=45)
delta4 = timedelta(milliseconds=500)
delta5 = timedelta(microseconds=1000000)  # = 1 วินาที

print(f"7 วัน: {delta1}")
print(f"12:30: {delta2}")
print(f"Complex: {delta3}")

# Attributes
print(f"delta3.days: {delta3.days}")                   # 17 (2*7+3)
print(f"delta3.seconds: {delta3.seconds}")             # วินาทีใน fraction of day
print(f"delta3.total_seconds(): {delta3.total_seconds()}")  # ทั้งหมดเป็นวินาที

# การคำนวณ datetime
today = date.today()
next_week = today + timedelta(days=7)
last_month = today - timedelta(days=30)
print(f"วันนี้: {today}")
print(f"สัปดาห์หน้า: {next_week}")
print(f"เดือนที่แล้ว: {last_month}")

# ความแตกต่างระหว่าง datetimes
dt1 = datetime(2024, 1, 1)
dt2 = datetime(2024, 12, 31, 23, 59, 59)
diff = dt2 - dt1
print(f"\nความแตกต่าง:")
print(f"  days: {diff.days}")
print(f"  total_seconds: {diff.total_seconds():,.0f}")
print(f"  hours: {diff.total_seconds() / 3600:.1f}")

# Arithmetic ด้วย timedelta
td = timedelta(days=10)
print(f"\nTimedelta arithmetic:")
print(f"  10 days + 5 days = {td + timedelta(days=5)}")
print(f"  10 days * 3 = {td * 3}")
print(f"  10 days / 2 = {td / 2}")
print(f"  10 days // 3 = {td // timedelta(days=3)}")  # จำนวนครั้ง
```

---

## Formatting: strftime และ strptime

### ตัวอย่าง 6: strftime - Format Codes ที่สำคัญ

```python
from datetime import datetime

now = datetime.now()

# Format codes
print("=== Format Codes ===")
print(f"%Y: {now.strftime('%Y')}")    # ปี 4 หลัก: 2024
print(f"%y: {now.strftime('%y')}")    # ปี 2 หลัก: 24
print(f"%m: {now.strftime('%m')}")    # เดือน 01-12
print(f"%d: {now.strftime('%d')}")    # วัน 01-31
print(f"%H: {now.strftime('%H')}")    # ชั่วโมง 00-23 (24h)
print(f"%I: {now.strftime('%I')}")    # ชั่วโมง 01-12 (12h)
print(f"%M: {now.strftime('%M')}")    # นาที 00-59
print(f"%S: {now.strftime('%S')}")    # วินาที 00-59
print(f"%f: {now.strftime('%f')}")    # microseconds 000000-999999
print(f"%p: {now.strftime('%p')}")    # AM/PM
print(f"%A: {now.strftime('%A')}")    # วันเต็ม: Monday
print(f"%a: {now.strftime('%a')}")    # วันย่อ: Mon
print(f"%B: {now.strftime('%B')}")    # เดือนเต็ม: January
print(f"%b: {now.strftime('%b')}")    # เดือนย่อ: Jan
print(f"%j: {now.strftime('%j')}")    # วันที่ในปี 001-366
print(f"%w: {now.strftime('%w')}")    # วันในสัปดาห์ 0=Sun
print(f"%W: {now.strftime('%W')}")    # สัปดาห์ในปี (Mon เริ่ม)
print(f"%Z: {now.strftime('%Z')}")    # timezone name

# รูปแบบที่นิยมใช้
print("\n=== Common Formats ===")
print(f"ISO: {now.strftime('%Y-%m-%d %H:%M:%S')}")
print(f"Date only: {now.strftime('%Y-%m-%d')}")
print(f"Time only: {now.strftime('%H:%M:%S')}")
print(f"Thai style: {now.strftime('%d/%m/%Y')}")
print(f"12h format: {now.strftime('%I:%M:%S %p')}")
print(f"Full: {now.strftime('%A, %B %d, %Y at %I:%M %p')}")
```

### ตัวอย่าง 7: strptime - Parse String เป็น datetime

```python
from datetime import datetime

# strptime(string, format)
date_strings = [
    ("2024-01-15", "%Y-%m-%d"),
    ("15/01/2024", "%d/%m/%Y"),
    ("January 15, 2024", "%B %d, %Y"),
    ("Mon Jan 15 10:30:45 2024", "%a %b %d %H:%M:%S %Y"),
    ("2024-01-15 10:30:45.123456", "%Y-%m-%d %H:%M:%S.%f"),
    ("15 Jan 2024 10:30 AM", "%d %b %Y %I:%M %p"),
]

print("strptime Examples:")
for date_str, fmt in date_strings:
    try:
        dt = datetime.strptime(date_str, fmt)
        print(f"  '{date_str}' → {dt}")
    except ValueError as e:
        print(f"  Error: {e}")

# ปัญหาที่พบบ่อย: Thai month names
# Python ใช้ English locale โดยปริยาย
# ต้อง map เองสำหรับภาษาไทย

thai_months = {
    'ม.ค.': 1, 'ก.พ.': 2, 'มี.ค.': 3, 'เม.ย.': 4,
    'พ.ค.': 5, 'มิ.ย.': 6, 'ก.ค.': 7, 'ส.ค.': 8,
    'ก.ย.': 9, 'ต.ค.': 10, 'พ.ย.': 11, 'ธ.ค.': 12
}

def parse_thai_date(date_str):
    """Parse วันที่ภาษาไทย: '15 ม.ค. 2567'"""
    import re
    match = re.match(r'(\d{1,2})\s+(\S+)\s+(\d{4})', date_str)
    if match:
        day = int(match.group(1))
        month_str = match.group(2)
        year = int(match.group(3))
        
        # แปลง Buddhist Era เป็น CE (ปี พ.ศ. - 543 = ค.ศ.)
        if year > 2500:
            year -= 543
        
        month = thai_months.get(month_str, 0)
        if month:
            return datetime(year, month, day)
    raise ValueError(f"ไม่สามารถ parse: {date_str}")

dt = parse_thai_date("15 ม.ค. 2567")
print(f"\nThai date: 15 ม.ค. 2567 → {dt}")
```

### ตัวอย่าง 8: isoformat และ fromisoformat

```python
from datetime import datetime, date, time, timezone

# isoformat() - มาตรฐาน ISO 8601
now = datetime.now()
dt_utc = datetime.now(timezone.utc)

print(f"isoformat: {now.isoformat()}")
# 2024-01-15T10:30:45.123456

print(f"isoformat sep=' ': {now.isoformat(sep=' ')}")
# 2024-01-15 10:30:45.123456

print(f"isoformat timespec='seconds': {now.isoformat(timespec='seconds')}")
# 2024-01-15T10:30:45

print(f"UTC isoformat: {dt_utc.isoformat()}")
# 2024-01-15T10:30:45.123456+00:00

# fromisoformat() - parse ISO 8601 (Python 3.7+)
iso_strings = [
    "2024-01-15",
    "2024-01-15T10:30:45",
    "2024-01-15T10:30:45.123456",
    "2024-01-15T10:30:45+07:00",
    "2024-01-15 10:30:45",  # space separator
]

for iso_str in iso_strings:
    try:
        dt = datetime.fromisoformat(iso_str)
        print(f"  '{iso_str}' → {dt}")
    except ValueError as e:
        print(f"  Error: {e}")
```

---

## Timezone-aware datetime

datetime สามารถเป็น **naive** (ไม่มี timezone info) หรือ **aware** (มี timezone info)

### ตัวอย่าง 9: Naive vs Aware datetime

```python
from datetime import datetime, timezone

# Naive datetime - ไม่มี timezone info
naive_dt = datetime.now()
print(f"Naive: {naive_dt}")
print(f"tzinfo: {naive_dt.tzinfo}")  # None

# Aware datetime - มี timezone info
aware_dt = datetime.now(timezone.utc)
print(f"Aware: {aware_dt}")
print(f"tzinfo: {aware_dt.tzinfo}")  # UTC

# ไม่สามารถเปรียบเทียบ naive กับ aware
try:
    diff = aware_dt - naive_dt
except TypeError as e:
    print(f"Error: {e}")  # can't subtract offset-naive...

# แปลง naive เป็น aware (กำหนด timezone)
import datetime as dt_module
local_tz = dt_module.timezone(dt_module.timedelta(hours=7))  # UTC+7 Thailand
naive_to_aware = naive_dt.replace(tzinfo=local_tz)
print(f"Made aware: {naive_to_aware}")

# UTC offset
utc_offset = aware_dt.utcoffset()
print(f"UTC offset: {utc_offset}")  # 0:00:00 สำหรับ UTC
```

---

## pytz และ zoneinfo

### ตัวอย่าง 10: zoneinfo (Python 3.9+)

```python
from datetime import datetime
from zoneinfo import ZoneInfo, available_timezones

# สร้าง aware datetime ด้วย zoneinfo
bangkok_tz = ZoneInfo("Asia/Bangkok")
tokyo_tz = ZoneInfo("Asia/Tokyo")
london_tz = ZoneInfo("Europe/London")
ny_tz = ZoneInfo("America/New_York")

# วันที่/เวลาปัจจุบันในแต่ละ timezone
utc_now = datetime.now(ZoneInfo("UTC"))
print(f"UTC:     {utc_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

bangkok_now = utc_now.astimezone(bangkok_tz)
print(f"Bangkok: {bangkok_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

tokyo_now = utc_now.astimezone(tokyo_tz)
print(f"Tokyo:   {tokyo_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

london_now = utc_now.astimezone(london_tz)
print(f"London:  {london_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

ny_now = utc_now.astimezone(ny_tz)
print(f"NY:      {ny_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

# สร้าง datetime ใน timezone ที่กำหนด
meeting_bkk = datetime(2024, 1, 15, 14, 0, 0, tzinfo=bangkok_tz)
print(f"\nการประชุมที่กรุงเทพ: {meeting_bkk}")
print(f"เวลา UTC: {meeting_bkk.astimezone(ZoneInfo('UTC'))}")

# Daylight Saving Time (DST)
# ลอนดอนมี DST ในช่วงฤดูร้อน (UTC+1) และฤดูหนาว (UTC+0)
summer = datetime(2024, 7, 15, 12, 0, 0, tzinfo=london_tz)
winter = datetime(2024, 1, 15, 12, 0, 0, tzinfo=london_tz)
print(f"\nLondon summer UTC offset: {summer.utcoffset()}")  # 1:00:00
print(f"London winter UTC offset: {winter.utcoffset()}")  # 0:00:00
```

### ตัวอย่าง 11: pytz (library ยอดนิยม)

```python
# ต้องติดตั้ง: pip install pytz
try:
    import pytz
    from datetime import datetime
    
    # สร้าง timezone
    bangkok = pytz.timezone("Asia/Bangkok")
    utc = pytz.UTC
    tokyo = pytz.timezone("Asia/Tokyo")
    
    # สร้าง aware datetime
    now_utc = datetime.now(utc)
    now_bkk = now_utc.astimezone(bangkok)
    now_tky = now_utc.astimezone(tokyo)
    
    print(f"UTC:     {now_utc.strftime('%Y-%m-%d %H:%M:%S %Z%z')}")
    print(f"Bangkok: {now_bkk.strftime('%Y-%m-%d %H:%M:%S %Z%z')}")
    print(f"Tokyo:   {now_tky.strftime('%Y-%m-%d %H:%M:%S %Z%z')}")
    
    # localize: กำหนด timezone ให้ naive datetime
    # ใช้ localize() แทน replace() เพื่อ handle DST ถูกต้อง
    naive = datetime(2024, 6, 15, 14, 30, 0)
    aware = bangkok.localize(naive)
    print(f"\nLocalized: {aware}")
    
    # ดู timezones ทั้งหมด
    print(f"\nจำนวน timezones: {len(pytz.all_timezones)}")
    print("Asian timezones:")
    asian = [tz for tz in pytz.all_timezones if tz.startswith("Asia/")][:10]
    for tz in asian:
        print(f"  {tz}")
    
except ImportError:
    print("pytz ไม่ได้ติดตั้ง ใช้ zoneinfo แทน (Python 3.9+)")
```

---

## UTC และ Timezone Conversions

### ตัวอย่าง 12: การแปลง Timezone

```python
from datetime import datetime
from zoneinfo import ZoneInfo

def convert_timezone(dt: datetime, target_tz: str) -> datetime:
    """แปลง datetime จาก timezone หนึ่งไปอีก timezone"""
    target = ZoneInfo(target_tz)
    if dt.tzinfo is None:
        # naive - assume UTC
        dt = dt.replace(tzinfo=ZoneInfo("UTC"))
    return dt.astimezone(target)

# สร้าง meeting เวลา 14:00 ที่กรุงเทพ
meeting_local = datetime(2024, 1, 15, 14, 0, 0, tzinfo=ZoneInfo("Asia/Bangkok"))

# ผู้เข้าร่วมจาก timezone ต่างๆ
attendees = {
    "Bangkok (host)": "Asia/Bangkok",
    "Tokyo": "Asia/Tokyo",
    "London": "Europe/London",
    "New York": "America/New_York",
    "Sydney": "Australia/Sydney",
    "Dubai": "Asia/Dubai",
}

print("Meeting Time Across Timezones:")
print(f"Meeting: {meeting_local.strftime('%Y-%m-%d %H:%M %Z')}")
print("-" * 45)
for city, tz in attendees.items():
    local_time = meeting_local.astimezone(ZoneInfo(tz))
    print(f"  {city:<20}: {local_time.strftime('%H:%M %Z (%z)')}")
```

### ตัวอย่าง 13: UTC Best Practices

```python
from datetime import datetime
from zoneinfo import ZoneInfo
import time

def utcnow() -> datetime:
    """ดึงเวลา UTC ปัจจุบัน (aware)"""
    return datetime.now(ZoneInfo("UTC"))

def to_utc(dt: datetime, local_tz: str = "Asia/Bangkok") -> datetime:
    """แปลง local time เป็น UTC"""
    if dt.tzinfo is None:
        dt = dt.replace(tzinfo=ZoneInfo(local_tz))
    return dt.astimezone(ZoneInfo("UTC"))

def from_utc(utc_dt: datetime, local_tz: str = "Asia/Bangkok") -> datetime:
    """แปลง UTC เป็น local time"""
    if utc_dt.tzinfo is None:
        utc_dt = utc_dt.replace(tzinfo=ZoneInfo("UTC"))
    return utc_dt.astimezone(ZoneInfo(local_tz))

def to_timestamp(dt: datetime) -> float:
    """แปลง datetime เป็น Unix timestamp"""
    if dt.tzinfo is None:
        dt = dt.replace(tzinfo=ZoneInfo("UTC"))
    return dt.timestamp()

def from_timestamp(ts: float, tz: str = "UTC") -> datetime:
    """แปลง Unix timestamp เป็น datetime"""
    return datetime.fromtimestamp(ts, tz=ZoneInfo(tz))

# ตัวอย่าง workflow ที่ถูกต้อง
print("=== UTC Best Practices ===")

# เก็บใน database เป็น UTC เสมอ
stored_utc = utcnow()
print(f"Stored UTC: {stored_utc.isoformat()}")

# แสดงผลให้ user ด้วย local timezone
displayed = from_utc(stored_utc, "Asia/Bangkok")
print(f"Displayed (Bangkok): {displayed.strftime('%d/%m/%Y %H:%M:%S %Z')}")

# รับ input จาก user เป็น local time, แปลงเป็น UTC ก่อนเก็บ
user_input = datetime(2024, 1, 15, 14, 30, 0)  # naive - Bangkok time
saved = to_utc(user_input, "Asia/Bangkok")
print(f"User input: {user_input} (Bangkok) → Saved: {saved} (UTC)")

# timestamp conversion
ts = to_timestamp(stored_utc)
recovered = from_timestamp(ts, "Asia/Bangkok")
print(f"Timestamp: {ts:.0f} → {recovered.strftime('%Y-%m-%d %H:%M:%S %Z')}")
```

---

## dateutil Library

`dateutil` เป็น library ที่ขยาย datetime module ให้มีความสามารถมากขึ้น

### ตัวอย่าง 14: dateutil.parser

```python
try:
    from dateutil import parser as dateutil_parser
    from datetime import datetime
    
    # parse() อ่านได้หลากหลาย format โดยไม่ต้องระบุ format
    date_strings = [
        "January 15, 2024",
        "Jan 15 2024",
        "15 Jan 2024",
        "2024-01-15",
        "01/15/2024",
        "15/01/2024 10:30",
        "Mon, 15 Jan 2024 10:30:45 GMT",
        "2024-01-15T10:30:45+07:00",
        "15 January 2024 10:30 PM",
    ]
    
    print("dateutil.parser.parse():")
    for ds in date_strings:
        try:
            dt = dateutil_parser.parse(ds)
            print(f"  '{ds}' → {dt}")
        except Exception as e:
            print(f"  Error: {ds}: {e}")

except ImportError:
    print("dateutil ไม่ได้ติดตั้ง: pip install python-dateutil")
    
    # ใช้ fromisoformat แทน
    from datetime import datetime
    print("ใช้ datetime.fromisoformat แทน:")
    dt = datetime.fromisoformat("2024-01-15T10:30:45")
    print(f"  {dt}")
```

### ตัวอย่าง 15: dateutil.relativedelta

```python
try:
    from dateutil.relativedelta import relativedelta
    from datetime import datetime, date
    
    today = date.today()
    
    # relativedelta เข้าใจ "เดือน" และ "ปี" อย่างถูกต้อง
    # timedelta ไม่รองรับ (เพราะเดือนมีจำนวนวันไม่แน่นอน)
    
    # เพิ่ม 1 เดือน
    next_month = today + relativedelta(months=1)
    print(f"วันนี้: {today}")
    print(f"เดือนหน้า: {next_month}")
    
    # เพิ่ม 1 ปี 2 เดือน 3 วัน
    future = today + relativedelta(years=1, months=2, days=3)
    print(f"อนาคต: {future}")
    
    # ลบ 6 เดือน
    past = today - relativedelta(months=6)
    print(f"6 เดือนก่อน: {past}")
    
    # คำนวณอายุ
    birthday = date(1990, 6, 15)
    age_delta = relativedelta(today, birthday)
    print(f"\nวันเกิด: {birthday}")
    print(f"อายุ: {age_delta.years} ปี {age_delta.months} เดือน {age_delta.days} วัน")
    
    # วันสุดท้ายของเดือน
    first_of_month = today.replace(day=1)
    last_of_month = first_of_month + relativedelta(months=1, days=-1)
    print(f"\nวันแรกของเดือน: {first_of_month}")
    print(f"วันสุดท้ายของเดือน: {last_of_month}")

except ImportError:
    print("python-dateutil ไม่ได้ติดตั้ง: pip install python-dateutil")
```

---

## time Module

### ตัวอย่าง 16: time Module พื้นฐาน

```python
import time

# time() - Unix timestamp (วินาทีตั้งแต่ 1 Jan 1970 UTC)
ts = time.time()
print(f"timestamp: {ts}")
print(f"timestamp: {ts:.3f}")  # มี milliseconds

# time_ns() - nanoseconds timestamp (Python 3.7+)
ts_ns = time.time_ns()
print(f"timestamp ns: {ts_ns}")

# sleep() - หยุดชั่วคราว
print("รอ 0.1 วินาที...")
time.sleep(0.1)
print("ต่อ!")

# perf_counter() - ความแม่นยำสูงสุดสำหรับ timing
start = time.perf_counter()
sum(range(1000000))
elapsed = time.perf_counter() - start
print(f"sum(range(1000000)): {elapsed*1000:.3f}ms")

# perf_counter_ns() - nanoseconds
start_ns = time.perf_counter_ns()
[x**2 for x in range(10000)]
elapsed_ns = time.perf_counter_ns() - start_ns
print(f"list comp: {elapsed_ns/1000:.1f}μs")

# process_time() - CPU time ของ process นี้ (ไม่รวม sleep)
start = time.process_time()
time.sleep(1)  # sleep ไม่นับ
x = sum(range(10000000))  # CPU ทำงาน
cpu_time = time.process_time() - start
print(f"CPU time (ไม่นับ sleep): {cpu_time:.3f}s")

# gmtime() และ localtime() - แปลง timestamp เป็น struct_time
gmt = time.gmtime()
local = time.localtime()

print(f"\nGMT: {gmt.tm_year}-{gmt.tm_mon:02d}-{gmt.tm_mday:02d}")
print(f"Local: {local.tm_year}-{local.tm_mon:02d}-{local.tm_mday:02d}")

# strftime กับ struct_time
formatted = time.strftime("%Y-%m-%d %H:%M:%S", local)
print(f"Formatted: {formatted}")

# mktime() - struct_time เป็น timestamp
import time
t = time.struct_time((2024, 1, 15, 10, 30, 45, 0, 0, -1))
ts = time.mktime(t)
print(f"mktime: {ts}")
```

### ตัวอย่าง 17: Monotonic Clock

```python
import time

# monotonic() - ไม่ถอยหลัง (ดีกว่า time() สำหรับ measuring)
# ไม่ได้รับผลจากการปรับเวลาระบบ
start = time.monotonic()
time.sleep(0.05)
elapsed = time.monotonic() - start
print(f"monotonic elapsed: {elapsed:.4f}s")

# monotonic_ns() - nanosecond precision
start_ns = time.monotonic_ns()
for _ in range(100000):
    pass
elapsed_ns = time.monotonic_ns() - start_ns
print(f"monotonic_ns: {elapsed_ns:,} ns = {elapsed_ns/1e6:.3f}ms")

# ตัวอย่าง: benchmark helper
class Benchmark:
    def __init__(self):
        self.results = {}
    
    def run(self, name, func, *args, repeat=5, **kwargs):
        times = []
        for _ in range(repeat):
            start = time.perf_counter_ns()
            func(*args, **kwargs)
            times.append(time.perf_counter_ns() - start)
        
        self.results[name] = {
            'mean_ns': sum(times) / len(times),
            'min_ns': min(times),
            'max_ns': max(times),
            'times': times
        }
    
    def report(self):
        print("Benchmark Results:")
        for name, stats in self.results.items():
            print(f"  {name}:")
            print(f"    Mean: {stats['mean_ns']/1e6:.3f}ms")
            print(f"    Min:  {stats['min_ns']/1e6:.3f}ms")
            print(f"    Max:  {stats['max_ns']/1e6:.3f}ms")

bm = Benchmark()
bm.run("list comp", lambda: [x**2 for x in range(10000)])
bm.run("map", lambda: list(map(lambda x: x**2, range(10000))))
bm.run("generator", lambda: list(x**2 for x in range(10000)))
bm.report()
```

---

## calendar Module

### ตัวอย่าง 18: calendar Module

```python
import calendar
from datetime import date

# ตรวจสอบปีอธิกสุรทิน
print("ปีอธิกสุรทิน:")
for year in range(2020, 2032):
    if calendar.isleap(year):
        print(f"  {year} (วัน leap year)")

print(f"\nจำนวนปีอธิกสุรทินระหว่าง 2000-2100: {calendar.leapdays(2000, 2100)}")

# จำนวนวันในเดือน
print("\nจำนวนวันในเดือน 2024:")
for month in range(1, 13):
    _, days = calendar.monthrange(2024, month)
    month_name = calendar.month_name[month]
    print(f"  {month_name:<12}: {days} วัน")

# monthcalendar() - ปฏิทินของเดือน
print("\nปฏิทินมกราคม 2024:")
print("Mon Tue Wed Thu Fri Sat Sun")
for week in calendar.monthcalendar(2024, 1):
    row = ""
    for day in week:
        if day == 0:
            row += "    "
        else:
            row += f"{day:3d} "
    print(row)

# ดูว่าวันที่เท่าไหร่ตกวันอะไร
year, month, day = 2024, 1, 15
weekday = calendar.weekday(year, month, day)
print(f"\n{year}-{month:02d}-{day:02d} เป็นวัน: {calendar.day_name[weekday]}")

# TextCalendar และ HTMLCalendar
tc = calendar.TextCalendar(calendar.MONDAY)  # เริ่มต้นสัปดาห์ที่วันจันทร์
print("\nText Calendar (January 2024):")
print(tc.formatmonth(2024, 1))
```

### ตัวอย่าง 19: calendar.itermonthdays2

```python
import calendar
from datetime import date

def get_working_days(year, month):
    """นับวันทำการในเดือน (วันจันทร์-ศุกร์)"""
    cal = calendar.monthcalendar(year, month)
    working_days = 0
    
    for week in cal:
        for i, day in enumerate(week):
            if day != 0 and i < 5:  # i=0..4 คือ จันทร์-ศุกร์
                working_days += 1
    
    return working_days

print("วันทำการในแต่ละเดือน ปี 2024:")
total_working = 0
for month in range(1, 13):
    days = get_working_days(2024, month)
    total_working += days
    month_name = calendar.month_name[month]
    print(f"  {month_name:<12}: {days} วัน")
print(f"รวมทั้งปี: {total_working} วัน")

# หาวันศุกร์ทั้งหมดในเดือน
def get_all_fridays(year, month):
    """ดึงวันศุกร์ทั้งหมดในเดือน"""
    fridays = []
    for day_num, weekday in calendar.itermonthdays2(year, month):
        if day_num != 0 and weekday == calendar.FRIDAY:
            fridays.append(date(year, month, day_num))
    return fridays

print("\nวันศุกร์ในเดือนมกราคม 2024:")
for friday in get_all_fridays(2024, 1):
    print(f"  {friday.strftime('%A, %B %d, %Y')}")
```

---

## Practical Patterns

### ตัวอย่าง 20: Age Calculator

```python
from datetime import date, datetime

def calculate_age(birthdate, reference_date=None):
    """คำนวณอายุอย่างแม่นยำ"""
    if reference_date is None:
        reference_date = date.today()
    
    if isinstance(birthdate, datetime):
        birthdate = birthdate.date()
    if isinstance(reference_date, datetime):
        reference_date = reference_date.date()
    
    # คำนวณอายุ
    years = reference_date.year - birthdate.year
    months = reference_date.month - birthdate.month
    days = reference_date.day - birthdate.day
    
    # ปรับ
    if days < 0:
        months -= 1
        # วันในเดือนที่แล้ว
        prev_month = reference_date.replace(day=1) - __import__('datetime').timedelta(days=1)
        days += prev_month.day
    
    if months < 0:
        years -= 1
        months += 12
    
    return years, months, days

def age_in_words(birthdate, reference_date=None):
    """แสดงอายุเป็นคำ"""
    years, months, days = calculate_age(birthdate, reference_date)
    
    parts = []
    if years > 0:
        parts.append(f"{years} ปี")
    if months > 0:
        parts.append(f"{months} เดือน")
    if days > 0 or not parts:
        parts.append(f"{days} วัน")
    
    return " ".join(parts)

def next_birthday(birthdate, reference_date=None):
    """หาวันเกิดถัดไป"""
    if reference_date is None:
        reference_date = date.today()
    
    next_bd = birthdate.replace(year=reference_date.year)
    if next_bd <= reference_date:
        next_bd = birthdate.replace(year=reference_date.year + 1)
    
    days_until = (next_bd - reference_date).days
    return next_bd, days_until

# ทดสอบ
birthdates = [
    date(1990, 6, 15),
    date(1985, 12, 25),
    date(2000, 2, 29),  # Leap year birthday
]

ref = date(2024, 6, 15)
print(f"Reference date: {ref}\n")

for bd in birthdates:
    years, months, days = calculate_age(bd, ref)
    next_bd, days_until = next_birthday(bd, ref)
    
    print(f"วันเกิด: {bd}")
    print(f"  อายุ: {age_in_words(bd, ref)}")
    print(f"  วันเกิดถัดไป: {next_bd} (อีก {days_until} วัน)")
    print()
```

### ตัวอย่าง 21: Business Days Calculator

```python
from datetime import date, timedelta
import calendar

# วันหยุดราชการไทย 2024 (ตัวอย่างบางส่วน)
THAI_PUBLIC_HOLIDAYS_2024 = {
    date(2024, 1, 1),   # วันปีใหม่
    date(2024, 2, 24),  # วันมาฆบูชา
    date(2024, 4, 6),   # วันจักรี
    date(2024, 4, 12),  # วันหยุดพิเศษ
    date(2024, 4, 13),  # วันสงกรานต์
    date(2024, 4, 14),  # วันสงกรานต์
    date(2024, 4, 15),  # วันสงกรานต์
    date(2024, 5, 1),   # วันแรงงาน
    date(2024, 5, 4),   # วันฉัตรมงคล
    date(2024, 5, 22),  # วันวิสาขบูชา
    date(2024, 6, 3),   # วันเฉลิมพระชนมพรรษาฯ (ราชินี)
    date(2024, 7, 20),  # วันอาสาฬหบูชา
    date(2024, 7, 21),  # วันเข้าพรรษา
    date(2024, 7, 22),  # วันหยุดพิเศษ
    date(2024, 7, 28),  # วันเฉลิมพระชนมพรรษาฯ (ร.10)
    date(2024, 8, 12),  # วันแม่แห่งชาติ
    date(2024, 10, 13), # วันนวมินทรมหาราช
    date(2024, 10, 23), # วันปิยมหาราช
    date(2024, 12, 5),  # วันพ่อแห่งชาติ
    date(2024, 12, 10), # วันรัฐธรรมนูญ
    date(2024, 12, 31), # วันสิ้นปี
}

def is_working_day(d, holidays=None):
    """ตรวจสอบว่าเป็นวันทำการหรือไม่"""
    if holidays is None:
        holidays = set()
    
    # ไม่ใช่เสาร์-อาทิตย์
    if d.weekday() >= 5:  # 5=Saturday, 6=Sunday
        return False
    
    # ไม่ใช่วันหยุด
    if d in holidays:
        return False
    
    return True

def add_business_days(start_date, num_days, holidays=None):
    """เพิ่มวันทำการ"""
    if holidays is None:
        holidays = set()
    
    current = start_date
    days_added = 0
    direction = 1 if num_days >= 0 else -1
    
    while days_added < abs(num_days):
        current += timedelta(days=direction)
        if is_working_day(current, holidays):
            days_added += 1
    
    return current

def count_business_days(start_date, end_date, holidays=None):
    """นับวันทำการระหว่างสองวันที่"""
    if holidays is None:
        holidays = set()
    
    if start_date > end_date:
        start_date, end_date = end_date, start_date
    
    count = 0
    current = start_date
    while current <= end_date:
        if is_working_day(current, holidays):
            count += 1
        current += timedelta(days=1)
    
    return count

def get_next_working_day(d, holidays=None):
    """หาวันทำการถัดไป"""
    if holidays is None:
        holidays = set()
    
    next_day = d + timedelta(days=1)
    while not is_working_day(next_day, holidays):
        next_day += timedelta(days=1)
    return next_day

# ทดสอบ
today = date(2024, 1, 15)  # วันจันทร์

print(f"วันนี้: {today} ({calendar.day_name[today.weekday()]})")
print(f"วันทำการ: {'ใช่' if is_working_day(today, THAI_PUBLIC_HOLIDAYS_2024) else 'ไม่ใช่'}")

# เพิ่ม 10 วันทำการ
deadline = add_business_days(today, 10, THAI_PUBLIC_HOLIDAYS_2024)
print(f"\nครบกำหนด (10 วันทำการ): {deadline}")

# นับวันทำการระหว่างสองวันที่
start = date(2024, 4, 1)
end = date(2024, 4, 30)
work_days = count_business_days(start, end, THAI_PUBLIC_HOLIDAYS_2024)
print(f"\nวันทำการเดือนเมษายน: {work_days} วัน")

# วันทำการถัดไปหลังวันหยุด
holiday = date(2024, 4, 13)  # สงกรานต์
next_work = get_next_working_day(holiday, THAI_PUBLIC_HOLIDAYS_2024)
print(f"\nวันทำการถัดไปหลัง {holiday}: {next_work}")
```

### ตัวอย่าง 22: Schedule System

```python
from datetime import datetime, date, time, timedelta
from zoneinfo import ZoneInfo
from typing import List, Optional
from dataclasses import dataclass, field
import uuid

@dataclass
class TimeSlot:
    start: datetime
    end: datetime
    
    @property
    def duration(self) -> timedelta:
        return self.end - self.start
    
    def overlaps_with(self, other: 'TimeSlot') -> bool:
        return (self.start < other.end and self.end > other.start)
    
    def __str__(self):
        return (f"{self.start.strftime('%H:%M')} - "
                f"{self.end.strftime('%H:%M')} "
                f"({int(self.duration.total_seconds()/60)} นาที)")

@dataclass
class Event:
    title: str
    slot: TimeSlot
    description: str = ""
    attendees: List[str] = field(default_factory=list)
    id: str = field(default_factory=lambda: str(uuid.uuid4())[:8])
    
    def __str__(self):
        return f"[{self.id}] {self.title} @ {self.slot}"

class Calendar:
    """Simple calendar system"""
    
    def __init__(self, timezone: str = "Asia/Bangkok"):
        self.tz = ZoneInfo(timezone)
        self.events: List[Event] = []
    
    def add_event(self, title: str, start: datetime, duration_minutes: int,
                  **kwargs) -> Event:
        """เพิ่ม event"""
        if start.tzinfo is None:
            start = start.replace(tzinfo=self.tz)
        
        end = start + timedelta(minutes=duration_minutes)
        slot = TimeSlot(start, end)
        
        # ตรวจสอบ conflict
        conflicts = self.get_conflicts(slot)
        if conflicts:
            raise ValueError(
                f"มี event ซ้อนทับ:\n" + 
                "\n".join(f"  - {e}" for e in conflicts)
            )
        
        event = Event(title=title, slot=slot, **kwargs)
        self.events.append(event)
        return event
    
    def get_conflicts(self, slot: TimeSlot) -> List[Event]:
        """หา events ที่ซ้อนทับกัน"""
        return [e for e in self.events if e.slot.overlaps_with(slot)]
    
    def get_events_for_date(self, d: date) -> List[Event]:
        """ดู events ในวันที่กำหนด"""
        return [
            e for e in self.events 
            if e.slot.start.date() == d
        ]
    
    def find_free_slots(self, d: date, 
                        duration_minutes: int,
                        work_start: time = time(9, 0),
                        work_end: time = time(18, 0)) -> List[TimeSlot]:
        """หาช่วงเวลาว่างในวันที่กำหนด"""
        day_start = datetime.combine(d, work_start, tzinfo=self.tz)
        day_end = datetime.combine(d, work_end, tzinfo=self.tz)
        
        # รวบรวม busy slots
        busy_slots = [
            e.slot for e in self.get_events_for_date(d)
        ]
        busy_slots.sort(key=lambda s: s.start)
        
        free_slots = []
        current = day_start
        
        for slot in busy_slots:
            if current < slot.start:
                potential = TimeSlot(current, slot.start)
                if potential.duration >= timedelta(minutes=duration_minutes):
                    free_slots.append(potential)
            current = max(current, slot.end)
        
        # ช่วงสุดท้าย
        if current < day_end:
            potential = TimeSlot(current, day_end)
            if potential.duration >= timedelta(minutes=duration_minutes):
                free_slots.append(potential)
        
        return free_slots

# ทดสอบ
cal = Calendar("Asia/Bangkok")
ref_date = date(2024, 1, 15)

# เพิ่ม events
morning_meeting = cal.add_event(
    "Morning Standup",
    datetime(2024, 1, 15, 9, 0),
    duration_minutes=30,
    attendees=["สมชาย", "สมหญิง", "วิชัย"]
)

lunch = cal.add_event(
    "Lunch Break",
    datetime(2024, 1, 15, 12, 0),
    duration_minutes=60
)

review = cal.add_event(
    "Code Review",
    datetime(2024, 1, 15, 14, 0),
    duration_minutes=90,
    attendees=["สมชาย", "วิชัย"]
)

# ดู events วันนี้
print(f"Events วันที่ {ref_date}:")
for event in cal.get_events_for_date(ref_date):
    print(f"  {event}")

# หาช่วงเวลาว่างสำหรับ meeting 60 นาที
print(f"\nช่วงเวลาว่าง (60 นาที):")
free = cal.find_free_slots(ref_date, duration_minutes=60)
for slot in free:
    print(f"  {slot}")

# ทดสอบ conflict
print("\nทดสอบ conflict:")
try:
    cal.add_event("Conflict Event", datetime(2024, 1, 15, 9, 15), 30)
except ValueError as e:
    print(f"  {e}")
```

---

## ISO 8601 Format

### ตัวอย่าง 23: ISO 8601 ครบถ้วน

```python
from datetime import datetime, date, time, timezone, timedelta
from zoneinfo import ZoneInfo

# ISO 8601 formats
now = datetime.now(ZoneInfo("UTC"))
bkk_now = now.astimezone(ZoneInfo("Asia/Bangkok"))

print("=== ISO 8601 Formats ===")

# Date only
print(f"Date: {now.date().isoformat()}")  # 2024-01-15

# Datetime (UTC)
print(f"UTC datetime: {now.isoformat()}")  # 2024-01-15T10:30:45.123456+00:00

# Datetime with timezone
print(f"Bangkok: {bkk_now.isoformat()}")  # 2024-01-15T17:30:45.123456+07:00

# Truncated versions
print(f"Seconds only: {now.isoformat(timespec='seconds')}")
print(f"Minutes only: {now.isoformat(timespec='minutes')}")

# Week format
d = date(2024, 1, 15)
iso_cal = d.isocalendar()
print(f"Week format: {d.year:04d}-W{iso_cal[1]:02d}-{iso_cal[2]}")  # 2024-W03-1

# Duration format: P[n]Y[n]M[n]DT[n]H[n]M[n]S
def format_iso_duration(td: timedelta) -> str:
    """แปลง timedelta เป็น ISO 8601 duration"""
    total_seconds = int(td.total_seconds())
    days, remainder = divmod(abs(total_seconds), 86400)
    hours, remainder = divmod(remainder, 3600)
    minutes, seconds = divmod(remainder, 60)
    
    sign = "-" if total_seconds < 0 else ""
    
    parts = [f"{sign}P"]
    years, days = divmod(days, 365)
    months, days = divmod(days, 30)
    
    if years:
        parts.append(f"{years}Y")
    if months:
        parts.append(f"{months}M")
    if days:
        parts.append(f"{days}D")
    
    time_parts = []
    if hours:
        time_parts.append(f"{hours}H")
    if minutes:
        time_parts.append(f"{minutes}M")
    if seconds:
        time_parts.append(f"{seconds}S")
    
    if time_parts:
        parts.append("T")
        parts.extend(time_parts)
    
    return "".join(parts)

durations = [
    timedelta(days=365),
    timedelta(days=7, hours=3, minutes=30),
    timedelta(hours=2, minutes=30, seconds=45),
    timedelta(minutes=90),
]

print("\nISO Duration formats:")
for td in durations:
    print(f"  {td} → {format_iso_duration(td)}")
```

### ตัวอย่าง 24: Timestamp Formats

```python
from datetime import datetime
from zoneinfo import ZoneInfo
import time

def datetime_to_formats(dt: datetime):
    """แสดง datetime ในหลาย formats"""
    print(f"Datetime object: {dt}")
    print(f"ISO 8601: {dt.isoformat()}")
    print(f"Unix timestamp: {dt.timestamp():.3f}")
    print(f"Epoch ms: {int(dt.timestamp() * 1000)}")
    print(f"RFC 2822: {dt.strftime('%a, %d %b %Y %H:%M:%S %z')}")
    print(f"Human: {dt.strftime('%B %d, %Y at %I:%M %p %Z')}")
    print(f"Short: {dt.strftime('%d/%m/%Y %H:%M')}")
    print()

now_utc = datetime.now(ZoneInfo("UTC"))
now_bkk = now_utc.astimezone(ZoneInfo("Asia/Bangkok"))

print("=== UTC ===")
datetime_to_formats(now_utc)

print("=== Bangkok ===")
datetime_to_formats(now_bkk)

# รับ timestamp จาก JavaScript (milliseconds)
js_timestamp = 1705312245000  # milliseconds
dt_from_js = datetime.fromtimestamp(js_timestamp / 1000, tz=ZoneInfo("UTC"))
print(f"From JS timestamp {js_timestamp}: {dt_from_js.isoformat()}")
```

---

## แบบฝึกหัด

### ข้อ 1: คำนวณอายุ (ละเอียด)

**คำตอบ:**

```python
from datetime import date

def detailed_age(birthdate: date, reference: date = None) -> dict:
    """คำนวณอายุอย่างละเอียด"""
    if reference is None:
        reference = date.today()
    
    total_days = (reference - birthdate).days
    
    # คำนวณ years, months, days
    years = reference.year - birthdate.year
    
    try:
        birthday_this_year = birthdate.replace(year=reference.year)
    except ValueError:  # Feb 29 ในปีที่ไม่ใช่ leap year
        birthday_this_year = birthdate.replace(year=reference.year, day=28)
    
    if reference < birthday_this_year:
        years -= 1
        try:
            birthday_this_year = birthdate.replace(year=reference.year - 1)
        except ValueError:
            birthday_this_year = birthdate.replace(year=reference.year - 1, day=28)
    
    months = reference.month - birthday_this_year.month
    if reference.day < birthday_this_year.day:
        months -= 1
    if months < 0:
        months += 12
    
    days = reference.day - birthday_this_year.day
    if days < 0:
        import calendar
        prev_month_days = calendar.monthrange(reference.year, reference.month - 1 or 12)[1]
        days += prev_month_days
    
    return {
        'years': years,
        'months': months,
        'days': days,
        'total_days': total_days,
        'total_weeks': total_days // 7,
        'is_birthday_today': reference.month == birthdate.month and reference.day == birthdate.day
    }

birthday = date(1990, 6, 15)
result = detailed_age(birthday, date(2024, 6, 15))
print(f"อายุ: {result['years']} ปี {result['months']} เดือน {result['days']} วัน")
print(f"รวม: {result['total_days']:,} วัน ({result['total_weeks']:,} สัปดาห์)")
print(f"วันเกิด: {'ใช่!' if result['is_birthday_today'] else 'ไม่ใช่'}")
```

### ข้อ 2: หาวันศุกร์ที่ 13 ในปี

**คำตอบ:**

```python
from datetime import date, timedelta
import calendar

def find_friday_13th(year):
    """หาวันศุกร์ที่ 13 ในปีที่กำหนด"""
    fridays_13 = []
    for month in range(1, 13):
        d = date(year, month, 13)
        if d.weekday() == 4:  # 4 = Friday
            fridays_13.append(d)
    return fridays_13

print("วันศุกร์ที่ 13 ปี 2020-2030:")
for year in range(2020, 2031):
    f13 = find_friday_13th(year)
    if f13:
        dates = ", ".join(str(d) for d in f13)
        print(f"  {year}: {dates}")
    else:
        print(f"  {year}: ไม่มี")
```

### ข้อ 3: Convert Unix Timestamp ไป-มา

**คำตอบ:**

```python
from datetime import datetime
from zoneinfo import ZoneInfo

def unix_to_datetime(timestamp: float, tz: str = "UTC") -> datetime:
    """Unix timestamp → aware datetime"""
    return datetime.fromtimestamp(timestamp, tz=ZoneInfo(tz))

def datetime_to_unix(dt: datetime) -> float:
    """aware datetime → Unix timestamp"""
    if dt.tzinfo is None:
        raise ValueError("datetime ต้องเป็น timezone-aware")
    return dt.timestamp()

# ทดสอบ
ts = 1705312245.0
dt = unix_to_datetime(ts, "Asia/Bangkok")
print(f"Timestamp {ts} → {dt}")
print(f"Back to timestamp: {datetime_to_unix(dt)}")
```

### ข้อ 4-10: แบบฝึกหัดเพิ่มเติม

**ข้อ 4**: สร้าง function `humanize_timedelta()` ที่แสดง timedelta เป็นภาษาธรรมชาติ ("2 ชั่วโมงที่แล้ว", "3 วันข้างหน้า")

**ข้อ 5**: สร้าง `DateRange` class ที่ iterable ผ่านช่วงวันที่

**ข้อ 6**: สร้าง function ที่แปลงปีพุทธศักราช (พ.ศ.) ↔ คริสต์ศักราช (ค.ศ.) อย่างถูกต้อง

**ข้อ 7**: สร้าง function นับวันหยุดสุดสัปดาห์ยาว (Long Weekend) ในปี

**ข้อ 8**: สร้าง recurring event generator ที่สร้าง events ตาม schedule (daily, weekly, monthly)

**ข้อ 9**: สร้าง timezone-aware cron expression parser เล็กๆ

**ข้อ 10**: สร้าง function วิเคราะห์ timestamp series หาค่าเฉลี่ย interval ระหว่าง events

---

## สรุป

| Class/Module | ใช้สำหรับ |
|-------------|----------|
| `date` | วันที่เท่านั้น |
| `time` | เวลาเท่านั้น |
| `datetime` | วันที่ + เวลา |
| `timedelta` | ช่วงเวลา / duration |
| `timezone` | timezone แบบ fixed offset |
| `zoneinfo.ZoneInfo` | timezone ตาม IANA database |
| `time` module | system time, sleep, timing |
| `calendar` | ปฏิทิน, วันในสัปดาห์ |
| `dateutil` | parse flexible formats, relativedelta |

**Best Practices:**
1. เก็บ datetime ใน database เป็น **UTC เสมอ**
2. ใช้ **aware datetime** (timezone-aware) ไม่ใช่ naive
3. ใช้ `datetime.now(timezone.utc)` ไม่ใช่ `datetime.utcnow()`
4. ใช้ `zoneinfo` (Python 3.9+) หรือ `pytz` สำหรับ IANA timezones
5. ใช้ `time.perf_counter()` สำหรับ timing ไม่ใช่ `time.time()`
6. ใช้ ISO 8601 format สำหรับ serialization และ interchange
7. ระวัง DST transitions เมื่อทำงานกับ local times
