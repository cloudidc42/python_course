# Part 45: HTTP & Requests Library

## บทนำ

HTTP (HyperText Transfer Protocol) เป็นโปรโตคอลพื้นฐานของ World Wide Web ที่ใช้สื่อสารระหว่าง clients และ servers Python มี library ที่ทรงพลังสำหรับทำงานกับ HTTP ได้แก่ `requests` (standard) และ `httpx` (modern async alternative)

---

## 1. HTTP Protocol Basics

### HTTP Methods

```
GET    - ดึงข้อมูล (Read)
POST   - ส่งข้อมูลใหม่ (Create)
PUT    - แทนที่ข้อมูลทั้งหมด (Replace)
PATCH  - อัพเดทบางส่วน (Partial Update)
DELETE - ลบข้อมูล (Delete)
HEAD   - เหมือน GET แต่ return header เท่านั้น
OPTIONS - ดู methods ที่ supported
```

```python
"""
HTTP Request Structure:
┌────────────────────────────────────────┐
│ GET /api/users?page=1 HTTP/1.1         │  <- Request Line
│ Host: api.example.com                   │  <- Headers
│ Authorization: Bearer token123          │
│ Accept: application/json               │
│ Content-Type: application/json         │
│                                        │  <- Empty line
│ {"username": "alice"}                  │  <- Body (สำหรับ POST/PUT/PATCH)
└────────────────────────────────────────┘

HTTP Response Structure:
┌────────────────────────────────────────┐
│ HTTP/1.1 200 OK                        │  <- Status Line
│ Content-Type: application/json         │  <- Response Headers
│ Content-Length: 256                    │
│ X-Rate-Limit-Remaining: 99            │
│                                        │  <- Empty line
│ {"id": 1, "username": "alice", ...}    │  <- Response Body
└────────────────────────────────────────┘
"""

# HTTP Status Codes
STATUS_CODES = {
    # 2xx Success
    200: "OK",
    201: "Created",
    204: "No Content",
    
    # 3xx Redirection
    301: "Moved Permanently",
    302: "Found (Temporary Redirect)",
    304: "Not Modified",
    
    # 4xx Client Errors
    400: "Bad Request",
    401: "Unauthorized",
    403: "Forbidden",
    404: "Not Found",
    405: "Method Not Allowed",
    409: "Conflict",
    422: "Unprocessable Entity",
    429: "Too Many Requests",
    
    # 5xx Server Errors
    500: "Internal Server Error",
    502: "Bad Gateway",
    503: "Service Unavailable",
    504: "Gateway Timeout",
}
```

---

## 2. requests Library ครบถ้วน

### ติดตั้ง

```bash
pip install requests
```

### Basic Methods

```python
import requests

# GET - ดึงข้อมูล
response = requests.get('https://httpbin.org/get')
print(f"Status: {response.status_code}")  # 200
print(f"Content-Type: {response.headers['Content-Type']}")

# POST - สร้างข้อมูล
response = requests.post(
    'https://httpbin.org/post',
    json={'name': 'Alice', 'age': 30}
)
print(response.json())

# PUT - แทนที่ข้อมูล
response = requests.put(
    'https://httpbin.org/put',
    json={'id': 1, 'name': 'Alice Updated'}
)

# PATCH - อัพเดทบางส่วน
response = requests.patch(
    'https://httpbin.org/patch',
    json={'name': 'Alice New Name'}
)

# DELETE - ลบข้อมูล
response = requests.delete('https://httpbin.org/delete')

# HEAD - ดู headers เท่านั้น
response = requests.head('https://httpbin.org/get')
print(f"Headers only, no body: {len(response.content)} bytes")

# OPTIONS
response = requests.options('https://httpbin.org/get')
print(f"Allowed: {response.headers.get('Allow', 'N/A')}")
```

---

## 3. Request Headers, Params, Data, JSON

### Headers

```python
import requests

# Custom headers
headers = {
    'User-Agent': 'MyApp/1.0 (Python)',
    'Accept': 'application/json',
    'Accept-Language': 'en-US,en;q=0.9,th;q=0.8',
    'X-Custom-Header': 'custom-value',
}

response = requests.get(
    'https://httpbin.org/headers',
    headers=headers
)
print(response.json())  # ดู headers ที่ส่งไป
```

### Query Parameters

```python
import requests

# วิธีที่ 1: ใส่ใน URL โดยตรง
response = requests.get('https://httpbin.org/get?key=value&page=1')

# วิธีที่ 2: ใช้ params dict (แนะนำ - จะ URL encode ให้อัตโนมัติ)
params = {
    'q': 'python programming',
    'page': 1,
    'limit': 20,
    'sort': 'newest',
    'tags': ['python', 'web'],  # list จะถูกส่งเป็นหลาย params
}

response = requests.get(
    'https://httpbin.org/get',
    params=params
)

print(f"URL: {response.url}")
# https://httpbin.org/get?q=python+programming&page=1&limit=20&sort=newest&tags=python&tags=web

print(response.json()['args'])
```

### Request Body (data vs json)

```python
import requests
import json

# Form data (Content-Type: application/x-www-form-urlencoded)
response = requests.post(
    'https://httpbin.org/post',
    data={
        'username': 'alice',
        'password': 'secret',
        'remember': 'true'
    }
)
print("Form data:", response.json()['form'])

# JSON data (Content-Type: application/json) - ใช้บ่อยกว่า
response = requests.post(
    'https://httpbin.org/post',
    json={
        'user': {
            'name': 'Alice',
            'email': 'alice@example.com',
            'preferences': {'theme': 'dark', 'notifications': True}
        }
    }
)
print("JSON data:", response.json()['json'])

# Raw data
response = requests.post(
    'https://httpbin.org/post',
    data='raw text body',
    headers={'Content-Type': 'text/plain'}
)

# Binary data
with open('image.png', 'rb') as f:
    response = requests.post(
        'https://httpbin.org/post',
        data=f.read(),
        headers={'Content-Type': 'image/png'}
    )
```

---

## 4. Response Handling

```python
import requests

response = requests.get('https://httpbin.org/json')

# Status Code
print(f"Status: {response.status_code}")
print(f"OK: {response.ok}")  # True ถ้า 200-299

# Response Body
print(f"Text: {response.text[:100]}")      # String
print(f"Bytes: {response.content[:20]}")   # Bytes
print(f"JSON: {response.json()}")           # Parsed JSON

# Headers
print(f"Content-Type: {response.headers['Content-Type']}")
print(f"All headers: {dict(response.headers)}")

# Encoding
print(f"Encoding: {response.encoding}")
response.encoding = 'utf-8'  # Override encoding

# URL history (redirects)
print(f"URL: {response.url}")
print(f"History: {response.history}")  # list ของ redirect responses

# Elapsed time
print(f"Time: {response.elapsed.total_seconds():.3f}s")

# Cookies
print(f"Cookies: {dict(response.cookies)}")
```

