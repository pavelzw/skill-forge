# CeTZ (Drawing)

CeTZ is a drawing library for Typst inspired by TikZ and Processing. It provides vector graphics capabilities with automatic canvas sizing and a flexible coordinate system.

## Quick Start

```typst
#import "@preview/cetz:0.4.2"

#cetz.canvas({
  import cetz.draw: *

  circle((0, 0), radius: 1)
  line((0, 0), (2, 1))
  rect((0, -2), (3, -1))
})
```

Draw functions are imported inside the canvas block to avoid overriding Typst's built-in functions like `line`.

## Canvas Configuration

```typst
// Default: 1 unit = 1cm
#cetz.canvas({
  import cetz.draw: *
  circle((0, 0), radius: 1)  // 1cm radius
})

// Custom unit size
#cetz.canvas(length: 0.5cm, {
  import cetz.draw: *
  circle((0, 0), radius: 1)  // 0.5cm radius
})

// Relative to parent width
#cetz.canvas(length: 10%, {
  import cetz.draw: *
  rect((0, 0), (5, 3))  // 50% x 30% of parent width
})
```

The canvas automatically grows to fit content.

## Basic Shapes

```typst
#cetz.canvas({
  import cetz.draw: *

  // Circle
  circle((0, 0), radius: 1)
  circle((3, 0), radius: (1, 0.5))  // Ellipse

  // Rectangle
  rect((0, -2), (2, -1))
  rect((3, -2), (5, -1), radius: 0.2)  // Rounded corners

  // Line and polyline
  line((0, -3), (2, -4))
  line((3, -3), (4, -4), (5, -3), (4, -3.5), close: true)

  // Arc
  arc((0, -5), start: 0deg, stop: 270deg, radius: 0.5)

  // Bezier curves
  bezier((0, -6), (3, -6), (1, -7), (2, -5))

  // Content (embed Typst content)
  content((5, 0), [Hello World])
})
```

## Styling

Three core styling parameters: `fill`, `stroke`, and `fill-rule`.

```typst
#cetz.canvas({
  import cetz.draw: *

  // Basic fill and stroke
  circle((0, 0), radius: 0.5, fill: blue, stroke: red)

  // Stroke options
  line((2, 0), (4, 0), stroke: (
    paint: green,
    thickness: 2pt,
    dash: "dashed",
    cap: "round",
  ))

  // No stroke
  rect((0, -2), (1, -1), fill: yellow, stroke: none)

  // Global styling with set-style
  set-style(
    stroke: (paint: navy, thickness: 1.5pt),
    fill: orange.lighten(50%),
  )
  circle((3, -1.5), radius: 0.5)
  rect((5, -2), (6, -1))
})
```

### Style Hierarchy

1. Function parameters (highest priority)
2. Element-type rules
3. Global defaults (lowest priority)

Dictionary values merge; non-dictionary values override entirely.

## Coordinate Systems

CeTZ supports 11 coordinate types:

### Cartesian (XYZ)

```typst
// Implicit array
circle((1, 2))
circle((1, 2, 0))  // With z

// Explicit dictionary
circle((x: 1, y: 2))
```

### Previous Position

```typst
line((0, 0), (1, 1))
circle(())  // Uses last position (1, 1)
```

### Relative

```typst
circle((0, 0), name: "a")
// 1 unit right and 1 up from "a"
circle((rel: (1, 1), to: "a.center"))
```

### Polar

```typst
// angle and radius
circle((angle: 45deg, radius: 2))
circle((45deg, 2))  // Shorthand

// Elliptical radius
circle((30deg, (2, 1)))
```

### Anchor References

```typst
circle((0, 0), name: "c")
line("c.north", "c.south")
line("c.east", "c.west")
circle("c.north-east", radius: 0.1)
```

### Perpendicular Intersection

```typst
// Vertical from a, horizontal from b
circle(((0, 0), "|-", (2, 2)))  // Result: (0, 2)
circle(((0, 0), "-|", (2, 2)))  // Result: (2, 0)
```

### Interpolation

```typst
// Midpoint between two coordinates
circle(((0, 0), 50%, (4, 4)))

// Absolute distance
circle(((0, 0), 1, (4, 4)))  // 1 unit from start
```

### Projection

```typst
// Project point onto line segment
circle((project: (1, 2), onto: ((0, 0), (4, 0))))
```

## Anchors

Every named element has anchor points for positioning.

### Naming Elements

```typst
circle((0, 0), name: "my-circle")
rect((2, 0), (4, 1), name: "my-rect")
```

### Anchor Types

