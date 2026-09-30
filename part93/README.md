# Part 93: CI/CD - GitHub Actions & GitLab CI

## บทนำ

CI/CD (Continuous Integration / Continuous Deployment) คือแนวทางปฏิบัติในการพัฒนา software ที่ช่วยให้ทีมสามารถ deliver code ได้อย่างรวดเร็ว ปลอดภัย และเชื่อถือได้ โดยอัตโนมัติ ตั้งแต่การ test ไปจนถึงการ deploy ขึ้น production

---

## 1. CI/CD Concepts และ Benefits

### ทำไมต้องใช้ CI/CD?

| ปัญหาแบบเดิม | แก้ด้วย CI/CD |
|--------------|---------------|
| Integration hell เมื่อรวม code | Integrate บ่อยครั้ง ตรวจพบปัญหาเร็ว |
| Manual testing ใช้เวลานาน | Automated tests ทำงานอัตโนมัติ |
| Deploy มีความเสี่ยงสูง | Automated deployment ทำซ้ำได้เหมือนกันทุกครั้ง |
| Feedback loop ช้า | Developer รู้ผลทันทีหลัง push |
| Rollback ยุ่งยาก | Version control + automated rollback |

### CI/CD Pipeline ทำงานอย่างไร?

```
Developer → Push Code → CI Server → Build → Test → Security Scan → Deploy → Monitor
                ↓
        GitHub Actions / GitLab CI / Jenkins / CircleCI
```

### ตัวอย่างที่ 1: CI/CD Pipeline Simulation ใน Python

```python
# ตัวอย่าง 1: จำลอง CI/CD Pipeline
from dataclasses import dataclass, field
from typing import List, Optional, Callable
from enum import Enum
import time
import subprocess
import sys

class StageStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    SUCCESS = "success"
    FAILED = "failed"
    SKIPPED = "skipped"

@dataclass
class PipelineStage:
    name: str
    command: Optional[str] = None
    action: Optional[Callable] = None
    depends_on: List[str] = field(default_factory=list)
    status: StageStatus = StageStatus.PENDING
    duration: float = 0.0
    output: str = ""

class CIPipeline:
    """จำลอง CI/CD Pipeline"""
    
    def __init__(self, name: str):
        self.name = name
        self.stages: List[PipelineStage] = []
        self.start_time = 0.0
    
    def add_stage(self, stage: PipelineStage):
        self.stages.append(stage)
    
    def run(self) -> bool:
        self.start_time = time.time()
        print(f"\n{'='*60}")
        print(f"Pipeline: {self.name}")
        print(f"{'='*60}")
        
        for stage in self.stages:
            # ตรวจสอบ dependencies
            for dep in stage.depends_on:
                dep_stage = next((s for s in self.stages if s.name == dep), None)
                if dep_stage and dep_stage.status == StageStatus.FAILED:
                    stage.status = StageStatus.SKIPPED
                    print(f"  ⏭  {stage.name}: SKIPPED (dependency {dep} failed)")
                    continue
            
            if stage.status == StageStatus.SKIPPED:
                continue
                
            stage.status = StageStatus.RUNNING
            print(f"\n  ▶  {stage.name}...")
            
            start = time.time()
            try:
                if stage.action:
                    stage.action()
                    stage.status = StageStatus.SUCCESS
                elif stage.command:
                    result = subprocess.run(
                        stage.command,
                        shell=True,
                        capture_output=True,
                        text=True,
                        timeout=60
                    )
                    stage.output = result.stdout
                    if result.returncode == 0:
                        stage.status = StageStatus.SUCCESS
                    else:
                        stage.status = StageStatus.FAILED
                        stage.output += result.stderr
                        
            except Exception as e:
                stage.status = StageStatus.FAILED
                stage.output = str(e)
            
            stage.duration = time.time() - start
            
            status_icon = "✅" if stage.status == StageStatus.SUCCESS else "❌"
            print(f"  {status_icon} {stage.name}: {stage.status.value} ({stage.duration:.2f}s)")
        
        total_time = time.time() - self.start_time
        success = all(s.status in [StageStatus.SUCCESS, StageStatus.SKIPPED] 
                      for s in self.stages)
        
        print(f"\n{'='*60}")
        print(f"Pipeline {'PASSED ✅' if success else 'FAILED ❌'} in {total_time:.2f}s")
        print(f"{'='*60}\n")
        return success

# ใช้งาน
def install_deps():
    print("    Installing dependencies...")
    time.sleep(0.1)

def run_tests():
    print("    Running pytest...")
    time.sleep(0.2)

def check_lint():
    print("    Running flake8...")
    time.sleep(0.1)

pipeline = CIPipeline("My Python App")
pipeline.add_stage(PipelineStage("install", action=install_deps))
pipeline.add_stage(PipelineStage("lint", action=check_lint, depends_on=["install"]))
pipeline.add_stage(PipelineStage("test", action=run_tests, depends_on=["install"]))
pipeline.run()
```

---

## 2. GitHub Actions: Workflow YAML Syntax

### โครงสร้างหลักของ Workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

# Triggers: กำหนดว่า workflow จะทำงานเมื่อไหร่
on:
  push:
    branches: [ main, develop ]
    paths:
      - '**.py'
      - 'requirements*.txt'
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 6 * * 1'   # ทุกวันจันทร์ 6:00 UTC
  workflow_dispatch:        # trigger ด้วยมือได้

# Environment variables ระดับ workflow
env:
  PYTHON_VERSION: '3.11'
  POETRY_VERSION: '1.6.1'

