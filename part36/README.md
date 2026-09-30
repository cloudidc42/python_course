# Part 36: Logging & Debugging

## บทนำ

ในการพัฒนาซอฟต์แวร์จริง การ logging และ debugging เป็นทักษะที่สำคัญมาก การ log ที่ดีช่วยให้เราติดตามการทำงานของโปรแกรม วินิจฉัยปัญหา และตรวจสอบ performance ได้ ส่วน debugging ช่วยให้เราค้นหาและแก้ไข bugs ได้อย่างมีประสิทธิภาพ

---

## 1. Logging Module

Python มี `logging` module ในตัวที่ทรงพลังและยืดหยุ่น ต่างจากการใช้ `print()` ธรรมดา logging ให้ข้อมูลที่ละเอียดกว่า เช่น timestamp, log level, module name และสามารถกำหนดให้บันทึกไปยังหลายที่พร้อมกันได้

### ทำไมถึงต้องใช้ logging แทน print()?

| Feature | print() | logging |
|---------|---------|---------|
| Severity levels | ไม่มี | มี (DEBUG, INFO, WARNING, ERROR, CRITICAL) |
| Timestamps | ไม่มี | มี |
| Module/function info | ไม่มี | มี |
| Output destination | stdout เท่านั้น | หลายที่พร้อมกัน |
| Enable/disable | ต้องลบ code | เปิด/ปิดได้ง่าย |
| Production use | ไม่เหมาะ | เหมาะมาก |

### ตัวอย่างที่ 1: การใช้ logging เบื้องต้น

```python
import logging

# การตั้งค่าพื้นฐาน - กำหนด format และ level
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# สร้าง logger สำหรับ module นี้
logger = logging.getLogger(__name__)

# ใช้งาน logger ในระดับต่างๆ
logger.debug("นี่คือ debug message - ใช้สำหรับข้อมูลละเอียดมาก")
logger.info("โปรแกรมเริ่มทำงานแล้ว")
logger.warning("ระวัง! disk space เหลือน้อยแล้ว")
logger.error("เกิดข้อผิดพลาด: ไม่สามารถเชื่อมต่อ database")
logger.critical("ระบบล้มเหลว! ต้องหยุดทำงานทันที")
```

---

## 2. Log Levels

Log levels ช่วยจัดระดับความสำคัญของ messages ทำให้เราสามารถกรองเฉพาะข้อมูลที่ต้องการได้

### ระดับ Log Levels (จากน้อยสุดถึงมากสุด)

| Level | ค่า | ใช้เมื่อ |
|-------|-----|---------|
| DEBUG | 10 | ข้อมูล debugging ละเอียด |
| INFO | 20 | ยืนยันว่าทำงานตามปกติ |
| WARNING | 30 | มีบางอย่างผิดปกติแต่ยังทำงานต่อได้ |
| ERROR | 40 | เกิดข้อผิดพลาด ฟังก์ชันบางส่วนทำงานไม่ได้ |
| CRITICAL | 50 | ข้อผิดพลาดร้ายแรง โปรแกรมอาจหยุดทำงาน |

### ตัวอย่างที่ 2: การทำงานของ Log Levels

```python
import logging

# เมื่อตั้ง level เป็น WARNING จะแสดงเฉพาะ WARNING ขึ้นไป
logging.basicConfig(level=logging.WARNING)
logger = logging.getLogger(__name__)

logger.debug("ข้อความนี้จะไม่แสดง")   # DEBUG < WARNING
logger.info("ข้อความนี้จะไม่แสดง")    # INFO < WARNING
logger.warning("ข้อความนี้แสดง!")     # WARNING = WARNING ✓
logger.error("ข้อความนี้แสดง!")       # ERROR > WARNING ✓
logger.critical("ข้อความนี้แสดง!")    # CRITICAL > WARNING ✓
```

### ตัวอย่างที่ 3: การตรวจสอบ Level ก่อน Log

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# การตรวจสอบ level ก่อน log ช่วยประหยัด performance
# เมื่อต้องสร้าง string ที่ซับซ้อน
if logger.isEnabledFor(logging.DEBUG):
    # code นี้จะรันเฉพาะเมื่อ DEBUG เปิดอยู่
    expensive_data = {"user": "john", "items": list(range(1000))}
    logger.debug("ข้อมูล: %s", expensive_data)

# วิธีที่ดีกว่าคือใช้ lazy evaluation ด้วย %s formatting
# แทนที่จะใช้ f-string ที่ evaluate ทันที
logger.debug("User ID: %d, Name: %s", 42, "John")  # ดีกว่า
# logger.debug(f"User ID: {42}, Name: {'John'}")   # ไม่แนะนำ
```

---

## 3. Handlers

Handler กำหนดว่า log messages จะถูกส่งไปที่ไหน

### ตัวอย่างที่ 4: StreamHandler - ส่งไปที่ console

```python
import logging
import sys

# สร้าง logger
logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# StreamHandler ส่ง log ไปยัง stdout หรือ stderr
stdout_handler = logging.StreamHandler(sys.stdout)
stdout_handler.setLevel(logging.DEBUG)

