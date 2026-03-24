# StreamSim

StreamSim is a browser-based classroom simulation for high school economics and statistics. It models a fictional streaming video market using an **agent-based model (ABM)**: 1,000 simulated viewers each independently decide each month whether to subscribe to, keep, or cancel streaming services based on price, content quality, genre preferences, and macroeconomic conditions.

## What it's for

Students use StreamSim to explore how individual decisions aggregate into market-level outcomes — and how changing a single variable (price, quality, inflation) ripples through the whole system. The tool is designed to make abstract concepts like price sensitivity, marginal utility, and macroeconomic multipliers visible and interactive.

## How to use it

Open `index.html` in any modern browser. No installation or internet connection required after the initial page load (CDN scripts load Chart.js and Tailwind CSS).

## Key controls

| Control | What it does |
|---|---|
| **Run / Pause** | Steps the simulation forward one month at a time (200ms/step) |
| **Restart** | Resets agents and history, keeping your current service and economy settings |
| **Reset Params** | Resets everything including parameters to defaults with new random service values |
| Economy sliders | Adjust household income growth and inflation; both independently scale how price-sensitive viewers become |
| Service cards | Set price, content quality, content volume, and genre focus for each service |
| Audience Genre Mix | Set the distribution of genre preferences across the viewer population |

## Scenarios

Three pre-configured classroom scenarios are available under **Scenarios** in the nav. Each loads specific parameter settings and poses a guiding question. Scenarios B and C include automatic pause points with on-screen instructions prompting students to change variables and observe the effect.

| Scenario | Concept |
|---|---|
| A — Price War | Does cheaper always win when quality is equal? |
| B — Recession Shock | How does a sudden income drop ripple through a streaming market? |
| C — Niche vs. Mass Market | High-quality niche content vs. high-volume broad content — which strategy wins? |

## How the model works

Each simulated viewer has an **income level**, **price sensitivity** (inversely correlated with income), **age group**, and **genre preference**. Every month each viewer scores every service using a utility function, then probabilistically subscribes or cancels based on that score.

- **Subscribing**: higher scores → higher probability, but each additional subscription is less likely (diminishing returns)
- **Canceling**: driven primarily by content quality — high-quality services are sticky even in a recession; low-quality services lose subscribers quickly regardless of price
- **Economy**: income growth and inflation each independently shift a spending pressure multiplier that scales every viewer's price sensitivity

See **How It Works** in the simulation nav for the full model equations and interactive charts.

## Files

```
index.html   — the entire simulation (self-contained)
README.md    — this file
```

## Disclosure

This code written with assistance from Claude Sonnet 4.6.
