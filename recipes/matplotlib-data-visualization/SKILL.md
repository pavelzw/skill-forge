---
name: matplotlib-data-visualization
description: >-
  Chart design and data storytelling guidelines for matplotlib.
  Use when creating visualizations that need to communicate a clear message,
  tell a story with data, or follow best practices for readability and design.
  Focuses on visualization principles rather than matplotlib API syntax.
license: BSD-3-Clause
---

# Storytelling with Data: Chart Design Guidelines for Matplotlib

This skill focuses on **visualization design principles** for creating clear, compelling charts that communicate your data's message. It is not a matplotlib API reference — it is a guide for making charts that are easy to read, honest, and visually appealing.

## Choose the Right Chart Type

**Avoid pie charts** for most use cases. Human perception is poor at comparing angles and areas. Use horizontal bar charts instead — they are the easiest chart type to read for categorical comparisons.

**Avoid grouped bar charts.** Side-by-side bars are hard to compare across groups. Use **stacked bar charts** if the message is about part-to-whole composition, or a **slope chart** if the message is about the change between two groups or time periods.

**Use horizontal bar charts** for categorical comparisons. They allow natural left-to-right reading and accommodate long category labels without rotation.

```python
import matplotlib.pyplot as plt

categories = ["Customer Support", "Engineering", "Marketing", "Sales", "Operations"]
values = [82, 95, 67, 78, 71]

fig, ax = plt.subplots(figsize=(8, 4))
bars = ax.barh(categories, values, color="#cccccc")
# Highlight the key bar
bars[1].set_color("#e63946")
ax.set_xlim(0, 110)
ax.set_title("Engineering leads in satisfaction scores", loc="left", fontweight="bold")
ax.spines[["top", "right", "bottom"]].set_visible(False)
ax.tick_params(left=False)
ax.xaxis.set_visible(False)

# Label bars directly instead of using an x-axis
for bar, val in zip(bars, values):
    ax.text(bar.get_width() + 1.5, bar.get_y() + bar.get_height() / 2,
            str(val), va="center", fontsize=10)

plt.tight_layout()
```

**Use vertical bar charts** when the x-axis represents chronological progression. Time reads naturally left-to-right on a horizontal axis.

**Use line charts** to show continuous data over time. Lines clearly convey trends, rates of change, and patterns. Use them instead of bar charts when the focus is on the shape of change rather than individual values.

**Use slope charts** to compare exactly two time periods or categories, emphasizing the direction and magnitude of change between them.

```python
fig, ax = plt.subplots(figsize=(4, 5))

# Two time periods
labels = ["2022", "2024"]
product_a = [45, 62]
product_b = [60, 55]

ax.plot([0, 1], product_a, "o-", color="#e63946", linewidth=2, markersize=8)
ax.plot([0, 1], product_b, "o-", color="#aaaaaa", linewidth=2, markersize=8)

# Label lines directly at their endpoints
ax.text(-0.15, product_a[0], f"Product A: {product_a[0]}", va="center", color="#e63946")
ax.text(1.08, product_a[1], f"{product_a[1]}", va="center", color="#e63946", fontweight="bold")
ax.text(-0.15, product_b[0], f"Product B: {product_b[0]}", va="center", color="#aaaaaa")
ax.text(1.08, product_b[1], f"{product_b[1]}", va="center", color="#aaaaaa")

ax.set_xticks([0, 1])
ax.set_xticklabels(labels)
ax.set_xlim(-0.4, 1.4)
ax.spines[["top", "right", "left", "bottom"]].set_visible(False)
ax.yaxis.set_visible(False)
ax.set_title("Product A overtook Product B", loc="left", fontweight="bold")

plt.tight_layout()
```

## Reduce Clutter

Every non-data element competes for attention. Remove anything that does not directly help the reader understand the data.

**Always despine at least the top and right spines** — they form a box around the data that adds no information. For charts where you label data directly (e.g., bar values on top of bars), you can also remove the left spine and y-axis entirely, leaving only the data and its labels.