```python
import requests

# Response เป็น None check pattern
def safe_get(url: str, **kwargs) -> dict:
    """Safe GET request ที่ handle errors"""
    
    try:
        response = requests.get(url, timeout=30, **kwargs)
        response.raise_for_status()  # Raise HTTPError สำหรับ 4xx/5xx
        return {
            'success': True,
            'data': response.json() if 'json' in response.headers.get('Content-Type', '') else response.text,
            'status_code': response.status_code,
            'headers': dict(response.headers)
        }
    except requests.exceptions.HTTPError as e:
        return {
            'success': False,
            'error': str(e),
            'status_code': e.response.status_code if e.response else None
        }
    except requests.exceptions.ConnectionError:
        return {'success': False, 'error': 'Connection failed'}
    except requests.exceptions.Timeout:
        return {'success': False, 'error': 'Request timed out'}
    except requests.exceptions.JSONDecodeError:
        return {'success': False, 'error': 'Invalid JSON response'}

result = safe_get('https://httpbin.org/json')
if result['success']:
    print("Data:", result['data'])
else:
    print("Error:", result['error'])
```

---

## 5. Status Codes

```python
import requests

def handle_response(response: requests.Response) -> dict:
    """Handle response ตาม status code"""
    
    if 200 <= response.status_code < 300:
        # Success
        if response.status_code == 200:
            return {'status': 'ok', 'data': response.json() if response.content else {}}
        elif response.status_code == 201:
            return {'status': 'created', 'data': response.json()}
        elif response.status_code == 204:
            return {'status': 'no_content', 'data': None}
    
    elif response.status_code == 301 or response.status_code == 302:
        # requests จะ follow redirect อัตโนมัติ (ไม่น่าเจอที่นี่)
        return {'status': 'redirect', 'location': response.headers.get('Location')}
    
    elif response.status_code == 400:
        raise ValueError(f"Bad request: {response.text}")
    
    elif response.status_code == 401:
        raise PermissionError("Authentication required")
    
    elif response.status_code == 403:
        raise PermissionError("Access forbidden")
    
    elif response.status_code == 404:
        raise FileNotFoundError(f"Resource not found: {response.url}")
    
    elif response.status_code == 409:
        raise ValueError(f"Conflict: {response.text}")
    
    elif response.status_code == 422:
        errors = response.json().get('detail', response.text)
        raise ValueError(f"Validation error: {errors}")
    
    elif response.status_code == 429:
        retry_after = response.headers.get('Retry-After', '60')
        raise Exception(f"Rate limited. Retry after {retry_after}s")
    
    elif response.status_code >= 500:
        raise RuntimeError(f"Server error {response.status_code}: {response.text}")
    
    response.raise_for_status()  # สำหรับ status codes อื่นๆ

# ตัวอย่างการใช้งาน
test_urls = [
    'https://httpbin.org/status/200',
    'https://httpbin.org/status/201',
    'https://httpbin.org/status/404',
    'https://httpbin.org/status/500',
]

for url in test_urls:
    try:
        response = requests.get(url, timeout=10)
        result = handle_response(response)
        print(f"{url.split('/')[-1]}: {result}")
    except Exception as e:
        print(f"{url.split('/')[-1]}: {type(e).__name__}: {e}")
```

---

## 6. Authentication

### Basic Authentication

```python
import requests
from requests.auth import HTTPBasicAuth, HTTPDigestAuth

# Basic Auth
response = requests.get(
    'https://httpbin.org/basic-auth/user/pass',
    auth=HTTPBasicAuth('user', 'pass')
)
print(f"Basic Auth: {response.status_code}")  # 200

# Shorthand
response = requests.get(
    'https://httpbin.org/basic-auth/user/pass',
    auth=('user', 'pass')  # tuple ก็ได้
)

# Digest Auth
response = requests.get(
    'https://httpbin.org/digest-auth/auth/user/pass',
    auth=HTTPDigestAuth('user', 'pass')
)
```

### Bearer Token (API Key)

```python
import requests

# Bearer Token (JWT)
def make_authenticated_request(
    url: str,
    token: str,
    method: str = 'GET',
    **kwargs
) -> requests.Response:
    
    headers = kwargs.pop('headers', {})
    headers['Authorization'] = f'Bearer {token}'
    
    return requests.request(method, url, headers=headers, **kwargs)

response = make_authenticated_request(
    'https://httpbin.org/bearer',
    token='your-jwt-token-here'
)
print(f"Bearer: {response.status_code}")

# API Key ใน header
headers = {
    'X-API-Key': 'your-api-key-here',
    'Authorization': 'ApiKey your-api-key-here',
}

# API Key ใน query params
params = {
    'api_key': 'your-api-key',
    'format': 'json'
}

response = requests.get(
    'https://api.example.com/data',
    headers={'X-API-Key': 'your-key'},
    params={'limit': 10}
)
```

### OAuth2 Authentication

```python
import requests
from typing import Optional

class OAuth2Client:
    """Simple OAuth2 client"""
    
    def __init__(
        self,
        client_id: str,
        client_secret: str,
        token_url: str,
        base_url: str
    ):
        self.client_id = client_id
        self.client_secret = client_secret
        self.token_url = token_url
        self.base_url = base_url
        self._access_token: Optional[str] = None
        self._token_expires_at: float = 0
    
    def get_token(self) -> str:
        """ดึง access token (client credentials flow)"""
        
        import time
        
        # ตรวจสอบ token ยังใช้ได้หรือเปล่า
        if self._access_token and time.time() < self._token_expires_at:
            return self._access_token
        
        # ขอ token ใหม่
        response = requests.post(
            self.token_url,
            data={
                'grant_type': 'client_credentials',
                'client_id': self.client_id,
                'client_secret': self.client_secret,
            }
        )
        response.raise_for_status()
        
        token_data = response.json()
        self._access_token = token_data['access_token']
        
        # คำนวณ expiration (ลบ 60 วินาทีเพื่อ safety margin)
        expires_in = token_data.get('expires_in', 3600)
        self._token_expires_at = time.time() + expires_in - 60
        
        return self._access_token
    
    def request(self, method: str, path: str, **kwargs) -> requests.Response:
        """Make authenticated request"""
        
        token = self.get_token()
        headers = kwargs.pop('headers', {})
        headers['Authorization'] = f'Bearer {token}'
        
        url = f"{self.base_url}{path}"
        return requests.request(method, url, headers=headers, **kwargs)
    
    def get(self, path: str, **kwargs) -> requests.Response:
        return self.request('GET', path, **kwargs)
    
    def post(self, path: str, **kwargs) -> requests.Response:
        return self.request('POST', path, **kwargs)

# ใช้งาน
# client = OAuth2Client(
#     client_id="your_client_id",
#     client_secret="your_client_secret",
#     token_url="https://auth.example.com/oauth/token",
#     base_url="https://api.example.com"
# )
# response = client.get("/users")
```