# Jobs: งานที่ทำงานแบบ parallel หรือ sequential
jobs:
  lint:
    name: Lint & Format Check
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
      
      - name: Install linting tools
        run: |
          pip install flake8 black isort mypy
      
      - name: Run flake8
        run: flake8 . --max-line-length=88 --exclude=.git,__pycache__
      
      - name: Check black formatting
        run: black --check .
      
      - name: Check import sorting
        run: isort --check-only .

  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: lint   # รอให้ lint ผ่านก่อน
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
      
      - name: Install dependencies
        run: pip install -r requirements.txt -r requirements-dev.txt
      
      - name: Run tests with coverage
        run: |
          pytest tests/ \
            --cov=app \
            --cov-report=xml \
            --cov-report=html \
            --cov-fail-under=80
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: htmlcov/
```

### ตัวอย่างที่ 2: เข้าใจ YAML Syntax ด้วย Python

```python
# ตัวอย่าง 2: parse และ validate GitHub Actions YAML
import yaml
import json
from typing import Dict, Any, List

def parse_workflow(yaml_content: str) -> Dict[str, Any]:
    """Parse GitHub Actions workflow YAML"""
    return yaml.safe_load(yaml_content)

def validate_workflow(workflow: Dict[str, Any]) -> List[str]:
    """ตรวจสอบ workflow ว่าถูกต้องหรือไม่"""
    errors = []
    
    # ตรวจสอบ required fields
    if 'name' not in workflow:
        errors.append("Missing 'name' field")
    
    if 'on' not in workflow:
        errors.append("Missing 'on' trigger field")
    
    if 'jobs' not in workflow:
        errors.append("Missing 'jobs' field")
        return errors
    
    # ตรวจสอบแต่ละ job
    for job_id, job in workflow['jobs'].items():
        if 'runs-on' not in job:
            errors.append(f"Job '{job_id}': missing 'runs-on'")
        
        if 'steps' not in job or not job['steps']:
            errors.append(f"Job '{job_id}': missing or empty 'steps'")
            continue
        
        for i, step in enumerate(job['steps']):
            if 'uses' not in step and 'run' not in step:
                errors.append(f"Job '{job_id}', step {i+1}: must have 'uses' or 'run'")
    
    return errors

def summarize_workflow(workflow: Dict[str, Any]) -> None:
    """แสดงสรุปของ workflow"""
    print(f"Workflow: {workflow.get('name', 'Unnamed')}")
    print(f"Triggers: {list(workflow.get('on', {}).keys())}")
    print(f"\nJobs:")
    
    for job_id, job in workflow.get('jobs', {}).items():
        steps = job.get('steps', [])
        needs = job.get('needs', [])
        print(f"  - {job_id} ({job.get('runs-on', 'unknown')})")
        if needs:
            print(f"    Depends on: {needs}")
        print(f"    Steps: {len(steps)}")

# ตัวอย่าง workflow
sample_workflow = """
name: Example CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest tests/
"""

workflow = parse_workflow(sample_workflow)
errors = validate_workflow(workflow)
if errors:
    print("Errors found:")
    for e in errors:
        print(f"  - {e}")
else:
    print("Workflow is valid!")
    summarize_workflow(workflow)
```

---

## 3. GitHub Actions: Triggers (on)

### ประเภทของ Triggers

```yaml
# ตัวอย่างที่ 3: Triggers ต่างๆ

# 1. Push trigger
on:
  push:
    branches:
      - main
      - 'release/**'
      - '!hotfix/**'    # ยกเว้น branches ที่ขึ้นต้นด้วย hotfix
    tags:
      - 'v*.*.*'        # version tags
    paths:
      - 'src/**'
      - '!docs/**'      # ยกเว้น docs

# 2. Pull Request trigger
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
    branches: [main, develop]

# 3. Scheduled trigger
on:
  schedule:
    - cron: '0 2 * * *'   # ทุกวัน 02:00 UTC
    - cron: '0 6 * * 1'   # ทุกวันจันทร์ 06:00 UTC

# 4. Manual trigger with inputs
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy to which environment?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      run_tests:
        description: 'Run tests before deploy?'
        type: boolean
        default: true
      version:
        description: 'Version to deploy'
        type: string

# 5. External events
on:
  repository_dispatch:
    types: [deploy-trigger, test-trigger]

# 6. Reusable workflow call
on:
  workflow_call:
    inputs:
      python-version:
        required: true
        type: string
    secrets:
      token:
        required: true
```

### ตัวอย่างที่ 4: Script สำหรับสร้าง Workflow Trigger Configuration

```python
# ตัวอย่าง 4: สร้าง workflow trigger configuration แบบ programmatic
import yaml
from dataclasses import dataclass, field
from typing import List, Optional, Dict, Any

@dataclass
class WorkflowTrigger:
    """Builder สำหรับ GitHub Actions triggers"""
    
    _config: Dict[str, Any] = field(default_factory=dict)
    
    def on_push(self, branches: List[str] = None, 
                paths: List[str] = None,
                tags: List[str] = None) -> 'WorkflowTrigger':
        push_config = {}
        if branches:
            push_config['branches'] = branches
        if paths:
            push_config['paths'] = paths
        if tags:
            push_config['tags'] = tags
        self._config['push'] = push_config
        return self
    
    def on_pull_request(self, branches: List[str] = None,
                        types: List[str] = None) -> 'WorkflowTrigger':
        pr_config = {}
        if branches:
            pr_config['branches'] = branches
        if types:
            pr_config['types'] = types
        self._config['pull_request'] = pr_config
        return self
    
    def on_schedule(self, cron: str) -> 'WorkflowTrigger':
        if 'schedule' not in self._config:
            self._config['schedule'] = []
        self._config['schedule'].append({'cron': cron})
        return self
    
    def on_manual(self, inputs: Dict[str, Any] = None) -> 'WorkflowTrigger':
        config = {}
        if inputs:
            config['inputs'] = inputs
        self._config['workflow_dispatch'] = config
        return self
    
    def build(self) -> Dict[str, Any]:
        return self._config

