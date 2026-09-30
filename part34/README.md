# Part 34: Testing with unittest

## บทนำ

"Code without tests is broken by design" - Jacob Kaplan-Moss

การทดสอบ (Testing) เป็นทักษะที่แยกนักพัฒนามืออาชีพออกจากมือสมัครเล่น การเขียน tests ที่ดีช่วยให้:
- ตรวจจับ bugs ก่อนที่ user จะเจอ
- Refactor code ได้อย่างมั่นใจ
- เป็น documentation ที่ทำงานได้จริง
- ออกแบบ code ให้ดีขึ้น (testable code = good design)

---

## 1. Why Testing Matters

### ประเภทของ Tests

```
Testing Pyramid:
         /\
        /  \
       / UI \
      / Tests\
     /--------\
    /Integration\
   /   Tests     \
  /--------------\
 /   Unit Tests   \
/------------------\

Unit Tests: เร็ว, isolate components, รันบ่อย
Integration Tests: ช้ากว่า, test components ร่วมกัน
UI/E2E Tests: ช้าที่สุด, test ทั้ง system
```

```python
# ตัวอย่างที่ 1: เหตุผลที่ต้องเขียน tests

# โค้ดที่ดูปกติแต่มี bug ซ่อนอยู่
def calculate_discount(price, discount_percent):
    """คำนวณราคาหลังหักส่วนลด"""
    return price * (1 - discount_percent)  # Bug! ควรหาร 100

# ถ้าไม่มี test อาจไม่รู้ว่า bug นี้มีอยู่
result = calculate_discount(100, 20)  # ได้ -1900 ไม่ใช่ 80!

# ด้วย test จะเห็น bug ทันที
import unittest

class TestDiscount(unittest.TestCase):
    def test_20_percent_off_100(self):
        result = calculate_discount(100, 20)
        self.assertEqual(result, 80)  # FAIL: got -1900

# Fix:
def calculate_discount_fixed(price, discount_percent):
    return price * (1 - discount_percent / 100)
```

---

## 2. unittest Module

unittest เป็น testing framework ที่มาพร้อมกับ Python ไม่ต้องติดตั้งเพิ่ม

```python
import unittest

# ตัวอย่างที่ 2: โครงสร้างพื้นฐานของ unittest
class TestBasicExample(unittest.TestCase):
    """TestCase class - รวม tests ที่เกี่ยวข้องกัน"""
    
    def test_addition(self):
        """Test method ต้องขึ้นต้นด้วย test_"""
        result = 1 + 1
        self.assertEqual(result, 2)
    
    def test_string_upper(self):
        self.assertEqual("hello".upper(), "HELLO")
    
    def test_list_length(self):
        self.assertEqual(len([1, 2, 3]), 3)

if __name__ == '__main__':
    unittest.main()
```

```bash
# รัน tests
python -m unittest test_basic.py        # รัน file เดียว
python -m unittest                       # auto discover
python -m unittest -v                    # verbose mode
python -m unittest TestBasicExample      # รัน class เดียว
python -m unittest TestBasicExample.test_addition  # รัน test เดียว
```

---

## 3. TestCase Class

```python
import unittest

# ตัวอย่างที่ 3: TestCase methods และ attributes
class TestCaseDemo(unittest.TestCase):
    
    # Class attribute สำหรับ test settings
    maxDiff = None  # ไม่จำกัดความยาวของ diff output
    
    def test_basic_structure(self):
        """แสดง structure ของ TestCase"""
        # self คือ instance ของ TestCase
        # มี methods สำหรับ assertions
        self.assertTrue(True)
        self.assertFalse(False)
        self.assertIsNone(None)
        self.assertIsNotNone("not none")
    
    def test_skip_example(self):
        """ข้าม test นี้"""
        self.skipTest("ยังไม่ implement")
    
    @unittest.skip("เหตุผลที่ skip")
    def test_skipped(self):
        pass
    
    @unittest.skipIf(True, "skip if condition is true")
    def test_conditional_skip(self):
        pass
    
    @unittest.expectedFailure
    def test_known_bug(self):
        """Test ที่รู้ว่า fail - ถ้า pass จะถือว่า unexpected"""
        self.assertEqual(1, 2)
```

---

## 4. setUp() และ tearDown()

```python
import unittest
import tempfile
import os

# ตัวอย่างที่ 4: setUp และ tearDown
class TestFileOperations(unittest.TestCase):
    
    def setUp(self):
        """รันก่อนทุก test method - สร้าง test fixtures"""
        print("\nsetUp: กำลังเตรียม test environment")
        
        # สร้าง temporary directory
        self.test_dir = tempfile.mkdtemp()
        
        # สร้างไฟล์ทดสอบ
        self.test_file = os.path.join(self.test_dir, 'test.txt')
        with open(self.test_file, 'w') as f:
            f.write("Hello, World!\n")
        
        # สร้าง test data
        self.test_data = {'name': 'Alice', 'age': 25}
    
    def tearDown(self):
        """รันหลังทุก test method - cleanup"""
        print("tearDown: กำลังทำความสะอาด")
        
        import shutil
        shutil.rmtree(self.test_dir, ignore_errors=True)
    
    def test_file_exists(self):
        """ตรวจสอบว่าไฟล์ถูกสร้าง"""
        self.assertTrue(os.path.exists(self.test_file))
    
    def test_file_content(self):
        """ตรวจสอบเนื้อหาไฟล์"""
        with open(self.test_file, 'r') as f:
            content = f.read()
        self.assertEqual(content, "Hello, World!\n")
    
    def test_data_structure(self):
        """ตรวจสอบ test data"""
        self.assertIn('name', self.test_data)
        self.assertEqual(self.test_data['age'], 25)
```

