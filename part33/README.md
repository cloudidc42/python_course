# Part 33: Virtual Environments & Dependency Management

## บทนำ

หนึ่งในปัญหาที่พบบ่อยที่สุดเมื่อพัฒนา Python projects คือ "dependency conflicts" - เมื่อ project A ต้องการ Django 3.2 แต่ project B ต้องการ Django 4.2 Virtual environments และ dependency management tools แก้ปัญหานี้ได้ ในบทนี้เราจะเรียนรู้ครบทุก tool ที่ Python developer ควรรู้

---

## 1. Python Environments และปัญหา Dependency Conflicts

### ปัญหาที่เกิดขึ้น

```
ระบบ Python ของคุณ:
/usr/local/lib/python3.11/
├── requests 2.28.0     ← project A ต้องการ 2.28
└── flask 2.0.0         ← project A ต้องการ 2.0

แต่ project B ต้องการ:
├── requests 2.31.0     ← conflict!
└── flask 3.0.0         ← conflict!
```

```python
# ตัวอย่างที่ 1: ดู Python environment ปัจจุบัน
import sys
import site

print(f"Python version: {sys.version}")
print(f"Python executable: {sys.executable}")
print(f"Site packages: {site.getsitepackages()}")
print(f"Virtual env: {'VIRTUAL_ENV' in __import__('os').environ}")
```

```python
# ตัวอย่างที่ 2: ตรวจสอบ packages ที่ติดตั้ง
import importlib.metadata
import sys

def list_installed_packages():
    """แสดงรายการ packages ที่ติดตั้งทั้งหมด"""
    packages = []
    for dist in importlib.metadata.distributions():
        packages.append({
            'name': dist.metadata['Name'],
            'version': dist.metadata['Version']
        })
    return sorted(packages, key=lambda x: x['name'].lower())

packages = list_installed_packages()
print(f"จำนวน packages ที่ติดตั้ง: {len(packages)}")
for pkg in packages[:10]:  # แสดง 10 อันแรก
    print(f"  {pkg['name']}: {pkg['version']}")
```

---

## 2. venv Module - Virtual Environment มาตรฐาน

### การสร้าง Virtual Environment

```bash
# ตัวอย่างที่ 3: สร้าง virtual environment
# สร้าง venv ในโฟลเดอร์ .venv
python3 -m venv .venv

# โครงสร้าง venv ที่สร้างขึ้น:
# .venv/
# ├── bin/           (Linux/Mac) หรือ Scripts/ (Windows)
# │   ├── python
# │   ├── python3
# │   ├── pip
# │   └── activate
# ├── lib/
# │   └── python3.11/
# │       └── site-packages/
# └── pyvenv.cfg
```

```bash
# ตัวอย่างที่ 4: activate virtual environment
# Linux/Mac:
source .venv/bin/activate

# Windows:
.venv\Scripts\activate

# Windows PowerShell:
.venv\Scripts\Activate.ps1

# ตรวจสอบว่า activate สำเร็จ
which python    # ควรชี้ไปที่ .venv/bin/python
python --version
```

```bash
# ตัวอย่างที่ 5: deactivate virtual environment
deactivate
```

### ตัวเลือกเพิ่มเติมของ venv

```bash
# ตัวอย่างที่ 6: สร้าง venv พร้อม options
# --system-site-packages: ใช้ packages จาก system Python ด้วย
python3 -m venv --system-site-packages .venv

# --copies: copy files แทน symlinks (Windows)
python3 -m venv --copies .venv

# --upgrade: upgrade venv หลังจาก Python upgrade
python3 -m venv --upgrade .venv

# --without-pip: ไม่ติดตั้ง pip
python3 -m venv --without-pip .venv

# ระบุ Python version ที่ต้องการ
python3.11 -m venv .venv311
python3.12 -m venv .venv312
```

```python
# ตัวอย่างที่ 7: ตรวจสอบ virtual environment ใน Python
import sys
import os

def check_venv():
    """ตรวจสอบว่าอยู่ใน virtual environment หรือไม่"""
    in_venv = (
        hasattr(sys, 'real_prefix') or  # virtualenv
        (hasattr(sys, 'base_prefix') and sys.base_prefix != sys.prefix)  # venv
    )
    
    if in_venv:
        venv_path = os.environ.get('VIRTUAL_ENV', 'unknown')
        print(f"อยู่ใน virtual environment: {venv_path}")
        print(f"Python: {sys.executable}")
    else:
        print("ไม่ได้อยู่ใน virtual environment - ระวัง!")
    
    return in_venv

check_venv()
```

---

## 3. pip คำสั่งทั้งหมด