# สร้าง formatter สำหรับ handler นี้
formatter = logging.Formatter(
    '%(asctime)s [%(levelname)8s] %(name)s: %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)
stdout_handler.setFormatter(formatter)

# เพิ่ม handler ให้กับ logger
logger.addHandler(stdout_handler)

logger.info("โปรแกรมเริ่มทำงาน")
logger.debug("กำลัง load configuration...")
logger.warning("พบ deprecated feature")
```

### ตัวอย่างที่ 5: FileHandler - บันทึกลงไฟล์

```python
import logging

logger = logging.getLogger('fileapp')
logger.setLevel(logging.DEBUG)

# FileHandler บันทึก log ลงในไฟล์
file_handler = logging.FileHandler(
    'app.log',
    mode='a',           # 'a' = append, 'w' = overwrite
    encoding='utf-8'    # รองรับ Unicode
)
file_handler.setLevel(logging.INFO)  # บันทึกเฉพาะ INFO ขึ้นไป

formatter = logging.Formatter(
    '%(asctime)s - %(levelname)s - %(funcName)s:%(lineno)d - %(message)s'
)
file_handler.setFormatter(formatter)

logger.addHandler(file_handler)

# ทดสอบ
for i in range(5):
    logger.info(f"ประมวลผล item {i+1}")
    if i == 3:
        logger.warning(f"item {i+1} มีปัญหา")

print("Log บันทึกลงไฟล์ app.log แล้ว")
```

### ตัวอย่างที่ 6: RotatingFileHandler - หมุนเวียนไฟล์ log

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger('rotating_app')
logger.setLevel(logging.DEBUG)

# RotatingFileHandler จะสร้างไฟล์ใหม่เมื่อไฟล์เต็ม
# maxBytes=1MB, backupCount=5 หมายถึง เก็บ app.log + app.log.1 ถึง app.log.5
rotating_handler = RotatingFileHandler(
    'rotating_app.log',
    maxBytes=1024 * 1024,  # 1 MB
    backupCount=5,
    encoding='utf-8'
)
rotating_handler.setLevel(logging.DEBUG)

formatter = logging.Formatter(
    '%(asctime)s - %(levelname)s - %(message)s'
)
rotating_handler.setFormatter(formatter)

logger.addHandler(rotating_handler)

# จำลองการ log จำนวนมาก
for i in range(100):
    logger.debug(f"Debug message {i}: {'x' * 100}")

print("RotatingFileHandler ทดสอบเสร็จแล้ว")
```

### ตัวอย่างที่ 7: TimedRotatingFileHandler - หมุนเวียนตามเวลา

```python
import logging
from logging.handlers import TimedRotatingFileHandler
from datetime import datetime

logger = logging.getLogger('timed_app')
logger.setLevel(logging.INFO)

# หมุนเวียนทุกวันเที่ยงคืน เก็บ 7 วัน
timed_handler = TimedRotatingFileHandler(
    'timed_app.log',
    when='midnight',    # 'S','M','H','D','W0'-'W6','midnight'
    interval=1,         # ทุก 1 วัน
    backupCount=7,      # เก็บ 7 ไฟล์ล่าสุด
    encoding='utf-8'
)
timed_handler.setLevel(logging.INFO)

formatter = logging.Formatter(
    '%(asctime)s - %(levelname)s - %(message)s'
)
timed_handler.setFormatter(formatter)

logger.addHandler(timed_handler)

logger.info(f"Application started at {datetime.now()}")
```

### ตัวอย่างที่ 8: ส่ง log ไปหลายที่พร้อมกัน

```python
import logging
import sys
from logging.handlers import RotatingFileHandler

def setup_logger(name: str, log_file: str) -> logging.Logger:
    """ตั้งค่า logger ที่ส่ง log ไปทั้ง console และไฟล์"""
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)

    # Console handler - แสดงเฉพาะ WARNING ขึ้นไป
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.WARNING)
    console_formatter = logging.Formatter(
        '%(levelname)s: %(message)s'
    )
    console_handler.setFormatter(console_formatter)

    # File handler - บันทึกทุก level
    file_handler = RotatingFileHandler(
        log_file,
        maxBytes=10 * 1024 * 1024,  # 10 MB
        backupCount=3,
        encoding='utf-8'
    )
    file_handler.setLevel(logging.DEBUG)
    file_formatter = logging.Formatter(
        '%(asctime)s [%(levelname)8s] %(name)s:%(lineno)d - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    file_handler.setFormatter(file_formatter)

    # เพิ่ม handlers
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

    return logger

# ใช้งาน
app_logger = setup_logger('myapp', 'myapp.log')
app_logger.debug("เริ่ม initialize...")     # เฉพาะในไฟล์
app_logger.info("โหลด config สำเร็จ")      # เฉพาะในไฟล์
app_logger.warning("API rate limit ใกล้จะเต็ม")  # console + ไฟล์
app_logger.error("Database timeout!")       # console + ไฟล์
```

---

## 4. Formatters

Formatter กำหนดรูปแบบของ log messages

### ตัวอย่างที่ 9: Format Attributes ที่ใช้บ่อย

```python
import logging

# ตาราง format attributes ที่ใช้บ่อย:
# %(asctime)s     - เวลา (ตาม datefmt)
# %(name)s        - ชื่อ logger
# %(levelname)s   - ชื่อ level (DEBUG, INFO, ...)
# %(levelno)d     - เลข level (10, 20, ...)
# %(pathname)s    - full path ของไฟล์
# %(filename)s    - ชื่อไฟล์
# %(module)s      - ชื่อ module
# %(funcName)s    - ชื่อฟังก์ชัน
# %(lineno)d      - เลขบรรทัด
# %(message)s     - ข้อความ log
# %(process)d     - process ID
# %(thread)d      - thread ID
# %(threadName)s  - ชื่อ thread

formatter = logging.Formatter(
    fmt='%(asctime)s | %(levelname)-8s | %(name)s | '
        '%(filename)s:%(lineno)d | %(funcName)s() | %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

handler = logging.StreamHandler()
handler.setFormatter(formatter)

logger = logging.getLogger('format_demo')
logger.setLevel(logging.DEBUG)
logger.addHandler(handler)

def process_user(user_id: int) -> None:
    logger.info("กำลัง process user %d", user_id)
    logger.debug("ดึงข้อมูล user จาก database")

process_user(123)
```

### ตัวอย่างที่ 10: Custom Formatter

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """Custom formatter ที่ output เป็น JSON"""

    def format(self, record: logging.LogRecord) -> str:
        log_data = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
        }

        # เพิ่ม exception info ถ้ามี
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)

        # เพิ่ม extra fields ถ้ามี
        for key, value in record.__dict__.items():
            if key not in ('msg', 'args', 'exc_info', 'exc_text',
                          'stack_info', 'levelname', 'name', 'pathname',
                          'filename', 'module', 'funcName', 'lineno',
                          'created', 'msecs', 'relativeCreated', 'thread',
                          'threadName', 'processName', 'process',
                          'message', 'levelno'):
                log_data[key] = value

        return json.dumps(log_data, ensure_ascii=False)

# ใช้งาน JSON formatter
json_handler = logging.StreamHandler()
json_handler.setFormatter(JSONFormatter())

logger = logging.getLogger('json_app')
logger.setLevel(logging.DEBUG)
logger.addHandler(json_handler)

# เพิ่ม extra data ใน log
logger.info("User logged in", extra={'user_id': 42, 'ip': '192.168.1.1'})

try:
    result = 1 / 0
except ZeroDivisionError:
    logger.error("Division by zero error", exc_info=True)
```

---

## 5. Logger Hierarchy

Logger ใน Python ทำงานแบบ hierarchy โดยใช้ namespace ที่แยกด้วย dot (.)

### ตัวอย่างที่ 11: Logger Hierarchy

```python
import logging

# Root logger (parent ของทุก logger)
root_logger = logging.getLogger()
root_logger.setLevel(logging.WARNING)

# Child loggers
app_logger = logging.getLogger('myapp')
db_logger = logging.getLogger('myapp.database')
api_logger = logging.getLogger('myapp.api')
auth_logger = logging.getLogger('myapp.api.auth')

# ตั้งค่า root logger handler
root_handler = logging.StreamHandler()
root_handler.setFormatter(
    logging.Formatter('ROOT: %(name)s - %(levelname)s - %(message)s')
)
root_logger.addHandler(root_handler)

# ตั้งค่า app logger เพิ่มเติม
app_handler = logging.StreamHandler()
app_handler.setFormatter(
    logging.Formatter('APP: %(name)s - %(levelname)s - %(message)s')
)
app_logger.addHandler(app_handler)
app_logger.setLevel(logging.DEBUG)  # override parent level

# ทดสอบ propagation
print("=== Testing Logger Hierarchy ===")
db_logger.debug("DB debug - propagates to myapp")
api_logger.info("API info - propagates to myapp")
auth_logger.warning("Auth warning - propagates up the chain")

# ปิด propagation
db_logger.propagate = False
print("\n=== After disabling propagation for db_logger ===")
db_logger.info("DB info - ไม่ propagate แล้ว")
```

### ตัวอย่างที่ 12: การจัดการ Logger ในโปรเจกต์

```python
# project_structure/
# ├── main.py
# ├── database.py
# └── api/
#     ├── __init__.py
#     └── handlers.py

# ในแต่ละไฟล์ใช้ __name__ เป็นชื่อ logger
# main.py
import logging

# ตั้งค่าครั้งเดียวที่ entry point
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)  # ชื่อจะเป็น '__main__'
logger.info("Application started")

# database.py
# logger = logging.getLogger(__name__)  # ชื่อจะเป็น 'database'

# api/handlers.py
# logger = logging.getLogger(__name__)  # ชื่อจะเป็น 'api.handlers'

