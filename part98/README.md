# Part 98: Advanced Testing Strategies & TDD

## สารบัญ

1. [Test Pyramid](#test-pyramid)
2. [TDD Methodology](#tdd)
3. [BDD with behave](#bdd)
4. [Property-based Testing (Hypothesis)](#hypothesis)
5. [Contract Testing (Pact)](#pact)
6. [Load Testing (Locust)](#locust)
7. [Performance Testing](#performance-testing)
8. [Mutation Testing (mutmut)](#mutation)
9. [Test Coverage Strategy](#coverage)
10. [Integration Testing Patterns](#integration)
11. [End-to-End Testing (Playwright)](#playwright)
12. [API Testing (httpx, respx)](#api-testing)
13. [Test Doubles](#test-doubles)
14. [Testing Async Code](#async-testing)
15. [แบบฝึกหัด](#exercises)

---

## 1. Test Pyramid <a name="test-pyramid"></a>

```
                    ┌─────────┐
                   /  E2E/UI  \      (Slow, expensive, few)
                  /   Tests    \
                 /─────────────\
                / Integration   \    (Medium)
               /    Tests       \
              /─────────────────\
             /    Unit Tests     \   (Fast, cheap, many)
            /───────────────────\
```

| Layer | Speed | Cost | Count | Scope |
|-------|-------|------|-------|-------|
| Unit | Fast (<1ms) | Cheap | Many (70%) | Single function |
| Integration | Medium (10-100ms) | Medium | Some (20%) | Multiple components |
| E2E | Slow (1-10s) | Expensive | Few (10%) | Full user flow |

```python
# ตัวอย่าง 1: Unit test ที่ดี
import pytest
from decimal import Decimal

class ShoppingCart:
    """Shopping cart implementation"""
    
    def __init__(self):
        self._items = {}
    
    def add_item(self, product_id: str, quantity: int, price: Decimal):
        if quantity <= 0:
            raise ValueError(f"Quantity must be positive, got {quantity}")
        if price < 0:
            raise ValueError(f"Price cannot be negative, got {price}")
        
        if product_id in self._items:
            self._items[product_id]['quantity'] += quantity
        else:
            self._items[product_id] = {'quantity': quantity, 'price': price}
    
    def remove_item(self, product_id: str):
        if product_id not in self._items:
            raise KeyError(f"Product {product_id} not in cart")
        del self._items[product_id]
    
    def get_total(self) -> Decimal:
        return sum(
            item['quantity'] * item['price']
            for item in self._items.values()
        )
    
    def apply_discount(self, percentage: int) -> Decimal:
        if not 0 <= percentage <= 100:
            raise ValueError(f"Discount must be 0-100%, got {percentage}")
        total = self.get_total()
        discount = total * Decimal(percentage) / 100
        return total - discount
    
    def __len__(self):
        return len(self._items)
    
    def is_empty(self) -> bool:
        return len(self._items) == 0


class TestShoppingCart:
    """Unit tests สำหรับ ShoppingCart"""
    
    def setup_method(self):
        """Run before each test"""
        self.cart = ShoppingCart()
    
    def test_new_cart_is_empty(self):
        assert self.cart.is_empty()
        assert len(self.cart) == 0
        assert self.cart.get_total() == Decimal('0')
    
    def test_add_single_item(self):
        self.cart.add_item("P001", 2, Decimal("19.99"))
        
        assert not self.cart.is_empty()
        assert len(self.cart) == 1
        assert self.cart.get_total() == Decimal("39.98")
    
    def test_add_multiple_quantities_of_same_item(self):
        self.cart.add_item("P001", 2, Decimal("10.00"))
        self.cart.add_item("P001", 3, Decimal("10.00"))
        
        assert len(self.cart) == 1  # ยังเป็น 1 product
        assert self.cart.get_total() == Decimal("50.00")
    
    def test_add_item_with_zero_quantity_raises(self):
        with pytest.raises(ValueError, match="Quantity must be positive"):
            self.cart.add_item("P001", 0, Decimal("10.00"))
    
    def test_add_item_with_negative_quantity_raises(self):
        with pytest.raises(ValueError):
            self.cart.add_item("P001", -1, Decimal("10.00"))
    
    def test_add_item_with_negative_price_raises(self):
        with pytest.raises(ValueError, match="Price cannot be negative"):
            self.cart.add_item("P001", 1, Decimal("-1.00"))
    
    def test_remove_item(self):
        self.cart.add_item("P001", 1, Decimal("10.00"))
        self.cart.remove_item("P001")
        
        assert self.cart.is_empty()
    
    def test_remove_nonexistent_item_raises(self):
        with pytest.raises(KeyError):
            self.cart.remove_item("NONEXISTENT")
    
    def test_apply_discount(self):
        self.cart.add_item("P001", 1, Decimal("100.00"))
        
        discounted = self.cart.apply_discount(20)
        assert discounted == Decimal("80.00")
    
    def test_apply_zero_discount(self):
        self.cart.add_item("P001", 1, Decimal("100.00"))
        assert self.cart.apply_discount(0) == Decimal("100.00")
    
    def test_apply_100_percent_discount(self):
        self.cart.add_item("P001", 1, Decimal("100.00"))
        assert self.cart.apply_discount(100) == Decimal("0.00")
    
    def test_invalid_discount_raises(self):
        with pytest.raises(ValueError):
            self.cart.apply_discount(101)

# Run tests
import pytest
# pytest.main([__file__, "-v"])  # uncomment to run

# Manual test run for demo
def run_tests():
    cart_tests = TestShoppingCart()
    
    test_methods = [m for m in dir(cart_tests) if m.startswith('test_')]
    passed = 0
    failed = 0
    
    for method_name in test_methods:
        cart_tests.setup_method()
        try:
            getattr(cart_tests, method_name)()
            print(f"  ✓ {method_name}")
            passed += 1
        except AssertionError as e:
            print(f"  ✗ {method_name}: {e}")
            failed += 1
        except Exception as e:
            print(f"  ✗ {method_name}: {type(e).__name__}: {e}")
            failed += 1
    
    print(f"\n{passed} passed, {failed} failed")

print("Unit Tests:")
run_tests()
```

---

## 2. TDD Methodology <a name="tdd"></a>

### Red-Green-Refactor Cycle

```
┌─────────┐     ┌─────────┐     ┌──────────┐
│   RED   │────▶│  GREEN  │────▶│ REFACTOR │
│ Write   │     │ Write   │     │ Clean up │
│ failing │     │ minimal │     │ code     │
│ test    │     │ code    │     │          │
└─────────┘     └─────────┘     └──────────┘
     ▲                                │
     └────────────────────────────────┘
```

```python
# ตัวอย่าง 2: TDD - สร้าง Password Validator

# STEP 1: RED - เขียน test ก่อน (ยังไม่มี implementation)
class TestPasswordValidator:
    
    def test_valid_password_passes(self):
        validator = PasswordValidator()
        result = validator.validate("SecurePass123!")
        assert result.is_valid
        assert not result.errors
    
    def test_too_short_password_fails(self):
        validator = PasswordValidator()
        result = validator.validate("Abc1!")
        assert not result.is_valid
        assert "at least 8 characters" in result.errors[0].lower()
    
    def test_no_uppercase_fails(self):
        validator = PasswordValidator()
        result = validator.validate("lowercase123!")
        assert not result.is_valid
        assert any("uppercase" in e.lower() for e in result.errors)
    
    def test_no_lowercase_fails(self):
        validator = PasswordValidator()
        result = validator.validate("UPPERCASE123!")
        assert not result.is_valid
    
    def test_no_digit_fails(self):
        validator = PasswordValidator()
        result = validator.validate("NoDigitsHere!")
        assert not result.is_valid
    
    def test_no_special_char_fails(self):
        validator = PasswordValidator()
        result = validator.validate("NoSpecial123")
        assert not result.is_valid
    
    def test_multiple_errors_reported(self):
        validator = PasswordValidator()
        result = validator.validate("weak")
        assert not result.is_valid
        assert len(result.errors) >= 2  # short + no uppercase + no digit + no special
    
    def test_common_password_rejected(self):
        validator = PasswordValidator(check_common=True)
        result = validator.validate("Password123!")
        assert not result.is_valid
        assert any("common" in e.lower() for e in result.errors)

# STEP 2: GREEN - เขียน implementation ที่ minimal ที่สุดที่ทำให้ tests ผ่าน
from dataclasses import dataclass, field
from typing import List
import re

@dataclass
class ValidationResult:
    is_valid: bool
    errors: List[str] = field(default_factory=list)

COMMON_PASSWORDS = {
    "Password123!", "Admin123!", "Welcome123!", 
    "Qwerty123!", "P@ssw0rd"
}

class PasswordValidator:
    """Password validator - TDD implementation"""
    
    def __init__(self, 
                 min_length: int = 8,
                 require_uppercase: bool = True,
                 require_lowercase: bool = True,
                 require_digit: bool = True,
                 require_special: bool = True,
                 check_common: bool = False):
        self.min_length = min_length
        self.require_uppercase = require_uppercase
        self.require_lowercase = require_lowercase
        self.require_digit = require_digit
        self.require_special = require_special
        self.check_common = check_common
    
    def validate(self, password: str) -> ValidationResult:
        errors = []
        
        if len(password) < self.min_length:
            errors.append(f"Password must be at least {self.min_length} characters long")
        
        if self.require_uppercase and not re.search(r'[A-Z]', password):
            errors.append("Password must contain at least one uppercase letter")
        
        if self.require_lowercase and not re.search(r'[a-z]', password):
            errors.append("Password must contain at least one lowercase letter")
        
        if self.require_digit and not re.search(r'\d', password):
            errors.append("Password must contain at least one digit")
        
        if self.require_special and not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            errors.append("Password must contain at least one special character")
        
        if self.check_common and password in COMMON_PASSWORDS:
            errors.append("Password is too common, please choose a different one")
        
        return ValidationResult(is_valid=len(errors) == 0, errors=errors)

# STEP 3: REFACTOR - ทำให้โค้ดดีขึ้น (tests ยังต้องผ่านทั้งหมด)
# (Implementation above is already clean)

# Run TDD tests
def run_tdd_tests():
    tests = TestPasswordValidator()
    methods = [m for m in dir(tests) if m.startswith('test_')]
    passed = failed = 0
    
    for method_name in methods:
        try:
            getattr(tests, method_name)()
            print(f"  ✓ {method_name}")
            passed += 1
        except Exception as e:
            print(f"  ✗ {method_name}: {e}")
            failed += 1
    
    print(f"\n{passed} passed, {failed} failed")

print("\nTDD - Password Validator Tests:")
run_tdd_tests()
```

```python
# ตัวอย่าง 3: TDD สำหรับ Business Logic
# สร้าง Discount Engine ด้วย TDD

# Tests ก่อน
class TestDiscountEngine:
    
    def setup_method(self):
        self.engine = DiscountEngine()
    
    def test_no_discount_for_new_customer(self):
        discount = self.engine.calculate(
            customer_type="new",
            order_total=100.0,
            items_count=1
        )
        assert discount == 0.0
    
    def test_5_percent_for_regular_customer(self):
        discount = self.engine.calculate(
            customer_type="regular",
            order_total=100.0,
            items_count=1
        )
        assert discount == 5.0
    
    def test_10_percent_for_premium_customer(self):
        discount = self.engine.calculate(
            customer_type="premium",
            order_total=100.0,
            items_count=1
        )
        assert discount == 10.0
    
    def test_extra_5_percent_for_large_orders(self):
        discount = self.engine.calculate(
            customer_type="regular",
            order_total=500.0,
            items_count=1
        )
        assert discount == 50.0  # 5% + 5% = 10%
    
    def test_bulk_discount_for_5_plus_items(self):
        discount = self.engine.calculate(
            customer_type="new",
            order_total=100.0,
            items_count=5
        )
        assert discount == 3.0  # 3% bulk discount
    
    def test_max_discount_cap(self):
        discount = self.engine.calculate(
            customer_type="premium",
            order_total=1000.0,
            items_count=10
        )
        # 10% + 5% + 3% = 18%, but cap at 15%
        assert discount == 150.0  # 15% of 1000


class DiscountEngine:
    """Discount calculation engine"""
    
    CUSTOMER_DISCOUNTS = {
        "new": 0,
        "regular": 5,
        "premium": 10
    }
    
    LARGE_ORDER_THRESHOLD = 400.0
    LARGE_ORDER_EXTRA = 5
    
    BULK_THRESHOLD = 5
    BULK_DISCOUNT = 3
    
    MAX_DISCOUNT = 15
    
    def calculate(self, customer_type: str, order_total: float, 
                  items_count: int) -> float:
        total_pct = 0
        
        # Customer type discount
        total_pct += self.CUSTOMER_DISCOUNTS.get(customer_type, 0)
        
        # Large order discount
        if order_total >= self.LARGE_ORDER_THRESHOLD:
            total_pct += self.LARGE_ORDER_EXTRA
        
        # Bulk discount
        if items_count >= self.BULK_THRESHOLD:
            total_pct += self.BULK_DISCOUNT
        
        # Apply cap
        total_pct = min(total_pct, self.MAX_DISCOUNT)
        
        return round(order_total * total_pct / 100, 2)

print("\nTDD - Discount Engine Tests:")
def run_discount_tests():
    tests = TestDiscountEngine()
    methods = [m for m in dir(tests) if m.startswith('test_')]
    passed = failed = 0
    
    for method in methods:
        tests.setup_method()
        try:
            getattr(tests, method)()
            print(f"  ✓ {method}")
            passed += 1
        except Exception as e:
            print(f"  ✗ {method}: {e}")
            failed += 1
    
    print(f"\n{passed} passed, {failed} failed")

run_discount_tests()
```

---

## 3. BDD - Behavior-Driven Development <a name="bdd"></a>

```python
# ตัวอย่าง 4: BDD style tests
# pip install behave

# Feature file: features/shopping.feature
FEATURE_FILE = """
Feature: Shopping Cart
  As a customer
  I want to add products to my cart
  So that I can purchase them

  Scenario: Add a single product to empty cart
    Given an empty shopping cart
    When I add 2 units of "Product A" at $10.00 each
    Then the cart should contain 1 product
    And the total should be $20.00

  Scenario: Apply discount to cart
    Given a shopping cart with:
      | product | quantity | price |
      | Widget  | 3        | 15.00 |
    When I apply a 10% discount
    Then the discounted total should be $40.50

  Scenario Outline: Validate discount percentage
    Given an empty shopping cart
    When I add 1 unit of "Test" at $100.00 each
    Then applying <discount>% discount should <result>

    Examples:
      | discount | result    |
      | 0        | succeed   |
      | 50       | succeed   |
      | 100      | succeed   |
      | 101      | fail      |
      | -1       | fail      |
"""

# Step definitions: features/steps/shopping_steps.py
STEP_DEFS = '''
from behave import given, when, then
from decimal import Decimal
from your_module import ShoppingCart

@given("an empty shopping cart")
def step_empty_cart(context):
    context.cart = ShoppingCart()

@when('I add {quantity:d} units of "{product}" at ${price:f} each')
def step_add_item(context, quantity, product, price):
    context.cart.add_item(product, quantity, Decimal(str(price)))

@then("the cart should contain {count:d} product")
def step_cart_count(context, count):
    assert len(context.cart) == count

@then("the total should be ${expected:f}")
def step_total(context, expected):
    assert context.cart.get_total() == Decimal(str(expected))
'''

print("BDD Feature File:")
print(FEATURE_FILE)
print("\nBDD Step Definitions:")
print(STEP_DEFS)
```

```python
# ตัวอย่าง 5: BDD ใน Pure Python (ไม่ต้องใช้ behave)
from typing import Callable, List
from dataclasses import dataclass

class BDDTest:
    """BDD-style test framework เบื้องต้น"""
    
    def __init__(self, feature_name: str):
        self.feature_name = feature_name
        self._scenarios = []
    
    def scenario(self, name: str):
        """Decorator สำหรับ scenario"""
        def decorator(func):
            self._scenarios.append((name, func))
            return func
        return decorator
    
    def run(self):
        print(f"\nFeature: {self.feature_name}")
        passed = failed = 0
        
        for name, func in self._scenarios:
            try:
                context = {}
                func(context)
                print(f"  ✓ Scenario: {name}")
                passed += 1
            except AssertionError as e:
                print(f"  ✗ Scenario: {name} - {e}")
                failed += 1
        
        print(f"\n{passed} passed, {failed} failed")

# Given/When/Then helpers
def given(description: str):
    print(f"    Given {description}", end="")
    
def when(description: str):
    print(f"    When {description}", end="")

def then(description: str, assertion: bool, message: str = ""):
    if assertion:
        print(f"    Then {description} ✓")
    else:
        raise AssertionError(f"Then {description} FAILED: {message}")

from decimal import Decimal

bdd = BDDTest("Shopping Cart")

@bdd.scenario("Add item to empty cart")
def test_add_item(ctx):
    given("an empty shopping cart\n")
    ctx['cart'] = ShoppingCart()
    
    when("I add 2 units of Product A at $10.00\n")
    ctx['cart'].add_item("A", 2, Decimal("10.00"))
    
    then("cart has 1 product", len(ctx['cart']) == 1)
    then("total is $20.00", ctx['cart'].get_total() == Decimal("20.00"))

@bdd.scenario("Apply 20% discount")
def test_discount(ctx):
    given("a cart with $100 item\n")
    ctx['cart'] = ShoppingCart()
    ctx['cart'].add_item("X", 1, Decimal("100.00"))
    
    when("I apply 20% discount\n")
    discounted = ctx['cart'].apply_discount(20)
    
    then("discounted total is $80.00", discounted == Decimal("80.00"))

@bdd.scenario("Remove item from cart")
def test_remove(ctx):
    given("a cart with items\n")
    ctx['cart'] = ShoppingCart()
    ctx['cart'].add_item("A", 1, Decimal("50.00"))
    ctx['cart'].add_item("B", 2, Decimal("25.00"))
    
    when("I remove item A\n")
    ctx['cart'].remove_item("A")
    
    then("cart has 1 product", len(ctx['cart']) == 1)
    then("total is $50.00", ctx['cart'].get_total() == Decimal("50.00"))

bdd.run()
```

---

## 4. Property-based Testing (Hypothesis) <a name="hypothesis"></a>

```python
# ตัวอย่าง 6: Hypothesis property-based testing
# pip install hypothesis

try:
    from hypothesis import given, settings, assume, example
    from hypothesis import strategies as st
    import pytest
    
    # ทดสอบด้วย properties แทนที่จะเป็น example-based
    
    class TestPasswordValidatorProperties:
        
        @given(st.text(min_size=8, max_size=100, 
                       alphabet=st.characters(
                           whitelist_categories=['Lu', 'Ll', 'Nd'],
                           whitelist_characters='!@#$%'
                       )))
        def test_valid_password_structure(self, password):
            """ทุก password ที่มีทุก character types ควรผ่าน"""
            # เพิ่ม uppercase, lowercase, digit, special อย่างน้อย 1 ตัว
            if (any(c.isupper() for c in password) and
                any(c.islower() for c in password) and
                any(c.isdigit() for c in password) and
                any(c in '!@#$%' for c in password)):
                
                validator = PasswordValidator()
                result = validator.validate(password)
                assert result.is_valid, f"Expected valid but got errors: {result.errors}"
        
        @given(st.integers(min_value=0, max_value=100))
        def test_discount_always_non_negative(self, percentage):
            cart = ShoppingCart()
            cart.add_item("X", 1, Decimal("100.00"))
            discounted = cart.apply_discount(percentage)
            assert discounted >= Decimal("0")
        
        @given(st.integers(min_value=0, max_value=100))
        def test_discount_never_exceeds_total(self, percentage):
            cart = ShoppingCart()
            cart.add_item("X", 1, Decimal("100.00"))
            total = cart.get_total()
            discounted = cart.apply_discount(percentage)
            assert discounted <= total
        
        @given(
            st.lists(
                st.tuples(
                    st.text(min_size=1, max_size=10),
                    st.integers(min_value=1, max_value=100),
                    st.decimals(min_value='0.01', max_value='1000', places=2)
                ),
                min_size=1,
                max_size=20,
                unique_by=lambda x: x[0]  # unique product IDs
            )
        )
        def test_total_equals_sum_of_items(self, items):
            cart = ShoppingCart()
            expected_total = Decimal("0")
            
            for product_id, quantity, price in items:
                cart.add_item(product_id, quantity, price)
                expected_total += quantity * price
            
            assert cart.get_total() == expected_total
    
    print("\nProperty-based Tests (Hypothesis):")
    prop_tests = TestPasswordValidatorProperties()
    
    # Test discount properties
    for pct in range(0, 101, 10):
        try:
            prop_tests.test_discount_always_non_negative(pct)
            prop_tests.test_discount_never_exceeds_total(pct)
        except Exception as e:
            print(f"  FAIL at {pct}%: {e}")
    
    print("  ✓ Discount properties hold for all percentages 0-100%")
    
    # Test total calculation
    test_items = [
        ("A", 2, Decimal("10.50")),
        ("B", 1, Decimal("25.00")),
        ("C", 3, Decimal("5.99")),
    ]
    prop_tests.test_total_equals_sum_of_items(test_items)
    print("  ✓ Total calculation property holds")

except ImportError:
    print("Hypothesis not installed. Run: pip install hypothesis")

# Custom property tests without Hypothesis
print("\nManual Property Tests:")

import random
import decimal

def test_sort_properties():
    """Test sorting properties that should always hold"""
    for _ in range(100):
        n = random.randint(0, 100)
        data = [random.randint(-1000, 1000) for _ in range(n)]
        
        sorted_data = sorted(data)
        
        # Property 1: Length preserved
        assert len(sorted_data) == len(data), "Length changed!"
        
        # Property 2: All elements present
        assert sorted(sorted_data) == sorted(data), "Elements changed!"
        
        # Property 3: Actually sorted
        for i in range(len(sorted_data) - 1):
            assert sorted_data[i] <= sorted_data[i+1], "Not sorted!"
    
    print("  ✓ Sort properties: length, elements, ordering")

def test_addition_properties():
    """Test mathematical properties of addition"""
    for _ in range(1000):
        a = random.randint(-10000, 10000)
        b = random.randint(-10000, 10000)
        c = random.randint(-10000, 10000)
        
        # Commutative: a + b == b + a
        assert a + b == b + a, f"Commutative failed: {a} + {b}"
        
        # Associative: (a + b) + c == a + (b + c)
        assert (a + b) + c == a + (b + c), f"Associative failed"
        
        # Identity: a + 0 == a
        assert a + 0 == a, f"Identity failed: {a}"
    
    print("  ✓ Addition properties: commutative, associative, identity")

def test_cart_total_properties():
    """Test cart total calculation properties"""
    for _ in range(100):
        cart = ShoppingCart()
        n_items = random.randint(1, 10)
        
        items = []
        for i in range(n_items):
            qty = random.randint(1, 10)
            price = Decimal(str(round(random.uniform(1.0, 100.0), 2)))
            product_id = f"P{i:03d}"
            cart.add_item(product_id, qty, price)
            items.append((qty, price))
        
        # Property: Total == sum of (qty * price)
        expected = sum(q * p for q, p in items)
        assert cart.get_total() == expected, "Total mismatch!"
    
    print("  ✓ Cart total property: total == sum(qty * price)")

test_sort_properties()
test_addition_properties()
test_cart_total_properties()
```

---

## 5. Contract Testing <a name="pact"></a>

```python
# ตัวอย่าง 7: Contract testing concepts
# pip install pact-python

CONTRACT_TESTING_EXPLANATION = """
Contract Testing ทำงานอย่างไร:

1. Consumer defines contract (what it expects from provider)
2. Contract saved as pact file (JSON)
3. Provider verifies it can fulfill contract

ตัวอย่าง Contract:
- Consumer: Frontend App
- Provider: User API

Consumer expects:
GET /users/123
Response: {
  "id": 123,
  "name": "string",
  "email": "string"
}
"""

print(CONTRACT_TESTING_EXPLANATION)

# Implement simplified contract testing
import json
from typing import Any, Dict, Optional, Callable
from dataclasses import dataclass

@dataclass
class Interaction:
    description: str
    request: Dict
    response: Dict

class MockProvider:
    """Mock provider สำหรับ consumer-side testing"""
    
    def __init__(self):
        self._interactions: Dict[str, Interaction] = {}
        self._recorded_requests = []
    
    def given(self, state: str) -> 'MockProvider':
        self._current_state = state
        return self
    
    def upon_receiving(self, description: str) -> 'MockProvider':
        self._current_description = description
        return self
    
    def with_request(self, method: str, path: str, 
                     headers: dict = None, body: any = None) -> 'MockProvider':
        self._current_request = {
            'method': method,
            'path': path,
            'headers': headers or {},
            'body': body
        }
        return self
    
    def will_respond_with(self, status: int, 
                          headers: dict = None,
                          body: any = None) -> 'MockProvider':
        key = f"{self._current_request['method']}:{self._current_request['path']}"
        
        interaction = Interaction(
            description=self._current_description,
            request=self._current_request,
            response={
                'status': status,
                'headers': headers or {},
                'body': body
            }
        )
        
        self._interactions[key] = interaction
        return self
    
    def simulate_request(self, method: str, path: str, **kwargs) -> dict:
        key = f"{method}:{path}"
        if key not in self._interactions:
            raise ValueError(f"No interaction defined for {key}")
        
        interaction = self._interactions[key]
        self._recorded_requests.append({'method': method, 'path': path})
        
        return {
            'status': interaction.response['status'],
            'body': interaction.response['body']
        }
    
    def generate_pact(self, consumer: str, provider: str) -> dict:
        return {
            'consumer': {'name': consumer},
            'provider': {'name': provider},
            'interactions': [
                {
                    'description': i.description,
                    'request': i.request,
                    'response': i.response
                }
                for i in self._interactions.values()
            ]
        }

# Consumer-side contract test
class UserServiceClient:
    """Client สำหรับ User Service"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
    
    def get_user(self, user_id: int) -> dict:
        # In real code, this uses requests library
        # Here we'll use mock
        pass
    
    def create_user(self, name: str, email: str) -> dict:
        pass

# Contract test
mock = MockProvider()

# Define expected interaction
mock.given("user 1 exists") \
    .upon_receiving("a request for user 1") \
    .with_request("GET", "/users/1") \
    .will_respond_with(
        status=200,
        body={"id": 1, "name": "John Doe", "email": "john@example.com"}
    )

mock.given("no users exist") \
    .upon_receiving("a request for nonexistent user") \
    .with_request("GET", "/users/999") \
    .will_respond_with(
        status=404,
        body={"error": "User not found"}
    )

# Consumer test using mock provider
def test_get_existing_user():
    response = mock.simulate_request("GET", "/users/1")
    
    assert response['status'] == 200
    user = response['body']
    assert 'id' in user
    assert 'name' in user
    assert 'email' in user
    print("  ✓ get existing user - contract satisfied")

def test_get_nonexistent_user():
    response = mock.simulate_request("GET", "/users/999")
    
    assert response['status'] == 404
    assert 'error' in response['body']
    print("  ✓ get nonexistent user - contract satisfied")

print("\nContract Testing Demo:")
test_get_existing_user()
test_get_nonexistent_user()

# Generate pact file
pact = mock.generate_pact("frontend-app", "user-service")
print(f"\nGenerated Pact:")
print(json.dumps(pact, indent=2))
```

---

## 6. Load Testing (Locust) <a name="locust"></a>

```python
# ตัวอย่าง 8: Locust load testing
# pip install locust

LOCUST_CODE = '''
# locustfile.py
from locust import HttpUser, task, between, events
from locust.exception import RescheduleTask
import random
import json

class WebsiteUser(HttpUser):
    """Simulates a typical website user"""
    
    # ระหว่าง requests ให้รอ 1-3 วินาที
    wait_time = between(1, 3)
    
    def on_start(self):
        """Setup - รันเมื่อ user เริ่มต้น"""
        self.login()
    
    def login(self):
        """Login and store auth token"""
        response = self.client.post("/auth/login", json={
            "email": f"user{random.randint(1, 1000)}@example.com",
            "password": "password123"
        })
        
        if response.status_code == 200:
            self.token = response.json().get("token")
            self.client.headers["Authorization"] = f"Bearer {self.token}"
        else:
            raise RescheduleTask()
    
    @task(10)  # weight 10 = called 10x more often
    def browse_products(self):
        """Browse product listing"""
        page = random.randint(1, 10)
        with self.client.get(
            f"/api/products?page={page}&size=20",
            name="/api/products",  # group similar URLs
            catch_response=True
        ) as response:
            if response.status_code != 200:
                response.failure(f"Got {response.status_code}")
            elif not response.json().get("items"):
                response.failure("Empty product list")
    
    @task(5)
    def view_product(self):
        """View single product"""
        product_id = random.randint(1, 1000)
        with self.client.get(
            f"/api/products/{product_id}",
            name="/api/products/[id]",
            catch_response=True
        ) as response:
            if response.status_code == 404:
                response.success()  # 404 is valid for missing products
            elif response.status_code != 200:
                response.failure(f"Got {response.status_code}")
    
    @task(3)
    def search_products(self):
        """Search for products"""
        queries = ["laptop", "phone", "tablet", "camera", "headphone"]
        query = random.choice(queries)
        
        self.client.get(
            f"/api/search?q={query}",
            name="/api/search"
        )
    
    @task(2)
    def add_to_cart(self):
        """Add item to cart"""
        self.client.post("/api/cart/items", json={
            "product_id": random.randint(1, 1000),
            "quantity": random.randint(1, 3)
        })
    
    @task(1)
    def checkout(self):
        """Complete checkout"""
        # View cart
        self.client.get("/api/cart")
        
        # Place order
        response = self.client.post("/api/orders", json={
            "payment_method": "credit_card",
            "address_id": 1
        })
        
        if response.status_code == 201:
            order_id = response.json().get("order_id")
            # Confirm order
            self.client.get(f"/api/orders/{order_id}")


class AdminUser(HttpUser):
    """Admin user with different behavior"""
    wait_time = between(2, 5)
    weight = 1  # 1 admin per 10 regular users
    
    @task
    def check_dashboard(self):
        self.client.get("/admin/dashboard")
    
    @task
    def export_reports(self):
        self.client.post("/admin/reports/export", json={
            "format": "csv",
            "date_range": "last_7_days"
        })


# Custom metrics
@events.request.add_listener
def on_request(request_type, name, response_time, response_length, 
               response, context, **kwargs):
    """Custom request listener"""
    if response_time > 1000:  # > 1 second
        print(f"SLOW REQUEST: {name} took {response_time:.0f}ms")

# Run: locust -f locustfile.py --host http://localhost:8000 --headless
#      --users 100 --spawn-rate 10 --run-time 60s
'''

print("Locust Load Testing:")
print(LOCUST_CODE)
```

```python
# ตัวอย่าง 9: Simple load test ที่รันได้จริง
import asyncio
import time
import random
import statistics
from typing import List, Tuple
from dataclasses import dataclass, field

@dataclass
class LoadTestResult:
    total_requests: int = 0
    successful_requests: int = 0
    failed_requests: int = 0
    durations: List[float] = field(default_factory=list)
    errors: List[str] = field(default_factory=list)
    
    @property
    def success_rate(self) -> float:
        if self.total_requests == 0:
            return 0
        return self.successful_requests / self.total_requests * 100
    
    @property
    def avg_duration_ms(self) -> float:
        return statistics.mean(self.durations) * 1000 if self.durations else 0
    
    @property
    def p99_duration_ms(self) -> float:
        if not self.durations:
            return 0
        sorted_d = sorted(self.durations)
        return sorted_d[int(len(sorted_d) * 0.99)] * 1000
    
    @property
    def throughput_rps(self) -> float:
        if not self.durations:
            return 0
        total_time = sum(self.durations)
        return len(self.durations) / total_time if total_time > 0 else 0

async def simulated_api_call(endpoint: str) -> Tuple[bool, float]:
    """Simulate an API call"""
    # Simulate variable latency
    if endpoint == "/api/products":
        delay = random.gauss(0.05, 0.01)  # mean 50ms
    elif endpoint == "/api/search":
        delay = random.gauss(0.1, 0.02)   # mean 100ms
    else:
        delay = random.gauss(0.03, 0.005) # mean 30ms
    
    delay = max(0.001, delay)
    
    await asyncio.sleep(delay)
    
    # Simulate 2% error rate
    success = random.random() > 0.02
    return success, delay

async def run_load_test(
    endpoints: List[str],
    concurrent_users: int,
    duration_seconds: float
) -> LoadTestResult:
    """Run load test"""
    result = LoadTestResult()
    end_time = time.time() + duration_seconds
    
    async def user_loop():
        while time.time() < end_time:
            endpoint = random.choice(endpoints)
            success, duration = await simulated_api_call(endpoint)
            
            result.total_requests += 1
            result.durations.append(duration)
            
            if success:
                result.successful_requests += 1
            else:
                result.failed_requests += 1
                result.errors.append(f"Error on {endpoint}")
            
            # Small think time between requests
            await asyncio.sleep(random.uniform(0.01, 0.05))
    
    # Run concurrent users
    tasks = [user_loop() for _ in range(concurrent_users)]
    await asyncio.gather(*tasks)
    
    return result

async def main():
    print("\nLoad Testing Demo:")
    
    endpoints = ["/api/products", "/api/search", "/api/users", "/api/cart"]
    
    # Low load
    print("\n  Low Load (10 users, 5 seconds):")
    result = await run_load_test(endpoints, concurrent_users=10, duration_seconds=5)
    print(f"    Requests: {result.total_requests}")
    print(f"    Success Rate: {result.success_rate:.1f}%")
    print(f"    Avg Latency: {result.avg_duration_ms:.1f}ms")
    print(f"    P99 Latency: {result.p99_duration_ms:.1f}ms")
    print(f"    Throughput: {result.throughput_rps:.1f} req/s")
    
    # High load
    print("\n  High Load (50 users, 5 seconds):")
    result = await run_load_test(endpoints, concurrent_users=50, duration_seconds=5)
    print(f"    Requests: {result.total_requests}")
    print(f"    Success Rate: {result.success_rate:.1f}%")
    print(f"    Avg Latency: {result.avg_duration_ms:.1f}ms")
    print(f"    P99 Latency: {result.p99_duration_ms:.1f}ms")
    print(f"    Throughput: {result.throughput_rps:.1f} req/s")

asyncio.run(main())
```

---

## 7. Performance Testing <a name="performance-testing"></a>

```python
# ตัวอย่าง 10: Performance benchmarks
import timeit
import statistics
import sys
import tracemalloc
from typing import Callable, Any

class PerformanceSuite:
    """Performance test suite"""
    
    def __init__(self, name: str):
        self.name = name
        self.results = []
    
    def benchmark(self, name: str, func: Callable, 
                  iterations: int = 1000,
                  memory: bool = True):
        """Run benchmark for a function"""
        
        # Time benchmark
        times = timeit.repeat(func, repeat=10, number=iterations)
        avg_time = statistics.mean(times) / iterations * 1000  # ms
        stdev = statistics.stdev(times) / iterations * 1000
        
        result = {
            'name': name,
            'avg_ms': round(avg_time, 4),
            'stdev_ms': round(stdev, 4),
            'min_ms': round(min(times) / iterations * 1000, 4),
            'max_ms': round(max(times) / iterations * 1000, 4),
        }
        
        # Memory benchmark
        if memory:
            tracemalloc.start()
            func()
            snapshot = tracemalloc.take_snapshot()
            tracemalloc.stop()
            
            top_stats = snapshot.statistics('lineno')
            total_memory = sum(stat.size for stat in top_stats)
            result['memory_kb'] = round(total_memory / 1024, 2)
        
        self.results.append(result)
        return result
    
    def performance_test(self, name: str, 
                         threshold_ms: float,
                         func: Callable,
                         iterations: int = 100):
        """Test that function runs within threshold"""
        result = self.benchmark(name, func, iterations, memory=False)
        
        passed = result['avg_ms'] <= threshold_ms
        status = "PASS" if passed else "FAIL"
        
        print(f"  [{status}] {name}")
        print(f"    Threshold: {threshold_ms}ms | Actual: {result['avg_ms']}ms")
        
        return passed
    
    def print_report(self):
        print(f"\nPerformance Report: {self.name}")
        print("=" * 60)
        
        for r in self.results:
            print(f"\n  {r['name']}")
            print(f"    Avg: {r['avg_ms']:.4f}ms ± {r['stdev_ms']:.4f}ms")
            print(f"    Min: {r['min_ms']:.4f}ms | Max: {r['max_ms']:.4f}ms")
            if 'memory_kb' in r:
                print(f"    Memory: {r['memory_kb']:.2f} KB")

# Run performance tests
suite = PerformanceSuite("Shopping Cart")

cart = ShoppingCart()
cart.add_item("P001", 10, Decimal("9.99"))

print("\nPerformance Tests:")
suite.performance_test(
    "add_item",
    threshold_ms=0.1,
    func=lambda: cart.add_item(f"P{random.randint(0,9999):04d}", 1, Decimal("10.00"))
)

suite.performance_test(
    "get_total",
    threshold_ms=0.01,
    func=cart.get_total
)

suite.performance_test(
    "apply_discount",
    threshold_ms=0.1,
    func=lambda: cart.apply_discount(10)
)
```

---

## 8. Mutation Testing <a name="mutation"></a>

```python
# ตัวอย่าง 11: Mutation testing concepts
# pip install mutmut

MUTMUT_GUIDE = """
Mutation Testing:
1. เปลี่ยน operators ใน code เล็กน้อย (mutations)
2. รัน tests
3. ถ้า tests ผ่าน = mutation survived (bad!) = tests ไม่ดีพอ
4. ถ้า tests fail = mutation killed (good!) = tests ตรวจจับได้

ตัวอย่าง mutations:
- a > b  →  a >= b  หรือ  a < b
- a + b  →  a - b  หรือ  a * b  
- True   →  False
- return x  →  return None
- if cond:  →  if not cond:

รัน mutmut:
$ mutmut run --paths-to-mutate=src/
$ mutmut results
$ mutmut show 1  # ดู mutation ที่ survived
"""
print(MUTMUT_GUIDE)

# Manual mutation testing demo
def count_mutations_survived(func, tests, mutations):
    """Demo mutation testing"""
    survived = []
    killed = []
    
    for mutation_name, mutated_func in mutations:
        all_tests_pass = True
        
        for test_name, test_func in tests:
            try:
                test_func(mutated_func)
            except (AssertionError, Exception):
                all_tests_pass = False
                break
        
        if all_tests_pass:
            survived.append(mutation_name)
        else:
            killed.append(mutation_name)
    
    return survived, killed

# Original function
def calculate_total(items):
    """Sum of price * quantity for all items"""
    total = 0
    for item in items:
        total += item['price'] * item['quantity']
    return total

# Tests
test_items = [
    {'price': 10.0, 'quantity': 2},
    {'price': 5.0, 'quantity': 3}
]

def test_basic(func):
    result = func(test_items)
    assert result == 35.0

def test_empty(func):
    result = func([])
    assert result == 0

def test_single(func):
    result = func([{'price': 100.0, 'quantity': 1}])
    assert result == 100.0

tests = [
    ("test_basic", test_basic),
    ("test_empty", test_empty),
    ("test_single", test_single),
]

# Mutations
def mutant_1(items):
    """Mutation: += to -="""
    total = 0
    for item in items:
        total -= item['price'] * item['quantity']  # BUG: -= instead of +=
    return total

def mutant_2(items):
    """Mutation: * to +"""
    total = 0
    for item in items:
        total += item['price'] + item['quantity']  # BUG: + instead of *
    return total

def mutant_3(items):
    """Mutation: return total to return 0"""
    total = 0
    for item in items:
        total += item['price'] * item['quantity']
    return 0  # BUG: always returns 0

mutations = [
    ("mutant_1 (total -= instead of +=)", mutant_1),
    ("mutant_2 (price + qty instead of *)", mutant_2),
    ("mutant_3 (always return 0)", mutant_3),
]

survived, killed = count_mutations_survived(calculate_total, tests, mutations)

print("\nMutation Testing Results:")
print(f"  Total mutations: {len(mutations)}")
print(f"  Killed (tests caught): {len(killed)}")
print(f"  Survived (tests missed): {len(survived)}")
print(f"  Mutation score: {len(killed)/len(mutations)*100:.0f}%")

if killed:
    print("\n  Killed mutations:")
    for m in killed:
        print(f"    ✓ {m}")

if survived:
    print("\n  Survived mutations (need better tests!):")
    for m in survived:
        print(f"    ✗ {m}")
```

---

## 9. Test Coverage Strategy <a name="coverage"></a>

```python
# ตัวอย่าง 12: coverage.py usage and strategies
# pip install coverage

COVERAGE_GUIDE = """
Coverage Types:
1. Line Coverage   - which lines were executed
2. Branch Coverage - which branches (if/else) were taken
3. Path Coverage   - which complete paths through code
4. MC/DC Coverage  - used in safety-critical systems

Run coverage:
$ coverage run -m pytest tests/
$ coverage report
$ coverage html  # generates htmlcov/index.html

.coveragerc configuration:
[run]
source = src/
branch = True
omit = 
    */tests/*
    */migrations/*
    setup.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    if __name__ == .__main__.:
    raise NotImplementedError
    pass
"""
print(COVERAGE_GUIDE)

# Coverage analysis demo
def analyze_coverage():
    """Demonstrate coverage concepts"""
    
    def function_to_test(x: int, y: int, mode: str = "add") -> int:
        """Function with multiple branches"""
        
        if mode == "add":          # branch 1
            result = x + y
        elif mode == "multiply":   # branch 2
            result = x * y
        elif mode == "max":        # branch 3
            result = x if x > y else y
        else:                      # branch 4
            raise ValueError(f"Unknown mode: {mode}")
        
        if result > 100:           # branch 5
            result = 100
        
        return result
    
    # Test cases that give different coverage levels
    
    # Minimal coverage (2/5 branches = 40%)
    minimal_tests = [
        lambda: function_to_test(1, 2, "add"),     # branch 1
        lambda: function_to_test(10, 20, "max"),   # branch 3
    ]
    
    # Better coverage (4/5 branches = 80%)
    better_tests = [
        lambda: function_to_test(1, 2, "add"),
        lambda: function_to_test(3, 4, "multiply"),
        lambda: function_to_test(10, 20, "max"),
        lambda: function_to_test(60, 50, "add"),   # branch 5 (result > 100)
    ]
    
    # Full coverage (5/5 branches = 100%)
    full_tests = [
        *[t for t in better_tests],
        lambda: (lambda: function_to_test(1, 2, "unknown"))()  # branch 4
    ]
    
    def count_branches_covered(tests):
        """Count which branches are covered"""
        branches_hit = set()
        
        for test in tests:
            try:
                test()
            except (ValueError, Exception):
                branches_hit.add('error_branch')
        
        return len(branches_hit)
    
    print("\nCoverage Strategy Demo:")
    print("  Minimal tests: likely 40-60% coverage")
    print("  Better tests:  likely 70-85% coverage")
    print("  Full tests:    ~100% branch coverage")
    
    # What 100% coverage doesn't guarantee
    print("\nImportant: 100% coverage ≠ bug-free!")
    print("  Missing: boundary conditions, negative tests,")
    print("          performance tests, security tests")

analyze_coverage()
```

---

## 10. Integration Testing Patterns <a name="integration"></a>

```python
# ตัวอย่าง 13: Integration tests with database
import sqlite3
import pytest
from contextlib import contextmanager

class UserRepository:
    """Repository pattern สำหรับ User data"""
    
    def __init__(self, db_connection):
        self.conn = db_connection
        self._create_tables()
    
    def _create_tables(self):
        self.conn.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        self.conn.commit()
    
    def create(self, name: str, email: str) -> dict:
        cursor = self.conn.execute(
            'INSERT INTO users (name, email) VALUES (?, ?)',
            (name, email)
        )
        self.conn.commit()
        return self.get_by_id(cursor.lastrowid)
    
    def get_by_id(self, user_id: int) -> dict:
        row = self.conn.execute(
            'SELECT * FROM users WHERE id = ?', (user_id,)
        ).fetchone()
        
        if not row:
            return None
        
        return dict(row)
    
    def get_by_email(self, email: str) -> dict:
        row = self.conn.execute(
            'SELECT * FROM users WHERE email = ?', (email,)
        ).fetchone()
        return dict(row) if row else None
    
    def update(self, user_id: int, name: str) -> dict:
        self.conn.execute(
            'UPDATE users SET name = ? WHERE id = ?',
            (name, user_id)
        )
        self.conn.commit()
        return self.get_by_id(user_id)
    
    def delete(self, user_id: int) -> bool:
        cursor = self.conn.execute(
            'DELETE FROM users WHERE id = ?', (user_id,)
        )
        self.conn.commit()
        return cursor.rowcount > 0
    
    def list_all(self) -> list:
        rows = self.conn.execute('SELECT * FROM users').fetchall()
        return [dict(row) for row in rows]


@contextmanager
def test_db():
    """Context manager สำหรับ test database"""
    conn = sqlite3.connect(':memory:')
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()


class TestUserRepository:
    """Integration tests สำหรับ UserRepository"""
    
    def setup_method(self):
        """Create fresh DB for each test"""
        self.conn = sqlite3.connect(':memory:')
        self.conn.row_factory = sqlite3.Row
        self.repo = UserRepository(self.conn)
    
    def teardown_method(self):
        self.conn.close()
    
    def test_create_user(self):
        user = self.repo.create("John Doe", "john@example.com")
        
        assert user['id'] is not None
        assert user['name'] == "John Doe"
        assert user['email'] == "john@example.com"
    
    def test_get_user_by_id(self):
        created = self.repo.create("Jane Doe", "jane@example.com")
        found = self.repo.get_by_id(created['id'])
        
        assert found['name'] == "Jane Doe"
        assert found['email'] == "jane@example.com"
    
    def test_get_nonexistent_user_returns_none(self):
        result = self.repo.get_by_id(9999)
        assert result is None
    
    def test_get_user_by_email(self):
        self.repo.create("Test User", "test@example.com")
        found = self.repo.get_by_email("test@example.com")
        
        assert found is not None
        assert found['name'] == "Test User"
    
    def test_duplicate_email_raises(self):
        self.repo.create("User 1", "same@example.com")
        
        import sqlite3
        with pytest.raises(sqlite3.IntegrityError):
            self.repo.create("User 2", "same@example.com")
    
    def test_update_user(self):
        user = self.repo.create("Old Name", "test@example.com")
        updated = self.repo.update(user['id'], "New Name")
        
        assert updated['name'] == "New Name"
        assert updated['email'] == "test@example.com"
    
    def test_delete_user(self):
        user = self.repo.create("To Delete", "delete@example.com")
        
        result = self.repo.delete(user['id'])
        assert result is True
        
        found = self.repo.get_by_id(user['id'])
        assert found is None
    
    def test_delete_nonexistent_user_returns_false(self):
        result = self.repo.delete(9999)
        assert result is False
    
    def test_list_all_users(self):
        self.repo.create("User A", "a@example.com")
        self.repo.create("User B", "b@example.com")
        self.repo.create("User C", "c@example.com")
        
        users = self.repo.list_all()
        assert len(users) == 3

# Run integration tests
print("\nIntegration Tests:")
import pytest as _pytest

def run_integration_tests():
    tests = TestUserRepository()
    methods = [m for m in dir(tests) if m.startswith('test_')]
    passed = failed = 0
    
    for method in methods:
        tests.setup_method()
        try:
            getattr(tests, method)()
            print(f"  ✓ {method}")
            passed += 1
        except Exception as e:
            print(f"  ✗ {method}: {e}")
            failed += 1
        finally:
            tests.teardown_method()
    
    print(f"\n{passed} passed, {failed} failed")

run_integration_tests()
```

---

## 11. End-to-End Testing (Playwright) <a name="playwright"></a>

```python
# ตัวอย่าง 14: Playwright E2E testing
# pip install playwright && playwright install

PLAYWRIGHT_CODE = '''
# tests/e2e/test_checkout.py
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="session")
def browser_context_args():
    return {
        "viewport": {"width": 1280, "height": 720},
        "locale": "th-TH",
        "timezone_id": "Asia/Bangkok"
    }

class TestCheckoutFlow:
    """E2E tests สำหรับ checkout flow"""
    
    def test_user_can_browse_and_add_to_cart(self, page: Page):
        # Navigate to shop
        page.goto("http://localhost:3000")
        expect(page).to_have_title("My Shop")
        
        # Browse products
        page.click("nav >> text=Products")
        expect(page.locator(".product-grid")).to_be_visible()
        
        # Add first product to cart
        page.locator(".product-card").first.click()
        expect(page.locator(".product-detail")).to_be_visible()
        
        page.click("button:has-text('Add to Cart')")
        expect(page.locator(".cart-badge")).to_have_text("1")
    
    def test_user_can_complete_checkout(self, page: Page):
        """Full checkout flow"""
        
        # Login
        page.goto("http://localhost:3000/login")
        page.fill('[name="email"]', "test@example.com")
        page.fill('[name="password"]', "password123")
        page.click('button[type="submit"]')
        
        expect(page).to_have_url("http://localhost:3000/dashboard")
        
        # Add item to cart
        page.goto("http://localhost:3000/products/1")
        page.click("button:has-text('Add to Cart')")
        
        # Go to checkout
        page.click(".cart-icon")
        page.click("text=Proceed to Checkout")
        
        # Fill shipping info
        page.fill('[name="address"]', "123 Test Street")
        page.fill('[name="city"]', "Bangkok")
        page.fill('[name="postal_code"]', "10110")
        
        # Fill payment
        page.fill('[name="card_number"]', "4111111111111111")
        page.fill('[name="expiry"]', "12/25")
        page.fill('[name="cvv"]', "123")
        
        # Place order
        page.click("button:has-text('Place Order')")
        
        # Verify confirmation
        expect(page.locator(".order-confirmation")).to_be_visible()
        expect(page.locator(".order-number")).to_contain_text("ORD-")
    
    def test_search_and_filter_products(self, page: Page):
        """Test search and filter functionality"""
        page.goto("http://localhost:3000/products")
        
        # Search
        page.fill('[placeholder="Search..."]', "laptop")
        page.keyboard.press("Enter")
        
        # Wait for results
        page.wait_for_selector(".search-results")
        
        # Filter by price
        page.click(".filter-toggle")
        page.fill('[name="min_price"]', "500")
        page.fill('[name="max_price"]', "1500")
        page.click("button:has-text('Apply Filter')")
        
        # Verify filtered results
        products = page.locator(".product-card")
        expect(products).to_have_count(pytest.approx(5, abs=3))
        
        # Verify price range
        for product in products.all():
            price_text = product.locator(".price").text_content()
            price = float(price_text.replace("$", "").replace(",", ""))
            assert 500 <= price <= 1500, f"Price {price} out of filter range"

# Run: pytest tests/e2e/ --browser chromium --headed
# or: pytest tests/e2e/ --browser chromium --headless
'''

print("Playwright E2E Tests:")
print(PLAYWRIGHT_CODE)
```

---

## 12. API Testing (httpx, respx) <a name="api-testing"></a>

```python
# ตัวอย่าง 15: API testing with httpx and respx
# pip install httpx respx pytest-asyncio

import asyncio
import json
from typing import Optional
from unittest.mock import AsyncMock, MagicMock, patch

# Simple HTTP client
class APIClient:
    """HTTP client สำหรับ API calls"""
    
    def __init__(self, base_url: str, timeout: float = 30.0):
        self.base_url = base_url
        self.timeout = timeout
        self._session = None
    
    async def get(self, path: str, **kwargs) -> dict:
        return await self._request("GET", path, **kwargs)
    
    async def post(self, path: str, **kwargs) -> dict:
        return await self._request("POST", path, **kwargs)
    
    async def put(self, path: str, **kwargs) -> dict:
        return await self._request("PUT", path, **kwargs)
    
    async def delete(self, path: str, **kwargs) -> dict:
        return await self._request("DELETE", path, **kwargs)
    
    async def _request(self, method: str, path: str, 
                       data: dict = None, **kwargs) -> dict:
        # Simulate HTTP request
        url = f"{self.base_url}{path}"
        
        # In real code: use httpx or aiohttp
        # async with httpx.AsyncClient() as client:
        #     response = await client.request(method, url, json=data)
        
        return {"status": 200, "data": {}, "url": url}


# Mock API responses for testing
class MockHTTPResponse:
    def __init__(self, status_code: int, json_data: dict = None, text: str = ""):
        self.status_code = status_code
        self._json = json_data or {}
        self.text = text or json.dumps(json_data or {})
    
    def json(self):
        return self._json
    
    def raise_for_status(self):
        if self.status_code >= 400:
            raise Exception(f"HTTP {self.status_code}: {self.text}")

class UserAPIClient:
    """User API client สำหรับ testing"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
    
    async def get_user(self, user_id: int) -> dict:
        # In real: return await self.http_client.get(f"/users/{user_id}")
        raise NotImplementedError
    
    async def create_user(self, name: str, email: str) -> dict:
        raise NotImplementedError
    
    async def update_user(self, user_id: int, data: dict) -> dict:
        raise NotImplementedError
    
    async def delete_user(self, user_id: int) -> bool:
        raise NotImplementedError

# Test with mock
class TestUserAPIClient:
    """API tests with mocked HTTP"""
    
    def setup_method(self):
        self.client = UserAPIClient("http://api.example.com")
    
    async def test_get_user_success(self):
        mock_response = MockHTTPResponse(
            status_code=200,
            json_data={"id": 1, "name": "John", "email": "john@example.com"}
        )
        
        # In real test: use respx to mock httpx requests
        # with respx.mock:
        #     respx.get("http://api.example.com/users/1").mock(
        #         return_value=httpx.Response(200, json=mock_response._json)
        #     )
        #     user = await self.client.get_user(1)
        
        # Simulate successful response
        user = mock_response.json()
        
        assert user['id'] == 1
        assert user['name'] == "John"
        assert '@' in user['email']
    
    async def test_get_user_not_found(self):
        mock_response = MockHTTPResponse(
            status_code=404,
            json_data={"error": "User not found"}
        )
        
        try:
            mock_response.raise_for_status()
            assert False, "Should have raised"
        except Exception as e:
            assert "404" in str(e)
    
    def run_all(self):
        print("\nAPI Tests:")
        loop = asyncio.new_event_loop()
        
        for method in [m for m in dir(self) if m.startswith('test_')]:
            self.setup_method()
            try:
                loop.run_until_complete(getattr(self, method)())
                print(f"  ✓ {method}")
            except Exception as e:
                print(f"  ✗ {method}: {e}")
        
        loop.close()

TestUserAPIClient().run_all()
```

---

## 13. Test Doubles <a name="test-doubles"></a>

```python
# ตัวอย่าง 16: Test Doubles - Mock, Stub, Fake, Spy
from unittest.mock import Mock, MagicMock, patch, call, ANY
from typing import Protocol

# Interface
class PaymentGateway(Protocol):
    def charge(self, amount: float, card_token: str) -> dict:
        ...
    
    def refund(self, transaction_id: str, amount: float) -> dict:
        ...

class EmailService(Protocol):
    def send(self, to: str, subject: str, body: str) -> bool:
        ...

# System Under Test
class OrderService:
    def __init__(self, payment_gateway: PaymentGateway, 
                 email_service: EmailService):
        self.payment = payment_gateway
        self.email = email_service
    
    def place_order(self, user_email: str, items: list, 
                    card_token: str) -> dict:
        total = sum(item['price'] * item['quantity'] for item in items)
        
        # Charge payment
        payment_result = self.payment.charge(total, card_token)
        
        if not payment_result.get('success'):
            raise ValueError(f"Payment failed: {payment_result.get('error')}")
        
        order_id = payment_result['transaction_id']
        
        # Send confirmation email
        self.email.send(
            to=user_email,
            subject=f"Order Confirmation #{order_id}",
            body=f"Your order of ${total:.2f} has been placed."
        )
        
        return {
            'order_id': order_id,
            'total': total,
            'status': 'confirmed'
        }
    
    def cancel_order(self, order_id: str, amount: float, 
                     user_email: str) -> dict:
        refund_result = self.payment.refund(order_id, amount)
        
        if refund_result.get('success'):
            self.email.send(
                to=user_email,
                subject=f"Order #{order_id} Cancelled",
                body=f"Refund of ${amount:.2f} initiated."
            )
        
        return refund_result


class TestOrderService:
    
    def setup_method(self):
        # ใช้ Mock objects
        self.mock_payment = Mock(spec=PaymentGateway)
        self.mock_email = Mock(spec=EmailService)
        self.service = OrderService(self.mock_payment, self.mock_email)
    
    def test_successful_order_charges_payment(self):
        # Setup mock
        self.mock_payment.charge.return_value = {
            'success': True,
            'transaction_id': 'txn_123'
        }
        self.mock_email.send.return_value = True
        
        items = [{'price': 50.0, 'quantity': 2}]
        result = self.service.place_order("user@example.com", items, "tok_visa")
        
        # Verify payment was called correctly
        self.mock_payment.charge.assert_called_once_with(100.0, "tok_visa")
        
        # Verify email was sent
        self.mock_email.send.assert_called_once()
        email_call = self.mock_email.send.call_args
        assert email_call[1]['to'] == "user@example.com" or \
               (email_call[0] and email_call[0][0] == "user@example.com")
        
        assert result['order_id'] == 'txn_123'
        assert result['total'] == 100.0
        print("  ✓ successful order charges payment")
    
    def test_failed_payment_raises_exception(self):
        self.mock_payment.charge.return_value = {
            'success': False,
            'error': 'Card declined'
        }
        
        items = [{'price': 50.0, 'quantity': 1}]
        
        try:
            self.service.place_order("user@example.com", items, "tok_declined")
            assert False, "Should have raised ValueError"
        except ValueError as e:
            assert "Payment failed" in str(e)
        
        # Verify email was NOT sent
        self.mock_email.send.assert_not_called()
        print("  ✓ failed payment raises exception")
    
    def test_order_total_calculated_correctly(self):
        self.mock_payment.charge.return_value = {
            'success': True,
            'transaction_id': 'txn_456'
        }
        self.mock_email.send.return_value = True
        
        items = [
            {'price': 10.0, 'quantity': 3},
            {'price': 25.0, 'quantity': 2}
        ]
        
        result = self.service.place_order("user@example.com", items, "tok_visa")
        
        # Verify correct total: 10*3 + 25*2 = 80
        self.mock_payment.charge.assert_called_with(80.0, "tok_visa")
        assert result['total'] == 80.0
        print("  ✓ order total calculated correctly")
    
    def test_cancel_order_sends_refund_email(self):
        self.mock_payment.refund.return_value = {
            'success': True,
            'refund_id': 'ref_123'
        }
        self.mock_email.send.return_value = True
        
        result = self.service.cancel_order("txn_123", 50.0, "user@example.com")
        
        assert result['success'] is True
        self.mock_email.send.assert_called_once()
        print("  ✓ cancel order sends refund email")
    
    def test_mock_spy_verify_call_count(self):
        """Spy สำหรับ verify จำนวน calls"""
        self.mock_payment.charge.return_value = {
            'success': True, 'transaction_id': 'txn_abc'
        }
        self.mock_email.send.return_value = True
        
        items = [{'price': 10.0, 'quantity': 1}]
        
        # Place multiple orders
        for i in range(3):
            self.service.place_order(f"user{i}@example.com", items, "tok_visa")
        
        # Verify call counts
        assert self.mock_payment.charge.call_count == 3
        assert self.mock_email.send.call_count == 3
        print("  ✓ spy verifies call counts")

print("\nTest Doubles Demo:")
tests = TestOrderService()
for method in [m for m in dir(tests) if m.startswith('test_')]:
    tests.setup_method()
    try:
        getattr(tests, method)()
    except Exception as e:
        print(f"  ✗ {method}: {e}")
```

```python
# ตัวอย่าง 17: Fake objects
class FakeEmailService:
    """Fake - simple implementation สำหรับ testing"""
    
    def __init__(self):
        self.sent_emails = []
    
    def send(self, to: str, subject: str, body: str) -> bool:
        self.sent_emails.append({
            'to': to,
            'subject': subject,
            'body': body
        })
        return True
    
    def get_email_for(self, address: str) -> list:
        return [e for e in self.sent_emails if e['to'] == address]
    
    def was_sent_to(self, address: str) -> bool:
        return any(e['to'] == address for e in self.sent_emails)

class FakePaymentGateway:
    """Fake payment gateway สำหรับ testing"""
    
    def __init__(self, should_succeed: bool = True):
        self.should_succeed = should_succeed
        self.transactions = {}
        self._txn_counter = 0
    
    def charge(self, amount: float, card_token: str) -> dict:
        if card_token == "tok_declined":
            return {'success': False, 'error': 'Card declined'}
        
        if not self.should_succeed:
            return {'success': False, 'error': 'Service unavailable'}
        
        self._txn_counter += 1
        txn_id = f"txn_{self._txn_counter:06d}"
        self.transactions[txn_id] = {
            'amount': amount,
            'card_token': card_token,
            'status': 'charged'
        }
        return {'success': True, 'transaction_id': txn_id}
    
    def refund(self, transaction_id: str, amount: float) -> dict:
        if transaction_id not in self.transactions:
            return {'success': False, 'error': 'Transaction not found'}
        
        self.transactions[transaction_id]['status'] = 'refunded'
        return {'success': True, 'refund_id': f"ref_{transaction_id}"}

# Using Fakes
fake_payment = FakePaymentGateway(should_succeed=True)
fake_email = FakeEmailService()
service = OrderService(fake_payment, fake_email)

items = [{'price': 29.99, 'quantity': 2}]
result = service.place_order("customer@example.com", items, "tok_visa")

print("\nFake Objects Demo:")
print(f"  Order placed: {result}")
print(f"  Transactions: {fake_payment.transactions}")
print(f"  Emails sent: {len(fake_email.sent_emails)}")
print(f"  Email to customer: {fake_email.was_sent_to('customer@example.com')}")
```

---

## 14. Testing Async Code <a name="async-testing"></a>

```python
# ตัวอย่าง 18: Testing async functions
import asyncio
import pytest
from typing import Optional

# Async service
class AsyncUserService:
    """Async user service"""
    
    def __init__(self, db, cache):
        self.db = db
        self.cache = cache
    
    async def get_user(self, user_id: int) -> Optional[dict]:
        # Check cache first
        cached = await self.cache.get(f"user:{user_id}")
        if cached:
            return cached
        
        # Get from DB
        user = await self.db.get_user(user_id)
        if user:
            await self.cache.set(f"user:{user_id}", user, ttl=300)
        
        return user
    
    async def create_user(self, name: str, email: str) -> dict:
        # Check email uniqueness
        existing = await self.db.get_user_by_email(email)
        if existing:
            raise ValueError(f"Email already exists: {email}")
        
        user = await self.db.create_user(name, email)
        await self.cache.delete(f"email:{email}")
        return user
    
    async def bulk_update(self, updates: list) -> list:
        """Update multiple users concurrently"""
        tasks = [
            self.db.update_user(uid, data) 
            for uid, data in updates
        ]
        return await asyncio.gather(*tasks)


# Async fakes
class FakeAsyncDB:
    def __init__(self):
        self._users = {}
        self._email_index = {}
        self._counter = 0
    
    async def get_user(self, user_id: int) -> Optional[dict]:
        await asyncio.sleep(0)  # simulate async
        return self._users.get(user_id)
    
    async def get_user_by_email(self, email: str) -> Optional[dict]:
        await asyncio.sleep(0)
        user_id = self._email_index.get(email)
        return self._users.get(user_id) if user_id else None
    
    async def create_user(self, name: str, email: str) -> dict:
        await asyncio.sleep(0)
        self._counter += 1
        user = {'id': self._counter, 'name': name, 'email': email}
        self._users[self._counter] = user
        self._email_index[email] = self._counter
        return user
    
    async def update_user(self, user_id: int, data: dict) -> dict:
        await asyncio.sleep(0)
        if user_id in self._users:
            self._users[user_id].update(data)
            return self._users[user_id]
        return None


class FakeAsyncCache:
    def __init__(self):
        self._store = {}
    
    async def get(self, key: str):
        await asyncio.sleep(0)
        return self._store.get(key)
    
    async def set(self, key: str, value, ttl: int = None):
        await asyncio.sleep(0)
        self._store[key] = value
    
    async def delete(self, key: str):
        await asyncio.sleep(0)
        self._store.pop(key, None)


class TestAsyncUserService:
    
    def setup_method(self):
        self.db = FakeAsyncDB()
        self.cache = FakeAsyncCache()
        self.service = AsyncUserService(self.db, self.cache)
    
    async def test_create_user(self):
        user = await self.service.create_user("Alice", "alice@example.com")
        
        assert user['id'] is not None
        assert user['name'] == "Alice"
        assert user['email'] == "alice@example.com"
    
    async def test_get_user_from_db(self):
        created = await self.service.create_user("Bob", "bob@example.com")
        found = await self.service.get_user(created['id'])
        
        assert found['name'] == "Bob"
    
    async def test_get_user_uses_cache(self):
        created = await self.service.create_user("Carol", "carol@example.com")
        
        # First call - from DB
        await self.service.get_user(created['id'])
        
        # Second call - from cache
        cached_key = f"user:{created['id']}"
        cached = await self.cache.get(cached_key)
        assert cached is not None
        assert cached['name'] == "Carol"
    
    async def test_duplicate_email_raises(self):
        await self.service.create_user("Dave", "dave@example.com")
        
        try:
            await self.service.create_user("Dave2", "dave@example.com")
            assert False, "Should have raised"
        except ValueError as e:
            assert "already exists" in str(e)
    
    async def test_bulk_update(self):
        user1 = await self.service.create_user("User1", "u1@example.com")
        user2 = await self.service.create_user("User2", "u2@example.com")
        
        updates = [
            (user1['id'], {'name': 'Updated1'}),
            (user2['id'], {'name': 'Updated2'})
        ]
        
        results = await self.service.bulk_update(updates)
        assert len(results) == 2
    
    def run_all(self):
        loop = asyncio.new_event_loop()
        passed = failed = 0
        
        for method in [m for m in dir(self) if m.startswith('test_')]:
            self.setup_method()
            try:
                loop.run_until_complete(getattr(self, method)())
                print(f"  ✓ {method}")
                passed += 1
            except Exception as e:
                print(f"  ✗ {method}: {e}")
                failed += 1
        
        loop.close()
        print(f"\n{passed} passed, {failed} failed")

print("\nAsync Testing Demo:")
TestAsyncUserService().run_all()
```

---

## 15. แบบฝึกหัด <a name="exercises"></a>

### แบบฝึกหัดที่ 1-8

```python
# แบบฝึกหัดที่ 1: TDD - Shopping Cart Tax Calculator

# โจทย์: เขียน test ก่อน แล้วค่อย implement
class TestTaxCalculator:
    def test_no_tax_for_zero(self):
        calc = TaxCalculator(rate=0.07)
        assert calc.calculate(0) == 0.0
    
    def test_basic_tax(self):
        calc = TaxCalculator(rate=0.07)
        assert calc.calculate(100.0) == 7.0
    
    def test_tax_rounds_to_2_decimal(self):
        calc = TaxCalculator(rate=0.07)
        assert calc.calculate(1.0) == 0.07
    
    def test_total_with_tax(self):
        calc = TaxCalculator(rate=0.07)
        assert calc.total_with_tax(100.0) == 107.0

# เฉลย
class TaxCalculator:
    def __init__(self, rate: float):
        if not 0 <= rate <= 1:
            raise ValueError(f"Tax rate must be 0-1, got {rate}")
        self.rate = rate
    
    def calculate(self, amount: float) -> float:
        return round(amount * self.rate, 2)
    
    def total_with_tax(self, amount: float) -> float:
        return round(amount + self.calculate(amount), 2)

def run_ex1():
    tests = TestTaxCalculator()
    methods = [m for m in dir(tests) if m.startswith('test_')]
    passed = failed = 0
    for method in methods:
        try:
            getattr(tests, method)()
            print(f"  ✓ {method}")
            passed += 1
        except Exception as e:
            print(f"  ✗ {method}: {e}")
            failed += 1
    print(f"  {passed} passed, {failed} failed")

print("\nEx 1 - TDD Tax Calculator:")
run_ex1()
```

```python
# แบบฝึกหัดที่ 2: Property-based testing สำหรับ sorting

import random

def quicksort(arr: list) -> list:
    """Quicksort implementation"""
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)

def test_quicksort_properties():
    """Property-based tests for quicksort"""
    print("\nEx 2 - Property-based Quicksort Testing:")
    
    properties_passed = 0
    
    for _ in range(1000):
        n = random.randint(0, 50)
        arr = [random.randint(-100, 100) for _ in range(n)]
        sorted_arr = quicksort(arr[:])
        
        # Property 1: Length preserved
        assert len(sorted_arr) == len(arr), "Length changed!"
        
        # Property 2: Sorted
        for i in range(len(sorted_arr) - 1):
            assert sorted_arr[i] <= sorted_arr[i+1], f"Not sorted at index {i}"
        
        # Property 3: Same elements
        assert sorted(sorted_arr) == sorted(arr), "Elements changed!"
        
        # Property 4: Idempotent
        assert quicksort(sorted_arr) == sorted_arr, "Not idempotent!"
        
        properties_passed += 1
    
    print(f"  ✓ All 4 properties hold for 1000 random arrays")
    print(f"  Tested {properties_passed} cases")

test_quicksort_properties()
```

```python
# แบบฝึกหัดที่ 3: Integration test สำหรับ Order system

import sqlite3

class OrderSystem:
    def __init__(self, db):
        self.db = db
        self._setup()
    
    def _setup(self):
        self.db.executescript('''
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER,
                total REAL,
                status TEXT DEFAULT 'pending'
            );
            CREATE TABLE IF NOT EXISTS order_items (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                order_id INTEGER,
                product_id TEXT,
                quantity INTEGER,
                price REAL
            );
        ''')
    
    def create_order(self, user_id: int, items: list) -> dict:
        total = sum(i['price'] * i['quantity'] for i in items)
        
        cursor = self.db.execute(
            'INSERT INTO orders (user_id, total) VALUES (?, ?)',
            (user_id, total)
        )
        order_id = cursor.lastrowid
        
        self.db.executemany(
            'INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?,?,?,?)',
            [(order_id, i['product_id'], i['quantity'], i['price']) for i in items]
        )
        self.db.commit()
        
        return {'id': order_id, 'user_id': user_id, 'total': total, 'status': 'pending'}
    
    def get_order(self, order_id: int) -> dict:
        row = self.db.execute('SELECT * FROM orders WHERE id=?', (order_id,)).fetchone()
        if not row:
            return None
        order = dict(row)
        items = self.db.execute(
            'SELECT * FROM order_items WHERE order_id=?', (order_id,)
        ).fetchall()
        order['items'] = [dict(i) for i in items]
        return order
    
    def cancel_order(self, order_id: int) -> bool:
        cursor = self.db.execute(
            "UPDATE orders SET status='cancelled' WHERE id=? AND status='pending'",
            (order_id,)
        )
        self.db.commit()
        return cursor.rowcount > 0

def test_order_system():
    print("\nEx 3 - Integration Test Order System:")
    
    conn = sqlite3.connect(':memory:')
    conn.row_factory = sqlite3.Row
    system = OrderSystem(conn)
    
    # Test 1: create order
    items = [
        {'product_id': 'P001', 'quantity': 2, 'price': 10.0},
        {'product_id': 'P002', 'quantity': 1, 'price': 25.0}
    ]
    order = system.create_order(user_id=1, items=items)
    assert order['total'] == 45.0
    assert order['status'] == 'pending'
    print("  ✓ create order with correct total")
    
    # Test 2: get order with items
    fetched = system.get_order(order['id'])
    assert fetched is not None
    assert len(fetched['items']) == 2
    print("  ✓ get order with items")
    
    # Test 3: cancel order
    result = system.cancel_order(order['id'])
    assert result is True
    cancelled = system.get_order(order['id'])
    assert cancelled['status'] == 'cancelled'
    print("  ✓ cancel order changes status")
    
    # Test 4: cannot cancel already cancelled
    result2 = system.cancel_order(order['id'])
    assert result2 is False
    print("  ✓ cannot cancel already cancelled order")
    
    conn.close()

test_order_system()
```

```python
# แบบฝึกหัดที่ 4: Mock-based tests

from unittest.mock import Mock, patch, MagicMock

class NotificationService:
    def __init__(self, email_client, sms_client):
        self.email = email_client
        self.sms = sms_client
    
    def notify_order_shipped(self, user: dict, order: dict):
        """Send shipping notification via email and SMS"""
        subject = f"Order #{order['id']} Shipped!"
        body = f"Your order has been shipped. Track: {order['tracking_number']}"
        
        self.email.send(
            to=user['email'],
            subject=subject,
            body=body
        )
        
        if user.get('phone'):
            self.sms.send(
                to=user['phone'],
                message=f"Order #{order['id']} shipped! Tracking: {order['tracking_number']}"
            )

def test_notification_mocks():
    print("\nEx 4 - Mock-based Tests:")
    
    mock_email = Mock()
    mock_sms = Mock()
    
    service = NotificationService(mock_email, mock_sms)
    
    user = {'email': 'user@example.com', 'phone': '+66812345678'}
    order = {'id': 'ORD-001', 'tracking_number': 'TH123456789'}
    
    service.notify_order_shipped(user, order)
    
    # Verify email was sent
    mock_email.send.assert_called_once()
    email_args = mock_email.send.call_args
    assert 'user@example.com' in str(email_args)
    assert 'ORD-001' in str(email_args)
    print("  ✓ email sent with correct data")
    
    # Verify SMS was sent
    mock_sms.send.assert_called_once()
    sms_args = mock_sms.send.call_args
    assert '+66812345678' in str(sms_args)
    print("  ✓ SMS sent to correct number")
    
    # Test without phone
    mock_email.reset_mock()
    mock_sms.reset_mock()
    user_no_phone = {'email': 'noPhone@example.com'}
    service.notify_order_shipped(user_no_phone, order)
    
    mock_email.send.assert_called_once()
    mock_sms.send.assert_not_called()
    print("  ✓ SMS not sent when no phone number")

test_notification_mocks()
```

```python
# แบบฝึกหัดที่ 5: Async test สำหรับ data pipeline

import asyncio
import random
from typing import List, AsyncIterator

async def generate_data(n: int) -> AsyncIterator[dict]:
    for i in range(n):
        await asyncio.sleep(0)  # simulate async
        yield {'id': i, 'value': random.gauss(100, 15), 'category': random.choice(['A','B','C'])}

async def filter_by_category(data: AsyncIterator, category: str) -> AsyncIterator:
    async for item in data:
        if item['category'] == category:
            yield item

async def transform_values(data: AsyncIterator) -> AsyncIterator:
    async for item in data:
        yield {**item, 'normalized': (item['value'] - 100) / 15}

async def collect(data: AsyncIterator) -> List[dict]:
    result = []
    async for item in data:
        result.append(item)
    return result

async def test_async_pipeline():
    print("\nEx 5 - Async Pipeline Test:")
    
    # Test 1: generates correct number of items
    data = generate_data(100)
    items = await collect(data)
    assert len(items) == 100
    print("  ✓ generates 100 items")
    
    # Test 2: filter reduces items
    data = generate_data(300)
    filtered = filter_by_category(data, 'A')
    items = await collect(filtered)
    
    assert all(i['category'] == 'A' for i in items)
    assert len(items) < 300
    print(f"  ✓ filter works: {len(items)}/300 items with category A")
    
    # Test 3: transform adds normalized field
    data = generate_data(10)
    transformed = transform_values(data)
    items = await collect(transformed)
    
    assert all('normalized' in i for i in items)
    print("  ✓ transform adds normalized field")
    
    # Test 4: pipeline composition
    data = generate_data(1000)
    pipeline = transform_values(filter_by_category(data, 'B'))
    items = await collect(pipeline)
    
    assert all(i['category'] == 'B' for i in items)
    assert all('normalized' in i for i in items)
    print(f"  ✓ pipeline composition works: {len(items)} items")

asyncio.run(test_async_pipeline())
```

```python
# แบบฝึกหัดที่ 6: Load test สำหรับ rate limiter

import asyncio
import time
from collections import deque

class RateLimiter:
    """Token bucket rate limiter"""
    
    def __init__(self, rate: float, capacity: int):
        self.rate = rate  # tokens per second
        self.capacity = capacity
        self._tokens = capacity
        self._last_refill = time.time()
        self._lock = asyncio.Lock()
    
    async def acquire(self) -> bool:
        async with self._lock:
            now = time.time()
            elapsed = now - self._last_refill
            
            # Refill tokens
            self._tokens = min(
                self.capacity,
                self._tokens + elapsed * self.rate
            )
            self._last_refill = now
            
            if self._tokens >= 1:
                self._tokens -= 1
                return True
            return False

async def load_test_rate_limiter():
    print("\nEx 6 - Rate Limiter Load Test:")
    
    # Allow 10 requests per second
    limiter = RateLimiter(rate=10, capacity=10)
    
    allowed = 0
    denied = 0
    
    # Send 50 requests instantly (should allow ~10, deny ~40)
    tasks = [limiter.acquire() for _ in range(50)]
    results = await asyncio.gather(*tasks)
    
    allowed = sum(results)
    denied = len(results) - allowed
    
    print(f"  Instant burst: {allowed} allowed, {denied} denied")
    assert allowed <= 10, f"Too many allowed: {allowed}"
    print("  ✓ Rate limiter restricts burst correctly")
    
    # Wait and retry
    await asyncio.sleep(1.0)
    
    allowed2 = 0
    for _ in range(10):
        if await limiter.acquire():
            allowed2 += 1
    
    print(f"  After 1s: {allowed2}/10 allowed")
    assert allowed2 >= 8, f"Too few allowed after refill: {allowed2}"
    print("  ✓ Rate limiter refills correctly")

asyncio.run(load_test_rate_limiter())
```

```python
# แบบฝึกหัดที่ 7: E2E test simulation

class BrowserSimulator:
    """Simulate browser behavior สำหรับ E2E testing"""
    
    def __init__(self):
        self.current_url = ""
        self.page_content = {}
        self.cookies = {}
        self.history = []
        self._setup_pages()
    
    def _setup_pages(self):
        self.page_content = {
            "/": {"title": "Home", "elements": ["nav", "hero", "featured_products"]},
            "/products": {"title": "Products", "elements": ["product_grid", "filter"]},
            "/login": {"title": "Login", "elements": ["email_input", "password_input", "submit"]},
            "/cart": {"title": "Cart", "elements": ["cart_items", "checkout_btn"]},
        }
    
    def navigate(self, url: str) -> bool:
        self.current_url = url
        self.history.append(url)
        return url in self.page_content
    
    def find_element(self, selector: str) -> bool:
        page = self.page_content.get(self.current_url, {})
        return selector in page.get("elements", [])
    
    def fill(self, field: str, value: str) -> bool:
        return self.find_element(field)
    
    def click(self, element: str) -> bool:
        return self.find_element(element)
    
    def get_title(self) -> str:
        return self.page_content.get(self.current_url, {}).get("title", "")
    
    def is_logged_in(self) -> bool:
        return "session" in self.cookies
    
    def login(self, email: str, password: str) -> bool:
        if password == "correct_password":
            self.cookies["session"] = "user_session_token"
            return True
        return False

def test_e2e_user_flow():
    print("\nEx 7 - E2E Test Simulation:")
    
    browser = BrowserSimulator()
    
    # Test 1: navigate to home
    result = browser.navigate("/")
    assert result is True
    assert browser.get_title() == "Home"
    print("  ✓ navigate to home page")
    
    # Test 2: navigate to products
    browser.navigate("/products")
    assert browser.find_element("product_grid")
    assert browser.find_element("filter")
    print("  ✓ products page has correct elements")
    
    # Test 3: login flow
    browser.navigate("/login")
    browser.fill("email_input", "user@example.com")
    browser.fill("password_input", "correct_password")
    success = browser.login("user@example.com", "correct_password")
    assert success
    assert browser.is_logged_in()
    print("  ✓ login flow works")
    
    # Test 4: failed login
    browser2 = BrowserSimulator()
    failed = browser2.login("user@example.com", "wrong_password")
    assert not failed
    assert not browser2.is_logged_in()
    print("  ✓ failed login rejected")
    
    # Test 5: navigation history
    browser3 = BrowserSimulator()
    browser3.navigate("/")
    browser3.navigate("/products")
    browser3.navigate("/cart")
    assert browser3.history == ["/", "/products", "/cart"]
    print("  ✓ navigation history tracked")

test_e2e_user_flow()
```

```python
# แบบฝึกหัดที่ 8: Comprehensive test suite

from unittest.mock import Mock, patch, MagicMock
import time
import threading

class EventSystem:
    """Simple event-driven system"""
    
    def __init__(self):
        self._handlers = {}
        self._history = []
    
    def on(self, event: str, handler):
        if event not in self._handlers:
            self._handlers[event] = []
        self._handlers[event].append(handler)
    
    def emit(self, event: str, **data):
        self._history.append({'event': event, 'data': data, 'time': time.time()})
        
        for handler in self._handlers.get(event, []):
            handler(**data)
    
    def get_history(self, event: str = None) -> list:
        if event:
            return [h for h in self._history if h['event'] == event]
        return self._history

def test_event_system_comprehensive():
    print("\nEx 8 - Comprehensive Event System Tests:")
    
    # Test 1: basic event subscription and emission
    es = EventSystem()
    results = []
    
    es.on("user.created", lambda user_id, **kw: results.append(user_id))
    es.emit("user.created", user_id=1, email="a@example.com")
    es.emit("user.created", user_id=2, email="b@example.com")
    
    assert results == [1, 2]
    print("  ✓ basic event subscription")
    
    # Test 2: multiple handlers for same event
    es2 = EventSystem()
    h1_calls = []
    h2_calls = []
    
    es2.on("order.placed", lambda **kw: h1_calls.append(kw))
    es2.on("order.placed", lambda **kw: h2_calls.append(kw))
    
    es2.emit("order.placed", order_id="ORD-001", amount=50.0)
    
    assert len(h1_calls) == 1
    assert len(h2_calls) == 1
    print("  ✓ multiple handlers receive event")
    
    # Test 3: event history
    es3 = EventSystem()
    es3.emit("login", user_id=1)
    es3.emit("login", user_id=2)
    es3.emit("logout", user_id=1)
    
    login_history = es3.get_history("login")
    assert len(login_history) == 2
    total_history = es3.get_history()
    assert len(total_history) == 3
    print("  ✓ event history recorded correctly")
    
    # Test 4: no handler for event (should not crash)
    es4 = EventSystem()
    es4.emit("nonexistent.event", data="test")
    assert len(es4.get_history()) == 1
    print("  ✓ emitting unhandled event doesn't crash")
    
    # Test 5: mock handler verification
    es5 = EventSystem()
    mock_handler = Mock()
    es5.on("payment.processed", mock_handler)
    
    es5.emit("payment.processed", 
             transaction_id="txn_123", 
             amount=100.0,
             status="success")
    
    mock_handler.assert_called_once_with(
        transaction_id="txn_123",
        amount=100.0,
        status="success"
    )
    print("  ✓ mock handler called with correct arguments")
    
    print("\n  All 5 comprehensive tests passed!")

test_event_system_comprehensive()
```

---

## สรุป

| Testing Type | Tools | When to Use |
|-------------|-------|-------------|
| Unit | pytest, unittest | Individual functions |
| TDD | pytest | New feature development |
| BDD | behave, pytest-bdd | Acceptance criteria |
| Property-based | Hypothesis | Mathematical properties |
| Contract | Pact | Microservices APIs |
| Integration | pytest + fixtures | DB, cache, external |
| Load | Locust | Performance requirements |
| E2E | Playwright | Critical user flows |
| Mutation | mutmut | Test quality check |

**Testing Best Practices:**
1. Test behavior, not implementation
2. Keep tests independent and isolated
3. Use descriptive test names (Given-When-Then)
4. Aim for meaningful coverage, not 100%
5. Test edge cases and error paths
6. Keep tests fast (mock external dependencies)
7. Review test code like production code

---

*Part 98 - Advanced Testing Strategies & TDD | Python Course*
