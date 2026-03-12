# 后端智能体系统

backend/src/agents/ 目录包含 DeerFlow 的核心智能体系统，包括主智能体（Lead Agent）、中间件链、检查点和记忆系统。

## 结构

```
backend/src/agents/
├── lead_agent/          # 主智能体
│   ├── agent.py        # 智能体创建和配置
│   └── prompt.py       # 系统提示词模板
├── middlewares/        # 中间件
│   ├── thread_data_middleware.py      # 线程数据
│   ├── uploads_middleware.py         # 文件上传
│   ├── sandbox_middleware.py         # 沙箱环境
│   ├── memory_middleware.py         # 记忆提取
│   ├── title_middleware.py          # 标题生成
│   ├── view_image_middleware.py     # 图片查看
│   ├── clarification_middleware.py   # 澄清请求
│   ├── todo_middleware.py           # 计划模式
│   ├── subagent_limit_middleware.py # 子智能体限制
│   └── dangling_tool_call_middleware.py # 挂起工具调用
├── checkpointer/        # 状态检查点
│   ├── provider.py             # 同步提供者
│   └── async_provider.py         # 异步提供者
├── memory/            # 记忆系统
│   └── memory.py              # 记忆提取和存储
├── thread_state.py     # 线程状态定义
└── __init__.py       # 模块导出
```

## 关键文件

| 文件 | 目的 |
|------|------|
| `lead_agent/agent.py` | 主智能体创建和中间件链配置 |
| `lead_agent/prompt.py` | 系统提示词模板，技能注入逻辑 |
| `middlewares/*.py` | 各中间件实现 |
| `checkpointer/provider.py` | SQLite 状态持久化 |
| `memory/memory.py` | LLM 驱动的记忆提取 |
| `thread_state.py` | ThreadState 类型定义 |

## 依赖
**本模块依赖**:
- `src.config` - 配置管理
- `src.models` - LLM 模型封装
- `src.sandbox` - 沙箱系统
- `src.subagents` - 子智能体系统
- `src.tools` - 工具系统
- `langchain` - LLM 交互
- `langgraph` - 智能体编排

- `langchain-core` - 核心组件

**依赖本模块的**:
- `src.gateway` - Gateway API
- `src.channels` - IM 渠道
- 所有用户请求

## 核心功能
- 创建和配置主智能体
- 执行中间件链
- 管理线程状态
- 持久化检查点
- 提取和存储记忆
