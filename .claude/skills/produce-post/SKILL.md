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
   parentId = '<primary_drive_folder_id>' and trashed = false
   ```
   Sort by `modifiedTime` descending (set `pageSize: 5` and pick the first non-folder file). Skip files modified more than 30 days ago — if all are stale, warn Jeff that the Drive folder may be empty or stale.
3. Get the file's metadata. You need a publicly accessible URL — call `get_file_permissions` to confirm. If the file isn't link-shared, **stop** and tell Jeff which file needs to be set to "Anyone with link can view."
4. The public URL is typically `https://drive.google.com/uc?id=<fileId>&export=download` for direct download. Use this for `upload-asset-from-url`.

### 4. Upload the asset to Canva

Call `Canva:upload-asset-from-url` with:
- `url`: the Drive direct-download URL
- `name`: `<slot>-<date>-source` (e.g. `01-tue-adult-training-videos-2026-05-05-source`)

Save the returned `asset_id`.

### 5. Generate the Canva design

Call `Canva:generate-design`:
- `brand_kit_id`: from `templates.json["canva_workspace"]["brand_kit_id"]`
- `design_type`: from the slot config
- `query`: the slot's `generation_prompt`
- `asset_ids`: `[<asset_id from step 4>]` — this asks Canva to insert the real asset directly during generation when possible

Pick the **first candidate** from the response (we're in fresh-generate mode; we trust the brand kit + prompt to land it close enough; Jeff can regenerate if it's off).

### 6. Materialize the candidate

Call `Canva:create-design-from-candidate` with the chosen `candidate_id` and `job_id`. Save the returned `design_id` and `design_url`.

### 7. Move the design into the project folder

Call `Canva:move-item-to-folder`:
- `item_id`: the design ID
- `folder_id`: from `templates.json["canva_workspace"]["folder_id"]`

If the move fails, log a warning but continue — the design still exists, just not filed.

### 8. Verify and tune the design (optional, only if asset wasn't inserted in step 5)

If the generated candidate already used the asset (Canva sometimes does this when `asset_ids` is provided), skip this step.

Otherwise:
1. `Canva:start-editing-transaction` on the new design
2. Inspect the returned elements. Find the largest media (image/video) element on page 1 — that's the hero.
3. `Canva:perform-editing-operations` with one `update_fill` op replacing the hero element's asset with the uploaded asset.
4. Optionally also `find_and_replace_text` if the placeholders from the prompt (e.g. "TECHNIQUE NAME") are still literal — replace them with neutral values like blanks or "—" so Vinz/Jeff can fill them in during the polish pass. **Do not** invent specific technique names; we don't have ground truth.
5. `Canva:commit-editing-transaction`. If anything fails, `cancel-editing-transaction` and log the issue (the un-tuned draft is still usable).

### 9. Save metadata to the repo

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

Also create stub files for step 5 (caption/hashtag library — not built yet):
- `content/<week>/<slot>/drafts/caption.md` — single line: `<!-- caption pending — generate via caption-library skill (not yet built) -->`
- `content/<week>/<slot>/drafts/hashtags.md` — single line: `<!-- hashtags pending — generate via caption-library skill (not yet built) -->`

### 10. Report back to Jeff

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