```python
# Minimum: always remove top and right
ax.spines[["top", "right"]].set_visible(False)

# When bars are labeled directly, also remove left spine and y-axis
ax.spines[["top", "right", "left"]].set_visible(False)
ax.yaxis.set_visible(False)

# Full despine for slope charts or minimal designs
ax.spines[["top", "right", "left", "bottom"]].set_visible(False)
```

**Remove gridlines.** If gridlines are necessary, use a light grey (`#eeeeee` or `#dddddd`).

```python
ax.yaxis.grid(True, color="#eeeeee", linewidth=0.8)
ax.set_axisbelow(True)
```

**Remove legends — label data directly.** Legends force the reader to look back and forth between the chart and the legend. Place labels next to the data they describe.

```python
fig, ax = plt.subplots(figsize=(8, 4))

months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
online = [120, 135, 150, 145, 170, 190]
retail = [200, 190, 180, 175, 165, 155]

ax.plot(months, online, color="#e63946", linewidth=2.5)
ax.plot(months, retail, color="#aaaaaa", linewidth=2.5)

# Label lines directly at the last data point
ax.text(len(months) - 0.85, online[-1] + 4, "Online", color="#e63946",
        fontweight="bold", fontsize=11)
ax.text(len(months) - 0.85, retail[-1] + 4, "Retail", color="#aaaaaa",
        fontweight="bold", fontsize=11)

ax.spines[["top", "right"]].set_visible(False)
ax.set_title("Online sales surpassing retail", loc="left", fontweight="bold")
plt.tight_layout()
```

**Label bars directly** instead of relying on axis tick marks. This eliminates the need for gridlines entirely.

```python
fig, ax = plt.subplots(figsize=(8, 3.5))
categories = ["Q1", "Q2", "Q3", "Q4"]
values = [24, 31, 28, 42]

bars = ax.bar(categories, values, color="#cccccc", width=0.5)
bars[-1].set_color("#e63946")  # Highlight Q4

# Place value labels on top of each bar
for bar, val in zip(bars, values):
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.8,
            str(val), ha="center", va="bottom", fontsize=11)

ax.spines[["top", "right", "left"]].set_visible(False)
ax.yaxis.set_visible(False)
ax.set_title("Q4 revenue spike driven by holiday campaign", loc="left", fontweight="bold")
plt.tight_layout()
```

**Avoid 3D effects, shadows, and gradients.** They distort data perception and add no information. Matplotlib's default 2D rendering is always the right choice.

**Use white space** to let the chart breathe. Generous margins and padding reduce visual noise.

## Use Color Intentionally

Color should encode meaning, not decoration. The most effective charts use color sparingly to draw attention.

**Use grey as your baseline.** Render most data in grey (`#cccccc` or `#aaaaaa`), then use a single bold accent color to highlight the element that matters.

```python
colors = ["#cccccc", "#cccccc", "#e63946", "#cccccc", "#cccccc"]
ax.bar(categories, values, color=colors)
```

**Limit your palette to 1-2 accent colors.** More colors create visual clutter and make it harder to identify what is important. A simple palette:
- Grey baseline: `#cccccc` or `#aaaaaa`
- Primary accent: one bold, saturated color (e.g. `#e63946`, `#1d3557`, `#2a9d8f`)
- Secondary accent (if needed): one muted complementary color

**Ensure sufficient contrast.** Text and data elements must be clearly visible against the background. Dark text on a white background, bold accent colors against grey.

**Use color consistently** across related charts. If "Product A" is red in one chart, it should be red in every chart within the same presentation or report.

## Align and Order Thoughtfully

**Left-align text.** It is easiest to read. Avoid center-aligned titles and labels.

```python
ax.set_title("Revenue by region", loc="left", fontweight="bold")
```

**Order data logically.** Sort bar charts by value (ascending or descending) unless the categories have an inherent order (e.g., months, age groups). This makes comparisons immediate.