**Border anchors** - compass directions and angles:
```typst
circle((0, 0), name: "c")
// Compass directions
content("c.north", [N])
content("c.south-east", [SE])
// Angle-based (0deg = right, 90deg = up)
content("c.45deg", [45°])
```

**Path anchors** - positions along the path:
```typst
line((0, 0), (4, 0), name: "L")
circle("L.start", radius: 0.1)
circle("L.mid", radius: 0.1)
circle("L.end", radius: 0.1)
circle("L.25%", radius: 0.1)  // 25% along path
```

### Positioning with Anchors

```typst
// Place circle's west anchor at origin
circle((0, 0), anchor: "west", radius: 0.5)
```

## Transformations

```typst
#cetz.canvas({
  import cetz.draw: *

  // Group transformations
  group({
    translate((2, 0))
    rotate(45deg)
    scale(0.5)
    rect((0, 0), (1, 1))
  })

  // Scale x and y independently
  group({
    scale((x: 2, y: 0.5))
    circle((0, -2), radius: 1)
  })

  // Rotate around a point
  group({
    rotate(30deg, origin: (1, 1))
    rect((0, 0), (2, 2))
  })
})
```

## Decorations and Marks

```typst
#cetz.canvas({
  import cetz.draw: *

  // Arrow marks
  line((0, 0), (2, 0), mark: (end: ">"))
  line((0, -1), (2, -1), mark: (start: "<", end: ">"))

  // Different mark styles
  line((0, -2), (2, -2), mark: (end: "o"))      // Circle
  line((0, -3), (2, -3), mark: (end: "|"))      // Bar
  line((0, -4), (2, -4), mark: (end: "stealth")) // Stealth arrow
})
```

## Groups and Scoping

```typst
#cetz.canvas({
  import cetz.draw: *

  // Styles are scoped to groups
  group({
    set-style(fill: red)
    circle((0, 0), radius: 0.5)
    circle((1, 0), radius: 0.5)
  })

  // Outside group: default style
  circle((2, 0), radius: 0.5)
})
```

## CeTZ-Plot for Charts

```typst
#import "@preview/cetz:0.4.2"
#import "@preview/cetz-plot:0.1.3": plot, chart

#cetz.canvas({
  plot.plot(
    size: (8, 5),
    x-label: [$x$],
    y-label: [$f(x)$],
    x-tick-step: 1,
    y-tick-step: 0.5,
    {
      // Function plot
      plot.add(
        domain: (-2, 2),
        x => calc.pow(x, 2),
        label: [$x^2$],
      )

      // Data points
      plot.add(
        ((0, 0), (1, 1), (2, 4)),
        mark: "o",
        label: [Data],
      )
    }
  )
})
```

### Chart Types

```typst
#cetz.canvas({
  import chart: *

  // Bar chart
  barchart(
    size: (6, 4),
    (
      ([A], 10),
      ([B], 20),
      ([C], 15),
    ),
  )
})
```

## Tree Diagrams

```typst
#import "@preview/cetz:0.4.2"

#cetz.canvas({
  import cetz.draw: *
  import cetz.tree: tree

  tree(
    draw-node: (node, ..) => content((), node.content),
    draw-edge: (from, to, ..) => line(from, to),
    ([Root],
      ([Child 1],
        [Leaf 1],
        [Leaf 2],
      ),
      ([Child 2],
        [Leaf 3],
      ),
    )
  )
})
```

## Practical Examples

### Flowchart

```typst
#cetz.canvas({
  import cetz.draw: *

  // Nodes
  rect((-1, 0), (1, 1), name: "start", radius: 0.2)
  content("start", [Start])

  rect((-1, -2), (1, -1), name: "process")
  content("process", [Process])

  rect((-1, -4), (1, -3), name: "end", radius: 0.2)
  content("end", [End])

  // Arrows
  line("start.south", "process.north", mark: (end: ">"))
  line("process.south", "end.north", mark: (end: ">"))
})
```

### Annotated Diagram

```typst
#cetz.canvas({
  import cetz.draw: *

  circle((0, 0), radius: 2, name: "main")
  content("main", [$A$])

  // Annotations
  line("main.45deg", (rel: (1, 1), to: "main.45deg"), mark: (end: ">"))
  content((rel: (1.2, 1.2), to: "main.45deg"), [Annotation], anchor: "south-west")
})
```

## Related Libraries

- **cetz-plot** - Plotting and charts (line, bar, pie, etc.)
- **cetz-venn** - Venn diagrams

## Resources

- [Typst Universe](https://typst.app/universe/package/cetz)
- [Documentation](https://cetz-package.github.io/docs/)
- [cetz-plot](https://typst.app/universe/package/cetz-plot)
