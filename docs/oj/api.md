# API 文档

---

## 目录
1. 题目管理相关接口（Step 1）
2. 评测相关接口（Step 2 & 3）
3. 用户管理相关接口（Step 4）
4. 评测日志相关接口（Step 5）
5. 数据持久化相关接口（Step 6）
6. 高级功能接口 (Advance)
7. 安全性说明

---

**系统初始化说明**：系统启动时会自动创建初始管理员账户，用户名为 `admin`，密码为 `admintestpassword`（请注意校验要求）。

---

## 状态码与异常

| HTTP 状态码 | 说明           | 示例场景           |
|-------------|----------------|--------------------|
| 200         | 正常           | 一切正常           |
| 400         | 参数错误       | 缺少/错误参数      |
| 401         | 未登录         | 窃取 API 参数        |
| 403         | 权限不足/禁用  | banned 用户/无权限 |
| 404         | 资源不存在     | 题目/评测不存在    |
| 429         | 频率超限       | 1min 内提交超过 3 次       |
| 500         | 服务器异常     | 未知错误           |

说明：

- API 异常处理顺序为 **401 > 403 > 400 > 429 > 409 > 404 > 500**
- 所有 API 接口的 JSON 响应都必须包含 `code` 字段，该字段的值应与 HTTP 状态码保持一致
- 服务器必须设置对应的 HTTP 状态码（不能全部返回 200）
- 错误响应格式应该类似：
```json
{"code": 404, "msg": "problem not found", "data": null}
```

---

## 1. 题目管理相关接口（Step 1）

### 查看题目列表
- 路径：`GET /api/problems/`
- 权限：所有已登录用户
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {"id": "sum_2", "title": "两数之和"},
    {"id": "max_num", "title": "最大数"}
  ]
}
```

### 添加题目
- 路径：`POST /api/problems/`
- 权限：所有已登录用户
- 参数（参考 Step1 文档）：
  - `id` (str, 必填): 题目唯一标识
  - `title` (str, 必填): 题目标题
  - `description` (str, 必填): 题目描述
  - `input_description` (str, 必填): 输入格式说明
  - `output_description` (str, 必填): 输出格式说明
  - `samples` (list, 必填): 样例输入输出，元素为 {input, output}
  - `constraints` (str, 必填): 数据范围和限制条件
  - `testcases` (list, 必填): 测试点，元素为 {input, output}
  - `hint` (str, 可选): 额外提示
  - `source` (str, 可选): 题目来源
  - `tags` (list, 可选): 题目标签
  - `time_limit` (float, 可选): 时间限制，默认单位为 "s"，默认值为 "3"
  - `memory_limit` (int, 可选): 内存限制，默认单位为 "MB"，默认值为 "128"
  - `author` (str, 可选): 题目作者
  - `difficulty` (str, 可选): 难度等级
- 响应：
```json
{"code": 200, "msg": "add success", "data": {"id": "sum_2"}}
```
- 异常：400 字段缺失/格式错误 / 401 未登录 (Step 4) / 409 id 已存在

### 删除题目
- 路径：`DELETE /api/problems/{problem_id}`
- 权限：仅管理员
- 参数：无（URL 路径参数：`problem_id`）
- 响应：
```json
{"code": 200, "msg": "delete success", "data": {"id": "sum_2"}}
```
- 异常：401 未登录 (Step 4) / 403 权限不足 / 404 题目不存在

### 查看题目信息
- 路径：`GET /api/problems/{problem_id}`
- 权限：所有已登录用户
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "id": "P1001",
    "title": "A+B Problem",
    "description": "输入两个整数 a, b，输出它们的和（|a|,|b| <= 10^9）。",
    "input_description": "输入两个整数 a 和 b。",
    "output_description": "输出 a+b 的结果。",
    "samples": [
      {
        "input": "1 2",
        "output": "3"
      }
    ],
    "constraints": "|a|,|b| <= 10^9",
    "testcases": [
      {
        "input": "1 2",
        "output": "3"
      }
    ],
    "hint": "有负数哦！",
    "source": "洛谷",
    "tags": ["基础题"],
    "time_limit": 1.0,
    "memory_limit": 128,
    "author": "Luogu",
    "difficulty": "入门"
  }
}
```
- 异常：401 未登录 (Step4) / 403 权限不足 / 404 题目不存在
-  默认字段需要返回本类型默认值，比如 `str` 类需返回 `""`，`list` 类需返回 `[]`

