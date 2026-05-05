# Handoff

> Single document for the next assistant (human or AI) picking up this project. Captures state, decisions, live test results, and gotchas that aren't obvious from reading the code.

---

## TL;DR

A weekly Instagram production system for **Alliance Cobrinha LA** (Brazilian Jiu-Jitsu academy, Los Angeles). Runs as Claude Code skills against a Canva MCP and a Google Drive MCP. The user is **Jeff**, a Senior Marketing Analyst in Manila.

**The promise:** every morning Manila time, Jeff says *"daily check"* — Claude Code finds tomorrow's LA-scheduled Instagram post, pulls a fresh asset from the right Drive folder, generates a Canva design with the brand kit applied, generates an on-brand caption + hashtag set, and saves the draft URL + copy to the repo. Drafts ready ~24 hours before posting for re-hash, edit, and Cobrinha/Dani approval.

**Current state: operational.** End-to-end test run succeeded. One sample draft is live in Canva. The system runs in *fresh-generate mode* — every run calls `Canva:generate-design` from scratch with the brand kit. Jeff explicitly chose this over registered-master mode after weighing the tradeoffs.

---

## Repo at a glance

```
alliance-cobrinha-la-social/
├── CLAUDE.md                       ← Strategy, voice, schedule, workflow encoded
├── README.md                       ← Team-facing quick start
├── TEMPLATES.md                    ← (legacy from registered-master plan; still useful as spec)
├── templates.json                  ← THE config: mode, theme_signature, slot bindings, generation_prompts
├── .gitignore
├── .claude/skills/
│   ├── README.md
│   ├── produce-post/SKILL.md       ← Workhorse (Drive → Canva → draft)
│   ├── daily-check/SKILL.md        ← Morning entry point
│   ├── weekly-status/SKILL.md      ← Read-only state check
│   └── caption-library/SKILL.md    ← Caption + hashtags per pillar
├── brand/
│   ├── README.md                   ← Canva brand kit + mood board references, Drive folder
│   └── caption-pool.json           ← Voice principles + hashtag pool (editable)
├── content/
│   ├── README.md
│   ├── _template/                  ← Pre-named week template, copy each Monday
│   │   ├── 01-tue-adult-training-videos/{raw,drafts,approved,brief.md}
│   │   ├── 02-thu-kids-images/...
│   │   ├── 03-fri-adult-images/...
│   │   └── 04-sat-kids-training-videos/...
│   └── 2026-W19/                   ← Live test week with one real draft in Friday slot
└── docs/strategy.md                ← The original PDF strategy in markdown
```

---

## The 4 weekly slots

| Slot folder | LA day | Content type | Drop (PST) | Drive source |
|-------------|--------|--------------|------------|--------------|
| `01-tue-adult-training-videos` | Tuesday | Technical Reel | 7:30 PM | Training Videos (Adult) |
| `02-thu-kids-images` | Thursday | Confidence Carousel | 6:00 PM | Kids Images |
| `03-fri-adult-images` | Friday | Lifestyle Photo | 8:00 PM | Adult Images |
| `04-sat-kids-training-videos` | Saturday | Summer Camp Reel | 6:30 PM | Training Videos (Kids) |

PST drop times = next-morning Manila. Run `daily-check` at 9 AM Manila → drafts ready before LA wakes up.

---

## Key decisions made (chronological)

1. **GitHub-first repo** — Vinz has access; not local-only.
2. **Slot folders renamed to match Drive categories** — was `01-tue-adult-reel` etc., became `01-tue-adult-training-videos` etc. The folder name now tells you which Drive subfolder is its primary source.
3. **Drive structure is asset-library, not per-post** — Cobrinha/Dani pile content into 7 categorized folders (Adult Images, Kids Images, Training Videos × 2, Testimonials × 2, Promotional Videos). Slots pull from primary folders; testimonials and promo videos are cross-pulls available to any slot.
4. **Two Canva designs registered as references, not templates:**
   - `DAHHci7o9v0` — *Cobrinha LA Brand Kit* (4 pages, visual brand documentation)
   - `DAHHq2IUR3o` — *Cobrinha LA* (11 pages, mood/style board)
5. **Operating mode locked to `fresh-generate`** — see *Why no master templates* below.
6. **Theme signature locked from the W19 Friday test draft** (`DAHIsVAwhso`) — Jeff approved that look as the target. Each slot's `generation_prompt` now references this layout pattern explicitly.

---

## Live test results

