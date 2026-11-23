# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Material Price Tracker and Trending Chart: a single-file, framework-free web app (`index.html`, inline CSS + vanilla JS in one IIFE) for tracking factory material prices against a per-item base price. The user explicitly requires **no frameworks or libraries** (the chart is hand-drawn on `<canvas>`), so don't add dependencies or a build step.

There is no build, lint or test tooling. Run it by opening `index.html` in a browser (file:// works). Note the built-in preview pane has no `localStorage`, so persistence can't be verified there; saves fail with a warning but in-memory state still works.

## Architecture

- **Master data**: `CATS` maps Meat / Ingredient / Expendables / Others to fixed item lists. "Others" is intentionally empty; users add items via `data.customItems[cat]`. `itemsOf(cat)` merges both.
- **Storage**: `localStorage` key `mptData` = `{customItems:{cat:[names]}, records:{"Cat|Item":{base:{ym,price}, prices:[{ym,price}]}}}`. `ym` is `"YYYY-MM"` (month/year only, string-compared for ordering). Export/Import buttons round-trip this JSON.
- **Rules enforced in `onSave`**: a base price must exist before new prices; new-price month must be after the base month; same month overwrites; base month can't move later than existing prices.
- **Chart** (`render()`): `buildSeries()` reads the chart filter (`#fCat`/`#fItem`, independent of the entry form). Single item plots actual prices against a dashed base line; "All items in category" (`__all`) plots % change vs each item's base (0% line) so scales are comparable. Colour rule: price below base = green, above = red, equal = grey. `segs()` splits line segments at base-line crossings so each piece takes the correct colour and the area fill is drawn per piece. X axis is categorical (sorted unique months), not time-scaled. Hit-testing for the tooltip uses the `hits` array built during drawing.
- `render()` also refreshes KPIs, legend and the table (`renderKpis/renderLegend/renderTable`); saving an entry syncs the chart filter to the saved item.
- User-supplied item names are inserted into innerHTML, so always pass them through `esc()`.
