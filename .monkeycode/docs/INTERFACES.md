# DeerFlow 接口文档

本文档描述 DeerFlow 系统的公开接口，包括 HTTP API、工具接口、技能定义和 IM 渠道接口。

## HTTP API

### 基础信息

**基础 URL**: `http://localhost:2026`

**认证**: 目前无认证要求（开发环境）

**请求格式**: JSON

**响应格式**: JSON

**路由规则**:
- `/api/langgraph/*` → LangGraph Server (Port 2024)
- `/api/*` (其他) → Gateway API (Port 8001)
- `/` (非 API) → Frontend (Next.js)

---

## Gateway API

### 模型管理

#### 获取可用模型列表

```http
GET /api/models
```

**响应**:

```json
{
  "models": [
    {
      "name": "gpt-4",
      "display_name": "GPT-4",
      "supports_vision": true,
      "supports_thinking": false
    },
    {
      "name": "claude-3-5-sonnet",
      "display_name": "Claude 3.5 Sonnet",
      "supports_vision": true,
      "supports_thinking": true
    }
  ]
}
```

---

### 技能管理

#### 获取技能列表

```http
GET /api/skills
```

**响应**:

```json
{
  "skills": [
    {
      "name": "deep-research",
      "path": "/mnt/skills/public/deep-research",
      "enabled": true,
      "description": "深度研究和报告生成"
    },
    {
      "name": "ppt-generation",
      "path": "/mnt/skills/public/ppt-generation",
      "enabled": true,
      "description": "PPT 演示文稿生成"
    }
  ]
}
```

#### 更新技能状态

```http
PUT /api/skills
```

**请求体**:

```json
{
  "skill_name": "deep-research",
  "enabled": false
}
```

**响应**:

```json
{
  "success": true,
  "skill_name": "deep-research",
  "enabled": false
}
```

#### 安装技能

```http
POST /api/skills/install
```

**请求体** (multipart/form-data):

```
skill_archive: <.skill 文件>
```

**响应**:

```json
{
  "success": true,
  "skill_name": "my-custom-skill",
  "path": "/mnt/skills/custom/my-custom-skill"
}
```

---

### 记忆管理

#### 获取记忆数据

```http
GET /api/memory
```

**响应**:

```json
{
  "user_context": {
    "work": "软件工程师，专注于 AI 和机器学习",
    "personal": "喜欢阅读科幻小说",
    "top_of_mind": "正在学习 LangGraph"
  },
  "history": [
    {
      "timestamp": "2024-01-15T10:30:00Z",
      "summary": "讨论了 LangGraph 的架构设计"
    }
  ],
  "facts": [
    {
      "content": "用户使用 Python 作为主要编程语言",
      "confidence": 0.9
    }
  ]
}
```

#### 强制重新加载记忆

```http
POST /api/memory/reload
```

**响应**:

```json
{
  "success": true,
  "message": "Memory reloaded successfully"
}
```

#### 获取记忆配置

```http
GET /api/memory/config
```

**响应**:

```json
{
  "enabled": true,
  "file_path": "~/.deer-flow/memory.json",
  "debounce_seconds": 30
}
```

#### 获取记忆状态（配置 + 数据）

```http
GET /api/memory/status
```

**响应**:

```json
{
  "config": {
    "enabled": true,
    "file_path": "~/.deer-flow/memory.json"
  },
  "data": {
    "user_context": {...},
    "history": [...],
    "facts": [...]
  }
}
```

---

### 文件上传

#### 上传文件到线程

```http
POST /api/threads/{thread_id}/uploads
```

**路径参数**:
- `thread_id` - 线程 ID

**请求体** (multipart/form-data):

```
files: <文件1>
files: <文件2>
```

**支持的文件类型**:
- PDF（自动转换为 Markdown）
- PPT/PPTX（自动转换为 Markdown）
- Excel/XLSX（自动转换为 Markdown）
- Word/DOCX（自动转换为 Markdown）
- 图片（PNG、JPG、GIF）
- 文本文件

**响应**:

```json
{
  "success": true,
  "files": [
    {
      "filename": "report.pdf",
      "path": "/mnt/user-data/uploads/thread-123/report.pdf",
      "size": 102400,
      "converted_to_markdown": true,
      "markdown_path": "/mnt/user-data/uploads/thread-123/report.md"
    }
  ]
}
```

#### 列出线程的上传文件

```http
GET /api/threads/{thread_id}/uploads/list
```

**路径参数**:
- `thread_id` - 线程 ID

**响应**:

```json
{
  "files": [
    {
      "filename": "report.pdf",
      "path": "/mnt/user-data/uploads/thread-123/report.pdf",
      "size": 102400,
      "uploaded_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

### 制品访问

#### 获取生成的制品

```http
GET /api/threads/{thread_id}/artifacts/{artifact_path}
```

**路径参数**:
- `thread_id` - 线程 ID
- `artifact_path` - 制品路径（支持多级路径）

**示例**:

```
GET /api/threads/thread-123/artifacts/outputs/report.pdf
GET /api/threads/thread-123/artifacts/workspace/data.json
```

**响应**: 文件内容（Content-Type 根据文件类型自动设置）

---

### MCP 管理

#### 获取 MCP 服务器配置

```http
GET /api/mcp/config
```

**响应**:

```json
{
  "servers": [
    {
      "name": "filesystem",
      "transport": "stdio",
      "command": "mcp-server-filesystem",
      "args": ["/mnt/user-data/workspace"],
      "enabled": true
    },
    {
      "name": "custom-api",
      "transport": "sse",
      "url": "https://api.example.com/mcp",
      "enabled": true,
      "oauth": {
        "flow": "client_credentials",
        "client_id": "...",
        "client_secret": "..."
      }
    }
  ]
}
```

#### 更新 MCP 服务器配置

```http
PUT /api/mcp/config
```

**请求体**:

```json
{
  "servers": [
    {
      "name": "filesystem",
      "enabled": false
    }
  ]
}
```

**响应**:

```json
{
  "success": true,
  "message": "MCP configuration updated"
}
```

---

## LangGraph API

### 创建助手运行

```http
POST /api/langgraph/threads/{thread_id}/runs
```

**路径参数**:
- `thread_id` - 线程 ID

**请求体**:

```json
{
  "assistant_id": "lead_agent",
  "input": {
    "messages": [
      {
        "role": "user",
        "content": "帮我分析一下这个 PDF 文件"
      }
    ]
  },
  "config": {
    "configurable": {
      "model_name": "gpt-4",
      "thinking_enabled": true,
      "is_plan_mode": false,
      "subagent_enabled": false
    }
  }
}
```

**响应**: SSE 流

### 流式响应格式

LangGraph 使用 Server-Sent Events (SSE) 进行流式响应：

```
event: values
data: {"messages": [...]}

event: messages-tuple
data: {"type": "ai", "content": "部分响应文本...", "tool_calls": []}

event: end
data: {"thread_id": "thread-123", "run_id": "run-456"}
```

**事件类型**:
- `values` - 完整状态值
- `messages-tuple` - 增量消息更新
- `end` - 运行结束

### 获取线程状态

```http
GET /api/langgraph/threads/{thread_id}/state
```

**响应**:

```json
{
  "values": {
    "messages": [...],
    "thread_id": "thread-123",
    "workspace_dir": "/path/to/workspace"
  },
  "next": ["agent"],
  "created_at": "2024-01-15T10:00:00Z",
  "modified_at": "2024-01-15T10:30:00Z"
}
```

### 获取线程历史

```http
GET /api/langgraph/threads/{thread_id}/history
```

**响应**:

```json
{
  "history": [
    {
      "role": "user",
      "content": "帮我分析一下这个 PDF 文件",
      "timestamp": "2024-01-15T10:00:00Z"
    },
    {
      "role": "assistant",
      "content": "好的，我已经分析了这个 PDF 文件...",
      "timestamp": "2024-01-15T10:01:00Z"
    }
  ]
}
```

### 搜索线程

```http
POST /api/langgraph/threads/search
```

**请求体**:

```json
{
  "query": "PDF 分析",
  "limit": 10,
  "offset": 0
}
```

**响应**:

```json
{
  "threads": [
    {
      "thread_id": "thread-123",
      "title": "PDF 文件分析",
      "created_at": "2024-01-15T10:00:00Z",
      "message_count": 5
    }
  ],
  "total": 15
}
```

---

## 工具接口

### 内置工具

#### bash

执行 shell 命令（在沙箱环境中）

```python
@tool
def bash(command: str) -> str:
    """Execute a bash command in the sandbox.
    
    Args:
        command: The bash command to execute
    
    Returns:
        Command output or error message
    """