```python
import unittest

# ตัวอย่างที่ 5: setUpClass และ tearDownClass
class TestExpensiveSetup(unittest.TestCase):
    """สำหรับ setup ที่ต้องทำแค่ครั้งเดียวต่อ class"""
    
    @classmethod
    def setUpClass(cls):
        """รันครั้งเดียวก่อนทุก tests ใน class"""
        print("\nsetUpClass: สร้าง database connection")
        cls.db_data = [1, 2, 3, 4, 5]
        cls.connection_count = 0
    
    @classmethod
    def tearDownClass(cls):
        """รันครั้งเดียวหลัง tests ทั้งหมดใน class"""
        print("tearDownClass: ปิด database connection")
        print(f"สร้าง connection {cls.connection_count} ครั้ง")
    
    def setUp(self):
        """รันก่อนทุก test"""
        self.__class__.connection_count += 1
    
    def test_first(self):
        self.assertEqual(self.db_data[0], 1)
    
    def test_last(self):
        self.assertEqual(self.db_data[-1], 5)
    
    def test_length(self):
        self.assertEqual(len(self.db_data), 5)
```

---

## 5. Assertions ทั้งหมด

```python
import unittest

# ตัวอย่างที่ 6: Equality assertions
class TestEqualityAssertions(unittest.TestCase):
    
    def test_equal(self):
        self.assertEqual(1 + 1, 2)
        self.assertEqual("hello", "hello")
        self.assertEqual([1, 2, 3], [1, 2, 3])
    
    def test_not_equal(self):
        self.assertNotEqual(1, 2)
        self.assertNotEqual("hello", "world")
    
    def test_almost_equal(self):
        """สำหรับ floating point"""
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=10)
        self.assertAlmostEqual(1.0, 1.0000001, delta=0.001)
    
    def test_not_almost_equal(self):
        self.assertNotAlmostEqual(1.0, 1.5, places=1)
```

```python
import unittest

# ตัวอย่างที่ 7: Boolean assertions
class TestBooleanAssertions(unittest.TestCase):
    
    def test_true_false(self):
        self.assertTrue(True)
        self.assertTrue(1)
        self.assertTrue("non-empty")
        self.assertTrue([1])
        
        self.assertFalse(False)
        self.assertFalse(0)
        self.assertFalse("")
        self.assertFalse([])
    
    def test_none(self):
        self.assertIsNone(None)
        self.assertIsNotNone(0)
        self.assertIsNotNone("")
        self.assertIsNotNone(False)  # False is not None!
```

```python
import unittest

# ตัวอย่างที่ 8: Comparison assertions
class TestComparisonAssertions(unittest.TestCase):
    
    def test_greater_less(self):
        self.assertGreater(5, 3)
        self.assertGreaterEqual(5, 5)
        self.assertLess(3, 5)
        self.assertLessEqual(5, 5)
    
    def test_in_not_in(self):
        self.assertIn('hello', ['hello', 'world'])
        self.assertIn('h', 'hello')
        self.assertIn('key', {'key': 'value'})
        
        self.assertNotIn('xyz', ['hello', 'world'])
    
    def test_is_isinstance(self):
        self.assertIs(None, None)
        self.assertIsNot([], [])  # สอง lists ต่างกัน
        
        self.assertIsInstance([], list)
        self.assertIsInstance("hello", (str, bytes))
        self.assertNotIsInstance(1, str)
```

```python
import unittest

# ตัวอย่างที่ 9: Exception assertions
class TestExceptionAssertions(unittest.TestCase):
    
    def test_raises(self):
        """ตรวจสอบว่า raise exception ที่ถูกต้อง"""
        with self.assertRaises(ValueError):
            int("not a number")
        
        with self.assertRaises(ZeroDivisionError):
            1 / 0
        
        with self.assertRaises(KeyError):
            {}['nonexistent']
    
    def test_raises_with_message(self):
        """ตรวจสอบ exception message"""
        with self.assertRaises(ValueError) as ctx:
            raise ValueError("invalid input: must be positive")
        
        # ตรวจสอบ exception message
        self.assertIn("invalid input", str(ctx.exception))
        self.assertEqual(type(ctx.exception).__name__, "ValueError")
    
    def test_raises_regex(self):
        """ตรวจสอบ exception message ด้วย regex"""
        with self.assertRaisesRegex(ValueError, r"must be \w+"):
            raise ValueError("value must be positive")
    
    def test_no_exception(self):
        """ตรวจสอบว่าไม่ raise exception"""
        # ถ้า raise จะ fail test
        result = int("42")
        self.assertEqual(result, 42)
```

```python
import unittest

# ตัวอย่างที่ 10: String assertions
class TestStringAssertions(unittest.TestCase):
    
    def test_regex(self):
        """ตรวจสอบด้วย regex"""
        import re
        email = "user@example.com"
        self.assertRegex(email, r'^\w+@\w+\.\w+$')
        
        invalid_email = "not-an-email"
        self.assertNotRegex(invalid_email, r'^\w+@\w+\.\w+$')
    
    def test_multiline(self):
        """เปรียบเทียบ multiline strings"""
        expected = "line1\nline2\nline3"
        actual = "line1\nline2\nline3"
        self.assertEqual(expected, actual)
```

