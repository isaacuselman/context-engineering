# Phase 0 — Context Engineering for Claude Code

Six self-paced exercises on writing CLAUDE.md files, budgeting context, designing subagents, writing specs, and decomposing tasks. The platform itself (`index.html` + `exercises/*.md`) is the capstone — it was built using the practices the exercises teach.

## Run locally

The app fetches markdown files at runtime, so `file://` will not work (browsers block `fetch()` on `file://` origins). Serve over HTTP:

```bash
cd phase-0
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

Progress is stored in `localStorage` under keys `completed-{1..6}` and `notes-{1..6}`. It is scoped to the origin, so local-dev and a deployed site do not share state.

## Deploy to GitHub Pages

1. Create an empty repo on GitHub (e.g. `context-engineering-phase-0`).
2. From the repo root:
   ```bash
   git add .
   git commit -m "Initial Phase 0 curriculum"
   git branch -M main
   git remote add origin git@github.com:<you>/context-engineering-phase-0.git
   git push -u origin main
   ```
3. In the GitHub repo settings, enable **Pages** from `main` branch, root (`/`).
4. Visit `https://<you>.github.io/context-engineering-phase-0/`.

## Structure

```
phase-0/
├── index.html          single-file app (HTML + CSS + JS)
├── exercises/          the six markdown exercises
├── docs/
│   └── source-material.md   primary-source research backing the exercises
└── README.md
```

No build step, no package manager, no framework. The only external dependency is [`marked@14`](https://www.npmjs.com/package/marked) loaded from jsdelivr.

## Out of scope for MVP

No manifest file (exercises are hardcoded in a JS array), no subagent frontmatter system, no answer-key collapsibles, no progress bar, no mobile-specific breakpoints, no multi-phase architecture. If Phase 1 happens, rebuild the structure at that point.
