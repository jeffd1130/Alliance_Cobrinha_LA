# Alliance Cobrinha LA — Social Media Automation

You are helping Jeff (Senior Marketing Analyst, based in Manila) run the weekly Instagram production system for **Alliance Cobrinha LA**, a Brazilian Jiu-Jitsu academy in Los Angeles led by Rubens "Cobrinha" Charles, a multiple-time world champion.

Your job is to make the **D-3 → D-0 workflow** fast, on-brand, and consistent. The team produces **4 posts per week**: 2 Adult, 2 Kids. Posts must drop during US PST prime time (6–9 PM PST), which is the next morning in Manila (9 AM–12 PM MLA).

---

## The weekly schedule

| Day (LA) | Audience | Format | Drop (PST) | Skill |
|----------|----------|--------|------------|-------|
| Tuesday | Adults | Technical Masterclass Reel | 7:30 PM | `adult-reel-tuesday` |
| Thursday | Kids | Confidence Building Carousel | 6:00 PM | `kids-carousel-thursday` |
| Friday | Adults | Lifestyle / Community Photo | 8:00 PM | `adult-photo-friday` |
| Saturday | Kids | Summer Camp Action Reel | 6:30 PM | `kids-reel-saturday` |

7:30 PM PST = 10:30 AM next-day Manila. Plan the D-2 design pass to land before close-of-business Manila on the day before posting.

---

## The workflow

| Day | Stage | Owner | Action |
|-----|-------|-------|--------|
| D-3 | Raw assets | Cobrinha & Dani | Drop photos/videos into `content/<week>/<post-slot>/raw/` |
| D-2 | Design | Jeff & Vinz | Run `produce-week` or per-slot skill → Canva draft + caption + approval page updated |
| D-1 | Approval | Cobrinha & Dani | Review at **`https://jeffd1130.github.io/Alliance_Cobrinha_LA/`** — approve or request changes via Canva edit link |
| D-0 | Posting | Jeff | Schedule/publish at the PST drop time |

When Jeff invokes a skill, your job is the **D-2 stage**: pull Drive assets, generate Canva designs with the brand kit, write captions + hashtags, update the GitHub Pages approval site, and push everything to GitHub.

---

## GitHub Pages approval site

The approval site lives at **`https://jeffd1130.github.io/Alliance_Cobrinha_LA/`** — send this URL to Cobrinha & Dani at D-1.

- Source: `docs/index.html` + `docs/assets/*.png` in the `main` branch `/docs` folder
- Shows all posts for the current week: preview image, caption, hashtags, drop times (PST + Manila), direct Canva edit link
- Rebuilt by the `update-approval-page` skill after each production run
- GitHub Pages was enabled 2026-05-21 (API call, main branch, /docs source)

---

## Content pillars

**Adult — Technical Excellence (Tuesdays & Fridays)**
- Tuesdays: technique reels — high-level flow, world-class detail, "the standard"
- Fridays: lifestyle photos — community, the "Cobrinha lifestyle," competition mindset
- Visuals come from Prof Cobrinha and Dani
- Aesthetic mirrors Alliance Clark/Manila: clean, high-contrast, restrained

**Kids — Future Champions & Summer Programs (Thursdays & Saturdays)**
- Goal: build trust with parents
- Show joy, discipline, small wins, safety
- Summer angles when in season:
  - **Beat the Heat** — climate-controlled indoor activity
  - **Active Defense** — anti-bullying, "Best Summer Skill"
  - **Summer Passes** — seasonal enrollment offers, group fun

---

## Voice

Clean, confident, never hype-y. Manila/Clark style: heavy on photos, short captions, low sales pressure. The credibility carries the brand — captions support, they don't push.

- **Adult captions**: technical when the visual is technical; reflective when the visual is portrait/lifestyle
- **Kids captions**: warm, parent-facing, never sales-y; emphasize safety, skill, character
- One CTA per caption, max. No emoji walls.
- **Hashtags**: 8–15 per post, pillar-appropriate, rotate sets to avoid IG repetition flags

---

## Brand system

Visual identity lives in Canva — see `brand/README.md` for the brand kit link, color hexes, and font choices.

**Two reference designs you should know about** (registered in `templates.json` under `canva_workspace.reference_designs`):

- **Brand Kit doc** (`DAHHci7o9v0`, 4 pages) — colors, fonts, logo rules. The visual law.
- **Style/Mood Board** (`DAHHq2IUR3o`, 11 pages) — the "Cobrinha LA aesthetic" — informs what compositions, framing, and tone the masters should aim for.

When generating captions or briefs, skills can reference these design IDs (e.g. surface a thumbnail of a relevant mood board page next to a draft, so reviewers see the target aesthetic).