---

## 7. Session Management

Session ช่วย:
- เก็บ cookies อัตโนมัติ
- Reuse TCP connections (performance)
- Share headers/auth ระหว่าง requests

```python
import requests

# ใช้ Session สำหรับ multiple requests ไปยัง domain เดียวกัน
with requests.Session() as session:
    
    # Set default headers สำหรับทุก request
    session.headers.update({
        'User-Agent': 'MyApp/1.0',
        'Accept': 'application/json',
    })
    
    # Login - cookies จะถูกเก็บอัตโนมัติ
    login_response = session.post(
        'https://httpbin.org/cookies/set?session_token=abc123',
    )
    
    print(f"Cookies after login: {dict(session.cookies)}")
    
    # Subsequent requests ใช้ cookies อัตโนมัติ
    profile_response = session.get('https://httpbin.org/cookies')
    print(f"Cookies sent: {profile_response.json()}")
    
    # Session จะ reuse TCP connection (keep-alive)
    for i in range(3):
        r = session.get(f'https://httpbin.org/get?req={i}')
        print(f"Request {i+1}: {r.status_code}")
```

```python
import requests
from typing import Optional

class APIClient:
    """Reusable API client ด้วย Session"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url.rstrip('/')
        self._session = requests.Session()
        self._session.headers.update({
            'Accept': 'application/json',
            'Content-Type': 'application/json',
        })
    
    def authenticate(self, username: str, password: str) -> bool:
        """Login และเก็บ session"""
        
        response = self._session.post(
            f"{self.base_url}/auth/login",
            json={'username': username, 'password': password}
        )
        
        if response.status_code == 200:
            token = response.json().get('token')
            if token:
                self._session.headers['Authorization'] = f'Bearer {token}'
            return True
        
        return False
    
    def get(self, path: str, **kwargs) -> requests.Response:
        return self._session.get(f"{self.base_url}{path}", **kwargs)
    
    def post(self, path: str, **kwargs) -> requests.Response:
        return self._session.post(f"{self.base_url}{path}", **kwargs)
    
    def put(self, path: str, **kwargs) -> requests.Response:
        return self._session.put(f"{self.base_url}{path}", **kwargs)
    
    def delete(self, path: str, **kwargs) -> requests.Response:
        return self._session.delete(f"{self.base_url}{path}", **kwargs)
    
    def close(self):
        self._session.close()
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()

# ใช้งาน
with APIClient("https://jsonplaceholder.typicode.com") as client:
    response = client.get("/posts/1")
    print(response.json())
    
    response = client.get("/posts", params={"userId": 1})
    posts = response.json()
    print(f"Posts by user 1: {len(posts)}")
```

---

## 8. Timeout และ Retry

### Timeout

```python
import requests
from requests.exceptions import Timeout, ConnectionError

# Simple timeout (ใช้ทั้ง connect และ read)
try:
    response = requests.get('https://httpbin.org/delay/3', timeout=2)
except Timeout:
    print("Request timed out!")

# Separate connect and read timeouts
try:
    response = requests.get(
        'https://httpbin.org/delay/5',
        timeout=(3, 10)  # (connect timeout, read timeout) in seconds
    )
except Timeout as e:
    print(f"Timeout: {e}")
```

### Retry with urllib3

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_retry_session(
    total_retries: int = 3,
    backoff_factor: float = 0.5,
    status_forcelist: tuple = (429, 500, 502, 503, 504)
) -> requests.Session:
    """สร้าง Session ที่มี automatic retry"""
    
    session = requests.Session()
    
    retry_strategy = Retry(
        total=total_retries,
        backoff_factor=backoff_factor,  # delay = backoff_factor * (2 ** (attempt - 1))
        status_forcelist=status_forcelist,
        allowed_methods=["GET", "POST", "PUT", "DELETE"],  # methods ที่ retry ได้
        raise_on_status=True  # raise exception เมื่อ retry หมด
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount('http://', adapter)
    session.mount('https://', adapter)
    
    return session

# ใช้งาน
session = create_retry_session(total_retries=3, backoff_factor=1.0)

try:
    response = session.get('https://httpbin.org/status/503', timeout=30)
    print(f"Success: {response.status_code}")
except requests.exceptions.RetryError as e:
    print(f"Max retries exceeded: {e}")
finally:
    session.close()
```

### Manual Retry with Exponential Backoff

```python
import requests
import time
import random
from typing import Optional, TypeVar, Callable

def retry_with_backoff(
    func: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    jitter: bool = True,
    exceptions: tuple = (requests.exceptions.RequestException,)
):
    """Decorator สำหรับ retry ด้วย exponential backoff"""
    
    def wrapper(*args, **kwargs):
        for attempt in range(max_retries + 1):
            try:
                return func(*args, **kwargs)
            
            except exceptions as e:
                if attempt == max_retries:
                    raise
                
                # คำนวณ delay ด้วย exponential backoff
                delay = min(base_delay * (2 ** attempt), max_delay)
                
                if jitter:
                    # เพิ่ม random jitter เพื่อป้องกัน thundering herd
                    delay = delay * (0.5 + random.random() * 0.5)
                
                print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay:.2f}s...")
                time.sleep(delay)
        
        return None
    
    return wrapper

@retry_with_backoff(max_retries=3, base_delay=0.5)
def fetch_data(url: str) -> dict:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()

# ทดสอบ
try:
    data = fetch_data('https://httpbin.org/json')
    print("Success:", data)
except requests.exceptions.RequestException as e:
    print(f"Failed after retries: {e}")
```

---

## 9. SSL/TLS

```python
import requests

# ตรวจสอบ SSL certificate (default: True)
response = requests.get('https://httpbin.org/get', verify=True)

# ใช้ custom CA bundle
response = requests.get(
    'https://your-internal-api.com/endpoint',
    verify='/path/to/ca-bundle.crt'
)

