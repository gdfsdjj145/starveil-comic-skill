# Reference: Faithful Script-to-Panel Direction Craft

Use this guide when adapting a user's existing story, script, novel chapter, or notes into StarVeil `create_comic_page` prompts. Your goal is **faithful visual translation** — preserve the author's canon and dialogue voice while making every page visually readable.

---

## 1. The 3-Layer Consistency Split (Never Mix Them in Prompts)

Separate every character's continuity into three distinct layers:

1. **Permanent Identity (anchored by `characterIds`)**: Face shape, eye color, hair, proportions, age cues, permanent marks. Do NOT endlessly re-describe facial anatomy once locked via `characterIds` — conflicting prose causes drift.
2. **Current Look (stated explicitly per scene/arc)**: Current outfit, coat, weather effects (wet hair, snow), bandages/injuries, or handheld prop. Explicitly state when an outfit stays the same or changes intentionally.
3. **Narrative State (carried page-to-page via `manifest.json` + `previousTaskId`)**: Where characters stand, what object is in which hand, time of day, and emotional momentum from the previous page's final panel.

---

## 2. Format & Panel Density Budgets

| Format | Aspect Ratio | Ideal Panels / Page | Target Dialogue Budget per Balloon |
|---|---|---|---|
| **Standard Comic Page** (`create_comic_page`) | `3:4` (`1086×1448`) | **3–4 panels** (max 5 for fast beats) | **≤ 14 English words** or **≤ 22 CJK characters** |
| **Vertical Webtoon Beat** (`create_webtoon_episode`) | `9:16` scroll | **1–3 vertical beats** with breathing gutters | **≤ 12 English words** or **≤ 18 CJK characters** |

### Why Concise Balloons Matter
- Balloons packed with 30+ words shrink font size below mobile legibility (`390px` phone width), invite spelling artifacts, and cover faces/action.
- **Author-Respect Rule**: If the user's source script has a 40-word speech, do **not** silently cut their punchline. Either split the line across two sequential panels (reaction shot + follow-up) or present a tightened version in `script.md` for their 1-line approval first.

---

## 3. High-Precision `scene` Prompt Structure for `create_comic_page`

Write concrete, filmable visual beats rather than abstract adjectives ("make it epic"):

```text
Style & Palette: [Consistent art style / LUT from bible.md, e.g. "cinematic neo-noir ink with cool cyan rain shadows and warm amber streetlights"].
Cast in scene: [Name A] (wearing [current outfit]), [Name B] (wearing [current outfit]).
Layout: 4-panel vertical-read 3:4 comic page with clean gutters and clear quiet space for speech bubbles.

Panel 1 (Wide establishing shot): [Concrete setting, lighting, character positions, visible action]. Caption box top-left: "[Exact verbatim caption]".
Panel 2 (Medium over-the-shoulder shot): [Visible physical action + expression]. [Name A] says: "[Exact verbatim dialogue]".
Panel 3 (Close-up reaction): [Eye direction, hand/prop detail, emotional shift]. [Name B] replies: "[Exact verbatim dialogue]".
Panel 4 (Dramatic low-angle payoff): [Visual twist or punchline that moves the plot forward]. SFX: "[Optional short SFX]".
```

### 6 Non-Negotiable Director Rules
1. **Quote every spoken word verbatim (`"..."`)**: Never summarize dialogue like *"she argues angrily"* — always write exact words inside double quotes and name the exact speaker.
2. **One visual beat per panel**: A character cannot open a door, walk across the room, and sit down inside one panel. Pick the single most expressive freeze-frame.
3. **Vary camera shots across panels**: Mix `Wide establishing` → `Medium action` → `Close-up emotion` → `Low/High dramatic payoff`. Four identical waist-up medium shots feel static and repetitive.
4. **Show, don't duplicate**: If the art clearly shows a spilled coffee cup, remove *"Oh no, I spilled my coffee!"* from the dialogue.
5. **Limit crowded casts per close-up**: Keep close-up and medium panels to **1–3 named characters** to eliminate face/outfit swaps.
6. **Reserve quiet space for text**: Avoid placing busy high-contrast patterns directly behind where top speech balloons belong.

---

## 4. Anti-Copy Page 2+ Continuation (`previousTaskId`)
When generating Page 2+ with `previousTaskId`:
- Explicitly start with *"Direct continuation after the previous page; use a fresh, distinct panel layout."*
- Briefly anchor the handoff state (*"Still wearing the rain-soaked trench coat, holding the silver lighter from the last panel"*).
- Do not repeat Panel 1's composition unless intentionally doing a comedic callback.
