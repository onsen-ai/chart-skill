---
name: chart-skill
description: Generate publication-grade charts and data visualizations from YAML specs. Use whenever the user needs charts, graphs, plots, or data visualizations for markdown documents, papers, reports, or presentations. Renders YAML Vega-Lite specs to SVG with themed styling (colors, fonts, labels, axes). Supports bar, line, area, scatter, stacked, and multi-series charts with automatic value labels. Always use this skill when creating any data visualization or chart image.
---

# Chart Skill

Generate publication-grade chart images from YAML Vega-Lite specs. Renders to SVG with themed styling, value labels, and responsive variants. Works with any AI coding agent.

All scripts are in `${CLAUDE_SKILL_DIR}/scripts/` and require Node.js 18+.

## First-Time Setup

**Before running any render command**, check if dependencies are installed:

```bash
ls ${CLAUDE_SKILL_DIR}/node_modules/vega/package.json
```

- **If it exists:** Ready to use.
- **If it doesn't exist:** Install dependencies by running:

```bash
cd ${CLAUDE_SKILL_DIR} && npm install --production
```

This installs vega, vega-lite, and js-yaml (~50MB, one-time). No setup wizard needed — the skill works with defaults immediately.

**Optional:** The user can run the interactive setup wizard to configure default theme, size, and output directory:

> ```
> cd ${CLAUDE_SKILL_DIR} && node scripts/setup.mjs
> ```
> (This requires interactive terminal input — you cannot run it directly.)

Check `~/.chart-skill/config.json` for saved preferences. If it doesn't exist, use defaults (theme: onsen, size: desktop, variant: light).

## Quick Reference

| Task | Command | Key Args |
|------|---------|----------|
| **Render chart** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --output PATH` |
| **Inline YAML** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--yaml 'mark: bar ...' --output PATH` |
| **Dark variant** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --variant dark` |
| **Mobile size** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --size mobile` |
| **All variants** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --all-variants --output-dir DIR` |
| **Custom theme** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --theme neutral` |
| **Custom size** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--spec FILE --width 800 --height 500` |
| **List themes** | `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs` | `--list-themes` |

## Workflow for Papers

1. **Receive data** — from query results, CSV, user-provided, or inline
2. **Write a YAML spec file** — save to a `.yaml` file
3. **Render** — `node ${CLAUDE_SKILL_DIR}/scripts/render.mjs --spec chart.yaml --output ./figures/chart.svg`
4. **Embed in markdown** — `![Chart title](./figures/chart.svg)`
5. **Iterate** — edit the YAML, re-render

The render script prints the output path to stdout. Use `--quiet` to suppress progress messages.

## Writing Chart Specs

Specs are YAML representations of Vega-Lite. The skill applies themed defaults automatically — you only need to specify data, mark type, and encoding.

### Bar chart (vertical)

```yaml
title: Revenue by Quarter
mark: bar
data:
  values:
    - quarter: Q1
      revenue: 120000
    - quarter: Q2
      revenue: 185000
    - quarter: Q3
      revenue: 210000
    - quarter: Q4
      revenue: 245000
encoding:
  x: { field: quarter, type: nominal, title: null }
  y: { field: revenue, type: quantitative, title: "Revenue ($)" }
```

### Bar chart (horizontal)

```yaml
title: Feature Usage
mark: bar
data:
  values:
    - feature: Search
      users: 84
    - feature: Export
      users: 58
    - feature: Filters
      users: 45
encoding:
  y: { field: feature, type: nominal, title: null, sort: "-x" }
  x: { field: users, type: quantitative, title: "Users (%)" }
```

### Line chart

```yaml
title: Monthly Active Users
mark: line
data:
  values:
    - month: Jan
      users: 1200
    - month: Feb
      users: 2400
    - month: Mar
      users: 3800
encoding:
  x: { field: month, type: ordinal, title: null }
  y: { field: users, type: quantitative, title: Users }
```

### Multi-series line (with legend)

```yaml
title: Revenue by Region
mark: line
data:
  values:
    - month: Jan
      region: EMEA
      revenue: 50000
    - month: Jan
      region: APAC
      revenue: 30000
    - month: Feb
      region: EMEA
      revenue: 62000
    - month: Feb
      region: APAC
      revenue: 38000
encoding:
  x: { field: month, type: ordinal, title: null }
  y: { field: revenue, type: quantitative, title: Revenue }
  color: { field: region, type: nominal, title: Region }
```

### Area chart

```yaml
title: Cumulative Users
mark: area
data:
  values:
    - month: Jan
      total: 500
    - month: Feb
      total: 1200
    - month: Mar
      total: 2100
encoding:
  x: { field: month, type: ordinal, title: null }
  y: { field: total, type: quantitative, title: Users }
```

### Stacked bar chart

```yaml
title: Tickets by Priority
mark: bar
data:
  values:
    - month: Jan
      priority: Low
      count: 45
    - month: Jan
      priority: High
      count: 12
encoding:
  x: { field: month, type: ordinal, title: null }
  y: { field: count, type: quantitative, title: Tickets }
  color: { field: priority, type: nominal, title: Priority }
```

### Scatter plot

```yaml
title: Duration vs Satisfaction
mark: point
data:
  values:
    - duration: 5
      score: 3.2
    - duration: 12
      score: 4.1
encoding:
  x: { field: duration, type: quantitative, title: "Duration (min)" }
  y: { field: score, type: quantitative, title: "Score" }
```

## Encoding Tips

- `type: nominal` — unordered categories (Vega-Lite may sort alphabetically)
- `type: ordinal` — ordered categories (preserves data order)
- `type: quantitative` — numbers
- `title: null` — hide axis title
- `sort: "-x"` — sort by descending value (useful for horizontal bars)
- Adding `color` to encoding creates multi-series with a legend

## Automatic Behaviors

These are applied by default — you don't need to specify them:

- **Value labels** on single-series bar/line/area charts (comma-formatted numbers above each data point)
- **Stacked bar totals** shown above each stack
- **Multi-series** charts skip value labels (too cluttered) and show a legend instead
- **Axis formatting**: 5 Y-axis ticks, horizontal labels, no X-axis tick marks
- **Bar styling**: square corners, background-colored borders between segments
- **Line styling**: 2px lines with hollow dots (white fill, colored border)
- **Scatter styling**: solid filled dots

## Available Themes

| Theme | Style | Best for |
|-------|-------|----------|
| `onsen` | Blue primary, warm palette | Product/marketing content |
| `neutral` | Grayscale | Academic papers, formal reports |

Custom themes: drop a JSON file in `~/.chart-skill/themes/` following the format of the built-in themes.

## Size Presets

| Size | Width | Height | Use for |
|------|-------|--------|---------|
| `desktop` | 728px | 420px | Web articles, reports, presentations |
| `mobile` | 600px | 400px | Mobile-optimized views |

Override with `--width` and `--height` for custom dimensions.

## Troubleshooting

| Error | Fix |
|-------|-----|
| `Cannot find module 'vega'` | Run `npm install` in the skill directory |
| `spec file not found` | Check the `--spec` path is correct |
| `Theme "X" not found` | Run `--list-themes` to see available themes |
| `invalid YAML` | Check YAML syntax — common issues: missing quotes around strings with special chars |
