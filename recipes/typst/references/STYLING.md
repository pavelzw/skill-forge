# Typst Styling Guide

Complete guide to Typst's styling system using set rules, show rules, and the context system.

## Set Rules

Set rules configure the default properties of elements. They apply from their location until the end of the current scope.

### Basic Set Rules

```typst
// Set text properties
#set text(
  font: "Libertinus Serif",
  size: 11pt,
  fill: black,
)

// Set page layout
#set page(
  paper: "a4",
  margin: (x: 2.5cm, y: 3cm),
)

// Set paragraph style
#set par(
  justify: true,
  leading: 0.65em,    // Line spacing
  first-line-indent: 1em,
)

// Set heading numbering
#set heading(numbering: "1.1")
```

### Common Set Rule Targets

| Element | Common Parameters |
|---------|-------------------|
| `text` | `font`, `size`, `fill`, `weight`, `style` |
| `page` | `paper`, `margin`, `numbering`, `header`, `footer` |
| `par` | `justify`, `leading`, `first-line-indent` |
| `heading` | `numbering`, `outlined` |
| `list` | `marker`, `indent` |
| `enum` | `numbering`, `indent` |
| `figure` | `placement`, `gap` |
| `table` | `columns`, `stroke`, `fill`, `align` |
| `math.equation` | `numbering` |

### Conditional Set Rules

```typst
// Set-if rule
#set text(red) if important

// Set based on condition
#{
  let draft = true
  if draft {
    set text(gray)
    set page(background: rotate(45deg, text(60pt, gray)[DRAFT]))
  }
  // Document content
}
```

### Scoped Set Rules

```typst
// Set rules are scoped to blocks
#[
  #set text(blue)
  This is blue.
]
This is back to default.

// Function scope
#let warning(body) = {
  set text(red)
  [*Warning:* #body]
}
#warning[Be careful!]
Normal text here.
```

## Show Rules

Show rules transform how elements are displayed.

### Show-Set Rules

Combine a selector with property configuration:

```typst
// All headings in blue
#show heading: set text(blue)

// Level 1 headings with specific styling
#show heading.where(level: 1): set text(size: 20pt, weight: "bold")

// Style code blocks
#show raw: set text(font: "Fira Code", size: 9pt)
```

### Transformational Show Rules

Replace element rendering with custom functions:

```typst
// Transform all headings
#show heading: it => {
  set text(blue)
  block(smallcaps(it.body))
}

// Custom level-1 headings
#show heading.where(level: 1): it => {
  pagebreak(weak: true)
  v(2em)
  text(20pt, weight: "bold")[#it.body]
  v(1em)
}

// Transform figures
#show figure: it => {
  set align(center)
  box(stroke: 1pt, inset: 10pt)[
    #it.body
    #v(5pt)
    #text(9pt)[#it.caption]
  ]
}
```

### Text Show Rules

Replace specific text patterns:

```typst
// Replace text
#show "Typst": [*Typst*]

// Replace with function
#show "TODO": it => box(fill: yellow, inset: 2pt)[#it]

// Using regex
#show regex("\d{4}-\d{2}-\d{2}"): it => {
  text(blue, underline(it))
}
```

### Selector Types

| Selector | Example | Matches |
|----------|---------|---------|
| Element | `heading` | All headings |
| Where | `heading.where(level: 1)` | Level 1 headings |
| Text | `"word"` | Exact text |
| Regex | `regex("\d+")` | Pattern matches |
| Label | `<my-label>` | Labeled elements |

### Combined Selectors

```typst
// Multiple selectors with or
#show heading.where(level: 1).or(heading.where(level: 2)): set text(blue)

// Apply to multiple elements
#show (heading, strong): set text(navy)
```

## Page Setup

### Page Dimensions

```typst
#set page(
  paper: "a4",           // Predefined size
  // Or custom size:
  width: 21cm,
  height: 29.7cm,
)

// Common paper sizes:
// "a0" - "a10", "b0" - "b10" (ISO)
// "us-letter", "us-legal", "us-executive"
```

### Margins

