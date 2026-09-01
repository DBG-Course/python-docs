# OJ 系统实验说明

## 实验目标

构建一个小型但功能完整的 Online Judge (OJ) 系统，分阶段实现，逐步掌握系统设计、API 开发、异步评测、权限控制和前后端交互等核心能力。进阶模块在此基础上引入大语言模型应用开发。

> 快速入门可参考[快速入门文档](https://lab.cs.tsinghua.edu.cn/rust/projects/oj/quick-start/)。

## 技术要求

**异步编程实践**：本次作业要求使用 FastAPI 的异步接口（`async def`）完成所有 API 开发，目的是让大家初步体验异步编程的概念和用法。异步编程是现代 Web 开发的重要技术，有助于提高应用程序的并发性能。**不使用异步编程接口将拿不到本次作业分数，请同学们务必注意。**

**项目规模**：为了让大家初步体验较大项目的开发，本次作业代码行数预计在两千行左右，请同学们合理规划时间，做好进度管理。

**提交规范**：要求按照 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/v1.0.0/) 规范编写 Git 提交信息，不符合规范的提交将酌情扣分。

---

## 基础模块（共30分）

| Step | 名称 | 主要功能描述 | 详细文档 |
| ---- | ---- | ------------ | -------- |
| Step 1 | 题目管理 | 题目配置加载、字段校验、增删改查 | [step1.md](project/step1.md) |
| Step 2 | 评测控制 | 程序执行、资源限制、输出比对、动态注册语言 | [step2.md](project/step2.md) |
| Step 3 | 评测管理 | 提交记录查询、状态管理、重新评测 | [step3.md](project/step3.md) |
| Step 4 | 用户管理 | 用户注册、登录、权限管理、用户信息查询 | [step4.md](project/step4.md) |
| Step 5 | 评测日志 | 测试点明细、日志查询、可见性与访问审计 | [step5.md](project/step5.md) |
| Step 6 | 前端交互 | 用户、题目和评测提交页面，与后端 API 对接 | [step6.md](project/step6.md) |

---

## 进阶模块（共 10 分）

| 模块 | 名称 | 主要功能描述 | 详细文档 |
| ---- | ---- | ------------ | -------- |
| Advance | AI 智能命题 | 在 OJ 系统上实现 AI 辅助命题 | [advance.md](project/advance.md) |

---

## API 文档

所有接口、参数、异常、状态码等详见 [api.md](api.md)。

---

## 评分标准

参见 [requirements.md](requirements.md)

---

## 学习资源

- **技术教程**:
  - [系统设计基础](https://github.com/donnemartin/system-design-primer)
  - [Python 异步编程](https://docs.python.org/3/library/asyncio.html)
  - [Agent 架构：从文本生成到工具调用](https://lab.cs.tsinghua.edu.cn/rust/projects/agent/agent-architecture/)

- **参考项目**:
  - [Codeforces](https://codeforces.com/) - 知名OJ平台
  - [LeetCode](https://leetcode.com/) - 编程练习平台
  - [HackerRank](https://www.hackerrank.com/) - 技术评测平台

!!! 致谢
    本实验作业参考了韩文弢老师开设的程序设计训练（Rust）[OJ 大作业](https://lab.cs.tsinghua.edu.cn/rust/projects/oj/background/)的设计思想与文档结构。转载前已获得授课教师授权。在此对原作者及相关文档贡献者表示感谢。