# วิธีนี้ทำให้ hierarchy ตามโครงสร้างไฟล์
# สามารถปิด/เปิด logging สำหรับ module เฉพาะได้ง่าย
logging.getLogger('database').setLevel(logging.WARNING)  # ลด log ของ database
logging.getLogger('api').setLevel(logging.DEBUG)         # เพิ่ม log ของ api
```

---

## 6. Configuration

### ตัวอย่างที่ 13: basicConfig

```python
import logging

# basicConfig - วิธีตั้งค่าที่ง่ายที่สุด
# ต้องเรียกก่อนที่จะมีการ log ใดๆ
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename='app.log',        # ถ้าไม่ระบุจะใช้ stderr
    filemode='a',              # 'a' หรือ 'w'
    encoding='utf-8'
)

# ตั้งค่า level สำหรับ root logger
logging.getLogger().setLevel(logging.DEBUG)

logging.info("basicConfig ตั้งค่าแล้ว")
logging.debug("Debug message")
```

### ตัวอย่างที่ 14: dictConfig - วิธีที่ยืดหยุ่นที่สุด

```python
import logging
import logging.config

# dictConfig ใช้ dictionary ตั้งค่า - เหมาะสำหรับโปรเจกต์ใหญ่
LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,  # ไม่ปิด logger ที่มีอยู่แล้ว

    'formatters': {
        'verbose': {
            'format': '%(asctime)s [%(levelname)8s] %(name)s:%(lineno)d - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
        'simple': {
            'format': '%(levelname)s: %(message)s'
        },
        'json': {
            '()': '__main__.JSONFormatter',  # Custom formatter class
        }
    },

    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'WARNING',
            'formatter': 'simple',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'verbose',
            'filename': 'app.log',
            'maxBytes': 10485760,  # 10 MB
            'backupCount': 5,
            'encoding': 'utf-8'
        },
        'error_file': {
            'class': 'logging.FileHandler',
            'level': 'ERROR',
            'formatter': 'verbose',
            'filename': 'errors.log',
            'encoding': 'utf-8'
        }
    },

    'loggers': {
        'myapp': {
            'level': 'DEBUG',
            'handlers': ['console', 'file'],
            'propagate': False
        },
        'myapp.database': {
            'level': 'WARNING',    # ลด log ของ database
            'handlers': ['file', 'error_file'],
            'propagate': False
        }
    },

    'root': {
        'level': 'INFO',
        'handlers': ['console']
    }
}

# นำ config ไปใช้
logging.config.dictConfig(LOGGING_CONFIG)

# ทดสอบ
logger = logging.getLogger('myapp')
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")

db_logger = logging.getLogger('myapp.database')
db_logger.debug("DB debug - ไม่แสดงเพราะ level เป็น WARNING")
db_logger.warning("DB warning - แสดง")
```

### ตัวอย่างที่ 15: fileConfig - โหลดจากไฟล์ .ini

```python
# logging.conf:
# [loggers]
# keys=root,myapp
#
# [handlers]
# keys=consoleHandler,fileHandler
#
# [formatters]
# keys=simpleFormatter
#
# [logger_root]
# level=DEBUG
# handlers=consoleHandler
#
# [logger_myapp]
# level=DEBUG
# handlers=fileHandler
# qualname=myapp
# propagate=0
#
# [handler_consoleHandler]
# class=StreamHandler
# level=WARNING
# formatter=simpleFormatter
# args=(sys.stdout,)
#
# [handler_fileHandler]
# class=FileHandler
# level=DEBUG
# formatter=simpleFormatter
# args=('app.log', 'a')
#
# [formatter_simpleFormatter]
# format=%(asctime)s - %(name)s - %(levelname)s - %(message)s

import logging
import logging.config
import os

# โหลด config จากไฟล์ (ถ้าไฟล์มีอยู่)
config_file = 'logging.conf'
if os.path.exists(config_file):
    logging.config.fileConfig(config_file, disable_existing_loggers=False)
else:
    # fallback ถ้าไม่มีไฟล์ config
    logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger('myapp')
logger.info("Logger configured from file")
```

---

## 7. Structured Logging

Structured logging เพิ่มข้อมูล context ใน log ทำให้วิเคราะห์ได้ง่ายขึ้น

### ตัวอย่างที่ 16: LoggerAdapter สำหรับ Context

```python
import logging

class UserContextAdapter(logging.LoggerAdapter):
    """เพิ่ม user context ให้ทุก log message"""

    def process(self, msg: str, kwargs: dict) -> tuple:
        # เพิ่ม user info ใน message
        user_info = f"[user={self.extra.get('user_id', 'unknown')}] "
        return user_info + msg, kwargs

# สร้าง base logger
base_logger = logging.getLogger('user_app')
base_logger.setLevel(logging.DEBUG)
handler = logging.StreamHandler()
handler.setFormatter(
    logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
)
base_logger.addHandler(handler)

# ใช้งาน adapter
def process_request(user_id: int, action: str) -> None:
    logger = UserContextAdapter(base_logger, {'user_id': user_id})
    logger.info(f"เริ่ม {action}")
    logger.debug(f"กำลัง validate {action}")
    logger.info(f"{action} สำเร็จ")

process_request(42, "purchase")
process_request(99, "login")
```

### ตัวอย่างที่ 17: Extra Fields ใน Log

```python
import logging
import uuid
from contextvars import ContextVar

# ContextVar สำหรับเก็บ request_id ใน async context
request_id_var: ContextVar[str] = ContextVar('request_id', default='')

class RequestIdFilter(logging.Filter):
    """Filter ที่เพิ่ม request_id ใน log record"""

    def filter(self, record: logging.LogRecord) -> bool:
        record.request_id = request_id_var.get() or str(uuid.uuid4())[:8]
        return True

# ตั้งค่า logger
logger = logging.getLogger('api')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.addFilter(RequestIdFilter())
handler.setFormatter(
    logging.Formatter(
        '%(asctime)s [%(request_id)s] %(levelname)s: %(message)s'
    )
)
logger.addHandler(handler)

# จำลอง HTTP requests
def handle_request(endpoint: str) -> None:
    # ตั้ง request ID ใหม่สำหรับแต่ละ request
    token = request_id_var.set(str(uuid.uuid4())[:8])
    try:
        logger.info(f"GET {endpoint}")
        logger.debug("กำลัง authenticate...")
        logger.info("Response 200 OK")
    finally:
        request_id_var.reset(token)

handle_request("/api/users")
handle_request("/api/products")
```

---

## 8. Third-party Libraries

### ตัวอย่างที่ 18: Loguru - Logging ที่ง่ายและทรงพลัง

```python
# ติดตั้ง: pip install loguru
from loguru import logger
import sys

# Loguru ใช้งานได้ทันทีโดยไม่ต้องตั้งค่า
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
logger.critical("Critical message")

# ลบ default handler และเพิ่มของเอง
logger.remove()
logger.add(
    sys.stdout,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
           "<level>{level: <8}</level> | "
           "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> | "
           "<level>{message}</level>",
    level="DEBUG",
    colorize=True
)

# เพิ่ม file logging
logger.add(
    "loguru_{time:YYYY-MM-DD}.log",
    rotation="1 day",           # หมุนทุกวัน
    retention="7 days",         # เก็บ 7 วัน
    compression="zip",          # บีบอัด
    level="INFO",
    encoding="utf-8"
)

