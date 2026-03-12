# 后端工具系统
backend/src/tools/ 目录包含 DeerFlow 的内置工具定义和注册。

## 结构
```
backend/src/tools/
├── builtins/              # 内置工具
│   ├── __init__.py
│   └── tools.py          # 工具注册和获取
└── tools.py               # 工具管理器
```

## 内置工具列表

| 工具 | 描述 |
|------|------|
| `bash` | 在沙箱中执行命令 |
| `read_file` | 读取文件内容 |
| `write_file` | 写入文件 |
| `str_replace` | 字符串替换（编辑文件） |
| `ls` | 列出目录内容 |
| `present_files` | 展示文件给用户 |
| `ask_clarification` | 请求用户澄清 |
| `view_image` | 查看图片 |
| `task` | 创建子智能体任务 |

## 添加新工具

1. 在 `backend/src/tools/builtins/` 创建工具文件
2. 使用 `@tool` 装饰器定义工具
3. 在 `tools.py` 中注册工具
4. 添加测试