**Two slots validated end-to-end against real Canva + Drive APIs:**

**Friday Adult Photo (image pipeline)** — W19, 2026-05-04
- Drive search → newest JPG picked → uploaded via `lh3.googleusercontent.com` → generated → moved to folder → caption + hashtags
- Live: https://www.canva.com/d/IUiSQlPvrzmvNFE
- Caption: *"The work between the highlights." — Alliance Cobrinha LA*
- 14 hashtags, week-19 rotation

**Tuesday Adult Reel (video pipeline)** — W19, 2026-05-05
- Drive search → newest video picked → uploaded via `drive.usercontent.google.com/download` with `confirm=t` → generated with video asset → moved to folder → caption + hashtags
- Live: https://www.canva.com/d/TzMzmJ1YD1MRsnm
- Source: 25 MB MOV, 1080×1920, 15 sec
- Caption: *"It's not the entry. It's the angle after the entry." — Alliance Cobrinha LA*
- Pattern also validated on 91 MB MP4 (test 5 in the URL strategy work)

The remaining two slots — Thursday Kids Carousel (image) and Saturday Kids Reel (video) — use the same proven URL patterns and skill flow. Neither is expected to surface new failure modes.

---

## Why no master templates (`fresh-generate` mode)

I originally planned a *registered-master* mode where 4 polished Canva templates would get duplicated each week with media swapped in. We didn't pursue it because:

1. **The Canva MCP doesn't expose a duplicate-design tool.** No `copy_design` or equivalent. Workarounds (export → re-import roundtrip, manual one-time duplication into N copies) add complexity and aren't tested.
2. **Page-by-page mapping is manual.** The mood board (`DAHHq2IUR3o`) has 11 pages but no clean 4-slot mapping; we'd need Jeff to tell us "Tue = page 5, Thu = page 2..." etc.
3. **Per-slot element ID discovery would require a one-time inspection pass per master.**

Jeff explicitly chose `fresh-generate` after weighing this. Revisit only if multiple weeks of fresh-generate output are visibly inconsistent. The brand kit (Canva-native, ID `kAGdeHDeWjQ`) auto-applies design tokens on every generation, which is doing most of the consistency work.

---

## Gotchas and quirks (the stuff you only learn by running it)

### 1. Brand kit reality ≠ original strategy PDF

The original strategy deck described the brand as "black + neon green/lime + bold sans-serif." The actual Canva brand kit produces **black + tan/beige organic blob shapes + serif headlines + sans-serif body**. Canva's brand kit overrides prompt-level color/font instructions, which is correct behavior. `CLAUDE.md` and `templates.json.theme_signature` were updated to match what the brand kit actually outputs (after the W19 test). If you ever fight the brand kit in a prompt, the brand kit wins — match it instead.

### 2. Google Drive URL pattern for Canva uploads

`Canva:upload-asset-from-url` cannot fetch from `https://drive.google.com/uc?id=<id>&export=download` for either images or videos — Google serves a download-confirmation interstitial page for that endpoint, returning non-200. **Use these patterns instead, by mime type:**

**Images:**
```
https://lh3.googleusercontent.com/d/<file_id>=w2000
```
Google's content host. The `=w2000` tail asks for a 2000-px-wide rendition. Image-only — DO NOT use for videos: Canva will silently store a poster-frame **image** instead of the video (you'll see `"type": "image"` in the response).

**Videos** (validated 2026-05-05 on 25 MB MOV and 91 MB MP4):
```
https://drive.usercontent.google.com/download?id=<file_id>&export=download&authuser=0&confirm=t
```
The newer Drive download endpoint. The `confirm=t` parameter bypasses the virus-scan interstitial that kicks in for files >25 MB. Returns `"type": "video"` with `width`, `height`, `duration` metadata.

### 3. Drive search query — `trashed` is NOT supported

The Drive MCP rejects `parentId = 'X' and trashed = false` with "Unsupported query field: trashed". Just use `parentId = 'X'` and accept that trashed files might appear (rare in practice).

### 4. Canva's editing API throws server errors on the mood board

`Canva:start-editing-transaction` against `DAHHq2IUR3o` (the 11-page mood board) returns a 500-class server error. Worked fine on `DAHIsVAwhso` (the W19 Friday draft). Reason unknown; possibly design size or shared-state related. If you need to inspect the mood board, retry later or use `get-design-pages` for thumbnails instead.

### 5. Canva thumbnails won't render in Claude widgets

