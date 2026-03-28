# StreamSim

StreamSim is a browser-based classroom simulation for high school economics and statistics. It models a fictional streaming video market using an **agent-based model (ABM)**: 1,000 simulated viewers each independently decide each month whether to subscribe to, keep, or cancel streaming services based on price, content quality, genre preferences, and macroeconomic conditions.

## What it's for

Students use StreamSim to explore how individual decisions aggregate into market-level outcomes — and how changing a single variable (price, quality, inflation) ripples through the whole system. The tool is designed to make abstract concepts like price sensitivity, marginal utility, and macroeconomic multipliers visible and interactive.

## How to use it

Open `index.html` in any modern browser. No installation or internet connection required after the initial page load (CDN scripts load Chart.js and Tailwind CSS).

## Key controls

| Control | What it does |
|---|---|
| **Run / Pause** | Steps the simulation forward one month at a time (200ms/step). Pausing mid-sim records a checkpoint card above the subscriber chart. |
| **Restart** | Resets agents and history, keeping your current service and economy settings |
| **Reset Params** | Resets everything including parameters to defaults with new random service values |
| **Simulation Length** | Sets the total run duration (1–20 years). Takes effect immediately on the chart. |
| **Pause Points timeline** | Click any point on the timeline bar to schedule an automatic pause at that month. Click an existing marker to remove it. |
| Economy sliders | Adjust household income growth and inflation; both independently scale how price-sensitive viewers become |
| Service cards | Set price, content quality, content volume, and genre focus for each service. Use the ✕ button to remove a service. |
| Number of Services | Adding services mid-sim introduces a new entrant — its chart line starts from the month it joined. |
| Audience Genre Mix | Set the distribution of genre preferences across the viewer population |

## Checkpoint cards

Whenever the simulation pauses — whether from a scenario, a scheduled pause, a manual click, or reaching the end — a checkpoint card appears above the subscriber chart. Each card shows:

- Subscriber counts per service (with change from the prior checkpoint)
- Parameter snapshot: economy settings, genre weights, and per-service values
- Changes since the last checkpoint highlighted in yellow; new/removed services flagged in the header
- The first card shows initial conditions only (no diff)

Cards are collapsible. Click the header to collapse; use the `>` arrow to expand the full settings list.

## Correlation matrix

A heatmap at the bottom of the dashboard shows Pearson correlations between each pair of service subscriber time series. **"Use monthly changes"** (on by default) computes correlations on month-over-month changes rather than raw counts, removing the shared growth trend that would otherwise inflate all correlations toward +1.

## Game Mode

A **Game** toggle in the top-right nav bar switches between Simulation mode and Game mode. Game mode adds goals, constraints, and a resource management layer on top of the same underlying ABM.

**"The Disruption"** (the first scenario, ~3 min):

1. **Pre-run** — 4 competitor services run for 12 months at 1.2× speed while you observe the market.
2. **Founding** — configure your service (StreamCo) using an 8-point budget: Quality costs 2 pts/level above 3, Volume costs 1 pt/level above 3, and each $1 below $10 price costs 1 pt. Genre is free. Unspent points become starting credits.
3. **Active phase** — 66 months of live play. Credits accumulate each month from `subscribers × price × 0.015`. The sim pauses every 6 months for a **Quarterly Strategy Review** where you spend credits on upgrades: +1 Quality (25¢), +1 Volume (15¢), -$1 Price (10¢), +$1 Price (free, reversible within the same review), Marketing Burst +15% subscribe rate for 3 months (20¢), Genre Pivot (30¢).
4. **Market events** fire mid-game — 1–3 per quarter, randomized each run (competitor quality investments, library expansions, price cuts, price hikes). No two runs are identical.
5. **Win target**: 175+ subscribers = Bronze, 225+ = Silver, 275+ = Gold.

Winning requires reasoning about the model equations: Quality = subscriber retention, Volume = growth rate, genre match = +30% utility, marketing boost = +2 utility score. Kids genre is deliberately unserved by competitors — the sim's clearest niche opportunity.

Additional scenario slots (The Comeback, Survive the Recession) are in the code but locked pending implementation.

## Scenarios

Four pre-configured classroom scenarios are available under **Scenarios** in the nav. Each loads specific parameter settings and poses a guiding question. Scenarios B, C, and D include automatic pause points with on-screen instructions prompting students to change variables and observe the effect.

| Scenario | Concept |
|---|---|
| A — Price War | Does cheaper always win when quality is equal? |
| B — Recession Shock | How does a sudden income drop ripple through a streaming market? |
| C — Niche vs. Mass Market | High-quality niche content vs. high-volume broad content — which strategy wins? |
| D — New Market Entrant | Two incumbents ignore a large audience segment. Can a new entrant exploit the gap? |

## How the model works

Each simulated viewer has an **income level**, **price sensitivity** (inversely correlated with income), **age group**, and **genre preference**. Every month each viewer scores every service using a utility function, then probabilistically subscribes or cancels based on that score.

- **Subscribing**: higher scores → higher probability, but each additional subscription is less likely (diminishing returns)
- **Canceling**: driven primarily by content quality — high-quality services are sticky even in a recession; low-quality services lose subscribers quickly regardless of price
- **Economy**: income growth and inflation each independently shift a spending pressure multiplier that scales every viewer's price sensitivity
- **New entrants**: services can be added mid-simulation; agents begin evaluating them immediately the following month

See **How It Works** in the simulation nav for the full model equations and interactive charts.

## Files

```
index.html   — the entire simulation (self-contained)
README.md    — this file
```

## Disclosure

This code written with assistance from Claude Sonnet 4.6.
