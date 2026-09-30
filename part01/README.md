# Part 01: Introduction to Python & Environment Setup

## สารบัญ (Table of Contents)

1. [Python คืออะไร?](#python-คืออะไร)
2. [ประวัติของ Python](#ประวัติของ-python)
3. [ทำไมต้องเรียน Python?](#ทำไมต้องเรียน-python)
4. [การติดตั้ง Python บน Windows](#การติดตั้ง-python-บน-windows)
5. [การติดตั้ง Python บน macOS](#การติดตั้ง-python-บน-macos)
6. [การติดตั้ง Python บน Linux](#การติดตั้ง-python-บน-linux)
7. [การติดตั้ง VS Code และ Extensions](#การติดตั้ง-vs-code-และ-extensions)
8. [Python REPL และการใช้งาน](#python-repl-และการใช้งาน)
9. [โปรแกรม Hello World แรก](#โปรแกรม-hello-world-แรก)
10. [การรันไฟล์ .py](#การรันไฟล์-py)
11. [Interactive Mode vs Script Mode](#interactive-mode-vs-script-mode)
12. [การใช้ pip และ Package Manager](#การใช้-pip-และ-package-manager)
13. [โครงสร้างโปรเจกต์ Python](#โครงสร้างโปรเจกต์-python)
14. [ตัวอย่างโค้ด](#ตัวอย่างโค้ด)
15. [แบบฝึกหัด](#แบบฝึกหัด)
16. [เฉลยแบบฝึกหัด](#เฉลยแบบฝึกหัด)

---

## Python คืออะไร?

Python เป็นภาษาโปรแกรมมิ่งระดับสูง (high-level programming language) ที่เน้นความอ่านง่ายและเขียนง่าย ถูกออกแบบมาให้โค้ดมีความชัดเจนและตรงไปตรงมา Python ใช้การเยื้อง (indentation) แทนวงเล็บปีกกาในการกำหนดโครงสร้างโค้ด ทำให้โค้ดอ่านเหมือนภาษาอังกฤษธรรมดา

### คุณสมบัติหลักของ Python

| คุณสมบัติ | คำอธิบาย |
|-----------|----------|
| **Interpreted** | Python แปลและรันโค้ดทีละบรรทัด ไม่ต้อง compile ก่อน |
| **Dynamically Typed** | ไม่ต้องประกาศชนิดตัวแปรล่วงหน้า |
| **Object-Oriented** | รองรับ OOP อย่างเต็มรูปแบบ |
| **Multi-paradigm** | รองรับหลายรูปแบบการเขียนโปรแกรม |
| **Cross-platform** | รันได้บน Windows, macOS, Linux |
| **Large Standard Library** | มี library มาตรฐานมากมาย |
| **Open Source** | ฟรี และ open source |

---

## ประวัติของ Python

Python ถูกสร้างโดย **Guido van Rossum** นักโปรแกรมเมอร์ชาวดัตช์ ในช่วงปลายทศวรรษ 1980

### ไทม์ไลน์สำคัญ

```
1989  - Guido van Rossum เริ่มพัฒนา Python
1991  - Python 0.9.0 เปิดตัวครั้งแรก
1994  - Python 1.0 ออกมา (รองรับ lambda, map, filter, reduce)
2000  - Python 2.0 ออกมา (เพิ่ม list comprehension, garbage collector)
2008  - Python 3.0 ออกมา (แก้ปัญหาหลายอย่างจาก Python 2)
2020  - Python 2 หยุดรับการสนับสนุน (End of Life)
2023  - Python 3.12 ออกมา พร้อมประสิทธิภาพที่ดีขึ้นมาก
2024  - Python 3.13 เปิดตัว
```

### ชื่อ "Python" มาจากไหน?

ชื่อ Python ไม่ได้มาจากงู! แต่มาจากรายการทีวีอังกฤษ **"Monty Python's Flying Circus"** ที่ Guido van Rossum ชื่นชอบ

---

## ทำไมต้องเรียน Python?

### 1. ความนิยมและตลาดงาน

Python ครองอันดับต้นๆ ในภาษาโปรแกรมมิ่งที่นิยมใช้มากที่สุด (TIOBE Index, Stack Overflow Survey) มีงานรองรับมากมายในสาขา:
- Data Science / Machine Learning
- Web Development (Django, Flask, FastAPI)
- Automation / Scripting
- Cybersecurity
- DevOps / Cloud
- Scientific Computing
- Game Development

### 2. ง่ายต่อการเรียนรู้

```python
# Python - ง่าย อ่านง่าย
name = "สมชาย"
age = 25
print(f"สวัสดี! ฉันชื่อ {name} อายุ {age} ปี")

# Java - ซับซ้อนกว่ามาก (ทำสิ่งเดียวกัน)
# public class Hello {
#     public static void main(String[] args) {
#         String name = "สมชาย";
#         int age = 25;
#         System.out.println("สวัสดี! ฉันชื่อ " + name + " อายุ " + age + " ปี");
#     }
# }
```

### 3. Library และ Framework ที่หลากหลาย

```
Data Science:    NumPy, Pandas, Matplotlib, Seaborn
ML/AI:           TensorFlow, PyTorch, Scikit-learn, Keras
Web Backend:     Django, Flask, FastAPI, Tornado
Web Scraping:    BeautifulSoup, Scrapy, Selenium
Automation:      Selenium, PyAutoGUI, Paramiko
Database:        SQLAlchemy, psycopg2, PyMongo
API:             Requests, httpx, aiohttp
```

### 4. ชุมชนขนาดใหญ่

- Stack Overflow: คำถาม Python ล้านกว่าข้อ
- PyPI (Python Package Index): package มากกว่า 400,000 รายการ
- GitHub: repository Python หลายล้าน

---

## การติดตั้ง Python บน Windows

### วิธีที่ 1: ติดตั้งจาก python.org (แนะนำ)

**ขั้นตอนที่ 1:** ไปที่ https://www.python.org/downloads/

**ขั้นตอนที่ 2:** คลิก "Download Python 3.x.x" (เวอร์ชันล่าสุด)

**ขั้นตอนที่ 3:** รันไฟล์ installer ที่ดาวน์โหลดมา

**ขั้นตอนที่ 4 (สำคัญมาก!):** ติ๊กช่อง **"Add Python to PATH"** ก่อนกด Install

```
[✓] Install launcher for all users (recommended)
[✓] Add Python to PATH   <--- ต้องติ๊กอันนี้!
```

**ขั้นตอนที่ 5:** คลิก "Install Now" และรอให้ติดตั้งเสร็จ

**ขั้นตอนที่ 6:** ตรวจสอบการติดตั้ง เปิด Command Prompt (cmd) และพิมพ์:

```cmd
python --version
pip --version
```

ควรได้ผลลัพธ์ประมาณ:
```
Python 3.12.0
pip 23.x.x from C:\Users\...\Python312\lib\site-packages\pip (python 3.12)
```

### วิธีที่ 2: ติดตั้งผ่าน Windows Store

1. เปิด Microsoft Store
2. ค้นหา "Python"
3. เลือก Python 3.x จาก Python Software Foundation
4. คลิก "Get" หรือ "Install"

### วิธีที่ 3: ติดตั้งผ่าน winget (Windows Package Manager)

เปิด PowerShell และรัน:

```powershell
winget install Python.Python.3.12
```

### การแก้ปัญหาทั่วไปบน Windows

**ปัญหา:** `'python' is not recognized as an internal or external command`

**วิธีแก้:** เพิ่ม Python ใน PATH ด้วยตนเอง:
1. กด `Windows + R` พิมพ์ `sysdm.cpl`
2. ไปที่ "Advanced" > "Environment Variables"
3. ใน "System variables" ค้นหา "Path" แล้วคลิก "Edit"
4. เพิ่ม path ของ Python: `C:\Users\[username]\AppData\Local\Programs\Python\Python312\`
5. เพิ่ม path ของ Scripts: `C:\Users\[username]\AppData\Local\Programs\Python\Python312\Scripts\`

---

## การติดตั้ง Python บน macOS

### วิธีที่ 1: ติดตั้งผ่าน Homebrew (แนะนำ)

**ขั้นตอนที่ 1:** ติดตั้ง Homebrew ก่อน (ถ้ายังไม่มี):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**ขั้นตอนที่ 2:** ติดตั้ง Python:

```bash
brew install python
```

**ขั้นตอนที่ 3:** ตรวจสอบ:

```bash
python3 --version
pip3 --version
```

### วิธีที่ 2: ติดตั้งจาก python.org

1. ไปที่ https://www.python.org/downloads/macos/
2. ดาวน์โหลด macOS installer
3. เปิดไฟล์ .pkg และทำตามขั้นตอน
4. ตรวจสอบการติดตั้งใน Terminal

### วิธีที่ 3: ใช้ pyenv (จัดการหลาย Python version)

```bash
# ติดตั้ง pyenv
brew install pyenv

# เพิ่มใน ~/.zshrc หรือ ~/.bash_profile
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

# โหลด config ใหม่
source ~/.zshrc

# ดู Python versions ที่ติดตั้งได้
pyenv install --list | grep "3.12"

# ติดตั้ง Python 3.12
pyenv install 3.12.0

# ตั้งเป็น global version
pyenv global 3.12.0

# ตรวจสอบ
python --version
```

---

## การติดตั้ง Python บน Linux

### Ubuntu / Debian

Python มักติดตั้งมาแล้ว ตรวจสอบก่อน:

```bash
python3 --version
```

**ถ้ายังไม่มี หรือต้องการ update:**

```bash
# อัปเดต package list
sudo apt update

# ติดตั้ง Python 3
sudo apt install python3

# ติดตั้ง pip
sudo apt install python3-pip

# ติดตั้ง python3-venv (สำหรับ virtual environment)
sudo apt install python3-venv

# ตรวจสอบ
python3 --version
pip3 --version
```

**ติดตั้งเวอร์ชันเฉพาะผ่าน deadsnakes PPA:**

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12
sudo apt install python3.12-venv
sudo apt install python3.12-dev
```

### Fedora / CentOS / RHEL

```bash
# Fedora
sudo dnf install python3

# CentOS/RHEL 8+
sudo dnf install python38

# ตรวจสอบ
python3 --version
```

### Arch Linux

```bash
sudo pacman -S python python-pip
```

### การตั้งค่า alias (Linux/macOS)

เพิ่มใน `~/.bashrc` หรือ `~/.zshrc`:

```bash
alias python=python3
alias pip=pip3
```

---

## การติดตั้ง VS Code และ Extensions

### ติดตั้ง VS Code

1. ไปที่ https://code.visualstudio.com/
2. ดาวน์โหลดสำหรับ OS ของคุณ
3. ติดตั้งตามขั้นตอนปกติ

### Extensions ที่จำเป็นสำหรับ Python

**1. Python (โดย Microsoft) - จำเป็นที่สุด**
- Extension ID: `ms-python.python`
- รองรับ IntelliSense, linting, debugging, testing
- รองรับ Jupyter Notebooks

```
วิธีติดตั้ง:
1. กด Ctrl+Shift+X (Extensions panel)
2. ค้นหา "Python"
3. เลือก "Python" โดย Microsoft
4. คลิก Install
```

**2. Pylance - Python Language Server**
- Extension ID: `ms-python.vscode-pylance`
- ให้ IntelliSense ที่ดีขึ้น, type checking

**3. Python Debugger**
- Extension ID: `ms-python.debugpy`
- ใช้ debug Python code

**4. Black Formatter (Code Formatter)**
- Extension ID: `ms-python.black-formatter`
- จัดรูปแบบโค้ดอัตโนมัติ

**5. Ruff (Linter)**
- Extension ID: `charliermarsh.ruff`
- ตรวจสอบ code style

**6. autoDocstring**
- Extension ID: `njpwerner.autodocstring`
- สร้าง docstring อัตโนมัติ

**7. GitLens**
- Extension ID: `eamodio.gitlens`
- ดู git history inline

### การตั้งค่า VS Code สำหรับ Python

สร้างไฟล์ `.vscode/settings.json` ในโปรเจกต์:

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "ms-python.black-formatter",
    "[python]": {
        "editor.defaultFormatter": "ms-python.black-formatter",
        "editor.formatOnSave": true
    },
    "python.linting.enabled": true,
    "python.linting.ruffEnabled": true,
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "files.trimTrailingWhitespace": true
}
```

### Keyboard Shortcuts ที่ใช้บ่อย

| Shortcut | การทำงาน |
|----------|---------|
| `F5` | Debug/Run |
| `Ctrl+F5` | Run without debugging |
| `Shift+Enter` | Run selection in terminal |
| `Ctrl+Shift+P` | Command palette |
| `Ctrl+` ` | เปิด terminal |
| `Ctrl+/` | Toggle comment |
| `Alt+Shift+F` | Format document |

---

## Python REPL และการใช้งาน

### REPL คืออะไร?

**REPL** ย่อมาจาก **Read-Eval-Print Loop** คือ interactive mode ของ Python ที่รับ input, ประมวลผล, แสดงผล แล้ววนซ้ำ

### เปิด Python REPL

```bash
# บน Windows
python

# บน macOS/Linux
python3
```

จะเห็น prompt แบบนี้:
```
Python 3.12.0 (main, Oct  2 2023, 15:37:07) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

### การใช้งาน REPL

```python
>>> 2 + 2
4
>>> "Hello" + " " + "World"
'Hello World'
>>> name = "Python"
>>> print(f"Hello {name}!")
Hello Python!
>>> type(42)
<class 'int'>
>>> help(print)
Help on built-in function print in module builtins:
...
>>> exit()  # หรือกด Ctrl+D เพื่อออก
```

### REPL Commands พิเศษ

```python
>>> help()          # เข้าสู่ help mode
>>> help(str)       # ดู help สำหรับ str class
>>> dir()           # ดู names ใน current scope
>>> dir(str)        # ดู methods ของ str
>>> type(42)        # ตรวจสอบ type
>>> id(42)          # ดู memory address
```

### IPython - REPL ที่ดีขึ้น

```bash
# ติดตั้ง IPython
pip install ipython

# รัน
ipython
```

IPython มีฟีเจอร์พิเศษ:

```python
In [1]: %timeit [x**2 for x in range(1000)]   # วัดเวลา
In [2]: %history                                 # ดู command history
In [3]: %run script.py                           # รัน script
In [4]: ?str.upper                               # ดู documentation
In [5]: ??str.upper                              # ดู source code
```

---

## โปรแกรม Hello World แรก

### วิธีที่ 1: ผ่าน REPL

```python
>>> print("Hello, World!")
Hello, World!
```

### วิธีที่ 2: สร้างไฟล์ .py

สร้างไฟล์ `hello.py`:

```python
# hello.py - โปรแกรม Python แรกของฉัน
print("Hello, World!")
print("สวัสดีโลก!")
print("ยินดีต้อนรับสู่ Python")
```

รันด้วย:

```bash
python hello.py
# หรือ
python3 hello.py
```

ผลลัพธ์:
```
Hello, World!
สวัสดีโลก!
ยินดีต้อนรับสู่ Python
```

### Hello World ที่สมบูรณ์กว่า

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
โปรแกรม Hello World ที่สมบูรณ์
สร้างวันที่: 2024-01-01
ผู้สร้าง: Python Learner
"""

def main():
    """ฟังก์ชันหลักของโปรแกรม"""
    # แสดงข้อความทักทาย
    print("=" * 40)
    print("Hello, World!")
    print("สวัสดีโลก!")
    print("=" * 40)
    
    # รับ input จากผู้ใช้
    name = input("กรุณาพิมพ์ชื่อของคุณ: ")
    print(f"สวัสดี {name}! ยินดีต้อนรับสู่ Python")

if __name__ == "__main__":
    main()
```

---

## การรันไฟล์ .py

### การรันพื้นฐาน

```bash
# รูปแบบทั่วไป
python filename.py
python3 filename.py

# ตัวอย่าง
python hello.py
python3 hello.py
```

### การรันพร้อม Arguments

```python
# script.py
import sys

print(f"จำนวน arguments: {len(sys.argv)}")
print(f"Arguments: {sys.argv}")

for i, arg in enumerate(sys.argv):
    print(f"  argv[{i}] = {arg}")
```

```bash
python script.py arg1 arg2 arg3
# Output:
# จำนวน arguments: 4
# Arguments: ['script.py', 'arg1', 'arg2', 'arg3']
#   argv[0] = script.py
#   argv[1] = arg1
#   argv[2] = arg2
#   argv[3] = arg3
```

### การรันใน VS Code

1. เปิดไฟล์ .py
2. กด **F5** เพื่อ debug หรือ **Ctrl+F5** เพื่อรันปกติ
3. หรือคลิกขวา > "Run Python File in Terminal"
4. หรือคลิกปุ่ม ▶ (Play) มุมบนขวา

### การรันเป็น Module

```bash
# รัน module ด้วย -m flag
python -m module_name

# ตัวอย่าง
python -m http.server 8000    # เปิด web server
python -m pip install package  # ติดตั้ง package
python -m venv .venv           # สร้าง virtual environment
```

### Shebang Line (Linux/macOS)

```python
#!/usr/bin/env python3
# บรรทัดแรกคือ shebang - บอก OS ว่าใช้ interpreter ไหนรันไฟล์นี้

print("Hello from Python script!")
```

```bash
# ต้องให้ permission execute ก่อน
chmod +x script.py

# รันได้โดยไม่ต้องพิมพ์ python3
./script.py
```

---

## Interactive Mode vs Script Mode

### Interactive Mode (REPL)

```
ข้อดี:
- ทดสอบโค้ดได้ทันที
- ดูผลลัพธ์ได้เลย
- เหมาะสำหรับทดลองและเรียนรู้
- มี auto-complete (ใน IPython)

ข้อเสีย:
- ไม่ save code
- ต้องพิมพ์ทุกครั้ง
- ไม่เหมาะสำหรับโปรแกรมซับซ้อน
```

```python
# Interactive Mode ตัวอย่าง
>>> x = 10
>>> y = 20
>>> x + y
30
>>> "Python" * 3
'PythonPythonPython'
```

### Script Mode (ไฟล์ .py)

```
ข้อดี:
- บันทึกโค้ดไว้ได้
- แชร์กับคนอื่นได้
- รันซ้ำได้หลายครั้ง
- เหมาะสำหรับโปรแกรมจริง
- สามารถ import ใน module อื่นได้

ข้อเสีย:
- ต้องสร้างไฟล์ก่อน
- ต้องรันทั้งไฟล์ (หรือบางส่วน)
```

```python
# script_mode.py
# Script Mode ตัวอย่าง

def calculate_area(width, height):
    """คำนวณพื้นที่สี่เหลี่ยม"""
    return width * height

def calculate_perimeter(width, height):
    """คำนวณเส้นรอบวงสี่เหลี่ยม"""
    return 2 * (width + height)

# โค้ดหลัก
width = 5
height = 3

area = calculate_area(width, height)
perimeter = calculate_perimeter(width, height)

print(f"กว้าง: {width}, สูง: {height}")
print(f"พื้นที่: {area}")
print(f"เส้นรอบวง: {perimeter}")
```

### เปรียบเทียบ

| คุณสมบัติ | Interactive Mode | Script Mode |
|-----------|-----------------|-------------|
| การแสดงผล | อัตโนมัติ | ต้องใช้ print() |
| การบันทึก | ไม่บันทึก | บันทึกในไฟล์ |
| การใช้งาน | ทดสอบ/เรียนรู้ | โปรแกรมจริง |
| ขนาดโค้ด | สั้น | ยาวได้ |

---

## การใช้ pip และ Package Manager

### pip คืออะไร?

**pip** (Pip Installs Packages) คือ package manager มาตรฐานของ Python ใช้ติดตั้ง, อัปเดต, ลบ Python packages จาก PyPI (Python Package Index)

### คำสั่ง pip พื้นฐาน

```bash
# ตรวจสอบ pip version
pip --version
pip3 --version

# ติดตั้ง package
pip install package_name
pip install requests
pip install numpy pandas matplotlib

# ติดตั้งเวอร์ชันเฉพาะ
pip install requests==2.28.0
pip install requests>=2.0.0

# อัปเดต package
pip install --upgrade package_name
pip install -U requests

# ลบ package
pip uninstall package_name
pip uninstall requests

# ดู packages ที่ติดตั้งไว้
pip list
pip freeze

# ดูรายละเอียด package
pip show requests

# ค้นหา package
pip search keyword    # (deprecated ใน pip เวอร์ชันใหม่)
```

### requirements.txt

ไฟล์ที่ระบุ dependencies ของโปรเจกต์:

```bash
# สร้าง requirements.txt จาก packages ที่ติดตั้ง
pip freeze > requirements.txt

# ติดตั้ง packages จาก requirements.txt
pip install -r requirements.txt
```

ตัวอย่าง `requirements.txt`:

```
requests==2.31.0
numpy>=1.24.0
pandas~=2.0.0
matplotlib
Flask>=3.0.0
```

### Virtual Environment (venv)

Virtual Environment คือ Python environment แยกสำหรับแต่ละโปรเจกต์ ป้องกัน package conflicts

```bash
# สร้าง virtual environment
python -m venv .venv
python3 -m venv .venv

# Activate (Windows)
.venv\Scripts\activate

# Activate (macOS/Linux)
source .venv/bin/activate

# ตรวจสอบว่า activate แล้ว (prompt จะเปลี่ยน)
(.venv) $

# ติดตั้ง packages ใน venv
pip install requests numpy pandas

# Deactivate
deactivate
```

### pipenv (Alternative)

```bash
# ติดตั้ง pipenv
pip install pipenv

# สร้าง environment และติดตั้ง package
pipenv install requests

# activate environment
pipenv shell

# รัน script ใน environment
pipenv run python script.py
```

### conda (Anaconda)

เหมาะสำหรับ Data Science:

```bash
# ติดตั้ง package
conda install numpy pandas matplotlib

# สร้าง environment
conda create -n myenv python=3.12

# activate
conda activate myenv

# deactivate
conda deactivate
```

---

## โครงสร้างโปรเจกต์ Python

### โครงสร้างพื้นฐาน

```
my_project/
├── .venv/                  # Virtual environment (ไม่ commit ใน git)
├── .gitignore              # ไฟล์ที่ git ไม่ต้อง track
├── README.md               # เอกสารอธิบายโปรเจกต์
├── requirements.txt        # Dependencies
├── setup.py / pyproject.toml  # Package configuration
├── main.py                 # จุดเริ่มต้นโปรแกรม
├── src/                    # Source code
│   └── mypackage/
│       ├── __init__.py
│       ├── module1.py
│       └── module2.py
├── tests/                  # Test files
│   ├── __init__.py
│   ├── test_module1.py
│   └── test_module2.py
└── docs/                   # Documentation
    └── index.md
```

### โครงสร้างสำหรับ Script อย่างง่าย

```
my_scripts/
├── script1.py
├── script2.py
├── utils.py
└── requirements.txt
```

### โครงสร้างสำหรับ Web App (Flask)

```
web_app/
├── app/
│   ├── __init__.py
│   ├── routes/
│   │   ├── __init__.py
│   │   └── main.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── static/
│   │   ├── css/
│   │   └── js/
│   └── templates/
│       └── index.html
├── tests/
├── .env
├── config.py
├── requirements.txt
└── run.py
```

### ไฟล์ .gitignore สำหรับ Python

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# Virtual Environment
.venv/
venv/
ENV/
env/

# Distribution / packaging
dist/
build/
*.egg-info/

# Testing
.pytest_cache/
.coverage
htmlcov/

# IDEs
.vscode/
.idea/
*.swp

# Environment variables
.env
.env.local

# Jupyter
.ipynb_checkpoints/
*.ipynb
```

---

## ตัวอย่างโค้ด

### ตัวอย่างที่ 1: Hello World พื้นฐาน

```python
# ตัวอย่างที่ 1: Hello World พื้นฐาน
print("Hello, World!")
print("สวัสดีโลก!")
```

### ตัวอย่างที่ 2: การรับ Input

```python
# ตัวอย่างที่ 2: รับ input จากผู้ใช้
name = input("ชื่อของคุณคือ? ")
print(f"สวัสดี {name}!")
```

### ตัวอย่างที่ 3: การคำนวณพื้นฐาน

```python
# ตัวอย่างที่ 3: การคำนวณพื้นฐาน
a = 10
b = 3

print(f"a = {a}, b = {b}")
print(f"a + b = {a + b}")       # บวก
print(f"a - b = {a - b}")       # ลบ
print(f"a * b = {a * b}")       # คูณ
print(f"a / b = {a / b}")       # หาร (ได้ float)
print(f"a // b = {a // b}")     # หารเอาจำนวนเต็ม
print(f"a % b = {a % b}")       # modulo (เศษ)
print(f"a ** b = {a ** b}")     # ยกกำลัง
```

### ตัวอย่างที่ 4: ตัวแปรและ Type

```python
# ตัวอย่างที่ 4: ตัวแปรและ Type
integer_num = 42
float_num = 3.14
text = "Hello Python"
is_active = True
nothing = None

print(type(integer_num))    # <class 'int'>
print(type(float_num))      # <class 'float'>
print(type(text))           # <class 'str'>
print(type(is_active))      # <class 'bool'>
print(type(nothing))        # <class 'NoneType'>
```

### ตัวอย่างที่ 5: String Operations

```python
# ตัวอย่างที่ 5: String Operations
name = "Python Programming"

print(name.upper())           # PYTHON PROGRAMMING
print(name.lower())           # python programming
print(name.replace("Python", "Java"))  # Java Programming
print(len(name))              # 18
print(name[0])                # P (character แรก)
print(name[-1])               # g (character สุดท้าย)
print(name[0:6])              # Python (slicing)
```

### ตัวอย่างที่ 6: List พื้นฐาน

```python
# ตัวอย่างที่ 6: List พื้นฐาน
fruits = ["apple", "banana", "cherry", "date"]

print(fruits)           # แสดง list ทั้งหมด
print(fruits[0])        # apple
print(fruits[-1])       # date
print(len(fruits))      # 4

fruits.append("elderberry")   # เพิ่มท้าย
print(fruits)

fruits.remove("banana")       # ลบ element
print(fruits)
```

### ตัวอย่างที่ 7: การวนซ้ำ (Loop)

```python
# ตัวอย่างที่ 7: for loop พื้นฐาน
for i in range(5):
    print(f"รอบที่ {i + 1}")

print("\n--- วนซ้ำใน list ---")
colors = ["red", "green", "blue"]
for color in colors:
    print(f"สี: {color}")
```

### ตัวอย่างที่ 8: เงื่อนไข (Conditions)

```python
# ตัวอย่างที่ 8: if-elif-else
age = 20

if age < 13:
    print("เด็ก")
elif age < 18:
    print("วัยรุ่น")
elif age < 65:
    print("ผู้ใหญ่")
else:
    print("ผู้สูงอายุ")
```

### ตัวอย่างที่ 9: Function พื้นฐาน

```python
# ตัวอย่างที่ 9: สร้าง Function
def greet(name, greeting="สวัสดี"):
    """ฟังก์ชันทักทาย"""
    return f"{greeting}, {name}!"

# เรียกใช้ function
print(greet("สมชาย"))           # สวัสดี, สมชาย!
print(greet("Alice", "Hello"))  # Hello, Alice!
```

### ตัวอย่างที่ 10: Dictionary พื้นฐาน

```python
# ตัวอย่างที่ 10: Dictionary
person = {
    "name": "สมชาย",
    "age": 25,
    "city": "กรุงเทพ",
    "hobbies": ["อ่านหนังสือ", "เล่นกีฬา"]
}

print(person["name"])       # สมชาย
print(person.get("age"))    # 25
print(person.keys())        # dict_keys(['name', 'age', 'city', 'hobbies'])
print(person.values())      # ค่าทั้งหมด

for key, value in person.items():
    print(f"{key}: {value}")
```

### ตัวอย่างที่ 11: การ Import Module

```python
# ตัวอย่างที่ 11: การ import module
import math
import random
import datetime

# math module
print(f"pi = {math.pi}")
print(f"sqrt(16) = {math.sqrt(16)}")
print(f"floor(3.7) = {math.floor(3.7)}")

# random module
print(f"random number: {random.random()}")
print(f"random int 1-10: {random.randint(1, 10)}")

# datetime
now = datetime.datetime.now()
print(f"วันเวลาปัจจุบัน: {now}")
print(f"ปี: {now.year}, เดือน: {now.month}, วัน: {now.day}")
```

### ตัวอย่างที่ 12: Exception Handling

```python
# ตัวอย่างที่ 12: การจัดการ Exception
try:
    number = int(input("กรุณาพิมพ์ตัวเลข: "))
    result = 100 / number
    print(f"100 / {number} = {result}")
except ValueError:
    print("Error: กรุณาพิมพ์ตัวเลขเท่านั้น!")
except ZeroDivisionError:
    print("Error: ไม่สามารถหารด้วยศูนย์ได้!")
except Exception as e:
    print(f"Error ที่ไม่คาดคิด: {e}")
finally:
    print("โปรแกรมทำงานเสร็จสิ้น")
```

### ตัวอย่างที่ 13: File Operations

```python
# ตัวอย่างที่ 13: การอ่านและเขียนไฟล์
# เขียนไฟล์
with open("test.txt", "w", encoding="utf-8") as f:
    f.write("บรรทัดที่ 1\n")
    f.write("บรรทัดที่ 2\n")
    f.write("บรรทัดที่ 3\n")

# อ่านไฟล์
with open("test.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)

# อ่านทีละบรรทัด
with open("test.txt", "r", encoding="utf-8") as f:
    for line_num, line in enumerate(f, 1):
        print(f"บรรทัด {line_num}: {line.strip()}")
```

### ตัวอย่างที่ 14: List Comprehension

```python
# ตัวอย่างที่ 14: List Comprehension
# แบบ traditional
squares_traditional = []
for i in range(1, 6):
    squares_traditional.append(i ** 2)

# แบบ list comprehension (แนะนำ)
squares = [i ** 2 for i in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]

# กรองด้วย condition
even_squares = [i ** 2 for i in range(1, 11) if i % 2 == 0]
print(even_squares)  # [4, 16, 36, 64, 100]
```

### ตัวอย่างที่ 15: Class พื้นฐาน

```python
# ตัวอย่างที่ 15: Class พื้นฐาน
class Dog:
    """คลาสสำหรับแทนสุนัข"""
    
    species = "Canis lupus familiaris"  # class attribute
    
    def __init__(self, name, breed, age):
        """Constructor"""
        self.name = name      # instance attributes
        self.breed = breed
        self.age = age
    
    def bark(self):
        """สุนัขเห่า"""
        return f"{self.name} says: Woof!"
    
    def __str__(self):
        return f"Dog({self.name}, {self.breed}, {self.age} ปี)"

# สร้าง object
dog1 = Dog("Buddy", "Golden Retriever", 3)
dog2 = Dog("Max", "German Shepherd", 5)

print(dog1)           # Dog(Buddy, Golden Retriever, 3 ปี)
print(dog1.bark())    # Buddy says: Woof!
print(dog1.species)   # Canis lupus familiaris
```

### ตัวอย่างที่ 16: Generator

```python
# ตัวอย่างที่ 16: Generator (ประหยัด memory)
def fibonacci_gen(n):
    """Generator สำหรับ Fibonacci sequence"""
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

# ใช้ generator
fib = fibonacci_gen(10)
for num in fib:
    print(num, end=" ")
# 0 1 1 2 3 5 8 13 21 34
```

### ตัวอย่างที่ 17: Lambda Function

```python
# ตัวอย่างที่ 17: Lambda Function
# แทน def ด้วย lambda สำหรับ function สั้นๆ
square = lambda x: x ** 2
add = lambda x, y: x + y

print(square(5))      # 25
print(add(3, 4))      # 7

# ใช้กับ map, filter, sorted
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
sorted_nums = sorted(numbers, key=lambda x: -x)  # เรียงจากมากไปน้อย
print(sorted_nums)    # [9, 6, 5, 5, 4, 3, 3, 2, 1, 1]

# map
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)        # [6, 2, 8, 2, 10, 18, 4, 12, 10, 6]

# filter
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)          # [4, 2, 6]
```

### ตัวอย่างที่ 18: Decorator

```python
# ตัวอย่างที่ 18: Decorator พื้นฐาน
import time

def timer(func):
    """Decorator วัดเวลาการทำงานของ function"""
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} ใช้เวลา {end - start:.4f} วินาที")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.1)
    return "เสร็จแล้ว"

result = slow_function()
print(result)
# slow_function ใช้เวลา 0.1002 วินาที
# เสร็จแล้ว
```

### ตัวอย่างที่ 19: Context Manager

```python
# ตัวอย่างที่ 19: Context Manager ด้วย contextlib
from contextlib import contextmanager

@contextmanager
def managed_resource(name):
    print(f"เปิด resource: {name}")
    try:
        yield name
    finally:
        print(f"ปิด resource: {name}")

with managed_resource("Database") as db:
    print(f"กำลังใช้ {db}")
# เปิด resource: Database
# กำลังใช้ Database
# ปิด resource: Database
```

### ตัวอย่างที่ 20: Dataclasses

```python
# ตัวอย่างที่ 20: Dataclasses (Python 3.7+)
from dataclasses import dataclass, field
from typing import List

@dataclass
class Student:
    name: str
    age: int
    grade: float = 0.0
    subjects: List[str] = field(default_factory=list)
    
    def is_passing(self) -> bool:
        return self.grade >= 50.0
    
    def add_subject(self, subject: str) -> None:
        self.subjects.append(subject)

# สร้าง instance
student = Student("สมหญิง", 20, 85.5)
student.add_subject("Mathematics")
student.add_subject("Python")

print(student)
# Student(name='สมหญิง', age=20, grade=85.5, subjects=['Mathematics', 'Python'])
print(f"ผ่านหรือไม่: {student.is_passing()}")  # ผ่านหรือไม่: True
```

### ตัวอย่างที่ 21: Async/Await พื้นฐาน

```python
# ตัวอย่างที่ 21: Async/Await
import asyncio

async def fetch_data(url):
    """จำลองการดึงข้อมูลจาก URL"""
    print(f"เริ่มดึงข้อมูลจาก {url}")
    await asyncio.sleep(1)  # จำลองการรอ
    print(f"ดึงข้อมูลจาก {url} เสร็จแล้ว")
    return f"Data from {url}"

async def main():
    # รันหลาย tasks พร้อมกัน
    results = await asyncio.gather(
        fetch_data("https://api1.example.com"),
        fetch_data("https://api2.example.com"),
        fetch_data("https://api3.example.com"),
    )
    for result in results:
        print(result)

# รัน async function
asyncio.run(main())
```

### ตัวอย่างที่ 22: Type Hints

```python
# ตัวอย่างที่ 22: Type Hints (Python 3.5+)
from typing import List, Dict, Optional, Tuple, Union

def calculate_statistics(numbers: List[float]) -> Dict[str, float]:
    """
    คำนวณสถิติพื้นฐาน
    
    Args:
        numbers: รายการตัวเลข
    
    Returns:
        Dictionary ประกอบด้วย mean, min, max
    """
    if not numbers:
        return {}
    
    return {
        "mean": sum(numbers) / len(numbers),
        "min": min(numbers),
        "max": max(numbers),
        "count": len(numbers)
    }

def find_user(user_id: int) -> Optional[str]:
    """ค้นหา user (อาจ return None ถ้าไม่พบ)"""
    users = {1: "Alice", 2: "Bob", 3: "Charlie"}
    return users.get(user_id)

# ทดสอบ
data = [1.5, 2.5, 3.5, 4.5, 5.5]
stats = calculate_statistics(data)
print(stats)  # {'mean': 3.5, 'min': 1.5, 'max': 5.5, 'count': 5}

user = find_user(2)
print(user)   # Bob

user = find_user(99)
print(user)   # None
```

---

## แบบฝึกหัด

### ข้อที่ 1: Hello และ Input

เขียนโปรแกรมที่รับชื่อและนามสกุลจากผู้ใช้ แล้วแสดงผลในรูปแบบ:
```
ชื่อเต็ม: [ชื่อ] [นามสกุล]
ตัวอักษรทั้งหมด: [จำนวน] ตัว
```

### ข้อที่ 2: การคำนวณ BMI

เขียนโปรแกรมคำนวณ BMI โดย:
- รับน้ำหนัก (kg) และส่วนสูง (m) จากผู้ใช้
- คำนวณ BMI = weight / (height ** 2)
- แสดงผล BMI และบอกว่าอยู่ในเกณฑ์ไหน (ต่ำกว่าเกณฑ์ / ปกติ / อ้วน)

### ข้อที่ 3: ตัวเลข Fibonacci

เขียนโปรแกรมแสดงตัวเลข Fibonacci 10 ตัวแรก:
```
0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

### ข้อที่ 4: เช็คจำนวนเฉพาะ

เขียนฟังก์ชัน `is_prime(n)` ที่ return `True` ถ้า n เป็นจำนวนเฉพาะ และแสดงจำนวนเฉพาะทั้งหมดระหว่าง 1-50

### ข้อที่ 5: สร้าง Calculator

เขียน calculator ง่ายๆ ที่รับ:
- ตัวเลข 2 ตัว
- operator (+, -, *, /)

แล้วแสดงผลการคำนวณ

### ข้อที่ 6: นับคำใน String

เขียนโปรแกรมที่รับ sentence จากผู้ใช้ แล้วแสดง:
- จำนวนคำทั้งหมด
- จำนวนอักขระ (ไม่นับช่องว่าง)
- คำที่ยาวที่สุด

### ข้อที่ 7: สร้างตาราง

เขียนโปรแกรมแสดงตารางสูตรคูณ 1-5:
```
1x1=1  1x2=2  ...
2x1=2  2x2=4  ...
...
```

### ข้อที่ 8: Palindrome Checker

เขียนฟังก์ชันตรวจสอบว่า string เป็น palindrome หรือไม่ (อ่านหน้าหลังเหมือนกัน เช่น "racecar", "madam")

### ข้อที่ 9: จัดการไฟล์

เขียนโปรแกรมที่:
- เขียนชื่อ 5 คนลงในไฟล์ `names.txt`
- อ่านไฟล์กลับมา
- เรียงชื่อตามตัวอักษร
- เขียนกลับลงไฟล์ใหม่ `names_sorted.txt`

### ข้อที่ 10: โปรแกรม Shopping Cart

เขียนโปรแกรม shopping cart ง่ายๆ ที่:
- มีรายการสินค้าพร้อมราคา (dictionary)
- ผู้ใช้เลือกสินค้า (loop)
- คำนวณราคารวม
- แสดงใบเสร็จ

---

## เฉลยแบบฝึกหัด

### เฉลยข้อที่ 1: Hello และ Input

```python
# เฉลยข้อที่ 1
first_name = input("กรุณาพิมพ์ชื่อ: ")
last_name = input("กรุณาพิมพ์นามสกุล: ")

full_name = f"{first_name} {last_name}"
char_count = len(first_name) + len(last_name)  # ไม่นับช่องว่าง

print(f"ชื่อเต็ม: {full_name}")
print(f"ตัวอักษรทั้งหมด: {char_count} ตัว")
```

### เฉลยข้อที่ 2: การคำนวณ BMI

```python
# เฉลยข้อที่ 2
weight = float(input("น้ำหนัก (kg): "))
height = float(input("ส่วนสูง (m): "))

bmi = weight / (height ** 2)
print(f"BMI ของคุณ: {bmi:.2f}")

if bmi < 18.5:
    category = "ต่ำกว่าเกณฑ์ (underweight)"
elif bmi < 25.0:
    category = "ปกติ (normal)"
elif bmi < 30.0:
    category = "น้ำหนักเกิน (overweight)"
else:
    category = "อ้วน (obese)"

print(f"เกณฑ์: {category}")
```

### เฉลยข้อที่ 3: Fibonacci

```python
# เฉลยข้อที่ 3
def fibonacci(n):
    """สร้าง Fibonacci sequence"""
    sequence = []
    a, b = 0, 1
    for _ in range(n):
        sequence.append(a)
        a, b = b, a + b
    return sequence

result = fibonacci(10)
print(", ".join(map(str, result)))
# 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

### เฉลยข้อที่ 4: จำนวนเฉพาะ

```python
# เฉลยข้อที่ 4
def is_prime(n):
    """ตรวจสอบว่า n เป็นจำนวนเฉพาะหรือไม่"""
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    for i in range(3, int(n**0.5) + 1, 2):
        if n % i == 0:
            return False
    return True

primes = [n for n in range(1, 51) if is_prime(n)]
print(f"จำนวนเฉพาะระหว่าง 1-50: {primes}")
```

### เฉลยข้อที่ 5: Calculator

```python
# เฉลยข้อที่ 5
def calculate(a, operator, b):
    """คำนวณตาม operator"""
    operations = {
        '+': lambda x, y: x + y,
        '-': lambda x, y: x - y,
        '*': lambda x, y: x * y,
        '/': lambda x, y: x / y if y != 0 else "Error: หารด้วยศูนย์!"
    }
    
    if operator not in operations:
        return "Error: operator ไม่ถูกต้อง"
    
    return operations[operator](a, b)

try:
    num1 = float(input("ตัวเลขที่ 1: "))
    op = input("Operator (+, -, *, /): ")
    num2 = float(input("ตัวเลขที่ 2: "))
    
    result = calculate(num1, op, num2)
    print(f"{num1} {op} {num2} = {result}")
except ValueError:
    print("กรุณาพิมพ์ตัวเลขที่ถูกต้อง")
```

### เฉลยข้อที่ 6: นับคำใน String

```python
# เฉลยข้อที่ 6
sentence = input("พิมพ์ประโยค: ")

words = sentence.split()
word_count = len(words)
char_count = len(sentence.replace(" ", ""))
longest_word = max(words, key=len) if words else ""

print(f"จำนวนคำ: {word_count} คำ")
print(f"จำนวนอักขระ (ไม่นับช่องว่าง): {char_count} ตัว")
print(f"คำที่ยาวที่สุด: {longest_word} ({len(longest_word)} ตัวอักษร)")
```

### เฉลยข้อที่ 7: ตารางสูตรคูณ

```python
# เฉลยข้อที่ 7
print("ตารางสูตรคูณ 1-5")
print("-" * 40)

for i in range(1, 6):
    row = ""
    for j in range(1, 6):
        row += f"{i}x{j}={i*j:<3} "
    print(row)
```

### เฉลยข้อที่ 8: Palindrome Checker

```python
# เฉลยข้อที่ 8
def is_palindrome(text):
    """ตรวจสอบว่าเป็น palindrome หรือไม่"""
    cleaned = text.lower().replace(" ", "")
    return cleaned == cleaned[::-1]

test_words = ["racecar", "hello", "madam", "python", "level", "world"]

for word in test_words:
    result = "เป็น palindrome" if is_palindrome(word) else "ไม่ใช่ palindrome"
    print(f"'{word}': {result}")
```

### เฉลยข้อที่ 9: จัดการไฟล์

```python
# เฉลยข้อที่ 9
# เขียนชื่อลงไฟล์
names = ["สมชาย", "มานี", "วิชัย", "อรุณ", "ปิยะ"]

with open("names.txt", "w", encoding="utf-8") as f:
    for name in names:
        f.write(name + "\n")

print("บันทึกชื่อลงไฟล์ names.txt แล้ว")

# อ่านไฟล์กลับมา
with open("names.txt", "r", encoding="utf-8") as f:
    read_names = [line.strip() for line in f if line.strip()]

# เรียงชื่อ
sorted_names = sorted(read_names)

# เขียนไฟล์ที่เรียงแล้ว
with open("names_sorted.txt", "w", encoding="utf-8") as f:
    for name in sorted_names:
        f.write(name + "\n")

print(f"ชื่อที่เรียงแล้ว: {sorted_names}")
print("บันทึกลง names_sorted.txt แล้ว")
```

### เฉลยข้อที่ 10: Shopping Cart

```python
# เฉลยข้อที่ 10
# รายการสินค้า
menu = {
    "กาแฟ": 45,
    "ชา": 35,
    "น้ำผลไม้": 55,
    "เค้ก": 75,
    "คุกกี้": 25
}

cart = {}

print("=== ร้านกาแฟ ===")
print("รายการสินค้า:")
for item, price in menu.items():
    print(f"  {item}: {price} บาท")

while True:
    print("\nพิมพ์ชื่อสินค้าเพื่อเพิ่ม (หรือ 'done' เพื่อสั่งซื้อ): ", end="")
    choice = input().strip()
    
    if choice.lower() == 'done':
        break
    
    if choice in menu:
        cart[choice] = cart.get(choice, 0) + 1
        print(f"เพิ่ม {choice} ใน cart แล้ว")
    else:
        print("ไม่พบสินค้าในเมนู")

# แสดงใบเสร็จ
print("\n" + "=" * 30)
print("ใบเสร็จ:")
print("-" * 30)

total = 0
for item, qty in cart.items():
    price = menu[item]
    subtotal = price * qty
    total += subtotal
    print(f"{item} x{qty}  {subtotal} บาท")

print("-" * 30)
print(f"รวมทั้งหมด: {total} บาท")
print("=" * 30)
print("ขอบคุณที่ใช้บริการ!")
```

---

## สรุป

ใน Part 01 นี้เราได้เรียนรู้:

1. **Python คืออะไร** - ภาษาโปรแกรมมิ่งที่ง่าย อ่านง่าย ใช้งานหลากหลาย
2. **การติดตั้ง** - บน Windows, macOS, Linux พร้อมวิธีการทดสอบ
3. **VS Code Setup** - extensions ที่จำเป็น การตั้งค่า
4. **REPL** - การใช้งาน interactive mode
5. **Hello World** - โปรแกรมแรก
6. **การรันไฟล์** - python script.py และ options ต่างๆ
7. **pip** - การจัดการ packages
8. **Virtual Environment** - การแยก environment ต่อโปรเจกต์
9. **โครงสร้างโปรเจกต์** - best practices

## ขั้นตอนต่อไป

- **Part 02**: Variables, Data Types & Operators
- ฝึกทำแบบฝึกหัดทั้ง 10 ข้อ
- ลองติดตั้ง Python และ VS Code
- ทดลองใช้ REPL สัก 15-30 นาที

---

*หมายเหตุ: ทุก code block ในบทเรียนนี้ทดสอบแล้วบน Python 3.12*
