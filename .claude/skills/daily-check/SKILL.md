# daily-check

Run once per day (manually each morning Manila time, or by a scheduled task). Finds posts due in the next 24 hours (LA time) and produces drafts for them. This is what delivers the "1 day prior, design ready for review" promise.

## Project

**Directory:** `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social`

**Team:**
- Jeff — operator, runs this skill each morning from Manila
- Cobrinha & Dani — approvers via Telegram
- Vinz — design (uploads assets to Drive)

## When to use

- Jeff runs this each morning Manila time
- A cron / scheduled task invokes it
- Jeff says "what's due tomorrow," "any drafts to make," or "morning check"

## Schedule (hardcoded — do not read from templates.json)

| LA Day | Slot ID | Drop PST | Manila drop time | Best Manila run |
|--------|---------|----------|-----------------|-----------------|
| Tuesday | `01-tue-adult-training-videos` | 7:30 PM PST | 10:30 AM Wed Manila | Tue morning Manila |
| Thursday | `02-thu-kids-images` | 6:00 PM PST | 9:00 AM Fri Manila | Thu morning Manila |
| Friday | `03-fri-adult-images` | 8:00 PM PST | 11:00 AM Sat Manila | Fri morning Manila |
| Saturday | `04-sat-kids-training-videos` | 6:30 PM PST | 9:30 AM Sun Manila | Sat morning Manila |

**Timezone note:** Jeff runs this from Manila (UTC+8). Manila is 15–16 hours ahead of LA (America/Los_Angeles, UTC-7 DST / UTC-8 standard). Always determine LA day-of-week using `America/Los_Angeles` — never Manila time. Running at Manila ~9 AM gives drafts a full LA day to be reviewed before posting.

## What it does

1. **Determines the LA day-of-week for tomorrow.** Use the `America/Los_Angeles` timezone. The schedule above is LA-anchored.
2. **Finds matching slots.** Cross-reference tomorrow's LA weekday against the hardcoded schedule table above.
3. **For each matching slot:**
   - Skip if a draft already exists (`content/<current-week>/<slot>/drafts/draft.json` is present and `status` is `draft-ready` or `approved`). Tell Jeff: "`<slot-id>` already has a draft ready — skipping. Use produce-post directly to regenerate."
   - Otherwise invoke `produce-post` for that slot.
4. **After all drafts are produced**, invoke `update-approval-page` to rebuild `docs/index.html` and push to GitHub Pages.
5. **Send Telegram notification.** After `update-approval-page` completes, send an approval message to **@jeffd321** via **@cobrinhaLAdesignsbot** with:
   - The GitHub Pages approval URL: `https://jeffd1130.github.io/Alliance_Cobrinha_LA/`
   - That day's drop time in **both PST and Manila time** (use the table above)
   - A short summary of what was produced (slot name, Canva URL)

   Example message:
   ```
   ✅ Alliance Cobrinha LA — approval ready

   Slot: Adult Training Videos (Tuesday)
   Drop: 7:30 PM PST / 10:30 AM Wed Manila

   Review here: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
   ```

6. **Reports back to Jeff.** Summary of every draft made or skipped, Canva URLs, and the approval page link.

## Edge cases

- **Tomorrow is Sunday or Monday** — no scheduled posts. Tell Jeff: "Nothing on the schedule for tomorrow (LA). Sleep in."
- **Two posts on consecutive days** — handled naturally; this skill only produces tomorrow's. The next day's run will pick up the following post.
- **Friday–Saturday window** — if Jeff runs this Friday morning Manila (Thursday evening LA), it finds Thursday LA's post (Kids Images). If he runs it Saturday morning Manila (Friday evening LA), it picks up Friday LA's post (Adult Images).
- **Stale Drive folder** — if `produce-post` fails because the Drive folder has no fresh assets, surface that prominently to Jeff so he can ping Vinz.
- **Telegram failure** — if the bot message fails, log the error but do not block the overall run. Report the failure in the final summary so Jeff can send manually.

## Output format

```
=== Daily Check — <date> Manila ===

LA tomorrow: <weekday>, <date>
Drop time: <PST> / <Manila>

Slots due:
  ✓ <slot-id> — draft ready: <canva-url>
  ⚠ <slot-id> — already has draft, skipped
  ✗ <slot-id> — failed: <reason>

Approval page: https://jeffd1130.github.io/Alliance_Cobrinha_LA/
Telegram: sent to @jeffd321 via @cobrinhaLAdesignsbot

Next checkpoint: <date> Manila (for <next-LA-day> drop)
```

## Automation hookup

To run this on a schedule, wrap a Claude Code call in a system cron entry. Example crontab line for 9 AM Manila daily:

```
0 9 * * * cd /Users/jeff/Documents/Claude/alliance-cobrinha-la-social && claude --skill daily-check
```

(Exact CLI syntax depends on your Claude Code version. Check the Anthropic docs or run `claude --help`.)