```python
# Sort by value for easier comparison
sorted_pairs = sorted(zip(values, categories))
sorted_values, sorted_categories = zip(*sorted_pairs)
ax.barh(sorted_categories, sorted_values)
```

**Place key comparisons side-by-side.** When comparing two things, put them next to each other rather than across the chart.

## Focus Attention

**Write titles that state the takeaway, not the chart type.** The title is the most-read element. Use it to tell the reader what they should learn.

- Bad: "Quarterly Revenue 2024"
- Good: "Q4 revenue surged 50% driven by holiday campaign"

```python
# Descriptive title that tells the story
ax.set_title("Customer churn dropped after onboarding redesign",
             loc="left", fontweight="bold", fontsize=13)

# Optional subtitle for context
ax.text(0, 1.02, "Monthly churn rate, Jan-Dec 2024",
        transform=ax.transAxes, fontsize=10, color="#666666")
```

**Use annotations to explain spikes, drops, or key data points.** Don't assume the reader will interpret the data the same way you do.

```python
ax.annotate("New policy\nimplemented",
            xy=(3, values[3]),              # Point to annotate
            xytext=(3.5, values[3] + 15),   # Text position
            fontsize=9, color="#666666",
            arrowprops=dict(arrowstyle="->", color="#999999", linewidth=1.2))
```

**Use pre-attentive attributes** — color, size, and position — to guide the eye. The highlighted element should be visible within the first second of looking at the chart.

**Never use diagonal text.** If labels don't fit horizontally, rotate the chart (use a horizontal bar chart) or abbreviate labels. Diagonal text is hard to read and looks cluttered.

```python
# Instead of rotating labels on a vertical bar chart:
# ax.set_xticklabels(labels, rotation=45)  # Avoid this

# Use a horizontal bar chart instead:
ax.barh(labels, values)
```

## Simplify the Y-Axis

**Start the y-axis at zero for bar charts.** Bars encode values by their length. A non-zero baseline exaggerates differences and misleads the reader. Line charts may use a non-zero baseline when focusing on variation.

**Remove unnecessary decimal places.** Display `42` instead of `42.00`. Match precision to what is meaningful.

**Format large numbers for readability.** Use K for thousands, M for millions.

```python
from matplotlib.ticker import FuncFormatter

ax.yaxis.set_major_formatter(FuncFormatter(lambda x, _: f"{x / 1_000:.0f}K"))
# or for millions:
ax.yaxis.set_major_formatter(FuncFormatter(lambda x, _: f"{x / 1_000_000:.1f}M"))
```

## Make It Accessible

**Use a minimum font size of 10pt.** Titles should be larger (13-16pt). If the chart will be projected or printed small, increase sizes further.

```python
plt.rcParams.update({
    "font.size": 11,
    "axes.titlesize": 14,
    "axes.labelsize": 12,
})
```

**Don't rely on color alone** to distinguish data series. Use direct labels, different line styles (`--`, `-.`, `:`), markers, or hatch patterns alongside color.

```python
ax.plot(x, y1, color="#e63946", linewidth=2, linestyle="-", marker="o", markersize=5)
ax.plot(x, y2, color="#1d3557", linewidth=2, linestyle="--", marker="s", markersize=5)
```

**Use hatch patterns** to differentiate bars or filled areas without relying on color. This is especially useful for colorblind-friendly charts and print/grayscale output. Common patterns: `/`, `\\`, `x`, `+`, `o`, `.`, `*`. Repeat characters to increase density (e.g., `//` is denser than `/`).

