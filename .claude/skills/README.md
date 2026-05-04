# Skills

This folder holds Claude Code skills, one per task type. **Skills are built in step 4** of the rollout — this README documents the pattern they will follow.

## Skill catalog (planned)

| Skill | Purpose | Built? |
|-------|---------|--------|
| `adult-reel-tuesday` | Produce Tuesday's Technical Masterclass Reel | ⬜ |
| `kids-carousel-thursday` | Produce Thursday's Confidence Building Carousel | ⬜ |
| `adult-photo-friday` | Produce Friday's Lifestyle / Community Photo | ⬜ |
| `kids-reel-saturday` | Produce Saturday's Summer Camp Action Reel | ⬜ |
| `weekly-planner` | Set up the next week's content folders, list status | ⬜ |
| `caption-generator` | Standalone caption + hashtag generator (used by all post skills) | ⬜ |

## Skill pattern

Each skill is a folder containing a `SKILL.md`. Per-post-type skills follow this contract:

**Inputs**
- `content/<week>/<post-slot>/raw/` — assets from D-3 drop
- `content/<week>/<post-slot>/brief.md` (optional) — context line(s)
- The matching Canva master template

**Outputs**
- A duplicated Canva design with assets swapped in
- A draft caption (saved to `drafts/caption.md`)
- A hashtag set (saved to `drafts/hashtags.md`)
- A draft URL printed for Cobrinha/Dani's D-1 review

**Side effects**
- Updates the post's status (e.g. via a `status.json` in the slot folder)
- Reminds Jeff of the PST drop time

## How a skill gets invoked

Inside Claude Code, Jeff says something like:

> "Run adult-reel-tuesday for this week"

or:

> "Make tomorrow's post"

The second form makes Claude Code consult the schedule in `CLAUDE.md` and pick the matching skill itself.