```bash
# ตัวอย่างที่ 8: pip คำสั่งพื้นฐาน

# ติดตั้ง package
pip install requests
pip install flask

# ติดตั้ง version ที่ระบุ
pip install requests==2.28.0
pip install "flask>=2.0,<3.0"
pip install "django~=4.2"   # ~= หมายถึง compatible release (4.2.x)

# ติดตั้งหลาย packages พร้อมกัน
pip install requests flask django

# ลบ package
pip uninstall requests
pip uninstall requests -y    # ไม่ถาม confirm

# อัพเดท package
pip install --upgrade requests
pip install -U requests      # -U เป็น shorthand

# แสดง packages ที่ติดตั้ง
pip list
pip list --outdated          # packages ที่มี version ใหม่กว่า
pip list --format=json       # ในรูปแบบ JSON

# ดู info ของ package
pip show requests
pip show --files requests    # แสดง files ที่ติดตั้ง
```

```bash
# ตัวอย่างที่ 9: pip search และ download

# ค้นหา packages (deprecated ใน pip 21.0+, ใช้ PyPI website แทน)
# pip search requests

# Download package โดยไม่ติดตั้ง
pip download requests -d ./packages/

# ติดตั้งจาก local directory
pip install ./packages/requests-2.28.0-py3-none-any.whl

# ติดตั้งจาก git repository
pip install git+https://github.com/psf/requests.git

# ติดตั้ง development dependencies
pip install -e .             # editable install (development mode)
pip install ".[dev]"         # ติดตั้งพร้อม extras

# ดู pip version
pip --version
pip -V
```

```bash
# ตัวอย่างที่ 10: pip config

# ดู pip config
pip config list

# ตั้ง custom index (private PyPI)
pip config set global.index-url https://pypi.mycompany.com/simple/

# เพิ่ม extra index
pip install --extra-index-url https://private.pypi.com/simple/ mypackage

# ติดตั้งแบบ offline
pip install --no-index --find-links=/path/to/packages requests

# ดู cache
pip cache info
pip cache list
pip cache purge
```

---

## 4. requirements.txt

```bash
# ตัวอย่างที่ 11: สร้าง requirements.txt
# freeze packages ที่ติดตั้งทั้งหมดพร้อม version
pip freeze > requirements.txt

# ติดตั้งจาก requirements.txt
pip install -r requirements.txt

# ติดตั้งพร้อม upgrade
pip install -r requirements.txt --upgrade
```

```text
# ตัวอย่างที่ 12: รูปแบบ requirements.txt ต่างๆ
# requirements.txt

# ระบุ version แน่นอน (recommended for production)
requests==2.28.0
flask==2.3.2
SQLAlchemy==2.0.0

# ระบุ minimum version
boto3>=1.26.0

# ระบุ range
django>=4.0,<5.0

# Compatible release
celery~=5.3.0

# ไม่ระบุ version (ไม่แนะนำ)
pytest

# Comment
# Development tools
black==23.7.0
flake8==6.1.0
mypy==1.5.0

# ติดตั้งจากไฟล์อื่น
-r base.txt

# ติดตั้งจาก git
git+https://github.com/org/mypackage.git@v1.0.0#egg=mypackage

# ติดตั้งจาก local path
-e ./local-package/
```

```bash
# ตัวอย่างที่ 13: Multiple requirements files
# โครงสร้าง requirements files ที่ดี:

# requirements/
# ├── base.txt       - packages สำหรับทุก environment
# ├── development.txt - packages สำหรับ development เท่านั้น
# ├── testing.txt    - packages สำหรับ testing
# └── production.txt - packages สำหรับ production

# requirements/base.txt
cat requirements/base.txt
# flask==2.3.2
# SQLAlchemy==2.0.0
# requests==2.28.0

# requirements/development.txt
cat requirements/development.txt
# -r base.txt
# flask-debugtoolbar==0.14.0
# black==23.7.0
# flake8==6.1.0

# requirements/testing.txt  
cat requirements/testing.txt
# -r base.txt
# pytest==7.4.0
# pytest-cov==4.1.0
# factory-boy==3.3.0

# ติดตั้ง
pip install -r requirements/development.txt
```

```python
# ตัวอย่างที่ 14: Script สร้าง requirements.txt อัตโนมัติ
import subprocess
import json

def get_installed_packages():
    """ดึงรายการ packages ที่ติดตั้ง"""
    result = subprocess.run(
        ['pip', 'list', '--format=json'],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)

def generate_requirements(output_file='requirements.txt', 
                           exclude_packages=None):
    """สร้าง requirements.txt"""
    exclude = exclude_packages or ['pip', 'setuptools', 'wheel']
    packages = get_installed_packages()
    
    lines = [f"# Auto-generated requirements - {__import__('datetime').date.today()}\n"]
    
    for pkg in packages:
        if pkg['name'].lower() not in [e.lower() for e in exclude]:
            lines.append(f"{pkg['name']}=={pkg['version']}\n")
    
    with open(output_file, 'w') as f:
        f.writelines(lines)
    
    print(f"บันทึก {len(lines)-1} packages ลง {output_file}")

generate_requirements(exclude_packages=['pip', 'setuptools', 'wheel'])
```

