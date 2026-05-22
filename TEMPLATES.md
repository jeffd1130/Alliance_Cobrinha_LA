# Master Templates

This project drives 4 weekly Instagram posts off 4 master Canva templates. Skills duplicate a master, swap in this week's media, swap in this week's text, and export a draft.

The bindings live in `templates.json` at the repo root.

---

## How the system works

```
Canva master template          One-time build (Vinz polishes generated starters)
        │
        │  duplicated each week by a skill
        ▼
Per-post draft in Canva        Has this week's media + caption swapped in
        │
        │  exported by the skill
        ▼
Draft URL + assets in repo     Lives in content/<week>/<slot>/drafts/
        │
        │  reviewed by Cobrinha & Dani at D-1
        ▼
Approved → posted at PST drop time
```

---

## The 4 templates

| Slot folder | Template name | Type | Dimensions | Day | Drop (PST) |
|-------------|---------------|------|------------|-----|------------|
| `01-tue-adult-training-videos` | ACL-Adult-Reel-Master | Story/Reel | 1080×1920 | Tue | 7:30 PM |
| `02-thu-kids-images` | ACL-Kids-Carousel-Master | Post (4:5) | 1080×1350 | Thu | 6:00 PM |
| `03-fri-adult-images` | ACL-Adult-Photo-Master | Post (4:5) | 1080×1350 | Fri | 8:00 PM |
| `04-sat-kids-training-videos` | ACL-Kids-Reel-Master | Story/Reel | 1080×1920 | Sat | 6:30 PM |

---

## Spec — what each template must contain

### 1. ACL-Adult-Reel-Master (Tuesdays — Technical Masterclass)

**Visual zones**
- Full-bleed hero zone (top ~75%) for technique video footage
- Top text overlay: technique name (bold sans-serif, lime accent allowed)
- Subtitle below: instructor credit
- Bottom strip: "ALLIANCE COBRINHA LA"
- Logo lockup, bottom right

**Placeholders the skill swaps**
- Media: `hero_video`
- Text: `technique_name`, `instructor_credit`

**Aesthetic**: clean, restrained, high-contrast. Movement carries the frame.

### 2. ACL-Kids-Carousel-Master (Thursdays — Confidence Building)

**Visual zones**
- Hero photo zone (top ~60%)
- Title overlay (warm headline, parent-facing)
- Subtitle line (program/age detail)
- Carousel/swipe indicator on right edge
- Logo lockup, bottom corner

**Placeholders the skill swaps**
- Media: `hero_photo`
- Text: `headline`, `subtitle`

**Aesthetic**: same brand, slightly warmer. Never cartoonish. Parent-trust-building.

### 3. ACL-Adult-Photo-Master (Fridays — Lifestyle / Community)

**Visual zones**
- Full-bleed photo
- ONE minimal text overlay (top OR bottom — not both)
- Logo lockup, very subtle

**Placeholders the skill swaps**
- Media: `hero_photo`
- Text: `overlay_text`

**Aesthetic**: closer to a magazine portrait than an ad. Quiet caption, not a headline.

### 4. ACL-Kids-Reel-Master (Saturdays — Summer Camp Action)

**Visual zones**
- Full-bleed hero zone (top ~75%) for kids action footage
- Top headline overlay (punchy)
- Subtitle line (program detail)
- Bottom CTA strip
- Logo lockup, bottom right
- Optional small lime sun icon near headline

**Placeholders the skill swaps**
- Media: `hero_video`
- Text: `headline`, `cta`

**Aesthetic**: high energy but on-brand. Joy, action, group fun. Never childish.

---

## Workflow: register a polished master

After Vinz finalizes a master in Canva:

1. Open the design in Canva and copy the URL — it looks like `https://www.canva.com/design/DAGXxxxx_xxx/edit`
2. The design ID is the part starting with `D` (e.g. `DAGXxxxx_xxx`)
3. Update `templates.json`:
   ```json
   "01-tue-adult-training-videos": {
     "design_id": "DAGXxxxx_xxx",
     "design_url": "https://www.canva.com/design/DAGXxxxx_xxx/edit",
     ...
   }
   ```
4. The `placeholder_element_ids` block stays `null` until step 4 (skills) — we'll discover element IDs by calling `start-editing-transaction` on the master and noting them down in a one-time pass.
5. Commit and push: `git commit -m "Register ACL-Adult-Reel-Master template ID" && git push`

---

## Workflow: discover placeholder element IDs (one-time, per template)

Once a master is polished and registered:

1. Run the `register-template-placeholders` skill (built in step 4) — it:
   - Calls `Canva:start-editing-transaction` on the master
   - Lists all elements (text + media)
   - Asks Jeff to label which element is which placeholder (e.g. "this text element is `technique_name`")
   - Writes the mapping to `placeholder_element_ids` in `templates.json`
   - Cancels the transaction (no edits made)
2. From then on, the per-post-type skill knows exactly which element to swap.

---

## Where the masters live in Canva

Folder: **Alliance Cobrinha LA** (`https://www.canva.com/folder/FAHIrnCFIt4`)

Brand kit applied: `kAGdeHDeWjQ` (the Alliance Cobrinha brand kit you provided).

---

## When a master is missing or wrong

If a skill runs and finds `design_id == null` for its slot, it:
- Stops
- Tells Jeff the master template hasn't been registered yet
- Points him at this doc to register it

If a skill runs and Canva returns a "design not found" error:
- Stops
- Tells Jeff the registered design_id is invalid
- Asks him to verify in Canva and update `templates.json`