---

## 2. 评测相关接口（Step 2 & 3）

> 请注意，Step 2 & 3 的查询评测结果 / 评测列表接口仅返回最终总分，单个测试点状态需要在评测日志中查询

### 提交评测
- 路径：`POST /api/submissions/`
- 参数：
  - `problem_id` (str, 必填): 题目编号
  - `language` (str, 必填): 语言（如 "python", "cpp"）
  - `code` (str, 必填): 用户代码内容
- 权限：登录用户
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {"submission_id": "123", "status": "pending"}
}
```
- 异常：400 参数错误 / 401 未登录 (Step 4) / 403 权限不足 / 404 题目不存在 & 语言不存在 / 429 提交频率超限

### 查询评测结果
- 路径：`GET /api/submissions/{submission_id}`
- 权限：仅本人或管理员
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "score": 10, // 本题获得分数
    "counts": 30, // 本题总分数（测试点数目 * 10）
  }
}
```
- 异常：401 未登录 (Step 4) / 403 权限不足 / 404 评测不存在

### 查询评测列表
- 路径：`GET /api/submissions/`
- 参数：`user_id`、`problem_id`、`status`、`page`、`page_size`
> 这五个参数均可选，其中 `user_id`、`problem_id` 为一级条件，其余为二级条件。一级条件不可以全部为空。
> 如果 `page` 和 `page_size` 全为空，表明查询所有数据；`page` 为空但 `page_size` 不为空表明选择第一页数据；需要认为 `page` 非空但 `page_size` 为空的情况属于参数错误。
> 如果未提供 `user_id`，那么管理员可查看此问题所有同学的记录，普通用户尽可查看此题自己的提交记录。
- 权限：本人/管理员
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": 
  {
    "total": 100, // 查询到的评测总数
    "submissions": 
    [
      // 如果 status 是 error / pending，则只需要返回 submission_id 和 status
      {"submission_id": "1", "status": "success", "score": 10, "counts": 30},
      {...}
    ]
  }
}
```

### 重新评测
- 路径：`PUT /api/submissions/{submission_id}/rejudge`
- 重新评测需覆盖原 `submission_id` 对应的内容
- 权限：仅管理员
- 参数：无（URL 路径参数：`submission_id`）
- 响应：
```json
{"code": 200, "msg": "rejudge started", "data": {"submission_id": "1", "status": "pending"}}
```
- 异常：401 未登录 (Step 4) / 403 权限不足 / 404 评测不存在

### 动态注册新语言 (Step 2)
- 路径：`POST /api/languages/`
- 参数：
  - `name` (str, 必填): 语言名称
  - `file_ext` (str, 必填): 代码文件扩展名
  - `compile_cmd` (str, 可选): 编译命令
  - `run_cmd` (str, 必填): 运行命令
  - `time_limit` (float, 可选): 默认单位为 "s"
  - `memory_limit` (int, 可选): 默认单位为 "MB"
- 权限：所有已登录用户
- 响应：
```json
{"code": 200, "msg": "language registered", "data": {"name": "go"}}
```
- 异常：400 参数错误 / 401 未登录 (Step 4) / 403 用户无权限

- 示例：
```json
{
  "name": "cpp",
  "file_ext": ".cpp",
  "compile_cmd": "g++ {src} -o {exe}", // 请注意，这里的 src 和 exe 需要是路径（如 test.cpp 不是路径，但是 ./test.cpp 或 /root/test.cpp 是路径）
  "run_cmd": "{exe}",
  "time_limit": 1.0,
  "memory_limit": 128
}
```
```json
{
  "name": "python",
  "file_ext": ".py",
  "run_cmd": "python3 {src}",
  "time_limit": 1.0,
  "memory_limit": 128
}
```

### 查询支持语言列表 (Step 2)
- 路径：`GET /api/languages/`
- 响应：
```json
{"code": 200, "msg": "success", "data": {"name": ["python", "cpp"]}}
```

---

## 3. 用户管理相关接口（Step 4）

### 用户登录
- 路径：`POST /api/auth/login`
- 参数：`username` (str, 必填), `password` (str, 必填)
- 响应：
```json
{"code": 200, "msg": "login success", "data": {"user_id": "1", "username": "alice", "role": "user"}}
```
- 异常：400 参数错误 / 401 用户名或密码错误 / 403 用户被禁用（Step 4）

### 用户登出
- 路径：`POST /api/auth/logout`
- 参数：无
- 权限：登录用户
- 响应：
```json
{"code": 200, "msg": "logout success", "data": null}
```
- 异常：401 未登录

### 创建管理员账户
- 路径：`POST /api/users/admin`
- 参数：`username` (str, 必填), `password` (str, 必填)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"user_id": "2", "username": "new_admin"}}
```
- 异常：400 用户名已存在 & 参数错误 / 401 未登录 (Step 4) / 403 用户无权限

