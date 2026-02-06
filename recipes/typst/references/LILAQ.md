# Lilaq (Scientific Visualization)

Lilaq is a plotting library for scientific data visualization.

## Quick Start

```typst
#import "@preview/lilaq:0.5.0" as lq

#lq.diagram(
  lq.plot((0, 1, 2, 3, 4), (3, 5, 4, 2, 3))
)
```

## Line and Scatter Plots

```typst
#let xs = (0, 1, 2, 3, 4)

#lq.diagram(
  title: [Experimental Data],
  xlabel: $x$,
  ylabel: $y$,

  // Line plot with markers
  lq.plot(xs, (3, 5, 4, 2, 3), mark: "s", label: [Series A]),

  // Function plot
  lq.plot(xs, x => 2*calc.cos(x) + 3, mark: "o", label: [Series B]),
)
```

## Bar Charts

```typst
#lq.diagram(
  lq.bar(
    (1, 2, 3, 4),
    (10, 25, 15, 30),
    width: 0.6,
  )
)
```

## Error Bars

```typst
#lq.diagram(
  lq.plot(
    (1, 2, 3, 4),
    (10, 15, 12, 18),
    yerr: (1, 2, 1.5, 2.5),
  )
)
```

## Scatter Plots

```typst
#lq.diagram(
  lq.scatter(
    (1, 2, 3, 4, 5),
    (2, 4, 3, 5, 4),
    size: 50,
    color: blue,
  )
)
```

## Multiple Axes

```typst
#lq.diagram(
  // Primary axis
  lq.plot(xs, ys1, label: [Temperature]),

  // Secondary axis
  lq.twinx(
    lq.plot(xs, ys2, color: red, label: [Pressure])
  ),
)
```

## Styling

```typst
#lq.diagram(
  width: 10cm,
  height: 6cm,
  grid: true,

  // Axis configuration
  xlim: (0, 10),
  ylim: (-1, 1),

  lq.plot(xs, ys),
)
```

## Supported Plot Types

- Line plots and scatter plots
- Bar charts and stem plots
- Boxplots and error bars
- Colormesh and contour plots
- Color bars and legends

## Resources

- [Typst Universe](https://typst.app/universe/package/lilaq)
- [Documentation](https://lilaq.org/)
- [Quickstart](https://lilaq.org/docs/quickstart)
