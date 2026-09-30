# Part 61 - FastAPI: WebSockets & Background Tasks

## สารบัญ

1. [WebSocket Protocol Overview](#1-websocket-protocol-overview)
2. [FastAPI WebSocket Endpoint](#2-fastapi-websocket-endpoint)
3. [WebSocket Connection Lifecycle](#3-websocket-connection-lifecycle)
4. [Broadcasting Messages to Multiple Clients](#4-broadcasting-messages-to-multiple-clients)
5. [Room/Channel Management Pattern](#5-roomchannel-management-pattern)
6. [WebSocket Authentication (Token-based)](#6-websocket-authentication-token-based)
7. [Background Tasks (@BackgroundTasks)](#7-background-tasks-backgroundtasks)
8. [Celery Integration กับ FastAPI](#8-celery-integration-กับ-fastapi)
9. [Task Queues Patterns](#9-task-queues-patterns)
10. [Scheduled Tasks (APScheduler)](#10-scheduled-tasks-apscheduler)
11. [Server-Sent Events (SSE) Alternative](#11-server-sent-events-sse-alternative)
12. [ตัวอย่างโปรแกรมจริง](#12-ตัวอย่างโปรแกรมจริง)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. WebSocket Protocol Overview

### WebSocket คืออะไร?

WebSocket เป็นโปรโตคอลการสื่อสารแบบ **full-duplex** (สองทิศทางพร้อมกัน) ที่ทำงานบน TCP connection เดียว ต่างจาก HTTP ปกติที่เป็นแบบ request-response WebSocket อนุญาตให้ทั้ง client และ server ส่งข้อมูลหากันได้ตลอดเวลาโดยไม่ต้องรอคำขอจากอีกฝ่าย

### เปรียบเทียบ HTTP vs WebSocket

| คุณสมบัติ | HTTP | WebSocket |
|-----------|------|-----------|
| การเชื่อมต่อ | เปิด-ปิดทุก request | เปิดค้างไว้ |
| ทิศทางข้อมูล | Client → Server (one-way) | สองทิศทาง (full-duplex) |
| Overhead | Header ใหญ่ทุก request | Header เล็กหลัง handshake |
| Use case | REST APIs, file transfer | Chat, gaming, live updates |
| Protocol | ws:// หรือ wss:// (TLS) | http:// หรือ https:// |

### WebSocket Handshake Process

เมื่อ client ต้องการเชื่อมต่อ WebSocket จะเริ่มด้วย HTTP Upgrade request:

```
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Server ตอบกลับด้วย:
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

หลังจากนั้น connection จะถูก upgrade เป็น WebSocket และสามารถส่งข้อมูลได้ทันที

### การติดตั้ง Dependencies

```bash
pip install fastapi uvicorn websockets python-jose[cryptography] passlib[bcrypt]
```

---

## 2. FastAPI WebSocket Endpoint

### WebSocket Endpoint พื้นฐาน

FastAPI รองรับ WebSocket ผ่าน `WebSocket` class ที่ import มาจาก `fastapi`

**ตัวอย่างที่ 1: WebSocket endpoint ง่ายๆ**

```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
<head><title>WebSocket Test</title></head>
<body>
    <h1>WebSocket Test</h1>
    <form action="" onsubmit="sendMessage(event)">
        <input type="text" id="messageText" autocomplete="off"/>
        <button>Send</button>
    </form>
    <ul id='messages'></ul>
    <script>
        var ws = new WebSocket("ws://localhost:8000/ws");
        ws.onmessage = function(event) {
            var messages = document.getElementById('messages');
            var message = document.createElement('li');
            message.appendChild(document.createTextNode(event.data));
            messages.appendChild(message);
        };
        function sendMessage(event) {
            var input = document.getElementById("messageText");
            ws.send(input.value);
            input.value = '';
            event.preventDefault();
        }
    </script>
</body>
</html>
"""

@app.get("/")
async def get():
    return HTMLResponse(html)

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    # รับการเชื่อมต่อ
    await websocket.accept()
    try:
        while True:
            # รับข้อความจาก client
            data = await websocket.receive_text()
            # ส่งข้อความกลับ
            await websocket.send_text(f"Message received: {data}")
    except Exception:
        pass
```

**ตัวอย่างที่ 2: รับและส่งข้อมูลแบบ JSON**

```python
from fastapi import FastAPI, WebSocket
from pydantic import BaseModel
import json

app = FastAPI()

class Message(BaseModel):
    type: str
    content: str
    sender: str = "anonymous"

@app.websocket("/ws/json")
async def websocket_json_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            # รับ JSON data
            data = await websocket.receive_json()
            message = Message(**data)
            
            # ประมวลผลตาม type
            if message.type == "chat":
                response = {
                    "type": "chat_response",
                    "content": f"Echo: {message.content}",
                    "from": "server"
                }
            elif message.type == "ping":
                response = {"type": "pong", "content": "alive"}
            else:
                response = {"type": "error", "content": "Unknown message type"}
            
            await websocket.send_json(response)
    except Exception as e:
        print(f"Connection closed: {e}")
```

**ตัวอย่างที่ 3: รับ Binary data (เช่น ไฟล์รูปภาพ)**

```python
from fastapi import FastAPI, WebSocket
import base64

app = FastAPI()

@app.websocket("/ws/binary")
async def websocket_binary_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            # รับ binary data
            data = await websocket.receive_bytes()
            
            # บันทึกไฟล์
            filename = f"received_{len(data)}_bytes.bin"
            with open(filename, "wb") as f:
                f.write(data)
            
            # ส่ง confirmation กลับ
            await websocket.send_text(f"Received {len(data)} bytes, saved as {filename}")
    except Exception as e:
        print(f"Error: {e}")
```

---

## 3. WebSocket Connection Lifecycle

### วงจรชีวิตของ WebSocket Connection

การเชื่อมต่อ WebSocket มี 4 ขั้นตอนหลัก:

1. **Connect** - Client ส่ง HTTP upgrade request
2. **Open** - Server ยอมรับการเชื่อมต่อ (accept)
3. **Message Exchange** - รับส่งข้อมูลสองทิศทาง
4. **Close** - ปิดการเชื่อมต่อ (ฝ่ายใดฝ่ายหนึ่ง)

**ตัวอย่างที่ 4: การจัดการ Lifecycle อย่างสมบูรณ์**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()

@app.websocket("/ws/lifecycle")
async def websocket_lifecycle(websocket: WebSocket):
    client_id = id(websocket)
    
    # Phase 1: Connect
    logger.info(f"Client {client_id} attempting to connect...")
    await websocket.accept()
    logger.info(f"Client {client_id} connected successfully")
    
    # ส่ง welcome message
    await websocket.send_json({
        "event": "connected",
        "message": "Welcome! You are now connected.",
        "client_id": str(client_id)
    })
    
    try:
        # Phase 3: Message Exchange
        while True:
            # รอรับข้อมูล - จะ block จนกว่าจะได้รับ
            message = await websocket.receive_text()
            logger.info(f"Client {client_id} sent: {message}")
            
            if message.lower() == "quit":
                # ส่ง goodbye message
                await websocket.send_json({
                    "event": "closing",
                    "message": "Goodbye! Connection will close."
                })
                break
            
            # Echo กลับ
            await websocket.send_text(f"Echo [{client_id}]: {message}")
    
    except WebSocketDisconnect as e:
        # Phase 4: Close (Client ตัดการเชื่อมต่อ)
        logger.info(f"Client {client_id} disconnected. Code: {e.code}, Reason: {e.reason}")
    
    except Exception as e:
        logger.error(f"Client {client_id} error: {e}")
    
    finally:
        # Cleanup
        logger.info(f"Client {client_id} connection cleaned up")
```

**ตัวอย่างที่ 5: WebSocket State Management**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from enum import Enum
import asyncio

app = FastAPI()

class ConnectionState(Enum):
    CONNECTING = "connecting"
    CONNECTED = "connected"
    DISCONNECTING = "disconnecting"
    DISCONNECTED = "disconnected"

class ManagedWebSocket:
    def __init__(self, websocket: WebSocket):
        self.websocket = websocket
        self.state = ConnectionState.CONNECTING
        self.message_count = 0
        self.client_id = id(websocket)
    
    async def connect(self):
        await self.websocket.accept()
        self.state = ConnectionState.CONNECTED
        print(f"[{self.client_id}] State: {self.state.value}")
    
    async def send(self, data: dict):
        if self.state == ConnectionState.CONNECTED:
            await self.websocket.send_json(data)
            self.message_count += 1
    
    async def receive(self):
        return await self.websocket.receive_text()
    
    async def disconnect(self, code: int = 1000):
        self.state = ConnectionState.DISCONNECTING
        await self.websocket.close(code=code)
        self.state = ConnectionState.DISCONNECTED
        print(f"[{self.client_id}] State: {self.state.value}")

@app.websocket("/ws/managed")
async def websocket_managed(websocket: WebSocket):
    conn = ManagedWebSocket(websocket)
    await conn.connect()
    
    await conn.send({
        "event": "welcome",
        "state": conn.state.value,
        "client_id": conn.client_id
    })
    
    try:
        while True:
            text = await conn.receive()
            await conn.send({
                "event": "echo",
                "data": text,
                "message_count": conn.message_count
            })
    except WebSocketDisconnect:
        print(f"Client {conn.client_id} disconnected after {conn.message_count} messages")
```

### WebSocket Close Codes

| Code | ความหมาย |
|------|----------|
| 1000 | Normal Closure - ปิดปกติ |
| 1001 | Going Away - Server กำลังปิด |
| 1002 | Protocol Error |
| 1003 | Unsupported Data |
| 1006 | Abnormal Closure - ไม่มี close frame |
| 1008 | Policy Violation |
| 1011 | Internal Server Error |

---

## 4. Broadcasting Messages to Multiple Clients

### ConnectionManager สำหรับจัดการหลาย Client

**ตัวอย่างที่ 6: ConnectionManager พื้นฐาน**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List
import json

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        # เก็บ list ของ active connections
        self.active_connections: List[WebSocket] = []
    
    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)
        print(f"New connection. Total: {len(self.active_connections)}")
    
    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)
        print(f"Disconnected. Remaining: {len(self.active_connections)}")
    
    async def send_personal_message(self, message: str, websocket: WebSocket):
        """ส่งข้อความให้ client คนเดียว"""
        await websocket.send_text(message)
    
    async def broadcast(self, message: str):
        """ส่งข้อความให้ทุก client ที่เชื่อมต่ออยู่"""
        disconnected = []
        for connection in self.active_connections:
            try:
                await connection.send_text(message)
            except Exception:
                disconnected.append(connection)
        
        # ลบ connections ที่ตัดไปแล้ว
        for conn in disconnected:
            self.active_connections.remove(conn)
    
    async def broadcast_json(self, data: dict):
        """ส่ง JSON ให้ทุก client"""
        message = json.dumps(data)
        await self.broadcast(message)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    
    # แจ้งให้ทุกคนรู้ว่ามี user ใหม่
    await manager.broadcast(f"Client #{client_id} joined the chat")
    
    try:
        while True:
            data = await websocket.receive_text()
            # ส่งข้อความส่วนตัวกลับ (confirm)
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            # broadcast ให้ทุกคน
            await manager.broadcast(f"Client #{client_id}: {data}")
    
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")
```

**ตัวอย่างที่ 7: Broadcast พร้อม metadata**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import Dict, Set
from datetime import datetime
import asyncio

app = FastAPI()

class AdvancedConnectionManager:
    def __init__(self):
        # Dict mapping websocket -> user info
        self.connections: Dict[WebSocket, dict] = {}
    
    async def connect(self, websocket: WebSocket, username: str):
        await websocket.accept()
        self.connections[websocket] = {
            "username": username,
            "connected_at": datetime.now().isoformat(),
            "message_count": 0
        }
    
    def disconnect(self, websocket: WebSocket):
        if websocket in self.connections:
            del self.connections[websocket]
    
    def get_user_list(self) -> list:
        return [info["username"] for info in self.connections.values()]
    
    async def broadcast(self, event_type: str, data: dict, exclude: WebSocket = None):
        message = {"type": event_type, "data": data, "timestamp": datetime.now().isoformat()}
        dead_connections = []
        
        for ws, info in self.connections.items():
            if ws == exclude:
                continue
            try:
                await ws.send_json(message)
            except Exception:
                dead_connections.append(ws)
        
        for ws in dead_connections:
            del self.connections[ws]

manager = AdvancedConnectionManager()

@app.websocket("/ws/chat/{username}")
async def chat_endpoint(websocket: WebSocket, username: str):
    await manager.connect(websocket, username)
    
    await manager.broadcast("user_joined", {
        "username": username,
        "users_online": manager.get_user_list()
    })
    
    try:
        while True:
            data = await websocket.receive_json()
            
            if data.get("type") == "message":
                manager.connections[websocket]["message_count"] += 1
                await manager.broadcast("chat_message", {
                    "from": username,
                    "text": data.get("text", ""),
                    "count": manager.connections[websocket]["message_count"]
                })
            
            elif data.get("type") == "who_is_online":
                await websocket.send_json({
                    "type": "user_list",
                    "users": manager.get_user_list()
                })
    
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast("user_left", {
            "username": username,
            "users_online": manager.get_user_list()
        })
```

---

## 5. Room/Channel Management Pattern

### การจัดการ Rooms และ Channels

ระบบ Chat มักต้องการแบ่งผู้ใช้เป็นกลุ่มย่อย เช่น ห้องสนทนา (rooms) หรือ channels

**ตัวอย่างที่ 8: Room Manager**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import Dict, Set, Optional
from dataclasses import dataclass, field
from datetime import datetime
import json

app = FastAPI()

@dataclass
class Room:
    name: str
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
    members: Set[WebSocket] = field(default_factory=set)
    message_history: list = field(default_factory=list)
    max_members: int = 50
    
    def is_full(self) -> bool:
        return len(self.members) >= self.max_members
    
    def member_count(self) -> int:
        return len(self.members)

class RoomManager:
    def __init__(self):
        self.rooms: Dict[str, Room] = {}
        self.user_rooms: Dict[WebSocket, str] = {}  # ws -> room_name
    
    def create_room(self, room_name: str, max_members: int = 50) -> Room:
        if room_name not in self.rooms:
            self.rooms[room_name] = Room(name=room_name, max_members=max_members)
            print(f"Room '{room_name}' created")
        return self.rooms[room_name]
    
    def get_room(self, room_name: str) -> Optional[Room]:
        return self.rooms.get(room_name)
    
    async def join_room(self, websocket: WebSocket, room_name: str, username: str) -> bool:
        room = self.rooms.get(room_name)
        if not room:
            room = self.create_room(room_name)
        
        if room.is_full():
            await websocket.send_json({
                "type": "error",
                "message": f"Room '{room_name}' is full ({room.max_members} members max)"
            })
            return False
        
        # ออกจาก room เดิมก่อน
        if websocket in self.user_rooms:
            await self.leave_room(websocket, self.user_rooms[websocket], username)
        
        room.members.add(websocket)
        self.user_rooms[websocket] = room_name
        
        # แจ้งสมาชิกในห้อง
        await self.broadcast_to_room(room_name, {
            "type": "user_joined_room",
            "username": username,
            "room": room_name,
            "member_count": room.member_count()
        }, exclude=websocket)
        
        # ส่ง history ให้ user ใหม่
        await websocket.send_json({
            "type": "room_joined",
            "room": room_name,
            "member_count": room.member_count(),
            "history": room.message_history[-20:]  # 20 ข้อความล่าสุด
        })
        
        return True
    
    async def leave_room(self, websocket: WebSocket, room_name: str, username: str):
        room = self.rooms.get(room_name)
        if room and websocket in room.members:
            room.members.discard(websocket)
            if websocket in self.user_rooms:
                del self.user_rooms[websocket]
            
            await self.broadcast_to_room(room_name, {
                "type": "user_left_room",
                "username": username,
                "room": room_name,
                "member_count": room.member_count()
            })
            
            # ลบ room ที่ว่างเปล่า
            if room.member_count() == 0:
                del self.rooms[room_name]
                print(f"Room '{room_name}' deleted (empty)")
    
    async def broadcast_to_room(self, room_name: str, data: dict, exclude: WebSocket = None):
        room = self.rooms.get(room_name)
        if not room:
            return
        
        dead_connections = set()
        for ws in room.members:
            if ws == exclude:
                continue
            try:
                await ws.send_json(data)
            except Exception:
                dead_connections.add(ws)
        
        room.members -= dead_connections
    
    async def send_message(self, websocket: WebSocket, text: str, username: str):
        room_name = self.user_rooms.get(websocket)
        if not room_name:
            await websocket.send_json({
                "type": "error",
                "message": "You are not in any room"
            })
            return
        
        room = self.rooms[room_name]
        msg = {
            "type": "room_message",
            "from": username,
            "text": text,
            "room": room_name,
            "timestamp": datetime.now().isoformat()
        }
        
        # เก็บ history
        room.message_history.append(msg)
        if len(room.message_history) > 100:
            room.message_history = room.message_history[-100:]
        
        await self.broadcast_to_room(room_name, msg)
    
    def list_rooms(self) -> list:
        return [
            {"name": name, "members": room.member_count(), "full": room.is_full()}
            for name, room in self.rooms.items()
        ]

room_manager = RoomManager()

# สร้าง default rooms
room_manager.create_room("general")
room_manager.create_room("tech")
room_manager.create_room("random")

@app.websocket("/ws/rooms/{username}")
async def room_websocket(websocket: WebSocket, username: str):
    await websocket.accept()
    
    try:
        while True:
            data = await websocket.receive_json()
            action = data.get("action")
            
            if action == "join":
                await room_manager.join_room(websocket, data.get("room", "general"), username)
            
            elif action == "leave":
                room_name = room_manager.user_rooms.get(websocket)
                if room_name:
                    await room_manager.leave_room(websocket, room_name, username)
            
            elif action == "message":
                await room_manager.send_message(websocket, data.get("text", ""), username)
            
            elif action == "list_rooms":
                await websocket.send_json({
                    "type": "room_list",
                    "rooms": room_manager.list_rooms()
                })
    
    except WebSocketDisconnect:
        room_name = room_manager.user_rooms.get(websocket)
        if room_name:
            await room_manager.leave_room(websocket, room_name, username)

@app.get("/rooms")
async def get_rooms():
    return room_manager.list_rooms()
```

---

## 6. WebSocket Authentication (Token-based)

### การยืนยันตัวตนใน WebSocket

WebSocket ไม่รองรับ HTTP headers หลังจาก initial handshake ดังนั้นการ authentication มักทำผ่าน:
1. **Query Parameters** - ส่ง token ใน URL
2. **First Message** - ส่ง token เป็นข้อความแรก
3. **Cookie** - ใช้ cookie จาก browser

**ตัวอย่างที่ 9: Token Authentication ผ่าน Query Parameter**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, Query, HTTPException
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from datetime import datetime, timedelta
from typing import Optional

app = FastAPI()

SECRET_KEY = "your-secret-key-change-in-production"
ALGORITHM = "HS256"

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str) -> Optional[dict]:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None

@app.post("/token")
async def get_token(username: str):
    """สร้าง token สำหรับ test"""
    token = create_access_token(
        data={"sub": username, "role": "user"},
        expires_delta=timedelta(hours=1)
    )
    return {"access_token": token, "token_type": "bearer"}

@app.websocket("/ws/secure")
async def secure_websocket(
    websocket: WebSocket,
    token: str = Query(..., description="JWT authentication token")
):
    # Verify token ก่อน accept
    payload = verify_token(token)
    
    if not payload:
        await websocket.close(code=4001, reason="Invalid or expired token")
        return
    
    username = payload.get("sub")
    role = payload.get("role")
    
    await websocket.accept()
    
    await websocket.send_json({
        "event": "authenticated",
        "username": username,
        "role": role,
        "message": f"Welcome, {username}!"
    })
    
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"[{username}]: {data}")
    
    except WebSocketDisconnect:
        print(f"User {username} disconnected")
```

**ตัวอย่างที่ 10: Authentication ผ่าน First Message**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from jose import JWTError, jwt
import asyncio

app = FastAPI()

SECRET_KEY = "change-this-secret"
ALGORITHM = "HS256"
AUTH_TIMEOUT = 10  # seconds

@app.websocket("/ws/auth-first-msg")
async def websocket_auth_first(websocket: WebSocket):
    await websocket.accept()
    
    # รอ authentication message
    await websocket.send_json({
        "type": "auth_required",
        "message": "Please send your token within 10 seconds"
    })
    
    try:
        # รอ token ด้วย timeout
        auth_data = await asyncio.wait_for(
            websocket.receive_json(),
            timeout=AUTH_TIMEOUT
        )
        
        if auth_data.get("type") != "auth":
            await websocket.send_json({"type": "error", "message": "Expected auth message"})
            await websocket.close(code=4001)
            return
        
        token = auth_data.get("token")
        payload = None
        
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        except JWTError:
            await websocket.send_json({"type": "error", "message": "Invalid token"})
            await websocket.close(code=4001)
            return
        
        username = payload.get("sub")
        
        await websocket.send_json({
            "type": "auth_success",
            "username": username,
            "message": "Authentication successful!"
        })
        
        # Main message loop
        while True:
            data = await websocket.receive_json()
            await websocket.send_json({
                "type": "response",
                "from": username,
                "echo": data
            })
    
    except asyncio.TimeoutError:
        await websocket.send_json({
            "type": "error",
            "message": "Authentication timeout"
        })
        await websocket.close(code=4008)
    
    except WebSocketDisconnect:
        print("Client disconnected during auth or session")
```

---

## 7. Background Tasks (@BackgroundTasks)

### FastAPI BackgroundTasks

`BackgroundTasks` ใน FastAPI ช่วยให้สามารถรัน task หลัง response ถูกส่งกลับไปแล้ว เหมาะสำหรับงานที่ไม่จำเป็นต้องรอผล เช่น ส่ง email, บันทึก log, หรือ cleanup

**ตัวอย่างที่ 11: Background Task พื้นฐาน**

```python
from fastapi import FastAPI, BackgroundTasks
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()

def write_log(message: str):
    """Task ที่รันใน background"""
    time.sleep(2)  # จำลองงานที่ใช้เวลา
    logger.info(f"Background log: {message}")

def send_notification(email: str, message: str):
    """จำลองการส่ง notification"""
    time.sleep(1)
    logger.info(f"Notification sent to {email}: {message}")

@app.post("/process")
async def process_item(
    item_id: int,
    background_tasks: BackgroundTasks
):
    # เพิ่ม background tasks
    background_tasks.add_task(write_log, f"Processing item {item_id}")
    background_tasks.add_task(
        send_notification,
        "user@example.com",
        f"Item {item_id} is being processed"
    )
    
    # Response ส่งกลับทันที ไม่ต้องรอ background tasks
    return {
        "message": f"Item {item_id} accepted for processing",
        "status": "processing"
    }
```

**ตัวอย่างที่ 12: Background Task กับ async function**

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel, EmailStr
import asyncio

app = FastAPI()

class UserRegistration(BaseModel):
    username: str
    email: str
    password: str

# Database simulation
users_db = {}

async def send_welcome_email(email: str, username: str):
    """Async background task"""
    await asyncio.sleep(2)  # จำลอง async I/O (เช่น SMTP)
    print(f"[EMAIL] Welcome email sent to {email} for user {username}")

async def setup_default_settings(user_id: str):
    """สร้าง default settings สำหรับ user ใหม่"""
    await asyncio.sleep(1)
    print(f"[SETUP] Default settings created for user {user_id}")

async def log_registration_event(username: str, ip: str = "unknown"):
    """บันทึก event ใน audit log"""
    await asyncio.sleep(0.5)
    print(f"[AUDIT] New registration: {username} from IP {ip}")

@app.post("/register")
async def register_user(
    user: UserRegistration,
    background_tasks: BackgroundTasks
):
    # บันทึก user
    user_id = f"user_{len(users_db) + 1}"
    users_db[user_id] = {
        "id": user_id,
        "username": user.username,
        "email": user.email,
        "active": True
    }
    
    # เพิ่ม background tasks หลายๆ อัน
    background_tasks.add_task(send_welcome_email, user.email, user.username)
    background_tasks.add_task(setup_default_settings, user_id)
    background_tasks.add_task(log_registration_event, user.username)
    
    # Response ทันที
    return {
        "user_id": user_id,
        "message": f"User {user.username} registered successfully",
        "note": "Welcome email will be sent shortly"
    }
```

**ตัวอย่างที่ 13: Background Task กับ Dependency Injection**

```python
from fastapi import FastAPI, BackgroundTasks, Depends
from typing import Optional
import asyncio
from datetime import datetime

app = FastAPI()

class EmailService:
    def __init__(self, smtp_host: str = "localhost"):
        self.smtp_host = smtp_host
    
    async def send_email(self, to: str, subject: str, body: str):
        """จำลองการส่ง email"""
        await asyncio.sleep(1)
        print(f"[EMAIL via {self.smtp_host}]")
        print(f"  To: {to}")
        print(f"  Subject: {subject}")
        print(f"  Body: {body[:50]}...")

class AuditLogger:
    async def log(self, action: str, user: str, details: dict = None):
        await asyncio.sleep(0.1)
        timestamp = datetime.now().isoformat()
        print(f"[AUDIT {timestamp}] {action} by {user}: {details}")

def get_email_service() -> EmailService:
    return EmailService(smtp_host="mail.example.com")

def get_audit_logger() -> AuditLogger:
    return AuditLogger()

@app.post("/orders/{order_id}/confirm")
async def confirm_order(
    order_id: str,
    background_tasks: BackgroundTasks,
    email_svc: EmailService = Depends(get_email_service),
    audit: AuditLogger = Depends(get_audit_logger)
):
    # Process order confirmation
    order = {"id": order_id, "status": "confirmed", "user_email": "customer@example.com"}
    
    # Background tasks
    background_tasks.add_task(
        email_svc.send_email,
        to=order["user_email"],
        subject=f"Order {order_id} Confirmed",
        body=f"Your order {order_id} has been confirmed and will be shipped soon."
    )
    background_tasks.add_task(
        audit.log,
        action="ORDER_CONFIRMED",
        user="system",
        details={"order_id": order_id}
    )
    
    return {"order_id": order_id, "status": "confirmed"}
```

---

## 8. Celery Integration กับ FastAPI

### Celery คืออะไร?

Celery เป็น distributed task queue ที่ใช้สำหรับงานหนักที่ต้องรัน asynchronously หรือ scheduled Celery ต้องการ message broker เช่น Redis หรือ RabbitMQ

```bash
pip install celery redis flower
# ต้องมี Redis running: docker run -d -p 6379:6379 redis:alpine
```

**ตัวอย่างที่ 14: Celery Setup กับ FastAPI**

```python
# celery_config.py
from celery import Celery

def make_celery(app_name: str = "fastapi_app") -> Celery:
    celery = Celery(
        app_name,
        broker="redis://localhost:6379/0",
        backend="redis://localhost:6379/0",
        include=["tasks"]  # module ที่มี tasks
    )
    
    celery.conf.update(
        task_serializer="json",
        accept_content=["json"],
        result_serializer="json",
        timezone="Asia/Bangkok",
        enable_utc=False,
        # Retry settings
        task_acks_late=True,
        task_reject_on_worker_lost=True,
        # Rate limits
        task_default_rate_limit="100/m",
    )
    
    return celery

celery_app = make_celery()
```

```python
# tasks.py
from celery_config import celery_app
import time

@celery_app.task(bind=True, max_retries=3)
def send_email_task(self, to: str, subject: str, body: str):
    """Task สำหรับส่ง email"""
    try:
        time.sleep(2)  # จำลองการส่ง email
        print(f"Email sent to {to}: {subject}")
        return {"status": "sent", "to": to}
    except Exception as exc:
        # Retry ถ้า fail
        raise self.retry(exc=exc, countdown=5)

@celery_app.task
def process_image(image_path: str, operations: list):
    """Task สำหรับ process รูปภาพ"""
    time.sleep(5)  # จำลอง heavy processing
    print(f"Processing {image_path} with {operations}")
    return {"status": "processed", "path": image_path}

@celery_app.task
def generate_report(user_id: str, report_type: str):
    """Task สำหรับ generate report"""
    time.sleep(10)
    report_data = {
        "user_id": user_id,
        "type": report_type,
        "data": {"total_sales": 1234, "new_users": 56},
        "generated_at": time.time()
    }
    return report_data
```

```python
# main.py - FastAPI app ที่ใช้ Celery
from fastapi import FastAPI
from celery.result import AsyncResult
from tasks import send_email_task, process_image, generate_report
from celery_config import celery_app

app = FastAPI()

@app.post("/send-email")
async def send_email(to: str, subject: str, body: str):
    """Dispatch email task ไปยัง Celery worker"""
    task = send_email_task.delay(to, subject, body)
    return {
        "task_id": task.id,
        "status": "queued",
        "message": "Email task has been queued"
    }

@app.post("/process-image")
async def process_image_endpoint(image_path: str):
    """Dispatch image processing task"""
    task = process_image.delay(
        image_path,
        ["resize", "compress", "watermark"]
    )
    return {"task_id": task.id, "status": "queued"}

@app.get("/tasks/{task_id}")
async def get_task_status(task_id: str):
    """ตรวจสอบสถานะของ task"""
    result = AsyncResult(task_id, app=celery_app)
    
    response = {
        "task_id": task_id,
        "status": result.status,
        "ready": result.ready(),
        "successful": result.successful() if result.ready() else None,
    }
    
    if result.ready():
        if result.successful():
            response["result"] = result.result
        else:
            response["error"] = str(result.result)
    elif result.status == "PROGRESS":
        response["progress"] = result.info
    
    return response

@app.delete("/tasks/{task_id}")
async def cancel_task(task_id: str):
    """ยกเลิก task ที่กำลัง queue อยู่"""
    result = AsyncResult(task_id, app=celery_app)
    result.revoke(terminate=True)
    return {"task_id": task_id, "status": "cancelled"}
```

**การ run Celery:**
```bash
# Run worker
celery -A celery_config.celery_app worker --loglevel=info

# Run Flower (monitoring UI)
celery -A celery_config.celery_app flower --port=5555

# Run FastAPI app
uvicorn main:app --reload
```

---

## 9. Task Queues Patterns

### Patterns ที่ใช้บ่อยใน Task Queues

**ตัวอย่างที่ 15: Priority Queue Pattern**

```python
from celery import Celery
from kombu import Queue, Exchange

celery_app = Celery("app", broker="redis://localhost:6379/0")

# กำหนด queues ที่มี priority ต่างกัน
celery_app.conf.task_queues = (
    Queue("high_priority", Exchange("high_priority"), routing_key="high"),
    Queue("default", Exchange("default"), routing_key="default"),
    Queue("low_priority", Exchange("low_priority"), routing_key="low"),
)

celery_app.conf.task_default_queue = "default"
celery_app.conf.task_default_exchange = "default"
celery_app.conf.task_default_routing_key = "default"

celery_app.conf.task_routes = {
    "tasks.send_urgent_notification": {"queue": "high_priority"},
    "tasks.send_email_task": {"queue": "default"},
    "tasks.generate_report": {"queue": "low_priority"},
}

@celery_app.task
def send_urgent_notification(user_id: str, message: str):
    """งานเร่งด่วนที่ต้องรีบทำ"""
    print(f"URGENT: {message} to user {user_id}")

@celery_app.task
def bulk_process_data(data_ids: list):
    """งาน batch ที่ไม่เร่งด่วน"""
    for item_id in data_ids:
        print(f"Processing item {item_id}")
    return {"processed": len(data_ids)}
```

**ตัวอย่างที่ 16: Chain และ Group Pattern (Workflow)**

```python
from celery import chain, group, chord
from celery_config import celery_app

@celery_app.task
def validate_data(data: dict) -> dict:
    """ตรวจสอบ data"""
    print(f"Validating: {data}")
    data["validated"] = True
    return data

@celery_app.task
def transform_data(data: dict) -> dict:
    """แปลง data"""
    data["transformed"] = True
    data["value"] = data.get("value", 0) * 2
    return data

@celery_app.task
def save_data(data: dict) -> dict:
    """บันทึก data"""
    data["saved"] = True
    return {"status": "success", "data": data}

@celery_app.task
def process_chunk(chunk: list) -> int:
    """ประมวลผล chunk ของ data"""
    return sum(chunk)

@celery_app.task
def aggregate_results(results: list) -> dict:
    """รวบรวมผลลัพธ์ทั้งหมด"""
    return {"total": sum(results), "chunks": len(results)}

# FastAPI endpoints ที่ใช้ workflow patterns
from fastapi import FastAPI
app = FastAPI()

@app.post("/pipeline/{item_id}")
async def run_pipeline(item_id: int):
    """Chain: validate -> transform -> save"""
    # Chain รัน tasks ต่อกัน output ของ task แรกเป็น input ของ task ถัดไป
    pipeline = chain(
        validate_data.s({"id": item_id, "value": 100}),
        transform_data.s(),
        save_data.s()
    )
    result = pipeline.delay()
    return {"task_id": result.id, "pattern": "chain"}

@app.post("/batch-process")
async def run_batch():
    """Group: รัน tasks พร้อมกัน"""
    data_chunks = [[1,2,3], [4,5,6], [7,8,9], [10,11,12]]
    
    # Group รัน tasks พร้อมกันแบบ parallel
    job = group(process_chunk.s(chunk) for chunk in data_chunks)
    result = job.delay()
    return {"group_id": result.id, "pattern": "group"}

@app.post("/chord-example")
async def run_chord():
    """Chord: parallel tasks แล้ว aggregate ผลลัพธ์"""
    data_chunks = [[1,2,3], [4,5,6], [7,8,9]]
    
    # Chord = Group + Callback
    # รัน process_chunk พร้อมกัน แล้วส่งผลทั้งหมดไปให้ aggregate_results
    job = chord(
        group(process_chunk.s(chunk) for chunk in data_chunks),
        aggregate_results.s()
    )
    result = job.delay()
    return {"chord_id": result.id, "pattern": "chord"}
```

---

## 10. Scheduled Tasks (APScheduler)

### APScheduler สำหรับ Scheduled Tasks

APScheduler ช่วยให้สามารถ schedule tasks ใน Python application โดยตรง

```bash
pip install apscheduler
```

**ตัวอย่างที่ 17: APScheduler กับ FastAPI**

```python
from fastapi import FastAPI
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.cron import CronTrigger
from apscheduler.triggers.interval import IntervalTrigger
from datetime import datetime
import asyncio

app = FastAPI()
scheduler = AsyncIOScheduler(timezone="Asia/Bangkok")

# Tasks ที่จะถูก schedule
async def cleanup_old_sessions():
    """ลบ sessions ที่หมดอายุ"""
    print(f"[{datetime.now()}] Cleaning up old sessions...")
    # Logic ลบ session ที่นี่
    print("Cleanup completed")

async def send_daily_report():
    """ส่ง daily report"""
    print(f"[{datetime.now()}] Generating and sending daily report...")
    await asyncio.sleep(2)  # จำลองการ generate report
    print("Daily report sent")

async def check_system_health():
    """ตรวจสอบสุขภาพระบบ"""
    print(f"[{datetime.now()}] Health check running...")
    # ตรวจสอบ database, cache, external services

async def process_pending_notifications():
    """ส่ง notifications ที่ค้างอยู่"""
    print(f"[{datetime.now()}] Processing pending notifications...")

@app.on_event("startup")
async def start_scheduler():
    # ทุก 5 นาที
    scheduler.add_job(
        cleanup_old_sessions,
        IntervalTrigger(minutes=5),
        id="cleanup_sessions",
        name="Cleanup Old Sessions",
        replace_existing=True
    )
    
    # ทุกวัน เวลา 08:00
    scheduler.add_job(
        send_daily_report,
        CronTrigger(hour=8, minute=0),
        id="daily_report",
        name="Daily Report",
        replace_existing=True
    )
    
    # ทุก 30 วินาที
    scheduler.add_job(
        check_system_health,
        IntervalTrigger(seconds=30),
        id="health_check",
        name="System Health Check",
        replace_existing=True
    )
    
    # ทุกชั่วโมง
    scheduler.add_job(
        process_pending_notifications,
        CronTrigger(minute=0),
        id="process_notifications",
        name="Process Notifications",
        replace_existing=True
    )
    
    scheduler.start()
    print("Scheduler started with jobs:")
    for job in scheduler.get_jobs():
        print(f"  - {job.name} (next: {job.next_run_time})")

@app.on_event("shutdown")
async def stop_scheduler():
    scheduler.shutdown()
    print("Scheduler stopped")

@app.get("/scheduler/jobs")
async def list_jobs():
    """ดู scheduled jobs ทั้งหมด"""
    jobs = []
    for job in scheduler.get_jobs():
        jobs.append({
            "id": job.id,
            "name": job.name,
            "next_run": str(job.next_run_time),
            "trigger": str(job.trigger)
        })
    return {"jobs": jobs}

@app.post("/scheduler/jobs/{job_id}/run")
async def run_job_now(job_id: str):
    """รัน job ทันที"""
    job = scheduler.get_job(job_id)
    if not job:
        return {"error": f"Job '{job_id}' not found"}
    job.modify(next_run_time=datetime.now())
    return {"message": f"Job '{job_id}' triggered manually"}

@app.post("/scheduler/jobs/{job_id}/pause")
async def pause_job(job_id: str):
    """หยุด job ชั่วคราว"""
    scheduler.pause_job(job_id)
    return {"message": f"Job '{job_id}' paused"}

@app.post("/scheduler/jobs/{job_id}/resume")
async def resume_job(job_id: str):
    """เริ่ม job ที่หยุดอยู่"""
    scheduler.resume_job(job_id)
    return {"message": f"Job '{job_id}' resumed"}
```

**ตัวอย่างที่ 18: Dynamic Job Creation**

```python
from fastapi import FastAPI
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.cron import CronTrigger
from pydantic import BaseModel
from typing import Optional
import asyncio

app = FastAPI()
scheduler = AsyncIOScheduler()

class JobConfig(BaseModel):
    job_id: str
    job_name: str
    cron_expression: str  # "minute hour day month weekday" e.g., "0 9 * * 1-5"
    url_to_call: str
    enabled: bool = True

async def call_webhook(url: str, job_id: str):
    """เรียก webhook URL"""
    import aiohttp
    async with aiohttp.ClientSession() as session:
        try:
            async with session.post(url, json={"job_id": job_id, "triggered_at": str(datetime.now())}) as resp:
                print(f"Webhook {url} called, status: {resp.status}")
        except Exception as e:
            print(f"Webhook call failed: {e}")

@app.post("/scheduler/create-job")
async def create_scheduled_job(config: JobConfig):
    """สร้าง scheduled job แบบ dynamic"""
    parts = config.cron_expression.split()
    if len(parts) != 5:
        return {"error": "Invalid cron expression (need 5 parts: min hour day month weekday)"}
    
    minute, hour, day, month, weekday = parts
    
    scheduler.add_job(
        call_webhook,
        CronTrigger(
            minute=minute,
            hour=hour,
            day=day,
            month=month,
            day_of_week=weekday
        ),
        id=config.job_id,
        name=config.job_name,
        args=[config.url_to_call, config.job_id],
        replace_existing=True
    )
    
    if not config.enabled:
        scheduler.pause_job(config.job_id)
    
    return {
        "message": f"Job '{config.job_name}' created",
        "next_run": str(scheduler.get_job(config.job_id).next_run_time)
    }

@app.delete("/scheduler/jobs/{job_id}")
async def delete_job(job_id: str):
    """ลบ scheduled job"""
    job = scheduler.get_job(job_id)
    if not job:
        return {"error": f"Job '{job_id}' not found"}
    scheduler.remove_job(job_id)
    return {"message": f"Job '{job_id}' deleted"}

@app.on_event("startup")
async def startup():
    scheduler.start()

@app.on_event("shutdown")  
async def shutdown():
    scheduler.shutdown()
```

---

## 11. Server-Sent Events (SSE) Alternative

### Server-Sent Events คืออะไร?

SSE เป็นทางเลือกแทน WebSocket สำหรับกรณีที่ต้องการ **one-way streaming** จาก server ไปยัง client เหมาะสำหรับ live feeds, notifications, หรือ real-time updates ที่ไม่ต้องการส่งข้อมูลจาก client

| คุณสมบัติ | SSE | WebSocket |
|-----------|-----|-----------|
| ทิศทาง | Server → Client เท่านั้น | สองทิศทาง |
| Protocol | HTTP | WS |
| Auto-reconnect | มีในตัว | ต้องเขียนเอง |
| Browser support | ดี | ดี |
| ผ่าน HTTP/2 | รองรับ | ยาก |

**ตัวอย่างที่ 19: SSE ด้วย StreamingResponse**

```python
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
import asyncio
import json
from datetime import datetime

app = FastAPI()

async def event_generator(request: Request):
    """Generator สำหรับ SSE"""
    event_id = 0
    
    while True:
        # ตรวจสอบว่า client ยังเชื่อมต่ออยู่
        if await request.is_disconnected():
            print("Client disconnected")
            break
        
        # สร้าง event data
        event_id += 1
        data = {
            "id": event_id,
            "time": datetime.now().isoformat(),
            "message": f"Event #{event_id}",
            "value": event_id * 10
        }
        
        # Format ตาม SSE specification
        # ต้องมี "data: " prefix และจบด้วย "\n\n"
        yield f"id: {event_id}\n"
        yield f"event: update\n"
        yield f"data: {json.dumps(data)}\n\n"
        
        await asyncio.sleep(1)

@app.get("/sse/events")
async def sse_endpoint(request: Request):
    return StreamingResponse(
        event_generator(request),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "X-Accel-Buffering": "no",  # สำหรับ Nginx
            "Connection": "keep-alive"
        }
    )

# HTML client สำหรับทดสอบ SSE
@app.get("/sse-demo")
async def sse_demo():
    html = """
    <!DOCTYPE html>
    <html>
    <head><title>SSE Demo</title></head>
    <body>
        <h1>Server-Sent Events Demo</h1>
        <div id="events"></div>
        <script>
            const eventsDiv = document.getElementById('events');
            const eventSource = new EventSource('/sse/events');
            
            eventSource.addEventListener('update', (e) => {
                const data = JSON.parse(e.data);
                const p = document.createElement('p');
                p.textContent = `Event #${data.id}: ${data.message} at ${data.time}`;
                eventsDiv.prepend(p);
            });
            
            eventSource.onerror = (e) => {
                console.log('SSE Error:', e);
            };
        </script>
    </body>
    </html>
    """
    from fastapi.responses import HTMLResponse
    return HTMLResponse(html)
```

**ตัวอย่างที่ 20: SSE สำหรับ Live Notifications**

```python
from fastapi import FastAPI, Request, BackgroundTasks
from fastapi.responses import StreamingResponse
import asyncio
import json
from datetime import datetime
from typing import Dict, List
import uuid

app = FastAPI()

# เก็บ notification queues สำหรับแต่ละ user
user_queues: Dict[str, asyncio.Queue] = {}

async def notification_stream(request: Request, user_id: str):
    """Stream notifications สำหรับ user คนเดียว"""
    queue = asyncio.Queue()
    user_queues[user_id] = queue
    
    try:
        # ส่ง connected event
        yield f"event: connected\ndata: {json.dumps({'user_id': user_id})}\n\n"
        
        while True:
            if await request.is_disconnected():
                break
            
            try:
                # รอ notification ใหม่ (timeout 30s เพื่อ heartbeat)
                notification = await asyncio.wait_for(queue.get(), timeout=30.0)
                yield f"event: notification\ndata: {json.dumps(notification)}\n\n"
            
            except asyncio.TimeoutError:
                # ส่ง heartbeat เพื่อรักษา connection
                yield f"event: heartbeat\ndata: {json.dumps({'ts': datetime.now().isoformat()})}\n\n"
    
    finally:
        if user_id in user_queues:
            del user_queues[user_id]

@app.get("/notifications/stream/{user_id}")
async def stream_notifications(request: Request, user_id: str):
    return StreamingResponse(
        notification_stream(request, user_id),
        media_type="text/event-stream",
        headers={"Cache-Control": "no-cache", "X-Accel-Buffering": "no"}
    )

@app.post("/notifications/send/{user_id}")
async def send_notification(user_id: str, message: str, type: str = "info"):
    """ส่ง notification ให้ user"""
    notification = {
        "id": str(uuid.uuid4()),
        "type": type,
        "message": message,
        "timestamp": datetime.now().isoformat()
    }
    
    if user_id in user_queues:
        await user_queues[user_id].put(notification)
        return {"status": "sent", "notification": notification}
    else:
        return {"status": "user_not_connected", "user_id": user_id}

@app.post("/notifications/broadcast")
async def broadcast_notification(message: str, type: str = "announcement"):
    """ส่ง notification ให้ทุก user ที่ online"""
    notification = {
        "id": str(uuid.uuid4()),
        "type": type,
        "message": message,
        "timestamp": datetime.now().isoformat()
    }
    
    sent_to = []
    for user_id, queue in user_queues.items():
        await queue.put(notification)
        sent_to.append(user_id)
    
    return {"status": "broadcast", "sent_to": sent_to, "count": len(sent_to)}
```

---

## 12. ตัวอย่างโปรแกรมจริง

### 12.1 Real-time Chat Application

**ตัวอย่างที่ 21: Complete Chat Application**

```python
# chat_app.py - Complete real-time chat app
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, HTTPException
from fastapi.responses import HTMLResponse
from pydantic import BaseModel
from typing import Dict, List, Optional, Set
from datetime import datetime
import json
import uuid

app = FastAPI(title="Real-time Chat App")

# Data Models
class User(BaseModel):
    user_id: str
    username: str
    avatar_color: str = "#4CAF50"

class Message(BaseModel):
    message_id: str
    room_id: str
    user_id: str
    username: str
    content: str
    timestamp: str
    message_type: str = "text"  # text, system, file

class ChatRoom(BaseModel):
    room_id: str
    name: str
    description: str = ""
    created_at: str
    member_count: int = 0

# In-memory storage
users: Dict[str, User] = {}
rooms: Dict[str, ChatRoom] = {
    "general": ChatRoom(
        room_id="general",
        name="General",
        description="General discussion",
        created_at=datetime.now().isoformat()
    ),
    "dev": ChatRoom(
        room_id="dev",
        name="Development",
        description="Tech talk",
        created_at=datetime.now().isoformat()
    )
}
room_messages: Dict[str, List[Message]] = {"general": [], "dev": []}
room_connections: Dict[str, Dict[str, WebSocket]] = {"general": {}, "dev": {}}

async def broadcast_to_room(room_id: str, data: dict, exclude_user: str = None):
    """ส่ง message ให้ทุกคนในห้อง"""
    if room_id not in room_connections:
        return
    
    dead_users = []
    for uid, ws in room_connections[room_id].items():
        if uid == exclude_user:
            continue
        try:
            await ws.send_json(data)
        except Exception:
            dead_users.append(uid)
    
    for uid in dead_users:
        del room_connections[room_id][uid]

@app.post("/users/register")
async def register_user(username: str):
    user_id = str(uuid.uuid4())[:8]
    colors = ["#E91E63", "#9C27B0", "#3F51B5", "#2196F3", "#009688", "#FF5722"]
    import random
    user = User(
        user_id=user_id,
        username=username,
        avatar_color=random.choice(colors)
    )
    users[user_id] = user
    return user

@app.get("/rooms")
async def get_rooms():
    result = []
    for room in rooms.values():
        room_dict = room.dict()
        room_dict["member_count"] = len(room_connections.get(room.room_id, {}))
        result.append(room_dict)
    return result

@app.get("/rooms/{room_id}/messages")
async def get_messages(room_id: str, limit: int = 50):
    if room_id not in room_messages:
        raise HTTPException(status_code=404, detail="Room not found")
    messages = room_messages[room_id]
    return messages[-limit:]

@app.websocket("/ws/chat/{user_id}/{room_id}")
async def chat_websocket(websocket: WebSocket, user_id: str, room_id: str):
    if user_id not in users:
        await websocket.close(code=4001, reason="User not registered")
        return
    
    if room_id not in rooms:
        await websocket.close(code=4004, reason="Room not found")
        return
    
    user = users[user_id]
    
    if room_id not in room_connections:
        room_connections[room_id] = {}
    
    await websocket.accept()
    room_connections[room_id][user_id] = websocket
    
    # ส่ง recent messages ให้ user ใหม่
    recent = room_messages.get(room_id, [])[-20:]
    await websocket.send_json({
        "type": "history",
        "messages": [m.dict() for m in recent],
        "room": rooms[room_id].dict(),
        "online_users": list(room_connections[room_id].keys())
    })
    
    # แจ้งคนอื่นว่า user เข้ามา
    system_msg = Message(
        message_id=str(uuid.uuid4()),
        room_id=room_id,
        user_id="system",
        username="System",
        content=f"{user.username} joined the room",
        timestamp=datetime.now().isoformat(),
        message_type="system"
    )
    room_messages[room_id].append(system_msg)
    
    await broadcast_to_room(room_id, {
        "type": "message",
        "message": system_msg.dict()
    }, exclude_user=user_id)
    
    try:
        while True:
            data = await websocket.receive_json()
            
            if data.get("type") == "message":
                msg = Message(
                    message_id=str(uuid.uuid4()),
                    room_id=room_id,
                    user_id=user_id,
                    username=user.username,
                    content=data.get("content", ""),
                    timestamp=datetime.now().isoformat()
                )
                
                room_messages[room_id].append(msg)
                if len(room_messages[room_id]) > 1000:
                    room_messages[room_id] = room_messages[room_id][-1000:]
                
                await broadcast_to_room(room_id, {
                    "type": "message",
                    "message": msg.dict()
                })
            
            elif data.get("type") == "typing":
                await broadcast_to_room(room_id, {
                    "type": "typing",
                    "user_id": user_id,
                    "username": user.username,
                    "is_typing": data.get("is_typing", False)
                }, exclude_user=user_id)
    
    except WebSocketDisconnect:
        del room_connections[room_id][user_id]
        
        leave_msg = Message(
            message_id=str(uuid.uuid4()),
            room_id=room_id,
            user_id="system",
            username="System",
            content=f"{user.username} left the room",
            timestamp=datetime.now().isoformat(),
            message_type="system"
        )
        room_messages[room_id].append(leave_msg)
        
        await broadcast_to_room(room_id, {
            "type": "message",
            "message": leave_msg.dict()
        })

# HTML Chat Interface
@app.get("/chat")
async def chat_interface():
    html = """
    <!DOCTYPE html>
    <html>
    <head>
        <title>Real-time Chat</title>
        <style>
            body { font-family: Arial; max-width: 800px; margin: 0 auto; padding: 20px; }
            #messages { height: 400px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; }
            .message { margin: 5px 0; }
            .system { color: gray; font-style: italic; }
            input[type=text] { width: 80%; padding: 5px; }
            button { padding: 5px 10px; }
        </style>
    </head>
    <body>
        <h1>Real-time Chat</h1>
        <div>
            <input id="username" placeholder="Username" />
            <input id="room" placeholder="Room (general/dev)" value="general" />
            <button onclick="connect()">Connect</button>
        </div>
        <div id="messages"></div>
        <div>
            <input id="msg" placeholder="Type a message..." onkeydown="if(event.key==='Enter') sendMessage()" />
            <button onclick="sendMessage()">Send</button>
        </div>
        <script>
            let ws = null;
            let userId = null;
            
            async function connect() {
                const username = document.getElementById('username').value;
                const room = document.getElementById('room').value;
                
                const resp = await fetch(`/users/register?username=${username}`, {method: 'POST'});
                const user = await resp.json();
                userId = user.user_id;
                
                ws = new WebSocket(`ws://localhost:8000/ws/chat/${userId}/${room}`);
                ws.onmessage = (e) => handleMessage(JSON.parse(e.data));
            }
            
            function handleMessage(data) {
                const msgs = document.getElementById('messages');
                if (data.type === 'message') {
                    const m = data.message;
                    const div = document.createElement('div');
                    div.className = 'message ' + (m.message_type === 'system' ? 'system' : '');
                    div.textContent = m.message_type === 'system' ? m.content : `[${m.username}]: ${m.content}`;
                    msgs.appendChild(div);
                    msgs.scrollTop = msgs.scrollHeight;
                }
                if (data.type === 'history') {
                    data.messages.forEach(m => handleMessage({type: 'message', message: m}));
                }
            }
            
            function sendMessage() {
                const input = document.getElementById('msg');
                if (ws && input.value) {
                    ws.send(JSON.stringify({type: 'message', content: input.value}));
                    input.value = '';
                }
            }
        </script>
    </body>
    </html>
    """
    return HTMLResponse(html)
```

### 12.2 Live Notifications System

**ตัวอย่างที่ 22: Live Notifications กับ SSE + Background Task**

```python
# notifications_app.py
from fastapi import FastAPI, BackgroundTasks, Request
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from typing import Dict, List, Optional
from datetime import datetime
import asyncio
import json
import uuid

app = FastAPI(title="Live Notifications System")

# Notification store
notification_queues: Dict[str, asyncio.Queue] = {}
notification_history: Dict[str, List[dict]] = {}

class NotificationRequest(BaseModel):
    user_ids: List[str]
    title: str
    message: str
    type: str = "info"  # info, success, warning, error
    action_url: Optional[str] = None
    priority: int = 1  # 1=low, 2=medium, 3=high

async def deliver_notification(user_id: str, notification: dict):
    """ส่ง notification ให้ user คนเดียว"""
    if user_id in notification_queues:
        await notification_queues[user_id].put(notification)
    
    # เก็บ history
    if user_id not in notification_history:
        notification_history[user_id] = []
    notification_history[user_id].append(notification)
    
    # เก็บแค่ 100 อัน
    if len(notification_history[user_id]) > 100:
        notification_history[user_id] = notification_history[user_id][-100:]

async def notification_event_stream(request: Request, user_id: str):
    """SSE stream สำหรับ user คนนึง"""
    queue = asyncio.Queue()
    notification_queues[user_id] = queue
    
    try:
        yield f"event: connected\ndata: {json.dumps({'user_id': user_id, 'status': 'connected'})}\n\n"
        
        # ส่ง unread notifications จาก history
        history = notification_history.get(user_id, [])
        unread = [n for n in history if not n.get("read", False)]
        if unread:
            yield f"event: history\ndata: {json.dumps({'notifications': unread})}\n\n"
        
        heartbeat_count = 0
        while True:
            if await request.is_disconnected():
                break
            
            try:
                notification = await asyncio.wait_for(queue.get(), timeout=25.0)
                yield f"event: notification\ndata: {json.dumps(notification)}\n\n"
            
            except asyncio.TimeoutError:
                heartbeat_count += 1
                yield f"event: heartbeat\ndata: {json.dumps({'count': heartbeat_count})}\n\n"
    
    finally:
        if user_id in notification_queues:
            del notification_queues[user_id]

@app.get("/notifications/stream/{user_id}")
async def stream_user_notifications(request: Request, user_id: str):
    return StreamingResponse(
        notification_event_stream(request, user_id),
        media_type="text/event-stream",
        headers={"Cache-Control": "no-cache", "X-Accel-Buffering": "no"}
    )

@app.post("/notifications/send")
async def send_notification(
    notification_req: NotificationRequest,
    background_tasks: BackgroundTasks
):
    notification_id = str(uuid.uuid4())
    notification = {
        "id": notification_id,
        "title": notification_req.title,
        "message": notification_req.message,
        "type": notification_req.type,
        "priority": notification_req.priority,
        "action_url": notification_req.action_url,
        "timestamp": datetime.now().isoformat(),
        "read": False
    }
    
    async def send_to_all():
        for user_id in notification_req.user_ids:
            await deliver_notification(user_id, notification)
    
    background_tasks.add_task(send_to_all)
    
    return {
        "notification_id": notification_id,
        "sent_to": len(notification_req.user_ids),
        "status": "queued"
    }

@app.get("/notifications/history/{user_id}")
async def get_notification_history(user_id: str, unread_only: bool = False):
    history = notification_history.get(user_id, [])
    if unread_only:
        history = [n for n in history if not n.get("read", False)]
    return {"user_id": user_id, "notifications": history}

@app.patch("/notifications/{user_id}/{notification_id}/read")
async def mark_as_read(user_id: str, notification_id: str):
    history = notification_history.get(user_id, [])
    for n in history:
        if n["id"] == notification_id:
            n["read"] = True
            return {"status": "marked_as_read"}
    return {"status": "not_found"}
```

### 12.3 Background Email Sender

**ตัวอย่างที่ 23: Background Email Sender ที่สมบูรณ์**

```python
# email_sender.py
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel, EmailStr
from typing import List, Optional, Dict
from datetime import datetime
import asyncio
import uuid

app = FastAPI(title="Background Email Sender")

# Email job tracking
email_jobs: Dict[str, dict] = {}

class EmailJob(BaseModel):
    job_id: str
    status: str  # queued, sending, sent, failed
    recipients: List[str]
    subject: str
    sent_count: int = 0
    failed_count: int = 0
    created_at: str
    completed_at: Optional[str] = None
    error: Optional[str] = None

class BulkEmailRequest(BaseModel):
    recipients: List[str]
    subject: str
    body: str
    template: Optional[str] = None
    variables: Optional[Dict[str, str]] = None

class SingleEmailRequest(BaseModel):
    to: str
    subject: str
    body: str
    cc: Optional[List[str]] = None
    bcc: Optional[List[str]] = None
    attachments: Optional[List[str]] = None

async def simulate_send_email(to: str, subject: str, body: str) -> bool:
    """จำลองการส่ง email - คืนค่า True ถ้าสำเร็จ"""
    await asyncio.sleep(0.5)  # จำลอง SMTP delay
    
    # จำลอง failure rate 5%
    import random
    if random.random() < 0.05:
        raise Exception(f"SMTP error: failed to deliver to {to}")
    
    print(f"[EMAIL] Sent: To={to}, Subject='{subject}'")
    return True

async def process_bulk_email(job_id: str, recipients: List[str], subject: str, body: str):
    """Process bulk email ใน background"""
    job = email_jobs[job_id]
    job["status"] = "sending"
    
    # ส่งแบบ batch (5 ครั้งพร้อมกัน)
    batch_size = 5
    
    for i in range(0, len(recipients), batch_size):
        batch = recipients[i:i + batch_size]
        tasks = [simulate_send_email(email, subject, body) for email in batch]
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        for email, result in zip(batch, results):
            if isinstance(result, Exception):
                job["failed_count"] += 1
                job["errors"] = job.get("errors", [])
                job["errors"].append({"email": email, "error": str(result)})
            else:
                job["sent_count"] += 1
        
        # Update progress
        progress = (job["sent_count"] + job["failed_count"]) / len(recipients) * 100
        job["progress"] = round(progress, 1)
    
    job["status"] = "completed"
    job["completed_at"] = datetime.now().isoformat()
    print(f"[JOB {job_id}] Completed: {job['sent_count']} sent, {job['failed_count']} failed")

@app.post("/email/send-single")
async def send_single_email(email: SingleEmailRequest, background_tasks: BackgroundTasks):
    """ส่ง email เดี่ยว ใน background"""
    job_id = str(uuid.uuid4())[:8]
    email_jobs[job_id] = {
        "job_id": job_id,
        "status": "queued",
        "type": "single",
        "to": email.to,
        "subject": email.subject,
        "created_at": datetime.now().isoformat()
    }
    
    background_tasks.add_task(
        simulate_send_email,
        email.to,
        email.subject,
        email.body
    )
    
    return {"job_id": job_id, "status": "queued"}

@app.post("/email/send-bulk")
async def send_bulk_email(request: BulkEmailRequest, background_tasks: BackgroundTasks):
    """ส่ง bulk email ให้ผู้รับหลายคน"""
    if not request.recipients:
        return {"error": "No recipients provided"}
    
    job_id = str(uuid.uuid4())[:8]
    email_jobs[job_id] = {
        "job_id": job_id,
        "status": "queued",
        "type": "bulk",
        "total": len(request.recipients),
        "sent_count": 0,
        "failed_count": 0,
        "progress": 0,
        "subject": request.subject,
        "created_at": datetime.now().isoformat()
    }
    
    background_tasks.add_task(
        process_bulk_email,
        job_id,
        request.recipients,
        request.subject,
        request.body
    )
    
    return {
        "job_id": job_id,
        "total_recipients": len(request.recipients),
        "status": "queued",
        "check_status_url": f"/email/jobs/{job_id}"
    }

@app.get("/email/jobs/{job_id}")
async def get_email_job_status(job_id: str):
    """ตรวจสอบสถานะ email job"""
    if job_id not in email_jobs:
        return {"error": "Job not found"}
    return email_jobs[job_id]

@app.get("/email/jobs")
async def list_email_jobs():
    """ดู email jobs ทั้งหมด"""
    return {
        "total": len(email_jobs),
        "jobs": list(email_jobs.values())
    }
```

### 12.4 ตัวอย่างที่ 24: WebSocket + Celery Integration

```python
# realtime_tasks.py - WebSocket ติดตาม Celery task progress
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from celery import Celery
from celery.result import AsyncResult
import asyncio

app = FastAPI()
celery_app = Celery("app", broker="redis://localhost:6379/0", backend="redis://localhost:6379/0")

@celery_app.task(bind=True)
def long_running_task(self, data: dict):
    """Task ที่ใช้เวลานานและรายงาน progress"""
    import time
    total_steps = 10
    
    for step in range(total_steps):
        time.sleep(1)
        progress = (step + 1) / total_steps * 100
        
        # อัพเดท task state
        self.update_state(
            state="PROGRESS",
            meta={
                "step": step + 1,
                "total": total_steps,
                "progress": round(progress, 1),
                "message": f"Processing step {step + 1}/{total_steps}"
            }
        )
    
    return {"status": "completed", "result": "Task finished successfully", "data": data}

@app.post("/tasks/start")
async def start_task(data: dict):
    """เริ่ม task และคืน task_id"""
    task = long_running_task.delay(data)
    return {"task_id": task.id}

@app.websocket("/ws/task/{task_id}")
async def watch_task(websocket: WebSocket, task_id: str):
    """WebSocket ที่ติดตาม task progress แบบ real-time"""
    await websocket.accept()
    
    await websocket.send_json({
        "type": "watching",
        "task_id": task_id,
        "message": "Watching task progress..."
    })
    
    try:
        while True:
            result = AsyncResult(task_id, app=celery_app)
            
            status = result.status
            
            if status == "PENDING":
                await websocket.send_json({
                    "type": "status",
                    "status": "pending",
                    "message": "Task is waiting in queue..."
                })
            
            elif status == "PROGRESS":
                meta = result.info
                await websocket.send_json({
                    "type": "progress",
                    "status": "running",
                    "progress": meta.get("progress", 0),
                    "message": meta.get("message", ""),
                    "step": meta.get("step"),
                    "total": meta.get("total")
                })
            
            elif status == "SUCCESS":
                await websocket.send_json({
                    "type": "completed",
                    "status": "success",
                    "result": result.result
                })
                break
            
            elif status in ("FAILURE", "REVOKED"):
                await websocket.send_json({
                    "type": "failed",
                    "status": status.lower(),
                    "error": str(result.result)
                })
                break
            
            await asyncio.sleep(0.5)
    
    except WebSocketDisconnect:
        print(f"Client stopped watching task {task_id}")
```

### 12.5 ตัวอย่างที่ 25: Complete Production-ready Setup

```python
# production_setup.py
from fastapi import FastAPI, WebSocket, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
from apscheduler.schedulers.asyncio import AsyncIOScheduler
import asyncio
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

scheduler = AsyncIOScheduler()

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Modern startup/shutdown (แทน on_event)"""
    # Startup
    logger.info("Starting application...")
    
    # เริ่ม scheduler
    scheduler.add_job(
        periodic_cleanup,
        "interval",
        minutes=10,
        id="cleanup"
    )
    scheduler.start()
    logger.info("Scheduler started")
    
    yield  # Application running
    
    # Shutdown
    logger.info("Shutting down...")
    scheduler.shutdown()
    logger.info("Shutdown complete")

app = FastAPI(title="Production FastAPI", lifespan=lifespan)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

async def periodic_cleanup():
    logger.info("Running periodic cleanup...")

active_connections = {}

@app.websocket("/ws/{client_id}")
async def production_websocket(websocket: WebSocket, client_id: str):
    await websocket.accept()
    active_connections[client_id] = websocket
    
    try:
        while True:
            data = await websocket.receive_json()
            await websocket.send_json({"echo": data, "client": client_id})
    except Exception:
        pass
    finally:
        active_connections.pop(client_id, None)

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "active_connections": len(active_connections),
        "scheduler_running": scheduler.running,
        "scheduler_jobs": len(scheduler.get_jobs())
    }
```

---

## 13. แบบฝึกหัด

### แบบฝึกหัดที่ 1: WebSocket Echo Server

**โจทย์:** สร้าง WebSocket endpoint ที่รับข้อความและส่งกลับพร้อมตัวเลข timestamp และ message count

**เฉลย:**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from datetime import datetime

app = FastAPI()

@app.websocket("/ws/echo")
async def echo_server(websocket: WebSocket):
    await websocket.accept()
    count = 0
    
    try:
        while True:
            text = await websocket.receive_text()
            count += 1
            await websocket.send_json({
                "original": text,
                "echo": f"ECHO: {text}",
                "count": count,
                "timestamp": datetime.now().isoformat()
            })
    except WebSocketDisconnect:
        print(f"Client disconnected after {count} messages")
```

---

### แบบฝึกหัดที่ 2: Multi-room Chat

**โจทย์:** สร้างระบบ chat ที่รองรับหลาย rooms โดย user สามารถ switch room ได้โดยไม่ต้อง reconnect

**เฉลย:**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import Dict, Set
import json

app = FastAPI()

rooms: Dict[str, Set[WebSocket]] = {}
ws_room: Dict[WebSocket, str] = {}
ws_name: Dict[WebSocket, str] = {}

async def send_to_room(room: str, msg: dict, exclude: WebSocket = None):
    for ws in list(rooms.get(room, set())):
        if ws != exclude:
            try:
                await ws.send_json(msg)
            except:
                rooms[room].discard(ws)

@app.websocket("/ws/multiroom/{username}")
async def multiroom_ws(websocket: WebSocket, username: str):
    await websocket.accept()
    ws_name[websocket] = username
    
    # เข้าห้อง default
    current_room = "lobby"
    rooms.setdefault(current_room, set()).add(websocket)
    ws_room[websocket] = current_room
    await websocket.send_json({"info": f"You are in room: {current_room}"})
    
    try:
        while True:
            data = await websocket.receive_json()
            
            if data.get("cmd") == "switch":
                new_room = data.get("room", "lobby")
                
                # ออกจากห้องเก่า
                rooms.get(current_room, set()).discard(websocket)
                await send_to_room(current_room, {"info": f"{username} left"})
                
                # เข้าห้องใหม่
                current_room = new_room
                rooms.setdefault(current_room, set()).add(websocket)
                ws_room[websocket] = current_room
                await websocket.send_json({"info": f"Switched to room: {current_room}"})
                await send_to_room(current_room, {"info": f"{username} joined"}, exclude=websocket)
            
            elif data.get("cmd") == "msg":
                await send_to_room(current_room, {
                    "from": username,
                    "room": current_room,
                    "text": data.get("text", "")
                })
    
    except WebSocketDisconnect:
        rooms.get(current_room, set()).discard(websocket)
        ws_room.pop(websocket, None)
        ws_name.pop(websocket, None)
        await send_to_room(current_room, {"info": f"{username} disconnected"})
```

---

### แบบฝึกหัดที่ 3: Background Task ส่ง Email Batch

**โจทย์:** สร้าง API endpoint ที่รับรายชื่อ email แล้วส่ง batch email ใน background พร้อมรายงานสถานะ

**เฉลย:**

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from typing import List, Dict
import asyncio
import uuid
from datetime import datetime

app = FastAPI()

class BatchEmailRequest(BaseModel):
    emails: List[str]
    subject: str
    body: str

jobs: Dict[str, dict] = {}

async def send_batch(job_id: str, emails: List[str], subject: str, body: str):
    jobs[job_id]["status"] = "running"
    
    for i, email in enumerate(emails):
        await asyncio.sleep(0.3)  # จำลอง SMTP
        jobs[job_id]["sent"] = i + 1
        jobs[job_id]["progress"] = round((i + 1) / len(emails) * 100, 1)
        print(f"Sent to {email}")
    
    jobs[job_id]["status"] = "done"
    jobs[job_id]["finished_at"] = datetime.now().isoformat()

@app.post("/batch-email")
async def batch_email(req: BatchEmailRequest, bg: BackgroundTasks):
    job_id = str(uuid.uuid4())[:8]
    jobs[job_id] = {
        "job_id": job_id,
        "status": "queued",
        "total": len(req.emails),
        "sent": 0,
        "progress": 0,
        "created_at": datetime.now().isoformat()
    }
    bg.add_task(send_batch, job_id, req.emails, req.subject, req.body)
    return {"job_id": job_id, "total": len(req.emails)}

@app.get("/batch-email/{job_id}")
async def job_status(job_id: str):
    return jobs.get(job_id, {"error": "Not found"})
```

---

### แบบฝึกหัดที่ 4: Scheduled Health Check

**โจทย์:** สร้าง FastAPI app ที่มี scheduled health check ทุก 1 นาที โดยตรวจสอบ external services และบันทึก log

**เฉลย:**

```python
from fastapi import FastAPI
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from datetime import datetime
from typing import List, Dict
import asyncio

app = FastAPI()
scheduler = AsyncIOScheduler()

health_log: List[Dict] = []

async def check_service(name: str, url: str) -> dict:
    """ตรวจสอบ service (จำลอง)"""
    await asyncio.sleep(0.1)
    import random
    ok = random.random() > 0.1  # 90% uptime simulation
    return {"service": name, "url": url, "healthy": ok}

async def run_health_check():
    """ตรวจสอบ services ทั้งหมด"""
    services = [
        ("database", "postgres://localhost:5432"),
        ("cache", "redis://localhost:6379"),
        ("api", "https://api.example.com"),
    ]
    
    results = await asyncio.gather(*[check_service(n, u) for n, u in services])
    
    entry = {
        "timestamp": datetime.now().isoformat(),
        "results": results,
        "all_healthy": all(r["healthy"] for r in results)
    }
    
    health_log.append(entry)
    if len(health_log) > 100:
        health_log.pop(0)
    
    if not entry["all_healthy"]:
        unhealthy = [r["service"] for r in results if not r["healthy"]]
        print(f"WARNING: Unhealthy services: {unhealthy}")

@app.on_event("startup")
async def start():
    scheduler.add_job(run_health_check, "interval", minutes=1, id="health_check")
    scheduler.start()

@app.on_event("shutdown")
async def stop():
    scheduler.shutdown()

@app.get("/health-log")
async def get_health_log(limit: int = 10):
    return {"log": health_log[-limit:], "total_checks": len(health_log)}

@app.post("/health-check/run-now")
async def trigger_health_check():
    await run_health_check()
    return health_log[-1] if health_log else {}
```

---

### แบบฝึกหัดที่ 5: WebSocket Authentication

**โจทย์:** สร้าง WebSocket endpoint ที่ตรวจสอบ JWT token ผ่าน query parameter และปฏิเสธการเชื่อมต่อถ้า token ไม่ถูกต้อง

**เฉลย:**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, Query
from jose import jwt, JWTError
from datetime import datetime, timedelta

app = FastAPI()
SECRET = "my-secret-key"

def create_token(sub: str) -> str:
    return jwt.encode({"sub": sub, "exp": datetime.utcnow() + timedelta(hours=1)}, SECRET)

def verify_token(token: str):
    try:
        return jwt.decode(token, SECRET, algorithms=["HS256"])
    except JWTError:
        return None

@app.get("/token/{username}")
async def get_token(username: str):
    return {"token": create_token(username)}

@app.websocket("/ws/secure")
async def secure_ws(
    websocket: WebSocket,
    token: str = Query(..., description="JWT token")
):
    payload = verify_token(token)
    
    if not payload:
        # ปฏิเสธก่อน accept เลย
        await websocket.close(code=4001, reason="Unauthorized")
        return
    
    username = payload["sub"]
    await websocket.accept()
    await websocket.send_json({"event": "auth_ok", "user": username})
    
    try:
        while True:
            msg = await websocket.receive_text()
            await websocket.send_text(f"[{username}] Echo: {msg}")
    except WebSocketDisconnect:
        print(f"{username} disconnected")
```

---

### แบบฝึกหัดที่ 6: SSE Progress Tracker

**โจทย์:** สร้าง API ที่รับ job ยาวๆ แล้ว stream progress ผ่าน SSE ให้ client ติดตามได้

**เฉลย:**

```python
from fastapi import FastAPI, Request, BackgroundTasks
from fastapi.responses import StreamingResponse
from typing import Dict
import asyncio
import json
import uuid

app = FastAPI()

job_progress: Dict[str, asyncio.Queue] = {}
job_status: Dict[str, dict] = {}

async def simulate_long_job(job_id: str, steps: int):
    job_status[job_id]["status"] = "running"
    
    for i in range(steps):
        await asyncio.sleep(1)
        progress = (i + 1) / steps * 100
        update = {
            "step": i + 1,
            "total": steps,
            "progress": round(progress, 1),
            "message": f"Processing step {i + 1}"
        }
        job_status[job_id].update(update)
        
        if job_id in job_progress:
            await job_progress[job_id].put(update)
    
    final = {"status": "done", "progress": 100, "message": "Job completed!"}
    job_status[job_id].update(final)
    if job_id in job_progress:
        await job_progress[job_id].put(final)

@app.post("/jobs/start")
async def start_job(steps: int = 5, bg: BackgroundTasks = None):
    job_id = str(uuid.uuid4())[:8]
    job_status[job_id] = {"job_id": job_id, "status": "queued", "progress": 0}
    bg.add_task(simulate_long_job, job_id, steps)
    return {"job_id": job_id, "stream_url": f"/jobs/{job_id}/stream"}

async def progress_stream(request: Request, job_id: str):
    queue = asyncio.Queue()
    job_progress[job_id] = queue
    
    try:
        while True:
            if await request.is_disconnected():
                break
            try:
                update = await asyncio.wait_for(queue.get(), timeout=30)
                yield f"data: {json.dumps(update)}\n\n"
                if update.get("status") == "done":
                    break
            except asyncio.TimeoutError:
                yield f"data: {json.dumps({'heartbeat': True})}\n\n"
    finally:
        job_progress.pop(job_id, None)

@app.get("/jobs/{job_id}/stream")
async def stream_job_progress(request: Request, job_id: str):
    if job_id not in job_status:
        return {"error": "Job not found"}
    return StreamingResponse(
        progress_stream(request, job_id),
        media_type="text/event-stream"
    )
```

---

### แบบฝึกหัดที่ 7: Celery Task Status Dashboard

**โจทย์:** สร้าง FastAPI endpoint สำหรับ submit งานหลายชิ้นพร้อมกัน และ endpoint สำหรับดู status ทั้งหมด

**เฉลย:**

```python
# Requires: pip install celery redis
from fastapi import FastAPI
from celery import Celery
from celery.result import AsyncResult
from typing import List
import time

app = FastAPI()
celery = Celery("tasks", broker="redis://localhost:6379/0", backend="redis://localhost:6379/0")

@celery.task
def compute_task(n: int) -> dict:
    time.sleep(n)
    return {"computed": n * n, "took_seconds": n}

@app.post("/tasks/submit-batch")
async def submit_batch(counts: List[int]):
    """Submit หลาย tasks พร้อมกัน"""
    task_ids = []
    for n in counts:
        task = compute_task.delay(n)
        task_ids.append(task.id)
    return {"task_ids": task_ids, "count": len(task_ids)}

@app.get("/tasks/status-batch")
async def batch_status(task_ids: str):
    """ตรวจสอบ status ของหลาย tasks พร้อมกัน (task_ids คั่นด้วย comma)"""
    ids = [t.strip() for t in task_ids.split(",")]
    results = []
    
    for task_id in ids:
        r = AsyncResult(task_id, app=celery)
        results.append({
            "task_id": task_id,
            "status": r.status,
            "result": r.result if r.ready() else None
        })
    
    done = sum(1 for r in results if r["status"] == "SUCCESS")
    return {
        "total": len(results),
        "done": done,
        "pending": len(results) - done,
        "tasks": results
    }
```

---

### แบบฝึกหัดที่ 8: Complete Real-time Dashboard

**โจทย์:** สร้าง dashboard ที่แสดงข้อมูล real-time (server stats) ผ่าน WebSocket พร้อม REST API สำหรับ query historical data

**เฉลย:**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse
from typing import List, Dict
from datetime import datetime
import asyncio
import random
import json

app = FastAPI(title="Real-time Dashboard")

# เก็บ stats history
stats_history: List[dict] = []
active_ws: List[WebSocket] = []

async def collect_stats() -> dict:
    """จำลองการเก็บ server stats"""
    return {
        "timestamp": datetime.now().isoformat(),
        "cpu": round(random.uniform(10, 90), 1),
        "memory": round(random.uniform(40, 80), 1),
        "requests_per_sec": random.randint(50, 500),
        "active_users": len(active_ws),
        "error_rate": round(random.uniform(0, 5), 2)
    }

async def stats_broadcaster():
    """Background task ที่ broadcast stats ทุกวินาที"""
    while True:
        await asyncio.sleep(1)
        stats = await collect_stats()
        stats_history.append(stats)
        
        if len(stats_history) > 300:  # เก็บ 5 นาที
            stats_history.pop(0)
        
        dead = []
        for ws in active_ws:
            try:
                await ws.send_json({"type": "stats", "data": stats})
            except:
                dead.append(ws)
        
        for ws in dead:
            active_ws.remove(ws)

@app.on_event("startup")
async def startup():
    asyncio.create_task(stats_broadcaster())

@app.websocket("/ws/dashboard")
async def dashboard_ws(websocket: WebSocket):
    await websocket.accept()
    active_ws.append(websocket)
    
    # ส่ง history ทันที
    await websocket.send_json({
        "type": "history",
        "data": stats_history[-60:]  # 1 นาทีล่าสุด
    })
    
    try:
        while True:
            # รอ command จาก client
            data = await websocket.receive_json()
            if data.get("cmd") == "get_history":
                await websocket.send_json({
                    "type": "history",
                    "data": stats_history[-int(data.get("minutes", 1)) * 60:]
                })
    except WebSocketDisconnect:
        active_ws.remove(websocket)

@app.get("/stats/history")
async def get_history(minutes: int = 5):
    """REST API สำหรับดู historical stats"""
    limit = minutes * 60
    return {
        "minutes": minutes,
        "data_points": len(stats_history[-limit:]),
        "stats": stats_history[-limit:]
    }

@app.get("/stats/summary")
async def get_summary():
    """สรุป stats เฉลี่ย"""
    if not stats_history:
        return {"error": "No data yet"}
    
    recent = stats_history[-60:] or stats_history
    return {
        "period": f"Last {len(recent)} seconds",
        "avg_cpu": round(sum(s["cpu"] for s in recent) / len(recent), 1),
        "avg_memory": round(sum(s["memory"] for s in recent) / len(recent), 1),
        "avg_rps": round(sum(s["requests_per_sec"] for s in recent) / len(recent)),
        "active_connections": len(active_ws)
    }

@app.get("/dashboard")
async def dashboard_ui():
    html = """<!DOCTYPE html>
<html><head><title>Real-time Dashboard</title>
<style>
  body { font-family: Arial; background: #1a1a2e; color: #e0e0e0; padding: 20px; }
  .metric { display: inline-block; background: #16213e; padding: 20px; margin: 10px; border-radius: 8px; min-width: 150px; text-align: center; }
  .metric h2 { margin: 0; font-size: 2em; color: #0f3460; color: #e94560; }
  .metric p { margin: 5px 0 0; color: #aaa; }
  h1 { color: #e94560; }
</style></head>
<body>
  <h1>Real-time Server Dashboard</h1>
  <div id="metrics">
    <div class="metric"><h2 id="cpu">-</h2><p>CPU %</p></div>
    <div class="metric"><h2 id="mem">-</h2><p>Memory %</p></div>
    <div class="metric"><h2 id="rps">-</h2><p>Req/sec</p></div>
    <div class="metric"><h2 id="users">-</h2><p>Active Users</p></div>
    <div class="metric"><h2 id="err">-</h2><p>Error Rate %</p></div>
  </div>
  <script>
    const ws = new WebSocket("ws://localhost:8000/ws/dashboard");
    ws.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      if (msg.type === "stats") {
        const d = msg.data;
        document.getElementById("cpu").textContent = d.cpu + "%";
        document.getElementById("mem").textContent = d.memory + "%";
        document.getElementById("rps").textContent = d.requests_per_sec;
        document.getElementById("users").textContent = d.active_users;
        document.getElementById("err").textContent = d.error_rate + "%";
      }
    };
  </script>
</body></html>"""
    return HTMLResponse(html)
```

---

## สรุปสิ่งที่เรียนรู้

ใน Part 61 เราได้เรียนรู้เกี่ยวกับ:

| หัวข้อ | สิ่งที่ทำได้ |
|--------|-------------|
| **WebSocket** | สร้าง real-time bidirectional communication |
| **Connection Lifecycle** | จัดการ connect, message, disconnect อย่างถูกต้อง |
| **Broadcasting** | ส่งข้อความหาหลาย clients พร้อมกัน |
| **Room Management** | แบ่งกลุ่ม users ตาม rooms/channels |
| **WS Authentication** | ป้องกัน endpoint ด้วย JWT tokens |
| **BackgroundTasks** | รัน tasks หลัง response ส่งกลับแล้ว |
| **Celery** | Distributed task queue สำหรับงานหนัก |
| **APScheduler** | Scheduled/Cron tasks ใน FastAPI |
| **SSE** | One-way streaming สำหรับ live updates |

### คำสั่งที่ใช้บ่อย

```bash
# รัน FastAPI app
uvicorn main:app --reload --port 8000

# รัน Celery worker
celery -A celery_config.celery_app worker --loglevel=info --concurrency=4

# รัน Celery beat (scheduler)
celery -A celery_config.celery_app beat --loglevel=info

# Flower monitoring
celery -A celery_config.celery_app flower

# รัน Redis (Docker)
docker run -d -p 6379:6379 redis:alpine

# Test WebSocket ด้วย wscat
npx wscat -c "ws://localhost:8000/ws/1"
```

### Best Practices

1. **WebSocket**: ใช้ `WebSocketDisconnect` exception handling เสมอ
2. **ConnectionManager**: ระวัง dead connections — ลบออกเมื่อ send ล้มเหลว
3. **Authentication**: Validate token ก่อน `accept()` ถ้าทำได้
4. **BackgroundTasks**: ใช้สำหรับงานเล็กๆ เร็วๆ — ถ้างานหนักให้ใช้ Celery
5. **Celery**: ตั้ง `max_retries` และ `countdown` สำหรับ retry logic
6. **APScheduler**: ใช้ `replace_existing=True` เพื่อป้องกัน duplicate jobs
7. **SSE**: ส่ง heartbeat เป็นระยะเพื่อรักษา connection
8. **Production**: ใช้ `lifespan` context manager แทน `on_event` (deprecated)

---

*Part 61 - FastAPI WebSockets & Background Tasks | Python Course*
