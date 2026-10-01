# 兼容性参考

本页以 `pyproject.toml` 和当前 CI 配置为准。项目处于 `0.x` Beta 阶段；升级前应阅读[更新记录](https://github.com/fuyuxiang/echo-agent/blob/master/CHANGELOG.md)并备份工作区。

## Python 与平台

`requires-python = ">=3.11"`。CI 测试 Python 3.11 和 3.12；更高版本满足安装条件，但当前未列入 CI 矩阵或包分类器。项目未在打包元数据中限定操作系统。具体通道、外部命令、浏览器和可选依赖在各操作系统上的表现需按部署环境验证；不要将未测试的平台或版本视作已获正式保证。

Gateway 使用 `aiohttp`，命令行使用 `argparse`。数据存储使用 SQLite；当前没有 PostgreSQL 后端。默认 Gateway 监听 `127.0.0.1:58123`，API 前缀为 `/api/v1`；没有默认的独立 Prometheus HTTP 监听端口。

## 必需依赖

以下列表从 `pyproject.toml` 的 `project.dependencies` 生成；安装器负责解析其传递依赖。

| 包 | 版本约束 |
|---|---|
| `pydantic` | `pydantic>=2.0` |
| `pydantic-settings` | `pydantic-settings>=2.0` |
| `pyyaml` | `pyyaml>=6.0` |
| `loguru` | `loguru>=0.7` |
| `aiohttp` | `aiohttp>=3.9` |
| `aiosqlite` | `aiosqlite>=0.20` |
| `croniter` | `croniter>=1.4` |
| `numpy` | `numpy>=1.24` |
| `fastembed` | `fastembed>=0.6` |
| `questionary` | `questionary>=2.0` |
| `prompt-toolkit` | `prompt-toolkit>=3.0` |
| `httpx` | `httpx>=0.23` |
| `rich` | `rich>=13.5` |

## 可选依赖组

组名与 `pyproject.toml` 的 `project.optional-dependencies` 一致。安装示例：`pip install "echo-agent[browser]"`。`all` 是预先列出的常用可选依赖集合，不应推断它等于未来所有新增组的并集。

| 组名 | 安装示例 |
|---|---|
| `openai` | `pip install "echo-agent[openai]"` |
| `anthropic` | `pip install "echo-agent[anthropic]"` |
| `bedrock` | `pip install "echo-agent[bedrock]"` |
| `gemini` | `pip install "echo-agent[gemini]"` |
| `allproviders` | `pip install "echo-agent[allproviders]"` |
| `vector` | `pip install "echo-agent[vector]"` |
| `container` | `pip install "echo-agent[container]"` |
| `process` | `pip install "echo-agent[process]"` |
| `fal` | `pip install "echo-agent[fal]"` |
| `weixin` | `pip install "echo-agent[weixin]"` |
| `browser` | `pip install "echo-agent[browser]"` |
| `otel` | `pip install "echo-agent[otel]"` |
| `tokenizers` | `pip install "echo-agent[tokenizers]"` |
| `documents` | `pip install "echo-agent[documents]"` |
| `tui` | `pip install "echo-agent[tui]"` |
| `all` | `pip install "echo-agent[all]"` |
| `dev` | `pip install "echo-agent[dev]"` |
| `docs` | `pip install "echo-agent[docs]"` |
| `skills` | `pip install "echo-agent[skills]"` |

## 升级与数据

SQLite 表结构迁移在连接初始化时自动执行。`echo-agent migrate` 只处理 USER 记忆归属与旧 `MEMORY.*.md` 分片，不是数据库表结构迁移命令。`checkpoint restore` 不恢复数据库、会话、记忆或日志。跨版本回退应停止服务并恢复与目标版本匹配的完整备份。操作步骤见[升级与数据迁移](../operations/upgrade-migrations.md)。
