# DeerFlow 开发者指南

## 项目目的

DeerFlow 是一个开源的超级智能体平台（Super Agent Harness），基于 LangGraph 和 LangChain 构建。它在更大的 AI 应用生态系统中担任智能体运行时的角色，为开发者提供构建能够实际"做事"的 AI 应用的完整基础设施。

**核心职责**:
- 提供智能体编排和状态管理（基于 LangGraph）
- 管理沙箱执行环境和文件系统
- 协调子智能体并行任务执行
- 维护跨会话的持久化记忆
- 集成多种工具和外部服务
- 提供可扩展的技能系统

**相关系统**:
- **LangChain/LangGraph** - 底层智能体框架
- **LLM 提供商** - OpenAI、Anthropic、Google、DeepSeek 等
- **工具服务** - Tavily、Jina AI、Firecrawl 等
- **IM 平台** - Telegram、Slack、飞书

## 环境搭建

### 前置条件

- **Python 3.12+** - 后端运行时
- **Node.js 22+** - 前端运行时
- **uv** - Python 包管理器（推荐）
- **pnpm** - Node.js 包管理器
- **Docker** - 沙箱容器运行时（可选，用于 Docker 沙箱模式）
- **Nginx** - 反向代理（开发环境自动启动）
- **API 密钥** - 至少一个 LLM 提供商的 API 密钥

### 安装

#### 克隆仓库

```bash
# 克隆仓库
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow

# 生成配置文件
make config
```

#### 配置模型

编辑 `config.yaml`，至少定义一个模型：

```yaml
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY  # 推荐使用环境变量
    max_tokens: 4096
    temperature: 0.7
```

#### 设置 API 密钥

选择以下方式之一：

**方式 A: 编辑 .env 文件（推荐）**

```bash
# 编辑项目根目录的 .env 文件
TAVILY_API_KEY=your-tavily-api-key
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key
INFOQUEST_API_KEY=your-infoquest-api-key
```

**方式 B: 导出环境变量**

```bash
export OPENAI_API_KEY=your-openai-api-key
export TAVILY_API_KEY=your-tavily-api-key
```

**方式 C: 直接在 config.yaml 中配置（不推荐用于生产环境）**

```yaml
models:
  - name: gpt-4
    api_key: your-actual-api-key-here
```

#### 安装依赖

```bash
# 检查前置条件
make check

# 安装所有依赖（后端 + 前端）
make install

# 或分别安装
cd backend && make install
cd frontend && pnpm install
```

#### 准备沙箱镜像（可选）

```bash
# 如果使用 Docker 沙箱模式
make setup-sandbox
```

### 环境变量

| 变量 | 必需 | 描述 | 示例 |
|------|------|------|------|
| `OPENAI_API_KEY` | 是* | OpenAI API 密钥 | `sk-...` |
| `ANTHROPIC_API_KEY` | 是* | Anthropic API 密钥 | `sk-ant-...` |
| `GOOGLE_API_KEY` | 是* | Google API 密钥 | `AIza...` |
| `DEEPSEEK_API_KEY` | 是* | DeepSeek API 密钥 | `sk-...` |
| `TAVILY_API_KEY` | 否 | Tavily 搜索 API 密钥 | `tvly-...` |
| `INFOQUEST_API_KEY` | 否 | InfoQuest API 密钥 | `...` |
| `TELEGRAM_BOT_TOKEN` | 否 | Telegram Bot Token | `123456789:ABC...` |
| `SLACK_BOT_TOKEN` | 否 | Slack Bot Token | `xoxb-...` |
| `SLACK_APP_TOKEN` | 否 | Slack App Token | `xapp-...` |
| `FEISHU_APP_ID` | 否 | 飞书 App ID | `cli_...` |
| `FEISHU_APP_SECRET` | 否 | 飞书 App Secret | `...` |
| `DEER_FLOW_CONFIG_PATH` | 否 | 配置文件路径 | `/path/to/config.yaml` |

\* 至少需要一个 LLM 提供商的 API 密钥

### 运行

#### 本地开发模式

```bash
# 启动所有服务（后端 + 前端 + Nginx）
make dev

# 访问应用
# http://localhost:2026
```

#### Docker 模式（推荐用于快速开始）

```bash
# 初始化 Docker 环境（只需执行一次）
make docker-init

# 启动服务
make docker-start

# 访问应用
# http://localhost:2026
```

#### 分别启动各服务

```bash
# 后端 LangGraph Server (Port 2024)
cd backend
make dev

# 后端 Gateway API (Port 8001)
cd backend
make gateway

# 前端开发服务器 (Port 3000)
cd frontend
pnpm dev

# Nginx 反向代理 (Port 2026)
nginx -c $(pwd)/docker/nginx.conf
```