# ปิด SSL verification (ไม่แนะนำ - ใช้เฉพาะ development)
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

response = requests.get(
    'https://self-signed.example.com',
    verify=False  # DANGEROUS in production!
)

# Client certificate authentication (mTLS)
response = requests.get(
    'https://api.example.com/secure',
    cert=('/path/to/client.crt', '/path/to/client.key')
)

# ตรวจสอบ certificate info
import ssl
import socket

def get_cert_info(hostname: str, port: int = 443) -> dict:
    """ดูข้อมูล SSL certificate"""
    
    context = ssl.create_default_context()
    
    with socket.create_connection((hostname, port)) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            cert = ssock.getpeercert()
            return {
                'subject': dict(x[0] for x in cert['subject']),
                'issuer': dict(x[0] for x in cert['issuer']),
                'valid_from': cert['notBefore'],
                'valid_to': cert['notAfter'],
                'version': cert['version'],
            }

# cert_info = get_cert_info('httpbin.org')
# print(cert_info)
```

---

## 10. Proxies

```python
import requests

# HTTP Proxy
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'https://proxy.example.com:8080',
}

response = requests.get(
    'https://httpbin.org/ip',
    proxies=proxies
)
print(f"IP through proxy: {response.json()['origin']}")

# SOCKS Proxy (ต้อง install requests[socks])
# pip install requests[socks]
socks_proxies = {
    'http': 'socks5://localhost:1080',
    'https': 'socks5://localhost:1080',
}

# Proxy กับ authentication
auth_proxy = {
    'http': 'http://username:password@proxy.example.com:8080',
    'https': 'http://username:password@proxy.example.com:8080',
}

# Environment variables proxies
# requests จะอ่าน HTTP_PROXY, HTTPS_PROXY, NO_PROXY จาก environment
import os
os.environ['HTTP_PROXY'] = 'http://proxy.example.com:8080'
os.environ['HTTPS_PROXY'] = 'https://proxy.example.com:8080'
os.environ['NO_PROXY'] = 'localhost,127.0.0.1,.internal.company.com'

# ปิด proxy สำหรับ specific requests
response = requests.get('https://httpbin.org/ip', proxies={'no_proxy': 'httpbin.org'})
```

---

## 11. Streaming Responses

```python
import requests
import shutil

# Streaming download - ไม่โหลดทั้งหมดเข้า memory
def download_file(url: str, dest_path: str, chunk_size: int = 8192) -> bool:
    """Download file ด้วย streaming"""
    
    try:
        with requests.get(url, stream=True, timeout=60) as response:
            response.raise_for_status()
            
            total_size = int(response.headers.get('content-length', 0))
            downloaded = 0
            
            with open(dest_path, 'wb') as f:
                for chunk in response.iter_content(chunk_size=chunk_size):
                    if chunk:  # กรอง keep-alive chunks
                        f.write(chunk)
                        downloaded += len(chunk)
                        
                        if total_size:
                            percent = (downloaded / total_size) * 100
                            print(f"\rProgress: {percent:.1f}% ({downloaded}/{total_size})", end='')
            
            print()
            print(f"Downloaded: {dest_path} ({downloaded} bytes)")
            return True
    
    except requests.exceptions.RequestException as e:
        print(f"Download failed: {e}")
        return False

# ดาวน์โหลดไฟล์
# download_file(
#     'https://httpbin.org/stream-bytes/10000',
#     '/tmp/test_download.bin'
# )

# Streaming JSON (Line-delimited JSON)
def stream_json_lines(url: str):
    """Stream JSONL (JSON Lines) format"""
    
    with requests.get(url, stream=True) as response:
        for line in response.iter_lines():
            if line:
                import json
                data = json.loads(line)
                yield data

# Streaming response สำหรับ real-time data
def stream_server_sent_events(url: str):
    """Stream Server-Sent Events (SSE)"""
    
    headers = {
        'Accept': 'text/event-stream',
        'Cache-Control': 'no-cache',
    }
    
    with requests.get(url, stream=True, headers=headers) as response:
        for line in response.iter_lines():
            if line:
                line = line.decode('utf-8')
                if line.startswith('data: '):
                    data = line[6:]  # ตัด 'data: ' ออก
                    yield data
```

---

## 12. File Uploads

```python
import requests
from pathlib import Path

# Upload single file
def upload_file(url: str, file_path: str, field_name: str = 'file') -> dict:
    """Upload file เดียว"""
    
    with open(file_path, 'rb') as f:
        filename = Path(file_path).name
        
        files = {
            field_name: (filename, f, 'application/octet-stream')
        }
        
        response = requests.post(url, files=files)
        response.raise_for_status()
        return response.json()

# Upload multiple files
def upload_multiple(url: str, file_paths: list) -> dict:
    """Upload หลายไฟล์พร้อมกัน"""
    
    files = []
    file_handles = []
    
    try:
        for path in file_paths:
            f = open(path, 'rb')
            file_handles.append(f)
            files.append(('files', (Path(path).name, f)))
        
        response = requests.post(url, files=files)
        response.raise_for_status()
        return response.json()
    
    finally:
        for f in file_handles:
            f.close()

# Upload กับ additional form data
def upload_with_metadata(url: str, file_path: str, metadata: dict) -> dict:
    """Upload file พร้อม metadata"""
    
    with open(file_path, 'rb') as f:
        files = {'file': (Path(file_path).name, f)}
        
        # Form data + file
        response = requests.post(
            url,
            files=files,
            data=metadata  # additional form fields
        )
        return response.json()

# Upload เป็น multipart form data
import io

def upload_text_as_file(url: str, content: str, filename: str) -> dict:
    """Upload text content เป็น file"""
    
    file_content = content.encode('utf-8')
    
    files = {
        'file': (
            filename,
            io.BytesIO(file_content),
            'text/plain; charset=utf-8'
        )
    }
    
    response = requests.post(url, files=files)
    return response.json()

# ทดสอบ
# result = upload_text_as_file(
#     'https://httpbin.org/post',
#     'Hello, World!',
#     'hello.txt'
# )
# print("Files received:", result.get('files'))
```

---

## 13. httpx (Modern Alternative)

httpx รองรับ HTTP/2, async/await, และมี API คล้าย requests

### ติดตั้ง

```bash
pip install httpx
```

### httpx Synchronous

```python
import httpx

# API คล้าย requests มาก
response = httpx.get('https://httpbin.org/json')
print(response.status_code)  # 200
print(response.json())

# POST
response = httpx.post(
    'https://httpbin.org/post',
    json={'name': 'Alice'}
)