```python
import unittest

# ตัวอย่างที่ 11: Collection assertions
class TestCollectionAssertions(unittest.TestCase):
    
    def test_dict_equal(self):
        """เปรียบเทียบ dictionaries"""
        self.assertDictEqual(
            {'a': 1, 'b': 2},
            {'b': 2, 'a': 1}  # ลำดับไม่สำคัญ
        )
    
    def test_list_equal(self):
        """เปรียบเทียบ lists"""
        self.assertListEqual([1, 2, 3], [1, 2, 3])
        
        # ลำดับสำคัญใน assertListEqual
        # [1, 2, 3] != [3, 2, 1]
    
    def test_set_equal(self):
        """เปรียบเทียบ sets"""
        self.assertSetEqual({1, 2, 3}, {3, 1, 2})
    
    def test_tuple_equal(self):
        self.assertTupleEqual((1, 2, 3), (1, 2, 3))
    
    def test_count_equal(self):
        """เปรียบเทียบโดยไม่สนใจลำดับ"""
        self.assertCountEqual([1, 2, 3], [3, 1, 2])
        self.assertCountEqual([1, 1, 2], [2, 1, 1])
```

---

## 6. Test Suites

```python
import unittest

# ตัวอย่างที่ 12: สร้าง Test Suite
class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(1 + 1, 2)
    
    def test_subtract(self):
        self.assertEqual(5 - 3, 2)

class TestString(unittest.TestCase):
    def test_upper(self):
        self.assertEqual("hello".upper(), "HELLO")
    
    def test_lower(self):
        self.assertEqual("HELLO".lower(), "hello")

# สร้าง suite ด้วยมือ
def suite():
    test_suite = unittest.TestSuite()
    
    # เพิ่ม tests เฉพาะที่ต้องการ
    test_suite.addTest(TestMath('test_add'))
    test_suite.addTest(unittest.TestLoader().loadTestsFromTestCase(TestString))
    
    return test_suite

# รัน suite
runner = unittest.TextTestRunner(verbosity=2)
result = runner.run(suite())
print(f"\nTests: {result.testsRun}")
print(f"Failures: {len(result.failures)}")
print(f"Errors: {len(result.errors)}")
print(f"Success: {result.wasSuccessful()}")
```

---

## 7. Test Discovery

```python
# ตัวอย่างที่ 13: Test Discovery

# โครงสร้างไฟล์:
# project/
# ├── src/
# │   └── calculator.py
# └── tests/
#     ├── __init__.py
#     ├── test_calculator.py
#     ├── test_string_utils.py
#     └── integration/
#         └── test_db.py

# python -m unittest discover
# จะหา test files ที่ match pattern test*.py
# เริ่มจาก current directory

# ตัวเลือก:
# -s, --start-directory : directory เริ่มต้น (default: .)
# -p, --pattern         : pattern ของ test files (default: test*.py)
# -t, --top-level-directory : top-level directory

# ตัวอย่าง:
# python -m unittest discover -s tests -p "test_*.py"
# python -m unittest discover -s tests -p "*_test.py"

import unittest

def load_tests(loader, tests, pattern):
    """Custom test loading"""
    suite = unittest.TestSuite()
    # เพิ่ม tests ที่ต้องการ
    return suite
```

---

## 8. Mocking - unittest.mock

Mocking ช่วยให้ทดสอบ code ที่ต้องพึ่งพา external dependencies โดยแทนที่ด้วย fake objects

```python
from unittest.mock import Mock, MagicMock, patch, call
import unittest

# ตัวอย่างที่ 14: Mock พื้นฐาน
class TestMockBasics(unittest.TestCase):
    
    def test_mock_return_value(self):
        """Mock ส่งคืนค่าที่กำหนด"""
        mock_func = Mock(return_value=42)
        
        result = mock_func()
        self.assertEqual(result, 42)
        mock_func.assert_called_once()
    
    def test_mock_side_effects(self):
        """Mock ที่มี side effects"""
        mock_func = Mock(side_effect=[1, 2, 3, StopIteration])
        
        self.assertEqual(mock_func(), 1)
        self.assertEqual(mock_func(), 2)
        self.assertEqual(mock_func(), 3)
        
        with self.assertRaises(StopIteration):
            mock_func()
    
    def test_mock_attributes(self):
        """Mock attribute access"""
        mock_obj = Mock()
        mock_obj.name = "Alice"
        mock_obj.get_age.return_value = 25
        
        self.assertEqual(mock_obj.name, "Alice")
        self.assertEqual(mock_obj.get_age(), 25)
```

```python
from unittest.mock import Mock, patch
import unittest

# ตัวอย่างที่ 15: Mock call verification
class TestCallVerification(unittest.TestCase):
    
    def test_call_count(self):
        mock_func = Mock()
        
        mock_func(1)
        mock_func(2)
        mock_func(3)
        
        self.assertEqual(mock_func.call_count, 3)
    
    def test_called_with(self):
        mock_func = Mock()
        mock_func("hello", "world")
        
        # ตรวจสอบว่าถูกเรียกด้วย arguments อะไร
        mock_func.assert_called_with("hello", "world")
        mock_func.assert_called_once_with("hello", "world")
    
    def test_call_history(self):
        mock_func = Mock()
        
        mock_func(1, x=10)
        mock_func(2, x=20)
        
        # ดู call history
        print(mock_func.call_args_list)
        # [call(1, x=10), call(2, x=20)]
        
        expected_calls = [call(1, x=10), call(2, x=20)]
        mock_func.assert_has_calls(expected_calls)
    
    def test_not_called(self):
        mock_func = Mock()
        mock_func.assert_not_called()
```

---

## 9. patch Decorator

