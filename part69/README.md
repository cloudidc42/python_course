# Part 69: WebSockets & Real-time Applications

## บทนำ

WebSocket เป็น protocol ที่ทำให้เกิด bidirectional communication ระหว่าง client และ server แบบ persistent connection ซึ่งต่างจาก HTTP ที่เป็น request-response model WebSocket ช่วยให้สร้าง real-time features อย่าง chat, live dashboard, collaborative editing ได้อย่างมีประสิทธิภาพ

## สารบัญ

1. [WebSocket Protocol](#1-websocket-protocol)
2. [Django Channels](#2-django-channels)
3. [Channel Layers (Redis)](#3-channel-layers-redis)
4. [ASGI](#4-asgi)
5. [WebSocket Consumers](#5-websocket-consumers)
6. [Group Communication](#6-group-communication)
7. [Django Channels Authentication](#7-django-channels-authentication)
8. [Presence Detection](#8-presence-detection)
9. [websockets Library](#9-websockets-library)
10. [SocketIO](#10-socketio)
11. [ตัวอย่างโปรแกรม](#11-ตัวอย่างโปรแกรม)
12. [แบบฝึกหัด](#12-แบบฝึกหัด)

---

## 1. WebSocket Protocol

### ความเข้าใจ WebSocket Protocol

```
HTTP (Traditional):
Client --> Request --> Server
Client <-- Response <-- Server
(Connection closed)

WebSocket:
Client --> HTTP Upgrade Request --> Server
Client <-- 101 Switching Protocols <-- Server
Client <------- Full Duplex -------> Server
(Connection stays open)
```

### WebSocket Handshake

```http
# Client ส่ง HTTP Upgrade Request
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# Server ตอบด้วย 101 Switching Protocols
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### WebSocket Frame Format

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)    |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - -+-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

### WebSocket ด้วย JavaScript (Client Side)

```javascript
// ตัวอย่างที่ 1: WebSocket Client ใน JavaScript
const ws = new WebSocket('ws://localhost:8000/ws/chat/room1/');

// Connection opened
ws.onopen = function(event) {
    console.log('Connected to WebSocket');
    
    // ส่ง message
    ws.send(JSON.stringify({
        type: 'chat_message',
        message: 'Hello, World!'
    }));
};

// รับ message
ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Received:', data);
};

// Connection closed
ws.onclose = function(event) {
    console.log('WebSocket closed:', event.code, event.reason);
};

// Error
ws.onerror = function(error) {
    console.error('WebSocket error:', error);
};

// ปิด connection
// ws.close();

// ReadyState values:
// 0 = CONNECTING
// 1 = OPEN
// 2 = CLOSING
// 3 = CLOSED
```

### WebSocket ด้วย Python (Client Side)

```python
# ตัวอย่างที่ 2: WebSocket Client ด้วย websockets library
# pip install websockets

import asyncio
import websockets
import json

async def websocket_client():
    uri = "ws://localhost:8000/ws/chat/room1/"
    
    async with websockets.connect(uri) as websocket:
        print(f"Connected to {uri}")
        
        # ส่ง message
        await websocket.send(json.dumps({
            'type': 'chat_message',
            'message': 'Hello from Python client!'
        }))
        
        # รับ messages
        async for message in websocket:
            data = json.loads(message)
            print(f"Received: {data}")
            
            if data.get('type') == 'system_message':
                if data.get('message') == 'CLOSE':
                    break

asyncio.run(websocket_client())
```

---

## 2. Django Channels

### Installation

```bash
pip install channels
pip install channels_redis  # สำหรับ Redis channel layer
pip install daphne          # ASGI server
```

### Basic Setup

```python
# ตัวอย่างที่ 3: Django Channels Setup
# settings.py
INSTALLED_APPS = [
    'daphne',  # ต้องอยู่ก่อน django.contrib.staticfiles
    'channels',
    # ... other apps
]

# ASGI Application
ASGI_APPLICATION = 'myproject.asgi.application'

# Channel Layers (ใช้ In-memory สำหรับ development)
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels.layers.InMemoryChannelLayer',
    }
}

# สำหรับ Production ใช้ Redis
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('127.0.0.1', 6379)],
        },
    },
}
```

---

## 3. Channel Layers (Redis)

### Redis Channel Layer Configuration

```python
# ตัวอย่างที่ 4: Redis Channel Layer
# settings.py
import os

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            # Single Redis instance
            'hosts': [(os.getenv('REDIS_HOST', 'localhost'), 6379)],
            
            # Redis with password
            # 'hosts': ['redis://:password@localhost:6379/0'],
            
            # Redis Sentinel (HA)
            # 'hosts': [
            #     {
            #         'sentinels': [('sentinel1', 26379), ('sentinel2', 26379)],
            #         'master_name': 'mymaster',
            #     }
            # ],
            
            # Settings
            'capacity': 1000,       # Max messages per channel
            'expiry': 60,           # Message expiry (seconds)
            'group_expiry': 86400,  # Group expiry (24 hours)
            'channel_capacity': {   # Per-channel capacity
                'http.request': 200,
                'http.response!*': 10,
            },
        },
    },
}
```

---

## 4. ASGI

### ASGI Application Setup

```python
# ตัวอย่างที่ 5: ASGI Application
# myproject/asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
from channels.security.websocket import AllowedHostsOriginValidator

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

django_asgi_app = get_asgi_application()

from chat.routing import websocket_urlpatterns

application = ProtocolTypeRouter({
    # Django HTTP requests
    'http': django_asgi_app,
    
    # WebSocket connections
    'websocket': AllowedHostsOriginValidator(
        AuthMiddlewareStack(
            URLRouter(websocket_urlpatterns)
        )
    ),
})
```

```python
# ตัวอย่างที่ 6: WebSocket URL Routing
# chat/routing.py
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer.as_asgi()),
    re_path(r'ws/notifications/$', consumers.NotificationConsumer.as_asgi()),
    re_path(r'ws/dashboard/$', consumers.DashboardConsumer.as_asgi()),
    re_path(r'ws/presence/$', consumers.PresenceConsumer.as_asgi()),
]
```

---

## 5. WebSocket Consumers

### Basic WebSocket Consumer

```python
# ตัวอย่างที่ 7: Basic WebSocket Consumer
# chat/consumers.py
import json
from channels.generic.websocket import WebsocketConsumer, AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from asgiref.sync import async_to_sync

class BasicChatConsumer(WebsocketConsumer):
    """Synchronous WebSocket Consumer"""
    
    def connect(self):
        """Called when WebSocket connection is opened"""
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        
        # Join room group
        async_to_sync(self.channel_layer.group_add)(
            self.room_group_name,
            self.channel_name
        )
        
        # Accept connection
        self.accept()
        
        print(f"User connected to room: {self.room_name}")
    
    def disconnect(self, close_code):
        """Called when WebSocket connection is closed"""
        # Leave room group
        async_to_sync(self.channel_layer.group_discard)(
            self.room_group_name,
            self.channel_name
        )
        
        print(f"User disconnected: {close_code}")
    
    def receive(self, text_data=None, bytes_data=None):
        """Called when message is received from WebSocket"""
        if text_data:
            data = json.loads(text_data)
            message = data.get('message', '')
            
            # Send message to room group
            async_to_sync(self.channel_layer.group_send)(
                self.room_group_name,
                {
                    'type': 'chat_message',
                    'message': message,
                    'username': str(self.scope['user']),
                }
            )
    
    def chat_message(self, event):
        """Receive message from room group"""
        # Send message to WebSocket
        self.send(text_data=json.dumps({
            'type': 'message',
            'message': event['message'],
            'username': event['username'],
        }))
