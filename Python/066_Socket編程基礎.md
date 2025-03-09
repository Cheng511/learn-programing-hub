[上一章：元編程進階](065_元編程進階.md) | [下一章：Socket編程進階](067_Socket編程進階.md)

# Python Socket編程基礎 🌐

## Socket概述

Socket是網絡通信的基礎，它提供了一種在不同計算機之間進行通信的方式。Python的`socket`模塊提供了完整的Socket編程支持。

### 1. Socket基本概念

```python
import socket

# Socket的基本屬性
print(f"Socket families: {socket.AF_INET}, {socket.AF_INET6}")
print(f"Socket types: {socket.SOCK_STREAM}, {socket.SOCK_DGRAM}")
print(f"Socket protocols: {socket.IPPROTO_TCP}, {socket.IPPROTO_UDP}")
```

### 2. 創建Socket

```python
# TCP Socket
tcp_socket = socket.socket(
    family=socket.AF_INET,     # IPv4
    type=socket.SOCK_STREAM,   # TCP
    proto=0                    # 默認協議
)

# UDP Socket
udp_socket = socket.socket(
    family=socket.AF_INET,     # IPv4
    type=socket.SOCK_DGRAM,    # UDP
    proto=0                    # 默認協議
)
```

## TCP編程

### 1. TCP服務器

```python
import socket
import threading
from typing import Tuple, Optional

class TCPServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 8888):
        self.host = host
        self.port = port
        self.server_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
        # 允許地址重用
        self.server_socket.setsockopt(
            socket.SOL_SOCKET,
            socket.SO_REUSEADDR,
            1
        )
    
    def handle_client(self,
                     client_socket: socket.socket,
                     address: Tuple[str, int]):
        """處理客戶端連接"""
        try:
            while True:
                # 接收數據
                data = client_socket.recv(1024)
                if not data:
                    break
                
                # 處理數據
                message = data.decode('utf-8')
                print(f"Received from {address}: {message}")
                
                # 發送響應
                response = f"Server received: {message}"
                client_socket.send(response.encode('utf-8'))
        
        except Exception as e:
            print(f"Error handling client {address}: {e}")
        
        finally:
            client_socket.close()
            print(f"Connection from {address} closed")
    
    def start(self):
        """啟動服務器"""
        try:
            # 綁定地址
            self.server_socket.bind((self.host, self.port))
            # 開始監聽
            self.server_socket.listen(5)
            print(f"Server listening on {self.host}:{self.port}")
            
            while True:
                # 接受客戶端連接
                client_socket, address = self.server_socket.accept()
                print(f"Accepted connection from {address}")
                
                # 創建新線程處理客戶端
                client_thread = threading.Thread(
                    target=self.handle_client,
                    args=(client_socket, address)
                )
                client_thread.start()
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.server_socket.close()

# 使用示例
if __name__ == '__main__':
    server = TCPServer()
    server.start()
```

### 2. TCP客戶端

```python
import socket
from typing import Optional

class TCPClient:
    def __init__(self, host: str = 'localhost',
                 port: int = 8888):
        self.host = host
        self.port = port
        self.client_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
    
    def connect(self) -> bool:
        """連接到服務器"""
        try:
            self.client_socket.connect((self.host, self.port))
            print(f"Connected to {self.host}:{self.port}")
            return True
        except Exception as e:
            print(f"Connection failed: {e}")
            return False
    
    def send_message(self, message: str) -> Optional[str]:
        """發送消息並接收響應"""
        try:
            # 發送數據
            self.client_socket.send(message.encode('utf-8'))
            
            # 接收響應
            response = self.client_socket.recv(1024)
            return response.decode('utf-8')
        
        except Exception as e:
            print(f"Error sending message: {e}")
            return None
    
    def close(self):
        """關閉連接"""
        self.client_socket.close()
        print("Connection closed")

# 使用示例
if __name__ == '__main__':
    client = TCPClient()
    if client.connect():
        try:
            while True:
                message = input("Enter message (or 'quit' to exit): ")
                if message.lower() == 'quit':
                    break
                
                response = client.send_message(message)
                if response:
                    print(f"Server response: {response}")
        
        finally:
            client.close()
```

