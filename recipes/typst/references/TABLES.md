# Typst Tables Guide

Complete guide to creating and styling tables in Typst.

## Basic Tables

```typst
// Simple table
#table(
  columns: 2,
  [*Name*], [*Score*],
  [Alice], [95],
  [Bob], [87],
)

// Three columns
#table(
  columns: 3,
  [ID], [Name], [Department],
  [1], [Alice], [Engineering],
  [2], [Bob], [Marketing],
  [3], [Carol], [Sales],
)
```

## Column Sizing

### Width Specifications

| Type | Example | Description |
|------|---------|-------------|
| `auto` | `auto` | Fit to content (default) |
| Length | `6cm`, `100pt` | Fixed width |
| Percentage | `40%` | Percentage of available space |
| Fraction | `1fr` | Distribute remaining space |

```typst
// Mixed column widths
#table(
  columns: (auto, 1fr, 100pt),
  [ID], [Description], [Price],
  [001], [A longer description that takes more space], [\$9.99],
)

// Equal fractional columns
#table(
  columns: (1fr, 1fr, 1fr),
  [A], [B], [C],
)

// Proportional fractions
#table(
  columns: (1fr, 2fr, 1fr),  // Middle column is twice as wide
  [Left], [Center], [Right],
)
```

## Headers and Footers

```typst
// Using table.header for semantic headers
#table(
  columns: 3,
  table.header[Product][Qty][Price],
  [Widget], [10], [\$5.00],
  [Gadget], [5], [\$12.00],
)

// With footer
#table(
  columns: 3,
  table.header[Product][Qty][Price],
  [Widget], [10], [\$5.00],
  [Gadget], [5], [\$12.00],
  table.footer[Total][][\$110.00],
)

// Header/footer repeat on page breaks
#show figure: set block(breakable: true)
#figure(
  table(
    columns: 2,
    table.header[Name][Value],
    // Many rows...
    table.footer[End][---],
  )
)
```

## Cell Alignment

```typst
// Single alignment for all
#table(
  columns: 3,
  align: center,
  [A], [B], [C],
)

// Per-column alignment (array)
#table(
  columns: 3,
  align: (right, center, left),
  [1], [2], [3],
)

// Function-based alignment
#table(
  columns: 3,
  align: (x, y) => {
    if y == 0 { center }        // Header centered
    else if x == 0 { right }    // First column right
    else { left }               // Rest left
  },
  [*ID*], [*Name*], [*Notes*],
  [1], [Alice], [Engineer],
)

// Combined alignment (horizontal + vertical)
#table(
  columns: 2,
  align: center + horizon,
  rows: (50pt, 50pt),
  [Centered], [Both ways],
)
```

## Row Heights

```typst
// Fixed row heights
#table(
  columns: 2,
  rows: 30pt,
  [A], [B],
)

// Array of heights
#table(
  columns: 2,
  rows: (40pt, 30pt, 30pt),
  [Header], [Header],
  [Row 1], [Data],
  [Row 2], [Data],
)

// Auto and fixed mixed
#table(
  columns: 2,
  rows: (auto, 50pt),
  [Fits content], [Fits content],
  [Fixed height], [Fixed height],
)
```

## Strokes and Borders

### Global Stroke

```typst
// No borders
#table(stroke: none, columns: 2, [A], [B])

// Thin stroke
#table(stroke: 0.5pt, columns: 2, [A], [B])

// Colored stroke
#table(stroke: blue, columns: 2, [A], [B])

// Combined thickness and color
#table(stroke: 0.5pt + gray, columns: 2, [A], [B])
```

### Selective Strokes

```typst
// Only horizontal lines
#table(
  stroke: (x: none, y: 0.5pt),
  columns: 2,
  [A], [B],
  [C], [D],
)

// Only vertical lines
#table(
  stroke: (x: 0.5pt, y: none),
  columns: 2,
  [A], [B],
)

// Per-side control
#table(
  stroke: (
    left: 2pt,
    right: 2pt,
    top: 1pt,
    bottom: 1pt,
  ),
  columns: 2,
  [A], [B],
)
```

### Function-Based Strokes

```typst
// Header separator only
#table(
  columns: 2,
  stroke: (_, y) => if y == 0 { (bottom: 1pt) },
  [*Header*], [*Header*],
  [Data], [Data],
)

// Alternating row borders
#table(
  columns: 2,
  stroke: (_, y) => if calc.odd(y) { (top: 0.5pt) },
  [A], [B],
  [C], [D],
  [E], [F],
)
```

### Manual Lines

