# Reference: Local Comic Workspace & Asset Management

Use this guide whenever a user brings existing local files — novel chapters, Markdown scripts, Obsidian vaults, character photos, or product docs — or wants to build a multi-page / multi-episode comic across multiple agent sessions.

## Core Design: Plain Files on Disk + Cloud Identity/Render IDs
Never trap the user's canon inside a proprietary web form or chat transcript. Keep human-readable Markdown/JSON in the user's own workspace and store only lightweight StarVeil ID mappings so **any agent (Claude Code, Antigravity, Cursor, Codex, OpenClaw)** can pick up seamlessly from Page N.

---

## Standard Directory Layout

```text
<project-root>/
├── bible.md                     # Author's world, tone, art direction, recurring visual rules
├── cast/                        # Local OC photos, turnaround sheets, or prop references
│   ├── mochi.jpg
│   └── detective-lin.png
├── starveil-project.json        # Cast ID map + shared style defaults
└── episodes/
    └── ep01/
        ├── source.md            # Optional: user's raw chapter / notes / outline
        ├── script.md            # Editable page-by-page panel beats & verbatim dialogue
        ├── pages/               # Saved lossless PNGs (p01.png, p02.png, p01-ja.png...)
        └── manifest.json        # Machine checkpoint for previousTaskId & versionId
```

*(If the user already has a folder structure like `chapters/` or `characters/`, respect their existing paths and place only `starveil-project.json` and `manifest.json` alongside them.)*

---

## 1. `starveil-project.json` Schema (Zero-Duplicate Cast Sync)

```json
{
  "schemaVersion": 1,
  "projectTitle": "Midnight Yarn Chronicles",
  "defaultLanguage": "en",
  "defaultArtStyle": "warm watercolor healing storybook comic, soft ink outlines, golden rim light",
  "characters": [
    {
      "localKey": "mochi",
      "displayName": "Mochi the Tabby",
      "localReferencePath": "./cast/mochi.jpg",
      "characterId": "chr_01abc...",
      "permanentIdentity": "Warm orange tabby kitten, white paws, bright amber eyes, tiny indigo developer hoodie",
      "defaultLook": "Indigo hoodie with small stitched pocket"
    }
  ]
}
```

### Sync Algorithm (Run Before Rendering)
1. Read `starveil-project.json` (if present) and call `list_my_characters` via StarVeil MCP.
2. For each character needed in the upcoming scene:
   - If `characterId` is already present and confirmed in `list_my_characters` (`readyForComics: true`), reuse it directly (`0 cr`).
   - Else if a matching local image exists in `cast/` (or a path provided by the user), encode it as base64 and call `create_character({ name, description, referenceImageBase64 })` (**0 credits**, ready immediately), then persist the returned `characterId` into `starveil-project.json`.
   - Only call `create_character({ generateReferenceImage: true })` (`20 cr`) when the user has **no** reference image and explicitly wants StarVeil to design a canonical full-body sheet (skip this if `freeTrial.freeFirstPageAvailable` is true and credits `< 20`).

---

## 2. `episodes/epXX/manifest.json` Schema (Cross-Session Continuity Checkpoint)

```json
{
  "episodeId": "ep01",
  "title": "The 3 AM Deploy",
  "pages": [
    {
      "pageNo": 1,
      "taskId": "task_p1_xxx",
      "versionId": "ver_p1_v2",
      "previousTaskId": null,
      "characterIds": ["chr_01abc..."],
      "status": "approved",
      "localImagePath": "./pages/p01.png",
      "readerUrl": "https://starveilai.com/w/...",
      "storyStateEnd": "Mochi is tangled completely in red yarn across the mechanical keyboard; dawn light is just starting outside.",
      "translations": {
        "ja": {
          "translationId": "tr_xxx",
          "localImagePath": "./pages/p01-ja.png"
        }
      }
    }
  ]
}
```

### Resuming After Days or Switching Agents
When the user says *"Continue Episode 1 Page 4"* (even in a fresh chat or different editor):
1. Read `starveil-project.json`, `bible.md`, and `episodes/ep01/manifest.json`.
2. Grab Page 3's `taskId` → pass as `previousTaskId` to `create_comic_page`.
3. Read Page 3's `storyStateEnd` (props held, injuries, lighting, location) so Panel 1 of Page 4 picks up seamlessly without narrative teleportation.
4. After `get_comic_page` returns `ready: true`, download `latestDownloadUrl` immediately (signed URLs expire in 10 minutes!) to `./pages/p04.png`, visually inspect, and append Page 4 to `manifest.json`.

---

## 3. Working With Existing User Material (Obsidian / Fountain / SillyTavern / Codebases)

### A. Obsidian Fiction Vaults (Auto-Detect Without Reorganizing)
Many long-form writers keep an Obsidian Story Bible shaped like:
```text
My Novel Vault/
├── 01_Manuscript/Act I/Chapter 01/Scene 1.md   (or Longform scenes)
├── 02_Worldbuilding/
│   ├── Characters/*.md                         (one note per OC with YAML frontmatter)
│   ├── Locations/*.md
│   └── Bestiary & Items/*.md
└── 04_Outlines & Plots/
```
- **Rule**: Never move or rename the user's Obsidian folders. Read their `Characters/*.md` YAML frontmatter (`name`, `appearance`, `outfit`, `age`, `traits`) and `Locations/*.md` directly in place, and store only `starveil-project.json` + `episodes/epXX/manifest.json` in a clean `comic-output/` subfolder (or vault root if requested).

### B. SillyTavern / Chub `chara_card_v2` & `v3` PNG Character Cards (0-Credit 1-Command Import!)
If the user brings `.png` character cards exported from SillyTavern, Chub, or Lorebook tools (which embed JSON metadata inside PNG `tEXt` `chara` / `ccv3` chunks):
- Run `starveil oc import ./path/to/card.png --project ./starveil-project.json` (or `--dry-run` first to inspect).
- The CLI parses the embedded JSON locally, builds a clean visual identity summary, uploads the exact PNG portrait to StarVeil at **0 credits** (`create_character`), and records the resulting `characterId` into `starveil-project.json` automatically.

### C. Fountain Comic Scripts (`.fountain`)
If the user brings `.fountain` comic scripts (`**PAGE 1**`, `_PANEL 1_`, `ALL CAPS CHARACTER` + indented dialogue + `(V.O.)` captions), parse each `**PAGE X**` block 1-to-1 into `create_comic_page` without rewriting dialogue. See `references/faithful-script-to-panels.md` for exact mapping rules.

### D. Long Novel Chapters & Codebases
- **Long Novel Chapters (`5,000+ words`)**: Do not stuff a full chapter into one prompt. First produce a concise scene-beat table (`episodes/epXX/script.md`) mapping 1 chapter → 4–8 comic pages, confirm which pages to render first (start with Page 1 as a visual pilot!), and render sequentially.
- **Git Repo / Release Notes / Blog Posts**: Read `git log`, `CHANGELOG.md`, or the article directly from disk; extract 1 clear narrative arc (Problem → Turning Point → Punchline) rather than listing dry bullet points.
- **Cost Transparency**: Before batch-rendering `N` pages, state exact credit math upfront (`N × 25 cr` for 3:4 pages; `10 cr/page` per translation; `0 cr` for photo OC uploads; `1st page + 1st edit free` on new accounts).
