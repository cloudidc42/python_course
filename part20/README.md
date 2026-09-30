# Part 20: Project - Scientific Calculator & Quiz Game

## สารบัญ
1. [โปรเจกต์ 1: Scientific Calculator](#โปรเจกต์-1-scientific-calculator)
   - [โครงสร้างโปรแกรม](#โครงสร้างโปรแกรม)
   - [Calculator Class](#calculator-class)
   - [Expression Parser](#expression-parser)
   - [Unit Converter](#unit-converter)
   - [History Feature](#history-feature)
   - [โค้ดสมบูรณ์](#โค้ดสมบูรณ์-calculator)
   - [วิธีรัน](#วิธีรัน-calculator)
2. [โปรเจกต์ 2: Quiz Game](#โปรเจกต์-2-quiz-game)
   - [โครงสร้างโปรแกรม](#โครงสร้างโปรแกรม-1)
   - [Question Management](#question-management)
   - [Score & Timer](#score--timer)
   - [Leaderboard](#leaderboard)
   - [File-based Storage](#file-based-storage)
   - [โค้ดสมบูรณ์](#โค้ดสมบูรณ์-quiz-game)
   - [วิธีรัน](#วิธีรัน-quiz-game)
3. [Ideas สำหรับการต่อยอด](#ideas-สำหรับการต่อยอด)

---

## โปรเจกต์ 1: Scientific Calculator

### โครงสร้างโปรแกรม

```
scientific_calculator/
├── calculator.py       # หลัก: Calculator class
├── parser.py           # Expression parser
├── converter.py        # Unit converter
├── history.py          # History manager
└── main.py             # Entry point
```

---

### Calculator Class

```python
# calculator.py - Calculator class หลัก

import math
import statistics
from typing import Union, List, Optional

Number = Union[int, float]

class CalculatorError(Exception):
    """Base exception สำหรับ Calculator"""
    pass

class DivisionByZeroError(CalculatorError):
    """Exception สำหรับการหารด้วยศูนย์"""
    pass

class InvalidInputError(CalculatorError):
    """Exception สำหรับ input ไม่ถูกต้อง"""
    pass

class MathDomainError(CalculatorError):
    """Exception สำหรับ math domain error"""
    pass


class Calculator:
    """Scientific Calculator พร้อม history และ memory"""
    
    def __init__(self):
        self._history: List[dict] = []
        self._memory: float = 0.0
        self._last_result: Optional[float] = None
    
    # === Basic Operations ===
    
    def add(self, a: Number, b: Number) -> float:
        """บวก a + b"""
        result = float(a) + float(b)
        self._record("add", [a, b], result, f"{a} + {b}")
        return result
    
    def subtract(self, a: Number, b: Number) -> float:
        """ลบ a - b"""
        result = float(a) - float(b)
        self._record("subtract", [a, b], result, f"{a} - {b}")
        return result
    
    def multiply(self, a: Number, b: Number) -> float:
        """คูณ a * b"""
        result = float(a) * float(b)
        self._record("multiply", [a, b], result, f"{a} × {b}")
        return result
    
    def divide(self, a: Number, b: Number) -> float:
        """หาร a / b"""
        if b == 0:
            raise DivisionByZeroError(f"ไม่สามารถหาร {a} ด้วยศูนย์")
        result = float(a) / float(b)
        self._record("divide", [a, b], result, f"{a} ÷ {b}")
        return result
    
    def floor_divide(self, a: Number, b: Number) -> int:
        """หารเอาจำนวนเต็ม a // b"""
        if b == 0:
            raise DivisionByZeroError("ไม่สามารถหารด้วยศูนย์")
        result = int(a) // int(b)
        self._record("floor_divide", [a, b], result, f"{a} // {b}")
        return result
    
    def modulo(self, a: Number, b: Number) -> float:
        """หารเอาเศษ a % b"""
        if b == 0:
            raise DivisionByZeroError("ไม่สามารถหารด้วยศูนย์")
        result = float(a) % float(b)
        self._record("modulo", [a, b], result, f"{a} % {b}")
        return result
    
    def power(self, base: Number, exponent: Number) -> float:
        """ยกกำลัง base ^ exponent"""
        try:
            result = float(base) ** float(exponent)
            if math.isnan(result) or math.isinf(result):
                raise OverflowError("ผลลัพธ์ไม่สิ้นสุด")
            self._record("power", [base, exponent], result, f"{base} ^ {exponent}")
            return result
        except OverflowError as e:
            raise CalculatorError(f"Overflow: {e}")
    
    # === Scientific Functions ===
    
    def sqrt(self, x: Number) -> float:
        """รากที่สอง"""
        if x < 0:
            raise MathDomainError(f"ไม่สามารถหาราก√{x} (ค่าลบ)")
        result = math.sqrt(float(x))
        self._record("sqrt", [x], result, f"√{x}")
        return result
    
    def cbrt(self, x: Number) -> float:
        """รากที่สาม"""
        result = math.copysign(abs(x) ** (1/3), x)
        self._record("cbrt", [x], result, f"∛{x}")
        return result
    
    def nth_root(self, x: Number, n: Number) -> float:
        """รากที่ n"""
        if n == 0:
            raise InvalidInputError("n ต้องไม่เป็น 0")
        if x < 0 and n % 2 == 0:
            raise MathDomainError(f"ไม่สามารถหารากที่ {n} ของค่าลบ")
        result = math.copysign(abs(x) ** (1/n), x)
        self._record("nth_root", [x, n], result, f"{n}√{x}")
        return result
    
    def absolute(self, x: Number) -> float:
        """ค่าสัมบูรณ์"""
        result = abs(float(x))
        self._record("abs", [x], result, f"|{x}|")
        return result
    
    def factorial(self, n: Number) -> int:
        """Factorial n!"""
        n = int(n)
        if n < 0:
            raise InvalidInputError("Factorial ต้องการ n >= 0")
        if n > 1000:
            raise InvalidInputError("n ใหญ่เกินไป (max 1000)")
        result = math.factorial(n)
        self._record("factorial", [n], result, f"{n}!")
        return result
    
    # === Trigonometry (degrees) ===
    
    def sin(self, degrees: Number) -> float:
        """sin (input เป็นองศา)"""
        result = math.sin(math.radians(float(degrees)))
        result = round(result, 10)  # แก้ floating point issues
        self._record("sin", [degrees], result, f"sin({degrees}°)")
        return result
    
    def cos(self, degrees: Number) -> float:
        """cos (input เป็นองศา)"""
        result = math.cos(math.radians(float(degrees)))
        result = round(result, 10)
        self._record("cos", [degrees], result, f"cos({degrees}°)")
        return result
    
    def tan(self, degrees: Number) -> float:
        """tan (input เป็นองศา)"""
        if degrees % 180 == 90:
            raise MathDomainError(f"tan({degrees}°) ไม่ defined")
        result = math.tan(math.radians(float(degrees)))
        result = round(result, 10)
        self._record("tan", [degrees], result, f"tan({degrees}°)")
        return result
    
    def asin(self, x: Number) -> float:
        """arcsin (ผลลัพธ์เป็นองศา)"""
        if not -1 <= x <= 1:
            raise MathDomainError(f"arcsin ต้องการ -1 ≤ x ≤ 1 (ได้รับ {x})")
        result = math.degrees(math.asin(float(x)))
        self._record("asin", [x], result, f"arcsin({x})")
        return result
    
    def acos(self, x: Number) -> float:
        """arccos (ผลลัพธ์เป็นองศา)"""
        if not -1 <= x <= 1:
            raise MathDomainError(f"arccos ต้องการ -1 ≤ x ≤ 1 (ได้รับ {x})")
        result = math.degrees(math.acos(float(x)))
        self._record("acos", [x], result, f"arccos({x})")
        return result
    
    def atan(self, x: Number) -> float:
        """arctan (ผลลัพธ์เป็นองศา)"""
        result = math.degrees(math.atan(float(x)))
        self._record("atan", [x], result, f"arctan({x})")
        return result
    
    def atan2(self, y: Number, x: Number) -> float:
        """atan2(y, x) (ผลลัพธ์เป็นองศา)"""
        result = math.degrees(math.atan2(float(y), float(x)))
        self._record("atan2", [y, x], result, f"atan2({y}, {x})")
        return result
    
    # === Logarithm ===
    
    def log(self, x: Number, base: Number = math.e) -> float:
        """Logarithm"""
        if x <= 0:
            raise MathDomainError(f"log ต้องการ x > 0 (ได้รับ {x})")
        if base <= 0 or base == 1:
            raise InvalidInputError("base ต้องมากกว่า 0 และไม่เท่ากับ 1")
        result = math.log(float(x), float(base))
        self._record("log", [x, base], result, f"log_{base}({x})")
        return result
    
    def log10(self, x: Number) -> float:
        """Log base 10"""
        if x <= 0:
            raise MathDomainError(f"log10 ต้องการ x > 0")
        result = math.log10(float(x))
        self._record("log10", [x], result, f"log10({x})")
        return result
    
    def log2(self, x: Number) -> float:
        """Log base 2"""
        if x <= 0:
            raise MathDomainError(f"log2 ต้องการ x > 0")
        result = math.log2(float(x))
        self._record("log2", [x], result, f"log2({x})")
        return result
    
    def exp(self, x: Number) -> float:
        """e^x"""
        try:
            result = math.exp(float(x))
            if math.isinf(result):
                raise OverflowError("Overflow")
            self._record("exp", [x], result, f"e^{x}")
            return result
        except OverflowError:
            raise CalculatorError(f"e^{x} มีค่าใหญ่เกินไป")
    
    # === Statistics ===
    
    def mean(self, data: List[Number]) -> float:
        """ค่าเฉลี่ย"""
        if not data:
            raise InvalidInputError("ข้อมูลต้องไม่ว่าง")
        result = statistics.mean(data)
        self._record("mean", data, result, f"mean{data}")
        return result
    
    def median(self, data: List[Number]) -> float:
        """ค่ากลาง"""
        if not data:
            raise InvalidInputError("ข้อมูลต้องไม่ว่าง")
        result = statistics.median(data)
        self._record("median", data, result, f"median{data}")
        return result
    
    def std_dev(self, data: List[Number]) -> float:
        """ส่วนเบี่ยงเบนมาตรฐาน"""
        if len(data) < 2:
            raise InvalidInputError("ต้องการข้อมูลอย่างน้อย 2 ตัว")
        result = statistics.stdev(data)
        self._record("stdev", data, result, f"stdev{data}")
        return result
    
    # === Memory Operations ===
    
    def memory_store(self, value: Number):
        """เก็บค่าใน memory"""
        self._memory = float(value)
        print(f"เก็บ {value} ใน memory แล้ว")
    
    def memory_recall(self) -> float:
        """เรียกค่าจาก memory"""
        return self._memory
    
    def memory_clear(self):
        """ล้าง memory"""
        self._memory = 0.0
        print("ล้าง memory แล้ว")
    
    def memory_add(self, value: Number):
        """บวกค่าเข้า memory"""
        self._memory += float(value)
    
    # === History ===
    
    def _record(self, operation: str, inputs: list, result, expression: str):
        """บันทึก operation ลง history"""
        import datetime
        
        self._last_result = float(result) if result is not None else None
        self._history.append({
            "id": len(self._history) + 1,
            "operation": operation,
            "inputs": inputs,
            "result": result,
            "expression": expression,
            "timestamp": datetime.datetime.now().isoformat(),
        })
    
    def get_history(self, last_n: int = None) -> List[dict]:
        """ดู history"""
        if last_n is None:
            return self._history.copy()
        return self._history[-last_n:]
    
    def clear_history(self):
        """ล้าง history"""
        self._history.clear()
        print("ล้าง history แล้ว")
    
    def show_history(self, last_n: int = 10):
        """แสดง history"""
        history = self.get_history(last_n)
        if not history:
            print("ยังไม่มี history")
            return
        
        print(f"\n=== History (ล่าสุด {len(history)} รายการ) ===")
        for item in history:
            result_str = (
                f"{item['result']:,.4f}" 
                if isinstance(item['result'], float) 
                else str(item['result'])
            )
            print(f"  #{item['id']:3d}: {item['expression']} = {result_str}")
    
    @property
    def last_result(self) -> Optional[float]:
        """ผลลัพธ์ล่าสุด"""
        return self._last_result
    
    @property
    def memory(self) -> float:
        """ค่าใน memory"""
        return self._memory
```

---

### Expression Parser

```python
# parser.py - Simple Expression Parser

import re
import math
from typing import Union

class ParserError(Exception):
    pass

class ExpressionParser:
    """Parser สำหรับคำนวณ mathematical expressions"""
    
    CONSTANTS = {
        "pi": math.pi,
        "e": math.e,
        "tau": math.tau,
        "inf": math.inf,
    }
    
    FUNCTIONS = {
        "sin": lambda x: math.sin(math.radians(x)),
        "cos": lambda x: math.cos(math.radians(x)),
        "tan": lambda x: math.tan(math.radians(x)),
        "asin": lambda x: math.degrees(math.asin(x)),
        "acos": lambda x: math.degrees(math.acos(x)),
        "atan": lambda x: math.degrees(math.atan(x)),
        "sqrt": math.sqrt,
        "cbrt": lambda x: math.copysign(abs(x)**(1/3), x),
        "abs": abs,
        "log": math.log10,
        "ln": math.log,
        "log2": math.log2,
        "exp": math.exp,
        "ceil": math.ceil,
        "floor": math.floor,
        "round": round,
        "factorial": math.factorial,
    }
    
    def __init__(self, calc=None):
        self.calc = calc
    
    def evaluate(self, expression: str) -> float:
        """คำนวณ expression"""
        # ทำความสะอาด expression
        expr = expression.strip()
        
        if not expr:
            raise ParserError("expression ว่าง")
        
        # แทนที่ constants
        for const_name, const_value in self.CONSTANTS.items():
            expr = re.sub(r'\b' + const_name + r'\b', str(const_value), expr)
        
        # แทนที่ ^ ด้วย **
        expr = expr.replace("^", "**")
        
        # ตรวจสอบความปลอดภัย
        self._validate_expression(expr)
        
        # แทนที่ฟังก์ชัน
        for func_name, func in self.FUNCTIONS.items():
            pattern = func_name + r'\('
            if re.search(pattern, expr):
                expr = self._replace_function(expr, func_name, func)
        
        try:
            # คำนวณ
            result = eval(expr)
            return float(result)
        except ZeroDivisionError:
            raise ParserError("หารด้วยศูนย์")
        except Exception as e:
            raise ParserError(f"ไม่สามารถคำนวณ '{expression}': {e}")
    
    def _validate_expression(self, expr: str):
        """ตรวจสอบความปลอดภัยของ expression"""
        # ตรวจสอบ characters ที่อนุญาต
        allowed = set('0123456789.+-*/()%** \t.eE')
        allowed.update(set('abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_'))
        
        # ตรวจสอบ dangerous keywords
        dangerous = ['import', 'exec', 'eval', 'open', '__', 'os', 'sys', 'subprocess']
        expr_lower = expr.lower()
        for keyword in dangerous:
            if keyword in expr_lower:
                raise ParserError(f"Expression ไม่ปลอดภัย: ห้ามใช้ '{keyword}'")
    
    def _replace_function(self, expr: str, func_name: str, func) -> str:
        """แทนที่ function call ใน expression"""
        # ง่ายๆ: ใช้ direct eval approach
        return expr  # ใช้ built-in eval approach
    
    def evaluate_safe(self, expression: str) -> Union[float, str]:
        """คำนวณพร้อม error handling"""
        try:
            return self.evaluate(expression)
        except ParserError as e:
            return f"Error: {e}"
        except Exception as e:
            return f"Error: {e}"
```

---

### Unit Converter

```python
# converter.py - Unit Converter

class UnitConverter:
    """แปลงหน่วย"""
    
    # Conversion factors (to base unit)
    CONVERSIONS = {
        "length": {
            "meters": 1.0,
            "kilometers": 1000.0,
            "centimeters": 0.01,
            "millimeters": 0.001,
            "inches": 0.0254,
            "feet": 0.3048,
            "yards": 0.9144,
            "miles": 1609.344,
            "nautical_miles": 1852.0,
            "light_years": 9.461e15,
        },
        "weight": {
            "kilograms": 1.0,
            "grams": 0.001,
            "milligrams": 0.000001,
            "pounds": 0.453592,
            "ounces": 0.0283495,
            "tons": 1000.0,
            "stone": 6.35029,
        },
        "temperature": None,  # ใช้ formula แทน
        "area": {
            "square_meters": 1.0,
            "square_kilometers": 1e6,
            "square_centimeters": 0.0001,
            "square_feet": 0.092903,
            "square_yards": 0.836127,
            "acres": 4046.86,
            "hectares": 10000.0,
            "square_miles": 2.59e6,
        },
        "volume": {
            "liters": 1.0,
            "milliliters": 0.001,
            "cubic_meters": 1000.0,
            "gallons_us": 3.78541,
            "gallons_uk": 4.54609,
            "cups": 0.236588,
            "pints": 0.473176,
            "quarts": 0.946353,
            "cubic_feet": 28.3168,
            "cubic_inches": 0.0163871,
        },
        "speed": {
            "meters_per_second": 1.0,
            "kilometers_per_hour": 0.277778,
            "miles_per_hour": 0.44704,
            "feet_per_second": 0.3048,
            "knots": 0.514444,
        },
        "digital": {
            "bytes": 1.0,
            "kilobytes": 1024.0,
            "megabytes": 1048576.0,
            "gigabytes": 1073741824.0,
            "terabytes": 1099511627776.0,
            "bits": 0.125,
            "kilobits": 128.0,
            "megabits": 131072.0,
        },
    }
    
    def convert(self, value: float, from_unit: str, to_unit: str, category: str) -> float:
        """แปลงหน่วย"""
        
        if category == "temperature":
            return self._convert_temperature(value, from_unit, to_unit)
        
        if category not in self.CONVERSIONS:
            raise ValueError(f"ไม่รู้จัก category: {category}")
        
        units = self.CONVERSIONS[category]
        
        if from_unit not in units:
            raise ValueError(f"ไม่รู้จัก unit: {from_unit}")
        
        if to_unit not in units:
            raise ValueError(f"ไม่รู้จัก unit: {to_unit}")
        
        # แปลงเป็น base unit ก่อน แล้วแปลงไป target
        base_value = value * units[from_unit]
        result = base_value / units[to_unit]
        
        return result
    
    def _convert_temperature(self, value: float, from_unit: str, to_unit: str) -> float:
        """แปลงอุณหภูมิ"""
        # แปลงเป็น Celsius ก่อน
        if from_unit == "celsius":
            celsius = value
        elif from_unit == "fahrenheit":
            celsius = (value - 32) * 5 / 9
        elif from_unit == "kelvin":
            celsius = value - 273.15
        else:
            raise ValueError(f"ไม่รู้จัก temperature unit: {from_unit}")
        
        # แปลงจาก Celsius ไป target
        if to_unit == "celsius":
            return celsius
        elif to_unit == "fahrenheit":
            return celsius * 9 / 5 + 32
        elif to_unit == "kelvin":
            return celsius + 273.15
        else:
            raise ValueError(f"ไม่รู้จัก temperature unit: {to_unit}")
    
    def get_categories(self) -> list:
        """ดู categories ทั้งหมด"""
        return list(self.CONVERSIONS.keys()) + ["temperature"]
    
    def get_units(self, category: str) -> list:
        """ดู units ใน category"""
        if category == "temperature":
            return ["celsius", "fahrenheit", "kelvin"]
        if category in self.CONVERSIONS:
            return list(self.CONVERSIONS[category].keys())
        return []


# ทดสอบ
converter = UnitConverter()

print("=== Unit Converter Tests ===")
conversions = [
    (100, "kilometers", "miles", "length"),
    (1, "kilograms", "pounds", "weight"),
    (100, "celsius", "fahrenheit", "temperature"),
    (0, "celsius", "kelvin", "temperature"),
    (1, "gigabytes", "megabytes", "digital"),
    (60, "miles_per_hour", "kilometers_per_hour", "speed"),
]

for value, from_unit, to_unit, cat in conversions:
    result = converter.convert(value, from_unit, to_unit, cat)
    print(f"{value} {from_unit} = {result:.4f} {to_unit}")
```

---

### โค้ดสมบูรณ์ Calculator

```python
# scientific_calculator.py - โค้ดสมบูรณ์พร้อมรัน

import math
import statistics
import re
import datetime
from typing import Union, List, Optional, Dict, Any


# ===== Custom Exceptions =====

class CalculatorError(Exception):
    pass

class DivisionByZeroError(CalculatorError):
    pass

class InvalidInputError(CalculatorError):
    pass

class MathDomainError(CalculatorError):
    pass


# ===== Calculator Class =====

class Calculator:
    """Scientific Calculator"""
    
    def __init__(self):
        self._history: List[Dict] = []
        self._memory: float = 0.0
        self._last_result: Optional[float] = None
    
    # --- Basic Operations ---
    
    def add(self, a, b):
        result = float(a) + float(b)
        self._record(f"{a} + {b}", result)
        return result
    
    def subtract(self, a, b):
        result = float(a) - float(b)
        self._record(f"{a} - {b}", result)
        return result
    
    def multiply(self, a, b):
        result = float(a) * float(b)
        self._record(f"{a} × {b}", result)
        return result
    
    def divide(self, a, b):
        if b == 0:
            raise DivisionByZeroError(f"ไม่สามารถหาร {a} ด้วย 0")
        result = float(a) / float(b)
        self._record(f"{a} ÷ {b}", result)
        return result
    
    def power(self, base, exponent):
        try:
            result = float(base) ** float(exponent)
            if math.isinf(result):
                raise OverflowError("ผลลัพธ์ใหญ่เกินไป")
            self._record(f"{base}^{exponent}", result)
            return result
        except OverflowError as e:
            raise CalculatorError(str(e))
    
    def sqrt(self, x):
        if x < 0:
            raise MathDomainError(f"√{x} ไม่ defined สำหรับจำนวนจริง")
        result = math.sqrt(float(x))
        self._record(f"√{x}", result)
        return result
    
    def factorial(self, n):
        n = int(n)
        if n < 0:
            raise InvalidInputError("Factorial ต้องการ n >= 0")
        if n > 170:
            raise InvalidInputError("n ใหญ่เกินไป")
        result = math.factorial(n)
        self._record(f"{n}!", result)
        return result
    
    # --- Trig (degrees) ---
    
    def sin(self, deg):
        result = round(math.sin(math.radians(float(deg))), 10)
        self._record(f"sin({deg}°)", result)
        return result
    
    def cos(self, deg):
        result = round(math.cos(math.radians(float(deg))), 10)
        self._record(f"cos({deg}°)", result)
        return result
    
    def tan(self, deg):
        if deg % 180 == 90:
            raise MathDomainError(f"tan({deg}°) ไม่ defined")
        result = round(math.tan(math.radians(float(deg))), 10)
        self._record(f"tan({deg}°)", result)
        return result
    
    # --- Log & Exp ---
    
    def log(self, x, base=10):
        if x <= 0:
            raise MathDomainError(f"log({x}) ไม่ defined")
        result = math.log(float(x), float(base))
        self._record(f"log_{base}({x})", result)
        return result
    
    def ln(self, x):
        if x <= 0:
            raise MathDomainError(f"ln({x}) ไม่ defined")
        result = math.log(float(x))
        self._record(f"ln({x})", result)
        return result
    
    def exp(self, x):
        try:
            result = math.exp(float(x))
            if math.isinf(result):
                raise OverflowError()
            self._record(f"e^{x}", result)
            return result
        except OverflowError:
            raise CalculatorError(f"e^{x} overflow")
    
    # --- Statistics ---
    
    def calc_mean(self, data: List):
        if not data:
            raise InvalidInputError("ข้อมูลต้องไม่ว่าง")
        result = statistics.mean(data)
        self._record(f"mean{data}", result)
        return result
    
    def calc_median(self, data: List):
        if not data:
            raise InvalidInputError("ข้อมูลต้องไม่ว่าง")
        result = statistics.median(data)
        self._record(f"median{data}", result)
        return result
    
    def calc_stdev(self, data: List):
        if len(data) < 2:
            raise InvalidInputError("ต้องการข้อมูลอย่างน้อย 2 ตัว")
        result = statistics.stdev(data)
        self._record(f"stdev{data}", result)
        return result
    
    # --- Memory ---
    
    def ms(self, value):
        """Memory Store"""
        self._memory = float(value)
    
    def mr(self) -> float:
        """Memory Recall"""
        return self._memory
    
    def mc(self):
        """Memory Clear"""
        self._memory = 0.0
    
    def mplus(self, value):
        """Memory Add"""
        self._memory += float(value)
    
    # --- History ---
    
    def _record(self, expression: str, result):
        self._last_result = float(result) if isinstance(result, (int, float)) else None
        self._history.append({
            "id": len(self._history) + 1,
            "expression": expression,
            "result": result,
            "timestamp": datetime.datetime.now().strftime("%H:%M:%S"),
        })
    
    def show_history(self, n: int = 10):
        entries = self._history[-n:]
        if not entries:
            print("ยังไม่มี history")
            return
        print(f"\n{'='*40}")
        print(f"{'#':>4}  {'Expression':<25} {'Result':>12}  {'Time'}")
        print(f"{'='*40}")
        for entry in entries:
            result = entry['result']
            if isinstance(result, float):
                result_str = f"{result:>12.6g}"
            else:
                result_str = f"{str(result):>12}"
            print(f"{entry['id']:>4}  {entry['expression']:<25} {result_str}  {entry['timestamp']}")
    
    def clear_history(self):
        self._history.clear()
    
    @property
    def last_result(self):
        return self._last_result
    
    @property
    def memory(self):
        return self._memory


# ===== Unit Converter =====

class UnitConverter:
    CONVERSIONS = {
        "length": {
            "m": 1.0, "km": 1000.0, "cm": 0.01, "mm": 0.001,
            "in": 0.0254, "ft": 0.3048, "yd": 0.9144, "mi": 1609.344,
        },
        "weight": {
            "kg": 1.0, "g": 0.001, "mg": 0.000001,
            "lb": 0.453592, "oz": 0.0283495,
        },
        "volume": {
            "L": 1.0, "mL": 0.001, "m3": 1000.0,
            "gal": 3.78541, "cup": 0.236588, "pt": 0.473176,
        },
        "speed": {
            "m/s": 1.0, "km/h": 0.277778, "mph": 0.44704, "kt": 0.514444,
        },
        "digital": {
            "B": 1.0, "KB": 1024.0, "MB": 1048576.0,
            "GB": 1073741824.0, "TB": 1099511627776.0,
        },
    }
    
    def convert(self, value: float, from_unit: str, to_unit: str, category: str) -> float:
        if category == "temperature":
            return self._convert_temp(value, from_unit, to_unit)
        
        if category not in self.CONVERSIONS:
            raise ValueError(f"ไม่รู้จัก category: {category}")
        
        units = self.CONVERSIONS[category]
        if from_unit not in units or to_unit not in units:
            raise ValueError(f"ไม่รู้จัก unit")
        
        return value * units[from_unit] / units[to_unit]
    
    def _convert_temp(self, value, from_unit, to_unit):
        # to celsius
        if from_unit == "C":
            c = value
        elif from_unit == "F":
            c = (value - 32) * 5 / 9
        elif from_unit == "K":
            c = value - 273.15
        else:
            raise ValueError(f"ไม่รู้จัก temperature unit: {from_unit}")
        
        # from celsius
        if to_unit == "C": return c
        elif to_unit == "F": return c * 9 / 5 + 32
        elif to_unit == "K": return c + 273.15
        else:
            raise ValueError(f"ไม่รู้จัก temperature unit: {to_unit}")


# ===== Main Interface =====

def show_help():
    print("""
╔══════════════════════════════════════════╗
║        Scientific Calculator Help        ║
╠══════════════════════════════════════════╣
║  พื้นฐาน:                               ║
║    add 5 3      → 5 + 3                 ║
║    sub 10 4     → 10 - 4                ║
║    mul 3 4      → 3 × 4                 ║
║    div 15 3     → 15 ÷ 3               ║
║    pow 2 10     → 2^10                  ║
║                                          ║
║  วิทยาศาสตร์:                           ║
║    sqrt 16      → √16                   ║
║    fact 5       → 5!                    ║
║    sin 30       → sin(30°)              ║
║    cos 60       → cos(60°)              ║
║    log 100      → log₁₀(100)           ║
║    ln 10        → ln(10)               ║
║    exp 2        → e^2                   ║
║                                          ║
║  สถิติ:                                  ║
║    mean 1 2 3 4 5                       ║
║    median 3 1 4 1 5                     ║
║    stdev 2 4 4 4 5 5 7 9               ║
║                                          ║
║  Memory:                                 ║
║    ms (memory store last result)        ║
║    mr (memory recall)                   ║
║    mc (memory clear)                    ║
║                                          ║
║  แปลงหน่วย:                             ║
║    conv 100 km m length                 ║
║    conv 0 C F temperature               ║
║                                          ║
║  อื่นๆ:                                  ║
║    hist      → แสดง history             ║
║    clear     → ล้าง history             ║
║    help      → แสดง help               ║
║    quit/exit → ออก                      ║
╚══════════════════════════════════════════╝
    """)

def run_calculator():
    """รัน interactive calculator"""
    calc = Calculator()
    conv = UnitConverter()
    
    print("╔══════════════════════════════════╗")
    print("║    Scientific Calculator v1.0    ║")
    print("║  พิมพ์ 'help' เพื่อดูคำสั่ง    ║")
    print("╚══════════════════════════════════╝")
    
    while True:
        try:
            user_input = input("\n> ").strip()
            
            if not user_input:
                continue
            
            parts = user_input.split()
            cmd = parts[0].lower()
            
            # ออก
            if cmd in ["quit", "exit", "q"]:
                print("ขอบคุณที่ใช้งาน Scientific Calculator!")
                break
            
            # Help
            elif cmd == "help":
                show_help()
            
            # History
            elif cmd == "hist":
                calc.show_history()
            
            # Clear
            elif cmd == "clear":
                calc.clear_history()
                print("ล้าง history แล้ว")
            
            # Memory
            elif cmd == "ms":
                if calc.last_result is not None:
                    calc.ms(calc.last_result)
                    print(f"เก็บ {calc.last_result} ใน Memory")
                else:
                    print("ยังไม่มีผลลัพธ์")
            
            elif cmd == "mr":
                print(f"Memory = {calc.memory}")
            
            elif cmd == "mc":
                calc.mc()
                print("ล้าง Memory แล้ว")
            
            # Unit Conversion
            elif cmd == "conv":
                if len(parts) < 5:
                    print("รูปแบบ: conv <value> <from_unit> <to_unit> <category>")
                    continue
                value = float(parts[1])
                from_unit, to_unit, category = parts[2], parts[3], parts[4]
                result = conv.convert(value, from_unit, to_unit, category)
                print(f"  {value} {from_unit} = {result:.6g} {to_unit}")
            
            # Basic operations
            elif cmd in ["add", "+"]:
                a, b = float(parts[1]), float(parts[2])
                result = calc.add(a, b)
                print(f"  {a} + {b} = {result}")
            
            elif cmd in ["sub", "-"]:
                a, b = float(parts[1]), float(parts[2])
                result = calc.subtract(a, b)
                print(f"  {a} - {b} = {result}")
            
            elif cmd in ["mul", "*"]:
                a, b = float(parts[1]), float(parts[2])
                result = calc.multiply(a, b)
                print(f"  {a} × {b} = {result}")
            
            elif cmd in ["div", "/"]:
                a, b = float(parts[1]), float(parts[2])
                result = calc.divide(a, b)
                print(f"  {a} ÷ {b} = {result}")
            
            elif cmd in ["pow", "**"]:
                a, b = float(parts[1]), float(parts[2])
                result = calc.power(a, b)
                print(f"  {a}^{b} = {result}")
            
            # Scientific
            elif cmd == "sqrt":
                x = float(parts[1])
                result = calc.sqrt(x)
                print(f"  √{x} = {result}")
            
            elif cmd == "fact":
                n = int(parts[1])
                result = calc.factorial(n)
                print(f"  {n}! = {result}")
            
            elif cmd == "sin":
                deg = float(parts[1])
                result = calc.sin(deg)
                print(f"  sin({deg}°) = {result:.6f}")
            
            elif cmd == "cos":
                deg = float(parts[1])
                result = calc.cos(deg)
                print(f"  cos({deg}°) = {result:.6f}")
            
            elif cmd == "tan":
                deg = float(parts[1])
                result = calc.tan(deg)
                print(f"  tan({deg}°) = {result:.6f}")
            
            elif cmd == "log":
                x = float(parts[1])
                base = float(parts[2]) if len(parts) > 2 else 10
                result = calc.log(x, base)
                print(f"  log_{base}({x}) = {result:.6f}")
            
            elif cmd == "ln":
                x = float(parts[1])
                result = calc.ln(x)
                print(f"  ln({x}) = {result:.6f}")
            
            elif cmd == "exp":
                x = float(parts[1])
                result = calc.exp(x)
                print(f"  e^{x} = {result:.6f}")
            
            # Statistics
            elif cmd == "mean":
                data = [float(x) for x in parts[1:]]
                result = calc.calc_mean(data)
                print(f"  mean = {result:.4f}")
            
            elif cmd == "median":
                data = [float(x) for x in parts[1:]]
                result = calc.calc_median(data)
                print(f"  median = {result:.4f}")
            
            elif cmd == "stdev":
                data = [float(x) for x in parts[1:]]
                result = calc.calc_stdev(data)
                print(f"  stdev = {result:.4f}")
            
            else:
                print(f"ไม่รู้จักคำสั่ง: {cmd} (พิมพ์ 'help' เพื่อดูคำสั่ง)")
        
        except (CalculatorError, ValueError) as e:
            print(f"  ❌ Error: {e}")
        except IndexError:
            print(f"  ❌ ขาด arguments")
        except KeyboardInterrupt:
            print("\nออกจากโปรแกรม")
            break
        except Exception as e:
            print(f"  ❌ Unexpected error: {e}")


# Demo สำหรับทดสอบ (ไม่ interactive)
def demo_calculator():
    """Demo การทำงาน"""
    calc = Calculator()
    conv = UnitConverter()
    
    print("\n" + "="*50)
    print("Scientific Calculator Demo")
    print("="*50)
    
    # Basic
    print("\n--- Basic Operations ---")
    print(f"15 + 7 = {calc.add(15, 7)}")
    print(f"20 - 8 = {calc.subtract(20, 8)}")
    print(f"6 × 9 = {calc.multiply(6, 9)}")
    print(f"100 ÷ 4 = {calc.divide(100, 4)}")
    print(f"2^10 = {calc.power(2, 10)}")
    
    # Scientific
    print("\n--- Scientific ---")
    print(f"√144 = {calc.sqrt(144)}")
    print(f"8! = {calc.factorial(8)}")
    print(f"sin(30°) = {calc.sin(30)}")
    print(f"cos(60°) = {calc.cos(60)}")
    print(f"log10(1000) = {calc.log(1000)}")
    print(f"ln(e) = {calc.ln(math.e):.6f}")
    print(f"e^2 = {calc.exp(2):.6f}")
    
    # Statistics
    data = [4, 7, 2, 9, 4, 3, 7, 4, 1, 8]
    print(f"\n--- Statistics for {data} ---")
    print(f"Mean = {calc.calc_mean(data):.2f}")
    print(f"Median = {calc.calc_median(data):.2f}")
    print(f"StdDev = {calc.calc_stdev(data):.2f}")
    
    # Memory
    print("\n--- Memory Operations ---")
    result = calc.multiply(25, 4)
    calc.ms(result)
    print(f"25 × 4 = {result} (เก็บใน Memory)")
    calc.mplus(50)
    print(f"Memory + 50 = {calc.memory}")
    print(f"Memory Recall = {calc.mr()}")
    
    # Unit Conversion
    print("\n--- Unit Conversions ---")
    tests = [
        (100, "km", "m", "length"),
        (60, "mph", "km/h", "speed"),
        (1, "GB", "MB", "digital"),
        (100, "C", "F", "temperature"),
    ]
    for val, from_u, to_u, cat in tests:
        result = conv.convert(val, from_u, to_u, cat)
        print(f"  {val} {from_u} = {result:.4g} {to_u}")
    
    # History
    calc.show_history(5)
    
    # Error handling
    print("\n--- Error Handling ---")
    try:
        calc.divide(10, 0)
    except DivisionByZeroError as e:
        print(f"  ✓ DivisionByZero caught: {e}")
    
    try:
        calc.sqrt(-4)
    except MathDomainError as e:
        print(f"  ✓ MathDomain caught: {e}")
    
    try:
        calc.factorial(-1)
    except InvalidInputError as e:
        print(f"  ✓ InvalidInput caught: {e}")

# รัน demo
demo_calculator()

# รัน interactive mode (uncomment เพื่อใช้งาน)
# run_calculator()
```

---

### วิธีรัน Calculator

```bash
# รัน demo
python scientific_calculator.py

# รัน interactive mode
# แก้ไขบรรทัดสุดท้ายของ scientific_calculator.py:
# demo_calculator()  -> ลบออกหรือ comment
# run_calculator()   -> uncomment

# ตัวอย่างการใช้งาน interactive mode:
# > add 15 7
#   15 + 7 = 22.0
# > sqrt 144
#   √144 = 12.0
# > sin 30
#   sin(30°) = 0.500000
# > conv 100 km m length
#   100 km = 100000 m
# > hist
```

---

## โปรเจกต์ 2: Quiz Game

### โครงสร้างโปรแกรม

```
quiz_game/
├── quiz.py         # หลัก: Quiz engine
├── questions.py    # Question bank
├── player.py       # Player & Score management
├── storage.py      # File-based storage
└── main.py         # Entry point
```

---

### Question Management

```python
# questions.py - Question Bank

from dataclasses import dataclass, field
from typing import List, Optional
import random

@dataclass
class Question:
    """คำถาม"""
    id: int
    text: str
    choices: List[str]
    correct_index: int  # 0-3
    category: str
    difficulty: str  # easy/medium/hard
    explanation: str = ""
    points: int = 10
    time_limit: int = 30  # seconds
    
    @property
    def correct_answer(self) -> str:
        return self.choices[self.correct_index]
    
    def is_correct(self, choice_index: int) -> bool:
        return choice_index == self.correct_index
    
    def display(self, show_number: int = None):
        num = f"ข้อ {show_number}: " if show_number else ""
        print(f"\n{num}{self.text}")
        print(f"[{self.difficulty.upper()}] [{self.category}] ({self.points} คะแนน)")
        for i, choice in enumerate(self.choices):
            letter = "ABCD"[i]
            print(f"  {letter}) {choice}")


class QuestionBank:
    """คลังคำถาม"""
    
    SAMPLE_QUESTIONS = [
        # Python
        {
            "text": "Python ถูกสร้างโดยใคร?",
            "choices": ["James Gosling", "Guido van Rossum", "Bjarne Stroustrup", "Dennis Ritchie"],
            "correct_index": 1,
            "category": "Python",
            "difficulty": "easy",
            "explanation": "Guido van Rossum สร้าง Python ในปี 1991",
            "points": 5,
        },
        {
            "text": "ผลลัพธ์ของ `type([])` ใน Python คืออะไร?",
            "choices": ["<class 'tuple'>", "<class 'array'>", "<class 'list'>", "<class 'dict'>"],
            "correct_index": 2,
            "category": "Python",
            "difficulty": "easy",
            "explanation": "[] คือ list literal ดังนั้น type จะเป็น list",
            "points": 5,
        },
        {
            "text": "Python keyword ใดที่ใช้สร้าง generator?",
            "choices": ["return", "yield", "generate", "produce"],
            "correct_index": 1,
            "category": "Python",
            "difficulty": "medium",
            "explanation": "yield ใช้สร้าง generator function",
            "points": 10,
        },
        {
            "text": "ข้อใดเป็น Python 3 f-string ที่ถูกต้อง?",
            "choices": ["f'Hello {name}'", "'Hello {name}'.format()", "\"Hello %s\" % name", "ถูกทั้ง ก และ ข"],
            "correct_index": 3,
            "category": "Python",
            "difficulty": "medium",
            "explanation": "ทั้ง f-string และ .format() ถูกต้อง",
            "points": 10,
        },
        {
            "text": "GIL ใน Python ย่อมาจากอะไร?",
            "choices": ["Global Interface Lock", "General Input Lock", "Global Interpreter Lock", "General Interpreter Layer"],
            "correct_index": 2,
            "category": "Python",
            "difficulty": "hard",
            "explanation": "GIL = Global Interpreter Lock ป้องกัน multiple threads รัน Python bytecode พร้อมกัน",
            "points": 15,
        },
        
        # Data Structures
        {
            "text": "Data structure ใดที่มี LIFO (Last In First Out)?",
            "choices": ["Queue", "Stack", "Array", "Tree"],
            "correct_index": 1,
            "category": "Data Structures",
            "difficulty": "easy",
            "explanation": "Stack ใช้ LIFO: ข้อมูลที่เพิ่มล่าสุดจะถูกนำออกก่อน",
            "points": 5,
        },
        {
            "text": "Big O notation ของ Binary Search คืออะไร?",
            "choices": ["O(1)", "O(n)", "O(log n)", "O(n²)"],
            "correct_index": 2,
            "category": "Data Structures",
            "difficulty": "medium",
            "explanation": "Binary search แบ่งครึ่งทุกครั้ง ทำให้เป็น O(log n)",
            "points": 10,
        },
        
        # Web
        {
            "text": "HTTP method ใดที่ใช้สำหรับการสร้าง resource ใหม่?",
            "choices": ["GET", "PUT", "POST", "DELETE"],
            "correct_index": 2,
            "category": "Web",
            "difficulty": "easy",
            "explanation": "POST ใช้สำหรับสร้าง resource ใหม่",
            "points": 5,
        },
        {
            "text": "HTTP status code 404 หมายถึงอะไร?",
            "choices": ["Server Error", "Unauthorized", "Not Found", "Forbidden"],
            "correct_index": 2,
            "category": "Web",
            "difficulty": "easy",
            "explanation": "404 Not Found - ไม่พบ resource ที่ต้องการ",
            "points": 5,
        },
        
        # General
        {
            "text": "ฐาน 2 ใน Computer Science คือฐาน...?",
            "choices": ["Decimal", "Binary", "Hexadecimal", "Octal"],
            "correct_index": 1,
            "category": "Computer Science",
            "difficulty": "easy",
            "explanation": "Binary (ฐาน 2) ใช้ตัวเลข 0 และ 1",
            "points": 5,
        },
        {
            "text": "ย่อว่า URL มาจาก?",
            "choices": ["Universal Resource Locator", "Uniform Resource Locator", "Universal Record Locator", "Unified Resource Link"],
            "correct_index": 1,
            "category": "Computer Science",
            "difficulty": "easy",
            "explanation": "URL = Uniform Resource Locator",
            "points": 5,
        },
        {
            "text": "Algorithm ใดที่มี Time Complexity O(n log n) โดยเฉลี่ย?",
            "choices": ["Bubble Sort", "Insertion Sort", "Quick Sort", "Selection Sort"],
            "correct_index": 2,
            "category": "Algorithms",
            "difficulty": "medium",
            "explanation": "Quick Sort มี average case O(n log n)",
            "points": 10,
        },
        {
            "text": "Design Pattern ใดเป็น Creational Pattern?",
            "choices": ["Observer", "Strategy", "Singleton", "Decorator"],
            "correct_index": 2,
            "category": "Design Patterns",
            "difficulty": "medium",
            "explanation": "Singleton เป็น Creational Pattern ที่ให้มี instance เดียว",
            "points": 10,
        },
        {
            "text": "SOLID ใน OOP ตัว S ย่อมาจาก?",
            "choices": ["Single Responsibility", "Simple Design", "Strong Typing", "Standard Interface"],
            "correct_index": 0,
            "category": "OOP",
            "difficulty": "medium",
            "explanation": "S = Single Responsibility Principle: class ควรมีหน้าที่เดียว",
            "points": 10,
        },
        {
            "text": "Decorator Pattern ใน Python ใช้ syntax ใด?",
            "choices": ["#decorator", "@decorator", "*decorator", "&decorator"],
            "correct_index": 1,
            "category": "Python",
            "difficulty": "easy",
            "explanation": "@ ใช้สำหรับ decorator ใน Python",
            "points": 5,
        },
    ]
    
    def __init__(self):
        self.questions: List[Question] = []
        self._load_sample_questions()
    
    def _load_sample_questions(self):
        """โหลด sample questions"""
        for i, q_data in enumerate(self.SAMPLE_QUESTIONS):
            question = Question(
                id=i + 1,
                text=q_data["text"],
                choices=q_data["choices"],
                correct_index=q_data["correct_index"],
                category=q_data["category"],
                difficulty=q_data["difficulty"],
                explanation=q_data.get("explanation", ""),
                points=q_data.get("points", 10),
                time_limit=30 if q_data["difficulty"] == "easy" else 
                           20 if q_data["difficulty"] == "medium" else 15,
            )
            self.questions.append(question)
    
    def add_question(self, question: Question):
        """เพิ่มคำถาม"""
        question.id = len(self.questions) + 1
        self.questions.append(question)
    
    def get_by_category(self, category: str) -> List[Question]:
        """ดูคำถามตาม category"""
        return [q for q in self.questions if q.category.lower() == category.lower()]
    
    def get_by_difficulty(self, difficulty: str) -> List[Question]:
        """ดูคำถามตามระดับ"""
        return [q for q in self.questions if q.difficulty.lower() == difficulty.lower()]
    
    def get_random_quiz(self, n: int = 10, difficulty: str = None, category: str = None) -> List[Question]:
        """สุ่มคำถามสำหรับ quiz"""
        pool = self.questions.copy()
        
        if difficulty:
            pool = [q for q in pool if q.difficulty == difficulty]
        
        if category:
            pool = [q for q in pool if q.category == category]
        
        if len(pool) < n:
            n = len(pool)
        
        return random.sample(pool, n)
    
    def get_categories(self) -> List[str]:
        """ดู categories ทั้งหมด"""
        return sorted(set(q.category for q in self.questions))
    
    def get_stats(self) -> dict:
        """สถิติของ question bank"""
        return {
            "total": len(self.questions),
            "by_difficulty": {
                diff: len(self.get_by_difficulty(diff))
                for diff in ["easy", "medium", "hard"]
            },
            "categories": {
                cat: len(self.get_by_category(cat))
                for cat in self.get_categories()
            }
        }
```

---

### Score & Timer

```python
# score_timer.py - Score tracking และ Timer

import time
from dataclasses import dataclass, field
from typing import List, Optional


@dataclass
class Answer:
    """การตอบคำถามหนึ่งข้อ"""
    question_id: int
    question_text: str
    chosen_index: int
    chosen_text: str
    correct_index: int
    correct_text: str
    is_correct: bool
    time_taken: float
    points_earned: int


@dataclass
class QuizSession:
    """Session ของ quiz"""
    player_name: str
    quiz_name: str
    answers: List[Answer] = field(default_factory=list)
    start_time: float = field(default_factory=time.time)
    end_time: Optional[float] = None
    
    @property
    def total_time(self) -> float:
        if self.end_time:
            return self.end_time - self.start_time
        return time.time() - self.start_time
    
    @property
    def total_score(self) -> int:
        return sum(a.points_earned for a in self.answers)
    
    @property
    def correct_count(self) -> int:
        return sum(1 for a in self.answers if a.is_correct)
    
    @property
    def total_questions(self) -> int:
        return len(self.answers)
    
    @property
    def accuracy(self) -> float:
        if not self.answers:
            return 0.0
        return self.correct_count / self.total_questions * 100
    
    def finish(self):
        """สิ้นสุด session"""
        self.end_time = time.time()
    
    def add_answer(self, answer: Answer):
        self.answers.append(answer)
    
    def get_summary(self) -> dict:
        return {
            "player": self.player_name,
            "quiz": self.quiz_name,
            "total_score": self.total_score,
            "correct": self.correct_count,
            "total": self.total_questions,
            "accuracy": f"{self.accuracy:.1f}%",
            "time": f"{self.total_time:.1f}s",
        }


class QuizTimer:
    """Timer สำหรับ quiz"""
    
    def __init__(self, time_limit: int = 30):
        self.time_limit = time_limit
        self.start_time: Optional[float] = None
    
    def start(self):
        self.start_time = time.time()
    
    @property
    def elapsed(self) -> float:
        if self.start_time is None:
            return 0.0
        return time.time() - self.start_time
    
    @property
    def remaining(self) -> float:
        return max(0.0, self.time_limit - self.elapsed)
    
    @property
    def is_expired(self) -> bool:
        return self.elapsed >= self.time_limit
    
    @property
    def progress_bar(self) -> str:
        """แสดง timer เป็น progress bar"""
        remaining = self.remaining
        total = self.time_limit
        
        bar_length = 20
        filled = int((remaining / total) * bar_length)
        
        if remaining > total * 0.5:
            color = "🟢"
        elif remaining > total * 0.25:
            color = "🟡"
        else:
            color = "🔴"
        
        bar = "█" * filled + "░" * (bar_length - filled)
        return f"[{bar}] {remaining:.0f}s {color}"
```

---

### โค้ดสมบูรณ์ Quiz Game

```python
# quiz_game.py - โค้ดสมบูรณ์พร้อมรัน

import json
import os
import time
import random
import datetime
from dataclasses import dataclass, field, asdict
from typing import List, Optional, Dict
from pathlib import Path


# ===== Data Classes =====

@dataclass
class QuizQuestion:
    id: int
    text: str
    choices: List[str]
    correct_index: int
    category: str
    difficulty: str
    explanation: str = ""
    points: int = 10
    time_limit: int = 30


@dataclass
class PlayerAnswer:
    question_id: int
    chosen_index: int
    is_correct: bool
    time_taken: float
    points_earned: int


@dataclass
class GameResult:
    player_name: str
    score: int
    correct: int
    total: int
    accuracy: float
    time_taken: float
    date: str
    difficulty: str
    category: str


# ===== Question Bank =====

QUESTIONS = [
    QuizQuestion(1, "Python ถูกสร้างโดยใคร?",
        ["James Gosling", "Guido van Rossum", "Bjarne Stroustrup", "Dennis Ritchie"],
        1, "Python", "easy", "Guido van Rossum สร้าง Python ในปี 1991", 5, 30),
    
    QuizQuestion(2, "ผลลัพธ์ของ type([]) ใน Python คืออะไร?",
        ["<class 'tuple'>", "<class 'array'>", "<class 'list'>", "<class 'dict'>"],
        2, "Python", "easy", "[] คือ list literal", 5, 30),
    
    QuizQuestion(3, "keyword ใดสร้าง generator ใน Python?",
        ["return", "yield", "generate", "create"],
        1, "Python", "medium", "yield ใช้สร้าง generator", 10, 20),
    
    QuizQuestion(4, "Data structure ใดที่ LIFO?",
        ["Queue", "Stack", "Heap", "Tree"],
        1, "Data Structures", "easy", "Stack: Last In First Out", 5, 30),
    
    QuizQuestion(5, "Big O ของ Binary Search คือ?",
        ["O(1)", "O(n)", "O(log n)", "O(n²)"],
        2, "Algorithms", "medium", "Binary search แบ่งครึ่งทุกครั้ง", 10, 20),
    
    QuizQuestion(6, "HTTP status 404 หมายถึง?",
        ["Server Error", "Unauthorized", "Not Found", "Forbidden"],
        2, "Web", "easy", "404 Not Found", 5, 30),
    
    QuizQuestion(7, "SOLID ตัว S ย่อมาจาก?",
        ["Single Responsibility", "Simple Design", "Strong Typing", "Static Interface"],
        0, "OOP", "medium", "Single Responsibility Principle", 10, 20),
    
    QuizQuestion(8, "ฐาน 2 ใน Computer Science คือ?",
        ["Decimal", "Binary", "Hexadecimal", "Octal"],
        1, "CS", "easy", "Binary ใช้ 0 และ 1", 5, 30),
    
    QuizQuestion(9, "Decorator ใน Python ใช้ symbol ใด?",
        ["#", "@", "*", "&"],
        1, "Python", "easy", "@decorator_name", 5, 30),
    
    QuizQuestion(10, "GIL ย่อมาจาก?",
        ["Global Interface Lock", "General Input Lock", "Global Interpreter Lock", "General Interpreter Layer"],
        2, "Python", "hard", "Global Interpreter Lock", 15, 15),
    
    QuizQuestion(11, "Algorithm ใดที่ O(n log n) เฉลี่ย?",
        ["Bubble Sort", "Insertion Sort", "Quick Sort", "Selection Sort"],
        2, "Algorithms", "medium", "Quick Sort average O(n log n)", 10, 20),
    
    QuizQuestion(12, "Singleton เป็น pattern ประเภทใด?",
        ["Structural", "Behavioral", "Creational", "Functional"],
        2, "Design Patterns", "medium", "Creational Pattern", 10, 20),
    
    QuizQuestion(13, "URL ย่อมาจาก?",
        ["Universal Resource Locator", "Uniform Resource Locator", "Unique Record Locator", "Universal Record Linker"],
        1, "Web", "easy", "Uniform Resource Locator", 5, 30),
    
    QuizQuestion(14, "Python list comprehension รูปแบบคือ?",
        ["(x for x in list)", "[x for x in list]", "{x for x in list}", "list(x for x in list)"],
        1, "Python", "easy", "[expression for item in iterable]", 5, 30),
    
    QuizQuestion(15, "ฟังก์ชัน zip() ทำอะไร?",
        ["บีบอัดไฟล์", "รวม iterables เป็น tuples", "เรียงลำดับข้อมูล", "กรองข้อมูล"],
        1, "Python", "easy", "zip รวม iterables เป็น tuples", 5, 30),
]


# ===== Storage =====

class Storage:
    """จัดการ file storage"""
    
    def __init__(self, data_dir: str = "."):
        self.data_dir = Path(data_dir)
        self.leaderboard_file = self.data_dir / "leaderboard.json"
    
    def save_result(self, result: GameResult):
        """บันทึกผลลัพธ์"""
        results = self.load_results()
        results.append(asdict(result))
        
        self.leaderboard_file.parent.mkdir(parents=True, exist_ok=True)
        with open(self.leaderboard_file, 'w', encoding='utf-8') as f:
            json.dump(results, f, ensure_ascii=False, indent=2)
    
    def load_results(self) -> List[dict]:
        """โหลดผลลัพธ์"""
        if not self.leaderboard_file.exists():
            return []
        
        try:
            with open(self.leaderboard_file, 'r', encoding='utf-8') as f:
                return json.load(f)
        except (json.JSONDecodeError, IOError):
            return []
    
    def get_top_scores(self, n: int = 10) -> List[dict]:
        """ดู top scores"""
        results = self.load_results()
        return sorted(results, key=lambda r: r["score"], reverse=True)[:n]
    
    def get_player_history(self, player_name: str) -> List[dict]:
        """ดูประวัติของผู้เล่น"""
        results = self.load_results()
        return [r for r in results if r["player_name"] == player_name]


# ===== Quiz Engine =====

class QuizGame:
    """เกม Quiz"""
    
    def __init__(self, storage: Storage = None):
        self.storage = storage or Storage("/tmp")
        self.questions = QUESTIONS
        self.current_session: Optional[Dict] = None
    
    def get_questions(self, n: int = 5, difficulty: str = None, category: str = None) -> List[QuizQuestion]:
        """เลือกคำถาม"""
        pool = self.questions.copy()
        
        if difficulty and difficulty != "all":
            pool = [q for q in pool if q.difficulty == difficulty]
        
        if category and category != "all":
            pool = [q for q in pool if q.category.lower() == category.lower()]
        
        n = min(n, len(pool))
        return random.sample(pool, n)
    
    def start_session(self, player_name: str, difficulty: str = "all", category: str = "all", num_questions: int = 5):
        """เริ่ม quiz session"""
        questions = self.get_questions(num_questions, difficulty, category)
        
        self.current_session = {
            "player_name": player_name,
            "difficulty": difficulty,
            "category": category,
            "questions": questions,
            "answers": [],
            "start_time": time.time(),
            "current_q": 0,
        }
        
        return questions
    
    def answer_question(self, choice_index: int) -> Dict:
        """ตอบคำถาม"""
        if not self.current_session:
            raise RuntimeError("ไม่มี session ที่กำลังทำงาน")
        
        session = self.current_session
        q_index = session["current_q"]
        
        if q_index >= len(session["questions"]):
            raise RuntimeError("ตอบครบทุกข้อแล้ว")
        
        question = session["questions"][q_index]
        is_correct = choice_index == question.correct_index
        
        # คำนวณคะแนน (bonus สำหรับตอบเร็ว)
        points = question.points if is_correct else 0
        
        answer = PlayerAnswer(
            question_id=question.id,
            chosen_index=choice_index,
            is_correct=is_correct,
            time_taken=0,
            points_earned=points,
        )
        
        session["answers"].append(answer)
        session["current_q"] += 1
        
        return {
            "is_correct": is_correct,
            "correct_answer": question.choices[question.correct_index],
            "chosen_answer": question.choices[choice_index] if 0 <= choice_index < 4 else "???",
            "explanation": question.explanation,
            "points_earned": points,
        }
    
    def end_session(self) -> GameResult:
        """จบ session และบันทึกผล"""
        if not self.current_session:
            raise RuntimeError("ไม่มี session")
        
        session = self.current_session
        answers = session["answers"]
        total_time = time.time() - session["start_time"]
        
        total_score = sum(a.points_earned for a in answers)
        correct_count = sum(1 for a in answers if a.is_correct)
        total_questions = len(answers)
        accuracy = correct_count / total_questions * 100 if total_questions > 0 else 0
        
        result = GameResult(
            player_name=session["player_name"],
            score=total_score,
            correct=correct_count,
            total=total_questions,
            accuracy=accuracy,
            time_taken=total_time,
            date=datetime.datetime.now().isoformat(),
            difficulty=session["difficulty"],
            category=session["category"],
        )
        
        # บันทึกผล
        self.storage.save_result(result)
        
        self.current_session = None
        return result
    
    def show_leaderboard(self, n: int = 10):
        """แสดง leaderboard"""
        top_scores = self.storage.get_top_scores(n)
        
        if not top_scores:
            print("ยังไม่มีคะแนน")
            return
        
        print("\n" + "="*55)
        print(f"{'🏆 LEADERBOARD':^55}")
        print("="*55)
        print(f"{'#':>3}  {'Name':<15} {'Score':>6}  {'Acc':>6}  {'Time':>6}")
        print("-"*55)
        
        for i, entry in enumerate(top_scores, 1):
            medal = ["🥇", "🥈", "🥉"][i-1] if i <= 3 else f"{i:>2}."
            name = entry["player_name"][:14]
            score = entry["score"]
            accuracy = entry["accuracy"]
            time_taken = entry["time_taken"]
            
            print(f"{medal:>3}  {name:<15} {score:>6}  {accuracy:>5.1f}%  {time_taken:>5.1f}s")
        
        print("="*55)
    
    def show_player_stats(self, player_name: str):
        """แสดงสถิติของผู้เล่น"""
        history = self.storage.get_player_history(player_name)
        
        if not history:
            print(f"ไม่พบประวัติของ {player_name}")
            return
        
        scores = [h["score"] for h in history]
        accuracies = [h["accuracy"] for h in history]
        
        print(f"\n=== สถิติของ {player_name} ===")
        print(f"จำนวนครั้งที่เล่น: {len(history)}")
        print(f"คะแนนสูงสุด: {max(scores)}")
        print(f"คะแนนต่ำสุด: {min(scores)}")
        print(f"คะแนนเฉลี่ย: {sum(scores)/len(scores):.1f}")
        print(f"ความแม่นยำเฉลี่ย: {sum(accuracies)/len(accuracies):.1f}%")


# ===== Interactive Game =====

def run_quiz_game():
    """รัน quiz game แบบ interactive"""
    
    game = QuizGame()
    
    def clear_screen():
        os.system('cls' if os.name == 'nt' else 'clear')
    
    def print_header():
        print("╔═══════════════════════════════════╗")
        print("║         🎯 Python Quiz Game 🎯     ║")
        print("╚═══════════════════════════════════╝")
    
    def get_player_name():
        while True:
            name = input("ชื่อผู้เล่น: ").strip()
            if name:
                return name
            print("กรุณาใส่ชื่อ")
    
    def select_difficulty():
        print("\nระดับความยาก:")
        print("  1) Easy")
        print("  2) Medium")
        print("  3) Hard")
        print("  4) ทั้งหมด")
        
        choice = input("เลือก (1-4): ").strip()
        return {
            "1": "easy", "2": "medium",
            "3": "hard", "4": "all"
        }.get(choice, "all")
    
    def select_num_questions():
        while True:
            try:
                n = int(input("จำนวนคำถาม (1-15): ").strip())
                if 1 <= n <= 15:
                    return n
                print("กรุณาใส่ 1-15")
            except ValueError:
                print("กรุณาใส่ตัวเลข")
    
    def play_round(player_name, difficulty, num_questions):
        """เล่น 1 รอบ"""
        questions = game.start_session(player_name, difficulty, "all", num_questions)
        
        if not questions:
            print("ไม่พบคำถาม")
            return None
        
        print(f"\n🎮 เริ่มเกม! {len(questions)} คำถาม")
        print("กด Enter เพื่อเริ่ม...")
        input()
        
        for q_num, question in enumerate(questions, 1):
            print(f"\n{'─'*45}")
            print(f"ข้อ {q_num}/{len(questions)} | "
                  f"[{question.difficulty.upper()}] [{question.category}] | "
                  f"{question.points} คะแนน")
            print(f"\n{question.text}\n")
            
            for i, choice in enumerate(question.choices):
                print(f"  {'ABCD'[i]}) {choice}")
            
            # รับ input
            while True:
                answer_input = input("\nคำตอบ (A/B/C/D): ").strip().upper()
                if answer_input in "ABCD" and len(answer_input) == 1:
                    choice_index = "ABCD".index(answer_input)
                    break
                print("กรุณาพิมพ์ A, B, C หรือ D")
            
            # ตรวจสอบคำตอบ
            result = game.answer_question(choice_index)
            
            if result["is_correct"]:
                print(f"\n  ✅ ถูกต้อง! +{result['points_earned']} คะแนน")
            else:
                print(f"\n  ❌ ผิด!")
                print(f"  คำตอบที่ถูกต้อง: {result['correct_answer']}")
            
            if result["explanation"]:
                print(f"  💡 {result['explanation']}")
            
            time.sleep(1)
        
        # จบ session
        final_result = game.end_session()
        return final_result
    
    def show_results(result: GameResult):
        """แสดงผลลัพธ์"""
        print("\n" + "="*45)
        print(f"{'🏁 ผลลัพธ์':^45}")
        print("="*45)
        print(f"ผู้เล่น: {result.player_name}")
        print(f"คะแนน: {result.score} คะแนน")
        print(f"ถูก: {result.correct}/{result.total} ข้อ")
        print(f"ความแม่นยำ: {result.accuracy:.1f}%")
        print(f"เวลา: {result.time_taken:.1f} วินาที")
        
        if result.accuracy >= 90:
            print("\n🏆 ยอดเยี่ยมมาก!")
        elif result.accuracy >= 70:
            print("\n⭐ ดีมาก!")
        elif result.accuracy >= 50:
            print("\n👍 ผ่านแล้ว!")
        else:
            print("\n📚 ลองใหม่อีกครั้งนะ!")
        
        print("="*45)
    
    # Main loop
    print_header()
    
    while True:
        print("\n📋 เมนูหลัก")
        print("  1) เล่นเกม")
        print("  2) Leaderboard")
        print("  3) สถิติของฉัน")
        print("  4) ออก")
        
        choice = input("\nเลือก: ").strip()
        
        if choice == "1":
            print_header()
            player_name = get_player_name()
            difficulty = select_difficulty()
            num_questions = select_num_questions()
            
            result = play_round(player_name, difficulty, num_questions)
            
            if result:
                show_results(result)
            
            input("\nกด Enter เพื่อกลับสู่เมนู...")
        
        elif choice == "2":
            game.show_leaderboard()
            input("\nกด Enter เพื่อกลับ...")
        
        elif choice == "3":
            name = input("ชื่อผู้เล่น: ").strip()
            game.show_player_stats(name)
            input("\nกด Enter เพื่อกลับ...")
        
        elif choice == "4":
            print("\nขอบคุณที่เล่น! ลาก่อน! 👋")
            break
        
        else:
            print("กรุณาเลือก 1-4")


# ===== Demo mode (ไม่ interactive) =====

def demo_quiz():
    """Demo quiz game"""
    print("\n" + "="*50)
    print("Quiz Game Demo")
    print("="*50)
    
    game = QuizGame()
    
    # แสดงสถิติ question bank
    print(f"\nคำถามทั้งหมด: {len(game.questions)} ข้อ")
    
    categories = set(q.category for q in game.questions)
    print(f"Categories: {', '.join(sorted(categories))}")
    
    difficulties = set(q.difficulty for q in game.questions)
    for diff in ["easy", "medium", "hard"]:
        count = sum(1 for q in game.questions if q.difficulty == diff)
        print(f"  {diff}: {count} ข้อ")
    
    # จำลองการเล่น (auto)
    print("\n--- จำลองการเล่น ---")
    player_name = "Demo Player"
    questions = game.start_session(player_name, "easy", "all", 5)
    
    print(f"ผู้เล่น: {player_name}")
    print(f"คำถาม: {len(questions)} ข้อ\n")
    
    for i, q in enumerate(questions, 1):
        print(f"ข้อ {i}: {q.text}")
        
        # สุ่มตอบ (70% ถูก)
        if random.random() < 0.7:
            chosen = q.correct_index  # ตอบถูก
        else:
            # สุ่มตอบผิด
            wrong = [j for j in range(4) if j != q.correct_index]
            chosen = random.choice(wrong)
        
        result = game.answer_question(chosen)
        status = "✅ ถูก" if result["is_correct"] else "❌ ผิด"
        print(f"  {status} - ตอบ: {q.choices[chosen]}")
        if not result["is_correct"]:
            print(f"  คำตอบถูก: {result['correct_answer']}")
    
    # จบและแสดงผล
    final_result = game.end_session()
    
    print(f"\n=== ผลลัพธ์ ===")
    print(f"คะแนน: {final_result.score}")
    print(f"ถูก: {final_result.correct}/{final_result.total}")
    print(f"ความแม่นยำ: {final_result.accuracy:.1f}%")
    print(f"เวลา: {final_result.time_taken:.1f}s")
    
    # แสดง leaderboard
    game.show_leaderboard()
    
    # สถิติผู้เล่น
    game.show_player_stats(player_name)


# รัน demo
demo_quiz()

# รัน interactive (uncomment)
# run_quiz_game()
```

---

### วิธีรัน Quiz Game

```bash
# รัน demo
python quiz_game.py

# รัน interactive mode
# แก้ไขบรรทัดสุดท้าย:
# demo_quiz()    -> comment out
# run_quiz_game() -> uncomment

# ตัวอย่าง interactive:
# 📋 เมนูหลัก
#   1) เล่นเกม
#   2) Leaderboard
#   3) สถิติของฉัน
#   4) ออก
#
# เลือก: 1
# ชื่อผู้เล่น: Alice
# ระดับความยาก: 1 (Easy)
# จำนวนคำถาม: 5
```

---

## Ideas สำหรับการต่อยอด

### Scientific Calculator

```python
# 1. เพิ่ม GUI ด้วย tkinter
"""
import tkinter as tk
from tkinter import ttk

class CalculatorGUI:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("Scientific Calculator")
        self.calc = Calculator()
        self._build_ui()
    
    def _build_ui(self):
        # Display
        self.display = tk.Entry(self.root, width=30, font=('Arial', 16))
        self.display.grid(row=0, columnspan=5, padx=5, pady=5)
        
        # Buttons
        buttons = [
            ('7', 1, 0), ('8', 1, 1), ('9', 1, 2), ('/', 1, 3), ('C', 1, 4),
            ('4', 2, 0), ('5', 2, 1), ('6', 2, 2), ('*', 2, 3), ('(', 2, 4),
            ('1', 3, 0), ('2', 3, 1), ('3', 3, 2), ('-', 3, 3), (')', 3, 4),
            ('0', 4, 0), ('.', 4, 1), ('=', 4, 2), ('+', 4, 3), ('^', 4, 4),
        ]
        
        for (text, row, col) in buttons:
            btn = tk.Button(self.root, text=text, width=5,
                          command=lambda t=text: self._button_click(t))
            btn.grid(row=row, column=col, padx=2, pady=2)
    
    def run(self):
        self.root.mainloop()
"""

# 2. เพิ่ม Graphing ด้วย matplotlib
"""
import matplotlib.pyplot as plt
import numpy as np

def plot_function(expression, x_range=(-10, 10)):
    x = np.linspace(x_range[0], x_range[1], 1000)
    y = [eval(expression.replace('x', str(xi))) for xi in x]
    
    plt.plot(x, y)
    plt.grid(True)
    plt.axhline(y=0, color='k')
    plt.axvline(x=0, color='k')
    plt.title(f"f(x) = {expression}")
    plt.show()
"""

# 3. เพิ่ม Complex Number support
"""
class ComplexCalculator(Calculator):
    def add_complex(self, a: complex, b: complex) -> complex:
        return a + b
    
    def polar_to_rect(self, r: float, theta_deg: float) -> complex:
        import cmath
        theta_rad = math.radians(theta_deg)
        return cmath.rect(r, theta_rad)
"""

# 4. เพิ่ม History Export
"""
import csv

def export_history_csv(calc: Calculator, filename: str):
    with open(filename, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=['id', 'expression', 'result', 'timestamp'])
        writer.writeheader()
        writer.writerows(calc.get_history())
    print(f"Export ไปที่ {filename}")
"""

print("=== Calculator Extension Ideas ===")
print("1. GUI ด้วย tkinter")
print("2. Graphing ด้วย matplotlib")
print("3. Complex numbers")
print("4. History export (CSV/JSON)")
print("5. Custom precision settings")
print("6. Equation solver")
print("7. Matrix operations")
print("8. Statistics charts")
```

### Quiz Game Extensions

```python
# 1. เพิ่ม Timer จริง
"""
import threading

class TimedQuestion:
    def __init__(self, time_limit=30):
        self.time_limit = time_limit
        self.answer = None
        self._timer = None
    
    def ask(self, question: QuizQuestion) -> Optional[int]:
        question.display()
        
        self._timer = threading.Timer(self.time_limit, self._on_timeout)
        self._timer.start()
        
        answer_input = input(f"คำตอบ (เวลา {self.time_limit}s): ").strip().upper()
        
        if self._timer:
            self._timer.cancel()
        
        if answer_input in "ABCD":
            return "ABCD".index(answer_input)
        return None
    
    def _on_timeout(self):
        print("\n⏰ หมดเวลา!")
"""

# 2. เพิ่ม Multiplayer
"""
class MultiplayerQuiz:
    def __init__(self, player_names: List[str]):
        self.players = {name: 0 for name in player_names}
        self.game = QuizGame()
    
    def play_round(self, question: QuizQuestion):
        print(f"\n{'='*40}")
        question.display()
        
        for player_name in self.players:
            answer_input = input(f"{player_name} ตอบ (A/B/C/D): ").strip().upper()
            if answer_input in "ABCD":
                choice = "ABCD".index(answer_input)
                if question.is_correct(choice):
                    self.players[player_name] += question.points
                    print(f"  ✅ {player_name} ถูก!")
            
        winner = max(self.players, key=self.players.get)
        print(f"\n🏆 ผู้นำ: {winner} ({self.players[winner]} คะแนน)")
"""

# 3. เพิ่มประเภทคำถามใหม่
"""
@dataclass
class TrueFalseQuestion(QuizQuestion):
    def __post_init__(self):
        self.choices = ["True", "False"]
        self.correct_index = 0 if self.answer else 1

@dataclass  
class FillBlankQuestion:
    text: str  # "Python is ___"
    answer: str
    category: str
    
    def is_correct(self, user_answer: str) -> bool:
        return user_answer.strip().lower() == self.answer.lower()
"""

# 4. เพิ่ม Category System
"""
class CategoryManager:
    def __init__(self):
        self.categories = {}
    
    def add_category(self, name: str, description: str, icon: str = "📚"):
        self.categories[name] = {
            "description": description,
            "icon": icon,
            "questions": []
        }
    
    def add_question_to_category(self, category: str, question: QuizQuestion):
        if category not in self.categories:
            self.categories[category] = {"questions": []}
        self.categories[category]["questions"].append(question)
"""

# 5. Web Interface ด้วย Flask
"""
from flask import Flask, jsonify, request

app = Flask(__name__)
game = QuizGame()

@app.route("/api/quiz/start", methods=["POST"])
def start_quiz():
    data = request.json
    player_name = data.get("player_name")
    difficulty = data.get("difficulty", "all")
    n = data.get("num_questions", 5)
    
    questions = game.start_session(player_name, difficulty, "all", n)
    return jsonify([{
        "id": q.id,
        "text": q.text,
        "choices": q.choices,
        "category": q.category,
        "difficulty": q.difficulty,
        "points": q.points
    } for q in questions])

@app.route("/api/quiz/answer", methods=["POST"])
def answer():
    data = request.json
    choice_index = data.get("choice_index")
    result = game.answer_question(choice_index)
    return jsonify(result)
"""

print("\n=== Quiz Game Extension Ideas ===")
print("1. Timer จริงสำหรับแต่ละคำถาม")
print("2. Multiplayer (real-time)")
print("3. ประเภทคำถามใหม่ (True/False, Fill-blank)")
print("4. Category ที่ user สร้างเองได้")
print("5. Web interface ด้วย Flask")
print("6. Mobile app ด้วย Kivy")
print("7. ระบบ hints")
print("8. Achievement badges")
print("9. Daily challenges")
print("10. Import คำถามจาก CSV/Excel")
```

---

## สรุป

ในบทนี้เราได้สร้าง 2 โปรเจกต์จริง:

### Scientific Calculator
- Calculator class พร้อม operations ครบ (basic, trig, log, statistics)
- Memory operations (MS, MR, MC, M+)
- History tracking ทุกการคำนวณ
- Unit converter (length, weight, temperature, speed, digital)
- Error handling ที่ครบถ้วน
- Interactive CLI interface

### Quiz Game  
- Question bank พร้อม categories และ difficulty levels
- Score tracking และ accuracy calculation
- Leaderboard system
- File-based storage (JSON)
- Player statistics
- Randomized questions

### สิ่งที่เรียนรู้จากโปรเจกต์นี้

| ทักษะ | นำมาใช้ใน |
|------|----------|
| Classes & OOP | Calculator, QuizGame, Storage |
| Error Handling | CalculatorError hierarchy |
| File I/O | JSON storage |
| dataclasses | QuizQuestion, GameResult |
| List comprehension | question filtering |
| Type hints | ทุกฟังก์ชัน |
| Functional concepts | pipeline, filter, map |
| Module organization | แยก classes ตาม responsibility |

### ขั้นต่อไป

หลังจาก Part 20 นี้ คุณพร้อมสำหรับ:
- **Part 21**: Object-Oriented Programming ขั้นสูง
- **Part 22**: File I/O & OS module
- **Part 23**: Regular Expressions (Regex)
- **Part 24**: Database (SQLite)
- **Part 25**: Web Scraping
- **Part 26**: API Development with Flask/FastAPI
