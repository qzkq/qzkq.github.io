---
title: Python操作数据库之pyodbc
date: 2026-01-02 08:05:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Python操作数据库之pyodbc)
# 一、 pyodbc 是什么？

**pyodbc** 是一个开源的 Python 模块，它提供了一个接口，允许 Python 程序通过 **ODBC (Open Database Connectivity)** 来访问各种数据库。

ODBC 是一个标准的数据库访问 API，它允许应用程序使用 SQL 作为标准语言来访问不同的数据库管理系统（DBMS）。这意味着，只要数据库提供了 ODBC 驱动程序，你就可以使用 pyodbc 来连接和操作它。

**核心价值：** 提供了一种**统一**的方式来在 Python 中与多种数据库（如 SQL Server, Oracle, MySQL, PostgreSQL, DB2, Access 等）进行交互。

---

# 二、 核心概念与工作原理

1.  **ODBC 驱动管理器**： 操作系统层面的一个组件（在 Windows 上是 `odbcad32.exe`，Linux/macOS 上是 `unixODBC` 或 `iODBC`），它管理所有已安装的 ODBC 驱动程序。
2.  **ODBC 驱动程序**： 由数据库厂商或第三方提供的特定于某种数据库的动态链接库（.dll 或 .so）。它负责将标准的 ODBC API 调用“翻译”成特定数据库能够理解的协议和命令。
3.  **pyodbc**： 作为 Python 和 ODBC 驱动管理器之间的桥梁。你的 Python 代码调用 pyodbc 的函数，pyodbc 再通过 ODBC 驱动管理器调用相应的 ODBC 驱动程序，最终与数据库进行通信。

**工作流程：**
`Python Code` -> `pyodbc module` -> `ODBC Driver Manager` -> `Specific ODBC Driver (e.g., SQL Server ODBC Driver)` -> `Database (e.g., SQL Server)`

---

# 三、 安装与配置

## 1. 安装 pyodbc

使用 pip 可以轻松安装：

```bash
pip install pyodbc
```

## 2. 安装 ODBC 驱动程序

这是最关键的一步。**你必须先为你想要连接的数据库安装对应的 ODBC 驱动程序**，然后 pyodbc 才能工作。

* **SQL Server**：

  * Windows: 通常已自带 `ODBC Driver 17 for SQL Server` 或更新版本。也可以从微软官网下载。

  * Linux/macOS: 参考微软官方文档安装，例如在 Ubuntu 上：

    ```bash
    curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
    curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
    sudo apt-get update
    sudo ACCEPT_EULA=Y apt-get install -y msodbcsql18
    ```

* **MySQL**： 下载并安装 **MySQL Connector/ODBC**。

* **PostgreSQL**： 下载并安装 **PSQLODBC** 驱动程序。

* **Oracle**： 下载并安装 **Oracle Instant Client**，其中包含 ODBC 驱动程序。

安装后，通常可以在系统的 ODBC 数据源管理器中看到已安装的驱动。

---

# 四、 基本使用流程与代码详解

使用 pyodbc 操作数据库通常遵循以下步骤：

## 1. 建立连接 (`connect`)

使用 `pyodbc.connect()` 函数，传入一个连接字符串（Connection String）。

**连接字符串格式：**
`DRIVER={Driver Name};SERVER=server_name;DATABASE=db_name;UID=user_name;PWD=password`

*   **DRIVER**: ODBC 驱动程序的名称。**这是最容易出错的地方**，名称必须完全匹配。你可以在 ODBC 数据源管理器中查看准确的驱动名称。
    *   例如，SQL Server 17/18 驱动： `DRIVER={ODBC Driver 17 for SQL Server};...`
    *   老版本 SQL Server: `DRIVER={SQL Server};...`
    * MySQL：`DRIVER={MySQL ODBC 8.0 Driver};...`
    * PostgreSQL：`DRIVER={PostgreSQL Unicode};...`
*   **SERVER**: 数据库服务器地址。可以是 IP、主机名、localhost，对于 SQL Server 还可以是实例名 `localhost\SQLEXPRESS`。
*   **DATABASE**: 要连接的数据库名。
*   **UID**: 用户名。
*   **PWD**: 密码。
*   其他常见参数： `PORT`（端口）， `Trusted_Connection=yes;`（Windows 身份验证）等。

**示例：连接 SQL Server**

```python
import pyodbc

# 方式1：Windows 身份验证
conn_str = (
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=localhost\SQLEXPRESS;'
    r'DATABASE=testdb;'
    r'Trusted_Connection=yes;'
)

# 方式2：SQL Server 身份验证
conn_str = (
    r'DRIVER={ODBC Driver 17 for SQL Server};'
    r'SERVER=localhost\SQLEXPRESS;'
    r'DATABASE=testdb;'
    r'UID=myusername;'
    r'PWD=mypassword;'
)

conn = pyodbc.connect(conn_str)
```

## 2. 创建游标 (`cursor`)

连接成功后，需要创建一个游标（Cursor）对象。游标用于管理获取数据的上下文，执行 SQL 语句并获取结果。

```python
cursor = conn.cursor()
```

## 3. 执行 SQL 语句 (`execute`)

使用游标的 `execute()` 方法来执行 SQL 语句。

*   **执行查询（SELECT）**： 会返回结果。
*   **执行数据修改（INSERT, UPDATE, DELETE）** 或 DDL（CREATE TABLE）： 不返回数据，但会返回受影响的行数。

