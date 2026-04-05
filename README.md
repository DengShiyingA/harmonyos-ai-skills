# Skills

Multi-tool knowledge pack for **HarmonyOS (鸿蒙) NEXT** native development. Source of truth is [`harmonyos-development/SKILL.md`](./harmonyos-development/SKILL.md) (Claude Code format); a build script produces ready-to-drop-in files for every major AI coding tool.

Covers HarmonyOS NEXT with **ArkTS**, **ArkUI**, Stage model abilities, state-management decorators, all major Kits (Ability / Network / Media / Location / ArkData / HiAI …), HAP/HSP/HAR packaging, atomic services, distributed features, and the official samples catalog taxonomy.

---

## Repository layout

```
skills/
├─ harmonyos-development/SKILL.md    # Source of truth (Claude Code format)
├─ scripts/build-dist.sh             # Regenerates dist/ from the source
└─ dist/                             # Generated — drop-in files per tool
   ├─ claude-code/                   # Native Claude Code skill
   ├─ cursor/                        # .cursorrules + .mdc rule
   ├─ copilot/                       # GitHub Copilot instructions
   ├─ continue/                      # Continue.dev rules
   ├─ windsurf/                      # Windsurf rules
   ├─ cline/                         # Cline / Roo Code instructions
   ├─ agents-md/                     # AGENTS.md standard (Codex, opencode, Amp, Aider…)
   ├─ gemini-cli/                    # Google Gemini CLI GEMINI.md
   ├─ plain/                         # Pure Markdown (ChatGPT/Gemini/DeepSeek/…)
   └─ system-prompt/                 # Universal system prompt
```

Regenerate `dist/` after editing the source:

```bash
./scripts/build-dist.sh
```

---

## Supported AI tools

### ✅ Native format (frontmatter-aware, auto-triggered)

| Tool | Install path | Source file |
|---|---|---|
| **Claude Code CLI** | `~/.claude/skills/harmonyos-development/` | `harmonyos-development/` |
| **Claude Agent SDK** | SDK skills config | `harmonyos-development/` |

### ✅ Rules-file format (auto-attached per project)

| Tool | Install path | Source file |
|---|---|---|
| **Cursor** (modern) | `.cursor/rules/harmonyos.mdc` | `dist/cursor/harmonyos.mdc` |
| **Cursor** (legacy) | `.cursorrules` (repo root) | `dist/cursor/.cursorrules` |
| **Windsurf / Codeium** | `.windsurfrules` (repo root) | `dist/windsurf/.windsurfrules` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | `dist/copilot/copilot-instructions.md` |
| **Continue.dev** | `.continue/rules/harmonyos.md` | `dist/continue/harmonyos.md` |
| **Cline / Roo Code** | Custom Instructions field | `dist/cline/custom-instructions.md` |
| **AGENTS.md standard** — Codex CLI, [opencode](https://github.com/sst/opencode), Amp, Aider, Jules | `AGENTS.md` (repo root) | `dist/agents-md/AGENTS.md` |
| **Google Gemini CLI** | `GEMINI.md` (repo root) or `~/.gemini/GEMINI.md` | `dist/gemini-cli/GEMINI.md` |

### ✅ Generic (paste as system prompt / custom instructions)

| Tool | How | Source file |
|---|---|---|
| **ChatGPT / GPT-4o** | Custom Instructions → paste | `dist/plain/harmonyos-knowledge.md` |
| **Google Gemini / AI Studio** | System Instructions → paste | `dist/plain/harmonyos-knowledge.md` |
| **DeepSeek / Qwen / 文心一言 / Kimi** | 系统提示 / 角色设定 | `dist/plain/harmonyos-knowledge.md` |
| **Ollama local models** | `ollama run model --system "$(cat dist/system-prompt/system.txt)"` | `dist/system-prompt/system.txt` |
| **Any LLM API** | Include as system message | `dist/system-prompt/system.txt` |

---

## Installation quick-start

### Claude Code (recommended)

```bash
git clone https://github.com/DengShiyingA/skills.git
mkdir -p ~/.claude/skills
cp -r skills/harmonyos-development ~/.claude/skills/
# Restart Claude Code
```

### Cursor

```bash
# Modern .mdc rule (per-project):
mkdir -p .cursor/rules
curl -o .cursor/rules/harmonyos.mdc \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/cursor/harmonyos.mdc
```

### GitHub Copilot

```bash
# Project-level, auto-applied:
mkdir -p .github
curl -o .github/copilot-instructions.md \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/copilot/copilot-instructions.md
```

### Windsurf

```bash
curl -o .windsurfrules \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/windsurf/.windsurfrules
```

### AGENTS.md (Codex CLI, opencode, Amp, Aider, Jules, …)

A single file at the repo root serves every tool that follows the emerging
[AGENTS.md standard](https://agents.md):

```bash
curl -o AGENTS.md \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/agents-md/AGENTS.md
```

For global (user-level) scope, each tool reads from its own path:

| Tool | Global path |
|---|---|
| OpenAI Codex CLI | `~/.codex/AGENTS.md` |
| sst/opencode | `~/.config/opencode/AGENTS.md` |
| Amp | `~/.config/amp/AGENTS.md` |

### Google Gemini CLI

```bash
# Project-level:
curl -o GEMINI.md \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/gemini-cli/GEMINI.md

# Global (user-level):
mkdir -p ~/.gemini
curl -o ~/.gemini/GEMINI.md \
  https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ/dist/gemini-cli/GEMINI.md
```

### ChatGPT / Gemini / DeepSeek / Qwen (manual)

1. Open `dist/plain/harmonyos-knowledge.md` and copy all contents
2. Paste into the tool's **Custom Instructions** / **System Prompt** / **角色设定** field
3. Start chatting about HarmonyOS — the model now has the knowledge

### Ollama / local models

```bash
ollama run qwen2.5-coder --system "$(cat dist/system-prompt/system.txt)"
```

---

## How activation works

### Claude Code
Reads `SKILL.md` frontmatter → auto-loads when user question matches `description`. **Zero manual invocation.**

### Cursor / Windsurf / Copilot / Continue
Project-scoped rules files. Any file edit or chat inside the project automatically gets the knowledge injected.

### ChatGPT / Gemini / generic
One-time paste per conversation (or permanent via Custom Instructions).

---

## Verifying it works

Ask the AI in any tool:

> "Explain `@ObjectLink` in ArkUI and when to use it instead of `@State`."

If the answer mentions **`@Observed` class requirement**, **array-of-items re-render behavior**, or **reassignment trick** — the knowledge is active.

---

## Updating

```bash
# Pull latest source
git pull

# Regenerate all dist/ formats
./scripts/build-dist.sh

# Re-copy to your tool's install path
cp -r harmonyos-development ~/.claude/skills/    # Claude Code
cp dist/cursor/harmonyos.mdc <project>/.cursor/rules/  # Cursor
# etc.
```

---

## Writing your own skill

Source format is Claude Code SKILL.md (YAML frontmatter + Markdown body):

```markdown
---
name: my-skill
description: Trigger keywords that help the AI recognize when to apply this skill.
---

# My Skill

## When to apply this skill
...

## Reference
...
```

Keep skills **focused** (one domain), **dense** (no filler), and **trigger-rich** (list likely user phrasings in `description`). Then run `./scripts/build-dist.sh` to generate all tool formats.
