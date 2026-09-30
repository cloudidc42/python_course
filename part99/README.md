# Part 99: Code Quality, Standards & Documentation

## สารบัญ

1. [PEP 8 Style Guide](#pep8)
2. [Code Formatting Tools](#formatting)
3. [Linting](#linting)
4. [Type Checking](#type-checking)
5. [Documentation with Sphinx](#sphinx)
6. [Docstring Standards](#docstrings)
7. [API Documentation (OpenAPI)](#openapi)
8. [MkDocs](#mkdocs)
9. [Pre-commit Hooks](#pre-commit)
10. [Code Review Best Practices](#code-review)
11. [Technical Debt Management](#tech-debt)
12. [Refactoring Safely](#refactoring)
13. [Git Best Practices](#git)
14. [Semantic Versioning](#semver)
15. [Changelog Management](#changelog)
16. [แบบฝึกหัด](#exercises)

---

## 1. PEP 8 Style Guide <a name="pep8"></a>

PEP 8 คือ Python Style Guide ที่เป็นมาตรฐาน ทุกคนในทีมควรรู้และปฏิบัติตาม

### Naming Conventions

```python
# ตัวอย่าง 1: Naming Conventions

# ✓ ถูก: Variables และ functions - snake_case
user_name = "Alice"
total_price = 99.99
max_retry_count = 3

def calculate_discount(price, percentage):
    return price * percentage / 100

def get_user_by_id(user_id):
    pass

# ✓ ถูก: Constants - UPPER_SNAKE_CASE
MAX_CONNECTIONS = 100
DATABASE_URL = "postgresql://localhost/mydb"
DEFAULT_TIMEOUT_SECONDS = 30

# ✓ ถูก: Classes - PascalCase (CapWords)
class UserAccount:
    pass

class PaymentProcessor:
    pass

class HTTPClient:  # Acronyms ทั้งหมดตัวใหญ่
    pass

# ✓ ถูก: Modules - lowercase_with_underscores
# user_service.py
# payment_processor.py
# database_connection.py

# ✓ ถูก: Packages - lowercase
# mypackage/
# user_management/

# ✓ ถูก: Private/Protected - leading underscore
class BankAccount:
    def __init__(self):
        self._balance = 0.0      # protected
        self.__account_key = ""  # private (name mangling)
    
    def _internal_helper(self):  # protected method
        pass
    
    def __validate_amount(self, amount):  # private method
        return amount > 0

# ✗ ผิด: ตัวอย่าง bad naming
userAccount = "Alice"      # camelCase for variable
User_Name = "Bob"          # inconsistent
l = 1                      # ambiguous (l vs 1)
O = 0                      # ambiguous (O vs 0)
x = 123                    # non-descriptive

print("Naming conventions: PEP 8 compliant")
```

### Indentation และ Whitespace

```python
# ตัวอย่าง 2: Indentation และ Line Length

# ✓ 4 spaces (ไม่ใช้ tab)
def calculate_total(items):
    total = 0
    for item in items:
        if item['active']:
            total += item['price'] * item['quantity']
    return total

# ✓ Line length สูงสุด 79 characters (หรือ 88 สำหรับ Black)
# ถ้ายาวเกินให้ใช้ continuation

# Method 1: Implicit line continuation inside brackets
result = (
    some_very_long_variable_name +
    another_long_variable_name +
    yet_another_variable
)

# Method 2: Function call continuation
def long_function_name(
    argument_one,
    argument_two,
    argument_three,
    keyword_argument=None,
):
    pass

# Method 3: Dict/list continuation
config = {
    'host': 'localhost',
    'port': 5432,
    'database': 'mydb',
    'user': 'postgres',
    'password': 'secret',
}

# ✓ Blank lines
class MyClass:
    """Class with proper blank lines."""
    
    CLASS_CONSTANT = 42  # class level
    
    def __init__(self):  # blank line before method
        self.value = 0
    
    def method_one(self):  # blank line between methods
        pass
    
    def method_two(self):
        pass


def function_one():  # two blank lines between top-level defs
    pass


def function_two():
    pass
```

### Imports

```python
# ตัวอย่าง 3: Import conventions

# ✓ ถูก: 1 import per line
import os
import sys
import json

# ✓ ถูก: grouping (stdlib, third-party, local)
# Standard library
import os
import sys
from pathlib import Path
from typing import Optional, List, Dict

# Third-party (blank line separating groups)
import requests
import sqlalchemy
from fastapi import FastAPI

# Local (blank line separating)
from mypackage.models import User
from mypackage.services import UserService

# ✗ ผิด: wildcard imports
# from os import *        # นำ names ทั้งหมดเข้ามา - confusing

# ✗ ผิด: import multiple on one line  
# import os, sys

# ✓ ถูก: from import is fine for specific items
from collections import defaultdict, OrderedDict
from typing import Tuple, Union, Any

# ✓ ถูก: aliases เมื่อจำเป็น
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

### Whitespace in Expressions

```python
# ตัวอย่าง 4: Whitespace rules

# ✓ ถูก: spaces around operators
x = 1 + 2
y = x * 3 - 1
result = (x + y) / 2

# ✗ ผิด: inconsistent or no spaces
# x=1+2
# x =1+2

# ✓ ถูก: no space around = in keyword arguments
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice", greeting="Hi")

# ✗ ผิด
# def greet(name, greeting = "Hello"):
# greet("Alice", greeting = "Hi")

# ✓ ถูก: no spaces before colon/comma
data = [1, 2, 3]
my_dict = {'key': 'value'}
coordinates = (1, 2)

# ✗ ผิด
# data = [1 , 2 , 3]
# my_dict = { 'key' : 'value' }

# ✓ ถูก: spaces after commas
def func(a, b, c):
    return a, b, c

# ✓ ถูก: comparison operators
if x == 0:
    pass
if x is None:
    pass
if x is not None:
    pass
if x in [1, 2, 3]:
    pass

# ✗ ผิด: comparing with None using ==
# if x == None:  # should use 'is'
#     pass
```

---

## 2. Code Formatting Tools <a name="formatting"></a>

### Black - The Uncompromising Formatter

```python
# ตัวอย่าง 5: Black formatter
# pip install black

# ก่อน Black format (ugly)
UGLY_CODE = '''
x=1+2
result=x*3-1
def my_function(x,y,z):
    if x>0:
        return x+y+z
    else:
        return 0
my_dict={'key1':'value1','key2':'value2','key3':'value3','key4':'value4'}
my_list=[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
'''

# หลัง Black format (clean)
CLEAN_CODE = '''
x = 1 + 2
result = x * 3 - 1


def my_function(x, y, z):
    if x > 0:
        return x + y + z
    else:
        return 0


my_dict = {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3",
    "key4": "value4",
}
my_list = [
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
    11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
]
'''

print("Black Formatter:")
print("Before:")
print(UGLY_CODE)
print("After:")
print(CLEAN_CODE)

# pyproject.toml configuration สำหรับ Black
PYPROJECT_TOML = """
[tool.black]
line-length = 88
target-version = ['py39', 'py310', 'py311']
include = '\\.pyi?$'
extend-exclude = '''
/(
    migrations
    | build
    | dist
)/
'''
"""

print("pyproject.toml configuration:")
print(PYPROJECT_TOML)
```

### isort - Import Sorting

```python
# ตัวอย่าง 6: isort - organize imports
# pip install isort

# ก่อน isort
UNSORTED_IMPORTS = """
from mypackage.models import User
import json
from typing import Optional
import requests
import os
from collections import defaultdict
import sys
"""

# หลัง isort
SORTED_IMPORTS = """
import json
import os
import sys
from collections import defaultdict
from typing import Optional

import requests

from mypackage.models import User
"""

print("isort - Import Sorting:")
print("Before isort:")
print(UNSORTED_IMPORTS)
print("After isort:")
print(SORTED_IMPORTS)

# .isort.cfg หรือ pyproject.toml
ISORT_CONFIG = """
[tool.isort]
profile = "black"
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
line_length = 88
known_first_party = ["mypackage"]
"""

print("isort configuration:")
print(ISORT_CONFIG)
```

### autopep8 - PEP 8 Compliance

```python
# ตัวอย่าง 7: autopep8
# pip install autopep8

BEFORE_AUTOPEP8 = """
x=1+2
if x >0:
    print('positive')
def foo( x, y ):
    return x+y
my_list=[1,2,3]
"""

print("autopep8 Demo:")
print("Before:")
print(BEFORE_AUTOPEP8)

# To format:
# autopep8 --in-place --aggressive --aggressive your_file.py

# Or check what it would change:
# autopep8 --diff your_file.py

print("After autopep8 (would fix spacing issues)")
```

---

## 3. Linting <a name="linting"></a>

```python
# ตัวอย่าง 8: flake8 - PEP 8 checker
# pip install flake8

FLAKE8_VIOLATIONS = """
# Example violations flake8 would catch:

import os,sys                  # E401: multiple imports on one line
import json                    # F401: unused import

x=1                           # E225: missing whitespace around operator
y = 1+2                       # E225: missing whitespace around operator
z = [ 1,2,3 ]                 # E201, E202: whitespace in brackets

def func(x,y,z):              # E231: missing whitespace after ','
    pass

class myClass:                 # N801: class should use CapWords
    def method( self ):        # E211, E201: whitespace before '('
        result = 1+1           # E225: missing whitespace
        if result==2:          # E225
            return True
        else: return False     # E701: multiple statements on one line
"""

print("flake8 violations example:")
print(FLAKE8_VIOLATIONS)

# .flake8 configuration
FLAKE8_CONFIG = """
[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude =
    .git,
    __pycache__,
    build,
    dist,
    migrations
per-file-ignores =
    tests/*: S101
max-complexity = 10
"""

print("flake8 configuration:")
print(FLAKE8_CONFIG)
```

```python
# ตัวอย่าง 9: pylint - comprehensive linter
# pip install pylint

PYLINT_EXAMPLE = """
# pylint catches more issues than flake8

# C0116: Missing function or method docstring
def add(a, b):
    return a + b

# W0611: Unused import
import json

# R0914: Too many local variables (> 15)
def complex_function():
    a = b = c = d = e = f = g = h = i = j = k = l = m = n = o = p = 1

# C0103: Variable name doesn't conform to snake_case
X = 1  # should be x for variable, X for constant

# W0622: Redefining built-in
list = [1, 2, 3]  # shadows built-in 'list'

# R0201: Method could be a function (doesn't use self)
class MyClass:
    def method(self, x, y):
        return x + y  # no self usage

# E1101: Module has no member
# import nonexistent
# nonexistent.function()
"""

print("pylint issues:")
print(PYLINT_EXAMPLE)

# .pylintrc
PYLINTRC = """
[MESSAGES CONTROL]
disable = 
    C0114,  # Missing module docstring
    C0115,  # Missing class docstring
    R0903,  # Too few public methods

[FORMAT]
max-line-length = 88

[DESIGN]
max-args = 8
max-attributes = 12
max-bool-expr = 5
max-branches = 12
max-locals = 20
max-returns = 6
max-statements = 50
max-parents = 7
"""

print("\n.pylintrc configuration:")
print(PYLINTRC)
```

```python
# ตัวอย่าง 10: ruff - fast linter (Rust-based)
# pip install ruff

RUFF_CONFIG = """
# pyproject.toml
[tool.ruff]
line-length = 88
target-version = "py311"

select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
    "N",   # pep8-naming
    "S",   # flake8-bandit (security)
]

ignore = [
    "E501",  # line too long (handled by formatter)
    "B008",  # do not perform function calls in default arguments
]

[tool.ruff.per-file-ignores]
"tests/**/*.py" = ["S101"]  # allow assert in tests
"migrations/*.py" = ["N999"]  # allow non-standard names

[tool.ruff.isort]
known-first-party = ["mypackage"]
"""

print("Ruff configuration:")
print(RUFF_CONFIG)

print("""
Ruff advantages:
- 10-100x faster than flake8 + isort + pylint combined
- Single tool replaces multiple linters
- Compatible with most flake8 plugins
- Excellent for CI/CD pipelines
""")
```

---

## 4. Type Checking <a name="type-checking"></a>

```python
# ตัวอย่าง 11: mypy - static type checker
# pip install mypy

from typing import Optional, List, Dict, Tuple, Union, Any, Callable
from typing import TypeVar, Generic, Protocol
from typing import overload
from dataclasses import dataclass
from enum import Enum

# Basic type annotations
def greet(name: str, times: int = 1) -> str:
    return f"Hello, {name}! " * times

# Optional
def find_user(user_id: int) -> Optional[dict]:
    # returns dict or None
    if user_id == 1:
        return {"id": 1, "name": "Alice"}
    return None

# Union (or use X | Y in Python 3.10+)
def process(data: Union[str, int, list]) -> str:
    if isinstance(data, str):
        return data.upper()
    elif isinstance(data, int):
        return str(data)
    else:
        return str(len(data))

# Generic types
def first(items: List[int]) -> Optional[int]:
    return items[0] if items else None

def get_value(data: Dict[str, int], key: str) -> Optional[int]:
    return data.get(key)

# TypeVar for generic functions
T = TypeVar('T')

def identity(x: T) -> T:
    return x

def last(items: List[T]) -> Optional[T]:
    return items[-1] if items else None

# Callable types
def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)

# Protocols (structural subtyping)
class Comparable(Protocol):
    def __lt__(self, other: Any) -> bool: ...
    def __gt__(self, other: Any) -> bool: ...

def find_max(items: List[Comparable]) -> Optional[Comparable]:
    if not items:
        return None
    return max(items)

# Dataclasses with types
@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0
    
    def distance_to(self, other: 'Point') -> float:
        return ((self.x - other.x)**2 + 
                (self.y - other.y)**2 + 
                (self.z - other.z)**2) ** 0.5

# Enums
class Color(Enum):
    RED = "red"
    GREEN = "green"
    BLUE = "blue"

def get_hex_color(color: Color) -> str:
    mapping: Dict[Color, str] = {
        Color.RED: "#FF0000",
        Color.GREEN: "#00FF00",
        Color.BLUE: "#0000FF",
    }
    return mapping[color]

# Test
p1 = Point(0.0, 0.0)
p2 = Point(3.0, 4.0)
print(f"\nType Annotations Demo:")
print(f"  greet('Alice', 2): {greet('Alice', 2)}")
print(f"  find_user(1): {find_user(1)}")
print(f"  distance: {p1.distance_to(p2)}")
print(f"  hex color: {get_hex_color(Color.RED)}")
```

```python
# ตัวอย่าง 12: Advanced type annotations
from typing import TypedDict, Final, Literal, ClassVar
from typing import NamedTuple, get_type_hints
from functools import cached_property

# TypedDict - dict with specific key types
class UserDict(TypedDict):
    id: int
    name: str
    email: str
    age: int

class PartialUserDict(TypedDict, total=False):
    name: str
    email: str
    age: int

def process_user(user: UserDict) -> str:
    return f"{user['name']} <{user['email']}>"

# Final - constants
MAX_RETRIES: Final[int] = 3
API_VERSION: Final = "v2"

# Literal - specific values only
def set_mode(mode: Literal["read", "write", "append"]) -> None:
    print(f"Setting mode to: {mode}")

def get_status() -> Literal["active", "inactive", "pending"]:
    return "active"

# NamedTuple
class Coordinate(NamedTuple):
    latitude: float
    longitude: float
    altitude: float = 0.0

coord = Coordinate(13.7563, 100.5018, 5.0)
print(f"  Coordinate: {coord}")

# ClassVar
class Config:
    _instances: ClassVar[Dict[str, 'Config']] = {}
    
    def __init__(self, name: str):
        self.name = name

# Overload for multiple signatures
@overload
def parse_value(value: str) -> str: ...

@overload
def parse_value(value: int) -> int: ...

@overload
def parse_value(value: float) -> float: ...

def parse_value(value):
    return value

# mypy configuration
MYPY_CONFIG = """
# mypy.ini or [tool.mypy] in pyproject.toml
[mypy]
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
disallow_incomplete_defs = True
check_untyped_defs = True
disallow_untyped_decorators = True
warn_redundant_casts = True
warn_unused_ignores = True
strict = True

[mypy-requests.*]
ignore_missing_imports = True

[mypy-tests.*]
disallow_untyped_defs = False
"""

print(f"\nmypy configuration:")
print(MYPY_CONFIG)

# pyright configuration  
PYRIGHTCONFIG = """
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__", "build"],
  "strict": ["src/core"],
  "reportMissingImports": true,
  "reportMissingTypeStubs": false,
  "pythonVersion": "3.11",
  "pythonPlatform": "Linux",
  "venvPath": ".",
  "venv": ".venv"
}
"""

print("pyrightconfig.json:")
print(PYRIGHTCONFIG)
```

---

## 5. Documentation with Sphinx <a name="sphinx"></a>

```python
# ตัวอย่าง 13: Sphinx documentation setup

SPHINX_SETUP = """
# Installation
pip install sphinx sphinx-rtd-theme sphinx-autodoc-typehints

# Initialize
sphinx-quickstart docs/

# docs/conf.py configuration
import os
import sys
sys.path.insert(0, os.path.abspath('../src'))

project = 'My Python Project'
copyright = '2024, Author Name'
author = 'Author Name'
release = '2.1.0'

extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.napoleon',      # Google/NumPy docstring support
    'sphinx.ext.viewcode',      # Add source code links
    'sphinx.ext.todo',          # Todo notes
    'sphinx.ext.coverage',      # Coverage report
    'sphinx_autodoc_typehints', # Type hints in docs
    'myst_parser',              # Markdown support
]

templates_path = ['_templates']
exclude_patterns = ['_build', 'Thumbs.db', '.DS_Store']
html_theme = 'sphinx_rtd_theme'

# Napoleon settings
napoleon_google_docstring = True
napoleon_numpy_docstring = True
napoleon_include_init_with_doc = False
napoleon_include_private_with_doc = False

# docs/index.rst
Contents:
.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
   installation
   quickstart
   api/modules
   contributing
   changelog

# Build documentation
make html           # Unix
./make.bat html     # Windows
"""

print("Sphinx Documentation Setup:")
print(SPHINX_SETUP)
```

---

## 6. Docstring Standards <a name="docstrings"></a>

```python
# ตัวอย่าง 14: Google style docstrings

def calculate_compound_interest(
    principal: float,
    rate: float,
    time: float,
    n: int = 1,
) -> float:
    """Calculate compound interest.
    
    Uses the formula: A = P(1 + r/n)^(nt)
    where A is the final amount, P is the principal,
    r is the annual interest rate, n is the number of
    times interest is compounded per year, and t is time.
    
    Args:
        principal: Initial investment amount in dollars.
            Must be positive.
        rate: Annual interest rate as a decimal (e.g., 0.05 for 5%).
            Must be between 0 and 1.
        time: Time period in years. Must be positive.
        n: Number of compounding periods per year.
            Defaults to 1 (annual compounding).
    
    Returns:
        The final amount after compound interest is applied.
        Returns the principal if rate is 0.
    
    Raises:
        ValueError: If principal is negative or zero.
        ValueError: If rate is not between 0 and 1.
        ValueError: If time is negative or zero.
        ValueError: If n is less than 1.
    
    Example:
        >>> calculate_compound_interest(1000, 0.05, 10)
        1628.8946267774418
        
        >>> calculate_compound_interest(1000, 0.05, 10, n=12)
        1647.0094598785172
        
        >>> calculate_compound_interest(5000, 0.08, 5, n=4)
        7429.737579828499
    
    Note:
        For continuous compounding, use n = float('inf').
        This function does not account for inflation or taxes.
    
    See Also:
        :func:`calculate_simple_interest`: For simple interest calculation.
        :class:`InvestmentCalculator`: For more complex calculations.
    """
    if principal <= 0:
        raise ValueError(f"Principal must be positive, got {principal}")
    if not 0 <= rate <= 1:
        raise ValueError(f"Rate must be between 0 and 1, got {rate}")
    if time <= 0:
        raise ValueError(f"Time must be positive, got {time}")
    if n < 1:
        raise ValueError(f"n must be at least 1, got {n}")
    
    return principal * (1 + rate / n) ** (n * time)


class BankAccount:
    """Represents a bank account with basic operations.
    
    This class provides a simple bank account implementation
    with deposit, withdrawal, and balance tracking features.
    
    Attributes:
        account_number: Unique identifier for the account.
        owner_name: Full name of the account owner.
        balance: Current account balance in dollars.
        transaction_history: List of all transactions.
    
    Example:
        >>> account = BankAccount("ACC001", "Alice Smith", initial_balance=1000.0)
        >>> account.deposit(500.0)
        >>> account.withdraw(200.0)
        >>> print(account.balance)
        1300.0
    """
    
    def __init__(
        self,
        account_number: str,
        owner_name: str,
        initial_balance: float = 0.0,
    ) -> None:
        """Initialize a new bank account.
        
        Args:
            account_number: Unique account identifier.
                Must be alphanumeric.
            owner_name: Full name of the account owner.
                Cannot be empty.
            initial_balance: Starting balance. Defaults to 0.0.
                Must be non-negative.
        
        Raises:
            ValueError: If account_number is empty or contains spaces.
            ValueError: If owner_name is empty.
            ValueError: If initial_balance is negative.
        """
        if not account_number or ' ' in account_number:
            raise ValueError("Account number must be non-empty without spaces")
        if not owner_name:
            raise ValueError("Owner name cannot be empty")
        if initial_balance < 0:
            raise ValueError(f"Initial balance cannot be negative: {initial_balance}")
        
        self.account_number = account_number
        self.owner_name = owner_name
        self.balance = initial_balance
        self.transaction_history: List[dict] = []
    
    def deposit(self, amount: float) -> float:
        """Add funds to the account.
        
        Args:
            amount: Amount to deposit. Must be positive.
        
        Returns:
            New balance after deposit.
        
        Raises:
            ValueError: If amount is not positive.
        
        Example:
            >>> account = BankAccount("ACC001", "Alice")
            >>> account.deposit(100.0)
            100.0
        """
        if amount <= 0:
            raise ValueError(f"Deposit amount must be positive, got {amount}")
        
        self.balance += amount
        self.transaction_history.append({
            'type': 'deposit',
            'amount': amount,
            'balance': self.balance
        })
        
        return self.balance
    
    def withdraw(self, amount: float) -> float:
        """Withdraw funds from the account.
        
        Args:
            amount: Amount to withdraw. Must be positive and
                not exceed current balance.
        
        Returns:
            New balance after withdrawal.
        
        Raises:
            ValueError: If amount is not positive.
            ValueError: If insufficient funds.
        """
        if amount <= 0:
            raise ValueError(f"Withdrawal amount must be positive, got {amount}")
        if amount > self.balance:
            raise ValueError(
                f"Insufficient funds: balance={self.balance}, requested={amount}"
            )
        
        self.balance -= amount
        self.transaction_history.append({
            'type': 'withdrawal',
            'amount': amount,
            'balance': self.balance
        })
        
        return self.balance

# Test
from typing import List
account = BankAccount("ACC001", "Alice Smith", 1000.0)
account.deposit(500.0)
account.withdraw(200.0)

print(f"\nBankAccount Demo:")
print(f"  Balance: ${account.balance:.2f}")
print(f"  Transactions: {len(account.transaction_history)}")

# Compound interest
result = calculate_compound_interest(1000, 0.05, 10)
print(f"\nCompound interest $1000 at 5% for 10 years: ${result:.2f}")
```

```python
# ตัวอย่าง 15: NumPy style docstrings

def polynomial_roots(coefficients: List[float]) -> List[complex]:
    """
    Find the roots of a polynomial equation.
    
    Parameters
    ----------
    coefficients : List[float]
        Coefficients of the polynomial in decreasing degree order.
        For example, [1, -3, 2] represents x^2 - 3x + 2.
        Must have at least 2 elements (at least degree 1).
    
    Returns
    -------
    List[complex]
        List of complex roots. Real roots will have imaginary
        part of 0. Length equals len(coefficients) - 1.
    
    Raises
    ------
    ValueError
        If coefficients list has fewer than 2 elements.
    ValueError
        If the leading coefficient (coefficients[0]) is 0.
    
    Examples
    --------
    Finding roots of x^2 - 5x + 6 = 0 (roots are 2 and 3):
    
    >>> roots = polynomial_roots([1, -5, 6])
    >>> sorted(roots, key=lambda x: x.real)
    [(2+0j), (3+0j)]
    
    Finding roots of x^2 + 1 = 0 (complex roots):
    
    >>> roots = polynomial_roots([1, 0, 1])
    >>> roots
    [(-0-1j), 1j]
    
    Notes
    -----
    Uses NumPy's roots function internally for numerical stability.
    For symbolic roots, consider using SymPy.
    
    The implementation uses the companion matrix eigenvalue approach,
    which may have numerical precision issues for very high degree
    polynomials or very large/small coefficients.
    
    See Also
    --------
    numpy.roots : NumPy's polynomial root finder
    numpy.polynomial.Polynomial : Full polynomial class
    
    References
    ----------
    .. [1] Golub, G.H., Van Loan, C.F. "Matrix Computations",
           4th ed., Johns Hopkins University Press, 2013.
    """
    if len(coefficients) < 2:
        raise ValueError("Need at least 2 coefficients (degree 1 polynomial)")
    if coefficients[0] == 0:
        raise ValueError("Leading coefficient cannot be zero")
    
    try:
        import numpy as np
        return list(np.roots(coefficients))
    except ImportError:
        # Fallback for quadratic without numpy
        if len(coefficients) == 3:
            a, b, c = coefficients
            discriminant = b**2 - 4*a*c
            sqrt_disc = complex(discriminant)**0.5
            return [(-b + sqrt_disc)/(2*a), (-b - sqrt_disc)/(2*a)]
        raise ImportError("NumPy required for polynomial degree > 2")

print("\nNumPy style docstring example function:")
try:
    roots = polynomial_roots([1, -5, 6])
    print(f"  Roots of x²-5x+6: {[round(r.real, 2) for r in roots]}")
except Exception as e:
    print(f"  Demo: roots would be [2.0, 3.0]")
```

---

## 7. API Documentation (OpenAPI) <a name="openapi"></a>

```python
# ตัวอย่าง 16: OpenAPI/Swagger documentation
# pip install fastapi

OPENAPI_EXAMPLE = '''
from fastapi import FastAPI, HTTPException, Query, Path
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
from enum import Enum

app = FastAPI(
    title="User Management API",
    description="""
    ## User Management API
    
    This API provides endpoints to manage users in the system.
    
    ### Features
    * Create, read, update and delete users
    * User authentication and authorization  
    * Role-based access control
    
    ### Authentication
    Use Bearer token authentication:
    ```
    Authorization: Bearer <token>
    ```
    """,
    version="2.1.0",
    terms_of_service="https://example.com/terms/",
    contact={
        "name": "API Support",
        "url": "https://example.com/support",
        "email": "api@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
)

class UserStatus(str, Enum):
    """User account status."""
    ACTIVE = "active"
    INACTIVE = "inactive"
    PENDING = "pending"

class UserCreate(BaseModel):
    """Schema for creating a new user."""
    
    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        description="Full name of the user",
        example="John Doe"
    )
    email: str = Field(
        ...,
        description="User\'s email address",
        example="john.doe@example.com"
    )
    age: int = Field(
        ...,
        ge=18,
        le=120,
        description="User age (must be 18 or older)",
        example=25
    )
    role: str = Field(
        default="user",
        description="User role in the system",
        example="user"
    )
    
    class Config:
        schema_extra = {
            "example": {
                "name": "John Doe",
                "email": "john@example.com",
                "age": 30,
                "role": "user"
            }
        }

class UserResponse(BaseModel):
    """Schema for user response."""
    id: int
    name: str
    email: str
    age: int
    role: str
    status: UserStatus
    created_at: str

@app.post(
    "/users",
    response_model=UserResponse,
    status_code=201,
    tags=["Users"],
    summary="Create new user",
    description="Create a new user account in the system",
    responses={
        201: {"description": "User created successfully"},
        400: {"description": "Invalid input data"},
        409: {"description": "Email already exists"},
    }
)
async def create_user(user: UserCreate):
    """
    Create a new user with the following information:
    
    - **name**: Full name (required)
    - **email**: Valid email address (required, must be unique)
    - **age**: Age in years (must be 18+)
    - **role**: User role (default: "user")
    """
    pass

@app.get(
    "/users/{user_id}",
    response_model=UserResponse,
    tags=["Users"],
    summary="Get user by ID",
    responses={
        200: {"description": "User found"},
        404: {"description": "User not found"},
    }
)
async def get_user(
    user_id: int = Path(..., description="The ID of the user", ge=1),
):
    pass

@app.get(
    "/users",
    response_model=List[UserResponse],
    tags=["Users"],
    summary="List all users",
)
async def list_users(
    status: Optional[UserStatus] = Query(None, description="Filter by status"),
    page: int = Query(1, ge=1, description="Page number"),
    size: int = Query(20, ge=1, le=100, description="Items per page"),
):
    pass
'''

print("OpenAPI Documentation Example:")
print(OPENAPI_EXAMPLE[:500])
print("... (Full code available)")

print("\nOpenAPI features in FastAPI:")
print("  - Auto-generated Swagger UI at /docs")
print("  - OpenAPI JSON at /openapi.json")
print("  - ReDoc at /redoc")
print("  - Request/Response schemas with examples")
print("  - Status code documentation")
```

---

## 8. MkDocs <a name="mkdocs"></a>

```python
# ตัวอย่าง 17: MkDocs configuration
# pip install mkdocs mkdocs-material mkdocstrings

MKDOCS_YML = """
site_name: My Python Project
site_url: https://myproject.github.io/docs/
site_author: Your Name
site_description: Comprehensive documentation for My Python Project

repo_name: username/myproject
repo_url: https://github.com/username/myproject
edit_uri: edit/main/docs/

theme:
  name: material
  palette:
    - scheme: default
      primary: blue
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      primary: blue
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - navigation.top
    - search.suggest
    - search.highlight
    - content.tabs.link
    - content.code.annotate
    - content.code.copy

plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          options:
            docstring_style: google
            show_source: true
            show_root_heading: true

markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.tabbed:
      alternate_style: true

nav:
  - Home: index.md
  - Getting Started:
    - Installation: getting-started/installation.md
    - Quickstart: getting-started/quickstart.md
    - Configuration: getting-started/configuration.md
  - User Guide:
    - Core Concepts: guide/concepts.md
    - Authentication: guide/authentication.md
    - API Reference: guide/api.md
  - API Reference:
    - Models: api/models.md
    - Services: api/services.md
    - Utilities: api/utils.md
  - Contributing:
    - Development Setup: contributing/setup.md
    - Code Style: contributing/style.md
    - Testing: contributing/testing.md
  - Changelog: changelog.md
"""

print("MkDocs configuration (mkdocs.yml):")
print(MKDOCS_YML)

print("""
MkDocs commands:
  mkdocs serve           - Preview locally at http://127.0.0.1:8000
  mkdocs build           - Build static site to site/ directory
  mkdocs gh-deploy       - Deploy to GitHub Pages
""")
```

---

## 9. Pre-commit Hooks <a name="pre-commit"></a>

```python
# ตัวอย่าง 18: Pre-commit hooks
# pip install pre-commit

PRE_COMMIT_CONFIG = """
# .pre-commit-config.yaml
repos:
  # Standard hooks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-toml
      - id: check-merge-conflict
      - id: check-added-large-files
        args: ['--maxkb=500']
      - id: detect-private-key
      - id: debug-statements     # เช็ค breakpoint(), pdb.set_trace()
      - id: check-docstring-first
      
  # Black formatter
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3.11
  
  # isort
  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ['--profile', 'black']
  
  # Ruff linter
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
  
  # mypy type checker
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies:
          - types-requests
          - types-redis
  
  # Security scanning
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.6
    hooks:
      - id: bandit
        args: ['-c', 'pyproject.toml']
  
  # Commit message format
  - repo: https://github.com/commitizen-tools/commitizen
    rev: v3.13.0
    hooks:
      - id: commitizen
        stages: [commit-msg]
"""

print("Pre-commit configuration (.pre-commit-config.yaml):")
print(PRE_COMMIT_CONFIG)

print("""
Pre-commit commands:
  pre-commit install          - Install hooks
  pre-commit run --all-files  - Run on all files
  pre-commit run black        - Run specific hook
  pre-commit autoupdate       - Update hook versions
""")
```

```python
# ตัวอย่าง 19: Custom pre-commit hook
CUSTOM_HOOK = """
# hooks/check_api_version.py
import sys
import re
import subprocess

def check_api_version_bump():
    '''Ensure API version is bumped when API files change'''
    
    # Get changed files
    result = subprocess.run(
        ['git', 'diff', '--cached', '--name-only'],
        capture_output=True, text=True
    )
    changed_files = result.stdout.strip().split('\\n')
    
    # Check if API files were changed
    api_files_changed = any(
        'api/' in f or f.endswith('_api.py') 
        for f in changed_files
    )
    
    if not api_files_changed:
        return 0
    
    # Check if version was bumped
    version_files_changed = any(
        f in ['version.py', 'setup.py', 'pyproject.toml']
        for f in changed_files
    )
    
    if not version_files_changed:
        print("WARNING: API files changed but version not updated!")
        print("Consider bumping the version in pyproject.toml")
        # Return 0 to warn only (not fail)
        # Return 1 to fail the commit
        return 0
    
    return 0

if __name__ == '__main__':
    sys.exit(check_api_version_bump())
"""

print("Custom pre-commit hook:")
print(CUSTOM_HOOK)
```

---

## 10. Code Review Best Practices <a name="code-review"></a>

```python
# ตัวอย่าง 20: Code review checklist and guidelines

CODE_REVIEW_CHECKLIST = """
Code Review Checklist:

CORRECTNESS
✓ Does the code do what the PR description says?
✓ Are edge cases handled?
✓ Is error handling appropriate?
✓ Are there potential race conditions?
✓ Is data validation thorough?

DESIGN
✓ Is the code following SOLID principles?
✓ Are responsibilities properly separated?
✓ Is there unnecessary coupling?
✓ Could this be simpler?
✓ Are abstractions appropriate?

PERFORMANCE
✓ Are there obvious N+1 query problems?
✓ Is there unnecessary computation in loops?
✓ Are heavy resources cached appropriately?
✓ Is pagination used for large datasets?

SECURITY
✓ Is user input validated and sanitized?
✓ Are SQL queries parameterized?
✓ Are sensitive data handled securely?
✓ Are authentication/authorization checks in place?
✓ Are secrets not hardcoded?

TESTING
✓ Are there tests for the new functionality?
✓ Do tests cover edge cases and error paths?
✓ Are tests meaningful (not just coverage)?

DOCUMENTATION
✓ Are complex parts documented?
✓ Are API changes reflected in docs?
✓ Is the PR description clear and complete?

CODE STYLE
✓ Does it follow project style guidelines?
✓ Are names descriptive and consistent?
✓ Is there duplicate code that should be refactored?
"""

print(CODE_REVIEW_CHECKLIST)
```

```python
# ตัวอย่าง 21: Good vs bad code review comments

REVIEW_EXAMPLES = """
Code Review Comment Examples:

=== BAD COMMENTS ===
❌ "This is wrong"
❌ "Why did you do it this way?"
❌ "This is bad code"
❌ "You need to fix this"
❌ "Clearly wrong approach"

=== GOOD COMMENTS ===
✓ Nit: Consider using a set here for O(1) lookup instead of list O(n)
  result = set(items)  # faster than list for contains check

✓ Question: What happens if `user_id` is None? 
  Should we add a guard clause at the top?

✓ Suggestion: This logic is repeated in UserService.create(). 
  Could we extract this into a shared utility function?

✓ Bug: If `items` is empty, `items[0]` will raise IndexError.
  Consider using `items[0] if items else None`

✓ Performance: This queries the DB in a loop (N+1).
  Consider using a single JOIN query:
  users = User.query.filter(User.id.in_(user_ids)).all()

✓ Security: User input passed directly to format string.
  Use parameterized query instead:
  cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
"""

print("Code Review Comment Guidelines:")
print(REVIEW_EXAMPLES)
```

---

## 11. Technical Debt Management <a name="tech-debt"></a>

```python
# ตัวอย่าง 22: Technical debt tracking

from dataclasses import dataclass, field
from enum import Enum
from typing import List, Optional
from datetime import date

class DebtSeverity(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

class DebtCategory(Enum):
    CODE_SMELL = "code_smell"
    OUTDATED_DEPENDENCY = "outdated_dependency"
    MISSING_TESTS = "missing_tests"
    MISSING_DOCS = "missing_docs"
    SECURITY = "security"
    PERFORMANCE = "performance"
    ARCHITECTURE = "architecture"

@dataclass
class TechDebtItem:
    """Represents a single technical debt item"""
    
    id: str
    title: str
    description: str
    category: DebtCategory
    severity: DebtSeverity
    file_path: str
    line_number: Optional[int] = None
    estimated_hours: float = 0.0
    assigned_to: Optional[str] = None
    created_date: date = field(default_factory=date.today)
    target_date: Optional[date] = None
    resolved: bool = False
    tags: List[str] = field(default_factory=list)
    
    def to_todo_comment(self) -> str:
        """Generate TODO comment for code"""
        tag = f"TODO[{self.id}]"
        return f"# {tag}: {self.title} [{self.severity.value}] - {self.description}"
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'title': self.title,
            'severity': self.severity.value,
            'category': self.category.value,
            'estimated_hours': self.estimated_hours,
            'resolved': self.resolved,
        }


class TechDebtRegistry:
    """Registry สำหรับ track technical debt"""
    
    def __init__(self):
        self._items: List[TechDebtItem] = []
    
    def add(self, item: TechDebtItem):
        self._items.append(item)
    
    def get_by_severity(self, severity: DebtSeverity) -> List[TechDebtItem]:
        return [i for i in self._items if i.severity == severity and not i.resolved]
    
    def get_by_category(self, category: DebtCategory) -> List[TechDebtItem]:
        return [i for i in self._items if i.category == category and not i.resolved]
    
    def get_total_debt_hours(self) -> float:
        return sum(i.estimated_hours for i in self._items if not i.resolved)
    
    def generate_report(self) -> str:
        active = [i for i in self._items if not i.resolved]
        resolved = [i for i in self._items if i.resolved]
        
        lines = ["Technical Debt Report", "=" * 40]
        lines.append(f"Active items: {len(active)}")
        lines.append(f"Resolved items: {len(resolved)}")
        lines.append(f"Total estimated hours: {self.get_total_debt_hours():.1f}h")
        lines.append("")
        
        for severity in [DebtSeverity.CRITICAL, DebtSeverity.HIGH, 
                         DebtSeverity.MEDIUM, DebtSeverity.LOW]:
            items = self.get_by_severity(severity)
            if items:
                lines.append(f"{severity.value.upper()} ({len(items)} items):")
                for item in items:
                    lines.append(f"  [{item.id}] {item.title} ({item.estimated_hours}h)")
        
        return "\n".join(lines)

# Usage
registry = TechDebtRegistry()

registry.add(TechDebtItem(
    id="TD-001",
    title="Replace synchronous DB calls with async",
    description="UserService.get_user() blocks the event loop",
    category=DebtCategory.PERFORMANCE,
    severity=DebtSeverity.HIGH,
    file_path="src/services/user_service.py",
    line_number=45,
    estimated_hours=8.0,
    tags=["async", "database"]
))

registry.add(TechDebtItem(
    id="TD-002",
    title="Add missing unit tests for PaymentService",
    description="Coverage below 60% for payment logic",
    category=DebtCategory.MISSING_TESTS,
    severity=DebtSeverity.MEDIUM,
    file_path="src/services/payment_service.py",
    estimated_hours=16.0,
    tags=["testing", "coverage"]
))

registry.add(TechDebtItem(
    id="TD-003",
    title="Upgrade SQLAlchemy to 2.0",
    description="Using deprecated 1.x API",
    category=DebtCategory.OUTDATED_DEPENDENCY,
    severity=DebtSeverity.LOW,
    file_path="requirements.txt",
    estimated_hours=24.0,
    tags=["dependency", "sqlalchemy"]
))

print("\nTechnical Debt Registry:")
print(registry.generate_report())
print(f"\nSample TODO comment: {registry._items[0].to_todo_comment()}")
```

---

## 12. Refactoring Safely <a name="refactoring"></a>

```python
# ตัวอย่าง 23: Refactoring techniques

# ก่อน Refactoring - code smell
def process_order_v1(order_data):
    """Old messy code"""
    # Calculate total (duplicated logic)
    t = 0
    for i in order_data['items']:
        t = t + i['p'] * i['q']
    
    # Apply discount (magic numbers)
    if order_data['customer_type'] == 'premium':
        t = t * 0.9
    elif order_data['customer_type'] == 'vip':
        t = t * 0.8
    
    # Tax (magic number)
    tax = t * 0.07
    
    # Shipping (complex nested condition)
    if t > 100:
        if order_data.get('express'):
            s = 0
        else:
            s = 0
    else:
        if order_data.get('express'):
            s = 25
        else:
            s = 10
    
    result = t + tax + s
    return result


# หลัง Refactoring - clean code
DISCOUNT_RATES = {
    'new': 0.0,
    'regular': 0.05,
    'premium': 0.10,
    'vip': 0.20,
}

TAX_RATE = 0.07
FREE_SHIPPING_THRESHOLD = 100.0
EXPRESS_SHIPPING_COST = 25.0
STANDARD_SHIPPING_COST = 10.0


def calculate_items_subtotal(items: list) -> float:
    """Calculate subtotal from order items."""
    return sum(item['price'] * item['quantity'] for item in items)


def calculate_discount(subtotal: float, customer_type: str) -> float:
    """Calculate discount based on customer type."""
    rate = DISCOUNT_RATES.get(customer_type, 0.0)
    return subtotal * rate


def calculate_tax(amount: float) -> float:
    """Calculate tax amount."""
    return round(amount * TAX_RATE, 2)


def calculate_shipping(subtotal: float, express: bool = False) -> float:
    """Calculate shipping cost."""
    if subtotal >= FREE_SHIPPING_THRESHOLD:
        return 0.0
    return EXPRESS_SHIPPING_COST if express else STANDARD_SHIPPING_COST


def process_order_v2(order_data: dict) -> dict:
    """Process order and calculate total."""
    items = order_data['items']
    customer_type = order_data.get('customer_type', 'new')
    is_express = order_data.get('express', False)
    
    subtotal = calculate_items_subtotal(items)
    discount = calculate_discount(subtotal, customer_type)
    discounted_subtotal = subtotal - discount
    tax = calculate_tax(discounted_subtotal)
    shipping = calculate_shipping(discounted_subtotal, is_express)
    
    total = discounted_subtotal + tax + shipping
    
    return {
        'subtotal': subtotal,
        'discount': discount,
        'tax': tax,
        'shipping': shipping,
        'total': round(total, 2)
    }

# Test both produce same result
old_order = {
    'items': [{'p': 50, 'q': 2}, {'p': 30, 'q': 1}],
    'customer_type': 'premium',
    'express': False
}
new_order = {
    'items': [{'price': 50, 'quantity': 2}, {'price': 30, 'quantity': 1}],
    'customer_type': 'premium',
    'express': False
}

old_total = process_order_v1(old_order)
new_result = process_order_v2(new_order)

print("\nRefactoring Demo:")
print(f"  Old total: ${old_total:.2f}")
print(f"  New total: ${new_result['total']:.2f}")
print(f"  Results match: {abs(old_total - new_result['total']) < 0.01}")
print(f"\nNew result breakdown:")
for key, value in new_result.items():
    print(f"  {key}: ${value:.2f}")
```

---

## 13. Git Best Practices <a name="git"></a>

```python
# ตัวอย่าง 24: Git workflow และ conventions

GIT_WORKFLOW = """
Git Best Practices:

1. BRANCHING STRATEGY (GitFlow)
main        - Production code only
develop     - Integration branch
feature/*   - New features
bugfix/*    - Bug fixes
hotfix/*    - Critical production fixes
release/*   - Release preparation

2. COMMIT MESSAGE FORMAT (Conventional Commits)
<type>(<scope>): <description>

Types:
feat:     New feature
fix:      Bug fix
docs:     Documentation changes
style:    Code formatting (no logic change)
refactor: Code refactoring
perf:     Performance improvement
test:     Adding or updating tests
chore:    Build/config changes
ci:       CI/CD changes
revert:   Revert previous commit

Examples:
feat(auth): add JWT token refresh endpoint
fix(cart): prevent duplicate items when adding same product
docs(api): update authentication documentation
test(user): add unit tests for password validation
refactor(db): extract query builder to separate class
perf(search): add index for full-text search
chore(deps): upgrade SQLAlchemy to 2.0

3. COMMIT MESSAGE RULES
- Present tense ("add feature" not "added feature")
- Imperative mood ("move cursor" not "moves cursor")
- Under 72 characters for subject line
- Add blank line before body
- Body explains WHY, not WHAT
- Reference issues: "Closes #123"

4. BRANCH NAMING
feature/user-authentication
feature/payment-gateway-integration
bugfix/cart-total-calculation
hotfix/security-csrf-token
release/v2.1.0
"""

print(GIT_WORKFLOW)

# .gitignore for Python projects
GITIGNORE = """
# .gitignore for Python

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
*.egg
*.egg-info/
dist/
build/
.eggs/
.mypy_cache/
.pytest_cache/
.ruff_cache/

# Virtual environments
.env
.venv
env/
venv/
ENV/

# IDE
.idea/
.vscode/
*.swp
*.swo

# Testing
.coverage
coverage.xml
htmlcov/
.hypothesis/
.tox/

# Secrets
.env.local
.env.production
*.pem
*.key
secrets.yaml

# Logs
*.log
logs/

# Database
*.db
*.sqlite3

# macOS
.DS_Store

# Documentation
docs/_build/
site/
"""

print("\n.gitignore template:")
print(GITIGNORE)
```

---

## 14. Semantic Versioning <a name="semver"></a>

```python
# ตัวอย่าง 25: Semantic Versioning

"""
Semantic Versioning: MAJOR.MINOR.PATCH

MAJOR = incompatible API changes
MINOR = add backward-compatible functionality
PATCH = backward-compatible bug fixes

Examples:
1.0.0 - Initial stable release
1.0.1 - Bug fix
1.1.0 - New feature added
1.1.1 - Bug fix for new feature
2.0.0 - Breaking API change

Pre-release versions:
1.0.0-alpha.1
1.0.0-beta.2
1.0.0-rc.1

Build metadata:
1.0.0+build.123
1.0.0+sha.5114f85
"""

from dataclasses import dataclass
from typing import Optional
import re

@dataclass
class SemanticVersion:
    """Semantic version representation"""
    
    major: int
    minor: int
    patch: int
    pre_release: Optional[str] = None
    build: Optional[str] = None
    
    def __str__(self) -> str:
        version = f"{self.major}.{self.minor}.{self.patch}"
        if self.pre_release:
            version += f"-{self.pre_release}"
        if self.build:
            version += f"+{self.build}"
        return version
    
    def __lt__(self, other: 'SemanticVersion') -> bool:
        if (self.major, self.minor, self.patch) != (other.major, other.minor, other.patch):
            return (self.major, self.minor, self.patch) < (other.major, other.minor, other.patch)
        
        # Pre-release versions have lower precedence
        if self.pre_release and not other.pre_release:
            return True
        if not self.pre_release and other.pre_release:
            return False
        if self.pre_release and other.pre_release:
            return self.pre_release < other.pre_release
        
        return False
    
    def __eq__(self, other: 'SemanticVersion') -> bool:
        return (self.major == other.major and 
                self.minor == other.minor and 
                self.patch == other.patch and
                self.pre_release == other.pre_release)
    
    def bump_major(self) -> 'SemanticVersion':
        return SemanticVersion(self.major + 1, 0, 0)
    
    def bump_minor(self) -> 'SemanticVersion':
        return SemanticVersion(self.major, self.minor + 1, 0)
    
    def bump_patch(self) -> 'SemanticVersion':
        return SemanticVersion(self.major, self.minor, self.patch + 1)
    
    def is_compatible_with(self, other: 'SemanticVersion') -> bool:
        """Check if compatible (same MAJOR version)"""
        return self.major == other.major
    
    @classmethod
    def parse(cls, version_str: str) -> 'SemanticVersion':
        """Parse version string"""
        pattern = r'^(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z-.]+))?(?:\+([0-9A-Za-z-.]+))?$'
        match = re.match(pattern, version_str)
        
        if not match:
            raise ValueError(f"Invalid version string: {version_str}")
        
        major, minor, patch, pre_release, build = match.groups()
        return cls(
            major=int(major),
            minor=int(minor),
            patch=int(patch),
            pre_release=pre_release,
            build=build
        )


# Test
v1 = SemanticVersion.parse("2.1.0")
v2 = SemanticVersion.parse("2.1.1")
v3 = SemanticVersion.parse("2.0.0-beta.1")
v4 = v1.bump_major()
v5 = v1.bump_minor()

print("\nSemantic Versioning:")
print(f"  v1: {v1}")
print(f"  v2: {v2}")
print(f"  v3: {v3}")
print(f"  v1 bump_major: {v4}")
print(f"  v1 bump_minor: {v5}")
print(f"  v1 < v2: {v1 < v2}")
print(f"  v3 < v1: {v3 < v1}")
print(f"  v1 compatible with v2: {v1.is_compatible_with(v2)}")
print(f"  v1 compatible with v4: {v1.is_compatible_with(v4)}")
```

---

## 15. Changelog Management <a name="changelog"></a>

```python
# ตัวอย่าง 26: Changelog generation

CHANGELOG_TEMPLATE = """
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New feature X that does Y

### Changed
- Updated Z to work differently

## [2.1.0] - 2024-01-15

### Added
- JWT token refresh endpoint (#234)
- Bulk user import API (#245)
- Elasticsearch integration for full-text search (#251)

### Changed
- Improved password validation with more entropy checks (#238)
- Updated user profile endpoints to support partial updates (#242)

### Fixed
- Fixed cart total calculation with multiple discounts (#235)
- Resolved race condition in inventory updates (#241)
- Corrected tax calculation for international orders (#248)

### Security
- Fixed SQL injection vulnerability in search endpoint (#240)
- Updated cryptography library to 41.0.5 (#244)

## [2.0.0] - 2023-12-01

### Breaking Changes
- Renamed `/api/v1/` to `/api/v2/` - old endpoints deprecated
- Changed authentication from session to JWT tokens
- Modified user response schema (removed deprecated fields)

### Added
- Complete API v2 with improved performance
- Real-time notifications via WebSocket
- Async database operations

### Removed
- Removed deprecated `/api/v1/` endpoints
- Removed XML response format support
"""

print("Changelog template (CHANGELOG.md):")
print(CHANGELOG_TEMPLATE)

# Auto-generate changelog from git commits
def parse_conventional_commits(commits: list) -> dict:
    """Parse conventional commits and group by type"""
    groups = {
        'feat': [],
        'fix': [],
        'docs': [],
        'perf': [],
        'security': [],
        'breaking': [],
    }
    
    pattern = re.compile(
        r'^(?P<type>feat|fix|docs|style|refactor|perf|test|chore|ci)'
        r'(?:\((?P<scope>[^)]+)\))?'
        r'(?P<breaking>!)?'
        r': (?P<description>.+)$'
    )
    
    for commit in commits:
        match = pattern.match(commit.get('message', ''))
        if match:
            ctype = match.group('type')
            scope = match.group('scope') or ''
            is_breaking = bool(match.group('breaking'))
            description = match.group('description')
            
            entry = {
                'scope': scope,
                'description': description,
                'hash': commit.get('hash', '')[:7]
            }
            
            if is_breaking:
                groups['breaking'].append(entry)
            elif ctype == 'feat':
                groups['feat'].append(entry)
            elif ctype == 'fix':
                groups['fix'].append(entry)
            elif ctype == 'perf':
                groups['perf'].append(entry)
    
    return groups

def generate_changelog_entry(version: str, date: str, groups: dict) -> str:
    """Generate changelog entry from parsed commits"""
    lines = [f"## [{version}] - {date}", ""]
    
    type_map = {
        'breaking': '### Breaking Changes',
        'feat': '### Added',
        'fix': '### Fixed',
        'perf': '### Performance',
        'security': '### Security',
    }
    
    for group_key, title in type_map.items():
        items = groups.get(group_key, [])
        if items:
            lines.append(title)
            for item in items:
                scope = f"**{item['scope']}**: " if item['scope'] else ""
                lines.append(f"- {scope}{item['description']} ({item['hash']})")
            lines.append("")
    
    return "\n".join(lines)

# Test
sample_commits = [
    {"message": "feat(auth): add JWT refresh token endpoint", "hash": "abc1234"},
    {"message": "fix(cart): fix total calculation with discounts", "hash": "def5678"},
    {"message": "feat!: rename API v1 to v2", "hash": "ghi9012"},
    {"message": "perf(db): add index for user email lookup", "hash": "jkl3456"},
    {"message": "fix(security): sanitize search input", "hash": "mno7890"},
]

groups = parse_conventional_commits(sample_commits)
changelog = generate_changelog_entry("2.1.0", "2024-01-15", groups)

print("\nAuto-generated Changelog Entry:")
print(changelog)
```

---

## 16. แบบฝึกหัด <a name="exercises"></a>

### แบบฝึกหัดที่ 1: PEP 8 Cleanup

```python
# ตัวอย่าง code ที่ผิด PEP 8 - ให้แก้ไข
MESSY_CODE = """
import os,sys,json
from typing import *
x=1
y=2

class userAccount:
    def __init__ (self,name,email,age):
        self.Name=name
        self.Email=email
        self.Age=age
    def GetInfo(self):
        return {'name':self.Name,'email':self.Email,'age':self.Age}
    def is_adult(self):
        if self.Age>=18:
            return True
        else:
            return False

def calculate(price,discount,tax_rate=0.07):
    r=price*(1-discount/100)
    r=r*(1+tax_rate)
    return r
"""

# เฉลย - clean version
CLEAN_CODE = """
import json
import os
import sys
from typing import Optional


class UserAccount:
    def __init__(self, name: str, email: str, age: int) -> None:
        self.name = name
        self.email = email
        self.age = age
    
    def get_info(self) -> dict:
        return {
            'name': self.name,
            'email': self.email,
            'age': self.age,
        }
    
    def is_adult(self) -> bool:
        return self.age >= 18


def calculate(
    price: float,
    discount: float,
    tax_rate: float = 0.07,
) -> float:
    discounted = price * (1 - discount / 100)
    return discounted * (1 + tax_rate)
"""

print("Ex 1 - PEP 8 Cleanup:")
print("Messy code has been cleaned up!")
print(CLEAN_CODE)
```

```python
# แบบฝึกหัดที่ 2: เพิ่ม Type Hints ครบถ้วน

# ก่อนเพิ่ม type hints
def before_types(data, filter_func, transform_func):
    result = []
    for item in data:
        if filter_func(item):
            transformed = transform_func(item)
            result.append(transformed)
    return result

# เฉลย: หลังเพิ่ม type hints
from typing import TypeVar, Callable, Iterable, List

T = TypeVar('T')
U = TypeVar('U')

def after_types(
    data: Iterable[T],
    filter_func: Callable[[T], bool],
    transform_func: Callable[[T], U],
) -> List[U]:
    """Filter and transform items from iterable.
    
    Args:
        data: Input iterable of items to process.
        filter_func: Predicate function to filter items.
        transform_func: Function to transform filtered items.
    
    Returns:
        List of transformed items that passed the filter.
    
    Example:
        >>> result = after_types(
        ...     range(10),
        ...     filter_func=lambda x: x % 2 == 0,
        ...     transform_func=lambda x: x ** 2
        ... )
        >>> result
        [0, 4, 16, 36, 64]
    """
    return [
        transform_func(item)
        for item in data
        if filter_func(item)
    ]

print("\nEx 2 - Type Hints:")
result = after_types(
    range(10),
    filter_func=lambda x: x % 2 == 0,
    transform_func=lambda x: x ** 2
)
print(f"  Result: {result}")
```

```python
# แบบฝึกหัดที่ 3: Write comprehensive docstrings

class DataValidator:
    """Validator for business data with comprehensive docstrings"""
    
    def validate_email(self, email: str) -> bool:
        """Validate email address format.
        
        Uses a simplified regex pattern that covers most common email formats.
        Does not verify if the email actually exists.
        
        Args:
            email: Email address to validate.
                Must be a string.
        
        Returns:
            True if email format is valid, False otherwise.
        
        Example:
            >>> validator = DataValidator()
            >>> validator.validate_email("user@example.com")
            True
            >>> validator.validate_email("invalid-email")
            False
            >>> validator.validate_email("")
            False
        
        Note:
            This function only validates format, not deliverability.
            For production use, consider sending a verification email.
        """
        if not email or not isinstance(email, str):
            return False
        
        import re
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, email))
    
    def validate_phone(self, phone: str, country_code: str = "TH") -> bool:
        """Validate phone number for specified country.
        
        Args:
            phone: Phone number to validate.
                Can include country code prefix (+66).
            country_code: ISO 3166-1 alpha-2 country code.
                Currently supports: TH (Thailand), US (United States).
                Defaults to "TH".
        
        Returns:
            True if phone number is valid for the country.
        
        Raises:
            ValueError: If country_code is not supported.
        
        Example:
            >>> validator = DataValidator()
            >>> validator.validate_phone("0812345678", "TH")
            True
            >>> validator.validate_phone("+66812345678", "TH")
            True
            >>> validator.validate_phone("1234567890", "US")
            True
        """
        import re
        
        patterns = {
            "TH": r'^(\+66|0)[89]\d{8}$',
            "US": r'^\+?1?\d{10}$',
        }
        
        if country_code not in patterns:
            raise ValueError(f"Unsupported country code: {country_code}")
        
        if not phone:
            return False
        
        phone_clean = re.sub(r'[\s\-\(\)]', '', phone)
        return bool(re.match(patterns[country_code], phone_clean))

print("\nEx 3 - Docstrings:")
v = DataValidator()
print(f"  validate_email('user@example.com'): {v.validate_email('user@example.com')}")
print(f"  validate_email('invalid'): {v.validate_email('invalid')}")
print(f"  validate_phone('0812345678', 'TH'): {v.validate_phone('0812345678', 'TH')}")
```

```python
# แบบฝึกหัดที่ 4: Create pre-commit hook simulation

import os
import re
import ast
from pathlib import Path
from typing import List, Tuple

class CodeChecker:
    """Simulates pre-commit hooks"""
    
    def __init__(self):
        self.issues: List[Tuple[str, int, str]] = []
    
    def check_file(self, content: str, filename: str = "test.py") -> List[str]:
        """Check code content for common issues"""
        issues = []
        lines = content.split('\n')
        
        for i, line in enumerate(lines, 1):
            # Check line length
            if len(line) > 88:
                issues.append(f"L{i}: Line too long ({len(line)} > 88 chars)")
            
            # Check for debug statements
            if re.search(r'\bpdb\.set_trace\(\)|breakpoint\(\)', line):
                issues.append(f"L{i}: Debug statement found: {line.strip()}")
            
            # Check for TODO without ticket
            todo_match = re.search(r'#\s*TODO\s*:', line, re.IGNORECASE)
            if todo_match and not re.search(r'TODO\[', line):
                issues.append(f"L{i}: TODO without ticket number")
            
            # Check for print statements in non-test files
            if 'test' not in filename and re.search(r'^\s*print\(', line):
                issues.append(f"L{i}: print() found (use logger instead)")
            
            # Check for == None (should be 'is None')
            if re.search(r'==\s*None', line):
                issues.append(f"L{i}: Use 'is None' instead of '== None'")
            
            # Check for trailing whitespace
            if line != line.rstrip():
                issues.append(f"L{i}: Trailing whitespace")
        
        return issues
    
    def check_syntax(self, content: str) -> List[str]:
        """Check Python syntax"""
        try:
            ast.parse(content)
            return []
        except SyntaxError as e:
            return [f"SyntaxError: {e.msg} at line {e.lineno}"]


# Test
checker = CodeChecker()

test_code = """
import os

x = None
if x == None:  # should use 'is None'
    print("x is None")  # debug print

# TODO: Fix this later (no ticket number)
result = some_function()   

def calculate(a, b):
    # breakpoint()  # debug
    return a + b
"""

print("\nEx 4 - Pre-commit Hook Simulation:")
issues = checker.check_file(test_code)
print(f"  Found {len(issues)} issues:")
for issue in issues:
    print(f"    ⚠ {issue}")
```

```python
# แบบฝึกหัดที่ 5: Refactor code with proper style

# โค้ดที่ต้อง refactor
def bad_code_1(lst):
    r = []
    for i in range(len(lst)):
        if lst[i]%2==0:
            r.append(lst[i]*lst[i])
    return r

def bad_code_2(d):
    keys = []
    for k in d.keys():
        keys.append(k)
    return sorted(keys)

def bad_code_3(x, y):
    if x == True:
        if y == True:
            return "both"
        else:
            return "x only"
    else:
        if y == True:
            return "y only"
        else:
            return "neither"

# เฉลย
def even_squares(numbers: List[int]) -> List[int]:
    """Return squares of even numbers in the list."""
    return [n ** 2 for n in numbers if n % 2 == 0]

def sorted_keys(mapping: dict) -> List:
    """Return sorted list of dictionary keys."""
    return sorted(mapping.keys())

def describe_conditions(x: bool, y: bool) -> str:
    """Describe which conditions are true."""
    if x and y:
        return "both"
    elif x:
        return "x only"
    elif y:
        return "y only"
    return "neither"

print("\nEx 5 - Refactoring:")
nums = [1, 2, 3, 4, 5, 6, 7, 8]
print(f"  Original even_squares([1..8]): {bad_code_1(nums)}")
print(f"  Refactored even_squares([1..8]): {even_squares(nums)}")
assert bad_code_1(nums) == even_squares(nums), "Results don't match!"
print("  ✓ Results match!")

d = {'z': 3, 'a': 1, 'm': 2}
assert bad_code_2(d) == sorted_keys(d)
print(f"  ✓ sorted_keys works: {sorted_keys(d)}")

for x, y in [(True, True), (True, False), (False, True), (False, False)]:
    assert bad_code_3(x, y) == describe_conditions(x, y)
print(f"  ✓ describe_conditions works correctly")
```

```python
# แบบฝึกหัดที่ 6: Create changelog entry

from datetime import date
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class ChangelogEntry:
    version: str
    date: date
    added: List[str] = field(default_factory=list)
    changed: List[str] = field(default_factory=list)
    fixed: List[str] = field(default_factory=list)
    removed: List[str] = field(default_factory=list)
    security: List[str] = field(default_factory=list)
    breaking: List[str] = field(default_factory=list)
    
    def to_markdown(self) -> str:
        lines = [f"## [{self.version}] - {self.date.isoformat()}", ""]
        
        sections = [
            ("### Breaking Changes", self.breaking),
            ("### Added", self.added),
            ("### Changed", self.changed),
            ("### Fixed", self.fixed),
            ("### Removed", self.removed),
            ("### Security", self.security),
        ]
        
        for title, items in sections:
            if items:
                lines.append(title)
                for item in items:
                    lines.append(f"- {item}")
                lines.append("")
        
        return "\n".join(lines)

print("\nEx 6 - Changelog Entry:")
entry = ChangelogEntry(
    version="2.1.0",
    date=date(2024, 1, 15),
    added=[
        "JWT token refresh endpoint",
        "Bulk user import API",
    ],
    fixed=[
        "Cart total calculation with multiple discounts",
        "Race condition in inventory updates",
    ],
    security=[
        "Fixed XSS vulnerability in user profile display",
    ]
)
print(entry.to_markdown())
```

```python
# แบบฝึกหัดที่ 7: Version comparison

def test_version_comparison():
    versions = [
        SemanticVersion.parse("1.0.0"),
        SemanticVersion.parse("2.0.0"),
        SemanticVersion.parse("1.5.0"),
        SemanticVersion.parse("1.0.1"),
        SemanticVersion.parse("2.1.0"),
        SemanticVersion.parse("1.0.0-beta.1"),
        SemanticVersion.parse("1.0.0-alpha.1"),
    ]
    
    sorted_versions = sorted(versions)
    
    print("\nEx 7 - Version Sorting:")
    print("Sorted versions:")
    for v in sorted_versions:
        print(f"  {v}")
    
    v1 = SemanticVersion.parse("2.1.0")
    v2 = SemanticVersion.parse("3.0.0")
    
    print(f"\nBump operations on {v1}:")
    print(f"  bump_major: {v1.bump_major()}")
    print(f"  bump_minor: {v1.bump_minor()}")
    print(f"  bump_patch: {v1.bump_patch()}")
    
    print(f"\nCompatibility:")
    print(f"  {v1} compatible with {v2}: {v1.is_compatible_with(v2)}")
    print(f"  {v1} compatible with {v1.bump_minor()}: {v1.is_compatible_with(v1.bump_minor())}")

test_version_comparison()
```

```python
# แบบฝึกหัดที่ 8: Complete quality pipeline

import subprocess
import sys
from typing import Tuple

class QualityPipeline:
    """Simulate a code quality pipeline"""
    
    def __init__(self):
        self.results = {}
    
    def run_style_check(self, code: str) -> Tuple[bool, List[str]]:
        """Check PEP 8 style"""
        issues = []
        lines = code.split('\n')
        
        for i, line in enumerate(lines, 1):
            if len(line) > 88:
                issues.append(f"E501 line {i}: line too long")
            if '\t' in line:
                issues.append(f"W191 line {i}: indentation contains tabs")
        
        return len(issues) == 0, issues
    
    def run_type_check(self, code: str) -> Tuple[bool, List[str]]:
        """Simulate mypy type checking"""
        issues = []
        
        # Simplified checks
        if 'def ' in code and '->' not in code:
            issues.append("note: Function missing return type annotation")
        
        return len(issues) == 0, issues
    
    def run_security_check(self, code: str) -> Tuple[bool, List[str]]:
        """Basic security scan"""
        issues = []
        
        dangerous_patterns = [
            (r'eval\(', "B307: Use of eval() is dangerous"),
            (r'exec\(', "B102: Use of exec() is dangerous"),
            (r'pickle\.loads', "B301: pickle.loads is unsafe"),
            (r'shell=True', "B602: subprocess with shell=True"),
        ]
        
        for pattern, message in dangerous_patterns:
            if re.search(pattern, code):
                issues.append(message)
        
        return len(issues) == 0, issues
    
    def run_docstring_check(self, code: str) -> Tuple[bool, List[str]]:
        """Check for missing docstrings"""
        issues = []
        
        try:
            tree = ast.parse(code)
            for node in ast.walk(tree):
                if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
                    if not ast.get_docstring(node):
                        if not node.name.startswith('_'):
                            issues.append(f"D100: Missing docstring for '{node.name}'")
        except SyntaxError:
            pass
        
        return len(issues) == 0, issues
    
    def run_all(self, code: str, filename: str = "code.py") -> dict:
        """Run complete quality pipeline"""
        checks = [
            ("Style (PEP 8)", self.run_style_check),
            ("Type annotations", self.run_type_check),
            ("Security", self.run_security_check),
            ("Docstrings", self.run_docstring_check),
        ]
        
        all_passed = True
        results = {}
        
        for name, check_func in checks:
            passed, issues = check_func(code)
            results[name] = {'passed': passed, 'issues': issues}
            if not passed:
                all_passed = False
        
        return {
            'all_passed': all_passed,
            'checks': results,
            'total_issues': sum(len(r['issues']) for r in results.values())
        }

# Test quality pipeline
print("\nEx 8 - Quality Pipeline:")

good_code = '''
import os
from typing import List


def calculate_sum(numbers: List[int]) -> int:
    """Calculate sum of a list of numbers.
    
    Args:
        numbers: List of integers to sum.
    
    Returns:
        Sum of all numbers.
    """
    return sum(numbers)
'''

bad_code = '''
import os
def calculate(x,y,z):
    result = eval("x+y+z")
    return result
'''

pipeline = QualityPipeline()

print("\nGood code check:")
results = pipeline.run_all(good_code)
print(f"  All passed: {results['all_passed']}")
print(f"  Total issues: {results['total_issues']}")

print("\nBad code check:")
results = pipeline.run_all(bad_code)
print(f"  All passed: {results['all_passed']}")
print(f"  Total issues: {results['total_issues']}")
for check, result in results['checks'].items():
    if result['issues']:
        print(f"  {check}:")
        for issue in result['issues']:
            print(f"    ✗ {issue}")
```

---

## สรุป

| Tool | Purpose | Config File |
|------|---------|-------------|
| Black | Auto-format code | pyproject.toml |
| isort | Sort imports | pyproject.toml |
| flake8 | Style checking | .flake8 |
| pylint | Comprehensive linting | .pylintrc |
| ruff | Fast all-in-one | pyproject.toml |
| mypy | Type checking | mypy.ini |
| pyright | Type checking (VSCode) | pyrightconfig.json |
| Sphinx | API documentation | docs/conf.py |
| MkDocs | Site documentation | mkdocs.yml |
| pre-commit | Automated checks | .pre-commit-config.yaml |

**Best Practices Summary:**
1. เริ่มต้นด้วย Black + isort + ruff สำหรับ formatting และ linting
2. เพิ่ม type hints ทุก public function
3. เขียน docstrings แบบ Google style อย่างน้อยสำหรับ public APIs
4. ตั้งค่า pre-commit hooks เพื่อ enforce standards อัตโนมัติ
5. ใช้ conventional commits สำหรับ meaningful changelog
6. Follow semantic versioning อย่างเคร่งครัด
7. Track technical debt ด้วย tickets ไม่ใช่แค่ TODO comments

---

*Part 99 - Code Quality, Standards & Documentation | Python Course*
