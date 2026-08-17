# FAQ

> 此处收集 OJ 系统实验常见问题，持续补充中

## FastAPI 的参数校验在实际逻辑之前，导致 `422` 会比其他错误优先？

可使用 `Depends` 解决~ 参考

```python
from fastapi import Request, Depends, HTTPException, status
from fastapi.routing import APIRouter
import json
from models import ProblemModel
from services import auth, problem_ops

router = APIRouter()

@router.post("/api/problems/")
async def router_add_problem(
    request: Request,
    current_user=Depends(auth.get_current_user), # !!!!
):
    body = await request.body()
    try:
        body_data = json.loads(body)
        problem = ProblemModel(**body_data)
    except Exception as e:
        raise HTTPException(status_code=400, detail=f"Invalid body: {e}")

    await problem_ops.file_save_problem(problem.id, problem.model_dump())
    return {"code": 200, "msg": "add success", "data": {"id": problem.id}}
```


## API 要求返回 `400`，但是 FastAPI 默认返回 `422`？

可添加中间件解决~ 参考

```python
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()

class TestModel(BaseModel):
    name: str
    age: int

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=400,
        content={"detail": "Request body has validation errors", "errors": exc.errors()},
    )

@app.post("/items/")
async def create_item(item: TestModel):
    return item
```

## 评测日志中，测例详情是什么？

请大家区分一次评测和一个测例。测例是指 `test case`，即评测时的一组输入输出。一次评测中，评测结果类似

```json
"submissions": [
  {
    "submission_id": "1",
    "user_id": "1",
    "problem_id": "sum_2",
    "language": "python",
    "code": "a, b = map(int, input().split())\nprint(a + b)",
    "details": [
      {"id": 1, "result": "AC", "time": 1.01, "memory": 130},
      {"id": 2, "result": "AC", "time": 1.01, "memory": 130}
    ],
    "score": 100,
    "counts": 100,
  }
]
```

其中，一个测例是指 `{"id": 1, "result": "AC", "time": 1.01, "memory": 130}`，测例详情即为

```json
"details": [
  {"id": 1, "result": "AC", "time": 1.01, "memory": 130},
  {"id": 2, "result": "AC", "time": 1.01, "memory": 130}
],
```

如果用户不可见，评测时不返回此字段即可。

## 同学们的操作系统有 `linux`, `macos`, `windows`，最终评测应该如何进行呢？

最终评分会结合 `linux` 自动评测及线下人工评测，因此需要大家适配 `linux` 风格指令。使用 `macos` 的同学可以兼容所有评测会用到的 `linux` 指令，使用 windows 的同学建议使用 `WSL` 虚拟化容器。安装请参考 [WSL 安装文档](https://docs.eesast.com/docs/tools/wsl)，使用请参考 [RUNOOB Linux 教程](https://www.runoob.com/linux/linux-command-manual.html)。**也推荐大家使用生成式人工智能查询 Linux 操作，本次大作业基本只会用到 `g++`, `python` 等常用指令**

## 如何理解评测状态的 [pending, success, error] 与测试点结果的 [AC, WA, ...] 之间关系？

请区分两种状态，一种是一个 submission 的状态，一种是评测点结果的状态。提交的时刻，评测并未完成，返回一定是 `pending`；一个用户提交了评测，然后这个时候用户立刻查询评测列表，这时可能评测完成或者未完成，所以在查询评测列表的时候会有多种评测状态。

## [仓库] 如何将自己的仓库与助教提供的示例仓库关联，及时拉取最新更新？

请参考 [仓库拉取教程](./gitpull.md)

## [环境] Python/依赖环境如何配置？

- 推荐使用 Python 3.8 及以上版本。
- 建议使用 venv/conda 创建虚拟环境，安装依赖时可参考 requirements.txt。
- FastAPI/Flask、pytest、requests、uvicorn 等常用包需提前安装。

## [API] 如何查阅和测试 API？

- 所有接口、参数、异常、状态码详见 [api.md](api.md)。
- 推荐使用 Postman、curl 或 httpie 进行本地 API 测试。
- 注意接口权限（如部分接口需登录/管理员权限）。

## [评测] 评测流程和判题标准有哪些注意事项？

- 评测需严格按照题目输入输出格式，不能有多余提示语。
- 支持多语言评测，需动态注册语言时请参考 API 文档。
- 评测时需限制运行时间、内存，超限应返回 TLE/MLE。
- 日志接口可用于调试和查看评测详情。

## [权限] 用户权限和接口访问控制说明

- 普通用户仅能访问/操作自己的评测、信息、日志。
- 管理员可管理所有用户、题目、评测、日志等。
- 权限不足时接口会返回 403。

## [实验要求] 代码/报告/演示提交注意事项

- 代码需结构清晰、注释规范，按要求提交至指定仓库。
- 报告建议为 PDF，结构清晰，图文并茂。
- 需保证代码/报告/演示内容一致，严禁抄袭。

## 如何获取内存用量？

参考如下代码

```python
import subprocess
import psutil
import threading
import time

def monitor_memory(proc, mem_limit_mb, result_holder):
    p = psutil.Process(proc.pid)
    while proc.poll() is None:
        mem_usage = p.memory_info().rss / (1024 ** 2)  # in MB
        if mem_usage > mem_limit_mb:
            proc.kill()
            result_holder["status"] = "MLE"
            return
        time.sleep(0.05)
    result_holder["status"] = "OK"

def run_user_code(cmd, mem_limit_mb, timeout_sec):
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    result_holder = {"status": "OK"}
    t = threading.Thread(target=monitor_memory, args=(proc, mem_limit_mb, result_holder))
    t.start()

    try:
        proc.wait(timeout=timeout_sec)
    except subprocess.TimeoutExpired:
        proc.kill()
        result_holder["status"] = "TLE"

    t.join()
    return result_holder["status"]
```

## [其他] 常见问题与解答

- Q: 有参考模板吗？
  - A: 请见[仓库](https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template)，如权限问题请联系助教。
- Q: 必须实现前端吗？
  - A: 仅 Advance2 需要前端，基础模块不强制。
- Q: 评测/查重/日志等接口必须完全一致吗？
  - A: 必须严格遵循 api.md 文档。
- Q: 可以用 AI/LLM 辅助开发吗？
  - A: 可以，但需注明引用和来源，严禁抄袭。

---

> 感谢刘青乐、马子润、刘宇哲、陈晓宇、唐恒毅、刘汉唐、常钫铭、孙钰杰、王懋源等同学为文档提出的意见

如有其他疑问，欢迎在课程群或私聊助教留言，助教会及时补充解答。