```

### Async Consumer

```python
# ตัวอย่างที่ 8: Async WebSocket Consumer (แนะนำ)
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.utils import timezone

class ChatConsumer(AsyncWebsocketConsumer):
    """Async WebSocket Consumer สำหรับ Chat"""
    
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        self.user = self.scope['user']
        
        # ตรวจสอบ authentication
        if not self.user.is_authenticated:
            await self.close(code=4001)
            return
        
        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        
        await self.accept()
        
        # แจ้ง room ว่า user เข้ามา
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'user_join',
                'username': self.user.username,
                'timestamp': timezone.now().isoformat(),
            }
        )
        
        # ส่ง message history
        messages = await self.get_recent_messages()
        await self.send(text_data=json.dumps({
            'type': 'message_history',
            'messages': messages
        }))
    
    async def disconnect(self, close_code):
        # แจ้ง room ว่า user ออกไป
        if hasattr(self, 'user') and self.user.is_authenticated:
            await self.channel_layer.group_send(
                self.room_group_name,
                {
                    'type': 'user_leave',
                    'username': self.user.username,
                    'timestamp': timezone.now().isoformat(),
                }
            )
        
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive(self, text_data=None, bytes_data=None):
        if not text_data:
            return
        
        try:
            data = json.loads(text_data)
            message_type = data.get('type', 'message')
            
            if message_type == 'message':
                await self.handle_message(data)
            elif message_type == 'typing':
                await self.handle_typing(data)
            elif message_type == 'ping':
                await self.send(text_data=json.dumps({'type': 'pong'}))
        except json.JSONDecodeError:
            await self.send(text_data=json.dumps({
                'type': 'error',
                'message': 'Invalid JSON'
            }))
    
    async def handle_message(self, data):
        """จัดการ chat message"""
        message = data.get('message', '').strip()
        
        if not message:
            return
        
        if len(message) > 2000:
            await self.send(text_data=json.dumps({
                'type': 'error',
                'message': 'Message too long (max 2000 characters)'
            }))
            return
        
        # Save to database
        message_obj = await self.save_message(message)
        
        # Broadcast to room
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'id': message_obj.id,
                'message': message,
                'username': self.user.username,
                'user_id': self.user.id,
                'timestamp': message_obj.created_at.isoformat(),
            }
        )
    
    async def handle_typing(self, data):
        """จัดการ typing indicator"""
        is_typing = data.get('is_typing', False)
        
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'typing_indicator',
                'username': self.user.username,
                'is_typing': is_typing,
            }
        )
    
    # Event handlers สำหรับรับ messages จาก channel layer
    async def chat_message(self, event):
        """รับ chat message จาก group"""
        await self.send(text_data=json.dumps({
            'type': 'message',
            'id': event.get('id'),
            'message': event['message'],
            'username': event['username'],
            'user_id': event.get('user_id'),
            'timestamp': event.get('timestamp'),
        }))
    
    async def user_join(self, event):
        """รับ user join notification"""
        await self.send(text_data=json.dumps({
            'type': 'user_join',
            'username': event['username'],
            'timestamp': event['timestamp'],
        }))
    
    async def user_leave(self, event):
        """รับ user leave notification"""
        await self.send(text_data=json.dumps({
            'type': 'user_leave',
            'username': event['username'],
            'timestamp': event['timestamp'],
        }))
    
    async def typing_indicator(self, event):
        """รับ typing indicator"""
        if event['username'] != self.user.username:
            await self.send(text_data=json.dumps({
                'type': 'typing',
                'username': event['username'],
                'is_typing': event['is_typing'],
            }))
    
    # Database methods
    @database_sync_to_async
    def save_message(self, message):
        from .models import ChatMessage, ChatRoom
        room, _ = ChatRoom.objects.get_or_create(name=self.room_name)
        return ChatMessage.objects.create(
            room=room,
            author=self.user,
            content=message
        )
    
    @database_sync_to_async
    def get_recent_messages(self, limit=50):
        from .models import ChatMessage, ChatRoom
        try:
            room = ChatRoom.objects.get(name=self.room_name)
            messages = ChatMessage.objects.filter(
                room=room
            ).select_related('author').order_by('-created_at')[:limit]
            
            return [{
                'id': msg.id,
                'message': msg.content,
                'username': msg.author.username,
                'timestamp': msg.created_at.isoformat(),
            } for msg in reversed(list(messages))]
        except ChatRoom.DoesNotExist:
            return []
```

### Models

```python
# ตัวอย่างที่ 9: Chat Models
# chat/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class ChatRoom(models.Model):
    name = models.CharField(max_length=100, unique=True)
    description = models.TextField(blank=True)
    is_private = models.BooleanField(default=False)
    members = models.ManyToManyField(User, blank=True, related_name='chat_rooms')
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.name

class ChatMessage(models.Model):
    room = models.ForeignKey(ChatRoom, on_delete=models.CASCADE, related_name='messages')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    is_edited = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f"{self.author}: {self.content[:50]}"
```

---

## 6. Group Communication

### Broadcasting Messages

```python
# ตัวอย่างที่ 10: Group Communication
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

# ส่ง message ไปยัง group จาก Django view (synchronous)
def send_notification_to_user(user_id, message):
    channel_layer = get_channel_layer()
    
    async_to_sync(channel_layer.group_send)(
        f"user_{user_id}_notifications",
        {
            'type': 'notification',
            'message': message,
        }
    )

# ส่ง message จาก Django signal
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Comment)
def notify_article_author(sender, instance, created, **kwargs):
    """แจ้งเตือน author เมื่อมี comment ใหม่"""
    if created:
        channel_layer = get_channel_layer()
        author_id = instance.article.author.id
        
        async_to_sync(channel_layer.group_send)(
            f"user_{author_id}_notifications",
            {
                'type': 'notification',
                'notification_type': 'new_comment',
                'message': f"{instance.author.username} commented on your article",
                'article_title': instance.article.title,
                'article_id': instance.article.id,
            }
        )

