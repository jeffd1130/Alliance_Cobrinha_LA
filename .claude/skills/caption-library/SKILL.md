# caption-library — Alliance Cobrinha LA

Generate an on-brand caption and rotated hashtag set for one Alliance Cobrinha LA weekly post. Reads pillar voice rules, uses W21 captions as the validated baseline, and saves output to the slot's `drafts/` folder.

---

## Project constants

| Key | Value |
|-----|-------|
| Project dir | `/Users/jeff/Documents/Claude/alliance-cobrinha-la-social` |
| Caption pool | `brand/caption-pool.json` |

---

## The 4 pillars

| Slot | Pillar | Audience | Tone |
|------|--------|----------|------|
| `01-tue-adult-training-videos` | `adult_technical` | Adults | Technical, world-class detail, Cobrinha's championship pedigree — "the standard" |
| `02-thu-kids-images` | `kids_confidence` | Parents of kids 4–12 | Warm, milestone-focused, safety-first — never sales-y |
| `03-fri-adult-images` | `adult_lifestyle` | Adults | Magazine-quiet, 1–3 lines max, reflective — community and the "Cobrinha lifestyle" |
| `04-sat-kids-training-videos` | `kids_summer` | Parents | Summer Camp angle — Beat the Heat (climate-controlled), Active Defense (anti-bullying), enrollment |

---

## When to use

- Called automatically by `produce-post` after the Canva design is saved
- Jeff says "redo the caption for Friday," "regenerate hashtags for Saturday," or "new caption for Thursday"
- Jeff provides a brief update ("the technique is de la Riva sweep — regenerate caption")

---

## Inputs

- **slot ID** — one of the 4 slots above
- **week** — defaults to current ISO week
- **brief override** (optional) — Jeff's extra context (technique name, event, milestone, etc.)

Also reads:
- `content/<week>/<slot>/brief.md` — context from Cobrinha/Dani; treat as authoritative
- `content/<week>/<slot>/drafts/draft.json` — source asset metadata
- `brand/caption-pool.json` — full voice rules, examples, hashtag pools

---

## Step-by-step

### 1. Resolve context

1. Read `brand/caption-pool.json`. Get the pillar block for this slot.
2. Read `brief.md` if present. If it names a technique, person, or event — use it. Brief overrides everything.
3. Read `draft.json` if present. Pull `drive_assets` filename hints if useful.

### 2. Compose the caption

Use the pillar's `tone_notes` and stay within `length_lines`. Start from the W21 baseline examples below — adapt rather than invent from scratch.

**Alliance Cobrinha LA voice rules (hard):**
- Clean, confident, never hype-y. No "crush it," "beast mode," "grind."
- Sentence case only. Never ALL CAPS.
- One CTA maximum — "DM us" or "Link in bio" only when the brief calls for it. Omitting CTA is fine.
- No emoji walls. One emoji max, only when it genuinely adds to the message.
- Never invent specific facts (names, dates, belt levels, competition results) the brief doesn't supply.
- Short is always better: adult lifestyle = 1–3 lines; kids = 3–5 lines; technical = 2–4 lines.
- If brief is thin, use the W21 baseline as the starting point and adapt minimally — ship a known-on-brand caption rather than over-engineer.

**W21 validated baseline captions (use as voice reference):**

`kids_confidence`:
```
Today she earned her first stripe. Last month she was nervous to step on the mat.

Kids BJJ at Alliance Cobrinha LA — ages 4–12. DM us for a free trial.
```

`adult_lifestyle`:
```
The work between the highlights.

One team. One standard.
```

`kids_summer`:
```
Climate-controlled. Coach-led. The best place for kids when it's 100° outside.

Summer Camp — ages 5–12. DM to enroll.
```

### 3. Compose the hashtag set

**Always include all 5 core tags:**
```
#AllianceCobrinha #AllianceCobrinhaLA #BJJ #JiuJitsu #BrazilianJiuJitsu
```

**Always include 2 location tags:**
```
#LosAngeles #BJJLA
```

**Always include rotating closer:**
```
#TrainSmart
```

**Add 4–6 pillar-specific tags** (rotate using `week_number % pool_length` as start index):

| Pillar | Tag pool |
|--------|----------|
| `kids_confidence` | `#KidsMartialArts` `#BJJKids` `#ConfidenceBuilding` `#AntiBullying` `#KidsJiuJitsu` `#MartialArtsKids` |
| `adult_lifestyle` | `#BJJLove` `#OssBrother` `#LifeOnTheMat` `#BJJEveryDay` `#JiuJitsuLife` `#GrappleLife` |
| `kids_summer` | `#SummerActivities` `#KidsMartialArts` `#FutureChampions` `#ActiveDefense` `#SummerCamp` `#KidsActivities` |
| `adult_technical` | `#BJJTechnique` `#MartialArts` `#Grappling` `#BJJBlackBelt` `#Submission` `#TechnicalBJJ` |

Example rotation for W21: `21 % 6 = 3` → start at index 3, pick 4 tags wrapping around if needed.

Total target: **12–15 hashtags**.

### 4. Save files

Write `content/<week>/<slot>/drafts/caption.md`:
```markdown
<!-- caption-library · <ISO datetime> · Pillar: <pillar> · Brief: <yes/no> -->

<the caption text>
```

Write `content/<week>/<slot>/drafts/hashtags.md`:
```markdown
<!-- caption-library · <ISO datetime> · Mix: 5 core + N pillar + 2 location + 1 rotating = M total -->

#Tag1 #Tag2 #Tag3 ...
```

Update `draft.json` to add:
```json
"caption": "<caption text>",
"hashtags": "<hashtag string>"
```

### 5. Report back

```
✓ 03-fri-adult-images caption ready
  Pillar: adult_lifestyle
  Caption: "The work between the highlights...."
  Hashtags: 12 total
  Files: content/2026-W21/03-fri-adult-images/drafts/caption.md + hashtags.md
```

---

## Failure modes

| Failure | Action |
|---------|--------|
| `brand/caption-pool.json` missing | Stop. Tell Jeff — this file is required. |
| `draft.json` missing | Continue without asset metadata. Use brief + pillar baseline. |
| `brief.md` missing or empty | Continue. Use W21 baseline examples as starting point. |
| Output feels off-brand | Jeff can re-run the skill — it's idempotent and overwrites the previous output. |

---

## What this skill does NOT do

- **Post to Instagram.** Output is saved to disk only.
- **Touch the Canva design.** Caption goes in the IG post, not on the image.
- **Generate visual content.** That's `produce-post`.
- **Guarantee perfection.** It's a strong starting point for Jeff and Vinz to polish.
