# Hermes 配置总结

> 迁移到远程服务器所需的所有配置信息

---

## 1. 模型配置

| 配置项 | 值 |
|--------|-----|
| 默认模型 | `minimaxai/minimax-m2.7` |
| Provider | `nvidia` |
| API 端点 | `https://integrate.api.nvidia.com/v1` |
| 最大对话轮次 | 90 |
| Gateway 超时 | 1800s |
| 推理强度 | medium |

---

## 2. MCP 服务器配置

### 2.1 CamoFox (浏览器自动化)

```yaml
mcp_servers:
  camofox:
    command: npx
    args:
      - -y
      - camofox-mcp
    env:
      CAMOFOX_URL: http://localhost:9377
```

**依赖**: camofox-browser Docker 容器运行在端口 9377

### 2.2 Blind-Spot (轻量级广告数据)

```yaml
mcp_servers:
  blind-spot:
    command: npx
    args:
      - mcp-remote
      - https://blindspot-demo-mcp.vercel.app/api/mcp
      - --transport
      - http-only
```

**状态**: 公开端点，无需鉴权

### 2.3 AdIntel-HF (完整版广告数据)

```yaml
mcp_servers:
  AdIntel-HF:
    command: npx
    args:
      - -y
      - mcp-remote
      - https://yongchengmu-adintel-mcp.hf.space
      - --transport
      - http-only
```

**状态**: 公开端点，无需鉴权，提供 23 个工具

---

## 3. 平台配置

### 3.1 Telegram

| 配置项 | 值 |
|--------|-----|
| Bot Token | `8632317662:...` (已隐藏) |
| Home Channel | `-5225550773` |
| 需要 @mention | true |
| 自动创建线程 | true |

### 3.2 API Server

| 配置项 | 值 |
|--------|-----|
| 启用 | true |
| 端口 | 8642 |

---

## 4. 工具集配置

```yaml
toolsets:
  - hermes-cli

platform_toolsets:
  cli:
    - hermes-cli
  telegram:
    - hermes-telegram
  discord:
    - hermes-discord
  whatsapp:
    - hermes-whatsapp
  slack:
    - hermes-slack
  signal:
    - hermes-signal
  homeassistant:
    - hermes-homeassistant
  qqbot:
    - hermes-qqbot
```

---

## 5. Agent 配置

```yaml
agent:
  max_turns: 90
  gateway_timeout: 1800
  restart_drain_timeout: 60
  api_max_retries: 3
  tool_use_enforcement: auto
  image_input_mode: auto
  reasoning_effort: medium
```

---

## 6. Terminal 配置

```yaml
terminal:
  backend: local
  timeout: 180
  docker_image: nikolaik/python-nodejs:python3.11-nodejs20
  container_cpu: 1
  container_memory: 5120MB
  container_disk: 51200MB
```

---

## 7. 记忆/内存配置

```yaml
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
  nudge_interval: 10
  flush_min_turns: 6
```

---

## 8. 语音配置 (TTS/STT)

### TTS

```yaml
tts:
  provider: edge
  edge:
    voice: en-US-AriaNeural
```

### STT

```yaml
stt:
  enabled: true
  provider: local  # whisper base model
```

---

## 9. 会话配置

```yaml
sessions:
  auto_prune: false
  retention_days: 90
  min_interval_hours: 24
```

---

## 10. 校验/审批配置

```yaml
approvals:
  mode: manual
  timeout: 60
  cron_mode: deny

security:
  redact_secrets: true
  tirith_enabled: true
  tirith_fail_open: true
```

---

## 11. 日志配置

```yaml
logging:
  level: INFO
  max_size_mb: 5
  backup_count: 3
```

---

## 12. 其他配置

| 配置项 | 值 |
|--------|-----|
| 时区 | 空 (默认) |
| 压缩 | 启用 (阈值 0.5, 目标比率 0.2) |
| 检查点 | 启用, 保留 50 个快照 |
| 代码执行模式 | project |
| 代码执行超时 | 300s |

---

## 服务器迁移清单

### 需要在服务器上安装

- [ ] Python 3.11+ (推荐 uv)
- [ ] Node.js 20+
- [ ] Docker (用于 camofox-browser)
- [ ] hermes-agent: `pip install hermes-agent`
- [ ] npx: `npm install -g npx`

### 需要在服务器上运行

1. **camofox-browser** (端口 9377)
   ```bash
   docker run -d --name camofox-browser -p 9377:9377 camoufox/browser:latest
   ```

2. **hermes-agent**
   ```bash
   hermes gateway run
   ```

3. **Open WebUI** (可选，端口 8080)
   ```bash
   docker run -d --name open-webui \
     -p 8080:8080 \
     -e OLLAMA_BASE_URL=http://localhost:8642/v1 \
     -e API_KEY=your-api-key \
     openwebui/webui:latest
   ```

### 需要复制到服务器的文件

```bash
# 1. 配置文件 (必须)
~/.hermes/config.yaml

# 2. 自定义 Skills (推荐)
~/.hermes/skills/

# 3. 记忆/用户配置 (可选)
~/.hermes/memory/
~/.hermes/.hermes_history
```

### 需要设置的环境变量

```bash
# NVIDIA API Key (如果使用 GPU 模型)
export NVIDIA_API_KEY="your-nvidia-api-key"

# Telegram Bot Token
export TELEGRAM_BOT_TOKEN="your-bot-token"
```

---

## 完整 config.yaml 路径

本地位置: `~/.hermes/config.yaml`

如需完整备份:
```bash
cp -r ~/.hermes ~/Desktop/projects/Hermes/hermes-config-backup/
```