# ใช้งาน context
logger.bind(user_id=42).info("User action")

# Exception logging อัตโนมัติ
@logger.catch
def risky_function(x: int) -> int:
    return 10 / x

risky_function(2)   # ปกติ
risky_function(0)   # จะ log exception โดยอัตโนมัติ
```

### ตัวอย่างที่ 19: Loguru Structured Logging

```python
from loguru import logger
import json

# Custom serializer สำหรับ JSON output
def serialize(record):
    subset = {
        "timestamp": record["time"].isoformat(),
        "level": record["level"].name,
        "message": record["message"],
        "file": record["file"].name,
        "line": record["line"],
        "function": record["function"],
    }
    # เพิ่ม extra data
    if record["extra"]:
        subset["extra"] = record["extra"]
    return json.dumps(subset, ensure_ascii=False)

def patching(record):
    record["extra"]["serialized"] = serialize(record)

logger.remove()
logger.add(
    "structured.log",
    format="{extra[serialized]}",
    level="DEBUG"
)

# ใช้งาน
logger.bind(user_id=42, action="login").info("User logged in")
logger.bind(order_id="ORD-123", amount=1500).info("Order created")
```

### ตัวอย่างที่ 20: Structlog

```python
# ติดตั้ง: pip install structlog
import structlog
import logging

# ตั้งค่า structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.UnicodeDecoder(),
        structlog.dev.ConsoleRenderer()  # ใช้ JSONRenderer() สำหรับ production
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

# สร้าง logger
log = structlog.get_logger()

# ใช้งาน
log.info("user_logged_in", user_id=42, ip="192.168.1.1")
log.warning("rate_limit_approaching", current=95, limit=100)
log.error("database_error", db="postgres", query="SELECT * FROM users")

# Bind context
user_log = log.bind(user_id=42, session_id="abc123")
user_log.info("profile_viewed")
user_log.info("cart_updated", items=3)
```

---

## 9. Debugging Techniques

### ตัวอย่างที่ 21: print() Debugging (วิธีเบื้องต้น)

```python
# วิธีนี้ง่ายแต่ไม่แนะนำสำหรับโปรเจกต์จริง
def calculate_discount(price: float, discount_pct: float) -> float:
    print(f"DEBUG: price={price}, discount_pct={discount_pct}")  # DEBUG print
    
    if discount_pct > 100:
        print("ERROR: discount_pct > 100!")  # DEBUG print
        return price
    
    discount_amount = price * (discount_pct / 100)
    print(f"DEBUG: discount_amount={discount_amount}")  # DEBUG print
    
    final_price = price - discount_amount
    print(f"DEBUG: final_price={final_price}")  # DEBUG print
    
    return final_price

result = calculate_discount(1000, 15)
print(f"Result: {result}")

# ปัญหาของวิธีนี้:
# 1. ต้องลบ print() ทุกตัวก่อน production
# 2. ไม่มี timestamp หรือ level
# 3. ยากต่อการควบคุม
```

### ตัวอย่างที่ 22: assert สำหรับ Debugging

```python
def divide(a: float, b: float) -> float:
    # assert ตรวจสอบ preconditions
    assert isinstance(a, (int, float)), f"a ต้องเป็น number ไม่ใช่ {type(a)}"
    assert isinstance(b, (int, float)), f"b ต้องเป็น number ไม่ใช่ {type(b)}"
    assert b != 0, "ห้าม divide ด้วย 0"
    
    result = a / b
    
    # assert ตรวจสอบ postconditions
    assert isinstance(result, float), "result ต้องเป็น float"
    
    return result

# ทดสอบ
print(divide(10, 3))    # ปกติ
# print(divide(10, 0))  # AssertionError: ห้าม divide ด้วย 0

# หมายเหตุ: assert ถูก disable เมื่อรันด้วย python -O (optimize)
# ดังนั้นใช้สำหรับ development เท่านั้น ไม่ใช่ production error handling
```

---

## 10. pdb Debugger

### ตัวอย่างที่ 23: การใช้ pdb พื้นฐาน

```python
import pdb

def complex_function(data: list) -> dict:
    result = {}
    
    for i, item in enumerate(data):
        # วาง breakpoint ที่นี่
        pdb.set_trace()  # โปรแกรมจะหยุดที่นี่และเปิด pdb console
        
        processed = item * 2
        result[f"item_{i}"] = processed
    
    return result

# คำสั่ง pdb ที่ใช้บ่อย:
# n (next)      - รันบรรทัดถัดไป (ไม่ step เข้าฟังก์ชัน)
# s (step)      - step เข้าไปในฟังก์ชัน
# c (continue)  - รันต่อจนถึง breakpoint ถัดไป
# q (quit)      - ออกจาก debugger
# p <expr>      - print ค่าของ expression
# pp <expr>     - pretty print
# l (list)      - แสดง source code รอบๆ บรรทัดปัจจุบัน
# w (where)     - แสดง call stack
# u (up)        - ขึ้นไป stack frame ด้านบน
# d (down)      - ลงไป stack frame ด้านล่าง
# b <line>      - ตั้ง breakpoint ที่บรรทัดนั้น
# h (help)      - แสดง help

# ทดสอบ (จะเปิด pdb console)
# data = [1, 2, 3, 4, 5]
# result = complex_function(data)
```

---

## 11. breakpoint() (Python 3.7+)

### ตัวอย่างที่ 24: breakpoint() และ PYTHONBREAKPOINT

```python
# breakpoint() เป็น built-in ใน Python 3.7+
# เทียบเท่ากับ import pdb; pdb.set_trace()
# แต่ยืดหยุ่นกว่าเพราะควบคุมได้ด้วย environment variable

def process_order(order_id: str, items: list, total: float) -> bool:
    """ประมวลผล order"""
    
    # Validate
    if not order_id:
        return False
    
    breakpoint()  # หยุดที่นี่เมื่อ debug
    
    if total <= 0:
        return False
    
    # Process items
    for item in items:
        print(f"Processing: {item}")
    
    return True

# ควบคุม breakpoint() ด้วย environment variable:
# PYTHONBREAKPOINT=0 python script.py          - ปิด breakpoints ทั้งหมด
# PYTHONBREAKPOINT=ipdb.set_trace python script.py - ใช้ ipdb แทน pdb
# PYTHONBREAKPOINT=pudb.set_trace python script.py - ใช้ pudb (visual debugger)

# ทดสอบ
# order = process_order("ORD-001", ["item1", "item2"], 500.0)
```

### ตัวอย่างที่ 25: Post-mortem Debugging

```python
import pdb
import sys

def buggy_function(data: dict) -> str:
    """ฟังก์ชันที่มี bug"""
    return data['nonexistent_key']  # จะเกิด KeyError

# วิธีที่ 1: ใช้ pdb.pm() หลัง exception
try:
    result = buggy_function({'key': 'value'})
except KeyError:
    print("เกิด KeyError!")
    pdb.pm()  # เปิด debugger ณ จุดที่ exception เกิด

# วิธีที่ 2: ใช้ pdb.post_mortem() กับ traceback
def debug_on_error(func, *args, **kwargs):
    """Wrapper ที่เปิด debugger เมื่อเกิด exception"""
    try:
        return func(*args, **kwargs)
    except Exception:
        import traceback
        traceback.print_exc()
        pdb.post_mortem(sys.exc_info()[2])

