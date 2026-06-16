# AGENTS.md

## Repo nature

This is a static **architecture recovery report** for the [PocketFlow](https://github.com/The-Pocket/PocketFlow) framework. There is no code to build, test, lint, or run. It contains only documentation files and diagrams.

## Key files

- `README.md` — the full report (architecture views, design decisions, quality analysis)
- `presentation.html` — Reveal.js slide deck summarizing the report
- `diagrams/diagram_*.png` — 20 numbered figures referenced inline by the report

## Diagrams

All diagrams live in `diagrams/` and are numbered `diagram_1.png` through `diagram_20.png`. The figure numbers and diagram file numbers do not always correspond one-to-one — for example, Figure 8 corresponds to `diagram_3.png`, Figure 9 to `diagram_4.png`, and so on. Refer to the README source to confirm the correct mapping.

## Presentation

`presentation.html` is a self-contained Reveal.js presentation loaded from CDN. Open with `open presentation.html` or serve via any static HTTP server.

## Git

`.claude/` is gitignored — it contains local Claude configuration only.

## No OpenCode config

There is no `opencode.json` file in this repo.