```python
fig, ax = plt.subplots(figsize=(6, 3.5))
categories = ["Segment A", "Segment B", "Segment C"]
base = [20, 35, 30]
extra = [25, 32, 34]

ax.barh(categories, base, color="#cccccc", edgecolor="#333333", linewidth=0.8,
        hatch="//", label="Existing")
ax.barh(categories, extra, left=base, color="#e63946", edgecolor="#333333",
        linewidth=0.8, hatch="xx", label="New")

ax.spines[["top", "right"]].set_visible(False)
ax.set_title("New revenue now exceeds existing in Segment C",
             loc="left", fontweight="bold")
plt.tight_layout()
```

**Test readability** by viewing the chart at the size it will actually be consumed — on a slide, in a report, on a dashboard. Step back from your screen to check that the key message is still visible.

## Workflow: Render, Inspect, Fix, Repeat

**This is the most important section in this skill.** Code that looks correct often produces charts with overlapping labels, clipped titles, or misaligned annotations. You cannot judge a chart from its code — you must look at the rendered image. Every chart must go through at least one render-inspect-fix cycle before it is finished.

### Step 1: Render to a temporary file

After generating any plot, always save it to a temporary PNG and read that image file to visually inspect the result. Never skip this step.

```python
import tempfile
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
# ... build your chart ...

with tempfile.NamedTemporaryFile(suffix=".png", delete=False) as f:
    fig.savefig(f.name, bbox_inches="tight", dpi=150)
    print(f.name)  # read this file to visually inspect the result
plt.close(fig)
```

### Step 2: Inspect the image for problems

Open and look at the saved image. Check for every one of these issues:

- **Overlapping text** — bar labels overlapping each other or bleeding into neighboring bars. This is the single most common problem and it almost always happens on the first render.
- **Overlapping annotations** — `ax.text` or `ax.annotate` calls placed at coordinates that looked reasonable in code but collide in the rendered output.
- **Clipped elements** — titles, subtitles, or labels cut off by the figure boundary.
- **Labels overlapping data** — text placed on top of bars, lines, or other data elements instead of next to them.
- **Cramped layout** — too many elements in too small a figure, leaving no white space.
- **Unreadable font sizes** — text that is too small at the actual output size.

### Step 3: Fix and re-render

If anything looks wrong — and it usually will on the first pass — fix the issue, save again, and inspect again. Repeat until the chart is clean. Common fixes for common problems:

**Overlapping bar labels** — the most frequent issue. When values are placed on top of or next to bars, they collide if bars are close together or values are similar.
- Increase `figsize` to give more room between bars
- Adjust the offset in `ax.text` (e.g., increase the `+ 1.5` padding)
- For horizontal bars with long value labels, extend `ax.set_xlim` to add right-side margin
- If labels still collide, consider labeling only the highlighted bar(s) and removing labels from grey/background bars

**Overlapping line labels** — when labeling lines directly at their endpoints, labels for lines with similar values will overlap.
- Add vertical offsets to separate them: nudge one label up and the other down
- If lines converge at the end, label them at a point where they are further apart
- As a last resort, use a minimal legend instead of direct labels

**Clipped titles or subtitles:**
- Use `bbox_inches="tight"` in `savefig` (always)
- Add `plt.subplots_adjust(top=0.85)` to create room above the axes
- Reduce `fontsize` of the title if it wraps or overflows

**Cramped layout:**
- Increase `figsize` — this is almost always the right first move
- Use `plt.tight_layout(pad=1.5)` or `plt.subplots_adjust()` for more padding
- Use `ax.set_xlim` / `ax.set_ylim` to add margin beyond the data range so labels outside the data area have space

**Annotations pointing to the wrong place or overlapping data:**
- Adjust `xytext` coordinates to move the annotation text
- Change `arrowprops` to use a longer or differently angled arrow
- Test several positions — what looks right in coordinate space often doesn't once rendered

### Step 4: Verify the final result

After fixes, render one more time and confirm:
- All text is readable and non-overlapping
- The highlighted data stands out immediately
- The title communicates the takeaway
- The chart looks intentional, not auto-generated

**Do not deliver a chart you have not visually inspected.** A chart with overlapping labels is worse than no chart at all — it looks broken and undermines the message.
