# produce-week

Produce all remaining drafts for the current ISO week in one shot: runs `produce-post` for every slot that doesn't already have an approved or ready draft, then rebuilds the GitHub Pages approval site via `update-approval-page`.

## When to use

- Jeff says "produce this week," "make all posts," "run the week," or "do everything"
- Start of D-2 before any drafts exist
- Resuming mid-week when only some slots are done

## What it does

1. **Compute the current ISO week** (e.g. `2026-W21`).
2. **Check each slot in posting order** (01 → 02 → 03 → 04):
   - If `content/<week>/<slot>/drafts/draft.json` exists and `status` is `draft-ready` or `approved`, skip it — tell Jeff it already has a draft.
   - Otherwise invoke `produce-post` for that slot.
3. **Skip past slots.** If a slot's `schedule.date_la` is strictly before today (LA time), skip it with a note: "Tuesday was yesterday — skipping."
4. **After all slots are done**, invoke `update-approval-page` to rebuild `docs/index.html` and push to GitHub.
5. **Report a final summary.**

## Output format

```
=== W21 Production Run ===

01 · Tue Adult Reel    — ⏭ skipped (past)
02 · Thu Kids Carousel — ✓ draft ready: https://www.canva.com/d/...
03 · Fri Adult Photos  — ✓ draft ready: https://www.canva.com/d/...
04 · Sat Kids Reel     — ✓ draft ready: https://www.canva.com/d/...

Approval site updated: https://jeffd1130.github.io/Alliance_Cobrinha_LA/

Send this link to Cobrinha & Dani for D-1 review.
Drop times:
  Thu  6:00 PM PST  (9:00 AM Fri Manila)
  Fri  8:00 PM PST  (11:00 AM Sat Manila)
  Sat  6:30 PM PST  (9:30 AM Sun Manila)
```

## Failure handling

- If any slot's `produce-post` fails, log it, continue to the next slot, and flag it in the final summary.
- If `update-approval-page` fails, log it but don't fail the whole run — the individual Canva URLs are already in the draft.json files.
- Never abort the whole run because one slot fails.

## What this skill does NOT do

- Post to Instagram. Posting is always Jeff's call.
- Re-run approved posts. Once a slot is `approved`, it's locked unless Jeff explicitly says "redo."