```python
from unittest.mock import patch, MagicMock
import unittest

# module ที่ต้องการ test
# สมมติว่า: src/payment.py
class PaymentProcessor:
    def process(self, amount, card_number):
        # จริงๆ จะเรียก external API
        from external_payment_api import charge  # สมมติ
        return charge(amount, card_number)

def get_weather(city: str) -> dict:
    """ดึงข้อมูลอากาศจาก API"""
    import urllib.request
    import json
    url = f"https://api.weather.com/{city}"
    with urllib.request.urlopen(url) as response:
        return json.loads(response.read())

# ตัวอย่างที่ 16: patch เป็น decorator
class TestWithPatch(unittest.TestCase):
    
    @patch('urllib.request.urlopen')
    def test_get_weather_success(self, mock_urlopen):
        """Mock HTTP request"""
        import json
        from io import BytesIO
        
        # กำหนดให้ mock ส่งคืน fake response
        fake_response = {'city': 'Bangkok', 'temp': 35, 'condition': 'Sunny'}
        mock_urlopen.return_value.__enter__.return_value.read.return_value = (
            json.dumps(fake_response).encode()
        )
        
        # เรียกฟังก์ชันที่ใช้ HTTP request
        # result = get_weather('Bangkok')
        # self.assertEqual(result['city'], 'Bangkok')
        
        # ตรวจสอบว่า urlopen ถูกเรียก
        # mock_urlopen.assert_called_once()
        
        self.assertTrue(True)  # placeholder
    
    @patch('builtins.open')
    def test_file_operations(self, mock_open):
        """Mock file operations"""
        mock_open.return_value.__enter__.return_value.read.return_value = "file content"
        
        # Code ที่อ่านไฟล์
        with open('test.txt', 'r') as f:
            content = f.read()
        
        self.assertEqual(content, "file content")
        mock_open.assert_called_with('test.txt', 'r')
```

```python
from unittest.mock import patch, MagicMock
import unittest
from datetime import datetime

# ตัวอย่างที่ 17: patch เป็น context manager
class DateTimeService:
    def get_current_date(self):
        return datetime.now().date()
    
    def is_weekend(self):
        return datetime.now().weekday() >= 5

class TestDateTimeService(unittest.TestCase):
    
    def test_is_weekend_saturday(self):
        service = DateTimeService()
        
        with patch('datetime.datetime') as mock_dt:
            # กำหนดให้เป็นวันเสาร์ (weekday=5)
            mock_dt.now.return_value.weekday.return_value = 5
            result = service.is_weekend()
        
        self.assertTrue(result)
    
    def test_is_weekday(self):
        service = DateTimeService()
        
        with patch('datetime.datetime') as mock_dt:
            # กำหนดให้เป็นวันจันทร์ (weekday=0)
            mock_dt.now.return_value.weekday.return_value = 0
            result = service.is_weekend()
        
        self.assertFalse(result)
```

---

## 10. MagicMock

```python
from unittest.mock import MagicMock
import unittest

# ตัวอย่างที่ 18: MagicMock สำหรับ magic methods
class TestMagicMock(unittest.TestCase):
    
    def test_context_manager(self):
        """Mock context manager"""
        mock_db = MagicMock()
        
        with mock_db as conn:
            conn.execute("SELECT 1")
        
        mock_db.__enter__.assert_called_once()
        mock_db.__exit__.assert_called_once()
    
    def test_iteration(self):
        """Mock iterable"""
        mock_file = MagicMock()
        mock_file.__iter__.return_value = iter(["line1\n", "line2\n", "line3\n"])
        
        lines = [line.strip() for line in mock_file]
        self.assertEqual(lines, ["line1", "line2", "line3"])
    
    def test_len_and_contains(self):
        """Mock __len__ และ __contains__"""
        mock_container = MagicMock()
        mock_container.__len__.return_value = 5
        mock_container.__contains__.return_value = True
        
        self.assertEqual(len(mock_container), 5)
        self.assertIn("anything", mock_container)
    
    def test_comparison(self):
        """Mock comparison operators"""
        mock_obj = MagicMock()
        mock_obj.__eq__.return_value = True
        mock_obj.__lt__.return_value = False
        
        self.assertTrue(mock_obj == "anything")
        self.assertFalse(mock_obj < "anything")
```

---

## 11. Side Effects

```python
from unittest.mock import Mock, patch
import unittest

# ตัวอย่างที่ 19: Side effects แบบต่างๆ
class TestSideEffects(unittest.TestCase):
    
    def test_exception_side_effect(self):
        """Mock ที่ raise exception"""
        mock_func = Mock(side_effect=ConnectionError("Database unavailable"))
        
        with self.assertRaises(ConnectionError):
            mock_func()
    
    def test_list_side_effect(self):
        """Mock ที่ส่งคืนค่าต่างกันทุกครั้ง"""
        mock_func = Mock(side_effect=[10, 20, 30])
        
        self.assertEqual(mock_func(), 10)
        self.assertEqual(mock_func(), 20)
        self.assertEqual(mock_func(), 30)
        
        with self.assertRaises(StopIteration):
            mock_func()  # หมดแล้ว
    
    def test_callable_side_effect(self):
        """Mock ด้วย function เป็น side effect"""
        def validate_age(age):
            if age < 0:
                raise ValueError("Age cannot be negative")
            return age * 2
        
        mock_process = Mock(side_effect=validate_age)
        
        self.assertEqual(mock_process(10), 20)
        self.assertEqual(mock_process(25), 50)
        
        with self.assertRaises(ValueError):
            mock_process(-1)
```

---

## 12. ตัวอย่างโปรแกรมจริง - Testing Calculator

