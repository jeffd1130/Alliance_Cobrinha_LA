# Alliance Cobrinha LA — Claude Code Onboarding for Vinz

Welcome to the Alliance Cobrinha LA social media automation system. This repo runs the weekly Instagram production pipeline for Alliance Cobrinha LA, a Brazilian Jiu-Jitsu academy in Los Angeles led by world champion Rubens "Cobrinha" Charles.

## Your role

You are **Vinz**, part of the creative team (Jeff & Vinz). Your job is the **D-2 design pass** — reviewing Canva drafts, polishing layouts, and signing off before Cobrinha/Dani approve at D-1.

## The weekly schedule

| Day (LA) | Audience | Format | Drop (PST) |
|----------|----------|--------|------------|
| Tuesday | Adults | Technical Reel | 7:30 PM |
| Thursday | Kids | Confidence Carousel | 6:00 PM |
| Friday | Adults | Lifestyle Photo | 8:00 PM |
| Saturday | Kids | Summer Camp Reel | 6:30 PM |

PST prime time = next morning Manila (9 AM–12 PM MLA).

## The workflow

| Day | Stage | Who | What |
|-----|-------|-----|------|
| D-3 | Assets | Cobrinha & Dani | Drop photos/videos into `content/<week>/<slot>/raw/` |
| D-2 | Design | **Jeff & Vinz** | Claude generates Canva draft + caption — you review and polish |
| D-1 | Approval | Cobrinha & Dani | Review Canva link, approve or request changes |
| D-0 | Posting | Jeff | Schedule and publish |

## Where to find drafts

All drafts live in:
```
content/2026-W##/
  01-tue-adult-training-videos/drafts/  ← draft.json, caption.md, hashtags.md
  02-thu-kids-images/drafts/
  03-fri-adult-images/drafts/
  04-sat-kids-training-videos/drafts/
```

Each `draft.json` has the Canva edit URL. Open it, polish the design, and let Jeff know when it's ready for D-1.

## Canva folder

All designs live here:
👉 https://www.canva.com/folder/FAHIrnCFIt4

## Brand rules (never break these)

- **Black background** dominant
- **Tan/beige organic blob shapes** in opposing corners
- **Large serif headlines** in white, top-left
- **Sans-serif body text** in white, small, bottom
- **Hero media in a framed window** — not full-bleed
- Brand kit ID: `kAGdeHDeWjQ` — always applied on generation

## Running Claude

Say things like:
- `"daily check"` — find and produce today's drafts
- `"produce Thursday's post"` — run one slot end-to-end
- `"weekly status"` — see what's done vs. pending
- `"redo the caption for Friday"` — regenerate copy only

## GitHub repo

All captions, hashtags, and draft metadata are committed here:
👉 https://github.com/jeffd1130/CobrinhaLA

Questions → Jeff.
