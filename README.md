# Skills

**HarmonyOS (鸿蒙) NEXT development knowledge, portable across every major AI coding tool.**

One source file — [`harmonyos-development/SKILL.md`](./harmonyos-development/SKILL.md) — is built into drop-in rules/instructions for Claude Code, Cursor, Copilot, Codex, Gemini CLI, Windsurf, Continue, Cline, and any generic chat AI.

**Scope:** ArkTS · ArkUI · Stage model (UIAbility / ExtensionAbility) · state decorators (@State / @Link / @Observed / @ObjectLink …) · HarmonyOS Kits (Ability / Network / Media / Location / ArkData / HiAI …) · HAP/HSP/HAR packaging · atomic services (元服务) · distributed features · 10-category samples catalog.

---

## Supported tools

| Tool | Install location | File |
|---|---|---|
| **Claude Code / Agent SDK** | `~/.claude/skills/harmonyos-development/` | `harmonyos-development/SKILL.md` |
| **Cursor** (modern) | `.cursor/rules/harmonyos.mdc` | `dist/cursor/harmonyos.mdc` |
| **Cursor** (legacy) | `.cursorrules` | `dist/cursor/.cursorrules` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | `dist/copilot/copilot-instructions.md` |
| **Windsurf / Codeium** | `.windsurfrules` | `dist/windsurf/.windsurfrules` |
| **Continue.dev** | `.continue/rules/harmonyos.md` | `dist/continue/harmonyos.md` |
| **Cline / Roo Code** | Custom Instructions field | `dist/cline/custom-instructions.md` |
| **AGENTS.md**: Codex CLI · [opencode](https://github.com/sst/opencode) · Amp · Aider · Jules | `AGENTS.md` (repo root) | `dist/agents-md/AGENTS.md` |
| **Gemini CLI** | `GEMINI.md` or `~/.gemini/GEMINI.md` | `dist/gemini-cli/GEMINI.md` |
| **ChatGPT · Gemini · DeepSeek · Qwen · Kimi · 文心一言** | paste into Custom Instructions / System Prompt | `dist/plain/harmonyos-knowledge.md` |
| **Ollama · any LLM API** | `--system` flag / system message | `dist/system-prompt/system.txt` |

All `dist/` files are generated from the single source. Edit `harmonyos-development/SKILL.md`, then run `./scripts/build-dist.sh` to regenerate.

---

## Quick install

Replace `<FILE>` in the URL below with one of the paths from the table above (e.g. `dist/cursor/harmonyos.mdc`):

```bash
RAW=https://raw.githubusercontent.com/DengShiyingA/skills/claude/liquid-glass-skills-guide-0xUpZ
curl -o <local-path> "$RAW/<FILE>"
```

### Common one-liners

```bash
# Claude Code
git clone https://github.com/DengShiyingA/skills.git && \
  mkdir -p ~/.claude/skills && \
  cp -r skills/harmonyos-development ~/.claude/skills/

# Cursor
mkdir -p .cursor/rules && curl -o .cursor/rules/harmonyos.mdc "$RAW/dist/cursor/harmonyos.mdc"

# GitHub Copilot
mkdir -p .github && curl -o .github/copilot-instructions.md "$RAW/dist/copilot/copilot-instructions.md"

# Windsurf
curl -o .windsurfrules "$RAW/dist/windsurf/.windsurfrules"

# AGENTS.md (Codex / opencode / Amp / Aider / Jules)
curl -o AGENTS.md "$RAW/dist/agents-md/AGENTS.md"

# Gemini CLI
curl -o GEMINI.md "$RAW/dist/gemini-cli/GEMINI.md"

# Ollama
ollama run qwen2.5-coder --system "$(curl -s $RAW/dist/system-prompt/system.txt)"
```

---

## How activation works

| Tool category | Trigger |
|---|---|
| **Claude Code** | `description` frontmatter matched against user turn — auto-loaded, zero manual invocation |
| **Project rule files** (Cursor, Copilot, Windsurf, Continue, AGENTS.md, GEMINI.md) | Auto-attached to every chat and edit inside the project |
| **Generic chat** (ChatGPT, Gemini, DeepSeek, Qwen, …) | One-time paste per conversation, or permanent via Custom Instructions |

**Sanity check:** ask the tool *"Explain `@ObjectLink` in ArkUI and when to use it instead of `@State`."*
If the answer mentions the `@Observed` class requirement, the array-item re-render rule, and the reassignment trick — the knowledge is active.

---

## Repository layout

```
skills/
├─ harmonyos-development/SKILL.md    ← source of truth (edit here)
├─ scripts/build-dist.sh             ← regenerates every dist/ file
├─ dist/                             ← generated drop-ins
│  ├─ claude-code/      cursor/       copilot/      continue/
│  ├─ windsurf/         cline/        agents-md/    gemini-cli/
│  ├─ plain/            system-prompt/
└─ README.md
```

---

## Updating

```bash
git pull
./scripts/build-dist.sh           # regenerate dist/
# re-copy whichever file your tool uses
```

---

## Authoring your own skill

Source is the Claude Code `SKILL.md` format — YAML frontmatter + Markdown body:

```markdown
---
name: my-skill
description: Trigger keywords so the AI knows when to apply this skill.
---

# My Skill
## When to apply
## Reference
```

Guidelines: **focused** (one domain), **dense** (no filler), **trigger-rich** (list likely user phrasings in `description`). Run `./scripts/build-dist.sh` to produce all tool formats.