### 用户注册
- 路径：`POST /api/users/`
- 参数：
  - `username` (str, 必填): 用户名
  - `password` (str, 必填): 密码
- 响应：
```json
{
  "code": 200, 
  "msg": "register success", 
  "data": 
  {
    "user_id": "1",
    "username": "xiaogang",
    "join_time": "2012-07-14", 
    "role": "user",
    "submit_count": 0,  // 用户提交数（按提交算，一个 problem 可贡献多次）
    "resolve_count": 0 // 用户通过数（按题目算，一个 problem 最多贡献一次）
  }
}
```
- 异常：400 用户名已存在 & 参数错误

### 查询用户信息
- 路径：`GET /api/users/{user_id}`
- 权限：仅本人或管理员
- 响应：
```json
{
  "code": 200, 
  "msg": "success", 
  "data": 
  {
    "user_id": "1",
    "username": "alice", 
    "join_time": "2012-07-14", 
    "role": "user",
    "submit_count": 80, 
    "resolve_count": 7
  }
}
```
- 异常：401 用户未登录 / 403 用户无权限 / 404 用户不存在

### 用户权限变更
- 路径：`PUT /api/users/{user_id}/role`
- 参数：
  - `role` (str, 必填): 新角色（如 "admin", "user", "banned"）
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "role updated", "data": {"user_id": "1", "role": "admin"}}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限 / 404 用户不存在 

### 用户列表查询
- 路径：`GET /api/users/`，参数：`page`、`page_size`（可选）
- 参数意义与 `GET /api/submissions/` 一致
- 权限：仅管理员
- 响应：
```json
{
  "code": 200, 
  "msg": "success", 
  "data": 
  {
    "total": 3, // 查询到的用户总数
    "users": 
    [
      {"user_id": "1", "username": "xiaoming", "role": "user", "join_time": "1924-08-17", "submit_count": 100, "resolve_count": 9},
      {"user_id": "2", "username": "xiaohong", "role": "user", "join_time": "1911-04-05", "submit_count": 90, "resolve_count": 8},
      {"user_id": "3", "username": "xiaogang", "role": "user", "join_time": "2012-07-14", "submit_count": 80, "resolve_count": 7},
    ]
  }
}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限 / 404 用户不存在

---

## 4. 评测日志相关接口（Step 5）

### 查询评测日志
- 路径：`GET /api/submissions/{submission_id}/log`
- 权限：仅本人（如果没有公开）或管理员
- 响应：
```json
{
  "code": 200, 
  "msg": "success", 
  "data": {
    "details": [ // 管理员可见 details；仅当该评测对应问题 public_cases 设置为 True 时用户可见
      {"id": 1, "result": "AC", "time": 1.01, "memory": 130},
      {"id": 2, "result": "TLE", "time": 1.01, "memory": 130},
      {"id": 3, "result": "MLE", "time": 1.01, "memory": 130},
    ],
    "score": 10,
    "counts": 30, // 总分数
  }
}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限 / 404 评测不存在

### 配置日志可见性
- 路径：`PUT /api/problems/{problem_id}/log_visibility`
- 权限：仅管理员
- 参数：
  - `public_cases` (bool，选填，默认为 False): 日志是否向所有人公开
