[上一章：Socket編程基礎](066_Socket編程基礎.md) | [下一章：網絡協議基礎](068_網絡協議基礎.md)

# Python Socket編程進階 🌐

## 非阻塞IO

### 1. 非阻塞Socket服務器

```python
import socket
import select
import threading
from typing import Dict, List, Set
import time

class NonBlockingServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8888):
        self.host = host
        self.port = port
        self.server_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
        self.server_socket.setsockopt(
            socket.SOL_SOCKET,
            socket.SO_REUSEADDR,
            1
        )
        # 設置為非阻塞模式
        self.server_socket.setblocking(False)
        
        self.clients: Set[socket.socket] = set()
        self.running = False
    
    def handle_client(self, client_socket: socket.socket):
        """處理客戶端連接"""
        try:
            while self.running:
                try:
                    # 非阻塞接收數據
                    data = client_socket.recv(1024)
                    if not data:
                        break
                    
                    # 處理數據
                    message = data.decode('utf-8')
                    print(f"Received: {message}")
                    
                    # 廣播消息
                    self.broadcast(message, exclude=client_socket)
                
                except BlockingIOError:
                    # 沒有數據可讀，繼續等待
                    time.sleep(0.1)
                except Exception as e:
                    print(f"Error handling client: {e}")
                    break
        
        finally:
            self.clients.remove(client_socket)
            client_socket.close()
            print("Client disconnected")
    
    def broadcast(self, message: str, exclude: socket.socket = None):
        """廣播消息給所有客戶端"""
        for client in self.clients:
            if client != exclude:
                try:
                    client.send(message.encode('utf-8'))
                except Exception as e:
                    print(f"Error broadcasting to client: {e}")
    
    def start(self):
        """啟動服務器"""
        try:
            self.server_socket.bind((self.host, self.port))
            self.server_socket.listen(5)
            self.running = True
            print(f"Server listening on {self.host}:{self.port}")
            
            while self.running:
                try:
                    # 非阻塞接受連接
                    client_socket, address = self.server_socket.accept()
                    client_socket.setblocking(False)
                    self.clients.add(client_socket)
                    print(f"New connection from {address}")
                    
                    # 創建新線程處理客戶端
                    thread = threading.Thread(
                        target=self.handle_client,
                        args=(client_socket,)
                    )
                    thread.start()
                
                except BlockingIOError:
                    # 沒有新的連接，繼續等待
                    time.sleep(0.1)
                except Exception as e:
                    print(f"Error accepting connection: {e}")
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.running = False
            for client in self.clients:
                client.close()
            self.server_socket.close()

# 使用示例
if __name__ == '__main__':
    server = NonBlockingServer()
    server.start()
```

### 2. 使用select實現多路複用

```python
import socket
import select
from typing import List, Dict, Set
import time

class SelectServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8888):
        self.host = host
        self.port = port
        self.server_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
        self.server_socket.setsockopt(
            socket.SOL_SOCKET,
            socket.SO_REUSEADDR,
            1
        )
        
        self.clients: Set[socket.socket] = set()
        self.running = False
    
    def handle_client(self, client_socket: socket.socket):
        """處理客戶端數據"""
        try:
            data = client_socket.recv(1024)
            if not data:
                return False
            
            message = data.decode('utf-8')
            print(f"Received: {message}")
            
            # 廣播消息
            self.broadcast(message, exclude=client_socket)
            return True
        
        except Exception as e:
            print(f"Error handling client: {e}")
            return False
    
    def broadcast(self, message: str, exclude: socket.socket = None):
        """廣播消息給所有客戶端"""
        for client in self.clients:
            if client != exclude:
                try:
                    client.send(message.encode('utf-8'))
                except Exception as e:
                    print(f"Error broadcasting to client: {e}")
    
    def start(self):
        """啟動服務器"""
        try:
            self.server_socket.bind((self.host, self.port))
            self.server_socket.listen(5)
            self.running = True
            print(f"Server listening on {self.host}:{self.port}")
            
            while self.running:
                # 準備select監聽列表
                readable, writable, exceptional = select.select(
                    [self.server_socket] + list(self.clients),
                    [],
                    [],
                    0.1  # 超時時間
                )
                
                for sock in readable:
                    if sock == self.server_socket:
                        # 處理新連接
                        client_socket, address = sock.accept()
                        self.clients.add(client_socket)
                        print(f"New connection from {address}")
                    else:
                        # 處理客戶端數據
                        if not self.handle_client(sock):
                            self.clients.remove(sock)
                            sock.close()
                
                # 處理異常的socket
                for sock in exceptional:
                    self.clients.remove(sock)
                    sock.close()
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.running = False
            for client in self.clients:
                client.close()
            self.server_socket.close()

# 使用示例
if __name__ == '__main__':
    server = SelectServer()
    server.start()
```

