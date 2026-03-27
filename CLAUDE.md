# CLAUDE.md — StreamSim

## What this project is

StreamSim is a single-file (`index.html`) browser-based agent-based model for high school classroom use, developed for the HLS/BKC Fellowship Data Adventure Day activity. It simulates a streaming video market with 1,000 agents. The intended audience is high school students with some economics or statistics background, plus their teachers.

**Everything lives in `index.html`.** There is no build system, no npm, no server. Changes are made directly to that file.

## Architecture

The simulation is self-contained HTML/CSS/JS with two CDN dependencies:
- **Chart.js** (`cdn.jsdelivr.net`) — line chart (subscriber history), bar chart (subscription distribution), and mini insight charts
- **Tailwind CSS** (`cdn.tailwindcss.com`) — utility styling

### Key JS structures

| Name | Type | Purpose |
|---|---|---|
| `Agent` | class | One simulated viewer. Holds income, age, genre preference, price sensitivity, active subscriptions, and tenure. `evaluate()` runs churn + subscribe logic each month. |
| `Service` | class | One streaming service. Holds price, content quality, content volume, genre focus, color, and `joinedMonth` (0 = present from start; >0 = mid-sim entrant). |
| `agents[]` / `services[]` | arrays | Live simulation state |
| `history[]` | array | `[{month, counts: {serviceId: count}}]` — used to draw the main chart |
| `pauseLines[]` | array | Months where pauses were recorded; drawn as vertical dashed lines on the chart via `pauseLinePlugin` |
| `scheduledPauses[]` | array | Pause events pending this run; each has `{month, title, body, steps}`. Populated from scenario `pauseEvents` and from user timeline clicks. |
| `userPauses[]` | array | Months the user manually placed on the pause timeline. Mirrored into `scheduledPauses`. |
| `lastPauseParams` | object | Snapshot from `snapshotParams()` at the last checkpoint (or sim init). Used to diff param changes in checkpoint cards. |
| `highlightSubCount` | int | -1 = no highlight; 0/1/2/3 = dim all dots except that subscription tier |
| `genreWeights` | object | `{Action, Drama, Comedy, Kids, 'Sci-Fi'}` — controls weighted random genre assignment |
| `MONTHS` | let (number) | Total simulation duration in months. Set from the duration slider at each `initSim()`. Default 48 (4 years). |
| `NAMES` | const array | `['NetStream', 'CineMax', 'GlowTV', 'EdgeVideo', 'PrimeWatch']` — service names by index. |

### Core model equations

**Utility / value score:**
```
U = (Quality × 0.6 + Volume × 0.4) × GenreFit − (Price × Sensitivity × EconomyFactor)
```
- `GenreFit` = 1.3 if service genre matches agent preference, else 1.0

**Economy Factor:**
```
M = 1 + (−IncomeGrowth × 0.10) + (Inflation × 0.04)
```
- Range: ~0.50 (boom) to ~1.90 (recession + high inflation)

**Subscribe probability** (per month, per unsubscribed service):
- U > 5 → 40% base; U 3–5 → 10%; U < 3 → 2%
- Multiplied by stacking discount: 1st sub ×1.0, 2nd ×0.5, 3rd ×0.2

**Churn probability** (per month, per active subscription):
- Quality ≥ 7 → `0.01 + max(0, M−1) × 0.06` (sticky but not recession-proof)
- Quality 4–7 → utility-based: U < 2 → 15%; U < 4 → 5%; else 2%
- Quality < 4 → 35% flat

### Agent dot colors

- 0 subscriptions → dark gray `#333`
- 1 subscription → solid service color
- 2 subscriptions → diagonal split gradient of both service colors
- 3 subscriptions → conic gradient of all three colors

## Key functions