# สร้าง trigger configuration
trigger = (WorkflowTrigger()
    .on_push(branches=['main', 'develop'], paths=['src/**', 'tests/**'])
    .on_pull_request(branches=['main'], types=['opened', 'synchronize'])
    .on_schedule('0 2 * * *')
    .on_manual()
    .build())

print("Generated 'on' configuration:")
print(yaml.dump({'on': trigger}, default_flow_style=False))
```

---

## 4. Actions Marketplace

### actions/checkout และ actions/setup-python

```yaml
# ตัวอย่างที่ 5: Actions Marketplace - การใช้งาน actions ที่สำคัญ

jobs:
  setup-example:
    runs-on: ubuntu-latest
    steps:
      # Checkout: ดึง source code มาใน runner
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0          # ดึง full history (สำหรับ semantic versioning)
          submodules: recursive   # รวม submodules ด้วย
          token: ${{ secrets.GITHUB_TOKEN }}
      
      # Setup Python: ติดตั้งและตั้งค่า Python
      - name: Set up Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'                    # cache pip dependencies
          cache-dependency-path: |        # ไฟล์ที่ใช้ generate cache key
            requirements.txt
            requirements-dev.txt
      
      # Cache: cache ไฟล์เพื่อความเร็ว
      - name: Cache Python packages
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      # Upload artifacts
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()    # upload แม้ test จะ fail
        with:
          name: test-results
          path: |
            test-results.xml
            coverage.xml
          retention-days: 30
      
      # Download artifacts (ใน job อื่น)
      - name: Download test results
        uses: actions/download-artifact@v3
        with:
          name: test-results
          path: ./downloaded-results
      
      # สร้าง GitHub Release
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
      
      # Slack notification
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment ${{ job.status }}!'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

### ตัวอย่างที่ 6: ใช้ Custom Action

```python
# action.py - Custom GitHub Action ที่เขียนด้วย Python
#!/usr/bin/env python3
"""Custom GitHub Action สำหรับ version bumping"""
import os
import sys
import re
import subprocess
from pathlib import Path

def get_input(name: str, required: bool = False) -> str:
    """ดึง action input จาก environment variable"""
    value = os.environ.get(f'INPUT_{name.upper().replace("-", "_")}', '')
    if required and not value:
        print(f"::error::Input '{name}' is required")
        sys.exit(1)
    return value

def set_output(name: str, value: str):
    """ตั้งค่า action output"""
    with open(os.environ.get('GITHUB_OUTPUT', '/dev/stdout'), 'a') as f:
        f.write(f"{name}={value}\n")

def add_summary(markdown: str):
    """เพิ่ม step summary"""
    with open(os.environ.get('GITHUB_STEP_SUMMARY', '/dev/stdout'), 'a') as f:
        f.write(markdown + '\n')

def bump_version(current: str, bump_type: str) -> str:
    """เพิ่ม version number"""
    parts = current.lstrip('v').split('.')
    major, minor, patch = int(parts[0]), int(parts[1]), int(parts[2])
    
    if bump_type == 'major':
        major += 1; minor = 0; patch = 0
    elif bump_type == 'minor':
        minor += 1; patch = 0
    elif bump_type == 'patch':
        patch += 1
    
    return f"v{major}.{minor}.{patch}"

def main():
    bump_type = get_input('bump-type', required=True)
    version_file = get_input('version-file') or 'version.txt'
    
    # อ่าน current version
    try:
        current_version = Path(version_file).read_text().strip()
    except FileNotFoundError:
        current_version = 'v0.0.0'
    
    # คำนวณ new version
    new_version = bump_version(current_version, bump_type)
    
    # เขียน version ใหม่
    Path(version_file).write_text(new_version + '\n')
    
    # ตั้งค่า outputs
    set_output('new-version', new_version)
    set_output('old-version', current_version)
    
    # เพิ่ม summary
    add_summary(f"""
## Version Bump
| | Version |
|--|--|
| Old | `{current_version}` |
| New | `{new_version}` |
| Type | {bump_type} |
""")
    
    print(f"Bumped version: {current_version} → {new_version}")

if __name__ == '__main__':
    main()
```

```yaml
# action.yml - Action definition
name: 'Python Version Bumper'
description: 'Bumps semantic version numbers'
inputs:
  bump-type:
    description: 'Type of bump: major, minor, patch'
    required: true
    default: 'patch'
  version-file:
    description: 'Path to version file'
    required: false
    default: 'version.txt'
outputs:
  new-version:
    description: 'The new version number'
  old-version:
    description: 'The previous version number'
runs:
  using: 'python3'
  main: 'action.py'
```

---

## 5. Testing Pipeline: pytest, Coverage, Flake8, Black, Mypy

### ตัวอย่างที่ 7: Complete Testing Workflow

