# 升级与数据迁移

升级前阅读 [Changelog](https://github.com/fuyuxiang/echo-agent/blob/master/CHANGELOG.md)，确认兼容性变化，并备份工作区。下列命令以默认工作区 `~/.echo-agent` 和已安装的用户级 Gateway 服务为例。使用自定义 `--workspace` 或 `--config` 时，应替换为实际路径。

## 升级已安装的包

```bash
echo-agent gateway stop
tar -czf "$HOME/echo-agent-backup-$(date +%Y%m%d).tar.gz" -C "$HOME" .echo-agent
pip install --upgrade "echo-agent[all]"
echo-agent config validate
echo-agent gateway start
echo-agent gateway status
```

如果没有安装后台服务，先停止前台运行的 `echo-agent run` 或 `echo-agent gateway` 进程，升级后重新启动。备份 SQLite 数据库前应停止写入。其他备份方式见[备份与恢复](backup-restore.md)。

SQLite 表结构迁移由 `echo_agent/storage/sqlite.py` 在数据库连接初始化时自动执行。**`echo-agent migrate` 不是数据库表结构迁移命令。**启动失败时应先查看错误日志，再决定是否从升级前备份恢复。

## `migrate` 命令的用途

当前 CLI 的 `migrate` 子命令处理 USER 记忆的归属键，以及旧 `MEMORY.*.md` 分片的导入：

| 命令 | 作用 |
|---|---|
| `echo-agent migrate status` | 统计命中 `memory.principal_bindings` 的旧 `source_session` 和可选的空 scope USER 记忆；显示最近的记忆备份 |
| `echo-agent migrate run --dry-run` | 预览将改写的 USER 记忆归属键 |
| `echo-agent migrate run` | 确认后备份 `user_memory.json`，并改写命中绑定关系的归属键 |
| `echo-agent migrate run --adopt-empty` | 同时收编空 scope、非 global 的 USER 记忆；应先预演并确认目标 owner |
| `echo-agent migrate memory-md` | 将旧 `MEMORY.*.md` 分片中的事实导入记忆存储 |
| `echo-agent migrate rollback` | 用最近的 `user_memory.json.migbak-*` 备份覆盖该文件；**不会回滚 SQLite 表结构** |

只有更新说明或运维计划要求迁移记忆归属时才运行这些命令。`status` 不报告数据库版本，也不列出待执行的 SQL 迁移。

## 回退版本

`checkpoint restore` 恢复工作区文件快照，不包含数据库、会话、记忆和日志目录。回退 Python 包也不会逆向改写数据库。需要恢复升级前的数据时，使用升级前创建的完整备份：

1. 停止 Gateway 或前台进程。
2. 保存当前数据副本，以便排查或重新升级。
3. 安装目标旧版本，并将其对应的完整备份恢复到原工作区。
4. 运行 `echo-agent config validate`，启动并检查日志与关键功能。

不要把 `echo-agent migrate rollback` 用作数据库回退。具体恢复命令见[备份与恢复](backup-restore.md)。跨多个 Beta 版本升级或降级时，逐版本查阅更新说明；不应假定每个数据变化都有可逆的迁移脚本。
