# Codly (Code Presentation)

Codly enhances code blocks with line numbers, highlighting, and annotations.

## Quick Start

```typst
#import "@preview/codly:1.3.0": *
#import "@preview/codly-languages:0.1.8": *

#show: codly-init.with()
#codly(languages: codly-languages)

```python
def greet(name):
    print(f"Hello, {name}!")
`` `
```

## Configuration

```typst
#codly(
  // Line numbers
  number-format: i => text(gray)[#i],

  // Zebra striping
  zebra-fill: luma(245),

  // Code styling
  stroke: 1pt + gray,
  radius: 4pt,
  padding: 8pt,
)
```

## Line Highlighting

```typst
#codly(highlights: (
  (line: 2, fill: yellow),
  (line: 4, tag: [Important!]),
))

```python
def calculate(x, y):
    result = x + y  # Highlighted
    return result   # Tagged
`` `
```

## Line Ranges and Skipping

```typst
// Show only lines 5-10
#codly(range: (5, 10))

// Skip lines (show ellipsis)
#codly(skip-lines: ((3, 5),))
```

## Annotations

```typst
#codly(annotations: (
  (start: 2, end: 4, label: [Loop body]),
))
```

## Key Features

- Smart indentation during line wrapping
- Line numbering with customizable formatting
- Syntax highlighting with language icons
- Code annotations for marking sections
- Line highlighting with fills and tags
- Zebra striping for readability

## Resources

- [Typst Universe](https://typst.app/universe/package/codly)
- [codly-languages](https://typst.app/universe/package/codly-languages)
