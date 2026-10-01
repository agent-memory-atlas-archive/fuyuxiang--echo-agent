# 部署方案

Echo Agent 可前台运行，也可将 Gateway 注册为系统服务。默认工作区为 `~/.echo-agent`；实际配置和数据位置以 `echo-agent config dump` 为准。

## 本机运行

```bash
pip install "echo-agent[all]"
echo-agent setup
echo-agent run
```

`echo-agent run` 启动完整 Agent；仅当 `gateway.enabled: true` 时同时启动 Gateway。若只需单独启动 Gateway，可运行 `echo-agent gateway`（前台）；用户级后台服务可用 `echo-agent gateway install` 和 `echo-agent gateway start` 管理。服务配置、日志及平台差异见[后台服务](background-service.md)。

默认 Gateway 地址为 `127.0.0.1:58123`，健康检查为 `http://127.0.0.1:58123/api/v1/health`。也可以通过 `--config`、`--workspace` 或对应配置项指定独立的配置和工作区。配置文件搜索顺序与工作区主要目录见[文件系统布局](../reference/filesystem-layout.md)。

## Docker 示例

仓库不提供预构建 Docker 镜像。以下示例需要自行保存为 `Dockerfile` 和 Compose 文件。容器中的 Gateway 必须监听 `0.0.0.0` 才能接收转发流量；绑定非回环地址时必须配置 API 令牌，否则 Gateway 拒绝启动。

```dockerfile
FROM python:3.11-slim
RUN pip install "echo-agent[all]"
EXPOSE 58123
ENTRYPOINT ["echo-agent", "gateway", "--workspace", "/data/echo-agent", "--config", "/data/echo-agent/echo-agent.yaml"]
```

将配置文件保存为 `./echo-agent.yaml`。下面的令牌仅为占位示例，实际部署应使用随机生成的值，并限制该文件的访问权限：

```yaml
workspace: /data/echo-agent
gateway:
  host: "0.0.0.0"
  port: 58123
  auth:
    mode: allowlist
    api_tokens: ["replace-with-a-random-token"]
    allowed_users: ["cli:local"]  # 默认 CLI 用户；其他调用方按实际身份添加
    allowed_hosts: ["localhost", "127.0.0.1"]
```

```yaml
services:
  echo-agent:
    build: .
    ports:
      - "127.0.0.1:58123:58123"
    volumes:
      - echo-agent-data:/data/echo-agent
      - ./echo-agent.yaml:/data/echo-agent/echo-agent.yaml:ro
    restart: unless-stopped

volumes:
  echo-agent-data:
```

启动后可运行 `docker compose up -d`、`docker compose logs -f echo-agent`。端口只映射到宿主机回环地址；若通过域名对外提供服务，在宿主机部署反向代理并设置 `gateway.auth.allowed_hosts` 为该域名。容器内工具执行会使用容器中的文件和进程环境；需要访问项目文件时，应显式挂载相应路径，并按工具配置限制访问范围。

## 反向代理

Gateway 本身提供 HTTP 与 WebSocket。若需要 HTTPS、域名或代理层访问控制，使用 [Gateway 反向代理配置](../integrations/gateway/reverse-proxy.md) 中的 nginx 或 Caddy 示例。代理应转发原始 `Host` 和认证头，并支持 WebSocket 升级。健康探针的默认路径为 `/api/v1/health`。

## 多实例

各实例使用独立的配置文件、工作区、监听端口和服务标识。不要让多个 Agent 进程写同一工作区的 SQLite 数据库与记忆文件，也不要把当前示例直接当作共享数据库的负载均衡方案。例如，分别以前台启动两个实例：

```bash
echo-agent gateway --config /srv/agent-a/echo-agent.yaml --workspace /srv/agent-a
echo-agent gateway --config /srv/agent-b/echo-agent.yaml --workspace /srv/agent-b
```

两个配置中的 `gateway.port` 必须不同。实际生产环境还需分别规划令牌、数据备份和进程管理。

## 相关文档

- [Gateway 认证](../integrations/gateway/authentication.md)
- [备份与恢复](backup-restore.md)
- [安全加固](security-hardening.md)
