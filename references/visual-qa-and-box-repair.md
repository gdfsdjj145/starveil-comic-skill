# Reference: Visual QA & 3-Zone Bounding-Box Repair

Never deliver a comic page blindly without verifying lettering and artwork. StarVeil 2.0 gives agents **two complementary QA passes**:
1. **Deterministic Machine Bubble QA (`get_comic_page({ taskId, inspectBubbles: true })`)**: Free & KV-cached per `versionId` (works even in headless CLI / OpenClaw without vision tokens!). Returns every detected speech bubble's literal `ocrText`, exact `0..1` `bbox`, pre-padded **`editRegion`** (+2.5% halo buffer), `typoAlerts`, and `missingExpectedLines` diffed against your quoted script.
2. **Visual PNG Inspection**: Download `latestDownloadUrl` (valid 10 minutes) and inspect faces, hands, tails, and panel framing visually (or in StarVeil Studio 2.0).

---

## 0. Automated Zero-Typo Loop (No Manual Coordinate Guessing!)

```json
// Step A: After ready=true, run deterministic bubble QA once
{ "tool": "get_comic_page", "arguments": { "taskId": "<taskId>", "inspectBubbles": true } }

// Step B: If bubbleQa.typoAlerts is non-empty, pass alert.editRegion straight into edit_comic_page
{
  "tool": "edit_comic_page",
  "arguments": {
    "taskId": "<taskId>",
    "clientRequestId": "auto-fix-b1",
    "instruction": "<alert.suggestedInstruction>",
    "region": "<alert.editRegion>"
  }
}
```

---

## 1. 7-Point Visual QA Rubric

Before approving a page or chaining Page 2+ via `previousTaskId`, inspect every panel for:

1. **Exact Lettering & Spelling**: Every word inside speech balloons and captions matches the user's script verbatim — zero typos, missing CJK strokes, or garbled letters.
2. **Speaker Attribution & Tail Direction**: Every speech bubble tail points cleanly to the right speaker; reading order flows naturally top-left → bottom-right (or right-to-left if manga mode was requested).
3. **Identity & Outfit Lock**: Face, hairstyle, species/markings, and current outfit remain recognizable against the OC reference and previous page.
4. **No Face/Outfit Swaps**: In multi-character panels, Character A and Character B did not trade hairstyles, jackets, or props.
5. **Hands, Props & Anatomy**: Finger counts, eye gaze, and held objects look natural at normal viewing size.
6. **Clean Framing (Zero Occlusion)**: Speech balloons do not cover eyes, mouths, or key storytelling props.
7. **Mobile Readability**: Dialogue stays crisp and legible when scaled to phone width (`~390px`).

---

## 2. The 3-Zone Bounding-Box Repair Model (`edit_comic_page`)

When 90%+ of a page works and only 1 speech bubble, hand, or expression failed, **never reroll the entire page** — use `edit_comic_page` (`25 cr`; **1st edit free** on new verified accounts) with a normalized `0..1` rectangle (`top-left = (0, 0)`, `bottom-right = (1, 1)`):

- **Anchor Zone (outside the box)**: Everything outside your `region` stays visually anchored (lighting, faces, other panels, good bubbles).
- **Halo Buffer (2%–3% padding outside the target outline)**: For speech bubbles, expand the box slightly past the black balloon border (~`0.02–0.03` on each side). Cropping strictly inside the white interior risks clipped ghost strokes from old letters.
- **Active Rewrite Zone (inside the box)**: State explicitly what changes AND what must stay identical.

---

## 3. Ready-to-Use Repair Templates

### Template A — Text-Only Speech Bubble Fix (Zero Art Drift)
```json
{
  "tool": "edit_comic_page",
  "arguments": {
    "taskId": "<taskId>",
    "clientRequestId": "fix-p1-bubble-1",
    "instruction": "Text-only edit inside the boxed white speech bubble: replace the dialogue with exact text \"He left at midnight.\" Match the existing comic lettering weight and bubble shape. Keep the character face, background art, panel border, and all surrounding areas completely unchanged.",
    "region": { "x": 0.52, "y": 0.05, "width": 0.42, "height": 0.16 }
  }
}
```

### Template B — Anatomy / Prop / Expression Repair
Include the full connected sub-element (e.g., wrist + hand + held object, not just two fingertips) so the model has coherent structural context:

```json
{
  "tool": "edit_comic_page",
  "arguments": {
    "taskId": "<taskId>",
    "clientRequestId": "fix-p1-hand-1",
    "instruction": "Repair the character's right hand holding the silver key so it has five natural fingers matching the surrounding ink line weight and lighting. Do not change the face, coat, background, or any speech bubbles.",
    "region": { "x": 0.18, "y": 0.56, "width": 0.28, "height": 0.24 }
  }
}
```

---

## 4. When to Repair vs. When to Split/Regenerate
- **Use `edit_comic_page`** when: 1–2 bubbles have typos, one hand/prop/expression drifted, or a translated punchline needs an idiom tweak.
- **Adjust the script & regenerate** when: the entire page layout is wrong, characters swapped roles across 3+ panels, or a balloon is physically too tiny for a 35-word paragraph (shorten the line or split into two panels first).
- **Always refresh `open_starveil_studio({ taskId })`** after an edit so the user can scrub the **Before / After wipe slider** to verify zero seams.