- 响应：
```json
{
  "code": 200,
  "msg": "log visibility updated",
  "data": {"problem_id": "sum_3_numbers", "public_cases": True}
}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限 / 404 题目不存在

### 日志访问审计
- 路径：`GET /api/logs/access/`
- 权限：仅管理员
- 其中 `status` 作为返回值，记录这次访问状态
- 不必记录未登录 / `submission` 不存在 / 参数错误时访问记录。
- 参数：
  - `user_id` (str, 可选)：按用户筛选
  - `problem_id` (str, 可选)：按题目筛选
  - `page` (int, 可选)：页码
  - `page_size` (int, 可选)：每页数量
- 参数意义与 `GET /api/submissions/` 一致
- 请注意，这里的 `action` 只有 `view_logs` 一个操作
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {"user_id": "test", "problem_id": "sum_3_numbers", "action": "view_log", "time": "2024-06-01", "status": "403"} // 这次访问用户无权限
  ]
}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限

---

## 5. 数据持久化相关接口（Step 6）

### 系统重置
- 路径：`POST /api/reset/`
- 权限：仅管理员（测试环境可不校验）
- 参数：无
- 响应：
```json
{"code": 200, "msg": "system reset successfully", "data": null}
```
- 异常：401 用户未登录 / 403 权限不足
- 说明：清空所有数据，重置系统到初始状态。在测试环境中可能不需要管理员权限。

### 数据导出
- 路径：`GET /api/export/`
- 权限：仅管理员
- 响应：
```json
{
  "code": 200, 
  "msg": "success", 
  "data": {
    "users": [
      {
        "user_id": "1", 
        "username": "admin", 
        "password": "hash of the password here, cannot be plaintext, should be able to be imported back.",
        "role": "admin",
        "join_time": "2024-01-01",
        "submit_count": 100,
        "resolve_count": 10
      }
    ],
    "problems": [
      {
        "id": "sum_2",
        "title": "两数之和",
        "description": "输入两个整数a,b，输出它们的和",
        "input_description": "输入两个整数a和b",
        "output_description": "输出a+b的结果",
        "samples": [{"input": "1 2", "output": "3"}],
        "constraints": "|a|,|b| <= 10^9",
        "testcases": [{"input": "1 2", "output": "3"}],
        "hint": "",
        "source": "",
        "tags": [],
        "time_limit": 1.0,
        "memory_limit": 128,
        "author": "",
        "difficulty": "",
        "public_cases": True // 注意 public_cases 是 problem 的属性
      }
    ],
    "submissions": [
      {
        "submission_id": "1",
        "user_id": "1",
        "problem_id": "sum_2",
        "language": "python",
        "code": "a, b = map(int, input().split())\nprint(a + b)",
        "status": "success",
        "details": [
          {"id": 1, "result": "AC", "time": 1.01, "memory": 130},
          {"id": 2, "result": "AC", "time": 1.01, "memory": 130}
        ],
        "score": 100,
        "counts": 100,
      }
    ]
  }
}
```
- 异常：401 用户未登录 / 403 用户无权限 / 500 服务器异常

### 数据导入
- 路径：`POST /api/import/`，参数：`file` (上传)，内容就是`/api/export`导出的格式。
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "import success", "data": null}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限 / 500 服务器异常

---

## 6. 高级功能接口 (Advance)

### Advance 1: Special Judge
- 上传 SPJ 脚本：`POST /api/problems/{problem_id}/spj`
  - 参数（form-data 或 multipart）：
    - `file` (file, 必填): SPJ 脚本文件
- 删除 SPJ 脚本：`DELETE /api/problems/{problem_id}/spj`
  - 参数：无
- 查询题目评测策略：`GET /api/problems/{problem_id}`

### Advance 4: 代码查重
- 发起查重检测：`POST /api/plagiarism/`
  - 参数：
    - `problem_id` (str, 必填): 题目编号
    - `threshold` (float, 可选): 相似度阈值（如 0.8）
    - 其他查重相关参数（如有）
- 查询查重结果：`GET /api/plagiarism/{task_id}`（仅管理员）
- 下载查重报告：`GET /api/plagiarism/{task_id}/report`（仅管理员）

---

## 7. 安全性说明

系统实现时需要注意相关的安全性要求，包括但不限于输入验证、权限控制、数据加密等。