# Complete-AI-Coding-Stack-Biforst-Claude-NVIDIA-VS-Code

![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Mac-blue?style=for-the-badge)
![Model](https://img.shields.io/badge/Model-MiniMax_M3-purple?style=for-the-badge)
![Provider](https://img.shields.io/badge/Provider-NVIDIA_NIM-76B900?style=for-the-badge)
![Cost](https://img.shields.io/badge/Cost-$0-brightgreen?style=for-the-badge)

> **One Bifrost Gateway. NVIDIA NIM free tier. MiniMax M3 (428B). Claude Code in CLI. Continue in VS Code. Zero cost.**

---

## 🎓 Complete AI Gateway Masterclass (Free)

✅ **Part 1:** Claude Desktop Without Anthropic (AI Gateway Basics)

✅ **Part 2:** Unlimited Claude + Gemini + OpenCode with Bifrost

✅ **Part 3:** Complete $0 AI Coding Stack (Claude + NVIDIA + VS Code)

> Watch the series in order to build the complete setup from scratch.

[▶ Part 1](https://www.youtube.com/watch?v=4DlNb2weD_s) •
[▶ Part 2](https://www.youtube.com/watch?v=LOrjVQnA4EY) •
[▶ Part 3](https://www.youtube.com/watch?v=LzPvQAkQ8tw)

---

## ⚡ What Is This?

Connect **Claude Code** (CLI agent) and **Continue** (VS Code extension) to a single **Bifrost AI Gateway** backed by **NVIDIA NIM's free tier** running **MiniMax M3** — a 428B parameter multimodal model with 1M context window.

| Tool | Role |
|------|------|
| 🌉 **Bifrost Gateway** | Open-source AI gateway — routing, logging, observability |
| 🟠 **Claude Code** | Anthropic's CLI coding agent |
| 🧩 **Continue** | VS Code AI coding extension |
| 🟢 **NVIDIA NIM** | Free inference API — MiniMax M3 and 100+ models |
| 🧠 **MiniMax M3** | 428B MoE model, 1M context, multimodal, free on NIM |

---

## 🎯 How It Works

```
Claude Code (CLI)  ──→┐
                       ├──→ Bifrost Gateway ──→ NVIDIA NIM ──→ MiniMax M3
Continue (VS Code) ──→┘         ↓
                           LLM Logs, Routing,
                           Observability Dashboard
```

Both agents route through one gateway. All traffic is logged. Zero API cost on NVIDIA NIM free tier.

---

## 🖥️ Requirements

| Requirement | Detail |
|------------|--------|
| OS | Windows / Mac |
| Node.js | v18+ |
| GPU | ❌ Not required |
| NVIDIA NIM Key | ✅ Free at build.nvidia.com |
| Anthropic Account | ❌ Not required |

---

## 📁 Workspace Structure

```
zero-cost-setup/
├── config.json          ← Bifrost gateway config (NVIDIA NIM provider)
├── config.db            ← Bifrost SQLite store (auto-generated)
└── .continue/
    └── config.json      ← Continue VS Code config
```

---

## 🚀 Setup

### Step 0 — Clean Slate (Skip if Fresh Machine)

Remove any existing Bifrost data before starting:

```powershell
# Stop Bifrost if running
taskkill /IM bifrost.exe /F 2>$null

# Delete all Bifrost data
Remove-Item -Recurse -Force "$env:APPDATA\bifrost" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\bifrost" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:USERPROFILE\.bifrost" -ErrorAction SilentlyContinue

# Verify clean
ls "$env:APPDATA\bifrost" 2>$null
ls "$env:LOCALAPPDATA\bifrost" 2>$null
ls "$env:USERPROFILE\.bifrost" 2>$null
# All should show "path not found"
```

---

### Step 1 — Create Workspace

```powershell
mkdir agent-gateway
cd agent-gateway
```

---

### Step 2 — Create Bifrost Gateway Config

NVIDIA NIM is not a built-in Bifrost provider. We add it as a custom OpenAI-compatible provider. Create `config.json` in your workspace:

```json
{
  "$schema": "https://www.getbifrost.ai/schema",
  "providers": {
    "nvidia": {
      "keys": [{
        "name": "nvidia",
        "value": "env.NVIDIA_API_KEY",
        "models": ["*"],
        "weight": 1.0
      }],
      "network_config": {
        "base_url": "https://integrate.api.nvidia.com",
        "default_request_timeout_in_seconds": 300,
        "max_retries": 3
      },
      "custom_provider_config": {
        "is_key_less": false,
        "base_provider_type": "openai",
        "allowed_requests": {
          "chat_completion": true,
          "chat_completion_stream": true,
          "list_models": true
        }
      }
    }
  }
}
```

> **Why custom config?** Bifrost has built-in support for Anthropic, OpenAI, Groq, and others — but not NVIDIA NIM. Since NIM uses OpenAI-compatible API format, we register it as a custom provider. This is the "hack" that makes it work.

---

### Step 3 — Get NVIDIA NIM API Key

1. Go to [build.nvidia.com](https://build.nvidia.com)
2. Sign in → click your profile → **API Keys**
3. Click **Generate API Key**
4. Copy the key — it starts with `nvapi-...`

> NVIDIA NIM free tier gives you generous token limits across 100+ models including MiniMax M3.

---

### Step 4 — Launch Bifrost Gateway

```powershell
# Set your NVIDIA NIM key
$env:NVIDIA_API_KEY = "nvapi-your-key-here"

# Launch Bifrost pointing to your workspace config
npx -y @maximhq/bifrost -app-dir .
```

Open `http://localhost:8080` — you should see:
- **Total Providers: 1** (nvidia — CUSTOM)
- **Total Models: 121+**

Click **Model Catalog → Models tab → filter "minimax"** to confirm:
- `nvidia / minimaxai/minimax-m2.7`
- `nvidia / minimaxai/minimax-m3`

---

### Step 5 — Test the Gateway

Open a new terminal (keep Bifrost running):

```powershell
$body = '{"model":"nvidia/minimaxai/minimax-m3","messages":[{"role":"user","content":"say hello"}],"max_tokens":50}'
$response = Invoke-RestMethod -Uri "http://localhost:8080/v1/chat/completions" -Method POST -ContentType "application/json" -Body $body
$response | ConvertTo-Json -Depth 5
```

Expected response:
```json
{
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Hi there! How can I help you today?"
    }
  }],
  "model": "minimaxai/minimax-m3"
}
```

Check Bifrost dashboard → **LLM Logs** → you should see the request logged with provider, model, tokens, cost.

---

### Step 6 — Connect Claude Code (CLI)

```powershell
# Install Claude Code if not installed
npm install -g @anthropic-ai/claude-code

# Set dummy key (Claude Code requires a key, Bifrost ignores it)
$env:ANTHROPIC_API_KEY = "dummy-key"
$env:ANTHROPIC_BASE_URL = "http://localhost:8080/v1"

# Launch Claude Code
claude
```

Inside Claude Code:
```
/model
```
Type `nvidia` to filter → select `nvidia/minimaxai/minimax-m3`

Test prompt:
```
Create a simple Python script that generates a random password with 16 characters.
```

Watch Bifrost LLM Logs update in real time.

---

### Step 7 — Connect Continue in VS Code

#### Install Continue Extension

Open VS Code → Extensions (Ctrl+Shift+X) → Search **"Continue"** → Install

#### Configure Continue

Open the Continue config file:

**Windows:** `C:\Users\<YourName>\.continue\config.json`

Or click the Continue gear icon → **Open config.json**

Replace contents with:

```json
{
  "models": [{
    "title": "Bifrost MiniMax M3",
    "provider": "openai",
    "model": "nvidia/minimaxai/minimax-m3",
    "apiBase": "http://localhost:8080/v1",
    "apiKey": "dummy-key"
  }]
}
```

Save the file.

#### Test Continue

1. Open the Continue panel (sidebar icon or Ctrl+L)
2. Select **"Bifrost MiniMax M3"** from the model dropdown
3. Send a test prompt:

```
What model are you? Confirm you are connected.
```

Check Bifrost dashboard → **LLM Logs** → new request appears from Continue.

---

## 🧪 Tested Results

| Client | Gateway | Provider | Model | Status |
|--------|---------|----------|-------|--------|
| curl test | Bifrost | NVIDIA NIM | MiniMax M3 | ✅ |
| Claude Code | Bifrost | NVIDIA NIM | MiniMax M3 | ✅ |
| Continue VS Code | Bifrost | NVIDIA NIM | MiniMax M3 | ✅ |

---

## 🧠 Why MiniMax M3?

| Feature | Detail |
|---------|--------|
| Parameters | 428B (Mixture of Experts) |
| Active Parameters | ~26B per token |
| Context Window | 1,000,000 tokens |
| Modalities | Text, Image, Video input |
| Tool Calling | ✅ Supported |
| Cost on NIM | Free tier |
| Architecture | MiniMax Sparse Attention (MSA) — 1/20th compute at 1M context |

Released May 31, 2026. Available free on NVIDIA NIM.

---

## 🔍 Config Reference

### Bifrost Gateway Config (`config.json`)

| Field | Value | Notes |
|-------|-------|-------|
| `base_url` | `https://integrate.api.nvidia.com` | No `/v1` — Bifrost appends it |
| `value` | `env.NVIDIA_API_KEY` | Reads from environment variable |
| `base_provider_type` | `openai` | NIM uses OpenAI-compatible format |
| `list_models` | `true` | Enables model discovery |

### Continue Config (`.continue/config.json`)

| Field | Value | Notes |
|-------|-------|-------|
| `provider` | `openai` | Use OpenAI-compatible format |
| `model` | `nvidia/minimaxai/minimax-m3` | `nvidia/` prefix routes to NIM |
| `apiBase` | `http://localhost:8080/v1` | Local Bifrost gateway |
| `apiKey` | `dummy-key` | Bifrost doesn't validate this |

### Claude Code Environment Variables

| Variable | Value | Notes |
|----------|-------|-------|
| `ANTHROPIC_API_KEY` | `dummy-key` | Required by Claude Code, ignored by Bifrost |
| `ANTHROPIC_BASE_URL` | `http://localhost:8080/v1` | Points to Bifrost |

---

## ⚠️ Known Issues

| Issue | Detail | Fix |
|-------|--------|-----|
| NVIDIA NIM not in Bifrost UI providers | NIM is not a built-in provider | Use `config.json` custom provider approach |
| `list_models` 404 on older Bifrost | Bug in v1.5.x UI provider setup | Use file-based config with `-app-dir` flag |
| Schema warning on startup | Missing `$schema` field | Add `"$schema": "https://www.getbifrost.ai/schema"` |
| PowerShell `curl` escaping | Built-in curl alias breaks JSON | Use `Invoke-RestMethod` instead |
| Claude Code `reasoning_effort` error | Some models reject this param | Use OpenRouter models for Claude Code |

---

## 🔧 Cleanup / Reset

```powershell
# Stop Bifrost
taskkill /IM bifrost.exe /F 2>$null

# Remove agents
npm uninstall -g @anthropic-ai/claude-code

# Delete all Bifrost data
Remove-Item -Recurse -Force "$env:APPDATA\bifrost" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\bifrost" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:USERPROFILE\.bifrost" -ErrorAction SilentlyContinue

# Clear environment variables
Remove-Item Env:NVIDIA_API_KEY -ErrorAction SilentlyContinue
Remove-Item Env:ANTHROPIC_API_KEY -ErrorAction SilentlyContinue
Remove-Item Env:ANTHROPIC_BASE_URL -ErrorAction SilentlyContinue
```

---

## 🔗 Resources

| Resource | Link |
|----------|------|
| Bifrost GitHub | [github.com/maximhq/bifrost](https://github.com/maximhq/bifrost) |
| Bifrost Docs | [docs.getbifrost.ai](https://docs.getbifrost.ai) |
| NVIDIA NIM | [build.nvidia.com](https://build.nvidia.com) |
| MiniMax M3 on NIM | [build.nvidia.com/minimaxai/minimax-m3](https://build.nvidia.com/minimaxai/minimax-m3) |
| MiniMax M3 on OpenRouter | [openrouter.ai/minimax/minimax-m3](https://openrouter.ai/minimax/minimax-m3) |
| Continue Docs | [continue.dev](https://continue.dev) |
| Claude Code Docs | [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code) |

---

## 🔗 Links

[![YouTube](https://img.shields.io/badge/YouTube-NetworkCoder-red?style=for-the-badge&logo=youtube)](https://www.youtube.com/@NetworkCoder)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=for-the-badge&logo=github)](https://github.com/network-tocoder)

---

<p align="center">
  <strong>428B Model. 1M Context. $0 Cost.</strong><br/>
  <em>One gateway. Two agents. Zero rate limits.</em>
</p>
