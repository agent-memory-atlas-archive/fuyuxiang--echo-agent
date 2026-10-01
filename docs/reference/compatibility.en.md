# Compatibility Reference

This page follows `pyproject.toml` and the current CI configuration. The project is in the `0.x` Beta series; read the [Changelog](https://github.com/fuyuxiang/echo-agent/blob/master/CHANGELOG.md) and back up the workspace before upgrading.

## Python and platforms

`requires-python = ">=3.11"`. CI tests Python 3.11 and 3.12. Newer versions meet the installation constraint but are absent from the current CI matrix and package classifiers. Package metadata does not restrict operating systems. Verify channels, external commands, browsers, and optional dependencies in the target deployment environment; an untested platform or version has no documented support guarantee.

The Gateway uses `aiohttp` and the CLI uses `argparse`. Storage uses SQLite; there is no PostgreSQL backend. The default Gateway address is `127.0.0.1:58123` with the `/api/v1` API prefix. No separate Prometheus HTTP port is enabled by default.

## Required dependencies

This list is generated from `project.dependencies` in `pyproject.toml`. The installer resolves transitive dependencies.

| Package | Version constraint |
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

## Optional dependency groups

These names come from `project.optional-dependencies` in `pyproject.toml`. Install one with, for example, `pip install "echo-agent[browser]"`. `all` is an explicitly listed set of common optional packages; do not assume it includes every group added in the future.

| Group | Installation |
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

## Upgrades and data

SQLite schema migrations run automatically during connection initialization. `echo-agent migrate` only handles USER memory ownership and legacy `MEMORY.*.md` shards; it is not a database schema migration command. `checkpoint restore` does not restore databases, sessions, memory, or logs. For a cross-version rollback, stop the service and restore a full backup matching the target version. See [upgrade and data migration](../operations/upgrade-migrations.en.md).
