# 第3章 因特网客户端编程

**一句话**：FTP、SMTP、POP3、NNTP 这些高层协议都是「在第 2 章 socket 之上包出来的专用客户端」，Python 标准库给每个都备好了现成模块。

**为什么需要它**：第 2 章讲的是裸 socket，但现实中你很少从零手写协议字节。文件传输、读新闻组、发邮件、收邮件——这些需求每天发生，且各自有成熟协议（FTP/SCP/HTTP 传文件，NNTP 读新闻，SMTP 发信，POP3/IMAP 收信）。Python 把这些协议的客户端都实现成了标准库模块，你只需调 API，不用关心底层 TCP 怎么握手。

**核心结构**：本章介绍多个因特网协议并写对应客户端。核心模块：`ftplib`（FTP）、`nntplib`（NNTP 新闻组）、`smtplib`（发邮件 SMTP）、`poplib`（收邮件 POP3），以及通用的 `urllib`（HTTP/URL 抓取）。它们共同点是**API 高度一致**：连上、发命令/传参、取结果、关连接。

**关键设计**：书里点明，这些高层协议「工作方式相似，API 也非常相似」并非巧合，而是设计之初就刻意保持一致——保持接口统一有大好处。学会一个，就能举一反三写出任意协议的客户端。HTTP、FTP、scp 属于「文件传输」类；SMTP/POP3/IMAP 属于「电子邮件」类；NNTP 属于「网络新闻」类。

**拆解**：

### 文件传输 FTP/HTTP
`ftplib.FTP(host)` 登录后 `retrbinary()` 下文件、`storbinary()` 上传；HTTP 侧用 `urllib.request.urlopen()` 取远程资源。书里强调这些「传输」类协议共享「连接-请求-取流」的节奏。

### 电子邮件 SMTP/POP3
`smtplib.SMTP(host)` 连上后 `sendmail()` 发信；`poplib.POP3(host)` 登录后 `retr()` 取信、`dele()` 删信。SMTP 负责「推」、POP3 负责「拉」，二者协议语义相反但客户端骨架相同。

### NNTP 网络新闻
`nntplib.NNTP(host)` 连新闻组服务器，`group()` 选组、`article()` 取文章。书里把它和邮件并列，说明「按主题订阅、拉取文章」也是一种客户端模式。

### 统一客户端 API 设计
书里总结：各协议客户端都遵循「连接 → 认证/选目标 → 操作 → 关闭」四步。刻意一致的接口让开发者能快速迁移到陌生协议，这也是标准库设计的用心之处。

**今天怎么做**：`urllib` 现在多被第三方 `requests` 取代（`requests.get(url).text` 一行搞定、自带会话与 JSON 解析）；发信也常用 `yagmail` 之类封装。但 `ftplib`/`smtplib`/`poplib` 仍是标准库一等公民，理解它们才能看懂 `requests` 底层在做什么——属延伸，非语料原文。

<div class="chapend" markdown="1">

**容易误解的地方**：有人以为「因特网客户端」就是用 `socket` 手写协议字节，其实书里明确——「现在不再使用底层的套接字创建因特网客户端，但模型完全相同」。你调 `smtplib` 时，底层仍然是 socket，只是库替你把 SMTP 命令序列封装好了。另一个坑：`poplib` 取回的是原始邮件文本（含头部与编码），要自己解析 `From/Subject` 和正文编码，不会自动给你结构化对象。

**记住这一句**：高层协议客户端（ftp/smtp/pop3/nntp）都是「socket 之上的专用封装」，API 刻意保持一致——学会一个的「连接→操作→关闭」四步，其余协议照猫画虎即可。

</div>