```python
import unittest
from unittest.mock import patch

# ตัวอย่างที่ 20: Calculator class ที่ต้องการ test
class Calculator:
    def __init__(self):
        self.history = []
    
    def add(self, a, b):
        result = a + b
        self.history.append(f"{a} + {b} = {result}")
        return result
    
    def subtract(self, a, b):
        result = a - b
        self.history.append(f"{a} - {b} = {result}")
        return result
    
    def multiply(self, a, b):
        result = a * b
        self.history.append(f"{a} * {b} = {result}")
        return result
    
    def divide(self, a, b):
        if b == 0:
            raise ZeroDivisionError("Cannot divide by zero")
        result = a / b
        self.history.append(f"{a} / {b} = {result}")
        return result
    
    def power(self, base, exp):
        result = base ** exp
        self.history.append(f"{base} ** {exp} = {result}")
        return result
    
    def sqrt(self, n):
        if n < 0:
            raise ValueError("Cannot take sqrt of negative number")
        import math
        result = math.sqrt(n)
        self.history.append(f"sqrt({n}) = {result}")
        return result
    
    def get_history(self):
        return self.history.copy()
    
    def clear_history(self):
        self.history.clear()

# Tests สำหรับ Calculator
class TestCalculatorBasic(unittest.TestCase):
    
    def setUp(self):
        self.calc = Calculator()
    
    def test_add_positive(self):
        self.assertEqual(self.calc.add(3, 4), 7)
    
    def test_add_negative(self):
        self.assertEqual(self.calc.add(-3, -4), -7)
    
    def test_add_mixed(self):
        self.assertEqual(self.calc.add(-3, 4), 1)
    
    def test_add_zero(self):
        self.assertEqual(self.calc.add(5, 0), 5)
    
    def test_subtract(self):
        self.assertEqual(self.calc.subtract(10, 3), 7)
    
    def test_multiply(self):
        self.assertEqual(self.calc.multiply(4, 5), 20)
    
    def test_multiply_by_zero(self):
        self.assertEqual(self.calc.multiply(100, 0), 0)
    
    def test_divide_normal(self):
        self.assertEqual(self.calc.divide(10, 2), 5.0)
    
    def test_divide_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            self.calc.divide(10, 0)
    
    def test_divide_result_is_float(self):
        result = self.calc.divide(7, 2)
        self.assertAlmostEqual(result, 3.5)
    
    def test_power(self):
        self.assertEqual(self.calc.power(2, 10), 1024)
    
    def test_sqrt_positive(self):
        self.assertAlmostEqual(self.calc.sqrt(9), 3.0)
    
    def test_sqrt_negative(self):
        with self.assertRaises(ValueError) as ctx:
            self.calc.sqrt(-1)
        self.assertIn("negative", str(ctx.exception).lower())

class TestCalculatorHistory(unittest.TestCase):
    
    def setUp(self):
        self.calc = Calculator()
    
    def test_history_initially_empty(self):
        self.assertEqual(self.calc.get_history(), [])
    
    def test_history_records_operations(self):
        self.calc.add(3, 4)
        self.calc.multiply(2, 5)
        
        history = self.calc.get_history()
        self.assertEqual(len(history), 2)
        self.assertIn("3 + 4 = 7", history[0])
    
    def test_history_cleared(self):
        self.calc.add(1, 2)
        self.calc.clear_history()
        
        self.assertEqual(self.calc.get_history(), [])
    
    def test_history_returns_copy(self):
        self.calc.add(1, 2)
        history = self.calc.get_history()
        history.append("TAMPERED")
        
        # Original history ไม่ถูกแก้ไข
        self.assertEqual(len(self.calc.get_history()), 1)

# รัน tests
if __name__ == '__main__':
    unittest.main(verbosity=2)
```

---

## 13. ตัวอย่างโปรแกรมจริง - Testing File Operations

```python
import unittest
import tempfile
import os
from unittest.mock import patch, mock_open

# ตัวอย่างที่ 21: File processor class
class FileProcessor:
    def read_file(self, filepath):
        if not os.path.exists(filepath):
            raise FileNotFoundError(f"File not found: {filepath}")
        with open(filepath, 'r', encoding='utf-8') as f:
            return f.read()
    
    def write_file(self, filepath, content):
        os.makedirs(os.path.dirname(filepath) or '.', exist_ok=True)
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)
        return len(content)
    
    def count_words(self, text):
        if not text.strip():
            return 0
        return len(text.split())
    
    def count_lines(self, text):
        if not text:
            return 0
        return len(text.splitlines())
    
    def process_csv_line(self, line):
        """แยก CSV line เป็น list"""
        import csv
        import io
        reader = csv.reader(io.StringIO(line))
        return next(reader)
    
    def get_file_stats(self, filepath):
        content = self.read_file(filepath)
        return {
            'words': self.count_words(content),
            'lines': self.count_lines(content),
            'chars': len(content),
            'bytes': os.path.getsize(filepath)
        }

class TestFileProcessor(unittest.TestCase):
    
    def setUp(self):
        self.processor = FileProcessor()
        self.temp_dir = tempfile.mkdtemp()
    
    def tearDown(self):
        import shutil
        shutil.rmtree(self.temp_dir)
    
    def _create_temp_file(self, content, filename='test.txt'):
        filepath = os.path.join(self.temp_dir, filename)
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)
        return filepath
    
    def test_read_existing_file(self):
        content = "Hello, World!\nThis is a test."
        filepath = self._create_temp_file(content)
        
        result = self.processor.read_file(filepath)
        self.assertEqual(result, content)
    
    def test_read_nonexistent_file(self):
        with self.assertRaises(FileNotFoundError):
            self.processor.read_file('/nonexistent/path/file.txt')
    
    def test_write_file(self):
        filepath = os.path.join(self.temp_dir, 'output.txt')
        content = "Test content"
        
        bytes_written = self.processor.write_file(filepath, content)
        
        self.assertEqual(bytes_written, len(content))
        self.assertTrue(os.path.exists(filepath))
        
        with open(filepath) as f:
            self.assertEqual(f.read(), content)
    
    def test_count_words(self):
        test_cases = [
            ("Hello World", 2),
            ("one two three four five", 5),
            ("", 0),
            ("   ", 0),
            ("single", 1),
        ]
        
        for text, expected in test_cases:
            with self.subTest(text=text):
                self.assertEqual(self.processor.count_words(text), expected)
    
    def test_count_lines(self):
        self.assertEqual(self.processor.count_lines("line1\nline2\nline3"), 3)
        self.assertEqual(self.processor.count_lines("single line"), 1)
        self.assertEqual(self.processor.count_lines(""), 0)
    
    def test_process_csv_line(self):
        result = self.processor.process_csv_line('Alice,25,"Bangkok, TH"')
        self.assertEqual(result, ['Alice', '25', 'Bangkok, TH'])
    
    def test_get_file_stats(self):
        content = "Hello World\nThis is line two\nThird line"
        filepath = self._create_temp_file(content)
        
        stats = self.processor.get_file_stats(filepath)
        
        self.assertEqual(stats['words'], 8)
        self.assertEqual(stats['lines'], 3)
        self.assertGreater(stats['bytes'], 0)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

---

## 14. ตัวอย่างโปรแกรมจริง - Testing API Calls

```python
import unittest
from unittest.mock import patch, MagicMock, Mock
import json