```yaml
# .github/workflows/test.yml
name: Testing Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install quality tools
        run: |
          pip install --upgrade pip
          pip install flake8 black isort mypy pylint
          pip install -r requirements.txt
      
      # Flake8 - PEP 8 style checking
      - name: Run Flake8
        run: |
          flake8 . \
            --count \
            --select=E9,F63,F7,F82 \
            --show-source \
            --statistics
          flake8 . \
            --count \
            --exit-zero \
            --max-complexity=10 \
            --max-line-length=88 \
            --statistics
      
      # Black - code formatter
      - name: Check Black formatting
        run: black --check --diff .
      
      # isort - import sorting
      - name: Check import order with isort
        run: isort --check-only --diff .
      
      # mypy - static type checking
      - name: Run Mypy type checking
        run: |
          mypy src/ \
            --ignore-missing-imports \
            --strict \
            --show-error-codes
        continue-on-error: true
      
      # pylint - comprehensive linting
      - name: Run Pylint
        run: |
          pylint src/ \
            --fail-under=8.0 \
            --output-format=colorized
        continue-on-error: true
  
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    needs: quality
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: --health-cmd "redis-cli ping"
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio pytest-xdist httpx
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
          ENVIRONMENT: test
        run: |
          pytest tests/ \
            --cov=src \
            --cov-branch \
            --cov-report=xml:coverage.xml \
            --cov-report=html:coverage_html \
            --cov-report=term-missing \
            --cov-fail-under=80 \
            -v \
            --tb=short \
            --junitxml=test-results.xml \
            -n auto    # parallel testing
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: coverage.xml
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: true
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: |
            test-results.xml
            coverage.xml
            coverage_html/
```

### ตัวอย่างที่ 8: pytest Configuration

```python
# conftest.py - pytest configuration
import pytest
import asyncio
from typing import AsyncGenerator, Generator
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import Base, get_db

# Test database
SQLALCHEMY_TEST_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_TEST_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture(scope="session")
def db_engine():
    """สร้าง test database"""
    Base.metadata.create_all(bind=engine)
    yield engine
    Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def db_session(db_engine):
    """สร้าง database session สำหรับแต่ละ test"""
    connection = db_engine.connect()
    transaction = connection.begin()
    session = TestingSessionLocal(bind=connection)
    
    yield session
    
    session.close()
    transaction.rollback()
    connection.close()

@pytest.fixture(scope="function")
def client(db_session) -> Generator:
    """สร้าง test client"""
    def override_get_db():
        yield db_session
    
    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()

@pytest.fixture
def sample_user(db_session):
    """สร้าง test user"""
    from app.models import User
    user = User(email="test@example.com", name="Test User")
    db_session.add(user)
    db_session.commit()
    db_session.refresh(user)
    return user
```

```python
# pytest.ini หรือ pyproject.toml
# pyproject.toml
"""
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--tb=short",
    "--strict-markers",
]
markers = [
    "slow: marks tests as slow",
    "integration: marks tests as integration tests",
    "unit: marks tests as unit tests",
]

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise NotImplementedError",
]
"""
```

---

## 6. Docker Build และ Push to Registry

### ตัวอย่างที่ 9: Docker Build และ Push Workflow

```yaml
# .github/workflows/docker.yml
name: Docker Build & Push

on:
  push:
    branches: [main]
    tags: ['v*.*.*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      # ตั้งค่า QEMU สำหรับ multi-platform builds
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      # ตั้งค่า Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # Login ไปยัง GitHub Container Registry
      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      # Login ไปยัง DockerHub ด้วย (optional)
      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      # สร้าง Docker image metadata (tags, labels)
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
            myusername/myapp
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=sha-
      
      # Build และ Push
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha          # GitHub Actions cache
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_DATE=${{ github.event.repository.updated_at }}
            VCS_REF=${{ github.sha }}
      
      # Scan สำหรับ vulnerabilities
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

### ตัวอย่างที่ 10: Dockerfile สำหรับ Python App

```dockerfile
# Dockerfile - multi-stage build สำหรับ Python app
# Stage 1: Builder
FROM python:3.11-slim AS builder

WORKDIR /build

# ติดตั้ง build dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# คัดลอกและติดตั้ง dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim AS runtime

WORKDIR /app

# ติดตั้ง runtime dependencies เท่านั้น
RUN apt-get update && apt-get install -y \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy installed packages จาก builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY src/ ./src/
COPY alembic/ ./alembic/
COPY alembic.ini .

# สร้าง non-root user
RUN useradd -m -u 1001 appuser && chown -R appuser:appuser /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000

