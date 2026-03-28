# OpenClaw Setup Workshop

> Step-by-step guide to running **OpenClaw** — your personal AI assistant — powered by **NVIDIA NIM** via Docker.

---

## What is OpenClaw?

OpenClaw is a config-first AI agent framework built in TypeScript. Write a `SOUL.md`, run one command, and your agent is live across WhatsApp, Telegram, Slack, Discord, and 10+ other messaging platforms.

- No compilation required — pre-built image available
- Supports 150+ LLM models via NVIDIA NIM
- Dashboard at `http://localhost:18789`

---

## Prerequisites

You need **3 things** before starting:

### 1. Git
Download: https://git-scm.com/downloads

Verify:
```bash
git --version
```

### 2. Docker

**Option A — Docker Desktop (recommended)**
Download: https://www.docker.com/products/docker-desktop/

**Option B — Docker Engine (Linux only)**
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

Verify:
```bash
docker --version
```

### 3. NVIDIA API Key (free)

1. Go to https://build.nvidia.com
2. Sign in or create a free account
3. Click **"Get API Key"** in the top right
4. Copy your key — it starts with `nvapi-`

> The free tier gives you access to 150+ NIM models with generous rate limits.

---

## Setup

### Step 1 — Clone This Repo

Everything you need is already in this repo — config folder, workspace, and agent templates.

```bash
git clone https://github.com/Clawbuilders/openclaw-setup.git
cd openclaw-setup
```

### Step 2 — Pull the Pre-Built Image

```bash
docker pull ghcr.io/openclaw/openclaw:latest
```

Verify:
```bash
docker images | grep openclaw
```

### Step 3 — Add Your NVIDIA API Key

The `workshop/.openclaw/` folder already exists in the repo. Just copy the example and add your key:

```bash
cp workshop/.openclaw/.env.example workshop/.openclaw/.env
```

Then open `workshop/.openclaw/.env` and replace `nvapi-your-key-here` with your actual key:

```
LLM_PROVIDER=nvidia
NVIDIA_API_KEY=nvapi-your-actual-key-here
MODEL_VERSION=nvidia/llama-3.1-nemotron-ultra-253b-v1
```

> Get your free key at https://build.nvidia.com → click **"Get API Key"**

### Step 4 — Run OpenClaw

```bash
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ./workshop/.openclaw:/root/.openclaw \
  -v ./workshop/workspace:/workspace \
  --env-file ./workshop/.openclaw/.env \
  ghcr.io/openclaw/openclaw:latest
```

### Step 5 — Verify It's Running

```bash
docker logs openclaw
```

Look for a line containing a dashboard URL with a token, like:
```
Dashboard: http://localhost:18789/?token=xxxx
```

Open that URL in your browser — your OpenClaw dashboard is live.

---

## Quick Test — Chat with Your Agent

From the dashboard, type a message in the chat box. Or get your token from logs:

```bash
docker logs openclaw | grep token
```

---

## Available NVIDIA NIM Models

Swap the `MODEL_VERSION` in `~/.openclaw/.env` and restart the container:

| Model | Best For |
|-------|----------|
| `nvidia/llama-3.1-nemotron-ultra-253b-v1` | Flagship reasoning (default) |
| `nvidia/llama-3.3-nemotron-super-49b-v1` | Fast + strong reasoning |
| `meta/llama-3.3-70b-instruct` | Fast, general-purpose |
| `meta/llama-4-maverick-17b-128e-instruct` | Multimodal, 128k context |
| `meta/llama-4-scout-17b-16e-instruct` | Efficient, low latency |
| `deepseek-ai/deepseek-r1` | Deep reasoning |
| `mistralai/mistral-large-2-instruct` | Mistral via NIM |
| `qwen/qwq-32b` | Qwen reasoning model |

Browse all 150+ models: https://build.nvidia.com/models

---

## Connect a Messaging Channel (Optional)

OpenClaw bridges your agent to messaging apps. Example for Telegram:

