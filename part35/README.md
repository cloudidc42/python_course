# Part 35: Testing with pytest

## บทนำ

pytest เป็น testing framework ที่ทันสมัยและทรงพลังที่สุดสำหรับ Python มีผู้นิยมใช้มากกว่า unittest เพราะเขียน tests ได้ง่ายกว่า มี features ครบครัน และมี plugin ecosystem ที่ยอดเยี่ยม

---

## 1. pytest vs unittest

| Feature | unittest | pytest |
|---------|----------|--------|
| Test functions | ต้องใช้ class | ไม่ต้องใช้ class |
| Assertions | self.assertEqual() | assert (Python native) |
| Fixtures | setUp/tearDown | @pytest.fixture |
| Parameterize | subTest | @pytest.mark.parametrize |
| Plugins | น้อย | 1000+ plugins |
| Output | พื้นฐาน | ละเอียด, สวยงาม |
| Compatibility | ใช้กับ unittest tests ได้ | รัน unittest tests ได้ด้วย |

```bash
# ติดตั้ง pytest
pip install pytest
pytest --version

# ติดตั้ง plugins ที่ใช้บ่อย
pip install pytest-cov pytest-mock pytest-xdist pytest-asyncio
```

---

## 2. Test Functions (No Classes Needed!)

```python
# ตัวอย่างที่ 1: pytest แบบพื้นฐาน - ไม่ต้องมี class
# file: test_basics.py

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def is_palindrome(s):
    s = s.lower().replace(' ', '')
    return s == s[::-1]

# Test functions - ขึ้นต้นด้วย test_
def test_add():
    assert add(3, 4) == 7

def test_add_negative():
    assert add(-3, -4) == -7

def test_subtract():
    assert subtract(10, 3) == 7

def test_palindrome_racecar():
    assert is_palindrome("racecar") is True

def test_palindrome_hello():
    assert is_palindrome("hello") is False

def test_palindrome_with_spaces():
    assert is_palindrome("race car") is True
```

```bash
# รัน tests
pytest test_basics.py                   # รัน file เดียว
pytest test_basics.py -v               # verbose
pytest test_basics.py::test_add        # รัน test เดียว
pytest -k "palindrome"                 # รัน tests ที่ชื่อ contain "palindrome"
pytest -k "add and not negative"       # boolean expressions
pytest --tb=short                      # traceback แบบสั้น
pytest -x                              # stop ทันทีเมื่อ fail
pytest --lf                            # รัน tests ที่ fail ล่าสุด
```

---

## 3. pytest Assertions

```python
# ตัวอย่างที่ 2: pytest assertions ที่ดีกว่า unittest
# pytest ให้ข้อมูลละเอียดเมื่อ fail

def test_assertion_equality():
    # ถ้า fail จะแสดง actual vs expected
    expected = [1, 2, 3, 4]
    actual = [1, 2, 3, 4]
    assert actual == expected

def test_assertion_in():
    fruits = ['apple', 'banana', 'cherry']
    assert 'banana' in fruits
    assert 'dragon' not in fruits

def test_assertion_isinstance():
    assert isinstance(42, int)
    assert isinstance("hello", (str, bytes))

def test_assertion_with_message():
    value = 42
    # ใส่ message เมื่อ fail
    assert value > 0, f"Value should be positive, got {value}"

def test_assertion_approximately():
    """pytest มี approx() สำหรับ floating point"""
    import pytest
    assert 0.1 + 0.2 == pytest.approx(0.3)
    assert 1.0 == pytest.approx(1.0000001, rel=1e-5)
    assert [0.1, 0.2] == pytest.approx([0.1, 0.2])

def test_assertion_exceptions():
    """ตรวจสอบ exceptions"""
    import pytest
    
    with pytest.raises(ValueError):
        int("not a number")
    
    with pytest.raises(ZeroDivisionError):
        1 / 0
    
    # ตรวจสอบ exception message
    with pytest.raises(ValueError, match=r"must be positive"):
        raise ValueError("value must be positive")
    
    # เก็บ exception info
    with pytest.raises(KeyError) as exc_info:
        {}['missing']
    
    assert 'missing' in str(exc_info.value)
```

```python
# ตัวอย่างที่ 3: Assertion rewriting - pytest แสดงค่าจริงเมื่อ fail
def test_complex_assertion():
    """pytest introspects expressions เมื่อ fail"""
    a = [1, 2, 3]
    b = [1, 2, 4]  # ต่างกันที่ index 2
    
    # ถ้า fail, pytest จะแสดง:
    # assert [1, 2, 3] == [1, 2, 4]
    # where [1, 2, 3] = a
    #   and [1, 2, 4] = b
    assert a == b  # จะ fail พร้อม detail

def test_dict_assertion():
    result = {'name': 'Alice', 'age': 25, 'city': 'Bangkok'}
    expected = {'name': 'Alice', 'age': 25, 'city': 'Bangkok'}
    assert result == expected
```

---

## 4. Fixtures (@pytest.fixture)

Fixtures คือ mechanism ที่ทรงพลังที่สุดของ pytest สำหรับ setup และ teardown

```python
import pytest
import tempfile
import os

# ตัวอย่างที่ 4: Basic fixtures
@pytest.fixture
def sample_data():
    """Fixture ที่ส่งคืนข้อมูลทดสอบ"""
    return {
        'users': ['Alice', 'Bob', 'Charlie'],
        'count': 3
    }

@pytest.fixture
def empty_list():
    """Fixture ที่ส่งคืน empty list"""
    return []

# ใช้ fixtures ใน test functions
def test_sample_data_count(sample_data):
    assert sample_data['count'] == 3

def test_sample_data_users(sample_data):
    assert 'Alice' in sample_data['users']

def test_empty_list(empty_list):
    assert len(empty_list) == 0
    empty_list.append(1)
    assert len(empty_list) == 1

# แต่ละ test ได้ fresh fixture ใหม่!
def test_still_empty(empty_list):
    assert len(empty_list) == 0  # ไม่ได้รับค่าจาก test ก่อน
```

```python
import pytest
import tempfile
import os

# ตัวอย่างที่ 5: Fixtures พร้อม setup/teardown
@pytest.fixture
def temp_file():
    """Fixture ที่สร้างและลบ temp file"""
    # Setup
    fd, filepath = tempfile.mkstemp()
    os.close(fd)
    
    with open(filepath, 'w') as f:
        f.write("test content")
    
    yield filepath  # ส่ง filepath ให้ test
    
    # Teardown (รันหลัง test เสมอ แม้ test fail)
    if os.path.exists(filepath):
        os.unlink(filepath)

@pytest.fixture
def temp_dir():
    """Fixture สำหรับ temporary directory"""
    dirpath = tempfile.mkdtemp()
    yield dirpath
    
    import shutil
    shutil.rmtree(dirpath, ignore_errors=True)

def test_file_exists(temp_file):
    assert os.path.exists(temp_file)

def test_file_content(temp_file):
    with open(temp_file) as f:
        content = f.read()
    assert content == "test content"

def test_write_to_temp_dir(temp_dir):
    filepath = os.path.join(temp_dir, 'new_file.txt')
    with open(filepath, 'w') as f:
        f.write("new content")
    assert os.path.exists(filepath)
```

