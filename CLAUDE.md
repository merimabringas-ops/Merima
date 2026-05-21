# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal weight-loss tracker — a single-page web app that sets a daily
calorie target from the user's stats, logs food/exercise/weigh-ins, and charts
progress toward a goal (default: lose 20 lbs in 8 weeks).

## Running it

There is **no build step and no dependencies**. The entire app is `index.html`.

- Open `index.html` in any modern browser (double-click, or on a phone).
- All data is stored in the browser via `localStorage` under the key
  `merima-tracker-v1`. There is no backend; data never leaves the device.
- To reset during development, clear that key or use the in-app "Reset all data".

## Architecture

Everything lives in `index.html`: inline `<style>`, three screen templates in
the markup, and one inline `<script>`.

- **Screens** — `#setup` (onboarding/edit plan), `#main` (the three tabs), and
  a fixed bottom `#nav`. Visibility is toggled directly; `switchTab()` swaps the
  `.active` class on `.tab` elements. There is no router/framework.
- **State** — a single `state = { profile, logs }` object. `logs` is keyed by
  `YYYY-MM-DD`; each day holds `{ food: [], exercise: [], weight }`. `load()`
  and `save()` sync it to `localStorage`; `save()` must be called after every
  mutation.
- **Calorie model** — `tdee()` uses the Mifflin-St Jeor BMR formula times an
  activity factor. `computeTarget()` subtracts the deficit needed to hit the
  goal, then clamps to a safe floor (`MIN_CAL`: 1200 female / 1500 male). When
  the goal pace would fall below that floor, the Progress tab shows a safety
  note with a realistic projection instead — keep that honesty if you touch
  this logic.
- **Rendering** — `renderToday()`, `renderWeight()`, `renderProgress()` rebuild
  their tab's DOM from `state` via `innerHTML`. `renderAll()` runs all three.
  After any state change, call the relevant render function.
- **Chart** — `drawChart()` hand-builds an SVG string (goal-pace dashed line +
  actual weigh-in line). No charting library.

## Conventions

- Imperial units in the UI (lbs, ft/in); converted to metric only inside
  `tdee()`. `LB_PER_CAL` assumes 3500 kcal ≈ 1 lb.
- Any user-supplied string rendered into `innerHTML` must go through
  `escapeHtml()`.
- Keep it a single self-contained file with zero dependencies — that is the
  point of the project (works offline, opens anywhere).