The widget sandbox blocks `document-export.canva.com` URLs. If you need Jeff to look at design thumbnails, ask him to open the Canva URL in his browser tab — don't try to render them in a `show_widget` call.

### 6. Repo state lives in `templates.json` and `caption-pool.json`

Skills are mostly stateless — they read these two files at runtime. Editing them changes behavior immediately, no code changes needed:
- `templates.json.mode` — `fresh-generate` or (eventually) `registered-master`
- `templates.json.templates.<slot>.generation_prompt` — what gets sent to `Canva:generate-design`
- `templates.json.theme_signature` — locked visual pattern (documented; prompts reference it)
- `brand/caption-pool.json.pillars.<pillar>.tone_notes` and `examples` — caption voice
- `brand/caption-pool.json.hashtags.*` — hashtag pools and rotation pattern

### 7. Brief.md is the steering wheel

When Jeff (or Cobrinha/Dani) want to influence a specific week's draft — "this is Lucas's first stripe ceremony, lean into that" — they add it to `content/<week>/<slot>/brief.md`. `caption-library` treats `brief.md` as authoritative.

---

## What's left to build

- **Step 7 — `post-approve` skill** — moves draft → approved status, sends a reminder at the PST drop time, helps Jeff prep the actual IG post (copy caption + hashtags to clipboard, etc.). Stubbed, not built.
- **Cron / scheduler hookup** — to make `daily-check` truly automatic instead of Jeff running it manually each morning. Currently a one-line wrapper around `claude code --skill daily-check`; deployment depends on Jeff's OS and where this runs.
- **Performance feedback loop** — once the system has a few weeks of posts live, there's no automated ingestion of which posts performed well. Eventually worth: a skill that pulls IG insights and tunes the caption pool / generation prompts based on what worked.

## What's been validated end-to-end

- ✅ Image slot pipeline (Friday Adult Photo, W19, draft `DAHIsVAwhso`)
- ✅ Video / reel slot pipeline (Tuesday Adult Reel, W19, draft `DAHIxqfehyI`) — validated on 25 MB MOV and 91 MB MP4
- ✅ Caption + hashtag generation (both test drafts have full caption.md and hashtags.md)
- ⬜ `daily-check` running end-to-end as a single command (the components work; needs one full no-touch test)
- ⬜ `post-approve` flow (not built)

---

## How to onboard quickly

1. **Read this file** (you're here).
2. **Read `CLAUDE.md`** — strategy, voice, schedule, workflow, file conventions.
3. **Skim `templates.json`** — gives you the slot definitions, the locked theme signature, and the per-slot generation prompts in one place.
4. **Read `produce-post/SKILL.md`** — that's the workhorse. The other skills are simpler.
5. **Look at `content/2026-W19/03-fri-adult-images/drafts/`** — see what a successful run produces.

If you're an AI assistant: when Jeff talks naturally ("make tomorrow's post," "where are we this week," "redo Friday's caption"), match to the right skill via `CLAUDE.md`'s skill catalog and the schedule table.

---

## Connected MCPs (required)

| MCP | Used for | If missing |
|-----|----------|-----------|
| Canva | Brand kit, generate-design, upload-asset, move to folder, editing | Stop. Tell Jeff to connect it. |
| Google Drive | Search files, get permissions, get metadata | Stop. Tell Jeff to connect it. |
| GitHub | Repo collaboration with Vinz | Optional for skills; required for Vinz's workflow |

---

## Important IDs (for quick reference)

| Thing | ID |
|-------|-----|
| Canva brand kit | `kAGdeHDeWjQ` |
| Canva workspace folder | `FAHIrnCFIt4` |
| Brand kit doc (reference) | `DAHHci7o9v0` |
| Mood board (reference) | `DAHHq2IUR3o` |
| W19 Friday draft (theme reference) | `DAHIsVAwhso` |
| Drive root folder | `1IEic4yEePNCGSPWLXu9T259iwoJ1J2um` |

All IDs are also in `templates.json` and `brand/README.md`.

---

## Voice in one paragraph

Clean, confident, never hype-y. Manila/Clark style: heavy on photos, short captions, low sales pressure. Adult content reads technical when the visual is technical, reflective when it's portrait/lifestyle. Kids content is warm and parent-facing — never sales-y, never cartoonish. One CTA per post max. No emoji walls. Sentence case. End with a quiet sign-off, not a hard CTA. *"One team. One standard."* is a brand anchor that always lands.

---

*Last updated: 2026-05-04 — handoff prepared at end of step 5.1 (theme lock-in).*
