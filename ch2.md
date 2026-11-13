# 第2章 网络编程

**一句话**：socket 是网络通信的「端点」，Python 用 `socket` 模块把客户端/服务器架构落成代码，让两台机器能互发字节流。

**为什么需要它**：当今大部分因特网协议（文件传输、邮件、新闻组）底层都是客户端/服务器模型：服务器无限运行、等请求、响应、再等；客户端一次性请求、收结果、结束。要自己实现一个能收发数据的程序，必须有一套通用的「通信端点」抽象，这就是套接字（socket）。IPv4 地址空间几近耗尽、向 IPv6 演进，更说明理解底层端点的重要性。

**核心结构**：套接字是「通信端点」。创建时要指定**地址族**（如 `AF_INET` 表示 IPv4）和**类型**（`SOCK_STREAM` 面向连接的 TCP、`SOCK_DGRAM` 无连接的 UDP）。典型 TCP 服务器流程：建 socket → `bind` 到地址 → `listen` → `accept` 阻塞等客户端；客户端：`connect` 后直接 `send`/`recv`。

**关键设计**：书里把客户端/服务器拆解成清晰角色——服务器唯一目的是等请求、响应、再等；客户端因特定请求联系服务器、发数据、等回应、结束事务。模型既可套硬件也可套软件。Python 还提供了**`SocketServer`**（Py3 改名 `socketserver`）模块，把「建连接、收请求、派发处理、回包」封装成可继承的类（`TCPServer`/`StreamRequestHandler`），并支持多线程/多进程服务器；另外简介了**Twisted** 这一异步网络框架。

**拆解**：

### 客户端/服务器架构
先分清「谁等、谁连」。服务器常驻，客户端短命。书里强调理解这个前提，才能看懂后面所有协议的 API 设计——它们都遵循同一套「请求-响应」节奏。

### 套接字创建与绑定
`socket.socket(family, type)` 建端点；服务器 `bind((host, port))` 占领端口，`listen()` 进入监听，`accept()` 拿出新连接 socket；客户端 `connect()` 后 `send/recv`。UDP 则无连接，直接 `sendto/recvfrom`。

### TCP 与 UDP
TCP（`SOCK_STREAM`）保证顺序与可靠，适合传文件、网页；UDP（`SOCK_DGRAM`）无连接、快但不保证到达，适合广播、实时音视频。书里提醒选错类型会让协议设计全盘皆输。

### SocketServer 与 Twisted
`socketserver` 用「基类 + 处理器类」省去样板；Twisted 走事件驱动、适合高并发长连接。两者都是「在裸 socket 之上加一层组织」，让业务代码只关心「收到什么、回什么」。

**今天怎么做**：裸 `socket` 现在多用于教学与协议调试；真实服务几乎都架在 `asyncio`（协程事件循环、原生 `async/await` 网络）、`Twisted` 或 `trio` 之上。书里提到的 `asyncore`/`select` 模块已基本被 `asyncio` 取代——属延伸，非语料原文。

<div class="chapend" markdown="1">

**容易误解的地方**：新手常以为 `accept()` 返回的是「新客户端对象」，其实它返回 `(conn, addr)` 二元组，其中 `conn` 才是和这个客户端通信的新 socket，原来的监听 socket 要继续 `accept` 下一个。另一个坑：TCP 是**字节流**不是消息流，`recv(1024)` 一次未必收全一个逻辑包，需要自己按长度或分隔符拼包。

**记住这一句**：socket 是「端点」不是「连接」——服务器 `bind+listen+accept` 拿到的是**新的连接 socket**，字节流要自己按协议拼包，TCP 不替你划分消息边界。

</div>