| Function | What it does |
|---|---|
| `initSim(overrides)` | Full reset. Pass `{keepParams: true}` to preserve slider/service values. Pass a scenario's `params` object to load that scenario. Sets `MONTHS` from the duration slider, resets `pauseLines`, `userPauses`, `lastPauseParams`, and clears checkpoint cards. |
| `step()` | Advances one month: checks for scheduled pauses, runs `agent.evaluate()` for all agents, records history, updates all UI. Fires a "Simulation Complete" checkpoint card when `currentMonth >= MONTHS`. |
| `addService(cfg)` | Adds a new service mid-sim without resetting history. Sets `joinedMonth = currentMonth + 1`. Patches a new dataset into the live chart with nulls for pre-entry months. |
| `removeService()` | Removes the last service from `services[]`, cancels all agent subscriptions to it, and splices its chart dataset. |
| `removeServiceById(id)` | Removes a specific service by id (not just the last one). Same cleanup as `removeService()`. |
| `renderServiceProfiles()` | Redraws the per-service age + genre breakdown bars. Called every 3 months during run and on reset. |
| `renderCorrMatrix()` | Renders the Pearson correlation heatmap between service subscriber time series. When "Use monthly changes" is checked, correlates first differences (removes shared growth trend). Only active services with `joinedMonth <= currentMonth` are included. |
| `showPauseCard(event)` | Prepends a checkpoint card above the subscriber chart. First card uses "Initial conditions" mode (no diff). Subsequent cards diff `event.params` against `lastPauseParams` and highlight changes. Detects new entrants / dropped services by name. |
| `buildPauseStats()` | Snapshots current subscriber counts per service with delta vs. prior pause. Handles mid-sim entrants (shows "entering market" if `joinedMonth > currentMonth`). |
| `snapshotParams()` | Returns `{economy, genres, services}` with values rounded to display precision. Used as the baseline for checkpoint card diffs. |
| `renderPauseTimeline()` | Redraws the interactive timeline bar showing scheduled pauses, fired pauses, and the current month indicator. |
| `handleTimelineClick(e)` | Handles clicks on the pause timeline. Adds or removes a user pause at the clicked month. |
| `renderInsightCharts()` | Creates the four Chart.js mini-charts inside the "How It Works" modal. Called via `requestAnimationFrame` after the modal HTML is injected. |
| `getScenarios()` | Returns the four scenario config objects (A–D). Scenarios with `pauseEvents` arrays will auto-pause at the specified months. |
| `styleDot(dot, agent)` | Sets background gradient and dimming on a single agent dot. |

## Keeping this file current

**This file must be updated whenever the ABM changes.** That includes any modification to model equations, coefficients, probability rules, agent/service properties, key function signatures, or simulation state variables. Treat CLAUDE.md as a living spec, not a snapshot — if the code and this file disagree, fix this file as part of the same change.

## Conventions and preferences

- **No build step ever.** All changes go directly in `index.html`. Do not introduce npm, webpack, or any compilation.
- **No new CDN dependencies** without a good reason. The two existing ones (Chart.js, Tailwind) are intentional and sufficient.
- **Audience is high school students.** Avoid jargon in UI-facing strings. Prefer plain English over economic terminology. Technical terms in the "How It Works" modal are fine and expected — but always paired with a plain-language explanation.
- **The "How It Works" modal must stay in sync with the model.** Any change to a formula, coefficient, or probability rule must be reflected in the modal text and its `renderInsightCharts()` data.
- **Scenarios are the primary pedagogical vehicle.** When adding features, consider whether they belong in a scenario's `pauseEvents` rather than as ambient UI.
- **`initSim({keepParams: true})`** is the "Restart" button. **`initSim()`** (no args) is full reset including randomized service values. Keep this distinction.
- The simulation runs at 200ms/step (`setInterval(step, 200)`). Don't change this without considering classroom pacing.

## Tone for UI strings

- Direct and plain: "People Subscribed" not "Market Saturation"
- Active: "Run Simulation" not "Execute"
- No emojis except the ⏸ pause marker on the chart (intentional)
- The "MODEL ASSUMPTION" badge in insights is intentional — it signals epistemic humility to students

## Things to watch out for

- `renderInsightCharts()` is called via `requestAnimationFrame` because the canvas elements are injected into the DOM by `showModal()` just before. Don't call it synchronously.
- The `pauseLinePlugin` is registered globally with `Chart.register()` before any chart is created. It reads from the module-level `pauseLines[]` array.
- **Mid-sim service count changes** no longer call `initSim`. When `currentMonth > 0`, changing the kCount dropdown calls `addService()` / `removeService()` instead. Only a fresh sim (month 0) does a full reinit via `initSim({keepParams: true})`.
- `updateChart()` now emits `null` for months before a service's `joinedMonth`, creating a visual gap on the chart for late-entering services.
- **Checkpoint card first-card detection** uses `container.children.length === 0` at render time. The first card always shows "Initial conditions" with no diff, regardless of `lastPauseParams`.
- `distChart`'s `onClick` handler does the dot highlighting directly — it doesn't go through a shared state update function. If you refactor dot rendering, make sure this still works.
- The correlation matrix uses first differences by default (checkbox checked). Raw counts will show spuriously high correlations due to shared growth trend — this is explained in the card description.
