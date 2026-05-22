# produce-post

Produce one Instagram post draft end-to-end: pull a fresh asset from the matching Google Drive folder, generate a Canva design with the brand kit applied, swap the asset and headline text into the design, export the draft URL, and save metadata to `content/<week>/<slot>/drafts/`.

This is the **core skill**. `daily-check` and other entry points all eventually call this.

## When to use

- Jeff says "make tomorrow's post," "produce Tuesday's reel," "run the Friday post," etc.
- Another skill (e.g. `daily-check`) invokes this with a specific slot ID
- Jeff wants to regenerate a draft that already exists ("redo Friday's draft")

## Inputs

- **slot ID** — one of `01-tue-adult-training-videos`, `02-thu-kids-images`, `03-fri-adult-images`, `04-sat-kids-training-videos`. If Jeff says "tomorrow's post," compute the LA day-of-week tomorrow and map to the slot.
- **week** — defaults to current ISO week (e.g. `2026-W18`). Override only if Jeff specifies.
- **asset override** (optional) — if Jeff says "use the file named X" or specifies a file in the slot's `brief.md`, use that. Otherwise use the most recently modified file in the slot's primary Drive folder.

## Required tools

- `Canva:list-brand-kits`, `Canva:generate-design`, `Canva:create-design-from-candidate`, `Canva:upload-asset-from-url`, `Canva:start-editing-transaction`, `Canva:perform-editing-operations`, `Canva:commit-editing-transaction`, `Canva:cancel-editing-transaction`, `Canva:move-item-to-folder`, `Canva:get-design`
- `Google Drive:search_files`, `Google Drive:get_file_metadata`

If any of these aren't available, **stop** and tell Jeff which connector needs to be added.

## Step-by-step

### 1. Resolve the slot

1. Read `templates.json`. Confirm `mode == "fresh-generate"` (or that the requested slot has `design_id != null` for registered-master mode).
2. Find the slot in `templates.json["templates"]`. Pull: `primary_drive_folder_id`, `generation_prompt`, `design_type`, `schedule`, `placeholders`.
3. If Jeff said "tomorrow's post," compute LA tomorrow's weekday in the *America/Los_Angeles* timezone, then match to the slot whose `schedule.day_la` matches. If no match (e.g. tomorrow is Sunday or Monday), tell Jeff there's no scheduled post and stop.

### 2. Ensure the week folder exists