# Consumer สำหรับ notifications
class NotificationConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.user = self.scope['user']
        
        if not self.user.is_authenticated:
            await self.close(code=4001)
            return
        
        self.group_name = f"user_{self.user.id}_notifications"
        
        await self.channel_layer.group_add(
            self.group_name,
            self.channel_name
        )
        
        await self.accept()
        
        # ส่ง unread notifications
        unread = await self.get_unread_notifications()
        if unread:
            await self.send(text_data=json.dumps({
                'type': 'unread_notifications',
                'notifications': unread
            }))
    
    async def disconnect(self, close_code):
        if hasattr(self, 'group_name'):
            await self.channel_layer.group_discard(
                self.group_name,
                self.channel_name
            )
    
    async def receive(self, text_data=None, bytes_data=None):
        if text_data:
            data = json.loads(text_data)
            if data.get('type') == 'mark_read':
                notification_id = data.get('notification_id')
                await self.mark_notification_read(notification_id)
    
    async def notification(self, event):
        """Receive notification from channel layer"""
        await self.send(text_data=json.dumps({
            'type': 'notification',
            'notification_type': event.get('notification_type', 'general'),
            'message': event['message'],
            'data': {k: v for k, v in event.items() if k not in ['type', 'message']},
            'timestamp': timezone.now().isoformat(),
        }))
    
    @database_sync_to_async
    def get_unread_notifications(self):
        from .models import Notification
        notifications = Notification.objects.filter(
            user=self.user,
            is_read=False
        ).order_by('-created_at')[:20]
        
        return [{
            'id': n.id,
            'type': n.notification_type,
            'message': n.message,
            'created_at': n.created_at.isoformat(),
        } for n in notifications]
    
    @database_sync_to_async
    def mark_notification_read(self, notification_id):
        from .models import Notification
        Notification.objects.filter(
            id=notification_id,
            user=self.user
        ).update(is_read=True)
```

---

## 7. Django Channels Authentication

### JWT Authentication Middleware

```python
# ตัวอย่างที่ 11: JWT Auth Middleware สำหรับ Channels
# chat/middleware.py
from channels.middleware import BaseMiddleware
from channels.db import database_sync_to_async
from django.contrib.auth.models import AnonymousUser

@database_sync_to_async
def get_user_from_jwt(token_str):
    """Validate JWT token และ return user"""
    try:
        from rest_framework_simplejwt.tokens import AccessToken
        token = AccessToken(token_str)
        user_id = token.payload.get('user_id')
        
        from django.contrib.auth import get_user_model
        User = get_user_model()
        return User.objects.get(pk=user_id)
    except Exception:
        return AnonymousUser()

class JWTAuthMiddleware(BaseMiddleware):
    """Middleware สำหรับ JWT authentication ใน WebSocket"""
    
    async def __call__(self, scope, receive, send):
        # ดึง token จาก query string: ws://localhost/ws/chat/?token=xxx
        from urllib.parse import parse_qs
        query_string = scope.get('query_string', b'').decode()
        params = parse_qs(query_string)
        
        token = None
        
        # ลอง query param ก่อน
        if 'token' in params:
            token = params['token'][0]
        else:
            # ลอง headers
            headers = dict(scope.get('headers', []))
            auth_header = headers.get(b'authorization', b'').decode()
            if auth_header.startswith('Bearer '):
                token = auth_header[7:]
        
        if token:
            scope['user'] = await get_user_from_jwt(token)
        else:
            scope['user'] = AnonymousUser()
        
        return await super().__call__(scope, receive, send)

# asgi.py - ใช้ JWTAuthMiddleware
from .chat.middleware import JWTAuthMiddleware

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AllowedHostsOriginValidator(
        JWTAuthMiddleware(
            URLRouter(websocket_urlpatterns)
        )
    ),
})
```

### Session Authentication

```python
# ตัวอย่างที่ 12: Session Authentication (built-in)
# channels.auth.AuthMiddlewareStack รองรับ session auth
# asgi.py
from channels.auth import AuthMiddlewareStack

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AllowedHostsOriginValidator(
        AuthMiddlewareStack(  # ใช้ Django session
            URLRouter(websocket_urlpatterns)
        )
    ),
})

# ใน consumer
class SecureConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        user = self.scope['user']  # Django User object
        
        if user.is_anonymous:
            await self.close()
            return
        
        await self.accept()
```

---

## 8. Presence Detection

### Online/Offline Status

```python
# ตัวอย่างที่ 13: Presence Detection
import json
from django.core.cache import cache
from channels.generic.websocket import AsyncWebsocketConsumer

class PresenceConsumer(AsyncWebsocketConsumer):
    """Consumer สำหรับ track online/offline status"""
    
    PRESENCE_GROUP = 'presence_updates'
    
    async def connect(self):
        self.user = self.scope['user']
        
        if not self.user.is_authenticated:
            await self.close()
            return
        
        self.user_channel = f"user_{self.user.id}_presence"
        
        # เข้า presence group
        await self.channel_layer.group_add(
            self.PRESENCE_GROUP,
            self.channel_name
        )
        
        await self.accept()
        
        # Mark user as online
        await self.set_user_online(True)
        
        # Broadcast presence update
        await self.channel_layer.group_send(
            self.PRESENCE_GROUP,
            {
                'type': 'presence_update',
                'user_id': self.user.id,
                'username': self.user.username,
                'status': 'online',
            }
        )
        
        # ส่ง online users list
        online_users = await self.get_online_users()
        await self.send(text_data=json.dumps({
            'type': 'online_users',
            'users': online_users
        }))
    
    async def disconnect(self, close_code):
        if hasattr(self, 'user') and self.user.is_authenticated:
            await self.set_user_online(False)
            
            await self.channel_layer.group_send(
                self.PRESENCE_GROUP,
                {
                    'type': 'presence_update',
                    'user_id': self.user.id,
                    'username': self.user.username,
                    'status': 'offline',
                }
            )
        
        await self.channel_layer.group_discard(
            self.PRESENCE_GROUP,
            self.channel_name
        )
    
    async def receive(self, text_data=None, bytes_data=None):
        if text_data:
            data = json.loads(text_data)
            
            if data.get('type') == 'heartbeat':
                # Update last seen
                await self.update_last_seen()
                await self.send(text_data=json.dumps({
                    'type': 'heartbeat_ack'
                }))
    
    async def presence_update(self, event):
        """Broadcast presence update"""
        await self.send(text_data=json.dumps({
            'type': 'presence_update',
            'user_id': event['user_id'],
            'username': event['username'],
            'status': event['status'],
        }))
    
    @database_sync_to_async
    def set_user_online(self, is_online):
        """Set user online status ใน cache"""
        cache_key = f"user_online_{self.user.id}"
        
        if is_online:
            cache.set(cache_key, True, timeout=60)  # Expire หลัง 60 วินาที
        else:
            cache.delete(cache_key)
        
        # Update database
        from django.contrib.auth import get_user_model
        User = get_user_model()
        User.objects.filter(pk=self.user.id).update(
            last_active=timezone.now()
        )
    
    @database_sync_to_async
    def get_online_users(self):
        """ดึง list of online users"""
        from django.contrib.auth import get_user_model
        User = get_user_model()
        
        users = User.objects.filter(is_active=True)
        online = []
        
        for user in users:
            if cache.get(f"user_online_{user.id}"):
                online.append({
                    'id': user.id,
                    'username': user.username,
                    'status': 'online'
                })
        
        return online
    
    @database_sync_to_async
    def update_last_seen(self):
        from django.contrib.auth import get_user_model
        User = get_user_model()
        User.objects.filter(pk=self.user.id).update(last_active=timezone.now())
        
        # Refresh cache expiry
        cache_key = f"user_online_{self.user.id}"
        cache.set(cache_key, True, timeout=60)
```

---

## 9. websockets Library

```python
# ตัวอย่างที่ 14: Pure Python WebSocket Server ด้วย websockets
# pip install websockets
import asyncio
import websockets
import json
from datetime import datetime

