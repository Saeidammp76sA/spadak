# Spadak v0.1 - Python Full-Stack Web Framework

🚀 **Spadak** is a modern, async-ready Python web framework that combines the power of server-side rendering, real-time WebSocket communication, and a built-in lightweight database - all without requiring Node.js!

## ✨ Features

- **🔥 Full-Stack Framework**: Backend + SSR + Real-time WebSocket support
- **💾 MiniDB**: Built-in key-value and JSON storage (file-based)
- **🔒 Security First**: CSRF, XSS, HSTS, X-Frame-Options protection
- **⚡ Async-Ready**: High concurrency with async/await support
- **🌐 Zero Node.js**: Pure Python solution
- **🛠️ Simple CLI**: Easy project creation and management
- **📱 Real-time**: WebSocket support for live applications
- **🎨 SSR Templates**: Custom `.spx` template engine

## 🚀 Quick Start

### Installation

```bash
pip install spadak
```

### Create Your First Project

```bash
# Create a new Spadak project
spadak new my-app
cd my-app

# Run the development server
spadak run
```

Your app will be available at `http://localhost:8000`

### Hello World Example

```python
from spadak import Spadak, route, render_template

app = Spadak()

@app.route("/")
async def home(request):
    return render_template("index.spx", {"message": "Hello, Spadak!"})

@app.route("/api/users")
async def get_users(request):
    return {"users": ["Alice", "Bob", "Charlie"]}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

## 📚 Core Components

### 1. ASGI Server & Routing

```python
from spadak import Spadak, route

app = Spadak()

# HTTP routes
@app.route("/", methods=["GET"])
async def index(request):
    return {"message": "Welcome to Spadak!"}

@app.route("/users/<user_id>", methods=["GET"])
async def get_user(request):
    user_id = request.path_params["user_id"]
    return {"user_id": user_id}

# POST route with form data
@app.route("/submit", methods=["POST"])
async def submit_form(request):
    form_data = await request.form()
    return {"received": dict(form_data)}
```

### 2. MiniDB - Built-in Database

```python
from spadak.db import MiniDB

# Initialize database
db = MiniDB("data/myapp.db")

# Key-value operations
await db.set("user:123", {"name": "Alice", "email": "alice@example.com"})
user = await db.get("user:123")

# JSON document storage
await db.set_json("config", {"theme": "dark", "language": "en"})
config = await db.get_json("config")

# Delete operations
await db.delete("user:123")

# List all keys
keys = await db.keys()
```

### 3. WebSocket Real-time Communication

```python
from spadak.ws import WebSocketManager

ws_manager = WebSocketManager()

@app.websocket("/ws")
async def websocket_endpoint(websocket):
    await ws_manager.connect(websocket)
    try:
        while True:
            message = await websocket.receive_text()
            # Broadcast to all connected clients
            await ws_manager.broadcast(f"User said: {message}")
    except:
        await ws_manager.disconnect(websocket)
```

### 4. Server-Side Rendering (SSR)

**Template file (`templates/index.spx`):**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ message }}</h1>
    <ul>
    {% for item in items %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>
</body>
</html>
```

**Python code:**
```python
from spadak.templates import render_template

@app.route("/")
async def home(request):
    context = {
        "title": "My Spadak App",
        "message": "Welcome!",
        "items": ["Feature 1", "Feature 2", "Feature 3"]
    }
    return render_template("index.spx", context)
```

### 5. Security Middleware

```python
from spadak.core.middleware import SecurityMiddleware

# Security is enabled by default
app = Spadak()

# Custom security configuration
app.add_middleware(SecurityMiddleware, {
    "csrf_protection": True,
    "xss_protection": True,
    "hsts_enabled": True,
    "frame_options": "DENY"
})
```

## 🎯 Real-time Chat Example

Here's a complete real-time chat application:

**`app.py`:**
```python
from spadak import Spadak
from spadak.db import MiniDB
from spadak.ws import WebSocketManager
from spadak.templates import render_template
import json
from datetime import datetime

app = Spadak()
db = MiniDB("chat.db")
ws_manager = WebSocketManager()

@app.route("/")
async def chat_home(request):
    # Get recent messages from database
    messages = await db.get("recent_messages") or []
    return render_template("chat.spx", {"messages": messages})

@app.websocket("/ws")
async def websocket_endpoint(websocket):
    await ws_manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            message_data = json.loads(data)
            
            # Create message object
            message = {
                "user": message_data["user"],
                "text": message_data["text"],
                "timestamp": datetime.now().isoformat()
            }
            
            # Store in database
            messages = await db.get("recent_messages") or []
            messages.append(message)
            # Keep only last 50 messages
            messages = messages[-50:]
            await db.set("recent_messages", messages)
            
            # Broadcast to all clients
            await ws_manager.broadcast(json.dumps(message))
    except:
        await ws_manager.disconnect(websocket)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

**`templates/chat.spx`:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Spadak Chat</title>
    <style>
        #messages { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
        .message { margin: 5px 0; }
        .user { font-weight: bold; color: #007bff; }
    </style>
</head>
<body>
    <h1>Real-time Chat with Spadak</h1>
    
    <div id="messages">
        {% for message in messages %}
        <div class="message">
            <span class="user">{{ message.user }}:</span>
            {{ message.text }}
            <small>({{ message.timestamp }})</small>
        </div>
        {% endfor %}
    </div>
    
    <div>
        <input type="text" id="username" placeholder="Your name" value="User">
        <input type="text" id="messageInput" placeholder="Type a message...">
        <button onclick="sendMessage()">Send</button>
    </div>
    
    <script>
        const ws = new WebSocket('ws://localhost:8000/ws');
        const messages = document.getElementById('messages');
        
        ws.onmessage = function(event) {
            const message = JSON.parse(event.data);
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message';
            messageDiv.innerHTML = `
                <span class="user">${message.user}:</span>
                ${message.text}
                <small>(${message.timestamp})</small>
            `;
            messages.appendChild(messageDiv);
            messages.scrollTop = messages.scrollHeight;
        };
        
        function sendMessage() {
            const username = document.getElementById('username').value;
            const messageInput = document.getElementById('messageInput');
            const text = messageInput.value;
            
            if (text.trim()) {
                ws.send(JSON.stringify({
                    user: username,
                    text: text
                }));
                messageInput.value = '';
            }
        }
        
        document.getElementById('messageInput').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });
    </script>
</body>
</html>
```

## 🛠️ CLI Commands

```bash
# Create a new project
spadak new my-project

# Run development server
spadak run

# Run with custom host and port
spadak run --host 0.0.0.0 --port 3000

# Run example applications
spadak run examples/chat

# Generate project templates
spadak generate component MyComponent
spadak generate template my-template
```

## 📁 Project Structure

When you create a new Spadak project:

```
my-project/
├── app.py              # Main application file
├── templates/          # SSR template files (.spx)
│   ├── base.spx
│   └── index.spx
├── static/             # Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── images/
├── data/               # MiniDB database files
├── config.py           # Configuration settings
└── requirements.txt    # Python dependencies
```

## 🔧 Configuration

**`config.py`:**
```python
class Config:
    DEBUG = True
    HOST = "0.0.0.0"
    PORT = 8000
    DATABASE_PATH = "data/app.db"
    TEMPLATE_DIR = "templates"
    STATIC_DIR = "static"
    
    # Security settings
    SECRET_KEY = "your-secret-key-here"
    CSRF_PROTECTION = True
    XSS_PROTECTION = True
    HSTS_ENABLED = True
    
class ProductionConfig(Config):
    DEBUG = False
    
class DevelopmentConfig(Config):
    DEBUG = True
```

## 🧪 Testing

```python
import pytest
from spadak.testing import TestClient
from myapp import app

@pytest.fixture
def client():
    return TestClient(app)

def test_home_page(client):
    response = client.get("/")
    assert response.status_code == 200
    assert "Welcome" in response.text

def test_api_endpoint(client):
    response = client.get("/api/users")
    assert response.status_code == 200
    data = response.json()
    assert "users" in data
```

## 🚀 Deployment

### Using Uvicorn (Recommended)

```bash
# Install uvicorn
pip install uvicorn[standard]

# Run in production
uvicorn app:app --host 0.0.0.0 --port 8000 --workers 4
```

### Using Docker

**`Dockerfile`:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 📖 API Reference

### Core Classes

- **`Spadak`**: Main application class
- **`MiniDB`**: Database operations
- **`WebSocketManager`**: WebSocket connection management
- **`render_template()`**: Template rendering function

### Decorators

- **`@app.route(path, methods=[])`**: HTTP route decorator
- **`@app.websocket(path)`**: WebSocket endpoint decorator
- **`@app.middleware`**: Custom middleware decorator

### Security Functions

- **`generate_csrf_token()`**: Generate CSRF token
- **`validate_csrf_token()`**: Validate CSRF token
- **`sanitize_html()`**: XSS protection

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

Spadak is released under the MIT License. See [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

- Inspired by FastAPI, Flask, and Django
- Built with modern Python async capabilities
- Designed for developer productivity and performance

---

**Made with ❤️ by the Spadak Team**

For more examples and documentation, visit our [GitHub repository](https://github.com/spadak/spadak).
