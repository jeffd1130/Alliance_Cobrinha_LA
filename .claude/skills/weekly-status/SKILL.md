# weekly-status — Alliance Cobrinha LA

Read-only snapshot of every slot in the current ISO week. Shows what's done, what's pending, what's overdue, and what actions are needed next.

---

## Project constants

| Key | Value |
|-----|-------|
| Project dir | `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social` |
| Approval page | `https://jeffd1130.github.io/Alliance_Cobrinha_LA/` |

---

## The 4 slots

| Slot | Day | Format | Drop PST | Drop Manila | Owner (D-3) |
|------|-----|--------|----------|-------------|-------------|
| `01-tue-adult-training-videos` | Tuesday | Adult Reel 9:16 | 7:30 PM | Wed 10:30 AM | Cobrinha & Dani |
| `02-thu-kids-images` | Thursday | Kids 4-slide Carousel | 6:00 PM | Fri 9:00 AM | Cobrinha & Dani |
| `03-fri-adult-images` | Friday | Adult 4-slide Carousel | 8:00 PM | Sat 11:00 AM | Cobrinha & Dani |
| `04-sat-kids-training-videos` | Saturday | Kids Reel 9:16 | 6:30 PM | Sun 9:30 AM | Cobrinha & Dani |

---

## Team

| Person | Role |
|--------|------|
| Cobrinha & Dani | D-3 raw asset drops; D-1 approvers |
| Jeff | D-2 producer (Manila); D-0 publisher |
| Vinz | Design polish |

---

## When to use

- Jeff says "weekly status," "where are we," "what's pending," or "what's left this week"
- Before a team check-in with Cobrinha/Dani
- After `daily-check` to confirm nothing fell through

---

## What it does

1. Compute current ISO week (`YYYY-W##`) and set LA "today" using `America/Los_Angeles`.
2. For each slot in order (01 → 02 → 03 → 04), read `content/<week>/<slot>/` and determine status.
3. Flag urgency based on proximity to drop time.
4. Suggest concrete next actions.

---

## Status levels

| Symbol | Status | Meaning |
|--------|--------|---------|
| `✓` | `approved` | Approved by Cobrinha/Dani — ready to post at D-0 |
| `✓` | `draft-ready` | Draft generated, awaiting D-1 approval |
| `📥` | `assets-dropped` | Raw files in `raw/` but draft not produced yet |
| `⏳` | `awaiting-raw` | No assets yet — Cobrinha/Dani haven't dropped files |
| `—` | `not-started` | Folder doesn't exist yet |
| `✗` | `MISSED` | Past drop time without being approved/posted |

---

## Action suggestions (automatic)

| Condition | Suggestion |
|-----------|------------|
| `awaiting-raw` and drop is ≤3 days away | "Ping Cobrinha & Dani on Telegram for raw assets" |
| `assets-dropped` and Jeff has capacity | "Run produce-post for [slot]" |
| `draft-ready` and D-1 is tomorrow | "Send approval page to Cobrinha & Dani: https://jeffd1130.github.io/Alliance_Cobrinha_LA/" |
| `approved` and D-0 is today | "Schedule/publish at [drop PST] PST ([drop Manila] Manila)" |
| `MISSED` | "Flag to Jeff immediately — post was not published" |

---

## Output format

```
=== Week 2026-W21 ===

Today (Manila):  Sat, May 23  10:00 AM PHT
Today (LA):      Fri, May 22   7:00 PM PDT

Slot                              Status            Drop (PST)          Drop (Manila)         Action
────────────────────────────────────────────────────────────────────────────────────────────────────────
01-tue-adult-training-videos      ✓ approved        Tue 7:30 PM PDT     Wed 10:30 AM Manila   Post today
02-thu-kids-images                ✓ draft-ready     Thu 6:00 PM PDT     Fri  9:00 AM Manila   Send approval link tomorrow
03-fri-adult-images               📥 assets-dropped Fri 8:00 PM PDT     Sat 11:00 AM Manila   Run produce-post
04-sat-kids-training-videos       ⏳ awaiting-raw   Sat 6:30 PM PDT     Sun  9:30 AM Manila   Ping Cobrinha & Dani

Approval page: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
```

If a slot is past its drop time without `approved` status:
```
01-tue-adult-training-videos      ✗ MISSED          Tue 7:30 PM PDT     Wed 10:30 AM Manila   ← investigate
```

---

## What this skill does NOT do

- Does not modify any state. Strictly read-only.
- Does not call Canva. Repo state is the source of truth.
- Does not send Telegram messages — only suggests them with the ready-to-use URL.