```python
import pytest

# ตัวอย่างที่ 6: Fixture scope
@pytest.fixture(scope="function")   # default: สร้างใหม่ทุก test function
def function_scoped():
    print("\nCreating function fixture")
    yield "function"
    print("\nDestroying function fixture")

@pytest.fixture(scope="class")      # สร้างครั้งเดียวต่อ class
def class_scoped():
    print("\nCreating class fixture")
    yield "class"
    print("\nDestroying class fixture")

@pytest.fixture(scope="module")     # สร้างครั้งเดียวต่อ module (file)
def module_scoped():
    print("\nCreating module fixture")
    data = {'connection': 'fake_db_connection', 'count': 0}
    yield data
    print(f"\nDestroying module fixture (used {data['count']} times)")

@pytest.fixture(scope="session")    # สร้างครั้งเดียวตลอด test session
def session_scoped():
    print("\nCreating session fixture")
    yield "session"
    print("\nDestroying session fixture")

def test_scope_demo(function_scoped, module_scoped, session_scoped):
    module_scoped['count'] += 1
    assert function_scoped == "function"
    assert session_scoped == "session"
```

```python
import pytest

# ตัวอย่างที่ 7: Fixtures ที่ใช้ fixtures อื่น
@pytest.fixture
def database_url():
    return "sqlite:///:memory:"

@pytest.fixture
def db_connection(database_url):
    """Fixture ที่ใช้ database_url fixture"""
    import sqlite3
    conn = sqlite3.connect(':memory:')
    conn.execute("CREATE TABLE test (id INTEGER PRIMARY KEY, value TEXT)")
    yield conn
    conn.close()

@pytest.fixture
def populated_db(db_connection):
    """Fixture ที่ใช้ db_connection"""
    db_connection.execute("INSERT INTO test (value) VALUES (?)", ("test_value",))
    db_connection.commit()
    return db_connection

def test_db_empty(db_connection):
    cursor = db_connection.execute("SELECT COUNT(*) FROM test")
    assert cursor.fetchone()[0] == 0

def test_db_populated(populated_db):
    cursor = populated_db.execute("SELECT COUNT(*) FROM test")
    assert cursor.fetchone()[0] == 1
```

---

## 5. conftest.py

conftest.py เป็นไฟล์พิเศษที่ pytest โหลดอัตโนมัติ ใส่ fixtures ที่ใช้ร่วมกันได้ที่นี่

```python
# conftest.py - fixtures ที่ available สำหรับทุก tests ในโฟลเดอร์
import pytest
import sqlite3
import tempfile
import os

# ตัวอย่างที่ 8: conftest.py structure
@pytest.fixture(scope="session")
def test_database():
    """Database fixture ที่ใช้ร่วมกันทั้ง session"""
    db_path = tempfile.mktemp(suffix='.db')
    conn = sqlite3.connect(db_path)
    
    conn.execute('''CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        username TEXT UNIQUE NOT NULL,
        email TEXT NOT NULL
    )''')
    conn.commit()
    
    yield conn
    
    conn.close()
    os.unlink(db_path)

@pytest.fixture
def clean_database(test_database):
    """Reset database สำหรับแต่ละ test"""
    test_database.execute("DELETE FROM users")
    test_database.commit()
    yield test_database
    # cleanup หลัง test

@pytest.fixture
def sample_users():
    return [
        {'username': 'alice', 'email': 'alice@test.com'},
        {'username': 'bob', 'email': 'bob@test.com'},
        {'username': 'charlie', 'email': 'charlie@test.com'},
    ]

# Hooks
def pytest_configure(config):
    """เรียกตอน pytest เริ่มต้น"""
    config.addinivalue_line("markers", "slow: mark test as slow")
    config.addinivalue_line("markers", "integration: mark as integration test")

def pytest_collection_modifyitems(items):
    """Modify collected tests"""
    for item in items:
        if "integration" in item.nodeid:
            item.add_marker(pytest.mark.slow)
```

---

## 6. Parametrize (@pytest.mark.parametrize)

```python
import pytest

# ตัวอย่างที่ 9: parametrize พื้นฐาน
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

@pytest.mark.parametrize("n,expected", [
    (2, True),
    (3, True),
    (4, False),
    (5, True),
    (9, False),
    (11, True),
    (1, False),
    (0, False),
    (-1, False),
])
def test_is_prime(n, expected):
    assert is_prime(n) == expected
```

```python
import pytest

# ตัวอย่างที่ 10: parametrize หลาย arguments
def calculate_bmi(weight_kg, height_m):
    return weight_kg / (height_m ** 2)

def bmi_category(bmi):
    if bmi < 18.5:
        return "Underweight"
    elif bmi < 25:
        return "Normal"
    elif bmi < 30:
        return "Overweight"
    else:
        return "Obese"

@pytest.mark.parametrize("weight,height,expected_category", [
    (50, 1.70, "Underweight"),
    (70, 1.75, "Normal"),
    (85, 1.70, "Overweight"),
    (100, 1.70, "Obese"),
])
def test_bmi_category(weight, height, expected_category):
    bmi = calculate_bmi(weight, height)
    assert bmi_category(bmi) == expected_category
```

```python
import pytest

# ตัวอย่างที่ 11: parametrize ซ้อนกัน (combinatorial testing)
@pytest.mark.parametrize("a", [1, 2, 3])
@pytest.mark.parametrize("b", [10, 20])
def test_multiplication(a, b):
    """จะรัน 3 * 2 = 6 test cases"""
    assert a * b == b * a  # commutative property

# ตัวอย่างที่ 12: parametrize พร้อม pytest.param สำหรับ ids และ marks
@pytest.mark.parametrize("text,expected", [
    pytest.param("hello world", 2, id="two_words"),
    pytest.param("", 0, id="empty_string"),
    pytest.param("  single  ", 1, id="single_with_spaces"),
    pytest.param("one two three four", 4, id="four_words", 
                 marks=pytest.mark.slow),
])
def test_word_count(text, expected):
    count = len(text.split()) if text.strip() else 0
    assert count == expected
```

---

## 7. Marks (@pytest.mark)