# Simple echo server
async def echo_handler(websocket, path):
    """WebSocket echo server"""
    client_info = f"{websocket.remote_address}"
    print(f"Client connected: {client_info}")
    
    try:
        async for message in websocket:
            print(f"Received from {client_info}: {message}")
            
            # Echo back
            response = {
                'type': 'echo',
                'original': message,
                'timestamp': datetime.now().isoformat()
            }
            await websocket.send(json.dumps(response))
    
    except websockets.exceptions.ConnectionClosed:
        print(f"Client disconnected: {client_info}")
    except Exception as e:
        print(f"Error: {e}")

async def main():
    server = await websockets.serve(
        echo_handler,
        "localhost",
        8765,
        ping_interval=30,
        ping_timeout=10,
    )
    print("WebSocket server started on ws://localhost:8765")
    await server.wait_closed()

asyncio.run(main())
```

```python
# ตัวอย่างที่ 15: Chat Server ด้วย websockets
import asyncio
import websockets
import json
from datetime import datetime
from typing import Set, Dict

class ChatRoom:
    """Chat room manager"""
    
    def __init__(self, name: str):
        self.name = name
        self.clients: Dict[str, websockets.WebSocketServerProtocol] = {}
    
    async def join(self, username: str, websocket):
        self.clients[username] = websocket
        await self.broadcast({
            'type': 'user_join',
            'username': username,
            'users': list(self.clients.keys()),
            'timestamp': datetime.now().isoformat()
        })
    
    async def leave(self, username: str):
        if username in self.clients:
            del self.clients[username]
        await self.broadcast({
            'type': 'user_leave',
            'username': username,
            'users': list(self.clients.keys()),
            'timestamp': datetime.now().isoformat()
        })
    
    async def broadcast(self, message: dict, exclude: str = None):
        if not self.clients:
            return
        
        message_str = json.dumps(message)
        
        # ส่งหาทุก client ยกเว้น exclude
        coros = [
            ws.send(message_str)
            for username, ws in self.clients.items()
            if username != exclude
        ]
        
        if coros:
            await asyncio.gather(*coros, return_exceptions=True)

# Global chat rooms
rooms: Dict[str, ChatRoom] = {}

async def chat_handler(websocket, path):
    username = None
    room = None
    
    try:
        # รอ join message
        init_message = await websocket.recv()
        data = json.loads(init_message)
        
        if data.get('type') != 'join':
            await websocket.send(json.dumps({
                'type': 'error',
                'message': 'Must join a room first'
            }))
            return
        
        username = data.get('username', 'Anonymous')
        room_name = data.get('room', 'general')
        
        # สร้าง room ถ้าไม่มี
        if room_name not in rooms:
            rooms[room_name] = ChatRoom(room_name)
        
        room = rooms[room_name]
        await room.join(username, websocket)
        
        # Main message loop
        async for message in websocket:
            data = json.loads(message)
            
            if data.get('type') == 'message':
                await room.broadcast({
                    'type': 'message',
                    'username': username,
                    'message': data.get('message', ''),
                    'timestamp': datetime.now().isoformat()
                })
    
    except websockets.exceptions.ConnectionClosed:
        pass
    except Exception as e:
        print(f"Error: {e}")
    finally:
        if username and room:
            await room.leave(username)

async def main():
    async with websockets.serve(chat_handler, "localhost", 8765):
        print("Chat server started on ws://localhost:8765")
        await asyncio.Future()  # Run forever

asyncio.run(main())
```

---

## 10. SocketIO

```bash
pip install python-socketio
pip install python-socketio[asyncio_client]
pip install aiohttp  # หรือ uvicorn
```

```python
# ตัวอย่างที่ 16: Socket.IO Server ด้วย python-socketio
import socketio
import asyncio
from datetime import datetime

# Create Socket.IO server
sio = socketio.AsyncServer(
    async_mode='asgi',
    cors_allowed_origins=['http://localhost:3000'],
    ping_timeout=60,
    ping_interval=25,
)

# Connect to ASGI app
from aiohttp import web
app = web.Application()
sio.attach(app)

# Connected users
users = {}
rooms = {}

@sio.event
async def connect(sid, environ):
    """Client connected"""
    print(f"Client connected: {sid}")
    
    await sio.emit('welcome', {
        'sid': sid,
        'timestamp': datetime.now().isoformat()
    }, to=sid)

@sio.event
async def disconnect(sid):
    """Client disconnected"""
    username = users.pop(sid, 'Unknown')
    print(f"Client disconnected: {username} ({sid})")
    
    # Remove from all rooms
    for room_name, room_users in rooms.items():
        if sid in room_users:
            room_users.discard(sid)
            await sio.emit('user_left', {
                'username': username,
                'room': room_name
            }, room=room_name)

@sio.event
async def join_room(sid, data):
    """Join a chat room"""
    username = data.get('username', 'Anonymous')
    room = data.get('room', 'general')
    
    users[sid] = username
    
    if room not in rooms:
        rooms[room] = set()
    rooms[room].add(sid)
    
    # Join Socket.IO room
    sio.enter_room(sid, room)
    
    # Notify room
    await sio.emit('user_joined', {
        'username': username,
        'room': room,
        'users': len(rooms[room])
    }, room=room)
    
    print(f"{username} joined room: {room}")

@sio.event
async def send_message(sid, data):
    """Send message to room"""
    username = users.get(sid, 'Unknown')
    room = data.get('room', 'general')
    message = data.get('message', '')
    
    if not message.strip():
        return
    
    await sio.emit('new_message', {
        'username': username,
        'message': message,
        'room': room,
        'timestamp': datetime.now().isoformat()
    }, room=room)

@sio.event
async def typing(sid, data):
    """User is typing"""
    username = users.get(sid, 'Unknown')
    room = data.get('room', 'general')
    is_typing = data.get('is_typing', False)
    
    await sio.emit('user_typing', {
        'username': username,
        'is_typing': is_typing
    }, room=room, skip_sid=sid)

if __name__ == '__main__':
    web.run_app(app, host='localhost', port=8080)
```

```python
# ตัวอย่างที่ 17: Socket.IO กับ Django
# pip install django-socketio
import socketio
from django.conf import settings

# Create async Socket.IO server
sio = socketio.AsyncServer(
    async_mode='asgi',
    cors_allowed_origins=settings.CORS_ALLOWED_ORIGINS
)

# Django ASGI app
from django.core.asgi import get_asgi_application
django_app = get_asgi_application()

# Combine Django + Socket.IO
application = socketio.ASGIApp(sio, django_app)
```

---

## 11. ตัวอย่างโปรแกรม

### Real-time Chat App (สมบูรณ์)

```python
# ตัวอย่างที่ 18: Complete Chat App Consumer
# chat/consumers.py
import json
import re
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.utils import timezone
from django.core.cache import cache