```

**示例**:

```json
{
  "name": "bash",
  "arguments": {
    "command": "ls -la /mnt/user-data/workspace"
  }
}
```

#### read_file

读取文件内容

```python
@tool
def read_file(file_path: str) -> str:
    """Read a file from the sandbox filesystem.
    
    Args:
        file_path: Path to the file (virtual path in sandbox)
    
    Returns:
        File content as string
    """
```

#### write_file

写入文件

```python
@tool
def write_file(file_path: str, content: str) -> str:
    """Write content to a file in the sandbox.
    
    Args:
        file_path: Path to the file (virtual path)
        content: Content to write
    
    Returns:
        Success message or error
    """
```

#### str_replace

字符串替换（用于编辑文件）

```python
@tool
def str_replace(file_path: str, old_str: str, new_str: str) -> str:
    """Replace a string in a file.
    
    Args:
        file_path: Path to the file
        old_str: String to replace
        new_str: Replacement string
    
    Returns:
        Success message or error
    """
```

#### ls

列出目录内容

```python
@tool
def ls(directory: str = "/mnt/user-data/workspace") -> str:
    """List directory contents.
    
    Args:
        directory: Directory path to list
    
    Returns:
        Directory listing as formatted string
    """
```

#### present_files

展示生成的文件给用户

```python
@tool
def present_files(files: list[str], title: str = "Generated Files") -> str:
    """Present generated files to the user.
    
    Args:
        files: List of file paths to present
        title: Title for the presentation
    
    Returns:
        Confirmation message
    """
```

#### ask_clarification

请求用户澄清

```python
@tool
def ask_clarification(question: str) -> str:
    """Ask the user for clarification.
    
    Args:
        question: Question to ask the user
    
    Returns:
        User's response (execution is interrupted)
    """
```

#### view_image

查看图片（视觉模型）

```python
@tool
def view_image(image_path: str) -> str:
    """View an image file.
    
    Args:
        image_path: Path to the image file
    
    Returns:
        Image description or confirmation
    """
```

#### task

创建子智能体任务

```python
@tool
def task(description: str, tasks: list[str]) -> str:
    """Create and execute subagent tasks.
    
    Args:
        description: Overall task description
        tasks: List of subtasks to execute in parallel
    
    Returns:
        Combined results from all subtasks
    """
```

---

### 社区工具

#### tavily_search

使用 Tavily 进行网络搜索

```python
@tool
def tavily_search(query: str, max_results: int = 5) -> str:
    """Search the web using Tavily.
    
    Args:
        query: Search query
        max_results: Maximum number of results
    
    Returns:
        Formatted search results
    """
```

#### jina_ai_fetch

使用 Jina AI 抓取网页内容

```python
@tool
def jina_ai_fetch(url: str) -> str:
    """Fetch and parse a web page using Jina AI.
    
    Args:
        url: URL to fetch
    
    Returns:
        Parsed content as markdown
    """
```

#### firecrawl_scrape

使用 Firecrawl 爬取网页

```python
@tool
def firecrawl_scrape(url: str) -> str:
    """Scrape a web page using Firecrawl.
    
    Args:
        url: URL to scrape
    
    Returns:
        Scraped content
    """
```

#### duckduckgo_image_search

使用 DuckDuckGo 搜索图片

```python
@tool
def duckduckgo_image_search(query: str, max_results: int = 5) -> str:
    """Search for images using DuckDuckGo.
    
    Args:
        query: Search query
        max_results: Maximum number of results
    
    Returns:
        List of image URLs and descriptions
    """
```

---

## 技能接口

### 技能定义格式

每个技能由一个 `SKILL.md` 文件定义：

```markdown
# Skill Name

