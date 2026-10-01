# Upgrade and Data Migration

Read the [Changelog](https://github.com/fuyuxiang/echo-agent/blob/master/CHANGELOG.md), check compatibility changes, and back up the workspace before upgrading. The commands below assume the default `~/.echo-agent` workspace and an installed user-level Gateway service. Use the actual paths for deployments with a custom `--workspace` or `--config`.

## Upgrade an installed package

```bash
echo-agent gateway stop
tar -czf "$HOME/echo-agent-backup-$(date +%Y%m%d).tar.gz" -C "$HOME" .echo-agent
pip install --upgrade "echo-agent[all]"
echo-agent config validate
echo-agent gateway start
echo-agent gateway status
```

If no background service is installed, stop the foreground `echo-agent run` or `echo-agent gateway` process before upgrading, then restart it. Stop writes before backing up the SQLite database. See [backup and restore](backup-restore.en.md) for other methods.

SQLite schema migrations run automatically when `echo_agent/storage/sqlite.py` initializes the database connection. **`echo-agent migrate` does not migrate the database schema.** If startup fails, inspect the error log before deciding whether to restore the pre-upgrade backup.

## What `migrate` does

The CLI `migrate` command handles USER memory ownership keys and imports legacy `MEMORY.*.md` shards:

| Command | Effect |
|---|---|
| `echo-agent migrate status` | Counts old `source_session` values matching `memory.principal_bindings` and optionally adoptable empty-scope USER entries; shows the latest memory backup |
| `echo-agent migrate run --dry-run` | Previews USER memory owner-key changes |
| `echo-agent migrate run` | After confirmation, backs up `user_memory.json` and changes matching owner keys |
| `echo-agent migrate run --adopt-empty` | Also adopts non-global USER entries with an empty scope; preview and verify the target owner first |
| `echo-agent migrate memory-md` | Imports facts from legacy `MEMORY.*.md` shards into memory storage |
| `echo-agent migrate rollback` | Restores the latest `user_memory.json.migbak-*` backup; **does not roll back SQLite schema** |

Run these commands only when a Changelog entry or an operations plan calls for memory ownership migration. `status` neither reports a database version nor lists pending SQL migrations.

## Roll back a release

`checkpoint restore` restores workspace file snapshots, excluding the database, sessions, memory, and logs. Downgrading the Python package does not reverse database changes. To restore pre-upgrade data:

1. Stop the Gateway or foreground process.
2. Keep a copy of the current data for diagnosis or another upgrade attempt.
3. Install the target older version and restore its matching full workspace backup.
4. Run `echo-agent config validate`, start the service, and inspect logs and critical features.

Do not use `echo-agent migrate rollback` as a database rollback. See [backup and restore](backup-restore.en.md) for restoration commands. For upgrades or downgrades spanning several Beta releases, read each intervening Changelog; do not assume every data change has a reversible migration script.