# วิธีที่ 3: รันด้วย python -m pdb script.py
# แล้วพิมพ์ 'c' เพื่อรันจนเกิด error
```

---

## 12. VS Code Debugging

### ตัวอย่างที่ 26: การตั้งค่า launch.json สำหรับ VS Code

```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": true
        },
        {
            "name": "Python: Debug Tests",
            "type": "python",
            "request": "launch",
            "module": "pytest",
            "args": ["-v", "--no-header", "${file}"],
            "console": "integratedTerminal",
            "justMyCode": false
        },
        {
            "name": "Python: FastAPI",
            "type": "python",
            "request": "launch",
            "module": "uvicorn",
            "args": ["main:app", "--reload", "--port", "8000"],
            "console": "integratedTerminal",
            "env": {
                "DEBUG": "true",
                "LOG_LEVEL": "debug"
            }
        },
        {
            "name": "Python: Attach",
            "type": "python",
            "request": "attach",
            "connect": {
                "host": "localhost",
                "port": 5678
            }
        }
    ]
}
```

### ตัวอย่างที่ 27: Debugging Tips ใน VS Code

```python
# VS Code Debugging Tips:
# 1. F5           - เริ่ม debug
# 2. F9           - toggle breakpoint
# 3. F10          - Step Over (เหมือน pdb 'n')
# 4. F11          - Step Into (เหมือน pdb 's')
# 5. Shift+F11    - Step Out
# 6. Ctrl+Shift+F5 - Restart
# 7. Shift+F5     - Stop

# Conditional Breakpoints - breakpoint ที่ทำงานเฉพาะเงื่อนไข
# คลิกขวาที่ breakpoint > Edit Breakpoint > condition

def process_items(items: list) -> None:
    for i, item in enumerate(items):
        # ใส่ conditional breakpoint ที่นี่:
        # condition: i == 5 หรือ item > 100
        result = item * 2
        print(f"Item {i}: {result}")

# Logpoints - เหมือน breakpoint แต่ log แทนที่จะหยุด
# คลิกขวาที่ breakpoint > Add Logpoint
# พิมพ์: Processing item {i} with value {item}

process_items(list(range(10)))
```

---

## 13. Remote Debugging

### ตัวอย่างที่ 28: Remote Debugging ด้วย debugpy

```python
# ติดตั้ง: pip install debugpy

# ในโปรแกรม (server/container):
import debugpy

# ให้ debugpy รอการเชื่อมต่อที่ port 5678
debugpy.listen(("0.0.0.0", 5678))

print("รอการเชื่อมต่อ debugger...")
debugpy.wait_for_client()  # หยุดรอจนกว่า VS Code จะเชื่อมต่อ
print("Debugger เชื่อมต่อแล้ว!")

# โปรแกรมปกติ
def main():
    for i in range(10):
        result = i ** 2
        print(f"{i}^2 = {result}")

main()

# ใน VS Code ใช้ "Python: Attach" configuration
# host: localhost (หรือ IP ของ server)
# port: 5678
```

---

## 14. Profiling

### ตัวอย่างที่ 29: cProfile - Standard Profiler

```python
import cProfile
import pstats
import io
from pstats import SortKey

def fibonacci(n: int) -> int:
    """ฟังก์ชัน fibonacci แบบ recursive (ช้า)"""
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

def compute_many_fibonacci() -> list:
    """คำนวณ fibonacci หลายๆ ค่า"""
    return [fibonacci(i) for i in range(30)]

# วิธีที่ 1: profile ทั้งโปรแกรม
# python -m cProfile -s cumulative script.py

# วิธีที่ 2: profile เฉพาะส่วน
profiler = cProfile.Profile()
profiler.enable()

result = compute_many_fibonacci()

profiler.disable()

# แสดงผล
stream = io.StringIO()
stats = pstats.Stats(profiler, stream=stream)
stats.sort_stats(SortKey.CUMULATIVE)  # เรียงตาม cumulative time
stats.print_stats(20)  # แสดง 20 อันดับแรก

print(stream.getvalue())

# วิธีที่ 3: ใช้ context manager
import cProfile
with cProfile.Profile() as pr:
    result = compute_many_fibonacci()

pr.print_stats(sort='cumulative')
```

### ตัวอย่างที่ 30: timeit สำหรับ Micro-benchmarking

```python
import timeit

# วัดเวลาการ join string แบบต่างๆ
setup = "data = list(range(1000))"

# วิธีที่ 1: string concatenation
test1 = """
result = ""
for item in data:
    result += str(item) + ", "
"""

# วิธีที่ 2: join
test2 = """
result = ", ".join(str(item) for item in data)
"""

# วิธีที่ 3: f-string
test3 = """
result = ", ".join(f"{item}" for item in data)
"""

# วัดเวลา
time1 = timeit.timeit(test1, setup=setup, number=1000)
time2 = timeit.timeit(test2, setup=setup, number=1000)
time3 = timeit.timeit(test3, setup=setup, number=1000)

print(f"String concat: {time1:.4f}s")
print(f"Join:          {time2:.4f}s")
print(f"F-string join: {time3:.4f}s")
print(f"\nJoin เร็วกว่า concat: {time1/time2:.1f}x")
```

### ตัวอย่างที่ 31: memory_profiler

```python
# ติดตั้ง: pip install memory-profiler
# รัน: python -m memory_profiler script.py

from memory_profiler import profile, memory_usage

@profile  # decorator จะแสดง memory usage ทีละบรรทัด
def memory_hungry_function() -> list:
    """ฟังก์ชันที่ใช้ memory มาก"""
    # สร้าง list ขนาดใหญ่
    big_list = [i for i in range(1_000_000)]
    
    # แปลงเป็น set
    big_set = set(big_list)
    
    # กรองข้อมูล
    filtered = [x for x in big_set if x % 2 == 0]
    
    return filtered

# วัด memory usage ของฟังก์ชัน
mem_usage = memory_usage((memory_hungry_function, [], {}))
print(f"Peak memory usage: {max(mem_usage):.2f} MB")
print(f"Memory increase: {max(mem_usage) - min(mem_usage):.2f} MB")
```

### ตัวอย่างที่ 32: line_profiler

```python
# ติดตั้ง: pip install line-profiler
# รัน: kernprof -l -v script.py

from line_profiler import LineProfiler

def slow_function(data: list) -> list:
    """ฟังก์ชันที่อาจมี bottleneck"""
    result = []
    
    for item in data:
        # ขั้นตอนที่ 1: แปลงข้อมูล
        processed = item ** 2
        
        # ขั้นตอนที่ 2: กรองข้อมูล
        if processed % 2 == 0:
            # ขั้นตอนที่ 3: เพิ่มใน result
            result.append(processed)
    
    return result

# ใช้ LineProfiler
profiler = LineProfiler()
profiler.add_function(slow_function)
profiler.enable()

data = list(range(10000))
result = slow_function(data)

profiler.disable()
profiler.print_stats()
```

### ตัวอย่างที่ 33: Logging Context Manager สำหรับ Performance

```python
import logging
import time
from contextlib import contextmanager
from functools import wraps
from typing import Callable, Any

logger = logging.getLogger(__name__)
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

@contextmanager
def log_performance(operation_name: str):
    """Context manager สำหรับ log เวลาที่ใช้"""
    start_time = time.perf_counter()
    logger.debug(f"เริ่ม: {operation_name}")
    
    try:
        yield
        elapsed = time.perf_counter() - start_time
        logger.info(f"สำเร็จ: {operation_name} ใช้เวลา {elapsed:.3f}s")
    except Exception as e:
        elapsed = time.perf_counter() - start_time
        logger.error(f"ล้มเหลว: {operation_name} หลังจาก {elapsed:.3f}s - {e}")
        raise

