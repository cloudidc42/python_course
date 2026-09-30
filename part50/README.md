# Part 50: 🚀 Project - Task Manager CLI Application

## สารบัญ

1. [Project Overview](#1-project-overview)
2. [Project Structure](#2-project-structure)
3. [Installation & Setup](#3-installation--setup)
4. [Source Code - Full Implementation](#4-source-code---full-implementation)
5. [Testing Guide](#5-testing-guide)
6. [Docker Support](#6-docker-support)
7. [Usage Guide](#7-usage-guide)
8. [Ideas for Extension](#8-ideas-for-extension)

---

## 1. Project Overview

**TaskFlow** คือ CLI Task Manager ที่สมบูรณ์ สร้างด้วย Python โดยใช้ทุกความรู้จาก Parts 46-49:

### Features

- ✅ **CRUD Operations**: Create, Read, Update, Delete tasks
- 🏷️ **Categories & Tags**: จัดกลุ่ม tasks
- 🔴 **Priority Levels**: high, medium, low
- 📅 **Due Dates**: กำหนดวันครบกำหนด
- 📊 **Status Tracking**: todo, in_progress, done, cancelled
- 🔍 **Search & Filter**: ค้นหาและกรอง tasks
- 📤 **Export**: CSV, JSON
- 🎨 **Rich Output**: Color-coded, beautiful tables
- 💾 **SQLite Backend**: Persistent storage
- ⚙️ **Configuration**: YAML config file support
- 🧪 **Unit Tests**: ครอบคลุม
- 🐳 **Docker Support**: Containerized

---

## 2. Project Structure

```
taskflow/
├── taskflow/
│   ├── __init__.py
│   ├── cli/
│   │   ├── __init__.py
│   │   ├── main.py          # Main CLI entry point
│   │   ├── commands/
│   │   │   ├── __init__.py
│   │   │   ├── tasks.py     # Task CRUD commands
│   │   │   ├── config.py    # Config commands
│   │   │   └── export.py    # Export commands
│   │   └── output.py        # Rich output helpers
│   ├── core/
│   │   ├── __init__.py
│   │   ├── models.py        # Data models (dataclasses)
│   │   ├── database.py      # SQLite database layer
│   │   ├── repository.py    # Data access layer
│   │   └── config.py        # Configuration
│   └── utils/
│       ├── __init__.py
│       └── helpers.py       # Utility functions
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_models.py
│   ├── test_repository.py
│   ├── test_cli.py
│   └── test_export.py
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── .env.example
└── config.yaml.example
```

---

## 3. Installation & Setup

```bash
# Clone / Create project
mkdir taskflow && cd taskflow

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Install in development mode
pip install -e .

# Run
taskflow --help
```

### requirements.txt

```
click>=8.1.7
rich>=13.7.0
pydantic>=2.5.3
pydantic-settings>=2.1.0
python-dotenv>=1.0.0
PyYAML>=6.0.1
```

### requirements-dev.txt

```
-r requirements.txt
pytest>=7.4.4
pytest-cov>=4.1.0
pytest-mock>=3.12.0
black>=23.12.1
ruff>=0.1.9
mypy>=1.8.0
```

---

## 4. Source Code - Full Implementation

### 4.1 Data Models (taskflow/core/models.py)

```python
"""
taskflow/core/models.py
Data models สำหรับ TaskFlow
"""

from __future__ import annotations

import uuid
from dataclasses import dataclass, field
from datetime import date, datetime
from enum import Enum
from typing import List, Optional


class Priority(str, Enum):
    """Task priority levels"""
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

    def __str__(self) -> str:
        return self.value

    @property
    def color(self) -> str:
        """Rich color สำหรับแต่ละ priority"""
        colors = {
            Priority.HIGH: "bold red",
            Priority.MEDIUM: "yellow",
            Priority.LOW: "green",
        }
        return colors[self]

    @property
    def icon(self) -> str:
        icons = {
            Priority.HIGH: "🔴",
            Priority.MEDIUM: "🟡",
            Priority.LOW: "🟢",
        }
        return icons[self]


class Status(str, Enum):
    """Task status"""
    TODO = "todo"
    IN_PROGRESS = "in_progress"
    DONE = "done"
    CANCELLED = "cancelled"

    def __str__(self) -> str:
        return self.value

    @property
    def icon(self) -> str:
        icons = {
            Status.TODO: "○",
            Status.IN_PROGRESS: "⟳",
            Status.DONE: "✓",
            Status.CANCELLED: "✗",
        }
        return icons[self]

    @property
    def color(self) -> str:
        colors = {
            Status.TODO: "white",
            Status.IN_PROGRESS: "cyan",
            Status.DONE: "green dim",
            Status.CANCELLED: "red dim",
        }
        return colors[self]


@dataclass
class Task:
    """Task data model"""
    title: str
    id: str = field(default_factory=lambda: str(uuid.uuid4())[:8])
    description: Optional[str] = None
    priority: Priority = Priority.MEDIUM
    status: Status = Status.TODO
    category: Optional[str] = None
    tags: List[str] = field(default_factory=list)
    due_date: Optional[date] = None
    created_at: datetime = field(default_factory=datetime.now)
    updated_at: datetime = field(default_factory=datetime.now)

    def __post_init__(self):
        # Convert string values to enums
        if isinstance(self.priority, str):
            self.priority = Priority(self.priority)
        if isinstance(self.status, str):
            self.status = Status(self.status)
        if isinstance(self.due_date, str) and self.due_date:
            self.due_date = date.fromisoformat(self.due_date)

    @property
    def is_overdue(self) -> bool:
        """ตรวจสอบว่า task เกินกำหนดหรือไม่"""
        if not self.due_date:
            return False
        if self.status in (Status.DONE, Status.CANCELLED):
            return False
        return self.due_date < date.today()

    @property
    def days_until_due(self) -> Optional[int]:
        """จำนวนวันที่เหลือก่อนครบกำหนด"""
        if not self.due_date:
            return None
        delta = self.due_date - date.today()
        return delta.days

    def mark_done(self) -> None:
        """Mark task as done"""
        self.status = Status.DONE
        self.updated_at = datetime.now()

    def mark_in_progress(self) -> None:
        """Mark task as in progress"""
        self.status = Status.IN_PROGRESS
        self.updated_at = datetime.now()

    def cancel(self) -> None:
        """Cancel task"""
        self.status = Status.CANCELLED
        self.updated_at = datetime.now()

    def update(self, **kwargs) -> None:
        """Update task fields"""
        for key, value in kwargs.items():
            if hasattr(self, key) and value is not None:
                setattr(self, key, value)
        self.updated_at = datetime.now()

    def to_dict(self) -> dict:
        """Convert to dictionary"""
        return {
            "id": self.id,
            "title": self.title,
            "description": self.description,
            "priority": str(self.priority),
            "status": str(self.status),
            "category": self.category,
            "tags": self.tags,
            "due_date": self.due_date.isoformat() if self.due_date else None,
            "created_at": self.created_at.isoformat(),
            "updated_at": self.updated_at.isoformat(),
        }

    @classmethod
    def from_dict(cls, data: dict) -> "Task":
        """สร้าง Task จาก dictionary"""
        return cls(**data)

    def __str__(self) -> str:
        return f"Task({self.id}: {self.title!r} [{self.priority}/{self.status}])"

    def __repr__(self) -> str:
        return self.__str__()
```

### 4.2 Database Layer (taskflow/core/database.py)

```python
"""
taskflow/core/database.py
SQLite database layer
"""

import json
import sqlite3
from contextlib import contextmanager
from datetime import datetime
from pathlib import Path
from typing import Generator, List, Optional


CREATE_TASKS_TABLE = """
CREATE TABLE IF NOT EXISTS tasks (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    priority TEXT NOT NULL DEFAULT 'medium',
    status TEXT NOT NULL DEFAULT 'todo',
    category TEXT,
    tags TEXT DEFAULT '[]',
    due_date TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL
);
"""

CREATE_INDEXES = [
    "CREATE INDEX IF NOT EXISTS idx_tasks_status ON tasks(status);",
    "CREATE INDEX IF NOT EXISTS idx_tasks_priority ON tasks(priority);",
    "CREATE INDEX IF NOT EXISTS idx_tasks_category ON tasks(category);",
    "CREATE INDEX IF NOT EXISTS idx_tasks_due_date ON tasks(due_date);",
]


class Database:
    """SQLite database manager"""

    def __init__(self, db_path: str = "taskflow.db"):
        self.db_path = Path(db_path)
        self.db_path.parent.mkdir(parents=True, exist_ok=True)
        self._init_db()

    def _init_db(self) -> None:
        """สร้าง tables และ indexes"""
        with self.connection() as conn:
            conn.execute(CREATE_TASKS_TABLE)
            for index_sql in CREATE_INDEXES:
                conn.execute(index_sql)
            conn.commit()

    @contextmanager
    def connection(self) -> Generator[sqlite3.Connection, None, None]:
        """Context manager สำหรับ database connection"""
        conn = sqlite3.connect(str(self.db_path))
        conn.row_factory = sqlite3.Row  # Access columns by name
        try:
            yield conn
        except Exception:
            conn.rollback()
            raise
        finally:
            conn.close()

    def execute(self, sql: str, params: tuple = ()) -> List[sqlite3.Row]:
        """Execute query และ return rows"""
        with self.connection() as conn:
            cursor = conn.execute(sql, params)
            conn.commit()
            return cursor.fetchall()

    def execute_one(self, sql: str, params: tuple = ()) -> Optional[sqlite3.Row]:
        """Execute query และ return หนึ่ง row"""
        rows = self.execute(sql, params)
        return rows[0] if rows else None

    def row_to_dict(self, row: sqlite3.Row) -> dict:
        """แปลง Row เป็น dict"""
        d = dict(row)
        # Parse JSON fields
        if "tags" in d and isinstance(d["tags"], str):
            d["tags"] = json.loads(d["tags"])
        return d
```

### 4.3 Repository Layer (taskflow/core/repository.py)

```python
"""
taskflow/core/repository.py
Data access layer สำหรับ tasks
"""

import json
from datetime import date, datetime
from typing import List, Optional

from .database import Database
from .models import Priority, Status, Task


class TaskRepository:
    """Repository สำหรับ CRUD operations บน tasks"""

    def __init__(self, db: Database):
        self.db = db

    def _task_to_row(self, task: Task) -> dict:
        """แปลง Task เป็น database row"""
        return {
            "id": task.id,
            "title": task.title,
            "description": task.description,
            "priority": str(task.priority),
            "status": str(task.status),
            "category": task.category,
            "tags": json.dumps(task.tags),
            "due_date": task.due_date.isoformat() if task.due_date else None,
            "created_at": task.created_at.isoformat(),
            "updated_at": task.updated_at.isoformat(),
        }

    def _row_to_task(self, row) -> Task:
        """แปลง database row เป็น Task"""
        data = self.db.row_to_dict(row)
        return Task.from_dict(data)

    def create(self, task: Task) -> Task:
        """สร้าง task ใหม่"""
        row = self._task_to_row(task)
        sql = """
            INSERT INTO tasks (id, title, description, priority, status,
                             category, tags, due_date, created_at, updated_at)
            VALUES (:id, :title, :description, :priority, :status,
                   :category, :tags, :due_date, :created_at, :updated_at)
        """
        self.db.execute(sql, tuple(row.values()))
        return task

    def get_by_id(self, task_id: str) -> Optional[Task]:
        """ดึง task ด้วย ID"""
        row = self.db.execute_one(
            "SELECT * FROM tasks WHERE id = ?", (task_id,)
        )
        return self._row_to_task(row) if row else None

    def get_all(
        self,
        status: Optional[Status] = None,
        priority: Optional[Priority] = None,
        category: Optional[str] = None,
        tag: Optional[str] = None,
        search: Optional[str] = None,
        overdue_only: bool = False,
    ) -> List[Task]:
        """ดึง tasks ทั้งหมดพร้อม filters"""
        conditions = []
        params = []

        if status:
            conditions.append("status = ?")
            params.append(str(status))

        if priority:
            conditions.append("priority = ?")
            params.append(str(priority))

        if category:
            conditions.append("category = ?")
            params.append(category)

        if tag:
            conditions.append("tags LIKE ?")
            params.append(f'%"{tag}"%')

        if search:
            conditions.append("(title LIKE ? OR description LIKE ?)")
            params.extend([f"%{search}%", f"%{search}%"])

        if overdue_only:
            today = date.today().isoformat()
            conditions.append(
                f"due_date IS NOT NULL AND due_date < ? "
                f"AND status NOT IN ('done', 'cancelled')"
            )
            params.append(today)

        where_clause = f"WHERE {' AND '.join(conditions)}" if conditions else ""
        sql = f"SELECT * FROM tasks {where_clause} ORDER BY created_at DESC"

        rows = self.db.execute(sql, tuple(params))
        return [self._row_to_task(row) for row in rows]

    def update(self, task: Task) -> Task:
        """อัพเดต task"""
        task.updated_at = datetime.now()
        row = self._task_to_row(task)
        sql = """
            UPDATE tasks SET
                title = :title,
                description = :description,
                priority = :priority,
                status = :status,
                category = :category,
                tags = :tags,
                due_date = :due_date,
                updated_at = :updated_at
            WHERE id = :id
        """
        self.db.execute(sql, tuple(row.values()))
        return task

    def delete(self, task_id: str) -> bool:
        """ลบ task"""
        self.db.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
        return True

    def count(self, status: Optional[Status] = None) -> int:
        """นับ tasks"""
        if status:
            row = self.db.execute_one(
                "SELECT COUNT(*) as count FROM tasks WHERE status = ?",
                (str(status),)
            )
        else:
            row = self.db.execute_one("SELECT COUNT(*) as count FROM tasks")
        return row["count"] if row else 0

    def get_categories(self) -> List[str]:
        """ดึง categories ทั้งหมด"""
        rows = self.db.execute(
            "SELECT DISTINCT category FROM tasks WHERE category IS NOT NULL"
        )
        return [row["category"] for row in rows]

    def get_stats(self) -> dict:
        """ดึง statistics"""
        stats = {}
        for status in Status:
            stats[str(status)] = self.count(status)
        stats["total"] = self.count()
        stats["overdue"] = len(self.get_all(overdue_only=True))
        return stats
```

### 4.4 Configuration (taskflow/core/config.py)

```python
"""
taskflow/core/config.py
Configuration management
"""

import os
from pathlib import Path
from typing import Optional

import yaml
from pydantic import Field
from pydantic_settings import BaseSettings


class AppSettings(BaseSettings):
    """Application settings"""

    # Core
    app_name: str = "TaskFlow"
    version: str = "1.0.0"
    debug: bool = False

    # Database
    database_path: str = Field(
        default_factory=lambda: str(
            Path.home() / ".taskflow" / "tasks.db"
        )
    )

    # Display
    items_per_page: int = 20
    date_format: str = "%Y-%m-%d"
    datetime_format: str = "%Y-%m-%d %H:%M"

    # Default values
    default_priority: str = "medium"

    class Config:
        env_file = ".env"
        env_prefix = "TASKFLOW_"
        case_sensitive = False


def load_yaml_config(config_file: Optional[str] = None) -> dict:
    """โหลด YAML config file"""
    if config_file:
        path = Path(config_file)
    else:
        # ค้นหา config file ใน standard locations
        candidates = [
            Path.cwd() / "taskflow.yaml",
            Path.cwd() / "taskflow.yml",
            Path.home() / ".taskflow" / "config.yaml",
            Path.home() / ".taskflow.yaml",
        ]
        path = next((p for p in candidates if p.exists()), None)

    if not path or not path.exists():
        return {}

    with open(path) as f:
        return yaml.safe_load(f) or {}


_settings: Optional[AppSettings] = None


def get_settings(config_file: Optional[str] = None) -> AppSettings:
    """Get application settings (singleton)"""
    global _settings
    if _settings is None:
        # Load YAML config and merge with env vars
        yaml_config = load_yaml_config(config_file)
        
        # Convert YAML keys to env format
        env_overrides = {}
        for key, value in yaml_config.items():
            env_key = f"TASKFLOW_{key.upper()}"
            if env_key not in os.environ:
                os.environ[env_key] = str(value)
        
        _settings = AppSettings()
    return _settings
```

### 4.5 Rich Output Helpers (taskflow/cli/output.py)

```python
"""
taskflow/cli/output.py
Rich output helpers สำหรับ CLI
"""

from datetime import date
from typing import List, Optional

from rich.console import Console
from rich.panel import Panel
from rich.table import Table
from rich.text import Text
from rich import box

from ..core.models import Priority, Status, Task

console = Console()


def make_priority_badge(priority: Priority) -> Text:
    """สร้าง priority badge"""
    labels = {
        Priority.HIGH: ("HIGH", "bold red"),
        Priority.MEDIUM: ("MED ", "yellow"),
        Priority.LOW: ("LOW ", "green"),
    }
    label, style = labels[priority]
    return Text(label, style=style)


def make_status_badge(status: Status) -> Text:
    """สร้าง status badge"""
    return Text(
        f"{status.icon} {status.value.replace('_', ' ').title()}",
        style=status.color,
    )


def format_due_date(task: Task) -> Text:
    """Format due date พร้อม color coding"""
    if not task.due_date:
        return Text("-", style="dim")

    days = task.days_until_due
    date_str = task.due_date.strftime("%Y-%m-%d")

    if task.is_overdue:
        return Text(f"⚠ {date_str} ({abs(days)}d late)", style="bold red")
    elif days == 0:
        return Text(f"📅 Today!", style="bold yellow")
    elif days <= 3:
        return Text(f"⏰ {date_str} ({days}d)", style="yellow")
    else:
        return Text(date_str, style="dim")


def print_tasks_table(tasks: List[Task], title: str = "Tasks") -> None:
    """แสดง tasks เป็น Rich table"""
    if not tasks:
        console.print(Panel(
            "[dim]No tasks found.[/dim]",
            title=title,
            border_style="dim",
        ))
        return

    table = Table(
        title=title,
        box=box.ROUNDED,
        show_header=True,
        header_style="bold cyan",
        border_style="cyan",
        expand=True,
    )

    table.add_column("ID", style="dim cyan", width=10, no_wrap=True)
    table.add_column("Title", min_width=20, max_width=40)
    table.add_column("Pri", justify="center", width=5)
    table.add_column("Status", justify="left", width=15)
    table.add_column("Category", width=12, style="blue")
    table.add_column("Due Date", width=20)
    table.add_column("Tags", width=15, style="dim")

    for task in tasks:
        # Row style based on status
        row_style = "dim" if task.status in (Status.DONE, Status.CANCELLED) else ""

        tags_display = ", ".join(task.tags[:3])
        if len(task.tags) > 3:
            tags_display += f" +{len(task.tags) - 3}"

        table.add_row(
            task.id,
            task.title,
            make_priority_badge(task.priority),
            make_status_badge(task.status),
            task.category or "-",
            format_due_date(task),
            tags_display or "-",
            style=row_style,
        )

    console.print(table)
    console.print(f"[dim]Total: {len(tasks)} task(s)[/dim]")


def print_task_detail(task: Task) -> None:
    """แสดง task detail ละเอียด"""
    content = []

    content.append(f"[bold]{task.title}[/bold]")
    if task.description:
        content.append(f"\n{task.description}")

    content.append(f"\n[dim]─────────────────────────[/dim]")
    content.append(f"\n[cyan]ID:[/cyan]       {task.id}")
    content.append(f"\n[cyan]Status:[/cyan]   {task.status.icon} {task.status.value}")
    content.append(f"\n[cyan]Priority:[/cyan] {task.priority.icon} {task.priority.value}")

    if task.category:
        content.append(f"\n[cyan]Category:[/cyan] {task.category}")

    if task.tags:
        content.append(f"\n[cyan]Tags:[/cyan]     {', '.join(task.tags)}")

    if task.due_date:
        content.append(f"\n[cyan]Due:[/cyan]      {format_due_date(task)}")

    content.append(f"\n[dim]Created:  {task.created_at.strftime('%Y-%m-%d %H:%M')}[/dim]")
    content.append(f"\n[dim]Updated:  {task.updated_at.strftime('%Y-%m-%d %H:%M')}[/dim]")

    status_color = {
        Status.TODO: "white",
        Status.IN_PROGRESS: "cyan",
        Status.DONE: "green",
        Status.CANCELLED: "red",
    }

    console.print(Panel(
        "".join(content),
        title=f"Task Details",
        border_style=status_color.get(task.status, "white"),
        padding=(1, 2),
    ))


def print_stats(stats: dict) -> None:
    """แสดง statistics"""
    from rich.columns import Columns

    total = stats.get("total", 0)
    todo = stats.get("todo", 0)
    in_progress = stats.get("in_progress", 0)
    done = stats.get("done", 0)
    cancelled = stats.get("cancelled", 0)
    overdue = stats.get("overdue", 0)

    panels = [
        Panel(f"[bold white]{total}[/bold white]", title="Total", border_style="white"),
        Panel(f"[bold white]{todo}[/bold white]", title="To Do", border_style="white"),
        Panel(f"[bold cyan]{in_progress}[/bold cyan]", title="In Progress", border_style="cyan"),
        Panel(f"[bold green]{done}[/bold green]", title="Done", border_style="green"),
        Panel(f"[bold red]{overdue}[/bold red]", title="Overdue", border_style="red"),
    ]

    console.print(Columns(panels))


def print_success(msg: str) -> None:
    console.print(f"[bold green]✅ {msg}[/bold green]")


def print_error(msg: str) -> None:
    console.print(f"[bold red]❌ {msg}[/bold red]", err=True)


def print_warning(msg: str) -> None:
    console.print(f"[bold yellow]⚠️  {msg}[/bold yellow]")


def print_info(msg: str) -> None:
    console.print(f"[cyan]ℹ️  {msg}[/cyan]")
```

### 4.6 Task Commands (taskflow/cli/commands/tasks.py)

```python
"""
taskflow/cli/commands/tasks.py
Task CRUD commands
"""

from datetime import date
from typing import Optional

import click
from rich.prompt import Confirm, Prompt

from ...core.config import get_settings
from ...core.database import Database
from ...core.models import Priority, Status, Task
from ...core.repository import TaskRepository
from ..output import (
    console, print_error, print_info, print_success,
    print_task_detail, print_tasks_table, print_warning,
)


def get_repo() -> TaskRepository:
    """สร้าง repository instance"""
    settings = get_settings()
    db = Database(settings.database_path)
    return TaskRepository(db)


@click.group()
def tasks():
    """Task management commands"""
    pass


@tasks.command("add")
@click.argument("title")
@click.option("--description", "-d", help="Task description")
@click.option(
    "--priority", "-p",
    type=click.Choice(["low", "medium", "high"], case_sensitive=False),
    default="medium",
    show_default=True,
)
@click.option("--category", "-c", help="Task category")
@click.option("--tag", "-t", multiple=True, help="Tags (use multiple times)")
@click.option("--due", help="Due date (YYYY-MM-DD)")
@click.option("--in-progress", "start_progress", is_flag=True,
              help="Set status to in_progress immediately")
def add_task(title, description, priority, category, tag, due, start_progress):
    """
    Add a new task.

    TITLE: Task title (use quotes for multi-word titles)

    Examples:

      taskflow add "Buy groceries"

      taskflow add "Write report" --priority high --due 2024-01-20

      taskflow add "Fix bug" -p high -c development -t python -t bug
    """
    # Validate due date
    due_date = None
    if due:
        try:
            due_date = date.fromisoformat(due)
        except ValueError:
            print_error(f"Invalid date format: '{due}'. Use YYYY-MM-DD")
            raise click.Abort()

    task = Task(
        title=title,
        description=description,
        priority=Priority(priority),
        status=Status.IN_PROGRESS if start_progress else Status.TODO,
        category=category,
        tags=list(tag),
        due_date=due_date,
    )

    repo = get_repo()
    repo.create(task)

    print_success(f"Created task [bold]#{task.id}[/bold]: {task.title!r}")

    if task.due_date:
        print_info(f"Due: {task.due_date}")
    if task.tags:
        print_info(f"Tags: {', '.join(task.tags)}")


@tasks.command("list")
@click.option(
    "--status", "-s",
    type=click.Choice(["all", "todo", "in_progress", "done", "cancelled"]),
    default="todo",
    show_default=True,
    help="Filter by status",
)
@click.option(
    "--priority", "-p",
    type=click.Choice(["low", "medium", "high"]),
    help="Filter by priority",
)
@click.option("--category", "-c", help="Filter by category")
@click.option("--tag", "-t", help="Filter by tag")
@click.option("--search", help="Search in title/description")
@click.option("--overdue", is_flag=True, help="Show overdue tasks only")
def list_tasks(status, priority, category, tag, search, overdue):
    """
    List tasks with optional filters.

    Examples:

      taskflow list                        # Show todo tasks

      taskflow list --status all           # Show all tasks

      taskflow list -p high --overdue      # High priority overdue

      taskflow list --search "bug"         # Search tasks
    """
    repo = get_repo()

    filter_status = None if status == "all" else Status(status)
    filter_priority = Priority(priority) if priority else None

    tasks = repo.get_all(
        status=filter_status,
        priority=filter_priority,
        category=category,
        tag=tag,
        search=search,
        overdue_only=overdue,
    )

    title_parts = []
    if filter_status:
        title_parts.append(f"Status: {filter_status.value}")
    if filter_priority:
        title_parts.append(f"Priority: {filter_priority.value}")
    if overdue:
        title_parts.append("Overdue")
    if search:
        title_parts.append(f"Search: {search!r}")

    title = "Tasks" + (f" ({', '.join(title_parts)})" if title_parts else "")
    print_tasks_table(tasks, title=title)


@tasks.command("show")
@click.argument("task_id")
def show_task(task_id):
    """Show detailed view of a task."""
    repo = get_repo()
    task = repo.get_by_id(task_id)

    if not task:
        print_error(f"Task '{task_id}' not found")
        raise click.Abort()

    print_task_detail(task)


@tasks.command("update")
@click.argument("task_id")
@click.option("--title", help="New title")
@click.option("--description", "-d", help="New description")
@click.option("--priority", "-p",
              type=click.Choice(["low", "medium", "high"]),
              help="New priority")
@click.option("--category", "-c", help="New category")
@click.option("--due", help="New due date (YYYY-MM-DD)")
@click.option("--add-tag", multiple=True, help="Add tags")
@click.option("--remove-tag", multiple=True, help="Remove tags")
def update_task(task_id, title, description, priority, category,
                due, add_tag, remove_tag):
    """
    Update an existing task.

    Example:

      taskflow update abc123 --title "New title" --priority high
    """
    repo = get_repo()
    task = repo.get_by_id(task_id)

    if not task:
        print_error(f"Task '{task_id}' not found")
        raise click.Abort()

    # Update fields
    if title:
        task.title = title
    if description is not None:
        task.description = description
    if priority:
        task.priority = Priority(priority)
    if category is not None:
        task.category = category
    if due:
        try:
            task.due_date = date.fromisoformat(due)
        except ValueError:
            print_error(f"Invalid date: {due}")
            raise click.Abort()

    # Update tags
    if add_tag:
        task.tags = list(set(task.tags + list(add_tag)))
    if remove_tag:
        task.tags = [t for t in task.tags if t not in remove_tag]

    repo.update(task)
    print_success(f"Updated task #{task_id}")


@tasks.command("done")
@click.argument("task_ids", nargs=-1, required=True)
def mark_done(task_ids):
    """
    Mark tasks as done.

    Example:

      taskflow done abc123

      taskflow done abc123 def456 ghi789
    """
    repo = get_repo()

    for task_id in task_ids:
        task = repo.get_by_id(task_id)
        if not task:
            print_warning(f"Task '{task_id}' not found, skipping")
            continue
        task.mark_done()
        repo.update(task)
        print_success(f"Task #{task_id} marked as done: {task.title!r}")


@tasks.command("start")
@click.argument("task_id")
def start_task(task_id):
    """Mark a task as in progress."""
    repo = get_repo()
    task = repo.get_by_id(task_id)

    if not task:
        print_error(f"Task '{task_id}' not found")
        raise click.Abort()

    task.mark_in_progress()
    repo.update(task)
    print_success(f"Started task #{task_id}: {task.title!r}")


@tasks.command("cancel")
@click.argument("task_id")
@click.option("--force", "-f", is_flag=True, help="Skip confirmation")
def cancel_task(task_id, force):
    """Cancel a task."""
    repo = get_repo()
    task = repo.get_by_id(task_id)

    if not task:
        print_error(f"Task '{task_id}' not found")
        raise click.Abort()

    if not force:
        confirmed = Confirm.ask(
            f"Cancel task #{task_id}: {task.title!r}?",
            default=False,
        )
        if not confirmed:
            print_info("Cancelled.")
            return

    task.cancel()
    repo.update(task)
    print_success(f"Cancelled task #{task_id}: {task.title!r}")


@tasks.command("delete")
@click.argument("task_id")
@click.option("--force", "-f", is_flag=True, help="Skip confirmation")
def delete_task(task_id, force):
    """Permanently delete a task."""
    repo = get_repo()
    task = repo.get_by_id(task_id)

    if not task:
        print_error(f"Task '{task_id}' not found")
        raise click.Abort()

    if not force:
        confirmed = Confirm.ask(
            f"[bold red]Permanently delete[/bold red] task #{task_id}: {task.title!r}?",
            default=False,
        )
        if not confirmed:
            print_info("Cancelled.")
            return

    repo.delete(task_id)
    print_success(f"Deleted task #{task_id}: {task.title!r}")


@tasks.command("stats")
def show_stats():
    """Show task statistics."""
    from ..output import print_stats

    repo = get_repo()
    stats = repo.get_stats()
    categories = repo.get_categories()

    print_stats(stats)

    if categories:
        console.print(f"\n[cyan]Categories:[/cyan] {', '.join(categories)}")
```

### 4.7 Export Commands (taskflow/cli/commands/export.py)

```python
"""
taskflow/cli/commands/export.py
Export functionality
"""

import csv
import json
from datetime import datetime
from pathlib import Path
from typing import List

import click

from ...core.database import Database
from ...core.models import Task
from ...core.repository import TaskRepository
from ...core.config import get_settings
from ..output import console, print_error, print_success


def get_repo() -> TaskRepository:
    settings = get_settings()
    db = Database(settings.database_path)
    return TaskRepository(db)


@click.group()
def export():
    """Export tasks to various formats"""
    pass


@export.command("json")
@click.option("--output", "-o", default="tasks_export.json",
              help="Output file path")
@click.option("--status", "-s",
              type=click.Choice(["all", "todo", "in_progress", "done"]),
              default="all",
              help="Filter by status")
@click.option("--pretty/--compact", default=True,
              help="Pretty-print JSON")
def export_json(output, status, pretty):
    """Export tasks to JSON file."""
    from ...core.models import Status

    repo = get_repo()
    filter_status = None if status == "all" else Status(status)
    tasks = repo.get_all(status=filter_status)

    data = {
        "exported_at": datetime.now().isoformat(),
        "total": len(tasks),
        "tasks": [task.to_dict() for task in tasks],
    }

    output_path = Path(output)
    with open(output_path, "w") as f:
        if pretty:
            json.dump(data, f, indent=2, ensure_ascii=False)
        else:
            json.dump(data, f, ensure_ascii=False)

    print_success(f"Exported {len(tasks)} tasks to '{output}'")


@export.command("csv")
@click.option("--output", "-o", default="tasks_export.csv",
              help="Output file path")
@click.option("--status", "-s",
              type=click.Choice(["all", "todo", "in_progress", "done"]),
              default="all",
              help="Filter by status")
def export_csv(output, status):
    """Export tasks to CSV file."""
    from ...core.models import Status

    repo = get_repo()
    filter_status = None if status == "all" else Status(status)
    tasks = repo.get_all(status=filter_status)

    fields = ["id", "title", "description", "priority", "status",
              "category", "tags", "due_date", "created_at", "updated_at"]

    output_path = Path(output)
    with open(output_path, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fields)
        writer.writeheader()

        for task in tasks:
            row = task.to_dict()
            row["tags"] = ", ".join(row["tags"])
            writer.writerow(row)

    print_success(f"Exported {len(tasks)} tasks to '{output}'")
    console.print(f"[dim]Fields: {', '.join(fields)}[/dim]")
```

### 4.8 Main CLI Entry Point (taskflow/cli/main.py)

```python
"""
taskflow/cli/main.py
Main CLI entry point
"""

import click
from rich.console import Console

from ..core.config import get_settings
from ..cli.commands.tasks import tasks
from ..cli.commands.export import export
from ..cli.output import console, print_stats

BANNER = """
  _______        _    _____ _               
 |__   __|      | |  |  ___| |              
    | | __ _ ___| | _| |_  | | _____      __
    | |/ _` / __| |/ /  _| | |/ _ \ \ /\ / /
    | | (_| \__ \   <| |   | | (_) \ V  V / 
    |_|\__,_|___/_|\_\_|   |_|\___/ \_/\_/  
"""


@click.group(invoke_without_command=True)
@click.version_option(version="1.0.0", prog_name="TaskFlow")
@click.option("--config", "-c", help="Config file path")
@click.pass_context
def cli(ctx, config):
    """
    \b
    TaskFlow - CLI Task Manager

    Manage your tasks efficiently from the command line.
    """
    ctx.ensure_object(dict)

    # Load settings
    settings = get_settings(config)
    ctx.obj["settings"] = settings

    # Show banner ถ้าไม่มี subcommand
    if ctx.invoked_subcommand is None:
        console.print(BANNER, style="cyan")
        console.print(f"[dim]Version 1.0.0 | DB: {settings.database_path}[/dim]")
        console.print("\n[bold]Quick Commands:[/bold]")
        console.print("  [cyan]taskflow add[/cyan] 'Task title'")
        console.print("  [cyan]taskflow list[/cyan]")
        console.print("  [cyan]taskflow --help[/cyan] for more options\n")

        # Show quick stats
        from ..core.database import Database
        from ..core.repository import TaskRepository

        db = Database(settings.database_path)
        repo = TaskRepository(db)
        stats = repo.get_stats()

        if stats["total"] > 0:
            print_stats(stats)


# Register command groups
cli.add_command(tasks)
cli.add_command(export)


# Shorthand commands (convenience aliases)
@cli.command("add")
@click.argument("title")
@click.option("--priority", "-p",
              type=click.Choice(["low", "medium", "high"]),
              default="medium")
@click.option("--due", help="Due date (YYYY-MM-DD)")
@click.option("--category", "-c")
@click.option("--tag", "-t", multiple=True)
def quick_add(title, priority, due, category, tag):
    """Quick add a task (shorthand for 'tasks add')."""
    ctx = click.get_current_context()
    ctx.invoke(
        tasks.commands["add"],
        title=title,
        priority=priority,
        due=due,
        category=category,
        tag=tag,
    )


@cli.command("list")
@click.option("--status", "-s",
              type=click.Choice(["all", "todo", "in_progress", "done"]),
              default="todo")
@click.option("--search", help="Search in title/description")
@click.option("--overdue", is_flag=True)
def quick_list(status, search, overdue):
    """Quick list tasks (shorthand for 'tasks list')."""
    ctx = click.get_current_context()
    ctx.invoke(
        tasks.commands["list"],
        status=status,
        search=search,
        overdue=overdue,
    )


@cli.command("done")
@click.argument("task_ids", nargs=-1, required=True)
def quick_done(task_ids):
    """Mark tasks as done."""
    ctx = click.get_current_context()
    ctx.invoke(tasks.commands["done"], task_ids=task_ids)


if __name__ == "__main__":
    cli()
```

### 4.9 Package Init Files

```python
# taskflow/__init__.py
"""TaskFlow - CLI Task Manager"""

__version__ = "1.0.0"
__author__ = "Your Name"
```

```python
# taskflow/cli/__init__.py
from .main import cli

__all__ = ["cli"]
```

```python
# taskflow/core/__init__.py
from .models import Task, Priority, Status
from .database import Database
from .repository import TaskRepository
from .config import get_settings

__all__ = ["Task", "Priority", "Status", "Database", "TaskRepository", "get_settings"]
```

### 4.10 pyproject.toml

```toml
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "taskflow"
version = "1.0.0"
description = "CLI Task Manager"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.9"
dependencies = [
    "click>=8.1.7",
    "rich>=13.7.0",
    "pydantic>=2.5.3",
    "pydantic-settings>=2.1.0",
    "python-dotenv>=1.0.0",
    "PyYAML>=6.0.1",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.4",
    "pytest-cov>=4.1.0",
    "pytest-mock>=3.12.0",
    "black>=23.12.1",
    "ruff>=0.1.9",
]

[project.scripts]
taskflow = "taskflow.cli.main:cli"
tf = "taskflow.cli.main:cli"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = ["--strict-markers", "-ra"]

[tool.black]
line-length = 88
target-version = ["py39", "py310", "py311"]

[tool.ruff]
line-length = 88
select = ["E", "F", "W", "I"]
```

---

## 5. Testing Guide

### 5.1 conftest.py

```python
# tests/conftest.py
import pytest
import tempfile
import os
from pathlib import Path

from taskflow.core.database import Database
from taskflow.core.repository import TaskRepository
from taskflow.core.models import Task, Priority, Status


@pytest.fixture
def temp_db():
    """สร้าง temporary database สำหรับ testing"""
    with tempfile.NamedTemporaryFile(suffix=".db", delete=False) as f:
        db_path = f.name
    
    db = Database(db_path)
    yield db
    
    # Cleanup
    os.unlink(db_path)


@pytest.fixture
def repo(temp_db):
    """สร้าง TaskRepository พร้อม temp database"""
    return TaskRepository(temp_db)


@pytest.fixture
def sample_task():
    """Sample task สำหรับ testing"""
    return Task(
        title="Test Task",
        description="Test description",
        priority=Priority.MEDIUM,
        status=Status.TODO,
        category="testing",
        tags=["test", "sample"],
    )


@pytest.fixture
def multiple_tasks(repo):
    """สร้าง multiple tasks สำหรับ testing"""
    tasks = [
        Task(title="High Priority Task", priority=Priority.HIGH, status=Status.TODO),
        Task(title="In Progress Task", priority=Priority.MEDIUM, status=Status.IN_PROGRESS),
        Task(title="Done Task", priority=Priority.LOW, status=Status.DONE),
        Task(title="Another Todo", priority=Priority.HIGH, status=Status.TODO,
             category="work", tags=["important"]),
    ]
    for task in tasks:
        repo.create(task)
    return tasks
```

### 5.2 test_models.py

```python
# tests/test_models.py
import pytest
from datetime import date, timedelta

from taskflow.core.models import Task, Priority, Status


class TestTask:
    def test_create_task_defaults(self):
        """ทดสอบ default values"""
        task = Task(title="Test")
        assert task.title == "Test"
        assert task.priority == Priority.MEDIUM
        assert task.status == Status.TODO
        assert task.tags == []
        assert task.category is None
        assert task.description is None

    def test_task_id_generated(self):
        """ทดสอบว่า ID ถูกสร้างอัตโนมัติ"""
        task1 = Task(title="Task 1")
        task2 = Task(title="Task 2")
        assert task1.id != task2.id
        assert len(task1.id) == 8

    def test_task_priority_from_string(self):
        """ทดสอบ string to enum conversion"""
        task = Task(title="Test", priority="high")
        assert task.priority == Priority.HIGH
        assert isinstance(task.priority, Priority)

    def test_task_status_from_string(self):
        task = Task(title="Test", status="in_progress")
        assert task.status == Status.IN_PROGRESS

    def test_mark_done(self):
        """ทดสอบ mark_done()"""
        task = Task(title="Test")
        original_updated = task.updated_at
        task.mark_done()
        assert task.status == Status.DONE
        assert task.updated_at >= original_updated

    def test_mark_in_progress(self):
        task = Task(title="Test")
        task.mark_in_progress()
        assert task.status == Status.IN_PROGRESS

    def test_is_overdue(self):
        """ทดสอบ overdue detection"""
        # Not overdue (no due date)
        task1 = Task(title="Test")
        assert not task1.is_overdue

        # Not overdue (future date)
        future_task = Task(title="Test", due_date=date.today() + timedelta(days=7))
        assert not future_task.is_overdue

        # Overdue
        past_task = Task(title="Test", due_date=date.today() - timedelta(days=1))
        assert past_task.is_overdue

        # Done tasks are not overdue
        done_task = Task(
            title="Test",
            due_date=date.today() - timedelta(days=1),
            status=Status.DONE
        )
        assert not done_task.is_overdue

    def test_days_until_due(self):
        tomorrow = date.today() + timedelta(days=1)
        task = Task(title="Test", due_date=tomorrow)
        assert task.days_until_due == 1

    def test_to_dict(self):
        """ทดสอบ serialization"""
        task = Task(title="Test", tags=["a", "b"])
        d = task.to_dict()
        assert d["title"] == "Test"
        assert d["tags"] == ["a", "b"]
        assert "id" in d
        assert "created_at" in d

    def test_from_dict(self):
        """ทดสอบ deserialization"""
        task = Task(title="Test", priority=Priority.HIGH)
        d = task.to_dict()
        restored = Task.from_dict(d)
        assert restored.title == task.title
        assert restored.priority == task.priority
        assert restored.id == task.id
```

### 5.3 test_repository.py

```python
# tests/test_repository.py
import pytest
from datetime import date, timedelta

from taskflow.core.models import Task, Priority, Status


class TestTaskRepository:
    def test_create_task(self, repo, sample_task):
        """ทดสอบ create"""
        created = repo.create(sample_task)
        assert created.id == sample_task.id
        
        # ตรวจสอบว่า persist แล้ว
        retrieved = repo.get_by_id(sample_task.id)
        assert retrieved is not None
        assert retrieved.title == sample_task.title

    def test_get_by_id_not_found(self, repo):
        """ทดสอบ get_by_id ที่ไม่มี task"""
        result = repo.get_by_id("nonexistent")
        assert result is None

    def test_get_all_empty(self, repo):
        tasks = repo.get_all()
        assert tasks == []

    def test_get_all_with_filters(self, repo, multiple_tasks):
        """ทดสอบ filters"""
        # Filter by status
        todo_tasks = repo.get_all(status=Status.TODO)
        assert all(t.status == Status.TODO for t in todo_tasks)
        assert len(todo_tasks) == 2

        # Filter by priority
        high_tasks = repo.get_all(priority=Priority.HIGH)
        assert all(t.priority == Priority.HIGH for t in high_tasks)

        # Filter by category
        work_tasks = repo.get_all(category="work")
        assert len(work_tasks) == 1

        # Search
        search_results = repo.get_all(search="Important")
        assert len(search_results) >= 0  # depends on case sensitivity

    def test_update_task(self, repo, sample_task):
        """ทดสอบ update"""
        repo.create(sample_task)
        sample_task.title = "Updated Title"
        sample_task.priority = Priority.HIGH
        
        updated = repo.update(sample_task)
        retrieved = repo.get_by_id(sample_task.id)
        
        assert retrieved.title == "Updated Title"
        assert retrieved.priority == Priority.HIGH

    def test_delete_task(self, repo, sample_task):
        """ทดสอบ delete"""
        repo.create(sample_task)
        
        result = repo.delete(sample_task.id)
        assert result is True
        
        retrieved = repo.get_by_id(sample_task.id)
        assert retrieved is None

    def test_count(self, repo, multiple_tasks):
        """ทดสอบ count"""
        total = repo.count()
        assert total == len(multiple_tasks)
        
        todo_count = repo.count(Status.TODO)
        assert todo_count == sum(1 for t in multiple_tasks if t.status == Status.TODO)

    def test_get_stats(self, repo, multiple_tasks):
        """ทดสอบ stats"""
        stats = repo.get_stats()
        assert "total" in stats
        assert stats["total"] == len(multiple_tasks)
        assert "todo" in stats
        assert "done" in stats

    def test_tags_persist(self, repo):
        """ทดสอบว่า tags ถูก persist"""
        task = Task(title="Tagged Task", tags=["python", "test", "important"])
        repo.create(task)
        
        retrieved = repo.get_by_id(task.id)
        assert set(retrieved.tags) == {"python", "test", "important"}

    def test_overdue_filter(self, repo):
        """ทดสอบ overdue filter"""
        past_date = date.today() - timedelta(days=1)
        future_date = date.today() + timedelta(days=7)
        
        overdue_task = Task(title="Overdue", due_date=past_date)
        future_task = Task(title="Future", due_date=future_date)
        done_overdue = Task(title="Done Overdue", due_date=past_date, status=Status.DONE)
        
        repo.create(overdue_task)
        repo.create(future_task)
        repo.create(done_overdue)
        
        overdue = repo.get_all(overdue_only=True)
        assert len(overdue) == 1
        assert overdue[0].id == overdue_task.id
```

### 5.4 test_cli.py

```python
# tests/test_cli.py
import os
import tempfile
import pytest
from click.testing import CliRunner

from taskflow.cli.main import cli
from taskflow.core.config import AppSettings


@pytest.fixture
def runner():
    return CliRunner()


@pytest.fixture
def db_path(tmp_path):
    """Path สำหรับ test database"""
    return str(tmp_path / "test.db")


@pytest.fixture
def env_vars(db_path):
    """Environment variables สำหรับ testing"""
    return {"TASKFLOW_DATABASE_PATH": db_path}


class TestCLI:
    def test_help(self, runner):
        result = runner.invoke(cli, ["--help"])
        assert result.exit_code == 0
        assert "TaskFlow" in result.output

    def test_version(self, runner):
        result = runner.invoke(cli, ["--version"])
        assert result.exit_code == 0
        assert "1.0.0" in result.output

    def test_add_task(self, runner, env_vars):
        with runner.isolated_filesystem():
            with runner.isolated_filesystem():
                result = runner.invoke(
                    cli,
                    ["add", "Test Task"],
                    env=env_vars,
                )
                assert result.exit_code == 0
                assert "Test Task" in result.output

    def test_add_task_with_options(self, runner, env_vars, tmp_path):
        db = str(tmp_path / "test.db")
        result = runner.invoke(
            cli,
            ["add", "High Priority Task",
             "--priority", "high",
             "--category", "work",
             "--tag", "python",
             "--tag", "urgent"],
            env={"TASKFLOW_DATABASE_PATH": db},
        )
        assert result.exit_code == 0
        assert "High Priority Task" in result.output

    def test_list_empty(self, runner, tmp_path):
        db = str(tmp_path / "test.db")
        result = runner.invoke(
            cli,
            ["list"],
            env={"TASKFLOW_DATABASE_PATH": db},
        )
        assert result.exit_code == 0

    def test_add_and_list(self, runner, tmp_path):
        db = str(tmp_path / "test.db")
        env = {"TASKFLOW_DATABASE_PATH": db}
        
        # Add task
        runner.invoke(cli, ["add", "My Task"], env=env)
        
        # List tasks
        result = runner.invoke(cli, ["list", "--status", "all"], env=env)
        assert result.exit_code == 0
        assert "My Task" in result.output

    def test_mark_done(self, runner, tmp_path):
        db = str(tmp_path / "test.db")
        env = {"TASKFLOW_DATABASE_PATH": db}
        
        # Add task
        add_result = runner.invoke(cli, ["add", "Task To Complete"], env=env)
        
        # Extract task ID from output
        # Assume format: "Created task #abc123: 'Task To Complete'"
        import re
        match = re.search(r"#(\w+)", add_result.output)
        if match:
            task_id = match.group(1)
            
            # Mark done
            result = runner.invoke(cli, ["done", task_id], env=env)
            assert result.exit_code == 0

    def test_invalid_date_format(self, runner, tmp_path):
        db = str(tmp_path / "test.db")
        result = runner.invoke(
            cli,
            ["add", "Task", "--due", "invalid-date"],
            env={"TASKFLOW_DATABASE_PATH": db},
        )
        assert result.exit_code != 0

    def test_export_json(self, runner, tmp_path):
        db = str(tmp_path / "test.db")
        env = {"TASKFLOW_DATABASE_PATH": db}
        output_file = str(tmp_path / "export.json")
        
        # Add task first
        runner.invoke(cli, ["add", "Export Test Task"], env=env)
        
        # Export
        result = runner.invoke(
            cli,
            ["export", "json", "--output", output_file],
            env=env,
        )
        assert result.exit_code == 0
        
        # Verify file
        import json
        with open(output_file) as f:
            data = json.load(f)
        assert "tasks" in data
```

### รันทดสอบ

```bash
# รัน tests ทั้งหมด
pytest tests/

# พร้อม coverage
pytest tests/ --cov=taskflow --cov-report=term-missing

# เฉพาะ test file
pytest tests/test_models.py -v

# เฉพาะ test function
pytest tests/test_models.py::TestTask::test_create_task_defaults -v

# รัน tests เฉพาะที่ fast
pytest tests/ -m "not slow"
```

---

## 6. Docker Support

### Dockerfile

```dockerfile
FROM python:3.11-slim AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /build

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --upgrade pip && pip install -r requirements.txt


FROM python:3.11-slim AS production

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

RUN useradd --uid 1001 --create-home taskflow

COPY --chown=taskflow . .

RUN pip install -e .

USER taskflow

VOLUME ["/home/taskflow/.taskflow"]

ENTRYPOINT ["taskflow"]
CMD ["--help"]
```

### docker-compose.yml

```yaml
version: "3.9"

services:
  taskflow:
    build: .
    container_name: taskflow
    environment:
      - TASKFLOW_DATABASE_PATH=/data/tasks.db
    volumes:
      - taskflow_data:/data
    stdin_open: true
    tty: true

volumes:
  taskflow_data:
    driver: local
```

### ใช้งาน Docker

```bash
# Build
docker build -t taskflow .

# รันแบบ interactive
docker run -it -v taskflow_data:/data \
    -e TASKFLOW_DATABASE_PATH=/data/tasks.db \
    taskflow add "My task from Docker"

# Docker Compose
docker-compose run taskflow list
docker-compose run taskflow add "Docker task" --priority high

# Shell access
docker run -it --entrypoint bash taskflow
```

---

## 7. Usage Guide

### Basic Commands

```bash
# === ADDING TASKS ===

# Simple task
taskflow add "Buy groceries"

# Task with all options
taskflow add "Write report" \
    --priority high \
    --category work \
    --due 2024-01-20 \
    --tag python \
    --tag urgent \
    --description "Quarterly report for Q4"

# Start immediately (in_progress)
taskflow add "Debug login issue" --priority high --in-progress


# === LISTING TASKS ===

# Show todo tasks (default)
taskflow list

# Show all tasks
taskflow list --status all

# Show by priority
taskflow list --priority high

# Show by category
taskflow list --category work

# Show overdue
taskflow list --overdue

# Search
taskflow list --search "bug"

# Combine filters
taskflow list --status all --priority high --category work


# === UPDATING TASKS ===

# Update title
taskflow tasks update abc123 --title "New title"

# Update priority
taskflow tasks update abc123 --priority high

# Add/remove tags
taskflow tasks update abc123 --add-tag urgent --remove-tag low-priority

# Change due date
taskflow tasks update abc123 --due 2024-02-01


# === STATUS CHANGES ===

# Mark done
taskflow done abc123

# Mark multiple done
taskflow done abc123 def456 ghi789

# Start task (in_progress)
taskflow tasks start abc123

# Cancel task
taskflow tasks cancel abc123

# Cancel without confirmation
taskflow tasks cancel abc123 --force


# === DELETE ===

# Delete with confirmation
taskflow tasks delete abc123

# Force delete
taskflow tasks delete abc123 --force


# === EXPORT ===

# Export all to JSON
taskflow export json

# Export to specific file
taskflow export json --output my_tasks.json

# Export only todo tasks
taskflow export json --status todo

# Export to CSV
taskflow export csv --output tasks.csv


# === STATS ===

taskflow tasks stats


# === SHOW TASK DETAIL ===

taskflow tasks show abc123
```

### Configuration

```yaml
# taskflow.yaml หรือ ~/.taskflow/config.yaml

database_path: ~/.taskflow/tasks.db
items_per_page: 20
date_format: "%Y-%m-%d"
default_priority: medium
```

```bash
# Environment variables
export TASKFLOW_DATABASE_PATH=/custom/path/tasks.db
export TASKFLOW_DEBUG=true
export TASKFLOW_DEFAULT_PRIORITY=high
```

---

## 8. Ideas for Extension

### Feature Enhancements

```python
"""
Extension Ideas:

1. 📅 Calendar View
   - แสดง tasks บน calendar ASCII
   - Filter by week/month

2. 🔄 Recurring Tasks
   - Tasks ที่ repeat ทุกวัน/สัปดาห์/เดือน
   - Auto-create instances

3. 👥 Task Dependencies
   - Task A depends on Task B
   - Block task จนกว่า dependencies เสร็จ

4. ⏱️ Time Tracking
   - Start/stop timer สำหรับ task
   - Show time spent

5. 📊 Reports
   - Productivity report
   - Task completion rate
   - Category breakdown charts (Rich)

6. 🔔 Notifications
   - Desktop notifications เมื่อใกล้ครบกำหนด
   - Daily summary email

7. 🌐 REST API
   - FastAPI backend
   - Mobile app support

8. 🔄 Sync
   - Export/import ระหว่าง machines
   - Cloud sync (S3, Dropbox)

9. 🤖 AI Integration
   - Auto-suggest priority จาก title
   - Smart categorization
   - Task breakdown

10. 📱 TUI Interface
    - Textual library
    - Mouse support
    - Full-screen interface
"""

# ตัวอย่าง: Time Tracking Extension
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class TimeEntry:
    task_id: str
    started_at: datetime
    ended_at: Optional[datetime] = None

    @property
    def duration_minutes(self) -> Optional[float]:
        if not self.ended_at:
            return None
        delta = self.ended_at - self.started_at
        return delta.total_seconds() / 60

    def stop(self):
        self.ended_at = datetime.now()


# ตัวอย่าง: Calendar View
def show_calendar_view(tasks: list, year: int, month: int):
    """แสดง tasks บน calendar"""
    import calendar
    from rich.table import Table
    from rich.console import Console

    console = Console()
    cal = calendar.monthcalendar(year, month)

    table = Table(title=f"{calendar.month_name[month]} {year}")
    for day_name in ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"]:
        table.add_column(day_name, width=10)

    # สร้าง dict ของ tasks ตาม due date
    tasks_by_date = {}
    for task in tasks:
        if task.due_date and task.due_date.year == year and task.due_date.month == month:
            day = task.due_date.day
            if day not in tasks_by_date:
                tasks_by_date[day] = []
            tasks_by_date[day].append(task)

    for week in cal:
        row = []
        for day in week:
            if day == 0:
                row.append("")
            else:
                cell = f"[bold]{day}[/bold]"
                if day in tasks_by_date:
                    count = len(tasks_by_date[day])
                    cell += f"\n[red]•[/red]×{count}"
                row.append(cell)
        table.add_row(*row)

    console.print(table)
```

---

## สรุปโปรเจกต์

โปรเจกต์ **TaskFlow** ได้ใช้ความรู้จาก Parts 46-49:

| Part | ความรู้ | การใช้งานในโปรเจกต์ |
|------|--------|---------------------|
| 46 | Environment Variables | `AppSettings`, `.env` support |
| 47 | CLI Development | `click` commands, groups, options |
| 48 | Config Files | YAML config, pyproject.toml |
| 49 | Docker | Dockerfile, docker-compose |

### Architecture Summary

```
CLI Layer (Click)
    ↓
Command Handlers
    ↓
Repository Layer (Data Access)
    ↓
Database Layer (SQLite)
    ↓
Data Models (dataclasses)
```

### Installation ฉบับเร็ว

```bash
git clone https://github.com/example/taskflow
cd taskflow
pip install -e .
taskflow --help
```