# Client (เหมือน Session)
with httpx.Client(
    base_url='https://httpbin.org',
    headers={'User-Agent': 'httpx-example/1.0'},
    timeout=30.0,
    follow_redirects=True
) as client:
    r1 = client.get('/get')
    r2 = client.post('/post', json={'data': 'hello'})
    print(r1.json())
    print(r2.json())
```

### httpx Async

```python
import httpx
import asyncio

async def fetch_all(urls: list) -> list:
    """Fetch หลาย URLs พร้อมกัน"""
    
    async with httpx.AsyncClient(timeout=30.0) as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks, return_exceptions=True)
        
        results = []
        for url, response in zip(urls, responses):
            if isinstance(response, Exception):
                results.append({'url': url, 'error': str(response)})
            else:
                results.append({
                    'url': url,
                    'status': response.status_code,
                    'data': response.json() if response.headers.get('content-type', '').startswith('application/json') else response.text[:100]
                })
        
        return results

# ใช้งาน
async def main():
    urls = [
        'https://httpbin.org/json',
        'https://httpbin.org/uuid',
        'https://httpbin.org/ip',
    ]
    
    results = await fetch_all(urls)
    for result in results:
        print(f"URL: {result['url']}")
        print(f"  Status: {result.get('status')}")
        print()

asyncio.run(main())

# Async Stream
async def download_streaming(url: str, dest: str):
    """Async streaming download"""
    
    async with httpx.AsyncClient() as client:
        async with client.stream('GET', url) as response:
            response.raise_for_status()
            
            with open(dest, 'wb') as f:
                async for chunk in response.aiter_bytes(chunk_size=8192):
                    f.write(chunk)
```

### httpx vs requests เปรียบเทียบ

```python
"""
requests vs httpx:

Feature                  | requests | httpx
-------------------------|----------|--------
Sync support             |    ✓     |   ✓
Async support            |    ✗     |   ✓
HTTP/2                   |    ✗     |   ✓ (pip install httpx[http2])
Type hints               |   ~~     |   ✓
Connection pooling       |    ✓     |   ✓
Cookies                  |    ✓     |   ✓
Redirects                |    ✓     |   ✓
Streaming                |    ✓     |   ✓
File uploads             |    ✓     |   ✓
Proxy support            |    ✓     |   ✓
Timeout                  |    ✓     |   ✓ (better defaults)
Retry (built-in)         |    ✓     |   ✗ (ใช้ tenacity แทน)
Ecosystem/Community      |   huge   |  growing
Mature                   |   ✓✓    |   ✓

เมื่อไหรใช้ httpx:
- ต้องการ async support
- ต้องการ HTTP/2
- FastAPI / asyncio ecosystem
- New projects

เมื่อไหรใช้ requests:
- Existing codebase
- ต้องการ ecosystem ที่ใหญ่กว่า (plugins, auth libraries)
- Synchronous code ที่ไม่ต้องการ async
"""
```

---

## 14. REST API Consumption Patterns

### Resource-based API Client

```python
import requests
from typing import Optional, Dict, Any, List
from dataclasses import dataclass
import json

@dataclass
class APIError(Exception):
    status_code: int
    message: str
    details: Optional[dict] = None
    
    def __str__(self):
        return f"API Error {self.status_code}: {self.message}"

class RESTClient:
    """Generic REST API Client"""
    
    def __init__(
        self,
        base_url: str,
        api_key: Optional[str] = None,
        timeout: float = 30.0,
        verify_ssl: bool = True
    ):
        self.base_url = base_url.rstrip('/')
        self.timeout = timeout
        
        self._session = requests.Session()
        self._session.verify = verify_ssl
        
        # ตั้งค่า default headers
        self._session.headers.update({
            'Content-Type': 'application/json',
            'Accept': 'application/json',
        })
        
        if api_key:
            self._session.headers['X-API-Key'] = api_key
    
    def _request(
        self,
        method: str,
        path: str,
        **kwargs
    ) -> requests.Response:
        """Internal request method"""
        
        url = f"{self.base_url}{path}"
        
        kwargs.setdefault('timeout', self.timeout)
        
        try:
            response = self._session.request(method, url, **kwargs)
            self._handle_errors(response)
            return response
        
        except requests.exceptions.ConnectionError as e:
            raise APIError(0, f"Connection failed: {e}")
        except requests.exceptions.Timeout:
            raise APIError(0, f"Request timed out after {self.timeout}s")
    
    def _handle_errors(self, response: requests.Response):
        """Handle HTTP errors"""
        
        if response.ok:
            return
        
        try:
            error_data = response.json()
        except:
            error_data = {'message': response.text}
        
        message = (
            error_data.get('message') or
            error_data.get('error') or
            error_data.get('detail') or
            f"HTTP {response.status_code}"
        )
        
        raise APIError(
            status_code=response.status_code,
            message=message,
            details=error_data
        )
    
    def get(self, path: str, params: dict = None) -> Any:
        """GET request"""
        response = self._request('GET', path, params=params)
        return response.json() if response.content else None
    
    def post(self, path: str, data: Any = None) -> Any:
        """POST request"""
        response = self._request('POST', path, json=data)
        return response.json() if response.content else None
    
    def put(self, path: str, data: Any = None) -> Any:
        """PUT request"""
        response = self._request('PUT', path, json=data)
        return response.json() if response.content else None
    
    def patch(self, path: str, data: Any = None) -> Any:
        """PATCH request"""
        response = self._request('PATCH', path, json=data)
        return response.json() if response.content else None
    
    def delete(self, path: str) -> bool:
        """DELETE request"""
        response = self._request('DELETE', path)
        return response.status_code in (200, 204)
    
    def close(self):
        self._session.close()
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()

class JSONPlaceholderClient(RESTClient):
    """Client สำหรับ JSONPlaceholder API"""
    
    def __init__(self):
        super().__init__("https://jsonplaceholder.typicode.com")
    
    # Posts
    def get_posts(self, user_id: int = None) -> List[dict]:
        params = {'userId': user_id} if user_id else None
        return self.get('/posts', params=params)
    
    def get_post(self, post_id: int) -> dict:
        return self.get(f'/posts/{post_id}')
    
    def create_post(self, title: str, body: str, user_id: int) -> dict:
        return self.post('/posts', {
            'title': title,
            'body': body,
            'userId': user_id
        })
    
    def update_post(self, post_id: int, title: str = None, body: str = None) -> dict:
        data = {}
        if title: data['title'] = title
        if body: data['body'] = body
        return self.patch(f'/posts/{post_id}', data)
    
    def delete_post(self, post_id: int) -> bool:
        return self.delete(f'/posts/{post_id}')
    
    # Users
    def get_users(self) -> List[dict]:
        return self.get('/users')
    
    def get_user_posts(self, user_id: int) -> List[dict]:
        return self.get('/posts', params={'userId': user_id})