技能的简要描述

## When to Use

描述何时使用这个技能，帮助智能体决定是否激活该技能

## Workflow

1. 步骤 1
2. 步骤 2
3. 步骤 3

## Best Practices

- 最佳实践 1
- 最佳实践 2

## Resources

- `scripts/setup.sh` - 初始化脚本
- `templates/report.md` - 报告模板
```

### 内置技能列表

| 技能名称 | 描述 |
|---------|------|
| `deep-research` | 深度研究和报告生成 |
| `ppt-generation` | PPT 演示文稿生成 |
| `image-generation` | AI 图片生成 |
| `video-generation` | 视频内容生成 |
| `podcast-generation` | 播客内容生成 |
| `data-analysis` | 数据分析 |
| `chart-visualization` | 图表可视化 |
| `frontend-design` | 前端设计指南 |
| `consulting-analysis` | 咨询分析 |
| `github-deep-research` | GitHub 深度研究 |
| `skill-creator` | 创建新技能 |
| `claude-to-deerflow` | Claude Code 集成 |
| `bootstrap` | 项目初始化 |
| `surprise-me` | 随机任务 |
| `vercel-deploy-claimable` | Vercel 部署 |
| `web-design-guidelines` | Web 设计指南 |

---

## IM 渠道接口

### Telegram Bot

**配置**:

```yaml
channels:
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
    allowed_users: []  # 空 = 允许所有用户
```

**命令**:

| 命令 | 描述 |
|------|------|
| `/new` | 开始新对话 |
| `/status` | 显示当前线程信息 |
| `/models` | 列出可用模型 |
| `/memory` | 查看记忆 |
| `/help` | 显示帮助 |

### Slack Bot

**配置**:

```yaml
channels:
  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN     # xoxb-...
    app_token: $SLACK_APP_TOKEN     # xapp-... (Socket Mode)
    allowed_users: []
```

**特点**: 使用 Socket Mode，无需公网 IP

### 飞书 Bot

**配置**:

```yaml
channels:
  feishu:
    enabled: true
    app_id: $FEISHU_APP_ID
    app_secret: $FEISHU_APP_SECRET
```

**特点**: 使用 WebSocket 长连接，无需公网 IP

---

## Python 嵌入式客户端

DeerFlow 提供了 Python 嵌入式客户端，可以直接在代码中使用而无需运行 HTTP 服务：

```python
from src.client import DeerFlowClient

# 创建客户端
client = DeerFlowClient()

# 对话
response = client.chat("分析这个 PDF 文件", thread_id="my-thread")

# 流式响应
for event in client.stream("你好"):
    if event.type == "messages-tuple" and event.data.get("type") == "ai":
        print(event.data["content"])

# 配置管理
models = client.list_models()        # {"models": [...]}
skills = client.list_skills()        # {"skills": [...]}

# 更新技能
client.update_skill("web-search", enabled=True)

# 上传文件
client.upload_files("thread-1", ["./report.pdf"])
```

---

## 错误响应格式

所有 API 在发生错误时返回统一格式：

```json
{
  "detail": "Error message describing what went wrong",
  "error_code": "INVALID_REQUEST",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**常见错误码**:

| 错误码 | HTTP 状态码 | 描述 |
|--------|-----------|------|
| `INVALID_REQUEST` | 400 | 请求参数无效 |
| `NOT_FOUND` | 404 | 资源不存在 |
| `INTERNAL_ERROR` | 500 | 服务器内部错误 |
| `MODEL_NOT_FOUND` | 404 | 模型不存在 |
| `THREAD_NOT_FOUND` | 404 | 线程不存在 |
| `UPLOAD_FAILED` | 400 | 文件上传失败 |

---

## Webhook 事件（未来功能）

未来版本将支持 Webhook 事件通知：

```json
{
  "event": "thread.completed",
  "thread_id": "thread-123",
  "run_id": "run-456",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "message_count": 10,
    "artifacts": ["report.pdf"]
  }
}
```

**计划支持的事件**:
- `thread.created`
- `thread.completed`
- `file.uploaded`
- `artifact.generated`
- `subagent.started`
- `subagent.completed`