---

## 5. pip-tools - Pinned Dependencies Management

pip-tools แก้ปัญหา transitive dependencies โดยสร้าง lock file

```bash
# ติดตั้ง pip-tools
pip install pip-tools

# สร้าง requirements.in (direct dependencies เท่านั้น)
# requirements.in:
# flask
# requests>=2.25.0
# SQLAlchemy

# Compile เพื่อสร้าง requirements.txt พร้อม pinned versions
pip-compile requirements.in

# ผลลัพธ์ requirements.txt จะมี transitive deps ด้วย:
# flask==2.3.2
#     via -r requirements.in
# requests==2.31.0
#     via -r requirements.in
# werkzeug==3.0.1
#     via flask
# ...

# อัพเดท dependencies
pip-compile --upgrade requirements.in

# sync environment ตาม requirements.txt
pip-sync requirements.txt

# Multiple input files
pip-compile requirements.in dev-requirements.in -o requirements-dev.txt
```

---

## 6. pipenv - Pipfile และ Virtual Environment ในที่เดียว

```bash
# ติดตั้ง pipenv
pip install pipenv

# สร้าง virtual environment และ Pipfile
pipenv --python 3.11

# ติดตั้ง package
pipenv install requests flask
pipenv install pytest --dev    # development-only package

# Activate shell
pipenv shell

# Run command ใน virtual environment โดยไม่ต้อง activate
pipenv run python app.py
pipenv run pytest

# สร้าง Pipfile.lock (lock file)
pipenv lock

# ติดตั้งจาก Pipfile
pipenv install
pipenv install --dev           # รวม dev packages

# ดู dependency graph
pipenv graph

# ตรวจสอบ security vulnerabilities
pipenv check

# ลบ virtual environment
pipenv --rm
```

```toml
# ตัวอย่างที่ 15: Pipfile structure
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"
flask = ">=2.0"
sqlalchemy = "~=2.0"

[dev-packages]
pytest = "*"
black = "*"
flake8 = "*"
mypy = "*"

[requires]
python_version = "3.11"

[scripts]
start = "python app.py"
test = "pytest tests/"
lint = "flake8 src/"
```

---

## 7. Poetry - Modern Dependency Management

Poetry เป็น tool ที่ทันสมัยที่สุด รวม package management, dependency resolution, building, และ publishing

```bash
# ตัวอย่างที่ 16: ติดตั้ง Poetry
curl -sSL https://install.python-poetry.org | python3 -

# หรือด้วย pip
pip install poetry

# ตรวจสอบ version
poetry --version

# สร้างโปรเจกต์ใหม่
poetry new my-project

# โครงสร้าง:
# my-project/
# ├── pyproject.toml
# ├── README.md
# ├── my_project/
# │   └── __init__.py
# └── tests/
#     └── __init__.py
```

```bash
# ตัวอย่างที่ 17: Poetry commands หลัก

# สร้าง pyproject.toml ในโปรเจกต์ที่มีอยู่
poetry init

# ติดตั้ง package
poetry add requests
poetry add "flask>=2.0"
poetry add pytest --group dev
poetry add sphinx --group docs

# ลบ package
poetry remove requests

# ติดตั้ง dependencies ทั้งหมด
poetry install
poetry install --with dev,docs    # รวม groups เพิ่มเติม
poetry install --without test     # ยกเว้น group

# อัพเดท dependencies
poetry update
poetry update requests            # อัพเดทแค่ requests

# Show dependencies
poetry show
poetry show --tree               # dependency tree
poetry show requests

# Virtual environment
poetry env info                   # ดู environment
poetry env use python3.11        # เปลี่ยน Python version
poetry env list                  # แสดง virtual envs

# Run commands
poetry run python app.py
poetry run pytest

# Activate shell
poetry shell
```

---

## 8. pyproject.toml - การตั้งค่าโปรเจกต์แบบใหม่