# ใช้งาน
with JSONPlaceholderClient() as client:
    # ดึง posts ทั้งหมด
    posts = client.get_posts()
    print(f"Total posts: {len(posts)}")
    
    # ดึง post เดียว
    post = client.get_post(1)
    print(f"Post 1: {post['title']}")
    
    # สร้าง post ใหม่
    new_post = client.create_post(
        title="My New Post",
        body="This is my post content",
        user_id=1
    )
    print(f"Created post with id: {new_post['id']}")
    
    # อัพเดท
    updated = client.update_post(1, title="Updated Title")
    print(f"Updated title: {updated['title']}")
    
    # ลบ
    deleted = client.delete_post(1)
    print(f"Deleted: {deleted}")
```

---

## 15. Error Handling ที่ครอบคลุม

```python
import requests
from requests.exceptions import (
    HTTPError,
    ConnectionError,
    Timeout,
    RequestException,
    SSLError,
    ProxyError,
    InvalidURL,
    TooManyRedirects
)
import logging
from typing import Optional, Any

logger = logging.getLogger(__name__)

class RequestHandler:
    """Comprehensive request handler"""
    
    def __init__(self):
        self._session = requests.Session()
    
    def execute(
        self,
        method: str,
        url: str,
        max_retries: int = 3,
        **kwargs
    ) -> Optional[requests.Response]:
        """Execute request ด้วย comprehensive error handling"""
        
        import time
        
        for attempt in range(max_retries):
            try:
                response = self._session.request(method, url, **kwargs)
                
                # Log request info
                logger.debug(f"{method} {url} -> {response.status_code} ({response.elapsed.total_seconds():.3f}s)")
                
                # Handle specific status codes
                if response.status_code == 429:
                    retry_after = int(response.headers.get('Retry-After', 60))
                    logger.warning(f"Rate limited. Waiting {retry_after}s...")
                    time.sleep(retry_after)
                    continue
                
                elif response.status_code >= 500:
                    if attempt < max_retries - 1:
                        delay = 2 ** attempt
                        logger.warning(f"Server error {response.status_code}. Retrying in {delay}s...")
                        time.sleep(delay)
                        continue
                
                response.raise_for_status()
                return response
            
            except SSLError as e:
                logger.error(f"SSL Error: {e}")
                raise  # ไม่ retry SSL errors
            
            except ProxyError as e:
                logger.error(f"Proxy Error: {e}")
                raise  # ไม่ retry proxy errors
            
            except InvalidURL as e:
                logger.error(f"Invalid URL: {e}")
                raise  # ไม่ retry invalid URLs
            
            except TooManyRedirects as e:
                logger.error(f"Too many redirects: {e}")
                raise
            
            except ConnectionError as e:
                if attempt < max_retries - 1:
                    delay = 2 ** attempt
                    logger.warning(f"Connection error. Retrying in {delay}s: {e}")
                    time.sleep(delay)
                else:
                    logger.error(f"Connection failed after {max_retries} attempts: {e}")
                    raise
            
            except Timeout as e:
                if attempt < max_retries - 1:
                    logger.warning(f"Timeout on attempt {attempt + 1}")
                else:
                    logger.error(f"Request timed out after {max_retries} attempts")
                    raise
            
            except HTTPError as e:
                # 4xx errors ไม่ retry (ยกเว้น 429)
                if e.response.status_code < 500:
                    logger.error(f"HTTP Error {e.response.status_code}: {e}")
                    raise
                
                if attempt < max_retries - 1:
                    delay = 2 ** attempt
                    logger.warning(f"HTTP {e.response.status_code}. Retrying in {delay}s...")
                    time.sleep(delay)
                else:
                    raise
        
        return None

# Context manager สำหรับ cleanup
class ManagedRequest:
    """Context manager สำหรับ HTTP requests"""
    
    def __init__(self, url: str, method: str = 'GET', **kwargs):
        self.url = url
        self.method = method
        self.kwargs = kwargs
        self._response = None
    
    def __enter__(self) -> requests.Response:
        self._response = requests.request(
            self.method,
            self.url,
            stream=True,  # สำหรับ streaming
            **self.kwargs
        )
        self._response.raise_for_status()
        return self._response
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self._response:
            self._response.close()

# ใช้งาน
with ManagedRequest('https://httpbin.org/stream-bytes/1000', timeout=10) as r:
    content = b''
    for chunk in r.iter_content(chunk_size=256):
        content += chunk
    print(f"Downloaded: {len(content)} bytes")
```

---

## 16. Advanced Patterns

### Pagination Helper

```python
import requests
from typing import Iterator, Dict, Any, Optional, Callable

def paginate(
    url: str,
    session: Optional[requests.Session] = None,
    page_param: str = 'page',
    start_page: int = 1,
    limit_param: str = 'limit',
    limit: int = 100,
    items_key: str = 'items',
    has_more_key: Optional[str] = None,
    next_url_key: Optional[str] = None,
    **request_kwargs
) -> Iterator[Any]:
    """Generic pagination helper"""
    
    req = session or requests
    current_url = url
    page = start_page
    
    while current_url:
        params = {**request_kwargs.pop('params', {})}
        
        if page_param:
            params[page_param] = page
        if limit_param:
            params[limit_param] = limit
        
        response = req.get(current_url, params=params, **request_kwargs)
        response.raise_for_status()
        data = response.json()
        
        items = data.get(items_key, data) if isinstance(data, dict) else data
        
        if not items:
            break
        
        yield from items
        
        # Determine next page
        if next_url_key and isinstance(data, dict):
            current_url = data.get(next_url_key)
        elif has_more_key and isinstance(data, dict):
            if not data.get(has_more_key):
                break
            page += 1
        elif isinstance(data, list):
            if len(items) < limit:
                break
            page += 1
        else:
            break

# ใช้งาน
# for post in paginate(
#     'https://jsonplaceholder.typicode.com/posts',
#     items_key=None,  # response เป็น list โดยตรง
#     limit=10
# ):
#     print(post['id'], post['title'])
```

### Async Batch Requests

```python
import httpx
import asyncio
from typing import List, Dict, Any