```typst
// Horizontal line
#table(
  columns: 2,
  stroke: none,
  [Header], [Header],
  table.hline(stroke: 1pt),
  [Data], [Data],
)

// Vertical line
#table(
  columns: 3,
  stroke: none,
  [A], table.vline(), [B], [C],
)

// Partial lines
#table(
  columns: 3,
  stroke: none,
  [A], [B], [C],
  table.hline(start: 1, end: 3),  // Skip first column
  [D], [E], [F],
)
```

## Fill and Background

### Solid Fill

```typst
// All cells same color
#table(
  columns: 2,
  fill: gray.lighten(80%),
  [A], [B],
)
```

### Column Striping

```typst
// Alternating column colors
#table(
  columns: 3,
  fill: (rgb("EAF2F5"), none, rgb("EAF2F5")),
  [A], [B], [C],
  [D], [E], [F],
)
```

### Row Striping

```typst
// Alternating row colors
#table(
  columns: 2,
  fill: (_, y) => if calc.odd(y) { rgb("EAF2F5") },
  [Header], [Header],
  [Row 1], [Data],
  [Row 2], [Data],
  [Row 3], [Data],
)

// Skip header row
#table(
  columns: 2,
  fill: (_, y) => if y > 0 and calc.odd(y) { rgb("EAF2F5") },
  [*Header*], [*Header*],
  [Row 1], [Data],
  [Row 2], [Data],
)
```

### Per-Cell Fill

```typst
#table(
  columns: 2,
  [Normal], table.cell(fill: yellow)[Highlighted],
  [Normal], [Normal],
)
```

## Cell Spanning

### Column Span

```typst
#table(
  columns: 3,
  table.cell(colspan: 3)[Full Width Header],
  [A], [B], [C],
  table.cell(colspan: 2)[Spans Two], [One],
)
```

### Row Span

```typst
#table(
  columns: 2,
  table.cell(rowspan: 2)[Spans Two Rows], [A],
  [B],
  [C], [D],
)
```

### Combined Spanning

```typst
#table(
  columns: 3,
  table.cell(colspan: 2, rowspan: 2)[Large Cell], [A],
  [B],
  [C], [D], [E],
)
```

## Cell Styling

```typst
#table(
  columns: 2,
  // Override stroke
  table.cell(stroke: 2pt + red)[Important], [Normal],

  // Override fill
  table.cell(fill: yellow)[Highlighted], [Normal],

  // Override alignment
  table.cell(align: right)[Right], [Default],

  // Override inset
  table.cell(inset: 20pt)[Padded], [Normal],
)
```

## Inset (Cell Padding)

```typst
// Uniform inset
#table(
  columns: 2,
  inset: 10pt,
  [A], [B],
)

// Per-side inset
#table(
  columns: 2,
  inset: (x: 15pt, y: 8pt),
  [A], [B],
)

// Full control
#table(
  columns: 2,
  inset: (left: 5pt, right: 5pt, top: 10pt, bottom: 10pt),
  [A], [B],
)
```

## Data Import

### From CSV

```typst
#let data = csv("sales.csv")

// Simple display
#table(
  columns: data.first().len(),
  ..data.flatten()
)

// With header styling
#table(
  columns: data.first().len(),
  table.header(..data.first().map(x => [*#x*])),
  ..data.slice(1).flatten()
)
```

### From JSON

```typst
#let data = json("data.json")

#table(
  columns: 2,
  [*Key*], [*Value*],
  ..data.pairs().map(((k, v)) => ([#k], [#v])).flatten()
)
```

## Breakable Tables

Tables in figures don't break across pages by default:

```typst
// Enable breaking
#show figure: set block(breakable: true)

#figure(
  table(
    columns: 2,
    table.header[Name][Value],
    // Many rows that span pages...
  ),
  caption: [Long data table],
)
```

## Table vs Grid

Use `grid` for layout (no default strokes), `table` for data:

```typst
// Grid for layout
#grid(
  columns: (1fr, 1fr),
  gutter: 20pt,
  [Left column content],
  [Right column content],
)

// Table for data
#table(
  columns: 2,
  [Name], [Value],
  [A], [1],
)
```

## Complete Example

```typst
#let data = (
  ("Product", "Q1", "Q2", "Q3", "Q4"),
  ("Widgets", "150", "180", "200", "175"),
  ("Gadgets", "90", "110", "95", "120"),
  ("Tools", "200", "190", "210", "230"),
)

#figure(
  table(
    columns: 5,
    align: (left, ..range(4).map(_ => right)),
    fill: (_, y) => if y == 0 { rgb("1a73e8").lighten(80%) },
    stroke: (x: none, y: 0.5pt + gray),
    inset: (x: 12pt, y: 8pt),

    // Header
    table.header(..data.first().map(x => [*#x*])),
    table.hline(stroke: 1pt),

    // Data rows
    ..data.slice(1).flatten(),
  ),
  caption: [Quarterly Sales Data],
)
```
