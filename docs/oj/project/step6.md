<!-- ## Step 6：持久化与安全控制

--- -->

### 模块目标

实现系统数据的持久化存储和管理功能，确保数据的完整性和系统的可维护性。

---

### 任务 1：系统重置功能

**这个接口是测例所依赖的，如果本接口没有正确实现可能导致所有CI失败**

- **重置接口**：提供清空所有数据、重置系统到初始状态的功能。
- **权限控制**：仅管理员可执行。

**API 实现要求：**

- `POST /api/reset/` - 系统重置
- 注意，`Step 6` 中，测例可见性属于题目的属性
- 清空所有用户、题目、提交等数据，以及退出当前登录状态
- 重新创建初始管理员账户（`admin` / `admintestpassword`）

---

### 任务 2：数据导出功能

- **导出格式**：导出为 JSON 格式。
- **数据完整性**：包含用户、题目、提交。

**API 实现要求：**

- `GET /api/export/` - 数据导出
- 注意，`Step 6` 中，测例可见性属于题目的属性
- 返回固定格式的 JSON 数据
- 包含 users、problems、submissions 等数据的结构
- 用户导出的时候包含密码的哈希，这一步不允许明文存储密码。建议使用加盐哈希，详见step4的要求。

---

### 任务 3：数据导入功能

- **文件上传**：支持通过文件上传的方式导入数据。
- **数据验证**：验证导入数据的格式和完整性。
- **错误处理**：处理导入过程中的各种异常情况。

**API 实现要求：**

- `POST /api/import/` - 数据导入
- 支持文件上传
- 验证数据格式并导入到系统中
- 处理重复数据和冲突情况（对于冲突数据，以新导入的为准）

**文件上传实现提示：**

```python
from fastapi import UploadFile, File, HTTPException
import json

@app.post("/api/import/")
async def import_data(file: UploadFile = File(...)):
    # 1. 检查文件类型
    if not file.filename.endswith('.json'):
        raise HTTPException(status_code=400, detail="Only JSON files supported")
    
    # 2. 读取并解析文件
    content = await file.read()
    data = json.loads(content.decode('utf-8'))
    
    # 3. 验证数据格式并导入
    # validate_and_import(data)
    
    return {"code": 200, "msg": "import success", "data": None}
```

- 使用 `UploadFile` 和 `File(...)` 处理文件上传
- 通过 `await file.read()` 读取文件内容
- 注意异常处理：文件格式错误、JSON 解析错误等

---

### 评分细则

| 功能/接口                | 分值    | 评分说明                         |
|--------------------------|-------|----------------------------------|
| 系统重置功能             | 0     | 清空数据、重置到初始状态          |
| 数据导出功能             | 2     | 完整导出、格式正确               |
| 数据导入功能             | 3     | 文件上传、数据验证、错误处理      |
| **小计**                 | **5** |                                  |