# ตัวอย่างที่ 22: API Client ที่ต้องการ test
class WeatherAPI:
    BASE_URL = "https://api.openweathermap.org/data/2.5"
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.session = None
    
    def get_current_weather(self, city: str) -> dict:
        import urllib.request
        import urllib.error
        
        url = f"{self.BASE_URL}/weather?q={city}&appid={self.api_key}"
        
        try:
            with urllib.request.urlopen(url, timeout=5) as response:
                data = json.loads(response.read())
                return {
                    'city': data['name'],
                    'temp': data['main']['temp'] - 273.15,  # Kelvin to Celsius
                    'description': data['weather'][0]['description'],
                    'humidity': data['main']['humidity']
                }
        except urllib.error.HTTPError as e:
            if e.code == 404:
                raise ValueError(f"City '{city}' not found")
            raise
        except urllib.error.URLError as e:
            raise ConnectionError(f"Network error: {e}")
    
    def get_forecast(self, city: str, days: int = 5) -> list:
        import urllib.request
        
        url = f"{self.BASE_URL}/forecast?q={city}&cnt={days}&appid={self.api_key}"
        
        with urllib.request.urlopen(url) as response:
            data = json.loads(response.read())
            return [{
                'date': item['dt_txt'],
                'temp': item['main']['temp'] - 273.15,
                'description': item['weather'][0]['description']
            } for item in data['list']]

class TestWeatherAPI(unittest.TestCase):
    
    def setUp(self):
        self.api = WeatherAPI(api_key="test-key-123")
    
    def _make_mock_response(self, data: dict, status_code: int = 200):
        """สร้าง mock HTTP response"""
        mock_response = MagicMock()
        mock_response.read.return_value = json.dumps(data).encode()
        mock_response.__enter__.return_value = mock_response
        mock_response.__exit__.return_value = False
        return mock_response
    
    @patch('urllib.request.urlopen')
    def test_get_weather_success(self, mock_urlopen):
        """ทดสอบกรณี API ตอบสำเร็จ"""
        # กำหนด mock response
        mock_data = {
            'name': 'Bangkok',
            'main': {'temp': 308.15, 'humidity': 70},
            'weather': [{'description': 'clear sky'}]
        }
        mock_urlopen.return_value = self._make_mock_response(mock_data)
        
        result = self.api.get_current_weather('Bangkok')
        
        self.assertEqual(result['city'], 'Bangkok')
        self.assertAlmostEqual(result['temp'], 35.0, places=1)
        self.assertEqual(result['description'], 'clear sky')
        self.assertEqual(result['humidity'], 70)
    
    @patch('urllib.request.urlopen')
    def test_get_weather_city_not_found(self, mock_urlopen):
        """ทดสอบกรณี city ไม่มีอยู่"""
        import urllib.error
        
        mock_urlopen.side_effect = urllib.error.HTTPError(
            url='', code=404, msg='Not Found', hdrs={}, fp=None
        )
        
        with self.assertRaises(ValueError) as ctx:
            self.api.get_current_weather('NonExistentCity')
        
        self.assertIn('NonExistentCity', str(ctx.exception))
    
    @patch('urllib.request.urlopen')
    def test_get_weather_network_error(self, mock_urlopen):
        """ทดสอบกรณี network error"""
        import urllib.error
        
        mock_urlopen.side_effect = urllib.error.URLError("Connection refused")
        
        with self.assertRaises(ConnectionError):
            self.api.get_current_weather('Bangkok')
    
    @patch('urllib.request.urlopen')
    def test_api_called_with_correct_url(self, mock_urlopen):
        """ตรวจสอบว่า URL ถูกต้อง"""
        mock_data = {
            'name': 'Chiang Mai',
            'main': {'temp': 300, 'humidity': 60},
            'weather': [{'description': 'sunny'}]
        }
        mock_urlopen.return_value = self._make_mock_response(mock_data)
        
        self.api.get_current_weather('Chiang Mai')
        
        # ตรวจสอบ URL ที่ถูกเรียก
        call_args = mock_urlopen.call_args
        called_url = call_args[0][0]
        
        self.assertIn('Chiang Mai', called_url)
        self.assertIn('test-key-123', called_url)
        self.assertIn('weather', called_url)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

---

## 15. Integration Tests

