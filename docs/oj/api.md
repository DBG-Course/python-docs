# API 文档

---

## 目录
1. 题目管理相关接口（Step 1）
2. 评测相关接口（Step 2 & 3）
3. 用户管理相关接口（Step 4）
4. 评测日志相关接口（Step 5）
5. 前端交互说明（Step 6）
6. AI 智能命题接口（Advance）
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
| 409         | 资源状态冲突   | id 已存在/任务已经结束 |
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

### 编辑题目
- 路径：`PUT /api/problems/{problem_id}`
- 权限：所有已登录用户
- 参数：与添加题目的字段一致。路径中的 `problem_id` 表示待编辑题目；请求体中的 `id` 必须与其一致。
- 说明：使用请求体中的题目配置更新原题目。字段校验规则与添加题目相同。
- 响应：
```json
{"code": 200, "msg": "update success", "data": {"id": "sum_2"}}
```
- 异常：400 字段缺失、格式错误或 id 不一致 / 401 未登录 / 404 题目不存在

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

> Step 2 和 Step 3 的查询评测结果接口返回评测状态、总分以及必要的编译或错误信息；单个测试点的结果、时间和内存信息通过 Step 5 的评测日志接口查询。评测列表只返回用于列表展示的摘要信息。

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
    "submission_id": "123",
    "status": "success",
    "score": 10, // 本题获得分数
    "counts": 30, // 本题总分数（测试点数目 * 10）
    "compile_info": {
      "result": "success",
      "message": ""
    },
    "run_info": {
      "result": "finished",
      "message": "3 test cases finished"
    },
    "error_info": ""
  }
}
```
- `compile_info` 用于返回编译是否成功及编译器信息；解释型语言可返回 `null`。
- `run_info` 用于返回程序运行阶段的总体结果；各测试点的详细结果仍通过评测日志接口查询。
- `error_info` 用于返回评测任务级别的错误信息。不得在其中泄露服务器敏感路径、密钥等信息。
- `pending` 状态至少返回 `submission_id` 和 `status`；尚未产生的字段可返回 `null`。
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
    {"user_id": "test", "problem_id": "sum_3_numbers", "action": "view_logs", "time": "2024-06-01", "status": "403"} // 这次访问用户无权限
  ]
}
```
- 异常：400 参数错误 / 401 用户未登录 / 403 用户无权限

---

## 5. 前端交互说明（Step 6）

Step 6 不新增一套独立的业务数据接口。前端应调用 Step 1 至 Step 5 已定义的接口完成相应操作。

| 页面组 | 主要接口 |
| ------ | -------- |
| 用户页面组 | `POST /api/users/`、`POST /api/auth/login`、`POST /api/auth/logout`、`GET /api/users/{user_id}`、`PUT /api/users/{user_id}/role` |
| 题目页面组 | `GET /api/problems/`、`GET /api/problems/{problem_id}`、`POST /api/problems/`、`PUT /api/problems/{problem_id}`、`DELETE /api/problems/{problem_id}` |
| 评测与提交页面组 | `POST /api/submissions/`、`GET /api/submissions/`、`GET /api/submissions/{submission_id}`、`GET /api/submissions/{submission_id}/log` |

前端应根据接口的 HTTP 状态码和 `{code, msg, data}` 响应结构展示操作结果。登录会话、用户角色和资源可见性均以后端判断为准，不能仅依赖前端隐藏按钮实现权限控制。

### 测试支持：系统重置

系统重置接口供自动测试恢复初始环境使用，不属于 Step 6 的评分内容。

- 路径：`POST /api/reset/`
- 权限：仅管理员（测试环境可不校验）
- 参数：无
- 响应：
```json
{"code": 200, "msg": "system reset successfully", "data": null}
```
- 异常：401 用户未登录 / 403 权限不足
- 说明：清空测试产生的用户、题目和提交数据，退出当前登录状态，并重新创建初始管理员账户。

---

## 6. AI 智能命题接口（Advance）

AI 智能命题的页面结构和技术方案不作统一限制。以下接口用于说明 R1 至 R4 所需的数据交互，可按照项目设计采用等价的路径、传输协议或字段结构；采用不同设计时，应在项目文档中说明接口及其行为。