```toml
# ตัวอย่างที่ 18: pyproject.toml สมบูรณ์
[tool.poetry]
name = "my-awesome-app"
version = "1.0.0"
description = "A demonstration Python application"
authors = ["Alice <alice@example.com>"]
readme = "README.md"
homepage = "https://github.com/alice/my-awesome-app"
repository = "https://github.com/alice/my-awesome-app"
documentation = "https://my-awesome-app.readthedocs.io"
keywords = ["python", "web", "api"]
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]
packages = [{include = "my_awesome_app", from = "src"}]

[tool.poetry.dependencies]
python = "^3.11"
flask = "^3.0"
sqlalchemy = "^2.0"
requests = "^2.28"
pydantic = "^2.0"
python-dotenv = "^1.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4"
pytest-cov = "^4.1"
black = "^23.7"
flake8 = "^6.1"
mypy = "^1.5"
isort = "^5.12"

[tool.poetry.group.docs.dependencies]
sphinx = "^7.0"
sphinx-rtd-theme = "^1.3"

[tool.poetry.scripts]
my-app = "my_awesome_app.cli:main"
migrate = "my_awesome_app.db:run_migrations"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

# Tool configurations
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
addopts = "-v --tb=short --cov=src"

[tool.black]
line-length = 88
target-version = ["py311"]
include = '\.pyi?$'
exclude = '''
/(
    \.git
  | \.venv
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
multi_line_output = 3

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
ignore_missing_imports = true

[tool.flake8]
max-line-length = 88
extend-ignore = ["E203", "W503"]
```

```bash
# ตัวอย่างที่ 19: Poetry build และ publish

# Build package (สร้าง wheel และ sdist)
poetry build

# ผลลัพธ์ใน dist/:
# dist/
# ├── my_awesome_app-1.0.0-py3-none-any.whl
# └── my_awesome_app-1.0.0.tar.gz

# Publish ไปยัง PyPI
poetry publish

# Publish ไปยัง test PyPI
poetry publish -r testpypi

# เพิ่ม repository
poetry config repositories.myrepo https://pypi.mycompany.com/simple/
poetry publish -r myrepo
```

---

## 9. conda Environments

conda เหมาะสำหรับ data science projects เพราะจัดการ non-Python packages ได้ด้วย

```bash
# ตัวอย่างที่ 20: conda commands

# ตรวจสอบ conda version
conda --version

# สร้าง environment ใหม่
conda create -n myenv python=3.11
conda create -n datascience python=3.11 numpy pandas matplotlib

# Activate environment
conda activate myenv

# Deactivate
conda deactivate

# ติดตั้ง packages
conda install numpy pandas scikit-learn
conda install -c conda-forge matplotlib

# แสดง environments ทั้งหมด
conda env list
conda info --envs

# ลบ environment
conda remove -n myenv --all

# Export environment
conda env export > environment.yml
conda env export --from-history > environment.yml  # only explicitly installed

# สร้าง environment จาก yml
conda env create -f environment.yml

# Clone environment
conda create --name new_env --clone existing_env

# อัพเดท packages
conda update numpy
conda update --all
```

```yaml
# ตัวอย่างที่ 21: environment.yml สำหรับ Data Science
name: datascience-env
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.11
  - numpy=1.24.0
  - pandas=2.0.0
  - matplotlib=3.7.0
  - scikit-learn=1.3.0
  - jupyter=1.0.0
  - ipykernel=6.25.0
  - pip:
    - torch==2.0.0
    - transformers==4.33.0
    - openai==0.27.8
```

---

## 10. Environment Variables (.env files)

```bash
# ตัวอย่างที่ 22: .env file format
# .env file (ไม่ควร commit ลง git!)
# ใส่ใน .gitignore:
# .env
# .env.*
# !.env.example

# .env
DATABASE_URL=postgresql://user:password@localhost/mydb
SECRET_KEY=your-very-secret-key-here-change-in-production
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# API Keys
OPENAI_API_KEY=sk-...
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...

# Email settings
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=myapp@gmail.com
EMAIL_HOST_PASSWORD=app-password-here

# Feature flags
FEATURE_NEW_UI=False
FEATURE_BETA_SEARCH=True
```

```bash
# .env.example - template สำหรับ developers คนอื่น
DATABASE_URL=postgresql://localhost/mydb_dev
SECRET_KEY=change-me-in-local
DEBUG=True
OPENAI_API_KEY=your-api-key-here
```

---

## 11. python-dotenv

```python
# pip install python-dotenv

# ตัวอย่างที่ 23: python-dotenv usage
from dotenv import load_dotenv
import os

# โหลด .env file
load_dotenv()

# หรือระบุ path
load_dotenv('/path/to/.env')

# ใช้ environment variables
db_url = os.getenv('DATABASE_URL')
secret_key = os.getenv('SECRET_KEY', 'default-secret')  # มี default
debug = os.getenv('DEBUG', 'False').lower() == 'true'  # แปลงเป็น bool

print(f"Database: {db_url}")
print(f"Debug mode: {debug}")
```

