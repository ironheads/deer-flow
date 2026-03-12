# Sandbox（沙箱）

Sandbox（沙箱）是 DeerFlow 中隔离代码执行环境的核心组件。它提供了一个安全的、隔离的环境，智能体可以在其中执行命令、读写文件，而不会影响宿主系统。

## 什么是 Sandbox？

Sandbox 是一个隔离的执行环境，为智能体提供文件系统和命令执行能力。每个 Thread 都有独立的沙箱环境，确保不同会话之间的完全隔离。沙箱支持本地执行和 Docker 容器执行两种模式。

**关键特征**:
- **隔离性**: 每个 Thread 的沙箱环境完全隔离
- **安全性**: 限制对宿主系统的访问，防止恶意操作
- **虚拟路径**: 使用虚拟路径映射，统一文件访问接口
- **多种模式**: 支持本地、Docker、Kubernetes 三种执行模式
- **可扩展**: 通过 Provider 接口支持自定义实现

## 代码位置

| 方面 | 位置 |
|------|------|
| 接口定义 | `backend/src/sandbox/interface.py` |
| 本地实现 | `backend/src/sandbox/local_provider.py` |
| Docker 实现 | `backend/src/community/aio_sandbox/provider.py` |
| 中间件 | `backend/src/sandbox/middleware.py` |
| 配置 | `backend/src/config/sandbox_config.py` |

## 结构

```python
# backend/src/sandbox/interface.py
class SandboxProvider(ABC):
    """沙箱提供者抽象接口"""
    
    @abstractmethod
    async def execute_command(self, command: str, cwd: str = "/mnt/user-data/workspace") -> CommandResult:
        """在沙箱中执行命令
        
        Args:
            command: 要执行的命令
            cwd: 工作目录
        
        Returns:
            CommandResult: 包含 stdout、stderr、exit_code
        """
        pass
    
    @abstractmethod
    async def read_file(self, file_path: str) -> str:
        """读取文件内容
        
        Args:
            file_path: 虚拟路径（如 /mnt/user-data/workspace/file.txt）
        
        Returns:
            文件内容
        """
        pass
    
    @abstractmethod
    async def write_file(self, file_path: str, content: str) -> None:
        """写入文件
        
        Args:
            file_path: 虚拟路径
            content: 文件内容
        """
        pass
    
    @abstractmethod
    async def list_dir(self, directory: str) -> list[str]:
        """列出目录内容
        
        Args:
            directory: 虚拟路径
        
        Returns:
            文件和目录列表
        """
        pass
```

### 关键字段

| 字段 | 类型 | 描述 | 约束 |
|------|------|------|------|
| `command` | `str` | 要执行的命令 | 非空 |
| `cwd` | `str` | 工作目录 | 默认 `/mnt/user-data/workspace` |
| `stdout` | `str` | 标准输出 | 执行结果 |
| `stderr` | `str` | 标准错误 | 错误信息 |
| `exit_code` | `int` | 退出码 | 0 表示成功 |

## 虚拟路径映射

沙箱使用虚拟路径系统，将容器内路径映射到线程特定的物理目录：

```
虚拟路径                      物理路径
/mnt/user-data/workspace/   → {thread_dir}/workspace/
/mnt/user-data/uploads/     → {thread_dir}/uploads/
/mnt/user-data/outputs/     → {thread_dir}/outputs/
/mnt/skills/                → deer-flow/skills/
```

**路径转换示例**:

```python
# 容器内路径
virtual_path = "/mnt/user-data/workspace/report.pdf"

# 转换为物理路径（本地模式）
physical_path = "~/.deer-flow/threads/thread-123/workspace/report.pdf"
```

## 执行模式

### 本地模式（Local）

在宿主文件系统上直接执行，适用于开发环境：

```yaml
# config.yaml
sandbox:
  use: src.sandbox.local_provider:LocalSandboxProvider
```

**特点**:
- 执行速度快
- 无需 Docker
- 安全性较低
- 适合本地开发

### Docker 模式

在 Docker 容器中执行，适用于生产环境：

```yaml
# config.yaml
sandbox:
  use: src.community.aio_sandbox:AioSandboxProvider
  image: deer-flow-sandbox:latest
  timeout: 300  # 秒
```

**特点**:
- 完全隔离
- 高安全性
- 需要预拉取镜像
- 适合生产部署

### Kubernetes 模式

通过 Provisioner 服务在 Kubernetes Pod 中执行：