CMD ["python", "-m", "uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 7. Deployment Workflows (AWS, GCP, Heroku)

### ตัวอย่างที่ 11: Deploy to AWS Elastic Beanstalk

```yaml
# .github/workflows/deploy-aws.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image to ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-python-app
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: .aws/task-definition.json
          service: my-service
          cluster: my-cluster
          wait-for-service-stability: true
```

### ตัวอย่างที่ 12: Deploy to Google Cloud Run

```yaml
# .github/workflows/deploy-gcp.yml
name: Deploy to Cloud Run

on:
  push:
    branches: [main]

env:
  PROJECT_ID: my-gcp-project
  SERVICE: my-python-app
  REGION: asia-southeast1

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
      
      - name: Configure Docker for GCR
        run: gcloud auth configure-docker
      
      - name: Build and Push to GCR
        run: |
          docker build -t gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.sha }} .
          docker push gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.sha }}
      
      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy ${{ env.SERVICE }} \
            --image gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.sha }} \
            --platform managed \
            --region ${{ env.REGION }} \
            --allow-unauthenticated \
            --set-env-vars "ENVIRONMENT=production" \
            --min-instances 1 \
            --max-instances 100 \
            --memory 512Mi \
            --cpu 1
```

### ตัวอย่างที่ 13: Deploy to Heroku

```yaml
# .github/workflows/deploy-heroku.yml
name: Deploy to Heroku

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Deploy to Heroku
        uses: akhileshns/heroku-deploy@v3.12.14
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          heroku_app_name: "my-python-app"
          heroku_email: "dev@example.com"
          usedocker: true
          docker_build_args: |
            NODE_ENV
        env:
          NODE_ENV: production
      
      # หรือ deploy ด้วย Git push
      - name: Deploy via Git
        env:
          HEROKU_API_KEY: ${{ secrets.HEROKU_API_KEY }}
        run: |
          git remote add heroku https://heroku:$HEROKU_API_KEY@git.heroku.com/my-app.git
          git push heroku main
```

---

## 8. Secrets Management ใน GitHub Actions

### ตัวอย่างที่ 14: การจัดการ Secrets อย่างถูกต้อง

```yaml
# การใช้ secrets ใน workflow
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # ใช้ Environment secrets
    
    steps:
      # ใช้ secrets โดยตรง
      - name: Use secret
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          # secrets จะถูก mask ใน logs อัตโนมัติ
          echo "Connecting to database..."
          # ไม่ควร echo secrets โดยตรง!
      
      # ใช้ AWS Secrets Manager
      - name: Get secrets from AWS Secrets Manager
        uses: aws-actions/aws-secretsmanager-get-secrets@v1
        with:
          secret-ids: |
            prod/myapp/database
            prod/myapp/api-keys
          parse-json-secrets: true
      
      # สร้าง .env file จาก secrets
      - name: Create .env file
        run: |
          cat << EOF > .env
          DATABASE_URL=${{ secrets.DATABASE_URL }}
          SECRET_KEY=${{ secrets.SECRET_KEY }}
          REDIS_URL=${{ secrets.REDIS_URL }}
          EOF
      
      # ลบ secrets file หลังใช้งาน
      - name: Cleanup
        if: always()
        run: rm -f .env
```

### ตัวอย่างที่ 15: Python script จัดการ Secrets

```python
# ตัวอย่าง 15: จัดการ secrets ใน Python application
import os
import json
import boto3
from typing import Optional, Dict, Any
from functools import lru_cache

class SecretsManager:
    """จัดการ secrets จากหลาย sources"""
    
    def __init__(self, environment: str = "development"):
        self.environment = environment
        self._cache: Dict[str, Any] = {}
    
    def get(self, key: str, default: Optional[str] = None) -> Optional[str]:
        """ดึง secret จาก environment variable ก่อน"""
        return os.environ.get(key, default)
    
    def get_aws_secret(self, secret_name: str, region: str = "ap-southeast-1") -> Dict:
        """ดึง secret จาก AWS Secrets Manager"""
        if secret_name in self._cache:
            return self._cache[secret_name]
        
        client = boto3.client('secretsmanager', region_name=region)
        
        try:
            response = client.get_secret_value(SecretId=secret_name)
            secret = json.loads(response['SecretString'])
            self._cache[secret_name] = secret
            return secret
        except Exception as e:
            raise ValueError(f"Cannot get secret '{secret_name}': {e}")
    
    @property
    def database_url(self) -> str:
        """สร้าง database URL"""
        if self.environment == "production":
            secret = self.get_aws_secret(f"prod/myapp/database")
            return (f"postgresql://{secret['username']}:{secret['password']}"
                   f"@{secret['host']}:{secret['port']}/{secret['database']}")
        
        return self.get('DATABASE_URL', 'sqlite:///./dev.db')
    
    def validate_required(self, *keys: str) -> None:
        """ตรวจสอบว่า required secrets มีครบ"""
        missing = [k for k in keys if not self.get(k)]
        if missing:
            raise EnvironmentError(
                f"Missing required secrets: {', '.join(missing)}"
            )

# ใช้งาน
secrets = SecretsManager(environment=os.getenv('ENVIRONMENT', 'development'))

# ตรวจสอบ required secrets ตอน startup
if secrets.environment == 'production':
    secrets.validate_required('SECRET_KEY', 'DATABASE_URL', 'REDIS_URL')

print(f"Environment: {secrets.environment}")
print(f"Database configured: {'Yes' if secrets.database_url else 'No'}")
```

---

## 9. Matrix Builds (Multiple Python Versions)

### ตัวอย่างที่ 16: Matrix Build Configuration

```yaml
# .github/workflows/matrix.yml
name: Matrix Build

on: [push, pull_request]

jobs:
  test:
    name: Test Python ${{ matrix.python-version }} on ${{ matrix.os }}
    
    strategy:
      fail-fast: false    # อย่าหยุดทุก matrix เมื่อหนึ่งล้มเหลว
      matrix:
        python-version: ['3.9', '3.10', '3.11', '3.12']
        os: [ubuntu-latest, windows-latest, macos-latest]
        # ยกเว้นบาง combinations
        exclude:
          - os: windows-latest
            python-version: '3.9'
        # เพิ่ม specific configurations
        include:
          - python-version: '3.12'
            os: ubuntu-latest
            run-coverage: true
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest
          pip install -r requirements.txt
      
      - name: Run tests
        run: pytest tests/ -v
      
      # Coverage เฉพาะ Python 3.12 บน Ubuntu
      - name: Run with coverage
        if: matrix.run-coverage == true
        run: pytest tests/ --cov=src --cov-report=xml
      
      - name: Upload coverage
        if: matrix.run-coverage == true
        uses: codecov/codecov-action@v3
  
  # Summarize matrix results
  summary:
    needs: test
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - name: Check matrix results
        run: |
          if [ "${{ needs.test.result }}" == "success" ]; then
            echo "All matrix builds passed! ✅"
          else
            echo "Some matrix builds failed! ❌"
            exit 1
          fi
```

### ตัวอย่างที่ 17: Python Version Compatibility Checker

```python
# ตัวอย่าง 17: ตรวจสอบ Python version compatibility
import sys
import platform
from typing import List, Tuple

def check_python_version(
    min_version: Tuple[int, int] = (3, 9),
    max_version: Tuple[int, int] = (3, 12)
) -> bool:
    """ตรวจสอบว่า Python version ที่ใช้อยู่รองรับหรือไม่"""
    current = sys.version_info[:2]
    
    if current < min_version:
        print(f"Python {'.'.join(map(str, min_version))}+ required, "
              f"got {'.'.join(map(str, current))}")
        return False
    
    if current > max_version:
        print(f"Warning: Python {'.'.join(map(str, current))} may not be tested")
        return True
    
    print(f"Python {'.'.join(map(str, current))}: Compatible ✅")
    return True

def get_environment_info() -> dict:
    """รวบรวมข้อมูล environment สำหรับ CI"""
    return {
        'python_version': sys.version,
        'python_version_info': {
            'major': sys.version_info.major,
            'minor': sys.version_info.minor,
            'micro': sys.version_info.micro,
        },
        'platform': platform.platform(),
        'architecture': platform.architecture()[0],
        'processor': platform.processor(),
        'os_name': platform.system(),
    }

# ตรวจสอบเมื่อ run ใน CI
if __name__ == '__main__':
    is_compatible = check_python_version()
    
    info = get_environment_info()
    print(f"\nEnvironment Info:")
    for key, value in info.items():
        if isinstance(value, dict):
            for k, v in value.items():
                print(f"  {key}.{k}: {v}")
        else:
            print(f"  {key}: {value}")
    
    sys.exit(0 if is_compatible else 1)
```

---

## 10. GitLab CI/CD Comparison

### เปรียบเทียบ GitHub Actions กับ GitLab CI

| Feature | GitHub Actions | GitLab CI/CD |
|---------|---------------|--------------|
| Config file | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| Runners | GitHub-hosted / Self-hosted | GitLab.com / Self-hosted |
| Registry | GitHub Container Registry | GitLab Container Registry |
| Triggers | `on:` keyword | `rules:` / `only:` / `except:` |
| Variables | `secrets` + `vars` | `variables` |
| Artifacts | `upload-artifact` action | `artifacts:` keyword |
| Cache | `actions/cache` action | `cache:` keyword |
| Environments | `environment:` keyword | `environment:` keyword |

### ตัวอย่างที่ 18: GitLab CI/CD Configuration

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.pip-cache"

# Global cache configuration
.python-cache:
  cache:
    key:
      files:
        - requirements.txt
        - requirements-dev.txt
    paths:
      - .pip-cache/
    policy: pull-push

# Templates (reusable)
.setup-python:
  image: python:3.11-slim
  before_script:
    - pip install --cache-dir $PIP_CACHE_DIR -r requirements.txt

# Build stage
build:
  stage: build
  extends: .setup-python
  script:
    - pip install build
    - python -m build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# Lint
lint:
  stage: test
  extends: .setup-python
  script:
    - pip install flake8 black mypy
    - flake8 src/
    - black --check src/
    - mypy src/ --ignore-missing-imports
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

# Unit Tests
unit-tests:
  stage: test
  extends:
    - .setup-python
    - .python-cache
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    DATABASE_URL: postgresql://testuser:testpass@postgres/testdb
    REDIS_URL: redis://redis:6379
  script:
    - pip install pytest pytest-cov pytest-asyncio
    - pytest tests/ --cov=src --cov-report=xml --cov-report=term
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    when: always

# Security Scan
security-scan:
  stage: security
  image: python:3.11-slim
  script:
    - pip install safety bandit
    - safety check -r requirements.txt
    - bandit -r src/ -f json -o bandit-report.json
  artifacts:
    paths:
      - bandit-report.json
    when: always
  allow_failure: true

# Deploy to staging
deploy-staging:
  stage: deploy
  image: google/cloud-sdk:alpine
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - develop
  script:
    - echo $GCP_SA_KEY | base64 -d > /tmp/sa-key.json
    - gcloud auth activate-service-account --key-file=/tmp/sa-key.json
    - gcloud run deploy myapp --image gcr.io/$GCP_PROJECT/myapp:$CI_COMMIT_SHA
      --region asia-southeast1 --platform managed

# Deploy to production (manual approval)
deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
  when: manual    # ต้องกด approve ด้วยมือ
  script:
    - echo "Deploying to production..."
```

---

## 11. Pre-commit Hooks

### ตัวอย่างที่ 19: Pre-commit Framework

```yaml
# .pre-commit-config.yaml
repos:
  # Python formatters
  - repo: https://github.com/psf/black
    rev: 23.11.0
    hooks:
      - id: black
        language_version: python3.11
  
  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]
  
  # Linting
  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        args: ['--max-line-length=88', '--extend-ignore=E203']
        additional_dependencies: [
          flake8-bugbear,
          flake8-comprehensions,
          flake8-simplify,
        ]
  
  # Type checking
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests, types-redis]
  
  # Security
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.5
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
        additional_dependencies: ["bandit[toml]"]
  
  # General file checks
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
      - id: debug-statements
      - id: no-commit-to-branch
        args: ['--branch', 'main', '--branch', 'master']
  
  # Commit message convention
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.0.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: [feat, fix, docs, style, refactor, test, chore]
  
  # Secrets detection
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