```python
# ตัวอย่างที่ 24: dotenv_values() สำหรับ dict
from dotenv import dotenv_values

# อ่าน .env เป็น dict (ไม่ set environment variables)
config = dotenv_values('.env')
print(config)  # {'DATABASE_URL': 'postgresql://...', ...}

# Combine multiple .env files
config = {
    **dotenv_values('.env'),         # base settings
    **dotenv_values('.env.local'),   # local overrides
    **os.environ                     # real environment overrides all
}
```

```python
# ตัวอย่างที่ 25: Settings class ด้วย pydantic-settings
# pip install pydantic-settings

from pydantic_settings import BaseSettings
from pydantic import Field
from typing import Optional, List

class Settings(BaseSettings):
    """Application settings with validation"""
    
    # Database
    database_url: str = Field(default="sqlite:///app.db")
    db_pool_size: int = Field(default=5, ge=1, le=100)
    
    # Security
    secret_key: str = Field(min_length=32)
    debug: bool = Field(default=False)
    allowed_hosts: List[str] = Field(default=["localhost"])
    
    # External APIs
    openai_api_key: Optional[str] = None
    
    # Email
    email_host: str = Field(default="localhost")
    email_port: int = Field(default=587)
    email_use_tls: bool = Field(default=True)
    
    model_config = {
        "env_file": ".env",
        "env_file_encoding": "utf-8",
        "case_sensitive": False,
        "env_prefix": ""
    }
    
    @property
    def is_production(self):
        return not self.debug and "production" in self.database_url

# ใช้งาน (ต้องมี .env file หรือ environment variables)
# settings = Settings()
# print(f"DB: {settings.database_url}")
# print(f"Production: {settings.is_production}")
```

---

## 12. Best Practices สำหรับ Python Projects

### โครงสร้าง project ที่ดี

```
my-project/
├── .git/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .venv/                    # virtual environment (ใน .gitignore)
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── models/
│       ├── services/
│       └── utils/
├── tests/
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── docs/
├── scripts/
│   ├── setup.sh
│   └── deploy.sh
├── .env.example              # template (commit นี้)
├── .env                      # actual secrets (ใน .gitignore)
├── .gitignore
├── .python-version           # pyenv version file
├── Makefile
├── pyproject.toml
├── poetry.lock               # หรือ requirements.txt
└── README.md
```

```makefile
# ตัวอย่างที่ 26: Makefile สำหรับ Python project
.PHONY: install dev test lint format check clean

# ติดตั้ง dependencies
install:
	poetry install --without dev

# ติดตั้ง development dependencies
dev:
	poetry install
	pre-commit install

# Run tests
test:
	poetry run pytest tests/ -v --cov=src --cov-report=html

# Lint code
lint:
	poetry run flake8 src/ tests/
	poetry run mypy src/

# Format code
format:
	poetry run black src/ tests/
	poetry run isort src/ tests/

# Check everything
check: lint test

# Clean up
clean:
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete
	rm -rf .coverage htmlcov/ dist/ build/

# Start development server
run:
	poetry run python -m my_project

# Docker
docker-build:
	docker build -t my-project .

docker-run:
	docker run -p 8000:8000 my-project
```

```gitignore
# ตัวอย่างที่ 27: .gitignore สำหรับ Python project
# Byte-compiled files
__pycache__/
*.py[cod]
*$py.class
*.pyc

# Distribution / packaging
dist/
build/
*.egg-info/
*.egg

# Virtual environments
.venv/
venv/
env/
ENV/
.env/

# Environment variables
.env
.env.local
.env.*.local

# Testing
.coverage
.coverage.*
htmlcov/
.pytest_cache/
.tox/
nosetests.xml

# Type checking
.mypy_cache/
.pytype/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Project specific
*.db
*.sqlite3
logs/
*.log
```

```bash
# ตัวอย่างที่ 28: pre-commit hooks
# pip install pre-commit
# หรือ poetry add pre-commit --group dev

# สร้าง .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        additional_dependencies: [flake8-bugbear]
EOF

# ติดตั้ง hooks
pre-commit install

# ทดสอบ run manually
pre-commit run --all-files
```

---

## 13. Docker กับ Python Projects

```dockerfile
# ตัวอย่างที่ 29: Dockerfile สำหรับ Python app
# Dockerfile

# Build stage
FROM python:3.11-slim as builder

WORKDIR /app

# ติดตั้ง Poetry
RUN pip install poetry==1.7.0

# Copy dependency files
COPY pyproject.toml poetry.lock ./

# Export requirements (ไม่รวม dev)
RUN poetry export -f requirements.txt --output requirements.txt --without-hashes

# Production stage
FROM python:3.11-slim

WORKDIR /app

# ติดตั้ง system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements จาก builder
COPY --from=builder /app/requirements.txt .

# ติดตั้ง Python packages
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY src/ ./src/

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Run app
CMD ["python", "-m", "my_project"]
```