class FullChatConsumer(AsyncWebsocketConsumer):
    """Complete chat consumer พร้อม features ครบครัน"""
    
    MAX_MESSAGE_LENGTH = 2000
    TYPING_TIMEOUT = 3  # วินาที
    
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        self.user = self.scope['user']
        
        if not self.user.is_authenticated:
            await self.close(code=4001)
            return
        
        # ตรวจสอบสิทธิ์เข้าถึง room
        can_join = await self.can_join_room()
        if not can_join:
            await self.close(code=4003)
            return
        
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        
        # Track presence
        await self.mark_online()
        
        await self.accept()
        
        # Broadcast join
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat.join',
                'user_id': self.user.id,
                'username': self.user.username,
                'avatar': await self.get_user_avatar(),
                'timestamp': timezone.now().isoformat(),
            }
        )
        
        # Send history และ online users
        history = await self.get_message_history()
        online_users = await self.get_online_users()
        
        await self.send(text_data=json.dumps({
            'type': 'init',
            'history': history,
            'online_users': online_users,
            'room': {
                'name': self.room_name,
                'member_count': len(online_users)
            }
        }))
    
    async def disconnect(self, close_code):
        if not hasattr(self, 'user') or not self.user.is_authenticated:
            return
        
        await self.mark_offline()
        
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat.leave',
                'user_id': self.user.id,
                'username': self.user.username,
                'timestamp': timezone.now().isoformat(),
            }
        )
        
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive(self, text_data=None, bytes_data=None):
        if not text_data:
            return
        
        try:
            data = json.loads(text_data)
        except json.JSONDecodeError:
            await self.send_error('Invalid JSON format')
            return
        
        msg_type = data.get('type')
        handlers = {
            'message': self.handle_message,
            'typing': self.handle_typing,
            'read': self.handle_read_receipt,
            'reaction': self.handle_reaction,
            'delete': self.handle_delete_message,
            'ping': self.handle_ping,
        }
        
        handler = handlers.get(msg_type)
        if handler:
            await handler(data)
        else:
            await self.send_error(f'Unknown message type: {msg_type}')
    
    async def handle_message(self, data):
        message = data.get('message', '').strip()
        
        if not message:
            return
        
        if len(message) > self.MAX_MESSAGE_LENGTH:
            await self.send_error(f'Message too long (max {self.MAX_MESSAGE_LENGTH} chars)')
            return
        
        # Filter malicious content
        message = self.sanitize_message(message)
        
        # Save to DB
        msg_obj = await self.save_message(message)
        
        # Broadcast
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat.message',
                'id': msg_obj.id,
                'message': message,
                'user_id': self.user.id,
                'username': self.user.username,
                'timestamp': msg_obj.created_at.isoformat(),
            }
        )
    
    async def handle_typing(self, data):
        is_typing = data.get('is_typing', False)
        
        cache_key = f"typing_{self.room_name}_{self.user.id}"
        
        if is_typing:
            cache.set(cache_key, True, self.TYPING_TIMEOUT)
        else:
            cache.delete(cache_key)
        
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat.typing',
                'user_id': self.user.id,
                'username': self.user.username,
                'is_typing': is_typing,
            }
        )
    
    async def handle_read_receipt(self, data):
        message_id = data.get('message_id')
        if message_id:
            await self.mark_message_read(message_id)
    
    async def handle_reaction(self, data):
        message_id = data.get('message_id')
        emoji = data.get('emoji')
        
        if not message_id or not emoji:
            return
        
        reaction = await self.toggle_reaction(message_id, emoji)
        
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat.reaction',
                'message_id': message_id,
                'emoji': emoji,
                'user_id': self.user.id,
                'username': self.user.username,
                'action': reaction['action'],
            }
        )
    
    async def handle_delete_message(self, data):
        message_id = data.get('message_id')
        
        success = await self.delete_message(message_id)
        
        if success:
            await self.channel_layer.group_send(
                self.room_group_name,
                {
                    'type': 'chat.delete',
                    'message_id': message_id,
                    'user_id': self.user.id,
                }
            )
    
    async def handle_ping(self, data):
        await self.send(text_data=json.dumps({'type': 'pong'}))
    
    # Event handlers
    async def chat_message(self, event):
        await self.send(text_data=json.dumps({
            'type': 'message',
            'id': event['id'],
            'message': event['message'],
            'user_id': event['user_id'],
            'username': event['username'],
            'timestamp': event['timestamp'],
        }))
    
    async def chat_join(self, event):
        await self.send(text_data=json.dumps({
            'type': 'user_join',
            'user_id': event['user_id'],
            'username': event['username'],
            'avatar': event.get('avatar'),
            'timestamp': event['timestamp'],
        }))
    
    async def chat_leave(self, event):
        await self.send(text_data=json.dumps({
            'type': 'user_leave',
            'user_id': event['user_id'],
            'username': event['username'],
            'timestamp': event['timestamp'],
        }))
    
    async def chat_typing(self, event):
        if event['user_id'] != self.user.id:
            await self.send(text_data=json.dumps({
                'type': 'typing',
                'user_id': event['user_id'],
                'username': event['username'],
                'is_typing': event['is_typing'],
            }))
    
    async def chat_reaction(self, event):
        await self.send(text_data=json.dumps({
            'type': 'reaction',
            'message_id': event['message_id'],
            'emoji': event['emoji'],
            'user_id': event['user_id'],
            'username': event['username'],
            'action': event['action'],
        }))
    
    async def chat_delete(self, event):
        await self.send(text_data=json.dumps({
            'type': 'message_deleted',
            'message_id': event['message_id'],
        }))
    
    # Helper methods
    def sanitize_message(self, message):
        """ลบ HTML tags และ scripts"""
        import bleach
        allowed_tags = ['b', 'i', 'u', 'em', 'strong', 'code', 'pre']
        return bleach.clean(message, tags=allowed_tags, strip=True)
    
    async def send_error(self, message):
        await self.send(text_data=json.dumps({
            'type': 'error',
            'message': message
        }))
    
    # Database operations
    @database_sync_to_async
    def can_join_room(self):
        from .models import ChatRoom
        try:
            room = ChatRoom.objects.get(name=self.room_name)
            if room.is_private:
                return room.members.filter(pk=self.user.pk).exists()
            return True
        except ChatRoom.DoesNotExist:
            return True  # Public room ใหม่
    
    @database_sync_to_async
    def save_message(self, content):
        from .models import ChatMessage, ChatRoom
        room, _ = ChatRoom.objects.get_or_create(name=self.room_name)
        return ChatMessage.objects.create(
            room=room,
            author=self.user,
            content=content
        )
    
    @database_sync_to_async
    def get_message_history(self, limit=50):
        from .models import ChatMessage, ChatRoom
        try:
            room = ChatRoom.objects.get(name=self.room_name)
            messages = list(ChatMessage.objects.filter(
                room=room,
                is_deleted=False
            ).select_related('author').order_by('-created_at')[:limit])
            
            return [{
                'id': m.id,
                'message': m.content,
                'user_id': m.author.id,
                'username': m.author.username,
                'timestamp': m.created_at.isoformat(),
            } for m in reversed(messages)]
        except ChatRoom.DoesNotExist:
            return []
    
    @database_sync_to_async
    def get_online_users(self):
        from django.contrib.auth import get_user_model
        User = get_user_model()
        
        users = []
        cache_prefix = f"presence_{self.room_name}_"
        
        for user in User.objects.filter(is_active=True):
            if cache.get(f"{cache_prefix}{user.id}"):
                users.append({
                    'id': user.id,
                    'username': user.username,
                })
        return users
    
    @database_sync_to_async
    def mark_online(self):
        cache.set(
            f"presence_{self.room_name}_{self.user.id}",
            True,
            timeout=300
        )
    
    @database_sync_to_async
    def mark_offline(self):
        cache.delete(f"presence_{self.room_name}_{self.user.id}")
    
    @database_sync_to_async
    def get_user_avatar(self):
        if hasattr(self.user, 'profile') and self.user.profile.avatar:
            return self.user.profile.avatar.url
        return None
    
    @database_sync_to_async
    def mark_message_read(self, message_id):
        from .models import ChatMessage, MessageReadReceipt
        try:
            message = ChatMessage.objects.get(pk=message_id)
            MessageReadReceipt.objects.get_or_create(
                message=message,
                user=self.user
            )
        except ChatMessage.DoesNotExist:
            pass
    
    @database_sync_to_async
    def toggle_reaction(self, message_id, emoji):
        from .models import ChatMessage, MessageReaction
        try:
            message = ChatMessage.objects.get(pk=message_id)
            reaction, created = MessageReaction.objects.get_or_create(
                message=message,
                user=self.user,
                emoji=emoji
            )
            if not created:
                reaction.delete()
                return {'action': 'removed'}
            return {'action': 'added'}
        except ChatMessage.DoesNotExist:
            return {'action': 'error'}
    
    @database_sync_to_async
    def delete_message(self, message_id):
        from .models import ChatMessage
        try:
            message = ChatMessage.objects.get(
                pk=message_id,
                author=self.user
            )
            message.is_deleted = True
            message.content = '[Message deleted]'
            message.save()
            return True
        except ChatMessage.DoesNotExist:
            return False
