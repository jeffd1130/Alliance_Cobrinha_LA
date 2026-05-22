# daily-check

Run once per day (manually each morning, or by a scheduled task). Finds posts due in the next 24 hours (LA time) and produces drafts for them. This is what delivers the "1 day prior, design ready for re-hash" promise.

## When to use

- Jeff runs this each morning Manila time
- A cron / scheduled task invokes it
- Jeff says "what's due tomorrow," "any drafts to make," or "morning check"

## What it does

1. **Determines the LA day-of-week for tomorrow.** Use the `America/Los_Angeles` timezone, not Manila. The schedule in `templates.json` is LA-anchored.
2. **Finds matching slots.** Reads `templates.json["templates"]` and selects every slot where `schedule.day_la` equals tomorrow's LA weekday.
3. **For each matching slot:**
   - Skip if a draft already exists (`content/<current-week>/<slot>/drafts/draft.json` is present and `status` is `draft-ready` or `approved`). Tell Jeff: "Tuesday already has a draft ready — skipping. Use produce-post directly to regenerate."
   - Otherwise invoke `produce-post` for that slot.
4. **After all drafts are produced**, invoke `update-approval-page` to rebuild `docs/index.html` and push to GitHub.
5. **Reports back.** Summary of every draft made or skipped, Canva URLs, and the approval page link.

## Edge cases

- **Tomorrow is Sunday or Monday** — no scheduled posts. Tell Jeff: "Nothing on the schedule for tomorrow (LA). Sleep in."
- **Two posts on consecutive days** — handled naturally; this skill only produces tomorrow's. The next day's run will pick up the day after.
- **Friday-Saturday window** — if Jeff runs this Friday morning Manila (Thursday evening LA), it should find Friday LA's post (Adult Photo). If he runs it Saturday morning Manila (Friday evening LA), it picks up Saturday LA's Kids Reel.
- **Stale Drive folder** — if `produce-post` fails because the Drive folder has no fresh assets, surface that prominently to Jeff so he can ping Cobrinha & Dani.

## Output format

```
=== Daily Check — <date> Manila ===

LA tomorrow: <weekday>, <date>

Slots due:
  ✓ <slot-id> — draft ready: <canva-url>
  ⚠ <slot-id> — already has draft, skipped
  ✗ <slot-id> — failed: <reason>

Next checkpoint: <date> Manila (for <next-LA-day>)
```

## Schedule reminder

| LA Day | Slot | Drop (PST) | Best Manila run time |
|--------|------|-----------|---------------------|
| Tue | `01-tue-adult-training-videos` | 7:30 PM | Mon morning Manila |
| Thu | `02-thu-kids-images` | 6:00 PM | Wed morning Manila |
| Fri | `03-fri-adult-images` | 8:00 PM | Thu morning Manila |
| Sat | `04-sat-kids-training-videos` | 6:30 PM | Fri morning Manila |

(Manila is 15-16 hours ahead of LA depending on DST, so Manila morning ≈ LA late afternoon/evening of the previous day. Running at Manila ~9 AM gives drafts a full LA day to be reviewed and tweaked before posting.)

## Automation hookup (future)

To run this on a schedule, wrap a Claude Code call in a system cron / Task Scheduler entry. Example crontab line for 9 AM Manila daily:

```
0 9 * * * cd ~/alliance-cobrinha-la-social && claude code --skill daily-check
```

(Exact CLI syntax depends on your Claude Code version. Check the Anthropic docs.)
