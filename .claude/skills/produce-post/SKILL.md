# produce-post — Alliance Cobrinha LA

Produce one Instagram post draft end-to-end for Alliance Cobrinha LA: pull a fresh asset from the slot's Google Drive folder, generate a Canva design with the ACL brand kit, write caption + hashtags, and save everything to the repo.

This is the **core skill**. `daily-check`, `produce-week`, and other entry points all call this.

---

## Project constants

| Key | Value |
|-----|-------|
| Project dir | `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social` |
| Canva brand kit | `kAGdeHDeWjQ` |
| Canva workspace folder | `FAHIrnCFIt4` (Alliance Cobrinha LA) |
| W19 theme reference design | `DAHIsVAwhso` |
| GitHub repo | `jeffd1130/Alliance_Cobrinha_LA` |
| Approval page | `https://jeffd1130.github.io/Alliance_Cobrinha_LA/` |

---

## The 4 slots

| Slot ID | Day | Format | Drop PST | Drop Manila | Drive Folder ID |
|---------|-----|--------|----------|-------------|-----------------|
| `01-tue-adult-training-videos` | Tuesday | Vertical reel 9:16 | 7:30 PM | Wed 10:30 AM | `1Gwk2sCsjy33o-oP4RyNOeFi9mCyGIA5h` |
| `02-thu-kids-images` | Thursday | 4-slide story-arc carousel | 6:00 PM | Fri 9:00 AM | `1qDvrFTJdbRlDYYMelWuN5KKk04bwpX9J` |
| `03-fri-adult-images` | Friday | 4-slide photo-gallery carousel | 8:00 PM | Sat 11:00 AM | `1lhVTu2tZ9Tgs3ZYFZnTC1MqJApldro1K` |
| `04-sat-kids-training-videos` | Saturday | Vertical reel 9:16 | 6:30 PM | Sun 9:30 AM | `1UBTq6UZSvW-8xiVAOacOr3ue-Zg4McG2` |

**Slot-specific asset rules (validated W21):**
- `02-thu-kids-images`: UUID filenames — no descriptive names. **Pick by file size >400 KB** (larger = better quality).
- `03-fri-adult-images`: Photos live in the **Image Resources subfolder** (`1lhVTu2tZ9Tgs3ZYFZnTC1MqJApldro1K`), NOT the root Adult Images folder. The folder ID above already points to the subfolder.
- `04-sat-kids-training-videos`: Folder is NOT empty — contains MP4 files. **Prefer MP4 over MOV** (smaller, faster Canva upload). If folder appears empty, re-query — Drive MCP occasionally misses files on first call.

---

## When to use

- Jeff says "make tomorrow's post," "produce Tuesday's reel," "run Friday's post," etc.
- Another skill (e.g. `daily-check`, `produce-week`) invokes this with a specific slot ID
- Jeff wants to regenerate a draft ("redo Friday's draft")

## Inputs

- **slot ID** — one of the 4 above. If Jeff says "tomorrow's post," compute LA tomorrow in `America/Los_Angeles` and map to slot.
- **week** — defaults to current ISO week (`YYYY-W##`). Override only if Jeff specifies.
- **asset override** (optional) — if Jeff or `brief.md` names a specific file, use it.

## Required tools

- `Canva:generate-design`, `Canva:create-design-from-candidate`, `Canva:upload-asset-from-url`, `Canva:merge-designs`, `Canva:move-item-to-folder`, `Canva:start-editing-transaction`, `Canva:perform-editing-operations`, `Canva:commit-editing-transaction`, `Canva:cancel-editing-transaction`
- `Google Drive:search_files`, `Google Drive:get_file_metadata`, `Google Drive:get_file_permissions`

If any are unavailable, stop and tell Jeff which connector needs reconnecting.

---

## Step-by-step

### 1. Resolve the slot

Map Jeff's natural language to the slot ID using the table above. If Jeff says "tomorrow's post," compute LA tomorrow in `America/Los_Angeles` and match the weekday. If tomorrow is Sunday or Monday, tell Jeff there is no scheduled post.

### 2. Ensure the week folder exists

Compute current ISO week as `YYYY-W##`. If `content/<week>/<slot>/` doesn't exist, create it with subfolders: `raw/`, `drafts/`, `approved/`. Check `content/<week>/<slot>/brief.md` for any context from Cobrinha or Dani.

### 3. Pick the source asset from Drive

Query `Google Drive:search_files` with `parentId = '<Drive Folder ID from table above>'`.

Sort by `modifiedTime` descending. Apply slot-specific rules:
- **Thu** (`02`): sort by file size descending instead; pick largest file >400 KB
- **Fri** (`03`): already pointed at Image Resources subfolder — pick newest JPG
- **Sat** (`04`): prefer `.mp4` over `.mov`; if no results, re-query once

Check file permissions: `get_file_permissions` must return a permission with `type: "anyone"`. If not, stop and tell Jeff to enable link-sharing on that file.

Build the Canva-fetchable URL:

**Images** (`image/jpeg`, `image/heic`):
```
https://lh3.googleusercontent.com/d/<FILE_ID>=w2000
```

**Videos** (`video/mp4`, `video/quicktime`):
```
https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download&authuser=0&confirm=t
```

**Hard rules:**
- NEVER use `drive.google.com/uc?id=X&export=download` — breaks on files >25 MB
- NEVER use the lh3 URL for videos — Canva silently stores a poster frame image instead of the video (symptom: upload response shows `"type":"image"` when source was a video)

### 4. Upload the asset to Canva

