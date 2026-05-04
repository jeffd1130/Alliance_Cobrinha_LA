# Skills

Claude Code skills for the Alliance Cobrinha LA weekly Instagram program. One per task type.

## Skill catalog

| Skill | Purpose | Built? |
|-------|---------|--------|
| `produce-post` | Workhorse: Drive asset → Canva design → draft URL for one slot | ✅ |
| `daily-check` | Morning entry point: find tomorrow's post(s) and produce them | ✅ |
| `weekly-status` | At-a-glance state of the current week's slots | ✅ |
| `caption-library` | Pillar-aware caption + hashtag generation | ⬜ |
| `post-approve` | Move approved drafts forward, send PST drop reminder | ⬜ |

## Operating mode

This project currently runs in **fresh-generate** mode (see `templates.json` → `mode`). Every weekly run calls `Canva:generate-design` from scratch with the brand kit. No master templates yet.

To switch a slot to **registered-master** mode (tighter visual consistency, faster runs):
1. Polish a master template in Canva (apply the spec from `TEMPLATES.md`)
2. Set the slot's `design_id` and `design_url` in `templates.json`
3. The next time `produce-post` runs that slot, it'll detect the master and duplicate-then-swap instead of fresh-generating

You can mix modes — some slots fresh-generate while others use registered masters.

## How skills are invoked

Inside Claude Code, talk naturally:

- *"Make tomorrow's post"* → `daily-check`
- *"Run Tuesday's reel"* / *"Produce 01-tue-adult-training-videos"* → `produce-post`
- *"Where are we this week"* → `weekly-status`
- *"Regenerate Friday's draft"* → `produce-post` (it overwrites)

Claude Code will read `CLAUDE.md`, identify the right skill, and run it.

## Skill contract

Every skill in this folder has a `SKILL.md` describing:
- When to use it
- Inputs
- Required tools (and what to do if they're missing)
- Step-by-step flow
- Failure modes
- What it explicitly does NOT do

Read the matching `SKILL.md` for full detail before invoking.