### 常用命令

```bash
# 检查环境
make check

# 安装依赖
make install

# 启动开发服务器
make dev

# 运行测试
make test

# 代码格式化
make format

# 代码检查
make lint

# 清理构建产物
make clean
```

## 开发工作流

### 代码质量工具

| 工具 | 命令 | 目的 |
|------|------|------|
| **后端** | | |
| Ruff | `cd backend && ruff check .` | Python 代码检查 |
| Ruff Format | `cd backend && ruff format .` | Python 代码格式化 |
| Pytest | `cd backend && pytest` | 单元测试 |
| **前端** | | |
| ESLint | `cd frontend && pnpm lint` | TypeScript/JavaScript 检查 |
| Prettier | `cd frontend && pnpm format` | 代码格式化 |
| TypeScript | `cd frontend && pnpm typecheck` | 类型检查 |
| Next.js | `cd frontend && pnpm check` | 综合检查（lint + typecheck） |

### 提交前检查

1. 运行代码检查：`make lint`
2. 运行格式化：`make format`
3. 运行测试：`make test`
4. 确保所有服务能正常启动：`make dev`

### 分支策略

- `main` - 主分支，生产就绪代码
- `feature/*` - 新功能分支
- `fix/*` - Bug 修复分支
- `refactor/*` - 代码重构分支
- `docs/*` - 文档更新分支

### Pull Request 流程

1. 从 `main` 创建功能分支
2. 编写代码和测试
3. 运行 `make lint` 和 `make test`
4. 创建 PR 并填写描述
5. 处理审查反馈
6. Squash 合并到 `main`

## 常见任务

### 添加新的中间件

**需修改的文件**:
1. `backend/src/agents/middlewares/my_middleware.py` - 创建中间件文件
2. `backend/src/agents/lead_agent/agent.py` - 注册中间件到链中
3. `backend/tests/test_my_middleware.py` - 添加测试

**步骤**:

1. 创建中间件文件：

```python
# backend/src/agents/middlewares/my_middleware.py
from langchain.agents.middleware import AgentMiddleware
from src.agents.thread_state import ThreadState

class MyMiddleware(AgentMiddleware):
    async def __call__(self, state: ThreadState, config: RunnableConfig):
        # 前置处理
        # ...
        
        # 调用下一个中间件
        result = await self.next(state, config)
        
        # 后置处理
        # ...
        
        return result
```

2. 在 `agent.py` 中注册：

```python
from src.agents.middlewares.my_middleware import MyMiddleware

# 在中间件链中适当位置添加
chain = (
    ThreadDataMiddleware()
    | MyMiddleware()  # 添加你的中间件
    | SandboxMiddleware()
    # ...
)
```

3. 添加测试

### 添加新的工具

**需修改的文件**:
1. `backend/src/tools/builtins/my_tool.py` - 工具实现
2. `backend/src/tools/tools.py` - 注册工具
3. `backend/tests/test_my_tool.py` - 测试

**步骤**:

1. 创建工具文件：

```python
# backend/src/tools/builtins/my_tool.py
from langchain_core.tools import tool

@tool
def my_tool(param: str) -> str:
    """工具描述
    
    Args:
        param: 参数描述
    
    Returns:
        返回值描述
    """
    # 实现逻辑
    return result
```

2. 在 `tools.py` 中注册：

```python
from src.tools.builtins.my_tool import my_tool

def get_builtin_tools():
    return [
        # ...其他工具
        my_tool,
    ]
```

### 添加新的技能

**需创建的文件**:
1. `skills/public/my-skill/SKILL.md` - 技能定义

**步骤**:

1. 创建技能目录和文件：

```bash
mkdir -p skills/public/my-skill
touch skills/public/my-skill/SKILL.md
```

2. 编写 `SKILL.md`：

```markdown
# My Skill

技能的简要描述

## When to Use

描述何时使用这个技能

## Workflow

1. 步骤 1
2. 步骤 2
3. 步骤 3

## Best Practices

- 最佳实践 1
- 最佳实践 2

## Resources

- 脚本引用
- 配置文件引用
```

3. 技能会自动被系统发现和加载

### 添加新的子智能体

**需修改的文件**:
1. `backend/src/subagents/builtins/my_agent.py` - 子智能体实现
2. `backend/src/subagents/registry.py` - 注册子智能体

**步骤**:

1. 创建子智能体：

```python
# backend/src/subagents/builtins/my_agent.py
from src.subagents.config import SubagentConfig

def create_my_agent(config: SubagentConfig):
    """创建自定义子智能体"""
    # 定义工具集
    tools = [...]
    
    # 创建智能体
    agent = create_agent(
        model=config.model,
        tools=tools,
        system_prompt="子智能体系统提示词"
    )
    
    return agent
```

