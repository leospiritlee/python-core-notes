# 第6章 数据库编程

**一句话**：DB-API 是 Python 访问关系数据库的统一接口，`connect()` 拿连接、`cursor()` 拿游标、`execute()` 跑 SQL，参数化查询既防注入又省解析。

**为什么需要它**：小应用用文件或 `shelve` 就够了，但大项目需要成熟的数据库系统。书里用 XKCD 漫画提醒 SQL 注入的危害——把用户输入直接拼进 SQL 字符串等于开门揖盗。Python 需要一套**统一的数据库 API**，让同样的代码逻辑能换底层数据库（SQLite、MySQL、PostgreSQL…）而不重写。

**核心结构**：持久化有三类机制——**文件**（普通文件、DBM）、**数据库系统**（关系/非关系）、**混合类型**（API、ORM、配置文件等）。DB-API 定义了所有数据库驱动必须遵守的接口：`connect()` 返回连接对象，连接 `.cursor()` 得到游标，游标 `.execute(sql)` 执行、`.fetchall()/.fetchone()` 取结果。

**关键设计**：书里重点讲**参数化查询**——`execute("SELECT * FROM t WHERE id=%s", (uid,))` 让驱动负责转义，杜绝注入，也让同一条 SQL 可被数据库缓存执行计划。还讲了**对象关系映射（ORM）**：把表映射成 Python 类、行映射成对象，业务代码操作对象而非拼 SQL。非关系数据库（键值、文档）也一并简介。

**拆解**：

### 持久化存储三机制
文件最轻、DBM/shelve 是「类字典的文件存储」、数据库适合大数据量。书里把 DBM、shelve（pickle+DBM）、ORM 都归为「混合类型」，是文件到数据库之间的过渡带。

### DB-API 接口规范
`connect()` → 连接；`conn.cursor()` → 游标；`cur.execute()` → 执行；`cur.fetch*()` → 取数；`conn.commit()`/`rollback()` → 事务。所有合规驱动（MySQLdb、psycopg2、sqlite3）都长这样，换库只换 `connect` 参数。

### 游标与参数化查询
游标是「在连接上开的执行会话」。占位符写法因库而异（`%s`、`?`、`:name`），但**永远用参数化**而非字符串拼接，这是 DB-API 给的安全底线。

### ORM 与非关系库
ORM（如 Django ORM、SQLAlchemy）用对象操作替你生成 SQL，开发快但有「N+1 查询」等性能陷阱；NoSQL（Redis、MongoDB）走自己的客户端 API，不走 DB-API。

**今天怎么做**：现代多用 SQLAlchemy（DB-API 之上的 ORM/核）或各框架自带 ORM；异步场景用 `aiomysql`/`asyncpg`。但理解 DB-API 这层「连接-游标-执行」骨架，是看懂所有 Python 数据库库的钥匙——属延伸，非语料原文。

<div class="chapend" markdown="1">

**容易误解的地方**：最大误区是「复用同一个游标跨多个 `commit` 没问题」——游标是轻量会话，长时间持有或大结果集 `fetchall()` 会占连接；正确做法是短命游标、用完即关、连接交还连接池。另一个坑：忘了 `conn.commit()`，INSERT/UPDATE 在事务里「成功执行」却没落库，看起来像「写进去了实际没有」。

**记住这一句**：DB-API 的骨架永远是「连接 → 游标 → 执行 → 取数 → 提交」，SQL 永远参数化、游标用完即关、写操作记得 `commit()`，否则要么被注入要么静默丢数据。

</div>
