# Part 22 - OOP: Inheritance & Method Resolution Order

## สารบัญ
1. [Inheritance พื้นฐาน](#inheritance-พื้นฐาน)
2. [super() Function](#super-function)
3. [Method Overriding](#method-overriding)
4. [Single Inheritance](#single-inheritance)
5. [Multiple Inheritance](#multiple-inheritance)
6. [MRO (Method Resolution Order)](#mro-method-resolution-order)
7. [isinstance() และ issubclass()](#isinstance-และ-issubclass)
8. [Mixin Classes](#mixin-classes)
9. [__bases__ และ __mro__](#__bases__-และ-__mro__)
10. [Abstract Base Classes เบื้องต้น](#abstract-base-classes-เบื้องต้น)
11. [ตัวอย่างโปรแกรมจริง](#ตัวอย่างโปรแกรมจริง)
12. [แบบฝึกหัด](#แบบฝึกหัด)

---

## Inheritance พื้นฐาน

**Inheritance (การสืบทอด)** คือกลไกที่ให้ class ใหม่ (subclass/child class) รับ attributes และ methods จาก class เดิม (superclass/parent class) ทำให้:
- ลดการเขียนโค้ดซ้ำ (DRY - Don't Repeat Yourself)
- สร้าง hierarchy ของ classes ที่มีความสัมพันธ์กัน
- ขยายหรือปรับแต่ง behavior ของ class เดิมได้

### ตัวอย่างที่ 1: Inheritance ง่ายที่สุด

```python
class Animal:
    """Parent class (Superclass)"""
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def eat(self):
        print(f"{self.name} กำลังกินอาหาร")
    
    def sleep(self):
        print(f"{self.name} กำลังนอนหลับ")
    
    def breathe(self):
        print(f"{self.name} กำลังหายใจ")
    
    def __str__(self):
        return f"{type(self).__name__}(name={self.name!r}, age={self.age})"


class Dog(Animal):
    """Child class (Subclass) - สืบทอดจาก Animal"""
    
    def bark(self):
        """Method ที่มีเฉพาะ Dog"""
        print(f"{self.name}: โฮ่ง โฮ่ง!")


class Cat(Animal):
    """Child class อีกตัว"""
    
    def meow(self):
        print(f"{self.name}: เมี๊ยว เมี๊ยว!")
    
    def purr(self):
        print(f"{self.name}: กรรรรร...")


# ทดสอบ
dog = Dog("บักโกง", 3)
cat = Cat("มะหมา", 2)

# Dog และ Cat สืบทอด methods จาก Animal
dog.eat()      # บักโกง กำลังกินอาหาร
dog.sleep()    # บักโกง กำลังนอนหลับ
dog.bark()     # บักโกง: โฮ่ง โฮ่ง!

cat.eat()      # มะหมา กำลังกินอาหาร
cat.meow()     # มะหมา: เมี๊ยว เมี๊ยว!
cat.purr()     # มะหมา: กรรรรร...

print(dog)     # Dog(name='บักโกง', age=3) <- ใช้ __str__ จาก Animal
print(cat)     # Cat(name='มะหมา', age=2)
```

### ตัวอย่างที่ 2: Inheritance หลายระดับ (Multi-level)

```python
class LivingThing:
    """Base class สำหรับสิ่งมีชีวิต"""
    
    def __init__(self, name):
        self.name = name
        self.is_alive = True
    
    def metabolize(self):
        print(f"{self.name} กำลัง metabolize")


class Animal(LivingThing):
    """สัตว์ - สืบทอดจาก LivingThing"""
    
    def __init__(self, name, legs):
        super().__init__(name)  # เรียก LivingThing.__init__
        self.legs = legs
    
    def move(self):
        print(f"{self.name} กำลังเคลื่อนไหว (มี {self.legs} ขา)")


class Mammal(Animal):
    """สัตว์เลี้ยงลูกด้วยนม - สืบทอดจาก Animal"""
    
    def __init__(self, name, legs, fur_color):
        super().__init__(name, legs)  # เรียก Animal.__init__
        self.fur_color = fur_color
    
    def nurse_young(self):
        print(f"{self.name} กำลังให้นมลูก")


class Dog(Mammal):
    """สุนัข - สืบทอดจาก Mammal"""
    
    def __init__(self, name, fur_color, breed):
        super().__init__(name, 4, fur_color)  # สุนัขมี 4 ขา
        self.breed = breed
    
    def bark(self):
        print(f"{self.name} ({self.breed}): โฮ่ง!")

# ทดสอบ - Dog สามารถใช้ methods ทุก level ได้
rex = Dog("Rex", "น้ำตาล", "German Shepherd")

rex.metabolize()    # จาก LivingThing
rex.move()          # จาก Animal
rex.nurse_young()   # จาก Mammal
rex.bark()          # จาก Dog

print(f"มีชีวิต: {rex.is_alive}")  # จาก LivingThing
print(f"ขา: {rex.legs}")            # จาก Animal
print(f"สี: {rex.fur_color}")       # จาก Mammal
print(f"พันธุ์: {rex.breed}")       # จาก Dog
```

---

## super() Function

`super()` ส่งคืน proxy object ที่ให้เรียก methods ของ parent class ใช้เพื่อ:
- เรียก `__init__` ของ parent
- เรียก method ที่ถูก override ใน parent
- ทำงานอย่างถูกต้องกับ Multiple Inheritance

### ตัวอย่างที่ 3: super() ใน __init__

```python
class Vehicle:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer = 0
    
    def drive(self, km):
        self.odometer += km
        print(f"{self.make} {self.model} ขับไป {km} กม. (รวม {self.odometer} กม.)")


class Car(Vehicle):
    def __init__(self, make, model, year, num_doors):
        super().__init__(make, model, year)  # เรียก Vehicle.__init__
        self.num_doors = num_doors           # เพิ่ม attribute ของ Car
    
    def honk(self):
        print(f"{self.make} {self.model}: บี๊บ!")


class ElectricCar(Car):
    def __init__(self, make, model, year, num_doors, battery_kwh):
        super().__init__(make, model, year, num_doors)  # เรียก Car.__init__
        self.battery_kwh = battery_kwh                  # เพิ่ม attribute ของ ElectricCar
        self.charge_level = 100  # เปอร์เซ็นต์
    
    def charge(self):
        self.charge_level = 100
        print(f"{self.make} {self.model}: ชาร์จเต็มแล้ว ({self.battery_kwh} kWh)")
    
    def drive(self, km):
        consumption = km * 0.15  # 15 kWh/100 km (สมมติ)
        self.charge_level -= consumption
        super().drive(km)  # เรียก Car.drive -> Vehicle.drive
        print(f"พลังงานที่เหลือ: {self.charge_level:.1f}%")


tesla = ElectricCar("Tesla", "Model 3", 2023, 4, 75)
tesla.drive(50)
tesla.charge()
tesla.honk()

print(f"ประตู: {tesla.num_doors}")
print(f"แบตเตอรี่: {tesla.battery_kwh} kWh")
```

### ตัวอย่างที่ 4: super() เพื่อขยาย Method

```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")


class TimestampLogger(Logger):
    def log(self, message):
        from datetime import datetime
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        # เรียก parent's log แล้วเพิ่ม timestamp
        super().log(f"[{timestamp}] {message}")


class FileLogger(TimestampLogger):
    def __init__(self, filename):
        self.filename = filename
        self.messages = []
    
    def log(self, message):
        super().log(message)  # พิมพ์ด้วย timestamp
        self.messages.append(message)  # บันทึกลง list ด้วย
    
    def save(self):
        # จำลองการบันทึกไฟล์
        print(f"บันทึก {len(self.messages)} messages ลงไฟล์ {self.filename}")


logger = FileLogger("app.log")
logger.log("เริ่มต้นระบบ")
logger.log("ผู้ใช้ login")
logger.log("สิ้นสุดการทำงาน")
logger.save()
```

---

## Method Overriding

**Method Overriding** คือการกำหนด method ใน subclass ที่มีชื่อเดียวกับ method ใน parent class ทำให้ subclass มี behavior ต่างออกไป

### ตัวอย่างที่ 5: Method Overriding พื้นฐาน

```python
class Shape:
    def __init__(self, color="white"):
        self.color = color
    
    def area(self):
        raise NotImplementedError("Subclasses must implement area()")
    
    def perimeter(self):
        raise NotImplementedError("Subclasses must implement perimeter()")
    
    def describe(self):
        """Method ที่ไม่ถูก override - ใช้ร่วมกัน"""
        print(f"รูปทรง: {type(self).__name__}, สี: {self.color}")
        print(f"  พื้นที่: {self.area():.2f}")
        print(f"  เส้นรอบรูป: {self.perimeter():.2f}")


class Circle(Shape):
    def __init__(self, radius, color="white"):
        super().__init__(color)
        self.radius = radius
    
    def area(self):  # Override Shape.area()
        import math
        return math.pi * self.radius ** 2
    
    def perimeter(self):  # Override Shape.perimeter()
        import math
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, width, height, color="white"):
        super().__init__(color)
        self.width = width
        self.height = height
    
    def area(self):  # Override
        return self.width * self.height
    
    def perimeter(self):  # Override
        return 2 * (self.width + self.height)


class Triangle(Shape):
    def __init__(self, a, b, c, color="white"):
        super().__init__(color)
        self.a = a  # ด้านทั้งสาม
        self.b = b
        self.c = c
    
    def area(self):  # Override - Heron's formula
        s = (self.a + self.b + self.c) / 2
        return (s * (s-self.a) * (s-self.b) * (s-self.c)) ** 0.5
    
    def perimeter(self):  # Override
        return self.a + self.b + self.c


# ทดสอบ
shapes = [
    Circle(5, "แดง"),
    Rectangle(4, 6, "น้ำเงิน"),
    Triangle(3, 4, 5, "เขียว"),
]

for shape in shapes:
    shape.describe()
    print()
```

### ตัวอย่างที่ 6: Method Overriding พร้อม super()

```python
class Animal:
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound
    
    def speak(self):
        print(f"{self.name} พูดว่า: {self.sound}")
    
    def describe(self):
        print(f"ฉันเป็น {type(self).__name__} ชื่อ {self.name}")


class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name, "โฮ่ง!")
        self.breed = breed
    
    def speak(self):  # Override + extend
        super().speak()  # เรียก Animal.speak() ก่อน
        print(f"  (พันธุ์ {self.breed} มักเห่าดัง)")
    
    def describe(self):  # Override + extend
        super().describe()  # เรียก Animal.describe() ก่อน
        print(f"  พันธุ์: {self.breed}")


class TrainedDog(Dog):
    def __init__(self, name, breed, tricks):
        super().__init__(name, breed)
        self.tricks = tricks
    
    def speak(self):  # Override + extend อีกชั้น
        print(f"[สุนัขฝึก] ", end="")
        super().speak()  # เรียก Dog.speak()
    
    def perform(self):
        print(f"{self.name} แสดง tricks: {', '.join(self.tricks)}")


# ทดสอบ
rex = TrainedDog("Rex", "Border Collie", ["นั่ง", "นอน", "กลิ้ง"])
rex.speak()
rex.describe()
rex.perform()
```

---

## Single Inheritance

**Single Inheritance** คือ class สืบทอดจาก parent class เพียงหนึ่งเดียว เป็นรูปแบบที่ง่ายและปลอดภัยที่สุด

### ตัวอย่างที่ 7: Single Inheritance Chain

```python
class Employee:
    """Base employee class"""
    
    _employee_count = 0
    
    def __init__(self, name, employee_id, base_salary):
        Employee._employee_count += 1
        self.name = name
        self.employee_id = employee_id
        self._base_salary = base_salary
        self.years_of_service = 0
    
    @property
    def base_salary(self):
        return self._base_salary
    
    @base_salary.setter
    def base_salary(self, value):
        if value < 0:
            raise ValueError("เงินเดือนต้องไม่ติดลบ")
        self._base_salary = value
    
    def calculate_pay(self):
        """คำนวณเงินเดือน - subclasses จะ override"""
        return self._base_salary
    
    def get_annual_bonus(self):
        """โบนัสประจำปี - 1 เดือน"""
        return self._base_salary
    
    def promote(self, raise_percent):
        old = self._base_salary
        self._base_salary *= (1 + raise_percent / 100)
        print(f"{self.name}: เลื่อนตำแหน่ง เงินเดือน {old:,.0f} -> {self._base_salary:,.0f}")
    
    def payslip(self):
        pay = self.calculate_pay()
        print(f"\nสลิปเงินเดือน: {self.name} ({self.employee_id})")
        print(f"  เงินเดือนที่ได้รับ: {pay:>12,.2f} บาท")
    
    def __str__(self):
        return f"{type(self).__name__}({self.employee_id}: {self.name})"
    
    @classmethod
    def total_employees(cls):
        return cls._employee_count


class Manager(Employee):
    """ผู้จัดการ"""
    
    def __init__(self, name, employee_id, base_salary, department, team_size):
        super().__init__(name, employee_id, base_salary)
        self.department = department
        self.team_size = team_size
        self._subordinates = []
    
    def add_subordinate(self, employee):
        self._subordinates.append(employee)
        print(f"{employee.name} อยู่ใต้บังคับบัญชา {self.name}")
    
    def calculate_pay(self):
        """ผู้จัดการได้เงินเดือนพื้นฐาน + bonus ตามขนาดทีม"""
        team_bonus = self.team_size * 500
        return super().calculate_pay() + team_bonus
    
    def get_annual_bonus(self):
        """ผู้จัดการได้โบนัส 2 เดือน"""
        return self._base_salary * 2
    
    def team_meeting(self, agenda):
        print(f"\n=== การประชุมทีม {self.department} ===")
        print(f"หัวหน้า: {self.name}")
        print(f"ผู้เข้าร่วม: {', '.join(e.name for e in self._subordinates)}")
        print(f"ระเบียบวาระ: {agenda}")
    
    def payslip(self):
        super().payslip()
        print(f"  โบนัสทีม: {self.team_size * 500:>12,.2f} บาท")
        print(f"  รวมทั้งสิ้น: {self.calculate_pay():>12,.2f} บาท")


class SeniorManager(Manager):
    """ผู้จัดการอาวุโส"""
    
    def __init__(self, name, employee_id, base_salary, department, team_size, budget):
        super().__init__(name, employee_id, base_salary, department, team_size)
        self.budget = budget
    
    def calculate_pay(self):
        """ผู้จัดการอาวุโสได้ 20% เพิ่มจาก Manager"""
        return super().calculate_pay() * 1.2
    
    def get_annual_bonus(self):
        """โบนัส 3 เดือน"""
        return self._base_salary * 3
    
    def approve_budget(self, amount, purpose):
        if amount > self.budget:
            print(f"ปฏิเสธ: วงเงิน {amount:,} เกิน budget {self.budget:,}")
        else:
            print(f"อนุมัติ {amount:,} บาท สำหรับ: {purpose}")


# ทดสอบ
emp1 = Employee("สมชาย ใจดี", "EMP001", 25000)
mgr1 = Manager("สมหญิง มีทรัพย์", "MGR001", 50000, "IT", 5)
smgr = SeniorManager("สมศักดิ์ ยิ่งใหญ่", "SMGR001", 80000, "Engineering", 20, 500000)

mgr1.add_subordinate(emp1)

emp1.payslip()
mgr1.payslip()

print(f"\nโบนัส {emp1.name}: {emp1.get_annual_bonus():,.0f} บาท")
print(f"โบนัส {mgr1.name}: {mgr1.get_annual_bonus():,.0f} บาท")
print(f"โบนัส {smgr.name}: {smgr.get_annual_bonus():,.0f} บาท")

smgr.approve_budget(100000, "ซื้อ Server ใหม่")
smgr.approve_budget(600000, "Renovation")
```

---

## Multiple Inheritance

**Multiple Inheritance** คือ class สืบทอดจาก parent classes หลายตัวพร้อมกัน Python รองรับ แต่ต้องใช้ด้วยความระมัดระวัง

### ตัวอย่างที่ 8: Multiple Inheritance พื้นฐาน

```python
class Flyable:
    """Mixin: สิ่งที่บินได้"""
    
    def fly(self):
        print(f"{self.name} กำลังบิน!")
    
    def land(self):
        print(f"{self.name} ลงจอด")


class Swimmable:
    """Mixin: สิ่งที่ว่ายน้ำได้"""
    
    def swim(self):
        print(f"{self.name} กำลังว่ายน้ำ!")
    
    def dive(self):
        print(f"{self.name} ดำน้ำ")


class Runnable:
    """Mixin: สิ่งที่วิ่งได้"""
    
    def run(self):
        print(f"{self.name} กำลังวิ่ง!")


class Animal:
    def __init__(self, name):
        self.name = name


class Duck(Animal, Flyable, Swimmable, Runnable):
    """เป็ด - บินได้, ว่ายน้ำได้, วิ่งได้"""
    
    def quack(self):
        print(f"{self.name}: แก้ก แก้ก!")


class Fish(Animal, Swimmable):
    """ปลา - ว่ายน้ำได้อย่างเดียว"""
    pass


class Eagle(Animal, Flyable, Runnable):
    """นกอินทรี - บินได้, วิ่งได้"""
    
    def hunt(self):
        print(f"{self.name} กำลังล่าเหยื่อ!")


# ทดสอบ
duck = Duck("โดนัลด์")
duck.fly()
duck.swim()
duck.run()
duck.quack()

fish = Fish("นีโม")
fish.swim()
fish.dive()
# fish.fly()  # AttributeError - ปลาบินไม่ได้

eagle = Eagle("ซุส")
eagle.fly()
eagle.run()
eagle.hunt()
# eagle.swim()  # AttributeError - นกอินทรีว่ายน้ำไม่ได้
```

### ตัวอย่างที่ 9: Diamond Problem

```python
# Diamond Problem - เกิดเมื่อ class เดียวกันปรากฏหลายครั้งใน hierarchy

class A:
    def method(self):
        print("A.method()")


class B(A):
    def method(self):
        print("B.method()")
        super().method()


class C(A):
    def method(self):
        print("C.method()")
        super().method()


class D(B, C):
    """D สืบทอดจากทั้ง B และ C ที่ต่างก็สืบทอดจาก A"""
    def method(self):
        print("D.method()")
        super().method()


# ทดสอบ - Python ใช้ MRO (C3 Linearization) จัดการ diamond problem
d = D()
d.method()
# D.method()
# B.method()
# C.method()
# A.method()

print("\nMRO ของ D:", [c.__name__ for c in D.__mro__])
# ['D', 'B', 'C', 'A', 'object']
```

---

## MRO (Method Resolution Order)

**MRO** กำหนดลำดับที่ Python ค้นหา method ใน class hierarchy ใช้ **C3 Linearization algorithm**

### ตัวอย่างที่ 10: ทำความเข้าใจ MRO

```python
class Base:
    def greet(self):
        print("Base.greet()")

class Mixin1(Base):
    def greet(self):
        print("Mixin1.greet()")
        super().greet()

class Mixin2(Base):
    def greet(self):
        print("Mixin2.greet()")
        super().greet()

class Child(Mixin1, Mixin2):
    def greet(self):
        print("Child.greet()")
        super().greet()

# แสดง MRO
print("MRO:", [c.__name__ for c in Child.__mro__])
# ['Child', 'Mixin1', 'Mixin2', 'Base', 'object']

c = Child()
c.greet()
# Child.greet()
# Mixin1.greet()
# Mixin2.greet()
# Base.greet()
```

### ตัวอย่างที่ 11: MRO กับ super() ที่ซับซ้อน

```python
class Logger:
    def log(self, msg, level="INFO"):
        print(f"[{level}] {msg}")

class TimestampMixin(Logger):
    def log(self, msg, level="INFO"):
        from datetime import datetime
        ts = datetime.now().strftime("%H:%M:%S")
        super().log(f"[{ts}] {msg}", level)

class PrefixMixin(Logger):
    prefix = "APP"
    
    def log(self, msg, level="INFO"):
        super().log(f"[{self.prefix}] {msg}", level)

class AppLogger(TimestampMixin, PrefixMixin):
    """รวม timestamp และ prefix"""
    pass

print("AppLogger MRO:")
for cls in AppLogger.__mro__:
    print(f"  {cls.__name__}")

app_log = AppLogger()
app_log.log("เริ่มต้นแอปพลิเคชัน")
# [INFO] [APP] [HH:MM:SS] เริ่มต้นแอปพลิเคชัน
```

### ตัวอย่างที่ 12: ตรวจสอบ MRO ด้วย mro() method

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
class E(C): pass
class F(D, E): pass

print("MRO ของ F:")
for i, cls in enumerate(F.mro()):
    print(f"  {i+1}. {cls.__name__}")

# ตรวจสอบว่า class ใดอยู่ใน MRO
print("\nA อยู่ใน MRO ของ F:", A in F.__mro__)  # True
print("B อยู่ใน MRO ของ F:", B in F.__mro__)  # True

# ตรวจสอบว่า method จะถูกเรียกจาก class ไหน
print("\nmethod 'greet' จะถูกเรียกจาก class ไหน:")
for cls in F.__mro__:
    if 'greet' in cls.__dict__:
        print(f"  -> จาก {cls.__name__}")
        break
else:
    print("  -> ไม่พบ greet")
```

---

## isinstance() และ issubclass()

ฟังก์ชันสำหรับตรวจสอบความสัมพันธ์ระหว่าง objects และ classes

### ตัวอย่างที่ 13: isinstance() และ issubclass()

```python
class Vehicle:
    pass

class Car(Vehicle):
    pass

class ElectricCar(Car):
    pass

car = Car()
ecar = ElectricCar()

# isinstance() - ตรวจสอบว่า object เป็น instance ของ class (หรือ subclass)
print("isinstance() :")
print(f"  isinstance(car, Car)      = {isinstance(car, Car)}")       # True
print(f"  isinstance(car, Vehicle)  = {isinstance(car, Vehicle)}")   # True (Car สืบทอดจาก Vehicle)
print(f"  isinstance(car, ElectricCar) = {isinstance(car, ElectricCar)}")  # False

print(f"  isinstance(ecar, ElectricCar) = {isinstance(ecar, ElectricCar)}")  # True
print(f"  isinstance(ecar, Car)     = {isinstance(ecar, Car)}")      # True
print(f"  isinstance(ecar, Vehicle) = {isinstance(ecar, Vehicle)}")  # True

# ตรวจสอบหลาย types พร้อมกัน
data = [1, "hello", 3.14, True, [1, 2, 3]]
for item in data:
    print(f"{item!r:20} is str or int: {isinstance(item, (str, int))}")

# issubclass() - ตรวจสอบความสัมพันธ์ระหว่าง classes
print("\nissubclass() :")
print(f"  issubclass(Car, Vehicle)   = {issubclass(Car, Vehicle)}")    # True
print(f"  issubclass(ElectricCar, Car)    = {issubclass(ElectricCar, Car)}")   # True
print(f"  issubclass(ElectricCar, Vehicle)= {issubclass(ElectricCar, Vehicle)}")# True
print(f"  issubclass(Vehicle, Car)   = {issubclass(Vehicle, Car)}")    # False
print(f"  issubclass(Car, Car)       = {issubclass(Car, Car)}")        # True (ตัวเอง)
```

### ตัวอย่างที่ 14: ใช้ isinstance() ใน Duck Typing

```python
class Duck:
    def quack(self):
        return "แก้ก!"
    
    def swim(self):
        return "Duck กำลังว่ายน้ำ"

class Person:
    def quack(self):
        return "ฉันเลียนเสียงเป็ด: แก้ก!"
    
    def swim(self):
        return "Person กำลังว่ายน้ำ"

class Robot:
    def quack(self):
        return "BEEP QUACK"
    # ไม่มี swim method


def in_the_pond(thing):
    """ทุกอย่างที่ quack และ swim ได้ถือว่าเป็น 'เป็ด' สำหรับ function นี้"""
    
    # วิธีที่ 1: ใช้ isinstance (ตรวจ type อย่างเข้มงวด)
    # if isinstance(thing, Duck):
    #     ...
    
    # วิธีที่ 2: Duck Typing - ใช้ hasattr (แนะนำสำหรับ Python)
    if hasattr(thing, 'quack') and hasattr(thing, 'swim'):
        print(thing.quack())
        print(thing.swim())
    else:
        print(f"{type(thing).__name__} ไม่สามารถอยู่ในสระน้ำได้")

duck = Duck()
person = Person()
robot = Robot()

in_the_pond(duck)    # Duck ทำได้
in_the_pond(person)  # Person ทำได้
in_the_pond(robot)   # Robot ทำไม่ได้ (ไม่มี swim)
```

---

## Mixin Classes

**Mixin** คือ class ที่ออกแบบมาเพื่อเพิ่ม functionality ให้ classes อื่น โดยไม่ใช่เป็น parent class หลัก

### ตัวอย่างที่ 15: Mixin Classes

```python
class JSONMixin:
    """เพิ่มความสามารถ JSON serialization"""
    
    def to_json(self):
        import json
        return json.dumps(self.__dict__, ensure_ascii=False, indent=2)
    
    @classmethod
    def from_json(cls, json_string):
        import json
        data = json.loads(json_string)
        obj = cls.__new__(cls)
        obj.__dict__.update(data)
        return obj


class PrintableMixin:
    """เพิ่มความสามารถ pretty printing"""
    
    def pretty_print(self):
        print(f"\n{'='*40}")
        print(f"  {type(self).__name__}")
        print(f"{'='*40}")
        for key, value in self.__dict__.items():
            if not key.startswith('_'):
                print(f"  {key:20}: {value}")
        print(f"{'='*40}\n")


class ComparableMixin:
    """เพิ่มความสามารถ comparison"""
    
    def __eq__(self, other):
        if type(self) != type(other):
            return NotImplemented
        return self.__dict__ == other.__dict__
    
    def __hash__(self):
        return hash(tuple(sorted(self.__dict__.items())))


class User(JSONMixin, PrintableMixin, ComparableMixin):
    """User class ที่ใช้ Mixins หลายตัว"""
    
    def __init__(self, username, email, age):
        self.username = username
        self.email = email
        self.age = age
    
    def __repr__(self):
        return f"User({self.username!r}, {self.email!r})"


class Product(JSONMixin, PrintableMixin):
    """Product class ที่ใช้บาง Mixins"""
    
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock


# ทดสอบ
user1 = User("john_doe", "john@example.com", 30)
user2 = User("jane_doe", "jane@example.com", 25)
user3 = User("john_doe", "john@example.com", 30)

user1.pretty_print()

json_str = user1.to_json()
print("JSON:")
print(json_str)

print(f"\nuser1 == user2: {user1 == user2}")  # False
print(f"user1 == user3: {user1 == user3}")  # True

product = Product("Python Book", 599, 100)
product.pretty_print()
product_json = product.to_json()
print("Product JSON:", product_json)
```

### ตัวอย่างที่ 16: Timestamp Mixin สำหรับ Database Models

```python
from datetime import datetime

class TimestampMixin:
    """เพิ่ม created_at, updated_at ให้กับ models"""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
    
    def touch(self):
        """อัปเดต updated_at timestamp"""
        self.updated_at = datetime.now()
    
    def age_seconds(self):
        """อายุของ record เป็นวินาที"""
        return (datetime.now() - self.created_at).total_seconds()


class SoftDeleteMixin:
    """Soft delete - ไม่ลบจริง แค่ mark ว่าลบแล้ว"""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.deleted_at = None
        self.is_deleted = False
    
    def delete(self):
        if not self.is_deleted:
            self.is_deleted = True
            self.deleted_at = datetime.now()
            print(f"Soft deleted: {self}")
    
    def restore(self):
        if self.is_deleted:
            self.is_deleted = False
            self.deleted_at = None
            print(f"Restored: {self}")


class BaseModel:
    """Base model class"""
    
    def __init__(self, id):
        self.id = id
    
    def save(self):
        print(f"Saving {type(self).__name__} id={self.id}")


class Article(TimestampMixin, SoftDeleteMixin, BaseModel):
    """Article model ใช้ทุก Mixins"""
    
    def __init__(self, id, title, content):
        super().__init__(id=id)  # จะ chain ผ่าน MRO
        self.title = title
        self.content = content
    
    def __str__(self):
        return f"Article(id={self.id}, title={self.title!r})"
    
    def __repr__(self):
        return str(self)


# ทดสอบ
article = Article(1, "Python Tutorial", "เนื้อหา Python...")
print(f"Created: {article.created_at.strftime('%Y-%m-%d %H:%M:%S')}")
print(f"Is deleted: {article.is_deleted}")

import time
time.sleep(0.1)  # รอ 0.1 วินาที

article.touch()
print(f"Updated: {article.updated_at.strftime('%Y-%m-%d %H:%M:%S')}")

article.delete()
print(f"Is deleted: {article.is_deleted}")

article.restore()
print(f"Is deleted: {article.is_deleted}")

print(f"MRO: {[c.__name__ for c in Article.__mro__]}")
```

---

## __bases__ และ __mro__

Attributes พิเศษสำหรับ introspection เกี่ยวกับ class hierarchy

### ตัวอย่างที่ 17: __bases__ และ __mro__

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

# __bases__ - parent classes โดยตรง (เพียงระดับเดียว)
print("A.__bases__:", A.__bases__)     # (<class 'object'>,)
print("B.__bases__:", B.__bases__)     # (<class '__main__.A'>,)
print("D.__bases__:", D.__bases__)     # (<class '__main__.B'>, <class '__main__.C'>)

# __mro__ - ลำดับ method resolution ทั้งหมด
print("\nD.__mro__:", D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)

# D.mro() - เหมือนกัน แต่คืน list
print("D.mro():", D.mro())

# ตรวจสอบ parent classes ทั้งหมดด้วยฟังก์ชัน
def get_all_parents(cls):
    """ดึง parent classes ทั้งหมด (ไม่รวมตัวเองและ object)"""
    return [c for c in cls.__mro__[1:] if c is not object]

print("\nParent classes ของ D:", [c.__name__ for c in get_all_parents(D)])

# ตรวจสอบ class hierarchy ทั้งหมด
def print_class_tree(cls, indent=0):
    """แสดง class hierarchy"""
    print("  " * indent + cls.__name__)
    for base in cls.__bases__:
        if base is not object:
            print_class_tree(base, indent + 1)

print("\nClass Tree ของ D:")
print_class_tree(D)
```

---

## Abstract Base Classes เบื้องต้น

**Abstract Base Classes (ABC)** ช่วยกำหนด "interface" ที่ subclasses ต้องปฏิบัติตาม

### ตัวอย่างที่ 18: ABC เบื้องต้น

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    """Abstract class - ไม่สามารถสร้าง instance ได้โดยตรง"""
    
    def __init__(self, color="white"):
        self.color = color
    
    @abstractmethod
    def area(self):
        """Subclasses ต้อง implement method นี้"""
        pass
    
    @abstractmethod
    def perimeter(self):
        """Subclasses ต้อง implement method นี้"""
        pass
    
    def describe(self):
        """Concrete method - ไม่ต้อง override"""
        print(f"{type(self).__name__}: พื้นที่={self.area():.2f}, "
              f"เส้นรอบรูป={self.perimeter():.2f}, สี={self.color}")


class Circle(Shape):
    def __init__(self, radius, color="white"):
        super().__init__(color)
        self.radius = radius
    
    def area(self):  # ต้อง implement
        import math
        return math.pi * self.radius ** 2
    
    def perimeter(self):  # ต้อง implement
        import math
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, w, h, color="white"):
        super().__init__(color)
        self.w = w
        self.h = h
    
    def area(self):
        return self.w * self.h
    
    def perimeter(self):
        return 2 * (self.w + self.h)


# ทดสอบ
try:
    s = Shape("red")  # TypeError! ไม่สามารถสร้าง instance ของ abstract class
except TypeError as e:
    print(f"Error: {e}")

c = Circle(5, "แดง")
r = Rectangle(4, 6, "น้ำเงิน")

c.describe()
r.describe()

shapes = [c, r, Circle(3, "เขียว")]
total_area = sum(s.area() for s in shapes)
print(f"\nพื้นที่รวม: {total_area:.2f}")
```

### ตัวอย่างที่ 19: ABC กับ register()

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass

# Virtual subclass - class ที่ไม่ได้สืบทอดจริงแต่ register ได้
class ExternalShape:
    """Class จาก library ภายนอกที่เราแก้ไขไม่ได้"""
    def draw(self):
        print("ExternalShape.draw()")

# Register ExternalShape เป็น virtual subclass ของ Drawable
Drawable.register(ExternalShape)

ext = ExternalShape()
print(isinstance(ext, Drawable))   # True (แม้ไม่ได้สืบทอด)
print(issubclass(ExternalShape, Drawable))  # True
```

---

## ตัวอย่างโปรแกรมจริง

### โปรแกรมที่ 1: Animal Hierarchy

```python
from abc import ABC, abstractmethod
from datetime import date

class Animal(ABC):
    """Abstract base class สำหรับสัตว์ทุกชนิด"""
    
    def __init__(self, name, species, birth_date, weight_kg):
        self.name = name
        self.species = species
        self.birth_date = birth_date
        self.weight_kg = weight_kg
        self._health = 100
        self._hunger = 50
    
    @property
    def age_years(self):
        today = date.today()
        return (today - self.birth_date).days / 365.25
    
    @property
    def health(self):
        return self._health
    
    @property
    def hunger(self):
        return self._hunger
    
    @abstractmethod
    def speak(self):
        """สัตว์แต่ละชนิดส่งเสียงต่างกัน"""
        pass
    
    @abstractmethod
    def move(self):
        """สัตว์แต่ละชนิดเคลื่อนไหวต่างกัน"""
        pass
    
    def eat(self, food, amount):
        self._hunger = max(0, self._hunger - amount * 5)
        self._health = min(100, self._health + amount)
        print(f"{self.name} กิน{food} {amount} กรัม | ความหิว: {self._hunger}%")
    
    def rest(self):
        self._health = min(100, self._health + 10)
        print(f"{self.name} พักผ่อน | สุขภาพ: {self._health}%")
    
    def status(self):
        print(f"\n=== {self.name} ({self.species}) ===")
        print(f"  อายุ: {self.age_years:.1f} ปี | น้ำหนัก: {self.weight_kg} กก.")
        print(f"  สุขภาพ: {self._health}% | ความหิว: {self._hunger}%")
    
    def __str__(self):
        return f"{type(self).__name__}(name={self.name!r}, species={self.species!r})"


class Mammal(Animal):
    """สัตว์เลี้ยงลูกด้วยนม"""
    
    def __init__(self, name, species, birth_date, weight_kg, fur_color):
        super().__init__(name, species, birth_date, weight_kg)
        self.fur_color = fur_color
    
    def move(self):
        print(f"{self.name} วิ่งด้วยขา")
    
    def groom(self):
        print(f"{self.name} ทำความสะอาดขน ({self.fur_color})")


class Bird(Animal):
    """นก"""
    
    def __init__(self, name, species, birth_date, weight_kg, wing_span_cm, can_fly=True):
        super().__init__(name, species, birth_date, weight_kg)
        self.wing_span_cm = wing_span_cm
        self.can_fly = can_fly
    
    def move(self):
        if self.can_fly:
            print(f"{self.name} บินด้วยปีก (wingspan: {self.wing_span_cm} cm)")
        else:
            print(f"{self.name} วิ่ง (บินไม่ได้)")
    
    def preen(self):
        print(f"{self.name} ดูแลขนนก")


class Reptile(Animal):
    """สัตว์เลื้อยคลาน"""
    
    def __init__(self, name, species, birth_date, weight_kg, is_venomous=False):
        super().__init__(name, species, birth_date, weight_kg)
        self.is_venomous = is_venomous
    
    def move(self):
        print(f"{self.name} เลื้อยคลาน")
    
    def bask(self):
        print(f"{self.name} ผึ่งแดด (cold-blooded)")


# Specific animals
class Lion(Mammal):
    def __init__(self, name, birth_date, weight_kg, has_mane=True):
        super().__init__(name, "Panthera leo", birth_date, weight_kg, "น้ำตาลทอง")
        self.has_mane = has_mane
    
    def speak(self):
        print(f"{self.name}: ROOOAAAR!")
    
    def hunt(self, prey):
        print(f"{self.name} ล่า{prey}")


class Parrot(Bird):
    def __init__(self, name, birth_date, weight_kg, vocabulary):
        super().__init__(name, "Psittacidae", birth_date, weight_kg, 30)
        self.vocabulary = vocabulary
    
    def speak(self):
        import random
        word = random.choice(self.vocabulary)
        print(f"{self.name}: {word}")
    
    def learn_word(self, word):
        self.vocabulary.append(word)
        print(f"{self.name} เรียนคำใหม่: '{word}'")


class Crocodile(Reptile):
    def __init__(self, name, birth_date, weight_kg, length_m):
        super().__init__(name, "Crocodylus", birth_date, weight_kg, is_venomous=False)
        self.length_m = length_m
    
    def speak(self):
        print(f"{self.name}: *เงียบ* (ฉันเป็นจระเข้)")
    
    def swim(self):
        print(f"{self.name} ว่ายน้ำ ({self.length_m}m ยาว)")


# =================== ทดสอบ Zoo ===================
class Zoo:
    """สวนสัตว์"""
    
    def __init__(self, name):
        self.name = name
        self.animals = []
    
    def add_animal(self, animal):
        self.animals.append(animal)
        print(f"เพิ่ม {animal.name} เข้าสวนสัตว์ {self.name}")
    
    def feeding_time(self):
        print(f"\n{'='*50}")
        print(f"เวลาให้อาหารที่ {self.name}")
        print(f"{'='*50}")
        for animal in self.animals:
            animal.eat("อาหารสัตว์", 50)
    
    def show_time(self):
        print(f"\n{'='*50}")
        print(f"การแสดงที่ {self.name}")
        print(f"{'='*50}")
        for animal in self.animals:
            animal.speak()
            animal.move()
    
    def status_check(self):
        for animal in self.animals:
            animal.status()
    
    def count_by_type(self):
        from collections import Counter
        type_counts = Counter(type(a).__name__ for a in self.animals)
        print(f"\nสัตว์ในสวนสัตว์ {self.name}:")
        for animal_type, count in type_counts.items():
            print(f"  {animal_type}: {count} ตัว")


zoo = Zoo("สวนสัตว์ Python")

lion = Lion("ซิมบ้า", date(2020, 3, 15), 180)
parrot = Parrot("Polly", date(2021, 6, 1), 0.5, ["สวัสดี", "Python", "อาหาร", "ออก"])
croc = Crocodile("Rex", date(2015, 1, 10), 500, 4.5)

zoo.add_animal(lion)
zoo.add_animal(parrot)
zoo.add_animal(croc)

zoo.feeding_time()
zoo.show_time()
zoo.count_by_type()

# ทดสอบ isinstance
print(f"\nทดสอบ isinstance:")
print(f"lion เป็น Animal: {isinstance(lion, Animal)}")
print(f"lion เป็น Mammal: {isinstance(lion, Mammal)}")
print(f"lion เป็น Bird: {isinstance(lion, Bird)}")
print(f"parrot เป็น Bird: {isinstance(parrot, Bird)}")
```

### โปรแกรมที่ 2: Shape Hierarchy

```python
import math
from abc import ABC, abstractmethod

class Shape2D(ABC):
    """Abstract 2D shape"""
    
    def __init__(self, color="white", filled=True):
        self.color = color
        self.filled = filled
    
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass
    
    def scale(self, factor):
        """Scale shape - subclasses override"""
        raise NotImplementedError
    
    def info(self):
        fill_str = "เติมสี" if self.filled else "ไม่เติมสี"
        print(f"{type(self).__name__} ({self.color}, {fill_str}):")
        print(f"  พื้นที่: {self.area():.4f}")
        print(f"  เส้นรอบรูป: {self.perimeter():.4f}")
    
    def __str__(self):
        return (f"{type(self).__name__}(area={self.area():.2f}, "
                f"perimeter={self.perimeter():.2f})")
    
    def __lt__(self, other):
        return self.area() < other.area()
    
    def __eq__(self, other):
        if type(self) != type(other):
            return False
        return abs(self.area() - other.area()) < 1e-9


class Circle(Shape2D):
    def __init__(self, radius, color="white", filled=True):
        super().__init__(color, filled)
        if radius <= 0:
            raise ValueError("รัศมีต้องมากกว่า 0")
        self.radius = radius
    
    def area(self):
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        return 2 * math.pi * self.radius
    
    def scale(self, factor):
        self.radius *= factor
        return self


class Rectangle(Shape2D):
    def __init__(self, width, height, color="white", filled=True):
        super().__init__(color, filled)
        if width <= 0 or height <= 0:
            raise ValueError("กว้างและสูงต้องมากกว่า 0")
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)
    
    def scale(self, factor):
        self.width *= factor
        self.height *= factor
        return self
    
    def diagonal(self):
        return math.sqrt(self.width**2 + self.height**2)


class Square(Rectangle):
    """สี่เหลี่ยมจัตุรัส - specialization ของ Rectangle"""
    
    def __init__(self, side, color="white", filled=True):
        super().__init__(side, side, color, filled)
    
    @property
    def side(self):
        return self.width
    
    @side.setter
    def side(self, value):
        self.width = value
        self.height = value  # สองด้านต้องเท่ากันเสมอ
    
    def scale(self, factor):
        self.side = self.side * factor
        return self


class Triangle(Shape2D):
    def __init__(self, a, b, c, color="white", filled=True):
        super().__init__(color, filled)
        if a + b <= c or b + c <= a or a + c <= b:
            raise ValueError("สามเหลี่ยมไม่ถูกต้อง")
        self.a = a
        self.b = b
        self.c = c
    
    def area(self):
        s = self.perimeter() / 2
        return math.sqrt(s * (s-self.a) * (s-self.b) * (s-self.c))
    
    def perimeter(self):
        return self.a + self.b + self.c
    
    @property
    def is_equilateral(self):
        return self.a == self.b == self.c
    
    @property
    def is_isosceles(self):
        return self.a == self.b or self.b == self.c or self.a == self.c


# ทดสอบ
shapes = [
    Circle(5, "แดง"),
    Rectangle(4, 6, "น้ำเงิน"),
    Square(4, "เขียว"),
    Triangle(3, 4, 5, "เหลือง"),
]

print("รูปทรงทั้งหมด:")
for shape in shapes:
    shape.info()
    print()

# เรียงตาม area
print("เรียงตาม area:")
for shape in sorted(shapes):
    print(f"  {shape}")

# คำนวณ area รวม
total = sum(s.area() for s in shapes)
print(f"\nพื้นที่รวม: {total:.2f}")

# ตรวจสอบ isinstance
print(f"\nSquare เป็น Rectangle: {isinstance(Square(3), Rectangle)}")
print(f"Square เป็น Shape2D: {isinstance(Square(3), Shape2D)}")
```

### โปรแกรมที่ 3: Employee Hierarchy

```python
from abc import ABC, abstractmethod
from datetime import date
from enum import Enum

class Department(Enum):
    ENGINEERING = "วิศวกรรม"
    MARKETING = "การตลาด"
    HR = "ทรัพยากรบุคคล"
    FINANCE = "การเงิน"
    IT = "IT"

class Employee(ABC):
    """Abstract base class สำหรับพนักงาน"""
    
    _next_id = 1000
    
    def __init__(self, name, department, hire_date):
        Employee._next_id += 1
        self.employee_id = f"EMP{Employee._next_id}"
        self.name = name
        self.department = department
        self.hire_date = hire_date
    
    @property
    def years_of_service(self):
        return (date.today() - self.hire_date).days / 365.25
    
    @abstractmethod
    def calculate_pay(self) -> float:
        """คำนวณเงินเดือน - ต้อง implement"""
        pass
    
    @abstractmethod
    def get_title(self) -> str:
        """ตำแหน่ง - ต้อง implement"""
        pass
    
    def annual_bonus(self) -> float:
        """โบนัสปีละ 1 เดือน (default)"""
        return self.calculate_pay()
    
    def payslip(self):
        print(f"\n{'='*50}")
        print(f"สลิปเงินเดือน: {self.name}")
        print(f"รหัส: {self.employee_id} | ตำแหน่ง: {self.get_title()}")
        print(f"แผนก: {self.department.value}")
        print(f"{'='*50}")
    
    def __str__(self):
        return (f"{self.get_title()}: {self.name} "
                f"({self.department.value})")


class FullTimeEmployee(Employee):
    """พนักงานประจำ"""
    
    def __init__(self, name, department, hire_date, monthly_salary):
        super().__init__(name, department, hire_date)
        self._monthly_salary = monthly_salary
    
    def get_title(self):
        if self.years_of_service >= 10:
            return "พนักงานอาวุโสระดับ 3"
        elif self.years_of_service >= 5:
            return "พนักงานอาวุโสระดับ 2"
        elif self.years_of_service >= 2:
            return "พนักงานอาวุโสระดับ 1"
        else:
            return "พนักงานใหม่"
    
    def calculate_pay(self):
        base = self._monthly_salary
        seniority_bonus = self.years_of_service * 500
        return base + seniority_bonus
    
    def annual_bonus(self):
        """โบนัสตาม performance (สมมติ 2 เดือน)"""
        return self.calculate_pay() * 2
    
    def payslip(self):
        super().payslip()
        print(f"เงินเดือนพื้นฐาน: {self._monthly_salary:>12,.2f} บาท")
        print(f"โบนัสอาวุโส:    {self.years_of_service * 500:>12,.2f} บาท")
        print(f"รวม:            {self.calculate_pay():>12,.2f} บาท")
        print(f"{'='*50}")


class PartTimeEmployee(Employee):
    """พนักงานนอกเวลา"""
    
    def __init__(self, name, department, hire_date, hourly_rate, hours_per_week):
        super().__init__(name, department, hire_date)
        self.hourly_rate = hourly_rate
        self.hours_per_week = hours_per_week
    
    def get_title(self):
        return "พนักงานนอกเวลา"
    
    def calculate_pay(self):
        weeks_per_month = 4.33
        return self.hourly_rate * self.hours_per_week * weeks_per_month
    
    def annual_bonus(self):
        """พนักงานนอกเวลาไม่มีโบนัส"""
        return 0
    
    def payslip(self):
        super().payslip()
        monthly = self.calculate_pay()
        print(f"อัตราชั่วโมงละ: {self.hourly_rate:>10,.2f} บาท")
        print(f"ชั่วโมง/สัปดาห์: {self.hours_per_week:>11} ชั่วโมง")
        print(f"รายได้ต่อเดือน: {monthly:>10,.2f} บาท")
        print(f"{'='*50}")


class Contractor(Employee):
    """ผู้รับเหมา"""
    
    def __init__(self, name, department, hire_date, project_rate, projects_per_month):
        super().__init__(name, department, hire_date)
        self.project_rate = project_rate
        self.projects_per_month = projects_per_month
    
    def get_title(self):
        return "ผู้รับเหมา"
    
    def calculate_pay(self):
        return self.project_rate * self.projects_per_month
    
    def annual_bonus(self):
        """ผู้รับเหมาไม่มีโบนัส"""
        return 0
    
    def payslip(self):
        super().payslip()
        print(f"ค่าโปรเจกต์:   {self.project_rate:>12,.2f} บาท/โปรเจกต์")
        print(f"โปรเจกต์/เดือน: {self.projects_per_month:>11} โปรเจกต์")
        print(f"รวม:           {self.calculate_pay():>12,.2f} บาท")
        print(f"{'='*50}")


# =================== ทดสอบ HR System ===================

class HRSystem:
    def __init__(self, company_name):
        self.company = company_name
        self.employees = []
    
    def hire(self, employee):
        self.employees.append(employee)
        print(f"รับ {employee.name} เป็น {employee.get_title()} แผนก {employee.department.value}")
    
    def monthly_payroll(self):
        print(f"\n{'='*60}")
        print(f"{'รายงานค่าตอบแทนรายเดือน: ' + self.company:^60}")
        print(f"{'='*60}")
        total = 0
        for emp in self.employees:
            pay = emp.calculate_pay()
            total += pay
            print(f"{emp.name:20} | {emp.get_title():20} | {pay:>12,.2f} บาท")
        print(f"{'-'*60}")
        print(f"{'รวมทั้งสิ้น':>43}: {total:>12,.2f} บาท")
        print(f"{'='*60}")
        return total
    
    def annual_bonus_report(self):
        print(f"\nรายงานโบนัสประจำปี:")
        for emp in self.employees:
            bonus = emp.annual_bonus()
            if bonus > 0:
                print(f"  {emp.name:20}: {bonus:>12,.2f} บาท")
    
    def get_by_department(self, dept):
        return [e for e in self.employees if e.department == dept]


hr = HRSystem("บริษัท Python จำกัด")

hr.hire(FullTimeEmployee("สมชาย ใจดี", Department.ENGINEERING, date(2020, 1, 15), 45000))
hr.hire(FullTimeEmployee("สมหญิง มีทรัพย์", Department.IT, date(2018, 6, 1), 55000))
hr.hire(PartTimeEmployee("สมศรี ทำงาน", Department.MARKETING, date(2023, 3, 1), 200, 20))
hr.hire(Contractor("สมปอง รับเหมา", Department.IT, date(2024, 1, 1), 15000, 2))

hr.monthly_payroll()
hr.annual_bonus_report()

it_team = hr.get_by_department(Department.IT)
print(f"\nทีม IT ({len(it_team)} คน):")
for emp in it_team:
    print(f"  {emp}")
```

---

## แบบฝึกหัด

### ข้อที่ 1: Library System Hierarchy
สร้าง class hierarchy:
- `LibraryItem(ABC)` - abstract base class
  - `Book(LibraryItem)`
  - `Magazine(LibraryItem)`
  - `DVD(LibraryItem)`
- แต่ละ class ต้อง implement `checkout()`, `return_item()`, `get_info()`

```python
# เฉลยข้อที่ 1
from abc import ABC, abstractmethod
from datetime import date, timedelta

class LibraryItem(ABC):
    def __init__(self, title, item_id, location):
        self.title = title
        self.item_id = item_id
        self.location = location
        self.available = True
        self._borrower = None
        self._due_date = None
    
    @property
    @abstractmethod
    def loan_period_days(self) -> int:
        """จำนวนวันที่ยืมได้"""
        pass
    
    @abstractmethod
    def get_info(self):
        """แสดงข้อมูล"""
        pass
    
    def checkout(self, borrower_name):
        if not self.available:
            raise RuntimeError(f"'{self.title}' ถูกยืมแล้ว")
        self.available = False
        self._borrower = borrower_name
        self._due_date = date.today() + timedelta(days=self.loan_period_days)
        print(f"'{self.title}' ยืมโดย {borrower_name} | กำหนดคืน: {self._due_date}")
    
    def return_item(self):
        if self.available:
            raise RuntimeError(f"'{self.title}' ไม่ได้ถูกยืม")
        
        today = date.today()
        overdue = (today - self._due_date).days if today > self._due_date else 0
        fine = overdue * self.calculate_fine_per_day()
        
        borrower = self._borrower
        self.available = True
        self._borrower = None
        self._due_date = None
        
        print(f"'{self.title}' คืนโดย {borrower}")
        if fine > 0:
            print(f"ค่าปรับ: {fine:.2f} บาท ({overdue} วัน)")
        return fine
    
    def calculate_fine_per_day(self):
        return 5  # Default 5 บาท/วัน
    
    def __str__(self):
        status = "ว่าง" if self.available else f"ยืมโดย {self._borrower}"
        return f"[{self.item_id}] {self.title} ({status})"


class Book(LibraryItem):
    def __init__(self, title, item_id, location, author, isbn, pages):
        super().__init__(title, item_id, location)
        self.author = author
        self.isbn = isbn
        self.pages = pages
    
    @property
    def loan_period_days(self):
        return 14  # หนังสือยืมได้ 2 สัปดาห์
    
    def get_info(self):
        print(f"📚 หนังสือ: {self.title}")
        print(f"   ผู้แต่ง: {self.author} | ISBN: {self.isbn}")
        print(f"   หน้า: {self.pages} | ตำแหน่ง: {self.location}")


class Magazine(LibraryItem):
    def __init__(self, title, item_id, location, issue_number, month, year):
        super().__init__(title, item_id, location)
        self.issue_number = issue_number
        self.month = month
        self.year = year
    
    @property
    def loan_period_days(self):
        return 7  # นิตยสารยืมได้ 1 สัปดาห์
    
    def calculate_fine_per_day(self):
        return 10  # ค่าปรับแพงกว่าหนังสือ
    
    def get_info(self):
        print(f"📰 นิตยสาร: {self.title}")
        print(f"   ฉบับที่: {self.issue_number} | เดือน: {self.month}/{self.year}")
        print(f"   ตำแหน่ง: {self.location}")


class DVD(LibraryItem):
    def __init__(self, title, item_id, location, director, duration_min, rating):
        super().__init__(title, item_id, location)
        self.director = director
        self.duration_min = duration_min
        self.rating = rating
    
    @property
    def loan_period_days(self):
        return 3  # DVD ยืมได้ 3 วัน
    
    def calculate_fine_per_day(self):
        return 20  # ค่าปรับ DVD แพงที่สุด
    
    def get_info(self):
        print(f"🎬 DVD: {self.title}")
        print(f"   ผู้กำกับ: {self.director} | ความยาว: {self.duration_min} นาที")
        print(f"   เรตติ้ง: {self.rating} | ตำแหน่ง: {self.location}")


# ทดสอบ
items = [
    Book("Python Programming", "B001", "A-101", "สมชาย ใจดี", "978-0000000001", 500),
    Magazine("National Geographic", "M001", "M-01", 456, "January", 2024),
    DVD("The Matrix", "D001", "D-01", "Wachowskis", 136, "R"),
]

for item in items:
    item.get_info()
    print()

items[0].checkout("สมหญิง")
items[1].checkout("สมศรี")
items[0].return_item()
```

### ข้อที่ 2 - 10: โจทย์ฝึกหัดเพิ่มเติม

```python
# ข้อที่ 2: Vehicle Hierarchy
# - Vehicle(ABC) -> Car, Truck, Motorcycle, Bicycle
# - แต่ละ class มี fuel_cost_per_km() ต่างกัน

# ข้อที่ 3: Food Hierarchy พร้อม Mixin
# - Food(ABC) -> VegetarianFood, NonVegetarianFood
# - NutritionalMixin เพิ่ม calories, protein, carbs
# - HealthyMixin มี health_score()

# ข้อที่ 4: Payment System
# - PaymentMethod(ABC) -> CreditCard, BankTransfer, QRCode
# - process_payment(), refund(), validate()

# ข้อที่ 5: Game Character Hierarchy
# - Character(ABC) -> Warrior, Mage, Archer
# - CombatMixin, MagicMixin, StealthMixin

# ข้อที่ 6: Social Media Post
# - Post(ABC) -> TextPost, ImagePost, VideoPost, StoryPost
# - ReactableMixin (like, love, laugh)
# - ShareableMixin

# ข้อที่ 7: ระบบ Permission
# - Permission(ABC) -> ReadPermission, WritePermission, AdminPermission
# - User class ที่มี permissions หลายอย่าง

# ข้อที่ 8: DataSource Hierarchy
# - DataSource(ABC) -> CSVSource, JSONSource, DatabaseSource
# - CacheMixin เพิ่ม caching
# - LoggingMixin เพิ่ม logging

# ข้อที่ 9: สร้าง Class hierarchy สำหรับ Menu
# - MenuItem(ABC) -> FoodItem, DrinkItem, ComboItem
# - DiscountableMixin
# - AvailabilityMixin

# ข้อที่ 10: UI Component Hierarchy
# - Widget(ABC) -> Button, TextField, Checkbox, Dropdown
# - ClickableMixin, FocusableMixin, ValidatableMixin
```

#### เฉลยข้อที่ 2: Vehicle Hierarchy

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    """Abstract base class สำหรับยานพาหนะ"""
    
    def __init__(self, make, model, year, max_speed_kmh):
        self.make = make
        self.model = model
        self.year = year
        self.max_speed_kmh = max_speed_kmh
        self._current_speed = 0
    
    @abstractmethod
    def fuel_cost_per_km(self) -> float:
        """ค่าใช้จ่ายต่อกิโลเมตร"""
        pass
    
    @abstractmethod
    def vehicle_type(self) -> str:
        pass
    
    def accelerate(self, speed):
        self._current_speed = min(speed, self.max_speed_kmh)
        print(f"{self.make} {self.model} เร่งความเร็วเป็น {self._current_speed} กม./ชม.")
    
    def brake(self):
        self._current_speed = 0
        print(f"{self.make} {self.model} หยุดแล้ว")
    
    def travel_cost(self, distance_km):
        return distance_km * self.fuel_cost_per_km()
    
    def __str__(self):
        return (f"{self.vehicle_type()}: {self.year} {self.make} {self.model} "
                f"(ความเร็วสูงสุด: {self.max_speed_kmh} กม./ชม.)")


class Car(Vehicle):
    def __init__(self, make, model, year, max_speed, num_doors, fuel_efficiency):
        super().__init__(make, model, year, max_speed)
        self.num_doors = num_doors
        self.fuel_efficiency = fuel_efficiency  # km/liter
        self.fuel_price = 40  # บาท/ลิตร
    
    def fuel_cost_per_km(self):
        return self.fuel_price / self.fuel_efficiency
    
    def vehicle_type(self):
        return "รถยนต์"


class ElectricCar(Car):
    def __init__(self, make, model, year, max_speed, num_doors, kwh_per_100km):
        super().__init__(make, model, year, max_speed, num_doors, 1)
        self.kwh_per_100km = kwh_per_100km
        self.electricity_price = 5  # บาท/kWh
    
    def fuel_cost_per_km(self):
        return (self.kwh_per_100km / 100) * self.electricity_price
    
    def vehicle_type(self):
        return "รถไฟฟ้า"


class Truck(Vehicle):
    def __init__(self, make, model, year, max_speed, payload_tons, fuel_efficiency):
        super().__init__(make, model, year, max_speed)
        self.payload_tons = payload_tons
        self.fuel_efficiency = fuel_efficiency  # km/liter
        self.fuel_price = 32  # บาท/ลิตร (ดีเซล)
    
    def fuel_cost_per_km(self):
        return self.fuel_price / self.fuel_efficiency
    
    def vehicle_type(self):
        return "รถบรรทุก"


class Bicycle(Vehicle):
    def __init__(self, make, model, year, max_speed, num_gears):
        super().__init__(make, model, year, max_speed)
        self.num_gears = num_gears
    
    def fuel_cost_per_km(self):
        return 0  # ปั่นฟรี!
    
    def vehicle_type(self):
        return "จักรยาน"


# ทดสอบ
vehicles = [
    Car("Toyota", "Camry", 2022, 200, 4, 15),
    ElectricCar("Tesla", "Model 3", 2023, 250, 4, 14),
    Truck("Isuzu", "D-Max", 2021, 160, 1, 10),
    Bicycle("Trek", "FX3", 2023, 30, 21),
]

distance = 100  # กม.

print(f"ค่าใช้จ่ายในการเดินทาง {distance} กม.:")
for v in vehicles:
    cost = v.travel_cost(distance)
    print(f"  {v.make} {v.model}: {cost:>8,.2f} บาท")
```

---

## สรุป Part 22

| แนวคิด | Syntax | ใช้เมื่อ |
|--------|--------|---------|
| Single inheritance | `class Child(Parent)` | class มี parent class เดียว |
| Multiple inheritance | `class Child(A, B, C)` | ต้องการ functionality จากหลาย class |
| super() | `super().method()` | เรียก parent's method |
| MRO | `Class.__mro__` | ดูลำดับการ resolve method |
| Abstract class | `class A(ABC)` | กำหนด interface ให้ subclass |
| @abstractmethod | decorator | บังคับให้ subclass implement |
| isinstance() | ฟังก์ชัน | ตรวจ type ของ object |
| issubclass() | ฟังก์ชัน | ตรวจ relationship ของ classes |
| Mixin | class พิเศษ | เพิ่ม functionality โดยไม่ใช่ parent หลัก |

**ต่อไป**: Part 23 - OOP Polymorphism & Duck Typing