---

## 14. pyenv - จัดการหลาย Python versions

```bash
# ตัวอย่างที่ 30: pyenv usage

# ติดตั้ง pyenv
curl https://pyenv.run | bash

# เพิ่มใน shell config (~/.bashrc หรือ ~/.zshrc)
# export PYENV_ROOT="$HOME/.pyenv"
# export PATH="$PYENV_ROOT/bin:$PATH"
# eval "$(pyenv init -)"

# แสดง Python versions ที่มี
pyenv install --list
pyenv install --list | grep "3.11"

# ติดตั้ง Python versions
pyenv install 3.11.5
pyenv install 3.12.0

# ดู versions ที่ติดตั้ง
pyenv versions

# ตั้ง global default
pyenv global 3.11.5

# ตั้ง local version (สร้างไฟล์ .python-version ในโฟลเดอร์นั้น)
pyenv local 3.12.0

# ตั้ง shell-specific version
pyenv shell 3.10.12

# ดู current version
pyenv version
python --version
```

---

## แบบฝึกหัด

### ข้อที่ 1: สร้าง Python Project Structure

**เฉลย:**
```python
import os
import subprocess
import sys

def create_project_structure(project_name: str, use_poetry: bool = True):
    """สร้างโครงสร้าง Python project"""
    
    # สร้าง directories
    dirs = [
        f"{project_name}/src/{project_name.replace('-', '_')}",
        f"{project_name}/tests/unit",
        f"{project_name}/tests/integration",
        f"{project_name}/docs",
        f"{project_name}/scripts",
    ]
    
    for d in dirs:
        os.makedirs(d, exist_ok=True)
        print(f"สร้าง: {d}/")
    
    # สร้างไฟล์
    files = {
        f"{project_name}/.gitignore": GITIGNORE_CONTENT,
        f"{project_name}/.env.example": ENV_EXAMPLE_CONTENT,
        f"{project_name}/src/{project_name.replace('-', '_')}/__init__.py": f'"""Package {project_name}."""\n\n__version__ = "0.1.0"\n',
        f"{project_name}/tests/__init__.py": "",
        f"{project_name}/tests/conftest.py": CONFTEST_CONTENT,
        f"{project_name}/Makefile": MAKEFILE_CONTENT,
    }
    
    for filepath, content in files.items():
        with open(filepath, 'w') as f:
            f.write(content)
        print(f"สร้าง: {filepath}")
    
    print(f"\nโปรเจกต์ {project_name} สร้างสำเร็จ!")
    print(f"cd {project_name} && python -m venv .venv && source .venv/bin/activate")

GITIGNORE_CONTENT = """__pycache__/
*.pyc
.venv/
.env
.coverage
dist/
build/
*.egg-info/
"""

ENV_EXAMPLE_CONTENT = """DATABASE_URL=sqlite:///dev.db
SECRET_KEY=change-me
DEBUG=True
"""

CONFTEST_CONTENT = """import pytest

@pytest.fixture
def sample_data():
    return {}
"""

MAKEFILE_CONTENT = """install:
\tpip install -r requirements.txt

test:
\tpytest tests/ -v

lint:
\tflake8 src/ tests/
"""

create_project_structure("my-awesome-project")
```

---

### ข้อที่ 2: Environment Manager

**เฉลย:**
```python
import subprocess
import sys
import os
import json
from pathlib import Path

class VenvManager:
    """จัดการ virtual environments"""
    
    def __init__(self, base_dir='.'):
        self.base_dir = Path(base_dir)
    
    def create(self, name='.venv', python_version=None):
        """สร้าง virtual environment"""
        venv_path = self.base_dir / name
        cmd = [sys.executable, '-m', 'venv', str(venv_path)]
        if python_version:
            # ใช้ python version ที่ระบุ (ต้องมี pyenv)
            cmd[0] = f"python{python_version}"
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0:
            print(f"สร้าง venv ที่ {venv_path} สำเร็จ!")
            return venv_path
        else:
            raise RuntimeError(f"ไม่สามารถสร้าง venv: {result.stderr}")
    
    def get_pip(self, venv_path):
        """หา pip ใน venv"""
        if sys.platform == 'win32':
            return venv_path / 'Scripts' / 'pip.exe'
        return venv_path / 'bin' / 'pip'
    
    def install_requirements(self, venv_path, requirements_file='requirements.txt'):
        """ติดตั้ง requirements"""
        pip = self.get_pip(Path(venv_path))
        result = subprocess.run(
            [str(pip), 'install', '-r', requirements_file],
            capture_output=True, text=True
        )
        if result.returncode == 0:
            print(f"ติดตั้ง requirements สำเร็จ!")
        else:
            print(f"Error: {result.stderr}")
    
    def list_packages(self, venv_path):
        """แสดง packages ใน venv"""
        pip = self.get_pip(Path(venv_path))
        result = subprocess.run(
            [str(pip), 'list', '--format=json'],
            capture_output=True, text=True
        )
        if result.returncode == 0:
            return json.loads(result.stdout)
        return []

# ใช้งาน (แสดงวิธีการ)
manager = VenvManager()
print("VenvManager สร้างสำเร็จ!")
print("ใช้ manager.create() เพื่อสร้าง virtual environment")
```