```python
import pytest
import sys

# ตัวอย่างที่ 13: Built-in marks

@pytest.mark.skip(reason="Feature not implemented yet")
def test_future_feature():
    pass

@pytest.mark.skipif(
    sys.platform == "win32",
    reason="Does not run on Windows"
)
def test_unix_only():
    import os
    assert os.path.exists('/etc')

@pytest.mark.xfail(reason="Known bug #123")
def test_known_bug():
    assert 1 == 2  # Expected to fail

@pytest.mark.xfail(strict=True, reason="Must fail")
def test_must_fail():
    assert 1 == 2

# Custom marks (ต้อง register ใน conftest.py หรือ pytest.ini)
@pytest.mark.slow
def test_slow_operation():
    import time
    time.sleep(0.1)
    assert True

@pytest.mark.integration
def test_with_database():
    assert True

@pytest.mark.smoke
def test_critical_path():
    assert True
```

```python
# ตัวอย่างที่ 14: รัน tests ด้วย marks
# pytest -m "slow"           - รัน tests ที่ mark เป็น slow
# pytest -m "not slow"       - รัน tests ที่ไม่ใช่ slow
# pytest -m "smoke or integration"
# pytest -m "integration and not slow"
```

---

## 8. pytest-cov - Code Coverage

```bash
# ติดตั้ง
pip install pytest-cov

# รัน tests พร้อม coverage
pytest --cov=src                              # coverage สำหรับ src/
pytest --cov=mypackage --cov-report=html      # HTML report
pytest --cov=. --cov-report=term-missing      # แสดง lines ที่ไม่มี coverage
pytest --cov-fail-under=80                    # fail ถ้า coverage < 80%

# สร้าง .coveragerc
cat > .coveragerc << 'EOF'
[run]
source = src
omit = 
    */tests/*
    */migrations/*
    setup.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError
    if TYPE_CHECKING:

[html]
directory = coverage_html
EOF
```

```python
# ตัวอย่างที่ 15: Code ที่มี coverage 100%
def classify_number(n):
    """จัดประเภทตัวเลข"""
    if n > 0:
        return "positive"
    elif n < 0:
        return "negative"
    else:
        return "zero"

def test_positive():
    assert classify_number(5) == "positive"

def test_negative():
    assert classify_number(-3) == "negative"

def test_zero():
    assert classify_number(0) == "zero"

# ทั้ง 3 branches ถูก test -> coverage 100%

# pragma: no cover สำหรับโค้ดที่ไม่ต้อง test
def debug_only():  # pragma: no cover
    import pdb
    pdb.set_trace()
```

---

## 9. Mocking กับ pytest-mock

```bash
pip install pytest-mock
```

```python
import pytest
from unittest.mock import Mock, patch, MagicMock

# ตัวอย่างที่ 16: pytest-mock ให้ mocker fixture
class UserService:
    def __init__(self, db, email_service):
        self.db = db
        self.email_service = email_service
    
    def create_user(self, username, email):
        # ตรวจสอบ duplicate
        existing = self.db.find_user(username)
        if existing:
            raise ValueError(f"Username '{username}' already taken")
        
        # สร้าง user
        user_id = self.db.create_user(username, email)
        
        # ส่ง welcome email
        self.email_service.send_welcome_email(email, username)
        
        return user_id

def test_create_user_success(mocker):
    """ใช้ mocker fixture จาก pytest-mock"""
    # สร้าง mock objects
    mock_db = mocker.MagicMock()
    mock_email = mocker.MagicMock()
    
    # กำหนด behavior
    mock_db.find_user.return_value = None  # ไม่มี user เดิม
    mock_db.create_user.return_value = 123
    
    service = UserService(mock_db, mock_email)
    result = service.create_user("alice", "alice@test.com")
    
    # ตรวจสอบ result
    assert result == 123
    
    # ตรวจสอบว่าถูกเรียกถูกต้อง
    mock_db.find_user.assert_called_once_with("alice")
    mock_db.create_user.assert_called_once_with("alice", "alice@test.com")
    mock_email.send_welcome_email.assert_called_once_with("alice@test.com", "alice")

def test_create_user_duplicate(mocker):
    """ทดสอบกรณี username ซ้ำ"""
    mock_db = mocker.MagicMock()
    mock_email = mocker.MagicMock()
    
    # กำหนดให้มี user เดิมอยู่แล้ว
    mock_db.find_user.return_value = {'id': 1, 'username': 'alice'}
    
    service = UserService(mock_db, mock_email)
    
    with pytest.raises(ValueError, match="already taken"):
        service.create_user("alice", "alice2@test.com")
    
    # ตรวจสอบว่าไม่ได้สร้าง user และไม่ส่ง email
    mock_db.create_user.assert_not_called()
    mock_email.send_welcome_email.assert_not_called()
```

```python
import pytest

# ตัวอย่างที่ 17: mocker.patch
def get_current_user():
    """ดึง user ที่ login อยู่"""
    import os
    return os.environ.get('CURRENT_USER', 'anonymous')

def greet_user():
    user = get_current_user()
    return f"Hello, {user}!"

def test_greet_user(mocker):
    mocker.patch('__main__.get_current_user', return_value='Alice')
    result = greet_user()
    assert result == "Hello, Alice!"

def test_mocker_spy(mocker):
    """Spy - track calls โดยไม่เปลี่ยน behavior"""
    import os
    spy = mocker.spy(os.path, 'exists')
    
    os.path.exists('/tmp')
    
    # ตรวจสอบว่าถูกเรียก
    spy.assert_called_once_with('/tmp')
```

---

## 10. Test Organization

```
# โครงสร้าง tests ที่ดี
project/
├── src/
│   └── myapp/
│       ├── __init__.py
│       ├── models.py
│       ├── services.py
│       └── utils.py
├── tests/
│   ├── conftest.py          # shared fixtures
│   ├── unit/
│   │   ├── conftest.py      # unit-specific fixtures
│   │   ├── test_models.py
│   │   ├── test_services.py
│   │   └── test_utils.py
│   ├── integration/
│   │   ├── conftest.py
│   │   └── test_database.py
│   └── e2e/
│       └── test_api.py
└── pytest.ini
```

```ini
# pytest.ini หรือ pyproject.toml [tool.pytest.ini_options]
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --tb=short
    --strict-markers
    --cov=src
    --cov-report=term-missing
    --cov-fail-under=80
markers =
    slow: mark test as slow
    integration: mark as integration test
    smoke: mark as smoke test
    unit: mark as unit test
filterwarnings =
    error::DeprecationWarning
```

```python
# ตัวอย่างที่ 18: การจัดกลุ่ม tests ใน class
class TestUserCreation:
    """กลุ่ม tests สำหรับ user creation"""
    
    def test_valid_user(self):
        user = {'username': 'alice', 'email': 'alice@test.com'}
        assert user['username'] == 'alice'
    
    def test_invalid_username(self):
        with pytest.raises(ValueError):
            raise ValueError("Invalid username")
    
    class TestWithDatabase:
        """Nested class สำหรับ integration tests"""
        
        def test_save_to_db(self):
            assert True

class TestUserDeletion:
    """กลุ่ม tests สำหรับ user deletion"""
    
    def test_delete_existing(self):
        assert True
```

