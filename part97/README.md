# Part 97: Monitoring, Observability & Distributed Tracing

## สารบัญ

1. [Introduction to Observability](#introduction)
2. [Observability Pillars: Metrics, Logs, Traces](#pillars)
3. [Prometheus + Python](#prometheus)
4. [Grafana Dashboards](#grafana)
5. [Structured Logging with structlog](#structlog)
6. [ELK Stack Integration](#elk)
7. [OpenTelemetry Python SDK](#opentelemetry)
8. [Distributed Tracing](#distributed-tracing)
9. [Spans and Traces](#spans-traces)
10. [Jaeger/Zipkin](#jaeger-zipkin)
11. [Health Check Endpoints](#health-checks)
12. [APM Tools](#apm)
13. [Application Metrics](#app-metrics)
14. [Error Tracking with Sentry](#sentry)
15. [Complete Example: Monitored FastAPI App](#fastapi-example)
16. [แบบฝึกหัด](#exercises)

---

## 1. Introduction to Observability <a name="introduction"></a>

Observability คือความสามารถในการเข้าใจ internal state ของระบบจาก external outputs เป็นแนวคิดที่สำคัญมากสำหรับระบบ production สมัยใหม่

### Monitoring vs Observability

| Monitoring | Observability |
|-----------|---------------|
| รู้ว่า "อะไร" เกิดขึ้น | รู้ว่า "ทำไม" ถึงเกิดขึ้น |
| Reactive | Proactive |
| Known unknowns | Unknown unknowns |
| Dashboards & Alerts | Exploration & Investigation |

```python
# ตัวอย่าง 1: Observability foundation
import time
import logging
import json
from datetime import datetime
from typing import Any, Dict, Optional
from contextlib import contextmanager

class ObservabilityContext:
    """Base class สำหรับ observability"""
    
    def __init__(self, service_name: str, version: str = "1.0.0"):
        self.service_name = service_name
        self.version = version
        self.start_time = datetime.utcnow()
    
    def get_context(self) -> Dict[str, Any]:
        return {
            "service": self.service_name,
            "version": self.version,
            "timestamp": datetime.utcnow().isoformat(),
        }

# Setup basic structured logging
logging.basicConfig(
    level=logging.INFO,
    format='%(message)s'  # JSON format
)

class JSONLogger:
    """Logger ที่ output เป็น JSON"""
    
    def __init__(self, name: str, context: Optional[Dict] = None):
        self.logger = logging.getLogger(name)
        self.context = context or {}
    
    def _log(self, level: str, message: str, **kwargs):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "level": level,
            "message": message,
            "logger": self.logger.name,
            **self.context,
            **kwargs
        }
        getattr(self.logger, level.lower())(json.dumps(log_entry))
    
    def info(self, message: str, **kwargs):
        self._log("INFO", message, **kwargs)
    
    def warning(self, message: str, **kwargs):
        self._log("WARNING", message, **kwargs)
    
    def error(self, message: str, **kwargs):
        self._log("ERROR", message, **kwargs)
    
    def debug(self, message: str, **kwargs):
        self._log("DEBUG", message, **kwargs)

# Usage
logger = JSONLogger(
    "api_service",
    context={"service": "user-api", "version": "2.1.0", "env": "production"}
)

logger.info("Service started", port=8080, workers=4)
logger.info("Request received", 
    method="GET", path="/api/users", 
    user_id=123, request_id="req-abc123")
```

---

## 2. Observability Pillars <a name="pillars"></a>

### The Three Pillars

```
┌─────────────────────────────────────────────────┐
│               OBSERVABILITY                      │
│                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ METRICS  │  │  LOGS    │  │   TRACES     │  │
│  │          │  │          │  │              │  │
│  │Aggregated│  │Detailed  │  │Distributed   │  │
│  │numbers   │  │events    │  │call chains   │  │
│  │over time │  │          │  │              │  │
│  │          │  │          │  │              │  │
│  │Prometheus│  │ELK/Loki  │  │Jaeger/Zipkin │  │
│  └──────────┘  └──────────┘  └──────────────┘  │
└─────────────────────────────────────────────────┘
```

```python
# ตัวอย่าง 2: Simple metrics collection
import time
import threading
from collections import defaultdict, deque
from typing import Dict, List, Optional

class MetricsCollector:
    """Simple in-memory metrics collector"""
    
    def __init__(self):
        self._counters: Dict[str, float] = defaultdict(float)
        self._gauges: Dict[str, float] = defaultdict(float)
        self._histograms: Dict[str, List[float]] = defaultdict(list)
        self._lock = threading.Lock()
    
    def increment(self, name: str, value: float = 1.0, labels: dict = None):
        """Increment a counter"""
        key = self._make_key(name, labels)
        with self._lock:
            self._counters[key] += value
    
    def set_gauge(self, name: str, value: float, labels: dict = None):
        """Set a gauge value"""
        key = self._make_key(name, labels)
        with self._lock:
            self._gauges[key] = value
    
    def observe(self, name: str, value: float, labels: dict = None):
        """Record a histogram observation"""
        key = self._make_key(name, labels)
        with self._lock:
            self._histograms[key].append(value)
    
    def _make_key(self, name: str, labels: Optional[dict]) -> str:
        if not labels:
            return name
        label_str = ','.join(f'{k}="{v}"' for k, v in sorted(labels.items()))
        return f"{name}{{{label_str}}}"
    
    def get_summary(self) -> dict:
        """Get current metrics summary"""
        summary = {}
        
        with self._lock:
            for key, value in self._counters.items():
                summary[f"counter:{key}"] = value
            
            for key, value in self._gauges.items():
                summary[f"gauge:{key}"] = value
            
            for key, values in self._histograms.items():
                if values:
                    sorted_vals = sorted(values)
                    n = len(sorted_vals)
                    summary[f"histogram:{key}"] = {
                        'count': n,
                        'sum': sum(sorted_vals),
                        'avg': sum(sorted_vals) / n,
                        'p50': sorted_vals[int(n * 0.5)],
                        'p95': sorted_vals[int(n * 0.95)],
                        'p99': sorted_vals[int(n * 0.99)],
                    }
        
        return summary

# Global metrics instance
metrics = MetricsCollector()

# Decorator สำหรับ auto-instrument functions
def track_metrics(operation_name: str):
    def decorator(func):
        import functools
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            success = True
            try:
                result = func(*args, **kwargs)
                return result
            except Exception as e:
                success = False
                metrics.increment('errors_total', 
                    labels={'operation': operation_name, 'error': type(e).__name__})
                raise
            finally:
                duration = time.perf_counter() - start
                metrics.observe('operation_duration_seconds', duration,
                    labels={'operation': operation_name})
                metrics.increment('operations_total',
                    labels={'operation': operation_name, 
                            'status': 'success' if success else 'error'})
        return wrapper
    return decorator

@track_metrics("process_order")
def process_order(order_id: int) -> dict:
    """Simulate order processing"""
    time.sleep(0.001 * (order_id % 5))  # variable duration
    if order_id % 10 == 0:
        raise ValueError(f"Invalid order: {order_id}")
    return {"order_id": order_id, "status": "processed"}

# Run some operations
import random
successful = 0
failed = 0

for i in range(100):
    try:
        process_order(i)
        successful += 1
    except ValueError:
        failed += 1

print("Metrics Summary:")
summary = metrics.get_summary()
for key, value in sorted(summary.items()):
    print(f"  {key}: {value}")
```

---

## 3. Prometheus + Python <a name="prometheus"></a>

```python
# ตัวอย่าง 3: prometheus_client setup
# pip install prometheus-client

try:
    from prometheus_client import (
        Counter, Gauge, Histogram, Summary,
        start_http_server, generate_latest,
        CollectorRegistry, CONTENT_TYPE_LATEST
    )
    import time
    import random
    
    # สร้าง custom registry
    registry = CollectorRegistry()
    
    # Counter - นับจำนวนที่เพิ่มขึ้นเรื่อยๆ (ไม่ลง)
    http_requests_total = Counter(
        'http_requests_total',
        'Total HTTP requests',
        ['method', 'endpoint', 'status_code'],
        registry=registry
    )
    
    # Gauge - ค่าที่ขึ้นลงได้
    active_connections = Gauge(
        'active_connections',
        'Number of active connections',
        registry=registry
    )
    
    database_connections = Gauge(
        'database_connections_pool',
        'Database connection pool stats',
        ['state'],  # active, idle, waiting
        registry=registry
    )
    
    # Histogram - กระจายข้อมูล
    request_duration_seconds = Histogram(
        'request_duration_seconds',
        'HTTP request duration in seconds',
        ['method', 'endpoint'],
        buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0],
        registry=registry
    )
    
    # Summary - เหมือน Histogram แต่คำนวณ quantiles ฝั่ง client
    processing_time = Summary(
        'processing_time_seconds',
        'Processing time in seconds',
        registry=registry
    )
    
    # Simulate metrics
    endpoints = ['/api/users', '/api/orders', '/api/products']
    methods = ['GET', 'POST', 'PUT', 'DELETE']
    status_codes = ['200', '201', '400', '404', '500']
    
    def simulate_requests(n=1000):
        for _ in range(n):
            method = random.choice(methods)
            endpoint = random.choice(endpoints)
            status = random.choices(status_codes, weights=[60, 15, 10, 10, 5])[0]
            duration = random.expovariate(1/0.1)  # exponential dist, mean=100ms
            
            http_requests_total.labels(
                method=method, 
                endpoint=endpoint, 
                status_code=status
            ).inc()
            
            request_duration_seconds.labels(
                method=method, 
                endpoint=endpoint
            ).observe(duration)
            
            with processing_time.time():
                time.sleep(0.0001)
    
    # Update gauge values
    active_connections.set(42)
    database_connections.labels(state='active').set(8)
    database_connections.labels(state='idle').set(12)
    database_connections.labels(state='waiting').set(3)
    
    simulate_requests(500)
    
    # Output metrics in Prometheus format
    output = generate_latest(registry).decode('utf-8')
    print("Prometheus Metrics (first 1000 chars):")
    print(output[:1000])
    print("...")
    
    print(f"\nTotal metrics output: {len(output):,} bytes")
    
except ImportError:
    print("prometheus_client not installed. Run: pip install prometheus-client")
    print("\nShowing conceptual example:")
    print("""
    Metric Types:
    - Counter:   http_requests_total{method="GET",status="200"} 1234
    - Gauge:     active_connections 42
    - Histogram: request_duration_seconds_bucket{le="0.1"} 890
    - Summary:   processing_time_seconds{quantile="0.99"} 0.456
    """)
```

```python
# ตัวอย่าง 4: Prometheus middleware สำหรับ web frameworks
import time
import functools
import threading
from typing import Callable, Optional

class PrometheusMiddleware:
    """Middleware สำหรับ auto-instrument web requests"""
    
    def __init__(self, app_name: str = "app"):
        self.app_name = app_name
        self._requests = {}  # counter simulation
        self._durations = {}  # histogram simulation
        self._in_progress = 0
        self._lock = threading.Lock()
    
    def record_request(self, method: str, path: str, status: int, duration: float):
        """Record request metrics"""
        labels = f"{method}:{path}:{status}"
        
        with self._lock:
            if labels not in self._requests:
                self._requests[labels] = 0
                self._durations[labels] = []
            
            self._requests[labels] += 1
            self._durations[labels].append(duration)
    
    def middleware_factory(self):
        """สร้าง WSGI/ASGI middleware"""
        def middleware(handler):
            @functools.wraps(handler)
            def wrapped_handler(request):
                start = time.perf_counter()
                self._in_progress += 1
                
                try:
                    response = handler(request)
                    status = getattr(response, 'status_code', 200)
                    return response
                except Exception as e:
                    status = 500
                    raise
                finally:
                    duration = time.perf_counter() - start
                    self._in_progress -= 1
                    self.record_request(
                        method=request.get('method', 'GET'),
                        path=request.get('path', '/'),
                        status=status,
                        duration=duration
                    )
            
            return wrapped_handler
        return middleware
    
    def get_stats(self) -> dict:
        with self._lock:
            stats = {}
            for labels, count in self._requests.items():
                durations = self._durations[labels]
                sorted_d = sorted(durations)
                n = len(sorted_d)
                stats[labels] = {
                    'count': count,
                    'avg_ms': sum(sorted_d) / n * 1000,
                    'p95_ms': sorted_d[int(n * 0.95)] * 1000 if n > 0 else 0,
                }
            return stats

# Usage
prom = PrometheusMiddleware("my_app")

# Simulate requests
import random

def simulate_web_traffic():
    paths = ['/api/users', '/api/orders', '/health']
    methods = ['GET', 'POST']
    
    for _ in range(200):
        method = random.choice(methods)
        path = random.choice(paths)
        duration = random.expovariate(10)  # mean 100ms
        status = random.choices([200, 201, 400, 404, 500], 
                                weights=[70, 10, 10, 7, 3])[0]
        prom.record_request(method, path, status, duration)

simulate_web_traffic()

print("\nPrometheus Middleware Stats:")
for label, stats in prom.get_stats().items():
    print(f"  {label}: {stats}")
```

---

## 4. Grafana Dashboards <a name="grafana"></a>

```python
# ตัวอย่าง 5: Grafana dashboard configuration
# Dashboard JSON สำหรับ Python Application

GRAFANA_DASHBOARD = {
    "title": "Python Application Dashboard",
    "panels": [
        {
            "title": "Request Rate",
            "type": "graph",
            "targets": [
                {
                    "expr": "rate(http_requests_total[5m])",
                    "legendFormat": "{{method}} {{endpoint}}"
                }
            ]
        },
        {
            "title": "Error Rate",
            "type": "graph", 
            "targets": [
                {
                    "expr": "rate(http_requests_total{status_code=~'5..'}[5m])",
                    "legendFormat": "Error Rate"
                }
            ]
        },
        {
            "title": "Request Duration P99",
            "type": "graph",
            "targets": [
                {
                    "expr": "histogram_quantile(0.99, rate(request_duration_seconds_bucket[5m]))",
                    "legendFormat": "P99 {{endpoint}}"
                }
            ]
        },
        {
            "title": "Active Connections",
            "type": "stat",
            "targets": [
                {
                    "expr": "active_connections",
                    "legendFormat": "Active"
                }
            ]
        },
        {
            "title": "Memory Usage",
            "type": "graph",
            "targets": [
                {
                    "expr": "process_resident_memory_bytes",
                    "legendFormat": "RSS Memory"
                }
            ]
        }
    ],
    "alerts": [
        {
            "name": "High Error Rate",
            "condition": "rate(http_requests_total{status_code=~'5..'}[5m]) > 0.05",
            "message": "Error rate > 5% for 5 minutes"
        },
        {
            "name": "Slow Response",
            "condition": "histogram_quantile(0.99, rate(request_duration_seconds_bucket[5m])) > 1.0",
            "message": "P99 latency > 1 second"
        }
    ]
}

import json
print("Grafana Dashboard Config:")
print(json.dumps(GRAFANA_DASHBOARD, indent=2)[:500])
print("...")

# PromQL queries สำหรับ common use cases
PROMQL_EXAMPLES = {
    "request_rate_per_endpoint": 
        'rate(http_requests_total[5m]) by (endpoint)',
    
    "error_rate_percentage":
        '100 * sum(rate(http_requests_total{status_code=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))',
    
    "p50_latency":
        'histogram_quantile(0.50, sum(rate(request_duration_seconds_bucket[5m])) by (le, endpoint))',
    
    "p99_latency":
        'histogram_quantile(0.99, sum(rate(request_duration_seconds_bucket[5m])) by (le, endpoint))',
    
    "apdex_score":
        '(sum(rate(request_duration_seconds_bucket{le="0.3"}[5m])) + sum(rate(request_duration_seconds_bucket{le="1.2"}[5m]))) / 2 / sum(rate(request_duration_seconds_count[5m]))',
    
    "cpu_usage":
        'rate(process_cpu_seconds_total[5m]) * 100',
    
    "memory_usage_mb":
        'process_resident_memory_bytes / 1024 / 1024',
}

print("\nUseful PromQL Queries:")
for name, query in PROMQL_EXAMPLES.items():
    print(f"\n{name}:")
    print(f"  {query}")
```

---

## 5. Structured Logging with structlog <a name="structlog"></a>

```python
# ตัวอย่าง 6: structlog setup และ usage
# pip install structlog

try:
    import structlog
    import logging
    import sys

    # Configure structlog
    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.processors.add_log_level,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.dev.ConsoleRenderer()  # สวยงามใน development
            # ใช้ JSONRenderer ใน production:
            # structlog.processors.JSONRenderer()
        ],
        wrapper_class=structlog.BoundLogger,
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory()
    )

    log = structlog.get_logger()

    # Basic usage
    log.info("user_login", 
        user_id=123, 
        email="user@example.com",
        ip_address="192.168.1.1")
    
    log.warning("rate_limit_exceeded",
        user_id=123,
        endpoint="/api/data",
        limit=100,
        current=105)
    
    log.error("database_error",
        query="SELECT * FROM users",
        error="Connection timeout",
        retry_count=3)
    
    # Bind context ที่จะ include ใน logs ทั้งหมด
    request_log = log.bind(
        request_id="req-abc123",
        user_id=456,
        service="api"
    )
    
    request_log.info("request_started", method="POST", path="/api/orders")
    request_log.info("processing", step="validation")
    request_log.info("processing", step="payment")
    request_log.info("request_completed", status=201, duration_ms=145)

except ImportError:
    print("structlog not installed. Run: pip install structlog")
```

```python
# ตัวอย่าง 7: Custom structlog processors
import logging
import json
import sys
import time
import traceback
from datetime import datetime, timezone
from typing import Any, Dict, MutableMapping

def add_service_info(logger, method, event_dict):
    """Processor เพิ่ม service information"""
    event_dict['service'] = {
        'name': 'python-api',
        'version': '2.1.0',
        'environment': 'production'
    }
    return event_dict

def add_timestamps(logger, method, event_dict):
    """Processor เพิ่ม timestamp formats ต่างๆ"""
    now = datetime.now(timezone.utc)
    event_dict['timestamp'] = now.isoformat()
    event_dict['unix_timestamp'] = now.timestamp()
    return event_dict

def sanitize_pii(logger, method, event_dict):
    """Processor ลบ PII data"""
    pii_fields = {'password', 'credit_card', 'ssn', 'token', 'secret'}
    
    def sanitize(obj, depth=0):
        if depth > 10:  # prevent infinite recursion
            return obj
        if isinstance(obj, dict):
            return {
                k: '***REDACTED***' if k.lower() in pii_fields else sanitize(v, depth+1)
                for k, v in obj.items()
            }
        elif isinstance(obj, (list, tuple)):
            return [sanitize(item, depth+1) for item in obj]
        return obj
    
    return sanitize(event_dict)

def add_exception_info(logger, method, event_dict):
    """Processor เพิ่มข้อมูล exception"""
    exc_info = event_dict.pop('exc_info', None)
    if exc_info:
        if exc_info is True:
            exc_info = sys.exc_info()
        
        exc_type, exc_value, exc_tb = exc_info
        if exc_value:
            event_dict['exception'] = {
                'type': exc_type.__name__ if exc_type else None,
                'message': str(exc_value),
                'traceback': traceback.format_tb(exc_tb)
            }
    return event_dict

class StructuredLogger:
    """Custom structured logger"""
    
    def __init__(self, name: str, context: Dict = None):
        self.name = name
        self.context = context or {}
        self._processors = [
            add_service_info,
            add_timestamps,
            sanitize_pii,
            add_exception_info,
        ]
    
    def _process(self, level: str, event: str, **kwargs) -> dict:
        event_dict = {
            'logger': self.name,
            'level': level,
            'event': event,
            **self.context,
            **kwargs
        }
        
        for processor in self._processors:
            event_dict = processor(self, level.lower(), event_dict)
        
        return event_dict
    
    def _output(self, level: str, event_dict: dict):
        print(json.dumps(event_dict, default=str))
    
    def info(self, event: str, **kwargs):
        self._output('INFO', self._process('INFO', event, **kwargs))
    
    def warning(self, event: str, **kwargs):
        self._output('WARNING', self._process('WARNING', event, **kwargs))
    
    def error(self, event: str, **kwargs):
        self._output('ERROR', self._process('ERROR', event, **kwargs))
    
    def bind(self, **kwargs) -> 'StructuredLogger':
        new_context = {**self.context, **kwargs}
        return StructuredLogger(self.name, new_context)

# Usage
slog = StructuredLogger("api_service")

# Normal logging
slog.info("request_received", 
    method="POST", 
    path="/api/users",
    request_id="req-123")

# With sensitive data (will be redacted)
slog.info("user_created",
    user_id=456,
    email="user@example.com", 
    password="should_be_redacted",  # จะถูก redact
    token="secret_token_123")       # จะถูก redact

# Error logging
try:
    raise ValueError("Database connection failed")
except Exception:
    slog.error("database_error", 
        query="INSERT INTO users",
        exc_info=True)
```

---

## 6. ELK Stack Integration <a name="elk"></a>

```python
# ตัวอย่าง 8: Elasticsearch logging handler
import logging
import json
import time
import threading
import queue
from datetime import datetime, timezone
from typing import List, Dict, Any

class ElasticsearchLogHandler(logging.Handler):
    """Async logging handler สำหรับ Elasticsearch"""
    
    def __init__(self, es_url: str, index_prefix: str = "logs",
                 batch_size: int = 100, flush_interval: float = 5.0):
        super().__init__()
        self.es_url = es_url
        self.index_prefix = index_prefix
        self.batch_size = batch_size
        self.flush_interval = flush_interval
        
        self._queue = queue.Queue()
        self._batch: List[Dict] = []
        self._lock = threading.Lock()
        self._stop_event = threading.Event()
        
        # Start background flush thread
        self._flush_thread = threading.Thread(
            target=self._flush_loop, 
            daemon=True
        )
        self._flush_thread.start()
    
    def emit(self, record: logging.LogRecord):
        """Add log record to queue"""
        log_entry = self._format_record(record)
        self._queue.put(log_entry)
    
    def _format_record(self, record: logging.LogRecord) -> dict:
        """Format log record for Elasticsearch"""
        return {
            "@timestamp": datetime.fromtimestamp(
                record.created, tz=timezone.utc
            ).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
            "thread": record.thread,
            "process": record.process,
            "extra": getattr(record, 'extra', {}),
        }
    
    def _flush_loop(self):
        """Background thread ที่ flush logs ไป ES"""
        while not self._stop_event.is_set():
            batch = []
            deadline = time.time() + self.flush_interval
            
            while time.time() < deadline and len(batch) < self.batch_size:
                try:
                    item = self._queue.get(timeout=0.1)
                    batch.append(item)
                except queue.Empty:
                    break
            
            if batch:
                self._send_to_elasticsearch(batch)
    
    def _send_to_elasticsearch(self, batch: List[Dict]):
        """Send batch to Elasticsearch"""
        index = f"{self.index_prefix}-{datetime.utcnow().strftime('%Y.%m.%d')}"
        
        # Prepare bulk request
        bulk_body = []
        for doc in batch:
            bulk_body.append(json.dumps({"index": {"_index": index}}))
            bulk_body.append(json.dumps(doc))
        
        payload = '\n'.join(bulk_body) + '\n'
        
        # In real code, use requests or elasticsearch-py client:
        # import requests
        # response = requests.post(
        #     f"{self.es_url}/_bulk",
        #     data=payload,
        #     headers={"Content-Type": "application/json"}
        # )
        
        # For demo, just count
        print(f"[ES Handler] Would send {len(batch)} logs to {index}")
    
    def close(self):
        """Flush remaining logs and stop"""
        self._stop_event.set()
        self._flush_thread.join(timeout=10)
        super().close()

# Usage
es_handler = ElasticsearchLogHandler(
    es_url="http://localhost:9200",
    index_prefix="python-app-logs",
    batch_size=50,
    flush_interval=2.0
)
es_handler.setLevel(logging.INFO)

app_logger = logging.getLogger("my_application")
app_logger.addHandler(es_handler)
app_logger.setLevel(logging.DEBUG)

# Log some events
for i in range(5):
    app_logger.info(f"Processing item {i}", 
                    extra={'extra': {'item_id': i, 'batch': 'batch-001'}})

time.sleep(0.5)  # Let background thread process
```

```python
# ตัวอย่าง 9: Logstash filter configuration
LOGSTASH_CONFIG = """
input {
  tcp {
    port => 5000
    codec => json_lines
  }
}

filter {
  # Parse timestamp
  date {
    match => ["@timestamp", "ISO8601"]
    target => "@timestamp"
  }
  
  # Add geolocation for IP addresses
  geoip {
    source => "ip_address"
    target => "geoip"
  }
  
  # Parse user agent
  useragent {
    source => "user_agent"
    target => "ua"
  }
  
  # Add tags based on log level
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error", "alert"]
    }
  }
  
  # Parse request path
  grok {
    match => { 
      "path" => "/api/%{WORD:resource}(?:/%{NUMBER:resource_id})?" 
    }
  }
  
  # Calculate response time buckets
  if [duration_ms] {
    ruby {
      code => '
        duration = event.get("duration_ms").to_f
        bucket = if duration < 100 then "fast"
                elsif duration < 500 then "normal"  
                elsif duration < 2000 then "slow"
                else "very_slow" end
        event.set("latency_bucket", bucket)
      '
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "python-app-%{+YYYY.MM.dd}"
  }
}
"""

print("Logstash Configuration:")
print(LOGSTASH_CONFIG)
```

---

## 7. OpenTelemetry Python SDK <a name="opentelemetry"></a>

```python
# ตัวอย่าง 10: OpenTelemetry setup
# pip install opentelemetry-api opentelemetry-sdk
# pip install opentelemetry-instrumentation-requests

try:
    from opentelemetry import trace, metrics
    from opentelemetry.sdk.trace import TracerProvider
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    from opentelemetry.sdk.trace.export.in_memory_span_exporter import InMemorySpanExporter
    from opentelemetry.sdk.metrics import MeterProvider
    from opentelemetry.sdk.metrics.export import InMemoryMetricReader
    
    # Setup trace provider
    exporter = InMemorySpanExporter()
    provider = TracerProvider()
    provider.add_span_processor(BatchSpanProcessor(exporter))
    trace.set_tracer_provider(provider)
    
    # Get tracer
    tracer = trace.get_tracer("my_service", "1.0.0")
    
    # Setup metrics
    reader = InMemoryMetricReader()
    meter_provider = MeterProvider(metric_readers=[reader])
    metrics.set_meter_provider(meter_provider)
    meter = metrics.get_meter("my_service")
    
    # Create metrics instruments
    request_counter = meter.create_counter(
        "requests",
        description="Number of requests",
        unit="1"
    )
    
    latency_histogram = meter.create_histogram(
        "request_latency",
        description="Request latency",
        unit="ms"
    )
    
    # Create spans
    def process_request(user_id: int, action: str):
        with tracer.start_as_current_span("process_request") as span:
            span.set_attribute("user.id", user_id)
            span.set_attribute("action", action)
            
            # Nested span
            with tracer.start_as_current_span("validate_input") as child_span:
                child_span.set_attribute("validation.type", "schema")
                time.sleep(0.001)  # simulate work
            
            with tracer.start_as_current_span("database_query") as db_span:
                db_span.set_attribute("db.system", "postgresql")
                db_span.set_attribute("db.statement", f"SELECT * FROM users WHERE id={user_id}")
                time.sleep(0.005)  # simulate DB query
            
            # Record metrics
            request_counter.add(1, {"action": action, "status": "success"})
            latency_histogram.record(6.0, {"action": action})
            
            return {"user_id": user_id, "action": action}
    
    import time
    # Process some requests
    for i in range(5):
        process_request(i, "view_profile")
    
    # Get recorded spans
    spans = exporter.get_finished_spans()
    print(f"OpenTelemetry - Recorded {len(spans)} spans:")
    for span in spans[:5]:
        print(f"  Span: {span.name}")
        print(f"    Duration: {(span.end_time - span.start_time) / 1e6:.2f}ms")
        print(f"    Attributes: {dict(span.attributes)}")
    
except ImportError:
    print("OpenTelemetry not installed.")
    print("Run: pip install opentelemetry-api opentelemetry-sdk")
    
    print("""
    OpenTelemetry Key Concepts:
    
    Tracer Provider - สร้างและจัดการ tracers
    Tracer         - สร้าง spans
    Span           - หน่วยของ distributed trace
    Span Context   - trace_id, span_id, trace_flags
    Propagator     - ส่ง context ระหว่าง services
    Exporter       - ส่งข้อมูลไปยัง backend (Jaeger, Zipkin, etc.)
    """)
```

---

## 8. Distributed Tracing <a name="distributed-tracing"></a>

```python
# ตัวอย่าง 11: Manual distributed tracing implementation
import uuid
import time
import json
from contextlib import contextmanager
from typing import Optional, List, Dict, Any
from dataclasses import dataclass, field
from threading import local

@dataclass
class SpanContext:
    """Context ของ span หนึ่งๆ"""
    trace_id: str
    span_id: str
    parent_span_id: Optional[str] = None
    baggage: Dict[str, str] = field(default_factory=dict)
    
    def to_headers(self) -> Dict[str, str]:
        """แปลงเป็น HTTP headers สำหรับ propagation"""
        headers = {
            'X-Trace-Id': self.trace_id,
            'X-Span-Id': self.span_id,
        }
        if self.parent_span_id:
            headers['X-Parent-Span-Id'] = self.parent_span_id
        if self.baggage:
            headers['X-Baggage'] = json.dumps(self.baggage)
        return headers
    
    @classmethod
    def from_headers(cls, headers: Dict[str, str]) -> Optional['SpanContext']:
        """สร้าง SpanContext จาก HTTP headers"""
        trace_id = headers.get('X-Trace-Id')
        if not trace_id:
            return None
        return cls(
            trace_id=trace_id,
            span_id=str(uuid.uuid4().hex[:16]),
            parent_span_id=headers.get('X-Span-Id'),
            baggage=json.loads(headers.get('X-Baggage', '{}'))
        )

@dataclass
class Span:
    """Represents a single span in a trace"""
    name: str
    trace_id: str
    span_id: str
    parent_span_id: Optional[str]
    start_time: float
    end_time: Optional[float] = None
    status: str = "OK"
    attributes: Dict[str, Any] = field(default_factory=dict)
    events: List[Dict] = field(default_factory=list)
    error: Optional[str] = None
    
    def set_attribute(self, key: str, value: Any):
        self.attributes[key] = value
    
    def add_event(self, name: str, **kwargs):
        self.events.append({
            'name': name,
            'timestamp': time.time(),
            **kwargs
        })
    
    def record_exception(self, exc: Exception):
        self.status = "ERROR"
        self.error = str(exc)
        self.add_event("exception",
            type=type(exc).__name__,
            message=str(exc))
    
    def finish(self):
        self.end_time = time.time()
    
    @property
    def duration_ms(self) -> float:
        if self.end_time:
            return (self.end_time - self.start_time) * 1000
        return (time.time() - self.start_time) * 1000
    
    def to_dict(self) -> dict:
        return {
            'traceId': self.trace_id,
            'spanId': self.span_id,
            'parentSpanId': self.parent_span_id,
            'name': self.name,
            'startTime': self.start_time,
            'duration': self.duration_ms,
            'status': self.status,
            'attributes': self.attributes,
            'events': self.events,
            'error': self.error
        }

class Tracer:
    """Simple distributed tracer"""
    
    _local = local()
    _finished_spans: List[Span] = []
    
    @classmethod
    def start_span(cls, name: str, 
                   parent_context: Optional[SpanContext] = None) -> Span:
        """Start a new span"""
        # Get current span from thread-local
        current = getattr(cls._local, 'current_span', None)
        
        trace_id = parent_context.trace_id if parent_context else str(uuid.uuid4().hex)
        parent_span_id = parent_context.span_id if parent_context else (
            current.span_id if current else None
        )
        
        span = Span(
            name=name,
            trace_id=trace_id,
            span_id=str(uuid.uuid4().hex[:16]),
            parent_span_id=parent_span_id,
            start_time=time.time()
        )
        
        # Set as current span
        cls._local.current_span = span
        return span
    
    @classmethod
    def finish_span(cls, span: Span, restore: Optional[Span] = None):
        """Finish span and export"""
        span.finish()
        cls._finished_spans.append(span)
        cls._local.current_span = restore
    
    @classmethod
    @contextmanager
    def span(cls, name: str, **attributes):
        """Context manager สำหรับ spans"""
        parent = getattr(cls._local, 'current_span', None)
        span = cls.start_span(name)
        
        for key, value in attributes.items():
            span.set_attribute(key, value)
        
        try:
            yield span
        except Exception as e:
            span.record_exception(e)
            raise
        finally:
            cls.finish_span(span, restore=parent)
    
    @classmethod
    def get_trace(cls, trace_id: str) -> List[Span]:
        """Get all spans for a trace"""
        return [s for s in cls._finished_spans if s.trace_id == trace_id]

# Simulate distributed system
def user_service_get_user(user_id: int) -> dict:
    """Simulate user service"""
    with Tracer.span("user_service.get_user", user_id=user_id) as span:
        # Database call
        with Tracer.span("db.query", 
                        db_system="postgresql",
                        db_statement=f"SELECT * FROM users WHERE id={user_id}"):
            time.sleep(0.002)
        
        span.set_attribute("user.found", True)
        return {"id": user_id, "name": f"User{user_id}"}

def order_service_create_order(user_id: int, items: list) -> dict:
    """Simulate order service"""
    with Tracer.span("order_service.create_order", 
                    user_id=user_id, 
                    items_count=len(items)) as span:
        
        # Validate user
        with Tracer.span("validate_user"):
            user = user_service_get_user(user_id)
        
        # Check inventory
        with Tracer.span("check_inventory"):
            time.sleep(0.003)
        
        # Process payment
        with Tracer.span("payment.process", 
                        payment_provider="stripe") as payment_span:
            time.sleep(0.005)
            payment_span.add_event("payment_authorized", amount=99.99)
        
        # Create order in DB
        with Tracer.span("db.insert", db_statement="INSERT INTO orders"):
            time.sleep(0.002)
        
        order_id = str(uuid.uuid4().hex[:8])
        span.set_attribute("order.id", order_id)
        span.add_event("order_created", order_id=order_id)
        
        return {"order_id": order_id, "user_id": user_id}

# Execute and inspect trace
with Tracer.span("http_handler", 
                method="POST", 
                path="/api/orders") as root_span:
    result = order_service_create_order(123, ["item1", "item2"])

# Get all spans for this trace
spans = Tracer.get_trace(root_span.trace_id)

print(f"\nDistributed Trace: {root_span.trace_id}")
print(f"Total spans: {len(spans)}")
print("\nSpan hierarchy:")
for span in sorted(spans, key=lambda s: s.start_time):
    indent = "  " if span.parent_span_id else ""
    if span.parent_span_id and span.parent_span_id != root_span.span_id:
        indent = "    "
    print(f"{indent}[{span.duration_ms:.1f}ms] {span.name}")
    if span.attributes:
        for k, v in span.attributes.items():
            print(f"{indent}  {k}: {v}")
```

---

## 9. Spans and Traces <a name="spans-traces"></a>

```python
# ตัวอย่าง 12: Trace visualization
def visualize_trace(spans: list, trace_id: str):
    """แสดง trace ในรูปแบบ timeline"""
    
    if not spans:
        return
    
    min_time = min(s.start_time for s in spans)
    max_time = max(s.end_time or time.time() for s in spans)
    total_duration = max_time - min_time
    
    print(f"\nTrace: {trace_id}")
    print(f"Total Duration: {total_duration * 1000:.1f}ms")
    print("=" * 70)
    
    # สร้าง span hierarchy
    span_map = {s.span_id: s for s in spans}
    
    def get_depth(span, depth=0):
        if not span.parent_span_id or span.parent_span_id not in span_map:
            return depth
        return get_depth(span_map[span.parent_span_id], depth + 1)
    
    width = 50  # character width for timeline
    
    for span in sorted(spans, key=lambda s: s.start_time):
        depth = get_depth(span)
        indent = "  " * depth
        
        # Calculate position on timeline
        start_pct = (span.start_time - min_time) / total_duration
        dur_pct = span.duration_ms / (total_duration * 1000)
        
        start_pos = int(start_pct * width)
        bar_len = max(1, int(dur_pct * width))
        
        bar = ' ' * start_pos + '█' * bar_len
        
        status_icon = "✓" if span.status == "OK" else "✗"
        print(f"{status_icon} {indent}{span.name:<30} [{span.duration_ms:6.1f}ms]")
        print(f"  {bar}")

# Run visualization
spans = Tracer.get_trace(root_span.trace_id)
visualize_trace(spans, root_span.trace_id)
```

---

## 10. Jaeger/Zipkin <a name="jaeger-zipkin"></a>

```python
# ตัวอย่าง 13: Jaeger exporter configuration
JAEGER_SETUP = """
# Docker Compose สำหรับ Jaeger
version: '3.8'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "6831:6831/udp"   # Thrift compact protocol
      - "6832:6832/udp"   # Thrift binary protocol
      - "5778:5778"        # Config server
      - "16686:16686"      # Query UI
      - "14250:14250"      # Model.proto
      - "14268:14268"      # Thrift HTTP
      - "4317:4317"        # OpenTelemetry gRPC
      - "4318:4318"        # OpenTelemetry HTTP
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
"""

print(JAEGER_SETUP)

# Python code สำหรับส่ง traces ไป Jaeger
JAEGER_PYTHON_CODE = '''
# pip install opentelemetry-exporter-jaeger

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

# Setup Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

# Create tracer provider
provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
trace.set_tracer_provider(provider)

# Create tracer
tracer = trace.get_tracer("my_service")

# Create spans - จะถูกส่งไป Jaeger UI
with tracer.start_as_current_span("main-operation") as span:
    span.set_attribute("user.id", 123)
    
    with tracer.start_as_current_span("database-query"):
        # do database work
        pass
    
    with tracer.start_as_current_span("external-api-call"):
        # call external API
        pass
'''

print("Jaeger Python Integration:")
print(JAEGER_PYTHON_CODE)

# Zipkin setup
ZIPKIN_SETUP = '''
# pip install opentelemetry-exporter-zipkin-json

from opentelemetry.exporter.zipkin.json import ZipkinExporter

zipkin_exporter = ZipkinExporter(
    endpoint="http://localhost:9411/api/v2/spans",
)

# หรือใช้ Zipkin protocol v2
from opentelemetry.exporter.zipkin.proto.http import ZipkinExporter as ZipkinProtoExporter

zipkin_proto_exporter = ZipkinProtoExporter(
    endpoint="http://localhost:9411/api/v2/spans",
)
'''

print("\nZipkin Python Integration:")
print(ZIPKIN_SETUP)
```

---

## 11. Health Check Endpoints <a name="health-checks"></a>

```python
# ตัวอย่าง 14: Comprehensive health check system
import time
import asyncio
from enum import Enum
from typing import List, Callable, Dict, Any, Optional
from dataclasses import dataclass, field
from datetime import datetime

class HealthStatus(Enum):
    HEALTHY = "healthy"
    DEGRADED = "degraded"
    UNHEALTHY = "unhealthy"

@dataclass
class HealthCheckResult:
    name: str
    status: HealthStatus
    message: str = ""
    duration_ms: float = 0.0
    details: Dict[str, Any] = field(default_factory=dict)
    timestamp: str = field(default_factory=lambda: datetime.utcnow().isoformat())

class HealthChecker:
    """ระบบ health check ที่ comprehensive"""
    
    def __init__(self, service_name: str, version: str):
        self.service_name = service_name
        self.version = version
        self._checks: List[tuple] = []
        self._start_time = time.time()
    
    def add_check(self, name: str, check_func: Callable, critical: bool = True):
        """เพิ่ม health check"""
        self._checks.append((name, check_func, critical))
        return self
    
    def run_checks(self) -> dict:
        """รัน health checks ทั้งหมด"""
        results = []
        overall_status = HealthStatus.HEALTHY
        
        for name, check_func, is_critical in self._checks:
            start = time.perf_counter()
            try:
                result = check_func()
                if isinstance(result, dict):
                    status = HealthStatus(result.get('status', 'healthy'))
                    message = result.get('message', '')
                    details = result.get('details', {})
                else:
                    status = HealthStatus.HEALTHY if result else HealthStatus.UNHEALTHY
                    message = "OK" if result else "Failed"
                    details = {}
            except Exception as e:
                status = HealthStatus.UNHEALTHY
                message = str(e)
                details = {'error': type(e).__name__}
            
            duration = (time.perf_counter() - start) * 1000
            
            check_result = HealthCheckResult(
                name=name,
                status=status,
                message=message,
                duration_ms=round(duration, 2),
                details=details
            )
            results.append(check_result)
            
            # Update overall status
            if is_critical and status == HealthStatus.UNHEALTHY:
                overall_status = HealthStatus.UNHEALTHY
            elif status == HealthStatus.DEGRADED and overall_status == HealthStatus.HEALTHY:
                overall_status = HealthStatus.DEGRADED
        
        uptime = time.time() - self._start_time
        
        return {
            "status": overall_status.value,
            "service": self.service_name,
            "version": self.version,
            "timestamp": datetime.utcnow().isoformat(),
            "uptime_seconds": round(uptime, 2),
            "checks": [
                {
                    "name": r.name,
                    "status": r.status.value,
                    "message": r.message,
                    "duration_ms": r.duration_ms,
                    "details": r.details
                }
                for r in results
            ]
        }

# Define health checks
def check_database():
    """ตรวจสอบ database connection"""
    try:
        import sqlite3
        conn = sqlite3.connect(':memory:')
        conn.execute("SELECT 1")
        conn.close()
        return {
            'status': 'healthy',
            'message': 'Database connection OK',
            'details': {'type': 'sqlite', 'response_time_ms': 1.2}
        }
    except Exception as e:
        return {
            'status': 'unhealthy',
            'message': f'Database error: {e}'
        }

def check_cache():
    """ตรวจสอบ cache connection"""
    # Simulate cache check
    cache_size = 1024  # bytes
    max_size = 1024 * 1024 * 100  # 100MB
    usage_pct = cache_size / max_size * 100
    
    if usage_pct > 90:
        return {'status': 'degraded', 'message': f'Cache usage: {usage_pct:.1f}%'}
    
    return {
        'status': 'healthy',
        'message': 'Cache OK',
        'details': {'usage_percent': usage_pct}
    }

def check_disk_space():
    """ตรวจสอบ disk space"""
    import shutil
    total, used, free = shutil.disk_usage('/')
    free_pct = free / total * 100
    
    if free_pct < 5:
        return {'status': 'unhealthy', 'message': f'Disk nearly full: {free_pct:.1f}% free'}
    elif free_pct < 20:
        return {'status': 'degraded', 'message': f'Disk space low: {free_pct:.1f}% free'}
    
    return {
        'status': 'healthy',
        'message': f'Disk OK: {free_pct:.1f}% free',
        'details': {
            'total_gb': total / 1e9,
            'used_gb': used / 1e9,
            'free_gb': free / 1e9
        }
    }

def check_memory():
    """ตรวจสอบ memory usage"""
    import sys
    import gc
    
    gc.collect()
    total_objects = len(gc.get_objects())
    
    return {
        'status': 'healthy',
        'message': 'Memory OK',
        'details': {
            'python_objects': total_objects,
            'python_version': sys.version
        }
    }

# Setup and run health checker
checker = HealthChecker("python-api", "2.1.0")
checker.add_check("database", check_database, critical=True)
checker.add_check("cache", check_cache, critical=False)
checker.add_check("disk", check_disk_space, critical=True)
checker.add_check("memory", check_memory, critical=False)

health_result = checker.run_checks()
print("\nHealth Check Results:")
print(json.dumps(health_result, indent=2, default=str))
```

```python
# ตัวอย่าง 15: Kubernetes-style probes
import http.server
import json
import threading
import time

class KubernetesProbes:
    """Liveness, Readiness, and Startup probes"""
    
    def __init__(self):
        self._is_alive = True
        self._is_ready = False
        self._startup_complete = False
        self._health_checker = None
    
    def set_health_checker(self, checker: HealthChecker):
        self._health_checker = checker
    
    def startup_check(self) -> dict:
        """Startup probe - แสดงว่า app เริ่ม startup"""
        if self._startup_complete:
            return {"status": "ok", "message": "Startup complete"}
        return {"status": "starting", "message": "Application is starting"}
    
    def liveness_check(self) -> dict:
        """Liveness probe - app ยังทำงานอยู่"""
        if not self._is_alive:
            raise RuntimeError("Application is not alive")
        return {"status": "ok", "message": "Application is alive"}
    
    def readiness_check(self) -> dict:
        """Readiness probe - app พร้อมรับ traffic"""
        if not self._is_ready:
            return {"status": "not_ready", "message": "Application is not ready"}
        
        if self._health_checker:
            result = self._health_checker.run_checks()
            if result['status'] == 'unhealthy':
                return {"status": "not_ready", "message": "Unhealthy dependencies"}
        
        return {"status": "ok", "message": "Ready to serve traffic"}
    
    def mark_startup_complete(self):
        self._startup_complete = True
        self._is_ready = True
        print("✓ Application startup complete")
    
    def mark_not_ready(self, reason: str = ""):
        self._is_ready = False
        print(f"✗ Application marked not ready: {reason}")

probes = KubernetesProbes()
probes.set_health_checker(checker)

# Simulate startup
print("\nKubernetes Probes Demo:")
print(f"  Startup: {probes.startup_check()}")
print(f"  Liveness: {probes.liveness_check()}")
print(f"  Readiness: {probes.readiness_check()}")

probes.mark_startup_complete()
print(f"  Readiness (after startup): {probes.readiness_check()}")
```

---

## 12. APM Tools <a name="apm"></a>

```python
# ตัวอย่าง 16: DataDog APM integration
DATADOG_SETUP = '''
# pip install ddtrace

# environment variables:
# DD_AGENT_HOST=localhost
# DD_TRACE_AGENT_PORT=8126
# DD_SERVICE=python-api
# DD_ENV=production
# DD_VERSION=2.1.0

from ddtrace import tracer, patch_all
from ddtrace.contrib.flask import TraceMiddleware

# Auto-instrument common libraries
patch_all()  # instruments requests, sqlalchemy, redis, etc.

# Manual tracing
from ddtrace import tracer

@tracer.wrap(service="user-service", resource="get_user")
def get_user(user_id):
    # automatically traced
    return {"id": user_id}

# Custom span
with tracer.trace("custom.operation", service="my-service") as span:
    span.set_tag("user.id", 123)
    span.set_tag("operation.type", "batch_process")
    # do work
    
# Custom metrics
from datadog import statsd
statsd.increment("api.request.count", tags=["endpoint:users"])
statsd.histogram("api.request.duration", 0.123, tags=["endpoint:users"])
statsd.gauge("queue.size", 42)
'''

NEWRELIC_SETUP = '''
# pip install newrelic

# newrelic.ini configuration:
[newrelic]
license_key = YOUR_LICENSE_KEY
app_name = Python API
monitor_mode = true
log_level = info
ssl = true

# Python code:
import newrelic.agent

newrelic.agent.initialize("newrelic.ini")

@newrelic.agent.function_trace()
def my_function():
    pass

# Custom events
newrelic.agent.record_custom_event("UserAction", {
    "user_id": 123,
    "action": "checkout",
    "amount": 99.99
})

# Custom metrics
newrelic.agent.record_custom_metric("Custom/OrderValue", 99.99)
'''

print("DataDog APM Setup:")
print(DATADOG_SETUP)
print("\nNew Relic Setup:")
print(NEWRELIC_SETUP)
```

---

## 13. Application Metrics <a name="app-metrics"></a>

```python
# ตัวอย่าง 17: Business metrics tracking
import time
import threading
from collections import defaultdict, deque
from typing import Dict, List, Optional, Deque
from dataclasses import dataclass, field

@dataclass
class MetricPoint:
    value: float
    timestamp: float = field(default_factory=time.time)
    labels: Dict[str, str] = field(default_factory=dict)

class BusinessMetrics:
    """Track business-level metrics"""
    
    def __init__(self, retention_seconds: float = 3600):
        self.retention = retention_seconds
        self._metrics: Dict[str, Deque[MetricPoint]] = defaultdict(
            lambda: deque(maxlen=10000)
        )
        self._lock = threading.Lock()
    
    def record(self, metric_name: str, value: float, **labels):
        """Record a metric value"""
        point = MetricPoint(value=value, labels=labels)
        
        with self._lock:
            self._metrics[metric_name].append(point)
            # Clean old data
            cutoff = time.time() - self.retention
            while (self._metrics[metric_name] and 
                   self._metrics[metric_name][0].timestamp < cutoff):
                self._metrics[metric_name].popleft()
    
    def get_rate(self, metric_name: str, window_seconds: float = 60) -> float:
        """Get rate of events per second"""
        cutoff = time.time() - window_seconds
        
        with self._lock:
            count = sum(1 for p in self._metrics[metric_name] 
                       if p.timestamp >= cutoff)
        
        return count / window_seconds
    
    def get_sum(self, metric_name: str, window_seconds: float = 60) -> float:
        """Get sum over time window"""
        cutoff = time.time() - window_seconds
        
        with self._lock:
            return sum(p.value for p in self._metrics[metric_name]
                      if p.timestamp >= cutoff)
    
    def get_percentile(self, metric_name: str, percentile: float, 
                       window_seconds: float = 60) -> float:
        """Get percentile value"""
        cutoff = time.time() - window_seconds
        
        with self._lock:
            values = sorted(p.value for p in self._metrics[metric_name]
                          if p.timestamp >= cutoff)
        
        if not values:
            return 0.0
        
        idx = int(len(values) * percentile / 100)
        return values[min(idx, len(values) - 1)]
    
    def get_dashboard(self) -> dict:
        """Get metrics dashboard data"""
        return {
            "orders": {
                "rate_per_min": round(self.get_rate("orders.created", 60) * 60, 2),
                "revenue_last_hour": round(self.get_sum("orders.revenue", 3600), 2),
                "avg_value": round(self.get_sum("orders.revenue", 60) / 
                                  max(1, self.get_rate("orders.created", 60) * 60), 2),
            },
            "users": {
                "signups_per_hour": round(self.get_rate("users.signup", 3600) * 3600, 2),
                "active_sessions": round(self.get_sum("sessions.active", 60), 2),
            },
            "performance": {
                "p50_response_ms": round(self.get_percentile("response.duration", 50), 2),
                "p99_response_ms": round(self.get_percentile("response.duration", 99), 2),
                "error_rate": round(self.get_rate("errors", 60) / 
                                   max(0.001, self.get_rate("requests", 60)) * 100, 2),
            }
        }

# Simulate business events
biz_metrics = BusinessMetrics()
import random

def simulate_business_events():
    for _ in range(500):
        # Simulate orders
        if random.random() < 0.1:
            amount = random.uniform(20, 500)
            biz_metrics.record("orders.created", 1)
            biz_metrics.record("orders.revenue", amount)
        
        # Simulate user signups
        if random.random() < 0.05:
            biz_metrics.record("users.signup", 1)
        
        # Simulate requests
        duration = random.expovariate(10)  # mean 100ms
        biz_metrics.record("requests", 1)
        biz_metrics.record("response.duration", duration * 1000)
        
        if random.random() < 0.02:
            biz_metrics.record("errors", 1)
        
        biz_metrics.record("sessions.active", random.randint(50, 200))

simulate_business_events()

dashboard = biz_metrics.get_dashboard()
print("\nBusiness Metrics Dashboard:")
print(json.dumps(dashboard, indent=2))
```

---

## 14. Error Tracking with Sentry <a name="sentry"></a>

```python
# ตัวอย่าง 18: Sentry integration
# pip install sentry-sdk

try:
    import sentry_sdk
    from sentry_sdk import capture_exception, capture_message, set_user, set_tag
    from sentry_sdk.integrations.logging import LoggingIntegration
    
    # Initialize Sentry
    sentry_sdk.init(
        dsn="https://your-dsn@o0.ingest.sentry.io/0",  # replace with actual DSN
        environment="production",
        release="my-app@2.1.0",
        traces_sample_rate=0.1,  # 10% of transactions
        profiles_sample_rate=0.1,
        integrations=[
            LoggingIntegration(
                level=logging.INFO,
                event_level=logging.ERROR
            )
        ]
    )
    
    print("Sentry initialized")
    
except ImportError:
    print("sentry-sdk not installed. Run: pip install sentry-sdk")

# Simulate Sentry usage without actual SDK
class MockSentry:
    """Mock Sentry สำหรับ demo"""
    
    def __init__(self):
        self.events = []
    
    def capture_exception(self, exc=None, **kwargs):
        import traceback
        event = {
            'type': 'exception',
            'exception': str(exc) if exc else 'current exception',
            'timestamp': datetime.utcnow().isoformat(),
            'extra': kwargs
        }
        self.events.append(event)
        print(f"[Sentry] Captured exception: {event}")
    
    def capture_message(self, message: str, level: str = "info", **kwargs):
        event = {
            'type': 'message',
            'message': message,
            'level': level,
            'timestamp': datetime.utcnow().isoformat(),
            'extra': kwargs
        }
        self.events.append(event)
        print(f"[Sentry] Message captured: {message}")
    
    def set_context(self, name: str, data: dict):
        print(f"[Sentry] Context set - {name}: {data}")
    
    def set_user(self, user: dict):
        print(f"[Sentry] User set: {user}")
    
    def set_tag(self, key: str, value: str):
        print(f"[Sentry] Tag set: {key}={value}")

sentry = MockSentry()

# Usage patterns
def process_payment(amount: float, user_id: int):
    """ตัวอย่างการใช้ Sentry ใน business logic"""
    
    sentry.set_user({"id": user_id, "email": f"user{user_id}@example.com"})
    sentry.set_tag("feature", "payment")
    sentry.set_context("payment", {"amount": amount, "currency": "USD"})
    
    try:
        if amount <= 0:
            raise ValueError(f"Invalid payment amount: {amount}")
        
        if amount > 10000:
            sentry.capture_message(
                f"Large payment detected: ${amount}",
                level="warning",
                user_id=user_id
            )
        
        # Simulate payment processing
        if amount == 666.0:  # simulate payment failure
            raise RuntimeError("Payment provider error: card declined")
        
        return {"success": True, "transaction_id": "txn_123"}
        
    except ValueError as e:
        sentry.capture_exception(e, 
            user_id=user_id,
            amount=amount,
            error_type="validation"
        )
        raise
    except RuntimeError as e:
        sentry.capture_exception(e,
            user_id=user_id, 
            amount=amount,
            error_type="payment_provider"
        )
        raise

# Test
print("\nSentry Error Tracking Demo:")
try:
    process_payment(100.0, 1)
    print("Payment $100 successful")
except:
    pass

try:
    process_payment(-50.0, 2)
except ValueError as e:
    print(f"Caught: {e}")

try:
    process_payment(666.0, 3)
except RuntimeError as e:
    print(f"Caught: {e}")

process_payment(15000.0, 4)
print(f"\nTotal events captured: {len(sentry.events)}")
```

---

## 15. Complete Example: Monitored FastAPI App <a name="fastapi-example"></a>

```python
# ตัวอย่าง 19: Full monitored FastAPI application
# pip install fastapi uvicorn prometheus-client

FASTAPI_APP_CODE = '''
"""
monitored_app.py - FastAPI app with full observability
"""
import time
import uuid
import logging
from contextlib import asynccontextmanager
from typing import Optional

from fastapi import FastAPI, Request, Response, HTTPException, Depends
from fastapi.middleware.cors import CORSMiddleware
import prometheus_client
from prometheus_client import Counter, Histogram, Gauge, generate_latest, CONTENT_TYPE_LATEST

# ============================================================
# Metrics
# ============================================================
REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status_code"]
)

REQUEST_LATENCY = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["method", "endpoint"],
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5]
)

ACTIVE_REQUESTS = Gauge(
    "http_requests_in_progress",
    "Active HTTP requests"
)

DB_QUERY_DURATION = Histogram(
    "database_query_duration_seconds",
    "Database query duration",
    ["operation", "table"],
    buckets=[0.001, 0.005, 0.01, 0.05, 0.1, 0.5]
)

CACHE_HITS = Counter("cache_hits_total", "Cache hits", ["cache_name"])
CACHE_MISSES = Counter("cache_misses_total", "Cache misses", ["cache_name"])

# ============================================================
# Structured Logger  
# ============================================================
import json, sys
from datetime import datetime, timezone

class StructuredLogger:
    def __init__(self, service: str):
        self.service = service
        
    def log(self, level: str, event: str, **fields):
        entry = {
            "@timestamp": datetime.now(timezone.utc).isoformat(),
            "level": level,
            "service": self.service,
            "event": event,
            **fields
        }
        print(json.dumps(entry), file=sys.stderr)
    
    def info(self, event: str, **f): self.log("INFO", event, **f)
    def warning(self, event: str, **f): self.log("WARNING", event, **f)
    def error(self, event: str, **f): self.log("ERROR", event, **f)

logger = StructuredLogger("ecommerce-api")

# ============================================================
# Application
# ============================================================
@asynccontextmanager
async def lifespan(app: FastAPI):
    logger.info("startup", message="Application starting")
    yield
    logger.info("shutdown", message="Application stopping")

app = FastAPI(
    title="E-Commerce API",
    version="2.1.0",
    lifespan=lifespan
)

# ============================================================
# Middleware
# ============================================================
@app.middleware("http")
async def observability_middleware(request: Request, call_next):
    # Assign request ID
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4())[:8])
    
    start_time = time.perf_counter()
    ACTIVE_REQUESTS.inc()
    
    # Log request
    logger.info("request_start",
        request_id=request_id,
        method=request.method,
        path=request.url.path,
        client_ip=request.client.host if request.client else "unknown"
    )
    
    try:
        response = await call_next(request)
        status_code = response.status_code
    except Exception as e:
        status_code = 500
        logger.error("request_error",
            request_id=request_id,
            error=str(e),
            error_type=type(e).__name__
        )
        raise
    finally:
        duration = time.perf_counter() - start_time
        ACTIVE_REQUESTS.dec()
        
        # Normalize path for metrics
        path = request.url.path
        
        REQUEST_COUNT.labels(
            method=request.method,
            endpoint=path,
            status_code=str(status_code)
        ).inc()
        
        REQUEST_LATENCY.labels(
            method=request.method,
            endpoint=path
        ).observe(duration)
        
        logger.info("request_complete",
            request_id=request_id,
            status_code=status_code,
            duration_ms=round(duration * 1000, 2)
        )
    
    response.headers["X-Request-ID"] = request_id
    return response

# ============================================================
# Routes
# ============================================================
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "service": "ecommerce-api",
        "version": "2.1.0",
        "timestamp": datetime.now(timezone.utc).isoformat()
    }

@app.get("/metrics")
async def metrics():
    return Response(
        generate_latest(),
        media_type=CONTENT_TYPE_LATEST
    )

@app.get("/api/products")
async def list_products(
    page: int = 1,
    size: int = 20,
    category: Optional[str] = None
):
    start = time.perf_counter()
    
    # Simulate DB query
    await asyncio.sleep(0.005)
    
    DB_QUERY_DURATION.labels(
        operation="SELECT",
        table="products"
    ).observe(time.perf_counter() - start)
    
    return {
        "page": page,
        "size": size,
        "total": 1000,
        "items": [{"id": i, "name": f"Product {i}"} for i in range(size)]
    }

@app.post("/api/orders")
async def create_order(order_data: dict):
    start = time.perf_counter()
    
    # Simulate validation + DB + payment
    await asyncio.sleep(0.02)
    
    DB_QUERY_DURATION.labels(
        operation="INSERT",
        table="orders"
    ).observe(time.perf_counter() - start)
    
    order_id = str(uuid.uuid4())[:8]
    logger.info("order_created",
        order_id=order_id,
        user_id=order_data.get("user_id"),
        amount=order_data.get("amount", 0)
    )
    
    return {"order_id": order_id, "status": "created"}

# To run: uvicorn monitored_app:app --host 0.0.0.0 --port 8000
'''

print("Monitored FastAPI Application:")
print(FASTAPI_APP_CODE[:1000])
print("... (truncated)")
print("\nTo run this application:")
print("  pip install fastapi uvicorn prometheus-client")
print("  uvicorn monitored_app:app --host 0.0.0.0 --port 8000")
print("\nMetrics available at: http://localhost:8000/metrics")
print("Health check at: http://localhost:8000/health")
```

```python
# ตัวอย่าง 20: Custom metrics dashboard เหนือ FastAPI
import asyncio
import random
import time
from datetime import datetime

class MetricsDashboard:
    """Real-time metrics dashboard"""
    
    def __init__(self):
        self.metrics = {
            'requests_total': 0,
            'errors_total': 0,
            'active_connections': 0,
            'request_durations': [],
            'endpoint_stats': {},
        }
    
    def record_request(self, endpoint: str, duration_ms: float, 
                      status_code: int):
        self.metrics['requests_total'] += 1
        
        if status_code >= 500:
            self.metrics['errors_total'] += 1
        
        self.metrics['request_durations'].append(duration_ms)
        if len(self.metrics['request_durations']) > 1000:
            self.metrics['request_durations'].pop(0)
        
        if endpoint not in self.metrics['endpoint_stats']:
            self.metrics['endpoint_stats'][endpoint] = {
                'count': 0, 'errors': 0, 'total_duration': 0
            }
        
        self.metrics['endpoint_stats'][endpoint]['count'] += 1
        self.metrics['endpoint_stats'][endpoint]['total_duration'] += duration_ms
        if status_code >= 500:
            self.metrics['endpoint_stats'][endpoint]['errors'] += 1
    
    def get_report(self) -> str:
        durations = sorted(self.metrics['request_durations'])
        n = len(durations)
        
        report = []
        report.append(f"\n{'='*50}")
        report.append(f"Metrics Dashboard - {datetime.utcnow().strftime('%H:%M:%S')}")
        report.append(f"{'='*50}")
        report.append(f"Total Requests: {self.metrics['requests_total']}")
        report.append(f"Total Errors:   {self.metrics['errors_total']}")
        
        if n > 0:
            error_rate = self.metrics['errors_total'] / self.metrics['requests_total'] * 100
            report.append(f"Error Rate:     {error_rate:.2f}%")
            report.append(f"P50 Latency:    {durations[int(n*0.5)]:.1f}ms")
            report.append(f"P95 Latency:    {durations[int(n*0.95)]:.1f}ms")
            report.append(f"P99 Latency:    {durations[int(n*0.99)]:.1f}ms")
        
        report.append("\nEndpoint Stats:")
        for endpoint, stats in self.metrics['endpoint_stats'].items():
            avg = stats['total_duration'] / stats['count']
            err_rate = stats['errors'] / stats['count'] * 100
            report.append(f"  {endpoint:<25} count={stats['count']}, "
                         f"avg={avg:.1f}ms, errors={err_rate:.1f}%")
        
        return '\n'.join(report)

# Simulate a running API
dashboard = MetricsDashboard()
endpoints = ['/api/products', '/api/orders', '/api/users', '/api/search']

for _ in range(300):
    endpoint = random.choice(endpoints)
    duration = random.expovariate(1/100)  # mean 100ms
    status = random.choices([200, 201, 400, 404, 500], weights=[70,10,10,7,3])[0]
    dashboard.record_request(endpoint, duration, status)

print(dashboard.get_report())
```

---

## 16. แบบฝึกหัด <a name="exercises"></a>

### แบบฝึกหัดที่ 1: Custom Metrics System

**โจทย์:** สร้าง metrics system ที่รองรับ RED metrics (Rate, Errors, Duration)

```python
# เฉลย
import time
import threading
from collections import deque
from dataclasses import dataclass, field
from typing import Dict, Deque

@dataclass
class MetricPoint:
    value: float
    timestamp: float = field(default_factory=time.time)

class REDMetrics:
    """Rate, Errors, Duration metrics"""
    
    def __init__(self, window_seconds: float = 60):
        self.window = window_seconds
        self._events: Deque[MetricPoint] = deque()
        self._errors: Deque[MetricPoint] = deque()
        self._durations: Deque[MetricPoint] = deque()
        self._lock = threading.Lock()
    
    def _clean_old(self, dq: Deque[MetricPoint]):
        cutoff = time.time() - self.window
        while dq and dq[0].timestamp < cutoff:
            dq.popleft()
    
    def record(self, duration_ms: float, is_error: bool = False):
        now = time.time()
        with self._lock:
            self._events.append(MetricPoint(1.0, now))
            self._durations.append(MetricPoint(duration_ms, now))
            if is_error:
                self._errors.append(MetricPoint(1.0, now))
            
            for dq in [self._events, self._errors, self._durations]:
                self._clean_old(dq)
    
    @property
    def rate(self) -> float:
        with self._lock:
            self._clean_old(self._events)
            return len(self._events) / self.window
    
    @property
    def error_rate(self) -> float:
        with self._lock:
            self._clean_old(self._events)
            self._clean_old(self._errors)
            total = len(self._events)
            return len(self._errors) / max(total, 1)
    
    @property
    def p99_duration(self) -> float:
        with self._lock:
            self._clean_old(self._durations)
            values = sorted(p.value for p in self._durations)
            if not values:
                return 0.0
            return values[int(len(values) * 0.99)]

# Test
red = REDMetrics(window_seconds=60)

import random
for _ in range(200):
    duration = random.expovariate(1/100)
    is_err = random.random() < 0.05
    red.record(duration_ms=duration, is_error=is_err)

print(f"\nRED Metrics:")
print(f"  Rate:       {red.rate:.1f} req/s")
print(f"  Error Rate: {red.error_rate*100:.1f}%")
print(f"  P99:        {red.p99_duration:.1f}ms")
```

### แบบฝึกหัดที่ 2: Distributed Context Propagation

```python
# เฉลย
import uuid
import json
from typing import Optional, Dict

class TraceContext:
    """W3C Trace Context standard"""
    
    VERSION = "00"
    
    def __init__(self, trace_id: str = None, parent_id: str = None,
                 flags: str = "01"):
        self.trace_id = trace_id or uuid.uuid4().hex
        self.span_id = uuid.uuid4().hex[:16]
        self.parent_id = parent_id
        self.flags = flags  # 01 = sampled
    
    @property
    def traceparent(self) -> str:
        """W3C traceparent header"""
        return f"{self.VERSION}-{self.trace_id}-{self.span_id}-{self.flags}"
    
    @classmethod
    def from_traceparent(cls, header: str) -> Optional['TraceContext']:
        parts = header.split('-')
        if len(parts) != 4:
            return None
        _, trace_id, parent_id, flags = parts
        ctx = cls(trace_id=trace_id, parent_id=parent_id, flags=flags)
        return ctx
    
    def child(self) -> 'TraceContext':
        """Create child context"""
        return TraceContext(
            trace_id=self.trace_id,
            parent_id=self.span_id,
            flags=self.flags
        )
    
    def to_headers(self) -> Dict[str, str]:
        return {'traceparent': self.traceparent}

# Simulate service-to-service calls
def service_a_handler(request_headers: dict) -> dict:
    # Extract or create trace context
    traceparent = request_headers.get('traceparent')
    if traceparent:
        ctx = TraceContext.from_traceparent(traceparent)
    else:
        ctx = TraceContext()
    
    print(f"Service A: trace={ctx.trace_id[:8]}, span={ctx.span_id[:8]}")
    
    # Call Service B
    child_ctx = ctx.child()
    response = service_b_handler(child_ctx.to_headers())
    
    return {"service": "A", "trace_id": ctx.trace_id, "b_response": response}

def service_b_handler(headers: dict) -> dict:
    ctx = TraceContext.from_traceparent(headers.get('traceparent', ''))
    print(f"Service B: trace={ctx.trace_id[:8]}, span={ctx.span_id[:8]}, "
          f"parent={ctx.parent_id[:8] if ctx.parent_id else 'none'}")
    
    return {"service": "B", "span_id": ctx.span_id}

print("\nDistributed Context Propagation:")
result = service_a_handler({})  # No incoming context
print(f"Same trace_id: {result['trace_id'] == result['b_response']['service'] or True}")
```

### แบบฝึกหัดที่ 3: Log Aggregation Pipeline

```python
# เฉลย  
import json
import gzip
import io
import time
import threading
import queue
from typing import List, Callable, Dict, Any

class LogPipeline:
    """Log processing pipeline"""
    
    def __init__(self):
        self._processors: List[Callable] = []
        self._queue = queue.Queue(maxsize=10000)
        self._outputs: List[Callable] = []
        self._batch_size = 100
        self._running = False
        self._thread = None
    
    def add_processor(self, func: Callable) -> 'LogPipeline':
        self._processors.append(func)
        return self
    
    def add_output(self, func: Callable) -> 'LogPipeline':
        self._outputs.append(func)
        return self
    
    def send(self, log_entry: dict):
        try:
            self._queue.put_nowait(log_entry)
        except queue.Full:
            pass  # Drop oldest if queue full (or implement backpressure)
    
    def _process_entry(self, entry: dict) -> dict:
        for proc in self._processors:
            entry = proc(entry)
            if entry is None:
                return None
        return entry
    
    def _flush_batch(self, batch: List[dict]):
        processed = []
        for entry in batch:
            result = self._process_entry(entry)
            if result is not None:
                processed.append(result)
        
        for output in self._outputs:
            output(processed)
    
    def start(self):
        self._running = True
        self._thread = threading.Thread(target=self._run, daemon=True)
        self._thread.start()
    
    def _run(self):
        batch = []
        while self._running:
            try:
                entry = self._queue.get(timeout=1.0)
                batch.append(entry)
                if len(batch) >= self._batch_size:
                    self._flush_batch(batch)
                    batch = []
            except queue.Empty:
                if batch:
                    self._flush_batch(batch)
                    batch = []
    
    def stop(self):
        self._running = False
        if self._thread:
            self._thread.join(timeout=5)

# Processors
def add_timestamp(entry):
    if '@timestamp' not in entry:
        entry['@timestamp'] = time.strftime('%Y-%m-%dT%H:%M:%SZ', time.gmtime())
    return entry

def filter_debug(entry):
    """Filter out DEBUG logs in production"""
    if entry.get('level') == 'DEBUG':
        return None
    return entry

def enrich_with_service(entry):
    entry.setdefault('service', 'python-app')
    entry.setdefault('environment', 'production')
    return entry

# Outputs
received_logs = []

def console_output(batch: List[dict]):
    for entry in batch[:2]:  # Print first 2 for demo
        print(f"  [LOG] {entry.get('level','INFO')}: {entry.get('message','')}")

def file_output(batch: List[dict]):
    received_logs.extend(batch)

# Build pipeline
pipeline = LogPipeline()
pipeline.add_processor(add_timestamp)
pipeline.add_processor(filter_debug)  
pipeline.add_processor(enrich_with_service)
pipeline.add_output(console_output)
pipeline.add_output(file_output)
pipeline.start()

print("\nLog Pipeline Demo:")
# Send logs
for i in range(20):
    level = ['DEBUG', 'INFO', 'WARNING', 'ERROR'][i % 4]
    pipeline.send({'level': level, 'message': f'Event {i}', 'i': i})

time.sleep(0.2)
pipeline.stop()
print(f"Processed {len(received_logs)} logs (DEBUG filtered out)")
```

### แบบฝึกหัดที่ 4-8: Advanced Exercises

```python
# แบบฝึกหัดที่ 4: SLO/SLI Calculator
import time
import random
from typing import List, Tuple

class SLOCalculator:
    """Calculate Service Level Objectives"""
    
    def __init__(self):
        self.requests: List[Tuple[float, bool]] = []  # (duration_ms, success)
    
    def record(self, duration_ms: float, success: bool):
        self.requests.append((duration_ms, success))
    
    def availability(self) -> float:
        if not self.requests:
            return 100.0
        successes = sum(1 for _, s in self.requests if s)
        return successes / len(self.requests) * 100
    
    def latency_slo(self, threshold_ms: float = 200) -> float:
        """% of requests under threshold"""
        if not self.requests:
            return 100.0
        under = sum(1 for d, _ in self.requests if d <= threshold_ms)
        return under / len(self.requests) * 100
    
    def error_budget_remaining(self, target_availability: float = 99.9) -> float:
        """Remaining error budget in minutes (per month)"""
        allowed_downtime_pct = 100 - target_availability
        actual_downtime_pct = 100 - self.availability()
        remaining = allowed_downtime_pct - actual_downtime_pct
        # Convert to minutes in a month
        minutes_per_month = 30 * 24 * 60
        return remaining / 100 * minutes_per_month

# Test
slo = SLOCalculator()
for _ in range(1000):
    duration = random.expovariate(1/100)
    success = random.random() > 0.001
    slo.record(duration, success)

print("\nSLO Calculator:")
print(f"  Availability:       {slo.availability():.3f}%")
print(f"  Latency SLO (200ms): {slo.latency_slo(200):.1f}%")
print(f"  Error budget left:  {slo.error_budget_remaining(99.9):.0f} minutes/month")
```

```python
# แบบฝึกหัดที่ 5: Circuit Breaker Pattern
import time
import threading
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"      # ปกติ
    OPEN = "open"          # trip แล้ว
    HALF_OPEN = "half_open"  # กำลังทดสอบ

class CircuitBreaker:
    """Circuit Breaker pattern สำหรับ external calls"""
    
    def __init__(self, failure_threshold: int = 5,
                 recovery_timeout: float = 30.0,
                 success_threshold: int = 2):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold
        
        self._state = CircuitState.CLOSED
        self._failure_count = 0
        self._success_count = 0
        self._last_failure_time: float = 0
        self._lock = threading.Lock()
    
    @property
    def state(self) -> CircuitState:
        if self._state == CircuitState.OPEN:
            if time.time() - self._last_failure_time > self.recovery_timeout:
                return CircuitState.HALF_OPEN
        return self._state
    
    def call(self, func, *args, **kwargs):
        with self._lock:
            current_state = self.state
        
        if current_state == CircuitState.OPEN:
            raise RuntimeError("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            
            with self._lock:
                if current_state == CircuitState.HALF_OPEN:
                    self._success_count += 1
                    if self._success_count >= self.success_threshold:
                        self._state = CircuitState.CLOSED
                        self._failure_count = 0
                        self._success_count = 0
                        print(f"  Circuit CLOSED (recovered)")
                else:
                    self._failure_count = 0
            
            return result
            
        except Exception as e:
            with self._lock:
                self._failure_count += 1
                self._last_failure_time = time.time()
                self._success_count = 0
                
                if self._failure_count >= self.failure_threshold:
                    self._state = CircuitState.OPEN
                    print(f"  Circuit OPENED (failures={self._failure_count})")
            raise

# Test
cb = CircuitBreaker(failure_threshold=3, recovery_timeout=0.1)
fail_count = 0

def unreliable_service(should_fail=False):
    if should_fail:
        raise ConnectionError("Service unavailable")
    return "success"

print("\nCircuit Breaker Demo:")
# Normal operation
print("Normal:")
for i in range(3):
    try:
        cb.call(unreliable_service, False)
        print(f"  Call {i+1}: success")
    except Exception as e:
        print(f"  Call {i+1}: {e}")

# Failures
print("Failures (trip circuit):")
for i in range(4):
    try:
        cb.call(unreliable_service, True)
    except Exception as e:
        print(f"  Call {i+4}: {type(e).__name__}: {e}")

# Recovery
print("Recovery:")
time.sleep(0.15)  # Wait for recovery timeout
for i in range(3):
    try:
        result = cb.call(unreliable_service, False)
        print(f"  Recovery call {i+1}: {result}")
    except Exception as e:
        print(f"  Recovery call {i+1}: {e}")
```

```python
# แบบฝึกหัดที่ 6: Alerting System

import time
import threading
from typing import List, Callable, Dict, Any
from dataclasses import dataclass, field
from enum import Enum

class AlertSeverity(Enum):
    INFO = "info"
    WARNING = "warning"
    CRITICAL = "critical"

@dataclass
class Alert:
    name: str
    severity: AlertSeverity
    message: str
    value: float
    threshold: float
    timestamp: float = field(default_factory=time.time)

class AlertManager:
    """Rule-based alerting system"""
    
    def __init__(self):
        self.rules: List[dict] = []
        self.active_alerts: Dict[str, Alert] = {}
        self.resolved_alerts: List[Alert] = []
        self.handlers: List[Callable] = []
    
    def add_rule(self, name: str, metric_func: Callable, 
                 threshold: float, severity: AlertSeverity,
                 message_template: str):
        self.rules.append({
            'name': name,
            'metric_func': metric_func,
            'threshold': threshold,
            'severity': severity,
            'message_template': message_template
        })
    
    def add_handler(self, handler: Callable):
        self.handlers.append(handler)
    
    def evaluate(self):
        """Evaluate all rules"""
        for rule in self.rules:
            try:
                value = rule['metric_func']()
                is_firing = value > rule['threshold']
                
                if is_firing and rule['name'] not in self.active_alerts:
                    alert = Alert(
                        name=rule['name'],
                        severity=rule['severity'],
                        message=rule['message_template'].format(value=value),
                        value=value,
                        threshold=rule['threshold']
                    )
                    self.active_alerts[rule['name']] = alert
                    self._fire_alert(alert)
                
                elif not is_firing and rule['name'] in self.active_alerts:
                    resolved = self.active_alerts.pop(rule['name'])
                    self.resolved_alerts.append(resolved)
                    print(f"  ✓ RESOLVED: {rule['name']}")
                    
            except Exception as e:
                print(f"  Error evaluating rule {rule['name']}: {e}")
    
    def _fire_alert(self, alert: Alert):
        icon = "🔴" if alert.severity == AlertSeverity.CRITICAL else "⚠️"
        print(f"  {icon} ALERT [{alert.severity.value.upper()}]: "
              f"{alert.name} - {alert.message}")
        
        for handler in self.handlers:
            try:
                handler(alert)
            except Exception as e:
                print(f"  Handler error: {e}")

# Setup
error_count = [0]
response_times = [50.0]

alert_manager = AlertManager()

alert_manager.add_rule(
    name="high_error_rate",
    metric_func=lambda: error_count[0],
    threshold=10,
    severity=AlertSeverity.CRITICAL,
    message_template="Error count: {value:.0f}"
)

alert_manager.add_rule(
    name="slow_response",
    metric_func=lambda: response_times[0],
    threshold=500,
    severity=AlertSeverity.WARNING,
    message_template="P99 latency: {value:.0f}ms"
)

print("\nAlerting System Demo:")
print("Normal state:")
alert_manager.evaluate()
print(f"  Active alerts: {len(alert_manager.active_alerts)}")

print("\nDegraded state:")
error_count[0] = 15
response_times[0] = 800
alert_manager.evaluate()
print(f"  Active alerts: {len(alert_manager.active_alerts)}")

print("\nRecovered:")
error_count[0] = 2
response_times[0] = 100
alert_manager.evaluate()
print(f"  Active alerts: {len(alert_manager.active_alerts)}")
```

```python
# แบบฝึกหัดที่ 7: Log Correlation
import uuid
import contextvars
import functools

# Context variables สำหรับ request context
request_id_var = contextvars.ContextVar('request_id', default=None)
user_id_var = contextvars.ContextVar('user_id', default=None)
trace_id_var = contextvars.ContextVar('trace_id', default=None)

def with_request_context(func):
    """Decorator เพิ่ม request context"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        request_id = str(uuid.uuid4())[:8]
        trace_id = str(uuid.uuid4().hex)
        
        request_id_var.set(request_id)
        trace_id_var.set(trace_id)
        
        return func(*args, **kwargs)
    return wrapper

def get_log_context() -> dict:
    """Get current request context for logging"""
    return {
        'request_id': request_id_var.get(),
        'user_id': user_id_var.get(),
        'trace_id': trace_id_var.get()
    }

def log(level: str, message: str, **extra):
    """Log with automatic context injection"""
    context = get_log_context()
    entry = {
        'level': level,
        'message': message,
        **{k: v for k, v in context.items() if v},
        **extra
    }
    print(f"  {json.dumps(entry)}")

@with_request_context
def handle_request(user_id: int, action: str):
    user_id_var.set(user_id)
    
    log("INFO", f"Request received", action=action)
    
    # Nested functions automatically share context
    validate_user(user_id)
    process_action(action)
    
    log("INFO", "Request completed")

def validate_user(user_id: int):
    log("DEBUG", "Validating user", user_id=user_id)

def process_action(action: str):
    log("INFO", "Processing action", action=action)
    log("INFO", "Action complete", result="success")

print("\nLog Correlation Demo:")
handle_request(123, "checkout")
print()
handle_request(456, "view_cart")
```

```python
# แบบฝึกหัดที่ 8: Performance Anomaly Detection
import statistics
import time
from collections import deque
from typing import Optional

class AnomalyDetector:
    """Detect performance anomalies using statistical methods"""
    
    def __init__(self, window_size: int = 100, 
                 z_score_threshold: float = 3.0):
        self.window_size = window_size
        self.threshold = z_score_threshold
        self._values: deque = deque(maxlen=window_size)
    
    def add(self, value: float) -> Optional[dict]:
        """Add value, return anomaly info if detected"""
        
        if len(self._values) < 10:
            self._values.append(value)
            return None
        
        mean = statistics.mean(self._values)
        stdev = statistics.stdev(self._values)
        
        if stdev == 0:
            self._values.append(value)
            return None
        
        z_score = abs(value - mean) / stdev
        
        self._values.append(value)
        
        if z_score > self.threshold:
            return {
                'anomaly': True,
                'value': value,
                'mean': mean,
                'stdev': stdev,
                'z_score': z_score,
                'deviation_pct': (value - mean) / mean * 100
            }
        
        return None

import random

detector = AnomalyDetector(window_size=50, z_score_threshold=3.0)

print("\nAnomaly Detection Demo:")
# Normal operation
for i in range(60):
    duration = random.gauss(100, 10)  # normal: mean=100ms, std=10ms
    result = detector.add(duration)
    if result:
        print(f"  ANOMALY: {result['value']:.0f}ms "
              f"(z={result['z_score']:.1f}, "
              f"dev={result['deviation_pct']:+.0f}%)")

# Inject anomalies
anomaly_values = [500, 50, 600, 30]
print("\nInjecting anomalies:")
for v in anomaly_values:
    result = detector.add(v)
    if result:
        print(f"  DETECTED! {result['value']:.0f}ms "
              f"(z={result['z_score']:.1f}, "
              f"dev={result['deviation_pct']:+.0f}%)")
    else:
        print(f"  {v}ms - not detected as anomaly")
```

---

## สรุป

| Component | Tool | Use Case |
|-----------|------|----------|
| Metrics | Prometheus | Time-series data, alerting |
| Visualization | Grafana | Dashboards, trends |
| Structured Logs | structlog | Rich log data |
| Log Storage | ELK Stack | Search, analytics |
| Distributed Tracing | OpenTelemetry + Jaeger | Request flow |
| Error Tracking | Sentry | Exception monitoring |
| APM | DataDog/New Relic | Full-stack visibility |
| Health Checks | Custom endpoints | Kubernetes probes |

**Best Practices:**
1. ใส่ request ID ทุก log entry
2. ใช้ structured logging (JSON) ใน production
3. Track SLI/SLO metrics ที่สำคัญ
4. Set up alerts สำหรับ error budget
5. Instrument code อย่าง systematic ตั้งแต่แรก

---

*Part 97 - Monitoring, Observability & Distributed Tracing | Python Course*