## 異步IO

### 1. 使用asyncio實現異步Socket

```python
import asyncio
import socket
from typing import Dict, Set
import time

class AsyncServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8888):
        self.host = host
        self.port = port
        self.clients: Set[asyncio.StreamReader] = set()
        self.running = False
    
    async def handle_client(self,
                          reader: asyncio.StreamReader,
                          writer: asyncio.StreamWriter):
        """處理客戶端連接"""
        client_id = id(reader)
        self.clients.add(reader)
        print(f"New client connected: {client_id}")
        
        try:
            while self.running:
                try:
                    # 異步讀取數據
                    data = await reader.read(1024)
                    if not data:
                        break
                    
                    # 處理數據
                    message = data.decode('utf-8')
                    print(f"Received from {client_id}: {message}")
                    
                    # 廣播消息
                    await self.broadcast(message, exclude=reader)
                
                except Exception as e:
                    print(f"Error handling client {client_id}: {e}")
                    break
        
        finally:
            self.clients.remove(reader)
            writer.close()
            await writer.wait_closed()
            print(f"Client {client_id} disconnected")
    
    async def broadcast(self, message: str,
                       exclude: asyncio.StreamReader = None):
        """廣播消息給所有客戶端"""
        for client in self.clients:
            if client != exclude:
                try:
                    client.write(message.encode('utf-8'))
                    await client.drain()
                except Exception as e:
                    print(f"Error broadcasting to client: {e}")
    
    async def start(self):
        """啟動服務器"""
        try:
            # 創建異步服務器
            server = await asyncio.start_server(
                self.handle_client,
                self.host,
                self.port
            )
            self.running = True
            print(f"Server listening on {self.host}:{self.port}")
            
            # 保持服務器運行
            async with server:
                await server.serve_forever()
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.running = False
            for client in self.clients:
                client.close()

# 使用示例
if __name__ == '__main__':
    server = AsyncServer()
    asyncio.run(server.start())
```

### 2. 異步HTTP客戶端

```python
import asyncio
import aiohttp
from typing import Dict, Optional, List
import time

class AsyncHTTPClient:
    def __init__(self, timeout: float = 30.0):
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def __aenter__(self):
        """創建會話"""
        self.session = aiohttp.ClientSession(timeout=self.timeout)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """關閉會話"""
        if self.session:
            await self.session.close()
    
    async def get(self, url: str) -> Optional[Dict]:
        """發送GET請求"""
        if not self.session:
            raise RuntimeError("Client not initialized")
        
        try:
            async with self.session.get(url) as response:
                return {
                    'status': response.status,
                    'headers': dict(response.headers),
                    'text': await response.text()
                }
        except Exception as e:
            print(f"Error making GET request: {e}")
            return None
    
    async def post(self, url: str,
                  data: Optional[Dict] = None,
                  json: Optional[Dict] = None) -> Optional[Dict]:
        """發送POST請求"""
        if not self.session:
            raise RuntimeError("Client not initialized")
        
        try:
            async with self.session.post(url, data=data, json=json) as response:
                return {
                    'status': response.status,
                    'headers': dict(response.headers),
                    'text': await response.text()
                }
        except Exception as e:
            print(f"Error making POST request: {e}")
            return None
    
    async def download_file(self, url: str,
                          filename: str) -> bool:
        """下載文件"""
        if not self.session:
            raise RuntimeError("Client not initialized")
        
        try:
            async with self.session.get(url) as response:
                if response.status == 200:
                    with open(filename, 'wb') as f:
                        while True:
                            chunk = await response.content.read(8192)
                            if not chunk:
                                break
                            f.write(chunk)
                    return True
                return False
        except Exception as e:
            print(f"Error downloading file: {e}")
            return False

# 使用示例
async def main():
    async with AsyncHTTPClient() as client:
        # 並發請求
        urls = [
            'http://example.com',
            'http://python.org',
            'http://github.com'
        ]
        
        tasks = [client.get(url) for url in urls]
        results = await asyncio.gather(*tasks)
        
        for url, result in zip(urls, results):
            if result:
                print(f"\nResponse from {url}:")
                print(f"Status: {result['status']}")
                print(f"Content length: {len(result['text'])}")
        
        # 下載文件
        success = await client.download_file(
            'https://example.com/file.txt',
            'downloaded_file.txt'
        )
        print(f"\nFile download {'successful' if success else 'failed'}")

if __name__ == '__main__':
    asyncio.run(main())
```

## 實戰示例

### 1. WebSocket服務器