---

## 11. pytest Plugins

```bash
# ตัวอย่างที่ 19: Popular pytest plugins

# pytest-xdist: รัน tests แบบ parallel
pip install pytest-xdist
pytest -n auto          # ใช้ CPU cores ทั้งหมด
pytest -n 4             # ใช้ 4 workers

# pytest-asyncio: test async code
pip install pytest-asyncio
# ในไฟล์:
# @pytest.mark.asyncio
# async def test_async():
#     result = await some_async_func()
#     assert result == expected

# pytest-timeout: timeout สำหรับ tests
pip install pytest-timeout
pytest --timeout=10     # 10 วินาทีต่อ test
# หรือ: @pytest.mark.timeout(5)

# pytest-benchmark: benchmark tests
pip install pytest-benchmark
# ในไฟล์:
# def test_speed(benchmark):
#     result = benchmark(my_function, arg1, arg2)
#     assert result == expected

# pytest-html: HTML report
pip install pytest-html
pytest --html=report.html --self-contained-html

# pytest-ordering: กำหนดลำดับ tests
pip install pytest-ordering
# @pytest.mark.first
# @pytest.mark.last
# @pytest.mark.order(2)

# Faker: generate fake data
pip install faker
# @pytest.fixture
# def fake_user(faker):
#     return {'name': faker.name(), 'email': faker.email()}
```

```python
import pytest
import asyncio

# ตัวอย่างที่ 20: pytest-asyncio
# pip install pytest-asyncio

async def fetch_data(url: str) -> dict:
    """Async function ที่ต้อง test"""
    await asyncio.sleep(0.01)  # simulate network
    return {'url': url, 'data': 'response'}

@pytest.mark.asyncio
async def test_fetch_data():
    result = await fetch_data("https://example.com")
    assert result['url'] == "https://example.com"
    assert 'data' in result

@pytest.fixture
async def async_client():
    """Async fixture"""
    # setup
    client = {'initialized': True}
    yield client
    # teardown

@pytest.mark.asyncio
async def test_with_async_fixture(async_client):
    assert async_client['initialized'] is True
```

---

## 12. TDD Workflow - Test-Driven Development

```python
# ตัวอย่างที่ 21: TDD step by step

# Step 1: เขียน test ก่อน (RED - test fails)
import pytest

def test_calculate_tax():
    """Test ก่อนมี implementation"""
    # Thailand VAT 7%
    assert calculate_tax(100) == pytest.approx(7.0)
    assert calculate_tax(1000) == pytest.approx(70.0)
    assert calculate_tax(0) == pytest.approx(0.0)

# ตอนนี้ยังไม่มี calculate_tax - จะ fail!

# Step 2: เขียน implementation น้อยที่สุดให้ pass (GREEN)
def calculate_tax(amount: float, rate: float = 0.07) -> float:
    return amount * rate

# Step 3: Refactor (REFACTOR)
def calculate_tax_v2(amount: float, rate: float = 0.07) -> float:
    """คำนวณภาษี"""
    if amount < 0:
        raise ValueError("Amount cannot be negative")
    if not 0 <= rate <= 1:
        raise ValueError("Rate must be between 0 and 1")
    return round(amount * rate, 2)

# Step 4: เพิ่ม tests ใหม่ (RED)
def test_calculate_tax_negative():
    with pytest.raises(ValueError, match="negative"):
        calculate_tax_v2(-100)

def test_calculate_tax_invalid_rate():
    with pytest.raises(ValueError):
        calculate_tax_v2(100, rate=1.5)

def test_calculate_tax_rounding():
    result = calculate_tax_v2(99.99)
    assert result == 7.0  # 99.99 * 0.07 = 6.9993 -> round to 7.0

# รัน: pytest test_tdd.py -v
```

---

## 13. BDD กับ pytest-bdd

```bash
pip install pytest-bdd
```

```gherkin
# ตัวอย่างที่ 22: Feature file (Gherkin syntax)
# features/shopping_cart.feature

Feature: Shopping Cart
  As a customer
  I want to add products to my cart
  So that I can purchase them

  Scenario: Add product to empty cart
    Given an empty shopping cart
    When I add "iPhone 15" with price 35000 and quantity 1
    Then the cart should have 1 item
    And the total should be 35000

  Scenario: Apply discount
    Given a cart with "MacBook" at 89000
    When I apply a 10% discount
    Then the discounted total should be 80100

  Scenario Outline: Calculate multiple items
    Given an empty shopping cart
    When I add <quantity> items at price <price>
    Then the total should be <total>

    Examples:
      | quantity | price | total  |
      | 2        | 100   | 200    |
      | 3        | 50    | 150    |
      | 1        | 1000  | 1000   |
```

```python
# test_shopping_cart_bdd.py
import pytest
from pytest_bdd import given, when, then, scenarios, parsers

scenarios('../features/shopping_cart.feature')

# Implementation classes
class Cart:
    def __init__(self):
        self.items = []
        self.discount = 0
    
    def add_item(self, name, price, quantity=1):
        self.items.append({'name': name, 'price': price, 'quantity': quantity})
    
    def get_total(self):
        return sum(item['price'] * item['quantity'] for item in self.items)
    
    def apply_discount(self, percent):
        self.discount = percent
    
    def get_discounted_total(self):
        return self.get_total() * (1 - self.discount / 100)

# Step definitions
@pytest.fixture
def cart():
    return Cart()

@given("an empty shopping cart")
def empty_cart(cart):
    assert len(cart.items) == 0
    return cart

@when(parsers.parse('I add "{product}" with price {price:d} and quantity {qty:d}'))
def add_product(cart, product, price, qty):
    cart.add_item(product, price, qty)

@then(parsers.parse("the cart should have {count:d} item"))
def check_item_count(cart, count):
    assert len(cart.items) == count

@then(parsers.parse("the total should be {total:d}"))
def check_total(cart, total):
    assert cart.get_total() == total
```

---

## 14. ตัวอย่างโปรแกรมจริง - Complete Test Suite สำหรับ Web API

```python
# app/api.py - code ที่ต้องการ test
from typing import List, Optional, Dict
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    id: int
    username: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True

class UserAPI:
    def __init__(self):
        self._users: Dict[int, User] = {}
        self._next_id = 1
    
    def create_user(self, username: str, email: str) -> User:
        if not username or not username.strip():
            raise ValueError("Username cannot be empty")
        if '@' not in email:
            raise ValueError("Invalid email format")
        
        # Check duplicate
        if any(u.username == username for u in self._users.values()):
            raise ValueError(f"Username '{username}' already exists")
        if any(u.email == email for u in self._users.values()):
            raise ValueError(f"Email '{email}' already registered")
        
        user = User(id=self._next_id, username=username, email=email)
        self._users[self._next_id] = user
        self._next_id += 1
        return user
    
    def get_user(self, user_id: int) -> Optional[User]:
        return self._users.get(user_id)
    
    def list_users(self, active_only: bool = False) -> List[User]:
        users = list(self._users.values())
        if active_only:
            users = [u for u in users if u.is_active]
        return users
    
    def update_user(self, user_id: int, **kwargs) -> Optional[User]:
        user = self._users.get(user_id)
        if not user:
            return None
        
        allowed = {'username', 'email', 'is_active'}
        for key, value in kwargs.items():
            if key in allowed:
                setattr(user, key, value)
        
        return user
    
    def delete_user(self, user_id: int) -> bool:
        if user_id in self._users:
            del self._users[user_id]
            return True
        return False
    
    def search_users(self, query: str) -> List[User]:
        query = query.lower()
        return [
            u for u in self._users.values()
            if query in u.username.lower() or query in u.email.lower()
        ]
```