```

### Live Dashboard Consumer

```python
# ตัวอย่างที่ 19: Live Dashboard Consumer
import json
import asyncio
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async

class DashboardConsumer(AsyncWebsocketConsumer):
    """Real-time dashboard สำหรับ admin"""
    
    async def connect(self):
        user = self.scope['user']
        
        if not user.is_authenticated or not user.is_staff:
            await self.close(code=4003)
            return
        
        await self.channel_layer.group_add('dashboard', self.channel_name)
        await self.accept()
        
        # ส่ง initial data
        stats = await self.get_stats()
        await self.send(text_data=json.dumps({
            'type': 'init',
            'stats': stats
        }))
        
        # Start periodic updates
        self.update_task = asyncio.ensure_future(self.send_periodic_updates())
    
    async def disconnect(self, close_code):
        if hasattr(self, 'update_task'):
            self.update_task.cancel()
        
        await self.channel_layer.group_discard('dashboard', self.channel_name)
    
    async def receive(self, text_data=None, bytes_data=None):
        if text_data:
            data = json.loads(text_data)
            if data.get('type') == 'request_stats':
                stats = await self.get_stats()
                await self.send(text_data=json.dumps({
                    'type': 'stats_update',
                    'stats': stats
                }))
    
    async def send_periodic_updates(self):
        """ส่ง stats ทุก 30 วินาที"""
        while True:
            try:
                await asyncio.sleep(30)
                stats = await self.get_stats()
                await self.send(text_data=json.dumps({
                    'type': 'stats_update',
                    'stats': stats
                }))
            except asyncio.CancelledError:
                break
            except Exception as e:
                print(f"Dashboard error: {e}")
    
    async def dashboard_update(self, event):
        """รับ real-time updates"""
        await self.send(text_data=json.dumps({
            'type': 'dashboard_update',
            'metric': event.get('metric'),
            'value': event.get('value'),
            'timestamp': event.get('timestamp'),
        }))
    
    @database_sync_to_async
    def get_stats(self):
        from django.contrib.auth import get_user_model
        from blog.models import Article, Comment
        from django.utils import timezone
        from datetime import timedelta
        
        User = get_user_model()
        now = timezone.now()
        
        return {
            'total_users': User.objects.count(),
            'new_users_today': User.objects.filter(
                date_joined__date=now.date()
            ).count(),
            'total_articles': Article.objects.count(),
            'published_articles': Article.objects.filter(
                status='published'
            ).count(),
            'total_comments': Comment.objects.count(),
            'comments_today': Comment.objects.filter(
                created_at__date=now.date()
            ).count(),
            'active_users': User.objects.filter(
                last_login__gte=now - timedelta(days=7)
            ).count(),
        }
```

### Collaborative Editor

```python
# ตัวอย่างที่ 20: Collaborative Text Editor
import json
import difflib
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async

class CollaborativeEditorConsumer(AsyncWebsocketConsumer):
    """Real-time collaborative text editor"""
    
    async def connect(self):
        self.document_id = self.scope['url_route']['kwargs']['document_id']
        self.doc_group = f'document_{self.document_id}'
        self.user = self.scope['user']
        
        if not self.user.is_authenticated:
            await self.close(code=4001)
            return
        
        # ตรวจสอบสิทธิ์
        can_edit = await self.check_permission()
        if not can_edit:
            await self.close(code=4003)
            return
        
        await self.channel_layer.group_add(self.doc_group, self.channel_name)
        await self.accept()
        
        # ส่ง document content
        doc = await self.get_document()
        await self.send(text_data=json.dumps({
            'type': 'document_init',
            'content': doc['content'],
            'version': doc['version'],
        }))
        
        # Announce presence
        await self.channel_layer.group_send(
            self.doc_group,
            {
                'type': 'editor_join',
                'user_id': self.user.id,
                'username': self.user.username,
            }
        )
    
    async def disconnect(self, close_code):
        await self.channel_layer.group_send(
            self.doc_group,
            {
                'type': 'editor_leave',
                'user_id': self.user.id,
                'username': self.user.username,
            }
        )
        await self.channel_layer.group_discard(self.doc_group, self.channel_name)
    
    async def receive(self, text_data=None, bytes_data=None):
        data = json.loads(text_data)
        
        if data['type'] == 'update':
            # Operational Transform หรือ simple diff
            success = await self.apply_update(
                data['content'],
                data['version'],
                data.get('cursor_pos')
            )
            
            if success:
                await self.channel_layer.group_send(
                    self.doc_group,
                    {
                        'type': 'content_update',
                        'content': data['content'],
                        'user_id': self.user.id,
                        'username': self.user.username,
                        'cursor_pos': data.get('cursor_pos'),
                    }
                )
        
        elif data['type'] == 'cursor_move':
            await self.channel_layer.group_send(
                self.doc_group,
                {
                    'type': 'cursor_update',
                    'user_id': self.user.id,
                    'username': self.user.username,
                    'cursor_pos': data.get('cursor_pos'),
                    'selection': data.get('selection'),
                }
            )
    
    async def content_update(self, event):
        if event['user_id'] != self.user.id:
            await self.send(text_data=json.dumps({
                'type': 'content_update',
                'content': event['content'],
                'user_id': event['user_id'],
                'username': event['username'],
            }))
    
    async def cursor_update(self, event):
        if event['user_id'] != self.user.id:
            await self.send(text_data=json.dumps({
                'type': 'cursor_update',
                'user_id': event['user_id'],
                'username': event['username'],
                'cursor_pos': event.get('cursor_pos'),
                'selection': event.get('selection'),
            }))
    
    async def editor_join(self, event):
        await self.send(text_data=json.dumps({
            'type': 'editor_joined',
            'user_id': event['user_id'],
            'username': event['username'],
        }))
    
    async def editor_leave(self, event):
        await self.send(text_data=json.dumps({
            'type': 'editor_left',
            'user_id': event['user_id'],
            'username': event['username'],
        }))
    
    @database_sync_to_async
    def get_document(self):
        from .models import Document
        doc = Document.objects.get(pk=self.document_id)
        return {'content': doc.content, 'version': doc.version}
    
    @database_sync_to_async
    def apply_update(self, new_content, version, cursor_pos):
        from .models import Document, DocumentHistory
        
        try:
            doc = Document.objects.get(pk=self.document_id)
            
            if doc.version != version:
                return False
            
            # Save history
            DocumentHistory.objects.create(
                document=doc,
                content=doc.content,
                edited_by=self.user,
                version=doc.version
            )
            
            # Update
            doc.content = new_content
            doc.version += 1
            doc.last_edited_by = self.user
            doc.save()
            
            return True
        except Exception:
            return False
    
    @database_sync_to_async
    def check_permission(self):
        from .models import Document
        try:
            doc = Document.objects.get(pk=self.document_id)
            return (
                doc.owner == self.user or
                doc.editors.filter(pk=self.user.pk).exists() or
                self.user.is_staff
            )
        except Document.DoesNotExist:
            return False