## UDP編程

### 1. UDP服務器

```python
import socket
from typing import Tuple

class UDPServer:
    def __init__(self, host: str = 'localhost',
                 port: int = 9999):
        self.host = host
        self.port = port
        self.server_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_DGRAM
        )
    
    def start(self):
        """啟動服務器"""
        try:
            # 綁定地址
            self.server_socket.bind((self.host, self.port))
            print(f"UDP Server listening on {self.host}:{self.port}")
            
            while True:
                # 接收數據
                data, address = self.server_socket.recvfrom(1024)
                message = data.decode('utf-8')
                print(f"Received from {address}: {message}")
                
                # 發送響應
                response = f"Server received: {message}"
                self.server_socket.sendto(
                    response.encode('utf-8'),
                    address
                )
        
        except KeyboardInterrupt:
            print("\nServer shutting down...")
        
        finally:
            self.server_socket.close()

# 使用示例
if __name__ == '__main__':
    server = UDPServer()
    server.start()
```

### 2. UDP客戶端

```python
import socket
from typing import Optional, Tuple

class UDPClient:
    def __init__(self, host: str = 'localhost',
                 port: int = 9999):
        self.host = host
        self.port = port
        self.client_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_DGRAM
        )
    
    def send_message(self, message: str) -> Optional[str]:
        """發送消息並接收響應"""
        try:
            # 發送數據
            self.client_socket.sendto(
                message.encode('utf-8'),
                (self.host, self.port)
            )
            
            # 接收響應
            data, _ = self.client_socket.recvfrom(1024)
            return data.decode('utf-8')
        
        except Exception as e:
            print(f"Error sending message: {e}")
            return None
    
    def close(self):
        """關閉socket"""
        self.client_socket.close()
        print("Socket closed")

# 使用示例
if __name__ == '__main__':
    client = UDPClient()
    try:
        while True:
            message = input("Enter message (or 'quit' to exit): ")
            if message.lower() == 'quit':
                break
            
            response = client.send_message(message)
            if response:
                print(f"Server response: {response}")
    
    finally:
        client.close()
```

## 實用工具

### 1. 端口掃描器

```python
import socket
import threading
from typing import List, Set
from queue import Queue
import time

class PortScanner:
    def __init__(self, target: str, start_port: int = 1,
                 end_port: int = 1024, timeout: float = 1.0):
        self.target = target
        self.start_port = start_port
        self.end_port = end_port
        self.timeout = timeout
        self.open_ports: Set[int] = set()
        self.port_queue = Queue()
        
        # 初始化端口隊列
        for port in range(start_port, end_port + 1):
            self.port_queue.put(port)
    
    def scan_port(self):
        """掃描單個端口"""
        while True:
            try:
                port = self.port_queue.get_nowait()
            except Queue.Empty:
                break
            
            try:
                # 創建socket並設置超時
                sock = socket.socket(
                    socket.AF_INET,
                    socket.SOCK_STREAM
                )
                sock.settimeout(self.timeout)
                
                # 嘗試連接
                result = sock.connect_ex((self.target, port))
                if result == 0:
                    self.open_ports.add(port)
                    print(f"Port {port} is open")
                
                sock.close()
            
            except Exception:
                pass
    
    def scan(self, num_threads: int = 100) -> List[int]:
        """開始掃描"""
        print(f"Scanning {self.target} "
              f"from port {self.start_port} "
              f"to {self.end_port}")
        
        start_time = time.time()
        
        # 創建並啟動線程
        threads = []
        for _ in range(min(num_threads,
                          self.end_port - self.start_port + 1)):
            thread = threading.Thread(target=self.scan_port)
            thread.start()
            threads.append(thread)
        
        # 等待所有線程完成
        for thread in threads:
            thread.join()
        
        end_time = time.time()
        duration = end_time - start_time
        
        print(f"\nScan completed in {duration:.2f} seconds")
        print(f"Found {len(self.open_ports)} open ports")
        
        return sorted(list(self.open_ports))

# 使用示例
if __name__ == '__main__':
    scanner = PortScanner('localhost', 1, 1024)
    open_ports = scanner.scan()
    print("\nOpen ports:", open_ports)
```

