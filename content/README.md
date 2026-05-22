# Content

One folder per ISO week. One subfolder per post slot.

## How a week gets set up

```bash
# Start of a new week (run from repo root)
cp -r content/_template content/2026-W18
```

Then rename the slot folders inside `2026-W18/` if needed (the template uses generic names).

## Slot folder structure

Each post slot has:

```
01-tue-adult-training-videos/
├── raw/         ← Cobrinha & Dani drop photos/videos here at D-3
├── drafts/      ← D-2 outputs land here (Canva exports, caption.md, hashtags.md, draft URL)
├── approved/    ← When Cobrinha/Dani green-light, files move here at D-1
└── brief.md    ← Optional one-paragraph context from leadership
```

## Naming convention

Slot folders are numbered in posting order with a short slug:

- `01-tue-adult-training-videos`
- `02-thu-kids-images`
- `03-fri-adult-images`
- `04-sat-kids-training-videos`

This keeps the folder listing in posting order and makes the slot identifiable at a glance.

## What lives in `_template/`

A blank week structure, ready to be copied. Don't add real assets here — only structural files.