**示例：查询数据**

```python
# 执行一个查询
cursor.execute("SELECT id, name, email FROM users WHERE id = ?", 1)

# 获取单条记录
row = cursor.fetchone()
if row:
    print(f"ID: {row.id}, Name: {row.name}, Email: {row.email}")
    # 也可以通过索引访问：row[0], row[1]

# 获取所有记录
rows = cursor.fetchall()
for row in rows:
    print(row)
```

**示例：插入数据**

```python
# 插入单条数据，使用参数化查询（非常重要！可以防止SQL注入）
sql = "INSERT INTO users (name, email) VALUES (?, ?)"
values = ('Alice', 'alice@example.com')
cursor.execute(sql, values)

# 必须提交事务才能使修改生效
conn.commit()
```

**参数化查询**是强烈推荐的做法，使用 `?` 作为占位符，然后将参数作为元组传递给 `execute()` 方法。

**示例：插入多条数据 (`executemany`)**

```python
sql = "INSERT INTO users (name, email) VALUES (?, ?)"
values = [
    ('Bob', 'bob@example.com'),
    ('Charlie', 'charlie@example.com')
]
cursor.executemany(sql, values)
conn.commit()
```

## 4. 处理结果

* `fetchone()`: 获取下一行记录。

* `fetchall()`: 获取所有剩余记录。

* `fetchval()`: 直接获取第一行第一列的值，适用于 `COUNT(*)` 等查询。

* 迭代游标本身也是一个生成器，可以逐行处理：

  ```python
  cursor.execute("SELECT name FROM products")
  for row in cursor:
      print(row.name)
  ```

## 5. 关闭连接 (`close`)

操作完成后，务必关闭游标和连接以释放资源。

```python
cursor.close()
conn.close()
```

**最佳实践：使用 `with` 语句（上下文管理器）**
从 pyodbc 4.0 开始，连接和游标都支持上下文管理器，可以自动关闭资源。

```python
with pyodbc.connect(conn_str) as conn:
    with conn.cursor() as cursor:
        cursor.execute("SELECT ...")
        for row in cursor:
            ...
# 离开 with 块后，cursor 和 conn 会自动关闭
```

---

# 五、 高级特性与最佳实践

1. **事务管理**

   *   默认情况下，pyodbc 处于**自动提交模式（autocommit=False）**，这意味着你需要手动调用 `conn.commit()` 来提交事务，或者 `conn.rollback()` 来回滚。
   *   可以在创建连接时启用自动提交：`conn = pyodbc.connect(conn_str, autocommit=True)`

2. **获取插入后的自增ID**

   * 对于有自增主键的表，插入后可能需要获取生成的ID。

   * **SQL Server**: `SELECT SCOPE_IDENTITY()`

   * **MySQL**: `SELECT LAST_INSERT_ID()`

   * **PostgreSQL**: `RETURNING id` 子句

   * 示例：

     ```python
     cursor.execute("INSERT INTO table (name) OUTPUT INSERTED.id VALUES (?)", 'New Name')
     new_id = cursor.fetchval()
     conn.commit()
     print(f"New record ID is: {new_id}")
     ```

3. **处理编码问题**

   *   某些数据库（如 SQL Server）和驱动程序可能会返回 `byte` 字符串而不是 Unicode 字符串。
   *   可以在连接字符串中指定编码：`...;CHARSET=utf8;...`（取决于驱动支持）
   *   或者在连接后设置：`conn.setdecoding(pyodbc.SQL_CHAR, encoding='utf-8')` 和 `conn.setencoding(encoding='utf-8')`。

4. **连接池**

   *   pyodbc 默认启用连接池，这对于 Web 应用等需要频繁创建和销毁连接的场景非常有用。通常不需要手动管理。

5. **错误处理**

   *   使用 `try...except` 块来捕获 `pyodbc.Error` 或其子类（如 `pyodbc.ProgrammingError`, `pyodbc.OperationalError`）。

   ```python
   try:
       cursor.execute("SELECT ...")
   except pyodbc.Error as e:
       print(f"Database error occurred: {e}")
   ```

---

# 六、 常见问题与排查（Troubleshooting）

1. **`Data source name not found and no default driver specified` (IM002)**

   *   **原因**： 连接字符串中的 `DRIVER={...}` 名称不正确或驱动程序未安装。
   *   **解决**： 检查驱动名称拼写，确保驱动程序已正确安装。

2. **`Login failed for user`**

   *   **原因**： 用户名或密码错误，或者服务器地址不对。
   *   **解决**： 仔细检查连接字符串中的 `SERVER`, `UID`, `PWD`。

3. **`Communication link failure`**

   *   **原因**： 网络问题，无法连接到数据库服务器。
   *   **解决**： 检查服务器防火墙是否开放了数据库端口（如 SQL Server 的 1433 端口），服务器是否正在运行。

4. **性能问题**

   * 插入大量数据时，使用 `executemany()` 通常比循环调用 `execute()` 快得多。

   * 考虑使用 `fast_executemany=True` 选项（仅部分驱动支持，如 SQL Server），可以极大提升批量插入性能。

     ```python
     conn = pyodbc.connect(conn_str, autocommit=True)
     cursor = conn.cursor()
     cursor.fast_executemany = True
     cursor.executemany(...)
     ```

![在这里插入图片描述](/img/aa558efb4b9a4d72ad31e49632bdb5eb.png)