### ตัวอย่างที่ 20: ตั้งค่า Pre-commit ด้วย Python Script

```python
# ตัวอย่าง 20: script ตั้งค่า pre-commit hooks
import subprocess
import sys
import os
from pathlib import Path

def run_command(cmd: list, check: bool = True) -> subprocess.CompletedProcess:
    """รัน command และ handle errors"""
    print(f"  Running: {' '.join(cmd)}")
    result = subprocess.run(cmd, capture_output=True, text=True)
    if check and result.returncode != 0:
        print(f"Error: {result.stderr}")
        sys.exit(1)
    return result

def setup_precommit():
    """ตั้งค่า pre-commit hooks"""
    print("Setting up pre-commit hooks...")
    
    # ตรวจสอบว่ามี .pre-commit-config.yaml
    if not Path('.pre-commit-config.yaml').exists():
        print("Error: .pre-commit-config.yaml not found!")
        sys.exit(1)
    
    # ติดตั้ง pre-commit
    print("\n1. Installing pre-commit...")
    run_command([sys.executable, '-m', 'pip', 'install', 'pre-commit'])
    
    # ติดตั้ง hooks
    print("\n2. Installing git hooks...")
    run_command(['pre-commit', 'install'])
    run_command(['pre-commit', 'install', '--hook-type', 'commit-msg'])
    
    # รัน hooks บนทุกไฟล์ครั้งแรก
    print("\n3. Running pre-commit on all files (first time)...")
    result = run_command(['pre-commit', 'run', '--all-files'], check=False)
    if result.returncode != 0:
        print("\nSome hooks failed. Please fix the issues above and run again.")
        print("Files have been auto-fixed where possible.")
    else:
        print("\n✅ All pre-commit hooks passed!")
    
    # แสดงรายการ hooks
    print("\n4. Installed hooks:")
    result = run_command(['pre-commit', 'list'])
    print(result.stdout)

def update_hooks():
    """อัปเดต pre-commit hooks เป็น version ล่าสุด"""
    print("Updating pre-commit hooks...")
    run_command(['pre-commit', 'autoupdate'])
    print("✅ Hooks updated!")

if __name__ == '__main__':
    if len(sys.argv) > 1 and sys.argv[1] == 'update':
        update_hooks()
    else:
        setup_precommit()
```

