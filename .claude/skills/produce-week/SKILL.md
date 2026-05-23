# produce-week

Produce all remaining drafts for the current ISO week in one shot for **Alliance Cobrinha LA**. Runs `produce-post` for every pending slot, then rebuilds the GitHub Pages approval site and sends the approval message via Telegram.

## Constants

| Key | Value |
|-----|-------|
| Project dir | `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social` |
| Approval site | `https://jeffd1130.github.io/Alliance_Cobrinha_LA/` |
| Telegram bot | @cobrinhaLAdesignsbot → delivers to @jeffd321 |
| Reviewers | Cobrinha & Dani |

## Slot table

| Slot | Day | Format | Drop PST | Drop Manila |
|------|-----|--------|----------|-------------|
| `01-tue-adult-training-videos` | Tuesday | Adult Reel | 7:30 PM | 10:30 AM Wed |
| `02-thu-kids-images` | Thursday | Kids 4-slide Carousel | 6:00 PM | 9:00 AM Fri |
| `03-fri-adult-images` | Friday | Adult 4-slide Carousel | 8:00 PM | 11:00 AM Sat |
| `04-sat-kids-training-videos` | Saturday | Kids Reel | 6:30 PM | 9:30 AM Sun |

## When to use

- Jeff says "produce this week," "make all posts," "run the week," or "do everything"
- Start of D-2 before any drafts exist
- Resuming mid-week when only some slots are done

## Execution order — PARALLEL IS THE DEFAULT

**Step 1 — Pre-flight (sequential):**
1. Compute the current ISO week (e.g. `2026-W21`).
2. For each slot, check whether `content/<week>/<slot>/drafts/draft.json` exists with `status` of `draft-ready` or `approved`. Mark it skip-done.
3. For each slot, check whether the slot's `schedule.date_la` is strictly before today (LA time). Mark it skip-past.

**Step 2 — Run pending slots (PARALLEL):**
Slots 02, 03, and 04 that are not skipped **must be launched as concurrent agents in a single message** — do NOT run them sequentially one after another. Only slot 01, if it needs to run, may run before the parallel batch (it is Tuesday's slot and is typically past by the time this skill is invoked mid-week).

Correct approach:
- Send one message that spawns three agents simultaneously: produce-post for 02, produce-post for 03, produce-post for 04.
- Wait for all three to complete before proceeding.

Incorrect approach (never do this):
- Run slot 02, wait for it to finish, then run slot 03, wait, then run slot 04.

**Step 3 — Merge & rebuild (sequential, after all agents finish):**
1. Invoke `update-approval-page` to rebuild `docs/index.html` and push to GitHub Pages.
2. Send Telegram approval message (see below).
3. Report final summary.

## Telegram approval message

After `update-approval-page` succeeds, send the following via @cobrinhaLAdesignsbot to @jeffd321:

```
Hey! This week's ACL content drafts are ready for review.

Approval link: https://jeffd1130.github.io/Alliance_Cobrinha_LA/

Drop schedule:
  Tue  7:30 PM PST  (10:30 AM Wed Manila)  — Adult Reel
  Thu  6:00 PM PST  (9:00 AM Fri Manila)   — Kids Carousel
  Fri  8:00 PM PST  (11:00 AM Sat Manila)  — Adult Carousel
  Sat  6:30 PM PST  (9:30 AM Sun Manila)   — Kids Reel

Please approve or leave feedback before D-1. Thank you!
```

Adjust the drop schedule lines to only include slots that were actually produced this run (omit skipped/past slots).

## Output format

```
=== W21 Production Run ===

01 · Tue Adult Reel      — ⏭ skipped (past)
02 · Thu Kids Carousel   — ✓ draft ready: https://www.canva.com/d/...   [parallel]
03 · Fri Adult Carousel  — ✓ draft ready: https://www.canva.com/d/...   [parallel]
04 · Sat Kids Reel       — ✓ draft ready: https://www.canva.com/d/...   [parallel]

Approval site updated: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
Telegram sent to @jeffd321 via @cobrinhaLAdesignsbot.

Drop times (produced slots only):
  Thu  6:00 PM PST  (9:00 AM Fri Manila)
  Fri  8:00 PM PST  (11:00 AM Sat Manila)
  Sat  6:30 PM PST  (9:30 AM Sun Manila)
```

## Failure handling

- If any slot's `produce-post` fails, log it, continue the other parallel slots, and flag it in the final summary.
- If `update-approval-page` fails, log it but don't abort — the individual Canva URLs are already in the `draft.json` files. Send the Telegram message with whatever URLs are available.
- If the Telegram send fails, log it and include the approval URL in the final summary so Jeff can share it manually.
- Never abort the whole run because one slot fails.

## What this skill does NOT do

- Post to Instagram. Posting is always Jeff's call.
- Re-run approved posts. Once a slot is `approved`, it's locked unless Jeff explicitly says "redo."
