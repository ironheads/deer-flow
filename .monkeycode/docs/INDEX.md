# DeerFlow 文档

DeerFlow（Deep Exploration and Efficient Research Flow）是一个开源的超级智能体平台，基于 LangGraph 和 LangChain 构建。本文档提供了系统架构、接口定义、开发指南和核心概念的完整说明。

**快速链接**: [架构](./ARCHITECTURE.md) | [接口](./INTERFACES.md) | [开发者指南](./DEVELOPER_GUIDE.md)

---

## 核心文档

### [架构文档](./ARCHITECTURE.md)
系统设计、技术栈、组件结构和数据流程。从这里开始了解 DeerFlow 如何运作。

**主要内容**:
- 系统概述和技术栈
- 项目结构和目录映射
- 子系统详解（Lead Agent、Middleware Chain、Sandbox、Subagent、Memory、Gateway API、IM Channels、Skills、Frontend Workspace）
- 架构图和时序图
- 数据流和关键设计决策

### [接口文档](./INTERFACES.md)
公开 API、工具接口、技能定义和 IM 渠道接口。集成或使用 DeerFlow 的参考文档。

**主要内容**:
- HTTP API（Gateway API 和 LangGraph API）
- 工具接口（内置工具和社区工具）
- 技能定义格式
- IM 渠道接口（Telegram、Slack、飞书）
- Python 嵌入式客户端
- 错误响应格式

### [开发者指南](./DEVELOPER_GUIDE.md)
环境搭建、开发工作流、编码规范和常见任务。贡献者必读。

**主要内容**:
- 项目目的和职责
- 环境搭建（前置条件、安装、运行）
- 开发工作流（代码质量、分支策略、PR 流程）
- 常见任务（添加中间件、工具、技能、子智能体、IM 渠道、LLM 模型）
- 编码规范（Python、TypeScript）
- 测试规范
- 项目配置
- 调试技巧和常见问题
- 性能优化和安全注意事项

---

## 模块

| 模块 | 描述 | README |
|------|------|--------|
| `backend/src/agents/` | 智能体系统，包括 Lead Agent、中间件链、检查点和记忆 | [README](../模块/backend-agents.md) |
| `backend/src/sandbox/` | 沙箱抽象层，提供隔离的执行环境 | [README](../模块/backend-sandbox.md) |
| `backend/src/subagents/` | 子智能体系统，支持并行任务执行 | [README](../模块/backend-subagents.md) |
| `backend/src/tools/` | 工具系统，包括内置工具和工具注册 | [README](../模块/backend-tools.md) |
| `backend/src/gateway/` | FastAPI Gateway，提供 REST API | [README](../模块/backend-gateway.md) |
| `backend/src/channels/` | IM 渠道集成（Telegram、Slack、飞书） | [README](../模块/backend-channels.md) |
| `frontend/src/core/threads/` | 前端线程管理，状态和生命周期 | [README](../模块/frontend-threads.md) |
| `frontend/src/core/api/` | 前端 API 客户端和数据获取 | [README](../模块/frontend-api.md) |
| `skills/public/` | 内置技能定义 | 各技能目录下的 SKILL.md |

---

## 核心概念

理解这些领域概念有助于导航代码库：

| 概念 | 描述 |
|------|------|
| [Thread（线程/会话）](./专有概念/Thread.md) | 用户与智能体交互的核心概念，代表一个独立的对话会话 |
| [Sandbox（沙箱）](./专有概念/Sandbox.md) | 隔离的代码执行环境，提供安全的文件系统和命令执行能力 |
| [Middleware（中间件）](./专有概念/Middleware.md) | 处理横切关注点的核心模式，每个中间件负责一个特定功能 |
| [Subagent（子智能体）](./专有概念/Subagent.md) | 异步任务委托和并发执行，支持复杂任务的分解 |
| [Memory（记忆）](./专有概念/Memory.md) | 跨会话持久化用户上下文和知识，通过 LLM 驱动的记忆提取 |

