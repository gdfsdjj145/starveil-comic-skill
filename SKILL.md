---
name: starveil-comic-studio
description: Skill-First comic & webtoon production desk (`SKILL.md` + zero-dep `starveil` CLI with automatic 1-click browser sign-in — no MCP config or restart needed; optional Streamable HTTP MCP at https://starveilai.com/api/mcp). Keeps your host agent as writer and local file manager while adding 0-credit photo OC vaults, continuous multi-page comics, drag-to-box repair, Live Studio 2.0 canvas, and 6-language translation.
---

# StarVeil AI — Skill-First Comic & Webtoon Studio (Zero MCP Setup Needed)

StarVeil AI is a **Skill-First plug-in comic production desk**, not a black-box story writer. Any agent with a shell (Claude Code, Codex, Cursor, Antigravity, OpenClaw) can read this file and start rendering comics immediately — **no `mcp.json` editing, no restart, and no manual API key copy-pasting**.

## Quick Start: Run Immediately (No MCP Config Required)

### Mode A — Default: `starveil` CLI (Works in Every Shell Agent Immediately)
1. **Check or auto-install the zero-dependency CLI (Node 20+, ~10s, zero npm packages)**:
   ```bash
   command -v starveil >/dev/null 2>&1 || [ -x "$HOME/.local/bin/starveil" ] || curl -fsSL https://starveilai.com/install.sh | bash
   ```
   *(If `starveil` is not yet in `PATH`, invoke `$HOME/.local/bin/starveil` directly.)*
2. **Zero-Friction Auto Browser Sign-In (Do NOT ask the user to hunt for API keys)**:
   - On the very first `starveil` command (start with `starveil balance`), if no credential is saved yet:
     - **Local desktop**: the CLI **automatically opens the user's browser** to a 1-click approval page (`https://starveilai.com/app/cli-auth?code=...`), waits for approval, saves a revocable `sv_live_...` key to `~/.config/starveil/config.json`, and completes the command.
     - **Remote / SSH / cloud sandbox (no local GUI browser)**: the CLI exits immediately with code `2` and prints JSON `{ "authRequired": true, "verificationUrl": "...", "deviceCode": "...", "resumeCommand": "starveil auth wait <deviceCode>" }`. Share `verificationUrl` with the user to click once, then run `starveil auth wait <deviceCode>` (or re-run your command once approved).
   - Headless CI / n8n / OpenClaw daemons can also set `STARVEIL_API_KEY="sv_live_..."` (created at `https://starveilai.com/app/agents`).

3. **Core CLI Commands (JSON stdout — ready for agent parsing)**:
   ```bash
   starveil balance                                                    # Check credits & free trial (Page 1 + 1st edit = 0 cr!)
   starveil oc list                                                    # List reusable saved OCs
   starveil oc create --name "Mira" --desc "..." --image ./photo.png   # Lock a photo OC at 0 credits (instant ready)
   starveil oc import ./chara-card.png --project ./starveil-project.json # Import SillyTavern chara_card_v2 PNG at 0 credits
   starveil page '<scene with "quoted dialogue">' --oc id1 --lang en --wait --inspect-bubbles --out ./p01.png
   starveil page '<Page 2 scene>' --oc id1 --continue <p1TaskId> --style-anchor <p1TaskId> --wait --inspect-bubbles --out ./p02.png
   starveil edit <taskId> 'Fix typo to "Hello!"' --region 0.12,0.08,0.34,0.16 --wait --out ./p01.png
   starveil studio --task <taskId>                                     # Auto-opens passwordless Live Studio 2.0 in browser & prints liveStudioUrl (--no-browser for URL only)
   starveil translate ./p01.png --to ja --wait --out ./p01-ja.png      # 6-language art-preserving translation (10 cr/page)
   starveil webtoon '<story>' --oc id1 --lang en --wait                # 9:16 vertical webtoon + hosted reader link
   ```

### Mode B — Optional Native MCP (If Already Mounted or for No-Shell Hosts)
- If your host **already has StarVeil MCP tools mounted** (`get_credit_balance`, `create_comic_page`, `open_starveil_studio`, etc.), call them directly instead of shelling out.
- Endpoint: `https://starveilai.com/api/mcp` (Streamable HTTP · OAuth 2.1 + `sv_live_` Bearer · Official Registry `com.starveilai/comic-webtoon-studio` · Glama Verified Healthy).

---

## Core Principle: Your Host Agent Stays the Writer & File Manager
- **Use the user's existing local materials first**: Read their local novel chapters, Obsidian vaults (`Characters/*.md` YAML frontmatter), Fountain `.fountain` comic scripts, SillyTavern `chara_card_v2` PNG character cards (`starveil oc import ./card.png`), product docs, git changelogs, or `cast/` image folders directly from disk. Never ask the user to copy-paste assets into a web form.
- **Respect the author's voice**: Never hijack plot beats or rewrite dialogue tone unless asked. Your job as director is faithful visual translation, clean panel pacing, legible balloons, and asset continuity.
- **Let StarVeil handle what LLMs cannot**: 0-credit persistent photo OC locking, anti-copy `previousTaskId` multi-page rendering, `0..1` drag-to-box speech-bubble/detail repair, 6-language art-preserving localization, and the DaVinci-style Live Studio 2.0 canvas (`open_starveil_studio` / `starveil studio`).

## Modular Reference Guides (Read on Demand)
- **Local Project & Asset Management (`starveil-project.json`, `cast/` sync, cross-agent resume)**:
  `references/local-project-workspace.md` (or `https://starveilai.com/skills/references/local-project-workspace.md`)
- **Faithful Script-to-Panel Direction (turn user prose/scripts into crisp comic pages without mutating canon)**:
  `references/faithful-script-to-panels.md` (or `https://starveilai.com/skills/references/faithful-script-to-panels.md`)
- **Visual QA & 3-Zone Bounding-Box Repair (`edit_comic_page` geometry & lettering checks)**:
  `references/visual-qa-and-box-repair.md` (or `https://starveilai.com/skills/references/visual-qa-and-box-repair.md`)

## Local Comic Workspace Convention (Optional, Zero Lock-In)
When working on a multi-page story, adaptation, or recurring series, organize the user's existing folder so any agent can resume across sessions:

```text
<project-root>/
├── bible.md                     # User's canon, art direction, tone & style notes (author-owned)
├── cast/                        # User's local OC photos / sheets (synced at 0 cr each)
├── starveil-project.json        # Maps local character names <-> StarVeil characterIds & default LUT
└── episodes/
    └── ep01/
        ├── script.md            # Human-editable page-by-page beats & exact quoted dialogue
        ├── pages/               # Saved lossless PNGs (p01.png, p02.png, p02-ja.png...)
        └── manifest.json        # Checkpoints: taskId, versionId, previousTaskId, readerUrl
```

## The 5-Step Plug-in Production Loop

### 1. Audit Local Assets, Cloud OCs & Free Trial
- Scan any local files/folders the user points to (`cast/`, `bible.md`, scripts, `starveil-project.json`, `episodes/*/manifest.json`).
- Run `starveil balance` & `starveil oc list` (or MCP `get_credit_balance` & `list_my_characters`). First CLI command auto-handles browser login if needed!
- **Zero-Friction First Comic Rule**: If `freeTrial.freeFirstPageAvailable: true` and credits `< 20`, skip paid AI OC generation (`20 cr`) — either lock a local photo via `starveil oc create --image ./photo.png` / `create_character(referenceImageBase64)` for **0 credits**, or render Page 1 directly so Page 1 (`0 cr`) and 1st edit (`0 cr`) stay free.
- **Zero-Duplicate Cast Sync**: Never recreate an OC that already exists in `starveil-project.json` or `starveil oc list`.

### 2. Draft Editable Panel Scripts (Faithful to User Canon)
- Break prose into 2–4 panels per 3:4 page following `references/faithful-script-to-panels.md`:
  - Preserve the user's exact dialogue verbatim inside `"double quotes"`.
  - Keep bubbles concise (target ≤ 14 English words or ≤ 22 CJK characters per balloon).
- For multi-page projects, write `episodes/epXX/script.md` and let the user review before rendering.

### 3. Render & Open Live Studio (`starveil page` / `create_comic_page` & `starveil studio` / `open_starveil_studio`)
- Render via `starveil page "<scene>" --oc id1 --lang <lang> --wait --inspect-bubbles --out ./episodes/ep01/pages/p01.png` (or MCP `create_comic_page`).
- Run `starveil studio --task <taskId>` (or MCP `open_starveil_studio({ taskId })`) and share the returned passwordless `liveStudioUrl` so the user can visually inspect, drag-to-box edit, or wipe-compare beside their terminal.

### 4. Deterministic Bubble QA, Box-Repair & Checkpoint Locally
- With `--inspect-bubbles` (or MCP `get_comic_page({ taskId, inspectBubbles: true })`, free & KV-cached per `versionId`), inspect `bubbleQa.detectedBubbles`, `ocrText`, and automatic `typoAlerts` with pixel-accurate `0..1` `editRegion` coordinates (+2.5% halo buffer).
- If `bubbleQa.typoAlerts` flags a misspelled bubble (or visual inspection spots a drifted hand/prop), **never reroll the whole page** — pass `editRegion` and the fix directly into `starveil edit <taskId> "<fix>" --region x,y,w,h --wait --out ./p01.png` (or MCP `edit_comic_page`).
- Record `{ pageNo, taskId, versionId, previousTaskId, characterIds, localPath, readerUrl }` in `episodes/epXX/manifest.json`.

### 5. Chain Sequels, Vertical Webtoons & 6-Language Exports
- **Page 2+ Continuity**: Always pass the previous approved page's `taskId` as `--continue <taskId>` (`previousTaskId`) and Page 1's `taskId` as `--style-anchor <p1TaskId>` to lock cast, outfits, lighting, and ink onto fresh panel layouts.
- **Vertical Webtoons (9:16)**: `starveil webtoon "<story>" --wait` (or MCP `create_webtoon_episode`) → share hosted `readerUrl` (`https://starveilai.com/w/...`).
- **6-Language Art-Preserving Localization (`10 cr/page`)**: `starveil translate <fileOrVersionId> --to en|zh-CN|zh-TW|ja|ko|es --wait --out ./p01-ja.png` (or MCP `translate_comic`).

## Built-in Quick Recipes
- `xhs-comic` — 2–3 page continuous 3:4 Xiaohongshu carousel + titles & tags
- `article-to-comic` — Distill any local doc, URL, or git release into an explainer comic
- `pet-or-selfie-comic` — 0-credit photo OC lock + instant Page 1
- `global-localize` — Batch translate approved pages across EN / ZH / JA / KO / ES

## Deterministic Multi-Page CLI Pipeline (`starveil-cli`)
For multi-page chapter runs or two-way sync between local folders and Live Studio 2.0:
- **Preflight & self-upgrade**: `starveil doctor [--project ./starveil-project.json]` checks Node 20+, endpoint reachability, auth status, credits & free trial, ready OCs, and local refs; `starveil upgrade` atomically self-updates from `https://starveilai.com/downloads/starveil.mjs`.
- **Self-healing chapter production**: `starveil produce ./chapter01.md --project ./starveil-project.json --resolution 1K --auto-qa --auto-fix --out ./pages [--resume] [--dry-run]` — parses Fountain (`**PAGE N**`) or Markdown (`## Page N` / `## 第1页` / YAML frontmatter), locks Page 1 as `styleAnchorTaskId`, chains `previousTaskId`, runs free bubble OCR QA (`qa-report.json`), auto-repairs misprinted speech balloons in-place via `edit_comic_page` (`--auto-fix`), and saves atomic checkpoints to `manifest.json`.
- **Two-way Live Studio sync**: `starveil sync [--manifest ./manifest.json] [--dry-run]` — pulls newer versions edited in the Live Studio 2.0 browser canvas (`v2/v3`), archives superseded PNGs into `pages/.history/`, and updates `manifest.json`.