```

---

## 12. แบบฝึกหัด

### ข้อ 1: Simple Echo WebSocket Server

สร้าง WebSocket server ที่ echo messages กลับมา

**เฉลย:**

```python
# echo_server.py
import asyncio
import websockets
import json
from datetime import datetime

connected_clients = set()

async def echo_handler(websocket):
    connected_clients.add(websocket)
    client_id = id(websocket)
    
    print(f"Client {client_id} connected. Total: {len(connected_clients)}")
    
    try:
        await websocket.send(json.dumps({
            'type': 'welcome',
            'client_id': client_id,
            'message': 'Connected to Echo Server'
        }))
        
        async for message in websocket:
            try:
                data = json.loads(message)
                
                response = {
                    'type': 'echo',
                    'original': data,
                    'echo_at': datetime.now().isoformat(),
                    'client_id': client_id,
                }
                
                await websocket.send(json.dumps(response))
                print(f"Echo to {client_id}: {message[:100]}")
            
            except json.JSONDecodeError:
                await websocket.send(json.dumps({
                    'type': 'error',
                    'message': 'Invalid JSON'
                }))
    
    except websockets.exceptions.ConnectionClosed as e:
        print(f"Client {client_id} disconnected: {e.code}")
    finally:
        connected_clients.discard(websocket)

async def main():
    print("Starting Echo Server on ws://localhost:8765")
    async with websockets.serve(echo_handler, "localhost", 8765):
        await asyncio.Future()

asyncio.run(main())
```

### ข้อ 2: Notification System

สร้างระบบ notification แบบ real-time

**เฉลย:**

```python
# notifications/consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.utils import timezone

class NotificationConsumer(AsyncWebsocketConsumer):
    
    async def connect(self):
        user = self.scope.get('user')
        
        if not user or not user.is_authenticated:
            await self.close(code=4001)
            return
        
        self.user = user
        self.notification_group = f"notifications_{user.id}"
        
        await self.channel_layer.group_add(
            self.notification_group,
            self.channel_name
        )
        
        await self.accept()
        
        # ส่ง unread count
        count = await self.get_unread_count()
        await self.send(text_data=json.dumps({
            'type': 'unread_count',
            'count': count
        }))
    
    async def disconnect(self, close_code):
        if hasattr(self, 'notification_group'):
            await self.channel_layer.group_discard(
                self.notification_group,
                self.channel_name
            )
    
    async def receive(self, text_data=None, bytes_data=None):
        if not text_data:
            return
        
        data = json.loads(text_data)
        action = data.get('action')
        
        if action == 'mark_read':
            notification_id = data.get('notification_id')
            await self.mark_read(notification_id)
            
        elif action == 'mark_all_read':
            await self.mark_all_read()
            await self.send(text_data=json.dumps({
                'type': 'all_read'
            }))
        
        elif action == 'get_notifications':
            notifications = await self.get_notifications()
            await self.send(text_data=json.dumps({
                'type': 'notifications_list',
                'notifications': notifications
            }))
    
    async def notification(self, event):
        """รับ notification จาก channel layer"""
        await self.send(text_data=json.dumps({
            'type': 'notification',
            'id': event.get('id'),
            'notification_type': event.get('notification_type'),
            'title': event.get('title'),
            'message': event.get('message'),
            'url': event.get('url'),
            'timestamp': timezone.now().isoformat(),
        }))
    
    @database_sync_to_async
    def get_unread_count(self):
        from .models import Notification
        return Notification.objects.filter(
            user=self.user,
            is_read=False
        ).count()
    
    @database_sync_to_async
    def get_notifications(self, limit=20):
        from .models import Notification
        notifications = Notification.objects.filter(
            user=self.user
        ).order_by('-created_at')[:limit]
        
        return [{
            'id': n.id,
            'type': n.notification_type,
            'title': n.title,
            'message': n.message,
            'is_read': n.is_read,
            'url': n.url,
            'created_at': n.created_at.isoformat(),
        } for n in notifications]
    
    @database_sync_to_async
    def mark_read(self, notification_id):
        from .models import Notification
        Notification.objects.filter(
            id=notification_id,
            user=self.user
        ).update(is_read=True, read_at=timezone.now())
    
    @database_sync_to_async
    def mark_all_read(self):
        from .models import Notification
        Notification.objects.filter(
            user=self.user,
            is_read=False
        ).update(is_read=True, read_at=timezone.now())

# Helper function สำหรับส่ง notification จาก views
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

def send_notification(user_id, notification_type, title, message, url=None, notification_id=None):
    """ส่ง notification ไปยัง user"""
    channel_layer = get_channel_layer()
    
    async_to_sync(channel_layer.group_send)(
        f"notifications_{user_id}",
        {
            'type': 'notification',
            'id': notification_id,
            'notification_type': notification_type,
            'title': title,
            'message': message,
            'url': url,
        }
    )
```

### ข้อ 3: Live Stock Price Ticker

สร้าง real-time stock price display

**เฉลย:**

```python
# stocks/consumers.py
import json
import asyncio
import random
from channels.generic.websocket import AsyncWebsocketConsumer