---

### ข้อที่ 3: Dependency Checker

**เฉลย:**
```python
import subprocess
import json
import re
from typing import Dict, List, Tuple

def check_dependencies() -> Dict:
    """ตรวจสอบ dependencies ทั้งหมด"""
    
    # ดู packages ที่ outdated
    result = subprocess.run(
        ['pip', 'list', '--outdated', '--format=json'],
        capture_output=True, text=True
    )
    
    outdated = []
    if result.returncode == 0:
        outdated = json.loads(result.stdout)
    
    # ดู packages ทั้งหมด
    result = subprocess.run(
        ['pip', 'list', '--format=json'],
        capture_output=True, text=True
    )
    
    all_packages = []
    if result.returncode == 0:
        all_packages = json.loads(result.stdout)
    
    return {
        'total': len(all_packages),
        'outdated': outdated,
        'outdated_count': len(outdated)
    }

def parse_requirements(filepath: str) -> List[Tuple[str, str]]:
    """Parse requirements.txt"""
    packages = []
    
    try:
        with open(filepath, 'r') as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith('#') or line.startswith('-'):
                    continue
                # Parse package name และ version constraint
                match = re.match(r'^([a-zA-Z0-9_\-\.]+)(.*)$', line)
                if match:
                    packages.append((match.group(1), match.group(2).strip()))
    except FileNotFoundError:
        print(f"ไม่พบไฟล์ {filepath}")
    
    return packages

def check_security(package_name: str) -> dict:
    """ตรวจสอบ security vulnerabilities (ใช้ pip-audit)"""
    # pip install pip-audit
    result = subprocess.run(
        ['pip-audit', '--format=json', '-r', '/dev/stdin'],
        input=f"{package_name}\n",
        capture_output=True, text=True
    )
    
    if result.returncode == 0:
        try:
            return json.loads(result.stdout)
        except:
            pass
    return {}

# ใช้งาน
print("กำลังตรวจสอบ dependencies...")
deps = check_dependencies()
print(f"รวม {deps['total']} packages")
print(f"Outdated: {deps['outdated_count']} packages")
if deps['outdated']:
    print("\nPackages ที่ outdated:")
    for pkg in deps['outdated'][:5]:
        print(f"  {pkg['name']}: {pkg['version']} -> {pkg['latest_version']}")
```

---

### ข้อที่ 4: Config Manager สำหรับหลาย Environments

**เฉลย:**
```python
import os
from pathlib import Path
from typing import Any, Dict, Optional

class MultiEnvConfig:
    """จัดการ config สำหรับหลาย environments"""
    
    ENVIRONMENTS = ['development', 'staging', 'production', 'testing']
    
    def __init__(self, base_dir='.'):
        self.base_dir = Path(base_dir)
        self.env = os.getenv('APP_ENV', 'development')
        self._config = self._load_config()
    
    def _load_dotenv(self, filepath: Path) -> Dict[str, str]:
        """อ่าน .env file"""
        config = {}
        if not filepath.exists():
            return config
        
        with open(filepath, 'r') as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith('#'):
                    continue
                if '=' in line:
                    key, _, value = line.partition('=')
                    value = value.strip('"\'')
                    config[key.strip()] = value
        return config
    
    def _load_config(self) -> Dict[str, str]:
        # เรียงลำดับ: base -> environment-specific -> local override
        config = {}
        
        # Load base .env
        config.update(self._load_dotenv(self.base_dir / '.env'))
        
        # Load environment-specific .env
        env_file = self.base_dir / f'.env.{self.env}'
        config.update(self._load_dotenv(env_file))
        
        # Load local override (ไม่ควร commit)
        config.update(self._load_dotenv(self.base_dir / '.env.local'))
        
        # Actual environment variables override all
        config.update(os.environ)
        
        return config
    
    def get(self, key: str, default: Any = None, required: bool = False) -> Any:
        value = self._config.get(key, default)
        if required and value is None:
            raise ValueError(f"Required config '{key}' not found in environment '{self.env}'")
        return value
    
    def get_bool(self, key: str, default: bool = False) -> bool:
        value = self.get(key, str(default))
        return value.lower() in ('true', '1', 'yes', 'on')
    
    def get_int(self, key: str, default: int = 0) -> int:
        try:
            return int(self.get(key, default))
        except (TypeError, ValueError):
            return default
    
    def get_list(self, key: str, separator: str = ',') -> list:
        value = self.get(key, '')
        return [v.strip() for v in value.split(separator) if v.strip()]
    
    @property
    def is_development(self):
        return self.env == 'development'
    
    @property
    def is_production(self):
        return self.env == 'production'

# สร้าง .env สำหรับ test
with open('.env', 'w') as f:
    f.write("DATABASE_URL=sqlite:///dev.db\nDEBUG=True\nSECRET_KEY=test-key-123456789012345678\n")

config = MultiEnvConfig()
print(f"Environment: {config.env}")
print(f"Debug: {config.get_bool('DEBUG')}")
print(f"DB: {config.get('DATABASE_URL')}")
```