Call `Canva:upload-asset-from-url`:
- `url`: the Drive URL from step 3
- `name`: `<slot-id>-<YYYY-MM-DD>-source` (e.g. `02-thu-kids-images-2026-05-21-source`)

Save the returned `asset_id`.

### 5. Generate the Canva design

Always use brand kit `kAGdeHDeWjQ`. Theme anchor: black background, tan/beige blob accents in opposing corners, large serif headline white top-left, hero media in framed window (not full-bleed), magazine-editorial restraint. Reference design `DAHIsVAwhso` (W19) if needed.

#### 5a. Single-slide reels (slots 01 and 04)

Call `Canva:generate-design`:
- `brand_kit_id`: `kAGdeHDeWjQ`
- `design_type`: `instagramReel` (1080×1920)
- `query`: slot's generation prompt from `templates.json`
- `asset_ids`: `[<asset_id>]`

Pick the first candidate. Materialize via `Canva:create-design-from-candidate`.

#### 5b. Photo-gallery carousel (slot 03 — Friday)

1. Pull the 4 newest JPGs from Drive folder `1lhVTu2tZ9Tgs3ZYFZnTC1MqJApldro1K`. Upload all 4 to Canva in parallel — you get 4 asset IDs.
2. Call `generate-design` for each of the 4 assets **in parallel** — same generation prompt, one asset per call.
3. Materialize each via `create-design-from-candidate` (first candidate each time). You now have 4 single-page designs.
4. Build the carousel via `Canva:merge-designs` — **ONE operation per call** (Canva constraint, validated W21):
   - Call 1: `type: "create_new_design"`, `title: "ACL Fri Adult Photos <week>"`, one `insert_pages` from design 1, `page_numbers: [1]`
   - Call 2: `type: "modify_existing_design"`, target the new carousel ID, one `insert_pages` from design 2
   - Call 3: same, design 3
   - Call 4: same, design 4
5. The merged 4-page design is the draft.

#### 5c. Story-arc carousel (slot 02 — Thursday)

4-slide arc: 1 text-only cover → 2 photo slides → 1 CTA slide.

1. Pull 2 kids photos from Drive folder `1qDvrFTJdbRlDYYMelWuN5KKk04bwpX9J` (pick by file size >400 KB). Upload both in parallel.
2. Generate 4 slides **in parallel**: slide 1 (cover — no asset_ids), slide 2 (photo, asset 1), slide 3 (photo, asset 2), slide 4 (CTA — no asset_ids).
3. Materialize each candidate.
4. Merge in order (1→2→3→4) via 4 sequential `merge-designs` calls — one operation each.

### 6. Move design into ACL workspace folder

Call `Canva:move-item-to-folder`:
- `item_id`: the design ID
- `folder_id`: `FAHIrnCFIt4`

If the move fails, log a warning and continue — the design still exists.

### 7. Tune the design (if asset wasn't auto-inserted)

If Canva's generator didn't place the asset in the hero slot:
1. `start-editing-transaction` on the design
2. Find the largest media element on page 1
3. `perform-editing-operations` with `update_fill` to swap in the uploaded asset
4. `commit-editing-transaction` (or `cancel-editing-transaction` on failure — the un-tuned draft is still usable)

### 8. Save metadata

Write `content/<week>/<slot>/drafts/draft.json`:

```json
{
  "week": "2026-W##",
  "slot": "<slot-id>",
  "day": "<weekday>",
  "date_la": "YYYY-MM-DD",
  "drop_pst": "HH:MM",
  "drop_manila": "HH:MM Weekday YYYY-MM-DD",
  "status": "draft",
  "canva": {
    "design_id": "<id>",
    "edit_url": "https://www.canva.com/d/<id>",
    "page_count": <n>,
    "format": "carousel|reel"
  },
  "drive_assets": [
    {"file_id": "<id>", "canva_asset_id": "<id>", "slide": 1}
  ],
  "generated_at": "<ISO datetime +08:00>"
}
```

### 8b. Invoke caption-library

Call `caption-library` for this slot. It generates the on-brand caption + hashtags and writes `drafts/caption.md` and `drafts/hashtags.md`. If it fails, log and continue — Jeff can run it standalone.

### 9. Report back

```
✓ <slot> draft ready
  Canva: https://www.canva.com/d/<id>
  Source: <filename> (Drive: <folder name>)
  Drops: <weekday> at <drop_pst> PST / <drop_manila> Manila
  Files: content/<week>/<slot>/drafts/
```

---

## Failure modes

| Failure | Action |
|---------|--------|
| Drive folder empty / no recent files | Stop. Tell Jeff to ping Cobrinha & Dani for D-3 assets. |
| File not link-shared (`type: anyone` missing) | Stop. Tell Jeff which file needs sharing enabled. |
| `generate-design` fails | Retry once. If it fails again, stop and tell Jeff. |
| `create-design-from-candidate` fails | Try next candidate. If all fail, stop. |
| Editing transaction fails | Cancel. Keep un-tuned draft. Log in `draft.json`. |
| lh3 URL returns image for a video slot | Wrong URL pattern used. Switch to `drive.usercontent.google.com` pattern and re-upload. |

---

## What this skill does NOT do

- **Post to Instagram.** Posting is always Jeff's manual call at D-0.
- **Generate captions directly.** That's `caption-library`, called automatically in step 8b.
- **Approve the draft.** Cobrinha & Dani approve at D-1 via the GitHub Pages review site.
- **Modify master templates.** Skills duplicate from masters — never edit the master itself.