```yaml
# config.yaml
sandbox:
  use: src.community.aio_sandbox:AioSandboxProvider
  provisioner_url: http://provisioner:8002
  kubeconfig_path: ~/.kube/config
```

**特点**:
- 支持大规模部署
- 资源限制和配额
- 高可用性
- 适合企业环境

## 工具集成

沙箱通过工具系统暴露给智能体：

### bash 工具

```python
@tool
def bash(command: str) -> str:
    """在沙箱中执行 bash 命令
    
    Args:
        command: bash 命令
    
    Returns:
        命令输出或错误信息
    """
    sandbox = get_sandbox_from_context()
    result = await sandbox.execute_command(command)
    return result.stdout or result.stderr
```

### 文件操作工具

```python
@tool
def read_file(file_path: str) -> str:
    """读取文件"""
    sandbox = get_sandbox_from_context()
    return await sandbox.read_file(file_path)

@tool
def write_file(file_path: str, content: str) -> str:
    """写入文件"""
    sandbox = get_sandbox_from_context()
    await sandbox.write_file(file_path, content)
    return f"Successfully wrote to {file_path}"

@tool
def ls(directory: str = "/mnt/user-data/workspace") -> str:
    """列出目录内容"""
    sandbox = get_sandbox_from_context()
    files = await sandbox.list_dir(directory)
    return "\n".join(files)
```

## 中间件集成

SandboxMiddleware 负责在每个请求开始时获取沙箱环境：

```python
# backend/src/sandbox/middleware.py
class SandboxMiddleware(AgentMiddleware):
    async def __call__(self, state: ThreadState, config: RunnableConfig):
        # 获取沙箱提供者
        provider = get_sandbox_provider()
        
        # 为当前线程获取沙箱环境
        sandbox = await provider.acquire(state["thread_id"])
        
        # 注入到状态
        state["sandbox"] = sandbox
        
        # 调用下一个中间件
        result = await self.next(state, config)
        
        # 释放沙箱
        await provider.release(sandbox)
        
        return result
```

## 配置示例

```yaml
# config.yaml

# 本地模式
sandbox:
  use: src.sandbox.local_provider:LocalSandboxProvider

# Docker 模式
sandbox:
  use: src.community.aio_sandbox:AioSandboxProvider
  image: deer-flow-sandbox:latest
  timeout: 300
  memory_limit: 512M
  cpu_limit: 1.0

# Kubernetes 模式
sandbox:
  use: src.community.aio_sandbox:AioSandboxProvider
  provisioner_url: http://provisioner:8002
  namespace: deer-flow
  resource_limits:
    memory: 512Mi
    cpu: 1000m
```

## 安全考虑

### 权限限制

1. **网络隔离**: Docker/K8s 模式默认禁用网络访问
2. **文件系统隔离**: 只能访问虚拟路径映射的目录
3. **资源限制**: 可配置内存、CPU、执行时间限制
4. **命令过滤**: 可配置禁止执行的命令

### 最佳实践

1. **生产环境使用 Docker/K8s 模式**
2. **设置合理的超时时间**
3. **限制资源使用**
4. **定期清理沙箱容器**
5. **监控沙箱执行日志**

## 常见问题

### 1. 权限错误

**问题**: 执行命令时提示权限被拒绝

**解决方案**:
```bash
# 检查文件权限
ls -la ~/.deer-flow/threads/

# 修复权限
chmod -R 755 ~/.deer-flow/threads/
```

### 2. Docker 镜像未找到

**问题**: 提示找不到沙箱镜像

**解决方案**:
```bash
# 拉取或构建镜像
make setup-sandbox

# 或手动构建
docker build -t deer-flow-sandbox:latest docker/sandbox/
```

### 3. 超时

**问题**: 命令执行超时

**解决方案**:
```yaml
# 增加超时时间
sandbox:
  timeout: 600  # 10 分钟
```

## 扩展开发

### 自定义 SandboxProvider

```python
# backend/src/custom/my_provider.py
from src.sandbox.interface import SandboxProvider, CommandResult

class MyCustomProvider(SandboxProvider):
    async def execute_command(self, command: str, cwd: str = "/") -> CommandResult:
        # 自定义实现
        pass
    
    async def read_file(self, file_path: str) -> str:
        # 自定义实现
        pass
    
    async def write_file(self, file_path: str, content: str) -> None:
        # 自定义实现
        pass
    
    async def list_dir(self, directory: str) -> list[str]:
        # 自定义实现
        pass
```

**配置**:

```yaml
sandbox:
  use: src.custom.my_provider:MyCustomProvider
  custom_param: value
```