```python
import unittest
import sqlite3
import tempfile
import os

# ตัวอย่างที่ 23: Integration test กับ database จริง
class UserRepository:
    def __init__(self, db_path):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    username TEXT UNIQUE NOT NULL,
                    email TEXT UNIQUE NOT NULL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
    
    def create_user(self, username, email):
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute(
                "INSERT INTO users (username, email) VALUES (?, ?)",
                (username, email)
            )
            return cursor.lastrowid
    
    def get_user(self, user_id):
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            row = conn.execute(
                "SELECT * FROM users WHERE id = ?", (user_id,)
            ).fetchone()
            return dict(row) if row else None
    
    def get_by_username(self, username):
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            row = conn.execute(
                "SELECT * FROM users WHERE username = ?", (username,)
            ).fetchone()
            return dict(row) if row else None
    
    def delete_user(self, user_id):
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.execute("DELETE FROM users WHERE id = ?", (user_id,))
            return cursor.rowcount > 0

class TestUserRepositoryIntegration(unittest.TestCase):
    """Integration tests ที่ใช้ database จริง (แต่เป็น temp)"""
    
    def setUp(self):
        """สร้าง temporary database"""
        self.db_file = tempfile.mktemp(suffix='.db')
        self.repo = UserRepository(self.db_file)
    
    def tearDown(self):
        """ลบ temporary database"""
        if os.path.exists(self.db_file):
            os.unlink(self.db_file)
    
    def test_create_and_retrieve_user(self):
        """Integration: สร้างและดึงข้อมูล user"""
        user_id = self.repo.create_user("alice", "alice@test.com")
        
        user = self.repo.get_user(user_id)
        
        self.assertIsNotNone(user)
        self.assertEqual(user['username'], "alice")
        self.assertEqual(user['email'], "alice@test.com")
    
    def test_duplicate_username_fails(self):
        """Integration: ทดสอบ unique constraint"""
        self.repo.create_user("bob", "bob@test.com")
        
        with self.assertRaises(sqlite3.IntegrityError):
            self.repo.create_user("bob", "bob2@test.com")
    
    def test_delete_user(self):
        """Integration: ลบ user"""
        user_id = self.repo.create_user("charlie", "charlie@test.com")
        
        result = self.repo.delete_user(user_id)
        self.assertTrue(result)
        
        user = self.repo.get_user(user_id)
        self.assertIsNone(user)
    
    def test_get_nonexistent_user(self):
        """Integration: ดึง user ที่ไม่มี"""
        user = self.repo.get_user(99999)
        self.assertIsNone(user)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

---

## 16. Test Fixtures

```python
import unittest

# ตัวอย่างที่ 24: ใช้ subTest สำหรับ parameterized-like testing
class TestWithSubTest(unittest.TestCase):
    
    def test_multiple_inputs(self):
        """subTest ให้รัน test หลาย inputs แบบ independent"""
        test_cases = [
            (0, True),
            (1, False),
            (2, True),
            (3, False),
            (100, True),
        ]
        
        def is_even(n):
            return n % 2 == 0
        
        for num, expected in test_cases:
            with self.subTest(num=num):
                result = is_even(num)
                self.assertEqual(
                    result, expected,
                    f"is_even({num}) should be {expected}, got {result}"
                )
    
    def test_string_operations(self):
        """Test หลาย string operations"""
        test_data = [
            ("hello", "HELLO", "upper"),
            ("WORLD", "world", "lower"),
            ("  spaces  ", "spaces", "strip"),
        ]
        
        for input_str, expected, operation in test_data:
            with self.subTest(operation=operation, input=input_str):
                result = getattr(input_str, operation)()
                self.assertEqual(result, expected)
```

---

## แบบฝึกหัด

### ข้อที่ 1: Testing BankAccount

**เฉลย:**
```python
import unittest

