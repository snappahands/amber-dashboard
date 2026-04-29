# Accounts Dashboard (AmberDashboard)

Static GitHub Pages dashboard for Deako's builder-account team. Live at **https://snappahands.github.io/amber-dashboard/**. The page title says "Accounts Dashboard" but the user may still call it "Amber's Dashboard" or "Builder Dashboard" — those are old names.

## This is its own git repo

Even though it lives at `C:\Users\slypi\ClaudeProjects\AmberDashboard\` inside the `deako-executive-dashboard` working tree, **this is a standalone repo** with its own remote: `https://snappahands@github.com/snappahands/amber-dashboard.git`. The parent repo must NOT track this directory (would commit as a gitlink/submodule and break Pages).

Always commit from inside this repo:

```bash
git -C AmberDashboard add ...
git -C AmberDashboard commit ...
git -C AmberDashboard push origin master
```

The remote needs the `snappahands@` username prefix or git-credential-manager hangs. See the parent CLAUDE.md for the kill recipe.

## Architecture

Single-page app, three relevant files:

- `index.html` — entire UI + logic (sidebar list, builder detail card, JSONBin sync)
- `data.js` — `const BUILDERS = [...]` — bundled fallback data (only used on first visit / JSONBin failure)
- `build.py` — script that regenerates `data.js` from `c:/Users/slypi/Downloads/builder_data_compact.json`. Re-running it overwrites any local edits to `data.js`, but JSONBin remains the live source of truth.

## Live data — JSONBin

`BUILDERS` is mirrored to JSONBin bin **`69cd569e36566621a86da1bb`** (master key embedded in `index.html`). Every visitor reads + writes the same store, so edits propagate to everyone.

Stale-while-revalidate: page renders instantly from a localStorage cache (`amber_dashboard_cache_v1`), then fetches JSONBin async. Edits autosave to localStorage immediately and push to JSONBin (no debounce here — push on every edit).

If the page renders empty for users (we hit this once after I edited `data.js` directly), the live state in JSONBin is what actually drives the UI. To restore from `data.js`: PUT the contents of `data.js` to JSONBin via the API (with a Mozilla User-Agent header — Cloudflare blocks default Python urllib).

## Conventions

- Version badge in the bottom-right (`#version-badge`). Bump it on every commit so the user can verify the right version is live.
- Every field is `contenteditable` — saves trigger via `onEdit → scheduleSave → pushToJsonBin`.
- "+ Add Builder" button at the top of the sidebar inserts a fresh empty entry. Delete is intentionally not exposed (we removed it earlier — public visitors should add/edit, not delete).
- Three nav bubbles at the top of the page link to the Executive and Initiatives dashboards (the other two in the trio).