```python
# tests/test_user_api.py - Complete test suite
import pytest
from datetime import datetime

# Import from app (adjust path as needed)
# from app.api import UserAPI, User

# สมมติว่า UserAPI อยู่ในไฟล์เดียวกัน
class UserAPI:
    def __init__(self):
        self._users = {}
        self._next_id = 1
    
    def create_user(self, username, email):
        if not username or not username.strip():
            raise ValueError("Username cannot be empty")
        if '@' not in email:
            raise ValueError("Invalid email format")
        if any(u['username'] == username for u in self._users.values()):
            raise ValueError(f"Username '{username}' already exists")
        if any(u['email'] == email for u in self._users.values()):
            raise ValueError(f"Email '{email}' already registered")
        user = {'id': self._next_id, 'username': username, 'email': email, 'is_active': True}
        self._users[self._next_id] = user
        self._next_id += 1
        return user
    
    def get_user(self, user_id):
        return self._users.get(user_id)
    
    def list_users(self, active_only=False):
        users = list(self._users.values())
        if active_only:
            users = [u for u in users if u['is_active']]
        return users
    
    def update_user(self, user_id, **kwargs):
        user = self._users.get(user_id)
        if not user:
            return None
        for k, v in kwargs.items():
            if k in ('username', 'email', 'is_active'):
                user[k] = v
        return user
    
    def delete_user(self, user_id):
        if user_id in self._users:
            del self._users[user_id]
            return True
        return False
    
    def search_users(self, query):
        q = query.lower()
        return [u for u in self._users.values()
                if q in u['username'].lower() or q in u['email'].lower()]

# Fixtures
@pytest.fixture
def api():
    return UserAPI()

@pytest.fixture
def api_with_users(api):
    """API ที่มี users แล้ว"""
    api.create_user("alice", "alice@test.com")
    api.create_user("bob", "bob@test.com")
    api.create_user("charlie", "charlie@test.com")
    return api

# ตัวอย่างที่ 23: Complete test suite

class TestCreateUser:
    """Tests สำหรับ create_user"""
    
    def test_create_valid_user(self, api):
        user = api.create_user("alice", "alice@test.com")
        
        assert user['id'] == 1
        assert user['username'] == "alice"
        assert user['email'] == "alice@test.com"
        assert user['is_active'] is True
    
    def test_create_multiple_users(self, api):
        user1 = api.create_user("alice", "alice@test.com")
        user2 = api.create_user("bob", "bob@test.com")
        
        assert user1['id'] != user2['id']
        assert len(api.list_users()) == 2
    
    @pytest.mark.parametrize("bad_username", ["", "  ", None])
    def test_create_empty_username_fails(self, api, bad_username):
        with pytest.raises((ValueError, TypeError)):
            api.create_user(bad_username, "test@test.com")
    
    @pytest.mark.parametrize("bad_email", [
        "notanemail",
        "missing-at-sign",
        "",
    ])
    def test_create_invalid_email_fails(self, api, bad_email):
        with pytest.raises(ValueError, match="Invalid email"):
            api.create_user("testuser", bad_email)
    
    def test_duplicate_username_fails(self, api_with_users):
        with pytest.raises(ValueError, match="already exists"):
            api_with_users.create_user("alice", "newalice@test.com")
    
    def test_duplicate_email_fails(self, api_with_users):
        with pytest.raises(ValueError, match="already registered"):
            api_with_users.create_user("newalice", "alice@test.com")

class TestGetUser:
    """Tests สำหรับ get_user"""
    
    def test_get_existing_user(self, api_with_users):
        user = api_with_users.get_user(1)
        assert user is not None
        assert user['username'] == "alice"
    
    def test_get_nonexistent_user(self, api):
        user = api.get_user(99999)
        assert user is None
    
    def test_get_returns_correct_user(self, api_with_users):
        user2 = api_with_users.get_user(2)
        assert user2['username'] == "bob"

class TestListUsers:
    """Tests สำหรับ list_users"""
    
    def test_list_all_users(self, api_with_users):
        users = api_with_users.list_users()
        assert len(users) == 3
    
    def test_list_empty(self, api):
        users = api.list_users()
        assert users == []
    
    def test_list_active_only(self, api_with_users):
        api_with_users.update_user(1, is_active=False)
        
        all_users = api_with_users.list_users()
        active_users = api_with_users.list_users(active_only=True)
        
        assert len(all_users) == 3
        assert len(active_users) == 2

class TestUpdateUser:
    """Tests สำหรับ update_user"""
    
    def test_update_username(self, api_with_users):
        updated = api_with_users.update_user(1, username="alice_updated")
        assert updated['username'] == "alice_updated"
    
    def test_update_is_active(self, api_with_users):
        updated = api_with_users.update_user(1, is_active=False)
        assert updated['is_active'] is False
    
    def test_update_nonexistent_user(self, api):
        result = api.update_user(99999, username="ghost")
        assert result is None

class TestDeleteUser:
    """Tests สำหรับ delete_user"""
    
    def test_delete_existing_user(self, api_with_users):
        result = api_with_users.delete_user(1)
        assert result is True
        assert api_with_users.get_user(1) is None
    
    def test_delete_nonexistent_user(self, api):
        result = api.delete_user(99999)
        assert result is False
    
    def test_delete_reduces_count(self, api_with_users):
        initial_count = len(api_with_users.list_users())
        api_with_users.delete_user(1)
        assert len(api_with_users.list_users()) == initial_count - 1

class TestSearchUsers:
    """Tests สำหรับ search_users"""
    
    def test_search_by_username(self, api_with_users):
        results = api_with_users.search_users("alice")
        assert len(results) == 1
        assert results[0]['username'] == "alice"
    
    def test_search_by_email(self, api_with_users):
        results = api_with_users.search_users("bob@test")
        assert len(results) == 1
        assert results[0]['username'] == "bob"
    
    def test_search_case_insensitive(self, api_with_users):
        results = api_with_users.search_users("ALICE")
        assert len(results) == 1
    
    def test_search_no_results(self, api_with_users):
        results = api_with_users.search_users("nonexistent_xyz")
        assert results == []
    
    def test_search_partial_match(self, api_with_users):
        results = api_with_users.search_users("test.com")
        assert len(results) == 3

# รัน tests
if __name__ == '__main__':
    pytest.main([__file__, '-v', '--tb=short'])
```

