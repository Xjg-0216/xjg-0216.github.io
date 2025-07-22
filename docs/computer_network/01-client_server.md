
# 网络通信

## 客户端（Client）

客户端是指向服务器请求服务的计算机或程序。客户端通常是用户直接交互的设备或应用程序，它发送请求到服务器，并等待服务器返回响应。

- **特点**：
    - **主动发起请求**：客户端主动向服务器发送请求，例如请求网页、数据或其他资源。
    - **资源有限**：相对于服务器，客户端通常资源（如处理能力、存储空间）较有限。
- **例子**：
    - **Web浏览器**：如Google Chrome、Mozilla Firefox等，通过HTTP向Web服务器请求网页。
    - **Email客户端**：如Microsoft Outlook、Thunderbird，通过IMAP或SMTP协议与邮件服务器通信。
    - **移动应用**：如微信、Twitter等，通过API与服务器进行数据交互。

## 服务端（Server）

服务端是指响应客户端请求的计算机或程序。服务器通常是提供某种服务的设备或应用程序，它接收客户端的请求，处理请求并返回响应。

- **特点**：
    - **被动等待请求**：服务器被动地等待客户端的请求，并处理这些请求。
    - **处理能力强**：通常具有强大的处理能力和大量的存储空间，能够处理大量并发请求。
    - **提供服务**：服务器提供各种服务，如Web服务、文件服务、数据库服务等。
- **例子**：
    - **Web服务器**：如Apache、Nginx，提供网页和其他Web资源。
    - **数据库服务器**：如MySQL、PostgreSQL，处理数据库查询和事务。
    - **文件服务器**：如FTP服务器，提供文件上传和下载服务。

### 客户端-服务器模型（Client-Server Model）

客户端-服务器模型是一种网络架构模型，在这种模型中，客户端和服务器各自承担特定的角色：

- **客户端**：发起请求，通常由终端用户使用。
- **服务器**：响应请求，提供资源或服务。

### 交互过程

1. **客户端发起请求**：客户端向服务器发送请求，例如请求一个网页或查询某个数据库记录。
2. **服务器接收请求**：服务器接收并处理客户端的请求。
3. **服务器返回响应**：服务器将处理结果返回给客户端。
4. **客户端接收响应**：客户端接收并处理服务器返回的响应，可能会显示给用户或进行进一步处理。




| 特性          | UDP                | TCP               |
| ----------- | ------------------ | ----------------- |
| **协议类型**    | 无连接（Datagram）      | 面向连接（Stream）      |
| **可靠性**     | 不可靠，不保证到达、顺序、去重    | 可靠，保证顺序、无丢包、无重复   |
| **传输速度**    | 快，开销小              | 稍慢，有握手确认、重传机制     |
| **数据格式**    | 数据报（Datagram），一条一条 | 字节流（Stream），连续数据流 |
| **适用场景**    | 视频直播、语音通话、游戏       | 网页浏览、文件传输、数据库访问   |
| **是否需要连接**  | ❌ 无连接（直接发）         | ✅ 三次握手建立连接        |
| **是否分包/组包** | 应用层处理              | TCP 内部处理（数据粘包/拆包） |


## UDP通信（无连接，快速传输）
>UDP（User Datagram Protocol，用户数据报协议）是一种简单的、无连接的传输层协议。UDP通信中的服务器和客户端各自承担特定的角色，通过发送和接收数据报进行通信。

### 1. UDP 服务端

主要负责监听特定端口并处理客户端发送的数据报。由于UDP是无连接的，服务端不需要维护客户端连接状态。

**工作流程**

1. 创建一个UDP套接字。
2. 绑定到一个指定的IP地址和端口。
3. 不断监听和接收来自客户端的数据报。
4. 处理接收到的数据并可能发送响应数据报回客户端。

```python
import socket

def udp_server():
    # 创建UDP套接字 AF_INET: 使用 IPv4 地；SOCK_DGRAM: 数据报套接字（UDP）
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    
    # 定义服务器地址和端口
    server_address = ("0.0.0.0", 12345)
    
    # 绑定套接字到地址和端口
    server_socket.bind(server_address)
    print("Server is listening on {}:{}".format(*server_address))
    
    while True:
        # 等待接收数据，最大4096字节，阻塞式，直到有数据到来才继续执行
        # data是收到的数据）（bytes类型），client_address是客户端的IP和端口
        data, client_address = server_socket.recvfrom(4096)
        print(f"Received {data.decode()} from {client_address}")
        
        # 发送响应
        if data.decode().lower() == "exit":
            print("Server exiting...")
            break
        
        response = "Hello, client!".encode()
        # 使用sendto发送信息回客户端
        server_socket.sendto(response, client_address)
    
    # 关闭套接字
    server_socket.close()

if __name__ == "__main__":
    udp_server()

```

**c++版本**
```cpp
//udp_server.cpp
#include <iostream>
#include <cstring> // memset, strcmp
#include <unistd.h> // close
#include <arpa/inet.h> // sockaddr_in, inet_ntoa

int main(){

    // 1. 创建UDP套接字
    int server_socket = socket(AF_INET, SOCK_DGRAM, 0);
    if (server_socket < 0){
        perror("socket creation failed");
        return 1;
    }

    // 2. 设置服务端地址结构体
    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr)); // 清空结构体
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY; // 监听所有网卡
    server_addr.sin_port = htons(12345); // 端口号

    // 3. 绑定socket到地址和端口
    if(bind(server_socket, (sockaddr*)&server_addr, sizeof(server_addr)) < 0){
        perror("bind failed");
        close(server_socket);
        return 1;
    }

    // 4. 接收并回复客户端
    char buffert[4096];
    sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    while(true){
        memset(buffer, 0, sizeof(buffer));
        int bytes_received = recvfrom(server_socket, buffer, sizeof(buffer)-1, 0, (sockaddr*)&client_addr, &client_len);
        if (bytes_received < 0){
            perror("revfrom failed");
            break;
        }

        buffer[bytes_received] = '\0';
        std::string message(buffer);
    }
    // 5. 关闭socket
    close(server_socket);
    return 0;
}
```



