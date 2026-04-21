# ScoutAgent 技术栈架构

ScoutAgent 是一个面向技术招聘场景的多智能体协作评估系统，整体由 **前端 SPA、后端 API/Agent 服务、本地高权限执行宿主、三类数据存储** 组成，统一通过 Docker Compose 编排部署。

## 总体架构

```
┌────────────────────────────────────────────────────────────────┐
│  Frontend (Browser SPA)                                        │
│  React 18 · TypeScript · Vite 5 · React Router 7 · Nginx       │
└──────────────────────────┬─────────────────────────────────────┘
                           │ REST + SSE (JWT Bearer)
┌──────────────────────────▼─────────────────────────────────────┐
│  Backend API (FastAPI · Uvicorn)                               │
│  路由层 · 鉴权 · 业务服务 · 流式输出                             │
├────────────────────────────────────────────────────────────────┤
│  Agent Runtime                                                 │
│  LangGraph · LangChain · OpenAI 兼容 LLM · MCP / Skills        │
├──────────────────┬──────────────────┬──────────────────────────┤
│  PostgreSQL 16   │  Redis Stack     │  Neo4j 5                 │
│  业务结构化数据   │  会话/检查点/缓存 │  技术知识图谱 (GraphRAG) │
└──────────────────┴──────────────────┴──────────────────────────┘
                           ▲
                           │ HTTP 拉取 / 心跳 / 结果回传
┌──────────────────────────┴─────────────────────────────────────┐
│  Local Agent Host (本机 127.0.0.1:17777)                        │
│  FastAPI · SQLite (aiosqlite) · OpenClaw SDK · 本地审批         │
└────────────────────────────────────────────────────────────────┘
```

## 前端

| 维度 | 技术选型 |
|------|---------|
| UI 框架 | React 18 |
| 语言 | TypeScript 5 |
| 构建工具 | Vite 5（`@vitejs/plugin-react`） |
| 路由 | React Router DOM 7 |
| 内容渲染 | `react-markdown` + `remark-gfm`（评估报告 / 对话内容） |
| 与后端通信 | `fetch` + REST，长任务通过 SSE（Server-Sent Events）流式接收 |
| 静态托管 | 生产环境由 Nginx (`nginx:alpine`) 提供静态资源与反向代理 |

源码目录：`frontend/src/`（`pages/`、`components/`、`api.ts`、`auth.tsx` 等）。

## 后端 API

| 维度 | 技术选型 |
|------|---------|
| 运行时 | Python（`python:slim` 容器） |
| Web 框架 | FastAPI |
| ASGI 服务器 | Uvicorn（`--workers 2`，多进程） |
| 数据访问 | SQLAlchemy ORM + `psycopg[binary]` 驱动 |
| 数据库迁移 | Alembic |
| 配置管理 | Pydantic / `pydantic-settings` + `python-dotenv` |
| 鉴权 | JWT（PyJWT）+ Bearer Token |
| 文件解析 | PyMuPDF（简历 PDF 解析）、`python-multipart`（上传） |
| 流式响应 | SSE 流（`stream_buffer.py`） |
| 可观测性 | LangSmith Tracing |

源码目录：`backend/app/`（`routers/`、`services/`、`models.py`、`schemas.py`、`auth/`、`agent/`、`eval/` 等）。

## Agent 框架

| 维度 | 技术选型 |
|------|---------|
| 编排框架 | **LangGraph**（多阶段 Agent Workflow、Planning、状态机） |
| 基础能力 | LangChain（Prompt、Tool、Chain 抽象） |
| LLM 接入 | OpenAI 兼容接口（`langchain-openai` + `openai` SDK） |
| Token 计算 | `tiktoken` |
| 状态/记忆 | LangGraph Checkpoint，由 **Redis** 持久化（`langgraph-checkpoint-redis`） |
| 工具扩展 | **MCP**（Model Context Protocol，`langchain-mcp-adapters` + `mcp`），通过 `backend/mcp_servers.json` 声明式配置；自研 Skills 模块化封装能力 |
| 知识图谱检索 | Neo4j 官方 Python Driver（GraphRAG，技能/框架/项目语义关联） |
| 业务模块 | `agent/planning`、`agent/assessment`、`agent/chat`、`agent/resume`、`agent/skills`、`agent/tools` |

## 本地执行宿主（Local Agent Host）

用于承接需要在用户本机以高权限执行的动作（如浏览器自动化），与云端解耦：

| 维度 | 技术选型 |
|------|---------|
| 框架 | FastAPI（仅监听 `127.0.0.1:17777`） |
| 本地存储 | SQLite + `aiosqlite`（已认领请求、审批记录、错误日志） |
| 通信 | `httpx` 主动向云端心跳 / 拉取请求 / 回传结果 |
| 操作适配 | OpenClaw SDK（浏览器与桌面操作） |
| 安全 | `cryptography`，本地审批流程（approve / reject） |

## 数据存储

| 存储 | 版本/镜像 | 用途 |
|------|----------|------|
| PostgreSQL | `postgres:16` / `postgres:alpine`（生产） | 企业、JD、候选人、CV、申请、评估等结构化业务数据 |
| Redis | `redis/redis-stack-server:latest` | 会话状态、缓存、LangGraph Checkpoint 持久化 |
| Neo4j | `neo4j:5` / `neo4j:latest`（生产） | 技术知识图谱（技能、框架、上下位关系），支撑 GraphRAG |
| SQLite | 本地文件 | 本地 Agent Host 的轻量状态 |

## 部署

| 维度 | 方案 |
|------|------|
| 容器编排 | Docker Compose（开发：`docker-compose.yml`；生产：`docker-compose.prod.yml`） |
| 服务拓扑（生产） | `frontend`（Nginx 80） → `backend`（FastAPI 8000）→ `postgres` / `neo4j` / `redis` |
| 镜像构建 | 后端：基于 `python:slim`，国内 PyPI 镜像加速；前端：`node:18-alpine` 多阶段构建后由 `nginx:alpine` 托管 |
| 健康检查 | 各服务均配置 `healthcheck`（`pg_isready`、`redis-cli ping`、Neo4j HTTP、`/api/health`） |
| 资源限制 | 生产 compose 为各服务设置 `memory limits/reservations`，并对 PostgreSQL、Neo4j 做 JVM/缓冲调优 |
| 配置注入 | `.env` 文件 + 环境变量（数据库连接、JWT 密钥、LLM Key、Neo4j 凭据等） |
| 本地一键启动 | `run.sh` 串行启动 Neo4j → PostgreSQL/Redis → Backend → Frontend |
| 本地宿主部署 | 用户本机独立运行 `python -m app.main`，与云端通过 HTTP 心跳建立连接 |