class BankAccount:
    def __init__(self, owner: str, initial_balance: float = 0):
        if initial_balance < 0:
            raise ValueError("Initial balance cannot be negative")
        self.owner = owner
        self._balance = initial_balance
        self._transactions = []
    
    def deposit(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        self._transactions.append(('deposit', amount))
        return self._balance
    
    def withdraw(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self._balance:
            raise ValueError("Insufficient funds")
        self._balance -= amount
        self._transactions.append(('withdrawal', amount))
        return self._balance
    
    @property
    def balance(self) -> float:
        return self._balance
    
    def get_transactions(self):
        return self._transactions.copy()

class TestBankAccount(unittest.TestCase):
    def setUp(self):
        self.account = BankAccount("Alice", 1000)
    
    def test_initial_balance(self):
        self.assertEqual(self.account.balance, 1000)
    
    def test_deposit_increases_balance(self):
        new_balance = self.account.deposit(500)
        self.assertEqual(new_balance, 1500)
        self.assertEqual(self.account.balance, 1500)
    
    def test_deposit_negative_raises(self):
        with self.assertRaises(ValueError):
            self.account.deposit(-100)
    
    def test_withdraw_decreases_balance(self):
        new_balance = self.account.withdraw(300)
        self.assertEqual(new_balance, 700)
    
    def test_withdraw_insufficient_funds(self):
        with self.assertRaises(ValueError) as ctx:
            self.account.withdraw(2000)
        self.assertIn("Insufficient", str(ctx.exception))
    
    def test_transactions_recorded(self):
        self.account.deposit(200)
        self.account.withdraw(100)
        
        transactions = self.account.get_transactions()
        self.assertEqual(len(transactions), 2)
        self.assertEqual(transactions[0], ('deposit', 200))
        self.assertEqual(transactions[1], ('withdrawal', 100))
    
    def test_negative_initial_balance_raises(self):
        with self.assertRaises(ValueError):
            BankAccount("Bob", -100)

suite = unittest.TestLoader().loadTestsFromTestCase(TestBankAccount)
runner = unittest.TextTestRunner(verbosity=2)
result = runner.run(suite)
print(f"\nResult: {'PASSED' if result.wasSuccessful() else 'FAILED'}")
```

---

### ข้อที่ 2: Testing Email Validator

**เฉลย:**
```python
import unittest
import re

def validate_email(email: str) -> bool:
    """Validate email format"""
    if not email or not isinstance(email, str):
        return False
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def validate_email_strict(email: str) -> tuple:
    """Validate email with detailed errors"""
    errors = []
    if not email:
        errors.append("Email cannot be empty")
        return False, errors
    if '@' not in email:
        errors.append("Missing @ symbol")
    parts = email.split('@')
    if len(parts) == 2:
        local, domain = parts
        if not local:
            errors.append("Local part is empty")
        if '.' not in domain:
            errors.append("Domain must have a TLD")
    return len(errors) == 0, errors

class TestEmailValidator(unittest.TestCase):
    
    def test_valid_emails(self):
        valid = ['user@example.com', 'user.name@domain.co.uk', 'user+tag@gmail.com']
        for email in valid:
            with self.subTest(email=email):
                self.assertTrue(validate_email(email), f"Should be valid: {email}")
    
    def test_invalid_emails(self):
        invalid = ['', 'notanemail', '@nodomain.com', 'user@', 'user@domain']
        for email in invalid:
            with self.subTest(email=email):
                self.assertFalse(validate_email(email), f"Should be invalid: {email}")
    
    def test_strict_validator(self):
        is_valid, errors = validate_email_strict("user@example.com")
        self.assertTrue(is_valid)
        self.assertEqual(errors, [])
        
        is_valid, errors = validate_email_strict("")
        self.assertFalse(is_valid)
        self.assertIn("cannot be empty", errors[0].lower())

suite = unittest.TestLoader().loadTestsFromTestCase(TestEmailValidator)
unittest.TextTestRunner(verbosity=2).run(suite)
```

---

### ข้อที่ 3-10: แบบฝึกหัดเพิ่มเติม

### ข้อที่ 3: Testing Shopping Cart

**เฉลย:**
```python
import unittest
from unittest.mock import patch, Mock

class Product:
    def __init__(self, id, name, price, stock):
        self.id = id
        self.name = name
        self.price = price
        self.stock = stock

class ShoppingCart:
    def __init__(self):
        self.items = {}  # product_id -> {product, quantity}
    
    def add_item(self, product: Product, quantity: int = 1):
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        if quantity > product.stock:
            raise ValueError(f"Insufficient stock: {product.stock} available")
        if product.id in self.items:
            self.items[product.id]['quantity'] += quantity
        else:
            self.items[product.id] = {'product': product, 'quantity': quantity}
    
    def remove_item(self, product_id: int):
        if product_id not in self.items:
            raise KeyError(f"Product {product_id} not in cart")
        del self.items[product_id]
    
    def get_total(self) -> float:
        return sum(
            item['product'].price * item['quantity']
            for item in self.items.values()
        )
    
    def apply_discount(self, percent: float) -> float:
        if not 0 <= percent <= 100:
            raise ValueError("Discount must be 0-100%")
        return self.get_total() * (1 - percent / 100)
    
    def item_count(self) -> int:
        return sum(item['quantity'] for item in self.items.values())

class TestShoppingCart(unittest.TestCase):
    def setUp(self):
        self.cart = ShoppingCart()
        self.phone = Product(1, "iPhone", 35000, 10)
        self.laptop = Product(2, "MacBook", 89000, 5)
    
    def test_add_item(self):
        self.cart.add_item(self.phone, 2)
        self.assertEqual(self.cart.item_count(), 2)
    
    def test_add_duplicate_item(self):
        self.cart.add_item(self.phone, 1)
        self.cart.add_item(self.phone, 2)
        self.assertEqual(self.cart.items[1]['quantity'], 3)
    
    def test_add_exceeds_stock(self):
        with self.assertRaises(ValueError) as ctx:
            self.cart.add_item(self.phone, 100)
        self.assertIn("stock", str(ctx.exception).lower())
    
    def test_remove_item(self):
        self.cart.add_item(self.phone)
        self.cart.remove_item(1)
        self.assertNotIn(1, self.cart.items)
    
    def test_get_total(self):
        self.cart.add_item(self.phone, 2)
        self.cart.add_item(self.laptop, 1)
        expected = 35000 * 2 + 89000 * 1
        self.assertEqual(self.cart.get_total(), expected)
    
    def test_apply_discount(self):
        self.cart.add_item(self.phone, 1)
        discounted = self.cart.apply_discount(10)
        self.assertAlmostEqual(discounted, 35000 * 0.9)
    
    def test_invalid_discount(self):
        self.cart.add_item(self.phone)
        with self.assertRaises(ValueError):
            self.cart.apply_discount(101)

suite = unittest.TestLoader().loadTestsFromTestCase(TestShoppingCart)
unittest.TextTestRunner(verbosity=2).run(suite)
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

1. **unittest module**: TestCase, setUp/tearDown, setUpClass/tearDownClass
2. **Assertions**: assertEqual, assertTrue, assertRaises และอื่นๆ ครบทุกชนิด
3. **Test Suites**: การจัดกลุ่ม tests
4. **Test Discovery**: ค้นหา tests อัตโนมัติ
5. **Mock**: แทนที่ dependencies ด้วย fake objects
6. **patch**: แทนที่ modules/functions ระหว่าง test
7. **MagicMock**: Mock ที่รองรับ magic methods
8. **Side Effects**: Exceptions, lists, functions
9. **Integration Tests**: ทดสอบ components ร่วมกัน

การเขียน tests ที่ดีต้องเป็น FIRST:
- **Fast**: รันเร็ว
- **Independent**: ไม่พึ่งพากัน
- **Repeatable**: ได้ผลเหมือนกันทุกครั้ง
- **Self-validating**: บอกได้ว่า pass หรือ fail
- **Timely**: เขียนพร้อมกับ code