class StockTickerConsumer(AsyncWebsocketConsumer):
    """Simulate real-time stock price updates"""
    
    STOCKS = {
        'AAPL': {'price': 150.00, 'change': 0},
        'GOOGL': {'price': 2800.00, 'change': 0},
        'MSFT': {'price': 380.00, 'change': 0},
        'AMZN': {'price': 3200.00, 'change': 0},
    }
    
    async def connect(self):
        await self.accept()
        self.subscribed_symbols = set()
        
        # ส่ง available stocks
        await self.send(text_data=json.dumps({
            'type': 'available_stocks',
            'stocks': list(self.STOCKS.keys())
        }))
    
    async def disconnect(self, close_code):
        pass
    
    async def receive(self, text_data=None, bytes_data=None):
        data = json.loads(text_data)
        
        if data.get('type') == 'subscribe':
            symbols = data.get('symbols', [])
            self.subscribed_symbols.update(symbols)
            
            # ส่ง current prices
            prices = {
                symbol: self.STOCKS.get(symbol)
                for symbol in self.subscribed_symbols
                if symbol in self.STOCKS
            }
            await self.send(text_data=json.dumps({
                'type': 'initial_prices',
                'prices': prices
            }))
            
            # Start streaming
            if not hasattr(self, 'stream_task'):
                self.stream_task = asyncio.ensure_future(self.stream_prices())
        
        elif data.get('type') == 'unsubscribe':
            symbols = data.get('symbols', [])
            self.subscribed_symbols.difference_update(symbols)
    
    async def stream_prices(self):
        while True:
            try:
                await asyncio.sleep(1)
                
                if not self.subscribed_symbols:
                    continue
                
                updates = {}
                for symbol in self.subscribed_symbols:
                    if symbol in self.STOCKS:
                        # Simulate price change
                        old_price = self.STOCKS[symbol]['price']
                        change_pct = random.uniform(-0.5, 0.5)
                        new_price = round(old_price * (1 + change_pct/100), 2)
                        change = round(new_price - old_price, 2)
                        
                        self.STOCKS[symbol]['price'] = new_price
                        self.STOCKS[symbol]['change'] = change
                        
                        updates[symbol] = {
                            'price': new_price,
                            'change': change,
                            'change_pct': round(change_pct, 2)
                        }
                
                if updates:
                    await self.send(text_data=json.dumps({
                        'type': 'price_update',
                        'updates': updates,
                        'timestamp': asyncio.get_event_loop().time()
                    }))
            
            except asyncio.CancelledError:
                break
```

### ข้อ 4: Typing Indicators

สร้าง typing indicator สำหรับ chat

**เฉลย:**

```python
# ตัวอย่างการ implement typing indicators
import json
import asyncio
from channels.generic.websocket import AsyncWebsocketConsumer
from django.core.cache import cache

class TypingAwareChatConsumer(AsyncWebsocketConsumer):
    TYPING_TIMEOUT = 3
    
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group = f'typing_chat_{self.room_name}'
        self.user = self.scope['user']
        
        if not self.user.is_authenticated:
            await self.close()
            return
        
        await self.channel_layer.group_add(self.room_group, self.channel_name)
        await self.accept()
        
        self.typing_task = None
    
    async def disconnect(self, close_code):
        if self.typing_task:
            self.typing_task.cancel()
        
        # Mark as stopped typing on disconnect
        await self.broadcast_typing(False)
        await self.channel_layer.group_discard(self.room_group, self.channel_name)
    
    async def receive(self, text_data=None, bytes_data=None):
        data = json.loads(text_data)
        
        if data['type'] == 'typing_start':
            await self.handle_typing_start()
        
        elif data['type'] == 'typing_stop':
            await self.handle_typing_stop()
        
        elif data['type'] == 'message':
            await self.handle_typing_stop()  # Auto-stop typing on send
            await self.handle_message(data['message'])
    
    async def handle_typing_start(self):
        # Cancel existing timeout
        if self.typing_task and not self.typing_task.done():
            self.typing_task.cancel()
        
        # Broadcast typing
        await self.broadcast_typing(True)
        
        # Set auto-stop timeout
        self.typing_task = asyncio.ensure_future(
            self.auto_stop_typing()
        )
    
    async def handle_typing_stop(self):
        if self.typing_task and not self.typing_task.done():
            self.typing_task.cancel()
        
        await self.broadcast_typing(False)
    
    async def auto_stop_typing(self):
        await asyncio.sleep(self.TYPING_TIMEOUT)
        await self.broadcast_typing(False)
    
    async def broadcast_typing(self, is_typing):
        await self.channel_layer.group_send(
            self.room_group,
            {
                'type': 'typing_status',
                'user_id': self.user.id,
                'username': self.user.username,
                'is_typing': is_typing,
            }
        )
    
    async def handle_message(self, message):
        await self.channel_layer.group_send(
            self.room_group,
            {
                'type': 'chat_message',
                'message': message,
                'username': self.user.username,
            }
        )
    
    async def typing_status(self, event):
        if event['user_id'] != self.user.id:
            await self.send(text_data=json.dumps({
                'type': 'typing',
                'username': event['username'],
                'is_typing': event['is_typing'],
            }))
    
    async def chat_message(self, event):
        await self.send(text_data=json.dumps({
            'type': 'message',
            'message': event['message'],
            'username': event['username'],
        }))
```

### ข้อ 5-8 (สรุป)

ข้อ 5: สร้าง Read receipts system  
ข้อ 6: สร้าง File sharing ผ่าน WebSocket  
ข้อ 7: สร้าง Multi-room support  
ข้อ 8: สร้าง Rate limiting สำหรับ WebSocket messages

```python
# ข้อ 8: Rate Limiting สำหรับ WebSocket
import asyncio
from collections import deque
from datetime import datetime

class RateLimitedConsumer(AsyncWebsocketConsumer):
    """WebSocket consumer พร้อม rate limiting"""
    
    RATE_LIMIT = 10  # messages
    RATE_WINDOW = 10  # วินาที
    
    async def connect(self):
        self.message_times = deque()
        await self.accept()
    
    async def receive(self, text_data=None, bytes_data=None):
        # Check rate limit
        if not await self.check_rate_limit():
            await self.send(text_data=json.dumps({
                'type': 'error',
                'message': 'Rate limit exceeded. Please slow down.',
                'retry_after': self.RATE_WINDOW
            }))
            return
        
        # Process message normally
        data = json.loads(text_data)
        await self.process_message(data)
    
    async def check_rate_limit(self):
        now = asyncio.get_event_loop().time()
        window_start = now - self.RATE_WINDOW
        
        # Remove old timestamps
        while self.message_times and self.message_times[0] < window_start:
            self.message_times.popleft()
        
        # Check limit
        if len(self.message_times) >= self.RATE_LIMIT:
            return False
        
        self.message_times.append(now)
        return True
    
    async def process_message(self, data):
        # Normal message processing
        await self.send(text_data=json.dumps({
            'type': 'ack',
            'message': 'Received'
        }))
```

---

## สรุป

WebSockets และ Real-time applications เป็น pattern สำคัญสำหรับ modern web apps:

1. **WebSocket Protocol** - Full-duplex communication ผ่าน single connection
2. **Django Channels** - ASGI-based extension สำหรับ WebSockets ใน Django
3. **Channel Layers** - Message passing ระหว่าง consumers ด้วย Redis
4. **ASGI** - Asynchronous Server Gateway Interface รองรับ async Django
5. **Consumers** - WebSocket handlers คล้าย Django views
6. **Group Communication** - Broadcasting ไปยังหลาย connections
7. **Authentication** - JWT หรือ Session auth สำหรับ WebSocket
8. **Presence Detection** - Track online/offline users
9. **websockets library** - Pure Python WebSocket implementation
10. **Socket.IO** - Higher-level WebSocket library with fallbacks

Use cases ที่เหมาะสม:
- Real-time chat
- Live notifications
- Live dashboard/analytics
- Collaborative editing
- Real-time games
- IoT data streaming

---

*หัวข้อถัดไป: Part 70 - Project: Full-Stack Blog API with FastAPI*
