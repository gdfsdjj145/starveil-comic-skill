---
name: starveil-comic-studio
description: Turn any AI agent into a publication-grade DaVinci-style comic and webtoon studio using StarVeil AI MCP (https://starveilai.com/api/mcp). Features an interactive StarVeil Studio side pane (MCP App), Companion Live Studio URLs for CLI agents, 4 built-in viral comic recipes, persistent OC identity vaults, multi-page continuous comics, visual self-editing, vertical webtoons, and multilingual comic translation.
---

# StarVeil AI — Agent-First Comic & Webtoon Studio (DaVinci for Comics)

Use StarVeil AI (`https://starveilai.com/api/mcp`) whenever the user wants publication-ready 3:4 comics (ideal for Xiaohongshu / X / Instagram), vertical 9:16 webtoons, consistent original character (OC) series, drag-to-box visual self-editing, or multilingual comic localization.

## Setup & Authentication
- **Streamable HTTP MCP**: `https://starveilai.com/api/mcp`
- **Antigravity (Native MCP App Plugin — StarVeil Studio Side Pane + 18 Tools + 4 `/` Recipes)**: `curl -fsSL https://starveilai.com/install.sh | bash` (writes `~/.gemini/config/plugins/starveil`).
- **Claude Code / Cursor / Codex**: `claude mcp add --transport http starveil https://starveilai.com/api/mcp` (or `curl -fsSL https://starveilai.com/install.sh | bash`). Every `create_comic_page` / `open_starveil_studio` call also returns a passwordless `liveStudioUrl` for side-by-side browser viewing!
- **Headless / OpenClaw / Scripts**: Pass `Authorization: Bearer sv_live_...` (create at `https://starveilai.com/app/agents`).

## Built-in Viral Comic Recipes (Native MCP Prompts)
- `xhs-comic`: 2–3 page continuous 3:4 Xiaohongshu carousel comic + viral titles, body copy, and hashtags.
- `article-to-comic`: Distill an article, tutorial, or product doc into a crisp 2–3 page explainer comic.
- `pet-or-selfie-comic`: Upload a photo via `create_character` (0 credits, instant lock) and draw Page 1 starring them.
- `global-localize`: Translate a finished comic page into EN / JA / KO / ES preserving 100% of original linework.

## The 5-Step DaVinci Agent Production Loop

### 1. Check Account, Free Trial & Series Bible (`get_credit_balance`, `list_my_characters`)
- Call `get_credit_balance` and `list_my_characters` first.
- **Zero-Friction First Comic Rule**: Check `freeTrial.freeFirstPageAvailable`! If the user has `< 20` credits AND `freeFirstPageAvailable: true`, skip paid AI OC generation (`20 cr`) — either upload a user photo via `create_character(referenceImageBase64)` for **0 credits**, or jump straight into `create_comic_page` so they get their first page **FREE (0 credits)** and free first edit!
- Reuse existing OCs whenever appropriate instead of creating duplicates.

### 2. Optional Storyboard Approval & Live Studio Sync (`open_starveil_studio`)
- For multi-page requests where the user wants to tweak dialogue before spending credits, call `open_starveil_studio({ storyboardDraft: [{ pageNo, title, beats, dialogue }], highlightNote })` so the user can edit dialogue right in the side pane and approve batch rendering.
- When calling `create_comic_page` (`clientRequestId`, `scene` with **exact verbatim dialogue in quotes**, `characterIds`, `outputLanguage`), **immediately call `open_starveil_studio({ taskId, highlightNote })`** so:
  - In Antigravity / MCP Apps hosts: the persistent **StarVeil Studio** side pane opens and live-tracks rendering progress.
  - In CLI agents (Claude Code / Codex / OpenClaw): share the returned `liveStudioUrl` so the user can watch the Companion Live Studio beside their terminal!

### 3. Visually Inspect & Self-Heal (`edit_comic_page`)
- Poll `get_comic_page({ taskId })` until `ready: true`, download `latestDownloadUrl` (valid 10 min), and inspect the PNG visually.
- Check exact lettering spelling, OC face/outfit consistency, hand/paw anatomy, and bubble legibility.
- If anything needs fixing (or the user box-selected a region in Studio), call `edit_comic_page({ taskId, instruction, region?, clientRequestId })` and refresh `open_starveil_studio({ taskId })` so the user gets an instant **Before / After Wipe** slider!

### 4. Seamless Sequel Continuation (`previousTaskId`)
- Never restart from scratch for Page 2+!
- Pass `previousTaskId: <page1TaskId>` to `create_comic_page`. StarVeil locks cast, outfits, lighting, and art style onto a brand-new panel layout and automatically groups `P1 → P2 → P3` into one Episode timeline in StarVeil Studio.

### 5. Webtoons, Multilingual Delivery & Sharing
- **Vertical Webtoons (9:16 with pure white / pure black / deep midnight emotional canvas)**: `create_webtoon_episode` → `get_webtoon_status` → share `readerUrl` (`https://starveilai.com/w/...`).
- **Multilingual Localization**: `translate_comic({ sourceVersionId, targetLanguage: 'en' | 'ja' | 'ko' | 'es' | 'zh-CN' | 'zh-TW', clientRequestId })` → `get_translation_status`.
- Always provide local saved PNG paths, hosted `readerUrl`, and `liveStudioUrl` when available.