def log_execution_time(func: Callable) -> Callable:
    """Decorator สำหรับ log เวลาที่ฟังก์ชันใช้"""
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        with log_performance(func.__name__):
            return func(*args, **kwargs)
    return wrapper

# ใช้งาน
@log_execution_time
def process_large_dataset(size: int) -> list:
    """จำลองการประมวลผลข้อมูลขนาดใหญ่"""
    time.sleep(0.1)  # จำลอง I/O
    return [i * 2 for i in range(size)]

# Context manager
with log_performance("database query"):
    time.sleep(0.05)  # จำลอง DB query

# Decorator
result = process_large_dataset(10000)
print(f"ผลลัพธ์: {len(result)} items")
```

### ตัวอย่างที่ 34: Exception Logging

```python
import logging
import traceback

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.DEBUG)

def safe_divide(a: float, b: float) -> float | None:
    """หาร a ด้วย b อย่างปลอดภัย"""
    try:
        result = a / b
        logger.debug("คำนวณ %s / %s = %s", a, b, result)
        return result
    except ZeroDivisionError:
        # exc_info=True จะเพิ่ม traceback ใน log
        logger.error("ไม่สามารถหารด้วย 0 ได้", exc_info=True)
        return None
    except TypeError as e:
        logger.error("Type error: %s", e, exc_info=True)
        return None

def process_with_exception_chain(data: list) -> None:
    """แสดงการ log exception chain"""
    try:
        try:
            result = data[100]  # IndexError
        except IndexError as e:
            raise ValueError("ข้อมูลไม่เพียงพอ") from e
    except ValueError:
        logger.exception("เกิดข้อผิดพลาดในการ process data")
        # logger.exception() เทียบเท่า logger.error(..., exc_info=True)

# ทดสอบ
print(safe_divide(10, 2))
print(safe_divide(10, 0))
process_with_exception_chain([1, 2, 3])
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: ตั้งค่า Logging System

**โจทย์:** สร้าง logging system สำหรับแอปพลิเคชัน e-commerce ที่:
- บันทึก DEBUG ขึ้นไปลงไฟล์ `app.log` (rotating, max 5MB, backup 3)
- แสดง WARNING ขึ้นไปที่ console
- บันทึก ERROR ขึ้นไปลงไฟล์ `errors.log` แยกต่างหาก
- Format มี timestamp, level, module name, line number

**เฉลย:**

```python
import logging
import logging.config
import sys

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'detailed': {
            'format': '%(asctime)s [%(levelname)-8s] %(name)s:%(lineno)d - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
        'console': {
            'format': '%(levelname)s: %(message)s'
        }
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'WARNING',
            'formatter': 'console',
            'stream': 'ext://sys.stdout'
        },
        'app_file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'detailed',
            'filename': 'app.log',
            'maxBytes': 5 * 1024 * 1024,
            'backupCount': 3,
            'encoding': 'utf-8'
        },
        'error_file': {
            'class': 'logging.FileHandler',
            'level': 'ERROR',
            'formatter': 'detailed',
            'filename': 'errors.log',
            'encoding': 'utf-8'
        }
    },
    'root': {
        'level': 'DEBUG',
        'handlers': ['console', 'app_file', 'error_file']
    }
}

logging.config.dictConfig(LOGGING_CONFIG)

# ทดสอบ
logger = logging.getLogger('ecommerce.orders')
logger.debug("กำลัง process order...")
logger.info("Order ORD-001 received")
logger.warning("Stock เหลือน้อยสำหรับ product P-42")
logger.error("Payment failed for order ORD-002")
logger.critical("Database connection lost!")
```

---

### แบบฝึกหัดที่ 2: Custom Logger สำหรับ API

**โจทย์:** สร้าง logger middleware ที่บันทึก HTTP requests และ responses พร้อม:
- Request ID ที่ unique
- Response time
- Status code
- User agent

**เฉลย:**

```python
import logging
import uuid
import time
from functools import wraps
from typing import Callable, Any

# ตั้งค่า logger
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(request_id)s] %(levelname)s: %(message)s'
)

class APILogger:
    """Logger สำหรับ API requests"""

    def __init__(self, name: str):
        self.logger = logging.getLogger(name)

    def log_request(
        self,
        method: str,
        path: str,
        user_agent: str = "Unknown"
    ) -> str:
        request_id = str(uuid.uuid4())[:8]
        self.logger.info(
            f"{method} {path}",
            extra={
                'request_id': request_id,
                'method': method,
                'path': path,
                'user_agent': user_agent
            }
        )
        return request_id

    def log_response(
        self,
        request_id: str,
        status_code: int,
        response_time: float
    ) -> None:
        level = logging.INFO if status_code < 400 else logging.ERROR
        self.logger.log(
            level,
            f"Response {status_code} ({response_time:.3f}s)",
            extra={'request_id': request_id}
        )

def api_logger_middleware(func: Callable) -> Callable:
    """Decorator สำหรับ log API calls"""
    logger = APILogger('api')

    @wraps(func)
    def wrapper(method: str, path: str, **kwargs: Any) -> Any:
        request_id = logger.log_request(method, path)
        start_time = time.perf_counter()

        try:
            result = func(method, path, **kwargs)
            elapsed = time.perf_counter() - start_time
            status = result.get('status', 200)
            logger.log_response(request_id, status, elapsed)
            return result
        except Exception as e:
            elapsed = time.perf_counter() - start_time
            logger.logger.error(
                f"Internal Server Error: {e}",
                extra={'request_id': request_id},
                exc_info=True
            )
            logger.log_response(request_id, 500, elapsed)
            raise

    return wrapper

@api_logger_middleware
def mock_api_call(method: str, path: str) -> dict:
    """จำลอง API call"""
    time.sleep(0.1)  # จำลอง network latency
    if path == "/api/error":
        raise ValueError("Something went wrong")
    return {'status': 200, 'data': 'OK'}

# ทดสอบ
mock_api_call("GET", "/api/users")
mock_api_call("POST", "/api/orders")
try:
    mock_api_call("GET", "/api/error")
except ValueError:
    pass
```

---

### แบบฝึกหัดที่ 3: Debugging ด้วย pdb

**โจทย์:** Debug ฟังก์ชันต่อไปนี้ที่มี bug โดยใช้ pdb

```python
def merge_sorted_lists(list1: list, list2: list) -> list:
    """รวม 2 sorted lists เป็น 1 sorted list"""
    result = []
    i, j = 0, 0

    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1

    # Bug: ขาดการ append ส่วนที่เหลือ
    # ต้องเพิ่ม:
    # result.extend(list1[i:])
    # result.extend(list2[j:])

    return result

# ทดสอบ - จะได้ผลผิด
result = merge_sorted_lists([1, 3, 5, 7, 9], [2, 4, 6, 8, 10])
print(f"Expected: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]")
print(f"Got:      {result}")
```

**เฉลย (ฟังก์ชันที่แก้แล้ว):**