2. 在 `registry.py` 中注册：

```python
from src.subagents.builtins.my_agent import create_my_agent

BUILTIN_AGENTS = {
    # ...其他智能体
    "my_agent": create_my_agent,
}
```

### 添加新的 IM 渠道

**需修改的文件**:
1. `backend/src/channels/my_channel.py` - 渠道实现
2. `config.yaml` - 添加配置项

**步骤**:

1. 继承 `BaseChannel` 类：

```python
# backend/src/channels/my_channel.py
from src.channels.base import BaseChannel

class MyChannel(BaseChannel):
    async def start(self):
        """启动渠道监听"""
        pass
    
    async def handle_message(self, message):
        """处理接收到的消息"""
        pass
    
    async def send_message(self, user_id, message):
        """发送消息给用户"""
        pass
```

2. 在 `config.yaml` 中配置：

```yaml
channels:
  my_channel:
    enabled: true
    api_key: $MY_CHANNEL_API_KEY
```

### 添加新的 LLM 模型支持

**需修改的文件**:
1. `config.yaml` - 添加模型配置

**步骤**:

在 `config.yaml` 中添加新模型：

```yaml
models:
  - name: my-model
    display_name: My Model
    use: langchain_provider:ChatModel  # LangChain 类路径
    model: model-identifier
    api_key: $MY_API_KEY
    max_tokens: 4096
    temperature: 0.7
    # 可选特性
    supports_vision: true
    supports_thinking: true
```

### 修复 Bug

**流程**:
1. 编写复现 bug 的失败测试
2. 在代码中定位根因
3. 用最小改动修复
4. 验证测试通过
5. 检查其他地方是否有类似问题

**示例提交**: `fix(sandbox): handle permission denied errors gracefully`

## 编码规范

### Python（后端）

**文件组织**:
- 每个文件一个模块
- 使用 `__init__.py` 导出公共 API
- 相关文件放在同一目录

**命名**:

| 类型 | 约定 | 示例 |
|------|------|------|
| 模块 | snake_case | `thread_state.py` |
| 类 | PascalCase | `ThreadState` |
| 函数 | snake_case | `get_thread_state` |
| 常量 | SCREAMING_SNAKE | `MAX_RETRIES` |
| 私有属性 | _leading_underscore | `_internal_state` |

**类型注解**:

```python
# 必须使用类型注解
def process_message(message: str, config: RunnableConfig) -> ThreadState:
    pass

# 使用 Optional 表示可选
from typing import Optional
def get_user(user_id: str) -> Optional[User]:
    pass
```

**错误处理**:

```python
# 推荐：特定异常
raise ValueError(f"Invalid thread ID: {thread_id}")

# 避免：通用异常
raise Exception("Something went wrong")
```

**日志**:

```python
import logging

logger = logging.getLogger(__name__)

# 包含上下文
logger.info("Thread created", extra={"thread_id": thread_id})

# 使用适当级别
logger.debug()  # 开发详情
logger.info()   # 正常操作
logger.warning()  # 可恢复问题
logger.error()  # 需要关注的故障
```

### TypeScript（前端）

**文件组织**:
- 每个文件一个组件/模块
- 使用 `index.ts` 导出公共 API
- 组件文件使用 PascalCase

**命名**:

| 类型 | 约定 | 示例 |
|------|------|------|
| 组件 | PascalCase | `ThreadList.tsx` |
| Hook | camelCase + use 前缀 | `useThread.ts` |
| 工具函数 | camelCase | `formatDate.ts` |
| 常量 | SCREAMING_SNAKE | `API_BASE_URL` |
| 类型/接口 | PascalCase | `ThreadState` |

**组件结构**:

```tsx
// 1. 导入
import { useState } from 'react'
import { Button } from '@/components/ui/button'

// 2. 类型定义
interface MyComponentProps {
  title: string
  onSubmit: () => void
}

// 3. 组件定义
export function MyComponent({ title, onSubmit }: MyComponentProps) {
  // 3.1 Hooks
  const [isOpen, setIsOpen] = useState(false)
  
  // 3.2 回调函数
  const handleClick = () => {
    setIsOpen(!isOpen)
  }
  
  // 3.3 渲染
  return (
    <div>
      <h1>{title}</h1>
      <Button onClick={handleClick}>Toggle</Button>
    </div>
  )
}
```

**错误处理**:

```tsx
// 使用 try-catch 处理异步错误
try {
  await submitForm(data)
} catch (error) {
  console.error('Form submission failed:', error)
  toast.error('提交失败，请重试')
}
```

### 测试规范

**Python 测试**:

