# The Bluest Hour

> "In certain latitudes there comes a span of time approaching and following the summer solstice, some weeks in all, when the twilights turn long and blue."
> — Joan Didion, *Blue Nights*

A small place-based app for Godfrey, Illinois that tells you when to start an evening walk so the middle of the walk lands in the deepest part of twilight. The current version also includes a short walk journal and an in-browser sentiment baseline.

## Why this repo matters for Level 2

This repo now functions as the **journal-first exemplar** for the paper phase:

1. [`research-journal.md`](./research-journal.md) shows a believable week-by-week research log.
2. [`PAPER.md`](./PAPER.md) shows a short paper built directly from that journal.
3. [`../bluest-hour-almanac/PAPER.md`](../bluest-hour-almanac/PAPER.md) is the advanced continuation, not the starting target.

The point is to show students a path they can actually copy:

- build something
- keep a journal while you test it
- pull a question, finding, and limitation out of the journal
- turn those into a short paper

## What the current project does

- predicts a 20-minute walk window using evening twilight data for Godfrey, IL
- compares astronomical timing with a local offset based on observation
- lets the user write a short reflection after the walk
- runs a binary sentiment baseline in the browser with Transformers.js

There are two related implementations in this repo family:

- **Static Pages version:** [`bluest-hour/`](.) — single-file HTML/JS version served with GitHub Pages
- **Hugging Face Space:** [`../bluest-hour-space/`](../bluest-hour-space/) — Gradio/Python version

## What students should read first

If you are using Bluest Hour as a course model, open these in order:

1. [`research-journal.md`](./research-journal.md)
2. [`PAPER.md`](./PAPER.md)
3. [`../bluest-hour-almanac/PAPER.md`](../bluest-hour-almanac/PAPER.md) only if you want to see a later, more literary redesign

## Current app summary

The app uses evening twilight data to estimate when the blue hour happens in Godfrey, Illinois. It centers a 20-minute walk around that moment, then lets the user record a short note about the walk. In the static version, the journaling layer uses Transformers.js with a DistilBERT SST-2 sentiment model as a simple baseline.

That baseline is useful because it is **not quite right** for reflective writing. The gap between "what the writing feels like" and "what the model labels it" is the research question that drives the exemplar journal and paper.

## Advanced continuation

The later project, [`../bluest-hour-almanac/`](../bluest-hour-almanac/), keeps the same core question but redesigns the interface and swaps the binary baseline for a richer GoEmotions model. That later paper is still valuable, but it is now framed as an advanced continuation of the simpler build → journal → paper chain in this repo.

## Related files

- [`SPECIFICATION.md`](./SPECIFICATION.md) — detailed technical notes for the static build
- [`research-journal.md`](./research-journal.md) — the week-by-week exemplar journal
- [`PAPER.md`](./PAPER.md) — the paper-lite exemplar

## Credits

- Twilight data: [Sunrise-Sunset API](https://sunrise-sunset.org/api)
- ML inference: [Transformers.js](https://huggingface.co/docs/transformers.js)
- Epigraph: Joan Didion, *Blue Nights* (2011)