```python
import pdb

def merge_sorted_lists_debug(list1: list, list2: list) -> list:
    """รวม 2 sorted lists เป็น 1 sorted list - debugged version"""
    result = []
    i, j = 0, 0

    # ใช้ pdb เพื่อ debug
    # pdb.set_trace()  # uncommment เพื่อ debug

    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1

    # Fix: เพิ่มส่วนที่เหลือของทั้ง 2 list
    result.extend(list1[i:])
    result.extend(list2[j:])

    return result

# ทดสอบ
result = merge_sorted_lists_debug([1, 3, 5, 7, 9], [2, 4, 6, 8, 10])
print(f"Expected: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]")
print(f"Got:      {result}")
assert result == [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], "Test failed!"
print("Test passed!")
```

---

### แบบฝึกหัดที่ 4: Performance Profiling

**โจทย์:** Profile โปรแกรมต่อไปนี้และหา bottleneck

**เฉลย:**

```python
import cProfile
import pstats
import io
from pstats import SortKey
import time

def find_primes_slow(n: int) -> list:
    """หาจำนวนเฉพาะ ≤ n แบบช้า"""
    primes = []
    for num in range(2, n + 1):
        is_prime = True
        for divisor in range(2, num):  # O(n²)
            if num % divisor == 0:
                is_prime = False
                break
        if is_prime:
            primes.append(num)
    return primes

def find_primes_fast(n: int) -> list:
    """หาจำนวนเฉพาะ ≤ n แบบ Sieve of Eratosthenes"""
    if n < 2:
        return []
    sieve = [True] * (n + 1)
    sieve[0] = sieve[1] = False
    for i in range(2, int(n**0.5) + 1):
        if sieve[i]:
            for j in range(i*i, n+1, i):
                sieve[j] = False
    return [i for i, is_prime in enumerate(sieve) if is_prime]

# Profile slow version
print("=== Profiling slow version ===")
with cProfile.Profile() as prof_slow:
    result_slow = find_primes_slow(5000)

stats_slow = pstats.Stats(prof_slow)
stats_slow.sort_stats(SortKey.CUMULATIVE)
stats_slow.print_stats(5)

# Profile fast version
print("\n=== Profiling fast version ===")
with cProfile.Profile() as prof_fast:
    result_fast = find_primes_fast(5000)

stats_fast = pstats.Stats(prof_fast)
stats_fast.sort_stats(SortKey.CUMULATIVE)
stats_fast.print_stats(5)

# เปรียบเทียบ
print(f"\nSlow found {len(result_slow)} primes")
print(f"Fast found {len(result_fast)} primes")
assert result_slow == result_fast, "Results don't match!"
print("Results match!")
```

---

### แบบฝึกหัดที่ 5: Structured Logging

**โจทย์:** สร้าง JSON logger สำหรับ microservice ที่มี request tracking

**เฉลย:**

```python
import logging
import json
import uuid
import time
from datetime import datetime, timezone
from typing import Optional

class JSONLogger:
    """Logger ที่ output เป็น JSON สำหรับ microservice"""

    def __init__(self, service_name: str, version: str):
        self.service_name = service_name
        self.version = version
        self._setup_logger()

    def _setup_logger(self) -> None:
        self.logger = logging.getLogger(self.service_name)
        self.logger.setLevel(logging.DEBUG)

        handler = logging.StreamHandler()
        handler.setFormatter(self._create_formatter())
        self.logger.addHandler(handler)

    def _create_formatter(self) -> logging.Formatter:
        class ServiceJSONFormatter(logging.Formatter):
            def __init__(self, service_name: str, version: str):
                super().__init__()
                self.service_name = service_name
                self.version = version

            def format(self, record: logging.LogRecord) -> str:
                log_entry = {
                    'timestamp': datetime.now(timezone.utc).isoformat(),
                    'service': self.service_name,
                    'version': self.version,
                    'level': record.levelname,
                    'message': record.getMessage(),
                    'logger': record.name,
                    'module': record.module,
                    'function': record.funcName,
                    'line': record.lineno,
                }

                # Extra fields
                for key in ('request_id', 'user_id', 'duration_ms', 'status_code'):
                    if hasattr(record, key):
                        log_entry[key] = getattr(record, key)

                if record.exc_info:
                    log_entry['exception'] = self.formatException(record.exc_info)

                return json.dumps(log_entry, ensure_ascii=False)

        return ServiceJSONFormatter(self.service_name, self.version)

    def request(self, method: str, path: str, request_id: Optional[str] = None) -> str:
        request_id = request_id or str(uuid.uuid4())
        self.logger.info(
            f"{method} {path}",
            extra={'request_id': request_id}
        )
        return request_id

    def response(
        self,
        request_id: str,
        status_code: int,
        duration_ms: float
    ) -> None:
        level = logging.INFO if status_code < 400 else logging.ERROR
        self.logger.log(
            level,
            f"Response {status_code}",
            extra={
                'request_id': request_id,
                'status_code': status_code,
                'duration_ms': round(duration_ms, 2)
            }
        )

# ทดสอบ
json_logger = JSONLogger("payment-service", "1.2.0")

request_id = json_logger.request("POST", "/api/payments")
time.sleep(0.05)  # จำลอง processing
json_logger.response(request_id, 200, 52.3)

request_id = json_logger.request("GET", "/api/payments/123")
json_logger.response(request_id, 404, 5.1)
```

---

### แบบฝึกหัดที่ 6: Exception Tracking

**โจทย์:** สร้าง exception tracker ที่บันทึก exceptions พร้อม context

**เฉลย:**

```python
import logging
import traceback
import sys
from functools import wraps
from typing import Type, Callable, Any

class ExceptionTracker:
    """ติดตามและบันทึก exceptions"""

    def __init__(self, logger_name: str):
        self.logger = logging.getLogger(logger_name)
        self.exception_counts: dict = {}

    def track(self, *exception_types: Type[Exception]):
        """Decorator สำหรับ track exceptions"""
        def decorator(func: Callable) -> Callable:
            @wraps(func)
            def wrapper(*args: Any, **kwargs: Any) -> Any:
                try:
                    return func(*args, **kwargs)
                except exception_types as e:
                    exc_type = type(e).__name__
                    self.exception_counts[exc_type] = \
                        self.exception_counts.get(exc_type, 0) + 1

                    self.logger.error(
                        f"Exception in {func.__name__}: {exc_type}: {e}",
                        exc_info=True,
                        extra={
                            'function': func.__name__,
                            'exception_type': exc_type,
                            'exception_count': self.exception_counts[exc_type],
                            'args': str(args[:3]),  # จำกัดขนาด
                        }
                    )
                    raise
            return wrapper
        return decorator

    def get_summary(self) -> dict:
        return self.exception_counts.copy()

# ตั้งค่า logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

tracker = ExceptionTracker('exception_tracker')

@tracker.track(ValueError, KeyError, TypeError)
def parse_user_data(data: dict) -> dict:
    """Parse user data - อาจเกิด exceptions"""
    return {
        'id': int(data['id']),          # KeyError ถ้าไม่มี 'id'
        'age': int(data['age']),        # ValueError ถ้า age ไม่ใช่ตัวเลข
        'name': data['name'].strip(),   # TypeError ถ้า name ไม่ใช่ string
    }

# ทดสอบ
test_cases = [
    {'id': '1', 'age': '25', 'name': 'Alice'},
    {'id': '2', 'age': 'invalid', 'name': 'Bob'},   # ValueError
    {'id': '3', 'name': 'Charlie'},                 # KeyError
    {'id': '4', 'age': '28', 'name': None},         # TypeError
]

for case in test_cases:
    try:
        result = parse_user_data(case)
        print(f"Success: {result}")
    except Exception as e:
        print(f"Failed: {type(e).__name__}")

print(f"\nException summary: {tracker.get_summary()}")
```

