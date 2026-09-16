---
name: bio-figure
description: "Generate publication-quality biology figures (bar, scatter, heatmap, line, volcano) following Nature/Cell/Science standards from raw data."
---

# Bio-Figure Skill

Generate submission-ready scientific figures from raw data (CSV/Excel). Target: Nature / Cell / Science aesthetic standards.

## Core Principles (Non-negotiable)

1. **Show every data point.** Bar charts MUST overlay raw jitter points. Never plot a bare mean bar. When n >= 3, show distribution.
2. **Audit the data first.** Before plotting, report: number of groups, n per group, independent/dependent variables, data type (continuous vs count), missing values. Confirm with the user before proceeding.
3. **No fabricated statistics.** Choose the test based on data type. Note the method in the figure legend or code comments. Only annotate p-values when sample size is sufficient.
4. **Restraint is premium.** No gridlines (unless justified), no background color, no 3D, no shadows, no gradient fills.
5. **Submission-ready output.** 300 DPI minimum. Single-column width 89 mm or double-column 183 mm (Nature standard). Font: Arial / Helvetica (sans-serif).

## Global Visual Spec

| Item | Standard |
|---|---|
| Font | Arial or Helvetica, sans-serif |
| Axis label size | 7 pt (single-col) / 8 pt (double-col) |
| Tick label size | 6.5 pt |
| Line width | Main axis 0.75 pt, secondary 0.5 pt |
| Error bar line width | 0.75 pt, cap width 3 pt |
| DPI | 300 (minimum for submission), export TIFF/PDF |
| Background | Pure white, no gridlines (at most faint horizontal grid alpha=0.2) |
| Palette | NPG-style palette (see below) |

### Default Palette (NPG style, low-saturation, high-discriminability)

