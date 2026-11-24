---
name: starveil-comic-studio
description: Plug-in comic & webtoon production desk (SKILL + Streamable HTTP MCP at https://starveilai.com/api/mcp) for Claude Code, Antigravity, Cursor, Codex, and OpenClaw. Keeps your host agent as writer and local file manager while adding local comic-project state tracking, 0-credit photo OC vaults, faithful script-to-panel direction, continuous multi-page comics, drag-to-box repair, live Studio 2.0 canvas, and 6-language translation.
---

# StarVeil AI — Plug-in Comic & Webtoon Studio Skill (Bring Your Own Agent & Local Files)

StarVeil AI (`https://starveilai.com/api/mcp`) is a **plug-in comic production desk**, not a restrictive black-box story writer.

## Core Principle: Your Host Agent Stays the Writer & File Manager
- **Use the user's existing local materials first**: Read their local novel chapters, Obsidian vaults (`Characters/*.md` YAML frontmatter), Fountain `.fountain` comic scripts, SillyTavern `chara_card_v2` PNG character cards (`starveil oc import ./card.png`), product docs, git changelogs, or `cast/` image folders directly from disk. Never ask the user to copy-paste assets into a closed web form.
- **Respect the author's voice**: Never hijack plot beats or rewrite dialogue tone unless the user asks you to brainstorm. Your job as director is faithful visual translation, clean panel pacing, legible balloons, and asset continuity.
- **Let StarVeil MCP do what LLMs cannot**: 0-credit persistent photo OC locking, anti-copy `previousTaskId` multi-page rendering, `0..1` drag-to-box speech-bubble/detail repair, 6-language art-preserving localization, and the DaVinci-style Live Studio 2.0 canvas.

## Modular Reference Guides (Read on Demand)
- **Local Project & Asset Management (`starveil-project.json`, `cast/` sync, cross-agent resume)**:
  `references/local-project-workspace.md` (or `https://starveilai.com/skills/references/local-project-workspace.md`)
- **Faithful Script-to-Panel Direction (turn user prose/scripts into crisp comic pages without mutating canon)**:
  `references/faithful-script-to-panels.md` (or `https://starveilai.com/skills/references/faithful-script-to-panels.md`)
- **Visual QA & 3-Zone Bounding-Box Repair (`edit_comic_page` geometry & lettering checks)**:
  `references/visual-qa-and-box-repair.md` (or `https://starveilai.com/skills/references/visual-qa-and-box-repair.md`)

## Setup & Authentication
- **Streamable HTTP MCP**: `https://starveilai.com/api/mcp`
- **Official MCP Registry**: `com.starveilai/comic-webtoon-studio` & `com.starveilai/star-veil-ai` · **Glama**: Verified Healthy (`18` agent tools + Live MCP App)
- **Claude Code (OAuth 2.1)**: `claude mcp add --transport http starveil https://starveilai.com/api/mcp`
- **Universal Installer (Antigravity / Claude Code / Cursor / Codex / CLI)**: `curl -fsSL https://starveilai.com/install.sh | bash`
- **Headless / OpenClaw / n8n / CI**: Pass `Authorization: Bearer sv_live_...` (create at `https://starveilai.com/app/agents`).

## Local Comic Workspace Convention (Optional, Zero Lock-In)
When working on a multi-page story, adaptation, or recurring series, organize the user's existing folder (or initialize a lightweight workspace) so any agent can resume across sessions:

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

Read `references/local-project-workspace.md` for exact JSON schemas and sync rules.

## The 5-Step Plug-in Production Loop

### 1. Audit Local Assets, Cloud OCs & Trial Credits
- Scan any local files/folders the user points to (`cast/`, `bible.md`, scripts, `starveil-project.json`, `episodes/*/manifest.json`).
- Call `get_credit_balance` and `list_my_characters` first.
- **Zero-Friction First Comic Rule**: If `freeTrial.freeFirstPageAvailable: true` and credits `< 20`, skip paid AI OC generation (`20 cr`) — either lock a local photo via `create_character(referenceImageBase64)` for **0 credits**, or proceed directly to `create_comic_page` so Page 1 (`0 cr`) and first edit (`0 cr`) stay free.
- **Zero-Duplicate Cast Sync**: Never recreate an OC that already exists in `starveil-project.json` or `list_my_characters`. When syncing local photos from `cast/`, upload at **0 credits** and record `{ localName, filePath, characterId }` back into `starveil-project.json`.

### 2. Draft Editable Panel Scripts (Faithful to User Canon)
- If adapting existing prose, a novel chapter, a blog post, or a repo, break it into 2–4 panels per 3:4 page following `references/faithful-script-to-panels.md`:
  - Preserve the user's exact dialogue verbatim inside `"double quotes"`.
  - Keep bubbles concise (target ≤ 14 English words or ≤ 22 CJK characters per balloon) — if a user line is long, propose splitting across two panels rather than silently rewriting their voice.
- For multi-page projects, write `episodes/epXX/script.md` and optionally call `open_starveil_studio({ storyboardDraft, highlightNote })` so the user can review and edit lines before spending credits.

### 3. Render & Sync Live Studio (`create_comic_page`, `open_starveil_studio`)
- Call `create_comic_page({ clientRequestId, scene, characterIds, outputLanguage, previousTaskId? })`.
- **Immediately** call `open_starveil_studio({ taskId, highlightNote })`:
  - In Antigravity / MCP Apps hosts: opens the persistent DaVinci-style side pane live.
  - In CLI agents (Claude Code / Cursor / Codex / OpenClaw): share the returned passwordless `liveStudioUrl` so the user can inspect beside their terminal.

### 4. Deterministic Bubble QA, Box-Repair & Checkpoint Locally (`get_comic_page`, `edit_comic_page`)
- Poll `get_comic_page({ taskId })` until `ready: true` (leave `inspectBubbles: false` while polling).
- Once `ready: true`, call **`get_comic_page({ taskId, inspectBubbles: true })`** (free, KV-cached per `versionId`) to get machine-detected `bubbleQa.detectedBubbles`, literal `ocrText`, automatic `typoAlerts`, and pixel-accurate `0..1` `editRegion` coordinates (+2.5% halo buffer) — plus download `latestDownloadUrl` (valid 10 min) into `episodes/epXX/pages/pXX.png` to visually check artwork & anatomy against `references/visual-qa-and-box-repair.md`.
- If `bubbleQa.typoAlerts` flags a misspelled bubble (or visual QA spots a drifted hand/prop), **never reroll the whole page** — pass `alert.editRegion` and `alert.suggestedInstruction` directly into `edit_comic_page({ taskId, instruction, region, clientRequestId })`, then refresh `open_starveil_studio({ taskId })` for Before/After wipe verification.
- Record `{ pageNo, taskId, versionId, previousTaskId, characterIds, localPath, readerUrl }` in `episodes/epXX/manifest.json`.

### 5. Chain Sequels, Vertical Webtoons & 6-Language Exports
- **Page 2+ Continuity**: Always pass the previous approved page's `taskId` as `previousTaskId` (read from `manifest.json` if resuming a later session!) to lock cast, outfits, lighting, and ink onto fresh panel layouts.
- **Vertical Webtoons (9:16)**: `create_webtoon_episode` → `get_webtoon_status` → share hosted `readerUrl` (`https://starveilai.com/w/...`).
- **6-Language Art-Preserving Localization (`10 cr/page`)**: `translate_comic({ sourceVersionId, targetLanguage: 'en' | 'zh-CN' | 'zh-TW' | 'ja' | 'ko' | 'es', clientRequestId })` → `get_translation_status` → save locally as `pXX-<lang>.png`.

## Built-in Quick Recipes (Native MCP Prompts)
For one-shot tasks without an existing project folder, invoke any native prompt directly:
- `xhs-comic` — 2–3 page continuous 3:4 Xiaohongshu carousel + titles & tags
- `article-to-comic` — Distill any local doc, URL, or git release into an explainer comic
- `pet-or-selfie-comic` — 0-credit photo OC lock + instant Page 1
- `global-localize` — Batch translate approved pages across EN / ZH / JA / KO / ES