### 模型配置

- 建议路径：`PUT /api/ai/model-config`
- 权限：已登录用户
- 参数：
  - `provider_url` (str, 必填)：模型提供商 URL
  - `model` (str, 必填)：模型名称
  - `api_key` (str, 必填)：模型密钥
  - `input_price` (float, 可选)：输入 Token 单价
  - `output_price` (float, 可选)：输出 Token 单价
  - `price_unit` (int, 可选)：计价 Token 数量单位，如 `1000000`
- 响应示例：
```json
{
  "code": 200,
  "msg": "model config updated",
  "data": {
    "provider_url": "https://model-provider.example/v1",
    "model": "example-model",
    "api_key_configured": true,
    "input_price": 1.0,
    "output_price": 2.0,
    "price_unit": 1000000
  }
}
```

模型密钥不得通过查询接口或普通响应返回。若系统保存模型密钥，应采取与其敏感程度相适应的保护措施。

### 创建智能命题任务

- 建议路径：`POST /api/ai/problem-tasks/`
- 权限：已登录用户
- 参数：
  - `requirement` (str, 必填)：本次命题需求
  - `problem_id` (str, 可选)：需要参考或修改的已有题目编号
  - 其他与项目功能相关的参数
- 响应示例：
```json
{
  "code": 200,
  "msg": "task created",
  "data": {"task_id": "ai-task-1", "status": "pending"}
}
```
- 异常：400 参数错误 / 401 用户未登录 / 404 指定题目不存在 / 500 服务器异常

### 查询任务状态和结果

- 建议路径：`GET /api/ai/problem-tasks/{task_id}`
- 权限：任务创建者或管理员
- 响应示例：
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "task_id": "ai-task-1",
    "status": "running",
    "progress": "正在处理命题需求",
    "result": null,
    "usage": {
      "input_tokens": 1200,
      "output_tokens": 350,
      "total_tokens": 1550,
      "cost": 0.0019,
      "currency": "USD"
    }
  }
}
```

任务状态至少应能区分等待、执行、完成、中断和失败。任务完成后，`result` 返回值应能够被出题界面使用；具体结构由项目设计确定。

### 实时进度

可通过流式响应、SSE、WebSocket 或轮询实现实时进度。采用 SSE 时，可参考：

- 建议路径：`GET /api/ai/problem-tasks/{task_id}/events`
- 权限：任务创建者或管理员
- 事件示例：
```text
event: progress
data: {"task_id":"ai-task-1","status":"running","message":"正在处理命题需求"}

event: usage
data: {"input_tokens":1200,"output_tokens":350,"total_tokens":1550,"cost":0.0019,"currency":"USD"}
```

### 中断任务

- 建议路径：`PUT /api/ai/problem-tasks/{task_id}/cancel`
- 权限：任务创建者或管理员
- 响应示例：
```json
{
  "code": 200,
  "msg": "task cancelled",
  "data": {"task_id": "ai-task-1", "status": "cancelled"}
}
```
- 异常：401 用户未登录 / 403 用户无权限 / 404 任务不存在 / 409 任务已经结束

中断操作应实际终止任务或阻止任务继续执行，而不是只停止前端的进度展示。

### Token 用量与价格

统计结果至少包含当前任务的 Token 用量和费用。输入、输出 Token 采用不同单价时，可按以下方式计算：

```text
费用 = 输入 Token 数 / 计价单位 × 输入单价
     + 输出 Token 数 / 计价单位 × 输出单价
```

模型接口不能提供完整 Token 用量时，应在页面和项目文档中说明所采用的统计或估算方式。

---

## 7. 安全性说明

系统实现时需要注意相关的安全性要求，包括但不限于：

- 对请求参数、上传内容和模型返回数据进行必要校验；
- 所有权限判断均在后端完成，不能以前端是否显示操作入口代替权限校验；
- 密码和模型密钥等敏感信息不得明文记录在日志中，也不得通过普通查询接口返回；
- 调用外部模型服务时应处理超时、失败和异常响应，避免任务长期占用资源；
- 如实现外部工具调用，应限制可用工具和参数范围，并对文件写入、命令执行等有副作用的操作进行安全控制。
