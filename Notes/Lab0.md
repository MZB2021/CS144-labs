# Lab0

在做实验之前需要了解基本的 `socket` 编程。什么是基本的 `socket` 编程呢。我认为就是认识基本的函数：
  1. 客户端如何建立连接
  2. 客户端如何接收/发送信息
  3. 服务端如何建立连接
  4. 服务端如何接受/发送信息

下面是客户端最基本的代码：
```cpp
// TODO: 验证代码准确性
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 8080
#define SERVER_IP "127.0.0.1"
#define MAX_BUFFER_SIZE 1024

int main() {
    /* 创建Socket
     * AF_INET 指定Ipv4
     * SOCK_STREAM TCP协议
     * 成功返回 socket 文件描述符
    */
    
    int client_socket = socket(AF_INET, SOCK_STREAM, 0);

    // 设置服务器地址
    struct sockaddr_in server_address;
    server_address.sin_family = AF_INET;
    /* host to network short type
     * 由于network都是 大端法，需要进行转化
     * port short类型就可以
    */
    server_address.sin_port = htons(PORT);
    inet_pton(AF_INET, SERVER_IP, &server_address.sin_addr);

    // 连接到服务器
    connect(client_socket, (struct sockaddr*)&server_address, sizeof(server_address));

    // 发送和接收数据
    char data_to_send[] = "Hello, Server!";
    send(client_socket, data_to_send, strlen(data_to_send), 0);

    char buffer[MAX_BUFFER_SIZE];
    recv(client_socket, buffer, sizeof(buffer), 0);
    printf("Server response: %s\n", buffer);

    // 关闭连接
    close(client_socket);

    return 0;
}
```
客户端包括：指定socket IP协议和TCP/UDP协议，提供服务器IP地址以及服务器端口号，通过 `connect()` 函数建立连接，通过 `send()`，`recv()` 发送/接受信息

服务器端基本代码：
```cpp
// TODO: 验证代码
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 8080
#define MAX_BUFFER_SIZE 1024

int main() {
    int server_socket, new_socket;
    struct sockaddr_in server_address, client_address;
    socklen_t client_address_len = sizeof(client_address);

    // 创建Socket
    if ((server_socket = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 设置服务器地址
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = INADDR_ANY;
    server_address.sin_port = htons(PORT);

    // 绑定Socket到指定地址和端口
    if (bind(server_socket, (struct sockaddr*)&server_address, sizeof(server_address)) == -1) {
        perror("Bind failed");
        exit(EXIT_FAILURE);
    }

    // 监听连接请求
    if (listen(server_socket, 5) == -1) {
        perror("Listen failed");
        exit(EXIT_FAILURE);
    }

    printf("Serve 接受连接并处理数据");
    while (1) {
        // 接受连接
        if ((new_socket = accept(server_socket, (struct sockaddr*)&client_address, &client_address_len)) == -1) {
            perror("Accept failed");
            exit(EXIT_FAILURE);
        }

        printf("Connection accepted from %s:%d\n", inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));

        // 接收和发送数据
        char buffer[MAX_BUFFER_SIZE];
        ssize_t bytes_received;

        while ((bytes_received = recv(new_socket, buffer, sizeof(buffer), 0)) > 0) {
            buffer[bytes_received] = '\0'; // 确保字符串结尾
            printf("Received message from client: %s\n", buffer);

            // 将接收到的数据发送回客户端
            send(new_socket, buffer, strlen(buffer), 0);
        }

        if (bytes_received == 0) {
            printf("Client disconnected\n");
        } else if (bytes_received == -1) {
            perror("Recv failed");
        }

        // 关闭连接
        close(new_socket);
    }

    // 关闭服务器Socket
    close(server_socket);

    return 0;
}

```