async def batch_requests(
    requests_config: List[Dict[str, Any]],
    concurrency: int = 10,
    timeout: float = 30.0
) -> List[Dict]:
    """ส่ง batch requests แบบ async"""
    
    semaphore = asyncio.Semaphore(concurrency)
    results = []
    
    async def fetch_one(config: Dict) -> Dict:
        async with semaphore:
            try:
                async with httpx.AsyncClient(timeout=timeout) as client:
                    method = config.get('method', 'GET')
                    url = config['url']
                    kwargs = {k: v for k, v in config.items() if k not in ('method', 'url')}
                    
                    response = await client.request(method, url, **kwargs)
                    response.raise_for_status()
                    
                    return {
                        'url': url,
                        'status': response.status_code,
                        'data': response.json() if 'json' in response.headers.get('content-type', '') else response.text,
                        'error': None
                    }
            
            except Exception as e:
                return {
                    'url': config.get('url', ''),
                    'status': None,
                    'data': None,
                    'error': str(e)
                }
    
    tasks = [fetch_one(config) for config in requests_config]
    results = await asyncio.gather(*tasks)
    return list(results)

# ใช้งาน
async def demo_batch():
    requests_to_make = [
        {'url': 'https://httpbin.org/json'},
        {'url': 'https://httpbin.org/uuid'},
        {'url': 'https://httpbin.org/ip'},
        {'method': 'POST', 'url': 'https://httpbin.org/post', 'json': {'test': True}},
    ]
    
    results = await batch_requests(requests_to_make, concurrency=4)
    
    for result in results:
        if result['error']:
            print(f"ERROR: {result['url']}: {result['error']}")
        else:
            print(f"OK: {result['url']}: {result['status']}")

asyncio.run(demo_batch())
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Basic REST API calls

```python
import requests

BASE_URL = "https://jsonplaceholder.typicode.com"

# CRUD operations

# 1. ดึง posts ทั้งหมด
def get_all_posts() -> list:
    response = requests.get(f"{BASE_URL}/posts")
    return response.json()

# 2. ดึง post ตาม ID
def get_post(post_id: int) -> dict:
    response = requests.get(f"{BASE_URL}/posts/{post_id}")
    if response.status_code == 404:
        return None
    return response.json()

# 3. สร้าง post ใหม่
def create_post(title: str, body: str, user_id: int) -> dict:
    response = requests.post(
        f"{BASE_URL}/posts",
        json={"title": title, "body": body, "userId": user_id}
    )
    response.raise_for_status()
    return response.json()

# 4. อัพเดท post
def update_post(post_id: int, **updates) -> dict:
    response = requests.patch(
        f"{BASE_URL}/posts/{post_id}",
        json=updates
    )
    return response.json()

# 5. ลบ post
def delete_post(post_id: int) -> bool:
    response = requests.delete(f"{BASE_URL}/posts/{post_id}")
    return response.status_code == 200

# Test
posts = get_all_posts()
print(f"Total posts: {len(posts)}")

post = get_post(1)
print(f"Post 1: {post['title']}")

new = create_post("Test Post", "Content here", user_id=1)
print(f"Created: {new}")

updated = update_post(1, title="New Title")
print(f"Updated title: {updated['title']}")

print(f"Deleted: {delete_post(1)}")
```

### แบบฝึกหัดที่ 2: Authentication Patterns

```python
import requests
from requests.auth import HTTPBasicAuth

# 1. Basic Auth
def test_basic_auth(username: str, password: str) -> bool:
    response = requests.get(
        f"https://httpbin.org/basic-auth/{username}/{password}",
        auth=HTTPBasicAuth(username, password)
    )
    return response.status_code == 200

# 2. Bearer Token
class TokenAuth(requests.auth.AuthBase):
    """Custom auth ด้วย Bearer token"""
    
    def __init__(self, token: str):
        self.token = token
    
    def __call__(self, r: requests.PreparedRequest) -> requests.PreparedRequest:
        r.headers['Authorization'] = f'Bearer {self.token}'
        return r

# ใช้ custom auth
response = requests.get(
    'https://httpbin.org/bearer',
    auth=TokenAuth('my-secret-token')
)
print(f"Bearer auth: {response.status_code}")

# 3. API Key
class APIKeyAuth(requests.auth.AuthBase):
    def __init__(self, api_key: str, header_name: str = 'X-API-Key'):
        self.api_key = api_key
        self.header_name = header_name
    
    def __call__(self, r):
        r.headers[self.header_name] = self.api_key
        return r

print(test_basic_auth('user', 'pass'))
```

### แบบฝึกหัดที่ 3: Session กับ Cookies

```python
import requests

with requests.Session() as session:
    # Set cookies สำหรับทดสอบ
    session.get('https://httpbin.org/cookies/set?session_id=abc123&user=alice')
    
    print("Session cookies:", dict(session.cookies))
    
    # Request ถัดไปส่ง cookies อัตโนมัติ
    r = session.get('https://httpbin.org/cookies')
    print("Cookies sent:", r.json())
    
    # ลบ cookie
    session.cookies.clear_session_cookies()
    print("After clear:", dict(session.cookies))
```

### แบบฝึกหัดที่ 4: File Upload

```python
import requests
import io

# Upload text as file
text_content = "Hello, this is a test file!\nLine 2\nLine 3"

files = {
    'file': ('test.txt', io.BytesIO(text_content.encode()), 'text/plain')
}

data = {
    'description': 'Test upload',
    'category': 'test'
}

response = requests.post(
    'https://httpbin.org/post',
    files=files,
    data=data
)

result = response.json()
print(f"Files: {list(result.get('files', {}).keys())}")
print(f"Form data: {result.get('form', {})}")
```

### แบบฝึกหัดที่ 5: Error Handling ครบถ้วน

```python
import requests
from requests.exceptions import HTTPError, ConnectionError, Timeout

def robust_get(url: str, max_retries: int = 3) -> dict:
    """GET request ที่ handle errors ครบถ้วน"""
    
    import time
    
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return {'success': True, 'data': response.json()}
        
        except HTTPError as e:
            code = e.response.status_code
            
            if code == 404:
                return {'success': False, 'error': 'Not found', 'code': 404}
            elif code == 401:
                return {'success': False, 'error': 'Unauthorized', 'code': 401}
            elif code >= 500:
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)
                    continue
                return {'success': False, 'error': f'Server error: {code}', 'code': code}
            else:
                return {'success': False, 'error': str(e), 'code': code}
        
        except Timeout:
            if attempt < max_retries - 1:
                time.sleep(1)
                continue
            return {'success': False, 'error': 'Timeout', 'code': None}
        
        except ConnectionError as e:
            if attempt < max_retries - 1:
                time.sleep(2)
                continue
            return {'success': False, 'error': f'Connection error: {e}', 'code': None}

# Test
urls = [
    'https://httpbin.org/json',
    'https://httpbin.org/status/404',
    'https://httpbin.org/status/500',
    'https://nonexistent.domain.xyz',
]

for url in urls:
    result = robust_get(url, max_retries=2)
    print(f"{url.split('/')[-1]}: success={result['success']}, error={result.get('error')}")
```