1. Compute current ISO week as `YYYY-W##` (use Python's `date.isocalendar()`).
2. If `content/<week>/` doesn't exist, copy `content/_template/` to `content/<week>/`.
3. Ensure `content/<week>/<slot>/raw/`, `drafts/`, `approved/` exist.

### 3. Pick the source asset from Drive

1. Check `content/<week>/<slot>/brief.md` — if it names a specific file, use that.
2. Otherwise call `Google Drive:search_files` with query:
   ```
   parentId = '<primary_drive_folder_id>'
   ```
   (Note: `trashed` is not a supported field in this Drive MCP; just rely on parentId.)
   Sort results by `modifiedTime` descending. Pick the newest file. For image slots prefer `image/jpeg` over `image/heif`. Skip files older than 30 days; if all are stale, warn Jeff.
3. Get the file's metadata + permissions. Check `get_file_permissions` returns a permission with `type: "anyone"` (any role). If not, **stop** and tell Jeff which file needs link-sharing turned on.
4. Build the Canva-fetchable URL based on the file's mimeType:

   **Images** (`image/*`) — use Google's content-server pattern:
   ```
   https://lh3.googleusercontent.com/d/<file_id>=w2000
   ```
   The `=w2000` suffix gets a 2000px-wide rendition. Bypasses Drive's download interstitial that breaks `drive.google.com/uc?id=X&export=download`.

   **Videos** (`video/*`, including `video/mp4` and `video/quicktime`) — use the new Drive download endpoint with `confirm=t`:
   ```
   https://drive.usercontent.google.com/download?id=<file_id>&export=download&authuser=0&confirm=t
   ```
   Validated up to 91 MB on both `.mp4` and `.mov` files (2026-05-05 test). Canva will return `"type": "video"` in the upload response with width, height, and duration metadata.

   **DO NOT** use the lh3 pattern for videos — Canva will accept the request and silently store a poster-frame **image** instead of the video. If you see `"type": "image"` in the upload response when the source was a video, that's the symptom.

   **DO NOT** use `drive.google.com/uc?id=X&export=download` for either. It returns Google's download-confirmation interstitial (HTTP non-200) for files larger than ~25 MB.

### 4. Upload the asset to Canva

Call `Canva:upload-asset-from-url` with:
- `url`: the Drive direct-download URL
- `name`: `<slot>-<date>-source` (e.g. `01-tue-adult-training-videos-2026-05-05-source`)

Save the returned `asset_id`.

### 5. Generate the Canva design

**Branch on `slot.carousel.enabled`:**

#### 5a. Single-slide path (default — videos and any slot without carousel.enabled)

Call `Canva:generate-design`:
- `brand_kit_id`: from `templates.json["canva_workspace"]["brand_kit_id"]`
- `design_type`: from the slot config
- `query`: the slot's `generation_prompt`
- `asset_ids`: `[<asset_id from step 4>]`

Pick the **first candidate** from the response. Materialize via `Canva:create-design-from-candidate`. Save `design_id` and `design_url`.

#### 5b. Carousel path (image slots with `carousel.enabled = true`)

If `carousel.pattern == "photo-gallery"` (Friday):

1. Pull `carousel.slide_count` newest images from the slot's Drive folder (not just 1). Repeat steps 3 + 4 of this skill for each — building one Canva-fetchable URL per asset and uploading each to Canva. You now have `slide_count` Canva asset IDs.
2. For each asset, call `generate-design` in parallel using the slot's `generation_prompt` (same prompt for every slide — that's what makes it photo-gallery).
3. For each generated job, materialize the **first** candidate via `create-design-from-candidate`. You now have `slide_count` materialized designs. Note: Canva's generator sometimes returns multi-page designs when asked for a single Instagram post; in step 4 of merging, always specify `page_numbers: [1]` to take only page 1.
4. Build the carousel via `Canva:merge-designs`:
   - First call: `type: "create_new_design"`, `title: "<slot> <week> — Photo Gallery Carousel"`, one `insert_pages` operation taking `page_numbers: [1]` from the first source design. **`merge-designs` only accepts ONE operation per call** despite the array schema; do not batch.
   - Subsequent calls: `type: "modify_existing_design"`, `design_id: <new carousel ID>`, one `insert_pages` operation per remaining source design.
5. The final carousel design is the slot's draft. Discard the per-slide source designs (or leave them in the workspace folder; they'll just clutter slightly). The merged design's ID and URL are what go in `draft.json`.

If `carousel.pattern == "story-arc"` (Thursday):

Same flow as photo-gallery, except:

1. Read `carousel.slide_prompts` — a dict mapping slide index → prompt. Slides labeled `*_photo` need a hero asset; slides labeled `*_cover` or `*_cta` are text-only.
2. Pull only as many fresh images as there are `*_photo` slides (typically 2 for the 4-slide arc). Use the smallest viable count to avoid burning Drive assets.
3. For text-only slides (`*_cover`, `*_cta`), call `generate-design` with **no `asset_ids`** — the brand kit alone produces the layout. For photo slides, attach one asset_id each.
4. Order matters: when calling `merge-designs` to assemble the carousel, insert in slide-prompt order (1_cover → 2_photo → 3_photo → 4_cta).

**One operation per merge call.** This is a Canva MCP constraint, not a stylistic choice. Validated 2026-05-05.

### 6. Move the design into the project folder

Call `Canva:move-item-to-folder`:
- `item_id`: the design ID
- `folder_id`: from `templates.json["canva_workspace"]["folder_id"]`

If the move fails, log a warning but continue — the design still exists, just not filed.

### 7. Verify and tune the design (optional, only if asset wasn't inserted in step 5)