### 2. 簡單的HTTP客戶端

```python
import socket
from typing import Dict, Optional
from urllib.parse import urlparse

class SimpleHTTPClient:
    def __init__(self, timeout: float = 10.0):
        self.timeout = timeout
    
    def get(self, url: str) -> Optional[str]:
        """發送GET請求"""
        try:
            # 解析URL
            parsed = urlparse(url)
            host = parsed.hostname
            port = parsed.port or 80
            path = parsed.path or '/'
            
            # 創建socket
            sock = socket.socket(
                socket.AF_INET,
                socket.SOCK_STREAM
            )
            sock.settimeout(self.timeout)
            
            # 連接服務器
            sock.connect((host, port))
            
            # 構建HTTP請求
            request = (
                f"GET {path} HTTP/1.1\r\n"
                f"Host: {host}\r\n"
                f"Connection: close\r\n"
                f"\r\n"
            )
            
            # 發送請求
            sock.send(request.encode('utf-8'))
            
            # 接收響應
            response = b""
            while True:
                data = sock.recv(4096)
                if not data:
                    break
                response += data
            
            return response.decode('utf-8')
        
        except Exception as e:
            print(f"Error: {e}")
            return None
        
        finally:
            sock.close()
    
    def parse_response(self, response: str) -> Dict[str, str]:
        """解析HTTP響應"""
        try:
            # 分離頭部和主體
            headers_raw, body = response.split('\r\n\r\n', 1)
            
            # 解析狀態行和頭部
            headers_lines = headers_raw.split('\r\n')
            status_line = headers_lines[0]
            headers = {}
            
            for line in headers_lines[1:]:
                if ':' in line:
                    key, value = line.split(':', 1)
                    headers[key.strip()] = value.strip()
            
            return {
                'status_line': status_line,
                'headers': headers,
                'body': body
            }
        
        except Exception as e:
            print(f"Error parsing response: {e}")
            return {}

# 使用示例
if __name__ == '__main__':
    client = SimpleHTTPClient()
    response = client.get('http://example.com')
    if response:
        parsed = client.parse_response(response)
        print("Status:", parsed.get('status_line'))
        print("\nHeaders:")
        for key, value in parsed.get('headers', {}).items():
            print(f"{key}: {value}")
        print("\nBody preview:")
        print(parsed.get('body', '')[:200])
```

## 練習題

1. **聊天室應用**
   實現一個基於TCP的多人聊天室：
   - 支持多客戶端同時連接
   - 實現用戶名註冊
   - 廣播消息功能
   - 私聊功能

2. **文件傳輸**
   開發一個基於UDP的文件傳輸程序：
   - 支持大文件分片傳輸
   - 實現校驗和驗證
   - 處理丟包重傳
   - 顯示傳輸進度

3. **網絡監控工具**
   創建一個網絡監控工具：
   - 監控指定端口
   - 記錄連接信息
   - 統計流量數據
   - 生成報告

## 小提醒 💡

1. 網絡編程基礎
   - 理解TCP/UDP區別
   - 注意錯誤處理
   - 正確關閉連接
   - 處理超時情況

2. 性能優化
   - 使用非阻塞IO
   - 實現並發處理
   - 優化緩衝區大小
   - 避免資源泄露

3. 安全考慮
   - 驗證輸入數據
   - 實現超時機制
   - 限制連接數量
   - 處理惡意請求

4. 調試技巧
   - 使用Wireshark
   - 記錄詳細日誌
   - 模擬網絡延遲
   - 測試異常情況

[上一章：元編程進階](065_元編程進階.md) | [下一章：Socket編程進階](067_Socket編程進階.md) 