```python
import asyncio
import websockets
from typing import Dict, Set
import json

class WebSocketServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8765):
        self.host = host
        self.port = port
        self.clients: Set[websockets.WebSocketServerProtocol] = set()
        self.running = False
    
    async def handle_client(self,
                          websocket: websockets.WebSocketServerProtocol,
                          path: str):
        """處理WebSocket連接"""
        client_id = id(websocket)
        self.clients.add(websocket)
        print(f"New client connected: {client_id}")
        
        try:
            while self.running:
                try:
                    # 接收消息
                    message = await websocket.recv()
                    data = json.loads(message)
                    print(f"Received from {client_id}: {data}")
                    
                    # 廣播消息
                    await self.broadcast({
                        'type': 'message',
                        'client_id': client_id,
                        'data': data
                    }, exclude=websocket)
                
                except websockets.exceptions.ConnectionClosed:
                    break
                except Exception as e:
                    print(f"Error handling client {client_id}: {e}")
                    break
        
        finally:
            self.clients.remove(websocket)
            print(f"Client {client_id} disconnected")
    
    async def broadcast(self, message: Dict,
                       exclude: websockets.WebSocketServerProtocol = None):
        """廣播消息給所有客戶端"""
        for client in self.clients:
            if client != exclude:
                try:
                    await client.send(json.dumps(message))
                except Exception as e:
                    print(f"Error broadcasting to client: {e}")
    
    async def start(self):
        """啟動服務器"""
        try:
            async with websockets.serve(
                self.handle_client,
                self.host,
                self.port
            ):
                self.running = True
                print(f"WebSocket server listening on {self.host}:{self.port}")
                
                # 保持服務器運行
                while self.running:
                    await asyncio.sleep(1)
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.running = False
            for client in self.clients:
                await client.close()

# 使用示例
if __name__ == '__main__':
    server = WebSocketServer()
    asyncio.run(server.start())
```

### 2. 異步代理服務器

```python
import asyncio
import aiohttp
from aiohttp import web
from typing import Dict, Optional
import logging

class ProxyServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8080):
        self.host = host
        self.port = port
        self.app = web.Application()
        self.app.router.add_route('*', '/{path:.*}', self.handle_request)
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def handle_request(self, request: web.Request) -> web.Response:
        """處理代理請求"""
        try:
            # 創建目標URL
            target_url = str(request.url).replace(
                f"{self.host}:{self.port}",
                request.match_info['path']
            )
            
            # 轉發請求
            async with self.session.request(
                request.method,
                target_url,
                headers=request.headers,
                data=await request.read() if request.body_exists else None,
                allow_redirects=False
            ) as response:
                # 創建響應
                return web.Response(
                    body=await response.read(),
                    status=response.status,
                    headers=response.headers
                )
        
        except Exception as e:
            logging.error(f"Proxy error: {e}")
            return web.Response(
                text=str(e),
                status=500
            )
    
    async def start(self):
        """啟動代理服務器"""
        try:
            # 創建會話
            self.session = aiohttp.ClientSession()
            
            # 啟動服務器
            runner = web.AppRunner(self.app)
            await runner.setup()
            site = web.TCPSite(runner, self.host, self.port)
            await site.start()
            
            print(f"Proxy server running on {self.host}:{self.port}")
            
            # 保持服務器運行
            while True:
                await asyncio.sleep(1)
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            if self.session:
                await self.session.close()

# 使用示例
if __name__ == '__main__':
    # 配置日誌
    logging.basicConfig(level=logging.INFO)
    
    # 啟動代理服務器
    proxy = ProxyServer()
    asyncio.run(proxy.start())
```

## 練習題

1. **異步聊天室**
   實現一個基於WebSocket的異步聊天室：
   - 使用asyncio和websockets
   - 支持房間功能
   - 實現用戶認證
   - 添加消息歷史

2. **高性能代理**
   開發一個高性能的異步代理服務器：
   - 支持HTTP/HTTPS
   - 實現緩存機制
   - 添加負載均衡
   - 監控性能指標

3. **實時數據流**
   創建一個實時數據流處理系統：
   - 使用非阻塞IO
   - 實現數據過濾
   - 支持多客戶端
   - 優化傳輸效率

## 小提醒 💡

1. 異步編程
   - 理解事件循環
   - 避免阻塞操作
   - 正確處理異常
   - 管理資源生命週期

2. 性能優化
   - 使用連接池
   - 實現緩存機制
   - 優化內存使用
   - 監控系統資源

3. 安全考慮
   - 驗證輸入數據
   - 實現訪問控制
   - 加密敏感信息
   - 防止DOS攻擊

4. 調試技巧
   - 使用異步調試器
   - 記錄詳細日誌
   - 模擬網絡條件
   - 壓力測試

[上一章：Socket編程基礎](066_Socket編程基礎.md) | [下一章：網絡協議基礎](068_網絡協議基礎.md) 