# Python Developer

从前端转向 AI Agent 后端的学习记录：学习计划、阶段笔记和项目代码。

## 这个仓库是什么

一份 24 周学习计划及其执行记录。目标不是泛泛地学 Python，而是具备独立开发、测试和部署 AI Agent 后端的能力——能解释系统为什么这样设计，并在出错时知道从哪里排查。

起点是完整的前端开发经验加零基础的 Python。计划据此把时间集中在真正陌生的部分：SQL 与关系数据库、Agent 核心、生产工程。

**[→ AI Agent 后端学习计划](./AI%20Agent%20后端学习计划.md)**

## 学习主线

```text
Python 基础
→ Web 与 HTTP
→ FastAPI
→ SQLite 与 SQL
→ PostgreSQL
→ 大模型 API
→ Agent 工具调用
→ 上下文与记忆
→ 评测、安全与部署
```

所有知识点围绕同一个项目逐步升级：一个个人知识库学习助理 Agent，从命令行程序演进到可部署的完整后端。

## 目录结构

| 目录 | 周数 | 阶段成果 |
|---|---|---|
| `00 环境与编程入门` | 第 1 周第 1～2 天 | 能独立创建、运行和管理 Python 项目 |
| `01 Python 核心基础` | 第 1 周第 3 天～第 3 周 | 命令行学习记录程序 |
| `02 HTTP 与 FastAPI` | 第 4～5 周 | 学习任务 REST API |
| `03 SQL、数据库与测试` | 第 6～9 周 | PostgreSQL 数据后端 |
| `04 大模型 API` | 第 10～12 周 | 流式聊天后端 |
| `05 Agent 核心` | 第 13～18 周 | 可自主调用工具的 Agent 与固定评测集 |
| `06 记忆、评测、安全与部署` | 第 19～24 周 | 可观测、可测试、可部署的 Agent 后端 |

## 技术栈

```text
Python · uv · Ruff · FastAPI · Pydantic · Pydantic Settings
SQLAlchemy 2.x · Alembic · SQLite · PostgreSQL
pytest · HTTPX · 大模型官方 Python SDK
Docker · OpenTelemetry · pgvector（按需）· Redis（按需）· MCP
```

技术栈随阶段逐步引入，不在第一天全部安装。Agent 框架不在必选项中——先手写并理解最小 Agent 循环，再根据项目复杂度评估是否需要。

## 暂时不在范围内

算法竞赛与大量刷题、数据分析、爬虫、桌面 GUI、Python 高级元编程、微服务与 Kubernetes、多 Agent 协作、没有评测依据的复杂 RAG。

这些不是不重要，而是需要在合适的阶段学习。

## 其他笔记

- [NestJS vs FastAPI：开发 AI Agent 应用，究竟该选 Node.js 还是 Python](./NestJS%20vs%20FastAPI%EF%BC%9A%E5%BC%80%E5%8F%91%20AI%20Agent%20%E5%BA%94%E7%94%A8%EF%BC%8C%E7%A9%B6%E7%AB%9F%E8%AF%A5%E9%80%89%20Node.js%20%E8%BF%98%E6%98%AF%20Python.md)

## 开源许可

本仓库采用 [MIT License](./LICENSE)。