---

## 12. ตัวอย่างโปรแกรมจริง: Complete CI/CD Pipeline สำหรับ FastAPI App

### โครงสร้างโปรเจกต์

```
fastapi-app/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       └── release.yml
├── src/
│   └── app/
│       ├── __init__.py
│       ├── main.py
│       ├── models.py
│       ├── routers/
│       └── services/
├── tests/
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── .pre-commit-config.yaml
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
└── Makefile
```

### ตัวอย่างที่ 21: FastAPI Application

```python
# src/app/main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr
from typing import List, Optional
import uvicorn

app = FastAPI(
    title="My FastAPI App",
    version="1.0.0",
    description="FastAPI app with complete CI/CD pipeline"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Models
class UserCreate(BaseModel):
    name: str
    email: EmailStr

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    
    class Config:
        from_attributes = True

# In-memory storage สำหรับตัวอย่าง
users_db: dict = {}
user_counter = 0

@app.get("/health")
async def health_check():
    return {"status": "healthy", "version": "1.0.0"}

@app.get("/users", response_model=List[UserResponse])
async def get_users():
    return list(users_db.values())

@app.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    global user_counter
    user_counter += 1
    new_user = {"id": user_counter, **user.dict()}
    users_db[user_counter] = new_user
    return new_user

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User {user_id} not found"
        )
    return users_db[user_id]

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### ตัวอย่างที่ 22: Complete CI/CD Workflow

```yaml
# .github/workflows/ci.yml - Complete CI Pipeline
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ========================================
  # Stage 1: Code Quality
  # ========================================
  quality:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install quality tools
        run: |
          pip install --upgrade pip
          pip install flake8 black isort mypy
          pip install -r requirements.txt
      
      - name: Run Black
        run: black --check --diff src/ tests/
      
      - name: Run isort
        run: isort --check-only --diff src/ tests/
      
      - name: Run Flake8
        run: flake8 src/ tests/ --max-line-length=88 --ignore=E203,W503
      
      - name: Run Mypy
        run: mypy src/ --ignore-missing-imports --strict
        continue-on-error: true
  
  # ========================================
  # Stage 2: Security Scan
  # ========================================
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install security tools
        run: pip install safety bandit semgrep
      
      - name: Check dependencies for vulnerabilities
        run: safety check -r requirements.txt --json > safety-report.json
        continue-on-error: true
      
      - name: Run Bandit security linter
        run: |
          bandit -r src/ \
            -f json \
            -o bandit-report.json \
            -ll    # medium and high severity only
        continue-on-error: true
      
      - name: Upload security reports
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: security-reports
          path: |
            safety-report.json
            bandit-report.json
  
  # ========================================
  # Stage 3: Tests (Matrix)
  # ========================================
  test:
    name: Test (Python ${{ matrix.python-version }})
    runs-on: ubuntu-latest
    needs: [quality]
    
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11', '3.12']
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio httpx
      
      - name: Run unit tests
        run: |
          pytest tests/unit/ \
            --cov=src \
            --cov-report=xml \
            --cov-report=term-missing \
            --junitxml=junit-${{ matrix.python-version }}.xml \
            -v
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: test-results-${{ matrix.python-version }}
          path: |
            junit-${{ matrix.python-version }}.xml
            coverage.xml
  
  # ========================================
  # Stage 4: Integration Tests
  # ========================================
  integration-test:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: [test]
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: pip install -r requirements.txt pytest pytest-asyncio httpx
      
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
        run: pytest tests/integration/ -v --tb=short
  
  # ========================================
  # Stage 5: Build Docker Image
  # ========================================
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [integration-test]
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/setup-buildx-action@v3
      
      - name: Log in to Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  # ========================================
  # Stage 6: Deploy to Staging
  # ========================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        env:
          DEPLOY_KEY: ${{ secrets.STAGING_DEPLOY_KEY }}
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
        run: |
          echo "Deploying image tag: $IMAGE_TAG"
          # Deploy commands here
  
  # ========================================
  # Stage 7: Deploy to Production
  # ========================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.PROD_DEPLOY_KEY }}
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
        run: |
          echo "Deploying to production: $IMAGE_TAG"
          # Production deploy commands here
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: |
            Production deployment ${{ job.status }}!
            Version: ${{ needs.build.outputs.image-tag }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

### ตัวอย่างที่ 23: Makefile สำหรับ Local Development

```makefile
# Makefile
.PHONY: install test lint format security clean run

# Variables
PYTHON = python3
PIP = pip
PYTEST = pytest
VENV = .venv

install:
	$(PIP) install --upgrade pip
	$(PIP) install -r requirements.txt -r requirements-dev.txt
	pre-commit install

test:
	$(PYTEST) tests/ -v --cov=src --cov-report=term-missing

test-unit:
	$(PYTEST) tests/unit/ -v

test-integration:
	$(PYTEST) tests/integration/ -v

lint:
	flake8 src/ tests/
	mypy src/ --ignore-missing-imports

format:
	black src/ tests/
	isort src/ tests/

format-check:
	black --check src/ tests/
	isort --check-only src/ tests/

security:
	safety check -r requirements.txt
	bandit -r src/ -ll

clean:
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete
	find . -type d -name ".pytest_cache" -delete
	rm -rf htmlcov/ .coverage coverage.xml

run:
	uvicorn src.app.main:app --reload --port 8000

docker-build:
	docker build -t myapp:local .

docker-run:
	docker run -p 8000:8000 myapp:local

ci: format-check lint test security
	@echo "All CI checks passed! ✅"
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Basic CI Workflow
สร้าง GitHub Actions workflow ที่:
- Trigger เมื่อ push ไปที่ main และ develop branches
- ติดตั้ง Python 3.11
- รัน `flake8` และ `black --check`
- รัน `pytest` พร้อม coverage report

**เฉลย:**
```yaml
name: Basic CI

on:
  push:
    branches: [main, develop]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install flake8 black pytest pytest-cov
          pip install -r requirements.txt
      
      - name: Lint
        run: |
          flake8 . --max-line-length=88
          black --check .
      
      - name: Test
        run: pytest --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

### แบบฝึกหัดที่ 2: Matrix Build
สร้าง workflow ที่ test บน Python 3.9, 3.10, 3.11 และ 3.12 พร้อมกัน

**เฉลย:**
```yaml
name: Matrix Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11', '3.12']
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install pytest && pip install -r requirements.txt
      - run: pytest tests/ -v
```

### แบบฝึกหัดที่ 3: Docker Build and Push
สร้าง workflow ที่ build Docker image และ push ไปยัง GitHub Container Registry

**เฉลย:**
```yaml
name: Docker

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### แบบฝึกหัดที่ 4: Pre-commit Hooks
ตั้งค่า pre-commit hooks ที่ตรวจสอบ: trailing whitespace, black formatting, flake8, และ detect-secrets

**เฉลย:**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: detect-private-key

  - repo: https://github.com/psf/black
    rev: 23.11.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        args: ['--max-line-length=88']

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
```

### แบบฝึกหัดที่ 5: Deployment with Environment Protection
สร้าง workflow ที่ deploy ไปยัง staging อัตโนมัติ แต่ต้องได้รับการ approve ด้วยมือก่อน deploy ไปยัง production

**เฉลย:**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        run: echo "Deploying to staging..."
  
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://myapp.com
    # การกำหนด environment protection rules ใน GitHub
    # ให้ตั้งค่า "Required reviewers" ใน Repository Settings > Environments
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Production
        run: echo "Deploying to production..."
```

### แบบฝึกหัดที่ 6: Complete Pipeline
สร้าง complete CI/CD pipeline สำหรับ Python package ที่รวม: quality checks, tests (matrix), build, และ publish ไปยัง PyPI เมื่อ tag version

**เฉลย:**
```yaml
name: Python Package CI/CD

on:
  push:
    branches: [main]
    tags: ['v*.*.*']
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install black flake8 mypy
      - run: black --check . && flake8 . && mypy src/
  
  test:
    needs: quality
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11', '3.12']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r requirements.txt pytest pytest-cov
      - run: pytest --cov=src --cov-fail-under=80
  
  publish:
    needs: test
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment: pypi
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Build package
        run: |
          pip install build twine
          python -m build
      - name: Publish to PyPI
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
        run: twine upload dist/*
```

---

## สรุป

Part 93 ครอบคลุมหัวข้อสำคัญของ CI/CD:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| CI/CD Concepts | pipeline stages, benefits, automation |
| GitHub Actions | workflow YAML, triggers, jobs, steps |
| Actions Marketplace | checkout, setup-python, cache, artifacts |
| Testing Pipeline | pytest, coverage, flake8, black, mypy |
| Docker | multi-stage builds, push to registries |
| Deployments | AWS ECS, GCP Cloud Run, Heroku |
| Secrets | GitHub secrets, AWS Secrets Manager |
| Matrix Builds | multi-version, multi-OS testing |
| GitLab CI | .gitlab-ci.yml, stages, runners |
| Pre-commit | hooks, formatters, security checks |
| Complete Pipeline | FastAPI app end-to-end CI/CD |

---

*Part 93 - CI/CD - GitHub Actions & GitLab CI | Python Course*