This is the `npg` palette from the R package [ggsci](https://github.com/nanxstats/ggsci), inspired by figures in Nature Publishing Group journals such as *Nature Reviews Cancer*. It is not an official palette published by Nature.

```python
nature_colors = [
    '#E64B35',  # vermilion
    '#4DBBD5',  # cyan-blue
    '#00A087',  # pine-green
    '#3C5488',  # dark-blue
    '#F39B7F',  # light-orange
    '#8491B4',  # grey-blue
    '#91D1C2',  # light-green
    '#DC0000',  # deep-red
    '#7E6148',  # brown
    '#B09C85',  # light-brown
]
```

Same group, different conditions: use light/dark shades of one color. Different groups: use distinct colors from the palette.

### Significance Annotations

- ns = not significant (p > 0.05)
- \* = p < 0.05
- \*\* = p < 0.01
- \*\*\* = p < 0.001
- \*\*\*\* = p < 0.0001

Comparison brackets: thin black line spanning compared groups above error bars, line width 0.5 pt, asterisks centered above the bracket.

## Rules by Figure Type

### 1. Bar Chart (most scrutinized by reviewers)

**When to use:** Compare means across independent groups (e.g. cell viability under different treatments).

**Must do:**
- Overlay all raw data points on each bar (jitter scatter), point size 3-4 pt (matplotlib `s` ≈ 10-16), white stroke 0.5 pt, point color a darker shade of the bar fill so points inside the bar stay visible
- Error bars: use SD when n < 10, SEM when n >= 10. State which in the legend or code comments
- Y-axis must start at 0 (unless magnitude differences are extreme; explain truncation in legend)
- Bar fill from palette, edge color 20% darker than fill
- Comparison brackets with asterisks above error bars with adequate spacing

**Forbidden:** 3D bars, gradient fills, bare mean bars without scatter, Y-axis not starting at 0, mixing error bar types in the same figure.

**Code skeleton (matplotlib + seaborn):**

```python
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import numpy as np

plt.rcParams['font.family'] = 'sans-serif'
plt.rcParams['font.sans-serif'] = ['Arial', 'Helvetica', 'DejaVu Sans']

nature_colors = ['#E64B35', '#4DBBD5', '#00A087', '#3C5488', '#F39B7F',
                 '#8491B4', '#91D1C2', '#DC0000', '#7E6148', '#B09C85']

# Input: groups (list of 1-D arrays), group_labels (list of str)
# Replace this example data with the user's data.
rng = np.random.default_rng(0)
groups = [rng.normal(100, 8, 6), rng.normal(72, 10, 6), rng.normal(45, 9, 6)]
group_labels = ['Control', 'Drug A', 'Drug B']

fill_colors = nature_colors[:len(groups)]
# Edge color: 20% darker than the fill
edge_colors = [tuple(c * 0.8 for c in mcolors.to_rgb(col)) for col in fill_colors]
# Points: 40% darker than the fill, so points inside the bar stay visible
point_colors = [tuple(c * 0.6 for c in mcolors.to_rgb(col)) for col in fill_colors]

# Error bars: SD if any group has n < 10, otherwise SEM.
# One type for the whole figure; state it in the legend.
use_sd = min(len(g) for g in groups) < 10
means = [np.mean(g) for g in groups]
sds = [np.std(g, ddof=1) for g in groups]
errs = sds if use_sd else [sd / np.sqrt(len(g)) for sd, g in zip(sds, groups)]
err_label = 'SD' if use_sd else 'SEM'

fig, ax = plt.subplots(figsize=(3.5, 2.8), dpi=300)  # 89 mm single column
x = np.arange(len(groups))

ax.bar(x, means, width=0.6, color=fill_colors,
       edgecolor=edge_colors, linewidth=0.75, zorder=2)

ax.errorbar(x, means, yerr=errs, fmt='none', color='black',
            capsize=3, elinewidth=0.75, capthick=0.75, zorder=3)

for i, g in enumerate(groups):
    ax.scatter(rng.normal(i, 0.08, size=len(g)), g,
               s=12, color=point_colors[i], edgecolor='white',
               linewidth=0.5, zorder=4)

ax.set_xticks(x)
ax.set_xticklabels(group_labels, fontsize=7)
ax.set_ylabel('Response (%)', fontsize=7)
ax.set_ylim(bottom=0)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
for side in ('left', 'bottom'):
    ax.spines[side].set_linewidth(0.75)
ax.tick_params(labelsize=6.5, width=0.75)
print(f'Error bars: mean ± {err_label}')  # copy into the figure legend
plt.tight_layout()
fig.savefig('fig_bar_example.png', dpi=300, bbox_inches='tight')
fig.savefig('fig_bar_example.tif', dpi=300, bbox_inches='tight')
```

### 2. Scatter Plot (correlation / distribution)

**When to use:** Relationship between two continuous variables (gene expression vs protein level, dose-response).

**Must do:**
- Correlation plots: draw linear fit line with 95% CI band (light grey, semi-transparent). Annotate R-squared and p-value in corner
- Grouped scatter: different groups by color; at most 3 marker shapes (circle / square / triangle)
- Point size 20-30 (matplotlib `s`), white stroke 0.5 pt
- Axis labels include units

**Volcano plot special rules:**
- X-axis: log2 fold change; Y-axis: -log10(adjusted p-value)
- Non-significant genes: light grey alpha=0.5
- Up-regulated significant: red
- Down-regulated significant: blue
- Threshold lines: x = +/-1 (log2FC), y = -log10(0.05) as dashed lines
- Annotate top 10 DE gene names, font 6 pt

**Forbidden:** Rainbow colormap for continuous variables (use viridis or YlOrRd), oversized points hiding data, missing fit line / stats.

### 3. Heatmap (expression profiles / matrices)

**When to use:** Gene/sample expression matrices, correlation matrices, multi-omics comparisons.

**Must do:**
- Default palette: `RdBu_r` (red-white-blue) for z-scores, `viridis` for raw values
- For clustered heatmaps: use `seaborn.clustermap`, keep dendrograms small
- Color bar must label z-score or raw value range
- Sample group annotation: thin color strip (annotation bar) at the top
- Font 6-7 pt, no overlapping row/column labels
- Turn off cell annotations when matrix > 50x50

**Forbidden:** Rainbow colormap (jet/rainbow), unlabeled color scale, overlapping labels, mixing cluster dendrograms with group annotations.

### 4. Line Plot (time series / dose-response)

**When to use:** Time courses (cell proliferation, growth curves), dose-response curves.

**Must do:**
- Each line + shaded error band (`fill_between`, alpha=0.15, same color as line) instead of error bars at each point
- Line width 1.5 pt
- Data point markers: small circles 3 pt, white stroke
- X-axis: if time/dose, ensure equal spacing or clearly label log scale
- Multiple groups distinguished by palette colors; legend inside upper-right or outside, never obscuring data

**Forbidden:** Lines thicker than 2 pt, dense error bars, unequal X spacing without log label, legend covering data.

## Workflow (execute in order every time)

1. **Ask:** Target journal (determines size/font), data structure, desired figure type
2. **Audit data:** Print first 5 rows, shape, column types, missing values
3. **Select figure type:** Based on data type and scientific question (if user's choice seems wrong, explain before plotting)
4. **Write code:** Python (matplotlib + seaborn + pandas), complete and runnable
5. **Export:** Save 300 DPI TIFF and PNG
6. **Self-check:**
   - Are all raw data points displayed?
   - Is the error bar type noted?
   - Is the font sans-serif and appropriately sized?
   - Is the specified palette used?
   - Are top/right spines removed?
   - Do axis labels include units?
   - Is the output 300 DPI?

## Output Requirements

- Code must be complete and runnable, no omitted imports
- Each figure saved as a separate file: `fig_<type>_<description>.tif`, plus a `.png` copy with the same name
- Multi-panel figures (a/b/c/d): use `plt.subplots` with unified layout; panel labels in bold 8 pt at upper-left of each subplot
- Final lines of code: `plt.savefig('fig_<type>_<description>.tif', dpi=300, bbox_inches='tight')` and the same call with `.png`