### 2. UDP 客户端

UDP 客户端负责向服务器发送数据报，并处理服务器的响应。由于UDP是无连接的，客户端不需要建立和关闭连接，只需发送数据报即可。

#### UDP 客户端的工作流程

1. 创建一个UDP套接字。
2. 向服务器发送数据报。
3. 接收来自服务器的响应数据报（如果有）。
4. 处理接收到的数据。

```python
import socket

def udp_client():
    # 创建UDP套接字 AF_INET: 使用 IPv4 地；SOCK_DGRAM: 数据报套接字（UDP）
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    
    # 定义服务器地址和端口
    server_address = ("127.0.0.1", 12345)
    
    try:
        # 发送数据到服务器
        message = "Hello, server!".encode()
        client_socket.sendto(message, server_address)
        
        # 接收服务器的响应
        data, server = client_socket.recvfrom(4096)
        print(f"Received {data.decode()} from server")
        
        # 发送退出消息
        exit_message = "exit".encode()
        client_socket.sendto(exit_message, server_address)
    
    finally:
        # 关闭套接字
        client_socket.close()

if __name__ == "__main__":
    udp_client()
```
**cpp版本**

```cpp
//udp_client.cpp

#include <iostream>
#include <cstring>
#include <unistd.h> // for memset
#include <arpa/inet.h> // for sockaddr_in, inet_pton

int main(){

    // 1. 创建udp socket
    int client_socket = socket(AF_INET, SOCK_DGRAM, 0);
    if (client_socket < 0){
        perror("socket creation failed");
        return 1;
    }
    // 2. 设置服务端地址结构体
    sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(12345);

    // 将ip转换为网络地址格式
    if (inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr) <= 0){
        std::error << "Invalid server IP address" << std::endl;
        close(client_socket);
        return 1;
    }

    // 3. 发送数据给服务器
    std::string message = "hello server!";
    int sent = sendto(client_socket, message.c_str(), message.size(), 0, (socketaddr*)&server_addr, sizeof(server_addr));

    if(sent < 0)
    {
        perror("sendto failed");
        close(client_socket);
        return 1;
    }

    // 4. 接收服务器响应
    char buffer[4096];
    memset(buffer, 0, sizeof(buffer));
    socklen_t addr_len = sizeof(server_addr);

    int received = recvfrom(client_socket, buffer, sizeof(buffer)-1, 0, (sockaddr*)&server_addr, &addr_len);

    if (received < 0){
        perror("recvfrom failed");
        close(client_socket);
        return 1;
    }
    buffer[received] = '\0';
    std::cout << "Received from server: " << buffer << std::endl;

    // 5. 发送exit 消息
    std::string exit_msg = "exit";
    sendto(client_socket, exit_msg.c_str(), exit_msg.size(), 0, (sockaddr*)&server_addr, sizeof(server_addr));

    // 6. 关闭socket
    close(client_socket);
    return 0;
}



```

## TCP通信

### 1. TCP服务端（监听，接受连接，双向通信）

```python
# tcp_server.py
import socket

def tcp_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address = ("0.0.0.0", 12345)
    server_socket.bind(server_address)
    server_socket.listen(1) # 开始监听，最大连接数为1

    print(f"TCP Server is listening on {server_address}")

    conn, client_address = server_socket.accept()
    print(f"Connected by {client_address}")

    try:
        while True:
            fata = conn.recv(1024)
            if not data:
                print("Connection closed by client")
                break
            print(f"Received from client: {data.decode{}}")

            if data.decode().lower() == "exit":
                print("Server exiting...")
                break

            conn.sendall(b"hello, tcp client!")
    finally:
            conn.close()
            server_socket.close()

if __name__ == "__main__":
    tcp_server()
```
### 2. TCP 客户端（连接，发送，接收）

```python
# tcp_client.py

import socket

def tcp_client():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address = ("127.0.0.1", 12345)
    client_socket.connect(server_address)

    try:
        # 发送数据
        message = "hello, server".encode()
        client_socket.sendall(message)

        # 接收响应
        data = client_socket.recv(1024)
        print(f"received from server: {data.decode()}")

        # 发送退出消息
        exit_message = "exit".encode()
        client_socket.sendall(exit_message)

    finally:
        client_socket.close()

if __name__ == "__main__":
    tcp_client()


```


| 操作行为        | UDP                  | TCP                            |
| ----------- | -------------------- | ------------------------------ |
| 创建 Socket   | `SOCK_DGRAM`         | `SOCK_STREAM`                  |
| 是否需要 bind() | 服务器需要，客户端不必须         | 服务器需要，客户端用 `connect()`         |
| 发送方式        | `sendto(data, addr)` | `sendall(data)`                |
| 接收方式        | `recvfrom()`         | `recv()`                       |
| 连接建立方式      | 无连接                  | 客户端 `connect()`，服务端 `accept()` |