### แบบฝึกหัดที่ 6: httpx Async

```python
import httpx
import asyncio

async def fetch_user_data(user_ids: list) -> list:
    """Fetch user data แบบ async"""
    
    async with httpx.AsyncClient(base_url="https://jsonplaceholder.typicode.com") as client:
        tasks = [client.get(f"/users/{uid}") for uid in user_ids]
        responses = await asyncio.gather(*tasks, return_exceptions=True)
        
        users = []
        for uid, response in zip(user_ids, responses):
            if isinstance(response, Exception):
                users.append({'id': uid, 'error': str(response)})
            elif response.status_code == 200:
                users.append(response.json())
            else:
                users.append({'id': uid, 'error': f'HTTP {response.status_code}'})
        
        return users

async def main():
    users = await fetch_user_data([1, 2, 3, 4, 5])
    for user in users:
        if 'error' not in user:
            print(f"User {user['id']}: {user['name']} ({user['email']})")
        else:
            print(f"Error for user: {user}")

asyncio.run(main())
```

### แบบฝึกหัดที่ 7: Rate Limiting

```python
import requests
import time
import threading
from collections import deque
from datetime import datetime

class RateLimitedClient:
    """HTTP client ที่มี rate limiting"""
    
    def __init__(self, calls_per_second: float = 1.0):
        self.calls_per_second = calls_per_second
        self.min_interval = 1.0 / calls_per_second
        self._last_call_time = 0
        self._lock = threading.Lock()
        self._session = requests.Session()
        self._call_count = 0
    
    def get(self, url: str, **kwargs) -> requests.Response:
        with self._lock:
            now = time.time()
            elapsed = now - self._last_call_time
            
            if elapsed < self.min_interval:
                time.sleep(self.min_interval - elapsed)
            
            self._last_call_time = time.time()
            self._call_count += 1
        
        response = self._session.get(url, **kwargs)
        return response

# ใช้งาน
client = RateLimitedClient(calls_per_second=2)  # 2 requests/second

urls = [f"https://httpbin.org/get?req={i}" for i in range(5)]
start = time.time()

for url in urls:
    r = client.get(url, timeout=10)
    print(f"{datetime.now().strftime('%H:%M:%S.%f')[:-3]} - Status: {r.status_code}")

elapsed = time.time() - start
print(f"\nTotal time: {elapsed:.2f}s for {len(urls)} requests")
print(f"Actual rate: {len(urls)/elapsed:.2f} req/s")
```

### แบบฝึกหัดที่ 8: Full API Client

```python
import requests
from typing import Optional, List, Dict, Any

class GitHubClient:
    """GitHub API Client"""
    
    BASE_URL = "https://api.github.com"
    
    def __init__(self, token: Optional[str] = None):
        self._session = requests.Session()
        self._session.headers.update({
            'Accept': 'application/vnd.github.v3+json',
            'User-Agent': 'Python-GitHub-Client/1.0',
        })
        if token:
            self._session.headers['Authorization'] = f'token {token}'
    
    def _get(self, path: str, params: dict = None) -> Any:
        url = f"{self.BASE_URL}{path}"
        response = self._session.get(url, params=params, timeout=30)
        response.raise_for_status()
        return response.json()
    
    def get_user(self, username: str) -> dict:
        """ดูข้อมูล user"""
        return self._get(f"/users/{username}")
    
    def get_repos(self, username: str, page: int = 1, per_page: int = 30) -> List[dict]:
        """ดู repositories ของ user"""
        return self._get(f"/users/{username}/repos", {
            'page': page,
            'per_page': per_page,
            'sort': 'updated'
        })
    
    def get_repo(self, owner: str, repo: str) -> dict:
        """ดูข้อมูล repository"""
        return self._get(f"/repos/{owner}/{repo}")
    
    def get_rate_limit(self) -> dict:
        """ดู rate limit ที่เหลืออยู่"""
        return self._get("/rate_limit")
    
    def search_repos(self, query: str, per_page: int = 10) -> dict:
        """ค้นหา repositories"""
        return self._get("/search/repositories", {
            'q': query,
            'per_page': per_page,
            'sort': 'stars',
            'order': 'desc'
        })

# ใช้งาน (ไม่ต้องใช้ token สำหรับ public data)
client = GitHubClient()

# ดูข้อมูล user
try:
    user = client.get_user('torvalds')
    print(f"Name: {user['name']}")
    print(f"Public repos: {user['public_repos']}")
    print(f"Followers: {user['followers']}")
    
    # ดู repos
    repos = client.get_repos('torvalds', per_page=5)
    print(f"\nTop repos:")
    for repo in repos:
        print(f"  {repo['name']}: ⭐ {repo['stargazers_count']}")
    
    # ค้นหา
    results = client.search_repos('python web scraping', per_page=3)
    print(f"\nSearch results: {results['total_count']} repos")
    for repo in results['items'][:3]:
        print(f"  {repo['full_name']}: ⭐ {repo['stargazers_count']}")

except requests.exceptions.HTTPError as e:
    print(f"GitHub API error: {e}")
except requests.exceptions.ConnectionError:
    print("Cannot connect to GitHub API")
```

---

## สรุป

| Feature | requests | httpx |
|---------|----------|-------|
| HTTP Methods | GET, POST, PUT, PATCH, DELETE | เหมือนกัน |
| Authentication | Basic, Bearer, Custom | เหมือนกัน |
| Sessions | requests.Session() | httpx.Client() |
| Async | ไม่รองรับ | รองรับ |
| Streaming | รองรับ | รองรับ |
| HTTP/2 | ไม่รองรับ | รองรับ |
| Type hints | บางส่วน | ครบถ้วน |

**Best Practices:**
1. ใช้ Session สำหรับ multiple requests ไปยัง server เดียว
2. ตั้ง timeout เสมอ ห้ามปล่อยค้างไว้
3. Handle errors ทุกกรณี (HTTP errors, Connection errors, Timeout)
4. ใช้ retry ด้วย exponential backoff
5. Respect rate limits ของ API
6. อย่า hardcode credentials ใน code
7. ใช้ httpx สำหรับ async applications
8. Validate SSL certificates ในทุก production environments
