# Part 47: CLI Development - argparse & Click

## สารบัญ

1. [Command-line Arguments คืออะไร?](#1-command-line-arguments-คืออะไร)
2. [sys.argv พื้นฐาน](#2-sysargv-พื้นฐาน)
3. [argparse Module](#3-argparse-module)
4. [Click Library](#4-click-library)
5. [Rich Library สำหรับ Beautiful CLI](#5-rich-library-สำหรับ-beautiful-cli)
6. [Typer - Modern CLI Framework](#6-typer---modern-cli-framework)
7. [โปรแกรมตัวอย่างจริง](#7-โปรแกรมตัวอย่างจริง)
8. [แบบฝึกหัด](#8-แบบฝึกหัด)

---

## 1. Command-line Arguments คืออะไร?

**Command-line arguments** คือ parameters ที่ส่งให้โปรแกรมผ่าน terminal เมื่อ start โปรแกรม

```bash
# รูปแบบทั่วไป
python program.py [arguments]

# ตัวอย่าง
python greet.py --name "Alice" --greeting "Hello"
python convert.py input.txt output.json --format json
python server.py --host 0.0.0.0 --port 8080 --debug
```

### ประเภทของ Arguments

| ประเภท | ตัวอย่าง | คำอธิบาย |
|--------|---------|----------|
| Positional | `python prog.py file.txt` | ระบุตำแหน่ง |
| Optional (flag) | `python prog.py --verbose` | มีหรือไม่มีก็ได้ |
| Optional (value) | `python prog.py --port 8080` | มีค่ากำกับ |
| Short option | `python prog.py -v -p 8080` | ตัวย่อ |

---

## 2. sys.argv พื้นฐาน

`sys.argv` คือ list ของ arguments ที่ส่งให้โปรแกรมผ่าน command line

### ตัวอย่างที่ 1: sys.argv พื้นฐาน

```python
# basic_args.py
import sys

# sys.argv[0] คือชื่อ script เสมอ
print(f"Script name: {sys.argv[0]}")
print(f"All arguments: {sys.argv}")
print(f"Number of args: {len(sys.argv)}")

# อ่าน argument แรก
if len(sys.argv) > 1:
    first_arg = sys.argv[1]
    print(f"First argument: {first_arg}")
else:
    print("No arguments provided!")

# รันด้วย: python basic_args.py hello world
# Output:
# Script name: basic_args.py
# All arguments: ['basic_args.py', 'hello', 'world']
# Number of args: 3
# First argument: hello
```

### ตัวอย่างที่ 2: Manual Argument Parsing

```python
# manual_parse.py
import sys

def parse_args(argv: list) -> dict:
    """Parse arguments manually (สำหรับเข้าใจ concept)"""
    args = {
        "positional": [],
        "verbose": False,
        "output": None,
        "format": "text",
    }
    
    i = 1  # Skip script name
    while i < len(argv):
        arg = argv[i]
        
        if arg in ("-v", "--verbose"):
            args["verbose"] = True
        elif arg in ("-o", "--output"):
            if i + 1 < len(argv):
                args["output"] = argv[i + 1]
                i += 1  # Skip next arg (value)
        elif arg in ("-f", "--format"):
            if i + 1 < len(argv):
                args["format"] = argv[i + 1]
                i += 1
        elif not arg.startswith("-"):
            args["positional"].append(arg)
        
        i += 1
    
    return args


# รันด้วย: python manual_parse.py file.txt -v -o output.txt -f json
result = parse_args(sys.argv)
print(result)
```

---

## 3. argparse Module

**argparse** เป็น standard library module สำหรับ parse command-line arguments อย่างมืออาชีพ

### ตัวอย่างที่ 3: argparse พื้นฐาน

```python
# greet.py
import argparse

def main():
    # สร้าง parser
    parser = argparse.ArgumentParser(
        description="A greeting program",
        epilog="Example: python greet.py Alice --greeting Hi"
    )
    
    # Positional argument (required)
    parser.add_argument(
        "name",
        help="Name of the person to greet"
    )
    
    # Optional argument with default
    parser.add_argument(
        "--greeting", "-g",
        default="Hello",
        help="Greeting to use (default: Hello)"
    )
    
    # Boolean flag
    parser.add_argument(
        "--uppercase", "-u",
        action="store_true",
        help="Display name in uppercase"
    )
    
    # Parse arguments
    args = parser.parse_args()
    
    # ใช้ arguments
    name = args.name.upper() if args.uppercase else args.name
    print(f"{args.greeting}, {name}!")


if __name__ == "__main__":
    main()

# รันด้วย:
# python greet.py Alice
# python greet.py Alice --greeting "Hi"
# python greet.py alice --uppercase
# python greet.py --help  (แสดง help อัตโนมัติ)
```

### ตัวอย่างที่ 4: Argument Types

```python
# types_demo.py
import argparse

def main():
    parser = argparse.ArgumentParser(description="Type conversion demo")
    
    # Integer
    parser.add_argument(
        "--port", "-p",
        type=int,
        default=8000,
        help="Port number (int)"
    )
    
    # Float
    parser.add_argument(
        "--rate",
        type=float,
        default=1.0,
        help="Rate multiplier (float)"
    )
    
    # Choices (เลือกได้เฉพาะค่าที่กำหนด)
    parser.add_argument(
        "--format", "-f",
        choices=["json", "csv", "xml", "text"],
        default="text",
        help="Output format"
    )
    
    # Multiple values (nargs)
    parser.add_argument(
        "--files",
        nargs="+",          # หนึ่งหรือมากกว่า
        help="Input files"
    )
    
    # Fixed number of values
    parser.add_argument(
        "--coords",
        nargs=2,
        type=float,
        metavar=("LAT", "LON"),
        help="Coordinates: latitude longitude"
    )
    
    # Append multiple values
    parser.add_argument(
        "--tag", "-t",
        action="append",
        dest="tags",
        help="Add tag (can be used multiple times)"
    )
    
    args = parser.parse_args()
    print(f"Port: {args.port} ({type(args.port).__name__})")
    print(f"Rate: {args.rate}")
    print(f"Format: {args.format}")
    print(f"Files: {args.files}")
    print(f"Coords: {args.coords}")
    print(f"Tags: {args.tags}")


if __name__ == "__main__":
    main()

# รันด้วย:
# python types_demo.py --port 9000 --format json --files a.txt b.txt --tag python --tag cli
```

### ตัวอย่างที่ 5: Subcommands (Subparsers)

```python
# git_like.py - โปรแกรมที่มี subcommands แบบ git
import argparse

def cmd_init(args):
    """สร้าง repository ใหม่"""
    name = args.name if args.name else "."
    print(f"Initialized empty repository in {name}")

def cmd_add(args):
    """เพิ่ม files"""
    for file in args.files:
        print(f"Adding file: {file}")
    if args.all:
        print("Adding all changed files")

def cmd_commit(args):
    """Commit changes"""
    print(f"Committing with message: '{args.message}'")
    if args.author:
        print(f"Author: {args.author}")

def cmd_log(args):
    """แสดง commit history"""
    limit = args.limit if args.limit else "all"
    print(f"Showing {limit} commits")
    if args.oneline:
        print("(one-line format)")

def main():
    # Main parser
    parser = argparse.ArgumentParser(
        description="Simple VCS tool",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Commands:
  init    Initialize a new repository
  add     Add files to staging
  commit  Record changes
  log     Show commit history
        """
    )
    parser.add_argument("--version", action="version", version="1.0.0")
    
    # Subparsers
    subparsers = parser.add_subparsers(
        title="commands",
        dest="command",      # เก็บชื่อ subcommand ไว้ใน args.command
        help="Available commands"
    )
    subparsers.required = True  # ต้องระบุ subcommand
    
    # 'init' subcommand
    init_parser = subparsers.add_parser("init", help="Initialize repository")
    init_parser.add_argument("name", nargs="?", help="Repository name")
    init_parser.set_defaults(func=cmd_init)
    
    # 'add' subcommand
    add_parser = subparsers.add_parser("add", help="Add files to staging")
    add_parser.add_argument("files", nargs="*", help="Files to add")
    add_parser.add_argument("--all", "-A", action="store_true", help="Add all files")
    add_parser.set_defaults(func=cmd_add)
    
    # 'commit' subcommand
    commit_parser = subparsers.add_parser("commit", help="Record changes")
    commit_parser.add_argument("--message", "-m", required=True, help="Commit message")
    commit_parser.add_argument("--author", help="Override author")
    commit_parser.set_defaults(func=cmd_commit)
    
    # 'log' subcommand
    log_parser = subparsers.add_parser("log", help="Show commit history")
    log_parser.add_argument("--limit", "-n", type=int, help="Number of commits")
    log_parser.add_argument("--oneline", action="store_true", help="One line per commit")
    log_parser.set_defaults(func=cmd_log)
    
    # Parse and dispatch
    args = parser.parse_args()
    args.func(args)


if __name__ == "__main__":
    main()

# รันด้วย:
# python git_like.py init myrepo
# python git_like.py add file1.py file2.py
# python git_like.py commit -m "Initial commit"
# python git_like.py log --limit 5 --oneline
```

### ตัวอย่างที่ 6: Argument Groups

```python
# argument_groups.py
import argparse

def main():
    parser = argparse.ArgumentParser(
        description="Server configuration demo"
    )
    
    # Required arguments group
    required = parser.add_argument_group("required arguments")
    required.add_argument(
        "--config", "-c",
        required=True,
        help="Path to config file"
    )
    
    # Server settings group
    server_group = parser.add_argument_group("server settings")
    server_group.add_argument("--host", default="0.0.0.0", help="Host to bind")
    server_group.add_argument("--port", "-p", type=int, default=8000, help="Port")
    server_group.add_argument("--workers", type=int, default=1, help="Worker count")
    
    # Debug group
    debug_group = parser.add_argument_group("debugging")
    debug_group.add_argument("--debug", action="store_true", help="Enable debug mode")
    debug_group.add_argument("--verbose", "-v", action="count", default=0,
                             help="Increase verbosity (-v, -vv, -vvv)")
    debug_group.add_argument("--log-file", help="Log to file")
    
    # Mutually exclusive group (เลือกได้แค่หนึ่ง)
    output_group = parser.add_mutually_exclusive_group()
    output_group.add_argument("--quiet", "-q", action="store_true", help="Suppress output")
    output_group.add_argument("--verbose-output", "-V", action="store_true",
                              help="Verbose output")
    
    args = parser.parse_args()
    
    print(f"Config: {args.config}")
    print(f"Server: {args.host}:{args.port}")
    print(f"Workers: {args.workers}")
    print(f"Verbosity: {args.verbose}")
    
    if args.quiet:
        print("Quiet mode")
    elif args.verbose_output:
        print("Verbose output mode")


if __name__ == "__main__":
    main()
```

### ตัวอย่างที่ 7: Custom Action

```python
# custom_action.py
import argparse

class KeyValueAction(argparse.Action):
    """Custom action สำหรับ key=value arguments"""
    
    def __call__(self, parser, namespace, values, option_string=None):
        # Initialize dict ถ้ายังไม่มี
        if not hasattr(namespace, "env") or getattr(namespace, "env") is None:
            setattr(namespace, "env", {})
        
        # Parse key=value
        for item in values:
            if "=" not in item:
                parser.error(f"Invalid format: '{item}'. Use KEY=VALUE")
            key, _, value = item.partition("=")
            getattr(namespace, "env")[key.strip()] = value.strip()


class ByteSizeAction(argparse.Action):
    """Custom action แปลง '10MB' เป็น bytes"""
    
    UNITS = {
        "B": 1,
        "KB": 1024,
        "MB": 1024 ** 2,
        "GB": 1024 ** 3,
    }
    
    def __call__(self, parser, namespace, values, option_string=None):
        value = values.upper()
        for unit, multiplier in self.UNITS.items():
            if value.endswith(unit):
                number = float(value[:-len(unit)])
                setattr(namespace, self.dest, int(number * multiplier))
                return
        
        try:
            setattr(namespace, self.dest, int(values))
        except ValueError:
            parser.error(f"Invalid size: '{values}'. Use format: 100MB, 1GB, etc.")


def main():
    parser = argparse.ArgumentParser()
    
    parser.add_argument(
        "--env", "-e",
        nargs="+",
        action=KeyValueAction,
        help="Set environment variables (KEY=VALUE)"
    )
    
    parser.add_argument(
        "--max-size",
        action=ByteSizeAction,
        default=100 * 1024 * 1024,  # 100MB
        help="Maximum file size (e.g., 10MB, 1GB)"
    )
    
    args = parser.parse_args()
    print(f"Env vars: {args.env}")
    print(f"Max size: {args.max_size:,} bytes")


if __name__ == "__main__":
    main()

# รันด้วย:
# python custom_action.py --env DB_HOST=localhost DB_PORT=5432 --max-size 50MB
```

---

## 4. Click Library

**Click** (Command Line Interface Creation Kit) เป็น library ที่ใช้ decorators ทำให้ code อ่านง่ายและ maintainable กว่า argparse

### การติดตั้ง

```bash
pip install click
```

### ตัวอย่างที่ 8: Click พื้นฐาน

```python
# click_basic.py
import click

@click.command()
@click.argument("name")
@click.option("--greeting", "-g", default="Hello", help="Greeting to use")
@click.option("--uppercase", "-u", is_flag=True, help="Uppercase the name")
@click.option("--count", "-n", default=1, type=int, help="Number of times to greet")
def greet(name, greeting, uppercase, count):
    """
    Greet a person.
    
    NAME is the person to greet.
    """
    display_name = name.upper() if uppercase else name
    for _ in range(count):
        click.echo(f"{greeting}, {display_name}!")

if __name__ == "__main__":
    greet()

# รันด้วย:
# python click_basic.py Alice
# python click_basic.py Alice --greeting Hi --count 3
# python click_basic.py alice --uppercase
# python click_basic.py --help
```

### ตัวอย่างที่ 9: Click Options ครบถ้วน

```python
# click_options.py
import click
from pathlib import Path

@click.command()
# Required option (no default)
@click.option("--output", "-o", required=True, type=click.Path(), help="Output file")

# File option (ต้องเป็น file จริง)
@click.option(
    "--input", "-i",
    type=click.File("r"),
    default="-",  # stdin by default
    help="Input file (default: stdin)"
)

# Choice option
@click.option(
    "--format", "-f",
    type=click.Choice(["json", "csv", "xml"], case_sensitive=False),
    default="json",
    show_default=True,
    help="Output format"
)

# Integer with range
@click.option(
    "--workers", "-w",
    type=click.IntRange(1, 32),
    default=4,
    show_default=True,
    help="Number of workers (1-32)"
)

# Multiple values
@click.option(
    "--exclude", "-e",
    multiple=True,
    help="Patterns to exclude (can be used multiple times)"
)

# Hidden option (ไม่แสดงใน help)
@click.option("--secret", hidden=True, help="Secret option")

# Boolean flag ที่มีทั้ง on/off
@click.option("--verbose/--no-verbose", "-v/-V", default=False)

def process(output, input, format, workers, exclude, secret, verbose):
    """Process files with various options."""
    click.echo(f"Output: {output}")
    click.echo(f"Format: {format}")
    click.echo(f"Workers: {workers}")
    click.echo(f"Exclude patterns: {exclude}")
    click.echo(f"Verbose: {verbose}")
    
    content = input.read()
    click.echo(f"Read {len(content)} bytes from input")

if __name__ == "__main__":
    process()
```

### ตัวอย่างที่ 10: Click Groups (Subcommands)

```python
# click_groups.py
import click

@click.group()
@click.option("--verbose", "-v", is_flag=True, help="Enable verbose output")
@click.pass_context
def cli(ctx, verbose):
    """
    Task Management Tool
    
    Manage your tasks from the command line.
    """
    # เก็บ context สำหรับ subcommands
    ctx.ensure_object(dict)
    ctx.obj["verbose"] = verbose
    
    if verbose:
        click.echo("Verbose mode enabled")


@cli.command()
@click.argument("title")
@click.option("--priority", "-p",
              type=click.Choice(["low", "medium", "high"]),
              default="medium")
@click.option("--due-date", "-d", help="Due date (YYYY-MM-DD)")
@click.pass_context
def add(ctx, title, priority, due_date):
    """Add a new task."""
    verbose = ctx.obj["verbose"]
    if verbose:
        click.echo(f"Creating task: {title!r}")
    
    click.echo(f"✅ Added task: {title!r} [{priority}]")
    if due_date:
        click.echo(f"   Due: {due_date}")


@cli.command()
@click.option("--all", "show_all", is_flag=True, help="Show all tasks including done")
@click.option("--priority", "-p",
              type=click.Choice(["low", "medium", "high"]),
              help="Filter by priority")
@click.pass_context
def list(ctx, show_all, priority):
    """List tasks."""
    # Mock data
    tasks = [
        {"id": 1, "title": "Buy groceries", "priority": "low", "done": False},
        {"id": 2, "title": "Write report", "priority": "high", "done": False},
        {"id": 3, "title": "Call doctor", "priority": "medium", "done": True},
    ]
    
    # Filter
    if not show_all:
        tasks = [t for t in tasks if not t["done"]]
    if priority:
        tasks = [t for t in tasks if t["priority"] == priority]
    
    if not tasks:
        click.echo("No tasks found.")
        return
    
    for task in tasks:
        status = "✓" if task["done"] else "○"
        click.echo(f"[{task['id']}] {status} {task['title']} ({task['priority']})")


@cli.command()
@click.argument("task_id", type=int)
@click.pass_context
def done(ctx, task_id):
    """Mark task as done."""
    click.echo(f"✅ Task {task_id} marked as done!")


@cli.command()
@click.argument("task_id", type=int)
@click.option("--force", is_flag=True, help="Skip confirmation")
@click.pass_context
def delete(ctx, task_id, force):
    """Delete a task."""
    if not force:
        click.confirm(f"Delete task {task_id}?", abort=True)
    click.echo(f"🗑️  Task {task_id} deleted.")


# Nested group
@cli.group()
def config():
    """Manage configuration."""
    pass


@config.command("set")
@click.argument("key")
@click.argument("value")
def config_set(key, value):
    """Set a config value."""
    click.echo(f"Set {key} = {value}")


@config.command("get")
@click.argument("key")
def config_get(key):
    """Get a config value."""
    click.echo(f"{key} = (value)")


if __name__ == "__main__":
    cli()

# รันด้วย:
# python click_groups.py add "Buy milk" --priority high
# python click_groups.py list
# python click_groups.py done 1
# python click_groups.py config set theme dark
# python click_groups.py --verbose list
```

### ตัวอย่างที่ 11: Click Prompts

```python
# click_prompts.py
import click

@click.command()
def setup():
    """Interactive setup wizard."""
    click.echo("=== App Setup Wizard ===\n")
    
    # Simple prompt
    name = click.prompt("Your name")
    
    # Prompt with default
    host = click.prompt("Database host", default="localhost")
    port = click.prompt("Database port", default=5432, type=int)
    
    # Password prompt (hidden input)
    password = click.prompt("Database password", hide_input=True)
    
    # Confirmation prompt
    password2 = click.prompt("Confirm password", hide_input=True, confirmation_prompt=True)
    
    # Yes/No prompt
    use_ssl = click.confirm("Use SSL?", default=True)
    
    # Choice prompt
    env = click.prompt(
        "Environment",
        type=click.Choice(["development", "staging", "production"]),
        default="development"
    )
    
    click.echo("\n=== Configuration ===")
    click.echo(f"Name: {name}")
    click.echo(f"Database: {host}:{port}")
    click.echo(f"SSL: {use_ssl}")
    click.echo(f"Environment: {env}")
    
    if click.confirm("\nSave configuration?", default=True):
        click.echo("✅ Configuration saved!")
    else:
        click.echo("❌ Cancelled.")


@click.command()
@click.option("--name", prompt="Your name", help="Name to greet")
@click.option("--email", prompt="Your email", help="Email address")
def register(name, email):
    """Register a new user (prompts if not provided)."""
    click.echo(f"Registered: {name} <{email}>")


if __name__ == "__main__":
    setup()
```

### ตัวอย่างที่ 12: Click Progress Bars

```python
# click_progress.py
import click
import time
import random

@click.command()
@click.argument("count", type=int, default=20)
def process_items(count):
    """Process items with progress bar."""
    
    # Basic progress bar
    click.echo("Processing items...")
    with click.progressbar(
        range(count),
        label="Processing",
        length=count,
        fill_char="█",
        empty_char="░",
        show_eta=True,
        show_percent=True,
    ) as progress:
        for item in progress:
            # Simulate work
            time.sleep(0.1)
    
    click.echo("Done!")


@click.command()
def download_files():
    """Simulate downloading files with progress."""
    files = [
        ("data.csv", 1024 * 1024 * 5),    # 5MB
        ("model.pkl", 1024 * 1024 * 50),   # 50MB
        ("images.zip", 1024 * 1024 * 200), # 200MB
    ]
    
    for filename, filesize in files:
        click.echo(f"Downloading {filename}...")
        
        downloaded = 0
        chunk_size = 1024 * 1024  # 1MB chunks
        
        with click.progressbar(
            length=filesize,
            label=f"  {filename}",
            show_pos=True,
        ) as bar:
            while downloaded < filesize:
                chunk = min(chunk_size, filesize - downloaded)
                time.sleep(0.05)  # Simulate network
                downloaded += chunk
                bar.update(chunk)
        
        click.echo(f"  ✅ {filename} downloaded!")


if __name__ == "__main__":
    process_items()
```

### ตัวอย่างที่ 13: Click Colors และ Styling

```python
# click_styling.py
import click
import sys

def print_header(text: str):
    """แสดง header ที่สวยงาม"""
    width = 50
    click.echo(click.style("=" * width, fg="cyan"))
    click.echo(click.style(text.center(width), fg="cyan", bold=True))
    click.echo(click.style("=" * width, fg="cyan"))


def print_success(msg: str):
    click.echo(click.style(f"✅ {msg}", fg="green"))


def print_error(msg: str):
    click.echo(click.style(f"❌ {msg}", fg="red", bold=True), err=True)


def print_warning(msg: str):
    click.echo(click.style(f"⚠️  {msg}", fg="yellow"))


def print_info(msg: str):
    click.echo(click.style(f"ℹ️  {msg}", fg="blue"))


@click.command()
@click.option("--color/--no-color", default=True, help="Use colors")
def demo(color):
    """Demo of Click styling."""
    # เปิด/ปิด colors
    if not color:
        # Override click.style to return plain text
        click.style = lambda text, **kwargs: text
    
    print_header("CLI Styling Demo")
    print_success("Operation completed successfully")
    print_error("Something went wrong")
    print_warning("This is a warning")
    print_info("Information message")
    
    # Colored text inline
    click.echo(
        "Status: " +
        click.style("RUNNING", fg="green") +
        " | Workers: " +
        click.style("4", fg="yellow", bold=True) +
        " | Memory: " +
        click.style("256MB", fg="blue")
    )
    
    # Different styles
    styles = [
        ("bold", {"bold": True}),
        ("dim", {"dim": True}),
        ("underline", {"underline": True}),
        ("blink", {"blink": True}),
        ("reverse", {"reverse": True}),
    ]
    
    for style_name, style_kwargs in styles:
        click.echo(f"  {style_name}: " + click.style(f"Sample Text", **style_kwargs))


if __name__ == "__main__":
    demo()
```

### ตัวอย่างที่ 14: Click Testing

```python
# test_cli.py - Testing click commands
import click
from click.testing import CliRunner

@click.command()
@click.argument("name")
@click.option("--greeting", default="Hello")
def greet(name, greeting):
    click.echo(f"{greeting}, {name}!")

# Tests
from click.testing import CliRunner

def test_greet_basic():
    runner = CliRunner()
    result = runner.invoke(greet, ["Alice"])
    assert result.exit_code == 0
    assert "Hello, Alice!" in result.output

def test_greet_with_option():
    runner = CliRunner()
    result = runner.invoke(greet, ["Bob", "--greeting", "Hi"])
    assert result.exit_code == 0
    assert "Hi, Bob!" in result.output

def test_greet_missing_arg():
    runner = CliRunner()
    result = runner.invoke(greet, [])
    assert result.exit_code != 0  # Should fail

def test_with_env_and_files():
    runner = CliRunner()
    # Test with environment variables
    with runner.isolated_filesystem():
        # Creates temp directory for file operations
        with open("test.txt", "w") as f:
            f.write("test content")
        result = runner.invoke(greet, ["Alice"])
        assert result.exit_code == 0


# รัน tests
if __name__ == "__main__":
    test_greet_basic()
    test_greet_with_option()
    test_greet_missing_arg()
    print("All tests passed!")
```

---

## 5. Rich Library สำหรับ Beautiful CLI

**Rich** เป็น library สำหรับทำ beautiful terminal output ด้วย colors, tables, progress bars และ more

### การติดตั้ง

```bash
pip install rich
```

### ตัวอย่างที่ 15: Rich พื้นฐาน

```python
# rich_basics.py
from rich.console import Console
from rich.text import Text
from rich.panel import Panel
from rich.table import Table
from rich import print as rprint

console = Console()

# Markdown-like markup
console.print("[bold]Bold text[/bold]")
console.print("[italic]Italic text[/italic]")
console.print("[underline]Underlined text[/underline]")
console.print("[red]Red text[/red]")
console.print("[green]Green text[/green]")
console.print("[blue on white]Blue on white background[/blue on white]")
console.print("[bold red]Bold red text[/bold red]")

# Panel (boxed content)
console.print(Panel("Hello, World!", title="Welcome", subtitle="Rich Demo"))

# Highlight code
from rich.syntax import Syntax
code = '''
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
'''
syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)
```

### ตัวอย่างที่ 16: Rich Tables

```python
# rich_tables.py
from rich.console import Console
from rich.table import Table
from rich import box

console = Console()

# Basic table
def show_tasks_table(tasks: list):
    """แสดง tasks เป็น table สวยๆ"""
    table = Table(
        title="Task List",
        box=box.ROUNDED,
        show_header=True,
        header_style="bold cyan",
    )
    
    # Add columns
    table.add_column("ID", style="dim", width=4)
    table.add_column("Title", style="white", min_width=20)
    table.add_column("Priority", justify="center", width=10)
    table.add_column("Status", justify="center", width=12)
    table.add_column("Due Date", justify="right", width=12)
    
    # Priority color mapping
    priority_colors = {
        "high": "[bold red]",
        "medium": "[yellow]",
        "low": "[green]",
    }
    
    status_icons = {
        "todo": "○",
        "in_progress": "⟳",
        "done": "✓",
    }
    
    # Add rows
    for task in tasks:
        priority = task["priority"]
        status = task["status"]
        
        priority_styled = f"{priority_colors[priority]}{priority.upper()}[/]"
        
        status_icon = status_icons.get(status, "?")
        status_styled = f"{status_icon} {status.replace('_', ' ').title()}"
        
        # Row style based on status
        row_style = "dim" if status == "done" else ""
        
        table.add_row(
            str(task["id"]),
            task["title"],
            priority_styled,
            status_styled,
            task.get("due_date", "-"),
            style=row_style,
        )
    
    console.print(table)


# Mock data
tasks = [
    {"id": 1, "title": "Implement login feature", "priority": "high",
     "status": "in_progress", "due_date": "2024-01-15"},
    {"id": 2, "title": "Write unit tests", "priority": "medium",
     "status": "todo", "due_date": "2024-01-20"},
    {"id": 3, "title": "Update documentation", "priority": "low",
     "status": "todo", "due_date": "2024-01-25"},
    {"id": 4, "title": "Deploy to staging", "priority": "high",
     "status": "done", "due_date": "2024-01-10"},
]

show_tasks_table(tasks)
```

### ตัวอย่างที่ 17: Rich Progress

```python
# rich_progress.py
import time
from rich.console import Console
from rich.progress import (
    Progress, SpinnerColumn, BarColumn, TextColumn,
    TimeElapsedColumn, TimeRemainingColumn, MofNCompleteColumn
)

console = Console()

# Basic progress bar
def basic_progress():
    with Progress() as progress:
        task1 = progress.add_task("[red]Downloading...", total=100)
        task2 = progress.add_task("[green]Processing...", total=50)
        task3 = progress.add_task("[cyan]Analyzing...", total=200)
        
        while not progress.finished:
            progress.update(task1, advance=1)
            progress.update(task2, advance=0.5)
            progress.update(task3, advance=2)
            time.sleep(0.05)


# Custom progress bar
def custom_progress(items: list):
    with Progress(
        SpinnerColumn(),
        "[progress.description]{task.description}",
        BarColumn(),
        MofNCompleteColumn(),
        "[progress.percentage]{task.percentage:>3.0f}%",
        TimeElapsedColumn(),
        TimeRemainingColumn(),
        console=console,
    ) as progress:
        task = progress.add_task("Processing files", total=len(items))
        
        results = []
        for item in items:
            # Simulate work
            time.sleep(0.2)
            results.append(f"processed_{item}")
            progress.update(task, advance=1, description=f"Processing: {item}")
        
        return results


# Track iterable
from rich.progress import track

def process_with_track():
    items = list(range(20))
    results = []
    
    for item in track(items, description="Processing..."):
        time.sleep(0.1)
        results.append(item * 2)
    
    return results


console.print("\n[bold]Basic Progress:[/bold]")
basic_progress()

console.print("\n[bold]Custom Progress:[/bold]")
files = [f"file_{i}.txt" for i in range(10)]
results = custom_progress(files)
console.print(f"Processed {len(results)} files")
```

### ตัวอย่างที่ 18: Rich Logging

```python
# rich_logging.py
import logging
from rich.logging import RichHandler
from rich.console import Console

# Setup rich logging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(message)s",
    datefmt="[%X]",
    handlers=[
        RichHandler(
            rich_tracebacks=True,
            show_path=True,
            markup=True,
        )
    ]
)

log = logging.getLogger("myapp")

# ใช้งาน
log.debug("Debug message")
log.info("Application started")
log.warning("[yellow]This is a warning[/yellow]")
log.error("Something went wrong!")

# Exception with rich traceback
try:
    result = 1 / 0
except ZeroDivisionError:
    log.exception("Caught an exception!")
```

---

## 6. Typer - Modern CLI Framework

**Typer** สร้างบน Click และใช้ Python type hints เพื่อสร้าง CLI โดยอัตโนมัติ

### การติดตั้ง

```bash
pip install typer[all]  # รวม rich สำหรับ pretty output
```

### ตัวอย่างที่ 19: Typer พื้นฐาน

```python
# typer_basic.py
import typer
from typing import Optional
from enum import Enum

app = typer.Typer(help="Modern CLI with Typer")


class Environment(str, Enum):
    development = "development"
    staging = "staging"
    production = "production"


@app.command()
def serve(
    host: str = typer.Option("0.0.0.0", help="Host to bind"),
    port: int = typer.Option(8000, help="Port to listen on"),
    workers: int = typer.Option(1, min=1, max=32, help="Number of workers"),
    env: Environment = typer.Option(Environment.development, help="Environment"),
    debug: bool = typer.Option(False, "--debug/--no-debug", help="Enable debug mode"),
    config: Optional[str] = typer.Option(None, help="Config file path"),
):
    """Start the application server."""
    typer.echo(f"Starting server: {host}:{port}")
    typer.echo(f"Environment: {env.value}")
    typer.echo(f"Workers: {workers}")
    if debug:
        typer.echo(typer.style("Debug mode enabled", fg=typer.colors.YELLOW))


@app.command()
def migrate(
    database: str = typer.Argument(..., help="Database to migrate"),
    dry_run: bool = typer.Option(False, "--dry-run", help="Preview without applying"),
    verbose: bool = typer.Option(False, "--verbose", "-v"),
):
    """Run database migrations."""
    if dry_run:
        typer.echo(f"[DRY RUN] Would migrate database: {database}")
        return
    
    typer.echo(f"Migrating database: {database}")
    
    with typer.progressbar(range(10), label="Applying migrations") as progress:
        import time
        for _ in progress:
            time.sleep(0.1)
    
    typer.echo(typer.style("✅ Migration complete!", fg=typer.colors.GREEN))


if __name__ == "__main__":
    app()

# รันด้วย:
# python typer_basic.py serve --port 9000 --env production
# python typer_basic.py migrate mydb --dry-run
# python typer_basic.py --help
```

### ตัวอย่างที่ 20: Typer with Callbacks

```python
# typer_callbacks.py
import typer
from typing import Optional

def version_callback(value: bool):
    """Callback สำหรับ --version"""
    if value:
        typer.echo("App v1.0.0")
        raise typer.Exit()


def validate_port(value: int) -> int:
    """Validate port number"""
    if not 1 <= value <= 65535:
        raise typer.BadParameter(f"Port must be between 1 and 65535, got {value}")
    return value


app = typer.Typer()


@app.callback()
def main(
    ctx: typer.Context,
    version: Optional[bool] = typer.Option(
        None, "--version", "-v",
        callback=version_callback,
        is_eager=True,  # process before other options
        help="Show version"
    ),
    verbose: bool = typer.Option(False, "--verbose", help="Verbose output"),
):
    """My CLI Application"""
    ctx.ensure_object(dict)
    ctx.obj["verbose"] = verbose


@app.command()
def run(
    ctx: typer.Context,
    port: int = typer.Option(8000, callback=validate_port, help="Server port"),
):
    """Run the application."""
    verbose = ctx.obj.get("verbose", False)
    if verbose:
        typer.echo(f"Starting on port {port}...")
    typer.echo(f"Running on http://localhost:{port}")


if __name__ == "__main__":
    app()
```

---

## 7. โปรแกรมตัวอย่างจริง

### ตัวอย่างที่ 21: File Organizer CLI

```python
#!/usr/bin/env python3
"""
file_organizer.py - CLI tool สำหรับจัดระเบียบไฟล์
"""

import click
import shutil
from pathlib import Path
from collections import defaultdict
from typing import Dict, List

# File type mappings
FILE_CATEGORIES: Dict[str, List[str]] = {
    "Images": [".jpg", ".jpeg", ".png", ".gif", ".bmp", ".svg", ".webp", ".ico"],
    "Videos": [".mp4", ".avi", ".mkv", ".mov", ".wmv", ".flv", ".webm"],
    "Audio": [".mp3", ".wav", ".flac", ".aac", ".ogg", ".m4a"],
    "Documents": [".pdf", ".doc", ".docx", ".txt", ".rtf", ".odt", ".md"],
    "Spreadsheets": [".xls", ".xlsx", ".csv", ".ods"],
    "Code": [".py", ".js", ".ts", ".html", ".css", ".java", ".cpp", ".c", ".go", ".rs"],
    "Archives": [".zip", ".tar", ".gz", ".rar", ".7z", ".bz2"],
    "Data": [".json", ".xml", ".yaml", ".yml", ".toml", ".sql", ".db"],
}


def get_category(extension: str) -> str:
    """หา category ของ extension"""
    ext = extension.lower()
    for category, extensions in FILE_CATEGORIES.items():
        if ext in extensions:
            return category
    return "Others"


@click.group()
@click.version_option(version="1.0.0")
def cli():
    """
    File Organizer - Organize files by type
    
    Automatically organizes files in a directory into categorized subdirectories.
    """
    pass


@cli.command()
@click.argument("source", type=click.Path(exists=True, file_okay=False))
@click.option("--dest", "-d", type=click.Path(), help="Destination directory (default: source)")
@click.option("--dry-run", is_flag=True, help="Preview without moving files")
@click.option("--recursive", "-r", is_flag=True, help="Process subdirectories")
@click.option("--verbose", "-v", is_flag=True, help="Show detailed output")
def organize(source, dest, dry_run, recursive, verbose):
    """
    Organize files in SOURCE directory by type.
    
    SOURCE: Directory to organize
    """
    source_path = Path(source)
    dest_path = Path(dest) if dest else source_path
    
    # Count files by category
    stats: Dict[str, int] = defaultdict(int)
    moves: List[tuple] = []
    
    # Collect files
    pattern = "**/*" if recursive else "*"
    files = [f for f in source_path.glob(pattern) if f.is_file()]
    
    click.echo(f"Found {len(files)} files to organize in '{source_path}'")
    
    if dry_run:
        click.echo(click.style("DRY RUN - No files will be moved", fg="yellow", bold=True))
    
    # Plan moves
    with click.progressbar(files, label="Analyzing files") as file_list:
        for file_path in file_list:
            category = get_category(file_path.suffix)
            category_dir = dest_path / category
            new_path = category_dir / file_path.name
            
            # Handle duplicates
            counter = 1
            while new_path.exists():
                stem = file_path.stem
                suffix = file_path.suffix
                new_path = category_dir / f"{stem}_{counter}{suffix}"
                counter += 1
            
            moves.append((file_path, new_path, category))
            stats[category] += 1
    
    # Execute moves
    moved = 0
    errors = 0
    
    for src, dst, category in moves:
        try:
            if verbose:
                click.echo(f"  {src.name} → {category}/{dst.name}")
            
            if not dry_run:
                dst.parent.mkdir(parents=True, exist_ok=True)
                shutil.move(str(src), str(dst))
            moved += 1
        except Exception as e:
            click.echo(click.style(f"Error moving {src.name}: {e}", fg="red"), err=True)
            errors += 1
    
    # Summary
    click.echo("\n" + "=" * 40)
    click.echo(click.style("Organization Summary:", bold=True))
    
    for category, count in sorted(stats.items(), key=lambda x: x[1], reverse=True):
        if count > 0:
            bar = "█" * min(count, 20)
            click.echo(f"  {category:<15} {count:>4} files  {bar}")
    
    click.echo(f"\nTotal: {moved} files processed, {errors} errors")
    
    if dry_run:
        click.echo(click.style("\nRun without --dry-run to actually move files", fg="yellow"))


@cli.command()
@click.argument("directory", type=click.Path(exists=True))
def stats(directory):
    """Show file statistics for DIRECTORY."""
    dir_path = Path(directory)
    
    category_stats: Dict[str, Dict] = defaultdict(lambda: {"count": 0, "size": 0})
    
    for file_path in dir_path.rglob("*"):
        if file_path.is_file():
            category = get_category(file_path.suffix)
            category_stats[category]["count"] += 1
            category_stats[category]["size"] += file_path.stat().st_size
    
    click.echo(f"\nFile Statistics for: {directory}")
    click.echo("=" * 55)
    click.echo(f"{'Category':<15} {'Files':>8} {'Size':>12}")
    click.echo("-" * 55)
    
    total_files = 0
    total_size = 0
    
    for category, info in sorted(category_stats.items()):
        size_mb = info["size"] / (1024 * 1024)
        click.echo(f"{category:<15} {info['count']:>8,} {size_mb:>10.1f} MB")
        total_files += info["count"]
        total_size += info["size"]
    
    click.echo("-" * 55)
    total_mb = total_size / (1024 * 1024)
    click.echo(f"{'TOTAL':<15} {total_files:>8,} {total_mb:>10.1f} MB")


if __name__ == "__main__":
    cli()

# รันด้วย:
# python file_organizer.py organize ~/Downloads --dry-run
# python file_organizer.py organize ~/Downloads --dest ~/Organized -v
# python file_organizer.py stats ~/Downloads
```

### ตัวอย่างที่ 22: Todo CLI

```python
#!/usr/bin/env python3
"""
todo_cli.py - Simple todo manager CLI
"""

import json
import click
from pathlib import Path
from datetime import datetime
from typing import List, Optional

TODO_FILE = Path.home() / ".todo_cli.json"


def load_todos() -> List[dict]:
    """โหลด todos จากไฟล์"""
    if TODO_FILE.exists():
        with open(TODO_FILE) as f:
            return json.load(f)
    return []


def save_todos(todos: List[dict]):
    """บันทึก todos ลงไฟล์"""
    with open(TODO_FILE, "w") as f:
        json.dump(todos, f, indent=2, default=str)


def get_next_id(todos: List[dict]) -> int:
    """หา ID ต่อไป"""
    if not todos:
        return 1
    return max(t["id"] for t in todos) + 1


@click.group()
def cli():
    """📋 Todo Manager - Manage your tasks from the command line"""
    pass


@cli.command("add")
@click.argument("title")
@click.option("--priority", "-p", 
              type=click.Choice(["low", "medium", "high"]),
              default="medium")
@click.option("--due", "-d", help="Due date (YYYY-MM-DD)")
@click.option("--tag", "-t", multiple=True, help="Tags")
def add_task(title, priority, due, tag):
    """Add a new task."""
    todos = load_todos()
    
    task = {
        "id": get_next_id(todos),
        "title": title,
        "priority": priority,
        "status": "todo",
        "tags": list(tag),
        "due_date": due,
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat(),
    }
    
    todos.append(task)
    save_todos(todos)
    
    click.echo(f"✅ Added task #{task['id']}: {title!r}")
    if due:
        click.echo(f"   Due: {due}")
    if tag:
        click.echo(f"   Tags: {', '.join(tag)}")


@cli.command("list")
@click.option("--status", "-s",
              type=click.Choice(["all", "todo", "in_progress", "done"]),
              default="todo")
@click.option("--priority", "-p",
              type=click.Choice(["low", "medium", "high"]),
              help="Filter by priority")
@click.option("--tag", "-t", help="Filter by tag")
def list_tasks(status, priority, tag):
    """List tasks."""
    todos = load_todos()
    
    # Filter
    filtered = todos
    if status != "all":
        filtered = [t for t in filtered if t["status"] == status]
    if priority:
        filtered = [t for t in filtered if t["priority"] == priority]
    if tag:
        filtered = [t for t in filtered if tag in t.get("tags", [])]
    
    if not filtered:
        click.echo("No tasks found.")
        return
    
    # Priority colors
    priority_styles = {
        "high": ("red", "HIGH"),
        "medium": ("yellow", "MED"),
        "low": ("green", "LOW"),
    }
    
    status_icons = {
        "todo": "○",
        "in_progress": "⟳",
        "done": "✓",
    }
    
    click.echo(f"\n{'ID':>4}  {'Status':>3}  {'Pri':>4}  {'Title':<30}  {'Due'}")
    click.echo("-" * 65)
    
    for task in filtered:
        pri = task["priority"]
        color, pri_label = priority_styles.get(pri, ("white", "?"))
        icon = status_icons.get(task["status"], "?")
        due = task.get("due_date", "-")
        
        title_display = task["title"][:30]
        pri_styled = click.style(f"{pri_label:>4}", fg=color)
        
        if task["status"] == "done":
            title_display = click.style(title_display, dim=True)
        
        click.echo(
            f"{task['id']:>4}  {icon:>3}   {pri_styled}  {title_display:<30}  {due}"
        )
    
    click.echo(f"\nShowing {len(filtered)} task(s)")


@cli.command("done")
@click.argument("task_id", type=int)
def mark_done(task_id):
    """Mark a task as done."""
    todos = load_todos()
    
    for task in todos:
        if task["id"] == task_id:
            task["status"] = "done"
            task["updated_at"] = datetime.now().isoformat()
            save_todos(todos)
            click.echo(f"✅ Task #{task_id} marked as done!")
            return
    
    click.echo(f"❌ Task #{task_id} not found", err=True)


@cli.command("delete")
@click.argument("task_id", type=int)
@click.option("--force", "-f", is_flag=True, help="Skip confirmation")
def delete_task(task_id, force):
    """Delete a task."""
    todos = load_todos()
    
    task = next((t for t in todos if t["id"] == task_id), None)
    if not task:
        click.echo(f"❌ Task #{task_id} not found", err=True)
        return
    
    if not force:
        click.echo(f"Task: {task['title']!r}")
        click.confirm("Delete this task?", abort=True)
    
    todos = [t for t in todos if t["id"] != task_id]
    save_todos(todos)
    click.echo(f"🗑️  Task #{task_id} deleted.")


if __name__ == "__main__":
    cli()
```

### ตัวอย่างที่ 23: File Converter CLI

```python
#!/usr/bin/env python3
"""
converter.py - File format converter CLI
"""

import json
import csv
import click
from pathlib import Path


def read_json(filepath: Path) -> list:
    with open(filepath) as f:
        data = json.load(f)
    if isinstance(data, list):
        return data
    return [data]


def read_csv(filepath: Path) -> list:
    with open(filepath, newline="") as f:
        reader = csv.DictReader(f)
        return list(reader)


def write_json(data: list, filepath: Path, indent: int = 2):
    with open(filepath, "w") as f:
        json.dump(data, f, indent=indent, ensure_ascii=False)


def write_csv(data: list, filepath: Path):
    if not data:
        return
    fieldnames = list(data[0].keys())
    with open(filepath, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)


@click.command()
@click.argument("input_file", type=click.Path(exists=True))
@click.argument("output_file", type=click.Path())
@click.option("--from-format", "-f",
              type=click.Choice(["json", "csv"]),
              help="Input format (auto-detected if not specified)")
@click.option("--to-format", "-t",
              type=click.Choice(["json", "csv"]),
              help="Output format (auto-detected if not specified)")
@click.option("--indent", default=2, type=int, help="JSON indent level")
@click.option("--verbose", "-v", is_flag=True)
def convert(input_file, output_file, from_format, to_format, indent, verbose):
    """
    Convert files between formats.
    
    Supports: JSON ↔ CSV
    """
    input_path = Path(input_file)
    output_path = Path(output_file)
    
    # Auto-detect formats
    src_format = from_format or input_path.suffix.lstrip(".")
    dst_format = to_format or output_path.suffix.lstrip(".")
    
    if verbose:
        click.echo(f"Input:  {input_path} ({src_format})")
        click.echo(f"Output: {output_path} ({dst_format})")
    
    # Read input
    readers = {"json": read_json, "csv": read_csv}
    reader = readers.get(src_format)
    if not reader:
        click.echo(f"❌ Unsupported input format: {src_format}", err=True)
        raise click.Abort()
    
    data = reader(input_path)
    if verbose:
        click.echo(f"Read {len(data)} records")
    
    # Write output
    writers = {"json": lambda d, p: write_json(d, p, indent), "csv": write_csv}
    writer = writers.get(dst_format)
    if not writer:
        click.echo(f"❌ Unsupported output format: {dst_format}", err=True)
        raise click.Abort()
    
    output_path.parent.mkdir(parents=True, exist_ok=True)
    writer(data, output_path)
    
    click.echo(f"✅ Converted {len(data)} records: {input_path.name} → {output_path.name}")


if __name__ == "__main__":
    convert()

# รันด้วย:
# python converter.py data.json output.csv
# python converter.py data.csv output.json --indent 4
```

### ตัวอย่างที่ 24: Complete CLI App Structure

```python
"""
โครงสร้าง project สำหรับ CLI app ขนาดใหญ่:

myapp/
├── myapp/
│   ├── __init__.py
│   ├── cli/
│   │   ├── __init__.py    # exports main cli group
│   │   ├── main.py        # main entry point
│   │   ├── commands/
│   │   │   ├── __init__.py
│   │   │   ├── server.py  # server subcommands
│   │   │   ├── db.py      # database subcommands
│   │   │   └── admin.py   # admin subcommands
│   │   └── utils.py       # shared utilities
│   ├── core/
│   │   └── ...
│   └── config.py
├── setup.py / pyproject.toml
└── README.md
"""

# myapp/cli/main.py
import click

@click.group()
@click.version_option()
@click.pass_context
def cli(ctx):
    """MyApp - The awesome application"""
    ctx.ensure_object(dict)


# myapp/cli/commands/server.py
import click

@click.group()
def server():
    """Server management commands"""
    pass


@server.command("start")
@click.option("--port", default=8000, type=int)
@click.option("--workers", default=1, type=int)
def server_start(port, workers):
    """Start the application server"""
    click.echo(f"Starting server on port {port} with {workers} workers...")


@server.command("stop")
@click.option("--force", is_flag=True)
def server_stop(force):
    """Stop the application server"""
    if force:
        click.echo("Force stopping server...")
    else:
        click.echo("Gracefully stopping server...")


# myapp/cli/__init__.py
# from .main import cli
# from .commands.server import server
# cli.add_command(server)
```

### ตัวอย่างที่ 25: CLI with Config File

```python
# cli_with_config.py
import click
import json
from pathlib import Path

CONFIG_FILE = Path.home() / ".myapp_config.json"


def load_config() -> dict:
    if CONFIG_FILE.exists():
        with open(CONFIG_FILE) as f:
            return json.load(f)
    return {}


def save_config(config: dict):
    CONFIG_FILE.parent.mkdir(parents=True, exist_ok=True)
    with open(CONFIG_FILE, "w") as f:
        json.dump(config, f, indent=2)


# Decorator สำหรับ pass config
pass_config = click.make_pass_decorator(dict, ensure=True)


@click.group()
@click.pass_context
def cli(ctx):
    """App with persistent configuration"""
    ctx.obj = load_config()


@cli.command()
@click.argument("key")
@click.argument("value")
@pass_config
def set(config, key, value):
    """Set a configuration value."""
    config[key] = value
    save_config(config)
    click.echo(f"Set: {key} = {value!r}")


@cli.command("get")
@click.argument("key")
@pass_config
def get_value(config, key):
    """Get a configuration value."""
    value = config.get(key)
    if value is None:
        click.echo(f"Key '{key}' not found", err=True)
    else:
        click.echo(f"{key} = {value!r}")


@cli.command()
@pass_config
def show(config):
    """Show all configuration."""
    if not config:
        click.echo("No configuration set.")
        return
    
    for key, value in sorted(config.items()):
        click.echo(f"  {key} = {value!r}")


@cli.command()
@click.argument("key")
@pass_config
def unset(config, key):
    """Remove a configuration value."""
    if key in config:
        del config[key]
        save_config(config)
        click.echo(f"Removed: {key}")
    else:
        click.echo(f"Key '{key}' not found", err=True)


if __name__ == "__main__":
    cli()
```

### ตัวอย่างที่ 26: argparse vs Click Comparison

```python
"""
เปรียบเทียบ argparse และ Click
"""

# === argparse version ===
import argparse

def argparse_version():
    parser = argparse.ArgumentParser(description="Search files")
    parser.add_argument("pattern", help="Search pattern")
    parser.add_argument("path", help="Directory to search")
    parser.add_argument("--case-insensitive", "-i", action="store_true")
    parser.add_argument("--recursive", "-r", action="store_true")
    parser.add_argument("--count", "-c", action="store_true",
                        help="Show count only")
    
    args = parser.parse_args()
    print(f"Searching for '{args.pattern}' in {args.path}")
    # implementation...


# === Click version (cleaner, more readable) ===
import click

@click.command()
@click.argument("pattern")
@click.argument("path", type=click.Path(exists=True))
@click.option("--case-insensitive", "-i", is_flag=True)
@click.option("--recursive", "-r", is_flag=True)
@click.option("--count", "-c", is_flag=True, help="Show count only")
def click_version(pattern, path, case_insensitive, recursive, count):
    """Search files for PATTERN in PATH."""
    click.echo(f"Searching for '{pattern}' in {path}")
    # implementation...


"""
เปรียบเทียบ:
| Feature          | argparse          | Click             |
|------------------|-------------------|-------------------|
| Syntax           | Procedural        | Decorator-based   |
| Readability      | Medium            | High              |
| Testing          | Manual setup      | Built-in runner   |
| Nested commands  | Subparsers (verbose)| Groups (simple)  |
| Color output     | Manual            | Built-in          |
| Prompts          | Manual            | Built-in          |
| Progress bars    | Manual            | Built-in          |
| Standard library | ✅ Yes            | ❌ Third-party    |
"""
```

### ตัวอย่างที่ 27: Entry Points (pyproject.toml)

```toml
# pyproject.toml - สำหรับ install CLI tool
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "myapp"
version = "1.0.0"
dependencies = [
    "click>=8.0",
    "rich>=13.0",
]

[project.scripts]
# ติดตั้งแล้วใช้คำสั่ง 'myapp' ได้เลย
myapp = "myapp.cli.main:cli"
# เพิ่ม alias
mytool = "myapp.cli.main:cli"

# ติดตั้งด้วย: pip install -e .
# แล้วใช้: myapp --help
```

---

## 8. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Calculator CLI (argparse)

```python
# เฉลย
import argparse

def calculate(a, b, operation):
    ops = {
        "add": a + b,
        "sub": a - b,
        "mul": a * b,
        "div": a / b if b != 0 else None,
    }
    return ops.get(operation)

def main():
    parser = argparse.ArgumentParser(
        description="Simple calculator",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python calc.py 10 5 add    # 10 + 5 = 15
  python calc.py 10 5 mul    # 10 * 5 = 50
  python calc.py 10 0 div    # Division by zero error
        """
    )
    
    parser.add_argument("a", type=float, help="First number")
    parser.add_argument("b", type=float, help="Second number")
    parser.add_argument(
        "operation",
        choices=["add", "sub", "mul", "div"],
        help="Operation to perform"
    )
    parser.add_argument("--round", "-r", type=int, default=None,
                        help="Round result to N decimal places")
    
    args = parser.parse_args()
    
    if args.operation == "div" and args.b == 0:
        parser.error("Cannot divide by zero!")
    
    result = calculate(args.a, args.b, args.operation)
    
    if args.round is not None:
        result = round(result, args.round)
    
    symbols = {"add": "+", "sub": "-", "mul": "×", "div": "÷"}
    symbol = symbols[args.operation]
    
    print(f"{args.a} {symbol} {args.b} = {result}")

if __name__ == "__main__":
    main()
```

### แบบฝึกหัดที่ 2: Password Generator CLI

```python
# เฉลย
import click
import secrets
import string

@click.command()
@click.option("--length", "-l", default=16, type=int, help="Password length")
@click.option("--count", "-n", default=1, type=int, help="Number of passwords")
@click.option("--no-upper", is_flag=True, help="Exclude uppercase")
@click.option("--no-lower", is_flag=True, help="Exclude lowercase")
@click.option("--no-digits", is_flag=True, help="Exclude digits")
@click.option("--no-special", is_flag=True, help="Exclude special characters")
@click.option("--copy", "-c", is_flag=True, help="Copy first password to clipboard")
def generate(length, count, no_upper, no_lower, no_digits, no_special, copy):
    """Generate secure random passwords."""
    chars = ""
    if not no_upper:
        chars += string.ascii_uppercase
    if not no_lower:
        chars += string.ascii_lowercase
    if not no_digits:
        chars += string.digits
    if not no_special:
        chars += string.punctuation
    
    if not chars:
        click.echo("Error: Must include at least one character type!", err=True)
        raise click.Abort()
    
    passwords = []
    for i in range(count):
        password = "".join(secrets.choice(chars) for _ in range(length))
        passwords.append(password)
        click.echo(password)
    
    if copy and passwords:
        try:
            import pyperclip
            pyperclip.copy(passwords[0])
            click.echo(click.style("\n✅ First password copied to clipboard!", fg="green"))
        except ImportError:
            click.echo("\n⚠️  Install pyperclip to use --copy: pip install pyperclip", err=True)

if __name__ == "__main__":
    generate()
```

### แบบฝึกหัดที่ 3: Word Counter CLI

```python
# เฉลย
import click
from pathlib import Path
from collections import Counter

@click.command()
@click.argument("files", nargs=-1, required=True, type=click.Path(exists=True))
@click.option("--words", "-w", is_flag=True, help="Count words")
@click.option("--lines", "-l", is_flag=True, help="Count lines")
@click.option("--chars", "-c", is_flag=True, help="Count characters")
@click.option("--top", "-t", type=int, default=0, help="Show top N most frequent words")
def wc(files, words, lines, chars, top):
    """Count words, lines, and characters in FILES."""
    # Default: show all counts
    if not any([words, lines, chars]):
        words = lines = chars = True
    
    total_words = total_lines = total_chars = 0
    
    for filepath in files:
        content = Path(filepath).read_text(encoding="utf-8", errors="replace")
        file_words = content.split()
        file_lines = content.splitlines()
        
        w = len(file_words)
        l = len(file_lines)
        c = len(content)
        
        total_words += w
        total_lines += l
        total_chars += c
        
        parts = []
        if lines:
            parts.append(f"{l:>8} lines")
        if words:
            parts.append(f"{w:>8} words")
        if chars:
            parts.append(f"{c:>8} chars")
        
        click.echo(f"  {'  '.join(parts)}  {filepath}")
        
        if top > 0:
            word_counts = Counter(w.lower().strip(".,!?;:\"'") for w in file_words)
            click.echo(f"\n  Top {top} words:")
            for word, count in word_counts.most_common(top):
                click.echo(f"    {count:>5}  {word}")
    
    if len(files) > 1:
        parts = []
        if lines:
            parts.append(f"{total_lines:>8} lines")
        if words:
            parts.append(f"{total_words:>8} words")
        if chars:
            parts.append(f"{total_chars:>8} chars")
        click.echo(f"  {'  '.join(parts)}  TOTAL")

if __name__ == "__main__":
    wc()
```

### แบบฝึกหัดที่ 4-8: Additional Exercises

```python
"""
แบบฝึกหัดที่ 4: Network Tool CLI
สร้าง CLI ที่:
- ping hostname ที่กำหนด N ครั้ง
- แสดง response time สำหรับแต่ละ ping
- แสดง statistics (min/max/avg)
- มี --timeout option
"""

# แบบฝึกหัดที่ 5: Log Analyzer CLI
"""
สร้าง CLI ที่:
- อ่าน log file
- Filter ด้วย log level (DEBUG, INFO, WARNING, ERROR)
- Filter ด้วย date range
- แสดง summary statistics
- Export ผลลัพธ์เป็น CSV
"""

# แบบฝึกหัดที่ 6: Database Backup CLI  
"""
สร้าง CLI ที่:
- backup SQLite database ไปยัง directory ที่กำหนด
- ตั้งชื่อ backup ด้วย timestamp อัตโนมัติ
- Keep N backups ล่าสุด (ลบเก่า)
- แสดง progress bar
- Verify backup หลัง backup สำเร็จ
"""

# แบบฝึกหัดที่ 7: API Test CLI
"""
สร้าง CLI ที่:
- ส่ง HTTP requests ไปยัง API endpoints
- Support GET, POST, PUT, DELETE
- ส่ง headers และ JSON body
- แสดง response อย่างสวยงาม
- Save responses ลงไฟล์
"""

# แบบฝึกหัดที่ 8: Project Initializer CLI
"""
สร้าง CLI ที่:
- สร้าง Python project structure ใหม่
- Support templates (web, cli, library)
- สร้าง pyproject.toml, README.md, .gitignore
- Initialize git repository
- Install dependencies อัตโนมัติ
"""

# เฉลยแบบฝึกหัดที่ 4 (เพิ่มเติม)
import click
import subprocess
import statistics
import re

@click.command()
@click.argument("hostname")
@click.option("--count", "-n", default=4, type=int, help="Number of pings")
@click.option("--timeout", "-t", default=5, type=int, help="Timeout in seconds")
def ping(hostname, count, timeout):
    """Ping a hostname and show statistics."""
    click.echo(f"Pinging {hostname} {count} times...")
    
    times = []
    success = 0
    
    for i in range(count):
        try:
            result = subprocess.run(
                ["ping", "-c", "1", "-W", str(timeout), hostname],
                capture_output=True, text=True, timeout=timeout + 1
            )
            
            if result.returncode == 0:
                # Parse time from output
                match = re.search(r"time=(\d+\.?\d*)\s*ms", result.stdout)
                if match:
                    t = float(match.group(1))
                    times.append(t)
                    success += 1
                    click.echo(f"  Reply from {hostname}: time={t:.1f}ms")
            else:
                click.echo(f"  Request {i+1}: timeout")
        
        except subprocess.TimeoutExpired:
            click.echo(f"  Request {i+1}: timeout")
    
    # Statistics
    click.echo(f"\n--- {hostname} ping statistics ---")
    click.echo(f"Sent: {count}, Received: {success}, Lost: {count - success}")
    
    if times:
        click.echo(f"Round-trip min/avg/max: "
                   f"{min(times):.1f}/{statistics.mean(times):.1f}/{max(times):.1f} ms")

if __name__ == "__main__":
    ping()
```

---

## สรุป

| Library | เมื่อไรควรใช้ |
|---------|--------------|
| `sys.argv` | Script เล็กๆ ง่ายๆ |
| `argparse` | Standard lib, feature ครบ |
| `click` | Production CLI, decorator syntax |
| `typer` | Type hints, modern Python |
| `rich` | Beautiful output สวยงาม |

### Installation

```bash
pip install click rich typer[all]
# หรือ
pip install click
pip install rich
pip install "typer[all]"
```
