<div align="center">

# 🧠 Continue.dev Client Setup — All OS Guide

**Step-by-step guide to connect [Continue.dev](https://continue.dev) in VS Code to your local NVIDIA H100 AI stack — works on Linux, Windows, and macOS.**

![OS Support](https://img.shields.io/badge/OS-Linux%20%C2%B7%20Windows%20%C2%B7%20macOS-2EA44F?style=for-the-badge)
![GPU](https://img.shields.io/badge/GPU-NVIDIA%20H100%2080GB-76B900?style=for-the-badge)
![Throughput](https://img.shields.io/badge/Throughput-43.6%20tok%2Fs%20%C2%B7%2032B%20Q8-00C0DF?style=for-the-badge)
![Ollama API](https://img.shields.io/badge/Ollama%20API-10.77.1.58%3A11434-F97316?style=for-the-badge)
![vLLM API](https://img.shields.io/badge/vLLM%20API-10.77.1.58%3A8000-8A2BE2?style=for-the-badge)

</div>

---

## 📌 Server Overview

|  |  |
|:---|:---|
| 🖥️ **AI Server** | `10.77.1.58` (`GPU01-58-GET-LocalAIModel`) — NVIDIA H100 80GB |
| 🐳 **Ollama API** | `http://10.77.1.58:11434` |
| ⚡ **vLLM API** | `http://10.77.1.58:8000` |
| 📈 **Benchmark** | **43.6 tok/s** generation (32B Q8) · **6,943 tok/s** prompt processing |

```text
Your machine (any OS)                     10.77.1.58 · GPU01-58-GET-LocalAIModel
┌─────────────────────────────┐    LAN    ┌──────────────────────────────────┐
│ VS Code + Continue.dev      │ ────────► │ NVIDIA H100 80GB                 │
│ config.yaml → AI server     │  :11434   │ Ollama :11434   vLLM :8000       │
└─────────────────────────────┘           └──────────────────────────────────┘
```

> [!NOTE]
> 🛡️ **100% on-prem** — no API keys, no cloud billing. Your code never leaves the LAN.

---

## 🗺️ Table of Contents

**Setup (do in order):**

0. [Step 0 — Verify Server Reachability](#step-0-verify-server-reachability)
1. [Step 1 — Install VS Code](#step-1-install-vs-code)
2. [Step 2 — Install the Continue Extension](#step-2-install-the-continue-extension)
3. [Step 3 — Deploy the Config File](#step-3-deploy-the-config-file)
   - [The Config File (paste-ready)](#the-config-file)
4. [Step 4 — Reload VS Code](#step-4-reload-vs-code)
5. [Step 5 — Verify the Connection](#step-5-verify-the-connection)

**Reference:**

- ⌨️ [Keyboard Shortcuts](#keyboard-shortcuts)
- 📎 [Context Providers `@`](#context-providers)
- 💬 [Slash Commands `/`](#slash-commands)
- 🔧 [Troubleshooting by OS](#troubleshooting-by-os)
- 💡 [Pro Tips for Vibe Coding](#pro-tips-for-vibe-coding)
- 📊 [Expected Performance](#expected-performance)

---

## 🎯 Step 0: Verify Server Reachability

Before anything else, confirm your client can actually reach the AI server.

### 🐧 Linux / macOS — Terminal

```bash
curl http://10.77.1.58:11434/api/tags
# Expected: {"models":[{"name":"coder-7b", ...}]}
```

### 🪟 Windows — PowerShell

```powershell
Invoke-RestMethod -Uri "http://10.77.1.58:11434/api/tags"
# Expected: models listed
```

### 🖥️ Windows — CMD

```cmd
curl http://10.77.1.58:11434/api/tags
```

> [!IMPORTANT]
> 🔒 If this fails, the server firewall only allows the **`10.77.1.0/24`** subnet.
>
> - Confirm your client VM is on the same LAN subnet (**`10.77.1.x`**).
> - If not, ask your admin to allow your IP:
>
>   ```bash
>   sudo ufw allow from YOUR_IP to any port 11434
>   ```

---

## 💻 Step 1: Install VS Code

Download from: **https://code.visualstudio.com/download**

| OS | Recommended Package |
|:---|:---|
| 🐧 Ubuntu / Debian | `.deb` package |
| 🐧 RHEL / Fedora | `.rpm` package |
| 🐧 Other Linux | `.tar.gz` — or Snap: `sudo snap install code --classic` |
| 🪟 Windows | **User Installer** (64-bit) |
| 🍎 macOS | **Universal** (Apple Silicon + Intel) `.dmg` |

> [!TIP]
> While you're in VS Code, run `Shell Command: Install 'code' command in PATH` from the Command Palette — Step 2 uses the `code` CLI.

---

## 🧩 Step 2: Install the Continue Extension

### Method A — VS Code Marketplace (Recommended) ✨

1. Open VS Code
2. Press `Ctrl+Shift+X` (Linux/Windows) or `Cmd+Shift+X` (macOS)
3. Search: `Continue`
4. Click **Install** on **“Continue - Codestral, Claude, and more”** by `Continue`

### Method B — Command Line

```bash
# Linux / macOS
code --install-extension Continue.continue

# Windows PowerShell
code --install-extension Continue.continue
```

### Method C — VSIX Offline Install (Air-Gapped Networks)

```bash
# Download on any machine with internet:
# https://open-vsx.org/extension/Continue/continue
# Then install:
code --install-extension continue-*.vsix
```

---

## 📁 Step 3: Deploy the Config File

This is the **most important step**. The config tells Continue where the AI server lives.

> [!TIP]
> Prefer not to touch the terminal? **Method C (Windows section below)** shows how to open `config.yaml` directly from the Continue panel — works from VS Code on any OS.

### 🐧 Linux Client

```bash
# Create the directory
mkdir -p ~/.continue

# Option A: Copy directly from server via SSH
scp get@10.77.1.58:~/ai-stack/continue/config.yaml ~/.continue/config.yaml

# Option B: Create manually and paste the config from
#           "The Config File" section below
nano ~/.continue/config.yaml

# Verify it looks right
head -10 ~/.continue/config.yaml
```

### 🪟 Windows Client

#### Method A — PowerShell + SCP

```powershell
# Create the directory
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.continue"

# Copy from server via SCP (requires OpenSSH or PuTTY's pscp)
scp get@10.77.1.58:~/ai-stack/continue/config.yaml "$env:USERPROFILE\.continue\config.yaml"

# Verify
Get-Content "$env:USERPROFILE\.continue\config.yaml" | Select-Object -First 10
```

#### Method B — PowerShell (Manual Create)

```powershell
# Create directory
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.continue"

# Open in Notepad to paste config
notepad "$env:USERPROFILE\.continue\config.yaml"
```

#### Method C — From VS Code Itself 🚀

1. Open VS Code → Click the **Continue icon** in the left sidebar
2. Click the **gear icon ⚙️** at the bottom of the Continue panel
3. Select **“Open config.yaml”**
4. Replace ALL contents with the config below
5. Save (`Ctrl+S`)

### 🍎 macOS Client

```bash
# Create the directory
mkdir -p ~/.continue

# Option A: Copy from server via SSH
scp get@10.77.1.58:~/ai-stack/continue/config.yaml ~/.continue/config.yaml

# Option B: Open and paste manually
open -e ~/.continue/config.yaml
# (opens in TextEdit — or use: code ~/.continue/config.yaml)

# Verify
head -10 ~/.continue/config.yaml
```

### 📍 Config File Locations — Quick Reference

| OS | Path |
|:---|:---|
| 🐧 Linux | `~/.continue/config.yaml` |
| 🪟 Windows | `C:\Users\<YourUsername>\.continue\config.yaml` |
| 🍎 macOS | `~/.continue/config.yaml` |

---

## 📋 The Config File

This is the heart of the setup. Paste it **exactly** into `config.yaml` on any client — no changes needed if your server is `10.77.1.58`.

```yaml
# ============================================================================
# Continue.dev v2 Configuration — H100 PCIe 80GB Local AI Stack
# Server: 10.77.1.58 (GPU01-58-GET-LocalAIModel)
# ============================================================================
name: H100-LocalAI-VibeCoding
version: 1.0.0
schema: v1

models:

  # PRIMARY CHAT / EDIT / APPLY: 32B Q8 — zero quality loss, 43 tok/s
  - name: "Qwen2.5-Coder 32B ★ Chat"
    provider: ollama
    model: coder-32b
    apiBase: http://10.77.1.58:11434
    contextLength: 65536
    completionOptions:
      temperature: 0.05
      topP: 0.85
      topK: 15
      maxTokens: 16384
    roles:
      - chat
      - edit
      - apply

  # HEAVY TASKS: 72B Q4 — complex refactoring, architecture, long context
  - name: "Qwen2.5-Coder 72B ⚡ Heavy"
    provider: ollama
    model: coder-72b
    apiBase: http://10.77.1.58:11434
    contextLength: 65536
    completionOptions:
      temperature: 0.05
      topP: 0.85
      topK: 15
      maxTokens: 16384
    roles:
      - chat
      - edit
      - apply

  # AUTOCOMPLETE: 7B Q8 — always hot in VRAM, sub-100ms FIM
  - name: "Qwen2.5-Coder 7B ⚡ Autocomplete"
    provider: ollama
    model: coder-7b
    apiBase: http://10.77.1.58:11434
    contextLength: 8192
    completionOptions:
      temperature: 0.0
      topP: 0.9
      topK: 10
      maxTokens: 256
    roles:
      - autocomplete

  # EMBEDDINGS: codebase RAG / semantic search
  - name: "Nomic Embed Text"
    provider: ollama
    model: nomic-embed-text
    apiBase: http://10.77.1.58:11434
    roles:
      - embed

tabAutocompleteModel:
  name: "Qwen2.5-Coder 7B ⚡ Autocomplete"
  provider: ollama
  model: coder-7b
  apiBase: http://10.77.1.58:11434

tabAutocompleteOptions:
  debounceDelay: 300
  maxPromptTokens: 1024
  prefixPercentage: 0.85
  disableInFiles:
    - "*.md"
    - "*.txt"
    - "*.json"
    - "*.yaml"
    - "*.yml"
    - "*.lock"
    - "*.log"
  useCopyBuffer: false
  useFileSuffix: true
  multilineCompletions: "auto"

context:
  providers:
    - name: codebase
      params:
        nRetrieve: 25
        nFinal: 10
        useReranking: false
    - name: file
    - name: folder
    - name: currentFile
    - name: open
    - name: code
    - name: diff
    - name: terminal
    - name: problems
    - name: url
    - name: docs
    - name: search
    - name: clipboard

slashCommands:
  - name: edit
    description: "Edit highlighted code with AI"
  - name: comment
    description: "Add docstrings and comments to code"
  - name: share
    description: "Copy current conversation"
  - name: cmd
    description: "Generate a shell command"
  - name: test
    description: "Write unit tests for selected code"
  - name: fix
    description: "Fix bugs and errors in selected code"
  - name: refactor
    description: "Refactor and improve code quality"
  - name: explain
    description: "Explain how the code works"
  - name: optimize
    description: "Optimize code for performance"
  - name: security
    description: "Audit code for security vulnerabilities"

docs:
  - name: "Python Docs"
    startUrl: "https://docs.python.org/3/"
  - name: "TypeScript Docs"
    startUrl: "https://www.typescriptlang.org/docs/"
  - name: "React Docs"
    startUrl: "https://react.dev/reference/react"
  - name: "Node.js Docs"
    startUrl: "https://nodejs.org/docs/latest/api/"
  - name: "Go Docs"
    startUrl: "https://pkg.go.dev/std"
  - name: "Rust Docs"
    startUrl: "https://doc.rust-lang.org/std/"

ui:
  displayRawMarkdown: false
  codeBlockToolbarPosition: top
```

> [!IMPORTANT]
> 📍 Keep the `apiBase` values pointing at `http://10.77.1.58:11434` on every client — only change them if the server IP changes.

---

## 🔄 Step 4: Reload VS Code

After saving the config:

1. Open the Command Palette — `Ctrl+Shift+P` (Linux/Windows) · `Cmd+Shift+P` (macOS)
2. Type: **`Developer: Reload Window`**
3. Press `Enter`

---

## ✅ Step 5: Verify the Connection

1. Press **`Ctrl+L`** (`Cmd+L` on macOS) → Continue sidebar opens
2. Select **“Qwen2.5-Coder 32B ★ Chat”** from the model dropdown at the top
3. Type: `hello, what model are you?`
4. Expected: response in **~2 seconds** mentioning Qwen2.5-Coder

> [!TIP]
> **Tab-autocomplete check:** open any `.py`, `.ts`, `.go`, or `.rs` file, start typing a function, and wait **300 ms** — a grey suggestion appears. Press **`Tab`** to accept, `Esc` to dismiss.

---

## ⌨️ Keyboard Shortcuts

| Shortcut (Linux/Win) | Shortcut (macOS) | Action |
|:---|:---|:---|
| `Ctrl+L` | `Cmd+L` | Open/focus Continue chat |
| `Ctrl+I` | `Cmd+I` | Inline edit (select code first) |
| `Tab` | `Tab` | Accept autocomplete suggestion |
| `Esc` | `Esc` | Dismiss autocomplete |
| `Ctrl+Shift+L` | `Cmd+Shift+L` | Add selected code to chat |
| `Ctrl+Shift+Enter` | `Cmd+Shift+Enter` | Submit chat message |
| `Alt+\` | `Option+\` | Trigger autocomplete manually |
| `Ctrl+Shift+R` | `Cmd+Shift+R` | Reject all edits |
| `Ctrl+Shift+A` | `Cmd+Shift+A` | Accept all edits |

---

## 📎 Context Providers (@)

Type **`@`** in the Continue chat box to open the context menu:

| Command | What it does |
|:---|:---|
| `@codebase` | Semantic RAG search across your entire project |
| `@file src/main.py` | Include a specific file |
| `@folder src/` | Include an entire folder |
| `@currentFile` | Include whatever file you have open |
| `@open` | Include all open editor tabs |
| `@code MyClass.myMethod` | Include a specific function/class by symbol |
| `@diff` | Include current git diff (staged + unstaged) |
| `@terminal` | Include last terminal output (great for errors) |
| `@problems` | Include VS Code diagnostics / red squiggles |
| `@url https://...` | Fetch and include any web page |
| `@docs Python Docs` | Search indexed documentation |
| `@search "pattern"` | Ripgrep text search across project |
| `@clipboard` | Include clipboard content |

---

## 💬 Slash Commands (/)

Type **`/`** in the Continue chat to trigger commands:

| Command | Use Case | Example |
|:---|:---|:---|
| `/edit` | Inline edit with instruction | Select function → `/edit add error handling` |
| `/fix` | Fix a bug or error | Paste error → `/fix` |
| `/test` | Generate tests | Select function → `/test` |
| `/comment` | Add docstrings/comments | Select code → `/comment` |
| `/refactor` | Clean up and improve code | Select class → `/refactor` |
| `/explain` | Step-by-step explanation | Select code → `/explain` |
| `/optimize` | Performance optimization | Select loop → `/optimize` |
| `/security` | Security audit | Select auth code → `/security` |
| `/cmd` | Generate shell command | `/cmd find all TODO comments` |

---

## 🔧 Troubleshooting by OS

### 🐧 Linux

```bash
# Config not found?
ls -la ~/.continue/config.yaml

# API unreachable?
ping 10.77.1.58
curl -v http://10.77.1.58:11434/api/tags

# VSCode can't find Continue extension?
code --list-extensions | grep -i continue

# Reinstall extension
code --uninstall-extension Continue.continue
code --install-extension Continue.continue

# Check Continue logs in VSCode:
# Help → Toggle Developer Tools → Console tab → filter "continue"
```

### 🪟 Windows

```powershell
# Config not found?
Get-Item "$env:USERPROFILE\.continue\config.yaml"

# API unreachable?
Test-NetConnection -ComputerName 10.77.1.58 -Port 11434
# or:
curl.exe http://10.77.1.58:11434/api/tags

# Common issue: Windows Firewall blocking outbound to port 11434
# Fix (run PowerShell as Administrator):
New-NetFirewallRule -DisplayName "Allow Ollama" -Direction Outbound `
  -Protocol TCP -RemotePort 11434 -Action Allow

# Check Continue logs:
# Help → Toggle Developer Tools → Console tab

# Config path wrong? Try both locations:
# C:\Users\<user>\.continue\config.yaml      ← standard
# %APPDATA%\Continue\config.yaml             ← alternative
```

### 🍎 macOS

```bash
# Config not found?
ls -la ~/.continue/config.yaml

# API unreachable?
ping 10.77.1.58
nc -zv 10.77.1.58 11434

# macOS firewall blocking?
# System Settings → Network → Firewall → allow VSCode outbound

# Check Continue logs:
# Help → Toggle Developer Tools → Console tab

# Permission issue?
chmod 644 ~/.continue/config.yaml

# Reinstall:
code --uninstall-extension Continue.continue && \
code --install-extension Continue.continue
```

---

## 💡 Pro Tips for Vibe Coding

### 1. Index Your Codebase for RAG (once per project)

```text
Ctrl+Shift+P → "Continue: Index Codebase"
```

After indexing, `@codebase` can semantically search your entire project. Essential for debugging across large codebases.

### 2. Switch to 72B for Hard Problems 🐘

In the Continue chat dropdown → switch to **“Qwen2.5-Coder 72B ⚡ Heavy”**.
First response takes ~15 s (model swap from RAM); subsequent responses are fast.

### 3. Best Workflow for Debugging 🐛

```text
1. Copy error from terminal
2. Ctrl+L (open Continue)
3. Type: @terminal @currentFile fix this error
4. Press Enter
```

### 4. Best Workflow for Adding Features ✨

```text
1. Ctrl+L → @codebase how is authentication currently implemented?
2. Review the answer
3. Select where to add code
4. Ctrl+I → "add OAuth2 support following the existing pattern"
```

### 5. Best Workflow for Code Review 👀

```text
1. Ctrl+L → @diff review these changes for bugs and security issues
```

### 6. Preload 32B Before Your Session (on the server) 🏁

```bash
# SSH to server, then:
ai-load coder-32b
```

This puts 32B into VRAM immediately so your first request is instant.

---

## 📊 Expected Performance

Measured from your **H100 80GB** server:

| Model | First Request | Subsequent | Tokens/sec |
|:---|:---|:---|:---:|
| `coder-7b` (autocomplete) | Instant (always hot) | <100 ms | ~180 tok/s |
| `coder-32b` (chat) | ~2 s (hot) / ~35 s (cold swap) | ~2–8 s | **~43 tok/s** |
| `coder-72b` (heavy) | ~15 s (swap from RAM) | ~5–15 s | ~18 tok/s |

> [!NOTE]
> ⏱️ A 500-token response from the 32B model takes **~11 seconds** — fast enough to feel real-time.

---

<div align="center">

**🏁 That's it — you're wired into the H100. Happy vibe coding!** 🚀

[⬆️ Back to top](#)

</div>