---

### ข้อที่ 5: Poetry Project Setup Script

**เฉลย:**
```python
import subprocess
import sys
import os
from pathlib import Path

def setup_poetry_project(project_name: str, python_version: str = "3.11"):
    """Script สร้าง Poetry project"""
    
    print(f"กำลังสร้าง Poetry project: {project_name}")
    
    # ตรวจสอบ Poetry
    result = subprocess.run(['poetry', '--version'], capture_output=True, text=True)
    if result.returncode != 0:
        print("ไม่พบ Poetry กรุณาติดตั้งก่อน:")
        print("curl -sSL https://install.python-poetry.org | python3 -")
        sys.exit(1)
    
    print(f"Poetry version: {result.stdout.strip()}")
    
    # สร้าง project
    result = subprocess.run(
        ['poetry', 'new', project_name],
        capture_output=True, text=True
    )
    
    if result.returncode != 0:
        print(f"Error: {result.stderr}")
        return
    
    project_dir = Path(project_name)
    
    # เพิ่ม common packages
    os.chdir(project_dir)
    
    packages_to_add = ['requests', 'python-dotenv', 'pydantic']
    dev_packages = ['pytest', 'pytest-cov', 'black', 'flake8', 'mypy', 'isort']
    
    print("\nติดตั้ง packages...")
    for pkg in packages_to_add:
        subprocess.run(['poetry', 'add', pkg], capture_output=True)
        print(f"  + {pkg}")
    
    for pkg in dev_packages:
        subprocess.run(['poetry', 'add', '--group', 'dev', pkg], capture_output=True)
        print(f"  + {pkg} (dev)")
    
    # สร้าง .env.example
    with open('.env.example', 'w') as f:
        f.write("APP_ENV=development\nSECRET_KEY=change-me\nDEBUG=True\n")
    
    # สร้าง .gitignore
    with open('.gitignore', 'w') as f:
        f.write("__pycache__/\n*.pyc\n.venv/\n.env\n.coverage\ndist/\nbuild/\n")
    
    print(f"\nสร้าง Poetry project '{project_name}' สำเร็จ!")
    print(f"\nต่อไป:")
    print(f"  cd {project_name}")
    print(f"  poetry shell")
    print(f"  cp .env.example .env")
    print(f"  poetry run pytest")

# ตัวอย่างการใช้งาน (ไม่รันในที่นี้ เพราะจะสร้างไฟล์จริง)
print("Script พร้อมใช้งาน!")
print("เรียกใช้: setup_poetry_project('my-project')")
```

---

## สรุป

ในส่วนนี้เราได้เรียนรู้:

1. **venv**: Virtual environment มาตรฐานของ Python - ง่าย built-in
2. **pip**: Package manager - คำสั่งพื้นฐานที่ต้องรู้
3. **requirements.txt**: การจัดการ dependencies แบบดั้งเดิม
4. **pip-tools**: สร้าง lock files สำหรับ reproducible installs
5. **pipenv**: ผสม virtualenv + package management
6. **Poetry**: Modern tool - แนะนำสำหรับโปรเจกต์ใหม่
7. **pyproject.toml**: มาตรฐานใหม่สำหรับ Python project configuration
8. **conda**: เหมาะสำหรับ data science
9. **python-dotenv**: จัดการ environment variables
10. **Best practices**: โครงสร้าง project, .gitignore, pre-commit hooks

สำหรับโปรเจกต์ใหม่ แนะนำใช้ Poetry + pyproject.toml เพราะเป็น modern standard ที่ PyPA แนะนำ!