```python
# tests/test_thread_state.py
import pytest
from src.agents.thread_state import ThreadState

class TestThreadState:
    def test_create_thread_state(self):
        """should create thread state with default values"""
        state = ThreadState()
        assert state.thread_id is not None
    
    def test_thread_state_with_custom_values(self):
        """should create thread state with custom values"""
        state = ThreadState(thread_id="custom-id")
        assert state.thread_id == "custom-id"
```

**TypeScript 测试**:

```typescript
// __tests__/components/ThreadList.test.tsx
import { render, screen } from '@testing-library/react'
import { ThreadList } from '@/components/ThreadList'

describe('ThreadList', () => {
  it('should render thread list', () => {
    render(<ThreadList threads={mockThreads} />)
    expect(screen.getByText('Thread 1')).toBeInTheDocument()
  })
})
```

## 项目配置

### 配置文件结构

```yaml
# config.yaml

# 模型配置
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY
    max_tokens: 4096

# 沙箱配置
sandbox:
  use: src.sandbox.local_provider:LocalSandboxProvider
  # 或 Docker 模式
  # use: src.community.aio_sandbox:AioSandboxProvider
  # provisioner_url: http://provisioner:8002

# 智能体配置
agent:
  default_model: gpt-4
  recursion_limit: 100

# 记忆配置
memory:
  enabled: true
  file_path: ~/.deer-flow/memory.json
  debounce_seconds: 30

# IM 渠道配置
channels:
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
  
  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN
    app_token: $SLACK_APP_TOKEN
```

### 环境特定配置

使用 `DEER_FLOW_CONFIG_PATH` 环境变量指定不同的配置文件：

```bash
# 开发环境
export DEER_FLOW_CONFIG_PATH=config.dev.yaml

# 生产环境
export DEER_FLOW_CONFIG_PATH=config.prod.yaml
```

## 调试技巧

### 后端调试

1. **启用详细日志**:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

2. **使用调试脚本**:

```bash
cd backend
python debug.py
```

3. **检查 LangGraph 状态**:

```python
from src.agents.checkpointer import get_checkpointer

checkpointer = get_checkpointer()
state = await checkpointer.aget(thread_id)
print(state)
```

### 前端调试

1. **React DevTools**: 安装浏览器扩展
2. **TanStack Query DevTools**: 开发模式下自动启用
3. **网络请求**: 查看浏览器开发者工具的 Network 标签

### 沙箱调试

1. **进入沙箱容器**:

```bash
docker exec -it deer-flow-sandbox bash
```

2. **查看沙箱日志**:

```bash
docker logs deer-flow-sandbox
```

## 常见问题

### 1. 端口冲突

**问题**: 启动时提示端口已被占用

**解决方案**:
```bash
# 查找占用端口的进程
lsof -i :2024
lsof -i :2026
lsof -i :8001

# 终止进程
kill -9 <PID>
```

### 2. 依赖安装失败

**问题**: uv 或 pnpm 安装依赖时报错

**解决方案**:
```bash
# 清理缓存
uv cache clean
pnpm store prune

# 重新安装
make install
```

### 3. 沙箱权限问题

**问题**: 沙箱执行命令时权限被拒绝

**解决方案**:
```bash
# 检查文件权限
ls -la ~/.deer-flow/

# 修复权限
chmod -R 755 ~/.deer-flow/
```

### 4. 模型配置错误

**问题**: 提示模型未找到或配置错误

**解决方案**:
1. 检查 `config.yaml` 中的模型配置
2. 确保环境变量已正确设置
3. 验证 API 密钥是否有效

### 5. 前端构建失败

**问题**: Next.js 构建时报错

**解决方案**:
```bash
# 清理构建缓存
cd frontend
rm -rf .next node_modules
pnpm install
pnpm build
```

## 性能优化

### 后端优化

1. **启用记忆去抖**: 避免频繁的 LLM 调用
2. **使用轻量级模型进行总结**: 配置 `summarization.model_name`
3. **限制子智能体并发**: 调整 `subagent_limit_middleware` 参数

### 前端优化

1. **使用 React.memo**: 避免不必要的重新渲染
2. **懒加载组件**: 使用 `dynamic import`
3. **优化图片**: 使用 Next.js Image 组件

## 安全注意事项

1. **绝不提交密钥**: 使用 `.env` 文件和环境变量
2. **沙箱隔离**: 生产环境使用 Docker 沙箱模式
3. **输入验证**: 验证所有用户输入
4. **API 密钥轮换**: 定期轮换 API 密钥
5. **HTTPS**: 生产环境使用 HTTPS

## 有用的资源

- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [LangChain 文档](https://python.langchain.com/)
- [Next.js 文档](https://nextjs.org/docs)
- [FastAPI 文档](https://fastapi.tiangolo.com/)
- [项目贡献指南](../../CONTRIBUTING.md)
- [后端架构文档](../../backend/CLAUDE.md)
