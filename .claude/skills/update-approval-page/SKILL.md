# update-approval-page

Rebuild `docs/index.html` from the current week's `draft.json` files, export Canva preview thumbnails to `docs/assets/`, and push everything to GitHub so the approval page at `https://jeffd1130.github.io/Alliance_Cobrinha_LA/` is current.

## When to use

- Jeff says "update the approval page," "refresh the review site," or "rebuild the preview"
- Called automatically at the end of `produce-week`
- After a single `produce-post` or `caption-library` run when Jeff wants to resync the page

## Required tools

- `Canva:export-design` — export PNG thumbnails from draft designs
- GitHub push (via the PostToolUse auto-push hook or manual git commands)

## Step-by-step

### 1. Determine the current week and collect draft.json files

1. Compute ISO week (e.g. `2026-W21`).
2. For each slot in order (02 → 03 → 04, skipping 01 if past):
   - Read `content/<week>/<slot>/drafts/draft.json`.
   - If missing or `status == "raw"`, mark that slot as "no draft" and skip it from the page.

### 2. Export preview images from Canva

For each slot with a draft:
- Call `Canva:export-design` on the draft's `canva.design_id`, format `png`, page 1 only.
- Save the exported PNG to `docs/assets/<slug>.png`:
  - `02-thu-kids-images` → `docs/assets/thu-kids-s1.png`
  - `03-fri-adult-images` → `docs/assets/fri-adults-s1.png`
  - `04-sat-kids-training-videos` → `docs/assets/sat-kids.png`
- If Canva export fails for a slot, use the existing PNG if present; if none, omit the image from that card (the card still renders without it).

### 3. Rebuild docs/index.html

Write `docs/index.html` from scratch using the collected data. The page must:

- **Match the brand aesthetic**: black background (`#0a0a0a`), white/tan text (`#f0ece4`, `#c9b99a`), `#111` card backgrounds, 1px `#222` borders.
- **Header**: "Alliance Cobrinha LA" (small caps, tan) + "Week N — Month DD–DD, YYYY" + "N posts ready for review" + yellow "⏳ Pending Approval" badge.
- **One card per slot**, in posting order:
  - Post header: day + slot label (e.g. "Thursday, May 21 · Kids Carousel"), format sub-line (e.g. "4 slides · Confidence Building"), yellow "⏳ Review" status pill
  - Preview image from `docs/assets/<slug>.png` (lazy-loaded)
  - Schedule chips: "LA drop: 6:00 PM PST" and "Manila: 9:00 AM Fri May 22"
  - Caption block (pre-formatted, preserves line breaks)
  - Hashtags block
  - "Open in Canva to Edit" link button using `canva.edit_url`
- **Footer**: approval instructions + Jeff's email (`jdelasarmas@angelcarehhs.com`)
- No JavaScript. No external CSS framework. Pure inline CSS.
- `<title>Alliance Cobrinha LA — W## Approval</title>`

Reference the existing `docs/index.html` as the style source. Do not deviate from the established color/font system.

### 4. Push to GitHub

The PostToolUse hook in `.claude/settings.json` fires on Write|Edit and auto-pushes. After saving `docs/index.html` and each asset PNG, verify the push landed:

```bash
cd /Users/jeff/Documents/Claude/alliance-cobrinha-la-social && git log --oneline -2
```

If the hook isn't firing (e.g. session opened from a different directory), run manually:

```bash
cd /Users/jeff/Documents/Claude/alliance-cobrinha-la-social
git add docs/
git commit -m "Update W<N> approval page — <date>"
git push origin main
```

### 5. Report back

```
✓ Approval page updated
  URL: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
  Posts included: Thu, Fri, Sat
  Pushed: <commit hash>

Send this link to Cobrinha & Dani for D-1 review.
```

## Edge cases

- **No drafts exist yet**: Tell Jeff there are no drafts for this week and stop.
- **Partial week** (only some slots have drafts): Include only the slots with drafts; note which are missing.
- **GitHub Pages propagation delay**: Changes go live in ~1–2 minutes after push. If Jeff says the page is stale, wait a moment and refresh.
- **Canva export returns multi-page**: Use page 1 only for the preview.