---

## 15. Database Tests

```python
import pytest
import sqlite3
import tempfile
import os

# ตัวอย่างที่ 24: Database tests พร้อม fixtures
@pytest.fixture(scope="module")
def db_path():
    """สร้าง temp database path"""
    fd, path = tempfile.mkstemp(suffix='.db')
    os.close(fd)
    yield path
    os.unlink(path)

@pytest.fixture(scope="module")
def db_schema(db_path):
    """สร้าง database schema ครั้งเดียวต่อ module"""
    conn = sqlite3.connect(db_path)
    conn.execute('''
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            price REAL NOT NULL,
            stock INTEGER DEFAULT 0
        )
    ''')
    conn.commit()
    conn.close()
    return db_path

@pytest.fixture
def db(db_schema):
    """สร้าง fresh connection พร้อม cleanup"""
    conn = sqlite3.connect(db_schema)
    conn.row_factory = sqlite3.Row
    
    yield conn
    
    # Cleanup: ลบข้อมูลทั้งหมด
    conn.execute("DELETE FROM products")
    conn.commit()
    conn.close()

class ProductDB:
    def __init__(self, conn):
        self.conn = conn
    
    def add_product(self, name, price, stock=0):
        cursor = self.conn.execute(
            "INSERT INTO products (name, price, stock) VALUES (?, ?, ?)",
            (name, price, stock)
        )
        self.conn.commit()
        return cursor.lastrowid
    
    def get_product(self, product_id):
        row = self.conn.execute(
            "SELECT * FROM products WHERE id = ?", (product_id,)
        ).fetchone()
        return dict(row) if row else None
    
    def update_stock(self, product_id, delta):
        self.conn.execute(
            "UPDATE products SET stock = stock + ? WHERE id = ?",
            (delta, product_id)
        )
        self.conn.commit()
    
    def get_low_stock(self, threshold=10):
        rows = self.conn.execute(
            "SELECT * FROM products WHERE stock <= ?", (threshold,)
        ).fetchall()
        return [dict(row) for row in rows]

class TestProductDB:
    
    def test_add_product(self, db):
        product_db = ProductDB(db)
        product_id = product_db.add_product("iPhone", 35000, stock=10)
        
        assert product_id == 1
        
        product = product_db.get_product(product_id)
        assert product is not None
        assert product['name'] == "iPhone"
        assert product['price'] == 35000
        assert product['stock'] == 10
    
    def test_update_stock(self, db):
        product_db = ProductDB(db)
        product_id = product_db.add_product("iPad", 32000, stock=5)
        
        product_db.update_stock(product_id, 10)
        
        product = product_db.get_product(product_id)
        assert product['stock'] == 15
    
    def test_low_stock_alert(self, db):
        product_db = ProductDB(db)
        product_db.add_product("Low Stock Item", 1000, stock=3)
        product_db.add_product("Normal Stock Item", 2000, stock=50)
        
        low_stock = product_db.get_low_stock(threshold=10)
        
        assert len(low_stock) == 1
        assert low_stock[0]['name'] == "Low Stock Item"
    
    def test_nonexistent_product(self, db):
        product_db = ProductDB(db)
        product = product_db.get_product(99999)
        assert product is None

if __name__ == '__main__':
    pytest.main([__file__, '-v'])
```

---

## 16. Advanced pytest Features

```python
import pytest

# ตัวอย่างที่ 25: pytest.approx สำหรับ floating point
def test_floating_point():
    assert 0.1 + 0.2 == pytest.approx(0.3)
    assert 1.0 / 3.0 == pytest.approx(0.333, rel=1e-3)
    
    # สำหรับ list
    result = [0.1 + 0.1 + 0.1, 0.2 + 0.2]
    assert result == pytest.approx([0.3, 0.4])
    
    # สำหรับ dict
    assert {'x': 0.1 + 0.2} == pytest.approx({'x': 0.3})

# ตัวอย่างที่ 26: capture output
def test_stdout_capture(capsys):
    """ตรวจสอบ stdout/stderr output"""
    print("Hello from print")
    
    captured = capsys.readouterr()
    
    assert "Hello from print" in captured.out
    assert captured.err == ""

def test_logging_capture(caplog):
    """ตรวจสอบ log messages"""
    import logging
    logger = logging.getLogger(__name__)
    
    logger.warning("Test warning message")
    
    assert "Test warning message" in caplog.text
    assert caplog.records[0].levelname == "WARNING"

# ตัวอย่างที่ 27: tmp_path fixture (built-in)
def test_with_tmp_path(tmp_path):
    """tmp_path เป็น built-in fixture ของ pytest"""
    # สร้างไฟล์ใน temp directory
    test_file = tmp_path / "test.txt"
    test_file.write_text("Hello, World!")
    
    assert test_file.exists()
    assert test_file.read_text() == "Hello, World!"
    
    # สร้าง subdirectory
    sub_dir = tmp_path / "subdir"
    sub_dir.mkdir()
    
    assert sub_dir.is_dir()
```

```python
import pytest

# ตัวอย่างที่ 28: Property-based testing ด้วย hypothesis
# pip install hypothesis

from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_add_commutative(a, b):
    """Hypothesis จะสร้าง test cases อัตโนมัติ"""
    assert a + b == b + a

@given(st.lists(st.integers()))
def test_sort_idempotent(lst):
    """Sort สองครั้งได้ผลเหมือน sort ครั้งเดียว"""
    assert sorted(sorted(lst)) == sorted(lst)

@given(st.text(), st.text())
def test_string_concatenation(s1, s2):
    """Concatenation ได้ความยาวที่ถูกต้อง"""
    result = s1 + s2
    assert len(result) == len(s1) + len(s2)
```

---

## 17. ตัวอย่างโปรแกรมจริง - Complete E-commerce Test Suite

