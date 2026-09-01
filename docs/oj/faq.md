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

## [前端] Step 6 必须实现哪些页面？

Step 6 是基础模块，至少实现以下三组页面：

- 用户页面组：用户注册页面、用户登录页面、用户信息展示页面、用户管理页面；
- 题目页面组：题目列表展示页面、题目详情展示页面、新增/编辑题目页面；
- 评测与提交页面组：提交记录列表展示页面、提交评测详情页面。

所有页面操作均应通过后端 API 完成。具体要求见 [Step 6：前端交互](project/step6.md)。

## [AI 智能命题] 必须实现工具调用或 Agent Loop 吗？

不要求。工具调用是扩展模型能力的一种设计方式，属于设计参考，不是独立的硬性评分项。项目可以根据实际功能选择单次模型调用、固定工作流、工具调用或其他合理方案。R1 至 R4 和评分维度以 [AI 智能命题](project/advance.md)页面为准。

## [AI 智能命题] 正例中列出的功能都必须实现吗？

不要求。正例和反例用于说明设计方向，不规定唯一实现方案。评分时将结合题目合理性、测试用例有效性和功能易用性进行评价。

## [AI 智能命题] 模型配置需要包含哪些内容？

至少应支持提供商 URL、模型名称和模型密钥，并确保配置实际用于模型请求。不得将模型提供商、模型名称或模型密钥固定在代码中。模型密钥不得通过日志或普通页面响应明文泄露。

## [AI 智能命题] 怎样满足实时进度和中断要求？

实时进度应在任务执行期间持续反映当前状态，而不是等待任务全部结束后一次性返回。可以使用流式响应、SSE、WebSocket 或轮询等方式。

中断操作应实际终止当前任务或阻止其继续执行。仅停止前端动画或关闭进度窗口，但后台任务仍继续运行，不满足中断要求。

## [AI 智能命题] Token 用量和价格如何统计？

应统计并展示当前任务的 Token 用量和费用。模型接口能够分别提供输入、输出 Token 时，应分别记录。费用根据实际采用的模型价格计算，并说明价格单位和计价依据；模型接口不能提供完整用量时，应说明采用的统计或估算方式及其限制。

## [其他] 常见问题与解答

- Q: 必须实现前端吗？
  - A: 必须。前端交互已经调整为基础模块 Step 6。
- Q: API 接口必须完全一致吗？
  - A: 基础模块使用的接口必须严格遵循 api.md。AI 智能命题因设计方案不同，可以采用等价接口，但需要在项目文档中说明路径、参数、状态和响应结构。
- Q: 可以用 AI/LLM 辅助开发吗？
  - A: 可以，但需注明引用和来源，严禁抄袭。

---

!!! 致谢
    感谢刘青乐、马子润、刘宇哲、陈晓宇、唐恒毅、刘汉唐、常钫铭、孙钰杰、王懋源等同学为文档提出的意见。

如有其他疑问，欢迎在课程群或私聊助教留言，助教会及时补充解答。
