# OJ 系统实验说明

> 本实验作业参考了清华大学韩文弢老师 Rust 程序设计训练 OJ 大作业（[作业背景](https://lab.cs.tsinghua.edu.cn/rust/projects/oj/background/)）。

## 实验目标

构建一个小型但功能完整的 Online Judge (OJ) 系统，分阶段实现，逐步掌握系统设计、API开发、安全控制等核心能力。

> 快速入门可参考[快速入门文档](https://lab.cs.tsinghua.edu.cn/rust/projects/oj/quick-start/)。

## 技术要求

**异步编程实践**：本次作业要求使用 FastAPI 的异步接口（`async def`）完成所有 API 开发，目的是让大家初步体验异步编程的概念和用法。异步编程是现代 Web 开发的重要技术，有助于提高应用程序的并发性能。**不使用异步编程接口将拿不到本次作业分数，请同学们务必注意。**

**项目规模**：为了让大家初步体验较大项目的开发，本次作业代码行数预计在两千行左右，请同学们合理规划时间，做好进度管理。

**提交规范**：要求按照 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/v1.0.0/) 规范编写 Git 提交信息，不符合规范的提交将酌情扣分。

---

## 基础模块（必做，共30分）

| Step  | 名称         | 主要功能描述                                   | 详细文档 |
|-------|--------------|-----------------------------------------------|----------|
| Step1 | 配置解析     | 题目配置加载、字段校验、异常处理               | [step1.md](project/step1.md) |
| Step2 | 评测控制     | 程序执行、资源限制、输出比对、动态注册语言     | [step2.md](project/step2.md) |
| Step3 | 用户系统     | 用户注册/更新、权限管理、人工判题接口          | [step3.md](project/step3.md) |
| Step4 | 任务状态管理 | 评测任务流转、调度、API                       | [step4.md](project/step4.md) |
| Step5 | 评测日志     | 日志结构化记录、日志接口、权限                 | [step5.md](project/step5.md) |
| Step6 | 数据持久化   | 数据存储、备份恢复、持久化结构与接口           | [step6.md](project/step6.md) |

---

## 进阶模块（选做，最多加10分）

| Advance | 名称         | 主要功能描述                                   |
|---------|--------------|-----------------------------------------------|
| Adv1    | Special Judge| 特殊题目评测，支持多种评测方式                 |
| Adv2    | 前端交互     | 极简前端界面（如 Streamlit），与后端交互        |
| Adv3    | 安全机制     | Docker 容器控制、命令过滤、资源限制            |
| Adv4    | 代码查重     | 查重算法实现、抄袭检测                         |

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
  - [Docker 容器技术](https://docs.docker.com/)

- **参考项目**:
  - [Codeforces](https://codeforces.com/) - 知名OJ平台
  - [LeetCode](https://leetcode.com/) - 编程练习平台
  - [HackerRank](https://www.hackerrank.com/) - 技术评测平台

## 致谢

本实验作业参考并借鉴了清华大学 Rust 程序设计训练 OJ 大作业的设计思想与文档结构，特此致谢韩文弢老师、陈嘉杰同学、陈晟祺同学等贡献者。