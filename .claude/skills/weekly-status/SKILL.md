# weekly-status

Show the state of every slot in the current ISO week. Useful for "what's pending," "where are we," or before a leadership check-in.

## When to use

- Jeff says "weekly status," "what's the state of the week," "where are we"
- After running `daily-check` to confirm nothing fell through

## What it does

1. Compute current ISO week as `YYYY-W##`.
2. Read every slot folder under `content/<week>/`.
3. For each slot, determine status:
   - **No folder** — week not set up yet
   - **`raw/` empty + no draft** — awaiting D-3 assets from Cobrinha/Dani
   - **`raw/` has files but no `drafts/draft.json`** — assets dropped, draft not yet generated
   - **`drafts/draft.json` exists** — draft ready (read its `status` field: `draft-ready`, `approved`, etc.)
   - **`approved/` has files** — approved, awaiting post
4. Compare against `schedule.drop_pst` to flag what's due in the next 24/48 hours.

## Output format

```
=== Week 2026-W18 ===

Today (Manila):  Wed, May 6
Today (LA):      Tue, May 5

Slot                              Status            Drops
─────────────────────────────────────────────────────────────
01-tue-adult-training-videos      ✓ approved        Tue 7:30 PM PST   ← TODAY
02-thu-kids-images                ✓ draft ready     Thu 6:00 PM PST
03-fri-adult-images               ⏳ awaiting raw   Fri 8:00 PM PST   ← need assets
04-sat-kids-training-videos       — not started     Sat 6:30 PM PST
```

If a slot is past its drop time without being approved/posted, flag it: `✗ MISSED — Tue 7:30 PM PST`.

## What this skill does NOT do

- It does not modify state. Read-only check.
- It does not call Canva. Repo state is the source of truth.