---

### แบบฝึกหัดที่ 7: Loguru Application

**โจทย์:** สร้าง application logger ด้วย Loguru ที่รองรับหลาย environment

**เฉลย:**

```python
# ต้องติดตั้ง loguru ก่อน: pip install loguru
import sys
import os
from loguru import logger

def configure_logging(environment: str = "development") -> None:
    """ตั้งค่า logging ตาม environment"""
    logger.remove()  # ลบ default handler

    if environment == "development":
        # Development: แสดงทุก level พร้อม color และ detailed format
        logger.add(
            sys.stdout,
            format="<green>{time:HH:mm:ss}</green> | "
                   "<level>{level: <8}</level> | "
                   "<cyan>{name}</cyan>:<cyan>{line}</cyan> | "
                   "<level>{message}</level>",
            level="DEBUG",
            colorize=True
        )
    elif environment == "production":
        # Production: บันทึกลงไฟล์ rotating พร้อม JSON format
        logger.add(
            "logs/app_{time:YYYY-MM-DD}.log",
            format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {name}:{line} | {message}",
            level="INFO",
            rotation="1 day",
            retention="30 days",
            compression="zip",
            encoding="utf-8"
        )
        logger.add(
            "logs/errors_{time:YYYY-MM-DD}.log",
            level="ERROR",
            rotation="1 day",
            retention="90 days",
            encoding="utf-8"
        )
    elif environment == "testing":
        # Testing: เก็บ log ใน memory สำหรับ test assertions
        logger.add(
            sys.stderr,
            level="WARNING",
            format="{level}: {message}"
        )

# ใช้งาน
env = os.getenv("ENVIRONMENT", "development")
configure_logging(env)

# ใช้ context binding
app_logger = logger.bind(app="myapp", version="1.0.0")

def process_payment(payment_id: str, amount: float) -> bool:
    log = app_logger.bind(payment_id=payment_id, amount=amount)
    log.info("Processing payment")

    try:
        if amount <= 0:
            raise ValueError(f"Invalid amount: {amount}")
        log.success("Payment processed successfully")
        return True
    except ValueError as e:
        log.error(f"Payment failed: {e}")
        return False

# ทดสอบ
process_payment("PAY-001", 500.0)
process_payment("PAY-002", -100.0)
```

---

### แบบฝึกหัดที่ 8: Complete Logging System

**โจทย์:** สร้าง complete logging system สำหรับ web application

**เฉลย:**

```python
import logging
import logging.config
import json
import uuid
import time
import sys
from datetime import datetime, timezone
from contextlib import contextmanager
from typing import Generator, Optional

class ApplicationLogger:
    """Complete logging system สำหรับ web application"""

    LOGGING_CONFIG = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'json': {
                '()': lambda: ApplicationLogger._create_json_formatter(),
            },
            'human': {
                'format': '%(asctime)s [%(levelname)-8s] %(name)s: %(message)s',
                'datefmt': '%Y-%m-%d %H:%M:%S'
            }
        },
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
                'formatter': 'human',
                'stream': 'ext://sys.stdout'
            },
            'app_file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'level': 'DEBUG',
                'formatter': 'json',
                'filename': 'webapp.log',
                'maxBytes': 50 * 1024 * 1024,
                'backupCount': 10,
                'encoding': 'utf-8'
            },
            'error_file': {
                'class': 'logging.FileHandler',
                'level': 'ERROR',
                'formatter': 'json',
                'filename': 'webapp_errors.log',
                'encoding': 'utf-8'
            }
        },
        'loggers': {
            'webapp': {
                'level': 'DEBUG',
                'handlers': ['console', 'app_file', 'error_file'],
                'propagate': False
            }
        }
    }

    @staticmethod
    def _create_json_formatter():
        class JSONFormatter(logging.Formatter):
            def format(self, record):
                return json.dumps({
                    'ts': datetime.now(timezone.utc).isoformat(),
                    'level': record.levelname,
                    'logger': record.name,
                    'msg': record.getMessage(),
                    'module': record.module,
                    'line': record.lineno,
                    **{k: v for k, v in record.__dict__.items()
                       if k.startswith('ctx_')}
                }, ensure_ascii=False)
        return JSONFormatter()

    def __init__(self):
        logging.config.dictConfig(self.LOGGING_CONFIG)
        self.logger = logging.getLogger('webapp')

    @contextmanager
    def request_context(
        self,
        method: str,
        path: str,
        user_id: Optional[int] = None
    ) -> Generator:
        request_id = str(uuid.uuid4())[:8]
        start_time = time.perf_counter()

        self.logger.info(
            f"→ {method} {path}",
            extra={
                'ctx_request_id': request_id,
                'ctx_user_id': user_id,
                'ctx_method': method,
                'ctx_path': path
            }
        )

        try:
            yield request_id
            elapsed = (time.perf_counter() - start_time) * 1000
            self.logger.info(
                f"← {method} {path} 200 ({elapsed:.1f}ms)",
                extra={'ctx_request_id': request_id, 'ctx_elapsed_ms': elapsed}
            )
        except Exception as e:
            elapsed = (time.perf_counter() - start_time) * 1000
            self.logger.error(
                f"← {method} {path} 500 ({elapsed:.1f}ms): {e}",
                extra={'ctx_request_id': request_id, 'ctx_elapsed_ms': elapsed},
                exc_info=True
            )
            raise

# ใช้งาน
app = ApplicationLogger()

with app.request_context("GET", "/api/users", user_id=42) as req_id:
    time.sleep(0.05)
    print(f"Processing request {req_id}")

with app.request_context("POST", "/api/orders", user_id=99) as req_id:
    time.sleep(0.1)
    print(f"Processing order request {req_id}")

# จำลอง error
try:
    with app.request_context("DELETE", "/api/admin/users") as req_id:
        raise PermissionError("Unauthorized access")
except PermissionError:
    print("Error handled")

print("\nLogging system ทำงานเสร็จแล้ว!")
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Logging Module** - การใช้ Python's built-in logging แทน print()
2. **Log Levels** - DEBUG, INFO, WARNING, ERROR, CRITICAL และการกรอง
3. **Handlers** - StreamHandler, FileHandler, RotatingFileHandler, TimedRotatingFileHandler
4. **Formatters** - การกำหนดรูปแบบ log messages รวมถึง custom JSON formatter
5. **Logger Hierarchy** - การจัดการ logger แบบ tree structure
6. **Configuration** - basicConfig, dictConfig, fileConfig
7. **Structured Logging** - LoggerAdapter, ContextVar, Extra fields
8. **Third-party** - Loguru, Structlog
9. **Debugging** - print(), assert, pdb, breakpoint()
10. **Profiling** - cProfile, timeit, memory_profiler, line_profiler

### Best Practices

- ใช้ `logging.getLogger(__name__)` ในแต่ละ module
- ตั้งค่าด้วย `dictConfig` สำหรับโปรเจกต์ใหญ่
- ใช้ `%s` formatting แทน f-string ใน log messages
- เพิ่ม context ด้วย `extra` parameter หรือ `LoggerAdapter`
- ใช้ `RotatingFileHandler` ป้องกัน disk เต็ม
- แยก error log ออกจาก application log

---

*ถัดไป: Part 37 - Concurrency: Threading*