```python
import pytest
from dataclasses import dataclass, field
from typing import List, Dict, Optional
from decimal import Decimal

# ตัวอย่างที่ 29: Complete e-commerce system tests

# Models
@dataclass
class Product:
    id: int
    name: str
    price: Decimal
    stock: int
    category: str

@dataclass  
class OrderItem:
    product: Product
    quantity: int
    
    @property
    def subtotal(self) -> Decimal:
        return self.product.price * self.quantity

@dataclass
class Order:
    id: int
    customer_id: int
    items: List[OrderItem] = field(default_factory=list)
    discount_percent: float = 0
    status: str = "pending"
    
    def add_item(self, product: Product, quantity: int):
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        if quantity > product.stock:
            raise ValueError(f"Insufficient stock")
        
        existing = next((i for i in self.items if i.product.id == product.id), None)
        if existing:
            if existing.quantity + quantity > product.stock:
                raise ValueError("Would exceed available stock")
            existing.quantity += quantity
        else:
            self.items.append(OrderItem(product=product, quantity=quantity))
    
    def remove_item(self, product_id: int):
        self.items = [i for i in self.items if i.product.id != product_id]
    
    @property
    def subtotal(self) -> Decimal:
        return sum(item.subtotal for item in self.items)
    
    @property
    def discount_amount(self) -> Decimal:
        return self.subtotal * Decimal(str(self.discount_percent)) / 100
    
    @property
    def total(self) -> Decimal:
        return self.subtotal - self.discount_amount
    
    def checkout(self):
        if not self.items:
            raise ValueError("Cannot checkout empty order")
        if self.status != "pending":
            raise ValueError(f"Cannot checkout order with status: {self.status}")
        
        # Deduct stock
        for item in self.items:
            item.product.stock -= item.quantity
        
        self.status = "confirmed"
        return self

# Fixtures
@pytest.fixture
def iphone():
    return Product(id=1, name="iPhone 15", price=Decimal("35000"), stock=10, category="Electronics")

@pytest.fixture  
def macbook():
    return Product(id=2, name="MacBook Pro", price=Decimal("89000"), stock=5, category="Electronics")

@pytest.fixture
def book():
    return Product(id=3, name="Python Book", price=Decimal("500"), stock=100, category="Books")

@pytest.fixture
def order(iphone, macbook):
    o = Order(id=1, customer_id=101)
    o.add_item(iphone, 2)
    o.add_item(macbook, 1)
    return o

# Tests
class TestProduct:
    def test_product_creation(self, iphone):
        assert iphone.name == "iPhone 15"
        assert iphone.price == Decimal("35000")
        assert iphone.stock == 10
    
    @pytest.mark.parametrize("product_id,name,price,stock", [
        (1, "Phone A", Decimal("10000"), 5),
        (2, "Phone B", Decimal("20000"), 10),
        (3, "Tablet", Decimal("15000"), 0),
    ])
    def test_various_products(self, product_id, name, price, stock):
        p = Product(id=product_id, name=name, price=price, stock=stock, category="Electronics")
        assert p.id == product_id
        assert p.price == price

class TestOrderItem:
    def test_subtotal_calculation(self, iphone):
        item = OrderItem(product=iphone, quantity=3)
        assert item.subtotal == Decimal("105000")

class TestOrder:
    def test_add_item(self, iphone):
        order = Order(id=1, customer_id=1)
        order.add_item(iphone, 2)
        
        assert len(order.items) == 1
        assert order.items[0].quantity == 2
    
    def test_add_duplicate_item_increases_quantity(self, iphone):
        order = Order(id=1, customer_id=1)
        order.add_item(iphone, 2)
        order.add_item(iphone, 3)
        
        assert len(order.items) == 1
        assert order.items[0].quantity == 5
    
    def test_add_exceeds_stock(self, iphone):
        order = Order(id=1, customer_id=1)
        
        with pytest.raises(ValueError, match="stock"):
            order.add_item(iphone, 100)
    
    def test_remove_item(self, order, iphone):
        order.remove_item(iphone.id)
        assert len(order.items) == 1
    
    def test_subtotal(self, order):
        expected = Decimal("35000") * 2 + Decimal("89000") * 1
        assert order.subtotal == expected
    
    @pytest.mark.parametrize("discount,expected_ratio", [
        (0, Decimal("1.0")),
        (10, Decimal("0.9")),
        (50, Decimal("0.5")),
    ])
    def test_discount(self, order, discount, expected_ratio):
        subtotal = order.subtotal
        order.discount_percent = discount
        
        expected_total = subtotal * expected_ratio
        assert order.total == pytest.approx(expected_total, rel=Decimal("0.001"))
    
    def test_checkout_success(self, order, iphone, macbook):
        initial_iphone_stock = iphone.stock
        initial_macbook_stock = macbook.stock
        
        result = order.checkout()
        
        assert result.status == "confirmed"
        assert iphone.stock == initial_iphone_stock - 2
        assert macbook.stock == initial_macbook_stock - 1
    
    def test_checkout_empty_order(self):
        order = Order(id=99, customer_id=1)
        
        with pytest.raises(ValueError, match="empty"):
            order.checkout()
    
    def test_checkout_already_confirmed(self, order):
        order.checkout()
        
        with pytest.raises(ValueError, match="status"):
            order.checkout()

if __name__ == '__main__':
    pytest.main([__file__, '-v', '--tb=short'])
```

---

## แบบฝึกหัด

### ข้อที่ 1: pytest fixtures สำหรับ User Authentication

**เฉลย:**
```python
import pytest
import hashlib
import secrets

class AuthService:
    def __init__(self):
        self._users = {}
    
    def register(self, username, password):
        if len(password) < 8:
            raise ValueError("Password too short")
        if username in self._users:
            raise ValueError("Username taken")
        salt = secrets.token_hex(16)
        hashed = hashlib.sha256((password + salt).encode()).hexdigest()
        self._users[username] = {'password_hash': hashed, 'salt': salt}
        return True
    
    def login(self, username, password):
        if username not in self._users:
            return False
        user = self._users[username]
        hashed = hashlib.sha256((password + user['salt']).encode()).hexdigest()
        return hashed == user['password_hash']
    
    def change_password(self, username, old_password, new_password):
        if not self.login(username, old_password):
            raise ValueError("Invalid credentials")
        if len(new_password) < 8:
            raise ValueError("New password too short")
        salt = secrets.token_hex(16)
        hashed = hashlib.sha256((new_password + salt).encode()).hexdigest()
        self._users[username] = {'password_hash': hashed, 'salt': salt}

@pytest.fixture
def auth():
    return AuthService()

@pytest.fixture
def registered_user(auth):
    auth.register("testuser", "password123")
    return auth

def test_register_success(auth):
    assert auth.register("alice", "securepass") is True

def test_register_short_password(auth):
    with pytest.raises(ValueError, match="short"):
        auth.register("bob", "short")

def test_login_success(registered_user):
    assert registered_user.login("testuser", "password123") is True

def test_login_wrong_password(registered_user):
    assert registered_user.login("testuser", "wrongpassword") is False

def test_login_unknown_user(auth):
    assert auth.login("unknown", "anypassword") is False

@pytest.mark.parametrize("password", ["12345678", "abcdefgh", "MyP@ssw0rd"])
def test_valid_passwords(auth, password):
    assert auth.register(f"user_{password[:4]}", password) is True

def test_change_password(registered_user):
    registered_user.change_password("testuser", "password123", "newpassword123")
    
    assert registered_user.login("testuser", "newpassword123") is True
    assert registered_user.login("testuser", "password123") is False

pytest.main([__file__, '-v'])
```

