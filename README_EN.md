<div align="center">

**English** | [简体中文](./README.md)

<img src="./assets/en/hero-banner.svg" alt="HarmonyOS AI Skill" width="100%"/>

# 🧠 HarmonyOS AI Skill

### The largest HarmonyOS knowledge pack for AI coding — make 11+ AI tools actually write ArkTS

*4,327 lines of battle-tested knowledge · 237 sections · 105+ code examples · covers HarmonyOS 6.1 (API 23) / 6.1.1 (API 24)*

[![License](https://img.shields.io/badge/License-MIT-yellow)](./LICENSE)
[![HarmonyOS](https://img.shields.io/badge/HarmonyOS-6.1%20%2F%206.1.1-black)](https://developer.huawei.com/consumer/cn/)
[![ArkTS](https://img.shields.io/badge/ArkTS-API%2022--24-blue)](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-get-started-V5)
[![Kits](https://img.shields.io/badge/Kits-60+-orange)](#whats-inside-the-knowledge)
[![AI Tools](https://img.shields.io/badge/AI_Tools-11+-purple)](#supported-ai-tools)
[![AGENTS.md](https://img.shields.io/badge/AGENTS.md-Standard-green)](https://agents.md)

[![Stars](https://img.shields.io/github/stars/DengShiyingA/harmonyos-ai-skill?style=social)](https://github.com/DengShiyingA/harmonyos-ai-skill/stargazers)
[![Last Commit](https://img.shields.io/github/last-commit/DengShiyingA/harmonyos-ai-skill)](https://github.com/DengShiyingA/harmonyos-ai-skill/commits)
[![Issues](https://img.shields.io/github/issues/DengShiyingA/harmonyos-ai-skill)](https://github.com/DengShiyingA/harmonyos-ai-skill/issues)

<br/>

**Ask Cursor to write HarmonyOS — it hands you React.**
**Ask Claude to edit `module.json5` — it writes `package.json`.**
**Ask Copilot about `@ObjectLink` — it says "that API doesn't exist."**

General-purpose LLMs have never systematically learned HarmonyOS — their training data barely contains ArkTS, the Stage model, or HarmonyOS Kits.
So I distilled the entire Huawei official documentation, best practices, and API reference into a **4,327-line knowledge pack you drop straight into the LLM's context**: ArkTS strict-mode syntax, all 60+ Kits, glassmorphism effects, AI super frame, app continuation, PersistenceV2 — it's all in here.

**One Markdown source → drop-in configs for 11+ AI tools.** Once installed, your AI writes HarmonyOS-correct code like an engineer who's actually read the docs — instead of turning `@State` into `useState`.

<br/>

[🚀 Install](#installation) · [📖 What's Inside](#whats-inside-the-knowledge) · [🛠️ Supported Tools](#supported-ai-tools) · [✅ Verify It Works](#verifying-it-works)

</div>

---

<div align="center">
<img src="./assets/en/before-after.svg" alt="Before vs After comparison" width="100%"/>
</div>

---

## ⚡ Quick start (Claude Code, 30 seconds)

Pick the command set for your OS — **copy-paste straight into your terminal**:

### 🍎 macOS

```bash
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git ~/src/harmonyos-ai-skill
mkdir -p ~/.claude/skills
ln -s ~/src/harmonyos-ai-skill/harmonyos-development ~/.claude/skills/harmonyos-development
# Restart Claude Code, then ask: "What skills are available?"
```

### 🐧 Linux

```bash
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git ~/src/harmonyos-ai-skill
mkdir -p ~/.claude/skills
ln -s ~/src/harmonyos-ai-skill/harmonyos-development ~/.claude/skills/harmonyos-development
# Restart Claude Code, then ask: "What skills are available?"
```

### 🪟 Windows (PowerShell 7+)

> ⚠️ **Must run in PowerShell, not CMD (Command Prompt)** — `New-Item` is a PowerShell cmdlet; CMD doesn't know it. Right-click Start → "Windows PowerShell (Admin)".

```powershell
# First enable "Developer Mode" (one-time): Settings → Privacy & security → For developers → toggle on
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git $HOME\src\harmonyos-ai-skill
New-Item -ItemType Directory -Force $HOME\.claude\skills | Out-Null
New-Item -ItemType SymbolicLink -Path $HOME\.claude\skills\harmonyos-development -Target $HOME\src\harmonyos-ai-skill\harmonyos-development
# Restart Claude Code, then ask: "What skills are available?"
```

> Don't want to enable Developer Mode? Replace the `New-Item -ItemType SymbolicLink ...` line with `Copy-Item -Recurse $HOME\src\harmonyos-ai-skill\harmonyos-development $HOME\.claude\skills\` (but you'll need to re-copy after upstream updates).

Using a different tool (Cursor / Copilot / ChatGPT...)? See [all install options below](#installation).

---

## Why you need this

| Question | Plain AI | With the skill installed |
|---|---|---|
| What do I write the UI in? | "Use React Native" | "Use ArkUI — `@Component struct` declarative components" |
| How do I manage state? | "useState / Redux" | "`@State` / `@ObjectLink`; for new code use V2 decorators (stable in API 23)" |
| Page navigation? | "react-router or Vue Router" | "`Navigation` + `NavPathStack.pushPath()` — Router is being phased out" |
| HTTP requests? | "axios / fetch" | "`@kit.NetworkKit`'s `http.createHttp()`, or the `@ohos/axios` third-party lib" |
| Camera permission? | A snippet of Android Manifest | `module.json5` config + 3-step `abilityAccessCtrl` flow with settings fallback |
| Background audio? | Vague hints about a service | "You MUST create AVSession + request `KEEP_BACKGROUND_RUNNING` long-task" |

The knowledge isn't in the AI's head — you have to feed it in. **That's what this repo does.**

Write the knowledge once — [`harmonyos-development/SKILL.md`](./harmonyos-development/SKILL.md) — and install it into every major AI coding tool via pre-built drop-in files.

<details>
<summary><b>🤔 What is a "skill"?</b> (click to expand)</summary>

A skill is a chunk of domain knowledge (in Markdown) that an AI coding tool loads as background context when you chat with it. Once installed, the AI "knows" the domain — it will give you HarmonyOS-correct answers instead of generic TypeScript / React advice. Different tools call them different things (skills, rules, instructions, system prompt), but they all work the same way: **extra text prepended to the model's context**.

</details>

**Requirements:** `git` and `curl` (or just copy-paste for web tools). No other dependencies.
**Freshness:** Tracks official release cadence. Currently covers HarmonyOS 6.1.1 (API 24) Beta (May 2026).

## What's inside the knowledge

<div align="center">
<img src="./assets/en/knowledge-map.svg" alt="Knowledge architecture" width="100%"/>
</div>

The skill teaches the AI everything needed to read, write, review, and debug HarmonyOS NEXT native apps (**4,327 lines of dense, actionable knowledge, 237 sections, 105+ code examples**):

- **Language & framework** — ArkTS strictness rules, naming conventions, 13 high-performance coding rules (const, TypedArrays, HashMap, lazy import, etc.), coding style guide
- **App architecture** — Stage model: UIAbility, ExtensionAbility, AbilityStage, WindowStage lifecycles; module.json5 / app.json5 configuration
- **ArkUI components** — component lifecycle (7 callbacks + execution order), layout containers with performance comparison, `@Reusable` component reuse, **Tabs navigation**, **Swiper carousel**, **WaterFlow**, **Grid**, **TextInput**, **AlertDialog/Toast**, 10 form components quick reference, AttributeModifier reusable styles
- **State management (V2 now stable)** — V1 decorators (`@State`/`@Prop`/`@Link`/`@Provide`-`@Consume`/`@Observed`+`@ObjectLink`/`@Watch`) + V2 decorators (`@ComponentV2`/`@Local`/`@Param`+`@Once`/`@Param`+`@Event`/`@ObservedV2`+`@Trace`/`@Monitor`) + **AppStorageV2** + **PersistenceV2** (auto-persisted) + StateStore global state, with observation depth rules, batch update tips, decorator selection priority
- **Navigation** — `Navigation` + `NavPathStack` full API, `Router` basic routing (deprecated, with migration notes), `App Linking` deep links
- **Animation** — `animateTo()`, `.animation()`, `keyframeAnimateTo()`, Curve enum, spring curves, `geometryTransition` shared element transitions, animation performance tips
- **List operations** — pull-down refresh (Refresh), load-more (onReachEnd), swipe-to-delete (swipeAction), drag reorder, ListItemGroup sticky headers, scroll-to-bottom, maintain scroll position
- **Performance** — `LazyForEach` + IDataSource, `@Reusable`, `cachedCount`, `onVisibleAreaChange`, cold start optimization (lazy import), memory optimization (LRUCache/Purgeable)
- **HarmonyOS Kits** — 60+ Kits across 7 categories with import keys + code examples
- **Kit detailed sections** — Camera Kit (incl. API 24 "Follow the Person" subject tracking), Audio Kit, AVPlayer/AVRecorder, Image Kit (decode/transform/encode), Scan Kit, Account Kit, Payment Kit, Push Kit, Map Kit, **Weather Service Kit**, Core Vision Kit (OCR/face/segmentation), Form Kit (service cards), AVSession Kit, Location Kit, Notification Kit, Share Kit
- **Data persistence** — relationalStore (SQLite CRUD + sendable), preferences (KV storage), fileIo (file R/W), DocumentViewPicker (file picker)
- **Networking** — HTTP requests, WebSocket, network connectivity monitoring, background upload/download (request.agent with resume)
- **Concurrency** — TaskPool vs Worker comparison, `@Concurrent` rules, `@Sendable` shared-heap mechanism
- **System capabilities** — permission request full flow (check→request→settings fallback), immersive window (expandSafeArea), dark mode (resource qualifiers/colorMode), keyboard adaptation (KeyboardAvoidMode), screen orientation, clipboard, custom fonts, desktop shortcuts, gesture conflict resolution (hitTestBehavior/priorityGesture), EventHub, startAbilityByType
- **Web** — ArkWeb component, JS↔ArkTS bridge, cookie management, request interception
- **Cross-device** — app continuation (onContinue/onCreate data migration), cross-module resource access (HAR/HSP)
- **Engineering quality** — security coding rules + network security config (HTTPS/cert pinning), code obfuscation (ArkGuard), arkxtest testing (JsUnit + UiTest), 18 common gotchas
- **Third-party libraries** — @ohos/axios (HTTP client), @ohos/pulltorefresh, @ohos/lottie (JSON animation), @ohos/imageknife (image caching), dayjs (date utils)
- **API 23 / 24 new features** — Navigation routing stack binding, Menu anchorPosition, UDMF/drag/crypto C APIs, relationalStore sendable enhancement, AI super frame, Camera Kit "Follow the Person" subject tracking, delayed preview, DevEco Studio API 24 support
- **Multi-device** — responsive breakpoints (xs/sm/md/lg/xl), GridRow/GridCol, foldable support
- **Packaging & tooling** — HAP/HSP/HAR, atomic services, DevEco Studio 6.1+ (hvigor), OHPM, ArkCompiler

---

## Supported AI tools

### 1. Native skill format (auto-invoked by description matching)

| Tool | Install path | How it activates |
|---|---|---|
| **Claude Code CLI** | `~/.claude/skills/harmonyos-development/` | Claude reads `SKILL.md` frontmatter `description` and auto-loads when your question mentions HarmonyOS / ArkTS / ArkUI / Stage model / etc. Zero manual invocation. |
| **Claude Agent SDK** | Put the `harmonyos-development/` folder anywhere, point the SDK at it via the `skills` parameter when constructing the agent | Same as Claude Code — description-based auto-loading. |

### 2. Project rules file (auto-attached to every session inside the project)

| Tool | Install path | Source file | Scope |
|---|---|---|---|
| **Cursor** (modern) | `.cursor/rules/harmonyos.mdc` | `dist/cursor/harmonyos.mdc` | Glob-matched on `*.ets`, `module.json5`, `oh-package.json5`, `build-profile.json5` |
| **Cursor** (legacy) | `.cursorrules` (repo root) | `dist/cursor/.cursorrules` | Always applied |
| **GitHub Copilot** | `.github/copilot-instructions.md` | `dist/copilot/copilot-instructions.md` | Always applied in this repo |
| **Windsurf / Codeium** | `.windsurfrules` (repo root) | `dist/windsurf/.windsurfrules` | Always applied |
| **Continue.dev** | `.continue/rules/harmonyos.md` | `dist/continue/harmonyos.md` | Always applied |
| **Cline / Roo Code** | Settings → Custom Instructions | `dist/cline/custom-instructions.md` | Per-workspace or global |
| **OpenAI Codex CLI · sst/opencode · Amp · Aider · Jules** | `AGENTS.md` (repo root) | `dist/agents-md/AGENTS.md` | Follows the shared [AGENTS.md](https://agents.md) standard |
| **Google Gemini CLI** | `GEMINI.md` (repo root) or `~/.gemini/GEMINI.md` (global) | `dist/gemini-cli/GEMINI.md` | Gemini CLI reads either path |

### 3. Generic — paste into any chat / API

| Tool | Where to paste | Source file |
|---|---|---|
| **ChatGPT / GPT-4 / GPT-5** | Settings → Personalization → Custom Instructions (or per-conversation system prompt) | `dist/plain/harmonyos-knowledge.md` |
| **Google Gemini / AI Studio** | System Instructions field | `dist/plain/harmonyos-knowledge.md` |
| **DeepSeek / Qwen / 文心一言 / Kimi / 智谱** | 系统提示 / 角色设定 field | `dist/plain/harmonyos-knowledge.md` |
| **Ollama local models** | `--system` flag | `dist/system-prompt/system.txt` |
| **Anthropic / OpenAI / any LLM API** | `system` message of your request body | `dist/system-prompt/system.txt` |

The two files differ only slightly: `plain/` is the raw Markdown; `system-prompt/` prepends a short role-framing sentence (*"You are an expert HarmonyOS NEXT developer…"*).

---

## Installation

All `curl` commands below use a shell variable `$RAW` — **run this once in every new terminal before the commands**:

```bash
export RAW=https://raw.githubusercontent.com/DengShiyingA/harmonyos-ai-skill/main
```

> **Windows PowerShell users:** use `$env:RAW = "..."` and replace `curl -o foo` with `Invoke-WebRequest -Uri "..." -OutFile foo`.
> **HOME path differences:** macOS/Linux uses `~`; Windows PowerShell uses `$HOME`; CMD uses `%USERPROFILE%`.

### Claude Code CLI

Pick **one** of the three options below:

```bash
# Option A — quick copy (simplest, static snapshot)
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git ~/src/harmonyos-ai-skill
mkdir -p ~/.claude/skills
cp -r ~/src/harmonyos-ai-skill/harmonyos-development ~/.claude/skills/

# Option B — symlink (recommended: auto-updates after upstream `git pull`)
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git ~/src/harmonyos-ai-skill
mkdir -p ~/.claude/skills
ln -s ~/src/harmonyos-ai-skill/harmonyos-development ~/.claude/skills/harmonyos-development

# Option C — project-local only (commit it so your whole team gets the skill)
cd <your-harmonyos-project-root>
mkdir -p .claude/skills/harmonyos-development
curl -o .claude/skills/harmonyos-development/SKILL.md "$RAW/harmonyos-development/SKILL.md"
```

After installing, **restart Claude Code**. To verify, ask it: *"What skills are available?"* — it should list `harmonyos-development`.

### Cursor

> **Run from:** your HarmonyOS project root (the one with `entry/` and `module.json5`)

```bash
# Recommended — modern glob-scoped .mdc rule
mkdir -p .cursor/rules
curl -o .cursor/rules/harmonyos.mdc "$RAW/dist/cursor/harmonyos.mdc"

# OR legacy single-file rules (if your Cursor version predates .mdc)
curl -o .cursorrules "$RAW/dist/cursor/.cursorrules"
```

The `.mdc` rule auto-activates only when you edit `.ets`, `module.json5`, etc., keeping context lean for non-HarmonyOS projects.

### GitHub Copilot

> **Run from:** your HarmonyOS project root

```bash
mkdir -p .github
curl -o .github/copilot-instructions.md "$RAW/dist/copilot/copilot-instructions.md"
```

Applies to Copilot Chat and inline suggestions whenever you're inside this repo. Commit it — your whole team benefits.

### Windsurf / Codeium

> **Run from:** your HarmonyOS project root

```bash
curl -o .windsurfrules "$RAW/dist/windsurf/.windsurfrules"
```

### Continue.dev

> **Run from:** your HarmonyOS project root

```bash
mkdir -p .continue/rules
curl -o .continue/rules/harmonyos.md "$RAW/dist/continue/harmonyos.md"
```

### Cline / Roo Code

1. Download the file: `curl -o harmonyos-instructions.md "$RAW/dist/cline/custom-instructions.md"`
2. In VS Code: open Cline / Roo settings → **Custom Instructions**
3. Paste the file contents into the workspace or global instructions field

### AGENTS.md standard (Codex CLI, opencode, Amp, Aider, Jules)

A single file at the repo root serves **every** tool that follows the [AGENTS.md standard](https://agents.md):

```bash
curl -o AGENTS.md "$RAW/dist/agents-md/AGENTS.md"
```

For user-level (global) scope, each tool reads a different path:

| Tool | Global path |
|---|---|
| OpenAI Codex CLI | `~/.codex/AGENTS.md` |
| sst/opencode | `~/.config/opencode/AGENTS.md` |
| Amp | `~/.config/amp/AGENTS.md` |
| Aider | uses `AGENTS.md` from current directory only |

Some tools layer multiple `AGENTS.md` files (nearest one wins / merged). Check each tool's docs.

### Google Gemini CLI

```bash
# Project-level (takes precedence):
curl -o GEMINI.md "$RAW/dist/gemini-cli/GEMINI.md"

# Global (applies to every Gemini CLI session):
mkdir -p ~/.gemini
curl -o ~/.gemini/GEMINI.md "$RAW/dist/gemini-cli/GEMINI.md"
```

### ChatGPT / Gemini web / DeepSeek / Qwen / Kimi / 文心一言

1. Open [`dist/plain/harmonyos-knowledge.md`](./dist/plain/harmonyos-knowledge.md) on GitHub
2. Click **Raw** → **Ctrl/Cmd + A** → **Ctrl/Cmd + C**
3. In your AI tool:
   - **ChatGPT:** Settings → Personalization → **Custom Instructions** → "How would you like ChatGPT to respond?" → paste
   - **Gemini web:** Start a new chat → enable **System Instructions** → paste
   - **DeepSeek / Qwen / 文心一言 / Kimi:** create a new "智能体" / "角色" / "Bot" → paste into system prompt
4. Start asking HarmonyOS questions — the model now has the knowledge loaded

### Ollama / local LLMs

```bash
# 1. Pull a capable model (qwen3-coder or qwen2.5-coder recommended for Chinese-heavy docs)
ollama pull qwen3-coder

# 2. One-liner launch with HarmonyOS system prompt baked in
ollama run qwen3-coder \
  --system "$(curl -s $RAW/dist/system-prompt/system.txt)"
```

Or bake it into a custom Modelfile (permanently saves this "HarmonyOS expert" model):

```bash
# 1. Download the system prompt
curl -o system.txt "$RAW/dist/system-prompt/system.txt"

# 2. Create a Modelfile
cat > Modelfile <<EOF
FROM qwen3-coder
SYSTEM """
$(cat system.txt)
"""
EOF

# 3. Register the custom model
ollama create harmonyos-coder -f Modelfile
ollama run harmonyos-coder
```

### Anthropic / OpenAI / any LLM API

```python
# First: pip install anthropic
import anthropic

# Assumes you've downloaded system.txt locally
# (curl -o system.txt "$RAW/dist/system-prompt/system.txt")
with open("system.txt") as f:
    system_prompt = f.read()

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env
response = client.messages.create(
    model="claude-opus-4-7",            # latest Opus; or claude-sonnet-4-6 / claude-haiku-4-5
    system=system_prompt,
    max_tokens=2048,
    messages=[{"role": "user", "content": "How do I make a service card in HarmonyOS?"}],
)
print(response.content[0].text)
```

---

## 🪟 Windows users (native PowerShell)

> Don't want to use WSL/Git Bash? Want to install directly in PowerShell? Here's the complete command translation — **every tool works on Windows**.
> Recommended: **PowerShell 7+** (`winget install Microsoft.PowerShell`).

<details>
<summary><b>📋 Click to expand full PowerShell install commands</b></summary>

**Prereq: run this once in every new PowerShell window**

```powershell
$env:RAW = "https://raw.githubusercontent.com/DengShiyingA/harmonyos-ai-skill/main"
```

### Claude Code CLI (Windows)

```powershell
# Option A — quick copy (simplest)
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git $HOME\src\harmonyos-ai-skill
New-Item -ItemType Directory -Force $HOME\.claude\skills | Out-Null
Copy-Item -Recurse $HOME\src\harmonyos-ai-skill\harmonyos-development $HOME\.claude\skills\

# Option B — symlink (recommended: needs admin or "Developer Mode" enabled)
git clone https://github.com/DengShiyingA/harmonyos-ai-skill.git $HOME\src\harmonyos-ai-skill
New-Item -ItemType Directory -Force $HOME\.claude\skills | Out-Null
New-Item -ItemType SymbolicLink -Path $HOME\.claude\skills\harmonyos-development -Target $HOME\src\harmonyos-ai-skill\harmonyos-development

# Option C — project-local only
Set-Location <your-harmonyos-project-root>
New-Item -ItemType Directory -Force .claude\skills\harmonyos-development | Out-Null
Invoke-WebRequest -Uri "$env:RAW/harmonyos-development/SKILL.md" -OutFile .claude\skills\harmonyos-development\SKILL.md
```

> **Enable Developer Mode (one-time):** Settings → Privacy & security → For developers → toggle "Developer Mode" on. After that, `New-Item -ItemType SymbolicLink` no longer needs admin.

### Cursor (Windows)

```powershell
# Recommended: modern .mdc rule
New-Item -ItemType Directory -Force .cursor\rules | Out-Null
Invoke-WebRequest -Uri "$env:RAW/dist/cursor/harmonyos.mdc" -OutFile .cursor\rules\harmonyos.mdc

# Or: legacy single-file rules
Invoke-WebRequest -Uri "$env:RAW/dist/cursor/.cursorrules" -OutFile .cursorrules
```

### GitHub Copilot (Windows)

```powershell
New-Item -ItemType Directory -Force .github | Out-Null
Invoke-WebRequest -Uri "$env:RAW/dist/copilot/copilot-instructions.md" -OutFile .github\copilot-instructions.md
```

### Windsurf / Codeium (Windows)

```powershell
Invoke-WebRequest -Uri "$env:RAW/dist/windsurf/.windsurfrules" -OutFile .windsurfrules
```

### Continue.dev (Windows)

```powershell
New-Item -ItemType Directory -Force .continue\rules | Out-Null
Invoke-WebRequest -Uri "$env:RAW/dist/continue/harmonyos.md" -OutFile .continue\rules\harmonyos.md
```

### AGENTS.md standard (Codex CLI / opencode / Amp / Aider) (Windows)

```powershell
Invoke-WebRequest -Uri "$env:RAW/dist/agents-md/AGENTS.md" -OutFile AGENTS.md
```

Global paths (PowerShell):
| Tool | Path |
|---|---|
| OpenAI Codex CLI | `$HOME\.codex\AGENTS.md` |
| sst/opencode | `$HOME\.config\opencode\AGENTS.md` |
| Amp | `$HOME\.config\amp\AGENTS.md` |

### Google Gemini CLI (Windows)

```powershell
# Project-level
Invoke-WebRequest -Uri "$env:RAW/dist/gemini-cli/GEMINI.md" -OutFile GEMINI.md

# Global
New-Item -ItemType Directory -Force $HOME\.gemini | Out-Null
Invoke-WebRequest -Uri "$env:RAW/dist/gemini-cli/GEMINI.md" -OutFile $HOME\.gemini\GEMINI.md
```

### Ollama local LLM (Windows)

```powershell
# 1. Pull the model
ollama pull qwen3-coder

# 2. Download the system prompt
Invoke-WebRequest -Uri "$env:RAW/dist/system-prompt/system.txt" -OutFile system.txt

# 3. One-liner launch
ollama run qwen3-coder --system (Get-Content system.txt -Raw)

# Or: bake into a custom Modelfile
@"
FROM qwen3-coder
SYSTEM ```"
$(Get-Content system.txt -Raw)
```"
"@ | Set-Content Modelfile
ollama create harmonyos-coder -f Modelfile
ollama run harmonyos-coder
```

### Anthropic SDK Python (cross-platform, identical)

Python code is identical on Windows and macOS/Linux. See the [Anthropic section above](#anthropic--openai--any-llm-api).

</details>

> **Common pitfall:** PowerShell error `'curl' is not recognized as a cmdlet`?
> In Windows PowerShell, `curl` is an alias for `Invoke-WebRequest` but **the arguments are NOT compatible**. Use the `Invoke-WebRequest` commands above instead of `curl`.

---

## How activation differs by tool

| Tool category | Trigger mechanism | Always on? |
|---|---|---|
| **Claude Code / Agent SDK** | LLM reads skill `description` and decides whether to load for this turn | No — on-demand, saves context |
| **Cursor `.mdc`** | Glob pattern matches current file | Scoped to `.ets` / HarmonyOS config files |
| **Cursor `.cursorrules`, `.windsurfrules`, Copilot instructions, AGENTS.md, GEMINI.md, Continue / Cline rules** | Always prepended to every turn inside the project | Yes |
| **ChatGPT / Gemini Custom Instructions** | Always prepended to every conversation for that account | Yes |
| **Per-conversation paste / API `system`** | Only the conversations where you paste | Per-call |

**Rule of thumb:** for HarmonyOS-only projects, use the "always-on" rule files. For mixed repos (e.g. you have both Android and HarmonyOS code), prefer Cursor's scoped `.mdc` or Claude Code's description-based loading.

---

## Verifying it works

Ask the AI:

> *"Explain `@ObjectLink` in ArkUI and when to use it instead of `@State`."*

The answer is **properly primed** if it mentions:

- ✅ You must mark the class with `@Observed` for `@ObjectLink` to work
- ✅ `@State` on an array of objects only reacts to array mutations (push/splice/reassign), not per-item property changes
- ✅ Wrap items with `@Observed` + use `@ObjectLink` in the row component
- ✅ Or reassign the whole object to trigger a rerender

If the answer is vague or generic TS / React-like ("use a state hook"), the knowledge is **not loaded**.

Other good probes:

- "What's the difference between FA model and Stage model?"
- "How do I declare and request runtime permissions in HarmonyOS?"
- "Which Kit do I use for HTTP requests?"
- "How do I build a service card (服务卡片)?"

---

## Repository layout

```
harmonyos-ai-skill/
├─ .gitignore
├─ LICENSE
├─ README_EN.md
├─ harmonyos-development/
│  └─ SKILL.md                          ← Source of truth. Edit only here.
├─ scripts/
│  └─ build-dist.sh                     ← Regenerates every dist/ file
├─ dist/                                ← Generated — do not edit by hand
│  ├─ claude-code/harmonyos-development/SKILL.md
│  ├─ cursor/harmonyos.mdc
│  ├─ cursor/.cursorrules
│  ├─ copilot/copilot-instructions.md
│  ├─ windsurf/.windsurfrules
│  ├─ continue/harmonyos.md
│  ├─ cline/custom-instructions.md
│  ├─ agents-md/AGENTS.md
│  ├─ gemini-cli/GEMINI.md
│  ├─ plain/harmonyos-knowledge.md
│  └─ system-prompt/system.txt
└─ README.md
```

**Single-source workflow:**

1. Edit `harmonyos-development/SKILL.md`
2. Run `./scripts/build-dist.sh`
3. Commit both the source and the regenerated `dist/`

---

## Updating to the latest version

```bash
cd /path/to/your/clone
git pull
./scripts/build-dist.sh
# then re-copy whichever file your tool reads
```

If you installed via `ln -s`, you only need `git pull` — the symlink picks up changes automatically.

---

## Authoring your own skill

The source format is Claude Code's `SKILL.md` — YAML frontmatter plus a Markdown body:

```markdown
---
name: my-skill-name
description: >
  First sentence: what domain this skill covers.
  Then an exhaustive list of trigger phrases the AI might match:
  keywords, API names, command names, user questions, synonyms.
---

# My Skill

## When to apply this skill
- Bullet list of concrete scenarios

## Reference
Dense, cite-able reference material: tables, code snippets, API signatures,
rules, gotchas. Avoid prose filler. Favour bullets and compact examples.
```

### Guidelines

- **Focused** — one domain per skill. Don't combine HarmonyOS + iOS + Android.
- **Dense** — cut every sentence that doesn't teach the AI something it can cite.
- **Trigger-rich** — list every plausible user phrasing in `description`, in English and Chinese if relevant. The LLM's matching is fuzzy but benefits from explicit keywords.
- **Actionable** — prefer concrete code/config snippets over abstract explanations.
- **Honest about gaps** — if a feature is deprecated, say so. If you don't have data, leave it out.

After editing the source file, run `./scripts/build-dist.sh` to regenerate every tool-specific drop-in under `dist/`.

---

## Troubleshooting

**The AI still gives generic TypeScript/React answers.**
- Confirm the file landed in the right path (see table in *Supported AI tools*).
- For Claude Code, run *"What skills are available?"* — if `harmonyos-development` isn't listed, restart Claude Code or check `~/.claude/skills/` directly.
- For project-rule tools (Cursor, Copilot, etc.), make sure you're editing files **inside the repo** where the rule file lives. Rules don't apply outside the repo.
- For paste-based tools (ChatGPT, DeepSeek, …), the system prompt is per-conversation; start a **new chat** after pasting.

**Rule file is too long for the tool's context limit.**
Unlikely — `SKILL.md` is ~4200 lines (~150 KB). All major AI tools (Claude/GPT-4/Gemini etc.) accept it. If you hit a limit (e.g. some local small models), trim sections from `dist/plain/harmonyos-knowledge.md` manually.

**`curl` fails with 404.**
The branch in the URL may have moved. Check `https://github.com/DengShiyingA/harmonyos-ai-skill/branches` and update `$RAW` accordingly.

**How do I update after the upstream repo changes?**
See the *Updating to the latest version* section above.

---

## License & contributing

Licensed under the **MIT License** — use it freely in personal and commercial projects.

Contributions welcome:
1. Fork the repo
2. Edit `harmonyos-development/SKILL.md` (the **only** file you should ever edit — `dist/` is generated)
3. Run `./scripts/build-dist.sh` to regenerate distribution files
4. Commit both the source and the regenerated `dist/`, then open a PR

Factual corrections, new gotchas, updated API names, and translations of the description field (for better trigger matching) are all welcome.