If the generated candidate already used the asset (Canva sometimes does this when `asset_ids` is provided), skip this step.

Otherwise:
1. `Canva:start-editing-transaction` on the new design
2. Inspect the returned elements. Find the largest media (image/video) element on page 1 — that's the hero.
3. `Canva:perform-editing-operations` with one `update_fill` op replacing the hero element's asset with the uploaded asset.
4. Optionally also `find_and_replace_text` if the placeholders from the prompt (e.g. "TECHNIQUE NAME") are still literal — replace them with neutral values like blanks or "—" so Vinz/Jeff can fill them in during the polish pass. **Do not** invent specific technique names; we don't have ground truth.
5. `Canva:commit-editing-transaction`. If anything fails, `cancel-editing-transaction` and log the issue (the un-tuned draft is still usable).

### 8. Save metadata to the repo

Create `content/<week>/<slot>/drafts/draft.json`:

```json
{
  "generated_at": "<ISO datetime>",
  "slot": "<slot ID>",
  "week": "<YYYY-W##>",
  "canva_design_id": "<design_id>",
  "canva_edit_url": "<design_url>",
  "source_asset": {
    "drive_file_id": "<file_id>",
    "drive_file_name": "<file_name>",
    "drive_folder": "<primary_drive_folder name>"
  },
  "generation_mode": "fresh-generate",
  "candidate_count": <number returned>,
  "candidate_picked_index": 0,
  "schedule": {
    "post_day_la": "<weekday>",
    "drop_pst": "<HH:MM>",
    "drop_mla": "<computed Manila time>"
  },
  "status": "draft-ready"
}
```

Also create stub files for the caption-library skill to fill in (step 9b):
- `content/<week>/<slot>/drafts/caption.md` — placeholder, will be overwritten by caption-library
- `content/<week>/<slot>/drafts/hashtags.md` — placeholder, will be overwritten by caption-library

### 8b. Invoke caption-library

After draft.json is saved, invoke the `caption-library` skill for this same slot. This generates the on-brand caption + hashtag set and overwrites the stubs from step 8. See `.claude/skills/caption-library/SKILL.md` for its contract.

If caption-library fails for any reason, log it but continue — the design draft is still usable; Jeff can run caption-library standalone afterward.

### 9. Report back to Jeff

Print a concise summary:

```
✓ <slot> draft ready
  Canva: <design_url>
  Source: <drive_file_name> (from <drive_folder>)
  Drops: <weekday> at <drop_pst> PST (<drop_mla> Manila)
  Files: content/<week>/<slot>/drafts/
```

## Failure modes

| Failure | Action |
|---------|--------|
| Drive folder empty / no recent files | Stop. Tell Jeff to ping Cobrinha/Dani for D-3 assets. |
| Drive file not link-shared | Stop. Tell Jeff to update sharing on the file or folder. |
| `generate-design` returns an error or empty | Retry once with the prompt. If it fails again, stop and tell Jeff. |
| `create-design-from-candidate` fails | Try the next candidate. If all fail, stop. |
| Editing transaction fails | Cancel transaction. Keep the un-tuned draft. Log the issue in `draft.json`. |

## What this skill does NOT do

- **Post to Instagram.** Posting is always Jeff's manual call.
- **Generate captions or hashtags.** That's the `caption-library` skill (step 5, not built yet).
- **Approve the draft.** Cobrinha & Dani do that at D-1 by reviewing the Canva URL.
- **Edit a registered master template.** Skills should never modify the master itself; they only duplicate from it.

## When to switch a slot to registered-master mode

Once Vinz polishes a master template for a slot:
1. Update `templates.json[<slot>].design_id` and `design_url`
2. Run a one-time `register-template-placeholders` flow (not yet built; for now manually note element IDs in `placeholder_element_ids`)
3. Skills will then duplicate that master instead of fresh-generating

The `produce-post` skill should detect `design_id != null` and switch flows automatically (this branch isn't built yet — we'll add it when the first master is ready).
