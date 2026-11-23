# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately tiny to-do tracker: plain HTML, CSS and vanilla JS. No frameworks, no build step, no package manager, no tests. Version control is git, with the remote on GitHub (`gh` is installed at `~/.local/bin`). Keep it that way. Do not introduce dependencies or tooling.

Run it by opening `index.html` in a browser (`open index.html`).

## Files

- `index.html` is the whole app (markup, `<style>`, `<script>` in one file).
- `mockup.html` is a **generated** page showing the app inside iPhone 16 (393×852) and Galaxy S25 (360×780) frames. Each frame is an `<iframe srcdoc="...">` containing an HTML-escaped copy of `index.html` with seeded sample data. Don't hand-edit the `srcdoc` values.

## Regenerating mockup.html

After any change to `index.html`, the two `srcdoc` copies go stale. They were regenerated with a perl script (python3 isn't installed) that:
1. reads `index.html` and replaces `let todos = load('todos', []);` with a seeded array (six sample items; dates are fixed in Nov–Dec 2026, so they eventually become overdue);
2. HTML-escapes `& < > "`;
3. substitutes it into every `srcdoc="[^"]*"` in `mockup.html` (it dies unless there are exactly 2 iframes).

That script lives in a session scratchpad and may no longer exist. If it's gone, rewrite it to do the same three steps. The seed anchor string above must stay in `index.html` or the regeneration breaks.

## Architecture of index.html

- **State**: a `todos` array of `{id, text, done, cat, due, time}`, persisted to `localStorage` key `todos`. Theme is stored under `theme`. All storage access is wrapped in try/catch, since it throws in sandboxed or `data:` contexts.
- **Rendering**: `render()` rebuilds the whole `<ul id="list">` from `todos` each time. It filters by the current category filter, then sorts: done items last, then by `sortKey` (`due + ' ' + (time||'23:59')`, with `'9999'` for items with no due date). Overdue is computed by comparing `sortKey` to a local "now" key.
- **Events**: a single delegated handler on the list, driven by `data-act` attributes (toggle, edit, del, save, cancel). Rows toggle into an inline edit mode; Enter commits and Esc cancels.
- **Categories**: exactly three, defined in `PRESETS` (Work, Personal, Others) with a colour each. Rows set a per-row `--c` CSS variable and tint themselves with `color-mix`. Stored entries with any other category are migrated to "Others" on load.
- **Category picker**: `catPicker()` builds a row of buttons (not a `<select>`). The chosen value lives in the container's `dataset.value`. It is used by the add form (`#cat`), the filter (`#filter`, with an "All" option), and the edit row (`.ec`).
- **Theme**: CSS custom properties with `:root[data-theme=dark]` overrides. It defaults to the system preference. A fixed top-right `role="switch"` toggle (`#theme`) with sun and moon SVGs controls it.
- **IDs**: new items use `Date.now() * 1000 + random(0..999)` to avoid collisions when several are added in the same millisecond.
