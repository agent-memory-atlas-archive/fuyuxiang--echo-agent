# Quickstart

Follow these steps to install, configure a model, and start your first conversation. The time required depends on dependency downloads and the model service.

---

## Step 1: Install

```bash
pip install "echo-agent[all]"
```

Verify the installation:

```bash
echo-agent --version
# echo-agent 0.3.8
```

!!! tip "Virtual Environment"
    It's recommended to install in a virtual environment to avoid dependency conflicts:

    ```bash
    python -m venv ~/.echo-agent/venv
    source ~/.echo-agent/venv/bin/activate
    pip install "echo-agent[all]"
    ```

---

## Step 2: Run the Setup Wizard

```bash
echo-agent setup
```

The wizard guides you through:

1. **Choose a model provider** — OpenAI, Anthropic, Gemini, Bedrock, OpenRouter, or OpenAI-compatible endpoints
2. **Enter your API key** — the key for your chosen provider
3. **Select a model** — e.g., `gpt-4o`, `claude-sonnet-4-20250514`, `gemini-2.0-flash`
4. **Basic settings** — Agent name, language preference, etc.

Configuration is saved to `~/.echo-agent/config.yaml`.

!!! note "Manual Configuration"
    Skip the wizard and edit the config file directly:

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

## Step 3: Start the Agent

```bash
echo-agent run
```

Enter a message at the terminal prompt after startup. If the model connection fails, check the configuration with `echo-agent config validate`.

---

## Step 4: Send Your First Message

Type a message directly in the terminal and press Enter:

```
You: Hello, tell me about yourself
```

The Agent will respond and remember the conversation. You can continue chatting — it maintains context across turns.

---

## Step 5: Verify Success

In another terminal, inspect the configuration and cost report:

```bash
# Check running status
echo-agent status

# View cost statistics
echo-agent cost

```

!!! tip "Verify Memory"
    Restart the Agent and ask what you talked about last time. If it recalls, the memory system is working correctly.

---

## Next Steps

You've successfully run Echo Agent. Here's where to go next:

- **Connect more platforms** — Integrate with DingTalk, WeChat, Slack, and more. See [Channel Configuration](../integrations/channels/index.md)
- **Run in background** — Configure as a system service for 24/7 availability. See [Deployment Guide](../operations/deployment.md)
- **Open the Dashboard** — Manage your Agent via the web panel:
  ```bash
  echo-agent gateway
  # Open http://127.0.0.1:58123 (default port)
  ```
- **Explore skills** — View loaded skills on the Dashboard Skills page. `echo-agent skill list-staged` lists only candidates awaiting approval.
- **Scheduled tasks** — Create jobs on the Dashboard Cron page. The CLI `cron` command only supports `list`, `authorize`, and `revoke`; see [scheduled jobs](../guides/scheduled-jobs.en.md).
