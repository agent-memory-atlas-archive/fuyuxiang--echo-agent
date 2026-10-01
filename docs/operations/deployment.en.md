# Deployment

Echo Agent can run in the foreground or as a registered Gateway service. The default workspace is `~/.echo-agent`; use `echo-agent config dump` to inspect the effective paths.

## Local process

```bash
pip install "echo-agent[all]"
echo-agent setup
echo-agent run
```

`echo-agent run` starts the full agent and starts the Gateway only when `gateway.enabled: true`. Use `echo-agent gateway` to run the Gateway separately in the foreground. For a managed background process, run `echo-agent gateway install` followed by `echo-agent gateway start`. See [background service](background-service.en.md).

The default Gateway address is `127.0.0.1:58123`, with a health check at `http://127.0.0.1:58123/api/v1/health`. Use `--config` and `--workspace` on supported commands for a separate instance. See the [filesystem layout](../reference/filesystem-layout.en.md) for configuration search and workspace paths.

## Docker example

The repository has no prebuilt Docker image. Save this example as a `Dockerfile`. Inside a container, the Gateway must bind to `0.0.0.0` for port forwarding; a non-loopback bind requires at least one API token or startup is refused.

```dockerfile
FROM python:3.11-slim
RUN pip install "echo-agent[all]"
EXPOSE 58123
ENTRYPOINT ["echo-agent", "gateway", "--workspace", "/data/echo-agent", "--config", "/data/echo-agent/echo-agent.yaml"]
```

Save the following as `./echo-agent.yaml`. Replace the token with a random value and restrict file access:

```yaml
workspace: /data/echo-agent
gateway:
  host: "0.0.0.0"
  port: 58123
  auth:
    mode: allowlist
    api_tokens: ["replace-with-a-random-token"]
    allowed_users: ["cli:local"]  # Default CLI user; add other callers explicitly
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

Start with `docker compose up -d` and inspect output with `docker compose logs -f echo-agent`. The published host port is loopback-only. For public access, add a host reverse proxy and list its domain in `gateway.auth.allowed_hosts`. Tool commands in this setup execute inside the container; mount project files explicitly when needed.

## Reverse proxy

For HTTPS or domain access, follow the [Gateway reverse proxy](../integrations/gateway/reverse-proxy.en.md) examples. Preserve the original Host and authentication headers and support WebSocket upgrades. The default health path is `/api/v1/health`.

## Multiple instances

Use a separate configuration, workspace, listen port, and service identity for each instance. Do not have multiple Agent processes write the same SQLite database or memory files. This is not a shared-database load-balancing configuration.

```bash
echo-agent gateway --config /srv/agent-a/echo-agent.yaml --workspace /srv/agent-a
echo-agent gateway --config /srv/agent-b/echo-agent.yaml --workspace /srv/agent-b
```

Set different `gateway.port` values in the two files, and plan credentials, backups, and process management separately.

## Related documentation

- [Gateway authentication](../integrations/gateway/authentication.en.md)
- [Backup and restore](backup-restore.en.md)
- [Security hardening](security-hardening.en.md)
