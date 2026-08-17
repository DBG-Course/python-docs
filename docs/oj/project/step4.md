<!-- ## Step 4：用户管理

--- -->

### 模块目标

实现用户的注册、登录登出、信息查询、权限管理、用户列表等功能，支持权限控制和安全校验。

---

### 前置知识要求

| 技术点         | 推荐学习内容           |
| -------------- | ---------------------- |
| 数据结构设计   | 用户表、权限字段       |
| Session 管理 | Cookie, Session |
| REST API 设计  | GET/POST/PUT 路由      |
| 权限控制       | 用户/管理员区分        |

---

### 任务分解

#### 任务 0：用户登录/登出/初始管理员
- 目标：实现用户登录、登出接口，系统启动时自动创建初始管理员账户（账号为 `admin` / 密码为 `admintestpassword`）。

**Session 机制原理**

Session 是 Web 应用中维持用户状态的重要机制。由于 HTTP 协议是无状态的，服务器无法直接识别连续请求来自同一用户，因此需要 Session 来解决这个问题。

**工作流程：**
1. 用户首次访问时，服务器创建一个唯一的 Session ID
2. 服务器将 Session ID 通过 Cookie 发送给客户端
3. 客户端后续请求会自动携带这个 Cookie
4. 服务器根据 Session ID 查找对应的用户信息

**Session 存储方式：**
- **内存存储**：速度快，但服务器重启会丢失，不适合生产环境
- **文件存储**：持久化，但并发性能较差
- **数据库存储**：可靠性高，支持分布式部署
- **Redis 存储**：高性能，支持过期策略，是主流选择

**安全考虑：**
- Session ID 需要足够随机，防止被猜测
- 使用 HTTPS 传输 Cookie，防止被窃取
- 设置合理的过期时间，平衡用户体验和安全性
- 登出时要清除服务器端的 Session 数据

Session 相比 JWT 的优势是可以立即失效（服务器端删除），劣势是需要服务器端存储。建议查阅相关框架的 Session 文档，理解具体实现细节。

??? question "Session ID 生成"
    Session ID 用来唯一标记某个用户的某次会话，并且需要被服务器端存储，
    所以你需要保证拿到大规模不重复的 ID。

    可以使用 `uuid` 库的 `uuid4` 函数。

**中间件机制**

Web 框架通常使用中间件（Middleware）来处理 Session 管理。中间件是在请求处理过程中的拦截器，可以在请求到达路由处理函数之前或之后执行特定逻辑。

Session 中间件的工作原理：
1. 请求到达时，中间件从 Cookie 中读取 Session ID
2. 根据 Session ID 从存储中加载用户数据
3. 将用户信息附加到请求对象上
4. 请求处理完成后，中间件将 Session 数据保存回存储
5. 如果需要，更新 Cookie 中的 Session ID

这样设计的好处是业务代码无需关心 Session 的底层实现，只需要通过框架提供的接口访问用户信息即可。

**FastAPI Session 中间件示例**

```python
from starlette.middleware.sessions import SessionMiddleware

app.add_middleware(SessionMiddleware, secret_key="your-secret-key")

@app.post("/login")
async def login(request: Request):
    request.session["user_id"] = 1
    return {"message": "登录成功"}
```

建议查阅 Starlette 官方文档中的 [SessionMiddleware](https://www.starlette.io/middleware/#sessionmiddleware) 了解详细用法。

#### 任务 1：用户注册
- 目标：提供用户注册 API。

**数据验证与唯一性约束**

用户注册需要验证输入数据的有效性。主要检查用户名是否已存在、密码是否符合要求。

**基本验证要点：**
- 检查用户名长度（3-40 字符）
- 检查密码长度（最少 6 位）
- 查询数据库确认用户名未被使用
- 密码需要加密后存储（使用bcrypt库）

**处理流程：**
1. 接收用户名、密码参数
2. 验证格式是否正确
3. 检查用户名是否已存在
4. 加密密码并存储到数据库
5. 返回成功信息和用户ID

#### 任务 2：用户信息查询
- 目标：提供 API 查询用户信息。

**权限控制基础**

用户信息查询需要控制权限，确保用户只能查看自己的信息，管理员可以查看所有用户信息。

**权限检查流程：**
1. 从 session 中获取当前登录用户信息
2. 检查要查询的用户ID是否是当前用户自己
3. 或者检查当前用户是否是管理员
4. 如果权限不足，返回403错误
5. 如果权限充足，返回用户信息（不包含密码）

**返回数据：**

```json
{
  "code": 200, 
  "msg": "success", 
  "data": 
  {
    "total": 3, // 查询到的用户总数
    "users": 
    [
      {"user_id": "1", "join_time": "1924-08-17", "submit_count": 100, "resolve_count": 9},
      {"user_id": "2", "join_time": "1911-04-05", "submit_count": 90, "resolve_count": 8},
      {"user_id": "3", "join_time": "2012-07-14", "submit_count": 80, "resolve_count": 7},
    ]
  }
}
```

??? info "时间获取"
    你可以使用如下命令获取 f'{year}-{month}-{day}' 格式的时间
    ```python
    from datetime import datetime
    now = datetime.now()
    date_str = now.strftime("%Y-%m-%d")
    ```

#### 任务 3：用户权限变更
- 目标：管理员可变更用户权限（如设为 admin/banned）。

如果用户被 ban，其再登录时会被阻止。

**管理员权限检查**

只有管理员可以修改用户权限，需要严格验证操作者身份。

**基本实现：**
1. 检查当前用户是否是管理员
2. 获取要修改的用户ID和新权限
3. 验证新权限值是否有效（如user、admin、banned）
4. 更新数据库中的用户权限
5. 记录操作日志（谁在什么时候修改了谁的权限）

!!! danger "权限提示"
    注意，在 step1 ~ step3 中，我们没有对题目上传 / 语言创建等进行权限控制。在添加用户权限后，我们需要更新之前的功能。为简化，我们规定为：题目上传 / 创建语言可以由任意用户执行，但是删除题目操作**仅管理员可执行**，暂不考虑删除语言。此外，你还需要修改之前的接口，在用户未登录时无法进行增删查改。

#### 任务 4：用户列表查询
- 目标：管理员可查询所有用户列表，支持分页、筛选。

**分页与筛选**

根据 API 文档，用户列表查询，支持分页参数。

**API 参数：**
- `page`（可选）：页码
- `page_size`（可选）：每页大小

**返回格式：**
```json
{
  "code": 200, 
  "msg": "success", 
  "data": 
  {
    "total": 3, // 查询到的用户总数
    "users": 
    [
      {"user_id": "1", "username": "xiaoming", "join_time": "1924-08-17", "submit_count": 100, "resolve_count": 9},
      {"user_id": "2", "username": "xiaohong", "join_time": "1911-04-05", "submit_count": 90, "resolve_count": 8},
      {"user_id": "3", "username": "xiaogang", "join_time": "2012-07-14", "submit_count": 80, "resolve_count": 7},
    ]
  }
}
```

---

### 评分细则

| 功能/接口                | 分值 | 评分说明                         |
|--------------------------|------|----------------------------------|
| 用户注册接口             | 2    | 路径、参数、响应、异常            |
| 用户信息查询接口         | 1    | 权限、响应、异常                  |
| 用户权限变更接口         | 1    | 权限、参数、响应、异常            |
| 用户列表查询接口         | 1    | 分页、筛选、权限                  |
| **小计**                 | **5**|                                  |