1. Create a bot via [@BotFather](https://t.me/BotFather) on Telegram
2. Copy the bot token
3. Add to `~/.openclaw/.env`:
```bash
TELEGRAM_BOT_TOKEN=your-token-here
```
4. Restart the container:
```bash
docker restart openclaw
```

Supported channels: Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Teams, Google Chat, iMessage, and more.

---

## Managing the Container

```bash
# Stop
docker stop openclaw

# Start again
docker start openclaw

# View logs
docker logs openclaw

# Follow live logs
docker logs -f openclaw

# Restart (picks up .env changes)
docker restart openclaw

# Remove container (keeps image and config)
docker rm openclaw
```

---

## Troubleshooting

### `Unable to find image` error
Pull the image explicitly:
```bash
docker pull ghcr.io/openclaw/openclaw:latest
```

### Dashboard shows no token in logs
```bash
docker logs openclaw 2>&1 | grep -i token
```
If empty, check your `.env` file is correctly mounted.

### `Permission denied` on volume mounts (Linux/SELinux)
```bash
chmod 755 ~/.openclaw ~/openclaw/workspace
```

### NVIDIA API errors (401 Unauthorized)
- Verify key starts with `nvapi-`
- No quotes around the value in `.env`: `NVIDIA_API_KEY=nvapi-xxx`
- Confirm key is active at https://build.nvidia.com

### Port 18789 already in use
```bash
lsof -i :18789

# Or run on a different port
docker run -d --name openclaw -p 18790:18789 ...
# Access at http://localhost:18790
```

### Docker Desktop out of memory (Mac/Windows)
Open Docker Desktop → Settings → Resources → increase Memory to at least 4 GB.

---

## Agent Templates

An OpenClaw agent is defined by three Markdown files in its workspace folder:

| File | Purpose |
|------|---------|
| `SOUL.md` | Personality, worldview, opinions, and values |
| `STYLE.md` | Voice, tone, formatting, and writing patterns |
| `SKILL.md` | Operating modes triggered by different request types |

This repo includes **3 ready-to-use agents** in the `agents/` folder:

### Included Agents

**`agents/assistant/`** — General-purpose helper
- Direct, concise answers
- Matches user tone
- Adapts between chat, task, summary, and comparison modes

**`agents/researcher/`** — Deep research with citations
- Cross-references multiple sources
- Flags conflicting evidence and confidence levels
- Modes: quick lookup, deep report, fact-check, trend analysis

**`agents/coder/`** — Senior software engineer
- Writes complete, runnable code
- Debugs, reviews, optimizes, and generates tests
- Security-conscious, no shortcuts on error handling

### How to Use These Agents

```bash
# 1. Copy an agent workspace to your OpenClaw config directory
cp -r agents/researcher ~/.openclaw/agents/

# 2. Add it to your openclaw.json (see config/openclaw.json)
# 3. Start it
openclaw start researcher

# 4. Chat with it
openclaw tui
```

### NVIDIA NIM Config

The `config/openclaw.json` file in this repo is pre-configured for **NVIDIA NIM**. Copy it to get started:

```bash
cp config/openclaw.json ~/.openclaw/openclaw.json
```

Make sure your `~/.openclaw/.env` has:
```
NVIDIA_API_KEY=nvapi-your-key-here
```

### Create Your Own Agent

```bash
mkdir -p ~/.openclaw/agents/my-agent
cd ~/.openclaw/agents/my-agent

# Create the three required files
touch SOUL.md STYLE.md SKILL.md
```

**SOUL.md template:**
```markdown
## Identity
Who you are and what you do in 2-3 sentences.

## Worldview
Specific, falsifiable beliefs about your domain.

## Opinions
- On [Topic]: Your position and why.

## Vocabulary
- Terms you use / avoid

## Boundaries
- What you won't do

## Pet Peeves
- What triggers a pushback
```

Browse 187 community agent templates: https://github.com/mergisi/awesome-openclaw-agents

---

## Links

- OpenClaw GitHub: https://github.com/openclaw/openclaw
- OpenClaw Docs: https://docs.openclaw.ai
- NVIDIA NIM on OpenClaw: https://docs.openclaw.ai/providers/nvidia
- NVIDIA NIM Models: https://build.nvidia.com/models
- Docker Desktop: https://www.docker.com/products/docker-desktop/
