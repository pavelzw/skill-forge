# Polylux (Presentations)

Polylux is a lightweight presentation package inspired by LaTeX beamer.

## Quick Start

```typst
#import "@preview/polylux:0.4.0": *

#set page(paper: "presentation-16-9")

#slide[
  = Welcome

  This is my first slide.
]

#slide[
  == Content Slide

  - Point one
  - Point two
]
```

## Dynamic Content

```typst
// Show content on specific subslides
#slide[
  #only(1)[First this appears]
  #only(2)[Then this appears]
  #only(3)[Finally this]
]

// Uncover content progressively (preserves space)
#slide[
  #uncover(1)[Always visible]
  #uncover(2)[Appears second]
  #uncover(3)[Appears third]
]

// One by one reveals
#slide[
  #one-by-one[
    - First item
    - Second item
    - Third item
  ]
]
```

## Layout Utilities

```typst
// Side by side content
#slide[
  #side-by-side[
    Left column content
  ][
    Right column content
  ]
]
```

## Progress Indicators

```typst
#slide[
  #progress-bar

  Content here...
]
```

## Handout Mode

```typst
// Generate PDF without animations
#enable-handout-mode(true)
```

## Key Functions

| Function | Description |
|----------|-------------|
| `#only(n)` | Show only on subslide n |
| `#uncover(n)` | Reveal on subslide n (preserves space) |
| `#one-by-one` | Sequential reveals |
| `#alternatives` | Switch between content |
| `#side-by-side` | Two-column layout |

## Resources

- [Typst Universe](https://typst.app/universe/package/polylux)
- [Documentation](https://polylux.dev/book/)