---

## 入门指南

### 项目新人？

按此路径学习：
1. **[架构文档](./ARCHITECTURE.md)** - 了解全局设计和核心组件
2. **[核心概念](#核心概念)** - 学习领域术语和关键抽象
3. **[开发者指南](./DEVELOPER_GUIDE.md)** - 搭建开发环境
4. **[接口文档](./INTERFACES.md)** - 探索公开 API 和工具

### 需要集成？

1. **[接口文档](./INTERFACES.md)** - HTTP API 契约和认证方式
2. **[架构文档](./ARCHITECTURE.md)** - 系统边界和数据流

### 首次贡献？

1. **[开发者指南](./DEVELOPER_GUIDE.md)** - 环境搭建和工作流
2. **[常见任务](./DEVELOPER_GUIDE.md#常见任务)** - 分步指南
3. **[编码规范](./DEVELOPER_GUIDE.md#编码规范)** - 代码风格

---

## 快速参考

### 命令

```bash
# 开发环境
make dev              # 启动开发服务器
make test             # 运行测试
make lint             # 代码检查
make format          # 代码格式化

# Docker 环境
make docker-init      # 初始化 Docker 环境
make docker-start     # 启动 Docker 服务

# 其他
make check            # 检查前置条件
make install          # 安装依赖
make clean            # 清理构建产物
```

### 重要文件

| 文件 | 目的 |
|------|------|
| `config.yaml` | 主配置文件，定义模型、沙箱、智能体等配置 |
| `.env` | 环境变量文件，存储 API 密钥 |
| `backend/langgraph.json` | LangGraph 服务器配置 |
| `backend/pyproject.toml` | Python 依赖和项目元数据 |
| `frontend/package.json` | Node.js 依赖和脚本 |
| `Makefile` | 构建命令和开发工具 |
| `skills/public/*/SKILL.md` | 技能定义文件 |

### 服务端口

| 服务 | 端口 | 描述 |
|------|------|------|
| Nginx | 2026 | 统一反向代理（访问入口） |
| LangGraph Server | 2024 | 智能体服务器 |
| Gateway API | 8001 | REST API 服务 |
| Frontend (dev) | 3000 | Next.js 开发服务器 |

**访问地址**: http://localhost:2026

---

## 技术栈概览

### 后端

- **Python 3.12+**
- **LangGraph 1.0+** - 智能体编排
- **LangChain 1.2+** - LLM 交互
- **FastAPI 0.115+** - Web 框架
- **Pydantic** - 数据验证
- **SQLite** - 状态持久化

### 前端

- **Node.js 22+**
- **Next.js 16.1+** - React 框架
- **React 19** - UI 库
- **TypeScript** - 类型系统
- **TanStack Query** - 状态管理
- **Tailwind CSS** - 样式框架

### 基础设施

- **Docker** - 容器运行时
- **Nginx** - 反向代理
- **uv** - Python 包管理
- **pnpm** - Node.js 包管理

---

## 常用链接

- **官方网站**: https://deerflow.tech/
- **GitHub 仓库**: https://github.com/bytedance/deer-flow
- **贡献指南**: ../../CONTRIBUTING.md
- **后端架构**: ../../backend/CLAUDE.md
- **配置指南**: ../../backend/docs/CONFIGURATION.md
- **MCP 服务器指南**: ../../backend/docs/MCP_SERVER.md

---

## 许可证

DeerFlow 是开源软件，采用 [MIT 许可证](../../LICENSE) 发布。

---

## 版本信息

- **当前版本**: 2.0
- **发布日期**: 2026 年 2 月
- **主要变化**: 从 v1.x 的深度研究框架完全重写为超级智能体平台

**注意**: DeerFlow 2.0 是全新重写，与 v1.x 不共享代码。如需 v1.x 版本，请访问 [`1.x` 分支](https://github.com/bytedance/deer-flow/tree/main-1.x)。
