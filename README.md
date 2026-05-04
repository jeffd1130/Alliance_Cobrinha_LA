# Alliance Cobrinha LA — Social Media Automation

Weekly Instagram production system for Alliance Cobrinha LA, run via Claude Code with Canva.

> One team, one standard, global reach.

---

## Who does what

| Role | People | Stage | Action |
|------|--------|-------|--------|
| Leadership | Prof Cobrinha & Dani | D-3, D-1 | Drop raw assets · approve drafts |
| Creative | Jeff & Vinz | D-2 | Run skills, design, edit, caption |
| Operations | Jeff | D-0 | Schedule, post, track |

---

## The week

Four posts per week, all timed to US PST prime time (6–9 PM PST):

| Day (LA) | Audience | Format | Drop (PST) | Drop (MLA) |
|----------|----------|--------|------------|------------|
| Tuesday | Adults | Technical Reel | 7:30 PM | 10:30 AM Wed |
| Thursday | Kids | Confidence Carousel | 6:00 PM | 9:00 AM Fri |
| Friday | Adults | Lifestyle Photo | 8:00 PM | 11:00 AM Sat |
| Saturday | Kids | Summer Camp Reel | 6:30 PM | 9:30 AM Sun |

---

## Quick start

**For Jeff or Vinz running the system:**

1. Clone this repo: `git clone <repo-url>`
2. Open in Claude Code
3. Connect the Canva MCP (one-time): in Claude Code, add the Canva connector
4. Create the week folder: `cp -r content/_template content/2026-W##` (use the ISO week)
5. Wait for Cobrinha/Dani to drop assets in the right `raw/` subfolders
6. Run the skill matching the day, e.g. `Tell Claude: "make tomorrow's post"`
7. Send the draft URL to Cobrinha/Dani for D-1 approval
8. Schedule for the PST drop time

**For Cobrinha or Dani dropping assets:**

- Each week has its own folder under `content/`
- Each post has four subfolders: `raw/`, `drafts/`, `approved/`, plus a `brief.md`
- Drop your photos/videos into the matching `raw/` folder by D-3
- Optional: add one or two lines to `brief.md` to give context (e.g. "this is Lucas after his first stripe")

---

## Folder map

```
.
├── CLAUDE.md                       ← Project context for Claude Code
├── README.md                       ← This file
├── .claude/skills/                 ← Per-task skills (built in step 4)
├── brand/                          ← Brand kit reference (built in step 2)
├── content/
│   ├── _template/                  ← Copy this when starting a new week
│   └── 2026-W##/                   ← One folder per ISO week
│       ├── 01-tue-adult-training-videos/
│       ├── 02-thu-kids-images/
│       ├── 03-fri-adult-images/
│       └── 04-sat-kids-training-videos/
└── docs/
    └── strategy.md                 ← Full strategy reference
```

---

## What's built so far

- ✅ **Step 1 — Project scaffold + CLAUDE.md** (you are here)
- ⬜ Step 2 — Brand reference doc
- ⬜ Step 3 — Canva master templates
- ⬜ Step 4 — Per-post-type skills
- ⬜ Step 5 — Caption + hashtag library
- ⬜ Step 6 — Weekly planner skill
- ⬜ Step 7 — Approval + posting handoff

---

## Notes

- **Large raw videos are gitignored.** Keep them in a shared cloud folder (Drive/Dropbox) and symlink into `raw/`, or commit only smaller reference assets.
- **Posting is always Jeff's call.** Claude Code produces drafts; humans hit publish.
- The strategy in this repo is summarized from the original Alliance Cobrinha LA Instagram strategy deck — full version in `docs/strategy.md`.