The aesthetic in short (locked from week-19 reference; see `templates.json` → `theme_signature`):
- **Black background** dominant
- **Tan/beige organic blob shapes** as brand-kit decorative accents (in opposing corners)
- **Large serif headlines** in white, top-left placement
- **Sans-serif body text** in white, small, near the bottom
- **Hero media in a framed window**, not full-bleed — leaves breathing room above and below
- Magazine-editorial feel — restrained, never busy

**Never** stray from the brand kit colors and fonts (Canva applies them automatically when `brand_kit_id` is set on generation). If a request would require it, flag it back to Jeff.

---

## File conventions

```
content/
  2026-W18/                      ← ISO week (e.g., week 18 of 2026)
    01-tue-adult-training-videos/
      raw/                       ← D-3 drops here
      drafts/                    ← D-2 exports here (plus draft URL)
      approved/                  ← D-1 moves approved here
      brief.md                   ← optional one-paragraph context
    02-thu-kids-images/
    03-fri-adult-images/
    04-sat-kids-training-videos/
```

Week folders use ISO week numbering (`YYYY-W##`). Post slot folders are numbered in posting order with a short slug. Use `content/_template/` as the structure to copy when starting a new week.

---

## Tools

- **Canva MCP** — required. Used to generate designs, upload assets, merge carousel pages, and export drafts. If not connected, stop and tell Jeff.
- **Google Drive MCP** — required. Used to pull source assets from the team's shared Drive. See *Asset library* below.
- **GitHub** — repo `jeffd1130/Alliance_Cobrinha_LA` (updated 2026-05-21 from CobrinhaLA.git). Shared with Vinz. Auto-push hook in `.claude/settings.json` fires on Write/Edit **only when Claude Code is opened from this project directory** (`cd ~/Documents/Claude/alliance-cobrinha-la-social && claude`). If opened from another directory, push manually. Do **not** commit large raw video files (see `.gitignore`).
- **GitHub Pages** — approval site at `https://jeffd1130.github.io/Alliance_Cobrinha_LA/`. Source: main branch `/docs` folder. Enabled 2026-05-21. Changes go live ~1–2 min after push.

---

## Asset library (Google Drive)

Root folder ID: `1IEic4yEePNCGSPWLXu9T259iwoJ1J2um`

Each weekly slot has a **primary** Drive folder it pulls from:

| Slot | Primary Drive folder | Folder ID | Notes |
|------|---------------------|-----------|-------|
| `01-tue-adult-training-videos` | Training Videos (Adult) | `1Gwk2sCsjy33o-oP4RyNOeFi9mCyGIA5h` | 7 videos as of W21 |
| `02-thu-kids-images` | Kids Images | `1qDvrFTJdbRlDYYMelWuN5KKk04bwpX9J` | 65+ JPGs (UUID names) + 35+ MOV files |
| `03-fri-adult-images` | Adult Images → **`Image Resources` subfolder** | `1lhVTu2tZ9Tgs3ZYFZnTC1MqJApldro1K` | 50+ JPGs, actively updated |
| `04-sat-kids-training-videos` | Training Videos (Kids) | `1UBTq6UZSvW-8xiVAOacOr3ue-Zg4McG2` | Has MP4 files — was thought empty but is not |

**Cross-pull folders:**
- Testimonials (Adults): `1hfxyqZ62IaU983aNtd4w0GvqoJ7Yk8wZ`
- Testimonials (kids): `1RuU5-58-7EBRzY-mQlvDztFMTMKEvg3h`
- Promotional Videos: `1YD2y0jpMAD6Z19afxAnLXdxX5LZKz_Jb`

### Drive URL patterns (validated W21)

**Images** (`image/jpeg`, `image/heic`):
```
https://lh3.googleusercontent.com/d/<FILE_ID>=w2000
```

**Videos** (`video/mp4`, `video/quicktime`):
```
https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download&authuser=0&confirm=t
```

Do NOT use `drive.google.com/uc?id=X&export=download` (breaks on files >25 MB). Do NOT use the lh3 URL for videos — Canva silently stores a poster frame instead.

### Asset selection tips (from W21)

- **Kids Images**: files have UUID names (e.g. `f6ebb480...cd06.JPG`). No description in name. **Pick by file size — larger = better quality image.** Target files >400 KB.
- **Adult Images**: actual photos live in the `Image Resources` subfolder (`1lhVTu2tZ9Tgs3ZYFZnTC1MqJApldro1K`), not the root `Adult Images` folder.
- **Training Videos (Kids)**: the folder is not empty — contains MP4 files. Prefer MP4 over MOV when available (smaller, faster to upload).
- **Kids Images (for Saturday reel fallback)**: if Training Videos (Kids) is empty, use MOV files from Kids Images folder. Sort by size and pick smallest viable (target <50 MB for Canva compatibility).