```typst
// Uniform margins
#set page(margin: 2cm)

// Different margins per side
#set page(margin: (
  top: 3cm,
  bottom: 2.5cm,
  left: 2cm,
  right: 2cm,
))

// Shorthand for x/y
#set page(margin: (x: 2cm, y: 3cm))

// Book-style margins
#set page(margin: (
  inside: 3cm,    // Binding side
  outside: 2cm,
  top: 2.5cm,
  bottom: 2cm,
))
```

### Headers and Footers

```typst
// Simple header
#set page(header: [Document Title])

// Header with alignment
#set page(header: [
  Document Title
  #h(1fr)
  Draft Version
])

// Context-aware header (page numbers)
#set page(header: context [
  #h(1fr)
  Page #counter(page).display()
])

// Different header on first page
#set page(header: context {
  if counter(page).get().first() > 1 [
    Document Title #h(1fr) Page #counter(page).display()
  ]
})

// Footer with page number
#set page(
  footer: context [
    #h(1fr)
    #counter(page).display("1 of 1", both: true)
    #h(1fr)
  ]
)
```

### Columns

```typst
// Two-column layout
#set page(columns: 2)

// With custom gutter
#set page(columns: 2, column-gutter: 1cm)

// Manual column break
#colbreak()
```

## Context System

The `context` keyword allows expressions to access their location in the document.

### Style Context

```typst
// Access current text size
#context {
  let size = text.size
  [Current size: #size]
}

// Access current font
#context {
  let font = text.font
  [Current font: #font.first()]
}
```

### Location Context

```typst
// Current page number
#context counter(page).display()

// Total pages
#context counter(page).final().first()

// Page X of Y
#context [
  Page #counter(page).display() of #counter(page).final().first()
]

// Current heading
#context {
  let headings = query(selector(heading).before(here()))
  if headings.len() > 0 {
    headings.last().body
  }
}
```

### Query System

```typst
// Find all headings
#context {
  let all-headings = query(heading)
  for h in all-headings [
    - #h.body
  ]
}

// Find specific elements
#context {
  let figures = query(figure.where(kind: image))
  [Found #figures.len() images]
}

// Query with location
#context {
  let prev = query(selector(heading).before(here()))
  let next = query(selector(heading).after(here()))
}
```

## Counters

### Built-in Counters

```typst
// Page counter
#counter(page).display()
#counter(page).display("i")        // Roman numerals
#counter(page).display("I")        // Uppercase Roman

// Heading counter
#counter(heading).display()
#counter(heading).display("1.1")

// Figure counter
#counter(figure).display()
```

### Counter Operations

```typst
// Reset counter
#counter(page).update(1)

// Step counter
#counter(page).step()

// Update with function
#counter(page).update(n => n + 5)

// Get counter value
#context counter(page).get()           // Current value
#context counter(page).final()         // Final value
#context counter(page).at(<label>)     // Value at label
```

### Custom Counters

```typst
#let example-counter = counter("example")

// Usage
#let example(body) = {
  example-counter.step()
  [*Example #context example-counter.display():* #body]
}

#example[First example]
#example[Second example]
```

## Document Metadata

```typst
#set document(
  title: "My Document",
  author: ("Alice", "Bob"),
  keywords: ("typst", "document"),
  date: datetime.today(),
)
```

## Template Pattern

```typst
// template.typ
#let article(
  title: none,
  authors: (),
  abstract: none,
  body,
) = {
  // Document setup
  set document(title: title, author: authors)
  set page(paper: "a4", margin: 2.5cm)
  set text(font: "Libertinus Serif", size: 11pt)
  set par(justify: true)
  set heading(numbering: "1.1")

  // Title block
  if title != none {
    align(center)[
      #text(17pt, weight: "bold")[#title]
      #v(1em)
      #authors.join(", ", last: " and ")
    ]
  }

  // Abstract
  if abstract != none {
    v(1em)
    align(center)[
      #block(width: 80%)[
        *Abstract* \
        #abstract
      ]
    ]
    v(1em)
  }

  // Body
  body
}

// main.typ
#import "template.typ": article

#show: article.with(
  title: "My Research Paper",
  authors: ("Alice Smith", "Bob Jones"),
  abstract: [This paper presents...],
)

= Introduction
Content goes here...
```
