# update-approval-page — Alliance Cobrinha LA

Rebuild `docs/index.html` from the current week's `draft.json` files, export Canva preview thumbnails, push to GitHub, and send the Telegram approval message to @jeffd321.

---

## Project constants

| Key | Value |
|-----|-------|
| Project dir | `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social` |
| GitHub repo | `jeffd1130/Alliance_Cobrinha_LA`, branch `main`, source `/docs` |
| GitHub Pages URL | `https://jeffd1130.github.io/Alliance_Cobrinha_LA/` |
| Jeff's email | `jdelasarmas@angelcarehhs.com` |
| Telegram bot | `@cobrinhaLAdesignsbot` |
| Approver handle | `@jeffd321` |

## Brand colors (for index.html)

| Token | Hex |
|-------|-----|
| Page background | `#0a0a0a` |
| Card background | `#111` |
| Card border | `#222` |
| White text | `#f0ece4` |
| Tan/gold accent | `#c9b99a` |
| Divider | `#1a1a1a` |

---

## When to use

- Jeff says "update the approval page," "refresh the review site," or "rebuild the preview"
- Called automatically at the end of `produce-week` and `daily-check`
- After a single `produce-post` or `caption-library` run when Jeff wants the page resynced

---

## Step-by-step

### 1. Collect draft.json files

Compute current ISO week. For each slot in posting order (02 → 03 → 04, skip 01 if past):
- Read `content/<week>/<slot>/drafts/draft.json`
- If missing or `status == "raw"`: mark as "no draft" and exclude from page

### 2. Export preview PNGs from Canva (in parallel)

For each slot with a draft, call `Canva:export-design`:
- `design_id`: from `draft.json → canva.design_id`
- `format`: `png`
- `page`: 1 only

Save to exact paths:

| Slot | PNG path |
|------|----------|
| `02-thu-kids-images` | `docs/assets/thu-kids-s1.png` |
| `03-fri-adult-images` | `docs/assets/fri-adults-s1.png` |
| `04-sat-kids-training-videos` | `docs/assets/sat-kids.png` |

If Canva export fails for a slot: use the existing PNG if present; if none exists, the card still renders without an image.

### 3. Rebuild docs/index.html

Write `docs/index.html` from scratch. Requirements:

- `<title>Alliance Cobrinha LA — W## Approval</title>`
- No JavaScript. No external CSS. Pure inline CSS.
- Colors must match brand constants above — do not deviate.
- Reference the existing `docs/index.html` as the layout source.

**Structure:**
```
<header>
  Alliance Cobrinha LA (small caps, tan #c9b99a)
  Week N — Month DD–DD, YYYY
  N posts ready for review
  ⏳ Pending Approval badge (gold border, dark bg)

<div class="posts">
  [one .post-card per slot, in posting order]

<footer>
  Approval instructions
  jdelasarmas@angelcarehhs.com
  Alliance Cobrinha LA · W## · Generated YYYY-MM-DD
```

**Each post card must include:**
1. Header: day + format label (e.g. "Thursday, May 21 · Kids Carousel"), sub-line (e.g. "4 slides · Confidence Building"), `⏳ Review` status pill
2. Preview image: `<img src="assets/<slug>.png" loading="lazy">`
3. Schedule chips: `📍 LA drop: 6:00 PM PST` and `🇵🇭 Manila: 9:00 AM Fri May 22`
4. Caption block (preserves line breaks via `white-space: pre-line`)
5. Hashtags block
6. Canva edit button: `<a href="<canva.edit_url>">Open in Canva to Edit</a>`

### 4. Push to GitHub

The PostToolUse hook auto-commits and pushes after Write/Edit **when Claude Code is opened from the project directory**. Verify push:

```bash
cd /Users/jeff/Documents/Claude/alliance-cobrinha-la-social && git log --oneline -2
```

If the hook isn't firing (session opened from elsewhere), push manually:

```bash
cd /Users/jeff/Documents/Claude/alliance-cobrinha-la-social
git add docs/
git commit -m "Update W<N> approval page — <YYYY-MM-DD>"
git push origin main
```

### 5. Send Telegram approval message

After push is confirmed, send via `@cobrinhaLAdesignsbot` to `@jeffd321`:

```
Alliance Cobrinha LA — W[N] ready for review ✅

[one line per slot produced:]
📅 Thu [date] · Kids Carousel · 6:00 PM PST / Fri 9:00 AM Manila
📅 Fri [date] · Adult Photos  · 8:00 PM PST / Sat 11:00 AM Manila
📅 Sat [date] · Kids Reel     · 6:30 PM PST / Sun 9:30 AM Manila

Review all posts here:
https://jeffd1130.github.io/Alliance_Cobrinha_LA/

Open each Canva link on the page to edit. Reply to approve or flag changes.
```

If `@jeffd321` has not yet messaged `@cobrinhaLAdesignsbot`, call `get_updates` first to retrieve their numeric chat ID. If Telegram MCP is not connected, log a warning and output the message for Jeff to send manually.

### 6. Report back

```
✓ Approval page updated
  URL: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
  Posts: Thu, Fri, Sat
  Pushed: <commit hash>
  Telegram: sent to @jeffd321 ✓

Changes live in ~1–2 min. Hard-refresh the URL if it looks stale.
```

---

## Edge cases

- **No drafts exist yet**: Stop. Tell Jeff there are no drafts for this week.
- **Partial week** (only some slots have drafts): Include only slots with drafts; note which are missing.
- **GitHub Pages propagation**: Changes go live ~1–2 min after push. Tell Jeff to hard-refresh (`Cmd+Shift+R`) if the page looks stale.
- **Canva export returns multi-page**: Always use page 1 only.