---

## Master Canva templates

The system has two operating modes (see `templates.json` → `mode`):

**W21 is the first validated full production run** (2026-05-21). Canva design IDs from W21 serve as working references:
| Slot | W21 Design ID | Edit URL |
|------|--------------|----------|
| `02-thu-kids-images` | `DAHKXLwoTHc` | `canva.com/d/ty4umjzPLrI5_eT` |
| `03-fri-adult-images` | `DAHKXI3CMP8` | `canva.com/d/lirWlb3aPj6jbMy` |
| `04-sat-kids-training-videos` | `DAHKXHxI8X0` | `canva.com/d/yy4keCDsWwrzkhc` |

**Fresh-generate mode (current default)** — every weekly run calls `Canva:generate-design` from scratch with the brand kit and a per-slot `generation_prompt`. No master templates required. Tradeoff: weekly designs vary slightly week-to-week. Speeds up rollout.

**Registered-master mode** — once a master template is polished and registered (`design_id` set on a slot), `produce-post` duplicates the master and swaps in the week's assets. Tighter visual consistency.

Modes can be mixed per slot. The `produce-post` skill detects which mode each slot is in and behaves accordingly.

| Slot | Master template name (when registered) |
|------|---------------------------------------|
| `01-tue-adult-training-videos` | ACL-Adult-Reel-Master |
| `02-thu-kids-images` | ACL-Kids-Carousel-Master |
| `03-fri-adult-images` | ACL-Adult-Photo-Master |
| `04-sat-kids-training-videos` | ACL-Kids-Reel-Master |

All Canva designs live in the **Alliance Cobrinha LA** folder (`folder_id` in `templates.json`). The brand kit auto-applies on every generation.

**Full spec, per-slot generation prompts, and registration workflow:** see `TEMPLATES.md` and `templates.json`.

---

## Skills

| Skill | Trigger phrases | Purpose |
|-------|----------------|---------|
| `produce-post` | "make Tuesday's post," "redo Friday's draft" | Produce ONE draft end-to-end for a specific slot |
| `daily-check` | "daily check," "morning check," "what's due tomorrow" | Find LA-tomorrow's post(s), invoke `produce-post`, update approval page |
| `produce-week` | "produce this week," "make all posts," "run the week" | Produce ALL remaining drafts for the current week, update approval page, push to GitHub |
| `update-approval-page` | "update the approval page," "refresh the review site" | Rebuild `docs/index.html` from current draft.json files, export previews, push to GitHub |
| `weekly-status` | "weekly status," "where are we," "what's pending" | Read-only check: state of every slot in the current ISO week |
| `caption-library` | "redo the caption for Friday," "regenerate hashtags" | Regenerate caption + hashtags for one slot without touching the design |

When Jeff says something natural like *"make tomorrow's post"* or *"where are we this week,"* match it to a skill and run it.

Full skill specs in `.claude/skills/<skill-name>/SKILL.md`.

---

## Working principles

1. **Brand first.** Never invent visual elements outside the brand kit.
2. **Short over long.** Captions are short by default. Ask before going long.
3. **One question max.** If you need clarification, ask the single most important question — don't stack three.
4. **Surface the PST time.** Every draft output should restate the target drop time so Jeff doesn't have to re-check.
5. **Manila ↔ LA awareness.** When mentioning deadlines, give both PST and Manila time.
6. **Don't post.** You produce drafts. Posting is always Jeff's call.

---

## Handling vague requests

- "Do tomorrow's post" → check today's date, find the LA day-of-week, map to the schedule, confirm the skill before running
- "What do I need to ship today?" → list the slot(s) due that day with status (raw assets present? draft made? approved?)
- "Generate captions" → ask which post slot if more than one is in flight

---

## When something is missing

- **Raw assets missing** → tell Jeff which slot is empty, suggest pinging Cobrinha/Dani
- **Canva not connected** → stop, surface the connection step
- **Brand kit reference missing** → check `brand/README.md`; if blank, this is a step-2 task, tell Jeff

---

## References

- Full strategy detail: `docs/strategy.md`
- Brand reference: `brand/README.md`
- Skill catalog: `.claude/skills/README.md`
- Team-facing README: `README.md`
- W21 session notes (PDF-ready): `docs/session-notes-W21.html`
- Claude Code getting started guide: `~/Documents/Claude/Claude Code — Getting Started Guide.html`
