# 快速上手

按以下步骤完成安装、模型配置和首次对话。所需时间取决于依赖下载和模型服务连接。

---

## 第一步：安装

```bash
pip install "echo-agent[all]"
```

验证安装成功：

```bash
echo-agent --version
# echo-agent 0.3.8
```

!!! tip "虚拟环境"
    建议在虚拟环境中安装，避免依赖冲突：

    ```bash
    python -m venv ~/.echo-agent/venv
    source ~/.echo-agent/venv/bin/activate
    pip install "echo-agent[all]"
    ```

---

## 第二步：运行 Setup 向导

```bash
echo-agent setup
```

向导会引导你完成以下配置：

1. **选择模型提供商** — OpenAI、Anthropic、Gemini、Bedrock、OpenRouter 或 OpenAI 兼容端点
2. **输入 API Key** — 对应提供商的密钥
3. **选择模型** — 如 `gpt-4o`、`claude-sonnet-4-20250514`、`gemini-2.0-flash` 等
4. **基本参数** — Agent 名称、语言偏好等

配置文件保存在 `~/.echo-agent/config.yaml`。

!!! note "也可手动配置"
    跳过向导直接编辑配置文件：

    ```bash
    mkdir -p ~/.echo-agent
    cat > ~/.echo-agent/config.yaml << 'EOF'
    models:
      default_model: gpt-4o
      providers:
        - name: openai
          models: [gpt-4o]
          api_key_env: OPENAI_API_KEY
    EOF
    export OPENAI_API_KEY=sk-your-key-here
    echo-agent config validate
    ```

---

## 第三步：启动 Agent

```bash
echo-agent run
```

启动后在终端提示符中输入消息。若连接模型失败，先运行 `echo-agent config validate` 检查配置。

---

## 第四步：发送第一条消息

在终端中直接输入消息并回车：

```
You: 你好，请介绍一下你自己
```

Agent 会回复并记住这次对话。你可以继续对话，Agent 具备上下文记忆能力。

---

## 第五步：验证成功

在另一个终端确认配置与成本报告可读取：

```bash
# 查看运行状态
echo-agent status

# 查看费用统计
echo-agent cost

```

!!! tip "验证记忆"
    重启 Agent 后再次对话，询问上次聊了什么——如果它能回忆起来，说明记忆系统工作正常。

---

## 下一步

你已经成功运行了 Echo Agent。接下来可以：

- **接入更多平台** — 将 Agent 接入钉钉、微信、Slack 等通道，见 [通道配置](../integrations/channels/index.md)
- **后台运行** — 配置为系统服务，保持 7×24 在线，见 [部署指南](../operations/deployment.md)
- **打开 Dashboard** — 通过 Web 面板管理 Agent：
  ```bash
  echo-agent gateway
  # 浏览器访问 http://127.0.0.1:58123（默认端口）
  ```
- **探索技能** — 在 Dashboard 的 Skills 页面查看已加载技能；`echo-agent skill list-staged` 只列出待审批技能。
- **定时任务** — 在 Dashboard 的 Cron 页面创建任务；CLI 的 `cron` 子命令只提供 `list`、`authorize` 和 `revoke`。参见[定时任务](../guides/scheduled-jobs.md)。
