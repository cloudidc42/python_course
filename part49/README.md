# Part 49: Docker for Python Developers

## สารบัญ

1. [Docker Concepts พื้นฐาน](#1-docker-concepts-พื้นฐาน)
2. [Dockerfile สำหรับ Python](#2-dockerfile-สำหรับ-python)
3. [Python Base Images](#3-python-base-images)
4. [Multi-stage Builds](#4-multi-stage-builds)
5. [.dockerignore](#5-dockerignore)
6. [Docker Compose](#6-docker-compose)
7. [Environment Variables ใน Docker](#7-environment-variables-ใน-docker)
8. [Volume Mounts](#8-volume-mounts)
9. [Container Networking](#9-container-networking)
10. [Docker Best Practices สำหรับ Python](#10-docker-best-practices-สำหรับ-python)
11. [Production Dockerfile](#11-production-dockerfile)
12. [Docker Security](#12-docker-security)
13. [Python Package Caching](#13-python-package-caching)
14. [แบบฝึกหัด](#14-แบบฝึกหัด)

---

## 1. Docker Concepts พื้นฐาน

### Docker คืออะไร?

Docker คือ platform สำหรับ containerization ที่ช่วยให้:
- โปรแกรมรันได้เหมือนกันทุก environment ("works on my machine" = ทุก machine)
- Isolate dependencies
- Scale ได้ง่าย
- Deploy ได้เร็ว

### Core Concepts

```
Image     = Blueprint ของ container (read-only)
          = ชั้นๆ ของ file system changes
          = เหมือน class ใน OOP

Container = Running instance ของ image (read-write)
          = เหมือน object ที่สร้างจาก class
          = มี lifecycle: created → running → stopped → removed

Volume    = Persistent storage สำหรับ container data
          = ข้อมูลยังอยู่แม้ container จะถูกลบ

Network   = Virtual network สำหรับ containers
          = Containers คุยกันผ่าน network
          = Isolated จาก host by default

Registry  = Repository สำหรับ Docker images
          = Docker Hub (public), ECR, GCR, etc.
```

### คำสั่ง Docker พื้นฐาน

```bash
# === Images ===
docker pull python:3.11-slim         # ดึง image จาก registry
docker images                         # แสดง images ทั้งหมด
docker rmi python:3.11-slim          # ลบ image
docker build -t myapp:1.0 .          # Build image จาก Dockerfile
docker tag myapp:1.0 myapp:latest    # Tag image

# === Containers ===
docker run python:3.11 python --version         # รัน container
docker run -it python:3.11 bash                 # interactive shell
docker run -d -p 8000:8000 myapp:1.0            # detached, port mapping
docker run --name mycontainer myapp:1.0         # กำหนดชื่อ
docker ps                                        # แสดง running containers
docker ps -a                                     # แสดงทุก containers
docker stop mycontainer                          # หยุด container
docker start mycontainer                         # เริ่ม container
docker rm mycontainer                            # ลบ container
docker logs mycontainer                          # ดู logs
docker exec -it mycontainer bash                # เข้าไปใน container

# === Volumes ===
docker volume create mydata                     # สร้าง volume
docker volume ls                                # แสดง volumes
docker volume rm mydata                         # ลบ volume

# === Cleanup ===
docker system prune                             # ลบ unused resources
docker system prune -a                          # ลบทุกอย่าง (careful!)
```

---

## 2. Dockerfile สำหรับ Python

Dockerfile คือ text file ที่บอก Docker ว่าจะ build image อย่างไร

### ตัวอย่างที่ 1: Dockerfile พื้นฐาน

```dockerfile
# Dockerfile

# 1. Base image
FROM python:3.11-slim

# 2. Set working directory
WORKDIR /app

# 3. Copy requirements first (for caching)
COPY requirements.txt .

# 4. Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy application code
COPY . .

# 6. Expose port
EXPOSE 8000

# 7. Run command
CMD ["python", "app.py"]
```

### คำอธิบาย Dockerfile Instructions

```dockerfile
# FROM - กำหนด base image (ต้องมีเสมอ)
FROM python:3.11-slim

# ARG - Build argument (เฉพาะตอน build)
ARG BUILD_DATE
ARG VERSION=1.0.0

# ENV - Environment variable (ตอน run ด้วย)
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8000

# WORKDIR - กำหนด working directory
WORKDIR /app

# COPY - Copy files จาก host เข้า container
COPY requirements.txt .
COPY src/ ./src/

# ADD - เหมือน COPY แต่รองรับ URL และ tar extraction
# (ใช้ COPY แทน ADD ยกเว้นต้องการ features พิเศษ)
ADD https://example.com/config.yaml .

# RUN - รัน command ตอน build
RUN pip install -r requirements.txt && \
    mkdir -p /app/logs

# EXPOSE - บอก port ที่ใช้ (documentation เท่านั้น)
EXPOSE 8000

# VOLUME - สร้าง mount point สำหรับ data
VOLUME ["/app/data", "/app/logs"]

# USER - เปลี่ยน user ที่ใช้รัน
USER appuser

# ENTRYPOINT - คำสั่งหลักที่รันเสมอ
ENTRYPOINT ["python", "-m", "uvicorn"]

# CMD - arguments สำหรับ ENTRYPOINT หรือ default command
CMD ["app:app", "--host", "0.0.0.0", "--port", "8000"]

# LABEL - Metadata
LABEL maintainer="developer@example.com" \
      version="1.0.0" \
      description="My Python App"

# HEALTHCHECK - ตรวจสอบ container health
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1
```

### ตัวอย่างที่ 2: Flask Application Dockerfile

```dockerfile
# Dockerfile สำหรับ Flask app

FROM python:3.11-slim

# Prevent Python from writing .pyc files
ENV PYTHONDONTWRITEBYTECODE=1
# Force stdout/stderr unbuffered (important for logging)
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# Use gunicorn for production
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

**requirements.txt:**
```
flask>=3.0.0
gunicorn>=21.0.0
psycopg2-binary>=2.9.0
redis>=5.0.0
```

```bash
# Build และรัน
docker build -t flask-app .
docker run -p 8000:8000 flask-app
```

### ตัวอย่างที่ 3: FastAPI Application Dockerfile

```dockerfile
# Dockerfile สำหรับ FastAPI app

FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy app
COPY ./app ./app

# Non-root user
RUN addgroup --system app && \
    adduser --system --group app && \
    chown -R app:app /app
USER app

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 3. Python Base Images

### เปรียบเทียบ Base Images

| Image | Size | ใช้เมื่อ |
|-------|------|---------|
| `python:3.11` | ~900MB | Development, ต้องการ tools ครบ |
| `python:3.11-slim` | ~130MB | Production ทั่วไป |
| `python:3.11-alpine` | ~50MB | Size สำคัญมาก, แต่ build ช้า |
| `python:3.11-slim-bullseye` | ~130MB | Debian Bullseye base |
| `distroless/python3` | ~50MB | Security-focused production |

### ตัวอย่างที่ 4: เปรียบเทียบ Base Images

```dockerfile
# === python:3.11 (full) ===
FROM python:3.11
# + ครบถ้วน, มี build tools, gcc
# - ขนาดใหญ่ ~900MB
# ใช้: development, ต้องการ compile C extensions

# === python:3.11-slim ===
FROM python:3.11-slim
# + เล็กกว่ามาก ~130MB
# + มี pip
# - ขาด build tools บางตัว
# - ต้อง install เพิ่ม ถ้า package ต้องการ system libs
# ใช้: production ทั่วไป (recommended)

# === python:3.11-alpine ===
FROM python:3.11-alpine
# + เล็กที่สุด ~50MB
# - ใช้ musl libc แทน glibc (อาจมีปัญหา compatibility)
# - pip install บาง package ช้ามาก (ต้อง compile)
# ใช้: เมื่อ size สำคัญมาก และ packages ทั้งหมดเป็น pure Python

# === แนะนำ: python:3.11-slim ===
FROM python:3.11-slim

# สำหรับ packages ที่ต้องการ system libs:
RUN apt-get update && apt-get install -y \
    libpq-dev \       # PostgreSQL client
    libssl-dev \      # SSL support  
    curl \            # health checks
    && rm -rf /var/lib/apt/lists/*  # ล้าง cache!
```

### ตัวอย่างที่ 5: Alpine Image พร้อม Workarounds

```dockerfile
# ใช้ alpine ด้วย workarounds สำหรับ packages ที่ต้อง compile
FROM python:3.11-alpine

# Alpine ใช้ apk แทน apt
RUN apk add --no-cache \
    gcc \
    musl-dev \
    postgresql-dev \
    libffi-dev \
    openssl-dev

# Install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ลบ build tools หลัง install เสร็จ
RUN apk del gcc musl-dev

COPY . .
CMD ["python", "app.py"]
```

---

## 4. Multi-stage Builds

Multi-stage builds ช่วยลดขนาด final image โดยแยก build environment ออกจาก runtime environment

### ตัวอย่างที่ 6: Multi-stage Build พื้นฐาน

```dockerfile
# ==================
# Stage 1: Builder
# ==================
FROM python:3.11-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages into a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt


# ==================
# Stage 2: Runtime
# ==================
FROM python:3.11-slim AS runtime

# Copy only the virtual environment from builder
COPY --from=builder /opt/venv /opt/venv

# Set path to use venv
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /app
COPY . .

# Non-root user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

EXPOSE 8000
CMD ["python", "app.py"]
```

### ตัวอย่างที่ 7: Multi-stage ที่ใช้ poetry

```dockerfile
# Multi-stage build ด้วย Poetry

# ==================
# Stage 1: Poetry build
# ==================
FROM python:3.11-slim AS builder

# Install poetry
ENV POETRY_NO_INTERACTION=1 \
    POETRY_VIRTUALENVS_IN_PROJECT=1 \
    POETRY_VIRTUALENVS_CREATE=1 \
    POETRY_CACHE_DIR=/tmp/poetry_cache

RUN pip install poetry==1.7.1

WORKDIR /app
COPY pyproject.toml poetry.lock ./

# Install dependencies (no dev dependencies)
RUN poetry install --only=main --no-root && rm -rf $POETRY_CACHE_DIR

# Copy source
COPY src ./src
RUN poetry install --only=main


# ==================
# Stage 2: Runtime
# ==================
FROM python:3.11-slim AS runtime

# Copy venv from builder
COPY --from=builder /app/.venv /app/.venv

# Set environment
ENV VIRTUAL_ENV=/app/.venv
ENV PATH="/app/.venv/bin:$PATH"

WORKDIR /app
COPY src ./src

USER 1000

CMD ["python", "-m", "myapp"]
```

### ตัวอย่างที่ 8: Multi-stage พร้อม Tests

```dockerfile
# Dockerfile ที่รัน tests ก่อน build production image

# ==================
# Stage 1: Base
# ==================
FROM python:3.11-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt requirements-dev.txt ./


# ==================
# Stage 2: Development/Test
# ==================
FROM base AS test

# Install all deps including dev
RUN pip install --no-cache-dir -r requirements.txt -r requirements-dev.txt

COPY . .

# Run tests (ถ้า fail จะ break the build)
RUN pytest tests/ -v --tb=short

# ==================
# Stage 3: Production
# ==================
FROM base AS production

# Install only production deps
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN adduser --disabled-password appuser
USER appuser

EXPOSE 8000
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000"]

# Build commands:
# docker build --target test -t myapp:test .
# docker build --target production -t myapp:prod .
```

---

## 5. .dockerignore

`.dockerignore` บอก Docker ว่าไฟล์ไหนไม่ต้อง copy เข้า container

### ตัวอย่างที่ 9: .dockerignore สมบูรณ์

```
# .dockerignore

# Version control
.git
.gitignore
.gitattributes

# Python
__pycache__/
*.py[cod]
*.pyo
.pytest_cache/
.mypy_cache/
.ruff_cache/
*.egg-info/
dist/
build/

# Virtual environments
.venv/
venv/
env/
ENV/

# Testing
tests/
.coverage
htmlcov/
.tox/

# Development configs
.env
.env.*
!.env.example

# IDE
.idea/
.vscode/
*.swp
*.swo
.DS_Store
Thumbs.db

# Docker
Dockerfile*
docker-compose*.yml
.dockerignore

# Documentation
docs/
*.md
!README.md

# Logs
*.log
logs/

# Data files (ใหญ่เกินไปสำหรับ image)
data/
*.csv
*.sqlite
*.db

# Secrets
*.pem
*.key
secrets/
```

---

## 6. Docker Compose

Docker Compose ใช้สำหรับรัน multiple containers พร้อมกัน

### ตัวอย่างที่ 10: docker-compose.yml พื้นฐาน

```yaml
# docker-compose.yml

version: "3.9"

services:
  # === Web Application ===
  web:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp_web
    ports:
      - "8000:8000"
    environment:
      - DEBUG=true
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
    volumes:
      - .:/app  # mount code สำหรับ development
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  # === PostgreSQL Database ===
  db:
    image: postgres:15-alpine
    container_name: myapp_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # === Redis Cache ===
  redis:
    image: redis:7-alpine
    container_name: myapp_redis
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped

  # === Celery Worker ===
  worker:
    build:
      context: .
    container_name: myapp_worker
    command: celery -A app.celery worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
    volumes:
      - .:/app
    depends_on:
      - db
      - redis
    restart: unless-stopped

  # === Nginx Reverse Proxy ===
  nginx:
    image: nginx:alpine
    container_name: myapp_nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./static:/app/static:ro
    depends_on:
      - web
    restart: unless-stopped

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  default:
    name: myapp_network
```

### ตัวอย่างที่ 11: Docker Compose สำหรับ Development

```yaml
# docker-compose.dev.yml

version: "3.9"

services:
  web:
    build:
      context: .
      target: development  # ใช้ dev stage
      args:
        BUILDKIT_INLINE_CACHE: 1
    volumes:
      # Hot reload - mount source code
      - .:/app
      # ป้องกันไม่ให้ host venv ทับ container venv
      - /app/.venv
      - /app/__pycache__
    environment:
      - APP_ENV=development
      - DEBUG=true
    # Override command สำหรับ dev (auto-reload)
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    ports:
      - "8000:8000"
    # Jupyter notebook port
      - "8888:8888"

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"  # เปิด port สำหรับ dev tools

  # pgAdmin สำหรับ database management
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - db

  # Redis Commander - Redis GUI
  redis-commander:
    image: rediscommander/redis-commander:latest
    environment:
      REDIS_HOSTS: local:redis:6379
    ports:
      - "8081:8081"
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### คำสั่ง Docker Compose

```bash
# Start services
docker-compose up               # foreground
docker-compose up -d            # detached (background)
docker-compose up --build       # build ก่อน start
docker-compose up web db        # เฉพาะบาง services

# Stop services
docker-compose down             # stop และ remove containers
docker-compose down -v          # + remove volumes
docker-compose down --rmi all   # + remove images

# View logs
docker-compose logs             # ทุก services
docker-compose logs web         # เฉพาะ web service
docker-compose logs -f web      # follow logs

# Execute commands
docker-compose exec web bash    # bash ใน web container
docker-compose exec web python manage.py migrate
docker-compose run --rm web pytest tests/

# Other
docker-compose ps               # แสดง status
docker-compose restart web      # restart service
docker-compose scale worker=3   # scale workers to 3
```

### ตัวอย่างที่ 12: Docker Compose Override Pattern

```yaml
# ใช้ หลาย compose files:
# docker-compose.yml - base config
# docker-compose.override.yml - dev overrides (auto-loaded)
# docker-compose.prod.yml - production overrides

# docker-compose.yml (base)
version: "3.9"

services:
  web:
    image: myapp:${VERSION:-latest}
    environment:
      - APP_NAME=MyApp

---
# docker-compose.override.yml (dev - auto-loaded)
version: "3.9"

services:
  web:
    build: .
    volumes:
      - .:/app
    environment:
      - DEBUG=true
    command: uvicorn app:app --reload

---
# docker-compose.prod.yml (production)
version: "3.9"

services:
  web:
    restart: always
    env_file: .env.production
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
```

```bash
# Development (auto-loads override)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## 7. Environment Variables ใน Docker

### ตัวอย่างที่ 13: Environment Variables Methods

```dockerfile
# Method 1: ENV instruction in Dockerfile (hardcoded - not recommended for secrets)
FROM python:3.11-slim
ENV APP_NAME=MyApp \
    PORT=8000 \
    LOG_LEVEL=INFO
```

```bash
# Method 2: docker run -e flag
docker run -e DEBUG=true -e DATABASE_URL=postgresql://localhost/mydb myapp

# Method 3: docker run --env-file
docker run --env-file .env myapp
```

```yaml
# Method 4: docker-compose.yml - environment
services:
  web:
    environment:
      - DEBUG=true
      - PORT=8000
      # Pass from host environment (no value = use host's value)
      - DATABASE_URL
      - SECRET_KEY

# Method 5: docker-compose.yml - env_file
services:
  web:
    env_file:
      - .env
      - .env.production
```

### ตัวอย่างที่ 14: Secrets ใน Docker

```yaml
# docker-compose.yml พร้อม Docker Secrets

version: "3.9"

services:
  web:
    image: myapp
    secrets:
      - db_password
      - secret_key
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - SECRET_KEY_FILE=/run/secrets/secret_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  secret_key:
    file: ./secrets/secret_key.txt
```

```python
# อ่าน secret จากไฟล์ใน container
import os

def read_secret(secret_name: str) -> str:
    """อ่าน Docker secret จาก /run/secrets/"""
    # ลองอ่านจาก file ก่อน (Docker Secrets)
    secret_file = f"/run/secrets/{secret_name}"
    if os.path.exists(secret_file):
        with open(secret_file) as f:
            return f.read().strip()
    
    # Fallback ไป environment variable
    env_var = secret_name.upper()
    value = os.environ.get(env_var)
    if not value:
        raise ValueError(f"Secret '{secret_name}' not found")
    return value


# ใช้งาน
db_password = read_secret("db_password")
secret_key = read_secret("secret_key")
```

---

## 8. Volume Mounts

### ตัวอย่างที่ 15: Types of Volumes

```yaml
# docker-compose.yml

version: "3.9"

services:
  web:
    image: myapp
    volumes:
      # 1. Named volume (managed by Docker)
      - app_data:/app/data
      
      # 2. Bind mount (host path → container path)
      - ./src:/app/src
      - ./config:/app/config:ro    # read-only
      
      # 3. Anonymous volume (temporary)
      - /app/temp
      
      # 4. Named volume with options
      - type: volume
        source: app_logs
        target: /app/logs
        volume:
          nocopy: true

volumes:
  app_data:
    driver: local
  app_logs:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=100m   # 100MB limit
```

### ตัวอย่างที่ 16: Volume สำหรับ Development

```yaml
# Development setup: code syncing + data persistence

version: "3.9"

services:
  web:
    build: .
    volumes:
      # Source code (hot reload)
      - .:/app
      
      # ป้องกัน host __pycache__ ทับ container
      - /app/__pycache__
      - /app/.pytest_cache
      
      # Persistent dev data
      - dev_db:/app/data

  db:
    image: postgres:15-alpine
    volumes:
      # Persistent database
      - postgres_dev:/var/lib/postgresql/data
      
      # SQL init scripts
      - ./docker/init.sql:/docker-entrypoint-initdb.d/init.sql:ro

volumes:
  dev_db:
  postgres_dev:
```

### ตัวอย่างที่ 17: Backup Volume Data

```bash
# Backup volume data
docker run --rm \
  -v myapp_postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz /data

# Restore from backup
docker run --rm \
  -v myapp_postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_backup.tar.gz -C /
```

---

## 9. Container Networking

### ตัวอย่างที่ 18: Docker Networks

```yaml
# docker-compose.yml พร้อม custom networks

version: "3.9"

services:
  web:
    image: myapp
    networks:
      - frontend
      - backend

  api:
    image: api-service
    networks:
      - backend

  db:
    image: postgres:15
    networks:
      - backend  # ไม่ expose ไป frontend

  nginx:
    image: nginx:alpine
    networks:
      - frontend
    ports:
      - "80:80"

networks:
  frontend:
    name: myapp_frontend
  backend:
    name: myapp_backend
    internal: true  # ไม่มี internet access
```

```bash
# Containers ใน same network คุยกันด้วย service name
# จาก web container:
curl http://db:5432       # ✅ ใช้ service name
curl http://localhost:5432  # ❌ ไม่ได้ (คนละ network namespace)
```

---

## 10. Docker Best Practices สำหรับ Python

### ตัวอย่างที่ 19: Layer Caching Optimization

```dockerfile
# ❌ แบบที่ไม่ดี - copy ทุกอย่างก่อน install
FROM python:3.11-slim
WORKDIR /app
COPY . .                          # Copy ทุกอย่าง
RUN pip install -r requirements.txt  # ต้อง re-install ทุกครั้งที่ code เปลี่ยน!

# ✅ แบบที่ดี - copy requirements ก่อน
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .           # Copy เฉพาะ requirements ก่อน
RUN pip install -r requirements.txt  # Cache layer นี้ถ้า requirements ไม่เปลี่ยน
COPY . .                          # Copy code หลังสุด
```

### ตัวอย่างที่ 20: Python-specific Best Practices

```dockerfile
FROM python:3.11-slim

# ✅ ตั้งค่า ENV ที่จำเป็นสำหรับ Python
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONFAULTHANDLER=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# PYTHONDONTWRITEBYTECODE=1
# → ไม่สร้าง .pyc files (ประหยัด space ใน container)

# PYTHONUNBUFFERED=1
# → ส่ง stdout/stderr ทันที ไม่ buffer (สำคัญสำหรับ logs)

# PYTHONFAULTHANDLER=1
# → Print Python traceback ถ้า crash (debugging)

# PIP_NO_CACHE_DIR=1
# → ไม่ cache pip downloads (ลด image size)

WORKDIR /app

# ✅ Upgrade pip ก่อน
RUN pip install --upgrade pip

# ✅ ติดตั้ง system dependencies อย่างมีประสิทธิภาพ
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libpq-dev \
        curl \
    && rm -rf /var/lib/apt/lists/*  # ✅ ล้าง cache เสมอ!

# ✅ pin package versions
COPY requirements.txt .
RUN pip install -r requirements.txt

# ✅ Non-root user
RUN useradd --create-home appuser
USER appuser

COPY --chown=appuser:appuser . .

EXPOSE 8000
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:8000"]
```

---

## 11. Production Dockerfile

### ตัวอย่างที่ 21: Production-Ready Dockerfile

```dockerfile
# Production Dockerfile - Best Practices สมบูรณ์

# ==================
# Stage 1: Build
# ==================
FROM python:3.11-slim AS builder

ARG DEBIAN_FRONTEND=noninteractive

# Build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Create virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PIP_NO_CACHE_DIR=1

# Install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt


# ==================
# Stage 2: Production
# ==================
FROM python:3.11-slim AS production

ARG DEBIAN_FRONTEND=noninteractive
ARG BUILD_DATE
ARG VERSION

# Labels
LABEL org.opencontainers.image.created="${BUILD_DATE}" \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.title="MyApp" \
      org.opencontainers.image.description="My Python Application"

# Runtime dependencies only
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy virtual environment
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONFAULTHANDLER=1

WORKDIR /app

# Create non-root user
RUN groupadd --gid 1001 appgroup && \
    useradd --uid 1001 --gid appgroup --shell /bin/bash --create-home appuser

# Copy application
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Graceful shutdown
STOPSIGNAL SIGTERM

CMD ["gunicorn", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--timeout", "120", \
     "--graceful-timeout", "30", \
     "--access-logfile", "-", \
     "--error-logfile", "-", \
     "app.main:app"]
```

### ตัวอย่างที่ 22: Production docker-compose.yml

```yaml
# docker-compose.prod.yml

version: "3.9"

services:
  web:
    image: myapp:${VERSION}
    restart: always
    env_file:
      - .env.production
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first  # zero-downtime deployment
        failure_action: rollback
      rollback_config:
        parallelism: 1
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:15-alpine
    restart: always
    env_file: .env.production
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/app/static:ro
    depends_on:
      - web

volumes:
  postgres_data:
  redis_data:
  static_files:
```

---

## 12. Docker Security

### ตัวอย่างที่ 23: Security Best Practices

```dockerfile
# Security-hardened Dockerfile

FROM python:3.11-slim

# 1. อย่าใช้ root user
RUN useradd --create-home --shell /bin/bash --uid 1001 appuser

# 2. ไม่ expose secrets ใน ENV หรือ ARG
# ❌ อย่าทำ:
# ENV SECRET_KEY=mysecret
# ARG DATABASE_PASSWORD=password

# ✅ ใช้ runtime env vars แทน

# 3. Read-only filesystem
# docker run --read-only myapp

# 4. Drop capabilities
# docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# 5. No new privileges
# docker run --security-opt=no-new-privileges myapp

# 6. Scan for vulnerabilities
# docker scan myapp  (ใช้ Snyk)
# trivy image myapp  (ใช้ Trivy)

WORKDIR /app

# 7. ใช้ --no-install-recommends เพื่อลด attack surface
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 8. Pin exact versions
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

# 9. Switch to non-root ก่อน EXPOSE
USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

```bash
# Security scanning
# ติดตั้ง trivy
brew install trivy  # macOS
# or
apt-get install trivy  # Ubuntu

# Scan image
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Scan Dockerfile
trivy config Dockerfile
```

---

## 13. Python Package Caching

### ตัวอย่างที่ 24: Package Caching Strategies

```dockerfile
# Strategy 1: BuildKit Cache Mount (แนะนำ)
# syntax=docker/dockerfile:1
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

# ใช้ BuildKit cache mount - pip cache persist ระหว่าง builds
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

COPY . .
CMD ["python", "app.py"]
```

```dockerfile
# Strategy 2: Virtual Environment Caching
FROM python:3.11-slim

WORKDIR /app

# สร้าง venv ใน fixed path
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

COPY . .
USER 1000
CMD ["python", "app.py"]
```

```bash
# Build ด้วย BuildKit
DOCKER_BUILDKIT=1 docker build -t myapp .

# หรือกำหนดใน docker buildx
docker buildx build -t myapp .
```

### ตัวอย่างที่ 25: requirements.txt Best Practices

```python
# requirements.txt - pin exact versions สำหรับ reproducibility

# Generate ด้วย pip-compile (pip-tools)
# pip install pip-tools
# pip-compile requirements.in  → สร้าง requirements.txt พร้อม hashes

# requirements.in (human-maintained)
# flask>=3.0
# sqlalchemy>=2.0
# gunicorn

# requirements.txt (generated - commit this!)
# flask==3.0.1 \
#     --hash=sha256:abc123...
# werkzeug==3.0.1 \
#     --hash=sha256:def456...

# ตัวอย่าง requirements structure ที่ดี:
```

```
requirements/
├── base.txt        # shared deps
├── production.txt  # production only (includes base.txt)
├── development.txt # dev tools (includes base.txt)
└── test.txt        # testing (includes development.txt)
```

```
# base.txt
flask==3.0.1
sqlalchemy==2.0.25
pydantic==2.5.3

# production.txt
-r base.txt
gunicorn==21.2.0
psycopg2-binary==2.9.9

# development.txt
-r base.txt
pytest==7.4.4
black==23.12.1
ruff==0.1.9
mypy==1.8.0

# test.txt
-r development.txt
pytest-cov==4.1.0
pytest-mock==3.12.0
```

```bash
# Docker build สำหรับ production
COPY requirements/base.txt requirements/production.txt requirements/
RUN pip install -r requirements/production.txt
```

---

## 14. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Flask App ใน Docker

```dockerfile
# เฉลย - Dockerfile สำหรับ Flask TODO app

FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN adduser --disabled-password --gecos '' appuser
COPY --chown=appuser:appuser . .
USER appuser

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:5000/ || exit 1

CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

```python
# app.py - Flask TODO app
from flask import Flask, jsonify, request

app = Flask(__name__)

todos = []
next_id = 1

@app.route('/health')
def health():
    return jsonify({"status": "ok"})

@app.route('/todos', methods=['GET'])
def get_todos():
    return jsonify(todos)

@app.route('/todos', methods=['POST'])
def create_todo():
    global next_id
    data = request.json
    todo = {"id": next_id, "title": data["title"], "done": False}
    todos.append(todo)
    next_id += 1
    return jsonify(todo), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```yaml
# docker-compose.yml
version: "3.9"

services:
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
    volumes:
      - .:/app
```

### แบบฝึกหัดที่ 2: Multi-stage Build

```dockerfile
# เฉลย - Multi-stage build สำหรับ FastAPI

# Stage 1: Build
FROM python:3.11-slim AS builder

WORKDIR /build

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt


# Stage 2: Test (optional - รัน tests ใน CI)
FROM builder AS test

COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

COPY . .
RUN python -m pytest tests/ -v


# Stage 3: Production
FROM python:3.11-slim AS production

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN useradd --uid 1001 appuser

COPY --chown=appuser . .
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

### แบบฝึกหัดที่ 3: Full Stack docker-compose

```yaml
# เฉลย - Full stack Python app

version: "3.9"

services:
  # FastAPI Backend
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://user:pass@postgres/mydb
      REDIS_URL: redis://redis:6379/0
      SECRET_KEY: ${SECRET_KEY}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - "8000:8000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Celery Worker
  worker:
    build:
      context: ./backend
    command: celery -A app.celery worker
    environment:
      DATABASE_URL: postgresql://user:pass@postgres/mydb
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      retries: 5

  # Nginx
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api

volumes:
  pgdata:
  redisdata:
```

### แบบฝึกหัดที่ 4: Data Science Jupyter Docker

```dockerfile
# เฉลย - Jupyter + Data Science environment

FROM python:3.11-slim

ENV JUPYTER_PORT=8888

WORKDIR /workspace

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir \
    jupyter==1.0.0 \
    numpy==1.26.2 \
    pandas==2.1.4 \
    matplotlib==3.8.2 \
    scikit-learn==1.3.2 \
    seaborn==0.13.0

# Jupyter config
RUN jupyter notebook --generate-config && \
    echo "c.NotebookApp.ip = '0.0.0.0'" >> ~/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.token = ''" >> ~/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.password = ''" >> ~/.jupyter/jupyter_notebook_config.py

EXPOSE 8888

VOLUME ["/workspace/notebooks", "/workspace/data"]

CMD ["jupyter", "notebook", "--no-browser", "--allow-root", "--ip=0.0.0.0"]
```

```yaml
# docker-compose.yml
version: "3.9"

services:
  jupyter:
    build: .
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/workspace/notebooks
      - ./data:/workspace/data
    environment:
      PYTHONPATH: /workspace
```

### แบบฝึกหัดที่ 5: CI/CD Pipeline Script

```bash
#!/bin/bash
# deploy.sh - Deployment script ที่ใช้ Docker

set -e  # Exit on error

VERSION=${1:-latest}
REGISTRY="registry.example.com"
APP_NAME="myapp"
IMAGE="${REGISTRY}/${APP_NAME}:${VERSION}"

echo "🐳 Building Docker image: ${IMAGE}"

# Build image
docker build \
    --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
    --build-arg VERSION=${VERSION} \
    --tag ${IMAGE} \
    --tag ${REGISTRY}/${APP_NAME}:latest \
    .

echo "🔍 Scanning for vulnerabilities..."
trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}

echo "✅ Tests passed"

echo "📤 Pushing image..."
docker push ${IMAGE}
docker push ${REGISTRY}/${APP_NAME}:latest

echo "🚀 Deploying..."
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d --no-build

echo "⏳ Waiting for health check..."
sleep 30

# Health check
if curl -f http://localhost/health; then
    echo "✅ Deployment successful!"
else
    echo "❌ Health check failed! Rolling back..."
    docker-compose -f docker-compose.prod.yml down
    # Pull previous version...
    exit 1
fi
```

### แบบฝึกหัดที่ 6: Python Script ที่ติดต่อ Docker

```python
# เฉลย - Python script ที่ใช้ Docker SDK

# pip install docker

import docker
import time
import sys

def run_python_in_container(script: str, timeout: int = 60) -> dict:
    """
    รัน Python script ใน Docker container
    Returns: {output, exit_code, error}
    """
    client = docker.from_env()
    
    container = client.containers.run(
        "python:3.11-slim",
        ["python", "-c", script],
        detach=True,
        mem_limit="128m",
        cpu_period=100000,
        cpu_quota=50000,  # 50% CPU
        network_disabled=True,  # ปิด network (security)
        remove=False,
    )
    
    try:
        container.wait(timeout=timeout)
        output = container.logs(stdout=True, stderr=False).decode()
        error = container.logs(stdout=False, stderr=True).decode()
        exit_code = container.attrs["State"]["ExitCode"]
        
        return {
            "output": output,
            "error": error,
            "exit_code": exit_code,
            "success": exit_code == 0,
        }
    finally:
        container.remove(force=True)


def build_image(dockerfile_path: str, tag: str) -> None:
    """Build Docker image"""
    client = docker.from_env()
    
    print(f"Building image: {tag}")
    image, logs = client.images.build(
        path=dockerfile_path,
        tag=tag,
        rm=True,    # Remove intermediate containers
        pull=True,  # Pull latest base image
    )
    
    for log in logs:
        if "stream" in log:
            print(log["stream"].strip())
    
    print(f"Built successfully: {image.tags}")


# ทดสอบ
result = run_python_in_container('print("Hello from container!")')
print(f"Output: {result['output']}")
print(f"Success: {result['success']}")
```

---

## สรุป

### Dockerfile Best Practices Checklist

```
✅ ใช้ specific version tags (python:3.11-slim ไม่ใช่ python:latest)
✅ ใช้ .dockerignore
✅ Copy requirements.txt ก่อน source code (layer caching)
✅ ใช้ multi-stage builds สำหรับ production
✅ ใช้ non-root user
✅ ตั้งค่า PYTHONDONTWRITEBYTECODE=1 และ PYTHONUNBUFFERED=1
✅ ล้าง apt cache หลัง install: rm -rf /var/lib/apt/lists/*
✅ Pin dependency versions
✅ เพิ่ม HEALTHCHECK
✅ Scan for vulnerabilities ก่อน deploy
```

### คำสั่ง Cheat Sheet

```bash
# Build
docker build -t app:1.0 .
docker build --target production -t app:1.0 .
DOCKER_BUILDKIT=1 docker build -t app:1.0 .

# Run
docker run -p 8000:8000 -e DEBUG=true app:1.0
docker run --env-file .env app:1.0
docker run -v $(pwd):/app app:1.0

# Compose
docker-compose up -d
docker-compose up --build
docker-compose down -v
docker-compose exec web bash
docker-compose logs -f

# Debug
docker logs container_name
docker exec -it container_name bash
docker inspect container_name
docker stats container_name
```

### Useful Environment Variables

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1    # ไม่สร้าง .pyc files
ENV PYTHONUNBUFFERED=1           # Unbuffered output
ENV PYTHONFAULTHANDLER=1         # Stack trace on crash
ENV PIP_NO_CACHE_DIR=1          # ไม่ cache pip downloads
ENV PIP_DISABLE_PIP_VERSION_CHECK=1  # ไม่ check pip version
ENV POETRY_NO_INTERACTION=1      # Non-interactive poetry
```
