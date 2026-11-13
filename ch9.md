# 第9章 Web 编程：CGI 与 WSGI

**一句话**：CGI 让 Web 服务器把表单数据交给外部 Python 程序、再把程序输出的 HTML 返回浏览器；WSGI 则把「框架」和「服务器」用统一接口粘合起来。

**为什么需要它**：静态 HTML 只能看、不能交互。一旦网页要处理用户输入（下单、搜索、登录），就需要「服务器收到请求 → 调外部程序生成动态 HTML → 返回」。CGI 是这套机制的早期标准；但它每次请求都启进程、难扩展。WSGI 横空出世，Philip Eby 那句被书里引的话点明：**WSGI 不是给应用作者用的 API，而是框架与服务器之间的粘合层**。

**核心结构**：CGI（Common Gateway Interface）定义「Web 服务器如何调用外部程序、通过环境变量/标准输入输出传数据」。WSGI（Web Server Gateway Interface，PEP 333 及 Py3 的 3333）定义「一个可调用对象（应用）如何被服务器调用」：服务器传 `(environ, start_response)`，应用返回可迭代的响应体。

**关键设计**：书里先讲 CGI 全貌——GET/POST 怎么传参、服务器怎么 `fork/exec` 外部脚本、脚本怎么从 `stdin`/环境变量读输入、把 HTML 写到 `stdout`。再过渡到 WSGI：它把「每次请求起进程」变成「一次加载、多次调用应用对象」，性能与可移植性都更好，且服务器和框架解耦——同一个 Django 应用能跑在任意 WSGI 服务器上。

**拆解**：

### CGI 原理
Web 服务器收到请求 → 调外部 CGI 程序 → 程序读环境变量（`QUERY_STRING` 等）和标准输入 → 输出 HTTP 头 + HTML 到标准输出 → 服务器转发给客户端。每次请求都是一次新进程，开销大。

### 表单 GET/POST
GET 把参数拼进 URL（`?k=v&...`），适合取数；POST 把参数放请求体，适合提交。CGI 程序要自己解析这两种来源，书里强调这是动态网页的入口。

### WSGI 规范 PEP 333
WSGI 应用是「可调用的 Python 对象」：接收 `environ`（请求信息字典）和 `start_response`（回调，用来发状态码和响应头），返回响应体的可迭代对象（如 `[b"hello"]`）。这份契约让服务器无需知道框架细节。

### wsgiref 与真实开发
`wsgiref` 是 WSGI 的参考实现，书里用它演示最小服务器（`run_wsgi_app` 类函数）。真实开发里它多作测试；生产用 gunicorn 等，但应用代码写法完全一致。

**今天怎么做**：CGI 已基本退出历史舞台；WSGI 仍是 Flask/Django 的部署底座，但新项目更爱 `ASGI`（异步、支持 WebSocket，如 FastAPI/uvicorn）。理解 WSGI 的「应用对象 + environ + start_response」契约，是看懂所有 Python Web 框架的钥匙——属延伸，非语料原文。

<div class="chapend" markdown="1">

**容易误解的地方**：常有人以为「WSGI 是写 Web 应用的 API」，其实它是**服务器与框架之间的接口**——你写 Flask 路由时根本不直接碰 `environ`/`start_response`，框架替你包好了。另一个坑：WSGI 应用必须返回**字节**（`b"..."`）而非 `str`，返回字符串会直接报错。

**记住这一句**：CGI 是「每次请求起进程跑脚本」、笨重难扩；WSGI 是「框架与服务器之间的粘合契约」——应用是个接收 `(environ, start_response)`、返回字节体的可调用对象，与具体服务器解耦。

</div>