---

### ข้อที่ 2: Parametrize สำหรับ Data Validation

**เฉลย:**
```python
import pytest
import re

def validate_thai_phone(phone: str) -> bool:
    """ตรวจสอบเบอร์โทรศัพท์ไทย"""
    phone = phone.replace('-', '').replace(' ', '')
    return bool(re.match(r'^0[689]\d{8}$', phone))

def validate_thai_id(id_number: str) -> bool:
    """ตรวจสอบเลขบัตรประชาชนไทย"""
    id_number = id_number.replace('-', '').replace(' ', '')
    if not re.match(r'^\d{13}$', id_number):
        return False
    total = sum(int(d) * (13 - i) for i, d in enumerate(id_number[:12]))
    check = (11 - total % 11) % 10
    return check == int(id_number[12])

@pytest.mark.parametrize("phone,valid", [
    ("0812345678", True),
    ("0912345678", True),
    ("0612345678", True),
    ("081-234-5678", True),
    ("1234567890", False),
    ("081234567", False),
    ("08123456789", False),
    ("", False),
])
def test_thai_phone(phone, valid):
    assert validate_thai_phone(phone) == valid

@pytest.mark.parametrize("id_num,valid", [
    ("1100702139549", True),   # valid checksum
    ("1234567890123", False),  # invalid checksum
    ("123456789012", False),   # too short
    ("12345678901234", False), # too long
])
def test_thai_id(id_num, valid):
    assert validate_thai_id(id_num) == valid

pytest.main([__file__, '-v'])
```

---

### ข้อที่ 3-10: แบบฝึกหัดเพิ่มเติม

### ข้อที่ 3: Testing Async Code

**เฉลย:**
```python
import pytest
import asyncio

async def fetch_user(user_id: int) -> dict:
    """Async function จำลอง API call"""
    await asyncio.sleep(0.01)  # simulate network delay
    
    users = {
        1: {'id': 1, 'name': 'Alice', 'email': 'alice@test.com'},
        2: {'id': 2, 'name': 'Bob', 'email': 'bob@test.com'},
    }
    
    if user_id not in users:
        raise ValueError(f"User {user_id} not found")
    
    return users[user_id]

async def fetch_multiple_users(user_ids: list) -> list:
    """Fetch หลาย users พร้อมกัน"""
    tasks = [fetch_user(uid) for uid in user_ids]
    return await asyncio.gather(*tasks, return_exceptions=True)

@pytest.mark.asyncio
async def test_fetch_existing_user():
    user = await fetch_user(1)
    assert user['name'] == 'Alice'
    assert user['id'] == 1

@pytest.mark.asyncio
async def test_fetch_nonexistent_user():
    with pytest.raises(ValueError, match="not found"):
        await fetch_user(99)

@pytest.mark.asyncio
async def test_fetch_multiple():
    results = await fetch_multiple_users([1, 2])
    
    names = [r['name'] for r in results if isinstance(r, dict)]
    assert 'Alice' in names
    assert 'Bob' in names

@pytest.mark.asyncio
async def test_fetch_multiple_with_error():
    results = await fetch_multiple_users([1, 99])
    
    success_count = sum(1 for r in results if isinstance(r, dict))
    error_count = sum(1 for r in results if isinstance(r, Exception))
    
    assert success_count == 1
    assert error_count == 1

# รัน: pytest test_async.py -v (ต้องมี pytest-asyncio)
print("Async tests defined - run with: pytest test_file.py -v")
```

---

### ข้อที่ 4: Testing กับ External Dependencies

**เฉลย:**
```python
import pytest
from unittest.mock import patch, MagicMock
import json

class PaymentGateway:
    API_URL = "https://api.payment.com/v1"
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def charge(self, amount: float, card_token: str, currency: str = "THB") -> dict:
        import urllib.request
        import urllib.error
        
        payload = json.dumps({
            "amount": int(amount * 100),  # บาทเป็น satang
            "currency": currency,
            "card_token": card_token
        }).encode()
        
        req = urllib.request.Request(
            f"{self.API_URL}/charges",
            data=payload,
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
        )
        
        try:
            with urllib.request.urlopen(req) as response:
                return json.loads(response.read())
        except urllib.error.HTTPError as e:
            error_body = json.loads(e.read())
            raise ValueError(f"Payment failed: {error_body.get('message', 'Unknown error')}")

@pytest.fixture
def payment():
    return PaymentGateway(api_key="test_key_123")

def test_successful_charge(payment):
    success_response = {
        "id": "ch_123",
        "amount": 3500000,
        "currency": "THB",
        "status": "succeeded"
    }
    
    mock_response = MagicMock()
    mock_response.read.return_value = json.dumps(success_response).encode()
    mock_response.__enter__.return_value = mock_response
    mock_response.__exit__.return_value = False
    
    with patch('urllib.request.urlopen', return_value=mock_response):
        result = payment.charge(35000, "tok_test_123")
    
    assert result['status'] == 'succeeded'
    assert result['id'] == 'ch_123'

def test_failed_charge(payment):
    import urllib.error
    
    error_body = json.dumps({"message": "Card declined"}).encode()
    
    with patch('urllib.request.urlopen') as mock_urlopen:
        mock_urlopen.side_effect = urllib.error.HTTPError(
            url='', code=402, msg='Payment Required',
            hdrs={}, fp=MagicMock(read=lambda: error_body)
        )
        
        with pytest.raises(ValueError, match="Payment failed"):
            payment.charge(1000, "tok_declined")

pytest.main([__file__, '-v'])
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

1. **pytest basics**: Test functions ที่ไม่ต้องใช้ class, assert แบบธรรมชาติ
2. **Assertions**: pytest.approx, assertRaises context manager
3. **Fixtures**: Setup/teardown ที่ reusable และ composable
4. **conftest.py**: Shared fixtures สำหรับทั้งโปรเจกต์
5. **Parametrize**: รัน test หลาย inputs อย่างกระชับ
6. **Marks**: skip, xfail, custom marks
7. **pytest-cov**: วัด code coverage
8. **pytest-mock**: Mock objects และ patches
9. **Test Organization**: โครงสร้างที่ดีสำหรับโปรเจกต์ขนาดใหญ่
10. **Plugins**: xdist, asyncio, benchmark, hypothesis
11. **TDD**: Red-Green-Refactor workflow
12. **BDD**: Behavior-Driven Development ด้วย pytest-bdd

**เคล็ดลับสำคัญ:**
- เขียน tests ก่อน code (TDD) หรืออย่างน้อยเขียนพร้อมกัน
- Test ที่ดีต้องอ่านเข้าใจง่าย - เป็น documentation ของ code
- ใช้ fixtures แทน copy-paste setup code
- parametrize เพื่อ test edge cases ได้ครบ
- Coverage สูงไม่เพียงพอ - ต้อง test behaviors ที่ถูกต้องด้วย
